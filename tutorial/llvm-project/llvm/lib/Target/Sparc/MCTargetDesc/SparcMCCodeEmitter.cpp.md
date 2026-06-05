# SparcMCCodeEmitter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/MCTargetDesc/SparcMCCodeEmitter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Provides MC-layer target descriptions such as fixups, asm info, code emission, object writing, and target registration.
  - **CN**: 提供 MC 层的目标描述，例如 fixup、汇编信息、编码发射、目标文件写出以及目标注册。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- SparcMCCodeEmitter.cpp - Convert Sparc code to machine code -------===//
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
// This file implements the SparcMCCodeEmitter class.
//
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/SparcFixupKinds.h"
#include "SparcMCTargetDesc.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCCodeEmitter.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `MCTargetDesc/SparcFixupKinds.h`, `SparcMCTargetDesc.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MCTargetDesc/SparcFixupKinds.h`, `SparcMCTargetDesc.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`。

### Lines 22-33
```cpp
#include "llvm/MC/MCFixup.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCObjectFileInfo.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/EndianStream.h"
#include <cassert>
#include <cstdint>

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCFixup.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCObjectFileInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCFixup.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCObjectFileInfo.h`。

### Lines 34-41
```cpp
using namespace llvm;

#define DEBUG_TYPE "mccodeemitter"

STATISTIC(MCNumEmitted, "Number of MC instructions emitted");

namespace {

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 42-51
```cpp
class SparcMCCodeEmitter : public MCCodeEmitter {
  MCContext &Ctx;

public:
  SparcMCCodeEmitter(const MCInstrInfo &, MCContext &ctx)
      : Ctx(ctx) {}
  SparcMCCodeEmitter(const SparcMCCodeEmitter &) = delete;
  SparcMCCodeEmitter &operator=(const SparcMCCodeEmitter &) = delete;
  ~SparcMCCodeEmitter() override = default;

```
- **EN**: Introduces declarations for `SparcMCCodeEmitter`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `SparcMCCodeEmitter` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 52-61
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

### Lines 62-75
```cpp
  /// getMachineOpValue - Return binary encoding of operand. If the machine
  /// operand requires relocation, record the relocation and return zero.
  unsigned getMachineOpValue(const MCInst &MI, const MCOperand &MO,
                             SmallVectorImpl<MCFixup> &Fixups,
                             const MCSubtargetInfo &STI) const;
  unsigned getCallTargetOpValue(const MCInst &MI, unsigned OpNo,
                             SmallVectorImpl<MCFixup> &Fixups,
                             const MCSubtargetInfo &STI) const;
  unsigned getBranchTargetOpValue(const MCInst &MI, unsigned OpNo,
                             SmallVectorImpl<MCFixup> &Fixups,
                             const MCSubtargetInfo &STI) const;
  unsigned getSImm5OpValue(const MCInst &MI, unsigned OpNo,
                           SmallVectorImpl<MCFixup> &Fixups,
                           const MCSubtargetInfo &STI) const;
```
- **EN**: Implements logic around `getMachineOpValue`, `getCallTargetOpValue`, `getBranchTargetOpValue`, `getSImm5OpValue`; this block returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getMachineOpValue`, `getCallTargetOpValue`, `getBranchTargetOpValue`, `getSImm5OpValue` 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 76-89
```cpp
  unsigned getSImm13OpValue(const MCInst &MI, unsigned OpNo,
                            SmallVectorImpl<MCFixup> &Fixups,
                            const MCSubtargetInfo &STI) const;
  unsigned getBranchPredTargetOpValue(const MCInst &MI, unsigned OpNo,
                                      SmallVectorImpl<MCFixup> &Fixups,
                                      const MCSubtargetInfo &STI) const;
  unsigned getBranchOnRegTargetOpValue(const MCInst &MI, unsigned OpNo,
                                       SmallVectorImpl<MCFixup> &Fixups,
                                       const MCSubtargetInfo &STI) const;
  unsigned getCompareAndBranchTargetOpValue(const MCInst &MI, unsigned OpNo,
                                            SmallVectorImpl<MCFixup> &Fixups,
                                            const MCSubtargetInfo &STI) const;
};

```
- **EN**: Implements logic around `getSImm13OpValue`, `getBranchPredTargetOpValue`, `getBranchOnRegTargetOpValue`, `getCompareAndBranchTargetOpValue`; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getSImm13OpValue`, `getBranchPredTargetOpValue`, `getBranchOnRegTargetOpValue`, `getCompareAndBranchTargetOpValue` 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 90-103
```cpp
} // end anonymous namespace

static void addFixup(SmallVectorImpl<MCFixup> &Fixups, uint32_t Offset,
                     const MCExpr *Value, uint16_t Kind) {
  bool PCRel = false;
  switch (Kind) {
  case ELF::R_SPARC_PC10:
  case ELF::R_SPARC_PC22:
  case ELF::R_SPARC_WDISP10:
  case ELF::R_SPARC_WDISP16:
  case ELF::R_SPARC_WDISP19:
  case ELF::R_SPARC_WDISP22:
  case Sparc::fixup_sparc_call30:
    PCRel = true;
```
- **EN**: Implements logic around `addFixup`; this block uses `switch`-based dispatch; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `addFixup` 实现具体逻辑；这一段使用 `switch` 分派，映射 fixup 或重定位，工作在 MC 层。

### Lines 104-117
```cpp
  }
  Fixups.push_back(MCFixup::create(Offset, Value, Kind, PCRel));
}

