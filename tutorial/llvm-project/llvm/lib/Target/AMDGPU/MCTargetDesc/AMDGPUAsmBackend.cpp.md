# AMDGPUAsmBackend.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/MCTargetDesc/AMDGPUAsmBackend.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUAsmBackend for the LLVM MC target description layer. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM MC 目标描述层中的 AMDGPUAsmBackend 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: File banner, includes, and setup
```cpp
//===-- AMDGPUAsmBackend.cpp - AMDGPU Assembler Backend -------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
/// \file
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/AMDGPUFixupKinds.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "Utils/AMDGPUBaseInfo.h"
#include "llvm/ADT/StringSwitch.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/MC/MCAsmBackend.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCAssembler.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCValue.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/EndianStream.h"
#include "llvm/TargetParser/TargetParser.h"

using namespace llvm;
using namespace llvm::AMDGPU;

namespace {

class AMDGPUAsmBackend : public MCAsmBackend {
public:
  AMDGPUAsmBackend(const Target &T) : MCAsmBackend(llvm::endianness::little) {}

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `AMDGPUAsmBackend`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`AMDGPUAsmBackend`。

### Lines 35-65: Defines applyFixup
```cpp
  void applyFixup(const MCFragment &, const MCFixup &, const MCValue &Target,
                  uint8_t *Data, uint64_t Value, bool IsResolved) override;
  bool fixupNeedsRelaxationAdvanced(const MCFragment &, const MCFixup &,
                                    const MCValue &, uint64_t,
                                    bool) const override;

  void relaxInstruction(MCInst &Inst,
                        const MCSubtargetInfo &STI) const override;

  bool mayNeedRelaxation(unsigned Opcode, ArrayRef<MCOperand> Operands,
                         const MCSubtargetInfo &STI) const override;

  unsigned getMinimumNopSize() const override;
  bool writeNopData(raw_ostream &OS, uint64_t Count,
                    const MCSubtargetInfo *STI) const override;

  std::optional<MCFixupKind> getFixupKind(StringRef Name) const override;
  MCFixupKindInfo getFixupKindInfo(MCFixupKind Kind) const override;
};

} //End anonymous namespace

void AMDGPUAsmBackend::relaxInstruction(MCInst &Inst,
                                        const MCSubtargetInfo &STI) const {
  MCInst Res;
  unsigned RelaxedOpcode = AMDGPU::getSOPPWithRelaxation(Inst.getOpcode());
  Res.setOpcode(RelaxedOpcode);
  Res.addOperand(Inst.getOperand(0));
  Inst = std::move(Res);
}

```
**EN:** This section contains concrete logic for applyFixup. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUAsmBackend::relaxInstruction`, `AMDGPU::getSOPPWithRelaxation`, `std::move`.
**CN:** 本节包含与 applyFixup 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUAsmBackend::relaxInstruction`, `AMDGPU::getSOPPWithRelaxation`, `std::move`。

### Lines 66-90: Implements AMDGPUAsmBackend::fixupNeedsRelaxationAdvanced
```cpp
bool AMDGPUAsmBackend::fixupNeedsRelaxationAdvanced(const MCFragment &,
                                                    const MCFixup &Fixup,
                                                    const MCValue &,
                                                    uint64_t Value,
                                                    bool Resolved) const {
  if (!Resolved)
    return true;
  // if the branch target has an offset of x3f this needs to be relaxed to
  // add a s_nop 0 immediately after branch to effectively increment offset
  // for hardware workaround in gfx1010
  return (((int64_t(Value)/4)-1) == 0x3f);
}

bool AMDGPUAsmBackend::mayNeedRelaxation(unsigned Opcode,
                                         ArrayRef<MCOperand> Operands,
                                         const MCSubtargetInfo &STI) const {
  if (!STI.hasFeature(AMDGPU::FeatureOffset3fBug))
    return false;

  if (AMDGPU::getSOPPWithRelaxation(Opcode) >= 0)
    return true;

  return false;
}

```
**EN:** This section contains concrete logic for AMDGPUAsmBackend::fixupNeedsRelaxationAdvanced. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUAsmBackend::fixupNeedsRelaxationAdvanced`, `AMDGPUAsmBackend::mayNeedRelaxation`, `AMDGPU::getSOPPWithRelaxation`.
**CN:** 本节包含与 AMDGPUAsmBackend::fixupNeedsRelaxationAdvanced 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUAsmBackend::fixupNeedsRelaxationAdvanced`, `AMDGPUAsmBackend::mayNeedRelaxation`, `AMDGPU::getSOPPWithRelaxation`。

### Lines 91-122: Defines getFixupKindNumBytes
```cpp
static unsigned getFixupKindNumBytes(unsigned Kind) {
  switch (Kind) {
  case AMDGPU::fixup_si_sopp_br:
    return 2;
  case FK_SecRel_1:
  case FK_Data_1:
    return 1;
  case FK_SecRel_2:
  case FK_Data_2:
    return 2;
  case FK_SecRel_4:
  case FK_Data_4:
    return 4;
  case FK_SecRel_8:
  case FK_Data_8:
    return 8;
  default:
    llvm_unreachable("Unknown fixup kind!");
  }
}

