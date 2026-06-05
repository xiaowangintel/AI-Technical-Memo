# ARMFrameLowering.h — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMFrameLowering.h`
- Repository: `llvm-project`
- Purpose (EN): Declares `ARMFrameLowering` for the ARM backend and exposes interfaces for stack-frame layout and prologue/epilogue lowering.
- 用途 (CN): 声明 ARM 后端中的 `ARMFrameLowering`，并提供与栈帧布局以及序言/结语生成相关的接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- ARMTargetFrameLowering.h - Define frame lowering for ARM -*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 9-10
```cpp
#ifndef LLVM_LIB_TARGET_ARM_ARMFRAMELOWERING_H
#define LLVM_LIB_TARGET_ARM_ARMFRAMELOWERING_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 12-13
```cpp
#include "llvm/CodeGen/TargetFrameLowering.h"
#include "llvm/Support/TypeSize.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 15-15
```cpp
namespace llvm {
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 17-19
```cpp
class ARMSubtarget;
class CalleeSavedInfo;
class MachineFunction;
```
- EN: Declares `ARMSubtarget`, packaging target-specific state and APIs around `ARMFrameLowering`.
- CN: 这里声明 `ARMSubtarget`，把与 `ARMFrameLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 21-23
```cpp
class ARMFrameLowering : public TargetFrameLowering {
protected:
  const ARMSubtarget &STI;
```
- EN: Declares `ARMFrameLowering`, packaging target-specific state and APIs around `ARMFrameLowering`.
- CN: 这里声明 `ARMFrameLowering`，把与 `ARMFrameLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 25-26
```cpp
public:
  explicit ARMFrameLowering(const ARMSubtarget &sti);
```
- EN: Declares `ARMFrameLowering`, a lowering routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ARMFrameLowering`，它是一个围绕子目标特性处理展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 28-31
```cpp
  /// emitProlog/emitEpilog - These methods insert prolog and epilog code into
  /// the function.
  void emitPrologue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
  void emitEpilogue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
```
- EN: Declares `emitPrologue`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitPrologue`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 33-36
```cpp
  bool spillCalleeSavedRegisters(MachineBasicBlock &MBB,
                                 MachineBasicBlock::iterator MI,
                                 ArrayRef<CalleeSavedInfo> CSI,
                                 const TargetRegisterInfo *TRI) const override;
```
- EN: Declares `spillCalleeSavedRegisters`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `spillCalleeSavedRegisters`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 38-42
```cpp
  bool
  restoreCalleeSavedRegisters(MachineBasicBlock &MBB,
                              MachineBasicBlock::iterator MI,
                              MutableArrayRef<CalleeSavedInfo> CSI,
                              const TargetRegisterInfo *TRI) const override;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 44-44
```cpp
  bool keepFramePointer(const MachineFunction &MF) const;
```
- EN: Declares `keepFramePointer`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `keepFramePointer`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 46-46
```cpp
  bool enableCalleeSaveSkip(const MachineFunction &MF) const override;
```
- EN: Declares `enableCalleeSaveSkip`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `enableCalleeSaveSkip`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 48-55
```cpp
  bool isFPReserved(const MachineFunction &MF) const;
  bool requiresAAPCSFrameRecord(const MachineFunction &MF) const;
  bool hasReservedCallFrame(const MachineFunction &MF) const override;
  bool canSimplifyCallFramePseudos(const MachineFunction &MF) const override;
  StackOffset getFrameIndexReference(const MachineFunction &MF, int FI,
                                     Register &FrameReg) const override;
  int ResolveFrameIndexReference(const MachineFunction &MF, int FI,
                                 Register &FrameReg, int SPAdj) const;
```
- EN: Declares `isFPReserved`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isFPReserved`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 57-60
```cpp
  void getCalleeSaves(const MachineFunction &MF,
                      BitVector &SavedRegs) const override;
  void determineCalleeSaves(MachineFunction &MF, BitVector &SavedRegs,
                            RegScavenger *RS) const override;
