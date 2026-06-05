# AMDGPUMCCodeEmitter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/MCTargetDesc/AMDGPUMCCodeEmitter.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUMCCodeEmitter for the LLVM MC target description layer. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM MC 目标描述层中的 AMDGPUMCCodeEmitter 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: File banner, includes, and setup
```cpp
//===-- AMDGPUMCCodeEmitter.cpp - AMDGPU Code Emitter ---------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// The AMDGPU code emitter produces machine code that can be executed
/// directly on the GPU device.
//
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/AMDGPUFixupKinds.h"
#include "MCTargetDesc/AMDGPUMCExpr.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "SIDefines.h"
#include "Utils/AMDGPUBaseInfo.h"
#include "llvm/ADT/APInt.h"
#include "llvm/MC/MCCodeEmitter.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/EndianStream.h"
#include <optional>

using namespace llvm;

namespace {

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 35-65: Declares class AMDGPUMCCodeEmitter
```cpp
class AMDGPUMCCodeEmitter : public MCCodeEmitter {
  const MCRegisterInfo &MRI;
  const MCInstrInfo &MCII;

public:
  AMDGPUMCCodeEmitter(const MCInstrInfo &MCII, const MCRegisterInfo &MRI)
      : MRI(MRI), MCII(MCII) {}

  /// Encode the instruction and write it to the OS.
  void encodeInstruction(const MCInst &MI, SmallVectorImpl<char> &CB,
                         SmallVectorImpl<MCFixup> &Fixups,
                         const MCSubtargetInfo &STI) const override;

  void getMachineOpValue(const MCInst &MI, const MCOperand &MO, APInt &Op,
                         SmallVectorImpl<MCFixup> &Fixups,
                         const MCSubtargetInfo &STI) const;

  void getMachineOpValueT16(const MCInst &MI, unsigned OpNo, APInt &Op,
                            SmallVectorImpl<MCFixup> &Fixups,
                            const MCSubtargetInfo &STI) const;

  void getMachineOpValueT16Lo128(const MCInst &MI, unsigned OpNo, APInt &Op,
                                 SmallVectorImpl<MCFixup> &Fixups,
                                 const MCSubtargetInfo &STI) const;

