# LoongArchMergeBaseOffset.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchMergeBaseOffset.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Provides target-specific implementation details for the LoongArch backend.
- **用途 (CN)**: 提供 LoongArch 后端的目标相关实现细节。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```cpp
   1: //===---- LoongArchMergeBaseOffset.cpp - Optimise address calculations ----===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // Merge the offset of address calculation into the offset field
  10: // of instructions in a global address lowering sequence.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "LoongArch.h"
  15: #include "LoongArchTargetMachine.h"
  16: #include "llvm/CodeGen/MachineFunctionPass.h"
  17: #include "llvm/CodeGen/Passes.h"
  18: #include "llvm/MC/TargetRegistry.h"
  19: #include "llvm/Support/Debug.h"
  20: #include "llvm/Target/TargetOptions.h"
  21: #include <optional>
  22: 
  23: using namespace llvm;
  24: 
  25: #define DEBUG_TYPE "loongarch-merge-base-offset"
  26: #define LoongArch_MERGE_BASE_OFFSET_NAME "LoongArch Merge Base Offset"
  27: 
  28: namespace {
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArch.h`, `LoongArchTargetMachine.h`, `MachineFunctionPass.h`, `Passes.h`, `TargetRegistry.h`, `Debug.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArch.h`, `LoongArchTargetMachine.h`, `MachineFunctionPass.h`, `Passes.h`, `TargetRegistry.h`, `Debug.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 29-56 / 第 29-56 行
```cpp
  29: 
  30: class LoongArchMergeBaseOffsetOpt : public MachineFunctionPass {
  31:   const LoongArchSubtarget *ST = nullptr;
  32:   MachineRegisterInfo *MRI;
  33: 
  34: public:
  35:   static char ID;
  36:   bool runOnMachineFunction(MachineFunction &Fn) override;
  37:   bool detectFoldable(MachineInstr &Hi20, MachineInstr *&Lo12,
  38:                       MachineInstr *&Lo20, MachineInstr *&Hi12,
  39:                       MachineInstr *&Last);
  40:   bool detectFoldable(MachineInstr &Hi20, MachineInstr *&Add,
  41:                       MachineInstr *&Lo12);
  42: 
  43:   bool detectAndFoldOffset(MachineInstr &Hi20, MachineInstr &Lo12,
  44:                            MachineInstr *&Lo20, MachineInstr *&Hi12,
  45:                            MachineInstr *&Last);
  46:   void foldOffset(MachineInstr &Hi20, MachineInstr &Lo12, MachineInstr *&Lo20,
  47:                   MachineInstr *&Hi12, MachineInstr *&Last, MachineInstr &Tail,
  48:                   int64_t Offset);
  49:   bool foldLargeOffset(MachineInstr &Hi20, MachineInstr &Lo12,
  50:                        MachineInstr *&Lo20, MachineInstr *&Hi12,
  51:                        MachineInstr *&Last, MachineInstr &TailAdd,
  52:                        Register GAReg);
  53: 
  54:   bool foldIntoMemoryOps(MachineInstr &Hi20, MachineInstr &Lo12,
  55:                          MachineInstr *&Lo20, MachineInstr *&Hi12,
  56:                          MachineInstr *&Last);
```
- **EN**: This block declares or refines TableGen records such as `LoongArchMergeBaseOffsetOpt`.
- **CN**: 该代码块声明或细化了 `LoongArchMergeBaseOffsetOpt` 等 TableGen 记录。

### Lines 57-84 / 第 57-84 行
```cpp
  57: 
  58:   LoongArchMergeBaseOffsetOpt() : MachineFunctionPass(ID) {}
  59: 
  60:   MachineFunctionProperties getRequiredProperties() const override {
  61:     return MachineFunctionProperties().setIsSSA();
  62:   }
  63: 
  64:   void getAnalysisUsage(AnalysisUsage &AU) const override {
  65:     AU.setPreservesCFG();
  66:     MachineFunctionPass::getAnalysisUsage(AU);
  67:   }
  68: 
  69:   StringRef getPassName() const override {
  70:     return LoongArch_MERGE_BASE_OFFSET_NAME;
  71:   }
  72: };
  73: } // end anonymous namespace
  74: 
  75: char LoongArchMergeBaseOffsetOpt::ID = 0;
  76: INITIALIZE_PASS(LoongArchMergeBaseOffsetOpt, DEBUG_TYPE,
  77:                 LoongArch_MERGE_BASE_OFFSET_NAME, false, false)
  78: 
  79: static inline bool isPCAddLo(unsigned Flags) {
  80:   switch (Flags) {
  81:   case LoongArchII::MO_PCADD_LO:
  82:   case LoongArchII::MO_GOT_PCADD_LO:
  83:   case LoongArchII::MO_IE_PCADD_LO:
  84:   case LoongArchII::MO_LD_PCADD_LO:
```
- **EN**: The range implements or declares functions including `LoongArchMergeBaseOffsetOpt`, `isPCAddLo`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `LoongArchMergeBaseOffsetOpt`, `isPCAddLo` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 85-112 / 第 85-112 行
```cpp
  85:   case LoongArchII::MO_GD_PCADD_LO:
  86:   case LoongArchII::MO_DESC_PCADD_LO:
  87:     return true;
  88:   }
  89:   return false;
  90: }
  91: 
  92: // Detect either of the patterns:
  93: //
  94: // 1. (small/medium):
  95: //   pcaxxu12i vreg1, %pc_hi20(s)
  96: //   addi.d    vreg2, vreg1, %pc_lo12(s)
  97: //
  98: // 2. (large):
  99: //   pcalau12i vreg1, %pc_hi20(s)
 100: //   addi.d    vreg2, $zero, %pc_lo12(s)
 101: //   lu32i.d   vreg3, vreg2, %pc64_lo20(s)
 102: //   lu52i.d   vreg4, vreg3, %pc64_hi12(s)
 103: //   add.d     vreg5, vreg4, vreg1
 104: 
 105: // The pattern is only accepted if:
 106: //    1) For small and medium pattern, the first instruction has only one use,
 107: //       which is the ADDI.
 108: //    2) For large pattern, the first four instructions each have only one use,
 109: //       and the user of the fourth instruction is ADD.
 110: //    3) The address operands have the appropriate type, reflecting the
 111: //       lowering of a global address or constant pool using the pattern.
 112: //    4) The offset value in the Global Address or Constant Pool is 0.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 113-140 / 第 113-140 行
