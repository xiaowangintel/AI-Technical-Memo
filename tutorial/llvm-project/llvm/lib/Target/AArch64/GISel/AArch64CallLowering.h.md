# AArch64CallLowering.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/GISel/AArch64CallLowering.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This file covers Call lowering. / 该文件实现 AArch64 后端中的调用降低。
## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Documented code section
```cpp
//===- AArch64CallLowering.h - Call lowering --------------------*- C++ -*-===//
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

#ifndef LLVM_LIB_TARGET_AARCH64_AARCH64CALLLOWERING_H
#define LLVM_LIB_TARGET_AARCH64_AARCH64CALLLOWERING_H
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 16-32: Includes and compile-time dependencies
```cpp

#include "llvm/ADT/ArrayRef.h"
#include "llvm/CodeGen/GlobalISel/CallLowering.h"
#include "llvm/IR/CallingConv.h"
#include <cstdint>
#include <functional>

namespace llvm {

class AArch64TargetLowering;
class CCValAssign;
class MachineIRBuilder;
class Type;

class AArch64CallLowering: public CallLowering {
public:
  AArch64CallLowering(const AArch64TargetLowering &TLI);
```
**EN:** This block pulls in the target-specific headers and LLVM infrastructure needed by the rest of the file.  
**CN:** 该代码块引入后续实现所需的目标专用头文件与 LLVM 基础设施。
### Lines 33-49: Core AArch64 backend logic
```cpp

  bool lowerReturn(MachineIRBuilder &MIRBuilder, const Value *Val,
                   ArrayRef<Register> VRegs, FunctionLoweringInfo &FLI,
                   Register SwiftErrorVReg) const override;

  bool canLowerReturn(MachineFunction &MF, CallingConv::ID CallConv,
                      SmallVectorImpl<BaseArgInfo> &Outs,
                      bool IsVarArg) const override;

  bool fallBackToDAGISel(const MachineFunction &MF) const override;

  bool lowerFormalArguments(MachineIRBuilder &MIRBuilder, const Function &F,
                            ArrayRef<ArrayRef<Register>> VRegs,
                            FunctionLoweringInfo &FLI) const override;

  bool lowerCall(MachineIRBuilder &MIRBuilder,
                 CallLoweringInfo &Info) const override;
```
**EN:** This block continues the file's main call lowering logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的调用降低主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 50-64: Function isEligibleForTailCallOptimization
```cpp

  /// Returns true if the call can be lowered as a tail call.
  bool
  isEligibleForTailCallOptimization(MachineIRBuilder &MIRBuilder,
                                    CallLoweringInfo &Info,
                                    SmallVectorImpl<ArgInfo> &InArgs,
                                    SmallVectorImpl<ArgInfo> &OutArgs) const;

  bool supportSwiftError() const override { return true; }

  bool isTypeIsValidForThisReturn(EVT Ty) const override;

private:
  using RegHandler = std::function<void(MachineIRBuilder &, Type *, unsigned,
                                        CCValAssign &)>;
```
**EN:** This block implements isEligibleForTailCallOptimization, advancing the file's call lowering flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 isEligibleForTailCallOptimization，通过 AArch64 专用的决策与数据处理推进本文件的调用降低流程。
### Lines 65-79: Namespace aliases and helper imports
```cpp

  using MemHandler =
      std::function<void(MachineIRBuilder &, int, CCValAssign &)>;

  void saveVarArgRegisters(MachineIRBuilder &MIRBuilder,
                           CallLowering::IncomingValueHandler &Handler,
                           CCState &CCInfo) const;

  bool lowerTailCall(MachineIRBuilder &MIRBuilder, CallLoweringInfo &Info,
                     SmallVectorImpl<ArgInfo> &OutArgs) const;

  bool
  doCallerAndCalleePassArgsTheSameWay(CallLoweringInfo &Info,
                                      MachineFunction &MF,
                                      SmallVectorImpl<ArgInfo> &InArgs) const;
```
**EN:** This block shortens names and makes common helper symbols available to later code.  
**CN:** 该代码块简化名称书写，并为后续代码引入常用辅助符号。
### Lines 80-88: Core AArch64 backend logic
```cpp

  bool
  areCalleeOutgoingArgsTailCallable(CallLoweringInfo &Info, MachineFunction &MF,
                                    SmallVectorImpl<ArgInfo> &OutArgs) const;
};

} // end namespace llvm

#endif // LLVM_LIB_TARGET_AARCH64_AARCH64CALLLOWERING_H
```
**EN:** This block continues the file's main call lowering logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的调用降低主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
- **EN:** Lowering calls into target-specific calling sequences **CN:** 将调用降低为目标专用调用序列
## Dependencies / 依赖关系
- **EN:** Core LLVM interfaces: llvm/ADT/ArrayRef.h, llvm/CodeGen/GlobalISel/CallLowering.h, llvm/IR/CallingConv.h **CN:** 核心 LLVM 接口：llvm/ADT/ArrayRef.h, llvm/CodeGen/GlobalISel/CallLowering.h, llvm/IR/CallingConv.h
- **EN:** Standard-library support: cstdint, functional **CN:** 标准库支持：cstdint, functional
- **EN:** Closely connected with neighboring AArch64 backend components responsible for call lowering. **CN:** 与周边负责调用降低的 AArch64 后端组件紧密协作。
