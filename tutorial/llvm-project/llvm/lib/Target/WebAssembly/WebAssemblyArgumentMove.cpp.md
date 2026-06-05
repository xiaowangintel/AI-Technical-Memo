# WebAssemblyArgumentMove.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyArgumentMove.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file moves ARGUMENT instructions after ScheduleDAG scheduling.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyArgumentMove.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyArgumentMove.cpp - Argument instruction moving ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-25

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file moves ARGUMENT instructions after ScheduleDAG scheduling.
///
/// Arguments are really live-in registers, however, since we use virtual
/// registers and LLVM doesn't support live-in virtual registers, we're
/// currently making do with ARGUMENT instructions which are placed at the top
/// of the entry block. The trick is to get them to *stay* at the top of the
/// entry block.
///
/// The ARGUMENTS physical register keeps these instructions pinned in place
/// during liveness-aware CodeGen passes, however one thing which does not
/// respect this is the ScheduleDAG scheduler. This pass is therefore run
/// immediately after that.
///
/// This is all hopefully a temporary solution until we find a better solution
/// for describing the live-in nature of arguments.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。 这一段包含调度或处理器模型元数据。

### Lines 26-32

```cpp
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "WebAssembly.h"
#include "WebAssemblyUtilities.h"
#include "llvm/CodeGen/MachineBlockFrequencyInfo.h"
#include "llvm/CodeGen/Passes.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 33-39

```cpp
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;

#define DEBUG_TYPE "wasm-argument-move"

namespace {
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 40-77

```cpp
class WebAssemblyArgumentMove final : public MachineFunctionPass {
public:
  static char ID; // Pass identification, replacement for typeid
  WebAssemblyArgumentMove() : MachineFunctionPass(ID) {}

  StringRef getPassName() const override { return "WebAssembly Argument Move"; }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addPreserved<MachineBlockFrequencyInfoWrapperPass>();
    AU.addPreservedID(MachineDominatorsID);
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  bool runOnMachineFunction(MachineFunction &MF) override;
};
} // end anonymous namespace

char WebAssemblyArgumentMove::ID = 0;
INITIALIZE_PASS(WebAssemblyArgumentMove, DEBUG_TYPE,
                "Move ARGUMENT instructions for WebAssembly", false, false)

FunctionPass *llvm::createWebAssemblyArgumentMove() {
  return new WebAssemblyArgumentMove();
}

bool WebAssemblyArgumentMove::runOnMachineFunction(MachineFunction &MF) {
  LLVM_DEBUG({
    dbgs() << "********** Argument Move **********\n"
           << "********** Function: " << MF.getName() << '\n';
  });

  bool Changed = false;
  MachineBasicBlock &EntryMBB = MF.front();
  MachineBasicBlock::iterator InsertPt = EntryMBB.end();

  // Look for the first NonArg instruction.
  for (MachineInstr &MI : EntryMBB) {
```
- **EN**: Declares a backend-facing type `WebAssemblyArgumentMove`, `MachineFunctionPass`, `getPassName` and outlines the API or state that nearby code will rely on. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明面向后端的类型 `WebAssemblyArgumentMove`, `MachineFunctionPass`, `getPassName`，并勾勒出周边代码会依赖的接口或状态。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 78-94

```cpp
    if (!WebAssembly::isArgument(MI.getOpcode())) {
      InsertPt = MI;
      break;
    }
  }

  // Now move any argument instructions later in the block
  // to before our first NonArg instruction.
  for (MachineInstr &MI : llvm::make_range(InsertPt, EntryMBB.end())) {
    if (WebAssembly::isArgument(MI.getOpcode())) {
      EntryMBB.insert(InsertPt, MI.removeFromParent());
      Changed = true;
    }
  }

  return Changed;
}
```
- **EN**: Implements helper routine(s) `isArgument`, `getOpcode`, `make_range` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `isArgument`, `getOpcode`, `make_range`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- Instruction selection or opcode handling / 指令选择或操作码处理
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `WebAssembly.h`
- `WebAssemblyUtilities.h`
- `llvm/CodeGen/MachineBlockFrequencyInfo.h`
- `llvm/CodeGen/Passes.h`
- `llvm/Support/Debug.h`
- `llvm/Support/raw_ostream.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
