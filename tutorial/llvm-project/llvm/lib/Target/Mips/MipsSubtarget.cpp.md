# MipsSubtarget.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsSubtarget.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements the Mips specific subclass of TargetSubtargetInfo.
- 用途 (CN): 实现 Mips 后端中的 `MipsSubtarget`，重点处理子目标特性选择与 CPU 特定行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- MipsSubtarget.cpp - Mips Subtarget Information --------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the Mips specific subclass of TargetSubtargetInfo.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-26
```cpp
#include "MipsSubtarget.h"
#include "Mips.h"
#include "MipsCallLowering.h"
#include "MipsLegalizerInfo.h"
#include "MipsRegisterBankInfo.h"
#include "MipsRegisterInfo.h"
#include "MipsSelectionDAGInfo.h"
#include "MipsTargetMachine.h"
#include "llvm/IR/Attributes.h"
#include "llvm/IR/Function.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 28-28
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 30-38
```cpp
cl::opt<CompactBranchPolicy> MipsCompactBranchPolicy(
    "mips-compact-branches", cl::Optional, cl::init(CB_Optimal),
    cl::desc("MIPS Specific: Compact branch policy."),
    cl::values(clEnumValN(CB_Never, "never",
                          "Do not use compact branches if possible."),
               clEnumValN(CB_Optimal, "optimal",
                          "Use compact branches where appropriate (default)."),
               clEnumValN(CB_Always, "always",
                          "Always use compact branches if possible.")));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 40-40
```cpp
#define DEBUG_TYPE "mips-subtarget"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 42-44
```cpp
#define GET_SUBTARGETINFO_TARGET_DESC
#define GET_SUBTARGETINFO_CTOR
#include "MipsGenSubtargetInfo.inc"
```
- EN: Connects this file to TableGen-generated declarations so target-specific enums and helper tables become available to C++ code.
- CN: 这里把文件连接到 TableGen 生成的声明，使目标相关枚举和辅助表能够在 C++ 代码中使用。

### Lines 46-52
```cpp
// FIXME: Maybe this should be on by default when Mips16 is specified
//
static cl::opt<bool>
    Mixed16_32("mips-mixed-16-32", cl::init(false),
               cl::desc("Allow for a mixture of Mips16 "
                        "and Mips32 code in a single output file"),
               cl::Hidden);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 54-57
```cpp
static cl::opt<bool> Mips_Os16("mips-os16", cl::init(false),
                               cl::desc("Compile all functions that don't use "
                                        "floating point as Mips 16"),
                               cl::Hidden);
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 59-61
```cpp
static cl::opt<bool> Mips16HardFloat("mips16-hard-float", cl::NotHidden,
                                     cl::desc("Enable mips16 hard float."),
                                     cl::init(false));
```
- EN: Declares `cl::desc`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::desc`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 63-66
```cpp
static cl::opt<bool>
    Mips16ConstantIslands("mips16-constant-islands", cl::NotHidden,
                          cl::desc("Enable mips16 constant islands."),
                          cl::init(true));
```
- EN: Declares `cl::desc`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::desc`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 68-70
```cpp
static cl::opt<bool>
    GPOpt("mgpopt", cl::Hidden,
          cl::desc("Enable gp-relative addressing of mips small data items"));
