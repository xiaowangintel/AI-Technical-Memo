# PPCMCCodeEmitter.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/MCTargetDesc/PPCMCCodeEmitter.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides MC layer support for the backend for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/MCTargetDesc/PPCMCCodeEmitter.cpp`，主要负责 PowerPC 后端的该后端的 MC 层支持。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCMCCodeEmitter.cpp - Convert PPC code to machine code -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-10

```cpp
//===----------------------------------------------------------------------===//
//
// This file implements the PPCMCCodeEmitter class.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file implements the PPCMCCodeEmitter class.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file implements the PPCMCCodeEmitter class.”。

### Lines 11-17

```cpp
//===----------------------------------------------------------------------===//

#include "PPCMCCodeEmitter.h"
#include "MCTargetDesc/PPCFixupKinds.h"
#include "PPCMCAsmInfo.h"
#include "PPCMCTargetDesc.h"
#include "llvm/ADT/SmallVector.h"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 18-24

```cpp
#include "llvm/ADT/Statistic.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCFixup.h"
#include "llvm/MC/MCInstrDesc.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/EndianStream.h"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 25-31

```cpp
#include "llvm/Support/MathExtras.h"
#include "llvm/TargetParser/Triple.h"
#include <cassert>
#include <cstdint>

using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 32-69

```cpp
#define DEBUG_TYPE "mccodeemitter"

STATISTIC(MCNumEmitted, "Number of MC instructions emitted");

MCCodeEmitter *llvm::createPPCMCCodeEmitter(const MCInstrInfo &MCII,
                                            MCContext &Ctx) {
  return new PPCMCCodeEmitter(MCII, Ctx);
}

static void addFixup(SmallVectorImpl<MCFixup> &Fixups, uint32_t Offset,
                     const MCExpr *Value, uint16_t Kind) {
  bool PCRel = false;
  switch (Kind) {
  case PPC::fixup_ppc_br24:
  case PPC::fixup_ppc_br24_notoc:
  case PPC::fixup_ppc_brcond14:
  case PPC::fixup_ppc_pcrel34:
  case PPC::fixup_ppc_pcrel32:
    PCRel = true;
  }
  Fixups.push_back(MCFixup::create(Offset, Value, Kind, PCRel));
}

unsigned PPCMCCodeEmitter::
getDirectBrEncoding(const MCInst &MI, unsigned OpNo,
                    SmallVectorImpl<MCFixup> &Fixups,
                    const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);

  if (MO.isReg() || MO.isImm())
    return getMachineOpValue(MI, MO, Fixups, STI);

  // Add a fixup for the branch target.
  addFixup(
      Fixups, 0, MO.getExpr(),
      (isNoTOCCallInstr(MI) ? PPC::fixup_ppc_br24_notoc : PPC::fixup_ppc_br24));
  return 0;
}
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Subtarget feature gating influences the behavior here.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 子目标特性裁剪会影响这里的行为。

### Lines 70-79

```cpp

