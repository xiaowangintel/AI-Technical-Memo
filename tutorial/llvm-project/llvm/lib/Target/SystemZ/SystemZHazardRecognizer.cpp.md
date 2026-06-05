# SystemZHazardRecognizer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZHazardRecognizer.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Provides target-specific implementation details for the SystemZ backend.
- **用途 (CN)**: 提供 SystemZ 后端的目标相关实现细节。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```cpp
   1: //=-- SystemZHazardRecognizer.h - SystemZ Hazard Recognizer -----*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines a hazard recognizer for the SystemZ scheduler.
  10: //
  11: // This class is used by the SystemZ scheduling strategy to maintain
  12: // the state during scheduling, and provide cost functions for
  13: // scheduling candidates. This includes:
  14: //
  15: // * Decoder grouping. A decoder group can maximally hold 3 uops, and
  16: // instructions that always begin a new group should be scheduled when
  17: // the current decoder group is empty.
  18: // * Processor resources usage. It is beneficial to balance the use of
  19: // resources.
  20: //
  21: // A goal is to consider all instructions, also those outside of any
  22: // scheduling region. Such instructions are "advanced" past and include
  23: // single instructions before a scheduling region, branches etc.
  24: //
  25: // A block that has only one predecessor continues scheduling with the state
  26: // of it (which may be updated by emitting branches).
  27: //
  28: // ===---------------------------------------------------------------------===//
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 29-56 / 第 29-56 行
```cpp
  29: 
  30: #include "SystemZHazardRecognizer.h"
  31: #include "llvm/ADT/Statistic.h"
  32: 
  33: using namespace llvm;
  34: 
  35: #define DEBUG_TYPE "machine-scheduler"
  36: 
  37: // This is the limit of processor resource usage at which the
  38: // scheduler should try to look for other instructions (not using the
  39: // critical resource).
  40: static cl::opt<int> ProcResCostLim("procres-cost-lim", cl::Hidden,
  41:                                    cl::desc("The OOO window for processor "
  42:                                             "resources during scheduling."),
  43:                                    cl::init(8));
  44: 
  45: unsigned SystemZHazardRecognizer::
  46: getNumDecoderSlots(SUnit *SU) const {
  47:   const MCSchedClassDesc *SC = getSchedClass(SU);
  48:   if (!SC->isValid())
  49:     return 0; // IMPLICIT_DEF / KILL -- will not make impact in output.
  50: 
  51:   assert((SC->NumMicroOps != 2 || (SC->BeginGroup && !SC->EndGroup)) &&
  52:          "Only cracked instruction can have 2 uops.");
  53:   assert((SC->NumMicroOps < 3 || (SC->BeginGroup && SC->EndGroup)) &&
  54:          "Expanded instructions always group alone.");
  55:   assert((SC->NumMicroOps < 3 || (SC->NumMicroOps % 3 == 0)) &&
  56:          "Expanded instructions fill the group(s).");
```
- **EN**: It imports dependencies such as `SystemZHazardRecognizer.h`, `Statistic.h` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `getNumDecoderSlots`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `SystemZHazardRecognizer.h`, `Statistic.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `getNumDecoderSlots` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 57-84 / 第 57-84 行
```cpp
  57: 
  58:   return SC->NumMicroOps;
  59: }
  60: 
  61: unsigned SystemZHazardRecognizer::getCurrCycleIdx(SUnit *SU) const {
  62:   unsigned Idx = CurrGroupSize;
  63:   if (GrpCount % 2)
  64:     Idx += 3;
  65: 
  66:   if (SU != nullptr && !fitsIntoCurrentGroup(SU)) {
  67:     if (Idx == 1 || Idx == 2)
  68:       Idx = 3;
  69:     else if (Idx == 4 || Idx == 5)
  70:       Idx = 0;
  71:   }
  72: 
  73:   return Idx;
  74: }
  75: 
  76: ScheduleHazardRecognizer::HazardType SystemZHazardRecognizer::
  77: getHazardType(SUnit *SU, int Stalls) {
  78:   return (fitsIntoCurrentGroup(SU) ? NoHazard : Hazard);
  79: }
  80: 
  81: void SystemZHazardRecognizer::Reset() {
  82:   CurrGroupSize = 0;
  83:   CurrGroupHas4RegOps = false;
  84:   clearProcResCounters();
```
- **EN**: The range implements or declares functions including `SystemZHazardRecognizer::getCurrCycleIdx`, `getHazardType`, `SystemZHazardRecognizer::Reset`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZHazardRecognizer::getCurrCycleIdx`, `getHazardType`, `SystemZHazardRecognizer::Reset` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 85-112 / 第 85-112 行
```cpp
  85:   GrpCount = 0;
  86:   LastFPdOpCycleIdx = UINT_MAX;
  87:   LastEmittedMI = nullptr;
  88:   LLVM_DEBUG(CurGroupDbg = "";);
  89: }
  90: 
  91: bool
  92: SystemZHazardRecognizer::fitsIntoCurrentGroup(SUnit *SU) const {
  93:   const MCSchedClassDesc *SC = getSchedClass(SU);
  94:   if (!SC->isValid())
  95:     return true;
  96: 
  97:   // A cracked instruction only fits into schedule if the current
  98:   // group is empty.
  99:   if (SC->BeginGroup)
 100:     return (CurrGroupSize == 0);
 101: 
 102:   // An instruction with 4 register operands will not fit in last slot.
 103:   assert ((CurrGroupSize < 2 || !CurrGroupHas4RegOps) &&
 104:           "Current decoder group is already full!");
 105:   if (CurrGroupSize == 2 && has4RegOps(SU->getInstr()))
 106:     return false;
 107: 
 108:   // Since a full group is handled immediately in EmitInstruction(),
 109:   // SU should fit into current group. NumSlots should be 1 or 0,
 110:   // since it is not a cracked or expanded instruction.
 111:   assert ((getNumDecoderSlots(SU) <= 1) && (CurrGroupSize < 3) &&
 112:           "Expected normal instruction to fit in non-full group!");
```
- **EN**: The range implements or declares functions including `SystemZHazardRecognizer::fitsIntoCurrentGroup`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZHazardRecognizer::fitsIntoCurrentGroup` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 113-140 / 第 113-140 行
```cpp
 113: 
 114:   return true;
 115: }
 116: 
 117: bool SystemZHazardRecognizer::has4RegOps(const MachineInstr *MI) const {
 118:   const MCInstrDesc &MID = MI->getDesc();
 119:   unsigned Count = 0;
 120:   for (unsigned OpIdx = 0; OpIdx < MID.getNumOperands(); OpIdx++) {
 121:     const TargetRegisterClass *RC = TII->getRegClass(MID, OpIdx);
 122:     if (RC == nullptr)
 123:       continue;
 124:     if (OpIdx >= MID.getNumDefs() &&
 125:         MID.getOperandConstraint(OpIdx, MCOI::TIED_TO) != -1)
 126:       continue;
 127:     Count++;
 128:   }
 129:   return Count >= 4;
 130: }
 131: 
 132: void SystemZHazardRecognizer::nextGroup() {
 133:   if (CurrGroupSize == 0)
 134:     return;
 135: 
 136:   LLVM_DEBUG(dumpCurrGroup("Completed decode group"));
 137:   LLVM_DEBUG(CurGroupDbg = "";);
 138: 
 139:   int NumGroups = ((CurrGroupSize > 3) ? (CurrGroupSize / 3) : 1);
 140:   assert((CurrGroupSize <= 3 || CurrGroupSize % 3 == 0) &&
```
- **EN**: The range implements or declares functions including `SystemZHazardRecognizer::has4RegOps`, `SystemZHazardRecognizer::nextGroup`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZHazardRecognizer::has4RegOps`, `SystemZHazardRecognizer::nextGroup` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 141-168 / 第 141-168 行
```cpp
 141:          "Current decoder group bad.");
 142: 
 143:   // Reset counter for next group.
 144:   CurrGroupSize = 0;
 145:   CurrGroupHas4RegOps = false;
 146: 
 147:   GrpCount += ((unsigned) NumGroups);
 148: 
 149:   // Decrease counters for execution units.
 150:   for (unsigned i = 0; i < SchedModel->getNumProcResourceKinds(); ++i)
 151:     ProcResourceCounters[i] = ((ProcResourceCounters[i] > NumGroups)
 152:                                    ? (ProcResourceCounters[i] - NumGroups)
 153:                                    : 0);
 154: 
 155:   // Clear CriticalResourceIdx if it is now below the threshold.
 156:   if (CriticalResourceIdx != UINT_MAX &&
 157:       (ProcResourceCounters[CriticalResourceIdx] <=
 158:        ProcResCostLim))
 159:     CriticalResourceIdx = UINT_MAX;
 160: 
 161:   LLVM_DEBUG(dumpState(););
 162: }
 163: 
 164: #ifndef NDEBUG // Debug output
 165: void SystemZHazardRecognizer::dumpSU(SUnit *SU, raw_ostream &OS) const {
 166:   OS << "SU(" << SU->NodeNum << "):";
 167:   OS << TII->getName(SU->getInstr()->getOpcode());
 168: 
```
- **EN**: The range implements or declares functions including `SystemZHazardRecognizer::dumpSU`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZHazardRecognizer::dumpSU` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 169-196 / 第 169-196 行
```cpp
 169:   const MCSchedClassDesc *SC = getSchedClass(SU);
 170:   if (!SC->isValid())
 171:     return;
 172: 
 173:   for (TargetSchedModel::ProcResIter
 174:          PI = SchedModel->getWriteProcResBegin(SC),
 175:          PE = SchedModel->getWriteProcResEnd(SC); PI != PE; ++PI) {
 176:     const MCProcResourceDesc &PRD =
 177:       *SchedModel->getProcResource(PI->ProcResourceIdx);
 178:     std::string FU(PRD.Name);
 179:     // trim e.g. Z13_FXaUnit -> FXa
 180:     FU = FU.substr(FU.find('_') + 1);
 181:     size_t Pos = FU.find("Unit");
 182:     if (Pos != std::string::npos)
 183:       FU.resize(Pos);
 184:     if (FU == "LS") // LSUnit -> LSU
 185:       FU = "LSU";
 186:     OS << "/" << FU;
 187: 
 188:     if (PI->ReleaseAtCycle> 1)
 189:       OS << "(" << PI->ReleaseAtCycle << "cyc)";
 190:   }
 191: 
 192:   if (SC->NumMicroOps > 1)
 193:     OS << "/" << SC->NumMicroOps << "uops";
 194:   if (SC->BeginGroup && SC->EndGroup)
 195:     OS << "/GroupsAlone";
 196:   else if (SC->BeginGroup)
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 197-224 / 第 197-224 行
```cpp
 197:     OS << "/BeginsGroup";
 198:   else if (SC->EndGroup)
 199:     OS << "/EndsGroup";
 200:   if (SU->isUnbuffered)
 201:     OS << "/Unbuffered";
 202:   if (has4RegOps(SU->getInstr()))
 203:     OS << "/4RegOps";
 204: }
 205: 
 206: void SystemZHazardRecognizer::dumpCurrGroup(std::string Msg) const {
 207:   dbgs() << "++ " << Msg;
 208:   dbgs() << ": ";
 209: 
 210:   if (CurGroupDbg.empty())
 211:     dbgs() << " <empty>\n";
 212:   else {
 213:     dbgs() << "{ " << CurGroupDbg << " }";
 214:     dbgs() << " (" << CurrGroupSize << " decoder slot"
 215:            << (CurrGroupSize > 1 ? "s":"")
 216:            << (CurrGroupHas4RegOps ? ", 4RegOps" : "")
 217:            << ")\n";
 218:   }
 219: }
 220: 
 221: void SystemZHazardRecognizer::dumpProcResourceCounters() const {
 222:   bool any = false;
 223: 
 224:   for (unsigned i = 0; i < SchedModel->getNumProcResourceKinds(); ++i)
```
- **EN**: The range implements or declares functions including `SystemZHazardRecognizer::dumpCurrGroup`, `dbgs`, `SystemZHazardRecognizer::dumpProcResourceCounters`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZHazardRecognizer::dumpCurrGroup`, `dbgs`, `SystemZHazardRecognizer::dumpProcResourceCounters` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 225-252 / 第 225-252 行
```cpp
 225:     if (ProcResourceCounters[i] > 0) {
 226:       any = true;
 227:       break;
 228:     }
 229: 
 230:   if (!any)
 231:     return;
 232: 
 233:   dbgs() << "++ | Resource counters: ";
 234:   for (unsigned i = 0; i < SchedModel->getNumProcResourceKinds(); ++i)
 235:     if (ProcResourceCounters[i] > 0)
 236:       dbgs() << SchedModel->getProcResource(i)->Name
 237:              << ":" << ProcResourceCounters[i] << " ";
 238:   dbgs() << "\n";
 239: 
 240:   if (CriticalResourceIdx != UINT_MAX)
 241:     dbgs() << "++ | Critical resource: "
 242:            << SchedModel->getProcResource(CriticalResourceIdx)->Name
 243:            << "\n";
 244: }
 245: 
 246: void SystemZHazardRecognizer::dumpState() const {
 247:   dumpCurrGroup("| Current decoder group");
 248:   dbgs() << "++ | Current cycle index: "
 249:          << getCurrCycleIdx() << "\n";
 250:   dumpProcResourceCounters();
 251:   if (LastFPdOpCycleIdx != UINT_MAX)
 252:     dbgs() << "++ | Last FPd cycle index: " << LastFPdOpCycleIdx << "\n";
```
- **EN**: The range implements or declares functions including `SystemZHazardRecognizer::dumpState`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZHazardRecognizer::dumpState` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 253-280 / 第 253-280 行
```cpp
 253: }
 254: 
 255: #endif //NDEBUG
 256: 
 257: void SystemZHazardRecognizer::clearProcResCounters() {
 258:   ProcResourceCounters.assign(SchedModel->getNumProcResourceKinds(), 0);
 259:   CriticalResourceIdx = UINT_MAX;
 260: }
 261: 
 262: static inline bool isBranchRetTrap(MachineInstr *MI) {
 263:   return (MI->isBranch() || MI->isReturn() ||
 264:           MI->getOpcode() == SystemZ::CondTrap);
 265: }
 266: 
 267: // Update state with SU as the next scheduled unit.
 268: void SystemZHazardRecognizer::
 269: EmitInstruction(SUnit *SU) {
 270:   const MCSchedClassDesc *SC = getSchedClass(SU);
 271:   LLVM_DEBUG(dbgs() << "++ HazardRecognizer emitting "; dumpSU(SU, dbgs());
 272:              dbgs() << "\n";);
 273:   LLVM_DEBUG(dumpCurrGroup("Decode group before emission"););
 274: 
 275:   // If scheduling an SU that must begin a new decoder group, move on
 276:   // to next group.
 277:   if (!fitsIntoCurrentGroup(SU))
 278:     nextGroup();
 279: 
 280:   LLVM_DEBUG(raw_string_ostream cgd(CurGroupDbg);
```
- **EN**: The range implements or declares functions including `SystemZHazardRecognizer::clearProcResCounters`, `isBranchRetTrap`, `EmitInstruction`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZHazardRecognizer::clearProcResCounters`, `isBranchRetTrap`, `EmitInstruction` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 281-308 / 第 281-308 行
```cpp
 281:              if (CurGroupDbg.length()) cgd << ", "; dumpSU(SU, cgd););
 282: 
 283:   LastEmittedMI = SU->getInstr();
 284: 
 285:   // After returning from a call, we don't know much about the state.
 286:   if (SU->isCall) {
 287:     LLVM_DEBUG(dbgs() << "++ Clearing state after call.\n";);
 288:     Reset();
 289:     LastEmittedMI = SU->getInstr();
 290:     return;
 291:   }
 292: 
 293:   // Increase counter for execution unit(s).
 294:   for (TargetSchedModel::ProcResIter
 295:          PI = SchedModel->getWriteProcResBegin(SC),
 296:          PE = SchedModel->getWriteProcResEnd(SC); PI != PE; ++PI) {
 297:     // Don't handle FPd together with the other resources.
 298:     if (SchedModel->getProcResource(PI->ProcResourceIdx)->BufferSize == 1)
 299:       continue;
 300:     int &CurrCounter =
 301:       ProcResourceCounters[PI->ProcResourceIdx];
 302:     CurrCounter += PI->ReleaseAtCycle;
 303:     // Check if this is now the new critical resource.
 304:     if ((CurrCounter > ProcResCostLim) &&
 305:         (CriticalResourceIdx == UINT_MAX ||
 306:          (PI->ProcResourceIdx != CriticalResourceIdx &&
 307:           CurrCounter >
 308:           ProcResourceCounters[CriticalResourceIdx]))) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 309-336 / 第 309-336 行
```cpp
 309:       LLVM_DEBUG(
 310:           dbgs() << "++ New critical resource: "
 311:                  << SchedModel->getProcResource(PI->ProcResourceIdx)->Name
 312:                  << "\n";);
 313:       CriticalResourceIdx = PI->ProcResourceIdx;
 314:     }
 315:   }
 316: 
 317:   // Make note of an instruction that uses a blocking resource (FPd).
 318:   if (SU->isUnbuffered) {
 319:     LastFPdOpCycleIdx = getCurrCycleIdx(SU);
 320:     LLVM_DEBUG(dbgs() << "++ Last FPd cycle index: " << LastFPdOpCycleIdx
 321:                       << "\n";);
 322:   }
 323: 
 324:   // Insert SU into current group by increasing number of slots used
 325:   // in current group.
 326:   CurrGroupSize += getNumDecoderSlots(SU);
 327:   CurrGroupHas4RegOps |= has4RegOps(SU->getInstr());
 328:   unsigned GroupLim = (CurrGroupHas4RegOps ? 2 : 3);
 329:   assert((CurrGroupSize <= GroupLim || CurrGroupSize == getNumDecoderSlots(SU))
 330:          && "SU does not fit into decoder group!");
 331: 
 332:   // Check if current group is now full/ended. If so, move on to next
 333:   // group to be ready to evaluate more candidates.
 334:   if (CurrGroupSize >= GroupLim || SC->EndGroup)
 335:     nextGroup();
 336: }
