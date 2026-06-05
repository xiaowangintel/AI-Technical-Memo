# WebAssemblyLowerBrUnless.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyLowerBrUnless.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file lowers br_unless into br_if with an inverted condition.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyLowerBrUnless.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyLowerBrUnless.cpp - Lower br_unless --------------------===//
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
/// \file
/// This file lowers br_unless into br_if with an inverted condition.
///
/// br_unless is not currently in the spec, but it's very convenient for LLVM
/// to use. This pass allows LLVM to use it, for now.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 15-21

```cpp
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "WebAssembly.h"
#include "WebAssemblyMachineFunctionInfo.h"
#include "WebAssemblySubtarget.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 22-28

```cpp
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;

#define DEBUG_TYPE "wasm-lower-br_unless"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 29-66

```cpp
namespace {
class WebAssemblyLowerBrUnless final : public MachineFunctionPass {
  StringRef getPassName() const override {
    return "WebAssembly Lower br_unless";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  bool runOnMachineFunction(MachineFunction &MF) override;

public:
  static char ID; // Pass identification, replacement for typeid
  WebAssemblyLowerBrUnless() : MachineFunctionPass(ID) {}
};
} // end anonymous namespace

char WebAssemblyLowerBrUnless::ID = 0;
INITIALIZE_PASS(WebAssemblyLowerBrUnless, DEBUG_TYPE,
                "Lowers br_unless into inverted br_if", false, false)

FunctionPass *llvm::createWebAssemblyLowerBrUnless() {
  return new WebAssemblyLowerBrUnless();
}

bool WebAssemblyLowerBrUnless::runOnMachineFunction(MachineFunction &MF) {
  LLVM_DEBUG(dbgs() << "********** Lowering br_unless **********\n"
                       "********** Function: "
                    << MF.getName() << '\n');

  auto &MFI = *MF.getInfo<WebAssemblyFunctionInfo>();
  const auto &TII = *MF.getSubtarget<WebAssemblySubtarget>().getInstrInfo();
  auto &MRI = MF.getRegInfo();

  for (auto &MBB : MF) {
    for (MachineInstr &MI : llvm::make_early_inc_range(MBB)) {
```
- **EN**: Declares a backend-facing type `WebAssemblyLowerBrUnless`, `getPassName`, `getAnalysisUsage` and outlines the API or state that nearby code will rely on. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明面向后端的类型 `WebAssemblyLowerBrUnless`, `getPassName`, `getAnalysisUsage`，并勾勒出周边代码会依赖的接口或状态。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 67-104

```cpp
      if (MI.getOpcode() != WebAssembly::BR_UNLESS)
        continue;

      Register Cond = MI.getOperand(1).getReg();
      bool Inverted = false;

      // Attempt to invert the condition in place.
      if (MFI.isVRegStackified(Cond)) {
        assert(MRI.hasOneDef(Cond));
        MachineInstr *Def = MRI.getVRegDef(Cond);
        switch (Def->getOpcode()) {
          using namespace WebAssembly;
        case EQ_I32:
          Def->setDesc(TII.get(NE_I32));
          Inverted = true;
          break;
        case NE_I32:
          Def->setDesc(TII.get(EQ_I32));
          Inverted = true;
          break;
        case GT_S_I32:
          Def->setDesc(TII.get(LE_S_I32));
          Inverted = true;
          break;
        case GE_S_I32:
          Def->setDesc(TII.get(LT_S_I32));
          Inverted = true;
          break;
        case LT_S_I32:
          Def->setDesc(TII.get(GE_S_I32));
          Inverted = true;
          break;
        case LE_S_I32:
          Def->setDesc(TII.get(GT_S_I32));
          Inverted = true;
          break;
        case GT_U_I32:
          Def->setDesc(TII.get(LE_U_I32));
```
- **EN**: Implements helper routine(s) `getOpcode`, `getOperand`, `getReg` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getOpcode`, `getOperand`, `getReg`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 105-142

```cpp
          Inverted = true;
          break;
        case GE_U_I32:
          Def->setDesc(TII.get(LT_U_I32));
          Inverted = true;
          break;
        case LT_U_I32:
          Def->setDesc(TII.get(GE_U_I32));
          Inverted = true;
          break;
        case LE_U_I32:
          Def->setDesc(TII.get(GT_U_I32));
          Inverted = true;
          break;
        case EQ_I64:
          Def->setDesc(TII.get(NE_I64));
          Inverted = true;
          break;
        case NE_I64:
          Def->setDesc(TII.get(EQ_I64));
          Inverted = true;
          break;
        case GT_S_I64:
          Def->setDesc(TII.get(LE_S_I64));
          Inverted = true;
          break;
        case GE_S_I64:
          Def->setDesc(TII.get(LT_S_I64));
          Inverted = true;
          break;
        case LT_S_I64:
          Def->setDesc(TII.get(GE_S_I64));
          Inverted = true;
          break;
        case LE_S_I64:
          Def->setDesc(TII.get(GT_S_I64));
          Inverted = true;
          break;
