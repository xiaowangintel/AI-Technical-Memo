# VEDisassembler.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/Disassembler/VEDisassembler.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements the target-specific disassembler that decodes instruction bytes into MCInst objects.
  - **CN**: 实现目标相关的反汇编器，把指令字节解码为 MCInst 对象。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- VEDisassembler.cpp - Disassembler for VE -----------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file is part of the VE Disassembler.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 12-24
```cpp

#include "MCTargetDesc/VEMCTargetDesc.h"
#include "TargetInfo/VETargetInfo.h"
#include "VE.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCDecoder.h"
#include "llvm/MC/MCDecoderOps.h"
#include "llvm/MC/MCDisassembler/MCDisassembler.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Compiler.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `MCTargetDesc/VEMCTargetDesc.h`, `TargetInfo/VETargetInfo.h`, `VE.h`, `llvm/MC/MCAsmInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MCTargetDesc/VEMCTargetDesc.h`, `TargetInfo/VETargetInfo.h`, `VE.h`, `llvm/MC/MCAsmInfo.h`。

### Lines 25-33
```cpp
using namespace llvm;
using namespace llvm::MCD;

#define DEBUG_TYPE "ve-disassembler"

typedef MCDisassembler::DecodeStatus DecodeStatus;

namespace {

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 34-46
```cpp
/// A disassembler class for VE.
class VEDisassembler : public MCDisassembler {
public:
  VEDisassembler(const MCSubtargetInfo &STI, MCContext &Ctx)
      : MCDisassembler(STI, Ctx) {}
  ~VEDisassembler() override = default;

  DecodeStatus getInstruction(MCInst &Instr, uint64_t &Size,
                              ArrayRef<uint8_t> Bytes, uint64_t Address,
                              raw_ostream &CStream) const override;
};
} // namespace

```
- **EN**: Introduces declarations for `for`, `VEDisassembler`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `for`, `VEDisassembler` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 47-59
```cpp
static MCDisassembler *createVEDisassembler(const Target &T,
                                            const MCSubtargetInfo &STI,
                                            MCContext &Ctx) {
  return new VEDisassembler(STI, Ctx);
}

extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
LLVMInitializeVEDisassembler() {
  // Register the disassembler.
  TargetRegistry::RegisterMCDisassembler(getTheVETarget(),
                                         createVEDisassembler);
}

```
- **EN**: Implements logic around `createVEDisassembler`, `VEDisassembler`, `RegisterMCDisassembler`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `createVEDisassembler`, `VEDisassembler`, `RegisterMCDisassembler` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 60-72
```cpp
// clang-format off
static const unsigned I32RegDecoderTable[] = {
    VE::SW0,  VE::SW1,  VE::SW2,  VE::SW3,  VE::SW4,  VE::SW5,  VE::SW6,
    VE::SW7,  VE::SW8,  VE::SW9,  VE::SW10, VE::SW11, VE::SW12, VE::SW13,
    VE::SW14, VE::SW15, VE::SW16, VE::SW17, VE::SW18, VE::SW19, VE::SW20,
    VE::SW21, VE::SW22, VE::SW23, VE::SW24, VE::SW25, VE::SW26, VE::SW27,
    VE::SW28, VE::SW29, VE::SW30, VE::SW31, VE::SW32, VE::SW33, VE::SW34,
    VE::SW35, VE::SW36, VE::SW37, VE::SW38, VE::SW39, VE::SW40, VE::SW41,
    VE::SW42, VE::SW43, VE::SW44, VE::SW45, VE::SW46, VE::SW47, VE::SW48,
    VE::SW49, VE::SW50, VE::SW51, VE::SW52, VE::SW53, VE::SW54, VE::SW55,
    VE::SW56, VE::SW57, VE::SW58, VE::SW59, VE::SW60, VE::SW61, VE::SW62,
    VE::SW63};

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 73-84
```cpp
static const unsigned I64RegDecoderTable[] = {
    VE::SX0,  VE::SX1,  VE::SX2,  VE::SX3,  VE::SX4,  VE::SX5,  VE::SX6,
    VE::SX7,  VE::SX8,  VE::SX9,  VE::SX10, VE::SX11, VE::SX12, VE::SX13,
    VE::SX14, VE::SX15, VE::SX16, VE::SX17, VE::SX18, VE::SX19, VE::SX20,
    VE::SX21, VE::SX22, VE::SX23, VE::SX24, VE::SX25, VE::SX26, VE::SX27,
    VE::SX28, VE::SX29, VE::SX30, VE::SX31, VE::SX32, VE::SX33, VE::SX34,
    VE::SX35, VE::SX36, VE::SX37, VE::SX38, VE::SX39, VE::SX40, VE::SX41,
    VE::SX42, VE::SX43, VE::SX44, VE::SX45, VE::SX46, VE::SX47, VE::SX48,
    VE::SX49, VE::SX50, VE::SX51, VE::SX52, VE::SX53, VE::SX54, VE::SX55,
    VE::SX56, VE::SX57, VE::SX58, VE::SX59, VE::SX60, VE::SX61, VE::SX62,
    VE::SX63};

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 85-96
```cpp
static const unsigned F32RegDecoderTable[] = {
    VE::SF0,  VE::SF1,  VE::SF2,  VE::SF3,  VE::SF4,  VE::SF5,  VE::SF6,
    VE::SF7,  VE::SF8,  VE::SF9,  VE::SF10, VE::SF11, VE::SF12, VE::SF13,
    VE::SF14, VE::SF15, VE::SF16, VE::SF17, VE::SF18, VE::SF19, VE::SF20,
    VE::SF21, VE::SF22, VE::SF23, VE::SF24, VE::SF25, VE::SF26, VE::SF27,
    VE::SF28, VE::SF29, VE::SF30, VE::SF31, VE::SF32, VE::SF33, VE::SF34,
    VE::SF35, VE::SF36, VE::SF37, VE::SF38, VE::SF39, VE::SF40, VE::SF41,
    VE::SF42, VE::SF43, VE::SF44, VE::SF45, VE::SF46, VE::SF47, VE::SF48,
    VE::SF49, VE::SF50, VE::SF51, VE::SF52, VE::SF53, VE::SF54, VE::SF55,
    VE::SF56, VE::SF57, VE::SF58, VE::SF59, VE::SF60, VE::SF61, VE::SF62,
    VE::SF63};

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 97-112
```cpp
static const unsigned F128RegDecoderTable[] = {
    VE::Q0,  VE::Q1,  VE::Q2,  VE::Q3,  VE::Q4,  VE::Q5,  VE::Q6,  VE::Q7,
    VE::Q8,  VE::Q9,  VE::Q10, VE::Q11, VE::Q12, VE::Q13, VE::Q14, VE::Q15,
    VE::Q16, VE::Q17, VE::Q18, VE::Q19, VE::Q20, VE::Q21, VE::Q22, VE::Q23,
    VE::Q24, VE::Q25, VE::Q26, VE::Q27, VE::Q28, VE::Q29, VE::Q30, VE::Q31};

static const unsigned V64RegDecoderTable[] = {
    VE::V0,  VE::V1,  VE::V2,  VE::V3,  VE::V4,  VE::V5,  VE::V6,  VE::V7,
    VE::V8,  VE::V9,  VE::V10, VE::V11, VE::V12, VE::V13, VE::V14, VE::V15,
    VE::V16, VE::V17, VE::V18, VE::V19, VE::V20, VE::V21, VE::V22, VE::V23,
    VE::V24, VE::V25, VE::V26, VE::V27, VE::V28, VE::V29, VE::V30, VE::V31,
    VE::V32, VE::V33, VE::V34, VE::V35, VE::V36, VE::V37, VE::V38, VE::V39,
    VE::V40, VE::V41, VE::V42, VE::V43, VE::V44, VE::V45, VE::V46, VE::V47,
    VE::V48, VE::V49, VE::V50, VE::V51, VE::V52, VE::V53, VE::V54, VE::V55,
    VE::V56, VE::V57, VE::V58, VE::V59, VE::V60, VE::V61, VE::V62, VE::V63};

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 113-121
```cpp
static const unsigned VMRegDecoderTable[] = {
    VE::VM0,  VE::VM1,  VE::VM2,  VE::VM3, VE::VM4,  VE::VM5,
    VE::VM6,  VE::VM7,  VE::VM8,  VE::VM9, VE::VM10, VE::VM11,
    VE::VM12, VE::VM13, VE::VM14, VE::VM15};

static const unsigned VM512RegDecoderTable[] = {VE::VMP0, VE::VMP1, VE::VMP2,
                                                VE::VMP3, VE::VMP4, VE::VMP5,
                                                VE::VMP6, VE::VMP7};

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 122-132
```cpp
static const unsigned MiscRegDecoderTable[] = {
    VE::USRCC,      VE::PSW,        VE::SAR,        VE::NoRegister,
    VE::NoRegister, VE::NoRegister, VE::NoRegister, VE::PMMR,
    VE::PMCR0,      VE::PMCR1,      VE::PMCR2,      VE::PMCR3,
    VE::NoRegister, VE::NoRegister, VE::NoRegister, VE::NoRegister,
    VE::PMC0,       VE::PMC1,       VE::PMC2,       VE::PMC3,
    VE::PMC4,       VE::PMC5,       VE::PMC6,       VE::PMC7,
    VE::PMC8,       VE::PMC9,       VE::PMC10,      VE::PMC11,
    VE::PMC12,      VE::PMC13,      VE::PMC14};
// clang-format on

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 133-142
```cpp
static DecodeStatus DecodeI32RegisterClass(MCInst &Inst, unsigned RegNo,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  if (RegNo > 63)
    return MCDisassembler::Fail;
  unsigned Reg = I32RegDecoderTable[RegNo];
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `DecodeI32RegisterClass`, `addOperand`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeI32RegisterClass`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 143-152
```cpp
static DecodeStatus DecodeI64RegisterClass(MCInst &Inst, unsigned RegNo,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  if (RegNo > 63)
    return MCDisassembler::Fail;
  unsigned Reg = I64RegDecoderTable[RegNo];
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `DecodeI64RegisterClass`, `addOperand`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeI64RegisterClass`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 153-162
```cpp
static DecodeStatus DecodeF32RegisterClass(MCInst &Inst, unsigned RegNo,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  if (RegNo > 63)
    return MCDisassembler::Fail;
  unsigned Reg = F32RegDecoderTable[RegNo];
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `DecodeF32RegisterClass`, `addOperand`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeF32RegisterClass`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 163-172
```cpp
static DecodeStatus DecodeF128RegisterClass(MCInst &Inst, unsigned RegNo,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  if (RegNo % 2 || RegNo > 63)
    return MCDisassembler::Fail;
  unsigned Reg = F128RegDecoderTable[RegNo / 2];
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `DecodeF128RegisterClass`, `addOperand`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeF128RegisterClass`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 173-186
```cpp
static DecodeStatus DecodeV64RegisterClass(MCInst &Inst, unsigned RegNo,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  unsigned Reg = VE::NoRegister;
  if (RegNo == 255)
    Reg = VE::VIX;
  else if (RegNo > 63)
    return MCDisassembler::Fail;
  else
    Reg = V64RegDecoderTable[RegNo];
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `DecodeV64RegisterClass`, `addOperand`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeV64RegisterClass`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 187-196
```cpp
static DecodeStatus DecodeVMRegisterClass(MCInst &Inst, unsigned RegNo,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  if (RegNo > 15)
    return MCDisassembler::Fail;
  unsigned Reg = VMRegDecoderTable[RegNo];
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `DecodeVMRegisterClass`, `addOperand`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeVMRegisterClass`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 197-206
```cpp
static DecodeStatus DecodeVM512RegisterClass(MCInst &Inst, unsigned RegNo,
                                             uint64_t Address,
                                             const MCDisassembler *Decoder) {
  if (RegNo % 2 || RegNo > 15)
    return MCDisassembler::Fail;
  unsigned Reg = VM512RegDecoderTable[RegNo / 2];
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `DecodeVM512RegisterClass`, `addOperand`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeVM512RegisterClass`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 207-218
```cpp
static DecodeStatus DecodeMISCRegisterClass(MCInst &Inst, unsigned RegNo,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  if (RegNo > 30)
    return MCDisassembler::Fail;
  unsigned Reg = MiscRegDecoderTable[RegNo];
  if (Reg == VE::NoRegister)
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `DecodeMISCRegisterClass`, `addOperand`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeMISCRegisterClass`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 219-230
```cpp
typedef DecodeStatus (*DecodeFunc)(MCInst &MI, unsigned RegNo, uint64_t Address,
                                   const MCDisassembler *Decoder);

static DecodeStatus DecodeASX(MCInst &MI, uint64_t insn, uint64_t Address,
                              const MCDisassembler *Decoder) {
  unsigned sy = fieldFromInstruction(insn, 40, 7);
  bool cy = fieldFromInstruction(insn, 47, 1);
  unsigned sz = fieldFromInstruction(insn, 32, 7);
  bool cz = fieldFromInstruction(insn, 39, 1);
  uint64_t simm32 = SignExtend64<32>(fieldFromInstruction(insn, 0, 32));
  DecodeStatus status;

```
- **EN**: Implements logic around `DecodeStatus`, `DecodeASX`, `fieldFromInstruction`, `SignExtend64<32>`; this block works at the MC layer.
- **CN**: 围绕 `DecodeStatus`, `DecodeASX`, `fieldFromInstruction`, `SignExtend64<32>` 实现具体逻辑；这一段工作在 MC 层。

### Lines 231-239
```cpp
  // Decode sz.
  if (cz) {
    status = DecodeI64RegisterClass(MI, sz, Address, Decoder);
    if (status != MCDisassembler::Success)
      return status;
  } else {
    MI.addOperand(MCOperand::createImm(0));
  }

```
- **EN**: Implements logic around `DecodeI64RegisterClass`, `addOperand`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `DecodeI64RegisterClass`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 240-248
```cpp
  // Decode sy.
  if (cy) {
    status = DecodeI64RegisterClass(MI, sy, Address, Decoder);
    if (status != MCDisassembler::Success)
      return status;
  } else {
    MI.addOperand(MCOperand::createImm(SignExtend32<7>(sy)));
  }

```
- **EN**: Implements logic around `DecodeI64RegisterClass`, `addOperand`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `DecodeI64RegisterClass`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 249-261
```cpp
  // Decode simm32.
  MI.addOperand(MCOperand::createImm(simm32));

  return MCDisassembler::Success;
}

static DecodeStatus DecodeAS(MCInst &MI, uint64_t insn, uint64_t Address,
                             const MCDisassembler *Decoder) {
  unsigned sz = fieldFromInstruction(insn, 32, 7);
  bool cz = fieldFromInstruction(insn, 39, 1);
  uint64_t simm32 = SignExtend64<32>(fieldFromInstruction(insn, 0, 32));
  DecodeStatus status;

```
- **EN**: Implements logic around `addOperand`, `DecodeAS`, `fieldFromInstruction`, `SignExtend64<32>`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `addOperand`, `DecodeAS`, `fieldFromInstruction`, `SignExtend64<32>` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 262-270
```cpp
  // Decode sz.
  if (cz) {
    status = DecodeI64RegisterClass(MI, sz, Address, Decoder);
    if (status != MCDisassembler::Success)
      return status;
  } else {
    MI.addOperand(MCOperand::createImm(0));
  }

```
- **EN**: Implements logic around `DecodeI64RegisterClass`, `addOperand`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `DecodeI64RegisterClass`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 271-281
```cpp
  // Decode simm32.
  MI.addOperand(MCOperand::createImm(simm32));

  return MCDisassembler::Success;
}

static DecodeStatus DecodeMem(MCInst &MI, uint64_t insn, uint64_t Address,
                              const MCDisassembler *Decoder, bool isLoad,
                              DecodeFunc DecodeSX) {
  unsigned sx = fieldFromInstruction(insn, 48, 7);

```
- **EN**: Implements logic around `addOperand`, `DecodeMem`, `fieldFromInstruction`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `addOperand`, `DecodeMem`, `fieldFromInstruction` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 282-292
```cpp
  DecodeStatus status;
  if (isLoad) {
    status = DecodeSX(MI, sx, Address, Decoder);
    if (status != MCDisassembler::Success)
      return status;
  }

  status = DecodeASX(MI, insn, Address, Decoder);
  if (status != MCDisassembler::Success)
    return status;

```
- **EN**: Implements logic around `DecodeSX`, `DecodeASX`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `DecodeSX`, `DecodeASX` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 293-305
```cpp
  if (!isLoad) {
    status = DecodeSX(MI, sx, Address, Decoder);
    if (status != MCDisassembler::Success)
      return status;
  }
  return MCDisassembler::Success;
}

static DecodeStatus DecodeMemAS(MCInst &MI, uint64_t insn, uint64_t Address,
                                const MCDisassembler *Decoder, bool isLoad,
                                DecodeFunc DecodeSX) {
  unsigned sx = fieldFromInstruction(insn, 48, 7);

```
- **EN**: Implements logic around `DecodeSX`, `DecodeMemAS`, `fieldFromInstruction`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeSX`, `DecodeMemAS`, `fieldFromInstruction` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 306-316
```cpp
  DecodeStatus status;
  if (isLoad) {
    status = DecodeSX(MI, sx, Address, Decoder);
    if (status != MCDisassembler::Success)
      return status;
  }

