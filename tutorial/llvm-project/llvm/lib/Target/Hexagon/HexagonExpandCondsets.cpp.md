# HexagonExpandCondsets.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonExpandCondsets.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon expansion of conditional-set pseudo operations.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100 / 第 1-100 行

```cpp
     1: //===- HexagonExpandCondsets.cpp ------------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: // Replace mux instructions with the corresponding legal instructions.
    10: // It is meant to work post-SSA, but still on virtual registers. It was
    11: // originally placed between register coalescing and machine instruction
    12: // scheduler.
    13: // In this place in the optimization sequence, live interval analysis had
    14: // been performed, and the live intervals should be preserved. A large part
    15: // of the code deals with preserving the liveness information.
    16: //
    17: // Liveness tracking aside, the main functionality of this pass is divided
    18: // into two steps. The first step is to replace an instruction
    19: //   %0 = C2_mux %1, %2, %3
    20: // with a pair of conditional transfers
    21: //   %0 = A2_tfrt %1, %2
    22: //   %0 = A2_tfrf %1, %3
    23: // It is the intention that the execution of this pass could be terminated
    24: // after this step, and the code generated would be functionally correct.
    25: //
    26: // If the uses of the source values %1 and %2 are kills, and their
    27: // definitions are predicable, then in the second step, the conditional
    28: // transfers will then be rewritten as predicated instructions. E.g.
    29: //   %0 = A2_or %1, %2
    30: //   %3 = A2_tfrt %99, killed %0
    31: // will be rewritten as
    32: //   %3 = A2_port %99, %1, %2
    33: //
    34: // This replacement has two variants: "up" and "down". Consider this case:
    35: //   %0 = A2_or %1, %2
    36: //   ... [intervening instructions] ...
    37: //   %3 = A2_tfrt %99, killed %0
    38: // variant "up":
    39: //   %3 = A2_port %99, %1, %2
    40: //   ... [intervening instructions, %0->vreg3] ...
    41: //   [deleted]
    42: // variant "down":
    43: //   [deleted]
    44: //   ... [intervening instructions] ...
    45: //   %3 = A2_port %99, %1, %2
    46: //
    47: // Both, one or none of these variants may be valid, and checks are made
    48: // to rule out inapplicable variants.
    49: //
    50: // As an additional optimization, before either of the two steps above is
    51: // executed, the pass attempts to coalesce the target register with one of
    52: // the source registers, e.g. given an instruction
    53: //   %3 = C2_mux %0, %1, %2
    54: // %3 will be coalesced with either %1 or %2. If this succeeds,
    55: // the instruction would then be (for example)
    56: //   %3 = C2_mux %0, %3, %2
    57: // and, under certain circumstances, this could result in only one predicated
    58: // instruction:
    59: //   %3 = A2_tfrf %0, %2
    60: //
    61: 
    62: // Splitting a definition of a register into two predicated transfers
    63: // creates a complication in liveness tracking. Live interval computation
    64: // will see both instructions as actual definitions, and will mark the
    65: // first one as dead. The definition is not actually dead, and this
    66: // situation will need to be fixed. For example:
    67: //   dead %1 = A2_tfrt ...  ; marked as dead
    68: //   %1 = A2_tfrf ...
    69: //
    70: // Since any of the individual predicated transfers may end up getting
    71: // removed (in case it is an identity copy), some pre-existing def may
    72: // be marked as dead after live interval recomputation:
    73: //   dead %1 = ...          ; marked as dead
    74: //   ...
    75: //   %1 = A2_tfrf ...       ; if A2_tfrt is removed
    76: // This case happens if %1 was used as a source in A2_tfrt, which means
    77: // that is it actually live at the A2_tfrf, and so the now dead definition
    78: // of %1 will need to be updated to non-dead at some point.
    79: //
    80: // This issue could be remedied by adding implicit uses to the predicated
    81: // transfers, but this will create a problem with subsequent predication,
    82: // since the transfers will no longer be possible to reorder. To avoid
    83: // that, the initial splitting will not add any implicit uses. These
    84: // implicit uses will be added later, after predication. The extra price,
    85: // however, is that finding the locations where the implicit uses need
    86: // to be added, and updating the live ranges will be more involved.
    87: 
    88: #include "Hexagon.h"
    89: #include "HexagonInstrInfo.h"
    90: #include "HexagonRegisterInfo.h"
    91: #include "llvm/ADT/DenseMap.h"
    92: #include "llvm/ADT/SetVector.h"
    93: #include "llvm/ADT/SmallVector.h"
    94: #include "llvm/ADT/StringRef.h"
    95: #include "llvm/CodeGen/LiveInterval.h"
    96: #include "llvm/CodeGen/LiveIntervals.h"
    97: #include "llvm/CodeGen/MachineBasicBlock.h"
    98: #include "llvm/CodeGen/MachineDominators.h"
    99: #include "llvm/CodeGen/MachineFunction.h"
   100: #include "llvm/CodeGen/MachineFunctionPass.h"
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. It imports headers such as Hexagon.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, llvm/ADT/DenseMap.h, ... (13 total), establishing the LLVM/Hexagon APIs used below. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonExpandCondsets, HexagonInstrInfo, HexagonRegisterInfo, showing how the code connects to sibling backend components.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这里引入了 Hexagon.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, llvm/ADT/DenseMap.h, ... (13 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonExpandCondsets, HexagonInstrInfo, HexagonRegisterInfo，说明了它与同级后端组件的连接关系。

### Lines 101-200 / 第 101-200 行

```cpp
   101: #include "llvm/CodeGen/MachineInstr.h"
   102: #include "llvm/CodeGen/MachineInstrBuilder.h"
   103: #include "llvm/CodeGen/MachineOperand.h"
   104: #include "llvm/CodeGen/MachineRegisterInfo.h"
   105: #include "llvm/CodeGen/SlotIndexes.h"
   106: #include "llvm/CodeGen/TargetRegisterInfo.h"
   107: #include "llvm/CodeGen/TargetSubtargetInfo.h"
   108: #include "llvm/IR/DebugLoc.h"
   109: #include "llvm/IR/Function.h"
   110: #include "llvm/InitializePasses.h"
   111: #include "llvm/MC/LaneBitmask.h"
   112: #include "llvm/Pass.h"
   113: #include "llvm/Support/CommandLine.h"
   114: #include "llvm/Support/Debug.h"
   115: #include "llvm/Support/ErrorHandling.h"
   116: #include "llvm/Support/raw_ostream.h"
   117: #include <cassert>
   118: #include <iterator>
   119: #include <map>
   120: #include <set>
   121: #include <utility>
   122: 
   123: #define DEBUG_TYPE "expand-condsets"
   124: 
   125: using namespace llvm;
   126: 
   127: static cl::opt<unsigned> OptTfrLimit("expand-condsets-tfr-limit",
   128:   cl::init(~0U), cl::Hidden, cl::desc("Max number of mux expansions"));
   129: static cl::opt<unsigned> OptCoaLimit("expand-condsets-coa-limit",
   130:   cl::init(~0U), cl::Hidden, cl::desc("Max number of segment coalescings"));
   131: 
   132: namespace {
   133: 
   134:   class HexagonExpandCondsets : public MachineFunctionPass {
   135:   public:
   136:     static char ID;
   137: 
   138:     HexagonExpandCondsets() : MachineFunctionPass(ID) {
   139:       if (OptCoaLimit.getPosition())
   140:         CoaLimitActive = true, CoaLimit = OptCoaLimit;
   141:       if (OptTfrLimit.getPosition())
   142:         TfrLimitActive = true, TfrLimit = OptTfrLimit;
   143:     }
   144: 
   145:     StringRef getPassName() const override { return "Hexagon Expand Condsets"; }
   146: 
   147:     void getAnalysisUsage(AnalysisUsage &AU) const override {
   148:       AU.addRequired<LiveIntervalsWrapperPass>();
   149:       AU.addPreserved<LiveIntervalsWrapperPass>();
   150:       AU.addPreserved<SlotIndexesWrapperPass>();
   151:       AU.addRequired<MachineDominatorTreeWrapperPass>();
   152:       AU.addPreserved<MachineDominatorTreeWrapperPass>();
   153:       MachineFunctionPass::getAnalysisUsage(AU);
   154:     }
   155: 
   156:     bool runOnMachineFunction(MachineFunction &MF) override;
   157: 
   158:   private:
   159:     const HexagonInstrInfo *HII = nullptr;
   160:     const TargetRegisterInfo *TRI = nullptr;
   161:     MachineDominatorTree *MDT;
   162:     MachineRegisterInfo *MRI = nullptr;
   163:     LiveIntervals *LIS = nullptr;
   164:     bool CoaLimitActive = false;
   165:     bool TfrLimitActive = false;
   166:     unsigned CoaLimit;
   167:     unsigned TfrLimit;
   168:     unsigned CoaCounter = 0;
   169:     unsigned TfrCounter = 0;
   170: 
   171:     // FIXME: Consolidate duplicate definitions of RegisterRef
   172:     struct RegisterRef {
   173:       RegisterRef(const MachineOperand &Op) : Reg(Op.getReg()),
   174:           Sub(Op.getSubReg()) {}
   175:       RegisterRef(unsigned R = 0, unsigned S = 0) : Reg(R), Sub(S) {}
   176: 
   177:       bool operator== (RegisterRef RR) const {
   178:         return Reg == RR.Reg && Sub == RR.Sub;
   179:       }
   180:       bool operator!= (RegisterRef RR) const { return !operator==(RR); }
   181:       bool operator< (RegisterRef RR) const {
   182:         return Reg < RR.Reg || (Reg == RR.Reg && Sub < RR.Sub);
   183:       }
   184: 
   185:       Register Reg;
   186:       unsigned Sub;
   187:     };
   188: 
   189:     using ReferenceMap = DenseMap<unsigned, unsigned>;
   190:     enum { Sub_Low = 0x1, Sub_High = 0x2, Sub_None = (Sub_Low | Sub_High) };
   191:     enum { Exec_Then = 0x10, Exec_Else = 0x20 };
   192: 
   193:     unsigned getMaskForSub(unsigned Sub);
   194:     bool isCondset(const MachineInstr &MI);
   195:     LaneBitmask getLaneMask(Register Reg, unsigned Sub);
   196: 
   197:     void addRefToMap(RegisterRef RR, ReferenceMap &Map, unsigned Exec);
   198:     bool isRefInMap(RegisterRef, ReferenceMap &Map, unsigned Exec);
   199: 
   200:     void updateDeadsInRange(Register Reg, LaneBitmask LM, LiveRange &Range);
```
- EN: It imports headers such as llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineOperand.h, llvm/CodeGen/MachineRegisterInfo.h, ... (21 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonExpandCondsets, RegisterRef, which carry the state or API of this component.
- CN: 这里引入了 llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineOperand.h, llvm/CodeGen/MachineRegisterInfo.h, ... (21 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonExpandCondsets, RegisterRef 等类型，用来承载该组件的状态或接口。

### Lines 201-300 / 第 201-300 行

```cpp
   201:     void updateKillFlags(Register Reg);
   202:     void updateDeadFlags(Register Reg);
   203:     void recalculateLiveInterval(Register Reg);
   204:     void removeInstr(MachineInstr &MI);
   205:     void updateLiveness(const std::set<Register> &RegSet, bool Recalc,
   206:                         bool UpdateKills, bool UpdateDeads);
   207:     void distributeLiveIntervals(const std::set<Register> &Regs);
   208: 
   209:     unsigned getCondTfrOpcode(const MachineOperand &SO, bool Cond);
   210:     MachineInstr *genCondTfrFor(MachineOperand &SrcOp,
   211:         MachineBasicBlock::iterator At, unsigned DstR,
   212:         unsigned DstSR, const MachineOperand &PredOp, bool PredSense,
   213:         bool ReadUndef, bool ImpUse);
   214:     bool split(MachineInstr &MI, std::set<Register> &UpdRegs);
   215: 
   216:     bool isPredicable(MachineInstr *MI);
   217:     MachineInstr *getReachingDefForPred(RegisterRef RD,
   218:         MachineBasicBlock::iterator UseIt, unsigned PredR, bool Cond);
   219:     bool canMoveOver(MachineInstr &MI, ReferenceMap &Defs, ReferenceMap &Uses);
   220:     bool canMoveMemTo(MachineInstr &MI, MachineInstr &ToI, bool IsDown);
   221:     void predicateAt(const MachineOperand &DefOp, MachineInstr &MI,
   222:                      MachineBasicBlock::iterator Where,
   223:                      const MachineOperand &PredOp, bool Cond,
   224:                      std::set<Register> &UpdRegs);
   225:     void renameInRange(RegisterRef RO, RegisterRef RN, unsigned PredR,
   226:         bool Cond, MachineBasicBlock::iterator First,
   227:         MachineBasicBlock::iterator Last);
   228:     bool predicate(MachineInstr &TfrI, bool Cond, std::set<Register> &UpdRegs);
   229:     bool predicateInBlock(MachineBasicBlock &B, std::set<Register> &UpdRegs);
   230: 
   231:     bool isIntReg(RegisterRef RR, unsigned &BW);
   232:     bool isIntraBlocks(LiveInterval &LI);
   233:     bool coalesceRegisters(RegisterRef R1, RegisterRef R2);
   234:     bool coalesceSegments(const SmallVectorImpl<MachineInstr *> &Condsets,
   235:                           std::set<Register> &UpdRegs);
   236:   };
   237: 
   238: } // end anonymous namespace
   239: 
   240: char HexagonExpandCondsets::ID = 0;
   241: char &llvm::HexagonExpandCondsetsID = HexagonExpandCondsets::ID;
   242: 
   243: INITIALIZE_PASS_BEGIN(HexagonExpandCondsets, "expand-condsets",
   244:   "Hexagon Expand Condsets", false, false)
   245: INITIALIZE_PASS_DEPENDENCY(MachineDominatorTreeWrapperPass)
   246: INITIALIZE_PASS_DEPENDENCY(SlotIndexesWrapperPass)
   247: INITIALIZE_PASS_DEPENDENCY(LiveIntervalsWrapperPass)
   248: INITIALIZE_PASS_END(HexagonExpandCondsets, "expand-condsets",
   249:   "Hexagon Expand Condsets", false, false)
   250: 
   251: unsigned HexagonExpandCondsets::getMaskForSub(unsigned Sub) {
   252:   switch (Sub) {
   253:     case Hexagon::isub_lo:
   254:     case Hexagon::vsub_lo:
   255:       return Sub_Low;
   256:     case Hexagon::isub_hi:
   257:     case Hexagon::vsub_hi:
   258:       return Sub_High;
   259:     case Hexagon::NoSubRegister:
   260:       return Sub_None;
   261:   }
   262:   llvm_unreachable("Invalid subregister");
   263: }
   264: 
   265: bool HexagonExpandCondsets::isCondset(const MachineInstr &MI) {
   266:   unsigned Opc = MI.getOpcode();
   267:   switch (Opc) {
   268:     case Hexagon::C2_mux:
   269:     case Hexagon::C2_muxii:
   270:     case Hexagon::C2_muxir:
   271:     case Hexagon::C2_muxri:
   272:     case Hexagon::PS_pselect:
   273:         return true;
   274:       break;
   275:   }
   276:   return false;
   277: }
   278: 
   279: LaneBitmask HexagonExpandCondsets::getLaneMask(Register Reg, unsigned Sub) {
   280:   assert(Reg.isVirtual());
   281:   return Sub != 0 ? TRI->getSubRegIndexLaneMask(Sub)
   282:                   : MRI->getMaxLaneMaskForVReg(Reg);
   283: }
   284: 
   285: void HexagonExpandCondsets::addRefToMap(RegisterRef RR, ReferenceMap &Map,
   286:       unsigned Exec) {
   287:   unsigned Mask = getMaskForSub(RR.Sub) | Exec;
   288:   Map[RR.Reg] |= Mask;
   289: }
   290: 
   291: bool HexagonExpandCondsets::isRefInMap(RegisterRef RR, ReferenceMap &Map,
   292:       unsigned Exec) {
   293:   ReferenceMap::iterator F = Map.find(RR.Reg);
   294:   if (F == Map.end())
   295:     return false;
   296:   unsigned Mask = getMaskForSub(RR.Sub) | Exec;
   297:   if (Mask & F->second)
   298:     return true;
   299:   return false;
   300: }
