# SystemZElimCompare.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZElimCompare.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Provides target-specific implementation details for the SystemZ backend.
- **用途 (CN)**: 提供 SystemZ 后端的目标相关实现细节。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```cpp
   1: //===-- SystemZElimCompare.cpp - Eliminate comparison instructions --------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This pass:
  10: // (1) tries to remove compares if CC already contains the required information
  11: // (2) fuses compares and branches into COMPARE AND BRANCH instructions
  12: //
  13: //===----------------------------------------------------------------------===//
  14: 
  15: #include "SystemZ.h"
  16: #include "SystemZInstrInfo.h"
  17: #include "SystemZTargetMachine.h"
  18: #include "llvm/ADT/SmallVector.h"
  19: #include "llvm/ADT/Statistic.h"
  20: #include "llvm/ADT/StringRef.h"
  21: #include "llvm/CodeGen/LiveRegUnits.h"
  22: #include "llvm/CodeGen/MachineBasicBlock.h"
  23: #include "llvm/CodeGen/MachineFunction.h"
  24: #include "llvm/CodeGen/MachineFunctionPass.h"
  25: #include "llvm/CodeGen/MachineInstr.h"
  26: #include "llvm/CodeGen/MachineInstrBuilder.h"
  27: #include "llvm/CodeGen/MachineOperand.h"
  28: #include "llvm/CodeGen/TargetRegisterInfo.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZ.h`, `SystemZInstrInfo.h`, `SystemZTargetMachine.h`, `SmallVector.h`, `Statistic.h`, `StringRef.h` that expose the LLVM and target interfaces used in later logic. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZ.h`, `SystemZInstrInfo.h`, `SystemZTargetMachine.h`, `SmallVector.h`, `Statistic.h`, `StringRef.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 29-56 / 第 29-56 行
```cpp
  29: #include "llvm/CodeGen/TargetSubtargetInfo.h"
  30: #include "llvm/MC/MCInstrDesc.h"
  31: #include <cassert>
  32: #include <cstdint>
  33: 
  34: using namespace llvm;
  35: 
  36: #define DEBUG_TYPE "systemz-elim-compare"
  37: 
  38: STATISTIC(BranchOnCounts, "Number of branch-on-count instructions");
  39: STATISTIC(LoadAndTraps, "Number of load-and-trap instructions");
  40: STATISTIC(EliminatedComparisons, "Number of eliminated comparisons");
  41: STATISTIC(FusedComparisons, "Number of fused compare-and-branch instructions");
  42: 
  43: namespace {
  44: 
  45: // Represents the references to a particular register in one or more
  46: // instructions.
  47: struct Reference {
  48:   Reference() = default;
  49: 
  50:   Reference &operator|=(const Reference &Other) {
  51:     Def |= Other.Def;
  52:     Use |= Other.Use;
  53:     return *this;
  54:   }
  55: 
  56:   explicit operator bool() const { return Def || Use; }
```
- **EN**: It imports dependencies such as `TargetSubtargetInfo.h`, `MCInstrDesc.h`, `cassert`, `cstdint` that expose the LLVM and target interfaces used in later logic. It introduces interface types such as `Reference`, shaping how other backend components interact with this file. The range implements or declares functions including `bool`.
- **CN**: 它引入了 `TargetSubtargetInfo.h`, `MCInstrDesc.h`, `cassert`, `cstdint` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 它引入了 `Reference` 等接口类型，定义了其他后端组件与本文件交互的方式。 这一段实现或声明了 `bool` 等函数。

