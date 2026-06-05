# MachineFunctionSplitter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/CodeGen/MachineFunctionSplitter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file contains `Split machine functions` logic inside LLVM CodeGen.
- **Purpose (CN)**: 该文件在 LLVM CodeGen 中实现与“Split machine functions”相关的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===-- MachineFunctionSplitter.cpp - Split machine functions //-----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// \file
// Uses profile information to split out cold blocks.
//
// This pass splits out cold machine basic blocks from the parent function. This
// implementation leverages the basic block section framework. Blocks marked
// cold by this pass are grouped together in a separate section prefixed with
// ".text.unlikely.*". The linker can then group these together as a cold
// section. The split part of the function is a contiguous region identified by
// the symbol "foo.cold". Grouping all cold blocks across functions together
// decreases fragmentation and improves icache and itlb utilization. Note that
// the overall changes to the binary size are negligible; only a small number of
// additional jump instructions may be introduced.
````
- **L1 EN**: Comment documents: `===-- MachineFunctionSplitter.cpp - Split machine functions //----------…`.
  **L1 CN**: 注释说明：`===-- MachineFunctionSplitter.cpp - Split machine functions //----------…`。
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
- **L9 EN**: Comment documents: `\file`.
  **L9 CN**: 注释说明：`\file`。
- **L10 EN**: Comment documents: `Uses profile information to split out cold blocks.`.
  **L10 CN**: 注释说明：`Uses profile information to split out cold blocks.`。
- **L11 EN**: Continues the surrounding comment block.
  **L11 CN**: 延续周围的注释块。
- **L12 EN**: Comment documents: `This pass splits out cold machine basic blocks from the parent function.…`.
  **L12 CN**: 注释说明：`This pass splits out cold machine basic blocks from the parent function.…`。
- **L13 EN**: Comment documents: `implementation leverages the basic block section framework. Blocks marke…`.
  **L13 CN**: 注释说明：`implementation leverages the basic block section framework. Blocks marke…`。
- **L14 EN**: Comment documents: `cold by this pass are grouped together in a separate section prefixed wi…`.
  **L14 CN**: 注释说明：`cold by this pass are grouped together in a separate section prefixed wi…`。
- **L15 EN**: Comment documents: `".text.unlikely.*". The linker can then group these together as a cold`.
  **L15 CN**: 注释说明：`".text.unlikely.*". The linker can then group these together as a cold`。
- **L16 EN**: Comment documents: `section. The split part of the function is a contiguous region identifie…`.
  **L16 CN**: 注释说明：`section. The split part of the function is a contiguous region identifie…`。
- **L17 EN**: Comment documents: `the symbol "foo.cold". Grouping all cold blocks across functions togethe…`.
  **L17 CN**: 注释说明：`the symbol "foo.cold". Grouping all cold blocks across functions togethe…`。
- **L18 EN**: Comment documents: `decreases fragmentation and improves icache and itlb utilization. Note t…`.
  **L18 CN**: 注释说明：`decreases fragmentation and improves icache and itlb utilization. Note t…`。
- **L19 EN**: Comment documents: `the overall changes to the binary size are negligible; only a small numb…`.
  **L19 CN**: 注释说明：`the overall changes to the binary size are negligible; only a small numb…`。
- **L20 EN**: Comment documents: `additional jump instructions may be introduced.`.
  **L20 CN**: 注释说明：`additional jump instructions may be introduced.`。

### Lines 21-40

````cpp
//
// For the original RFC of this pass please see
// https://groups.google.com/d/msg/llvm-dev/RUegaMg-iqc/wFAVxa6fCgAJ
//===----------------------------------------------------------------------===//

#include "llvm/ADT/SmallVector.h"
#include "llvm/Analysis/BranchProbabilityInfo.h"
#include "llvm/Analysis/EHUtils.h"
#include "llvm/Analysis/ProfileSummaryInfo.h"
#include "llvm/CodeGen/BasicBlockSectionUtils.h"
#include "llvm/CodeGen/BasicBlockSectionsProfileReader.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineBlockFrequencyInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineModuleInfo.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/IR/Function.h"
#include "llvm/InitializePasses.h"
````
- **L21 EN**: Continues the surrounding comment block.
  **L21 CN**: 延续周围的注释块。
- **L22 EN**: Comment documents: `For the original RFC of this pass please see`.
  **L22 CN**: 注释说明：`For the original RFC of this pass please see`。
- **L23 EN**: Comment documents: `https://groups.google.com/d/msg/llvm-dev/RUegaMg-iqc/wFAVxa6fCgAJ`.
  **L23 CN**: 注释说明：`https://groups.google.com/d/msg/llvm-dev/RUegaMg-iqc/wFAVxa6fCgAJ`。
- **L24 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L24 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L25 EN**: Separates nearby statements for readability.
  **L25 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L26 EN**: Includes LLVM header `llvm/ADT/SmallVector.h` for SmallVector support.
  **L26 CN**: 引入 LLVM 头文件 `llvm/ADT/SmallVector.h`，用于 SmallVector 相关支持。
- **L27 EN**: Includes LLVM header `llvm/Analysis/BranchProbabilityInfo.h` for BranchProbabilityInfo support.
  **L27 CN**: 引入 LLVM 头文件 `llvm/Analysis/BranchProbabilityInfo.h`，用于 BranchProbabilityInfo 相关支持。
- **L28 EN**: Includes LLVM header `llvm/Analysis/EHUtils.h` for EHUtils support.
  **L28 CN**: 引入 LLVM 头文件 `llvm/Analysis/EHUtils.h`，用于 EHUtils 相关支持。