```
- EN: It opens namespaces (char) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as updateKillFlags, updateDeadFlags, recalculateLiveInterval, removeInstr, ... (31 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（char），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 updateKillFlags, updateDeadFlags, recalculateLiveInterval, removeInstr, ... (31 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 301-400 / 第 301-400 行

```cpp
   301: 
   302: void HexagonExpandCondsets::updateKillFlags(Register Reg) {
   303:   auto KillAt = [this,Reg] (SlotIndex K, LaneBitmask LM) -> void {
   304:     // Set the <kill> flag on a use of Reg whose lane mask is contained in LM.
   305:     MachineInstr *MI = LIS->getInstructionFromIndex(K);
   306:     for (unsigned i = 0, e = MI->getNumOperands(); i != e; ++i) {
   307:       MachineOperand &Op = MI->getOperand(i);
   308:       if (!Op.isReg() || !Op.isUse() || Op.getReg() != Reg ||
   309:           MI->isRegTiedToDefOperand(i))
   310:         continue;
   311:       LaneBitmask SLM = getLaneMask(Reg, Op.getSubReg());
   312:       if ((SLM & LM) == SLM) {
   313:         // Only set the kill flag on the first encountered use of Reg in this
   314:         // instruction.
   315:         Op.setIsKill(true);
   316:         break;
   317:       }
   318:     }
   319:   };
   320: 
   321:   LiveInterval &LI = LIS->getInterval(Reg);
   322:   for (auto I = LI.begin(), E = LI.end(); I != E; ++I) {
   323:     if (!I->end.isRegister())
   324:       continue;
   325:     // Do not mark the end of the segment as <kill>, if the next segment
   326:     // starts with a predicated instruction.
   327:     auto NextI = std::next(I);
   328:     if (NextI != E && NextI->start.isRegister()) {
   329:       MachineInstr *DefI = LIS->getInstructionFromIndex(NextI->start);
   330:       if (HII->isPredicated(*DefI))
   331:         continue;
   332:     }
   333:     bool WholeReg = true;
   334:     if (LI.hasSubRanges()) {
   335:       auto EndsAtI = [I] (LiveInterval::SubRange &S) -> bool {
   336:         LiveRange::iterator F = S.find(I->end);
   337:         return F != S.end() && I->end == F->end;
   338:       };
   339:       // Check if all subranges end at I->end. If so, make sure to kill
   340:       // the whole register.
   341:       for (LiveInterval::SubRange &S : LI.subranges()) {
   342:         if (EndsAtI(S))
   343:           KillAt(I->end, S.LaneMask);
   344:         else
   345:           WholeReg = false;
   346:       }
   347:     }
   348:     if (WholeReg)
   349:       KillAt(I->end, MRI->getMaxLaneMaskForVReg(Reg));
   350:   }
   351: }
   352: 
   353: void HexagonExpandCondsets::updateDeadsInRange(Register Reg, LaneBitmask LM,
   354:                                                LiveRange &Range) {
   355:   assert(Reg.isVirtual());
   356:   if (Range.empty())
   357:     return;
   358: 
   359:   // Return two booleans: { def-modifes-reg, def-covers-reg }.
   360:   auto IsRegDef = [this,Reg,LM] (MachineOperand &Op) -> std::pair<bool,bool> {
   361:     if (!Op.isReg() || !Op.isDef())
   362:       return { false, false };
   363:     Register DR = Op.getReg(), DSR = Op.getSubReg();
   364:     if (!DR.isVirtual() || DR != Reg)
   365:       return { false, false };
   366:     LaneBitmask SLM = getLaneMask(DR, DSR);
   367:     LaneBitmask A = SLM & LM;
   368:     return { A.any(), A == SLM };
   369:   };
   370: 
   371:   // The splitting step will create pairs of predicated definitions without
   372:   // any implicit uses (since implicit uses would interfere with predication).
   373:   // This can cause the reaching defs to become dead after live range
   374:   // recomputation, even though they are not really dead.
   375:   // We need to identify predicated defs that need implicit uses, and
   376:   // dead defs that are not really dead, and correct both problems.
   377: 
   378:   auto Dominate = [this] (SetVector<MachineBasicBlock*> &Defs,
   379:                           MachineBasicBlock *Dest) -> bool {
   380:     for (MachineBasicBlock *D : Defs) {
   381:       if (D != Dest && MDT->dominates(D, Dest))
   382:         return true;
   383:     }
   384:     MachineBasicBlock *Entry = &Dest->getParent()->front();
   385:     SetVector<MachineBasicBlock *> Work(llvm::from_range, Dest->predecessors());
   386:     for (unsigned i = 0; i < Work.size(); ++i) {
   387:       MachineBasicBlock *B = Work[i];
   388:       if (Defs.count(B))
   389:         continue;
   390:       if (B == Entry)
   391:         return false;
   392:       Work.insert_range(B->predecessors());
   393:     }
   394:     return true;
   395:   };
   396: 
   397:   // First, try to extend live range within individual basic blocks. This
   398:   // will leave us only with dead defs that do not reach any predicated
   399:   // defs in the same block.
   400:   SetVector<MachineBasicBlock*> Defs;
```
- EN: It declares or implements routines such as HexagonExpandCondsets::updateKillFlags, getInstructionFromIndex, getOperand, getLaneMask, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonExpandCondsets, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonExpandCondsets::updateKillFlags, getInstructionFromIndex, getOperand, getLaneMask, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonExpandCondsets，说明了它与同级后端组件的连接关系。

### Lines 401-500 / 第 401-500 行

```cpp
   401:   SmallVector<SlotIndex,4> PredDefs;
   402:   for (auto &Seg : Range) {
   403:     if (!Seg.start.isRegister())
   404:       continue;
   405:     MachineInstr *DefI = LIS->getInstructionFromIndex(Seg.start);
   406:     Defs.insert(DefI->getParent());
   407:     if (HII->isPredicated(*DefI))
   408:       PredDefs.push_back(Seg.start);
   409:   }
   410: 
   411:   SmallVector<SlotIndex,8> Undefs;
   412:   LiveInterval &LI = LIS->getInterval(Reg);
   413:   LI.computeSubRangeUndefs(Undefs, LM, *MRI, *LIS->getSlotIndexes());
   414: 
   415:   for (auto &SI : PredDefs) {
   416:     MachineBasicBlock *BB = LIS->getMBBFromIndex(SI);
   417:     auto P = Range.extendInBlock(Undefs, LIS->getMBBStartIdx(BB), SI);
   418:     if (P.first != nullptr || P.second)
   419:       SI = SlotIndex();
   420:   }
   421: 
   422:   // Calculate reachability for those predicated defs that were not handled
   423:   // by the in-block extension.
   424:   SmallVector<SlotIndex,4> ExtTo;
   425:   for (auto &SI : PredDefs) {
   426:     if (!SI.isValid())
   427:       continue;
   428:     MachineBasicBlock *BB = LIS->getMBBFromIndex(SI);
   429:     if (BB->pred_empty())
   430:       continue;
   431:     // If the defs from this range reach SI via all predecessors, it is live.
   432:     // It can happen that SI is reached by the defs through some paths, but
   433:     // not all. In the IR coming into this optimization, SI would not be
   434:     // considered live, since the defs would then not jointly dominate SI.
   435:     // That means that SI is an overwriting def, and no implicit use is
   436:     // needed at this point. Do not add SI to the extension points, since
   437:     // extendToIndices will abort if there is no joint dominance.
   438:     // If the abort was avoided by adding extra undefs added to Undefs,
   439:     // extendToIndices could actually indicate that SI is live, contrary
   440:     // to the original IR.
   441:     if (Dominate(Defs, BB))
   442:       ExtTo.push_back(SI);
   443:   }
   444: 
   445:   if (!ExtTo.empty())
   446:     LIS->extendToIndices(Range, ExtTo, Undefs);
   447: 
   448:   // Remove <dead> flags from all defs that are not dead after live range
   449:   // extension, and collect all def operands. They will be used to generate
   450:   // the necessary implicit uses.
   451:   // At the same time, add <dead> flag to all defs that are actually dead.
   452:   // This can happen, for example, when a mux with identical inputs is
   453:   // replaced with a COPY: the use of the predicate register disappears and
   454:   // the dead can become dead.
   455:   std::set<RegisterRef> DefRegs;
   456:   for (auto &Seg : Range) {
   457:     if (!Seg.start.isRegister())
   458:       continue;
   459:     MachineInstr *DefI = LIS->getInstructionFromIndex(Seg.start);
   460:     for (auto &Op : DefI->operands()) {
   461:       auto P = IsRegDef(Op);
   462:       if (P.second && Seg.end.isDead()) {
   463:         Op.setIsDead(true);
   464:       } else if (P.first) {
   465:         DefRegs.insert(Op);
   466:         Op.setIsDead(false);
   467:       }
   468:     }
   469:   }
   470: 
   471:   // Now, add implicit uses to each predicated def that is reached
   472:   // by other defs.
   473:   for (auto &Seg : Range) {
   474:     if (!Seg.start.isRegister() || !Range.liveAt(Seg.start.getPrevSlot()))
   475:       continue;
   476:     MachineInstr *DefI = LIS->getInstructionFromIndex(Seg.start);
   477:     if (!HII->isPredicated(*DefI))
   478:       continue;
   479:     // Construct the set of all necessary implicit uses, based on the def
   480:     // operands in the instruction. We need to tie the implicit uses to
   481:     // the corresponding defs.
   482:     std::map<RegisterRef,unsigned> ImpUses;
   483:     for (unsigned i = 0, e = DefI->getNumOperands(); i != e; ++i) {
   484:       MachineOperand &Op = DefI->getOperand(i);
   485:       if (!Op.isReg() || !DefRegs.count(Op))
   486:         continue;
   487:       if (Op.isDef()) {
   488:         // Tied defs will always have corresponding uses, so no extra
   489:         // implicit uses are needed.
   490:         if (!Op.isTied())
   491:           ImpUses.insert({Op, i});
   492:       } else {
   493:         // This function can be called for the same register with different
   494:         // lane masks. If the def in this instruction was for the whole
   495:         // register, we can get here more than once. Avoid adding multiple
   496:         // implicit uses (or adding an implicit use when an explicit one is
   497:         // present).
   498:         if (Op.isTied())
   499:           ImpUses.erase(Op);
   500:       }
```
- EN: It declares or implements routines such as getInstructionFromIndex, insert, getInterval, computeSubRangeUndefs, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 getInstructionFromIndex, insert, getInterval, computeSubRangeUndefs, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 501-600 / 第 501-600 行

```cpp
   501:     }
   502:     if (ImpUses.empty())
   503:       continue;
   504:     MachineFunction &MF = *DefI->getParent()->getParent();
   505:     for (auto [R, DefIdx] : ImpUses) {
   506:       MachineInstrBuilder(MF, DefI).addReg(R.Reg, RegState::Implicit, R.Sub);
   507:       DefI->tieOperands(DefIdx, DefI->getNumOperands()-1);
   508:     }
   509:   }
   510: }
   511: 
   512: void HexagonExpandCondsets::updateDeadFlags(Register Reg) {
   513:   LiveInterval &LI = LIS->getInterval(Reg);
   514:   if (LI.hasSubRanges()) {
   515:     for (LiveInterval::SubRange &S : LI.subranges()) {
   516:       updateDeadsInRange(Reg, S.LaneMask, S);
   517:       LIS->shrinkToUses(S, Reg);
   518:     }
   519:     LI.clear();
   520:     LIS->constructMainRangeFromSubranges(LI);
   521:   } else {
   522:     updateDeadsInRange(Reg, MRI->getMaxLaneMaskForVReg(Reg), LI);
   523:   }
   524: }
   525: 
   526: void HexagonExpandCondsets::recalculateLiveInterval(Register Reg) {
   527:   LIS->removeInterval(Reg);
   528:   LIS->createAndComputeVirtRegInterval(Reg);
   529: }
   530: 
   531: void HexagonExpandCondsets::removeInstr(MachineInstr &MI) {
   532:   LIS->RemoveMachineInstrFromMaps(MI);
   533:   MI.eraseFromParent();
   534: }
   535: 
   536: void HexagonExpandCondsets::updateLiveness(const std::set<Register> &RegSet,
   537:                                            bool Recalc, bool UpdateKills,
   538:                                            bool UpdateDeads) {
   539:   UpdateKills |= UpdateDeads;
   540:   for (Register R : RegSet) {
   541:     if (!R.isVirtual()) {
   542:       assert(R.isPhysical());
   543:       // There shouldn't be any physical registers as operands, except
   544:       // possibly reserved registers.
   545:       assert(MRI->isReserved(R));
   546:       continue;
   547:     }
   548:     if (Recalc)
   549:       recalculateLiveInterval(R);
   550:     if (UpdateKills)
   551:       MRI->clearKillFlags(R);
   552:     if (UpdateDeads)
   553:       updateDeadFlags(R);
   554:     // Fixing <dead> flags may extend live ranges, so reset <kill> flags
   555:     // after that.
   556:     if (UpdateKills)
   557:       updateKillFlags(R);
   558:     assert(LIS->getInterval(R).verify());
   559:   }
   560: }
   561: 
   562: void HexagonExpandCondsets::distributeLiveIntervals(
   563:     const std::set<Register> &Regs) {
   564:   ConnectedVNInfoEqClasses EQC(*LIS);
   565:   for (Register R : Regs) {
   566:     if (!R.isVirtual())
   567:       continue;
   568:     LiveInterval &LI = LIS->getInterval(R);
   569:     unsigned NumComp = EQC.Classify(LI);
   570:     if (NumComp == 1)
   571:       continue;
   572: 
   573:     SmallVector<LiveInterval*> NewLIs;
   574:     const TargetRegisterClass *RC = MRI->getRegClass(LI.reg());
   575:     for (unsigned I = 1; I < NumComp; ++I) {
   576:       Register NewR = MRI->createVirtualRegister(RC);
   577:       NewLIs.push_back(&LIS->createEmptyInterval(NewR));
   578:     }
   579:     EQC.Distribute(LI, NewLIs.begin(), *MRI);
   580:   }
   581: }
   582: 
   583: /// Get the opcode for a conditional transfer of the value in SO (source
   584: /// operand). The condition (true/false) is given in Cond.
   585: unsigned HexagonExpandCondsets::getCondTfrOpcode(const MachineOperand &SO,
   586:       bool IfTrue) {
   587:   if (SO.isReg()) {
   588:     MCRegister PhysR;
   589:     RegisterRef RS = SO;
   590:     if (RS.Reg.isVirtual()) {
   591:       const TargetRegisterClass *VC = MRI->getRegClass(RS.Reg);
   592:       assert(VC->begin() != VC->end() && "Empty register class");
   593:       PhysR = *VC->begin();
   594:     } else {
   595:       PhysR = RS.Reg;
   596:     }
   597:     MCRegister PhysS = (RS.Sub == 0) ? PhysR : TRI->getSubReg(PhysR, RS.Sub);
   598:     const TargetRegisterClass *RC = TRI->getMinimalPhysRegClass(PhysS);
   599:     switch (TRI->getRegSizeInBits(*RC)) {
   600:       case 32:
```
- EN: It declares or implements routines such as getParent, MachineInstrBuilder, tieOperands, HexagonExpandCondsets::updateDeadFlags, ... (28 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonExpandCondsets, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getParent, MachineInstrBuilder, tieOperands, HexagonExpandCondsets::updateDeadFlags, ... (28 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonExpandCondsets，说明了它与同级后端组件的连接关系。

### Lines 601-700 / 第 601-700 行

```cpp
   601:         return IfTrue ? Hexagon::A2_tfrt : Hexagon::A2_tfrf;
   602:       case 64:
   603:         return IfTrue ? Hexagon::A2_tfrpt : Hexagon::A2_tfrpf;
   604:     }
   605:     llvm_unreachable("Invalid register operand");
   606:   }
   607:   switch (SO.getType()) {
   608:     case MachineOperand::MO_Immediate:
   609:     case MachineOperand::MO_FPImmediate:
   610:     case MachineOperand::MO_ConstantPoolIndex:
   611:     case MachineOperand::MO_TargetIndex:
   612:     case MachineOperand::MO_JumpTableIndex:
   613:     case MachineOperand::MO_ExternalSymbol:
   614:     case MachineOperand::MO_GlobalAddress:
   615:     case MachineOperand::MO_BlockAddress:
   616:       return IfTrue ? Hexagon::C2_cmoveit : Hexagon::C2_cmoveif;
   617:     default:
   618:       break;
   619:   }
   620:   llvm_unreachable("Unexpected source operand");
   621: }
   622: 
   623: /// Generate a conditional transfer, copying the value SrcOp to the
   624: /// destination register DstR:DstSR, and using the predicate register from
   625: /// PredOp. The Cond argument specifies whether the predicate is to be
   626: /// if(PredOp), or if(!PredOp).
   627: MachineInstr *HexagonExpandCondsets::genCondTfrFor(MachineOperand &SrcOp,
   628:       MachineBasicBlock::iterator At,
   629:       unsigned DstR, unsigned DstSR, const MachineOperand &PredOp,
   630:       bool PredSense, bool ReadUndef, bool ImpUse) {
   631:   MachineInstr *MI = SrcOp.getParent();
   632:   MachineBasicBlock &B = *At->getParent();
   633:   const DebugLoc &DL = MI->getDebugLoc();
   634: 
   635:   // Don't avoid identity copies here (i.e. if the source and the destination
   636:   // are the same registers). It is actually better to generate them here,
   637:   // since this would cause the copy to potentially be predicated in the next
   638:   // step. The predication will remove such a copy if it is unable to
   639:   /// predicate.
   640: 
   641:   unsigned Opc = getCondTfrOpcode(SrcOp, PredSense);
   642:   RegState DstState = RegState::Define | getUndefRegState(ReadUndef);
   643:   RegState PredState = getRegState(PredOp) & ~RegState::Kill;
   644:   MachineInstrBuilder MIB;
   645: 
   646:   if (SrcOp.isReg()) {
   647:     RegState SrcState = getRegState(SrcOp);
   648:     if (RegisterRef(SrcOp) == RegisterRef(DstR, DstSR))
   649:       SrcState &= ~RegState::Kill;
   650:     MIB = BuildMI(B, At, DL, HII->get(Opc))
   651:             .addReg(DstR, DstState, DstSR)
   652:             .addReg(PredOp.getReg(), PredState, PredOp.getSubReg())
   653:             .addReg(SrcOp.getReg(), SrcState, SrcOp.getSubReg());
   654:   } else {
   655:     MIB = BuildMI(B, At, DL, HII->get(Opc))
   656:             .addReg(DstR, DstState, DstSR)
   657:             .addReg(PredOp.getReg(), PredState, PredOp.getSubReg())
   658:             .add(SrcOp);
   659:   }
   660: 
   661:   LLVM_DEBUG(dbgs() << "created an initial copy: " << *MIB);
   662:   return &*MIB;
   663: }
   664: 
   665: /// Replace a MUX instruction MI with a pair A2_tfrt/A2_tfrf. This function
   666: /// performs all necessary changes to complete the replacement.
   667: bool HexagonExpandCondsets::split(MachineInstr &MI,
   668:                                   std::set<Register> &UpdRegs) {
   669:   if (TfrLimitActive) {
   670:     if (TfrCounter >= TfrLimit)
   671:       return false;
   672:     TfrCounter++;
   673:   }
   674:   LLVM_DEBUG(dbgs() << "\nsplitting " << printMBBReference(*MI.getParent())
   675:                     << ": " << MI);
   676:   MachineOperand &MD = MI.getOperand(0);  // Definition
   677:   MachineOperand &MP = MI.getOperand(1);  // Predicate register
   678:   assert(MD.isDef());
   679:   Register DR = MD.getReg(), DSR = MD.getSubReg();
   680:   bool ReadUndef = MD.isUndef();
   681:   MachineBasicBlock::iterator At = MI;
   682: 
   683:   auto updateRegs = [&UpdRegs] (const MachineInstr &MI) -> void {
   684:     for (auto &Op : MI.operands()) {
   685:       if (Op.isReg())
   686:         UpdRegs.insert(Op.getReg());
   687:     }
   688:   };
   689: 
   690:   // If this is a mux of the same register, just replace it with COPY.
   691:   // Ideally, this would happen earlier, so that register coalescing would
   692:   // see it.
   693:   MachineOperand &ST = MI.getOperand(2);
   694:   MachineOperand &SF = MI.getOperand(3);
   695:   if (ST.isReg() && SF.isReg()) {
   696:     RegisterRef RT(ST);
   697:     if (RT == RegisterRef(SF)) {
   698:       // Copy regs to update first.
   699:       updateRegs(MI);
   700:       MI.setDesc(HII->get(TargetOpcode::COPY));
```
- EN: It declares or implements routines such as llvm_unreachable, getParent, getDebugLoc, here, ... (16 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonExpandCondsets, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 llvm_unreachable, getParent, getDebugLoc, here, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonExpandCondsets，说明了它与同级后端组件的连接关系。

### Lines 701-800 / 第 701-800 行

```cpp
   701:       RegState S = getRegState(ST);
   702:       while (MI.getNumOperands() > 1)
   703:         MI.removeOperand(MI.getNumOperands()-1);
   704:       MachineFunction &MF = *MI.getParent()->getParent();
   705:       MachineInstrBuilder(MF, MI).addReg(RT.Reg, S, RT.Sub);
   706:       return true;
   707:     }
   708:   }
   709: 
   710:   // First, create the two individual conditional transfers, and add each
   711:   // of them to the live intervals information. Do that first and then remove
   712:   // the old instruction from live intervals.
   713:   MachineInstr *TfrT =
   714:       genCondTfrFor(ST, At, DR, DSR, MP, true, ReadUndef, false);
   715:   MachineInstr *TfrF =
   716:       genCondTfrFor(SF, At, DR, DSR, MP, false, ReadUndef, true);
   717:   LIS->InsertMachineInstrInMaps(*TfrT);
   718:   LIS->InsertMachineInstrInMaps(*TfrF);
   719: 
   720:   // Will need to recalculate live intervals for all registers in MI.
   721:   updateRegs(MI);
   722: 
   723:   removeInstr(MI);
   724:   return true;
   725: }
   726: 
   727: bool HexagonExpandCondsets::isPredicable(MachineInstr *MI) {
   728:   if (HII->isPredicated(*MI) || !HII->isPredicable(*MI))
   729:     return false;
   730:   if (MI->hasUnmodeledSideEffects() || MI->mayStore())
   731:     return false;
   732:   // Reject instructions with multiple defs (e.g. post-increment loads).
   733:   bool HasDef = false;
   734:   for (auto &Op : MI->operands()) {
   735:     if (!Op.isReg() || !Op.isDef())
   736:       continue;
   737:     if (HasDef)
   738:       return false;
   739:     HasDef = true;
   740:   }
   741:   for (auto &Mo : MI->memoperands()) {
   742:     if (Mo->isVolatile() || Mo->isAtomic())
   743:       return false;
   744:   }
   745:   return true;
   746: }
   747: 
   748: /// Find the reaching definition for a predicated use of RD. The RD is used
   749: /// under the conditions given by PredR and Cond, and this function will ignore
   750: /// definitions that set RD under the opposite conditions.
   751: MachineInstr *HexagonExpandCondsets::getReachingDefForPred(RegisterRef RD,
   752:       MachineBasicBlock::iterator UseIt, unsigned PredR, bool Cond) {
   753:   MachineBasicBlock &B = *UseIt->getParent();
   754:   MachineBasicBlock::iterator I = UseIt, S = B.begin();
   755:   if (I == S)
   756:     return nullptr;
   757: 
   758:   bool PredValid = true;
   759:   do {
   760:     --I;
   761:     MachineInstr *MI = &*I;
   762:     // Check if this instruction can be ignored, i.e. if it is predicated
   763:     // on the complementary condition.
   764:     if (PredValid && HII->isPredicated(*MI)) {
   765:       if (MI->readsRegister(PredR, /*TRI=*/nullptr) &&
   766:           (Cond != HII->isPredicatedTrue(*MI)))
   767:         continue;
   768:     }
   769: 
   770:     // Check the defs. If the PredR is defined, invalidate it. If RD is
   771:     // defined, return the instruction or 0, depending on the circumstances.
   772:     for (auto &Op : MI->operands()) {
   773:       if (!Op.isReg() || !Op.isDef())
   774:         continue;
   775:       RegisterRef RR = Op;
   776:       if (RR.Reg == PredR) {
   777:         PredValid = false;
   778:         continue;
   779:       }
   780:       if (RR.Reg != RD.Reg)
   781:         continue;
   782:       // If the "Reg" part agrees, there is still the subregister to check.
   783:       // If we are looking for %1:loreg, we can skip %1:hireg, but
   784:       // not %1 (w/o subregisters).
   785:       if (RR.Sub == RD.Sub)
   786:         return MI;
   787:       if (RR.Sub == 0 || RD.Sub == 0)
   788:         return nullptr;
   789:       // We have different subregisters, so we can continue looking.
   790:     }
   791:   } while (I != S);
   792: 
   793:   return nullptr;
   794: }
   795: 
   796: /// Check if the instruction MI can be safely moved over a set of instructions
   797: /// whose side-effects (in terms of register defs and uses) are expressed in
   798: /// the maps Defs and Uses. These maps reflect the conditional defs and uses
   799: /// that depend on the same predicate register to allow moving instructions
   800: /// over instructions predicated on the opposite condition.