### Lines 57-84 / 第 57-84 行
```cpp
  57: 
  58:   // True if the register is defined or used in some form, either directly or
  59:   // via a sub- or super-register.
  60:   bool Def = false;
  61:   bool Use = false;
  62: };
  63: 
  64: class SystemZElimCompare : public MachineFunctionPass {
  65: public:
  66:   static char ID;
  67: 
  68:   SystemZElimCompare() : MachineFunctionPass(ID) {}
  69: 
  70:   bool processBlock(MachineBasicBlock &MBB);
  71:   bool runOnMachineFunction(MachineFunction &F) override;
  72: 
  73:   MachineFunctionProperties getRequiredProperties() const override {
  74:     return MachineFunctionProperties().setNoVRegs();
  75:   }
  76: 
  77: private:
  78:   Reference getRegReferences(MachineInstr &MI, unsigned Reg);
  79:   bool convertToBRCT(MachineInstr &MI, MachineInstr &Compare,
  80:                      SmallVectorImpl<MachineInstr *> &CCUsers);
  81:   bool convertToLoadAndTrap(MachineInstr &MI, MachineInstr &Compare,
  82:                             SmallVectorImpl<MachineInstr *> &CCUsers);
  83:   bool convertToLoadAndTest(MachineInstr &MI, MachineInstr &Compare,
  84:                             SmallVectorImpl<MachineInstr *> &CCUsers);
```
- **EN**: This block declares or refines TableGen records such as `SystemZElimCompare`. The range implements or declares functions including `SystemZElimCompare`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `SystemZElimCompare` 等 TableGen 记录。 这一段实现或声明了 `SystemZElimCompare` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 85-112 / 第 85-112 行
```cpp
  85:   bool convertToLogical(MachineInstr &MI, MachineInstr &Compare,
  86:                         SmallVectorImpl<MachineInstr *> &CCUsers);
  87:   bool adjustCCMasksForInstr(MachineInstr &MI, MachineInstr &Compare,
  88:                              SmallVectorImpl<MachineInstr *> &CCUsers,
  89:                              unsigned ConvOpc = 0);
  90:   bool optimizeCompareZero(MachineInstr &Compare,
  91:                            SmallVectorImpl<MachineInstr *> &CCUsers);
  92:   bool fuseCompareOperations(MachineInstr &Compare,
  93:                              SmallVectorImpl<MachineInstr *> &CCUsers);
  94: 
  95:   const SystemZInstrInfo *TII = nullptr;
  96:   const TargetRegisterInfo *TRI = nullptr;
  97: };
  98: 
  99: char SystemZElimCompare::ID = 0;
 100: 
 101: } // end anonymous namespace
 102: 
 103: INITIALIZE_PASS(SystemZElimCompare, DEBUG_TYPE,
 104:                 "SystemZ Comparison Elimination", false, false)
 105: 
 106: // Returns true if MI is an instruction whose output equals the value in Reg.
 107: static bool preservesValueOf(MachineInstr &MI, unsigned Reg) {
 108:   switch (MI.getOpcode()) {
 109:   case SystemZ::LR:
 110:   case SystemZ::LGR:
 111:   case SystemZ::LGFR:
 112:   case SystemZ::LTR:
```
- **EN**: The range implements or declares functions including `preservesValueOf`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `preservesValueOf` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 113-140 / 第 113-140 行
```cpp
 113:   case SystemZ::LTGR:
 114:   case SystemZ::LTGFR:
 115:     if (MI.getOperand(1).getReg() == Reg)
 116:       return true;
 117:   }
 118: 
 119:   return false;
 120: }
 121: 
 122: // Return true if any CC result of MI would (perhaps after conversion)
 123: // reflect the value of Reg.
 124: static bool resultTests(MachineInstr &MI, unsigned Reg) {
 125:   if (MI.getNumOperands() > 0 && MI.getOperand(0).isReg() &&
 126:       MI.getOperand(0).isDef() && MI.getOperand(0).getReg() == Reg)
 127:     return true;
 128: 
 129:   return (preservesValueOf(MI, Reg));
 130: }
 131: 
 132: // Describe the references to Reg or any of its aliases in MI.
 133: Reference SystemZElimCompare::getRegReferences(MachineInstr &MI, unsigned Reg) {
 134:   Reference Ref;
 135:   if (MI.isDebugInstr())
 136:     return Ref;
 137: 
 138:   for (const MachineOperand &MO : MI.operands()) {
 139:     if (MO.isReg()) {
 140:       if (Register MOReg = MO.getReg()) {
```
- **EN**: The range implements or declares functions including `resultTests`, `SystemZElimCompare::getRegReferences`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `resultTests`, `SystemZElimCompare::getRegReferences` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 141-168 / 第 141-168 行
```cpp
 141:         if (TRI->regsOverlap(MOReg, Reg)) {
 142:           if (MO.isUse())
 143:             Ref.Use = true;
 144:           else if (MO.isDef())
 145:             Ref.Def = true;
 146:         }
 147:       }
 148:     }
 149:   }
 150:   return Ref;
 151: }
 152: 
 153: // Compare compares the result of MI against zero.  If MI is an addition
 154: // of -1 and if CCUsers is a single branch on nonzero, eliminate the addition
 155: // and convert the branch to a BRCT(G) or BRCTH.  Return true on success.
 156: bool SystemZElimCompare::convertToBRCT(
 157:     MachineInstr &MI, MachineInstr &Compare,
 158:     SmallVectorImpl<MachineInstr *> &CCUsers) {
 159:   // Check whether we have an addition of -1.
 160:   unsigned Opcode = MI.getOpcode();
 161:   unsigned BRCT;
 162:   if (Opcode == SystemZ::AHI)
 163:     BRCT = SystemZ::BRCT;
 164:   else if (Opcode == SystemZ::AGHI)
 165:     BRCT = SystemZ::BRCTG;
 166:   else if (Opcode == SystemZ::AIH)
 167:     BRCT = SystemZ::BRCTH;
 168:   else
```
- **EN**: The range implements or declares functions including `SystemZElimCompare::convertToBRCT`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZElimCompare::convertToBRCT` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 169-196 / 第 169-196 行
```cpp
 169:     return false;
 170:   if (MI.getOperand(2).getImm() != -1)
 171:     return false;
 172: 
 173:   // Check whether we have a single JLH.
 174:   if (CCUsers.size() != 1)
 175:     return false;
 176:   MachineInstr *Branch = CCUsers[0];
 177:   if (Branch->getOpcode() != SystemZ::BRC ||
 178:       Branch->getOperand(0).getImm() != SystemZ::CCMASK_ICMP ||
 179:       Branch->getOperand(1).getImm() != SystemZ::CCMASK_CMP_NE)
 180:     return false;
 181: 
 182:   // We already know that there are no references to the register between
 183:   // MI and Compare.  Make sure that there are also no references between
 184:   // Compare and Branch.
 185:   unsigned SrcReg = TII->getCompareSourceReg(Compare);
 186:   MachineBasicBlock::iterator MBBI = Compare, MBBE = Branch;
 187:   for (++MBBI; MBBI != MBBE; ++MBBI)
 188:     if (getRegReferences(*MBBI, SrcReg))
 189:       return false;
 190: 
 191:   // The transformation is OK.  Rebuild Branch as a BRCT(G) or BRCTH.
 192:   MachineOperand Target(Branch->getOperand(2));
 193:   while (Branch->getNumOperands())
 194:     Branch->removeOperand(0);
 195:   Branch->setDesc(TII->get(BRCT));
 196:   MachineInstrBuilder MIB(*Branch->getParent()->getParent(), Branch);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 197-224 / 第 197-224 行
