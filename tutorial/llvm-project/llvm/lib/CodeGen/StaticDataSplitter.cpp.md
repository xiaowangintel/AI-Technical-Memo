# StaticDataSplitter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/CodeGen/StaticDataSplitter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file contains `See https://llvm.org/LICENSE.txt for license information` logic inside LLVM CodeGen.
- **Purpose (CN)**: 该文件在 LLVM CodeGen 中实现与“See https://llvm.org/LICENSE.txt for license information”相关的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===- StaticDataSplitter.cpp ---------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// The pass uses branch profile data to assign hotness based section qualifiers
// for the following types of static data:
// - Jump tables
// - Module-internal global variables
// - Constant pools
//
// For the original RFC of this pass please see
// https://discourse.llvm.org/t/rfc-profile-guided-static-data-partitioning/83744

#include "llvm/ADT/Statistic.h"
#include "llvm/Analysis/ProfileSummaryInfo.h"
#include "llvm/Analysis/StaticDataProfileInfo.h"
````
- **L1 EN**: Comment documents: `===- StaticDataSplitter.cpp --------------------------------------------…`.
  **L1 CN**: 注释说明：`===- StaticDataSplitter.cpp --------------------------------------------…`。
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
- **L9 EN**: Comment documents: `The pass uses branch profile data to assign hotness based section qualif…`.
  **L9 CN**: 注释说明：`The pass uses branch profile data to assign hotness based section qualif…`。
- **L10 EN**: Comment documents: `for the following types of static data:`.
  **L10 CN**: 注释说明：`for the following types of static data:`。
- **L11 EN**: Comment documents: `- Jump tables`.
  **L11 CN**: 注释说明：`- Jump tables`。
- **L12 EN**: Comment documents: `- Module-internal global variables`.
  **L12 CN**: 注释说明：`- Module-internal global variables`。
- **L13 EN**: Comment documents: `- Constant pools`.
  **L13 CN**: 注释说明：`- Constant pools`。
- **L14 EN**: Continues the surrounding comment block.
  **L14 CN**: 延续周围的注释块。
- **L15 EN**: Comment documents: `For the original RFC of this pass please see`.
  **L15 CN**: 注释说明：`For the original RFC of this pass please see`。
- **L16 EN**: Comment documents: `https://discourse.llvm.org/t/rfc-profile-guided-static-data-partitioning…`.
  **L16 CN**: 注释说明：`https://discourse.llvm.org/t/rfc-profile-guided-static-data-partitioning…`。
- **L17 EN**: Separates nearby statements for readability.
  **L17 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L18 EN**: Includes LLVM header `llvm/ADT/Statistic.h` for Statistic support.
  **L18 CN**: 引入 LLVM 头文件 `llvm/ADT/Statistic.h`，用于 Statistic 相关支持。
- **L19 EN**: Includes LLVM header `llvm/Analysis/ProfileSummaryInfo.h` for ProfileSummaryInfo support.
  **L19 CN**: 引入 LLVM 头文件 `llvm/Analysis/ProfileSummaryInfo.h`，用于 ProfileSummaryInfo 相关支持。
- **L20 EN**: Includes LLVM header `llvm/Analysis/StaticDataProfileInfo.h` for StaticDataProfileInfo support.
  **L20 CN**: 引入 LLVM 头文件 `llvm/Analysis/StaticDataProfileInfo.h`，用于 StaticDataProfileInfo 相关支持。

### Lines 21-40

````cpp
#include "llvm/CodeGen/MBFIWrapper.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineBlockFrequencyInfo.h"
#include "llvm/CodeGen/MachineBranchProbabilityInfo.h"
#include "llvm/CodeGen/MachineConstantPool.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineJumpTableInfo.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/IR/GlobalVariable.h"
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
#include "llvm/Target/TargetLoweringObjectFile.h"

using namespace llvm;

#define DEBUG_TYPE "static-data-splitter"

STATISTIC(NumHotJumpTables, "Number of hot jump tables seen.");
STATISTIC(NumColdJumpTables, "Number of cold jump tables seen.");
````
- **L21 EN**: Includes LLVM header `llvm/CodeGen/MBFIWrapper.h` for MBFIWrapper support.
  **L21 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MBFIWrapper.h`，用于 MBFIWrapper 相关支持。
- **L22 EN**: Includes LLVM header `llvm/CodeGen/MachineBasicBlock.h` for MachineBasicBlock support.
  **L22 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineBasicBlock.h`，用于 MachineBasicBlock 相关支持。
- **L23 EN**: Includes LLVM header `llvm/CodeGen/MachineBlockFrequencyInfo.h` for MachineBlockFrequencyInfo support.
  **L23 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineBlockFrequencyInfo.h`，用于 MachineBlockFrequencyInfo 相关支持。
- **L24 EN**: Includes LLVM header `llvm/CodeGen/MachineBranchProbabilityInfo.h` for MachineBranchProbabilityInfo support.
  **L24 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineBranchProbabilityInfo.h`，用于 MachineBranchProbabilityInfo 相关支持。
- **L25 EN**: Includes LLVM header `llvm/CodeGen/MachineConstantPool.h` for MachineConstantPool support.
  **L25 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineConstantPool.h`，用于 MachineConstantPool 相关支持。
- **L26 EN**: Includes LLVM header `llvm/CodeGen/MachineFunction.h` for MachineFunction support.
  **L26 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineFunction.h`，用于 MachineFunction 相关支持。
- **L27 EN**: Includes LLVM header `llvm/CodeGen/MachineFunctionPass.h` for MachineFunctionPass support.
  **L27 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineFunctionPass.h`，用于 MachineFunctionPass 相关支持。
- **L28 EN**: Includes LLVM header `llvm/CodeGen/MachineJumpTableInfo.h` for MachineJumpTableInfo support.
  **L28 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineJumpTableInfo.h`，用于 MachineJumpTableInfo 相关支持。
- **L29 EN**: Includes LLVM header `llvm/CodeGen/Passes.h` for Passes support.
  **L29 CN**: 引入 LLVM 头文件 `llvm/CodeGen/Passes.h`，用于 Passes 相关支持。
- **L30 EN**: Includes LLVM header `llvm/IR/GlobalVariable.h` for GlobalVariable support.
  **L30 CN**: 引入 LLVM 头文件 `llvm/IR/GlobalVariable.h`，用于 GlobalVariable 相关支持。
