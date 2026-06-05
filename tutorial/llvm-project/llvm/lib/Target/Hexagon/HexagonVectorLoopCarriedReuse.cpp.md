# HexagonVectorLoopCarriedReuse.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonVectorLoopCarriedReuse.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): See info about DepChain in the comments at the top of this file.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===- HexagonVectorLoopCarriedReuse.cpp ----------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This pass removes the computation of provably redundant expressions that have
    10: // been computed earlier in a previous iteration. It relies on the use of PHIs
    11: // to identify loop carried dependences. This is scalar replacement for vector
    12: // types.
    13: //
    14: //===----------------------------------------------------------------------===//
    15: 
    16: #include "HexagonVectorLoopCarriedReuse.h"
    17: #include "Hexagon.h"
    18: #include "llvm/ADT/SetVector.h"
    19: #include "llvm/ADT/SmallVector.h"
    20: #include "llvm/ADT/Statistic.h"
    21: #include "llvm/Analysis/LoopInfo.h"
    22: #include "llvm/Analysis/LoopPass.h"
    23: #include "llvm/IR/BasicBlock.h"
    24: #include "llvm/IR/DerivedTypes.h"
    25: #include "llvm/IR/IRBuilder.h"
    26: #include "llvm/IR/Instruction.h"
    27: #include "llvm/IR/Instructions.h"
    28: #include "llvm/IR/IntrinsicInst.h"
    29: #include "llvm/IR/Intrinsics.h"
    30: #include "llvm/IR/IntrinsicsHexagon.h"
    31: #include "llvm/IR/Use.h"
    32: #include "llvm/IR/Value.h"
    33: #include "llvm/InitializePasses.h"
    34: #include "llvm/Pass.h"
    35: #include "llvm/Support/Casting.h"
    36: #include "llvm/Support/CommandLine.h"
    37: #include "llvm/Support/Compiler.h"
    38: #include "llvm/Support/Debug.h"
    39: #include "llvm/Support/raw_ostream.h"
    40: #include "llvm/Transforms/Scalar.h"
    41: #include "llvm/Transforms/Utils.h"
    42: #include <cassert>
    43: #include <map>
    44: #include <memory>
    45: #include <set>
    46: 
    47: using namespace llvm;
    48: 
    49: #define DEBUG_TYPE "hexagon-vlcr"
    50: 
