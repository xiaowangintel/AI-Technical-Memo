# ARMTargetStreamer.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/MCTargetDesc/ARMTargetStreamer.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements the ARMTargetStreamer class.
- 用途 (CN): 实现 ARM 后端中的 `ARMTargetStreamer`，重点处理MC 层目标描述与编码支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- ARMTargetStreamer.cpp - ARMTargetStreamer class --*- C++ -*---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the ARMTargetStreamer class.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-20
```cpp
#include "MCTargetDesc/ARMMCTargetDesc.h"
#include "llvm/MC/ConstantPools.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/Support/ARMBuildAttributes.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 22-22
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 24-26
```cpp
//
// ARMTargetStreamer Implementation
//
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 28-29
```cpp
ARMTargetStreamer::ARMTargetStreamer(MCStreamer &S)
    : MCTargetStreamer(S), ConstantPools(new AssemblerConstantPools()) {}
```
- EN: Implements `ARMTargetStreamer::ARMTargetStreamer`, a target-specific routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTargetStreamer::ARMTargetStreamer`，它是一个围绕MC streamer 交互展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 31-31
```cpp
ARMTargetStreamer::~ARMTargetStreamer() = default;
```
- EN: Declares `ARMTargetStreamer::~ARMTargetStreamer`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ARMTargetStreamer::~ARMTargetStreamer`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 33-37
```cpp
// The constant pool handling is shared by all ARMTargetStreamer
// implementations.
const MCExpr *ARMTargetStreamer::addConstantPoolEntry(const MCExpr *Expr, SMLoc Loc) {
  return ConstantPools->addEntry(Streamer, Expr, 4, Loc);
}
```
- EN: Implements `ARMTargetStreamer::addConstantPoolEntry`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTargetStreamer::addConstantPoolEntry`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 39-42
```cpp
void ARMTargetStreamer::emitCurrentConstantPool() {
  ConstantPools->emitForCurrentSection(Streamer);
  ConstantPools->clearCacheForCurrentSection(Streamer);
}
```
- EN: Implements `ARMTargetStreamer::emitCurrentConstantPool`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTargetStreamer::emitCurrentConstantPool`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 44-47
```cpp
// finish() - write out any non-empty assembler constant pools.
void ARMTargetStreamer::emitConstantPools() {
  ConstantPools->emitAll(Streamer);
}
```
- EN: Implements `ARMTargetStreamer::emitConstantPools`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTargetStreamer::emitConstantPools`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 49-50
```cpp
// reset() - Reset any state
void ARMTargetStreamer::reset() {}
```
- EN: Implements `ARMTargetStreamer::reset`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTargetStreamer::reset`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 52-55
```cpp
void ARMTargetStreamer::emitInst(uint32_t Inst, char Suffix) {
  unsigned Size;
  char Buffer[4];
  const bool LittleEndian = getContext().getAsmInfo().isLittleEndian();
```
- EN: Implements `ARMTargetStreamer::emitInst`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTargetStreamer::emitInst`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 57-59
```cpp
  switch (Suffix) {
  case '\0':
    Size = 4;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 61-64
```cpp
    for (unsigned II = 0, IE = Size; II != IE; II++) {
      const unsigned I = LittleEndian ? (Size - II - 1) : II;
      Buffer[Size - II - 1] = uint8_t(Inst >> I * CHAR_BIT);
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 66-69
```cpp
    break;
  case 'n':
  case 'w':
    Size = (Suffix == 'n' ? 2 : 4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 71-78
```cpp
    // Thumb wide instructions are emitted as a pair of 16-bit words of the
    // appropriate endianness.
    for (unsigned II = 0, IE = Size; II != IE; II = II + 2) {
      const unsigned I0 = LittleEndian ? II + 0 : II + 1;
      const unsigned I1 = LittleEndian ? II + 1 : II + 0;
      Buffer[Size - II - 2] = uint8_t(Inst >> I0 * CHAR_BIT);
      Buffer[Size - II - 1] = uint8_t(Inst >> I1 * CHAR_BIT);
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 80-85
```cpp
    break;
  default:
    llvm_unreachable("Invalid Suffix");
  }
  getStreamer().emitBytes(StringRef(Buffer, Size));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 87-101
```cpp
// The remaining callbacks should be handled separately by each
// streamer.
void ARMTargetStreamer::emitFnStart() {}
void ARMTargetStreamer::emitFnEnd() {}
void ARMTargetStreamer::emitCantUnwind() {}
void ARMTargetStreamer::emitPersonality(const MCSymbol *Personality) {}
void ARMTargetStreamer::emitPersonalityIndex(unsigned Index) {}
void ARMTargetStreamer::emitHandlerData() {}
void ARMTargetStreamer::emitSetFP(MCRegister FpReg, MCRegister SpReg,
                                  int64_t Offset) {}
void ARMTargetStreamer::emitMovSP(MCRegister Reg, int64_t Offset) {}
void ARMTargetStreamer::emitPad(int64_t Offset) {}
void ARMTargetStreamer::emitRegSave(const SmallVectorImpl<MCRegister> &RegList,
                                    bool isVector) {}
void ARMTargetStreamer::emitUnwindRaw(int64_t StackOffset,
```
- EN: Implements `ARMTargetStreamer::emitFnStart`, a emission/printing routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTargetStreamer::emitFnStart`，它是一个围绕寄存器管理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 102-116
```cpp
                                      const SmallVectorImpl<uint8_t> &Opcodes) {
}
void ARMTargetStreamer::switchVendor(StringRef Vendor) {}
void ARMTargetStreamer::emitAttribute(unsigned Attribute, unsigned Value) {}
void ARMTargetStreamer::emitTextAttribute(unsigned Attribute,
                                          StringRef String) {}