- **L31 EN**: Includes LLVM header `llvm/InitializePasses.h` for InitializePasses support.
  **L31 CN**: 引入 LLVM 头文件 `llvm/InitializePasses.h`，用于 InitializePasses 相关支持。
- **L32 EN**: Includes LLVM header `llvm/Pass.h` for Pass support.
  **L32 CN**: 引入 LLVM 头文件 `llvm/Pass.h`，用于 Pass 相关支持。
- **L33 EN**: Includes LLVM header `llvm/Target/TargetLoweringObjectFile.h` for TargetLoweringObjectFile support.
  **L33 CN**: 引入 LLVM 头文件 `llvm/Target/TargetLoweringObjectFile.h`，用于 TargetLoweringObjectFile 相关支持。
- **L34 EN**: Separates nearby statements for readability.
  **L34 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L35 EN**: Imports namespace `llvm` into this translation unit.
  **L35 CN**: 将命名空间 `llvm` 引入当前编译单元。
- **L36 EN**: Separates nearby statements for readability.
  **L36 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L37 EN**: Defines the LLVM debug channel used by this file.
  **L37 CN**: 定义该文件使用的 LLVM 调试通道。
- **L38 EN**: Separates nearby statements for readability.
  **L38 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L39 EN**: Registers a pass statistic counter.
  **L39 CN**: 注册一个 pass 统计计数器。
- **L40 EN**: Registers a pass statistic counter.
  **L40 CN**: 注册一个 pass 统计计数器。

### Lines 41-60

````cpp
STATISTIC(NumUnknownJumpTables,
          "Number of jump tables with unknown hotness. They are from functions "
          "without profile information.");

class StaticDataSplitter : public MachineFunctionPass {
  const MachineBranchProbabilityInfo *MBPI = nullptr;
  const MachineBlockFrequencyInfo *MBFI = nullptr;
  const ProfileSummaryInfo *PSI = nullptr;
  StaticDataProfileInfo *SDPI = nullptr;

  // If the global value is a local linkage global variable, return it.
  // Otherwise, return nullptr.
  const GlobalVariable *getLocalLinkageGlobalVariable(const GlobalValue *GV);

  // Returns true if the global variable is in one of {.rodata, .bss, .data,
  // .data.rel.ro} sections.
  bool inStaticDataSection(const GlobalVariable &GV, const TargetMachine &TM);

  // Returns the constant if the operand refers to a global variable or constant
  // that gets lowered to static data sections. Otherwise, return nullptr.
````
- **L41 EN**: Registers a pass statistic counter.
  **L41 CN**: 注册一个 pass 统计计数器。
- **L42 EN**: Continues logic with `"Number of jump tables with unknown hotness. They are from functions "`.
  **L42 CN**: 继续处理逻辑：`"Number of jump tables with unknown hotness. They are from functions "`。
- **L43 EN**: Executes statement `"without profile information.");`.
  **L43 CN**: 执行语句 `"without profile information.");`。
- **L44 EN**: Separates nearby statements for readability.
  **L44 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L45 EN**: Starts the declaration of class `StaticDataSplitter`.
  **L45 CN**: 开始声明 class `StaticDataSplitter`。
- **L46 EN**: Assigns or initializes `const MachineBranchProbabilityInfo *MBPI`.
  **L46 CN**: 对 `const MachineBranchProbabilityInfo *MBPI` 进行赋值或初始化。
- **L47 EN**: Assigns or initializes `const MachineBlockFrequencyInfo *MBFI`.
  **L47 CN**: 对 `const MachineBlockFrequencyInfo *MBFI` 进行赋值或初始化。
- **L48 EN**: Assigns or initializes `const ProfileSummaryInfo *PSI`.
  **L48 CN**: 对 `const ProfileSummaryInfo *PSI` 进行赋值或初始化。
- **L49 EN**: Assigns or initializes `StaticDataProfileInfo *SDPI`.
  **L49 CN**: 对 `StaticDataProfileInfo *SDPI` 进行赋值或初始化。
- **L50 EN**: Separates nearby statements for readability.
  **L50 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L51 EN**: Comment documents: `If the global value is a local linkage global variable, return it.`.
  **L51 CN**: 注释说明：`If the global value is a local linkage global variable, return it.`。
- **L52 EN**: Comment documents: `Otherwise, return nullptr.`.
  **L52 CN**: 注释说明：`Otherwise, return nullptr.`。
- **L53 EN**: Executes statement `const GlobalVariable *getLocalLinkageGlobalVariable(const GlobalValue *G…`.
  **L53 CN**: 执行语句 `const GlobalVariable *getLocalLinkageGlobalVariable(const GlobalValue *G…`。
- **L54 EN**: Separates nearby statements for readability.
  **L54 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L55 EN**: Comment documents: `Returns true if the global variable is in one of {.rodata, .bss, .data,`.
  **L55 CN**: 注释说明：`Returns true if the global variable is in one of {.rodata, .bss, .data,`。
- **L56 EN**: Comment documents: `.data.rel.ro} sections.`.
  **L56 CN**: 注释说明：`.data.rel.ro} sections.`。
- **L57 EN**: Declares function or method `inStaticDataSection`.
  **L57 CN**: 声明函数或方法 `inStaticDataSection`。
- **L58 EN**: Separates nearby statements for readability.
  **L58 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L59 EN**: Comment documents: `Returns the constant if the operand refers to a global variable or const…`.
  **L59 CN**: 注释说明：`Returns the constant if the operand refers to a global variable or const…`。
- **L60 EN**: Comment documents: `that gets lowered to static data sections. Otherwise, return nullptr.`.
  **L60 CN**: 注释说明：`that gets lowered to static data sections. Otherwise, return nullptr.`。

### Lines 61-80

````cpp
  const Constant *getConstant(const MachineOperand &Op, const TargetMachine &TM,
                              const MachineConstantPool *MCP);

  // Use profiles to partition static data.
  bool partitionStaticDataWithProfiles(MachineFunction &MF);

  // Update LLVM statistics for a machine function with profiles.
  void updateStatsWithProfiles(const MachineFunction &MF);

  // Update LLVM statistics for a machine function without profiles.
  void updateStatsWithoutProfiles(const MachineFunction &MF);

  void annotateStaticDataWithoutProfiles(const MachineFunction &MF);

public:
  static char ID;

  StaticDataSplitter() : MachineFunctionPass(ID) {}

