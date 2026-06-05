# BPFAdjustOpt.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/BPF/BPFAdjustOpt.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Applies target-specific post-processing or tuning adjustments to generated code.
- 目的（中文）: 对生成代码执行目标专用的后处理或调优修正。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===---------------- BPFAdjustOpt.cpp - Adjust Optimization --------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // Adjust optimization to make the code more kernel verifier friendly.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "BPF.h"
  14: #include "BPFCORE.h"
  15: #include "llvm/IR/Instruction.h"
  16: #include "llvm/IR/Instructions.h"
  17: #include "llvm/IR/IntrinsicsBPF.h"
  18: #include "llvm/IR/Module.h"
  19: #include "llvm/IR/PatternMatch.h"
  20: #include "llvm/IR/Type.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 21-40

```cpp
  21: #include "llvm/IR/User.h"
  22: #include "llvm/IR/Value.h"
  23: #include "llvm/Pass.h"
  24: #include "llvm/Transforms/Utils/BasicBlockUtils.h"
  25: 
  26: #define DEBUG_TYPE "bpf-adjust-opt"
  27: 
  28: using namespace llvm;
  29: using namespace llvm::PatternMatch;
  30: 
  31: static cl::opt<bool>
  32:     DisableBPFserializeICMP("bpf-disable-serialize-icmp", cl::Hidden,
  33:                             cl::desc("BPF: Disable Serializing ICMP insns."),
  34:                             cl::init(false));
  35: 
  36: static cl::opt<bool> DisableBPFavoidSpeculation(
  37:     "bpf-disable-avoid-speculation", cl::Hidden,
  38:     cl::desc("BPF: Disable Avoiding Speculative Code Motion."),
  39:     cl::init(false));
  40: 
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。

### Lines 41-60

```cpp
  41: namespace {
  42: class BPFAdjustOptImpl {
  43:   struct PassThroughInfo {
  44:     Instruction *Input;
  45:     Instruction *UsedInst;
  46:     uint32_t OpIdx;
  47:     PassThroughInfo(Instruction *I, Instruction *U, uint32_t Idx)
  48:         : Input(I), UsedInst(U), OpIdx(Idx) {}
  49:   };
  50: 
  51: public:
  52:   BPFAdjustOptImpl(Module *M) : M(M) {}
  53: 
  54:   bool run();
  55: 
  56: private:
  57:   Module *M;
  58:   SmallVector<PassThroughInfo, 16> PassThroughs;
  59: 
  60:   bool adjustICmpToBuiltin();
```

- EN: This chunk introduces interfaces or data structures such as BPFAdjustOptImpl, PassThroughInfo, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as Input, BPFAdjustOptImpl contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: 这一段引入了 BPFAdjustOptImpl, PassThroughInfo 等接口或数据结构，用于组织该文件暴露的目标专用行为。 Input, BPFAdjustOptImpl 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 61-80

```cpp
  61:   void adjustBasicBlock(BasicBlock &BB);
  62:   bool serializeICMPCrossBB(BasicBlock &BB);
  63:   void adjustInst(Instruction &I);
  64:   bool serializeICMPInBB(Instruction &I);
  65:   bool avoidSpeculation(Instruction &I);
  66:   bool insertPassThrough();
  67: };
  68: 
  69: } // End anonymous namespace
  70: 
  71: bool BPFAdjustOptImpl::run() {
  72:   bool Changed = adjustICmpToBuiltin();
  73: 
  74:   for (Function &F : *M)
  75:     for (auto &BB : F) {
  76:       adjustBasicBlock(BB);
  77:       for (auto &I : BB)
  78:         adjustInst(I);
  79:     }
  80:   return insertPassThrough() || Changed;
```

- EN: Function bodies or method definitions such as run contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: run 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 81-100

```cpp
  81: }
  82: 
  83: // Commit acabad9ff6bf ("[InstCombine] try to canonicalize icmp with
  84: // trunc op into mask and cmp") added a transformation to
  85: // convert "(conv)a < power_2_const" to "a & <const>" in certain
  86: // cases and bpf kernel verifier has to handle the resulted code
  87: // conservatively and this may reject otherwise legitimate program.
  88: // Here, we change related icmp code to a builtin which will
  89: // be restored to original icmp code later to prevent that
  90: // InstCombine transformatin.
  91: bool BPFAdjustOptImpl::adjustICmpToBuiltin() {
  92:   bool Changed = false;
  93:   ICmpInst *ToBeDeleted = nullptr;
  94:   for (Function &F : *M)
  95:     for (auto &BB : F)
  96:       for (auto &I : BB) {
  97:         if (ToBeDeleted) {
  98:           ToBeDeleted->eraseFromParent();
  99:           ToBeDeleted = nullptr;
 100:         }
```

- EN: Function bodies or method definitions such as adjustICmpToBuiltin contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: adjustICmpToBuiltin 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 101-120

```cpp
 101: 
 102:         auto *Icmp = dyn_cast<ICmpInst>(&I);
 103:         if (!Icmp)
 104:           continue;
 105: 
 106:         Value *Op0 = Icmp->getOperand(0);
 107:         if (!isa<TruncInst>(Op0))
 108:           continue;
 109: 
 110:         auto ConstOp1 = dyn_cast<ConstantInt>(Icmp->getOperand(1));
 111:         if (!ConstOp1)
 112:           continue;
 113: 
 114:         auto ConstOp1Val = ConstOp1->getValue().getZExtValue();
 115:         auto Op = Icmp->getPredicate();
 116:         if (Op == ICmpInst::ICMP_ULT || Op == ICmpInst::ICMP_UGE) {
 117:           if ((ConstOp1Val - 1) & ConstOp1Val)
 118:             continue;
 119:         } else if (Op == ICmpInst::ICMP_ULE || Op == ICmpInst::ICMP_UGT) {
 120:           if (ConstOp1Val & (ConstOp1Val + 1))
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 121-140

```cpp
 121:             continue;
 122:         } else {
 123:           continue;
 124:         }
 125: 
 126:         Constant *Opcode =
 127:             ConstantInt::get(Type::getInt32Ty(BB.getContext()), Op);
 128:         Function *Fn = Intrinsic::getOrInsertDeclaration(
 129:             M, Intrinsic::bpf_compare, {Op0->getType(), ConstOp1->getType()});
 130:         auto *NewInst = CallInst::Create(Fn, {Opcode, Op0, ConstOp1});
 131:         NewInst->insertBefore(I.getIterator());
 132:         Icmp->replaceAllUsesWith(NewInst);
 133:         Changed = true;
 134:         ToBeDeleted = Icmp;
 135:       }
 136: 
 137:   return Changed;
 138: }
 139: 
 140: bool BPFAdjustOptImpl::insertPassThrough() {
```

- EN: Function bodies or method definitions such as insertPassThrough contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: insertPassThrough 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 141-160

```cpp
 141:   for (auto &Info : PassThroughs) {
 142:     auto *CI = BPFCoreSharedInfo::insertPassThrough(
 143:         M, Info.UsedInst->getParent(), Info.Input, Info.UsedInst);
 144:     Info.UsedInst->setOperand(Info.OpIdx, CI);
 145:   }
 146: 
 147:   return !PassThroughs.empty();
 148: }
 149: 
 150: // To avoid combining conditionals in the same basic block by
 151: // instrcombine optimization.
 152: bool BPFAdjustOptImpl::serializeICMPInBB(Instruction &I) {
 153:   // For:
 154:   //   comp1 = icmp <opcode> ...;
 155:   //   comp2 = icmp <opcode> ...;
 156:   //   ... or comp1 comp2 ...
 157:   // changed to:
 158:   //   comp1 = icmp <opcode> ...;
 159:   //   comp2 = icmp <opcode> ...;
 160:   //   new_comp1 = __builtin_bpf_passthrough(seq_num, comp1)
```

- EN: Function bodies or method definitions such as serializeICMPInBB contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: serializeICMPInBB 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 161-180

```cpp
 161:   //   ... or new_comp1 comp2 ...
 162:   Value *Op0, *Op1;
 163:   // Use LogicalOr (accept `or i1` as well as `select i1 Op0, true, Op1`)
 164:   if (!match(&I, m_LogicalOr(m_Value(Op0), m_Value(Op1))))
 165:     return false;
 166:   auto *Icmp1 = dyn_cast<ICmpInst>(Op0);
 167:   if (!Icmp1)
 168:     return false;
 169:   auto *Icmp2 = dyn_cast<ICmpInst>(Op1);
 170:   if (!Icmp2)
 171:     return false;
 172: 
 173:   Value *Icmp1Op0 = Icmp1->getOperand(0);
 174:   Value *Icmp2Op0 = Icmp2->getOperand(0);
 175:   if (Icmp1Op0 != Icmp2Op0)
 176:     return false;
 177: 
 178:   // Now we got two icmp instructions which feed into
 179:   // an "or" instruction.
 180:   PassThroughInfo Info(Icmp1, &I, 0);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 181-200

```cpp
 181:   PassThroughs.push_back(Info);
 182:   return true;
 183: }
 184: 
 185: // To avoid combining conditionals in the same basic block by
 186: // instrcombine optimization.
 187: bool BPFAdjustOptImpl::serializeICMPCrossBB(BasicBlock &BB) {
 188:   // For:
 189:   //   B1:
 190:   //     comp1 = icmp <opcode> ...;
 191:   //     if (comp1) goto B2 else B3;
 192:   //   B2:
 193:   //     comp2 = icmp <opcode> ...;
 194:   //     if (comp2) goto B4 else B5;
 195:   //   B4:
 196:   //     ...
 197:   // changed to:
 198:   //   B1:
 199:   //     comp1 = icmp <opcode> ...;
 200:   //     comp1 = __builtin_bpf_passthrough(seq_num, comp1);
```

- EN: Function bodies or method definitions such as serializeICMPCrossBB contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: serializeICMPCrossBB 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 201-220

```cpp
 201:   //     if (comp1) goto B2 else B3;
 202:   //   B2:
 203:   //     comp2 = icmp <opcode> ...;
 204:   //     if (comp2) goto B4 else B5;
 205:   //   B4:
 206:   //     ...
 207: 
 208:   // Check basic predecessors, if two of them (say B1, B2) are using
 209:   // icmp instructions to generate conditions and one is the predesessor
 210:   // of another (e.g., B1 is the predecessor of B2). Add a passthrough
 211:   // barrier after icmp inst of block B1.
 212:   BasicBlock *B2 = BB.getSinglePredecessor();
 213:   if (!B2)
 214:     return false;
 215: 
 216:   BasicBlock *B1 = B2->getSinglePredecessor();
 217:   if (!B1)
 218:     return false;
 219: 
 220:   Instruction *TI = B2->getTerminator();
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 221-240

```cpp
 221:   auto *BI = dyn_cast<CondBrInst>(TI);
 222:   if (!BI)
 223:     return false;
 224:   auto *Cond = dyn_cast<ICmpInst>(BI->getCondition());
 225:   if (!Cond || &*B2->getFirstNonPHIIt() != Cond)
 226:     return false;
 227:   Value *B2Op0 = Cond->getOperand(0);
 228:   auto Cond2Op = Cond->getPredicate();
 229: 
 230:   TI = B1->getTerminator();
 231:   BI = dyn_cast<CondBrInst>(TI);
 232:   if (!BI)
 233:     return false;
 234:   Cond = dyn_cast<ICmpInst>(BI->getCondition());
 235:   if (!Cond)
 236:     return false;
 237:   Value *B1Op0 = Cond->getOperand(0);
 238:   auto Cond1Op = Cond->getPredicate();
 239: 
 240:   if (B1Op0 != B2Op0)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 241-260

```cpp
 241:     return false;
 242: 
 243:   if (Cond1Op == ICmpInst::ICMP_SGT || Cond1Op == ICmpInst::ICMP_SGE) {
 244:     if (Cond2Op != ICmpInst::ICMP_SLT && Cond2Op != ICmpInst::ICMP_SLE)
 245:       return false;
 246:   } else if (Cond1Op == ICmpInst::ICMP_SLT || Cond1Op == ICmpInst::ICMP_SLE) {
 247:     if (Cond2Op != ICmpInst::ICMP_SGT && Cond2Op != ICmpInst::ICMP_SGE)
 248:       return false;
 249:   } else if (Cond1Op == ICmpInst::ICMP_ULT || Cond1Op == ICmpInst::ICMP_ULE) {
 250:     if (Cond2Op != ICmpInst::ICMP_UGT && Cond2Op != ICmpInst::ICMP_UGE)
 251:       return false;
 252:   } else if (Cond1Op == ICmpInst::ICMP_UGT || Cond1Op == ICmpInst::ICMP_UGE) {
 253:     if (Cond2Op != ICmpInst::ICMP_ULT && Cond2Op != ICmpInst::ICMP_ULE)
 254:       return false;
 255:   } else {
 256:     return false;
 257:   }
 258: 
 259:   PassThroughInfo Info(Cond, BI, 0);
 260:   PassThroughs.push_back(Info);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 261-280

```cpp
 261: 
 262:   return true;
 263: }
 264: 
 265: // To avoid speculative hoisting certain computations out of
 266: // a basic block.
 267: bool BPFAdjustOptImpl::avoidSpeculation(Instruction &I) {
 268:   if (auto *LdInst = dyn_cast<LoadInst>(&I)) {
 269:     if (auto *GV = dyn_cast<GlobalVariable>(LdInst->getOperand(0))) {
 270:       if (GV->hasAttribute(BPFCoreSharedInfo::AmaAttr) ||
 271:           GV->hasAttribute(BPFCoreSharedInfo::TypeIdAttr))
 272:         return false;
 273:     }
 274:   }
 275: 
 276:   if (!isa<LoadInst>(&I) && !isa<CallInst>(&I))
 277:     return false;
 278: 
 279:   // For:
 280:   //   B1:
```

- EN: Function bodies or method definitions such as avoidSpeculation contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: avoidSpeculation 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 281-300

```cpp
 281:   //     var = ...
 282:   //     ...
 283:   //     /* icmp may not be in the same block as var = ... */
 284:   //     comp1 = icmp <opcode> var, <const>;
 285:   //     if (comp1) goto B2 else B3;
 286:   //   B2:
 287:   //     ... var ...
 288:   // change to:
 289:   //   B1:
 290:   //     var = ...
 291:   //     ...
 292:   //     /* icmp may not be in the same block as var = ... */
 293:   //     comp1 = icmp <opcode> var, <const>;
 294:   //     if (comp1) goto B2 else B3;
 295:   //   B2:
 296:   //     var = __builtin_bpf_passthrough(seq_num, var);
 297:   //     ... var ...
 298:   bool isCandidate = false;
 299:   SmallVector<PassThroughInfo, 4> Candidates;
 300:   for (User *U : I.users()) {
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 301-320

```cpp
 301:     Instruction *Inst = dyn_cast<Instruction>(U);
 302:     if (!Inst)
 303:       continue;
 304: 
 305:     // May cover a little bit more than the
 306:     // above pattern.
 307:     if (auto *Icmp1 = dyn_cast<ICmpInst>(Inst)) {
 308:       Value *Icmp1Op1 = Icmp1->getOperand(1);
 309:       if (!isa<Constant>(Icmp1Op1))
 310:         return false;
 311:       isCandidate = true;
 312:       continue;
 313:     }
 314: 
 315:     // Ignore the use in the same basic block as the definition.
 316:     if (Inst->getParent() == I.getParent())
 317:       continue;
 318: 
 319:     // use in a different basic block, If there is a call or
 320:     // load/store insn before this instruction in this basic
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 321-340

```cpp
 321:     // block. Most likely it cannot be hoisted out. Skip it.
 322:     for (auto &I2 : *Inst->getParent()) {
 323:       if (isa<CallInst>(&I2))
 324:         return false;
 325:       if (isa<LoadInst>(&I2) || isa<StoreInst>(&I2))
 326:         return false;
 327:       if (&I2 == Inst)
 328:         break;
 329:     }
 330: 
 331:     // It should be used in a GEP or a simple arithmetic like
 332:     // ZEXT/SEXT which is used for GEP.
 333:     if (Inst->getOpcode() == Instruction::ZExt ||
 334:         Inst->getOpcode() == Instruction::SExt) {
 335:       PassThroughInfo Info(&I, Inst, 0);
 336:       Candidates.push_back(Info);
 337:     } else if (auto *GI = dyn_cast<GetElementPtrInst>(Inst)) {
 338:       // traverse GEP inst to find Use operand index
 339:       unsigned i, e;
 340:       for (i = 1, e = GI->getNumOperands(); i != e; ++i) {
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 341-360

```cpp
 341:         Value *V = GI->getOperand(i);
 342:         if (V == &I)
 343:           break;
 344:       }
 345:       if (i == e)
 346:         continue;
 347: 
 348:       PassThroughInfo Info(&I, GI, i);
 349:       Candidates.push_back(Info);
 350:     }
 351:   }
 352: 
 353:   if (!isCandidate || Candidates.empty())
 354:     return false;
 355: 
 356:   llvm::append_range(PassThroughs, Candidates);
 357:   return true;
 358: }
 359: 
 360: void BPFAdjustOptImpl::adjustBasicBlock(BasicBlock &BB) {
```

- EN: Function bodies or method definitions such as adjustBasicBlock contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: adjustBasicBlock 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 361-375

```cpp
 361:   if (!DisableBPFserializeICMP && serializeICMPCrossBB(BB))
 362:     return;
 363: }
 364: 
 365: void BPFAdjustOptImpl::adjustInst(Instruction &I) {
 366:   if (!DisableBPFserializeICMP && serializeICMPInBB(I))
 367:     return;
 368:   if (!DisableBPFavoidSpeculation && avoidSpeculation(I))
 369:     return;
 370: }
 371: 
 372: PreservedAnalyses BPFAdjustOptPass::run(Module &M, ModuleAnalysisManager &AM) {
 373:   return BPFAdjustOptImpl(&M).run() ? PreservedAnalyses::none()
 374:                                     : PreservedAnalyses::all();
 375: }
```

- EN: Function bodies or method definitions such as adjustInst, run contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: adjustInst, run 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

## Key Concepts / 关键概念

- Target backend support code / 目标后端支持代码
- LLVM CodeGen integration / LLVM CodeGen 集成

## Dependencies / 依赖关系

- Direct includes / 直接包含: `BPF.h`, `BPFCORE.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicsBPF.h`, `llvm/IR/Module.h`, `llvm/IR/PatternMatch.h`, `llvm/IR/Type.h`
