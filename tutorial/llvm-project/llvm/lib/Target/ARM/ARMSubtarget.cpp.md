# ARMSubtarget.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMSubtarget.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements the ARM specific subclass of TargetSubtargetInfo.
- 用途 (CN): 实现 ARM 后端中的 `ARMSubtarget`，重点处理子目标特性选择与 CPU 特定行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- ARMSubtarget.cpp - ARM Subtarget Information ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the ARM specific subclass of TargetSubtargetInfo.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-13
```cpp
#include "ARM.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 15-29
```cpp
#include "ARMCallLowering.h"
#include "ARMFrameLowering.h"
#include "ARMInstrInfo.h"
#include "ARMLegalizerInfo.h"
#include "ARMRegisterBankInfo.h"
#include "ARMSubtarget.h"
#include "ARMTargetMachine.h"
#include "MCTargetDesc/ARMMCTargetDesc.h"
#include "Thumb1FrameLowering.h"
#include "Thumb1InstrInfo.h"
#include "Thumb2InstrInfo.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/Twine.h"
#include "llvm/CodeGen/GlobalISel/InstructionSelect.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 30-39
```cpp
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/GlobalValue.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCTargetOptions.h"
#include "llvm/Support/CodeGen.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Target/TargetOptions.h"
#include "llvm/TargetParser/ARMTargetParser.h"
#include "llvm/TargetParser/Triple.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 41-41
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 43-43
```cpp
#define DEBUG_TYPE "arm-subtarget"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 45-47
```cpp
#define GET_SUBTARGETINFO_TARGET_DESC
#define GET_SUBTARGETINFO_CTOR
#include "ARMGenSubtargetInfo.inc"
```
- EN: Connects this file to TableGen-generated declarations so target-specific enums and helper tables become available to C++ code.
- CN: 这里把文件连接到 TableGen 生成的声明，使目标相关枚举和辅助表能够在 C++ 代码中使用。

### Lines 49-51
```cpp
static cl::opt<bool>
UseFusedMulOps("arm-use-mulops",
               cl::init(true), cl::Hidden);
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 53-56
```cpp
enum ITMode {
  DefaultIT,
  RestrictedIT
};
```
- EN: Defines enumeration `ITMode` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `ITMode`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 58-63
```cpp
static cl::opt<ITMode>
    IT(cl::desc("IT block support"), cl::Hidden, cl::init(DefaultIT),
       cl::values(clEnumValN(DefaultIT, "arm-default-it",
                             "Generate any type of IT block"),
                  clEnumValN(RestrictedIT, "arm-restrict-it",
                             "Disallow complex IT blocks")));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 65-69
```cpp
/// ForceFastISel - Use the fast-isel, even for subtargets where it is not
/// currently supported (for testing only).
static cl::opt<bool>
ForceFastISel("arm-force-fast-isel",
               cl::init(false), cl::Hidden);
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 71-77
```cpp
/// initializeSubtargetDependencies - Initializes using a CPU and feature string
/// so that we can use initializer lists for subtarget initialization.
ARMSubtarget &ARMSubtarget::initializeSubtargetDependencies(StringRef CPU,
                                                            StringRef FS) {
  initSubtargetFeatures(CPU, FS);
  return *this;
}
```
- EN: Implements `ARMSubtarget::initializeSubtargetDependencies`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::initializeSubtargetDependencies`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 79-83
```cpp
ARMFrameLowering *ARMSubtarget::initializeFrameLowering(StringRef CPU,
                                                        StringRef FS) {
  ARMSubtarget &STI = initializeSubtargetDependencies(CPU, FS);
  if (STI.isThumb1Only())
    return (ARMFrameLowering *)new Thumb1FrameLowering(STI);
```
- EN: Implements `ARMSubtarget::initializeFrameLowering`, a lowering routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::initializeFrameLowering`，它是一个围绕子目标特性处理展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 85-86
```cpp
  return new ARMFrameLowering(STI);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 88-101