  StringRef getPassName() const override { return "Static Data Splitter"; }
````
- **L61 EN**: Continues logic with `const Constant *getConstant(const MachineOperand &Op, const TargetMachin…`.
  **L61 CN**: 继续处理逻辑：`const Constant *getConstant(const MachineOperand &Op, const TargetMachin…`。
- **L62 EN**: Executes statement `const MachineConstantPool *MCP);`.
  **L62 CN**: 执行语句 `const MachineConstantPool *MCP);`。
- **L63 EN**: Separates nearby statements for readability.
  **L63 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L64 EN**: Comment documents: `Use profiles to partition static data.`.
  **L64 CN**: 注释说明：`Use profiles to partition static data.`。
- **L65 EN**: Declares function or method `partitionStaticDataWithProfiles`.
  **L65 CN**: 声明函数或方法 `partitionStaticDataWithProfiles`。
- **L66 EN**: Separates nearby statements for readability.
  **L66 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L67 EN**: Comment documents: `Update LLVM statistics for a machine function with profiles.`.
  **L67 CN**: 注释说明：`Update LLVM statistics for a machine function with profiles.`。
- **L68 EN**: Declares function or method `updateStatsWithProfiles`.
  **L68 CN**: 声明函数或方法 `updateStatsWithProfiles`。
- **L69 EN**: Separates nearby statements for readability.
  **L69 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L70 EN**: Comment documents: `Update LLVM statistics for a machine function without profiles.`.
  **L70 CN**: 注释说明：`Update LLVM statistics for a machine function without profiles.`。
- **L71 EN**: Declares function or method `updateStatsWithoutProfiles`.
  **L71 CN**: 声明函数或方法 `updateStatsWithoutProfiles`。
- **L72 EN**: Separates nearby statements for readability.
  **L72 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L73 EN**: Declares function or method `annotateStaticDataWithoutProfiles`.
  **L73 CN**: 声明函数或方法 `annotateStaticDataWithoutProfiles`。
- **L74 EN**: Separates nearby statements for readability.
  **L74 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L75 EN**: Continues logic with `public:`.
  **L75 CN**: 继续处理逻辑：`public:`。
- **L76 EN**: Executes statement `static char ID;`.
  **L76 CN**: 执行语句 `static char ID;`。
- **L77 EN**: Separates nearby statements for readability.
  **L77 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L78 EN**: Continues logic with `StaticDataSplitter() : MachineFunctionPass(ID) {}`.
  **L78 CN**: 继续处理逻辑：`StaticDataSplitter() : MachineFunctionPass(ID) {}`。
- **L79 EN**: Separates nearby statements for readability.
  **L79 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L80 EN**: Provides part of the signature for `getPassName`.
  **L80 CN**: 给出 `getPassName` 的一部分签名。

### Lines 81-100

````cpp

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    MachineFunctionPass::getAnalysisUsage(AU);
    AU.addRequired<MachineBranchProbabilityInfoWrapperPass>();
    AU.addRequired<MachineBlockFrequencyInfoWrapperPass>();
    AU.addRequired<ProfileSummaryInfoWrapperPass>();
    AU.addRequired<StaticDataProfileInfoWrapperPass>();
    // This pass does not modify any required analysis results except
    // StaticDataProfileInfoWrapperPass, but StaticDataProfileInfoWrapperPass
    // is made an immutable pass that it won't be re-scheduled by pass manager
    // anyway. So mark setPreservesAll() here for faster compile time.
    AU.setPreservesAll();
  }

  bool runOnMachineFunction(MachineFunction &MF) override;
};

bool StaticDataSplitter::runOnMachineFunction(MachineFunction &MF) {
  MBPI = &getAnalysis<MachineBranchProbabilityInfoWrapperPass>().getMBPI();
  MBFI = &getAnalysis<MachineBlockFrequencyInfoWrapperPass>().getMBFI();
````
- **L81 EN**: Separates nearby statements for readability.
  **L81 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L82 EN**: Begins the definition of `getAnalysisUsage`.
  **L82 CN**: 开始定义 `getAnalysisUsage`。
- **L83 EN**: Declares function or method `getAnalysisUsage`.
  **L83 CN**: 声明函数或方法 `getAnalysisUsage`。
- **L84 EN**: Executes statement `AU.addRequired<MachineBranchProbabilityInfoWrapperPass>();`.
  **L84 CN**: 执行语句 `AU.addRequired<MachineBranchProbabilityInfoWrapperPass>();`。
- **L85 EN**: Executes statement `AU.addRequired<MachineBlockFrequencyInfoWrapperPass>();`.
  **L85 CN**: 执行语句 `AU.addRequired<MachineBlockFrequencyInfoWrapperPass>();`。
- **L86 EN**: Executes statement `AU.addRequired<ProfileSummaryInfoWrapperPass>();`.
  **L86 CN**: 执行语句 `AU.addRequired<ProfileSummaryInfoWrapperPass>();`。
- **L87 EN**: Executes statement `AU.addRequired<StaticDataProfileInfoWrapperPass>();`.
  **L87 CN**: 执行语句 `AU.addRequired<StaticDataProfileInfoWrapperPass>();`。
- **L88 EN**: Comment documents: `This pass does not modify any required analysis results except`.
  **L88 CN**: 注释说明：`This pass does not modify any required analysis results except`。
- **L89 EN**: Comment documents: `StaticDataProfileInfoWrapperPass, but StaticDataProfileInfoWrapperPass`.
  **L89 CN**: 注释说明：`StaticDataProfileInfoWrapperPass, but StaticDataProfileInfoWrapperPass`。
- **L90 EN**: Comment documents: `is made an immutable pass that it won't be re-scheduled by pass manager`.
  **L90 CN**: 注释说明：`is made an immutable pass that it won't be re-scheduled by pass manager`。
- **L91 EN**: Comment documents: `anyway. So mark setPreservesAll() here for faster compile time.`.
  **L91 CN**: 注释说明：`anyway. So mark setPreservesAll() here for faster compile time.`。
- **L92 EN**: Executes statement `AU.setPreservesAll();`.
  **L92 CN**: 执行语句 `AU.setPreservesAll();`。
- **L93 EN**: Closes the current scope.
  **L93 CN**: 关闭当前作用域。
- **L94 EN**: Separates nearby statements for readability.
  **L94 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L95 EN**: Declares function or method `runOnMachineFunction`.
  **L95 CN**: 声明函数或方法 `runOnMachineFunction`。
- **L96 EN**: Closes the current scope.
  **L96 CN**: 关闭当前作用域。
- **L97 EN**: Separates nearby statements for readability.
  **L97 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L98 EN**: Begins the definition of `runOnMachineFunction`.
  **L98 CN**: 开始定义 `runOnMachineFunction`。
- **L99 EN**: Assigns or initializes `MBPI`.
  **L99 CN**: 对 `MBPI` 进行赋值或初始化。
- **L100 EN**: Assigns or initializes `MBFI`.
  **L100 CN**: 对 `MBFI` 进行赋值或初始化。

### Lines 101-120

````cpp
  PSI = &getAnalysis<ProfileSummaryInfoWrapperPass>().getPSI();

  SDPI = &getAnalysis<StaticDataProfileInfoWrapperPass>()
              .getStaticDataProfileInfo();

  const bool ProfileAvailable = PSI && PSI->hasProfileSummary() && MBFI &&
                                MF.getFunction().hasProfileData();

  if (!ProfileAvailable) {
    annotateStaticDataWithoutProfiles(MF);
    updateStatsWithoutProfiles(MF);
    return false;
  }

  bool Changed = partitionStaticDataWithProfiles(MF);

  updateStatsWithProfiles(MF);
  return Changed;
}