  status = DecodeAS(MI, insn, Address, Decoder);
  if (status != MCDisassembler::Success)
    return status;

```
- **EN**: Implements logic around `DecodeSX`, `DecodeAS`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `DecodeSX`, `DecodeAS` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 317-329
```cpp
  if (!isLoad) {
    status = DecodeSX(MI, sx, Address, Decoder);
    if (status != MCDisassembler::Success)
      return status;
  }
  return MCDisassembler::Success;
}

static DecodeStatus DecodeLoadI32(MCInst &Inst, uint64_t insn, uint64_t Address,
                                  const MCDisassembler *Decoder) {
  return DecodeMem(Inst, insn, Address, Decoder, true, DecodeI32RegisterClass);
}

```
- **EN**: Implements logic around `DecodeSX`, `DecodeLoadI32`, `DecodeMem`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeSX`, `DecodeLoadI32`, `DecodeMem` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 330-340
```cpp
static DecodeStatus DecodeStoreI32(MCInst &Inst, uint64_t insn,
                                   uint64_t Address,
                                   const MCDisassembler *Decoder) {
  return DecodeMem(Inst, insn, Address, Decoder, false, DecodeI32RegisterClass);
}

static DecodeStatus DecodeLoadI64(MCInst &Inst, uint64_t insn, uint64_t Address,
                                  const MCDisassembler *Decoder) {
  return DecodeMem(Inst, insn, Address, Decoder, true, DecodeI64RegisterClass);
}

```
- **EN**: Implements logic around `DecodeStoreI32`, `DecodeMem`, `DecodeLoadI64`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeStoreI32`, `DecodeMem`, `DecodeLoadI64` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 341-351
```cpp
static DecodeStatus DecodeStoreI64(MCInst &Inst, uint64_t insn,
                                   uint64_t Address,
                                   const MCDisassembler *Decoder) {
  return DecodeMem(Inst, insn, Address, Decoder, false, DecodeI64RegisterClass);
}

