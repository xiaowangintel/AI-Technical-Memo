# PPCSubtarget.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCSubtarget.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCSubtarget.h - Define Subtarget for the PPC.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCSubtarget.h`，主要负责 PowerPC 后端的子目标特性建模。 文件内容以接口、类型声明和协作关系说明为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCSubtarget.h - Define Subtarget for the PPC ----------*- C++ -*--===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. Subtarget feature gating influences the behavior here.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 子目标特性裁剪会影响这里的行为。

### Lines 7-10

```cpp
//===----------------------------------------------------------------------===//
//
// This file declares the PowerPC specific subclass of TargetSubtargetInfo.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file declares the PowerPC specific subclass of TargetSubtargetInfo.". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file declares the PowerPC specific subclass of TargetSubtargetInfo.”。 子目标特性裁剪会影响这里的行为。

### Lines 11-17

```cpp
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_POWERPC_PPCSUBTARGET_H
#define LLVM_LIB_TARGET_POWERPC_PPCSUBTARGET_H

#include "PPCFrameLowering.h"
#include "PPCISelLowering.h"
```
- **EN**: Pulls in direct dependencies required by this subtarget feature modeling, so later declarations can reuse LLVM infrastructure and target-specific helpers. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这一段引入该子目标特性建模所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 该区间与栈帧布局或栈访问相关。

### Lines 18-24

```cpp
#include "PPCInstrInfo.h"
#include "llvm/CodeGen/GlobalISel/CallLowering.h"
#include "llvm/CodeGen/GlobalISel/LegalizerInfo.h"
#include "llvm/CodeGen/RegisterBankInfo.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/MC/MCInstrItineraries.h"
```
- **EN**: Pulls in direct dependencies required by this subtarget feature modeling, so later declarations can reuse LLVM infrastructure and target-specific helpers. This range participates in the GlobalISel pipeline.
- **CN**: 这一段引入该子目标特性建模所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段参与 GlobalISel 流水线。

### Lines 25-32

```cpp
#include "llvm/TargetParser/Triple.h"

#define GET_SUBTARGETINFO_HEADER
#include "PPCGenSubtargetInfo.inc"

// GCC #defines PPC on Linux but we use it as our namespace name
#undef PPC
```
- **EN**: Pulls in direct dependencies required by this subtarget feature modeling, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该子目标特性建模所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 33-58

```cpp
namespace llvm {
class SelectionDAGTargetInfo;
class StringRef;

namespace PPC {
  // -m directive values.
enum {
  DIR_NONE,
  DIR_32,
  DIR_440,
  DIR_601,
  DIR_602,
  DIR_603,
  DIR_7400,
  DIR_750,
  DIR_970,
  DIR_A2,
  DIR_E500,
  DIR_E500mc,
  DIR_E5500,
  DIR_PWR3,
  DIR_PWR4,
  DIR_PWR5,
  DIR_PWR5X,
  DIR_PWR6,
  DIR_PWR6X,
```
- **EN**: Opens or re-enters namespace scope so backend symbols stay grouped under LLVM naming conventions. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里进入或重新进入命名空间作用域，使后端符号按照 LLVM 的命名约定组织在一起。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 59-68

```cpp
  DIR_PWR7,
  DIR_PWR8,
  DIR_PWR9,
  DIR_PWR10,
  DIR_PWR11,
  DIR_PWR_FUTURE,
  DIR_64
};
}
```
- **EN**: Continues the PowerPC backend subtarget feature modeling with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的子目标特性建模，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 69-87

```cpp
class GlobalValue;

class PPCSubtarget : public PPCGenSubtargetInfo {
public:
  enum POPCNTDKind {
    POPCNTD_Unavailable,
    POPCNTD_Slow,
    POPCNTD_Fast
  };

protected:
  /// stackAlignment - The minimum alignment known to hold of the stack frame on
  /// entry to the function and which must be maintained by every function.
  Align StackAlignment;

  /// Selected instruction itineraries (one entry per itinerary class.)
  InstrItineraryData InstrItins;

// Bool members corresponding to the SubtargetFeatures defined in tablegen.
```
- **EN**: Declares a backend-facing type `GlobalValue`, `PPCSubtarget`, `itineraries` and outlines the API or state that nearby code will rely on. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里声明面向后端的类型 `GlobalValue`, `PPCSubtarget`, `itineraries`，并勾勒出周边代码会依赖的接口或状态。 该区间与栈帧布局或栈访问相关。

### Lines 88-113

```cpp
#define GET_SUBTARGETINFO_MACRO(ATTRIBUTE, DEFAULT, GETTER)                    \
  bool ATTRIBUTE = DEFAULT;
#include "PPCGenSubtargetInfo.inc"