```
- EN: It imports headers such as HexagonVectorLoopCarriedReuse.h, Hexagon.h, llvm/ADT/SetVector.h, llvm/ADT/SmallVector.h, ... (30 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里引入了 HexagonVectorLoopCarriedReuse.h, Hexagon.h, llvm/ADT/SetVector.h, llvm/ADT/SmallVector.h, ... (30 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 51-100 / 第 51-100 行

```cpp
    51: STATISTIC(HexagonNumVectorLoopCarriedReuse,
    52:           "Number of values that were reused from a previous iteration.");
    53: 
    54: static cl::opt<int> HexagonVLCRIterationLim(
    55:     "hexagon-vlcr-iteration-lim", cl::Hidden,
    56:     cl::desc("Maximum distance of loop carried dependences that are handled"),
    57:     cl::init(2));
    58: 
    59: namespace {
    60: 
    61:   // See info about DepChain in the comments at the top of this file.
    62:   using ChainOfDependences = SmallVector<Instruction *, 4>;
    63: 
    64:   class DepChain {
    65:     ChainOfDependences Chain;
    66: 
    67:   public:
    68:     bool isIdentical(DepChain &Other) const {
    69:       if (Other.size() != size())
    70:         return false;
    71:       ChainOfDependences &OtherChain = Other.getChain();
    72:       for (int i = 0; i < size(); ++i) {
    73:         if (Chain[i] != OtherChain[i])
    74:           return false;
    75:       }
    76:       return true;
    77:     }
    78: 
    79:     ChainOfDependences &getChain() {
    80:       return Chain;
    81:     }
    82: 
    83:     int size() const {
    84:       return Chain.size();
    85:     }
    86: 
    87:     void clear() {
    88:       Chain.clear();
    89:     }
    90: 
    91:     void push_back(Instruction *I) {
    92:       Chain.push_back(I);
    93:     }
    94: 
    95:     int iterations() const {
    96:       return size() - 1;
    97:     }
    98: 
    99:     Instruction *front() const {
   100:       return Chain.front();
```
- EN: It declares types such as DepChain, which carry the state or API of this component. It defines declarative TableGen records like DepChain; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as STATISTIC, HexagonVLCRIterationLim, isIdentical, getChain, ... (9 total), translating Hexagon-specific policy into reusable code paths. Command-line options appear here to gate diagnostics or target-specific tuning behavior.
- CN: 这里声明了 DepChain 等类型，用来承载该组件的状态或接口。 这里定义了 DepChain 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 STATISTIC, HexagonVLCRIterationLim, isIdentical, getChain, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里出现了命令行选项，用于控制诊断开关或目标相关的调优行为。

### Lines 101-150 / 第 101-150 行

```cpp
   101:     }
   102: 
   103:     Instruction *back() const {
   104:       return Chain.back();
   105:     }
   106: 
   107:     Instruction *&operator[](const int index) {
   108:       return Chain[index];
   109:     }
   110: 
   111:    friend raw_ostream &operator<< (raw_ostream &OS, const DepChain &D);
   112:   };
   113: 
   114:   [[maybe_unused]]
   115:   raw_ostream &operator<<(raw_ostream &OS, const DepChain &D) {
   116:     const ChainOfDependences &CD = D.Chain;
   117:     int ChainSize = CD.size();
   118:     OS << "**DepChain Start::**\n";
   119:     for (int i = 0; i < ChainSize -1; ++i) {
   120:       OS << *(CD[i]) << " -->\n";
   121:     }
   122:     OS << *CD[ChainSize-1] << "\n";
   123:     return OS;
   124:   }
   125: 
   126:   struct ReuseValue {
   127:     Instruction *Inst2Replace = nullptr;
   128: 
   129:     // In the new PHI node that we'll construct this is the value that'll be
   130:     // used over the backedge. This is the value that gets reused from a
   131:     // previous iteration.
   132:     Instruction *BackedgeInst = nullptr;
   133:     std::map<Instruction *, DepChain *> DepChains;
   134:     int Iterations = -1;
   135: 
   136:     ReuseValue() = default;
   137: 
   138:     void reset() {
   139:       Inst2Replace = nullptr;
   140:       BackedgeInst = nullptr;
   141:       DepChains.clear();
   142:       Iterations = -1;
   143:     }
   144:     bool isDefined() { return Inst2Replace != nullptr; }
   145:   };
   146: 
   147:   [[maybe_unused]]
   148:   raw_ostream &operator<<(raw_ostream &OS, const ReuseValue &RU) {
   149:     OS << "** ReuseValue ***\n";
   150:     OS << "Instruction to Replace: " << *(RU.Inst2Replace) << "\n";
```
- EN: It declares types such as ReuseValue, which carry the state or API of this component. It declares or implements routines such as back, size, ReuseValue, reset, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 ReuseValue 等类型，用来承载该组件的状态或接口。 这里声明或实现了 back, size, ReuseValue, reset, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 151-200 / 第 151-200 行

```cpp
   151:     OS << "Backedge Instruction: " << *(RU.BackedgeInst) << "\n";
   152:     return OS;
   153:   }
   154: 
   155:   class HexagonVectorLoopCarriedReuseLegacyPass : public LoopPass {
   156:   public:
   157:     static char ID;
   158: 
   159:     explicit HexagonVectorLoopCarriedReuseLegacyPass() : LoopPass(ID) {}
   160: 
   161:     StringRef getPassName() const override {
   162:       return "Hexagon-specific loop carried reuse for HVX vectors";
   163:     }
   164: 
   165:     void getAnalysisUsage(AnalysisUsage &AU) const override {
   166:       AU.addRequiredID(LoopSimplifyID);
   167:       AU.addRequiredID(LCSSAID);
   168:       AU.addPreservedID(LCSSAID);
   169:       AU.setPreservesCFG();
   170:     }
   171: 
   172:     bool runOnLoop(Loop *L, LPPassManager &LPM) override;
   173:   };
   174: 
   175:   class HexagonVectorLoopCarriedReuse {
   176:   public:
   177:     HexagonVectorLoopCarriedReuse(Loop *L) : CurLoop(L){};
   178: 
   179:     bool run();
   180: 
   181:   private:
   182:     SetVector<DepChain *> Dependences;
   183:     std::set<Instruction *> ReplacedInsts;
   184:     Loop *CurLoop;
   185:     ReuseValue ReuseCandidate;
   186: 
   187:     bool doVLCR();
   188:     void findLoopCarriedDeps();
   189:     void findValueToReuse();
   190:     void findDepChainFromPHI(Instruction *I, DepChain &D);
   191:     void reuseValue();
   192:     Value *findValueInBlock(Value *Op, BasicBlock *BB);
   193:     DepChain *getDepChainBtwn(Instruction *I1, Instruction *I2, int Iters);
   194:     bool isEquivalentOperation(Instruction *I1, Instruction *I2);
   195:     bool canReplace(Instruction *I);
   196:     bool isCallInstCommutative(CallInst *C);
   197:   };
   198: 
   199: } // end anonymous namespace
   200: 
