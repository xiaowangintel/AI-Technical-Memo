# HexagonLoopIdiomRecognition.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonLoopIdiomRecognition.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): #include "llvm/TargetParser/Triple.h
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-200 / 第 1-200 行

```cpp
     1: //===- HexagonLoopIdiomRecognition.cpp ------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #include "HexagonLoopIdiomRecognition.h"
    10: #include "Hexagon.h"
    11: #include "llvm/ADT/APInt.h"
    12: #include "llvm/ADT/DenseMap.h"
    13: #include "llvm/ADT/SetVector.h"
    14: #include "llvm/ADT/SmallPtrSet.h"
    15: #include "llvm/ADT/SmallVector.h"
    16: #include "llvm/ADT/StringRef.h"
    17: #include "llvm/Analysis/AliasAnalysis.h"
    18: #include "llvm/Analysis/InstructionSimplify.h"
    19: #include "llvm/Analysis/LoopAnalysisManager.h"
    20: #include "llvm/Analysis/LoopInfo.h"
    21: #include "llvm/Analysis/LoopPass.h"
    22: #include "llvm/Analysis/MemoryLocation.h"
    23: #include "llvm/Analysis/ScalarEvolution.h"
    24: #include "llvm/Analysis/ScalarEvolutionExpressions.h"
    25: #include "llvm/Analysis/TargetLibraryInfo.h"
    26: #include "llvm/Analysis/ValueTracking.h"
    27: #include "llvm/IR/Attributes.h"
    28: #include "llvm/IR/BasicBlock.h"
    29: #include "llvm/IR/Constant.h"
    30: #include "llvm/IR/Constants.h"
    31: #include "llvm/IR/DataLayout.h"
    32: #include "llvm/IR/DebugLoc.h"
    33: #include "llvm/IR/DerivedTypes.h"
    34: #include "llvm/IR/Dominators.h"
    35: #include "llvm/IR/Function.h"
    36: #include "llvm/IR/IRBuilder.h"
    37: #include "llvm/IR/InstrTypes.h"
    38: #include "llvm/IR/Instruction.h"
    39: #include "llvm/IR/Instructions.h"
    40: #include "llvm/IR/Intrinsics.h"
    41: #include "llvm/IR/IntrinsicsHexagon.h"
    42: #include "llvm/IR/Module.h"
    43: #include "llvm/IR/PassManager.h"
    44: #include "llvm/IR/PatternMatch.h"
    45: #include "llvm/IR/RuntimeLibcalls.h"
    46: #include "llvm/IR/Type.h"
    47: #include "llvm/IR/User.h"
    48: #include "llvm/IR/Value.h"
    49: #include "llvm/InitializePasses.h"
    50: #include "llvm/Pass.h"
    51: #include "llvm/Support/Casting.h"
    52: #include "llvm/Support/CommandLine.h"
    53: #include "llvm/Support/Compiler.h"
    54: #include "llvm/Support/Debug.h"
    55: #include "llvm/Support/ErrorHandling.h"
    56: #include "llvm/Support/KnownBits.h"
    57: #include "llvm/Support/raw_ostream.h"
    58: #include "llvm/TargetParser/Triple.h"
    59: #include "llvm/Transforms/Scalar.h"
    60: #include "llvm/Transforms/Utils.h"
    61: #include "llvm/Transforms/Utils/Local.h"
    62: #include "llvm/Transforms/Utils/ScalarEvolutionExpander.h"
    63: #include <algorithm>
    64: #include <array>
    65: #include <cassert>
    66: #include <cstdint>
    67: #include <cstdlib>
    68: #include <deque>
    69: #include <functional>
    70: #include <iterator>
    71: #include <map>
    72: #include <set>
    73: #include <utility>
    74: #include <vector>
    75: 
    76: #define DEBUG_TYPE "hexagon-lir"
    77: 
    78: using namespace llvm;
    79: 
    80: static cl::opt<bool> DisableMemcpyIdiom("disable-memcpy-idiom",
    81:   cl::Hidden, cl::init(false),
    82:   cl::desc("Disable generation of memcpy in loop idiom recognition"));
    83: 
    84: static cl::opt<bool> DisableMemmoveIdiom("disable-memmove-idiom",
    85:   cl::Hidden, cl::init(false),
    86:   cl::desc("Disable generation of memmove in loop idiom recognition"));
    87: 
    88: static cl::opt<unsigned> RuntimeMemSizeThreshold("runtime-mem-idiom-threshold",
    89:   cl::Hidden, cl::init(0), cl::desc("Threshold (in bytes) for the runtime "
    90:   "check guarding the memmove."));
    91: 
    92: static cl::opt<unsigned> CompileTimeMemSizeThreshold(
    93:   "compile-time-mem-idiom-threshold", cl::Hidden, cl::init(64),
    94:   cl::desc("Threshold (in bytes) to perform the transformation, if the "
    95:     "runtime loop count (mem transfer size) is known at compile-time."));
    96: 
    97: static cl::opt<bool> OnlyNonNestedMemmove("only-nonnested-memmove-idiom",
    98:   cl::Hidden, cl::init(true),
    99:   cl::desc("Only enable generating memmove in non-nested loops"));
   100: 
   101: static cl::opt<bool> HexagonVolatileMemcpy(
   102:     "disable-hexagon-volatile-memcpy", cl::Hidden, cl::init(false),
   103:     cl::desc("Enable Hexagon-specific memcpy for volatile destination."));
   104: 
   105: static cl::opt<unsigned> SimplifyLimit("hlir-simplify-limit", cl::init(10000),
   106:   cl::Hidden, cl::desc("Maximum number of simplification steps in HLIR"));
   107: 
   108: namespace {
   109: 
   110: class HexagonLoopIdiomRecognize {
   111: public:
   112:   explicit HexagonLoopIdiomRecognize(AliasAnalysis *AA, DominatorTree *DT,
   113:                                      LoopInfo *LF, const TargetLibraryInfo *TLI,
   114:                                      ScalarEvolution *SE)
   115:       : AA(AA), DT(DT), LF(LF), TLI(TLI), SE(SE) {}
   116: 
   117:   bool run(Loop *L);
   118: 
   119: private:
   120:   int getSCEVStride(const SCEVAddRecExpr *StoreEv);
   121:   bool isLegalStore(Loop *CurLoop, StoreInst *SI);
   122:   void collectStores(Loop *CurLoop, BasicBlock *BB,
   123:                      SmallVectorImpl<StoreInst *> &Stores);
   124:   bool processCopyingStore(Loop *CurLoop, StoreInst *SI, const SCEV *BECount);
   125:   bool coverLoop(Loop *L, SmallVectorImpl<Instruction *> &Insts) const;
   126:   bool runOnLoopBlock(Loop *CurLoop, BasicBlock *BB, const SCEV *BECount,
   127:                       SmallVectorImpl<BasicBlock *> &ExitBlocks);
   128:   bool runOnCountableLoop(Loop *L);
   129: 
   130:   AliasAnalysis *AA;
   131:   const DataLayout *DL;
   132:   DominatorTree *DT;
   133:   LoopInfo *LF;
   134:   const TargetLibraryInfo *TLI;
   135:   ScalarEvolution *SE;
   136:   bool HasMemcpy, HasMemmove;
   137: };
   138: 
   139: class HexagonLoopIdiomRecognizeLegacyPass : public LoopPass {
   140: public:
   141:   static char ID;
   142: 
   143:   explicit HexagonLoopIdiomRecognizeLegacyPass() : LoopPass(ID) {}
   144: 
   145:   StringRef getPassName() const override {
   146:     return "Recognize Hexagon-specific loop idioms";
   147:   }
   148: 
   149:   void getAnalysisUsage(AnalysisUsage &AU) const override {
   150:     AU.addRequired<LoopInfoWrapperPass>();
   151:     AU.addRequiredID(LoopSimplifyID);
   152:     AU.addRequiredID(LCSSAID);
   153:     AU.addRequired<AAResultsWrapperPass>();
   154:     AU.addRequired<ScalarEvolutionWrapperPass>();
   155:     AU.addRequired<DominatorTreeWrapperPass>();
   156:     AU.addRequired<TargetLibraryInfoWrapperPass>();
   157:     AU.addPreserved<TargetLibraryInfoWrapperPass>();
   158:   }
   159: 
   160:   bool runOnLoop(Loop *L, LPPassManager &LPM) override;
   161: };
   162: 
   163: struct Simplifier {
   164:   struct Rule {
   165:     using FuncType = std::function<Value *(Instruction *, LLVMContext &)>;
   166:     Rule(StringRef N, FuncType F) : Name(N), Fn(F) {}
   167:     StringRef Name; // For debugging.
   168:     FuncType Fn;
   169:   };
   170: 
   171:   void addRule(StringRef N, const Rule::FuncType &F) {
   172:     Rules.push_back(Rule(N, F));
   173:   }
   174: 
   175: private:
   176:   struct WorkListType {
   177:     WorkListType() = default;
   178: 
   179:     void push_back(Value *V) {
   180:       // Do not push back duplicates.
   181:       if (S.insert(V).second)
   182:         Q.push_back(V);
   183:     }
   184: 
   185:     Value *pop_front_val() {
   186:       Value *V = Q.front();
   187:       Q.pop_front();
   188:       S.erase(V);
   189:       return V;
   190:     }
   191: 
   192:     bool empty() const { return Q.empty(); }
   193: 
   194:   private:
   195:     std::deque<Value *> Q;
   196:     std::set<Value *> S;
   197:   };
   198: 
   199:   using ValueSetType = std::set<Value *>;
   200: 
```
- EN: It imports headers such as HexagonLoopIdiomRecognition.h, Hexagon.h, llvm/ADT/APInt.h, llvm/ADT/DenseMap.h, ... (66 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonLoopIdiomRecognize, HexagonLoopIdiomRecognizeLegacyPass, Simplifier, Rule, ... (5 total), which carry the state or API of this component.
- CN: 这里引入了 HexagonLoopIdiomRecognition.h, Hexagon.h, llvm/ADT/APInt.h, llvm/ADT/DenseMap.h, ... (66 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonLoopIdiomRecognize, HexagonLoopIdiomRecognizeLegacyPass, Simplifier, Rule, ... (5 total) 等类型，用来承载该组件的状态或接口。

### Lines 201-400 / 第 201-400 行

```cpp
   201:   std::vector<Rule> Rules;
   202: 
   203: public:
   204:   struct Context {
   205:     using ValueMapType = DenseMap<Value *, Value *>;
   206: 
   207:     Value *Root;
   208:     ValueSetType Used;   // The set of all cloned values used by Root.
   209:     ValueSetType Clones; // The set of all cloned values.
   210:     LLVMContext &Ctx;
   211: 
   212:     Context(Instruction *Exp)
   213:         : Ctx(Exp->getParent()->getParent()->getContext()) {
   214:       initialize(Exp);
   215:     }
   216: 
   217:     ~Context() { cleanup(); }
   218: 
   219:     void print(raw_ostream &OS, const Value *V) const;
   220:     Value *materialize(BasicBlock *B, BasicBlock::iterator At);
   221: 
   222:   private:
   223:     friend struct Simplifier;
   224: 
   225:     void initialize(Instruction *Exp);
   226:     void cleanup();
   227: 
   228:     template <typename FuncT> void traverse(Value *V, FuncT F);
   229:     void record(Value *V);
   230:     void use(Value *V);
   231:     void unuse(Value *V);
   232: 
   233:     bool equal(const Instruction *I, const Instruction *J) const;
   234:     Value *find(Value *Tree, Value *Sub) const;
   235:     Value *subst(Value *Tree, Value *OldV, Value *NewV);
   236:     void replace(Value *OldV, Value *NewV);
   237:     void link(Instruction *I, BasicBlock *B, BasicBlock::iterator At);
   238:   };
   239: 
   240:   Value *simplify(Context &C);
   241: };
   242: 
   243:   struct PE {
   244:     PE(const Simplifier::Context &c, Value *v = nullptr) : C(c), V(v) {}
   245: 
   246:     const Simplifier::Context &C;
   247:     const Value *V;
   248:   };
   249: 
   250:   LLVM_ATTRIBUTE_USED
   251:   raw_ostream &operator<<(raw_ostream &OS, const PE &P) {
   252:     P.C.print(OS, P.V ? P.V : P.C.Root);
   253:     return OS;
   254:   }
   255: 
   256: } // end anonymous namespace
   257: 
   258: char HexagonLoopIdiomRecognizeLegacyPass::ID = 0;
   259: 
   260: INITIALIZE_PASS_BEGIN(HexagonLoopIdiomRecognizeLegacyPass, "hexagon-loop-idiom",
   261:                       "Recognize Hexagon-specific loop idioms", false, false)
   262: INITIALIZE_PASS_DEPENDENCY(LoopInfoWrapperPass)
   263: INITIALIZE_PASS_DEPENDENCY(LoopSimplify)
   264: INITIALIZE_PASS_DEPENDENCY(LCSSAWrapperPass)
   265: INITIALIZE_PASS_DEPENDENCY(ScalarEvolutionWrapperPass)
   266: INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass)
   267: INITIALIZE_PASS_DEPENDENCY(TargetLibraryInfoWrapperPass)
   268: INITIALIZE_PASS_DEPENDENCY(AAResultsWrapperPass)
   269: INITIALIZE_PASS_END(HexagonLoopIdiomRecognizeLegacyPass, "hexagon-loop-idiom",
   270:                     "Recognize Hexagon-specific loop idioms", false, false)
   271: 
   272: template <typename FuncT>
   273: void Simplifier::Context::traverse(Value *V, FuncT F) {
   274:   WorkListType Q;
   275:   Q.push_back(V);
   276: 
   277:   while (!Q.empty()) {
   278:     Instruction *U = dyn_cast<Instruction>(Q.pop_front_val());
   279:     if (!U || U->getParent())
   280:       continue;
   281:     if (!F(U))
   282:       continue;
   283:     for (Value *Op : U->operands())
   284:       Q.push_back(Op);
   285:   }
   286: }
   287: 
   288: void Simplifier::Context::print(raw_ostream &OS, const Value *V) const {
   289:   const auto *U = dyn_cast<const Instruction>(V);
   290:   if (!U) {
   291:     OS << V << '(' << *V << ')';
   292:     return;
   293:   }
   294: 
   295:   if (U->getParent()) {
   296:     OS << U << '(';
   297:     U->printAsOperand(OS, true);
   298:     OS << ')';
   299:     return;
   300:   }
   301: 
   302:   unsigned N = U->getNumOperands();
   303:   if (N != 0)
   304:     OS << U << '(';
   305:   OS << U->getOpcodeName();
   306:   for (const Value *Op : U->operands()) {
   307:     OS << ' ';
   308:     print(OS, Op);
   309:   }
   310:   if (N != 0)
   311:     OS << ')';
   312: }
   313: 
   314: void Simplifier::Context::initialize(Instruction *Exp) {
   315:   // Perform a deep clone of the expression, set Root to the root
   316:   // of the clone, and build a map from the cloned values to the
   317:   // original ones.
   318:   ValueMapType M;
   319:   BasicBlock *Block = Exp->getParent();
   320:   WorkListType Q;
   321:   Q.push_back(Exp);
   322: 
   323:   while (!Q.empty()) {
   324:     Value *V = Q.pop_front_val();
   325:     if (M.contains(V))
   326:       continue;
   327:     if (Instruction *U = dyn_cast<Instruction>(V)) {
   328:       if (isa<PHINode>(U) || U->getParent() != Block)
   329:         continue;
   330:       for (Value *Op : U->operands())
   331:         Q.push_back(Op);
   332:       M.insert({U, U->clone()});
   333:     }
   334:   }
   335: 
   336:   for (std::pair<Value*,Value*> P : M) {
   337:     Instruction *U = cast<Instruction>(P.second);
   338:     for (unsigned i = 0, n = U->getNumOperands(); i != n; ++i) {
   339:       auto F = M.find(U->getOperand(i));
   340:       if (F != M.end())
   341:         U->setOperand(i, F->second);
   342:     }
   343:   }
   344: 
   345:   auto R = M.find(Exp);
   346:   assert(R != M.end());
   347:   Root = R->second;
   348: 
   349:   record(Root);
   350:   use(Root);
   351: }
   352: 
   353: void Simplifier::Context::record(Value *V) {
   354:   auto Record = [this](Instruction *U) -> bool {
   355:     Clones.insert(U);
   356:     return true;
   357:   };
   358:   traverse(V, Record);
   359: }
   360: 
   361: void Simplifier::Context::use(Value *V) {
   362:   auto Use = [this](Instruction *U) -> bool {
   363:     Used.insert(U);
   364:     return true;
   365:   };
   366:   traverse(V, Use);
   367: }
   368: 
   369: void Simplifier::Context::unuse(Value *V) {
   370:   if (!isa<Instruction>(V) || cast<Instruction>(V)->getParent() != nullptr)
   371:     return;
   372: 
   373:   auto Unuse = [this](Instruction *U) -> bool {
   374:     if (!U->use_empty())
   375:       return false;
   376:     Used.erase(U);
   377:     return true;
   378:   };
   379:   traverse(V, Unuse);
   380: }
   381: 
   382: Value *Simplifier::Context::subst(Value *Tree, Value *OldV, Value *NewV) {
   383:   if (Tree == OldV)
   384:     return NewV;
   385:   if (OldV == NewV)
   386:     return Tree;
   387: 
   388:   WorkListType Q;
   389:   Q.push_back(Tree);
   390:   while (!Q.empty()) {
   391:     Instruction *U = dyn_cast<Instruction>(Q.pop_front_val());
   392:     // If U is not an instruction, or it's not a clone, skip it.
   393:     if (!U || U->getParent())
   394:       continue;
   395:     for (unsigned i = 0, n = U->getNumOperands(); i != n; ++i) {
   396:       Value *Op = U->getOperand(i);
   397:       if (Op == OldV) {
   398:         U->setOperand(i, NewV);
   399:         unuse(OldV);
   400:       } else {
```
- EN: It opens namespaces (char) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as Context, PE, which carry the state or API of this component. It declares or implements routines such as Context, initialize, cleanup, print, ... (37 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里打开了命名空间（char），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 Context, PE 等类型，用来承载该组件的状态或接口。 这里声明或实现了 Context, initialize, cleanup, print, ... (37 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 401-600 / 第 401-600 行

```cpp
   401:         Q.push_back(Op);
   402:       }
   403:     }
   404:   }
   405:   return Tree;
   406: }
   407: 
   408: void Simplifier::Context::replace(Value *OldV, Value *NewV) {
   409:   if (Root == OldV) {
   410:     Root = NewV;
   411:     use(Root);
   412:     return;
   413:   }
   414: 
   415:   // NewV may be a complex tree that has just been created by one of the
   416:   // transformation rules. We need to make sure that it is commoned with
   417:   // the existing Root to the maximum extent possible.
   418:   // Identify all subtrees of NewV (including NewV itself) that have
   419:   // equivalent counterparts in Root, and replace those subtrees with
   420:   // these counterparts.
   421:   WorkListType Q;
   422:   Q.push_back(NewV);
   423:   while (!Q.empty()) {
   424:     Value *V = Q.pop_front_val();
   425:     Instruction *U = dyn_cast<Instruction>(V);
   426:     if (!U || U->getParent())
   427:       continue;
   428:     if (Value *DupV = find(Root, V)) {
   429:       if (DupV != V)
   430:         NewV = subst(NewV, V, DupV);
   431:     } else {
   432:       for (Value *Op : U->operands())
   433:         Q.push_back(Op);
   434:     }
   435:   }
   436: 
   437:   // Now, simply replace OldV with NewV in Root.
   438:   Root = subst(Root, OldV, NewV);
   439:   use(Root);
   440: }
   441: 
   442: void Simplifier::Context::cleanup() {
   443:   for (Value *V : Clones) {
   444:     Instruction *U = cast<Instruction>(V);
   445:     if (!U->getParent())
   446:       U->dropAllReferences();
   447:   }
   448: 
   449:   for (Value *V : Clones) {
   450:     Instruction *U = cast<Instruction>(V);
   451:     if (!U->getParent())
   452:       U->deleteValue();
   453:   }
   454: }
   455: 
   456: bool Simplifier::Context::equal(const Instruction *I,
   457:                                 const Instruction *J) const {
   458:   if (I == J)
   459:     return true;
   460:   if (!I->isSameOperationAs(J))
   461:     return false;
   462:   if (isa<PHINode>(I))
   463:     return I->isIdenticalTo(J);
   464: 
   465:   for (unsigned i = 0, n = I->getNumOperands(); i != n; ++i) {
   466:     Value *OpI = I->getOperand(i), *OpJ = J->getOperand(i);
   467:     if (OpI == OpJ)
   468:       continue;
   469:     auto *InI = dyn_cast<const Instruction>(OpI);
   470:     auto *InJ = dyn_cast<const Instruction>(OpJ);
   471:     if (InI && InJ) {
   472:       if (!equal(InI, InJ))
   473:         return false;
   474:     } else if (InI != InJ || !InI)
   475:       return false;
   476:   }
   477:   return true;
   478: }
   479: 
   480: Value *Simplifier::Context::find(Value *Tree, Value *Sub) const {
   481:   Instruction *SubI = dyn_cast<Instruction>(Sub);
   482:   WorkListType Q;
   483:   Q.push_back(Tree);
   484: 
   485:   while (!Q.empty()) {
   486:     Value *V = Q.pop_front_val();
   487:     if (V == Sub)
   488:       return V;
   489:     Instruction *U = dyn_cast<Instruction>(V);
   490:     if (!U || U->getParent())
   491:       continue;
   492:     if (SubI && equal(SubI, U))
   493:       return U;
   494:     assert(!isa<PHINode>(U));
   495:     for (Value *Op : U->operands())
   496:       Q.push_back(Op);
   497:   }
   498:   return nullptr;
   499: }
   500: 
   501: void Simplifier::Context::link(Instruction *I, BasicBlock *B,
   502:       BasicBlock::iterator At) {
   503:   if (I->getParent())
   504:     return;
   505: 
   506:   for (Value *Op : I->operands()) {
   507:     if (Instruction *OpI = dyn_cast<Instruction>(Op))
   508:       link(OpI, B, At);
   509:   }
   510: 
   511:   I->insertInto(B, At);
   512: }
   513: 
   514: Value *Simplifier::Context::materialize(BasicBlock *B,
   515:       BasicBlock::iterator At) {
   516:   if (Instruction *RootI = dyn_cast<Instruction>(Root))
   517:     link(RootI, B, At);
   518:   return Root;
   519: }
   520: 
   521: Value *Simplifier::simplify(Context &C) {
   522:   WorkListType Q;
   523:   Q.push_back(C.Root);
   524:   unsigned Count = 0;
   525:   const unsigned Limit = SimplifyLimit;
   526: 
   527:   while (!Q.empty()) {
   528:     if (Count++ >= Limit)
   529:       break;
   530:     Instruction *U = dyn_cast<Instruction>(Q.pop_front_val());
   531:     if (!U || U->getParent() || !C.Used.count(U))
   532:       continue;
   533:     bool Changed = false;
   534:     for (Rule &R : Rules) {
   535:       Value *W = R.Fn(U, C.Ctx);
   536:       if (!W)
   537:         continue;
   538:       Changed = true;
   539:       C.record(W);
   540:       C.replace(U, W);
   541:       Q.push_back(C.Root);
   542:       break;
   543:     }
   544:     if (!Changed) {
   545:       for (Value *Op : U->operands())
   546:         Q.push_back(Op);
   547:     }
   548:   }
   549:   return Count < Limit ? C.Root : nullptr;
   550: }
   551: 
   552: //===----------------------------------------------------------------------===//
   553: //
   554: //          Implementation of PolynomialMultiplyRecognize
   555: //
   556: //===----------------------------------------------------------------------===//
   557: 
   558: namespace {
   559: 
   560:   class PolynomialMultiplyRecognize {
   561:   public:
   562:     explicit PolynomialMultiplyRecognize(Loop *loop, const DataLayout &dl,
   563:         const DominatorTree &dt, const TargetLibraryInfo &tli,
   564:         ScalarEvolution &se)
   565:       : CurLoop(loop), DL(dl), DT(dt), TLI(tli), SE(se) {}
   566: 
   567:     bool recognize();
   568: 
   569:   private:
   570:     using ValueSeq = SetVector<Value *>;
   571: 
   572:     IntegerType *getPmpyType() const {
   573:       LLVMContext &Ctx = CurLoop->getHeader()->getParent()->getContext();
   574:       return IntegerType::get(Ctx, 32);
   575:     }
   576: 
   577:     bool isPromotableTo(Value *V, IntegerType *Ty);
   578:     void promoteTo(Instruction *In, IntegerType *DestTy, BasicBlock *LoopB);
   579:     bool promoteTypes(BasicBlock *LoopB, BasicBlock *ExitB);
   580: 
   581:     Value *getCountIV(BasicBlock *BB);
   582:     bool findCycle(Value *Out, Value *In, ValueSeq &Cycle);
   583:     void classifyCycle(Instruction *DivI, ValueSeq &Cycle, ValueSeq &Early,
   584:           ValueSeq &Late);
   585:     bool classifyInst(Instruction *UseI, ValueSeq &Early, ValueSeq &Late);
   586:     bool commutesWithShift(Instruction *I);
   587:     bool highBitsAreZero(Value *V, unsigned IterCount);
   588:     bool keepsHighBitsZero(Value *V, unsigned IterCount);
   589:     bool isOperandShifted(Instruction *I, Value *Op);
   590:     bool convertShiftsToLeft(BasicBlock *LoopB, BasicBlock *ExitB,
   591:           unsigned IterCount);
   592:     void cleanupLoopBody(BasicBlock *LoopB);
   593: 
   594:     struct ParsedValues {
   595:       ParsedValues() = default;
   596: 
   597:       Value *M = nullptr;
   598:       Value *P = nullptr;
   599:       Value *Q = nullptr;
   600:       Value *R = nullptr;
```
- EN: It declares types such as PolynomialMultiplyRecognize, ParsedValues, which carry the state or API of this component. It defines declarative TableGen records like PolynomialMultiplyRecognize; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as push_back, Simplifier::Context::replace, use, pop_front_val, ... (39 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 PolynomialMultiplyRecognize, ParsedValues 等类型，用来承载该组件的状态或接口。 这里定义了 PolynomialMultiplyRecognize 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 push_back, Simplifier::Context::replace, use, pop_front_val, ... (39 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 601-800 / 第 601-800 行

```cpp
   601:       Value *X = nullptr;
   602:       Instruction *Res = nullptr;
   603:       unsigned IterCount = 0;
   604:       bool Left = false;
   605:       bool Inv = false;
   606:     };
   607: 
   608:     bool matchLeftShift(SelectInst *SelI, Value *CIV, ParsedValues &PV);
   609:     bool matchRightShift(SelectInst *SelI, ParsedValues &PV);
   610:     bool scanSelect(SelectInst *SI, BasicBlock *LoopB, BasicBlock *PrehB,
   611:           Value *CIV, ParsedValues &PV, bool PreScan);
   612:     unsigned getInverseMxN(unsigned QP);
   613:     Value *generate(BasicBlock::iterator At, ParsedValues &PV);
   614: 
   615:     void setupPreSimplifier(Simplifier &S);
   616:     void setupPostSimplifier(Simplifier &S);
   617: 
   618:     Loop *CurLoop;
   619:     const DataLayout &DL;
   620:     const DominatorTree &DT;
   621:     const TargetLibraryInfo &TLI;
   622:     ScalarEvolution &SE;
   623:   };
   624: 
   625: } // end anonymous namespace
   626: 
   627: Value *PolynomialMultiplyRecognize::getCountIV(BasicBlock *BB) {
   628:   pred_iterator PI = pred_begin(BB), PE = pred_end(BB);
   629:   if (std::distance(PI, PE) != 2)
   630:     return nullptr;
   631:   BasicBlock *PB = (*PI == BB) ? *std::next(PI) : *PI;
   632: 
   633:   for (auto I = BB->begin(), E = BB->end(); I != E && isa<PHINode>(I); ++I) {
   634:     auto *PN = cast<PHINode>(I);
   635:     Value *InitV = PN->getIncomingValueForBlock(PB);
   636:     if (!isa<ConstantInt>(InitV) || !cast<ConstantInt>(InitV)->isZero())
   637:       continue;
   638:     Value *IterV = PN->getIncomingValueForBlock(BB);
   639:     auto *BO = dyn_cast<BinaryOperator>(IterV);
   640:     if (!BO)
   641:       continue;
   642:     if (BO->getOpcode() != Instruction::Add)
   643:       continue;
   644:     Value *IncV = nullptr;
   645:     if (BO->getOperand(0) == PN)
   646:       IncV = BO->getOperand(1);
   647:     else if (BO->getOperand(1) == PN)
   648:       IncV = BO->getOperand(0);
   649:     if (IncV == nullptr)
   650:       continue;
   651: 
   652:     if (auto *T = dyn_cast<ConstantInt>(IncV))
   653:       if (T->isOne())
   654:         return PN;
   655:   }
   656:   return nullptr;
   657: }
   658: 
   659: static void replaceAllUsesOfWithIn(Value *I, Value *J, BasicBlock *BB) {
   660:   for (auto UI = I->user_begin(), UE = I->user_end(); UI != UE;) {
   661:     Use &TheUse = UI.getUse();
   662:     ++UI;
   663:     if (auto *II = dyn_cast<Instruction>(TheUse.getUser()))
   664:       if (BB == II->getParent())
   665:         II->replaceUsesOfWith(I, J);
   666:   }
   667: }
   668: 
   669: bool PolynomialMultiplyRecognize::matchLeftShift(SelectInst *SelI,
   670:       Value *CIV, ParsedValues &PV) {
   671:   // Match the following:
   672:   //   select (X & (1 << i)) != 0 ? R ^ (Q << i) : R
   673:   //   select (X & (1 << i)) == 0 ? R : R ^ (Q << i)
   674:   // The condition may also check for equality with the masked value, i.e
   675:   //   select (X & (1 << i)) == (1 << i) ? R ^ (Q << i) : R
   676:   //   select (X & (1 << i)) != (1 << i) ? R : R ^ (Q << i);
   677: 
   678:   Value *CondV = SelI->getCondition();
   679:   Value *TrueV = SelI->getTrueValue();
   680:   Value *FalseV = SelI->getFalseValue();
   681: 
   682:   using namespace PatternMatch;
   683: 
   684:   CmpPredicate P;
   685:   Value *A = nullptr, *B = nullptr, *C = nullptr;
   686: 
   687:   if (!match(CondV, m_ICmp(P, m_And(m_Value(A), m_Value(B)), m_Value(C))) &&
   688:       !match(CondV, m_ICmp(P, m_Value(C), m_And(m_Value(A), m_Value(B)))))
   689:     return false;
   690:   if (P != CmpInst::ICMP_EQ && P != CmpInst::ICMP_NE)
   691:     return false;
   692:   // Matched: select (A & B) == C ? ... : ...
   693:   //          select (A & B) != C ? ... : ...
   694: 
   695:   Value *X = nullptr, *Sh1 = nullptr;
   696:   // Check (A & B) for (X & (1 << i)):
   697:   if (match(A, m_Shl(m_One(), m_Specific(CIV)))) {
   698:     Sh1 = A;
   699:     X = B;
   700:   } else if (match(B, m_Shl(m_One(), m_Specific(CIV)))) {
   701:     Sh1 = B;
   702:     X = A;
   703:   } else {
   704:     // TODO: Could also check for an induction variable containing single
   705:     // bit shifted left by 1 in each iteration.
   706:     return false;
   707:   }
   708: 
   709:   bool TrueIfZero;
   710: 
   711:   // Check C against the possible values for comparison: 0 and (1 << i):
   712:   if (match(C, m_Zero()))
   713:     TrueIfZero = (P == CmpInst::ICMP_EQ);
   714:   else if (C == Sh1)
   715:     TrueIfZero = (P == CmpInst::ICMP_NE);
   716:   else
   717:     return false;
   718: 
   719:   // So far, matched:
   720:   //   select (X & (1 << i)) ? ... : ...
   721:   // including variations of the check against zero/non-zero value.
   722: 
   723:   Value *ShouldSameV = nullptr, *ShouldXoredV = nullptr;
   724:   if (TrueIfZero) {
   725:     ShouldSameV = TrueV;
   726:     ShouldXoredV = FalseV;
   727:   } else {
   728:     ShouldSameV = FalseV;
   729:     ShouldXoredV = TrueV;
   730:   }
   731: 
   732:   Value *Q = nullptr, *R = nullptr, *Y = nullptr, *Z = nullptr;
   733:   Value *T = nullptr;
   734:   if (match(ShouldXoredV, m_Xor(m_Value(Y), m_Value(Z)))) {
   735:     // Matched: select +++ ? ... : Y ^ Z
   736:     //          select +++ ? Y ^ Z : ...
   737:     // where +++ denotes previously checked matches.
   738:     if (ShouldSameV == Y)
   739:       T = Z;
   740:     else if (ShouldSameV == Z)
   741:       T = Y;
   742:     else
   743:       return false;
   744:     R = ShouldSameV;
   745:     // Matched: select +++ ? R : R ^ T
   746:     //          select +++ ? R ^ T : R
   747:     // depending on TrueIfZero.
   748: 
   749:   } else if (match(ShouldSameV, m_Zero())) {
   750:     // Matched: select +++ ? 0 : ...
   751:     //          select +++ ? ... : 0
   752:     if (!SelI->hasOneUse())
   753:       return false;
   754:     T = ShouldXoredV;
   755:     // Matched: select +++ ? 0 : T
   756:     //          select +++ ? T : 0
   757: 
   758:     Value *U = *SelI->user_begin();
   759:     if (!match(U, m_c_Xor(m_Specific(SelI), m_Value(R))))
   760:       return false;
   761:     // Matched: xor (select +++ ? 0 : T), R
   762:     //          xor (select +++ ? T : 0), R
   763:   } else
   764:     return false;
   765: 
   766:   // The xor input value T is isolated into its own match so that it could
   767:   // be checked against an induction variable containing a shifted bit
   768:   // (todo).
   769:   // For now, check against (Q << i).
   770:   if (!match(T, m_Shl(m_Value(Q), m_Specific(CIV))) &&
   771:       !match(T, m_Shl(m_ZExt(m_Value(Q)), m_ZExt(m_Specific(CIV)))))
   772:     return false;
   773:   // Matched: select +++ ? R : R ^ (Q << i)
   774:   //          select +++ ? R ^ (Q << i) : R
   775: 
   776:   PV.X = X;
   777:   PV.Q = Q;
   778:   PV.R = R;
   779:   PV.Left = true;
   780:   return true;
   781: }
   782: 
   783: bool PolynomialMultiplyRecognize::matchRightShift(SelectInst *SelI,
   784:       ParsedValues &PV) {
   785:   // Match the following:
   786:   //   select (X & 1) != 0 ? (R >> 1) ^ Q : (R >> 1)
   787:   //   select (X & 1) == 0 ? (R >> 1) : (R >> 1) ^ Q
   788:   // The condition may also check for equality with the masked value, i.e
   789:   //   select (X & 1) == 1 ? (R >> 1) ^ Q : (R >> 1)
   790:   //   select (X & 1) != 1 ? (R >> 1) : (R >> 1) ^ Q
   791: 
   792:   Value *CondV = SelI->getCondition();
   793:   Value *TrueV = SelI->getTrueValue();
   794:   Value *FalseV = SelI->getFalseValue();
   795: 
   796:   using namespace PatternMatch;
   797: 
   798:   Value *C = nullptr;
   799:   CmpPredicate P;
   800:   bool TrueIfZero;
```
- EN: It opens namespaces (Value, PatternMatch) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as matchLeftShift, matchRightShift, scanSelect, getInverseMxN, ... (24 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里打开了命名空间（Value, PatternMatch），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 matchLeftShift, matchRightShift, scanSelect, getInverseMxN, ... (24 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 801-1000 / 第 801-1000 行

```cpp
   801: 
   802:   if (match(CondV, m_c_ICmp(P, m_Value(C), m_Zero()))) {
   803:     if (P != CmpInst::ICMP_EQ && P != CmpInst::ICMP_NE)
   804:       return false;
   805:     // Matched: select C == 0 ? ... : ...
   806:     //          select C != 0 ? ... : ...
   807:     TrueIfZero = (P == CmpInst::ICMP_EQ);
   808:   } else if (match(CondV, m_c_ICmp(P, m_Value(C), m_One()))) {
   809:     if (P != CmpInst::ICMP_EQ && P != CmpInst::ICMP_NE)
   810:       return false;
   811:     // Matched: select C == 1 ? ... : ...
   812:     //          select C != 1 ? ... : ...
   813:     TrueIfZero = (P == CmpInst::ICMP_NE);
   814:   } else
   815:     return false;
   816: 
   817:   Value *X = nullptr;
   818:   if (!match(C, m_And(m_Value(X), m_One())))
   819:     return false;
   820:   // Matched: select (X & 1) == +++ ? ... : ...
   821:   //          select (X & 1) != +++ ? ... : ...
   822: 
   823:   Value *R = nullptr, *Q = nullptr;
   824:   if (TrueIfZero) {
   825:     // The select's condition is true if the tested bit is 0.
   826:     // TrueV must be the shift, FalseV must be the xor.
   827:     if (!match(TrueV, m_LShr(m_Value(R), m_One())))
   828:       return false;
   829:     // Matched: select +++ ? (R >> 1) : ...
   830:     if (!match(FalseV, m_c_Xor(m_Specific(TrueV), m_Value(Q))))
   831:       return false;
   832:     // Matched: select +++ ? (R >> 1) : (R >> 1) ^ Q
   833:     // with commuting ^.
   834:   } else {
   835:     // The select's condition is true if the tested bit is 1.
   836:     // TrueV must be the xor, FalseV must be the shift.
   837:     if (!match(FalseV, m_LShr(m_Value(R), m_One())))
   838:       return false;
   839:     // Matched: select +++ ? ... : (R >> 1)
   840:     if (!match(TrueV, m_c_Xor(m_Specific(FalseV), m_Value(Q))))
   841:       return false;
   842:     // Matched: select +++ ? (R >> 1) ^ Q : (R >> 1)
   843:     // with commuting ^.
   844:   }
   845: 
   846:   PV.X = X;
   847:   PV.Q = Q;
   848:   PV.R = R;
   849:   PV.Left = false;
   850:   return true;
   851: }
   852: 
   853: bool PolynomialMultiplyRecognize::scanSelect(SelectInst *SelI,
   854:       BasicBlock *LoopB, BasicBlock *PrehB, Value *CIV, ParsedValues &PV,
   855:       bool PreScan) {
   856:   using namespace PatternMatch;
   857: 
   858:   // The basic pattern for R = P.Q is:
   859:   // for i = 0..31
   860:   //   R = phi (0, R')
   861:   //   if (P & (1 << i))        ; test-bit(P, i)
   862:   //     R' = R ^ (Q << i)
   863:   //
   864:   // Similarly, the basic pattern for R = (P/Q).Q - P
   865:   // for i = 0..31
   866:   //   R = phi(P, R')
   867:   //   if (R & (1 << i))
   868:   //     R' = R ^ (Q << i)
   869: 
   870:   // There exist idioms, where instead of Q being shifted left, P is shifted
   871:   // right. This produces a result that is shifted right by 32 bits (the
   872:   // non-shifted result is 64-bit).
   873:   //
   874:   // For R = P.Q, this would be:
   875:   // for i = 0..31
   876:   //   R = phi (0, R')
   877:   //   if ((P >> i) & 1)
   878:   //     R' = (R >> 1) ^ Q      ; R is cycled through the loop, so it must
   879:   //   else                     ; be shifted by 1, not i.
   880:   //     R' = R >> 1
   881:   //
   882:   // And for the inverse:
   883:   // for i = 0..31
   884:   //   R = phi (P, R')
   885:   //   if (R & 1)
   886:   //     R' = (R >> 1) ^ Q
   887:   //   else
   888:   //     R' = R >> 1
   889: 
   890:   // The left-shifting idioms share the same pattern:
   891:   //   select (X & (1 << i)) ? R ^ (Q << i) : R
   892:   // Similarly for right-shifting idioms:
   893:   //   select (X & 1) ? (R >> 1) ^ Q
   894: 
   895:   if (matchLeftShift(SelI, CIV, PV)) {
   896:     // If this is a pre-scan, getting this far is sufficient.
   897:     if (PreScan)
   898:       return true;
   899: 
   900:     // Need to make sure that the SelI goes back into R.
   901:     auto *RPhi = dyn_cast<PHINode>(PV.R);
   902:     if (!RPhi)
   903:       return false;
   904:     if (SelI != RPhi->getIncomingValueForBlock(LoopB))
   905:       return false;
   906:     PV.Res = SelI;
   907: 
   908:     // If X is loop invariant, it must be the input polynomial, and the
   909:     // idiom is the basic polynomial multiply.
   910:     if (CurLoop->isLoopInvariant(PV.X)) {
   911:       PV.P = PV.X;
   912:       PV.Inv = false;
   913:     } else {
   914:       // X is not loop invariant. If X == R, this is the inverse pmpy.
   915:       // Otherwise, check for an xor with an invariant value. If the
   916:       // variable argument to the xor is R, then this is still a valid
   917:       // inverse pmpy.
   918:       PV.Inv = true;
   919:       if (PV.X != PV.R) {
   920:         Value *Var = nullptr, *Inv = nullptr, *X1 = nullptr, *X2 = nullptr;
   921:         if (!match(PV.X, m_Xor(m_Value(X1), m_Value(X2))))
   922:           return false;
   923:         auto *I1 = dyn_cast<Instruction>(X1);
   924:         auto *I2 = dyn_cast<Instruction>(X2);
   925:         if (!I1 || I1->getParent() != LoopB) {
   926:           Var = X2;
   927:           Inv = X1;
   928:         } else if (!I2 || I2->getParent() != LoopB) {
   929:           Var = X1;
   930:           Inv = X2;
   931:         } else
   932:           return false;
   933:         if (Var != PV.R)
   934:           return false;
   935:         PV.M = Inv;
   936:       }
   937:       // The input polynomial P still needs to be determined. It will be
   938:       // the entry value of R.
   939:       Value *EntryP = RPhi->getIncomingValueForBlock(PrehB);
   940:       PV.P = EntryP;
   941:     }
   942: 
   943:     return true;
   944:   }
   945: 
   946:   if (matchRightShift(SelI, PV)) {
   947:     // If this is an inverse pattern, the Q polynomial must be known at
   948:     // compile time.
   949:     if (PV.Inv && !isa<ConstantInt>(PV.Q))
   950:       return false;
   951:     if (PreScan)
   952:       return true;
   953:     // There is no exact matching of right-shift pmpy.
   954:     return false;
   955:   }
   956: 
   957:   return false;
   958: }
   959: 
   960: bool PolynomialMultiplyRecognize::isPromotableTo(Value *Val,
   961:       IntegerType *DestTy) {
   962:   IntegerType *T = dyn_cast<IntegerType>(Val->getType());
   963:   if (!T || T->getBitWidth() > DestTy->getBitWidth())
   964:     return false;
   965:   if (T->getBitWidth() == DestTy->getBitWidth())
   966:     return true;
   967:   // Non-instructions are promotable. The reason why an instruction may not
   968:   // be promotable is that it may produce a different result if its operands
   969:   // and the result are promoted, for example, it may produce more non-zero
   970:   // bits. While it would still be possible to represent the proper result
   971:   // in a wider type, it may require adding additional instructions (which
   972:   // we don't want to do).
   973:   Instruction *In = dyn_cast<Instruction>(Val);
   974:   if (!In)
   975:     return true;
   976:   // The bitwidth of the source type is smaller than the destination.
   977:   // Check if the individual operation can be promoted.
   978:   switch (In->getOpcode()) {
   979:     case Instruction::PHI:
   980:     case Instruction::ZExt:
   981:     case Instruction::And:
   982:     case Instruction::Or:
   983:     case Instruction::Xor:
   984:     case Instruction::LShr: // Shift right is ok.
   985:     case Instruction::Select:
   986:     case Instruction::Trunc:
   987:       return true;
   988:     case Instruction::ICmp:
   989:       if (CmpInst *CI = cast<CmpInst>(In))
   990:         return CI->isEquality() || CI->isUnsigned();
   991:       llvm_unreachable("Cast failed unexpectedly");
   992:     case Instruction::Add:
   993:       return In->hasNoSignedWrap() && In->hasNoUnsignedWrap();
   994:   }
   995:   return false;
   996: }
   997: 
   998: void PolynomialMultiplyRecognize::promoteTo(Instruction *In,
   999:       IntegerType *DestTy, BasicBlock *LoopB) {
  1000:   Type *OrigTy = In->getType();
```
- EN: It opens namespaces (PatternMatch) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as PolynomialMultiplyRecognize::scanSelect, phi, dyn_cast<PHINode>, dyn_cast<Instruction>, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里打开了命名空间（PatternMatch），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 PolynomialMultiplyRecognize::scanSelect, phi, dyn_cast<PHINode>, dyn_cast<Instruction>, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 1001-1200 / 第 1001-1200 行

```cpp
  1001:   assert(!OrigTy->isVoidTy() && "Invalid instruction to promote");
  1002: 
  1003:   // Leave boolean values alone.
  1004:   if (!In->getType()->isIntegerTy(1))
  1005:     In->mutateType(DestTy);
  1006:   unsigned DestBW = DestTy->getBitWidth();
  1007: 
  1008:   // Handle PHIs.
  1009:   if (PHINode *P = dyn_cast<PHINode>(In)) {
  1010:     unsigned N = P->getNumIncomingValues();
  1011:     for (unsigned i = 0; i != N; ++i) {
  1012:       BasicBlock *InB = P->getIncomingBlock(i);
  1013:       if (InB == LoopB)
  1014:         continue;
  1015:       Value *InV = P->getIncomingValue(i);
  1016:       IntegerType *Ty = cast<IntegerType>(InV->getType());
  1017:       // Do not promote values in PHI nodes of type i1.
  1018:       if (Ty != P->getType()) {
  1019:         // If the value type does not match the PHI type, the PHI type
  1020:         // must have been promoted.
  1021:         assert(Ty->getBitWidth() < DestBW);
  1022:         InV = IRBuilder<>(InB->getTerminator()).CreateZExt(InV, DestTy);
  1023:         P->setIncomingValue(i, InV);
  1024:       }
  1025:     }
  1026:   } else if (ZExtInst *Z = dyn_cast<ZExtInst>(In)) {
  1027:     Value *Op = Z->getOperand(0);
  1028:     if (Op->getType() == Z->getType())
  1029:       Z->replaceAllUsesWith(Op);
  1030:     Z->eraseFromParent();
  1031:     return;
  1032:   }
  1033:   if (TruncInst *T = dyn_cast<TruncInst>(In)) {
  1034:     IntegerType *TruncTy = cast<IntegerType>(OrigTy);
  1035:     Value *Mask = ConstantInt::get(DestTy, (1u << TruncTy->getBitWidth()) - 1);
  1036:     Value *And = IRBuilder<>(In).CreateAnd(T->getOperand(0), Mask);
  1037:     T->replaceAllUsesWith(And);
  1038:     T->eraseFromParent();
  1039:     return;
  1040:   }
  1041: 
  1042:   // Promote immediates.
  1043:   for (unsigned i = 0, n = In->getNumOperands(); i != n; ++i) {
  1044:     if (ConstantInt *CI = dyn_cast<ConstantInt>(In->getOperand(i)))
  1045:       if (CI->getBitWidth() < DestBW)
  1046:         In->setOperand(i, ConstantInt::get(DestTy, CI->getZExtValue()));
  1047:   }
  1048: }
  1049: 
  1050: bool PolynomialMultiplyRecognize::promoteTypes(BasicBlock *LoopB,
  1051:       BasicBlock *ExitB) {
  1052:   assert(LoopB);
  1053:   // Skip loops where the exit block has more than one predecessor. The values
  1054:   // coming from the loop block will be promoted to another type, and so the
  1055:   // values coming into the exit block from other predecessors would also have
  1056:   // to be promoted.
  1057:   if (!ExitB || (ExitB->getSinglePredecessor() != LoopB))
  1058:     return false;
  1059:   IntegerType *DestTy = getPmpyType();
  1060:   // Check if the exit values have types that are no wider than the type
  1061:   // that we want to promote to.
  1062:   unsigned DestBW = DestTy->getBitWidth();
  1063:   for (PHINode &P : ExitB->phis()) {
  1064:     if (P.getNumIncomingValues() != 1)
  1065:       return false;
  1066:     assert(P.getIncomingBlock(0) == LoopB);
  1067:     IntegerType *T = dyn_cast<IntegerType>(P.getType());
  1068:     if (!T || T->getBitWidth() > DestBW)
  1069:       return false;
  1070:   }
  1071: 
  1072:   // Check all instructions in the loop.
  1073:   for (Instruction &In : *LoopB)
  1074:     if (!In.isTerminator() && !isPromotableTo(&In, DestTy))
  1075:       return false;
  1076: 
  1077:   // Perform the promotion.
  1078:   SmallVector<Instruction *> LoopIns(llvm::make_pointer_range(*LoopB));
  1079:   for (Instruction *In : LoopIns)
  1080:     if (!In->isTerminator())
  1081:       promoteTo(In, DestTy, LoopB);
  1082: 
  1083:   // Fix up the PHI nodes in the exit block.
  1084:   BasicBlock::iterator End = ExitB->getFirstNonPHIIt();
  1085:   for (auto I = ExitB->begin(); I != End; ++I) {
  1086:     PHINode *P = dyn_cast<PHINode>(I);
  1087:     if (!P)
  1088:       break;
  1089:     Type *Ty0 = P->getIncomingValue(0)->getType();
  1090:     Type *PTy = P->getType();
  1091:     if (PTy != Ty0) {
  1092:       assert(Ty0 == DestTy);
  1093:       // In order to create the trunc, P must have the promoted type.
  1094:       P->mutateType(Ty0);
  1095:       Value *T = IRBuilder<>(ExitB, End).CreateTrunc(P, PTy);
  1096:       // In order for the RAUW to work, the types of P and T must match.
  1097:       P->mutateType(PTy);
  1098:       P->replaceAllUsesWith(T);
  1099:       // Final update of the P's type.
  1100:       P->mutateType(Ty0);
  1101:       cast<Instruction>(T)->setOperand(0, P);
  1102:     }
  1103:   }
  1104: 
  1105:   return true;
  1106: }
  1107: 
  1108: bool PolynomialMultiplyRecognize::findCycle(Value *Out, Value *In,
  1109:       ValueSeq &Cycle) {
  1110:   // Out = ..., In, ...
  1111:   if (Out == In)
  1112:     return true;
  1113: 
  1114:   auto *BB = cast<Instruction>(Out)->getParent();
  1115:   bool HadPhi = false;
  1116: 
  1117:   for (auto *U : Out->users()) {
  1118:     auto *I = dyn_cast<Instruction>(&*U);
  1119:     if (I == nullptr || I->getParent() != BB)
  1120:       continue;
  1121:     // Make sure that there are no multi-iteration cycles, e.g.
  1122:     //   p1 = phi(p2)
  1123:     //   p2 = phi(p1)
  1124:     // The cycle p1->p2->p1 would span two loop iterations.
  1125:     // Check that there is only one phi in the cycle.
  1126:     bool IsPhi = isa<PHINode>(I);
  1127:     if (IsPhi && HadPhi)
  1128:       return false;
  1129:     HadPhi |= IsPhi;
  1130:     if (!Cycle.insert(I))
  1131:       return false;
  1132:     if (findCycle(I, In, Cycle))
  1133:       break;
  1134:     Cycle.remove(I);
  1135:   }
  1136:   return !Cycle.empty();
  1137: }
  1138: 
  1139: void PolynomialMultiplyRecognize::classifyCycle(Instruction *DivI,
  1140:       ValueSeq &Cycle, ValueSeq &Early, ValueSeq &Late) {
  1141:   // All the values in the cycle that are between the phi node and the
  1142:   // divider instruction will be classified as "early", all other values
  1143:   // will be "late".
  1144: 
  1145:   bool IsE = true;
  1146:   unsigned I, N = Cycle.size();
  1147:   for (I = 0; I < N; ++I) {
  1148:     Value *V = Cycle[I];
  1149:     if (DivI == V)
  1150:       IsE = false;
  1151:     else if (!isa<PHINode>(V))
  1152:       continue;
  1153:     // Stop if found either.
  1154:     break;
  1155:   }
  1156:   // "I" is the index of either DivI or the phi node, whichever was first.
  1157:   // "E" is "false" or "true" respectively.
  1158:   ValueSeq &First = !IsE ? Early : Late;
  1159:   for (unsigned J = 0; J < I; ++J)
  1160:     First.insert(Cycle[J]);
  1161: 
  1162:   ValueSeq &Second = IsE ? Early : Late;
  1163:   Second.insert(Cycle[I]);
  1164:   for (++I; I < N; ++I) {
  1165:     Value *V = Cycle[I];
  1166:     if (DivI == V || isa<PHINode>(V))
  1167:       break;
  1168:     Second.insert(V);
  1169:   }
  1170: 
  1171:   for (; I < N; ++I)
  1172:     First.insert(Cycle[I]);
  1173: }
  1174: 
  1175: bool PolynomialMultiplyRecognize::classifyInst(Instruction *UseI,
  1176:       ValueSeq &Early, ValueSeq &Late) {
  1177:   // Select is an exception, since the condition value does not have to be
  1178:   // classified in the same way as the true/false values. The true/false
  1179:   // values do have to be both early or both late.
  1180:   if (UseI->getOpcode() == Instruction::Select) {
  1181:     Value *TV = UseI->getOperand(1), *FV = UseI->getOperand(2);
  1182:     if (Early.count(TV) || Early.count(FV)) {
  1183:       if (Late.count(TV) || Late.count(FV))
  1184:         return false;
  1185:       Early.insert(UseI);
  1186:     } else if (Late.count(TV) || Late.count(FV)) {
  1187:       if (Early.count(TV) || Early.count(FV))
  1188:         return false;
  1189:       Late.insert(UseI);
  1190:     }
  1191:     return true;
  1192:   }
  1193: 
  1194:   // Not sure what would be the example of this, but the code below relies
  1195:   // on having at least one operand.
  1196:   if (UseI->getNumOperands() == 0)
  1197:     return true;
  1198: 
  1199:   bool AE = true, AL = true;
  1200:   for (auto &I : UseI->operands()) {
```
- EN: It declares or implements routines such as assert, getBitWidth, getNumIncomingValues, getIncomingBlock, ... (30 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 assert, getBitWidth, getNumIncomingValues, getIncomingBlock, ... (30 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1201-1400 / 第 1201-1400 行

```cpp
  1201:     if (Early.count(&*I))
  1202:       AL = false;
  1203:     else if (Late.count(&*I))
  1204:       AE = false;
  1205:   }
  1206:   // If the operands appear "all early" and "all late" at the same time,
  1207:   // then it means that none of them are actually classified as either.
  1208:   // This is harmless.
  1209:   if (AE && AL)
  1210:     return true;
  1211:   // Conversely, if they are neither "all early" nor "all late", then
  1212:   // we have a mixture of early and late operands that is not a known
  1213:   // exception.
  1214:   if (!AE && !AL)
  1215:     return false;
  1216: 
  1217:   // Check that we have covered the two special cases.
  1218:   assert(AE != AL);
  1219: 
  1220:   if (AE)
  1221:     Early.insert(UseI);
  1222:   else
  1223:     Late.insert(UseI);
  1224:   return true;
  1225: }
  1226: 
  1227: bool PolynomialMultiplyRecognize::commutesWithShift(Instruction *I) {
  1228:   switch (I->getOpcode()) {
  1229:     case Instruction::And:
  1230:     case Instruction::Or:
  1231:     case Instruction::Xor:
  1232:     case Instruction::LShr:
  1233:     case Instruction::Shl:
  1234:     case Instruction::Select:
  1235:     case Instruction::ICmp:
  1236:     case Instruction::PHI:
  1237:       break;
  1238:     default:
  1239:       return false;
  1240:   }
  1241:   return true;
  1242: }
  1243: 
  1244: bool PolynomialMultiplyRecognize::highBitsAreZero(Value *V,
  1245:       unsigned IterCount) {
  1246:   auto *T = dyn_cast<IntegerType>(V->getType());
  1247:   if (!T)
  1248:     return false;
  1249: 
  1250:   KnownBits Known(T->getBitWidth());
  1251:   computeKnownBits(V, Known, DL);
  1252:   return Known.countMinLeadingZeros() >= IterCount;
  1253: }
  1254: 
  1255: bool PolynomialMultiplyRecognize::keepsHighBitsZero(Value *V,
  1256:       unsigned IterCount) {
  1257:   // Assume that all inputs to the value have the high bits zero.
  1258:   // Check if the value itself preserves the zeros in the high bits.
  1259:   if (auto *C = dyn_cast<ConstantInt>(V))
  1260:     return C->getValue().countl_zero() >= IterCount;
  1261: 
  1262:   if (auto *I = dyn_cast<Instruction>(V)) {
  1263:     switch (I->getOpcode()) {
  1264:       case Instruction::And:
  1265:       case Instruction::Or:
  1266:       case Instruction::Xor:
  1267:       case Instruction::LShr:
  1268:       case Instruction::Select:
  1269:       case Instruction::ICmp:
  1270:       case Instruction::PHI:
  1271:       case Instruction::ZExt:
  1272:         return true;
  1273:     }
  1274:   }
  1275: 
  1276:   return false;
  1277: }
  1278: 
  1279: bool PolynomialMultiplyRecognize::isOperandShifted(Instruction *I, Value *Op) {
  1280:   unsigned Opc = I->getOpcode();
  1281:   if (Opc == Instruction::Shl || Opc == Instruction::LShr)
  1282:     return Op != I->getOperand(1);
  1283:   return true;
  1284: }
  1285: 
  1286: bool PolynomialMultiplyRecognize::convertShiftsToLeft(BasicBlock *LoopB,
  1287:       BasicBlock *ExitB, unsigned IterCount) {
  1288:   Value *CIV = getCountIV(LoopB);
  1289:   if (CIV == nullptr)
  1290:     return false;
  1291:   auto *CIVTy = dyn_cast<IntegerType>(CIV->getType());
  1292:   if (CIVTy == nullptr)
  1293:     return false;
  1294: 
  1295:   ValueSeq RShifts;
  1296:   ValueSeq Early, Late, Cycled;
  1297: 
  1298:   // Find all value cycles that contain logical right shifts by 1.
  1299:   for (Instruction &I : *LoopB) {
  1300:     using namespace PatternMatch;
  1301: 
  1302:     Value *V = nullptr;
  1303:     if (!match(&I, m_LShr(m_Value(V), m_One())))
  1304:       continue;
  1305:     ValueSeq C;
  1306:     if (!findCycle(&I, V, C))
  1307:       continue;
  1308: 
  1309:     // Found a cycle.
  1310:     C.insert(&I);
  1311:     classifyCycle(&I, C, Early, Late);
  1312:     Cycled.insert_range(C);
  1313:     RShifts.insert(&I);
  1314:   }
  1315: 
  1316:   // Find the set of all values affected by the shift cycles, i.e. all
  1317:   // cycled values, and (recursively) all their users.
  1318:   ValueSeq Users(llvm::from_range, Cycled);
  1319:   for (unsigned i = 0; i < Users.size(); ++i) {
  1320:     Value *V = Users[i];
  1321:     if (!isa<IntegerType>(V->getType()))
  1322:       return false;
  1323:     auto *R = cast<Instruction>(V);
  1324:     // If the instruction does not commute with shifts, the loop cannot
  1325:     // be unshifted.
  1326:     if (!commutesWithShift(R))
  1327:       return false;
  1328:     for (User *U : R->users()) {
  1329:       auto *T = cast<Instruction>(U);
  1330:       // Skip users from outside of the loop. They will be handled later.
  1331:       // Also, skip the right-shifts and phi nodes, since they mix early
  1332:       // and late values.
  1333:       if (T->getParent() != LoopB || RShifts.count(T) || isa<PHINode>(T))
  1334:         continue;
  1335: 
  1336:       Users.insert(T);
  1337:       if (!classifyInst(T, Early, Late))
  1338:         return false;
  1339:     }
  1340:   }
  1341: 
  1342:   if (Users.empty())
  1343:     return false;
  1344: 
  1345:   // Verify that high bits remain zero.
  1346:   ValueSeq Internal(llvm::from_range, Users);
  1347:   ValueSeq Inputs;
  1348:   for (unsigned i = 0; i < Internal.size(); ++i) {
  1349:     auto *R = dyn_cast<Instruction>(Internal[i]);
  1350:     if (!R)
  1351:       continue;
  1352:     for (Value *Op : R->operands()) {
  1353:       auto *T = dyn_cast<Instruction>(Op);
  1354:       if (T && T->getParent() != LoopB)
  1355:         Inputs.insert(Op);
  1356:       else
  1357:         Internal.insert(Op);
  1358:     }
  1359:   }
  1360:   for (Value *V : Inputs)
  1361:     if (!highBitsAreZero(V, IterCount))
  1362:       return false;
  1363:   for (Value *V : Internal)
  1364:     if (!keepsHighBitsZero(V, IterCount))
  1365:       return false;
  1366: 
  1367:   // Finally, the work can be done. Unshift each user.
  1368:   IRBuilder<> IRB(LoopB);
  1369:   std::map<Value*,Value*> ShiftMap;
  1370: 
  1371:   using CastMapType = std::map<std::pair<Value *, Type *>, Value *>;
  1372: 
  1373:   CastMapType CastMap;
  1374: 
  1375:   auto upcast = [](CastMapType &CM, IRBuilder<> &IRB, Value *V,
  1376:                    IntegerType *Ty) -> Value * {
  1377:     auto [H, Inserted] = CM.try_emplace(std::make_pair(V, Ty));
  1378:     if (Inserted)
  1379:       H->second = IRB.CreateIntCast(V, Ty, false);
  1380:     return H->second;
  1381:   };
  1382: 
  1383:   for (auto I = LoopB->begin(), E = LoopB->end(); I != E; ++I) {
  1384:     using namespace PatternMatch;
  1385: 
  1386:     if (isa<PHINode>(I) || !Users.count(&*I))
  1387:       continue;
  1388: 
  1389:     // Match lshr x, 1.
  1390:     Value *V = nullptr;
  1391:     if (match(&*I, m_LShr(m_Value(V), m_One()))) {
  1392:       replaceAllUsesOfWithIn(&*I, V, LoopB);
  1393:       continue;
  1394:     }
  1395:     // For each non-cycled operand, replace it with the corresponding
  1396:     // value shifted left.
  1397:     for (auto &J : I->operands()) {
  1398:       Value *Op = J.get();
  1399:       if (!isOperandShifted(&*I, Op))
  1400:         continue;
```
- EN: It opens namespaces (PatternMatch) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as assert, insert, PolynomialMultiplyRecognize::commutesWithShift, PolynomialMultiplyRecognize::highBitsAreZero, ... (23 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（PatternMatch），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 assert, insert, PolynomialMultiplyRecognize::commutesWithShift, PolynomialMultiplyRecognize::highBitsAreZero, ... (23 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1401-1600 / 第 1401-1600 行

```cpp
  1401:       if (Users.count(Op))
  1402:         continue;
  1403:       // Skip shifting zeros.
  1404:       if (isa<ConstantInt>(Op) && cast<ConstantInt>(Op)->isZero())
  1405:         continue;
  1406:       // Check if we have already generated a shift for this value.
  1407:       auto F = ShiftMap.find(Op);
  1408:       Value *W = (F != ShiftMap.end()) ? F->second : nullptr;
  1409:       if (W == nullptr) {
  1410:         IRB.SetInsertPoint(&*I);
  1411:         // First, the shift amount will be CIV or CIV+1, depending on
  1412:         // whether the value is early or late. Instead of creating CIV+1,
  1413:         // do a single shift of the value.
  1414:         Value *ShAmt = CIV, *ShVal = Op;
  1415:         auto *VTy = cast<IntegerType>(ShVal->getType());
  1416:         auto *ATy = cast<IntegerType>(ShAmt->getType());
  1417:         if (Late.count(&*I))
  1418:           ShVal = IRB.CreateShl(Op, ConstantInt::get(VTy, 1));
  1419:         // Second, the types of the shifted value and the shift amount
  1420:         // must match.
  1421:         if (VTy != ATy) {
  1422:           if (VTy->getBitWidth() < ATy->getBitWidth())
  1423:             ShVal = upcast(CastMap, IRB, ShVal, ATy);
  1424:           else
  1425:             ShAmt = upcast(CastMap, IRB, ShAmt, VTy);
  1426:         }
  1427:         // Ready to generate the shift and memoize it.
  1428:         W = IRB.CreateShl(ShVal, ShAmt);
  1429:         ShiftMap.insert(std::make_pair(Op, W));
  1430:       }
  1431:       I->replaceUsesOfWith(Op, W);
  1432:     }
  1433:   }
  1434: 
  1435:   // Update the users outside of the loop to account for having left
  1436:   // shifts. They would normally be shifted right in the loop, so shift
  1437:   // them right after the loop exit.
  1438:   // Take advantage of the loop-closed SSA form, which has all the post-
  1439:   // loop values in phi nodes.
  1440:   IRB.SetInsertPoint(ExitB, ExitB->getFirstInsertionPt());
  1441:   for (auto P = ExitB->begin(), Q = ExitB->end(); P != Q; ++P) {
  1442:     if (!isa<PHINode>(P))
  1443:       break;
  1444:     auto *PN = cast<PHINode>(P);
  1445:     Value *U = PN->getIncomingValueForBlock(LoopB);
  1446:     if (!Users.count(U))
  1447:       continue;
  1448:     Value *S = IRB.CreateLShr(PN, ConstantInt::get(PN->getType(), IterCount));
  1449:     PN->replaceAllUsesWith(S);
  1450:     // The above RAUW will create
  1451:     //   S = lshr S, IterCount
  1452:     // so we need to fix it back into
  1453:     //   S = lshr PN, IterCount
  1454:     cast<User>(S)->replaceUsesOfWith(S, PN);
  1455:   }
  1456: 
  1457:   return true;
  1458: }
  1459: 
  1460: void PolynomialMultiplyRecognize::cleanupLoopBody(BasicBlock *LoopB) {
  1461:   for (auto &I : *LoopB)
  1462:     if (Value *SV = simplifyInstruction(&I, {DL, &TLI, &DT}))
  1463:       I.replaceAllUsesWith(SV);
  1464: 
  1465:   for (Instruction &I : llvm::make_early_inc_range(*LoopB))
  1466:     RecursivelyDeleteTriviallyDeadInstructions(&I, &TLI);
  1467: }
  1468: 
  1469: unsigned PolynomialMultiplyRecognize::getInverseMxN(unsigned QP) {
  1470:   // Arrays of coefficients of Q and the inverse, C.
  1471:   // Q[i] = coefficient at x^i.
  1472:   std::array<char,32> Q, C;
  1473: 
  1474:   for (unsigned i = 0; i < 32; ++i) {
  1475:     Q[i] = QP & 1;
  1476:     QP >>= 1;
  1477:   }
  1478:   assert(Q[0] == 1);
  1479: 
  1480:   // Find C, such that
  1481:   // (Q[n]*x^n + ... + Q[1]*x + Q[0]) * (C[n]*x^n + ... + C[1]*x + C[0]) = 1
  1482:   //
  1483:   // For it to have a solution, Q[0] must be 1. Since this is Z2[x], the
  1484:   // operations * and + are & and ^ respectively.
  1485:   //
  1486:   // Find C[i] recursively, by comparing i-th coefficient in the product
  1487:   // with 0 (or 1 for i=0).
  1488:   //
  1489:   // C[0] = 1, since C[0] = Q[0], and Q[0] = 1.
  1490:   C[0] = 1;
  1491:   for (unsigned i = 1; i < 32; ++i) {
  1492:     // Solve for C[i] in:
  1493:     //   C[0]Q[i] ^ C[1]Q[i-1] ^ ... ^ C[i-1]Q[1] ^ C[i]Q[0] = 0
  1494:     // This is equivalent to
  1495:     //   C[0]Q[i] ^ C[1]Q[i-1] ^ ... ^ C[i-1]Q[1] ^ C[i] = 0
  1496:     // which is
  1497:     //   C[0]Q[i] ^ C[1]Q[i-1] ^ ... ^ C[i-1]Q[1] = C[i]
  1498:     unsigned T = 0;
  1499:     for (unsigned j = 0; j < i; ++j)
  1500:       T = T ^ (C[j] & Q[i-j]);
  1501:     C[i] = T;
  1502:   }
  1503: 
  1504:   unsigned QV = 0;
  1505:   for (unsigned i = 0; i < 32; ++i)
  1506:     if (C[i])
  1507:       QV |= (1 << i);
  1508: 
  1509:   return QV;
  1510: }
  1511: 
  1512: Value *PolynomialMultiplyRecognize::generate(BasicBlock::iterator At,
  1513:       ParsedValues &PV) {
  1514:   IRBuilder<> B(&*At);
  1515:   Module *M = At->getParent()->getParent()->getParent();
  1516:   Function *PMF =
  1517:       Intrinsic::getOrInsertDeclaration(M, Intrinsic::hexagon_M4_pmpyw);
  1518: 
  1519:   Value *P = PV.P, *Q = PV.Q, *P0 = P;
  1520:   unsigned IC = PV.IterCount;
  1521: 
  1522:   if (PV.M != nullptr)
  1523:     P0 = P = B.CreateXor(P, PV.M);
  1524: 
  1525:   // Create a bit mask to clear the high bits beyond IterCount.
  1526:   auto *BMI = ConstantInt::get(P->getType(), APInt::getLowBitsSet(32, IC));
  1527: 
  1528:   if (PV.IterCount != 32)
  1529:     P = B.CreateAnd(P, BMI);
  1530: 
  1531:   if (PV.Inv) {
  1532:     auto *QI = dyn_cast<ConstantInt>(PV.Q);
  1533:     assert(QI && QI->getBitWidth() <= 32);
  1534: 
  1535:     // Again, clearing bits beyond IterCount.
  1536:     unsigned M = (1 << PV.IterCount) - 1;
  1537:     unsigned Tmp = (QI->getZExtValue() | 1) & M;
  1538:     unsigned QV = getInverseMxN(Tmp) & M;
  1539:     auto *QVI = ConstantInt::get(QI->getType(), QV);
  1540:     P = B.CreateCall(PMF, {P, QVI});
  1541:     P = B.CreateTrunc(P, QI->getType());
  1542:     if (IC != 32)
  1543:       P = B.CreateAnd(P, BMI);
  1544:   }
  1545: 
  1546:   Value *R = B.CreateCall(PMF, {P, Q});
  1547: 
  1548:   if (PV.M != nullptr)
  1549:     R = B.CreateXor(R, B.CreateIntCast(P0, R->getType(), false));
  1550: 
  1551:   return R;
  1552: }
  1553: 
  1554: static bool hasZeroSignBit(const Value *V) {
  1555:   if (const auto *CI = dyn_cast<const ConstantInt>(V))
  1556:     return CI->getValue().isNonNegative();
  1557:   const Instruction *I = dyn_cast<const Instruction>(V);
  1558:   if (!I)
  1559:     return false;
  1560:   switch (I->getOpcode()) {
  1561:     case Instruction::LShr:
  1562:       if (const auto SI = dyn_cast<const ConstantInt>(I->getOperand(1)))
  1563:         return SI->getZExtValue() > 0;
  1564:       return false;
  1565:     case Instruction::Or:
  1566:     case Instruction::Xor:
  1567:       return hasZeroSignBit(I->getOperand(0)) &&
  1568:              hasZeroSignBit(I->getOperand(1));
  1569:     case Instruction::And:
  1570:       return hasZeroSignBit(I->getOperand(0)) ||
  1571:              hasZeroSignBit(I->getOperand(1));
  1572:   }
  1573:   return false;
  1574: }
  1575: 
  1576: void PolynomialMultiplyRecognize::setupPreSimplifier(Simplifier &S) {
  1577:   S.addRule("sink-zext",
  1578:     // Sink zext past bitwise operations.
  1579:     [](Instruction *I, LLVMContext &Ctx) -> Value* {
  1580:       if (I->getOpcode() != Instruction::ZExt)
  1581:         return nullptr;
  1582:       Instruction *T = dyn_cast<Instruction>(I->getOperand(0));
  1583:       if (!T)
  1584:         return nullptr;
  1585:       switch (T->getOpcode()) {
  1586:         case Instruction::And:
  1587:         case Instruction::Or:
  1588:         case Instruction::Xor:
  1589:           break;
  1590:         default:
  1591:           return nullptr;
  1592:       }
  1593:       IRBuilder<> B(Ctx);
  1594:       return B.CreateBinOp(cast<BinaryOperator>(T)->getOpcode(),
  1595:                            B.CreateZExt(T->getOperand(0), I->getType()),
  1596:                            B.CreateZExt(T->getOperand(1), I->getType()));
  1597:     });
  1598:   S.addRule("xor/and -> and/xor",
  1599:     // (xor (and x a) (and y a)) -> (and (xor x y) a)
  1600:     [](Instruction *I, LLVMContext &Ctx) -> Value* {
```
- EN: It declares or implements routines such as find, SetInsertPoint, cast<IntegerType>, upcast, ... (27 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 find, SetInsertPoint, cast<IntegerType>, upcast, ... (27 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1601-1800 / 第 1601-1800 行

```cpp
  1601:       if (I->getOpcode() != Instruction::Xor)
  1602:         return nullptr;
  1603:       Instruction *And0 = dyn_cast<Instruction>(I->getOperand(0));
  1604:       Instruction *And1 = dyn_cast<Instruction>(I->getOperand(1));
  1605:       if (!And0 || !And1)
  1606:         return nullptr;
  1607:       if (And0->getOpcode() != Instruction::And ||
  1608:           And1->getOpcode() != Instruction::And)
  1609:         return nullptr;
  1610:       if (And0->getOperand(1) != And1->getOperand(1))
  1611:         return nullptr;
  1612:       IRBuilder<> B(Ctx);
  1613:       return B.CreateAnd(B.CreateXor(And0->getOperand(0), And1->getOperand(0)),
  1614:                          And0->getOperand(1));
  1615:     });
  1616:   S.addRule("sink binop into select",
  1617:     // (Op (select c x y) z) -> (select c (Op x z) (Op y z))
  1618:     // (Op x (select c y z)) -> (select c (Op x y) (Op x z))
  1619:     [](Instruction *I, LLVMContext &Ctx) -> Value* {
  1620:       BinaryOperator *BO = dyn_cast<BinaryOperator>(I);
  1621:       if (!BO)
  1622:         return nullptr;
  1623:       Instruction::BinaryOps Op = BO->getOpcode();
  1624:       if (SelectInst *Sel = dyn_cast<SelectInst>(BO->getOperand(0))) {
  1625:         IRBuilder<> B(Ctx);
  1626:         Value *X = Sel->getTrueValue(), *Y = Sel->getFalseValue();
  1627:         Value *Z = BO->getOperand(1);
  1628:         return B.CreateSelect(Sel->getCondition(),
  1629:                               B.CreateBinOp(Op, X, Z),
  1630:                               B.CreateBinOp(Op, Y, Z));
  1631:       }
  1632:       if (SelectInst *Sel = dyn_cast<SelectInst>(BO->getOperand(1))) {
  1633:         IRBuilder<> B(Ctx);
  1634:         Value *X = BO->getOperand(0);
  1635:         Value *Y = Sel->getTrueValue(), *Z = Sel->getFalseValue();
  1636:         return B.CreateSelect(Sel->getCondition(),
  1637:                               B.CreateBinOp(Op, X, Y),
  1638:                               B.CreateBinOp(Op, X, Z));
  1639:       }
  1640:       return nullptr;
  1641:     });
  1642:   S.addRule("fold select-select",
  1643:     // (select c (select c x y) z) -> (select c x z)
  1644:     // (select c x (select c y z)) -> (select c x z)
  1645:     [](Instruction *I, LLVMContext &Ctx) -> Value* {
  1646:       SelectInst *Sel = dyn_cast<SelectInst>(I);
  1647:       if (!Sel)
  1648:         return nullptr;
  1649:       IRBuilder<> B(Ctx);
  1650:       Value *C = Sel->getCondition();
  1651:       if (SelectInst *Sel0 = dyn_cast<SelectInst>(Sel->getTrueValue())) {
  1652:         if (Sel0->getCondition() == C)
  1653:           return B.CreateSelect(C, Sel0->getTrueValue(), Sel->getFalseValue());
  1654:       }
  1655:       if (SelectInst *Sel1 = dyn_cast<SelectInst>(Sel->getFalseValue())) {
  1656:         if (Sel1->getCondition() == C)
  1657:           return B.CreateSelect(C, Sel->getTrueValue(), Sel1->getFalseValue());
  1658:       }
  1659:       return nullptr;
  1660:     });
  1661:   S.addRule("or-signbit -> xor-signbit",
  1662:     // (or (lshr x 1) 0x800.0) -> (xor (lshr x 1) 0x800.0)
  1663:     [](Instruction *I, LLVMContext &Ctx) -> Value* {
  1664:       if (I->getOpcode() != Instruction::Or)
  1665:         return nullptr;
  1666:       ConstantInt *Msb = dyn_cast<ConstantInt>(I->getOperand(1));
  1667:       if (!Msb || !Msb->getValue().isSignMask())
  1668:         return nullptr;
  1669:       if (!hasZeroSignBit(I->getOperand(0)))
  1670:         return nullptr;
  1671:       return IRBuilder<>(Ctx).CreateXor(I->getOperand(0), Msb);
  1672:     });
  1673:   S.addRule("sink lshr into binop",
  1674:     // (lshr (BitOp x y) c) -> (BitOp (lshr x c) (lshr y c))
  1675:     [](Instruction *I, LLVMContext &Ctx) -> Value* {
  1676:       if (I->getOpcode() != Instruction::LShr)
  1677:         return nullptr;
  1678:       BinaryOperator *BitOp = dyn_cast<BinaryOperator>(I->getOperand(0));
  1679:       if (!BitOp)
  1680:         return nullptr;
  1681:       switch (BitOp->getOpcode()) {
  1682:         case Instruction::And:
  1683:         case Instruction::Or:
  1684:         case Instruction::Xor:
  1685:           break;
  1686:         default:
  1687:           return nullptr;
  1688:       }
  1689:       IRBuilder<> B(Ctx);
  1690:       Value *S = I->getOperand(1);
  1691:       return B.CreateBinOp(BitOp->getOpcode(),
  1692:                 B.CreateLShr(BitOp->getOperand(0), S),
  1693:                 B.CreateLShr(BitOp->getOperand(1), S));
  1694:     });
  1695:   S.addRule("expose bitop-const",
  1696:     // (BitOp1 (BitOp2 x a) b) -> (BitOp2 x (BitOp1 a b))
  1697:     [](Instruction *I, LLVMContext &Ctx) -> Value* {
  1698:       auto IsBitOp = [](unsigned Op) -> bool {
  1699:         switch (Op) {
  1700:           case Instruction::And:
  1701:           case Instruction::Or:
  1702:           case Instruction::Xor:
  1703:             return true;
  1704:         }
  1705:         return false;
  1706:       };
  1707:       BinaryOperator *BitOp1 = dyn_cast<BinaryOperator>(I);
  1708:       if (!BitOp1 || !IsBitOp(BitOp1->getOpcode()))
  1709:         return nullptr;
  1710:       BinaryOperator *BitOp2 = dyn_cast<BinaryOperator>(BitOp1->getOperand(0));
  1711:       if (!BitOp2 || !IsBitOp(BitOp2->getOpcode()))
  1712:         return nullptr;
  1713:       ConstantInt *CA = dyn_cast<ConstantInt>(BitOp2->getOperand(1));
  1714:       ConstantInt *CB = dyn_cast<ConstantInt>(BitOp1->getOperand(1));
  1715:       if (!CA || !CB)
  1716:         return nullptr;
  1717:       IRBuilder<> B(Ctx);
  1718:       Value *X = BitOp2->getOperand(0);
  1719:       return B.CreateBinOp(BitOp2->getOpcode(), X,
  1720:                 B.CreateBinOp(BitOp1->getOpcode(), CA, CB));
  1721:     });
  1722:   S.addRule("select with trunc cond to select with icmp cond",
  1723:             // select (trunc x to i1) -> select (icmp ne (and x, 1), 0)
  1724:             // select (xor (trunc x to i1) 1) -> select (icmp eq (and x, 1), 0)
  1725:             [](Instruction *I, LLVMContext &Ctx) -> Value * {
  1726:               SelectInst *Sel = dyn_cast<SelectInst>(I);
  1727:               if (!Sel)
  1728:                 return nullptr;
  1729:               Value *C = Sel->getCondition();
  1730:               Value *X;
  1731:               using namespace PatternMatch;
  1732:               if (!(match(C, m_Trunc(m_Value(X))) ||
  1733:                     match(C, m_Not(m_Trunc(m_Value(X))))))
  1734:                 return nullptr;
  1735: 
  1736:               IRBuilder<> B(Ctx);
  1737:               Type *Ty = X->getType();
  1738:               Value *And = B.CreateAnd(X, ConstantInt::get(Ty, 1));
  1739:               Value *Icmp = B.CreateICmp(isa<TruncInst>(C) ? ICmpInst::ICMP_NE
  1740:                                                            : ICmpInst::ICMP_EQ,
  1741:                                          And, ConstantInt::get(Ty, 0));
  1742:               return B.CreateSelect(Icmp, Sel->getTrueValue(),
  1743:                                     Sel->getFalseValue());
  1744:             });
  1745: }
  1746: 
  1747: void PolynomialMultiplyRecognize::setupPostSimplifier(Simplifier &S) {
  1748:   S.addRule("(and (xor (and x a) y) b) -> (and (xor x y) b), if b == b&a",
  1749:     [](Instruction *I, LLVMContext &Ctx) -> Value* {
  1750:       if (I->getOpcode() != Instruction::And)
  1751:         return nullptr;
  1752:       Instruction *Xor = dyn_cast<Instruction>(I->getOperand(0));
  1753:       ConstantInt *C0 = dyn_cast<ConstantInt>(I->getOperand(1));
  1754:       if (!Xor || !C0)
  1755:         return nullptr;
  1756:       if (Xor->getOpcode() != Instruction::Xor)
  1757:         return nullptr;
  1758:       Instruction *And0 = dyn_cast<Instruction>(Xor->getOperand(0));
  1759:       Instruction *And1 = dyn_cast<Instruction>(Xor->getOperand(1));
  1760:       // Pick the first non-null and.
  1761:       if (!And0 || And0->getOpcode() != Instruction::And)
  1762:         std::swap(And0, And1);
  1763:       ConstantInt *C1 = dyn_cast<ConstantInt>(And0->getOperand(1));
  1764:       if (!C1)
  1765:         return nullptr;
  1766:       uint32_t V0 = C0->getZExtValue();
  1767:       uint32_t V1 = C1->getZExtValue();
  1768:       if (V0 != (V0 & V1))
  1769:         return nullptr;
  1770:       IRBuilder<> B(Ctx);
  1771:       return B.CreateAnd(B.CreateXor(And0->getOperand(0), And1), C0);
  1772:     });
  1773: }
  1774: 
  1775: bool PolynomialMultiplyRecognize::recognize() {
  1776:   LLVM_DEBUG(dbgs() << "Starting PolynomialMultiplyRecognize on loop\n"
  1777:                     << *CurLoop << '\n');
  1778:   // Restrictions:
  1779:   // - The loop must consist of a single block.
  1780:   // - The iteration count must be known at compile-time.
  1781:   // - The loop must have an induction variable starting from 0, and
  1782:   //   incremented in each iteration of the loop.
  1783:   BasicBlock *LoopB = CurLoop->getHeader();
  1784:   LLVM_DEBUG(dbgs() << "Loop header:\n" << *LoopB);
  1785: 
  1786:   if (LoopB != CurLoop->getLoopLatch())
  1787:     return false;
  1788:   BasicBlock *ExitB = CurLoop->getExitBlock();
  1789:   if (ExitB == nullptr)
  1790:     return false;
  1791:   BasicBlock *EntryB = CurLoop->getLoopPreheader();
  1792:   if (EntryB == nullptr)
  1793:     return false;
  1794: 
  1795:   unsigned IterCount = 0;
  1796:   const SCEV *CT = SE.getBackedgeTakenCount(CurLoop);
  1797:   if (isa<SCEVCouldNotCompute>(CT))
  1798:     return false;
  1799:   if (auto *CV = dyn_cast<SCEVConstant>(CT))
  1800:     IterCount = CV->getValue()->getZExtValue() + 1;
```
- EN: It opens namespaces (PatternMatch) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as dyn_cast<Instruction>, B, CreateAnd, dyn_cast<BinaryOperator>, ... (23 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里打开了命名空间（PatternMatch），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 dyn_cast<Instruction>, B, CreateAnd, dyn_cast<BinaryOperator>, ... (23 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 1801-2000 / 第 1801-2000 行

```cpp
  1801: 
  1802:   Value *CIV = getCountIV(LoopB);
  1803:   if (CIV == nullptr)
  1804:     return false;
  1805:   ParsedValues PV;
  1806:   Simplifier PreSimp;
  1807:   PV.IterCount = IterCount;
  1808:   LLVM_DEBUG(dbgs() << "Loop IV: " << *CIV << "\nIterCount: " << IterCount
  1809:                     << '\n');
  1810: 
  1811:   setupPreSimplifier(PreSimp);
  1812: 
  1813:   // Perform a preliminary scan of select instructions to see if any of them
  1814:   // looks like a generator of the polynomial multiply steps. Assume that a
  1815:   // loop can only contain a single transformable operation, so stop the
  1816:   // traversal after the first reasonable candidate was found.
  1817:   // XXX: Currently this approach can modify the loop before being 100% sure
  1818:   // that the transformation can be carried out.
  1819:   bool FoundPreScan = false;
  1820:   auto FeedsPHI = [LoopB](const Value *V) -> bool {
  1821:     for (const Value *U : V->users()) {
  1822:       if (const auto *P = dyn_cast<const PHINode>(U))
  1823:         if (P->getParent() == LoopB)
  1824:           return true;
  1825:     }
  1826:     return false;
  1827:   };
  1828:   for (Instruction &In : *LoopB) {
  1829:     SelectInst *SI = dyn_cast<SelectInst>(&In);
  1830:     if (!SI || !FeedsPHI(SI))
  1831:       continue;
  1832: 
  1833:     Simplifier::Context C(SI);
  1834:     Value *T = PreSimp.simplify(C);
  1835:     SelectInst *SelI = (T && isa<SelectInst>(T)) ? cast<SelectInst>(T) : SI;
  1836:     LLVM_DEBUG(dbgs() << "scanSelect(pre-scan): " << PE(C, SelI) << '\n');
  1837:     if (scanSelect(SelI, LoopB, EntryB, CIV, PV, true)) {
  1838:       FoundPreScan = true;
  1839:       if (SelI != SI) {
  1840:         Value *NewSel = C.materialize(LoopB, SI->getIterator());
  1841:         SI->replaceAllUsesWith(NewSel);
  1842:         RecursivelyDeleteTriviallyDeadInstructions(SI, &TLI);
  1843:       }
  1844:       break;
  1845:     }
  1846:   }
  1847: 
  1848:   if (!FoundPreScan) {
  1849:     LLVM_DEBUG(dbgs() << "Have not found candidates for pmpy\n");
  1850:     return false;
  1851:   }
  1852: 
  1853:   if (!PV.Left) {
  1854:     // The right shift version actually only returns the higher bits of
  1855:     // the result (each iteration discards the LSB). If we want to convert it
  1856:     // to a left-shifting loop, the working data type must be at least as
  1857:     // wide as the target's pmpy instruction.
  1858:     if (!promoteTypes(LoopB, ExitB))
  1859:       return false;
  1860:     // Run post-promotion simplifications.
  1861:     Simplifier PostSimp;
  1862:     setupPostSimplifier(PostSimp);
  1863:     for (Instruction &In : *LoopB) {
  1864:       SelectInst *SI = dyn_cast<SelectInst>(&In);
  1865:       if (!SI || !FeedsPHI(SI))
  1866:         continue;
  1867:       Simplifier::Context C(SI);
  1868:       Value *T = PostSimp.simplify(C);
  1869:       SelectInst *SelI = dyn_cast_or_null<SelectInst>(T);
  1870:       if (SelI != SI) {
  1871:         Value *NewSel = C.materialize(LoopB, SI->getIterator());
  1872:         SI->replaceAllUsesWith(NewSel);
  1873:         RecursivelyDeleteTriviallyDeadInstructions(SI, &TLI);
  1874:       }
  1875:       break;
  1876:     }
  1877: 
  1878:     if (!convertShiftsToLeft(LoopB, ExitB, IterCount))
  1879:       return false;
  1880:     cleanupLoopBody(LoopB);
  1881:   }
  1882: 
  1883:   // Scan the loop again, find the generating select instruction.
  1884:   bool FoundScan = false;
  1885:   for (Instruction &In : *LoopB) {
  1886:     SelectInst *SelI = dyn_cast<SelectInst>(&In);
  1887:     if (!SelI)
  1888:       continue;
  1889:     LLVM_DEBUG(dbgs() << "scanSelect: " << *SelI << '\n');
  1890:     FoundScan = scanSelect(SelI, LoopB, EntryB, CIV, PV, false);
  1891:     if (FoundScan)
  1892:       break;
  1893:   }
  1894:   assert(FoundScan);
  1895: 
  1896:   LLVM_DEBUG({
  1897:     StringRef PP = (PV.M ? "(P+M)" : "P");
  1898:     if (!PV.Inv)
  1899:       dbgs() << "Found pmpy idiom: R = " << PP << ".Q\n";
  1900:     else
  1901:       dbgs() << "Found inverse pmpy idiom: R = (" << PP << "/Q).Q) + "
  1902:              << PP << "\n";
  1903:     dbgs() << "  Res:" << *PV.Res << "\n  P:" << *PV.P << "\n";
  1904:     if (PV.M)
  1905:       dbgs() << "  M:" << *PV.M << "\n";
  1906:     dbgs() << "  Q:" << *PV.Q << "\n";
  1907:     dbgs() << "  Iteration count:" << PV.IterCount << "\n";
  1908:   });
  1909: 
  1910:   BasicBlock::iterator At(EntryB->getTerminator());
  1911:   Value *PM = generate(At, PV);
  1912:   if (PM == nullptr)
  1913:     return false;
  1914: 
  1915:   if (PM->getType() != PV.Res->getType())
  1916:     PM = IRBuilder<>(&*At).CreateIntCast(PM, PV.Res->getType(), false);
  1917: 
  1918:   PV.Res->replaceAllUsesWith(PM);
  1919:   PV.Res->eraseFromParent();
  1920:   return true;
  1921: }
  1922: 
  1923: int HexagonLoopIdiomRecognize::getSCEVStride(const SCEVAddRecExpr *S) {
  1924:   if (const SCEVConstant *SC = dyn_cast<SCEVConstant>(S->getOperand(1)))
  1925:     return SC->getAPInt().getSExtValue();
  1926:   return 0;
  1927: }
  1928: 
  1929: bool HexagonLoopIdiomRecognize::isLegalStore(Loop *CurLoop, StoreInst *SI) {
  1930:   // Allow volatile stores if HexagonVolatileMemcpy is enabled.
  1931:   if (!(SI->isVolatile() && HexagonVolatileMemcpy) && !SI->isSimple())
  1932:     return false;
  1933: 
  1934:   Value *StoredVal = SI->getValueOperand();
  1935:   Value *StorePtr = SI->getPointerOperand();
  1936: 
  1937:   // Reject stores that are so large that they overflow an unsigned.
  1938:   uint64_t SizeInBits = DL->getTypeSizeInBits(StoredVal->getType());
  1939:   if ((SizeInBits & 7) || (SizeInBits >> 32) != 0)
  1940:     return false;
  1941: 
  1942:   // See if the pointer expression is an AddRec like {base,+,1} on the current
  1943:   // loop, which indicates a strided store.  If we have something else, it's a
  1944:   // random store we can't handle.
  1945:   auto *StoreEv = dyn_cast<SCEVAddRecExpr>(SE->getSCEV(StorePtr));
  1946:   if (!StoreEv || StoreEv->getLoop() != CurLoop || !StoreEv->isAffine())
  1947:     return false;
  1948: 
  1949:   // Check to see if the stride matches the size of the store.  If so, then we
  1950:   // know that every byte is touched in the loop.
  1951:   int Stride = getSCEVStride(StoreEv);
  1952:   if (Stride == 0)
  1953:     return false;
  1954:   unsigned StoreSize = DL->getTypeStoreSize(SI->getValueOperand()->getType());
  1955:   if (StoreSize != unsigned(std::abs(Stride)))
  1956:     return false;
  1957: 
  1958:   // The store must be feeding a non-volatile load.
  1959:   LoadInst *LI = dyn_cast<LoadInst>(SI->getValueOperand());
  1960:   if (!LI || !LI->isSimple())
  1961:     return false;
  1962: 
  1963:   // See if the pointer expression is an AddRec like {base,+,1} on the current
  1964:   // loop, which indicates a strided load.  If we have something else, it's a
  1965:   // random load we can't handle.
  1966:   Value *LoadPtr = LI->getPointerOperand();
  1967:   auto *LoadEv = dyn_cast<SCEVAddRecExpr>(SE->getSCEV(LoadPtr));
  1968:   if (!LoadEv || LoadEv->getLoop() != CurLoop || !LoadEv->isAffine())
  1969:     return false;
  1970: 
  1971:   // The store and load must share the same stride.
  1972:   if (StoreEv->getOperand(1) != LoadEv->getOperand(1))
  1973:     return false;
  1974: 
  1975:   // Success.  This store can be converted into a memcpy.
  1976:   return true;
  1977: }
  1978: 
  1979: /// mayLoopAccessLocation - Return true if the specified loop might access the
  1980: /// specified pointer location, which is a loop-strided access.  The 'Access'
  1981: /// argument specifies what the verboten forms of access are (read or write).
  1982: static bool
  1983: mayLoopAccessLocation(Value *Ptr, ModRefInfo Access, Loop *L,
  1984:                       const SCEV *BECount, unsigned StoreSize,
  1985:                       AliasAnalysis &AA,
  1986:                       SmallPtrSetImpl<Instruction *> &Ignored) {
  1987:   // Get the location that may be stored across the loop.  Since the access
  1988:   // is strided positively through memory, we say that the modified location
  1989:   // starts at the pointer and has infinite size.
  1990:   LocationSize AccessSize = LocationSize::afterPointer();
  1991: 
  1992:   // If the loop iterates a fixed number of times, we can refine the access
  1993:   // size to be exactly the size of the memset, which is (BECount+1)*StoreSize
  1994:   if (const SCEVConstant *BECst = dyn_cast<SCEVConstant>(BECount))
  1995:     AccessSize = LocationSize::precise((BECst->getValue()->getZExtValue() + 1) *
  1996:                                        StoreSize);
  1997: 
  1998:   // TODO: For this to be really effective, we have to dive into the pointer
  1999:   // operand in the store.  Store to &A[i] of 100 will always return may alias
  2000:   // with store of &A[100], we need to StoreLoc to be "A" with size of 100,
```
- EN: It declares or implements routines such as getCountIV, LLVM_DEBUG, setupPreSimplifier, dyn_cast<SelectInst>, ... (29 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonLoopIdiomRecognize, HexagonVolatileMemcpy, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getCountIV, LLVM_DEBUG, setupPreSimplifier, dyn_cast<SelectInst>, ... (29 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonLoopIdiomRecognize, HexagonVolatileMemcpy，说明了它与同级后端组件的连接关系。

### Lines 2001-2200 / 第 2001-2200 行

```cpp
  2001:   // which will then no-alias a store to &A[100].
  2002:   MemoryLocation StoreLoc(Ptr, AccessSize);
  2003: 
  2004:   for (auto *B : L->blocks())
  2005:     for (auto &I : *B)
  2006:       if (Ignored.count(&I) == 0 &&
  2007:           isModOrRefSet(AA.getModRefInfo(&I, StoreLoc) & Access))
  2008:         return true;
  2009: 
  2010:   return false;
  2011: }
  2012: 
  2013: void HexagonLoopIdiomRecognize::collectStores(Loop *CurLoop, BasicBlock *BB,
  2014:       SmallVectorImpl<StoreInst*> &Stores) {
  2015:   Stores.clear();
  2016:   for (Instruction &I : *BB)
  2017:     if (StoreInst *SI = dyn_cast<StoreInst>(&I))
  2018:       if (isLegalStore(CurLoop, SI))
  2019:         Stores.push_back(SI);
  2020: }
  2021: 
  2022: bool HexagonLoopIdiomRecognize::processCopyingStore(Loop *CurLoop,
  2023:       StoreInst *SI, const SCEV *BECount) {
  2024:   assert((SI->isSimple() || (SI->isVolatile() && HexagonVolatileMemcpy)) &&
  2025:          "Expected only non-volatile stores, or Hexagon-specific memcpy"
  2026:          "to volatile destination.");
  2027: 
  2028:   Value *StorePtr = SI->getPointerOperand();
  2029:   auto *StoreEv = cast<SCEVAddRecExpr>(SE->getSCEV(StorePtr));
  2030:   unsigned Stride = getSCEVStride(StoreEv);
  2031:   unsigned StoreSize = DL->getTypeStoreSize(SI->getValueOperand()->getType());
  2032:   if (Stride != StoreSize)
  2033:     return false;
  2034: 
  2035:   // See if the pointer expression is an AddRec like {base,+,1} on the current
  2036:   // loop, which indicates a strided load.  If we have something else, it's a
  2037:   // random load we can't handle.
  2038:   auto *LI = cast<LoadInst>(SI->getValueOperand());
  2039:   auto *LoadEv = cast<SCEVAddRecExpr>(SE->getSCEV(LI->getPointerOperand()));
  2040: 
  2041:   // The trip count of the loop and the base pointer of the addrec SCEV is
  2042:   // guaranteed to be loop invariant, which means that it should dominate the
  2043:   // header.  This allows us to insert code for it in the preheader.
  2044:   BasicBlock *Preheader = CurLoop->getLoopPreheader();
  2045:   Instruction *ExpPt = Preheader->getTerminator();
  2046:   IRBuilder<> Builder(ExpPt);
  2047:   SCEVExpander Expander(*SE, "hexagon-loop-idiom");
  2048: 
  2049:   Type *IntPtrTy = Builder.getIntPtrTy(*DL, SI->getPointerAddressSpace());
  2050: 
  2051:   // Okay, we have a strided store "p[i]" of a loaded value.  We can turn
  2052:   // this into a memcpy/memmove in the loop preheader now if we want.  However,
  2053:   // this would be unsafe to do if there is anything else in the loop that may
  2054:   // read or write the memory region we're storing to.  For memcpy, this
  2055:   // includes the load that feeds the stores.  Check for an alias by generating
  2056:   // the base address and checking everything.
  2057:   Value *StoreBasePtr = Expander.expandCodeFor(StoreEv->getStart(),
  2058:       Builder.getPtrTy(SI->getPointerAddressSpace()), ExpPt);
  2059:   Value *LoadBasePtr = nullptr;
  2060: 
  2061:   bool Overlap = false;
  2062:   bool DestVolatile = SI->isVolatile();
  2063:   Type *BECountTy = BECount->getType();
  2064: 
  2065:   if (DestVolatile) {
  2066:     // The trip count must fit in i32, since it is the type of the "num_words"
  2067:     // argument to hexagon_memcpy_forward_vp4cp4n2.
  2068:     if (StoreSize != 4 || DL->getTypeSizeInBits(BECountTy) > 32) {
  2069: CleanupAndExit:
  2070:       // If we generated new code for the base pointer, clean up.
  2071:       Expander.clear();
  2072:       if (StoreBasePtr && (LoadBasePtr != StoreBasePtr)) {
  2073:         RecursivelyDeleteTriviallyDeadInstructions(StoreBasePtr, TLI);
  2074:         StoreBasePtr = nullptr;
  2075:       }
  2076:       if (LoadBasePtr) {
  2077:         RecursivelyDeleteTriviallyDeadInstructions(LoadBasePtr, TLI);
  2078:         LoadBasePtr = nullptr;
  2079:       }
  2080:       return false;
  2081:     }
  2082:   }
  2083: 
  2084:   SmallPtrSet<Instruction*, 2> Ignore1;
  2085:   Ignore1.insert(SI);
  2086:   if (mayLoopAccessLocation(StoreBasePtr, ModRefInfo::ModRef, CurLoop, BECount,
  2087:                             StoreSize, *AA, Ignore1)) {
  2088:     // Check if the load is the offending instruction.
  2089:     Ignore1.insert(LI);
  2090:     if (mayLoopAccessLocation(StoreBasePtr, ModRefInfo::ModRef, CurLoop,
  2091:                               BECount, StoreSize, *AA, Ignore1)) {
  2092:       // Still bad. Nothing we can do.
  2093:       goto CleanupAndExit;
  2094:     }
  2095:     // It worked with the load ignored.
  2096:     Overlap = true;
  2097:   }
  2098: 
  2099:   if (!Overlap) {
  2100:     if (DisableMemcpyIdiom || !HasMemcpy)
  2101:       goto CleanupAndExit;
  2102:   } else {
  2103:     // Don't generate memmove if this function will be inlined. This is
  2104:     // because the caller will undergo this transformation after inlining.
  2105:     Function *Func = CurLoop->getHeader()->getParent();
  2106:     if (Func->hasFnAttribute(Attribute::AlwaysInline))
  2107:       goto CleanupAndExit;
  2108: 
  2109:     // In case of a memmove, the call to memmove will be executed instead
  2110:     // of the loop, so we need to make sure that there is nothing else in
  2111:     // the loop than the load, store and instructions that these two depend
  2112:     // on.
  2113:     SmallVector<Instruction*,2> Insts;
  2114:     Insts.push_back(SI);
  2115:     Insts.push_back(LI);
  2116:     if (!coverLoop(CurLoop, Insts))
  2117:       goto CleanupAndExit;
  2118: 
  2119:     if (DisableMemmoveIdiom || !HasMemmove)
  2120:       goto CleanupAndExit;
  2121:     bool IsNested = CurLoop->getParentLoop() != nullptr;
  2122:     if (IsNested && OnlyNonNestedMemmove)
  2123:       goto CleanupAndExit;
  2124:   }
  2125: 
  2126:   // For a memcpy, we have to make sure that the input array is not being
  2127:   // mutated by the loop.
  2128:   LoadBasePtr = Expander.expandCodeFor(LoadEv->getStart(),
  2129:       Builder.getPtrTy(LI->getPointerAddressSpace()), ExpPt);
  2130: 
  2131:   SmallPtrSet<Instruction*, 2> Ignore2;
  2132:   Ignore2.insert(SI);
  2133:   if (mayLoopAccessLocation(LoadBasePtr, ModRefInfo::Mod, CurLoop, BECount,
  2134:                             StoreSize, *AA, Ignore2))
  2135:     goto CleanupAndExit;
  2136: 
  2137:   // Check the stride.
  2138:   bool StridePos = getSCEVStride(LoadEv) >= 0;
  2139: 
  2140:   // Currently, the volatile memcpy only emulates traversing memory forward.
  2141:   if (!StridePos && DestVolatile)
  2142:     goto CleanupAndExit;
  2143: 
  2144:   bool RuntimeCheck = (Overlap || DestVolatile);
  2145: 
  2146:   BasicBlock *ExitB;
  2147:   if (RuntimeCheck) {
  2148:     // The runtime check needs a single exit block.
  2149:     SmallVector<BasicBlock*, 8> ExitBlocks;
  2150:     CurLoop->getUniqueExitBlocks(ExitBlocks);
  2151:     if (ExitBlocks.size() != 1)
  2152:       goto CleanupAndExit;
  2153:     ExitB = ExitBlocks[0];
  2154:   }
  2155: 
  2156:   // The # stored bytes is (BECount+1)*Size.  Expand the trip count out to
  2157:   // pointer size if it isn't already.
  2158:   LLVMContext &Ctx = SI->getContext();
  2159:   BECount = SE->getTruncateOrZeroExtend(BECount, IntPtrTy);
  2160:   DebugLoc DLoc = SI->getDebugLoc();
  2161: 
  2162:   const SCEV *NumBytesS =
  2163:       SE->getAddExpr(BECount, SE->getOne(IntPtrTy), SCEV::FlagNUW);
  2164:   if (StoreSize != 1)
  2165:     NumBytesS = SE->getMulExpr(NumBytesS, SE->getConstant(IntPtrTy, StoreSize),
  2166:                                SCEV::FlagNUW);
  2167:   Value *NumBytes = Expander.expandCodeFor(NumBytesS, IntPtrTy, ExpPt);
  2168:   if (Instruction *In = dyn_cast<Instruction>(NumBytes))
  2169:     if (Value *Simp = simplifyInstruction(In, {*DL, TLI, DT}))
  2170:       NumBytes = Simp;
  2171: 
  2172:   CallInst *NewCall;
  2173: 
  2174:   if (RuntimeCheck) {
  2175:     unsigned Threshold = RuntimeMemSizeThreshold;
  2176:     if (ConstantInt *CI = dyn_cast<ConstantInt>(NumBytes)) {
  2177:       uint64_t C = CI->getZExtValue();
  2178:       if (Threshold != 0 && C < Threshold)
  2179:         goto CleanupAndExit;
  2180:       if (C < CompileTimeMemSizeThreshold)
  2181:         goto CleanupAndExit;
  2182:     }
  2183: 
  2184:     BasicBlock *Header = CurLoop->getHeader();
  2185:     Function *Func = Header->getParent();
  2186:     Loop *ParentL = LF->getLoopFor(Preheader);
  2187:     StringRef HeaderName = Header->getName();
  2188: 
  2189:     // Create a new (empty) preheader, and update the PHI nodes in the
  2190:     // header to use the new preheader.
  2191:     BasicBlock *NewPreheader = BasicBlock::Create(Ctx, HeaderName+".rtli.ph",
  2192:                                                   Func, Header);
  2193:     if (ParentL)
  2194:       ParentL->addBasicBlockToLoop(NewPreheader, *LF);
  2195:     IRBuilder<>(NewPreheader).CreateBr(Header);
  2196:     for (auto &In : *Header) {
  2197:       PHINode *PN = dyn_cast<PHINode>(&In);
  2198:       if (!PN)
  2199:         break;
  2200:       int bx = PN->getBasicBlockIndex(Preheader);
```
- EN: It declares or implements routines such as StoreLoc, HexagonLoopIdiomRecognize::collectStores, clear, HexagonLoopIdiomRecognize::processCopyingStore, ... (35 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonLoopIdiomRecognize, HexagonVolatileMemcpy, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 StoreLoc, HexagonLoopIdiomRecognize::collectStores, clear, HexagonLoopIdiomRecognize::processCopyingStore, ... (35 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonLoopIdiomRecognize, HexagonVolatileMemcpy，说明了它与同级后端组件的连接关系。

### Lines 2201-2400 / 第 2201-2400 行

```cpp
  2201:       if (bx >= 0)
  2202:         PN->setIncomingBlock(bx, NewPreheader);
  2203:     }
  2204:     DT->addNewBlock(NewPreheader, Preheader);
  2205:     DT->changeImmediateDominator(Header, NewPreheader);
  2206: 
  2207:     // Check for safe conditions to execute memmove.
  2208:     // If stride is positive, copying things from higher to lower addresses
  2209:     // is equivalent to memmove.  For negative stride, it's the other way
  2210:     // around.  Copying forward in memory with positive stride may not be
  2211:     // same as memmove since we may be copying values that we just stored
  2212:     // in some previous iteration.
  2213:     Value *LA = Builder.CreatePtrToInt(LoadBasePtr, IntPtrTy);
  2214:     Value *SA = Builder.CreatePtrToInt(StoreBasePtr, IntPtrTy);
  2215:     Value *LowA = StridePos ? SA : LA;
  2216:     Value *HighA = StridePos ? LA : SA;
  2217:     Value *CmpA = Builder.CreateICmpULT(LowA, HighA);
  2218:     Value *Cond = CmpA;
  2219: 
  2220:     // Check for distance between pointers. Since the case LowA < HighA
  2221:     // is checked for above, assume LowA >= HighA.
  2222:     Value *Dist = Builder.CreateSub(LowA, HighA);
  2223:     Value *CmpD = Builder.CreateICmpSLE(NumBytes, Dist);
  2224:     Value *CmpEither = Builder.CreateOr(Cond, CmpD);
  2225:     Cond = CmpEither;
  2226: 
  2227:     if (Threshold != 0) {
  2228:       Type *Ty = NumBytes->getType();
  2229:       Value *Thr = ConstantInt::get(Ty, Threshold);
  2230:       Value *CmpB = Builder.CreateICmpULT(Thr, NumBytes);
  2231:       Value *CmpBoth = Builder.CreateAnd(Cond, CmpB);
  2232:       Cond = CmpBoth;
  2233:     }
  2234:     BasicBlock *MemmoveB = BasicBlock::Create(Ctx, Header->getName()+".rtli",
  2235:                                               Func, NewPreheader);
  2236:     if (ParentL)
  2237:       ParentL->addBasicBlockToLoop(MemmoveB, *LF);
  2238:     Instruction *OldT = Preheader->getTerminator();
  2239:     Builder.CreateCondBr(Cond, MemmoveB, NewPreheader);
  2240:     OldT->eraseFromParent();
  2241:     Preheader->setName(Preheader->getName()+".old");
  2242:     DT->addNewBlock(MemmoveB, Preheader);
  2243:     // Find the new immediate dominator of the exit block.
  2244:     BasicBlock *ExitD = Preheader;
  2245:     for (BasicBlock *PB : predecessors(ExitB)) {
  2246:       ExitD = DT->findNearestCommonDominator(ExitD, PB);
  2247:       if (!ExitD)
  2248:         break;
  2249:     }
  2250:     // If the prior immediate dominator of ExitB was dominated by the
  2251:     // old preheader, then the old preheader becomes the new immediate
  2252:     // dominator.  Otherwise don't change anything (because the newly
  2253:     // added blocks are dominated by the old preheader).
  2254:     if (ExitD && DT->dominates(Preheader, ExitD)) {
  2255:       DomTreeNode *BN = DT->getNode(ExitB);
  2256:       DomTreeNode *DN = DT->getNode(ExitD);
  2257:       BN->setIDom(DN);
  2258:     }
  2259: 
  2260:     // Add a call to memmove to the conditional block.
  2261:     IRBuilder<> CondBuilder(MemmoveB);
  2262:     CondBuilder.CreateBr(ExitB);
  2263:     CondBuilder.SetInsertPoint(MemmoveB->getTerminator());
  2264: 
  2265:     if (DestVolatile) {
  2266:       Type *Int32Ty = Type::getInt32Ty(Ctx);
  2267:       Type *PtrTy = PointerType::get(Ctx, 0);
  2268:       Type *VoidTy = Type::getVoidTy(Ctx);
  2269:       Module *M = Func->getParent();
  2270: 
  2271:       // FIXME: This should check if the call is supported
  2272:       StringRef HexagonVolatileMemcpyName =
  2273:           RTLIB::RuntimeLibcallsInfo::getLibcallImplName(
  2274:               RTLIB::impl_hexagon_memcpy_forward_vp4cp4n2);
  2275:       FunctionCallee Fn = M->getOrInsertFunction(
  2276:           HexagonVolatileMemcpyName, VoidTy, PtrTy, PtrTy, Int32Ty);
  2277: 
  2278:       const SCEV *OneS = SE->getConstant(Int32Ty, 1);
  2279:       const SCEV *BECount32 = SE->getTruncateOrZeroExtend(BECount, Int32Ty);
  2280:       const SCEV *NumWordsS = SE->getAddExpr(BECount32, OneS, SCEV::FlagNUW);
  2281:       Value *NumWords = Expander.expandCodeFor(NumWordsS, Int32Ty,
  2282:                                                MemmoveB->getTerminator());
  2283:       if (Instruction *In = dyn_cast<Instruction>(NumWords))
  2284:         if (Value *Simp = simplifyInstruction(In, {*DL, TLI, DT}))
  2285:           NumWords = Simp;
  2286: 
  2287:       NewCall = CondBuilder.CreateCall(Fn,
  2288:                                        {StoreBasePtr, LoadBasePtr, NumWords});
  2289:     } else {
  2290:       NewCall = CondBuilder.CreateMemMove(
  2291:           StoreBasePtr, SI->getAlign(), LoadBasePtr, LI->getAlign(), NumBytes);
  2292:     }
  2293:   } else {
  2294:     NewCall = Builder.CreateMemCpy(StoreBasePtr, SI->getAlign(), LoadBasePtr,
  2295:                                    LI->getAlign(), NumBytes);
  2296:     // Okay, the memcpy has been formed.  Zap the original store and
  2297:     // anything that feeds into it.
  2298:     RecursivelyDeleteTriviallyDeadInstructions(SI, TLI);
  2299:   }
  2300: 
  2301:   NewCall->setDebugLoc(DLoc);
  2302: 
  2303:   LLVM_DEBUG(dbgs() << "  Formed " << (Overlap ? "memmove: " : "memcpy: ")
  2304:                     << *NewCall << "\n"
  2305:                     << "    from load ptr=" << *LoadEv << " at: " << *LI << "\n"
  2306:                     << "    from store ptr=" << *StoreEv << " at: " << *SI
  2307:                     << "\n");
  2308: 
  2309:   return true;
  2310: }
  2311: 
  2312: // Check if the instructions in Insts, together with their dependencies
  2313: // cover the loop in the sense that the loop could be safely eliminated once
  2314: // the instructions in Insts are removed.
  2315: bool HexagonLoopIdiomRecognize::coverLoop(Loop *L,
  2316:       SmallVectorImpl<Instruction*> &Insts) const {
  2317:   SmallPtrSet<BasicBlock *, 8> LoopBlocks;
  2318:   LoopBlocks.insert_range(L->blocks());
  2319: 
  2320:   SetVector<Instruction *> Worklist(llvm::from_range, Insts);
  2321: 
  2322:   // Collect all instructions from the loop that the instructions in Insts
  2323:   // depend on (plus their dependencies, etc.).  These instructions will
  2324:   // constitute the expression trees that feed those in Insts, but the trees
  2325:   // will be limited only to instructions contained in the loop.
  2326:   for (unsigned i = 0; i < Worklist.size(); ++i) {
  2327:     Instruction *In = Worklist[i];
  2328:     for (auto I = In->op_begin(), E = In->op_end(); I != E; ++I) {
  2329:       Instruction *OpI = dyn_cast<Instruction>(I);
  2330:       if (!OpI)
  2331:         continue;
  2332:       BasicBlock *PB = OpI->getParent();
  2333:       if (!LoopBlocks.count(PB))
  2334:         continue;
  2335:       Worklist.insert(OpI);
  2336:     }
  2337:   }
  2338: 
  2339:   // Scan all instructions in the loop, if any of them have a user outside
  2340:   // of the loop, or outside of the expressions collected above, then either
  2341:   // the loop has a side-effect visible outside of it, or there are
  2342:   // instructions in it that are not involved in the original set Insts.
  2343:   for (auto *B : L->blocks()) {
  2344:     for (auto &In : *B) {
  2345:       if (isa<UncondBrInst, CondBrInst>(In))
  2346:         continue;
  2347:       if (!Worklist.count(&In) && In.mayHaveSideEffects())
  2348:         return false;
  2349:       for (auto *K : In.users()) {
  2350:         Instruction *UseI = dyn_cast<Instruction>(K);
  2351:         if (!UseI)
  2352:           continue;
  2353:         BasicBlock *UseB = UseI->getParent();
  2354:         if (LF->getLoopFor(UseB) != L)
  2355:           return false;
  2356:       }
  2357:     }
  2358:   }
  2359: 
  2360:   return true;
  2361: }
  2362: 
  2363: /// runOnLoopBlock - Process the specified block, which lives in a counted loop
  2364: /// with the specified backedge count.  This block is known to be in the current
  2365: /// loop and not in any subloops.
  2366: bool HexagonLoopIdiomRecognize::runOnLoopBlock(Loop *CurLoop, BasicBlock *BB,
  2367:       const SCEV *BECount, SmallVectorImpl<BasicBlock*> &ExitBlocks) {
  2368:   // We can only promote stores in this block if they are unconditionally
  2369:   // executed in the loop.  For a block to be unconditionally executed, it has
  2370:   // to dominate all the exit blocks of the loop.  Verify this now.
  2371:   auto DominatedByBB = [this,BB] (BasicBlock *EB) -> bool {
  2372:     return DT->dominates(BB, EB);
  2373:   };
  2374:   if (!all_of(ExitBlocks, DominatedByBB))
  2375:     return false;
  2376: 
  2377:   bool MadeChange = false;
  2378:   // Look for store instructions, which may be optimized to memset/memcpy.
  2379:   SmallVector<StoreInst*,8> Stores;
  2380:   collectStores(CurLoop, BB, Stores);
  2381: 
  2382:   // Optimize the store into a memcpy, if it feeds an similarly strided load.
  2383:   for (auto &SI : Stores)
  2384:     MadeChange |= processCopyingStore(CurLoop, SI, BECount);
  2385: 
  2386:   return MadeChange;
  2387: }
  2388: 
  2389: bool HexagonLoopIdiomRecognize::runOnCountableLoop(Loop *L) {
  2390:   PolynomialMultiplyRecognize PMR(L, *DL, *DT, *TLI, *SE);
  2391:   if (PMR.recognize())
  2392:     return true;
  2393: 
  2394:   if (!HasMemcpy && !HasMemmove)
  2395:     return false;
  2396: 
  2397:   const SCEV *BECount = SE->getBackedgeTakenCount(L);
  2398:   assert(!isa<SCEVCouldNotCompute>(BECount) &&
  2399:          "runOnCountableLoop() called on a loop without a predictable"
  2400:          "backedge-taken count");
```
- EN: It declares or implements routines such as addNewBlock, changeImmediateDominator, CreatePtrToInt, CreateICmpULT, ... (50 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonVolatileMemcpyName, HexagonLoopIdiomRecognize, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 addNewBlock, changeImmediateDominator, CreatePtrToInt, CreateICmpULT, ... (50 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonVolatileMemcpyName, HexagonLoopIdiomRecognize，说明了它与同级后端组件的连接关系。

### Lines 2401-2469 / 第 2401-2469 行

```cpp
  2401: 
  2402:   SmallVector<BasicBlock *, 8> ExitBlocks;
  2403:   L->getUniqueExitBlocks(ExitBlocks);
  2404: 
  2405:   bool Changed = false;
  2406: 
  2407:   // Scan all the blocks in the loop that are not in subloops.
  2408:   for (auto *BB : L->getBlocks()) {
  2409:     // Ignore blocks in subloops.
  2410:     if (LF->getLoopFor(BB) != L)
  2411:       continue;
  2412:     Changed |= runOnLoopBlock(L, BB, BECount, ExitBlocks);
  2413:   }
  2414: 
  2415:   return Changed;
  2416: }
  2417: 
  2418: bool HexagonLoopIdiomRecognize::run(Loop *L) {
  2419:   const Module &M = *L->getHeader()->getParent()->getParent();
  2420:   if (M.getTargetTriple().getArch() != Triple::hexagon)
  2421:     return false;
  2422: 
  2423:   // If the loop could not be converted to canonical form, it must have an
  2424:   // indirectbr in it, just give up.
  2425:   if (!L->getLoopPreheader())
  2426:     return false;
  2427: 
  2428:   // Disable loop idiom recognition if the function's name is a common idiom.
  2429:   StringRef Name = L->getHeader()->getParent()->getName();
  2430:   if (Name == "memset" || Name == "memcpy" || Name == "memmove")
  2431:     return false;
  2432: 
  2433:   DL = &L->getHeader()->getDataLayout();
  2434: 
  2435:   HasMemcpy = TLI->has(LibFunc_memcpy);
  2436:   HasMemmove = TLI->has(LibFunc_memmove);
  2437: 
  2438:   if (SE->hasLoopInvariantBackedgeTakenCount(L))
  2439:     return runOnCountableLoop(L);
  2440:   return false;
  2441: }
  2442: 
  2443: bool HexagonLoopIdiomRecognizeLegacyPass::runOnLoop(Loop *L,
  2444:                                                     LPPassManager &LPM) {
  2445:   if (skipLoop(L))
  2446:     return false;
  2447: 
  2448:   auto *AA = &getAnalysis<AAResultsWrapperPass>().getAAResults();
  2449:   auto *DT = &getAnalysis<DominatorTreeWrapperPass>().getDomTree();
  2450:   auto *LF = &getAnalysis<LoopInfoWrapperPass>().getLoopInfo();
  2451:   auto *TLI = &getAnalysis<TargetLibraryInfoWrapperPass>().getTLI(
  2452:       *L->getHeader()->getParent());
  2453:   auto *SE = &getAnalysis<ScalarEvolutionWrapperPass>().getSE();
  2454:   return HexagonLoopIdiomRecognize(AA, DT, LF, TLI, SE).run(L);
  2455: }
  2456: 
  2457: Pass *llvm::createHexagonLoopIdiomPass() {
  2458:   return new HexagonLoopIdiomRecognizeLegacyPass();
  2459: }
  2460: 
  2461: PreservedAnalyses
  2462: HexagonLoopIdiomRecognitionPass::run(Loop &L, LoopAnalysisManager &AM,
  2463:                                      LoopStandardAnalysisResults &AR,
  2464:                                      LPMUpdater &U) {
  2465:   return HexagonLoopIdiomRecognize(&AR.AA, &AR.DT, &AR.LI, &AR.TLI, &AR.SE)
  2466:                  .run(&L)
  2467:              ? getLoopPassPreservedAnalyses()
  2468:              : PreservedAnalyses::all();
  2469: }
```
- EN: It declares or implements routines such as getUniqueExitBlocks, runOnLoopBlock, HexagonLoopIdiomRecognize::run, getHeader, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonLoopIdiomRecognize, HexagonLoopIdiomRecognizeLegacyPass, HexagonLoopIdiomRecognitionPass, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getUniqueExitBlocks, runOnLoopBlock, HexagonLoopIdiomRecognize::run, getHeader, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonLoopIdiomRecognize, HexagonLoopIdiomRecognizeLegacyPass, HexagonLoopIdiomRecognitionPass，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- loop-aware code generation / 循环相关代码生成
- instruction semantics / 指令语义
- pattern-driven instruction selection / 基于模式的指令选择
- intrinsic mapping / Intrinsic 映射

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonLoopIdiomRecognition.h, Hexagon.h, llvm/ADT/APInt.h, llvm/ADT/DenseMap.h, llvm/ADT/SetVector.h, llvm/ADT/SmallPtrSet.h, llvm/ADT/SmallVector.h, llvm/ADT/StringRef.h, llvm/Analysis/AliasAnalysis.h, llvm/Analysis/InstructionSimplify.h, ... (66 total)`
- Hexagon symbols / Hexagon 符号: `HexagonLoopIdiomRecognition, HexagonVolatileMemcpy, HexagonLoopIdiomRecognize, HexagonLoopIdiomRecognizeLegacyPass, HexagonVolatileMemcpyName, HexagonLoopIdiomRecognitionPass`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