void ARMTargetStreamer::emitIntTextAttribute(unsigned Attribute,
                                             unsigned IntValue,
                                             StringRef StringValue) {}
void ARMTargetStreamer::emitArch(ARM::ArchKind Arch) {}
void ARMTargetStreamer::emitArchExtension(uint64_t ArchExt) {}
void ARMTargetStreamer::emitObjectArch(ARM::ArchKind Arch) {}
void ARMTargetStreamer::emitFPU(ARM::FPUKind FPU) {}
void ARMTargetStreamer::finishAttributeSection() {}
void ARMTargetStreamer::annotateTLSDescriptorSequence(
```
- EN: Implements `ARMTargetStreamer::switchVendor`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTargetStreamer::switchVendor`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 117-122
```cpp
    const MCSymbolRefExpr *SRE) {}
void ARMTargetStreamer::emitSyntaxUnified() {}
void ARMTargetStreamer::emitCode16() {}
void ARMTargetStreamer::emitCode32() {}
void ARMTargetStreamer::emitThumbFunc(MCSymbol *Symbol) {}
void ARMTargetStreamer::emitThumbSet(MCSymbol *Symbol, const MCExpr *Value) {}
```
- EN: Implements `ARMTargetStreamer::emitSyntaxUnified`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTargetStreamer::emitSyntaxUnified`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 124-133
```cpp
void ARMTargetStreamer::emitARMWinCFIAllocStack(unsigned Size, bool Wide) {}
void ARMTargetStreamer::emitARMWinCFISaveRegMask(unsigned Mask, bool Wide) {}
void ARMTargetStreamer::emitARMWinCFISaveSP(unsigned Reg) {}
void ARMTargetStreamer::emitARMWinCFISaveFRegs(unsigned First, unsigned Last) {}
void ARMTargetStreamer::emitARMWinCFISaveLR(unsigned Offset) {}
void ARMTargetStreamer::emitARMWinCFINop(bool Wide) {}
void ARMTargetStreamer::emitARMWinCFIPrologEnd(bool Fragment) {}
void ARMTargetStreamer::emitARMWinCFIEpilogStart(unsigned Condition) {}
void ARMTargetStreamer::emitARMWinCFIEpilogEnd() {}
void ARMTargetStreamer::emitARMWinCFICustom(unsigned Opcode) {}
```
- EN: Implements `ARMTargetStreamer::emitARMWinCFIAllocStack`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTargetStreamer::emitARMWinCFIAllocStack`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 135-137
```cpp
static ARMBuildAttrs::CPUArch getArchForCPU(const MCSubtargetInfo &STI) {
  if (STI.getCPU() == "xscale")
    return ARMBuildAttrs::v5TEJ;
```
- EN: Implements `getArchForCPU`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getArchForCPU`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 139-153
```cpp
  if (STI.hasFeature(ARM::HasV9_0aOps))
    return ARMBuildAttrs::v9_A;
  else if (STI.hasFeature(ARM::HasV8Ops)) {
    if (STI.hasFeature(ARM::FeatureRClass))
      return ARMBuildAttrs::v8_R;
    return ARMBuildAttrs::v8_A;
  } else if (STI.hasFeature(ARM::HasV8_1MMainlineOps))
    return ARMBuildAttrs::v8_1_M_Main;
  else if (STI.hasFeature(ARM::HasV8MMainlineOps))
    return ARMBuildAttrs::v8_M_Main;
  else if (STI.hasFeature(ARM::HasV7Ops)) {
    if (STI.hasFeature(ARM::FeatureMClass) && STI.hasFeature(ARM::FeatureDSP))
      return ARMBuildAttrs::v7E_M;
    return ARMBuildAttrs::v7;
  } else if (STI.hasFeature(ARM::HasV6T2Ops))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 154-168
```cpp
    return ARMBuildAttrs::v6T2;
  else if (STI.hasFeature(ARM::HasV8MBaselineOps))
    return ARMBuildAttrs::v8_M_Base;
  else if (STI.hasFeature(ARM::HasV6MOps))
    return ARMBuildAttrs::v6S_M;
  else if (STI.hasFeature(ARM::HasV6Ops))
    return ARMBuildAttrs::v6;
  else if (STI.hasFeature(ARM::HasV5TEOps))
    return ARMBuildAttrs::v5TE;
  else if (STI.hasFeature(ARM::HasV5TOps))
    return ARMBuildAttrs::v5T;
  else if (STI.hasFeature(ARM::HasV4TOps))
    return ARMBuildAttrs::v4T;
  else
    return ARMBuildAttrs::v4;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 169-169
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 171-176
```cpp
static bool isV8M(const MCSubtargetInfo &STI) {
  // Note that v8M Baseline is a subset of v6T2!
  return (STI.hasFeature(ARM::HasV8MBaselineOps) &&
          !STI.hasFeature(ARM::HasV6T2Ops)) ||
         STI.hasFeature(ARM::HasV8MMainlineOps);
}
```
- EN: Implements `isV8M`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isV8M`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 178-181
```cpp
/// Emit the build attributes that only depend on the hardware that we expect
// /to be available, and not on the ABI, or any source-language choices.
void ARMTargetStreamer::emitTargetAttributes(const MCSubtargetInfo &STI) {
  switchVendor("aeabi");
```
- EN: Implements `ARMTargetStreamer::emitTargetAttributes`, a emission/printing routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTargetStreamer::emitTargetAttributes`，它是一个围绕子目标特性处理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 183-196
```cpp
  const StringRef CPUString = STI.getCPU();
  if (!CPUString.empty() && !CPUString.starts_with("generic")) {
    // FIXME: remove krait check when GNU tools support krait cpu
    if (STI.hasFeature(ARM::ProcKrait)) {
      emitTextAttribute(ARMBuildAttrs::CPU_name, "cortex-a9");
      // We consider krait as a "cortex-a9" + hwdiv CPU
      // Enable hwdiv through ".arch_extension idiv"
      if (STI.hasFeature(ARM::FeatureHWDivThumb) ||
          STI.hasFeature(ARM::FeatureHWDivARM))
        emitArchExtension(ARM::AEK_HWDIVTHUMB | ARM::AEK_HWDIVARM);
    } else {
      emitTextAttribute(ARMBuildAttrs::CPU_name, CPUString);
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 198-198
```cpp
  emitAttribute(ARMBuildAttrs::CPU_arch, getArchForCPU(STI));
```
- EN: Declares `emitAttribute`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitAttribute`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 200-209
```cpp
  if (STI.hasFeature(ARM::FeatureAClass)) {
    emitAttribute(ARMBuildAttrs::CPU_arch_profile,
                      ARMBuildAttrs::ApplicationProfile);
  } else if (STI.hasFeature(ARM::FeatureRClass)) {
    emitAttribute(ARMBuildAttrs::CPU_arch_profile,
                      ARMBuildAttrs::RealTimeProfile);
  } else if (STI.hasFeature(ARM::FeatureMClass)) {
    emitAttribute(ARMBuildAttrs::CPU_arch_profile,
                      ARMBuildAttrs::MicroControllerProfile);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 211-213
```cpp
  emitAttribute(ARMBuildAttrs::ARM_ISA_use, STI.hasFeature(ARM::FeatureNoARM)
                                                ? ARMBuildAttrs::Not_Allowed
                                                : ARMBuildAttrs::Allowed);
```
- EN: Declares `emitAttribute`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitAttribute`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 215-223
```cpp
  if (isV8M(STI)) {
    emitAttribute(ARMBuildAttrs::THUMB_ISA_use,
                      ARMBuildAttrs::AllowThumbDerived);
  } else if (STI.hasFeature(ARM::FeatureThumb2)) {
    emitAttribute(ARMBuildAttrs::THUMB_ISA_use,
                      ARMBuildAttrs::AllowThumb32);
  } else if (STI.hasFeature(ARM::HasV4TOps)) {
    emitAttribute(ARMBuildAttrs::THUMB_ISA_use, ARMBuildAttrs::Allowed);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 225-239
```cpp
  if (STI.hasFeature(ARM::FeatureNEON)) {
    /* NEON is not exactly a VFP architecture, but GAS emit one of
     * neon/neon-fp-armv8/neon-vfpv4/vfpv3/vfpv2 for .fpu parameters */
    if (STI.hasFeature(ARM::FeatureFPARMv8)) {
      if (STI.hasFeature(ARM::FeatureCrypto))
        emitFPU(ARM::FK_CRYPTO_NEON_FP_ARMV8);
      else
        emitFPU(ARM::FK_NEON_FP_ARMV8);
    } else if (STI.hasFeature(ARM::FeatureVFP4))
      emitFPU(ARM::FK_NEON_VFPV4);
    else
      emitFPU(STI.hasFeature(ARM::FeatureFP16) ? ARM::FK_NEON_FP16
                                               : ARM::FK_NEON);
    // Emit Tag_Advanced_SIMD_arch for ARMv8 architecture
    if (STI.hasFeature(ARM::HasV8Ops))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 240-254
```cpp
      emitAttribute(ARMBuildAttrs::Advanced_SIMD_arch,
                    STI.hasFeature(ARM::HasV8_1aOps)
                        ? ARMBuildAttrs::AllowNeonARMv8_1a
                        : ARMBuildAttrs::AllowNeonARMv8);
  } else {
    if (STI.hasFeature(ARM::FeatureFPARMv8_D16_SP)) {
      // FPv5 and FP-ARMv8 have the same instructions, so are modeled as one
      // FPU, but there are two different names for it depending on the CPU.
      if (STI.hasFeature(ARM::FeatureD32))
        emitFPU(ARM::FK_FP_ARMV8);
      else {
        emitFPU(STI.hasFeature(ARM::FeatureFP64) ? ARM::FK_FPV5_D16
                                                 : ARM::FK_FPV5_SP_D16);
        if (STI.hasFeature(ARM::HasMVEFloatOps))
          emitArchExtension(ARM::AEK_MVE | ARM::AEK_DSP | ARM::AEK_FP);
```
- EN: Declares `emitAttribute`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitAttribute`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 255-269
```cpp
      }
    } else if (STI.hasFeature(ARM::FeatureVFP4_D16_SP))
      emitFPU(STI.hasFeature(ARM::FeatureD32)
                  ? ARM::FK_VFPV4
                  : (STI.hasFeature(ARM::FeatureFP64) ? ARM::FK_VFPV4_D16
                                                      : ARM::FK_FPV4_SP_D16));
    else if (STI.hasFeature(ARM::FeatureVFP3_D16_SP))
      emitFPU(
          STI.hasFeature(ARM::FeatureD32)
              // +d32
              ? (STI.hasFeature(ARM::FeatureFP16) ? ARM::FK_VFPV3_FP16
                                                  : ARM::FK_VFPV3)
              // -d32
              : (STI.hasFeature(ARM::FeatureFP64)
                     ? (STI.hasFeature(ARM::FeatureFP16)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 270-276
```cpp
                            ? ARM::FK_VFPV3_D16_FP16
                            : ARM::FK_VFPV3_D16)
                     : (STI.hasFeature(ARM::FeatureFP16) ? ARM::FK_VFPV3XD_FP16
                                                         : ARM::FK_VFPV3XD)));
    else if (STI.hasFeature(ARM::FeatureVFP2_SP))
      emitFPU(ARM::FK_VFPV2);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 278-281
```cpp
  // ABI_HardFP_use attribute to indicate single precision FP.
  if (STI.hasFeature(ARM::FeatureVFP2_SP) && !STI.hasFeature(ARM::FeatureFP64))
    emitAttribute(ARMBuildAttrs::ABI_HardFP_use,
                  ARMBuildAttrs::HardFPSinglePrecision);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 283-284
```cpp
  if (STI.hasFeature(ARM::FeatureFP16))
    emitAttribute(ARMBuildAttrs::FP_HP_extension, ARMBuildAttrs::AllowHPFP);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 286-287
```cpp
  if (STI.hasFeature(ARM::FeatureMP))
    emitAttribute(ARMBuildAttrs::MPextension_use, ARMBuildAttrs::AllowMP);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 289-292
```cpp
  if (STI.hasFeature(ARM::HasMVEFloatOps))
    emitAttribute(ARMBuildAttrs::MVE_arch, ARMBuildAttrs::AllowMVEIntegerAndFloat);
  else if (STI.hasFeature(ARM::HasMVEIntegerOps))
    emitAttribute(ARMBuildAttrs::MVE_arch, ARMBuildAttrs::AllowMVEInteger);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 294-301
```cpp
  // Hardware divide in ARM mode is part of base arch, starting from ARMv8.
  // If only Thumb hwdiv is present, it must also be in base arch (ARMv7-R/M).
  // It is not possible to produce DisallowDIV: if hwdiv is present in the base
  // arch, supplying -hwdiv downgrades the effective arch, via ClearImpliedBits.
  // AllowDIVExt is only emitted if hwdiv isn't available in the base arch;
  // otherwise, the default value (AllowDIVIfExists) applies.
  if (STI.hasFeature(ARM::FeatureHWDivARM) && !STI.hasFeature(ARM::HasV8Ops))
    emitAttribute(ARMBuildAttrs::DIV_use, ARMBuildAttrs::AllowDIVExt);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 303-304
```cpp
  if (STI.hasFeature(ARM::FeatureDSP) && isV8M(STI))
    emitAttribute(ARMBuildAttrs::DSP_extension, ARMBuildAttrs::Allowed);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 306-311
```cpp
  if (STI.hasFeature(ARM::FeatureStrictAlign))
    emitAttribute(ARMBuildAttrs::CPU_unaligned_access,
                  ARMBuildAttrs::Not_Allowed);
  else
    emitAttribute(ARMBuildAttrs::CPU_unaligned_access,
                  ARMBuildAttrs::Allowed);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 313-321
```cpp
  if (STI.hasFeature(ARM::FeatureTrustZone) &&
      STI.hasFeature(ARM::FeatureVirtualization))
    emitAttribute(ARMBuildAttrs::Virtualization_use,
                  ARMBuildAttrs::AllowTZVirtualization);
  else if (STI.hasFeature(ARM::FeatureTrustZone))
    emitAttribute(ARMBuildAttrs::Virtualization_use, ARMBuildAttrs::AllowTZ);
  else if (STI.hasFeature(ARM::FeatureVirtualization))
    emitAttribute(ARMBuildAttrs::Virtualization_use,
                  ARMBuildAttrs::AllowVirtualization);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 323-327
```cpp
  if (STI.hasFeature(ARM::FeaturePACBTI)) {
    emitAttribute(ARMBuildAttrs::PAC_extension, ARMBuildAttrs::AllowPAC);
    emitAttribute(ARMBuildAttrs::BTI_extension, ARMBuildAttrs::AllowBTI);
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 329-339
```cpp
MCTargetStreamer *
llvm::createARMObjectTargetStreamer(MCStreamer &S, const MCSubtargetInfo &STI) {
  const Triple &TT = STI.getTargetTriple();
  if (TT.isOSBinFormatELF())
    return createARMObjectTargetELFStreamer(S);
  if (TT.isOSBinFormatCOFF())
    return createARMObjectTargetWinCOFFStreamer(S);
  if (TT.isOSBinFormatMachO())
    return createARMObjectTargetMachOStreamer(S);
  return new ARMTargetStreamer(S);
}
```
- EN: Implements `llvm::createARMObjectTargetStreamer`, a query/helper routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createARMObjectTargetStreamer`，它是一个围绕MC streamer 交互展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: MC-layer target description and encoding support.
  - CN: 核心职责：MC 层目标描述与编码支持。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MCTargetDesc/ARMMCTargetDesc.h`.
  - CN: 后端本地头文件：`MCTargetDesc/ARMMCTargetDesc.h`。
- EN: LLVM infrastructure headers: `llvm/MC/ConstantPools.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCStreamer.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/Support/ARMBuildAttributes.h`.
  - CN: LLVM 基础设施头文件：`llvm/MC/ConstantPools.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCStreamer.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/Support/ARMBuildAttributes.h`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