````
- **L101 EN**: Assigns or initializes `PSI`.
  **L101 CN**: 对 `PSI` 进行赋值或初始化。
- **L102 EN**: Separates nearby statements for readability.
  **L102 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L103 EN**: Continues logic with `SDPI = &getAnalysis<StaticDataProfileInfoWrapperPass>()`.
  **L103 CN**: 继续处理逻辑：`SDPI = &getAnalysis<StaticDataProfileInfoWrapperPass>()`。
- **L104 EN**: Executes statement `.getStaticDataProfileInfo();`.
  **L104 CN**: 执行语句 `.getStaticDataProfileInfo();`。
- **L105 EN**: Separates nearby statements for readability.
  **L105 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L106 EN**: Continues logic with `const bool ProfileAvailable = PSI && PSI->hasProfileSummary() && MBFI &&`.
  **L106 CN**: 继续处理逻辑：`const bool ProfileAvailable = PSI && PSI->hasProfileSummary() && MBFI &&`。
- **L107 EN**: Executes statement `MF.getFunction().hasProfileData();`.
  **L107 CN**: 执行语句 `MF.getFunction().hasProfileData();`。
- **L108 EN**: Separates nearby statements for readability.
  **L108 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L109 EN**: Begins a conditional branch.
  **L109 CN**: 开始一个条件分支。
- **L110 EN**: Executes statement `annotateStaticDataWithoutProfiles(MF);`.
  **L110 CN**: 执行语句 `annotateStaticDataWithoutProfiles(MF);`。
- **L111 EN**: Executes statement `updateStatsWithoutProfiles(MF);`.
  **L111 CN**: 执行语句 `updateStatsWithoutProfiles(MF);`。
- **L112 EN**: Returns `false` to the caller.
  **L112 CN**: 向调用者返回 `false`。
- **L113 EN**: Closes the current scope.
  **L113 CN**: 关闭当前作用域。
- **L114 EN**: Separates nearby statements for readability.
  **L114 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L115 EN**: Assigns or initializes `bool Changed`.
  **L115 CN**: 对 `bool Changed` 进行赋值或初始化。
- **L116 EN**: Separates nearby statements for readability.
  **L116 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L117 EN**: Executes statement `updateStatsWithProfiles(MF);`.
  **L117 CN**: 执行语句 `updateStatsWithProfiles(MF);`。
- **L118 EN**: Returns `Changed` to the caller.
  **L118 CN**: 向调用者返回 `Changed`。
- **L119 EN**: Closes the current scope.
  **L119 CN**: 关闭当前作用域。
- **L120 EN**: Separates nearby statements for readability.
  **L120 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 121-140

````cpp
const Constant *
StaticDataSplitter::getConstant(const MachineOperand &Op,
                                const TargetMachine &TM,
                                const MachineConstantPool *MCP) {
  if (!Op.isGlobal() && !Op.isCPI())
    return nullptr;

  if (Op.isGlobal()) {
    // Find global variables with local linkage.
    const GlobalVariable *GV = getLocalLinkageGlobalVariable(Op.getGlobal());
    // Skip those not eligible for annotation or not in static data sections.
    if (!GV || !llvm::memprof::IsAnnotationOK(*GV) ||
        !inStaticDataSection(*GV, TM))
      return nullptr;
    return GV;
  }
  assert(Op.isCPI() && "Op must be constant pool index in this branch");
  int CPI = Op.getIndex();
  if (CPI == -1)
    return nullptr;
````
- **L121 EN**: Continues logic with `const Constant *`.
  **L121 CN**: 继续处理逻辑：`const Constant *`。
- **L122 EN**: Provides part of the signature for `getConstant`.
  **L122 CN**: 给出 `getConstant` 的一部分签名。
- **L123 EN**: Continues logic with `const TargetMachine &TM,`.
  **L123 CN**: 继续处理逻辑：`const TargetMachine &TM,`。
- **L124 EN**: Starts block `const MachineConstantPool *MCP)`.
  **L124 CN**: 开始代码块 `const MachineConstantPool *MCP)`。
- **L125 EN**: Begins a conditional branch.
  **L125 CN**: 开始一个条件分支。
- **L126 EN**: Returns `nullptr` to the caller.
  **L126 CN**: 向调用者返回 `nullptr`。
- **L127 EN**: Separates nearby statements for readability.
  **L127 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L128 EN**: Begins a conditional branch.
  **L128 CN**: 开始一个条件分支。
- **L129 EN**: Comment documents: `Find global variables with local linkage.`.
  **L129 CN**: 注释说明：`Find global variables with local linkage.`。
- **L130 EN**: Assigns or initializes `const GlobalVariable *GV`.
  **L130 CN**: 对 `const GlobalVariable *GV` 进行赋值或初始化。
- **L131 EN**: Comment documents: `Skip those not eligible for annotation or not in static data sections.`.
  **L131 CN**: 注释说明：`Skip those not eligible for annotation or not in static data sections.`。
- **L132 EN**: Begins a conditional branch.
  **L132 CN**: 开始一个条件分支。
- **L133 EN**: Continues logic with `!inStaticDataSection(*GV, TM))`.
  **L133 CN**: 继续处理逻辑：`!inStaticDataSection(*GV, TM))`。
- **L134 EN**: Returns `nullptr` to the caller.
  **L134 CN**: 向调用者返回 `nullptr`。
- **L135 EN**: Returns `GV` to the caller.
  **L135 CN**: 向调用者返回 `GV`。
- **L136 EN**: Closes the current scope.
  **L136 CN**: 关闭当前作用域。
- **L137 EN**: Checks an invariant in debug builds.
  **L137 CN**: 在调试构建中检查一个不变量。
- **L138 EN**: Assigns or initializes `int CPI`.
  **L138 CN**: 对 `int CPI` 进行赋值或初始化。
- **L139 EN**: Begins a conditional branch.
  **L139 CN**: 开始一个条件分支。
- **L140 EN**: Returns `nullptr` to the caller.
  **L140 CN**: 向调用者返回 `nullptr`。

### Lines 141-160

````cpp

  assert(MCP != nullptr && "Constant pool info is not available.");
  const MachineConstantPoolEntry &CPE = MCP->getConstants()[CPI];

  if (CPE.isMachineConstantPoolEntry())
    return nullptr;

  return CPE.Val.ConstVal;
}