```
- EN: It declares or implements routines such as getRegState, getParent, MachineInstrBuilder, genCondTfrFor, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonExpandCondsets, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getRegState, getParent, MachineInstrBuilder, genCondTfrFor, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonExpandCondsets，说明了它与同级后端组件的连接关系。

### Lines 801-900 / 第 801-900 行

```cpp
   801: bool HexagonExpandCondsets::canMoveOver(MachineInstr &MI, ReferenceMap &Defs,
   802:                                         ReferenceMap &Uses) {
   803:   // In order to be able to safely move MI over instructions that define
   804:   // "Defs" and use "Uses", no def operand from MI can be defined or used
   805:   // and no use operand can be defined.
   806:   for (auto &Op : MI.operands()) {
   807:     if (!Op.isReg())
   808:       continue;
   809:     RegisterRef RR = Op;
   810:     // For physical register we would need to check register aliases, etc.
   811:     // and we don't want to bother with that. It would be of little value
   812:     // before the actual register rewriting (from virtual to physical).
   813:     if (!RR.Reg.isVirtual())
   814:       return false;
   815:     // No redefs for any operand.
   816:     if (isRefInMap(RR, Defs, Exec_Then))
   817:       return false;
   818:     // For defs, there cannot be uses.
   819:     if (Op.isDef() && isRefInMap(RR, Uses, Exec_Then))
   820:       return false;
   821:   }
   822:   return true;
   823: }
   824: 
   825: /// Check if the instruction accessing memory (TheI) can be moved to the
   826: /// location ToI.
   827: bool HexagonExpandCondsets::canMoveMemTo(MachineInstr &TheI, MachineInstr &ToI,
   828:                                          bool IsDown) {
   829:   bool IsLoad = TheI.mayLoad(), IsStore = TheI.mayStore();
   830:   if (!IsLoad && !IsStore)
   831:     return true;
   832:   if (HII->areMemAccessesTriviallyDisjoint(TheI, ToI))
   833:     return true;
   834:   if (TheI.hasUnmodeledSideEffects())
   835:     return false;
   836: 
   837:   MachineBasicBlock::iterator StartI = IsDown ? TheI : ToI;
   838:   MachineBasicBlock::iterator EndI = IsDown ? ToI : TheI;
   839:   bool Ordered = TheI.hasOrderedMemoryRef();
   840: 
   841:   // Search for aliased memory reference in (StartI, EndI).
   842:   for (MachineInstr &MI : llvm::make_range(std::next(StartI), EndI)) {
   843:     if (MI.hasUnmodeledSideEffects())
   844:       return false;
   845:     bool L = MI.mayLoad(), S = MI.mayStore();
   846:     if (!L && !S)
   847:       continue;
   848:     if (Ordered && MI.hasOrderedMemoryRef())
   849:       return false;
   850: 
   851:     bool Conflict = (L && IsStore) || S;
   852:     if (Conflict)
   853:       return false;
   854:   }
   855:   return true;
   856: }
   857: 
   858: /// Generate a predicated version of MI (where the condition is given via
   859: /// PredR and Cond) at the point indicated by Where.
   860: void HexagonExpandCondsets::predicateAt(const MachineOperand &DefOp,
   861:                                         MachineInstr &MI,
   862:                                         MachineBasicBlock::iterator Where,
   863:                                         const MachineOperand &PredOp, bool Cond,
   864:                                         std::set<Register> &UpdRegs) {
   865:   // The problem with updating live intervals is that we can move one def
   866:   // past another def. In particular, this can happen when moving an A2_tfrt
   867:   // over an A2_tfrf defining the same register. From the point of view of
   868:   // live intervals, these two instructions are two separate definitions,
   869:   // and each one starts another live segment. LiveIntervals's "handleMove"
   870:   // does not allow such moves, so we need to handle it ourselves. To avoid
   871:   // invalidating liveness data while we are using it, the move will be
   872:   // implemented in 4 steps: (1) add a clone of the instruction MI at the
   873:   // target location, (2) update liveness, (3) delete the old instruction,
   874:   // and (4) update liveness again.
   875: 
   876:   MachineBasicBlock &B = *MI.getParent();
   877:   DebugLoc DL = Where->getDebugLoc();  // "Where" points to an instruction.
   878:   unsigned Opc = MI.getOpcode();
   879:   unsigned PredOpc = HII->getCondOpcode(Opc, !Cond);
   880:   MachineInstrBuilder MB = BuildMI(B, Where, DL, HII->get(PredOpc));
   881:   unsigned Ox = 0, NP = MI.getNumOperands();
   882:   // Skip all defs from MI first.
   883:   while (Ox < NP) {
   884:     MachineOperand &MO = MI.getOperand(Ox);
   885:     if (!MO.isReg() || !MO.isDef())
   886:       break;
   887:     Ox++;
   888:   }
   889:   // Add the new def, then the predicate register, then the rest of the
   890:   // operands.
   891:   MB.addReg(DefOp.getReg(), getRegState(DefOp), DefOp.getSubReg());
   892:   MB.addReg(PredOp.getReg(), getUndefRegState(PredOp.isUndef()),
   893:             PredOp.getSubReg());
   894:   while (Ox < NP) {
   895:     MachineOperand &MO = MI.getOperand(Ox);
   896:     if (!MO.isReg() || !MO.isImplicit())
   897:       MB.add(MO);
   898:     Ox++;
   899:   }
   900:   MB.cloneMemRefs(MI);
```
- EN: It declares or implements routines such as HexagonExpandCondsets::canMoveOver, memory, mayLoad, hasOrderedMemoryRef, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonExpandCondsets, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonExpandCondsets::canMoveOver, memory, mayLoad, hasOrderedMemoryRef, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonExpandCondsets，说明了它与同级后端组件的连接关系。

### Lines 901-1000 / 第 901-1000 行

```cpp
   901: 
   902:   MachineInstr *NewI = MB;
   903:   NewI->clearKillInfo();
   904:   LIS->InsertMachineInstrInMaps(*NewI);
   905: 
   906:   for (auto &Op : NewI->operands()) {
   907:     if (Op.isReg())
   908:       UpdRegs.insert(Op.getReg());
   909:   }
   910: }
   911: 
   912: /// In the range [First, Last], rename all references to the "old" register RO
   913: /// to the "new" register RN, but only in instructions predicated on the given
   914: /// condition.
   915: void HexagonExpandCondsets::renameInRange(RegisterRef RO, RegisterRef RN,
   916:       unsigned PredR, bool Cond, MachineBasicBlock::iterator First,
   917:       MachineBasicBlock::iterator Last) {
   918:   MachineBasicBlock::iterator End = std::next(Last);
   919:   for (MachineInstr &MI : llvm::make_range(First, End)) {
   920:     // Do not touch instructions that are not predicated, or are predicated
   921:     // on the opposite condition.
   922:     if (!HII->isPredicated(MI))
   923:       continue;
   924:     if (!MI.readsRegister(PredR, /*TRI=*/nullptr) ||
   925:         (Cond != HII->isPredicatedTrue(MI)))
   926:       continue;
   927: 
   928:     for (auto &Op : MI.operands()) {
   929:       if (!Op.isReg() || RO != RegisterRef(Op))
   930:         continue;
   931:       Op.setReg(RN.Reg);
   932:       Op.setSubReg(RN.Sub);
   933:       // In practice, this isn't supposed to see any defs.
   934:       assert(!Op.isDef() && "Not expecting a def");
   935:     }
   936:   }
   937: }
   938: 
   939: /// For a given conditional copy, predicate the definition of the source of
   940: /// the copy under the given condition (using the same predicate register as
   941: /// the copy).
   942: bool HexagonExpandCondsets::predicate(MachineInstr &TfrI, bool Cond,
   943:                                       std::set<Register> &UpdRegs) {
   944:   // TfrI - A2_tfr[tf] Instruction (not A2_tfrsi).
   945:   unsigned Opc = TfrI.getOpcode();
   946:   (void)Opc;
   947:   assert(Opc == Hexagon::A2_tfrt || Opc == Hexagon::A2_tfrf);
   948:   LLVM_DEBUG(dbgs() << "\nattempt to predicate if-" << (Cond ? "true" : "false")
   949:                     << ": " << TfrI);
   950: 
   951:   MachineOperand &MD = TfrI.getOperand(0);
   952:   MachineOperand &MP = TfrI.getOperand(1);
   953:   MachineOperand &MS = TfrI.getOperand(2);
   954:   // The source operand should be a <kill>. This is not strictly necessary,
   955:   // but it makes things a lot simpler. Otherwise, we would need to rename
   956:   // some registers, which would complicate the transformation considerably.
   957:   if (!MS.isKill())
   958:     return false;
   959:   // Avoid predicating instructions that define a subregister if subregister
   960:   // liveness tracking is not enabled.
   961:   if (MD.getSubReg() && !MRI->shouldTrackSubRegLiveness(MD.getReg()))
   962:     return false;
   963: 
   964:   RegisterRef RT(MS);
   965:   Register PredR = MP.getReg();
   966:   MachineInstr *DefI = getReachingDefForPred(RT, TfrI, PredR, Cond);
   967:   if (!DefI || !isPredicable(DefI))
   968:     return false;
   969: 
   970:   LLVM_DEBUG(dbgs() << "Source def: " << *DefI);
   971: 
   972:   // Collect the information about registers defined and used between the
   973:   // DefI and the TfrI.
   974:   // Map: reg -> bitmask of subregs
   975:   ReferenceMap Uses, Defs;
   976:   MachineBasicBlock::iterator DefIt = DefI, TfrIt = TfrI;
   977: 
   978:   // Check if the predicate register is valid between DefI and TfrI.
   979:   // If it is, we can then ignore instructions predicated on the negated
   980:   // conditions when collecting def and use information.
   981:   bool PredValid = true;
   982:   for (MachineInstr &MI : llvm::make_range(std::next(DefIt), TfrIt)) {
   983:     if (!MI.modifiesRegister(PredR, nullptr))
   984:       continue;
   985:     PredValid = false;
   986:     break;
   987:   }
   988: 
   989:   for (MachineInstr &MI : llvm::make_range(std::next(DefIt), TfrIt)) {
   990:     // If this instruction is predicated on the same register, it could
   991:     // potentially be ignored.
   992:     // By default assume that the instruction executes on the same condition
   993:     // as TfrI (Exec_Then), and also on the opposite one (Exec_Else).
   994:     unsigned Exec = Exec_Then | Exec_Else;
   995:     if (PredValid && HII->isPredicated(MI) &&
   996:         MI.readsRegister(PredR, /*TRI=*/nullptr))
   997:       Exec = (Cond == HII->isPredicatedTrue(MI)) ? Exec_Then : Exec_Else;
   998: 
   999:     for (auto &Op : MI.operands()) {
  1000:       if (!Op.isReg())
```
- EN: It declares or implements routines such as clearKillInfo, InsertMachineInstrInMaps, HexagonExpandCondsets::renameInRange, std::next, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonExpandCondsets, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 clearKillInfo, InsertMachineInstrInMaps, HexagonExpandCondsets::renameInRange, std::next, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonExpandCondsets，说明了它与同级后端组件的连接关系。

### Lines 1001-1100 / 第 1001-1100 行

```cpp
  1001:         continue;
  1002:       // We don't want to deal with physical registers. The reason is that
  1003:       // they can be aliased with other physical registers. Aliased virtual
  1004:       // registers must share the same register number, and can only differ
  1005:       // in the subregisters, which we are keeping track of. Physical
  1006:       // registers ters no longer have subregisters---their super- and
  1007:       // subregisters are other physical registers, and we are not checking
  1008:       // that.
  1009:       RegisterRef RR = Op;
  1010:       if (!RR.Reg.isVirtual())
  1011:         return false;
  1012: 
  1013:       ReferenceMap &Map = Op.isDef() ? Defs : Uses;
  1014:       if (Op.isDef() && Op.isUndef()) {
  1015:         assert(RR.Sub && "Expecting a subregister on <def,read-undef>");
  1016:         // If this is a <def,read-undef>, then it invalidates the non-written
  1017:         // part of the register. For the purpose of checking the validity of
  1018:         // the move, assume that it modifies the whole register.
  1019:         RR.Sub = 0;
  1020:       }
  1021:       addRefToMap(RR, Map, Exec);
  1022:     }
  1023:   }
  1024: 
  1025:   // The situation:
  1026:   //   RT = DefI
  1027:   //   ...
  1028:   //   RD = TfrI ..., RT
  1029: 
  1030:   // If the register-in-the-middle (RT) is used or redefined between
  1031:   // DefI and TfrI, we may not be able proceed with this transformation.
  1032:   // We can ignore a def that will not execute together with TfrI, and a
  1033:   // use that will. If there is such a use (that does execute together with
  1034:   // TfrI), we will not be able to move DefI down. If there is a use that
  1035:   // executed if TfrI's condition is false, then RT must be available
  1036:   // unconditionally (cannot be predicated).
  1037:   // Essentially, we need to be able to rename RT to RD in this segment.
  1038:   if (isRefInMap(RT, Defs, Exec_Then) || isRefInMap(RT, Uses, Exec_Else))
  1039:     return false;
  1040:   RegisterRef RD = MD;
  1041:   // If the predicate register is defined between DefI and TfrI, the only
  1042:   // potential thing to do would be to move the DefI down to TfrI, and then
  1043:   // predicate. The reaching def (DefI) must be movable down to the location
  1044:   // of the TfrI.
  1045:   // If the target register of the TfrI (RD) is not used or defined between
  1046:   // DefI and TfrI, consider moving TfrI up to DefI.
  1047:   bool CanUp =   canMoveOver(TfrI, Defs, Uses);
  1048:   bool CanDown = canMoveOver(*DefI, Defs, Uses);
  1049:   // The TfrI does not access memory, but DefI could. Check if it's safe
  1050:   // to move DefI down to TfrI.
  1051:   if (DefI->mayLoadOrStore()) {
  1052:     if (!canMoveMemTo(*DefI, TfrI, true))
  1053:       CanDown = false;
  1054:   }
  1055: 
  1056:   LLVM_DEBUG(dbgs() << "Can move up: " << (CanUp ? "yes" : "no")
  1057:                     << ", can move down: " << (CanDown ? "yes\n" : "no\n"));
  1058:   MachineBasicBlock::iterator PastDefIt = std::next(DefIt);
  1059:   if (CanUp)
  1060:     predicateAt(MD, *DefI, PastDefIt, MP, Cond, UpdRegs);
  1061:   else if (CanDown)
  1062:     predicateAt(MD, *DefI, TfrIt, MP, Cond, UpdRegs);
  1063:   else
  1064:     return false;
  1065: 
  1066:   if (RT != RD) {
  1067:     renameInRange(RT, RD, PredR, Cond, PastDefIt, TfrIt);
  1068:     UpdRegs.insert(RT.Reg);
  1069:   }
  1070: 
  1071:   removeInstr(TfrI);
  1072:   removeInstr(*DefI);
  1073:   return true;
  1074: }
  1075: 
  1076: /// Predicate all cases of conditional copies in the specified block.
  1077: bool HexagonExpandCondsets::predicateInBlock(MachineBasicBlock &B,
  1078:                                              std::set<Register> &UpdRegs) {
  1079:   bool Changed = false;
  1080:   for (MachineInstr &MI : llvm::make_early_inc_range(B)) {
  1081:     unsigned Opc = MI.getOpcode();
  1082:     if (Opc == Hexagon::A2_tfrt || Opc == Hexagon::A2_tfrf) {
  1083:       bool Done = predicate(MI, (Opc == Hexagon::A2_tfrt), UpdRegs);
  1084:       if (!Done) {
  1085:         // If we didn't predicate I, we may need to remove it in case it is
  1086:         // an "identity" copy, e.g.  %1 = A2_tfrt %2, %1.
  1087:         if (RegisterRef(MI.getOperand(0)) == RegisterRef(MI.getOperand(2))) {
  1088:           for (auto &Op : MI.operands()) {
  1089:             if (Op.isReg())
  1090:               UpdRegs.insert(Op.getReg());
  1091:           }
  1092:           removeInstr(MI);
  1093:         }
  1094:       }
  1095:       Changed |= Done;
  1096:     }
  1097:   }
  1098:   return Changed;
  1099: }
  1100: 
