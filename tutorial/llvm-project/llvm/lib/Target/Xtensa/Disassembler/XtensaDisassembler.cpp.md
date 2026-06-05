# XtensaDisassembler.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/Disassembler/XtensaDisassembler.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements the target-specific disassembler that decodes instruction bytes into MCInst objects.
  - **CN**: 实现目标相关的反汇编器，把指令字节解码为 MCInst 对象。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```cpp
//===-- XtensaDisassembler.cpp - Disassembler for Xtensa ------------------===//
//
//                     The LLVM Compiler Infrastructure
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 10-26
```cpp
//
// This file implements the XtensaDisassembler class.
//
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/XtensaMCTargetDesc.h"
#include "TargetInfo/XtensaTargetInfo.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCDecoder.h"
#include "llvm/MC/MCDecoderOps.h"
#include "llvm/MC/MCDisassembler/MCDisassembler.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Endian.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `MCTargetDesc/XtensaMCTargetDesc.h`, `TargetInfo/XtensaTargetInfo.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCDecoder.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MCTargetDesc/XtensaMCTargetDesc.h`, `TargetInfo/XtensaTargetInfo.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCDecoder.h`。

### Lines 27-35
```cpp
using namespace llvm;
using namespace llvm::MCD;

#define DEBUG_TYPE "Xtensa-disassembler"

using DecodeStatus = MCDisassembler::DecodeStatus;

namespace {

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 36-44
```cpp
class XtensaDisassembler : public MCDisassembler {
  bool IsLittleEndian;

public:
  XtensaDisassembler(const MCSubtargetInfo &STI, MCContext &Ctx, bool isLE)
      : MCDisassembler(STI, Ctx), IsLittleEndian(isLE) {}

  bool hasDensity() const { return STI.hasFeature(Xtensa::FeatureDensity); }

```
- **EN**: Introduces declarations for `XtensaDisassembler`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `XtensaDisassembler` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 45-56
```cpp
  DecodeStatus getInstruction(MCInst &Instr, uint64_t &Size,
                              ArrayRef<uint8_t> Bytes, uint64_t Address,
                              raw_ostream &CStream) const override;
};
} // end anonymous namespace

static MCDisassembler *createXtensaDisassembler(const Target &T,
                                                const MCSubtargetInfo &STI,
                                                MCContext &Ctx) {
  return new XtensaDisassembler(STI, Ctx, true);
}

```
- **EN**: Implements logic around `getInstruction`, `createXtensaDisassembler`, `XtensaDisassembler`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `getInstruction`, `createXtensaDisassembler`, `XtensaDisassembler` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 57-66
```cpp
extern "C" LLVM_EXTERNAL_VISIBILITY void LLVMInitializeXtensaDisassembler() {
  TargetRegistry::RegisterMCDisassembler(getTheXtensaTarget(),
                                         createXtensaDisassembler);
}

const MCPhysReg ARDecoderTable[] = {
    Xtensa::A0,  Xtensa::SP,  Xtensa::A2,  Xtensa::A3, Xtensa::A4,  Xtensa::A5,
    Xtensa::A6,  Xtensa::A7,  Xtensa::A8,  Xtensa::A9, Xtensa::A10, Xtensa::A11,
    Xtensa::A12, Xtensa::A13, Xtensa::A14, Xtensa::A15};