bool StaticDataSplitter::partitionStaticDataWithProfiles(MachineFunction &MF) {
  // If any of the static data (jump tables, global variables, constant pools)
  // are captured by the analysis, set `Changed` to true. Note this pass won't
  // invalidate any analysis pass (see `getAnalysisUsage` above), so the main
  // purpose of tracking and conveying the change (to pass manager) is
  // informative as opposed to invalidating any analysis results. As an example
  // of where this information is useful, `PMDataManager::dumpPassInfo` will
  // only dump pass info if a local change happens, otherwise a pass appears as
  // "skipped".
  bool Changed = false;
````
- **L141 EN**: Separates nearby statements for readability.
  **L141 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L142 EN**: Checks an invariant in debug builds.
  **L142 CN**: 在调试构建中检查一个不变量。
- **L143 EN**: Assigns or initializes `const MachineConstantPoolEntry &CPE`.
  **L143 CN**: 对 `const MachineConstantPoolEntry &CPE` 进行赋值或初始化。
- **L144 EN**: Separates nearby statements for readability.
  **L144 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L145 EN**: Begins a conditional branch.
  **L145 CN**: 开始一个条件分支。
- **L146 EN**: Returns `nullptr` to the caller.
  **L146 CN**: 向调用者返回 `nullptr`。
- **L147 EN**: Separates nearby statements for readability.
  **L147 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L148 EN**: Returns `CPE.Val.ConstVal` to the caller.
  **L148 CN**: 向调用者返回 `CPE.Val.ConstVal`。
- **L149 EN**: Closes the current scope.
  **L149 CN**: 关闭当前作用域。
- **L150 EN**: Separates nearby statements for readability.
  **L150 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L151 EN**: Begins the definition of `partitionStaticDataWithProfiles`.
  **L151 CN**: 开始定义 `partitionStaticDataWithProfiles`。
- **L152 EN**: Comment documents: `If any of the static data (jump tables, global variables, constant pools…`.
  **L152 CN**: 注释说明：`If any of the static data (jump tables, global variables, constant pools…`。
- **L153 EN**: Comment documents: `are captured by the analysis, set 'Changed' to true. Note this pass won'…`.
  **L153 CN**: 注释说明：`are captured by the analysis, set 'Changed' to true. Note this pass won'…`。
- **L154 EN**: Comment documents: `invalidate any analysis pass (see 'getAnalysisUsage' above), so the main`.
  **L154 CN**: 注释说明：`invalidate any analysis pass (see 'getAnalysisUsage' above), so the main`。
- **L155 EN**: Comment documents: `purpose of tracking and conveying the change (to pass manager) is`.
  **L155 CN**: 注释说明：`purpose of tracking and conveying the change (to pass manager) is`。
- **L156 EN**: Comment documents: `informative as opposed to invalidating any analysis results. As an examp…`.
  **L156 CN**: 注释说明：`informative as opposed to invalidating any analysis results. As an examp…`。
- **L157 EN**: Comment documents: `of where this information is useful, 'PMDataManager::dumpPassInfo' will`.
  **L157 CN**: 注释说明：`of where this information is useful, 'PMDataManager::dumpPassInfo' will`。
- **L158 EN**: Comment documents: `only dump pass info if a local change happens, otherwise a pass appears …`.
  **L158 CN**: 注释说明：`only dump pass info if a local change happens, otherwise a pass appears …`。
- **L159 EN**: Comment documents: `"skipped".`.
  **L159 CN**: 注释说明：`"skipped".`。
- **L160 EN**: Assigns or initializes `bool Changed`.
  **L160 CN**: 对 `bool Changed` 进行赋值或初始化。

### Lines 161-180

````cpp

  MachineJumpTableInfo *MJTI = MF.getJumpTableInfo();

  // Jump table could be used by either terminating instructions or
  // non-terminating ones, so we walk all instructions and use
  // `MachineOperand::isJTI()` to identify jump table operands.
  // Similarly, `MachineOperand::isCPI()` is used to identify constant pool
  // usages in the same loop.
  for (const auto &MBB : MF) {
    std::optional<uint64_t> Count = MBFI->getBlockProfileCount(&MBB);
    for (const MachineInstr &I : MBB) {
      for (const MachineOperand &Op : I.operands()) {
        if (!Op.isJTI() && !Op.isGlobal() && !Op.isCPI())
          continue;

        if (Op.isJTI()) {
          assert(MJTI != nullptr && "Jump table info is not available.");
          const int JTI = Op.getIndex();
          // This is not a source block of jump table.
          if (JTI == -1)
````
- **L161 EN**: Separates nearby statements for readability.
  **L161 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L162 EN**: Assigns or initializes `MachineJumpTableInfo *MJTI`.
  **L162 CN**: 对 `MachineJumpTableInfo *MJTI` 进行赋值或初始化。
- **L163 EN**: Separates nearby statements for readability.
  **L163 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L164 EN**: Comment documents: `Jump table could be used by either terminating instructions or`.
  **L164 CN**: 注释说明：`Jump table could be used by either terminating instructions or`。
- **L165 EN**: Comment documents: `non-terminating ones, so we walk all instructions and use`.
  **L165 CN**: 注释说明：`non-terminating ones, so we walk all instructions and use`。
- **L166 EN**: Comment documents: `'MachineOperand::isJTI()' to identify jump table operands.`.
  **L166 CN**: 注释说明：`'MachineOperand::isJTI()' to identify jump table operands.`。
- **L167 EN**: Comment documents: `Similarly, 'MachineOperand::isCPI()' is used to identify constant pool`.
  **L167 CN**: 注释说明：`Similarly, 'MachineOperand::isCPI()' is used to identify constant pool`。
- **L168 EN**: Comment documents: `usages in the same loop.`.
  **L168 CN**: 注释说明：`usages in the same loop.`。
- **L169 EN**: Starts a loop over a sequence or range.
  **L169 CN**: 开始遍历序列或范围的循环。
- **L170 EN**: Assigns or initializes `std::optional<uint64_t> Count`.
  **L170 CN**: 对 `std::optional<uint64_t> Count` 进行赋值或初始化。
- **L171 EN**: Starts a loop over a sequence or range.
  **L171 CN**: 开始遍历序列或范围的循环。
- **L172 EN**: Starts a loop over a sequence or range.
  **L172 CN**: 开始遍历序列或范围的循环。
- **L173 EN**: Begins a conditional branch.
  **L173 CN**: 开始一个条件分支。
- **L174 EN**: Skips to the next loop iteration.
  **L174 CN**: 跳到下一次循环迭代。
- **L175 EN**: Separates nearby statements for readability.
  **L175 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L176 EN**: Begins a conditional branch.
  **L176 CN**: 开始一个条件分支。
- **L177 EN**: Checks an invariant in debug builds.
  **L177 CN**: 在调试构建中检查一个不变量。
- **L178 EN**: Assigns or initializes `const int JTI`.
  **L178 CN**: 对 `const int JTI` 进行赋值或初始化。
- **L179 EN**: Comment documents: `This is not a source block of jump table.`.
  **L179 CN**: 注释说明：`This is not a source block of jump table.`。
- **L180 EN**: Begins a conditional branch.
  **L180 CN**: 开始一个条件分支。

### Lines 181-200

````cpp
            continue;

          auto Hotness = MachineFunctionDataHotness::Hot;

          // Hotness is based on source basic block hotness.
          // TODO: PSI APIs are about instruction hotness. Introduce API for
          // data access hotness.
          if (Count && PSI->isColdCount(*Count))
            Hotness = MachineFunctionDataHotness::Cold;

          Changed |= MJTI->updateJumpTableEntryHotness(JTI, Hotness);
        } else if (const Constant *C =
                       getConstant(Op, MF.getTarget(), MF.getConstantPool())) {
          SDPI->addConstantProfileCount(C, Count);
          Changed = true;
        }
      }
    }
  }
  return Changed;
````
- **L181 EN**: Skips to the next loop iteration.
  **L181 CN**: 跳到下一次循环迭代。
- **L182 EN**: Separates nearby statements for readability.
  **L182 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L183 EN**: Assigns or initializes `auto Hotness`.
  **L183 CN**: 对 `auto Hotness` 进行赋值或初始化。
- **L184 EN**: Separates nearby statements for readability.
  **L184 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L185 EN**: Comment documents: `Hotness is based on source basic block hotness.`.
  **L185 CN**: 注释说明：`Hotness is based on source basic block hotness.`。
- **L186 EN**: Comment documents: `TODO: PSI APIs are about instruction hotness. Introduce API for`.
  **L186 CN**: 注释说明：`TODO: PSI APIs are about instruction hotness. Introduce API for`。
- **L187 EN**: Comment documents: `data access hotness.`.
  **L187 CN**: 注释说明：`data access hotness.`。
- **L188 EN**: Begins a conditional branch.
  **L188 CN**: 开始一个条件分支。
- **L189 EN**: Assigns or initializes `Hotness`.
  **L189 CN**: 对 `Hotness` 进行赋值或初始化。
- **L190 EN**: Separates nearby statements for readability.
  **L190 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L191 EN**: Assigns or initializes `Changed |`.
  **L191 CN**: 对 `Changed |` 进行赋值或初始化。
- **L192 EN**: Continues logic with `} else if (const Constant *C =`.
  **L192 CN**: 继续处理逻辑：`} else if (const Constant *C =`。
- **L193 EN**: Starts block `getConstant(Op, MF.getTarget(), MF.getConstantPool()))`.
  **L193 CN**: 开始代码块 `getConstant(Op, MF.getTarget(), MF.getConstantPool()))`。
- **L194 EN**: Executes statement `SDPI->addConstantProfileCount(C, Count);`.
  **L194 CN**: 执行语句 `SDPI->addConstantProfileCount(C, Count);`。
- **L195 EN**: Assigns or initializes `Changed`.
  **L195 CN**: 对 `Changed` 进行赋值或初始化。
- **L196 EN**: Closes the current scope.
  **L196 CN**: 关闭当前作用域。
- **L197 EN**: Closes the current scope.
  **L197 CN**: 关闭当前作用域。
- **L198 EN**: Closes the current scope.
  **L198 CN**: 关闭当前作用域。
- **L199 EN**: Closes the current scope.
  **L199 CN**: 关闭当前作用域。
- **L200 EN**: Returns `Changed` to the caller.
  **L200 CN**: 向调用者返回 `Changed`。

### Lines 201-220

````cpp
}