/// Check if Opcode corresponds to a call instruction that should be marked
/// with the NOTOC relocation.
bool PPCMCCodeEmitter::isNoTOCCallInstr(const MCInst &MI) const {
  unsigned Opcode = MI.getOpcode();
  if (!MCII.get(Opcode).isCall())
    return false;

  switch (Opcode) {
  default:
```
- **EN**: Implements helper routine(s) `isNoTOCCallInstr`, `getOpcode`, `get` for this portion of the PowerPC backend MC layer support for the backend. The logic interacts with LLVM's MC layer.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `isNoTOCCallInstr`, `getOpcode`, `get`。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 80-87

```cpp
#ifndef NDEBUG
    llvm_unreachable("Unknown call opcode");
#endif
    return false;
  case PPC::BL8_NOTOC:
  case PPC::BL8_NOTOC_TLS:
  case PPC::BL8_NOTOC_RM:
    return true;
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `llvm_unreachable`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `llvm_unreachable`。

### Lines 88-125

```cpp
#ifndef NDEBUG
  case PPC::BL8:
  case PPC::BL:
  case PPC::BL8_TLS:
  case PPC::BL_TLS:
  case PPC::BLA8:
  case PPC::BLA:
  case PPC::BCCL:
  case PPC::BCCLA:
  case PPC::BCL:
  case PPC::BCLn:
  case PPC::BL8_NOP:
  case PPC::BL_NOP:
  case PPC::BL8_NOP_TLS:
  case PPC::BLA8_NOP:
  case PPC::BCTRL8:
  case PPC::BCTRL:
  case PPC::BCCCTRL8:
  case PPC::BCCCTRL:
  case PPC::BCCTRL8:
  case PPC::BCCTRL:
  case PPC::BCCTRL8n:
  case PPC::BCCTRLn:
  case PPC::BL8_RM:
  case PPC::BLA8_RM:
  case PPC::BL8_NOP_RM:
  case PPC::BLA8_NOP_RM:
  case PPC::BCTRL8_RM:
  case PPC::BCTRL8_LDinto_toc:
  case PPC::BCTRL8_LDinto_toc_RM:
  case PPC::BL8_TLS_:
  case PPC::TCRETURNdi8:
  case PPC::TCRETURNai8:
  case PPC::TCRETURNri8:
  case PPC::TAILBCTR8:
  case PPC::TAILB8:
  case PPC::TAILBA8:
  case PPC::BCLalways:
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

### Lines 126-163

```cpp
  case PPC::BLRL:
  case PPC::BCCLRL:
  case PPC::BCLRL:
  case PPC::BCLRLn:
  case PPC::BDZL:
  case PPC::BDNZL:
  case PPC::BDZLA:
  case PPC::BDNZLA:
  case PPC::BDZLp:
  case PPC::BDNZLp:
  case PPC::BDZLAp:
  case PPC::BDNZLAp:
  case PPC::BDZLm:
  case PPC::BDNZLm:
  case PPC::BDZLAm:
  case PPC::BDNZLAm:
  case PPC::BDZLRL:
  case PPC::BDNZLRL:
  case PPC::BDZLRLp:
  case PPC::BDNZLRLp:
  case PPC::BDZLRLm:
  case PPC::BDNZLRLm:
  case PPC::BL_RM:
  case PPC::BLA_RM:
  case PPC::BL_NOP_RM:
  case PPC::BCTRL_RM:
  case PPC::TCRETURNdi:
  case PPC::TCRETURNai:
  case PPC::TCRETURNri:
  case PPC::BCTRL_LWZinto_toc:
  case PPC::BCTRL_LWZinto_toc_RM:
  case PPC::BL_LWZinto_toc:
  case PPC::BL_LWZinto_toc_RM:
  case PPC::BL8_LDinto_toc:
  case PPC::BL8_LDinto_toc_RM:
  case PPC::TAILBCTR:
  case PPC::TAILB:
  case PPC::TAILBA:
```
- **EN**: Implements dispatch logic that chooses specialized target behavior for different opcodes, modes, or ABI cases.
- **CN**: 这一段实现分派逻辑，为不同操作码、模式或 ABI 情况选择特定的目标行为。

### Lines 164-201

```cpp
    return false;
#endif
  }
}

unsigned PPCMCCodeEmitter::getCondBrEncoding(const MCInst &MI, unsigned OpNo,
                                     SmallVectorImpl<MCFixup> &Fixups,
                                     const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isReg() || MO.isImm()) return getMachineOpValue(MI, MO, Fixups, STI);

  // Add a fixup for the branch target.
  addFixup(Fixups, 0, MO.getExpr(), PPC::fixup_ppc_brcond14);
  return 0;
}

