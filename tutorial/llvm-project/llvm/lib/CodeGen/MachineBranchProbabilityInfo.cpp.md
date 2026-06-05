# MachineBranchProbabilityInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/CodeGen/MachineBranchProbabilityInfo.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file contains `Machine Branch Probability Info` logic inside LLVM CodeGen.
- **Purpose (CN)**: 该文件在 LLVM CodeGen 中实现与“Machine Branch Probability Info”相关的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===- MachineBranchProbabilityInfo.cpp - Machine Branch Probability Info -===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This analysis uses probability info stored in Machine Basic Blocks.
//
//===----------------------------------------------------------------------===//

#include "llvm/CodeGen/MachineBranchProbabilityInfo.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/InitializePasses.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/raw_ostream.h"

using namespace llvm;

````
- **L1 EN**: Comment documents: `===- MachineBranchProbabilityInfo.cpp - Machine Branch Probability Info …`.
  **L1 CN**: 注释说明：`===- MachineBranchProbabilityInfo.cpp - Machine Branch Probability Info …`。
- **L2 EN**: Continues the surrounding comment block.
  **L2 CN**: 延续周围的注释块。
- **L3 EN**: Comment documents: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Except…`.
  **L3 CN**: 注释说明：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Except…`。
- **L4 EN**: Comment documents: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment documents: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Continues the surrounding comment block.
  **L6 CN**: 延续周围的注释块。
- **L7 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L7 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L8 EN**: Continues the surrounding comment block.
  **L8 CN**: 延续周围的注释块。
- **L9 EN**: Comment documents: `This analysis uses probability info stored in Machine Basic Blocks.`.
  **L9 CN**: 注释说明：`This analysis uses probability info stored in Machine Basic Blocks.`。
- **L10 EN**: Continues the surrounding comment block.
  **L10 CN**: 延续周围的注释块。
- **L11 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L11 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L12 EN**: Separates nearby statements for readability.
  **L12 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L13 EN**: Includes LLVM header `llvm/CodeGen/MachineBranchProbabilityInfo.h` for MachineBranchProbabilityInfo support.
  **L13 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineBranchProbabilityInfo.h`，用于 MachineBranchProbabilityInfo 相关支持。
- **L14 EN**: Includes LLVM header `llvm/CodeGen/MachineBasicBlock.h` for MachineBasicBlock support.
  **L14 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineBasicBlock.h`，用于 MachineBasicBlock 相关支持。
- **L15 EN**: Includes LLVM header `llvm/InitializePasses.h` for InitializePasses support.
  **L15 CN**: 引入 LLVM 头文件 `llvm/InitializePasses.h`，用于 InitializePasses 相关支持。
- **L16 EN**: Includes LLVM header `llvm/Support/CommandLine.h` for CommandLine support.
  **L16 CN**: 引入 LLVM 头文件 `llvm/Support/CommandLine.h`，用于 CommandLine 相关支持。
- **L17 EN**: Includes LLVM header `llvm/Support/raw_ostream.h` for raw_ostream support.
  **L17 CN**: 引入 LLVM 头文件 `llvm/Support/raw_ostream.h`，用于 raw_ostream 相关支持。
- **L18 EN**: Separates nearby statements for readability.
  **L18 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L19 EN**: Imports namespace `llvm` into this translation unit.
  **L19 CN**: 将命名空间 `llvm` 引入当前编译单元。
- **L20 EN**: Separates nearby statements for readability.
  **L20 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 21-40

````cpp
INITIALIZE_PASS(MachineBranchProbabilityInfoWrapperPass, "machine-branch-prob",
                "Machine Branch Probability Analysis", false, true)
namespace llvm {
cl::opt<unsigned>
    StaticLikelyProb("static-likely-prob",
                     cl::desc("branch probability threshold in percentage"
                              " to be considered very likely"),
                     cl::init(80), cl::Hidden);

cl::opt<unsigned> ProfileLikelyProb(
    "profile-likely-prob",
    cl::desc("branch probability threshold in percentage to be considered"
             " very likely when profile is available"),
    cl::init(51), cl::Hidden);
} // namespace llvm

