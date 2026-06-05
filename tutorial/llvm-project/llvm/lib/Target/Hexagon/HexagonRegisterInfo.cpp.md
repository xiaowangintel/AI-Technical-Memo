# HexagonRegisterInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonRegisterInfo.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file contains the Hexagon implementation of the TargetRegisterInfo
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。 重点涉及寄存器模型。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===-- HexagonRegisterInfo.cpp - Hexagon Register Information ------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This file contains the Hexagon implementation of the TargetRegisterInfo
    10: // class.
    11: //
    12: //===----------------------------------------------------------------------===//
    13: 
    14: #include "HexagonRegisterInfo.h"
    15: #include "HexagonMachineFunctionInfo.h"
    16: #include "HexagonSubtarget.h"
    17: #include "llvm/ADT/BitVector.h"
    18: #include "llvm/ADT/STLExtras.h"
    19: #include "llvm/ADT/SmallSet.h"
    20: #include "llvm/CodeGen/LiveIntervals.h"
    21: #include "llvm/CodeGen/LiveRegUnits.h"
    22: #include "llvm/CodeGen/MachineFrameInfo.h"
    23: #include "llvm/CodeGen/MachineFunction.h"
    24: #include "llvm/CodeGen/MachineInstrBuilder.h"
    25: #include "llvm/CodeGen/MachineRegisterInfo.h"
    26: #include "llvm/CodeGen/PseudoSourceValue.h"
    27: #include "llvm/CodeGen/RegisterScavenging.h"
    28: #include "llvm/CodeGen/TargetInstrInfo.h"
    29: #include "llvm/IR/Function.h"
    30: #include "llvm/IR/Type.h"
    31: #include "llvm/Support/CommandLine.h"
    32: #include "llvm/Support/Debug.h"
    33: #include "llvm/Support/ErrorHandling.h"
    34: #include "llvm/Support/raw_ostream.h"
    35: #include "llvm/Target/TargetOptions.h"
    36: 
    37: #define GET_REGINFO_TARGET_DESC
    38: #include "HexagonGenRegisterInfo.inc"
    39: 
    40: using namespace llvm;
    41: 
    42: static cl::opt<unsigned> FrameIndexSearchRange(
    43:     "hexagon-frame-index-search-range", cl::init(32), cl::Hidden,
    44:     cl::desc("Limit on instruction search range in frame index elimination"));
    45: 
    46: static cl::opt<unsigned> FrameIndexReuseLimit(
    47:     "hexagon-frame-index-reuse-limit", cl::init(~0), cl::Hidden,
    48:     cl::desc("Limit on the number of reused registers in frame index "
    49:     "elimination"));
    50: 