unsigned PPCMCCodeEmitter::
getAbsDirectBrEncoding(const MCInst &MI, unsigned OpNo,
                       SmallVectorImpl<MCFixup> &Fixups,
                       const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isReg() || MO.isImm()) return getMachineOpValue(MI, MO, Fixups, STI);

  // Add a fixup for the branch target.
  addFixup(Fixups, 0, MO.getExpr(), PPC::fixup_ppc_br24abs);
  return 0;
}

unsigned PPCMCCodeEmitter::
getAbsCondBrEncoding(const MCInst &MI, unsigned OpNo,
                     SmallVectorImpl<MCFixup> &Fixups,
                     const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isReg() || MO.isImm()) return getMachineOpValue(MI, MO, Fixups, STI);

  // Add a fixup for the branch target.
  addFixup(Fixups, 0, MO.getExpr(), PPC::fixup_ppc_brcond14abs);
  return 0;
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Subtarget feature gating influences the behavior here.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 子目标特性裁剪会影响这里的行为。

### Lines 202-239

```cpp
}

unsigned
PPCMCCodeEmitter::getVSRpEvenEncoding(const MCInst &MI, unsigned OpNo,
                                      SmallVectorImpl<MCFixup> &Fixups,
                                      const MCSubtargetInfo &STI) const {
  assert(MI.getOperand(OpNo).isReg() && "Operand should be a register");
  unsigned RegBits = getMachineOpValue(MI, MI.getOperand(OpNo), Fixups, STI)
                     << 1;
  return RegBits;
}

template <MCFixupKind Fixup>
uint64_t PPCMCCodeEmitter::getImmEncoding(const MCInst &MI, unsigned OpNo,
                                          SmallVectorImpl<MCFixup> &Fixups,
                                          const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  assert(!MO.isReg() && "Not expecting a register for this operand.");
  if (MO.isImm())
    return getMachineOpValue(MI, MO, Fixups, STI);

  uint32_t Offset = 0;
  if (Fixup == PPC::fixup_ppc_half16)
    Offset = IsLittleEndian ? 0 : 2;

  // Add a fixup for the immediate field.
  addFixup(Fixups, Offset, MO.getExpr(), Fixup);
  return 0;
}

unsigned PPCMCCodeEmitter::getDispRIEncoding(const MCInst &MI, unsigned OpNo,
                                             SmallVectorImpl<MCFixup> &Fixups,
                                             const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isImm())
    return getMachineOpValue(MI, MO, Fixups, STI) & 0xFFFF;

  // Add a fixup for the displacement field.
```
- **EN**: Implements helper routine(s) `getVSRpEvenEncoding`, `getOperand`, `isReg` for this portion of the PowerPC backend MC layer support for the backend. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `getVSRpEvenEncoding`, `getOperand`, `isReg`。 子目标特性裁剪会影响这里的行为。

### Lines 240-277

```cpp
  addFixup(Fixups, IsLittleEndian ? 0 : 2, MO.getExpr(), PPC::fixup_ppc_half16);
  return 0;
}

unsigned
PPCMCCodeEmitter::getDispRIXEncoding(const MCInst &MI, unsigned OpNo,
                                     SmallVectorImpl<MCFixup> &Fixups,
                                     const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isImm())
    return ((getMachineOpValue(MI, MO, Fixups, STI) >> 2) & 0x3FFF);

  // Add a fixup for the displacement field.
  addFixup(Fixups, IsLittleEndian ? 0 : 2, MO.getExpr(),
           PPC::fixup_ppc_half16ds);
  return 0;
}

unsigned
PPCMCCodeEmitter::getDispRIX16Encoding(const MCInst &MI, unsigned OpNo,
                                       SmallVectorImpl<MCFixup> &Fixups,
                                       const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isImm()) {
    assert(!(MO.getImm() % 16) &&
           "Expecting an immediate that is a multiple of 16");
    return ((getMachineOpValue(MI, MO, Fixups, STI) >> 4) & 0xFFF);
  }

  // Otherwise add a fixup for the displacement field.
  addFixup(Fixups, IsLittleEndian ? 0 : 2, MO.getExpr(),
           PPC::fixup_ppc_half16dq);
  return 0;
}

unsigned
PPCMCCodeEmitter::getDispRIHashEncoding(const MCInst &MI, unsigned OpNo,
                                        SmallVectorImpl<MCFixup> &Fixups,
```
- **EN**: Implements helper routine(s) `addFixup`, `getExpr`, `getDispRIXEncoding` for this portion of the PowerPC backend MC layer support for the backend. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `addFixup`, `getExpr`, `getDispRIXEncoding`。 子目标特性裁剪会影响这里的行为。

