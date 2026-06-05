# R600AsmPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/R600AsmPrinter.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements R600AsmPrinter for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 R600AsmPrinter 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: File banner, license, and overview
```cpp
//===-- R600AsmPrinter.cpp - R600 Assembly printer ------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
///
/// The R600AsmPrinter is used to print both assembly string and also binary
/// code.  When passed an MCAsmStreamer it prints assembly and when passed
/// an MCObjectStreamer it outputs binary code.
//
//===----------------------------------------------------------------------===//

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。

### Lines 17-39: Header dependencies and setup
```cpp
#include "R600AsmPrinter.h"
#include "MCTargetDesc/R600MCTargetDesc.h"
#include "R600Defines.h"
#include "R600MachineFunctionInfo.h"
#include "R600Subtarget.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCSectionELF.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/Target/TargetLoweringObjectFile.h"

using namespace llvm;

AsmPrinter *
llvm::createR600AsmPrinterPass(TargetMachine &TM,
                               std::unique_ptr<MCStreamer> &&Streamer) {
  return new R600AsmPrinter(TM, std::move(Streamer));
}

R600AsmPrinter::R600AsmPrinter(TargetMachine &TM,
                               std::unique_ptr<MCStreamer> Streamer)
  : AsmPrinter(TM, std::move(Streamer)) { }

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `llvm::createR600AsmPrinterPass`, `std::move`, `R600AsmPrinter::R600AsmPrinter`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`llvm::createR600AsmPrinterPass`, `std::move`, `R600AsmPrinter::R600AsmPrinter`。

### Lines 40-61: Implements R600AsmPrinter::getPassName
```cpp
StringRef R600AsmPrinter::getPassName() const {
  return "R600 Assembly Printer";
}

void R600AsmPrinter::EmitProgramInfoR600(const MachineFunction &MF) {
  unsigned MaxGPR = 0;
  bool killPixel = false;
  const R600Subtarget &STM = MF.getSubtarget<R600Subtarget>();
  const R600RegisterInfo *RI = STM.getRegisterInfo();
  const R600MachineFunctionInfo *MFI = MF.getInfo<R600MachineFunctionInfo>();

  for (const MachineBasicBlock &MBB : MF) {
    for (const MachineInstr &MI : MBB) {
      if (MI.getOpcode() == R600::KILLGT)
        killPixel = true;
      unsigned numOperands = MI.getNumOperands();
      for (unsigned op_idx = 0; op_idx < numOperands; op_idx++) {
        const MachineOperand &MO = MI.getOperand(op_idx);
        if (!MO.isReg())
          continue;
        unsigned HWReg = RI->getHWRegIndex(MO.getReg());

```
**EN:** This section contains concrete logic for R600AsmPrinter::getPassName. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600AsmPrinter::getPassName`, `R600AsmPrinter::EmitProgramInfoR600`.
**CN:** 本节包含与 R600AsmPrinter::getPassName 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600AsmPrinter::getPassName`, `R600AsmPrinter::EmitProgramInfoR600`。

### Lines 62-85: Switch-based control flow
```cpp
        // Register with value > 127 aren't GPR
        if (HWReg > 127)
          continue;
        MaxGPR = std::max(MaxGPR, HWReg);
      }
    }
  }

  unsigned RsrcReg;
  if (STM.getGeneration() >= AMDGPUSubtarget::EVERGREEN) {
    // Evergreen / Northern Islands
    switch (MF.getFunction().getCallingConv()) {
    default: [[fallthrough]];
    case CallingConv::AMDGPU_CS: RsrcReg = R_0288D4_SQ_PGM_RESOURCES_LS; break;
    case CallingConv::AMDGPU_GS: RsrcReg = R_028878_SQ_PGM_RESOURCES_GS; break;
    case CallingConv::AMDGPU_PS: RsrcReg = R_028844_SQ_PGM_RESOURCES_PS; break;
    case CallingConv::AMDGPU_VS: RsrcReg = R_028860_SQ_PGM_RESOURCES_VS; break;
    }
  } else {
    // R600 / R700
    switch (MF.getFunction().getCallingConv()) {
    default: [[fallthrough]];
    case CallingConv::AMDGPU_GS: [[fallthrough]];
    case CallingConv::AMDGPU_CS: [[fallthrough]];
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::max`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::max`。

### Lines 86-108: Declares emitInt32
```cpp
    case CallingConv::AMDGPU_VS: RsrcReg = R_028868_SQ_PGM_RESOURCES_VS; break;
    case CallingConv::AMDGPU_PS: RsrcReg = R_028850_SQ_PGM_RESOURCES_PS; break;
    }
  }

  OutStreamer->emitInt32(RsrcReg);
  OutStreamer->emitIntValue(S_NUM_GPRS(MaxGPR + 1) |
                           S_STACK_SIZE(MFI->CFStackSize), 4);
  OutStreamer->emitInt32(R_02880C_DB_SHADER_CONTROL);
  OutStreamer->emitInt32(S_02880C_KILL_ENABLE(killPixel));

  if (AMDGPU::isCompute(MF.getFunction().getCallingConv())) {
    OutStreamer->emitInt32(R_0288E8_SQ_LDS_ALLOC);
    OutStreamer->emitIntValue(alignTo(MFI->getLDSSize(), 4) >> 2, 4);
  }
}

bool R600AsmPrinter::runOnMachineFunction(MachineFunction &MF) {


  // Functions needs to be cacheline (256B) aligned.
  MF.ensureAlignment(Align(256));

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPU::isCompute`, `R600AsmPrinter::runOnMachineFunction`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPU::isCompute`, `R600AsmPrinter::runOnMachineFunction`。

### Lines 109-131: Declares SetupMachineFunction
```cpp
  SetupMachineFunction(MF);

  MCContext &Context = getObjFileLowering().getContext();
  MCSectionELF *ConfigSection =
      Context.getELFSection(".AMDGPU.config", ELF::SHT_PROGBITS, 0);
  OutStreamer->switchSection(ConfigSection);

  EmitProgramInfoR600(MF);

  emitFunctionBody();

  if (isVerbose()) {
    MCSectionELF *CommentSection =
        Context.getELFSection(".AMDGPU.csdata", ELF::SHT_PROGBITS, 0);
    OutStreamer->switchSection(CommentSection);

    R600MachineFunctionInfo *MFI = MF.getInfo<R600MachineFunctionInfo>();
    OutStreamer->emitRawComment(
      Twine("SQ_PGM_RESOURCES:STACK_SIZE = " + Twine(MFI->CFStackSize)));
  }

  return false;
}
```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `llvm::createR600AsmPrinterPass`, `std::move`, `R600AsmPrinter::R600AsmPrinter`, `R600AsmPrinter::getPassName`, `R600AsmPrinter::EmitProgramInfoR600`, `std::max`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; lowering / 降低
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"R600AsmPrinter.h"`
- `"MCTargetDesc/R600MCTargetDesc.h"`
- `"R600Defines.h"`
- `"R600MachineFunctionInfo.h"`
- `"R600Subtarget.h"`
- `"llvm/BinaryFormat/ELF.h"`
- `"llvm/MC/MCContext.h"`
- `"llvm/MC/MCSectionELF.h"`
- `"llvm/MC/MCStreamer.h"`
- `"llvm/Target/TargetLoweringObjectFile.h"`
