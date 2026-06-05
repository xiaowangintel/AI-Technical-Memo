# RISCVMachObjectWriter.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/MCTargetDesc/RISCVMachObjectWriter.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements shared pass declarations and backend-wide utilities for the RISC-V target. / 实现RISC-V 目标的共享 pass 声明与后端通用工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: File prologue and imported dependencies / 文件前言与导入依赖
```cpp
//===-- RISCVMachObjectWriter.cpp - RISC-V Mach Object Writer -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/RISCVFixupKinds.h"
#include "MCTargetDesc/RISCVMCTargetDesc.h"
#include "RISCVMCAsmInfo.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/ADT/Twine.h"
#include "llvm/BinaryFormat/MachO.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCAsmInfoDarwin.h"
#include "llvm/MC/MCAssembler.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCFixup.h"
#include "llvm/MC/MCMachObjectWriter.h"
#include "llvm/MC/MCSection.h"
#include "llvm/MC/MCSectionMachO.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/MCValue.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/MathExtras.h"
#include <cassert>
```
**EN:** This opening block combines banner comments with the first wave of includes, giving readers both context and the main compile-time dependencies.

**CN:** 该开头区段同时包含说明性注释与首批 include，让读者先看到文件定位，再看到主要编译期依赖。

### Lines 29-43: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp
#include <cstdint>

using namespace llvm;

namespace {

class RISCVMachObjectWriter : public MCMachObjectTargetWriter {
  bool getRISCVFixupKindMachOInfo(const MCFixup &Fixup, unsigned &RelocType,
                                  const MCValue Sym, unsigned &Log2Size,
                                  const MCAssembler &Asm);

public:
  RISCVMachObjectWriter(uint32_t CPUType, uint32_t CPUSubtype)
      : MCMachObjectTargetWriter(false, CPUType, CPUSubtype) {}
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 44-58: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
  void recordRelocation(MachObjectWriter *Writer, MCAssembler &Asm,
                        const MCFragment *Fragment, const MCFixup &Fixup,
                        MCValue Target, uint64_t &FixedValue) override;
};

} // end anonymous namespace

