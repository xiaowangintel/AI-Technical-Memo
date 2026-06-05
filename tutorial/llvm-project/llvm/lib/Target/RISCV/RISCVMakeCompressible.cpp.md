# RISCVMakeCompressible.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVMakeCompressible.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements transformations that make instructions encodable as RISC-V compressed forms. / 实现使指令可编码为 RISC-V 压缩形式的变换。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: Commentary and design intent / 注释与设计意图
```cpp
//===-- RISCVMakeCompressible.cpp - Make more instructions compressible ---===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass searches for instructions that are prevented from being compressed
// by one of the following:
//
//   1. The use of a single uncompressed register.
//   2. A base register + offset where the offset is too large to be compressed
//   and the base register may or may not be compressed.
//
//
// For case 1, if a compressed register is available, then the uncompressed
// register is copied to the compressed register and its uses are replaced.
//
// For example, storing zero uses the incompressible zero register:
//   sw zero, 0(a0)   # if zero
//   sw zero, 8(a0)   # if zero
//   sw zero, 4(a0)   # if zero
//   sw zero, 24(a0)   # if zero
//
// If a compressed register (e.g. a1) is available, the above can be transformed
// to the following to improve code size:
//   li a1, 0
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 29-56: Commentary and design intent / 注释与设计意图
```cpp
//   c.sw a1, 0(a0)
//   c.sw a1, 8(a0)
//   c.sw a1, 4(a0)
//   c.sw a1, 24(a0)
//
//
// For case 2, if a compressed register is available, then the original base
// is copied and adjusted such that:
//
//   new_base_register = base_register + adjustment
//   base_register + large_offset = new_base_register + small_offset
//
// For example, the following offsets are too large for c.sw:
//   lui a2, 983065
//   sw  a1, -236(a2)
//   sw  a1, -240(a2)
//   sw  a1, -244(a2)
//   sw  a1, -248(a2)
//   sw  a1, -252(a2)
//   sw  a0, -256(a2)
//
// If a compressed register is available (e.g. a3), a new base could be created
// such that the addresses can accessed with a compressible offset, thus
// improving code size:
//   lui a2, 983065
//   addi  a3, a2, -256
//   c.sw  a1, 20(a3)
//   c.sw  a1, 16(a3)
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 57-74: File prologue and imported dependencies / 文件前言与导入依赖
```cpp
//   c.sw  a1, 12(a3)
//   c.sw  a1, 8(a3)
//   c.sw  a1, 4(a3)
//   c.sw  a0, 0(a3)
//
//
// This optimization is only applied if there are enough uses of the copied
// register for code size to be reduced.
//
//===----------------------------------------------------------------------===//

#include "RISCV.h"
#include "RISCVSubtarget.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/CodeGen/RegisterScavenging.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Debug.h"
```
**EN:** This opening block combines banner comments with the first wave of includes, giving readers both context and the main compile-time dependencies.

**CN:** 该开头区段同时包含说明性注释与首批 include，让读者先看到文件定位，再看到主要编译期依赖。