  /// Use a fixup to encode the simm16 field for SOPP branch
  ///        instructions.
  void getSOPPBrEncoding(const MCInst &MI, unsigned OpNo, APInt &Op,
                         SmallVectorImpl<MCFixup> &Fixups,
                         const MCSubtargetInfo &STI) const;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUMCCodeEmitter`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUMCCodeEmitter`。

### Lines 66-98: Defines getSMEMOffsetEncoding
```cpp
  void getSMEMOffsetEncoding(const MCInst &MI, unsigned OpNo, APInt &Op,
                             SmallVectorImpl<MCFixup> &Fixups,
                             const MCSubtargetInfo &STI) const;

  void getSDWASrcEncoding(const MCInst &MI, unsigned OpNo, APInt &Op,
                          SmallVectorImpl<MCFixup> &Fixups,
                          const MCSubtargetInfo &STI) const;

  void getSDWAVopcDstEncoding(const MCInst &MI, unsigned OpNo, APInt &Op,
                              SmallVectorImpl<MCFixup> &Fixups,
                              const MCSubtargetInfo &STI) const;

  void getAVOperandEncoding(const MCInst &MI, unsigned OpNo, APInt &Op,
                            SmallVectorImpl<MCFixup> &Fixups,
                            const MCSubtargetInfo &STI) const;

private:
  uint64_t getImplicitOpSelHiEncoding(int Opcode) const;
  void getMachineOpValueCommon(const MCInst &MI, const MCOperand &MO,
                               unsigned OpNo, APInt &Op,
                               SmallVectorImpl<MCFixup> &Fixups,
                               const MCSubtargetInfo &STI) const;

  /// Encode an fp or int literal.
  std::optional<uint64_t>
  getLitEncoding(const MCInstrDesc &Desc, const MCOperand &MO, unsigned OpNo,
                 const MCSubtargetInfo &STI,
                 bool HasMandatoryLiteral = false) const;

  void getBinaryCodeForInstr(const MCInst &MI, SmallVectorImpl<MCFixup> &Fixups,
                             APInt &Inst, APInt &Scratch,
                             const MCSubtargetInfo &STI) const;

```
**EN:** This section contains concrete logic for getSMEMOffsetEncoding. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getSMEMOffsetEncoding 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 99-131: Defines postEncodeVOP3
```cpp
  template <bool HasSrc0, bool HasSrc1, bool HasSrc2>
  APInt postEncodeVOP3(const MCInst &MI, APInt EncodedValue,
                       const MCSubtargetInfo &STI) const;

  APInt postEncodeVOPCX(const MCInst &MI, APInt EncodedValue,
                        const MCSubtargetInfo &STI) const;
};

} // end anonymous namespace

MCCodeEmitter *llvm::createAMDGPUMCCodeEmitter(const MCInstrInfo &MCII,
                                               MCContext &Ctx) {
  return new AMDGPUMCCodeEmitter(MCII, *Ctx.getRegisterInfo());
}

static void addFixup(SmallVectorImpl<MCFixup> &Fixups, uint32_t Offset,
                     const MCExpr *Value, uint16_t Kind, bool PCRel = false) {
  Fixups.push_back(MCFixup::create(Offset, Value, Kind, PCRel));
}

// Returns the encoding value to use if the given integer is an integer inline
// immediate value, or 0 if it is not.
template <typename IntTy>
static uint32_t getIntInlineImmEncoding(IntTy Imm) {
  if (Imm >= 0 && Imm <= 64)
    return 128 + Imm;

  if (Imm >= -16 && Imm <= -1)
    return 192 + std::abs(Imm);

  return 0;
}

```
**EN:** This section contains concrete logic for postEncodeVOP3. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `llvm::createAMDGPUMCCodeEmitter`, `MCFixup::create`, `std::abs`.
**CN:** 本节包含与 postEncodeVOP3 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`llvm::createAMDGPUMCCodeEmitter`, `MCFixup::create`, `std::abs`。

### Lines 132-164: Defines getLit16Encoding
```cpp
static uint32_t getLit16Encoding(uint16_t Val, const MCSubtargetInfo &STI) {
  uint16_t IntImm = getIntInlineImmEncoding(static_cast<int16_t>(Val));
  if (IntImm != 0)
    return IntImm;

  if (Val == 0x3800) // 0.5
    return 240;

  if (Val == 0xB800) // -0.5
    return 241;

  if (Val == 0x3C00) // 1.0
    return 242;

  if (Val == 0xBC00) // -1.0
    return 243;

  if (Val == 0x4000) // 2.0
    return 244;

  if (Val == 0xC000) // -2.0
    return 245;

  if (Val == 0x4400) // 4.0
    return 246;

  if (Val == 0xC400) // -4.0
    return 247;

  if (Val == 0x3118 && // 1.0 / (2.0 * pi)
      STI.hasFeature(AMDGPU::FeatureInv2PiInlineImm))
    return 248;

```
**EN:** This section contains concrete logic for getLit16Encoding. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getLit16Encoding 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 165-196: Defines getLitBF16Encoding
```cpp
  return 255;
}

static uint32_t getLitBF16Encoding(uint16_t Val) {
  uint16_t IntImm = getIntInlineImmEncoding(static_cast<int16_t>(Val));
  if (IntImm != 0)
    return IntImm;

  // clang-format off
  switch (Val) {
  case 0x3F00: return 240; // 0.5
  case 0xBF00: return 241; // -0.5
  case 0x3F80: return 242; // 1.0
  case 0xBF80: return 243; // -1.0
  case 0x4000: return 244; // 2.0
  case 0xC000: return 245; // -2.0
  case 0x4080: return 246; // 4.0
  case 0xC080: return 247; // -4.0
  case 0x3E22: return 248; // 1.0 / (2.0 * pi)
  default:     return 255;
  }
  // clang-format on
}

static uint32_t getLit32Encoding(uint32_t Val, const MCSubtargetInfo &STI) {
  uint32_t IntImm = getIntInlineImmEncoding(static_cast<int32_t>(Val));
  if (IntImm != 0)
    return IntImm;

  if (Val == llvm::bit_cast<uint32_t>(0.5f))
    return 240;

```
**EN:** This section contains concrete logic for getLitBF16Encoding. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getLitBF16Encoding 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 197-228: Conditional logic and checks
```cpp
  if (Val == llvm::bit_cast<uint32_t>(-0.5f))
    return 241;

  if (Val == llvm::bit_cast<uint32_t>(1.0f))
    return 242;

  if (Val == llvm::bit_cast<uint32_t>(-1.0f))
    return 243;

  if (Val == llvm::bit_cast<uint32_t>(2.0f))
    return 244;

  if (Val == llvm::bit_cast<uint32_t>(-2.0f))
    return 245;

  if (Val == llvm::bit_cast<uint32_t>(4.0f))
    return 246;

  if (Val == llvm::bit_cast<uint32_t>(-4.0f))
    return 247;

  if (Val == 0x3e22f983 && // 1.0 / (2.0 * pi)
      STI.hasFeature(AMDGPU::FeatureInv2PiInlineImm))
    return 248;

  return 255;
}

static uint32_t getLit16IntEncoding(uint32_t Val, const MCSubtargetInfo &STI) {
  return getLit32Encoding(Val, STI);
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 229-262: Defines getLit64Encoding
```cpp
static uint32_t getLit64Encoding(const MCInstrDesc &Desc, uint64_t Val,
                                 const MCSubtargetInfo &STI, bool IsFP) {
  uint32_t IntImm = getIntInlineImmEncoding(static_cast<int64_t>(Val));
  if (IntImm != 0)
    return IntImm;

  if (Val == llvm::bit_cast<uint64_t>(0.5))
    return 240;

  if (Val == llvm::bit_cast<uint64_t>(-0.5))
    return 241;

  if (Val == llvm::bit_cast<uint64_t>(1.0))
    return 242;

  if (Val == llvm::bit_cast<uint64_t>(-1.0))
    return 243;

  if (Val == llvm::bit_cast<uint64_t>(2.0))
    return 244;

  if (Val == llvm::bit_cast<uint64_t>(-2.0))
    return 245;

  if (Val == llvm::bit_cast<uint64_t>(4.0))
    return 246;

  if (Val == llvm::bit_cast<uint64_t>(-4.0))
    return 247;

  if (Val == 0x3fc45f306dc9c882 && // 1.0 / (2.0 * pi)
      STI.hasFeature(AMDGPU::FeatureInv2PiInlineImm))
    return 248;

```
**EN:** This section contains concrete logic for getLit64Encoding. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getLit64Encoding 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 263-296: Conditional logic and checks
```cpp
  // The rest part needs to align with AMDGPUInstPrinter::printLiteral64.

  bool CanUse64BitLiterals =
      STI.hasFeature(AMDGPU::Feature64BitLiterals) &&
      !(Desc.TSFlags & (SIInstrFlags::VOP3 | SIInstrFlags::VOP3P));
  if (IsFP) {
    return CanUse64BitLiterals && Lo_32(Val) ? 254 : 255;
  }

  return CanUse64BitLiterals && (!isInt<32>(Val) || !isUInt<32>(Val)) ? 254
                                                                      : 255;
}

std::optional<uint64_t> AMDGPUMCCodeEmitter::getLitEncoding(
    const MCInstrDesc &Desc, const MCOperand &MO, unsigned OpNo,
    const MCSubtargetInfo &STI, bool HasMandatoryLiteral) const {
  const MCOperandInfo &OpInfo = Desc.operands()[OpNo];
  int64_t Imm = 0;
  if (MO.isExpr()) {
    if (!MO.getExpr()->evaluateAsAbsolute(Imm) ||
        AMDGPU::isLitExpr(MO.getExpr())) {
      if (OpInfo.OperandType == AMDGPU::OPERAND_KIMM16 ||
          OpInfo.OperandType == AMDGPU::OPERAND_KIMM32 ||
          OpInfo.OperandType == AMDGPU::OPERAND_KIMM64)
        return Imm;
      if (STI.hasFeature(AMDGPU::Feature64BitLiterals) &&
          AMDGPU::getOperandSize(OpInfo) == 8 &&
          AMDGPU::getExprKind(MO.getExpr()) != AMDGPUMCExpr::AGVK_Lit)
        return 254;
      return 255;
    }
  } else {
    assert(!MO.isDFPImm());

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPUMCCodeEmitter::getLitEncoding`, `AMDGPU::isLitExpr`, `AMDGPU::getOperandSize`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPUMCCodeEmitter::getLitEncoding`, `AMDGPU::isLitExpr`, `AMDGPU::getOperandSize`。

### Lines 297-327: Switch-based control flow
```cpp
    if (!MO.isImm())
      return {};

    Imm = MO.getImm();
  }

  switch (OpInfo.OperandType) {
  case AMDGPU::OPERAND_REG_IMM_INT32:
  case AMDGPU::OPERAND_REG_IMM_FP32:
  case AMDGPU::OPERAND_REG_INLINE_C_INT32:
  case AMDGPU::OPERAND_REG_INLINE_C_FP32:
  case AMDGPU::OPERAND_REG_INLINE_AC_INT32:
  case AMDGPU::OPERAND_REG_INLINE_AC_FP32:
  case AMDGPU::OPERAND_REG_IMM_V2INT32:
  case AMDGPU::OPERAND_REG_IMM_V2FP32:
  case AMDGPU::OPERAND_INLINE_SPLIT_BARRIER_INT32:
    return getLit32Encoding(static_cast<uint32_t>(Imm), STI);

  case AMDGPU::OPERAND_REG_IMM_INT64:
  case AMDGPU::OPERAND_REG_INLINE_C_INT64:
    return getLit64Encoding(Desc, static_cast<uint64_t>(Imm), STI, false);

  case AMDGPU::OPERAND_REG_INLINE_C_FP64:
  case AMDGPU::OPERAND_REG_INLINE_AC_FP64:
    return getLit64Encoding(Desc, static_cast<uint64_t>(Imm), STI, true);

  case AMDGPU::OPERAND_REG_IMM_FP64: {
    auto Enc = getLit64Encoding(Desc, static_cast<uint64_t>(Imm), STI, true);
    return (HasMandatoryLiteral && Enc == 255) ? 254 : Enc;
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 328-361: Result computation and returns
```cpp
  case AMDGPU::OPERAND_REG_IMM_INT16:
  case AMDGPU::OPERAND_REG_INLINE_C_INT16:
    return getLit16IntEncoding(static_cast<uint32_t>(Imm), STI);

  case AMDGPU::OPERAND_REG_IMM_FP16:
  case AMDGPU::OPERAND_REG_INLINE_C_FP16:
    // FIXME Is this correct? What do inline immediates do on SI for f16 src
    // which does not have f16 support?
    return getLit16Encoding(static_cast<uint16_t>(Imm), STI);

  case AMDGPU::OPERAND_REG_IMM_BF16:
  case AMDGPU::OPERAND_REG_INLINE_C_BF16:
    // We don't actually need to check Inv2Pi here because BF16 instructions can
    // only be emitted for targets that already support the feature.
    return getLitBF16Encoding(static_cast<uint16_t>(Imm));

  case AMDGPU::OPERAND_REG_IMM_V2INT16:
  case AMDGPU::OPERAND_REG_INLINE_C_V2INT16:
    return AMDGPU::getInlineEncodingV2I16(static_cast<uint32_t>(Imm))
        .value_or(255);

  case AMDGPU::OPERAND_REG_IMM_V2FP16:
  case AMDGPU::OPERAND_REG_INLINE_C_V2FP16:
    return AMDGPU::getInlineEncodingV2F16(static_cast<uint32_t>(Imm))
        .value_or(255);

  case AMDGPU::OPERAND_REG_IMM_V2FP16_SPLAT:
    // V_PK_FMAC_F16 has different inline constant behavior on pre-GFX11 vs
    // GFX11+: pre-GFX11 produces (f16, 0), GFX11+ duplicates f16 to both
    // halves.
    return AMDGPU::getPKFMACF16InlineEncoding(static_cast<uint32_t>(Imm),
                                              AMDGPU::isGFX11Plus(STI))
        .value_or(255);

```
**EN:** This section finalizes intermediate state and returns the value that the caller or pass pipeline needs. Main symbols: `AMDGPU::getInlineEncodingV2I16`, `AMDGPU::getInlineEncodingV2F16`, `AMDGPU::getPKFMACF16InlineEncoding`.
**CN:** 本节整理中间状态并返回调用者或 Pass 流水线所需的结果。 主要符号：`AMDGPU::getInlineEncodingV2I16`, `AMDGPU::getInlineEncodingV2F16`, `AMDGPU::getPKFMACF16InlineEncoding`。

### Lines 362-392: Namespace declarations and scope setup
```cpp
  case AMDGPU::OPERAND_REG_IMM_V2BF16:
  case AMDGPU::OPERAND_REG_INLINE_C_V2BF16:
    return AMDGPU::getInlineEncodingV2BF16(static_cast<uint32_t>(Imm))
        .value_or(255);

  case AMDGPU::OPERAND_REG_IMM_NOINLINE_V2FP16:
    return 255;

  case AMDGPU::OPERAND_KIMM32:
  case AMDGPU::OPERAND_KIMM16:
  case AMDGPU::OPERAND_KIMM64:
    return Imm;
  default:
    llvm_unreachable("invalid operand size");
  }
}

uint64_t AMDGPUMCCodeEmitter::getImplicitOpSelHiEncoding(int Opcode) const {
  using namespace AMDGPU::VOP3PEncoding;

  if (AMDGPU::hasNamedOperand(Opcode, AMDGPU::OpName::op_sel_hi)) {
    if (AMDGPU::hasNamedOperand(Opcode, AMDGPU::OpName::src2))
      return 0;
    if (AMDGPU::hasNamedOperand(Opcode, AMDGPU::OpName::src1))
      return OP_SEL_HI_2;
    if (AMDGPU::hasNamedOperand(Opcode, AMDGPU::OpName::src0))
      return OP_SEL_HI_1 | OP_SEL_HI_2;
  }
  return OP_SEL_HI_0 | OP_SEL_HI_1 | OP_SEL_HI_2;
}

```
**EN:** This section arranges declarations into namespace scope so related AMDGPU functionality stays organized and avoids symbol collisions. Main symbols: `AMDGPU::getInlineEncodingV2BF16`, `AMDGPUMCCodeEmitter::getImplicitOpSelHiEncoding`, `AMDGPU::hasNamedOperand`.
**CN:** 本节将声明组织到命名空间作用域中，使相关 AMDGPU 功能保持清晰并避免符号冲突。 主要符号：`AMDGPU::getInlineEncodingV2BF16`, `AMDGPUMCCodeEmitter::getImplicitOpSelHiEncoding`, `AMDGPU::hasNamedOperand`。

### Lines 393-420: Implements AMDGPUMCCodeEmitter::encodeInstruction
```cpp
void AMDGPUMCCodeEmitter::encodeInstruction(const MCInst &MI,
                                            SmallVectorImpl<char> &CB,
                                            SmallVectorImpl<MCFixup> &Fixups,
                                            const MCSubtargetInfo &STI) const {
  int Opcode = MI.getOpcode();
  APInt Encoding, Scratch;
  getBinaryCodeForInstr(MI, Fixups, Encoding, Scratch,  STI);
  const MCInstrDesc &Desc = MCII.get(MI.getOpcode());
  unsigned bytes = Desc.getSize();

  // Set unused op_sel_hi bits to 1 for VOP3P and MAI instructions.
  // Note that accvgpr_read/write are MAI, have src0, but do not use op_sel.
  if (((Desc.TSFlags & SIInstrFlags::VOP3P) ||
       Opcode == AMDGPU::V_ACCVGPR_READ_B32_vi ||
       Opcode == AMDGPU::V_ACCVGPR_WRITE_B32_vi) &&
      // Matrix B format operand reuses op_sel_hi.
      !AMDGPU::hasNamedOperand(Opcode, AMDGPU::OpName::matrix_b_fmt) &&
      // Matrix B scale operand reuses op_sel_hi.
      !AMDGPU::hasNamedOperand(Opcode, AMDGPU::OpName::matrix_b_scale) &&
      // Matrix B reuse operand reuses op_sel_hi.
      !AMDGPU::hasNamedOperand(Opcode, AMDGPU::OpName::matrix_b_reuse)) {
    Encoding |= getImplicitOpSelHiEncoding(Opcode);
  }

  for (unsigned i = 0; i < bytes; i++) {
    CB.push_back((uint8_t)Encoding.extractBitsAsZExtValue(8, 8 * i));
  }

```
**EN:** This section contains concrete logic for AMDGPUMCCodeEmitter::encodeInstruction. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUMCCodeEmitter::encodeInstruction`, `AMDGPU::hasNamedOperand`.
**CN:** 本节包含与 AMDGPUMCCodeEmitter::encodeInstruction 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUMCCodeEmitter::encodeInstruction`, `AMDGPU::hasNamedOperand`。

### Lines 421-453: Conditional logic and checks
```cpp
  // NSA encoding.
  if (AMDGPU::isGFX10Plus(STI) && Desc.TSFlags & SIInstrFlags::MIMG) {
    int vaddr0 = AMDGPU::getNamedOperandIdx(MI.getOpcode(),
                                            AMDGPU::OpName::vaddr0);
    int srsrc = AMDGPU::getNamedOperandIdx(MI.getOpcode(),
                                           AMDGPU::OpName::srsrc);
    assert(vaddr0 >= 0 && srsrc > vaddr0);
    unsigned NumExtraAddrs = srsrc - vaddr0 - 1;
    unsigned NumPadding = (-NumExtraAddrs) & 3;

    for (unsigned i = 0; i < NumExtraAddrs; ++i) {
      getMachineOpValue(MI, MI.getOperand(vaddr0 + 1 + i), Encoding, Fixups,
                        STI);
      CB.push_back((uint8_t)Encoding.getLimitedValue());
    }
    CB.append(NumPadding, 0);
  }

  if ((bytes > 8 && STI.hasFeature(AMDGPU::FeatureVOP3Literal)) ||
      (bytes > 4 && !STI.hasFeature(AMDGPU::FeatureVOP3Literal)))
    return;

  // Do not print literals from SISrc Operands for insts with mandatory literals
  if (AMDGPU::hasNamedOperand(MI.getOpcode(), AMDGPU::OpName::imm))
    return;

  // Check for additional literals
  for (unsigned i = 0, e = Desc.getNumOperands(); i < e; ++i) {

    // Check if this operand should be encoded as [SV]Src
    if (!AMDGPU::isSISrcOperand(Desc, i))
      continue;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::isGFX10Plus`, `AMDGPU::getNamedOperandIdx`, `AMDGPU::hasNamedOperand`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::isGFX10Plus`, `AMDGPU::getNamedOperandIdx`, `AMDGPU::hasNamedOperand`。

### Lines 454-485: Conditional logic and checks
```cpp
    // Is this operand a literal immediate?
    const MCOperand &Op = MI.getOperand(i);
    auto Enc = getLitEncoding(Desc, Op, i, STI);
    if (!Enc || (*Enc != 255 && *Enc != 254))
      continue;

    // Yes! Encode it
    int64_t Imm = 0;

    bool IsLit = false;
    if (Op.isImm())
      Imm = Op.getImm();
    else if (Op.isExpr()) {
      if (const auto *C = dyn_cast<MCConstantExpr>(Op.getExpr())) {
        Imm = C->getValue();
      } else if (AMDGPU::isLitExpr(Op.getExpr())) {
        IsLit = true;
        Imm = AMDGPU::getLitValue(Op.getExpr());
      }
    } else // Exprs will be replaced with a fixup value.
      llvm_unreachable("Must be immediate or expr");

    if (*Enc == 254) {
      assert(STI.hasFeature(AMDGPU::Feature64BitLiterals));
      support::endian::write<uint64_t>(CB, Imm, llvm::endianness::little);
    } else {
      auto OpType =
          static_cast<AMDGPU::OperandType>(Desc.operands()[i].OperandType);
      Imm = AMDGPU::encode32BitLiteral(Imm, OpType, IsLit);
      support::endian::write<uint32_t>(CB, Imm, llvm::endianness::little);
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::isLitExpr`, `AMDGPU::getLitValue`, `AMDGPU::encode32BitLiteral`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::isLitExpr`, `AMDGPU::getLitValue`, `AMDGPU::encode32BitLiteral`。

### Lines 486-514: Implements AMDGPUMCCodeEmitter::getSOPPBrEncoding
```cpp
    // Only one literal value allowed
    break;
  }
}

void AMDGPUMCCodeEmitter::getSOPPBrEncoding(const MCInst &MI, unsigned OpNo,
                                            APInt &Op,
                                            SmallVectorImpl<MCFixup> &Fixups,
                                            const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);