- **L29 EN**: Includes LLVM header `llvm/Analysis/ProfileSummaryInfo.h` for ProfileSummaryInfo support.
  **L29 CN**: 引入 LLVM 头文件 `llvm/Analysis/ProfileSummaryInfo.h`，用于 ProfileSummaryInfo 相关支持。
- **L30 EN**: Includes LLVM header `llvm/CodeGen/BasicBlockSectionUtils.h` for BasicBlockSectionUtils support.
  **L30 CN**: 引入 LLVM 头文件 `llvm/CodeGen/BasicBlockSectionUtils.h`，用于 BasicBlockSectionUtils 相关支持。
- **L31 EN**: Includes LLVM header `llvm/CodeGen/BasicBlockSectionsProfileReader.h` for BasicBlockSectionsProfileReader support.
  **L31 CN**: 引入 LLVM 头文件 `llvm/CodeGen/BasicBlockSectionsProfileReader.h`，用于 BasicBlockSectionsProfileReader 相关支持。
- **L32 EN**: Includes LLVM header `llvm/CodeGen/MachineBasicBlock.h` for MachineBasicBlock support.
  **L32 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineBasicBlock.h`，用于 MachineBasicBlock 相关支持。
- **L33 EN**: Includes LLVM header `llvm/CodeGen/MachineBlockFrequencyInfo.h` for MachineBlockFrequencyInfo support.
  **L33 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineBlockFrequencyInfo.h`，用于 MachineBlockFrequencyInfo 相关支持。
- **L34 EN**: Includes LLVM header `llvm/CodeGen/MachineFunction.h` for MachineFunction support.
  **L34 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineFunction.h`，用于 MachineFunction 相关支持。
- **L35 EN**: Includes LLVM header `llvm/CodeGen/MachineFunctionPass.h` for MachineFunctionPass support.
  **L35 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineFunctionPass.h`，用于 MachineFunctionPass 相关支持。
- **L36 EN**: Includes LLVM header `llvm/CodeGen/MachineModuleInfo.h` for MachineModuleInfo support.
  **L36 CN**: 引入 LLVM 头文件 `llvm/CodeGen/MachineModuleInfo.h`，用于 MachineModuleInfo 相关支持。
- **L37 EN**: Includes LLVM header `llvm/CodeGen/Passes.h` for Passes support.
  **L37 CN**: 引入 LLVM 头文件 `llvm/CodeGen/Passes.h`，用于 Passes 相关支持。
- **L38 EN**: Includes LLVM header `llvm/CodeGen/TargetInstrInfo.h` for TargetInstrInfo support.
  **L38 CN**: 引入 LLVM 头文件 `llvm/CodeGen/TargetInstrInfo.h`，用于 TargetInstrInfo 相关支持。
- **L39 EN**: Includes LLVM header `llvm/IR/Function.h` for Function support.
  **L39 CN**: 引入 LLVM 头文件 `llvm/IR/Function.h`，用于 Function 相关支持。
- **L40 EN**: Includes LLVM header `llvm/InitializePasses.h` for InitializePasses support.
  **L40 CN**: 引入 LLVM 头文件 `llvm/InitializePasses.h`，用于 InitializePasses 相关支持。

### Lines 41-60

````cpp
#include "llvm/Support/CommandLine.h"
#include <optional>

using namespace llvm;

// FIXME: This cutoff value is CPU dependent and should be moved to
// TargetTransformInfo once we consider enabling this on other platforms.
// The value is expressed as a ProfileSummaryInfo integer percentile cutoff.
// Defaults to 999950, i.e. all blocks colder than 99.995 percentile are split.
// The default was empirically determined to be optimal when considering cutoff
// values between 99%-ile to 100%-ile with respect to iTLB and icache metrics on
// Intel CPUs.
static cl::opt<unsigned>
    PercentileCutoff("mfs-psi-cutoff",
                     cl::desc("Percentile profile summary cutoff used to "
                              "determine cold blocks. Unused if set to zero."),
                     cl::init(999950), cl::Hidden);

