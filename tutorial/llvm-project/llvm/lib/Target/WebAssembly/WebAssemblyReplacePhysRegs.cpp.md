# WebAssemblyReplacePhysRegs.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyReplacePhysRegs.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements a pass that replaces physical registers with virtual registers.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyReplacePhysRegs.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyReplacePhysRegs.cpp - Replace phys regs with virt regs -===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. Register assignment and register-class constraints matter here.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 7-18

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements a pass that replaces physical registers with
/// virtual registers.
///
/// LLVM expects certain physical registers, such as a stack pointer. However,
/// WebAssembly doesn't actually have such physical registers. This pass is run
/// once LLVM no longer needs these registers, and replaces them with virtual
/// registers, so they can participate in register stackifying and coloring in
/// the normal way.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 19-25

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

### Lines 26-33

```cpp
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;

#define DEBUG_TYPE "wasm-replace-phys-regs"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 34-71

```cpp
namespace {
class WebAssemblyReplacePhysRegs final : public MachineFunctionPass {
public:
  static char ID; // Pass identification, replacement for typeid
  WebAssemblyReplacePhysRegs() : MachineFunctionPass(ID) {}

private:
  StringRef getPassName() const override {
    return "WebAssembly Replace Physical Registers";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  bool runOnMachineFunction(MachineFunction &MF) override;
};
} // end anonymous namespace

char WebAssemblyReplacePhysRegs::ID = 0;
INITIALIZE_PASS(WebAssemblyReplacePhysRegs, DEBUG_TYPE,
                "Replace physical registers with virtual registers", false,
                false)

FunctionPass *llvm::createWebAssemblyReplacePhysRegs() {
  return new WebAssemblyReplacePhysRegs();
}

bool WebAssemblyReplacePhysRegs::runOnMachineFunction(MachineFunction &MF) {
  LLVM_DEBUG({
    dbgs() << "********** Replace Physical Registers **********\n"
           << "********** Function: " << MF.getName() << '\n';
  });

  MachineRegisterInfo &MRI = MF.getRegInfo();
  auto &TRI = *MF.getSubtarget<WebAssemblySubtarget>().getRegisterInfo();
  bool Changed = false;
```
- **EN**: Declares a backend-facing type `WebAssemblyReplacePhysRegs`, `MachineFunctionPass`, `getPassName` and outlines the API or state that nearby code will rely on. Register assignment and register-class constraints matter here.
- **CN**: 这里声明面向后端的类型 `WebAssemblyReplacePhysRegs`, `MachineFunctionPass`, `getPassName`，并勾勒出周边代码会依赖的接口或状态。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 72-107

```cpp

  assert(!mustPreserveAnalysisID(LiveIntervalsID) &&
         "LiveIntervals shouldn't be active yet!");

  for (unsigned PReg = WebAssembly::NoRegister + 1;
       PReg < WebAssembly::NUM_TARGET_REGS; ++PReg) {
    // Skip fake registers that are never used explicitly.
    if (PReg == WebAssembly::VALUE_STACK || PReg == WebAssembly::ARGUMENTS)
      continue;

    // Replace explicit uses of the physical register with a virtual register.
    const TargetRegisterClass *RC = TRI.getMinimalPhysRegClass(PReg);
    unsigned VReg = WebAssembly::NoRegister;
    for (MachineOperand &MO :
         llvm::make_early_inc_range(MRI.reg_operands(PReg))) {
      if (!MO.isImplicit()) {
        if (VReg == WebAssembly::NoRegister) {
          VReg = MRI.createVirtualRegister(RC);
          if (PReg == TRI.getFrameRegister(MF)) {
            auto FI = MF.getInfo<WebAssemblyFunctionInfo>();
            assert(!FI->isFrameBaseVirtual());
            FI->setFrameBaseVreg(VReg);
            LLVM_DEBUG({
              dbgs() << "replacing preg " << PReg << " with " << VReg << " ("
                     << Register(VReg).virtRegIndex() << ")\n";
            });
          }
        }
        MO.setReg(VReg);
        Changed = true;
      }
    }
  }

  return Changed;
}
```
- **EN**: Implements helper routine(s) `mustPreserveAnalysisID`, `getMinimalPhysRegClass`, `make_early_inc_range` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `mustPreserveAnalysisID`, `getMinimalPhysRegClass`, `make_early_inc_range`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- Register modeling / 寄存器建模
- Subtarget features / 子目标特性
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `WebAssembly.h`
- `WebAssemblyMachineFunctionInfo.h`
- `WebAssemblySubtarget.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/CodeGen/Passes.h`
- `llvm/Support/Debug.h`
- `llvm/Support/raw_ostream.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
