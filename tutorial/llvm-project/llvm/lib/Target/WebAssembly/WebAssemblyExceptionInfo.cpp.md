# WebAssemblyExceptionInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyExceptionInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: \brief This file implements WebAssemblyException information analysis.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyExceptionInfo.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===--- WebAssemblyExceptionInfo.cpp - Exception Infomation --------------===//
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
/// \brief This file implements WebAssemblyException information analysis.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 12-18

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssemblyExceptionInfo.h"
#include "WebAssemblyUtilities.h"
#include "llvm/ADT/PostOrderIterator.h"
#include "llvm/CodeGen/MachineDominanceFrontier.h"
#include "llvm/CodeGen/MachineDominators.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 19-25

```cpp
#include "llvm/IR/Function.h"
#include "llvm/InitializePasses.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/Target/TargetMachine.h"

using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 26-63

```cpp
#define DEBUG_TYPE "wasm-exception-info"

char WebAssemblyExceptionInfo::ID = 0;

INITIALIZE_PASS_BEGIN(WebAssemblyExceptionInfo, DEBUG_TYPE,
                      "WebAssembly Exception Information", true, true)
INITIALIZE_PASS_DEPENDENCY(MachineDominatorTreeWrapperPass)
INITIALIZE_PASS_DEPENDENCY(MachineDominanceFrontierWrapperPass)
INITIALIZE_PASS_END(WebAssemblyExceptionInfo, DEBUG_TYPE,
                    "WebAssembly Exception Information", true, true)

bool WebAssemblyExceptionInfo::runOnMachineFunction(MachineFunction &MF) {
  LLVM_DEBUG(dbgs() << "********** Exception Info Calculation **********\n"
                       "********** Function: "
                    << MF.getName() << '\n');
  releaseMemory();
  if (MF.getTarget().getMCAsmInfo().getExceptionHandlingType() !=
          ExceptionHandling::Wasm ||
      !MF.getFunction().hasPersonalityFn())
    return false;
  auto &MDT = getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  auto &MDF = getAnalysis<MachineDominanceFrontierWrapperPass>().getMDF();
  recalculate(MF, MDT, MDF);
  LLVM_DEBUG(dump());
  return false;
}

