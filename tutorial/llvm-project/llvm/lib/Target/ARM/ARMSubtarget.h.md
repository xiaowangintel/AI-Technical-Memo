# ARMSubtarget.h — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMSubtarget.h`
- Repository: `llvm-project`
- Purpose (EN): This file declares the ARM specific subclass of TargetSubtargetInfo.
- 用途 (CN): 声明 ARM 后端中的 `ARMSubtarget`，并提供与子目标特性选择与 CPU 特定行为相关的接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- ARMSubtarget.h - Define Subtarget for the ARM ----------*- C++ -*--===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file declares the ARM specific subclass of TargetSubtargetInfo.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-14
```cpp
#ifndef LLVM_LIB_TARGET_ARM_ARMSUBTARGET_H
#define LLVM_LIB_TARGET_ARM_ARMSUBTARGET_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 16-30
```cpp
#include "ARMBaseInstrInfo.h"
#include "ARMBaseRegisterInfo.h"
#include "ARMConstantPoolValue.h"
#include "ARMFrameLowering.h"
#include "ARMISelLowering.h"
#include "ARMMachineFunctionInfo.h"
#include "ARMSelectionDAGInfo.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/CodeGen/GlobalISel/CallLowering.h"
#include "llvm/CodeGen/GlobalISel/InstructionSelector.h"
#include "llvm/CodeGen/GlobalISel/LegalizerInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/RegisterBankInfo.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/MC/MCInstrItineraries.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 31-37
```cpp
#include "llvm/MC/MCSchedule.h"
#include "llvm/Target/TargetMachine.h"
#include "llvm/Target/TargetOptions.h"
#include "llvm/TargetParser/Triple.h"
#include <bitset>
#include <memory>
#include <string>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 39-40
```cpp
#define GET_SUBTARGETINFO_HEADER
#include "ARMGenSubtargetInfo.inc"
```
- EN: Connects this file to TableGen-generated declarations so target-specific enums and helper tables become available to C++ code.
- CN: 这里把文件连接到 TableGen 生成的声明，使目标相关枚举和辅助表能够在 C++ 代码中使用。

### Lines 42-42
```cpp
namespace llvm {
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 44-46
```cpp
class ARMBaseTargetMachine;
class GlobalValue;
class StringRef;
```
- EN: Declares `ARMBaseTargetMachine`, packaging target-specific state and APIs around `ARMSubtarget`.
- CN: 这里声明 `ARMBaseTargetMachine`，把与 `ARMSubtarget` 相关的目标特定状态和 API 组织在一起。

### Lines 48-57
```cpp
class ARMSubtarget : public ARMGenSubtargetInfo {
protected:
  enum ARMProcFamilyEnum {
    Others,
#define ARM_PROCESSOR_FAMILY(ENUM) ENUM,
#include "llvm/TargetParser/ARMTargetParserDef.inc"
#undef ARM_PROCESSOR_FAMILY
  };
  enum ARMProcClassEnum {
    None,
```
- EN: Declares `ARMSubtarget`, packaging target-specific state and APIs around `ARMSubtarget`.
- CN: 这里声明 `ARMSubtarget`，把与 `ARMSubtarget` 相关的目标特定状态和 API 组织在一起。

### Lines 59-67
```cpp
    AClass,
    MClass,
    RClass
  };
  enum ARMArchEnum {
#define ARM_ARCHITECTURE(ENUM) ENUM,
#include "llvm/TargetParser/ARMTargetParserDef.inc"
#undef ARM_ARCHITECTURE
  };
```
- EN: Defines enumeration `ARMArchEnum` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `ARMArchEnum`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 69-82
```cpp
public:
  /// What kind of timing do load multiple/store multiple instructions have.
  enum ARMLdStMultipleTiming {
    /// Can load/store 2 registers/cycle.
    DoubleIssue,
    /// Can load/store 2 registers/cycle, but needs an extra cycle if the access
    /// is not 64-bit aligned.
    DoubleIssueCheckUnalignedAccess,
    /// Can load/store 1 register/cycle.
    SingleIssue,
    /// Can load/store 1 register/cycle, but needs an extra cycle for address
    /// computation and potentially also for register writeback.
    SingleIssuePlusExtras,
  };
```
- EN: Defines enumeration `ARMLdStMultipleTiming` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `ARMLdStMultipleTiming`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 84-90
```cpp
  /// How the push and pop instructions of callee saved general-purpose
  /// registers should be split.
  enum PushPopSplitVariation {
    /// All GPRs can be pushed in a single instruction.
    /// push {r0-r12, lr}
    /// vpush {d8-d15}
    NoSplit,
```
- EN: Defines enumeration `PushPopSplitVariation` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `PushPopSplitVariation`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 92-102
```cpp
    /// R7 and LR must be adjacent, because R7 is the frame pointer, and must
    /// point to a frame record consisting of the previous frame pointer and the
    /// return address.
    /// push {r0-r7, lr}
    /// push {r8-r12}
    /// vpush {d8-d15}
    /// Note that Thumb1 changes this layout when the frame pointer is R11,
    /// using a longer sequence of instructions because R11 can't be used by a
    /// Thumb1 push instruction. This doesn't currently have a separate enum
    /// value, and is handled entriely within Thumb1FrameLowering::emitPrologue.
    SplitR7,
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 104-111
```cpp
    /// When the stack frame size is not known (because of variable-sized
    /// objects or realignment), Windows SEH requires the callee-saved registers
    /// to be stored in three regions, with R11 and LR below the floating-point
    /// registers.
    /// push {r0-r10, r12}
    /// vpush {d8-d15}
    /// push {r11, lr}
    SplitR11WindowsSEH,
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 113-124
```cpp
    /// When generating AAPCS-compilant frame chains, R11 is the frame pointer,
    /// and must be pushed adjacent to the return address (LR). Normally this
    /// isn't a problem, because the only register between them is r12, which is
    /// the intra-procedure-call scratch register, so doesn't need to be saved.
    /// However, when PACBTI is in use, r12 contains the authentication code, so
    /// does need to be saved. This means that we need a separate push for R11
    /// and LR.
    /// push {r0-r10, r12}
    /// push {r11, lr}
    /// vpush {d8-d15}
    SplitR11AAPCSSignRA,
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 126-130
```cpp
protected:
// Bool members corresponding to the SubtargetFeatures defined in tablegen
#define GET_SUBTARGETINFO_MACRO(ATTRIBUTE, DEFAULT, GETTER)                    \
  bool ATTRIBUTE = DEFAULT;
#include "ARMGenSubtargetInfo.inc"
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 132-133
```cpp
  /// ARMProcFamily - ARM processor family: Cortex-A8, Cortex-A9, and others.
  ARMProcFamilyEnum ARMProcFamily = Others;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 135-136
```cpp
  /// ARMProcClass - ARM processor class: None, AClass, RClass or MClass.
  ARMProcClassEnum ARMProcClass = None;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 138-139
```cpp
  /// ARMArch - ARM architecture
  ARMArchEnum ARMArch = ARMv4t;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 141-143
```cpp
  /// UseMulOps - True if non-microcoded fused integer multiply-add and
  /// multiply-subtract instructions should be used.
  bool UseMulOps = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 145-148
```cpp
  /// SupportsTailCall - True if the OS supports tail call. The dynamic linker
  /// must be able to synthesize call stubs for interworking between ARM and
  /// Thumb.
  bool SupportsTailCall = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 150-152
```cpp
  /// RestrictIT - If true, the subtarget disallows generation of complex IT
  ///  blocks.
  bool RestrictIT = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 154-156
```cpp
  /// stackAlignment - The minimum alignment known to hold of the stack frame on
  /// entry to the function and which must be maintained by every function.
  Align stackAlignment = Align(4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 158-159
```cpp
  /// CPUString - String name of used CPU.
  std::string CPUString;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 161-161
```cpp
  unsigned MaxInterleaveFactor = 1;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 163-164
```cpp
  /// Clearance before partial register updates (in number of instructions)
  unsigned PartialUpdateClearance = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 166-168
```cpp
  /// What kind of timing do load multiple/store multiple have (double issue,
  /// single issue etc).
  ARMLdStMultipleTiming LdStMultipleTiming = SingleIssue;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 170-172
```cpp
  /// The adjustment that we need to apply to get the operand latency from the
  /// operand cycle returned by the itinerary data for pre-ISel operands.
  int PreISelOperandLatencyAdjustment = 2;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 174-175
```cpp
  /// What alignment is preferred for loop bodies and functions, in log2(bytes).
  unsigned PreferBranchLogAlignment = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 177-180
```cpp
  /// The cost factor for MVE instructions, representing the multiple beats an
  // instruction can take. The default is 2, (set in initSubtargetFeatures so
  // that we can use subtarget features less than 2).
  unsigned MVEVectorCostFactor = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 182-184
```cpp
  /// OptMinSize - True if we're optimising for minimum code size, equal to
  /// the function attribute.
  bool OptMinSize = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 186-187
```cpp
  /// IsLittle - The target is Little Endian
  bool IsLittle;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 189-193
```cpp
  /// DM - Denormal mode
  /// NEON and VFP RunFast mode are not IEEE 754 compliant,
  /// use this field to determine whether to generate NEON/VFP
  /// instructions in related function.
  DenormalMode DM;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 195-196
```cpp
  /// TargetTriple - What processor and OS we're targeting.
  Triple TargetTriple;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 198-199
```cpp
  /// SchedModel - Processor specific instruction costs.
  MCSchedModel SchedModel;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 201-202
```cpp
  /// Selected instruction itineraries (one entry per itinerary class.)
  InstrItineraryData InstrItins;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 204-205
```cpp
  /// Options passed via command line that could influence the target
  const TargetOptions &Options;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 207-207
```cpp
  const ARMBaseTargetMachine &TM;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 209-215
```cpp
public:
  /// This constructor initializes the data members to match that
  /// of the specified triple.
  ///
  ARMSubtarget(const Triple &TT, const std::string &CPU, const std::string &FS,
               const ARMBaseTargetMachine &TM, bool IsLittle,
               bool MinSize = false, DenormalMode DM = DenormalMode::getIEEE());
```
- EN: Declares `ARMSubtarget`, a target-specific routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ARMSubtarget`，它是一个围绕目标机器策略展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 217-221
```cpp
  /// getMaxInlineSizeThreshold - Returns the maximum memset / memcpy size
  /// that still makes it profitable to inline the call.
  unsigned getMaxInlineSizeThreshold() const {
    return 64;
  }
```
- EN: Implements `getMaxInlineSizeThreshold`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMaxInlineSizeThreshold`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 223-227
```cpp
  /// getMaxMemcpyTPInlineSizeThreshold - Returns the maximum size
  /// that still makes it profitable to inline a llvm.memcpy as a Tail
  /// Predicated loop.
  /// This threshold should only be used for constant size inputs.
  unsigned getMaxMemcpyTPInlineSizeThreshold() const { return 128; }
```
- EN: Implements `getMaxMemcpyTPInlineSizeThreshold`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMaxMemcpyTPInlineSizeThreshold`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 229-231
```cpp
  /// ParseSubtargetFeatures - Parses features string setting specified
  /// subtarget options.  Definition of function is auto generated by tblgen.
  void ParseSubtargetFeatures(StringRef CPU, StringRef TuneCPU, StringRef FS);
```
- EN: Declares `ParseSubtargetFeatures`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ParseSubtargetFeatures`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 233-235
```cpp
  /// initializeSubtargetDependencies - Initializes using a CPU and feature string
  /// so that we can use initializer lists for subtarget initialization.
  ARMSubtarget &initializeSubtargetDependencies(StringRef CPU, StringRef FS);
```
- EN: Declares `initializeSubtargetDependencies`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `initializeSubtargetDependencies`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 237-239
```cpp
  const ARMSelectionDAGInfo *getSelectionDAGInfo() const override {
    return &TSInfo;
  }
```
- EN: Implements `getSelectionDAGInfo`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getSelectionDAGInfo`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 241-243
```cpp
  const ARMBaseInstrInfo *getInstrInfo() const override {
    return InstrInfo.get();
  }
```
- EN: Implements `getInstrInfo`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getInstrInfo`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 245-247
```cpp
  const ARMTargetLowering *getTargetLowering() const override {
    return &TLInfo;
  }
```
- EN: Implements `getTargetLowering`, a lowering routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getTargetLowering`，它是一个围绕目标相关状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 249-251
```cpp
  const ARMFrameLowering *getFrameLowering() const override {
    return FrameLowering.get();
  }
```
- EN: Implements `getFrameLowering`, a lowering routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getFrameLowering`，它是一个围绕目标相关状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 253-255
```cpp
  const ARMBaseRegisterInfo *getRegisterInfo() const override {
    return &InstrInfo->getRegisterInfo();
  }
```
- EN: Implements `getRegisterInfo`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getRegisterInfo`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 257-261
```cpp
  const CallLowering *getCallLowering() const override;
  InstructionSelector *getInstructionSelector() const override;
  const LegalizerInfo *getLegalizerInfo() const override;
  const RegisterBankInfo *getRegBankInfo() const override;
  void initLibcallLoweringInfo(LibcallLoweringInfo &Info) const override;
```
- EN: Declares `getCallLowering`, a lowering routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getCallLowering`，它是一个围绕寄存器管理展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 263-269
```cpp
private:
  ARMSelectionDAGInfo TSInfo;
  // Either Thumb1FrameLowering or ARMFrameLowering.
  std::unique_ptr<ARMFrameLowering> FrameLowering;
  // Either Thumb1InstrInfo or Thumb2InstrInfo.
  std::unique_ptr<ARMBaseInstrInfo> InstrInfo;
  ARMTargetLowering   TLInfo;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 271-275
```cpp
  /// GlobalISel related APIs.
  std::unique_ptr<CallLowering> CallLoweringInfo;
  std::unique_ptr<InstructionSelector> InstSelector;
  std::unique_ptr<LegalizerInfo> Legalizer;
  std::unique_ptr<RegisterBankInfo> RegBankInfo;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 277-278
```cpp
  void initSubtargetFeatures(StringRef CPU, StringRef FS);
  ARMFrameLowering *initializeFrameLowering(StringRef CPU, StringRef FS);
```
- EN: Declares `initSubtargetFeatures`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `initSubtargetFeatures`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 280-285
```cpp
  std::bitset<8> CoprocCDE = {};
public:
// Getters for SubtargetFeatures defined in tablegen
#define GET_SUBTARGETINFO_MACRO(ATTRIBUTE, DEFAULT, GETTER)                    \
  bool GETTER() const { return ATTRIBUTE; }
#include "ARMGenSubtargetInfo.inc"
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 287-301
```cpp
  /// @{
  /// These functions are obsolete, please consider adding subtarget features
  /// or properties instead of calling them.
  bool isCortexA5() const { return ARMProcFamily == CortexA5; }
  bool isCortexA7() const { return ARMProcFamily == CortexA7; }
  bool isCortexA8() const { return ARMProcFamily == CortexA8; }
  bool isCortexA9() const { return ARMProcFamily == CortexA9; }
  bool isCortexA15() const { return ARMProcFamily == CortexA15; }
  bool isSwift()    const { return ARMProcFamily == Swift; }
  bool isCortexM3() const { return ARMProcFamily == CortexM3; }
  bool isCortexM55() const { return ARMProcFamily == CortexM55; }
  bool isCortexM7() const { return ARMProcFamily == CortexM7; }
  bool isCortexM85() const { return ARMProcFamily == CortexM85; }
  bool isLikeA9() const { return isCortexA9() || isCortexA15() || isKrait(); }
  bool isCortexR5() const { return ARMProcFamily == CortexR5; }
```
- EN: Implements `isCortexA5`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isCortexA5`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 302-303
```cpp
  bool isKrait() const { return ARMProcFamily == Krait; }
  /// @}
```
- EN: Implements `isKrait`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isKrait`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 305-305
```cpp
  bool hasARMOps() const { return !NoARM; }
```
- EN: Implements `hasARMOps`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasARMOps`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 307-309
```cpp
  bool useNEONForSinglePrecisionFP() const {
    return hasNEON() && hasNEONForFP();
  }
```
- EN: Implements `useNEONForSinglePrecisionFP`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `useNEONForSinglePrecisionFP`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 311-314
```cpp
  bool hasVFP2Base() const { return hasVFPv2SP(); }
  bool hasVFP3Base() const { return hasVFPv3D16SP(); }
  bool hasVFP4Base() const { return hasVFPv4D16SP(); }
  bool hasFPARMv8Base() const { return hasFPARMv8D16SP(); }
```
- EN: Implements `hasVFP2Base`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasVFP2Base`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 316-318
```cpp
  bool hasAnyDataBarrier() const {
    return HasDataBarrier || (hasV6Ops() && !isThumb());
  }
```
- EN: Implements `hasAnyDataBarrier`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasAnyDataBarrier`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 320-332
```cpp
  bool useMulOps() const { return UseMulOps; }
  bool useFPVMLx() const { return !SlowFPVMLx; }
  bool useFPVFMx() const {
    return !isTargetDarwin() && hasVFP4Base() && !SlowFPVFMx;
  }
  bool useFPVFMx16() const { return useFPVFMx() && hasFullFP16(); }
  bool useFPVFMx64() const { return useFPVFMx() && hasFP64(); }
  bool hasBaseDSP() const {
    if (isThumb())
      return hasThumb2() && hasDSP();
    else
      return hasV5TEOps();
  }
```
- EN: Implements `useMulOps`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `useMulOps`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 334-335
```cpp
  /// Return true if the CPU supports any kind of instruction fusion.
  bool hasFusion() const { return hasFuseAES() || hasFuseLiterals(); }
```
- EN: Implements `hasFusion`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasFusion`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 337-337
```cpp
  const Triple &getTargetTriple() const { return TargetTriple; }
```
- EN: Implements `getTargetTriple`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getTargetTriple`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 339-350
```cpp
  /// @{
  /// These properties are per-module, please use the TargetMachine
  /// TargetTriple.
  bool isTargetDarwin() const { return TargetTriple.isOSDarwin(); }
  bool isTargetIOS() const { return TargetTriple.isiOS(); }
  bool isTargetWatchOS() const { return TargetTriple.isWatchOS(); }
  bool isTargetWatchABI() const { return TargetTriple.isWatchABI(); }
  bool isTargetDriverKit() const { return TargetTriple.isDriverKit(); }
  bool isTargetFuchsia() const { return TargetTriple.isOSFuchsia(); }
  bool isTargetLinux() const { return TargetTriple.isOSLinux(); }
  bool isTargetNetBSD() const { return TargetTriple.isOSNetBSD(); }
  bool isTargetWindows() const { return TargetTriple.isOSWindows(); }
```
- EN: Implements `isTargetDarwin`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isTargetDarwin`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 352-354
```cpp
  bool isTargetCOFF() const { return TargetTriple.isOSBinFormatCOFF(); }
  bool isTargetELF() const { return TargetTriple.isOSBinFormatELF(); }
  bool isTargetMachO() const { return TargetTriple.isOSBinFormatMachO(); }
```
- EN: Implements `isTargetCOFF`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isTargetCOFF`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 356-356
```cpp
  bool isTargetAEABI() const { return TargetTriple.isTargetAEABI(); }
```
- EN: Implements `isTargetAEABI`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isTargetAEABI`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 358-358
```cpp
  bool isTargetGNUAEABI() const { return TargetTriple.isTargetGNUAEABI(); }
```
- EN: Implements `isTargetGNUAEABI`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isTargetGNUAEABI`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 360-360
```cpp
  bool isTargetMuslAEABI() const { return TargetTriple.isTargetMuslAEABI(); }
```
- EN: Implements `isTargetMuslAEABI`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isTargetMuslAEABI`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 362-367
```cpp
  // ARM Targets that support EHABI exception handling standard
  // Darwin uses SjLj. Other targets might need more checks.
  bool isTargetEHABICompatible() const {
    return TargetTriple.isTargetEHABICompatible();
  }
  /// @}
```
- EN: Implements `isTargetEHABICompatible`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isTargetEHABICompatible`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 369-371
```cpp
  bool isReadTPSoft() const {
    return !(isReadTPTPIDRURW() || isReadTPTPIDRURO() || isReadTPTPIDRPRW());
  }
```
- EN: Implements `isReadTPSoft`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isReadTPSoft`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 373-373
```cpp
  bool isTargetAndroid() const { return TargetTriple.isAndroid(); }
```
- EN: Implements `isTargetAndroid`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isTargetAndroid`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 375-375
```cpp
  bool isXRaySupported() const override;
```
- EN: Declares `isXRaySupported`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isXRaySupported`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 377-378
```cpp
  bool isROPI() const;
  bool isRWPI() const;
```
- EN: Declares `isROPI`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isROPI`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 380-387
```cpp
  bool useMachineScheduler() const { return UseMISched; }
  bool useMachinePipeliner() const { return UseMIPipeliner; }
  bool hasMinSize() const { return OptMinSize; }
  bool isThumb1Only() const { return isThumb() && !hasThumb2(); }
  bool isThumb2() const { return isThumb() && hasThumb2(); }
  bool isMClass() const { return ARMProcClass == MClass; }
  bool isRClass() const { return ARMProcClass == RClass; }
  bool isAClass() const { return ARMProcClass == AClass; }
```
- EN: Implements `useMachineScheduler`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `useMachineScheduler`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 389-391
```cpp
  bool isR9Reserved() const {
    return isTargetMachO() ? (ReserveR9 || !HasV6Ops) : ReserveR9;
  }
```
- EN: Implements `isR9Reserved`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isR9Reserved`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 393-398
```cpp
  MCPhysReg getFramePointerReg() const {
    if (isTargetDarwin() ||
        (!isTargetWindows() && isThumb() && !createAAPCSFrameChain()))
      return ARM::R7;
    return ARM::R11;
  }
```
- EN: Implements `getFramePointerReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getFramePointerReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 400-401
```cpp
  enum PushPopSplitVariation
  getPushPopSplitVariation(const MachineFunction &MF) const;
```
- EN: Defines enumeration `PushPopSplitVariation` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `PushPopSplitVariation`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 403-403
```cpp
  bool useStride4VFPs() const;
```
- EN: Declares `useStride4VFPs`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `useStride4VFPs`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 405-405
```cpp
  bool useMovt() const;
```
- EN: Declares `useMovt`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `useMovt`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 407-407
```cpp
  bool supportsTailCall() const { return SupportsTailCall; }
```
- EN: Implements `supportsTailCall`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `supportsTailCall`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 409-409
```cpp
  bool allowsUnalignedMem() const { return !StrictAlign; }
```
- EN: Implements `allowsUnalignedMem`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `allowsUnalignedMem`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 411-411
```cpp
  bool restrictIT() const { return RestrictIT; }
```
- EN: Implements `restrictIT`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `restrictIT`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 413-413
```cpp
  const std::string & getCPUString() const { return CPUString; }
```
- EN: Implements `getCPUString`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getCPUString`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 415-415
```cpp
  bool isLittle() const { return IsLittle; }
```
- EN: Implements `isLittle`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isLittle`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 417-417
```cpp
  unsigned getMispredictionPenalty() const;
```
- EN: Declares `getMispredictionPenalty`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getMispredictionPenalty`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 419-420
```cpp
  /// Returns true if machine scheduler should be enabled.
  bool enableMachineScheduler() const override;
```
- EN: Declares `enableMachineScheduler`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `enableMachineScheduler`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 422-424
```cpp
  /// Returns true if machine pipeliner should be enabled.
  bool enableMachinePipeliner() const override;
  bool useDFAforSMS() const override;
```
- EN: Declares `enableMachinePipeliner`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `enableMachinePipeliner`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 426-427
```cpp
  /// True for some subtargets at > -O0.
  bool enablePostRAScheduler() const override;
```
- EN: Declares `enablePostRAScheduler`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `enablePostRAScheduler`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 429-430
```cpp
  /// True for some subtargets at > -O0.
  bool enablePostRAMachineScheduler() const override;
```
- EN: Declares `enablePostRAMachineScheduler`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `enablePostRAMachineScheduler`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 432-433
```cpp
  /// Check whether this subtarget wants to use subregister liveness.
  bool enableSubRegLiveness() const override;
```
- EN: Declares `enableSubRegLiveness`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `enableSubRegLiveness`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 435-437
```cpp
  /// Enable use of alias analysis during code generation (during MI
  /// scheduling, DAGCombine, etc.).
  bool useAA() const override { return true; }
```
- EN: Implements `useAA`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `useAA`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 439-443
```cpp
  /// getInstrItins - Return the instruction itineraries based on subtarget
  /// selection.
  const InstrItineraryData *getInstrItineraryData() const override {
    return &InstrItins;
  }
```
- EN: Implements `getInstrItineraryData`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getInstrItineraryData`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 445-448
```cpp
  /// getStackAlignment - Returns the minimum alignment known to hold of the
  /// stack frame on entry to the function and which must be maintained by every
  /// function for this subtarget.
  Align getStackAlignment() const { return stackAlignment; }
```
- EN: Implements `getStackAlignment`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getStackAlignment`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 450-453
```cpp
  // Returns the required alignment for LDRD/STRD instructions
  Align getDualLoadStoreAlignment() const {
    return Align(hasV7Ops() || allowsUnalignedMem() ? 4 : 8);
  }
```
- EN: Implements `getDualLoadStoreAlignment`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getDualLoadStoreAlignment`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 455-455
```cpp
  unsigned getMaxInterleaveFactor() const { return MaxInterleaveFactor; }
```
- EN: Implements `getMaxInterleaveFactor`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMaxInterleaveFactor`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 457-457
```cpp
  unsigned getPartialUpdateClearance() const { return PartialUpdateClearance; }
```
- EN: Implements `getPartialUpdateClearance`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPartialUpdateClearance`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 459-461
```cpp
  ARMLdStMultipleTiming getLdStMultipleTiming() const {
    return LdStMultipleTiming;
  }
```
- EN: Implements `getLdStMultipleTiming`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getLdStMultipleTiming`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 463-465
```cpp
  int getPreISelOperandLatencyAdjustment() const {
    return PreISelOperandLatencyAdjustment;
  }
```
- EN: Implements `getPreISelOperandLatencyAdjustment`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPreISelOperandLatencyAdjustment`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 467-468
```cpp
  /// True if the GV will be accessed via an indirect symbol.
  bool isGVIndirectSymbol(const GlobalValue *GV) const;
```
- EN: Declares `isGVIndirectSymbol`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isGVIndirectSymbol`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 470-471
```cpp
  /// Returns the constant pool modifier needed to access the GV.
  bool isGVInGOT(const GlobalValue *GV) const;
```
- EN: Declares `isGVInGOT`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isGVInGOT`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 473-474
```cpp
  /// True if fast-isel is used.
  bool useFastISel() const;
```
- EN: Declares `useFastISel`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `useFastISel`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 476-485
```cpp
  /// Returns the correct return opcode for the current feature set.
  /// Use BX if available to allow mixing thumb/arm code, but fall back
  /// to plain mov pc,lr on ARMv4.
  unsigned getReturnOpcode() const {
    if (isThumb())
      return ARM::tBX_RET;
    if (hasV4TOps())
      return ARM::BX_RET;
    return ARM::MOVPCLR;
  }
```
- EN: Implements `getReturnOpcode`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getReturnOpcode`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 487-492
```cpp
  /// Allow movt+movw for PIC global address calculation.
  /// ELF does not have GOT relocations for movt+movw.
  /// ROPI does not use GOT.
  bool allowPositionIndependentMovt() const {
    return isROPI() || !isTargetELF();
  }
```
- EN: Implements `allowPositionIndependentMovt`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `allowPositionIndependentMovt`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 494-496
```cpp
  unsigned getPreferBranchLogAlignment() const {
    return PreferBranchLogAlignment;
  }
```
- EN: Implements `getPreferBranchLogAlignment`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPreferBranchLogAlignment`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 498-503
```cpp
  unsigned
  getMVEVectorCostFactor(TargetTransformInfo::TargetCostKind CostKind) const {
    if (CostKind == TargetTransformInfo::TCK_CodeSize)
      return 1;
    return MVEVectorCostFactor;
  }
```
- EN: Implements `getMVEVectorCostFactor`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMVEVectorCostFactor`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 505-508
```cpp
  bool ignoreCSRForAllocationOrder(const MachineFunction &MF,
                                   MCRegister PhysReg) const override;
  unsigned getGPRAllocationOrder(const MachineFunction &MF) const;
};
```
- EN: Declares `ignoreCSRForAllocationOrder`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ignoreCSRForAllocationOrder`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 510-510
```cpp
} // end namespace llvm
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 512-512
```cpp
#endif  // LLVM_LIB_TARGET_ARM_ARMSUBTARGET_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