### Lines 75-88: Header guard and interface framing / 头文件保护与接口框架
```cpp
using namespace llvm;

#define DEBUG_TYPE "riscv-make-compressible"
#define RISCV_COMPRESS_INSTRS_NAME "RISC-V Make Compressible"

namespace {

struct RISCVMakeCompressibleOpt : public MachineFunctionPass {
  static char ID;

  bool runOnMachineFunction(MachineFunction &Fn) override;

  RISCVMakeCompressibleOpt() : MachineFunctionPass(ID) {}
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 89-116: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
  StringRef getPassName() const override { return RISCV_COMPRESS_INSTRS_NAME; }
};
} // namespace

char RISCVMakeCompressibleOpt::ID = 0;
INITIALIZE_PASS(RISCVMakeCompressibleOpt, "riscv-make-compressible",
                RISCV_COMPRESS_INSTRS_NAME, false, false)

// Return log2(widthInBytes) of load/store done by Opcode.
static unsigned log2LdstWidth(unsigned Opcode) {
  switch (Opcode) {
  default:
    llvm_unreachable("Unexpected opcode");
  case RISCV::LBU:
  case RISCV::SB:
  case RISCV::QC_E_LBU:
  case RISCV::QC_E_SB:
    return 0;
  case RISCV::LH:
  case RISCV::LH_INX:
  case RISCV::LHU:
  case RISCV::SH:
  case RISCV::SH_INX:
  case RISCV::QC_E_LH:
  case RISCV::QC_E_LHU:
  case RISCV::QC_E_SH:
    return 1;
  case RISCV::LW:
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 117-132: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  case RISCV::LW_INX:
  case RISCV::SW:
  case RISCV::SW_INX:
  case RISCV::FLW:
  case RISCV::FSW:
  case RISCV::QC_E_LW:
  case RISCV::QC_E_SW:
    return 2;
  case RISCV::LD:
  case RISCV::LD_RV32:
  case RISCV::SD:
  case RISCV::SD_RV32:
  case RISCV::FLD:
  case RISCV::FSD:
    return 3;
  }
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 133-160: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
}

// Return bit field size of immediate operand of Opcode.
static unsigned offsetMask(unsigned Opcode) {
  switch (Opcode) {
  default:
    llvm_unreachable("Unexpected opcode");
  case RISCV::LBU:
  case RISCV::SB:
  case RISCV::QC_E_LBU:
  case RISCV::QC_E_SB:
    return maskTrailingOnes<unsigned>(2U);
  case RISCV::LH:
  case RISCV::LH_INX:
  case RISCV::LHU:
  case RISCV::SH:
  case RISCV::SH_INX:
  case RISCV::QC_E_LH:
  case RISCV::QC_E_LHU:
  case RISCV::QC_E_SH:
    return maskTrailingOnes<unsigned>(1U);
  case RISCV::LW:
  case RISCV::LW_INX:
  case RISCV::SW:
  case RISCV::SW_INX:
  case RISCV::FLW:
  case RISCV::FSW:
  case RISCV::LD:
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 161-176: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  case RISCV::LD_RV32:
  case RISCV::SD:
  case RISCV::SD_RV32:
  case RISCV::FLD:
  case RISCV::FSD:
  case RISCV::QC_E_LW:
  case RISCV::QC_E_SW:
    return maskTrailingOnes<unsigned>(5U);
  }
}

// Return a mask for the offset bits of a non-stack-pointer based compressed
// load/store.
static uint8_t compressedLDSTOffsetMask(unsigned Opcode) {
  return offsetMask(Opcode) << log2LdstWidth(Opcode);
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 177-190: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

// Return true if Offset fits within a compressed stack-pointer based
// load/store.
static bool compressibleSPOffset(int64_t Offset, unsigned Opcode) {
  // Compressed sp-based loads and stores only work for 32/64 bits.
  switch (log2LdstWidth(Opcode)) {
  case 2:
    return isShiftedUInt<6, 2>(Offset);
  case 3:
    return isShiftedUInt<6, 3>(Offset);
  }
  return false;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 191-207: Type declaration for getBaseAdjustForCompression / getBaseAdjustForCompression 的类型声明
```cpp
// Given an offset for a load/store, return the adjustment required to the base
// register such that the address can be accessed with a compressible offset.
// This will return 0 if the offset is already compressible.
static int64_t getBaseAdjustForCompression(int64_t Offset, unsigned Opcode) {
  // Return the excess bits that do not fit in a compressible offset.
  return Offset & ~compressedLDSTOffsetMask(Opcode);
}

// Return true if Reg is in a compressed register class.
static bool isCompressedReg(Register Reg) {
  return RISCV::GPRCRegClass.contains(Reg) ||
         RISCV::GPRF16CRegClass.contains(Reg) ||
         RISCV::GPRF32CRegClass.contains(Reg) ||
         RISCV::FPR32CRegClass.contains(Reg) ||
         RISCV::FPR64CRegClass.contains(Reg) ||
         RISCV::GPRPairCRegClass.contains(Reg);
}
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 208-235: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

// Return true if MI is a load for which there exists a compressed version.
static bool isCompressibleLoad(const MachineInstr &MI) {
  const RISCVSubtarget &STI = MI.getMF()->getSubtarget<RISCVSubtarget>();

  switch (MI.getOpcode()) {
  default:
    return false;
  case RISCV::LBU:
  case RISCV::LH:
  case RISCV::LH_INX:
  case RISCV::LHU:
    return STI.hasStdExtZcb();
  case RISCV::LW:
  case RISCV::LW_INX:
  case RISCV::LD:
    return STI.hasStdExtZca();
  case RISCV::LD_RV32:
    return STI.hasStdExtZclsd();
  case RISCV::FLW:
    return !STI.is64Bit() && STI.hasStdExtZcf();
  case RISCV::FLD:
    return STI.hasStdExtZcd();
  // For the Xqcilo loads we mark it as compressible only if Xqcilia is also
  // enabled so that QC_E_ADDI can be used to create the new base.
  case RISCV::QC_E_LBU:
  case RISCV::QC_E_LH:
  case RISCV::QC_E_LHU:
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 236-263: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    return !STI.is64Bit() && STI.hasVendorXqcilo() && STI.hasVendorXqcilia() &&
           STI.hasStdExtZcb();
  case RISCV::QC_E_LW:
    return !STI.is64Bit() && STI.hasVendorXqcilo() && STI.hasVendorXqcilia();
  }
}