static DecodeStatus DecodeLoadF32(MCInst &Inst, uint64_t insn, uint64_t Address,
                                  const MCDisassembler *Decoder) {
  return DecodeMem(Inst, insn, Address, Decoder, true, DecodeF32RegisterClass);
}

```
- **EN**: Implements logic around `DecodeStoreI64`, `DecodeMem`, `DecodeLoadF32`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeStoreI64`, `DecodeMem`, `DecodeLoadF32` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 352-364
```cpp
static DecodeStatus DecodeStoreF32(MCInst &Inst, uint64_t insn,
                                   uint64_t Address,
                                   const MCDisassembler *Decoder) {
  return DecodeMem(Inst, insn, Address, Decoder, false, DecodeF32RegisterClass);
}

static DecodeStatus DecodeLoadASI64(MCInst &Inst, uint64_t insn,
                                    uint64_t Address,
                                    const MCDisassembler *Decoder) {
  return DecodeMemAS(Inst, insn, Address, Decoder, true,
                     DecodeI64RegisterClass);
}

```
- **EN**: Implements logic around `DecodeStoreF32`, `DecodeMem`, `DecodeLoadASI64`, `DecodeMemAS`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeStoreF32`, `DecodeMem`, `DecodeLoadASI64`, `DecodeMemAS` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 365-378
```cpp
static DecodeStatus DecodeStoreASI64(MCInst &Inst, uint64_t insn,
                                     uint64_t Address,
                                     const MCDisassembler *Decoder) {
  return DecodeMemAS(Inst, insn, Address, Decoder, false,
                     DecodeI64RegisterClass);
}

