# SystemZRegisterInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZRegisterInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file defines or implements target register information for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责定义或实现目标寄存器信息。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```cpp
   1: //===-- SystemZRegisterInfo.cpp - SystemZ register information ------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "SystemZRegisterInfo.h"
  10: #include "SystemZInstrInfo.h"
  11: #include "SystemZSubtarget.h"
  12: #include "llvm/ADT/SmallSet.h"
  13: #include "llvm/CodeGen/LiveIntervals.h"
  14: #include "llvm/CodeGen/MachineInstrBuilder.h"
  15: #include "llvm/CodeGen/MachineRegisterInfo.h"
  16: #include "llvm/CodeGen/TargetFrameLowering.h"
  17: #include "llvm/CodeGen/VirtRegMap.h"
  18: #include "llvm/IR/DebugInfoMetadata.h"
  19: 
  20: using namespace llvm;
  21: 
  22: #define GET_REGINFO_TARGET_DESC
  23: #include "SystemZGenRegisterInfo.inc"
  24: 
  25: // Given that MO is a GRX32 operand, return either GR32 or GRH32 if MO
  26: // somehow belongs in it. Otherwise, return GRX32.
  27: static const TargetRegisterClass *getRC32(MachineOperand &MO,
  28:                                           const VirtRegMap *VRM,
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZRegisterInfo.h`, `SystemZInstrInfo.h`, `SystemZSubtarget.h`, `SmallSet.h`, `LiveIntervals.h`, `MachineInstrBuilder.h` that expose the LLVM and target interfaces used in later logic. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZRegisterInfo.h`, `SystemZInstrInfo.h`, `SystemZSubtarget.h`, `SmallSet.h`, `LiveIntervals.h`, `MachineInstrBuilder.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 29-56 / 第 29-56 行
```cpp
  29:                                           const MachineRegisterInfo *MRI) {
  30:   const TargetRegisterClass *RC = MRI->getRegClass(MO.getReg());
  31: 
  32:   if (SystemZ::GR32BitRegClass.hasSubClassEq(RC) ||
  33:       MO.getSubReg() == SystemZ::subreg_ll32 ||
  34:       MO.getSubReg() == SystemZ::subreg_l32)
  35:     return &SystemZ::GR32BitRegClass;
  36:   if (SystemZ::GRH32BitRegClass.hasSubClassEq(RC) ||
  37:       MO.getSubReg() == SystemZ::subreg_lh32 ||
  38:       MO.getSubReg() == SystemZ::subreg_h32)
  39:     return &SystemZ::GRH32BitRegClass;
  40: 
  41:   if (VRM && VRM->hasPhys(MO.getReg())) {
  42:     Register PhysReg = VRM->getPhys(MO.getReg());
  43:     if (SystemZ::GR32BitRegClass.contains(PhysReg))
  44:       return &SystemZ::GR32BitRegClass;
  45:     assert (SystemZ::GRH32BitRegClass.contains(PhysReg) &&
  46:             "Phys reg not in GR32 or GRH32?");
  47:     return &SystemZ::GRH32BitRegClass;
  48:   }
  49: 
  50:   assert (RC == &SystemZ::GRX32BitRegClass);
  51:   return RC;
  52: }
  53: 
  54: // Pass the registers of RC as hints while making sure that if any of these
  55: // registers are copy hints (and therefore already in Hints), hint them
  56: // first.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 57-84 / 第 57-84 行
