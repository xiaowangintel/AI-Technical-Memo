# MSP430Disassembler.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/MSP430/Disassembler/MSP430Disassembler.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements the target-specific disassembler that decodes instruction bytes into MCInst objects.
  - **CN**: 实现目标相关的反汇编器，把指令字节解码为 MCInst 对象。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- MSP430Disassembler.cpp - Disassembler for MSP430 ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the MSP430Disassembler class.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 12-25
```cpp

#include "MCTargetDesc/MSP430MCTargetDesc.h"
#include "MSP430.h"
#include "TargetInfo/MSP430TargetInfo.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCDecoder.h"
#include "llvm/MC/MCDecoderOps.h"
#include "llvm/MC/MCDisassembler/MCDisassembler.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/Endian.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `MCTargetDesc/MSP430MCTargetDesc.h`, `MSP430.h`, `TargetInfo/MSP430TargetInfo.h`, `llvm/MC/MCContext.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MCTargetDesc/MSP430MCTargetDesc.h`, `MSP430.h`, `TargetInfo/MSP430TargetInfo.h`, `llvm/MC/MCContext.h`。

### Lines 26-38
```cpp
using namespace llvm;
using namespace llvm::MCD;

#define DEBUG_TYPE "msp430-disassembler"

typedef MCDisassembler::DecodeStatus DecodeStatus;