  if (MO.isExpr()) {
    const MCExpr *Expr = MO.getExpr();
    addFixup(Fixups, 0, Expr, AMDGPU::fixup_si_sopp_br, true);
    Op = APInt::getZero(96);
  } else {
    getMachineOpValue(MI, MO, Op, Fixups, STI);
  }
}

void AMDGPUMCCodeEmitter::getSMEMOffsetEncoding(
    const MCInst &MI, unsigned OpNo, APInt &Op,
    SmallVectorImpl<MCFixup> &Fixups, const MCSubtargetInfo &STI) const {
  auto Offset = MI.getOperand(OpNo).getImm();
  // VI only supports 20-bit unsigned offsets.
  assert(!AMDGPU::isVI(STI) || isUInt<20>(Offset));
  Op = Offset;
}

```
**EN:** This section contains concrete logic for AMDGPUMCCodeEmitter::getSOPPBrEncoding. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUMCCodeEmitter::getSOPPBrEncoding`, `APInt::getZero`, `AMDGPUMCCodeEmitter::getSMEMOffsetEncoding`.
**CN:** 本节包含与 AMDGPUMCCodeEmitter::getSOPPBrEncoding 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUMCCodeEmitter::getSOPPBrEncoding`, `APInt::getZero`, `AMDGPUMCCodeEmitter::getSMEMOffsetEncoding`。

### Lines 515-545: Implements AMDGPUMCCodeEmitter::getSDWASrcEncoding
```cpp
void AMDGPUMCCodeEmitter::getSDWASrcEncoding(const MCInst &MI, unsigned OpNo,
                                             APInt &Op,
                                             SmallVectorImpl<MCFixup> &Fixups,
                                             const MCSubtargetInfo &STI) const {
  using namespace AMDGPU::SDWA;

  uint64_t RegEnc = 0;

  const MCOperand &MO = MI.getOperand(OpNo);

  if (MO.isReg()) {
    MCRegister Reg = MO.getReg();
    RegEnc |= MRI.getEncodingValue(Reg);
    RegEnc &= SDWA9EncValues::SRC_VGPR_MASK;
    if (AMDGPU::isSGPR(AMDGPU::mc2PseudoReg(Reg), &MRI)) {
      RegEnc |= SDWA9EncValues::SRC_SGPR_MASK;
    }
    Op = RegEnc;
    return;
  } else {
    const MCInstrDesc &Desc = MCII.get(MI.getOpcode());
    auto Enc = getLitEncoding(Desc, MO, OpNo, STI);
    if (Enc && *Enc != 255) {
      Op = *Enc | SDWA9EncValues::SRC_SGPR_MASK;
      return;
    }
  }

  llvm_unreachable("Unsupported operand kind");
}