static cl::opt<unsigned> ColdCountThreshold(
    "mfs-count-threshold",
````
- **L41 EN**: Includes LLVM header `llvm/Support/CommandLine.h` for CommandLine support.
  **L41 CN**: 引入 LLVM 头文件 `llvm/Support/CommandLine.h`，用于 CommandLine 相关支持。
- **L42 EN**: Includes system header `optional`.
  **L42 CN**: 引入系统头文件 `optional`。
- **L43 EN**: Separates nearby statements for readability.
  **L43 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L44 EN**: Imports namespace `llvm` into this translation unit.
  **L44 CN**: 将命名空间 `llvm` 引入当前编译单元。
- **L45 EN**: Separates nearby statements for readability.
  **L45 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L46 EN**: Comment documents: `FIXME: This cutoff value is CPU dependent and should be moved to`.
  **L46 CN**: 注释说明：`FIXME: This cutoff value is CPU dependent and should be moved to`。
- **L47 EN**: Comment documents: `TargetTransformInfo once we consider enabling this on other platforms.`.
  **L47 CN**: 注释说明：`TargetTransformInfo once we consider enabling this on other platforms.`。
- **L48 EN**: Comment documents: `The value is expressed as a ProfileSummaryInfo integer percentile cutoff…`.
  **L48 CN**: 注释说明：`The value is expressed as a ProfileSummaryInfo integer percentile cutoff…`。
- **L49 EN**: Comment documents: `Defaults to 999950, i.e. all blocks colder than 99.995 percentile are sp…`.
  **L49 CN**: 注释说明：`Defaults to 999950, i.e. all blocks colder than 99.995 percentile are sp…`。
- **L50 EN**: Comment documents: `The default was empirically determined to be optimal when considering cu…`.
  **L50 CN**: 注释说明：`The default was empirically determined to be optimal when considering cu…`。
- **L51 EN**: Comment documents: `values between 99%-ile to 100%-ile with respect to iTLB and icache metri…`.
  **L51 CN**: 注释说明：`values between 99%-ile to 100%-ile with respect to iTLB and icache metri…`。
- **L52 EN**: Comment documents: `Intel CPUs.`.
  **L52 CN**: 注释说明：`Intel CPUs.`。
- **L53 EN**: Declares LLVM command-line option `command-line option`.
  **L53 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L54 EN**: Continues logic with `PercentileCutoff("mfs-psi-cutoff",`.
  **L54 CN**: 继续处理逻辑：`PercentileCutoff("mfs-psi-cutoff",`。
- **L55 EN**: Provides part of the signature for `desc`.
  **L55 CN**: 给出 `desc` 的一部分签名。
- **L56 EN**: Continues logic with `"determine cold blocks. Unused if set to zero."),`.
  **L56 CN**: 继续处理逻辑：`"determine cold blocks. Unused if set to zero."),`。
- **L57 EN**: Declares function or method `init`.
  **L57 CN**: 声明函数或方法 `init`。
- **L58 EN**: Separates nearby statements for readability.
  **L58 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L59 EN**: Declares LLVM command-line option `command-line option`.
  **L59 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L60 EN**: Continues logic with `"mfs-count-threshold",`.
  **L60 CN**: 继续处理逻辑：`"mfs-count-threshold",`。

### Lines 61-80

````cpp
    cl::desc(
        "Minimum number of times a block must be executed to be retained."),
    cl::init(1), cl::Hidden);

static cl::opt<bool> SplitAllEHCode(
    "mfs-split-ehcode",
    cl::desc("Splits all EH code and it's descendants by default."),
    cl::init(false), cl::Hidden);

namespace {

class MachineFunctionSplitter : public MachineFunctionPass {
public:
  static char ID;
  MachineFunctionSplitter() : MachineFunctionPass(ID) {}

  StringRef getPassName() const override {
    return "Machine Function Splitter Transformation";
  }

````
- **L61 EN**: Provides part of the signature for `desc`.
  **L61 CN**: 给出 `desc` 的一部分签名。
- **L62 EN**: Continues logic with `"Minimum number of times a block must be executed to be retained."),`.
  **L62 CN**: 继续处理逻辑：`"Minimum number of times a block must be executed to be retained."),`。
- **L63 EN**: Declares function or method `init`.
  **L63 CN**: 声明函数或方法 `init`。
- **L64 EN**: Separates nearby statements for readability.
  **L64 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L65 EN**: Declares LLVM command-line option `command-line option`.
  **L65 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L66 EN**: Continues logic with `"mfs-split-ehcode",`.
  **L66 CN**: 继续处理逻辑：`"mfs-split-ehcode",`。
- **L67 EN**: Provides part of the signature for `desc`.
  **L67 CN**: 给出 `desc` 的一部分签名。
- **L68 EN**: Declares function or method `init`.
  **L68 CN**: 声明函数或方法 `init`。
- **L69 EN**: Separates nearby statements for readability.
  **L69 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L70 EN**: Opens namespace ``.
  **L70 CN**: 打开命名空间 ``。
- **L71 EN**: Separates nearby statements for readability.
  **L71 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L72 EN**: Starts the declaration of class `MachineFunctionSplitter`.
  **L72 CN**: 开始声明 class `MachineFunctionSplitter`。
- **L73 EN**: Continues logic with `public:`.
  **L73 CN**: 继续处理逻辑：`public:`。
- **L74 EN**: Executes statement `static char ID;`.
  **L74 CN**: 执行语句 `static char ID;`。
- **L75 EN**: Continues logic with `MachineFunctionSplitter() : MachineFunctionPass(ID) {}`.
  **L75 CN**: 继续处理逻辑：`MachineFunctionSplitter() : MachineFunctionPass(ID) {}`。
- **L76 EN**: Separates nearby statements for readability.
  **L76 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L77 EN**: Begins the definition of `getPassName`.
  **L77 CN**: 开始定义 `getPassName`。
- **L78 EN**: Returns `"Machine Function Splitter Transformation"` to the caller.
  **L78 CN**: 向调用者返回 `"Machine Function Splitter Transformation"`。
- **L79 EN**: Closes the current scope.
  **L79 CN**: 关闭当前作用域。
- **L80 EN**: Separates nearby statements for readability.
  **L80 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 81-100

````cpp
  void getAnalysisUsage(AnalysisUsage &AU) const override;

  bool runOnMachineFunction(MachineFunction &F) override;
};
} // end anonymous namespace

/// setDescendantEHBlocksCold - This splits all EH pads and blocks reachable
/// only by EH pad as cold. This will help mark EH pads statically cold
/// instead of relying on profile data.
static void setDescendantEHBlocksCold(MachineFunction &MF) {
  DenseSet<MachineBasicBlock *> EHBlocks;
  computeEHOnlyBlocks(MF, EHBlocks);
  for (auto Block : EHBlocks) {
    Block->setSectionID(MBBSectionID::ColdSectionID);
  }
}