```
- EN: It declares types such as HexagonVectorLoopCarriedReuseLegacyPass, HexagonVectorLoopCarriedReuse, which carry the state or API of this component. It defines declarative TableGen records like HexagonVectorLoopCarriedReuseLegacyPass, HexagonVectorLoopCarriedReuse; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as HexagonVectorLoopCarriedReuseLegacyPass, getPassName, getAnalysisUsage, addRequiredID, ... (19 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 HexagonVectorLoopCarriedReuseLegacyPass, HexagonVectorLoopCarriedReuse 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonVectorLoopCarriedReuseLegacyPass, HexagonVectorLoopCarriedReuse 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 HexagonVectorLoopCarriedReuseLegacyPass, getPassName, getAnalysisUsage, addRequiredID, ... (19 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 201-250 / 第 201-250 行

```cpp
   201: char HexagonVectorLoopCarriedReuseLegacyPass::ID = 0;
   202: 
   203: INITIALIZE_PASS_BEGIN(HexagonVectorLoopCarriedReuseLegacyPass, "hexagon-vlcr",
   204:                       "Hexagon-specific predictive commoning for HVX vectors",
   205:                       false, false)
   206: INITIALIZE_PASS_DEPENDENCY(LoopSimplify)
   207: INITIALIZE_PASS_DEPENDENCY(LCSSAWrapperPass)
   208: INITIALIZE_PASS_END(HexagonVectorLoopCarriedReuseLegacyPass, "hexagon-vlcr",
   209:                     "Hexagon-specific predictive commoning for HVX vectors",
   210:                     false, false)
   211: 
   212: PreservedAnalyses
   213: HexagonVectorLoopCarriedReusePass::run(Loop &L, LoopAnalysisManager &LAM,
   214:                                        LoopStandardAnalysisResults &AR,
   215:                                        LPMUpdater &U) {
   216:   HexagonVectorLoopCarriedReuse Vlcr(&L);
   217:   if (!Vlcr.run())
   218:     return PreservedAnalyses::all();
   219:   PreservedAnalyses PA;
   220:   PA.preserveSet<CFGAnalyses>();
   221:   return PA;
   222: }
   223: 
   224: bool HexagonVectorLoopCarriedReuseLegacyPass::runOnLoop(Loop *L,
   225:                                                         LPPassManager &LPM) {
   226:   if (skipLoop(L))
   227:     return false;
   228:   HexagonVectorLoopCarriedReuse Vlcr(L);
   229:   return Vlcr.run();
   230: }
   231: 
   232: bool HexagonVectorLoopCarriedReuse::run() {
   233:   if (!CurLoop->getLoopPreheader())
   234:     return false;
   235: 
   236:   // Work only on innermost loops.
   237:   if (!CurLoop->getSubLoops().empty())
   238:     return false;
   239: 
   240:   // Work only on single basic blocks loops.
   241:   if (CurLoop->getNumBlocks() != 1)
   242:     return false;
   243: 
   244:   return doVLCR();
   245: }
   246: 
   247: bool HexagonVectorLoopCarriedReuse::isCallInstCommutative(CallInst *C) {
   248:   switch (C->getCalledFunction()->getIntrinsicID()) {
   249:     case Intrinsic::hexagon_V6_vaddb:
   250:     case Intrinsic::hexagon_V6_vaddb_128B:
```
- EN: It declares or implements routines such as INITIALIZE_PASS_BEGIN, Vlcr, preserveSet<CFGAnalyses>, HexagonVectorLoopCarriedReuseLegacyPass::runOnLoop, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonVectorLoopCarriedReuseLegacyPass, HexagonVectorLoopCarriedReusePass, HexagonVectorLoopCarriedReuse, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 INITIALIZE_PASS_BEGIN, Vlcr, preserveSet<CFGAnalyses>, HexagonVectorLoopCarriedReuseLegacyPass::runOnLoop, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonVectorLoopCarriedReuseLegacyPass, HexagonVectorLoopCarriedReusePass, HexagonVectorLoopCarriedReuse，说明了它与同级后端组件的连接关系。

### Lines 251-300 / 第 251-300 行

```cpp
   251:     case Intrinsic::hexagon_V6_vaddh:
   252:     case Intrinsic::hexagon_V6_vaddh_128B:
   253:     case Intrinsic::hexagon_V6_vaddw:
   254:     case Intrinsic::hexagon_V6_vaddw_128B:
   255:     case Intrinsic::hexagon_V6_vaddubh:
   256:     case Intrinsic::hexagon_V6_vaddubh_128B:
   257:     case Intrinsic::hexagon_V6_vadduhw:
   258:     case Intrinsic::hexagon_V6_vadduhw_128B:
   259:     case Intrinsic::hexagon_V6_vaddhw:
   260:     case Intrinsic::hexagon_V6_vaddhw_128B:
   261:     case Intrinsic::hexagon_V6_vmaxb:
   262:     case Intrinsic::hexagon_V6_vmaxb_128B:
   263:     case Intrinsic::hexagon_V6_vmaxh:
   264:     case Intrinsic::hexagon_V6_vmaxh_128B:
   265:     case Intrinsic::hexagon_V6_vmaxw:
   266:     case Intrinsic::hexagon_V6_vmaxw_128B:
   267:     case Intrinsic::hexagon_V6_vmaxub:
   268:     case Intrinsic::hexagon_V6_vmaxub_128B:
   269:     case Intrinsic::hexagon_V6_vmaxuh:
   270:     case Intrinsic::hexagon_V6_vmaxuh_128B:
   271:     case Intrinsic::hexagon_V6_vminub:
   272:     case Intrinsic::hexagon_V6_vminub_128B:
   273:     case Intrinsic::hexagon_V6_vminuh:
   274:     case Intrinsic::hexagon_V6_vminuh_128B:
   275:     case Intrinsic::hexagon_V6_vminb:
   276:     case Intrinsic::hexagon_V6_vminb_128B:
   277:     case Intrinsic::hexagon_V6_vminh:
   278:     case Intrinsic::hexagon_V6_vminh_128B:
   279:     case Intrinsic::hexagon_V6_vminw:
   280:     case Intrinsic::hexagon_V6_vminw_128B:
   281:     case Intrinsic::hexagon_V6_vmpyub:
   282:     case Intrinsic::hexagon_V6_vmpyub_128B:
   283:     case Intrinsic::hexagon_V6_vmpyuh:
   284:     case Intrinsic::hexagon_V6_vmpyuh_128B:
   285:     case Intrinsic::hexagon_V6_vavgub:
   286:     case Intrinsic::hexagon_V6_vavgub_128B:
   287:     case Intrinsic::hexagon_V6_vavgh:
   288:     case Intrinsic::hexagon_V6_vavgh_128B:
   289:     case Intrinsic::hexagon_V6_vavguh:
   290:     case Intrinsic::hexagon_V6_vavguh_128B:
   291:     case Intrinsic::hexagon_V6_vavgw:
   292:     case Intrinsic::hexagon_V6_vavgw_128B:
   293:     case Intrinsic::hexagon_V6_vavgb:
   294:     case Intrinsic::hexagon_V6_vavgb_128B:
   295:     case Intrinsic::hexagon_V6_vavguw:
   296:     case Intrinsic::hexagon_V6_vavguw_128B:
   297:     case Intrinsic::hexagon_V6_vabsdiffh:
   298:     case Intrinsic::hexagon_V6_vabsdiffh_128B:
   299:     case Intrinsic::hexagon_V6_vabsdiffub:
   300:     case Intrinsic::hexagon_V6_vabsdiffub_128B:
```
- EN: Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 301-350 / 第 301-350 行

```cpp
   301:     case Intrinsic::hexagon_V6_vabsdiffuh:
   302:     case Intrinsic::hexagon_V6_vabsdiffuh_128B:
   303:     case Intrinsic::hexagon_V6_vabsdiffw:
   304:     case Intrinsic::hexagon_V6_vabsdiffw_128B:
   305:       return true;
   306:     default:
   307:       return false;
   308:   }
   309: }
   310: 
   311: bool HexagonVectorLoopCarriedReuse::isEquivalentOperation(Instruction *I1,
   312:                                                           Instruction *I2) {
   313:   if (!I1->isSameOperationAs(I2))
   314:     return false;
   315:   // This check is in place specifically for intrinsics. isSameOperationAs will
   316:   // return two for any two hexagon intrinsics because they are essentially the
   317:   // same instruction (CallInst). We need to scratch the surface to see if they
   318:   // are calls to the same function.
   319:   if (CallInst *C1 = dyn_cast<CallInst>(I1)) {
   320:     if (CallInst *C2 = dyn_cast<CallInst>(I2)) {
   321:       if (C1->getCalledFunction() != C2->getCalledFunction())
   322:         return false;
   323:     }
   324:   }
   325: 
   326:   // If both the Instructions are of Vector Type and any of the element
   327:   // is integer constant, check their values too for equivalence.
   328:   if (I1->getType()->isVectorTy() && I2->getType()->isVectorTy()) {
   329:     unsigned NumOperands = I1->getNumOperands();
   330:     for (unsigned i = 0; i < NumOperands; ++i) {
   331:       ConstantInt *C1 = dyn_cast<ConstantInt>(I1->getOperand(i));
   332:       ConstantInt *C2 = dyn_cast<ConstantInt>(I2->getOperand(i));
   333:       if(!C1) continue;
   334:       assert(C2);
   335:       if (C1->getSExtValue() != C2->getSExtValue())
   336:         return false;
   337:     }
   338:   }
   339: 
   340:   return true;
   341: }
   342: 
   343: bool HexagonVectorLoopCarriedReuse::canReplace(Instruction *I) {
   344:   const IntrinsicInst *II = dyn_cast<IntrinsicInst>(I);
   345:   if (!II)
   346:     return true;
   347: 
   348:   switch (II->getIntrinsicID()) {
   349:   case Intrinsic::hexagon_V6_hi:
   350:   case Intrinsic::hexagon_V6_lo:
```
- EN: It declares or implements routines such as HexagonVectorLoopCarriedReuse::isEquivalentOperation, instruction, getNumOperands, dyn_cast<ConstantInt>, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonVectorLoopCarriedReuse, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonVectorLoopCarriedReuse::isEquivalentOperation, instruction, getNumOperands, dyn_cast<ConstantInt>, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonVectorLoopCarriedReuse，说明了它与同级后端组件的连接关系。

### Lines 351-400 / 第 351-400 行

```cpp
   351:   case Intrinsic::hexagon_V6_hi_128B:
   352:   case Intrinsic::hexagon_V6_lo_128B:
   353:     LLVM_DEBUG(dbgs() << "Not considering for reuse: " << *II << "\n");
   354:     return false;
   355:   default:
   356:     return true;
   357:   }
   358: }
   359: void HexagonVectorLoopCarriedReuse::findValueToReuse() {
   360:   for (auto *D : Dependences) {
   361:     LLVM_DEBUG(dbgs() << "Processing dependence " << *(D->front()) << "\n");
   362:     if (D->iterations() > HexagonVLCRIterationLim) {
   363:       LLVM_DEBUG(
   364:           dbgs()
   365:           << ".. Skipping because number of iterations > than the limit\n");
   366:       continue;
   367:     }
   368: 
   369:     PHINode *PN = cast<PHINode>(D->front());
   370:     Instruction *BEInst = D->back();
   371:     int Iters = D->iterations();
   372:     BasicBlock *BB = PN->getParent();
   373:     LLVM_DEBUG(dbgs() << "Checking if any uses of " << *PN
   374:                       << " can be reused\n");
   375: 
   376:     SmallVector<Instruction *, 4> PNUsers;
   377:     for (Use &U : PN->uses()) {
   378:       Instruction *User = cast<Instruction>(U.getUser());
   379: 
   380:       if (User->getParent() != BB)
   381:         continue;
   382:       if (ReplacedInsts.count(User)) {
   383:         LLVM_DEBUG(dbgs() << *User
   384:                           << " has already been replaced. Skipping...\n");
   385:         continue;
   386:       }
   387:       if (isa<PHINode>(User))
   388:         continue;
   389:       if (User->mayHaveSideEffects())
   390:         continue;
   391:       if (!canReplace(User))
   392:         continue;
   393: 
   394:       PNUsers.push_back(User);
   395:     }
   396:     LLVM_DEBUG(dbgs() << PNUsers.size() << " use(s) of the PHI in the block\n");
   397: 
   398:     // For each interesting use I of PN, find an Instruction BEUser that
   399:     // performs the same operation as I on BEInst and whose other operands,
   400:     // if any, can also be rematerialized in OtherBB. We stop when we find the
```
- EN: It declares or implements routines such as LLVM_DEBUG, HexagonVectorLoopCarriedReuse::findValueToReuse, cast<PHINode>, back, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonVectorLoopCarriedReuse, HexagonVLCRIterationLim, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 LLVM_DEBUG, HexagonVectorLoopCarriedReuse::findValueToReuse, cast<PHINode>, back, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonVectorLoopCarriedReuse, HexagonVLCRIterationLim，说明了它与同级后端组件的连接关系。

### Lines 401-450 / 第 401-450 行

```cpp
   401:     // first such Instruction BEUser. This is because once BEUser is
   402:     // rematerialized in OtherBB, we may find more such "fixup" opportunities
   403:     // in this block. So, we'll start over again.
   404:     for (Instruction *I : PNUsers) {
   405:       for (Use &U : BEInst->uses()) {
   406:         Instruction *BEUser = cast<Instruction>(U.getUser());
   407: 
   408:         if (BEUser->getParent() != BB)
   409:           continue;
   410:         if (!isEquivalentOperation(I, BEUser))
   411:           continue;
   412: 
   413:         int NumOperands = I->getNumOperands();
   414: 
   415:         // Take operands of each PNUser one by one and try to find DepChain
   416:         // with every operand of the BEUser. If any of the operands of BEUser
   417:         // has DepChain with current operand of the PNUser, break the matcher
   418:         // loop. Keep doing this for Every PNUser operand. If PNUser operand
   419:         // does not have DepChain with any of the BEUser operand, break the
   420:         // outer matcher loop, mark the BEUser as null and reset the ReuseCandidate.
   421:         // This ensures that DepChain exist for all the PNUser operand with
   422:         // BEUser operand. This also ensures that DepChains are independent of
   423:         // the positions in PNUser and BEUser.
   424:         std::map<Instruction *, DepChain *> DepChains;
   425:         CallInst *C1 = dyn_cast<CallInst>(I);
   426:         if ((I && I->isCommutative()) || (C1 && isCallInstCommutative(C1))) {
   427:           bool Found = false;
   428:           for (int OpNo = 0; OpNo < NumOperands; ++OpNo) {
   429:             Value *Op = I->getOperand(OpNo);
   430:             Instruction *OpInst = dyn_cast<Instruction>(Op);
   431:             Found = false;
   432:             for (int T = 0; T < NumOperands; ++T) {
   433:               Value *BEOp = BEUser->getOperand(T);
   434:               Instruction *BEOpInst = dyn_cast<Instruction>(BEOp);
   435:               if (!OpInst && !BEOpInst) {
   436:                 if (Op == BEOp) {
   437:                   Found = true;
   438:                   break;
   439:                 }
   440:               }
   441: 
   442:               if ((OpInst && !BEOpInst) || (!OpInst && BEOpInst))
   443:                 continue;
   444: 
   445:               DepChain *D = getDepChainBtwn(OpInst, BEOpInst, Iters);
   446: 
   447:               if (D) {
   448:                 Found = true;
   449:                 DepChains[OpInst] = D;
   450:                 break;
```
- EN: It declares or implements routines such as cast<Instruction>, getNumOperands, dyn_cast<CallInst>, getOperand, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 cast<Instruction>, getNumOperands, dyn_cast<CallInst>, getOperand, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 451-500 / 第 451-500 行

```cpp
   451:               }
   452:             }
   453:             if (!Found) {
   454:               BEUser = nullptr;
   455:               break;
   456:             }
   457:           }
   458:         } else {
   459: 
   460:           for (int OpNo = 0; OpNo < NumOperands; ++OpNo) {
   461:             Value *Op = I->getOperand(OpNo);
   462:             Value *BEOp = BEUser->getOperand(OpNo);
   463: 
   464:             Instruction *OpInst = dyn_cast<Instruction>(Op);
   465:             if (!OpInst) {
   466:               if (Op == BEOp)
   467:                 continue;
   468:               // Do not allow reuse to occur when the operands may be different
   469:               // values.
   470:               BEUser = nullptr;
   471:               break;
   472:             }
   473: 
   474:             Instruction *BEOpInst = dyn_cast<Instruction>(BEOp);
   475:             DepChain *D = getDepChainBtwn(OpInst, BEOpInst, Iters);
   476: 
   477:             if (D) {
   478:               DepChains[OpInst] = D;
   479:             } else {
   480:               BEUser = nullptr;
   481:               break;
   482:             }
   483:           }
   484:         }
   485:         if (BEUser) {
   486:           LLVM_DEBUG(dbgs() << "Found Value for reuse.\n");
   487:           ReuseCandidate.Inst2Replace = I;
   488:           ReuseCandidate.BackedgeInst = BEUser;
   489:           ReuseCandidate.DepChains = std::move(DepChains);
   490:           ReuseCandidate.Iterations = Iters;
   491:           return;
   492:         }
   493:         ReuseCandidate.reset();
   494:       }
   495:     }
   496:   }
   497:   ReuseCandidate.reset();
   498: }
   499: 
   500: Value *HexagonVectorLoopCarriedReuse::findValueInBlock(Value *Op,
```
- EN: It declares or implements routines such as getOperand, dyn_cast<Instruction>, getDepChainBtwn, LLVM_DEBUG, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonVectorLoopCarriedReuse, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getOperand, dyn_cast<Instruction>, getDepChainBtwn, LLVM_DEBUG, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonVectorLoopCarriedReuse，说明了它与同级后端组件的连接关系。

### Lines 501-550 / 第 501-550 行

```cpp
   501:                                                        BasicBlock *BB) {
   502:   PHINode *PN = dyn_cast<PHINode>(Op);
   503:   assert(PN);
   504:   Value *ValueInBlock = PN->getIncomingValueForBlock(BB);
   505:   return ValueInBlock;
   506: }
   507: 
   508: void HexagonVectorLoopCarriedReuse::reuseValue() {
   509:   LLVM_DEBUG(dbgs() << ReuseCandidate);
   510:   Instruction *Inst2Replace = ReuseCandidate.Inst2Replace;
   511:   Instruction *BEInst = ReuseCandidate.BackedgeInst;
   512:   int NumOperands = Inst2Replace->getNumOperands();
   513:   std::map<Instruction *, DepChain *> &DepChains = ReuseCandidate.DepChains;
   514:   int Iterations = ReuseCandidate.Iterations;
   515:   BasicBlock *LoopPH = CurLoop->getLoopPreheader();
   516:   assert(!DepChains.empty() && "No DepChains");
   517:   LLVM_DEBUG(dbgs() << "reuseValue is making the following changes\n");
   518: 
   519:   SmallVector<Instruction *, 4> InstsInPreheader;
   520:   for (int i = 0; i < Iterations; ++i) {
   521:     Instruction *InstInPreheader = Inst2Replace->clone();
   522:     for (int j = 0; j < NumOperands; ++j) {
   523:       Instruction *I = dyn_cast<Instruction>(Inst2Replace->getOperand(j));
   524:       if (!I)
   525:         continue;
   526:       // Get the DepChain corresponding to this operand.
   527:       DepChain &D = *DepChains[I];
   528:       // Get the PHI for the iteration number and find
   529:       // the incoming value from the Loop Preheader for
   530:       // that PHI.
   531:       Value *ValInPreheader = findValueInBlock(D[i], LoopPH);
   532:       InstInPreheader->setOperand(j, ValInPreheader);
   533:     }
   534:     InstsInPreheader.push_back(InstInPreheader);
   535:     InstInPreheader->setName(Inst2Replace->getName() + ".hexagon.vlcr");
   536:     InstInPreheader->insertBefore(LoopPH->getTerminator()->getIterator());
   537:     LLVM_DEBUG(dbgs() << "Added " << *InstInPreheader << " to "
   538:                       << LoopPH->getName() << "\n");
   539:   }
   540:   BasicBlock *BB = BEInst->getParent();
   541:   IRBuilder<> IRB(BB);
   542:   IRB.SetInsertPoint(BB, BB->getFirstNonPHIIt());
   543:   Value *BEVal = BEInst;
   544:   PHINode *NewPhi;
   545:   for (int i = Iterations-1; i >=0 ; --i) {
   546:     Instruction *InstInPreheader = InstsInPreheader[i];
   547:     NewPhi = IRB.CreatePHI(InstInPreheader->getType(), 2);
   548:     NewPhi->addIncoming(InstInPreheader, LoopPH);
   549:     NewPhi->addIncoming(BEVal, BB);
   550:     LLVM_DEBUG(dbgs() << "Adding " << *NewPhi << " to " << BB->getName()
```
- EN: It declares or implements routines such as dyn_cast<PHINode>, assert, getIncomingValueForBlock, HexagonVectorLoopCarriedReuse::reuseValue, ... (19 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonVectorLoopCarriedReuse, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 dyn_cast<PHINode>, assert, getIncomingValueForBlock, HexagonVectorLoopCarriedReuse::reuseValue, ... (19 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonVectorLoopCarriedReuse，说明了它与同级后端组件的连接关系。

### Lines 551-600 / 第 551-600 行

```cpp
   551:                       << "\n");
   552:     BEVal = NewPhi;
   553:   }
   554:   // We are in LCSSA form. So, a value defined inside the Loop is used only
   555:   // inside the loop. So, the following is safe.
   556:   Inst2Replace->replaceAllUsesWith(NewPhi);
   557:   ReplacedInsts.insert(Inst2Replace);
   558:   ++HexagonNumVectorLoopCarriedReuse;
   559: }
   560: 
   561: bool HexagonVectorLoopCarriedReuse::doVLCR() {
   562:   assert(CurLoop->getSubLoops().empty() &&
   563:          "Can do VLCR on the innermost loop only");
   564:   assert((CurLoop->getNumBlocks() == 1) &&
   565:          "Can do VLCR only on single block loops");
   566: 
   567:   bool Changed = false;
   568:   bool Continue;
   569: 
   570:   LLVM_DEBUG(dbgs() << "Working on Loop: " << *CurLoop->getHeader() << "\n");
   571:   do {
   572:     // Reset datastructures.
   573:     Dependences.clear();
   574:     Continue = false;
   575: 
   576:     findLoopCarriedDeps();
   577:     findValueToReuse();
   578:     if (ReuseCandidate.isDefined()) {
   579:       reuseValue();
   580:       Changed = true;
   581:       Continue = true;
   582:     }
   583:     llvm::for_each(Dependences, std::default_delete<DepChain>());
   584:   } while (Continue);
   585:   return Changed;
   586: }
   587: 
   588: void HexagonVectorLoopCarriedReuse::findDepChainFromPHI(Instruction *I,
   589:                                                         DepChain &D) {
   590:   PHINode *PN = dyn_cast<PHINode>(I);
   591:   if (!PN) {
   592:     D.push_back(I);
   593:     return;
   594:   } else {
   595:     auto NumIncomingValues = PN->getNumIncomingValues();
   596:     if (NumIncomingValues != 2) {
   597:       D.clear();
   598:       return;
   599:     }
   600: 
```
- EN: It declares or implements routines such as replaceAllUsesWith, insert, HexagonVectorLoopCarriedReuse::doVLCR, assert, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonNumVectorLoopCarriedReuse, HexagonVectorLoopCarriedReuse, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 replaceAllUsesWith, insert, HexagonVectorLoopCarriedReuse::doVLCR, assert, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonNumVectorLoopCarriedReuse, HexagonVectorLoopCarriedReuse，说明了它与同级后端组件的连接关系。

### Lines 601-650 / 第 601-650 行

```cpp
   601:     BasicBlock *BB = PN->getParent();
   602:     if (BB != CurLoop->getHeader()) {
   603:       D.clear();
   604:       return;
   605:     }
   606: 
   607:     Value *BEVal = PN->getIncomingValueForBlock(BB);
   608:     Instruction *BEInst = dyn_cast<Instruction>(BEVal);
   609:     // This is a single block loop with a preheader, so at least
   610:     // one value should come over the backedge.
   611:     assert(BEInst && "There should be a value over the backedge");
   612: 
   613:     Value *PreHdrVal =
   614:       PN->getIncomingValueForBlock(CurLoop->getLoopPreheader());
   615:     if(!PreHdrVal || !isa<Instruction>(PreHdrVal)) {
   616:       D.clear();
   617:       return;
   618:     }
   619:     D.push_back(PN);
   620:     findDepChainFromPHI(BEInst, D);
   621:   }
   622: }
   623: 
   624: DepChain *HexagonVectorLoopCarriedReuse::getDepChainBtwn(Instruction *I1,
   625:                                                          Instruction *I2,
   626:                                                          int Iters) {
   627:   for (auto *D : Dependences) {
   628:     if (D->front() == I1 && D->back() == I2 && D->iterations() == Iters)
   629:       return D;
   630:   }
   631:   return nullptr;
   632: }
   633: 
   634: void HexagonVectorLoopCarriedReuse::findLoopCarriedDeps() {
   635:   BasicBlock *BB = CurLoop->getHeader();
   636:   for (auto I = BB->begin(), E = BB->end(); I != E && isa<PHINode>(I); ++I) {
   637:     auto *PN = cast<PHINode>(I);
   638:     if (!isa<VectorType>(PN->getType()))
   639:       continue;
   640: 
   641:     DepChain *D = new DepChain();
   642:     findDepChainFromPHI(PN, *D);
   643:     if (D->size() != 0)
   644:       Dependences.insert(D);
   645:     else
   646:       delete D;
   647:   }
   648:   LLVM_DEBUG(dbgs() << "Found " << Dependences.size() << " dependences\n");
   649:   LLVM_DEBUG(for (const DepChain *D : Dependences) dbgs() << *D << "\n";);
   650: }
