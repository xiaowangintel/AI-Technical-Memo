# SparcMCTargetDesc.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/MCTargetDesc/SparcMCTargetDesc.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Provides MC-layer target descriptions such as fixups, asm info, code emission, object writing, and target registration.
  - **CN**: 提供 MC 层的目标描述，例如 fixup、汇编信息、编码发射、目标文件写出以及目标注册。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- SparcMCTargetDesc.cpp - Sparc Target Descriptions -----------------===//
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
// This file provides Sparc specific target descriptions.
//
//===----------------------------------------------------------------------===//

#include "SparcMCTargetDesc.h"
#include "SparcInstPrinter.h"
#include "SparcMCAsmInfo.h"
#include "SparcTargetStreamer.h"
#include "TargetInfo/SparcTargetInfo.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/TargetRegistry.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `SparcMCTargetDesc.h`, `SparcInstPrinter.h`, `SparcMCAsmInfo.h`, `SparcTargetStreamer.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `SparcMCTargetDesc.h`, `SparcInstPrinter.h`, `SparcMCAsmInfo.h`, `SparcTargetStreamer.h`。

### Lines 22-30
```cpp
#include "llvm/Support/Compiler.h"
#include "llvm/Support/ErrorHandling.h"

namespace llvm {
namespace SparcASITag {
#define GET_ASITagsList_IMPL
#include "SparcGenSearchableTables.inc"
} // end namespace SparcASITag

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/Support/Compiler.h`, `llvm/Support/ErrorHandling.h`, `SparcGenSearchableTables.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/Support/Compiler.h`, `llvm/Support/ErrorHandling.h`, `SparcGenSearchableTables.inc`。

### Lines 31-38
```cpp
namespace SparcPrefetchTag {
#define GET_PrefetchTagsList_IMPL
#include "SparcGenSearchableTables.inc"
} // end namespace SparcPrefetchTag
} // end namespace llvm

using namespace llvm;

```
- **EN**: Pulls in the headers needed for this implementation, including `SparcGenSearchableTables.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `SparcGenSearchableTables.inc`。

### Lines 39-45
```cpp
#define GET_INSTRINFO_MC_DESC
#define ENABLE_INSTR_PREDICATE_VERIFIER
#include "SparcGenInstrInfo.inc"

#define GET_SUBTARGETINFO_MC_DESC
#include "SparcGenSubtargetInfo.inc"

```
- **EN**: Pulls in the headers needed for this implementation, including `SparcGenInstrInfo.inc`, `SparcGenSubtargetInfo.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `SparcGenInstrInfo.inc`, `SparcGenSubtargetInfo.inc`。

### Lines 46-58
```cpp
#define GET_REGINFO_MC_DESC
#include "SparcGenRegisterInfo.inc"

static MCAsmInfo *createSparcMCAsmInfo(const MCRegisterInfo &MRI,
                                       const Triple &TT,
                                       const MCTargetOptions &Options) {
  MCAsmInfo *MAI = new SparcELFMCAsmInfo(TT, Options);
  unsigned Reg = MRI.getDwarfRegNum(SP::O6, true);
  MCCFIInstruction Inst = MCCFIInstruction::cfiDefCfa(nullptr, Reg, 0);
  MAI->addInitialFrameState(Inst);
  return MAI;
}

```
- **EN**: Pulls in the headers needed for this implementation, including `SparcGenRegisterInfo.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `SparcGenRegisterInfo.inc`。

### Lines 59-68
```cpp
static MCAsmInfo *createSparcV9MCAsmInfo(const MCRegisterInfo &MRI,
                                         const Triple &TT,
                                         const MCTargetOptions &Options) {
  MCAsmInfo *MAI = new SparcELFMCAsmInfo(TT, Options);
  unsigned Reg = MRI.getDwarfRegNum(SP::O6, true);
  MCCFIInstruction Inst = MCCFIInstruction::cfiDefCfa(nullptr, Reg, 2047);
  MAI->addInitialFrameState(Inst);
  return MAI;
}

