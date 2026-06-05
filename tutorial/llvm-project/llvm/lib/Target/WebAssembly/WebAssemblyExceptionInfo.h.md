# WebAssemblyExceptionInfo.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyExceptionInfo.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: \brief This file implements WebAssemblyException information analysis.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyExceptionInfo.h`，主要负责 WebAssembly 后端的后端的公共或内部声明。 文件内容以接口、类型声明和协作关系说明为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyExceptionInfo.h - WebAssembly Exception Info -*- C++ -*-===//
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

#ifndef LLVM_LIB_TARGET_WEBASSEMBLY_WEBASSEMBLYEXCEPTIONINFO_H
#define LLVM_LIB_TARGET_WEBASSEMBLY_WEBASSEMBLYEXCEPTIONINFO_H

#include "WebAssembly.h"
#include "llvm/ADT/SmallPtrSet.h"
```
- **EN**: Pulls in direct dependencies required by this public or internal declarations for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端的公共或内部声明所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 19-41

```cpp
#include "llvm/CodeGen/MachineFunctionPass.h"

namespace llvm {

class MachineDominatorTree;
class MachineDominanceFrontier;

// WebAssembly instructions for exception handling are structured as follows:
//   try
//     instructions*
//   catch             ----|
//     instructions*       | -> A WebAssemblyException consists of this region
//   end               ----|
//
// A WebAssemblyException object contains BBs that belong to a 'catch' part of
// the try-catch-end structure to be created later. 'try' and 'end' markers
// are not present at this stage and will be generated in CFGStackify pass.
// Because CFGSort requires all the BBs within a catch part to be sorted
// together as it does for loops, this pass calculates the nesting structure of
// catch part of exceptions in a function.
//
// An exception catch part is defined as a BB with catch instruction and all
// other BBs dominated by this BB.
```
- **EN**: Pulls in direct dependencies required by this public or internal declarations for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这一段引入该后端的公共或内部声明所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 该区间与栈帧布局或栈访问相关。

### Lines 42-67

```cpp
class WebAssemblyException {
  MachineBasicBlock *EHPad = nullptr;

  WebAssemblyException *ParentException = nullptr;
  std::vector<std::unique_ptr<WebAssemblyException>> SubExceptions;
  std::vector<MachineBasicBlock *> Blocks;
  SmallPtrSet<MachineBasicBlock *, 8> BlockSet;

public:
  WebAssemblyException(MachineBasicBlock *EHPad) : EHPad(EHPad) {}
  WebAssemblyException(const WebAssemblyException &) = delete;
  const WebAssemblyException &operator=(const WebAssemblyException &) = delete;

  MachineBasicBlock *getEHPad() const { return EHPad; }
  MachineBasicBlock *getHeader() const { return EHPad; }
  WebAssemblyException *getParentException() const { return ParentException; }
  void setParentException(WebAssemblyException *WE) { ParentException = WE; }

  bool contains(const WebAssemblyException *WE) const {
    if (WE == this)
      return true;
    if (!WE)
      return false;
    return contains(WE->getParentException());
  }
  bool contains(const MachineBasicBlock *MBB) const {
```
- **EN**: Declares a backend-facing type `WebAssemblyException`, `EHPad`, `getEHPad` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `WebAssemblyException`, `EHPad`, `getEHPad`，并勾勒出周边代码会依赖的接口或状态。

### Lines 68-93

```cpp
    return BlockSet.count(MBB);
  }

  void addToBlocksSet(MachineBasicBlock *MBB) { BlockSet.insert(MBB); }
  void removeFromBlocksSet(MachineBasicBlock *MBB) { BlockSet.erase(MBB); }
  void addToBlocksVector(MachineBasicBlock *MBB) { Blocks.push_back(MBB); }
  void addBlock(MachineBasicBlock *MBB) {
    Blocks.push_back(MBB);
    BlockSet.insert(MBB);
  }
  ArrayRef<MachineBasicBlock *> getBlocks() const { return Blocks; }
  using block_iterator = ArrayRef<MachineBasicBlock *>::const_iterator;
  block_iterator block_begin() const { return getBlocks().begin(); }
  block_iterator block_end() const { return getBlocks().end(); }
  inline iterator_range<block_iterator> blocks() const {
    return make_range(block_begin(), block_end());
  }
  unsigned getNumBlocks() const { return Blocks.size(); }
  std::vector<MachineBasicBlock *> &getBlocksVector() { return Blocks; }
  SmallPtrSetImpl<MachineBasicBlock *> &getBlocksSet() { return BlockSet; }