### Lines 278-315

```cpp
                                        const MCSubtargetInfo &STI) const {
  // Encode imm for the hash load/store to stack for the ROP Protection
  // instructions.
  const MCOperand &MO = MI.getOperand(OpNo);

  assert(MO.isImm() && "Expecting an immediate operand.");
  assert(!(MO.getImm() % 8) && "Expecting offset to be 8 byte aligned.");

  unsigned DX = (MO.getImm() >> 3) & 0x3F;
  return DX;
}

uint64_t
PPCMCCodeEmitter::getDispRI34PCRelEncoding(const MCInst &MI, unsigned OpNo,
                                           SmallVectorImpl<MCFixup> &Fixups,
                                           const MCSubtargetInfo &STI) const {
  // Encode the displacement part of pc-relative memri34, which is an imm34.
  // The 34 bit immediate can fall into one of three cases:
  // 1) It is a relocation to be filled in by the linker represented as:
  //    (MCExpr::SymbolRef)
  // 2) It is a relocation + SignedOffset represented as:
  //    (MCExpr::Binary(MCExpr::SymbolRef + MCExpr::Constant))
  // 3) It is a known value at compile time.

  // If this is not a MCExpr then we are in case 3) and we are dealing with
  // a value known at compile time, not a relocation.
  const MCOperand &MO = MI.getOperand(OpNo);
  if (!MO.isExpr())
    return (getMachineOpValue(MI, MO, Fixups, STI)) & 0x3FFFFFFFFUL;

  // At this point in the function it is known that MO is of type MCExpr.
  // Therefore we are dealing with either case 1) a symbol ref or
  // case 2) a symbol ref plus a constant.
  const MCExpr *Expr = MO.getExpr();
  switch (Expr->getKind()) {
  default:
    llvm_unreachable("Unsupported MCExpr for getMemRI34PCRelEncoding.");
  case MCExpr::SymbolRef: {
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Encode imm for the hash load/store to stack for the ROP Protection". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Encode imm for the hash load/store to stack for the ROP Protection”。 子目标特性裁剪会影响这里的行为。

### Lines 316-353

```cpp
    // Relocation alone.
    const MCSymbolRefExpr *SRE = cast<MCSymbolRefExpr>(Expr);
    (void)SRE;
    // Currently these are the only valid PCRelative Relocations.
    assert(is_contained({PPC::S_PCREL, PPC::S_GOT_PCREL, PPC::S_GOT_TLSGD_PCREL,
                         PPC::S_GOT_TLSLD_PCREL, PPC::S_GOT_TPREL_PCREL},
                        SRE->getSpecifier()) &&
           "specifier must be S_PCREL, S_GOT_PCREL, S_GOT_TLSGD_PCREL, "
           "S_GOT_TLSLD_PCREL, or S_GOT_TPREL_PCREL");
    // Generate the fixup for the relocation.
    addFixup(Fixups, 0, Expr, PPC::fixup_ppc_pcrel34);
    // Put zero in the location of the immediate. The linker will fill in the
    // correct value based on the relocation.
    return 0;
  }
  case MCExpr::Binary: {
    // Relocation plus some offset.
    const MCBinaryExpr *BE = cast<MCBinaryExpr>(Expr);
    assert(BE->getOpcode() == MCBinaryExpr::Add &&
           "Binary expression opcode must be an add.");

    const MCExpr *LHS = BE->getLHS();
    const MCExpr *RHS = BE->getRHS();

    // Need to check in both directions. Reloc+Offset and Offset+Reloc.
    if (LHS->getKind() != MCExpr::SymbolRef)
      std::swap(LHS, RHS);

    if (LHS->getKind() != MCExpr::SymbolRef ||
        RHS->getKind() != MCExpr::Constant)
      llvm_unreachable("Expecting to have one constant and one relocation.");

    const MCSymbolRefExpr *SRE = cast<MCSymbolRefExpr>(LHS);
    (void)SRE;
    assert(isInt<34>(cast<MCConstantExpr>(RHS)->getValue()) &&
           "Value must fit in 34 bits.");

    // Currently these are the only valid PCRelative Relocations.
```
- **EN**: Implements helper routine(s) `is_contained`, `getSpecifier`, `addFixup` for this portion of the PowerPC backend MC layer support for the backend. The logic interacts with LLVM's MC layer.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `is_contained`, `getSpecifier`, `addFixup`。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 354-391

```cpp
    assert((getSpecifier(SRE) == PPC::S_PCREL ||
            getSpecifier(SRE) == PPC::S_GOT_PCREL) &&
           "VariantKind must be VK_PCREL or VK_GOT_PCREL");
    // Generate the fixup for the relocation.
    addFixup(Fixups, 0, Expr, PPC::fixup_ppc_pcrel34);
    // Put zero in the location of the immediate. The linker will fill in the
    // correct value based on the relocation.
    return 0;
    }
  }
}