static void finishAdjustingBasicBlocksAndLandingPads(MachineFunction &MF) {
  auto Comparator = [](const MachineBasicBlock &X, const MachineBasicBlock &Y) {
    return X.getSectionID().Type < Y.getSectionID().Type;
````
- **L81 EN**: Declares function or method `getAnalysisUsage`.
  **L81 CN**: 声明函数或方法 `getAnalysisUsage`。
- **L82 EN**: Separates nearby statements for readability.
  **L82 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L83 EN**: Declares function or method `runOnMachineFunction`.
  **L83 CN**: 声明函数或方法 `runOnMachineFunction`。
- **L84 EN**: Closes the current scope.
  **L84 CN**: 关闭当前作用域。
- **L85 EN**: Continues logic with `} // end anonymous namespace`.
  **L85 CN**: 继续处理逻辑：`} // end anonymous namespace`。
- **L86 EN**: Separates nearby statements for readability.
  **L86 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L87 EN**: Comment documents: `setDescendantEHBlocksCold - This splits all EH pads and blocks reachable`.
  **L87 CN**: 注释说明：`setDescendantEHBlocksCold - This splits all EH pads and blocks reachable`。
- **L88 EN**: Comment documents: `only by EH pad as cold. This will help mark EH pads statically cold`.
  **L88 CN**: 注释说明：`only by EH pad as cold. This will help mark EH pads statically cold`。
- **L89 EN**: Comment documents: `instead of relying on profile data.`.
  **L89 CN**: 注释说明：`instead of relying on profile data.`。
- **L90 EN**: Begins the definition of `setDescendantEHBlocksCold`.
  **L90 CN**: 开始定义 `setDescendantEHBlocksCold`。
- **L91 EN**: Executes statement `DenseSet<MachineBasicBlock *> EHBlocks;`.
  **L91 CN**: 执行语句 `DenseSet<MachineBasicBlock *> EHBlocks;`。
- **L92 EN**: Executes statement `computeEHOnlyBlocks(MF, EHBlocks);`.
  **L92 CN**: 执行语句 `computeEHOnlyBlocks(MF, EHBlocks);`。
- **L93 EN**: Starts a loop over a sequence or range.
  **L93 CN**: 开始遍历序列或范围的循环。
- **L94 EN**: Executes statement `Block->setSectionID(MBBSectionID::ColdSectionID);`.
  **L94 CN**: 执行语句 `Block->setSectionID(MBBSectionID::ColdSectionID);`。
- **L95 EN**: Closes the current scope.
  **L95 CN**: 关闭当前作用域。
- **L96 EN**: Closes the current scope.
  **L96 CN**: 关闭当前作用域。
- **L97 EN**: Separates nearby statements for readability.
  **L97 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L98 EN**: Begins the definition of `finishAdjustingBasicBlocksAndLandingPads`.
  **L98 CN**: 开始定义 `finishAdjustingBasicBlocksAndLandingPads`。
- **L99 EN**: Starts block `auto Comparator = [](const MachineBasicBlock &X, const MachineBasicBlock…`.
  **L99 CN**: 开始代码块 `auto Comparator = [](const MachineBasicBlock &X, const MachineBasicBlock…`。
- **L100 EN**: Returns `X.getSectionID().Type < Y.getSectionID().Type` to the caller.
  **L100 CN**: 向调用者返回 `X.getSectionID().Type < Y.getSectionID().Type`。

### Lines 101-120

````cpp
  };
  llvm::sortBasicBlocksAndUpdateBranches(MF, Comparator);
  llvm::avoidZeroOffsetLandingPad(MF);
}

static bool isColdBlock(const MachineBasicBlock &MBB,
                        const MachineBlockFrequencyInfo *MBFI,
                        ProfileSummaryInfo *PSI) {
  std::optional<uint64_t> Count = MBFI->getBlockProfileCount(&MBB);
  // For instrumentation profiles and sample profiles, we use different ways
  // to judge whether a block is cold and should be split.
  if (PSI->hasInstrumentationProfile() || PSI->hasCSInstrumentationProfile()) {
    // If using instrument profile, which is deemed "accurate", no count means
    // cold.
    if (!Count)
      return true;
    if (PercentileCutoff > 0)
      return PSI->isColdCountNthPercentile(PercentileCutoff, *Count);
    // Fallthrough to end of function.
  } else if (PSI->hasSampleProfile()) {
````
- **L101 EN**: Closes the current scope.
  **L101 CN**: 关闭当前作用域。
- **L102 EN**: Declares function or method `sortBasicBlocksAndUpdateBranches`.
  **L102 CN**: 声明函数或方法 `sortBasicBlocksAndUpdateBranches`。
- **L103 EN**: Declares function or method `avoidZeroOffsetLandingPad`.
  **L103 CN**: 声明函数或方法 `avoidZeroOffsetLandingPad`。
- **L104 EN**: Closes the current scope.
  **L104 CN**: 关闭当前作用域。
- **L105 EN**: Separates nearby statements for readability.
  **L105 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L106 EN**: Provides part of the signature for `isColdBlock`.
  **L106 CN**: 给出 `isColdBlock` 的一部分签名。
- **L107 EN**: Continues logic with `const MachineBlockFrequencyInfo *MBFI,`.
  **L107 CN**: 继续处理逻辑：`const MachineBlockFrequencyInfo *MBFI,`。
- **L108 EN**: Starts block `ProfileSummaryInfo *PSI)`.
  **L108 CN**: 开始代码块 `ProfileSummaryInfo *PSI)`。
- **L109 EN**: Assigns or initializes `std::optional<uint64_t> Count`.
  **L109 CN**: 对 `std::optional<uint64_t> Count` 进行赋值或初始化。
- **L110 EN**: Comment documents: `For instrumentation profiles and sample profiles, we use different ways`.
  **L110 CN**: 注释说明：`For instrumentation profiles and sample profiles, we use different ways`。
- **L111 EN**: Comment documents: `to judge whether a block is cold and should be split.`.
  **L111 CN**: 注释说明：`to judge whether a block is cold and should be split.`。
- **L112 EN**: Begins a conditional branch.
  **L112 CN**: 开始一个条件分支。
- **L113 EN**: Comment documents: `If using instrument profile, which is deemed "accurate", no count means`.
  **L113 CN**: 注释说明：`If using instrument profile, which is deemed "accurate", no count means`。
- **L114 EN**: Comment documents: `cold.`.
  **L114 CN**: 注释说明：`cold.`。
- **L115 EN**: Begins a conditional branch.
  **L115 CN**: 开始一个条件分支。
- **L116 EN**: Returns `true` to the caller.
  **L116 CN**: 向调用者返回 `true`。
- **L117 EN**: Begins a conditional branch.
  **L117 CN**: 开始一个条件分支。
- **L118 EN**: Returns `PSI->isColdCountNthPercentile(PercentileCutoff, *Count)` to the caller.
  **L118 CN**: 向调用者返回 `PSI->isColdCountNthPercentile(PercentileCutoff, *Count)`。
- **L119 EN**: Comment documents: `Fallthrough to end of function.`.
  **L119 CN**: 注释说明：`Fallthrough to end of function.`。
- **L120 EN**: Starts block `} else if (PSI->hasSampleProfile())`.
  **L120 CN**: 开始代码块 `} else if (PSI->hasSampleProfile())`。

### Lines 121-140

````cpp
    // For sample profile, no count means "do not judege coldness".
    if (!Count)
      return false;
  }

  return (*Count < ColdCountThreshold);
}