```
**EN:** This section contains concrete logic for AMDGPUMCCodeEmitter::getSDWASrcEncoding. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUMCCodeEmitter::getSDWASrcEncoding`, `AMDGPU::isSGPR`, `AMDGPU::mc2PseudoReg`.
**CN:** 本节包含与 AMDGPUMCCodeEmitter::getSDWASrcEncoding 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUMCCodeEmitter::getSDWASrcEncoding`, `AMDGPU::isSGPR`, `AMDGPU::mc2PseudoReg`。

### Lines 546-577: Implements AMDGPUMCCodeEmitter::getSDWAVopcDstEncoding
```cpp
void AMDGPUMCCodeEmitter::getSDWAVopcDstEncoding(
    const MCInst &MI, unsigned OpNo, APInt &Op,
    SmallVectorImpl<MCFixup> &Fixups, const MCSubtargetInfo &STI) const {
  using namespace AMDGPU::SDWA;

  uint64_t RegEnc = 0;

  const MCOperand &MO = MI.getOperand(OpNo);

  MCRegister Reg = MO.getReg();
  if (Reg != AMDGPU::VCC && Reg != AMDGPU::VCC_LO) {
    RegEnc |= MRI.getEncodingValue(Reg);
    RegEnc &= SDWA9EncValues::VOPC_DST_SGPR_MASK;
    RegEnc |= SDWA9EncValues::VOPC_DST_VCC_MASK;
  }
  Op = RegEnc;
}