```
- **EN**: Implements logic around `createSparcV9MCAsmInfo`, `SparcELFMCAsmInfo`, `getDwarfRegNum`, `cfiDefCfa`, ...; this block returns target-specific results.
- **CN**: 围绕 `createSparcV9MCAsmInfo`, `SparcELFMCAsmInfo`, `getDwarfRegNum`, `cfiDefCfa`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 69-80
```cpp
static MCInstrInfo *createSparcMCInstrInfo() {
  MCInstrInfo *X = new MCInstrInfo();
  InitSparcMCInstrInfo(X);
  return X;
}

static MCRegisterInfo *createSparcMCRegisterInfo(const Triple &TT) {
  MCRegisterInfo *X = new MCRegisterInfo();
  InitSparcMCRegisterInfo(X, SP::O7);
  return X;
}

```
- **EN**: Implements logic around `createSparcMCInstrInfo`, `MCInstrInfo`, `InitSparcMCInstrInfo`, `createSparcMCRegisterInfo`, ...; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `createSparcMCInstrInfo`, `MCInstrInfo`, `InitSparcMCInstrInfo`, `createSparcMCRegisterInfo`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 81-92
```cpp
static MCSubtargetInfo *
createSparcMCSubtargetInfo(const Triple &TT, StringRef CPU, StringRef FS) {
  if (CPU.empty())
    CPU = TT.getArch() == Triple::sparcv9 ? "v9" : "v8";

  MCSubtargetInfo *STI =
      createSparcMCSubtargetInfoImpl(TT, CPU, /*TuneCPU=*/CPU, FS);
  if (TT.isSPARC64() && !STI->hasFeature(Sparc::Feature64Bit)) {
    FeatureBitset Features = STI->getFeatureBits();
    STI->setFeatureBits(Features.set(Sparc::Feature64Bit));
  }

```
- **EN**: Implements logic around `createSparcMCSubtargetInfo`, `getArch`, `createSparcMCSubtargetInfoImpl`, `getFeatureBits`, ...; this block applies conditional target rules.
- **CN**: 围绕 `createSparcMCSubtargetInfo`, `getArch`, `createSparcMCSubtargetInfoImpl`, `getFeatureBits`, ... 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 93-100
```cpp
  return STI;
}

static MCTargetStreamer *
createObjectTargetStreamer(MCStreamer &S, const MCSubtargetInfo &STI) {
  return new SparcTargetELFStreamer(S, STI);
}

```
- **EN**: Implements logic around `createObjectTargetStreamer`, `SparcTargetELFStreamer`; this block returns target-specific results.
- **CN**: 围绕 `createObjectTargetStreamer`, `SparcTargetELFStreamer` 实现具体逻辑；这一段返回目标相关结果。

### Lines 101-110
```cpp
static MCTargetStreamer *createTargetAsmStreamer(MCStreamer &S,
                                                 formatted_raw_ostream &OS,
                                                 MCInstPrinter *InstPrint) {
  return new SparcTargetAsmStreamer(S, OS);
}

static MCTargetStreamer *createNullTargetStreamer(MCStreamer &S) {
  return new SparcTargetStreamer(S);
}

```
- **EN**: Implements logic around `createTargetAsmStreamer`, `SparcTargetAsmStreamer`, `createNullTargetStreamer`, `SparcTargetStreamer`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `createTargetAsmStreamer`, `SparcTargetAsmStreamer`, `createNullTargetStreamer`, `SparcTargetStreamer` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 111-118
```cpp
static MCInstPrinter *createSparcMCInstPrinter(const Triple &T,
                                               unsigned SyntaxVariant,
                                               const MCAsmInfo &MAI,
                                               const MCInstrInfo &MII,
                                               const MCRegisterInfo &MRI) {
  return new SparcInstPrinter(MAI, MII, MRI);
}

