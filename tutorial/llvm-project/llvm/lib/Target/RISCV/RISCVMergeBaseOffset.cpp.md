# RISCVMergeBaseOffset.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVMergeBaseOffset.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements combining base and offset computations in RISC-V addressing patterns. / 实现合并 RISC-V 寻址模式中的基址与偏移计算。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: File prologue and imported dependencies / 文件前言与导入依赖
```cpp
//===----- RISCVMergeBaseOffset.cpp - Optimise address calculations  ------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Merge the offset of address calculation into the offset field
// of instructions in a global address lowering sequence.
//
//===----------------------------------------------------------------------===//

#include "RISCV.h"
#include "RISCVTargetMachine.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Debug.h"
#include "llvm/Target/TargetOptions.h"
#include <optional>
using namespace llvm;
```
**EN:** This opening block combines banner comments with the first wave of includes, giving readers both context and the main compile-time dependencies.

**CN:** 该开头区段同时包含说明性注释与首批 include，让读者先看到文件定位，再看到主要编译期依赖。

### Lines 24-44: Header guard and interface framing / 头文件保护与接口框架
```cpp
#define DEBUG_TYPE "riscv-merge-base-offset"
#define RISCV_MERGE_BASE_OFFSET_NAME "RISC-V Merge Base Offset"
namespace {

class RISCVMergeBaseOffsetOpt : public MachineFunctionPass {
  const RISCVSubtarget *ST = nullptr;
  MachineRegisterInfo *MRI;

public:
  static char ID;
  bool runOnMachineFunction(MachineFunction &Fn) override;
  bool detectFoldable(MachineInstr &Hi, MachineInstr *&Lo);

  bool detectAndFoldOffset(MachineInstr &Hi, MachineInstr &Lo);
  bool foldOffset(MachineInstr &Hi, MachineInstr &Lo, MachineInstr &Tail,
                  int64_t Offset);
  bool foldLargeOffset(MachineInstr &Hi, MachineInstr &Lo,
                       MachineInstr &TailAdd, Register GSReg);
  bool foldShiftedOffset(MachineInstr &Hi, MachineInstr &Lo,
                         MachineInstr &TailShXAdd, Register GSReg);
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 45-58: Function implementation: MachineFunctionPass::getAnalysisUsage / 函数实现：MachineFunctionPass::getAnalysisUsage
```cpp
  bool foldIntoMemoryOps(MachineInstr &Hi, MachineInstr &Lo);
  bool foldShxaddIntoScaledMemory(MachineInstr &Hi, MachineInstr &Lo);

  RISCVMergeBaseOffsetOpt() : MachineFunctionPass(ID) {}

  MachineFunctionProperties getRequiredProperties() const override {
    return MachineFunctionProperties().setIsSSA();
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 59-86: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
  StringRef getPassName() const override {
    return RISCV_MERGE_BASE_OFFSET_NAME;
  }
};
} // end anonymous namespace

char RISCVMergeBaseOffsetOpt::ID = 0;
INITIALIZE_PASS(RISCVMergeBaseOffsetOpt, DEBUG_TYPE,
                RISCV_MERGE_BASE_OFFSET_NAME, false, false)

