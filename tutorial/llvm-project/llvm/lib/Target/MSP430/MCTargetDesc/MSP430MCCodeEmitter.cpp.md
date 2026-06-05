# MSP430MCCodeEmitter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/MSP430/MCTargetDesc/MSP430MCCodeEmitter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Provides MC-layer target descriptions such as fixups, asm info, code emission, object writing, and target registration.
  - **CN**: 提供 MC 层的目标描述，例如 fixup、汇编信息、编码发射、目标文件写出以及目标注册。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- MSP430MCCodeEmitter.cpp - Convert MSP430 code to machine code -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-16
```cpp
//
// This file implements the MSP430MCCodeEmitter class.
//
//===----------------------------------------------------------------------===//

#include "MSP430.h"
#include "MCTargetDesc/MSP430MCTargetDesc.h"
#include "MCTargetDesc/MSP430FixupKinds.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `MSP430.h`, `MCTargetDesc/MSP430MCTargetDesc.h`, `MCTargetDesc/MSP430FixupKinds.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MSP430.h`, `MCTargetDesc/MSP430MCTargetDesc.h`, `MCTargetDesc/MSP430FixupKinds.h`。

### Lines 17-28
```cpp
#include "llvm/ADT/APFloat.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/MC/MCCodeEmitter.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCFixup.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/Support/EndianStream.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ADT/APFloat.h`, `llvm/ADT/SmallVector.h`, `llvm/MC/MCCodeEmitter.h`, `llvm/MC/MCContext.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ADT/APFloat.h`, `llvm/ADT/SmallVector.h`, `llvm/MC/MCCodeEmitter.h`, `llvm/MC/MCContext.h`。

### Lines 29-36
```cpp
#define DEBUG_TYPE "mccodeemitter"

namespace llvm {

class MSP430MCCodeEmitter : public MCCodeEmitter {
  MCContext &Ctx;
  MCInstrInfo const &MCII;

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 37-46
```cpp
  // Offset keeps track of current word number being emitted
  // inside a particular instruction.
  mutable unsigned Offset;

  /// TableGen'erated function for getting the binary encoding for an
  /// instruction.
  uint64_t getBinaryCodeForInstr(const MCInst &MI,
                                 SmallVectorImpl<MCFixup> &Fixups,
                                 const MCSubtargetInfo &STI) const;

```
- **EN**: Implements logic around `getBinaryCodeForInstr`; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getBinaryCodeForInstr` 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 47-54
```cpp
  /// Returns the binary encoding of operands.
  ///
  /// If an operand requires relocation, the relocation is recorded
  /// and zero is returned.
  unsigned getMachineOpValue(const MCInst &MI, const MCOperand &MO,
                             SmallVectorImpl<MCFixup> &Fixups,
                             const MCSubtargetInfo &STI) const;

```
- **EN**: Implements logic around `getMachineOpValue`; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getMachineOpValue` 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 55-62
```cpp
  unsigned getMemOpValue(const MCInst &MI, unsigned Op,
                         SmallVectorImpl<MCFixup> &Fixups,
                         const MCSubtargetInfo &STI) const;

  unsigned getPCRelImmOpValue(const MCInst &MI, unsigned Op,
                              SmallVectorImpl<MCFixup> &Fixups,
                              const MCSubtargetInfo &STI) const;

```
- **EN**: Implements logic around `getMemOpValue`, `getPCRelImmOpValue`; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getMemOpValue`, `getPCRelImmOpValue` 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 63-70
```cpp
  unsigned getCGImmOpValue(const MCInst &MI, unsigned Op,
                           SmallVectorImpl<MCFixup> &Fixups,
                           const MCSubtargetInfo &STI) const;

  unsigned getCCOpValue(const MCInst &MI, unsigned Op,
                        SmallVectorImpl<MCFixup> &Fixups,
                        const MCSubtargetInfo &STI) const;

```
- **EN**: Implements logic around `getCGImmOpValue`, `getCCOpValue`; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getCGImmOpValue`, `getCCOpValue` 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 71-79
```cpp
public:
  MSP430MCCodeEmitter(MCContext &ctx, MCInstrInfo const &MCII)
      : Ctx(ctx), MCII(MCII) {}

  void encodeInstruction(const MCInst &MI, SmallVectorImpl<char> &CB,
                         SmallVectorImpl<MCFixup> &Fixups,
                         const MCSubtargetInfo &STI) const override;
};

