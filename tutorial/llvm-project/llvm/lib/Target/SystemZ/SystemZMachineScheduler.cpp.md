# SystemZMachineScheduler.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZMachineScheduler.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Provides target-specific implementation details for the SystemZ backend.
- **用途 (CN)**: 提供 SystemZ 后端的目标相关实现细节。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```cpp
   1: //-- SystemZMachineScheduler.cpp - SystemZ Scheduler Interface -*- C++ -*---==//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "SystemZMachineScheduler.h"
  10: #include "llvm/CodeGen/MachineLoopInfo.h"
  11: 
  12: using namespace llvm;
  13: 
  14: #define DEBUG_TYPE "machine-scheduler"
  15: 
  16: /// Pre-RA scheduling ///
  17: 
  18: static bool isRegDef(const MachineOperand &MO) {
  19:   return MO.isReg() && MO.isDef();
  20: }
  21: 
  22: void SystemZPreRASchedStrategy::initializeLatencyReduction() {
  23:   // Enable latency reduction for a region that has a considerable amount of
  24:   // data sequences that should be interlaved. These are SUs that only have
  25:   // one data predecessor / successor edge(s) to their adjacent instruction(s)
  26:   // in the input order. Disable if region has many SUs relative to the
  27:   // overall height.
  28:   unsigned DAGHeight = 0;
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZMachineScheduler.h`, `MachineLoopInfo.h` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `isRegDef`, `SystemZPreRASchedStrategy::initializeLatencyReduction`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZMachineScheduler.h`, `MachineLoopInfo.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `isRegDef`, `SystemZPreRASchedStrategy::initializeLatencyReduction` 等函数。