// Detect either of the patterns:
//
// 1. (medlow pattern):
//   a. lui   vreg1, %hi(s)
//      addi  vreg2, vreg1, %lo(s)
//
//   b.  qc.e.li vreg1, s
//
// 2. (medany pattern):
// .Lpcrel_hi1:
//   auipc vreg1, %pcrel_hi(s)
//   addi  vreg2, vreg1, %pcrel_lo(.Lpcrel_hi1)
//
// The pattern is only accepted if:
//    1) The first instruction has only one use, which is the ADDI.
//    2) The address operands have the appropriate type, reflecting the
//       lowering of a global address or constant pool using medlow or medany.
//    3) The offset value in the Global Address or Constant Pool is 0.
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 87-100: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
bool RISCVMergeBaseOffsetOpt::detectFoldable(MachineInstr &Hi,
                                             MachineInstr *&Lo) {
  auto HiOpc = Hi.getOpcode();
  if (HiOpc != RISCV::LUI && HiOpc != RISCV::AUIPC &&
      HiOpc != RISCV::PseudoMovAddr && HiOpc != RISCV::QC_E_LI)
    return false;

  const MachineOperand &HiOp1 = Hi.getOperand(1);
  unsigned ExpectedFlags = HiOpc == RISCV::AUIPC     ? RISCVII::MO_PCREL_HI
                           : HiOpc == RISCV::QC_E_LI ? RISCVII::MO_None
                                                     : RISCVII::MO_HI;
  if (HiOp1.getTargetFlags() != ExpectedFlags)
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 101-117: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (!(HiOp1.isGlobal() || HiOp1.isCPI() || HiOp1.isBlockAddress()) ||
      HiOp1.getOffset() != 0)
    return false;

  if (HiOpc == RISCV::PseudoMovAddr || HiOpc == RISCV::QC_E_LI) {
    // Most of the code should handle it correctly without modification by
    // setting Lo and Hi both point to PseudoMovAddr/QC_E_LI
    Lo = &Hi;
  } else {
    Register HiDestReg = Hi.getOperand(0).getReg();
    if (!MRI->hasOneUse(HiDestReg))
      return false;

    Lo = &*MRI->use_instr_begin(HiDestReg);
    if (Lo->getOpcode() != RISCV::ADDI)
      return false;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 118-131: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  if (HiOpc != RISCV::QC_E_LI) {
    const MachineOperand &LoOp2 = Lo->getOperand(2);
    if (HiOpc == RISCV::LUI || HiOpc == RISCV::PseudoMovAddr) {
      if (LoOp2.getTargetFlags() != RISCVII::MO_LO ||
          !(LoOp2.isGlobal() || LoOp2.isCPI() || LoOp2.isBlockAddress()) ||
          LoOp2.getOffset() != 0)
        return false;
    } else {
      assert(HiOpc == RISCV::AUIPC);
      if (LoOp2.getTargetFlags() != RISCVII::MO_PCREL_LO ||
          LoOp2.getType() != MachineOperand::MO_MCSymbol)
        return false;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 132-146: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  }

  if (HiOp1.isGlobal()) {
    LLVM_DEBUG(dbgs() << "  Found lowered global address: "
                      << *HiOp1.getGlobal() << "\n");
  } else if (HiOp1.isBlockAddress()) {
    LLVM_DEBUG(dbgs() << "  Found lowered basic address: "
                      << *HiOp1.getBlockAddress() << "\n");
  } else if (HiOp1.isCPI()) {
    LLVM_DEBUG(dbgs() << "  Found lowered constant pool: " << HiOp1.getIndex()
                      << "\n");
  }

  return true;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 147-165: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

// Update the offset in Hi and Lo instructions.
// Delete the tail instruction and update all the uses to use the
// output from Lo.
bool RISCVMergeBaseOffsetOpt::foldOffset(MachineInstr &Hi, MachineInstr &Lo,
                                         MachineInstr &Tail, int64_t Offset) {
  assert(isInt<32>(Offset) && "Unexpected offset");

  // If Hi is an AUIPC, don't fold the offset if it is outside the bounds of
  // the global object. The object may be within 2GB of the PC, but addresses
  // outside of the object might not be.
  auto HiOpc = Hi.getOpcode();
  if (HiOpc == RISCV::AUIPC && Hi.getOperand(1).isGlobal()) {
    const GlobalValue *GV = Hi.getOperand(1).getGlobal();
    Type *Ty = GV->getValueType();
    if (!Ty->isSized() || Offset < 0 ||
        (uint64_t)Offset > GV->getDataLayout().getTypeAllocSize(Ty))
      return false;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 166-180: Definitions and supporting logic / 定义与支撑逻辑
```cpp

  // Put the offset back in Hi and the Lo
  Hi.getOperand(1).setOffset(Offset);
  if (Hi.getOpcode() != RISCV::AUIPC && Hi.getOpcode() != RISCV::QC_E_LI)
    Lo.getOperand(2).setOffset(Offset);
  // Delete the tail instruction.
  Register LoOp0Reg = Lo.getOperand(0).getReg();
  Register TailOp0Reg = Tail.getOperand(0).getReg();
  MRI->constrainRegClass(LoOp0Reg, MRI->getRegClass(TailOp0Reg));
  MRI->replaceRegWith(TailOp0Reg, LoOp0Reg);
  Tail.eraseFromParent();
  LLVM_DEBUG(dbgs() << "  Merged offset " << Offset << " into base.\n"
                    << "     " << Hi << "     " << Lo;);
  return true;
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 181-208: Commentary and design intent / 注释与设计意图
```cpp

// Detect patterns for large offsets that are passed into an ADD instruction.
// If the pattern is found, updates the offset in Hi and Lo instructions
// and deletes TailAdd and the instructions that produced the offset.
//
//                     Base address lowering is of the form:
//                       Hi:  lui   vreg1, %hi(s)
//                       Lo:  addi  vreg2, vreg1, %lo(s)
//                       /                                  \
//                      /                                    \
//                     /                                      \
//                    /  The large offset can be of two forms: \
//  1) Offset that has non zero bits in lower      2) Offset that has non zero
//     12 bits and upper 20 bits                      bits in upper 20 bits only
//   OffseLUI: lui   vreg3, 4
// OffsetTail: addi  voff, vreg3, 188                OffsetTail: lui  voff, 128
//                    \                                        /
//                     \                                      /
//                      \                                    /
//                       \                                  /
//                         TailAdd: add  vreg4, vreg2, voff
bool RISCVMergeBaseOffsetOpt::foldLargeOffset(MachineInstr &Hi,
                                              MachineInstr &Lo,
                                              MachineInstr &TailAdd,
                                              Register GAReg) {
  assert((TailAdd.getOpcode() == RISCV::ADD) && "Expected ADD instruction!");
  Register Rs = TailAdd.getOperand(1).getReg();
  Register Rt = TailAdd.getOperand(2).getReg();
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 209-225: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  Register Reg = Rs == GAReg ? Rt : Rs;

  // Can't fold if the register has more than one use.
  if (!Reg.isVirtual() || !MRI->hasOneUse(Reg))
    return false;
  // This can point to an ADDI(W) or a LUI:
  MachineInstr &OffsetTail = *MRI->getVRegDef(Reg);
  auto OffsetTailOpc = OffsetTail.getOpcode();
  if (OffsetTailOpc == RISCV::ADDI || OffsetTailOpc == RISCV::ADDIW) {
    // The offset value has non zero bits in both %hi and %lo parts.
    // Detect an ADDI that feeds from a LUI instruction.
    MachineOperand &AddiImmOp = OffsetTail.getOperand(2);
    if (AddiImmOp.getTargetFlags() != RISCVII::MO_None)
      return false;
    Register AddiReg = OffsetTail.getOperand(1).getReg();
    int64_t OffLo = AddiImmOp.getImm();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 226-253: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    // Handle rs1 of ADDI is X0.
    if (AddiReg == RISCV::X0) {
      LLVM_DEBUG(dbgs() << "  Offset Instrs: " << OffsetTail);
      if (!foldOffset(Hi, Lo, TailAdd, OffLo))
        return false;
      OffsetTail.eraseFromParent();
      return true;
    }

    MachineInstr &OffsetLui = *MRI->getVRegDef(AddiReg);
    MachineOperand &LuiImmOp = OffsetLui.getOperand(1);
    if (OffsetLui.getOpcode() != RISCV::LUI ||
        LuiImmOp.getTargetFlags() != RISCVII::MO_None ||
        !MRI->hasOneUse(OffsetLui.getOperand(0).getReg()))
      return false;
    int64_t Offset = SignExtend64<32>(LuiImmOp.getImm() << 12);
    Offset += OffLo;
    // RV32 ignores the upper 32 bits. ADDIW sign extends the result.
    if (!ST->is64Bit() || OffsetTailOpc == RISCV::ADDIW)
      Offset = SignExtend64<32>(Offset);
    // We can only fold simm32 offsets.
    if (!isInt<32>(Offset))
      return false;
    LLVM_DEBUG(dbgs() << "  Offset Instrs: " << OffsetTail
                      << "                 " << OffsetLui);
    if (!foldOffset(Hi, Lo, TailAdd, Offset))
      return false;
    OffsetTail.eraseFromParent();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 254-267: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    OffsetLui.eraseFromParent();
    return true;
  } else if (OffsetTailOpc == RISCV::LUI) {
    // The offset value has all zero bits in the lower 12 bits. Only LUI
    // exists.
    LLVM_DEBUG(dbgs() << "  Offset Instr: " << OffsetTail);
    int64_t Offset = SignExtend64<32>(OffsetTail.getOperand(1).getImm() << 12);
    if (!foldOffset(Hi, Lo, TailAdd, Offset))
      return false;
    OffsetTail.eraseFromParent();
    return true;
  }
  return false;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 268-288: Commentary and design intent / 注释与设计意图
```cpp

// Detect patterns for offsets that are passed into a SHXADD instruction.
// The offset has 1, 2, or 3 trailing zeros and fits in simm13, simm14, simm15.
// The constant is created with addi voff, x0, C, and shXadd is used to
// fill insert the trailing zeros and do the addition.
// If the pattern is found, updates the offset in Hi and Lo instructions
// and deletes TailShXAdd and the instructions that produced the offset.
//
// Hi:         lui     vreg1, %hi(s)
// Lo:         addi    vreg2, vreg1, %lo(s)
// OffsetTail: addi    voff, x0, C
// TailAdd:    shXadd  vreg4, voff, vreg2
bool RISCVMergeBaseOffsetOpt::foldShiftedOffset(MachineInstr &Hi,
                                                MachineInstr &Lo,
                                                MachineInstr &TailShXAdd,
                                                Register GAReg) {
  assert((TailShXAdd.getOpcode() == RISCV::SH1ADD ||
          TailShXAdd.getOpcode() == RISCV::SH2ADD ||
          TailShXAdd.getOpcode() == RISCV::SH3ADD) &&
         "Expected SHXADD instruction!");
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 289-306: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  if (GAReg != TailShXAdd.getOperand(2).getReg())
    return false;

  // The first source is the shifted operand.
  Register Rs1 = TailShXAdd.getOperand(1).getReg();

  // Can't fold if the register has more than one use.
  if (!Rs1.isVirtual() || !MRI->hasOneUse(Rs1))
    return false;
  // This can point to an ADDI X0, C.
  MachineInstr &OffsetTail = *MRI->getVRegDef(Rs1);
  if (OffsetTail.getOpcode() != RISCV::ADDI)
    return false;
  if (!OffsetTail.getOperand(1).isReg() ||
      OffsetTail.getOperand(1).getReg() != RISCV::X0 ||
      !OffsetTail.getOperand(2).isImm())
    return false;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 307-325: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  int64_t Offset = OffsetTail.getOperand(2).getImm();
  assert(isInt<12>(Offset) && "Unexpected offset");

  unsigned ShAmt;
  switch (TailShXAdd.getOpcode()) {
  default: llvm_unreachable("Unexpected opcode");
  case RISCV::SH1ADD: ShAmt = 1; break;
  case RISCV::SH2ADD: ShAmt = 2; break;
  case RISCV::SH3ADD: ShAmt = 3; break;
  }

  Offset = (uint64_t)Offset << ShAmt;

  LLVM_DEBUG(dbgs() << "  Offset Instr: " << OffsetTail);
  if (!foldOffset(Hi, Lo, TailShXAdd, Offset))
    return false;
  OffsetTail.eraseFromParent();
  return true;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 326-353: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

bool RISCVMergeBaseOffsetOpt::detectAndFoldOffset(MachineInstr &Hi,
                                                  MachineInstr &Lo) {
  Register DestReg = Lo.getOperand(0).getReg();

  // Look for arithmetic instructions we can get an offset from.
  // We might be able to remove the arithmetic instructions by folding the
  // offset into the LUI+ADDI.
  if (!MRI->hasOneUse(DestReg))
    return false;

  // Lo has only one use.
  MachineInstr &Tail = *MRI->use_instr_begin(DestReg);
  switch (Tail.getOpcode()) {
  default:
    LLVM_DEBUG(dbgs() << "Don't know how to get offset from this instr: "
                      << Tail);
    break;
  case RISCV::ADDI:
  case RISCV::QC_E_ADDI:
  case RISCV::QC_E_ADDAI: {
    // Offset is simply an immediate operand.
    int64_t Offset = Tail.getOperand(2).getImm();
    if (Tail.getOpcode() == RISCV::ADDI) {
      // We might have two ADDIs in a row.
      Register TailDestReg = Tail.getOperand(0).getReg();
      if (MRI->hasOneUse(TailDestReg)) {
        MachineInstr &TailTail = *MRI->use_instr_begin(TailDestReg);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 354-367: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
        if (TailTail.getOpcode() == RISCV::ADDI) {
          Offset += TailTail.getOperand(2).getImm();
          LLVM_DEBUG(dbgs() << "  Offset Instrs: " << Tail << TailTail);
          if (!foldOffset(Hi, Lo, TailTail, Offset))
            return false;
          Tail.eraseFromParent();
          return true;
        }
      }
    }

    LLVM_DEBUG(dbgs() << "  Offset Instr: " << Tail);
    return foldOffset(Hi, Lo, Tail, Offset);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 368-385: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  case RISCV::ADD:
    // The offset is too large to fit in the immediate field of ADDI.
    // This can be in two forms:
    // 1) LUI hi_Offset followed by:
    //    ADDI lo_offset
    //    This happens in case the offset has non zero bits in
    //    both hi 20 and lo 12 bits.
    // 2) LUI (offset20)
    //    This happens in case the lower 12 bits of the offset are zeros.
    return foldLargeOffset(Hi, Lo, Tail, DestReg);
  case RISCV::SH1ADD:
  case RISCV::SH2ADD:
  case RISCV::SH3ADD:
    // The offset is too large to fit in the immediate field of ADDI.
    // It may be encoded as (SH2ADD (ADDI X0, C), DestReg) or
    // (SH3ADD (ADDI X0, C), DestReg).
    return foldShiftedOffset(Hi, Lo, Tail, DestReg);
  }
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 386-410: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  return false;
}

static void overwriteMachineOperandInPlace(MachineOperand &MO,
                                           const MachineOperand &ImmOp) {
  switch (ImmOp.getType()) {
  case MachineOperand::MO_ConstantPoolIndex:
    MO.ChangeToCPI(ImmOp.getIndex(), ImmOp.getOffset(), ImmOp.getTargetFlags());
    break;
  case MachineOperand::MO_GlobalAddress:
    MO.ChangeToGA(ImmOp.getGlobal(), ImmOp.getOffset(), ImmOp.getTargetFlags());
    break;
  case MachineOperand::MO_MCSymbol:
    MO.ChangeToMCSymbol(ImmOp.getMCSymbol(), ImmOp.getTargetFlags());
    MO.setOffset(ImmOp.getOffset());
    break;
  case MachineOperand::MO_BlockAddress:
    MO.ChangeToBA(ImmOp.getBlockAddress(), ImmOp.getOffset(),
                  ImmOp.getTargetFlags());
    break;
  default:
    report_fatal_error("unsupported machine operand type");
    break;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 411-431: Function implementation: RISCVMergeBaseOffsetOpt::foldIntoMemoryOps / 函数实现：RISCVMergeBaseOffsetOpt::foldIntoMemoryOps
```cpp
}

bool RISCVMergeBaseOffsetOpt::foldIntoMemoryOps(MachineInstr &Hi,
                                                MachineInstr &Lo) {
  Register DestReg = Lo.getOperand(0).getReg();

  // If all the uses are memory ops with the same offset, we can transform:
  //
  // 1. (medlow pattern):
  //   a. Hi:   lui vreg1, %hi(foo)          --->  lui vreg1, %hi(foo+8)
  //      Lo:   addi vreg2, vreg1, %lo(foo)  --->  lw vreg3, lo(foo+8)(vreg1)
  //      Tail: lw vreg3, 8(vreg2)
  //
  //   b. Hi:   qc.e.li vreg1, foo           ---> qc.e.li vreg1, foo+8
  //      Tail: lw vreg2, 8(vreg1)           ---> lw vreg2, 0(vreg1)
  //
  // 2. (medany pattern):
  // Hi: 1:auipc vreg1, %pcrel_hi(s)         ---> auipc vreg1, %pcrel_hi(foo+8)
  // Lo:   addi  vreg2, vreg1, %pcrel_lo(1b) ---> lw vreg3, %pcrel_lo(1b)(vreg1)
  // Tail: lw vreg3, 8(vreg2)
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 432-459: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  std::optional<int64_t> CommonOffset;
  DenseMap<const MachineInstr *, SmallVector<unsigned>>
      InlineAsmMemoryOpIndexesMap;
  for (const MachineInstr &UseMI : MRI->use_instructions(DestReg)) {
    switch (UseMI.getOpcode()) {
    default:
      LLVM_DEBUG(dbgs() << "Not a load or store instruction: " << UseMI);
      return false;
    case RISCV::LB:
    case RISCV::LH:
    case RISCV::LH_INX:
    case RISCV::LW:
    case RISCV::LW_INX:
    case RISCV::LBU:
    case RISCV::LHU:
    case RISCV::LWU:
    case RISCV::LD:
    case RISCV::LD_RV32:
    case RISCV::FLH:
    case RISCV::FLW:
    case RISCV::FLD:
    case RISCV::SB:
    case RISCV::SH:
    case RISCV::SH_INX:
    case RISCV::SW:
    case RISCV::SW_INX:
    case RISCV::SD:
    case RISCV::SD_RV32:
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 460-476: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    case RISCV::FSH:
    case RISCV::FSW:
    case RISCV::FSD: {
      if (UseMI.getOperand(1).isFI())
        return false;
      // Register defined by Lo should not be the value register.
      if (DestReg == UseMI.getOperand(0).getReg())
        return false;
      assert(DestReg == UseMI.getOperand(1).getReg() &&
             "Expected base address use");
      // All load/store instructions must use the same offset.
      int64_t Offset = UseMI.getOperand(2).getImm();
      if (CommonOffset && Offset != CommonOffset)
        return false;
      CommonOffset = Offset;
      break;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 477-504: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    case RISCV::PseudoCCLD:
    case RISCV::PseudoCCLW:
    case RISCV::PseudoCCLWU:
    case RISCV::PseudoCCLH:
    case RISCV::PseudoCCLHU:
    case RISCV::PseudoCCLB:
    case RISCV::PseudoCCLBU: {
      // The SFB Pseudos are like their non-SFB counterparts but have more
      // operands.
      if (UseMI.getOperand(2).isFI())
        return false;
      // Register defined by Lo should not be the (tied) false value, or a
      // register used in the branch predicate.
      if (DestReg == UseMI.getOperand(1).getReg() ||
          DestReg == UseMI.getOperand(5).getReg())
        return false;
      if (UseMI.getOperand(6).isReg() &&
          DestReg == UseMI.getOperand(6).getReg())
        return false;
      assert(DestReg == UseMI.getOperand(2).getReg() &&
             "Expected base address use");
      // All load/store instructions must use the same offset.
      int64_t Offset = UseMI.getOperand(3).getImm();
      if (CommonOffset && Offset != CommonOffset)
        return false;
      CommonOffset = Offset;
      break;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 505-518: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    case RISCV::INLINEASM:
    case RISCV::INLINEASM_BR: {
      SmallVector<unsigned> InlineAsmMemoryOpIndexes;
      unsigned NumOps = 0;
      for (unsigned I = InlineAsm::MIOp_FirstOperand;
           I < UseMI.getNumOperands(); I += 1 + NumOps) {
        const MachineOperand &FlagsMO = UseMI.getOperand(I);
        // Should be an imm.
        if (!FlagsMO.isImm())
          continue;

        const InlineAsm::Flag Flags(FlagsMO.getImm());
        NumOps = Flags.getNumOperandRegisters();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 519-535: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
        // Memory constraints have two operands.
        if (NumOps != 2 || !Flags.isMemKind()) {
          // If the register is used by something other than a memory
          // constraint, we should not fold.
          for (unsigned J = 0; J < NumOps; ++J) {
            const MachineOperand &MO = UseMI.getOperand(I + 1 + J);
            if (MO.isReg() && MO.getReg() == DestReg)
              return false;
          }
          continue;
        }

        // We can't do this for constraint A because AMO instructions don't have
        // an immediate offset field.
        if (Flags.getMemoryConstraintID() == InlineAsm::ConstraintCode::A)
          return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 536-550: Definitions and supporting logic / 定义与支撑逻辑
```cpp
        const MachineOperand &AddrMO = UseMI.getOperand(I + 1);
        if (!AddrMO.isReg() || AddrMO.getReg() != DestReg)
          continue;

        const MachineOperand &OffsetMO = UseMI.getOperand(I + 2);
        if (!OffsetMO.isImm())
          continue;

        // All inline asm memory operands must use the same offset.
        int64_t Offset = OffsetMO.getImm();
        if (CommonOffset && Offset != CommonOffset)
          return false;
        CommonOffset = Offset;
        InlineAsmMemoryOpIndexes.push_back(I + 1);
      }
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 551-569: Definitions and supporting logic / 定义与支撑逻辑
```cpp
      InlineAsmMemoryOpIndexesMap.insert(
          std::make_pair(&UseMI, InlineAsmMemoryOpIndexes));
      break;
    }
    }
  }

  // We found a common offset.
  // Update the offsets in global address lowering.
  // We may have already folded some arithmetic so we need to add to any
  // existing offset.
  int64_t NewOffset = Hi.getOperand(1).getOffset() + *CommonOffset;
  // RV32 ignores the upper 32 bits.
  if (!ST->is64Bit())
    NewOffset = SignExtend64<32>(NewOffset);
  // We can only fold simm32 offsets.
  if (!isInt<32>(NewOffset))
    return false;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 570-583: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  Hi.getOperand(1).setOffset(NewOffset);
  MachineOperand &ImmOp =
      Hi.getOpcode() == RISCV::QC_E_LI ? Lo.getOperand(1) : Lo.getOperand(2);
  auto HiOpc = Hi.getOpcode();
  // Expand PseudoMovAddr into LUI
  if (HiOpc == RISCV::PseudoMovAddr) {
    auto *TII = ST->getInstrInfo();
    Hi.setDesc(TII->get(RISCV::LUI));
    Hi.removeOperand(2);
  }

  if (HiOpc != RISCV::AUIPC)
    ImmOp.setOffset(NewOffset);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 584-611: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Update the immediate in the load/store instructions to add the offset.
  for (MachineInstr &UseMI :
       llvm::make_early_inc_range(MRI->use_instructions(DestReg))) {
    if (UseMI.getOpcode() == RISCV::INLINEASM ||
        UseMI.getOpcode() == RISCV::INLINEASM_BR) {
      auto &InlineAsmMemoryOpIndexes = InlineAsmMemoryOpIndexesMap[&UseMI];
      for (unsigned I : InlineAsmMemoryOpIndexes) {
        MachineOperand &MO = UseMI.getOperand(I + 1);
        overwriteMachineOperandInPlace(MO, ImmOp);
      }
    } else {
      unsigned ImmIdx;
      switch (UseMI.getOpcode()) {
      case RISCV::INLINEASM:
      case RISCV::INLINEASM_BR:
        llvm_unreachable("Should have been dealt with before this else");
      case RISCV::PseudoCCLD:
      case RISCV::PseudoCCLW:
      case RISCV::PseudoCCLWU:
      case RISCV::PseudoCCLH:
      case RISCV::PseudoCCLHU:
      case RISCV::PseudoCCLB:
      case RISCV::PseudoCCLBU:
        ImmIdx = 3;
        break;
      case RISCV::LB:
      case RISCV::LH:
      case RISCV::LH_INX:
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 612-636: Definitions and supporting logic / 定义与支撑逻辑
```cpp
      case RISCV::LW:
      case RISCV::LW_INX:
      case RISCV::LBU:
      case RISCV::LHU:
      case RISCV::LWU:
      case RISCV::LD:
      case RISCV::LD_RV32:
      case RISCV::FLH:
      case RISCV::FLW:
      case RISCV::FLD:
      case RISCV::SB:
      case RISCV::SH:
      case RISCV::SH_INX:
      case RISCV::SW:
      case RISCV::SW_INX:
      case RISCV::SD:
      case RISCV::SD_RV32:
      case RISCV::FSH:
      case RISCV::FSW:
      case RISCV::FSD:
        ImmIdx = 2;
        break;
      default:
        llvm_unreachable("Unknown Instruction");
      }
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 637-651: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

      MachineOperand &MO = UseMI.getOperand(ImmIdx);
      if (Hi.getOpcode() == RISCV::QC_E_LI) {
        MO.ChangeToImmediate(0);
      } else {
        overwriteMachineOperandInPlace(MO, ImmOp);
      }
    }
  }

  // Prevent Lo (originally PseudoMovAddr, which is also pointed by Hi) from
  // being erased
  if (&Lo == &Hi)
    return true;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 652-665: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  MRI->replaceRegWith(Lo.getOperand(0).getReg(), Hi.getOperand(0).getReg());
  Lo.eraseFromParent();
  return true;
}

// Try to fold sequences of the form:
//   Hi/lo:    qc.e.li vreg1, s           -> qc.e.li vreg1, s+imm
//   TailAdd:  shxadd vreg2, vreg3, vreg1 -> deleted
//   Tail:     lx vreg4, imm(vreg2)       -> qc.lrx vreg4, vreg1, vreg3, (1-7)
bool RISCVMergeBaseOffsetOpt::foldShxaddIntoScaledMemory(MachineInstr &Hi,
                                                         MachineInstr &Lo) {
  if (!ST->hasVendorXqcisls() || ST->is64Bit())
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 666-693: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (Hi.getOpcode() != RISCV::QC_E_LI)
    return false;

  Register BaseReg = Hi.getOperand(0).getReg();
  if (!BaseReg.isVirtual() || !MRI->hasOneUse(BaseReg))
    return false;

  MachineInstr &ShxAdd = *MRI->use_instr_begin(BaseReg);
  unsigned ShxOpc = ShxAdd.getOpcode();
  unsigned ShAmt = 0;
  switch (ShxOpc) {
  default:
    return false;
  case RISCV::SH1ADD:
    ShAmt = 1;
    break;
  case RISCV::SH2ADD:
    ShAmt = 2;
    break;
  case RISCV::SH3ADD:
    ShAmt = 3;
    break;
  case RISCV::QC_SHLADD:
    uint8_t ShlImm = ShxAdd.getOperand(3).getImm();
    if (ShlImm > 7)
      return false;
    ShAmt = ShlImm;
    break;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 694-708: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  }

  // shxadd Rd, Rs1, Rs2
  Register ScaledReg = ShxAdd.getOperand(0).getReg();
  Register IndexReg = ShxAdd.getOperand(1).getReg();

  if (!IndexReg.isVirtual())
    return false;

  if (ShxAdd.getOperand(2).getReg() != BaseReg)
    return false;

  if (!ScaledReg.isVirtual() || !MRI->hasOneUse(ScaledReg))
    return false;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 709-736: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  MachineInstr &TailMem = *MRI->use_instr_begin(ScaledReg);
  unsigned Opc = TailMem.getOpcode();
  unsigned NewOpc = 0;

  switch (Opc) {
  case RISCV::LB:
    NewOpc = RISCV::QC_LRB;
    break;
  case RISCV::LBU:
    NewOpc = RISCV::QC_LRBU;
    break;
  case RISCV::LH:
    NewOpc = RISCV::QC_LRH;
    break;
  case RISCV::LHU:
    NewOpc = RISCV::QC_LRHU;
    break;
  case RISCV::LW:
    NewOpc = RISCV::QC_LRW;
    break;
  case RISCV::SB:
    NewOpc = RISCV::QC_SRB;
    break;
  case RISCV::SH:
    NewOpc = RISCV::QC_SRH;
    break;
  case RISCV::SW:
    NewOpc = RISCV::QC_SRW;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 737-751: Definitions and supporting logic / 定义与支撑逻辑
```cpp
    break;
  default:
    return false;
  }

  if (!TailMem.getOperand(1).isReg() ||
      TailMem.getOperand(1).getReg() != ScaledReg)
    return false;
  if (!TailMem.getOperand(2).isImm())
    return false;
  int64_t Imm = TailMem.getOperand(2).getImm();

  // Update QC_E_LI offset.
  int64_t NewOffset = SignExtend64<32>(Hi.getOperand(1).getOffset() + Imm);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 752-767: Type declaration for required / required 的类型声明
```cpp
  Hi.getOperand(1).setOffset(NewOffset);

  // Build scaled load/store.
  auto *TII = ST->getInstrInfo();
  auto *MBB = TailMem.getParent();

  // Ensure index register satisfies GPRNoX0 class required by QC_LR*/QC_SR*.
  MRI->constrainRegClass(IndexReg, &RISCV::GPRNoX0RegClass);

  BuildMI(*MBB, TailMem, TailMem.getDebugLoc(), TII->get(NewOpc))
      .add(TailMem.getOperand(0))
      .addReg(BaseReg, getKillRegState(ShxAdd.getOperand(2).isKill()))
      .addReg(IndexReg, getKillRegState(ShxAdd.getOperand(1).isKill()))
      .addImm(ShAmt)
      .cloneMemRefs(TailMem);
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 768-790: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  TailMem.eraseFromParent();
  ShxAdd.eraseFromParent();
  return true;
}

bool RISCVMergeBaseOffsetOpt::runOnMachineFunction(MachineFunction &Fn) {
  if (skipFunction(Fn.getFunction()))
    return false;

  ST = &Fn.getSubtarget<RISCVSubtarget>();

  bool MadeChange = false;
  MRI = &Fn.getRegInfo();
  for (MachineBasicBlock &MBB : Fn) {
    LLVM_DEBUG(dbgs() << "MBB: " << MBB.getName() << "\n");
    for (MachineInstr &Hi : MBB) {
      MachineInstr *Lo = nullptr;
      if (!detectFoldable(Hi, Lo))
        continue;
      MadeChange |= detectAndFoldOffset(Hi, *Lo);
      MadeChange |= foldIntoMemoryOps(Hi, *Lo);
      MadeChange |= foldShxaddIntoScaledMemory(Hi, *Lo);
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 791-799: Function implementation: llvm::createRISCVMergeBaseOffsetOptPass / 函数实现：llvm::createRISCVMergeBaseOffsetOptPass
```cpp
  }

  return MadeChange;
}

/// Returns an instance of the Merge Base Offset Optimization pass.
FunctionPass *llvm::createRISCVMergeBaseOffsetOptPass() {
  return new RISCVMergeBaseOffsetOpt();
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

## Key Concepts / 关键概念
- **RISC-V backend structure** / **RISC-V 后端结构**

## Dependencies / 依赖关系
- `RISCV.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVTargetMachine.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineFunctionPass.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/Passes.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/TargetRegistry.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Support/Debug.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Target/TargetOptions.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `optional` — Directly referenced by this file. / 该文件直接引用的依赖。