```
- **EN**: Implements logic around `MSP430MCCodeEmitter`, `Ctx`, `encodeInstruction`; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `MSP430MCCodeEmitter`, `Ctx`, `encodeInstruction` 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 80-93
```cpp
static void addFixup(SmallVectorImpl<MCFixup> &Fixups, uint32_t Offset,
                     const MCExpr *Value, uint16_t Kind) {
  bool PCRel = false;
  switch (Kind) {
  case MSP430::fixup_10_pcrel:
  case MSP430::fixup_16_pcrel:
  case MSP430::fixup_16_pcrel_byte:
  case MSP430::fixup_2x_pcrel:
  case MSP430::fixup_rl_pcrel:
    PCRel = true;
  }
  Fixups.push_back(MCFixup::create(Offset, Value, Kind, PCRel));
}

```
- **EN**: Implements logic around `addFixup`, `push_back`; this block uses `switch`-based dispatch; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `addFixup`, `push_back` 实现具体逻辑；这一段使用 `switch` 分派，映射 fixup 或重定位，工作在 MC 层。

### Lines 94-101
```cpp
void MSP430MCCodeEmitter::encodeInstruction(const MCInst &MI,
                                            SmallVectorImpl<char> &CB,
                                            SmallVectorImpl<MCFixup> &Fixups,
                                            const MCSubtargetInfo &STI) const {
  const MCInstrDesc &Desc = MCII.get(MI.getOpcode());
  // Get byte count of instruction.
  unsigned Size = Desc.getSize();

```
- **EN**: Implements logic around `encodeInstruction`, `get`, `getSize`; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `encodeInstruction`, `get`, `getSize` 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 102-114
```cpp
  // Initialize fixup offset
  Offset = 2;

  uint64_t BinaryOpCode = getBinaryCodeForInstr(MI, Fixups, STI);
  size_t WordCount = Size / 2;

  while (WordCount--) {
    support::endian::write(CB, (uint16_t)BinaryOpCode,
                           llvm::endianness::little);
    BinaryOpCode >>= 16;
  }
}

```
- **EN**: Implements logic around `getBinaryCodeForInstr`, `write`; this block maps fixups or relocations.
- **CN**: 围绕 `getBinaryCodeForInstr`, `write` 实现具体逻辑；这一段映射 fixup 或重定位。

### Lines 115-121
```cpp
unsigned MSP430MCCodeEmitter::getMachineOpValue(const MCInst &MI,
                                                const MCOperand &MO,
                                                SmallVectorImpl<MCFixup> &Fixups,
                                                const MCSubtargetInfo &STI) const {
  if (MO.isReg())
    return Ctx.getRegisterInfo()->getEncodingValue(MO.getReg());

```
- **EN**: Implements logic around `getMachineOpValue`, `getRegisterInfo`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getMachineOpValue`, `getRegisterInfo` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 122-132
```cpp
  if (MO.isImm()) {
    Offset += 2;
    return MO.getImm();
  }

  assert(MO.isExpr() && "Expected expr operand");
  addFixup(Fixups, Offset, MO.getExpr(), MSP430::fixup_16_byte);
  Offset += 2;
  return 0;
}

```
- **EN**: Implements logic around `getImm`, `assert`, `addFixup`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `getImm`, `assert`, `addFixup` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 133-139
```cpp
unsigned MSP430MCCodeEmitter::getMemOpValue(const MCInst &MI, unsigned Op,
                                            SmallVectorImpl<MCFixup> &Fixups,
                                            const MCSubtargetInfo &STI) const {
  const MCOperand &MO1 = MI.getOperand(Op);
  assert(MO1.isReg() && "Register operand expected");
  unsigned Reg = Ctx.getRegisterInfo()->getEncodingValue(MO1.getReg());

```
- **EN**: Implements logic around `getMemOpValue`, `getOperand`, `assert`, `getRegisterInfo`; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getMemOpValue`, `getOperand`, `assert`, `getRegisterInfo` 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 140-153
```cpp
  const MCOperand &MO2 = MI.getOperand(Op + 1);
  if (MO2.isImm()) {
    Offset += 2;
    return ((unsigned)MO2.getImm() << 4) | Reg;
  }

  assert(MO2.isExpr() && "Expr operand expected");
  MSP430::Fixups FixupKind;
  switch (Reg) {
  case 0:
    FixupKind = MSP430::fixup_16_pcrel_byte;
    break;
  case 2:
    FixupKind = MSP430::fixup_16_byte;
```
- **EN**: Implements logic around `getOperand`, `assert`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `getOperand`, `assert` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 154-163
```cpp
    break;
  default:
    FixupKind = MSP430::fixup_16_byte;
    break;
  }
  addFixup(Fixups, Offset, MO2.getExpr(), FixupKind);
  Offset += 2;
  return Reg;
}