```
- **EN**: Implements logic around `RegisterMCDisassembler`.
- **CN**: 围绕 `RegisterMCDisassembler` 实现具体逻辑。

### Lines 67-77
```cpp
static DecodeStatus DecodeARRegisterClass(MCInst &Inst, uint64_t RegNo,
                                          uint64_t Address,
                                          const void *Decoder) {
  if (RegNo >= std::size(ARDecoderTable))
    return MCDisassembler::Fail;

  MCPhysReg Reg = ARDecoderTable[RegNo];
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `DecodeARRegisterClass`, `addOperand`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeARRegisterClass`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 78-88
```cpp
static DecodeStatus DecodeMRRegisterClass(MCInst &Inst, uint64_t RegNo,
                                          uint64_t Address,
                                          const void *Decoder) {
  if (RegNo > 3)
    return MCDisassembler::Fail;

  MCPhysReg Reg = Xtensa::M0 + RegNo;
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `DecodeMRRegisterClass`, `addOperand`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeMRRegisterClass`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 89-99
```cpp
static DecodeStatus DecodeMR01RegisterClass(MCInst &Inst, uint64_t RegNo,
                                            uint64_t Address,
                                            const void *Decoder) {
  if (RegNo > 1)
    return MCDisassembler::Fail;

  MCPhysReg Reg = Xtensa::M0 + RegNo;
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `DecodeMR01RegisterClass`, `addOperand`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeMR01RegisterClass`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 100-110
```cpp
static DecodeStatus DecodeMR23RegisterClass(MCInst &Inst, uint64_t RegNo,
                                            uint64_t Address,
                                            const void *Decoder) {
  if (RegNo > 1)
    return MCDisassembler::Fail;

  MCPhysReg Reg = Xtensa::M2 + RegNo;
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `DecodeMR23RegisterClass`, `addOperand`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeMR23RegisterClass`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 111-121
```cpp
static DecodeStatus DecodeFPRRegisterClass(MCInst &Inst, uint64_t RegNo,
                                           uint64_t Address,
                                           const void *Decoder) {
  if (RegNo > 15)
    return MCDisassembler::Fail;

  MCPhysReg Reg = Xtensa::F0 + RegNo;
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `DecodeFPRRegisterClass`, `addOperand`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeFPRRegisterClass`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 122-131
```cpp
static DecodeStatus DecodeURRegisterClass(MCInst &Inst, uint64_t RegNo,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  if (RegNo > 255)
    return MCDisassembler::Fail;

  Xtensa::RegisterAccessType RAType = Inst.getOpcode() == Xtensa::WUR
                                          ? Xtensa::REGISTER_WRITE
                                          : Xtensa::REGISTER_READ;

```
- **EN**: Implements logic around `DecodeURRegisterClass`, `getOpcode`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeURRegisterClass`, `getOpcode` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 132-143
```cpp
  const XtensaDisassembler *Dis =
      static_cast<const XtensaDisassembler *>(Decoder);
  const MCRegisterInfo *MRI = Dis->getContext().getRegisterInfo();
  MCPhysReg Reg = Xtensa::getUserRegister(RegNo, *MRI);
  if (!Xtensa::checkRegister(Reg, Decoder->getSubtargetInfo().getFeatureBits(),
                             RAType))
    return MCDisassembler::Fail;

  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `getContext`, `getUserRegister`, `addOperand`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getContext`, `getUserRegister`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 144-161
```cpp
struct DecodeRegister {
  MCPhysReg Reg;
  uint32_t RegNo;
};

const DecodeRegister SRDecoderTable[] = {
    {Xtensa::LBEG, 0},         {Xtensa::LEND, 1},
    {Xtensa::LCOUNT, 2},       {Xtensa::SAR, 3},
    {Xtensa::BREG, 4},         {Xtensa::LITBASE, 5},
    {Xtensa::SCOMPARE1, 12},   {Xtensa::ACCLO, 16},
    {Xtensa::ACCHI, 17},       {Xtensa::M0, 32},
    {Xtensa::M1, 33},          {Xtensa::M2, 34},
    {Xtensa::M3, 35},          {Xtensa::WINDOWBASE, 72},
    {Xtensa::WINDOWSTART, 73}, {Xtensa::IBREAKENABLE, 96},
    {Xtensa::MEMCTL, 97},      {Xtensa::ATOMCTL, 99},
    {Xtensa::DDR, 104},        {Xtensa::IBREAKA0, 128},
    {Xtensa::IBREAKA1, 129},   {Xtensa::DBREAKA0, 144},
    {Xtensa::DBREAKA1, 145},   {Xtensa::DBREAKC0, 160},
```
- **EN**: Introduces declarations for `DecodeRegister`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `DecodeRegister` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 162-179
```cpp
    {Xtensa::DBREAKC1, 161},   {Xtensa::CONFIGID0, 176},
    {Xtensa::EPC1, 177},       {Xtensa::EPC2, 178},
    {Xtensa::EPC3, 179},       {Xtensa::EPC4, 180},
    {Xtensa::EPC5, 181},       {Xtensa::EPC6, 182},
    {Xtensa::EPC7, 183},       {Xtensa::DEPC, 192},
    {Xtensa::EPS2, 194},       {Xtensa::EPS3, 195},
    {Xtensa::EPS4, 196},       {Xtensa::EPS5, 197},
    {Xtensa::EPS6, 198},       {Xtensa::EPS7, 199},
    {Xtensa::CONFIGID1, 208},  {Xtensa::EXCSAVE1, 209},
    {Xtensa::EXCSAVE2, 210},   {Xtensa::EXCSAVE3, 211},
    {Xtensa::EXCSAVE4, 212},   {Xtensa::EXCSAVE5, 213},
    {Xtensa::EXCSAVE6, 214},   {Xtensa::EXCSAVE7, 215},
    {Xtensa::CPENABLE, 224},   {Xtensa::INTERRUPT, 226},
    {Xtensa::INTCLEAR, 227},   {Xtensa::INTENABLE, 228},
    {Xtensa::PS, 230},         {Xtensa::VECBASE, 231},
    {Xtensa::EXCCAUSE, 232},   {Xtensa::DEBUGCAUSE, 233},
    {Xtensa::CCOUNT, 234},     {Xtensa::PRID, 235},
    {Xtensa::ICOUNT, 236},     {Xtensa::ICOUNTLEVEL, 237},
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 180-190
```cpp
    {Xtensa::EXCVADDR, 238},   {Xtensa::CCOMPARE0, 240},
    {Xtensa::CCOMPARE1, 241},  {Xtensa::CCOMPARE2, 242},
    {Xtensa::MISC0, 244},      {Xtensa::MISC1, 245},
    {Xtensa::MISC2, 246},      {Xtensa::MISC3, 247}};

static DecodeStatus DecodeSRRegisterClass(MCInst &Inst, uint64_t RegNo,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  if (RegNo > 255)
    return MCDisassembler::Fail;

```
- **EN**: Implements logic around `DecodeSRRegisterClass`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeSRRegisterClass` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 191-200
```cpp
  Xtensa::RegisterAccessType RAType =
      Inst.getOpcode() == Xtensa::WSR
          ? Xtensa::REGISTER_WRITE
          : (Inst.getOpcode() == Xtensa::RSR ? Xtensa::REGISTER_READ
                                             : Xtensa::REGISTER_EXCHANGE);

  for (unsigned i = 0; i < std::size(SRDecoderTable); i++) {
    if (SRDecoderTable[i].RegNo == RegNo) {
      MCPhysReg Reg = SRDecoderTable[i].Reg;

```
- **EN**: Implements logic around `getOpcode`; this block applies conditional target rules.
- **CN**: 围绕 `getOpcode` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 201-210
```cpp
      // Handle special case. The INTERRUPT/INTSET registers use the same
      // encoding, but INTERRUPT used for read and INTSET for write.
      if (Reg == Xtensa::INTERRUPT && RAType == Xtensa::REGISTER_WRITE) {
        Reg = Xtensa::INTSET;
      }

      if (!Xtensa::checkRegister(
              Reg, Decoder->getSubtargetInfo().getFeatureBits(), RAType))
        return MCDisassembler::Fail;

```
- **EN**: Implements logic around `getSubtargetInfo`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getSubtargetInfo` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 211-224
```cpp
      Inst.addOperand(MCOperand::createReg(Reg));
      return MCDisassembler::Success;
    }
  }

  return MCDisassembler::Fail;
}

static DecodeStatus DecodeBRRegisterClass(MCInst &Inst, uint64_t RegNo,
                                          uint64_t Address,
                                          const void *Decoder) {
  if (RegNo > 15)
    return MCDisassembler::Fail;

```
- **EN**: Implements logic around `addOperand`, `DecodeBRRegisterClass`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `addOperand`, `DecodeBRRegisterClass` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 225-236
```cpp
  MCPhysReg Reg = Xtensa::B0 + RegNo;
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

static bool tryAddingSymbolicOperand(int64_t Value, bool isBranch,
                                     uint64_t Address, uint64_t Offset,
                                     uint64_t InstSize, MCInst &MI,
                                     const void *Decoder) {
  const MCDisassembler *Dis = static_cast<const MCDisassembler *>(Decoder);
  return Dis->tryAddingSymbolicOperand(MI, Value, Address, isBranch, Offset,
                                       /*OpSize=*/0, InstSize);
```
- **EN**: Implements logic around `addOperand`, `tryAddingSymbolicOperand`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `addOperand`, `tryAddingSymbolicOperand` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 237-246
```cpp
}

static DecodeStatus decodeCallOperand(MCInst &Inst, uint64_t Imm,
                                      int64_t Address, const void *Decoder) {
  assert(isUInt<18>(Imm) && "Invalid immediate");
  Inst.addOperand(
      MCOperand::createImm(SignExtend64<20>(Imm << 2) + (Address & 0x3)));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `decodeCallOperand`, `assert`, `addOperand`, `createImm`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `decodeCallOperand`, `assert`, `addOperand`, `createImm` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 247-264
```cpp
static DecodeStatus decodeJumpOperand(MCInst &Inst, uint64_t Imm,
                                      int64_t Address, const void *Decoder) {
  assert(isUInt<18>(Imm) && "Invalid immediate");
  Inst.addOperand(MCOperand::createImm(SignExtend64<18>(Imm)));
  return MCDisassembler::Success;
}

static DecodeStatus decodeBranchOperand(MCInst &Inst, uint64_t Imm,
                                        int64_t Address, const void *Decoder) {
  switch (Inst.getOpcode()) {
  case Xtensa::BEQZ:
  case Xtensa::BGEZ:
  case Xtensa::BLTZ:
  case Xtensa::BNEZ:
    assert(isUInt<12>(Imm) && "Invalid immediate");
    if (!tryAddingSymbolicOperand(SignExtend64<12>(Imm) + 4 + Address, true,
                                  Address, 0, 3, Inst, Decoder))
      Inst.addOperand(MCOperand::createImm(SignExtend64<12>(Imm)));
```
- **EN**: Implements logic around `decodeJumpOperand`, `assert`, `addOperand`, `decodeBranchOperand`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `decodeJumpOperand`, `assert`, `addOperand`, `decodeBranchOperand` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 265-274
```cpp
    break;
  default:
    assert(isUInt<8>(Imm) && "Invalid immediate");
    if (!tryAddingSymbolicOperand(SignExtend64<8>(Imm) + 4 + Address, true,
                                  Address, 0, 3, Inst, Decoder))
      Inst.addOperand(MCOperand::createImm(SignExtend64<8>(Imm)));
  }
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `assert`, `addOperand`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `assert`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 275-284
```cpp
static DecodeStatus decodeLoopOperand(MCInst &Inst, uint64_t Imm,
                                      int64_t Address, const void *Decoder) {

  assert(isUInt<8>(Imm) && "Invalid immediate");
  if (!tryAddingSymbolicOperand(Imm + 4 + Address, true, Address, 0, 3, Inst,
                                Decoder))
    Inst.addOperand(MCOperand::createImm(Imm));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `decodeLoopOperand`, `assert`, `addOperand`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `decodeLoopOperand`, `assert`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 285-293
```cpp
static DecodeStatus decodeL32ROperand(MCInst &Inst, uint64_t Imm,
                                      int64_t Address, const void *Decoder) {

  assert(isUInt<16>(Imm) && "Invalid immediate");
  Inst.addOperand(MCOperand::createImm(
      SignExtend64<17>((Imm << 2) + 0x40000 + (Address & 0x3))));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `decodeL32ROperand`, `assert`, `addOperand`, `SignExtend64<17>`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `decodeL32ROperand`, `assert`, `addOperand`, `SignExtend64<17>` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 294-308
```cpp
static DecodeStatus decodeImm8Operand(MCInst &Inst, uint64_t Imm,
                                      int64_t Address, const void *Decoder) {
  assert(isUInt<8>(Imm) && "Invalid immediate");
  Inst.addOperand(MCOperand::createImm(SignExtend64<8>(Imm)));
  return MCDisassembler::Success;
}

static DecodeStatus decodeImm8_sh8Operand(MCInst &Inst, uint64_t Imm,
                                          int64_t Address,
                                          const void *Decoder) {
  assert(isUInt<8>(Imm) && "Invalid immediate");
  Inst.addOperand(MCOperand::createImm(SignExtend64<16>(Imm << 8)));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `decodeImm8Operand`, `assert`, `addOperand`, `decodeImm8_sh8Operand`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `decodeImm8Operand`, `assert`, `addOperand`, `decodeImm8_sh8Operand` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 309-322
```cpp
static DecodeStatus decodeImm12Operand(MCInst &Inst, uint64_t Imm,
                                       int64_t Address, const void *Decoder) {
  assert(isUInt<12>(Imm) && "Invalid immediate");
  Inst.addOperand(MCOperand::createImm(SignExtend64<12>(Imm)));
  return MCDisassembler::Success;
}

static DecodeStatus decodeUimm4Operand(MCInst &Inst, uint64_t Imm,
                                       int64_t Address, const void *Decoder) {
  assert(isUInt<4>(Imm) && "Invalid immediate");
  Inst.addOperand(MCOperand::createImm(Imm));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `decodeImm12Operand`, `assert`, `addOperand`, `decodeUimm4Operand`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `decodeImm12Operand`, `assert`, `addOperand`, `decodeUimm4Operand` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 323-336
```cpp
static DecodeStatus decodeUimm5Operand(MCInst &Inst, uint64_t Imm,
                                       int64_t Address, const void *Decoder) {
  assert(isUInt<5>(Imm) && "Invalid immediate");
  Inst.addOperand(MCOperand::createImm(Imm));
  return MCDisassembler::Success;
}

static DecodeStatus decodeImm1_16Operand(MCInst &Inst, uint64_t Imm,
                                         int64_t Address, const void *Decoder) {
  assert(isUInt<4>(Imm) && "Invalid immediate");
  Inst.addOperand(MCOperand::createImm(Imm + 1));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `decodeUimm5Operand`, `assert`, `addOperand`, `decodeImm1_16Operand`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `decodeUimm5Operand`, `assert`, `addOperand`, `decodeImm1_16Operand` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 337-347
```cpp
static DecodeStatus decodeImm1n_15Operand(MCInst &Inst, uint64_t Imm,
                                          int64_t Address,
                                          const void *Decoder) {
  assert(isUInt<4>(Imm) && "Invalid immediate");
  if (!Imm)
    Inst.addOperand(MCOperand::createImm(-1));
  else
    Inst.addOperand(MCOperand::createImm(Imm));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `decodeImm1n_15Operand`, `assert`, `addOperand`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `decodeImm1n_15Operand`, `assert`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 348-358
```cpp
static DecodeStatus decodeImm32n_95Operand(MCInst &Inst, uint64_t Imm,
                                           int64_t Address,
                                           const void *Decoder) {
  assert(isUInt<7>(Imm) && "Invalid immediate");
  if ((Imm & 0x60) == 0x60)
    Inst.addOperand(MCOperand::createImm((~0x1f) | Imm));
  else
    Inst.addOperand(MCOperand::createImm(Imm));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `decodeImm32n_95Operand`, `assert`, `addOperand`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `decodeImm32n_95Operand`, `assert`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 359-373
```cpp
static DecodeStatus decodeImm8n_7Operand(MCInst &Inst, uint64_t Imm,
                                         int64_t Address, const void *Decoder) {
  assert(isUInt<4>(Imm) && "Invalid immediate");
  Inst.addOperand(MCOperand::createImm(Imm > 7 ? Imm - 16 : Imm));
  return MCDisassembler::Success;
}

static DecodeStatus decodeImm64n_4nOperand(MCInst &Inst, uint64_t Imm,
                                           int64_t Address,
                                           const void *Decoder) {
  assert(isUInt<6>(Imm) && ((Imm & 0x3) == 0) && "Invalid immediate");
  Inst.addOperand(MCOperand::createImm((~0x3f) | (Imm)));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `decodeImm8n_7Operand`, `assert`, `addOperand`, `decodeImm64n_4nOperand`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `decodeImm8n_7Operand`, `assert`, `addOperand`, `decodeImm64n_4nOperand` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 374-389
```cpp
static DecodeStatus decodeEntry_Imm12OpValue(MCInst &Inst, uint64_t Imm,
                                             int64_t Address,
                                             const void *Decoder) {
  assert(isUInt<15>(Imm) && ((Imm & 0x7) == 0) && "Invalid immediate");
  Inst.addOperand(MCOperand::createImm(Imm));
  return MCDisassembler::Success;
}

static DecodeStatus decodeShimm1_31Operand(MCInst &Inst, uint64_t Imm,
                                           int64_t Address,
                                           const void *Decoder) {
  assert(isUInt<5>(Imm) && "Invalid immediate");
  Inst.addOperand(MCOperand::createImm(32 - Imm));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `decodeEntry_Imm12OpValue`, `assert`, `addOperand`, `decodeShimm1_31Operand`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `decodeEntry_Imm12OpValue`, `assert`, `addOperand`, `decodeShimm1_31Operand` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 390-399
```cpp
static int64_t TableB4const[16] = {-1, 1,  2,  3,  4,  5,  6,   7,
                                   8,  10, 12, 16, 32, 64, 128, 256};
static DecodeStatus decodeB4constOperand(MCInst &Inst, uint64_t Imm,
                                         int64_t Address, const void *Decoder) {
  assert(isUInt<4>(Imm) && "Invalid immediate");

  Inst.addOperand(MCOperand::createImm(TableB4const[Imm]));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `decodeB4constOperand`, `assert`, `addOperand`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `decodeB4constOperand`, `assert`, `addOperand` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 400-410
```cpp
static int64_t TableB4constu[16] = {32768, 65536, 2,  3,  4,  5,  6,   7,
                                    8,     10,    12, 16, 32, 64, 128, 256};
static DecodeStatus decodeB4constuOperand(MCInst &Inst, uint64_t Imm,
                                          int64_t Address,
                                          const void *Decoder) {
  assert(isUInt<4>(Imm) && "Invalid immediate");

  Inst.addOperand(MCOperand::createImm(TableB4constu[Imm]));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `decodeB4constuOperand`, `assert`, `addOperand`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `decodeB4constuOperand`, `assert`, `addOperand` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 411-425
```cpp
static DecodeStatus decodeImm7_22Operand(MCInst &Inst, uint64_t Imm,
                                         int64_t Address, const void *Decoder) {
  assert(isUInt<4>(Imm) && "Invalid immediate");
  Inst.addOperand(MCOperand::createImm(Imm + 7));
  return MCDisassembler::Success;
}

static DecodeStatus decodeMem8Operand(MCInst &Inst, uint64_t Imm,
                                      int64_t Address, const void *Decoder) {
  assert(isUInt<12>(Imm) && "Invalid immediate");
  DecodeARRegisterClass(Inst, Imm & 0xf, Address, Decoder);
  Inst.addOperand(MCOperand::createImm((Imm >> 4) & 0xff));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `decodeImm7_22Operand`, `assert`, `addOperand`, `decodeMem8Operand`, ...; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `decodeImm7_22Operand`, `assert`, `addOperand`, `decodeMem8Operand`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 426-441
```cpp
static DecodeStatus decodeMem16Operand(MCInst &Inst, uint64_t Imm,
                                       int64_t Address, const void *Decoder) {
  assert(isUInt<12>(Imm) && "Invalid immediate");
  DecodeARRegisterClass(Inst, Imm & 0xf, Address, Decoder);
  Inst.addOperand(MCOperand::createImm((Imm >> 3) & 0x1fe));
  return MCDisassembler::Success;
}

static DecodeStatus decodeMem32Operand(MCInst &Inst, uint64_t Imm,
                                       int64_t Address, const void *Decoder) {
  assert(isUInt<12>(Imm) && "Invalid immediate");
  DecodeARRegisterClass(Inst, Imm & 0xf, Address, Decoder);
  Inst.addOperand(MCOperand::createImm((Imm >> 2) & 0x3fc));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `decodeMem16Operand`, `assert`, `DecodeARRegisterClass`, `addOperand`, ...; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `decodeMem16Operand`, `assert`, `DecodeARRegisterClass`, `addOperand`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 442-450
```cpp
static DecodeStatus decodeMem32nOperand(MCInst &Inst, uint64_t Imm,
                                        int64_t Address, const void *Decoder) {
  assert(isUInt<8>(Imm) && "Invalid immediate");
  DecodeARRegisterClass(Inst, Imm & 0xf, Address, Decoder);
  Inst.addOperand(MCOperand::createImm((Imm >> 2) & 0x3c));
  return MCDisassembler::Success;
}

/// Read two bytes from the ArrayRef and return 16 bit data sorted
```
- **EN**: Implements logic around `decodeMem32nOperand`, `assert`, `DecodeARRegisterClass`, `addOperand`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `decodeMem32nOperand`, `assert`, `DecodeARRegisterClass`, `addOperand` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 451-460
```cpp
/// according to the given endianness.
static DecodeStatus readInstruction16(ArrayRef<uint8_t> Bytes, uint64_t Address,
                                      uint64_t &Size, uint64_t &Insn,
                                      bool IsLittleEndian) {
  // We want to read exactly 2 Bytes of data.
  if (Bytes.size() < 2) {
    Size = 0;
    return MCDisassembler::Fail;
  }

```
- **EN**: Implements logic around `readInstruction16`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `readInstruction16` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 461-469
```cpp
  if (!IsLittleEndian) {
    report_fatal_error("Big-endian mode currently is not supported!");
  } else {
    Insn = (Bytes[1] << 8) | Bytes[0];
  }

  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `report_fatal_error`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `report_fatal_error` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 470-479
```cpp
/// Read three bytes from the ArrayRef and return 24 bit data
static DecodeStatus readInstruction24(ArrayRef<uint8_t> Bytes, uint64_t Address,
                                      uint64_t &Size, uint64_t &Insn,
                                      bool IsLittleEndian) {
  // We want to read exactly 3 Bytes of data.
  if (Bytes.size() < 3) {
    Size = 0;
    return MCDisassembler::Fail;
  }

```
- **EN**: Implements logic around `readInstruction24`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `readInstruction24` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 480-488
```cpp
  if (!IsLittleEndian) {
    report_fatal_error("Big-endian mode currently is not supported!");
  } else {
    Insn = (Bytes[2] << 16) | (Bytes[1] << 8) | (Bytes[0] << 0);
  }

  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `report_fatal_error`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `report_fatal_error` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 489-497
```cpp
#include "XtensaGenDisassemblerTables.inc"

DecodeStatus XtensaDisassembler::getInstruction(MCInst &MI, uint64_t &Size,
                                                ArrayRef<uint8_t> Bytes,
                                                uint64_t Address,
                                                raw_ostream &CS) const {
  uint64_t Insn;
  DecodeStatus Result;

```
- **EN**: Pulls in the headers needed for this implementation, including `XtensaGenDisassemblerTables.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `XtensaGenDisassemblerTables.inc`。

### Lines 498-510
```cpp
  // Parse 16-bit instructions
  if (hasDensity()) {
    Result = readInstruction16(Bytes, Address, Size, Insn, IsLittleEndian);
    if (Result == MCDisassembler::Fail)
      return MCDisassembler::Fail;
    LLVM_DEBUG(dbgs() << "Trying Xtensa 16-bit instruction table :\n");
    Result = decodeInstruction(DecoderTable16, MI, Insn, Address, this, STI);
    if (Result != MCDisassembler::Fail) {
      Size = 2;
      return Result;
    }
  }

```
- **EN**: Implements logic around `readInstruction16`, `decodeInstruction`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `readInstruction16`, `decodeInstruction` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 511-522
```cpp
  // Parse Core 24-bit instructions
  Result = readInstruction24(Bytes, Address, Size, Insn, IsLittleEndian);
  if (Result == MCDisassembler::Fail)
    return MCDisassembler::Fail;
  LLVM_DEBUG(dbgs() << "Trying Xtensa 24-bit instruction table :\n");
  Result = decodeInstruction(DecoderTable24, MI, Insn, Address, this, STI);
  if (Result != MCDisassembler::Fail) {
    Size = 3;
    return Result;
  }
  return Result;
}
```
- **EN**: Implements logic around `readInstruction24`, `decodeInstruction`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `readInstruction24`, `decodeInstruction` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

## Key Concepts / 关键概念

- **Disassembly / 反汇编**:
  - **EN**: Decodes instruction bytes into MCInst objects
  - **CN**: 将指令字节解码为 MCInst

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MCTargetDesc/XtensaMCTargetDesc.h`, `TargetInfo/XtensaTargetInfo.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCDecoder.h`, `llvm/MC/MCDecoderOps.h`, `llvm/MC/MCDisassembler/MCDisassembler.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCRegisterInfo.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/TargetRegistry.h`, `llvm/Support/Endian.h`, `XtensaGenDisassemblerTables.inc`
- **LLVM subsystems / LLVM 子系统**: MC, Support
