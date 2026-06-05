# RISCVDisassembler.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/Disassembler/RISCVDisassembler.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements RISC-V machine-code decoding into LLVM MC instructions. / 实现将 RISC-V 机器码解码为 LLVM MC 指令的逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-27: File prologue and imported dependencies / 文件前言与导入依赖
```cpp
//===-- RISCVDisassembler.cpp - Disassembler for RISC-V -------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the RISCVDisassembler class.
//
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/RISCVBaseInfo.h"
#include "MCTargetDesc/RISCVMCTargetDesc.h"
#include "TargetInfo/RISCVTargetInfo.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCDecoder.h"
#include "llvm/MC/MCDecoderOps.h"
#include "llvm/MC/MCDisassembler/MCDisassembler.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/Endian.h"
```
**EN:** This opening block combines banner comments with the first wave of includes, giving readers both context and the main compile-time dependencies.

**CN:** 该开头区段同时包含说明性注释与首批 include，让读者先看到文件定位，再看到主要编译期依赖。

### Lines 28-43: Header guard and interface framing / 头文件保护与接口框架
```cpp
using namespace llvm;
using namespace llvm::MCD;

#define DEBUG_TYPE "riscv-disassembler"

typedef MCDisassembler::DecodeStatus DecodeStatus;

namespace {
class RISCVDisassembler : public MCDisassembler {
  std::unique_ptr<MCInstrInfo const> const MCII;

public:
  RISCVDisassembler(const MCSubtargetInfo &STI, MCContext &Ctx,
                    MCInstrInfo const *MCII)
      : MCDisassembler(STI, Ctx), MCII(MCII) {}
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 44-59: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  DecodeStatus getInstruction(MCInst &Instr, uint64_t &Size,
                              ArrayRef<uint8_t> Bytes, uint64_t Address,
                              raw_ostream &CStream) const override;

private:
  DecodeStatus getInstruction48(MCInst &Instr, uint64_t &Size,
                                ArrayRef<uint8_t> Bytes, uint64_t Address,
                                raw_ostream &CStream) const;

  DecodeStatus getInstruction32(MCInst &Instr, uint64_t &Size,
                                ArrayRef<uint8_t> Bytes, uint64_t Address,
                                raw_ostream &CStream) const;
  DecodeStatus getInstruction16(MCInst &Instr, uint64_t &Size,
                                ArrayRef<uint8_t> Bytes, uint64_t Address,
                                raw_ostream &CStream) const;
};
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 60-79: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
} // end anonymous namespace

static MCDisassembler *createRISCVDisassembler(const Target &T,
                                               const MCSubtargetInfo &STI,
                                               MCContext &Ctx) {
  return new RISCVDisassembler(STI, Ctx, T.createMCInstrInfo());
}

extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
LLVMInitializeRISCVDisassembler() {
  // Register the disassembler for each target.
  TargetRegistry::RegisterMCDisassembler(getTheRISCV32Target(),
                                         createRISCVDisassembler);
  TargetRegistry::RegisterMCDisassembler(getTheRISCV64Target(),
                                         createRISCVDisassembler);
  TargetRegistry::RegisterMCDisassembler(getTheRISCV32beTarget(),
                                         createRISCVDisassembler);
  TargetRegistry::RegisterMCDisassembler(getTheRISCV64beTarget(),
                                         createRISCVDisassembler);
}
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 80-94: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

template <unsigned FirstReg, unsigned NumRegsInClass, unsigned RVELimit = 0>
static DecodeStatus DecodeSimpleRegisterClass(MCInst &Inst, uint32_t RegNo,
                                              uint64_t Address,
                                              const MCDisassembler *Decoder) {
  bool CheckRVE = RVELimit != 0 &&
                  Decoder->getSubtargetInfo().hasFeature(RISCV::FeatureStdExtE);

  if (RegNo >= NumRegsInClass || (CheckRVE && RegNo >= RVELimit))
    return MCDisassembler::Fail;

  MCRegister Reg = FirstReg + RegNo;
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 95-108: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

constexpr auto DecodeGPRRegisterClass =
    DecodeSimpleRegisterClass<RISCV::X0, 32, /*RVELimit=*/16>;

static DecodeStatus DecodeGPRX1X5RegisterClass(MCInst &Inst, uint32_t RegNo,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  MCRegister Reg = RISCV::X0 + RegNo;
  if (Reg != RISCV::X1 && Reg != RISCV::X5)
    return MCDisassembler::Fail;

  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 109-128: Function implementation: MCOperand::createReg / 函数实现：MCOperand::createReg
```cpp