void AMDGPUMCCodeEmitter::getAVOperandEncoding(
    const MCInst &MI, unsigned OpNo, APInt &Op,
    SmallVectorImpl<MCFixup> &Fixups, const MCSubtargetInfo &STI) const {
  MCRegister Reg = MI.getOperand(OpNo).getReg();
  unsigned Enc = MRI.getEncodingValue(Reg);
  unsigned Idx = Enc & AMDGPU::HWEncoding::LO256_REG_IDX_MASK;
  bool IsVGPROrAGPR =
      Enc & (AMDGPU::HWEncoding::IS_VGPR | AMDGPU::HWEncoding::IS_AGPR);

  // VGPR and AGPR have the same encoding, but SrcA and SrcB operands of mfma
  // instructions use acc[0:1] modifier bits to distinguish. These bits are
  // encoded as a virtual 9th bit of the register for these operands.
  bool IsAGPR = Enc & AMDGPU::HWEncoding::IS_AGPR;

```
**EN:** This section contains concrete logic for AMDGPUMCCodeEmitter::getSDWAVopcDstEncoding. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUMCCodeEmitter::getSDWAVopcDstEncoding`, `AMDGPUMCCodeEmitter::getAVOperandEncoding`.
**CN:** 本节包含与 AMDGPUMCCodeEmitter::getSDWAVopcDstEncoding 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUMCCodeEmitter::getSDWAVopcDstEncoding`, `AMDGPUMCCodeEmitter::getAVOperandEncoding`。

### Lines 578-604: Defines needsPCRel
```cpp
  Op = Idx | (IsVGPROrAGPR << 8) | (IsAGPR << 9);
}