  /// Which cpu directive was used.
  unsigned CPUDirective;

  bool IsLittleEndian;

  POPCNTDKind HasPOPCNTD;

  const PPCTargetMachine &TM;
  PPCFrameLowering FrameLowering;
  PPCInstrInfo InstrInfo;
  PPCTargetLowering TLInfo;

  // SelectionDAGISel related APIs.
  std::unique_ptr<const SelectionDAGTargetInfo> TSInfo;

  /// GlobalISel related APIs.
  std::unique_ptr<CallLowering> CallLoweringInfo;
  std::unique_ptr<LegalizerInfo> Legalizer;
  std::unique_ptr<RegisterBankInfo> RegBankInfo;
  std::unique_ptr<InstructionSelector> InstSelector;

public:
```
- **EN**: Pulls in direct dependencies required by this subtarget feature modeling, so later declarations can reuse LLVM infrastructure and target-specific helpers. This range participates in the GlobalISel pipeline.
- **CN**: 这一段引入该子目标特性建模所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段参与 GlobalISel 流水线。

### Lines 114-139

```cpp
  /// This constructor initializes the data members to match that
  /// of the specified triple.
  ///
  PPCSubtarget(const Triple &TT, StringRef CPU, StringRef TuneCPU, StringRef FS,
               const PPCTargetMachine &TM);

  ~PPCSubtarget() override;

  /// ParseSubtargetFeatures - Parses features string setting specified
  /// subtarget options.  Definition of function is auto generated by tblgen.
  void ParseSubtargetFeatures(StringRef CPU, StringRef TuneCPU, StringRef FS);

  /// getStackAlignment - Returns the minimum alignment known to hold of the
  /// stack frame on entry to the function and which must be maintained by every
  /// function for this subtarget.
  Align getStackAlignment() const { return StackAlignment; }

  /// getCPUDirective - Returns the -m directive specified for the cpu.
  ///
  unsigned getCPUDirective() const { return CPUDirective; }