```
- EN: It imports headers such as HexagonRegisterInfo.h, HexagonMachineFunctionInfo.h, HexagonSubtarget.h, llvm/ADT/BitVector.h, ... (23 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as FrameIndexSearchRange, FrameIndexReuseLimit, translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 HexagonRegisterInfo.h, HexagonMachineFunctionInfo.h, HexagonSubtarget.h, llvm/ADT/BitVector.h, ... (23 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 FrameIndexSearchRange, FrameIndexReuseLimit 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 51-100 / 第 51-100 行

```cpp
    51: HexagonRegisterInfo::HexagonRegisterInfo(unsigned HwMode)
    52:     : HexagonGenRegisterInfo(Hexagon::R31, 0/*DwarfFlavor*/, 0/*EHFlavor*/,
    53:                              0/*PC*/, HwMode) {}
    54: 
    55: bool HexagonRegisterInfo::isGlobalReg(MCPhysReg Reg) const {
    56:   switch (Reg) {
    57:   case Hexagon::R29:
    58:   case Hexagon::R30:
    59:   case Hexagon::R31:
    60:     return true;
    61:   }
    62:   return false;
    63: }
    64: 
    65: bool HexagonRegisterInfo::isFakeReg(MCPhysReg Reg) const {
    66:   // VF0-VF31 are fake registers used as sub-registers in HVX vector pairs
    67:   if (Reg >= Hexagon::VF0 && Reg <= Hexagon::VF31)
    68:     return true;
    69:   // VFR0-VFR31 are fake registers used for reversed vector pairs
    70:   if (Reg >= Hexagon::VFR0 && Reg <= Hexagon::VFR31)
    71:     return true;
    72:   return false;
    73: }
    74: 
    75: bool HexagonRegisterInfo::isEHReturnCalleeSaveReg(Register R) const {
    76:   return R == Hexagon::R0 || R == Hexagon::R1 || R == Hexagon::R2 ||
    77:          R == Hexagon::R3 || R == Hexagon::D0 || R == Hexagon::D1;
    78: }
    79: 
    80: const MCPhysReg *
    81: HexagonRegisterInfo::getCallerSavedRegs(const MachineFunction *MF,
    82:       const TargetRegisterClass *RC) const {
    83:   using namespace Hexagon;
    84: 
    85:   static const MCPhysReg Int32[] = {
    86:     R0, R1, R2, R3, R4, R5, R6, R7, R8, R9, R10, R11, R12, R13, R14, R15, 0
    87:   };
    88:   static const MCPhysReg Int64[] = {
    89:     D0, D1, D2, D3, D4, D5, D6, D7, 0
    90:   };
    91:   static const MCPhysReg Pred[] = {
    92:     P0, P1, P2, P3, 0
    93:   };
    94:   static const MCPhysReg VecSgl[] = {
    95:      V0,  V1,  V2,  V3,  V4,  V5,  V6,  V7,  V8,  V9, V10, V11, V12, V13,
    96:     V14, V15, V16, V17, V18, V19, V20, V21, V22, V23, V24, V25, V26, V27,
    97:     V28, V29, V30, V31,   0
    98:   };
    99:   static const MCPhysReg VecDbl[] = {
   100:     W0, W1, W2, W3, W4, W5, W6, W7, W8, W9, W10, W11, W12, W13, W14, W15, 0
```
- EN: It opens namespaces (Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as HexagonRegisterInfo::HexagonRegisterInfo, HexagonRegisterInfo::isGlobalReg, HexagonRegisterInfo::isFakeReg, HexagonRegisterInfo::isEHReturnCalleeSaveReg, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonRegisterInfo, HexagonGenRegisterInfo, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 HexagonRegisterInfo::HexagonRegisterInfo, HexagonRegisterInfo::isGlobalReg, HexagonRegisterInfo::isFakeReg, HexagonRegisterInfo::isEHReturnCalleeSaveReg, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonRegisterInfo, HexagonGenRegisterInfo，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```cpp
   101:   };
   102:   static const MCPhysReg VecPred[] = {
   103:     Q0, Q1, Q2, Q3, 0
   104:   };
   105: 
   106:   switch (RC->getID()) {
   107:     case IntRegsRegClassID:
   108:       return Int32;
   109:     case DoubleRegsRegClassID:
   110:       return Int64;
   111:     case PredRegsRegClassID:
   112:       return Pred;
   113:     case HvxVRRegClassID:
   114:       return VecSgl;
   115:     case HvxWRRegClassID:
   116:       return VecDbl;
   117:     case HvxQRRegClassID:
   118:       return VecPred;
   119:     default:
   120:       break;
   121:   }
   122: 
   123:   static const MCPhysReg Empty[] = { 0 };
   124: #ifndef NDEBUG
   125:   dbgs() << "Register class: " << getRegClassName(RC) << "\n";
   126: #endif
   127:   llvm_unreachable("Unexpected register class");
   128:   return Empty;
   129: }
   130: 
   131: 
   132: const MCPhysReg *
   133: HexagonRegisterInfo::getCalleeSavedRegs(const MachineFunction *MF) const {
   134:   static const MCPhysReg CalleeSavedRegsV3[] = {
   135:     Hexagon::R16,   Hexagon::R17,   Hexagon::R18,   Hexagon::R19,
   136:     Hexagon::R20,   Hexagon::R21,   Hexagon::R22,   Hexagon::R23,
   137:     Hexagon::R24,   Hexagon::R25,   Hexagon::R26,   Hexagon::R27, 0
   138:   };
   139: 
   140:   // Functions that contain a call to __builtin_eh_return also save the first 4
   141:   // parameter registers.
   142:   static const MCPhysReg CalleeSavedRegsV3EHReturn[] = {
   143:     Hexagon::R0,    Hexagon::R1,    Hexagon::R2,    Hexagon::R3,
   144:     Hexagon::R16,   Hexagon::R17,   Hexagon::R18,   Hexagon::R19,
   145:     Hexagon::R20,   Hexagon::R21,   Hexagon::R22,   Hexagon::R23,
   146:     Hexagon::R24,   Hexagon::R25,   Hexagon::R26,   Hexagon::R27, 0
   147:   };
   148: 
   149:   bool HasEHReturn = MF->getInfo<HexagonMachineFunctionInfo>()->hasEHReturn();
   150: 
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as llvm_unreachable, HexagonRegisterInfo::getCalleeSavedRegs, getInfo<HexagonMachineFunctionInfo>, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonRegisterInfo, HexagonMachineFunctionInfo, showing how the code connects to sibling backend components.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 llvm_unreachable, HexagonRegisterInfo::getCalleeSavedRegs, getInfo<HexagonMachineFunctionInfo> 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonRegisterInfo, HexagonMachineFunctionInfo，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151:   return HasEHReturn ? CalleeSavedRegsV3EHReturn : CalleeSavedRegsV3;
   152: }
   153: 
   154: 
   155: const uint32_t *HexagonRegisterInfo::getCallPreservedMask(
   156:       const MachineFunction &MF, CallingConv::ID) const {
   157:   return HexagonCSR_RegMask;
   158: }
   159: 
   160: 
   161: BitVector HexagonRegisterInfo::getReservedRegs(const MachineFunction &MF)
   162:       const {
   163:   BitVector Reserved(getNumRegs());
   164:   Reserved.set(Hexagon::R29);
   165:   Reserved.set(Hexagon::R30);
   166:   Reserved.set(Hexagon::R31);
   167:   Reserved.set(Hexagon::VTMP);
   168: 
   169:   // Guest registers.
   170:   Reserved.set(Hexagon::GELR);        // G0
   171:   Reserved.set(Hexagon::GSR);         // G1
   172:   Reserved.set(Hexagon::GOSP);        // G2
   173:   Reserved.set(Hexagon::G3);          // G3
   174: 
   175:   // Control registers.
   176:   Reserved.set(Hexagon::SA0);         // C0
   177:   Reserved.set(Hexagon::LC0);         // C1
   178:   Reserved.set(Hexagon::SA1);         // C2
   179:   Reserved.set(Hexagon::LC1);         // C3
   180:   Reserved.set(Hexagon::P3_0);        // C4
   181:   Reserved.set(Hexagon::USR);         // C8
   182:   Reserved.set(Hexagon::PC);          // C9
   183:   Reserved.set(Hexagon::UGP);         // C10
   184:   Reserved.set(Hexagon::GP);          // C11
   185:   Reserved.set(Hexagon::CS0);         // C12
   186:   Reserved.set(Hexagon::CS1);         // C13
   187:   Reserved.set(Hexagon::UPCYCLELO);   // C14
   188:   Reserved.set(Hexagon::UPCYCLEHI);   // C15
   189:   Reserved.set(Hexagon::FRAMELIMIT);  // C16
   190:   Reserved.set(Hexagon::FRAMEKEY);    // C17
   191:   Reserved.set(Hexagon::PKTCOUNTLO);  // C18
   192:   Reserved.set(Hexagon::PKTCOUNTHI);  // C19
   193:   Reserved.set(Hexagon::UTIMERLO);    // C30
   194:   Reserved.set(Hexagon::UTIMERHI);    // C31
   195:   // Out of the control registers, only C8 is explicitly defined in
   196:   // HexagonRegisterInfo.td. If others are defined, make sure to add
   197:   // them here as well.
   198:   Reserved.set(Hexagon::C8);
   199:   Reserved.set(Hexagon::USR_OVF);
   200: 
```
- EN: It declares or implements routines such as HexagonRegisterInfo::getCallPreservedMask, HexagonRegisterInfo::getReservedRegs, Reserved, set, translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonRegisterInfo, HexagonCSR_RegMask, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonRegisterInfo::getCallPreservedMask, HexagonRegisterInfo::getReservedRegs, Reserved, set 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonRegisterInfo, HexagonCSR_RegMask，说明了它与同级后端组件的连接关系。

### Lines 201-250 / 第 201-250 行

```cpp
   201:   // Leveraging these registers will require more work to recognize
   202:   // the new semantics posed, Hi/LoVec patterns, etc.
   203:   // Note well: if enabled, they should be restricted to only
   204:   // where `HST.useHVXOps() && HST.hasV67Ops()` is true.
   205:   for (auto Reg : Hexagon_MC::GetVectRegRev())
   206:     Reserved.set(Reg);
   207: 
   208:   if (MF.getSubtarget<HexagonSubtarget>().hasReservedR19())
   209:     Reserved.set(Hexagon::R19);
   210: 
   211:   Register AP =
   212:       MF.getInfo<HexagonMachineFunctionInfo>()->getStackAlignBaseReg();
   213:   if (AP.isValid())
   214:     Reserved.set(AP);
   215: 
   216:   for (int x = Reserved.find_first(); x >= 0; x = Reserved.find_next(x))
   217:     markSuperRegs(Reserved, x);
   218: 
   219:   return Reserved;
   220: }
   221: 
   222: bool HexagonRegisterInfo::eliminateFrameIndex(MachineBasicBlock::iterator II,
   223:                                               int SPAdj, unsigned FIOp,
   224:                                               RegScavenger *RS) const {
   225:   static unsigned ReuseCount = 0;
   226:   //
   227:   // Hexagon_TODO: Do we need to enforce this for Hexagon?
   228:   assert(SPAdj == 0 && "Unexpected");
   229: 
   230:   MachineInstr &MI = *II;
   231:   MachineBasicBlock &MB = *MI.getParent();
   232:   MachineFunction &MF = *MB.getParent();
   233:   auto &HST = MF.getSubtarget<HexagonSubtarget>();
   234:   auto &HII = *HST.getInstrInfo();
   235:   auto &HFI = *HST.getFrameLowering();
   236: 
   237:   Register BP;
   238:   int FI = MI.getOperand(FIOp).getIndex();
   239:   // Select the base pointer (BP) and calculate the actual offset from BP
   240:   // to the beginning of the object at index FI.
   241:   int Offset = HFI.getFrameIndexReference(MF, FI, BP).getFixed();
   242:   // Add the offset from the instruction.
   243:   int RealOffset = Offset + MI.getOperand(FIOp+1).getImm();
   244: 
   245:   unsigned Opc = MI.getOpcode();
   246:   switch (Opc) {
   247:     case Hexagon::PS_fia:
   248:       MI.setDesc(HII.get(Hexagon::A2_addi));
   249:       MI.getOperand(FIOp).ChangeToImmediate(RealOffset);
   250:       MI.removeOperand(FIOp+1);
```
- EN: It declares or implements routines such as useHVXOps, getInfo<HexagonMachineFunctionInfo>, find_next, HexagonRegisterInfo::eliminateFrameIndex, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include Hexagon_MC, HexagonSubtarget, HexagonMachineFunctionInfo, HexagonRegisterInfo, ... (5 total), showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 useHVXOps, getInfo<HexagonMachineFunctionInfo>, find_next, HexagonRegisterInfo::eliminateFrameIndex, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 Hexagon_MC, HexagonSubtarget, HexagonMachineFunctionInfo, HexagonRegisterInfo, ... (5 total)，说明了它与同级后端组件的连接关系。

### Lines 251-300 / 第 251-300 行

```cpp
   251:       return false;
   252:     case Hexagon::PS_fi:
   253:       // Set up the instruction for updating below.
   254:       MI.setDesc(HII.get(Hexagon::A2_addi));
   255:       break;
   256:   }
   257: 
   258:   if (!HII.isValidOffset(Opc, RealOffset, this)) {
   259:     // If the offset is not valid, calculate the address in a temporary
   260:     // register and use it with offset 0.
   261:     int InstOffset = 0;
   262:     // The actual base register (BP) is typically shared between many
   263:     // instructions where frame indices are being replaced. In scalar
   264:     // instructions the offset range is large, and the need for an extra
   265:     // add instruction is infrequent. Vector loads/stores, however, have
   266:     // a much smaller offset range: [-8, 7), or #s4. In those cases it
   267:     // makes sense to "standardize" the immediate in the "addi" instruction
   268:     // so that multiple loads/stores could be based on it.
   269:     bool IsPair = false;
   270:     switch (MI.getOpcode()) {
   271:       // All of these instructions have the same format: base+#s4.
   272:       case Hexagon::PS_vloadrw_ai:
   273:       case Hexagon::PS_vloadrw_nt_ai:
   274:       case Hexagon::PS_vstorerw_ai:
   275:       case Hexagon::PS_vstorerw_nt_ai:
   276:         IsPair = true;
   277:         [[fallthrough]];
   278:       case Hexagon::PS_vloadrv_ai:
   279:       case Hexagon::PS_vloadrv_nt_ai:
   280:       case Hexagon::PS_vstorerv_ai:
   281:       case Hexagon::PS_vstorerv_nt_ai:
   282:       case Hexagon::V6_vL32b_ai:
   283:       case Hexagon::V6_vS32b_ai: {
   284:         unsigned HwLen = HST.getVectorLength();
   285:         if (RealOffset % HwLen == 0) {
   286:           int VecOffset = RealOffset / HwLen;
   287:           // Rewrite the offset as "base + [-8, 7)".
   288:           VecOffset += 8;
   289:           // Pairs are expanded into two instructions: make sure that both
   290:           // can use the same base (i.e. VecOffset+1 is not a different
   291:           // multiple of 16 than VecOffset).
   292:           if (!IsPair || (VecOffset + 1) % 16 != 0) {
   293:             RealOffset = (VecOffset & -16) * HwLen;
   294:             InstOffset = (VecOffset % 16 - 8) * HwLen;
   295:           }
   296:         }
   297:       }
   298:     }
   299: 
   300:     // Search backwards in the block for "Reg = A2_addi BP, RealOffset".
```
- EN: It declares or implements routines such as setDesc, getVectorLength, base, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 setDesc, getVectorLength, base 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 301-350 / 第 301-350 行

```cpp
   301:     // This will give us a chance to avoid creating a new register.
   302:     Register ReuseBP;
   303: 
   304:     if (ReuseCount < FrameIndexReuseLimit) {
   305:       unsigned SearchCount = 0, SearchRange = FrameIndexSearchRange;
   306:       SmallSet<Register,2> SeenVRegs;
   307:       bool PassedCall = false;
   308:       LiveRegUnits Defs(*this), Uses(*this);
   309: 
   310:       for (auto I = std::next(II.getReverse()), E = MB.rend(); I != E; ++I) {
   311:         if (SearchCount == SearchRange)
   312:           break;
   313:         ++SearchCount;
   314:         const MachineInstr &BI = *I;
   315:         LiveRegUnits::accumulateUsedDefed(BI, Defs, Uses, this);
   316:         PassedCall |= BI.isCall();
   317:         for (const MachineOperand &Op : BI.operands()) {
   318:           if (SeenVRegs.size() > 1)
   319:             break;
   320:           if (Op.isReg() && Op.getReg().isVirtual())
   321:             SeenVRegs.insert(Op.getReg());
   322:         }
   323:         if (BI.getOpcode() != Hexagon::A2_addi)
   324:           continue;
   325:         if (BI.getOperand(1).getReg() != BP)
   326:           continue;
   327:         const auto &Op2 = BI.getOperand(2);
   328:         if (!Op2.isImm() || Op2.getImm() != RealOffset)
   329:           continue;
   330: 
   331:         Register R = BI.getOperand(0).getReg();
   332:         if (R.isPhysical()) {
   333:           if (Defs.available(R))
   334:             ReuseBP = R;
   335:         } else if (R.isVirtual()) {
   336:           // Extending a range of a virtual register can be dangerous,
   337:           // since the scavenger will need to find a physical register
   338:           // for it. Avoid extending the range past a function call,
   339:           // and avoid overlapping it with another virtual register.
   340:           if (!PassedCall && SeenVRegs.size() <= 1)
   341:             ReuseBP = R;
   342:         }
   343:         break;
   344:       }
   345:       if (ReuseBP)
   346:         ++ReuseCount;
   347:     }
   348: 
   349:     auto &MRI = MF.getRegInfo();
   350:     if (!ReuseBP) {
```
- EN: It declares or implements routines such as Defs, LiveRegUnits::accumulateUsedDefed, isCall, getOperand, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 Defs, LiveRegUnits::accumulateUsedDefed, isCall, getOperand, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 351-400 / 第 351-400 行

```cpp
   351:       ReuseBP = MRI.createVirtualRegister(&Hexagon::IntRegsRegClass);
   352:       const DebugLoc &DL = MI.getDebugLoc();
   353:       BuildMI(MB, II, DL, HII.get(Hexagon::A2_addi), ReuseBP)
   354:         .addReg(BP)
   355:         .addImm(RealOffset);
   356:     }
   357:     BP = ReuseBP;
   358:     RealOffset = InstOffset;
   359:   }
   360: 
   361:   MI.getOperand(FIOp).ChangeToRegister(BP, false, false, false);
   362:   MI.getOperand(FIOp+1).ChangeToImmediate(RealOffset);
   363:   return false;
   364: }
   365: 
   366: 
   367: bool HexagonRegisterInfo::shouldCoalesce(MachineInstr *MI,
   368:       const TargetRegisterClass *SrcRC, unsigned SubReg,
   369:       const TargetRegisterClass *DstRC, unsigned DstSubReg,
   370:       const TargetRegisterClass *NewRC, LiveIntervals &LIS) const {
   371:   // Coalescing will extend the live interval of the destination register.
   372:   // If the destination register is a vector pair, avoid introducing function
   373:   // calls into the interval, since it could result in a spilling of a pair
   374:   // instead of a single vector.
   375:   MachineFunction &MF = *MI->getParent()->getParent();
   376:   const HexagonSubtarget &HST = MF.getSubtarget<HexagonSubtarget>();
   377:   if (!HST.useHVXOps() || NewRC->getID() != Hexagon::HvxWRRegClass.getID())
   378:     return true;
   379:   bool SmallSrc = SrcRC->getID() == Hexagon::HvxVRRegClass.getID();
   380:   bool SmallDst = DstRC->getID() == Hexagon::HvxVRRegClass.getID();
   381:   if (!SmallSrc && !SmallDst)
   382:     return true;
   383: 
   384:   Register DstReg = MI->getOperand(0).getReg();
   385:   Register SrcReg = MI->getOperand(1).getReg();
   386:   const SlotIndexes &Indexes = *LIS.getSlotIndexes();
   387:   auto HasCall = [&Indexes] (const LiveInterval::Segment &S) {
   388:     for (SlotIndex I = S.start.getBaseIndex(), E = S.end.getBaseIndex();
   389:          I != E; I = I.getNextIndex()) {
   390:       if (const MachineInstr *MI = Indexes.getInstructionFromIndex(I))
   391:         if (MI->isCall())
   392:           return true;
   393:     }
   394:     return false;
   395:   };
   396: 
   397:   if (SmallSrc == SmallDst) {
   398:     // Both must be true, because the case for both being false was
   399:     // checked earlier. Both registers will be coalesced into a register
   400:     // of a wider class (HvxWR), and we don't want its live range to
```
- EN: It declares or implements routines such as createVirtualRegister, getDebugLoc, BuildMI, getOperand, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonRegisterInfo, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 createVirtualRegister, getDebugLoc, BuildMI, getOperand, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonRegisterInfo, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 401-450 / 第 401-450 行

```cpp
   401:     // span over calls.
   402:     return !any_of(LIS.getInterval(DstReg), HasCall) &&
   403:            !any_of(LIS.getInterval(SrcReg), HasCall);
   404:   }
   405: 
   406:   // If one register is large (HvxWR) and the other is small (HvxVR), then
   407:   // coalescing is ok if the large is already live across a function call,
   408:   // or if the small one is not.
   409:   Register SmallReg = SmallSrc ? SrcReg : DstReg;
   410:   Register LargeReg = SmallSrc ? DstReg : SrcReg;
   411:   return  any_of(LIS.getInterval(LargeReg), HasCall) ||
   412:          !any_of(LIS.getInterval(SmallReg), HasCall);
   413: }
   414: 
   415: 
   416: Register HexagonRegisterInfo::getFrameRegister(const MachineFunction
   417:                                                &MF) const {
   418:   const HexagonFrameLowering *TFI = getFrameLowering(MF);
   419:   if (TFI->hasFP(MF))
   420:     return getFrameRegister();
   421:   return getStackRegister();
   422: }
   423: 
   424: 
   425: Register HexagonRegisterInfo::getFrameRegister() const {
   426:   return Hexagon::R30;
   427: }
   428: 
   429: 
   430: Register HexagonRegisterInfo::getStackRegister() const {
   431:   return Hexagon::R29;
   432: }
   433: 
   434: 
   435: unsigned HexagonRegisterInfo::getHexagonSubRegIndex(
   436:       const TargetRegisterClass &RC, unsigned GenIdx) const {
   437:   assert(GenIdx == Hexagon::ps_sub_lo || GenIdx == Hexagon::ps_sub_hi);
   438: 
   439:   static const unsigned ISub[] = { Hexagon::isub_lo, Hexagon::isub_hi };
   440:   static const unsigned VSub[] = { Hexagon::vsub_lo, Hexagon::vsub_hi };
   441:   static const unsigned WSub[] = { Hexagon::wsub_lo, Hexagon::wsub_hi };
   442: 
   443:   switch (RC.getID()) {
   444:     case Hexagon::CtrRegs64RegClassID:
   445:     case Hexagon::DoubleRegsRegClassID:
   446:       return ISub[GenIdx];
   447:     case Hexagon::HvxWRRegClassID:
   448:       return VSub[GenIdx];
   449:     case Hexagon::HvxVQRRegClassID:
   450:       return WSub[GenIdx];
```
- EN: It declares or implements routines such as any_of, HexagonRegisterInfo::getFrameRegister, getFrameLowering, getStackRegister, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonRegisterInfo, HexagonFrameLowering, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 any_of, HexagonRegisterInfo::getFrameRegister, getFrameLowering, getStackRegister, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonRegisterInfo, HexagonFrameLowering，说明了它与同级后端组件的连接关系。

### Lines 451-468 / 第 451-468 行

```cpp
   451:   }
   452: 
   453:   if (!RC.superclasses().empty())
   454:     return getHexagonSubRegIndex(*getRegClass(*RC.superclasses().begin()),
   455:                                  GenIdx);
   456: 
   457:   llvm_unreachable("Invalid register class");
   458: }
   459: 
   460: bool HexagonRegisterInfo::useFPForScavengingIndex(const MachineFunction &MF)
   461:       const {
   462:   return MF.getSubtarget<HexagonSubtarget>().getFrameLowering()->hasFP(MF);
   463: }
   464: 
   465: const TargetRegisterClass *
   466: HexagonRegisterInfo::getPointerRegClass(unsigned Kind) const {
   467:   return &Hexagon::IntRegsRegClass;
   468: }
