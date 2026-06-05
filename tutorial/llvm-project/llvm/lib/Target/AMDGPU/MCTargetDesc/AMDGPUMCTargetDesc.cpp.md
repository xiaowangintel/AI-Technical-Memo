# AMDGPUMCTargetDesc.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/MCTargetDesc/AMDGPUMCTargetDesc.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUMCTargetDesc for the LLVM MC target description layer. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM MC 目标描述层中的 AMDGPUMCTargetDesc 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: File banner, includes, and setup
```cpp
//===-- AMDGPUMCTargetDesc.cpp - AMDGPU Target Descriptions ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// This file provides AMDGPU specific target descriptions.
//
//===----------------------------------------------------------------------===//

#include "AMDGPUMCTargetDesc.h"
#include "AMDGPUELFStreamer.h"
#include "AMDGPUInstPrinter.h"
#include "AMDGPUMCAsmInfo.h"
#include "AMDGPUTargetStreamer.h"
#include "R600InstPrinter.h"
#include "R600MCTargetDesc.h"
#include "TargetInfo/AMDGPUTargetInfo.h"
#include "llvm/MC/MCAsmBackend.h"
#include "llvm/MC/MCCodeEmitter.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCELFStreamer.h"
#include "llvm/MC/MCInstPrinter.h"
#include "llvm/MC/MCInstrDesc.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Compiler.h"
```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 35-61: Header dependencies and setup
```cpp

using namespace llvm;

#define GET_INSTRINFO_MC_DESC
#define ENABLE_INSTR_PREDICATE_VERIFIER
#include "AMDGPUGenInstrInfo.inc"

#define GET_SUBTARGETINFO_MC_DESC
#include "AMDGPUGenSubtargetInfo.inc"

#define NoSchedModel NoSchedModelR600
#define GET_SUBTARGETINFO_MC_DESC
#include "R600GenSubtargetInfo.inc"
#undef NoSchedModelR600

#define GET_REGINFO_MC_DESC
#include "AMDGPUGenRegisterInfo.inc"

#define GET_REGINFO_MC_DESC
#include "R600GenRegisterInfo.inc"

static MCInstrInfo *createAMDGPUMCInstrInfo() {
  MCInstrInfo *X = new MCInstrInfo();
  InitAMDGPUMCInstrInfo(X);
  return X;
}

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。

### Lines 62-91: Defines createAMDGPUMCRegisterInfo
```cpp
static MCRegisterInfo *createAMDGPUMCRegisterInfo(const Triple &TT) {
  MCRegisterInfo *X = new MCRegisterInfo();
  if (TT.getArch() == Triple::r600)
    InitR600MCRegisterInfo(X, 0);
  else
    InitAMDGPUMCRegisterInfo(X, AMDGPU::PC_REG);
  return X;
}

MCRegisterInfo *llvm::createGCNMCRegisterInfo(AMDGPUDwarfFlavour DwarfFlavour) {
  MCRegisterInfo *X = new MCRegisterInfo();
  InitAMDGPUMCRegisterInfo(X, AMDGPU::PC_REG, DwarfFlavour, DwarfFlavour);
  return X;
}

