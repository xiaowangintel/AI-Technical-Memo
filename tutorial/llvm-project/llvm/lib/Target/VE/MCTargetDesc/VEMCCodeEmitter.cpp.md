# VEMCCodeEmitter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/MCTargetDesc/VEMCCodeEmitter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Provides MC-layer target descriptions such as fixups, asm info, code emission, object writing, and target registration.
  - **CN**: 提供 MC 层的目标描述，例如 fixup、汇编信息、编码发射、目标文件写出以及目标注册。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- VEMCCodeEmitter.cpp - Convert VE code to machine code -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-21
```cpp
//
// This file implements the VEMCCodeEmitter class.
//
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/VEFixupKinds.h"
#include "VE.h"
#include "VEMCAsmInfo.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/MC/MCCodeEmitter.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCFixup.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `MCTargetDesc/VEFixupKinds.h`, `VE.h`, `VEMCAsmInfo.h`, `llvm/ADT/SmallVector.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MCTargetDesc/VEFixupKinds.h`, `VE.h`, `VEMCAsmInfo.h`, `llvm/ADT/SmallVector.h`。

### Lines 22-30
```cpp
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/Support/EndianStream.h"
#include <cassert>
#include <cstdint>

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCInst.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCRegisterInfo.h`, `llvm/MC/MCSubtargetInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCInst.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCRegisterInfo.h`, `llvm/MC/MCSubtargetInfo.h`。

### Lines 31-38
```cpp
using namespace llvm;

#define DEBUG_TYPE "mccodeemitter"

STATISTIC(MCNumEmitted, "Number of MC instructions emitted");

namespace {

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 39-48
```cpp
class VEMCCodeEmitter : public MCCodeEmitter {
  MCContext &Ctx;

public:
  VEMCCodeEmitter(const MCInstrInfo &, MCContext &ctx)
      : Ctx(ctx) {}
  VEMCCodeEmitter(const VEMCCodeEmitter &) = delete;
  VEMCCodeEmitter &operator=(const VEMCCodeEmitter &) = delete;
  ~VEMCCodeEmitter() override = default;

```
- **EN**: Introduces declarations for `VEMCCodeEmitter`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `VEMCCodeEmitter` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 49-58
```cpp
  void encodeInstruction(const MCInst &MI, SmallVectorImpl<char> &CB,
                         SmallVectorImpl<MCFixup> &Fixups,
                         const MCSubtargetInfo &STI) const override;

  // getBinaryCodeForInstr - TableGen'erated function for getting the
  // binary encoding for an instruction.
  uint64_t getBinaryCodeForInstr(const MCInst &MI,
                                 SmallVectorImpl<MCFixup> &Fixups,
                                 const MCSubtargetInfo &STI) const;

```
- **EN**: Implements logic around `encodeInstruction`, `getBinaryCodeForInstr`; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `encodeInstruction`, `getBinaryCodeForInstr` 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 59-72
```cpp
  /// getMachineOpValue - Return binary encoding of operand. If the machine
  /// operand requires relocation, record the relocation and return zero.
  unsigned getMachineOpValue(const MCInst &MI, const MCOperand &MO,
                             SmallVectorImpl<MCFixup> &Fixups,
                             const MCSubtargetInfo &STI) const;

  uint64_t getBranchTargetOpValue(const MCInst &MI, unsigned OpNo,
                                  SmallVectorImpl<MCFixup> &Fixups,
                                  const MCSubtargetInfo &STI) const;
  uint64_t getCCOpValue(const MCInst &MI, unsigned OpNo,
                        SmallVectorImpl<MCFixup> &Fixups,
                        const MCSubtargetInfo &STI) const;
  uint64_t getRDOpValue(const MCInst &MI, unsigned OpNo,
                        SmallVectorImpl<MCFixup> &Fixups,
```
- **EN**: Implements logic around `getMachineOpValue`, `getBranchTargetOpValue`, `getCCOpValue`, `getRDOpValue`; this block returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getMachineOpValue`, `getBranchTargetOpValue`, `getCCOpValue`, `getRDOpValue` 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 73-86
```cpp
                        const MCSubtargetInfo &STI) const;
};

} // end anonymous namespace

static void addFixup(SmallVectorImpl<MCFixup> &Fixups, uint32_t Offset,
                     const MCExpr *Value, uint16_t Kind) {
  bool PCRel = false;
  switch (Kind) {
  case VE::fixup_ve_srel32:
  case VE::fixup_ve_pc_hi32:
  case VE::fixup_ve_pc_lo32:
    PCRel = true;
  }
```
- **EN**: Implements logic around `addFixup`; this block uses `switch`-based dispatch; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `addFixup` 实现具体逻辑；这一段使用 `switch` 分派，映射 fixup 或重定位，工作在 MC 层。

### Lines 87-96
```cpp
  Fixups.push_back(MCFixup::create(Offset, Value, Kind, PCRel));
}