```
- EN: It declares or implements routines such as llvm_unreachable, HexagonRegisterInfo::useFPForScavengingIndex, getSubtarget<HexagonSubtarget>, HexagonRegisterInfo::getPointerRegClass, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonRegisterInfo, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 llvm_unreachable, HexagonRegisterInfo::useFPForScavengingIndex, getSubtarget<HexagonSubtarget>, HexagonRegisterInfo::getPointerRegClass 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonRegisterInfo, HexagonSubtarget，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制
- ABI and stack-frame lowering / ABI 与栈帧下沉

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonRegisterInfo.h, HexagonMachineFunctionInfo.h, HexagonSubtarget.h, llvm/ADT/BitVector.h, llvm/ADT/STLExtras.h, llvm/ADT/SmallSet.h, llvm/CodeGen/LiveIntervals.h, llvm/CodeGen/LiveRegUnits.h, llvm/CodeGen/MachineFrameInfo.h, llvm/CodeGen/MachineFunction.h, ... (23 total)`
- Hexagon symbols / Hexagon 符号: `HexagonRegisterInfo, HexagonMachineFunctionInfo, HexagonSubtarget, HexagonGenRegisterInfo, HexagonCSR_RegMask, Hexagon_MC, Hexagon_TODO, HexagonFrameLowering`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