MachineBranchProbabilityAnalysis::Result
MachineBranchProbabilityAnalysis::run(MachineFunction &,
                                      MachineFunctionAnalysisManager &) {
  return MachineBranchProbabilityInfo();
````
- **L21 EN**: Continues logic with `INITIALIZE_PASS(MachineBranchProbabilityInfoWrapperPass, "machine-branch…`.
  **L21 CN**: 继续处理逻辑：`INITIALIZE_PASS(MachineBranchProbabilityInfoWrapperPass, "machine-branch…`。
- **L22 EN**: Continues logic with `"Machine Branch Probability Analysis", false, true)`.
  **L22 CN**: 继续处理逻辑：`"Machine Branch Probability Analysis", false, true)`。
- **L23 EN**: Opens namespace `llvm`.
  **L23 CN**: 打开命名空间 `llvm`。
- **L24 EN**: Declares LLVM command-line option `command-line option`.
  **L24 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L25 EN**: Continues logic with `StaticLikelyProb("static-likely-prob",`.
  **L25 CN**: 继续处理逻辑：`StaticLikelyProb("static-likely-prob",`。
- **L26 EN**: Provides part of the signature for `desc`.
  **L26 CN**: 给出 `desc` 的一部分签名。
- **L27 EN**: Continues logic with `" to be considered very likely"),`.
  **L27 CN**: 继续处理逻辑：`" to be considered very likely"),`。
- **L28 EN**: Declares function or method `init`.
  **L28 CN**: 声明函数或方法 `init`。
- **L29 EN**: Separates nearby statements for readability.
  **L29 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L30 EN**: Declares LLVM command-line option `command-line option`.
  **L30 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L31 EN**: Continues logic with `"profile-likely-prob",`.
  **L31 CN**: 继续处理逻辑：`"profile-likely-prob",`。
- **L32 EN**: Provides part of the signature for `desc`.
  **L32 CN**: 给出 `desc` 的一部分签名。
- **L33 EN**: Continues logic with `" very likely when profile is available"),`.
  **L33 CN**: 继续处理逻辑：`" very likely when profile is available"),`。
- **L34 EN**: Declares function or method `init`.
  **L34 CN**: 声明函数或方法 `init`。
- **L35 EN**: Continues logic with `} // namespace llvm`.
  **L35 CN**: 继续处理逻辑：`} // namespace llvm`。
- **L36 EN**: Separates nearby statements for readability.
  **L36 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L37 EN**: Continues logic with `MachineBranchProbabilityAnalysis::Result`.
  **L37 CN**: 继续处理逻辑：`MachineBranchProbabilityAnalysis::Result`。
- **L38 EN**: Provides part of the signature for `run`.
  **L38 CN**: 给出 `run` 的一部分签名。
- **L39 EN**: Starts block `MachineFunctionAnalysisManager &)`.
  **L39 CN**: 开始代码块 `MachineFunctionAnalysisManager &)`。
- **L40 EN**: Returns `MachineBranchProbabilityInfo()` to the caller.
  **L40 CN**: 向调用者返回 `MachineBranchProbabilityInfo()`。

### Lines 41-60

````cpp
}

PreservedAnalyses
MachineBranchProbabilityPrinterPass::run(MachineFunction &MF,
                                         MachineFunctionAnalysisManager &MFAM) {
  OS << "Printing analysis 'Machine Branch Probability Analysis' for machine "
        "function '"
     << MF.getName() << "':\n";
  auto &MBPI = MFAM.getResult<MachineBranchProbabilityAnalysis>(MF);
  for (const MachineBasicBlock &MBB : MF) {
    for (const MachineBasicBlock *Succ : MBB.successors())
      MBPI.printEdgeProbability(OS << "  ", &MBB, Succ);
  }
  return PreservedAnalyses::all();
}

char MachineBranchProbabilityInfoWrapperPass::ID = 0;