// Return true if MI is a store for which there exists a compressed version.
static bool isCompressibleStore(const MachineInstr &MI) {
  const RISCVSubtarget &STI = MI.getMF()->getSubtarget<RISCVSubtarget>();

  switch (MI.getOpcode()) {
  default:
    return false;
  case RISCV::SB:
  case RISCV::SH:
  case RISCV::SH_INX:
    return STI.hasStdExtZcb();
  case RISCV::SW:
  case RISCV::SW_INX:
  case RISCV::SD:
    return STI.hasStdExtZca();
  case RISCV::SD_RV32:
    return STI.hasStdExtZclsd();
  case RISCV::FSW:
    return !STI.is64Bit() && STI.hasStdExtZcf();
  case RISCV::FSD:
    return STI.hasStdExtZcd();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 264-289: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // For the Xqcilo stores we mark it as compressible only if Xqcilia is also
  // enabled so that QC_E_ADDI can be used to create the new base.
  case RISCV::QC_E_SB:
  case RISCV::QC_E_SH:
    return !STI.is64Bit() && STI.hasVendorXqcilo() && STI.hasVendorXqcilia() &&
           STI.hasStdExtZcb();
  case RISCV::QC_E_SW:
    return !STI.is64Bit() && STI.hasVendorXqcilo() && STI.hasVendorXqcilia();
  }
}

// Find a single register and/or large offset which, if compressible, would
// allow the given instruction to be compressed.
//
// Possible return values:
//
//   {Reg, 0}               - Uncompressed Reg needs replacing with a compressed
//                            register.
//   {Reg, N}               - Reg needs replacing with a compressed register and
//                            N needs adding to the new register. (Reg may be
//                            compressed or uncompressed).
//   {RISCV::NoRegister, 0} - No suitable optimization found for this
//   instruction.
static RegImmPair getRegImmPairPreventingCompression(const MachineInstr &MI) {
  const unsigned Opcode = MI.getOpcode();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 290-308: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (isCompressibleLoad(MI) || isCompressibleStore(MI)) {
    const MachineOperand &MOImm = MI.getOperand(2);
    if (!MOImm.isImm())
      return RegImmPair(Register(), 0);

    int64_t Offset = MOImm.getImm();
    int64_t NewBaseAdjust = getBaseAdjustForCompression(Offset, Opcode);
    Register Base = MI.getOperand(1).getReg();

    // Memory accesses via the stack pointer do not have a requirement for
    // either of the registers to be compressible and can take a larger offset.
    if (RISCV::SPRegClass.contains(Base)) {
      if (!compressibleSPOffset(Offset, Opcode) && NewBaseAdjust)
        return RegImmPair(Base, NewBaseAdjust);
    } else {
      Register SrcDest = MI.getOperand(0).getReg();
      bool SrcDestCompressed = isCompressedReg(SrcDest);
      bool BaseCompressed = isCompressedReg(Base);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 309-323: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
      // If only Base and/or offset prevent compression, then return Base and
      // any adjustment required to make the offset compressible.
      if ((!BaseCompressed || NewBaseAdjust) && SrcDestCompressed)
        return RegImmPair(Base, NewBaseAdjust);

      // For loads, we can only change the base register since dest is defined
      // rather than used.
      //
      // For stores, we can change SrcDest (and Base if SrcDest == Base) but
      // cannot resolve an incompressible offset in this case.
      if (isCompressibleStore(MI)) {
        if (!SrcDestCompressed && (BaseCompressed || SrcDest == Base) &&
            !NewBaseAdjust)
          return RegImmPair(SrcDest, NewBaseAdjust);
      }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 324-341: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    }
  }
  return RegImmPair(Register(), 0);
}