```
- **EN**: Implements logic around `createSparcMCInstPrinter`, `SparcInstPrinter`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `createSparcMCInstPrinter`, `SparcInstPrinter` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 119-125
```cpp
extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
LLVMInitializeSparcTargetMC() {
  // Register the MC asm info.
  RegisterMCAsmInfoFn X(getTheSparcTarget(), createSparcMCAsmInfo);
  RegisterMCAsmInfoFn Y(getTheSparcV9Target(), createSparcV9MCAsmInfo);
  RegisterMCAsmInfoFn Z(getTheSparcelTarget(), createSparcMCAsmInfo);

```
- **EN**: Implements logic around `X`, `Y`, `Z`.
- **CN**: 围绕 `X`, `Y`, `Z` 实现具体逻辑。

### Lines 126-133
```cpp
  for (Target *T :
       {&getTheSparcTarget(), &getTheSparcV9Target(), &getTheSparcelTarget()}) {
    // Register the MC instruction info.
    TargetRegistry::RegisterMCInstrInfo(*T, createSparcMCInstrInfo);

    // Register the MC register info.
    TargetRegistry::RegisterMCRegInfo(*T, createSparcMCRegisterInfo);

```
- **EN**: Implements logic around `getTheSparcTarget`, `RegisterMCInstrInfo`, `RegisterMCRegInfo`; this block works at the MC layer.
- **CN**: 围绕 `getTheSparcTarget`, `RegisterMCInstrInfo`, `RegisterMCRegInfo` 实现具体逻辑；这一段工作在 MC 层。

### Lines 134-142
```cpp
    // Register the MC subtarget info.
    TargetRegistry::RegisterMCSubtargetInfo(*T, createSparcMCSubtargetInfo);

    // Register the MC Code Emitter.
    TargetRegistry::RegisterMCCodeEmitter(*T, createSparcMCCodeEmitter);

    // Register the asm backend.
    TargetRegistry::RegisterMCAsmBackend(*T, createSparcAsmBackend);

```
- **EN**: Implements logic around `RegisterMCSubtargetInfo`, `RegisterMCCodeEmitter`, `RegisterMCAsmBackend`.
- **CN**: 围绕 `RegisterMCSubtargetInfo`, `RegisterMCCodeEmitter`, `RegisterMCAsmBackend` 实现具体逻辑。

### Lines 143-149
```cpp
    // Register the object target streamer.
    TargetRegistry::RegisterObjectTargetStreamer(*T,
                                                 createObjectTargetStreamer);

    // Register the asm streamer.
    TargetRegistry::RegisterAsmTargetStreamer(*T, createTargetAsmStreamer);

```
- **EN**: Implements logic around `RegisterObjectTargetStreamer`, `RegisterAsmTargetStreamer`.
- **CN**: 围绕 `RegisterObjectTargetStreamer`, `RegisterAsmTargetStreamer` 实现具体逻辑。

### Lines 150-156
```cpp
    // Register the null streamer.
    TargetRegistry::RegisterNullTargetStreamer(*T, createNullTargetStreamer);

    // Register the MCInstPrinter
    TargetRegistry::RegisterMCInstPrinter(*T, createSparcMCInstPrinter);
  }
}
```
- **EN**: Implements logic around `RegisterNullTargetStreamer`, `RegisterMCInstPrinter`; this block works at the MC layer.
- **CN**: 围绕 `RegisterNullTargetStreamer`, `RegisterMCInstPrinter` 实现具体逻辑；这一段工作在 MC 层。

## Key Concepts / 关键概念

- **MC layer / MC 层**:
  - **EN**: Connects the backend to LLVM's MC infrastructure
  - **CN**: 把后端接入 LLVM 的 MC 基础设施

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `SparcMCTargetDesc.h`, `SparcInstPrinter.h`, `SparcMCAsmInfo.h`, `SparcTargetStreamer.h`, `TargetInfo/SparcTargetInfo.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCRegisterInfo.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/TargetRegistry.h`, `llvm/Support/Compiler.h`, `llvm/Support/ErrorHandling.h`, `SparcGenSearchableTables.inc` ... (+3 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support
- **Generated macros / 生成宏**: `GET_ASIT`, `GET_INSTRINFO_MC_DESC`, `GET_P`, `GET_REGINFO_MC_DESC`, `GET_SUBTARGETINFO_MC_DESC`
