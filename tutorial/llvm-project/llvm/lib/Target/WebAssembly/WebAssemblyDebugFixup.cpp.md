# WebAssemblyDebugFixup.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyDebugFixup.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Several prior passes may "stackify" registers, here we ensure any references in such registers in debug_value instructions become stack relative also. This is done in a separate pass such that not all previous passes need to track stack depth when values get stackified.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyDebugFixup.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyDebugFixup.cpp - Debug Fixup ------------------===//
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
/// Several prior passes may "stackify" registers, here we ensure any references
/// in such registers in debug_value instructions become stack relative also.
/// This is done in a separate pass such that not all previous passes need to
/// track stack depth when values get stackified.
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
#include "WebAssemblyUtilities.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 22-29

```cpp
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 30-67

```cpp
#define DEBUG_TYPE "wasm-debug-fixup"

namespace {
class WebAssemblyDebugFixup final : public MachineFunctionPass {
  StringRef getPassName() const override { return "WebAssembly Debug Fixup"; }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  bool runOnMachineFunction(MachineFunction &MF) override;

public:
  static char ID; // Pass identification, replacement for typeid
  WebAssemblyDebugFixup() : MachineFunctionPass(ID) {}
};
} // end anonymous namespace

char WebAssemblyDebugFixup::ID = 0;
INITIALIZE_PASS(
    WebAssemblyDebugFixup, DEBUG_TYPE,
    "Ensures debug_value's that have been stackified become stack relative",
    false, false)

FunctionPass *llvm::createWebAssemblyDebugFixup() {
  return new WebAssemblyDebugFixup();
}

// At this very end of the compilation pipeline, if any DBG_VALUEs with
// registers remain, it means they are dangling info which we failed to update
// when their corresponding def instruction was transformed/moved/splitted etc.
// Because Wasm cannot access values in LLVM virtual registers in the debugger,
// these dangling DBG_VALUEs in effect kill the effect of any previous DBG_VALUE
// associated with the variable, which will appear as "optimized out".
static void setDanglingDebugValuesUndef(MachineBasicBlock &MBB,
                                        const TargetInstrInfo *TII) {
  for (auto &MI : llvm::make_early_inc_range(MBB)) {
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `WebAssemblyDebugFixup`, `getPassName`, `getAnalysisUsage`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `WebAssemblyDebugFixup`, `getPassName`, `getAnalysisUsage`。

### Lines 68-84

```cpp
    if (MI.isDebugValue() && MI.getDebugOperand(0).isReg() &&
        !MI.isUndefDebugValue()) {
      LLVM_DEBUG(dbgs() << "Warning: dangling DBG_VALUE set to undef: " << MI
                        << "\n");
      MI.setDebugValueUndef();
    }
  }
}

bool WebAssemblyDebugFixup::runOnMachineFunction(MachineFunction &MF) {
  LLVM_DEBUG(dbgs() << "********** Debug Fixup **********\n"
                       "********** Function: "
                    << MF.getName() << '\n');

  WebAssemblyFunctionInfo &MFI = *MF.getInfo<WebAssemblyFunctionInfo>();
  const auto *TII = MF.getSubtarget<WebAssemblySubtarget>().getInstrInfo();
```
- **EN**: Implements helper routine(s) `isDebugValue`, `getDebugOperand`, `isReg` for this portion of the WebAssembly backend backend implementation logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `isDebugValue`, `getDebugOperand`, `isReg`。 子目标特性裁剪会影响这里的行为。

### Lines 85-122

```cpp
  struct StackElem {
    unsigned Reg;
    MachineInstr *DebugValue;
  };
  std::vector<StackElem> Stack;
  for (MachineBasicBlock &MBB : MF) {
    // We may insert into this list.
    for (auto MII = MBB.begin(); MII != MBB.end(); ++MII) {
      MachineInstr &MI = *MII;
      if (MI.isDebugValue()) {
        auto &MO = MI.getOperand(0);
        // Also check if not a $noreg: likely a DBG_VALUE we just inserted.
        if (MO.isReg() && MO.getReg().isValid() &&
            MFI.isVRegStackified(MO.getReg())) {
          // Found a DBG_VALUE with a stackified register we will
          // change into a stack operand.
          // Search for register rather than assume it is on top (which it
          // typically is if it appears right after the def), since
          // DBG_VALUE's may shift under some circumstances.
          for (auto &Elem : reverse(Stack)) {
            if (MO.getReg() == Elem.Reg) {
              auto Depth = static_cast<unsigned>(&Elem - &Stack[0]);
              LLVM_DEBUG(dbgs() << "Debug Value VReg " << printReg(MO.getReg())
                                << " -> Stack Relative " << Depth << "\n");
              MO.ChangeToTargetIndex(WebAssembly::TI_OPERAND_STACK, Depth);
              // Save the DBG_VALUE instruction that defined this stackified
              // variable since later we need it to construct another one on
              // pop.
              Elem.DebugValue = &MI;
              break;
            }
          }
          // If the Reg was not found, we have a DBG_VALUE outside of its
          // def-use range, and we leave it unmodified as reg, which means
          // it will be culled later.
        }
      } else {
        // Track stack depth.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "We may insert into this list.". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“We may insert into this list.”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 123-158

```cpp
        for (MachineOperand &MO : reverse(MI.explicit_uses())) {
          if (MO.isReg() && MFI.isVRegStackified(MO.getReg())) {
            auto Prev = Stack.back();
            Stack.pop_back();
            assert(Prev.Reg == MO.getReg() &&
                   "WebAssemblyDebugFixup: Pop: Register not matched!");
            // We should not put a DBG_VALUE after a terminator; debug ranges
            // are terminated at the end of a BB anyway.
            if (Prev.DebugValue && !MI.isTerminator()) {
              // This stackified reg is a variable that started life at
              // Prev.DebugValue, so now that we're popping it we must insert
              // a $noreg DBG_VALUE for the variable to end it, right after
              // the current instruction.
              BuildMI(*Prev.DebugValue->getParent(), std::next(MII),
                      Prev.DebugValue->getDebugLoc(),
                      TII->get(WebAssembly::DBG_VALUE), false, Register(),
                      Prev.DebugValue->getOperand(2).getMetadata(),
                      Prev.DebugValue->getOperand(3).getMetadata());
            }
          }
        }
        for (MachineOperand &MO : MI.defs()) {
          if (MO.isReg() && MFI.isVRegStackified(MO.getReg())) {
            Stack.push_back({MO.getReg(), nullptr});
          }
        }
      }
    }
    assert(Stack.empty() &&
           "WebAssemblyDebugFixup: Stack not empty at end of basic block!");

    setDanglingDebugValuesUndef(MBB, TII);
  }

  return true;
}
```
- **EN**: Implements helper routine(s) `reverse`, `explicit_uses`, `isReg` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `reverse`, `explicit_uses`, `isReg`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Object format integration / 目标文件格式集成
- Declarative TableGen records / 声明式 TableGen 记录
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `WebAssembly.h`
- `WebAssemblyMachineFunctionInfo.h`
- `WebAssemblySubtarget.h`
- `WebAssemblyUtilities.h`
- `llvm/CodeGen/MachineFrameInfo.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/Passes.h`
- `llvm/Support/Debug.h`
- `llvm/Support/raw_ostream.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