```cpp
 197:   MIB.add(MI.getOperand(0)).add(MI.getOperand(1)).add(Target);
 198:   // Add a CC def to BRCT(G), since we may have to split them again if the
 199:   // branch displacement overflows.  BRCTH has a 32-bit displacement, so
 200:   // this is not necessary there.
 201:   if (BRCT != SystemZ::BRCTH)
 202:     MIB.addReg(SystemZ::CC, RegState::ImplicitDefine | RegState::Dead);
 203:   // The debug instr tracking for the counter now used by BRCT needs to be
 204:   // updated.
 205:   MI.getParent()->getParent()->substituteDebugValuesForInst(MI, *MIB);
 206:   MI.eraseFromParent();
 207:   return true;
 208: }
 209: 
 210: // Compare compares the result of MI against zero.  If MI is a suitable load
 211: // instruction and if CCUsers is a single conditional trap on zero, eliminate
 212: // the load and convert the branch to a load-and-trap.  Return true on success.
 213: bool SystemZElimCompare::convertToLoadAndTrap(
 214:     MachineInstr &MI, MachineInstr &Compare,
 215:     SmallVectorImpl<MachineInstr *> &CCUsers) {
 216:   unsigned LATOpcode = TII->getLoadAndTrap(MI.getOpcode());
 217:   if (!LATOpcode)
 218:     return false;
 219: 
 220:   // Check whether we have a single CondTrap that traps on zero.
 221:   if (CCUsers.size() != 1)
 222:     return false;
 223:   MachineInstr *Branch = CCUsers[0];
 224:   if (Branch->getOpcode() != SystemZ::CondTrap ||
```
- **EN**: The range implements or declares functions including `SystemZElimCompare::convertToLoadAndTrap`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZElimCompare::convertToLoadAndTrap` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 225-252 / 第 225-252 行
```cpp
 225:       Branch->getOperand(0).getImm() != SystemZ::CCMASK_ICMP ||
 226:       Branch->getOperand(1).getImm() != SystemZ::CCMASK_CMP_EQ)
 227:     return false;
 228: 
 229:   // We already know that there are no references to the register between
 230:   // MI and Compare.  Make sure that there are also no references between
 231:   // Compare and Branch.
 232:   unsigned SrcReg = TII->getCompareSourceReg(Compare);
 233:   MachineBasicBlock::iterator MBBI = Compare, MBBE = Branch;
 234:   for (++MBBI; MBBI != MBBE; ++MBBI)
 235:     if (getRegReferences(*MBBI, SrcReg))
 236:       return false;
 237: 
 238:   // The transformation is OK.  Rebuild Branch as a load-and-trap.
 239:   while (Branch->getNumOperands())
 240:     Branch->removeOperand(0);
 241:   Branch->setDesc(TII->get(LATOpcode));
 242:   MachineInstrBuilder(*Branch->getParent()->getParent(), Branch)
 243:       .add(MI.getOperand(0))
 244:       .add(MI.getOperand(1))
 245:       .add(MI.getOperand(2))
 246:       .add(MI.getOperand(3));
 247:   // The debug instr tracking for the load target now used by the load-and-trap
 248:   // needs to be updated.
 249:   MI.getParent()->getParent()->substituteDebugValuesForInst(MI, *Branch);
 250:   MI.eraseFromParent();
 251:   return true;
 252: }
