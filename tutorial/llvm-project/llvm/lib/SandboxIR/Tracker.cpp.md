# Tracker.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/SandboxIR/Tracker.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**: This file belongs to `lib/SandboxIR` and implements logic, data handling, or helper flows related to `Tracker`. / 该文件位于 `lib/SandboxIR`，主要实现与 `Tracker` 相关的接口、数据结构或辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
//===- Tracker.cpp --------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/SandboxIR/Tracker.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/Instruction.h"
#include "llvm/IR/StructuralHash.h"
#include "llvm/SandboxIR/Instruction.h"

using namespace llvm::sandboxir;

#ifndef NDEBUG

std::string IRSnapshotChecker::dumpIR(const llvm::Function &F) const {
```

- **L1**: Banner comment marking a file section boundary. / 横幅注释，用于标记文件分节。
- **L2**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L3**: Comment documents the nearby logic or transformation intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑或变换意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment documents the nearby logic or transformation intent: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑或变换意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment documents the nearby logic or transformation intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑或变换意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L7**: Banner comment marking a file section boundary. / 横幅注释，用于标记文件分节。
- **L8**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L9**: Includes `llvm/SandboxIR/Tracker.h` to access local declarations used by this file. / 引入 `llvm/SandboxIR/Tracker.h` 以使用本文件使用的本地声明。
- **L10**: Includes `llvm/ADT/STLExtras.h` to access LLVM ADT data structures/utilities. / 引入 `llvm/ADT/STLExtras.h` 以使用LLVM ADT 数据结构/工具。
- **L11**: Includes `llvm/IR/BasicBlock.h` to access LLVM IR core types and builders. / 引入 `llvm/IR/BasicBlock.h` 以使用LLVM IR 核心类型与构造工具。
- **L12**: Includes `llvm/IR/Instruction.h` to access LLVM IR core types and builders. / 引入 `llvm/IR/Instruction.h` 以使用LLVM IR 核心类型与构造工具。
- **L13**: Includes `llvm/IR/StructuralHash.h` to access LLVM IR core types and builders. / 引入 `llvm/IR/StructuralHash.h` 以使用LLVM IR 核心类型与构造工具。
- **L14**: Includes `llvm/SandboxIR/Instruction.h` to access local declarations used by this file. / 引入 `llvm/SandboxIR/Instruction.h` 以使用本文件使用的本地声明。
- **L15**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L16**: Brings namespace `llvm::sandboxir` into the local scope. / 将命名空间 `llvm::sandboxir` 引入当前作用域。
- **L17**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L18**: Preprocessor directive controls conditional compilation or build behavior: `#ifndef NDEBUG`. / 预处理指令控制条件编译或构建行为：`#ifndef NDEBUG`。
- **L19**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L20**: Starts the definition of function or method `IRSnapshotChecker::dumpIR`. / 开始定义函数或方法 `IRSnapshotChecker::dumpIR`。

### Lines 21-40

```cpp
  std::string Result;
  raw_string_ostream SS(Result);
  F.print(SS, /*AssemblyAnnotationWriter=*/nullptr);
  return Result;
}

IRSnapshotChecker::ContextSnapshot IRSnapshotChecker::takeSnapshot() const {
  ContextSnapshot Result;
  for (const auto &Entry : Ctx.LLVMModuleToModuleMap)
    for (const auto &F : *Entry.first) {
      FunctionSnapshot Snapshot;
      Snapshot.Hash = StructuralHash(F, /*DetailedHash=*/true);
      Snapshot.TextualIR = dumpIR(F);
      Result[&F] = Snapshot;
    }
  return Result;
}

bool IRSnapshotChecker::diff(const ContextSnapshot &Orig,
                             const ContextSnapshot &Curr) const {
```

- **L21**: Executes a standalone statement or declaration: `std::string Result;`. / 执行一条独立语句或声明：`std::string Result;`。
- **L22**: Executes call or statement centered on `raw_string_ostream SS`. / 执行以 `raw_string_ostream SS` 为核心的调用或语句。
- **L23**: Initializes or updates `F.print(SS, /*AssemblyAnnotationWriter` from the right-hand expression. / 使用右侧表达式初始化或更新 `F.print(SS, /*AssemblyAnnotationWriter`。
- **L24**: Returns control, optionally with a value: `return Result;`. / 返回控制流，并可附带返回值：`return Result;`。
- **L25**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L26**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L27**: Starts the definition of function or method `IRSnapshotChecker::takeSnapshot`. / 开始定义函数或方法 `IRSnapshotChecker::takeSnapshot`。
- **L28**: Executes a standalone statement or declaration: `ContextSnapshot Result;`. / 执行一条独立语句或声明：`ContextSnapshot Result;`。
- **L29**: Starts a loop over a range or sequence: `for (const auto &Entry : Ctx.LLVMModuleToModuleMap)`. / 开始遍历某个范围或序列的循环：`for (const auto &Entry : Ctx.LLVMModuleToModuleMap)`。
- **L30**: Starts a loop over a range or sequence: `for (const auto &F : *Entry.first) {`. / 开始遍历某个范围或序列的循环：`for (const auto &F : *Entry.first) {`。
- **L31**: Executes a standalone statement or declaration: `FunctionSnapshot Snapshot;`. / 执行一条独立语句或声明：`FunctionSnapshot Snapshot;`。
- **L32**: Initializes or updates `Snapshot.Hash` from the right-hand expression. / 使用右侧表达式初始化或更新 `Snapshot.Hash`。
- **L33**: Initializes or updates `Snapshot.TextualIR` from the right-hand expression. / 使用右侧表达式初始化或更新 `Snapshot.TextualIR`。
- **L34**: Initializes or updates `Result[&F]` from the right-hand expression. / 使用右侧表达式初始化或更新 `Result[&F]`。
- **L35**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L36**: Returns control, optionally with a value: `return Result;`. / 返回控制流，并可附带返回值：`return Result;`。
- **L37**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L38**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L39**: Continues a multi-line argument list or initializer: `bool IRSnapshotChecker::diff(const ContextSnapshot &Orig,`. / 继续一个多行参数列表或初始化器：`bool IRSnapshotChecker::diff(const ContextSnapshot &Orig,`。
- **L40**: Continues the surrounding expression or declaration: `const ContextSnapshot &Curr) const {`. / 继续构造周围的表达式或声明：`const ContextSnapshot &Curr) const {`。

### Lines 41-60

```cpp
  bool DifferenceFound = false;
  for (const auto &[F, OrigFS] : Orig) {
    auto CurrFSIt = Curr.find(F);
    if (CurrFSIt == Curr.end()) {
      DifferenceFound = true;
      dbgs() << "Function " << F->getName() << " not found in current IR.\n";
      dbgs() << OrigFS.TextualIR << "\n";
      continue;
    }
    const FunctionSnapshot &CurrFS = CurrFSIt->second;
    if (OrigFS.Hash != CurrFS.Hash) {
      DifferenceFound = true;
      dbgs() << "Found IR difference in Function " << F->getName() << "\n";
      dbgs() << "Original:\n" << OrigFS.TextualIR << "\n";
      dbgs() << "Current:\n" << CurrFS.TextualIR << "\n";
    }
  }
  // Check that Curr doesn't contain any new functions.
  for (const auto &[F, CurrFS] : Curr) {
    if (!Orig.contains(F)) {
```

- **L41**: Initializes or updates `bool DifferenceFound` from the right-hand expression. / 使用右侧表达式初始化或更新 `bool DifferenceFound`。
- **L42**: Starts a loop over a range or sequence: `for (const auto &[F, OrigFS] : Orig) {`. / 开始遍历某个范围或序列的循环：`for (const auto &[F, OrigFS] : Orig) {`。
- **L43**: Initializes or updates `auto CurrFSIt` from the right-hand expression. / 使用右侧表达式初始化或更新 `auto CurrFSIt`。
- **L44**: Introduces a conditional branch: `if (CurrFSIt == Curr.end()) {`. / 引入条件分支：`if (CurrFSIt == Curr.end()) {`。
- **L45**: Initializes or updates `DifferenceFound` from the right-hand expression. / 使用右侧表达式初始化或更新 `DifferenceFound`。
- **L46**: Executes call or statement centered on `dbgs`. / 执行以 `dbgs` 为核心的调用或语句。
- **L47**: Executes call or statement centered on `dbgs`. / 执行以 `dbgs` 为核心的调用或语句。
- **L48**: Executes a standalone statement or declaration: `continue;`. / 执行一条独立语句或声明：`continue;`。
- **L49**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L50**: Initializes or updates `const FunctionSnapshot &CurrFS` from the right-hand expression. / 使用右侧表达式初始化或更新 `const FunctionSnapshot &CurrFS`。
- **L51**: Introduces a conditional branch: `if (OrigFS.Hash != CurrFS.Hash) {`. / 引入条件分支：`if (OrigFS.Hash != CurrFS.Hash) {`。
- **L52**: Initializes or updates `DifferenceFound` from the right-hand expression. / 使用右侧表达式初始化或更新 `DifferenceFound`。
- **L53**: Executes call or statement centered on `dbgs`. / 执行以 `dbgs` 为核心的调用或语句。
- **L54**: Executes call or statement centered on `dbgs`. / 执行以 `dbgs` 为核心的调用或语句。
- **L55**: Executes call or statement centered on `dbgs`. / 执行以 `dbgs` 为核心的调用或语句。
- **L56**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L57**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L58**: Comment documents the nearby logic or transformation intent: `Check that Curr doesn't contain any new functions.`. / 注释说明了附近代码的逻辑或变换意图：`Check that Curr doesn't contain any new functions.`。
- **L59**: Starts a loop over a range or sequence: `for (const auto &[F, CurrFS] : Curr) {`. / 开始遍历某个范围或序列的循环：`for (const auto &[F, CurrFS] : Curr) {`。
- **L60**: Introduces a conditional branch: `if (!Orig.contains(F)) {`. / 引入条件分支：`if (!Orig.contains(F)) {`。

### Lines 61-80

```cpp
      DifferenceFound = true;
      dbgs() << "Function " << F->getName()
             << " found in current IR but not in original snapshot.\n";
      dbgs() << CurrFS.TextualIR << "\n";
    }
  }
  return DifferenceFound;
}

void IRSnapshotChecker::save() { OrigContextSnapshot = takeSnapshot(); }

void IRSnapshotChecker::expectNoDiff() {
  ContextSnapshot CurrContextSnapshot = takeSnapshot();
  if (diff(OrigContextSnapshot, CurrContextSnapshot)) {
    llvm_unreachable(
        "Original and current IR differ! Probably a checkpointing bug.");
  }
}

void UseSet::dump() const {
```

- **L61**: Initializes or updates `DifferenceFound` from the right-hand expression. / 使用右侧表达式初始化或更新 `DifferenceFound`。
- **L62**: Continues the surrounding expression or declaration: `dbgs() << "Function " << F->getName()`. / 继续构造周围的表达式或声明：`dbgs() << "Function " << F->getName()`。
- **L63**: Executes a standalone statement or declaration: `<< " found in current IR but not in original snapshot.\n";`. / 执行一条独立语句或声明：`<< " found in current IR but not in original snapshot.\n";`。
- **L64**: Executes call or statement centered on `dbgs`. / 执行以 `dbgs` 为核心的调用或语句。
- **L65**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L66**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L67**: Returns control, optionally with a value: `return DifferenceFound;`. / 返回控制流，并可附带返回值：`return DifferenceFound;`。
- **L68**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L69**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L70**: Continues the surrounding expression or declaration: `void IRSnapshotChecker::save() { OrigContextSnapshot = takeSnapshot(); }`. / 继续构造周围的表达式或声明：`void IRSnapshotChecker::save() { OrigContextSnapshot = takeSnapshot(); }`。
- **L71**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L72**: Starts the definition of function or method `IRSnapshotChecker::expectNoDiff`. / 开始定义函数或方法 `IRSnapshotChecker::expectNoDiff`。
- **L73**: Initializes or updates `ContextSnapshot CurrContextSnapshot` from the right-hand expression. / 使用右侧表达式初始化或更新 `ContextSnapshot CurrContextSnapshot`。
- **L74**: Introduces a conditional branch: `if (diff(OrigContextSnapshot, CurrContextSnapshot)) {`. / 引入条件分支：`if (diff(OrigContextSnapshot, CurrContextSnapshot)) {`。
- **L75**: Continues a multi-line argument list or initializer: `llvm_unreachable(`. / 继续一个多行参数列表或初始化器：`llvm_unreachable(`。
- **L76**: Executes a standalone statement or declaration: `"Original and current IR differ! Probably a checkpointing bug.");`. / 执行一条独立语句或声明：`"Original and current IR differ! Probably a checkpointing bug.");`。
- **L77**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L78**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L79**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L80**: Starts the definition of function or method `UseSet::dump`. / 开始定义函数或方法 `UseSet::dump`。

### Lines 81-100

```cpp
  dump(dbgs());
  dbgs() << "\n";
}

void UseSwap::dump() const {
  dump(dbgs());
  dbgs() << "\n";
}
#endif // NDEBUG

PHIRemoveIncoming::PHIRemoveIncoming(PHINode *PHI, unsigned RemovedIdx)
    : PHI(PHI), RemovedIdx(RemovedIdx) {
  RemovedV = PHI->getIncomingValue(RemovedIdx);
  RemovedBB = PHI->getIncomingBlock(RemovedIdx);
}

void PHIRemoveIncoming::revert(Tracker &Tracker) {
  // Special case: if the removed incoming value is the last.
  unsigned NumIncoming = PHI->getNumIncomingValues();
  if (NumIncoming == RemovedIdx) {
```

- **L81**: Executes call or statement centered on `dump`. / 执行以 `dump` 为核心的调用或语句。
- **L82**: Executes call or statement centered on `dbgs`. / 执行以 `dbgs` 为核心的调用或语句。
- **L83**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L84**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L85**: Starts the definition of function or method `UseSwap::dump`. / 开始定义函数或方法 `UseSwap::dump`。
- **L86**: Executes call or statement centered on `dump`. / 执行以 `dump` 为核心的调用或语句。
- **L87**: Executes call or statement centered on `dbgs`. / 执行以 `dbgs` 为核心的调用或语句。
- **L88**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L89**: Preprocessor directive controls conditional compilation or build behavior: `#endif // NDEBUG`. / 预处理指令控制条件编译或构建行为：`#endif // NDEBUG`。
- **L90**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L91**: Continues the surrounding expression or declaration: `PHIRemoveIncoming::PHIRemoveIncoming(PHINode *PHI, unsigned RemovedIdx)`. / 继续构造周围的表达式或声明：`PHIRemoveIncoming::PHIRemoveIncoming(PHINode *PHI, unsigned RemovedIdx)`。
- **L92**: Starts the definition of function or method `PHI`. / 开始定义函数或方法 `PHI`。
- **L93**: Initializes or updates `RemovedV` from the right-hand expression. / 使用右侧表达式初始化或更新 `RemovedV`。
- **L94**: Initializes or updates `RemovedBB` from the right-hand expression. / 使用右侧表达式初始化或更新 `RemovedBB`。
- **L95**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L96**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L97**: Starts the definition of function or method `PHIRemoveIncoming::revert`. / 开始定义函数或方法 `PHIRemoveIncoming::revert`。
- **L98**: Comment documents the nearby logic or transformation intent: `Special case: if the removed incoming value is the last.`. / 注释说明了附近代码的逻辑或变换意图：`Special case: if the removed incoming value is the last.`。
- **L99**: Initializes or updates `unsigned NumIncoming` from the right-hand expression. / 使用右侧表达式初始化或更新 `unsigned NumIncoming`。
- **L100**: Introduces a conditional branch: `if (NumIncoming == RemovedIdx) {`. / 引入条件分支：`if (NumIncoming == RemovedIdx) {`。

### Lines 101-120

```cpp
    PHI->addIncoming(RemovedV, RemovedBB);
    return;
  }
  // Move the incoming value currently at `RemovedIdx` to the end, restore the
  // old incoming value back to `RemovedIdx`.
  PHI->addIncoming(PHI->getIncomingValue(RemovedIdx),
                   PHI->getIncomingBlock(RemovedIdx));
  PHI->setIncomingValue(RemovedIdx, RemovedV);
  PHI->setIncomingBlock(RemovedIdx, RemovedBB);
}

#ifndef NDEBUG
void PHIRemoveIncoming::dump() const {
  dump(dbgs());
  dbgs() << "\n";
}
#endif // NDEBUG

PHIAddIncoming::PHIAddIncoming(PHINode *PHI)
    : PHI(PHI), Idx(PHI->getNumIncomingValues()) {}
```

- **L101**: Executes call or statement centered on `PHI->addIncoming`. / 执行以 `PHI->addIncoming` 为核心的调用或语句。
- **L102**: Executes a standalone statement or declaration: `return;`. / 执行一条独立语句或声明：`return;`。
- **L103**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L104**: Comment documents the nearby logic or transformation intent: `Move the incoming value currently at \`RemovedIdx\` to the end, restore the`. / 注释说明了附近代码的逻辑或变换意图：`Move the incoming value currently at \`RemovedIdx\` to the end, restore the`。
- **L105**: Comment documents the nearby logic or transformation intent: `old incoming value back to \`RemovedIdx\`.`. / 注释说明了附近代码的逻辑或变换意图：`old incoming value back to \`RemovedIdx\`.`。
- **L106**: Continues a multi-line argument list or initializer: `PHI->addIncoming(PHI->getIncomingValue(RemovedIdx),`. / 继续一个多行参数列表或初始化器：`PHI->addIncoming(PHI->getIncomingValue(RemovedIdx),`。
- **L107**: Executes call or statement centered on `PHI->getIncomingBlock`. / 执行以 `PHI->getIncomingBlock` 为核心的调用或语句。
- **L108**: Executes call or statement centered on `PHI->setIncomingValue`. / 执行以 `PHI->setIncomingValue` 为核心的调用或语句。
- **L109**: Executes call or statement centered on `PHI->setIncomingBlock`. / 执行以 `PHI->setIncomingBlock` 为核心的调用或语句。
- **L110**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L111**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L112**: Preprocessor directive controls conditional compilation or build behavior: `#ifndef NDEBUG`. / 预处理指令控制条件编译或构建行为：`#ifndef NDEBUG`。
- **L113**: Starts the definition of function or method `PHIRemoveIncoming::dump`. / 开始定义函数或方法 `PHIRemoveIncoming::dump`。
- **L114**: Executes call or statement centered on `dump`. / 执行以 `dump` 为核心的调用或语句。
- **L115**: Executes call or statement centered on `dbgs`. / 执行以 `dbgs` 为核心的调用或语句。
- **L116**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L117**: Preprocessor directive controls conditional compilation or build behavior: `#endif // NDEBUG`. / 预处理指令控制条件编译或构建行为：`#endif // NDEBUG`。
- **L118**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L119**: Continues the surrounding expression or declaration: `PHIAddIncoming::PHIAddIncoming(PHINode *PHI)`. / 继续构造周围的表达式或声明：`PHIAddIncoming::PHIAddIncoming(PHINode *PHI)`。
- **L120**: Continues a multi-line argument list or initializer: `: PHI(PHI), Idx(PHI->getNumIncomingValues()) {}`. / 继续一个多行参数列表或初始化器：`: PHI(PHI), Idx(PHI->getNumIncomingValues()) {}`。

### Lines 121-140

```cpp

void PHIAddIncoming::revert(Tracker &Tracker) { PHI->removeIncomingValue(Idx); }

#ifndef NDEBUG
void PHIAddIncoming::dump() const {
  dump(dbgs());
  dbgs() << "\n";
}
#endif // NDEBUG

Tracker::~Tracker() {
  assert(Changes.empty() && "You must accept or revert changes!");
}

EraseFromParent::EraseFromParent(std::unique_ptr<sandboxir::Value> &&ErasedIPtr)
    : ErasedIPtr(std::move(ErasedIPtr)) {
  auto *I = cast<Instruction>(this->ErasedIPtr.get());
  auto LLVMInstrs = I->getLLVMInstrs();
  // Iterate in reverse program order.
  for (auto *LLVMI : reverse(LLVMInstrs)) {
```

- **L121**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L122**: Continues the surrounding expression or declaration: `void PHIAddIncoming::revert(Tracker &Tracker) { PHI->removeIncomingValue(Idx); }`. / 继续构造周围的表达式或声明：`void PHIAddIncoming::revert(Tracker &Tracker) { PHI->removeIncomingValue(Idx); }`。
- **L123**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L124**: Preprocessor directive controls conditional compilation or build behavior: `#ifndef NDEBUG`. / 预处理指令控制条件编译或构建行为：`#ifndef NDEBUG`。
- **L125**: Starts the definition of function or method `PHIAddIncoming::dump`. / 开始定义函数或方法 `PHIAddIncoming::dump`。
- **L126**: Executes call or statement centered on `dump`. / 执行以 `dump` 为核心的调用或语句。
- **L127**: Executes call or statement centered on `dbgs`. / 执行以 `dbgs` 为核心的调用或语句。
- **L128**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L129**: Preprocessor directive controls conditional compilation or build behavior: `#endif // NDEBUG`. / 预处理指令控制条件编译或构建行为：`#endif // NDEBUG`。
- **L130**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L131**: Starts the definition of function or method `Tracker::~Tracker`. / 开始定义函数或方法 `Tracker::~Tracker`。
- **L132**: Checks an internal invariant with an assertion: `assert(Changes.empty() && "You must accept or revert changes!");`. / 通过断言检查内部不变式：`assert(Changes.empty() && "You must accept or revert changes!");`。
- **L133**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L134**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L135**: Continues the surrounding expression or declaration: `EraseFromParent::EraseFromParent(std::unique_ptr<sandboxir::Value> &&ErasedIPtr)`. / 继续构造周围的表达式或声明：`EraseFromParent::EraseFromParent(std::unique_ptr<sandboxir::Value> &&ErasedIPtr)`。
- **L136**: Starts the definition of function or method `ErasedIPtr`. / 开始定义函数或方法 `ErasedIPtr`。
- **L137**: Initializes or updates `auto *I` from the right-hand expression. / 使用右侧表达式初始化或更新 `auto *I`。
- **L138**: Initializes or updates `auto LLVMInstrs` from the right-hand expression. / 使用右侧表达式初始化或更新 `auto LLVMInstrs`。
- **L139**: Comment documents the nearby logic or transformation intent: `Iterate in reverse program order.`. / 注释说明了附近代码的逻辑或变换意图：`Iterate in reverse program order.`。
- **L140**: Starts a loop over a range or sequence: `for (auto *LLVMI : reverse(LLVMInstrs)) {`. / 开始遍历某个范围或序列的循环：`for (auto *LLVMI : reverse(LLVMInstrs)) {`。

### Lines 141-160

```cpp
    SmallVector<llvm::Value *> Operands;
    Operands.reserve(LLVMI->getNumOperands());
    for (auto [OpNum, Use] : enumerate(LLVMI->operands()))
      Operands.push_back(Use.get());
    InstrData.push_back({Operands, LLVMI});
  }
  assert(is_sorted(InstrData,
                   [](const auto &D0, const auto &D1) {
                     return D0.LLVMI->comesBefore(D1.LLVMI);
                   }) &&
         "Expected reverse program order!");
  auto *BotLLVMI = cast<llvm::Instruction>(I->Val);
  if (BotLLVMI->getNextNode() != nullptr)
    NextLLVMIOrBB = BotLLVMI->getNextNode();
  else
    NextLLVMIOrBB = BotLLVMI->getParent();
}

void EraseFromParent::accept() {
  for (const auto &IData : InstrData)
```

- **L141**: Executes a standalone statement or declaration: `SmallVector<llvm::Value *> Operands;`. / 执行一条独立语句或声明：`SmallVector<llvm::Value *> Operands;`。
- **L142**: Executes call or statement centered on `Operands.reserve`. / 执行以 `Operands.reserve` 为核心的调用或语句。
- **L143**: Starts a loop over a range or sequence: `for (auto [OpNum, Use] : enumerate(LLVMI->operands()))`. / 开始遍历某个范围或序列的循环：`for (auto [OpNum, Use] : enumerate(LLVMI->operands()))`。
- **L144**: Executes call or statement centered on `Operands.push_back`. / 执行以 `Operands.push_back` 为核心的调用或语句。
- **L145**: Executes call or statement centered on `InstrData.push_back`. / 执行以 `InstrData.push_back` 为核心的调用或语句。
- **L146**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L147**: Checks an internal invariant with an assertion: `assert(is_sorted(InstrData,`. / 通过断言检查内部不变式：`assert(is_sorted(InstrData,`。
- **L148**: Starts the definition of function or method `[]`. / 开始定义函数或方法 `[]`。
- **L149**: Returns control, optionally with a value: `return D0.LLVMI->comesBefore(D1.LLVMI);`. / 返回控制流，并可附带返回值：`return D0.LLVMI->comesBefore(D1.LLVMI);`。
- **L150**: Continues the surrounding expression or declaration: `}) &&`. / 继续构造周围的表达式或声明：`}) &&`。
- **L151**: Executes a standalone statement or declaration: `"Expected reverse program order!");`. / 执行一条独立语句或声明：`"Expected reverse program order!");`。
- **L152**: Initializes or updates `auto *BotLLVMI` from the right-hand expression. / 使用右侧表达式初始化或更新 `auto *BotLLVMI`。
- **L153**: Introduces a conditional branch: `if (BotLLVMI->getNextNode() != nullptr)`. / 引入条件分支：`if (BotLLVMI->getNextNode() != nullptr)`。
- **L154**: Initializes or updates `NextLLVMIOrBB` from the right-hand expression. / 使用右侧表达式初始化或更新 `NextLLVMIOrBB`。
- **L155**: Provides the fallback branch for earlier conditions: `else`. / 为前面的条件提供兜底分支：`else`。
- **L156**: Initializes or updates `NextLLVMIOrBB` from the right-hand expression. / 使用右侧表达式初始化或更新 `NextLLVMIOrBB`。
- **L157**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L158**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L159**: Starts the definition of function or method `EraseFromParent::accept`. / 开始定义函数或方法 `EraseFromParent::accept`。
- **L160**: Starts a loop over a range or sequence: `for (const auto &IData : InstrData)`. / 开始遍历某个范围或序列的循环：`for (const auto &IData : InstrData)`。

### Lines 161-180

```cpp
    IData.LLVMI->deleteValue();
}

void EraseFromParent::revert(Tracker &Tracker) {
  // Place the bottom-most instruction first.
  auto [Operands, BotLLVMI] = InstrData[0];
  if (auto *NextLLVMI = dyn_cast<llvm::Instruction *>(NextLLVMIOrBB)) {
    BotLLVMI->insertBefore(NextLLVMI->getIterator());
  } else {
    auto *LLVMBB = cast<llvm::BasicBlock *>(NextLLVMIOrBB);
    BotLLVMI->insertInto(LLVMBB, LLVMBB->end());
  }
  for (auto [OpNum, Op] : enumerate(Operands))
    BotLLVMI->setOperand(OpNum, Op);

  // Go over the rest of the instructions and stack them on top.
  for (auto [Operands, LLVMI] : drop_begin(InstrData)) {
    LLVMI->insertBefore(BotLLVMI->getIterator());
    for (auto [OpNum, Op] : enumerate(Operands))
      LLVMI->setOperand(OpNum, Op);
```

- **L161**: Executes call or statement centered on `IData.LLVMI->deleteValue`. / 执行以 `IData.LLVMI->deleteValue` 为核心的调用或语句。
- **L162**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L163**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L164**: Starts the definition of function or method `EraseFromParent::revert`. / 开始定义函数或方法 `EraseFromParent::revert`。
- **L165**: Comment documents the nearby logic or transformation intent: `Place the bottom-most instruction first.`. / 注释说明了附近代码的逻辑或变换意图：`Place the bottom-most instruction first.`。
- **L166**: Initializes or updates `auto [Operands, BotLLVMI]` from the right-hand expression. / 使用右侧表达式初始化或更新 `auto [Operands, BotLLVMI]`。
- **L167**: Introduces a conditional branch: `if (auto *NextLLVMI = dyn_cast<llvm::Instruction *>(NextLLVMIOrBB)) {`. / 引入条件分支：`if (auto *NextLLVMI = dyn_cast<llvm::Instruction *>(NextLLVMIOrBB)) {`。
- **L168**: Executes call or statement centered on `BotLLVMI->insertBefore`. / 执行以 `BotLLVMI->insertBefore` 为核心的调用或语句。
- **L169**: Continues the surrounding expression or declaration: `} else {`. / 继续构造周围的表达式或声明：`} else {`。
- **L170**: Initializes or updates `auto *LLVMBB` from the right-hand expression. / 使用右侧表达式初始化或更新 `auto *LLVMBB`。
- **L171**: Executes call or statement centered on `BotLLVMI->insertInto`. / 执行以 `BotLLVMI->insertInto` 为核心的调用或语句。
- **L172**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L173**: Starts a loop over a range or sequence: `for (auto [OpNum, Op] : enumerate(Operands))`. / 开始遍历某个范围或序列的循环：`for (auto [OpNum, Op] : enumerate(Operands))`。
- **L174**: Executes call or statement centered on `BotLLVMI->setOperand`. / 执行以 `BotLLVMI->setOperand` 为核心的调用或语句。
- **L175**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L176**: Comment documents the nearby logic or transformation intent: `Go over the rest of the instructions and stack them on top.`. / 注释说明了附近代码的逻辑或变换意图：`Go over the rest of the instructions and stack them on top.`。
- **L177**: Starts a loop over a range or sequence: `for (auto [Operands, LLVMI] : drop_begin(InstrData)) {`. / 开始遍历某个范围或序列的循环：`for (auto [Operands, LLVMI] : drop_begin(InstrData)) {`。
- **L178**: Executes call or statement centered on `LLVMI->insertBefore`. / 执行以 `LLVMI->insertBefore` 为核心的调用或语句。
- **L179**: Starts a loop over a range or sequence: `for (auto [OpNum, Op] : enumerate(Operands))`. / 开始遍历某个范围或序列的循环：`for (auto [OpNum, Op] : enumerate(Operands))`。
- **L180**: Executes call or statement centered on `LLVMI->setOperand`. / 执行以 `LLVMI->setOperand` 为核心的调用或语句。

### Lines 181-200

```cpp
    BotLLVMI = LLVMI;
  }
  Tracker.getContext().registerValue(std::move(ErasedIPtr));
}

#ifndef NDEBUG
void EraseFromParent::dump() const {
  dump(dbgs());
  dbgs() << "\n";
}
#endif // NDEBUG

RemoveFromParent::RemoveFromParent(Instruction *RemovedI) : RemovedI(RemovedI) {
  if (auto *NextI = RemovedI->getNextNode())
    NextInstrOrBB = NextI;
  else
    NextInstrOrBB = RemovedI->getParent();
}

void RemoveFromParent::revert(Tracker &Tracker) {
```

- **L181**: Initializes or updates `BotLLVMI` from the right-hand expression. / 使用右侧表达式初始化或更新 `BotLLVMI`。
- **L182**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L183**: Executes call or statement centered on `Tracker.getContext`. / 执行以 `Tracker.getContext` 为核心的调用或语句。
- **L184**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L185**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L186**: Preprocessor directive controls conditional compilation or build behavior: `#ifndef NDEBUG`. / 预处理指令控制条件编译或构建行为：`#ifndef NDEBUG`。
- **L187**: Starts the definition of function or method `EraseFromParent::dump`. / 开始定义函数或方法 `EraseFromParent::dump`。
- **L188**: Executes call or statement centered on `dump`. / 执行以 `dump` 为核心的调用或语句。
- **L189**: Executes call or statement centered on `dbgs`. / 执行以 `dbgs` 为核心的调用或语句。
- **L190**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L191**: Preprocessor directive controls conditional compilation or build behavior: `#endif // NDEBUG`. / 预处理指令控制条件编译或构建行为：`#endif // NDEBUG`。
- **L192**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L193**: Starts the definition of function or method `RemoveFromParent::RemoveFromParent`. / 开始定义函数或方法 `RemoveFromParent::RemoveFromParent`。
- **L194**: Introduces a conditional branch: `if (auto *NextI = RemovedI->getNextNode())`. / 引入条件分支：`if (auto *NextI = RemovedI->getNextNode())`。
- **L195**: Initializes or updates `NextInstrOrBB` from the right-hand expression. / 使用右侧表达式初始化或更新 `NextInstrOrBB`。
- **L196**: Provides the fallback branch for earlier conditions: `else`. / 为前面的条件提供兜底分支：`else`。
- **L197**: Initializes or updates `NextInstrOrBB` from the right-hand expression. / 使用右侧表达式初始化或更新 `NextInstrOrBB`。
- **L198**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L199**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L200**: Starts the definition of function or method `RemoveFromParent::revert`. / 开始定义函数或方法 `RemoveFromParent::revert`。

### Lines 201-220

```cpp
  if (auto *NextI = dyn_cast<Instruction *>(NextInstrOrBB)) {
    RemovedI->insertBefore(NextI);
  } else {
    auto *BB = cast<BasicBlock *>(NextInstrOrBB);
    RemovedI->insertInto(BB, BB->end());
  }
}

#ifndef NDEBUG
void RemoveFromParent::dump() const {
  dump(dbgs());
  dbgs() << "\n";
}
#endif

CatchSwitchAddHandler::CatchSwitchAddHandler(CatchSwitchInst *CSI)
    : CSI(CSI), HandlerIdx(CSI->getNumHandlers()) {}

void CatchSwitchAddHandler::revert(Tracker &Tracker) {
  // TODO: This should ideally use sandboxir::CatchSwitchInst::removeHandler()
```

- **L201**: Introduces a conditional branch: `if (auto *NextI = dyn_cast<Instruction *>(NextInstrOrBB)) {`. / 引入条件分支：`if (auto *NextI = dyn_cast<Instruction *>(NextInstrOrBB)) {`。
- **L202**: Executes call or statement centered on `RemovedI->insertBefore`. / 执行以 `RemovedI->insertBefore` 为核心的调用或语句。
- **L203**: Continues the surrounding expression or declaration: `} else {`. / 继续构造周围的表达式或声明：`} else {`。
- **L204**: Initializes or updates `auto *BB` from the right-hand expression. / 使用右侧表达式初始化或更新 `auto *BB`。
- **L205**: Executes call or statement centered on `RemovedI->insertInto`. / 执行以 `RemovedI->insertInto` 为核心的调用或语句。
- **L206**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L207**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L208**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L209**: Preprocessor directive controls conditional compilation or build behavior: `#ifndef NDEBUG`. / 预处理指令控制条件编译或构建行为：`#ifndef NDEBUG`。
- **L210**: Starts the definition of function or method `RemoveFromParent::dump`. / 开始定义函数或方法 `RemoveFromParent::dump`。
- **L211**: Executes call or statement centered on `dump`. / 执行以 `dump` 为核心的调用或语句。
- **L212**: Executes call or statement centered on `dbgs`. / 执行以 `dbgs` 为核心的调用或语句。
- **L213**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L214**: Preprocessor directive controls conditional compilation or build behavior: `#endif`. / 预处理指令控制条件编译或构建行为：`#endif`。
- **L215**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L216**: Continues the surrounding expression or declaration: `CatchSwitchAddHandler::CatchSwitchAddHandler(CatchSwitchInst *CSI)`. / 继续构造周围的表达式或声明：`CatchSwitchAddHandler::CatchSwitchAddHandler(CatchSwitchInst *CSI)`。
- **L217**: Continues a multi-line argument list or initializer: `: CSI(CSI), HandlerIdx(CSI->getNumHandlers()) {}`. / 继续一个多行参数列表或初始化器：`: CSI(CSI), HandlerIdx(CSI->getNumHandlers()) {}`。
- **L218**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L219**: Starts the definition of function or method `CatchSwitchAddHandler::revert`. / 开始定义函数或方法 `CatchSwitchAddHandler::revert`。
- **L220**: Comment highlights an implementation note: `TODO: This should ideally use sandboxir::CatchSwitchInst::removeHandler()`. / 注释强调了一条实现说明：`TODO: This should ideally use sandboxir::CatchSwitchInst::removeHandler()`。

### Lines 221-240

```cpp
  // once it gets implemented.
  auto *LLVMCSI = cast<llvm::CatchSwitchInst>(CSI->Val);
  LLVMCSI->removeHandler(LLVMCSI->handler_begin() + HandlerIdx);
}

SwitchRemoveCase::SwitchRemoveCase(SwitchInst *Switch) : Switch(Switch) {
  for (const auto &C : Switch->cases())
    Cases.push_back({C.getCaseValue(), C.getCaseSuccessor()});
}

void SwitchRemoveCase::revert(Tracker &Tracker) {
  // SwitchInst::removeCase doesn't provide any guarantees about the order of
  // cases after removal. In order to preserve the original ordering, we save
  // all of them and, when reverting, clear them all then insert them in the
  // desired order. This still relies on the fact that `addCase` will insert
  // them at the end, but it is documented to invalidate `case_end()` so it's
  // probably okay.
  unsigned NumCases = Switch->getNumCases();
  for (unsigned I = 0; I < NumCases; ++I)
    Switch->removeCase(Switch->case_begin());
```

- **L221**: Comment documents the nearby logic or transformation intent: `once it gets implemented.`. / 注释说明了附近代码的逻辑或变换意图：`once it gets implemented.`。
- **L222**: Initializes or updates `auto *LLVMCSI` from the right-hand expression. / 使用右侧表达式初始化或更新 `auto *LLVMCSI`。
- **L223**: Executes call or statement centered on `LLVMCSI->removeHandler`. / 执行以 `LLVMCSI->removeHandler` 为核心的调用或语句。
- **L224**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L225**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L226**: Starts the definition of function or method `SwitchRemoveCase::SwitchRemoveCase`. / 开始定义函数或方法 `SwitchRemoveCase::SwitchRemoveCase`。
- **L227**: Starts a loop over a range or sequence: `for (const auto &C : Switch->cases())`. / 开始遍历某个范围或序列的循环：`for (const auto &C : Switch->cases())`。
- **L228**: Executes call or statement centered on `Cases.push_back`. / 执行以 `Cases.push_back` 为核心的调用或语句。
- **L229**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L230**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L231**: Starts the definition of function or method `SwitchRemoveCase::revert`. / 开始定义函数或方法 `SwitchRemoveCase::revert`。
- **L232**: Comment documents the nearby logic or transformation intent: `SwitchInst::removeCase doesn't provide any guarantees about the order of`. / 注释说明了附近代码的逻辑或变换意图：`SwitchInst::removeCase doesn't provide any guarantees about the order of`。
- **L233**: Comment documents the nearby logic or transformation intent: `cases after removal. In order to preserve the original ordering, we save`. / 注释说明了附近代码的逻辑或变换意图：`cases after removal. In order to preserve the original ordering, we save`。
- **L234**: Comment documents the nearby logic or transformation intent: `all of them and, when reverting, clear them all then insert them in the`. / 注释说明了附近代码的逻辑或变换意图：`all of them and, when reverting, clear them all then insert them in the`。
- **L235**: Comment documents the nearby logic or transformation intent: `desired order. This still relies on the fact that \`addCase\` will insert`. / 注释说明了附近代码的逻辑或变换意图：`desired order. This still relies on the fact that \`addCase\` will insert`。
- **L236**: Comment documents the nearby logic or transformation intent: `them at the end, but it is documented to invalidate \`case_end()\` so it's`. / 注释说明了附近代码的逻辑或变换意图：`them at the end, but it is documented to invalidate \`case_end()\` so it's`。
- **L237**: Comment documents the nearby logic or transformation intent: `probably okay.`. / 注释说明了附近代码的逻辑或变换意图：`probably okay.`。
- **L238**: Initializes or updates `unsigned NumCases` from the right-hand expression. / 使用右侧表达式初始化或更新 `unsigned NumCases`。
- **L239**: Starts a loop over a range or sequence: `for (unsigned I = 0; I < NumCases; ++I)`. / 开始遍历某个范围或序列的循环：`for (unsigned I = 0; I < NumCases; ++I)`。
- **L240**: Executes call or statement centered on `Switch->removeCase`. / 执行以 `Switch->removeCase` 为核心的调用或语句。

### Lines 241-260

```cpp
  for (auto &Case : Cases)
    Switch->addCase(Case.Val, Case.Dest);
}

#ifndef NDEBUG
void SwitchRemoveCase::dump() const {
  dump(dbgs());
  dbgs() << "\n";
}
#endif // NDEBUG

void SwitchAddCase::revert(Tracker &Tracker) {
  auto It = Switch->findCaseValue(Val);
  Switch->removeCase(It);
}

#ifndef NDEBUG
void SwitchAddCase::dump() const {
  dump(dbgs());
  dbgs() << "\n";
```

- **L241**: Starts a loop over a range or sequence: `for (auto &Case : Cases)`. / 开始遍历某个范围或序列的循环：`for (auto &Case : Cases)`。
- **L242**: Executes call or statement centered on `Switch->addCase`. / 执行以 `Switch->addCase` 为核心的调用或语句。
- **L243**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L244**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L245**: Preprocessor directive controls conditional compilation or build behavior: `#ifndef NDEBUG`. / 预处理指令控制条件编译或构建行为：`#ifndef NDEBUG`。
- **L246**: Starts the definition of function or method `SwitchRemoveCase::dump`. / 开始定义函数或方法 `SwitchRemoveCase::dump`。
- **L247**: Executes call or statement centered on `dump`. / 执行以 `dump` 为核心的调用或语句。
- **L248**: Executes call or statement centered on `dbgs`. / 执行以 `dbgs` 为核心的调用或语句。
- **L249**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L250**: Preprocessor directive controls conditional compilation or build behavior: `#endif // NDEBUG`. / 预处理指令控制条件编译或构建行为：`#endif // NDEBUG`。
- **L251**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L252**: Starts the definition of function or method `SwitchAddCase::revert`. / 开始定义函数或方法 `SwitchAddCase::revert`。
- **L253**: Initializes or updates `auto It` from the right-hand expression. / 使用右侧表达式初始化或更新 `auto It`。
- **L254**: Executes call or statement centered on `Switch->removeCase`. / 执行以 `Switch->removeCase` 为核心的调用或语句。
- **L255**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L256**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L257**: Preprocessor directive controls conditional compilation or build behavior: `#ifndef NDEBUG`. / 预处理指令控制条件编译或构建行为：`#ifndef NDEBUG`。
- **L258**: Starts the definition of function or method `SwitchAddCase::dump`. / 开始定义函数或方法 `SwitchAddCase::dump`。
- **L259**: Executes call or statement centered on `dump`. / 执行以 `dump` 为核心的调用或语句。
- **L260**: Executes call or statement centered on `dbgs`. / 执行以 `dbgs` 为核心的调用或语句。

### Lines 261-280

```cpp
}
#endif // NDEBUG

MoveInstr::MoveInstr(Instruction *MovedI) : MovedI(MovedI) {
  if (auto *NextI = MovedI->getNextNode())
    NextInstrOrBB = NextI;
  else
    NextInstrOrBB = MovedI->getParent();
}

void MoveInstr::revert(Tracker &Tracker) {
  if (auto *NextI = dyn_cast<Instruction *>(NextInstrOrBB)) {
    MovedI->moveBefore(NextI);
  } else {
    auto *BB = cast<BasicBlock *>(NextInstrOrBB);
    MovedI->moveBefore(*BB, BB->end());
  }
}

#ifndef NDEBUG
```

- **L261**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L262**: Preprocessor directive controls conditional compilation or build behavior: `#endif // NDEBUG`. / 预处理指令控制条件编译或构建行为：`#endif // NDEBUG`。
- **L263**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L264**: Starts the definition of function or method `MoveInstr::MoveInstr`. / 开始定义函数或方法 `MoveInstr::MoveInstr`。
- **L265**: Introduces a conditional branch: `if (auto *NextI = MovedI->getNextNode())`. / 引入条件分支：`if (auto *NextI = MovedI->getNextNode())`。
- **L266**: Initializes or updates `NextInstrOrBB` from the right-hand expression. / 使用右侧表达式初始化或更新 `NextInstrOrBB`。
- **L267**: Provides the fallback branch for earlier conditions: `else`. / 为前面的条件提供兜底分支：`else`。
- **L268**: Initializes or updates `NextInstrOrBB` from the right-hand expression. / 使用右侧表达式初始化或更新 `NextInstrOrBB`。
- **L269**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L270**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L271**: Starts the definition of function or method `MoveInstr::revert`. / 开始定义函数或方法 `MoveInstr::revert`。
- **L272**: Introduces a conditional branch: `if (auto *NextI = dyn_cast<Instruction *>(NextInstrOrBB)) {`. / 引入条件分支：`if (auto *NextI = dyn_cast<Instruction *>(NextInstrOrBB)) {`。
- **L273**: Executes call or statement centered on `MovedI->moveBefore`. / 执行以 `MovedI->moveBefore` 为核心的调用或语句。
- **L274**: Continues the surrounding expression or declaration: `} else {`. / 继续构造周围的表达式或声明：`} else {`。
- **L275**: Initializes or updates `auto *BB` from the right-hand expression. / 使用右侧表达式初始化或更新 `auto *BB`。
- **L276**: Executes call or statement centered on `MovedI->moveBefore`. / 执行以 `MovedI->moveBefore` 为核心的调用或语句。
- **L277**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L278**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L279**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L280**: Preprocessor directive controls conditional compilation or build behavior: `#ifndef NDEBUG`. / 预处理指令控制条件编译或构建行为：`#ifndef NDEBUG`。

### Lines 281-300

```cpp
void MoveInstr::dump() const {
  dump(dbgs());
  dbgs() << "\n";
}
#endif

void InsertIntoBB::revert(Tracker &Tracker) { InsertedI->removeFromParent(); }

InsertIntoBB::InsertIntoBB(Instruction *InsertedI) : InsertedI(InsertedI) {}

#ifndef NDEBUG
void InsertIntoBB::dump() const {
  dump(dbgs());
  dbgs() << "\n";
}
#endif

void CreateAndInsertInst::revert(Tracker &Tracker) { NewI->eraseFromParent(); }

#ifndef NDEBUG
```

- **L281**: Starts the definition of function or method `MoveInstr::dump`. / 开始定义函数或方法 `MoveInstr::dump`。
- **L282**: Executes call or statement centered on `dump`. / 执行以 `dump` 为核心的调用或语句。
- **L283**: Executes call or statement centered on `dbgs`. / 执行以 `dbgs` 为核心的调用或语句。
- **L284**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L285**: Preprocessor directive controls conditional compilation or build behavior: `#endif`. / 预处理指令控制条件编译或构建行为：`#endif`。
- **L286**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L287**: Continues the surrounding expression or declaration: `void InsertIntoBB::revert(Tracker &Tracker) { InsertedI->removeFromParent(); }`. / 继续构造周围的表达式或声明：`void InsertIntoBB::revert(Tracker &Tracker) { InsertedI->removeFromParent(); }`。
- **L288**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L289**: Continues the surrounding expression or declaration: `InsertIntoBB::InsertIntoBB(Instruction *InsertedI) : InsertedI(InsertedI) {}`. / 继续构造周围的表达式或声明：`InsertIntoBB::InsertIntoBB(Instruction *InsertedI) : InsertedI(InsertedI) {}`。
- **L290**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L291**: Preprocessor directive controls conditional compilation or build behavior: `#ifndef NDEBUG`. / 预处理指令控制条件编译或构建行为：`#ifndef NDEBUG`。
- **L292**: Starts the definition of function or method `InsertIntoBB::dump`. / 开始定义函数或方法 `InsertIntoBB::dump`。
- **L293**: Executes call or statement centered on `dump`. / 执行以 `dump` 为核心的调用或语句。
- **L294**: Executes call or statement centered on `dbgs`. / 执行以 `dbgs` 为核心的调用或语句。
- **L295**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L296**: Preprocessor directive controls conditional compilation or build behavior: `#endif`. / 预处理指令控制条件编译或构建行为：`#endif`。
- **L297**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L298**: Continues the surrounding expression or declaration: `void CreateAndInsertInst::revert(Tracker &Tracker) { NewI->eraseFromParent(); }`. / 继续构造周围的表达式或声明：`void CreateAndInsertInst::revert(Tracker &Tracker) { NewI->eraseFromParent(); }`。
- **L299**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L300**: Preprocessor directive controls conditional compilation or build behavior: `#ifndef NDEBUG`. / 预处理指令控制条件编译或构建行为：`#ifndef NDEBUG`。

### Lines 301-320

```cpp
void CreateAndInsertInst::dump() const {
  dump(dbgs());
  dbgs() << "\n";
}
#endif

ShuffleVectorSetMask::ShuffleVectorSetMask(ShuffleVectorInst *SVI)
    : SVI(SVI), PrevMask(SVI->getShuffleMask()) {}

void ShuffleVectorSetMask::revert(Tracker &Tracker) {
  SVI->setShuffleMask(PrevMask);
}

#ifndef NDEBUG
void ShuffleVectorSetMask::dump() const {
  dump(dbgs());
  dbgs() << "\n";
}
#endif

```

- **L301**: Starts the definition of function or method `CreateAndInsertInst::dump`. / 开始定义函数或方法 `CreateAndInsertInst::dump`。
- **L302**: Executes call or statement centered on `dump`. / 执行以 `dump` 为核心的调用或语句。
- **L303**: Executes call or statement centered on `dbgs`. / 执行以 `dbgs` 为核心的调用或语句。
- **L304**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L305**: Preprocessor directive controls conditional compilation or build behavior: `#endif`. / 预处理指令控制条件编译或构建行为：`#endif`。
- **L306**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L307**: Continues the surrounding expression or declaration: `ShuffleVectorSetMask::ShuffleVectorSetMask(ShuffleVectorInst *SVI)`. / 继续构造周围的表达式或声明：`ShuffleVectorSetMask::ShuffleVectorSetMask(ShuffleVectorInst *SVI)`。
- **L308**: Continues a multi-line argument list or initializer: `: SVI(SVI), PrevMask(SVI->getShuffleMask()) {}`. / 继续一个多行参数列表或初始化器：`: SVI(SVI), PrevMask(SVI->getShuffleMask()) {}`。
- **L309**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L310**: Starts the definition of function or method `ShuffleVectorSetMask::revert`. / 开始定义函数或方法 `ShuffleVectorSetMask::revert`。
- **L311**: Executes call or statement centered on `SVI->setShuffleMask`. / 执行以 `SVI->setShuffleMask` 为核心的调用或语句。
- **L312**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L313**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L314**: Preprocessor directive controls conditional compilation or build behavior: `#ifndef NDEBUG`. / 预处理指令控制条件编译或构建行为：`#ifndef NDEBUG`。
- **L315**: Starts the definition of function or method `ShuffleVectorSetMask::dump`. / 开始定义函数或方法 `ShuffleVectorSetMask::dump`。
- **L316**: Executes call or statement centered on `dump`. / 执行以 `dump` 为核心的调用或语句。
- **L317**: Executes call or statement centered on `dbgs`. / 执行以 `dbgs` 为核心的调用或语句。
- **L318**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L319**: Preprocessor directive controls conditional compilation or build behavior: `#endif`. / 预处理指令控制条件编译或构建行为：`#endif`。
- **L320**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 321-340

```cpp
CmpSwapOperands::CmpSwapOperands(CmpInst *Cmp) : Cmp(Cmp) {}

void CmpSwapOperands::revert(Tracker &Tracker) { Cmp->swapOperands(); }
#ifndef NDEBUG
void CmpSwapOperands::dump() const {
  dump(dbgs());
  dbgs() << "\n";
}
#endif

void Tracker::save() {
  State = TrackerState::Record;
  // Record the last index in `Changes` that we will revert.
  Snapshots.push_back(Changes.size());
#if !defined(NDEBUG) && defined(EXPENSIVE_CHECKS)
  SnapshotChecker.emplace_back(Ctx);
  SnapshotChecker.back().save();
#endif
}

```

- **L321**: Continues the surrounding expression or declaration: `CmpSwapOperands::CmpSwapOperands(CmpInst *Cmp) : Cmp(Cmp) {}`. / 继续构造周围的表达式或声明：`CmpSwapOperands::CmpSwapOperands(CmpInst *Cmp) : Cmp(Cmp) {}`。
- **L322**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L323**: Continues the surrounding expression or declaration: `void CmpSwapOperands::revert(Tracker &Tracker) { Cmp->swapOperands(); }`. / 继续构造周围的表达式或声明：`void CmpSwapOperands::revert(Tracker &Tracker) { Cmp->swapOperands(); }`。
- **L324**: Preprocessor directive controls conditional compilation or build behavior: `#ifndef NDEBUG`. / 预处理指令控制条件编译或构建行为：`#ifndef NDEBUG`。
- **L325**: Starts the definition of function or method `CmpSwapOperands::dump`. / 开始定义函数或方法 `CmpSwapOperands::dump`。
- **L326**: Executes call or statement centered on `dump`. / 执行以 `dump` 为核心的调用或语句。
- **L327**: Executes call or statement centered on `dbgs`. / 执行以 `dbgs` 为核心的调用或语句。
- **L328**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L329**: Preprocessor directive controls conditional compilation or build behavior: `#endif`. / 预处理指令控制条件编译或构建行为：`#endif`。
- **L330**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L331**: Starts the definition of function or method `Tracker::save`. / 开始定义函数或方法 `Tracker::save`。
- **L332**: Initializes or updates `State` from the right-hand expression. / 使用右侧表达式初始化或更新 `State`。
- **L333**: Comment documents the nearby logic or transformation intent: `Record the last index in \`Changes\` that we will revert.`. / 注释说明了附近代码的逻辑或变换意图：`Record the last index in \`Changes\` that we will revert.`。
- **L334**: Executes call or statement centered on `Snapshots.push_back`. / 执行以 `Snapshots.push_back` 为核心的调用或语句。
- **L335**: Preprocessor directive controls conditional compilation or build behavior: `#if !defined(NDEBUG) && defined(EXPENSIVE_CHECKS)`. / 预处理指令控制条件编译或构建行为：`#if !defined(NDEBUG) && defined(EXPENSIVE_CHECKS)`。
- **L336**: Executes call or statement centered on `SnapshotChecker.emplace_back`. / 执行以 `SnapshotChecker.emplace_back` 为核心的调用或语句。
- **L337**: Executes call or statement centered on `SnapshotChecker.back`. / 执行以 `SnapshotChecker.back` 为核心的调用或语句。
- **L338**: Preprocessor directive controls conditional compilation or build behavior: `#endif`. / 预处理指令控制条件编译或构建行为：`#endif`。
- **L339**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L340**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 341-360

```cpp
void Tracker::revert() {
  assert(State == TrackerState::Record && "Forgot to save()!");
  State = TrackerState::Reverting;
  const unsigned ToRevert = Changes.size() - Snapshots.back();
  unsigned CntReverts = 0;
  for (auto &Change : reverse(Changes)) {
    // Stop reverting if we reach the index of the last snapshot.
    if (CntReverts++ == ToRevert)
      break;
    Change->revert(*this);
  }
  Changes.erase(Changes.end() - ToRevert, Changes.end());
  Snapshots.pop_back();
#if !defined(NDEBUG) && defined(EXPENSIVE_CHECKS)
  SnapshotChecker.back().expectNoDiff();
  SnapshotChecker.pop_back();
#endif
  State = Snapshots.empty() ? TrackerState::Disabled : TrackerState::Record;
}

```

- **L341**: Starts the definition of function or method `Tracker::revert`. / 开始定义函数或方法 `Tracker::revert`。
- **L342**: Checks an internal invariant with an assertion: `assert(State == TrackerState::Record && "Forgot to save()!");`. / 通过断言检查内部不变式：`assert(State == TrackerState::Record && "Forgot to save()!");`。
- **L343**: Initializes or updates `State` from the right-hand expression. / 使用右侧表达式初始化或更新 `State`。
- **L344**: Initializes or updates `const unsigned ToRevert` from the right-hand expression. / 使用右侧表达式初始化或更新 `const unsigned ToRevert`。
- **L345**: Initializes or updates `unsigned CntReverts` from the right-hand expression. / 使用右侧表达式初始化或更新 `unsigned CntReverts`。
- **L346**: Starts a loop over a range or sequence: `for (auto &Change : reverse(Changes)) {`. / 开始遍历某个范围或序列的循环：`for (auto &Change : reverse(Changes)) {`。
- **L347**: Comment documents the nearby logic or transformation intent: `Stop reverting if we reach the index of the last snapshot.`. / 注释说明了附近代码的逻辑或变换意图：`Stop reverting if we reach the index of the last snapshot.`。
- **L348**: Introduces a conditional branch: `if (CntReverts++ == ToRevert)`. / 引入条件分支：`if (CntReverts++ == ToRevert)`。
- **L349**: Executes a standalone statement or declaration: `break;`. / 执行一条独立语句或声明：`break;`。
- **L350**: Executes call or statement centered on `Change->revert`. / 执行以 `Change->revert` 为核心的调用或语句。
- **L351**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L352**: Executes call or statement centered on `Changes.erase`. / 执行以 `Changes.erase` 为核心的调用或语句。
- **L353**: Executes call or statement centered on `Snapshots.pop_back`. / 执行以 `Snapshots.pop_back` 为核心的调用或语句。
- **L354**: Preprocessor directive controls conditional compilation or build behavior: `#if !defined(NDEBUG) && defined(EXPENSIVE_CHECKS)`. / 预处理指令控制条件编译或构建行为：`#if !defined(NDEBUG) && defined(EXPENSIVE_CHECKS)`。
- **L355**: Executes call or statement centered on `SnapshotChecker.back`. / 执行以 `SnapshotChecker.back` 为核心的调用或语句。
- **L356**: Executes call or statement centered on `SnapshotChecker.pop_back`. / 执行以 `SnapshotChecker.pop_back` 为核心的调用或语句。
- **L357**: Preprocessor directive controls conditional compilation or build behavior: `#endif`. / 预处理指令控制条件编译或构建行为：`#endif`。
- **L358**: Initializes or updates `State` from the right-hand expression. / 使用右侧表达式初始化或更新 `State`。
- **L359**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L360**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 361-380

```cpp
void Tracker::accept() {
  assert(State == TrackerState::Record && "Forgot to save()!");
  State = TrackerState::Disabled;
  for (auto &Change : Changes)
    Change->accept();
  Changes.clear();
  Snapshots.clear();
#if !defined(NDEBUG) && defined(EXPENSIVE_CHECKS)
  SnapshotChecker.clear();
#endif
}

#ifndef NDEBUG
void Tracker::dump(raw_ostream &OS) const {
  unsigned SnapshotCnt = 0;
  for (auto [Idx, ChangePtr] : enumerate(Changes)) {
    OS << Idx << ". ";
    ChangePtr->dump(OS);
    if (find(Snapshots, Idx) != Snapshots.end())
      OS << " [Snapshot " << SnapshotCnt++ << "]";
```

- **L361**: Starts the definition of function or method `Tracker::accept`. / 开始定义函数或方法 `Tracker::accept`。
- **L362**: Checks an internal invariant with an assertion: `assert(State == TrackerState::Record && "Forgot to save()!");`. / 通过断言检查内部不变式：`assert(State == TrackerState::Record && "Forgot to save()!");`。
- **L363**: Initializes or updates `State` from the right-hand expression. / 使用右侧表达式初始化或更新 `State`。
- **L364**: Starts a loop over a range or sequence: `for (auto &Change : Changes)`. / 开始遍历某个范围或序列的循环：`for (auto &Change : Changes)`。
- **L365**: Executes call or statement centered on `Change->accept`. / 执行以 `Change->accept` 为核心的调用或语句。
- **L366**: Executes call or statement centered on `Changes.clear`. / 执行以 `Changes.clear` 为核心的调用或语句。
- **L367**: Executes call or statement centered on `Snapshots.clear`. / 执行以 `Snapshots.clear` 为核心的调用或语句。
- **L368**: Preprocessor directive controls conditional compilation or build behavior: `#if !defined(NDEBUG) && defined(EXPENSIVE_CHECKS)`. / 预处理指令控制条件编译或构建行为：`#if !defined(NDEBUG) && defined(EXPENSIVE_CHECKS)`。
- **L369**: Executes call or statement centered on `SnapshotChecker.clear`. / 执行以 `SnapshotChecker.clear` 为核心的调用或语句。
- **L370**: Preprocessor directive controls conditional compilation or build behavior: `#endif`. / 预处理指令控制条件编译或构建行为：`#endif`。
- **L371**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L372**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L373**: Preprocessor directive controls conditional compilation or build behavior: `#ifndef NDEBUG`. / 预处理指令控制条件编译或构建行为：`#ifndef NDEBUG`。
- **L374**: Starts the definition of function or method `Tracker::dump`. / 开始定义函数或方法 `Tracker::dump`。
- **L375**: Initializes or updates `unsigned SnapshotCnt` from the right-hand expression. / 使用右侧表达式初始化或更新 `unsigned SnapshotCnt`。
- **L376**: Starts a loop over a range or sequence: `for (auto [Idx, ChangePtr] : enumerate(Changes)) {`. / 开始遍历某个范围或序列的循环：`for (auto [Idx, ChangePtr] : enumerate(Changes)) {`。
- **L377**: Executes a standalone statement or declaration: `OS << Idx << ". ";`. / 执行一条独立语句或声明：`OS << Idx << ". ";`。
- **L378**: Executes call or statement centered on `ChangePtr->dump`. / 执行以 `ChangePtr->dump` 为核心的调用或语句。
- **L379**: Introduces a conditional branch: `if (find(Snapshots, Idx) != Snapshots.end())`. / 引入条件分支：`if (find(Snapshots, Idx) != Snapshots.end())`。
- **L380**: Executes a standalone statement or declaration: `OS << " [Snapshot " << SnapshotCnt++ << "]";`. / 执行一条独立语句或声明：`OS << " [Snapshot " << SnapshotCnt++ << "]";`。

### Lines 381-388

```cpp
    OS << "\n";
  }
}
void Tracker::dump() const {
  dump(dbgs());
  dbgs() << "\n";
}
#endif // NDEBUG
```

- **L381**: Executes a standalone statement or declaration: `OS << "\n";`. / 执行一条独立语句或声明：`OS << "\n";`。
- **L382**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L383**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L384**: Starts the definition of function or method `Tracker::dump`. / 开始定义函数或方法 `Tracker::dump`。
- **L385**: Executes call or statement centered on `dump`. / 执行以 `dump` 为核心的调用或语句。
- **L386**: Executes call or statement centered on `dbgs`. / 执行以 `dbgs` 为核心的调用或语句。
- **L387**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L388**: Preprocessor directive controls conditional compilation or build behavior: `#endif // NDEBUG`. / 预处理指令控制条件编译或构建行为：`#endif // NDEBUG`。

## Key Concepts / 关键概念

- **LLVM utility/container usage / LLVM 工具与容器使用**
- **Core LLVM data model interaction / LLVM 核心数据模型交互**
- **Module/file-scoped coordination / 模块/文件级协调**
- **`Tracker` focused implementation / 围绕 `Tracker` 的实现逻辑**

## Dependencies / 依赖关系

- `llvm/SandboxIR/Tracker.h`: Provides local declarations used by this file. / 提供本文件使用的本地声明。
- `llvm/ADT/STLExtras.h`: Provides LLVM ADT data structures/utilities. / 提供LLVM ADT 数据结构/工具。
- `llvm/IR/BasicBlock.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/Instruction.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/StructuralHash.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/SandboxIR/Instruction.h`: Provides local declarations used by this file. / 提供本文件使用的本地声明。