### Lines 29-56 / 第 29-56 行
```cpp
  29:   for (unsigned Idx = 0, End = DAG->SUnits.size(); Idx != End; ++Idx)
  30:     DAGHeight = std::max(DAGHeight, DAG->SUnits[Idx].getHeight());
  31:   RegionPolicy.DisableLatencyHeuristic =
  32:       DAG->SUnits.size() >= 3 * std::max(DAGHeight, 1u);
  33:   if ((HasDataSequences = !RegionPolicy.DisableLatencyHeuristic)) {
  34:     unsigned CurrSequence = 0, NumSeqNodes = 0;
  35:     auto countSequence = [&CurrSequence, &NumSeqNodes]() {
  36:       if (CurrSequence >= 2)
  37:         NumSeqNodes += CurrSequence;
  38:       CurrSequence = 0;
  39:     };
  40:     for (unsigned Idx = 0, End = DAG->SUnits.size(); Idx != End; ++Idx) {
  41:       const SUnit *SU = &DAG->SUnits[Idx];
  42:       bool InDataSequence = true;
  43:       // One Data pred to MI just above, or no preds.
  44:       unsigned NumPreds = 0;
  45:       for (const SDep &Pred : SU->Preds)
  46:         if (++NumPreds != 1 || Pred.getKind() != SDep::Data ||
  47:             Pred.getSUnit()->NodeNum != Idx - 1)
  48:           InDataSequence = false;
  49:       // One Data succ or no succs (ignoring ExitSU).
  50:       unsigned NumSuccs = 0;
  51:       for (const SDep &Succ : SU->Succs)
  52:         if (Succ.getSUnit() != &DAG->ExitSU &&
  53:             (++NumSuccs != 1 || Succ.getKind() != SDep::Data))
  54:           InDataSequence = false;
  55:       // Another type of node or one that does not have a single data pred
  56:       // ends any previous sequence.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 57-84 / 第 57-84 行
```cpp
  57:       if (!InDataSequence || !NumPreds)
  58:         countSequence();
  59:       if (InDataSequence)
  60:         CurrSequence++;
  61:     }
  62:     countSequence();
  63:     if (NumSeqNodes >= std::max(size_t(4), DAG->SUnits.size() / 4)) {
  64:       LLVM_DEBUG(dbgs() << "Number of nodes in def-use sequences: "
  65:                         << NumSeqNodes << ". ";);
  66:     } else
  67:       HasDataSequences = false;
  68:   }
  69: }
  70: 
  71: bool SystemZPreRASchedStrategy::definesCmp0Src(const MachineInstr *MI,
  72:                                                bool CCDef) const {
  73:   if (Cmp0SrcReg != SystemZ::NoRegister && MI->getNumOperands() &&
  74:       (MI->getDesc().hasImplicitDefOfPhysReg(SystemZ::CC) || !CCDef)) {
  75:     const MachineOperand &MO0 = MI->getOperand(0);
  76:     if (isRegDef(MO0) && MO0.getReg() == Cmp0SrcReg)
  77:       return true;
  78:   }
  79:   return false;
  80: }
  81: 
  82: bool SystemZPreRASchedStrategy::tryCandidate(SchedCandidate &Cand,
  83:                                              SchedCandidate &TryCand,
  84:                                              SchedBoundary *Zone) const {
```
- **EN**: The range implements or declares functions including `SystemZPreRASchedStrategy::definesCmp0Src`, `SystemZPreRASchedStrategy::tryCandidate`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZPreRASchedStrategy::definesCmp0Src`, `SystemZPreRASchedStrategy::tryCandidate` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 85-112 / 第 85-112 行
```cpp
  85:   assert(Zone && !Zone->isTop() && "Bottom-Up scheduling only.");
  86: 
  87:   // Initialize the candidate if needed.
  88:   if (!Cand.isValid()) {
  89:     TryCand.Reason = FirstValid;
  90:     return true;
  91:   }
  92: 
  93:   // Bias physreg defs and copies to their uses and definitions respectively.
  94:   if (tryBiasPhysRegs(TryCand, Cand, Zone, /*BiasPRegsExtra=*/true))
  95:     return TryCand.Reason != NoCand;
  96: 
  97:   // Don't extend the scheduled latency in regions with many nodes in data
  98:   // sequences, or for (single block loop) regions that are acyclically
  99:   // (within a single loop iteration) latency limited. IsAcyclicLatencyLimited
 100:   // is set only after initialization in registerRoots(), which is why it is
 101:   // checked here instead of earlier.
 102:   if (!RegionPolicy.DisableLatencyHeuristic &&
 103:       (HasDataSequences || Rem.IsAcyclicLatencyLimited))
 104:     if (const SUnit *HigherSU =
 105:             TryCand.SU->getHeight() > Cand.SU->getHeight()   ? TryCand.SU
 106:             : TryCand.SU->getHeight() < Cand.SU->getHeight() ? Cand.SU
 107:                                                              : nullptr)
 108:       if (HigherSU->getHeight() > Zone->getScheduledLatency() &&
 109:           HigherSU->getDepth() < computeRemLatency(*Zone)) {
 110:         // One or both SUs increase the scheduled latency.
 111:         tryLess(TryCand.SU->getHeight(), Cand.SU->getHeight(), TryCand, Cand,
 112:                 GenericSchedulerBase::BotHeightReduce);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 113-140 / 第 113-140 行
```cpp
 113:         return TryCand.Reason != NoCand;
 114:       }
 115: 
 116:   // Weak edges help copy coalescing.
 117:   if (tryLess(TryCand.SU->WeakSuccsLeft, Cand.SU->WeakSuccsLeft, TryCand, Cand,
 118:               Weak))
 119:     return TryCand.Reason != NoCand;
 120: 
 121:   // Help compare with zero elimination.
 122:   if (tryGreater(definesCmp0Src(TryCand.SU->getInstr()),
 123:                  definesCmp0Src(Cand.SU->getInstr()), TryCand, Cand, Weak))
 124:     return TryCand.Reason != NoCand;
 125: 
 126:   // Fall through to original instruction order.
 127:   if (TryCand.SU->NodeNum > Cand.SU->NodeNum) {
 128:     TryCand.Reason = NodeOrder;
 129:     return true;
 130:   }
 131: 
 132:   return false;
 133: }
 134: 
 135: void SystemZPreRASchedStrategy::initPolicy(MachineBasicBlock::iterator Begin,
 136:                                            MachineBasicBlock::iterator End,
 137:                                            unsigned NumRegionInstrs) {
 138:   // Avoid setting up the register pressure tracker for small regions to save
 139:   // compile time. Currently only used for computeCyclicCriticalPath() which
 140:   // is used for single block loops.
```
- **EN**: The range implements or declares functions including `SystemZPreRASchedStrategy::initPolicy`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZPreRASchedStrategy::initPolicy` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 141-168 / 第 141-168 行
```cpp
 141:   MachineBasicBlock *MBB = Begin->getParent();
 142:   RegionPolicy.ShouldTrackPressure =
 143:       MBB->isSuccessor(MBB) && NumRegionInstrs >= 8;
 144: 
 145:   // These heuristics has so far seemed to work better without adding a
 146:   // top-down boundary.
 147:   RegionPolicy.OnlyBottomUp = true;
 148:   BotIdx = NumRegionInstrs - 1;
 149:   this->NumRegionInstrs = NumRegionInstrs;
 150: }
 151: 
 152: void SystemZPreRASchedStrategy::initialize(ScheduleDAGMI *dag) {
 153:   GenericScheduler::initialize(dag);
 154: 
 155:   Cmp0SrcReg = SystemZ::NoRegister;
 156: 
 157:   initializeLatencyReduction();
 158:   LLVM_DEBUG(dbgs() << "Latency scheduling " << (HasDataSequences ? "" : "not ")
 159:                     << "enabled for data sequences.\n";);
 160: }
 161: 
 162: void SystemZPreRASchedStrategy::schedNode(SUnit *SU, bool IsTopNode) {
 163:   GenericScheduler::schedNode(SU, IsTopNode);
 164: 
 165:   const SystemZInstrInfo *TII = static_cast<const SystemZInstrInfo *>(DAG->TII);
 166:   MachineInstr *MI = SU->getInstr();
 167:   if (TII->isCompareZero(*MI))
 168:     Cmp0SrcReg = TII->getCompareSourceReg(*MI);
```
- **EN**: The range implements or declares functions including `SystemZPreRASchedStrategy::initialize`, `LLVM_DEBUG`, `SystemZPreRASchedStrategy::schedNode`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZPreRASchedStrategy::initialize`, `LLVM_DEBUG`, `SystemZPreRASchedStrategy::schedNode` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 169-196 / 第 169-196 行
```cpp
 169:   else if (MI->getDesc().hasImplicitDefOfPhysReg(SystemZ::CC) ||
 170:            definesCmp0Src(MI, /*CCDef=*/false))
 171:     Cmp0SrcReg = SystemZ::NoRegister;
 172: }
 173: 
 174: /// Post-RA scheduling ///
 175: 
 176: #ifndef NDEBUG
 177: // Print the set of SUs
 178: void SystemZPostRASchedStrategy::SUSet::
 179: dump(SystemZHazardRecognizer &HazardRec) const {
 180:   dbgs() << "{";
 181:   for (auto &SU : *this) {
 182:     HazardRec.dumpSU(SU, dbgs());
 183:     if (SU != *rbegin())
 184:       dbgs() << ",  ";
 185:   }
 186:   dbgs() << "}\n";
 187: }
 188: #endif
 189: 
 190: // Try to find a single predecessor that would be interesting for the
 191: // scheduler in the top-most region of MBB.
 192: static MachineBasicBlock *getSingleSchedPred(MachineBasicBlock *MBB,
 193:                                              const MachineLoop *Loop) {
 194:   MachineBasicBlock *PredMBB = nullptr;
 195:   if (MBB->pred_size() == 1)
 196:     PredMBB = *MBB->pred_begin();
```
- **EN**: The range implements or declares functions including `dump`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `dump` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 197-224 / 第 197-224 行
```cpp
 197: 
 198:   // The loop header has two predecessors, return the latch, but not for a
 199:   // single block loop.
 200:   if (MBB->pred_size() == 2 && Loop != nullptr && Loop->getHeader() == MBB) {
 201:     for (MachineBasicBlock *Pred : MBB->predecessors())
 202:       if (Loop->contains(Pred))
 203:         PredMBB = (Pred == MBB ? nullptr : Pred);
 204:   }
 205: 
 206:   assert ((PredMBB == nullptr || !Loop || Loop->contains(PredMBB))
 207:           && "Loop MBB should not consider predecessor outside of loop.");
 208: 
 209:   return PredMBB;
 210: }
 211: 
 212: void SystemZPostRASchedStrategy::
 213: advanceTo(MachineBasicBlock::iterator NextBegin) {
 214:   MachineBasicBlock::iterator LastEmittedMI = HazardRec->getLastEmittedMI();
 215:   MachineBasicBlock::iterator I =
 216:     ((LastEmittedMI != nullptr && LastEmittedMI->getParent() == MBB) ?
 217:      std::next(LastEmittedMI) : MBB->begin());
 218: 
 219:   for (; I != NextBegin; ++I) {
 220:     if (I->isPosition() || I->isDebugInstr())
 221:       continue;
 222:     HazardRec->emitInstruction(&*I);
 223:   }
 224: }