void VEMCCodeEmitter::encodeInstruction(const MCInst &MI,
                                        SmallVectorImpl<char> &CB,
                                        SmallVectorImpl<MCFixup> &Fixups,
                                        const MCSubtargetInfo &STI) const {
  uint64_t Bits = getBinaryCodeForInstr(MI, Fixups, STI);
  support::endian::write<uint64_t>(CB, Bits, llvm::endianness::little);

```
- **EN**: Implements logic around `push_back`, `encodeInstruction`, `getBinaryCodeForInstr`, `write<uint64_t>`; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `push_back`, `encodeInstruction`, `getBinaryCodeForInstr`, `write<uint64_t>` 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 97-108
```cpp
  ++MCNumEmitted; // Keep track of the # of mi's emitted.
}

unsigned VEMCCodeEmitter::getMachineOpValue(const MCInst &MI,
                                            const MCOperand &MO,
                                            SmallVectorImpl<MCFixup> &Fixups,
                                            const MCSubtargetInfo &STI) const {
  if (MO.isReg())
    return Ctx.getRegisterInfo()->getEncodingValue(MO.getReg());
  if (MO.isImm())
    return static_cast<unsigned>(MO.getImm());

```
- **EN**: Implements logic around `getMachineOpValue`, `getRegisterInfo`, `static_cast<unsigned>`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getMachineOpValue`, `getRegisterInfo`, `static_cast<unsigned>` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 109-117
```cpp
  assert(MO.isExpr());

  const MCExpr *Expr = MO.getExpr();
  if (const auto *SExpr = dyn_cast<MCSpecifierExpr>(Expr)) {
    auto Kind = VE::getFixupKind(SExpr->getSpecifier());
    addFixup(Fixups, 0, Expr, Kind);
    return 0;
  }

```
- **EN**: Implements logic around `assert`, `getExpr`, `getFixupKind`, `addFixup`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `assert`, `getExpr`, `getFixupKind`, `addFixup` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 118-125
```cpp
  int64_t Res;
  if (Expr->evaluateAsAbsolute(Res))
    return Res;

  llvm_unreachable("Unhandled expression!");
  return 0;
}

```
- **EN**: Implements logic around `llvm_unreachable`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `llvm_unreachable` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 126-133
```cpp
uint64_t
VEMCCodeEmitter::getBranchTargetOpValue(const MCInst &MI, unsigned OpNo,
                                        SmallVectorImpl<MCFixup> &Fixups,
                                        const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isReg() || MO.isImm())
    return getMachineOpValue(MI, MO, Fixups, STI);

```
- **EN**: Implements logic around `getBranchTargetOpValue`, `getOperand`, `getMachineOpValue`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getBranchTargetOpValue`, `getOperand`, `getMachineOpValue` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 134-147
```cpp
  addFixup(Fixups, 0, MO.getExpr(), VE::fixup_ve_srel32);
  return 0;
}

uint64_t VEMCCodeEmitter::getCCOpValue(const MCInst &MI, unsigned OpNo,
                                       SmallVectorImpl<MCFixup> &Fixups,
                                       const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isImm())
    return VECondCodeToVal(
        static_cast<VECC::CondCode>(getMachineOpValue(MI, MO, Fixups, STI)));
  return 0;
}

```
- **EN**: Implements logic around `addFixup`, `getCCOpValue`, `getOperand`, `VECondCodeToVal`, ...; this block applies conditional target rules; returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `addFixup`, `getCCOpValue`, `getOperand`, `VECondCodeToVal`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 148-157
```cpp
uint64_t VEMCCodeEmitter::getRDOpValue(const MCInst &MI, unsigned OpNo,
                                       SmallVectorImpl<MCFixup> &Fixups,
                                       const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isImm())
    return VERDToVal(static_cast<VERD::RoundingMode>(
        getMachineOpValue(MI, MO, Fixups, STI)));
  return 0;
}

```
- **EN**: Implements logic around `getRDOpValue`, `getOperand`, `VERDToVal`, `getMachineOpValue`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getRDOpValue`, `getOperand`, `VERDToVal`, `getMachineOpValue` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 158-163
```cpp
#include "VEGenMCCodeEmitter.inc"

MCCodeEmitter *llvm::createVEMCCodeEmitter(const MCInstrInfo &MCII,
                                           MCContext &Ctx) {
  return new VEMCCodeEmitter(MCII, Ctx);
}
```
- **EN**: Pulls in the headers needed for this implementation, including `VEGenMCCodeEmitter.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `VEGenMCCodeEmitter.inc`。

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

- **Direct includes / 直接包含**: `MCTargetDesc/VEFixupKinds.h`, `VE.h`, `VEMCAsmInfo.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/MC/MCCodeEmitter.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCFixup.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCRegisterInfo.h` ... (+6 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support