uint64_t
PPCMCCodeEmitter::getDispRI34Encoding(const MCInst &MI, unsigned OpNo,
                                      SmallVectorImpl<MCFixup> &Fixups,
                                      const MCSubtargetInfo &STI) const {
  // Encode the displacement part of a memri34.
  const MCOperand &MO = MI.getOperand(OpNo);
  return (getMachineOpValue(MI, MO, Fixups, STI)) & 0x3FFFFFFFFUL;
}

unsigned
PPCMCCodeEmitter::getDispSPE8Encoding(const MCInst &MI, unsigned OpNo,
                                      SmallVectorImpl<MCFixup> &Fixups,
                                      const MCSubtargetInfo &STI) const {
  // Encode imm as a dispSPE8, which has the low 5-bits of (imm / 8).
  const MCOperand &MO = MI.getOperand(OpNo);
  assert(MO.isImm());
  return getMachineOpValue(MI, MO, Fixups, STI) >> 3;
}

unsigned
PPCMCCodeEmitter::getDispSPE4Encoding(const MCInst &MI, unsigned OpNo,
                                      SmallVectorImpl<MCFixup> &Fixups,
                                      const MCSubtargetInfo &STI) const {
  // Encode imm as a dispSPE8, which has the low 5-bits of (imm / 4).
  const MCOperand &MO = MI.getOperand(OpNo);
  assert(MO.isImm());
```
- **EN**: Implements helper routine(s) `getSpecifier`, `addFixup`, `getDispRI34Encoding` for this portion of the PowerPC backend MC layer support for the backend. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `getSpecifier`, `addFixup`, `getDispRI34Encoding`。 子目标特性裁剪会影响这里的行为。

### Lines 392-429

```cpp
  return getMachineOpValue(MI, MO, Fixups, STI) >> 2;
}

unsigned
PPCMCCodeEmitter::getDispSPE2Encoding(const MCInst &MI, unsigned OpNo,
                                      SmallVectorImpl<MCFixup> &Fixups,
                                      const MCSubtargetInfo &STI) const {
  // Encode imm as a dispSPE8, which has the low 5-bits of (imm / 2).
  const MCOperand &MO = MI.getOperand(OpNo);
  assert(MO.isImm());
  return getMachineOpValue(MI, MO, Fixups, STI) >> 1;
}