```cpp
  57: static void addHints(ArrayRef<MCPhysReg> Order,
  58:                      SmallVectorImpl<MCPhysReg> &Hints,
  59:                      const TargetRegisterClass *RC,
  60:                      const MachineRegisterInfo *MRI) {
  61:   SmallSet<unsigned, 4> CopyHints(llvm::from_range, Hints);
  62:   Hints.clear();
  63:   for (MCPhysReg Reg : Order)
  64:     if (CopyHints.count(Reg) &&
  65:         RC->contains(Reg) && !MRI->isReserved(Reg))
  66:       Hints.push_back(Reg);
  67:   for (MCPhysReg Reg : Order)
  68:     if (!CopyHints.count(Reg) &&
  69:         RC->contains(Reg) && !MRI->isReserved(Reg))
  70:       Hints.push_back(Reg);
  71: }
  72: 
  73: bool SystemZRegisterInfo::getRegAllocationHints(
  74:     Register VirtReg, ArrayRef<MCPhysReg> Order,
  75:     SmallVectorImpl<MCPhysReg> &Hints, const MachineFunction &MF,
  76:     const VirtRegMap *VRM, const LiveRegMatrix *Matrix) const {
  77:   const MachineRegisterInfo *MRI = &MF.getRegInfo();
  78:   const SystemZSubtarget &Subtarget = MF.getSubtarget<SystemZSubtarget>();
  79:   const TargetRegisterInfo *TRI = Subtarget.getRegisterInfo();
  80: 
  81:   bool BaseImplRetVal = TargetRegisterInfo::getRegAllocationHints(
  82:       VirtReg, Order, Hints, MF, VRM, Matrix);
  83: 
  84:   if (VRM != nullptr) {
```
- **EN**: The range implements or declares functions including `addHints`, `SystemZRegisterInfo::getRegAllocationHints`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `addHints`, `SystemZRegisterInfo::getRegAllocationHints` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 85-112 / 第 85-112 行
```cpp
  85:     // Add any two address hints after any copy hints.
  86:     SmallSet<unsigned, 4> TwoAddrHints;
  87:     for (auto &Use : MRI->reg_nodbg_instructions(VirtReg))
  88:       if (SystemZ::getTwoOperandOpcode(Use.getOpcode()) != -1) {
  89:         const MachineOperand *VRRegMO = nullptr;
  90:         const MachineOperand *OtherMO = nullptr;
  91:         const MachineOperand *CommuMO = nullptr;
  92:         if (VirtReg == Use.getOperand(0).getReg()) {
  93:           VRRegMO = &Use.getOperand(0);
  94:           OtherMO = &Use.getOperand(1);
  95:           if (Use.isCommutable())
  96:             CommuMO = &Use.getOperand(2);
  97:         } else if (VirtReg == Use.getOperand(1).getReg()) {
  98:           VRRegMO = &Use.getOperand(1);
  99:           OtherMO = &Use.getOperand(0);
 100:         } else if (VirtReg == Use.getOperand(2).getReg() &&
 101:                    Use.isCommutable()) {
 102:           VRRegMO = &Use.getOperand(2);
 103:           OtherMO = &Use.getOperand(0);
 104:         } else
 105:           continue;
 106: 
 107:         auto tryAddHint = [&](const MachineOperand *MO) -> void {
 108:           Register Reg = MO->getReg();
 109:           Register PhysReg =
 110:               Reg.isPhysical() ? Reg : Register(VRM->getPhys(Reg));
 111:           if (PhysReg) {
 112:             if (MO->getSubReg())
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 113-140 / 第 113-140 行
```cpp
 113:               PhysReg = getSubReg(PhysReg, MO->getSubReg());
 114:             if (VRRegMO->getSubReg())
 115:               PhysReg = getMatchingSuperReg(PhysReg, VRRegMO->getSubReg(),
 116:                                             MRI->getRegClass(VirtReg));
 117:             if (!MRI->isReserved(PhysReg) && !is_contained(Hints, PhysReg))
 118:               TwoAddrHints.insert(PhysReg);
 119:           }
 120:         };
 121:         tryAddHint(OtherMO);
 122:         if (CommuMO)
 123:           tryAddHint(CommuMO);
 124:       }
 125:     for (MCPhysReg OrderReg : Order)
 126:       if (TwoAddrHints.count(OrderReg))
 127:         Hints.push_back(OrderReg);
 128:   }
 129: 
 130:   if (MRI->getRegClass(VirtReg) == &SystemZ::GRX32BitRegClass) {
 131:     SmallVector<Register, 8> Worklist;
 132:     SmallSet<Register, 4> DoneRegs;
 133:     Worklist.push_back(VirtReg);
 134:     while (Worklist.size()) {
 135:       Register Reg = Worklist.pop_back_val();
 136:       if (!DoneRegs.insert(Reg).second)
 137:         continue;
 138: 
 139:       for (auto &Use : MRI->reg_instructions(Reg)) {
 140:         // For LOCRMux, see if the other operand is already a high or low
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 141-168 / 第 141-168 行
```cpp
 141:         // register, and in that case give the corresponding hints for
 142:         // VirtReg. LOCR instructions need both operands in either high or
 143:         // low parts. Same handling for SELRMux.
 144:         if (Use.getOpcode() == SystemZ::LOCRMux ||
 145:             Use.getOpcode() == SystemZ::SELRMux) {
 146:           MachineOperand &TrueMO = Use.getOperand(1);
 147:           MachineOperand &FalseMO = Use.getOperand(2);
 148:           const TargetRegisterClass *RC =
 149:             TRI->getCommonSubClass(getRC32(FalseMO, VRM, MRI),
 150:                                    getRC32(TrueMO, VRM, MRI));
 151:           if (Use.getOpcode() == SystemZ::SELRMux)
 152:             RC = TRI->getCommonSubClass(RC,
 153:                                         getRC32(Use.getOperand(0), VRM, MRI));
 154:           if (RC && RC != &SystemZ::GRX32BitRegClass) {
 155:             addHints(Order, Hints, RC, MRI);
 156:             // Return true to make these hints the only regs available to
 157:             // RA. This may mean extra spilling but since the alternative is
 158:             // a jump sequence expansion of the LOCRMux, it is preferred.
 159:             return true;
 160:           }
 161: 
 162:           // Add the other operand of the LOCRMux to the worklist.
 163:           Register OtherReg =
 164:               (TrueMO.getReg() == Reg ? FalseMO.getReg() : TrueMO.getReg());
 165:           if (MRI->getRegClass(OtherReg) == &SystemZ::GRX32BitRegClass)
 166:             Worklist.push_back(OtherReg);
 167:         } // end LOCRMux
 168:         else if (Use.getOpcode() == SystemZ::CHIMux ||
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 169-196 / 第 169-196 行
```cpp
 169:                  Use.getOpcode() == SystemZ::CFIMux) {
 170:           if (Use.getOperand(1).getImm() == 0) {
 171:             bool OnlyLMuxes = true;
 172:             for (MachineInstr &DefMI : MRI->def_instructions(VirtReg))
 173:               if (DefMI.getOpcode() != SystemZ::LMux)
 174:                 OnlyLMuxes = false;
 175:             if (OnlyLMuxes) {
 176:               addHints(Order, Hints, &SystemZ::GR32BitRegClass, MRI);
 177:               // Return false to make these hints preferred but not obligatory.
 178:               return false;
 179:             }
 180:           }
 181:         } // end CHIMux / CFIMux
 182:       }
 183:     }
 184:   }
 185: 
 186:   return BaseImplRetVal;
 187: }
 188: 
 189: const MCPhysReg *
 190: SystemZXPLINK64Registers::getCalleeSavedRegs(const MachineFunction *MF) const {
 191:   const SystemZSubtarget &Subtarget = MF->getSubtarget<SystemZSubtarget>();
 192:   return Subtarget.hasVector() ? CSR_SystemZ_XPLINK64_Vector_SaveList
 193:                                : CSR_SystemZ_XPLINK64_SaveList;
 194: }
 195: 
 196: const MCPhysReg *