const GlobalVariable *
StaticDataSplitter::getLocalLinkageGlobalVariable(const GlobalValue *GV) {
  // LLVM IR Verifier requires that a declaration must have valid declaration
  // linkage, and local linkages are not among the valid ones. So there is no
  // need to check GV is not a declaration here.
  return (GV && GV->hasLocalLinkage()) ? dyn_cast<GlobalVariable>(GV) : nullptr;
}

bool StaticDataSplitter::inStaticDataSection(const GlobalVariable &GV,
                                             const TargetMachine &TM) {

  SectionKind Kind = TargetLoweringObjectFile::getKindForGlobal(&GV, TM);
  return Kind.isData() || Kind.isReadOnly() || Kind.isReadOnlyWithRel() ||
         Kind.isBSS();
}

void StaticDataSplitter::updateStatsWithProfiles(const MachineFunction &MF) {
  if (!AreStatisticsEnabled())
````
- **L201 EN**: Closes the current scope.
  **L201 CN**: 关闭当前作用域。
- **L202 EN**: Separates nearby statements for readability.
  **L202 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L203 EN**: Continues logic with `const GlobalVariable *`.
  **L203 CN**: 继续处理逻辑：`const GlobalVariable *`。
- **L204 EN**: Begins the definition of `getLocalLinkageGlobalVariable`.
  **L204 CN**: 开始定义 `getLocalLinkageGlobalVariable`。
- **L205 EN**: Comment documents: `LLVM IR Verifier requires that a declaration must have valid declaration`.
  **L205 CN**: 注释说明：`LLVM IR Verifier requires that a declaration must have valid declaration`。
- **L206 EN**: Comment documents: `linkage, and local linkages are not among the valid ones. So there is no`.
  **L206 CN**: 注释说明：`linkage, and local linkages are not among the valid ones. So there is no`。
- **L207 EN**: Comment documents: `need to check GV is not a declaration here.`.
  **L207 CN**: 注释说明：`need to check GV is not a declaration here.`。
- **L208 EN**: Returns `(GV && GV->hasLocalLinkage()) ? dyn_cast<GlobalVariable>(GV) : nullptr` to the caller.
  **L208 CN**: 向调用者返回 `(GV && GV->hasLocalLinkage()) ? dyn_cast<GlobalVariable>(GV) : nullptr`。
- **L209 EN**: Closes the current scope.
  **L209 CN**: 关闭当前作用域。
- **L210 EN**: Separates nearby statements for readability.
  **L210 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L211 EN**: Provides part of the signature for `inStaticDataSection`.
  **L211 CN**: 给出 `inStaticDataSection` 的一部分签名。
- **L212 EN**: Starts block `const TargetMachine &TM)`.
  **L212 CN**: 开始代码块 `const TargetMachine &TM)`。
- **L213 EN**: Separates nearby statements for readability.
  **L213 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L214 EN**: Declares function or method `getKindForGlobal`.
  **L214 CN**: 声明函数或方法 `getKindForGlobal`。
- **L215 EN**: Returns `Kind.isData() || Kind.isReadOnly() || Kind.isReadOnlyWithRel() ||` to the caller.
  **L215 CN**: 向调用者返回 `Kind.isData() || Kind.isReadOnly() || Kind.isReadOnlyWithRel() ||`。
- **L216 EN**: Executes statement `Kind.isBSS();`.
  **L216 CN**: 执行语句 `Kind.isBSS();`。
- **L217 EN**: Closes the current scope.
  **L217 CN**: 关闭当前作用域。
- **L218 EN**: Separates nearby statements for readability.
  **L218 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L219 EN**: Begins the definition of `updateStatsWithProfiles`.
  **L219 CN**: 开始定义 `updateStatsWithProfiles`。
- **L220 EN**: Begins a conditional branch.
  **L220 CN**: 开始一个条件分支。

### Lines 221-240

````cpp
    return;

  if (const MachineJumpTableInfo *MJTI = MF.getJumpTableInfo()) {
    for (const auto &JumpTable : MJTI->getJumpTables()) {
      if (JumpTable.Hotness == MachineFunctionDataHotness::Hot) {
        ++NumHotJumpTables;
      } else {
        assert(JumpTable.Hotness == MachineFunctionDataHotness::Cold &&
               "A jump table is either hot or cold when profile information is "
               "available.");
        ++NumColdJumpTables;
      }
    }
  }
}