```
- **EN**: The range implements or declares functions including `MachineInstrBuilder`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `MachineInstrBuilder` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 253-280 / 第 253-280 行
```cpp
 253: 
 254: // If MI is a load instruction, try to convert it into a LOAD AND TEST.
 255: // Return true on success.
 256: bool SystemZElimCompare::convertToLoadAndTest(
 257:     MachineInstr &MI, MachineInstr &Compare,
 258:     SmallVectorImpl<MachineInstr *> &CCUsers) {
 259: 
 260:   // Try to adjust CC masks for the LOAD AND TEST opcode that could replace MI.
 261:   unsigned Opcode = TII->getLoadAndTest(MI.getOpcode());
 262:   if (!Opcode || !adjustCCMasksForInstr(MI, Compare, CCUsers, Opcode))
 263:     return false;
 264: 
 265:   // Rebuild to get the CC operand in the right place.
 266:   auto MIB = BuildMI(*MI.getParent(), MI, MI.getDebugLoc(), TII->get(Opcode));
 267:   for (const auto &MO : MI.operands())
 268:     MIB.add(MO);
 269:   MIB.setMemRefs(MI.memoperands());
 270:   // The debug instr tracking for the load target now needs to be updated
 271:   // because the load has moved to a new instruction
 272:   MI.getParent()->getParent()->substituteDebugValuesForInst(MI, *MIB);
 273:   MI.eraseFromParent();
 274: 
 275:   // Mark instruction as not raising an FP exception if applicable.  We already
 276:   // verified earlier that this move is valid.
 277:   if (!Compare.mayRaiseFPException())
 278:     MIB.setMIFlag(MachineInstr::MIFlag::NoFPExcept);
 279: 
 280:   return true;
```
- **EN**: The range implements or declares functions including `SystemZElimCompare::convertToLoadAndTest`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZElimCompare::convertToLoadAndTest` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 281-308 / 第 281-308 行
```cpp
 281: }
 282: 
 283: // See if MI is an instruction with an equivalent "logical" opcode that can
 284: // be used and replace MI. This is useful for EQ/NE comparisons where the
 285: // "nsw" flag is missing since the "logical" opcode always sets CC to reflect
 286: // the result being zero or non-zero.
 287: bool SystemZElimCompare::convertToLogical(
 288:     MachineInstr &MI, MachineInstr &Compare,
 289:     SmallVectorImpl<MachineInstr *> &CCUsers) {
 290: 
 291:   unsigned ConvOpc = 0;
 292:   switch (MI.getOpcode()) {
 293:   case SystemZ::AR:   ConvOpc = SystemZ::ALR;   break;
 294:   case SystemZ::ARK:  ConvOpc = SystemZ::ALRK;  break;
 295:   case SystemZ::AGR:  ConvOpc = SystemZ::ALGR;  break;
 296:   case SystemZ::AGRK: ConvOpc = SystemZ::ALGRK; break;
 297:   case SystemZ::A:    ConvOpc = SystemZ::AL;    break;
 298:   case SystemZ::AY:   ConvOpc = SystemZ::ALY;   break;
 299:   case SystemZ::AG:   ConvOpc = SystemZ::ALG;   break;
 300:   default: break;
 301:   }
 302:   if (!ConvOpc || !adjustCCMasksForInstr(MI, Compare, CCUsers, ConvOpc))
 303:     return false;
 304: 
 305:   // Operands should be identical, so just change the opcode and remove the
 306:   // dead flag on CC.
 307:   MI.setDesc(TII->get(ConvOpc));
 308:   MI.clearRegisterDeads(SystemZ::CC);
```
- **EN**: The range implements or declares functions including `SystemZElimCompare::convertToLogical`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZElimCompare::convertToLogical` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 309-336 / 第 309-336 行
```cpp
 309:   return true;
 310: }
 311: 
 312: #ifndef NDEBUG
 313: static bool isAddWithImmediate(unsigned Opcode) {
 314:   switch(Opcode) {
 315:   case SystemZ::AHI:
 316:   case SystemZ::AHIK:
 317:   case SystemZ::AGHI:
 318:   case SystemZ::AGHIK:
 319:   case SystemZ::AFI:
 320:   case SystemZ::AIH:
 321:   case SystemZ::AGFI:
 322:     return true;
 323:   default: break;
 324:   }
 325:   return false;
 326: }
 327: #endif
 328: 
 329: // The CC users in CCUsers are testing the result of a comparison of some
 330: // value X against zero and we know that any CC value produced by MI would
 331: // also reflect the value of X.  ConvOpc may be used to pass the transfomed
 332: // opcode MI will have if this succeeds.  Try to adjust CCUsers so that they
 333: // test the result of MI directly, returning true on success.  Leave
 334: // everything unchanged on failure.
 335: bool SystemZElimCompare::adjustCCMasksForInstr(
 336:     MachineInstr &MI, MachineInstr &Compare,
```
- **EN**: The range implements or declares functions including `isAddWithImmediate`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isAddWithImmediate` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 337-364 / 第 337-364 行
```cpp
 337:     SmallVectorImpl<MachineInstr *> &CCUsers,
 338:     unsigned ConvOpc) {
 339:   unsigned CompareFlags = Compare.getDesc().TSFlags;
 340:   unsigned CompareCCValues = SystemZII::getCCValues(CompareFlags);
 341:   int Opcode = (ConvOpc ? ConvOpc : MI.getOpcode());
 342:   const MCInstrDesc &Desc = TII->get(Opcode);
 343:   unsigned MIFlags = Desc.TSFlags;
 344: 
 345:   // If Compare may raise an FP exception, we can only eliminate it
 346:   // if MI itself would have already raised the exception.
 347:   if (Compare.mayRaiseFPException()) {
 348:     // If the caller will change MI to use ConvOpc, only test whether
 349:     // ConvOpc is suitable; it is on the caller to set the MI flag.
 350:     if (ConvOpc && !Desc.mayRaiseFPException())
 351:       return false;
 352:     // If the caller will not change MI, we test the MI flag here.
 353:     if (!ConvOpc && !MI.mayRaiseFPException())
 354:       return false;
 355:   }
 356: 
 357:   // See which compare-style condition codes are available.
 358:   unsigned CCValues = SystemZII::getCCValues(MIFlags);
 359:   unsigned ReusableCCMask = CCValues;
 360:   // For unsigned comparisons with zero, only equality makes sense.
 361:   if (CompareFlags & SystemZII::IsLogical)
 362:     ReusableCCMask &= SystemZ::CCMASK_CMP_EQ;
 363:   unsigned OFImplies = 0;
 364:   bool LogicalMI = false;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 365-392 / 第 365-392 行
```cpp
 365:   bool MIEquivalentToCmp = false;
 366:   if (MI.getFlag(MachineInstr::NoSWrap) &&
 367:       (MIFlags & SystemZII::CCIfNoSignedWrap)) {
 368:     // If MI has the NSW flag set in combination with the
 369:     // SystemZII::CCIfNoSignedWrap flag, all CCValues are valid.
 370:   }
 371:   else if ((MIFlags & SystemZII::CCIfNoSignedWrap) &&
 372:            MI.getOperand(2).isImm()) {
 373:     // Signed addition of immediate. If adding a positive immediate
 374:     // overflows, the result must be less than zero. If adding a negative
 375:     // immediate overflows, the result must be larger than zero (except in
 376:     // the special case of adding the minimum value of the result range, in
 377:     // which case we cannot predict whether the result is larger than or
 378:     // equal to zero).
 379:     assert(isAddWithImmediate(Opcode) && "Expected an add with immediate.");
 380:     assert(!MI.mayLoadOrStore() && "Expected an immediate term.");
 381:     int64_t RHS = MI.getOperand(2).getImm();
 382:     if (SystemZ::GRX32BitRegClass.contains(MI.getOperand(0).getReg()) &&
 383:         RHS == INT32_MIN)
 384:       return false;
 385:     OFImplies = (RHS > 0 ? SystemZ::CCMASK_CMP_LT : SystemZ::CCMASK_CMP_GT);
 386:   }
 387:   else if ((MIFlags & SystemZII::IsLogical) && CCValues) {
 388:     // Use CCMASK_CMP_EQ to match with CCUsers. On success CCMask:s will be
 389:     // converted to CCMASK_LOGICAL_ZERO or CCMASK_LOGICAL_NONZERO.
 390:     LogicalMI = true;
 391:     ReusableCCMask = SystemZ::CCMASK_CMP_EQ;
 392:   }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 393-420 / 第 393-420 行
```cpp
 393:   else {
 394:     ReusableCCMask &= SystemZII::getCompareZeroCCMask(MIFlags);
 395:     assert((ReusableCCMask & ~CCValues) == 0 && "Invalid CCValues");
 396:     MIEquivalentToCmp =
 397:       ReusableCCMask == CCValues && CCValues == CompareCCValues;
 398:   }
 399:   if (ReusableCCMask == 0)
 400:     return false;
 401: 
 402:   if (!MIEquivalentToCmp) {
 403:     // Now check whether these flags are enough for all users.
 404:     SmallVector<MachineOperand *, 4> AlterMasks;
 405:     for (MachineInstr *CCUserMI : CCUsers) {
 406:       // Fail if this isn't a use of CC that we understand.
 407:       unsigned Flags = CCUserMI->getDesc().TSFlags;
 408:       unsigned FirstOpNum;
 409:       if (Flags & SystemZII::CCMaskFirst)
 410:         FirstOpNum = 0;
 411:       else if (Flags & SystemZII::CCMaskLast)
 412:         FirstOpNum = CCUserMI->getNumExplicitOperands() - 2;
 413:       else
 414:         return false;
 415: 
 416:       // Check whether the instruction predicate treats all CC values
 417:       // outside of ReusableCCMask in the same way.  In that case it
 418:       // doesn't matter what those CC values mean.
 419:       unsigned CCValid = CCUserMI->getOperand(FirstOpNum).getImm();
 420:       unsigned CCMask = CCUserMI->getOperand(FirstOpNum + 1).getImm();
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 421-448 / 第 421-448 行
```cpp
 421:       assert(CCValid == CompareCCValues && (CCMask & ~CCValid) == 0 &&
 422:              "Corrupt CC operands of CCUser.");
 423:       unsigned OutValid = ~ReusableCCMask & CCValid;
 424:       unsigned OutMask = ~ReusableCCMask & CCMask;
 425:       if (OutMask != 0 && OutMask != OutValid)
 426:         return false;
 427: 
 428:       AlterMasks.push_back(&CCUserMI->getOperand(FirstOpNum));
 429:       AlterMasks.push_back(&CCUserMI->getOperand(FirstOpNum + 1));
 430:     }
 431: 
 432:     // All users are OK.  Adjust the masks for MI.
 433:     for (unsigned I = 0, E = AlterMasks.size(); I != E; I += 2) {
 434:       AlterMasks[I]->setImm(CCValues);
 435:       unsigned CCMask = AlterMasks[I + 1]->getImm();
 436:       if (LogicalMI) {
 437:         // Translate the CCMask into its "logical" value.
 438:         CCMask = (CCMask == SystemZ::CCMASK_CMP_EQ ?
 439:                   SystemZ::CCMASK_LOGICAL_ZERO : SystemZ::CCMASK_LOGICAL_NONZERO);
 440:         CCMask &= CCValues; // Logical subtracts never set CC=0.
 441:       } else {
 442:         if (CCMask & ~ReusableCCMask)
 443:           CCMask = (CCMask & ReusableCCMask) | (CCValues & ~ReusableCCMask);
 444:         CCMask |= (CCMask & OFImplies) ? SystemZ::CCMASK_ARITH_OVERFLOW : 0;
 445:       }
 446:       AlterMasks[I + 1]->setImm(CCMask);
 447:     }
 448:   }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 449-476 / 第 449-476 行