unsigned PPCMCCodeEmitter::getTLSRegEncoding(const MCInst &MI, unsigned OpNo,
                                       SmallVectorImpl<MCFixup> &Fixups,
                                       const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isReg()) return getMachineOpValue(MI, MO, Fixups, STI);

  // Add a fixup for the TLS register, which simply provides a relocation
  // hint to the linker that this statement is part of a relocation sequence.
  // Return the thread-pointer register's encoding. Add a one byte displacement
  // if using PC relative memops.
  const MCExpr *Expr = MO.getExpr();
  const MCSymbolRefExpr *SRE = cast<MCSymbolRefExpr>(Expr);
  bool IsPCRel = getSpecifier(SRE) == PPC::S_TLS_PCREL;
  addFixup(Fixups, IsPCRel ? 1 : 0, Expr, PPC::fixup_ppc_nofixup);
  const Triple &TT = STI.getTargetTriple();
  bool isPPC64 = TT.isPPC64();
  return CTX.getRegisterInfo()->getEncodingValue(isPPC64 ? PPC::X13 : PPC::R2);
}

unsigned PPCMCCodeEmitter::getTLSCallEncoding(const MCInst &MI, unsigned OpNo,
                                       SmallVectorImpl<MCFixup> &Fixups,
                                       const MCSubtargetInfo &STI) const {
  // For special TLS calls, we need two fixups; one for the branch target
  // (__tls_get_addr), which we create via getDirectBrEncoding as usual,
  // and one for the TLSGD or TLSLD symbol, which is emitted here.
```
- **EN**: Implements helper routine(s) `getMachineOpValue`, `getDispSPE2Encoding`, `of` for this portion of the PowerPC backend MC layer support for the backend. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `getMachineOpValue`, `getDispSPE2Encoding`, `of`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 430-467

```cpp
  const MCOperand &MO = MI.getOperand(OpNo+1);
  addFixup(Fixups, 0, MO.getExpr(), PPC::fixup_ppc_nofixup);
  return getDirectBrEncoding(MI, OpNo, Fixups, STI);
}

unsigned PPCMCCodeEmitter::
get_crbitm_encoding(const MCInst &MI, unsigned OpNo,
                    SmallVectorImpl<MCFixup> &Fixups,
                    const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  assert((MI.getOpcode() == PPC::MTOCRF || MI.getOpcode() == PPC::MTOCRF8 ||
          MI.getOpcode() == PPC::MFOCRF || MI.getOpcode() == PPC::MFOCRF8) &&
         (MO.getReg() >= PPC::CR0 && MO.getReg() <= PPC::CR7));
  return 0x80 >> CTX.getRegisterInfo()->getEncodingValue(MO.getReg());
}

// Get the index for this operand in this instruction. This is needed for
// computing the register number in PPC::getRegNumForOperand() for
// any instructions that use a different numbering scheme for registers in
// different operands.
static unsigned getOpIdxForMO(const MCInst &MI, const MCOperand &MO) {
  for (unsigned i = 0; i < MI.getNumOperands(); i++) {
    const MCOperand &Op = MI.getOperand(i);
    if (&Op == &MO)
      return i;
  }
  llvm_unreachable("This operand is not part of this instruction");
  return ~0U; // Silence any warnings about no return.
}