void StaticDataSplitter::annotateStaticDataWithoutProfiles(
    const MachineFunction &MF) {
  for (const auto &MBB : MF)
    for (const MachineInstr &I : MBB)
````
- **L221 EN**: Returns control to the caller.
  **L221 CN**: 将控制流返回给调用者。
- **L222 EN**: Separates nearby statements for readability.
  **L222 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L223 EN**: Begins a conditional branch.
  **L223 CN**: 开始一个条件分支。
- **L224 EN**: Starts a loop over a sequence or range.
  **L224 CN**: 开始遍历序列或范围的循环。
- **L225 EN**: Begins a conditional branch.
  **L225 CN**: 开始一个条件分支。
- **L226 EN**: Executes statement `++NumHotJumpTables;`.
  **L226 CN**: 执行语句 `++NumHotJumpTables;`。
- **L227 EN**: Starts block `} else`.
  **L227 CN**: 开始代码块 `} else`。
- **L228 EN**: Checks an invariant in debug builds.
  **L228 CN**: 在调试构建中检查一个不变量。
- **L229 EN**: Continues logic with `"A jump table is either hot or cold when profile information is "`.
  **L229 CN**: 继续处理逻辑：`"A jump table is either hot or cold when profile information is "`。
- **L230 EN**: Executes statement `"available.");`.
  **L230 CN**: 执行语句 `"available.");`。
- **L231 EN**: Executes statement `++NumColdJumpTables;`.
  **L231 CN**: 执行语句 `++NumColdJumpTables;`。
- **L232 EN**: Closes the current scope.
  **L232 CN**: 关闭当前作用域。
- **L233 EN**: Closes the current scope.
  **L233 CN**: 关闭当前作用域。
- **L234 EN**: Closes the current scope.
  **L234 CN**: 关闭当前作用域。
- **L235 EN**: Closes the current scope.
  **L235 CN**: 关闭当前作用域。
- **L236 EN**: Separates nearby statements for readability.
  **L236 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L237 EN**: Provides part of the signature for `annotateStaticDataWithoutProfiles`.
  **L237 CN**: 给出 `annotateStaticDataWithoutProfiles` 的一部分签名。
- **L238 EN**: Starts block `const MachineFunction &MF)`.
  **L238 CN**: 开始代码块 `const MachineFunction &MF)`。
- **L239 EN**: Starts a loop over a sequence or range.
  **L239 CN**: 开始遍历序列或范围的循环。
- **L240 EN**: Starts a loop over a sequence or range.
  **L240 CN**: 开始遍历序列或范围的循环。

### Lines 241-260

````cpp
      for (const MachineOperand &Op : I.operands())
        if (const Constant *C =
                getConstant(Op, MF.getTarget(), MF.getConstantPool()))
          SDPI->addConstantProfileCount(C, std::nullopt);
}

void StaticDataSplitter::updateStatsWithoutProfiles(const MachineFunction &MF) {
  if (!AreStatisticsEnabled())
    return;

  if (const MachineJumpTableInfo *MJTI = MF.getJumpTableInfo()) {
    NumUnknownJumpTables += MJTI->getJumpTables().size();
  }
}

char StaticDataSplitter::ID = 0;

INITIALIZE_PASS_BEGIN(StaticDataSplitter, DEBUG_TYPE, "Split static data",
                      false, false)
INITIALIZE_PASS_DEPENDENCY(MachineBranchProbabilityInfoWrapperPass)
````
- **L241 EN**: Starts a loop over a sequence or range.
  **L241 CN**: 开始遍历序列或范围的循环。