void WebAssemblyExceptionInfo::recalculate(
    MachineFunction &MF, MachineDominatorTree &MDT,
    const MachineDominanceFrontier &MDF) {
  // Postorder traversal of the dominator tree.
  SmallVector<std::unique_ptr<WebAssemblyException>, 8> Exceptions;
  for (auto *DomNode : post_order(&MDT)) {
    MachineBasicBlock *EHPad = DomNode->getBlock();
    if (!EHPad->isEHPad())
      continue;
    auto WE = std::make_unique<WebAssemblyException>(EHPad);
    discoverAndMapException(WE.get(), MDT, MDF);
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. The logic interacts with LLVM's MC layer.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 64-101

```cpp
    Exceptions.push_back(std::move(WE));
  }

  // Add BBs to exceptions' block set. This is a preparation to take out
  // remaining incorect BBs from exceptions, because we need to iterate over BBs
  // for each exception.
  for (auto *DomNode : post_order(&MDT)) {
    MachineBasicBlock *MBB = DomNode->getBlock();
    WebAssemblyException *WE = getExceptionFor(MBB);
    for (; WE; WE = WE->getParentException())
      WE->addToBlocksSet(MBB);
  }

  // Add BBs to exceptions' block vector
  for (auto *DomNode : post_order(&MDT)) {
    MachineBasicBlock *MBB = DomNode->getBlock();
    WebAssemblyException *WE = getExceptionFor(MBB);
    for (; WE; WE = WE->getParentException())
      WE->addToBlocksVector(MBB);
  }

  SmallVector<WebAssemblyException*, 8> ExceptionPointers;
  ExceptionPointers.reserve(Exceptions.size());

  // Add subexceptions to exceptions
  for (auto &WE : Exceptions) {
    ExceptionPointers.push_back(WE.get());
    if (WE->getParentException())
      WE->getParentException()->getSubExceptions().push_back(std::move(WE));
    else
      addTopLevelException(std::move(WE));
  }

  // For convenience, Blocks and SubExceptions are inserted in postorder.
  // Reverse the lists.
  for (auto *WE : ExceptionPointers) {
    WE->reverseBlock();
    std::reverse(WE->getSubExceptions().begin(), WE->getSubExceptions().end());
```
- **EN**: Implements helper routine(s) `push_back`, `move`, `post_order` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `push_back`, `move`, `post_order`。

### Lines 102-139

```cpp
  }
}

void WebAssemblyExceptionInfo::releaseMemory() {
  BBMap.clear();
  TopLevelExceptions.clear();
}

void WebAssemblyExceptionInfo::getAnalysisUsage(AnalysisUsage &AU) const {
  AU.setPreservesAll();
  AU.addRequired<MachineDominatorTreeWrapperPass>();
  AU.addRequired<MachineDominanceFrontierWrapperPass>();
  MachineFunctionPass::getAnalysisUsage(AU);
}

void WebAssemblyExceptionInfo::discoverAndMapException(
    WebAssemblyException *WE, const MachineDominatorTree &MDT,
    const MachineDominanceFrontier &MDF) {
  unsigned NumBlocks = 0;
  unsigned NumSubExceptions = 0;

  // Map blocks that belong to a catchpad / cleanuppad
  MachineBasicBlock *EHPad = WE->getEHPad();
  SmallVector<MachineBasicBlock *, 8> WL;
  WL.push_back(EHPad);
  while (!WL.empty()) {
    MachineBasicBlock *MBB = WL.pop_back_val();

    // Find its outermost discovered exception. If this is a discovered block,
    // check if it is already discovered to be a subexception of this exception.
    WebAssemblyException *SubE = getOutermostException(MBB);
    if (SubE) {
      if (SubE != WE) {
        // Discover a subexception of this exception.
        SubE->setParentException(WE);
        ++NumSubExceptions;
        NumBlocks += SubE->getBlocksVector().capacity();
        // All blocks that belong to this subexception have been already
```
- **EN**: Implements helper routine(s) `releaseMemory`, `clear`, `getAnalysisUsage` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `releaseMemory`, `clear`, `getAnalysisUsage`。

### Lines 140-177

```cpp
        // discovered. Skip all of them. Add the subexception's landing pad's
        // dominance frontier to the worklist.
        for (auto &Frontier : MDF.find(SubE->getEHPad())->second)
          if (MDT.dominates(EHPad, Frontier))
            WL.push_back(Frontier);
      }
      continue;
    }

    // This is an undiscovered block. Map it to the current exception.
    changeExceptionFor(MBB, WE);
    ++NumBlocks;

    // Add successors dominated by the current BB to the worklist.
    for (auto *Succ : MBB->successors())
      if (MDT.dominates(EHPad, Succ))
        WL.push_back(Succ);
  }

  WE->getSubExceptions().reserve(NumSubExceptions);
  WE->reserveBlocks(NumBlocks);
}

WebAssemblyException *
WebAssemblyExceptionInfo::getOutermostException(MachineBasicBlock *MBB) const {
  WebAssemblyException *WE = getExceptionFor(MBB);
  if (WE) {
    while (WebAssemblyException *Parent = WE->getParentException())
      WE = Parent;
  }
  return WE;
}

void WebAssemblyException::print(raw_ostream &OS, unsigned Depth) const {
  OS.indent(Depth * 2) << "Exception at depth " << getExceptionDepth()
                       << " containing: ";

  for (unsigned I = 0; I < getBlocks().size(); ++I) {
```
- **EN**: Implements helper routine(s) `find`, `getEHPad`, `dominates` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `find`, `getEHPad`, `dominates`。

### Lines 178-196

```cpp
    MachineBasicBlock *MBB = getBlocks()[I];
    if (I)
      OS << ", ";
    OS << "%bb." << MBB->getNumber();
    if (const auto *BB = MBB->getBasicBlock())
      if (BB->hasName())
        OS << "." << BB->getName();

    if (getEHPad() == MBB)
      OS << " (landing-pad)";
  }
  OS << "\n";

  for (auto &SubE : SubExceptions)
    SubE->print(OS, Depth + 2);
}

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
LLVM_DUMP_METHOD void WebAssemblyException::dump() const { print(dbgs()); }
```
- **EN**: Implements helper routine(s) `getBlocks`, `getNumber`, `getBasicBlock` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getBlocks`, `getNumber`, `getBasicBlock`。

### Lines 197-207

```cpp
#endif

raw_ostream &operator<<(raw_ostream &OS, const WebAssemblyException &WE) {
  WE.print(OS);
  return OS;
}

void WebAssemblyExceptionInfo::print(raw_ostream &OS, const Module *) const {
  for (auto &WE : TopLevelExceptions)
    WE->print(OS);
}
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `print`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `print`。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- Target machine configuration / 目标机器配置
- Object format integration / 目标文件格式集成
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblyExceptionInfo.h`
- `WebAssemblyUtilities.h`
- `llvm/ADT/PostOrderIterator.h`
- `llvm/CodeGen/MachineDominanceFrontier.h`
- `llvm/CodeGen/MachineDominators.h`
- `llvm/IR/Function.h`
- `llvm/InitializePasses.h`
- `llvm/MC/MCAsmInfo.h`
- `llvm/Target/TargetMachine.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM MC layer / LLVM MC 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
