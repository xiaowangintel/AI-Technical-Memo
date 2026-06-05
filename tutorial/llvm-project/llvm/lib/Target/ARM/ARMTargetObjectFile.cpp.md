# ARMTargetObjectFile.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMTargetObjectFile.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `ARMTargetObjectFile` for the ARM backend, focusing on object-file sections, relocations, and data placement.
- 用途 (CN): 实现 ARM 后端中的 `ARMTargetObjectFile`，重点处理目标文件节区、重定位与数据放置。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- llvm/Target/ARMTargetObjectFile.cpp - ARM Object Info Impl --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 9-22
```cpp
#include "ARMTargetObjectFile.h"
#include "ARMSubtarget.h"
#include "ARMTargetMachine.h"
#include "MCTargetDesc/ARMMCAsmInfo.h"
#include "llvm/BinaryFormat/Dwarf.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCSectionELF.h"
#include "llvm/MC/MCValue.h"
#include "llvm/MC/SectionKind.h"
#include "llvm/Target/TargetMachine.h"
#include <cassert>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 24-25
```cpp
using namespace llvm;
using namespace dwarf;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 27-29
```cpp
//===----------------------------------------------------------------------===//
//                               ELF Target
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 31-34
```cpp
ARMElfTargetObjectFile::ARMElfTargetObjectFile() {
  PLTRelativeSpecifier = ARM::S_PREL31;
  SupportIndirectSymViaGOTPCRel = true;
}
```
- EN: Implements `ARMElfTargetObjectFile::ARMElfTargetObjectFile`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMElfTargetObjectFile::ARMElfTargetObjectFile`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 36-41
```cpp
void ARMElfTargetObjectFile::Initialize(MCContext &Ctx,
                                        const TargetMachine &TM) {
  const ARMBaseTargetMachine &ARM_TM = static_cast<const ARMBaseTargetMachine &>(TM);
  bool isAAPCS_ABI = ARM_TM.TargetABI == ARM::ARMABI::ARM_ABI_AAPCS;
  bool genExecuteOnly =
      ARM_TM.getMCSubtargetInfo().hasFeature(ARM::FeatureExecuteOnly);
```
- EN: Implements `ARMElfTargetObjectFile::Initialize`, a target-specific routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMElfTargetObjectFile::Initialize`，它是一个围绕目标机器策略展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 43-44
```cpp
  TargetLoweringObjectFileELF::Initialize(Ctx, TM);
  InitializeELF(isAAPCS_ABI);