static bool needsPCRel(const MCExpr *Expr) {
  switch (Expr->getKind()) {
  case MCExpr::SymbolRef: {
    auto *SE = cast<MCSymbolRefExpr>(Expr);
    auto Spec = AMDGPU::getSpecifier(SE);
    return Spec != AMDGPUMCExpr::S_ABS32_LO &&
           Spec != AMDGPUMCExpr::S_ABS32_HI && Spec != AMDGPUMCExpr::S_ABS64;
  }
  case MCExpr::Binary: {
    auto *BE = cast<MCBinaryExpr>(Expr);
    if (BE->getOpcode() == MCBinaryExpr::Sub)
      return false;
    return needsPCRel(BE->getLHS()) || needsPCRel(BE->getRHS());
  }
  case MCExpr::Unary:
    return needsPCRel(cast<MCUnaryExpr>(Expr)->getSubExpr());
  case MCExpr::Specifier:
  case MCExpr::Target:
  case MCExpr::Constant:
    return false;
  }
  llvm_unreachable("invalid kind");
}

```
**EN:** This section contains concrete logic for needsPCRel. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU::getSpecifier`.
**CN:** 本节包含与 needsPCRel 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU::getSpecifier`。

### Lines 605-638: Implements AMDGPUMCCodeEmitter::getMachineOpValue
```cpp
void AMDGPUMCCodeEmitter::getMachineOpValue(const MCInst &MI,
                                            const MCOperand &MO, APInt &Op,
                                            SmallVectorImpl<MCFixup> &Fixups,
                                            const MCSubtargetInfo &STI) const {
  if (MO.isReg()){
    unsigned Enc = MRI.getEncodingValue(MO.getReg());
    unsigned Idx = Enc & AMDGPU::HWEncoding::LO256_REG_IDX_MASK;
    bool IsVGPROrAGPR =
        Enc & (AMDGPU::HWEncoding::IS_VGPR | AMDGPU::HWEncoding::IS_AGPR);
    Op = Idx | (IsVGPROrAGPR << 8);
    return;
  }
  unsigned OpNo = &MO - MI.begin();
  getMachineOpValueCommon(MI, MO, OpNo, Op, Fixups, STI);
}