```cpp
 449: 
 450:   // CC is now live after MI.
 451:   if (!ConvOpc)
 452:     MI.clearRegisterDeads(SystemZ::CC);
 453: 
 454:   // Check if MI lies before Compare.
 455:   bool BeforeCmp = false;
 456:   MachineBasicBlock::iterator MBBI = MI, MBBE = MI.getParent()->end();
 457:   for (++MBBI; MBBI != MBBE; ++MBBI)
 458:     if (MBBI == Compare) {
 459:       BeforeCmp = true;
 460:       break;
 461:     }
 462: 
 463:   // Clear any intervening kills of CC.
 464:   if (BeforeCmp) {
 465:     MachineBasicBlock::iterator MBBI = MI, MBBE = Compare;
 466:     for (++MBBI; MBBI != MBBE; ++MBBI)
 467:       MBBI->clearRegisterKills(SystemZ::CC, TRI);
 468:   }
 469: 
 470:   return true;
 471: }
 472: 
 473: // Try to optimize cases where comparison instruction Compare is testing
 474: // a value against zero.  Return true on success and if Compare should be
 475: // deleted as dead.  CCUsers is the list of instructions that use the CC
 476: // value produced by Compare.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 477-504 / 第 477-504 行
```cpp
 477: bool SystemZElimCompare::optimizeCompareZero(
 478:     MachineInstr &Compare, SmallVectorImpl<MachineInstr *> &CCUsers) {
 479:   if (!TII->isCompareZero(Compare))
 480:     return false;
 481: 
 482:   // Search back for CC results that are based on the first operand.
 483:   unsigned SrcReg = TII->getCompareSourceReg(Compare);
 484:   MachineBasicBlock &MBB = *Compare.getParent();
 485:   Reference CCRefs;
 486:   Reference SrcRefs;
 487:   for (MachineBasicBlock::reverse_iterator MBBI =
 488:          std::next(MachineBasicBlock::reverse_iterator(&Compare)),
 489:          MBBE = MBB.rend(); MBBI != MBBE;) {
 490:     MachineInstr &MI = *MBBI++;
 491:     if (resultTests(MI, SrcReg)) {
 492:       // Try to remove both MI and Compare by converting a branch to BRCT(G).
 493:       // or a load-and-trap instruction.  We don't care in this case whether
 494:       // CC is modified between MI and Compare.
 495:       if (!CCRefs.Use && !SrcRefs) {
 496:         if (convertToBRCT(MI, Compare, CCUsers)) {
 497:           BranchOnCounts += 1;
 498:           return true;
 499:         }
 500:         if (convertToLoadAndTrap(MI, Compare, CCUsers)) {
 501:           LoadAndTraps += 1;
 502:           return true;
 503:         }
 504:       }
```
- **EN**: The range implements or declares functions including `SystemZElimCompare::optimizeCompareZero`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZElimCompare::optimizeCompareZero` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 505-532 / 第 505-532 行
```cpp
 505:       // Try to eliminate Compare by reusing a CC result from MI.
 506:       if ((!CCRefs && convertToLoadAndTest(MI, Compare, CCUsers)) ||
 507:           (!CCRefs.Def &&
 508:            (adjustCCMasksForInstr(MI, Compare, CCUsers) ||
 509:             convertToLogical(MI, Compare, CCUsers)))) {
 510:         EliminatedComparisons += 1;
 511:         return true;
 512:       }
 513:     }
 514:     SrcRefs |= getRegReferences(MI, SrcReg);
 515:     if (SrcRefs.Def)
 516:       break;
 517:     CCRefs |= getRegReferences(MI, SystemZ::CC);
 518:     if (CCRefs.Use && CCRefs.Def)
 519:       break;
 520:     // Eliminating a Compare that may raise an FP exception will move
 521:     // raising the exception to some earlier MI.  We cannot do this if
 522:     // there is anything in between that might change exception flags.
 523:     if (Compare.mayRaiseFPException() &&
 524:         (MI.isCall() || MI.hasUnmodeledSideEffects()))
 525:       break;
 526:   }
 527: 
 528:   // Also do a forward search to handle cases where an instruction after the
 529:   // compare can be converted, like
 530:   // CGHI %r0d, 0; %r1d = LGR %r0d  =>  LTGR %r1d, %r0d
 531:   auto MIRange = llvm::make_range(
 532:       std::next(MachineBasicBlock::iterator(&Compare)), MBB.end());
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 533-560 / 第 533-560 行
```cpp
 533:   for (MachineInstr &MI : llvm::make_early_inc_range(MIRange)) {
 534:     if (preservesValueOf(MI, SrcReg)) {
 535:       // Try to eliminate Compare by reusing a CC result from MI.
 536:       if (convertToLoadAndTest(MI, Compare, CCUsers)) {
 537:         EliminatedComparisons += 1;
 538:         return true;
 539:       }
 540:     }
 541:     if (getRegReferences(MI, SrcReg).Def)
 542:       return false;
 543:     if (getRegReferences(MI, SystemZ::CC))
 544:       return false;
 545:   }
 546: 
 547:   return false;
 548: }
 549: 
 550: // Try to fuse comparison instruction Compare into a later branch.
 551: // Return true on success and if Compare is therefore redundant.
 552: bool SystemZElimCompare::fuseCompareOperations(
 553:     MachineInstr &Compare, SmallVectorImpl<MachineInstr *> &CCUsers) {
 554:   // See whether we have a single branch with which to fuse.
 555:   if (CCUsers.size() != 1)
 556:     return false;
 557:   MachineInstr *Branch = CCUsers[0];
 558:   SystemZII::FusedCompareType Type;
 559:   switch (Branch->getOpcode()) {
 560:   case SystemZ::BRC:
```
- **EN**: The range implements or declares functions including `SystemZElimCompare::fuseCompareOperations`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZElimCompare::fuseCompareOperations` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 561-588 / 第 561-588 行
```cpp
 561:     Type = SystemZII::CompareAndBranch;
 562:     break;
 563:   case SystemZ::CondReturn:
 564:     Type = SystemZII::CompareAndReturn;
 565:     break;
 566:   case SystemZ::CallBCR:
 567:     Type = SystemZII::CompareAndSibcall;
 568:     break;
 569:   case SystemZ::CondTrap:
 570:     Type = SystemZII::CompareAndTrap;
 571:     break;
 572:   default:
 573:     return false;
 574:   }
 575: 
 576:   // See whether we have a comparison that can be fused.
 577:   unsigned FusedOpcode =
 578:       TII->getFusedCompare(Compare.getOpcode(), Type, &Compare);
 579:   if (!FusedOpcode)
 580:     return false;
 581: 
 582:   // Make sure that the operands are available at the branch.
 583:   // SrcReg2 is the register if the source operand is a register,
 584:   // 0 if the source operand is immediate, and the base register
 585:   // if the source operand is memory (index is not supported).
 586:   Register SrcReg = Compare.getOperand(0).getReg();
 587:   Register SrcReg2 =
 588:     Compare.getOperand(1).isReg() ? Compare.getOperand(1).getReg() : Register();
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 589-616 / 第 589-616 行
```cpp
 589:   MachineBasicBlock::iterator MBBI = Compare, MBBE = Branch;
 590:   for (++MBBI; MBBI != MBBE; ++MBBI)
 591:     if (MBBI->modifiesRegister(SrcReg, TRI) ||
 592:         (SrcReg2 && MBBI->modifiesRegister(SrcReg2, TRI)))
 593:       return false;
 594: 
 595:   // Read the branch mask, target (if applicable), regmask (if applicable).
 596:   MachineOperand CCMask(MBBI->getOperand(1));
 597:   assert((CCMask.getImm() & ~SystemZ::CCMASK_ICMP) == 0 &&
 598:          "Invalid condition-code mask for integer comparison");
 599:   // This is only valid for CompareAndBranch and CompareAndSibcall.
 600:   MachineOperand Target(MBBI->getOperand(
 601:     (Type == SystemZII::CompareAndBranch ||
 602:      Type == SystemZII::CompareAndSibcall) ? 2 : 0));
 603:   const uint32_t *RegMask;
 604:   if (Type == SystemZII::CompareAndSibcall)
 605:     RegMask = MBBI->getOperand(3).getRegMask();
 606: 
 607:   // Clear out all current operands.
 608:   int CCUse = MBBI->findRegisterUseOperandIdx(SystemZ::CC, TRI, false);
 609:   assert(CCUse >= 0 && "BRC/BCR must use CC");
 610:   Branch->removeOperand(CCUse);
 611:   // Remove regmask (sibcall).
 612:   if (Type == SystemZII::CompareAndSibcall)
 613:     Branch->removeOperand(3);
 614:   // Remove target (branch or sibcall).
 615:   if (Type == SystemZII::CompareAndBranch ||
 616:       Type == SystemZII::CompareAndSibcall)
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 617-644 / 第 617-644 行
```cpp
 617:     Branch->removeOperand(2);
 618:   Branch->removeOperand(1);
 619:   Branch->removeOperand(0);
 620: 
 621:   // Rebuild Branch as a fused compare and branch.
 622:   // SrcNOps is the number of MI operands of the compare instruction
 623:   // that we need to copy over.
 624:   unsigned SrcNOps = 2;
 625:   if (FusedOpcode == SystemZ::CLT || FusedOpcode == SystemZ::CLGT)
 626:     SrcNOps = 3;
 627:   Branch->setDesc(TII->get(FusedOpcode));
 628:   MachineInstrBuilder MIB(*Branch->getParent()->getParent(), Branch);
 629:   for (unsigned I = 0; I < SrcNOps; I++)
 630:     MIB.add(Compare.getOperand(I));
 631:   MIB.add(CCMask);
 632: 
 633:   if (Type == SystemZII::CompareAndBranch) {
 634:     // Only conditional branches define CC, as they may be converted back
 635:     // to a non-fused branch because of a long displacement.  Conditional
 636:     // returns don't have that problem.
 637:     MIB.add(Target).addReg(SystemZ::CC,
 638:                            RegState::ImplicitDefine | RegState::Dead);
 639:   }
 640: 
 641:   if (Type == SystemZII::CompareAndSibcall) {
 642:     MIB.add(Target);
 643:     MIB.addRegMask(RegMask);
 644:   }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 645-672 / 第 645-672 行
```cpp
 645: 
 646:   // Clear any intervening kills of SrcReg and SrcReg2.
 647:   MBBI = Compare;
 648:   for (++MBBI; MBBI != MBBE; ++MBBI) {
 649:     MBBI->clearRegisterKills(SrcReg, TRI);
 650:     if (SrcReg2)
 651:       MBBI->clearRegisterKills(SrcReg2, TRI);
 652:   }
 653:   FusedComparisons += 1;
 654:   return true;
 655: }
 656: 
 657: // Process all comparison instructions in MBB.  Return true if something
 658: // changed.
 659: bool SystemZElimCompare::processBlock(MachineBasicBlock &MBB) {
 660:   bool Changed = false;
 661: 
 662:   // Walk backwards through the block looking for comparisons, recording
 663:   // all CC users as we go.  The subroutines can delete Compare and
 664:   // instructions before it.
 665:   LiveRegUnits LiveRegs(*TRI);
 666:   LiveRegs.addLiveOuts(MBB);
 667:   bool CompleteCCUsers = LiveRegs.available(SystemZ::CC);
 668:   SmallVector<MachineInstr *, 4> CCUsers;
 669:   MachineBasicBlock::iterator MBBI = MBB.end();
 670:   while (MBBI != MBB.begin()) {
 671:     MachineInstr &MI = *--MBBI;
 672:     if (CompleteCCUsers && (MI.isCompare() || TII->isLoadAndTestAsCmp(MI)) &&
```
- **EN**: The range implements or declares functions including `SystemZElimCompare::processBlock`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZElimCompare::processBlock` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 673-700 / 第 673-700 行
```cpp
 673:         (optimizeCompareZero(MI, CCUsers) ||
 674:          fuseCompareOperations(MI, CCUsers))) {
 675:       ++MBBI;
 676:       MI.eraseFromParent();
 677:       Changed = true;
 678:       CCUsers.clear();
 679:       continue;
 680:     }
 681: 
 682:     if (MI.definesRegister(SystemZ::CC, /*TRI=*/nullptr)) {
 683:       CCUsers.clear();
 684:       CompleteCCUsers = true;
 685:     }
 686:     if (MI.readsRegister(SystemZ::CC, /*TRI=*/nullptr) && CompleteCCUsers)
 687:       CCUsers.push_back(&MI);
 688:   }
 689:   return Changed;
 690: }
 691: 
 692: bool SystemZElimCompare::runOnMachineFunction(MachineFunction &F) {
 693:   if (skipFunction(F.getFunction()))
 694:     return false;
 695: 
 696:   TII = F.getSubtarget<SystemZSubtarget>().getInstrInfo();
 697:   TRI = &TII->getRegisterInfo();
 698: 
 699:   bool Changed = false;
 700:   for (auto &MBB : F)
```
- **EN**: The range implements or declares functions including `fuseCompareOperations`, `SystemZElimCompare::runOnMachineFunction`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `fuseCompareOperations`, `SystemZElimCompare::runOnMachineFunction` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 701-708 / 第 701-708 行
```cpp
 701:     Changed |= processBlock(MBB);
 702: 
 703:   return Changed;
 704: }
 705: 
 706: FunctionPass *llvm::createSystemZElimComparePass(SystemZTargetMachine &TM) {
 707:   return new SystemZElimCompare();
 708: }
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。

## Dependencies / 依赖关系
- `SystemZ.h`
- `SystemZInstrInfo.h`
- `SystemZTargetMachine.h`
- `llvm/ADT/SmallVector.h`
- `llvm/ADT/Statistic.h`
- `llvm/ADT/StringRef.h`
- `llvm/CodeGen/LiveRegUnits.h`
- `llvm/CodeGen/MachineBasicBlock.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/MachineInstr.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineOperand.h`
- `llvm/CodeGen/TargetRegisterInfo.h`
- `llvm/CodeGen/TargetSubtargetInfo.h`
- `llvm/MC/MCInstrDesc.h`
- `...` (2 more include dependencies omitted for brevity / 其余 2 个 include 依赖已省略)