```
- EN: Declares `TargetLoweringObjectFileELF::Initialize`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `TargetLoweringObjectFileELF::Initialize`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 46-48
```cpp
  if (isAAPCS_ABI) {
    LSDASection = nullptr;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 50-61
```cpp
  // Make code section unreadable when in execute-only mode
  if (genExecuteOnly) {
    unsigned Type = ELF::SHT_PROGBITS;
    unsigned Flags =
        ELF::SHF_EXECINSTR | ELF::SHF_ALLOC | ELF::SHF_ARM_PURECODE;
    // Since we cannot modify flags for an existing section, we create a new
    // section with the right flags, and use 0 as the unique ID for
    // execute-only text
    TextSection =
        Ctx.getELFSection(".text", Type, Flags, 0, "", false, 0U, nullptr);
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 63-63
```cpp
MCRegister ARMElfTargetObjectFile::getStaticBase() const { return ARM::R9; }
```
- EN: Implements `ARMElfTargetObjectFile::getStaticBase`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMElfTargetObjectFile::getStaticBase`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 65-73
```cpp
const MCExpr *ARMElfTargetObjectFile::getIndirectSymViaGOTPCRel(
    const GlobalValue *GV, const MCSymbol *Sym, const MCValue &MV,
    int64_t Offset, MachineModuleInfo *MMI, MCStreamer &Streamer) const {
  int64_t FinalOffset = Offset + MV.getConstant();
  const MCExpr *Res =
      MCSymbolRefExpr::create(Sym, ARM::S_GOT_PREL, getContext());
  const MCExpr *Off = MCConstantExpr::create(FinalOffset, getContext());
  return MCBinaryExpr::createAdd(Res, Off, getContext());
}
```
- EN: Implements `ARMElfTargetObjectFile::getIndirectSymViaGOTPCRel`, a query/helper routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMElfTargetObjectFile::getIndirectSymViaGOTPCRel`，它是一个围绕MC streamer 交互展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 75-78
```cpp
const MCExpr *ARMElfTargetObjectFile::
getIndirectSymViaRWPI(const MCSymbol *Sym) const {
  return MCSymbolRefExpr::create(Sym, ARM::S_SBREL, getContext());
}
```
- EN: Implements `MCSymbolRefExpr::create`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MCSymbolRefExpr::create`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 80-85
```cpp
const MCExpr *ARMElfTargetObjectFile::getTTypeGlobalReference(
    const GlobalValue *GV, unsigned Encoding, const TargetMachine &TM,
    MachineModuleInfo *MMI, MCStreamer &Streamer) const {
  if (TM.getMCAsmInfo().getExceptionHandlingType() != ExceptionHandling::ARM)
    return TargetLoweringObjectFileELF::getTTypeGlobalReference(
        GV, Encoding, TM, MMI, Streamer);
```
- EN: Implements `ARMElfTargetObjectFile::getTTypeGlobalReference`, a query/helper routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMElfTargetObjectFile::getTTypeGlobalReference`，它是一个围绕MC streamer 交互展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 87-87
```cpp
  assert(Encoding == DW_EH_PE_absptr && "Can handle absptr encoding only");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 89-91
```cpp
  return MCSymbolRefExpr::create(TM.getSymbol(GV), ARM::S_TARGET2,
                                 getContext());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 93-96
```cpp
const MCExpr *ARMElfTargetObjectFile::
getDebugThreadLocalSymbol(const MCSymbol *Sym) const {
  return MCSymbolRefExpr::create(Sym, ARM::S_TLSLDO, getContext());
}
```
- EN: Implements `MCSymbolRefExpr::create`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MCSymbolRefExpr::create`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 98-104
```cpp
static bool isExecuteOnlyFunction(const GlobalObject *GO, SectionKind SK,
                                  const TargetMachine &TM) {
  if (const Function *F = dyn_cast<Function>(GO))
    if (TM.getSubtarget<ARMSubtarget>(*F).genExecuteOnly() && SK.isText())
      return true;
  return false;
}
```
- EN: Implements `isExecuteOnlyFunction`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isExecuteOnlyFunction`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 106-110
```cpp
MCSection *ARMElfTargetObjectFile::getExplicitSectionGlobal(
    const GlobalObject *GO, SectionKind SK, const TargetMachine &TM) const {
  // Set execute-only access for the explicit section
  if (isExecuteOnlyFunction(GO, SK, TM))
    SK = SectionKind::getExecuteOnly();
```
- EN: Implements `ARMElfTargetObjectFile::getExplicitSectionGlobal`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMElfTargetObjectFile::getExplicitSectionGlobal`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 112-113
```cpp
  return TargetLoweringObjectFileELF::getExplicitSectionGlobal(GO, SK, TM);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 115-119
```cpp
MCSection *ARMElfTargetObjectFile::SelectSectionForGlobal(
    const GlobalObject *GO, SectionKind SK, const TargetMachine &TM) const {
  // Place the global in the execute-only text section
  if (isExecuteOnlyFunction(GO, SK, TM))
    SK = SectionKind::getExecuteOnly();
```
- EN: Implements `ARMElfTargetObjectFile::SelectSectionForGlobal`, a target-specific routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMElfTargetObjectFile::SelectSectionForGlobal`，它是一个围绕目标机器策略展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 121-122
```cpp
  return TargetLoweringObjectFileELF::SelectSectionForGlobal(GO, SK, TM);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

## Key Concepts / 关键概念

- EN: Primary role: object-file sections, relocations, and data placement.
  - CN: 核心职责：目标文件节区、重定位与数据放置。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARMTargetObjectFile.h`, `ARMSubtarget.h`, `ARMTargetMachine.h`, `MCTargetDesc/ARMMCAsmInfo.h`.
  - CN: 后端本地头文件：`ARMTargetObjectFile.h`, `ARMSubtarget.h`, `ARMTargetMachine.h`, `MCTargetDesc/ARMMCAsmInfo.h`。
- EN: LLVM infrastructure headers: `llvm/BinaryFormat/Dwarf.h`, `llvm/BinaryFormat/ELF.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCSectionELF.h`, `llvm/MC/MCValue.h`, `llvm/MC/SectionKind.h` ... (+1 more).
  - CN: LLVM 基础设施头文件：`llvm/BinaryFormat/Dwarf.h`, `llvm/BinaryFormat/ELF.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCSectionELF.h`, `llvm/MC/MCValue.h`, `llvm/MC/SectionKind.h` ... (+1 more)。
- EN: Standard/system headers: `cassert`.
  - CN: 标准库/系统头文件：`cassert`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