namespace {
class MSP430Disassembler : public MCDisassembler {
  DecodeStatus getInstructionI(MCInst &MI, uint64_t &Size,
                               ArrayRef<uint8_t> Bytes, uint64_t Address,
                               raw_ostream &CStream) const;

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 39-50
```cpp
  DecodeStatus getInstructionII(MCInst &MI, uint64_t &Size,
                                ArrayRef<uint8_t> Bytes, uint64_t Address,
                                raw_ostream &CStream) const;

  DecodeStatus getInstructionCJ(MCInst &MI, uint64_t &Size,
                                ArrayRef<uint8_t> Bytes, uint64_t Address,
                                raw_ostream &CStream) const;

public:
  MSP430Disassembler(const MCSubtargetInfo &STI, MCContext &Ctx)
      : MCDisassembler(STI, Ctx) {}

```
- **EN**: Implements logic around `getInstructionII`, `getInstructionCJ`, `MSP430Disassembler`, `MCDisassembler`; this block works at the MC layer.
- **CN**: 围绕 `getInstructionII`, `getInstructionCJ`, `MSP430Disassembler`, `MCDisassembler` 实现具体逻辑；这一段工作在 MC 层。

### Lines 51-62
```cpp
  DecodeStatus getInstruction(MCInst &MI, uint64_t &Size,
                              ArrayRef<uint8_t> Bytes, uint64_t Address,
                              raw_ostream &CStream) const override;
};
} // end anonymous namespace

static MCDisassembler *createMSP430Disassembler(const Target &T,
                                                const MCSubtargetInfo &STI,
                                                MCContext &Ctx) {
  return new MSP430Disassembler(STI, Ctx);
}

```
- **EN**: Implements logic around `getInstruction`, `createMSP430Disassembler`, `MSP430Disassembler`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `getInstruction`, `createMSP430Disassembler`, `MSP430Disassembler` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 63-75
```cpp
extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
LLVMInitializeMSP430Disassembler() {
  TargetRegistry::RegisterMCDisassembler(getTheMSP430Target(),
                                         createMSP430Disassembler);
}

static const unsigned GR8DecoderTable[] = {
  MSP430::PCB,  MSP430::SPB,  MSP430::SRB,  MSP430::CGB,
  MSP430::R4B,  MSP430::R5B,  MSP430::R6B,  MSP430::R7B,
  MSP430::R8B,  MSP430::R9B,  MSP430::R10B, MSP430::R11B,
  MSP430::R12B, MSP430::R13B, MSP430::R14B, MSP430::R15B
};

```
- **EN**: Implements logic around `RegisterMCDisassembler`.
- **CN**: 围绕 `RegisterMCDisassembler` 实现具体逻辑。

### Lines 76-86
```cpp
static DecodeStatus DecodeGR8RegisterClass(MCInst &MI, uint64_t RegNo,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  if (RegNo > 15)
    return MCDisassembler::Fail;

  unsigned Reg = GR8DecoderTable[RegNo];
  MI.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

```
- **EN**: Implements logic around `DecodeGR8RegisterClass`, `addOperand`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeGR8RegisterClass`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 87-99
```cpp
static const unsigned GR16DecoderTable[] = {
  MSP430::PC,  MSP430::SP,  MSP430::SR,  MSP430::CG,
  MSP430::R4,  MSP430::R5,  MSP430::R6,  MSP430::R7,
  MSP430::R8,  MSP430::R9,  MSP430::R10, MSP430::R11,
  MSP430::R12, MSP430::R13, MSP430::R14, MSP430::R15
};

static DecodeStatus DecodeGR16RegisterClass(MCInst &MI, uint64_t RegNo,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  if (RegNo > 15)
    return MCDisassembler::Fail;

```
- **EN**: Implements logic around `DecodeGR16RegisterClass`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `DecodeGR16RegisterClass` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 100-117
```cpp
  unsigned Reg = GR16DecoderTable[RegNo];
  MI.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}

static DecodeStatus DecodeCGImm(MCInst &MI, uint64_t Bits, uint64_t Address,
                                const MCDisassembler *Decoder) {
  int64_t Imm;
  switch (Bits) {
  default:
    llvm_unreachable("Invalid immediate value");
  case 0x22: Imm =  4; break;
  case 0x32: Imm =  8; break;
  case 0x03: Imm =  0; break;
  case 0x13: Imm =  1; break;
  case 0x23: Imm =  2; break;
  case 0x33: Imm = -1; break;
  }
```
- **EN**: Implements logic around `addOperand`, `DecodeCGImm`, `llvm_unreachable`; this block uses `switch`-based dispatch; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `addOperand`, `DecodeCGImm`, `llvm_unreachable` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，工作在 MC 层。

### Lines 118-127
```cpp
  MI.addOperand(MCOperand::createImm(Imm));
  return MCDisassembler::Success;
}

static DecodeStatus DecodeMemOperand(MCInst &MI, uint64_t Bits,
                                     uint64_t Address,
                                     const MCDisassembler *Decoder) {
  unsigned Reg = Bits & 15;
  unsigned Imm = Bits >> 4;

```
- **EN**: Implements logic around `addOperand`, `DecodeMemOperand`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `addOperand`, `DecodeMemOperand` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 128-137
```cpp
  if (DecodeGR16RegisterClass(MI, Reg, Address, Decoder) !=
      MCDisassembler::Success)
    return MCDisassembler::Fail;
  
  MI.addOperand(MCOperand::createImm((int16_t)Imm));
  return MCDisassembler::Success;
}

#include "MSP430GenDisassemblerTables.inc"

```
- **EN**: Pulls in the headers needed for this implementation, including `MSP430GenDisassemblerTables.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `MSP430GenDisassemblerTables.inc`。

### Lines 138-149
```cpp
enum AddrMode {
  amInvalid = 0,
  amRegister,
  amIndexed,
  amIndirect,
  amIndirectPost,
  amSymbolic,
  amImmediate,
  amAbsolute,
  amConstant
};

```
- **EN**: Introduces declarations for `AddrMode`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `AddrMode` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 150-167
```cpp
static AddrMode DecodeSrcAddrMode(unsigned Rs, unsigned As) {
  switch (Rs) {
  case 0:
    if (As == 1) return amSymbolic;
    if (As == 2) return amInvalid;
    if (As == 3) return amImmediate;
    break;
  case 2:
    if (As == 1) return amAbsolute;
    if (As == 2) return amConstant;
    if (As == 3) return amConstant;
    break;
  case 3:
    return amConstant;
  default:
    break;
  }
  switch (As) {
```
- **EN**: Implements logic around `DecodeSrcAddrMode`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `DecodeSrcAddrMode` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 168-176
```cpp
  case 0: return amRegister;
  case 1: return amIndexed;
  case 2: return amIndirect;
  case 3: return amIndirectPost;
  default:
    llvm_unreachable("As out of range");
  }
}

```
- **EN**: Implements logic around `llvm_unreachable`; this block returns target-specific results.
- **CN**: 围绕 `llvm_unreachable` 实现具体逻辑；这一段返回目标相关结果。

### Lines 177-188
```cpp
static AddrMode DecodeSrcAddrModeI(unsigned Insn) {
  unsigned Rs = fieldFromInstruction(Insn, 8, 4);
  unsigned As = fieldFromInstruction(Insn, 4, 2);
  return DecodeSrcAddrMode(Rs, As);
}

static AddrMode DecodeSrcAddrModeII(unsigned Insn) {
  unsigned Rs = fieldFromInstruction(Insn, 0, 4);
  unsigned As = fieldFromInstruction(Insn, 4, 2);
  return DecodeSrcAddrMode(Rs, As);
}

```
- **EN**: Implements logic around `DecodeSrcAddrModeI`, `fieldFromInstruction`, `DecodeSrcAddrMode`, `DecodeSrcAddrModeII`; this block returns target-specific results.
- **CN**: 围绕 `DecodeSrcAddrModeI`, `fieldFromInstruction`, `DecodeSrcAddrMode`, `DecodeSrcAddrModeII` 实现具体逻辑；这一段返回目标相关结果。

### Lines 189-200
```cpp
static AddrMode DecodeDstAddrMode(unsigned Insn) {
  unsigned Rd = fieldFromInstruction(Insn, 0, 4);
  unsigned Ad = fieldFromInstruction(Insn, 7, 1);
  switch (Rd) {
  case 0: return Ad ? amSymbolic : amRegister;
  case 2: return Ad ? amAbsolute : amRegister;
  default:
    break;
  }
  return Ad ? amIndexed : amRegister;
}

```
- **EN**: Implements logic around `DecodeDstAddrMode`, `fieldFromInstruction`; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `DecodeDstAddrMode`, `fieldFromInstruction` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 201-218
```cpp
static const uint8_t *getDecoderTable(AddrMode SrcAM, unsigned Words) {
  assert(0 < Words && Words < 4 && "Incorrect number of words");
  switch (SrcAM) {
  default:
    llvm_unreachable("Invalid addressing mode");
  case amRegister:
    assert(Words < 3 && "Incorrect number of words");
    return Words == 2 ? DecoderTableAlpha32 : DecoderTableAlpha16;
  case amConstant:
    assert(Words < 3 && "Incorrect number of words");
    return Words == 2 ? DecoderTableBeta32 : DecoderTableBeta16;
  case amIndexed:
  case amSymbolic:
  case amImmediate:
  case amAbsolute:
    assert(Words > 1 && "Incorrect number of words");
    return Words == 2 ? DecoderTableGamma32 : DecoderTableGamma48;
  case amIndirect:
```
- **EN**: Implements logic around `getDecoderTable`, `assert`, `llvm_unreachable`; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `getDecoderTable`, `assert`, `llvm_unreachable` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 219-236
```cpp
  case amIndirectPost:
    assert(Words < 3 && "Incorrect number of words");
    return Words == 2 ? DecoderTableDelta32 : DecoderTableDelta16;
  }
}

DecodeStatus MSP430Disassembler::getInstructionI(MCInst &MI, uint64_t &Size,
                                                 ArrayRef<uint8_t> Bytes,
                                                 uint64_t Address,
                                                 raw_ostream &CStream) const {
  uint64_t Insn = support::endian::read16le(Bytes.data());
  AddrMode SrcAM = DecodeSrcAddrModeI(Insn);
  AddrMode DstAM = DecodeDstAddrMode(Insn);
  if (SrcAM == amInvalid || DstAM == amInvalid) {
    Size = 2; // skip one word and let disassembler to try further
    return MCDisassembler::Fail;
  }

```
- **EN**: Implements logic around `assert`, `getInstructionI`, `read16le`, `DecodeSrcAddrModeI`, ...; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `assert`, `getInstructionI`, `read16le`, `DecodeSrcAddrModeI`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 237-254
```cpp
  unsigned Words = 1;
  switch (SrcAM) {
  case amIndexed:
  case amSymbolic:
  case amImmediate:
  case amAbsolute:
    if (Bytes.size() < (Words + 1) * 2) {
      Size = 2;
      return DecodeStatus::Fail;
    }
    Insn |= (uint64_t)support::endian::read16le(Bytes.data() + 2) << 16;
    ++Words;
    break;
  default:
    break;
  }
  switch (DstAM) {
  case amIndexed:
```
- **EN**: Implements logic around `read16le`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `read16le` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 255-268
```cpp
  case amSymbolic:
  case amAbsolute:
    if (Bytes.size() < (Words + 1) * 2) {
      Size = 2;
      return DecodeStatus::Fail;
    }
    Insn |= (uint64_t)support::endian::read16le(Bytes.data() + Words * 2)
        << (Words * 16);
    ++Words;
    break;
  default:
    break;
  }

```
- **EN**: Implements logic around `read16le`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `read16le` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 269-279
```cpp
  DecodeStatus Result = decodeInstruction(getDecoderTable(SrcAM, Words), MI,
                                          Insn, Address, this, STI);
  if (Result != MCDisassembler::Fail) {
    Size = Words * 2;
    return Result;
  }

  Size = 2;
  return DecodeStatus::Fail;
}

```
- **EN**: Implements logic around `decodeInstruction`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `decodeInstruction` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 280-290
```cpp
DecodeStatus MSP430Disassembler::getInstructionII(MCInst &MI, uint64_t &Size,
                                                  ArrayRef<uint8_t> Bytes,
                                                  uint64_t Address,
                                                  raw_ostream &CStream) const {
  uint64_t Insn = support::endian::read16le(Bytes.data());
  AddrMode SrcAM = DecodeSrcAddrModeII(Insn);
  if (SrcAM == amInvalid) {
    Size = 2; // skip one word and let disassembler to try further
    return MCDisassembler::Fail;
  }

```
- **EN**: Implements logic around `getInstructionII`, `read16le`, `DecodeSrcAddrModeII`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `getInstructionII`, `read16le`, `DecodeSrcAddrModeII` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 291-307
```cpp
  unsigned Words = 1;
  switch (SrcAM) {
  case amIndexed:
  case amSymbolic:
  case amImmediate:
  case amAbsolute:
    if (Bytes.size() < (Words + 1) * 2) {
      Size = 2;
      return DecodeStatus::Fail;
    }
    Insn |= (uint64_t)support::endian::read16le(Bytes.data() + 2) << 16;
    ++Words;
    break;
  default:
    break;
  }

```
- **EN**: Implements logic around `read16le`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `read16le` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 308-319
```cpp
  const uint8_t *DecoderTable = Words == 2 ? DecoderTable32 : DecoderTable16;
  DecodeStatus Result = decodeInstruction(DecoderTable, MI, Insn, Address,
                                          this, STI);
  if (Result != MCDisassembler::Fail) {
    Size = Words * 2;
    return Result;
  }

  Size = 2;
  return DecodeStatus::Fail;
}

```
- **EN**: Implements logic around `decodeInstruction`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `decodeInstruction` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 320-334
```cpp
static MSP430CC::CondCodes getCondCode(unsigned Cond) {
  switch (Cond) {
  case 0: return MSP430CC::COND_NE;
  case 1: return MSP430CC::COND_E;
  case 2: return MSP430CC::COND_LO;
  case 3: return MSP430CC::COND_HS;
  case 4: return MSP430CC::COND_N;
  case 5: return MSP430CC::COND_GE;
  case 6: return MSP430CC::COND_L;
  case 7: return MSP430CC::COND_NONE;
  default:
    llvm_unreachable("Cond out of range");
  }
}

```
- **EN**: Implements logic around `getCondCode`, `llvm_unreachable`; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `getCondCode`, `llvm_unreachable` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 335-344
```cpp
DecodeStatus MSP430Disassembler::getInstructionCJ(MCInst &MI, uint64_t &Size,
                                                  ArrayRef<uint8_t> Bytes,
                                                  uint64_t Address,
                                                  raw_ostream &CStream) const {
  uint64_t Insn = support::endian::read16le(Bytes.data());
  unsigned Cond = fieldFromInstruction(Insn, 10, 3);
  unsigned Offset = fieldFromInstruction(Insn, 0, 10);

  MI.addOperand(MCOperand::createImm(SignExtend32(Offset, 10)));

```
- **EN**: Implements logic around `getInstructionCJ`, `read16le`, `fieldFromInstruction`, `addOperand`; this block works at the MC layer.
- **CN**: 围绕 `getInstructionCJ`, `read16le`, `fieldFromInstruction`, `addOperand` 实现具体逻辑；这一段工作在 MC 层。

### Lines 345-355
```cpp
  if (Cond == 7)
    MI.setOpcode(MSP430::JMP);
  else {
    MI.setOpcode(MSP430::JCC);
    MI.addOperand(MCOperand::createImm(getCondCode(Cond)));
  }

  Size = 2;
  return DecodeStatus::Success;
}

```
- **EN**: Implements logic around `setOpcode`, `addOperand`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `setOpcode`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 356-364
```cpp
DecodeStatus MSP430Disassembler::getInstruction(MCInst &MI, uint64_t &Size,
                                                ArrayRef<uint8_t> Bytes,
                                                uint64_t Address,
                                                raw_ostream &CStream) const {
  if (Bytes.size() < 2) {
    Size = 0;
    return MCDisassembler::Fail;
  }

```
- **EN**: Implements logic around `getInstruction`; this block applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `getInstruction` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 365-375
```cpp
  uint64_t Insn = support::endian::read16le(Bytes.data());
  unsigned Opc = fieldFromInstruction(Insn, 13, 3);
  switch (Opc) {
  case 0:
    return getInstructionII(MI, Size, Bytes, Address, CStream);
  case 1:
    return getInstructionCJ(MI, Size, Bytes, Address, CStream);
  default:
    return getInstructionI(MI, Size, Bytes, Address, CStream);
  }
}
```
- **EN**: Implements logic around `read16le`, `fieldFromInstruction`, `getInstructionII`, `getInstructionCJ`, ...; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `read16le`, `fieldFromInstruction`, `getInstructionII`, `getInstructionCJ`, ... 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

## Key Concepts / 关键概念

- **Disassembly / 反汇编**:
  - **EN**: Decodes instruction bytes into MCInst objects
  - **CN**: 将指令字节解码为 MCInst

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MCTargetDesc/MSP430MCTargetDesc.h`, `MSP430.h`, `TargetInfo/MSP430TargetInfo.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCDecoder.h`, `llvm/MC/MCDecoderOps.h`, `llvm/MC/MCDisassembler/MCDisassembler.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/TargetRegistry.h`, `llvm/Support/Compiler.h`, `llvm/Support/Endian.h` ... (+1 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support