bool MachineFunctionSplitter::runOnMachineFunction(MachineFunction &MF) {
  if (skipFunction(MF.getFunction()))
    return false;

  // Do not split functions when -basic-block-sections=all is specified.
  if (MF.getTarget().getBBSectionsType() == llvm::BasicBlockSection::All)
    return false;
  // We target functions with profile data. Static information in the form
  // of exception handling code may be split to cold if user passes the
  // mfs-split-ehcode flag.
  bool UseProfileData = MF.getFunction().hasProfileData();
  if (!UseProfileData && !SplitAllEHCode)
````
- **L121 EN**: Comment documents: `For sample profile, no count means "do not judege coldness".`.
  **L121 CN**: 注释说明：`For sample profile, no count means "do not judege coldness".`。
- **L122 EN**: Begins a conditional branch.
  **L122 CN**: 开始一个条件分支。
- **L123 EN**: Returns `false` to the caller.
  **L123 CN**: 向调用者返回 `false`。
- **L124 EN**: Closes the current scope.
  **L124 CN**: 关闭当前作用域。
- **L125 EN**: Separates nearby statements for readability.
  **L125 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L126 EN**: Returns `(*Count < ColdCountThreshold)` to the caller.
  **L126 CN**: 向调用者返回 `(*Count < ColdCountThreshold)`。
- **L127 EN**: Closes the current scope.
  **L127 CN**: 关闭当前作用域。
- **L128 EN**: Separates nearby statements for readability.
  **L128 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L129 EN**: Begins the definition of `runOnMachineFunction`.
  **L129 CN**: 开始定义 `runOnMachineFunction`。
- **L130 EN**: Begins a conditional branch.
  **L130 CN**: 开始一个条件分支。
- **L131 EN**: Returns `false` to the caller.
  **L131 CN**: 向调用者返回 `false`。
- **L132 EN**: Separates nearby statements for readability.
  **L132 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L133 EN**: Comment documents: `Do not split functions when -basic-block-sections=all is specified.`.
  **L133 CN**: 注释说明：`Do not split functions when -basic-block-sections=all is specified.`。
- **L134 EN**: Begins a conditional branch.
  **L134 CN**: 开始一个条件分支。
- **L135 EN**: Returns `false` to the caller.
  **L135 CN**: 向调用者返回 `false`。
- **L136 EN**: Comment documents: `We target functions with profile data. Static information in the form`.
  **L136 CN**: 注释说明：`We target functions with profile data. Static information in the form`。
- **L137 EN**: Comment documents: `of exception handling code may be split to cold if user passes the`.
  **L137 CN**: 注释说明：`of exception handling code may be split to cold if user passes the`。
- **L138 EN**: Comment documents: `mfs-split-ehcode flag.`.
  **L138 CN**: 注释说明：`mfs-split-ehcode flag.`。
- **L139 EN**: Assigns or initializes `bool UseProfileData`.
  **L139 CN**: 对 `bool UseProfileData` 进行赋值或初始化。
- **L140 EN**: Begins a conditional branch.
  **L140 CN**: 开始一个条件分支。

### Lines 141-160

````cpp
    return false;

  const TargetInstrInfo &TII = *MF.getSubtarget().getInstrInfo();
  if (!TII.isFunctionSafeToSplit(MF))
    return false;

  // Do not split functions with BasicBlockSections profiles as they will
  // be split by the BasicBlockSections pass.
  auto BBSectionsProfile =
      getAnalysisIfAvailable<BasicBlockSectionsProfileReaderWrapperPass>();
  if (BBSectionsProfile != nullptr &&
      BBSectionsProfile->getBBSPR().isFunctionHot(MF.getName()))
    return false;

  // Renumbering blocks here preserves the order of the blocks as
  // sortBasicBlocksAndUpdateBranches uses the numeric identifier to sort
  // blocks. Preserving the order of blocks is essential to retaining decisions
  // made by prior passes such as MachineBlockPlacement.
  MF.RenumberBlocks();
  MF.setBBSectionsType(BasicBlockSection::Preset);
````
- **L141 EN**: Returns `false` to the caller.
  **L141 CN**: 向调用者返回 `false`。
- **L142 EN**: Separates nearby statements for readability.
  **L142 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L143 EN**: Assigns or initializes `const TargetInstrInfo &TII`.
  **L143 CN**: 对 `const TargetInstrInfo &TII` 进行赋值或初始化。
- **L144 EN**: Begins a conditional branch.
  **L144 CN**: 开始一个条件分支。
- **L145 EN**: Returns `false` to the caller.
  **L145 CN**: 向调用者返回 `false`。
- **L146 EN**: Separates nearby statements for readability.
  **L146 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L147 EN**: Comment documents: `Do not split functions with BasicBlockSections profiles as they will`.
  **L147 CN**: 注释说明：`Do not split functions with BasicBlockSections profiles as they will`。
- **L148 EN**: Comment documents: `be split by the BasicBlockSections pass.`.
  **L148 CN**: 注释说明：`be split by the BasicBlockSections pass.`。
- **L149 EN**: Continues logic with `auto BBSectionsProfile =`.
  **L149 CN**: 继续处理逻辑：`auto BBSectionsProfile =`。
- **L150 EN**: Executes statement `getAnalysisIfAvailable<BasicBlockSectionsProfileReaderWrapperPass>();`.
  **L150 CN**: 执行语句 `getAnalysisIfAvailable<BasicBlockSectionsProfileReaderWrapperPass>();`。
- **L151 EN**: Begins a conditional branch.
  **L151 CN**: 开始一个条件分支。
- **L152 EN**: Continues logic with `BBSectionsProfile->getBBSPR().isFunctionHot(MF.getName()))`.
  **L152 CN**: 继续处理逻辑：`BBSectionsProfile->getBBSPR().isFunctionHot(MF.getName()))`。
- **L153 EN**: Returns `false` to the caller.
  **L153 CN**: 向调用者返回 `false`。
- **L154 EN**: Separates nearby statements for readability.
  **L154 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L155 EN**: Comment documents: `Renumbering blocks here preserves the order of the blocks as`.
  **L155 CN**: 注释说明：`Renumbering blocks here preserves the order of the blocks as`。
- **L156 EN**: Comment documents: `sortBasicBlocksAndUpdateBranches uses the numeric identifier to sort`.
  **L156 CN**: 注释说明：`sortBasicBlocksAndUpdateBranches uses the numeric identifier to sort`。
- **L157 EN**: Comment documents: `blocks. Preserving the order of blocks is essential to retaining decisio…`.
  **L157 CN**: 注释说明：`blocks. Preserving the order of blocks is essential to retaining decisio…`。
- **L158 EN**: Comment documents: `made by prior passes such as MachineBlockPlacement.`.
  **L158 CN**: 注释说明：`made by prior passes such as MachineBlockPlacement.`。
- **L159 EN**: Executes statement `MF.RenumberBlocks();`.
  **L159 CN**: 执行语句 `MF.RenumberBlocks();`。
- **L160 EN**: Executes statement `MF.setBBSectionsType(BasicBlockSection::Preset);`.
  **L160 CN**: 执行语句 `MF.setBBSectionsType(BasicBlockSection::Preset);`。

### Lines 161-180

````cpp

  MachineBlockFrequencyInfo *MBFI = nullptr;
  ProfileSummaryInfo *PSI = nullptr;
  if (UseProfileData) {
    MBFI = &getAnalysis<MachineBlockFrequencyInfoWrapperPass>().getMBFI();
    PSI = &getAnalysis<ProfileSummaryInfoWrapperPass>().getPSI();
    // If we don't have a good profile (sample profile is not deemed
    // as a "good profile") and the function is not hot, then early
    // return. (Because we can only trust hot functions when profile
    // quality is not good.)
    if (PSI->hasSampleProfile() && !PSI->isFunctionHotInCallGraph(&MF, *MBFI)) {
      // Split all EH code and it's descendant statically by default.
      if (SplitAllEHCode)
        setDescendantEHBlocksCold(MF);
      finishAdjustingBasicBlocksAndLandingPads(MF);
      return true;
    }
  }

  SmallVector<MachineBasicBlock *, 2> LandingPads;
````
- **L161 EN**: Separates nearby statements for readability.
  **L161 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L162 EN**: Assigns or initializes `MachineBlockFrequencyInfo *MBFI`.
  **L162 CN**: 对 `MachineBlockFrequencyInfo *MBFI` 进行赋值或初始化。
- **L163 EN**: Assigns or initializes `ProfileSummaryInfo *PSI`.
  **L163 CN**: 对 `ProfileSummaryInfo *PSI` 进行赋值或初始化。
- **L164 EN**: Begins a conditional branch.
  **L164 CN**: 开始一个条件分支。
- **L165 EN**: Assigns or initializes `MBFI`.
  **L165 CN**: 对 `MBFI` 进行赋值或初始化。
- **L166 EN**: Assigns or initializes `PSI`.
  **L166 CN**: 对 `PSI` 进行赋值或初始化。
- **L167 EN**: Comment documents: `If we don't have a good profile (sample profile is not deemed`.
  **L167 CN**: 注释说明：`If we don't have a good profile (sample profile is not deemed`。
- **L168 EN**: Comment documents: `as a "good profile") and the function is not hot, then early`.
  **L168 CN**: 注释说明：`as a "good profile") and the function is not hot, then early`。
- **L169 EN**: Comment documents: `return. (Because we can only trust hot functions when profile`.
  **L169 CN**: 注释说明：`return. (Because we can only trust hot functions when profile`。
- **L170 EN**: Comment documents: `quality is not good.)`.
  **L170 CN**: 注释说明：`quality is not good.)`。
- **L171 EN**: Begins a conditional branch.
  **L171 CN**: 开始一个条件分支。
- **L172 EN**: Comment documents: `Split all EH code and it's descendant statically by default.`.
  **L172 CN**: 注释说明：`Split all EH code and it's descendant statically by default.`。
- **L173 EN**: Begins a conditional branch.
  **L173 CN**: 开始一个条件分支。
- **L174 EN**: Executes statement `setDescendantEHBlocksCold(MF);`.
  **L174 CN**: 执行语句 `setDescendantEHBlocksCold(MF);`。
- **L175 EN**: Executes statement `finishAdjustingBasicBlocksAndLandingPads(MF);`.
  **L175 CN**: 执行语句 `finishAdjustingBasicBlocksAndLandingPads(MF);`。
- **L176 EN**: Returns `true` to the caller.
  **L176 CN**: 向调用者返回 `true`。
- **L177 EN**: Closes the current scope.
  **L177 CN**: 关闭当前作用域。
- **L178 EN**: Closes the current scope.
  **L178 CN**: 关闭当前作用域。
- **L179 EN**: Separates nearby statements for readability.
  **L179 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L180 EN**: Executes statement `SmallVector<MachineBasicBlock *, 2> LandingPads;`.
  **L180 CN**: 执行语句 `SmallVector<MachineBasicBlock *, 2> LandingPads;`。

### Lines 181-200

````cpp
  for (auto &MBB : MF) {
    if (MBB.isEntryBlock())
      continue;

    if (MBB.isEHPad())
      LandingPads.push_back(&MBB);
    else if (UseProfileData && isColdBlock(MBB, MBFI, PSI) &&
             TII.isMBBSafeToSplitToCold(MBB) && !SplitAllEHCode)
      MBB.setSectionID(MBBSectionID::ColdSectionID);
  }

  // Split all EH code and it's descendant statically by default.
  if (SplitAllEHCode)
    setDescendantEHBlocksCold(MF);
  // We only split out eh pads if all of them are cold.
  else {
    // Here we have UseProfileData == true.
    bool HasHotLandingPads = false;
    for (const MachineBasicBlock *LP : LandingPads) {
      if (!isColdBlock(*LP, MBFI, PSI) || !TII.isMBBSafeToSplitToCold(*LP))
````
- **L181 EN**: Starts a loop over a sequence or range.
  **L181 CN**: 开始遍历序列或范围的循环。
- **L182 EN**: Begins a conditional branch.
  **L182 CN**: 开始一个条件分支。
- **L183 EN**: Skips to the next loop iteration.
  **L183 CN**: 跳到下一次循环迭代。
- **L184 EN**: Separates nearby statements for readability.
  **L184 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L185 EN**: Begins a conditional branch.
  **L185 CN**: 开始一个条件分支。
- **L186 EN**: Executes statement `LandingPads.push_back(&MBB);`.
  **L186 CN**: 执行语句 `LandingPads.push_back(&MBB);`。
- **L187 EN**: Checks an alternate conditional path.
  **L187 CN**: 检查一个备用条件分支。
- **L188 EN**: Continues logic with `TII.isMBBSafeToSplitToCold(MBB) && !SplitAllEHCode)`.
  **L188 CN**: 继续处理逻辑：`TII.isMBBSafeToSplitToCold(MBB) && !SplitAllEHCode)`。
- **L189 EN**: Executes statement `MBB.setSectionID(MBBSectionID::ColdSectionID);`.
  **L189 CN**: 执行语句 `MBB.setSectionID(MBBSectionID::ColdSectionID);`。
- **L190 EN**: Closes the current scope.
  **L190 CN**: 关闭当前作用域。
- **L191 EN**: Separates nearby statements for readability.
  **L191 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L192 EN**: Comment documents: `Split all EH code and it's descendant statically by default.`.
  **L192 CN**: 注释说明：`Split all EH code and it's descendant statically by default.`。
- **L193 EN**: Begins a conditional branch.
  **L193 CN**: 开始一个条件分支。
- **L194 EN**: Executes statement `setDescendantEHBlocksCold(MF);`.
  **L194 CN**: 执行语句 `setDescendantEHBlocksCold(MF);`。
- **L195 EN**: Comment documents: `We only split out eh pads if all of them are cold.`.
  **L195 CN**: 注释说明：`We only split out eh pads if all of them are cold.`。
- **L196 EN**: Handles the fallback branch.
  **L196 CN**: 处理兜底分支。
- **L197 EN**: Comment documents: `Here we have UseProfileData == true.`.
  **L197 CN**: 注释说明：`Here we have UseProfileData == true.`。
- **L198 EN**: Assigns or initializes `bool HasHotLandingPads`.
  **L198 CN**: 对 `bool HasHotLandingPads` 进行赋值或初始化。
- **L199 EN**: Starts a loop over a sequence or range.
  **L199 CN**: 开始遍历序列或范围的循环。
- **L200 EN**: Begins a conditional branch.
  **L200 CN**: 开始一个条件分支。

### Lines 201-220

````cpp
        HasHotLandingPads = true;
    }
    if (!HasHotLandingPads) {
      for (MachineBasicBlock *LP : LandingPads)
        LP->setSectionID(MBBSectionID::ColdSectionID);
    }
  }

  finishAdjustingBasicBlocksAndLandingPads(MF);
  return true;
}