bool RISCVMachObjectWriter::getRISCVFixupKindMachOInfo(const MCFixup &Fixup,
                                                       unsigned &RelocType,
                                                       const MCValue Sym,
                                                       unsigned &Log2Size,
                                                       const MCAssembler &Asm) {
  RelocType = unsigned(MachO::RISCV_RELOC_UNSIGNED);
  Log2Size = ~0U;
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 59-86: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (Sym.getSpecifier() == RISCV::S_GOT_HI) {
    Log2Size = Log2_32(4);
    RelocType = unsigned(MachO::RISCV_RELOC_GOT_HI20);
    return true;
  }

  switch (Fixup.getKind()) {
  default:
    return false;

  case FK_Data_1:
    Log2Size = Log2_32(1);
    return true;
  case FK_Data_2:
    Log2Size = Log2_32(2);
    return true;
  case FK_Data_4:
    Log2Size = Log2_32(4);
    return true;
  case FK_Data_8:
    Log2Size = Log2_32(8);
    return true;
  case RISCV::fixup_riscv_pcrel_lo12_i:
  case RISCV::fixup_riscv_pcrel_lo12_s:
    llvm_unreachable("lo12 fixups should have been resolved elsewhere");
  case RISCV::fixup_riscv_lo12_i:
  case RISCV::fixup_riscv_lo12_s:
    Log2Size = Log2_32(4);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 87-108: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    RelocType = MachO::RISCV_RELOC_LO12;
    return true;
  case RISCV::fixup_riscv_pcrel_hi20:
    Log2Size = Log2_32(4);
    if (Sym.getSpecifier() != RISCV::S_PCREL_HI) {
      Asm.getContext().reportError(Fixup.getLoc(),
                                   "unknown AUIPC relocation kind");
      return false;
    }
    RelocType = unsigned(MachO::RISCV_RELOC_HI20);
    return true;
  case RISCV::fixup_riscv_hi20:
    Log2Size = Log2_32(4);
    RelocType = unsigned(MachO::RISCV_RELOC_HI20);
    return true;
  case RISCV::fixup_riscv_call:
  case RISCV::fixup_riscv_call_plt:
  case RISCV::fixup_riscv_jal:
    Log2Size = Log2_32(4);
    RelocType = unsigned(MachO::RISCV_RELOC_BRANCH21);
    return true;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 109-128: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
}

static bool canUseLocalRelocation(const MCSectionMachO &Section,
                                  const MCSymbol &Symbol, unsigned Log2Size) {
  // Debug info sections can use local relocations.
  if (Section.hasAttribute(MachO::S_ATTR_DEBUG))
    return true;

  // Otherwise, only pointer sized relocations are supported.
  if (Log2Size != 2)
    return false;

  // But only if they don't point to a few forbidden sections.
  if (!Symbol.isInSection())
    return true;
  const MCSectionMachO &RefSec =
      static_cast<const MCSectionMachO &>(Symbol.getSection());
  if (RefSec.getType() == MachO::S_CSTRING_LITERALS)
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 129-149: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (RefSec.getSegmentName() == "__DATA" &&
      RefSec.getName() == "__objc_classrefs")
    return false;

  return true;
}

static void emitRelocation(MachObjectWriter *Writer, const MCFragment *Fragment,
                           uint32_t FixupOffset, const MCSymbol *RelSymbol,
                           unsigned Index, bool IsPCRel, unsigned Log2Size,
                           unsigned Type) {

  assert(isUInt<2>(Log2Size) && "Invalid Log2Size");
  assert(isUInt<4>(Type) && "Invalid type");
  assert(isUInt<24>(Index) && "Invalid Index");
  MachO::any_relocation_info MRE;
  MRE.r_word0 = FixupOffset;
  MRE.r_word1 =
      (Index << 0) | (IsPCRel << 24) | (Log2Size << 25) | (Type << 28);
  Writer->addRelocation(RelSymbol, Fragment->getParent(), MRE);
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 150-170: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

static bool checkSymbolBase(const MCSymbol *Base, const MCSymbol *Symbol,
                            const MCFixup &Fixup, MCAssembler &Asm) {
  if (!Base) {
    Asm.getContext().reportError(
        Fixup.getLoc(),
        "unsupported relocation of local symbol '" + Symbol->getName() +
            "'. Must have non-local symbol earlier in section.");
    return false;
  }
  return true;
}

template <unsigned Bits>
static bool isValidInt(const uint64_t &FixedValue, const char *Msg,
                       MCAssembler &Asm, const SMLoc Loc) {
  const bool IsValid = isInt<Bits>(FixedValue);
  if (!IsValid)
    Asm.getContext().reportError(Loc, Msg);
  return IsValid;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 171-191: Function implementation: RISCVMachObjectWriter::recordRelocation / 函数实现：RISCVMachObjectWriter::recordRelocation
```cpp

extern const MCFixup *getPCRelHiFixup(const MCSpecifierExpr &Expr,
                                      const MCFragment **DFOut);

void RISCVMachObjectWriter::recordRelocation(
    MachObjectWriter *Writer, MCAssembler &Asm, const MCFragment *Fragment,
    const MCFixup &Fixup, MCValue Target, uint64_t &FixedValue) {
  const bool IsPCRel =
      Fixup.isPCRel() || Target.getSpecifier() == RISCV::S_GOT_HI;

  // See <reloc.h>.
  uint32_t FixupOffset = Asm.getFragmentOffset(*Fragment);
  unsigned Log2Size = 0;
  int64_t Value = 0;
  unsigned Index = 0;
  unsigned Type = 0;
  const unsigned Kind = Fixup.getKind();
  const MCSymbol *RelSymbol = nullptr;
  bool RequireExtraAddend = false;
  uint64_t ExtraAddendValue = 0;
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 192-213: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  FixupOffset += Fixup.getOffset();

  // RISC-V pcrel relocation addends do not include the section offset.
  if (IsPCRel)
    FixedValue += FixupOffset;

  // AUIPC fixups use relocations for the whole symbol value and only
  // put the addend in the instruction itself. Clear out any value the
  // generic code figured out from the symbol definition.
  if (Kind == RISCV::fixup_riscv_pcrel_hi20)
    FixedValue = 0;

  // %pcrel_lo relocations directly target the same symbol as the
  // corresponding AUIPC, and encode (inline) an offset to that AUIPC
  // in the immediate field of the instruction itself.
  if (Kind == RISCV::fixup_riscv_pcrel_lo12_i ||
      Kind == RISCV::fixup_riscv_pcrel_lo12_s) {
    const MCFragment *AUIPCDF;
    const MCFixup *AUIPCFixup =
        getPCRelHiFixup(cast<MCSpecifierExpr>(*Fixup.getValue()), &AUIPCDF);
    assert(AUIPCFixup);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 214-227: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    // Calculate the offset from this fixup to the AUIPC it references, this
    // will be put into the instruction itself.
    FixedValue =
        Asm.getFragmentOffset(*AUIPCDF) + AUIPCFixup->getOffset() - FixupOffset;
    if (!isInt<12>(FixedValue)) {
      RequireExtraAddend = true;
      ExtraAddendValue = FixedValue;
      if (!isValidInt<24>(
              ExtraAddendValue,
              "AUIPC out of range of corresponding %pcrel_lo instruction", Asm,
              Fixup.getLoc()))
        return;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 228-243: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    // Retarget the rest of this function to reference the AUIPC's symbol.
    MCValue RealTarget;
    if (!AUIPCFixup->getValue()->evaluateAsValue(RealTarget, Asm)) {
      Asm.getContext().reportError(AUIPCFixup->getLoc(),
                                   "cannot understand AUIPC target");
      return;
    }
    if (RealTarget.getSubSym()) {
      Asm.getContext().reportError(AUIPCFixup->getLoc(),
                                   "AUIPC target with symbol difference");
      return;
    }

    Target = MCValue::get(RealTarget.getAddSym(), /*SymB*/ nullptr,
                          RealTarget.getConstant(), RISCV::S_PCREL_LO);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 244-264: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    Log2Size = Log2_32(4);
    const auto Spec = RealTarget.getSpecifier();
    Type = Spec == RISCV::S_GOT_HI ? MachO::RISCV_RELOC_GOT_LO12
                                   : MachO::RISCV_RELOC_LO12;
  } else if (!getRISCVFixupKindMachOInfo(Fixup, Type, Target, Log2Size, Asm)) {
    Asm.getContext().reportError(Fixup.getLoc(), "unknown RISC-V fixup kind");
    return;
  }

  // imm19 relocations are for conditional branches, which require
  // assembler local symbols. If we got here, that's not what we have,
  // so report an error.
  if (Kind == RISCV::fixup_riscv_branch ||
      Kind == RISCV::fixup_riscv_rvc_jump ||
      Kind == RISCV::fixup_riscv_rvc_branch) {
    Asm.getContext().reportError(
        Fixup.getLoc(), "conditional branch requires assembler-local"
                        " label. '" +
                            Target.getAddSym()->getName() + "' is external.");
    return;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 265-286: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  Value = Target.getConstant();

  // Only .word and %pcrel_lo instructions inline the pc-relative
  // offset in the instruction, but only if such offset fits the
  // 12-bit immediate of an addi or an lw instrucion.
  if ((Type != MachO::RISCV_RELOC_UNSIGNED && Type != MachO::RISCV_RELOC_LO12 &&
       Type != MachO::RISCV_RELOC_GOT_LO12) ||
      RequireExtraAddend)
    FixedValue = 0;

  // Emit relocations.

  // Constants.
  if (Target.isAbsolute()) {
    // FIXME: Should this always be extern?
    // SymbolNum of 0 indicates the absolute section.
    if (IsPCRel) {
      Asm.getContext().reportError(Fixup.getLoc(),
                                   "PC relative absolute relocation!");
      return;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 287-305: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    emitRelocation(Writer, Fragment, FixupOffset, RelSymbol, Index,
                   /*IsPCRel*/ false, /*Log2Size*/ ~0U,
                   /*Type*/ MachO::RISCV_RELOC_UNSIGNED);
    return;
  }

  // A - B + constant
  if (const MCSymbol *B = Target.getSubSym()) {
    const MCSymbol *A = Target.getAddSym();
    const MCSymbol *A_Base = Writer->getAtom(*A);
    const MCSymbol *B_Base = Writer->getAtom(*B);

    // We don't support PCrel relocations of differences.
    if (IsPCRel) {
      Asm.getContext().reportError(Fixup.getLoc(),
                                   "unsupported pc-relative relocation of "
                                   "difference");
      return;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 306-326: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

    // Ensure both symbols have base atoms for external relocations.
    if (!checkSymbolBase(A_Base, A, Fixup, Asm) ||
        !checkSymbolBase(B_Base, B, Fixup, Asm))
      return;

    if (A_Base && A_Base == B_Base) {
      Asm.getContext().reportError(
          Fixup.getLoc(), "unsupported relocation with identical base");
      return;
    }

    Value +=
        (!A->getFragment() ? 0 : Writer->getSymbolAddress(*A)) -
        (!A_Base || !A_Base->getFragment() ? 0
                                           : Writer->getSymbolAddress(*A_Base));
    Value -=
        (!B->getFragment() ? 0 : Writer->getSymbolAddress(*B)) -
        (!B_Base || !B_Base->getFragment() ? 0
                                           : Writer->getSymbolAddress(*B_Base));
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 327-342: Type declaration for relocation_info / relocation_info 的类型声明
```cpp
    // If there's any addend left to handle, inline it in the instruction's
    // immediate.
    FixedValue = Value;
    if (!isValidInt<12>(
            FixedValue,
            "AUIPC out of range of corresponding %pcrel_lo instruction", Asm,
            Fixup.getLoc()))
      return;

    emitRelocation(Writer, Fragment, FixupOffset, /*RelSymbol*/ A_Base, Index,
                   IsPCRel, Log2Size, /*Type*/ MachO::RISCV_RELOC_UNSIGNED);
    // struct relocation_info (8 bytes)
    emitRelocation(Writer, Fragment, FixupOffset, /*RelSymbol*/ B_Base, Index,
                   IsPCRel, Log2Size, /*Type*/ MachO::RISCV_RELOC_SUBTRACTOR);
    return;
  }
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 343-356: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  // A + constant
  if (const MCSymbol *Symbol = Target.getAddSym()) {
    assert(!Target.getSubSym() && "invalid expression");
    const MCSectionMachO &Section =
        static_cast<const MCSectionMachO &>(*Fragment->getParent());

    const bool CanUseLocalRelocation =
        canUseLocalRelocation(Section, *Symbol, Log2Size);
    if (Symbol->isTemporary() && (Value || !CanUseLocalRelocation)) {
      if (!Symbol->isInSection()) {
        checkSymbolBase(nullptr, Symbol, Fixup, Asm);
        return;
      }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 357-374: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
      const MCSection &Sec = Symbol->getSection();
      if (!MCAsmInfoDarwin::isSectionAtomizableBySymbols(Sec))
        Symbol->setUsedInReloc();
    }

    const MCSymbol *Base = Writer->getAtom(*Symbol);
    // If the symbol is a variable it can either be in a section and
    // we have a base or it is absolute and should have been expanded.
    assert(!Symbol->isVariable() || Base);

    // Relocations inside debug sections always use local relocations when
    // possible. This seems to be done because the debugger doesn't fully
    // understand relocation entries and expects to find values that
    // have already been fixed up.
    if (Symbol->isInSection()) {
      if (Section.hasAttribute(MachO::S_ATTR_DEBUG))
        Base = nullptr;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 375-389: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

    // RISC-V uses external relocations as much as possible. For debug
    // sections, and for pointer-sized relocations (.quad), we allow section
    // relocations.  It's code sections that run into trouble.
    if (Base) {
      RelSymbol = Base;

      // Add the local offset, if needed.
      if (Base != Symbol)
        Value += Asm.getSymbolOffset(*Symbol) - Asm.getSymbolOffset(*Base);
    } else if (Symbol->isInSection()) {
      if (!CanUseLocalRelocation) {
        checkSymbolBase(nullptr, Symbol, Fixup, Asm);
        return;
      }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 390-410: Type declaration for relocation_info / relocation_info 的类型声明
```cpp
      // Adjust the relocation to be section-relative.
      // The index is the section ordinal (1-based).
      const MCSection &Sec = Symbol->getSection();
      Index = Sec.getOrdinal() + 1;
      Value += Writer->getSymbolAddress(*Symbol);

      if (IsPCRel)
        Value -= Writer->getFragmentAddress(Asm, Fragment) + Fixup.getOffset();
    } else {
      llvm_unreachable(
          "This constant variable should have been expanded during evaluation");
    }
    if (Type == MachO::RISCV_RELOC_UNSIGNED) {
      // If there's any addend left to handle, encode it in the instruction.
      FixedValue = Value;
      // struct relocation_info (8 bytes)
      emitRelocation(Writer, Fragment, FixupOffset, RelSymbol, Index,
                     /*IsPCRel*/ false, Log2Size,
                     /*Type*/ MachO::RISCV_RELOC_UNSIGNED);
      return;
    }
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 411-425: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    // We have an addend offset that is encoded in the relocation
    // record, not inlined in the instruction.
    if (Value) {
      if (!isValidInt<24>(Value, "addend too big for relocation", Asm,
                          Fixup.getLoc()))
        return;

      emitRelocation(Writer, Fragment, FixupOffset, RelSymbol, Index, IsPCRel,
                     Log2Size, Type);
      // Now set up the Addend relocation.
      emitRelocation(Writer, Fragment, FixupOffset, /*RelSymbol*/ nullptr,
                     Value & 0xffffff, /*IsPCRel*/ false, /*Log2Size*/ 2,
                     /*Type*/ MachO::RISCV_RELOC_ADDEND);
      return;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 426-441: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

    emitRelocation(Writer, Fragment, FixupOffset, RelSymbol, Index, IsPCRel,
                   Log2Size, Type);
  }

  if (RequireExtraAddend) {
    emitRelocation(Writer, Fragment, FixupOffset, /*RelSymbol*/ nullptr,
                   ExtraAddendValue & 0xffffff, /*IsPCRel*/ false,
                   /*Log2Size*/ 2, /*Type*/ MachO::RISCV_RELOC_ADDEND);
  }
}

std::unique_ptr<MCObjectTargetWriter>
llvm::createRISCVMachObjectWriter(uint32_t CPUType, uint32_t CPUSubtype) {
  return std::make_unique<RISCVMachObjectWriter>(CPUType, CPUSubtype);
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

## Key Concepts / 关键概念
- **MC layer target description** / **MC 层目标描述**

## Dependencies / 依赖关系
- `MCTargetDesc/RISCVFixupKinds.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `MCTargetDesc/RISCVMCTargetDesc.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVMCAsmInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/ADT/StringExtras.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/ADT/Twine.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/BinaryFormat/MachO.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCAsmInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCAsmInfoDarwin.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCAssembler.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCContext.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCExpr.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCFixup.h` — Directly referenced by this file. / 该文件直接引用的依赖。