```cpp
ARMSubtarget::ARMSubtarget(const Triple &TT, const std::string &CPU,
                           const std::string &FS,
                           const ARMBaseTargetMachine &TM, bool IsLittle,
                           bool MinSize, DenormalMode DM)
    : ARMGenSubtargetInfo(TT, CPU, /*TuneCPU*/ CPU, FS),
      UseMulOps(UseFusedMulOps), CPUString(CPU), OptMinSize(MinSize),
      IsLittle(IsLittle), DM(DM), TargetTriple(TT), Options(TM.Options), TM(TM),
      FrameLowering(initializeFrameLowering(CPU, FS)),
      // At this point initializeSubtargetDependencies has been called so
      // we can query directly.
      InstrInfo(isThumb1Only() ? (ARMBaseInstrInfo *)new Thumb1InstrInfo(*this)
                : !isThumb()   ? (ARMBaseInstrInfo *)new ARMInstrInfo(*this)
                             : (ARMBaseInstrInfo *)new Thumb2InstrInfo(*this)),
      TLInfo(TM, *this) {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 103-104
```cpp
  CallLoweringInfo.reset(new ARMCallLowering(*getTargetLowering()));
  Legalizer.reset(new ARMLegalizerInfo(*this));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 106-106
```cpp
  auto *RBI = new ARMRegisterBankInfo(*getRegisterInfo());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 108-111
```cpp
  // FIXME: At this point, we can't rely on Subtarget having RBI.
  // It's awkward to mix passing RBI and the Subtarget; should we pass
  // TII/TRI as well?
  InstSelector.reset(createARMInstructionSelector(TM, *this, *RBI));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 113-114