```
- **EN**: The range implements or declares functions including `SystemZXPLINK64Registers::getCalleeSavedRegs`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZXPLINK64Registers::getCalleeSavedRegs` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 197-224 / 第 197-224 行
```cpp
 197: SystemZELFRegisters::getCalleeSavedRegs(const MachineFunction *MF) const {
 198:   const SystemZSubtarget &Subtarget = MF->getSubtarget<SystemZSubtarget>();
 199:   if (MF->getFunction().getCallingConv() == CallingConv::GHC)
 200:     return CSR_SystemZ_NoRegs_SaveList;
 201:   if (MF->getFunction().getCallingConv() == CallingConv::AnyReg)
 202:     return Subtarget.hasVector()? CSR_SystemZ_AllRegs_Vector_SaveList
 203:                                 : CSR_SystemZ_AllRegs_SaveList;
 204:   if (MF->getSubtarget().getTargetLowering()->supportSwiftError() &&
 205:       MF->getFunction().getAttributes().hasAttrSomewhere(
 206:           Attribute::SwiftError))
 207:     return CSR_SystemZ_SwiftError_SaveList;
 208:   return CSR_SystemZ_ELF_SaveList;
 209: }
 210: 
 211: const uint32_t *
 212: SystemZXPLINK64Registers::getCallPreservedMask(const MachineFunction &MF,
 213:                                                CallingConv::ID CC) const {
 214:   const SystemZSubtarget &Subtarget = MF.getSubtarget<SystemZSubtarget>();
 215:   return Subtarget.hasVector() ? CSR_SystemZ_XPLINK64_Vector_RegMask
 216:                                : CSR_SystemZ_XPLINK64_RegMask;
 217: }
 218: 
 219: const uint32_t *
 220: SystemZELFRegisters::getCallPreservedMask(const MachineFunction &MF,
 221:                                           CallingConv::ID CC) const {
 222:   const SystemZSubtarget &Subtarget = MF.getSubtarget<SystemZSubtarget>();
 223:   if (CC == CallingConv::GHC)
 224:     return CSR_SystemZ_NoRegs_RegMask;
```
- **EN**: The range implements or declares functions including `SystemZXPLINK64Registers::getCallPreservedMask`, `SystemZELFRegisters::getCallPreservedMask`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZXPLINK64Registers::getCallPreservedMask`, `SystemZELFRegisters::getCallPreservedMask` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 225-252 / 第 225-252 行
```cpp
 225:   if (CC == CallingConv::AnyReg)
 226:     return Subtarget.hasVector()? CSR_SystemZ_AllRegs_Vector_RegMask
 227:                                 : CSR_SystemZ_AllRegs_RegMask;
 228:   if (MF.getSubtarget().getTargetLowering()->supportSwiftError() &&
 229:       MF.getFunction().getAttributes().hasAttrSomewhere(
 230:           Attribute::SwiftError))
 231:     return CSR_SystemZ_SwiftError_RegMask;
 232:   return CSR_SystemZ_ELF_RegMask;
 233: }
 234: 
 235: SystemZRegisterInfo::SystemZRegisterInfo(unsigned int RA, unsigned int HwMode)
 236:     : SystemZGenRegisterInfo(RA, /*DwarfFlavour=*/0, /*EHFlavour=*/0, /*PC=*/0,
 237:                              HwMode) {}
 238: 
 239: const MCPhysReg *
 240: SystemZRegisterInfo::getCalleeSavedRegs(const MachineFunction *MF) const {
 241: 
 242:   const SystemZSubtarget *Subtarget = &MF->getSubtarget<SystemZSubtarget>();
 243:   SystemZCallingConventionRegisters *Regs = Subtarget->getSpecialRegisters();
 244: 
 245:   return Regs->getCalleeSavedRegs(MF);
 246: }
 247: 
 248: const uint32_t *
 249: SystemZRegisterInfo::getCallPreservedMask(const MachineFunction &MF,
 250:                                           CallingConv::ID CC) const {
 251: 
 252:   const SystemZSubtarget *Subtarget = &MF.getSubtarget<SystemZSubtarget>();
```
- **EN**: The range implements or declares functions including `SystemZGenRegisterInfo`, `SystemZRegisterInfo::getCallPreservedMask`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZGenRegisterInfo`, `SystemZRegisterInfo::getCallPreservedMask` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-280 / 第 253-280 行
```cpp
 253:   SystemZCallingConventionRegisters *Regs = Subtarget->getSpecialRegisters();
 254:   return Regs->getCallPreservedMask(MF, CC);
 255: }
 256: 
 257: const uint32_t *SystemZRegisterInfo::getNoPreservedMask() const {
 258:   return CSR_SystemZ_NoRegs_RegMask;
 259: }
 260: 
 261: BitVector
 262: SystemZRegisterInfo::getReservedRegs(const MachineFunction &MF) const {
 263:   BitVector Reserved(getNumRegs());
 264:   const SystemZFrameLowering *TFI = getFrameLowering(MF);
 265:   const SystemZSubtarget *Subtarget = &MF.getSubtarget<SystemZSubtarget>();
 266:   SystemZCallingConventionRegisters *Regs = Subtarget->getSpecialRegisters();
 267:   if (TFI->hasFP(MF))
 268:     // The frame pointer. Reserve all aliases.
 269:     for (MCRegAliasIterator AI(Regs->getFramePointerRegister(), this, true);
 270:          AI.isValid(); ++AI)
 271:       Reserved.set(*AI);
 272: 
 273:   // Reserve all aliases for the stack pointer.
 274:   for (MCRegAliasIterator AI(Regs->getStackPointerRegister(), this, true);
 275:        AI.isValid(); ++AI)
 276:     Reserved.set(*AI);
 277: 
 278:   // A0 and A1 hold the thread pointer.
 279:   Reserved.set(SystemZ::A0);
 280:   Reserved.set(SystemZ::A1);
```
- **EN**: The range implements or declares functions including `SystemZRegisterInfo::getReservedRegs`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZRegisterInfo::getReservedRegs` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 281-308 / 第 281-308 行
```cpp
 281: 
 282:   // FPC is the floating-point control register.
 283:   Reserved.set(SystemZ::FPC);
 284: 
 285:   return Reserved;
 286: }
 287: 
 288: bool
 289: SystemZRegisterInfo::eliminateFrameIndex(MachineBasicBlock::iterator MI,
 290:                                          int SPAdj, unsigned FIOperandNum,
 291:                                          RegScavenger *RS) const {
 292:   assert(SPAdj == 0 && "Outgoing arguments should be part of the frame");
 293: 
 294:   MachineBasicBlock &MBB = *MI->getParent();
 295:   MachineFunction &MF = *MBB.getParent();
 296:   auto *TII = MF.getSubtarget<SystemZSubtarget>().getInstrInfo();
 297:   const SystemZFrameLowering *TFI = getFrameLowering(MF);
 298:   DebugLoc DL = MI->getDebugLoc();
 299: 
 300:   // Decompose the frame index into a base and offset.
 301:   int FrameIndex = MI->getOperand(FIOperandNum).getIndex();
 302:   Register BasePtr;
 303:   int64_t Offset =
 304:       (TFI->getFrameIndexReference(MF, FrameIndex, BasePtr).getFixed() +
 305:        MI->getOperand(FIOperandNum + 1).getImm());
 306: 
 307:   // Special handling of dbg_value instructions.
 308:   if (MI->isDebugValue()) {
```
- **EN**: The range implements or declares functions including `SystemZRegisterInfo::eliminateFrameIndex`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZRegisterInfo::eliminateFrameIndex` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 309-336 / 第 309-336 行
```cpp
 309:     MI->getOperand(FIOperandNum).ChangeToRegister(BasePtr, /*isDef*/ false);
 310:     if (MI->isNonListDebugValue()) {
 311:       MI->getDebugOffset().ChangeToImmediate(Offset);
 312:     } else {
 313:       unsigned OpIdx = MI->getDebugOperandIndex(&MI->getOperand(FIOperandNum));
 314:       SmallVector<uint64_t, 3> Ops;
 315:       DIExpression::appendOffset(
 316:           Ops, TFI->getFrameIndexReference(MF, FrameIndex, BasePtr).getFixed());
 317:       MI->getDebugExpressionOp().setMetadata(
 318:           DIExpression::appendOpsToArg(MI->getDebugExpression(), Ops, OpIdx));
 319:     }
 320:     return false;
 321:   }
 322: 
 323:   // See if the offset is in range, or if an equivalent instruction that
 324:   // accepts the offset exists.
 325:   unsigned Opcode = MI->getOpcode();
 326:   unsigned OpcodeForOffset = TII->getOpcodeForOffset(Opcode, Offset, &*MI);
 327:   if (OpcodeForOffset) {
 328:     if (OpcodeForOffset == SystemZ::LE &&
 329:         MF.getSubtarget<SystemZSubtarget>().hasVector()) {
 330:       // If LE is ok for offset, use LDE instead on z13.
 331:       OpcodeForOffset = SystemZ::LDE32;
 332:     }
 333:     MI->getOperand(FIOperandNum).ChangeToRegister(BasePtr, false);
 334:   }
 335:   else {
 336:     // Create an anchor point that is in range.  Start at 0xffff so that
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 337-364 / 第 337-364 行
```cpp
 337:     // can use LLILH to load the immediate.
 338:     int64_t OldOffset = Offset;
 339:     int64_t Mask = 0xffff;
 340:     do {
 341:       Offset = OldOffset & Mask;
 342:       OpcodeForOffset = TII->getOpcodeForOffset(Opcode, Offset);
 343:       Mask >>= 1;
 344:       assert(Mask && "One offset must be OK");
 345:     } while (!OpcodeForOffset);
 346: 
 347:     Register ScratchReg =
 348:         MF.getRegInfo().createVirtualRegister(&SystemZ::ADDR64BitRegClass);
 349:     int64_t HighOffset = OldOffset - Offset;
 350: 
 351:     if (MI->getDesc().TSFlags & SystemZII::HasIndex
 352:         && MI->getOperand(FIOperandNum + 2).getReg() == 0) {
 353:       // Load the offset into the scratch register and use it as an index.
 354:       // The scratch register then dies here.
 355:       TII->loadImmediate(MBB, MI, ScratchReg, HighOffset);
 356:       MI->getOperand(FIOperandNum).ChangeToRegister(BasePtr, false);
 357:       MI->getOperand(FIOperandNum + 2).ChangeToRegister(ScratchReg,
 358:                                                         false, false, true);
 359:     } else {
 360:       // Load the anchor address into a scratch register.
 361:       unsigned LAOpcode = TII->getOpcodeForOffset(SystemZ::LA, HighOffset);
 362:       if (LAOpcode)
 363:         BuildMI(MBB, MI, DL, TII->get(LAOpcode),ScratchReg)
 364:           .addReg(BasePtr).addImm(HighOffset).addReg(0);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 365-392 / 第 365-392 行
```cpp
 365:       else {
 366:         // Load the high offset into the scratch register and use it as
 367:         // an index.
 368:         TII->loadImmediate(MBB, MI, ScratchReg, HighOffset);
 369:         BuildMI(MBB, MI, DL, TII->get(SystemZ::LA), ScratchReg)
 370:           .addReg(BasePtr, RegState::Kill).addImm(0).addReg(ScratchReg);
 371:       }
 372: 
 373:       // Use the scratch register as the base.  It then dies here.
 374:       MI->getOperand(FIOperandNum).ChangeToRegister(ScratchReg,
 375:                                                     false, false, true);
 376:     }
 377:   }
 378:   MI->setDesc(TII->get(OpcodeForOffset));
 379:   MI->getOperand(FIOperandNum + 1).ChangeToImmediate(Offset);
 380:   return false;
 381: }
 382: 
 383: bool SystemZRegisterInfo::shouldCoalesce(MachineInstr *MI,
 384:                                          const TargetRegisterClass *SrcRC,
 385:                                          unsigned SubReg,
 386:                                          const TargetRegisterClass *DstRC,
 387:                                          unsigned DstSubReg,
 388:                                          const TargetRegisterClass *NewRC,
 389:                                          LiveIntervals &LIS) const {
 390:   assert (MI->isCopy() && "Only expecting COPY instructions");
 391: 
 392:   // Coalesce anything which is not a COPY involving a subreg to/from GR128.
```
- **EN**: The range implements or declares functions including `BuildMI`, `SystemZRegisterInfo::shouldCoalesce`.
- **CN**: 这一段实现或声明了 `BuildMI`, `SystemZRegisterInfo::shouldCoalesce` 等函数。

### Lines 393-420 / 第 393-420 行
```cpp
 393:   if (!(NewRC->hasSuperClassEq(&SystemZ::GR128BitRegClass) &&
 394:         (getRegSizeInBits(*SrcRC) <= 64 || getRegSizeInBits(*DstRC) <= 64) &&
 395:         !MI->getOperand(1).isUndef()))
 396:     return true;
 397: 
 398:   // Allow coalescing of a GR128 subreg COPY only if the subreg liverange is
 399:   // local to one MBB with not too many interferring physreg clobbers. Otherwise
 400:   // regalloc may run out of registers.
 401:   unsigned SubregOpIdx = getRegSizeInBits(*SrcRC) == 128 ? 0 : 1;
 402:   LiveInterval &LI = LIS.getInterval(MI->getOperand(SubregOpIdx).getReg());
 403: 
 404:   // Check that the subreg is local to MBB.
 405:   MachineBasicBlock *MBB = MI->getParent();
 406:   MachineInstr *FirstMI = LIS.getInstructionFromIndex(LI.beginIndex());
 407:   MachineInstr *LastMI = LIS.getInstructionFromIndex(LI.endIndex());
 408:   if (!FirstMI || FirstMI->getParent() != MBB ||
 409:       !LastMI || LastMI->getParent() != MBB)
 410:     return false;
 411: 
 412:   // Check if coalescing seems safe by finding the set of clobbered physreg
 413:   // pairs in the region.
 414:   BitVector PhysClobbered(getNumRegs());
 415:   for (MachineBasicBlock::iterator MII = FirstMI,
 416:                                    MEE = std::next(LastMI->getIterator());
 417:        MII != MEE; ++MII)
 418:     for (const MachineOperand &MO : MII->operands())
 419:       if (MO.isReg() && MO.getReg().isPhysical()) {
 420:         for (MCPhysReg SI : superregs_inclusive(MO.getReg()))
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 421-448 / 第 421-448 行
```cpp
 421:           if (NewRC->contains(SI)) {
 422:             PhysClobbered.set(SI);
 423:             break;
 424:           }
 425:       }
 426: 
 427:   // Demand an arbitrary margin of free regs.
 428:   unsigned const DemandedFreeGR128 = 3;
 429:   if (PhysClobbered.count() > (NewRC->getNumRegs() - DemandedFreeGR128))
 430:     return false;
 431: 
 432:   return true;
 433: }
 434: 
 435: Register
 436: SystemZRegisterInfo::getFrameRegister(const MachineFunction &MF) const {
 437:   const SystemZFrameLowering *TFI = getFrameLowering(MF);
 438:   const SystemZSubtarget *Subtarget = &MF.getSubtarget<SystemZSubtarget>();
 439:   SystemZCallingConventionRegisters *Regs = Subtarget->getSpecialRegisters();
 440: 
 441:   return TFI->hasFP(MF) ? Regs->getFramePointerRegister()
 442:                         : Regs->getStackPointerRegister();
 443: }
 444: 
 445: const TargetRegisterClass *
 446: SystemZRegisterInfo::getCrossCopyRegClass(const TargetRegisterClass *RC) const {
 447:   if (RC == &SystemZ::CCRRegClass)
 448:     return &SystemZ::GR32BitRegClass;
```
- **EN**: The range implements or declares functions including `SystemZRegisterInfo::getFrameRegister`, `SystemZRegisterInfo::getCrossCopyRegClass`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZRegisterInfo::getFrameRegister`, `SystemZRegisterInfo::getCrossCopyRegClass` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 449-451 / 第 449-451 行
```cpp
 449:   return RC;
 450: }
 451: 
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target register information for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。

## Dependencies / 依赖关系
- `SystemZRegisterInfo.h`
- `SystemZInstrInfo.h`
- `SystemZSubtarget.h`
- `llvm/ADT/SmallSet.h`
- `llvm/CodeGen/LiveIntervals.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/CodeGen/TargetFrameLowering.h`
- `llvm/CodeGen/VirtRegMap.h`
- `llvm/IR/DebugInfoMetadata.h`
- `SystemZGenRegisterInfo.inc`
