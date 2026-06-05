# AMDGPUCallLowering.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUCallLowering.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPUCallLowering in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 AMDGPUCallLowering 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: File banner, includes, and setup
```cpp
//===- lib/Target/AMDGPU/AMDGPUCallLowering.h - Call lowering -*- C++ -*---===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This file describes how to lower LLVM calls to machine code calls.
///
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_AMDGPUCALLLOWERING_H
#define LLVM_LIB_TARGET_AMDGPU_AMDGPUCALLLOWERING_H

#include "llvm/CodeGen/GlobalISel/CallLowering.h"

namespace llvm {

class GCNSubtarget;
class MachineInstrBuilder;
class SIMachineFunctionInfo;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `GCNSubtarget`, `MachineInstrBuilder`, `SIMachineFunctionInfo`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`GCNSubtarget`, `MachineInstrBuilder`, `SIMachineFunctionInfo`。

### Lines 25-48: Declares class AMDGPUCallLowering
```cpp
class AMDGPUCallLowering final : public CallLowering {
  void lowerParameterPtr(Register DstReg, MachineIRBuilder &B,
                         uint64_t Offset) const;

  void lowerParameter(MachineIRBuilder &B, ArgInfo &AI, uint64_t Offset,
                      Align Alignment) const;

  bool canLowerReturn(MachineFunction &MF, CallingConv::ID CallConv,
                      SmallVectorImpl<BaseArgInfo> &Outs,
                      bool IsVarArg) const override;

  bool lowerReturnVal(MachineIRBuilder &B, const Value *Val,
                      ArrayRef<Register> VRegs, MachineInstrBuilder &Ret) const;

  void addOriginalExecToReturn(MachineFunction &MF,
                               MachineInstrBuilder &Ret) const;

public:
  AMDGPUCallLowering(const TargetLowering &TLI);

  bool lowerReturn(MachineIRBuilder &B, const Value *Val,
                   ArrayRef<Register> VRegs,
                   FunctionLoweringInfo &FLI) const override;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUCallLowering`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUCallLowering`。

### Lines 49-69: Defines lowerFormalArgumentsKernel
```cpp
  bool lowerFormalArgumentsKernel(MachineIRBuilder &B, const Function &F,
                                  ArrayRef<ArrayRef<Register>> VRegs) const;

  bool lowerFormalArguments(MachineIRBuilder &B, const Function &F,
                            ArrayRef<ArrayRef<Register>> VRegs,
                            FunctionLoweringInfo &FLI) const override;

  bool passSpecialInputs(MachineIRBuilder &MIRBuilder,
                         CCState &CCInfo,
                         SmallVectorImpl<std::pair<MCRegister, Register>> &ArgRegs,
                         CallLoweringInfo &Info) const;

  bool
  doCallerAndCalleePassArgsTheSameWay(CallLoweringInfo &Info,
                                      MachineFunction &MF,
                                      SmallVectorImpl<ArgInfo> &InArgs) const;

  bool
  areCalleeOutgoingArgsTailCallable(CallLoweringInfo &Info, MachineFunction &MF,
                                    SmallVectorImpl<ArgInfo> &OutArgs) const;

```
**EN:** This section contains concrete logic for lowerFormalArgumentsKernel. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 lowerFormalArgumentsKernel 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 70-89: Defines isEligibleForTailCallOptimization
```cpp
  /// Returns true if the call can be lowered as a tail call.
  bool
  isEligibleForTailCallOptimization(MachineIRBuilder &MIRBuilder,
                                    CallLoweringInfo &Info,
                                    SmallVectorImpl<ArgInfo> &InArgs,
                                    SmallVectorImpl<ArgInfo> &OutArgs) const;

  void handleImplicitCallArguments(
      MachineIRBuilder &MIRBuilder, MachineInstrBuilder &CallInst,
      const GCNSubtarget &ST, const SIMachineFunctionInfo &MFI,
      CallingConv::ID CalleeCC,
      ArrayRef<std::pair<MCRegister, Register>> ImplicitArgRegs) const;

  bool lowerTailCall(MachineIRBuilder &MIRBuilder, CallLoweringInfo &Info,
                     SmallVectorImpl<ArgInfo> &OutArgs) const;
  bool lowerChainCall(MachineIRBuilder &MIRBuilder,
                      CallLoweringInfo &Info) const;
  bool lowerCall(MachineIRBuilder &MIRBuilder,
                 CallLoweringInfo &Info) const override;

```
**EN:** This section contains concrete logic for isEligibleForTailCallOptimization. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 isEligibleForTailCallOptimization 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 90-94: Preprocessor guards and macros
```cpp
  static CCAssignFn *CCAssignFnForCall(CallingConv::ID CC, bool IsVarArg);
  static CCAssignFn *CCAssignFnForReturn(CallingConv::ID CC, bool IsVarArg);
};
} // End of namespace llvm;
#endif
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `GCNSubtarget`, `MachineInstrBuilder`, `SIMachineFunctionInfo`, `AMDGPUCallLowering`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; lowering / 降低
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"llvm/CodeGen/GlobalISel/CallLowering.h"`
