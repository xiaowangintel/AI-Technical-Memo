# RISCVELFObjectWriter.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/MCTargetDesc/RISCVELFObjectWriter.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements RISC-V ELF object writer behavior. / 实现RISC-V ELF 目标文件写出行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: File prologue and imported dependencies / 文件前言与导入依赖
```cpp
//===-- RISCVELFObjectWriter.cpp - RISC-V ELF Writer ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/RISCVFixupKinds.h"
#include "MCTargetDesc/RISCVMCAsmInfo.h"
#include "MCTargetDesc/RISCVMCTargetDesc.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCELFObjectWriter.h"
#include "llvm/MC/MCFixup.h"
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/MC/MCValue.h"
#include "llvm/Support/ErrorHandling.h"
```
**EN:** This opening block combines banner comments with the first wave of includes, giving readers both context and the main compile-time dependencies.

**CN:** 该开头区段同时包含说明性注释与首批 include，让读者先看到文件定位，再看到主要编译期依赖。

### Lines 19-34: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
using namespace llvm;

namespace {
class RISCVELFObjectWriter : public MCELFObjectTargetWriter {
public:
  RISCVELFObjectWriter(uint8_t OSABI, bool Is64Bit);

  ~RISCVELFObjectWriter() override;

  // Return true if the given relocation must be with a symbol rather than
  // section plus offset.
  bool needsRelocateWithSymbol(const MCValue &, unsigned Type) const override {
    // TODO: this is very conservative, update once RISC-V psABI requirements
    //       are clarified.
    return true;
  }
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 35-45: Function implementation: RISCVELFObjectWriter::RISCVELFObjectWriter / 函数实现：RISCVELFObjectWriter::RISCVELFObjectWriter
```cpp

protected:
  unsigned getRelocType(const MCFixup &, const MCValue &,
                        bool IsPCRel) const override;
};
}

RISCVELFObjectWriter::RISCVELFObjectWriter(uint8_t OSABI, bool Is64Bit)
    : MCELFObjectTargetWriter(Is64Bit, OSABI, ELF::EM_RISCV,
                              /*HasRelocationAddend*/ true) {}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 46-65: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
RISCVELFObjectWriter::~RISCVELFObjectWriter() = default;

unsigned RISCVELFObjectWriter::getRelocType(const MCFixup &Fixup,
                                            const MCValue &Target,
                                            bool IsPCRel) const {
  auto Kind = Fixup.getKind();
  auto Spec = Target.getSpecifier();
  switch (Spec) {
  case ELF::R_RISCV_TPREL_HI20:
  case ELF::R_RISCV_TLS_GOT_HI20:
  case ELF::R_RISCV_TLS_GD_HI20:
  case ELF::R_RISCV_TLSDESC_HI20:
    if (auto *SA = const_cast<MCSymbol *>(Target.getAddSym()))
      static_cast<MCSymbolELF *>(SA)->setType(ELF::STT_TLS);
    break;
  case ELF::R_RISCV_PLT32:
  case ELF::R_RISCV_GOT32_PCREL:
    if (Kind == FK_Data_4)
      break;
    reportError(Fixup.getLoc(), "%" + RISCV::getSpecifierName(Spec) +
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 66-76: Definitions and supporting logic / 定义与支撑逻辑
```cpp
                                    " can only be used in a .word directive");
    return ELF::R_RISCV_NONE;
  default:
    break;
  }

  // Extract the relocation type from the fixup kind, after applying STT_TLS as
  // needed.
  if (mc::isRelocation(Fixup.getKind()))
    return Kind;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 77-96: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (IsPCRel) {
    switch (Kind) {
    default:
      reportError(Fixup.getLoc(), "unsupported relocation type");
      return ELF::R_RISCV_NONE;
    case FK_Data_4:
      return ELF::R_RISCV_32_PCREL;
    case RISCV::fixup_riscv_pcrel_hi20:
      return ELF::R_RISCV_PCREL_HI20;
    case RISCV::fixup_riscv_pcrel_lo12_i:
      return ELF::R_RISCV_PCREL_LO12_I;
    case RISCV::fixup_riscv_pcrel_lo12_s:
      return ELF::R_RISCV_PCREL_LO12_S;
    case RISCV::fixup_riscv_jal:
      return ELF::R_RISCV_JAL;
    case RISCV::fixup_riscv_branch:
      return ELF::R_RISCV_BRANCH;
    case RISCV::fixup_riscv_rvc_jump:
      return ELF::R_RISCV_RVC_JUMP;
    case RISCV::fixup_riscv_rvc_branch:
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 97-108: Definitions and supporting logic / 定义与支撑逻辑
```cpp
      return ELF::R_RISCV_RVC_BRANCH;
    case RISCV::fixup_riscv_call:
      return ELF::R_RISCV_CALL_PLT;
    case RISCV::fixup_riscv_call_plt:
      return ELF::R_RISCV_CALL_PLT;
    case RISCV::fixup_riscv_qc_e_branch:
      return ELF::R_RISCV_QC_E_BRANCH;
    case RISCV::fixup_riscv_qc_e_call_plt:
      return ELF::R_RISCV_QC_E_CALL_PLT;
    case RISCV::fixup_riscv_nds_branch_10:
      return ELF::R_RISCV_NDS_BRANCH_10;
    }
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 109-128: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  }

  switch (Kind) {
  default:
    reportError(Fixup.getLoc(), "unsupported relocation type");
    return ELF::R_RISCV_NONE;

  case FK_Data_1:
    reportError(Fixup.getLoc(), "1-byte data relocations not supported");
    return ELF::R_RISCV_NONE;
  case FK_Data_2:
    reportError(Fixup.getLoc(), "2-byte data relocations not supported");
    return ELF::R_RISCV_NONE;
  case FK_Data_4:
    switch (Spec) {
    case ELF::R_RISCV_32_PCREL:
    case ELF::R_RISCV_GOT32_PCREL:
    case ELF::R_RISCV_PLT32:
      return Spec;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 129-145: Definitions and supporting logic / 定义与支撑逻辑
```cpp
    return ELF::R_RISCV_32;
  case FK_Data_8:
    return ELF::R_RISCV_64;
  case RISCV::fixup_riscv_hi20:
    return ELF::R_RISCV_HI20;
  case RISCV::fixup_riscv_lo12_i:
    return ELF::R_RISCV_LO12_I;
  case RISCV::fixup_riscv_lo12_s:
    return ELF::R_RISCV_LO12_S;
  case RISCV::fixup_riscv_rvc_imm:
    reportError(Fixup.getLoc(), "No relocation for CI-type instructions");
    return ELF::R_RISCV_NONE;
  case RISCV::fixup_riscv_qc_e_32:
    return ELF::R_RISCV_QC_E_32;
  case RISCV::fixup_riscv_qc_abs20_u:
    return ELF::R_RISCV_QC_ABS20_U;
  }
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 146-151: Function implementation: llvm::createRISCVELFObjectWriter / 函数实现：llvm::createRISCVELFObjectWriter
```cpp
}

std::unique_ptr<MCObjectTargetWriter>
llvm::createRISCVELFObjectWriter(uint8_t OSABI, bool Is64Bit) {
  return std::make_unique<RISCVELFObjectWriter>(OSABI, Is64Bit);
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

## Key Concepts / 关键概念
- **MC layer target description** / **MC 层目标描述**

## Dependencies / 依赖关系
- `MCTargetDesc/RISCVFixupKinds.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `MCTargetDesc/RISCVMCAsmInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `MCTargetDesc/RISCVMCTargetDesc.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCContext.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCELFObjectWriter.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCFixup.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCObjectWriter.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCValue.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Support/ErrorHandling.h` — Directly referenced by this file. / 该文件直接引用的依赖。
