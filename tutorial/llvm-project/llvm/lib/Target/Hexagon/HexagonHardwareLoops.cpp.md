# HexagonHardwareLoops.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonHardwareLoops.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon hardware-loop recognition and transformation.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。 重点涉及循环优化。

## Line-by-Line Analysis / 逐行分析

### Lines 1-200 / 第 1-200 行

```cpp
     1: //===- HexagonHardwareLoops.cpp - Identify and generate hardware loops ----===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This pass identifies loops where we can generate the Hexagon hardware
    10: // loop instruction.  The hardware loop can perform loop branches with a
    11: // zero-cycle overhead.
    12: //
    13: // The pattern that defines the induction variable can changed depending on
    14: // prior optimizations.  For example, the IndVarSimplify phase run by 'opt'
    15: // normalizes induction variables, and the Loop Strength Reduction pass
    16: // run by 'llc' may also make changes to the induction variable.
    17: // The pattern detected by this phase is due to running Strength Reduction.
    18: //
    19: // Criteria for hardware loops:
    20: //  - Countable loops (w/ ind. var for a trip count)
    21: //  - Assumes loops are normalized by IndVarSimplify
    22: //  - Try inner-most loops first
    23: //  - No function calls in loops.
    24: //
    25: //===----------------------------------------------------------------------===//
    26: 
    27: #include "Hexagon.h"
    28: #include "HexagonInstrInfo.h"
    29: #include "HexagonSubtarget.h"
    30: #include "llvm/ADT/ArrayRef.h"
    31: #include "llvm/ADT/STLExtras.h"
    32: #include "llvm/ADT/SmallSet.h"
    33: #include "llvm/ADT/SmallVector.h"
    34: #include "llvm/ADT/Statistic.h"
    35: #include "llvm/ADT/StringRef.h"
    36: #include "llvm/CodeGen/MachineBasicBlock.h"
    37: #include "llvm/CodeGen/MachineDominators.h"
    38: #include "llvm/CodeGen/MachineFunction.h"
    39: #include "llvm/CodeGen/MachineFunctionPass.h"
    40: #include "llvm/CodeGen/MachineInstr.h"
    41: #include "llvm/CodeGen/MachineInstrBuilder.h"
    42: #include "llvm/CodeGen/MachineLoopInfo.h"
    43: #include "llvm/CodeGen/MachineOperand.h"
    44: #include "llvm/CodeGen/MachineRegisterInfo.h"
    45: #include "llvm/CodeGen/TargetRegisterInfo.h"
    46: #include "llvm/IR/DebugLoc.h"
    47: #include "llvm/InitializePasses.h"
    48: #include "llvm/Pass.h"
    49: #include "llvm/Support/CommandLine.h"
    50: #include "llvm/Support/Debug.h"
    51: #include "llvm/Support/ErrorHandling.h"
    52: #include "llvm/Support/MathExtras.h"
    53: #include "llvm/Support/raw_ostream.h"
    54: #include <cassert>
    55: #include <cstdint>
    56: #include <cstdlib>
    57: #include <iterator>
    58: #include <map>
    59: #include <set>
    60: #include <string>
    61: #include <utility>
    62: #include <vector>
    63: 
    64: using namespace llvm;
    65: 
    66: #define DEBUG_TYPE "hwloops"
    67: 
    68: #ifndef NDEBUG
    69: static cl::opt<int> HWLoopLimit("hexagon-max-hwloop", cl::Hidden, cl::init(-1));
    70: 
    71: // Option to create preheader only for a specific function.
    72: static cl::opt<std::string> PHFn("hexagon-hwloop-phfn", cl::Hidden,
    73:                                  cl::init(""));
    74: #endif
    75: 
    76: // Option to create a preheader if one doesn't exist.
    77: static cl::opt<bool> HWCreatePreheader("hexagon-hwloop-preheader",
    78:     cl::Hidden, cl::init(true),
    79:     cl::desc("Add a preheader to a hardware loop if one doesn't exist"));
    80: 
    81: // Turn it off by default. If a preheader block is not created here, the
    82: // software pipeliner may be unable to find a block suitable to serve as
    83: // a preheader. In that case SWP will not run.
    84: static cl::opt<bool> SpecPreheader("hwloop-spec-preheader", cl::Hidden,
    85:                                    cl::desc("Allow speculation of preheader "
    86:                                             "instructions"));
    87: 
    88: STATISTIC(NumHWLoops, "Number of loops converted to hardware loops");
    89: 
    90: namespace {
    91: 
    92:   class CountValue;
    93: 
    94:   struct HexagonHardwareLoops : public MachineFunctionPass {
    95:     MachineLoopInfo            *MLI;
    96:     MachineRegisterInfo        *MRI;
    97:     MachineDominatorTree       *MDT;
    98:     const HexagonInstrInfo     *TII;
    99:     const HexagonRegisterInfo  *TRI;
   100: #ifndef NDEBUG
   101:     static int Counter;
   102: #endif
   103: 
   104:   public:
   105:     static char ID;
   106: 
   107:     HexagonHardwareLoops() : MachineFunctionPass(ID) {}
   108: 
   109:     bool runOnMachineFunction(MachineFunction &MF) override;
   110: 
   111:     StringRef getPassName() const override { return "Hexagon Hardware Loops"; }
   112: 
   113:     void getAnalysisUsage(AnalysisUsage &AU) const override {
   114:       AU.addRequired<MachineDominatorTreeWrapperPass>();
   115:       AU.addRequired<MachineLoopInfoWrapperPass>();
   116:       MachineFunctionPass::getAnalysisUsage(AU);
   117:     }
   118: 
   119:   private:
   120:     using LoopFeederMap = std::map<Register, MachineInstr *>;
   121: 
   122:     /// Kinds of comparisons in the compare instructions.
   123:     struct Comparison {
   124:       enum Kind {
   125:         EQ  = 0x01,
   126:         NE  = 0x02,
   127:         L   = 0x04,
   128:         G   = 0x08,
   129:         U   = 0x40,
   130:         LTs = L,
   131:         LEs = L | EQ,
   132:         GTs = G,
   133:         GEs = G | EQ,
   134:         LTu = L      | U,
   135:         LEu = L | EQ | U,
   136:         GTu = G      | U,
   137:         GEu = G | EQ | U
   138:       };
   139: 
   140:       static Kind getSwappedComparison(Kind Cmp) {
   141:         assert ((!((Cmp & L) && (Cmp & G))) && "Malformed comparison operator");
   142:         if ((Cmp & L) || (Cmp & G))
   143:           return (Kind)(Cmp ^ (L|G));
   144:         return Cmp;
   145:       }
   146: 
   147:       static Kind getNegatedComparison(Kind Cmp) {
   148:         if ((Cmp & L) || (Cmp & G))
   149:           return (Kind)((Cmp ^ (L | G)) ^ EQ);
   150:         if ((Cmp & NE) || (Cmp & EQ))
   151:           return (Kind)(Cmp ^ (EQ | NE));
   152:         return (Kind)0;
   153:       }
   154: 
   155:       static bool isSigned(Kind Cmp) {
   156:         return (Cmp & (L | G) && !(Cmp & U));
   157:       }
   158: 
   159:       static bool isUnsigned(Kind Cmp) {
   160:         return (Cmp & U);
   161:       }
   162:     };
   163: 
   164:     /// Find the register that contains the loop controlling
   165:     /// induction variable.
   166:     /// If successful, it will return true and set the \p Reg, \p IVBump
   167:     /// and \p IVOp arguments.  Otherwise it will return false.
   168:     /// The returned induction register is the register R that follows the
   169:     /// following induction pattern:
   170:     /// loop:
   171:     ///   R = phi ..., [ R.next, LatchBlock ]
   172:     ///   R.next = R + #bump
   173:     ///   if (R.next < #N) goto loop
   174:     /// IVBump is the immediate value added to R, and IVOp is the instruction
   175:     /// "R.next = R + #bump".
   176:     bool findInductionRegister(MachineLoop *L, Register &Reg,
   177:                                int64_t &IVBump, MachineInstr *&IVOp) const;
   178: 
   179:     /// Return the comparison kind for the specified opcode.
   180:     Comparison::Kind getComparisonKind(unsigned CondOpc,
   181:                                        MachineOperand *InitialValue,
   182:                                        const MachineOperand *Endvalue,
   183:                                        int64_t IVBump) const;
   184: 
   185:     /// Analyze the statements in a loop to determine if the loop
   186:     /// has a computable trip count and, if so, return a value that represents
   187:     /// the trip count expression.
   188:     CountValue *getLoopTripCount(MachineLoop *L,
   189:                                  SmallVectorImpl<MachineInstr *> &OldInsts);
   190: 
   191:     /// Return the expression that represents the number of times
   192:     /// a loop iterates.  The function takes the operands that represent the
   193:     /// loop start value, loop end value, and induction value.  Based upon
   194:     /// these operands, the function attempts to compute the trip count.
   195:     /// If the trip count is not directly available (as an immediate value,
   196:     /// or a register), the function will attempt to insert computation of it
   197:     /// to the loop's preheader.
   198:     CountValue *computeCount(MachineLoop *Loop, const MachineOperand *Start,
   199:                              const MachineOperand *End, Register IVReg,
   200:                              int64_t IVBump, Comparison::Kind Cmp) const;
```
- EN: It imports headers such as Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/ArrayRef.h, ... (36 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as CountValue, HexagonHardwareLoops, Comparison, Kind, which carry the state or API of this component.
- CN: 这里引入了 Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/ArrayRef.h, ... (36 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 CountValue, HexagonHardwareLoops, Comparison, Kind 等类型，用来承载该组件的状态或接口。

### Lines 201-400 / 第 201-400 行

```cpp
   201: 
   202:     /// Return true if the instruction is not valid within a hardware
   203:     /// loop.
   204:     bool isInvalidLoopOperation(const MachineInstr *MI,
   205:                                 bool IsInnerHWLoop) const;
   206: 
   207:     /// Return true if the loop contains an instruction that inhibits
   208:     /// using the hardware loop.
   209:     bool containsInvalidInstruction(MachineLoop *L, bool IsInnerHWLoop) const;
   210: 
   211:     /// Given a loop, check if we can convert it to a hardware loop.
   212:     /// If so, then perform the conversion and return true.
   213:     bool convertToHardwareLoop(MachineLoop *L, bool &L0used, bool &L1used);
   214: 
   215:     /// Return true if the instruction is now dead.
   216:     bool isDead(const MachineInstr *MI,
   217:                 SmallVectorImpl<MachineInstr *> &DeadPhis) const;
   218: 
   219:     /// Remove the instruction if it is now dead.
   220:     void removeIfDead(MachineInstr *MI);
   221: 
   222:     /// Make sure that the "bump" instruction executes before the
   223:     /// compare.  We need that for the IV fixup, so that the compare
   224:     /// instruction would not use a bumped value that has not yet been
   225:     /// defined.  If the instructions are out of order, try to reorder them.
   226:     bool orderBumpCompare(MachineInstr *BumpI, MachineInstr *CmpI);
   227: 
   228:     /// Return true if MO and MI pair is visited only once. If visited
   229:     /// more than once, this indicates there is recursion. In such a case,
   230:     /// return false.
   231:     bool isLoopFeeder(MachineLoop *L, MachineBasicBlock *A, MachineInstr *MI,
   232:                       const MachineOperand *MO,
   233:                       LoopFeederMap &LoopFeederPhi) const;
   234: 
   235:     /// Return true if the Phi may generate a value that may underflow,
   236:     /// or may wrap.
   237:     bool phiMayWrapOrUnderflow(MachineInstr *Phi, const MachineOperand *EndVal,
   238:                                MachineBasicBlock *MBB, MachineLoop *L,
   239:                                LoopFeederMap &LoopFeederPhi) const;
   240: 
   241:     /// Return true if the induction variable may underflow an unsigned
   242:     /// value in the first iteration.
   243:     bool loopCountMayWrapOrUnderFlow(const MachineOperand *InitVal,
   244:                                      const MachineOperand *EndVal,
   245:                                      MachineBasicBlock *MBB, MachineLoop *L,
   246:                                      LoopFeederMap &LoopFeederPhi) const;
   247: 
   248:     /// Check if the given operand has a compile-time known constant
   249:     /// value. Return true if yes, and false otherwise. When returning true, set
   250:     /// Val to the corresponding constant value.
   251:     bool checkForImmediate(const MachineOperand &MO, int64_t &Val) const;
   252: 
   253:     /// Check if the operand has a compile-time known constant value.
   254:     bool isImmediate(const MachineOperand &MO) const {
   255:       int64_t V;
   256:       return checkForImmediate(MO, V);
   257:     }
   258: 
   259:     /// Return the immediate for the specified operand.
   260:     int64_t getImmediate(const MachineOperand &MO) const {
   261:       int64_t V;
   262:       if (!checkForImmediate(MO, V))
   263:         llvm_unreachable("Invalid operand");
   264:       return V;
   265:     }
   266: 
   267:     /// Reset the given machine operand to now refer to a new immediate
   268:     /// value.  Assumes that the operand was already referencing an immediate
   269:     /// value, either directly, or via a register.
   270:     void setImmediate(MachineOperand &MO, int64_t Val);
   271: 
   272:     /// Fix the data flow of the induction variable.
   273:     /// The desired flow is: phi ---> bump -+-> comparison-in-latch.
   274:     ///                                     |
   275:     ///                                     +-> back to phi
   276:     /// where "bump" is the increment of the induction variable:
   277:     ///   iv = iv + #const.
   278:     /// Due to some prior code transformations, the actual flow may look
   279:     /// like this:
   280:     ///   phi -+-> bump ---> back to phi
   281:     ///        |
   282:     ///        +-> comparison-in-latch (against upper_bound-bump),
   283:     /// i.e. the comparison that controls the loop execution may be using
   284:     /// the value of the induction variable from before the increment.
   285:     ///
   286:     /// Return true if the loop's flow is the desired one (i.e. it's
   287:     /// either been fixed, or no fixing was necessary).
   288:     /// Otherwise, return false.  This can happen if the induction variable
   289:     /// couldn't be identified, or if the value in the latch's comparison
   290:     /// cannot be adjusted to reflect the post-bump value.
   291:     bool fixupInductionVariable(MachineLoop *L);
   292: 
   293:     /// Given a loop, if it does not have a preheader, create one.
   294:     /// Return the block that is the preheader.
   295:     MachineBasicBlock *createPreheaderForLoop(MachineLoop *L);
   296:   };
   297: 
   298:   char HexagonHardwareLoops::ID = 0;
   299: #ifndef NDEBUG
   300:   int HexagonHardwareLoops::Counter = 0;
   301: #endif
   302: 
   303:   /// Abstraction for a trip count of a loop. A smaller version
   304:   /// of the MachineOperand class without the concerns of changing the
   305:   /// operand representation.
   306:   class CountValue {
   307:   public:
   308:     enum CountValueType {
   309:       CV_Register,
   310:       CV_Immediate
   311:     };
   312: 
   313:   private:
   314:     CountValueType Kind;
   315:     union Values {
   316:       Values() : R{Register(), 0} {}
   317:       Values(const Values&) = default;
   318:       struct {
   319:         Register Reg;
   320:         unsigned Sub;
   321:       } R;
   322:       unsigned ImmVal;
   323:     } Contents;
   324: 
   325:   public:
   326:     explicit CountValue(CountValueType t, Register v, unsigned u = 0) {
   327:       Kind = t;
   328:       if (Kind == CV_Register) {
   329:         Contents.R.Reg = v;
   330:         Contents.R.Sub = u;
   331:       } else {
   332:         Contents.ImmVal = v;
   333:       }
   334:     }
   335: 
   336:     bool isReg() const { return Kind == CV_Register; }
   337:     bool isImm() const { return Kind == CV_Immediate; }
   338: 
   339:     Register getReg() const {
   340:       assert(isReg() && "Wrong CountValue accessor");
   341:       return Contents.R.Reg;
   342:     }
   343: 
   344:     unsigned getSubReg() const {
   345:       assert(isReg() && "Wrong CountValue accessor");
   346:       return Contents.R.Sub;
   347:     }
   348: 
   349:     unsigned getImm() const {
   350:       assert(isImm() && "Wrong CountValue accessor");
   351:       return Contents.ImmVal;
   352:     }
   353: 
   354:     void print(raw_ostream &OS, const TargetRegisterInfo *TRI = nullptr) const {
   355:       if (isReg()) { OS << printReg(Contents.R.Reg, TRI, Contents.R.Sub); }
   356:       if (isImm()) { OS << Contents.ImmVal; }
   357:     }
   358:   };
   359: 
   360: } // end anonymous namespace
   361: 
   362: INITIALIZE_PASS_BEGIN(HexagonHardwareLoops, "hwloops",
   363:                       "Hexagon Hardware Loops", false, false)
   364: INITIALIZE_PASS_DEPENDENCY(MachineDominatorTreeWrapperPass)
   365: INITIALIZE_PASS_DEPENDENCY(MachineLoopInfoWrapperPass)
   366: INITIALIZE_PASS_END(HexagonHardwareLoops, "hwloops",
   367:                     "Hexagon Hardware Loops", false, false)
   368: 
   369: FunctionPass *llvm::createHexagonHardwareLoops() {
   370:   return new HexagonHardwareLoops();
   371: }
   372: 
   373: bool HexagonHardwareLoops::runOnMachineFunction(MachineFunction &MF) {
   374:   LLVM_DEBUG(dbgs() << "********* Hexagon Hardware Loops *********\n");
   375:   if (skipFunction(MF.getFunction()))
   376:     return false;
   377: 
   378:   bool Changed = false;
   379: 
   380:   MLI = &getAnalysis<MachineLoopInfoWrapperPass>().getLI();
   381:   MRI = &MF.getRegInfo();
   382:   MDT = &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
   383:   const HexagonSubtarget &HST = MF.getSubtarget<HexagonSubtarget>();
   384:   TII = HST.getInstrInfo();
   385:   TRI = HST.getRegisterInfo();
   386: 
   387:   for (auto &L : *MLI)
   388:     if (L->isOutermost()) {
   389:       bool L0Used = false;
   390:       bool L1Used = false;
   391:       Changed |= convertToHardwareLoop(L, L0Used, L1Used);
   392:     }
   393: 
   394:   return Changed;
   395: }
   396: 
   397: bool HexagonHardwareLoops::findInductionRegister(MachineLoop *L,
   398:                                                  Register &Reg,
   399:                                                  int64_t &IVBump,
   400:                                                  MachineInstr *&IVOp
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (INITIALIZE_PASS_BEGIN) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as CountValue, CountValueType, which carry the state or API of this component. It defines declarative TableGen records like CountValue; these records are consumed by TableGen instead of executed directly.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（INITIALIZE_PASS_BEGIN），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 CountValue, CountValueType 等类型，用来承载该组件的状态或接口。 这里定义了 CountValue 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 401-600 / 第 401-600 行

```cpp
   401:                                                  ) const {
   402:   MachineBasicBlock *Header = L->getHeader();
   403:   MachineBasicBlock *Preheader = MLI->findLoopPreheader(L, SpecPreheader);
   404:   MachineBasicBlock *Latch = L->getLoopLatch();
   405:   MachineBasicBlock *ExitingBlock = L->findLoopControlBlock();
   406:   if (!Header || !Preheader || !Latch || !ExitingBlock)
   407:     return false;
   408: 
   409:   // This pair represents an induction register together with an immediate
   410:   // value that will be added to it in each loop iteration.
   411:   using RegisterBump = std::pair<Register, int64_t>;
   412: 
   413:   // Mapping:  R.next -> (R, bump), where R, R.next and bump are derived
   414:   // from an induction operation
   415:   //   R.next = R + bump
   416:   // where bump is an immediate value.
   417:   using InductionMap = std::map<Register, RegisterBump>;
   418: 
   419:   InductionMap IndMap;
   420: 
   421:   using instr_iterator = MachineBasicBlock::instr_iterator;
   422: 
   423:   for (instr_iterator I = Header->instr_begin(), E = Header->instr_end();
   424:        I != E && I->isPHI(); ++I) {
   425:     MachineInstr *Phi = &*I;
   426: 
   427:     // Have a PHI instruction.  Get the operand that corresponds to the
   428:     // latch block, and see if is a result of an addition of form "reg+imm",
   429:     // where the "reg" is defined by the PHI node we are looking at.
   430:     for (unsigned i = 1, n = Phi->getNumOperands(); i < n; i += 2) {
   431:       if (Phi->getOperand(i+1).getMBB() != Latch)
   432:         continue;
   433: 
   434:       Register PhiOpReg = Phi->getOperand(i).getReg();
   435:       MachineInstr *DI = MRI->getVRegDef(PhiOpReg);
   436: 
   437:       if (DI->getDesc().isAdd()) {
   438:         // If the register operand to the add is the PHI we're looking at, this
   439:         // meets the induction pattern.
   440:         Register IndReg = DI->getOperand(1).getReg();
   441:         MachineOperand &Opnd2 = DI->getOperand(2);
   442:         int64_t V;
   443:         if (MRI->getVRegDef(IndReg) == Phi && checkForImmediate(Opnd2, V)) {
   444:           Register UpdReg = DI->getOperand(0).getReg();
   445:           IndMap.insert(std::make_pair(UpdReg, std::make_pair(IndReg, V)));
   446:         }
   447:       }
   448:     }  // for (i)
   449:   }  // for (instr)
   450: 
   451:   SmallVector<MachineOperand,2> Cond;
   452:   MachineBasicBlock *TB = nullptr, *FB = nullptr;
   453:   bool NotAnalyzed = TII->analyzeBranch(*ExitingBlock, TB, FB, Cond, false);
   454:   if (NotAnalyzed)
   455:     return false;
   456: 
   457:   Register PredR;
   458:   unsigned PredPos;
   459:   RegState PredRegFlags;
   460:   if (!TII->getPredReg(Cond, PredR, PredPos, PredRegFlags))
   461:     return false;
   462: 
   463:   MachineInstr *PredI = MRI->getVRegDef(PredR);
   464:   if (!PredI->isCompare())
   465:     return false;
   466: 
   467:   Register CmpReg1, CmpReg2;
   468:   int64_t CmpImm = 0, CmpMask = 0;
   469:   bool CmpAnalyzed =
   470:       TII->analyzeCompare(*PredI, CmpReg1, CmpReg2, CmpMask, CmpImm);
   471:   // Fail if the compare was not analyzed, or it's not comparing a register
   472:   // with an immediate value.  Not checking the mask here, since we handle
   473:   // the individual compare opcodes (including A4_cmpb*) later on.
   474:   if (!CmpAnalyzed)
   475:     return false;
   476: 
   477:   // Exactly one of the input registers to the comparison should be among
   478:   // the induction registers.
   479:   InductionMap::iterator IndMapEnd = IndMap.end();
   480:   InductionMap::iterator F = IndMapEnd;
   481:   if (CmpReg1 != 0) {
   482:     InductionMap::iterator F1 = IndMap.find(CmpReg1);
   483:     if (F1 != IndMapEnd)
   484:       F = F1;
   485:   }
   486:   if (CmpReg2 != 0) {
   487:     InductionMap::iterator F2 = IndMap.find(CmpReg2);
   488:     if (F2 != IndMapEnd) {
   489:       if (F != IndMapEnd)
   490:         return false;
   491:       F = F2;
   492:     }
   493:   }
   494:   if (F == IndMapEnd)
   495:     return false;
   496: 
   497:   Reg = F->second.first;
   498:   IVBump = F->second.second;
   499:   IVOp = MRI->getVRegDef(F->first);
   500:   return true;
   501: }
   502: 
   503: // Return the comparison kind for the specified opcode.
   504: HexagonHardwareLoops::Comparison::Kind
   505: HexagonHardwareLoops::getComparisonKind(unsigned CondOpc,
   506:                                         MachineOperand *InitialValue,
   507:                                         const MachineOperand *EndValue,
   508:                                         int64_t IVBump) const {
   509:   Comparison::Kind Cmp = (Comparison::Kind)0;
   510:   switch (CondOpc) {
   511:   case Hexagon::C2_cmpeq:
   512:   case Hexagon::C2_cmpeqi:
   513:   case Hexagon::C2_cmpeqp:
   514:     Cmp = Comparison::EQ;
   515:     break;
   516:   case Hexagon::C4_cmpneq:
   517:   case Hexagon::C4_cmpneqi:
   518:     Cmp = Comparison::NE;
   519:     break;
   520:   case Hexagon::C2_cmplt:
   521:     Cmp = Comparison::LTs;
   522:     break;
   523:   case Hexagon::C2_cmpltu:
   524:     Cmp = Comparison::LTu;
   525:     break;
   526:   case Hexagon::C4_cmplte:
   527:   case Hexagon::C4_cmpltei:
   528:     Cmp = Comparison::LEs;
   529:     break;
   530:   case Hexagon::C4_cmplteu:
   531:   case Hexagon::C4_cmplteui:
   532:     Cmp = Comparison::LEu;
   533:     break;
   534:   case Hexagon::C2_cmpgt:
   535:   case Hexagon::C2_cmpgti:
   536:   case Hexagon::C2_cmpgtp:
   537:     Cmp = Comparison::GTs;
   538:     break;
   539:   case Hexagon::C2_cmpgtu:
   540:   case Hexagon::C2_cmpgtui:
   541:   case Hexagon::C2_cmpgtup:
   542:     Cmp = Comparison::GTu;
   543:     break;
   544:   case Hexagon::C2_cmpgei:
   545:     Cmp = Comparison::GEs;
   546:     break;
   547:   case Hexagon::C2_cmpgeui:
   548:     Cmp = Comparison::GEs;
   549:     break;
   550:   default:
   551:     return (Comparison::Kind)0;
   552:   }
   553:   return Cmp;
   554: }
   555: 
   556: /// Analyze the statements in a loop to determine if the loop has
   557: /// a computable trip count and, if so, return a value that represents
   558: /// the trip count expression.
   559: ///
   560: /// This function iterates over the phi nodes in the loop to check for
   561: /// induction variable patterns that are used in the calculation for
   562: /// the number of time the loop is executed.
   563: CountValue *HexagonHardwareLoops::getLoopTripCount(MachineLoop *L,
   564:     SmallVectorImpl<MachineInstr *> &OldInsts) {
   565:   MachineBasicBlock *TopMBB = L->getTopBlock();
   566:   MachineBasicBlock::pred_iterator PI = TopMBB->pred_begin();
   567:   assert(PI != TopMBB->pred_end() &&
   568:          "Loop must have more than one incoming edge!");
   569:   MachineBasicBlock *Backedge = *PI++;
   570:   if (PI == TopMBB->pred_end())  // dead loop?
   571:     return nullptr;
   572:   MachineBasicBlock *Incoming = *PI++;
   573:   if (PI != TopMBB->pred_end())  // multiple backedges?
   574:     return nullptr;
   575: 
   576:   // Make sure there is one incoming and one backedge and determine which
   577:   // is which.
   578:   if (L->contains(Incoming)) {
   579:     if (L->contains(Backedge))
   580:       return nullptr;
   581:     std::swap(Incoming, Backedge);
   582:   } else if (!L->contains(Backedge))
   583:     return nullptr;
   584: 
   585:   // Look for the cmp instruction to determine if we can get a useful trip
   586:   // count.  The trip count can be either a register or an immediate.  The
   587:   // location of the value depends upon the type (reg or imm).
   588:   MachineBasicBlock *ExitingBlock = L->findLoopControlBlock();
   589:   if (!ExitingBlock)
   590:     return nullptr;
   591: 
   592:   Register IVReg = 0;
   593:   int64_t IVBump = 0;
   594:   MachineInstr *IVOp;
   595:   bool FoundIV = findInductionRegister(L, IVReg, IVBump, IVOp);
   596:   if (!FoundIV)
   597:     return nullptr;
   598: 
   599:   MachineBasicBlock *Preheader = MLI->findLoopPreheader(L, SpecPreheader);
   600: 
```
- EN: It declares or implements routines such as getHeader, findLoopPreheader, getLoopLatch, findLoopControlBlock, ... (20 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonHardwareLoops, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getHeader, findLoopPreheader, getLoopLatch, findLoopControlBlock, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonHardwareLoops，说明了它与同级后端组件的连接关系。

### Lines 601-800 / 第 601-800 行

```cpp
   601:   MachineOperand *InitialValue = nullptr;
   602:   MachineInstr *IV_Phi = MRI->getVRegDef(IVReg);
   603:   MachineBasicBlock *Latch = L->getLoopLatch();
   604:   for (unsigned i = 1, n = IV_Phi->getNumOperands(); i < n; i += 2) {
   605:     MachineBasicBlock *MBB = IV_Phi->getOperand(i+1).getMBB();
   606:     if (MBB == Preheader)
   607:       InitialValue = &IV_Phi->getOperand(i);
   608:     else if (MBB == Latch)
   609:       IVReg = IV_Phi->getOperand(i).getReg();  // Want IV reg after bump.
   610:   }
   611:   if (!InitialValue)
   612:     return nullptr;
   613: 
   614:   SmallVector<MachineOperand,2> Cond;
   615:   MachineBasicBlock *TB = nullptr, *FB = nullptr;
   616:   bool NotAnalyzed = TII->analyzeBranch(*ExitingBlock, TB, FB, Cond, false);
   617:   if (NotAnalyzed)
   618:     return nullptr;
   619: 
   620:   MachineBasicBlock *Header = L->getHeader();
   621:   // TB must be non-null.  If FB is also non-null, one of them must be
   622:   // the header.  Otherwise, branch to TB could be exiting the loop, and
   623:   // the fall through can go to the header.
   624:   assert (TB && "Exit block without a branch?");
   625:   if (ExitingBlock != Latch && (TB == Latch || FB == Latch)) {
   626:     MachineBasicBlock *LTB = nullptr, *LFB = nullptr;
   627:     SmallVector<MachineOperand,2> LCond;
   628:     bool NotAnalyzed = TII->analyzeBranch(*Latch, LTB, LFB, LCond, false);
   629:     if (NotAnalyzed)
   630:       return nullptr;
   631:     if (TB == Latch)
   632:       TB = (LTB == Header) ? LTB : LFB;
   633:     else
   634:       FB = (LTB == Header) ? LTB: LFB;
   635:   }
   636:   assert ((!FB || TB == Header || FB == Header) && "Branches not to header?");
   637:   if (!TB || (FB && TB != Header && FB != Header))
   638:     return nullptr;
   639: 
   640:   // Branches of form "if (!P) ..." cause HexagonInstrInfo::analyzeBranch
   641:   // to put imm(0), followed by P in the vector Cond.
   642:   // If TB is not the header, it means that the "not-taken" path must lead
   643:   // to the header.
   644:   bool Negated = TII->predOpcodeHasNot(Cond) ^ (TB != Header);
   645:   Register PredReg;
   646:   unsigned PredPos;
   647:   RegState PredRegFlags;
   648:   if (!TII->getPredReg(Cond, PredReg, PredPos, PredRegFlags))
   649:     return nullptr;
   650:   MachineInstr *CondI = MRI->getVRegDef(PredReg);
   651:   unsigned CondOpc = CondI->getOpcode();
   652: 
   653:   Register CmpReg1, CmpReg2;
   654:   int64_t Mask = 0, ImmValue = 0;
   655:   bool AnalyzedCmp =
   656:       TII->analyzeCompare(*CondI, CmpReg1, CmpReg2, Mask, ImmValue);
   657:   if (!AnalyzedCmp)
   658:     return nullptr;
   659: 
   660:   // The comparison operator type determines how we compute the loop
   661:   // trip count.
   662:   OldInsts.push_back(CondI);
   663:   OldInsts.push_back(IVOp);
   664: 
   665:   // Sadly, the following code gets information based on the position
   666:   // of the operands in the compare instruction.  This has to be done
   667:   // this way, because the comparisons check for a specific relationship
   668:   // between the operands (e.g. is-less-than), rather than to find out
   669:   // what relationship the operands are in (as on PPC).
   670:   Comparison::Kind Cmp;
   671:   bool isSwapped = false;
   672:   const MachineOperand &Op1 = CondI->getOperand(1);
   673:   const MachineOperand &Op2 = CondI->getOperand(2);
   674:   const MachineOperand *EndValue = nullptr;
   675: 
   676:   if (Op1.isReg()) {
   677:     if (Op2.isImm() || Op1.getReg() == IVReg)
   678:       EndValue = &Op2;
   679:     else {
   680:       EndValue = &Op1;
   681:       isSwapped = true;
   682:     }
   683:   }
   684: 
   685:   if (!EndValue)
   686:     return nullptr;
   687: 
   688:   Cmp = getComparisonKind(CondOpc, InitialValue, EndValue, IVBump);
   689:   if (!Cmp)
   690:     return nullptr;
   691:   if (Negated)
   692:     Cmp = Comparison::getNegatedComparison(Cmp);
   693:   if (isSwapped)
   694:     Cmp = Comparison::getSwappedComparison(Cmp);
   695: 
   696:   if (InitialValue->isReg()) {
   697:     Register R = InitialValue->getReg();
   698:     MachineBasicBlock *DefBB = MRI->getVRegDef(R)->getParent();
   699:     if (!MDT->properlyDominates(DefBB, Header)) {
   700:       int64_t V;
   701:       if (!checkForImmediate(*InitialValue, V))
   702:         return nullptr;
   703:     }
   704:     OldInsts.push_back(MRI->getVRegDef(R));
   705:   }
   706:   if (EndValue->isReg()) {
   707:     Register R = EndValue->getReg();
   708:     MachineBasicBlock *DefBB = MRI->getVRegDef(R)->getParent();
   709:     if (!MDT->properlyDominates(DefBB, Header)) {
   710:       int64_t V;
   711:       if (!checkForImmediate(*EndValue, V))
   712:         return nullptr;
   713:     }
   714:     OldInsts.push_back(MRI->getVRegDef(R));
   715:   }
   716: 
   717:   return computeCount(L, InitialValue, EndValue, IVReg, IVBump, Cmp);
   718: }
   719: 
   720: /// Helper function that returns the expression that represents the
   721: /// number of times a loop iterates.  The function takes the operands that
   722: /// represent the loop start value, loop end value, and induction value.
   723: /// Based upon these operands, the function attempts to compute the trip count.
   724: CountValue *HexagonHardwareLoops::computeCount(MachineLoop *Loop,
   725:                                                const MachineOperand *Start,
   726:                                                const MachineOperand *End,
   727:                                                Register IVReg,
   728:                                                int64_t IVBump,
   729:                                                Comparison::Kind Cmp) const {
   730:   LLVM_DEBUG(llvm::dbgs() << "Loop: " << *Loop << "\n");
   731:   LLVM_DEBUG(llvm::dbgs() << "Initial Value: " << *Start << "\n");
   732:   LLVM_DEBUG(llvm::dbgs() << "End Value: " << *End << "\n");
   733:   LLVM_DEBUG(llvm::dbgs() << "Inc/Dec Value: " << IVBump << "\n");
   734:   LLVM_DEBUG(llvm::dbgs() << "Comparison: " << Cmp << "\n");
   735:   // Cannot handle comparison EQ, i.e. while (A == B).
   736:   if (Cmp == Comparison::EQ)
   737:     return nullptr;
   738: 
   739:   // Check if either the start or end values are an assignment of an immediate.
   740:   // If so, use the immediate value rather than the register.
   741:   if (Start->isReg()) {
   742:     const MachineInstr *StartValInstr = MRI->getVRegDef(Start->getReg());
   743:     if (StartValInstr && (StartValInstr->getOpcode() == Hexagon::A2_tfrsi ||
   744:                           StartValInstr->getOpcode() == Hexagon::A2_tfrpi))
   745:       Start = &StartValInstr->getOperand(1);
   746:   }
   747:   if (End->isReg()) {
   748:     const MachineInstr *EndValInstr = MRI->getVRegDef(End->getReg());
   749:     if (EndValInstr && (EndValInstr->getOpcode() == Hexagon::A2_tfrsi ||
   750:                         EndValInstr->getOpcode() == Hexagon::A2_tfrpi))
   751:       End = &EndValInstr->getOperand(1);
   752:   }
   753: 
   754:   if (!Start->isReg() && !Start->isImm())
   755:     return nullptr;
   756:   if (!End->isReg() && !End->isImm())
   757:     return nullptr;
   758: 
   759:   bool CmpLess =     Cmp & Comparison::L;
   760:   bool CmpGreater =  Cmp & Comparison::G;
   761:   bool CmpHasEqual = Cmp & Comparison::EQ;
   762: 
   763:   // Avoid certain wrap-arounds.  This doesn't detect all wrap-arounds.
   764:   if (CmpLess && IVBump < 0)
   765:     // Loop going while iv is "less" with the iv value going down.  Must wrap.
   766:     return nullptr;
   767: 
   768:   if (CmpGreater && IVBump > 0)
   769:     // Loop going while iv is "greater" with the iv value going up.  Must wrap.
   770:     return nullptr;
   771: 
   772:   // Phis that may feed into the loop.
   773:   LoopFeederMap LoopFeederPhi;
   774: 
   775:   // Check if the initial value may be zero and can be decremented in the first
   776:   // iteration. If the value is zero, the endloop instruction will not decrement
   777:   // the loop counter, so we shouldn't generate a hardware loop in this case.
   778:   if (loopCountMayWrapOrUnderFlow(Start, End, Loop->getLoopPreheader(), Loop,
   779:                                   LoopFeederPhi))
   780:       return nullptr;
   781: 
   782:   if (Start->isImm() && End->isImm()) {
   783:     // Both, start and end are immediates.
   784:     int64_t StartV = Start->getImm();
   785:     int64_t EndV = End->getImm();
   786:     int64_t Dist = EndV - StartV;
   787:     if (Dist == 0)
   788:       return nullptr;
   789: 
   790:     bool Exact = (Dist % IVBump) == 0;
   791: 
   792:     if (Cmp == Comparison::NE) {
   793:       if (!Exact)
   794:         return nullptr;
   795:       if ((Dist < 0) ^ (IVBump < 0))
   796:         return nullptr;
   797:     }
   798: 
   799:     // For comparisons that include the final value (i.e. include equality
   800:     // with the final value), we need to increase the distance by 1.
```
- EN: It declares or implements routines such as getVRegDef, getLoopLatch, getOperand, analyzeBranch, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstrInfo, HexagonHardwareLoops, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getVRegDef, getLoopLatch, getOperand, analyzeBranch, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo, HexagonHardwareLoops，说明了它与同级后端组件的连接关系。

### Lines 801-1000 / 第 801-1000 行

```cpp
   801:     if (CmpHasEqual)
   802:       Dist = Dist > 0 ? Dist+1 : Dist-1;
   803: 
   804:     // For the loop to iterate, CmpLess should imply Dist > 0.  Similarly,
   805:     // CmpGreater should imply Dist < 0.  These conditions could actually
   806:     // fail, for example, in unreachable code (which may still appear to be
   807:     // reachable in the CFG).
   808:     if ((CmpLess && Dist < 0) || (CmpGreater && Dist > 0))
   809:       return nullptr;
   810: 
   811:     // "Normalized" distance, i.e. with the bump set to +-1.
   812:     int64_t Dist1 = (IVBump > 0) ? (Dist +  (IVBump - 1)) / IVBump
   813:                                  : (-Dist + (-IVBump - 1)) / (-IVBump);
   814:     assert (Dist1 > 0 && "Fishy thing.  Both operands have the same sign.");
   815: 
   816:     uint64_t Count = Dist1;
   817: 
   818:     if (Count > 0xFFFFFFFFULL)
   819:       return nullptr;
   820: 
   821:     return new CountValue(CountValue::CV_Immediate, Count);
   822:   }
   823: 
   824:   // A general case: Start and End are some values, but the actual
   825:   // iteration count may not be available.  If it is not, insert
   826:   // a computation of it into the preheader.
   827: 
   828:   // If the induction variable bump is not a power of 2, quit.
   829:   // Otherwise we'd need a general integer division.
   830:   if (!isPowerOf2_64(std::abs(IVBump)))
   831:     return nullptr;
   832: 
   833:   MachineBasicBlock *PH = MLI->findLoopPreheader(Loop, SpecPreheader);
   834:   assert (PH && "Should have a preheader by now");
   835:   MachineBasicBlock::iterator InsertPos = PH->getFirstTerminator();
   836:   DebugLoc DL;
   837:   if (InsertPos != PH->end())
   838:     DL = InsertPos->getDebugLoc();
   839: 
   840:   // If Start is an immediate and End is a register, the trip count
   841:   // will be "reg - imm".  Hexagon's "subtract immediate" instruction
   842:   // is actually "reg + -imm".
   843: 
   844:   // If the loop IV is going downwards, i.e. if the bump is negative,
   845:   // then the iteration count (computed as End-Start) will need to be
   846:   // negated.  To avoid the negation, just swap Start and End.
   847:   if (IVBump < 0) {
   848:     std::swap(Start, End);
   849:     IVBump = -IVBump;
   850:     std::swap(CmpLess, CmpGreater);
   851:   }
   852:   // Cmp may now have a wrong direction, e.g.  LEs may now be GEs.
   853:   // Signedness, and "including equality" are preserved.
   854: 
   855:   bool RegToImm = Start->isReg() && End->isImm(); // for (reg..imm)
   856:   bool RegToReg = Start->isReg() && End->isReg(); // for (reg..reg)
   857: 
   858:   int64_t StartV = 0, EndV = 0;
   859:   if (Start->isImm())
   860:     StartV = Start->getImm();
   861:   if (End->isImm())
   862:     EndV = End->getImm();
   863: 
   864:   int64_t AdjV = 0;
   865:   // To compute the iteration count, we would need this computation:
   866:   //   Count = (End - Start + (IVBump-1)) / IVBump
   867:   // or, when CmpHasEqual:
   868:   //   Count = (End - Start + (IVBump-1)+1) / IVBump
   869:   // The "IVBump-1" part is the adjustment (AdjV).  We can avoid
   870:   // generating an instruction specifically to add it if we can adjust
   871:   // the immediate values for Start or End.
   872: 
   873:   if (CmpHasEqual) {
   874:     // Need to add 1 to the total iteration count.
   875:     if (Start->isImm())
   876:       StartV--;
   877:     else if (End->isImm())
   878:       EndV++;
   879:     else
   880:       AdjV += 1;
   881:   }
   882: 
   883:   if (Cmp != Comparison::NE) {
   884:     if (Start->isImm())
   885:       StartV -= (IVBump-1);
   886:     else if (End->isImm())
   887:       EndV += (IVBump-1);
   888:     else
   889:       AdjV += (IVBump-1);
   890:   }
   891: 
   892:   Register R = 0;
   893:   unsigned SR = 0;
   894:   if (Start->isReg()) {
   895:     R = Start->getReg();
   896:     SR = Start->getSubReg();
   897:   } else {
   898:     R = End->getReg();
   899:     SR = End->getSubReg();
   900:   }
   901:   const TargetRegisterClass *RC = MRI->getRegClass(R);
   902:   // Hardware loops cannot handle 64-bit registers.  If it's a double
   903:   // register, it has to have a subregister.
   904:   if (!SR && RC == &Hexagon::DoubleRegsRegClass)
   905:     return nullptr;
   906:   const TargetRegisterClass *IntRC = &Hexagon::IntRegsRegClass;
   907: 
   908:   // Compute DistR (register with the distance between Start and End).
   909:   Register DistR;
   910:   unsigned DistSR;
   911: 
   912:   // Avoid special case, where the start value is an imm(0).
   913:   if (Start->isImm() && StartV == 0) {
   914:     DistR = End->getReg();
   915:     DistSR = End->getSubReg();
   916:   } else {
   917:     const MCInstrDesc &SubD = RegToReg ? TII->get(Hexagon::A2_sub) :
   918:                               (RegToImm ? TII->get(Hexagon::A2_subri) :
   919:                                           TII->get(Hexagon::A2_addi));
   920:     if (RegToReg || RegToImm) {
   921:       Register SubR = MRI->createVirtualRegister(IntRC);
   922:       MachineInstrBuilder SubIB =
   923:         BuildMI(*PH, InsertPos, DL, SubD, SubR);
   924: 
   925:       if (RegToReg)
   926:         SubIB.addReg(End->getReg(), {}, End->getSubReg())
   927:             .addReg(Start->getReg(), {}, Start->getSubReg());
   928:       else
   929:         SubIB.addImm(EndV).addReg(Start->getReg(), {}, Start->getSubReg());
   930:       DistR = SubR;
   931:     } else {
   932:       // If the loop has been unrolled, we should use the original loop count
   933:       // instead of recalculating the value. This will avoid additional
   934:       // 'Add' instruction.
   935:       const MachineInstr *EndValInstr = MRI->getVRegDef(End->getReg());
   936:       if (EndValInstr->getOpcode() == Hexagon::A2_addi &&
   937:           EndValInstr->getOperand(1).getSubReg() == 0 &&
   938:           EndValInstr->getOperand(2).getImm() == StartV) {
   939:         DistR = EndValInstr->getOperand(1).getReg();
   940:       } else {
   941:         Register SubR = MRI->createVirtualRegister(IntRC);
   942:         MachineInstrBuilder SubIB =
   943:           BuildMI(*PH, InsertPos, DL, SubD, SubR);
   944:         SubIB.addReg(End->getReg(), {}, End->getSubReg()).addImm(-StartV);
   945:         DistR = SubR;
   946:       }
   947:     }
   948:     DistSR = 0;
   949:   }
   950: 
   951:   // From DistR, compute AdjR (register with the adjusted distance).
   952:   Register AdjR;
   953:   unsigned AdjSR;
   954: 
   955:   if (AdjV == 0) {
   956:     AdjR = DistR;
   957:     AdjSR = DistSR;
   958:   } else {
   959:     // Generate CountR = ADD DistR, AdjVal
   960:     Register AddR = MRI->createVirtualRegister(IntRC);
   961:     MCInstrDesc const &AddD = TII->get(Hexagon::A2_addi);
   962:     BuildMI(*PH, InsertPos, DL, AddD, AddR)
   963:         .addReg(DistR, {}, DistSR)
   964:         .addImm(AdjV);
   965: 
   966:     AdjR = AddR;
   967:     AdjSR = 0;
   968:   }
   969: 
   970:   // From AdjR, compute CountR (register with the final count).
   971:   Register CountR;
   972:   unsigned CountSR;
   973: 
   974:   if (IVBump == 1) {
   975:     CountR = AdjR;
   976:     CountSR = AdjSR;
   977:   } else {
   978:     // The IV bump is a power of two. Log_2(IV bump) is the shift amount.
   979:     unsigned Shift = Log2_32(IVBump);
   980: 
   981:     // Generate NormR = LSR DistR, Shift.
   982:     Register LsrR = MRI->createVirtualRegister(IntRC);
   983:     const MCInstrDesc &LsrD = TII->get(Hexagon::S2_lsr_i_r);
   984:     BuildMI(*PH, InsertPos, DL, LsrD, LsrR)
   985:         .addReg(AdjR, {}, AdjSR)
   986:         .addImm(Shift);
   987: 
   988:     CountR = LsrR;
   989:     CountSR = 0;
   990:   }
   991: 
   992:   const TargetRegisterClass *PredRC = &Hexagon::PredRegsRegClass;
   993:   Register MuxR = CountR;
   994:   unsigned MuxSR = CountSR;
   995:   // For the loop count to be valid unsigned number, CmpLess should imply
   996:   // Dist >= 0. Similarly, CmpGreater should imply Dist < 0. We can skip the
   997:   // check if the initial distance is zero and the comparison is LTu || LTEu.
   998:   if (!(Start->isImm() && StartV == 0 && Comparison::isUnsigned(Cmp) &&
   999:         CmpLess) &&
  1000:       (CmpLess || CmpGreater)) {
```
- EN: It declares or implements routines such as assert, CountValue, findLoopPreheader, getFirstTerminator, ... (19 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 assert, CountValue, findLoopPreheader, getFirstTerminator, ... (19 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 1001-1200 / 第 1001-1200 行

```cpp
  1001:     // Generate:
  1002:     //   DistCheck = CMP_GT DistR,  0   --> CmpLess
  1003:     //   DistCheck = CMP_GT DistR, -1   --> CmpGreater
  1004:     Register DistCheckR = MRI->createVirtualRegister(PredRC);
  1005:     const MCInstrDesc &DistCheckD = TII->get(Hexagon::C2_cmpgti);
  1006:     BuildMI(*PH, InsertPos, DL, DistCheckD, DistCheckR)
  1007:         .addReg(DistR, {}, DistSR)
  1008:         .addImm((CmpLess) ? 0 : -1);
  1009: 
  1010:     // Generate:
  1011:     //   MUXR = MUX DistCheck, CountR, 1   --> CmpLess
  1012:     //   MUXR = MUX DistCheck, 1, CountR   --> CmpGreater
  1013:     MuxR = MRI->createVirtualRegister(IntRC);
  1014:     if (CmpLess) {
  1015:       const MCInstrDesc &MuxD = TII->get(Hexagon::C2_muxir);
  1016:       BuildMI(*PH, InsertPos, DL, MuxD, MuxR)
  1017:           .addReg(DistCheckR)
  1018:           .addReg(CountR, {}, CountSR)
  1019:           .addImm(1);
  1020:     } else {
  1021:       const MCInstrDesc &MuxD = TII->get(Hexagon::C2_muxri);
  1022:       BuildMI(*PH, InsertPos, DL, MuxD, MuxR)
  1023:           .addReg(DistCheckR)
  1024:           .addImm(1)
  1025:           .addReg(CountR, {}, CountSR);
  1026:     }
  1027:     MuxSR = 0;
  1028:   }
  1029: 
  1030:   return new CountValue(CountValue::CV_Register, MuxR, MuxSR);
  1031: }
  1032: 
  1033: /// Return true if the operation is invalid within hardware loop.
  1034: bool HexagonHardwareLoops::isInvalidLoopOperation(const MachineInstr *MI,
  1035:                                                   bool IsInnerHWLoop) const {
  1036:   // Call is not allowed because the callee may use a hardware loop except for
  1037:   // the case when the call never returns.
  1038:   if (MI->getDesc().isCall())
  1039:     return !TII->doesNotReturn(*MI);
  1040: 
  1041:   // Check if the instruction defines a hardware loop register.
  1042:   using namespace Hexagon;
  1043: 
  1044:   static const Register Regs01[] = { LC0, SA0, LC1, SA1 };
  1045:   static const Register Regs1[]  = { LC1, SA1 };
  1046:   auto CheckRegs = IsInnerHWLoop ? ArrayRef(Regs01) : ArrayRef(Regs1);
  1047:   for (Register R : CheckRegs)
  1048:     if (MI->modifiesRegister(R, TRI))
  1049:       return true;
  1050: 
  1051:   return false;
  1052: }
  1053: 
  1054: /// Return true if the loop contains an instruction that inhibits
  1055: /// the use of the hardware loop instruction.
  1056: bool HexagonHardwareLoops::containsInvalidInstruction(MachineLoop *L,
  1057:     bool IsInnerHWLoop) const {
  1058:   LLVM_DEBUG(dbgs() << "\nhw_loop head, "
  1059:                     << printMBBReference(**L->block_begin()));
  1060:   for (MachineBasicBlock *MBB : L->getBlocks()) {
  1061:     for (const MachineInstr &MI : *MBB) {
  1062:       if (isInvalidLoopOperation(&MI, IsInnerHWLoop)) {
  1063:         LLVM_DEBUG(dbgs() << "\nCannot convert to hw_loop due to:";
  1064:                    MI.dump(););
  1065:         return true;
  1066:       }
  1067:     }
  1068:   }
  1069:   return false;
  1070: }
  1071: 
  1072: /// Returns true if the instruction is dead.  This was essentially
  1073: /// copied from DeadMachineInstructionElim::isDead, but with special cases
  1074: /// for inline asm, physical registers and instructions with side effects
  1075: /// removed.
  1076: bool HexagonHardwareLoops::isDead(const MachineInstr *MI,
  1077:                               SmallVectorImpl<MachineInstr *> &DeadPhis) const {
  1078:   // Examine each operand.
  1079:   for (const MachineOperand &MO : MI->operands()) {
  1080:     if (!MO.isReg() || !MO.isDef())
  1081:       continue;
  1082: 
  1083:     Register Reg = MO.getReg();
  1084:     if (MRI->use_nodbg_empty(Reg))
  1085:       continue;
  1086: 
  1087:     using use_nodbg_iterator = MachineRegisterInfo::use_nodbg_iterator;
  1088: 
  1089:     // This instruction has users, but if the only user is the phi node for the
  1090:     // parent block, and the only use of that phi node is this instruction, then
  1091:     // this instruction is dead: both it (and the phi node) can be removed.
  1092:     use_nodbg_iterator I = MRI->use_nodbg_begin(Reg);
  1093:     use_nodbg_iterator End = MRI->use_nodbg_end();
  1094:     if (std::next(I) != End || !I->getParent()->isPHI())
  1095:       return false;
  1096: 
  1097:     MachineInstr *OnePhi = I->getParent();
  1098:     for (const MachineOperand &OPO : OnePhi->operands()) {
  1099:       if (!OPO.isReg() || !OPO.isDef())
  1100:         continue;
  1101: 
  1102:       Register OPReg = OPO.getReg();
  1103:       use_nodbg_iterator nextJ;
  1104:       for (use_nodbg_iterator J = MRI->use_nodbg_begin(OPReg);
  1105:            J != End; J = nextJ) {
  1106:         nextJ = std::next(J);
  1107:         MachineOperand &Use = *J;
  1108:         MachineInstr *UseMI = Use.getParent();
  1109: 
  1110:         // If the phi node has a user that is not MI, bail.
  1111:         if (MI != UseMI)
  1112:           return false;
  1113:       }
  1114:     }
  1115:     DeadPhis.push_back(OnePhi);
  1116:   }
  1117: 
  1118:   // If there are no defs with uses, the instruction is dead.
  1119:   return true;
  1120: }
  1121: 
  1122: void HexagonHardwareLoops::removeIfDead(MachineInstr *MI) {
  1123:   // This procedure was essentially copied from DeadMachineInstructionElim.
  1124: 
  1125:   SmallVector<MachineInstr*, 1> DeadPhis;
  1126:   if (isDead(MI, DeadPhis)) {
  1127:     LLVM_DEBUG(dbgs() << "HW looping will remove: " << *MI);
  1128: 
  1129:     // It is possible that some DBG_VALUE instructions refer to this
  1130:     // instruction.  Examine each def operand for such references;
  1131:     // if found, mark the DBG_VALUE as undef (but don't delete it).
  1132:     for (const MachineOperand &MO : MI->operands()) {
  1133:       if (!MO.isReg() || !MO.isDef())
  1134:         continue;
  1135:       Register Reg = MO.getReg();
  1136:       // We use make_early_inc_range here because setReg below invalidates the
  1137:       // iterator.
  1138:       for (MachineOperand &MO :
  1139:            llvm::make_early_inc_range(MRI->use_operands(Reg))) {
  1140:         MachineInstr *UseMI = MO.getParent();
  1141:         if (UseMI == MI)
  1142:           continue;
  1143:         if (MO.isDebug())
  1144:           MO.setReg(0U);
  1145:       }
  1146:     }
  1147: 
  1148:     MI->eraseFromParent();
  1149:     for (unsigned i = 0; i < DeadPhis.size(); ++i)
  1150:       DeadPhis[i]->eraseFromParent();
  1151:   }
  1152: }
  1153: 
  1154: /// Check if the loop is a candidate for converting to a hardware
  1155: /// loop.  If so, then perform the transformation.
  1156: ///
  1157: /// This function works on innermost loops first.  A loop can be converted
  1158: /// if it is a counting loop; either a register value or an immediate.
  1159: ///
  1160: /// The code makes several assumptions about the representation of the loop
  1161: /// in llvm.
  1162: bool HexagonHardwareLoops::convertToHardwareLoop(MachineLoop *L,
  1163:                                                  bool &RecL0used,
  1164:                                                  bool &RecL1used) {
  1165:   // This is just to confirm basic correctness.
  1166:   assert(L->getHeader() && "Loop without a header?");
  1167: 
  1168:   bool Changed = false;
  1169:   bool L0Used = false;
  1170:   bool L1Used = false;
  1171: 
  1172:   // Process nested loops first.
  1173:   for (MachineLoop *I : *L) {
  1174:     Changed |= convertToHardwareLoop(I, RecL0used, RecL1used);
  1175:     L0Used |= RecL0used;
  1176:     L1Used |= RecL1used;
  1177:   }
  1178: 
  1179:   // If a nested loop has been converted, then we can't convert this loop.
  1180:   if (Changed && L0Used && L1Used)
  1181:     return Changed;
  1182: 
  1183:   unsigned LOOP_i;
  1184:   unsigned LOOP_r;
  1185:   unsigned ENDLOOP;
  1186: 
  1187:   // Flag used to track loopN instruction:
  1188:   // 1 - Hardware loop is being generated for the inner most loop.
  1189:   // 0 - Hardware loop is being generated for the outer loop.
  1190:   unsigned IsInnerHWLoop = 1;
  1191: 
  1192:   if (L0Used) {
  1193:     LOOP_i = Hexagon::J2_loop1i;
  1194:     LOOP_r = Hexagon::J2_loop1r;
  1195:     ENDLOOP = Hexagon::ENDLOOP1;
  1196:     IsInnerHWLoop = 0;
  1197:   } else {
  1198:     LOOP_i = Hexagon::J2_loop0i;
  1199:     LOOP_r = Hexagon::J2_loop0r;
  1200:     ENDLOOP = Hexagon::ENDLOOP0;
```
- EN: It opens namespaces (Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as createVirtualRegister, get, addImm, CountValue, ... (23 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 createVirtualRegister, get, addImm, CountValue, ... (23 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1201-1400 / 第 1201-1400 行

```cpp
  1201:   }
  1202: 
  1203: #ifndef NDEBUG
  1204:   // Stop trying after reaching the limit (if any).
  1205:   int Limit = HWLoopLimit;
  1206:   if (Limit >= 0) {
  1207:     if (Counter >= HWLoopLimit)
  1208:       return false;
  1209:     Counter++;
  1210:   }
  1211: #endif
  1212: 
  1213:   // Does the loop contain any invalid instructions?
  1214:   if (containsInvalidInstruction(L, IsInnerHWLoop))
  1215:     return false;
  1216: 
  1217:   MachineBasicBlock *LastMBB = L->findLoopControlBlock();
  1218:   // Don't generate hw loop if the loop has more than one exit.
  1219:   if (!LastMBB)
  1220:     return false;
  1221: 
  1222:   MachineBasicBlock::iterator LastI = LastMBB->getFirstTerminator();
  1223:   if (LastI == LastMBB->end())
  1224:     return false;
  1225: 
  1226:   // Is the induction variable bump feeding the latch condition?
  1227:   if (!fixupInductionVariable(L))
  1228:     return false;
  1229: 
  1230:   // Ensure the loop has a preheader: the loop instruction will be
  1231:   // placed there.
  1232:   MachineBasicBlock *Preheader = MLI->findLoopPreheader(L, SpecPreheader);
  1233:   if (!Preheader) {
  1234:     Preheader = createPreheaderForLoop(L);
  1235:     if (!Preheader)
  1236:       return false;
  1237:   }
  1238: 
  1239:   MachineBasicBlock::iterator InsertPos = Preheader->getFirstTerminator();
  1240: 
  1241:   SmallVector<MachineInstr*, 2> OldInsts;
  1242:   // Are we able to determine the trip count for the loop?
  1243:   CountValue *TripCount = getLoopTripCount(L, OldInsts);
  1244:   if (!TripCount)
  1245:     return false;
  1246: 
  1247:   // Is the trip count available in the preheader?
  1248:   if (TripCount->isReg()) {
  1249:     // There will be a use of the register inserted into the preheader,
  1250:     // so make sure that the register is actually defined at that point.
  1251:     MachineInstr *TCDef = MRI->getVRegDef(TripCount->getReg());
  1252:     MachineBasicBlock *BBDef = TCDef->getParent();
  1253:     if (!MDT->dominates(BBDef, Preheader))
  1254:       return false;
  1255:   }
  1256: 
  1257:   // Determine the loop start.
  1258:   MachineBasicBlock *TopBlock = L->getTopBlock();
  1259:   MachineBasicBlock *ExitingBlock = L->findLoopControlBlock();
  1260:   MachineBasicBlock *LoopStart = nullptr;
  1261:   if (ExitingBlock !=  L->getLoopLatch()) {
  1262:     MachineBasicBlock *TB = nullptr, *FB = nullptr;
  1263:     SmallVector<MachineOperand, 2> Cond;
  1264: 
  1265:     if (TII->analyzeBranch(*ExitingBlock, TB, FB, Cond, false))
  1266:       return false;
  1267: 
  1268:     if (L->contains(TB))
  1269:       LoopStart = TB;
  1270:     else if (L->contains(FB))
  1271:       LoopStart = FB;
  1272:     else
  1273:       return false;
  1274:   }
  1275:   else
  1276:     LoopStart = TopBlock;
  1277: 
  1278:   // Convert the loop to a hardware loop.
  1279:   LLVM_DEBUG(dbgs() << "Change to hardware loop at "; L->dump());
  1280:   DebugLoc DL;
  1281:   if (InsertPos != Preheader->end())
  1282:     DL = InsertPos->getDebugLoc();
  1283: 
  1284:   if (TripCount->isReg()) {
  1285:     // Create a copy of the loop count register.
  1286:     Register CountReg = MRI->createVirtualRegister(&Hexagon::IntRegsRegClass);
  1287:     BuildMI(*Preheader, InsertPos, DL, TII->get(TargetOpcode::COPY), CountReg)
  1288:         .addReg(TripCount->getReg(), {}, TripCount->getSubReg());
  1289:     // Add the Loop instruction to the beginning of the loop.
  1290:     BuildMI(*Preheader, InsertPos, DL, TII->get(LOOP_r)).addMBB(LoopStart)
  1291:       .addReg(CountReg);
  1292:   } else {
  1293:     assert(TripCount->isImm() && "Expecting immediate value for trip count");
  1294:     // Add the Loop immediate instruction to the beginning of the loop,
  1295:     // if the immediate fits in the instructions.  Otherwise, we need to
  1296:     // create a new virtual register.
  1297:     int64_t CountImm = TripCount->getImm();
  1298:     if (!TII->isValidOffset(LOOP_i, CountImm, TRI)) {
  1299:       Register CountReg = MRI->createVirtualRegister(&Hexagon::IntRegsRegClass);
  1300:       BuildMI(*Preheader, InsertPos, DL, TII->get(Hexagon::A2_tfrsi), CountReg)
  1301:         .addImm(CountImm);
  1302:       BuildMI(*Preheader, InsertPos, DL, TII->get(LOOP_r))
  1303:         .addMBB(LoopStart).addReg(CountReg);
  1304:     } else
  1305:       BuildMI(*Preheader, InsertPos, DL, TII->get(LOOP_i))
  1306:         .addMBB(LoopStart).addImm(CountImm);
  1307:   }
  1308: 
  1309:   // Make sure the loop start always has a reference in the CFG.
  1310:   LoopStart->setMachineBlockAddressTaken();
  1311: 
  1312:   // Replace the loop branch with an endloop instruction.
  1313:   DebugLoc LastIDL = LastI->getDebugLoc();
  1314:   BuildMI(*LastMBB, LastI, LastIDL, TII->get(ENDLOOP)).addMBB(LoopStart);
  1315: 
  1316:   // The loop ends with either:
  1317:   //  - a conditional branch followed by an unconditional branch, or
  1318:   //  - a conditional branch to the loop start.
  1319:   if (LastI->getOpcode() == Hexagon::J2_jumpt ||
  1320:       LastI->getOpcode() == Hexagon::J2_jumpf) {
  1321:     // Delete one and change/add an uncond. branch to out of the loop.
  1322:     MachineBasicBlock *BranchTarget = LastI->getOperand(1).getMBB();
  1323:     LastI = LastMBB->erase(LastI);
  1324:     if (!L->contains(BranchTarget)) {
  1325:       if (LastI != LastMBB->end())
  1326:         LastI = LastMBB->erase(LastI);
  1327:       SmallVector<MachineOperand, 0> Cond;
  1328:       TII->insertBranch(*LastMBB, BranchTarget, nullptr, Cond, LastIDL);
  1329:     }
  1330:   } else {
  1331:     // Conditional branch to loop start; just delete it.
  1332:     LastMBB->erase(LastI);
  1333:   }
  1334:   delete TripCount;
  1335: 
  1336:   // The induction operation and the comparison may now be
  1337:   // unneeded. If these are unneeded, then remove them.
  1338:   for (unsigned i = 0; i < OldInsts.size(); ++i)
  1339:     removeIfDead(OldInsts[i]);
  1340: 
  1341:   ++NumHWLoops;
  1342: 
  1343:   // Set RecL1used and RecL0used only after hardware loop has been
  1344:   // successfully generated. Doing it earlier can cause wrong loop instruction
  1345:   // to be used.
  1346:   if (L0Used) // Loop0 was already used. So, the correct loop must be loop1.
  1347:     RecL1used = true;
  1348:   else
  1349:     RecL0used = true;
  1350: 
  1351:   return true;
  1352: }
  1353: 
  1354: bool HexagonHardwareLoops::orderBumpCompare(MachineInstr *BumpI,
  1355:                                             MachineInstr *CmpI) {
  1356:   assert (BumpI != CmpI && "Bump and compare in the same instruction?");
  1357: 
  1358:   MachineBasicBlock *BB = BumpI->getParent();
  1359:   if (CmpI->getParent() != BB)
  1360:     return false;
  1361: 
  1362:   using instr_iterator = MachineBasicBlock::instr_iterator;
  1363: 
  1364:   // Check if things are in order to begin with.
  1365:   for (instr_iterator I(BumpI), E = BB->instr_end(); I != E; ++I)
  1366:     if (&*I == CmpI)
  1367:       return true;
  1368: 
  1369:   // Out of order.
  1370:   Register PredR = CmpI->getOperand(0).getReg();
  1371:   bool FoundBump = false;
  1372:   instr_iterator CmpIt = CmpI->getIterator(), NextIt = std::next(CmpIt);
  1373:   for (instr_iterator I = NextIt, E = BB->instr_end(); I != E; ++I) {
  1374:     MachineInstr *In = &*I;
  1375:     for (unsigned i = 0, n = In->getNumOperands(); i < n; ++i) {
  1376:       MachineOperand &MO = In->getOperand(i);
  1377:       if (MO.isReg() && MO.isUse()) {
  1378:         if (MO.getReg() == PredR)  // Found an intervening use of PredR.
  1379:           return false;
  1380:       }
  1381:     }
  1382: 
  1383:     if (In == BumpI) {
  1384:       BB->splice(++BumpI->getIterator(), BB, CmpI->getIterator());
  1385:       FoundBump = true;
  1386:       break;
  1387:     }
  1388:   }
  1389:   assert (FoundBump && "Cannot determine instruction order");
  1390:   return FoundBump;
  1391: }
  1392: 
  1393: /// This function is required to break recursion. Visiting phis in a loop may
  1394: /// result in recursion during compilation. We break the recursion by making
  1395: /// sure that we visit a MachineOperand and its definition in a
  1396: /// MachineInstruction only once. If we attempt to visit more than once, then
  1397: /// there is recursion, and will return false.
  1398: bool HexagonHardwareLoops::isLoopFeeder(MachineLoop *L, MachineBasicBlock *A,
  1399:                                         MachineInstr *MI,
  1400:                                         const MachineOperand *MO,
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as findLoopControlBlock, getFirstTerminator, findLoopPreheader, createPreheaderForLoop, ... (24 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 findLoopControlBlock, getFirstTerminator, findLoopPreheader, createPreheaderForLoop, ... (24 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1401-1600 / 第 1401-1600 行

```cpp
  1401:                                         LoopFeederMap &LoopFeederPhi) const {
  1402:   if (LoopFeederPhi.find(MO->getReg()) == LoopFeederPhi.end()) {
  1403:     LLVM_DEBUG(dbgs() << "\nhw_loop head, "
  1404:                       << printMBBReference(**L->block_begin()));
  1405:     // Ignore all BBs that form Loop.
  1406:     if (llvm::is_contained(L->getBlocks(), A))
  1407:       return false;
  1408:     MachineInstr *Def = MRI->getVRegDef(MO->getReg());
  1409:     LoopFeederPhi.insert(std::make_pair(MO->getReg(), Def));
  1410:     return true;
  1411:   } else
  1412:     // Already visited node.
  1413:     return false;
  1414: }
  1415: 
  1416: /// Return true if a Phi may generate a value that can underflow.
  1417: /// This function calls loopCountMayWrapOrUnderFlow for each Phi operand.
  1418: bool HexagonHardwareLoops::phiMayWrapOrUnderflow(
  1419:     MachineInstr *Phi, const MachineOperand *EndVal, MachineBasicBlock *MBB,
  1420:     MachineLoop *L, LoopFeederMap &LoopFeederPhi) const {
  1421:   assert(Phi->isPHI() && "Expecting a Phi.");
  1422:   // Walk through each Phi, and its used operands. Make sure that
  1423:   // if there is recursion in Phi, we won't generate hardware loops.
  1424:   for (int i = 1, n = Phi->getNumOperands(); i < n; i += 2)
  1425:     if (isLoopFeeder(L, MBB, Phi, &(Phi->getOperand(i)), LoopFeederPhi))
  1426:       if (loopCountMayWrapOrUnderFlow(&(Phi->getOperand(i)), EndVal,
  1427:                                       Phi->getParent(), L, LoopFeederPhi))
  1428:         return true;
  1429:   return false;
  1430: }
  1431: 
  1432: /// Return true if the induction variable can underflow in the first iteration.
  1433: /// An example, is an initial unsigned value that is 0 and is decrement in the
  1434: /// first itertion of a do-while loop.  In this case, we cannot generate a
  1435: /// hardware loop because the endloop instruction does not decrement the loop
  1436: /// counter if it is <= 1. We only need to perform this analysis if the
  1437: /// initial value is a register.
  1438: ///
  1439: /// This function assumes the initial value may underflow unless proven
  1440: /// otherwise. If the type is signed, then we don't care because signed
  1441: /// underflow is undefined. We attempt to prove the initial value is not
  1442: /// zero by performing a crude analysis of the loop counter. This function
  1443: /// checks if the initial value is used in any comparison prior to the loop
  1444: /// and, if so, assumes the comparison is a range check. This is inexact,
  1445: /// but will catch the simple cases.
  1446: bool HexagonHardwareLoops::loopCountMayWrapOrUnderFlow(
  1447:     const MachineOperand *InitVal, const MachineOperand *EndVal,
  1448:     MachineBasicBlock *MBB, MachineLoop *L,
  1449:     LoopFeederMap &LoopFeederPhi) const {
  1450:   // Only check register values since they are unknown.
  1451:   if (!InitVal->isReg())
  1452:     return false;
  1453: 
  1454:   if (!EndVal->isImm())
  1455:     return false;
  1456: 
  1457:   // A register value that is assigned an immediate is a known value, and it
  1458:   // won't underflow in the first iteration.
  1459:   int64_t Imm;
  1460:   if (checkForImmediate(*InitVal, Imm))
  1461:     return (EndVal->getImm() == Imm);
  1462: 
  1463:   Register Reg = InitVal->getReg();
  1464: 
  1465:   // We don't know the value of a physical register.
  1466:   if (!Reg.isVirtual())
  1467:     return true;
  1468: 
  1469:   MachineInstr *Def = MRI->getVRegDef(Reg);
  1470:   if (!Def)
  1471:     return true;
  1472: 
  1473:   // If the initial value is a Phi or copy and the operands may not underflow,
  1474:   // then the definition cannot be underflow either.
  1475:   if (Def->isPHI() && !phiMayWrapOrUnderflow(Def, EndVal, Def->getParent(),
  1476:                                              L, LoopFeederPhi))
  1477:     return false;
  1478:   if (Def->isCopy() && !loopCountMayWrapOrUnderFlow(&(Def->getOperand(1)),
  1479:                                                     EndVal, Def->getParent(),
  1480:                                                     L, LoopFeederPhi))
  1481:     return false;
  1482: 
  1483:   // Iterate over the uses of the initial value. If the initial value is used
  1484:   // in a compare, then we assume this is a range check that ensures the loop
  1485:   // doesn't underflow. This is not an exact test and should be improved.
  1486:   for (MachineRegisterInfo::use_instr_nodbg_iterator I = MRI->use_instr_nodbg_begin(Reg),
  1487:          E = MRI->use_instr_nodbg_end(); I != E; ++I) {
  1488:     MachineInstr *MI = &*I;
  1489:     Register CmpReg1, CmpReg2;
  1490:     int64_t CmpMask = 0, CmpValue = 0;
  1491: 
  1492:     if (!TII->analyzeCompare(*MI, CmpReg1, CmpReg2, CmpMask, CmpValue))
  1493:       continue;
  1494: 
  1495:     MachineBasicBlock *TBB = nullptr, *FBB = nullptr;
  1496:     SmallVector<MachineOperand, 2> Cond;
  1497:     if (TII->analyzeBranch(*MI->getParent(), TBB, FBB, Cond, false))
  1498:       continue;
  1499: 
  1500:     Comparison::Kind Cmp =
  1501:         getComparisonKind(MI->getOpcode(), nullptr, nullptr, 0);
  1502:     if (Cmp == 0)
  1503:       continue;
  1504:     if (TII->predOpcodeHasNot(Cond) ^ (TBB != MBB))
  1505:       Cmp = Comparison::getNegatedComparison(Cmp);
  1506:     if (CmpReg2 != 0 && CmpReg2 == Reg)
  1507:       Cmp = Comparison::getSwappedComparison(Cmp);
  1508: 
  1509:     // Signed underflow is undefined.
  1510:     if (Comparison::isSigned(Cmp))
  1511:       return false;
  1512: 
  1513:     // Check if there is a comparison of the initial value. If the initial value
  1514:     // is greater than or not equal to another value, then assume this is a
  1515:     // range check.
  1516:     if ((Cmp & Comparison::G) || Cmp == Comparison::NE)
  1517:       return false;
  1518:   }
  1519: 
  1520:   // OK - this is a hack that needs to be improved. We really need to analyze
  1521:   // the instructions performed on the initial value. This works on the simplest
  1522:   // cases only.
  1523:   if (!Def->isCopy() && !Def->isPHI())
  1524:     return false;
  1525: 
  1526:   return true;
  1527: }
  1528: 
  1529: bool HexagonHardwareLoops::checkForImmediate(const MachineOperand &MO,
  1530:                                              int64_t &Val) const {
  1531:   if (MO.isImm()) {
  1532:     Val = MO.getImm();
  1533:     return true;
  1534:   }
  1535:   if (!MO.isReg())
  1536:     return false;
  1537: 
  1538:   // MO is a register. Check whether it is defined as an immediate value,
  1539:   // and if so, get the value of it in TV. That value will then need to be
  1540:   // processed to handle potential subregisters in MO.
  1541:   int64_t TV;
  1542: 
  1543:   Register R = MO.getReg();
  1544:   if (!R.isVirtual())
  1545:     return false;
  1546:   MachineInstr *DI = MRI->getVRegDef(R);
  1547:   unsigned DOpc = DI->getOpcode();
  1548:   switch (DOpc) {
  1549:     case TargetOpcode::COPY:
  1550:     case Hexagon::A2_tfrsi:
  1551:     case Hexagon::A2_tfrpi:
  1552:     case Hexagon::CONST32:
  1553:     case Hexagon::CONST64:
  1554:       // Call recursively to avoid an extra check whether operand(1) is
  1555:       // indeed an immediate (it could be a global address, for example),
  1556:       // plus we can handle COPY at the same time.
  1557:       if (!checkForImmediate(DI->getOperand(1), TV))
  1558:         return false;
  1559:       break;
  1560:     case Hexagon::A2_combineii:
  1561:     case Hexagon::A4_combineir:
  1562:     case Hexagon::A4_combineii:
  1563:     case Hexagon::A4_combineri:
  1564:     case Hexagon::A2_combinew: {
  1565:       const MachineOperand &S1 = DI->getOperand(1);
  1566:       const MachineOperand &S2 = DI->getOperand(2);
  1567:       int64_t V1, V2;
  1568:       if (!checkForImmediate(S1, V1) || !checkForImmediate(S2, V2))
  1569:         return false;
  1570:       TV = V2 | (static_cast<uint64_t>(V1) << 32);
  1571:       break;
  1572:     }
  1573:     case TargetOpcode::REG_SEQUENCE: {
  1574:       const MachineOperand &S1 = DI->getOperand(1);
  1575:       const MachineOperand &S3 = DI->getOperand(3);
  1576:       int64_t V1, V3;
  1577:       if (!checkForImmediate(S1, V1) || !checkForImmediate(S3, V3))
  1578:         return false;
  1579:       unsigned Sub2 = DI->getOperand(2).getImm();
  1580:       unsigned Sub4 = DI->getOperand(4).getImm();
  1581:       if (Sub2 == Hexagon::isub_lo && Sub4 == Hexagon::isub_hi)
  1582:         TV = V1 | (V3 << 32);
  1583:       else if (Sub2 == Hexagon::isub_hi && Sub4 == Hexagon::isub_lo)
  1584:         TV = V3 | (V1 << 32);
  1585:       else
  1586:         llvm_unreachable("Unexpected form of REG_SEQUENCE");
  1587:       break;
  1588:     }
  1589: 
  1590:     default:
  1591:       return false;
  1592:   }
  1593: 
  1594:   // By now, we should have successfully obtained the immediate value defining
  1595:   // the register referenced in MO. Handle a potential use of a subregister.
  1596:   switch (MO.getSubReg()) {
  1597:     case Hexagon::isub_lo:
  1598:       Val = TV & 0xFFFFFFFFULL;
  1599:       break;
  1600:     case Hexagon::isub_hi:
```
- EN: It declares or implements routines such as LLVM_DEBUG, getVRegDef, insert, HexagonHardwareLoops::phiMayWrapOrUnderflow, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonHardwareLoops, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 LLVM_DEBUG, getVRegDef, insert, HexagonHardwareLoops::phiMayWrapOrUnderflow, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonHardwareLoops，说明了它与同级后端组件的连接关系。

### Lines 1601-1800 / 第 1601-1800 行

```cpp
  1601:       Val = (TV >> 32) & 0xFFFFFFFFULL;
  1602:       break;
  1603:     default:
  1604:       Val = TV;
  1605:       break;
  1606:   }
  1607:   return true;
  1608: }
  1609: 
  1610: void HexagonHardwareLoops::setImmediate(MachineOperand &MO, int64_t Val) {
  1611:   if (MO.isImm()) {
  1612:     MO.setImm(Val);
  1613:     return;
  1614:   }
  1615: 
  1616:   assert(MO.isReg());
  1617:   Register R = MO.getReg();
  1618:   MachineInstr *DI = MRI->getVRegDef(R);
  1619: 
  1620:   const TargetRegisterClass *RC = MRI->getRegClass(R);
  1621:   Register NewR = MRI->createVirtualRegister(RC);
  1622:   MachineBasicBlock &B = *DI->getParent();
  1623:   DebugLoc DL = DI->getDebugLoc();
  1624:   BuildMI(B, DI, DL, TII->get(DI->getOpcode()), NewR).addImm(Val);
  1625:   MO.setReg(NewR);
  1626: }
  1627: 
  1628: bool HexagonHardwareLoops::fixupInductionVariable(MachineLoop *L) {
  1629:   MachineBasicBlock *Header = L->getHeader();
  1630:   MachineBasicBlock *Latch = L->getLoopLatch();
  1631:   MachineBasicBlock *ExitingBlock = L->findLoopControlBlock();
  1632: 
  1633:   if (!(Header && Latch && ExitingBlock))
  1634:     return false;
  1635: 
  1636:   // These data structures follow the same concept as the corresponding
  1637:   // ones in findInductionRegister (where some comments are).
  1638:   using RegisterBump = std::pair<Register, int64_t>;
  1639:   using RegisterInduction = std::pair<Register, RegisterBump>;
  1640:   using RegisterInductionSet = std::set<RegisterInduction>;
  1641: 
  1642:   // Register candidates for induction variables, with their associated bumps.
  1643:   RegisterInductionSet IndRegs;
  1644: 
  1645:   // Look for induction patterns:
  1646:   //   %1 = PHI ..., [ latch, %2 ]
  1647:   //   %2 = ADD %1, imm
  1648:   using instr_iterator = MachineBasicBlock::instr_iterator;
  1649: 
  1650:   for (instr_iterator I = Header->instr_begin(), E = Header->instr_end();
  1651:        I != E && I->isPHI(); ++I) {
  1652:     MachineInstr *Phi = &*I;
  1653: 
  1654:     // Have a PHI instruction.
  1655:     for (unsigned i = 1, n = Phi->getNumOperands(); i < n; i += 2) {
  1656:       if (Phi->getOperand(i+1).getMBB() != Latch)
  1657:         continue;
  1658: 
  1659:       Register PhiReg = Phi->getOperand(i).getReg();
  1660:       MachineInstr *DI = MRI->getVRegDef(PhiReg);
  1661: 
  1662:       if (DI->getDesc().isAdd()) {
  1663:         // If the register operand to the add/sub is the PHI we are looking
  1664:         // at, this meets the induction pattern.
  1665:         Register IndReg = DI->getOperand(1).getReg();
  1666:         MachineOperand &Opnd2 = DI->getOperand(2);
  1667:         int64_t V;
  1668:         if (MRI->getVRegDef(IndReg) == Phi && checkForImmediate(Opnd2, V)) {
  1669:           Register UpdReg = DI->getOperand(0).getReg();
  1670:           IndRegs.insert(std::make_pair(UpdReg, std::make_pair(IndReg, V)));
  1671:         }
  1672:       }
  1673:     }  // for (i)
  1674:   }  // for (instr)
  1675: 
  1676:   if (IndRegs.empty())
  1677:     return false;
  1678: 
  1679:   MachineBasicBlock *TB = nullptr, *FB = nullptr;
  1680:   SmallVector<MachineOperand,2> Cond;
  1681:   // analyzeBranch returns true if it fails to analyze branch.
  1682:   bool NotAnalyzed = TII->analyzeBranch(*ExitingBlock, TB, FB, Cond, false);
  1683:   if (NotAnalyzed || Cond.empty())
  1684:     return false;
  1685: 
  1686:   if (ExitingBlock != Latch && (TB == Latch || FB == Latch)) {
  1687:     MachineBasicBlock *LTB = nullptr, *LFB = nullptr;
  1688:     SmallVector<MachineOperand,2> LCond;
  1689:     bool NotAnalyzed = TII->analyzeBranch(*Latch, LTB, LFB, LCond, false);
  1690:     if (NotAnalyzed)
  1691:       return false;
  1692: 
  1693:     // Since latch is not the exiting block, the latch branch should be an
  1694:     // unconditional branch to the loop header.
  1695:     if (TB == Latch)
  1696:       TB = (LTB == Header) ? LTB : LFB;
  1697:     else
  1698:       FB = (LTB == Header) ? LTB : LFB;
  1699:   }
  1700:   if (TB != Header) {
  1701:     if (FB != Header) {
  1702:       // The latch/exit block does not go back to the header.
  1703:       return false;
  1704:     }
  1705:     // FB is the header (i.e., uncond. jump to branch header)
  1706:     // In this case, the LoopBody -> TB should not be a back edge otherwise
  1707:     // it could result in an infinite loop after conversion to hw_loop.
  1708:     // This case can happen when the Latch has two jumps like this:
  1709:     // Jmp_c OuterLoopHeader <-- TB
  1710:     // Jmp   InnerLoopHeader <-- FB
  1711:     if (MDT->dominates(TB, FB))
  1712:       return false;
  1713:   }
  1714: 
  1715:   // Expecting a predicate register as a condition.  It won't be a hardware
  1716:   // predicate register at this point yet, just a vreg.
  1717:   // HexagonInstrInfo::analyzeBranch for negated branches inserts imm(0)
  1718:   // into Cond, followed by the predicate register.  For non-negated branches
  1719:   // it's just the register.
  1720:   unsigned CSz = Cond.size();
  1721:   if (CSz != 1 && CSz != 2)
  1722:     return false;
  1723: 
  1724:   if (!Cond[CSz-1].isReg())
  1725:     return false;
  1726: 
  1727:   Register P = Cond[CSz - 1].getReg();
  1728:   MachineInstr *PredDef = MRI->getVRegDef(P);
  1729: 
  1730:   if (!PredDef->isCompare())
  1731:     return false;
  1732: 
  1733:   SmallSet<Register,2> CmpRegs;
  1734:   MachineOperand *CmpImmOp = nullptr;
  1735: 
  1736:   // Go over all operands to the compare and look for immediate and register
  1737:   // operands.  Assume that if the compare has a single register use and a
  1738:   // single immediate operand, then the register is being compared with the
  1739:   // immediate value.
  1740:   for (MachineOperand &MO : PredDef->operands()) {
  1741:     if (MO.isReg()) {
  1742:       // Skip all implicit references.  In one case there was:
  1743:       //   %140 = FCMPUGT32_rr %138, %139, implicit %usr
  1744:       if (MO.isImplicit())
  1745:         continue;
  1746:       if (MO.isUse()) {
  1747:         if (!isImmediate(MO)) {
  1748:           CmpRegs.insert(MO.getReg());
  1749:           continue;
  1750:         }
  1751:         // Consider the register to be the "immediate" operand.
  1752:         if (CmpImmOp)
  1753:           return false;
  1754:         CmpImmOp = &MO;
  1755:       }
  1756:     } else if (MO.isImm()) {
  1757:       if (CmpImmOp)    // A second immediate argument?  Confusing.  Bail out.
  1758:         return false;
  1759:       CmpImmOp = &MO;
  1760:     }
  1761:   }
  1762: 
  1763:   if (CmpRegs.empty())
  1764:     return false;
  1765: 
  1766:   // Check if the compared register follows the order we want.  Fix if needed.
  1767:   for (RegisterInductionSet::iterator I = IndRegs.begin(), E = IndRegs.end();
  1768:        I != E; ++I) {
  1769:     // This is a success.  If the register used in the comparison is one that
  1770:     // we have identified as a bumped (updated) induction register, there is
  1771:     // nothing to do.
  1772:     if (CmpRegs.count(I->first))
  1773:       return true;
  1774: 
  1775:     // Otherwise, if the register being compared comes out of a PHI node,
  1776:     // and has been recognized as following the induction pattern, and is
  1777:     // compared against an immediate, we can fix it.
  1778:     const RegisterBump &RB = I->second;
  1779:     if (CmpRegs.count(RB.first)) {
  1780:       if (!CmpImmOp) {
  1781:         // If both operands to the compare instruction are registers, see if
  1782:         // it can be changed to use induction register as one of the operands.
  1783:         MachineInstr *IndI = nullptr;
  1784:         MachineInstr *nonIndI = nullptr;
  1785:         MachineOperand *IndMO = nullptr;
  1786:         MachineOperand *nonIndMO = nullptr;
  1787: 
  1788:         for (unsigned i = 1, n = PredDef->getNumOperands(); i < n; ++i) {
  1789:           MachineOperand &MO = PredDef->getOperand(i);
  1790:           if (MO.isReg() && MO.getReg() == RB.first) {
  1791:             LLVM_DEBUG(dbgs() << "\n DefMI(" << i
  1792:                               << ") = " << *(MRI->getVRegDef(I->first)));
  1793:             if (IndI)
  1794:               return false;
  1795: 
  1796:             IndI = MRI->getVRegDef(I->first);
  1797:             IndMO = &MO;
  1798:           } else if (MO.isReg()) {
  1799:             LLVM_DEBUG(dbgs() << "\n DefMI(" << i
  1800:                               << ") = " << *(MRI->getVRegDef(MO.getReg())));
```
- EN: It declares or implements routines such as HexagonHardwareLoops::setImmediate, setImm, assert, getReg, ... (21 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonHardwareLoops, HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonHardwareLoops::setImmediate, setImm, assert, getReg, ... (21 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonHardwareLoops, HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 1801-2000 / 第 1801-2000 行

```cpp
  1801:             if (nonIndI)
  1802:               return false;
  1803: 
  1804:             nonIndI = MRI->getVRegDef(MO.getReg());
  1805:             nonIndMO = &MO;
  1806:           }
  1807:         }
  1808:         if (IndI && nonIndI &&
  1809:             nonIndI->getOpcode() == Hexagon::A2_addi &&
  1810:             nonIndI->getOperand(2).isImm() &&
  1811:             nonIndI->getOperand(2).getImm() == - RB.second) {
  1812:           bool Order = orderBumpCompare(IndI, PredDef);
  1813:           if (Order) {
  1814:             IndMO->setReg(I->first);
  1815:             nonIndMO->setReg(nonIndI->getOperand(1).getReg());
  1816:             return true;
  1817:           }
  1818:         }
  1819:         return false;
  1820:       }
  1821: 
  1822:       // It is not valid to do this transformation on an unsigned comparison
  1823:       // because it may underflow.
  1824:       Comparison::Kind Cmp =
  1825:           getComparisonKind(PredDef->getOpcode(), nullptr, nullptr, 0);
  1826:       if (!Cmp || Comparison::isUnsigned(Cmp))
  1827:         return false;
  1828: 
  1829:       // If the register is being compared against an immediate, try changing
  1830:       // the compare instruction to use induction register and adjust the
  1831:       // immediate operand.
  1832:       int64_t CmpImm = getImmediate(*CmpImmOp);
  1833:       int64_t V = RB.second;
  1834:       // Handle Overflow (64-bit).
  1835:       if (((V > 0) && (CmpImm > INT64_MAX - V)) ||
  1836:           ((V < 0) && (CmpImm < INT64_MIN - V)))
  1837:         return false;
  1838:       CmpImm += V;
  1839:       // Most comparisons of register against an immediate value allow
  1840:       // the immediate to be constant-extended. There are some exceptions
  1841:       // though. Make sure the new combination will work.
  1842:       if (CmpImmOp->isImm() && !TII->isExtendable(*PredDef) &&
  1843:           !TII->isValidOffset(PredDef->getOpcode(), CmpImm, TRI, false))
  1844:         return false;
  1845: 
  1846:       // Make sure that the compare happens after the bump.  Otherwise,
  1847:       // after the fixup, the compare would use a yet-undefined register.
  1848:       MachineInstr *BumpI = MRI->getVRegDef(I->first);
  1849:       bool Order = orderBumpCompare(BumpI, PredDef);
  1850:       if (!Order)
  1851:         return false;
  1852: 
  1853:       // Finally, fix the compare instruction.
  1854:       setImmediate(*CmpImmOp, CmpImm);
  1855:       for (MachineOperand &MO : PredDef->operands()) {
  1856:         if (MO.isReg() && MO.getReg() == RB.first) {
  1857:           MO.setReg(I->first);
  1858:           return true;
  1859:         }
  1860:       }
  1861:     }
  1862:   }
  1863: 
  1864:   return false;
  1865: }
  1866: 
  1867: /// createPreheaderForLoop - Create a preheader for a given loop.
  1868: MachineBasicBlock *HexagonHardwareLoops::createPreheaderForLoop(
  1869:       MachineLoop *L) {
  1870:   if (MachineBasicBlock *TmpPH = MLI->findLoopPreheader(L, SpecPreheader))
  1871:     return TmpPH;
  1872:   if (!HWCreatePreheader)
  1873:     return nullptr;
  1874: 
  1875:   MachineBasicBlock *Header = L->getHeader();
  1876:   MachineBasicBlock *Latch = L->getLoopLatch();
  1877:   MachineBasicBlock *ExitingBlock = L->findLoopControlBlock();
  1878:   MachineFunction *MF = Header->getParent();
  1879:   DebugLoc DL;
  1880: 
  1881: #ifndef NDEBUG
  1882:   if ((!PHFn.empty()) && (PHFn != MF->getName()))
  1883:     return nullptr;
  1884: #endif
  1885: 
  1886:   if (!Latch || !ExitingBlock || Header->hasAddressTaken())
  1887:     return nullptr;
  1888: 
  1889:   using instr_iterator = MachineBasicBlock::instr_iterator;
  1890: 
  1891:   // Verify that all existing predecessors have analyzable branches
  1892:   // (or no branches at all).
  1893:   using MBBVector = std::vector<MachineBasicBlock *>;
  1894: 
  1895:   MBBVector Preds(Header->pred_begin(), Header->pred_end());
  1896:   SmallVector<MachineOperand,2> Tmp1;
  1897:   MachineBasicBlock *TB = nullptr, *FB = nullptr;
  1898: 
  1899:   if (TII->analyzeBranch(*ExitingBlock, TB, FB, Tmp1, false))
  1900:     return nullptr;
  1901: 
  1902:   for (MachineBasicBlock *PB : Preds) {
  1903:     bool NotAnalyzed = TII->analyzeBranch(*PB, TB, FB, Tmp1, false);
  1904:     if (NotAnalyzed)
  1905:       return nullptr;
  1906:   }
  1907: 
  1908:   MachineBasicBlock *NewPH = MF->CreateMachineBasicBlock();
  1909:   MF->insert(Header->getIterator(), NewPH);
  1910: 
  1911:   if (Header->pred_size() > 2) {
  1912:     // Ensure that the header has only two predecessors: the preheader and
  1913:     // the loop latch.  Any additional predecessors of the header should
  1914:     // join at the newly created preheader. Inspect all PHI nodes from the
  1915:     // header and create appropriate corresponding PHI nodes in the preheader.
  1916: 
  1917:     for (instr_iterator I = Header->instr_begin(), E = Header->instr_end();
  1918:          I != E && I->isPHI(); ++I) {
  1919:       MachineInstr *PN = &*I;
  1920: 
  1921:       const MCInstrDesc &PD = TII->get(TargetOpcode::PHI);
  1922:       MachineInstr *NewPN = MF->CreateMachineInstr(PD, DL);
  1923:       NewPH->insert(NewPH->end(), NewPN);
  1924: 
  1925:       Register PR = PN->getOperand(0).getReg();
  1926:       const TargetRegisterClass *RC = MRI->getRegClass(PR);
  1927:       Register NewPR = MRI->createVirtualRegister(RC);
  1928:       NewPN->addOperand(MachineOperand::CreateReg(NewPR, true));
  1929: 
  1930:       // Copy all non-latch operands of a header's PHI node to the newly
  1931:       // created PHI node in the preheader.
  1932:       for (unsigned i = 1, n = PN->getNumOperands(); i < n; i += 2) {
  1933:         Register PredR = PN->getOperand(i).getReg();
  1934:         unsigned PredRSub = PN->getOperand(i).getSubReg();
  1935:         MachineBasicBlock *PredB = PN->getOperand(i+1).getMBB();
  1936:         if (PredB == Latch)
  1937:           continue;
  1938: 
  1939:         MachineOperand MO = MachineOperand::CreateReg(PredR, false);
  1940:         MO.setSubReg(PredRSub);
  1941:         NewPN->addOperand(MO);
  1942:         NewPN->addOperand(MachineOperand::CreateMBB(PredB));
  1943:       }
  1944: 
  1945:       // Remove copied operands from the old PHI node and add the value
  1946:       // coming from the preheader's PHI.
  1947:       for (int i = PN->getNumOperands()-2; i > 0; i -= 2) {
  1948:         MachineBasicBlock *PredB = PN->getOperand(i+1).getMBB();
  1949:         if (PredB != Latch) {
  1950:           PN->removeOperand(i+1);
  1951:           PN->removeOperand(i);
  1952:         }
  1953:       }
  1954:       PN->addOperand(MachineOperand::CreateReg(NewPR, false));
  1955:       PN->addOperand(MachineOperand::CreateMBB(NewPH));
  1956:     }
  1957:   } else {
  1958:     assert(Header->pred_size() == 2);
  1959: 
  1960:     // The header has only two predecessors, but the non-latch predecessor
  1961:     // is not a preheader (e.g. it has other successors, etc.)
  1962:     // In such a case we don't need any extra PHI nodes in the new preheader,
  1963:     // all we need is to adjust existing PHIs in the header to now refer to
  1964:     // the new preheader.
  1965:     for (instr_iterator I = Header->instr_begin(), E = Header->instr_end();
  1966:          I != E && I->isPHI(); ++I) {
  1967:       MachineInstr *PN = &*I;
  1968:       for (unsigned i = 1, n = PN->getNumOperands(); i < n; i += 2) {
  1969:         MachineOperand &MO = PN->getOperand(i+1);
  1970:         if (MO.getMBB() != Latch)
  1971:           MO.setMBB(NewPH);
  1972:       }
  1973:     }
  1974:   }
  1975: 
  1976:   // "Reroute" the CFG edges to link in the new preheader.
  1977:   // If any of the predecessors falls through to the header, insert a branch
  1978:   // to the new preheader in that place.
  1979:   SmallVector<MachineOperand,1> Tmp2;
  1980:   SmallVector<MachineOperand,1> EmptyCond;
  1981: 
  1982:   TB = FB = nullptr;
  1983: 
  1984:   for (MachineBasicBlock *PB : Preds) {
  1985:     if (PB != Latch) {
  1986:       Tmp2.clear();
  1987:       bool NotAnalyzed = TII->analyzeBranch(*PB, TB, FB, Tmp2, false);
  1988:       (void)NotAnalyzed; // suppress compiler warning
  1989:       assert (!NotAnalyzed && "Should be analyzable!");
  1990:       if (TB != Header && (Tmp2.empty() || FB != Header))
  1991:         TII->insertBranch(*PB, NewPH, nullptr, EmptyCond, DL);
  1992:       PB->ReplaceUsesOfBlockWith(Header, NewPH);
  1993:     }
  1994:   }
  1995: 
  1996:   // It can happen that the latch block will fall through into the header.
  1997:   // Insert an unconditional branch to the header.
  1998:   TB = FB = nullptr;
  1999:   bool LatchNotAnalyzed = TII->analyzeBranch(*Latch, TB, FB, Tmp2, false);
  2000:   (void)LatchNotAnalyzed; // suppress compiler warning
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as getVRegDef, orderBumpCompare, setReg, getComparisonKind, ... (29 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 getVRegDef, orderBumpCompare, setReg, getComparisonKind, ... (29 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 2001-2024 / 第 2001-2024 行

```cpp
  2001:   assert (!LatchNotAnalyzed && "Should be analyzable!");
  2002:   if (!TB && !FB)
  2003:     TII->insertBranch(*Latch, Header, nullptr, EmptyCond, DL);
  2004: 
  2005:   // Finally, the branch from the preheader to the header.
  2006:   TII->insertBranch(*NewPH, Header, nullptr, EmptyCond, DL);
  2007:   NewPH->addSuccessor(Header);
  2008: 
  2009:   MachineLoop *ParentLoop = L->getParentLoop();
  2010:   if (ParentLoop)
  2011:     ParentLoop->addBasicBlockToLoop(NewPH, *MLI);
  2012: 
  2013:   // Update the dominator information with the new preheader.
  2014:   if (MDT) {
  2015:     if (MachineDomTreeNode *HN = MDT->getNode(Header)) {
  2016:       if (MachineDomTreeNode *DHN = HN->getIDom()) {
  2017:         MDT->addNewBlock(NewPH, DHN->getBlock());
  2018:         MDT->changeImmediateDominator(Header, NewPH);
  2019:       }
  2020:     }
  2021:   }
  2022: 
  2023:   return NewPH;
  2024: }
```
- EN: It declares or implements routines such as assert, insertBranch, addSuccessor, getParentLoop, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 assert, insertBranch, addSuccessor, getParentLoop, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

## Key Concepts / 关键概念

- loop-aware code generation / 循环相关代码生成
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/ArrayRef.h, llvm/ADT/STLExtras.h, llvm/ADT/SmallSet.h, llvm/ADT/SmallVector.h, llvm/ADT/Statistic.h, llvm/ADT/StringRef.h, llvm/CodeGen/MachineBasicBlock.h, ... (36 total)`
- Hexagon symbols / Hexagon 符号: `HexagonHardwareLoops, HexagonInstrInfo, HexagonSubtarget, HexagonRegisterInfo`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