static uint64_t adjustFixupValue(const MCFixup &Fixup, uint64_t Value,
                                 MCContext *Ctx) {
  int64_t SignedValue = static_cast<int64_t>(Value);

  switch (Fixup.getKind()) {
  case AMDGPU::fixup_si_sopp_br: {
    int64_t BrImm = (SignedValue - 4) / 4;

    if (Ctx && !isInt<16>(BrImm))
      Ctx->reportError(Fixup.getLoc(), "branch size exceeds simm16");

```
**EN:** This section contains concrete logic for getFixupKindNumBytes. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getFixupKindNumBytes 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 123-153: Conditional logic and checks
```cpp
    return BrImm;
  }
  case FK_Data_1:
  case FK_Data_2:
  case FK_Data_4:
  case FK_Data_8:
  case FK_SecRel_4:
    return Value;
  default:
    llvm_unreachable("unhandled fixup kind");
  }
}

void AMDGPUAsmBackend::applyFixup(const MCFragment &F, const MCFixup &Fixup,
                                  const MCValue &Target, uint8_t *Data,
                                  uint64_t Value, bool IsResolved) {
  if (Target.getSpecifier())
    IsResolved = false;
  maybeAddReloc(F, Fixup, Target, Value, IsResolved);
  if (mc::isRelocation(Fixup.getKind()))
    return;

  Value = adjustFixupValue(Fixup, Value, &getContext());
  if (!Value)
    return; // Doesn't change encoding.

  MCFixupKindInfo Info = getFixupKindInfo(Fixup.getKind());

  // Shift the value into position.
  Value <<= Info.TargetOffset;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPUAsmBackend::applyFixup`, `mc::isRelocation`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPUAsmBackend::applyFixup`, `mc::isRelocation`。

### Lines 154-187: Header dependencies and setup
```cpp
  unsigned NumBytes = getFixupKindNumBytes(Fixup.getKind());
  assert(Fixup.getOffset() + NumBytes <= F.getSize() &&
         "Invalid fixup offset!");

  // For each byte of the fragment that the fixup touches, mask in the bits from
  // the fixup value.
  for (unsigned i = 0; i != NumBytes; ++i)
    Data[i] |= static_cast<uint8_t>((Value >> (i * 8)) & 0xff);
}

std::optional<MCFixupKind>
AMDGPUAsmBackend::getFixupKind(StringRef Name) const {
  auto Type = StringSwitch<unsigned>(Name)
#define ELF_RELOC(Name, Value) .Case(#Name, Value)
#include "llvm/BinaryFormat/ELFRelocs/AMDGPU.def"
#undef ELF_RELOC
                  .Case("BFD_RELOC_NONE", ELF::R_AMDGPU_NONE)
                  .Case("BFD_RELOC_32", ELF::R_AMDGPU_ABS32)
                  .Case("BFD_RELOC_64", ELF::R_AMDGPU_ABS64)
                  .Default(-1u);
  if (Type != -1u)
    return static_cast<MCFixupKind>(FirstLiteralRelocationKind + Type);
  return std::nullopt;
}

MCFixupKindInfo AMDGPUAsmBackend::getFixupKindInfo(MCFixupKind Kind) const {
  const static MCFixupKindInfo Infos[AMDGPU::NumTargetFixupKinds] = {
      // name                   offset bits  flags
      {"fixup_si_sopp_br", 0, 16, 0},
  };

  if (mc::isRelocation(Kind))
    return {};

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `AMDGPUAsmBackend::getFixupKind`, `AMDGPUAsmBackend::getFixupKindInfo`, `mc::isRelocation`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`AMDGPUAsmBackend::getFixupKind`, `AMDGPUAsmBackend::getFixupKindInfo`, `mc::isRelocation`。

### Lines 188-219: Defines assert
```cpp
  if (Kind < FirstTargetFixupKind)
    return MCAsmBackend::getFixupKindInfo(Kind);

  assert(unsigned(Kind - FirstTargetFixupKind) < AMDGPU::NumTargetFixupKinds &&
         "Invalid kind!");
  return Infos[Kind - FirstTargetFixupKind];
}

unsigned AMDGPUAsmBackend::getMinimumNopSize() const {
  return 4;
}