```
- **EN**: Implements logic around `addFixup`; this block returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `addFixup` 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位。

### Lines 164-170
```cpp
unsigned MSP430MCCodeEmitter::getPCRelImmOpValue(const MCInst &MI, unsigned Op,
                                                 SmallVectorImpl<MCFixup> &Fixups,
                                                 const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(Op);
  if (MO.isImm())
    return MO.getImm();

```
- **EN**: Implements logic around `getPCRelImmOpValue`, `getOperand`, `getImm`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getPCRelImmOpValue`, `getOperand`, `getImm` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 171-181
```cpp
  assert(MO.isExpr() && "Expr operand expected");
  addFixup(Fixups, 0, MO.getExpr(), MSP430::fixup_10_pcrel);
  return 0;
}

unsigned MSP430MCCodeEmitter::getCGImmOpValue(const MCInst &MI, unsigned Op,
                                              SmallVectorImpl<MCFixup> &Fixups,
                                              const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(Op);
  assert(MO.isImm() && "Expr operand expected");

```
- **EN**: Implements logic around `assert`, `addFixup`, `getCGImmOpValue`, `getOperand`; this block returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `assert`, `addFixup`, `getCGImmOpValue`, `getOperand` 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 182-194
```cpp
  int64_t Imm = MO.getImm();
  switch (Imm) {
  default:
    llvm_unreachable("Invalid immediate value");
  case 4:  return 0x22;
  case 8:  return 0x32;
  case 0:  return 0x03;
  case 1:  return 0x13;
  case 2:  return 0x23;
  case -1: return 0x33;
  }
}

```
- **EN**: Implements logic around `getImm`, `llvm_unreachable`; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `getImm`, `llvm_unreachable` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 195-208
```cpp
unsigned MSP430MCCodeEmitter::getCCOpValue(const MCInst &MI, unsigned Op,
                                           SmallVectorImpl<MCFixup> &Fixups,
                                           const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(Op);
  assert(MO.isImm() && "Immediate operand expected");
  switch (MO.getImm()) {
  case MSP430CC::COND_NE: return 0;
  case MSP430CC::COND_E:  return 1;
  case MSP430CC::COND_LO: return 2;
  case MSP430CC::COND_HS: return 3;
  case MSP430CC::COND_N:  return 4;
  case MSP430CC::COND_GE: return 5;
  case MSP430CC::COND_L:  return 6;
  default:
```
- **EN**: Implements logic around `getCCOpValue`, `getOperand`, `assert`; this block uses `switch`-based dispatch; returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getCCOpValue`, `getOperand`, `assert` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 209-217
```cpp
    llvm_unreachable("Unknown condition code");
  }
}

MCCodeEmitter *createMSP430MCCodeEmitter(const MCInstrInfo &MCII,
                                         MCContext &Ctx) {
  return new MSP430MCCodeEmitter(Ctx, MCII);
}

```
- **EN**: Implements logic around `llvm_unreachable`, `createMSP430MCCodeEmitter`, `MSP430MCCodeEmitter`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `llvm_unreachable`, `createMSP430MCCodeEmitter`, `MSP430MCCodeEmitter` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 218-220
```cpp
#include "MSP430GenMCCodeEmitter.inc"

} // end of namespace llvm
```
- **EN**: Pulls in the headers needed for this implementation, including `MSP430GenMCCodeEmitter.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `MSP430GenMCCodeEmitter.inc`。

## Key Concepts / 关键概念

- **MC layer / MC 层**:
  - **EN**: Connects the backend to LLVM's MC infrastructure
  - **CN**: 把后端接入 LLVM 的 MC 基础设施
- **Binary encoding / 二进制编码**:
  - **EN**: Turns MCInst structures into encoded bytes
  - **CN**: 将 MCInst 结构编码为字节流
- **Fixups and relocations / Fixup 与重定位**:
  - **EN**: Bridges symbolic references to concrete relocation records
  - **CN**: 把符号引用连接到具体重定位记录

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MSP430.h`, `MCTargetDesc/MSP430MCTargetDesc.h`, `MCTargetDesc/MSP430FixupKinds.h`, `llvm/ADT/APFloat.h`, `llvm/ADT/SmallVector.h`, `llvm/MC/MCCodeEmitter.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCFixup.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCRegisterInfo.h` ... (+3 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support