static DecodeStatus DecodeGPRX1RegisterClass(MCInst &Inst,
                                             const MCDisassembler *Decoder) {
  Inst.addOperand(MCOperand::createReg(RISCV::X1));
  return MCDisassembler::Success;
}

static DecodeStatus DecodeSPRegisterClass(MCInst &Inst,
                                          const MCDisassembler *Decoder) {
  Inst.addOperand(MCOperand::createReg(RISCV::X2));
  return MCDisassembler::Success;
}

static DecodeStatus DecodeSPRegisterClass(MCInst &Inst, uint64_t RegNo,
                                          uint32_t Address,
                                          const MCDisassembler *Decoder) {
  assert(RegNo == 2);
  Inst.addOperand(MCOperand::createReg(RISCV::X2));
  return MCDisassembler::Success;
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 129-143: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

static DecodeStatus DecodeGPRX5RegisterClass(MCInst &Inst,
                                             const MCDisassembler *Decoder) {
  Inst.addOperand(MCOperand::createReg(RISCV::X5));
  return MCDisassembler::Success;
}

static DecodeStatus DecodeGPRNoX0RegisterClass(MCInst &Inst, uint32_t RegNo,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  if (RegNo == 0)
    return MCDisassembler::Fail;

  return DecodeGPRRegisterClass(Inst, RegNo, Address, Decoder);
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 144-159: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

static DecodeStatus DecodeGPRNoX2RegisterClass(MCInst &Inst, uint64_t RegNo,
                                               uint32_t Address,
                                               const MCDisassembler *Decoder) {
  if (RegNo == 2)
    return MCDisassembler::Fail;

  return DecodeGPRRegisterClass(Inst, RegNo, Address, Decoder);
}

static DecodeStatus DecodeGPRNoX31RegisterClass(MCInst &Inst, uint32_t RegNo,
                                                uint64_t Address,
                                                const MCDisassembler *Decoder) {
  if (RegNo == 31) {
    return MCDisassembler::Fail;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 160-178: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  return DecodeGPRRegisterClass(Inst, RegNo, Address, Decoder);
}

static DecodeStatus DecodeGPRPairRegisterClass(MCInst &Inst, uint32_t RegNo,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  if (RegNo >= 32 || RegNo % 2)
    return MCDisassembler::Fail;

  const RISCVDisassembler *Dis =
      static_cast<const RISCVDisassembler *>(Decoder);
  const MCRegisterInfo *RI = Dis->getContext().getRegisterInfo();
  MCRegister Reg = RI->getMatchingSuperReg(
      RISCV::X0 + RegNo, RISCV::sub_gpr_even,
      &RISCVMCRegisterClasses[RISCV::GPRPairRegClassID]);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 179-194: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

static DecodeStatus
DecodeGPRPairNoX0RegisterClass(MCInst &Inst, uint32_t RegNo, uint64_t Address,
                               const MCDisassembler *Decoder) {
  if (RegNo == 0)
    return MCDisassembler::Fail;

  return DecodeGPRPairRegisterClass(Inst, RegNo, Address, Decoder);
}

static DecodeStatus DecodeGPRPairCRegisterClass(MCInst &Inst, uint32_t RegNo,
                                                uint64_t Address,
                                                const MCDisassembler *Decoder) {
  if (RegNo >= 8 || RegNo % 2)
    return MCDisassembler::Fail;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 195-210: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  const RISCVDisassembler *Dis =
      static_cast<const RISCVDisassembler *>(Decoder);
  const MCRegisterInfo *RI = Dis->getContext().getRegisterInfo();
  MCRegister Reg = RI->getMatchingSuperReg(
      RISCV::X8 + RegNo, RISCV::sub_gpr_even,
      &RISCVMCRegisterClasses[RISCV::GPRPairCRegClassID]);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

static DecodeStatus DecodeSR07RegisterClass(MCInst &Inst, uint32_t RegNo,
                                            uint64_t Address,
                                            const void *Decoder) {
  if (RegNo >= 8)
    return MCDisassembler::Fail;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 211-229: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  MCRegister Reg = (RegNo < 2) ? (RegNo + RISCV::X8) : (RegNo - 2 + RISCV::X18);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

template <unsigned RegisterClass, unsigned NumRegsInClass, unsigned LMul>
static DecodeStatus DecodeVectorRegisterClass(MCInst &Inst, uint32_t RegNo,
                                              uint64_t Address,
                                              const MCDisassembler *Decoder) {
  if (RegNo >= NumRegsInClass || RegNo % LMul)
    return MCDisassembler::Fail;

  const RISCVDisassembler *Dis =
      static_cast<const RISCVDisassembler *>(Decoder);
  const MCRegisterInfo *RI = Dis->getContext().getRegisterInfo();
  MCRegister Reg =
      RI->getMatchingSuperReg(RISCV::V0 + RegNo, RISCV::sub_vrm1_0,
                              &RISCVMCRegisterClasses[RegisterClass]);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 230-243: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

static DecodeStatus DecodeTRM2RegisterClass(MCInst &Inst, uint32_t RegNo,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  if (RegNo > 15 || RegNo % 2)
    return MCDisassembler::Fail;

  MCRegister Reg = RISCV::T0 + RegNo;
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 244-261: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

static DecodeStatus DecodeTRM4RegisterClass(MCInst &Inst, uint32_t RegNo,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  if (RegNo > 15 || RegNo % 4)
    return MCDisassembler::Fail;

  MCRegister Reg = RISCV::T0 + RegNo;
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

static DecodeStatus decodeVMaskReg(MCInst &Inst, uint32_t RegNo,
                                   uint64_t Address,
                                   const MCDisassembler *Decoder) {
  if (RegNo >= 2)
    return MCDisassembler::Fail;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 262-278: Function implementation: MCOperand::createReg / 函数实现：MCOperand::createReg
```cpp
  MCRegister Reg = (RegNo == 0) ? RISCV::V0 : RISCV::NoRegister;

  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

static DecodeStatus decodeImmThreeOperand(MCInst &Inst,
                                          const MCDisassembler *Decoder) {
  Inst.addOperand(MCOperand::createImm(3));
  return MCDisassembler::Success;
}

static DecodeStatus decodeImmFourOperand(MCInst &Inst,
                                         const MCDisassembler *Decoder) {
  Inst.addOperand(MCOperand::createImm(4));
  return MCDisassembler::Success;
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 279-294: Function implementation: MCOperand::createImm / 函数实现：MCOperand::createImm
```cpp

template <unsigned N>
static DecodeStatus decodeUImmOperand(MCInst &Inst, uint32_t Imm,
                                      int64_t Address,
                                      const MCDisassembler *Decoder) {
  assert(isUInt<N>(Imm) && "Invalid immediate");
  Inst.addOperand(MCOperand::createImm(Imm));
  return MCDisassembler::Success;
}

template <unsigned Width, unsigned LowerBound>
static DecodeStatus decodeUImmOperandGE(MCInst &Inst, uint32_t Imm,
                                        int64_t Address,
                                        const MCDisassembler *Decoder) {
  assert(isUInt<Width>(Imm) && "Invalid immediate");
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 295-310: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (Imm < LowerBound)
    return MCDisassembler::Fail;

  Inst.addOperand(MCOperand::createImm(Imm));
  return MCDisassembler::Success;
}

template <unsigned Width, unsigned LowerBound>
static DecodeStatus decodeUImmPlus1OperandGE(MCInst &Inst, uint32_t Imm,
                                             int64_t Address,
                                             const MCDisassembler *Decoder) {
  assert(isUInt<Width>(Imm) && "Invalid immediate");

  if ((Imm + 1) < LowerBound)
    return MCDisassembler::Fail;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 311-328: Function implementation: MCOperand::createImm / 函数实现：MCOperand::createImm
```cpp
  Inst.addOperand(MCOperand::createImm(Imm + 1));
  return MCDisassembler::Success;
}

static DecodeStatus decodeUImmSlistOperand(MCInst &Inst, uint32_t Imm,
                                           int64_t Address,
                                           const MCDisassembler *Decoder) {
  assert(isUInt<3>(Imm) && "Invalid Slist immediate");
  const uint8_t Slist[] = {0, 1, 2, 4, 8, 16, 15, 31};
  Inst.addOperand(MCOperand::createImm(Slist[Imm]));
  return MCDisassembler::Success;
}

static DecodeStatus decodeUImmLog2XLenOperand(MCInst &Inst, uint32_t Imm,
                                              int64_t Address,
                                              const MCDisassembler *Decoder) {
  assert(isUInt<6>(Imm) && "Invalid immediate");
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 329-344: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (!Decoder->getSubtargetInfo().hasFeature(RISCV::Feature64Bit) &&
      !isUInt<5>(Imm))
    return MCDisassembler::Fail;

  Inst.addOperand(MCOperand::createImm(Imm));
  return MCDisassembler::Success;
}

template <unsigned N>
static DecodeStatus decodeUImmNonZeroOperand(MCInst &Inst, uint32_t Imm,
                                             int64_t Address,
                                             const MCDisassembler *Decoder) {
  if (Imm == 0)
    return MCDisassembler::Fail;
  return decodeUImmOperand<N>(Inst, Imm, Address, Decoder);
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 345-361: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

static DecodeStatus
decodeUImmLog2XLenNonZeroOperand(MCInst &Inst, uint32_t Imm, int64_t Address,
                                 const MCDisassembler *Decoder) {
  if (Imm == 0)
    return MCDisassembler::Fail;
  return decodeUImmLog2XLenOperand(Inst, Imm, Address, Decoder);
}

template <unsigned N>
static DecodeStatus decodeUImmPlus1Operand(MCInst &Inst, uint32_t Imm,
                                           int64_t Address,
                                           const MCDisassembler *Decoder) {
  assert(isUInt<N>(Imm) && "Invalid immediate");
  Inst.addOperand(MCOperand::createImm(Imm + 1));
  return MCDisassembler::Success;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 362-379: Function implementation: MCOperand::createImm / 函数实现：MCOperand::createImm
```cpp

static DecodeStatus decodeImmZibiOperand(MCInst &Inst, uint32_t Imm,
                                         int64_t Address,
                                         const MCDisassembler *Decoder) {
  assert(isUInt<5>(Imm) && "Invalid immediate");
  Inst.addOperand(MCOperand::createImm(Imm ? Imm : -1LL));
  return MCDisassembler::Success;
}

template <unsigned N>
static DecodeStatus decodeSImmOperand(MCInst &Inst, uint32_t Imm,
                                      int64_t Address,
                                      const MCDisassembler *Decoder) {
  assert(isUInt<N>(Imm) && "Invalid immediate");
  // Sign-extend the number in the bottom N bits of Imm
  Inst.addOperand(MCOperand::createImm(SignExtend64<N>(Imm)));
  return MCDisassembler::Success;
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 380-400: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

template <unsigned N>
static DecodeStatus decodeSImmNonZeroOperand(MCInst &Inst, uint32_t Imm,
                                             int64_t Address,
                                             const MCDisassembler *Decoder) {
  if (Imm == 0)
    return MCDisassembler::Fail;
  return decodeSImmOperand<N>(Inst, Imm, Address, Decoder);
}

template <unsigned T, unsigned N>
static DecodeStatus decodeSImmOperandAndLslN(MCInst &Inst, uint32_t Imm,
                                             int64_t Address,
                                             const MCDisassembler *Decoder) {
  assert(isUInt<T - N + 1>(Imm) && "Invalid immediate");
  // Sign-extend the number in the bottom T bits of Imm after accounting for
  // the fact that the T bit immediate is stored in T-N bits (the LSB is
  // always zero)
  Inst.addOperand(MCOperand::createImm(SignExtend64<T>(Imm << N)));
  return MCDisassembler::Success;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 401-418: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

static DecodeStatus decodeCLUIImmOperand(MCInst &Inst, uint32_t Imm,
                                         int64_t Address,
                                         const MCDisassembler *Decoder) {
  assert(isUInt<6>(Imm) && "Invalid immediate");
  if (Imm == 0)
    return MCDisassembler::Fail;
  Imm = SignExtend64<6>(Imm) & 0xfffff;
  Inst.addOperand(MCOperand::createImm(Imm));
  return MCDisassembler::Success;
}

static DecodeStatus decodeFRMArg(MCInst &Inst, uint32_t Imm, int64_t Address,
                                 const MCDisassembler *Decoder) {
  assert(isUInt<3>(Imm) && "Invalid immediate");
  if (!llvm::RISCVFPRndMode::isValidRoundingMode(Imm))
    return MCDisassembler::Fail;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 419-432: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  Inst.addOperand(MCOperand::createImm(Imm));
  return MCDisassembler::Success;
}

static DecodeStatus decodeZcmpRlist(MCInst &Inst, uint32_t Imm,
                                    uint64_t Address,
                                    const MCDisassembler *Decoder) {
  bool IsRVE = Decoder->getSubtargetInfo().hasFeature(RISCV::FeatureStdExtE);
  if (Imm < RISCVZC::RA || (IsRVE && Imm >= RISCVZC::RA_S0_S2))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(Imm));
  return MCDisassembler::Success;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 433-449: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp
static DecodeStatus decodeXqccmpRlistS0(MCInst &Inst, uint32_t Imm,
                                        uint64_t Address,
                                        const MCDisassembler *Decoder) {
  if (Imm < RISCVZC::RA_S0)
    return MCDisassembler::Fail;
  return decodeZcmpRlist(Inst, Imm, Address, Decoder);
}

#include "RISCVGenDisassemblerTables.inc"

namespace {

struct DecoderListEntry {
  const uint8_t *Table;
  FeatureBitset ContainedFeatures;
  const char *Desc;
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 450-463: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
  bool haveContainedFeatures(const FeatureBitset &ActiveFeatures) const {
    return ContainedFeatures.none() ||
           (ContainedFeatures & ActiveFeatures).any();
  }
};

} // end anonymous namespace

static constexpr FeatureBitset XCVFeatureGroup = {
    RISCV::FeatureVendorXCVbitmanip, RISCV::FeatureVendorXCVelw,
    RISCV::FeatureVendorXCVmac,      RISCV::FeatureVendorXCVmem,
    RISCV::FeatureVendorXCValu,      RISCV::FeatureVendorXCVsimd,
    RISCV::FeatureVendorXCVbi};
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 464-478: Definitions and supporting logic / 定义与支撑逻辑
```cpp
static constexpr FeatureBitset XRivosFeatureGroup = {
    RISCV::FeatureVendorXRivosVizip,
};

static constexpr FeatureBitset XqciFeatureGroup = {
    RISCV::FeatureVendorXqcia,   RISCV::FeatureVendorXqciac,
    RISCV::FeatureVendorXqcibi,  RISCV::FeatureVendorXqcibm,
    RISCV::FeatureVendorXqcicli, RISCV::FeatureVendorXqcicm,
    RISCV::FeatureVendorXqcics,  RISCV::FeatureVendorXqcicsr,
    RISCV::FeatureVendorXqciint, RISCV::FeatureVendorXqciio,
    RISCV::FeatureVendorXqcilb,  RISCV::FeatureVendorXqcili,
    RISCV::FeatureVendorXqcilia, RISCV::FeatureVendorXqcilo,
    RISCV::FeatureVendorXqcilsm, RISCV::FeatureVendorXqcisim,
    RISCV::FeatureVendorXqcisls, RISCV::FeatureVendorXqcisync,
};
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 479-497: Definitions and supporting logic / 定义与支撑逻辑
```cpp

static constexpr FeatureBitset XSfVectorGroup = {
    RISCV::FeatureVendorXSfvcp,          RISCV::FeatureVendorXSfvqmaccdod,
    RISCV::FeatureVendorXSfvqmaccqoq,    RISCV::FeatureVendorXSfvfwmaccqqq,
    RISCV::FeatureVendorXSfvfnrclipxfqf, RISCV::FeatureVendorXSfmmbase,
    RISCV::FeatureVendorXSfvfexpa,       RISCV::FeatureVendorXSfvfexpa64e,
    RISCV::FeatureVendorXSfvfbfexp16e,   RISCV::FeatureVendorXSfvfexp16e,
    RISCV::FeatureVendorXSfvfexp32e};
static constexpr FeatureBitset XSfSystemGroup = {
    RISCV::FeatureVendorXSiFivecdiscarddlone,
    RISCV::FeatureVendorXSiFivecflushdlone,
};

static constexpr FeatureBitset XMIPSGroup = {
    RISCV::FeatureVendorXMIPSLSP,
    RISCV::FeatureVendorXMIPSCMov,
    RISCV::FeatureVendorXMIPSCBOP,
    RISCV::FeatureVendorXMIPSEXECTL,
};
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 498-512: Definitions and supporting logic / 定义与支撑逻辑
```cpp

static constexpr FeatureBitset XTHeadGroup = {
    RISCV::FeatureVendorXTHeadBa,      RISCV::FeatureVendorXTHeadBb,
    RISCV::FeatureVendorXTHeadBs,      RISCV::FeatureVendorXTHeadCondMov,
    RISCV::FeatureVendorXTHeadCmo,     RISCV::FeatureVendorXTHeadFMemIdx,
    RISCV::FeatureVendorXTHeadMac,     RISCV::FeatureVendorXTHeadMemIdx,
    RISCV::FeatureVendorXTHeadMemPair, RISCV::FeatureVendorXTHeadSync,
    RISCV::FeatureVendorXTHeadVdot};

static constexpr FeatureBitset XAndesGroup = {
    RISCV::FeatureVendorXAndesPerf,      RISCV::FeatureVendorXAndesBFHCvt,
    RISCV::FeatureVendorXAndesVBFHCvt,   RISCV::FeatureVendorXAndesVSIntH,
    RISCV::FeatureVendorXAndesVSIntLoad, RISCV::FeatureVendorXAndesVPackFPH,
    RISCV::FeatureVendorXAndesVDot};
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 513-538: Function implementation / 函数实现
```cpp
static constexpr FeatureBitset XSMTGroup = {RISCV::FeatureVendorXSMTVDot};

static constexpr FeatureBitset XAIFGroup = {RISCV::FeatureVendorXAIFET};

static constexpr DecoderListEntry DecoderList32[]{
    // Vendor Extensions
    {DecoderTableXCV32, XCVFeatureGroup, "CORE-V extensions"},
    {DecoderTableXRivos32, XRivosFeatureGroup, "Rivos"},
    {DecoderTableXqci32, XqciFeatureGroup, "Qualcomm uC Extensions"},
    {DecoderTableXVentana32,
     {RISCV::FeatureVendorXVentanaCondOps},
     "XVentanaCondOps"},
    {DecoderTableXTHead32, XTHeadGroup, "T-Head extensions"},
    {DecoderTableXSfvector32, XSfVectorGroup, "SiFive vector extensions"},
    {DecoderTableXSfsystem32, XSfSystemGroup, "SiFive system extensions"},
    {DecoderTableXSfcease32, {RISCV::FeatureVendorXSfcease}, "SiFive sf.cease"},
    {DecoderTableXMIPS32, XMIPSGroup, "Mips extensions"},
    {DecoderTableXAndes32, XAndesGroup, "Andes extensions"},
    {DecoderTableXSMT32, XSMTGroup, "SpacemiT extensions"},
    {DecoderTableXAIF32, XAIFGroup, "AI Foundry extensions"},
    // Standard Extensions
    {DecoderTable32, {}, "standard 32-bit instructions"},
    {DecoderTableRV32Only32, {}, "RV32-only standard 32-bit instructions"},
    {DecoderTableZfinx32, {}, "Zfinx (Float in Integer)"},
    {DecoderTableZdinxRV32Only32, {}, "RV32-only Zdinx (Double in Integer)"},
};
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 539-555: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp

namespace {
// Define bitwidths for various types used to instantiate the decoder.
template <> constexpr uint32_t InsnBitWidth<uint16_t> = 16;
template <> constexpr uint32_t InsnBitWidth<uint32_t> = 32;
// Use uint64_t to represent 48 bit instructions.
template <> constexpr uint32_t InsnBitWidth<uint64_t> = 48;
} // namespace

DecodeStatus RISCVDisassembler::getInstruction32(MCInst &MI, uint64_t &Size,
                                                 ArrayRef<uint8_t> Bytes,
                                                 uint64_t Address,
                                                 raw_ostream &CS) const {
  if (Bytes.size() < 4) {
    Size = 0;
    return MCDisassembler::Fail;
  }
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 556-569: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  Size = 4;

  uint32_t Insn = support::endian::read32le(Bytes.data());

  for (const DecoderListEntry &Entry : DecoderList32) {
    if (!Entry.haveContainedFeatures(STI.getFeatureBits()))
      continue;

    LLVM_DEBUG(dbgs() << "Trying " << Entry.Desc << " table:\n");
    DecodeStatus Result =
        decodeInstruction(Entry.Table, MI, Insn, Address, this, STI);
    if (Result == MCDisassembler::Fail)
      continue;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 570-592: Function implementation / 函数实现
```cpp
    return Result;
  }

  return MCDisassembler::Fail;
}

static constexpr DecoderListEntry DecoderList16[]{
    // Vendor Extensions
    {DecoderTableXqci16, XqciFeatureGroup, "Qualcomm uC 16-bit"},
    {DecoderTableXqccmp16,
     {RISCV::FeatureVendorXqccmp},
     "Xqccmp (Qualcomm 16-bit Push/Pop & Double Move Instructions)"},
    {DecoderTableXwchc16, {RISCV::FeatureVendorXwchc}, "WCH QingKe XW"},
    // Standard Extensions
    // DecoderTableZicfiss16 must be checked before DecoderTable16.
    {DecoderTableZicfiss16, {}, "Zicfiss (Shadow Stack 16-bit)"},
    {DecoderTable16, {}, "standard 16-bit instructions"},
    {DecoderTableRV32Only16, {}, "RV32-only 16-bit instructions"},
    // Zc* instructions incompatible with Zcf or Zcd
    {DecoderTableZcOverlap16,
     {},
     "ZcOverlap (16-bit Instructions overlapping with Zcf/Zcd)"},
};
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 593-609: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

DecodeStatus RISCVDisassembler::getInstruction16(MCInst &MI, uint64_t &Size,
                                                 ArrayRef<uint8_t> Bytes,
                                                 uint64_t Address,
                                                 raw_ostream &CS) const {
  if (Bytes.size() < 2) {
    Size = 0;
    return MCDisassembler::Fail;
  }
  Size = 2;

  uint16_t Insn = support::endian::read16le(Bytes.data());

  for (const DecoderListEntry &Entry : DecoderList16) {
    if (!Entry.haveContainedFeatures(STI.getFeatureBits()))
      continue;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 610-623: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    LLVM_DEBUG(dbgs() << "Trying " << Entry.Desc << " table:\n");
    DecodeStatus Result =
        decodeInstruction(Entry.Table, MI, Insn, Address, this, STI);
    if (Result != MCDisassembler::Fail)
      return Result;
  }

  return MCDisassembler::Fail;
}

static constexpr DecoderListEntry DecoderList48[]{
    {DecoderTableXqci48, XqciFeatureGroup, "Qualcomm uC 48bit"},
};
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 624-637: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
DecodeStatus RISCVDisassembler::getInstruction48(MCInst &MI, uint64_t &Size,
                                                 ArrayRef<uint8_t> Bytes,
                                                 uint64_t Address,
                                                 raw_ostream &CS) const {
  if (Bytes.size() < 6) {
    Size = 0;
    return MCDisassembler::Fail;
  }
  Size = 6;

  uint64_t Insn = 0;
  for (size_t i = Size; i-- != 0;)
    Insn += (static_cast<uint64_t>(Bytes[i]) << 8 * i);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 638-652: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  for (const DecoderListEntry &Entry : DecoderList48) {
    if (!Entry.haveContainedFeatures(STI.getFeatureBits()))
      continue;

    LLVM_DEBUG(dbgs() << "Trying " << Entry.Desc << " table:\n");
    DecodeStatus Result =
        decodeInstruction(Entry.Table, MI, Insn, Address, this, STI);
    if (Result == MCDisassembler::Fail)
      continue;

    return Result;
  }

  return MCDisassembler::Fail;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 653-667: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

DecodeStatus RISCVDisassembler::getInstruction(MCInst &MI, uint64_t &Size,
                                               ArrayRef<uint8_t> Bytes,
                                               uint64_t Address,
                                               raw_ostream &CS) const {
  CommentStream = &CS;
  // It's a 16 bit instruction if bit 0 and 1 are not 0b11.
  if ((Bytes[0] & 0b11) != 0b11)
    return getInstruction16(MI, Size, Bytes, Address, CS);

  // Try to decode as a 32-bit instruction first.
  DecodeStatus Result = getInstruction32(MI, Size, Bytes, Address, CS);
  if (Result != MCDisassembler::Fail)
    return Result;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 668-683: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // If bits [4:2] are 0b111 this might be a 48-bit or larger instruction,
  // otherwise assume it's an unknown 32-bit instruction.
  if ((Bytes[0] & 0b1'1100) != 0b1'1100) {
    Size = Bytes.size() >= 4 ? 4 : 0;
    return MCDisassembler::Fail;
  }

  // 48-bit instructions are encoded as 0bxx011111.
  if ((Bytes[0] & 0b11'1111) == 0b01'1111)
    return getInstruction48(MI, Size, Bytes, Address, CS);

  // 64-bit instructions are encoded as 0x0111111.
  if ((Bytes[0] & 0b111'1111) == 0b011'1111) {
    Size = Bytes.size() >= 8 ? 8 : 0;
    return MCDisassembler::Fail;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 684-699: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  // Remaining cases need to check a second byte.
  if (Bytes.size() < 2) {
    Size = 0;
    return MCDisassembler::Fail;
  }

  // 80-bit through 176-bit instructions are encoded as 0bxnnnxxxx_x1111111.
  // Where the number of bits is (80 + (nnn * 16)) for nnn != 0b111.
  unsigned nnn = (Bytes[1] >> 4) & 0b111;
  if (nnn != 0b111) {
    Size = 10 + (nnn * 2);
    if (Bytes.size() < Size)
      Size = 0;
    return MCDisassembler::Fail;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 700-704: Definitions and supporting logic / 定义与支撑逻辑
```cpp

  // Remaining encodings are reserved for > 176-bit instructions.
  Size = 0;
  return MCDisassembler::Fail;
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

## Key Concepts / 关键概念
- **Instruction decoding** / **指令解码**

## Dependencies / 依赖关系
- `MCTargetDesc/RISCVBaseInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `MCTargetDesc/RISCVMCTargetDesc.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `TargetInfo/RISCVTargetInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCContext.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCDecoder.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCDecoderOps.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCDisassembler/MCDisassembler.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCInst.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCInstrInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCRegisterInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCSubtargetInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/TargetRegistry.h` — Directly referenced by this file. / 该文件直接引用的依赖。