MachineBranchProbabilityInfoWrapperPass::
    MachineBranchProbabilityInfoWrapperPass()
````
- **L41 EN**: Closes the current scope.
  **L41 CN**: 关闭当前作用域。
- **L42 EN**: Separates nearby statements for readability.
  **L42 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L43 EN**: Continues logic with `PreservedAnalyses`.
  **L43 CN**: 继续处理逻辑：`PreservedAnalyses`。
- **L44 EN**: Provides part of the signature for `run`.
  **L44 CN**: 给出 `run` 的一部分签名。
- **L45 EN**: Starts block `MachineFunctionAnalysisManager &MFAM)`.
  **L45 CN**: 开始代码块 `MachineFunctionAnalysisManager &MFAM)`。
- **L46 EN**: Continues logic with `OS << "Printing analysis 'Machine Branch Probability Analysis' for machi…`.
  **L46 CN**: 继续处理逻辑：`OS << "Printing analysis 'Machine Branch Probability Analysis' for machi…`。
- **L47 EN**: Continues logic with `"function '"`.
  **L47 CN**: 继续处理逻辑：`"function '"`。
- **L48 EN**: Executes statement `<< MF.getName() << "':\n";`.
  **L48 CN**: 执行语句 `<< MF.getName() << "':\n";`。
- **L49 EN**: Assigns or initializes `auto &MBPI`.
  **L49 CN**: 对 `auto &MBPI` 进行赋值或初始化。
- **L50 EN**: Starts a loop over a sequence or range.
  **L50 CN**: 开始遍历序列或范围的循环。
- **L51 EN**: Starts a loop over a sequence or range.
  **L51 CN**: 开始遍历序列或范围的循环。
- **L52 EN**: Executes statement `MBPI.printEdgeProbability(OS << " ", &MBB, Succ);`.
  **L52 CN**: 执行语句 `MBPI.printEdgeProbability(OS << " ", &MBB, Succ);`。
- **L53 EN**: Closes the current scope.
  **L53 CN**: 关闭当前作用域。
- **L54 EN**: Returns `PreservedAnalyses::all()` to the caller.
  **L54 CN**: 向调用者返回 `PreservedAnalyses::all()`。
- **L55 EN**: Closes the current scope.
  **L55 CN**: 关闭当前作用域。
- **L56 EN**: Separates nearby statements for readability.
  **L56 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L57 EN**: Assigns or initializes `char MachineBranchProbabilityInfoWrapperPass::ID`.
  **L57 CN**: 对 `char MachineBranchProbabilityInfoWrapperPass::ID` 进行赋值或初始化。
- **L58 EN**: Separates nearby statements for readability.
  **L58 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L59 EN**: Continues logic with `MachineBranchProbabilityInfoWrapperPass::`.
  **L59 CN**: 继续处理逻辑：`MachineBranchProbabilityInfoWrapperPass::`。
- **L60 EN**: Continues logic with `MachineBranchProbabilityInfoWrapperPass()`.
  **L60 CN**: 继续处理逻辑：`MachineBranchProbabilityInfoWrapperPass()`。

### Lines 61-80

````cpp
    : ImmutablePass(ID) {}

void MachineBranchProbabilityInfoWrapperPass::anchor() {}

AnalysisKey MachineBranchProbabilityAnalysis::Key;

bool MachineBranchProbabilityInfo::invalidate(
    MachineFunction &, const PreservedAnalyses &PA,
    MachineFunctionAnalysisManager::Invalidator &) {
  auto PAC = PA.getChecker<MachineBranchProbabilityAnalysis>();
  return !PAC.preservedWhenStateless();
}

BranchProbability
MachineBranchProbabilityInfo::getEdgeProbability(const MachineBasicBlock *Src,
                                                 unsigned SuccIdx) const {
  return Src->getSuccProbability(Src->succ_begin() + SuccIdx);
}

