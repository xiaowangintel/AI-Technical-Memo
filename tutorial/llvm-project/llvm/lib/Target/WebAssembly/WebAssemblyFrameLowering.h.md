# WebAssemblyFrameLowering.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyFrameLowering.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This class implements WebAssembly-specific bits of TargetFrameLowering class.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyFrameLowering.h`，主要负责 WebAssembly 后端的栈帧 lowering 逻辑。 文件内容以接口、类型声明和协作关系说明为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
// WebAssemblyFrameLowering.h - TargetFrameLowering for WebAssembly -*- C++ -*-/
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 该区间与栈帧布局或栈访问相关。

### Lines 7-12

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This class implements WebAssembly-specific bits of
/// TargetFrameLowering class.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file". Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。 该区间与栈帧布局或栈访问相关。

### Lines 13-19

```cpp
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_WEBASSEMBLY_WEBASSEMBLYFRAMELOWERING_H
#define LLVM_LIB_TARGET_WEBASSEMBLY_WEBASSEMBLYFRAMELOWERING_H

#include "llvm/CodeGen/TargetFrameLowering.h"
```
- **EN**: Pulls in direct dependencies required by this stack frame lowering logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这一段引入该栈帧 lowering 逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 该区间与栈帧布局或栈访问相关。

### Lines 20-45

```cpp
namespace llvm {

class WebAssemblyFrameLowering final : public TargetFrameLowering {
public:
  /// Size of the red zone for the user stack (leaf functions can use this much
  /// space below the stack pointer without writing it back to __stack_pointer
  /// global).
  // TODO: (ABI) Revisit and decide how large it should be.
  static const size_t RedZoneSize = 128;

  WebAssemblyFrameLowering()
      : TargetFrameLowering(StackGrowsDown, /*StackAlignment=*/Align(16),
                            /*LocalAreaOffset=*/0,
                            /*TransientStackAlignment=*/Align(16),
                            /*StackRealignable=*/true) {}

  MachineBasicBlock::iterator
  eliminateCallFramePseudoInstr(MachineFunction &MF, MachineBasicBlock &MBB,
                                MachineBasicBlock::iterator I) const override;

  /// These methods insert prolog and epilog code into the function.
  void emitPrologue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
  void emitEpilogue(MachineFunction &MF, MachineBasicBlock &MBB) const override;

  bool hasReservedCallFrame(const MachineFunction &MF) const override;
  bool isSupportedStackID(TargetStackID::Value ID) const override;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Size of the red zone for the user stack (leaf functions can use this much". Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Size of the red zone for the user stack (leaf functions can use this much”。 该区间与栈帧布局或栈访问相关。

### Lines 46-71

```cpp
  DwarfFrameBase getDwarfFrameBase(const MachineFunction &MF) const override;

  bool needsPrologForEH(const MachineFunction &MF) const;

  /// Write SP back to __stack_pointer global.
  void writeSPToGlobal(unsigned SrcReg, MachineFunction &MF,
                       MachineBasicBlock &MBB,
                       MachineBasicBlock::iterator &InsertStore,
                       const DebugLoc &DL) const;

  // Returns the index of the WebAssembly local to which the stack object
  // FrameIndex in MF should be allocated, or std::nullopt.
  static std::optional<unsigned> getLocalForStackObject(MachineFunction &MF,
                                                        int FrameIndex);

  static unsigned getSPReg(const MachineFunction &MF);
  static unsigned getFPReg(const MachineFunction &MF);
  static unsigned getOpcConst(const MachineFunction &MF);
  static unsigned getOpcAdd(const MachineFunction &MF);
  static unsigned getOpcSub(const MachineFunction &MF);
  static unsigned getOpcAnd(const MachineFunction &MF);
  static unsigned getOpcGlobGet(const MachineFunction &MF);
  static unsigned getOpcGlobSet(const MachineFunction &MF);

protected:
  bool hasFPImpl(const MachineFunction &MF) const override;
```
- **EN**: Declares function entry points including `getDwarfFrameBase`, `needsPrologForEH`, `writeSPToGlobal` that other backend components call later. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `getDwarfFrameBase`, `needsPrologForEH`, `writeSPToGlobal`。 该区间与栈帧布局或栈访问相关。

### Lines 72-81

```cpp

private:
  bool hasBP(const MachineFunction &MF) const;
  bool needsSPForLocalFrame(const MachineFunction &MF) const;
  bool needsSP(const MachineFunction &MF) const;
  bool needsSPWriteback(const MachineFunction &MF) const;
};

} // end namespace llvm
```
- **EN**: Declares function entry points including `hasBP`, `needsSPForLocalFrame`, `needsSP` that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `hasBP`, `needsSPForLocalFrame`, `needsSP`。

### Lines 82-82

```cpp
#endif
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Stack frame lowering logic / 栈帧 lowering 逻辑
- Stack frame management / 栈帧管理
- Declarative TableGen records / 声明式 TableGen 记录
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `llvm/CodeGen/TargetFrameLowering.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
