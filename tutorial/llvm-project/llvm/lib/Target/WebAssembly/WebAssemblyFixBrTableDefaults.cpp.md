# WebAssemblyFixBrTableDefaults.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyFixBrTableDefaults.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: guarding br_table instructions. Since jump tables on most targets cannot handle out of range indices, LLVM emits these checks before most jump tables. But br_table takes a default branch target as an argument, so it does not need the range checks.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyFixBrTableDefaults.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//=- WebAssemblyFixBrTableDefaults.cpp - Fix br_table default branch targets -//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-14

```cpp
//===----------------------------------------------------------------------===//
///
/// \file This file implements a pass that eliminates redundant range checks
/// guarding br_table instructions. Since jump tables on most targets cannot
/// handle out of range indices, LLVM emits these checks before most jump
/// tables. But br_table takes a default branch target as an argument, so it
/// does not need the range checks.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file This file implements a pass that eliminates redundant range checks".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file This file implements a pass that eliminates redundant range checks”。

### Lines 15-21

```cpp
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "WebAssembly.h"
#include "WebAssemblySubtarget.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 22-28

```cpp
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/Pass.h"

using namespace llvm;

#define DEBUG_TYPE "wasm-fix-br-table-defaults"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 29-66

```cpp
namespace {

class WebAssemblyFixBrTableDefaults final : public MachineFunctionPass {
  StringRef getPassName() const override {
    return "WebAssembly Fix br_table Defaults";
  }

