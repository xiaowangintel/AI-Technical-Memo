# LanaiMCCodeEmitter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Lanai/MCTargetDesc/LanaiMCCodeEmitter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Provides MC-layer target descriptions such as fixups, asm info, code emission, object writing, and target registration.
  - **CN**: 提供 MC 层的目标描述，例如 fixup、汇编信息、编码发射、目标文件写出以及目标注册。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- LanaiMCCodeEmitter.cpp - Convert Lanai code to machine code -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the LanaiMCCodeEmitter class.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 12-29
```cpp

#include "LanaiAluCode.h"
#include "MCTargetDesc/LanaiBaseInfo.h"
#include "MCTargetDesc/LanaiFixupKinds.h"
#include "MCTargetDesc/LanaiMCAsmInfo.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/MC/MCCodeEmitter.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCFixup.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/EndianStream.h"
#include <cassert>
#include <cstdint>

```
- **EN**: Pulls in the headers needed for this implementation, including `LanaiAluCode.h`, `MCTargetDesc/LanaiBaseInfo.h`, `MCTargetDesc/LanaiFixupKinds.h`, `MCTargetDesc/LanaiMCAsmInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `LanaiAluCode.h`, `MCTargetDesc/LanaiBaseInfo.h`, `MCTargetDesc/LanaiFixupKinds.h`, `MCTargetDesc/LanaiMCAsmInfo.h`。

### Lines 30-44
```cpp
#define DEBUG_TYPE "mccodeemitter"

STATISTIC(MCNumEmitted, "Number of MC instructions emitted");

namespace llvm {

namespace {

class LanaiMCCodeEmitter : public MCCodeEmitter {
public:
  LanaiMCCodeEmitter(const MCInstrInfo &MCII, MCContext &C) {}
  LanaiMCCodeEmitter(const LanaiMCCodeEmitter &) = delete;
  void operator=(const LanaiMCCodeEmitter &) = delete;
  ~LanaiMCCodeEmitter() override = default;

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 45-53
```cpp
  // The functions below are called by TableGen generated functions for getting
  // the binary encoding of instructions/opereands.

  // getBinaryCodeForInstr - TableGen'erated function for getting the
  // binary encoding for an instruction.
  uint64_t getBinaryCodeForInstr(const MCInst &Inst,
                                 SmallVectorImpl<MCFixup> &Fixups,
                                 const MCSubtargetInfo &SubtargetInfo) const;

```
- **EN**: Implements logic around `getBinaryCodeForInstr`; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getBinaryCodeForInstr` 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 54-63
```cpp
  // getMachineOpValue - Return binary encoding of operand. If the machine
  // operand requires relocation, record the relocation and return zero.
  unsigned getMachineOpValue(const MCInst &Inst, const MCOperand &MCOp,
                             SmallVectorImpl<MCFixup> &Fixups,
                             const MCSubtargetInfo &SubtargetInfo) const;

  unsigned getRiMemoryOpValue(const MCInst &Inst, unsigned OpNo,
                              SmallVectorImpl<MCFixup> &Fixups,
                              const MCSubtargetInfo &SubtargetInfo) const;

```
- **EN**: Implements logic around `getMachineOpValue`, `getRiMemoryOpValue`; this block returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getMachineOpValue`, `getRiMemoryOpValue` 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 64-75
```cpp
  unsigned getRrMemoryOpValue(const MCInst &Inst, unsigned OpNo,
                              SmallVectorImpl<MCFixup> &Fixups,
                              const MCSubtargetInfo &SubtargetInfo) const;

  unsigned getSplsOpValue(const MCInst &Inst, unsigned OpNo,
                          SmallVectorImpl<MCFixup> &Fixups,
                          const MCSubtargetInfo &SubtargetInfo) const;

  unsigned getBranchTargetOpValue(const MCInst &Inst, unsigned OpNo,
                                  SmallVectorImpl<MCFixup> &Fixups,
                                  const MCSubtargetInfo &SubtargetInfo) const;

```
- **EN**: Implements logic around `getRrMemoryOpValue`, `getSplsOpValue`, `getBranchTargetOpValue`; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getRrMemoryOpValue`, `getSplsOpValue`, `getBranchTargetOpValue` 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 76-86
```cpp
  void encodeInstruction(const MCInst &Inst, SmallVectorImpl<char> &CB,
                         SmallVectorImpl<MCFixup> &Fixups,
                         const MCSubtargetInfo &SubtargetInfo) const override;