## Key Concepts / 关键概念

- EN: Primary role: subtarget feature selection and CPU-specific behavior.
  - CN: 核心职责：子目标特性选择与 CPU 特定行为。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: Stack-frame and spill/reload concerns are reflected in the API or implementation.
  - CN: API 或实现中体现了栈帧以及溢出/重载相关问题。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARMBaseInstrInfo.h`, `ARMBaseRegisterInfo.h`, `ARMConstantPoolValue.h`, `ARMFrameLowering.h`, `ARMISelLowering.h`, `ARMMachineFunctionInfo.h`, `ARMSelectionDAGInfo.h`, `ARMGenSubtargetInfo.inc` ... (+2 more).
  - CN: 后端本地头文件：`ARMBaseInstrInfo.h`, `ARMBaseRegisterInfo.h`, `ARMConstantPoolValue.h`, `ARMFrameLowering.h`, `ARMISelLowering.h`, `ARMMachineFunctionInfo.h`, `ARMSelectionDAGInfo.h`, `ARMGenSubtargetInfo.inc` ... (+2 more)。
- EN: LLVM infrastructure headers: `llvm/Analysis/TargetTransformInfo.h`, `llvm/CodeGen/GlobalISel/CallLowering.h`, `llvm/CodeGen/GlobalISel/InstructionSelector.h`, `llvm/CodeGen/GlobalISel/LegalizerInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/RegisterBankInfo.h`, `llvm/CodeGen/TargetSubtargetInfo.h`, `llvm/MC/MCInstrItineraries.h` ... (+6 more).
  - CN: LLVM 基础设施头文件：`llvm/Analysis/TargetTransformInfo.h`, `llvm/CodeGen/GlobalISel/CallLowering.h`, `llvm/CodeGen/GlobalISel/InstructionSelector.h`, `llvm/CodeGen/GlobalISel/LegalizerInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/RegisterBankInfo.h`, `llvm/CodeGen/TargetSubtargetInfo.h`, `llvm/MC/MCInstrItineraries.h` ... (+6 more)。
- EN: Standard/system headers: `bitset`, `memory`, `string`.
  - CN: 标准库/系统头文件：`bitset`, `memory`, `string`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