static DecodeStatus DecodeCAS(MCInst &MI, uint64_t insn, uint64_t Address,
                              const MCDisassembler *Decoder, bool isImmOnly,
                              bool isUImm, DecodeFunc DecodeSX) {
  unsigned sx = fieldFromInstruction(insn, 48, 7);
  bool cy = fieldFromInstruction(insn, 47, 1);
  unsigned sy = fieldFromInstruction(insn, 40, 7);

```
- **EN**: Implements logic around `DecodeStoreASI64`, `DecodeMemAS`, `DecodeCAS`, `fieldFromInstruction`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeStoreASI64`, `DecodeMemAS`, `DecodeCAS`, `fieldFromInstruction` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 379-389
```cpp
  // Add $sx.
  DecodeStatus status;
  status = DecodeSX(MI, sx, Address, Decoder);
  if (status != MCDisassembler::Success)
    return status;

  // Add $disp($sz).
  status = DecodeAS(MI, insn, Address, Decoder);
  if (status != MCDisassembler::Success)
    return status;

```
- **EN**: Implements logic around `DecodeSX`, `DecodeAS`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `DecodeSX`, `DecodeAS` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 390-401
```cpp
  // Add $sy.
  if (cy && !isImmOnly) {
    status = DecodeSX(MI, sy, Address, Decoder);
    if (status != MCDisassembler::Success)
      return status;
  } else {
    if (isUImm)
      MI.addOperand(MCOperand::createImm(sy));
    else
      MI.addOperand(MCOperand::createImm(SignExtend32<7>(sy)));
  }

```
- **EN**: Implements logic around `DecodeSX`, `addOperand`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `DecodeSX`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 402-415
```cpp
  // Add $sd.
  status = DecodeSX(MI, sx, Address, Decoder);
  if (status != MCDisassembler::Success)
    return status;

  return MCDisassembler::Success;
}

static DecodeStatus DecodeTS1AMI64(MCInst &MI, uint64_t insn, uint64_t Address,
                                   const MCDisassembler *Decoder) {
  return DecodeCAS(MI, insn, Address, Decoder, false, true,
                   DecodeI64RegisterClass);
}

```
- **EN**: Implements logic around `DecodeSX`, `DecodeTS1AMI64`, `DecodeCAS`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeSX`, `DecodeTS1AMI64`, `DecodeCAS` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 416-427
```cpp
static DecodeStatus DecodeTS1AMI32(MCInst &MI, uint64_t insn, uint64_t Address,
                                   const MCDisassembler *Decoder) {
  return DecodeCAS(MI, insn, Address, Decoder, false, true,
                   DecodeI32RegisterClass);
}

static DecodeStatus DecodeCASI64(MCInst &MI, uint64_t insn, uint64_t Address,
                                 const MCDisassembler *Decoder) {
  return DecodeCAS(MI, insn, Address, Decoder, false, false,
                   DecodeI64RegisterClass);
}

```
- **EN**: Implements logic around `DecodeTS1AMI32`, `DecodeCAS`, `DecodeCASI64`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeTS1AMI32`, `DecodeCAS`, `DecodeCASI64` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 428-438
```cpp
static DecodeStatus DecodeCASI32(MCInst &MI, uint64_t insn, uint64_t Address,
                                 const MCDisassembler *Decoder) {
  return DecodeCAS(MI, insn, Address, Decoder, false, false,
                   DecodeI32RegisterClass);
}

static DecodeStatus DecodeCall(MCInst &Inst, uint64_t insn, uint64_t Address,
                               const MCDisassembler *Decoder) {
  return DecodeMem(Inst, insn, Address, Decoder, true, DecodeI64RegisterClass);
}

```
- **EN**: Implements logic around `DecodeCASI32`, `DecodeCAS`, `DecodeCall`, `DecodeMem`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeCASI32`, `DecodeCAS`, `DecodeCall`, `DecodeMem` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 439-452
```cpp
static DecodeStatus DecodeSIMM7(MCInst &MI, uint64_t insn, uint64_t Address,
                                const MCDisassembler *Decoder) {
  uint64_t tgt = SignExtend64<7>(insn);
  MI.addOperand(MCOperand::createImm(tgt));
  return MCDisassembler::Success;
}

static DecodeStatus DecodeSIMM32(MCInst &MI, uint64_t insn, uint64_t Address,
                                 const MCDisassembler *Decoder) {
  uint64_t tgt = SignExtend64<32>(insn);
  MI.addOperand(MCOperand::createImm(tgt));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `DecodeSIMM7`, `SignExtend64<7>`, `addOperand`, `DecodeSIMM32`, ...; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeSIMM7`, `SignExtend64<7>`, `addOperand`, `DecodeSIMM32`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 453-468
```cpp
static bool isIntegerBCKind(MCInst &MI) {

#define BCm_kind(NAME)                                                         \
  case NAME##rri:                                                              \
  case NAME##rzi:                                                              \
  case NAME##iri:                                                              \
  case NAME##izi:                                                              \
  case NAME##rri_nt:                                                           \
  case NAME##rzi_nt:                                                           \
  case NAME##iri_nt:                                                           \
  case NAME##izi_nt:                                                           \
  case NAME##rri_t:                                                            \
  case NAME##rzi_t:                                                            \
  case NAME##iri_t:                                                            \
  case NAME##izi_t:

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 469-485
```cpp
#define BCRm_kind(NAME)                                                        \
  case NAME##rr:                                                               \
  case NAME##ir:                                                               \
  case NAME##rr_nt:                                                            \
  case NAME##ir_nt:                                                            \
  case NAME##rr_t:                                                             \
  case NAME##ir_t:

  {
    using namespace llvm::VE;
    switch (MI.getOpcode()) {
      BCm_kind(BCFL) BCm_kind(BCFW) BCRm_kind(BRCFL)
          BCRm_kind(BRCFW) return true;
    }
  }
#undef BCm_kind

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 486-495
```cpp
  return false;
}

// Decode CC Operand field.
static DecodeStatus DecodeCCOperand(MCInst &MI, uint64_t cf, uint64_t Address,
                                    const MCDisassembler *Decoder) {
  MI.addOperand(MCOperand::createImm(VEValToCondCode(cf, isIntegerBCKind(MI))));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `DecodeCCOperand`, `addOperand`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeCCOperand`, `addOperand` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 496-510
```cpp
// Decode RD Operand field.
static DecodeStatus DecodeRDOperand(MCInst &MI, uint64_t cf, uint64_t Address,
                                    const MCDisassembler *Decoder) {
  MI.addOperand(MCOperand::createImm(VEValToRD(cf)));
  return MCDisassembler::Success;
}

// Decode branch condition instruction and CCOperand field in it.
static DecodeStatus DecodeBranchCondition(MCInst &MI, uint64_t insn,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  unsigned cf = fieldFromInstruction(insn, 48, 4);
  bool cy = fieldFromInstruction(insn, 47, 1);
  unsigned sy = fieldFromInstruction(insn, 40, 7);

```
- **EN**: Implements logic around `DecodeRDOperand`, `addOperand`, `DecodeBranchCondition`, `fieldFromInstruction`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeRDOperand`, `addOperand`, `DecodeBranchCondition`, `fieldFromInstruction` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 511-523
```cpp
  // Decode cf.
  MI.addOperand(MCOperand::createImm(VEValToCondCode(cf, isIntegerBCKind(MI))));

  // Decode sy.
  DecodeStatus status;
  if (cy) {
    status = DecodeI64RegisterClass(MI, sy, Address, Decoder);
    if (status != MCDisassembler::Success)
      return status;
  } else {
    MI.addOperand(MCOperand::createImm(SignExtend32<7>(sy)));
  }

```
- **EN**: Implements logic around `addOperand`, `DecodeI64RegisterClass`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `addOperand`, `DecodeI64RegisterClass` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 524-534
```cpp
  // Decode MEMri.
  return DecodeAS(MI, insn, Address, Decoder);
}

static DecodeStatus DecodeBranchConditionAlways(MCInst &MI, uint64_t insn,
                                                uint64_t Address,
                                                const MCDisassembler *Decoder) {
  // Decode MEMri.
  return DecodeAS(MI, insn, Address, Decoder);
}

```
- **EN**: Implements logic around `DecodeAS`, `DecodeBranchConditionAlways`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeAS`, `DecodeBranchConditionAlways` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 535-546
```cpp
#include "VEGenDisassemblerTables.inc"

/// Read four bytes from the ArrayRef and return 32 bit word.
static DecodeStatus readInstruction64(ArrayRef<uint8_t> Bytes, uint64_t Address,
                                      uint64_t &Size, uint64_t &Insn,
                                      bool IsLittleEndian) {
  // We want to read exactly 8 Bytes of data.
  if (Bytes.size() < 8) {
    Size = 0;
    return MCDisassembler::Fail;
  }

```
- **EN**: Pulls in the headers needed for this implementation, including `VEGenDisassemblerTables.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `VEGenDisassemblerTables.inc`。

### Lines 547-556
```cpp
  Insn = IsLittleEndian
             ? ((uint64_t)Bytes[0] << 0) | ((uint64_t)Bytes[1] << 8) |
                   ((uint64_t)Bytes[2] << 16) | ((uint64_t)Bytes[3] << 24) |
                   ((uint64_t)Bytes[4] << 32) | ((uint64_t)Bytes[5] << 40) |
                   ((uint64_t)Bytes[6] << 48) | ((uint64_t)Bytes[7] << 56)
             : ((uint64_t)Bytes[7] << 0) | ((uint64_t)Bytes[6] << 8) |
                   ((uint64_t)Bytes[5] << 16) | ((uint64_t)Bytes[4] << 24) |
                   ((uint64_t)Bytes[3] << 32) | ((uint64_t)Bytes[2] << 40) |
                   ((uint64_t)Bytes[1] << 48) | ((uint64_t)Bytes[0] << 56);

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 557-570
```cpp
  return MCDisassembler::Success;
}

DecodeStatus VEDisassembler::getInstruction(MCInst &Instr, uint64_t &Size,
                                            ArrayRef<uint8_t> Bytes,
                                            uint64_t Address,
                                            raw_ostream &CStream) const {
  uint64_t Insn;
  bool isLittleEndian = getContext().getAsmInfo().isLittleEndian();
  DecodeStatus Result =
      readInstruction64(Bytes, Address, Size, Insn, isLittleEndian);
  if (Result == MCDisassembler::Fail)
    return MCDisassembler::Fail;

```
- **EN**: Implements logic around `getInstruction`, `getContext`, `readInstruction64`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `getInstruction`, `getContext`, `readInstruction64` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 571-579
```cpp
  // Calling the auto-generated decoder function.

  Result = decodeInstruction(DecoderTableVE64, Instr, Insn, Address, this, STI);

  if (Result != MCDisassembler::Fail) {
    Size = 8;
    return Result;
  }

```
- **EN**: Implements logic around `decodeInstruction`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `decodeInstruction` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 580-581
```cpp
  return MCDisassembler::Fail;
}
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

## Key Concepts / 关键概念

- **Disassembly / 反汇编**:
  - **EN**: Decodes instruction bytes into MCInst objects
  - **CN**: 将指令字节解码为 MCInst

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MCTargetDesc/VEMCTargetDesc.h`, `TargetInfo/VETargetInfo.h`, `VE.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCDecoder.h`, `llvm/MC/MCDecoderOps.h`, `llvm/MC/MCDisassembler/MCDisassembler.h`, `llvm/MC/MCInst.h`, `llvm/MC/TargetRegistry.h`, `llvm/Support/Compiler.h`, `VEGenDisassemblerTables.inc`
- **LLVM subsystems / LLVM 子系统**: MC, Support