  const std::vector<std::unique_ptr<WebAssemblyException>> &
  getSubExceptions() const {
    return SubExceptions;
  }
  std::vector<std::unique_ptr<WebAssemblyException>> &getSubExceptions() {
```
- **EN**: Implements helper routine(s) `count`, `addToBlocksSet`, `insert` for this portion of the WebAssembly backend public or internal declarations for the backend.
- **CN**: 这里实现了 WebAssembly 后端该部分后端的公共或内部声明所需的辅助例程 `count`, `addToBlocksSet`, `insert`。

### Lines 94-119

```cpp
    return SubExceptions;
  }
  void addSubException(std::unique_ptr<WebAssemblyException> E) {
    SubExceptions.push_back(std::move(E));
  }
  using iterator = decltype(SubExceptions)::const_iterator;
  iterator begin() const { return SubExceptions.begin(); }
  iterator end() const { return SubExceptions.end(); }

  void reserveBlocks(unsigned Size) { Blocks.reserve(Size); }
  void reverseBlock(unsigned From = 0) {
    std::reverse(Blocks.begin() + From, Blocks.end());
  }

  // Return the nesting level. An outermost one has depth 1.
  unsigned getExceptionDepth() const {
    unsigned D = 1;
    for (const WebAssemblyException *CurException = ParentException;
         CurException; CurException = CurException->ParentException)
      ++D;
    return D;
  }

  void print(raw_ostream &OS, unsigned Depth = 0) const;
  void dump() const;
};
```
- **EN**: Implements helper routine(s) `addSubException`, `push_back`, `move` for this portion of the WebAssembly backend public or internal declarations for the backend.
- **CN**: 这里实现了 WebAssembly 后端该部分后端的公共或内部声明所需的辅助例程 `addSubException`, `push_back`, `move`。

### Lines 120-145

```cpp

raw_ostream &operator<<(raw_ostream &OS, const WebAssemblyException &WE);

class WebAssemblyExceptionInfo final : public MachineFunctionPass {
  // Mapping of basic blocks to the innermost exception they occur in
  DenseMap<const MachineBasicBlock *, WebAssemblyException *> BBMap;
  std::vector<std::unique_ptr<WebAssemblyException>> TopLevelExceptions;

  void discoverAndMapException(WebAssemblyException *WE,
                               const MachineDominatorTree &MDT,
                               const MachineDominanceFrontier &MDF);
  WebAssemblyException *getOutermostException(MachineBasicBlock *MBB) const;

public:
  static char ID;
  WebAssemblyExceptionInfo() : MachineFunctionPass(ID) {}
  ~WebAssemblyExceptionInfo() override { releaseMemory(); }
  WebAssemblyExceptionInfo(const WebAssemblyExceptionInfo &) = delete;
  WebAssemblyExceptionInfo &
  operator=(const WebAssemblyExceptionInfo &) = delete;

  bool runOnMachineFunction(MachineFunction &) override;
  void releaseMemory() override;
  void recalculate(MachineFunction &MF, MachineDominatorTree &MDT,
                   const MachineDominanceFrontier &MDF);
  void getAnalysisUsage(AnalysisUsage &AU) const override;
```
- **EN**: Declares a backend-facing type `WebAssemblyExceptionInfo`, `discoverAndMapException`, `getOutermostException` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `WebAssemblyExceptionInfo`, `discoverAndMapException`, `getOutermostException`，并勾勒出周边代码会依赖的接口或状态。

### Lines 146-171

```cpp

  bool empty() const { return TopLevelExceptions.empty(); }

  // Return the innermost exception that MBB lives in. If the block is not in an
  // exception, null is returned.
  WebAssemblyException *getExceptionFor(const MachineBasicBlock *MBB) const {
    return BBMap.lookup(MBB);
  }

  void changeExceptionFor(const MachineBasicBlock *MBB,
                          WebAssemblyException *WE) {
    if (!WE) {
      BBMap.erase(MBB);
      return;
    }
    BBMap[MBB] = WE;
  }

  void addTopLevelException(std::unique_ptr<WebAssemblyException> WE) {
    assert(!WE->getParentException() && "Not a top level exception!");
    TopLevelExceptions.push_back(std::move(WE));
  }

  void print(raw_ostream &OS, const Module *M = nullptr) const override;
};
```
- **EN**: Implements helper routine(s) `empty`, `getExceptionFor`, `lookup` for this portion of the WebAssembly backend public or internal declarations for the backend.
- **CN**: 这里实现了 WebAssembly 后端该部分后端的公共或内部声明所需的辅助例程 `empty`, `getExceptionFor`, `lookup`。

### Lines 172-174

```cpp
} // end namespace llvm

#endif
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Public or internal declarations for the backend / 后端的公共或内部声明
- Stack frame management / 栈帧管理
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssembly.h`
- `llvm/ADT/SmallPtrSet.h`
- `llvm/CodeGen/MachineFunctionPass.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