```
- EN: It declares or implements routines such as getParent, clear, getIncomingValueForBlock, dyn_cast<Instruction>, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonVectorLoopCarriedReuse, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getParent, clear, getIncomingValueForBlock, dyn_cast<Instruction>, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonVectorLoopCarriedReuse，说明了它与同级后端组件的连接关系。

### Lines 651-654 / 第 651-654 行

```cpp
   651: 
   652: Pass *llvm::createHexagonVectorLoopCarriedReuseLegacyPass() {
   653:   return new HexagonVectorLoopCarriedReuseLegacyPass();
   654: }
```
- EN: It declares or implements routines such as llvm::createHexagonVectorLoopCarriedReuseLegacyPass, HexagonVectorLoopCarriedReuseLegacyPass, translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonVectorLoopCarriedReuseLegacyPass, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 llvm::createHexagonVectorLoopCarriedReuseLegacyPass, HexagonVectorLoopCarriedReuseLegacyPass 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonVectorLoopCarriedReuseLegacyPass，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- loop-aware code generation / 循环相关代码生成
- instruction semantics / 指令语义
- intrinsic mapping / Intrinsic 映射
- generated metadata tables / 生成的元数据表

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonVectorLoopCarriedReuse.h, Hexagon.h, llvm/ADT/SetVector.h, llvm/ADT/SmallVector.h, llvm/ADT/Statistic.h, llvm/Analysis/LoopInfo.h, llvm/Analysis/LoopPass.h, llvm/IR/BasicBlock.h, llvm/IR/DerivedTypes.h, llvm/IR/IRBuilder.h, ... (30 total)`
- Hexagon symbols / Hexagon 符号: `HexagonVectorLoopCarriedReuse, HexagonNumVectorLoopCarriedReuse, HexagonVLCRIterationLim, HexagonVectorLoopCarriedReuseLegacyPass, HexagonVectorLoopCarriedReusePass`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