void SparcMCCodeEmitter::encodeInstruction(const MCInst &MI,
                                           SmallVectorImpl<char> &CB,
                                           SmallVectorImpl<MCFixup> &Fixups,
                                           const MCSubtargetInfo &STI) const {
  unsigned Bits = getBinaryCodeForInstr(MI, Fixups, STI);
  support::endian::write(CB, Bits,
                         Ctx.getAsmInfo().isLittleEndian()
                             ? llvm::endianness::little
                             : llvm::endianness::big);

```
- **EN**: Implements logic around `push_back`, `encodeInstruction`, `getBinaryCodeForInstr`, `write`, ...; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `push_back`, `encodeInstruction`, `getBinaryCodeForInstr`, `write`, ... 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 118-131
```cpp
  // Some instructions have phantom operands that only contribute a fixup entry.
  unsigned SymOpNo = 0;
  switch (MI.getOpcode()) {
  default: break;
  case SP::TLS_CALL:   SymOpNo = 1; break;
  case SP::GDOP_LDrr:
  case SP::GDOP_LDXrr:
  case SP::TLS_ADDrr:
  case SP::TLS_LDrr:
  case SP::TLS_LDXrr:  SymOpNo = 3; break;
  }
  if (SymOpNo != 0) {
    const MCOperand &MO = MI.getOperand(SymOpNo);
    uint64_t op = getMachineOpValue(MI, MO, Fixups, STI);
```
- **EN**: Implements logic around `getOperand`, `getMachineOpValue`; this block uses `switch`-based dispatch; applies conditional target rules; maps fixups or relocations.
- **CN**: 围绕 `getOperand`, `getMachineOpValue` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，映射 fixup 或重定位。

### Lines 132-138
```cpp
    assert(op == 0 && "Unexpected operand value!");
    (void)op; // suppress warning.
  }

  ++MCNumEmitted;  // Keep track of the # of mi's emitted.
}

```
- **EN**: Implements logic around `assert`.
- **CN**: 围绕 `assert` 实现具体逻辑。

### Lines 139-145
```cpp
unsigned SparcMCCodeEmitter::
getMachineOpValue(const MCInst &MI, const MCOperand &MO,
                  SmallVectorImpl<MCFixup> &Fixups,
                  const MCSubtargetInfo &STI) const {
  if (MO.isReg())
    return Ctx.getRegisterInfo()->getEncodingValue(MO.getReg());

```
- **EN**: Implements logic around `getMachineOpValue`, `getRegisterInfo`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getMachineOpValue`, `getRegisterInfo` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 146-155
```cpp
  if (MO.isImm())
    return MO.getImm();

  assert(MO.isExpr());
  const MCExpr *Expr = MO.getExpr();
  if (auto *SExpr = dyn_cast<MCSpecifierExpr>(Expr)) {
    addFixup(Fixups, 0, Expr, SExpr->getSpecifier());
    return 0;
  }

```
- **EN**: Implements logic around `getImm`, `assert`, `getExpr`, `addFixup`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `getImm`, `assert`, `getExpr`, `addFixup` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 156-163
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

### Lines 164-171
```cpp
unsigned SparcMCCodeEmitter::getSImm5OpValue(const MCInst &MI, unsigned OpNo,
                                             SmallVectorImpl<MCFixup> &Fixups,
                                             const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);

  if (MO.isImm())
    return MO.getImm();

```
- **EN**: Implements logic around `getSImm5OpValue`, `getOperand`, `getImm`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getSImm5OpValue`, `getOperand`, `getImm` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 172-180
```cpp
  assert(MO.isExpr() &&
         "getSImm5OpValue expects only expressions or an immediate");

  const MCExpr *Expr = MO.getExpr();

  // Constant value, no fixup is needed
  if (const MCConstantExpr *CE = dyn_cast<MCConstantExpr>(Expr))
    return CE->getValue();

```
- **EN**: Implements logic around `assert`, `getExpr`, `getValue`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `assert`, `getExpr`, `getValue` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 181-188
```cpp
  if (auto *SExpr = dyn_cast<MCSpecifierExpr>(Expr)) {
    addFixup(Fixups, 0, Expr, SExpr->getSpecifier());
    return 0;
  }
  addFixup(Fixups, 0, Expr, ELF::R_SPARC_5);
  return 0;
}

```
- **EN**: Implements logic around `addFixup`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `addFixup` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 189-197
```cpp
unsigned
SparcMCCodeEmitter::getSImm13OpValue(const MCInst &MI, unsigned OpNo,
                                     SmallVectorImpl<MCFixup> &Fixups,
                                     const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);

  if (MO.isImm())
    return MO.getImm();