```cpp
 113: bool LoongArchMergeBaseOffsetOpt::detectFoldable(MachineInstr &Hi20,
 114:                                                  MachineInstr *&Lo12,
 115:                                                  MachineInstr *&Lo20,
 116:                                                  MachineInstr *&Hi12,
 117:                                                  MachineInstr *&Last) {
 118:   if (Hi20.getOpcode() != LoongArch::PCALAU12I &&
 119:       Hi20.getOpcode() != LoongArch::PCADDU12I)
 120:     return false;
 121: 
 122:   const MachineOperand &Hi20Op1 = Hi20.getOperand(1);
 123:   if (LoongArchII::getDirectFlags(Hi20Op1) != LoongArchII::MO_PCREL_HI &&
 124:       LoongArchII::getDirectFlags(Hi20Op1) != LoongArchII::MO_PCADD_HI)
 125:     return false;
 126: 
 127:   auto isGlobalOrCPIOrBlockAddress = [](const MachineOperand &Op) {
 128:     return Op.isGlobal() || Op.isCPI() || Op.isBlockAddress();
 129:   };
 130: 
 131:   if (!isGlobalOrCPIOrBlockAddress(Hi20Op1) || Hi20Op1.getOffset() != 0)
 132:     return false;
 133: 
 134:   Register HiDestReg = Hi20.getOperand(0).getReg();
 135:   if (!MRI->hasOneUse(HiDestReg))
 136:     return false;
 137: 
 138:   MachineInstr *UseInst = &*MRI->use_instr_begin(HiDestReg);
 139:   if (UseInst->getOpcode() != LoongArch::ADD_D) {
 140:     Lo12 = UseInst;
```
- **EN**: The range implements or declares functions including `LoongArchMergeBaseOffsetOpt::detectFoldable`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchMergeBaseOffsetOpt::detectFoldable` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 141-168 / 第 141-168 行
```cpp
 141:     if ((ST->is64Bit() && Lo12->getOpcode() != LoongArch::ADDI_D) ||
 142:         (!ST->is64Bit() && Lo12->getOpcode() != LoongArch::ADDI_W))
 143:       return false;
 144:   } else {
 145:     assert(ST->is64Bit());
 146:     Last = UseInst;
 147: 
 148:     Register LastOp1Reg = Last->getOperand(1).getReg();
 149:     if (!LastOp1Reg.isVirtual())
 150:       return false;
 151:     Hi12 = MRI->getVRegDef(LastOp1Reg);
 152:     const MachineOperand &Hi12Op2 = Hi12->getOperand(2);
 153:     if (Hi12Op2.getTargetFlags() != LoongArchII::MO_PCREL64_HI)
 154:       return false;
 155:     if (!isGlobalOrCPIOrBlockAddress(Hi12Op2) || Hi12Op2.getOffset() != 0)
 156:       return false;
 157:     if (!MRI->hasOneUse(Hi12->getOperand(0).getReg()))
 158:       return false;
 159: 
 160:     Lo20 = MRI->getVRegDef(Hi12->getOperand(1).getReg());
 161:     const MachineOperand &Lo20Op2 = Lo20->getOperand(2);
 162:     if (Lo20Op2.getTargetFlags() != LoongArchII::MO_PCREL64_LO)
 163:       return false;
 164:     if (!isGlobalOrCPIOrBlockAddress(Lo20Op2) || Lo20Op2.getOffset() != 0)
 165:       return false;
 166:     if (!MRI->hasOneUse(Lo20->getOperand(0).getReg()))
 167:       return false;
 168: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 169-196 / 第 169-196 行
```cpp
 169:     Lo12 = MRI->getVRegDef(Lo20->getOperand(1).getReg());
 170:     if (!MRI->hasOneUse(Lo12->getOperand(0).getReg()))
 171:       return false;
 172:   }
 173: 
 174:   const MachineOperand &Lo12Op2 = Lo12->getOperand(2);
 175:   assert(Hi20.getOpcode() == LoongArch::PCALAU12I ||
 176:          Hi20.getOpcode() == LoongArch::PCADDU12I);
 177:   if ((LoongArchII::getDirectFlags(Lo12Op2) != LoongArchII::MO_PCREL_LO &&
 178:        !isPCAddLo(LoongArchII::getDirectFlags(Lo12Op2))) ||
 179:       !(isGlobalOrCPIOrBlockAddress(Lo12Op2) || Lo12Op2.isMCSymbol()) ||
 180:       Lo12Op2.getOffset() != 0)
 181:     return false;
 182: 
 183:   if (Hi20Op1.isGlobal()) {
 184:     LLVM_DEBUG(dbgs() << "  Found lowered global address: "
 185:                       << *Hi20Op1.getGlobal() << "\n");
 186:   } else if (Hi20Op1.isBlockAddress()) {
 187:     LLVM_DEBUG(dbgs() << "  Found lowered basic address: "
 188:                       << *Hi20Op1.getBlockAddress() << "\n");
 189:   } else if (Hi20Op1.isCPI()) {
 190:     LLVM_DEBUG(dbgs() << "  Found lowered constant pool: " << Hi20Op1.getIndex()
 191:                       << "\n");
 192:   }
 193: 
 194:   return true;
 195: }
 196: 
```
- **EN**: The range implements or declares functions including `LLVM_DEBUG`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LLVM_DEBUG` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 197-224 / 第 197-224 行
```cpp
 197: // Detect the pattern:
 198: //
 199: // (small/medium):
 200: //   lu12i.w  vreg1, %le_hi20_r(s)
 201: //   add.w/d  vreg2, vreg1, r2, %le_add_r(s)
 202: //   addi.w/d vreg3, vreg2, %le_lo12_r(s)
 203: 
 204: // The pattern is only accepted if:
 205: //    1) The first instruction has only one use, which is the PseudoAddTPRel.
 206: //       The second instruction has only one use, which is the ADDI. The
 207: //       second instruction's last operand is the tp register.
 208: //    2) The address operands have the appropriate type, reflecting the
 209: //       lowering of a thread_local global address using the pattern.
 210: //    3) The offset value in the ThreadLocal Global Address is 0.
 211: bool LoongArchMergeBaseOffsetOpt::detectFoldable(MachineInstr &Hi20,
 212:                                                  MachineInstr *&Add,
 213:                                                  MachineInstr *&Lo12) {
 214:   if (Hi20.getOpcode() != LoongArch::LU12I_W)
 215:     return false;
 216: 
 217:   auto isGlobalOrCPI = [](const MachineOperand &Op) {
 218:     return Op.isGlobal() || Op.isCPI();
 219:   };
 220: 
 221:   const MachineOperand &Hi20Op1 = Hi20.getOperand(1);
 222:   if (LoongArchII::getDirectFlags(Hi20Op1) != LoongArchII::MO_LE_HI_R ||
 223:       !isGlobalOrCPI(Hi20Op1) || Hi20Op1.getOffset() != 0)
 224:     return false;
```
- **EN**: The range implements or declares functions including `LoongArchMergeBaseOffsetOpt::detectFoldable`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchMergeBaseOffsetOpt::detectFoldable` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 225-252 / 第 225-252 行
```cpp
 225: 
 226:   Register HiDestReg = Hi20.getOperand(0).getReg();
 227:   if (!MRI->hasOneUse(HiDestReg))
 228:     return false;
 229: 
 230:   Add = &*MRI->use_instr_begin(HiDestReg);
 231:   if ((ST->is64Bit() && Add->getOpcode() != LoongArch::PseudoAddTPRel_D) ||
 232:       (!ST->is64Bit() && Add->getOpcode() != LoongArch::PseudoAddTPRel_W))
 233:     return false;
 234: 
 235:   if (Add->getOperand(2).getReg() != LoongArch::R2)
 236:     return false;
 237: 
 238:   const MachineOperand &AddOp3 = Add->getOperand(3);
 239:   if (LoongArchII::getDirectFlags(AddOp3) != LoongArchII::MO_LE_ADD_R ||
 240:       !(isGlobalOrCPI(AddOp3) || AddOp3.isMCSymbol()) ||
 241:       AddOp3.getOffset() != 0)
 242:     return false;
 243: 
 244:   Register AddDestReg = Add->getOperand(0).getReg();
 245:   if (!MRI->hasOneUse(AddDestReg))
 246:     return false;
 247: 
 248:   Lo12 = &*MRI->use_instr_begin(AddDestReg);
 249:   if ((ST->is64Bit() && Lo12->getOpcode() != LoongArch::ADDI_D) ||
 250:       (!ST->is64Bit() && Lo12->getOpcode() != LoongArch::ADDI_W))
 251:     return false;
 252: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-280 / 第 253-280 行
```cpp
 253:   const MachineOperand &Lo12Op2 = Lo12->getOperand(2);
 254:   if (LoongArchII::getDirectFlags(Lo12Op2) != LoongArchII::MO_LE_LO_R ||
 255:       !(isGlobalOrCPI(Lo12Op2) || Lo12Op2.isMCSymbol()) ||
 256:       Lo12Op2.getOffset() != 0)
 257:     return false;
 258: 
 259:   if (Hi20Op1.isGlobal()) {
 260:     LLVM_DEBUG(dbgs() << "  Found lowered global address: "
 261:                       << *Hi20Op1.getGlobal() << "\n");
 262:   } else if (Hi20Op1.isCPI()) {
 263:     LLVM_DEBUG(dbgs() << "  Found lowered constant pool: " << Hi20Op1.getIndex()
 264:                       << "\n");
 265:   }
 266: 
 267:   return true;
 268: }
 269: 
 270: // Update the offset in Hi20, (Add), Lo12, (Lo20 and Hi12) instructions.
 271: // Delete the tail instruction and update all the uses to use the
 272: // output from Last.
 273: void LoongArchMergeBaseOffsetOpt::foldOffset(
 274:     MachineInstr &Hi20, MachineInstr &Lo12, MachineInstr *&Lo20,
 275:     MachineInstr *&Hi12, MachineInstr *&Last, MachineInstr &Tail,
 276:     int64_t Offset) {
 277:   // Put the offset back in Hi and the Lo
 278:   MachineOperand &Lo12Op2 = Lo12.getOperand(2);
 279:   Hi20.getOperand(1).setOffset(Offset);
 280:   if (!isPCAddLo(LoongArchII::getDirectFlags(Lo12Op2)))
```
- **EN**: The range implements or declares functions including `LLVM_DEBUG`, `LoongArchMergeBaseOffsetOpt::foldOffset`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LLVM_DEBUG`, `LoongArchMergeBaseOffsetOpt::foldOffset` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 281-308 / 第 281-308 行
```cpp
 281:     Lo12Op2.setOffset(Offset);
 282:   if (Lo20 && Hi12) {
 283:     Lo20->getOperand(2).setOffset(Offset);
 284:     Hi12->getOperand(2).setOffset(Offset);
 285:   }
 286: 
 287:   // For tls-le, offset of the second PseudoAddTPRel instr should also be
 288:   // updated.
 289:   MachineInstr *Add = &*MRI->use_instr_begin(Hi20.getOperand(0).getReg());
 290:   if (Hi20.getOpcode() == LoongArch::LU12I_W)
 291:     Add->getOperand(3).setOffset(Offset);
 292: 
 293:   // Delete the tail instruction.
 294:   MachineInstr *Def = Last ? Last : &Lo12;
 295:   MRI->constrainRegClass(Def->getOperand(0).getReg(),
 296:                          MRI->getRegClass(Tail.getOperand(0).getReg()));
 297:   MRI->replaceRegWith(Tail.getOperand(0).getReg(), Def->getOperand(0).getReg());
 298:   Tail.eraseFromParent();
 299: 
 300:   LLVM_DEBUG(dbgs() << "  Merged offset " << Offset << " into base.\n"
 301:                     << "     " << Hi20;);
 302:   if (Hi20.getOpcode() == LoongArch::LU12I_W) {
 303:     LLVM_DEBUG(dbgs() << "     " << *Add;);
 304:   }
 305:   LLVM_DEBUG(dbgs() << "     " << Lo12;);
 306:   if (Lo20 && Hi12) {
 307:     LLVM_DEBUG(dbgs() << "     " << *Lo20 << "     " << *Hi12;);
 308:   }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 309-336 / 第 309-336 行
```cpp
 309: }
 310: 
 311: // Detect patterns for large offsets that are passed into an ADD instruction.
 312: // If the pattern is found, updates the offset in Hi20, (Add), Lo12,
 313: // (Lo20 and Hi12) instructions and deletes TailAdd and the instructions that
 314: // produced the offset.
 315: //
 316: //   (The instructions marked with "!" are not necessarily present)
 317: //
 318: //        Base address lowering is of the form:
 319: //           1) pcala:
 320: //             Hi20:  pcaxxu12i vreg1, %pc_hi20(s)
 321: //        +--- Lo12:  addi.d vreg2, vreg1, %pc_lo12(s)
 322: //        |    Lo20:  lu32i.d vreg2, %pc64_lo20(s) !
 323: //        +--- Hi12:  lu52i.d vreg2, vreg2, %pc64_hi12(s) !
 324: //        |
 325: //        |  2) tls-le:
 326: //        |    Hi20:  lu12i.w vreg1, %le_hi20_r(s)
 327: //        |    Add:   add.w/d vreg1, vreg1, r2, %le_add_r(s)
 328: //        +--- Lo12:  addi.w/d vreg2, vreg1, %le_lo12_r(s)
 329: //        |
 330: //        | The large offset can be one of the forms:
 331: //        |
 332: //        +-> 1) Offset that has non zero bits in Hi20 and Lo12 bits:
 333: //        |     OffsetHi20: lu12i.w vreg3, 4
 334: //        |     OffsetLo12: ori voff, vreg3, 188    ------------------+
 335: //        |                                                           |
 336: //        +-> 2) Offset that has non zero bits in Hi20 bits only:     |
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 337-364 / 第 337-364 行
```cpp
 337: //        |     OffsetHi20: lu12i.w voff, 128       ------------------+
 338: //        |                                                           |
 339: //        +-> 3) Offset that has non zero bits in Lo20 bits:          |
 340: //        |     OffsetHi20: lu12i.w vreg3, 121 !                      |
 341: //        |     OffsetLo12: ori voff, vreg3, 122 !                    |
 342: //        |     OffsetLo20: lu32i.d voff, 123       ------------------+
 343: //        +-> 4) Offset that has non zero bits in Hi12 bits:          |
 344: //              OffsetHi20: lu12i.w vreg3, 121 !                      |
 345: //              OffsetLo12: ori voff, vreg3, 122 !                    |
 346: //              OffsetLo20: lu32i.d vreg3, 123 !                      |
 347: //              OffsetHi12: lu52i.d voff, vrg3, 124 ------------------+
 348: //                                                                    |
 349: //        TailAdd: add.d  vreg4, vreg2, voff       <------------------+
 350: //
 351: bool LoongArchMergeBaseOffsetOpt::foldLargeOffset(
 352:     MachineInstr &Hi20, MachineInstr &Lo12, MachineInstr *&Lo20,
 353:     MachineInstr *&Hi12, MachineInstr *&Last, MachineInstr &TailAdd,
 354:     Register GAReg) {
 355:   assert((TailAdd.getOpcode() == LoongArch::ADD_W ||
 356:           TailAdd.getOpcode() == LoongArch::ADD_D) &&
 357:          "Expected ADD instruction!");
 358:   Register Rs = TailAdd.getOperand(1).getReg();
 359:   Register Rt = TailAdd.getOperand(2).getReg();
 360:   Register Reg = Rs == GAReg ? Rt : Rs;
 361:   SmallVector<MachineInstr *, 4> Instrs;
 362:   int64_t Offset = 0;
 363:   int64_t Mask = -1;
 364: 
```
- **EN**: The range implements or declares functions including `LoongArchMergeBaseOffsetOpt::foldLargeOffset`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchMergeBaseOffsetOpt::foldLargeOffset` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 365-392 / 第 365-392 行
```cpp
 365:   // This can point to one of [ORI, LU12I.W, LU32I.D, LU52I.D]:
 366:   for (int i = 0; i < 4; i++) {
 367:     // Handle Reg is R0.
 368:     if (Reg == LoongArch::R0)
 369:       break;
 370: 
 371:     // Can't fold if the register has more than one use.
 372:     if (!Reg.isVirtual() || !MRI->hasOneUse(Reg))
 373:       return false;
 374: 
 375:     MachineInstr *Curr = MRI->getVRegDef(Reg);
 376:     if (!Curr)
 377:       break;
 378: 
 379:     switch (Curr->getOpcode()) {
 380:     default:
 381:       // Can't fold if the instruction opcode is unexpected.
 382:       return false;
 383:     case LoongArch::ORI: {
 384:       MachineOperand ImmOp = Curr->getOperand(2);
 385:       if (ImmOp.getTargetFlags() != LoongArchII::MO_None)
 386:         return false;
 387:       Offset += ImmOp.getImm();
 388:       Reg = Curr->getOperand(1).getReg();
 389:       Instrs.push_back(Curr);
 390:       break;
 391:     }
 392:     case LoongArch::LU12I_W: {
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 393-420 / 第 393-420 行
```cpp
 393:       MachineOperand ImmOp = Curr->getOperand(1);
 394:       if (ImmOp.getTargetFlags() != LoongArchII::MO_None)
 395:         return false;
 396:       Offset += SignExtend64<32>(ImmOp.getImm() << 12) & Mask;
 397:       Reg = LoongArch::R0;
 398:       Instrs.push_back(Curr);
 399:       break;
 400:     }
 401:     case LoongArch::LU32I_D: {
 402:       MachineOperand ImmOp = Curr->getOperand(2);
 403:       if (ImmOp.getTargetFlags() != LoongArchII::MO_None || !Lo20)
 404:         return false;
 405:       Offset += SignExtend64<52>(ImmOp.getImm() << 32) & Mask;
 406:       Mask ^= 0x000FFFFF00000000ULL;
 407:       Reg = Curr->getOperand(1).getReg();
 408:       Instrs.push_back(Curr);
 409:       break;
 410:     }
 411:     case LoongArch::LU52I_D: {
 412:       MachineOperand ImmOp = Curr->getOperand(2);
 413:       if (ImmOp.getTargetFlags() != LoongArchII::MO_None || !Hi12)
 414:         return false;
 415:       Offset += ImmOp.getImm() << 52;
 416:       Mask ^= 0xFFF0000000000000ULL;
 417:       Reg = Curr->getOperand(1).getReg();
 418:       Instrs.push_back(Curr);
 419:       break;
 420:     }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 421-448 / 第 421-448 行
```cpp
 421:     }
 422:   }
 423: 
 424:   // Can't fold if the offset is not extracted.
 425:   if (!Offset)
 426:     return false;
 427: 
 428:   foldOffset(Hi20, Lo12, Lo20, Hi12, Last, TailAdd, Offset);
 429:   LLVM_DEBUG(dbgs() << "  Offset Instrs:\n");
 430:   for (auto I : Instrs) {
 431:     LLVM_DEBUG(dbgs() << "                 " << *I);
 432:     I->eraseFromParent();
 433:   }
 434: 
 435:   return true;
 436: }
 437: 
 438: bool LoongArchMergeBaseOffsetOpt::detectAndFoldOffset(MachineInstr &Hi20,
 439:                                                       MachineInstr &Lo12,
 440:                                                       MachineInstr *&Lo20,
 441:                                                       MachineInstr *&Hi12,
 442:                                                       MachineInstr *&Last) {
 443:   Register DestReg =
 444:       Last ? Last->getOperand(0).getReg() : Lo12.getOperand(0).getReg();
 445: 
 446:   // Look for arithmetic instructions we can get an offset from.
 447:   // We might be able to remove the arithmetic instructions by folding the
 448:   // offset into the PCAXXU12I+(ADDI/ADDI+LU32I+LU52I) or
```
- **EN**: The range implements or declares functions including `LoongArchMergeBaseOffsetOpt::detectAndFoldOffset`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchMergeBaseOffsetOpt::detectAndFoldOffset` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 449-476 / 第 449-476 行
```cpp
 449:   // LU12I_W+PseudoAddTPRel+ADDI.
 450:   if (!MRI->hasOneUse(DestReg))
 451:     return false;
 452: 
 453:   // DestReg has only one use.
 454:   MachineInstr &Tail = *MRI->use_instr_begin(DestReg);
 455:   switch (Tail.getOpcode()) {
 456:   default:
 457:     LLVM_DEBUG(dbgs() << "Don't know how to get offset from this instr:"
 458:                       << Tail);
 459:     break;
 460:   case LoongArch::ADDI_W:
 461:     if (ST->is64Bit())
 462:       return false;
 463:     [[fallthrough]];
 464:   case LoongArch::ADDI_D:
 465:   case LoongArch::ADDU16I_D: {
 466:     // Offset is simply an immediate operand.
 467:     int64_t Offset = Tail.getOperand(2).getImm();
 468:     if (Tail.getOpcode() == LoongArch::ADDU16I_D)
 469:       Offset = SignExtend64<32>(Offset << 16);
 470: 
 471:     // We might have two ADDIs in a row.
 472:     Register TailDestReg = Tail.getOperand(0).getReg();
 473:     if (MRI->hasOneUse(TailDestReg)) {
 474:       MachineInstr &TailTail = *MRI->use_instr_begin(TailDestReg);
 475:       if (ST->is64Bit() && TailTail.getOpcode() == LoongArch::ADDI_W)
 476:         return false;
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 477-504 / 第 477-504 行
```cpp
 477:       if (TailTail.getOpcode() == LoongArch::ADDI_W ||
 478:           TailTail.getOpcode() == LoongArch::ADDI_D) {
 479:         Offset += TailTail.getOperand(2).getImm();
 480:         LLVM_DEBUG(dbgs() << "  Offset Instrs: " << Tail << TailTail);
 481:         foldOffset(Hi20, Lo12, Lo20, Hi12, Last, TailTail, Offset);
 482:         Tail.eraseFromParent();
 483:         return true;
 484:       }
 485:     }
 486: 
 487:     LLVM_DEBUG(dbgs() << "  Offset Instr: " << Tail);
 488:     foldOffset(Hi20, Lo12, Lo20, Hi12, Last, Tail, Offset);
 489:     return true;
 490:   }
 491:   case LoongArch::ADD_W:
 492:     if (ST->is64Bit())
 493:       return false;
 494:     [[fallthrough]];
 495:   case LoongArch::ADD_D:
 496:     // The offset is too large to fit in the immediate field of ADDI.
 497:     return foldLargeOffset(Hi20, Lo12, Lo20, Hi12, Last, Tail, DestReg);
 498:     break;
 499:   }
 500: 
 501:   return false;
 502: }
 503: 
 504: // Memory access opcode mapping for transforms.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 505-532 / 第 505-532 行
