# WebAssemblyMCTargetDesc.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/MCTargetDesc/WebAssemblyMCTargetDesc.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides WebAssembly-specific target descriptions.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/MCTargetDesc/WebAssemblyMCTargetDesc.cpp`，主要负责 WebAssembly 后端的该后端的 MC 层支持。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyMCTargetDesc.cpp - WebAssembly Target Descriptions -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-11

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file provides WebAssembly-specific target descriptions.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 12-18

```cpp
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "MCTargetDesc/WebAssemblyInstPrinter.h"
#include "MCTargetDesc/WebAssemblyMCAsmInfo.h"
#include "MCTargetDesc/WebAssemblyTargetStreamer.h"
#include "TargetInfo/WebAssemblyTargetInfo.h"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 19-26

```cpp
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/ErrorHandling.h"
using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 27-33

```cpp
#define DEBUG_TYPE "wasm-mc-target-desc"

#define GET_INSTRINFO_MC_DESC
#define ENABLE_INSTR_PREDICATE_VERIFIER
#include "WebAssemblyGenInstrInfo.inc"

#define GET_SUBTARGETINFO_MC_DESC
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 34-71

```cpp
#include "WebAssemblyGenSubtargetInfo.inc"

#define GET_REGINFO_MC_DESC
#include "WebAssemblyGenRegisterInfo.inc"

static MCAsmInfo *createMCAsmInfo(const MCRegisterInfo & /*MRI*/,
                                  const Triple &TT,
                                  const MCTargetOptions &Options) {
  return new WebAssemblyMCAsmInfo(TT, Options);
}

static MCInstrInfo *createMCInstrInfo() {
  auto *X = new MCInstrInfo();
  InitWebAssemblyMCInstrInfo(X);
  return X;
}

static MCRegisterInfo *createMCRegisterInfo(const Triple & /*T*/) {
  auto *X = new MCRegisterInfo();
  InitWebAssemblyMCRegisterInfo(X, 0);
  return X;
}

static MCInstPrinter *createMCInstPrinter(const Triple & /*T*/,
                                          unsigned SyntaxVariant,
                                          const MCAsmInfo &MAI,
                                          const MCInstrInfo &MII,
                                          const MCRegisterInfo &MRI) {
  assert(SyntaxVariant == 0 && "WebAssembly only has one syntax variant");
  return new WebAssemblyInstPrinter(MAI, MII, MRI);
}

static MCCodeEmitter *createCodeEmitter(const MCInstrInfo &MCII,
                                        MCContext &Ctx) {
  return createWebAssemblyMCCodeEmitter(MCII, Ctx);
}

static MCAsmBackend *createAsmBackend(const Target & /*T*/,
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 72-109

```cpp
                                      const MCSubtargetInfo &STI,
                                      const MCRegisterInfo & /*MRI*/,
                                      const MCTargetOptions & /*Options*/) {
  return createWebAssemblyAsmBackend(STI.getTargetTriple());
}

static MCSubtargetInfo *createMCSubtargetInfo(const Triple &TT, StringRef CPU,
                                              StringRef FS) {
  if (CPU.empty())
    CPU = "generic";
  return createWebAssemblyMCSubtargetInfoImpl(TT, CPU, /*TuneCPU*/ CPU, FS);
}

static MCTargetStreamer *
createObjectTargetStreamer(MCStreamer &S, const MCSubtargetInfo &STI) {
  return new WebAssemblyTargetWasmStreamer(S);
}

static MCTargetStreamer *
createAsmTargetStreamer(MCStreamer &S, formatted_raw_ostream &OS,
                        MCInstPrinter * /*InstPrint*/) {
  return new WebAssemblyTargetAsmStreamer(S, OS);
}

static MCTargetStreamer *createNullTargetStreamer(MCStreamer &S) {
  return new WebAssemblyTargetNullStreamer(S);
}

// Force static initialization.
extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
LLVMInitializeWebAssemblyTargetMC() {
  for (Target *T :
       {&getTheWebAssemblyTarget32(), &getTheWebAssemblyTarget64()}) {
    // Register the MC asm info.
    RegisterMCAsmInfoFn X(*T, createMCAsmInfo);

    // Register the MC instruction info.
    TargetRegistry::RegisterMCInstrInfo(*T, createMCInstrInfo);
```
- **EN**: Implements helper routine(s) `createWebAssemblyAsmBackend`, `getTargetTriple`, `createMCSubtargetInfo` for this portion of the WebAssembly backend MC layer support for the backend. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 MC 层支持所需的辅助例程 `createWebAssemblyAsmBackend`, `getTargetTriple`, `createMCSubtargetInfo`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 110-134

```cpp

    // Register the MC register info.
    TargetRegistry::RegisterMCRegInfo(*T, createMCRegisterInfo);

    // Register the MCInstPrinter.
    TargetRegistry::RegisterMCInstPrinter(*T, createMCInstPrinter);

    // Register the MC code emitter.
    TargetRegistry::RegisterMCCodeEmitter(*T, createCodeEmitter);

    // Register the ASM Backend.
    TargetRegistry::RegisterMCAsmBackend(*T, createAsmBackend);

    // Register the MC subtarget info.
    TargetRegistry::RegisterMCSubtargetInfo(*T, createMCSubtargetInfo);

    // Register the object target streamer.
    TargetRegistry::RegisterObjectTargetStreamer(*T,
                                                 createObjectTargetStreamer);
    // Register the asm target streamer.
    TargetRegistry::RegisterAsmTargetStreamer(*T, createAsmTargetStreamer);
    // Register the null target streamer.
    TargetRegistry::RegisterNullTargetStreamer(*T, createNullTargetStreamer);
  }
}
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Register the MC register info.". Register assignment and register-class constraints matter here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Register the MC register info.”。 这里重点涉及寄存器分配与寄存器类约束。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- MC layer support for the backend / 该后端的 MC 层支持
- Register modeling / 寄存器建模
- Subtarget features / 子目标特性
- Assembly or MC emission / 汇编或 MC 发射
- Object format integration / 目标文件格式集成
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `MCTargetDesc/WebAssemblyInstPrinter.h`
- `MCTargetDesc/WebAssemblyMCAsmInfo.h`
- `MCTargetDesc/WebAssemblyTargetStreamer.h`
- `TargetInfo/WebAssemblyTargetInfo.h`
- `llvm/MC/MCInstrInfo.h`
- `llvm/MC/MCRegisterInfo.h`
- `llvm/MC/MCSubtargetInfo.h`
- `llvm/MC/TargetRegistry.h`
- `llvm/Support/Compiler.h`
- `llvm/Support/ErrorHandling.h`
- `WebAssemblyGenInstrInfo.inc`
- `WebAssemblyGenSubtargetInfo.inc`
- `WebAssemblyGenRegisterInfo.inc`

### Important Collaborators / 重要协作组件

- LLVM MC layer / LLVM MC 层
- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