```
- **EN**: Implements logic around `getSImm13OpValue`, `getOperand`, `getImm`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getSImm13OpValue`, `getOperand`, `getImm` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 198-206
```cpp
  assert(MO.isExpr() &&
         "getSImm13OpValue expects only expressions or an immediate");

  const MCExpr *Expr = MO.getExpr();

  // Constant value, no fixup is needed
  if (const MCConstantExpr *CE = dyn_cast<MCConstantExpr>(Expr))
    return CE->getValue();

```
- **EN**: Implements logic around `assert`, `getExpr`, `getValue`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `assert`, `getExpr`, `getValue` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 207-214
```cpp
  if (auto *SExpr = dyn_cast<MCSpecifierExpr>(Expr)) {
    addFixup(Fixups, 0, Expr, SExpr->getSpecifier());
    return 0;
  }
  addFixup(Fixups, 0, Expr, Sparc::fixup_sparc_13);
  return 0;
}

```
- **EN**: Implements logic around `addFixup`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `addFixup` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 215-224
```cpp
unsigned SparcMCCodeEmitter::
getCallTargetOpValue(const MCInst &MI, unsigned OpNo,
                     SmallVectorImpl<MCFixup> &Fixups,
                     const MCSubtargetInfo &STI) const {
  if (MI.getOpcode() == SP::TLS_CALL) {
    // No fixups for __tls_get_addr. Will emit for fixups for tls_symbol in
    // encodeInstruction.
    return 0;
  }

```
- **EN**: Implements logic around `getCallTargetOpValue`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getCallTargetOpValue` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 225-237
```cpp
  const MCOperand &MO = MI.getOperand(OpNo);
  addFixup(Fixups, 0, MO.getExpr(), Sparc::fixup_sparc_call30);
  return 0;
}

unsigned SparcMCCodeEmitter::
getBranchTargetOpValue(const MCInst &MI, unsigned OpNo,
                  SmallVectorImpl<MCFixup> &Fixups,
                  const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isReg() || MO.isImm())
    return getMachineOpValue(MI, MO, Fixups, STI);

```
- **EN**: Implements logic around `getOperand`, `addFixup`, `getBranchTargetOpValue`, `getMachineOpValue`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getOperand`, `addFixup`, `getBranchTargetOpValue`, `getMachineOpValue` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 238-248
```cpp
  addFixup(Fixups, 0, MO.getExpr(), ELF::R_SPARC_WDISP22);
  return 0;
}

unsigned SparcMCCodeEmitter::getBranchPredTargetOpValue(
    const MCInst &MI, unsigned OpNo, SmallVectorImpl<MCFixup> &Fixups,
    const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isReg() || MO.isImm())
    return getMachineOpValue(MI, MO, Fixups, STI);

```
- **EN**: Implements logic around `addFixup`, `getBranchPredTargetOpValue`, `getOperand`, `getMachineOpValue`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `addFixup`, `getBranchPredTargetOpValue`, `getOperand`, `getMachineOpValue` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 249-259
```cpp
  addFixup(Fixups, 0, MO.getExpr(), ELF::R_SPARC_WDISP19);
  return 0;
}

unsigned SparcMCCodeEmitter::getBranchOnRegTargetOpValue(
    const MCInst &MI, unsigned OpNo, SmallVectorImpl<MCFixup> &Fixups,
    const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isReg() || MO.isImm())
    return getMachineOpValue(MI, MO, Fixups, STI);

```
- **EN**: Implements logic around `addFixup`, `getBranchOnRegTargetOpValue`, `getOperand`, `getMachineOpValue`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `addFixup`, `getBranchOnRegTargetOpValue`, `getOperand`, `getMachineOpValue` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 260-270
```cpp
  addFixup(Fixups, 0, MO.getExpr(), ELF::R_SPARC_WDISP16);
  return 0;
}

unsigned SparcMCCodeEmitter::getCompareAndBranchTargetOpValue(
    const MCInst &MI, unsigned OpNo, SmallVectorImpl<MCFixup> &Fixups,
    const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isImm())
    return getMachineOpValue(MI, MO, Fixups, STI);

```
- **EN**: Implements logic around `addFixup`, `getCompareAndBranchTargetOpValue`, `getOperand`, `getMachineOpValue`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `addFixup`, `getCompareAndBranchTargetOpValue`, `getOperand`, `getMachineOpValue` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 271-280
```cpp
  addFixup(Fixups, 0, MO.getExpr(), ELF::R_SPARC_WDISP10);
  return 0;
}

#include "SparcGenMCCodeEmitter.inc"

MCCodeEmitter *llvm::createSparcMCCodeEmitter(const MCInstrInfo &MCII,
                                              MCContext &Ctx) {
  return new SparcMCCodeEmitter(MCII, Ctx);
}
```
- **EN**: Pulls in the headers needed for this implementation, including `SparcGenMCCodeEmitter.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `SparcGenMCCodeEmitter.inc`。

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

- **Direct includes / 直接包含**: `MCTargetDesc/SparcFixupKinds.h`, `SparcMCTargetDesc.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/BinaryFormat/ELF.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCCodeEmitter.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCFixup.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstrInfo.h` ... (+9 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support