void MachineFunctionSplitter::getAnalysisUsage(AnalysisUsage &AU) const {
  AU.addRequired<MachineModuleInfoWrapperPass>();
  AU.addRequired<MachineBlockFrequencyInfoWrapperPass>();
  AU.addRequired<ProfileSummaryInfoWrapperPass>();
  AU.addUsedIfAvailable<BasicBlockSectionsProfileReaderWrapperPass>();
}

char MachineFunctionSplitter::ID = 0;
````
- **L201 EN**: Assigns or initializes `HasHotLandingPads`.
  **L201 CN**: 对 `HasHotLandingPads` 进行赋值或初始化。
- **L202 EN**: Closes the current scope.
  **L202 CN**: 关闭当前作用域。
- **L203 EN**: Begins a conditional branch.
  **L203 CN**: 开始一个条件分支。
- **L204 EN**: Starts a loop over a sequence or range.
  **L204 CN**: 开始遍历序列或范围的循环。
- **L205 EN**: Executes statement `LP->setSectionID(MBBSectionID::ColdSectionID);`.
  **L205 CN**: 执行语句 `LP->setSectionID(MBBSectionID::ColdSectionID);`。
- **L206 EN**: Closes the current scope.
  **L206 CN**: 关闭当前作用域。
- **L207 EN**: Closes the current scope.
  **L207 CN**: 关闭当前作用域。