uint64_t PPCMCCodeEmitter::
getMachineOpValue(const MCInst &MI, const MCOperand &MO,
                  SmallVectorImpl<MCFixup> &Fixups,
                  const MCSubtargetInfo &STI) const {
  if (MO.isReg()) {
    // MTOCRF/MFOCRF should go through get_crbitm_encoding for the CR operand.
    // The GPR operand should come through here though.
    assert((MI.getOpcode() != PPC::MTOCRF && MI.getOpcode() != PPC::MTOCRF8 &&
```
- **EN**: Implements helper routine(s) `getOperand`, `addFixup`, `getExpr` for this portion of the PowerPC backend MC layer support for the backend. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `getOperand`, `addFixup`, `getExpr`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 468-505

```cpp
            MI.getOpcode() != PPC::MFOCRF && MI.getOpcode() != PPC::MFOCRF8) ||
           MO.getReg() < PPC::CR0 || MO.getReg() > PPC::CR7);
    unsigned OpNo = getOpIdxForMO(MI, MO);
    MCRegister Reg =
        PPC::getRegNumForOperand(MCII.get(MI.getOpcode()), MO.getReg(), OpNo);
    return CTX.getRegisterInfo()->getEncodingValue(Reg);
  }

  assert(MO.isImm() &&
         "Relocation required in an instruction that we cannot encode!");
  return MO.getImm();
}

void PPCMCCodeEmitter::encodeInstruction(const MCInst &MI,
                                         SmallVectorImpl<char> &CB,
                                         SmallVectorImpl<MCFixup> &Fixups,
                                         const MCSubtargetInfo &STI) const {
  uint64_t Bits = getBinaryCodeForInstr(MI, Fixups, STI);

  // Output the constant in big/little endian byte order.
  unsigned Size = getInstSizeInBytes(MI);
  llvm::endianness E =
      IsLittleEndian ? llvm::endianness::little : llvm::endianness::big;
  switch (Size) {
  case 0:
    break;
  case 4:
    support::endian::write<uint32_t>(CB, Bits, E);
    break;
  case 8:
    // If we emit a pair of instructions, the first one is
    // always in the top 32 bits, even on little-endian.
    support::endian::write<uint32_t>(CB, Bits >> 32, E);
    support::endian::write<uint32_t>(CB, Bits, E);
    break;
  default:
    llvm_unreachable("Invalid instruction size");
  }
```
- **EN**: Implements helper routine(s) `getOpcode`, `getReg`, `getOpIdxForMO` for this portion of the PowerPC backend MC layer support for the backend. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `getOpcode`, `getReg`, `getOpIdxForMO`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 506-520

```cpp

  ++MCNumEmitted; // Keep track of the # of mi's emitted.
}

// Get the number of bytes used to encode the given MCInst.
unsigned PPCMCCodeEmitter::getInstSizeInBytes(const MCInst &MI) const {
  unsigned Opcode = MI.getOpcode();
  const MCInstrDesc &Desc = MCII.get(Opcode);
  return Desc.getSize();
}

bool PPCMCCodeEmitter::isPrefixedInstruction(const MCInst &MI) const {
  return MCII.get(MI.getOpcode()).TSFlags & PPCII::Prefixed;
}
```
- **EN**: Implements helper routine(s) `getInstSizeInBytes`, `getOpcode`, `get` for this portion of the PowerPC backend MC layer support for the backend. The logic interacts with LLVM's MC layer.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `getInstSizeInBytes`, `getOpcode`, `get`。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 521-521

```cpp
#include "PPCGenMCCodeEmitter.inc"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- MC layer support for the backend / 该后端的 MC 层支持
- Register modeling / 寄存器建模
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPCMCCodeEmitter.h`
- `MCTargetDesc/PPCFixupKinds.h`
- `PPCMCAsmInfo.h`
- `PPCMCTargetDesc.h`
- `llvm/ADT/SmallVector.h`
- `llvm/ADT/Statistic.h`
- `llvm/MC/MCExpr.h`
- `llvm/MC/MCFixup.h`
- `llvm/MC/MCInstrDesc.h`
- `llvm/MC/MCRegisterInfo.h`
- `llvm/Support/Casting.h`
- `llvm/Support/EndianStream.h`
- `llvm/Support/MathExtras.h`
- `llvm/TargetParser/Triple.h`
- `cassert`
- `cstdint`
- `PPCGenMCCodeEmitter.inc`

### Important Collaborators / 重要协作组件

- LLVM MC layer / LLVM MC 层
- LLVM Support utilities / LLVM Support 工具层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
