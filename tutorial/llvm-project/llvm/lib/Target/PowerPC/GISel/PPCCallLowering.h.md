# PPCCallLowering.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/GISel/PPCCallLowering.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file describes how to lower LLVM calls to machine code calls.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/GISel/PPCCallLowering.h`，主要负责 PowerPC 后端的GlobalISel 调用约定 lowering 逻辑。 文件内容以接口、类型声明和协作关系说明为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCCallLowering.h - Call lowering for GlobalISel -------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. This range participates in the GlobalISel pipeline.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 这一段参与 GlobalISel 流水线。

### Lines 7-11

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file describes how to lower LLVM calls to machine code calls.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 12-19

```cpp
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_POWERPC_GISEL_PPCCALLLOWERING_H
#define LLVM_LIB_TARGET_POWERPC_GISEL_PPCCALLLOWERING_H

#include "PPCISelLowering.h"
#include "llvm/CodeGen/GlobalISel/CallLowering.h"
```
- **EN**: Pulls in direct dependencies required by this GlobalISel call lowering logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. This range participates in the GlobalISel pipeline.
- **CN**: 这一段引入该GlobalISel 调用约定 lowering 逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段参与 GlobalISel 流水线。

### Lines 20-37

```cpp
namespace llvm {

class PPCTargetLowering;

class PPCCallLowering : public CallLowering {
public:
  PPCCallLowering(const PPCTargetLowering &TLI);

  bool lowerReturn(MachineIRBuilder &MIRBuilder, const Value *Val,
                   ArrayRef<Register> VRegs, FunctionLoweringInfo &FLI,
                   Register SwiftErrorVReg) const override;
  bool lowerFormalArguments(MachineIRBuilder &MIRBuilder, const Function &F,
                            ArrayRef<ArrayRef<Register>> VRegs,
                            FunctionLoweringInfo &FLI) const override;
  bool lowerCall(MachineIRBuilder &MIRBuilder,
                 CallLoweringInfo &Info) const override;
};
```
- **EN**: Opens or re-enters namespace scope so backend symbols stay grouped under LLVM naming conventions. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里进入或重新进入命名空间作用域，使后端符号按照 LLVM 的命名约定组织在一起。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 38-61

```cpp
class PPCIncomingValueHandler : public CallLowering::IncomingValueHandler {
public:
  PPCIncomingValueHandler(MachineIRBuilder &MIRBuilder,
                          MachineRegisterInfo &MRI)
      : CallLowering::IncomingValueHandler(MIRBuilder, MRI) {}

  uint64_t StackUsed;

private:
  void assignValueToReg(Register ValVReg, Register PhysReg,
                        const CCValAssign &VA,
                        ISD::ArgFlagsTy Flags = {}) override;

  void assignValueToAddress(Register ValVReg, Register Addr, LLT MemTy,
                            const MachinePointerInfo &MPO,
                            const CCValAssign &VA) override;

  Register getStackAddress(uint64_t Size, int64_t Offset,
                           MachinePointerInfo &MPO,
                           ISD::ArgFlagsTy Flags) override;

  virtual void markPhysRegUsed(unsigned PhysReg) = 0;
};
```
- **EN**: Declares a backend-facing type `PPCIncomingValueHandler`, `IncomingValueHandler`, `assignValueToReg` and outlines the API or state that nearby code will rely on. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明面向后端的类型 `PPCIncomingValueHandler`, `IncomingValueHandler`, `assignValueToReg`，并勾勒出周边代码会依赖的接口或状态。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 62-72

```cpp
class FormalArgHandler : public PPCIncomingValueHandler {

  void markPhysRegUsed(unsigned PhysReg) override;

public:
  FormalArgHandler(MachineIRBuilder &MIRBuilder, MachineRegisterInfo &MRI)
      : PPCIncomingValueHandler(MIRBuilder, MRI) {}
};

} // end namespace llvm
```
- **EN**: Declares a backend-facing type `FormalArgHandler`, `markPhysRegUsed`, `PPCIncomingValueHandler` and outlines the API or state that nearby code will rely on. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明面向后端的类型 `FormalArgHandler`, `markPhysRegUsed`, `PPCIncomingValueHandler`，并勾勒出周边代码会依赖的接口或状态。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 73-73

```cpp
#endif
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- GlobalISel call lowering logic / GlobalISel 调用约定 lowering 逻辑
- GlobalISel pipeline / GlobalISel 流水线
- SelectionDAG lowering / SelectionDAG lowering
- Calling convention handling / 调用约定处理
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Instruction selection or opcode handling / 指令选择或操作码处理

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPCISelLowering.h`
- `llvm/CodeGen/GlobalISel/CallLowering.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
