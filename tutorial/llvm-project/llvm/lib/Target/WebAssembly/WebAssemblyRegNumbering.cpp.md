# WebAssemblyRegNumbering.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyRegNumbering.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements a pass which assigns WebAssembly register numbers for CodeGen virtual registers.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyRegNumbering.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyRegNumbering.cpp - Register Numbering ------------------===//
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
/// This file implements a pass which assigns WebAssembly register
/// numbers for CodeGen virtual registers.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 13-19

```cpp
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "WebAssembly.h"
#include "WebAssemblyMachineFunctionInfo.h"
#include "WebAssemblyUtilities.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 该区间与栈帧布局或栈访问相关。

### Lines 20-26

```cpp
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 27-64

```cpp
#define DEBUG_TYPE "wasm-reg-numbering"

namespace {
class WebAssemblyRegNumbering final : public MachineFunctionPass {
  StringRef getPassName() const override {
    return "WebAssembly Register Numbering";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  bool runOnMachineFunction(MachineFunction &MF) override;

public:
  static char ID; // Pass identification, replacement for typeid
  WebAssemblyRegNumbering() : MachineFunctionPass(ID) {}
};
} // end anonymous namespace

char WebAssemblyRegNumbering::ID = 0;
INITIALIZE_PASS(WebAssemblyRegNumbering, DEBUG_TYPE,
                "Assigns WebAssembly register numbers for virtual registers",
                false, false)

FunctionPass *llvm::createWebAssemblyRegNumbering() {
  return new WebAssemblyRegNumbering();
}

bool WebAssemblyRegNumbering::runOnMachineFunction(MachineFunction &MF) {
  LLVM_DEBUG(dbgs() << "********** Register Numbering **********\n"
                       "********** Function: "
                    << MF.getName() << '\n');

  WebAssemblyFunctionInfo &MFI = *MF.getInfo<WebAssemblyFunctionInfo>();
  MachineRegisterInfo &MRI = MF.getRegInfo();
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Register assignment and register-class constraints matter here.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 65-102

```cpp
  MFI.initWARegs(MRI);

  // WebAssembly argument registers are in the same index space as local
  // variables. Assign the numbers for them first.
  MachineBasicBlock &EntryMBB = MF.front();
  for (MachineInstr &MI : EntryMBB) {
    if (!WebAssembly::isArgument(MI.getOpcode()))
      break;

    int64_t Imm = MI.getOperand(1).getImm();
    LLVM_DEBUG(dbgs() << "Arg VReg " << printReg(MI.getOperand(0).getReg())
                      << " -> WAReg " << Imm << "\n");
    MFI.setWAReg(MI.getOperand(0).getReg(), Imm);
  }

  // Then assign regular WebAssembly registers for all remaining used
  // virtual registers. TODO: Consider sorting the registers by frequency of
  // use, to maximize usage of small immediate fields.
  unsigned NumVRegs = MF.getRegInfo().getNumVirtRegs();
  unsigned NumStackRegs = 0;
  // Start the numbering for locals after the arg regs
  unsigned CurReg = MFI.getParams().size();
  for (unsigned VRegIdx = 0; VRegIdx < NumVRegs; ++VRegIdx) {
    Register VReg = Register::index2VirtReg(VRegIdx);
    // Skip unused registers.
    if (MRI.use_empty(VReg))
      continue;
    // Handle stackified registers.
    if (MFI.isVRegStackified(VReg)) {
      LLVM_DEBUG(dbgs() << "VReg " << printReg(VReg) << " -> WAReg "
                        << (INT32_MIN | NumStackRegs) << "\n");
      MFI.setWAReg(VReg, INT32_MIN | NumStackRegs++);
      continue;
    }
    if (MFI.getWAReg(VReg) == WebAssembly::UnusedReg) {
      LLVM_DEBUG(dbgs() << "VReg " << printReg(VReg) << " -> WAReg " << CurReg
                        << "\n");
      MFI.setWAReg(VReg, CurReg++);
```
- **EN**: Implements helper routine(s) `initWARegs`, `front`, `isArgument` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `initWARegs`, `front`, `isArgument`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 103-107

```cpp
    }
  }

  return true;
}
```
- **EN**: Finishes the current helper or query by returning the target-specific result gathered in the preceding logic.
- **CN**: 这里通过返回语句结束当前辅助函数或查询，并给出前面逻辑计算出的目标相关结果。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Instruction selection or opcode handling / 指令选择或操作码处理
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `WebAssembly.h`
- `WebAssemblyMachineFunctionInfo.h`
- `WebAssemblyUtilities.h`
- `llvm/CodeGen/MachineFrameInfo.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/CodeGen/Passes.h`
- `llvm/Support/Debug.h`
- `llvm/Support/raw_ostream.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