void AMDGPUMCCodeEmitter::getMachineOpValueT16(
    const MCInst &MI, unsigned OpNo, APInt &Op,
    SmallVectorImpl<MCFixup> &Fixups, const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isReg()) {
    unsigned Enc = MRI.getEncodingValue(MO.getReg());
    unsigned Idx = Enc & AMDGPU::HWEncoding::REG_IDX_MASK;
    bool IsVGPR = Enc & AMDGPU::HWEncoding::IS_VGPR;
    Op = Idx | (IsVGPR << 8);
    return;
  }
  getMachineOpValueCommon(MI, MO, OpNo, Op, Fixups, STI);
  // VGPRs include the suffix/op_sel bit in the register encoding, but
  // immediates and SGPRs include it in src_modifiers. Therefore, copy the
  // op_sel bit from the src operands into src_modifier operands if Op is
  // src_modifiers and the corresponding src is a VGPR
  int SrcMOIdx = -1;
  assert(OpNo < INT_MAX);
```
**EN:** This section contains concrete logic for AMDGPUMCCodeEmitter::getMachineOpValue. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUMCCodeEmitter::getMachineOpValue`, `AMDGPUMCCodeEmitter::getMachineOpValueT16`.
**CN:** 本节包含与 AMDGPUMCCodeEmitter::getMachineOpValue 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUMCCodeEmitter::getMachineOpValue`, `AMDGPUMCCodeEmitter::getMachineOpValueT16`。

### Lines 639-667: Implements AMDGPU::getNamedOperandIdx
```cpp
  if ((int)OpNo == AMDGPU::getNamedOperandIdx(MI.getOpcode(),
                                              AMDGPU::OpName::src0_modifiers)) {
    SrcMOIdx = AMDGPU::getNamedOperandIdx(MI.getOpcode(), AMDGPU::OpName::src0);
    int VDstMOIdx =
        AMDGPU::getNamedOperandIdx(MI.getOpcode(), AMDGPU::OpName::vdst);
    if (VDstMOIdx != -1) {
      auto DstReg = MI.getOperand(VDstMOIdx).getReg();
      if (AMDGPU::isHi16Reg(DstReg, MRI))
        Op |= SISrcMods::DST_OP_SEL;
    }
  } else if ((int)OpNo == AMDGPU::getNamedOperandIdx(
                              MI.getOpcode(), AMDGPU::OpName::src1_modifiers))
    SrcMOIdx = AMDGPU::getNamedOperandIdx(MI.getOpcode(), AMDGPU::OpName::src1);
  else if ((int)OpNo == AMDGPU::getNamedOperandIdx(
                            MI.getOpcode(), AMDGPU::OpName::src2_modifiers))
    SrcMOIdx = AMDGPU::getNamedOperandIdx(MI.getOpcode(), AMDGPU::OpName::src2);
  if (SrcMOIdx == -1)
    return;

  const MCOperand &SrcMO = MI.getOperand(SrcMOIdx);
  if (!SrcMO.isReg())
    return;
  auto SrcReg = SrcMO.getReg();
  if (AMDGPU::isSGPR(SrcReg, &MRI))
    return;
  if (AMDGPU::isHi16Reg(SrcReg, MRI))
    Op |= SISrcMods::OP_SEL_0;
}

```
**EN:** This section contains concrete logic for AMDGPU::getNamedOperandIdx. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU::getNamedOperandIdx`, `AMDGPU::isHi16Reg`, `AMDGPU::isSGPR`.
**CN:** 本节包含与 AMDGPU::getNamedOperandIdx 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU::getNamedOperandIdx`, `AMDGPU::isHi16Reg`, `AMDGPU::isSGPR`。

### Lines 668-689: Implements AMDGPUMCCodeEmitter::getMachineOpValueT16Lo128
```cpp
void AMDGPUMCCodeEmitter::getMachineOpValueT16Lo128(
    const MCInst &MI, unsigned OpNo, APInt &Op,
    SmallVectorImpl<MCFixup> &Fixups, const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isReg()) {
    uint16_t Encoding = MRI.getEncodingValue(MO.getReg());
    unsigned RegIdx = Encoding & AMDGPU::HWEncoding::LO256_REG_IDX_MASK;
    bool IsHi = Encoding & AMDGPU::HWEncoding::IS_HI16;
    bool IsVGPR = Encoding & AMDGPU::HWEncoding::IS_VGPR;
    assert((!IsVGPR || isUInt<7>(RegIdx)) && "VGPR0-VGPR127 expected!");
    Op = (IsVGPR ? 0x100 : 0) | (IsHi ? 0x80 : 0) | RegIdx;
    return;
  }
  getMachineOpValueCommon(MI, MO, OpNo, Op, Fixups, STI);
}