BranchProbability MachineBranchProbabilityInfo::getEdgeProbability(
````
- **L61 EN**: Provides part of the signature for `ImmutablePass`.
  **L61 CN**: 给出 `ImmutablePass` 的一部分签名。
- **L62 EN**: Separates nearby statements for readability.
  **L62 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L63 EN**: Provides part of the signature for `anchor`.
  **L63 CN**: 给出 `anchor` 的一部分签名。
- **L64 EN**: Separates nearby statements for readability.
  **L64 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L65 EN**: Executes statement `AnalysisKey MachineBranchProbabilityAnalysis::Key;`.
  **L65 CN**: 执行语句 `AnalysisKey MachineBranchProbabilityAnalysis::Key;`。
- **L66 EN**: Separates nearby statements for readability.
  **L66 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L67 EN**: Provides part of the signature for `invalidate`.
  **L67 CN**: 给出 `invalidate` 的一部分签名。
- **L68 EN**: Continues logic with `MachineFunction &, const PreservedAnalyses &PA,`.
  **L68 CN**: 继续处理逻辑：`MachineFunction &, const PreservedAnalyses &PA,`。
- **L69 EN**: Starts block `MachineFunctionAnalysisManager::Invalidator &)`.
  **L69 CN**: 开始代码块 `MachineFunctionAnalysisManager::Invalidator &)`。
- **L70 EN**: Assigns or initializes `auto PAC`.
  **L70 CN**: 对 `auto PAC` 进行赋值或初始化。
- **L71 EN**: Returns `!PAC.preservedWhenStateless()` to the caller.
  **L71 CN**: 向调用者返回 `!PAC.preservedWhenStateless()`。
- **L72 EN**: Closes the current scope.
  **L72 CN**: 关闭当前作用域。
- **L73 EN**: Separates nearby statements for readability.
  **L73 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L74 EN**: Continues logic with `BranchProbability`.
  **L74 CN**: 继续处理逻辑：`BranchProbability`。
- **L75 EN**: Provides part of the signature for `getEdgeProbability`.
  **L75 CN**: 给出 `getEdgeProbability` 的一部分签名。
- **L76 EN**: Starts block `unsigned SuccIdx) const`.
  **L76 CN**: 开始代码块 `unsigned SuccIdx) const`。
- **L77 EN**: Returns `Src->getSuccProbability(Src->succ_begin() + SuccIdx)` to the caller.
  **L77 CN**: 向调用者返回 `Src->getSuccProbability(Src->succ_begin() + SuccIdx)`。
- **L78 EN**: Closes the current scope.
  **L78 CN**: 关闭当前作用域。
- **L79 EN**: Separates nearby statements for readability.
  **L79 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L80 EN**: Provides part of the signature for `getEdgeProbability`.
  **L80 CN**: 给出 `getEdgeProbability` 的一部分签名。

### Lines 81-100

````cpp
    const MachineBasicBlock *Src, const MachineBasicBlock *Dst) const {
  // This is a linear search. Try to use the const_succ_iterator version when
  // possible.
  return Src->getSuccProbability(find(Src->successors(), Dst));
}

bool MachineBranchProbabilityInfo::isEdgeHot(
    const MachineBasicBlock *Src, const MachineBasicBlock *Dst) const {
  BranchProbability HotProb(StaticLikelyProb, 100);
  return getEdgeProbability(Src, Dst) > HotProb;
}

