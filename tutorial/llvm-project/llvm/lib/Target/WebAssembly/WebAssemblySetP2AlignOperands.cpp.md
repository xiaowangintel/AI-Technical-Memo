# WebAssemblySetP2AlignOperands.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblySetP2AlignOperands.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file sets the p2align operands on load and store instructions.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblySetP2AlignOperands.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//=- WebAssemblySetP2AlignOperands.cpp - Set alignments on loads and stores -=//
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
///
/// \file
/// This file sets the p2align operands on load and store instructions.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 12-18

```cpp
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "WebAssembly.h"
#include "WebAssemblyInstrInfo.h"
#include "llvm/CodeGen/MachineBlockFrequencyInfo.h"
#include "llvm/CodeGen/MachineMemOperand.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 19-25

```cpp
#include "llvm/CodeGen/Passes.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;

#define DEBUG_TYPE "wasm-set-p2align-operands"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 26-63

```cpp
namespace {
class WebAssemblySetP2AlignOperands final : public MachineFunctionPass {
public:
  static char ID; // Pass identification, replacement for typeid
  WebAssemblySetP2AlignOperands() : MachineFunctionPass(ID) {}

  StringRef getPassName() const override {
    return "WebAssembly Set p2align Operands";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addPreserved<MachineBlockFrequencyInfoWrapperPass>();
    AU.addPreservedID(MachineDominatorsID);
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  bool runOnMachineFunction(MachineFunction &MF) override;
};
} // end anonymous namespace

char WebAssemblySetP2AlignOperands::ID = 0;
INITIALIZE_PASS(WebAssemblySetP2AlignOperands, DEBUG_TYPE,
                "Set the p2align operands for WebAssembly loads and stores",
                false, false)

FunctionPass *llvm::createWebAssemblySetP2AlignOperands() {
  return new WebAssemblySetP2AlignOperands();
}

static void rewriteP2Align(MachineInstr &MI, unsigned OperandNo) {
  assert(MI.getOperand(OperandNo).getImm() == 0 &&
         "ISel should set p2align operands to 0");
  assert(MI.hasOneMemOperand() &&
         "Load and store instructions have exactly one mem operand");
  assert((*MI.memoperands_begin())->getSize() ==
             (UINT64_C(1) << WebAssembly::GetDefaultP2Align(MI.getOpcode())) &&
         "Default p2align value should be natural");
```
- **EN**: Declares a backend-facing type `WebAssemblySetP2AlignOperands`, `MachineFunctionPass`, `getPassName` and outlines the API or state that nearby code will rely on. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明面向后端的类型 `WebAssemblySetP2AlignOperands`, `MachineFunctionPass`, `getPassName`，并勾勒出周边代码会依赖的接口或状态。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 64-96

```cpp
  assert(MI.getDesc().operands()[OperandNo].OperandType ==
             WebAssembly::OPERAND_P2ALIGN &&
         "Load and store instructions should have a p2align operand");
  uint64_t P2Align = Log2((*MI.memoperands_begin())->getAlign());

  // WebAssembly does not currently support supernatural alignment.
  P2Align = std::min(P2Align,
                     uint64_t(WebAssembly::GetDefaultP2Align(MI.getOpcode())));

  MI.getOperand(OperandNo).setImm(P2Align);
}

bool WebAssemblySetP2AlignOperands::runOnMachineFunction(MachineFunction &MF) {
  LLVM_DEBUG({
    dbgs() << "********** Set p2align Operands **********\n"
           << "********** Function: " << MF.getName() << '\n';
  });

  bool Changed = false;

  for (auto &MBB : MF) {
    for (auto &MI : MBB) {
      int16_t P2AlignOpNum = WebAssembly::getNamedOperandIdx(
          MI.getOpcode(), WebAssembly::OpName::p2align);
      if (P2AlignOpNum != -1) {
        rewriteP2Align(MI, P2AlignOpNum);
        Changed = true;
      }
    }
  }

  return Changed;
}
```
- **EN**: Implements helper routine(s) `getDesc`, `operands`, `Log2` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getDesc`, `operands`, `Log2`。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- Instruction selection or opcode handling / 指令选择或操作码处理
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `WebAssembly.h`
- `WebAssemblyInstrInfo.h`
- `llvm/CodeGen/MachineBlockFrequencyInfo.h`
- `llvm/CodeGen/MachineMemOperand.h`
- `llvm/CodeGen/Passes.h`
- `llvm/Support/Debug.h`
- `llvm/Support/raw_ostream.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
