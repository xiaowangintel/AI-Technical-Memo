# RISCVSubtarget.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVSubtarget.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements CPU-specific and feature-specific subtarget modeling for RISC-V. / 实现RISC-V 的 CPU 专用与特性专用子目标建模。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Commentary and design intent / 注释与设计意图
```cpp
//===-- RISCVSubtarget.cpp - RISC-V Subtarget Information -----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the RISC-V specific subclass of TargetSubtargetInfo.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 12-24: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#include "RISCVSubtarget.h"
#include "GISel/RISCVCallLowering.h"
#include "GISel/RISCVInlineAsmLowering.h"
#include "GISel/RISCVLegalizerInfo.h"
#include "RISCV.h"
#include "RISCVFrameLowering.h"
#include "RISCVSelectionDAGInfo.h"
#include "RISCVTargetMachine.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/ErrorHandling.h"
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 25-35: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp
using namespace llvm;

#define DEBUG_TYPE "riscv-subtarget"

#define GET_SUBTARGETINFO_TARGET_DESC
#define GET_SUBTARGETINFO_CTOR
#include "RISCVGenSubtargetInfo.inc"

#define GET_RISCV_MACRO_FUSION_PRED_IMPL
#include "RISCVGenMacroFusion.inc"
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 36-47: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp
namespace llvm::RISCVTuneInfoTable {

#define GET_RISCVTuneInfoTable_IMPL
#include "RISCVGenSearchableTables.inc"
} // namespace llvm::RISCVTuneInfoTable

static cl::opt<unsigned> RVVVectorLMULMax(
    "riscv-v-fixed-length-vector-lmul-max",
    cl::desc("The maximum LMUL value to use for fixed length vectors. "
             "Fractional LMUL values are not supported."),
    cl::init(8), cl::Hidden);
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 48-57: Definitions and supporting logic / 定义与支撑逻辑
```cpp
static cl::opt<bool> RISCVDisableUsingConstantPoolForLargeInts(
    "riscv-disable-using-constant-pool-for-large-ints",
    cl::desc("Disable using constant pool for large integers."),
    cl::init(false), cl::Hidden);

static cl::opt<unsigned> RISCVMaxBuildIntsCost(
    "riscv-max-build-ints-cost",
    cl::desc("The maximum cost used for building integers."), cl::init(0),
    cl::Hidden);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 58-69: Definitions and supporting logic / 定义与支撑逻辑
```cpp
static cl::opt<bool> UseAA("riscv-use-aa", cl::init(true),
                           cl::desc("Enable the use of AA during codegen."));

static cl::opt<unsigned> RISCVMinimumJumpTableEntries(
    "riscv-min-jump-table-entries", cl::Hidden,
    cl::desc("Set minimum number of entries to use a jump table on RISCV"));

static cl::opt<bool> UseMIPSLoadStorePairsOpt(
    "use-riscv-mips-load-store-pairs",
    cl::desc("Enable the load/store pair optimization pass"), cl::init(false),
    cl::Hidden);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 70-84: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
static cl::opt<bool> UseMIPSCCMovInsn("use-riscv-mips-ccmov",
                                      cl::desc("Use 'mips.ccmov' instruction"),
                                      cl::init(true), cl::Hidden);

void RISCVSubtarget::anchor() {}

RISCVSubtarget &
RISCVSubtarget::initializeSubtargetDependencies(const Triple &TT, StringRef CPU,
                                                StringRef TuneCPU, StringRef FS,
                                                StringRef ABIName) {
  // Determine default and user-specified characteristics
  bool Is64Bit = TT.isArch64Bit();
  if (CPU.empty() || CPU == "generic")
    CPU = Is64Bit ? "generic-rv64" : "generic-rv32";
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 85-95: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  if (TuneCPU.empty())
    TuneCPU = CPU;
  if (TuneCPU == "generic")
    TuneCPU = Is64Bit ? "generic-rv64" : "generic-rv32";

  TuneInfo = RISCVTuneInfoTable::getRISCVTuneInfo(TuneCPU);
  // If there is no TuneInfo for this CPU, we fail back to generic.
  if (!TuneInfo)
    TuneInfo = RISCVTuneInfoTable::getRISCVTuneInfo("generic");
  assert(TuneInfo && "TuneInfo shouldn't be nullptr!");
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 96-105: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  ParseSubtargetFeatures(CPU, TuneCPU, FS);

  RISCV::updateCZceFeatureImplications(*this);

  // Re-sync the flags.
  HasStdExtZcd = hasFeature(RISCV::FeatureStdExtZcd);
  HasStdExtZcf = hasFeature(RISCV::FeatureStdExtZcf);
  HasStdExtC = hasFeature(RISCV::FeatureStdExtC);
  HasStdExtZce = hasFeature(RISCV::FeatureStdExtZce);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 106-123: Function implementation: RISCVABI::computeTargetABI / 函数实现：RISCVABI::computeTargetABI
```cpp
  TargetABI = RISCVABI::computeTargetABI(TT, getFeatureBits(), ABIName);
  RISCVFeatures::validate(TT, getFeatureBits());
  return *this;
}