static bool isXqciloLdSt(const MachineInstr &MI) {
  switch (MI.getOpcode()) {
  default:
    return false;
  case RISCV::QC_E_SB:
  case RISCV::QC_E_SH:
  case RISCV::QC_E_SW:
  case RISCV::QC_E_LBU:
  case RISCV::QC_E_LH:
  case RISCV::QC_E_LHU:
  case RISCV::QC_E_LW:
    return true;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 342-357: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
}

// Check all uses after FirstMI of the given register, keeping a vector of
// instructions that would be compressible if the given register (and offset if
// applicable) were compressible.
//
// If there are enough uses for this optimization to improve code size and a
// compressed register is available, return that compressed register.
static Register analyzeCompressibleUses(MachineInstr &FirstMI,
                                        RegImmPair RegImm,
                                        SmallVectorImpl<MachineInstr *> &MIs) {
  MachineBasicBlock &MBB = *FirstMI.getParent();
  const TargetRegisterInfo *TRI =
      MBB.getParent()->getSubtarget().getRegisterInfo();
  bool XqciloLdSt = false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 358-371: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  for (MachineBasicBlock::instr_iterator I = FirstMI.getIterator(),
                                         E = MBB.instr_end();
       I != E; ++I) {
    MachineInstr &MI = *I;

    // Determine if this is an instruction which would benefit from using the
    // new register.
    RegImmPair CandidateRegImm = getRegImmPairPreventingCompression(MI);
    if (CandidateRegImm.Reg == RegImm.Reg &&
        CandidateRegImm.Imm == RegImm.Imm) {
      XqciloLdSt |= isXqciloLdSt(MI);
      MIs.push_back(&MI);
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 372-399: Commentary and design intent / 注释与设计意图
```cpp
    // If RegImm.Reg is modified by this instruction, then we cannot optimize
    // past this instruction. If the register is already compressed, then it may
    // possible to optimize a large offset in the current instruction - this
    // will have been detected by the preceding call to
    // getRegImmPairPreventingCompression.
    if (MI.modifiesRegister(RegImm.Reg, TRI))
      break;
  }

  // Adjusting the base costs:
  // a.                  --> addi (uncompressed 4 bytes)
  //     lw/sw (4 bytes) --> compressed to 2 bytes
  //     lw/sw (4 bytes) --> compressed to 2 bytes
  //     lw/sw (4 bytes) --> compressed to 2 bytes
  // at least three lw/sw instructions for code size reduction.
  //
  // b.                       --> qc.e.addi (uncompressed 6 bytes)
  //     qc.e.lw/sw (6 bytes) --> compressed to 2 bytes
  //     qc.e.lw/sw (6 bytes) --> compressed to 2 bytes
  // at least two qc.e.lw/sw instructions for code size reduction.
  //
  // If no base adjustment is required, then copying the register costs one new
  // c.mv (or c.li Rd, 0 for "copying" the zero register) and therefore two uses
  // are required for a code size reduction. For GPR pairs, we need 2 ADDIs to
  // copy so we need three users.
  unsigned BaseCost = XqciloLdSt ? 2 : 3;
  unsigned CopyCost = RISCV::GPRPairRegClass.contains(RegImm.Reg) ? 2 : 1;
  assert((RegImm.Imm == 0 || CopyCost == 1) && "GPRPair should have zero imm");
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 400-422: Type declaration for from / from 的类型声明
```cpp
  if (MIs.size() <= CopyCost || (RegImm.Imm != 0 && MIs.size() < BaseCost))
    return Register();

  // Find a compressible register which will be available from the first
  // instruction we care about to the last.
  const TargetRegisterClass *RCToScavenge;

  // Work out the compressed register class from which to scavenge.
  if (RISCV::GPRRegClass.contains(RegImm.Reg))
    RCToScavenge = &RISCV::GPRCRegClass;
  else if (RISCV::GPRF16RegClass.contains(RegImm.Reg))
    RCToScavenge = &RISCV::GPRF16CRegClass;
  else if (RISCV::GPRF32RegClass.contains(RegImm.Reg))
    RCToScavenge = &RISCV::GPRF32CRegClass;
  else if (RISCV::FPR32RegClass.contains(RegImm.Reg))
    RCToScavenge = &RISCV::FPR32CRegClass;
  else if (RISCV::FPR64RegClass.contains(RegImm.Reg))
    RCToScavenge = &RISCV::FPR64CRegClass;
  else if (RISCV::GPRPairRegClass.contains(RegImm.Reg))
    RCToScavenge = &RISCV::GPRPairCRegClass;
  else
    return Register();
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 423-440: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  RegScavenger RS;
  RS.enterBasicBlockEnd(MBB);
  RS.backward(std::next(MIs.back()->getIterator()));
  return RS.scavengeRegisterBackwards(*RCToScavenge, FirstMI.getIterator(),
                                      /*RestoreAfter=*/false, /*SPAdj=*/0,
                                      /*AllowSpill=*/false);
}

// Update uses of the old register in the given instruction to the new register.
static void updateOperands(MachineInstr &MI, RegImmPair OldRegImm,
                           Register NewReg) {
  unsigned Opcode = MI.getOpcode();

  // If this pass is extended to support more instructions, the check for
  // definedness may need to be strengthened.
  assert((isCompressibleLoad(MI) || isCompressibleStore(MI)) &&
         "Unsupported instruction for this optimization.");
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 441-460: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  int SkipN = 0;

  // Skip the first (value) operand to a store instruction (except if the store
  // offset is zero) in order to avoid an incorrect transformation.
  // e.g. sd a0, 808(a0) to addi a2, a0, 768; sd a2, 40(a2)
  if (isCompressibleStore(MI) && OldRegImm.Imm != 0)
    SkipN = 1;

  // Update registers
  for (MachineOperand &MO : drop_begin(MI.operands(), SkipN))
    if (MO.isReg() && MO.getReg() == OldRegImm.Reg) {
      // Do not update operands that define the old register.
      //
      // The new register was scavenged for the range of instructions that are
      // being updated, therefore it should not be defined within this range
      // except possibly in the final instruction.
      if (MO.isDef()) {
        assert(isCompressibleLoad(MI));
        continue;
      }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 461-475: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
      // Update reg
      MO.setReg(NewReg);
    }

  // Update offset
  MachineOperand &MOImm = MI.getOperand(2);
  int64_t NewOffset = MOImm.getImm() & compressedLDSTOffsetMask(Opcode);
  MOImm.setImm(NewOffset);
}

bool RISCVMakeCompressibleOpt::runOnMachineFunction(MachineFunction &Fn) {
  // This is a size optimization.
  if (skipFunction(Fn.getFunction()) || !Fn.getFunction().hasMinSize())
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 476-491: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  const RISCVSubtarget &STI = Fn.getSubtarget<RISCVSubtarget>();
  const RISCVInstrInfo &TII = *STI.getInstrInfo();

  // This optimization only makes sense if compressed instructions are emitted.
  if (!STI.hasStdExtZca())
    return false;

  for (MachineBasicBlock &MBB : Fn) {
    LLVM_DEBUG(dbgs() << "MBB: " << MBB.getName() << "\n");
    for (MachineInstr &MI : MBB) {
      // Determine if this instruction would otherwise be compressed if not for
      // an incompressible register or offset.
      RegImmPair RegImm = getRegImmPairPreventingCompression(MI);
      if (!RegImm.Reg && RegImm.Imm == 0)
        continue;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 492-511: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
      // Determine if there is a set of instructions for which replacing this
      // register with a compressed register (and compressible offset if
      // applicable) is possible and will allow compression.
      SmallVector<MachineInstr *, 8> MIs;
      Register NewReg = analyzeCompressibleUses(MI, RegImm, MIs);
      if (!NewReg)
        continue;

      // Create the appropriate copy and/or offset.
      if (RISCV::GPRRegClass.contains(RegImm.Reg)) {
        if (isInt<12>(RegImm.Imm)) {
          BuildMI(MBB, MI, MI.getDebugLoc(), TII.get(RISCV::ADDI), NewReg)
              .addReg(RegImm.Reg)
              .addImm(RegImm.Imm);
        } else {
          assert(STI.hasVendorXqcilia() && isInt<26>(RegImm.Imm));
          BuildMI(MBB, MI, MI.getDebugLoc(), TII.get(RISCV::QC_E_ADDI), NewReg)
              .addReg(RegImm.Reg)
              .addImm(RegImm.Imm);
        }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 512-525: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
      } else {
        assert(RegImm.Imm == 0);
        TII.copyPhysReg(MBB, MI, MI.getDebugLoc(), NewReg, RegImm.Reg,
                        /*KillSrc*/ false);
      }

      // Update the set of instructions to use the compressed register and
      // compressible offset instead. These instructions should now be
      // compressible.
      // TODO: Update all uses if RegImm.Imm == 0? Not just those that are
      // expected to become compressible.
      for (MachineInstr *UpdateMI : MIs)
        updateOperands(*UpdateMI, RegImm, NewReg);
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 526-533: Function implementation: llvm::createRISCVMakeCompressibleOptPass / 函数实现：llvm::createRISCVMakeCompressibleOptPass
```cpp
  }
  return true;
}

/// Returns an instance of the Make Compressible Optimization pass.
FunctionPass *llvm::createRISCVMakeCompressibleOptPass() {
  return new RISCVMakeCompressibleOpt();
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

## Key Concepts / 关键概念
- **RISC-V backend structure** / **RISC-V 后端结构**

## Dependencies / 依赖关系
- `RISCV.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVSubtarget.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/Passes.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/RegisterScavenging.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/TargetRegistry.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Support/Debug.h` — Directly referenced by this file. / 该文件直接引用的依赖。
