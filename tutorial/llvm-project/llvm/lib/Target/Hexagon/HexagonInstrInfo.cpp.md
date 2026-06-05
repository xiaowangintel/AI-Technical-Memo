# HexagonInstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonInstrInfo.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): HexagonInstrInfo.cpp - Hexagon Instruction Information
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。 重点涉及指令语义与选择。

## Line-by-Line Analysis / 逐行分析

### Lines 1-200 / 第 1-200 行

```cpp
     1: //===- HexagonInstrInfo.cpp - Hexagon Instruction Information -------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This file contains the Hexagon implementation of the TargetInstrInfo class.
    10: //
    11: //===----------------------------------------------------------------------===//
    12: 
    13: #include "HexagonInstrInfo.h"
    14: #include "HexagonFrameLowering.h"
    15: #include "HexagonHazardRecognizer.h"
    16: #include "HexagonRegisterInfo.h"
    17: #include "HexagonSubtarget.h"
    18: #include "llvm/ADT/ArrayRef.h"
    19: #include "llvm/ADT/SmallPtrSet.h"
    20: #include "llvm/ADT/SmallVector.h"
    21: #include "llvm/ADT/StringExtras.h"
    22: #include "llvm/ADT/StringRef.h"
    23: #include "llvm/CodeGen/DFAPacketizer.h"
    24: #include "llvm/CodeGen/LiveIntervals.h"
    25: #include "llvm/CodeGen/LivePhysRegs.h"
    26: #include "llvm/CodeGen/MachineBasicBlock.h"
    27: #include "llvm/CodeGen/MachineBranchProbabilityInfo.h"
    28: #include "llvm/CodeGen/MachineFrameInfo.h"
    29: #include "llvm/CodeGen/MachineFunction.h"
    30: #include "llvm/CodeGen/MachineInstr.h"
    31: #include "llvm/CodeGen/MachineInstrBuilder.h"
    32: #include "llvm/CodeGen/MachineInstrBundle.h"
    33: #include "llvm/CodeGen/MachineMemOperand.h"
    34: #include "llvm/CodeGen/MachineOperand.h"
    35: #include "llvm/CodeGen/MachineRegisterInfo.h"
    36: #include "llvm/CodeGen/ScheduleDAG.h"
    37: #include "llvm/CodeGen/TargetInstrInfo.h"
    38: #include "llvm/CodeGen/TargetOpcodes.h"
    39: #include "llvm/CodeGen/TargetRegisterInfo.h"
    40: #include "llvm/CodeGen/TargetSubtargetInfo.h"
    41: #include "llvm/CodeGenTypes/MachineValueType.h"
    42: #include "llvm/IR/DebugLoc.h"
    43: #include "llvm/IR/GlobalVariable.h"
    44: #include "llvm/MC/MCAsmInfo.h"
    45: #include "llvm/MC/MCInstBuilder.h"
    46: #include "llvm/MC/MCInstrDesc.h"
    47: #include "llvm/MC/MCInstrItineraries.h"
    48: #include "llvm/Support/BranchProbability.h"
    49: #include "llvm/Support/CommandLine.h"
    50: #include "llvm/Support/Debug.h"
    51: #include "llvm/Support/ErrorHandling.h"
    52: #include "llvm/Support/MathExtras.h"
    53: #include "llvm/Support/raw_ostream.h"
    54: #include "llvm/Target/TargetMachine.h"
    55: #include <cassert>
    56: #include <cctype>
    57: #include <cstdint>
    58: #include <cstring>
    59: #include <iterator>
    60: #include <optional>
    61: #include <string>
    62: #include <utility>
    63: 
    64: using namespace llvm;
    65: 
    66: #define DEBUG_TYPE "hexagon-instrinfo"
    67: 
    68: #define GET_INSTRINFO_CTOR_DTOR
    69: #define GET_INSTRMAP_INFO
    70: #include "HexagonDepTimingClasses.h"
    71: #include "HexagonGenDFAPacketizer.inc"
    72: #include "HexagonGenInstrInfo.inc"
    73: 
    74: cl::opt<bool> ScheduleInlineAsm("hexagon-sched-inline-asm", cl::Hidden,
    75:   cl::init(false), cl::desc("Do not consider inline-asm a scheduling/"
    76:                             "packetization boundary."));
    77: 
    78: static cl::opt<bool> EnableBranchPrediction("hexagon-enable-branch-prediction",
    79:   cl::Hidden, cl::init(true), cl::desc("Enable branch prediction"));
    80: 
    81: static cl::opt<bool> DisableNVSchedule(
    82:     "disable-hexagon-nv-schedule", cl::Hidden,
    83:     cl::desc("Disable schedule adjustment for new value stores."));
    84: 
    85: static cl::opt<bool> EnableTimingClassLatency(
    86:   "enable-timing-class-latency", cl::Hidden, cl::init(false),
    87:   cl::desc("Enable timing class latency"));
    88: 
    89: static cl::opt<bool> EnableALUForwarding(
    90:   "enable-alu-forwarding", cl::Hidden, cl::init(true),
    91:   cl::desc("Enable vec alu forwarding"));
    92: 
    93: static cl::opt<bool> EnableACCForwarding(
    94:   "enable-acc-forwarding", cl::Hidden, cl::init(true),
    95:   cl::desc("Enable vec acc forwarding"));
    96: 
    97: static cl::opt<bool> BranchRelaxAsmLarge("branch-relax-asm-large",
    98:                                          cl::init(true), cl::Hidden,
    99:                                          cl::desc("branch relax asm"));
   100: 
   101: static cl::opt<bool>
   102:     UseDFAHazardRec("dfa-hazard-rec", cl::init(true), cl::Hidden,
   103:                     cl::desc("Use the DFA based hazard recognizer."));
   104: 
   105: /// Constants for Hexagon instructions.
   106: const int Hexagon_MEMW_OFFSET_MAX = 4095;
   107: const int Hexagon_MEMW_OFFSET_MIN = -4096;
   108: const int Hexagon_MEMD_OFFSET_MAX = 8191;
   109: const int Hexagon_MEMD_OFFSET_MIN = -8192;
   110: const int Hexagon_MEMH_OFFSET_MAX = 2047;
   111: const int Hexagon_MEMH_OFFSET_MIN = -2048;
   112: const int Hexagon_MEMB_OFFSET_MAX = 1023;
   113: const int Hexagon_MEMB_OFFSET_MIN = -1024;
   114: const int Hexagon_ADDI_OFFSET_MAX = 32767;
   115: const int Hexagon_ADDI_OFFSET_MIN = -32768;
   116: 
   117: // Pin the vtable to this file.
   118: void HexagonInstrInfo::anchor() {}
   119: 
   120: HexagonInstrInfo::HexagonInstrInfo(const HexagonSubtarget &ST)
   121:     : HexagonGenInstrInfo(ST, RegInfo, Hexagon::ADJCALLSTACKDOWN,
   122:                           Hexagon::ADJCALLSTACKUP),
   123:       RegInfo(ST.getHwMode()), Subtarget(ST) {}
   124: 
   125: namespace llvm {
   126: namespace HexagonFUnits {
   127:   bool isSlot0Only(unsigned units);
   128: }
   129: }
   130: 
   131: static bool isIntRegForSubInst(Register Reg) {
   132:   return (Reg >= Hexagon::R0 && Reg <= Hexagon::R7) ||
   133:          (Reg >= Hexagon::R16 && Reg <= Hexagon::R23);
   134: }
   135: 
   136: static bool isDblRegForSubInst(Register Reg, const HexagonRegisterInfo &HRI) {
   137:   return isIntRegForSubInst(HRI.getSubReg(Reg, Hexagon::isub_lo)) &&
   138:          isIntRegForSubInst(HRI.getSubReg(Reg, Hexagon::isub_hi));
   139: }
   140: 
   141: /// Calculate number of instructions excluding the debug instructions.
   142: static unsigned nonDbgMICount(MachineBasicBlock::const_instr_iterator MIB,
   143:                               MachineBasicBlock::const_instr_iterator MIE) {
   144:   unsigned Count = 0;
   145:   for (; MIB != MIE; ++MIB) {
   146:     if (!MIB->isDebugInstr())
   147:       ++Count;
   148:   }
   149:   return Count;
   150: }
   151: 
   152: // Check if the A2_tfrsi instruction is cheap or not. If the operand has
   153: // to be constant-extendend it is not cheap since it occupies two slots
   154: // in a packet.
   155: bool HexagonInstrInfo::isAsCheapAsAMove(const MachineInstr &MI) const {
   156:   // Enable the following steps only at Os/Oz
   157:   if (!(MI.getMF()->getFunction().hasOptSize()))
   158:     return MI.isAsCheapAsAMove();
   159: 
   160:   if (MI.getOpcode() == Hexagon::A2_tfrsi) {
   161:     auto Op = MI.getOperand(1);
   162:     // If the instruction has a global address as operand, it is not cheap
   163:     // since the operand will be constant extended.
   164:     if (Op.isGlobal())
   165:       return false;
   166:     // If the instruction has an operand of size > 16bits, its will be
   167:     // const-extended and hence, it is not cheap.
   168:     if (Op.isImm()) {
   169:       int64_t Imm = Op.getImm();
   170:       if (!isInt<16>(Imm))
   171:         return false;
   172:     }
   173:   }
   174:   return MI.isAsCheapAsAMove();
   175: }
   176: 
   177: // Do not sink floating point instructions that updates USR register.
   178: // Example:
   179: //    feclearexcept
   180: //    F2_conv_w2sf
   181: //    fetestexcept
   182: // MachineSink sinks F2_conv_w2sf and we are not able to catch exceptions.
   183: // TODO: On some of these floating point instructions, USR is marked as Use.
   184: // In reality, these instructions also Def the USR. If USR is marked as Def,
   185: // some of the assumptions in assembler packetization are broken.
   186: bool HexagonInstrInfo::shouldSink(const MachineInstr &MI) const {
   187:   // Assumption: A floating point instruction that reads the USR will write
   188:   // the USR as well.
   189:   if (isFloat(MI) && MI.hasRegisterImplicitUseOperand(Hexagon::USR))
   190:     return false;
   191:   return true;
   192: }
   193: 
   194: /// Find the hardware loop instruction used to set-up the specified loop.
   195: /// On Hexagon, we have two instructions used to set-up the hardware loop
   196: /// (LOOP0, LOOP1) with corresponding endloop (ENDLOOP0, ENDLOOP1) instructions
   197: /// to indicate the end of a loop.
   198: MachineInstr *HexagonInstrInfo::findLoopInstr(MachineBasicBlock *BB,
   199:       unsigned EndLoopOp, MachineBasicBlock *TargetBB,
   200:       SmallPtrSet<MachineBasicBlock *, 8> &Visited) const {
```
- EN: It imports headers such as HexagonInstrInfo.h, HexagonFrameLowering.h, HexagonHazardRecognizer.h, HexagonRegisterInfo.h, ... (53 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm, HexagonFUnits) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as ScheduleInlineAsm, EnableBranchPrediction, DisableNVSchedule, EnableTimingClassLatency, ... (20 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 HexagonInstrInfo.h, HexagonFrameLowering.h, HexagonHazardRecognizer.h, HexagonRegisterInfo.h, ... (53 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm, HexagonFUnits），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 ScheduleInlineAsm, EnableBranchPrediction, DisableNVSchedule, EnableTimingClassLatency, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 201-400 / 第 201-400 行

```cpp
   201:   unsigned LOOPi;
   202:   unsigned LOOPr;
   203:   if (EndLoopOp == Hexagon::ENDLOOP0) {
   204:     LOOPi = Hexagon::J2_loop0i;
   205:     LOOPr = Hexagon::J2_loop0r;
   206:   } else { // EndLoopOp == Hexagon::EndLOOP1
   207:     LOOPi = Hexagon::J2_loop1i;
   208:     LOOPr = Hexagon::J2_loop1r;
   209:   }
   210: 
   211:   // The loop set-up instruction will be in a predecessor block
   212:   for (MachineBasicBlock *PB : BB->predecessors()) {
   213:     // If this has been visited, already skip it.
   214:     if (!Visited.insert(PB).second)
   215:       continue;
   216:     if (PB == BB)
   217:       continue;
   218:     for (MachineInstr &I : llvm::reverse(PB->instrs())) {
   219:       unsigned Opc = I.getOpcode();
   220:       if (Opc == LOOPi || Opc == LOOPr)
   221:         return &I;
   222:       // We've reached a different loop, which means the loop01 has been
   223:       // removed.
   224:       if (Opc == EndLoopOp && I.getOperand(0).getMBB() != TargetBB)
   225:         return nullptr;
   226:     }
   227:     // Check the predecessors for the LOOP instruction.
   228:     if (MachineInstr *Loop = findLoopInstr(PB, EndLoopOp, TargetBB, Visited))
   229:       return Loop;
   230:   }
   231:   return nullptr;
   232: }
   233: 
   234: /// Gather register def/uses from MI.
   235: /// This treats possible (predicated) defs as actually happening ones
   236: /// (conservatively).
   237: static inline void parseOperands(const MachineInstr &MI,
   238:       SmallVectorImpl<Register> &Defs, SmallVectorImpl<Register> &Uses) {
   239:   Defs.clear();
   240:   Uses.clear();
   241: 
   242:   for (const MachineOperand &MO : MI.operands()) {
   243:     if (!MO.isReg())
   244:       continue;
   245: 
   246:     Register Reg = MO.getReg();
   247:     if (!Reg)
   248:       continue;
   249: 
   250:     if (MO.isUse())
   251:       Uses.push_back(MO.getReg());
   252: 
   253:     if (MO.isDef())
   254:       Defs.push_back(MO.getReg());
   255:   }
   256: }
   257: 
   258: // Position dependent, so check twice for swap.
   259: static bool isDuplexPairMatch(unsigned Ga, unsigned Gb) {
   260:   switch (Ga) {
   261:   case HexagonII::HSIG_None:
   262:   default:
   263:     return false;
   264:   case HexagonII::HSIG_L1:
   265:     return (Gb == HexagonII::HSIG_L1 || Gb == HexagonII::HSIG_A);
   266:   case HexagonII::HSIG_L2:
   267:     return (Gb == HexagonII::HSIG_L1 || Gb == HexagonII::HSIG_L2 ||
   268:             Gb == HexagonII::HSIG_A);
   269:   case HexagonII::HSIG_S1:
   270:     return (Gb == HexagonII::HSIG_L1 || Gb == HexagonII::HSIG_L2 ||
   271:             Gb == HexagonII::HSIG_S1 || Gb == HexagonII::HSIG_A);
   272:   case HexagonII::HSIG_S2:
   273:     return (Gb == HexagonII::HSIG_L1 || Gb == HexagonII::HSIG_L2 ||
   274:             Gb == HexagonII::HSIG_S1 || Gb == HexagonII::HSIG_S2 ||
   275:             Gb == HexagonII::HSIG_A);
   276:   case HexagonII::HSIG_A:
   277:     return (Gb == HexagonII::HSIG_A);
   278:   case HexagonII::HSIG_Compound:
   279:     return (Gb == HexagonII::HSIG_Compound);
   280:   }
   281:   return false;
   282: }
   283: 
   284: /// isLoadFromStackSlot - If the specified machine instruction is a direct
   285: /// load from a stack slot, return the virtual or physical register number of
   286: /// the destination along with the FrameIndex of the loaded stack slot.  If
   287: /// not, return 0.  This predicate must return 0 if the instruction has
   288: /// any side effects other than loading from the stack slot.
   289: Register HexagonInstrInfo::isLoadFromStackSlot(const MachineInstr &MI,
   290:                                                int &FrameIndex) const {
   291:   switch (MI.getOpcode()) {
   292:     default:
   293:       break;
   294:     case Hexagon::L2_loadri_io:
   295:     case Hexagon::L2_loadrd_io:
   296:     case Hexagon::V6_vL32b_ai:
   297:     case Hexagon::V6_vL32b_nt_ai:
   298:     case Hexagon::V6_vL32Ub_ai:
   299:     case Hexagon::LDriw_pred:
   300:     case Hexagon::LDriw_ctr:
   301:     case Hexagon::PS_vloadrq_ai:
   302:     case Hexagon::PS_vloadrw_ai:
   303:     case Hexagon::PS_vloadrw_nt_ai: {
   304:       const MachineOperand OpFI = MI.getOperand(1);
   305:       if (!OpFI.isFI())
   306:         return 0;
   307:       const MachineOperand OpOff = MI.getOperand(2);
   308:       if (!OpOff.isImm() || OpOff.getImm() != 0)
   309:         return 0;
   310:       FrameIndex = OpFI.getIndex();
   311:       return MI.getOperand(0).getReg();
   312:     }
   313: 
   314:     case Hexagon::L2_ploadrit_io:
   315:     case Hexagon::L2_ploadrif_io:
   316:     case Hexagon::L2_ploadrdt_io:
   317:     case Hexagon::L2_ploadrdf_io: {
   318:       const MachineOperand OpFI = MI.getOperand(2);
   319:       if (!OpFI.isFI())
   320:         return 0;
   321:       const MachineOperand OpOff = MI.getOperand(3);
   322:       if (!OpOff.isImm() || OpOff.getImm() != 0)
   323:         return 0;
   324:       FrameIndex = OpFI.getIndex();
   325:       return MI.getOperand(0).getReg();
   326:     }
   327:   }
   328: 
   329:   return 0;
   330: }
   331: 
   332: /// isStoreToStackSlot - If the specified machine instruction is a direct
   333: /// store to a stack slot, return the virtual or physical register number of
   334: /// the source reg along with the FrameIndex of the loaded stack slot.  If
   335: /// not, return 0.  This predicate must return 0 if the instruction has
   336: /// any side effects other than storing to the stack slot.
   337: Register HexagonInstrInfo::isStoreToStackSlot(const MachineInstr &MI,
   338:                                               int &FrameIndex) const {
   339:   switch (MI.getOpcode()) {
   340:     default:
   341:       break;
   342:     case Hexagon::S2_storerb_io:
   343:     case Hexagon::S2_storerh_io:
   344:     case Hexagon::S2_storeri_io:
   345:     case Hexagon::S2_storerd_io:
   346:     case Hexagon::V6_vS32b_ai:
   347:     case Hexagon::V6_vS32Ub_ai:
   348:     case Hexagon::STriw_pred:
   349:     case Hexagon::STriw_ctr:
   350:     case Hexagon::PS_vstorerq_ai:
   351:     case Hexagon::PS_vstorerw_ai: {
   352:       const MachineOperand &OpFI = MI.getOperand(0);
   353:       if (!OpFI.isFI())
   354:         return 0;
   355:       const MachineOperand &OpOff = MI.getOperand(1);
   356:       if (!OpOff.isImm() || OpOff.getImm() != 0)
   357:         return 0;
   358:       FrameIndex = OpFI.getIndex();
   359:       return MI.getOperand(2).getReg();
   360:     }
   361: 
   362:     case Hexagon::S2_pstorerbt_io:
   363:     case Hexagon::S2_pstorerbf_io:
   364:     case Hexagon::S2_pstorerht_io:
   365:     case Hexagon::S2_pstorerhf_io:
   366:     case Hexagon::S2_pstorerit_io:
   367:     case Hexagon::S2_pstorerif_io:
   368:     case Hexagon::S2_pstorerdt_io:
   369:     case Hexagon::S2_pstorerdf_io: {
   370:       const MachineOperand &OpFI = MI.getOperand(1);
   371:       if (!OpFI.isFI())
   372:         return 0;
   373:       const MachineOperand &OpOff = MI.getOperand(2);
   374:       if (!OpOff.isImm() || OpOff.getImm() != 0)
   375:         return 0;
   376:       FrameIndex = OpFI.getIndex();
   377:       return MI.getOperand(3).getReg();
   378:     }
   379:   }
   380: 
   381:   return 0;
   382: }
   383: 
   384: /// This function checks if the instruction or bundle of instructions
   385: /// has load from stack slot and returns frameindex and machine memory
   386: /// operand of that instruction if true.
   387: bool HexagonInstrInfo::hasLoadFromStackSlot(
   388:     const MachineInstr &MI,
   389:     SmallVectorImpl<const MachineMemOperand *> &Accesses) const {
   390:   if (MI.isBundle()) {
   391:     const MachineBasicBlock *MBB = MI.getParent();
   392:     MachineBasicBlock::const_instr_iterator MII = MI.getIterator();
   393:     for (++MII; MII != MBB->instr_end() && MII->isInsideBundle(); ++MII)
   394:       if (TargetInstrInfo::hasLoadFromStackSlot(*MII, Accesses))
   395:         return true;
   396:     return false;
   397:   }
   398: 
   399:   return TargetInstrInfo::hasLoadFromStackSlot(MI, Accesses);
   400: }
```
- EN: It declares or implements routines such as getOpcode, possible, clear, getReg, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getOpcode, possible, clear, getReg, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII, HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 401-600 / 第 401-600 行

```cpp
   401: 
   402: /// This function checks if the instruction or bundle of instructions
   403: /// has store to stack slot and returns frameindex and machine memory
   404: /// operand of that instruction if true.
   405: bool HexagonInstrInfo::hasStoreToStackSlot(
   406:     const MachineInstr &MI,
   407:     SmallVectorImpl<const MachineMemOperand *> &Accesses) const {
   408:   if (MI.isBundle()) {
   409:     const MachineBasicBlock *MBB = MI.getParent();
   410:     MachineBasicBlock::const_instr_iterator MII = MI.getIterator();
   411:     for (++MII; MII != MBB->instr_end() && MII->isInsideBundle(); ++MII)
   412:       if (TargetInstrInfo::hasStoreToStackSlot(*MII, Accesses))
   413:         return true;
   414:     return false;
   415:   }
   416: 
   417:   return TargetInstrInfo::hasStoreToStackSlot(MI, Accesses);
   418: }
   419: 
   420: /// This function can analyze one/two way branching only and should (mostly) be
   421: /// called by target independent side.
   422: /// First entry is always the opcode of the branching instruction, except when
   423: /// the Cond vector is supposed to be empty, e.g., when analyzeBranch fails, a
   424: /// BB with only unconditional jump. Subsequent entries depend upon the opcode,
   425: /// e.g. Jump_c p will have
   426: /// Cond[0] = Jump_c
   427: /// Cond[1] = p
   428: /// HW-loop ENDLOOP:
   429: /// Cond[0] = ENDLOOP
   430: /// Cond[1] = MBB
   431: /// New value jump:
   432: /// Cond[0] = Hexagon::CMPEQri_f_Jumpnv_t_V4 -- specific opcode
   433: /// Cond[1] = R
   434: /// Cond[2] = Imm
   435: bool HexagonInstrInfo::analyzeBranch(MachineBasicBlock &MBB,
   436:                                      MachineBasicBlock *&TBB,
   437:                                      MachineBasicBlock *&FBB,
   438:                                      SmallVectorImpl<MachineOperand> &Cond,
   439:                                      bool AllowModify) const {
   440:   TBB = nullptr;
   441:   FBB = nullptr;
   442:   Cond.clear();
   443: 
   444:   // If the block has no terminators, it just falls into the block after it.
   445:   MachineBasicBlock::instr_iterator I = MBB.instr_end();
   446:   if (I == MBB.instr_begin())
   447:     return false;
   448: 
   449:   // A basic block may looks like this:
   450:   //
   451:   //  [   insn
   452:   //     EH_LABEL
   453:   //      insn
   454:   //      insn
   455:   //      insn
   456:   //     EH_LABEL
   457:   //      insn     ]
   458:   //
   459:   // It has two succs but does not have a terminator
   460:   // Don't know how to handle it.
   461:   do {
   462:     --I;
   463:     if (I->isEHLabel())
   464:       // Don't analyze EH branches.
   465:       return true;
   466:   } while (I != MBB.instr_begin());
   467: 
   468:   I = MBB.instr_end();
   469:   --I;
   470: 
   471:   while (I->isDebugInstr()) {
   472:     if (I == MBB.instr_begin())
   473:       return false;
   474:     --I;
   475:   }
   476: 
   477:   bool JumpToBlock = I->getOpcode() == Hexagon::J2_jump &&
   478:                      I->getOperand(0).isMBB();
   479:   // Delete the J2_jump if it's equivalent to a fall-through.
   480:   if (AllowModify && JumpToBlock &&
   481:       MBB.isLayoutSuccessor(I->getOperand(0).getMBB())) {
   482:     LLVM_DEBUG(dbgs() << "\nErasing the jump to successor block\n";);
   483:     I->eraseFromParent();
   484:     I = MBB.instr_end();
   485:     if (I == MBB.instr_begin())
   486:       return false;
   487:     --I;
   488:   }
   489:   if (!isUnpredicatedTerminator(*I))
   490:     return false;
   491: 
   492:   // Get the last instruction in the block.
   493:   MachineInstr *LastInst = &*I;
   494:   MachineInstr *SecondLastInst = nullptr;
   495:   // Find one more terminator if present.
   496:   while (true) {
   497:     if (&*I != LastInst && !I->isBundle() && isUnpredicatedTerminator(*I)) {
   498:       if (!SecondLastInst)
   499:         SecondLastInst = &*I;
   500:       else
   501:         // This is a third branch.
   502:         return true;
   503:     }
   504:     if (I == MBB.instr_begin())
   505:       break;
   506:     --I;
   507:   }
   508: 
   509:   int LastOpcode = LastInst->getOpcode();
   510:   int SecLastOpcode = SecondLastInst ? SecondLastInst->getOpcode() : 0;
   511:   // If the branch target is not a basic block, it could be a tail call.
   512:   // (It is, if the target is a function.)
   513:   if (LastOpcode == Hexagon::J2_jump && !LastInst->getOperand(0).isMBB())
   514:     return true;
   515:   if (SecLastOpcode == Hexagon::J2_jump &&
   516:       !SecondLastInst->getOperand(0).isMBB())
   517:     return true;
   518: 
   519:   bool LastOpcodeHasJMP_c = PredOpcodeHasJMP_c(LastOpcode);
   520:   bool LastOpcodeHasNVJump = isNewValueJump(*LastInst);
   521: 
   522:   if (LastOpcodeHasJMP_c && !LastInst->getOperand(1).isMBB())
   523:     return true;
   524: 
   525:   // If there is only one terminator instruction, process it.
   526:   if (LastInst && !SecondLastInst) {
   527:     if (LastOpcode == Hexagon::J2_jump) {
   528:       TBB = LastInst->getOperand(0).getMBB();
   529:       return false;
   530:     }
   531:     if (isEndLoopN(LastOpcode)) {
   532:       TBB = LastInst->getOperand(0).getMBB();
   533:       Cond.push_back(MachineOperand::CreateImm(LastInst->getOpcode()));
   534:       Cond.push_back(LastInst->getOperand(0));
   535:       return false;
   536:     }
   537:     if (LastOpcodeHasJMP_c) {
   538:       TBB = LastInst->getOperand(1).getMBB();
   539:       Cond.push_back(MachineOperand::CreateImm(LastInst->getOpcode()));
   540:       Cond.push_back(LastInst->getOperand(0));
   541:       return false;
   542:     }
   543:     // Only supporting rr/ri versions of new-value jumps.
   544:     if (LastOpcodeHasNVJump && (LastInst->getNumExplicitOperands() == 3)) {
   545:       TBB = LastInst->getOperand(2).getMBB();
   546:       Cond.push_back(MachineOperand::CreateImm(LastInst->getOpcode()));
   547:       Cond.push_back(LastInst->getOperand(0));
   548:       Cond.push_back(LastInst->getOperand(1));
   549:       return false;
   550:     }
   551:     LLVM_DEBUG(dbgs() << "\nCant analyze " << printMBBReference(MBB)
   552:                       << " with one jump\n";);
   553:     // Otherwise, don't know what this is.
   554:     return true;
   555:   }
   556: 
   557:   bool SecLastOpcodeHasJMP_c = PredOpcodeHasJMP_c(SecLastOpcode);
   558:   bool SecLastOpcodeHasNVJump = isNewValueJump(*SecondLastInst);
   559:   if (SecLastOpcodeHasJMP_c && (LastOpcode == Hexagon::J2_jump)) {
   560:     if (!SecondLastInst->getOperand(1).isMBB())
   561:       return true;
   562:     TBB =  SecondLastInst->getOperand(1).getMBB();
   563:     Cond.push_back(MachineOperand::CreateImm(SecondLastInst->getOpcode()));
   564:     Cond.push_back(SecondLastInst->getOperand(0));
   565:     FBB = LastInst->getOperand(0).getMBB();
   566:     return false;
   567:   }
   568: 
   569:   // Only supporting rr/ri versions of new-value jumps.
   570:   if (SecLastOpcodeHasNVJump &&
   571:       (SecondLastInst->getNumExplicitOperands() == 3) &&
   572:       (LastOpcode == Hexagon::J2_jump)) {
   573:     TBB = SecondLastInst->getOperand(2).getMBB();
   574:     Cond.push_back(MachineOperand::CreateImm(SecondLastInst->getOpcode()));
   575:     Cond.push_back(SecondLastInst->getOperand(0));
   576:     Cond.push_back(SecondLastInst->getOperand(1));
   577:     FBB = LastInst->getOperand(0).getMBB();
   578:     return false;
   579:   }
   580: 
   581:   // If the block ends with two Hexagon:JMPs, handle it.  The second one is not
   582:   // executed, so remove it.
   583:   if (SecLastOpcode == Hexagon::J2_jump && LastOpcode == Hexagon::J2_jump) {
   584:     TBB = SecondLastInst->getOperand(0).getMBB();
   585:     I = LastInst->getIterator();
   586:     if (AllowModify)
   587:       I->eraseFromParent();
   588:     return false;
   589:   }
   590: 
   591:   // If the block ends with an ENDLOOP, and J2_jump, handle it.
   592:   if (isEndLoopN(SecLastOpcode) && LastOpcode == Hexagon::J2_jump) {
   593:     TBB = SecondLastInst->getOperand(0).getMBB();
   594:     Cond.push_back(MachineOperand::CreateImm(SecondLastInst->getOpcode()));
   595:     Cond.push_back(SecondLastInst->getOperand(0));
   596:     FBB = LastInst->getOperand(0).getMBB();
   597:     return false;
   598:   }
   599:   LLVM_DEBUG(dbgs() << "\nCant analyze " << printMBBReference(MBB)
   600:                     << " with two jumps";);
```
- EN: It declares or implements routines such as HexagonInstrInfo::hasStoreToStackSlot, getParent, getIterator, instr_end, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonInstrInfo::hasStoreToStackSlot, getParent, getIterator, instr_end, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 601-800 / 第 601-800 行

```cpp
   601:   // Otherwise, can't handle this.
   602:   return true;
   603: }
   604: 
   605: unsigned HexagonInstrInfo::removeBranch(MachineBasicBlock &MBB,
   606:                                         int *BytesRemoved) const {
   607:   assert(!BytesRemoved && "code size not handled");
   608: 
   609:   LLVM_DEBUG(dbgs() << "\nRemoving branches out of " << printMBBReference(MBB));
   610:   MachineBasicBlock::iterator I = MBB.end();
   611:   unsigned Count = 0;
   612:   while (I != MBB.begin()) {
   613:     --I;
   614:     if (I->isDebugInstr())
   615:       continue;
   616:     // Only removing branches from end of MBB.
   617:     if (!I->isBranch())
   618:       return Count;
   619:     if (Count && (I->getOpcode() == Hexagon::J2_jump))
   620:       llvm_unreachable("Malformed basic block: unconditional branch not last");
   621:     MBB.erase(&MBB.back());
   622:     I = MBB.end();
   623:     ++Count;
   624:   }
   625:   return Count;
   626: }
   627: 
   628: unsigned HexagonInstrInfo::insertBranch(MachineBasicBlock &MBB,
   629:                                         MachineBasicBlock *TBB,
   630:                                         MachineBasicBlock *FBB,
   631:                                         ArrayRef<MachineOperand> Cond,
   632:                                         const DebugLoc &DL,
   633:                                         int *BytesAdded) const {
   634:   unsigned BOpc   = Hexagon::J2_jump;
   635:   unsigned BccOpc = Hexagon::J2_jumpt;
   636:   assert(validateBranchCond(Cond) && "Invalid branching condition");
   637:   assert(TBB && "insertBranch must not be told to insert a fallthrough");
   638:   assert(!BytesAdded && "code size not handled");
   639: 
   640:   // Check if reverseBranchCondition has asked to reverse this branch
   641:   // If we want to reverse the branch an odd number of times, we want
   642:   // J2_jumpf.
   643:   if (!Cond.empty() && Cond[0].isImm())
   644:     BccOpc = Cond[0].getImm();
   645: 
   646:   if (!FBB) {
   647:     if (Cond.empty()) {
   648:       // Due to a bug in TailMerging/CFG Optimization, we need to add a
   649:       // special case handling of a predicated jump followed by an
   650:       // unconditional jump. If not, Tail Merging and CFG Optimization go
   651:       // into an infinite loop.
   652:       MachineBasicBlock *NewTBB, *NewFBB;
   653:       SmallVector<MachineOperand, 4> Cond;
   654:       auto Term = MBB.getFirstTerminator();
   655:       if (Term != MBB.end() && isPredicated(*Term) &&
   656:           !analyzeBranch(MBB, NewTBB, NewFBB, Cond, false) &&
   657:           MachineFunction::iterator(NewTBB) == ++MBB.getIterator()) {
   658:         reverseBranchCondition(Cond);
   659:         removeBranch(MBB);
   660:         return insertBranch(MBB, TBB, nullptr, Cond, DL);
   661:       }
   662:       BuildMI(&MBB, DL, get(BOpc)).addMBB(TBB);
   663:     } else if (isEndLoopN(Cond[0].getImm())) {
   664:       int EndLoopOp = Cond[0].getImm();
   665:       assert(Cond[1].isMBB());
   666:       // Since we're adding an ENDLOOP, there better be a LOOP instruction.
   667:       // Check for it, and change the BB target if needed.
   668:       SmallPtrSet<MachineBasicBlock *, 8> VisitedBBs;
   669:       MachineInstr *Loop = findLoopInstr(TBB, EndLoopOp, Cond[1].getMBB(),
   670:                                          VisitedBBs);
   671:       assert(Loop != nullptr && "Inserting an ENDLOOP without a LOOP");
   672:       Loop->getOperand(0).setMBB(TBB);
   673:       // Add the ENDLOOP after the finding the LOOP0.
   674:       BuildMI(&MBB, DL, get(EndLoopOp)).addMBB(TBB);
   675:     } else if (isNewValueJump(Cond[0].getImm())) {
   676:       assert((Cond.size() == 3) && "Only supporting rr/ri version of nvjump");
   677:       // New value jump
   678:       // (ins IntRegs:$src1, IntRegs:$src2, brtarget:$offset)
   679:       // (ins IntRegs:$src1, u5Imm:$src2, brtarget:$offset)
   680:       RegState Flags1 = getUndefRegState(Cond[1].isUndef());
   681:       LLVM_DEBUG(dbgs() << "\nInserting NVJump for "
   682:                         << printMBBReference(MBB););
   683:       if (Cond[2].isReg()) {
   684:         RegState Flags2 = getUndefRegState(Cond[2].isUndef());
   685:         BuildMI(&MBB, DL, get(BccOpc)).addReg(Cond[1].getReg(), Flags1).
   686:           addReg(Cond[2].getReg(), Flags2).addMBB(TBB);
   687:       } else if(Cond[2].isImm()) {
   688:         BuildMI(&MBB, DL, get(BccOpc)).addReg(Cond[1].getReg(), Flags1).
   689:           addImm(Cond[2].getImm()).addMBB(TBB);
   690:       } else
   691:         llvm_unreachable("Invalid condition for branching");
   692:     } else {
   693:       assert((Cond.size() == 2) && "Malformed cond vector");
   694:       const MachineOperand &RO = Cond[1];
   695:       RegState Flags = getUndefRegState(RO.isUndef());
   696:       BuildMI(&MBB, DL, get(BccOpc)).addReg(RO.getReg(), Flags).addMBB(TBB);
   697:     }
   698:     return 1;
   699:   }
   700:   assert((!Cond.empty()) &&
   701:          "Cond. cannot be empty when multiple branchings are required");
   702:   assert((!isNewValueJump(Cond[0].getImm())) &&
   703:          "NV-jump cannot be inserted with another branch");
   704:   // Special case for hardware loops.  The condition is a basic block.
   705:   if (isEndLoopN(Cond[0].getImm())) {
   706:     int EndLoopOp = Cond[0].getImm();
   707:     assert(Cond[1].isMBB());
   708:     // Since we're adding an ENDLOOP, there better be a LOOP instruction.
   709:     // Check for it, and change the BB target if needed.
   710:     SmallPtrSet<MachineBasicBlock *, 8> VisitedBBs;
   711:     MachineInstr *Loop = findLoopInstr(TBB, EndLoopOp, Cond[1].getMBB(),
   712:                                        VisitedBBs);
   713:     assert(Loop != nullptr && "Inserting an ENDLOOP without a LOOP");
   714:     Loop->getOperand(0).setMBB(TBB);
   715:     // Add the ENDLOOP after the finding the LOOP0.
   716:     BuildMI(&MBB, DL, get(EndLoopOp)).addMBB(TBB);
   717:   } else {
   718:     const MachineOperand &RO = Cond[1];
   719:     RegState Flags = getUndefRegState(RO.isUndef());
   720:     BuildMI(&MBB, DL, get(BccOpc)).addReg(RO.getReg(), Flags).addMBB(TBB);
   721:   }
   722:   BuildMI(&MBB, DL, get(BOpc)).addMBB(FBB);
   723: 
   724:   return 2;
   725: }
   726: 
   727: namespace {
   728: class HexagonPipelinerLoopInfo : public TargetInstrInfo::PipelinerLoopInfo {
   729:   MachineInstr *Loop, *EndLoop;
   730:   MachineFunction *MF;
   731:   const HexagonInstrInfo *TII;
   732:   int64_t TripCount;
   733:   Register LoopCount;
   734:   DebugLoc DL;
   735: 
   736: public:
   737:   HexagonPipelinerLoopInfo(MachineInstr *Loop, MachineInstr *EndLoop)
   738:       : Loop(Loop), EndLoop(EndLoop), MF(Loop->getParent()->getParent()),
   739:         TII(MF->getSubtarget<HexagonSubtarget>().getInstrInfo()),
   740:         DL(Loop->getDebugLoc()) {
   741:     // Inspect the Loop instruction up-front, as it may be deleted when we call
   742:     // createTripCountGreaterCondition.
   743:     TripCount = Loop->getOpcode() == Hexagon::J2_loop0r
   744:                     ? -1
   745:                     : Loop->getOperand(1).getImm();
   746:     if (TripCount == -1)
   747:       LoopCount = Loop->getOperand(1).getReg();
   748:   }
   749: 
   750:   bool shouldIgnoreForPipelining(const MachineInstr *MI) const override {
   751:     // Only ignore the terminator.
   752:     return MI == EndLoop;
   753:   }
   754: 
   755:   std::optional<bool> createTripCountGreaterCondition(
   756:       int TC, MachineBasicBlock &MBB,
   757:       SmallVectorImpl<MachineOperand> &Cond) override {
   758:     if (TripCount == -1) {
   759:       // Check if we're done with the loop.
   760:       Register Done = TII->createVR(MF, MVT::i1);
   761:       MachineInstr *NewCmp = BuildMI(&MBB, DL,
   762:                                      TII->get(Hexagon::C2_cmpgtui), Done)
   763:                                  .addReg(LoopCount)
   764:                                  .addImm(TC);
   765:       Cond.push_back(MachineOperand::CreateImm(Hexagon::J2_jumpf));
   766:       Cond.push_back(NewCmp->getOperand(0));
   767:       return {};
   768:     }
   769: 
   770:     return TripCount > TC;
   771:   }
   772: 
   773:   void setPreheader(MachineBasicBlock *NewPreheader) override {
   774:     NewPreheader->splice(NewPreheader->getFirstTerminator(), Loop->getParent(),
   775:                          Loop);
   776:   }
   777: 
   778:   void adjustTripCount(int TripCountAdjust) override {
   779:     // If the loop trip count is a compile-time value, then just change the
   780:     // value.
   781:     if (Loop->getOpcode() == Hexagon::J2_loop0i ||
   782:         Loop->getOpcode() == Hexagon::J2_loop1i) {
   783:       int64_t TripCount = Loop->getOperand(1).getImm() + TripCountAdjust;
   784:       assert(TripCount > 0 && "Can't create an empty or negative loop!");
   785:       Loop->getOperand(1).setImm(TripCount);
   786:       return;
   787:     }
   788: 
   789:     // The loop trip count is a run-time value. We generate code to subtract
   790:     // one from the trip count, and update the loop instruction.
   791:     Register LoopCount = Loop->getOperand(1).getReg();
   792:     Register NewLoopCount = TII->createVR(MF, MVT::i32);
   793:     BuildMI(*Loop->getParent(), Loop, Loop->getDebugLoc(),
   794:             TII->get(Hexagon::A2_addi), NewLoopCount)
   795:         .addReg(LoopCount)
   796:         .addImm(TripCountAdjust);
   797:     Loop->getOperand(1).setReg(NewLoopCount);
   798:   }
   799: 
   800:   void disposed(LiveIntervals *LIS) override {
```
- EN: It declares types such as HexagonPipelinerLoopInfo, which carry the state or API of this component. It defines declarative TableGen records like HexagonPipelinerLoopInfo; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as HexagonInstrInfo::removeBranch, assert, LLVM_DEBUG, end, ... (26 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 HexagonPipelinerLoopInfo 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonPipelinerLoopInfo 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 HexagonInstrInfo::removeBranch, assert, LLVM_DEBUG, end, ... (26 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 801-1000 / 第 801-1000 行

```cpp
   801:     if (LIS)
   802:       LIS->RemoveMachineInstrFromMaps(*Loop);
   803:     Loop->eraseFromParent();
   804:   }
   805: };
   806: } // namespace
   807: 
   808: std::unique_ptr<TargetInstrInfo::PipelinerLoopInfo>
   809: HexagonInstrInfo::analyzeLoopForPipelining(MachineBasicBlock *LoopBB) const {
   810:   // We really "analyze" only hardware loops right now.
   811:   MachineBasicBlock::iterator I = LoopBB->getFirstTerminator();
   812: 
   813:   if (I != LoopBB->end() && isEndLoopN(I->getOpcode())) {
   814:     SmallPtrSet<MachineBasicBlock *, 8> VisitedBBs;
   815:     MachineInstr *LoopInst = findLoopInstr(
   816:         LoopBB, I->getOpcode(), I->getOperand(0).getMBB(), VisitedBBs);
   817:     if (LoopInst)
   818:       return std::make_unique<HexagonPipelinerLoopInfo>(LoopInst, &*I);
   819:   }
   820:   return nullptr;
   821: }
   822: 
   823: bool HexagonInstrInfo::isProfitableToIfCvt(MachineBasicBlock &MBB,
   824:       unsigned NumCycles, unsigned ExtraPredCycles,
   825:       BranchProbability Probability) const {
   826:   return nonDbgBBSize(&MBB) <= 3;
   827: }
   828: 
   829: bool HexagonInstrInfo::isProfitableToIfCvt(MachineBasicBlock &TMBB,
   830:       unsigned NumTCycles, unsigned ExtraTCycles, MachineBasicBlock &FMBB,
   831:       unsigned NumFCycles, unsigned ExtraFCycles, BranchProbability Probability)
   832:       const {
   833:   return nonDbgBBSize(&TMBB) <= 3 && nonDbgBBSize(&FMBB) <= 3;
   834: }
   835: 
   836: bool HexagonInstrInfo::isProfitableToDupForIfCvt(MachineBasicBlock &MBB,
   837:       unsigned NumInstrs, BranchProbability Probability) const {
   838:   return NumInstrs <= 4;
   839: }
   840: 
   841: static void getLiveInRegsAt(LivePhysRegs &Regs, const MachineInstr &MI) {
   842:   SmallVector<std::pair<MCPhysReg, const MachineOperand*>,2> Clobbers;
   843:   const MachineBasicBlock &B = *MI.getParent();
   844:   Regs.addLiveIns(B);
   845:   auto E = MachineBasicBlock::const_iterator(MI.getIterator());
   846:   for (auto I = B.begin(); I != E; ++I) {
   847:     Clobbers.clear();
   848:     Regs.stepForward(*I, Clobbers);
   849:   }
   850: }
   851: 
   852: static void getLiveOutRegsAt(LivePhysRegs &Regs, const MachineInstr &MI) {
   853:   const MachineBasicBlock &B = *MI.getParent();
   854:   Regs.addLiveOuts(B);
   855:   auto E = ++MachineBasicBlock::const_iterator(MI.getIterator()).getReverse();
   856:   for (auto I = B.rbegin(); I != E; ++I)
   857:     Regs.stepBackward(*I);
   858: }
   859: 
   860: void HexagonInstrInfo::copyPhysReg(MachineBasicBlock &MBB,
   861:                                    MachineBasicBlock::iterator I,
   862:                                    const DebugLoc &DL, Register DestReg,
   863:                                    Register SrcReg, bool KillSrc,
   864:                                    bool RenamableDest,
   865:                                    bool RenamableSrc) const {
   866:   const HexagonRegisterInfo &HRI = *Subtarget.getRegisterInfo();
   867:   RegState KillFlag = getKillRegState(KillSrc);
   868: 
   869:   if (Hexagon::IntRegsRegClass.contains(SrcReg, DestReg)) {
   870:     BuildMI(MBB, I, DL, get(Hexagon::A2_tfr), DestReg)
   871:       .addReg(SrcReg, KillFlag);
   872:     return;
   873:   }
   874:   if (Hexagon::DoubleRegsRegClass.contains(SrcReg, DestReg)) {
   875:     BuildMI(MBB, I, DL, get(Hexagon::A2_tfrp), DestReg)
   876:       .addReg(SrcReg, KillFlag);
   877:     return;
   878:   }
   879:   if (Hexagon::PredRegsRegClass.contains(SrcReg, DestReg)) {
   880:     // Map Pd = Ps to Pd = or(Ps, Ps).
   881:     BuildMI(MBB, I, DL, get(Hexagon::C2_or), DestReg)
   882:       .addReg(SrcReg).addReg(SrcReg, KillFlag);
   883:     return;
   884:   }
   885:   if (Hexagon::CtrRegsRegClass.contains(DestReg) &&
   886:       Hexagon::IntRegsRegClass.contains(SrcReg)) {
   887:     BuildMI(MBB, I, DL, get(Hexagon::A2_tfrrcr), DestReg)
   888:       .addReg(SrcReg, KillFlag);
   889:     return;
   890:   }
   891:   if (Hexagon::IntRegsRegClass.contains(DestReg) &&
   892:       Hexagon::CtrRegsRegClass.contains(SrcReg)) {
   893:     BuildMI(MBB, I, DL, get(Hexagon::A2_tfrcrr), DestReg)
   894:       .addReg(SrcReg, KillFlag);
   895:     return;
   896:   }
   897:   if (Hexagon::ModRegsRegClass.contains(DestReg) &&
   898:       Hexagon::IntRegsRegClass.contains(SrcReg)) {
   899:     BuildMI(MBB, I, DL, get(Hexagon::A2_tfrrcr), DestReg)
   900:       .addReg(SrcReg, KillFlag);
   901:     return;
   902:   }
   903:   if (Hexagon::PredRegsRegClass.contains(SrcReg) &&
   904:       Hexagon::IntRegsRegClass.contains(DestReg)) {
   905:     BuildMI(MBB, I, DL, get(Hexagon::C2_tfrpr), DestReg)
   906:       .addReg(SrcReg, KillFlag);
   907:     return;
   908:   }
   909:   if (Hexagon::IntRegsRegClass.contains(SrcReg) &&
   910:       Hexagon::PredRegsRegClass.contains(DestReg)) {
   911:     BuildMI(MBB, I, DL, get(Hexagon::C2_tfrrp), DestReg)
   912:       .addReg(SrcReg, KillFlag);
   913:     return;
   914:   }
   915:   if (Hexagon::PredRegsRegClass.contains(SrcReg) &&
   916:       Hexagon::IntRegsRegClass.contains(DestReg)) {
   917:     BuildMI(MBB, I, DL, get(Hexagon::C2_tfrpr), DestReg)
   918:       .addReg(SrcReg, KillFlag);
   919:     return;
   920:   }
   921:   if (Hexagon::HvxVRRegClass.contains(SrcReg, DestReg)) {
   922:     BuildMI(MBB, I, DL, get(Hexagon::V6_vassign), DestReg).
   923:       addReg(SrcReg, KillFlag);
   924:     return;
   925:   }
   926:   if (Hexagon::HvxWRRegClass.contains(SrcReg, DestReg)) {
   927:     LivePhysRegs LiveAtMI(HRI);
   928:     getLiveInRegsAt(LiveAtMI, *I);
   929:     Register SrcLo = HRI.getSubReg(SrcReg, Hexagon::vsub_lo);
   930:     Register SrcHi = HRI.getSubReg(SrcReg, Hexagon::vsub_hi);
   931:     RegState UndefLo = getUndefRegState(!LiveAtMI.contains(SrcLo));
   932:     RegState UndefHi = getUndefRegState(!LiveAtMI.contains(SrcHi));
   933:     BuildMI(MBB, I, DL, get(Hexagon::V6_vcombine), DestReg)
   934:       .addReg(SrcHi, KillFlag | UndefHi)
   935:       .addReg(SrcLo, KillFlag | UndefLo);
   936:     return;
   937:   }
   938:   if (Hexagon::HvxQRRegClass.contains(SrcReg, DestReg)) {
   939:     BuildMI(MBB, I, DL, get(Hexagon::V6_pred_and), DestReg)
   940:       .addReg(SrcReg)
   941:       .addReg(SrcReg, KillFlag);
   942:     return;
   943:   }
   944:   if (Hexagon::HvxQRRegClass.contains(SrcReg) &&
   945:       Hexagon::HvxVRRegClass.contains(DestReg)) {
   946:     llvm_unreachable("Unimplemented pred to vec");
   947:     return;
   948:   }
   949:   if (Hexagon::HvxQRRegClass.contains(DestReg) &&
   950:       Hexagon::HvxVRRegClass.contains(SrcReg)) {
   951:     llvm_unreachable("Unimplemented vec to pred");
   952:     return;
   953:   }
   954: 
   955: #ifndef NDEBUG
   956:   // Show the invalid registers to ease debugging.
   957:   dbgs() << "Invalid registers for copy in " << printMBBReference(MBB) << ": "
   958:          << printReg(DestReg, &HRI) << " = " << printReg(SrcReg, &HRI) << '\n';
   959: #endif
   960:   llvm_unreachable("Unimplemented");
   961: }
   962: 
   963: void HexagonInstrInfo::storeRegToStackSlot(MachineBasicBlock &MBB,
   964:                                            MachineBasicBlock::iterator I,
   965:                                            Register SrcReg, bool isKill, int FI,
   966:                                            const TargetRegisterClass *RC,
   967:                                            Register VReg,
   968:                                            MachineInstr::MIFlag Flags) const {
   969:   DebugLoc DL = MBB.findDebugLoc(I);
   970:   MachineFunction &MF = *MBB.getParent();
   971:   MachineFrameInfo &MFI = MF.getFrameInfo();
   972:   RegState KillFlag = getKillRegState(isKill);
   973: 
   974:   MachineMemOperand *MMO = MF.getMachineMemOperand(
   975:       MachinePointerInfo::getFixedStack(MF, FI), MachineMemOperand::MOStore,
   976:       MFI.getObjectSize(FI), MFI.getObjectAlign(FI));
   977: 
   978:   if (Hexagon::IntRegsRegClass.hasSubClassEq(RC)) {
   979:     BuildMI(MBB, I, DL, get(Hexagon::S2_storeri_io))
   980:       .addFrameIndex(FI).addImm(0)
   981:       .addReg(SrcReg, KillFlag).addMemOperand(MMO);
   982:   } else if (Hexagon::DoubleRegsRegClass.hasSubClassEq(RC)) {
   983:     BuildMI(MBB, I, DL, get(Hexagon::S2_storerd_io))
   984:       .addFrameIndex(FI).addImm(0)
   985:       .addReg(SrcReg, KillFlag).addMemOperand(MMO);
   986:   } else if (Hexagon::PredRegsRegClass.hasSubClassEq(RC)) {
   987:     BuildMI(MBB, I, DL, get(Hexagon::STriw_pred))
   988:       .addFrameIndex(FI).addImm(0)
   989:       .addReg(SrcReg, KillFlag).addMemOperand(MMO);
   990:   } else if (Hexagon::ModRegsRegClass.hasSubClassEq(RC)) {
   991:     BuildMI(MBB, I, DL, get(Hexagon::STriw_ctr))
   992:       .addFrameIndex(FI).addImm(0)
   993:       .addReg(SrcReg, KillFlag).addMemOperand(MMO);
   994:   } else if (Hexagon::HvxQRRegClass.hasSubClassEq(RC)) {
   995:     BuildMI(MBB, I, DL, get(Hexagon::PS_vstorerq_ai))
   996:       .addFrameIndex(FI).addImm(0)
   997:       .addReg(SrcReg, KillFlag).addMemOperand(MMO);
   998:   } else if (Hexagon::HvxVRRegClass.hasSubClassEq(RC)) {
   999:     BuildMI(MBB, I, DL, get(Hexagon::PS_vstorerv_ai))
  1000:       .addFrameIndex(FI).addImm(0)
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (std) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as eraseFromParent, HexagonInstrInfo::analyzeLoopForPipelining, getFirstTerminator, findLoopInstr, ... (28 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（std），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 eraseFromParent, HexagonInstrInfo::analyzeLoopForPipelining, getFirstTerminator, findLoopInstr, ... (28 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 1001-1200 / 第 1001-1200 行

```cpp
  1001:       .addReg(SrcReg, KillFlag).addMemOperand(MMO);
  1002:   } else if (Hexagon::HvxWRRegClass.hasSubClassEq(RC)) {
  1003:     BuildMI(MBB, I, DL, get(Hexagon::PS_vstorerw_ai))
  1004:       .addFrameIndex(FI).addImm(0)
  1005:       .addReg(SrcReg, KillFlag).addMemOperand(MMO);
  1006:   } else {
  1007:     llvm_unreachable("Unimplemented");
  1008:   }
  1009: }
  1010: 
  1011: void HexagonInstrInfo::loadRegFromStackSlot(MachineBasicBlock &MBB,
  1012:                                             MachineBasicBlock::iterator I,
  1013:                                             Register DestReg, int FI,
  1014:                                             const TargetRegisterClass *RC,
  1015:                                             Register VReg, unsigned SubReg,
  1016:                                             MachineInstr::MIFlag Flags) const {
  1017:   DebugLoc DL = MBB.findDebugLoc(I);
  1018:   MachineFunction &MF = *MBB.getParent();
  1019:   MachineFrameInfo &MFI = MF.getFrameInfo();
  1020: 
  1021:   MachineMemOperand *MMO = MF.getMachineMemOperand(
  1022:       MachinePointerInfo::getFixedStack(MF, FI), MachineMemOperand::MOLoad,
  1023:       MFI.getObjectSize(FI), MFI.getObjectAlign(FI));
  1024: 
  1025:   if (Hexagon::IntRegsRegClass.hasSubClassEq(RC)) {
  1026:     BuildMI(MBB, I, DL, get(Hexagon::L2_loadri_io), DestReg)
  1027:       .addFrameIndex(FI).addImm(0).addMemOperand(MMO);
  1028:   } else if (Hexagon::DoubleRegsRegClass.hasSubClassEq(RC)) {
  1029:     BuildMI(MBB, I, DL, get(Hexagon::L2_loadrd_io), DestReg)
  1030:       .addFrameIndex(FI).addImm(0).addMemOperand(MMO);
  1031:   } else if (Hexagon::PredRegsRegClass.hasSubClassEq(RC)) {
  1032:     BuildMI(MBB, I, DL, get(Hexagon::LDriw_pred), DestReg)
  1033:       .addFrameIndex(FI).addImm(0).addMemOperand(MMO);
  1034:   } else if (Hexagon::ModRegsRegClass.hasSubClassEq(RC)) {
  1035:     BuildMI(MBB, I, DL, get(Hexagon::LDriw_ctr), DestReg)
  1036:       .addFrameIndex(FI).addImm(0).addMemOperand(MMO);
  1037:   } else if (Hexagon::HvxQRRegClass.hasSubClassEq(RC)) {
  1038:     BuildMI(MBB, I, DL, get(Hexagon::PS_vloadrq_ai), DestReg)
  1039:       .addFrameIndex(FI).addImm(0).addMemOperand(MMO);
  1040:   } else if (Hexagon::HvxVRRegClass.hasSubClassEq(RC)) {
  1041:     BuildMI(MBB, I, DL, get(Hexagon::PS_vloadrv_ai), DestReg)
  1042:       .addFrameIndex(FI).addImm(0).addMemOperand(MMO);
  1043:   } else if (Hexagon::HvxWRRegClass.hasSubClassEq(RC)) {
  1044:     BuildMI(MBB, I, DL, get(Hexagon::PS_vloadrw_ai), DestReg)
  1045:       .addFrameIndex(FI).addImm(0).addMemOperand(MMO);
  1046:   } else {
  1047:     llvm_unreachable("Can't store this register to stack slot");
  1048:   }
  1049: }
  1050: 
  1051: /// expandPostRAPseudo - This function is called for all pseudo instructions
  1052: /// that remain after register allocation. Many pseudo instructions are
  1053: /// created to help register allocation. This is the place to convert them
  1054: /// into real instructions. The target can edit MI in place, or it can insert
  1055: /// new instructions and erase MI. The function should return true if
  1056: /// anything was changed.
  1057: bool HexagonInstrInfo::expandPostRAPseudo(MachineInstr &MI) const {
  1058:   MachineBasicBlock &MBB = *MI.getParent();
  1059:   MachineFunction &MF = *MBB.getParent();
  1060:   MachineRegisterInfo &MRI = MF.getRegInfo();
  1061:   const HexagonRegisterInfo &HRI = *Subtarget.getRegisterInfo();
  1062:   LivePhysRegs LiveIn(HRI), LiveOut(HRI);
  1063:   DebugLoc DL = MI.getDebugLoc();
  1064:   unsigned Opc = MI.getOpcode();
  1065: 
  1066:   auto RealCirc = [&](unsigned Opc, bool HasImm, unsigned MxOp) {
  1067:     Register Mx = MI.getOperand(MxOp).getReg();
  1068:     Register CSx = (Mx == Hexagon::M0 ? Hexagon::CS0 : Hexagon::CS1);
  1069:     BuildMI(MBB, MI, DL, get(Hexagon::A2_tfrrcr), CSx)
  1070:         .add(MI.getOperand((HasImm ? 5 : 4)));
  1071:     auto MIB = BuildMI(MBB, MI, DL, get(Opc)).add(MI.getOperand(0))
  1072:         .add(MI.getOperand(1)).add(MI.getOperand(2)).add(MI.getOperand(3));
  1073:     if (HasImm)
  1074:       MIB.add(MI.getOperand(4));
  1075:     MIB.addReg(CSx, RegState::Implicit);
  1076:     MBB.erase(MI);
  1077:     return true;
  1078:   };
  1079: 
  1080:   auto UseAligned = [&](const MachineInstr &MI, Align NeedAlign) {
  1081:     if (MI.memoperands().empty())
  1082:       return false;
  1083:     return all_of(MI.memoperands(), [NeedAlign](const MachineMemOperand *MMO) {
  1084:       return MMO->getAlign() >= NeedAlign;
  1085:     });
  1086:   };
  1087: 
  1088:   switch (Opc) {
  1089:     case Hexagon::PS_call_instrprof_custom: {
  1090:       auto Op0 = MI.getOperand(0);
  1091:       assert(Op0.isGlobal() &&
  1092:              "First operand must be a global containing handler name.");
  1093:       const GlobalValue *NameVar = Op0.getGlobal();
  1094:       const GlobalVariable *GV = dyn_cast<GlobalVariable>(NameVar);
  1095:       auto *Arr = cast<ConstantDataArray>(GV->getInitializer());
  1096:       StringRef NameStr = Arr->isCString() ? Arr->getAsCString() : Arr->getAsString();
  1097: 
  1098:       MachineOperand &Op1 = MI.getOperand(1);
  1099:       // Set R0 with the imm value to be passed to the custom profiling handler.
  1100:       BuildMI(MBB, MI, DL, get(Hexagon::A2_tfrsi), Hexagon::R0)
  1101:         .addImm(Op1.getImm());
  1102:       // The call to the custom handler is being treated as a special one as the
  1103:       // callee is responsible for saving and restoring all the registers
  1104:       // (including caller saved registers) it needs to modify. This is
  1105:       // done to reduce the impact of instrumentation on the code being
  1106:       // instrumented/profiled.
  1107:       // NOTE: R14, R15 and R28 are reserved for PLT handling. These registers
  1108:       // are in the Def list of the Hexagon::PS_call_instrprof_custom and
  1109:       // therefore will be handled appropriately duing register allocation.
  1110: 
  1111:       // TODO: It may be a good idea to add a separate pseudo instruction for
  1112:       // static relocation which doesn't need to reserve r14, r15 and r28.
  1113: 
  1114:       auto MIB = BuildMI(MBB, MI, DL, get(Hexagon::J2_call))
  1115:                  .addUse(Hexagon::R0, RegState::Implicit|RegState::InternalRead)
  1116:                  .addDef(Hexagon::R29, RegState::ImplicitDefine)
  1117:                  .addDef(Hexagon::R30, RegState::ImplicitDefine)
  1118:                  .addDef(Hexagon::R14, RegState::ImplicitDefine)
  1119:                  .addDef(Hexagon::R15, RegState::ImplicitDefine)
  1120:                  .addDef(Hexagon::R28, RegState::ImplicitDefine);
  1121:       const char *cstr = MF.createExternalSymbolName(NameStr);
  1122:       MIB.addExternalSymbol(cstr);
  1123:       MBB.erase(MI);
  1124:       return true;
  1125:     }
  1126:     case TargetOpcode::COPY: {
  1127:       MachineOperand &MD = MI.getOperand(0);
  1128:       MachineOperand &MS = MI.getOperand(1);
  1129:       MachineBasicBlock::iterator MBBI = MI.getIterator();
  1130:       if (MD.getReg() != MS.getReg() && !MS.isUndef()) {
  1131:         copyPhysReg(MBB, MI, DL, MD.getReg(), MS.getReg(), MS.isKill());
  1132:         std::prev(MBBI)->copyImplicitOps(*MBB.getParent(), MI);
  1133:       }
  1134:       MBB.erase(MBBI);
  1135:       return true;
  1136:     }
  1137:     case Hexagon::PS_aligna:
  1138:       BuildMI(MBB, MI, DL, get(Hexagon::A2_andir), MI.getOperand(0).getReg())
  1139:           .addReg(HRI.getFrameRegister())
  1140:           .addImm(-MI.getOperand(1).getImm());
  1141:       MBB.erase(MI);
  1142:       return true;
  1143:     case Hexagon::V6_vassignp: {
  1144:       Register SrcReg = MI.getOperand(1).getReg();
  1145:       Register DstReg = MI.getOperand(0).getReg();
  1146:       Register SrcLo = HRI.getSubReg(SrcReg, Hexagon::vsub_lo);
  1147:       Register SrcHi = HRI.getSubReg(SrcReg, Hexagon::vsub_hi);
  1148:       getLiveInRegsAt(LiveIn, MI);
  1149:       RegState UndefLo = getUndefRegState(!LiveIn.contains(SrcLo));
  1150:       RegState UndefHi = getUndefRegState(!LiveIn.contains(SrcHi));
  1151:       RegState Kill = getKillRegState(MI.getOperand(1).isKill());
  1152:       BuildMI(MBB, MI, DL, get(Hexagon::V6_vcombine), DstReg)
  1153:           .addReg(SrcHi, UndefHi)
  1154:           .addReg(SrcLo, Kill | UndefLo);
  1155:       MBB.erase(MI);
  1156:       return true;
  1157:     }
  1158:     case Hexagon::V6_lo: {
  1159:       Register SrcReg = MI.getOperand(1).getReg();
  1160:       Register DstReg = MI.getOperand(0).getReg();
  1161:       Register SrcSubLo = HRI.getSubReg(SrcReg, Hexagon::vsub_lo);
  1162:       copyPhysReg(MBB, MI, DL, DstReg, SrcSubLo, MI.getOperand(1).isKill());
  1163:       MBB.erase(MI);
  1164:       MRI.clearKillFlags(SrcSubLo);
  1165:       return true;
  1166:     }
  1167:     case Hexagon::V6_hi: {
  1168:       Register SrcReg = MI.getOperand(1).getReg();
  1169:       Register DstReg = MI.getOperand(0).getReg();
  1170:       Register SrcSubHi = HRI.getSubReg(SrcReg, Hexagon::vsub_hi);
  1171:       copyPhysReg(MBB, MI, DL, DstReg, SrcSubHi, MI.getOperand(1).isKill());
  1172:       MBB.erase(MI);
  1173:       MRI.clearKillFlags(SrcSubHi);
  1174:       return true;
  1175:     }
  1176:     case Hexagon::PS_vloadrv_ai: {
  1177:       Register DstReg = MI.getOperand(0).getReg();
  1178:       const MachineOperand &BaseOp = MI.getOperand(1);
  1179:       assert(BaseOp.getSubReg() == 0);
  1180:       int Offset = MI.getOperand(2).getImm();
  1181:       Align NeedAlign = HRI.getSpillAlign(Hexagon::HvxVRRegClass);
  1182:       unsigned NewOpc = UseAligned(MI, NeedAlign) ? Hexagon::V6_vL32b_ai
  1183:                                                   : Hexagon::V6_vL32Ub_ai;
  1184:       BuildMI(MBB, MI, DL, get(NewOpc), DstReg)
  1185:           .addReg(BaseOp.getReg(), getRegState(BaseOp))
  1186:           .addImm(Offset)
  1187:           .cloneMemRefs(MI);
  1188:       MBB.erase(MI);
  1189:       return true;
  1190:     }
  1191:     case Hexagon::PS_vloadrw_ai: {
  1192:       Register DstReg = MI.getOperand(0).getReg();
  1193:       const MachineOperand &BaseOp = MI.getOperand(1);
  1194:       assert(BaseOp.getSubReg() == 0);
  1195:       int Offset = MI.getOperand(2).getImm();
  1196:       unsigned VecOffset = HRI.getSpillSize(Hexagon::HvxVRRegClass);
  1197:       Align NeedAlign = HRI.getSpillAlign(Hexagon::HvxVRRegClass);
  1198:       unsigned NewOpc = UseAligned(MI, NeedAlign) ? Hexagon::V6_vL32b_ai
  1199:                                                   : Hexagon::V6_vL32Ub_ai;
  1200:       BuildMI(MBB, MI, DL, get(NewOpc),
```
- EN: It declares or implements routines such as addReg, BuildMI, llvm_unreachable, HexagonInstrInfo::loadRegFromStackSlot, ... (34 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonInstrInfo, HexagonRegisterInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 addReg, BuildMI, llvm_unreachable, HexagonInstrInfo::loadRegFromStackSlot, ... (34 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo, HexagonRegisterInfo，说明了它与同级后端组件的连接关系。

### Lines 1201-1400 / 第 1201-1400 行

```cpp
  1201:               HRI.getSubReg(DstReg, Hexagon::vsub_lo))
  1202:           .addReg(BaseOp.getReg(), getRegState(BaseOp) & ~RegState::Kill)
  1203:           .addImm(Offset)
  1204:           .cloneMemRefs(MI);
  1205:       BuildMI(MBB, MI, DL, get(NewOpc),
  1206:               HRI.getSubReg(DstReg, Hexagon::vsub_hi))
  1207:           .addReg(BaseOp.getReg(), getRegState(BaseOp))
  1208:           .addImm(Offset + VecOffset)
  1209:           .cloneMemRefs(MI);
  1210:       MBB.erase(MI);
  1211:       return true;
  1212:     }
  1213:     case Hexagon::PS_vstorerv_ai: {
  1214:       const MachineOperand &SrcOp = MI.getOperand(2);
  1215:       assert(SrcOp.getSubReg() == 0);
  1216:       const MachineOperand &BaseOp = MI.getOperand(0);
  1217:       assert(BaseOp.getSubReg() == 0);
  1218:       int Offset = MI.getOperand(1).getImm();
  1219:       Align NeedAlign = HRI.getSpillAlign(Hexagon::HvxVRRegClass);
  1220:       unsigned NewOpc = UseAligned(MI, NeedAlign) ? Hexagon::V6_vS32b_ai
  1221:                                                   : Hexagon::V6_vS32Ub_ai;
  1222:       BuildMI(MBB, MI, DL, get(NewOpc))
  1223:           .addReg(BaseOp.getReg(), getRegState(BaseOp))
  1224:           .addImm(Offset)
  1225:           .addReg(SrcOp.getReg(), getRegState(SrcOp))
  1226:           .cloneMemRefs(MI);
  1227:       MBB.erase(MI);
  1228:       return true;
  1229:     }
  1230:     case Hexagon::PS_vstorerw_ai: {
  1231:       Register SrcReg = MI.getOperand(2).getReg();
  1232:       const MachineOperand &BaseOp = MI.getOperand(0);
  1233:       assert(BaseOp.getSubReg() == 0);
  1234:       int Offset = MI.getOperand(1).getImm();
  1235:       unsigned VecOffset = HRI.getSpillSize(Hexagon::HvxVRRegClass);
  1236:       Align NeedAlign = HRI.getSpillAlign(Hexagon::HvxVRRegClass);
  1237:       unsigned NewOpc = UseAligned(MI, NeedAlign) ? Hexagon::V6_vS32b_ai
  1238:                                                   : Hexagon::V6_vS32Ub_ai;
  1239:       BuildMI(MBB, MI, DL, get(NewOpc))
  1240:           .addReg(BaseOp.getReg(), getRegState(BaseOp) & ~RegState::Kill)
  1241:           .addImm(Offset)
  1242:           .addReg(HRI.getSubReg(SrcReg, Hexagon::vsub_lo))
  1243:           .cloneMemRefs(MI);
  1244:       BuildMI(MBB, MI, DL, get(NewOpc))
  1245:           .addReg(BaseOp.getReg(), getRegState(BaseOp))
  1246:           .addImm(Offset + VecOffset)
  1247:           .addReg(HRI.getSubReg(SrcReg, Hexagon::vsub_hi))
  1248:           .cloneMemRefs(MI);
  1249:       MBB.erase(MI);
  1250:       return true;
  1251:     }
  1252:     case Hexagon::PS_true: {
  1253:       Register Reg = MI.getOperand(0).getReg();
  1254:       BuildMI(MBB, MI, DL, get(Hexagon::C2_orn), Reg)
  1255:         .addReg(Reg, RegState::Undef)
  1256:         .addReg(Reg, RegState::Undef);
  1257:       MBB.erase(MI);
  1258:       return true;
  1259:     }
  1260:     case Hexagon::PS_false: {
  1261:       Register Reg = MI.getOperand(0).getReg();
  1262:       BuildMI(MBB, MI, DL, get(Hexagon::C2_andn), Reg)
  1263:         .addReg(Reg, RegState::Undef)
  1264:         .addReg(Reg, RegState::Undef);
  1265:       MBB.erase(MI);
  1266:       return true;
  1267:     }
  1268:     case Hexagon::PS_qtrue: {
  1269:       BuildMI(MBB, MI, DL, get(Hexagon::V6_veqw), MI.getOperand(0).getReg())
  1270:         .addReg(Hexagon::V0, RegState::Undef)
  1271:         .addReg(Hexagon::V0, RegState::Undef);
  1272:       MBB.erase(MI);
  1273:       return true;
  1274:     }
  1275:     case Hexagon::PS_qfalse: {
  1276:       BuildMI(MBB, MI, DL, get(Hexagon::V6_vgtw), MI.getOperand(0).getReg())
  1277:         .addReg(Hexagon::V0, RegState::Undef)
  1278:         .addReg(Hexagon::V0, RegState::Undef);
  1279:       MBB.erase(MI);
  1280:       return true;
  1281:     }
  1282:     case Hexagon::PS_vdd0: {
  1283:       Register Vd = MI.getOperand(0).getReg();
  1284:       BuildMI(MBB, MI, DL, get(Hexagon::V6_vsubw_dv), Vd)
  1285:         .addReg(Vd, RegState::Undef)
  1286:         .addReg(Vd, RegState::Undef);
  1287:       MBB.erase(MI);
  1288:       return true;
  1289:     }
  1290:     case Hexagon::PS_vmulw: {
  1291:       // Expand a 64-bit vector multiply into 2 32-bit scalar multiplies.
  1292:       Register DstReg = MI.getOperand(0).getReg();
  1293:       Register Src1Reg = MI.getOperand(1).getReg();
  1294:       Register Src2Reg = MI.getOperand(2).getReg();
  1295:       Register Src1SubHi = HRI.getSubReg(Src1Reg, Hexagon::isub_hi);
  1296:       Register Src1SubLo = HRI.getSubReg(Src1Reg, Hexagon::isub_lo);
  1297:       Register Src2SubHi = HRI.getSubReg(Src2Reg, Hexagon::isub_hi);
  1298:       Register Src2SubLo = HRI.getSubReg(Src2Reg, Hexagon::isub_lo);
  1299:       BuildMI(MBB, MI, MI.getDebugLoc(), get(Hexagon::M2_mpyi),
  1300:               HRI.getSubReg(DstReg, Hexagon::isub_hi))
  1301:           .addReg(Src1SubHi)
  1302:           .addReg(Src2SubHi);
  1303:       BuildMI(MBB, MI, MI.getDebugLoc(), get(Hexagon::M2_mpyi),
  1304:               HRI.getSubReg(DstReg, Hexagon::isub_lo))
  1305:           .addReg(Src1SubLo)
  1306:           .addReg(Src2SubLo);
  1307:       MBB.erase(MI);
  1308:       MRI.clearKillFlags(Src1SubHi);
  1309:       MRI.clearKillFlags(Src1SubLo);
  1310:       MRI.clearKillFlags(Src2SubHi);
  1311:       MRI.clearKillFlags(Src2SubLo);
  1312:       return true;
  1313:     }
  1314:     case Hexagon::PS_vmulw_acc: {
  1315:       // Expand 64-bit vector multiply with addition into 2 scalar multiplies.
  1316:       Register DstReg = MI.getOperand(0).getReg();
  1317:       Register Src1Reg = MI.getOperand(1).getReg();
  1318:       Register Src2Reg = MI.getOperand(2).getReg();
  1319:       Register Src3Reg = MI.getOperand(3).getReg();
  1320:       Register Src1SubHi = HRI.getSubReg(Src1Reg, Hexagon::isub_hi);
  1321:       Register Src1SubLo = HRI.getSubReg(Src1Reg, Hexagon::isub_lo);
  1322:       Register Src2SubHi = HRI.getSubReg(Src2Reg, Hexagon::isub_hi);
  1323:       Register Src2SubLo = HRI.getSubReg(Src2Reg, Hexagon::isub_lo);
  1324:       Register Src3SubHi = HRI.getSubReg(Src3Reg, Hexagon::isub_hi);
  1325:       Register Src3SubLo = HRI.getSubReg(Src3Reg, Hexagon::isub_lo);
  1326:       BuildMI(MBB, MI, MI.getDebugLoc(), get(Hexagon::M2_maci),
  1327:               HRI.getSubReg(DstReg, Hexagon::isub_hi))
  1328:           .addReg(Src1SubHi)
  1329:           .addReg(Src2SubHi)
  1330:           .addReg(Src3SubHi);
  1331:       BuildMI(MBB, MI, MI.getDebugLoc(), get(Hexagon::M2_maci),
  1332:               HRI.getSubReg(DstReg, Hexagon::isub_lo))
  1333:           .addReg(Src1SubLo)
  1334:           .addReg(Src2SubLo)
  1335:           .addReg(Src3SubLo);
  1336:       MBB.erase(MI);
  1337:       MRI.clearKillFlags(Src1SubHi);
  1338:       MRI.clearKillFlags(Src1SubLo);
  1339:       MRI.clearKillFlags(Src2SubHi);
  1340:       MRI.clearKillFlags(Src2SubLo);
  1341:       MRI.clearKillFlags(Src3SubHi);
  1342:       MRI.clearKillFlags(Src3SubLo);
  1343:       return true;
  1344:     }
  1345:     case Hexagon::PS_pselect: {
  1346:       const MachineOperand &Op0 = MI.getOperand(0);
  1347:       const MachineOperand &Op1 = MI.getOperand(1);
  1348:       const MachineOperand &Op2 = MI.getOperand(2);
  1349:       const MachineOperand &Op3 = MI.getOperand(3);
  1350:       Register Rd = Op0.getReg();
  1351:       Register Pu = Op1.getReg();
  1352:       Register Rs = Op2.getReg();
  1353:       Register Rt = Op3.getReg();
  1354:       DebugLoc DL = MI.getDebugLoc();
  1355:       RegState K1 = getKillRegState(Op1.isKill());
  1356:       RegState K2 = getKillRegState(Op2.isKill());
  1357:       RegState K3 = getKillRegState(Op3.isKill());
  1358:       if (Rd != Rs)
  1359:         BuildMI(MBB, MI, DL, get(Hexagon::A2_tfrpt), Rd)
  1360:             .addReg(Pu, (Rd == Rt) ? K1 : RegState::NoFlags)
  1361:             .addReg(Rs, K2);
  1362:       if (Rd != Rt)
  1363:         BuildMI(MBB, MI, DL, get(Hexagon::A2_tfrpf), Rd)
  1364:           .addReg(Pu, K1)
  1365:           .addReg(Rt, K3);
  1366:       MBB.erase(MI);
  1367:       return true;
  1368:     }
  1369:     case Hexagon::PS_vselect: {
  1370:       const MachineOperand &Op0 = MI.getOperand(0);
  1371:       const MachineOperand &Op1 = MI.getOperand(1);
  1372:       const MachineOperand &Op2 = MI.getOperand(2);
  1373:       const MachineOperand &Op3 = MI.getOperand(3);
  1374:       getLiveOutRegsAt(LiveOut, MI);
  1375:       bool IsDestLive = !LiveOut.available(MRI, Op0.getReg());
  1376:       Register PReg = Op1.getReg();
  1377:       assert(Op1.getSubReg() == 0);
  1378:       RegState PState = getRegState(Op1);
  1379: 
  1380:       if (Op0.getReg() != Op2.getReg()) {
  1381:         RegState S =
  1382:             Op0.getReg() != Op3.getReg() ? PState & ~RegState::Kill : PState;
  1383:         auto T = BuildMI(MBB, MI, DL, get(Hexagon::V6_vcmov))
  1384:                      .add(Op0)
  1385:                      .addReg(PReg, S)
  1386:                      .add(Op2);
  1387:         if (IsDestLive)
  1388:           T.addReg(Op0.getReg(), RegState::Implicit);
  1389:         IsDestLive = true;
  1390:       }
  1391:       if (Op0.getReg() != Op3.getReg()) {
  1392:         auto T = BuildMI(MBB, MI, DL, get(Hexagon::V6_vncmov))
  1393:                      .add(Op0)
  1394:                      .addReg(PReg, PState)
  1395:                      .add(Op3);
  1396:         if (IsDestLive)
  1397:           T.addReg(Op0.getReg(), RegState::Implicit);
  1398:       }
  1399:       MBB.erase(MI);
  1400:       return true;
```
- EN: It declares or implements routines such as getSubReg, BuildMI, erase, getOperand, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 getSubReg, BuildMI, erase, getOperand, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1401-1600 / 第 1401-1600 行

```cpp
  1401:     }
  1402:     case Hexagon::PS_wselect: {
  1403:       MachineOperand &Op0 = MI.getOperand(0);
  1404:       MachineOperand &Op1 = MI.getOperand(1);
  1405:       MachineOperand &Op2 = MI.getOperand(2);
  1406:       MachineOperand &Op3 = MI.getOperand(3);
  1407:       getLiveOutRegsAt(LiveOut, MI);
  1408:       bool IsDestLive = !LiveOut.available(MRI, Op0.getReg());
  1409:       Register PReg = Op1.getReg();
  1410:       assert(Op1.getSubReg() == 0);
  1411:       RegState PState = getRegState(Op1);
  1412: 
  1413:       if (Op0.getReg() != Op2.getReg()) {
  1414:         RegState S =
  1415:             Op0.getReg() != Op3.getReg() ? PState & ~RegState::Kill : PState;
  1416:         Register SrcLo = HRI.getSubReg(Op2.getReg(), Hexagon::vsub_lo);
  1417:         Register SrcHi = HRI.getSubReg(Op2.getReg(), Hexagon::vsub_hi);
  1418:         auto T = BuildMI(MBB, MI, DL, get(Hexagon::V6_vccombine))
  1419:                      .add(Op0)
  1420:                      .addReg(PReg, S)
  1421:                      .addReg(SrcHi)
  1422:                      .addReg(SrcLo);
  1423:         if (IsDestLive)
  1424:           T.addReg(Op0.getReg(), RegState::Implicit);
  1425:         IsDestLive = true;
  1426:       }
  1427:       if (Op0.getReg() != Op3.getReg()) {
  1428:         Register SrcLo = HRI.getSubReg(Op3.getReg(), Hexagon::vsub_lo);
  1429:         Register SrcHi = HRI.getSubReg(Op3.getReg(), Hexagon::vsub_hi);
  1430:         auto T = BuildMI(MBB, MI, DL, get(Hexagon::V6_vnccombine))
  1431:                      .add(Op0)
  1432:                      .addReg(PReg, PState)
  1433:                      .addReg(SrcHi)
  1434:                      .addReg(SrcLo);
  1435:         if (IsDestLive)
  1436:           T.addReg(Op0.getReg(), RegState::Implicit);
  1437:       }
  1438:       MBB.erase(MI);
  1439:       return true;
  1440:     }
  1441: 
  1442:     case Hexagon::PS_crash: {
  1443:       // Generate a misaligned load that is guaranteed to cause a crash.
  1444:       class CrashPseudoSourceValue : public PseudoSourceValue {
  1445:       public:
  1446:         CrashPseudoSourceValue(const TargetMachine &TM)
  1447:             : PseudoSourceValue(TargetCustom, TM) {}
  1448: 
  1449:         bool isConstant(const MachineFrameInfo *) const override {
  1450:           return false;
  1451:         }
  1452:         bool isAliased(const MachineFrameInfo *) const override {
  1453:           return false;
  1454:         }
  1455:         bool mayAlias(const MachineFrameInfo *) const override {
  1456:           return false;
  1457:         }
  1458:         void printCustom(raw_ostream &OS) const override {
  1459:           OS << "MisalignedCrash";
  1460:         }
  1461:       };
  1462: 
  1463:       static const CrashPseudoSourceValue CrashPSV(MF.getTarget());
  1464:       MachineMemOperand *MMO = MF.getMachineMemOperand(
  1465:           MachinePointerInfo(&CrashPSV),
  1466:           MachineMemOperand::MOLoad | MachineMemOperand::MOVolatile, 8,
  1467:           Align(1));
  1468:       BuildMI(MBB, MI, DL, get(Hexagon::PS_loadrdabs), Hexagon::D13)
  1469:         .addImm(0xBADC0FEE)  // Misaligned load.
  1470:         .addMemOperand(MMO);
  1471:       MBB.erase(MI);
  1472:       return true;
  1473:     }
  1474: 
  1475:     case Hexagon::PS_tailcall_i:
  1476:       MI.setDesc(get(Hexagon::J2_jump));
  1477:       return true;
  1478:     case Hexagon::PS_tailcall_r:
  1479:     case Hexagon::PS_jmpret:
  1480:       MI.setDesc(get(Hexagon::J2_jumpr));
  1481:       return true;
  1482:     case Hexagon::PS_jmprett:
  1483:       MI.setDesc(get(Hexagon::J2_jumprt));
  1484:       return true;
  1485:     case Hexagon::PS_jmpretf:
  1486:       MI.setDesc(get(Hexagon::J2_jumprf));
  1487:       return true;
  1488:     case Hexagon::PS_jmprettnewpt:
  1489:       MI.setDesc(get(Hexagon::J2_jumprtnewpt));
  1490:       return true;
  1491:     case Hexagon::PS_jmpretfnewpt:
  1492:       MI.setDesc(get(Hexagon::J2_jumprfnewpt));
  1493:       return true;
  1494:     case Hexagon::PS_jmprettnew:
  1495:       MI.setDesc(get(Hexagon::J2_jumprtnew));
  1496:       return true;
  1497:     case Hexagon::PS_jmpretfnew:
  1498:       MI.setDesc(get(Hexagon::J2_jumprfnew));
  1499:       return true;
  1500: 
  1501:     case Hexagon::PS_loadrub_pci:
  1502:       return RealCirc(Hexagon::L2_loadrub_pci, /*HasImm*/true,  /*MxOp*/4);
  1503:     case Hexagon::PS_loadrb_pci:
  1504:       return RealCirc(Hexagon::L2_loadrb_pci,  /*HasImm*/true,  /*MxOp*/4);
  1505:     case Hexagon::PS_loadruh_pci:
  1506:       return RealCirc(Hexagon::L2_loadruh_pci, /*HasImm*/true,  /*MxOp*/4);
  1507:     case Hexagon::PS_loadrh_pci:
  1508:       return RealCirc(Hexagon::L2_loadrh_pci,  /*HasImm*/true,  /*MxOp*/4);
  1509:     case Hexagon::PS_loadri_pci:
  1510:       return RealCirc(Hexagon::L2_loadri_pci,  /*HasImm*/true,  /*MxOp*/4);
  1511:     case Hexagon::PS_loadrd_pci:
  1512:       return RealCirc(Hexagon::L2_loadrd_pci,  /*HasImm*/true,  /*MxOp*/4);
  1513:     case Hexagon::PS_loadrub_pcr:
  1514:       return RealCirc(Hexagon::L2_loadrub_pcr, /*HasImm*/false, /*MxOp*/3);
  1515:     case Hexagon::PS_loadrb_pcr:
  1516:       return RealCirc(Hexagon::L2_loadrb_pcr,  /*HasImm*/false, /*MxOp*/3);
  1517:     case Hexagon::PS_loadruh_pcr:
  1518:       return RealCirc(Hexagon::L2_loadruh_pcr, /*HasImm*/false, /*MxOp*/3);
  1519:     case Hexagon::PS_loadrh_pcr:
  1520:       return RealCirc(Hexagon::L2_loadrh_pcr,  /*HasImm*/false, /*MxOp*/3);
  1521:     case Hexagon::PS_loadri_pcr:
  1522:       return RealCirc(Hexagon::L2_loadri_pcr,  /*HasImm*/false, /*MxOp*/3);
  1523:     case Hexagon::PS_loadrd_pcr:
  1524:       return RealCirc(Hexagon::L2_loadrd_pcr,  /*HasImm*/false, /*MxOp*/3);
  1525:     case Hexagon::PS_storerb_pci:
  1526:       return RealCirc(Hexagon::S2_storerb_pci, /*HasImm*/true,  /*MxOp*/3);
  1527:     case Hexagon::PS_storerh_pci:
  1528:       return RealCirc(Hexagon::S2_storerh_pci, /*HasImm*/true,  /*MxOp*/3);
  1529:     case Hexagon::PS_storerf_pci:
  1530:       return RealCirc(Hexagon::S2_storerf_pci, /*HasImm*/true,  /*MxOp*/3);
  1531:     case Hexagon::PS_storeri_pci:
  1532:       return RealCirc(Hexagon::S2_storeri_pci, /*HasImm*/true,  /*MxOp*/3);
  1533:     case Hexagon::PS_storerd_pci:
  1534:       return RealCirc(Hexagon::S2_storerd_pci, /*HasImm*/true,  /*MxOp*/3);
  1535:     case Hexagon::PS_storerb_pcr:
  1536:       return RealCirc(Hexagon::S2_storerb_pcr, /*HasImm*/false, /*MxOp*/2);
  1537:     case Hexagon::PS_storerh_pcr:
  1538:       return RealCirc(Hexagon::S2_storerh_pcr, /*HasImm*/false, /*MxOp*/2);
  1539:     case Hexagon::PS_storerf_pcr:
  1540:       return RealCirc(Hexagon::S2_storerf_pcr, /*HasImm*/false, /*MxOp*/2);
  1541:     case Hexagon::PS_storeri_pcr:
  1542:       return RealCirc(Hexagon::S2_storeri_pcr, /*HasImm*/false, /*MxOp*/2);
  1543:     case Hexagon::PS_storerd_pcr:
  1544:       return RealCirc(Hexagon::S2_storerd_pcr, /*HasImm*/false, /*MxOp*/2);
  1545:   }
  1546: 
  1547:   return false;
  1548: }
  1549: 
  1550: MachineBasicBlock::instr_iterator
  1551: HexagonInstrInfo::expandVGatherPseudo(MachineInstr &MI) const {
  1552:   MachineBasicBlock &MBB = *MI.getParent();
  1553:   const DebugLoc &DL = MI.getDebugLoc();
  1554:   unsigned Opc = MI.getOpcode();
  1555:   MachineBasicBlock::iterator First;
  1556: 
  1557:   switch (Opc) {
  1558:   case Hexagon::V6_vgather_vscatter_mh_pseudo:
  1559:     // This is mainly a place holder. It will be extended.
  1560:     First = BuildMI(MBB, MI, DL, get(Hexagon::V6_vgathermh))
  1561:                 .add(MI.getOperand(2))
  1562:                 .add(MI.getOperand(3))
  1563:                 .add(MI.getOperand(4));
  1564:     BuildMI(MBB, MI, DL, get(Hexagon::V6_vscattermh))
  1565:         .add(MI.getOperand(2))
  1566:         .add(MI.getOperand(3))
  1567:         .add(MI.getOperand(4))
  1568:         .addReg(Hexagon::VTMP);
  1569:     MBB.erase(MI);
  1570:     return First.getInstrIterator();
  1571:   case Hexagon::V6_vgathermh_pseudo:
  1572:     First = BuildMI(MBB, MI, DL, get(Hexagon::V6_vgathermh))
  1573:                 .add(MI.getOperand(2))
  1574:                 .add(MI.getOperand(3))
  1575:                 .add(MI.getOperand(4));
  1576:     BuildMI(MBB, MI, DL, get(Hexagon::V6_vS32b_new_ai))
  1577:         .add(MI.getOperand(0))
  1578:         .addImm(MI.getOperand(1).getImm())
  1579:         .addReg(Hexagon::VTMP);
  1580:     MBB.erase(MI);
  1581:     return First.getInstrIterator();
  1582: 
  1583:   case Hexagon::V6_vgathermw_pseudo:
  1584:     First = BuildMI(MBB, MI, DL, get(Hexagon::V6_vgathermw))
  1585:                 .add(MI.getOperand(2))
  1586:                 .add(MI.getOperand(3))
  1587:                 .add(MI.getOperand(4));
  1588:     BuildMI(MBB, MI, DL, get(Hexagon::V6_vS32b_new_ai))
  1589:         .add(MI.getOperand(0))
  1590:         .addImm(MI.getOperand(1).getImm())
  1591:         .addReg(Hexagon::VTMP);
  1592:     MBB.erase(MI);
  1593:     return First.getInstrIterator();
  1594: 
  1595:   case Hexagon::V6_vgathermhw_pseudo:
  1596:     First = BuildMI(MBB, MI, DL, get(Hexagon::V6_vgathermhw))
  1597:                 .add(MI.getOperand(2))
  1598:                 .add(MI.getOperand(3))
  1599:                 .add(MI.getOperand(4));
  1600:     BuildMI(MBB, MI, DL, get(Hexagon::V6_vS32b_new_ai))
```
- EN: It declares types such as CrashPseudoSourceValue, which carry the state or API of this component. It defines declarative TableGen records like CrashPseudoSourceValue; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as getOperand, getLiveOutRegsAt, available, getReg, ... (23 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 CrashPseudoSourceValue 等类型，用来承载该组件的状态或接口。 这里定义了 CrashPseudoSourceValue 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 getOperand, getLiveOutRegsAt, available, getReg, ... (23 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 1601-1800 / 第 1601-1800 行

```cpp
  1601:         .add(MI.getOperand(0))
  1602:         .addImm(MI.getOperand(1).getImm())
  1603:         .addReg(Hexagon::VTMP);
  1604:     MBB.erase(MI);
  1605:     return First.getInstrIterator();
  1606: 
  1607:   case Hexagon::V6_vgathermhq_pseudo:
  1608:     First = BuildMI(MBB, MI, DL, get(Hexagon::V6_vgathermhq))
  1609:                 .add(MI.getOperand(2))
  1610:                 .add(MI.getOperand(3))
  1611:                 .add(MI.getOperand(4))
  1612:                 .add(MI.getOperand(5));
  1613:     BuildMI(MBB, MI, DL, get(Hexagon::V6_vS32b_new_ai))
  1614:         .add(MI.getOperand(0))
  1615:         .addImm(MI.getOperand(1).getImm())
  1616:         .addReg(Hexagon::VTMP);
  1617:     MBB.erase(MI);
  1618:     return First.getInstrIterator();
  1619: 
  1620:   case Hexagon::V6_vgathermwq_pseudo:
  1621:     First = BuildMI(MBB, MI, DL, get(Hexagon::V6_vgathermwq))
  1622:                 .add(MI.getOperand(2))
  1623:                 .add(MI.getOperand(3))
  1624:                 .add(MI.getOperand(4))
  1625:                 .add(MI.getOperand(5));
  1626:     BuildMI(MBB, MI, DL, get(Hexagon::V6_vS32b_new_ai))
  1627:         .add(MI.getOperand(0))
  1628:         .addImm(MI.getOperand(1).getImm())
  1629:         .addReg(Hexagon::VTMP);
  1630:     MBB.erase(MI);
  1631:     return First.getInstrIterator();
  1632: 
  1633:   case Hexagon::V6_vgathermhwq_pseudo:
  1634:     First = BuildMI(MBB, MI, DL, get(Hexagon::V6_vgathermhwq))
  1635:                 .add(MI.getOperand(2))
  1636:                 .add(MI.getOperand(3))
  1637:                 .add(MI.getOperand(4))
  1638:                 .add(MI.getOperand(5));
  1639:     BuildMI(MBB, MI, DL, get(Hexagon::V6_vS32b_new_ai))
  1640:         .add(MI.getOperand(0))
  1641:         .addImm(MI.getOperand(1).getImm())
  1642:         .addReg(Hexagon::VTMP);
  1643:     MBB.erase(MI);
  1644:     return First.getInstrIterator();
  1645:   }
  1646: 
  1647:   return MI.getIterator();
  1648: }
  1649: 
  1650: // We indicate that we want to reverse the branch by
  1651: // inserting the reversed branching opcode.
  1652: bool HexagonInstrInfo::reverseBranchCondition(
  1653:       SmallVectorImpl<MachineOperand> &Cond) const {
  1654:   if (Cond.empty())
  1655:     return true;
  1656:   assert(Cond[0].isImm() && "First entry in the cond vector not imm-val");
  1657:   unsigned opcode = Cond[0].getImm();
  1658:   //unsigned temp;
  1659:   assert(get(opcode).isBranch() && "Should be a branching condition.");
  1660:   if (isEndLoopN(opcode))
  1661:     return true;
  1662:   unsigned NewOpcode = getInvertedPredicatedOpcode(opcode);
  1663:   Cond[0].setImm(NewOpcode);
  1664:   return false;
  1665: }
  1666: 
  1667: void HexagonInstrInfo::insertNoop(MachineBasicBlock &MBB,
  1668:       MachineBasicBlock::iterator MI) const {
  1669:   DebugLoc DL;
  1670:   BuildMI(MBB, MI, DL, get(Hexagon::A2_nop));
  1671: }
  1672: 
  1673: bool HexagonInstrInfo::isPostIncrement(const MachineInstr &MI) const {
  1674:   return getAddrMode(MI) == HexagonII::PostInc;
  1675: }
  1676: 
  1677: // Returns true if an instruction is predicated irrespective of the predicate
  1678: // sense. For example, all of the following will return true.
  1679: // if (p0) R1 = add(R2, R3)
  1680: // if (!p0) R1 = add(R2, R3)
  1681: // if (p0.new) R1 = add(R2, R3)
  1682: // if (!p0.new) R1 = add(R2, R3)
  1683: // Note: New-value stores are not included here as in the current
  1684: // implementation, we don't need to check their predicate sense.
  1685: bool HexagonInstrInfo::isPredicated(const MachineInstr &MI) const {
  1686:   const uint64_t F = MI.getDesc().TSFlags;
  1687:   return (F >> HexagonII::PredicatedPos) & HexagonII::PredicatedMask;
  1688: }
  1689: 
  1690: bool HexagonInstrInfo::PredicateInstruction(
  1691:     MachineInstr &MI, ArrayRef<MachineOperand> Cond) const {
  1692:   if (Cond.empty() || isNewValueJump(Cond[0].getImm()) ||
  1693:       isEndLoopN(Cond[0].getImm())) {
  1694:     LLVM_DEBUG(dbgs() << "\nCannot predicate:"; MI.dump(););
  1695:     return false;
  1696:   }
  1697:   int Opc = MI.getOpcode();
  1698:   assert (isPredicable(MI) && "Expected predicable instruction");
  1699:   bool invertJump = predOpcodeHasNot(Cond);
  1700: 
  1701:   // We have to predicate MI "in place", i.e. after this function returns,
  1702:   // MI will need to be transformed into a predicated form. To avoid com-
  1703:   // plicated manipulations with the operands (handling tied operands,
  1704:   // etc.), build a new temporary instruction, then overwrite MI with it.
  1705: 
  1706:   MachineBasicBlock &B = *MI.getParent();
  1707:   DebugLoc DL = MI.getDebugLoc();
  1708:   unsigned PredOpc = getCondOpcode(Opc, invertJump);
  1709:   MachineInstrBuilder T = BuildMI(B, MI, DL, get(PredOpc));
  1710:   unsigned NOp = 0, NumOps = MI.getNumOperands();
  1711:   while (NOp < NumOps) {
  1712:     MachineOperand &Op = MI.getOperand(NOp);
  1713:     if (!Op.isReg() || !Op.isDef() || Op.isImplicit())
  1714:       break;
  1715:     T.add(Op);
  1716:     NOp++;
  1717:   }
  1718: 
  1719:   Register PredReg;
  1720:   unsigned PredRegPos;
  1721:   RegState PredRegFlags = {};
  1722:   bool GotPredReg = getPredReg(Cond, PredReg, PredRegPos, PredRegFlags);
  1723:   (void)GotPredReg;
  1724:   assert(GotPredReg);
  1725:   T.addReg(PredReg, PredRegFlags);
  1726:   while (NOp < NumOps)
  1727:     T.add(MI.getOperand(NOp++));
  1728: 
  1729:   MI.setDesc(get(PredOpc));
  1730:   while (unsigned n = MI.getNumOperands())
  1731:     MI.removeOperand(n-1);
  1732:   for (unsigned i = 0, n = T->getNumOperands(); i < n; ++i)
  1733:     MI.addOperand(T->getOperand(i));
  1734: 
  1735:   MachineBasicBlock::instr_iterator TI = T->getIterator();
  1736:   B.erase(TI);
  1737: 
  1738:   MachineRegisterInfo &MRI = B.getParent()->getRegInfo();
  1739:   MRI.clearKillFlags(PredReg);
  1740:   return true;
  1741: }
  1742: 
  1743: bool HexagonInstrInfo::SubsumesPredicate(ArrayRef<MachineOperand> Pred1,
  1744:       ArrayRef<MachineOperand> Pred2) const {
  1745:   // TODO: Fix this
  1746:   return false;
  1747: }
  1748: 
  1749: bool HexagonInstrInfo::ClobbersPredicate(MachineInstr &MI,
  1750:                                          std::vector<MachineOperand> &Pred,
  1751:                                          bool SkipDead) const {
  1752:   const HexagonRegisterInfo &HRI = *Subtarget.getRegisterInfo();
  1753: 
  1754:   for (const MachineOperand &MO : MI.operands()) {
  1755:     if (MO.isReg()) {
  1756:       if (!MO.isDef())
  1757:         continue;
  1758:       const TargetRegisterClass* RC = HRI.getMinimalPhysRegClass(MO.getReg());
  1759:       if (RC == &Hexagon::PredRegsRegClass) {
  1760:         Pred.push_back(MO);
  1761:         return true;
  1762:       }
  1763:       continue;
  1764:     } else if (MO.isRegMask()) {
  1765:       for (Register PR : Hexagon::PredRegsRegClass) {
  1766:         if (!MI.modifiesRegister(PR, &HRI))
  1767:           continue;
  1768:         Pred.push_back(MO);
  1769:         return true;
  1770:       }
  1771:     }
  1772:   }
  1773:   return false;
  1774: }
  1775: 
  1776: bool HexagonInstrInfo::isPredicable(const MachineInstr &MI) const {
  1777:   if (!MI.getDesc().isPredicable())
  1778:     return false;
  1779: 
  1780:   if (MI.isCall() || isTailCall(MI)) {
  1781:     if (!Subtarget.usePredicatedCalls())
  1782:       return false;
  1783:   }
  1784: 
  1785:   // HVX loads are not predicable on v60, but are on v62.
  1786:   if (!Subtarget.hasV62Ops()) {
  1787:     switch (MI.getOpcode()) {
  1788:       case Hexagon::V6_vL32b_ai:
  1789:       case Hexagon::V6_vL32b_pi:
  1790:       case Hexagon::V6_vL32b_ppu:
  1791:       case Hexagon::V6_vL32b_cur_ai:
  1792:       case Hexagon::V6_vL32b_cur_pi:
  1793:       case Hexagon::V6_vL32b_cur_ppu:
  1794:       case Hexagon::V6_vL32b_nt_ai:
  1795:       case Hexagon::V6_vL32b_nt_pi:
  1796:       case Hexagon::V6_vL32b_nt_ppu:
  1797:       case Hexagon::V6_vL32b_tmp_ai:
  1798:       case Hexagon::V6_vL32b_tmp_pi:
  1799:       case Hexagon::V6_vL32b_tmp_ppu:
  1800:       case Hexagon::V6_vL32b_nt_cur_ai:
```
- EN: It declares or implements routines such as add, erase, getInstrIterator, BuildMI, ... (33 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonInstrInfo, HexagonII, HexagonRegisterInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 add, erase, getInstrIterator, BuildMI, ... (33 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo, HexagonII, HexagonRegisterInfo，说明了它与同级后端组件的连接关系。

### Lines 1801-2000 / 第 1801-2000 行

```cpp
  1801:       case Hexagon::V6_vL32b_nt_cur_pi:
  1802:       case Hexagon::V6_vL32b_nt_cur_ppu:
  1803:       case Hexagon::V6_vL32b_nt_tmp_ai:
  1804:       case Hexagon::V6_vL32b_nt_tmp_pi:
  1805:       case Hexagon::V6_vL32b_nt_tmp_ppu:
  1806:         return false;
  1807:     }
  1808:   }
  1809:   return true;
  1810: }
  1811: 
  1812: bool HexagonInstrInfo::isAssociativeAndCommutative(const MachineInstr &Inst,
  1813:                                                    bool Invert) const {
  1814:   if (Invert)
  1815:     return false;
  1816: 
  1817:   switch (Inst.getOpcode()) {
  1818:   // TODO: Add more instructions to be handled by MachineCombiner.
  1819:   case Hexagon::F2_sfadd:
  1820:     return Inst.getFlag(MachineInstr::MIFlag::FmReassoc);
  1821:   default:
  1822:     return false;
  1823:   }
  1824: }
  1825: 
  1826: bool HexagonInstrInfo::isSchedulingBoundary(const MachineInstr &MI,
  1827:                                             const MachineBasicBlock *MBB,
  1828:                                             const MachineFunction &MF) const {
  1829:   // Debug info is never a scheduling boundary. It's necessary to be explicit
  1830:   // due to the special treatment of IT instructions below, otherwise a
  1831:   // dbg_value followed by an IT will result in the IT instruction being
  1832:   // considered a scheduling hazard, which is wrong. It should be the actual
  1833:   // instruction preceding the dbg_value instruction(s), just like it is
  1834:   // when debug info is not present.
  1835:   if (MI.isDebugInstr())
  1836:     return false;
  1837: 
  1838:   // Throwing call is a boundary.
  1839:   if (MI.isCall()) {
  1840:     // Don't mess around with no return calls.
  1841:     if (doesNotReturn(MI))
  1842:       return true;
  1843:     // If any of the block's successors is a landing pad, this could be a
  1844:     // throwing call.
  1845:     for (auto *I : MBB->successors())
  1846:       if (I->isEHPad())
  1847:         return true;
  1848:   }
  1849: 
  1850:   // Terminators and labels can't be scheduled around.
  1851:   if (MI.getDesc().isTerminator() || MI.isPosition())
  1852:     return true;
  1853: 
  1854:   // INLINEASM_BR can jump to another block
  1855:   if (MI.getOpcode() == TargetOpcode::INLINEASM_BR)
  1856:     return true;
  1857: 
  1858:   if (MI.isInlineAsm() && !ScheduleInlineAsm)
  1859:     return true;
  1860: 
  1861:   return false;
  1862: }
  1863: 
  1864: /// Measure the specified inline asm to determine an approximation of its
  1865: /// length.
  1866: /// Comments (which run till the next SeparatorString or newline) do not
  1867: /// count as an instruction.
  1868: /// Any other non-whitespace text is considered an instruction, with
  1869: /// multiple instructions separated by SeparatorString or newlines.
  1870: /// Variable-length instructions are not handled here; this function
  1871: /// may be overloaded in the target code to do that.
  1872: /// Hexagon counts the number of ##'s and adjust for that many
  1873: /// constant exenders.
  1874: unsigned HexagonInstrInfo::getInlineAsmLength(const char *Str,
  1875:                                               const MCAsmInfo &MAI,
  1876:                                               const TargetSubtargetInfo *STI) const {
  1877:   StringRef AStr(Str);
  1878:   // Count the number of instructions in the asm.
  1879:   bool atInsnStart = true;
  1880:   unsigned Length = 0;
  1881:   const unsigned MaxInstLength = MAI.getMaxInstLength(STI);
  1882:   for (; *Str; ++Str) {
  1883:     if (*Str == '\n' || strncmp(Str, MAI.getSeparatorString(),
  1884:                                 strlen(MAI.getSeparatorString())) == 0)
  1885:       atInsnStart = true;
  1886:     if (atInsnStart && !isSpace(static_cast<unsigned char>(*Str))) {
  1887:       Length += MaxInstLength;
  1888:       atInsnStart = false;
  1889:     }
  1890:     if (atInsnStart && strncmp(Str, MAI.getCommentString().data(),
  1891:                                MAI.getCommentString().size()) == 0)
  1892:       atInsnStart = false;
  1893:   }
  1894: 
  1895:   // Add to size number of constant extenders seen * 4.
  1896:   StringRef Occ("##");
  1897:   Length += AStr.count(Occ)*4;
  1898:   return Length;
  1899: }
  1900: 
  1901: ScheduleHazardRecognizer*
  1902: HexagonInstrInfo::CreateTargetPostRAHazardRecognizer(
  1903:       const InstrItineraryData *II, const ScheduleDAG *DAG) const {
  1904:   if (UseDFAHazardRec)
  1905:     return new HexagonHazardRecognizer(II, this, Subtarget);
  1906:   return TargetInstrInfo::CreateTargetPostRAHazardRecognizer(II, DAG);
  1907: }
  1908: 
  1909: /// For a comparison instruction, return the source registers in
  1910: /// \p SrcReg and \p SrcReg2 if having two register operands, and the value it
  1911: /// compares against in CmpValue. Return true if the comparison instruction
  1912: /// can be analyzed.
  1913: bool HexagonInstrInfo::analyzeCompare(const MachineInstr &MI, Register &SrcReg,
  1914:                                       Register &SrcReg2, int64_t &Mask,
  1915:                                       int64_t &Value) const {
  1916:   unsigned Opc = MI.getOpcode();
  1917: 
  1918:   // Set mask and the first source register.
  1919:   switch (Opc) {
  1920:     case Hexagon::C2_cmpeq:
  1921:     case Hexagon::C2_cmpeqp:
  1922:     case Hexagon::C2_cmpgt:
  1923:     case Hexagon::C2_cmpgtp:
  1924:     case Hexagon::C2_cmpgtu:
  1925:     case Hexagon::C2_cmpgtup:
  1926:     case Hexagon::C4_cmpneq:
  1927:     case Hexagon::C4_cmplte:
  1928:     case Hexagon::C4_cmplteu:
  1929:     case Hexagon::C2_cmpeqi:
  1930:     case Hexagon::C2_cmpgti:
  1931:     case Hexagon::C2_cmpgtui:
  1932:     case Hexagon::C4_cmpneqi:
  1933:     case Hexagon::C4_cmplteui:
  1934:     case Hexagon::C4_cmpltei:
  1935:       SrcReg = MI.getOperand(1).getReg();
  1936:       Mask = ~0;
  1937:       break;
  1938:     case Hexagon::A4_cmpbeq:
  1939:     case Hexagon::A4_cmpbgt:
  1940:     case Hexagon::A4_cmpbgtu:
  1941:     case Hexagon::A4_cmpbeqi:
  1942:     case Hexagon::A4_cmpbgti:
  1943:     case Hexagon::A4_cmpbgtui:
  1944:       SrcReg = MI.getOperand(1).getReg();
  1945:       Mask = 0xFF;
  1946:       break;
  1947:     case Hexagon::A4_cmpheq:
  1948:     case Hexagon::A4_cmphgt:
  1949:     case Hexagon::A4_cmphgtu:
  1950:     case Hexagon::A4_cmpheqi:
  1951:     case Hexagon::A4_cmphgti:
  1952:     case Hexagon::A4_cmphgtui:
  1953:       SrcReg = MI.getOperand(1).getReg();
  1954:       Mask = 0xFFFF;
  1955:       break;
  1956:   }
  1957: 
  1958:   // Set the value/second source register.
  1959:   switch (Opc) {
  1960:     case Hexagon::C2_cmpeq:
  1961:     case Hexagon::C2_cmpeqp:
  1962:     case Hexagon::C2_cmpgt:
  1963:     case Hexagon::C2_cmpgtp:
  1964:     case Hexagon::C2_cmpgtu:
  1965:     case Hexagon::C2_cmpgtup:
  1966:     case Hexagon::A4_cmpbeq:
  1967:     case Hexagon::A4_cmpbgt:
  1968:     case Hexagon::A4_cmpbgtu:
  1969:     case Hexagon::A4_cmpheq:
  1970:     case Hexagon::A4_cmphgt:
  1971:     case Hexagon::A4_cmphgtu:
  1972:     case Hexagon::C4_cmpneq:
  1973:     case Hexagon::C4_cmplte:
  1974:     case Hexagon::C4_cmplteu:
  1975:       SrcReg2 = MI.getOperand(2).getReg();
  1976:       Value = 0;
  1977:       return true;
  1978: 
  1979:     case Hexagon::C2_cmpeqi:
  1980:     case Hexagon::C2_cmpgtui:
  1981:     case Hexagon::C2_cmpgti:
  1982:     case Hexagon::C4_cmpneqi:
  1983:     case Hexagon::C4_cmplteui:
  1984:     case Hexagon::C4_cmpltei:
  1985:     case Hexagon::A4_cmpbeqi:
  1986:     case Hexagon::A4_cmpbgti:
  1987:     case Hexagon::A4_cmpbgtui:
  1988:     case Hexagon::A4_cmpheqi:
  1989:     case Hexagon::A4_cmphgti:
  1990:     case Hexagon::A4_cmphgtui: {
  1991:       SrcReg2 = 0;
  1992:       const MachineOperand &Op2 = MI.getOperand(2);
  1993:       if (!Op2.isImm())
  1994:         return false;
  1995:       Value = MI.getOperand(2).getImm();
  1996:       return true;
  1997:     }
  1998:   }
  1999: 
  2000:   return false;
```
- EN: It declares or implements routines such as HexagonInstrInfo::isAssociativeAndCommutative, getFlag, HexagonInstrInfo::isSchedulingBoundary, HexagonInstrInfo::getInlineAsmLength, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstrInfo, HexagonHazardRecognizer, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonInstrInfo::isAssociativeAndCommutative, getFlag, HexagonInstrInfo::isSchedulingBoundary, HexagonInstrInfo::getInlineAsmLength, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo, HexagonHazardRecognizer，说明了它与同级后端组件的连接关系。

### Lines 2001-2200 / 第 2001-2200 行

```cpp
  2001: }
  2002: 
  2003: unsigned HexagonInstrInfo::getInstrLatency(const InstrItineraryData *ItinData,
  2004:                                            const MachineInstr &MI,
  2005:                                            unsigned *PredCost) const {
  2006:   return getInstrTimingClassLatency(ItinData, MI);
  2007: }
  2008: 
  2009: DFAPacketizer *HexagonInstrInfo::CreateTargetScheduleState(
  2010:     const TargetSubtargetInfo &STI) const {
  2011:   const InstrItineraryData *II = STI.getInstrItineraryData();
  2012:   return static_cast<const HexagonSubtarget&>(STI).createDFAPacketizer(II);
  2013: }
  2014: 
  2015: // Inspired by this pair:
  2016: //  %r13 = L2_loadri_io %r29, 136; mem:LD4[FixedStack0]
  2017: //  S2_storeri_io %r29, 132, killed %r1; flags:  mem:ST4[FixedStack1]
  2018: // Currently AA considers the addresses in these instructions to be aliasing.
  2019: bool HexagonInstrInfo::areMemAccessesTriviallyDisjoint(
  2020:     const MachineInstr &MIa, const MachineInstr &MIb) const {
  2021:   if (MIa.hasUnmodeledSideEffects() || MIb.hasUnmodeledSideEffects() ||
  2022:       MIa.hasOrderedMemoryRef() || MIb.hasOrderedMemoryRef())
  2023:     return false;
  2024: 
  2025:   // Instructions that are pure loads, not loads and stores like memops are not
  2026:   // dependent.
  2027:   if (MIa.mayLoad() && !isMemOp(MIa) && MIb.mayLoad() && !isMemOp(MIb))
  2028:     return true;
  2029: 
  2030:   // Get the base register in MIa.
  2031:   unsigned BasePosA, OffsetPosA;
  2032:   if (!getBaseAndOffsetPosition(MIa, BasePosA, OffsetPosA))
  2033:     return false;
  2034:   const MachineOperand &BaseA = MIa.getOperand(BasePosA);
  2035:   Register BaseRegA = BaseA.getReg();
  2036:   unsigned BaseSubA = BaseA.getSubReg();
  2037: 
  2038:   // Get the base register in MIb.
  2039:   unsigned BasePosB, OffsetPosB;
  2040:   if (!getBaseAndOffsetPosition(MIb, BasePosB, OffsetPosB))
  2041:     return false;
  2042:   const MachineOperand &BaseB = MIb.getOperand(BasePosB);
  2043:   Register BaseRegB = BaseB.getReg();
  2044:   unsigned BaseSubB = BaseB.getSubReg();
  2045: 
  2046:   if (BaseRegA != BaseRegB || BaseSubA != BaseSubB)
  2047:     return false;
  2048: 
  2049:   // Get the access sizes.
  2050:   unsigned SizeA = getMemAccessSize(MIa);
  2051:   unsigned SizeB = getMemAccessSize(MIb);
  2052: 
  2053:   // Get the offsets. Handle immediates only for now.
  2054:   const MachineOperand &OffA = MIa.getOperand(OffsetPosA);
  2055:   const MachineOperand &OffB = MIb.getOperand(OffsetPosB);
  2056:   if (!MIa.getOperand(OffsetPosA).isImm() ||
  2057:       !MIb.getOperand(OffsetPosB).isImm())
  2058:     return false;
  2059:   int OffsetA = isPostIncrement(MIa) ? 0 : OffA.getImm();
  2060:   int OffsetB = isPostIncrement(MIb) ? 0 : OffB.getImm();
  2061: 
  2062:   // This is a mem access with the same base register and known offsets from it.
  2063:   // Reason about it.
  2064:   if (OffsetA > OffsetB) {
  2065:     uint64_t OffDiff = (uint64_t)((int64_t)OffsetA - (int64_t)OffsetB);
  2066:     return SizeB <= OffDiff;
  2067:   }
  2068:   if (OffsetA < OffsetB) {
  2069:     uint64_t OffDiff = (uint64_t)((int64_t)OffsetB - (int64_t)OffsetA);
  2070:     return SizeA <= OffDiff;
  2071:   }
  2072: 
  2073:   return false;
  2074: }
  2075: 
  2076: /// If the instruction is an increment of a constant value, return the amount.
  2077: bool HexagonInstrInfo::getIncrementValue(const MachineInstr &MI,
  2078:       int &Value) const {
  2079:   if (isPostIncrement(MI)) {
  2080:     unsigned BasePos = 0, OffsetPos = 0;
  2081:     if (!getBaseAndOffsetPosition(MI, BasePos, OffsetPos))
  2082:       return false;
  2083:     const MachineOperand &OffsetOp = MI.getOperand(OffsetPos);
  2084:     if (OffsetOp.isImm()) {
  2085:       Value = OffsetOp.getImm();
  2086:       return true;
  2087:     }
  2088:   } else if (MI.getOpcode() == Hexagon::A2_addi) {
  2089:     const MachineOperand &AddOp = MI.getOperand(2);
  2090:     if (AddOp.isImm()) {
  2091:       Value = AddOp.getImm();
  2092:       return true;
  2093:     }
  2094:   }
  2095: 
  2096:   return false;
  2097: }
  2098: 
  2099: std::pair<unsigned, unsigned>
  2100: HexagonInstrInfo::decomposeMachineOperandsTargetFlags(unsigned TF) const {
  2101:   return std::make_pair(TF & ~HexagonII::MO_Bitmasks,
  2102:                         TF & HexagonII::MO_Bitmasks);
  2103: }
  2104: 
  2105: ArrayRef<std::pair<unsigned, const char*>>
  2106: HexagonInstrInfo::getSerializableDirectMachineOperandTargetFlags() const {
  2107:   using namespace HexagonII;
  2108: 
  2109:   static const std::pair<unsigned, const char*> Flags[] = {
  2110:     {MO_PCREL,  "hexagon-pcrel"},
  2111:     {MO_GOT,    "hexagon-got"},
  2112:     {MO_LO16,   "hexagon-lo16"},
  2113:     {MO_HI16,   "hexagon-hi16"},
  2114:     {MO_GPREL,  "hexagon-gprel"},
  2115:     {MO_GDGOT,  "hexagon-gdgot"},
  2116:     {MO_GDPLT,  "hexagon-gdplt"},
  2117:     {MO_IE,     "hexagon-ie"},
  2118:     {MO_IEGOT,  "hexagon-iegot"},
  2119:     {MO_TPREL,  "hexagon-tprel"}
  2120:   };
  2121:   return ArrayRef(Flags);
  2122: }
  2123: 
  2124: ArrayRef<std::pair<unsigned, const char*>>
  2125: HexagonInstrInfo::getSerializableBitmaskMachineOperandTargetFlags() const {
  2126:   using namespace HexagonII;
  2127: 
  2128:   static const std::pair<unsigned, const char*> Flags[] = {
  2129:     {HMOTF_ConstExtended, "hexagon-ext"}
  2130:   };
  2131:   return ArrayRef(Flags);
  2132: }
  2133: 
  2134: Register HexagonInstrInfo::createVR(MachineFunction *MF, MVT VT) const {
  2135:   MachineRegisterInfo &MRI = MF->getRegInfo();
  2136:   const TargetRegisterClass *TRC;
  2137:   if (VT == MVT::i1) {
  2138:     TRC = &Hexagon::PredRegsRegClass;
  2139:   } else if (VT == MVT::i32 || VT == MVT::f32) {
  2140:     TRC = &Hexagon::IntRegsRegClass;
  2141:   } else if (VT == MVT::i64 || VT == MVT::f64) {
  2142:     TRC = &Hexagon::DoubleRegsRegClass;
  2143:   } else {
  2144:     llvm_unreachable("Cannot handle this register class");
  2145:   }
  2146: 
  2147:   Register NewReg = MRI.createVirtualRegister(TRC);
  2148:   return NewReg;
  2149: }
  2150: 
  2151: bool HexagonInstrInfo::isAbsoluteSet(const MachineInstr &MI) const {
  2152:   return (getAddrMode(MI) == HexagonII::AbsoluteSet);
  2153: }
  2154: 
  2155: bool HexagonInstrInfo::isAccumulator(const MachineInstr &MI) const {
  2156:   const uint64_t F = MI.getDesc().TSFlags;
  2157:   return((F >> HexagonII::AccumulatorPos) & HexagonII::AccumulatorMask);
  2158: }
  2159: 
  2160: bool HexagonInstrInfo::isBaseImmOffset(const MachineInstr &MI) const {
  2161:   return getAddrMode(MI) == HexagonII::BaseImmOffset;
  2162: }
  2163: 
  2164: bool HexagonInstrInfo::isComplex(const MachineInstr &MI) const {
  2165:   return !isTC1(MI) && !isTC2Early(MI) && !MI.getDesc().mayLoad() &&
  2166:          !MI.getDesc().mayStore() &&
  2167:          MI.getDesc().getOpcode() != Hexagon::S2_allocframe &&
  2168:          MI.getDesc().getOpcode() != Hexagon::L2_deallocframe &&
  2169:          !isMemOp(MI) && !MI.isBranch() && !MI.isReturn() && !MI.isCall();
  2170: }
  2171: 
  2172: // Return true if the instruction is a compound branch instruction.
  2173: bool HexagonInstrInfo::isCompoundBranchInstr(const MachineInstr &MI) const {
  2174:   return getType(MI) == HexagonII::TypeCJ && MI.isBranch();
  2175: }
  2176: 
  2177: // TODO: In order to have isExtendable for fpimm/f32Ext, we need to handle
  2178: // isFPImm and later getFPImm as well.
  2179: bool HexagonInstrInfo::isConstExtended(const MachineInstr &MI) const {
  2180:   const uint64_t F = MI.getDesc().TSFlags;
  2181:   unsigned isExtended = (F >> HexagonII::ExtendedPos) & HexagonII::ExtendedMask;
  2182:   if (isExtended) // Instruction must be extended.
  2183:     return true;
  2184: 
  2185:   unsigned isExtendable =
  2186:     (F >> HexagonII::ExtendablePos) & HexagonII::ExtendableMask;
  2187:   if (!isExtendable)
  2188:     return false;
  2189: 
  2190:   if (MI.isCall())
  2191:     return false;
  2192: 
  2193:   short ExtOpNum = getCExtOpNum(MI);
  2194:   const MachineOperand &MO = MI.getOperand(ExtOpNum);
  2195:   // Use MO operand flags to determine if MO
  2196:   // has the HMOTF_ConstExtended flag set.
  2197:   if (MO.getTargetFlags() & HexagonII::HMOTF_ConstExtended)
  2198:     return true;
  2199:   // If this is a Machine BB address we are talking about, and it is
  2200:   // not marked as extended, say so.
```
- EN: It opens namespaces (HexagonII) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as HexagonInstrInfo::getInstrLatency, getInstrTimingClassLatency, HexagonInstrInfo::CreateTargetScheduleState, getInstrItineraryData, ... (31 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstrInfo, HexagonSubtarget, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（HexagonII），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 HexagonInstrInfo::getInstrLatency, getInstrTimingClassLatency, HexagonInstrInfo::CreateTargetScheduleState, getInstrItineraryData, ... (31 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo, HexagonSubtarget, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 2201-2400 / 第 2201-2400 行

```cpp
  2201:   if (MO.isMBB())
  2202:     return false;
  2203: 
  2204:   // We could be using an instruction with an extendable immediate and shoehorn
  2205:   // a global address into it. If it is a global address it will be constant
  2206:   // extended. We do this for COMBINE.
  2207:   if (MO.isGlobal() || MO.isSymbol() || MO.isBlockAddress() ||
  2208:       MO.isJTI() || MO.isCPI() || MO.isFPImm())
  2209:     return true;
  2210: 
  2211:   // If the extendable operand is not 'Immediate' type, the instruction should
  2212:   // have 'isExtended' flag set.
  2213:   assert(MO.isImm() && "Extendable operand must be Immediate type");
  2214: 
  2215:   int64_t Value = MO.getImm();
  2216:   if ((F >> HexagonII::ExtentSignedPos) & HexagonII::ExtentSignedMask) {
  2217:     int32_t SValue = Value;
  2218:     int32_t MinValue = getMinValue(MI);
  2219:     int32_t MaxValue = getMaxValue(MI);
  2220:     return SValue < MinValue || SValue > MaxValue;
  2221:   }
  2222:   uint32_t UValue = Value;
  2223:   uint32_t MinValue = getMinValue(MI);
  2224:   uint32_t MaxValue = getMaxValue(MI);
  2225:   return UValue < MinValue || UValue > MaxValue;
  2226: }
  2227: 
  2228: bool HexagonInstrInfo::isDeallocRet(const MachineInstr &MI) const {
  2229:   switch (MI.getOpcode()) {
  2230:   case Hexagon::L4_return:
  2231:   case Hexagon::L4_return_t:
  2232:   case Hexagon::L4_return_f:
  2233:   case Hexagon::L4_return_tnew_pnt:
  2234:   case Hexagon::L4_return_fnew_pnt:
  2235:   case Hexagon::L4_return_tnew_pt:
  2236:   case Hexagon::L4_return_fnew_pt:
  2237:     return true;
  2238:   }
  2239:   return false;
  2240: }
  2241: 
  2242: // Return true when ConsMI uses a register defined by ProdMI.
  2243: bool HexagonInstrInfo::isDependent(const MachineInstr &ProdMI,
  2244:       const MachineInstr &ConsMI) const {
  2245:   if (!ProdMI.getDesc().getNumDefs())
  2246:     return false;
  2247:   const HexagonRegisterInfo &HRI = *Subtarget.getRegisterInfo();
  2248: 
  2249:   SmallVector<Register, 4> DefsA;
  2250:   SmallVector<Register, 4> DefsB;
  2251:   SmallVector<Register, 8> UsesA;
  2252:   SmallVector<Register, 8> UsesB;
  2253: 
  2254:   parseOperands(ProdMI, DefsA, UsesA);
  2255:   parseOperands(ConsMI, DefsB, UsesB);
  2256: 
  2257:   for (auto &RegA : DefsA)
  2258:     for (auto &RegB : UsesB) {
  2259:       // True data dependency.
  2260:       if (RegA == RegB)
  2261:         return true;
  2262: 
  2263:       if (RegA.isPhysical() && llvm::is_contained(HRI.subregs(RegA), RegB))
  2264:         return true;
  2265: 
  2266:       if (RegB.isPhysical() && llvm::is_contained(HRI.subregs(RegB), RegA))
  2267:         return true;
  2268:     }
  2269: 
  2270:   return false;
  2271: }
  2272: 
  2273: // Returns true if the instruction is already a .cur.
  2274: bool HexagonInstrInfo::isDotCurInst(const MachineInstr &MI) const {
  2275:   switch (MI.getOpcode()) {
  2276:   case Hexagon::V6_vL32b_cur_pi:
  2277:   case Hexagon::V6_vL32b_cur_ai:
  2278:     return true;
  2279:   }
  2280:   return false;
  2281: }
  2282: 
  2283: // Returns true, if any one of the operands is a dot new
  2284: // insn, whether it is predicated dot new or register dot new.
  2285: bool HexagonInstrInfo::isDotNewInst(const MachineInstr &MI) const {
  2286:   if (isNewValueInst(MI) || (isPredicated(MI) && isPredicatedNew(MI)))
  2287:     return true;
  2288: 
  2289:   return false;
  2290: }
  2291: 
  2292: /// Symmetrical. See if these two instructions are fit for duplex pair.
  2293: bool HexagonInstrInfo::isDuplexPair(const MachineInstr &MIa,
  2294:       const MachineInstr &MIb) const {
  2295:   HexagonII::SubInstructionGroup MIaG = getDuplexCandidateGroup(MIa);
  2296:   HexagonII::SubInstructionGroup MIbG = getDuplexCandidateGroup(MIb);
  2297:   return (isDuplexPairMatch(MIaG, MIbG) || isDuplexPairMatch(MIbG, MIaG));
  2298: }
  2299: 
  2300: bool HexagonInstrInfo::isEndLoopN(unsigned Opcode) const {
  2301:   return (Opcode == Hexagon::ENDLOOP0 ||
  2302:           Opcode == Hexagon::ENDLOOP1);
  2303: }
  2304: 
  2305: bool HexagonInstrInfo::isExpr(unsigned OpType) const {
  2306:   switch(OpType) {
  2307:   case MachineOperand::MO_MachineBasicBlock:
  2308:   case MachineOperand::MO_GlobalAddress:
  2309:   case MachineOperand::MO_ExternalSymbol:
  2310:   case MachineOperand::MO_JumpTableIndex:
  2311:   case MachineOperand::MO_ConstantPoolIndex:
  2312:   case MachineOperand::MO_BlockAddress:
  2313:     return true;
  2314:   default:
  2315:     return false;
  2316:   }
  2317: }
  2318: 
  2319: bool HexagonInstrInfo::isExtendable(const MachineInstr &MI) const {
  2320:   const MCInstrDesc &MID = MI.getDesc();
  2321:   const uint64_t F = MID.TSFlags;
  2322:   if ((F >> HexagonII::ExtendablePos) & HexagonII::ExtendableMask)
  2323:     return true;
  2324: 
  2325:   // TODO: This is largely obsolete now. Will need to be removed
  2326:   // in consecutive patches.
  2327:   switch (MI.getOpcode()) {
  2328:     // PS_fi and PS_fia remain special cases.
  2329:     case Hexagon::PS_fi:
  2330:     case Hexagon::PS_fia:
  2331:       return true;
  2332:     default:
  2333:       return false;
  2334:   }
  2335:   return  false;
  2336: }
  2337: 
  2338: // This returns true in two cases:
  2339: // - The OP code itself indicates that this is an extended instruction.
  2340: // - One of MOs has been marked with HMOTF_ConstExtended flag.
  2341: bool HexagonInstrInfo::isExtended(const MachineInstr &MI) const {
  2342:   // First check if this is permanently extended op code.
  2343:   const uint64_t F = MI.getDesc().TSFlags;
  2344:   if ((F >> HexagonII::ExtendedPos) & HexagonII::ExtendedMask)
  2345:     return true;
  2346:   // Use MO operand flags to determine if one of MI's operands
  2347:   // has HMOTF_ConstExtended flag set.
  2348:   for (const MachineOperand &MO : MI.operands())
  2349:     if (MO.getTargetFlags() & HexagonII::HMOTF_ConstExtended)
  2350:       return true;
  2351:   return  false;
  2352: }
  2353: 
  2354: bool HexagonInstrInfo::isFloat(const MachineInstr &MI) const {
  2355:   unsigned Opcode = MI.getOpcode();
  2356:   const uint64_t F = get(Opcode).TSFlags;
  2357:   return (F >> HexagonII::FPPos) & HexagonII::FPMask;
  2358: }
  2359: 
  2360: // No V60 HVX VMEM with A_INDIRECT.
  2361: bool HexagonInstrInfo::isHVXMemWithAIndirect(const MachineInstr &I,
  2362:       const MachineInstr &J) const {
  2363:   if (!isHVXVec(I))
  2364:     return false;
  2365:   if (!I.mayLoad() && !I.mayStore())
  2366:     return false;
  2367:   return J.isIndirectBranch() || isIndirectCall(J) || isIndirectL4Return(J);
  2368: }
  2369: 
  2370: bool HexagonInstrInfo::isIndirectCall(const MachineInstr &MI) const {
  2371:   switch (MI.getOpcode()) {
  2372:   case Hexagon::J2_callr:
  2373:   case Hexagon::J2_callrf:
  2374:   case Hexagon::J2_callrt:
  2375:   case Hexagon::PS_call_nr:
  2376:     return true;
  2377:   }
  2378:   return false;
  2379: }
  2380: 
  2381: bool HexagonInstrInfo::isIndirectL4Return(const MachineInstr &MI) const {
  2382:   switch (MI.getOpcode()) {
  2383:   case Hexagon::L4_return:
  2384:   case Hexagon::L4_return_t:
  2385:   case Hexagon::L4_return_f:
  2386:   case Hexagon::L4_return_fnew_pnt:
  2387:   case Hexagon::L4_return_fnew_pt:
  2388:   case Hexagon::L4_return_tnew_pnt:
  2389:   case Hexagon::L4_return_tnew_pt:
  2390:     return true;
  2391:   }
  2392:   return false;
  2393: }
  2394: 
  2395: bool HexagonInstrInfo::isJumpR(const MachineInstr &MI) const {
  2396:   switch (MI.getOpcode()) {
  2397:   case Hexagon::J2_jumpr:
  2398:   case Hexagon::J2_jumprt:
  2399:   case Hexagon::J2_jumprf:
  2400:   case Hexagon::J2_jumprtnewpt:
```
- EN: It declares or implements routines such as assert, getImm, getMinValue, getMaxValue, ... (24 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonII, HexagonInstrInfo, HexagonRegisterInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 assert, getImm, getMinValue, getMaxValue, ... (24 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonII, HexagonInstrInfo, HexagonRegisterInfo，说明了它与同级后端组件的连接关系。

### Lines 2401-2600 / 第 2401-2600 行

```cpp
  2401:   case Hexagon::J2_jumprfnewpt:
  2402:   case Hexagon::J2_jumprtnew:
  2403:   case Hexagon::J2_jumprfnew:
  2404:     return true;
  2405:   }
  2406:   return false;
  2407: }
  2408: 
  2409: // Return true if a given MI can accommodate given offset.
  2410: // Use abs estimate as oppose to the exact number.
  2411: // TODO: This will need to be changed to use MC level
  2412: // definition of instruction extendable field size.
  2413: bool HexagonInstrInfo::isJumpWithinBranchRange(const MachineInstr &MI,
  2414:       unsigned offset) const {
  2415:   // This selection of jump instructions matches to that what
  2416:   // analyzeBranch can parse, plus NVJ.
  2417:   if (isNewValueJump(MI)) // r9:2
  2418:     return isInt<11>(offset);
  2419: 
  2420:   switch (MI.getOpcode()) {
  2421:   // Still missing Jump to address condition on register value.
  2422:   default:
  2423:     return false;
  2424:   case Hexagon::J2_jump: // bits<24> dst; // r22:2
  2425:   case Hexagon::J2_call:
  2426:   case Hexagon::PS_call_nr:
  2427:     return isInt<24>(offset);
  2428:   case Hexagon::J2_jumpt: //bits<17> dst; // r15:2
  2429:   case Hexagon::J2_jumpf:
  2430:   case Hexagon::J2_jumptnew:
  2431:   case Hexagon::J2_jumptnewpt:
  2432:   case Hexagon::J2_jumpfnew:
  2433:   case Hexagon::J2_jumpfnewpt:
  2434:   case Hexagon::J2_callt:
  2435:   case Hexagon::J2_callf:
  2436:     return isInt<17>(offset);
  2437:   case Hexagon::J2_loop0i:
  2438:   case Hexagon::J2_loop0iext:
  2439:   case Hexagon::J2_loop0r:
  2440:   case Hexagon::J2_loop0rext:
  2441:   case Hexagon::J2_loop1i:
  2442:   case Hexagon::J2_loop1iext:
  2443:   case Hexagon::J2_loop1r:
  2444:   case Hexagon::J2_loop1rext:
  2445:     return isInt<9>(offset);
  2446:   // TODO: Add all the compound branches here. Can we do this in Relation model?
  2447:   case Hexagon::J4_cmpeqi_tp0_jump_nt:
  2448:   case Hexagon::J4_cmpeqi_tp1_jump_nt:
  2449:   case Hexagon::J4_cmpeqn1_tp0_jump_nt:
  2450:   case Hexagon::J4_cmpeqn1_tp1_jump_nt:
  2451:     return isInt<11>(offset);
  2452:   }
  2453: }
  2454: 
  2455: bool HexagonInstrInfo::isLateSourceInstr(const MachineInstr &MI) const {
  2456:   // Instructions with iclass A_CVI_VX and attribute A_CVI_LATE uses a multiply
  2457:   // resource, but all operands can be received late like an ALU instruction.
  2458:   return getType(MI) == HexagonII::TypeCVI_VX_LATE;
  2459: }
  2460: 
  2461: bool HexagonInstrInfo::isLoopN(const MachineInstr &MI) const {
  2462:   unsigned Opcode = MI.getOpcode();
  2463:   return Opcode == Hexagon::J2_loop0i    ||
  2464:          Opcode == Hexagon::J2_loop0r    ||
  2465:          Opcode == Hexagon::J2_loop0iext ||
  2466:          Opcode == Hexagon::J2_loop0rext ||
  2467:          Opcode == Hexagon::J2_loop1i    ||
  2468:          Opcode == Hexagon::J2_loop1r    ||
  2469:          Opcode == Hexagon::J2_loop1iext ||
  2470:          Opcode == Hexagon::J2_loop1rext;
  2471: }
  2472: 
  2473: bool HexagonInstrInfo::isMemOp(const MachineInstr &MI) const {
  2474:   switch (MI.getOpcode()) {
  2475:     default: return false;
  2476:     case Hexagon::L4_iadd_memopw_io:
  2477:     case Hexagon::L4_isub_memopw_io:
  2478:     case Hexagon::L4_add_memopw_io:
  2479:     case Hexagon::L4_sub_memopw_io:
  2480:     case Hexagon::L4_and_memopw_io:
  2481:     case Hexagon::L4_or_memopw_io:
  2482:     case Hexagon::L4_iadd_memoph_io:
  2483:     case Hexagon::L4_isub_memoph_io:
  2484:     case Hexagon::L4_add_memoph_io:
  2485:     case Hexagon::L4_sub_memoph_io:
  2486:     case Hexagon::L4_and_memoph_io:
  2487:     case Hexagon::L4_or_memoph_io:
  2488:     case Hexagon::L4_iadd_memopb_io:
  2489:     case Hexagon::L4_isub_memopb_io:
  2490:     case Hexagon::L4_add_memopb_io:
  2491:     case Hexagon::L4_sub_memopb_io:
  2492:     case Hexagon::L4_and_memopb_io:
  2493:     case Hexagon::L4_or_memopb_io:
  2494:     case Hexagon::L4_ior_memopb_io:
  2495:     case Hexagon::L4_ior_memoph_io:
  2496:     case Hexagon::L4_ior_memopw_io:
  2497:     case Hexagon::L4_iand_memopb_io:
  2498:     case Hexagon::L4_iand_memoph_io:
  2499:     case Hexagon::L4_iand_memopw_io:
  2500:     return true;
  2501:   }
  2502:   return false;
  2503: }
  2504: 
  2505: bool HexagonInstrInfo::isNewValue(const MachineInstr &MI) const {
  2506:   const uint64_t F = MI.getDesc().TSFlags;
  2507:   return (F >> HexagonII::NewValuePos) & HexagonII::NewValueMask;
  2508: }
  2509: 
  2510: bool HexagonInstrInfo::isNewValue(unsigned Opcode) const {
  2511:   const uint64_t F = get(Opcode).TSFlags;
  2512:   return (F >> HexagonII::NewValuePos) & HexagonII::NewValueMask;
  2513: }
  2514: 
  2515: bool HexagonInstrInfo::isNewValueInst(const MachineInstr &MI) const {
  2516:   return isNewValueJump(MI) || isNewValueStore(MI);
  2517: }
  2518: 
  2519: bool HexagonInstrInfo::isNewValueJump(const MachineInstr &MI) const {
  2520:   return isNewValue(MI) && MI.isBranch();
  2521: }
  2522: 
  2523: bool HexagonInstrInfo::isNewValueJump(unsigned Opcode) const {
  2524:   return isNewValue(Opcode) && get(Opcode).isBranch() && isPredicated(Opcode);
  2525: }
  2526: 
  2527: bool HexagonInstrInfo::isNewValueStore(const MachineInstr &MI) const {
  2528:   const uint64_t F = MI.getDesc().TSFlags;
  2529:   return (F >> HexagonII::NVStorePos) & HexagonII::NVStoreMask;
  2530: }
  2531: 
  2532: bool HexagonInstrInfo::isNewValueStore(unsigned Opcode) const {
  2533:   const uint64_t F = get(Opcode).TSFlags;
  2534:   return (F >> HexagonII::NVStorePos) & HexagonII::NVStoreMask;
  2535: }
  2536: 
  2537: // Returns true if a particular operand is extendable for an instruction.
  2538: bool HexagonInstrInfo::isOperandExtended(const MachineInstr &MI,
  2539:     unsigned OperandNum) const {
  2540:   const uint64_t F = MI.getDesc().TSFlags;
  2541:   return ((F >> HexagonII::ExtendableOpPos) & HexagonII::ExtendableOpMask)
  2542:           == OperandNum;
  2543: }
  2544: 
  2545: bool HexagonInstrInfo::isPredicatedNew(const MachineInstr &MI) const {
  2546:   const uint64_t F = MI.getDesc().TSFlags;
  2547:   assert(isPredicated(MI));
  2548:   return (F >> HexagonII::PredicatedNewPos) & HexagonII::PredicatedNewMask;
  2549: }
  2550: 
  2551: bool HexagonInstrInfo::isPredicatedNew(unsigned Opcode) const {
  2552:   const uint64_t F = get(Opcode).TSFlags;
  2553:   assert(isPredicated(Opcode));
  2554:   return (F >> HexagonII::PredicatedNewPos) & HexagonII::PredicatedNewMask;
  2555: }
  2556: 
  2557: bool HexagonInstrInfo::isPredicatedTrue(const MachineInstr &MI) const {
  2558:   const uint64_t F = MI.getDesc().TSFlags;
  2559:   return !((F >> HexagonII::PredicatedFalsePos) &
  2560:            HexagonII::PredicatedFalseMask);
  2561: }
  2562: 
  2563: bool HexagonInstrInfo::isPredicatedTrue(unsigned Opcode) const {
  2564:   const uint64_t F = get(Opcode).TSFlags;
  2565:   // Make sure that the instruction is predicated.
  2566:   assert((F>> HexagonII::PredicatedPos) & HexagonII::PredicatedMask);
  2567:   return !((F >> HexagonII::PredicatedFalsePos) &
  2568:            HexagonII::PredicatedFalseMask);
  2569: }
  2570: 
  2571: bool HexagonInstrInfo::isPredicated(unsigned Opcode) const {
  2572:   const uint64_t F = get(Opcode).TSFlags;
  2573:   return (F >> HexagonII::PredicatedPos) & HexagonII::PredicatedMask;
  2574: }
  2575: 
  2576: bool HexagonInstrInfo::isPredicateLate(unsigned Opcode) const {
  2577:   const uint64_t F = get(Opcode).TSFlags;
  2578:   return (F >> HexagonII::PredicateLatePos) & HexagonII::PredicateLateMask;
  2579: }
  2580: 
  2581: bool HexagonInstrInfo::isPredictedTaken(unsigned Opcode) const {
  2582:   const uint64_t F = get(Opcode).TSFlags;
  2583:   assert(get(Opcode).isBranch() &&
  2584:          (isPredicatedNew(Opcode) || isNewValue(Opcode)));
  2585:   return (F >> HexagonII::TakenPos) & HexagonII::TakenMask;
  2586: }
  2587: 
  2588: bool HexagonInstrInfo::isSaveCalleeSavedRegsCall(const MachineInstr &MI) const {
  2589:   return MI.getOpcode() == Hexagon::SAVE_REGISTERS_CALL_V4 ||
  2590:          MI.getOpcode() == Hexagon::SAVE_REGISTERS_CALL_V4_EXT ||
  2591:          MI.getOpcode() == Hexagon::SAVE_REGISTERS_CALL_V4_PIC ||
  2592:          MI.getOpcode() == Hexagon::SAVE_REGISTERS_CALL_V4_EXT_PIC;
  2593: }
  2594: 
  2595: bool HexagonInstrInfo::isSignExtendingLoad(const MachineInstr &MI) const {
  2596:   switch (MI.getOpcode()) {
  2597:   // Byte
  2598:   case Hexagon::L2_loadrb_io:
  2599:   case Hexagon::L4_loadrb_ur:
  2600:   case Hexagon::L4_loadrb_ap:
```
- EN: It declares or implements routines such as HexagonInstrInfo::isJumpWithinBranchRange, isInt<24>, isInt<17>, isInt<9>, ... (24 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonInstrInfo::isJumpWithinBranchRange, isInt<24>, isInt<17>, isInt<9>, ... (24 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 2601-2800 / 第 2601-2800 行

```cpp
  2601:   case Hexagon::L2_loadrb_pr:
  2602:   case Hexagon::L2_loadrb_pbr:
  2603:   case Hexagon::L2_loadrb_pi:
  2604:   case Hexagon::L2_loadrb_pci:
  2605:   case Hexagon::L2_loadrb_pcr:
  2606:   case Hexagon::L2_loadbsw2_io:
  2607:   case Hexagon::L4_loadbsw2_ur:
  2608:   case Hexagon::L4_loadbsw2_ap:
  2609:   case Hexagon::L2_loadbsw2_pr:
  2610:   case Hexagon::L2_loadbsw2_pbr:
  2611:   case Hexagon::L2_loadbsw2_pi:
  2612:   case Hexagon::L2_loadbsw2_pci:
  2613:   case Hexagon::L2_loadbsw2_pcr:
  2614:   case Hexagon::L2_loadbsw4_io:
  2615:   case Hexagon::L4_loadbsw4_ur:
  2616:   case Hexagon::L4_loadbsw4_ap:
  2617:   case Hexagon::L2_loadbsw4_pr:
  2618:   case Hexagon::L2_loadbsw4_pbr:
  2619:   case Hexagon::L2_loadbsw4_pi:
  2620:   case Hexagon::L2_loadbsw4_pci:
  2621:   case Hexagon::L2_loadbsw4_pcr:
  2622:   case Hexagon::L4_loadrb_rr:
  2623:   case Hexagon::L2_ploadrbt_io:
  2624:   case Hexagon::L2_ploadrbt_pi:
  2625:   case Hexagon::L2_ploadrbf_io:
  2626:   case Hexagon::L2_ploadrbf_pi:
  2627:   case Hexagon::L2_ploadrbtnew_io:
  2628:   case Hexagon::L2_ploadrbfnew_io:
  2629:   case Hexagon::L4_ploadrbt_rr:
  2630:   case Hexagon::L4_ploadrbf_rr:
  2631:   case Hexagon::L4_ploadrbtnew_rr:
  2632:   case Hexagon::L4_ploadrbfnew_rr:
  2633:   case Hexagon::L2_ploadrbtnew_pi:
  2634:   case Hexagon::L2_ploadrbfnew_pi:
  2635:   case Hexagon::L4_ploadrbt_abs:
  2636:   case Hexagon::L4_ploadrbf_abs:
  2637:   case Hexagon::L4_ploadrbtnew_abs:
  2638:   case Hexagon::L4_ploadrbfnew_abs:
  2639:   case Hexagon::L2_loadrbgp:
  2640:   // Half
  2641:   case Hexagon::L2_loadrh_io:
  2642:   case Hexagon::L4_loadrh_ur:
  2643:   case Hexagon::L4_loadrh_ap:
  2644:   case Hexagon::L2_loadrh_pr:
  2645:   case Hexagon::L2_loadrh_pbr:
  2646:   case Hexagon::L2_loadrh_pi:
  2647:   case Hexagon::L2_loadrh_pci:
  2648:   case Hexagon::L2_loadrh_pcr:
  2649:   case Hexagon::L4_loadrh_rr:
  2650:   case Hexagon::L2_ploadrht_io:
  2651:   case Hexagon::L2_ploadrht_pi:
  2652:   case Hexagon::L2_ploadrhf_io:
  2653:   case Hexagon::L2_ploadrhf_pi:
  2654:   case Hexagon::L2_ploadrhtnew_io:
  2655:   case Hexagon::L2_ploadrhfnew_io:
  2656:   case Hexagon::L4_ploadrht_rr:
  2657:   case Hexagon::L4_ploadrhf_rr:
  2658:   case Hexagon::L4_ploadrhtnew_rr:
  2659:   case Hexagon::L4_ploadrhfnew_rr:
  2660:   case Hexagon::L2_ploadrhtnew_pi:
  2661:   case Hexagon::L2_ploadrhfnew_pi:
  2662:   case Hexagon::L4_ploadrht_abs:
  2663:   case Hexagon::L4_ploadrhf_abs:
  2664:   case Hexagon::L4_ploadrhtnew_abs:
  2665:   case Hexagon::L4_ploadrhfnew_abs:
  2666:   case Hexagon::L2_loadrhgp:
  2667:     return true;
  2668:   default:
  2669:     return false;
  2670:   }
  2671: }
  2672: 
  2673: bool HexagonInstrInfo::isSolo(const MachineInstr &MI) const {
  2674:   const uint64_t F = MI.getDesc().TSFlags;
  2675:   return (F >> HexagonII::SoloPos) & HexagonII::SoloMask;
  2676: }
  2677: 
  2678: bool HexagonInstrInfo::isSpillPredRegOp(const MachineInstr &MI) const {
  2679:   switch (MI.getOpcode()) {
  2680:   case Hexagon::STriw_pred:
  2681:   case Hexagon::LDriw_pred:
  2682:     return true;
  2683:   default:
  2684:     return false;
  2685:   }
  2686: }
  2687: 
  2688: bool HexagonInstrInfo::isTailCall(const MachineInstr &MI) const {
  2689:   if (!MI.isBranch())
  2690:     return false;
  2691: 
  2692:   for (auto &Op : MI.operands())
  2693:     if (Op.isGlobal() || Op.isSymbol())
  2694:       return true;
  2695:   return false;
  2696: }
  2697: 
  2698: // Returns true when SU has a timing class TC1.
  2699: bool HexagonInstrInfo::isTC1(const MachineInstr &MI) const {
  2700:   unsigned SchedClass = MI.getDesc().getSchedClass();
  2701:   return is_TC1(SchedClass);
  2702: }
  2703: 
  2704: bool HexagonInstrInfo::isTC2(const MachineInstr &MI) const {
  2705:   unsigned SchedClass = MI.getDesc().getSchedClass();
  2706:   return is_TC2(SchedClass);
  2707: }
  2708: 
  2709: bool HexagonInstrInfo::isTC2Early(const MachineInstr &MI) const {
  2710:   unsigned SchedClass = MI.getDesc().getSchedClass();
  2711:   return is_TC2early(SchedClass);
  2712: }
  2713: 
  2714: bool HexagonInstrInfo::isTC4x(const MachineInstr &MI) const {
  2715:   unsigned SchedClass = MI.getDesc().getSchedClass();
  2716:   return is_TC4x(SchedClass);
  2717: }
  2718: 
  2719: // Schedule this ASAP.
  2720: bool HexagonInstrInfo::isToBeScheduledASAP(const MachineInstr &MI1,
  2721:       const MachineInstr &MI2) const {
  2722:   if (mayBeCurLoad(MI1)) {
  2723:     // if (result of SU is used in Next) return true;
  2724:     Register DstReg = MI1.getOperand(0).getReg();
  2725:     int N = MI2.getNumOperands();
  2726:     for (int I = 0; I < N; I++)
  2727:       if (MI2.getOperand(I).isReg() && DstReg == MI2.getOperand(I).getReg())
  2728:         return true;
  2729:   }
  2730:   if (mayBeNewStore(MI2))
  2731:     if (MI2.getOpcode() == Hexagon::V6_vS32b_pi)
  2732:       if (MI1.getOperand(0).isReg() && MI2.getOperand(3).isReg() &&
  2733:           MI1.getOperand(0).getReg() == MI2.getOperand(3).getReg())
  2734:         return true;
  2735:   return false;
  2736: }
  2737: 
  2738: bool HexagonInstrInfo::isHVXVec(const MachineInstr &MI) const {
  2739:   const uint64_t V = getType(MI);
  2740:   return HexagonII::TypeCVI_FIRST <= V && V <= HexagonII::TypeCVI_LAST;
  2741: }
  2742: 
  2743: // Check if the Offset is a valid auto-inc imm by Load/Store Type.
  2744: bool HexagonInstrInfo::isValidAutoIncImm(const EVT VT, int Offset) const {
  2745:   int Size = VT.getSizeInBits() / 8;
  2746:   if (Offset % Size != 0)
  2747:     return false;
  2748:   int Count = Offset / Size;
  2749: 
  2750:   switch (VT.getSimpleVT().SimpleTy) {
  2751:     // For scalars the auto-inc is s4
  2752:     case MVT::i8:
  2753:     case MVT::i16:
  2754:     case MVT::i32:
  2755:     case MVT::i64:
  2756:     case MVT::f32:
  2757:     case MVT::f64:
  2758:     case MVT::v2i16:
  2759:     case MVT::v2i32:
  2760:     case MVT::v4i8:
  2761:     case MVT::v4i16:
  2762:     case MVT::v8i8:
  2763:       return isInt<4>(Count);
  2764:     // For HVX vectors the auto-inc is s3
  2765:     case MVT::v64i8:
  2766:     case MVT::v32i16:
  2767:     case MVT::v16i32:
  2768:     case MVT::v8i64:
  2769:     case MVT::v128i8:
  2770:     case MVT::v64i16:
  2771:     case MVT::v32i32:
  2772:     case MVT::v16i64:
  2773:       return isInt<3>(Count);
  2774:     default:
  2775:       break;
  2776:   }
  2777: 
  2778:   llvm_unreachable("Not an valid type!");
  2779: }
  2780: 
  2781: bool HexagonInstrInfo::isValidOffset(unsigned Opcode, int Offset,
  2782:       const TargetRegisterInfo *TRI, bool Extend) const {
  2783:   // This function is to check whether the "Offset" is in the correct range of
  2784:   // the given "Opcode". If "Offset" is not in the correct range, "A2_addi" is
  2785:   // inserted to calculate the final address. Due to this reason, the function
  2786:   // assumes that the "Offset" has correct alignment.
  2787:   // We used to assert if the offset was not properly aligned, however,
  2788:   // there are cases where a misaligned pointer recast can cause this
  2789:   // problem, and we need to allow for it. The front end warns of such
  2790:   // misaligns with respect to load size.
  2791:   switch (Opcode) {
  2792:   case Hexagon::PS_vstorerq_ai:
  2793:   case Hexagon::PS_vstorerv_ai:
  2794:   case Hexagon::PS_vstorerw_ai:
  2795:   case Hexagon::PS_vstorerw_nt_ai:
  2796:   case Hexagon::PS_vloadrq_ai:
  2797:   case Hexagon::PS_vloadrv_ai:
  2798:   case Hexagon::PS_vloadrw_ai:
  2799:   case Hexagon::PS_vloadrw_nt_ai:
  2800:   case Hexagon::V6_vL32b_ai:
```
- EN: It declares or implements routines such as HexagonInstrInfo::isSolo, HexagonInstrInfo::isSpillPredRegOp, HexagonInstrInfo::isTailCall, HexagonInstrInfo::isTC1, ... (22 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonInstrInfo::isSolo, HexagonInstrInfo::isSpillPredRegOp, HexagonInstrInfo::isTailCall, HexagonInstrInfo::isTC1, ... (22 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 2801-3000 / 第 2801-3000 行

```cpp
  2801:   case Hexagon::V6_vS32b_ai:
  2802:   case Hexagon::V6_vS32b_pred_ai:
  2803:   case Hexagon::V6_vS32b_npred_ai:
  2804:   case Hexagon::V6_vS32b_qpred_ai:
  2805:   case Hexagon::V6_vS32b_nqpred_ai:
  2806:   case Hexagon::V6_vS32b_new_ai:
  2807:   case Hexagon::V6_vS32b_new_pred_ai:
  2808:   case Hexagon::V6_vS32b_new_npred_ai:
  2809:   case Hexagon::V6_vS32b_nt_pred_ai:
  2810:   case Hexagon::V6_vS32b_nt_npred_ai:
  2811:   case Hexagon::V6_vS32b_nt_new_ai:
  2812:   case Hexagon::V6_vS32b_nt_new_pred_ai:
  2813:   case Hexagon::V6_vS32b_nt_new_npred_ai:
  2814:   case Hexagon::V6_vS32b_nt_qpred_ai:
  2815:   case Hexagon::V6_vS32b_nt_nqpred_ai:
  2816:   case Hexagon::V6_vL32b_nt_ai:
  2817:   case Hexagon::V6_vS32b_nt_ai:
  2818:   case Hexagon::V6_vL32Ub_ai:
  2819:   case Hexagon::V6_vS32Ub_ai:
  2820:   case Hexagon::V6_vL32b_cur_ai:
  2821:   case Hexagon::V6_vL32b_tmp_ai:
  2822:   case Hexagon::V6_vL32b_pred_ai:
  2823:   case Hexagon::V6_vL32b_npred_ai:
  2824:   case Hexagon::V6_vL32b_cur_pred_ai:
  2825:   case Hexagon::V6_vL32b_cur_npred_ai:
  2826:   case Hexagon::V6_vL32b_tmp_pred_ai:
  2827:   case Hexagon::V6_vL32b_tmp_npred_ai:
  2828:   case Hexagon::V6_vL32b_nt_cur_ai:
  2829:   case Hexagon::V6_vL32b_nt_tmp_ai:
  2830:   case Hexagon::V6_vL32b_nt_pred_ai:
  2831:   case Hexagon::V6_vL32b_nt_npred_ai:
  2832:   case Hexagon::V6_vL32b_nt_cur_pred_ai:
  2833:   case Hexagon::V6_vL32b_nt_cur_npred_ai:
  2834:   case Hexagon::V6_vL32b_nt_tmp_pred_ai:
  2835:   case Hexagon::V6_vL32b_nt_tmp_npred_ai:
  2836:   case Hexagon::V6_vS32Ub_npred_ai:
  2837:   case Hexagon::V6_vgathermh_pseudo:
  2838:   case Hexagon::V6_vgather_vscatter_mh_pseudo:
  2839:   case Hexagon::V6_vgathermw_pseudo:
  2840:   case Hexagon::V6_vgathermhw_pseudo:
  2841:   case Hexagon::V6_vgathermhq_pseudo:
  2842:   case Hexagon::V6_vgathermwq_pseudo:
  2843:   case Hexagon::V6_vgathermhwq_pseudo: {
  2844:     unsigned VectorSize = TRI->getSpillSize(Hexagon::HvxVRRegClass);
  2845:     assert(isPowerOf2_32(VectorSize));
  2846:     if (Offset & (VectorSize-1))
  2847:       return false;
  2848:     return isInt<4>(Offset >> Log2_32(VectorSize));
  2849:   }
  2850: 
  2851:   case Hexagon::J2_loop0i:
  2852:   case Hexagon::J2_loop1i:
  2853:     return isUInt<10>(Offset);
  2854: 
  2855:   case Hexagon::S4_storeirb_io:
  2856:   case Hexagon::S4_storeirbt_io:
  2857:   case Hexagon::S4_storeirbf_io:
  2858:     return isUInt<6>(Offset);
  2859: 
  2860:   case Hexagon::S4_storeirh_io:
  2861:   case Hexagon::S4_storeirht_io:
  2862:   case Hexagon::S4_storeirhf_io:
  2863:     return isShiftedUInt<6,1>(Offset);
  2864: 
  2865:   case Hexagon::S4_storeiri_io:
  2866:   case Hexagon::S4_storeirit_io:
  2867:   case Hexagon::S4_storeirif_io:
  2868:     return isShiftedUInt<6,2>(Offset);
  2869:   // Handle these two compare instructions that are not extendable.
  2870:   case Hexagon::A4_cmpbeqi:
  2871:     return isUInt<8>(Offset);
  2872:   case Hexagon::A4_cmpbgti:
  2873:     return isInt<8>(Offset);
  2874:   }
  2875: 
  2876:   if (Extend)
  2877:     return true;
  2878: 
  2879:   switch (Opcode) {
  2880:   case Hexagon::L2_loadri_io:
  2881:   case Hexagon::S2_storeri_io:
  2882:     return (Offset >= Hexagon_MEMW_OFFSET_MIN) &&
  2883:       (Offset <= Hexagon_MEMW_OFFSET_MAX);
  2884: 
  2885:   case Hexagon::L2_loadrd_io:
  2886:   case Hexagon::S2_storerd_io:
  2887:     return (Offset >= Hexagon_MEMD_OFFSET_MIN) &&
  2888:       (Offset <= Hexagon_MEMD_OFFSET_MAX);
  2889: 
  2890:   case Hexagon::L2_loadrh_io:
  2891:   case Hexagon::L2_loadruh_io:
  2892:   case Hexagon::S2_storerh_io:
  2893:   case Hexagon::S2_storerf_io:
  2894:     return (Offset >= Hexagon_MEMH_OFFSET_MIN) &&
  2895:       (Offset <= Hexagon_MEMH_OFFSET_MAX);
  2896: 
  2897:   case Hexagon::L2_loadrb_io:
  2898:   case Hexagon::L2_loadrub_io:
  2899:   case Hexagon::S2_storerb_io:
  2900:     return (Offset >= Hexagon_MEMB_OFFSET_MIN) &&
  2901:       (Offset <= Hexagon_MEMB_OFFSET_MAX);
  2902: 
  2903:   case Hexagon::A2_addi:
  2904:     return (Offset >= Hexagon_ADDI_OFFSET_MIN) &&
  2905:       (Offset <= Hexagon_ADDI_OFFSET_MAX);
  2906: 
  2907:   case Hexagon::L4_iadd_memopw_io:
  2908:   case Hexagon::L4_isub_memopw_io:
  2909:   case Hexagon::L4_add_memopw_io:
  2910:   case Hexagon::L4_sub_memopw_io:
  2911:   case Hexagon::L4_iand_memopw_io:
  2912:   case Hexagon::L4_ior_memopw_io:
  2913:   case Hexagon::L4_and_memopw_io:
  2914:   case Hexagon::L4_or_memopw_io:
  2915:     return (0 <= Offset && Offset <= 255);
  2916: 
  2917:   case Hexagon::L4_iadd_memoph_io:
  2918:   case Hexagon::L4_isub_memoph_io:
  2919:   case Hexagon::L4_add_memoph_io:
  2920:   case Hexagon::L4_sub_memoph_io:
  2921:   case Hexagon::L4_iand_memoph_io:
  2922:   case Hexagon::L4_ior_memoph_io:
  2923:   case Hexagon::L4_and_memoph_io:
  2924:   case Hexagon::L4_or_memoph_io:
  2925:     return (0 <= Offset && Offset <= 127);
  2926: 
  2927:   case Hexagon::L4_iadd_memopb_io:
  2928:   case Hexagon::L4_isub_memopb_io:
  2929:   case Hexagon::L4_add_memopb_io:
  2930:   case Hexagon::L4_sub_memopb_io:
  2931:   case Hexagon::L4_iand_memopb_io:
  2932:   case Hexagon::L4_ior_memopb_io:
  2933:   case Hexagon::L4_and_memopb_io:
  2934:   case Hexagon::L4_or_memopb_io:
  2935:     return (0 <= Offset && Offset <= 63);
  2936: 
  2937:   // LDriw_xxx and STriw_xxx are pseudo operations, so it has to take offset of
  2938:   // any size. Later pass knows how to handle it.
  2939:   case Hexagon::STriw_pred:
  2940:   case Hexagon::LDriw_pred:
  2941:   case Hexagon::STriw_ctr:
  2942:   case Hexagon::LDriw_ctr:
  2943:     return true;
  2944: 
  2945:   case Hexagon::PS_fi:
  2946:   case Hexagon::PS_fia:
  2947:   case Hexagon::INLINEASM:
  2948:     return true;
  2949: 
  2950:   case Hexagon::L2_ploadrbt_io:
  2951:   case Hexagon::L2_ploadrbf_io:
  2952:   case Hexagon::L2_ploadrubt_io:
  2953:   case Hexagon::L2_ploadrubf_io:
  2954:   case Hexagon::S2_pstorerbt_io:
  2955:   case Hexagon::S2_pstorerbf_io:
  2956:     return isUInt<6>(Offset);
  2957: 
  2958:   case Hexagon::L2_ploadrht_io:
  2959:   case Hexagon::L2_ploadrhf_io:
  2960:   case Hexagon::L2_ploadruht_io:
  2961:   case Hexagon::L2_ploadruhf_io:
  2962:   case Hexagon::S2_pstorerht_io:
  2963:   case Hexagon::S2_pstorerhf_io:
  2964:     return isShiftedUInt<6,1>(Offset);
  2965: 
  2966:   case Hexagon::L2_ploadrit_io:
  2967:   case Hexagon::L2_ploadrif_io:
  2968:   case Hexagon::S2_pstorerit_io:
  2969:   case Hexagon::S2_pstorerif_io:
  2970:     return isShiftedUInt<6,2>(Offset);
  2971: 
  2972:   case Hexagon::L2_ploadrdt_io:
  2973:   case Hexagon::L2_ploadrdf_io:
  2974:   case Hexagon::S2_pstorerdt_io:
  2975:   case Hexagon::S2_pstorerdf_io:
  2976:     return isShiftedUInt<6,3>(Offset);
  2977: 
  2978:   case Hexagon::L2_loadbsw2_io:
  2979:   case Hexagon::L2_loadbzw2_io:
  2980:     return isShiftedInt<11,1>(Offset);
  2981: 
  2982:   case Hexagon::L2_loadbsw4_io:
  2983:   case Hexagon::L2_loadbzw4_io:
  2984:     return isShiftedInt<11,2>(Offset);
  2985:   } // switch
  2986: 
  2987:   dbgs() << "Failed Opcode is : " << Opcode << " (" << getName(Opcode)
  2988:          << ")\n";
  2989:   llvm_unreachable("No offset range is defined for this opcode. "
  2990:                    "Please define it in the above switch statement!");
  2991: }
  2992: 
  2993: bool HexagonInstrInfo::isVecAcc(const MachineInstr &MI) const {
  2994:   return isHVXVec(MI) && isAccumulator(MI);
  2995: }
  2996: 
  2997: bool HexagonInstrInfo::isVecALU(const MachineInstr &MI) const {
  2998:   const uint64_t F = get(MI.getOpcode()).TSFlags;
  2999:   const uint64_t V = ((F >> HexagonII::TypePos) & HexagonII::TypeMask);
  3000:   return
```
- EN: It declares or implements routines such as getSpillSize, assert, isInt<4>, isUInt<10>, ... (11 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include Hexagon_MEMW_OFFSET_MIN, Hexagon_MEMW_OFFSET_MAX, Hexagon_MEMD_OFFSET_MIN, Hexagon_MEMD_OFFSET_MAX, ... (12 total), showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getSpillSize, assert, isInt<4>, isUInt<10>, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 Hexagon_MEMW_OFFSET_MIN, Hexagon_MEMW_OFFSET_MAX, Hexagon_MEMD_OFFSET_MIN, Hexagon_MEMD_OFFSET_MAX, ... (12 total)，说明了它与同级后端组件的连接关系。

### Lines 3001-3200 / 第 3001-3200 行

```cpp
  3001:     V == HexagonII::TypeCVI_VA         ||
  3002:     V == HexagonII::TypeCVI_VA_DV;
  3003: }
  3004: 
  3005: bool HexagonInstrInfo::isVecUsableNextPacket(const MachineInstr &ProdMI,
  3006:       const MachineInstr &ConsMI) const {
  3007:   if (EnableACCForwarding && isVecAcc(ProdMI) && isVecAcc(ConsMI))
  3008:     return true;
  3009: 
  3010:   if (EnableALUForwarding && (isVecALU(ConsMI) || isLateSourceInstr(ConsMI)))
  3011:     return true;
  3012: 
  3013:   if (mayBeNewStore(ConsMI))
  3014:     return true;
  3015: 
  3016:   return false;
  3017: }
  3018: 
  3019: bool HexagonInstrInfo::isZeroExtendingLoad(const MachineInstr &MI) const {
  3020:   switch (MI.getOpcode()) {
  3021:   // Byte
  3022:   case Hexagon::L2_loadrub_io:
  3023:   case Hexagon::L4_loadrub_ur:
  3024:   case Hexagon::L4_loadrub_ap:
  3025:   case Hexagon::L2_loadrub_pr:
  3026:   case Hexagon::L2_loadrub_pbr:
  3027:   case Hexagon::L2_loadrub_pi:
  3028:   case Hexagon::L2_loadrub_pci:
  3029:   case Hexagon::L2_loadrub_pcr:
  3030:   case Hexagon::L2_loadbzw2_io:
  3031:   case Hexagon::L4_loadbzw2_ur:
  3032:   case Hexagon::L4_loadbzw2_ap:
  3033:   case Hexagon::L2_loadbzw2_pr:
  3034:   case Hexagon::L2_loadbzw2_pbr:
  3035:   case Hexagon::L2_loadbzw2_pi:
  3036:   case Hexagon::L2_loadbzw2_pci:
  3037:   case Hexagon::L2_loadbzw2_pcr:
  3038:   case Hexagon::L2_loadbzw4_io:
  3039:   case Hexagon::L4_loadbzw4_ur:
  3040:   case Hexagon::L4_loadbzw4_ap:
  3041:   case Hexagon::L2_loadbzw4_pr:
  3042:   case Hexagon::L2_loadbzw4_pbr:
  3043:   case Hexagon::L2_loadbzw4_pi:
  3044:   case Hexagon::L2_loadbzw4_pci:
  3045:   case Hexagon::L2_loadbzw4_pcr:
  3046:   case Hexagon::L4_loadrub_rr:
  3047:   case Hexagon::L2_ploadrubt_io:
  3048:   case Hexagon::L2_ploadrubt_pi:
  3049:   case Hexagon::L2_ploadrubf_io:
  3050:   case Hexagon::L2_ploadrubf_pi:
  3051:   case Hexagon::L2_ploadrubtnew_io:
  3052:   case Hexagon::L2_ploadrubfnew_io:
  3053:   case Hexagon::L4_ploadrubt_rr:
  3054:   case Hexagon::L4_ploadrubf_rr:
  3055:   case Hexagon::L4_ploadrubtnew_rr:
  3056:   case Hexagon::L4_ploadrubfnew_rr:
  3057:   case Hexagon::L2_ploadrubtnew_pi:
  3058:   case Hexagon::L2_ploadrubfnew_pi:
  3059:   case Hexagon::L4_ploadrubt_abs:
  3060:   case Hexagon::L4_ploadrubf_abs:
  3061:   case Hexagon::L4_ploadrubtnew_abs:
  3062:   case Hexagon::L4_ploadrubfnew_abs:
  3063:   case Hexagon::L2_loadrubgp:
  3064:   // Half
  3065:   case Hexagon::L2_loadruh_io:
  3066:   case Hexagon::L4_loadruh_ur:
  3067:   case Hexagon::L4_loadruh_ap:
  3068:   case Hexagon::L2_loadruh_pr:
  3069:   case Hexagon::L2_loadruh_pbr:
  3070:   case Hexagon::L2_loadruh_pi:
  3071:   case Hexagon::L2_loadruh_pci:
  3072:   case Hexagon::L2_loadruh_pcr:
  3073:   case Hexagon::L4_loadruh_rr:
  3074:   case Hexagon::L2_ploadruht_io:
  3075:   case Hexagon::L2_ploadruht_pi:
  3076:   case Hexagon::L2_ploadruhf_io:
  3077:   case Hexagon::L2_ploadruhf_pi:
  3078:   case Hexagon::L2_ploadruhtnew_io:
  3079:   case Hexagon::L2_ploadruhfnew_io:
  3080:   case Hexagon::L4_ploadruht_rr:
  3081:   case Hexagon::L4_ploadruhf_rr:
  3082:   case Hexagon::L4_ploadruhtnew_rr:
  3083:   case Hexagon::L4_ploadruhfnew_rr:
  3084:   case Hexagon::L2_ploadruhtnew_pi:
  3085:   case Hexagon::L2_ploadruhfnew_pi:
  3086:   case Hexagon::L4_ploadruht_abs:
  3087:   case Hexagon::L4_ploadruhf_abs:
  3088:   case Hexagon::L4_ploadruhtnew_abs:
  3089:   case Hexagon::L4_ploadruhfnew_abs:
  3090:   case Hexagon::L2_loadruhgp:
  3091:     return true;
  3092:   default:
  3093:     return false;
  3094:   }
  3095: }
  3096: 
  3097: // Add latency to instruction.
  3098: bool HexagonInstrInfo::addLatencyToSchedule(const MachineInstr &MI1,
  3099:       const MachineInstr &MI2) const {
  3100:   if (isHVXVec(MI1) && isHVXVec(MI2))
  3101:     if (!isVecUsableNextPacket(MI1, MI2))
  3102:       return true;
  3103:   return false;
  3104: }
  3105: 
  3106: /// Get the base register and byte offset of a load/store instr.
  3107: bool HexagonInstrInfo::getMemOperandsWithOffsetWidth(
  3108:     const MachineInstr &LdSt, SmallVectorImpl<const MachineOperand *> &BaseOps,
  3109:     int64_t &Offset, bool &OffsetIsScalable, LocationSize &Width,
  3110:     const TargetRegisterInfo *TRI) const {
  3111:   OffsetIsScalable = false;
  3112:   const MachineOperand *BaseOp = getBaseAndOffset(LdSt, Offset, Width);
  3113:   if (!BaseOp || !BaseOp->isReg())
  3114:     return false;
  3115:   BaseOps.push_back(BaseOp);
  3116:   return true;
  3117: }
  3118: 
  3119: /// Can these instructions execute at the same time in a bundle.
  3120: bool HexagonInstrInfo::canExecuteInBundle(const MachineInstr &First,
  3121:       const MachineInstr &Second) const {
  3122:   if (Second.mayStore() && First.getOpcode() == Hexagon::S2_allocframe) {
  3123:     const MachineOperand &Op = Second.getOperand(0);
  3124:     if (Op.isReg() && Op.isUse() && Op.getReg() == Hexagon::R29)
  3125:       return true;
  3126:   }
  3127:   if (DisableNVSchedule)
  3128:     return false;
  3129:   if (mayBeNewStore(Second)) {
  3130:     // Make sure the definition of the first instruction is the value being
  3131:     // stored.
  3132:     const MachineOperand &Stored =
  3133:       Second.getOperand(Second.getNumOperands() - 1);
  3134:     if (!Stored.isReg())
  3135:       return false;
  3136:     for (unsigned i = 0, e = First.getNumOperands(); i < e; ++i) {
  3137:       const MachineOperand &Op = First.getOperand(i);
  3138:       if (Op.isReg() && Op.isDef() && Op.getReg() == Stored.getReg())
  3139:         return true;
  3140:     }
  3141:   }
  3142:   return false;
  3143: }
  3144: 
  3145: bool HexagonInstrInfo::doesNotReturn(const MachineInstr &CallMI) const {
  3146:   unsigned Opc = CallMI.getOpcode();
  3147:   return Opc == Hexagon::PS_call_nr || Opc == Hexagon::PS_callr_nr;
  3148: }
  3149: 
  3150: bool HexagonInstrInfo::hasEHLabel(const MachineBasicBlock *B) const {
  3151:   for (auto &I : *B)
  3152:     if (I.isEHLabel())
  3153:       return true;
  3154:   return false;
  3155: }
  3156: 
  3157: // Returns true if an instruction can be converted into a non-extended
  3158: // equivalent instruction.
  3159: bool HexagonInstrInfo::hasNonExtEquivalent(const MachineInstr &MI) const {
  3160:   short NonExtOpcode;
  3161:   // Check if the instruction has a register form that uses register in place
  3162:   // of the extended operand, if so return that as the non-extended form.
  3163:   if (Hexagon::getRegForm(MI.getOpcode()) >= 0)
  3164:     return true;
  3165: 
  3166:   if (MI.getDesc().mayLoad() || MI.getDesc().mayStore()) {
  3167:     // Check addressing mode and retrieve non-ext equivalent instruction.
  3168: 
  3169:     switch (getAddrMode(MI)) {
  3170:     case HexagonII::Absolute:
  3171:       // Load/store with absolute addressing mode can be converted into
  3172:       // base+offset mode.
  3173:       NonExtOpcode = Hexagon::changeAddrMode_abs_io(MI.getOpcode());
  3174:       break;
  3175:     case HexagonII::BaseImmOffset:
  3176:       // Load/store with base+offset addressing mode can be converted into
  3177:       // base+register offset addressing mode. However left shift operand should
  3178:       // be set to 0.
  3179:       NonExtOpcode = Hexagon::changeAddrMode_io_rr(MI.getOpcode());
  3180:       break;
  3181:     case HexagonII::BaseLongOffset:
  3182:       NonExtOpcode = Hexagon::changeAddrMode_ur_rr(MI.getOpcode());
  3183:       break;
  3184:     default:
  3185:       return false;
  3186:     }
  3187:     if (NonExtOpcode < 0)
  3188:       return false;
  3189:     return true;
  3190:   }
  3191:   return false;
  3192: }
  3193: 
  3194: bool HexagonInstrInfo::hasPseudoInstrPair(const MachineInstr &MI) const {
  3195:   return Hexagon::getRealHWInstr(MI.getOpcode(),
  3196:                                  Hexagon::InstrType_Pseudo) >= 0;
  3197: }
  3198: 
  3199: bool HexagonInstrInfo::hasUncondBranch(const MachineBasicBlock *B)
  3200:       const {
```
- EN: It declares or implements routines such as HexagonInstrInfo::isVecUsableNextPacket, HexagonInstrInfo::isZeroExtendingLoad, HexagonInstrInfo::addLatencyToSchedule, HexagonInstrInfo::getMemOperandsWithOffsetWidth, ... (17 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonInstrInfo::isVecUsableNextPacket, HexagonInstrInfo::isZeroExtendingLoad, HexagonInstrInfo::addLatencyToSchedule, HexagonInstrInfo::getMemOperandsWithOffsetWidth, ... (17 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII, HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 3201-3400 / 第 3201-3400 行

```cpp
  3201:   MachineBasicBlock::const_iterator I = B->getFirstTerminator(), E = B->end();
  3202:   while (I != E) {
  3203:     if (I->isBarrier())
  3204:       return true;
  3205:     ++I;
  3206:   }
  3207:   return false;
  3208: }
  3209: 
  3210: // Returns true, if a LD insn can be promoted to a cur load.
  3211: bool HexagonInstrInfo::mayBeCurLoad(const MachineInstr &MI) const {
  3212:   const uint64_t F = MI.getDesc().TSFlags;
  3213:   return ((F >> HexagonII::mayCVLoadPos) & HexagonII::mayCVLoadMask) &&
  3214:          Subtarget.hasV60Ops();
  3215: }
  3216: 
  3217: // Returns true, if a ST insn can be promoted to a new-value store.
  3218: bool HexagonInstrInfo::mayBeNewStore(const MachineInstr &MI) const {
  3219:   if (MI.mayStore() && !Subtarget.useNewValueStores())
  3220:     return false;
  3221: 
  3222:   const uint64_t F = MI.getDesc().TSFlags;
  3223:   return (F >> HexagonII::mayNVStorePos) & HexagonII::mayNVStoreMask;
  3224: }
  3225: 
  3226: bool HexagonInstrInfo::producesStall(const MachineInstr &ProdMI,
  3227:       const MachineInstr &ConsMI) const {
  3228:   // There is no stall when ProdMI is not a V60 vector.
  3229:   if (!isHVXVec(ProdMI))
  3230:     return false;
  3231: 
  3232:   // There is no stall when ProdMI and ConsMI are not dependent.
  3233:   if (!isDependent(ProdMI, ConsMI))
  3234:     return false;
  3235: 
  3236:   // When Forward Scheduling is enabled, there is no stall if ProdMI and ConsMI
  3237:   // are scheduled in consecutive packets.
  3238:   if (isVecUsableNextPacket(ProdMI, ConsMI))
  3239:     return false;
  3240: 
  3241:   return true;
  3242: }
  3243: 
  3244: bool HexagonInstrInfo::producesStall(const MachineInstr &MI,
  3245:       MachineBasicBlock::const_instr_iterator BII) const {
  3246:   // There is no stall when I is not a V60 vector.
  3247:   if (!isHVXVec(MI))
  3248:     return false;
  3249: 
  3250:   MachineBasicBlock::const_instr_iterator MII = BII;
  3251:   MachineBasicBlock::const_instr_iterator MIE = MII->getParent()->instr_end();
  3252: 
  3253:   if (!MII->isBundle())
  3254:     return producesStall(*MII, MI);
  3255: 
  3256:   for (++MII; MII != MIE && MII->isInsideBundle(); ++MII) {
  3257:     const MachineInstr &J = *MII;
  3258:     if (producesStall(J, MI))
  3259:       return true;
  3260:   }
  3261:   return false;
  3262: }
  3263: 
  3264: bool HexagonInstrInfo::predCanBeUsedAsDotNew(const MachineInstr &MI,
  3265:       Register PredReg) const {
  3266:   for (const MachineOperand &MO : MI.operands()) {
  3267:     // Predicate register must be explicitly defined.
  3268:     if (MO.isRegMask() && MO.clobbersPhysReg(PredReg))
  3269:       return false;
  3270:     if (MO.isReg() && MO.isDef() && MO.isImplicit() && (MO.getReg() == PredReg))
  3271:       return false;
  3272:   }
  3273: 
  3274:   // Instruction that produce late predicate cannot be used as sources of
  3275:   // dot-new.
  3276:   switch (MI.getOpcode()) {
  3277:     case Hexagon::A4_addp_c:
  3278:     case Hexagon::A4_subp_c:
  3279:     case Hexagon::A4_tlbmatch:
  3280:     case Hexagon::A5_ACS:
  3281:     case Hexagon::F2_sfinvsqrta:
  3282:     case Hexagon::F2_sfrecipa:
  3283:     case Hexagon::J2_endloop0:
  3284:     case Hexagon::J2_endloop01:
  3285:     case Hexagon::J2_ploop1si:
  3286:     case Hexagon::J2_ploop1sr:
  3287:     case Hexagon::J2_ploop2si:
  3288:     case Hexagon::J2_ploop2sr:
  3289:     case Hexagon::J2_ploop3si:
  3290:     case Hexagon::J2_ploop3sr:
  3291:     case Hexagon::S2_cabacdecbin:
  3292:     case Hexagon::S2_storew_locked:
  3293:     case Hexagon::S4_stored_locked:
  3294:       return false;
  3295:   }
  3296:   return true;
  3297: }
  3298: 
  3299: bool HexagonInstrInfo::PredOpcodeHasJMP_c(unsigned Opcode) const {
  3300:   return Opcode == Hexagon::J2_jumpt      ||
  3301:          Opcode == Hexagon::J2_jumptpt    ||
  3302:          Opcode == Hexagon::J2_jumpf      ||
  3303:          Opcode == Hexagon::J2_jumpfpt    ||
  3304:          Opcode == Hexagon::J2_jumptnew   ||
  3305:          Opcode == Hexagon::J2_jumpfnew   ||
  3306:          Opcode == Hexagon::J2_jumptnewpt ||
  3307:          Opcode == Hexagon::J2_jumpfnewpt;
  3308: }
  3309: 
  3310: bool HexagonInstrInfo::predOpcodeHasNot(ArrayRef<MachineOperand> Cond) const {
  3311:   if (Cond.empty() || !isPredicated(Cond[0].getImm()))
  3312:     return false;
  3313:   return !isPredicatedTrue(Cond[0].getImm());
  3314: }
  3315: 
  3316: unsigned HexagonInstrInfo::getAddrMode(const MachineInstr &MI) const {
  3317:   const uint64_t F = MI.getDesc().TSFlags;
  3318:   return (F >> HexagonII::AddrModePos) & HexagonII::AddrModeMask;
  3319: }
  3320: 
  3321: // Returns the base register in a memory access (load/store). The offset is
  3322: // returned in Offset and the access size is returned in AccessSize.
  3323: // If the base operand has a subregister or the offset field does not contain
  3324: // an immediate value, return nullptr.
  3325: MachineOperand *
  3326: HexagonInstrInfo::getBaseAndOffset(const MachineInstr &MI, int64_t &Offset,
  3327:                                    LocationSize &AccessSize) const {
  3328:   // Return if it is not a base+offset type instruction or a MemOp.
  3329:   if (getAddrMode(MI) != HexagonII::BaseImmOffset &&
  3330:       getAddrMode(MI) != HexagonII::BaseLongOffset && !isMemOp(MI) &&
  3331:       !isPostIncrement(MI))
  3332:     return nullptr;
  3333: 
  3334:   AccessSize = LocationSize::precise(getMemAccessSize(MI));
  3335: 
  3336:   unsigned BasePos = 0, OffsetPos = 0;
  3337:   if (!getBaseAndOffsetPosition(MI, BasePos, OffsetPos))
  3338:     return nullptr;
  3339: 
  3340:   // Post increment updates its EA after the mem access,
  3341:   // so we need to treat its offset as zero.
  3342:   if (isPostIncrement(MI)) {
  3343:     Offset = 0;
  3344:   } else {
  3345:     const MachineOperand &OffsetOp = MI.getOperand(OffsetPos);
  3346:     if (!OffsetOp.isImm())
  3347:       return nullptr;
  3348:     Offset = OffsetOp.getImm();
  3349:   }
  3350: 
  3351:   const MachineOperand &BaseOp = MI.getOperand(BasePos);
  3352:   if (BaseOp.getSubReg() != 0)
  3353:     return nullptr;
  3354:   return &const_cast<MachineOperand&>(BaseOp);
  3355: }
  3356: 
  3357: /// Return the position of the base and offset operands for this instruction.
  3358: bool HexagonInstrInfo::getBaseAndOffsetPosition(const MachineInstr &MI,
  3359:       unsigned &BasePos, unsigned &OffsetPos) const {
  3360:   if (!isAddrModeWithOffset(MI) && !isPostIncrement(MI))
  3361:     return false;
  3362: 
  3363:   // Deal with memops first.
  3364:   if (isMemOp(MI)) {
  3365:     BasePos = 0;
  3366:     OffsetPos = 1;
  3367:   } else if (MI.mayStore()) {
  3368:     BasePos = 0;
  3369:     OffsetPos = 1;
  3370:   } else if (MI.mayLoad()) {
  3371:     BasePos = 1;
  3372:     OffsetPos = 2;
  3373:   } else
  3374:     return false;
  3375: 
  3376:   if (isPredicated(MI)) {
  3377:     BasePos++;
  3378:     OffsetPos++;
  3379:   }
  3380:   if (isPostIncrement(MI)) {
  3381:     BasePos++;
  3382:     OffsetPos++;
  3383:   }
  3384: 
  3385:   if (!MI.getOperand(BasePos).isReg() || !MI.getOperand(OffsetPos).isImm())
  3386:     return false;
  3387: 
  3388:   return true;
  3389: }
  3390: 
  3391: // Inserts branching instructions in reverse order of their occurrence.
  3392: // e.g. jump_t t1 (i1)
  3393: // jump t2        (i2)
  3394: // Jumpers = {i2, i1}
  3395: SmallVector<MachineInstr*, 2> HexagonInstrInfo::getBranchingInstrs(
  3396:       MachineBasicBlock& MBB) const {
  3397:   SmallVector<MachineInstr*, 2> Jumpers;
  3398:   // If the block has no terminators, it just falls into the block after it.
  3399:   MachineBasicBlock::instr_iterator I = MBB.instr_end();
  3400:   if (I == MBB.instr_begin())
```
- EN: It declares or implements routines such as getFirstTerminator, HexagonInstrInfo::mayBeCurLoad, HexagonInstrInfo::mayBeNewStore, HexagonInstrInfo::producesStall, ... (18 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getFirstTerminator, HexagonInstrInfo::mayBeCurLoad, HexagonInstrInfo::mayBeNewStore, HexagonInstrInfo::producesStall, ... (18 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 3401-3600 / 第 3401-3600 行

```cpp
  3401:     return Jumpers;
  3402: 
  3403:   // A basic block may looks like this:
  3404:   //
  3405:   //  [   insn
  3406:   //     EH_LABEL
  3407:   //      insn
  3408:   //      insn
  3409:   //      insn
  3410:   //     EH_LABEL
  3411:   //      insn     ]
  3412:   //
  3413:   // It has two succs but does not have a terminator
  3414:   // Don't know how to handle it.
  3415:   do {
  3416:     --I;
  3417:     if (I->isEHLabel())
  3418:       return Jumpers;
  3419:   } while (I != MBB.instr_begin());
  3420: 
  3421:   I = MBB.instr_end();
  3422:   --I;
  3423: 
  3424:   while (I->isDebugInstr()) {
  3425:     if (I == MBB.instr_begin())
  3426:       return Jumpers;
  3427:     --I;
  3428:   }
  3429:   if (!isUnpredicatedTerminator(*I))
  3430:     return Jumpers;
  3431: 
  3432:   // Get the last instruction in the block.
  3433:   MachineInstr *LastInst = &*I;
  3434:   Jumpers.push_back(LastInst);
  3435:   MachineInstr *SecondLastInst = nullptr;
  3436:   // Find one more terminator if present.
  3437:   do {
  3438:     if (&*I != LastInst && !I->isBundle() && isUnpredicatedTerminator(*I)) {
  3439:       if (!SecondLastInst) {
  3440:         SecondLastInst = &*I;
  3441:         Jumpers.push_back(SecondLastInst);
  3442:       } else // This is a third branch.
  3443:         return Jumpers;
  3444:     }
  3445:     if (I == MBB.instr_begin())
  3446:       break;
  3447:     --I;
  3448:   } while (true);
  3449:   return Jumpers;
  3450: }
  3451: 
  3452: // Returns Operand Index for the constant extended instruction.
  3453: unsigned HexagonInstrInfo::getCExtOpNum(const MachineInstr &MI) const {
  3454:   const uint64_t F = MI.getDesc().TSFlags;
  3455:   return (F >> HexagonII::ExtendableOpPos) & HexagonII::ExtendableOpMask;
  3456: }
  3457: 
  3458: // See if instruction could potentially be a duplex candidate.
  3459: // If so, return its group. Zero otherwise.
  3460: HexagonII::CompoundGroup HexagonInstrInfo::getCompoundCandidateGroup(
  3461:       const MachineInstr &MI) const {
  3462:   Register DstReg, SrcReg, Src1Reg, Src2Reg;
  3463: 
  3464:   switch (MI.getOpcode()) {
  3465:   default:
  3466:     return HexagonII::HCG_None;
  3467:   //
  3468:   // Compound pairs.
  3469:   // "p0=cmp.eq(Rs16,Rt16); if (p0.new) jump:nt #r9:2"
  3470:   // "Rd16=#U6 ; jump #r9:2"
  3471:   // "Rd16=Rs16 ; jump #r9:2"
  3472:   //
  3473:   case Hexagon::C2_cmpeq:
  3474:   case Hexagon::C2_cmpgt:
  3475:   case Hexagon::C2_cmpgtu:
  3476:     DstReg = MI.getOperand(0).getReg();
  3477:     Src1Reg = MI.getOperand(1).getReg();
  3478:     Src2Reg = MI.getOperand(2).getReg();
  3479:     if (Hexagon::PredRegsRegClass.contains(DstReg) &&
  3480:         (Hexagon::P0 == DstReg || Hexagon::P1 == DstReg) &&
  3481:         isIntRegForSubInst(Src1Reg) && isIntRegForSubInst(Src2Reg))
  3482:       return HexagonII::HCG_A;
  3483:     break;
  3484:   case Hexagon::C2_cmpeqi:
  3485:   case Hexagon::C2_cmpgti:
  3486:   case Hexagon::C2_cmpgtui:
  3487:     // P0 = cmp.eq(Rs,#u2)
  3488:     DstReg = MI.getOperand(0).getReg();
  3489:     SrcReg = MI.getOperand(1).getReg();
  3490:     if (Hexagon::PredRegsRegClass.contains(DstReg) &&
  3491:         (Hexagon::P0 == DstReg || Hexagon::P1 == DstReg) &&
  3492:         isIntRegForSubInst(SrcReg) && MI.getOperand(2).isImm() &&
  3493:         ((isUInt<5>(MI.getOperand(2).getImm())) ||
  3494:          (MI.getOperand(2).getImm() == -1)))
  3495:       return HexagonII::HCG_A;
  3496:     break;
  3497:   case Hexagon::A2_tfr:
  3498:     // Rd = Rs
  3499:     DstReg = MI.getOperand(0).getReg();
  3500:     SrcReg = MI.getOperand(1).getReg();
  3501:     if (isIntRegForSubInst(DstReg) && isIntRegForSubInst(SrcReg))
  3502:       return HexagonII::HCG_A;
  3503:     break;
  3504:   case Hexagon::A2_tfrsi:
  3505:     // Rd = #u6
  3506:     // Do not test for #u6 size since the const is getting extended
  3507:     // regardless and compound could be formed.
  3508:     DstReg = MI.getOperand(0).getReg();
  3509:     if (isIntRegForSubInst(DstReg))
  3510:       return HexagonII::HCG_A;
  3511:     break;
  3512:   case Hexagon::S2_tstbit_i:
  3513:     DstReg = MI.getOperand(0).getReg();
  3514:     Src1Reg = MI.getOperand(1).getReg();
  3515:     if (Hexagon::PredRegsRegClass.contains(DstReg) &&
  3516:         (Hexagon::P0 == DstReg || Hexagon::P1 == DstReg) &&
  3517:         MI.getOperand(2).isImm() &&
  3518:         isIntRegForSubInst(Src1Reg) && (MI.getOperand(2).getImm() == 0))
  3519:       return HexagonII::HCG_A;
  3520:     break;
  3521:   // The fact that .new form is used pretty much guarantees
  3522:   // that predicate register will match. Nevertheless,
  3523:   // there could be some false positives without additional
  3524:   // checking.
  3525:   case Hexagon::J2_jumptnew:
  3526:   case Hexagon::J2_jumpfnew:
  3527:   case Hexagon::J2_jumptnewpt:
  3528:   case Hexagon::J2_jumpfnewpt:
  3529:     Src1Reg = MI.getOperand(0).getReg();
  3530:     if (Hexagon::PredRegsRegClass.contains(Src1Reg) &&
  3531:         (Hexagon::P0 == Src1Reg || Hexagon::P1 == Src1Reg))
  3532:       return HexagonII::HCG_B;
  3533:     break;
  3534:   // Transfer and jump:
  3535:   // Rd=#U6 ; jump #r9:2
  3536:   // Rd=Rs ; jump #r9:2
  3537:   // Do not test for jump range here.
  3538:   case Hexagon::J2_jump:
  3539:   case Hexagon::RESTORE_DEALLOC_RET_JMP_V4:
  3540:   case Hexagon::RESTORE_DEALLOC_RET_JMP_V4_PIC:
  3541:     return HexagonII::HCG_C;
  3542:   }
  3543: 
  3544:   return HexagonII::HCG_None;
  3545: }
  3546: 
  3547: // Returns -1 when there is no opcode found.
  3548: unsigned HexagonInstrInfo::getCompoundOpcode(const MachineInstr &GA,
  3549:       const MachineInstr &GB) const {
  3550:   assert(getCompoundCandidateGroup(GA) == HexagonII::HCG_A);
  3551:   assert(getCompoundCandidateGroup(GB) == HexagonII::HCG_B);
  3552:   if ((GA.getOpcode() != Hexagon::C2_cmpeqi) ||
  3553:       (GB.getOpcode() != Hexagon::J2_jumptnew))
  3554:     return -1u;
  3555:   Register DestReg = GA.getOperand(0).getReg();
  3556:   if (!GB.readsRegister(DestReg, /*TRI=*/nullptr))
  3557:     return -1u;
  3558:   if (DestReg != Hexagon::P0 && DestReg != Hexagon::P1)
  3559:     return -1u;
  3560:   // The value compared against must be either u5 or -1.
  3561:   const MachineOperand &CmpOp = GA.getOperand(2);
  3562:   if (!CmpOp.isImm())
  3563:     return -1u;
  3564:   int V = CmpOp.getImm();
  3565:   if (V == -1)
  3566:     return DestReg == Hexagon::P0 ? Hexagon::J4_cmpeqn1_tp0_jump_nt
  3567:                                   : Hexagon::J4_cmpeqn1_tp1_jump_nt;
  3568:   if (!isUInt<5>(V))
  3569:     return -1u;
  3570:   return DestReg == Hexagon::P0 ? Hexagon::J4_cmpeqi_tp0_jump_nt
  3571:                                 : Hexagon::J4_cmpeqi_tp1_jump_nt;
  3572: }
  3573: 
  3574: // Returns -1 if there is no opcode found.
  3575: int HexagonInstrInfo::getDuplexOpcode(const MachineInstr &MI,
  3576:                                       bool ForBigCore) const {
  3577:   // Static table to switch the opcodes across Tiny Core and Big Core.
  3578:   // dup_ opcodes are Big core opcodes.
  3579:   // NOTE: There are special instructions that need to handled later.
  3580:   // L4_return* instructions, they will only occupy SLOT0 (on big core too).
  3581:   // PS_jmpret - This pseudo translates to J2_jumpr which occupies only SLOT2.
  3582:   // The compiler need to base the root instruction to L6_return_map_to_raw
  3583:   // which can go any slot.
  3584:   static const std::map<unsigned, unsigned> DupMap = {
  3585:       {Hexagon::A2_add, Hexagon::dup_A2_add},
  3586:       {Hexagon::A2_addi, Hexagon::dup_A2_addi},
  3587:       {Hexagon::A2_andir, Hexagon::dup_A2_andir},
  3588:       {Hexagon::A2_combineii, Hexagon::dup_A2_combineii},
  3589:       {Hexagon::A2_sxtb, Hexagon::dup_A2_sxtb},
  3590:       {Hexagon::A2_sxth, Hexagon::dup_A2_sxth},
  3591:       {Hexagon::A2_tfr, Hexagon::dup_A2_tfr},
  3592:       {Hexagon::A2_tfrsi, Hexagon::dup_A2_tfrsi},
  3593:       {Hexagon::A2_zxtb, Hexagon::dup_A2_zxtb},
  3594:       {Hexagon::A2_zxth, Hexagon::dup_A2_zxth},
  3595:       {Hexagon::A4_combineii, Hexagon::dup_A4_combineii},
  3596:       {Hexagon::A4_combineir, Hexagon::dup_A4_combineir},
  3597:       {Hexagon::A4_combineri, Hexagon::dup_A4_combineri},
  3598:       {Hexagon::C2_cmoveif, Hexagon::dup_C2_cmoveif},
  3599:       {Hexagon::C2_cmoveit, Hexagon::dup_C2_cmoveit},
  3600:       {Hexagon::C2_cmovenewif, Hexagon::dup_C2_cmovenewif},
```
- EN: It declares or implements routines such as instr_end, push_back, HexagonInstrInfo::getCExtOpNum, HexagonInstrInfo::getCompoundCandidateGroup, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 instr_end, push_back, HexagonInstrInfo::getCExtOpNum, HexagonInstrInfo::getCompoundCandidateGroup, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 3601-3800 / 第 3601-3800 行

```cpp
  3601:       {Hexagon::C2_cmovenewit, Hexagon::dup_C2_cmovenewit},
  3602:       {Hexagon::C2_cmpeqi, Hexagon::dup_C2_cmpeqi},
  3603:       {Hexagon::L2_deallocframe, Hexagon::dup_L2_deallocframe},
  3604:       {Hexagon::L2_loadrb_io, Hexagon::dup_L2_loadrb_io},
  3605:       {Hexagon::L2_loadrd_io, Hexagon::dup_L2_loadrd_io},
  3606:       {Hexagon::L2_loadrh_io, Hexagon::dup_L2_loadrh_io},
  3607:       {Hexagon::L2_loadri_io, Hexagon::dup_L2_loadri_io},
  3608:       {Hexagon::L2_loadrub_io, Hexagon::dup_L2_loadrub_io},
  3609:       {Hexagon::L2_loadruh_io, Hexagon::dup_L2_loadruh_io},
  3610:       {Hexagon::S2_allocframe, Hexagon::dup_S2_allocframe},
  3611:       {Hexagon::S2_storerb_io, Hexagon::dup_S2_storerb_io},
  3612:       {Hexagon::S2_storerd_io, Hexagon::dup_S2_storerd_io},
  3613:       {Hexagon::S2_storerh_io, Hexagon::dup_S2_storerh_io},
  3614:       {Hexagon::S2_storeri_io, Hexagon::dup_S2_storeri_io},
  3615:       {Hexagon::S4_storeirb_io, Hexagon::dup_S4_storeirb_io},
  3616:       {Hexagon::S4_storeiri_io, Hexagon::dup_S4_storeiri_io},
  3617:   };
  3618:   unsigned OpNum = MI.getOpcode();
  3619:   // Conversion to Big core.
  3620:   if (ForBigCore) {
  3621:     auto Iter = DupMap.find(OpNum);
  3622:     if (Iter != DupMap.end())
  3623:       return Iter->second;
  3624:   } else { // Conversion to Tiny core.
  3625:     for (const auto &Iter : DupMap)
  3626:       if (Iter.second == OpNum)
  3627:         return Iter.first;
  3628:   }
  3629:   return -1;
  3630: }
  3631: 
  3632: int HexagonInstrInfo::getCondOpcode(int Opc, bool invertPredicate) const {
  3633:   enum Hexagon::PredSense inPredSense;
  3634:   inPredSense = invertPredicate ? Hexagon::PredSense_false :
  3635:                                   Hexagon::PredSense_true;
  3636:   int CondOpcode = Hexagon::getPredOpcode(Opc, inPredSense);
  3637:   if (CondOpcode >= 0) // Valid Conditional opcode/instruction
  3638:     return CondOpcode;
  3639: 
  3640:   llvm_unreachable("Unexpected predicable instruction");
  3641: }
  3642: 
  3643: // Return the cur value instruction for a given store.
  3644: int HexagonInstrInfo::getDotCurOp(const MachineInstr &MI) const {
  3645:   switch (MI.getOpcode()) {
  3646:   default: llvm_unreachable("Unknown .cur type");
  3647:   case Hexagon::V6_vL32b_pi:
  3648:     return Hexagon::V6_vL32b_cur_pi;
  3649:   case Hexagon::V6_vL32b_ai:
  3650:     return Hexagon::V6_vL32b_cur_ai;
  3651:   case Hexagon::V6_vL32b_nt_pi:
  3652:     return Hexagon::V6_vL32b_nt_cur_pi;
  3653:   case Hexagon::V6_vL32b_nt_ai:
  3654:     return Hexagon::V6_vL32b_nt_cur_ai;
  3655:   case Hexagon::V6_vL32b_ppu:
  3656:     return Hexagon::V6_vL32b_cur_ppu;
  3657:   case Hexagon::V6_vL32b_nt_ppu:
  3658:     return Hexagon::V6_vL32b_nt_cur_ppu;
  3659:   }
  3660:   return 0;
  3661: }
  3662: 
  3663: // Return the regular version of the .cur instruction.
  3664: int HexagonInstrInfo::getNonDotCurOp(const MachineInstr &MI) const {
  3665:   switch (MI.getOpcode()) {
  3666:   default: llvm_unreachable("Unknown .cur type");
  3667:   case Hexagon::V6_vL32b_cur_pi:
  3668:     return Hexagon::V6_vL32b_pi;
  3669:   case Hexagon::V6_vL32b_cur_ai:
  3670:     return Hexagon::V6_vL32b_ai;
  3671:   case Hexagon::V6_vL32b_nt_cur_pi:
  3672:     return Hexagon::V6_vL32b_nt_pi;
  3673:   case Hexagon::V6_vL32b_nt_cur_ai:
  3674:     return Hexagon::V6_vL32b_nt_ai;
  3675:   case Hexagon::V6_vL32b_cur_ppu:
  3676:     return Hexagon::V6_vL32b_ppu;
  3677:   case Hexagon::V6_vL32b_nt_cur_ppu:
  3678:     return Hexagon::V6_vL32b_nt_ppu;
  3679:   }
  3680:   return 0;
  3681: }
  3682: 
  3683: // The diagram below shows the steps involved in the conversion of a predicated
  3684: // store instruction to its .new predicated new-value form.
  3685: //
  3686: // Note: It doesn't include conditional new-value stores as they can't be
  3687: // converted to .new predicate.
  3688: //
  3689: //               p.new NV store [ if(p0.new)memw(R0+#0)=R2.new ]
  3690: //                ^           ^
  3691: //               /             \ (not OK. it will cause new-value store to be
  3692: //              /               X conditional on p0.new while R2 producer is
  3693: //             /                 \ on p0)
  3694: //            /                   \.
  3695: //     p.new store                 p.old NV store
  3696: // [if(p0.new)memw(R0+#0)=R2]    [if(p0)memw(R0+#0)=R2.new]
  3697: //            ^                  ^
  3698: //             \                /
  3699: //              \              /
  3700: //               \            /
  3701: //                 p.old store
  3702: //             [if (p0)memw(R0+#0)=R2]
  3703: //
  3704: // The following set of instructions further explains the scenario where
  3705: // conditional new-value store becomes invalid when promoted to .new predicate
  3706: // form.
  3707: //
  3708: // { 1) if (p0) r0 = add(r1, r2)
  3709: //   2) p0 = cmp.eq(r3, #0) }
  3710: //
  3711: //   3) if (p0) memb(r1+#0) = r0  --> this instruction can't be grouped with
  3712: // the first two instructions because in instr 1, r0 is conditional on old value
  3713: // of p0 but its use in instr 3 is conditional on p0 modified by instr 2 which
  3714: // is not valid for new-value stores.
  3715: // Predicated new value stores (i.e. if (p0) memw(..)=r0.new) are excluded
  3716: // from the "Conditional Store" list. Because a predicated new value store
  3717: // would NOT be promoted to a double dot new store. See diagram below:
  3718: // This function returns yes for those stores that are predicated but not
  3719: // yet promoted to predicate dot new instructions.
  3720: //
  3721: //                          +---------------------+
  3722: //                    /-----| if (p0) memw(..)=r0 |---------\~
  3723: //                   ||     +---------------------+         ||
  3724: //          promote  ||       /\       /\                   ||  promote
  3725: //                   ||      /||\     /||\                  ||
  3726: //                  \||/    demote     ||                  \||/
  3727: //                   \/       ||       ||                   \/
  3728: //       +-------------------------+   ||   +-------------------------+
  3729: //       | if (p0.new) memw(..)=r0 |   ||   | if (p0) memw(..)=r0.new |
  3730: //       +-------------------------+   ||   +-------------------------+
  3731: //                        ||           ||         ||
  3732: //                        ||         demote      \||/
  3733: //                      promote        ||         \/ NOT possible
  3734: //                        ||           ||         /\~
  3735: //                       \||/          ||        /||\~
  3736: //                        \/           ||         ||
  3737: //                      +-----------------------------+
  3738: //                      | if (p0.new) memw(..)=r0.new |
  3739: //                      +-----------------------------+
  3740: //                           Double Dot New Store
  3741: //
  3742: // Returns the most basic instruction for the .new predicated instructions and
  3743: // new-value stores.
  3744: // For example, all of the following instructions will be converted back to the
  3745: // same instruction:
  3746: // 1) if (p0.new) memw(R0+#0) = R1.new  --->
  3747: // 2) if (p0) memw(R0+#0)= R1.new      -------> if (p0) memw(R0+#0) = R1
  3748: // 3) if (p0.new) memw(R0+#0) = R1      --->
  3749: //
  3750: // To understand the translation of instruction 1 to its original form, consider
  3751: // a packet with 3 instructions.
  3752: // { p0 = cmp.eq(R0,R1)
  3753: //   if (p0.new) R2 = add(R3, R4)
  3754: //   R5 = add (R3, R1)
  3755: // }
  3756: // if (p0) memw(R5+#0) = R2 <--- trying to include it in the previous packet
  3757: //
  3758: // This instruction can be part of the previous packet only if both p0 and R2
  3759: // are promoted to .new values. This promotion happens in steps, first
  3760: // predicate register is promoted to .new and in the next iteration R2 is
  3761: // promoted. Therefore, in case of dependence check failure (due to R5) during
  3762: // next iteration, it should be converted back to its most basic form.
  3763: 
  3764: // Return the new value instruction for a given store.
  3765: int HexagonInstrInfo::getDotNewOp(const MachineInstr &MI) const {
  3766:   int NVOpcode = Hexagon::getNewValueOpcode(MI.getOpcode());
  3767:   if (NVOpcode >= 0) // Valid new-value store instruction.
  3768:     return NVOpcode;
  3769: 
  3770:   switch (MI.getOpcode()) {
  3771:   default:
  3772:     report_fatal_error(Twine("Unknown .new type: ") +
  3773:                        std::to_string(MI.getOpcode()));
  3774:   case Hexagon::S4_storerb_ur:
  3775:     return Hexagon::S4_storerbnew_ur;
  3776: 
  3777:   case Hexagon::S2_storerb_pci:
  3778:     return Hexagon::S2_storerb_pci;
  3779: 
  3780:   case Hexagon::S2_storeri_pci:
  3781:     return Hexagon::S2_storeri_pci;
  3782: 
  3783:   case Hexagon::S2_storerh_pci:
  3784:     return Hexagon::S2_storerh_pci;
  3785: 
  3786:   case Hexagon::S2_storerd_pci:
  3787:     return Hexagon::S2_storerd_pci;
  3788: 
  3789:   case Hexagon::S2_storerf_pci:
  3790:     return Hexagon::S2_storerf_pci;
  3791: 
  3792:   case Hexagon::V6_vS32b_ai:
  3793:     return Hexagon::V6_vS32b_new_ai;
  3794: 
  3795:   case Hexagon::V6_vS32b_pi:
  3796:     return Hexagon::V6_vS32b_new_pi;
  3797:   }
  3798:   return 0;
  3799: }
  3800: 
```
- EN: It declares types such as Hexagon::PredSense, which carry the state or API of this component. It declares or implements routines such as getOpcode, find, HexagonInstrInfo::getCondOpcode, Hexagon::getPredOpcode, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明了 Hexagon::PredSense 等类型，用来承载该组件的状态或接口。 这里声明或实现了 getOpcode, find, HexagonInstrInfo::getCondOpcode, Hexagon::getPredOpcode, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 3801-4000 / 第 3801-4000 行

```cpp
  3801: // Returns the opcode to use when converting MI, which is a conditional jump,
  3802: // into a conditional instruction which uses the .new value of the predicate.
  3803: // We also use branch probabilities to add a hint to the jump.
  3804: // If MBPI is null, all edges will be treated as equally likely for the
  3805: // purposes of establishing a predication hint.
  3806: int HexagonInstrInfo::getDotNewPredJumpOp(const MachineInstr &MI,
  3807:       const MachineBranchProbabilityInfo *MBPI) const {
  3808:   // We assume that block can have at most two successors.
  3809:   const MachineBasicBlock *Src = MI.getParent();
  3810:   const MachineOperand &BrTarget = MI.getOperand(1);
  3811:   bool Taken = false;
  3812:   const BranchProbability OneHalf(1, 2);
  3813: 
  3814:   auto getEdgeProbability = [MBPI] (const MachineBasicBlock *Src,
  3815:                                     const MachineBasicBlock *Dst) {
  3816:     if (MBPI)
  3817:       return MBPI->getEdgeProbability(Src, Dst);
  3818:     return BranchProbability(1, Src->succ_size());
  3819:   };
  3820: 
  3821:   if (BrTarget.isMBB()) {
  3822:     const MachineBasicBlock *Dst = BrTarget.getMBB();
  3823:     Taken = getEdgeProbability(Src, Dst) >= OneHalf;
  3824:   } else {
  3825:     // The branch target is not a basic block (most likely a function).
  3826:     // Since BPI only gives probabilities for targets that are basic blocks,
  3827:     // try to identify another target of this branch (potentially a fall-
  3828:     // -through) and check the probability of that target.
  3829:     //
  3830:     // The only handled branch combinations are:
  3831:     // - one conditional branch,
  3832:     // - one conditional branch followed by one unconditional branch.
  3833:     // Otherwise, assume not-taken.
  3834:     assert(MI.isConditionalBranch());
  3835:     const MachineBasicBlock &B = *MI.getParent();
  3836:     bool SawCond = false, Bad = false;
  3837:     for (const MachineInstr &I : B) {
  3838:       if (!I.isBranch())
  3839:         continue;
  3840:       if (I.isConditionalBranch()) {
  3841:         SawCond = true;
  3842:         if (&I != &MI) {
  3843:           Bad = true;
  3844:           break;
  3845:         }
  3846:       }
  3847:       if (I.isUnconditionalBranch() && !SawCond) {
  3848:         Bad = true;
  3849:         break;
  3850:       }
  3851:     }
  3852:     if (!Bad) {
  3853:       MachineBasicBlock::const_instr_iterator It(MI);
  3854:       MachineBasicBlock::const_instr_iterator NextIt = std::next(It);
  3855:       if (NextIt == B.instr_end()) {
  3856:         // If this branch is the last, look for the fall-through block.
  3857:         for (const MachineBasicBlock *SB : B.successors()) {
  3858:           if (!B.isLayoutSuccessor(SB))
  3859:             continue;
  3860:           Taken = getEdgeProbability(Src, SB) < OneHalf;
  3861:           break;
  3862:         }
  3863:       } else {
  3864:         assert(NextIt->isUnconditionalBranch());
  3865:         // Find the first MBB operand and assume it's the target.
  3866:         const MachineBasicBlock *BT = nullptr;
  3867:         for (const MachineOperand &Op : NextIt->operands()) {
  3868:           if (!Op.isMBB())
  3869:             continue;
  3870:           BT = Op.getMBB();
  3871:           break;
  3872:         }
  3873:         Taken = BT && getEdgeProbability(Src, BT) < OneHalf;
  3874:       }
  3875:     } // if (!Bad)
  3876:   }
  3877: 
  3878:   // The Taken flag should be set to something reasonable by this point.
  3879: 
  3880:   switch (MI.getOpcode()) {
  3881:   case Hexagon::J2_jumpt:
  3882:     return Taken ? Hexagon::J2_jumptnewpt : Hexagon::J2_jumptnew;
  3883:   case Hexagon::J2_jumpf:
  3884:     return Taken ? Hexagon::J2_jumpfnewpt : Hexagon::J2_jumpfnew;
  3885: 
  3886:   default:
  3887:     llvm_unreachable("Unexpected jump instruction.");
  3888:   }
  3889: }
  3890: 
  3891: // Return .new predicate version for an instruction.
  3892: int HexagonInstrInfo::getDotNewPredOp(const MachineInstr &MI,
  3893:       const MachineBranchProbabilityInfo *MBPI) const {
  3894:   switch (MI.getOpcode()) {
  3895:   // Conditional Jumps
  3896:   case Hexagon::J2_jumpt:
  3897:   case Hexagon::J2_jumpf:
  3898:     return getDotNewPredJumpOp(MI, MBPI);
  3899:   }
  3900: 
  3901:   int NewOpcode = Hexagon::getPredNewOpcode(MI.getOpcode());
  3902:   if (NewOpcode >= 0)
  3903:     return NewOpcode;
  3904:   return 0;
  3905: }
  3906: 
  3907: int HexagonInstrInfo::getDotOldOp(const MachineInstr &MI) const {
  3908:   int NewOp = MI.getOpcode();
  3909:   if (isPredicated(NewOp) && isPredicatedNew(NewOp)) { // Get predicate old form
  3910:     NewOp = Hexagon::getPredOldOpcode(NewOp);
  3911:     // All Hexagon architectures have prediction bits on dot-new branches,
  3912:     // but only Hexagon V60+ has prediction bits on dot-old ones. Make sure
  3913:     // to pick the right opcode when converting back to dot-old.
  3914:     if (!Subtarget.hasFeature(Hexagon::ArchV60)) {
  3915:       switch (NewOp) {
  3916:       case Hexagon::J2_jumptpt:
  3917:         NewOp = Hexagon::J2_jumpt;
  3918:         break;
  3919:       case Hexagon::J2_jumpfpt:
  3920:         NewOp = Hexagon::J2_jumpf;
  3921:         break;
  3922:       case Hexagon::J2_jumprtpt:
  3923:         NewOp = Hexagon::J2_jumprt;
  3924:         break;
  3925:       case Hexagon::J2_jumprfpt:
  3926:         NewOp = Hexagon::J2_jumprf;
  3927:         break;
  3928:       }
  3929:     }
  3930:     assert(NewOp >= 0 &&
  3931:            "Couldn't change predicate new instruction to its old form.");
  3932:   }
  3933: 
  3934:   if (isNewValueStore(NewOp)) { // Convert into non-new-value format
  3935:     NewOp = Hexagon::getNonNVStore(NewOp);
  3936:     assert(NewOp >= 0 && "Couldn't change new-value store to its old form.");
  3937:   }
  3938: 
  3939:   if (Subtarget.hasV60Ops())
  3940:     return NewOp;
  3941: 
  3942:   // Subtargets prior to V60 didn't support 'taken' forms of predicated jumps.
  3943:   switch (NewOp) {
  3944:   case Hexagon::J2_jumpfpt:
  3945:     return Hexagon::J2_jumpf;
  3946:   case Hexagon::J2_jumptpt:
  3947:     return Hexagon::J2_jumpt;
  3948:   case Hexagon::J2_jumprfpt:
  3949:     return Hexagon::J2_jumprf;
  3950:   case Hexagon::J2_jumprtpt:
  3951:     return Hexagon::J2_jumprt;
  3952:   }
  3953:   return NewOp;
  3954: }
  3955: 
  3956: // See if instruction could potentially be a duplex candidate.
  3957: // If so, return its group. Zero otherwise.
  3958: HexagonII::SubInstructionGroup HexagonInstrInfo::getDuplexCandidateGroup(
  3959:       const MachineInstr &MI) const {
  3960:   Register DstReg, SrcReg, Src1Reg, Src2Reg;
  3961:   const HexagonRegisterInfo &HRI = *Subtarget.getRegisterInfo();
  3962: 
  3963:   switch (MI.getOpcode()) {
  3964:   default:
  3965:     return HexagonII::HSIG_None;
  3966:   //
  3967:   // Group L1:
  3968:   //
  3969:   // Rd = memw(Rs+#u4:2)
  3970:   // Rd = memub(Rs+#u4:0)
  3971:   case Hexagon::L2_loadri_io:
  3972:   case Hexagon::dup_L2_loadri_io:
  3973:     DstReg = MI.getOperand(0).getReg();
  3974:     SrcReg = MI.getOperand(1).getReg();
  3975:     // Special case this one from Group L2.
  3976:     // Rd = memw(r29+#u5:2)
  3977:     if (isIntRegForSubInst(DstReg)) {
  3978:       if (Hexagon::IntRegsRegClass.contains(SrcReg) &&
  3979:           HRI.getStackRegister() == SrcReg &&
  3980:           MI.getOperand(2).isImm() &&
  3981:           isShiftedUInt<5,2>(MI.getOperand(2).getImm()))
  3982:         return HexagonII::HSIG_L2;
  3983:       // Rd = memw(Rs+#u4:2)
  3984:       if (isIntRegForSubInst(SrcReg) &&
  3985:           (MI.getOperand(2).isImm() &&
  3986:           isShiftedUInt<4,2>(MI.getOperand(2).getImm())))
  3987:         return HexagonII::HSIG_L1;
  3988:     }
  3989:     break;
  3990:   case Hexagon::L2_loadrub_io:
  3991:   case Hexagon::dup_L2_loadrub_io:
  3992:     // Rd = memub(Rs+#u4:0)
  3993:     DstReg = MI.getOperand(0).getReg();
  3994:     SrcReg = MI.getOperand(1).getReg();
  3995:     if (isIntRegForSubInst(DstReg) && isIntRegForSubInst(SrcReg) &&
  3996:         MI.getOperand(2).isImm() && isUInt<4>(MI.getOperand(2).getImm()))
  3997:       return HexagonII::HSIG_L1;
  3998:     break;
  3999:   //
  4000:   // Group L2:
```
- EN: It declares or implements routines such as HexagonInstrInfo::getDotNewPredJumpOp, getParent, getOperand, OneHalf, ... (22 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonInstrInfo, HexagonII, HexagonRegisterInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonInstrInfo::getDotNewPredJumpOp, getParent, getOperand, OneHalf, ... (22 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo, HexagonII, HexagonRegisterInfo，说明了它与同级后端组件的连接关系。

### Lines 4001-4200 / 第 4001-4200 行

```cpp
  4001:   //
  4002:   // Rd = memh/memuh(Rs+#u3:1)
  4003:   // Rd = memb(Rs+#u3:0)
  4004:   // Rd = memw(r29+#u5:2) - Handled above.
  4005:   // Rdd = memd(r29+#u5:3)
  4006:   // deallocframe
  4007:   // [if ([!]p0[.new])] dealloc_return
  4008:   // [if ([!]p0[.new])] jumpr r31
  4009:   case Hexagon::L2_loadrh_io:
  4010:   case Hexagon::L2_loadruh_io:
  4011:   case Hexagon::dup_L2_loadrh_io:
  4012:   case Hexagon::dup_L2_loadruh_io:
  4013:     // Rd = memh/memuh(Rs+#u3:1)
  4014:     DstReg = MI.getOperand(0).getReg();
  4015:     SrcReg = MI.getOperand(1).getReg();
  4016:     if (isIntRegForSubInst(DstReg) && isIntRegForSubInst(SrcReg) &&
  4017:         MI.getOperand(2).isImm() &&
  4018:         isShiftedUInt<3,1>(MI.getOperand(2).getImm()))
  4019:       return HexagonII::HSIG_L2;
  4020:     break;
  4021:   case Hexagon::L2_loadrb_io:
  4022:   case Hexagon::dup_L2_loadrb_io:
  4023:     // Rd = memb(Rs+#u3:0)
  4024:     DstReg = MI.getOperand(0).getReg();
  4025:     SrcReg = MI.getOperand(1).getReg();
  4026:     if (isIntRegForSubInst(DstReg) && isIntRegForSubInst(SrcReg) &&
  4027:         MI.getOperand(2).isImm() &&
  4028:         isUInt<3>(MI.getOperand(2).getImm()))
  4029:       return HexagonII::HSIG_L2;
  4030:     break;
  4031:   case Hexagon::L2_loadrd_io:
  4032:   case Hexagon::dup_L2_loadrd_io:
  4033:     // Rdd = memd(r29+#u5:3)
  4034:     DstReg = MI.getOperand(0).getReg();
  4035:     SrcReg = MI.getOperand(1).getReg();
  4036:     if (isDblRegForSubInst(DstReg, HRI) &&
  4037:         Hexagon::IntRegsRegClass.contains(SrcReg) &&
  4038:         HRI.getStackRegister() == SrcReg &&
  4039:         MI.getOperand(2).isImm() &&
  4040:         isShiftedUInt<5,3>(MI.getOperand(2).getImm()))
  4041:       return HexagonII::HSIG_L2;
  4042:     break;
  4043:   // dealloc_return is not documented in Hexagon Manual, but marked
  4044:   // with A_SUBINSN attribute in iset_v4classic.py.
  4045:   case Hexagon::RESTORE_DEALLOC_RET_JMP_V4:
  4046:   case Hexagon::RESTORE_DEALLOC_RET_JMP_V4_PIC:
  4047:   case Hexagon::L4_return:
  4048:   case Hexagon::L2_deallocframe:
  4049:   case Hexagon::dup_L2_deallocframe:
  4050:     return HexagonII::HSIG_L2;
  4051:   case Hexagon::EH_RETURN_JMPR:
  4052:   case Hexagon::PS_jmpret:
  4053:   case Hexagon::SL2_jumpr31:
  4054:     // jumpr r31
  4055:     // Actual form JMPR implicit-def %pc, implicit %r31, implicit internal %r0
  4056:     DstReg = MI.getOperand(0).getReg();
  4057:     if (Hexagon::IntRegsRegClass.contains(DstReg) && (Hexagon::R31 == DstReg))
  4058:       return HexagonII::HSIG_L2;
  4059:     break;
  4060:   case Hexagon::PS_jmprett:
  4061:   case Hexagon::PS_jmpretf:
  4062:   case Hexagon::PS_jmprettnewpt:
  4063:   case Hexagon::PS_jmpretfnewpt:
  4064:   case Hexagon::PS_jmprettnew:
  4065:   case Hexagon::PS_jmpretfnew:
  4066:   case Hexagon::SL2_jumpr31_t:
  4067:   case Hexagon::SL2_jumpr31_f:
  4068:   case Hexagon::SL2_jumpr31_tnew:
  4069:   case Hexagon::SL2_jumpr31_fnew:
  4070:     DstReg = MI.getOperand(1).getReg();
  4071:     SrcReg = MI.getOperand(0).getReg();
  4072:     // [if ([!]p0[.new])] jumpr r31
  4073:     if ((Hexagon::PredRegsRegClass.contains(SrcReg) &&
  4074:         (Hexagon::P0 == SrcReg)) &&
  4075:         (Hexagon::IntRegsRegClass.contains(DstReg) && (Hexagon::R31 == DstReg)))
  4076:       return HexagonII::HSIG_L2;
  4077:     break;
  4078:   case Hexagon::L4_return_t:
  4079:   case Hexagon::L4_return_f:
  4080:   case Hexagon::L4_return_tnew_pnt:
  4081:   case Hexagon::L4_return_fnew_pnt:
  4082:   case Hexagon::L4_return_tnew_pt:
  4083:   case Hexagon::L4_return_fnew_pt:
  4084:     // [if ([!]p0[.new])] dealloc_return
  4085:     SrcReg = MI.getOperand(0).getReg();
  4086:     if (Hexagon::PredRegsRegClass.contains(SrcReg) && (Hexagon::P0 == SrcReg))
  4087:       return HexagonII::HSIG_L2;
  4088:     break;
  4089:   //
  4090:   // Group S1:
  4091:   //
  4092:   // memw(Rs+#u4:2) = Rt
  4093:   // memb(Rs+#u4:0) = Rt
  4094:   case Hexagon::S2_storeri_io:
  4095:   case Hexagon::dup_S2_storeri_io:
  4096:     // Special case this one from Group S2.
  4097:     // memw(r29+#u5:2) = Rt
  4098:     Src1Reg = MI.getOperand(0).getReg();
  4099:     Src2Reg = MI.getOperand(2).getReg();
  4100:     if (Hexagon::IntRegsRegClass.contains(Src1Reg) &&
  4101:         isIntRegForSubInst(Src2Reg) &&
  4102:         HRI.getStackRegister() == Src1Reg && MI.getOperand(1).isImm() &&
  4103:         isShiftedUInt<5,2>(MI.getOperand(1).getImm()))
  4104:       return HexagonII::HSIG_S2;
  4105:     // memw(Rs+#u4:2) = Rt
  4106:     if (isIntRegForSubInst(Src1Reg) && isIntRegForSubInst(Src2Reg) &&
  4107:         MI.getOperand(1).isImm() &&
  4108:         isShiftedUInt<4,2>(MI.getOperand(1).getImm()))
  4109:       return HexagonII::HSIG_S1;
  4110:     break;
  4111:   case Hexagon::S2_storerb_io:
  4112:   case Hexagon::dup_S2_storerb_io:
  4113:     // memb(Rs+#u4:0) = Rt
  4114:     Src1Reg = MI.getOperand(0).getReg();
  4115:     Src2Reg = MI.getOperand(2).getReg();
  4116:     if (isIntRegForSubInst(Src1Reg) && isIntRegForSubInst(Src2Reg) &&
  4117:         MI.getOperand(1).isImm() && isUInt<4>(MI.getOperand(1).getImm()))
  4118:       return HexagonII::HSIG_S1;
  4119:     break;
  4120:   //
  4121:   // Group S2:
  4122:   //
  4123:   // memh(Rs+#u3:1) = Rt
  4124:   // memw(r29+#u5:2) = Rt
  4125:   // memd(r29+#s6:3) = Rtt
  4126:   // memw(Rs+#u4:2) = #U1
  4127:   // memb(Rs+#u4) = #U1
  4128:   // allocframe(#u5:3)
  4129:   case Hexagon::S2_storerh_io:
  4130:   case Hexagon::dup_S2_storerh_io:
  4131:     // memh(Rs+#u3:1) = Rt
  4132:     Src1Reg = MI.getOperand(0).getReg();
  4133:     Src2Reg = MI.getOperand(2).getReg();
  4134:     if (isIntRegForSubInst(Src1Reg) && isIntRegForSubInst(Src2Reg) &&
  4135:         MI.getOperand(1).isImm() &&
  4136:         isShiftedUInt<3,1>(MI.getOperand(1).getImm()))
  4137:       return HexagonII::HSIG_S1;
  4138:     break;
  4139:   case Hexagon::S2_storerd_io:
  4140:   case Hexagon::dup_S2_storerd_io:
  4141:     // memd(r29+#s6:3) = Rtt
  4142:     Src1Reg = MI.getOperand(0).getReg();
  4143:     Src2Reg = MI.getOperand(2).getReg();
  4144:     if (isDblRegForSubInst(Src2Reg, HRI) &&
  4145:         Hexagon::IntRegsRegClass.contains(Src1Reg) &&
  4146:         HRI.getStackRegister() == Src1Reg && MI.getOperand(1).isImm() &&
  4147:         isShiftedInt<6,3>(MI.getOperand(1).getImm()))
  4148:       return HexagonII::HSIG_S2;
  4149:     break;
  4150:   case Hexagon::S4_storeiri_io:
  4151:   case Hexagon::dup_S4_storeiri_io:
  4152:     // memw(Rs+#u4:2) = #U1
  4153:     Src1Reg = MI.getOperand(0).getReg();
  4154:     if (isIntRegForSubInst(Src1Reg) && MI.getOperand(1).isImm() &&
  4155:         isShiftedUInt<4,2>(MI.getOperand(1).getImm()) &&
  4156:         MI.getOperand(2).isImm() && isUInt<1>(MI.getOperand(2).getImm()))
  4157:       return HexagonII::HSIG_S2;
  4158:     break;
  4159:   case Hexagon::S4_storeirb_io:
  4160:   case Hexagon::dup_S4_storeirb_io:
  4161:     // memb(Rs+#u4) = #U1
  4162:     Src1Reg = MI.getOperand(0).getReg();
  4163:     if (isIntRegForSubInst(Src1Reg) &&
  4164:         MI.getOperand(1).isImm() && isUInt<4>(MI.getOperand(1).getImm()) &&
  4165:         MI.getOperand(2).isImm() && isUInt<1>(MI.getOperand(2).getImm()))
  4166:       return HexagonII::HSIG_S2;
  4167:     break;
  4168:   case Hexagon::S2_allocframe:
  4169:   case Hexagon::dup_S2_allocframe:
  4170:     if (MI.getOperand(2).isImm() &&
  4171:         isShiftedUInt<5,3>(MI.getOperand(2).getImm()))
  4172:       return HexagonII::HSIG_S1;
  4173:     break;
  4174:   //
  4175:   // Group A:
  4176:   //
  4177:   // Rx = add(Rx,#s7)
  4178:   // Rd = Rs
  4179:   // Rd = #u6
  4180:   // Rd = #-1
  4181:   // if ([!]P0[.new]) Rd = #0
  4182:   // Rd = add(r29,#u6:2)
  4183:   // Rx = add(Rx,Rs)
  4184:   // P0 = cmp.eq(Rs,#u2)
  4185:   // Rdd = combine(#0,Rs)
  4186:   // Rdd = combine(Rs,#0)
  4187:   // Rdd = combine(#u2,#U2)
  4188:   // Rd = add(Rs,#1)
  4189:   // Rd = add(Rs,#-1)
  4190:   // Rd = sxth/sxtb/zxtb/zxth(Rs)
  4191:   // Rd = and(Rs,#1)
  4192:   case Hexagon::A2_addi:
  4193:   case Hexagon::dup_A2_addi:
  4194:     DstReg = MI.getOperand(0).getReg();
  4195:     SrcReg = MI.getOperand(1).getReg();
  4196:     if (isIntRegForSubInst(DstReg)) {
  4197:       // Rd = add(r29,#u6:2)
  4198:       if (Hexagon::IntRegsRegClass.contains(SrcReg) &&
  4199:         HRI.getStackRegister() == SrcReg && MI.getOperand(2).isImm() &&
  4200:         isShiftedUInt<6,2>(MI.getOperand(2).getImm()))
```
- EN: It declares or implements routines such as memuh, getOperand, memb, memd, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 memuh, getOperand, memb, memd, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII，说明了它与同级后端组件的连接关系。

### Lines 4201-4400 / 第 4201-4400 行

```cpp
  4201:         return HexagonII::HSIG_A;
  4202:       // Rx = add(Rx,#s7)
  4203:       if ((DstReg == SrcReg) && MI.getOperand(2).isImm() &&
  4204:           isInt<7>(MI.getOperand(2).getImm()))
  4205:         return HexagonII::HSIG_A;
  4206:       // Rd = add(Rs,#1)
  4207:       // Rd = add(Rs,#-1)
  4208:       if (isIntRegForSubInst(SrcReg) && MI.getOperand(2).isImm() &&
  4209:           ((MI.getOperand(2).getImm() == 1) ||
  4210:           (MI.getOperand(2).getImm() == -1)))
  4211:         return HexagonII::HSIG_A;
  4212:     }
  4213:     break;
  4214:   case Hexagon::A2_add:
  4215:   case Hexagon::dup_A2_add:
  4216:     // Rx = add(Rx,Rs)
  4217:     DstReg = MI.getOperand(0).getReg();
  4218:     Src1Reg = MI.getOperand(1).getReg();
  4219:     Src2Reg = MI.getOperand(2).getReg();
  4220:     if (isIntRegForSubInst(DstReg) && (DstReg == Src1Reg) &&
  4221:         isIntRegForSubInst(Src2Reg))
  4222:       return HexagonII::HSIG_A;
  4223:     break;
  4224:   case Hexagon::A2_andir:
  4225:   case Hexagon::dup_A2_andir:
  4226:     // Same as zxtb.
  4227:     // Rd16=and(Rs16,#255)
  4228:     // Rd16=and(Rs16,#1)
  4229:     DstReg = MI.getOperand(0).getReg();
  4230:     SrcReg = MI.getOperand(1).getReg();
  4231:     if (isIntRegForSubInst(DstReg) && isIntRegForSubInst(SrcReg) &&
  4232:         MI.getOperand(2).isImm() &&
  4233:         ((MI.getOperand(2).getImm() == 1) ||
  4234:         (MI.getOperand(2).getImm() == 255)))
  4235:       return HexagonII::HSIG_A;
  4236:     break;
  4237:   case Hexagon::A2_tfr:
  4238:   case Hexagon::dup_A2_tfr:
  4239:     // Rd = Rs
  4240:     DstReg = MI.getOperand(0).getReg();
  4241:     SrcReg = MI.getOperand(1).getReg();
  4242:     if (isIntRegForSubInst(DstReg) && isIntRegForSubInst(SrcReg))
  4243:       return HexagonII::HSIG_A;
  4244:     break;
  4245:   case Hexagon::A2_tfrsi:
  4246:   case Hexagon::dup_A2_tfrsi:
  4247:     // Rd = #u6
  4248:     // Do not test for #u6 size since the const is getting extended
  4249:     // regardless and compound could be formed.
  4250:     // Rd = #-1
  4251:     DstReg = MI.getOperand(0).getReg();
  4252:     if (isIntRegForSubInst(DstReg))
  4253:       return HexagonII::HSIG_A;
  4254:     break;
  4255:   case Hexagon::C2_cmoveit:
  4256:   case Hexagon::C2_cmovenewit:
  4257:   case Hexagon::C2_cmoveif:
  4258:   case Hexagon::C2_cmovenewif:
  4259:   case Hexagon::dup_C2_cmoveit:
  4260:   case Hexagon::dup_C2_cmovenewit:
  4261:   case Hexagon::dup_C2_cmoveif:
  4262:   case Hexagon::dup_C2_cmovenewif:
  4263:     // if ([!]P0[.new]) Rd = #0
  4264:     // Actual form:
  4265:     // %r16 = C2_cmovenewit internal %p0, 0, implicit undef %r16;
  4266:     DstReg = MI.getOperand(0).getReg();
  4267:     SrcReg = MI.getOperand(1).getReg();
  4268:     if (isIntRegForSubInst(DstReg) &&
  4269:         Hexagon::PredRegsRegClass.contains(SrcReg) && Hexagon::P0 == SrcReg &&
  4270:         MI.getOperand(2).isImm() && MI.getOperand(2).getImm() == 0)
  4271:       return HexagonII::HSIG_A;
  4272:     break;
  4273:   case Hexagon::C2_cmpeqi:
  4274:   case Hexagon::dup_C2_cmpeqi:
  4275:     // P0 = cmp.eq(Rs,#u2)
  4276:     DstReg = MI.getOperand(0).getReg();
  4277:     SrcReg = MI.getOperand(1).getReg();
  4278:     if (Hexagon::PredRegsRegClass.contains(DstReg) &&
  4279:         Hexagon::P0 == DstReg && isIntRegForSubInst(SrcReg) &&
  4280:         MI.getOperand(2).isImm() && isUInt<2>(MI.getOperand(2).getImm()))
  4281:       return HexagonII::HSIG_A;
  4282:     break;
  4283:   case Hexagon::A2_combineii:
  4284:   case Hexagon::A4_combineii:
  4285:   case Hexagon::dup_A2_combineii:
  4286:   case Hexagon::dup_A4_combineii:
  4287:     // Rdd = combine(#u2,#U2)
  4288:     DstReg = MI.getOperand(0).getReg();
  4289:     if (isDblRegForSubInst(DstReg, HRI) &&
  4290:         ((MI.getOperand(1).isImm() && isUInt<2>(MI.getOperand(1).getImm())) ||
  4291:         (MI.getOperand(1).isGlobal() &&
  4292:         isUInt<2>(MI.getOperand(1).getOffset()))) &&
  4293:         ((MI.getOperand(2).isImm() && isUInt<2>(MI.getOperand(2).getImm())) ||
  4294:         (MI.getOperand(2).isGlobal() &&
  4295:         isUInt<2>(MI.getOperand(2).getOffset()))))
  4296:       return HexagonII::HSIG_A;
  4297:     break;
  4298:   case Hexagon::A4_combineri:
  4299:   case Hexagon::dup_A4_combineri:
  4300:     // Rdd = combine(Rs,#0)
  4301:     // Rdd = combine(Rs,#0)
  4302:     DstReg = MI.getOperand(0).getReg();
  4303:     SrcReg = MI.getOperand(1).getReg();
  4304:     if (isDblRegForSubInst(DstReg, HRI) && isIntRegForSubInst(SrcReg) &&
  4305:         ((MI.getOperand(2).isImm() && MI.getOperand(2).getImm() == 0) ||
  4306:         (MI.getOperand(2).isGlobal() && MI.getOperand(2).getOffset() == 0)))
  4307:       return HexagonII::HSIG_A;
  4308:     break;
  4309:   case Hexagon::A4_combineir:
  4310:   case Hexagon::dup_A4_combineir:
  4311:     // Rdd = combine(#0,Rs)
  4312:     DstReg = MI.getOperand(0).getReg();
  4313:     SrcReg = MI.getOperand(2).getReg();
  4314:     if (isDblRegForSubInst(DstReg, HRI) && isIntRegForSubInst(SrcReg) &&
  4315:         ((MI.getOperand(1).isImm() && MI.getOperand(1).getImm() == 0) ||
  4316:         (MI.getOperand(1).isGlobal() && MI.getOperand(1).getOffset() == 0)))
  4317:       return HexagonII::HSIG_A;
  4318:     break;
  4319:   case Hexagon::A2_sxtb:
  4320:   case Hexagon::A2_sxth:
  4321:   case Hexagon::A2_zxtb:
  4322:   case Hexagon::A2_zxth:
  4323:   case Hexagon::dup_A2_sxtb:
  4324:   case Hexagon::dup_A2_sxth:
  4325:   case Hexagon::dup_A2_zxtb:
  4326:   case Hexagon::dup_A2_zxth:
  4327:     // Rd = sxth/sxtb/zxtb/zxth(Rs)
  4328:     DstReg = MI.getOperand(0).getReg();
  4329:     SrcReg = MI.getOperand(1).getReg();
  4330:     if (isIntRegForSubInst(DstReg) && isIntRegForSubInst(SrcReg))
  4331:       return HexagonII::HSIG_A;
  4332:     break;
  4333:   }
  4334: 
  4335:   return HexagonII::HSIG_None;
  4336: }
  4337: 
  4338: short HexagonInstrInfo::getEquivalentHWInstr(const MachineInstr &MI) const {
  4339:   return Hexagon::getRealHWInstr(MI.getOpcode(), Hexagon::InstrType_Real);
  4340: }
  4341: 
  4342: unsigned HexagonInstrInfo::getInstrTimingClassLatency(
  4343:       const InstrItineraryData *ItinData, const MachineInstr &MI) const {
  4344:   // Default to one cycle for no itinerary. However, an "empty" itinerary may
  4345:   // still have a MinLatency property, which getStageLatency checks.
  4346:   if (!ItinData)
  4347:     return getInstrLatency(ItinData, MI);
  4348: 
  4349:   if (MI.isTransient())
  4350:     return 0;
  4351:   return ItinData->getStageLatency(MI.getDesc().getSchedClass());
  4352: }
  4353: 
  4354: /// getOperandLatency - Compute and return the use operand latency of a given
  4355: /// pair of def and use.
  4356: /// In most cases, the static scheduling itinerary was enough to determine the
  4357: /// operand latency. But it may not be possible for instructions with variable
  4358: /// number of defs / uses.
  4359: ///
  4360: /// This is a raw interface to the itinerary that may be directly overridden by
  4361: /// a target. Use computeOperandLatency to get the best estimate of latency.
  4362: std::optional<unsigned> HexagonInstrInfo::getOperandLatency(
  4363:     const InstrItineraryData *ItinData, const MachineInstr &DefMI,
  4364:     unsigned DefIdx, const MachineInstr &UseMI, unsigned UseIdx) const {
  4365:   const HexagonRegisterInfo &HRI = *Subtarget.getRegisterInfo();
  4366: 
  4367:   // Get DefIdx and UseIdx for super registers.
  4368:   const MachineOperand &DefMO = DefMI.getOperand(DefIdx);
  4369: 
  4370:   if (DefMO.isReg() && DefMO.getReg().isPhysical()) {
  4371:     if (DefMO.isImplicit()) {
  4372:       for (MCPhysReg SR : HRI.superregs(DefMO.getReg())) {
  4373:         int Idx = DefMI.findRegisterDefOperandIdx(SR, &HRI, false, false);
  4374:         if (Idx != -1) {
  4375:           DefIdx = Idx;
  4376:           break;
  4377:         }
  4378:       }
  4379:     }
  4380: 
  4381:     const MachineOperand &UseMO = UseMI.getOperand(UseIdx);
  4382:     if (UseMO.isImplicit()) {
  4383:       for (MCPhysReg SR : HRI.superregs(UseMO.getReg())) {
  4384:         int Idx = UseMI.findRegisterUseOperandIdx(SR, &HRI, false);
  4385:         if (Idx != -1) {
  4386:           UseIdx = Idx;
  4387:           break;
  4388:         }
  4389:       }
  4390:     }
  4391:   }
  4392: 
  4393:   std::optional<unsigned> Latency = TargetInstrInfo::getOperandLatency(
  4394:       ItinData, DefMI, DefIdx, UseMI, UseIdx);
  4395:   if (Latency == 0)
  4396:     // We should never have 0 cycle latency between two instructions unless
  4397:     // they can be packetized together. However, this decision can't be made
  4398:     // here.
  4399:     Latency = 1;
  4400:   return Latency;
```
- EN: It declares or implements routines such as add, getOperand, and, eq, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, HexagonInstrInfo, HexagonRegisterInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 add, getOperand, and, eq, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII, HexagonInstrInfo, HexagonRegisterInfo，说明了它与同级后端组件的连接关系。

### Lines 4401-4600 / 第 4401-4600 行

```cpp
  4401: }
  4402: 
  4403: // inverts the predication logic.
  4404: // p -> NotP
  4405: // NotP -> P
  4406: bool HexagonInstrInfo::getInvertedPredSense(
  4407:       SmallVectorImpl<MachineOperand> &Cond) const {
  4408:   if (Cond.empty())
  4409:     return false;
  4410:   unsigned Opc = getInvertedPredicatedOpcode(Cond[0].getImm());
  4411:   Cond[0].setImm(Opc);
  4412:   return true;
  4413: }
  4414: 
  4415: unsigned HexagonInstrInfo::getInvertedPredicatedOpcode(const int Opc) const {
  4416:   int InvPredOpcode;
  4417:   InvPredOpcode = isPredicatedTrue(Opc) ? Hexagon::getFalsePredOpcode(Opc)
  4418:                                         : Hexagon::getTruePredOpcode(Opc);
  4419:   if (InvPredOpcode >= 0) // Valid instruction with the inverted predicate.
  4420:     return InvPredOpcode;
  4421: 
  4422:   llvm_unreachable("Unexpected predicated instruction");
  4423: }
  4424: 
  4425: // Returns the max value that doesn't need to be extended.
  4426: int HexagonInstrInfo::getMaxValue(const MachineInstr &MI) const {
  4427:   const uint64_t F = MI.getDesc().TSFlags;
  4428:   unsigned isSigned = (F >> HexagonII::ExtentSignedPos)
  4429:                     & HexagonII::ExtentSignedMask;
  4430:   unsigned bits =  (F >> HexagonII::ExtentBitsPos)
  4431:                     & HexagonII::ExtentBitsMask;
  4432: 
  4433:   if (isSigned) // if value is signed
  4434:     return ~(-1U << (bits - 1));
  4435:   else
  4436:     return ~(-1U << bits);
  4437: }
  4438: 
  4439: 
  4440: bool HexagonInstrInfo::isAddrModeWithOffset(const MachineInstr &MI) const {
  4441:   switch (MI.getOpcode()) {
  4442:   case Hexagon::L2_loadrbgp:
  4443:   case Hexagon::L2_loadrdgp:
  4444:   case Hexagon::L2_loadrhgp:
  4445:   case Hexagon::L2_loadrigp:
  4446:   case Hexagon::L2_loadrubgp:
  4447:   case Hexagon::L2_loadruhgp:
  4448:   case Hexagon::S2_storerbgp:
  4449:   case Hexagon::S2_storerbnewgp:
  4450:   case Hexagon::S2_storerhgp:
  4451:   case Hexagon::S2_storerhnewgp:
  4452:   case Hexagon::S2_storerigp:
  4453:   case Hexagon::S2_storerinewgp:
  4454:   case Hexagon::S2_storerdgp:
  4455:   case Hexagon::S2_storerfgp:
  4456:     return true;
  4457:   }
  4458:   const uint64_t F = MI.getDesc().TSFlags;
  4459:   unsigned addrMode =
  4460:     ((F >> HexagonII::AddrModePos) & HexagonII::AddrModeMask);
  4461:   // Disallow any base+offset instruction. The assembler does not yet reorder
  4462:   // based up any zero offset instruction.
  4463:   return (addrMode == HexagonII::BaseRegOffset ||
  4464:           addrMode == HexagonII::BaseImmOffset ||
  4465:           addrMode == HexagonII::BaseLongOffset);
  4466: }
  4467: 
  4468: bool HexagonInstrInfo::isPureSlot0(const MachineInstr &MI) const {
  4469:   // Workaround for the Global Scheduler. Sometimes, it creates
  4470:   // A4_ext as a Pseudo instruction and calls this function to see if
  4471:   // it can be added to an existing bundle. Since the instruction doesn't
  4472:   // belong to any BB yet, we can't use getUnits API.
  4473:   if (MI.getOpcode() == Hexagon::A4_ext)
  4474:     return false;
  4475: 
  4476:   unsigned FuncUnits = getUnits(MI);
  4477:   return HexagonFUnits::isSlot0Only(FuncUnits);
  4478: }
  4479: 
  4480: bool HexagonInstrInfo::isRestrictNoSlot1Store(const MachineInstr &MI) const {
  4481:   const uint64_t F = MI.getDesc().TSFlags;
  4482:   return ((F >> HexagonII::RestrictNoSlot1StorePos) &
  4483:           HexagonII::RestrictNoSlot1StoreMask);
  4484: }
  4485: 
  4486: void HexagonInstrInfo::changeDuplexOpcode(MachineBasicBlock::instr_iterator MII,
  4487:                                           bool ToBigInstrs) const {
  4488:   int Opcode = -1;
  4489:   if (ToBigInstrs) { // To BigCore Instr.
  4490:     // Check if the instruction can form a Duplex.
  4491:     if (getDuplexCandidateGroup(*MII))
  4492:       // Get the opcode marked "dup_*" tag.
  4493:       Opcode = getDuplexOpcode(*MII, ToBigInstrs);
  4494:   } else // To TinyCore Instr.
  4495:     Opcode = getDuplexOpcode(*MII, ToBigInstrs);
  4496: 
  4497:   // Change the opcode of the instruction.
  4498:   if (Opcode >= 0)
  4499:     MII->setDesc(get(Opcode));
  4500: }
  4501: 
  4502: // This function is used to translate instructions to facilitate generating
  4503: // Duplexes on TinyCore.
  4504: void HexagonInstrInfo::translateInstrsForDup(MachineFunction &MF,
  4505:                                              bool ToBigInstrs) const {
  4506:   for (auto &MB : MF)
  4507:     for (MachineBasicBlock::instr_iterator Instr = MB.instr_begin(),
  4508:                                            End = MB.instr_end();
  4509:          Instr != End; ++Instr)
  4510:       changeDuplexOpcode(Instr, ToBigInstrs);
  4511: }
  4512: 
  4513: // This is a specialized form of above function.
  4514: void HexagonInstrInfo::translateInstrsForDup(
  4515:     MachineBasicBlock::instr_iterator MII, bool ToBigInstrs) const {
  4516:   MachineBasicBlock *MBB = MII->getParent();
  4517:   while ((MII != MBB->instr_end()) && MII->isInsideBundle()) {
  4518:     changeDuplexOpcode(MII, ToBigInstrs);
  4519:     ++MII;
  4520:   }
  4521: }
  4522: 
  4523: unsigned HexagonInstrInfo::getMemAccessSize(const MachineInstr &MI) const {
  4524:   using namespace HexagonII;
  4525: 
  4526:   const uint64_t F = MI.getDesc().TSFlags;
  4527:   unsigned S = (F >> MemAccessSizePos) & MemAccesSizeMask;
  4528:   unsigned Size = getMemAccessSizeInBytes(MemAccessSize(S));
  4529:   if (Size != 0)
  4530:     return Size;
  4531:   // Y2_dcfetchbo is special
  4532:   if (MI.getOpcode() == Hexagon::Y2_dcfetchbo)
  4533:     return HexagonII::DoubleWordAccess;
  4534: 
  4535:   // Handle vector access sizes.
  4536:   const HexagonRegisterInfo &HRI = *Subtarget.getRegisterInfo();
  4537:   switch (S) {
  4538:     case HexagonII::HVXVectorAccess:
  4539:       return HRI.getSpillSize(Hexagon::HvxVRRegClass);
  4540:     default:
  4541:       llvm_unreachable("Unexpected instruction");
  4542:   }
  4543: }
  4544: 
  4545: // Returns the min value that doesn't need to be extended.
  4546: int HexagonInstrInfo::getMinValue(const MachineInstr &MI) const {
  4547:   const uint64_t F = MI.getDesc().TSFlags;
  4548:   unsigned isSigned = (F >> HexagonII::ExtentSignedPos)
  4549:                     & HexagonII::ExtentSignedMask;
  4550:   unsigned bits =  (F >> HexagonII::ExtentBitsPos)
  4551:                     & HexagonII::ExtentBitsMask;
  4552: 
  4553:   if (isSigned) // if value is signed
  4554:     return -1U << (bits - 1);
  4555:   else
  4556:     return 0;
  4557: }
  4558: 
  4559: // Returns opcode of the non-extended equivalent instruction.
  4560: short HexagonInstrInfo::getNonExtOpcode(const MachineInstr &MI) const {
  4561:   // Check if the instruction has a register form that uses register in place
  4562:   // of the extended operand, if so return that as the non-extended form.
  4563:   short NonExtOpcode = Hexagon::getRegForm(MI.getOpcode());
  4564:     if (NonExtOpcode >= 0)
  4565:       return NonExtOpcode;
  4566: 
  4567:   if (MI.getDesc().mayLoad() || MI.getDesc().mayStore()) {
  4568:     // Check addressing mode and retrieve non-ext equivalent instruction.
  4569:     switch (getAddrMode(MI)) {
  4570:     case HexagonII::Absolute:
  4571:       return Hexagon::changeAddrMode_abs_io(MI.getOpcode());
  4572:     case HexagonII::BaseImmOffset:
  4573:       return Hexagon::changeAddrMode_io_rr(MI.getOpcode());
  4574:     case HexagonII::BaseLongOffset:
  4575:       return Hexagon::changeAddrMode_ur_rr(MI.getOpcode());
  4576: 
  4577:     default:
  4578:       return -1;
  4579:     }
  4580:   }
  4581:   return -1;
  4582: }
  4583: 
  4584: bool HexagonInstrInfo::getPredReg(ArrayRef<MachineOperand> Cond,
  4585:                                   Register &PredReg, unsigned &PredRegPos,
  4586:                                   RegState &PredRegFlags) const {
  4587:   if (Cond.empty())
  4588:     return false;
  4589:   assert(Cond.size() == 2);
  4590:   if (isNewValueJump(Cond[0].getImm()) || Cond[1].isMBB()) {
  4591:     LLVM_DEBUG(dbgs() << "No predregs for new-value jumps/endloop");
  4592:     return false;
  4593:   }
  4594:   PredReg = Cond[1].getReg();
  4595:   PredRegPos = 1;
  4596:   // See IfConversion.cpp why we add RegState::Implicit | RegState::Undef
  4597:   PredRegFlags = {};
  4598:   if (Cond[1].isImplicit())
  4599:     PredRegFlags = RegState::Implicit;
  4600:   if (Cond[1].isUndef())
```
- EN: It opens namespaces (HexagonII) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as HexagonInstrInfo::getInvertedPredSense, getInvertedPredicatedOpcode, setImm, HexagonInstrInfo::getInvertedPredicatedOpcode, ... (31 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（HexagonII），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 HexagonInstrInfo::getInvertedPredSense, getInvertedPredicatedOpcode, setImm, HexagonInstrInfo::getInvertedPredicatedOpcode, ... (31 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 4601-4800 / 第 4601-4800 行

```cpp
  4601:     PredRegFlags |= RegState::Undef;
  4602:   return true;
  4603: }
  4604: 
  4605: short HexagonInstrInfo::getPseudoInstrPair(const MachineInstr &MI) const {
  4606:   return Hexagon::getRealHWInstr(MI.getOpcode(), Hexagon::InstrType_Pseudo);
  4607: }
  4608: 
  4609: short HexagonInstrInfo::getRegForm(const MachineInstr &MI) const {
  4610:   return Hexagon::getRegForm(MI.getOpcode());
  4611: }
  4612: 
  4613: // Return the number of bytes required to encode the instruction.
  4614: // Hexagon instructions are fixed length, 4 bytes, unless they
  4615: // use a constant extender, which requires another 4 bytes.
  4616: // For debug instructions and prolog labels, return 0.
  4617: unsigned HexagonInstrInfo::getSize(const MachineInstr &MI) const {
  4618:   if (MI.isDebugInstr() || MI.isPosition())
  4619:     return 0;
  4620: 
  4621:   unsigned Size = MI.getDesc().getSize();
  4622:   if (!Size)
  4623:     // Assume the default insn size in case it cannot be determined
  4624:     // for whatever reason.
  4625:     Size = HEXAGON_INSTR_SIZE;
  4626: 
  4627:   if (isConstExtended(MI) || isExtended(MI))
  4628:     Size += HEXAGON_INSTR_SIZE;
  4629: 
  4630:   // Try and compute number of instructions in asm.
  4631:   if (BranchRelaxAsmLarge && MI.getOpcode() == Hexagon::INLINEASM) {
  4632:     const MachineBasicBlock &MBB = *MI.getParent();
  4633:     const MachineFunction *MF = MBB.getParent();
  4634:     const MCAsmInfo &MAI = MF->getTarget().getMCAsmInfo();
  4635: 
  4636:     // Count the number of register definitions to find the asm string.
  4637:     unsigned NumDefs = 0;
  4638:     for (; MI.getOperand(NumDefs).isReg() && MI.getOperand(NumDefs).isDef();
  4639:          ++NumDefs)
  4640:       assert(NumDefs != MI.getNumOperands()-2 && "No asm string?");
  4641: 
  4642:     assert(MI.getOperand(NumDefs).isSymbol() && "No asm string?");
  4643:     // Disassemble the AsmStr and approximate number of instructions.
  4644:     const char *AsmStr = MI.getOperand(NumDefs).getSymbolName();
  4645:     Size = getInlineAsmLength(AsmStr, MAI);
  4646:   }
  4647: 
  4648:   return Size;
  4649: }
  4650: 
  4651: uint64_t HexagonInstrInfo::getType(const MachineInstr &MI) const {
  4652:   const uint64_t F = MI.getDesc().TSFlags;
  4653:   return (F >> HexagonII::TypePos) & HexagonII::TypeMask;
  4654: }
  4655: 
  4656: InstrStage::FuncUnits HexagonInstrInfo::getUnits(const MachineInstr &MI) const {
  4657:   const InstrItineraryData &II = *Subtarget.getInstrItineraryData();
  4658:   const InstrStage &IS = *II.beginStage(MI.getDesc().getSchedClass());
  4659: 
  4660:   return IS.getUnits();
  4661: }
  4662: 
  4663: // Calculate size of the basic block without debug instructions.
  4664: unsigned HexagonInstrInfo::nonDbgBBSize(const MachineBasicBlock *BB) const {
  4665:   return nonDbgMICount(BB->instr_begin(), BB->instr_end());
  4666: }
  4667: 
  4668: unsigned HexagonInstrInfo::nonDbgBundleSize(
  4669:       MachineBasicBlock::const_iterator BundleHead) const {
  4670:   assert(BundleHead->isBundle() && "Not a bundle header");
  4671:   auto MII = BundleHead.getInstrIterator();
  4672:   // Skip the bundle header.
  4673:   return nonDbgMICount(++MII, getBundleEnd(BundleHead.getInstrIterator()));
  4674: }
  4675: 
  4676: /// immediateExtend - Changes the instruction in place to one using an immediate
  4677: /// extender.
  4678: void HexagonInstrInfo::immediateExtend(MachineInstr &MI) const {
  4679:   assert((isExtendable(MI)||isConstExtended(MI)) &&
  4680:                                "Instruction must be extendable");
  4681:   // Find which operand is extendable.
  4682:   short ExtOpNum = getCExtOpNum(MI);
  4683:   MachineOperand &MO = MI.getOperand(ExtOpNum);
  4684:   // This needs to be something we understand.
  4685:   assert((MO.isMBB() || MO.isImm()) &&
  4686:          "Branch with unknown extendable field type");
  4687:   // Mark given operand as extended.
  4688:   MO.addTargetFlag(HexagonII::HMOTF_ConstExtended);
  4689: }
  4690: 
  4691: bool HexagonInstrInfo::invertAndChangeJumpTarget(
  4692:       MachineInstr &MI, MachineBasicBlock *NewTarget) const {
  4693:   LLVM_DEBUG(dbgs() << "\n[invertAndChangeJumpTarget] to "
  4694:                     << printMBBReference(*NewTarget);
  4695:              MI.dump(););
  4696:   assert(MI.isBranch());
  4697:   unsigned NewOpcode = getInvertedPredicatedOpcode(MI.getOpcode());
  4698:   int TargetPos = MI.getNumOperands() - 1;
  4699:   // In general branch target is the last operand,
  4700:   // but some implicit defs added at the end might change it.
  4701:   while ((TargetPos > -1) && !MI.getOperand(TargetPos).isMBB())
  4702:     --TargetPos;
  4703:   assert((TargetPos >= 0) && MI.getOperand(TargetPos).isMBB());
  4704:   MI.getOperand(TargetPos).setMBB(NewTarget);
  4705:   if (EnableBranchPrediction && isPredicatedNew(MI)) {
  4706:     NewOpcode = reversePrediction(NewOpcode);
  4707:   }
  4708:   MI.setDesc(get(NewOpcode));
  4709:   return true;
  4710: }
  4711: 
  4712: void HexagonInstrInfo::genAllInsnTimingClasses(MachineFunction &MF) const {
  4713:   /* +++ The code below is used to generate complete set of Hexagon Insn +++ */
  4714:   MachineFunction::iterator A = MF.begin();
  4715:   MachineBasicBlock &B = *A;
  4716:   MachineBasicBlock::iterator I = B.begin();
  4717:   DebugLoc DL = I->getDebugLoc();
  4718:   MachineInstr *NewMI;
  4719: 
  4720:   for (unsigned insn = TargetOpcode::GENERIC_OP_END+1;
  4721:        insn < Hexagon::INSTRUCTION_LIST_END; ++insn) {
  4722:     NewMI = BuildMI(B, I, DL, get(insn));
  4723:     LLVM_DEBUG(dbgs() << "\n"
  4724:                       << getName(NewMI->getOpcode())
  4725:                       << "  Class: " << NewMI->getDesc().getSchedClass());
  4726:     NewMI->eraseFromParent();
  4727:   }
  4728:   /* --- The code above is used to generate complete set of Hexagon Insn --- */
  4729: }
  4730: 
  4731: // inverts the predication logic.
  4732: // p -> NotP
  4733: // NotP -> P
  4734: bool HexagonInstrInfo::reversePredSense(MachineInstr &MI) const {
  4735:   LLVM_DEBUG(dbgs() << "\nTrying to reverse pred. sense of:"; MI.dump());
  4736:   MI.setDesc(get(getInvertedPredicatedOpcode(MI.getOpcode())));
  4737:   return true;
  4738: }
  4739: 
  4740: // Reverse the branch prediction.
  4741: unsigned HexagonInstrInfo::reversePrediction(unsigned Opcode) const {
  4742:   int PredRevOpcode = -1;
  4743:   if (isPredictedTaken(Opcode))
  4744:     PredRevOpcode = Hexagon::notTakenBranchPrediction(Opcode);
  4745:   else
  4746:     PredRevOpcode = Hexagon::takenBranchPrediction(Opcode);
  4747:   assert(PredRevOpcode > 0);
  4748:   return PredRevOpcode;
  4749: }
  4750: 
  4751: // TODO: Add more rigorous validation.
  4752: bool HexagonInstrInfo::validateBranchCond(const ArrayRef<MachineOperand> &Cond)
  4753:       const {
  4754:   return Cond.empty() || (Cond[0].isImm() && (Cond.size() != 1));
  4755: }
  4756: 
  4757: void HexagonInstrInfo::
  4758: setBundleNoShuf(MachineBasicBlock::instr_iterator MIB) const {
  4759:   assert(MIB->isBundle());
  4760:   MachineOperand &Operand = MIB->getOperand(0);
  4761:   if (Operand.isImm())
  4762:     Operand.setImm(Operand.getImm() | memShufDisabledMask);
  4763:   else
  4764:     MIB->addOperand(MachineOperand::CreateImm(memShufDisabledMask));
  4765: }
  4766: 
  4767: bool HexagonInstrInfo::getBundleNoShuf(const MachineInstr &MIB) const {
  4768:   assert(MIB.isBundle());
  4769:   const MachineOperand &Operand = MIB.getOperand(0);
  4770:   return (Operand.isImm() && (Operand.getImm() & memShufDisabledMask) != 0);
  4771: }
  4772: 
  4773: bool HexagonInstrInfo::isQFPMul(const MachineInstr *MI) const {
  4774:   return (MI->getOpcode() == Hexagon::V6_vmpy_qf16_hf ||
  4775:           MI->getOpcode() == Hexagon::V6_vmpy_qf16_mix_hf ||
  4776:           MI->getOpcode() == Hexagon::V6_vmpy_qf32_hf ||
  4777:           MI->getOpcode() == Hexagon::V6_vmpy_qf32_mix_hf ||
  4778:           MI->getOpcode() == Hexagon::V6_vmpy_qf32_sf ||
  4779:           MI->getOpcode() == Hexagon::V6_vmpy_qf16_mix_hf ||
  4780:           MI->getOpcode() == Hexagon::V6_vmpy_qf16 ||
  4781:           MI->getOpcode() == Hexagon::V6_vmpy_qf32_mix_hf ||
  4782:           MI->getOpcode() == Hexagon::V6_vmpy_qf32_qf16 ||
  4783:           MI->getOpcode() == Hexagon::V6_vmpy_qf32);
  4784: }
  4785: 
  4786: namespace llvm::HexagonII {
  4787: 
  4788: static constexpr RegTypeInfo make(RegType Out, RegType In1 = RegType::Unknown,
  4789:                                   RegType In2 = RegType::Unknown,
  4790:                                   RegType In3 = RegType::Unknown) {
  4791:   RegTypeInfo I;
  4792:   I.Output = Out;
  4793:   I.Input1 = In1;
  4794:   I.Input2 = In2;
  4795:   I.Input3 = In3;
  4796:   return I;
  4797: }
  4798: 
  4799: RegTypeInfo getRegTypeInfo(unsigned Opcode) {
  4800:   switch (Opcode) {
```
- EN: It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as HexagonInstrInfo::getPseudoInstrPair, Hexagon::getRealHWInstr, HexagonInstrInfo::getRegForm, Hexagon::getRegForm, ... (45 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 HexagonInstrInfo::getPseudoInstrPair, Hexagon::getRealHWInstr, HexagonInstrInfo::getRegForm, Hexagon::getRegForm, ... (45 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 4801-4992 / 第 4801-4992 行

```cpp
  4801:   default:
  4802:     return {};
  4803: 
  4804:   case Hexagon::V6_vabs_qf16_hf:
  4805:     return make(RegType::QF16);
  4806:   case Hexagon::V6_vabs_qf16_qf16:
  4807:     return make(RegType::QF16, RegType::QF16);
  4808:   case Hexagon::V6_vabs_qf32_qf32:
  4809:     return make(RegType::QF32, RegType::QF32);
  4810:   case Hexagon::V6_vabs_qf32_sf:
  4811:     return make(RegType::QF32);
  4812:   case Hexagon::V6_vadd_hf:
  4813:     return make(RegType::QF16);
  4814:   case Hexagon::V6_vadd_qf16:
  4815:     return make(RegType::QF16, RegType::QF16, RegType::QF16);
  4816:   case Hexagon::V6_vadd_qf16_mix:
  4817:     return make(RegType::QF16, RegType::QF16);
  4818:   case Hexagon::V6_vadd_qf32:
  4819:     return make(RegType::QF32, RegType::QF32, RegType::QF32);
  4820:   case Hexagon::V6_vadd_qf32_mix:
  4821:     return make(RegType::QF32, RegType::QF32);
  4822:   case Hexagon::V6_vadd_sf:
  4823:     return make(RegType::QF32);
  4824:   case Hexagon::V6_vconv_bf_qf32:
  4825:     return make(RegType::Unknown, RegType::QF32);
  4826:   case Hexagon::V6_vconv_f8_qf16:
  4827:     return make(RegType::Unknown, RegType::QF16);
  4828:   case Hexagon::V6_vconv_hf_qf16:
  4829:     return make(RegType::Unknown, RegType::QF16);
  4830:   case Hexagon::V6_vconv_hf_qf32:
  4831:     return make(RegType::Unknown, RegType::QF32);
  4832:   case Hexagon::V6_vconv_qf16_f8:
  4833:     return make(RegType::QF16);
  4834:   case Hexagon::V6_vconv_qf16_hf:
  4835:     return make(RegType::QF16);
  4836:   case Hexagon::V6_vconv_qf16_qf16:
  4837:     return make(RegType::QF16, RegType::QF16);
  4838:   case Hexagon::V6_vconv_qf32_qf32:
  4839:     return make(RegType::QF32, RegType::QF32);
  4840:   case Hexagon::V6_vconv_qf32_sf:
  4841:     return make(RegType::QF32);
  4842:   case Hexagon::V6_vconv_sf_qf32:
  4843:     return make(RegType::Unknown, RegType::QF32);
  4844:   case Hexagon::V6_vilog2_qf16:
  4845:     return make(RegType::Unknown, RegType::QF16);
  4846:   case Hexagon::V6_vilog2_qf32:
  4847:     return make(RegType::Unknown, RegType::QF32);
  4848:   case Hexagon::V6_vmpy_qf16:
  4849:     return make(RegType::QF16, RegType::QF16, RegType::QF16);
  4850:   case Hexagon::V6_vmpy_qf16_hf:
  4851:     return make(RegType::QF16);
  4852:   case Hexagon::V6_vmpy_qf16_mix_hf:
  4853:     return make(RegType::QF16, RegType::QF16);
  4854:   case Hexagon::V6_vmpy_qf32:
  4855:     return make(RegType::QF32, RegType::QF32, RegType::QF32);
  4856:   case Hexagon::V6_vmpy_qf32_hf:
  4857:     return make(RegType::QF32);
  4858:   case Hexagon::V6_vmpy_qf32_mix_hf:
  4859:     return make(RegType::QF32, RegType::QF16);
  4860:   case Hexagon::V6_vmpy_qf32_qf16:
  4861:     return make(RegType::QF32, RegType::QF16, RegType::QF16);
  4862:   case Hexagon::V6_vmpy_qf32_sf:
  4863:     return make(RegType::QF32);
  4864:   case Hexagon::V6_vmpy_rt_hf:
  4865:     return make(RegType::QF16);
  4866:   case Hexagon::V6_vmpy_rt_qf16:
  4867:     return make(RegType::QF16, RegType::QF16);
  4868:   case Hexagon::V6_vmpy_rt_sf:
  4869:     return make(RegType::QF32);
  4870:   case Hexagon::V6_vneg_qf16_hf:
  4871:     return make(RegType::QF16);
  4872:   case Hexagon::V6_vneg_qf16_qf16:
  4873:     return make(RegType::QF16, RegType::QF16);
  4874:   case Hexagon::V6_vneg_qf32_qf32:
  4875:     return make(RegType::QF32, RegType::QF32);
  4876:   case Hexagon::V6_vneg_qf32_sf:
  4877:     return make(RegType::QF32);
  4878:   case Hexagon::V6_vsub_hf:
  4879:     return make(RegType::QF16);
  4880:   case Hexagon::V6_vsub_qf16:
  4881:     return make(RegType::QF16, RegType::QF16, RegType::QF16);
  4882:   case Hexagon::V6_vsub_qf16_mix:
  4883:     return make(RegType::QF16, RegType::QF16);
  4884:   case Hexagon::V6_vsub_qf32:
  4885:     return make(RegType::QF32, RegType::QF32, RegType::QF32);
  4886:   case Hexagon::V6_vsub_qf32_mix:
  4887:     return make(RegType::QF32, RegType::QF32);
  4888:   case Hexagon::V6_vsub_sf:
  4889:     return make(RegType::QF32);
  4890:   case Hexagon::V6_vsub_sf_mix:
  4891:     return make(RegType::QF32, RegType::Unknown, RegType::QF32);
  4892:   case Hexagon::V6_vsub_hf_mix:
  4893:     return make(RegType::QF16, RegType::Unknown, RegType::QF16);
  4894:   }
  4895: }
  4896: 
  4897: } // namespace llvm::HexagonII
  4898: 
  4899: bool HexagonInstrInfo::usesQF32Operand(MachineInstr *MI, unsigned Index) const {
  4900:   auto Info = HexagonII::getRegTypeInfo(MI->getOpcode());
  4901:   switch (Index) {
  4902:   case 1:
  4903:     return Info.Input1 == HexagonII::RegType::QF32;
  4904:   case 2:
  4905:     return Info.Input2 == HexagonII::RegType::QF32;
  4906:   case 3:
  4907:     return Info.Input3 == HexagonII::RegType::QF32;
  4908:   case 0:
  4909:     return Info.Input1 == HexagonII::RegType::QF32 ||
  4910:            Info.Input2 == HexagonII::RegType::QF32 ||
  4911:            Info.Input3 == HexagonII::RegType::QF32;
  4912:   default: // No instruction with more than 3 operands uses QF32.
  4913:     return false;
  4914:   }
  4915:   return false;
  4916: }
  4917: 
  4918: bool HexagonInstrInfo::usesQF16Operand(MachineInstr *MI, unsigned Index) const {
  4919:   auto Info = HexagonII::getRegTypeInfo(MI->getOpcode());
  4920:   switch (Index) {
  4921:   case 1:
  4922:     return Info.Input1 == HexagonII::RegType::QF16;
  4923:   case 2:
  4924:     return Info.Input2 == HexagonII::RegType::QF16;
  4925:   case 3:
  4926:     return Info.Input3 == HexagonII::RegType::QF16;
  4927:   case 0:
  4928:     return Info.Input1 == HexagonII::RegType::QF16 ||
  4929:            Info.Input2 == HexagonII::RegType::QF16 ||
  4930:            Info.Input3 == HexagonII::RegType::QF16;
  4931:   default: // No instruction with more than 3 operands uses QF16.
  4932:     return false;
  4933:   }
  4934:   return false;
  4935: }
  4936: 
  4937: bool HexagonInstrInfo::usesQFOperand(MachineInstr *MI, unsigned Index) const {
  4938:   return usesQF32Operand(MI, Index) || usesQF16Operand(MI, Index);
  4939: }
  4940: 
  4941: bool HexagonInstrInfo::isQFP32Instr(MachineInstr *MI) const {
  4942:   return HexagonII::getOpRegType(MI->getOpcode()) == HexagonII::RegType::QF32;
  4943: }
  4944: 
  4945: bool HexagonInstrInfo::isQFP16Instr(MachineInstr *MI) const {
  4946:   return HexagonII::getOpRegType(MI->getOpcode()) == HexagonII::RegType::QF16;
  4947: }
  4948: 
  4949: bool HexagonInstrInfo::isQFPInstr(MachineInstr *MI) const {
  4950:   return isQFP32Instr(MI) || isQFP16Instr(MI);
  4951: }
  4952: 
  4953: // Addressing mode relations.
  4954: short HexagonInstrInfo::changeAddrMode_abs_io(short Opc) const {
  4955:   return Opc >= 0 ? Hexagon::changeAddrMode_abs_io(Opc) : Opc;
  4956: }
  4957: 
  4958: short HexagonInstrInfo::changeAddrMode_io_abs(short Opc) const {
  4959:   return Opc >= 0 ? Hexagon::changeAddrMode_io_abs(Opc) : Opc;
  4960: }
  4961: 
  4962: short HexagonInstrInfo::changeAddrMode_io_pi(short Opc) const {
  4963:   return Opc >= 0 ? Hexagon::changeAddrMode_io_pi(Opc) : Opc;
  4964: }
  4965: 
  4966: short HexagonInstrInfo::changeAddrMode_io_rr(short Opc) const {
  4967:   return Opc >= 0 ? Hexagon::changeAddrMode_io_rr(Opc) : Opc;
  4968: }
  4969: 
  4970: short HexagonInstrInfo::changeAddrMode_pi_io(short Opc) const {
  4971:   return Opc >= 0 ? Hexagon::changeAddrMode_pi_io(Opc) : Opc;
  4972: }
  4973: 
  4974: short HexagonInstrInfo::changeAddrMode_rr_io(short Opc) const {
  4975:   return Opc >= 0 ? Hexagon::changeAddrMode_rr_io(Opc) : Opc;
  4976: }
  4977: 
  4978: short HexagonInstrInfo::changeAddrMode_rr_ur(short Opc) const {
  4979:   return Opc >= 0 ? Hexagon::changeAddrMode_rr_ur(Opc) : Opc;
  4980: }
  4981: 
  4982: short HexagonInstrInfo::changeAddrMode_ur_rr(short Opc) const {
  4983:   return Opc >= 0 ? Hexagon::changeAddrMode_ur_rr(Opc) : Opc;
  4984: }
  4985: 
  4986: MCInst HexagonInstrInfo::getNop() const {
  4987:   static const MCInst Nop = MCInstBuilder(Hexagon::A2_nop);
  4988: 
  4989:   return MCInstBuilder(Hexagon::BUNDLE)
  4990:     .addImm(0)
  4991:     .addInst(&Nop);
  4992: }
```
- EN: It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as make, HexagonInstrInfo::usesQF32Operand, HexagonII::getRegTypeInfo, HexagonInstrInfo::usesQF16Operand, ... (20 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 make, HexagonInstrInfo::usesQF32Operand, HexagonII::getRegTypeInfo, HexagonInstrInfo::usesQF16Operand, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII, HexagonInstrInfo，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- VLIW packetization / VLIW 成包
- instruction scheduling models / 指令调度模型
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- assembly/MC integration / 汇编/MC 集成
- MC-layer target description / MC 层目标描述

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonInstrInfo.h, HexagonFrameLowering.h, HexagonHazardRecognizer.h, HexagonRegisterInfo.h, HexagonSubtarget.h, llvm/ADT/ArrayRef.h, llvm/ADT/SmallPtrSet.h, llvm/ADT/SmallVector.h, llvm/ADT/StringExtras.h, llvm/ADT/StringRef.h, ... (53 total)`
- Hexagon symbols / Hexagon 符号: `HexagonInstrInfo, HexagonFrameLowering, HexagonHazardRecognizer, HexagonRegisterInfo, HexagonSubtarget, HexagonDepTimingClasses, HexagonGenDFAPacketizer, HexagonGenInstrInfo, Hexagon_MEMW_OFFSET_MAX, Hexagon_MEMW_OFFSET_MIN, ... (21 total)`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
