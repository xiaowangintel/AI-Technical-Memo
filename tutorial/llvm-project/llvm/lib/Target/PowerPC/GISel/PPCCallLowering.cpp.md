# PPCCallLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/GISel/PPCCallLowering.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements the lowering of LLVM calls to machine code calls for GlobalISel.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/GISel/PPCCallLowering.cpp`，主要负责 PowerPC 后端的GlobalISel 调用约定 lowering 逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

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

### Lines 7-12

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements the lowering of LLVM calls to machine code calls for
/// GlobalISel.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file". This range participates in the GlobalISel pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。 这一段参与 GlobalISel 流水线。

### Lines 13-19

```cpp
//===----------------------------------------------------------------------===//

#include "PPCCallLowering.h"
#include "PPCCallingConv.h"
#include "PPCISelLowering.h"
#include "llvm/CodeGen/CallingConvLower.h"
#include "llvm/CodeGen/GlobalISel/CallLowering.h"
```
- **EN**: Pulls in direct dependencies required by this GlobalISel call lowering logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. This range participates in the GlobalISel pipeline.
- **CN**: 这一段引入该GlobalISel 调用约定 lowering 逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段参与 GlobalISel 流水线。

### Lines 20-27

```cpp
#include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/TargetCallingConv.h"

#define DEBUG_TYPE "ppc-call-lowering"

using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this GlobalISel call lowering logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该GlobalISel 调用约定 lowering 逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 28-65

```cpp
namespace {

struct OutgoingArgHandler : public CallLowering::OutgoingValueHandler {
  OutgoingArgHandler(MachineIRBuilder &MIRBuilder, MachineRegisterInfo &MRI,
                     MachineInstrBuilder MIB)
      : OutgoingValueHandler(MIRBuilder, MRI), MIB(MIB) {}

  void assignValueToReg(Register ValVReg, Register PhysReg,
                        const CCValAssign &VA,
                        ISD::ArgFlagsTy Flags = {}) override;
  void assignValueToAddress(Register ValVReg, Register Addr, LLT MemTy,
                            const MachinePointerInfo &MPO,
                            const CCValAssign &VA) override;
  Register getStackAddress(uint64_t Size, int64_t Offset,
                           MachinePointerInfo &MPO,
                           ISD::ArgFlagsTy Flags) override;

  MachineInstrBuilder MIB;
};
} // namespace

void OutgoingArgHandler::assignValueToReg(Register ValVReg, Register PhysReg,
                                          const CCValAssign &VA,
                                          ISD::ArgFlagsTy Flags) {
  MIB.addUse(PhysReg, RegState::Implicit);
  Register ExtReg = extendRegister(ValVReg, VA);
  MIRBuilder.buildCopy(PhysReg, ExtReg);
}

void OutgoingArgHandler::assignValueToAddress(Register ValVReg, Register Addr,
                                              LLT MemTy,
                                              const MachinePointerInfo &MPO,
                                              const CCValAssign &VA) {
  llvm_unreachable("unimplemented");
}

Register OutgoingArgHandler::getStackAddress(uint64_t Size, int64_t Offset,
                                             MachinePointerInfo &MPO,
```
- **EN**: Declares a backend-facing type `OutgoingArgHandler`, `OutgoingValueHandler`, `MIB` and outlines the API or state that nearby code will rely on. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明面向后端的类型 `OutgoingArgHandler`, `OutgoingValueHandler`, `MIB`，并勾勒出周边代码会依赖的接口或状态。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 66-103

```cpp
                                             ISD::ArgFlagsTy Flags) {
  llvm_unreachable("unimplemented");
}

PPCCallLowering::PPCCallLowering(const PPCTargetLowering &TLI)
    : CallLowering(&TLI) {}

bool PPCCallLowering::lowerReturn(MachineIRBuilder &MIRBuilder,
                                  const Value *Val, ArrayRef<Register> VRegs,
                                  FunctionLoweringInfo &FLI,
                                  Register SwiftErrorVReg) const {
  auto MIB = MIRBuilder.buildInstrNoInsert(PPC::BLR8);
  bool Success = true;
  MachineFunction &MF = MIRBuilder.getMF();
  const Function &F = MF.getFunction();
  MachineRegisterInfo &MRI = MF.getRegInfo();
  auto &DL = F.getDataLayout();
  if (!VRegs.empty()) {
    // Setup the information about the return value.
    ArgInfo OrigArg{VRegs, Val->getType(), 0};
    setArgFlags(OrigArg, AttributeList::ReturnIndex, DL, F);

    // Split the return value into consecutive registers if needed.
    SmallVector<ArgInfo, 8> SplitArgs;
    splitToValueTypes(OrigArg, SplitArgs, DL, F.getCallingConv());

    // Use the calling convention callback to determine type and location of
    // return value.
    OutgoingValueAssigner ArgAssigner(RetCC_PPC);

    // Handler to move the return value into the correct location.
    OutgoingArgHandler ArgHandler(MIRBuilder, MRI, MIB);

    // Iterate over all return values, and move them to the assigned location.
    Success = determineAndHandleAssignments(ArgHandler, ArgAssigner, SplitArgs,
                                            MIRBuilder, F.getCallingConv(),
                                            F.isVarArg());
  }
```
- **EN**: Implements helper routine(s) `llvm_unreachable`, `PPCCallLowering`, `CallLowering` for this portion of the PowerPC backend GlobalISel call lowering logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分GlobalISel 调用约定 lowering 逻辑所需的辅助例程 `llvm_unreachable`, `PPCCallLowering`, `CallLowering`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 104-141

```cpp
  MIRBuilder.insertInstr(MIB);
  return Success;
}