raw_ostream &MachineBranchProbabilityInfo::printEdgeProbability(
    raw_ostream &OS, const MachineBasicBlock *Src,
    const MachineBasicBlock *Dst) const {

  const BranchProbability Prob = getEdgeProbability(Src, Dst);
  OS << "edge " << printMBBReference(*Src) << " -> " << printMBBReference(*Dst)
     << " probability is " << Prob
     << (isEdgeHot(Src, Dst) ? " [HOT edge]\n" : "\n");
````
- **L81 EN**: Starts block `const MachineBasicBlock *Src, const MachineBasicBlock *Dst) const`.
  **L81 CN**: 开始代码块 `const MachineBasicBlock *Src, const MachineBasicBlock *Dst) const`。
- **L82 EN**: Comment documents: `This is a linear search. Try to use the const_succ_iterator version when`.
  **L82 CN**: 注释说明：`This is a linear search. Try to use the const_succ_iterator version when`。
- **L83 EN**: Comment documents: `possible.`.
  **L83 CN**: 注释说明：`possible.`。
- **L84 EN**: Returns `Src->getSuccProbability(find(Src->successors(), Dst))` to the caller.
  **L84 CN**: 向调用者返回 `Src->getSuccProbability(find(Src->successors(), Dst))`。
- **L85 EN**: Closes the current scope.
  **L85 CN**: 关闭当前作用域。
- **L86 EN**: Separates nearby statements for readability.
  **L86 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L87 EN**: Provides part of the signature for `isEdgeHot`.
  **L87 CN**: 给出 `isEdgeHot` 的一部分签名。
- **L88 EN**: Starts block `const MachineBasicBlock *Src, const MachineBasicBlock *Dst) const`.
  **L88 CN**: 开始代码块 `const MachineBasicBlock *Src, const MachineBasicBlock *Dst) const`。
- **L89 EN**: Declares function or method `HotProb`.
  **L89 CN**: 声明函数或方法 `HotProb`。
- **L90 EN**: Returns `getEdgeProbability(Src, Dst) > HotProb` to the caller.
  **L90 CN**: 向调用者返回 `getEdgeProbability(Src, Dst) > HotProb`。
- **L91 EN**: Closes the current scope.
  **L91 CN**: 关闭当前作用域。
- **L92 EN**: Separates nearby statements for readability.
  **L92 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L93 EN**: Provides part of the signature for `printEdgeProbability`.
  **L93 CN**: 给出 `printEdgeProbability` 的一部分签名。
- **L94 EN**: Continues logic with `raw_ostream &OS, const MachineBasicBlock *Src,`.
  **L94 CN**: 继续处理逻辑：`raw_ostream &OS, const MachineBasicBlock *Src,`。
- **L95 EN**: Starts block `const MachineBasicBlock *Dst) const`.
  **L95 CN**: 开始代码块 `const MachineBasicBlock *Dst) const`。
- **L96 EN**: Separates nearby statements for readability.
  **L96 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L97 EN**: Assigns or initializes `const BranchProbability Prob`.
  **L97 CN**: 对 `const BranchProbability Prob` 进行赋值或初始化。
- **L98 EN**: Continues logic with `OS << "edge " << printMBBReference(*Src) << " -> " << printMBBReference(…`.
  **L98 CN**: 继续处理逻辑：`OS << "edge " << printMBBReference(*Src) << " -> " << printMBBReference(…`。
- **L99 EN**: Continues logic with `<< " probability is " << Prob`.
  **L99 CN**: 继续处理逻辑：`<< " probability is " << Prob`。
- **L100 EN**: Executes statement `<< (isEdgeHot(Src, Dst) ? " [HOT edge]\n" : "\n");`.
  **L100 CN**: 执行语句 `<< (isEdgeHot(Src, Dst) ? " [HOT edge]\n" : "\n");`。

### Lines 101-103

````cpp

  return OS;
}
````
- **L101 EN**: Separates nearby statements for readability.
  **L101 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L102 EN**: Returns `OS` to the caller.
  **L102 CN**: 向调用者返回 `OS`。
- **L103 EN**: Closes the current scope.
  **L103 CN**: 关闭当前作用域。

## Key Concepts / 关键概念
- **Machine function processing** / **MachineFunction 处理**
- **Machine basic block handling** / **MachineBasicBlock 处理**
- **Control-flow updates** / **控制流更新**

## Dependencies / 依赖关系
- **LLVM headers / LLVM 头文件**: `llvm/CodeGen/MachineBranchProbabilityInfo.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/InitializePasses.h`, `llvm/Support/CommandLine.h`, `llvm/Support/raw_ostream.h`
- **Primary dependency domains / 主要依赖域**: CodeGen internals, LLVM support utilities, and C++ runtime helpers. / CodeGen 内部组件、LLVM 支持工具以及 C++ 运行时辅助设施。