```
- EN: Declares `getCalleeSaves`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getCalleeSaves`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 62-64
```cpp
  /// Update the IsRestored flag on LR if it is spilled, based on the return
  /// instructions.
  static void updateLRRestored(MachineFunction &MF);
```
- EN: Declares `updateLRRestored`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `updateLRRestored`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 66-67
```cpp
  void processFunctionBeforeFrameFinalized(
      MachineFunction &MF, RegScavenger *RS = nullptr) const override;
```
- EN: Declares `processFunctionBeforeFrameFinalized`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `processFunctionBeforeFrameFinalized`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 69-70
```cpp
  void adjustForSegmentedStacks(MachineFunction &MF,
                                MachineBasicBlock &MBB) const override;
```
- EN: Declares `adjustForSegmentedStacks`, a mutation/build routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `adjustForSegmentedStacks`，它是一个围绕机器函数状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 72-73
```cpp
  /// Returns true if the target will correctly handle shrink wrapping.
  bool enableShrinkWrapping(const MachineFunction &MF) const override;
```
- EN: Declares `enableShrinkWrapping`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `enableShrinkWrapping`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 75-79
```cpp
  bool isProfitableForNoCSROpt(const Function &F) const override {
    // The no-CSR optimisation is bad for code size on ARM, because we can save
    // many registers with a single PUSH/POP pair.
    return false;
  }
```
- EN: Implements `isProfitableForNoCSROpt`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isProfitableForNoCSROpt`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 81-84
```cpp
  bool
  assignCalleeSavedSpillSlots(MachineFunction &MF,
                              const TargetRegisterInfo *TRI,
                              std::vector<CalleeSavedInfo> &CSI) const override;
```
- EN: Declares `assignCalleeSavedSpillSlots`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `assignCalleeSavedSpillSlots`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 86-87
```cpp
  const SpillSlot *
  getCalleeSavedSpillSlots(unsigned &NumEntries) const override;
```
- EN: Declares `getCalleeSavedSpillSlots`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getCalleeSavedSpillSlots`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 89-90
```cpp
protected:
  bool hasFPImpl(const MachineFunction &MF) const override;
```
- EN: Declares `hasFPImpl`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `hasFPImpl`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 92-100
```cpp
private:
  void emitPushInst(MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
                    ArrayRef<CalleeSavedInfo> CSI, unsigned StmOpc,
                    unsigned StrOpc, bool NoGap,
                    function_ref<bool(unsigned)> Func) const;
  void emitPopInst(MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
                   MutableArrayRef<CalleeSavedInfo> CSI, unsigned LdmOpc,
                   unsigned LdrOpc, bool isVarArg, bool NoGap,
                   function_ref<bool(unsigned)> Func) const;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 102-104
```cpp
  void emitFPStatusSaves(MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
                         ArrayRef<CalleeSavedInfo> CSI,
                         unsigned PushOneOpc) const;
```
- EN: Declares `emitFPStatusSaves`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitFPStatusSaves`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 106-109
```cpp
  void emitFPStatusRestores(MachineBasicBlock &MBB,
                            MachineBasicBlock::iterator MI,
                            MutableArrayRef<CalleeSavedInfo> CSI,
                            unsigned LdrOpc) const;
```
- EN: Declares `emitFPStatusRestores`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitFPStatusRestores`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 111-115
```cpp
  MachineBasicBlock::iterator
  eliminateCallFramePseudoInstr(MachineFunction &MF,
                                MachineBasicBlock &MBB,
                                MachineBasicBlock::iterator MI) const override;
};
```
- EN: Declares `eliminateCallFramePseudoInstr`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `eliminateCallFramePseudoInstr`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 117-117
```cpp
} // end namespace llvm
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 119-119
```cpp
#endif // LLVM_LIB_TARGET_ARM_ARMFRAMELOWERING_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

## Key Concepts / 关键概念

- EN: Primary role: stack-frame layout and prologue/epilogue lowering.
  - CN: 核心职责：栈帧布局以及序言/结语生成。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: Stack-frame and spill/reload concerns are reflected in the API or implementation.
  - CN: API 或实现中体现了栈帧以及溢出/重载相关问题。

## Dependencies / 依赖关系

- EN: Backend-local headers: `(none)`.
  - CN: 后端本地头文件：`(none)`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/TargetFrameLowering.h`, `llvm/Support/TypeSize.h`.
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/TargetFrameLowering.h`, `llvm/Support/TypeSize.h`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