bool PPCCallLowering::lowerCall(MachineIRBuilder &MIRBuilder,
                                CallLoweringInfo &Info) const {
  return false;
}

bool PPCCallLowering::lowerFormalArguments(MachineIRBuilder &MIRBuilder,
                                           const Function &F,
                                           ArrayRef<ArrayRef<Register>> VRegs,
                                           FunctionLoweringInfo &FLI) const {
  MachineFunction &MF = MIRBuilder.getMF();
  MachineRegisterInfo &MRI = MF.getRegInfo();
  const auto &DL = F.getDataLayout();
  auto &TLI = *getTLI<PPCTargetLowering>();

  // Loop over each arg, set flags and split to single value types
  SmallVector<ArgInfo, 8> SplitArgs;
  unsigned I = 0;
  for (const auto &Arg : F.args()) {
    if (DL.getTypeStoreSize(Arg.getType()).isZero())
      continue;

    ArgInfo OrigArg{VRegs[I], Arg, I};
    setArgFlags(OrigArg, I + AttributeList::FirstArgIndex, DL, F);
    splitToValueTypes(OrigArg, SplitArgs, DL, F.getCallingConv());
    ++I;
  }

  CCAssignFn *AssignFn =
      TLI.ccAssignFnForCall(F.getCallingConv(), false, F.isVarArg());
  IncomingValueAssigner ArgAssigner(AssignFn);
  FormalArgHandler ArgHandler(MIRBuilder, MRI);
  return determineAndHandleAssignments(ArgHandler, ArgAssigner, SplitArgs,
                                       MIRBuilder, F.getCallingConv(),
                                       F.isVarArg());
```
- **EN**: Implements helper routine(s) `insertInstr`, `lowerCall`, `lowerFormalArguments` for this portion of the PowerPC backend GlobalISel call lowering logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分GlobalISel 调用约定 lowering 逻辑所需的辅助例程 `insertInstr`, `lowerCall`, `lowerFormalArguments`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 142-179

```cpp
}

void PPCIncomingValueHandler::assignValueToReg(Register ValVReg,
                                               Register PhysReg,
                                               const CCValAssign &VA,
                                               ISD::ArgFlagsTy Flags) {
  markPhysRegUsed(PhysReg);
  IncomingValueHandler::assignValueToReg(ValVReg, PhysReg, VA);
}

void PPCIncomingValueHandler::assignValueToAddress(
    Register ValVReg, Register Addr, LLT MemTy, const MachinePointerInfo &MPO,
    const CCValAssign &VA) {
  // define a lambda expression to load value
  auto BuildLoad = [](MachineIRBuilder &MIRBuilder,
                      const MachinePointerInfo &MPO, LLT MemTy,
                      const DstOp &Res, Register Addr) {
    MachineFunction &MF = MIRBuilder.getMF();
    auto *MMO = MF.getMachineMemOperand(MPO, MachineMemOperand::MOLoad, MemTy,
                                        inferAlignFromPtrInfo(MF, MPO));
    return MIRBuilder.buildLoad(Res, Addr, *MMO);
  };

  BuildLoad(MIRBuilder, MPO, MemTy, ValVReg, Addr);
}

Register PPCIncomingValueHandler::getStackAddress(uint64_t Size, int64_t Offset,
                                                  MachinePointerInfo &MPO,
                                                  ISD::ArgFlagsTy Flags) {
  auto &MFI = MIRBuilder.getMF().getFrameInfo();
  const bool IsImmutable = !Flags.isByVal();
  int FI = MFI.CreateFixedObject(Size, Offset, IsImmutable);
  MPO = MachinePointerInfo::getFixedStack(MIRBuilder.getMF(), FI);

  // Build Frame Index based on whether the machine is 32-bit or 64-bit
  llvm::LLT FramePtr = LLT::pointer(
      0, MIRBuilder.getMF().getDataLayout().getPointerSizeInBits());
  MachineInstrBuilder AddrReg = MIRBuilder.buildFrameIndex(FramePtr, FI);
```
- **EN**: Implements helper routine(s) `assignValueToReg`, `markPhysRegUsed`, `assignValueToAddress` for this portion of the PowerPC backend GlobalISel call lowering logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分GlobalISel 调用约定 lowering 逻辑所需的辅助例程 `assignValueToReg`, `markPhysRegUsed`, `assignValueToAddress`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 180-187

```cpp
  StackUsed = std::max(StackUsed, Size + Offset);
  return AddrReg.getReg(0);
}

void FormalArgHandler::markPhysRegUsed(unsigned PhysReg) {
  MIRBuilder.getMRI()->addLiveIn(PhysReg);
  MIRBuilder.getMBB().addLiveIn(PhysReg);
}
```
- **EN**: Implements helper routine(s) `max`, `getReg`, `markPhysRegUsed` for this portion of the PowerPC backend GlobalISel call lowering logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 PowerPC 后端该部分GlobalISel 调用约定 lowering 逻辑所需的辅助例程 `max`, `getReg`, `markPhysRegUsed`。 这里重点涉及寄存器分配与寄存器类约束。

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

- `PPCCallLowering.h`
- `PPCCallingConv.h`
- `PPCISelLowering.h`
- `llvm/CodeGen/CallingConvLower.h`
- `llvm/CodeGen/GlobalISel/CallLowering.h`
- `llvm/CodeGen/GlobalISel/MachineIRBuilder.h`
- `llvm/CodeGen/MachineFrameInfo.h`
- `llvm/CodeGen/TargetCallingConv.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