  /// getInstrItins - Return the instruction itineraries based on subtarget
  /// selection.
  const InstrItineraryData *getInstrItineraryData() const override {
    return &InstrItins;
  }
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This constructor initializes the data members to match that". Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This constructor initializes the data members to match that”。 该区间与栈帧布局或栈访问相关。

### Lines 140-165

```cpp

  const PPCFrameLowering *getFrameLowering() const override {
    return &FrameLowering;
  }
  const PPCInstrInfo *getInstrInfo() const override { return &InstrInfo; }
  const PPCTargetLowering *getTargetLowering() const override {
    return &TLInfo;
  }

  const SelectionDAGTargetInfo *getSelectionDAGInfo() const override;

  const PPCRegisterInfo *getRegisterInfo() const override {
    return &getInstrInfo()->getRegisterInfo();
  }
  const PPCTargetMachine &getTargetMachine() const { return TM; }

  /// initializeSubtargetDependencies - Initializes using a CPU, a TuneCPU,  and
  /// feature string so that we can use initializer lists for subtarget
  /// initialization.
  PPCSubtarget &initializeSubtargetDependencies(StringRef CPU,
                                                StringRef TuneCPU,
                                                StringRef FS);

private:
  void initializeEnvironment();
  void initSubtargetFeatures(StringRef CPU, StringRef TuneCPU, StringRef FS);
```
- **EN**: Implements helper routine(s) `getFrameLowering`, `getInstrInfo`, `getTargetLowering` for this portion of the PowerPC backend subtarget feature modeling. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 PowerPC 后端该部分子目标特性建模所需的辅助例程 `getFrameLowering`, `getInstrInfo`, `getTargetLowering`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 166-178

```cpp

public:
  // useSoftFloat - Return true if soft-float option is turned on.
  bool useSoftFloat() const {
    if (isAIXABI() && !HasHardFloat)
      report_fatal_error("soft-float is not yet supported on AIX.");
    return !HasHardFloat;
  }

  // isLittleEndian - True if generating little-endian code
  bool isLittleEndian() const { return IsLittleEndian; }

// Getters for SubtargetFeatures defined in tablegen.
```
- **EN**: Implements helper routine(s) `useSoftFloat`, `isAIXABI`, `report_fatal_error` for this portion of the PowerPC backend subtarget feature modeling. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分子目标特性建模所需的辅助例程 `useSoftFloat`, `isAIXABI`, `report_fatal_error`。 子目标特性裁剪会影响这里的行为。

### Lines 179-204

```cpp
#define GET_SUBTARGETINFO_MACRO(ATTRIBUTE, DEFAULT, GETTER)                    \
  bool GETTER() const { return ATTRIBUTE; }
#include "PPCGenSubtargetInfo.inc"

  Align getPlatformStackAlignment() const {
    return Align(16);
  }

  unsigned  getRedZoneSize() const {
    if (isPPC64())
      // 288 bytes = 18*8 (FPRs) + 18*8 (GPRs, GPR13 reserved)
      return 288;

    // AIX PPC32: 220 bytes = 18*8 (FPRs) + 19*4 (GPRs);
    // PPC32 SVR4ABI has no redzone.
    return isAIXABI() ? 220 : 0;
  }

  bool needsSwapsForVSXMemOps() const {
    return hasVSX() && isLittleEndian() && !hasP9Vector();
  }

  POPCNTDKind hasPOPCNTD() const { return HasPOPCNTD; }

  bool isTargetELF() const { return getTargetTriple().isOSBinFormatELF(); }
  bool isTargetMachO() const { return getTargetTriple().isOSBinFormatMachO(); }
```
- **EN**: Pulls in direct dependencies required by this subtarget feature modeling, so later declarations can reuse LLVM infrastructure and target-specific helpers. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这一段引入该子目标特性建模所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 该区间与栈帧布局或栈访问相关。

### Lines 205-230

```cpp
  bool isTargetLinux() const { return getTargetTriple().isOSLinux(); }

  bool isAIXABI() const { return getTargetTriple().isOSAIX(); }
  bool isSVR4ABI() const { return !isAIXABI(); }
  bool isELFv2ABI() const;

  bool is64BitELFABI() const { return isSVR4ABI() && isPPC64(); }
  bool is32BitELFABI() const { return isSVR4ABI() && !isPPC64(); }
  bool isUsingPCRelativeCalls() const;

  /// Originally, this function return hasISEL(). Now we always enable it,
  /// but may expand the ISEL instruction later.
  bool enableEarlyIfConversion() const override { return true; }

  /// Scheduling customization.
  bool enableMachineScheduler() const override;
  /// Pipeliner customization.
  bool enableMachinePipeliner() const override;
  /// Machine Pipeliner customization
  bool useDFAforSMS() const override;
  /// This overrides the PostRAScheduler bit in the SchedModel for each CPU.
  bool enablePostRAScheduler() const override;
  AntiDepBreakMode getAntiDepBreakMode() const override;
  void getCriticalPathRCs(RegClassVector &CriticalPathRCs) const override;

  void overrideSchedPolicy(MachineSchedPolicy &Policy,
```
- **EN**: Implements helper routine(s) `isTargetLinux`, `getTargetTriple`, `isOSLinux` for this portion of the PowerPC backend subtarget feature modeling. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里实现了 PowerPC 后端该部分子目标特性建模所需的辅助例程 `isTargetLinux`, `getTargetTriple`, `isOSLinux`。 这一段包含调度或处理器模型元数据。

### Lines 231-256

```cpp
                           const SchedRegion &Region) const override;

  bool useAA() const override;

  bool enableSubRegLiveness() const override;

  bool enableSpillageCopyElimination() const override { return true; }

  /// True if the GV will be accessed via an indirect symbol.
  bool isGVIndirectSymbol(const GlobalValue *GV) const;

  MVT getScalarIntVT() const { return isPPC64() ? MVT::i64 : MVT::i32; }

  /// Calculates the effective code model for argument GV.
  CodeModel::Model getCodeModel(const TargetMachine &TM,
                                const GlobalValue *GV) const;

  /// True if the ABI is descriptor based.
  bool usesFunctionDescriptors() const {
    // Both 32-bit and 64-bit AIX are descriptor based. For ELF only the 64-bit
    // v1 ABI uses descriptors.
    return isAIXABI() || (is64BitELFABI() && !isELFv2ABI());
  }

  unsigned descriptorTOCAnchorOffset() const {
    assert(usesFunctionDescriptors() &&
```
- **EN**: Implements helper routine(s) `useAA`, `enableSubRegLiveness`, `enableSpillageCopyElimination` for this portion of the PowerPC backend subtarget feature modeling. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里实现了 PowerPC 后端该部分子目标特性建模所需的辅助例程 `useAA`, `enableSubRegLiveness`, `enableSpillageCopyElimination`。 这一段包含调度或处理器模型元数据。

### Lines 257-282

```cpp
           "Should only be called when the target uses descriptors.");
    return IsPPC64 ? 8 : 4;
  }

  unsigned descriptorEnvironmentPointerOffset() const {
    assert(usesFunctionDescriptors() &&
           "Should only be called when the target uses descriptors.");
    return IsPPC64 ? 16 : 8;
  }

  MCRegister getEnvironmentPointerRegister() const {
    assert(usesFunctionDescriptors() &&
           "Should only be called when the target uses descriptors.");
     return IsPPC64 ? PPC::X11 : PPC::R11;
  }

  MCRegister getTOCPointerRegister() const {
    assert((is64BitELFABI() || isAIXABI()) &&
           "Should only be called when the target is a TOC based ABI.");
    return IsPPC64 ? PPC::X2 : PPC::R2;
  }

  MCRegister getThreadPointerRegister() const {
    assert((is64BitELFABI() || isAIXABI()) &&
           "Should only be called for targets with a thread pointer register.");
    return IsPPC64 ? PPC::X13 : PPC::R13;
```
- **EN**: Implements helper routine(s) `descriptorEnvironmentPointerOffset`, `usesFunctionDescriptors`, `getEnvironmentPointerRegister` for this portion of the PowerPC backend subtarget feature modeling.
- **CN**: 这里实现了 PowerPC 后端该部分子目标特性建模所需的辅助例程 `descriptorEnvironmentPointerOffset`, `usesFunctionDescriptors`, `getEnvironmentPointerRegister`。

### Lines 283-308

```cpp
  }

  MCRegister getStackPointerRegister() const {
    return IsPPC64 ? PPC::X1 : PPC::R1;
  }

  MCRegister getGlueCodeDescriptorRegister() const {
    return IsPPC64 ? PPC::X11 : PPC::R11;
  }

  bool isXRaySupported() const override { return IsPPC64 && IsLittleEndian; }

  bool isPredictableSelectIsExpensive() const {
    return PredictableSelectIsExpensive;
  }

  // Select allocation orders of GPRC and G8RC. It should be strictly consistent
  // with corresponding AltOrders in PPCRegisterInfo.td.
  unsigned getGPRAllocationOrderIdx() const {
    if (is64BitELFABI())
      return 1;
    if (isAIXABI())
      return 2;
    return 0;
  }
```
- **EN**: Implements helper routine(s) `getStackPointerRegister`, `getGlueCodeDescriptorRegister`, `isXRaySupported` for this portion of the PowerPC backend subtarget feature modeling. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 PowerPC 后端该部分子目标特性建模所需的辅助例程 `getStackPointerRegister`, `getGlueCodeDescriptorRegister`, `isXRaySupported`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 309-316

```cpp
  // GlobalISEL
  const CallLowering *getCallLowering() const override;
  const RegisterBankInfo *getRegBankInfo() const override;
  const LegalizerInfo *getLegalizerInfo() const override;
  InstructionSelector *getInstructionSelector() const override;
};
} // End llvm namespace
```
- **EN**: Declares function entry points including `getCallLowering`, `getRegBankInfo`, `getLegalizerInfo` that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `getCallLowering`, `getRegBankInfo`, `getLegalizerInfo`。

### Lines 317-317

```cpp
#endif
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Subtarget feature modeling / 子目标特性建模
- GlobalISel pipeline / GlobalISel 流水线
- SelectionDAG lowering / SelectionDAG lowering
- Calling convention handling / 调用约定处理
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPCFrameLowering.h`
- `PPCISelLowering.h`
- `PPCInstrInfo.h`
- `llvm/CodeGen/GlobalISel/CallLowering.h`
- `llvm/CodeGen/GlobalISel/LegalizerInfo.h`
- `llvm/CodeGen/RegisterBankInfo.h`
- `llvm/CodeGen/TargetSubtargetInfo.h`
- `llvm/IR/DataLayout.h`
- `llvm/MC/MCInstrItineraries.h`
- `llvm/TargetParser/Triple.h`
- `PPCGenSubtargetInfo.inc`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM MC layer / LLVM MC 层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
