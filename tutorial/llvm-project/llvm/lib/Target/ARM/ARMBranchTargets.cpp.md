# ARMBranchTargets.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMBranchTargets.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `ARMBranchTargets` for the ARM backend, focusing on branch-target validation and security-related branch handling.
- 用途 (CN): 实现 ARM 后端中的 `ARMBranchTargets`，重点处理分支目标校验与安全相关分支处理。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
//===-- ARMBranchTargets.cpp -- Harden code using v8.1-M BTI extension -----==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass inserts BTI instructions at the start of every function and basic
// block which could be indirectly called. The hardware will (when enabled)
// trap when an indirect branch or call instruction targets an instruction
// which is not a valid BTI instruction. This is intended to guard against
// control-flow hijacking attacks.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 17-24
```cpp
#include "ARM.h"
#include "ARMInstrInfo.h"
#include "ARMMachineFunctionInfo.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineJumpTableInfo.h"
#include "llvm/CodeGen/MachineModuleInfo.h"
#include "llvm/Support/Debug.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 26-26
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 28-29
```cpp
#define DEBUG_TYPE "arm-branch-targets"
#define ARM_BRANCH_TARGETS_NAME "ARM Branch Targets"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 31-38
```cpp
namespace {
class ARMBranchTargets : public MachineFunctionPass {
public:
  static char ID;
  ARMBranchTargets() : MachineFunctionPass(ID) {}
  void getAnalysisUsage(AnalysisUsage &AU) const override;
  bool runOnMachineFunction(MachineFunction &MF) override;
  StringRef getPassName() const override { return ARM_BRANCH_TARGETS_NAME; }
```
- EN: Declares `ARMBranchTargets`, packaging target-specific state and APIs around `ARMBranchTargets`.
- CN: 这里声明 `ARMBranchTargets`，把与 `ARMBranchTargets` 相关的目标特定状态和 API 组织在一起。

### Lines 40-43
```cpp
private:
  void addBTI(const ARMInstrInfo &TII, MachineBasicBlock &MBB, bool IsFirstBB);
};
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 45-45
```cpp
char ARMBranchTargets::ID = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 47-48
```cpp
INITIALIZE_PASS(ARMBranchTargets, "arm-branch-targets", ARM_BRANCH_TARGETS_NAME,
                false, false)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 50-53
```cpp
void ARMBranchTargets::getAnalysisUsage(AnalysisUsage &AU) const {
  AU.setPreservesCFG();
  MachineFunctionPass::getAnalysisUsage(AU);
}
```
- EN: Implements `ARMBranchTargets::getAnalysisUsage`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBranchTargets::getAnalysisUsage`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 55-57
```cpp
FunctionPass *llvm::createARMBranchTargetsPass() {
  return new ARMBranchTargets();
}
```
- EN: Implements `llvm::createARMBranchTargetsPass`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createARMBranchTargetsPass`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 59-61
```cpp
bool ARMBranchTargets::runOnMachineFunction(MachineFunction &MF) {
  if (!MF.getInfo<ARMFunctionInfo>()->branchTargetEnforcement())
    return false;
```
- EN: Implements `ARMBranchTargets::runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBranchTargets::runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 63-66
```cpp
  LLVM_DEBUG(dbgs() << "********** ARM Branch Targets  **********\n"
                    << "********** Function: " << MF.getName() << '\n');
  const ARMInstrInfo &TII =
      *static_cast<const ARMInstrInfo *>(MF.getSubtarget().getInstrInfo());
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 68-70
```cpp
  bool MadeChange = false;
  for (MachineBasicBlock &MBB : MF) {
    bool IsFirstBB = &MBB == &MF.front();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 72-78
```cpp
    // Every function can potentially be called indirectly (even if it has
    // static linkage, due to linker-generated veneers).
    // If the block itself is address-taken, or is an exception landing pad, it
    // could be indirectly branched to.
    // Jump tables only emit indirect jumps (JUMPTABLE_ADDRS) in ARM or Thumb1
    // modes. These modes do not support PACBTI. As a result, BTI instructions
    // are not added in the destination blocks.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 80-85
```cpp
    if (IsFirstBB || MBB.isMachineBlockAddressTaken() ||
        MBB.isIRBlockAddressTaken() || MBB.isEHPad()) {
      addBTI(TII, MBB, IsFirstBB);
      MadeChange = true;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 87-88
```cpp
  return MadeChange;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 90-100
```cpp
/// Insert a BTI/PACBTI instruction into a given basic block \c MBB. If
/// \c IsFirstBB is true (meaning that this is the first BB in a function) try
/// to find a PAC instruction and replace it with PACBTI. Otherwise just insert
/// a BTI instruction.
/// The point of insertion is in the beginning of the BB, immediately after meta
/// instructions (such labels in exception handling landing pads).
void ARMBranchTargets::addBTI(const ARMInstrInfo &TII, MachineBasicBlock &MBB,
                              bool IsFirstBB) {
  // Which instruction to insert: BTI or PACBTI
  unsigned OpCode = ARM::t2BTI;
  unsigned MIFlags = 0;
```
- EN: Implements `ARMBranchTargets::addBTI`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBranchTargets::addBTI`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 102-105
```cpp
  // Skip meta instructions, including EH labels
  auto MBBI = llvm::find_if_not(MBB.instrs(), [](const MachineInstr &MI) {
    return MI.isMetaInstruction();
  });
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 107-119
```cpp
  // If this is the first BB in a function, check if it starts with a PAC
  // instruction and in that case remove the PAC instruction.
  if (IsFirstBB) {
    if (MBBI != MBB.instr_end() && MBBI->getOpcode() == ARM::t2PAC) {
      LLVM_DEBUG(dbgs() << "Removing a 'PAC' instr from BB '" << MBB.getName()
                        << "' to replace with PACBTI\n");
      OpCode = ARM::t2PACBTI;
      MIFlags = MachineInstr::FrameSetup;
      auto NextMBBI = std::next(MBBI);
      MBBI->eraseFromParent();
      MBBI = NextMBBI;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 121-127
```cpp
  LLVM_DEBUG(dbgs() << "Inserting a '"
                    << (OpCode == ARM::t2BTI ? "BTI" : "PACBTI")
                    << "' instr into BB '" << MBB.getName() << "'\n");
  // Finally, insert a new instruction (either PAC or PACBTI)
  BuildMI(MBB, MBBI, MBB.findDebugLoc(MBBI), TII.get(OpCode))
      .setMIFlags(MIFlags);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

## Key Concepts / 关键概念

- EN: Primary role: branch-target validation and security-related branch handling.
  - CN: 核心职责：分支目标校验与安全相关分支处理。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: The component plugs into LLVM machine-function passes or codegen pipelines.
  - CN: 该组件会接入 LLVM 的机器函数 Pass 或代码生成流水线。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARM.h`, `ARMInstrInfo.h`, `ARMMachineFunctionInfo.h`.
  - CN: 后端本地头文件：`ARM.h`, `ARMInstrInfo.h`, `ARMMachineFunctionInfo.h`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineJumpTableInfo.h`, `llvm/CodeGen/MachineModuleInfo.h`, `llvm/Support/Debug.h`.
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineJumpTableInfo.h`, `llvm/CodeGen/MachineModuleInfo.h`, `llvm/Support/Debug.h`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