- **L208 EN**: Separates nearby statements for readability.
  **L208 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L209 EN**: Executes statement `finishAdjustingBasicBlocksAndLandingPads(MF);`.
  **L209 CN**: 执行语句 `finishAdjustingBasicBlocksAndLandingPads(MF);`。
- **L210 EN**: Returns `true` to the caller.
  **L210 CN**: 向调用者返回 `true`。
- **L211 EN**: Closes the current scope.
  **L211 CN**: 关闭当前作用域。
- **L212 EN**: Separates nearby statements for readability.
  **L212 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L213 EN**: Begins the definition of `getAnalysisUsage`.
  **L213 CN**: 开始定义 `getAnalysisUsage`。
- **L214 EN**: Executes statement `AU.addRequired<MachineModuleInfoWrapperPass>();`.
  **L214 CN**: 执行语句 `AU.addRequired<MachineModuleInfoWrapperPass>();`。
- **L215 EN**: Executes statement `AU.addRequired<MachineBlockFrequencyInfoWrapperPass>();`.
  **L215 CN**: 执行语句 `AU.addRequired<MachineBlockFrequencyInfoWrapperPass>();`。
- **L216 EN**: Executes statement `AU.addRequired<ProfileSummaryInfoWrapperPass>();`.
  **L216 CN**: 执行语句 `AU.addRequired<ProfileSummaryInfoWrapperPass>();`。
