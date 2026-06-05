# R600MCCodeEmitter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/MCTargetDesc/R600MCCodeEmitter.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements R600MCCodeEmitter for the LLVM MC target description layer. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM MC 目标描述层中的 R600MCCodeEmitter 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: File banner, license, and overview
```cpp
//===- R600MCCodeEmitter.cpp - Code Emitter for R600->Cayman GPU families -===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
///
/// The R600 code emitter produces machine code that can be executed
/// directly on the GPU device.
//
//===----------------------------------------------------------------------===//

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。

### Lines 16-39: Header dependencies and setup
```cpp
#include "MCTargetDesc/R600MCTargetDesc.h"
#include "R600Defines.h"
#include "llvm/MC/MCCodeEmitter.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/Support/EndianStream.h"

using namespace llvm;

namespace {

class R600MCCodeEmitter : public MCCodeEmitter {
  const MCRegisterInfo &MRI;
  const MCInstrInfo &MCII;

public:
  R600MCCodeEmitter(const MCInstrInfo &mcii, const MCRegisterInfo &mri)
    : MRI(mri), MCII(mcii) {}
  R600MCCodeEmitter(const R600MCCodeEmitter &) = delete;
  R600MCCodeEmitter &operator=(const R600MCCodeEmitter &) = delete;

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `R600MCCodeEmitter`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`R600MCCodeEmitter`。

### Lines 40-62: Defines encodeInstruction
```cpp
  /// Encode the instruction and write it to the OS.
  void encodeInstruction(const MCInst &MI, SmallVectorImpl<char> &CB,
                         SmallVectorImpl<MCFixup> &Fixups,
                         const MCSubtargetInfo &STI) const override;

  /// \returns the encoding for an MCOperand.
  uint64_t getMachineOpValue(const MCInst &MI, const MCOperand &MO,
                             SmallVectorImpl<MCFixup> &Fixups,
                             const MCSubtargetInfo &STI) const;

private:
  void emit(uint32_t value, SmallVectorImpl<char> &CB) const;
  void emit(uint64_t value, SmallVectorImpl<char> &CB) const;

  unsigned getHWReg(MCRegister Reg) const;

  uint64_t getBinaryCodeForInstr(const MCInst &MI,
                                 SmallVectorImpl<MCFixup> &Fixups,
                                 const MCSubtargetInfo &STI) const;
};

} // end anonymous namespace

```
**EN:** This section contains concrete logic for encodeInstruction. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 encodeInstruction 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 63-84: Type declarations and aliases
```cpp
enum RegElement {
  ELEMENT_X = 0,
  ELEMENT_Y,
  ELEMENT_Z,
  ELEMENT_W
};

enum FCInstr {
  FC_IF_PREDICATE = 0,
  FC_ELSE,
  FC_ENDIF,
  FC_BGNLOOP,
  FC_ENDLOOP,
  FC_BREAK_PREDICATE,
  FC_CONTINUE
};

MCCodeEmitter *llvm::createR600MCCodeEmitter(const MCInstrInfo &MCII,
                                             MCContext &Ctx) {
  return new R600MCCodeEmitter(MCII, *Ctx.getRegisterInfo());
}

```
**EN:** This section introduces supporting types or aliases that simplify later declarations and backend logic. Main symbols: `RegElement`, `FCInstr`, `llvm::createR600MCCodeEmitter`.
**CN:** 本节引入辅助类型或别名，以简化后续声明与后端逻辑。 主要符号：`RegElement`, `FCInstr`, `llvm::createR600MCCodeEmitter`。

### Lines 85-103: Implements R600MCCodeEmitter::encodeInstruction
```cpp
void R600MCCodeEmitter::encodeInstruction(const MCInst &MI,
                                          SmallVectorImpl<char> &CB,
                                          SmallVectorImpl<MCFixup> &Fixups,
                                          const MCSubtargetInfo &STI) const {
  const MCInstrDesc &Desc = MCII.get(MI.getOpcode());
  if (MI.getOpcode() == R600::RETURN ||
    MI.getOpcode() == R600::FETCH_CLAUSE ||
    MI.getOpcode() == R600::ALU_CLAUSE ||
    MI.getOpcode() == R600::BUNDLE ||
    MI.getOpcode() == R600::KILL) {
    return;
  }
  if (IS_VTX(Desc)) {
    uint64_t InstWord01 = getBinaryCodeForInstr(MI, Fixups, STI);
    uint32_t InstWord2 = MI.getOperand(2).getImm(); // Offset
    if (!(STI.hasFeature(R600::FeatureCaymanISA))) {
      InstWord2 |= 1 << 19; // Mega-Fetch bit
    }

```
**EN:** This section contains concrete logic for R600MCCodeEmitter::encodeInstruction. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600MCCodeEmitter::encodeInstruction`.
**CN:** 本节包含与 R600MCCodeEmitter::encodeInstruction 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600MCCodeEmitter::encodeInstruction`。

### Lines 104-122: Conditional logic and checks
```cpp
    emit(InstWord01, CB);
    emit(InstWord2, CB);
    emit((uint32_t)0, CB);
  } else if (IS_TEX(Desc)) {
    int64_t Sampler = MI.getOperand(14).getImm();

    int64_t SrcSelect[4] = {
        MI.getOperand(2).getImm(), MI.getOperand(3).getImm(),
        MI.getOperand(4).getImm(), MI.getOperand(5).getImm()};
    int64_t Offsets[3] = {MI.getOperand(6).getImm() & 0x1F,
                          MI.getOperand(7).getImm() & 0x1F,
                          MI.getOperand(8).getImm() & 0x1F};

    uint64_t Word01 = getBinaryCodeForInstr(MI, Fixups, STI);
    uint32_t Word2 = Sampler << 15 | SrcSelect[ELEMENT_X] << 20 |
                     SrcSelect[ELEMENT_Y] << 23 | SrcSelect[ELEMENT_Z] << 26 |
                     SrcSelect[ELEMENT_W] << 29 | Offsets[0] << 0 |
                     Offsets[1] << 5 | Offsets[2] << 10;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 123-146: Conditional logic and checks
```cpp
    emit(Word01, CB);
    emit(Word2, CB);
    emit((uint32_t)0, CB);
  } else {
    uint64_t Inst = getBinaryCodeForInstr(MI, Fixups, STI);
    if ((STI.hasFeature(R600::FeatureR600ALUInst)) &&
       ((Desc.TSFlags & R600_InstFlag::OP1) ||
         Desc.TSFlags & R600_InstFlag::OP2)) {
      uint64_t ISAOpCode = Inst & (0x3FFULL << 39);
      Inst &= ~(0x3FFULL << 39);
      Inst |= ISAOpCode << 1;
    }
    emit(Inst, CB);
  }
}

