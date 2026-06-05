# PPCEarlyReturn.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCEarlyReturn.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides backend implementation logic for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCEarlyReturn.cpp`，主要负责 PowerPC 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===------------- PPCEarlyReturn.cpp - Form Early Returns ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-11

```cpp
//===----------------------------------------------------------------------===//
//
// A pass that form early (predicated) returns. If-conversion handles some of
// this, but this pass picks up some remaining cases.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "A pass that form early (predicated) returns. If-conversion handles some of". Notable symbols in this range include `early`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“A pass that form early (predicated) returns. If-conversion handles some of”。 该区间中较显眼的符号包括 `early`。

### Lines 12-18

```cpp
//===----------------------------------------------------------------------===//

#include "PPC.h"
#include "PPCInstrInfo.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 该区间与栈帧布局或栈访问相关。

### Lines 19-25

```cpp
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineMemOperand.h"
#include "llvm/Support/ErrorHandling.h"

using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 26-33

```cpp
#define DEBUG_TYPE "ppc-early-ret"
STATISTIC(NumBCLR, "Number of early conditional returns");
STATISTIC(NumBLR,  "Number of early returns");

namespace {
  // PPCEarlyReturn pass - For simple functions without epilogue code, move
  // returns up, and create conditional returns, to avoid unnecessary
  // branch-to-blr sequences.
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `STATISTIC`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `STATISTIC`。

### Lines 34-71

```cpp
  struct PPCEarlyReturn : public MachineFunctionPass {
    static char ID;
    PPCEarlyReturn() : MachineFunctionPass(ID) {}

    const TargetInstrInfo *TII;

protected:
    bool processBlock(MachineBasicBlock &ReturnMBB) {
      bool Changed = false;

      MachineBasicBlock::iterator I = ReturnMBB.begin();
      I = ReturnMBB.SkipPHIsLabelsAndDebug(I);

      // The block must be essentially empty except for the blr.
      if (I == ReturnMBB.end() ||
          (I->getOpcode() != PPC::BLR && I->getOpcode() != PPC::BLR8) ||
          I != ReturnMBB.getLastNonDebugInstr())
        return Changed;

      SmallVector<MachineBasicBlock*, 8> PredToRemove;
      for (MachineBasicBlock *Pred : ReturnMBB.predecessors()) {
        bool OtherReference = false, BlockChanged = false;

        if (Pred->empty())
          continue;

        for (MachineBasicBlock::iterator J = Pred->getLastNonDebugInstr();;) {
          if (J == Pred->end())
            break;

          if (J->getOpcode() == PPC::B) {
            if (J->getOperand(0).getMBB() == &ReturnMBB) {
              // This is an unconditional branch to the return. Replace the
              // branch with a blr.
              MachineInstr *MI = ReturnMBB.getParent()->CloneMachineInstr(&*I);
              Pred->insert(J, MI);

              MachineBasicBlock::iterator K = J--;
```
- **EN**: Implements helper routine(s) `PPCEarlyReturn`, `MachineFunctionPass`, `processBlock` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `PPCEarlyReturn`, `MachineFunctionPass`, `processBlock`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 72-109

```cpp
              K->eraseFromParent();
              BlockChanged = true;
              ++NumBLR;
              continue;
            }
          } else if (J->getOpcode() == PPC::BCC) {
            if (J->getOperand(2).getMBB() == &ReturnMBB) {
              // This is a conditional branch to the return. Replace the branch
              // with a bclr.
              MachineInstr *MI = ReturnMBB.getParent()->CloneMachineInstr(&*I);
              MI->setDesc(TII->get(PPC::BCCLR));
              MachineInstrBuilder(*ReturnMBB.getParent(), MI)
                  .add(J->getOperand(0))
                  .add(J->getOperand(1));
              Pred->insert(J, MI);

              MachineBasicBlock::iterator K = J--;
              K->eraseFromParent();
              BlockChanged = true;
              ++NumBCLR;
              continue;
            }
          } else if (J->getOpcode() == PPC::BC || J->getOpcode() == PPC::BCn) {
            if (J->getOperand(1).getMBB() == &ReturnMBB) {
              // This is a conditional branch to the return. Replace the branch
              // with a bclr.
              MachineInstr *MI = ReturnMBB.getParent()->CloneMachineInstr(&*I);
              MI->setDesc(
                  TII->get(J->getOpcode() == PPC::BC ? PPC::BCLR : PPC::BCLRn));
              MachineInstrBuilder(*ReturnMBB.getParent(), MI)
                  .add(J->getOperand(0));
              Pred->insert(J, MI);

              MachineBasicBlock::iterator K = J--;
              K->eraseFromParent();
              BlockChanged = true;
              ++NumBCLR;
              continue;
```
- **EN**: Implements helper routine(s) `eraseFromParent`, `getOpcode`, `getOperand` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `eraseFromParent`, `getOpcode`, `getOperand`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 110-147

```cpp
            }
          } else if (J->isBranch()) {
            if (J->isIndirectBranch()) {
              if (ReturnMBB.hasAddressTaken())
                OtherReference = true;
            } else
              for (unsigned i = 0; i < J->getNumOperands(); ++i)
                if (J->getOperand(i).isMBB() &&
                    J->getOperand(i).getMBB() == &ReturnMBB)
                  OtherReference = true;
          } else if (!J->isTerminator() && !J->isDebugInstr())
            break;

          if (J == Pred->begin())
            break;

          --J;
        }

        if (Pred->canFallThrough() && Pred->isLayoutSuccessor(&ReturnMBB))
          OtherReference = true;

        // Predecessors are stored in a vector and can't be removed here.
        if (!OtherReference && BlockChanged) {
          PredToRemove.push_back(Pred);
        }

        if (BlockChanged)
          Changed = true;
      }

      for (MachineBasicBlock *MBB : PredToRemove)
        MBB->removeSuccessor(&ReturnMBB, true);

      if (Changed && !ReturnMBB.hasAddressTaken()) {
        // We now might be able to merge this blr-only block into its
        // by-layout predecessor.
        if (ReturnMBB.pred_size() == 1) {
```
- **EN**: Implements helper routine(s) `isBranch`, `isIndirectBranch`, `hasAddressTaken` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `isBranch`, `isIndirectBranch`, `hasAddressTaken`。

### Lines 148-185

```cpp
          MachineBasicBlock &PrevMBB = **ReturnMBB.pred_begin();
          if (PrevMBB.isLayoutSuccessor(&ReturnMBB) && PrevMBB.canFallThrough()) {
            // Move the blr into the preceding block.
            PrevMBB.splice(PrevMBB.end(), &ReturnMBB, I);
            PrevMBB.removeSuccessor(&ReturnMBB, true);
          }
        }

        if (ReturnMBB.pred_empty())
          ReturnMBB.eraseFromParent();
      }

      return Changed;
    }

public:
    bool runOnMachineFunction(MachineFunction &MF) override {
      if (skipFunction(MF.getFunction()))
        return false;

      TII = MF.getSubtarget().getInstrInfo();

      bool Changed = false;

      // If the function does not have at least two blocks, then there is
      // nothing to do.
      if (MF.size() < 2)
        return Changed;

      for (MachineBasicBlock &B : llvm::make_early_inc_range(MF))
        Changed |= processBlock(B);

      return Changed;
    }

    MachineFunctionProperties getRequiredProperties() const override {
      return MachineFunctionProperties().setNoVRegs();
    }
```
- **EN**: Implements helper routine(s) `pred_begin`, `isLayoutSuccessor`, `canFallThrough` for this portion of the PowerPC backend backend implementation logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `pred_begin`, `isLayoutSuccessor`, `canFallThrough`。 子目标特性裁剪会影响这里的行为。

### Lines 186-198

```cpp

    void getAnalysisUsage(AnalysisUsage &AU) const override {
      MachineFunctionPass::getAnalysisUsage(AU);
    }
  };
}

INITIALIZE_PASS(PPCEarlyReturn, DEBUG_TYPE,
                "PowerPC Early-Return Creation", false, false)

char PPCEarlyReturn::ID = 0;
FunctionPass*
llvm::createPPCEarlyReturnPass() { return new PPCEarlyReturn(); }
```
- **EN**: Implements helper routine(s) `getAnalysisUsage`, `INITIALIZE_PASS`, `createPPCEarlyReturnPass` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getAnalysisUsage`, `INITIALIZE_PASS`, `createPPCEarlyReturnPass`。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Backend implementation logic / 后端实现逻辑
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPC.h`
- `PPCInstrInfo.h`
- `llvm/ADT/STLExtras.h`
- `llvm/ADT/Statistic.h`
- `llvm/CodeGen/MachineFrameInfo.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineMemOperand.h`
- `llvm/Support/ErrorHandling.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