```
- **EN**: The range implements or declares functions including `assert`, `advanceTo`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `assert`, `advanceTo` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 225-252 / 第 225-252 行
```cpp
 225: 
 226: void SystemZPostRASchedStrategy::initialize(ScheduleDAGMI *dag) {
 227:   Available.clear();  // -misched-cutoff.
 228:   LLVM_DEBUG(HazardRec->dumpState(););
 229: }
 230: 
 231: void SystemZPostRASchedStrategy::enterMBB(MachineBasicBlock *NextMBB) {
 232:   assert ((SchedStates.find(NextMBB) == SchedStates.end()) &&
 233:           "Entering MBB twice?");
 234:   LLVM_DEBUG(dbgs() << "** Entering " << printMBBReference(*NextMBB));
 235: 
 236:   MBB = NextMBB;
 237: 
 238:   /// Create a HazardRec for MBB, save it in SchedStates and set HazardRec to
 239:   /// point to it.
 240:   HazardRec = SchedStates[MBB] = new SystemZHazardRecognizer(TII, &SchedModel);
 241:   LLVM_DEBUG(const MachineLoop *Loop = MLI->getLoopFor(MBB);
 242:              if (Loop && Loop->getHeader() == MBB) dbgs() << " (Loop header)";
 243:              dbgs() << ":\n";);
 244: 
 245:   // Try to take over the state from a single predecessor, if it has been
 246:   // scheduled. If this is not possible, we are done.
 247:   MachineBasicBlock *SinglePredMBB =
 248:     getSingleSchedPred(MBB, MLI->getLoopFor(MBB));
 249:   if (SinglePredMBB == nullptr)
 250:     return;
 251:   auto It = SchedStates.find(SinglePredMBB);
 252:   if (It == SchedStates.end())
```
- **EN**: The range implements or declares functions including `SystemZPostRASchedStrategy::initialize`, `SystemZPostRASchedStrategy::enterMBB`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZPostRASchedStrategy::initialize`, `SystemZPostRASchedStrategy::enterMBB` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-280 / 第 253-280 行
```cpp
 253:     return;
 254: 
 255:   LLVM_DEBUG(dbgs() << "** Continued scheduling from "
 256:                     << printMBBReference(*SinglePredMBB) << "\n";);
 257: 
 258:   HazardRec->copyState(It->second);
 259:   LLVM_DEBUG(HazardRec->dumpState(););
 260: 
 261:   // Emit incoming terminator(s). Be optimistic and assume that branch
 262:   // prediction will generally do "the right thing".
 263:   for (MachineInstr &MI : SinglePredMBB->terminators()) {
 264:     LLVM_DEBUG(dbgs() << "** Emitting incoming branch: "; MI.dump(););
 265:     bool TakenBranch = (MI.isBranch() &&
 266:                         (TII->getBranchInfo(MI).isIndirect() ||
 267:                          TII->getBranchInfo(MI).getMBBTarget() == MBB));
 268:     HazardRec->emitInstruction(&MI, TakenBranch);
 269:     if (TakenBranch)
 270:       break;
 271:   }
 272: }
 273: 
 274: void SystemZPostRASchedStrategy::leaveMBB() {
 275:   LLVM_DEBUG(dbgs() << "** Leaving " << printMBBReference(*MBB) << "\n";);
 276: 
 277:   // Advance to first terminator. The successor block will handle terminators
 278:   // dependent on CFG layout (T/NT branch etc).
 279:   advanceTo(MBB->getFirstTerminator());
 280: }
```
- **EN**: The range implements or declares functions including `SystemZPostRASchedStrategy::leaveMBB`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZPostRASchedStrategy::leaveMBB` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 281-308 / 第 281-308 行
```cpp
 281: 
 282: SystemZPostRASchedStrategy::
 283: SystemZPostRASchedStrategy(const MachineSchedContext *C)
 284:   : MLI(C->MLI),
 285:     TII(static_cast<const SystemZInstrInfo *>
 286:         (C->MF->getSubtarget().getInstrInfo())),
 287:     MBB(nullptr), HazardRec(nullptr) {
 288:   const TargetSubtargetInfo *ST = &C->MF->getSubtarget();
 289:   SchedModel.init(ST);
 290: }
 291: 
 292: SystemZPostRASchedStrategy::~SystemZPostRASchedStrategy() {
 293:   // Delete hazard recognizers kept around for each MBB.
 294:   for (auto I : SchedStates) {
 295:     SystemZHazardRecognizer *hazrec = I.second;
 296:     delete hazrec;
 297:   }
 298: }
 299: 
 300: void SystemZPostRASchedStrategy::initPolicy(MachineBasicBlock::iterator Begin,
 301:                                             MachineBasicBlock::iterator End,
 302:                                             unsigned NumRegionInstrs) {
 303:   // Don't emit the terminators.
 304:   if (Begin->isTerminator())
 305:     return;
 306: 
 307:   // Emit any instructions before start of region.
 308:   advanceTo(Begin);
```
- **EN**: The range implements or declares functions including `SystemZPostRASchedStrategy`, `SystemZPostRASchedStrategy::initPolicy`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZPostRASchedStrategy`, `SystemZPostRASchedStrategy::initPolicy` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 309-336 / 第 309-336 行
```cpp
 309: }
 310: 
 311: // Pick the next node to schedule.
 312: SUnit *SystemZPostRASchedStrategy::pickNode(bool &IsTopNode) {
 313:   // Only scheduling top-down.
 314:   IsTopNode = true;
 315: 
 316:   if (Available.empty())
 317:     return nullptr;
 318: 
 319:   // If only one choice, return it.
 320:   if (Available.size() == 1) {
 321:     LLVM_DEBUG(dbgs() << "** Only one: ";
 322:                HazardRec->dumpSU(*Available.begin(), dbgs()); dbgs() << "\n";);
 323:     return *Available.begin();
 324:   }
 325: 
 326:   // All nodes that are possible to schedule are stored in the Available set.
 327:   LLVM_DEBUG(dbgs() << "** Available: "; Available.dump(*HazardRec););
 328: 
 329:   Candidate Best;
 330:   for (auto *SU : Available) {
 331: 
 332:     // SU is the next candidate to be compared against current Best.
 333:     Candidate c(SU, *HazardRec);
 334: 
 335:     // Remeber which SU is the best candidate.
 336:     if (Best.SU == nullptr || c < Best) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 337-364 / 第 337-364 行
```cpp
 337:       Best = c;
 338:       LLVM_DEBUG(dbgs() << "** Best so far: ";);
 339:     } else
 340:       LLVM_DEBUG(dbgs() << "** Tried      : ";);
 341:     LLVM_DEBUG(HazardRec->dumpSU(c.SU, dbgs()); c.dumpCosts();
 342:                dbgs() << " Height:" << c.SU->getHeight(); dbgs() << "\n";);
 343: 
 344:     // Once we know we have seen all SUs that affect grouping or use unbuffered
 345:     // resources, we can stop iterating if Best looks good.
 346:     if (!SU->isScheduleHigh && Best.noCost())
 347:       break;
 348:   }
 349: 
 350:   assert (Best.SU != nullptr);
 351:   return Best.SU;
 352: }
 353: 
 354: SystemZPostRASchedStrategy::Candidate::
 355: Candidate(SUnit *SU_, SystemZHazardRecognizer &HazardRec) : Candidate() {
 356:   SU = SU_;
 357: 
 358:   // Check the grouping cost. For a node that must begin / end a
 359:   // group, it is positive if it would do so prematurely, or negative
 360:   // if it would fit naturally into the schedule.
 361:   GroupingCost = HazardRec.groupingCost(SU);
 362: 
 363:   // Check the resources cost for this SU.
 364:   ResourcesCost = HazardRec.resourcesCost(SU);
```
- **EN**: The range implements or declares functions including `Candidate`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `Candidate` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 365-392 / 第 365-392 行
```cpp
 365: }
 366: 
 367: bool SystemZPostRASchedStrategy::Candidate::
 368: operator<(const Candidate &other) {
 369: 
 370:   // Check decoder grouping.
 371:   if (GroupingCost < other.GroupingCost)
 372:     return true;
 373:   if (GroupingCost > other.GroupingCost)
 374:     return false;
 375: 
 376:   // Compare the use of resources.
 377:   if (ResourcesCost < other.ResourcesCost)
 378:     return true;
 379:   if (ResourcesCost > other.ResourcesCost)
 380:     return false;
 381: 
 382:   // Higher SU is otherwise generally better.
 383:   if (SU->getHeight() > other.SU->getHeight())
 384:     return true;
 385:   if (SU->getHeight() < other.SU->getHeight())
 386:     return false;
 387: 
 388:   // If all same, fall back to original order.
 389:   if (SU->NodeNum < other.SU->NodeNum)
 390:     return true;
 391: 
 392:   return false;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 393-414 / 第 393-414 行
```cpp
 393: }
 394: 
 395: void SystemZPostRASchedStrategy::schedNode(SUnit *SU, bool IsTopNode) {
 396:   LLVM_DEBUG(dbgs() << "** Scheduling SU(" << SU->NodeNum << ") ";
 397:              if (Available.size() == 1) dbgs() << "(only one) ";
 398:              Candidate c(SU, *HazardRec); c.dumpCosts(); dbgs() << "\n";);
 399: 
 400:   // Remove SU from Available set and update HazardRec.
 401:   Available.erase(SU);
 402:   HazardRec->EmitInstruction(SU);
 403: }
 404: 
 405: void SystemZPostRASchedStrategy::releaseTopNode(SUnit *SU) {
 406:   // Set isScheduleHigh flag on all SUs that we want to consider first in
 407:   // pickNode().
 408:   const MCSchedClassDesc *SC = HazardRec->getSchedClass(SU);
 409:   bool AffectsGrouping = (SC->isValid() && (SC->BeginGroup || SC->EndGroup));
 410:   SU->isScheduleHigh = (AffectsGrouping || SU->isUnbuffered);
 411: 
 412:   // Put all released SUs in the Available set.
 413:   Available.insert(SU);
 414: }
```
- **EN**: The range implements or declares functions including `SystemZPostRASchedStrategy::schedNode`, `SystemZPostRASchedStrategy::releaseTopNode`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZPostRASchedStrategy::schedNode`, `SystemZPostRASchedStrategy::releaseTopNode` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Scheduling model**: Provides instruction itineraries or scheduling classes for performance modelling. / 提供指令行程或调度类以支持性能建模。

## Dependencies / 依赖关系
- `SystemZMachineScheduler.h`
- `llvm/CodeGen/MachineLoopInfo.h`