void R600MCCodeEmitter::emit(uint32_t Value, SmallVectorImpl<char> &CB) const {
  support::endian::write(CB, Value, llvm::endianness::little);
}

void R600MCCodeEmitter::emit(uint64_t Value, SmallVectorImpl<char> &CB) const {
  support::endian::write(CB, Value, llvm::endianness::little);
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `R600MCCodeEmitter::emit`, `endian::write`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`R600MCCodeEmitter::emit`, `endian::write`。

### Lines 147-160: Implements R600MCCodeEmitter::getHWReg
```cpp
unsigned R600MCCodeEmitter::getHWReg(MCRegister Reg) const {
  return MRI.getEncodingValue(Reg) & HW_REG_MASK;
}

uint64_t R600MCCodeEmitter::getMachineOpValue(const MCInst &MI,
                                              const MCOperand &MO,
                                        SmallVectorImpl<MCFixup> &Fixups,
                                        const MCSubtargetInfo &STI) const {
  if (MO.isReg()) {
    if (HAS_NATIVE_OPERANDS(MCII.get(MI.getOpcode()).TSFlags))
      return MRI.getEncodingValue(MO.getReg());
    return getHWReg(MO.getReg());
  }

```
**EN:** This section contains concrete logic for R600MCCodeEmitter::getHWReg. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600MCCodeEmitter::getHWReg`, `R600MCCodeEmitter::getMachineOpValue`.
**CN:** 本节包含与 R600MCCodeEmitter::getHWReg 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600MCCodeEmitter::getHWReg`, `R600MCCodeEmitter::getMachineOpValue`。

### Lines 161-177: Header dependencies and setup
```cpp
  if (MO.isExpr()) {
    // We put rodata at the end of code section, then map the entire
    // code secetion as vtx buf. Thus the section relative address is the
    // correct one.
    // Each R600 literal instruction has two operands
    // We can't easily get the order of the current one, so compare against
    // the first one and adjust offset.
    const unsigned offset = (&MO == &MI.getOperand(0)) ? 0 : 4;
    Fixups.push_back(MCFixup::create(offset, MO.getExpr(), FK_SecRel_4));
    return 0;
  }

  assert(MO.isImm());
  return MO.getImm();
}

#include "R600GenMCCodeEmitter.inc"
```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `MCFixup::create`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`MCFixup::create`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `R600MCCodeEmitter`, `RegElement`, `FCInstr`, `llvm::createR600MCCodeEmitter`, `R600MCCodeEmitter::encodeInstruction`, `R600MCCodeEmitter::emit`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; predicates and constraints / 谓词与约束
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"MCTargetDesc/R600MCTargetDesc.h"`
- `"R600Defines.h"`
- `"llvm/MC/MCCodeEmitter.h"`
- `"llvm/MC/MCContext.h"`
- `"llvm/MC/MCInst.h"`
- `"llvm/MC/MCInstrInfo.h"`
- `"llvm/MC/MCRegisterInfo.h"`
- `"llvm/MC/MCSubtargetInfo.h"`
- `"llvm/Support/EndianStream.h"`
- `"R600GenMCCodeEmitter.inc"`