- **L242 EN**: Begins a conditional branch.
  **L242 CN**: 开始一个条件分支。
- **L243 EN**: Continues logic with `getConstant(Op, MF.getTarget(), MF.getConstantPool()))`.
  **L243 CN**: 继续处理逻辑：`getConstant(Op, MF.getTarget(), MF.getConstantPool()))`。
- **L244 EN**: Executes statement `SDPI->addConstantProfileCount(C, std::nullopt);`.
  **L244 CN**: 执行语句 `SDPI->addConstantProfileCount(C, std::nullopt);`。
- **L245 EN**: Closes the current scope.
  **L245 CN**: 关闭当前作用域。
- **L246 EN**: Separates nearby statements for readability.
  **L246 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L247 EN**: Begins the definition of `updateStatsWithoutProfiles`.
  **L247 CN**: 开始定义 `updateStatsWithoutProfiles`。
- **L248 EN**: Begins a conditional branch.
  **L248 CN**: 开始一个条件分支。
- **L249 EN**: Returns control to the caller.
  **L249 CN**: 将控制流返回给调用者。
- **L250 EN**: Separates nearby statements for readability.
  **L250 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L251 EN**: Begins a conditional branch.
  **L251 CN**: 开始一个条件分支。
- **L252 EN**: Assigns or initializes `NumUnknownJumpTables +`.
  **L252 CN**: 对 `NumUnknownJumpTables +` 进行赋值或初始化。
- **L253 EN**: Closes the current scope.
  **L253 CN**: 关闭当前作用域。
- **L254 EN**: Closes the current scope.
  **L254 CN**: 关闭当前作用域。
- **L255 EN**: Separates nearby statements for readability.
  **L255 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L256 EN**: Assigns or initializes `char StaticDataSplitter::ID`.
  **L256 CN**: 对 `char StaticDataSplitter::ID` 进行赋值或初始化。
- **L257 EN**: Separates nearby statements for readability.
  **L257 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L258 EN**: Continues logic with `INITIALIZE_PASS_BEGIN(StaticDataSplitter, DEBUG_TYPE, "Split static data…`.
  **L258 CN**: 继续处理逻辑：`INITIALIZE_PASS_BEGIN(StaticDataSplitter, DEBUG_TYPE, "Split static data…`。
- **L259 EN**: Continues logic with `false, false)`.
  **L259 CN**: 继续处理逻辑：`false, false)`。
- **L260 EN**: Continues logic with `INITIALIZE_PASS_DEPENDENCY(MachineBranchProbabilityInfoWrapperPass)`.
  **L260 CN**: 继续处理逻辑：`INITIALIZE_PASS_DEPENDENCY(MachineBranchProbabilityInfoWrapperPass)`。

### Lines 261-269

````cpp
INITIALIZE_PASS_DEPENDENCY(MachineBlockFrequencyInfoWrapperPass)
INITIALIZE_PASS_DEPENDENCY(ProfileSummaryInfoWrapperPass)
INITIALIZE_PASS_DEPENDENCY(StaticDataProfileInfoWrapperPass)
INITIALIZE_PASS_END(StaticDataSplitter, DEBUG_TYPE, "Split static data", false,
                    false)

MachineFunctionPass *llvm::createStaticDataSplitterPass() {
  return new StaticDataSplitter();
}
````
- **L261 EN**: Continues logic with `INITIALIZE_PASS_DEPENDENCY(MachineBlockFrequencyInfoWrapperPass)`.
  **L261 CN**: 继续处理逻辑：`INITIALIZE_PASS_DEPENDENCY(MachineBlockFrequencyInfoWrapperPass)`。
- **L262 EN**: Continues logic with `INITIALIZE_PASS_DEPENDENCY(ProfileSummaryInfoWrapperPass)`.
  **L262 CN**: 继续处理逻辑：`INITIALIZE_PASS_DEPENDENCY(ProfileSummaryInfoWrapperPass)`。
- **L263 EN**: Continues logic with `INITIALIZE_PASS_DEPENDENCY(StaticDataProfileInfoWrapperPass)`.
  **L263 CN**: 继续处理逻辑：`INITIALIZE_PASS_DEPENDENCY(StaticDataProfileInfoWrapperPass)`。
- **L264 EN**: Continues logic with `INITIALIZE_PASS_END(StaticDataSplitter, DEBUG_TYPE, "Split static data",…`.
  **L264 CN**: 继续处理逻辑：`INITIALIZE_PASS_END(StaticDataSplitter, DEBUG_TYPE, "Split static data",…`。
- **L265 EN**: Continues logic with `false)`.
  **L265 CN**: 继续处理逻辑：`false)`。
- **L266 EN**: Separates nearby statements for readability.
  **L266 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L267 EN**: Begins the definition of `createStaticDataSplitterPass`.
  **L267 CN**: 开始定义 `createStaticDataSplitterPass`。
- **L268 EN**: Returns `new StaticDataSplitter()` to the caller.
  **L268 CN**: 向调用者返回 `new StaticDataSplitter()`。
- **L269 EN**: Closes the current scope.
  **L269 CN**: 关闭当前作用域。

## Key Concepts / 关键概念
- **Machine function processing** / **MachineFunction 处理**
- **Machine basic block handling** / **MachineBasicBlock 处理**
- **Machine instruction manipulation** / **机器指令操作**
- **Control-flow updates** / **控制流更新**
- **Debug information handling** / **调试信息处理**
- **Target-specific hooks** / **目标相关钩子**

## Dependencies / 依赖关系
- **LLVM headers / LLVM 头文件**: `llvm/ADT/Statistic.h`, `llvm/Analysis/ProfileSummaryInfo.h`, `llvm/Analysis/StaticDataProfileInfo.h`, `llvm/CodeGen/MBFIWrapper.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineBlockFrequencyInfo.h`, `llvm/CodeGen/MachineBranchProbabilityInfo.h`, `llvm/CodeGen/MachineConstantPool.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineJumpTableInfo.h`, `llvm/CodeGen/Passes.h`, `llvm/IR/GlobalVariable.h`, `llvm/InitializePasses.h`, `llvm/Pass.h`, `llvm/Target/TargetLoweringObjectFile.h`
- **Primary dependency domains / 主要依赖域**: CodeGen internals, LLVM support utilities, and C++ runtime helpers. / CodeGen 内部组件、LLVM 支持工具以及 C++ 运行时辅助设施。