  bool runOnMachineFunction(MachineFunction &MF) override;

public:
  static char ID; // Pass identification, replacement for typeid
  WebAssemblyFixBrTableDefaults() : MachineFunctionPass(ID) {}
};

char WebAssemblyFixBrTableDefaults::ID = 0;

// Target independent selection dag assumes that it is ok to use PointerTy
// as the index for a "switch", whereas Wasm so far only has a 32-bit br_table.
// See e.g. SelectionDAGBuilder::visitJumpTableHeader
// We have a 64-bit br_table in the tablegen defs as a result, which does get
// selected, and thus we get incorrect truncates/extensions happening on
// wasm64. Here we fix that.
void fixBrTableIndex(MachineInstr &MI, MachineBasicBlock *MBB,
                     MachineFunction &MF) {
  // Only happens on wasm64.
  auto &WST = MF.getSubtarget<WebAssemblySubtarget>();
  if (!WST.hasAddr64())
    return;

  assert(MI.getDesc().getOpcode() == WebAssembly::BR_TABLE_I64 &&
         "64-bit br_table pseudo instruction expected");

  // Find extension op, if any. It sits in the previous BB before the branch.
  auto ExtMI = MF.getRegInfo().getVRegDef(MI.getOperand(0).getReg());
  if (ExtMI->getOpcode() == WebAssembly::I64_EXTEND_U_I32) {
    // Unnecessarily extending a 32-bit value to 64, remove it.
    auto ExtDefReg = ExtMI->getOperand(0).getReg();
    assert(MI.getOperand(0).getReg() == ExtDefReg);
```
- **EN**: Declares a backend-facing type `WebAssemblyFixBrTableDefaults`, `getPassName`, `runOnMachineFunction` and outlines the API or state that nearby code will rely on. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明面向后端的类型 `WebAssemblyFixBrTableDefaults`, `getPassName`, `runOnMachineFunction`，并勾勒出周边代码会依赖的接口或状态。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 67-104

```cpp
    MI.getOperand(0).setReg(ExtMI->getOperand(1).getReg());
    if (MF.getRegInfo().use_nodbg_empty(ExtDefReg)) {
      // No more users of extend, delete it.
      ExtMI->eraseFromParent();
    }
  } else {
    // Incoming 64-bit value that needs to be truncated.
    Register Reg32 =
        MF.getRegInfo().createVirtualRegister(&WebAssembly::I32RegClass);
    BuildMI(*MBB, MI.getIterator(), MI.getDebugLoc(),
            WST.getInstrInfo()->get(WebAssembly::I32_WRAP_I64), Reg32)
        .addReg(MI.getOperand(0).getReg());
    MI.getOperand(0).setReg(Reg32);
  }

  // We now have a 32-bit operand in all cases, so change the instruction
  // accordingly.
  MI.setDesc(WST.getInstrInfo()->get(WebAssembly::BR_TABLE_I32));
}

// `MI` is a br_table instruction with a dummy default target argument. This
// function finds and adds the default target argument and removes any redundant
// range check preceding the br_table. Returns the MBB that the br_table is
// moved into so it can be removed from further consideration, or nullptr if the
// br_table cannot be optimized.
MachineBasicBlock *fixBrTableDefault(MachineInstr &MI, MachineBasicBlock *MBB,
                                     MachineFunction &MF) {
  // Get the header block, which contains the redundant range check.
  assert(MBB->pred_size() == 1 && "Expected a single guard predecessor");
  auto *HeaderMBB = *MBB->pred_begin();

  // Find the conditional jump to the default target. If it doesn't exist, the
  // default target is unreachable anyway, so we can keep the existing dummy
  // target.
  MachineBasicBlock *TBB = nullptr, *FBB = nullptr;
  SmallVector<MachineOperand, 2> Cond;
  const auto &TII = *MF.getSubtarget<WebAssemblySubtarget>().getInstrInfo();
  bool Analyzed = !TII.analyzeBranch(*HeaderMBB, TBB, FBB, Cond);
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "No more users of extend, delete it.". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“No more users of extend, delete it.”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 105-142

```cpp
  assert(Analyzed && "Could not analyze jump header branches");
  (void)Analyzed;

  // Here are the possible outcomes. '_' is nullptr, `J` is the jump table block
  // aka MBB, 'D' is the default block.
  //
  // TBB | FBB | Meaning
  //  _  |  _  | No default block, header falls through to jump table
  //  J  |  _  | No default block, header jumps to the jump table
  //  D  |  _  | Header jumps to the default and falls through to the jump table
  //  D  |  J  | Header jumps to the default and also to the jump table
  if (TBB && TBB != MBB) {
    assert((FBB == nullptr || FBB == MBB) &&
           "Expected jump or fallthrough to br_table block");
    assert(Cond.size() == 2 && Cond[1].isReg() && "Unexpected condition info");

    // If the range check checks an i64 value, we cannot optimize it out because
    // the i64 index is truncated to an i32, making values over 2^32
    // indistinguishable from small numbers. There are also other strange edge
    // cases that can arise in practice that we don't want to reason about, so
    // conservatively only perform the optimization if the range check is the
    // normal case of an i32.gt_u.
    MachineRegisterInfo &MRI = MF.getRegInfo();
    auto *RangeCheck = MRI.getVRegDef(Cond[1].getReg());
    assert(RangeCheck != nullptr);
    if (RangeCheck->getOpcode() != WebAssembly::GT_U_I32)
      return nullptr;

    // Remove the dummy default target and install the real one.
    MI.removeOperand(MI.getNumExplicitOperands() - 1);
    MI.addOperand(MF, MachineOperand::CreateMBB(TBB));
  }

  // Remove any branches from the header and splice in the jump table instead
  TII.removeBranch(*HeaderMBB, nullptr);
  HeaderMBB->splice(HeaderMBB->end(), MBB, MBB->begin(), MBB->end());

  // Update CFG to skip the old jump table block. Remove shared successors
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Here are the possible outcomes. '_' is nullptr, `J` is the jump table block". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Here are the possible outcomes. '_' is nullptr, `J` is the jump table block”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 143-180

```cpp
  // before transferring to avoid duplicated successors.
  HeaderMBB->removeSuccessor(MBB);
  for (auto &Succ : MBB->successors())
    if (HeaderMBB->isSuccessor(Succ))
      HeaderMBB->removeSuccessor(Succ);
  HeaderMBB->transferSuccessorsAndUpdatePHIs(MBB);

  // Remove the old jump table block from the function
  MF.erase(MBB);

  return HeaderMBB;
}

bool WebAssemblyFixBrTableDefaults::runOnMachineFunction(MachineFunction &MF) {
  LLVM_DEBUG(dbgs() << "********** Fixing br_table Default Targets **********\n"
                       "********** Function: "
                    << MF.getName() << '\n');

  bool Changed = false;
  SetVector<MachineBasicBlock *, SmallVector<MachineBasicBlock *, 16>,
            DenseSet<MachineBasicBlock *>, 16>
      MBBSet;
  for (auto &MBB : MF)
    MBBSet.insert(&MBB);

  while (!MBBSet.empty()) {
    MachineBasicBlock *MBB = *MBBSet.begin();
    MBBSet.remove(MBB);
    for (auto &MI : *MBB) {
      if (WebAssembly::isBrTable(MI.getOpcode())) {
        fixBrTableIndex(MI, MBB, MF);
        auto *Fixed = fixBrTableDefault(MI, MBB, MF);
        if (Fixed != nullptr) {
          MBBSet.remove(Fixed);
          Changed = true;
        }
        break;
      }
```
- **EN**: Implements helper routine(s) `removeSuccessor`, `successors`, `isSuccessor` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `removeSuccessor`, `successors`, `isSuccessor`。

### Lines 181-201

```cpp
    }
  }

  if (Changed) {
    // We rewrote part of the function; recompute relevant things.
    MF.RenumberBlocks();
    return true;
  }

  return false;
}

} // end anonymous namespace

INITIALIZE_PASS(WebAssemblyFixBrTableDefaults, DEBUG_TYPE,
                "Removes range checks and sets br_table default targets", false,
                false)

FunctionPass *llvm::createWebAssemblyFixBrTableDefaults() {
  return new WebAssemblyFixBrTableDefaults();
}
```
- **EN**: Implements helper routine(s) `RenumberBlocks`, `INITIALIZE_PASS`, `createWebAssemblyFixBrTableDefaults` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `RenumberBlocks`, `INITIALIZE_PASS`, `createWebAssemblyFixBrTableDefaults`。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- SelectionDAG lowering / SelectionDAG lowering
- Register modeling / 寄存器建模
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Object format integration / 目标文件格式集成
- Vector or SIMD behavior / 向量或 SIMD 行为

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `WebAssembly.h`
- `WebAssemblySubtarget.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/Pass.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