- **L217 EN**: Executes statement `AU.addUsedIfAvailable<BasicBlockSectionsProfileReaderWrapperPass>();`.
  **L217 CN**: 执行语句 `AU.addUsedIfAvailable<BasicBlockSectionsProfileReaderWrapperPass>();`。
- **L218 EN**: Closes the current scope.
  **L218 CN**: 关闭当前作用域。
- **L219 EN**: Separates nearby statements for readability.
  **L219 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L220 EN**: Assigns or initializes `char MachineFunctionSplitter::ID`.
  **L220 CN**: 对 `char MachineFunctionSplitter::ID` 进行赋值或初始化。

### Lines 221-227

````cpp
INITIALIZE_PASS(MachineFunctionSplitter, "machine-function-splitter",
                "Split machine functions using profile information", false,
                false)

MachineFunctionPass *llvm::createMachineFunctionSplitterPass() {
  return new MachineFunctionSplitter();
}
````
- **L221 EN**: Continues logic with `INITIALIZE_PASS(MachineFunctionSplitter, "machine-function-splitter",`.
  **L221 CN**: 继续处理逻辑：`INITIALIZE_PASS(MachineFunctionSplitter, "machine-function-splitter",`。
- **L222 EN**: Continues logic with `"Split machine functions using profile information", false,`.
  **L222 CN**: 继续处理逻辑：`"Split machine functions using profile information", false,`。
- **L223 EN**: Continues logic with `false)`.
  **L223 CN**: 继续处理逻辑：`false)`。
- **L224 EN**: Separates nearby statements for readability.
  **L224 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L225 EN**: Begins the definition of `createMachineFunctionSplitterPass`.
  **L225 CN**: 开始定义 `createMachineFunctionSplitterPass`。
- **L226 EN**: Returns `new MachineFunctionSplitter()` to the caller.
  **L226 CN**: 向调用者返回 `new MachineFunctionSplitter()`。
- **L227 EN**: Closes the current scope.
  **L227 CN**: 关闭当前作用域。

## Key Concepts / 关键概念
- **Machine function processing** / **MachineFunction 处理**
- **Machine basic block handling** / **MachineBasicBlock 处理**
- **Control-flow updates** / **控制流更新**
- **Target-specific hooks** / **目标相关钩子**

## Dependencies / 依赖关系
- **LLVM headers / LLVM 头文件**: `llvm/ADT/SmallVector.h`, `llvm/Analysis/BranchProbabilityInfo.h`, `llvm/Analysis/EHUtils.h`, `llvm/Analysis/ProfileSummaryInfo.h`, `llvm/CodeGen/BasicBlockSectionUtils.h`, `llvm/CodeGen/BasicBlockSectionsProfileReader.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineBlockFrequencyInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineModuleInfo.h`, `llvm/CodeGen/Passes.h`, `llvm/CodeGen/TargetInstrInfo.h`, `llvm/IR/Function.h`, `llvm/InitializePasses.h`, `llvm/Support/CommandLine.h`
- **System headers / 系统头文件**: `optional`
- **Primary dependency domains / 主要依赖域**: CodeGen internals, LLVM support utilities, and C++ runtime helpers. / CodeGen 内部组件、LLVM 支持工具以及 C++ 运行时辅助设施。