```
- EN: It declares or implements routines such as assert, addRefToMap, def, canMoveOver, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonExpandCondsets, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 assert, addRefToMap, def, canMoveOver, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonExpandCondsets，说明了它与同级后端组件的连接关系。

### Lines 1101-1200 / 第 1101-1200 行

```cpp
  1101: bool HexagonExpandCondsets::isIntReg(RegisterRef RR, unsigned &BW) {
  1102:   if (!RR.Reg.isVirtual())
  1103:     return false;
  1104:   const TargetRegisterClass *RC = MRI->getRegClass(RR.Reg);
  1105:   if (RC == &Hexagon::IntRegsRegClass) {
  1106:     BW = 32;
  1107:     return true;
  1108:   }
  1109:   if (RC == &Hexagon::DoubleRegsRegClass) {
  1110:     BW = (RR.Sub != 0) ? 32 : 64;
  1111:     return true;
  1112:   }
  1113:   return false;
  1114: }
  1115: 
  1116: bool HexagonExpandCondsets::isIntraBlocks(LiveInterval &LI) {
  1117:   for (LiveRange::Segment &LR : LI) {
  1118:     // Range must start at a register...
  1119:     if (!LR.start.isRegister())
  1120:       return false;
  1121:     // ...and end in a register or in a dead slot.
  1122:     if (!LR.end.isRegister() && !LR.end.isDead())
  1123:       return false;
  1124:   }
  1125:   return true;
  1126: }
  1127: 
  1128: bool HexagonExpandCondsets::coalesceRegisters(RegisterRef R1, RegisterRef R2) {
  1129:   if (CoaLimitActive) {
  1130:     if (CoaCounter >= CoaLimit)
  1131:       return false;
  1132:     CoaCounter++;
  1133:   }
  1134:   unsigned BW1, BW2;
  1135:   if (!isIntReg(R1, BW1) || !isIntReg(R2, BW2) || BW1 != BW2)
  1136:     return false;
  1137:   if (MRI->isLiveIn(R1.Reg))
  1138:     return false;
  1139:   if (MRI->isLiveIn(R2.Reg))
  1140:     return false;
  1141: 
  1142:   LiveInterval &L1 = LIS->getInterval(R1.Reg);
  1143:   LiveInterval &L2 = LIS->getInterval(R2.Reg);
  1144:   if (L2.empty())
  1145:     return false;
  1146:   if (L1.hasSubRanges() || L2.hasSubRanges())
  1147:     return false;
  1148:   bool Overlap = L1.overlaps(L2);
  1149: 
  1150:   LLVM_DEBUG(dbgs() << "compatible registers: ("
  1151:                     << (Overlap ? "overlap" : "disjoint") << ")\n  "
  1152:                     << printReg(R1.Reg, TRI, R1.Sub) << "  " << L1 << "\n  "
  1153:                     << printReg(R2.Reg, TRI, R2.Sub) << "  " << L2 << "\n");
  1154:   if (R1.Sub || R2.Sub)
  1155:     return false;
  1156:   if (Overlap)
  1157:     return false;
  1158: 
  1159:   // Coalescing could have a negative impact on scheduling, so try to limit
  1160:   // to some reasonable extent. Only consider coalescing segments, when one
  1161:   // of them does not cross basic block boundaries.
  1162:   if (!isIntraBlocks(L1) && !isIntraBlocks(L2))
  1163:     return false;
  1164: 
  1165:   MRI->replaceRegWith(R2.Reg, R1.Reg);
  1166: 
  1167:   // Move all live segments from L2 to L1.
  1168:   using ValueInfoMap = DenseMap<VNInfo *, VNInfo *>;
  1169:   ValueInfoMap VM;
  1170:   for (LiveRange::Segment &I : L2) {
  1171:     VNInfo *NewVN, *OldVN = I.valno;
  1172:     ValueInfoMap::iterator F = VM.find(OldVN);
  1173:     if (F == VM.end()) {
  1174:       NewVN = L1.getNextValue(I.valno->def, LIS->getVNInfoAllocator());
  1175:       VM.insert(std::make_pair(OldVN, NewVN));
  1176:     } else {
  1177:       NewVN = F->second;
  1178:     }
  1179:     L1.addSegment(LiveRange::Segment(I.start, I.end, NewVN));
  1180:   }
  1181:   while (!L2.empty())
  1182:     L2.removeSegment(*L2.begin());
  1183:   LIS->removeInterval(R2.Reg);
  1184: 
  1185:   updateKillFlags(R1.Reg);
  1186:   LLVM_DEBUG(dbgs() << "coalesced: " << L1 << "\n");
  1187:   assert(L1.verify());
  1188: 
  1189:   return true;
  1190: }
  1191: 
  1192: /// Attempt to coalesce one of the source registers to a MUX instruction with
  1193: /// the destination register. This could lead to having only one predicated
  1194: /// instruction in the end instead of two.
  1195: bool HexagonExpandCondsets::coalesceSegments(
  1196:     const SmallVectorImpl<MachineInstr *> &Condsets,
  1197:     std::set<Register> &UpdRegs) {
  1198:   SmallVector<MachineInstr*,16> TwoRegs;
  1199:   for (MachineInstr *MI : Condsets) {
  1200:     MachineOperand &S1 = MI->getOperand(2), &S2 = MI->getOperand(3);
```
- EN: It declares or implements routines such as HexagonExpandCondsets::isIntReg, getRegClass, HexagonExpandCondsets::isIntraBlocks, HexagonExpandCondsets::coalesceRegisters, ... (17 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonExpandCondsets, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonExpandCondsets::isIntReg, getRegClass, HexagonExpandCondsets::isIntraBlocks, HexagonExpandCondsets::coalesceRegisters, ... (17 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonExpandCondsets，说明了它与同级后端组件的连接关系。

### Lines 1201-1300 / 第 1201-1300 行

```cpp
  1201:     if (!S1.isReg() && !S2.isReg())
  1202:       continue;
  1203:     TwoRegs.push_back(MI);
  1204:   }
  1205: 
  1206:   bool Changed = false;
  1207:   for (MachineInstr *CI : TwoRegs) {
  1208:     RegisterRef RD = CI->getOperand(0);
  1209:     RegisterRef RP = CI->getOperand(1);
  1210:     MachineOperand &S1 = CI->getOperand(2), &S2 = CI->getOperand(3);
  1211:     bool Done = false;
  1212:     // Consider this case:
  1213:     //   %1 = instr1 ...
  1214:     //   %2 = instr2 ...
  1215:     //   %0 = C2_mux ..., %1, %2
  1216:     // If %0 was coalesced with %1, we could end up with the following
  1217:     // code:
  1218:     //   %0 = instr1 ...
  1219:     //   %2 = instr2 ...
  1220:     //   %0 = A2_tfrf ..., %2
  1221:     // which will later become:
  1222:     //   %0 = instr1 ...
  1223:     //   %0 = instr2_cNotPt ...
  1224:     // i.e. there will be an unconditional definition (instr1) of %0
  1225:     // followed by a conditional one. The output dependency was there before
  1226:     // and it unavoidable, but if instr1 is predicable, we will no longer be
  1227:     // able to predicate it here.
  1228:     // To avoid this scenario, don't coalesce the destination register with
  1229:     // a source register that is defined by a predicable instruction.
  1230:     if (S1.isReg()) {
  1231:       RegisterRef RS = S1;
  1232:       MachineInstr *RDef = getReachingDefForPred(RS, CI, RP.Reg, true);
  1233:       if (!RDef || !HII->isPredicable(*RDef)) {
  1234:         Done = coalesceRegisters(RD, RegisterRef(S1));
  1235:         if (Done) {
  1236:           UpdRegs.insert(RD.Reg);
  1237:           UpdRegs.insert(S1.getReg());
  1238:         }
  1239:       }
  1240:     }
  1241:     if (!Done && S2.isReg()) {
  1242:       RegisterRef RS = S2;
  1243:       MachineInstr *RDef = getReachingDefForPred(RS, CI, RP.Reg, false);
  1244:       if (!RDef || !HII->isPredicable(*RDef)) {
  1245:         Done = coalesceRegisters(RD, RegisterRef(S2));
  1246:         if (Done) {
  1247:           UpdRegs.insert(RD.Reg);
  1248:           UpdRegs.insert(S2.getReg());
  1249:         }
  1250:       }
  1251:     }
  1252:     Changed |= Done;
  1253:   }
  1254:   return Changed;
  1255: }
  1256: 
  1257: bool HexagonExpandCondsets::runOnMachineFunction(MachineFunction &MF) {
  1258:   if (skipFunction(MF.getFunction()))
  1259:     return false;
  1260: 
  1261:   HII = static_cast<const HexagonInstrInfo*>(MF.getSubtarget().getInstrInfo());
  1262:   TRI = MF.getSubtarget().getRegisterInfo();
  1263:   MDT = &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  1264:   LIS = &getAnalysis<LiveIntervalsWrapperPass>().getLIS();
  1265:   MRI = &MF.getRegInfo();
  1266: 
  1267:   LLVM_DEBUG(LIS->print(dbgs() << "Before expand-condsets\n"));
  1268: 
  1269:   bool Changed = false;
  1270:   std::set<Register> CoalUpd, PredUpd;
  1271: 
  1272:   SmallVector<MachineInstr*,16> Condsets;
  1273:   for (auto &B : MF) {
  1274:     for (auto &I : B) {
  1275:       if (isCondset(I))
  1276:         Condsets.push_back(&I);
  1277:     }
  1278:   }
  1279: 
  1280:   // Try to coalesce the target of a mux with one of its sources.
  1281:   // This could eliminate a register copy in some circumstances.
  1282:   Changed |= coalesceSegments(Condsets, CoalUpd);
  1283: 
  1284:   // Update kill flags on all source operands. This is done here because
  1285:   // at this moment (when expand-condsets runs), there are no kill flags
  1286:   // in the IR (they have been removed by live range analysis).
  1287:   // Updating them right before we split is the easiest, because splitting
  1288:   // adds definitions which would interfere with updating kills afterwards.
  1289:   std::set<Register> KillUpd;
  1290:   for (MachineInstr *MI : Condsets) {
  1291:     for (MachineOperand &Op : MI->operands()) {
  1292:       if (Op.isReg() && Op.isUse()) {
  1293:         if (!CoalUpd.count(Op.getReg()))
  1294:           KillUpd.insert(Op.getReg());
  1295:       }
  1296:     }
  1297:   }
  1298:   updateLiveness(KillUpd, false, true, false);
  1299:   LLVM_DEBUG(LIS->print(dbgs() << "After coalescing\n"));
  1300: 
```
- EN: It declares or implements routines such as push_back, getOperand, definition, getReachingDefForPred, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonExpandCondsets, HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 push_back, getOperand, definition, getReachingDefForPred, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonExpandCondsets, HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 1301-1344 / 第 1301-1344 行

```cpp
  1301:   // First, simply split all muxes into a pair of conditional transfers
  1302:   // and update the live intervals to reflect the new arrangement. The
  1303:   // goal is to update the kill flags, since predication will rely on
  1304:   // them.
  1305:   for (MachineInstr *MI : Condsets)
  1306:     Changed |= split(*MI, PredUpd);
  1307:   Condsets.clear(); // The contents of Condsets are invalid here anyway.
  1308: 
  1309:   // Do not update live ranges after splitting. Recalculation of live
  1310:   // intervals removes kill flags, which were preserved by splitting on
  1311:   // the source operands of condsets. These kill flags are needed by
  1312:   // predication, and after splitting they are difficult to recalculate
  1313:   // (because of predicated defs), so make sure they are left untouched.
  1314:   // Predication does not use live intervals.
  1315:   LLVM_DEBUG(LIS->print(dbgs() << "After splitting\n"));
  1316: 
  1317:   // Traverse all blocks and collapse predicable instructions feeding
  1318:   // conditional transfers into predicated instructions.
  1319:   // Walk over all the instructions again, so we may catch pre-existing
  1320:   // cases that were not created in the previous step.
  1321:   for (auto &B : MF)
  1322:     Changed |= predicateInBlock(B, PredUpd);
  1323:   LLVM_DEBUG(LIS->print(dbgs() << "After predicating\n"));
  1324: 
  1325:   PredUpd.insert(CoalUpd.begin(), CoalUpd.end());
  1326:   updateLiveness(PredUpd, true, true, true);
  1327: 
  1328:   if (Changed)
  1329:     distributeLiveIntervals(PredUpd);
  1330: 
  1331:   LLVM_DEBUG({
  1332:     if (Changed)
  1333:       LIS->print(dbgs() << "After expand-condsets\n");
  1334:   });
  1335: 
  1336:   return Changed;
  1337: }
  1338: 
  1339: //===----------------------------------------------------------------------===//
  1340: //                         Public Constructor Functions
  1341: //===----------------------------------------------------------------------===//
  1342: FunctionPass *llvm::createHexagonExpandCondsets() {
  1343:   return new HexagonExpandCondsets();
  1344: }
```
- EN: It declares or implements routines such as clear, LLVM_DEBUG, insert, updateLiveness, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonExpandCondsets, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 clear, LLVM_DEBUG, insert, updateLiveness, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonExpandCondsets，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- Hexagon backend integration / Hexagon 后端集成
- LLVM code generation plumbing / LLVM 代码生成基础设施

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, llvm/ADT/DenseMap.h, llvm/ADT/SetVector.h, llvm/ADT/SmallVector.h, llvm/ADT/StringRef.h, llvm/CodeGen/LiveInterval.h, llvm/CodeGen/LiveIntervals.h, llvm/CodeGen/MachineBasicBlock.h, ... (34 total)`
- Hexagon symbols / Hexagon 符号: `HexagonExpandCondsets, HexagonInstrInfo, HexagonRegisterInfo, HexagonExpandCondsetsID`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
