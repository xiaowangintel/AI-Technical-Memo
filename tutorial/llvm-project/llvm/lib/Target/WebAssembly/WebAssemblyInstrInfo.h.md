# WebAssemblyInstrInfo.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyInstrInfo.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file contains the WebAssembly implementation of the TargetInstrInfo class.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyInstrInfo.h`，主要负责 WebAssembly 后端的指令信息与 lowering 辅助逻辑。 文件内容以接口、类型声明和协作关系说明为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//=- WebAssemblyInstrInfo.h - WebAssembly Instruction Information -*- C++ -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-12

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file contains the WebAssembly implementation of the
/// TargetInstrInfo class.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 13-19

```cpp
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_WEBASSEMBLY_WEBASSEMBLYINSTRINFO_H
#define LLVM_LIB_TARGET_WEBASSEMBLY_WEBASSEMBLYINSTRINFO_H

#include "WebAssemblyRegisterInfo.h"
#include "llvm/ADT/ArrayRef.h"
```
- **EN**: Pulls in direct dependencies required by this instruction information and lowering helpers, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该指令信息与 lowering 辅助逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 20-27

```cpp
#include "llvm/CodeGen/TargetInstrInfo.h"

#define GET_INSTRINFO_HEADER
#include "WebAssemblyGenInstrInfo.inc"

#define GET_INSTRINFO_OPERAND_ENUM
#include "WebAssemblyGenInstrInfo.inc"
```
- **EN**: Pulls in direct dependencies required by this instruction information and lowering helpers, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该指令信息与 lowering 辅助逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 28-53

```cpp
namespace llvm {

class WebAssemblySubtarget;

class WebAssemblyInstrInfo final : public WebAssemblyGenInstrInfo {
  const WebAssemblyRegisterInfo RI;

public:
  explicit WebAssemblyInstrInfo(const WebAssemblySubtarget &STI);

  const WebAssemblyRegisterInfo &getRegisterInfo() const { return RI; }

  bool isReMaterializableImpl(const MachineInstr &MI) const override;

  void copyPhysReg(MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
                   const DebugLoc &DL, Register DestReg, Register SrcReg,
                   bool KillSrc, bool RenamableDest = false,
                   bool RenamableSrc = false) const override;
  MachineInstr *commuteInstructionImpl(MachineInstr &MI, bool NewMI,
                                       unsigned OpIdx1,
                                       unsigned OpIdx2) const override;

  bool analyzeBranch(MachineBasicBlock &MBB, MachineBasicBlock *&TBB,
                     MachineBasicBlock *&FBB,
                     SmallVectorImpl<MachineOperand> &Cond,
                     bool AllowModify = false) const override;
```
- **EN**: Opens or re-enters namespace scope so backend symbols stay grouped under LLVM naming conventions. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里进入或重新进入命名空间作用域，使后端符号按照 LLVM 的命名约定组织在一起。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 54-73

```cpp
  unsigned removeBranch(MachineBasicBlock &MBB,
                        int *BytesRemoved = nullptr) const override;
  unsigned insertBranch(MachineBasicBlock &MBB, MachineBasicBlock *TBB,
                        MachineBasicBlock *FBB, ArrayRef<MachineOperand> Cond,
                        const DebugLoc &DL,
                        int *BytesAdded = nullptr) const override;
  bool
  reverseBranchCondition(SmallVectorImpl<MachineOperand> &Cond) const override;

  ArrayRef<std::pair<int, const char *>>
  getSerializableTargetIndices() const override;

  const MachineOperand &getCalleeOperand(const MachineInstr &MI) const override;

  bool isExplicitTargetIndexDef(const MachineInstr &MI, int &Index,
                                int64_t &Offset) const override;
};

} // end namespace llvm
```
- **EN**: Declares function entry points including `removeBranch`, `insertBranch`, `reverseBranchCondition` that other backend components call later. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `removeBranch`, `insertBranch`, `reverseBranchCondition`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 74-74

```cpp
#endif
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Instruction information and lowering helpers / 指令信息与 lowering 辅助逻辑
- Register modeling / 寄存器建模
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblyRegisterInfo.h`
- `llvm/ADT/ArrayRef.h`
- `llvm/CodeGen/TargetInstrInfo.h`
- `WebAssemblyGenInstrInfo.inc`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