```
- **EN**: The range implements or declares functions including `assert`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `assert` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 337-364 / 第 337-364 行
```cpp
 337: 
 338: int SystemZHazardRecognizer::groupingCost(SUnit *SU) const {
 339:   const MCSchedClassDesc *SC = getSchedClass(SU);
 340:   if (!SC->isValid())
 341:     return 0;
 342: 
 343:   // If SU begins new group, it can either break a current group early
 344:   // or fit naturally if current group is empty (negative cost).
 345:   if (SC->BeginGroup) {
 346:     if (CurrGroupSize)
 347:       return 3 - CurrGroupSize;
 348:     return -1;
 349:   }
 350: 
 351:   // Similarly, a group-ending SU may either fit well (last in group), or
 352:   // end the group prematurely.
 353:   if (SC->EndGroup) {
 354:     unsigned ResultingGroupSize = (CurrGroupSize + getNumDecoderSlots(SU));
 355:     if (ResultingGroupSize < 3)
 356:       return (3 - ResultingGroupSize);
 357:     return -1;
 358:   }
 359: 
 360:   // An instruction with 4 register operands will not fit in last slot.
 361:   if (CurrGroupSize == 2 && has4RegOps(SU->getInstr()))
 362:     return 1;
 363: 
 364:   // Most instructions can be placed in any decoder slot.
```
- **EN**: The range implements or declares functions including `SystemZHazardRecognizer::groupingCost`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZHazardRecognizer::groupingCost` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 365-392 / 第 365-392 行
```cpp
 365:   return 0;
 366: }
 367: 
 368: bool SystemZHazardRecognizer::isFPdOpPreferred_distance(SUnit *SU) const {
 369:   assert (SU->isUnbuffered);
 370:   // If this is the first FPd op, it should be scheduled high.
 371:   if (LastFPdOpCycleIdx == UINT_MAX)
 372:     return true;
 373:   // If this is not the first PFd op, it should go into the other side
 374:   // of the processor to use the other FPd unit there. This should
 375:   // generally happen if two FPd ops are placed with 2 other
 376:   // instructions between them (modulo 6).
 377:   unsigned SUCycleIdx = getCurrCycleIdx(SU);
 378:   if (LastFPdOpCycleIdx > SUCycleIdx)
 379:     return ((LastFPdOpCycleIdx - SUCycleIdx) == 3);
 380:   return ((SUCycleIdx - LastFPdOpCycleIdx) == 3);
 381: }
 382: 
 383: int SystemZHazardRecognizer::
 384: resourcesCost(SUnit *SU) {
 385:   int Cost = 0;
 386: 
 387:   const MCSchedClassDesc *SC = getSchedClass(SU);
 388:   if (!SC->isValid())
 389:     return 0;
 390: 
 391:   // For a FPd op, either return min or max value as indicated by the
 392:   // distance to any prior FPd op.
```
- **EN**: The range implements or declares functions including `SystemZHazardRecognizer::isFPdOpPreferred_distance`, `resourcesCost`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZHazardRecognizer::isFPdOpPreferred_distance`, `resourcesCost` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 393-420 / 第 393-420 行
```cpp
 393:   if (SU->isUnbuffered)
 394:     Cost = (isFPdOpPreferred_distance(SU) ? INT_MIN : INT_MAX);
 395:   // For other instructions, give a cost to the use of the critical resource.
 396:   else if (CriticalResourceIdx != UINT_MAX) {
 397:     for (TargetSchedModel::ProcResIter
 398:            PI = SchedModel->getWriteProcResBegin(SC),
 399:            PE = SchedModel->getWriteProcResEnd(SC); PI != PE; ++PI)
 400:       if (PI->ProcResourceIdx == CriticalResourceIdx)
 401:         Cost = PI->ReleaseAtCycle;
 402:   }
 403: 
 404:   return Cost;
 405: }
 406: 
 407: void SystemZHazardRecognizer::emitInstruction(MachineInstr *MI,
 408:                                               bool TakenBranch) {
 409:   // Make a temporary SUnit.
 410:   SUnit SU(MI, 0);
 411: 
 412:   // Set interesting flags.
 413:   SU.isCall = MI->isCall();
 414: 
 415:   const MCSchedClassDesc *SC = SchedModel->resolveSchedClass(MI);
 416:   for (const MCWriteProcResEntry &PRE :
 417:          make_range(SchedModel->getWriteProcResBegin(SC),
 418:                     SchedModel->getWriteProcResEnd(SC))) {
 419:     switch (SchedModel->getProcResource(PRE.ProcResourceIdx)->BufferSize) {
 420:     case 0:
```
- **EN**: The range implements or declares functions including `SystemZHazardRecognizer::emitInstruction`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZHazardRecognizer::emitInstruction` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 421-448 / 第 421-448 行
```cpp
 421:       SU.hasReservedResource = true;
 422:       break;
 423:     case 1:
 424:       SU.isUnbuffered = true;
 425:       break;
 426:     default:
 427:       break;
 428:     }
 429:   }
 430: 
 431:   unsigned GroupSizeBeforeEmit = CurrGroupSize;
 432:   EmitInstruction(&SU);
 433: 
 434:   if (!TakenBranch && isBranchRetTrap(MI)) {
 435:     // NT Branch on second slot ends group.
 436:     if (GroupSizeBeforeEmit == 1)
 437:       nextGroup();
 438:   }
 439: 
 440:   if (TakenBranch && CurrGroupSize > 0)
 441:     nextGroup();
 442: 
 443:   assert ((!MI->isTerminator() || isBranchRetTrap(MI)) &&
 444:           "Scheduler: unhandled terminator!");
 445: }
 446: 
 447: void SystemZHazardRecognizer::
 448: copyState(SystemZHazardRecognizer *Incoming) {
```
- **EN**: The range implements or declares functions including `copyState`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `copyState` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 449-460 / 第 449-460 行
```cpp
 449:   // Current decoder group
 450:   CurrGroupSize = Incoming->CurrGroupSize;
 451:   LLVM_DEBUG(CurGroupDbg = Incoming->CurGroupDbg;);
 452: 
 453:   // Processor resources
 454:   ProcResourceCounters = Incoming->ProcResourceCounters;
 455:   CriticalResourceIdx = Incoming->CriticalResourceIdx;
 456: 
 457:   // FPd
 458:   LastFPdOpCycleIdx = Incoming->LastFPdOpCycleIdx;
 459:   GrpCount = Incoming->GrpCount;
 460: }
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。
- **Scheduling model**: Provides instruction itineraries or scheduling classes for performance modelling. / 提供指令行程或调度类以支持性能建模。

## Dependencies / 依赖关系
- `SystemZHazardRecognizer.h`
- `llvm/ADT/Statistic.h`