```
- EN: Declares `cl::desc`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::desc`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 72-77
```cpp
bool MipsSubtarget::DspWarningPrinted = false;
bool MipsSubtarget::MSAWarningPrinted = false;
bool MipsSubtarget::VirtWarningPrinted = false;
bool MipsSubtarget::CRCWarningPrinted = false;
bool MipsSubtarget::GINVWarningPrinted = false;
bool MipsSubtarget::MIPS1WarningPrinted = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 79-79
```cpp
void MipsSubtarget::anchor() {}
```
- EN: Implements `MipsSubtarget::anchor`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSubtarget::anchor`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 81-95
```cpp
MipsSubtarget::MipsSubtarget(const Triple &TT, StringRef CPU, StringRef FS,
                             bool little, const MipsTargetMachine &TM,
                             MaybeAlign StackAlignOverride)
    : MipsGenSubtargetInfo(TT, CPU, /*TuneCPU*/ CPU, FS),
      MipsArchVersion(MipsDefault), IsLittle(little), IsSoftFloat(false),
      IsSingleFloat(false), IsFPXX(false), NoABICalls(false), Abs2008(false),
      IsFP64bit(false), UseOddSPReg(true), IsNaN2008bit(false),
      IsGP64bit(false), HasVFPU(false), HasCnMips(false), HasCnMipsP(false),
      IsR5900(false), FixR5900(false), HasMips3_32(false), HasMips3_32r2(false),
      HasMips4_32(false), HasMips4_32r2(false), HasMips5_32r2(false),
      InMips16Mode(false), InMips16HardFloat(Mips16HardFloat),
      InMicroMipsMode(false), HasDSP(false), HasDSPR2(false), HasDSPR3(false),
      AllowMixed16_32(Mixed16_32 || Mips_Os16), Os16(Mips_Os16), HasMSA(false),
      UseTCCInDIV(false), HasSym32(false), HasEVA(false), DisableMadd4(false),
      HasMT(false), HasCRC(false), HasVirt(false), HasGINV(false),
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 96-102
```cpp
      UseIndirectJumpsHazard(false), StrictAlign(false),
      UseCompactBranches(MipsCompactBranchPolicy != CB_Never),
      StackAlignOverride(StackAlignOverride), TM(TM), TargetTriple(TT),
      InstrInfo(
          MipsInstrInfo::create(initializeSubtargetDependencies(CPU, FS, TM))),
      FrameLowering(MipsFrameLowering::create(*this)),
      TLInfo(MipsTargetLowering::create(TM, *this)) {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 104-105
```cpp
  if (MipsArchVersion == MipsDefault)
    MipsArchVersion = Mips32;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 107-111
```cpp
  // MIPS-I has not been tested.
  if (MipsArchVersion == Mips1 && !MIPS1WarningPrinted) {
    errs() << "warning: MIPS-I support is experimental\n";
    MIPS1WarningPrinted = true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 113-116
```cpp
  // Don't even attempt to generate code for MIPS-V. It has not
  // been tested and currently exists for the integrated assembler only.
  if (MipsArchVersion == Mips5)
    report_fatal_error("Code generation for MIPS-V is not implemented", false);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 118-120
```cpp
  // Check if Architecture and ABI are compatible.
  assert(((!isGP64bit() && isABI_O32()) || isGP64bit()) &&
         "Invalid  Arch & ABI pair.");
```
- EN: Declares `isGP64bit`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isGP64bit`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 122-125
```cpp
  if (hasMSA() && !isFP64bit())
    report_fatal_error("MSA requires a 64-bit FPU register file (FR=1 mode). "
                       "See -mattr=+fp64.",
                       false);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 127-130
```cpp
  if (isFP64bit() && !hasMips64() && hasMips32() && !hasMips32r2())
    report_fatal_error(
        "FPU with 64-bit registers is not available on MIPS32 pre revision 2. "
        "Use -mcpu=mips32r2 or greater.", false);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 132-133
```cpp
  if (!isABI_O32() && !useOddSPReg())
    report_fatal_error("-mattr=+nooddspreg requires the O32 ABI.", false);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 135-136
```cpp
  if (IsFPXX && (isABI_N32() || isABI_N64()))
    report_fatal_error("FPXX is not permitted for the N32/N64 ABI's.", false);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 138-139
```cpp
  if (hasMips64r6() && InMicroMipsMode)
    report_fatal_error("microMIPS64R6 is not supported", false);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 141-142
```cpp
  if (!isABI_O32() && InMicroMipsMode)
    report_fatal_error("microMIPS64 is not supported.", false);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 144-156
```cpp
  if (UseIndirectJumpsHazard) {
    if (InMicroMipsMode)
      report_fatal_error(
          "cannot combine indirect jumps with hazard barriers and microMIPS");
    if (!hasMips32r2())
      report_fatal_error(
          "indirect jumps with hazard barriers requires MIPS32R2 or later");
  }
  if (inAbs2008Mode() && hasMips32() && !hasMips32r2()) {
    report_fatal_error("IEEE 754-2008 abs.fmt is not supported for the given "
                       "architecture.",
                       false);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 158-159
```cpp
  if (hasMips32r6()) {
    StringRef ISA = hasMips64r6() ? "MIPS64r6" : "MIPS32r6";
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 161-166
```cpp
    assert(isFP64bit());
    assert(isNaN2008());
    assert(inAbs2008Mode());
    if (hasDSP())
      report_fatal_error(ISA + " is not compatible with the DSP ASE", false);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 168-169
```cpp
  if (NoABICalls && TM.isPositionIndependent())
    report_fatal_error("position-independent code requires '-mabicalls'");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 171-172
```cpp
  if (isABI_N64() && !TM.isPositionIndependent() && !hasSym32())
    NoABICalls = true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 174-180
```cpp
  // Set UseSmallSection.
  UseSmallSection = GPOpt;
  if (!NoABICalls && GPOpt) {
    errs() << "warning: cannot use small-data accesses for '-mabicalls'"
           << "\n";
    UseSmallSection = false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 182-196
```cpp
  if (hasDSPR2() && !DspWarningPrinted) {
    if (hasMips64() && !hasMips64r2()) {
      errs() << "warning: the 'dspr2' ASE requires MIPS64 revision 2 or "
             << "greater\n";
      DspWarningPrinted = true;
    } else if (hasMips32() && !hasMips32r2()) {
      errs() << "warning: the 'dspr2' ASE requires MIPS32 revision 2 or "
             << "greater\n";
      DspWarningPrinted = true;
    }
  } else if (hasDSP() && !DspWarningPrinted) {
    if (hasMips64() && !hasMips64r2()) {
      errs() << "warning: the 'dsp' ASE requires MIPS64 revision 2 or "
             << "greater\n";
      DspWarningPrinted = true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 197-202
```cpp
    } else if (hasMips32() && !hasMips32r2()) {
      errs() << "warning: the 'dsp' ASE requires MIPS32 revision 2 or "
             << "greater\n";
      DspWarningPrinted = true;
    }
  }
```
- EN: Implements `hasMips32`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMips32`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 204-204
```cpp
  StringRef ArchName = hasMips64() ? "MIPS64" : "MIPS32";
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 206-220
```cpp
  if (!hasMips32r5() && hasMSA() && !MSAWarningPrinted) {
    errs() << "warning: the 'msa' ASE requires " << ArchName
           << " revision 5 or greater\n";
    MSAWarningPrinted = true;
  }
  if (!hasMips32r5() && hasVirt() && !VirtWarningPrinted) {
    errs() << "warning: the 'virt' ASE requires " << ArchName
           << " revision 5 or greater\n";
    VirtWarningPrinted = true;
  }
  if (!hasMips32r6() && hasCRC() && !CRCWarningPrinted) {
    errs() << "warning: the 'crc' ASE requires " << ArchName
           << " revision 6 or greater\n";
    CRCWarningPrinted = true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 221-225
```cpp
  if (!hasMips32r6() && hasGINV() && !GINVWarningPrinted) {
    errs() << "warning: the 'ginv' ASE requires " << ArchName
           << " revision 6 or greater\n";
    GINVWarningPrinted = true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 227-227
```cpp
  TSInfo = std::make_unique<MipsSelectionDAGInfo>();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 229-230
```cpp
  CallLoweringInfo.reset(new MipsCallLowering(*getTargetLowering()));
  Legalizer.reset(new MipsLegalizerInfo(*this));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 232-235
```cpp
  auto *RBI = new MipsRegisterBankInfo(*getRegisterInfo());
  RegBankInfo.reset(RBI);
  InstSelector.reset(createMipsInstructionSelector(TM, *this, *RBI));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 237-237
```cpp
MipsSubtarget::~MipsSubtarget() = default;
```
- EN: Declares `MipsSubtarget::~MipsSubtarget`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `MipsSubtarget::~MipsSubtarget`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 239-241
```cpp
bool MipsSubtarget::isPositionIndependent() const {
  return TM.isPositionIndependent();
}
```
- EN: Implements `MipsSubtarget::isPositionIndependent`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSubtarget::isPositionIndependent`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 243-244
```cpp
/// This overrides the PostRAScheduler bit in the SchedModel for any CPU.
bool MipsSubtarget::enablePostRAScheduler() const { return true; }
```
- EN: Implements `MipsSubtarget::enablePostRAScheduler`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSubtarget::enablePostRAScheduler`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 246-250
```cpp
void MipsSubtarget::getCriticalPathRCs(RegClassVector &CriticalPathRCs) const {
  CriticalPathRCs.clear();
  CriticalPathRCs.push_back(isGP64bit() ? &Mips::GPR64RegClass
                                        : &Mips::GPR32RegClass);
}
```
- EN: Implements `MipsSubtarget::getCriticalPathRCs`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSubtarget::getCriticalPathRCs`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 252-254
```cpp
CodeGenOptLevel MipsSubtarget::getOptLevelToEnablePostRAScheduler() const {
  return CodeGenOptLevel::Aggressive;
}
```
- EN: Implements `MipsSubtarget::getOptLevelToEnablePostRAScheduler`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSubtarget::getOptLevelToEnablePostRAScheduler`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 256-259
```cpp
MipsSubtarget &
MipsSubtarget::initializeSubtargetDependencies(StringRef CPU, StringRef FS,
                                               const TargetMachine &TM) {
  StringRef CPUName = MIPS_MC::selectMipsCPU(TM.getTargetTriple(), CPU);
```
- EN: Implements `MipsSubtarget::initializeSubtargetDependencies`, a target-specific routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSubtarget::initializeSubtargetDependencies`，它是一个围绕目标机器策略展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 261-264
```cpp
  // Parse features string.
  ParseSubtargetFeatures(CPUName, /*TuneCPU*/ CPUName, FS);
  if (InMips16Mode && !IsSoftFloat)
    InMips16HardFloat = true;
```
- EN: Declares `ParseSubtargetFeatures`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ParseSubtargetFeatures`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 266-273
```cpp
  if (StackAlignOverride)
    stackAlignment = *StackAlignOverride;
  else if (isABI_N32() || isABI_N64())
    stackAlignment = Align(16);
  else {
    assert(isABI_O32() && "Unknown ABI for stack alignment!");
    stackAlignment = Align(8);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 275-277
```cpp
  if ((isABI_N32() || isABI_N64()) && !isGP64bit())
    reportFatalUsageError("64-bit code requested on a subtarget that doesn't "
                          "support it!");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 279-280
```cpp
  return *this;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 282-286
```cpp
bool MipsSubtarget::useConstantIslands() {
  LLVM_DEBUG(dbgs() << "use constant islands " << Mips16ConstantIslands
                    << "\n");
  return Mips16ConstantIslands;
}
```
- EN: Implements `MipsSubtarget::useConstantIslands`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSubtarget::useConstantIslands`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 288-290
```cpp
Reloc::Model MipsSubtarget::getRelocationModel() const {
  return TM.getRelocationModel();
}
```
- EN: Implements `MipsSubtarget::getRelocationModel`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSubtarget::getRelocationModel`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 292-295
```cpp
bool MipsSubtarget::isABI_N64() const { return getABI().IsN64(); }
bool MipsSubtarget::isABI_N32() const { return getABI().IsN32(); }
bool MipsSubtarget::isABI_O32() const { return getABI().IsO32(); }
const MipsABIInfo &MipsSubtarget::getABI() const { return TM.getABI(); }
```
- EN: Implements `MipsSubtarget::isABI_N64`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSubtarget::isABI_N64`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 297-299
```cpp
const SelectionDAGTargetInfo *MipsSubtarget::getSelectionDAGInfo() const {
  return TSInfo.get();
}
```
- EN: Implements `MipsSubtarget::getSelectionDAGInfo`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSubtarget::getSelectionDAGInfo`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 301-303
```cpp
const CallLowering *MipsSubtarget::getCallLowering() const {
  return CallLoweringInfo.get();
}
```
- EN: Implements `MipsSubtarget::getCallLowering`, a lowering routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSubtarget::getCallLowering`，它是一个围绕子目标特性处理展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 305-307
```cpp
const LegalizerInfo *MipsSubtarget::getLegalizerInfo() const {
  return Legalizer.get();
}
```
- EN: Implements `MipsSubtarget::getLegalizerInfo`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSubtarget::getLegalizerInfo`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 309-311
```cpp
const RegisterBankInfo *MipsSubtarget::getRegBankInfo() const {
  return RegBankInfo.get();
}
```
- EN: Implements `MipsSubtarget::getRegBankInfo`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSubtarget::getRegBankInfo`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 313-315
```cpp
InstructionSelector *MipsSubtarget::getInstructionSelector() const {
  return InstSelector.get();
}
```
- EN: Implements `MipsSubtarget::getInstructionSelector`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSubtarget::getInstructionSelector`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 317-331
```cpp
// Libcalls for which no helper is generated. Sorted by name for binary search.
const RTLIB::LibcallImpl MipsSubtarget::HardFloatLibCalls[34] = {
    RTLIB::impl___mips16_adddf3,        RTLIB::impl___mips16_addsf3,
    RTLIB::impl___mips16_divdf3,        RTLIB::impl___mips16_divsf3,
    RTLIB::impl___mips16_eqdf2,         RTLIB::impl___mips16_eqsf2,
    RTLIB::impl___mips16_extendsfdf2,   RTLIB::impl___mips16_fix_truncdfsi,
    RTLIB::impl___mips16_fix_truncsfsi, RTLIB::impl___mips16_floatsidf,
    RTLIB::impl___mips16_floatsisf,     RTLIB::impl___mips16_floatunsidf,
    RTLIB::impl___mips16_floatunsisf,   RTLIB::impl___mips16_gedf2,
    RTLIB::impl___mips16_gesf2,         RTLIB::impl___mips16_gtdf2,
    RTLIB::impl___mips16_gtsf2,         RTLIB::impl___mips16_ledf2,
    RTLIB::impl___mips16_lesf2,         RTLIB::impl___mips16_ltdf2,
    RTLIB::impl___mips16_ltsf2,         RTLIB::impl___mips16_muldf3,
    RTLIB::impl___mips16_mulsf3,        RTLIB::impl___mips16_nedf2,
    RTLIB::impl___mips16_nesf2,         RTLIB::impl___mips16_ret_dc,
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 332-335
```cpp
    RTLIB::impl___mips16_ret_df,        RTLIB::impl___mips16_ret_sc,
    RTLIB::impl___mips16_ret_sf,        RTLIB::impl___mips16_subdf3,
    RTLIB::impl___mips16_subsf3,        RTLIB::impl___mips16_truncdfsf2,
    RTLIB::impl___mips16_unorddf2,      RTLIB::impl___mips16_unordsf2};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 337-347
```cpp
void MipsSubtarget::initLibcallLoweringInfo(LibcallLoweringInfo &Info) const {
  if (inMips16Mode() && !useSoftFloat()) {
    for (unsigned I = 0; I != std::size(HardFloatLibCalls); ++I) {
      assert((I == 0 || HardFloatLibCalls[I - 1] < HardFloatLibCalls[I]) &&
             "Array not sorted!");
      RTLIB::Libcall LC =
          RTLIB::RuntimeLibcallsInfo::getLibcallFromImpl(HardFloatLibCalls[I]);
      Info.setLibcallImpl(LC, HardFloatLibCalls[I]);
    }
  }
}
```
- EN: Implements `MipsSubtarget::initLibcallLoweringInfo`, a lowering routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSubtarget::initLibcallLoweringInfo`，它是一个围绕子目标特性处理展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: subtarget feature selection and CPU-specific behavior.
  - CN: 核心职责：子目标特性选择与 CPU 特定行为。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: Stack-frame and spill/reload concerns are reflected in the API or implementation.
  - CN: API 或实现中体现了栈帧以及溢出/重载相关问题。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MipsSubtarget.h`, `Mips.h`, `MipsCallLowering.h`, `MipsLegalizerInfo.h`, `MipsRegisterBankInfo.h`, `MipsRegisterInfo.h`, `MipsSelectionDAGInfo.h`, `MipsTargetMachine.h` ... (+1 more).
  - CN: 后端本地头文件：`MipsSubtarget.h`, `Mips.h`, `MipsCallLowering.h`, `MipsLegalizerInfo.h`, `MipsRegisterBankInfo.h`, `MipsRegisterInfo.h`, `MipsSelectionDAGInfo.h`, `MipsTargetMachine.h` ... (+1 more)。
- EN: LLVM infrastructure headers: `llvm/IR/Attributes.h`, `llvm/IR/Function.h`, `llvm/MC/TargetRegistry.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`.
  - CN: LLVM 基础设施头文件：`llvm/IR/Attributes.h`, `llvm/IR/Function.h`, `llvm/MC/TargetRegistry.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