```cpp
 505: static unsigned getNewOpc(unsigned Op, bool isLarge) {
 506:   switch (Op) {
 507:   case LoongArch::LD_B:
 508:     return isLarge ? LoongArch::LDX_B : LoongArch::LD_B;
 509:   case LoongArch::LD_H:
 510:     return isLarge ? LoongArch::LDX_H : LoongArch::LD_H;
 511:   case LoongArch::LD_W:
 512:   case LoongArch::LDPTR_W:
 513:     return isLarge ? LoongArch::LDX_W : LoongArch::LD_W;
 514:   case LoongArch::LD_D:
 515:   case LoongArch::LDPTR_D:
 516:     return isLarge ? LoongArch::LDX_D : LoongArch::LD_D;
 517:   case LoongArch::LD_BU:
 518:     return isLarge ? LoongArch::LDX_BU : LoongArch::LD_BU;
 519:   case LoongArch::LD_HU:
 520:     return isLarge ? LoongArch::LDX_HU : LoongArch::LD_HU;
 521:   case LoongArch::LD_WU:
 522:     return isLarge ? LoongArch::LDX_WU : LoongArch::LD_WU;
 523:   case LoongArch::FLD_S:
 524:     return isLarge ? LoongArch::FLDX_S : LoongArch::FLD_S;
 525:   case LoongArch::FLD_D:
 526:     return isLarge ? LoongArch::FLDX_D : LoongArch::FLD_D;
 527:   case LoongArch::VLD:
 528:     return isLarge ? LoongArch::VLDX : LoongArch::VLD;
 529:   case LoongArch::XVLD:
 530:     return isLarge ? LoongArch::XVLDX : LoongArch::XVLD;
 531:   case LoongArch::VLDREPL_B:
 532:     return LoongArch::VLDREPL_B;
```
- **EN**: The range implements or declares functions including `getNewOpc`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `getNewOpc` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 533-560 / 第 533-560 行
```cpp
 533:   case LoongArch::XVLDREPL_B:
 534:     return LoongArch::XVLDREPL_B;
 535:   case LoongArch::ST_B:
 536:     return isLarge ? LoongArch::STX_B : LoongArch::ST_B;
 537:   case LoongArch::ST_H:
 538:     return isLarge ? LoongArch::STX_H : LoongArch::ST_H;
 539:   case LoongArch::ST_W:
 540:   case LoongArch::STPTR_W:
 541:     return isLarge ? LoongArch::STX_W : LoongArch::ST_W;
 542:   case LoongArch::ST_D:
 543:   case LoongArch::STPTR_D:
 544:     return isLarge ? LoongArch::STX_D : LoongArch::ST_D;
 545:   case LoongArch::FST_S:
 546:     return isLarge ? LoongArch::FSTX_S : LoongArch::FST_S;
 547:   case LoongArch::FST_D:
 548:     return isLarge ? LoongArch::FSTX_D : LoongArch::FST_D;
 549:   case LoongArch::VST:
 550:     return isLarge ? LoongArch::VSTX : LoongArch::VST;
 551:   case LoongArch::XVST:
 552:     return isLarge ? LoongArch::XVSTX : LoongArch::XVST;
 553:   default:
 554:     llvm_unreachable("Unexpected opcode for replacement");
 555:   }
 556: }
 557: 
 558: bool LoongArchMergeBaseOffsetOpt::foldIntoMemoryOps(MachineInstr &Hi20,
 559:                                                     MachineInstr &Lo12,
 560:                                                     MachineInstr *&Lo20,
```
- **EN**: The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 561-588 / 第 561-588 行
```cpp
 561:                                                     MachineInstr *&Hi12,
 562:                                                     MachineInstr *&Last) {
 563:   Register DestReg =
 564:       Last ? Last->getOperand(0).getReg() : Lo12.getOperand(0).getReg();
 565: 
 566:   // If all the uses are memory ops with the same offset, we can transform:
 567:   //
 568:   // 1. (small/medium):
 569:   //  1.1. pcala
 570:   //   pcaxxu12i vreg1, %pc_hi20(s)
 571:   //   addi.d    vreg2, vreg1, %pc_lo12(s)
 572:   //   ld.w      vreg3, 8(vreg2)
 573:   //
 574:   //   =>
 575:   //
 576:   //   pcalxx12i vreg1, %pc_hi20(s+8)
 577:   //   ld.w      vreg3, vreg1, %pc_lo12(s+8)(vreg1)
 578:   //
 579:   //  1.2. tls-le
 580:   //   lu12i.w  vreg1, %le_hi20_r(s)
 581:   //   add.w/d  vreg2, vreg1, r2, %le_add_r(s)
 582:   //   addi.w/d vreg3, vreg2, %le_lo12_r(s)
 583:   //   ld.w     vreg4, 8(vreg3)
 584:   //
 585:   //   =>
 586:   //
 587:   //   lu12i.w vreg1, %le_hi20_r(s+8)
 588:   //   add.w/d vreg2, vreg1, r2, %le_add_r(s+8)
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 589-616 / 第 589-616 行
```cpp
 589:   //   ld.w    vreg4, vreg2, %le_lo12_r(s+8)(vreg2)
 590:   //
 591:   // 2. (large):
 592:   //   pcalau12i vreg1, %pc_hi20(s)
 593:   //   addi.d    vreg2, $zero, %pc_lo12(s)
 594:   //   lu32i.d   vreg3, vreg2, %pc64_lo20(s)
 595:   //   lu52i.d   vreg4, vreg3, %pc64_hi12(s)
 596:   //   add.d     vreg5, vreg4, vreg1
 597:   //   ld.w      vreg6, 8(vreg5)
 598:   //
 599:   //   =>
 600:   //
 601:   //   pcalau12i vreg1, %pc_hi20(s+8)
 602:   //   addi.d    vreg2, $zero, %pc_lo12(s+8)
 603:   //   lu32i.d   vreg3, vreg2, %pc64_lo20(s+8)
 604:   //   lu52i.d   vreg4, vreg3, %pc64_hi12(s+8)
 605:   //   ldx.w     vreg6, vreg4, vreg1
 606: 
 607:   std::optional<int64_t> CommonOffset;
 608:   DenseMap<const MachineInstr *, SmallVector<unsigned>>
 609:       InlineAsmMemoryOpIndexesMap;
 610:   for (const MachineInstr &UseMI : MRI->use_instructions(DestReg)) {
 611:     switch (UseMI.getOpcode()) {
 612:     default:
 613:       LLVM_DEBUG(dbgs() << "Not a load or store instruction: " << UseMI);
 614:       return false;
 615:     case LoongArch::VLDREPL_B:
 616:     case LoongArch::XVLDREPL_B:
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 617-644 / 第 617-644 行
```cpp
 617:       // We can't do this for large pattern.
 618:       if (Last)
 619:         return false;
 620:       [[fallthrough]];
 621:     case LoongArch::LD_B:
 622:     case LoongArch::LD_H:
 623:     case LoongArch::LD_W:
 624:     case LoongArch::LD_D:
 625:     case LoongArch::LD_BU:
 626:     case LoongArch::LD_HU:
 627:     case LoongArch::LD_WU:
 628:     case LoongArch::LDPTR_W:
 629:     case LoongArch::LDPTR_D:
 630:     case LoongArch::FLD_S:
 631:     case LoongArch::FLD_D:
 632:     case LoongArch::VLD:
 633:     case LoongArch::XVLD:
 634:     case LoongArch::ST_B:
 635:     case LoongArch::ST_H:
 636:     case LoongArch::ST_W:
 637:     case LoongArch::ST_D:
 638:     case LoongArch::STPTR_W:
 639:     case LoongArch::STPTR_D:
 640:     case LoongArch::FST_S:
 641:     case LoongArch::FST_D:
 642:     case LoongArch::VST:
 643:     case LoongArch::XVST: {
 644:       if (UseMI.getOperand(1).isFI())
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 645-672 / 第 645-672 行
```cpp
 645:         return false;
 646:       // Register defined by Lo should not be the value register.
 647:       if (DestReg == UseMI.getOperand(0).getReg())
 648:         return false;
 649:       assert(DestReg == UseMI.getOperand(1).getReg() &&
 650:              "Expected base address use");
 651:       // All load/store instructions must use the same offset.
 652:       int64_t Offset = UseMI.getOperand(2).getImm();
 653:       if (CommonOffset && Offset != CommonOffset)
 654:         return false;
 655:       CommonOffset = Offset;
 656:       break;
 657:     }
 658:     case LoongArch::INLINEASM:
 659:     case LoongArch::INLINEASM_BR: {
 660:       // We can't do this for large pattern.
 661:       if (Last)
 662:         return false;
 663:       SmallVector<unsigned> InlineAsmMemoryOpIndexes;
 664:       unsigned NumOps = 0;
 665:       for (unsigned I = InlineAsm::MIOp_FirstOperand;
 666:            I < UseMI.getNumOperands(); I += 1 + NumOps) {
 667:         const MachineOperand &FlagsMO = UseMI.getOperand(I);
 668:         // Should be an imm.
 669:         if (!FlagsMO.isImm())
 670:           continue;
 671: 
 672:         const InlineAsm::Flag Flags(FlagsMO.getImm());
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 673-700 / 第 673-700 行
```cpp
 673:         NumOps = Flags.getNumOperandRegisters();
 674: 
 675:         // Memory constraints have two operands.
 676:         if (NumOps != 2 || !Flags.isMemKind()) {
 677:           // If the register is used by something other than a memory contraint,
 678:           // we should not fold.
 679:           for (unsigned J = 0; J < NumOps; ++J) {
 680:             const MachineOperand &MO = UseMI.getOperand(I + 1 + J);
 681:             if (MO.isReg() && MO.getReg() == DestReg)
 682:               return false;
 683:           }
 684:           continue;
 685:         }
 686: 
 687:         // We can only do this for constraint m.
 688:         if (Flags.getMemoryConstraintID() != InlineAsm::ConstraintCode::m)
 689:           return false;
 690: 
 691:         const MachineOperand &AddrMO = UseMI.getOperand(I + 1);
 692:         if (!AddrMO.isReg() || AddrMO.getReg() != DestReg)
 693:           continue;
 694: 
 695:         const MachineOperand &OffsetMO = UseMI.getOperand(I + 2);
 696:         if (!OffsetMO.isImm())
 697:           continue;
 698: 
 699:         // All inline asm memory operands must use the same offset.
 700:         int64_t Offset = OffsetMO.getImm();
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 701-728 / 第 701-728 行
```cpp
 701:         if (CommonOffset && Offset != CommonOffset)
 702:           return false;
 703:         CommonOffset = Offset;
 704:         InlineAsmMemoryOpIndexes.push_back(I + 1);
 705:       }
 706:       InlineAsmMemoryOpIndexesMap.insert(
 707:           std::make_pair(&UseMI, InlineAsmMemoryOpIndexes));
 708:       break;
 709:     }
 710:     }
 711:   }
 712: 
 713:   // We found a common offset.
 714:   // Update the offsets in global address lowering.
 715:   // We may have already folded some arithmetic so we need to add to any
 716:   // existing offset.
 717:   int64_t NewOffset = Hi20.getOperand(1).getOffset() + *CommonOffset;
 718:   // LA32 ignores the upper 32 bits.
 719:   if (!ST->is64Bit())
 720:     NewOffset = SignExtend64<32>(NewOffset);
 721:   // We can only fold simm32 offsets.
 722:   if (!isInt<32>(NewOffset))
 723:     return false;
 724: 
 725:   // If optimized by this pass successfully, MO_RELAX bitmask target-flag should
 726:   // be removed from the pcala code sequence. Code sequence of tls-le can still
 727:   // be relaxed after being optimized.
 728:   //
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 729-756 / 第 729-756 行
```cpp
 729:   // For example:
 730:   //   pcaxxu12i $a0, %pc_hi20(symbol)
 731:   //   addi.d $a0, $a0, %pc_lo12(symbol)
 732:   //   ld.w $a0, $a0, 0
 733:   //
 734:   //   =>
 735:   //
 736:   //   pcaxxu12i $a0, %pc_hi20(symbol)
 737:   //   ld.w $a0, $a0, %pc_lo12(symbol)
 738:   //
 739:   // Code sequence optimized before can be relax by linker. But after being
 740:   // optimized, it cannot be relaxed any more. So MO_RELAX flag should not be
 741:   // carried by them.
 742:   Hi20.getOperand(1).setOffset(NewOffset);
 743:   MachineOperand &ImmOp = Lo12.getOperand(2);
 744:   if (!isPCAddLo(LoongArchII::getDirectFlags(ImmOp)))
 745:     ImmOp.setOffset(NewOffset);
 746:   if (Lo20 && Hi12) {
 747:     Lo20->getOperand(2).setOffset(NewOffset);
 748:     Hi12->getOperand(2).setOffset(NewOffset);
 749:   }
 750:   if (Hi20.getOpcode() == LoongArch::PCADDU12I ||
 751:       Hi20.getOpcode() == LoongArch::PCALAU12I) {
 752:     Hi20.getOperand(1).setTargetFlags(
 753:         LoongArchII::getDirectFlags(Hi20.getOperand(1)));
 754:     ImmOp.setTargetFlags(LoongArchII::getDirectFlags(ImmOp));
 755:   } else if (Hi20.getOpcode() == LoongArch::LU12I_W) {
 756:     MachineInstr *Add = &*MRI->use_instr_begin(Hi20.getOperand(0).getReg());
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 757-784 / 第 757-784 行
```cpp
 757:     Add->getOperand(3).setOffset(NewOffset);
 758:   }
 759: 
 760:   // Update the immediate in the load/store instructions to add the offset.
 761:   const LoongArchInstrInfo &TII = *ST->getInstrInfo();
 762:   for (MachineInstr &UseMI :
 763:        llvm::make_early_inc_range(MRI->use_instructions(DestReg))) {
 764:     if (UseMI.getOpcode() == LoongArch::INLINEASM ||
 765:         UseMI.getOpcode() == LoongArch::INLINEASM_BR) {
 766:       auto &InlineAsmMemoryOpIndexes = InlineAsmMemoryOpIndexesMap[&UseMI];
 767:       for (unsigned I : InlineAsmMemoryOpIndexes) {
 768:         MachineOperand &MO = UseMI.getOperand(I + 1);
 769:         switch (ImmOp.getType()) {
 770:         case MachineOperand::MO_GlobalAddress:
 771:           MO.ChangeToGA(ImmOp.getGlobal(), ImmOp.getOffset(),
 772:                         LoongArchII::getDirectFlags(ImmOp));
 773:           break;
 774:         case MachineOperand::MO_MCSymbol:
 775:           MO.ChangeToMCSymbol(ImmOp.getMCSymbol(),
 776:                               LoongArchII::getDirectFlags(ImmOp));
 777:           MO.setOffset(ImmOp.getOffset());
 778:           break;
 779:         case MachineOperand::MO_BlockAddress:
 780:           MO.ChangeToBA(ImmOp.getBlockAddress(), ImmOp.getOffset(),
 781:                         LoongArchII::getDirectFlags(ImmOp));
 782:           break;
 783:         case MachineOperand::MO_ConstantPoolIndex:
 784:           MO.ChangeToCPI(ImmOp.getIndex(), ImmOp.getOffset(),
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 785-812 / 第 785-812 行
```cpp
 785:                          LoongArchII::getDirectFlags(ImmOp));
 786:           break;
 787:         default:
 788:           report_fatal_error("unsupported machine operand type");
 789:           break;
 790:         }
 791:       }
 792:     } else {
 793:       UseMI.setDesc(TII.get(getNewOpc(UseMI.getOpcode(), Last)));
 794:       if (Last) {
 795:         UseMI.removeOperand(2);
 796:         UseMI.removeOperand(1);
 797:         UseMI.addOperand(Last->getOperand(1));
 798:         UseMI.addOperand(Last->getOperand(2));
 799:         UseMI.getOperand(1).setIsKill(false);
 800:         UseMI.getOperand(2).setIsKill(false);
 801:       } else {
 802:         UseMI.removeOperand(2);
 803:         UseMI.addOperand(ImmOp);
 804:       }
 805:     }
 806:   }
 807: 
 808:   if (Last) {
 809:     Last->eraseFromParent();
 810:     return true;
 811:   }
 812: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 813-840 / 第 813-840 行
```cpp
 813:   if (Hi20.getOpcode() == LoongArch::PCADDU12I ||
 814:       Hi20.getOpcode() == LoongArch::PCALAU12I) {
 815:     MRI->replaceRegWith(Lo12.getOperand(0).getReg(),
 816:                         Hi20.getOperand(0).getReg());
 817:   } else if (Hi20.getOpcode() == LoongArch::LU12I_W) {
 818:     MachineInstr *Add = &*MRI->use_instr_begin(Hi20.getOperand(0).getReg());
 819:     MRI->replaceRegWith(Lo12.getOperand(0).getReg(),
 820:                         Add->getOperand(0).getReg());
 821:   }
 822:   Lo12.eraseFromParent();
 823:   return true;
 824: }
 825: 
 826: bool LoongArchMergeBaseOffsetOpt::runOnMachineFunction(MachineFunction &Fn) {
 827:   if (skipFunction(Fn.getFunction()))
 828:     return false;
 829: 
 830:   ST = &Fn.getSubtarget<LoongArchSubtarget>();
 831: 
 832:   bool MadeChange = false;
 833:   MRI = &Fn.getRegInfo();
 834:   for (MachineBasicBlock &MBB : Fn) {
 835:     LLVM_DEBUG(dbgs() << "MBB: " << MBB.getName() << "\n");
 836:     for (MachineInstr &Hi20 : MBB) {
 837:       MachineInstr *Lo12 = nullptr;
 838:       MachineInstr *Lo20 = nullptr;
 839:       MachineInstr *Hi12 = nullptr;
 840:       MachineInstr *Last = nullptr;
```
- **EN**: The range implements or declares functions including `LoongArchMergeBaseOffsetOpt::runOnMachineFunction`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchMergeBaseOffsetOpt::runOnMachineFunction` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 841-868 / 第 841-868 行
```cpp
 841:       if (Hi20.getOpcode() == LoongArch::PCADDU12I ||
 842:           Hi20.getOpcode() == LoongArch::PCALAU12I) {
 843:         // Detect foldable pcala code sequence in small/medium/large code model.
 844:         if (!detectFoldable(Hi20, Lo12, Lo20, Hi12, Last))
 845:           continue;
 846:       } else if (Hi20.getOpcode() == LoongArch::LU12I_W) {
 847:         MachineInstr *Add = nullptr;
 848:         // Detect foldable tls-le code sequence in small/medium code model.
 849:         if (!detectFoldable(Hi20, Add, Lo12))
 850:           continue;
 851:       } else {
 852:         continue;
 853:       }
 854:       // For tls-le, we do not pass the second PseudoAddTPRel instr in order to
 855:       // reuse the existing hooks and the last three paramaters should always be
 856:       // nullptr.
 857:       MadeChange |= detectAndFoldOffset(Hi20, *Lo12, Lo20, Hi12, Last);
 858:       MadeChange |= foldIntoMemoryOps(Hi20, *Lo12, Lo20, Hi12, Last);
 859:     }
 860:   }
 861: 
 862:   return MadeChange;
 863: }
 864: 
 865: /// Returns an instance of the Merge Base Offset Optimization pass.
 866: FunctionPass *llvm::createLoongArchMergeBaseOffsetOptPass() {
 867:   return new LoongArchMergeBaseOffsetOpt();
 868: }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `LoongArch.h`
- `LoongArchTargetMachine.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/Passes.h`
- `llvm/MC/TargetRegistry.h`
- `llvm/Support/Debug.h`
- `llvm/Target/TargetOptions.h`
- `optional`