bool AMDGPUAsmBackend::writeNopData(raw_ostream &OS, uint64_t Count,
                                    const MCSubtargetInfo *STI) const {
  // If the count is not aligned to the minimum instruction alignment, we must
  // be writing data into the text section (otherwise we have unaligned
  // instructions, and thus have far bigger problems), so just write zeros
  // instead.
  unsigned MinInstAlignment = getContext().getAsmInfo().getMinInstAlignment();
  OS.write_zeros(Count % MinInstAlignment);

  // We are properly aligned, so write NOPs as requested.
  Count /= MinInstAlignment;

  // FIXME: R600 support.
  // s_nop 0
  const uint32_t Encoded_S_NOP_0 = 0xbf800000;

  assert(MinInstAlignment == sizeof(Encoded_S_NOP_0));
  for (uint64_t I = 0; I != Count; ++I)
    support::endian::write<uint32_t>(OS, Encoded_S_NOP_0, Endian);

```
**EN:** This section contains concrete logic for assert. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MCAsmBackend::getFixupKindInfo`, `AMDGPUAsmBackend::getMinimumNopSize`, `AMDGPUAsmBackend::writeNopData`.
**CN:** 本节包含与 assert 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MCAsmBackend::getFixupKindInfo`, `AMDGPUAsmBackend::getMinimumNopSize`, `AMDGPUAsmBackend::writeNopData`。

### Lines 220-252: Declares class ELFAMDGPUAsmBackend
```cpp
  return true;
}

//===----------------------------------------------------------------------===//
// ELFAMDGPUAsmBackend class
//===----------------------------------------------------------------------===//

namespace {

class ELFAMDGPUAsmBackend : public AMDGPUAsmBackend {
  bool Is64Bit;
  bool HasRelocationAddend;
  uint8_t OSABI = ELF::ELFOSABI_NONE;

public:
  ELFAMDGPUAsmBackend(const Target &T, const Triple &TT)
      : AMDGPUAsmBackend(T), Is64Bit(TT.isAMDGCN()),
        HasRelocationAddend(TT.getOS() == Triple::AMDHSA) {
    switch (TT.getOS()) {
    case Triple::AMDHSA:
      OSABI = ELF::ELFOSABI_AMDGPU_HSA;
      break;
    case Triple::AMDPAL:
      OSABI = ELF::ELFOSABI_AMDGPU_PAL;
      break;
    case Triple::Mesa3D:
      OSABI = ELF::ELFOSABI_AMDGPU_MESA3D;
      break;
    default:
      break;
    }
  }

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `ELFAMDGPUAsmBackend`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`ELFAMDGPUAsmBackend`。

### Lines 253-266: Defines createObjectTargetWriter
```cpp
  std::unique_ptr<MCObjectTargetWriter>
  createObjectTargetWriter() const override {
    return createAMDGPUELFObjectWriter(Is64Bit, OSABI, HasRelocationAddend);
  }
};

} // end anonymous namespace

MCAsmBackend *llvm::createAMDGPUAsmBackend(const Target &T,
                                           const MCSubtargetInfo &STI,
                                           const MCRegisterInfo &MRI,
                                           const MCTargetOptions &Options) {
  return new ELFAMDGPUAsmBackend(T, STI.getTargetTriple());
}
```
**EN:** This section contains concrete logic for createObjectTargetWriter. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `llvm::createAMDGPUAsmBackend`.
**CN:** 本节包含与 createObjectTargetWriter 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`llvm::createAMDGPUAsmBackend`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPUAsmBackend`, `ELFAMDGPUAsmBackend`, `AMDGPUAsmBackend::relaxInstruction`, `AMDGPU::getSOPPWithRelaxation`, `std::move`, `AMDGPUAsmBackend::fixupNeedsRelaxationAdvanced`
- **Main themes / 核心主题**: instruction semantics / 指令语义; subtarget modeling / 子目标建模; assembly handling / 汇编处理; feature description / 特性描述
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"MCTargetDesc/AMDGPUFixupKinds.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"Utils/AMDGPUBaseInfo.h"`
- `"llvm/ADT/StringSwitch.h"`
- `"llvm/BinaryFormat/ELF.h"`
- `"llvm/MC/MCAsmBackend.h"`
- `"llvm/MC/MCAsmInfo.h"`
- `"llvm/MC/MCAssembler.h"`
- `"llvm/MC/MCContext.h"`
- `"llvm/MC/MCObjectWriter.h"`
- `"llvm/MC/MCSubtargetInfo.h"`
- `"llvm/MC/MCValue.h"`
- `"llvm/MC/TargetRegistry.h"`
- `"llvm/Support/EndianStream.h"`
- `"llvm/TargetParser/TargetParser.h"`
- `"llvm/BinaryFormat/ELFRelocs/AMDGPU.def"`