RISCVSubtarget::RISCVSubtarget(const Triple &TT, StringRef CPU,
                               StringRef TuneCPU, StringRef FS,
                               StringRef ABIName, unsigned RVVVectorBitsMin,
                               unsigned RVVVectorBitsMax,
                               const TargetMachine &TM)
    : RISCVGenSubtargetInfo(TT, CPU, TuneCPU, FS),
      IsLittleEndian(TT.isLittleEndian()), RVVVectorBitsMin(RVVVectorBitsMin),
      RVVVectorBitsMax(RVVVectorBitsMax),
      FrameLowering(
          initializeSubtargetDependencies(TT, CPU, TuneCPU, FS, ABIName)),
      InstrInfo(*this), TLInfo(TM, *this) {
  TSInfo = std::make_unique<RISCVSelectionDAGInfo>();
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 124-136: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

RISCVSubtarget::~RISCVSubtarget() = default;

const SelectionDAGTargetInfo *RISCVSubtarget::getSelectionDAGInfo() const {
  return TSInfo.get();
}

const InlineAsmLowering *RISCVSubtarget::getInlineAsmLowering() const {
  if (!InlineAsmLoweringInfo)
    InlineAsmLoweringInfo.reset(
        new RISCVInlineAsmLowering(getTargetLowering()));
  return InlineAsmLoweringInfo.get();
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 137-149: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

const CallLowering *RISCVSubtarget::getCallLowering() const {
  if (!CallLoweringInfo)
    CallLoweringInfo.reset(new RISCVCallLowering(*getTargetLowering()));
  return CallLoweringInfo.get();
}

InstructionSelector *RISCVSubtarget::getInstructionSelector() const {
  if (!InstSelector) {
    InstSelector.reset(createRISCVInstructionSelector(
        *static_cast<const RISCVTargetMachine *>(&TLInfo.getTargetMachine()),
        *this, *getRegBankInfo()));
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 150-163: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  return InstSelector.get();
}

const LegalizerInfo *RISCVSubtarget::getLegalizerInfo() const {
  if (!Legalizer)
    Legalizer.reset(new RISCVLegalizerInfo(*this));
  return Legalizer.get();
}

const RISCVRegisterBankInfo *RISCVSubtarget::getRegBankInfo() const {
  if (!RegBankInfo)
    RegBankInfo.reset(new RISCVRegisterBankInfo(getHwMode()));
  return RegBankInfo.get();
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 164-173: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

bool RISCVSubtarget::useConstantPoolForLargeInts() const {
  return !RISCVDisableUsingConstantPoolForLargeInts;
}

// Returns true if VT is a P extension packed SIMD type.
bool RISCVSubtarget::isPExtPackedType(MVT VT) const {
  if (!HasStdExtP)
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 174-185: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // RV32 supports 32-bit and 64-bit vectors. RV64 only support 64-bit vectors.
  if (!is64Bit() && (VT == MVT::v4i8 || VT == MVT::v2i16))
    return true;

  return VT == MVT::v8i8 || VT == MVT::v4i16 || VT == MVT::v2i32;
}

// Returns true if VT is a P extension packed double-wide SIMD type.
bool RISCVSubtarget::isPExtPackedDoubleType(MVT VT) const {
  if (!HasStdExtP || is64Bit())
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 186-198: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  return VT == MVT::v8i8 || VT == MVT::v4i16 || VT == MVT::v2i32;
}

unsigned RISCVSubtarget::getMaxBuildIntsCost() const {
  // Loading integer from constant pool needs two instructions (the reason why
  // the minimum cost is 2): an address calculation instruction and a load
  // instruction. Usually, address calculation and instructions used for
  // building integers (addi, slli, etc.) can be done in one cycle, so here we
  // set the default cost to (LoadLatency + 1) if no threshold is provided.
  return RISCVMaxBuildIntsCost == 0
             ? getSchedModel().LoadLatency + 1
             : std::max<unsigned>(2, RISCVMaxBuildIntsCost);
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 199-209: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

unsigned RISCVSubtarget::getMaxRVVVectorSizeInBits() const {
  assert(hasVInstructions() &&
         "Tried to get vector length without Zve or V extension support!");

  // ZvlLen specifies the minimum required vlen. The upper bound provided by
  // riscv-v-vector-bits-max should be no less than it.
  if (RVVVectorBitsMax != 0 && RVVVectorBitsMax < ZvlLen)
    report_fatal_error("riscv-v-vector-bits-max specified is lower "
                       "than the Zvl*b limitation");
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 210-219: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  return RVVVectorBitsMax;
}

unsigned RISCVSubtarget::getMinRVVVectorSizeInBits() const {
  assert(hasVInstructions() &&
         "Tried to get vector length without Zve or V extension support!");

  if (RVVVectorBitsMin == -1U)
    return ZvlLen;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 220-236: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // ZvlLen specifies the minimum required vlen. The lower bound provided by
  // riscv-v-vector-bits-min should be no less than it.
  if (RVVVectorBitsMin != 0 && RVVVectorBitsMin < ZvlLen)
    report_fatal_error("riscv-v-vector-bits-min specified is lower "
                       "than the Zvl*b limitation");

  return RVVVectorBitsMin;
}

unsigned RISCVSubtarget::getMaxLMULForFixedLengthVectors() const {
  assert(hasVInstructions() &&
         "Tried to get vector length without Zve or V extension support!");
  assert(RVVVectorLMULMax <= 8 &&
         llvm::has_single_bit<uint32_t>(RVVVectorLMULMax) &&
         "V extension requires a LMUL to be at most 8 and a power of 2!");
  return llvm::bit_floor(std::clamp<unsigned>(RVVVectorLMULMax, 1, 8));
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 237-247: Function implementation: RISCVSubtarget::useRVVForFixedLengthVectors / 函数实现：RISCVSubtarget::useRVVForFixedLengthVectors
```cpp

bool RISCVSubtarget::useRVVForFixedLengthVectors() const {
  return hasVInstructions() &&
         getMinRVVVectorSizeInBits() >= RISCV::RVVBitsPerBlock;
}

bool RISCVSubtarget::enableSubRegLiveness() const { return true; }

bool RISCVSubtarget::enableMachinePipeliner() const {
  return getSchedModel().hasInstrSchedModel();
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 248-257: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

void RISCVSubtarget::mirFileLoaded(MachineFunction &MF) const {
  // We usually compute max call frame size after ISel. Do the computation now
  // if the .mir file didn't specify it. Note that this will probably give you
  // bogus values after PEI has eliminated the callframe setup/destroy pseudo
  // instructions, specify explicitly if you need it to be correct.
  MachineFrameInfo &MFI = MF.getFrameInfo();
  if (!MFI.isMaxCallFrameSizeComputed())
    MFI.computeMaxCallFrameSize(MF);
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 258-267: Function implementation: RISCVSubtarget::useAA / 函数实现：RISCVSubtarget::useAA
```cpp

  /// Enable use of alias analysis during code generation (during MI
  /// scheduling, DAGCombine, etc.).
bool RISCVSubtarget::useAA() const { return UseAA; }

unsigned RISCVSubtarget::getMinimumJumpTableEntries() const {
  return RISCVMinimumJumpTableEntries.getNumOccurrences() > 0
             ? RISCVMinimumJumpTableEntries
             : TuneInfo->MinimumJumpTableEntries;
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 268-279: Function implementation: RISCVSubtarget::overrideSchedPolicy / 函数实现：RISCVSubtarget::overrideSchedPolicy
```cpp

void RISCVSubtarget::overrideSchedPolicy(MachineSchedPolicy &Policy,
                                         const SchedRegion &Region) const {
  // Do bidirectional scheduling since it provides a more balanced scheduling
  // leading to better performance. This will increase compile time.
  Policy.OnlyTopDown = false;
  Policy.OnlyBottomUp = false;

  // Disabling the latency heuristic can reduce the number of spills/reloads but
  // will cause some regressions on some cores.
  Policy.DisableLatencyHeuristic = DisableLatencySchedHeuristic;
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 280-297: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Spilling is generally expensive on all RISC-V cores, so always enable
  // register-pressure tracking. This will increase compile time.
  Policy.ShouldTrackPressure = true;
}

void RISCVSubtarget::overridePostRASchedPolicy(
    MachineSchedPolicy &Policy, const SchedRegion &Region) const {
  MISched::Direction PostRASchedDirection = getPostRASchedDirection();
  if (PostRASchedDirection == MISched::TopDown) {
    Policy.OnlyTopDown = true;
    Policy.OnlyBottomUp = false;
  } else if (PostRASchedDirection == MISched::BottomUp) {
    Policy.OnlyTopDown = false;
    Policy.OnlyBottomUp = true;
  } else if (PostRASchedDirection == MISched::Bidirectional) {
    Policy.OnlyTopDown = false;
    Policy.OnlyBottomUp = false;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 298-306: Function implementation: RISCVSubtarget::useMIPSLoadStorePairs / 函数实现：RISCVSubtarget::useMIPSLoadStorePairs
```cpp
}

bool RISCVSubtarget::useMIPSLoadStorePairs() const {
  return UseMIPSLoadStorePairsOpt && HasVendorXMIPSLSP;
}

bool RISCVSubtarget::useMIPSCCMovInsn() const {
  return UseMIPSCCMovInsn && HasVendorXMIPSCMov;
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

## Key Concepts / 关键概念
- **CPU and feature modeling** / **CPU 与特性建模**

## Dependencies / 依赖关系
- `RISCVSubtarget.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `GISel/RISCVCallLowering.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `GISel/RISCVInlineAsmLowering.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `GISel/RISCVLegalizerInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCV.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVFrameLowering.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVSelectionDAGInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVTargetMachine.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineFrameInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/TargetRegistry.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Support/ErrorHandling.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVGenSubtargetInfo.inc` — Directly referenced by this file. / 该文件直接引用的依赖。