void AMDGPUMCCodeEmitter::getMachineOpValueCommon(
    const MCInst &MI, const MCOperand &MO, unsigned OpNo, APInt &Op,
    SmallVectorImpl<MCFixup> &Fixups, const MCSubtargetInfo &STI) const {
  bool isLikeImm = false;
  int64_t Val;

```
**EN:** This section contains concrete logic for AMDGPUMCCodeEmitter::getMachineOpValueT16Lo128. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUMCCodeEmitter::getMachineOpValueT16Lo128`, `AMDGPUMCCodeEmitter::getMachineOpValueCommon`.
**CN:** 本节包含与 AMDGPUMCCodeEmitter::getMachineOpValueT16Lo128 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUMCCodeEmitter::getMachineOpValueT16Lo128`, `AMDGPUMCCodeEmitter::getMachineOpValueCommon`。

### Lines 690-723: Conditional logic and checks
```cpp
  if (MO.isImm()) {
    Val = MO.getImm();
    isLikeImm = true;
  } else if (MO.isExpr() && MO.getExpr()->evaluateAsAbsolute(Val)) {
    isLikeImm = true;
  } else if (MO.isExpr()) {
    // FIXME: If this is expression is PCRel or not should not depend on what
    // the expression looks like. Given that this is just a general expression,
    // it should probably be FK_Data_4 and whatever is producing
    //
    //    s_add_u32 s2, s2, (extern_const_addrspace+16
    //
    // And expecting a PCRel should instead produce
    //
    // .Ltmp1:
    //   s_add_u32 s2, s2, (extern_const_addrspace+16)-.Ltmp1
    bool PCRel = needsPCRel(MO.getExpr());
    const MCInstrDesc &Desc = MCII.get(MI.getOpcode());
    uint32_t Offset = Desc.getSize();
    assert(Offset == 4 || Offset == 8);
    unsigned Size = AMDGPU::getOperandSize(Desc, OpNo);
    MCFixupKind Kind = MCFixup::getDataKindForSize(Size);
    addFixup(Fixups, Offset, MO.getExpr(), Kind, PCRel);
  }

  const MCInstrDesc &Desc = MCII.get(MI.getOpcode());
  if (AMDGPU::isSISrcOperand(Desc, OpNo)) {
    bool HasMandatoryLiteral =
        AMDGPU::hasNamedOperand(MI.getOpcode(), AMDGPU::OpName::imm);
    if (auto Enc = getLitEncoding(Desc, MO, OpNo, STI, HasMandatoryLiteral)) {
      Op = *Enc;
      return;
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::getOperandSize`, `MCFixup::getDataKindForSize`, `AMDGPU::isSISrcOperand`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::getOperandSize`, `MCFixup::getDataKindForSize`, `AMDGPU::isSISrcOperand`。

### Lines 724-751: Declares llvm_unreachable
```cpp
    llvm_unreachable("Operand not supported for SISrc");
  }

  if (isLikeImm) {
    Op = Val;
    return;
  }

  llvm_unreachable("Encoding of this operand type is not supported yet.");
}

template <bool HasSrc0, bool HasSrc1, bool HasSrc2>
APInt AMDGPUMCCodeEmitter::postEncodeVOP3(const MCInst &MI, APInt EncodedValue,
                                          const MCSubtargetInfo &STI) const {
  if (!AMDGPU::isGFX10Plus(STI))
    return EncodedValue;
  // Set unused source fields in VOP3 encodings to inline immediate 0 to avoid
  // hardware conservatively assuming the instruction reads SGPRs.
  constexpr uint64_t InlineImmediate0 = 0x80;
  if (!HasSrc0)
    EncodedValue |= InlineImmediate0 << 32;
  if (!HasSrc1)
    EncodedValue |= InlineImmediate0 << 41;
  if (!HasSrc2)
    EncodedValue |= InlineImmediate0 << 50;
  return EncodedValue;
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPUMCCodeEmitter::postEncodeVOP3`, `AMDGPU::isGFX10Plus`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPUMCCodeEmitter::postEncodeVOP3`, `AMDGPU::isGFX10Plus`。

### Lines 752-768: Header dependencies and setup
```cpp
APInt AMDGPUMCCodeEmitter::postEncodeVOPCX(const MCInst &MI, APInt EncodedValue,
                                           const MCSubtargetInfo &STI) const {
  // GFX10+ v_cmpx opcodes promoted to VOP3 have implied dst=EXEC.
  // Documentation requires dst to be encoded as EXEC (0x7E),
  // but it looks like the actual value encoded for dst operand
  // is ignored by HW. It was decided to define dst as "do not care"
  // in td files to allow disassembler accept any dst value.
  // However, dst is encoded as EXEC for compatibility with SP3.
  [[maybe_unused]] const MCInstrDesc &Desc = MCII.get(MI.getOpcode());
  assert((Desc.TSFlags & SIInstrFlags::VOP3) &&
         Desc.hasImplicitDefOfPhysReg(AMDGPU::EXEC));
  EncodedValue |= MRI.getEncodingValue(AMDGPU::EXEC_LO) &
                  AMDGPU::HWEncoding::LO256_REG_IDX_MASK;
  return postEncodeVOP3<true, true, false>(MI, EncodedValue, STI);
}

#include "AMDGPUGenMCCodeEmitter.inc"
```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `AMDGPUMCCodeEmitter::postEncodeVOPCX`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`AMDGPUMCCodeEmitter::postEncodeVOPCX`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPUMCCodeEmitter`, `llvm::createAMDGPUMCCodeEmitter`, `MCFixup::create`, `std::abs`, `AMDGPUMCCodeEmitter::getLitEncoding`, `AMDGPU::isLitExpr`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; assembly handling / 汇编处理
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"MCTargetDesc/AMDGPUFixupKinds.h"`
- `"MCTargetDesc/AMDGPUMCExpr.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"SIDefines.h"`
- `"Utils/AMDGPUBaseInfo.h"`
- `"llvm/ADT/APInt.h"`
- `"llvm/MC/MCCodeEmitter.h"`
- `"llvm/MC/MCContext.h"`
- `"llvm/MC/MCExpr.h"`
- `"llvm/MC/MCInstrInfo.h"`
- `"llvm/MC/MCRegisterInfo.h"`
- `"llvm/MC/MCSubtargetInfo.h"`
- `"llvm/Support/Casting.h"`
- `"llvm/Support/EndianStream.h"`
- `<optional>`
- `"AMDGPUGenMCCodeEmitter.inc"`