static MCSubtargetInfo *
createAMDGPUMCSubtargetInfo(const Triple &TT, StringRef CPU, StringRef FS) {
  if (TT.getArch() == Triple::r600)
    return createR600MCSubtargetInfoImpl(TT, CPU, /*TuneCPU*/ CPU, FS);

  MCSubtargetInfo *STI =
      createAMDGPUMCSubtargetInfoImpl(TT, CPU, /*TuneCPU*/ CPU, FS);

  bool IsWave64 = STI->hasFeature(AMDGPU::FeatureWavefrontSize64);
  bool IsWave32 = STI->hasFeature(AMDGPU::FeatureWavefrontSize32);

  // FIXME: We should error for the default target.
  if (STI->getFeatureBits().none())
    STI->ToggleFeature(AMDGPU::FeatureSouthernIslands);

```
**EN:** This section contains concrete logic for createAMDGPUMCRegisterInfo. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `llvm::createGCNMCRegisterInfo`.
**CN:** 本节包含与 createAMDGPUMCRegisterInfo 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`llvm::createGCNMCRegisterInfo`。

### Lines 92-117: Conditional logic and checks
```cpp
  if (!IsWave64 && !IsWave32) {
    // If there is no default wave size it must be a generation before gfx10,
    // these have FeatureWavefrontSize64 in their definition already. For gfx10+
    // set wave32 as a default.
    STI->ToggleFeature(AMDGPU::isGFX10Plus(*STI)
                           ? AMDGPU::FeatureWavefrontSize32
                           : AMDGPU::FeatureWavefrontSize64);
  } else if (IsWave64 && IsWave32) {
    // The wave size is mutually exclusive. If both somehow end up set, wave32
    // wins if supported.
    STI->ToggleFeature(AMDGPU::supportsWave32(*STI)
                           ? AMDGPU::FeatureWavefrontSize64
                           : AMDGPU::FeatureWavefrontSize32);

    // If both wavesizes were manually requested, hack in a feature to permit
    // assembling modules with mixed wavesizes.
    STI->ToggleFeature(AMDGPU::FeatureAssemblerPermissiveWavesize);
  }

  assert((STI->hasFeature(AMDGPU::FeatureWavefrontSize64) !=
          STI->hasFeature(AMDGPU::FeatureWavefrontSize32)) &&
         "wavesize features are mutually exclusive");

  return STI;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::isGFX10Plus`, `AMDGPU::supportsWave32`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::isGFX10Plus`, `AMDGPU::supportsWave32`。

### Lines 118-151: Defines createAMDGPUMCInstPrinter
```cpp
static MCInstPrinter *createAMDGPUMCInstPrinter(const Triple &T,
                                                unsigned SyntaxVariant,
                                                const MCAsmInfo &MAI,
                                                const MCInstrInfo &MII,
                                                const MCRegisterInfo &MRI) {
  if (T.getArch() == Triple::r600)
    return new R600InstPrinter(MAI, MII, MRI);
  return new AMDGPUInstPrinter(MAI, MII, MRI);
}

static MCTargetStreamer *
createAMDGPUAsmTargetStreamer(MCStreamer &S, formatted_raw_ostream &OS,
                              MCInstPrinter *InstPrint) {
  return new AMDGPUTargetAsmStreamer(S, OS);
}

static MCTargetStreamer * createAMDGPUObjectTargetStreamer(
                                                   MCStreamer &S,
                                                   const MCSubtargetInfo &STI) {
  return new AMDGPUTargetELFStreamer(S, STI);
}

static MCTargetStreamer *createAMDGPUNullTargetStreamer(MCStreamer &S) {
  return new AMDGPUTargetStreamer(S);
}

static MCStreamer *createMCStreamer(const Triple &T, MCContext &Context,
                                    std::unique_ptr<MCAsmBackend> &&MAB,
                                    std::unique_ptr<MCObjectWriter> &&OW,
                                    std::unique_ptr<MCCodeEmitter> &&Emitter) {
  return createAMDGPUELFStreamer(T, Context, std::move(MAB), std::move(OW),
                                 std::move(Emitter));
}

```
**EN:** This section contains concrete logic for createAMDGPUMCInstPrinter. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::move`.
**CN:** 本节包含与 createAMDGPUMCInstPrinter 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::move`。

### Lines 152-185: Implements AMDGPUMCInstrAnalysis::evaluateBranch
```cpp
namespace llvm {
namespace AMDGPU {

bool AMDGPUMCInstrAnalysis::evaluateBranch(const MCInst &Inst, uint64_t Addr,
                                           uint64_t Size,
                                           uint64_t &Target) const {
  if (Inst.getNumOperands() == 0 || !Inst.getOperand(0).isImm() ||
      Info->get(Inst.getOpcode()).operands()[0].OperandType !=
          MCOI::OPERAND_PCREL)
    return false;

  int64_t Imm = Inst.getOperand(0).getImm();
  // Our branches take a simm16.
  Target = SignExtend64<16>(Imm) * 4 + Addr + Size;
  return true;
}

void AMDGPUMCInstrAnalysis::updateState(const MCInst &Inst,
                                        const MCSubtargetInfo *STI,
                                        uint64_t Addr) {
  if (Inst.getOpcode() == AMDGPU::S_SET_VGPR_MSB_gfx12)
    VgprMSBs = Inst.getOperand(0).getImm() & 0xff;
  else if (Inst.getOpcode() == AMDGPU::S_SETREG_IMM32_B32_gfx12 &&
           STI->hasFeature(AMDGPU::Feature1024AddressableVGPRs)) {
    VgprMSBs = AMDGPU::convertSetRegImmToVgprMSBs(
                   Inst, STI->hasFeature(AMDGPU::FeatureSetregVGPRMSBFixup))
                   .value_or(0);
  } else if (isTerminator(Inst))
    VgprMSBs = 0;
}

} // end namespace AMDGPU
} // end namespace llvm

```
**EN:** This section contains concrete logic for AMDGPUMCInstrAnalysis::evaluateBranch. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUMCInstrAnalysis::evaluateBranch`, `AMDGPUMCInstrAnalysis::updateState`, `AMDGPU::convertSetRegImmToVgprMSBs`.
**CN:** 本节包含与 AMDGPUMCInstrAnalysis::evaluateBranch 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUMCInstrAnalysis::evaluateBranch`, `AMDGPUMCInstrAnalysis::updateState`, `AMDGPU::convertSetRegImmToVgprMSBs`。

### Lines 186-216: Defines createAMDGPUMCInstrAnalysis
```cpp
static MCInstrAnalysis *createAMDGPUMCInstrAnalysis(const MCInstrInfo *Info) {
  return new AMDGPU::AMDGPUMCInstrAnalysis(Info);
}

extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
LLVMInitializeAMDGPUTargetMC() {

  TargetRegistry::RegisterMCInstrInfo(getTheGCNTarget(), createAMDGPUMCInstrInfo);
  TargetRegistry::RegisterMCInstrInfo(getTheR600Target(),
                                      createR600MCInstrInfo);
  for (Target *T : {&getTheR600Target(), &getTheGCNTarget()}) {
    RegisterMCAsmInfo<AMDGPUMCAsmInfo> X(*T);

    TargetRegistry::RegisterMCRegInfo(*T, createAMDGPUMCRegisterInfo);
    TargetRegistry::RegisterMCSubtargetInfo(*T, createAMDGPUMCSubtargetInfo);
    TargetRegistry::RegisterMCInstPrinter(*T, createAMDGPUMCInstPrinter);
    TargetRegistry::RegisterMCInstrAnalysis(*T, createAMDGPUMCInstrAnalysis);
    TargetRegistry::RegisterMCAsmBackend(*T, createAMDGPUAsmBackend);
    TargetRegistry::RegisterELFStreamer(*T, createMCStreamer);
  }

  // R600 specific registration
  TargetRegistry::RegisterMCCodeEmitter(getTheR600Target(),
                                        createR600MCCodeEmitter);
  TargetRegistry::RegisterObjectTargetStreamer(
      getTheR600Target(), createAMDGPUObjectTargetStreamer);

  // GCN specific registration
  TargetRegistry::RegisterMCCodeEmitter(getTheGCNTarget(),
                                        createAMDGPUMCCodeEmitter);

```
**EN:** This section contains concrete logic for createAMDGPUMCInstrAnalysis. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU::AMDGPUMCInstrAnalysis`, `TargetRegistry::RegisterMCInstrInfo`, `TargetRegistry::RegisterMCRegInfo`.
**CN:** 本节包含与 createAMDGPUMCInstrAnalysis 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU::AMDGPUMCInstrAnalysis`, `TargetRegistry::RegisterMCInstrInfo`, `TargetRegistry::RegisterMCRegInfo`。

### Lines 217-223: Implements TargetRegistry::RegisterAsmTargetStreamer
```cpp
  TargetRegistry::RegisterAsmTargetStreamer(getTheGCNTarget(),
                                            createAMDGPUAsmTargetStreamer);
  TargetRegistry::RegisterObjectTargetStreamer(
      getTheGCNTarget(), createAMDGPUObjectTargetStreamer);
  TargetRegistry::RegisterNullTargetStreamer(getTheGCNTarget(),
                                             createAMDGPUNullTargetStreamer);
}
```
**EN:** This section contains concrete logic for TargetRegistry::RegisterAsmTargetStreamer. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `TargetRegistry::RegisterAsmTargetStreamer`, `TargetRegistry::RegisterObjectTargetStreamer`, `TargetRegistry::RegisterNullTargetStreamer`.
**CN:** 本节包含与 TargetRegistry::RegisterAsmTargetStreamer 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`TargetRegistry::RegisterAsmTargetStreamer`, `TargetRegistry::RegisterObjectTargetStreamer`, `TargetRegistry::RegisterNullTargetStreamer`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `llvm::createGCNMCRegisterInfo`, `AMDGPU::isGFX10Plus`, `AMDGPU::supportsWave32`, `std::move`, `AMDGPUMCInstrAnalysis::evaluateBranch`, `AMDGPUMCInstrAnalysis::updateState`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; scheduling / 调度
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUMCTargetDesc.h"`
- `"AMDGPUELFStreamer.h"`
- `"AMDGPUInstPrinter.h"`
- `"AMDGPUMCAsmInfo.h"`
- `"AMDGPUTargetStreamer.h"`
- `"R600InstPrinter.h"`
- `"R600MCTargetDesc.h"`
- `"TargetInfo/AMDGPUTargetInfo.h"`
- `"llvm/MC/MCAsmBackend.h"`
- `"llvm/MC/MCCodeEmitter.h"`
- `"llvm/MC/MCContext.h"`
- `"llvm/MC/MCELFStreamer.h"`
- `"llvm/MC/MCInstPrinter.h"`
- `"llvm/MC/MCInstrDesc.h"`
- `"llvm/MC/MCInstrInfo.h"`
- `"llvm/MC/MCObjectWriter.h"`
- `"llvm/MC/MCRegisterInfo.h"`
- `"llvm/MC/MCStreamer.h"`
- `"llvm/MC/MCSubtargetInfo.h"`
- `"llvm/MC/TargetRegistry.h"`
- ... and 6 more direct dependencies / 以及另外 6 个直接依赖