```
- **EN**: Declares function entry points including `setDesc`, `get` that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `setDesc`, `get`。

### Lines 143-180

```cpp
        case GT_U_I64:
          Def->setDesc(TII.get(LE_U_I64));
          Inverted = true;
          break;
        case GE_U_I64:
          Def->setDesc(TII.get(LT_U_I64));
          Inverted = true;
          break;
        case LT_U_I64:
          Def->setDesc(TII.get(GE_U_I64));
          Inverted = true;
          break;
        case LE_U_I64:
          Def->setDesc(TII.get(GT_U_I64));
          Inverted = true;
          break;
        case EQ_F32:
          Def->setDesc(TII.get(NE_F32));
          Inverted = true;
          break;
        case NE_F32:
          Def->setDesc(TII.get(EQ_F32));
          Inverted = true;
          break;
        case EQ_F64:
          Def->setDesc(TII.get(NE_F64));
          Inverted = true;
          break;
        case NE_F64:
          Def->setDesc(TII.get(EQ_F64));
          Inverted = true;
          break;
        case EQZ_I32: {
          // Invert an eqz by replacing it with its operand.
          Cond = Def->getOperand(1).getReg();
          Def->eraseFromParent();
          Inverted = true;
          break;
```
- **EN**: Implements helper routine(s) `setDesc`, `get`, `getOperand` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `setDesc`, `get`, `getOperand`。

### Lines 181-209

```cpp
        }
        default:
          break;
        }
      }

      // If we weren't able to invert the condition in place. Insert an
      // instruction to invert it.
      if (!Inverted) {
        Register Tmp = MRI.createVirtualRegister(&WebAssembly::I32RegClass);
        BuildMI(MBB, &MI, MI.getDebugLoc(), TII.get(WebAssembly::EQZ_I32), Tmp)
            .addReg(Cond);
        MFI.stackifyVReg(MRI, Tmp);
        Cond = Tmp;
        Inverted = true;
      }

      // The br_unless condition has now been inverted. Insert a br_if and
      // delete the br_unless.
      assert(Inverted);
      BuildMI(MBB, &MI, MI.getDebugLoc(), TII.get(WebAssembly::BR_IF))
          .add(MI.getOperand(0))
          .addReg(Cond);
      MBB.erase(&MI);
    }
  }

  return true;
}
```
- **EN**: Implements helper routine(s) `createVirtualRegister`, `BuildMI`, `getDebugLoc` for this portion of the WebAssembly backend backend implementation logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `createVirtualRegister`, `BuildMI`, `getDebugLoc`。 这里重点涉及寄存器分配与寄存器类约束。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `WebAssembly.h`
- `WebAssemblyMachineFunctionInfo.h`
- `WebAssemblySubtarget.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/Support/Debug.h`
- `llvm/Support/raw_ostream.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