  unsigned adjustPqBitsRmAndRrm(const MCInst &Inst, unsigned Value,
                                const MCSubtargetInfo &STI) const;

  unsigned adjustPqBitsSpls(const MCInst &Inst, unsigned Value,
                            const MCSubtargetInfo &STI) const;
};

```
- **EN**: Implements logic around `encodeInstruction`, `adjustPqBitsRmAndRrm`, `adjustPqBitsSpls`; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `encodeInstruction`, `adjustPqBitsRmAndRrm`, `adjustPqBitsSpls` 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 87-104
```cpp
} // end anonymous namespace

static Lanai::Fixups FixupKind(const MCExpr *Expr) {
  if (isa<MCSymbolRefExpr>(Expr))
    return Lanai::FIXUP_LANAI_21;
  if (const MCSpecifierExpr *McExpr = dyn_cast<MCSpecifierExpr>(Expr)) {
    Lanai::Specifier ExprKind = McExpr->getSpecifier();
    switch (ExprKind) {
    case Lanai::S_None:
      return Lanai::FIXUP_LANAI_21;
    case Lanai::S_ABS_HI:
      return Lanai::FIXUP_LANAI_HI16;
    case Lanai::S_ABS_LO:
      return Lanai::FIXUP_LANAI_LO16;
    }
  }
  return Lanai::Fixups(0);
}
```
- **EN**: Implements logic around `FixupKind`, `getSpecifier`, `Fixups`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `FixupKind`, `getSpecifier`, `Fixups` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 105-115
```cpp

// getMachineOpValue - Return binary encoding of operand. If the machine
// operand requires relocation, record the relocation and return zero.
unsigned LanaiMCCodeEmitter::getMachineOpValue(
    const MCInst &Inst, const MCOperand &MCOp, SmallVectorImpl<MCFixup> &Fixups,
    const MCSubtargetInfo &SubtargetInfo) const {
  if (MCOp.isReg())
    return getLanaiRegisterNumbering(MCOp.getReg());
  if (MCOp.isImm())
    return static_cast<unsigned>(MCOp.getImm());

```
- **EN**: Implements logic around `getMachineOpValue`, `getLanaiRegisterNumbering`, `static_cast<unsigned>`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getMachineOpValue`, `getLanaiRegisterNumbering`, `static_cast<unsigned>` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 116-125
```cpp
  // MCOp must be an expression
  assert(MCOp.isExpr());
  const MCExpr *Expr = MCOp.getExpr();

  // Extract the symbolic reference side of a binary expression.
  if (Expr->getKind() == MCExpr::Binary) {
    const MCBinaryExpr *BinaryExpr = static_cast<const MCBinaryExpr *>(Expr);
    Expr = BinaryExpr->getLHS();
  }

```
- **EN**: Implements logic around `assert`, `getExpr`, `getLHS`; this block applies conditional target rules.
- **CN**: 围绕 `assert`, `getExpr`, `getLHS` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 126-138
```cpp
  assert(isa<MCSpecifierExpr>(Expr) || Expr->getKind() == MCExpr::SymbolRef);
  // Push fixup (all info is contained within)
  Fixups.push_back(
      MCFixup::create(0, MCOp.getExpr(), MCFixupKind(FixupKind(Expr))));
  return 0;
}

// Helper function to adjust P and Q bits on load and store instructions.
static unsigned adjustPqBits(const MCInst &Inst, unsigned Value,
                             unsigned PBitShift, unsigned QBitShift) {
  const MCOperand AluOp = Inst.getOperand(3);
  unsigned AluCode = AluOp.getImm();

```
- **EN**: Implements logic around `assert`, `push_back`, `create`, `adjustPqBits`, ...; this block returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `assert`, `push_back`, `create`, `adjustPqBits`, ... 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 139-147
```cpp
  // Set the P bit to one iff the immediate is nonzero and not a post-op
  // instruction.
  const MCOperand Op2 = Inst.getOperand(2);
  Value &= ~(1 << PBitShift);
  if (!LPAC::isPostOp(AluCode) &&
      ((Op2.isImm() && Op2.getImm() != 0) ||
       (Op2.isReg() && Op2.getReg() != Lanai::R0) || (Op2.isExpr())))
    Value |= (1 << PBitShift);

```
- **EN**: Implements logic around `getOperand`, `~`, `isImm`, `isReg`; this block applies conditional target rules.
- **CN**: 围绕 `getOperand`, `~`, `isImm`, `isReg` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 148-158
```cpp
  // Set the Q bit to one iff it is a post- or pre-op instruction.
  assert(Inst.getOperand(0).isReg() && Inst.getOperand(1).isReg() &&
         "Expected register operand.");
  Value &= ~(1 << QBitShift);
  if (LPAC::modifiesOp(AluCode) && ((Op2.isImm() && Op2.getImm() != 0) ||
                                    (Op2.isReg() && Op2.getReg() != Lanai::R0)))
    Value |= (1 << QBitShift);

  return Value;
}

```
- **EN**: Implements logic around `assert`, `~`, `isReg`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `assert`, `~`, `isReg` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 159-170
```cpp
unsigned
LanaiMCCodeEmitter::adjustPqBitsRmAndRrm(const MCInst &Inst, unsigned Value,
                                         const MCSubtargetInfo &STI) const {
  return adjustPqBits(Inst, Value, 17, 16);
}

unsigned
LanaiMCCodeEmitter::adjustPqBitsSpls(const MCInst &Inst, unsigned Value,
                                     const MCSubtargetInfo &STI) const {
  return adjustPqBits(Inst, Value, 11, 10);
}

```
- **EN**: Implements logic around `adjustPqBitsRmAndRrm`, `adjustPqBits`, `adjustPqBitsSpls`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `adjustPqBitsRmAndRrm`, `adjustPqBits`, `adjustPqBitsSpls` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 171-181
```cpp
void LanaiMCCodeEmitter::encodeInstruction(
    const MCInst &Inst, SmallVectorImpl<char> &CB,
    SmallVectorImpl<MCFixup> &Fixups,
    const MCSubtargetInfo &SubtargetInfo) const {
  // Get instruction encoding and emit it
  unsigned Value = getBinaryCodeForInstr(Inst, Fixups, SubtargetInfo);
  ++MCNumEmitted; // Keep track of the number of emitted insns.

  support::endian::write<uint32_t>(CB, Value, llvm::endianness::big);
}

```
- **EN**: Implements logic around `encodeInstruction`, `getBinaryCodeForInstr`, `write<uint32_t>`; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `encodeInstruction`, `getBinaryCodeForInstr`, `write<uint32_t>` 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 182-190
```cpp
// Encode Lanai Memory Operand
unsigned LanaiMCCodeEmitter::getRiMemoryOpValue(
    const MCInst &Inst, unsigned OpNo, SmallVectorImpl<MCFixup> &Fixups,
    const MCSubtargetInfo &SubtargetInfo) const {
  unsigned Encoding;
  const MCOperand Op1 = Inst.getOperand(OpNo + 0);
  const MCOperand Op2 = Inst.getOperand(OpNo + 1);
  const MCOperand AluOp = Inst.getOperand(OpNo + 2);

```
- **EN**: Implements logic around `getRiMemoryOpValue`, `getOperand`; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getRiMemoryOpValue`, `getOperand` 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 191-201
```cpp
  assert(Op1.isReg() && "First operand is not register.");
  assert((Op2.isImm() || Op2.isExpr()) &&
         "Second operand is neither an immediate nor an expression.");
  assert((LPAC::getAluOp(AluOp.getImm()) == LPAC::ADD) &&
         "Register immediate only supports addition operator");

  Encoding = (getLanaiRegisterNumbering(Op1.getReg()) << 18);
  if (Op2.isImm()) {
    assert(isInt<16>(Op2.getImm()) &&
           "Constant value truncated (limited to 16-bit)");

```
- **EN**: Implements logic around `assert`, `getLanaiRegisterNumbering`, `truncated`; this block applies conditional target rules.
- **CN**: 围绕 `assert`, `getLanaiRegisterNumbering`, `truncated` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 202-211
```cpp
    Encoding |= (Op2.getImm() & 0xffff);
    if (Op2.getImm() != 0) {
      if (LPAC::isPreOp(AluOp.getImm()))
        Encoding |= (0x3 << 16);
      if (LPAC::isPostOp(AluOp.getImm()))
        Encoding |= (0x1 << 16);
    }
  } else
    getMachineOpValue(Inst, Op2, Fixups, SubtargetInfo);

```
- **EN**: Implements logic around `getImm`, `getMachineOpValue`; this block applies conditional target rules; maps fixups or relocations.
- **CN**: 围绕 `getImm`, `getMachineOpValue` 实现具体逻辑；这一段应用条件化的目标规则，映射 fixup 或重定位。

### Lines 212-222
```cpp
  return Encoding;
}

unsigned LanaiMCCodeEmitter::getRrMemoryOpValue(
    const MCInst &Inst, unsigned OpNo, SmallVectorImpl<MCFixup> &Fixups,
    const MCSubtargetInfo &SubtargetInfo) const {
  unsigned Encoding;
  const MCOperand Op1 = Inst.getOperand(OpNo + 0);
  const MCOperand Op2 = Inst.getOperand(OpNo + 1);
  const MCOperand AluMCOp = Inst.getOperand(OpNo + 2);

```
- **EN**: Implements logic around `getRrMemoryOpValue`, `getOperand`; this block returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getRrMemoryOpValue`, `getOperand` 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 223-240
```cpp
  assert(Op1.isReg() && "First operand is not register.");
  Encoding = (getLanaiRegisterNumbering(Op1.getReg()) << 15);
  assert(Op2.isReg() && "Second operand is not register.");
  Encoding |= (getLanaiRegisterNumbering(Op2.getReg()) << 10);

  assert(AluMCOp.isImm() && "Third operator is not immediate.");
  // Set BBB
  unsigned AluOp = AluMCOp.getImm();
  Encoding |= LPAC::encodeLanaiAluCode(AluOp) << 5;
  // Set P and Q
  if (LPAC::isPreOp(AluOp))
    Encoding |= (0x3 << 8);
  if (LPAC::isPostOp(AluOp))
    Encoding |= (0x1 << 8);
  // Set JJJJ
  switch (LPAC::getAluOp(AluOp)) {
  case LPAC::SHL:
  case LPAC::SRL:
```
- **EN**: Implements logic around `assert`, `getLanaiRegisterNumbering`, `getImm`, `encodeLanaiAluCode`; this block uses `switch`-based dispatch; applies conditional target rules.
- **CN**: 围绕 `assert`, `getLanaiRegisterNumbering`, `getImm`, `encodeLanaiAluCode` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则。

### Lines 241-249
```cpp
    Encoding |= 0x10;
    break;
  case LPAC::SRA:
    Encoding |= 0x18;
    break;
  default:
    break;
  }

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 250-261
```cpp
  return Encoding;
}

unsigned
LanaiMCCodeEmitter::getSplsOpValue(const MCInst &Inst, unsigned OpNo,
                                   SmallVectorImpl<MCFixup> &Fixups,
                                   const MCSubtargetInfo &SubtargetInfo) const {
  unsigned Encoding;
  const MCOperand Op1 = Inst.getOperand(OpNo + 0);
  const MCOperand Op2 = Inst.getOperand(OpNo + 1);
  const MCOperand AluOp = Inst.getOperand(OpNo + 2);

```
- **EN**: Implements logic around `getSplsOpValue`, `getOperand`; this block returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getSplsOpValue`, `getOperand` 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 262-272
```cpp
  assert(Op1.isReg() && "First operand is not register.");
  assert((Op2.isImm() || Op2.isExpr()) &&
         "Second operand is neither an immediate nor an expression.");
  assert((LPAC::getAluOp(AluOp.getImm()) == LPAC::ADD) &&
         "Register immediate only supports addition operator");

  Encoding = (getLanaiRegisterNumbering(Op1.getReg()) << 12);
  if (Op2.isImm()) {
    assert(isInt<10>(Op2.getImm()) &&
           "Constant value truncated (limited to 10-bit)");

```
- **EN**: Implements logic around `assert`, `getLanaiRegisterNumbering`, `truncated`; this block applies conditional target rules.
- **CN**: 围绕 `assert`, `getLanaiRegisterNumbering`, `truncated` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 273-282
```cpp
    Encoding |= (Op2.getImm() & 0x3ff);
    if (Op2.getImm() != 0) {
      if (LPAC::isPreOp(AluOp.getImm()))
        Encoding |= (0x3 << 10);
      if (LPAC::isPostOp(AluOp.getImm()))
        Encoding |= (0x1 << 10);
    }
  } else
    getMachineOpValue(Inst, Op2, Fixups, SubtargetInfo);

```
- **EN**: Implements logic around `getImm`, `getMachineOpValue`; this block applies conditional target rules; maps fixups or relocations.
- **CN**: 围绕 `getImm`, `getMachineOpValue` 实现具体逻辑；这一段应用条件化的目标规则，映射 fixup 或重定位。

### Lines 283-292
```cpp
  return Encoding;
}

unsigned LanaiMCCodeEmitter::getBranchTargetOpValue(
    const MCInst &Inst, unsigned OpNo, SmallVectorImpl<MCFixup> &Fixups,
    const MCSubtargetInfo &SubtargetInfo) const {
  const MCOperand &MCOp = Inst.getOperand(OpNo);
  if (MCOp.isReg() || MCOp.isImm())
    return getMachineOpValue(Inst, MCOp, Fixups, SubtargetInfo);

```
- **EN**: Implements logic around `getBranchTargetOpValue`, `getOperand`, `getMachineOpValue`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getBranchTargetOpValue`, `getOperand`, `getMachineOpValue` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 293-301
```cpp
  Fixups.push_back(MCFixup::create(0, MCOp.getExpr(), Lanai::FIXUP_LANAI_25));

  return 0;
}

#include "LanaiGenMCCodeEmitter.inc"

} // end namespace llvm

```
- **EN**: Pulls in the headers needed for this implementation, including `LanaiGenMCCodeEmitter.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `LanaiGenMCCodeEmitter.inc`。

### Lines 302-306
```cpp
llvm::MCCodeEmitter *
llvm::createLanaiMCCodeEmitter(const MCInstrInfo &InstrInfo,
                               MCContext &context) {
  return new LanaiMCCodeEmitter(InstrInfo, context);
}
```
- **EN**: Implements logic around `createLanaiMCCodeEmitter`, `LanaiMCCodeEmitter`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `createLanaiMCCodeEmitter`, `LanaiMCCodeEmitter` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

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

- **Direct includes / 直接包含**: `LanaiAluCode.h`, `MCTargetDesc/LanaiBaseInfo.h`, `MCTargetDesc/LanaiFixupKinds.h`, `MCTargetDesc/LanaiMCAsmInfo.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/MC/MCCodeEmitter.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCFixup.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCSubtargetInfo.h` ... (+5 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support