```cpp
  RegBankInfo.reset(RBI);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 116-118
```cpp
const CallLowering *ARMSubtarget::getCallLowering() const {
  return CallLoweringInfo.get();
}
```
- EN: Implements `ARMSubtarget::getCallLowering`, a lowering routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::getCallLowering`，它是一个围绕子目标特性处理展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 120-122
```cpp
InstructionSelector *ARMSubtarget::getInstructionSelector() const {
  return InstSelector.get();
}
```
- EN: Implements `ARMSubtarget::getInstructionSelector`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::getInstructionSelector`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 124-126
```cpp
const LegalizerInfo *ARMSubtarget::getLegalizerInfo() const {
  return Legalizer.get();
}
```
- EN: Implements `ARMSubtarget::getLegalizerInfo`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::getLegalizerInfo`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 128-130
```cpp
const RegisterBankInfo *ARMSubtarget::getRegBankInfo() const {
  return RegBankInfo.get();
}
```
- EN: Implements `ARMSubtarget::getRegBankInfo`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::getRegBankInfo`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 132-146
```cpp
void ARMSubtarget::initLibcallLoweringInfo(LibcallLoweringInfo &Info) const {
  const Triple &TT = getTargetTriple();
  if (TT.isOSBinFormatMachO()) {
    // Uses VFP for Thumb libfuncs if available.
    if (isThumb() && hasVFP2Base() && hasARMOps() && !useSoftFloat()) {
      // clang-format off
      static const struct {
        const RTLIB::Libcall Op;
        const RTLIB::LibcallImpl Impl;
      } LibraryCalls[] = {
        // Single-precision floating-point arithmetic.
        { RTLIB::ADD_F32, RTLIB::impl___addsf3vfp },
        { RTLIB::SUB_F32, RTLIB::impl___subsf3vfp },
        { RTLIB::MUL_F32, RTLIB::impl___mulsf3vfp },
        { RTLIB::DIV_F32, RTLIB::impl___divsf3vfp },
```
- EN: Implements `ARMSubtarget::initLibcallLoweringInfo`, a lowering routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::initLibcallLoweringInfo`，它是一个围绕子目标特性处理展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 148-152
```cpp
        // Double-precision floating-point arithmetic.
        { RTLIB::ADD_F64, RTLIB::impl___adddf3vfp },
        { RTLIB::SUB_F64, RTLIB::impl___subdf3vfp },
        { RTLIB::MUL_F64, RTLIB::impl___muldf3vfp },
        { RTLIB::DIV_F64, RTLIB::impl___divdf3vfp },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 154-161
```cpp
        // Single-precision comparisons.
        { RTLIB::OEQ_F32, RTLIB::impl___eqsf2vfp },
        { RTLIB::UNE_F32, RTLIB::impl___nesf2vfp },
        { RTLIB::OLT_F32, RTLIB::impl___ltsf2vfp },
        { RTLIB::OLE_F32, RTLIB::impl___lesf2vfp },
        { RTLIB::OGE_F32, RTLIB::impl___gesf2vfp },
        { RTLIB::OGT_F32, RTLIB::impl___gtsf2vfp },
        { RTLIB::UO_F32,  RTLIB::impl___unordsf2vfp },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 163-170
```cpp
        // Double-precision comparisons.
        { RTLIB::OEQ_F64, RTLIB::impl___eqdf2vfp },
        { RTLIB::UNE_F64, RTLIB::impl___nedf2vfp },
        { RTLIB::OLT_F64, RTLIB::impl___ltdf2vfp },
        { RTLIB::OLE_F64, RTLIB::impl___ledf2vfp },
        { RTLIB::OGE_F64, RTLIB::impl___gedf2vfp },
        { RTLIB::OGT_F64, RTLIB::impl___gtdf2vfp },
        { RTLIB::UO_F64,  RTLIB::impl___unorddf2vfp },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 172-178
```cpp
        // Floating-point to integer conversions.
        // i64 conversions are done via library routines even when generating VFP
        // instructions, so use the same ones.
        { RTLIB::FPTOSINT_F64_I32, RTLIB::impl___fixdfsivfp },
        { RTLIB::FPTOUINT_F64_I32, RTLIB::impl___fixunsdfsivfp },
        { RTLIB::FPTOSINT_F32_I32, RTLIB::impl___fixsfsivfp },
        { RTLIB::FPTOUINT_F32_I32, RTLIB::impl___fixunssfsivfp },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 180-182
```cpp
        // Conversions between floating types.
        { RTLIB::FPROUND_F64_F32, RTLIB::impl___truncdfsf2vfp },
        { RTLIB::FPEXT_F32_F64,   RTLIB::impl___extendsfdf2vfp },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 184-194
```cpp
        // Integer to floating-point conversions.
        // i64 conversions are done via library routines even when generating VFP
        // instructions, so use the same ones.
        // FIXME: There appears to be some naming inconsistency in ARM libgcc:
        // e.g., __floatunsidf vs. __floatunssidfvfp.
        { RTLIB::SINTTOFP_I32_F64, RTLIB::impl___floatsidfvfp },
        { RTLIB::UINTTOFP_I32_F64, RTLIB::impl___floatunssidfvfp },
        { RTLIB::SINTTOFP_I32_F32, RTLIB::impl___floatsisfvfp },
        { RTLIB::UINTTOFP_I32_F32, RTLIB::impl___floatunssisfvfp },
      };
      // clang-format on
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 196-200
```cpp
      for (const auto &LC : LibraryCalls)
        Info.setLibcallImpl(LC.Op, LC.Impl);
    }
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 202-205
```cpp
bool ARMSubtarget::isXRaySupported() const {
  // We don't currently support Thumb, but Windows requires Thumb.
  return hasV6Ops() && hasARMOps() && !isTargetWindows();
}
```
- EN: Implements `ARMSubtarget::isXRaySupported`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::isXRaySupported`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 207-209
```cpp
void ARMSubtarget::initSubtargetFeatures(StringRef CPU, StringRef FS) {
  if (CPUString.empty()) {
    CPUString = "generic";
```
- EN: Implements `ARMSubtarget::initSubtargetFeatures`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::initSubtargetFeatures`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 211-222
```cpp
    if (isTargetDarwin()) {
      StringRef ArchName = TargetTriple.getArchName();
      ARM::ArchKind AK = ARM::parseArch(ArchName);
      if (AK == ARM::ArchKind::ARMV7S)
        // Default to the Swift CPU when targeting armv7s/thumbv7s.
        CPUString = "swift";
      else if (AK == ARM::ArchKind::ARMV7K)
        // Default to the Cortex-a7 CPU when targeting armv7k/thumbv7k.
        // ARMv7k does not use SjLj exception handling.
        CPUString = "cortex-a7";
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 224-234
```cpp
  // Insert the architecture feature derived from the target triple into the
  // feature string. This is important for setting features that are implied
  // based on the architecture version.
  std::string ArchFS = ARM_MC::ParseARMTriple(TargetTriple, CPUString);
  if (!FS.empty()) {
    if (!ArchFS.empty())
      ArchFS = (Twine(ArchFS) + "," + FS).str();
    else
      ArchFS = std::string(FS);
  }
  ParseSubtargetFeatures(CPUString, /*TuneCPU*/ CPUString, ArchFS);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 236-238
```cpp
  // FIXME: This used enable V6T2 support implicitly for Thumb2 mode.
  // Assert this for now to make the change obvious.
  assert(hasV6T2Ops() || !hasThumb2());
```
- EN: Declares `hasV6T2Ops`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `hasV6T2Ops`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 240-246
```cpp
  if (genExecuteOnly()) {
    // Execute only support for >= v8-M Baseline requires movt support
    if (hasV8MBaselineOps())
      NoMovt = false;
    if (!hasV6MOps())
      report_fatal_error("Cannot generate execute-only code for this target");
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 248-249
```cpp
  // Keep a pointer to static instruction cost data for the specified CPU.
  SchedModel = getSchedModelForCPU(CPUString);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 251-252
```cpp
  // Initialize scheduling itinerary for the specified CPU.
  InstrItins = getInstrItineraryForCPU(CPUString);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 254-256
```cpp
  // FIXME: this is invalid for WindowsCE
  if (isTargetWindows())
    NoARM = true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 258-261
```cpp
  if (TM.isAAPCS_ABI())
    stackAlignment = Align(8);
  if (TM.isAAPCS16_ABI())
    stackAlignment = Align(16);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 263-274
```cpp
  // FIXME: Completely disable sibcall for Thumb1 since ThumbRegisterInfo::
  // emitEpilogue is not ready for them. Thumb tail calls also use t2B, as
  // the Thumb1 16-bit unconditional branch doesn't have sufficient relocation
  // support in the assembler and linker to be used. This would need to be
  // fixed to fully support tail calls in Thumb1.
  //
  // For ARMv8-M, we /do/ implement tail calls.  Doing this is tricky for v8-M
  // baseline, since the LDM/POP instruction on Thumb doesn't take LR.  This
  // means if we need to reload LR, it takes extra instructions, which outweighs
  // the value of the tail call; but here we don't know yet whether LR is going
  // to be used. We take the optimistic approach of generating the tail call and
  // perhaps taking a hit if we need to restore the LR.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 276-279
```cpp
  // Thumb1 PIC calls to external symbols use BX, so they can be tail calls,
  // but we need to make sure there are enough registers; the only valid
  // registers are the 4 used for parameters.  We don't currently do this
  // case.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 281-281
```cpp
  SupportsTailCall = !isThumb1Only() || hasV8MBaselineOps();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 283-290
```cpp
  switch (IT) {
  case DefaultIT:
    RestrictIT = false;
    break;
  case RestrictedIT:
    RestrictIT = true;
    break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 292-296
```cpp
  // NEON f32 ops are non-IEEE 754 compliant. Darwin is ok with it by default.
  const FeatureBitset &Bits = getFeatureBits();
  if ((Bits[ARM::ProcA5] || Bits[ARM::ProcA8]) && // Where this matters
      (isTargetDarwin() || DM == DenormalMode::getPreserveSign()))
    HasNEONForFP = true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 298-303
```cpp
  const ARM::ArchKind Arch = ARM::parseArch(TargetTriple.getArchName());
  if (isRWPI() ||
      (isTargetIOS() &&
       (Arch == ARM::ArchKind::ARMV6K || Arch == ARM::ArchKind::ARMV6) &&
       TargetTriple.isOSVersionLT(3, 0)))
    ReserveR9 = true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 305-307
```cpp
  // If MVEVectorCostFactor is still 0 (has not been set to anything else), default it to 2
  if (MVEVectorCostFactor == 0)
    MVEVectorCostFactor = 2;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 309-323
```cpp
  // FIXME: Teach TableGen to deal with these instead of doing it manually here.
  switch (ARMProcFamily) {
  case Others:
  case CortexA5:
    break;
  case CortexA7:
    LdStMultipleTiming = DoubleIssue;
    break;
  case CortexA8:
    LdStMultipleTiming = DoubleIssue;
    break;
  case CortexA9:
    LdStMultipleTiming = DoubleIssueCheckUnalignedAccess;
    PreISelOperandLatencyAdjustment = 1;
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 324-338
```cpp
  case CortexA12:
    break;
  case CortexA15:
    MaxInterleaveFactor = 2;
    PreISelOperandLatencyAdjustment = 1;
    PartialUpdateClearance = 12;
    break;
  case CortexA17:
  case CortexA32:
  case CortexA35:
  case CortexA53:
  case CortexA55:
  case CortexA57:
  case CortexA72:
  case CortexA73:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 339-353
```cpp
  case CortexA75:
  case CortexA76:
  case CortexA77:
  case CortexA78:
  case CortexA78AE:
  case CortexA78C:
  case CortexA510:
  case CortexA710:
  case CortexR4:
  case CortexR5:
  case CortexR7:
  case CortexM3:
  case CortexM55:
  case CortexM7:
  case CortexM85:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 354-368
```cpp
  case CortexR52:
  case CortexR52plus:
  case CortexX1:
  case CortexX1C:
    break;
  case Exynos:
    LdStMultipleTiming = SingleIssuePlusExtras;
    MaxInterleaveFactor = 4;
    if (!isThumb())
      PreferBranchLogAlignment = 3;
    break;
  case Kryo:
    break;
  case Krait:
    PreISelOperandLatencyAdjustment = 1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 369-379
```cpp
    break;
  case NeoverseV1:
    break;
  case Swift:
    MaxInterleaveFactor = 2;
    LdStMultipleTiming = SingleIssuePlusExtras;
    PreISelOperandLatencyAdjustment = 1;
    PartialUpdateClearance = 12;
    break;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 381-386
```cpp
bool ARMSubtarget::isROPI() const {
  // FIXME: This should ideally come from a function attribute, to work
  // correctly with LTO.
  return TM.getRelocationModel() == Reloc::ROPI ||
         TM.getRelocationModel() == Reloc::ROPI_RWPI;
}
```
- EN: Implements `ARMSubtarget::isROPI`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::isROPI`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 388-393
```cpp
bool ARMSubtarget::isRWPI() const {
  // FIXME: This should ideally come from a function attribute, to work
  // correctly with LTO.
  return TM.getRelocationModel() == Reloc::RWPI ||
         TM.getRelocationModel() == Reloc::ROPI_RWPI;
}
```
- EN: Implements `ARMSubtarget::isRWPI`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::isRWPI`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 395-397
```cpp
bool ARMSubtarget::isGVIndirectSymbol(const GlobalValue *GV) const {
  return TM.isGVIndirectSymbol(GV);
}
```
- EN: Implements `ARMSubtarget::isGVIndirectSymbol`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::isGVIndirectSymbol`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 399-401
```cpp
bool ARMSubtarget::isGVInGOT(const GlobalValue *GV) const {
  return isTargetELF() && TM.isPositionIndependent() && !GV->isDSOLocal();
}
```
- EN: Implements `ARMSubtarget::isGVInGOT`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::isGVInGOT`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 403-405
```cpp
unsigned ARMSubtarget::getMispredictionPenalty() const {
  return SchedModel.MispredictPenalty;
}
```
- EN: Implements `ARMSubtarget::getMispredictionPenalty`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::getMispredictionPenalty`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 407-418
```cpp
bool ARMSubtarget::enableMachineScheduler() const {
  // The MachineScheduler can increase register usage, so we use more high
  // registers and end up with more T2 instructions that cannot be converted to
  // T1 instructions. At least until we do better at converting to thumb1
  // instructions, on cortex-m at Oz where we are size-paranoid, don't use the
  // Machine scheduler, relying on the DAG register pressure scheduler instead.
  if (isMClass() && hasMinSize())
    return false;
  // Enable the MachineScheduler before register allocation for subtargets
  // with the use-misched feature.
  return useMachineScheduler();
}
```
- EN: Implements `ARMSubtarget::enableMachineScheduler`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::enableMachineScheduler`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 420-424
```cpp
bool ARMSubtarget::enableSubRegLiveness() const {
  // Enable SubRegLiveness for MVE to better optimize s subregs for mqpr regs
  // and q subregs for qqqqpr regs.
  return hasMVEIntegerOps();
}
```
- EN: Implements `ARMSubtarget::enableSubRegLiveness`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::enableSubRegLiveness`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 426-430
```cpp
bool ARMSubtarget::enableMachinePipeliner() const {
  // Enable the MachinePipeliner before register allocation for subtargets
  // with the use-mipipeliner feature.
  return getSchedModel().hasInstrSchedModel() && useMachinePipeliner();
}
```
- EN: Implements `ARMSubtarget::enableMachinePipeliner`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::enableMachinePipeliner`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 432-432
```cpp
bool ARMSubtarget::useDFAforSMS() const { return false; }
```
- EN: Implements `ARMSubtarget::useDFAforSMS`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::useDFAforSMS`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 434-442
```cpp
// This overrides the PostRAScheduler bit in the SchedModel for any CPU.
bool ARMSubtarget::enablePostRAScheduler() const {
  if (enableMachineScheduler())
    return false;
  if (disablePostRAScheduler())
    return false;
  // Thumb1 cores will generally not benefit from post-ra scheduling
  return !isThumb1Only();
}
```
- EN: Implements `ARMSubtarget::enablePostRAScheduler`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::enablePostRAScheduler`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 444-450
```cpp
bool ARMSubtarget::enablePostRAMachineScheduler() const {
  if (!enableMachineScheduler())
    return false;
  if (disablePostRAScheduler())
    return false;
  return !isThumb1Only();
}
```
- EN: Implements `ARMSubtarget::enablePostRAMachineScheduler`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::enablePostRAMachineScheduler`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 452-458
```cpp
bool ARMSubtarget::useStride4VFPs() const {
  // For general targets, the prologue can grow when VFPs are allocated with
  // stride 4 (more vpush instructions). But WatchOS uses a compact unwind
  // format which it's more important to get right.
  return isTargetWatchABI() ||
         (useWideStrideVFP() && !OptMinSize);
}
```
- EN: Implements `ARMSubtarget::useStride4VFPs`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::useStride4VFPs`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 460-466
```cpp
bool ARMSubtarget::useMovt() const {
  // NOTE Windows on ARM needs to use mov.w/mov.t pairs to materialise 32-bit
  // immediates as it is inherently position independent, and may be out of
  // range otherwise.
  return !NoMovt && hasV8MBaselineOps() &&
         (isTargetWindows() || !OptMinSize || genExecuteOnly());
}
```
- EN: Implements `ARMSubtarget::useMovt`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::useMovt`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 468-471
```cpp
bool ARMSubtarget::useFastISel() const {
  // Enable fast-isel for any target, for testing only.
  if (ForceFastISel)
    return true;
```
- EN: Implements `ARMSubtarget::useFastISel`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::useFastISel`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 473-475
```cpp
  // Limit fast-isel to the targets that are or have been tested.
  if (!hasV6Ops())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 477-480
```cpp
  // Thumb2 support on iOS; ARM support on iOS and Linux.
  return TM.Options.EnableFastISel && ((isTargetMachO() && !isThumb1Only()) ||
                                       (isTargetLinux() && !isThumb()));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 482-492
```cpp
unsigned ARMSubtarget::getGPRAllocationOrder(const MachineFunction &MF) const {
  // The GPR register class has multiple possible allocation orders, with
  // tradeoffs preferred by different sub-architectures and optimisation goals.
  // The allocation orders are:
  // 0: (the default tablegen order, not used)
  // 1: r14, r0-r13
  // 2: r0-r7
  // 3: r0-r7, r12, lr, r8-r11
  // Note that the register allocator will change this order so that
  // callee-saved registers are used later, as they require extra work in the
  // prologue/epilogue (though we sometimes override that).
```
- EN: Declares `has`, packaging target-specific state and APIs around `ARMSubtarget`.
- CN: 这里声明 `has`，把与 `ARMSubtarget` 相关的目标特定状态和 API 组织在一起。

### Lines 494-496
```cpp
  // For thumb1-only targets, only the low registers are allocatable.
  if (isThumb1Only())
    return 2;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 498-506
```cpp
  // Allocate low registers first, so we can select more 16-bit instructions.
  // We also (in ignoreCSRForAllocationOrder) override  the default behaviour
  // with regards to callee-saved registers, because pushing extra registers is
  // much cheaper (in terms of code size) than using high registers. After
  // that, we allocate r12 (doesn't need to be saved), lr (saving it means we
  // can return with the pop, don't need an extra "bx lr") and then the rest of
  // the high registers.
  if (isThumb2() && MF.getFunction().hasMinSize())
    return 3;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 508-511
```cpp
  // Otherwise, allocate in the default order, using LR first because saving it
  // allows a shorter epilogue sequence.
  return 1;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 513-522
```cpp
bool ARMSubtarget::ignoreCSRForAllocationOrder(const MachineFunction &MF,
                                               MCRegister PhysReg) const {
  // To minimize code size in Thumb2, we prefer the usage of low regs (lower
  // cost per use) so we can  use narrow encoding. By default, caller-saved
  // registers (e.g. lr, r12) are always  allocated first, regardless of
  // their cost per use. When optForMinSize, we prefer the low regs even if
  // they are CSR because usually push/pop can be folded into existing ones.
  return isThumb2() && MF.getFunction().hasMinSize() &&
         ARM::GPRRegClass.contains(PhysReg);
}
```
- EN: Implements `ARMSubtarget::ignoreCSRForAllocationOrder`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::ignoreCSRForAllocationOrder`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 524-527
```cpp
ARMSubtarget::PushPopSplitVariation
ARMSubtarget::getPushPopSplitVariation(const MachineFunction &MF) const {
  const Function &F = MF.getFunction();
  const MachineFrameInfo &MFI = MF.getFrameInfo();
```
- EN: Implements `ARMSubtarget::getPushPopSplitVariation`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSubtarget::getPushPopSplitVariation`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 529-532
```cpp
  // Thumb1 always splits the pushes at R7, because the Thumb1 push instruction
  // cannot use high registers except for lr.
  if (isThumb1Only())
    return SplitR7;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 534-539
```cpp
  // If R7 is the frame pointer, we must split at R7 to ensure that the
  // previous frame pointer (R7) and return address (LR) are adjacent on the
  // stack, to form a valid frame record.
  if (getFramePointerReg() == ARM::R7 &&
      MF.getTarget().Options.FramePointerIsReserved(MF))
    return SplitR7;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 541-549
```cpp
  // Returns SplitR11WindowsSEH when the stack pointer needs to be
  // restored from the frame pointer r11 + an offset and Windows CFI is enabled.
  // This stack unwinding cannot be expressed with SEH unwind opcodes when done
  // with a single push, making it necessary to split the push into r4-r10, and
  // another containing r11+lr.
  if (MF.getTarget().getMCAsmInfo().usesWindowsCFI() &&
      F.needsUnwindTableEntry() &&
      (MFI.hasVarSizedObjects() || getRegisterInfo()->hasStackRealignment(MF)))
    return SplitR11WindowsSEH;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 551-559
```cpp
  // Returns SplitR11AAPCSSignRA when the frame pointer is R11, requiring R11
  // and LR to be adjacent on the stack, and branch signing is enabled,
  // requiring R12 to be on the stack.
  if (MF.getInfo<ARMFunctionInfo>()->shouldSignReturnAddress() &&
      getFramePointerReg() == ARM::R11 &&
      MF.getTarget().Options.FramePointerIsReserved(MF))
    return SplitR11AAPCSSignRA;
  return NoSplit;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

## Key Concepts / 关键概念

- EN: Primary role: subtarget feature selection and CPU-specific behavior.
  - CN: 核心职责：子目标特性选择与 CPU 特定行为。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: Stack-frame and spill/reload concerns are reflected in the API or implementation.
  - CN: API 或实现中体现了栈帧以及溢出/重载相关问题。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARM.h`, `ARMCallLowering.h`, `ARMFrameLowering.h`, `ARMInstrInfo.h`, `ARMLegalizerInfo.h`, `ARMRegisterBankInfo.h`, `ARMSubtarget.h`, `ARMTargetMachine.h` ... (+5 more).
  - CN: 后端本地头文件：`ARM.h`, `ARMCallLowering.h`, `ARMFrameLowering.h`, `ARMInstrInfo.h`, `ARMLegalizerInfo.h`, `ARMRegisterBankInfo.h`, `ARMSubtarget.h`, `ARMTargetMachine.h` ... (+5 more)。
- EN: LLVM infrastructure headers: `llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h`, `llvm/CodeGen/GlobalISel/InstructionSelect.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/IR/Function.h`, `llvm/IR/GlobalValue.h`, `llvm/MC/MCAsmInfo.h` ... (+6 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h`, `llvm/CodeGen/GlobalISel/InstructionSelect.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/IR/Function.h`, `llvm/IR/GlobalValue.h`, `llvm/MC/MCAsmInfo.h` ... (+6 more)。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
