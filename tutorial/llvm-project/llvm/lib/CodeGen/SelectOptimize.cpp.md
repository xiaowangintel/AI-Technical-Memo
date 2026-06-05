# SelectOptimize.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/CodeGen/SelectOptimize.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file contains `Convert select to branches if profitable` logic inside LLVM CodeGen.
- **Purpose (CN)**: 该文件在 LLVM CodeGen 中实现与“Convert select to branches if profitable”相关的逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===--- SelectOptimize.cpp - Convert select to branches if profitable ---===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass converts selects to conditional jumps when profitable.
//
//===----------------------------------------------------------------------===//

#include "llvm/CodeGen/SelectOptimize.h"
#include "llvm/ADT/SetVector.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/Analysis/BlockFrequencyInfo.h"
#include "llvm/Analysis/BranchProbabilityInfo.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/Analysis/OptimizationRemarkEmitter.h"
````
- **L1 EN**: Comment documents: `===--- SelectOptimize.cpp - Convert select to branches if profitable ---…`.
  **L1 CN**: 注释说明：`===--- SelectOptimize.cpp - Convert select to branches if profitable ---…`。
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
- **L9 EN**: Comment documents: `This pass converts selects to conditional jumps when profitable.`.
  **L9 CN**: 注释说明：`This pass converts selects to conditional jumps when profitable.`。
- **L10 EN**: Continues the surrounding comment block.
  **L10 CN**: 延续周围的注释块。
- **L11 EN**: Comment documents: `===---------------------------------------------------------------------…`.
  **L11 CN**: 注释说明：`===---------------------------------------------------------------------…`。
- **L12 EN**: Separates nearby statements for readability.
  **L12 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L13 EN**: Includes LLVM header `llvm/CodeGen/SelectOptimize.h` for SelectOptimize support.
  **L13 CN**: 引入 LLVM 头文件 `llvm/CodeGen/SelectOptimize.h`，用于 SelectOptimize 相关支持。
- **L14 EN**: Includes LLVM header `llvm/ADT/SetVector.h` for SetVector support.
  **L14 CN**: 引入 LLVM 头文件 `llvm/ADT/SetVector.h`，用于 SetVector 相关支持。
- **L15 EN**: Includes LLVM header `llvm/ADT/SmallVector.h` for SmallVector support.
  **L15 CN**: 引入 LLVM 头文件 `llvm/ADT/SmallVector.h`，用于 SmallVector 相关支持。
- **L16 EN**: Includes LLVM header `llvm/ADT/Statistic.h` for Statistic support.
  **L16 CN**: 引入 LLVM 头文件 `llvm/ADT/Statistic.h`，用于 Statistic 相关支持。
- **L17 EN**: Includes LLVM header `llvm/Analysis/BlockFrequencyInfo.h` for BlockFrequencyInfo support.
  **L17 CN**: 引入 LLVM 头文件 `llvm/Analysis/BlockFrequencyInfo.h`，用于 BlockFrequencyInfo 相关支持。
- **L18 EN**: Includes LLVM header `llvm/Analysis/BranchProbabilityInfo.h` for BranchProbabilityInfo support.
  **L18 CN**: 引入 LLVM 头文件 `llvm/Analysis/BranchProbabilityInfo.h`，用于 BranchProbabilityInfo 相关支持。
- **L19 EN**: Includes LLVM header `llvm/Analysis/LoopInfo.h` for LoopInfo support.
  **L19 CN**: 引入 LLVM 头文件 `llvm/Analysis/LoopInfo.h`，用于 LoopInfo 相关支持。
- **L20 EN**: Includes LLVM header `llvm/Analysis/OptimizationRemarkEmitter.h` for OptimizationRemarkEmitter support.
  **L20 CN**: 引入 LLVM 头文件 `llvm/Analysis/OptimizationRemarkEmitter.h`，用于 OptimizationRemarkEmitter 相关支持。

### Lines 21-40

````cpp
#include "llvm/Analysis/ProfileSummaryInfo.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/CodeGen/TargetLowering.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/CodeGen/TargetSchedule.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/Instruction.h"
#include "llvm/IR/PatternMatch.h"
#include "llvm/IR/ProfDataUtils.h"
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
#include "llvm/Support/ScaledNumber.h"
#include "llvm/Target/TargetMachine.h"
#include "llvm/Transforms/Utils/SizeOpts.h"
#include <algorithm>
````
- **L21 EN**: Includes LLVM header `llvm/Analysis/ProfileSummaryInfo.h` for ProfileSummaryInfo support.
  **L21 CN**: 引入 LLVM 头文件 `llvm/Analysis/ProfileSummaryInfo.h`，用于 ProfileSummaryInfo 相关支持。
- **L22 EN**: Includes LLVM header `llvm/Analysis/TargetTransformInfo.h` for TargetTransformInfo support.
  **L22 CN**: 引入 LLVM 头文件 `llvm/Analysis/TargetTransformInfo.h`，用于 TargetTransformInfo 相关支持。
- **L23 EN**: Includes LLVM header `llvm/CodeGen/Passes.h` for Passes support.
  **L23 CN**: 引入 LLVM 头文件 `llvm/CodeGen/Passes.h`，用于 Passes 相关支持。
- **L24 EN**: Includes LLVM header `llvm/CodeGen/TargetLowering.h` for TargetLowering support.
  **L24 CN**: 引入 LLVM 头文件 `llvm/CodeGen/TargetLowering.h`，用于 TargetLowering 相关支持。
- **L25 EN**: Includes LLVM header `llvm/CodeGen/TargetPassConfig.h` for TargetPassConfig support.
  **L25 CN**: 引入 LLVM 头文件 `llvm/CodeGen/TargetPassConfig.h`，用于 TargetPassConfig 相关支持。
- **L26 EN**: Includes LLVM header `llvm/CodeGen/TargetSchedule.h` for TargetSchedule support.
  **L26 CN**: 引入 LLVM 头文件 `llvm/CodeGen/TargetSchedule.h`，用于 TargetSchedule 相关支持。
- **L27 EN**: Includes LLVM header `llvm/CodeGen/TargetSubtargetInfo.h` for TargetSubtargetInfo support.
  **L27 CN**: 引入 LLVM 头文件 `llvm/CodeGen/TargetSubtargetInfo.h`，用于 TargetSubtargetInfo 相关支持。
- **L28 EN**: Includes LLVM header `llvm/IR/BasicBlock.h` for BasicBlock support.
  **L28 CN**: 引入 LLVM 头文件 `llvm/IR/BasicBlock.h`，用于 BasicBlock 相关支持。
- **L29 EN**: Includes LLVM header `llvm/IR/Dominators.h` for Dominators support.
  **L29 CN**: 引入 LLVM 头文件 `llvm/IR/Dominators.h`，用于 Dominators 相关支持。
- **L30 EN**: Includes LLVM header `llvm/IR/Function.h` for Function support.
  **L30 CN**: 引入 LLVM 头文件 `llvm/IR/Function.h`，用于 Function 相关支持。
- **L31 EN**: Includes LLVM header `llvm/IR/IRBuilder.h` for IRBuilder support.
  **L31 CN**: 引入 LLVM 头文件 `llvm/IR/IRBuilder.h`，用于 IRBuilder 相关支持。
- **L32 EN**: Includes LLVM header `llvm/IR/Instruction.h` for Instruction support.
  **L32 CN**: 引入 LLVM 头文件 `llvm/IR/Instruction.h`，用于 Instruction 相关支持。
- **L33 EN**: Includes LLVM header `llvm/IR/PatternMatch.h` for PatternMatch support.
  **L33 CN**: 引入 LLVM 头文件 `llvm/IR/PatternMatch.h`，用于 PatternMatch 相关支持。
- **L34 EN**: Includes LLVM header `llvm/IR/ProfDataUtils.h` for ProfDataUtils support.
  **L34 CN**: 引入 LLVM 头文件 `llvm/IR/ProfDataUtils.h`，用于 ProfDataUtils 相关支持。
- **L35 EN**: Includes LLVM header `llvm/InitializePasses.h` for InitializePasses support.
  **L35 CN**: 引入 LLVM 头文件 `llvm/InitializePasses.h`，用于 InitializePasses 相关支持。
- **L36 EN**: Includes LLVM header `llvm/Pass.h` for Pass support.
  **L36 CN**: 引入 LLVM 头文件 `llvm/Pass.h`，用于 Pass 相关支持。
- **L37 EN**: Includes LLVM header `llvm/Support/ScaledNumber.h` for ScaledNumber support.
  **L37 CN**: 引入 LLVM 头文件 `llvm/Support/ScaledNumber.h`，用于 ScaledNumber 相关支持。
- **L38 EN**: Includes LLVM header `llvm/Target/TargetMachine.h` for TargetMachine support.
  **L38 CN**: 引入 LLVM 头文件 `llvm/Target/TargetMachine.h`，用于 TargetMachine 相关支持。
- **L39 EN**: Includes LLVM header `llvm/Transforms/Utils/SizeOpts.h` for SizeOpts support.
  **L39 CN**: 引入 LLVM 头文件 `llvm/Transforms/Utils/SizeOpts.h`，用于 SizeOpts 相关支持。
- **L40 EN**: Includes system header `algorithm`.
  **L40 CN**: 引入系统头文件 `algorithm`。

### Lines 41-60

````cpp
#include <queue>
#include <stack>

using namespace llvm;
using namespace llvm::PatternMatch;

#define DEBUG_TYPE "select-optimize"

STATISTIC(NumSelectOptAnalyzed,
          "Number of select groups considered for conversion to branch");
STATISTIC(NumSelectConvertedExpColdOperand,
          "Number of select groups converted due to expensive cold operand");
STATISTIC(NumSelectConvertedHighPred,
          "Number of select groups converted due to high-predictability");
STATISTIC(NumSelectUnPred,
          "Number of select groups not converted due to unpredictability");
STATISTIC(NumSelectColdBB,
          "Number of select groups not converted due to cold basic block");
STATISTIC(NumSelectConvertedLoop,
          "Number of select groups converted due to loop-level analysis");
````
- **L41 EN**: Includes system header `queue`.
  **L41 CN**: 引入系统头文件 `queue`。
- **L42 EN**: Includes system header `stack`.
  **L42 CN**: 引入系统头文件 `stack`。
- **L43 EN**: Separates nearby statements for readability.
  **L43 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L44 EN**: Imports namespace `llvm` into this translation unit.
  **L44 CN**: 将命名空间 `llvm` 引入当前编译单元。
- **L45 EN**: Imports namespace `llvm::PatternMatch` into this translation unit.
  **L45 CN**: 将命名空间 `llvm::PatternMatch` 引入当前编译单元。
- **L46 EN**: Separates nearby statements for readability.
  **L46 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L47 EN**: Defines the LLVM debug channel used by this file.
  **L47 CN**: 定义该文件使用的 LLVM 调试通道。
- **L48 EN**: Separates nearby statements for readability.
  **L48 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L49 EN**: Registers a pass statistic counter.
  **L49 CN**: 注册一个 pass 统计计数器。
- **L50 EN**: Executes statement `"Number of select groups considered for conversion to branch");`.
  **L50 CN**: 执行语句 `"Number of select groups considered for conversion to branch");`。
- **L51 EN**: Registers a pass statistic counter.
  **L51 CN**: 注册一个 pass 统计计数器。
- **L52 EN**: Executes statement `"Number of select groups converted due to expensive cold operand");`.
  **L52 CN**: 执行语句 `"Number of select groups converted due to expensive cold operand");`。
- **L53 EN**: Registers a pass statistic counter.
  **L53 CN**: 注册一个 pass 统计计数器。
- **L54 EN**: Executes statement `"Number of select groups converted due to high-predictability");`.
  **L54 CN**: 执行语句 `"Number of select groups converted due to high-predictability");`。
- **L55 EN**: Registers a pass statistic counter.
  **L55 CN**: 注册一个 pass 统计计数器。
- **L56 EN**: Executes statement `"Number of select groups not converted due to unpredictability");`.
  **L56 CN**: 执行语句 `"Number of select groups not converted due to unpredictability");`。
- **L57 EN**: Registers a pass statistic counter.
  **L57 CN**: 注册一个 pass 统计计数器。
- **L58 EN**: Executes statement `"Number of select groups not converted due to cold basic block");`.
  **L58 CN**: 执行语句 `"Number of select groups not converted due to cold basic block");`。
- **L59 EN**: Registers a pass statistic counter.
  **L59 CN**: 注册一个 pass 统计计数器。
- **L60 EN**: Executes statement `"Number of select groups converted due to loop-level analysis");`.
  **L60 CN**: 执行语句 `"Number of select groups converted due to loop-level analysis");`。

### Lines 61-80

````cpp
STATISTIC(NumSelectsConverted, "Number of selects converted");

static cl::opt<unsigned> ColdOperandThreshold(
    "cold-operand-threshold",
    cl::desc("Maximum frequency of path for an operand to be considered cold."),
    cl::init(20), cl::Hidden);

static cl::opt<unsigned> ColdOperandMaxCostMultiplier(
    "cold-operand-max-cost-multiplier",
    cl::desc("Maximum cost multiplier of TCC_expensive for the dependence "
             "slice of a cold operand to be considered inexpensive."),
    cl::init(1), cl::Hidden);

static cl::opt<unsigned>
    GainGradientThreshold("select-opti-loop-gradient-gain-threshold",
                          cl::desc("Gradient gain threshold (%)."),
                          cl::init(25), cl::Hidden);

static cl::opt<unsigned>
    GainCycleThreshold("select-opti-loop-cycle-gain-threshold",
````
- **L61 EN**: Registers a pass statistic counter.
  **L61 CN**: 注册一个 pass 统计计数器。
- **L62 EN**: Separates nearby statements for readability.
  **L62 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L63 EN**: Declares LLVM command-line option `command-line option`.
  **L63 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L64 EN**: Continues logic with `"cold-operand-threshold",`.
  **L64 CN**: 继续处理逻辑：`"cold-operand-threshold",`。
- **L65 EN**: Provides part of the signature for `desc`.
  **L65 CN**: 给出 `desc` 的一部分签名。
- **L66 EN**: Declares function or method `init`.
  **L66 CN**: 声明函数或方法 `init`。
- **L67 EN**: Separates nearby statements for readability.
  **L67 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L68 EN**: Declares LLVM command-line option `command-line option`.
  **L68 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L69 EN**: Continues logic with `"cold-operand-max-cost-multiplier",`.
  **L69 CN**: 继续处理逻辑：`"cold-operand-max-cost-multiplier",`。
- **L70 EN**: Provides part of the signature for `desc`.
  **L70 CN**: 给出 `desc` 的一部分签名。
- **L71 EN**: Continues logic with `"slice of a cold operand to be considered inexpensive."),`.
  **L71 CN**: 继续处理逻辑：`"slice of a cold operand to be considered inexpensive."),`。
- **L72 EN**: Declares function or method `init`.
  **L72 CN**: 声明函数或方法 `init`。
- **L73 EN**: Separates nearby statements for readability.
  **L73 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L74 EN**: Declares LLVM command-line option `command-line option`.
  **L74 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L75 EN**: Continues logic with `GainGradientThreshold("select-opti-loop-gradient-gain-threshold",`.
  **L75 CN**: 继续处理逻辑：`GainGradientThreshold("select-opti-loop-gradient-gain-threshold",`。
- **L76 EN**: Provides part of the signature for `desc`.
  **L76 CN**: 给出 `desc` 的一部分签名。
- **L77 EN**: Declares function or method `init`.
  **L77 CN**: 声明函数或方法 `init`。
- **L78 EN**: Separates nearby statements for readability.
  **L78 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L79 EN**: Declares LLVM command-line option `command-line option`.
  **L79 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L80 EN**: Continues logic with `GainCycleThreshold("select-opti-loop-cycle-gain-threshold",`.
  **L80 CN**: 继续处理逻辑：`GainCycleThreshold("select-opti-loop-cycle-gain-threshold",`。

### Lines 81-100

````cpp
                       cl::desc("Minimum gain per loop (in cycles) threshold."),
                       cl::init(4), cl::Hidden);

static cl::opt<unsigned> GainRelativeThreshold(
    "select-opti-loop-relative-gain-threshold",
    cl::desc(
        "Minimum relative gain per loop threshold (1/X). Defaults to 12.5%"),
    cl::init(8), cl::Hidden);

static cl::opt<unsigned> MispredictDefaultRate(
    "mispredict-default-rate", cl::Hidden, cl::init(25),
    cl::desc("Default mispredict rate (initialized to 25%)."));

static cl::opt<bool>
    DisableLoopLevelHeuristics("disable-loop-level-heuristics", cl::Hidden,
                               cl::init(false),
                               cl::desc("Disable loop-level heuristics."));

namespace {

````
- **L81 EN**: Provides part of the signature for `desc`.
  **L81 CN**: 给出 `desc` 的一部分签名。
- **L82 EN**: Declares function or method `init`.
  **L82 CN**: 声明函数或方法 `init`。
- **L83 EN**: Separates nearby statements for readability.
  **L83 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L84 EN**: Declares LLVM command-line option `command-line option`.
  **L84 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L85 EN**: Continues logic with `"select-opti-loop-relative-gain-threshold",`.
  **L85 CN**: 继续处理逻辑：`"select-opti-loop-relative-gain-threshold",`。
- **L86 EN**: Provides part of the signature for `desc`.
  **L86 CN**: 给出 `desc` 的一部分签名。
- **L87 EN**: Continues logic with `"Minimum relative gain per loop threshold (1/X). Defaults to 12.5%"),`.
  **L87 CN**: 继续处理逻辑：`"Minimum relative gain per loop threshold (1/X). Defaults to 12.5%"),`。
- **L88 EN**: Declares function or method `init`.
  **L88 CN**: 声明函数或方法 `init`。
- **L89 EN**: Separates nearby statements for readability.
  **L89 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L90 EN**: Declares LLVM command-line option `command-line option`.
  **L90 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L91 EN**: Provides part of the signature for `init`.
  **L91 CN**: 给出 `init` 的一部分签名。
- **L92 EN**: Declares function or method `desc`.
  **L92 CN**: 声明函数或方法 `desc`。
- **L93 EN**: Separates nearby statements for readability.
  **L93 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L94 EN**: Declares LLVM command-line option `command-line option`.
  **L94 CN**: 声明 LLVM 命令行选项 `command-line option`。
- **L95 EN**: Continues logic with `DisableLoopLevelHeuristics("disable-loop-level-heuristics", cl::Hidden,`.
  **L95 CN**: 继续处理逻辑：`DisableLoopLevelHeuristics("disable-loop-level-heuristics", cl::Hidden,`。
- **L96 EN**: Provides part of the signature for `init`.
  **L96 CN**: 给出 `init` 的一部分签名。
- **L97 EN**: Declares function or method `desc`.
  **L97 CN**: 声明函数或方法 `desc`。
- **L98 EN**: Separates nearby statements for readability.
  **L98 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L99 EN**: Opens namespace ``.
  **L99 CN**: 打开命名空间 ``。
- **L100 EN**: Separates nearby statements for readability.
  **L100 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 101-120

````cpp
class SelectOptimizeImpl {
  const TargetMachine *TM = nullptr;
  const TargetSubtargetInfo *TSI = nullptr;
  const TargetLowering *TLI = nullptr;
  const TargetTransformInfo *TTI = nullptr;
  const LoopInfo *LI = nullptr;
  BlockFrequencyInfo *BFI;
  ProfileSummaryInfo *PSI = nullptr;
  OptimizationRemarkEmitter *ORE = nullptr;
  TargetSchedModel TSchedModel;

public:
  SelectOptimizeImpl() = default;
  SelectOptimizeImpl(const TargetMachine *TM) : TM(TM){};
  PreservedAnalyses run(Function &F, FunctionAnalysisManager &FAM);
  bool runOnFunction(Function &F, Pass &P);

  using Scaled64 = ScaledNumber<uint64_t>;

  struct CostInfo {
````
- **L101 EN**: Starts the declaration of class `SelectOptimizeImpl`.
  **L101 CN**: 开始声明 class `SelectOptimizeImpl`。
- **L102 EN**: Assigns or initializes `const TargetMachine *TM`.
  **L102 CN**: 对 `const TargetMachine *TM` 进行赋值或初始化。
- **L103 EN**: Assigns or initializes `const TargetSubtargetInfo *TSI`.
  **L103 CN**: 对 `const TargetSubtargetInfo *TSI` 进行赋值或初始化。
- **L104 EN**: Assigns or initializes `const TargetLowering *TLI`.
  **L104 CN**: 对 `const TargetLowering *TLI` 进行赋值或初始化。
- **L105 EN**: Assigns or initializes `const TargetTransformInfo *TTI`.
  **L105 CN**: 对 `const TargetTransformInfo *TTI` 进行赋值或初始化。
- **L106 EN**: Assigns or initializes `const LoopInfo *LI`.
  **L106 CN**: 对 `const LoopInfo *LI` 进行赋值或初始化。
- **L107 EN**: Executes statement `BlockFrequencyInfo *BFI;`.
  **L107 CN**: 执行语句 `BlockFrequencyInfo *BFI;`。
- **L108 EN**: Assigns or initializes `ProfileSummaryInfo *PSI`.
  **L108 CN**: 对 `ProfileSummaryInfo *PSI` 进行赋值或初始化。
- **L109 EN**: Assigns or initializes `OptimizationRemarkEmitter *ORE`.
  **L109 CN**: 对 `OptimizationRemarkEmitter *ORE` 进行赋值或初始化。
- **L110 EN**: Executes statement `TargetSchedModel TSchedModel;`.
  **L110 CN**: 执行语句 `TargetSchedModel TSchedModel;`。
- **L111 EN**: Separates nearby statements for readability.
  **L111 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L112 EN**: Continues logic with `public:`.
  **L112 CN**: 继续处理逻辑：`public:`。
- **L113 EN**: Assigns or initializes `SelectOptimizeImpl()`.
  **L113 CN**: 对 `SelectOptimizeImpl()` 进行赋值或初始化。
- **L114 EN**: Executes statement `SelectOptimizeImpl(const TargetMachine *TM) : TM(TM){};`.
  **L114 CN**: 执行语句 `SelectOptimizeImpl(const TargetMachine *TM) : TM(TM){};`。
- **L115 EN**: Declares function or method `run`.
  **L115 CN**: 声明函数或方法 `run`。
- **L116 EN**: Declares function or method `runOnFunction`.
  **L116 CN**: 声明函数或方法 `runOnFunction`。
- **L117 EN**: Separates nearby statements for readability.
  **L117 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L118 EN**: Introduces alias or using-declaration `using Scaled64 = ScaledNumber<uint64_t>`.
  **L118 CN**: 引入别名或 using 声明 `using Scaled64 = ScaledNumber<uint64_t>`。
- **L119 EN**: Separates nearby statements for readability.
  **L119 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L120 EN**: Starts the declaration of struct `CostInfo`.
  **L120 CN**: 开始声明 struct `CostInfo`。

### Lines 121-140

````cpp
    /// Predicated cost (with selects as conditional moves).
    Scaled64 PredCost;
    /// Non-predicated cost (with selects converted to branches).
    Scaled64 NonPredCost;
  };

  /// SelectLike is an abstraction over SelectInst and other operations that can
  /// act like selects. For example Or(Zext(icmp), X) can be treated like
  /// select(icmp, X|1, X).
  class SelectLike {
    /// The select (/or) instruction.
    Instruction *I;
    /// Whether this select is inverted, "not(cond), FalseVal, TrueVal", as
    /// opposed to the original condition.
    bool Inverted = false;

    /// The index of the operand that depends on condition. Only for select-like
    /// instruction such as Or/Add.
    unsigned CondIdx;

````
- **L121 EN**: Comment documents: `Predicated cost (with selects as conditional moves).`.
  **L121 CN**: 注释说明：`Predicated cost (with selects as conditional moves).`。
- **L122 EN**: Executes statement `Scaled64 PredCost;`.
  **L122 CN**: 执行语句 `Scaled64 PredCost;`。
- **L123 EN**: Comment documents: `Non-predicated cost (with selects converted to branches).`.
  **L123 CN**: 注释说明：`Non-predicated cost (with selects converted to branches).`。
- **L124 EN**: Executes statement `Scaled64 NonPredCost;`.
  **L124 CN**: 执行语句 `Scaled64 NonPredCost;`。
- **L125 EN**: Closes the current scope.
  **L125 CN**: 关闭当前作用域。
- **L126 EN**: Separates nearby statements for readability.
  **L126 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L127 EN**: Comment documents: `SelectLike is an abstraction over SelectInst and other operations that c…`.
  **L127 CN**: 注释说明：`SelectLike is an abstraction over SelectInst and other operations that c…`。
- **L128 EN**: Comment documents: `act like selects. For example Or(Zext(icmp), X) can be treated like`.
  **L128 CN**: 注释说明：`act like selects. For example Or(Zext(icmp), X) can be treated like`。
- **L129 EN**: Comment documents: `select(icmp, X|1, X).`.
  **L129 CN**: 注释说明：`select(icmp, X|1, X).`。
- **L130 EN**: Starts the declaration of class `SelectLike`.
  **L130 CN**: 开始声明 class `SelectLike`。
- **L131 EN**: Comment documents: `The select (/or) instruction.`.
  **L131 CN**: 注释说明：`The select (/or) instruction.`。
- **L132 EN**: Executes statement `Instruction *I;`.
  **L132 CN**: 执行语句 `Instruction *I;`。
- **L133 EN**: Comment documents: `Whether this select is inverted, "not(cond), FalseVal, TrueVal", as`.
  **L133 CN**: 注释说明：`Whether this select is inverted, "not(cond), FalseVal, TrueVal", as`。
- **L134 EN**: Comment documents: `opposed to the original condition.`.
  **L134 CN**: 注释说明：`opposed to the original condition.`。
- **L135 EN**: Assigns or initializes `bool Inverted`.
  **L135 CN**: 对 `bool Inverted` 进行赋值或初始化。
- **L136 EN**: Separates nearby statements for readability.
  **L136 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L137 EN**: Comment documents: `The index of the operand that depends on condition. Only for select-like`.
  **L137 CN**: 注释说明：`The index of the operand that depends on condition. Only for select-like`。
- **L138 EN**: Comment documents: `instruction such as Or/Add.`.
  **L138 CN**: 注释说明：`instruction such as Or/Add.`。
- **L139 EN**: Executes statement `unsigned CondIdx;`.
  **L139 CN**: 执行语句 `unsigned CondIdx;`。
- **L140 EN**: Separates nearby statements for readability.
  **L140 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 141-160

````cpp
  public:
    SelectLike(Instruction *I, bool Inverted = false, unsigned CondIdx = 0)
        : I(I), Inverted(Inverted), CondIdx(CondIdx) {}

    Instruction *getI() { return I; }
    const Instruction *getI() const { return I; }

    Type *getType() const { return I->getType(); }

    unsigned getConditionOpIndex() { return CondIdx; };

    /// Return the true value for the SelectLike instruction. Note this may not
    /// exist for all SelectLike instructions. For example, for `or(zext(c), x)`
    /// the true value would be `or(x,1)`. As this value does not exist, nullptr
    /// is returned.
    Value *getTrueValue(bool HonorInverts = true) const {
      if (Inverted && HonorInverts)
        return getFalseValue(/*HonorInverts=*/false);
      if (auto *Sel = dyn_cast<SelectInst>(I))
        return Sel->getTrueValue();
````
- **L141 EN**: Continues logic with `public:`.
  **L141 CN**: 继续处理逻辑：`public:`。
- **L142 EN**: Continues logic with `SelectLike(Instruction *I, bool Inverted = false, unsigned CondIdx = 0)`.
  **L142 CN**: 继续处理逻辑：`SelectLike(Instruction *I, bool Inverted = false, unsigned CondIdx = 0)`。
- **L143 EN**: Provides part of the signature for `I`.
  **L143 CN**: 给出 `I` 的一部分签名。
- **L144 EN**: Separates nearby statements for readability.
  **L144 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L145 EN**: Continues logic with `Instruction *getI() { return I; }`.
  **L145 CN**: 继续处理逻辑：`Instruction *getI() { return I; }`。
- **L146 EN**: Continues logic with `const Instruction *getI() const { return I; }`.
  **L146 CN**: 继续处理逻辑：`const Instruction *getI() const { return I; }`。
- **L147 EN**: Separates nearby statements for readability.
  **L147 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L148 EN**: Continues logic with `Type *getType() const { return I->getType(); }`.
  **L148 CN**: 继续处理逻辑：`Type *getType() const { return I->getType(); }`。
- **L149 EN**: Separates nearby statements for readability.
  **L149 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L150 EN**: Declares function or method `getConditionOpIndex`.
  **L150 CN**: 声明函数或方法 `getConditionOpIndex`。
- **L151 EN**: Separates nearby statements for readability.
  **L151 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L152 EN**: Comment documents: `Return the true value for the SelectLike instruction. Note this may not`.
  **L152 CN**: 注释说明：`Return the true value for the SelectLike instruction. Note this may not`。
- **L153 EN**: Comment documents: `exist for all SelectLike instructions. For example, for 'or(zext(c), x)'`.
  **L153 CN**: 注释说明：`exist for all SelectLike instructions. For example, for 'or(zext(c), x)'`。
- **L154 EN**: Comment documents: `the true value would be 'or(x,1)'. As this value does not exist, nullptr`.
  **L154 CN**: 注释说明：`the true value would be 'or(x,1)'. As this value does not exist, nullptr`。
- **L155 EN**: Comment documents: `is returned.`.
  **L155 CN**: 注释说明：`is returned.`。
- **L156 EN**: Starts block `Value *getTrueValue(bool HonorInverts = true) const`.
  **L156 CN**: 开始代码块 `Value *getTrueValue(bool HonorInverts = true) const`。
- **L157 EN**: Begins a conditional branch.
  **L157 CN**: 开始一个条件分支。
- **L158 EN**: Returns `getFalseValue(/*HonorInverts=*/false)` to the caller.
  **L158 CN**: 向调用者返回 `getFalseValue(/*HonorInverts=*/false)`。
- **L159 EN**: Begins a conditional branch.
  **L159 CN**: 开始一个条件分支。
- **L160 EN**: Returns `Sel->getTrueValue()` to the caller.
  **L160 CN**: 向调用者返回 `Sel->getTrueValue()`。

### Lines 161-180

````cpp
      // Or(zext) case - The true value is Or(X), so return nullptr as the value
      // does not yet exist.
      if (isa<BinaryOperator>(I))
        return nullptr;

      llvm_unreachable("Unhandled case in getTrueValue");
    }

    /// Return the false value for the SelectLike instruction. For example the
    /// getFalseValue of a select or `x` in `or(zext(c), x)` (which is
    /// `select(c, x|1, x)`)
    Value *getFalseValue(bool HonorInverts = true) const {
      if (Inverted && HonorInverts)
        return getTrueValue(/*HonorInverts=*/false);
      if (auto *Sel = dyn_cast<SelectInst>(I))
        return Sel->getFalseValue();
      // We are on the branch where the condition is zero, which means BinOp
      // does not perform any computation, and we can simply return the operand
      // that is not related to the condition
      if (auto *BO = dyn_cast<BinaryOperator>(I))
````
- **L161 EN**: Comment documents: `Or(zext) case - The true value is Or(X), so return nullptr as the value`.
  **L161 CN**: 注释说明：`Or(zext) case - The true value is Or(X), so return nullptr as the value`。
- **L162 EN**: Comment documents: `does not yet exist.`.
  **L162 CN**: 注释说明：`does not yet exist.`。
- **L163 EN**: Begins a conditional branch.
  **L163 CN**: 开始一个条件分支。
- **L164 EN**: Returns `nullptr` to the caller.
  **L164 CN**: 向调用者返回 `nullptr`。
- **L165 EN**: Separates nearby statements for readability.
  **L165 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L166 EN**: Executes statement `llvm_unreachable("Unhandled case in getTrueValue");`.
  **L166 CN**: 执行语句 `llvm_unreachable("Unhandled case in getTrueValue");`。
- **L167 EN**: Closes the current scope.
  **L167 CN**: 关闭当前作用域。
- **L168 EN**: Separates nearby statements for readability.
  **L168 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L169 EN**: Comment documents: `Return the false value for the SelectLike instruction. For example the`.
  **L169 CN**: 注释说明：`Return the false value for the SelectLike instruction. For example the`。
- **L170 EN**: Comment documents: `getFalseValue of a select or 'x' in 'or(zext(c), x)' (which is`.
  **L170 CN**: 注释说明：`getFalseValue of a select or 'x' in 'or(zext(c), x)' (which is`。
- **L171 EN**: Comment documents: `'select(c, x|1, x)')`.
  **L171 CN**: 注释说明：`'select(c, x|1, x)')`。
- **L172 EN**: Starts block `Value *getFalseValue(bool HonorInverts = true) const`.
  **L172 CN**: 开始代码块 `Value *getFalseValue(bool HonorInverts = true) const`。
- **L173 EN**: Begins a conditional branch.
  **L173 CN**: 开始一个条件分支。
- **L174 EN**: Returns `getTrueValue(/*HonorInverts=*/false)` to the caller.
  **L174 CN**: 向调用者返回 `getTrueValue(/*HonorInverts=*/false)`。
- **L175 EN**: Begins a conditional branch.
  **L175 CN**: 开始一个条件分支。
- **L176 EN**: Returns `Sel->getFalseValue()` to the caller.
  **L176 CN**: 向调用者返回 `Sel->getFalseValue()`。
- **L177 EN**: Comment documents: `We are on the branch where the condition is zero, which means BinOp`.
  **L177 CN**: 注释说明：`We are on the branch where the condition is zero, which means BinOp`。
- **L178 EN**: Comment documents: `does not perform any computation, and we can simply return the operand`.
  **L178 CN**: 注释说明：`does not perform any computation, and we can simply return the operand`。
- **L179 EN**: Comment documents: `that is not related to the condition`.
  **L179 CN**: 注释说明：`that is not related to the condition`。
- **L180 EN**: Begins a conditional branch.
  **L180 CN**: 开始一个条件分支。

### Lines 181-200

````cpp
        return BO->getOperand(1 - CondIdx);

      llvm_unreachable("Unhandled case in getFalseValue");
    }

    /// Return the NonPredCost cost of the op on \p isTrue branch, given the
    /// costs in \p InstCostMap. This may need to be generated for select-like
    /// instructions.
    Scaled64 getOpCostOnBranch(
        bool IsTrue, const DenseMap<const Instruction *, CostInfo> &InstCostMap,
        const TargetTransformInfo *TTI) {
      auto *V = IsTrue ? getTrueValue() : getFalseValue();
      if (V) {
        if (auto *IV = dyn_cast<Instruction>(V)) {
          auto It = InstCostMap.find(IV);
          return It != InstCostMap.end() ? It->second.NonPredCost
                                         : Scaled64::getZero();
        }
        return Scaled64::getZero();
      }
````
- **L181 EN**: Returns `BO->getOperand(1 - CondIdx)` to the caller.
  **L181 CN**: 向调用者返回 `BO->getOperand(1 - CondIdx)`。
- **L182 EN**: Separates nearby statements for readability.
  **L182 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L183 EN**: Executes statement `llvm_unreachable("Unhandled case in getFalseValue");`.
  **L183 CN**: 执行语句 `llvm_unreachable("Unhandled case in getFalseValue");`。
- **L184 EN**: Closes the current scope.
  **L184 CN**: 关闭当前作用域。
- **L185 EN**: Separates nearby statements for readability.
  **L185 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L186 EN**: Comment documents: `Return the NonPredCost cost of the op on \p isTrue branch, given the`.
  **L186 CN**: 注释说明：`Return the NonPredCost cost of the op on \p isTrue branch, given the`。
- **L187 EN**: Comment documents: `costs in \p InstCostMap. This may need to be generated for select-like`.
  **L187 CN**: 注释说明：`costs in \p InstCostMap. This may need to be generated for select-like`。
- **L188 EN**: Comment documents: `instructions.`.
  **L188 CN**: 注释说明：`instructions.`。
- **L189 EN**: Provides part of the signature for `getOpCostOnBranch`.
  **L189 CN**: 给出 `getOpCostOnBranch` 的一部分签名。
- **L190 EN**: Continues logic with `bool IsTrue, const DenseMap<const Instruction *, CostInfo> &InstCostMap,`.
  **L190 CN**: 继续处理逻辑：`bool IsTrue, const DenseMap<const Instruction *, CostInfo> &InstCostMap,`。
- **L191 EN**: Starts block `const TargetTransformInfo *TTI)`.
  **L191 CN**: 开始代码块 `const TargetTransformInfo *TTI)`。
- **L192 EN**: Assigns or initializes `auto *V`.
  **L192 CN**: 对 `auto *V` 进行赋值或初始化。
- **L193 EN**: Begins a conditional branch.
  **L193 CN**: 开始一个条件分支。
- **L194 EN**: Begins a conditional branch.
  **L194 CN**: 开始一个条件分支。
- **L195 EN**: Assigns or initializes `auto It`.
  **L195 CN**: 对 `auto It` 进行赋值或初始化。
- **L196 EN**: Returns `It != InstCostMap.end() ? It->second.NonPredCost` to the caller.
  **L196 CN**: 向调用者返回 `It != InstCostMap.end() ? It->second.NonPredCost`。
- **L197 EN**: Declares function or method `getZero`.
  **L197 CN**: 声明函数或方法 `getZero`。
- **L198 EN**: Closes the current scope.
  **L198 CN**: 关闭当前作用域。
- **L199 EN**: Returns `Scaled64::getZero()` to the caller.
  **L199 CN**: 向调用者返回 `Scaled64::getZero()`。
- **L200 EN**: Closes the current scope.
  **L200 CN**: 关闭当前作用域。

### Lines 201-220

````cpp
      // If getTrue(False)Value() return nullptr, it means we are dealing with
      // select-like instructions on the branch where the actual computation is
      // happening. In that case the cost is equal to the cost of computation +
      // cost of non-dependant on condition operand
      InstructionCost Cost = TTI->getArithmeticInstrCost(
          getI()->getOpcode(), I->getType(), TargetTransformInfo::TCK_Latency,
          {TargetTransformInfo::OK_AnyValue, TargetTransformInfo::OP_None},
          {TTI::OK_UniformConstantValue, TTI::OP_PowerOf2});
      auto TotalCost = Scaled64::get(Cost.getValue());
      if (auto *OpI = dyn_cast<Instruction>(I->getOperand(1 - CondIdx))) {
        auto It = InstCostMap.find(OpI);
        if (It != InstCostMap.end())
          TotalCost += It->second.NonPredCost;
      }
      return TotalCost;
    }
  };

private:
  // Select groups consist of consecutive select-like instructions with the same
````
- **L201 EN**: Comment documents: `If getTrue(False)Value() return nullptr, it means we are dealing with`.
  **L201 CN**: 注释说明：`If getTrue(False)Value() return nullptr, it means we are dealing with`。
- **L202 EN**: Comment documents: `select-like instructions on the branch where the actual computation is`.
  **L202 CN**: 注释说明：`select-like instructions on the branch where the actual computation is`。
- **L203 EN**: Comment documents: `happening. In that case the cost is equal to the cost of computation +`.
  **L203 CN**: 注释说明：`happening. In that case the cost is equal to the cost of computation +`。
- **L204 EN**: Comment documents: `cost of non-dependant on condition operand`.
  **L204 CN**: 注释说明：`cost of non-dependant on condition operand`。
- **L205 EN**: Continues logic with `InstructionCost Cost = TTI->getArithmeticInstrCost(`.
  **L205 CN**: 继续处理逻辑：`InstructionCost Cost = TTI->getArithmeticInstrCost(`。
- **L206 EN**: Continues logic with `getI()->getOpcode(), I->getType(), TargetTransformInfo::TCK_Latency,`.
  **L206 CN**: 继续处理逻辑：`getI()->getOpcode(), I->getType(), TargetTransformInfo::TCK_Latency,`。
- **L207 EN**: Continues logic with `{TargetTransformInfo::OK_AnyValue, TargetTransformInfo::OP_None},`.
  **L207 CN**: 继续处理逻辑：`{TargetTransformInfo::OK_AnyValue, TargetTransformInfo::OP_None},`。
- **L208 EN**: Executes statement `{TTI::OK_UniformConstantValue, TTI::OP_PowerOf2});`.
  **L208 CN**: 执行语句 `{TTI::OK_UniformConstantValue, TTI::OP_PowerOf2});`。
- **L209 EN**: Declares function or method `get`.
  **L209 CN**: 声明函数或方法 `get`。
- **L210 EN**: Begins a conditional branch.
  **L210 CN**: 开始一个条件分支。
- **L211 EN**: Assigns or initializes `auto It`.
  **L211 CN**: 对 `auto It` 进行赋值或初始化。
- **L212 EN**: Begins a conditional branch.
  **L212 CN**: 开始一个条件分支。
- **L213 EN**: Assigns or initializes `TotalCost +`.
  **L213 CN**: 对 `TotalCost +` 进行赋值或初始化。
- **L214 EN**: Closes the current scope.
  **L214 CN**: 关闭当前作用域。
- **L215 EN**: Returns `TotalCost` to the caller.
  **L215 CN**: 向调用者返回 `TotalCost`。
- **L216 EN**: Closes the current scope.
  **L216 CN**: 关闭当前作用域。
- **L217 EN**: Closes the current scope.
  **L217 CN**: 关闭当前作用域。
- **L218 EN**: Separates nearby statements for readability.
  **L218 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L219 EN**: Continues logic with `private:`.
  **L219 CN**: 继续处理逻辑：`private:`。
- **L220 EN**: Comment documents: `Select groups consist of consecutive select-like instructions with the s…`.
  **L220 CN**: 注释说明：`Select groups consist of consecutive select-like instructions with the s…`。

### Lines 221-240

````cpp
  // condition. Between select-likes could be any number of auxiliary
  // instructions related to the condition like not, zext, ashr/lshr
  struct SelectGroup {
    Value *Condition;
    SmallVector<SelectLike, 2> Selects;
  };
  using SelectGroups = SmallVector<SelectGroup, 2>;

  // Converts select instructions of a function to conditional jumps when deemed
  // profitable. Returns true if at least one select was converted.
  bool optimizeSelects(Function &F);

  // Heuristics for determining which select instructions can be profitably
  // conveted to branches. Separate heuristics for selects in inner-most loops
  // and the rest of code regions (base heuristics for non-inner-most loop
  // regions).
  void optimizeSelectsBase(Function &F, SelectGroups &ProfSIGroups);
  void optimizeSelectsInnerLoops(Function &F, SelectGroups &ProfSIGroups);

  // Converts to branches the select groups that were deemed
````
- **L221 EN**: Comment documents: `condition. Between select-likes could be any number of auxiliary`.
  **L221 CN**: 注释说明：`condition. Between select-likes could be any number of auxiliary`。
- **L222 EN**: Comment documents: `instructions related to the condition like not, zext, ashr/lshr`.
  **L222 CN**: 注释说明：`instructions related to the condition like not, zext, ashr/lshr`。
- **L223 EN**: Starts the declaration of struct `SelectGroup`.
  **L223 CN**: 开始声明 struct `SelectGroup`。
- **L224 EN**: Executes statement `Value *Condition;`.
  **L224 CN**: 执行语句 `Value *Condition;`。
- **L225 EN**: Executes statement `SmallVector<SelectLike, 2> Selects;`.
  **L225 CN**: 执行语句 `SmallVector<SelectLike, 2> Selects;`。
- **L226 EN**: Closes the current scope.
  **L226 CN**: 关闭当前作用域。
- **L227 EN**: Introduces alias or using-declaration `using SelectGroups = SmallVector<SelectGroup, 2>`.
  **L227 CN**: 引入别名或 using 声明 `using SelectGroups = SmallVector<SelectGroup, 2>`。
- **L228 EN**: Separates nearby statements for readability.
  **L228 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L229 EN**: Comment documents: `Converts select instructions of a function to conditional jumps when dee…`.
  **L229 CN**: 注释说明：`Converts select instructions of a function to conditional jumps when dee…`。
- **L230 EN**: Comment documents: `profitable. Returns true if at least one select was converted.`.
  **L230 CN**: 注释说明：`profitable. Returns true if at least one select was converted.`。
- **L231 EN**: Declares function or method `optimizeSelects`.
  **L231 CN**: 声明函数或方法 `optimizeSelects`。
- **L232 EN**: Separates nearby statements for readability.
  **L232 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L233 EN**: Comment documents: `Heuristics for determining which select instructions can be profitably`.
  **L233 CN**: 注释说明：`Heuristics for determining which select instructions can be profitably`。
- **L234 EN**: Comment documents: `conveted to branches. Separate heuristics for selects in inner-most loop…`.
  **L234 CN**: 注释说明：`conveted to branches. Separate heuristics for selects in inner-most loop…`。
- **L235 EN**: Comment documents: `and the rest of code regions (base heuristics for non-inner-most loop`.
  **L235 CN**: 注释说明：`and the rest of code regions (base heuristics for non-inner-most loop`。
- **L236 EN**: Comment documents: `regions).`.
  **L236 CN**: 注释说明：`regions).`。
- **L237 EN**: Declares function or method `optimizeSelectsBase`.
  **L237 CN**: 声明函数或方法 `optimizeSelectsBase`。
- **L238 EN**: Declares function or method `optimizeSelectsInnerLoops`.
  **L238 CN**: 声明函数或方法 `optimizeSelectsInnerLoops`。
- **L239 EN**: Separates nearby statements for readability.
  **L239 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L240 EN**: Comment documents: `Converts to branches the select groups that were deemed`.
  **L240 CN**: 注释说明：`Converts to branches the select groups that were deemed`。

### Lines 241-260

````cpp
  // profitable-to-convert.
  void convertProfitableSIGroups(SelectGroups &ProfSIGroups);

  // Splits selects of a given basic block into select groups.
  void collectSelectGroups(BasicBlock &BB, SelectGroups &SIGroups);

  // Determines for which select groups it is profitable converting to branches
  // (base and inner-most-loop heuristics).
  void findProfitableSIGroupsBase(SelectGroups &SIGroups,
                                  SelectGroups &ProfSIGroups);
  void findProfitableSIGroupsInnerLoops(const Loop *L, SelectGroups &SIGroups,
                                        SelectGroups &ProfSIGroups);

  // Determines if a select group should be converted to a branch (base
  // heuristics).
  bool isConvertToBranchProfitableBase(const SelectGroup &ASI);

  // Returns true if there are expensive instructions in the cold value
  // operand's (if any) dependence slice of any of the selects of the given
  // group.
````
- **L241 EN**: Comment documents: `profitable-to-convert.`.
  **L241 CN**: 注释说明：`profitable-to-convert.`。
- **L242 EN**: Declares function or method `convertProfitableSIGroups`.
  **L242 CN**: 声明函数或方法 `convertProfitableSIGroups`。
- **L243 EN**: Separates nearby statements for readability.
  **L243 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L244 EN**: Comment documents: `Splits selects of a given basic block into select groups.`.
  **L244 CN**: 注释说明：`Splits selects of a given basic block into select groups.`。
- **L245 EN**: Declares function or method `collectSelectGroups`.
  **L245 CN**: 声明函数或方法 `collectSelectGroups`。
- **L246 EN**: Separates nearby statements for readability.
  **L246 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L247 EN**: Comment documents: `Determines for which select groups it is profitable converting to branch…`.
  **L247 CN**: 注释说明：`Determines for which select groups it is profitable converting to branch…`。
- **L248 EN**: Comment documents: `(base and inner-most-loop heuristics).`.
  **L248 CN**: 注释说明：`(base and inner-most-loop heuristics).`。
- **L249 EN**: Provides part of the signature for `findProfitableSIGroupsBase`.
  **L249 CN**: 给出 `findProfitableSIGroupsBase` 的一部分签名。
- **L250 EN**: Executes statement `SelectGroups &ProfSIGroups);`.
  **L250 CN**: 执行语句 `SelectGroups &ProfSIGroups);`。
- **L251 EN**: Provides part of the signature for `findProfitableSIGroupsInnerLoops`.
  **L251 CN**: 给出 `findProfitableSIGroupsInnerLoops` 的一部分签名。
- **L252 EN**: Executes statement `SelectGroups &ProfSIGroups);`.
  **L252 CN**: 执行语句 `SelectGroups &ProfSIGroups);`。
- **L253 EN**: Separates nearby statements for readability.
  **L253 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L254 EN**: Comment documents: `Determines if a select group should be converted to a branch (base`.
  **L254 CN**: 注释说明：`Determines if a select group should be converted to a branch (base`。
- **L255 EN**: Comment documents: `heuristics).`.
  **L255 CN**: 注释说明：`heuristics).`。
- **L256 EN**: Declares function or method `isConvertToBranchProfitableBase`.
  **L256 CN**: 声明函数或方法 `isConvertToBranchProfitableBase`。
- **L257 EN**: Separates nearby statements for readability.
  **L257 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L258 EN**: Comment documents: `Returns true if there are expensive instructions in the cold value`.
  **L258 CN**: 注释说明：`Returns true if there are expensive instructions in the cold value`。
- **L259 EN**: Comment documents: `operand's (if any) dependence slice of any of the selects of the given`.
  **L259 CN**: 注释说明：`operand's (if any) dependence slice of any of the selects of the given`。
- **L260 EN**: Comment documents: `group.`.
  **L260 CN**: 注释说明：`group.`。

### Lines 261-280

````cpp
  bool hasExpensiveColdOperand(const SelectGroup &ASI);

  // For a given source instruction, collect its backwards dependence slice
  // consisting of instructions exclusively computed for producing the operands
  // of the source instruction.
  void getExclBackwardsSlice(Instruction *I, std::stack<Instruction *> &Slice,
                             Instruction *SI, bool ForSinking = false);

  // Returns true if the condition of the select is highly predictable.
  bool isSelectHighlyPredictable(const SelectLike SI);

  // Loop-level checks to determine if a non-predicated version (with branches)
  // of the given loop is more profitable than its predicated version.
  bool checkLoopHeuristics(const Loop *L, const CostInfo LoopDepth[2]);

  // Computes instruction and loop-critical-path costs for both the predicated
  // and non-predicated version of the given loop.
  bool computeLoopCosts(const Loop *L, const SelectGroups &SIGroups,
                        DenseMap<const Instruction *, CostInfo> &InstCostMap,
                        CostInfo *LoopCost);
````
- **L261 EN**: Declares function or method `hasExpensiveColdOperand`.
  **L261 CN**: 声明函数或方法 `hasExpensiveColdOperand`。
- **L262 EN**: Separates nearby statements for readability.
  **L262 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L263 EN**: Comment documents: `For a given source instruction, collect its backwards dependence slice`.
  **L263 CN**: 注释说明：`For a given source instruction, collect its backwards dependence slice`。
- **L264 EN**: Comment documents: `consisting of instructions exclusively computed for producing the operan…`.
  **L264 CN**: 注释说明：`consisting of instructions exclusively computed for producing the operan…`。
- **L265 EN**: Comment documents: `of the source instruction.`.
  **L265 CN**: 注释说明：`of the source instruction.`。
- **L266 EN**: Provides part of the signature for `getExclBackwardsSlice`.
  **L266 CN**: 给出 `getExclBackwardsSlice` 的一部分签名。
- **L267 EN**: Assigns or initializes `Instruction *SI, bool ForSinking`.
  **L267 CN**: 对 `Instruction *SI, bool ForSinking` 进行赋值或初始化。
- **L268 EN**: Separates nearby statements for readability.
  **L268 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L269 EN**: Comment documents: `Returns true if the condition of the select is highly predictable.`.
  **L269 CN**: 注释说明：`Returns true if the condition of the select is highly predictable.`。
- **L270 EN**: Declares function or method `isSelectHighlyPredictable`.
  **L270 CN**: 声明函数或方法 `isSelectHighlyPredictable`。
- **L271 EN**: Separates nearby statements for readability.
  **L271 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L272 EN**: Comment documents: `Loop-level checks to determine if a non-predicated version (with branche…`.
  **L272 CN**: 注释说明：`Loop-level checks to determine if a non-predicated version (with branche…`。
- **L273 EN**: Comment documents: `of the given loop is more profitable than its predicated version.`.
  **L273 CN**: 注释说明：`of the given loop is more profitable than its predicated version.`。
- **L274 EN**: Declares function or method `checkLoopHeuristics`.
  **L274 CN**: 声明函数或方法 `checkLoopHeuristics`。
- **L275 EN**: Separates nearby statements for readability.
  **L275 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L276 EN**: Comment documents: `Computes instruction and loop-critical-path costs for both the predicate…`.
  **L276 CN**: 注释说明：`Computes instruction and loop-critical-path costs for both the predicate…`。
- **L277 EN**: Comment documents: `and non-predicated version of the given loop.`.
  **L277 CN**: 注释说明：`and non-predicated version of the given loop.`。
- **L278 EN**: Provides part of the signature for `computeLoopCosts`.
  **L278 CN**: 给出 `computeLoopCosts` 的一部分签名。
- **L279 EN**: Continues logic with `DenseMap<const Instruction *, CostInfo> &InstCostMap,`.
  **L279 CN**: 继续处理逻辑：`DenseMap<const Instruction *, CostInfo> &InstCostMap,`。
- **L280 EN**: Executes statement `CostInfo *LoopCost);`.
  **L280 CN**: 执行语句 `CostInfo *LoopCost);`。

### Lines 281-300

````cpp

  // Returns a set of all the select instructions in the given select groups.
  SmallDenseMap<const Instruction *, SelectLike, 2>
  getSImap(const SelectGroups &SIGroups);

  // Returns a map from select-like instructions to the corresponding select
  // group.
  SmallDenseMap<const Instruction *, const SelectGroup *, 2>
  getSGmap(const SelectGroups &SIGroups);

  // Returns the latency cost of a given instruction.
  std::optional<uint64_t> computeInstCost(const Instruction *I);

  // Returns the misprediction cost of a given select when converted to branch.
  Scaled64 getMispredictionCost(const SelectLike SI, const Scaled64 CondCost);

  // Returns the cost of a branch when the prediction is correct.
  Scaled64 getPredictedPathCost(Scaled64 TrueCost, Scaled64 FalseCost,
                                const SelectLike SI);

````
- **L281 EN**: Separates nearby statements for readability.
  **L281 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L282 EN**: Comment documents: `Returns a set of all the select instructions in the given select groups.`.
  **L282 CN**: 注释说明：`Returns a set of all the select instructions in the given select groups.`。
- **L283 EN**: Continues logic with `SmallDenseMap<const Instruction *, SelectLike, 2>`.
  **L283 CN**: 继续处理逻辑：`SmallDenseMap<const Instruction *, SelectLike, 2>`。
- **L284 EN**: Executes statement `getSImap(const SelectGroups &SIGroups);`.
  **L284 CN**: 执行语句 `getSImap(const SelectGroups &SIGroups);`。
- **L285 EN**: Separates nearby statements for readability.
  **L285 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L286 EN**: Comment documents: `Returns a map from select-like instructions to the corresponding select`.
  **L286 CN**: 注释说明：`Returns a map from select-like instructions to the corresponding select`。
- **L287 EN**: Comment documents: `group.`.
  **L287 CN**: 注释说明：`group.`。
- **L288 EN**: Continues logic with `SmallDenseMap<const Instruction *, const SelectGroup *, 2>`.
  **L288 CN**: 继续处理逻辑：`SmallDenseMap<const Instruction *, const SelectGroup *, 2>`。
- **L289 EN**: Executes statement `getSGmap(const SelectGroups &SIGroups);`.
  **L289 CN**: 执行语句 `getSGmap(const SelectGroups &SIGroups);`。
- **L290 EN**: Separates nearby statements for readability.
  **L290 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L291 EN**: Comment documents: `Returns the latency cost of a given instruction.`.
  **L291 CN**: 注释说明：`Returns the latency cost of a given instruction.`。
- **L292 EN**: Declares function or method `computeInstCost`.
  **L292 CN**: 声明函数或方法 `computeInstCost`。
- **L293 EN**: Separates nearby statements for readability.
  **L293 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L294 EN**: Comment documents: `Returns the misprediction cost of a given select when converted to branc…`.
  **L294 CN**: 注释说明：`Returns the misprediction cost of a given select when converted to branc…`。
- **L295 EN**: Declares function or method `getMispredictionCost`.
  **L295 CN**: 声明函数或方法 `getMispredictionCost`。
- **L296 EN**: Separates nearby statements for readability.
  **L296 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L297 EN**: Comment documents: `Returns the cost of a branch when the prediction is correct.`.
  **L297 CN**: 注释说明：`Returns the cost of a branch when the prediction is correct.`。
- **L298 EN**: Provides part of the signature for `getPredictedPathCost`.
  **L298 CN**: 给出 `getPredictedPathCost` 的一部分签名。
- **L299 EN**: Executes statement `const SelectLike SI);`.
  **L299 CN**: 执行语句 `const SelectLike SI);`。
- **L300 EN**: Separates nearby statements for readability.
  **L300 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 301-320

````cpp
  // Returns true if the target architecture supports lowering a given select.
  bool isSelectKindSupported(const SelectLike SI);
};

class SelectOptimize : public FunctionPass {
  SelectOptimizeImpl Impl;

public:
  static char ID;

  SelectOptimize() : FunctionPass(ID) {}

  bool runOnFunction(Function &F) override {
    if (skipFunction(F))
      return false;

    return Impl.runOnFunction(F, *this);
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
````
- **L301 EN**: Comment documents: `Returns true if the target architecture supports lowering a given select…`.
  **L301 CN**: 注释说明：`Returns true if the target architecture supports lowering a given select…`。
- **L302 EN**: Declares function or method `isSelectKindSupported`.
  **L302 CN**: 声明函数或方法 `isSelectKindSupported`。
- **L303 EN**: Closes the current scope.
  **L303 CN**: 关闭当前作用域。
- **L304 EN**: Separates nearby statements for readability.
  **L304 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L305 EN**: Starts the declaration of class `SelectOptimize`.
  **L305 CN**: 开始声明 class `SelectOptimize`。
- **L306 EN**: Executes statement `SelectOptimizeImpl Impl;`.
  **L306 CN**: 执行语句 `SelectOptimizeImpl Impl;`。
- **L307 EN**: Separates nearby statements for readability.
  **L307 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L308 EN**: Continues logic with `public:`.
  **L308 CN**: 继续处理逻辑：`public:`。
- **L309 EN**: Executes statement `static char ID;`.
  **L309 CN**: 执行语句 `static char ID;`。
- **L310 EN**: Separates nearby statements for readability.
  **L310 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L311 EN**: Continues logic with `SelectOptimize() : FunctionPass(ID) {}`.
  **L311 CN**: 继续处理逻辑：`SelectOptimize() : FunctionPass(ID) {}`。
- **L312 EN**: Separates nearby statements for readability.
  **L312 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L313 EN**: Begins the definition of `runOnFunction`.
  **L313 CN**: 开始定义 `runOnFunction`。
- **L314 EN**: Begins a conditional branch.
  **L314 CN**: 开始一个条件分支。
- **L315 EN**: Returns `false` to the caller.
  **L315 CN**: 向调用者返回 `false`。
- **L316 EN**: Separates nearby statements for readability.
  **L316 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L317 EN**: Returns `Impl.runOnFunction(F, *this)` to the caller.
  **L317 CN**: 向调用者返回 `Impl.runOnFunction(F, *this)`。
- **L318 EN**: Closes the current scope.
  **L318 CN**: 关闭当前作用域。
- **L319 EN**: Separates nearby statements for readability.
  **L319 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L320 EN**: Begins the definition of `getAnalysisUsage`.
  **L320 CN**: 开始定义 `getAnalysisUsage`。

### Lines 321-340

````cpp
    AU.addRequired<ProfileSummaryInfoWrapperPass>();
    AU.addRequired<TargetPassConfig>();
    AU.addRequired<TargetTransformInfoWrapperPass>();
    AU.addRequired<LoopInfoWrapperPass>();
    AU.addRequired<BlockFrequencyInfoWrapperPass>();
    AU.addRequired<OptimizationRemarkEmitterWrapperPass>();
  }
};

} // namespace

PreservedAnalyses SelectOptimizePass::run(Function &F,
                                          FunctionAnalysisManager &FAM) {
  SelectOptimizeImpl Impl(TM);
  return Impl.run(F, FAM);
}

char SelectOptimize::ID = 0;

INITIALIZE_PASS_BEGIN(SelectOptimize, DEBUG_TYPE, "Optimize selects", false,
````
- **L321 EN**: Executes statement `AU.addRequired<ProfileSummaryInfoWrapperPass>();`.
  **L321 CN**: 执行语句 `AU.addRequired<ProfileSummaryInfoWrapperPass>();`。
- **L322 EN**: Executes statement `AU.addRequired<TargetPassConfig>();`.
  **L322 CN**: 执行语句 `AU.addRequired<TargetPassConfig>();`。
- **L323 EN**: Executes statement `AU.addRequired<TargetTransformInfoWrapperPass>();`.
  **L323 CN**: 执行语句 `AU.addRequired<TargetTransformInfoWrapperPass>();`。
- **L324 EN**: Executes statement `AU.addRequired<LoopInfoWrapperPass>();`.
  **L324 CN**: 执行语句 `AU.addRequired<LoopInfoWrapperPass>();`。
- **L325 EN**: Executes statement `AU.addRequired<BlockFrequencyInfoWrapperPass>();`.
  **L325 CN**: 执行语句 `AU.addRequired<BlockFrequencyInfoWrapperPass>();`。
- **L326 EN**: Executes statement `AU.addRequired<OptimizationRemarkEmitterWrapperPass>();`.
  **L326 CN**: 执行语句 `AU.addRequired<OptimizationRemarkEmitterWrapperPass>();`。
- **L327 EN**: Closes the current scope.
  **L327 CN**: 关闭当前作用域。
- **L328 EN**: Closes the current scope.
  **L328 CN**: 关闭当前作用域。
- **L329 EN**: Separates nearby statements for readability.
  **L329 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L330 EN**: Continues logic with `} // namespace`.
  **L330 CN**: 继续处理逻辑：`} // namespace`。
- **L331 EN**: Separates nearby statements for readability.
  **L331 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L332 EN**: Provides part of the signature for `run`.
  **L332 CN**: 给出 `run` 的一部分签名。
- **L333 EN**: Starts block `FunctionAnalysisManager &FAM)`.
  **L333 CN**: 开始代码块 `FunctionAnalysisManager &FAM)`。
- **L334 EN**: Declares function or method `Impl`.
  **L334 CN**: 声明函数或方法 `Impl`。
- **L335 EN**: Returns `Impl.run(F, FAM)` to the caller.
  **L335 CN**: 向调用者返回 `Impl.run(F, FAM)`。
- **L336 EN**: Closes the current scope.
  **L336 CN**: 关闭当前作用域。
- **L337 EN**: Separates nearby statements for readability.
  **L337 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L338 EN**: Assigns or initializes `char SelectOptimize::ID`.
  **L338 CN**: 对 `char SelectOptimize::ID` 进行赋值或初始化。
- **L339 EN**: Separates nearby statements for readability.
  **L339 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L340 EN**: Continues logic with `INITIALIZE_PASS_BEGIN(SelectOptimize, DEBUG_TYPE, "Optimize selects", fa…`.
  **L340 CN**: 继续处理逻辑：`INITIALIZE_PASS_BEGIN(SelectOptimize, DEBUG_TYPE, "Optimize selects", fa…`。

### Lines 341-360

````cpp
                      false)
INITIALIZE_PASS_DEPENDENCY(LoopInfoWrapperPass)
INITIALIZE_PASS_DEPENDENCY(ProfileSummaryInfoWrapperPass)
INITIALIZE_PASS_DEPENDENCY(TargetPassConfig)
INITIALIZE_PASS_DEPENDENCY(TargetTransformInfoWrapperPass)
INITIALIZE_PASS_DEPENDENCY(BlockFrequencyInfoWrapperPass)
INITIALIZE_PASS_DEPENDENCY(OptimizationRemarkEmitterWrapperPass)
INITIALIZE_PASS_END(SelectOptimize, DEBUG_TYPE, "Optimize selects", false,
                    false)

FunctionPass *llvm::createSelectOptimizePass() { return new SelectOptimize(); }

PreservedAnalyses SelectOptimizeImpl::run(Function &F,
                                          FunctionAnalysisManager &FAM) {
  TSI = TM->getSubtargetImpl(F);
  TLI = TSI->getTargetLowering();

  // If none of the select types are supported then skip this pass.
  // This is an optimization pass. Legality issues will be handled by
  // instruction selection.
````
- **L341 EN**: Continues logic with `false)`.
  **L341 CN**: 继续处理逻辑：`false)`。
- **L342 EN**: Continues logic with `INITIALIZE_PASS_DEPENDENCY(LoopInfoWrapperPass)`.
  **L342 CN**: 继续处理逻辑：`INITIALIZE_PASS_DEPENDENCY(LoopInfoWrapperPass)`。
- **L343 EN**: Continues logic with `INITIALIZE_PASS_DEPENDENCY(ProfileSummaryInfoWrapperPass)`.
  **L343 CN**: 继续处理逻辑：`INITIALIZE_PASS_DEPENDENCY(ProfileSummaryInfoWrapperPass)`。
- **L344 EN**: Continues logic with `INITIALIZE_PASS_DEPENDENCY(TargetPassConfig)`.
  **L344 CN**: 继续处理逻辑：`INITIALIZE_PASS_DEPENDENCY(TargetPassConfig)`。
- **L345 EN**: Continues logic with `INITIALIZE_PASS_DEPENDENCY(TargetTransformInfoWrapperPass)`.
  **L345 CN**: 继续处理逻辑：`INITIALIZE_PASS_DEPENDENCY(TargetTransformInfoWrapperPass)`。
- **L346 EN**: Continues logic with `INITIALIZE_PASS_DEPENDENCY(BlockFrequencyInfoWrapperPass)`.
  **L346 CN**: 继续处理逻辑：`INITIALIZE_PASS_DEPENDENCY(BlockFrequencyInfoWrapperPass)`。
- **L347 EN**: Continues logic with `INITIALIZE_PASS_DEPENDENCY(OptimizationRemarkEmitterWrapperPass)`.
  **L347 CN**: 继续处理逻辑：`INITIALIZE_PASS_DEPENDENCY(OptimizationRemarkEmitterWrapperPass)`。
- **L348 EN**: Continues logic with `INITIALIZE_PASS_END(SelectOptimize, DEBUG_TYPE, "Optimize selects", fals…`.
  **L348 CN**: 继续处理逻辑：`INITIALIZE_PASS_END(SelectOptimize, DEBUG_TYPE, "Optimize selects", fals…`。
- **L349 EN**: Continues logic with `false)`.
  **L349 CN**: 继续处理逻辑：`false)`。
- **L350 EN**: Separates nearby statements for readability.
  **L350 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L351 EN**: Provides part of the signature for `createSelectOptimizePass`.
  **L351 CN**: 给出 `createSelectOptimizePass` 的一部分签名。
- **L352 EN**: Separates nearby statements for readability.
  **L352 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L353 EN**: Provides part of the signature for `run`.
  **L353 CN**: 给出 `run` 的一部分签名。
- **L354 EN**: Starts block `FunctionAnalysisManager &FAM)`.
  **L354 CN**: 开始代码块 `FunctionAnalysisManager &FAM)`。
- **L355 EN**: Assigns or initializes `TSI`.
  **L355 CN**: 对 `TSI` 进行赋值或初始化。
- **L356 EN**: Assigns or initializes `TLI`.
  **L356 CN**: 对 `TLI` 进行赋值或初始化。
- **L357 EN**: Separates nearby statements for readability.
  **L357 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L358 EN**: Comment documents: `If none of the select types are supported then skip this pass.`.
  **L358 CN**: 注释说明：`If none of the select types are supported then skip this pass.`。
- **L359 EN**: Comment documents: `This is an optimization pass. Legality issues will be handled by`.
  **L359 CN**: 注释说明：`This is an optimization pass. Legality issues will be handled by`。
- **L360 EN**: Comment documents: `instruction selection.`.
  **L360 CN**: 注释说明：`instruction selection.`。

### Lines 361-380

````cpp
  if (!TLI->isSelectSupported(TargetLowering::ScalarValSelect) &&
      !TLI->isSelectSupported(TargetLowering::ScalarCondVectorVal) &&
      !TLI->isSelectSupported(TargetLowering::VectorMaskSelect))
    return PreservedAnalyses::all();

  TTI = &FAM.getResult<TargetIRAnalysis>(F);
  if (!TTI->enableSelectOptimize())
    return PreservedAnalyses::all();

  PSI = FAM.getResult<ModuleAnalysisManagerFunctionProxy>(F)
            .getCachedResult<ProfileSummaryAnalysis>(*F.getParent());
  if (!PSI)
    reportFatalUsageError("this pass requires the profile-summary module "
                          "analysis to be available");
  BFI = &FAM.getResult<BlockFrequencyAnalysis>(F);

  // When optimizing for size, selects are preferable over branches.
  if (llvm::shouldOptimizeForSize(&F, PSI, BFI))
    return PreservedAnalyses::all();

````
- **L361 EN**: Begins a conditional branch.
  **L361 CN**: 开始一个条件分支。
- **L362 EN**: Continues logic with `!TLI->isSelectSupported(TargetLowering::ScalarCondVectorVal) &&`.
  **L362 CN**: 继续处理逻辑：`!TLI->isSelectSupported(TargetLowering::ScalarCondVectorVal) &&`。
- **L363 EN**: Continues logic with `!TLI->isSelectSupported(TargetLowering::VectorMaskSelect))`.
  **L363 CN**: 继续处理逻辑：`!TLI->isSelectSupported(TargetLowering::VectorMaskSelect))`。
- **L364 EN**: Returns `PreservedAnalyses::all()` to the caller.
  **L364 CN**: 向调用者返回 `PreservedAnalyses::all()`。
- **L365 EN**: Separates nearby statements for readability.
  **L365 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L366 EN**: Assigns or initializes `TTI`.
  **L366 CN**: 对 `TTI` 进行赋值或初始化。
- **L367 EN**: Begins a conditional branch.
  **L367 CN**: 开始一个条件分支。
- **L368 EN**: Returns `PreservedAnalyses::all()` to the caller.
  **L368 CN**: 向调用者返回 `PreservedAnalyses::all()`。
- **L369 EN**: Separates nearby statements for readability.
  **L369 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L370 EN**: Continues logic with `PSI = FAM.getResult<ModuleAnalysisManagerFunctionProxy>(F)`.
  **L370 CN**: 继续处理逻辑：`PSI = FAM.getResult<ModuleAnalysisManagerFunctionProxy>(F)`。
- **L371 EN**: Executes statement `.getCachedResult<ProfileSummaryAnalysis>(*F.getParent());`.
  **L371 CN**: 执行语句 `.getCachedResult<ProfileSummaryAnalysis>(*F.getParent());`。
- **L372 EN**: Begins a conditional branch.
  **L372 CN**: 开始一个条件分支。
- **L373 EN**: Continues logic with `reportFatalUsageError("this pass requires the profile-summary module "`.
  **L373 CN**: 继续处理逻辑：`reportFatalUsageError("this pass requires the profile-summary module "`。
- **L374 EN**: Executes statement `"analysis to be available");`.
  **L374 CN**: 执行语句 `"analysis to be available");`。
- **L375 EN**: Assigns or initializes `BFI`.
  **L375 CN**: 对 `BFI` 进行赋值或初始化。
- **L376 EN**: Separates nearby statements for readability.
  **L376 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L377 EN**: Comment documents: `When optimizing for size, selects are preferable over branches.`.
  **L377 CN**: 注释说明：`When optimizing for size, selects are preferable over branches.`。
- **L378 EN**: Begins a conditional branch.
  **L378 CN**: 开始一个条件分支。
- **L379 EN**: Returns `PreservedAnalyses::all()` to the caller.
  **L379 CN**: 向调用者返回 `PreservedAnalyses::all()`。
- **L380 EN**: Separates nearby statements for readability.
  **L380 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 381-400

````cpp
  LI = &FAM.getResult<LoopAnalysis>(F);
  ORE = &FAM.getResult<OptimizationRemarkEmitterAnalysis>(F);
  TSchedModel.init(TSI);

  bool Changed = optimizeSelects(F);
  return Changed ? PreservedAnalyses::none() : PreservedAnalyses::all();
}

bool SelectOptimizeImpl::runOnFunction(Function &F, Pass &P) {
  TM = &P.getAnalysis<TargetPassConfig>().getTM<TargetMachine>();
  TSI = TM->getSubtargetImpl(F);
  TLI = TSI->getTargetLowering();

  // If none of the select types are supported then skip this pass.
  // This is an optimization pass. Legality issues will be handled by
  // instruction selection.
  if (!TLI->isSelectSupported(TargetLowering::ScalarValSelect) &&
      !TLI->isSelectSupported(TargetLowering::ScalarCondVectorVal) &&
      !TLI->isSelectSupported(TargetLowering::VectorMaskSelect))
    return false;
````
- **L381 EN**: Assigns or initializes `LI`.
  **L381 CN**: 对 `LI` 进行赋值或初始化。
- **L382 EN**: Assigns or initializes `ORE`.
  **L382 CN**: 对 `ORE` 进行赋值或初始化。
- **L383 EN**: Executes statement `TSchedModel.init(TSI);`.
  **L383 CN**: 执行语句 `TSchedModel.init(TSI);`。
- **L384 EN**: Separates nearby statements for readability.
  **L384 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L385 EN**: Assigns or initializes `bool Changed`.
  **L385 CN**: 对 `bool Changed` 进行赋值或初始化。
- **L386 EN**: Returns `Changed ? PreservedAnalyses::none() : PreservedAnalyses::all()` to the caller.
  **L386 CN**: 向调用者返回 `Changed ? PreservedAnalyses::none() : PreservedAnalyses::all()`。
- **L387 EN**: Closes the current scope.
  **L387 CN**: 关闭当前作用域。
- **L388 EN**: Separates nearby statements for readability.
  **L388 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L389 EN**: Begins the definition of `runOnFunction`.
  **L389 CN**: 开始定义 `runOnFunction`。
- **L390 EN**: Assigns or initializes `TM`.
  **L390 CN**: 对 `TM` 进行赋值或初始化。
- **L391 EN**: Assigns or initializes `TSI`.
  **L391 CN**: 对 `TSI` 进行赋值或初始化。
- **L392 EN**: Assigns or initializes `TLI`.
  **L392 CN**: 对 `TLI` 进行赋值或初始化。
- **L393 EN**: Separates nearby statements for readability.
  **L393 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L394 EN**: Comment documents: `If none of the select types are supported then skip this pass.`.
  **L394 CN**: 注释说明：`If none of the select types are supported then skip this pass.`。
- **L395 EN**: Comment documents: `This is an optimization pass. Legality issues will be handled by`.
  **L395 CN**: 注释说明：`This is an optimization pass. Legality issues will be handled by`。
- **L396 EN**: Comment documents: `instruction selection.`.
  **L396 CN**: 注释说明：`instruction selection.`。
- **L397 EN**: Begins a conditional branch.
  **L397 CN**: 开始一个条件分支。
- **L398 EN**: Continues logic with `!TLI->isSelectSupported(TargetLowering::ScalarCondVectorVal) &&`.
  **L398 CN**: 继续处理逻辑：`!TLI->isSelectSupported(TargetLowering::ScalarCondVectorVal) &&`。
- **L399 EN**: Continues logic with `!TLI->isSelectSupported(TargetLowering::VectorMaskSelect))`.
  **L399 CN**: 继续处理逻辑：`!TLI->isSelectSupported(TargetLowering::VectorMaskSelect))`。
- **L400 EN**: Returns `false` to the caller.
  **L400 CN**: 向调用者返回 `false`。

### Lines 401-420

````cpp

  TTI = &P.getAnalysis<TargetTransformInfoWrapperPass>().getTTI(F);

  if (!TTI->enableSelectOptimize())
    return false;

  LI = &P.getAnalysis<LoopInfoWrapperPass>().getLoopInfo();
  BFI = &P.getAnalysis<BlockFrequencyInfoWrapperPass>().getBFI();
  PSI = &P.getAnalysis<ProfileSummaryInfoWrapperPass>().getPSI();
  ORE = &P.getAnalysis<OptimizationRemarkEmitterWrapperPass>().getORE();
  TSchedModel.init(TSI);

  // When optimizing for size, selects are preferable over branches.
  if (llvm::shouldOptimizeForSize(&F, PSI, BFI))
    return false;

  return optimizeSelects(F);
}

bool SelectOptimizeImpl::optimizeSelects(Function &F) {
````
- **L401 EN**: Separates nearby statements for readability.
  **L401 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L402 EN**: Assigns or initializes `TTI`.
  **L402 CN**: 对 `TTI` 进行赋值或初始化。
- **L403 EN**: Separates nearby statements for readability.
  **L403 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L404 EN**: Begins a conditional branch.
  **L404 CN**: 开始一个条件分支。
- **L405 EN**: Returns `false` to the caller.
  **L405 CN**: 向调用者返回 `false`。
- **L406 EN**: Separates nearby statements for readability.
  **L406 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L407 EN**: Assigns or initializes `LI`.
  **L407 CN**: 对 `LI` 进行赋值或初始化。
- **L408 EN**: Assigns or initializes `BFI`.
  **L408 CN**: 对 `BFI` 进行赋值或初始化。
- **L409 EN**: Assigns or initializes `PSI`.
  **L409 CN**: 对 `PSI` 进行赋值或初始化。
- **L410 EN**: Assigns or initializes `ORE`.
  **L410 CN**: 对 `ORE` 进行赋值或初始化。
- **L411 EN**: Executes statement `TSchedModel.init(TSI);`.
  **L411 CN**: 执行语句 `TSchedModel.init(TSI);`。
- **L412 EN**: Separates nearby statements for readability.
  **L412 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L413 EN**: Comment documents: `When optimizing for size, selects are preferable over branches.`.
  **L413 CN**: 注释说明：`When optimizing for size, selects are preferable over branches.`。
- **L414 EN**: Begins a conditional branch.
  **L414 CN**: 开始一个条件分支。
- **L415 EN**: Returns `false` to the caller.
  **L415 CN**: 向调用者返回 `false`。
- **L416 EN**: Separates nearby statements for readability.
  **L416 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L417 EN**: Returns `optimizeSelects(F)` to the caller.
  **L417 CN**: 向调用者返回 `optimizeSelects(F)`。
- **L418 EN**: Closes the current scope.
  **L418 CN**: 关闭当前作用域。
- **L419 EN**: Separates nearby statements for readability.
  **L419 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L420 EN**: Begins the definition of `optimizeSelects`.
  **L420 CN**: 开始定义 `optimizeSelects`。

### Lines 421-440

````cpp
  // Determine for which select groups it is profitable converting to branches.
  SelectGroups ProfSIGroups;
  // Base heuristics apply only to non-loops and outer loops.
  optimizeSelectsBase(F, ProfSIGroups);
  // Separate heuristics for inner-most loops.
  optimizeSelectsInnerLoops(F, ProfSIGroups);

  // Convert to branches the select groups that were deemed
  // profitable-to-convert.
  convertProfitableSIGroups(ProfSIGroups);

  // Code modified if at least one select group was converted.
  return !ProfSIGroups.empty();
}

void SelectOptimizeImpl::optimizeSelectsBase(Function &F,
                                             SelectGroups &ProfSIGroups) {
  // Collect all the select groups.
  SelectGroups SIGroups;
  for (BasicBlock &BB : F) {
````
- **L421 EN**: Comment documents: `Determine for which select groups it is profitable converting to branche…`.
  **L421 CN**: 注释说明：`Determine for which select groups it is profitable converting to branche…`。
- **L422 EN**: Executes statement `SelectGroups ProfSIGroups;`.
  **L422 CN**: 执行语句 `SelectGroups ProfSIGroups;`。
- **L423 EN**: Comment documents: `Base heuristics apply only to non-loops and outer loops.`.
  **L423 CN**: 注释说明：`Base heuristics apply only to non-loops and outer loops.`。
- **L424 EN**: Executes statement `optimizeSelectsBase(F, ProfSIGroups);`.
  **L424 CN**: 执行语句 `optimizeSelectsBase(F, ProfSIGroups);`。
- **L425 EN**: Comment documents: `Separate heuristics for inner-most loops.`.
  **L425 CN**: 注释说明：`Separate heuristics for inner-most loops.`。
- **L426 EN**: Executes statement `optimizeSelectsInnerLoops(F, ProfSIGroups);`.
  **L426 CN**: 执行语句 `optimizeSelectsInnerLoops(F, ProfSIGroups);`。
- **L427 EN**: Separates nearby statements for readability.
  **L427 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L428 EN**: Comment documents: `Convert to branches the select groups that were deemed`.
  **L428 CN**: 注释说明：`Convert to branches the select groups that were deemed`。
- **L429 EN**: Comment documents: `profitable-to-convert.`.
  **L429 CN**: 注释说明：`profitable-to-convert.`。
- **L430 EN**: Executes statement `convertProfitableSIGroups(ProfSIGroups);`.
  **L430 CN**: 执行语句 `convertProfitableSIGroups(ProfSIGroups);`。
- **L431 EN**: Separates nearby statements for readability.
  **L431 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L432 EN**: Comment documents: `Code modified if at least one select group was converted.`.
  **L432 CN**: 注释说明：`Code modified if at least one select group was converted.`。
- **L433 EN**: Returns `!ProfSIGroups.empty()` to the caller.
  **L433 CN**: 向调用者返回 `!ProfSIGroups.empty()`。
- **L434 EN**: Closes the current scope.
  **L434 CN**: 关闭当前作用域。
- **L435 EN**: Separates nearby statements for readability.
  **L435 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L436 EN**: Provides part of the signature for `optimizeSelectsBase`.
  **L436 CN**: 给出 `optimizeSelectsBase` 的一部分签名。
- **L437 EN**: Starts block `SelectGroups &ProfSIGroups)`.
  **L437 CN**: 开始代码块 `SelectGroups &ProfSIGroups)`。
- **L438 EN**: Comment documents: `Collect all the select groups.`.
  **L438 CN**: 注释说明：`Collect all the select groups.`。
- **L439 EN**: Executes statement `SelectGroups SIGroups;`.
  **L439 CN**: 执行语句 `SelectGroups SIGroups;`。
- **L440 EN**: Starts a loop over a sequence or range.
  **L440 CN**: 开始遍历序列或范围的循环。

### Lines 441-460

````cpp
    // Base heuristics apply only to non-loops and outer loops.
    Loop *L = LI->getLoopFor(&BB);
    if (L && L->isInnermost())
      continue;
    collectSelectGroups(BB, SIGroups);
  }

  // Determine for which select groups it is profitable converting to branches.
  findProfitableSIGroupsBase(SIGroups, ProfSIGroups);
}

void SelectOptimizeImpl::optimizeSelectsInnerLoops(Function &F,
                                                   SelectGroups &ProfSIGroups) {
  SmallVector<Loop *, 4> Loops(LI->begin(), LI->end());
  // Need to check size on each iteration as we accumulate child loops.
  for (unsigned long i = 0; i < Loops.size(); ++i)
    llvm::append_range(Loops, Loops[i]->getSubLoops());

  for (Loop *L : Loops) {
    if (!L->isInnermost())
````
- **L441 EN**: Comment documents: `Base heuristics apply only to non-loops and outer loops.`.
  **L441 CN**: 注释说明：`Base heuristics apply only to non-loops and outer loops.`。
- **L442 EN**: Assigns or initializes `Loop *L`.
  **L442 CN**: 对 `Loop *L` 进行赋值或初始化。
- **L443 EN**: Begins a conditional branch.
  **L443 CN**: 开始一个条件分支。
- **L444 EN**: Skips to the next loop iteration.
  **L444 CN**: 跳到下一次循环迭代。
- **L445 EN**: Executes statement `collectSelectGroups(BB, SIGroups);`.
  **L445 CN**: 执行语句 `collectSelectGroups(BB, SIGroups);`。
- **L446 EN**: Closes the current scope.
  **L446 CN**: 关闭当前作用域。
- **L447 EN**: Separates nearby statements for readability.
  **L447 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L448 EN**: Comment documents: `Determine for which select groups it is profitable converting to branche…`.
  **L448 CN**: 注释说明：`Determine for which select groups it is profitable converting to branche…`。
- **L449 EN**: Executes statement `findProfitableSIGroupsBase(SIGroups, ProfSIGroups);`.
  **L449 CN**: 执行语句 `findProfitableSIGroupsBase(SIGroups, ProfSIGroups);`。
- **L450 EN**: Closes the current scope.
  **L450 CN**: 关闭当前作用域。
- **L451 EN**: Separates nearby statements for readability.
  **L451 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L452 EN**: Provides part of the signature for `optimizeSelectsInnerLoops`.
  **L452 CN**: 给出 `optimizeSelectsInnerLoops` 的一部分签名。
- **L453 EN**: Starts block `SelectGroups &ProfSIGroups)`.
  **L453 CN**: 开始代码块 `SelectGroups &ProfSIGroups)`。
- **L454 EN**: Declares function or method `Loops`.
  **L454 CN**: 声明函数或方法 `Loops`。
- **L455 EN**: Comment documents: `Need to check size on each iteration as we accumulate child loops.`.
  **L455 CN**: 注释说明：`Need to check size on each iteration as we accumulate child loops.`。
- **L456 EN**: Starts a loop over a sequence or range.
  **L456 CN**: 开始遍历序列或范围的循环。
- **L457 EN**: Declares function or method `append_range`.
  **L457 CN**: 声明函数或方法 `append_range`。
- **L458 EN**: Separates nearby statements for readability.
  **L458 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L459 EN**: Starts a loop over a sequence or range.
  **L459 CN**: 开始遍历序列或范围的循环。
- **L460 EN**: Begins a conditional branch.
  **L460 CN**: 开始一个条件分支。

### Lines 461-480

````cpp
      continue;

    SelectGroups SIGroups;
    for (BasicBlock *BB : L->getBlocks())
      collectSelectGroups(*BB, SIGroups);

    findProfitableSIGroupsInnerLoops(L, SIGroups, ProfSIGroups);
  }
}

/// Returns optimised value on \p IsTrue branch. For SelectInst that would be
/// either True or False value. For (BinaryOperator) instructions, where the
/// condition may be skipped, the operation will use a non-conditional operand.
/// For example, for `or(V,zext(cond))` this function would return V.
/// However, if the conditional operand on \p IsTrue branch matters, we create a
/// clone of instruction at the end of that branch \p B and replace the
/// condition operand with a constant.
///
/// Also /p OptSelects contains previously optimised select-like instructions.
/// If the current value uses one of the optimised values, we can optimise it
````
- **L461 EN**: Skips to the next loop iteration.
  **L461 CN**: 跳到下一次循环迭代。
- **L462 EN**: Separates nearby statements for readability.
  **L462 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L463 EN**: Executes statement `SelectGroups SIGroups;`.
  **L463 CN**: 执行语句 `SelectGroups SIGroups;`。
- **L464 EN**: Starts a loop over a sequence or range.
  **L464 CN**: 开始遍历序列或范围的循环。
- **L465 EN**: Executes statement `collectSelectGroups(*BB, SIGroups);`.
  **L465 CN**: 执行语句 `collectSelectGroups(*BB, SIGroups);`。
- **L466 EN**: Separates nearby statements for readability.
  **L466 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L467 EN**: Executes statement `findProfitableSIGroupsInnerLoops(L, SIGroups, ProfSIGroups);`.
  **L467 CN**: 执行语句 `findProfitableSIGroupsInnerLoops(L, SIGroups, ProfSIGroups);`。
- **L468 EN**: Closes the current scope.
  **L468 CN**: 关闭当前作用域。
- **L469 EN**: Closes the current scope.
  **L469 CN**: 关闭当前作用域。
- **L470 EN**: Separates nearby statements for readability.
  **L470 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L471 EN**: Comment documents: `Returns optimised value on \p IsTrue branch. For SelectInst that would b…`.
  **L471 CN**: 注释说明：`Returns optimised value on \p IsTrue branch. For SelectInst that would b…`。
- **L472 EN**: Comment documents: `either True or False value. For (BinaryOperator) instructions, where the`.
  **L472 CN**: 注释说明：`either True or False value. For (BinaryOperator) instructions, where the`。
- **L473 EN**: Comment documents: `condition may be skipped, the operation will use a non-conditional opera…`.
  **L473 CN**: 注释说明：`condition may be skipped, the operation will use a non-conditional opera…`。
- **L474 EN**: Comment documents: `For example, for 'or(V,zext(cond))' this function would return V.`.
  **L474 CN**: 注释说明：`For example, for 'or(V,zext(cond))' this function would return V.`。
- **L475 EN**: Comment documents: `However, if the conditional operand on \p IsTrue branch matters, we crea…`.
  **L475 CN**: 注释说明：`However, if the conditional operand on \p IsTrue branch matters, we crea…`。
- **L476 EN**: Comment documents: `clone of instruction at the end of that branch \p B and replace the`.
  **L476 CN**: 注释说明：`clone of instruction at the end of that branch \p B and replace the`。
- **L477 EN**: Comment documents: `condition operand with a constant.`.
  **L477 CN**: 注释说明：`condition operand with a constant.`。
- **L478 EN**: Continues the surrounding comment block.
  **L478 CN**: 延续周围的注释块。
- **L479 EN**: Comment documents: `Also /p OptSelects contains previously optimised select-like instruction…`.
  **L479 CN**: 注释说明：`Also /p OptSelects contains previously optimised select-like instruction…`。
- **L480 EN**: Comment documents: `If the current value uses one of the optimised values, we can optimise i…`.
  **L480 CN**: 注释说明：`If the current value uses one of the optimised values, we can optimise i…`。

### Lines 481-500

````cpp
/// further by replacing it with the corresponding value on the given branch
static Value *getTrueOrFalseValue(
    SelectOptimizeImpl::SelectLike &SI, bool isTrue,
    SmallDenseMap<Instruction *, std::pair<Value *, Value *>, 2> &OptSelects,
    BasicBlock *B) {
  Value *V = isTrue ? SI.getTrueValue() : SI.getFalseValue();
  if (V) {
    if (auto *IV = dyn_cast<Instruction>(V))
      if (auto It = OptSelects.find(IV); It != OptSelects.end())
        return isTrue ? It->second.first : It->second.second;
    return V;
  }

  auto *BO = cast<BinaryOperator>(SI.getI());
  assert((BO->getOpcode() == Instruction::Add ||
          BO->getOpcode() == Instruction::Or ||
          BO->getOpcode() == Instruction::Sub) &&
         "Only currently handling Add, Or and Sub binary operators.");

  auto *CBO = BO->clone();
````
- **L481 EN**: Comment documents: `further by replacing it with the corresponding value on the given branch`.
  **L481 CN**: 注释说明：`further by replacing it with the corresponding value on the given branch`。
- **L482 EN**: Continues logic with `static Value *getTrueOrFalseValue(`.
  **L482 CN**: 继续处理逻辑：`static Value *getTrueOrFalseValue(`。
- **L483 EN**: Continues logic with `SelectOptimizeImpl::SelectLike &SI, bool isTrue,`.
  **L483 CN**: 继续处理逻辑：`SelectOptimizeImpl::SelectLike &SI, bool isTrue,`。
- **L484 EN**: Continues logic with `SmallDenseMap<Instruction *, std::pair<Value *, Value *>, 2> &OptSelects…`.
  **L484 CN**: 继续处理逻辑：`SmallDenseMap<Instruction *, std::pair<Value *, Value *>, 2> &OptSelects…`。
- **L485 EN**: Starts block `BasicBlock *B)`.
  **L485 CN**: 开始代码块 `BasicBlock *B)`。
- **L486 EN**: Assigns or initializes `Value *V`.
  **L486 CN**: 对 `Value *V` 进行赋值或初始化。
- **L487 EN**: Begins a conditional branch.
  **L487 CN**: 开始一个条件分支。
- **L488 EN**: Begins a conditional branch.
  **L488 CN**: 开始一个条件分支。
- **L489 EN**: Begins a conditional branch.
  **L489 CN**: 开始一个条件分支。
- **L490 EN**: Returns `isTrue ? It->second.first : It->second.second` to the caller.
  **L490 CN**: 向调用者返回 `isTrue ? It->second.first : It->second.second`。
- **L491 EN**: Returns `V` to the caller.
  **L491 CN**: 向调用者返回 `V`。
- **L492 EN**: Closes the current scope.
  **L492 CN**: 关闭当前作用域。
- **L493 EN**: Separates nearby statements for readability.
  **L493 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L494 EN**: Assigns or initializes `auto *BO`.
  **L494 CN**: 对 `auto *BO` 进行赋值或初始化。
- **L495 EN**: Checks an invariant in debug builds.
  **L495 CN**: 在调试构建中检查一个不变量。
- **L496 EN**: Continues logic with `BO->getOpcode() == Instruction::Or ||`.
  **L496 CN**: 继续处理逻辑：`BO->getOpcode() == Instruction::Or ||`。
- **L497 EN**: Continues logic with `BO->getOpcode() == Instruction::Sub) &&`.
  **L497 CN**: 继续处理逻辑：`BO->getOpcode() == Instruction::Sub) &&`。
- **L498 EN**: Executes statement `"Only currently handling Add, Or and Sub binary operators.");`.
  **L498 CN**: 执行语句 `"Only currently handling Add, Or and Sub binary operators.");`。
- **L499 EN**: Separates nearby statements for readability.
  **L499 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L500 EN**: Assigns or initializes `auto *CBO`.
  **L500 CN**: 对 `auto *CBO` 进行赋值或初始化。

### Lines 501-520

````cpp
  auto CondIdx = SI.getConditionOpIndex();
  auto *AuxI = cast<Instruction>(CBO->getOperand(CondIdx));
  if (isa<ZExtInst>(AuxI) || isa<LShrOperator>(AuxI)) {
    CBO->setOperand(CondIdx, ConstantInt::get(CBO->getType(), 1));
  } else {
    assert((isa<AShrOperator>(AuxI) || isa<SExtInst>(AuxI)) &&
           "Unexpected opcode");
    CBO->setOperand(CondIdx, ConstantInt::getAllOnesValue(CBO->getType()));
  }

  unsigned OtherIdx = 1 - CondIdx;
  if (auto *IV = dyn_cast<Instruction>(CBO->getOperand(OtherIdx))) {
    if (auto It = OptSelects.find(IV); It != OptSelects.end())
      CBO->setOperand(OtherIdx, isTrue ? It->second.first : It->second.second);
  }
  CBO->insertBefore(B->getTerminator()->getIterator());
  return CBO;
}

void SelectOptimizeImpl::convertProfitableSIGroups(SelectGroups &ProfSIGroups) {
````
- **L501 EN**: Assigns or initializes `auto CondIdx`.
  **L501 CN**: 对 `auto CondIdx` 进行赋值或初始化。
- **L502 EN**: Assigns or initializes `auto *AuxI`.
  **L502 CN**: 对 `auto *AuxI` 进行赋值或初始化。
- **L503 EN**: Begins a conditional branch.
  **L503 CN**: 开始一个条件分支。
- **L504 EN**: Declares function or method `setOperand`.
  **L504 CN**: 声明函数或方法 `setOperand`。
- **L505 EN**: Starts block `} else`.
  **L505 CN**: 开始代码块 `} else`。
- **L506 EN**: Checks an invariant in debug builds.
  **L506 CN**: 在调试构建中检查一个不变量。
- **L507 EN**: Executes statement `"Unexpected opcode");`.
  **L507 CN**: 执行语句 `"Unexpected opcode");`。
- **L508 EN**: Declares function or method `setOperand`.
  **L508 CN**: 声明函数或方法 `setOperand`。
- **L509 EN**: Closes the current scope.
  **L509 CN**: 关闭当前作用域。
- **L510 EN**: Separates nearby statements for readability.
  **L510 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L511 EN**: Assigns or initializes `unsigned OtherIdx`.
  **L511 CN**: 对 `unsigned OtherIdx` 进行赋值或初始化。
- **L512 EN**: Begins a conditional branch.
  **L512 CN**: 开始一个条件分支。
- **L513 EN**: Begins a conditional branch.
  **L513 CN**: 开始一个条件分支。
- **L514 EN**: Executes statement `CBO->setOperand(OtherIdx, isTrue ? It->second.first : It->second.second)…`.
  **L514 CN**: 执行语句 `CBO->setOperand(OtherIdx, isTrue ? It->second.first : It->second.second)…`。
- **L515 EN**: Closes the current scope.
  **L515 CN**: 关闭当前作用域。
- **L516 EN**: Executes statement `CBO->insertBefore(B->getTerminator()->getIterator());`.
  **L516 CN**: 执行语句 `CBO->insertBefore(B->getTerminator()->getIterator());`。
- **L517 EN**: Returns `CBO` to the caller.
  **L517 CN**: 向调用者返回 `CBO`。
- **L518 EN**: Closes the current scope.
  **L518 CN**: 关闭当前作用域。
- **L519 EN**: Separates nearby statements for readability.
  **L519 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L520 EN**: Begins the definition of `convertProfitableSIGroups`.
  **L520 CN**: 开始定义 `convertProfitableSIGroups`。

### Lines 521-540

````cpp
  for (SelectGroup &ASI : ProfSIGroups) {
    // The code transformation here is a modified version of the sinking
    // transformation in CodeGenPrepare::optimizeSelectInst with a more
    // aggressive strategy of which instructions to sink.
    //
    // TODO: eliminate the redundancy of logic transforming selects to branches
    // by removing CodeGenPrepare::optimizeSelectInst and optimizing here
    // selects for all cases (with and without profile information).

    // Transform a sequence like this:
    //    start:
    //       %cmp = cmp uge i32 %a, %b
    //       %sel = select i1 %cmp, i32 %c, i32 %d
    //
    // Into:
    //    start:
    //       %cmp = cmp uge i32 %a, %b
    //       %cmp.frozen = freeze %cmp
    //       br i1 %cmp.frozen, label %select.true, label %select.false
    //    select.true:
````
- **L521 EN**: Starts a loop over a sequence or range.
  **L521 CN**: 开始遍历序列或范围的循环。
- **L522 EN**: Comment documents: `The code transformation here is a modified version of the sinking`.
  **L522 CN**: 注释说明：`The code transformation here is a modified version of the sinking`。
- **L523 EN**: Comment documents: `transformation in CodeGenPrepare::optimizeSelectInst with a more`.
  **L523 CN**: 注释说明：`transformation in CodeGenPrepare::optimizeSelectInst with a more`。
- **L524 EN**: Comment documents: `aggressive strategy of which instructions to sink.`.
  **L524 CN**: 注释说明：`aggressive strategy of which instructions to sink.`。
- **L525 EN**: Continues the surrounding comment block.
  **L525 CN**: 延续周围的注释块。
- **L526 EN**: Comment documents: `TODO: eliminate the redundancy of logic transforming selects to branches`.
  **L526 CN**: 注释说明：`TODO: eliminate the redundancy of logic transforming selects to branches`。
- **L527 EN**: Comment documents: `by removing CodeGenPrepare::optimizeSelectInst and optimizing here`.
  **L527 CN**: 注释说明：`by removing CodeGenPrepare::optimizeSelectInst and optimizing here`。
- **L528 EN**: Comment documents: `selects for all cases (with and without profile information).`.
  **L528 CN**: 注释说明：`selects for all cases (with and without profile information).`。
- **L529 EN**: Separates nearby statements for readability.
  **L529 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L530 EN**: Comment documents: `Transform a sequence like this:`.
  **L530 CN**: 注释说明：`Transform a sequence like this:`。
- **L531 EN**: Comment documents: `start:`.
  **L531 CN**: 注释说明：`start:`。
- **L532 EN**: Comment documents: `%cmp = cmp uge i32 %a, %b`.
  **L532 CN**: 注释说明：`%cmp = cmp uge i32 %a, %b`。
- **L533 EN**: Comment documents: `%sel = select i1 %cmp, i32 %c, i32 %d`.
  **L533 CN**: 注释说明：`%sel = select i1 %cmp, i32 %c, i32 %d`。
- **L534 EN**: Continues the surrounding comment block.
  **L534 CN**: 延续周围的注释块。
- **L535 EN**: Comment documents: `Into:`.
  **L535 CN**: 注释说明：`Into:`。
- **L536 EN**: Comment documents: `start:`.
  **L536 CN**: 注释说明：`start:`。
- **L537 EN**: Comment documents: `%cmp = cmp uge i32 %a, %b`.
  **L537 CN**: 注释说明：`%cmp = cmp uge i32 %a, %b`。
- **L538 EN**: Comment documents: `%cmp.frozen = freeze %cmp`.
  **L538 CN**: 注释说明：`%cmp.frozen = freeze %cmp`。
- **L539 EN**: Comment documents: `br i1 %cmp.frozen, label %select.true, label %select.false`.
  **L539 CN**: 注释说明：`br i1 %cmp.frozen, label %select.true, label %select.false`。
- **L540 EN**: Comment documents: `select.true:`.
  **L540 CN**: 注释说明：`select.true:`。

### Lines 541-560

````cpp
    //       br label %select.end
    //    select.false:
    //       br label %select.end
    //    select.end:
    //       %sel = phi i32 [ %c, %select.true ], [ %d, %select.false ]
    //
    // %cmp should be frozen, otherwise it may introduce undefined behavior.
    // In addition, we may sink instructions that produce %c or %d into the
    // destination(s) of the new branch.
    // If the true or false blocks do not contain a sunken instruction, that
    // block and its branch may be optimized away. In that case, one side of the
    // first branch will point directly to select.end, and the corresponding PHI
    // predecessor block will be the start block.

    // Find all the instructions that can be soundly sunk to the true/false
    // blocks. These are instructions that are computed solely for producing the
    // operands of the select instructions in the group and can be sunk without
    // breaking the semantics of the LLVM IR (e.g., cannot sink instructions
    // with side effects).
    SmallVector<std::stack<Instruction *>, 2> TrueSlices, FalseSlices;
````
- **L541 EN**: Comment documents: `br label %select.end`.
  **L541 CN**: 注释说明：`br label %select.end`。
- **L542 EN**: Comment documents: `select.false:`.
  **L542 CN**: 注释说明：`select.false:`。
- **L543 EN**: Comment documents: `br label %select.end`.
  **L543 CN**: 注释说明：`br label %select.end`。
- **L544 EN**: Comment documents: `select.end:`.
  **L544 CN**: 注释说明：`select.end:`。
- **L545 EN**: Comment documents: `%sel = phi i32 [ %c, %select.true ], [ %d, %select.false ]`.
  **L545 CN**: 注释说明：`%sel = phi i32 [ %c, %select.true ], [ %d, %select.false ]`。
- **L546 EN**: Continues the surrounding comment block.
  **L546 CN**: 延续周围的注释块。
- **L547 EN**: Comment documents: `%cmp should be frozen, otherwise it may introduce undefined behavior.`.
  **L547 CN**: 注释说明：`%cmp should be frozen, otherwise it may introduce undefined behavior.`。
- **L548 EN**: Comment documents: `In addition, we may sink instructions that produce %c or %d into the`.
  **L548 CN**: 注释说明：`In addition, we may sink instructions that produce %c or %d into the`。
- **L549 EN**: Comment documents: `destination(s) of the new branch.`.
  **L549 CN**: 注释说明：`destination(s) of the new branch.`。
- **L550 EN**: Comment documents: `If the true or false blocks do not contain a sunken instruction, that`.
  **L550 CN**: 注释说明：`If the true or false blocks do not contain a sunken instruction, that`。
- **L551 EN**: Comment documents: `block and its branch may be optimized away. In that case, one side of th…`.
  **L551 CN**: 注释说明：`block and its branch may be optimized away. In that case, one side of th…`。
- **L552 EN**: Comment documents: `first branch will point directly to select.end, and the corresponding PH…`.
  **L552 CN**: 注释说明：`first branch will point directly to select.end, and the corresponding PH…`。
- **L553 EN**: Comment documents: `predecessor block will be the start block.`.
  **L553 CN**: 注释说明：`predecessor block will be the start block.`。
- **L554 EN**: Separates nearby statements for readability.
  **L554 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L555 EN**: Comment documents: `Find all the instructions that can be soundly sunk to the true/false`.
  **L555 CN**: 注释说明：`Find all the instructions that can be soundly sunk to the true/false`。
- **L556 EN**: Comment documents: `blocks. These are instructions that are computed solely for producing th…`.
  **L556 CN**: 注释说明：`blocks. These are instructions that are computed solely for producing th…`。
- **L557 EN**: Comment documents: `operands of the select instructions in the group and can be sunk without`.
  **L557 CN**: 注释说明：`operands of the select instructions in the group and can be sunk without`。
- **L558 EN**: Comment documents: `breaking the semantics of the LLVM IR (e.g., cannot sink instructions`.
  **L558 CN**: 注释说明：`breaking the semantics of the LLVM IR (e.g., cannot sink instructions`。
- **L559 EN**: Comment documents: `with side effects).`.
  **L559 CN**: 注释说明：`with side effects).`。
- **L560 EN**: Executes statement `SmallVector<std::stack<Instruction *>, 2> TrueSlices, FalseSlices;`.
  **L560 CN**: 执行语句 `SmallVector<std::stack<Instruction *>, 2> TrueSlices, FalseSlices;`。

### Lines 561-580

````cpp
    typedef std::stack<Instruction *>::size_type StackSizeType;
    StackSizeType maxTrueSliceLen = 0, maxFalseSliceLen = 0;
    for (SelectLike &SI : ASI.Selects) {
      if (!isa<SelectInst>(SI.getI()))
        continue;
      // For each select, compute the sinkable dependence chains of the true and
      // false operands.
      if (auto *TI = dyn_cast_or_null<Instruction>(SI.getTrueValue())) {
        std::stack<Instruction *> TrueSlice;
        getExclBackwardsSlice(TI, TrueSlice, SI.getI(), true);
        maxTrueSliceLen = std::max(maxTrueSliceLen, TrueSlice.size());
        TrueSlices.push_back(TrueSlice);
      }
      if (auto *FI = dyn_cast_or_null<Instruction>(SI.getFalseValue())) {
        if (isa<SelectInst>(SI.getI()) || !FI->hasOneUse()) {
          std::stack<Instruction *> FalseSlice;
          getExclBackwardsSlice(FI, FalseSlice, SI.getI(), true);
          maxFalseSliceLen = std::max(maxFalseSliceLen, FalseSlice.size());
          FalseSlices.push_back(FalseSlice);
        }
````
- **L561 EN**: Executes statement `typedef std::stack<Instruction *>::size_type StackSizeType;`.
  **L561 CN**: 执行语句 `typedef std::stack<Instruction *>::size_type StackSizeType;`。
- **L562 EN**: Assigns or initializes `StackSizeType maxTrueSliceLen`.
  **L562 CN**: 对 `StackSizeType maxTrueSliceLen` 进行赋值或初始化。
- **L563 EN**: Starts a loop over a sequence or range.
  **L563 CN**: 开始遍历序列或范围的循环。
- **L564 EN**: Begins a conditional branch.
  **L564 CN**: 开始一个条件分支。
- **L565 EN**: Skips to the next loop iteration.
  **L565 CN**: 跳到下一次循环迭代。
- **L566 EN**: Comment documents: `For each select, compute the sinkable dependence chains of the true and`.
  **L566 CN**: 注释说明：`For each select, compute the sinkable dependence chains of the true and`。
- **L567 EN**: Comment documents: `false operands.`.
  **L567 CN**: 注释说明：`false operands.`。
- **L568 EN**: Begins a conditional branch.
  **L568 CN**: 开始一个条件分支。
- **L569 EN**: Executes statement `std::stack<Instruction *> TrueSlice;`.
  **L569 CN**: 执行语句 `std::stack<Instruction *> TrueSlice;`。
- **L570 EN**: Executes statement `getExclBackwardsSlice(TI, TrueSlice, SI.getI(), true);`.
  **L570 CN**: 执行语句 `getExclBackwardsSlice(TI, TrueSlice, SI.getI(), true);`。
- **L571 EN**: Declares function or method `max`.
  **L571 CN**: 声明函数或方法 `max`。
- **L572 EN**: Executes statement `TrueSlices.push_back(TrueSlice);`.
  **L572 CN**: 执行语句 `TrueSlices.push_back(TrueSlice);`。
- **L573 EN**: Closes the current scope.
  **L573 CN**: 关闭当前作用域。
- **L574 EN**: Begins a conditional branch.
  **L574 CN**: 开始一个条件分支。
- **L575 EN**: Begins a conditional branch.
  **L575 CN**: 开始一个条件分支。
- **L576 EN**: Executes statement `std::stack<Instruction *> FalseSlice;`.
  **L576 CN**: 执行语句 `std::stack<Instruction *> FalseSlice;`。
- **L577 EN**: Executes statement `getExclBackwardsSlice(FI, FalseSlice, SI.getI(), true);`.
  **L577 CN**: 执行语句 `getExclBackwardsSlice(FI, FalseSlice, SI.getI(), true);`。
- **L578 EN**: Declares function or method `max`.
  **L578 CN**: 声明函数或方法 `max`。
- **L579 EN**: Executes statement `FalseSlices.push_back(FalseSlice);`.
  **L579 CN**: 执行语句 `FalseSlices.push_back(FalseSlice);`。
- **L580 EN**: Closes the current scope.
  **L580 CN**: 关闭当前作用域。

### Lines 581-600

````cpp
      }
    }
    // In the case of multiple select instructions in the same group, the order
    // of non-dependent instructions (instructions of different dependence
    // slices) in the true/false blocks appears to affect performance.
    // Interleaving the slices seems to experimentally be the optimal approach.
    // This interleaving scheduling allows for more ILP (with a natural downside
    // of increasing a bit register pressure) compared to a simple ordering of
    // one whole chain after another. One would expect that this ordering would
    // not matter since the scheduling in the backend of the compiler  would
    // take care of it, but apparently the scheduler fails to deliver optimal
    // ILP with a naive ordering here.
    SmallVector<Instruction *, 2> TrueSlicesInterleaved, FalseSlicesInterleaved;
    for (StackSizeType IS = 0; IS < maxTrueSliceLen; ++IS) {
      for (auto &S : TrueSlices) {
        if (!S.empty()) {
          TrueSlicesInterleaved.push_back(S.top());
          S.pop();
        }
      }
````
- **L581 EN**: Closes the current scope.
  **L581 CN**: 关闭当前作用域。
- **L582 EN**: Closes the current scope.
  **L582 CN**: 关闭当前作用域。
- **L583 EN**: Comment documents: `In the case of multiple select instructions in the same group, the order`.
  **L583 CN**: 注释说明：`In the case of multiple select instructions in the same group, the order`。
- **L584 EN**: Comment documents: `of non-dependent instructions (instructions of different dependence`.
  **L584 CN**: 注释说明：`of non-dependent instructions (instructions of different dependence`。
- **L585 EN**: Comment documents: `slices) in the true/false blocks appears to affect performance.`.
  **L585 CN**: 注释说明：`slices) in the true/false blocks appears to affect performance.`。
- **L586 EN**: Comment documents: `Interleaving the slices seems to experimentally be the optimal approach.`.
  **L586 CN**: 注释说明：`Interleaving the slices seems to experimentally be the optimal approach.`。
- **L587 EN**: Comment documents: `This interleaving scheduling allows for more ILP (with a natural downsid…`.
  **L587 CN**: 注释说明：`This interleaving scheduling allows for more ILP (with a natural downsid…`。
- **L588 EN**: Comment documents: `of increasing a bit register pressure) compared to a simple ordering of`.
  **L588 CN**: 注释说明：`of increasing a bit register pressure) compared to a simple ordering of`。
- **L589 EN**: Comment documents: `one whole chain after another. One would expect that this ordering would`.
  **L589 CN**: 注释说明：`one whole chain after another. One would expect that this ordering would`。
- **L590 EN**: Comment documents: `not matter since the scheduling in the backend of the compiler would`.
  **L590 CN**: 注释说明：`not matter since the scheduling in the backend of the compiler would`。
- **L591 EN**: Comment documents: `take care of it, but apparently the scheduler fails to deliver optimal`.
  **L591 CN**: 注释说明：`take care of it, but apparently the scheduler fails to deliver optimal`。
- **L592 EN**: Comment documents: `ILP with a naive ordering here.`.
  **L592 CN**: 注释说明：`ILP with a naive ordering here.`。
- **L593 EN**: Executes statement `SmallVector<Instruction *, 2> TrueSlicesInterleaved, FalseSlicesInterlea…`.
  **L593 CN**: 执行语句 `SmallVector<Instruction *, 2> TrueSlicesInterleaved, FalseSlicesInterlea…`。
- **L594 EN**: Starts a loop over a sequence or range.
  **L594 CN**: 开始遍历序列或范围的循环。
- **L595 EN**: Starts a loop over a sequence or range.
  **L595 CN**: 开始遍历序列或范围的循环。
- **L596 EN**: Begins a conditional branch.
  **L596 CN**: 开始一个条件分支。
- **L597 EN**: Executes statement `TrueSlicesInterleaved.push_back(S.top());`.
  **L597 CN**: 执行语句 `TrueSlicesInterleaved.push_back(S.top());`。
- **L598 EN**: Executes statement `S.pop();`.
  **L598 CN**: 执行语句 `S.pop();`。
- **L599 EN**: Closes the current scope.
  **L599 CN**: 关闭当前作用域。
- **L600 EN**: Closes the current scope.
  **L600 CN**: 关闭当前作用域。

### Lines 601-620

````cpp
    }
    for (StackSizeType IS = 0; IS < maxFalseSliceLen; ++IS) {
      for (auto &S : FalseSlices) {
        if (!S.empty()) {
          FalseSlicesInterleaved.push_back(S.top());
          S.pop();
        }
      }
    }

    // We split the block containing the select(s) into two blocks.
    SelectLike &SI = ASI.Selects.front();
    SelectLike &LastSI = ASI.Selects.back();
    BasicBlock *StartBlock = SI.getI()->getParent();
    BasicBlock::iterator SplitPt = ++(BasicBlock::iterator(LastSI.getI()));
    // With RemoveDIs turned off, SplitPt can be a dbg.* intrinsic. With
    // RemoveDIs turned on, SplitPt would instead point to the next
    // instruction. To match existing dbg.* intrinsic behaviour with RemoveDIs,
    // tell splitBasicBlock that we want to include any DbgVariableRecords
    // attached to SplitPt in the splice.
````
- **L601 EN**: Closes the current scope.
  **L601 CN**: 关闭当前作用域。
- **L602 EN**: Starts a loop over a sequence or range.
  **L602 CN**: 开始遍历序列或范围的循环。
- **L603 EN**: Starts a loop over a sequence or range.
  **L603 CN**: 开始遍历序列或范围的循环。
- **L604 EN**: Begins a conditional branch.
  **L604 CN**: 开始一个条件分支。
- **L605 EN**: Executes statement `FalseSlicesInterleaved.push_back(S.top());`.
  **L605 CN**: 执行语句 `FalseSlicesInterleaved.push_back(S.top());`。
- **L606 EN**: Executes statement `S.pop();`.
  **L606 CN**: 执行语句 `S.pop();`。
- **L607 EN**: Closes the current scope.
  **L607 CN**: 关闭当前作用域。
- **L608 EN**: Closes the current scope.
  **L608 CN**: 关闭当前作用域。
- **L609 EN**: Closes the current scope.
  **L609 CN**: 关闭当前作用域。
- **L610 EN**: Separates nearby statements for readability.
  **L610 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L611 EN**: Comment documents: `We split the block containing the select(s) into two blocks.`.
  **L611 CN**: 注释说明：`We split the block containing the select(s) into two blocks.`。
- **L612 EN**: Assigns or initializes `SelectLike &SI`.
  **L612 CN**: 对 `SelectLike &SI` 进行赋值或初始化。
- **L613 EN**: Assigns or initializes `SelectLike &LastSI`.
  **L613 CN**: 对 `SelectLike &LastSI` 进行赋值或初始化。
- **L614 EN**: Assigns or initializes `BasicBlock *StartBlock`.
  **L614 CN**: 对 `BasicBlock *StartBlock` 进行赋值或初始化。
- **L615 EN**: Declares function or method `iterator`.
  **L615 CN**: 声明函数或方法 `iterator`。
- **L616 EN**: Comment documents: `With RemoveDIs turned off, SplitPt can be a dbg.* intrinsic. With`.
  **L616 CN**: 注释说明：`With RemoveDIs turned off, SplitPt can be a dbg.* intrinsic. With`。
- **L617 EN**: Comment documents: `RemoveDIs turned on, SplitPt would instead point to the next`.
  **L617 CN**: 注释说明：`RemoveDIs turned on, SplitPt would instead point to the next`。
- **L618 EN**: Comment documents: `instruction. To match existing dbg.* intrinsic behaviour with RemoveDIs,`.
  **L618 CN**: 注释说明：`instruction. To match existing dbg.* intrinsic behaviour with RemoveDIs,`。
- **L619 EN**: Comment documents: `tell splitBasicBlock that we want to include any DbgVariableRecords`.
  **L619 CN**: 注释说明：`tell splitBasicBlock that we want to include any DbgVariableRecords`。
- **L620 EN**: Comment documents: `attached to SplitPt in the splice.`.
  **L620 CN**: 注释说明：`attached to SplitPt in the splice.`。

### Lines 621-640

````cpp
    SplitPt.setHeadBit(true);
    BasicBlock *EndBlock = StartBlock->splitBasicBlock(SplitPt, "select.end");
    BFI->setBlockFreq(EndBlock, BFI->getBlockFreq(StartBlock));
    // Delete the unconditional branch that was just created by the split.
    StartBlock->getTerminator()->eraseFromParent();

    // Move any debug/pseudo and auxiliary instructions that were in-between the
    // select group to the newly-created end block.
    SmallVector<Instruction *, 2> SinkInstrs;
    auto DIt = SI.getI()->getIterator();
    auto NIt = ASI.Selects.begin();
    while (&*DIt != LastSI.getI()) {
      if (NIt != ASI.Selects.end() && &*DIt == NIt->getI())
        ++NIt;
      else
        SinkInstrs.push_back(&*DIt);
      DIt++;
    }
    auto InsertionPoint = EndBlock->getFirstInsertionPt();
    for (auto *DI : SinkInstrs)
````
- **L621 EN**: Executes statement `SplitPt.setHeadBit(true);`.
  **L621 CN**: 执行语句 `SplitPt.setHeadBit(true);`。
- **L622 EN**: Assigns or initializes `BasicBlock *EndBlock`.
  **L622 CN**: 对 `BasicBlock *EndBlock` 进行赋值或初始化。
- **L623 EN**: Executes statement `BFI->setBlockFreq(EndBlock, BFI->getBlockFreq(StartBlock));`.
  **L623 CN**: 执行语句 `BFI->setBlockFreq(EndBlock, BFI->getBlockFreq(StartBlock));`。
- **L624 EN**: Comment documents: `Delete the unconditional branch that was just created by the split.`.
  **L624 CN**: 注释说明：`Delete the unconditional branch that was just created by the split.`。
- **L625 EN**: Executes statement `StartBlock->getTerminator()->eraseFromParent();`.
  **L625 CN**: 执行语句 `StartBlock->getTerminator()->eraseFromParent();`。
- **L626 EN**: Separates nearby statements for readability.
  **L626 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L627 EN**: Comment documents: `Move any debug/pseudo and auxiliary instructions that were in-between th…`.
  **L627 CN**: 注释说明：`Move any debug/pseudo and auxiliary instructions that were in-between th…`。
- **L628 EN**: Comment documents: `select group to the newly-created end block.`.
  **L628 CN**: 注释说明：`select group to the newly-created end block.`。
- **L629 EN**: Executes statement `SmallVector<Instruction *, 2> SinkInstrs;`.
  **L629 CN**: 执行语句 `SmallVector<Instruction *, 2> SinkInstrs;`。
- **L630 EN**: Assigns or initializes `auto DIt`.
  **L630 CN**: 对 `auto DIt` 进行赋值或初始化。
- **L631 EN**: Assigns or initializes `auto NIt`.
  **L631 CN**: 对 `auto NIt` 进行赋值或初始化。
- **L632 EN**: Starts a while loop controlled by a condition.
  **L632 CN**: 开始一个由条件控制的 while 循环。
- **L633 EN**: Begins a conditional branch.
  **L633 CN**: 开始一个条件分支。
- **L634 EN**: Executes statement `++NIt;`.
  **L634 CN**: 执行语句 `++NIt;`。
- **L635 EN**: Handles the fallback branch.
  **L635 CN**: 处理兜底分支。
- **L636 EN**: Executes statement `SinkInstrs.push_back(&*DIt);`.
  **L636 CN**: 执行语句 `SinkInstrs.push_back(&*DIt);`。
- **L637 EN**: Executes statement `DIt++;`.
  **L637 CN**: 执行语句 `DIt++;`。
- **L638 EN**: Closes the current scope.
  **L638 CN**: 关闭当前作用域。
- **L639 EN**: Assigns or initializes `auto InsertionPoint`.
  **L639 CN**: 对 `auto InsertionPoint` 进行赋值或初始化。
- **L640 EN**: Starts a loop over a sequence or range.
  **L640 CN**: 开始遍历序列或范围的循环。

### Lines 641-660

````cpp
      DI->moveBeforePreserving(InsertionPoint);

    // Duplicate implementation for DbgRecords, the non-instruction debug-info
    // format. Helper lambda for moving DbgRecords to the end block.
    auto TransferDbgRecords = [&](Instruction &I) {
      for (auto &DbgRecord :
           llvm::make_early_inc_range(I.getDbgRecordRange())) {
        DbgRecord.removeFromParent();
        EndBlock->insertDbgRecordBefore(&DbgRecord,
                                        EndBlock->getFirstInsertionPt());
      }
    };

    // Iterate over all instructions in between SI and LastSI, not including
    // SI itself. These are all the variable assignments that happen "in the
    // middle" of the select group.
    auto R = make_range(std::next(SI.getI()->getIterator()),
                        std::next(LastSI.getI()->getIterator()));
    llvm::for_each(R, TransferDbgRecords);

````
- **L641 EN**: Executes statement `DI->moveBeforePreserving(InsertionPoint);`.
  **L641 CN**: 执行语句 `DI->moveBeforePreserving(InsertionPoint);`。
- **L642 EN**: Separates nearby statements for readability.
  **L642 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L643 EN**: Comment documents: `Duplicate implementation for DbgRecords, the non-instruction debug-info`.
  **L643 CN**: 注释说明：`Duplicate implementation for DbgRecords, the non-instruction debug-info`。
- **L644 EN**: Comment documents: `format. Helper lambda for moving DbgRecords to the end block.`.
  **L644 CN**: 注释说明：`format. Helper lambda for moving DbgRecords to the end block.`。
- **L645 EN**: Starts block `auto TransferDbgRecords = [&](Instruction &I)`.
  **L645 CN**: 开始代码块 `auto TransferDbgRecords = [&](Instruction &I)`。
- **L646 EN**: Starts a loop over a sequence or range.
  **L646 CN**: 开始遍历序列或范围的循环。
- **L647 EN**: Begins the definition of `make_early_inc_range`.
  **L647 CN**: 开始定义 `make_early_inc_range`。
- **L648 EN**: Executes statement `DbgRecord.removeFromParent();`.
  **L648 CN**: 执行语句 `DbgRecord.removeFromParent();`。
- **L649 EN**: Continues logic with `EndBlock->insertDbgRecordBefore(&DbgRecord,`.
  **L649 CN**: 继续处理逻辑：`EndBlock->insertDbgRecordBefore(&DbgRecord,`。
- **L650 EN**: Executes statement `EndBlock->getFirstInsertionPt());`.
  **L650 CN**: 执行语句 `EndBlock->getFirstInsertionPt());`。
- **L651 EN**: Closes the current scope.
  **L651 CN**: 关闭当前作用域。
- **L652 EN**: Closes the current scope.
  **L652 CN**: 关闭当前作用域。
- **L653 EN**: Separates nearby statements for readability.
  **L653 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L654 EN**: Comment documents: `Iterate over all instructions in between SI and LastSI, not including`.
  **L654 CN**: 注释说明：`Iterate over all instructions in between SI and LastSI, not including`。
- **L655 EN**: Comment documents: `SI itself. These are all the variable assignments that happen "in the`.
  **L655 CN**: 注释说明：`SI itself. These are all the variable assignments that happen "in the`。
- **L656 EN**: Comment documents: `middle" of the select group.`.
  **L656 CN**: 注释说明：`middle" of the select group.`。
- **L657 EN**: Provides part of the signature for `make_range`.
  **L657 CN**: 给出 `make_range` 的一部分签名。
- **L658 EN**: Declares function or method `next`.
  **L658 CN**: 声明函数或方法 `next`。
- **L659 EN**: Declares function or method `for_each`.
  **L659 CN**: 声明函数或方法 `for_each`。
- **L660 EN**: Separates nearby statements for readability.
  **L660 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 661-680

````cpp
    // These are the new basic blocks for the conditional branch.
    // At least one will become an actual new basic block.
    BasicBlock *TrueBlock = nullptr, *FalseBlock = nullptr;
    UncondBrInst *TrueBranch = nullptr, *FalseBranch = nullptr;
    // Checks if select-like instruction would materialise on the given branch
    auto HasSelectLike = [](SelectGroup &SG, bool IsTrue) {
      for (auto &SL : SG.Selects) {
        if ((IsTrue ? SL.getTrueValue() : SL.getFalseValue()) == nullptr)
          return true;
      }
      return false;
    };
    if (!TrueSlicesInterleaved.empty() || HasSelectLike(ASI, true)) {
      TrueBlock = BasicBlock::Create(EndBlock->getContext(), "select.true.sink",
                                     EndBlock->getParent(), EndBlock);
      TrueBranch = UncondBrInst::Create(EndBlock, TrueBlock);
      TrueBranch->setDebugLoc(LastSI.getI()->getDebugLoc());
      for (Instruction *TrueInst : TrueSlicesInterleaved)
        TrueInst->moveBefore(TrueBranch->getIterator());
    }
````
- **L661 EN**: Comment documents: `These are the new basic blocks for the conditional branch.`.
  **L661 CN**: 注释说明：`These are the new basic blocks for the conditional branch.`。
- **L662 EN**: Comment documents: `At least one will become an actual new basic block.`.
  **L662 CN**: 注释说明：`At least one will become an actual new basic block.`。
- **L663 EN**: Assigns or initializes `BasicBlock *TrueBlock`.
  **L663 CN**: 对 `BasicBlock *TrueBlock` 进行赋值或初始化。
- **L664 EN**: Assigns or initializes `UncondBrInst *TrueBranch`.
  **L664 CN**: 对 `UncondBrInst *TrueBranch` 进行赋值或初始化。
- **L665 EN**: Comment documents: `Checks if select-like instruction would materialise on the given branch`.
  **L665 CN**: 注释说明：`Checks if select-like instruction would materialise on the given branch`。
- **L666 EN**: Starts block `auto HasSelectLike = [](SelectGroup &SG, bool IsTrue)`.
  **L666 CN**: 开始代码块 `auto HasSelectLike = [](SelectGroup &SG, bool IsTrue)`。
- **L667 EN**: Starts a loop over a sequence or range.
  **L667 CN**: 开始遍历序列或范围的循环。
- **L668 EN**: Begins a conditional branch.
  **L668 CN**: 开始一个条件分支。
- **L669 EN**: Returns `true` to the caller.
  **L669 CN**: 向调用者返回 `true`。
- **L670 EN**: Closes the current scope.
  **L670 CN**: 关闭当前作用域。
- **L671 EN**: Returns `false` to the caller.
  **L671 CN**: 向调用者返回 `false`。
- **L672 EN**: Closes the current scope.
  **L672 CN**: 关闭当前作用域。
- **L673 EN**: Begins a conditional branch.
  **L673 CN**: 开始一个条件分支。
- **L674 EN**: Provides part of the signature for `Create`.
  **L674 CN**: 给出 `Create` 的一部分签名。
- **L675 EN**: Executes statement `EndBlock->getParent(), EndBlock);`.
  **L675 CN**: 执行语句 `EndBlock->getParent(), EndBlock);`。
- **L676 EN**: Declares function or method `Create`.
  **L676 CN**: 声明函数或方法 `Create`。
- **L677 EN**: Executes statement `TrueBranch->setDebugLoc(LastSI.getI()->getDebugLoc());`.
  **L677 CN**: 执行语句 `TrueBranch->setDebugLoc(LastSI.getI()->getDebugLoc());`。
- **L678 EN**: Starts a loop over a sequence or range.
  **L678 CN**: 开始遍历序列或范围的循环。
- **L679 EN**: Executes statement `TrueInst->moveBefore(TrueBranch->getIterator());`.
  **L679 CN**: 执行语句 `TrueInst->moveBefore(TrueBranch->getIterator());`。
- **L680 EN**: Closes the current scope.
  **L680 CN**: 关闭当前作用域。

### Lines 681-700

````cpp
    if (!FalseSlicesInterleaved.empty() || HasSelectLike(ASI, false)) {
      FalseBlock =
          BasicBlock::Create(EndBlock->getContext(), "select.false.sink",
                             EndBlock->getParent(), EndBlock);
      FalseBranch = UncondBrInst::Create(EndBlock, FalseBlock);
      FalseBranch->setDebugLoc(LastSI.getI()->getDebugLoc());
      for (Instruction *FalseInst : FalseSlicesInterleaved)
        FalseInst->moveBefore(FalseBranch->getIterator());
    }
    // If there was nothing to sink, then arbitrarily choose the 'false' side
    // for a new input value to the PHI.
    if (TrueBlock == FalseBlock) {
      assert(TrueBlock == nullptr &&
             "Unexpected basic block transform while optimizing select");

      FalseBlock = BasicBlock::Create(StartBlock->getContext(), "select.false",
                                      EndBlock->getParent(), EndBlock);
      auto *FalseBranch = UncondBrInst::Create(EndBlock, FalseBlock);
      FalseBranch->setDebugLoc(SI.getI()->getDebugLoc());
    }
````
- **L681 EN**: Begins a conditional branch.
  **L681 CN**: 开始一个条件分支。
- **L682 EN**: Continues logic with `FalseBlock =`.
  **L682 CN**: 继续处理逻辑：`FalseBlock =`。
- **L683 EN**: Provides part of the signature for `Create`.
  **L683 CN**: 给出 `Create` 的一部分签名。
- **L684 EN**: Executes statement `EndBlock->getParent(), EndBlock);`.
  **L684 CN**: 执行语句 `EndBlock->getParent(), EndBlock);`。
- **L685 EN**: Declares function or method `Create`.
  **L685 CN**: 声明函数或方法 `Create`。
- **L686 EN**: Executes statement `FalseBranch->setDebugLoc(LastSI.getI()->getDebugLoc());`.
  **L686 CN**: 执行语句 `FalseBranch->setDebugLoc(LastSI.getI()->getDebugLoc());`。
- **L687 EN**: Starts a loop over a sequence or range.
  **L687 CN**: 开始遍历序列或范围的循环。
- **L688 EN**: Executes statement `FalseInst->moveBefore(FalseBranch->getIterator());`.
  **L688 CN**: 执行语句 `FalseInst->moveBefore(FalseBranch->getIterator());`。
- **L689 EN**: Closes the current scope.
  **L689 CN**: 关闭当前作用域。
- **L690 EN**: Comment documents: `If there was nothing to sink, then arbitrarily choose the 'false' side`.
  **L690 CN**: 注释说明：`If there was nothing to sink, then arbitrarily choose the 'false' side`。
- **L691 EN**: Comment documents: `for a new input value to the PHI.`.
  **L691 CN**: 注释说明：`for a new input value to the PHI.`。
- **L692 EN**: Begins a conditional branch.
  **L692 CN**: 开始一个条件分支。
- **L693 EN**: Checks an invariant in debug builds.
  **L693 CN**: 在调试构建中检查一个不变量。
- **L694 EN**: Executes statement `"Unexpected basic block transform while optimizing select");`.
  **L694 CN**: 执行语句 `"Unexpected basic block transform while optimizing select");`。
- **L695 EN**: Separates nearby statements for readability.
  **L695 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L696 EN**: Provides part of the signature for `Create`.
  **L696 CN**: 给出 `Create` 的一部分签名。
- **L697 EN**: Executes statement `EndBlock->getParent(), EndBlock);`.
  **L697 CN**: 执行语句 `EndBlock->getParent(), EndBlock);`。
- **L698 EN**: Declares function or method `Create`.
  **L698 CN**: 声明函数或方法 `Create`。
- **L699 EN**: Executes statement `FalseBranch->setDebugLoc(SI.getI()->getDebugLoc());`.
  **L699 CN**: 执行语句 `FalseBranch->setDebugLoc(SI.getI()->getDebugLoc());`。
- **L700 EN**: Closes the current scope.
  **L700 CN**: 关闭当前作用域。

### Lines 701-720

````cpp

    // Insert the real conditional branch based on the original condition.
    // If we did not create a new block for one of the 'true' or 'false' paths
    // of the condition, it means that side of the branch goes to the end block
    // directly and the path originates from the start block from the point of
    // view of the new PHI.
    BasicBlock *TT, *FT;
    if (TrueBlock == nullptr) {
      TT = EndBlock;
      FT = FalseBlock;
      TrueBlock = StartBlock;
    } else if (FalseBlock == nullptr) {
      TT = TrueBlock;
      FT = EndBlock;
      FalseBlock = StartBlock;
    } else {
      TT = TrueBlock;
      FT = FalseBlock;
    }
    IRBuilder<> IB(SI.getI());
````
- **L701 EN**: Separates nearby statements for readability.
  **L701 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L702 EN**: Comment documents: `Insert the real conditional branch based on the original condition.`.
  **L702 CN**: 注释说明：`Insert the real conditional branch based on the original condition.`。
- **L703 EN**: Comment documents: `If we did not create a new block for one of the 'true' or 'false' paths`.
  **L703 CN**: 注释说明：`If we did not create a new block for one of the 'true' or 'false' paths`。
- **L704 EN**: Comment documents: `of the condition, it means that side of the branch goes to the end block`.
  **L704 CN**: 注释说明：`of the condition, it means that side of the branch goes to the end block`。
- **L705 EN**: Comment documents: `directly and the path originates from the start block from the point of`.
  **L705 CN**: 注释说明：`directly and the path originates from the start block from the point of`。
- **L706 EN**: Comment documents: `view of the new PHI.`.
  **L706 CN**: 注释说明：`view of the new PHI.`。
- **L707 EN**: Executes statement `BasicBlock *TT, *FT;`.
  **L707 CN**: 执行语句 `BasicBlock *TT, *FT;`。
- **L708 EN**: Begins a conditional branch.
  **L708 CN**: 开始一个条件分支。
- **L709 EN**: Assigns or initializes `TT`.
  **L709 CN**: 对 `TT` 进行赋值或初始化。
- **L710 EN**: Assigns or initializes `FT`.
  **L710 CN**: 对 `FT` 进行赋值或初始化。
- **L711 EN**: Assigns or initializes `TrueBlock`.
  **L711 CN**: 对 `TrueBlock` 进行赋值或初始化。
- **L712 EN**: Starts block `} else if (FalseBlock == nullptr)`.
  **L712 CN**: 开始代码块 `} else if (FalseBlock == nullptr)`。
- **L713 EN**: Assigns or initializes `TT`.
  **L713 CN**: 对 `TT` 进行赋值或初始化。
- **L714 EN**: Assigns or initializes `FT`.
  **L714 CN**: 对 `FT` 进行赋值或初始化。
- **L715 EN**: Assigns or initializes `FalseBlock`.
  **L715 CN**: 对 `FalseBlock` 进行赋值或初始化。
- **L716 EN**: Starts block `} else`.
  **L716 CN**: 开始代码块 `} else`。
- **L717 EN**: Assigns or initializes `TT`.
  **L717 CN**: 对 `TT` 进行赋值或初始化。
- **L718 EN**: Assigns or initializes `FT`.
  **L718 CN**: 对 `FT` 进行赋值或初始化。
- **L719 EN**: Closes the current scope.
  **L719 CN**: 关闭当前作用域。
- **L720 EN**: Declares function or method `IB`.
  **L720 CN**: 声明函数或方法 `IB`。

### Lines 721-740

````cpp
    auto *CondFr =
        IB.CreateFreeze(ASI.Condition, ASI.Condition->getName() + ".frozen");

    SmallDenseMap<Instruction *, std::pair<Value *, Value *>, 2> INS;

    // Use reverse iterator because later select may use the value of the
    // earlier select, and we need to propagate value through earlier select
    // to get the PHI operand.
    InsertionPoint = EndBlock->begin();
    for (SelectLike &SI : ASI.Selects) {
      // The select itself is replaced with a PHI Node.
      PHINode *PN = PHINode::Create(SI.getType(), 2, "");
      PN->insertBefore(InsertionPoint);
      PN->takeName(SI.getI());
      // Current instruction might be a condition of some other group, so we
      // need to replace it there to avoid dangling pointer
      if (PN->getType()->isIntegerTy(1)) {
        for (auto &SG : ProfSIGroups) {
          if (SG.Condition == SI.getI())
            SG.Condition = PN;
````
- **L721 EN**: Continues logic with `auto *CondFr =`.
  **L721 CN**: 继续处理逻辑：`auto *CondFr =`。
- **L722 EN**: Executes statement `IB.CreateFreeze(ASI.Condition, ASI.Condition->getName() + ".frozen");`.
  **L722 CN**: 执行语句 `IB.CreateFreeze(ASI.Condition, ASI.Condition->getName() + ".frozen");`。
- **L723 EN**: Separates nearby statements for readability.
  **L723 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L724 EN**: Executes statement `SmallDenseMap<Instruction *, std::pair<Value *, Value *>, 2> INS;`.
  **L724 CN**: 执行语句 `SmallDenseMap<Instruction *, std::pair<Value *, Value *>, 2> INS;`。
- **L725 EN**: Separates nearby statements for readability.
  **L725 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L726 EN**: Comment documents: `Use reverse iterator because later select may use the value of the`.
  **L726 CN**: 注释说明：`Use reverse iterator because later select may use the value of the`。
- **L727 EN**: Comment documents: `earlier select, and we need to propagate value through earlier select`.
  **L727 CN**: 注释说明：`earlier select, and we need to propagate value through earlier select`。
- **L728 EN**: Comment documents: `to get the PHI operand.`.
  **L728 CN**: 注释说明：`to get the PHI operand.`。
- **L729 EN**: Assigns or initializes `InsertionPoint`.
  **L729 CN**: 对 `InsertionPoint` 进行赋值或初始化。
- **L730 EN**: Starts a loop over a sequence or range.
  **L730 CN**: 开始遍历序列或范围的循环。
- **L731 EN**: Comment documents: `The select itself is replaced with a PHI Node.`.
  **L731 CN**: 注释说明：`The select itself is replaced with a PHI Node.`。
- **L732 EN**: Declares function or method `Create`.
  **L732 CN**: 声明函数或方法 `Create`。
- **L733 EN**: Executes statement `PN->insertBefore(InsertionPoint);`.
  **L733 CN**: 执行语句 `PN->insertBefore(InsertionPoint);`。
- **L734 EN**: Executes statement `PN->takeName(SI.getI());`.
  **L734 CN**: 执行语句 `PN->takeName(SI.getI());`。
- **L735 EN**: Comment documents: `Current instruction might be a condition of some other group, so we`.
  **L735 CN**: 注释说明：`Current instruction might be a condition of some other group, so we`。
- **L736 EN**: Comment documents: `need to replace it there to avoid dangling pointer`.
  **L736 CN**: 注释说明：`need to replace it there to avoid dangling pointer`。
- **L737 EN**: Begins a conditional branch.
  **L737 CN**: 开始一个条件分支。
- **L738 EN**: Starts a loop over a sequence or range.
  **L738 CN**: 开始遍历序列或范围的循环。
- **L739 EN**: Begins a conditional branch.
  **L739 CN**: 开始一个条件分支。
- **L740 EN**: Assigns or initializes `SG.Condition`.
  **L740 CN**: 对 `SG.Condition` 进行赋值或初始化。

### Lines 741-760

````cpp
        }
      }
      SI.getI()->replaceAllUsesWith(PN);
      auto *TV = getTrueOrFalseValue(SI, true, INS, TrueBlock);
      auto *FV = getTrueOrFalseValue(SI, false, INS, FalseBlock);
      INS[PN] = {TV, FV};
      PN->addIncoming(TV, TrueBlock);
      PN->addIncoming(FV, FalseBlock);
      PN->setDebugLoc(SI.getI()->getDebugLoc());
      ++NumSelectsConverted;
    }
    IB.CreateCondBr(CondFr, TT, FT, SI.getI());

    // Remove the old select instructions, now that they are not longer used.
    for (SelectLike &SI : ASI.Selects)
      SI.getI()->eraseFromParent();
  }
}

void SelectOptimizeImpl::collectSelectGroups(BasicBlock &BB,
````
- **L741 EN**: Closes the current scope.
  **L741 CN**: 关闭当前作用域。
- **L742 EN**: Closes the current scope.
  **L742 CN**: 关闭当前作用域。
- **L743 EN**: Executes statement `SI.getI()->replaceAllUsesWith(PN);`.
  **L743 CN**: 执行语句 `SI.getI()->replaceAllUsesWith(PN);`。
- **L744 EN**: Assigns or initializes `auto *TV`.
  **L744 CN**: 对 `auto *TV` 进行赋值或初始化。
- **L745 EN**: Assigns or initializes `auto *FV`.
  **L745 CN**: 对 `auto *FV` 进行赋值或初始化。
- **L746 EN**: Assigns or initializes `INS[PN]`.
  **L746 CN**: 对 `INS[PN]` 进行赋值或初始化。
- **L747 EN**: Executes statement `PN->addIncoming(TV, TrueBlock);`.
  **L747 CN**: 执行语句 `PN->addIncoming(TV, TrueBlock);`。
- **L748 EN**: Executes statement `PN->addIncoming(FV, FalseBlock);`.
  **L748 CN**: 执行语句 `PN->addIncoming(FV, FalseBlock);`。
- **L749 EN**: Executes statement `PN->setDebugLoc(SI.getI()->getDebugLoc());`.
  **L749 CN**: 执行语句 `PN->setDebugLoc(SI.getI()->getDebugLoc());`。
- **L750 EN**: Executes statement `++NumSelectsConverted;`.
  **L750 CN**: 执行语句 `++NumSelectsConverted;`。
- **L751 EN**: Closes the current scope.
  **L751 CN**: 关闭当前作用域。
- **L752 EN**: Executes statement `IB.CreateCondBr(CondFr, TT, FT, SI.getI());`.
  **L752 CN**: 执行语句 `IB.CreateCondBr(CondFr, TT, FT, SI.getI());`。
- **L753 EN**: Separates nearby statements for readability.
  **L753 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L754 EN**: Comment documents: `Remove the old select instructions, now that they are not longer used.`.
  **L754 CN**: 注释说明：`Remove the old select instructions, now that they are not longer used.`。
- **L755 EN**: Starts a loop over a sequence or range.
  **L755 CN**: 开始遍历序列或范围的循环。
- **L756 EN**: Executes statement `SI.getI()->eraseFromParent();`.
  **L756 CN**: 执行语句 `SI.getI()->eraseFromParent();`。
- **L757 EN**: Closes the current scope.
  **L757 CN**: 关闭当前作用域。
- **L758 EN**: Closes the current scope.
  **L758 CN**: 关闭当前作用域。
- **L759 EN**: Separates nearby statements for readability.
  **L759 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L760 EN**: Provides part of the signature for `collectSelectGroups`.
  **L760 CN**: 给出 `collectSelectGroups` 的一部分签名。

### Lines 761-780

````cpp
                                             SelectGroups &SIGroups) {
  // Represents something that can be considered as select instruction.
  // Auxiliary instruction are instructions that depends on a condition and have
  // zero or some constant value on True/False branch, such as:
  // * ZExt(1bit)
  // * SExt(1bit)
  // * Not(1bit)
  // * A(L)Shr(Val), ValBitSize - 1, where there is a condition like `Val <= 0`
  // earlier in the BB. For conditions that check the sign of the Val compiler
  // may generate shifts instead of ZExt/SExt.
  struct SelectLikeInfo {
    Value *Cond;
    bool IsAuxiliary;
    bool IsInverted;
    unsigned ConditionIdx;
  };

  DenseMap<Value *, SelectLikeInfo> SelectInfo;
  // Keeps visited comparisons to help identify AShr/LShr variants of auxiliary
  // instructions.
````
- **L761 EN**: Starts block `SelectGroups &SIGroups)`.
  **L761 CN**: 开始代码块 `SelectGroups &SIGroups)`。
- **L762 EN**: Comment documents: `Represents something that can be considered as select instruction.`.
  **L762 CN**: 注释说明：`Represents something that can be considered as select instruction.`。
- **L763 EN**: Comment documents: `Auxiliary instruction are instructions that depends on a condition and h…`.
  **L763 CN**: 注释说明：`Auxiliary instruction are instructions that depends on a condition and h…`。
- **L764 EN**: Comment documents: `zero or some constant value on True/False branch, such as:`.
  **L764 CN**: 注释说明：`zero or some constant value on True/False branch, such as:`。
- **L765 EN**: Comment documents: `ZExt(1bit)`.
  **L765 CN**: 注释说明：`ZExt(1bit)`。
- **L766 EN**: Comment documents: `SExt(1bit)`.
  **L766 CN**: 注释说明：`SExt(1bit)`。
- **L767 EN**: Comment documents: `Not(1bit)`.
  **L767 CN**: 注释说明：`Not(1bit)`。
- **L768 EN**: Comment documents: `A(L)Shr(Val), ValBitSize - 1, where there is a condition like 'Val <= 0'`.
  **L768 CN**: 注释说明：`A(L)Shr(Val), ValBitSize - 1, where there is a condition like 'Val <= 0'`。
- **L769 EN**: Comment documents: `earlier in the BB. For conditions that check the sign of the Val compile…`.
  **L769 CN**: 注释说明：`earlier in the BB. For conditions that check the sign of the Val compile…`。
- **L770 EN**: Comment documents: `may generate shifts instead of ZExt/SExt.`.
  **L770 CN**: 注释说明：`may generate shifts instead of ZExt/SExt.`。
- **L771 EN**: Starts the declaration of struct `SelectLikeInfo`.
  **L771 CN**: 开始声明 struct `SelectLikeInfo`。
- **L772 EN**: Executes statement `Value *Cond;`.
  **L772 CN**: 执行语句 `Value *Cond;`。
- **L773 EN**: Executes statement `bool IsAuxiliary;`.
  **L773 CN**: 执行语句 `bool IsAuxiliary;`。
- **L774 EN**: Executes statement `bool IsInverted;`.
  **L774 CN**: 执行语句 `bool IsInverted;`。
- **L775 EN**: Executes statement `unsigned ConditionIdx;`.
  **L775 CN**: 执行语句 `unsigned ConditionIdx;`。
- **L776 EN**: Closes the current scope.
  **L776 CN**: 关闭当前作用域。
- **L777 EN**: Separates nearby statements for readability.
  **L777 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L778 EN**: Executes statement `DenseMap<Value *, SelectLikeInfo> SelectInfo;`.
  **L778 CN**: 执行语句 `DenseMap<Value *, SelectLikeInfo> SelectInfo;`。
- **L779 EN**: Comment documents: `Keeps visited comparisons to help identify AShr/LShr variants of auxilia…`.
  **L779 CN**: 注释说明：`Keeps visited comparisons to help identify AShr/LShr variants of auxilia…`。
- **L780 EN**: Comment documents: `instructions.`.
  **L780 CN**: 注释说明：`instructions.`。

### Lines 781-800

````cpp
  SmallSetVector<CmpInst *, 4> SeenCmp;

  // Check if the instruction is SelectLike or might be part of SelectLike
  // expression, put information into SelectInfo and return the iterator to the
  // inserted position.
  auto ProcessSelectInfo = [&SelectInfo, &SeenCmp](Instruction *I) {
    if (auto *Cmp = dyn_cast<CmpInst>(I)) {
      SeenCmp.insert(Cmp);
      return SelectInfo.end();
    }

    Value *Cond;
    if (match(I, m_OneUse(m_ZExtOrSExt(m_Value(Cond)))) &&
        Cond->getType()->isIntegerTy(1)) {
      bool Inverted = match(Cond, m_Not(m_Value(Cond)));
      return SelectInfo.insert({I, {Cond, true, Inverted, 0}}).first;
    }

    if (match(I, m_Not(m_Value(Cond)))) {
      return SelectInfo.insert({I, {Cond, true, true, 0}}).first;
````
- **L781 EN**: Executes statement `SmallSetVector<CmpInst *, 4> SeenCmp;`.
  **L781 CN**: 执行语句 `SmallSetVector<CmpInst *, 4> SeenCmp;`。
- **L782 EN**: Separates nearby statements for readability.
  **L782 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L783 EN**: Comment documents: `Check if the instruction is SelectLike or might be part of SelectLike`.
  **L783 CN**: 注释说明：`Check if the instruction is SelectLike or might be part of SelectLike`。
- **L784 EN**: Comment documents: `expression, put information into SelectInfo and return the iterator to t…`.
  **L784 CN**: 注释说明：`expression, put information into SelectInfo and return the iterator to t…`。
- **L785 EN**: Comment documents: `inserted position.`.
  **L785 CN**: 注释说明：`inserted position.`。
- **L786 EN**: Starts block `auto ProcessSelectInfo = [&SelectInfo, &SeenCmp](Instruction *I)`.
  **L786 CN**: 开始代码块 `auto ProcessSelectInfo = [&SelectInfo, &SeenCmp](Instruction *I)`。
- **L787 EN**: Begins a conditional branch.
  **L787 CN**: 开始一个条件分支。
- **L788 EN**: Executes statement `SeenCmp.insert(Cmp);`.
  **L788 CN**: 执行语句 `SeenCmp.insert(Cmp);`。
- **L789 EN**: Returns `SelectInfo.end()` to the caller.
  **L789 CN**: 向调用者返回 `SelectInfo.end()`。
- **L790 EN**: Closes the current scope.
  **L790 CN**: 关闭当前作用域。
- **L791 EN**: Separates nearby statements for readability.
  **L791 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L792 EN**: Executes statement `Value *Cond;`.
  **L792 CN**: 执行语句 `Value *Cond;`。
- **L793 EN**: Begins a conditional branch.
  **L793 CN**: 开始一个条件分支。
- **L794 EN**: Starts block `Cond->getType()->isIntegerTy(1))`.
  **L794 CN**: 开始代码块 `Cond->getType()->isIntegerTy(1))`。
- **L795 EN**: Assigns or initializes `bool Inverted`.
  **L795 CN**: 对 `bool Inverted` 进行赋值或初始化。
- **L796 EN**: Returns `SelectInfo.insert({I, {Cond, true, Inverted, 0}}).first` to the caller.
  **L796 CN**: 向调用者返回 `SelectInfo.insert({I, {Cond, true, Inverted, 0}}).first`。
- **L797 EN**: Closes the current scope.
  **L797 CN**: 关闭当前作用域。
- **L798 EN**: Separates nearby statements for readability.
  **L798 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L799 EN**: Begins a conditional branch.
  **L799 CN**: 开始一个条件分支。
- **L800 EN**: Returns `SelectInfo.insert({I, {Cond, true, true, 0}}).first` to the caller.
  **L800 CN**: 向调用者返回 `SelectInfo.insert({I, {Cond, true, true, 0}}).first`。

### Lines 801-820

````cpp
    }

    // Select instruction are what we are usually looking for.
    if (match(I, m_Select(m_Value(Cond), m_Value(), m_Value()))) {
      bool Inverted = match(Cond, m_Not(m_Value(Cond)));
      return SelectInfo.insert({I, {Cond, false, Inverted, 0}}).first;
    }
    Value *Val;
    ConstantInt *Shift;
    if (match(I, m_Shr(m_Value(Val), m_ConstantInt(Shift))) &&
        I->getType()->getIntegerBitWidth() == Shift->getZExtValue() + 1) {
      for (auto *CmpI : SeenCmp) {
        auto Pred = CmpI->getPredicate();
        if (Val != CmpI->getOperand(0))
          continue;
        if ((Pred == CmpInst::ICMP_SGT &&
             match(CmpI->getOperand(1), m_ConstantInt<-1>())) ||
            (Pred == CmpInst::ICMP_SGE &&
             match(CmpI->getOperand(1), m_Zero())) ||
            (Pred == CmpInst::ICMP_SLT &&
````
- **L801 EN**: Closes the current scope.
  **L801 CN**: 关闭当前作用域。
- **L802 EN**: Separates nearby statements for readability.
  **L802 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L803 EN**: Comment documents: `Select instruction are what we are usually looking for.`.
  **L803 CN**: 注释说明：`Select instruction are what we are usually looking for.`。
- **L804 EN**: Begins a conditional branch.
  **L804 CN**: 开始一个条件分支。
- **L805 EN**: Assigns or initializes `bool Inverted`.
  **L805 CN**: 对 `bool Inverted` 进行赋值或初始化。
- **L806 EN**: Returns `SelectInfo.insert({I, {Cond, false, Inverted, 0}}).first` to the caller.
  **L806 CN**: 向调用者返回 `SelectInfo.insert({I, {Cond, false, Inverted, 0}}).first`。
- **L807 EN**: Closes the current scope.
  **L807 CN**: 关闭当前作用域。
- **L808 EN**: Executes statement `Value *Val;`.
  **L808 CN**: 执行语句 `Value *Val;`。
- **L809 EN**: Executes statement `ConstantInt *Shift;`.
  **L809 CN**: 执行语句 `ConstantInt *Shift;`。
- **L810 EN**: Begins a conditional branch.
  **L810 CN**: 开始一个条件分支。
- **L811 EN**: Starts block `I->getType()->getIntegerBitWidth() == Shift->getZExtValue() + 1)`.
  **L811 CN**: 开始代码块 `I->getType()->getIntegerBitWidth() == Shift->getZExtValue() + 1)`。
- **L812 EN**: Starts a loop over a sequence or range.
  **L812 CN**: 开始遍历序列或范围的循环。
- **L813 EN**: Assigns or initializes `auto Pred`.
  **L813 CN**: 对 `auto Pred` 进行赋值或初始化。
- **L814 EN**: Begins a conditional branch.
  **L814 CN**: 开始一个条件分支。
- **L815 EN**: Skips to the next loop iteration.
  **L815 CN**: 跳到下一次循环迭代。
- **L816 EN**: Begins a conditional branch.
  **L816 CN**: 开始一个条件分支。
- **L817 EN**: Continues logic with `match(CmpI->getOperand(1), m_ConstantInt<-1>())) ||`.
  **L817 CN**: 继续处理逻辑：`match(CmpI->getOperand(1), m_ConstantInt<-1>())) ||`。
- **L818 EN**: Continues logic with `(Pred == CmpInst::ICMP_SGE &&`.
  **L818 CN**: 继续处理逻辑：`(Pred == CmpInst::ICMP_SGE &&`。
- **L819 EN**: Continues logic with `match(CmpI->getOperand(1), m_Zero())) ||`.
  **L819 CN**: 继续处理逻辑：`match(CmpI->getOperand(1), m_Zero())) ||`。
- **L820 EN**: Continues logic with `(Pred == CmpInst::ICMP_SLT &&`.
  **L820 CN**: 继续处理逻辑：`(Pred == CmpInst::ICMP_SLT &&`。

### Lines 821-840

````cpp
             match(CmpI->getOperand(1), m_Zero())) ||
            (Pred == CmpInst::ICMP_SLE &&
             match(CmpI->getOperand(1), m_ConstantInt<-1>()))) {
          bool Inverted =
              Pred == CmpInst::ICMP_SGT || Pred == CmpInst::ICMP_SGE;
          return SelectInfo.insert({I, {CmpI, true, Inverted, 0}}).first;
        }
      }
      return SelectInfo.end();
    }

    // An BinOp(Aux(X), Y) can also be treated like a select, with condition X
    // and values Y|1 and Y.
    // `Aux` can be either `ZExt(1bit)`, `SExt(1bit)` or `XShr(Val), ValBitSize
    // - 1` `BinOp` can be Add, Sub, Or
    Value *X;
    auto MatchZExtOrSExtPattern =
        m_c_BinOp(m_Value(), m_OneUse(m_ZExtOrSExt(m_Value(X))));
    auto MatchShiftPattern =
        m_c_BinOp(m_Value(), m_OneUse(m_Shr(m_Value(X), m_ConstantInt(Shift))));
````
- **L821 EN**: Continues logic with `match(CmpI->getOperand(1), m_Zero())) ||`.
  **L821 CN**: 继续处理逻辑：`match(CmpI->getOperand(1), m_Zero())) ||`。
- **L822 EN**: Continues logic with `(Pred == CmpInst::ICMP_SLE &&`.
  **L822 CN**: 继续处理逻辑：`(Pred == CmpInst::ICMP_SLE &&`。
- **L823 EN**: Starts block `match(CmpI->getOperand(1), m_ConstantInt<-1>())))`.
  **L823 CN**: 开始代码块 `match(CmpI->getOperand(1), m_ConstantInt<-1>())))`。
- **L824 EN**: Continues logic with `bool Inverted =`.
  **L824 CN**: 继续处理逻辑：`bool Inverted =`。
- **L825 EN**: Assigns or initializes `Pred`.
  **L825 CN**: 对 `Pred` 进行赋值或初始化。
- **L826 EN**: Returns `SelectInfo.insert({I, {CmpI, true, Inverted, 0}}).first` to the caller.
  **L826 CN**: 向调用者返回 `SelectInfo.insert({I, {CmpI, true, Inverted, 0}}).first`。
- **L827 EN**: Closes the current scope.
  **L827 CN**: 关闭当前作用域。
- **L828 EN**: Closes the current scope.
  **L828 CN**: 关闭当前作用域。
- **L829 EN**: Returns `SelectInfo.end()` to the caller.
  **L829 CN**: 向调用者返回 `SelectInfo.end()`。
- **L830 EN**: Closes the current scope.
  **L830 CN**: 关闭当前作用域。
- **L831 EN**: Separates nearby statements for readability.
  **L831 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L832 EN**: Comment documents: `An BinOp(Aux(X), Y) can also be treated like a select, with condition X`.
  **L832 CN**: 注释说明：`An BinOp(Aux(X), Y) can also be treated like a select, with condition X`。
- **L833 EN**: Comment documents: `and values Y|1 and Y.`.
  **L833 CN**: 注释说明：`and values Y|1 and Y.`。
- **L834 EN**: Comment documents: `'Aux' can be either 'ZExt(1bit)', 'SExt(1bit)' or 'XShr(Val), ValBitSize`.
  **L834 CN**: 注释说明：`'Aux' can be either 'ZExt(1bit)', 'SExt(1bit)' or 'XShr(Val), ValBitSize`。
- **L835 EN**: Comment documents: `- 1' 'BinOp' can be Add, Sub, Or`.
  **L835 CN**: 注释说明：`- 1' 'BinOp' can be Add, Sub, Or`。
- **L836 EN**: Executes statement `Value *X;`.
  **L836 CN**: 执行语句 `Value *X;`。
- **L837 EN**: Continues logic with `auto MatchZExtOrSExtPattern =`.
  **L837 CN**: 继续处理逻辑：`auto MatchZExtOrSExtPattern =`。
- **L838 EN**: Executes statement `m_c_BinOp(m_Value(), m_OneUse(m_ZExtOrSExt(m_Value(X))));`.
  **L838 CN**: 执行语句 `m_c_BinOp(m_Value(), m_OneUse(m_ZExtOrSExt(m_Value(X))));`。
- **L839 EN**: Continues logic with `auto MatchShiftPattern =`.
  **L839 CN**: 继续处理逻辑：`auto MatchShiftPattern =`。
- **L840 EN**: Executes statement `m_c_BinOp(m_Value(), m_OneUse(m_Shr(m_Value(X), m_ConstantInt(Shift))));`.
  **L840 CN**: 执行语句 `m_c_BinOp(m_Value(), m_OneUse(m_Shr(m_Value(X), m_ConstantInt(Shift))));`。

### Lines 841-860

````cpp

    // This check is unnecessary, but it prevents costly access to the
    // SelectInfo map.
    if ((match(I, MatchZExtOrSExtPattern) && X->getType()->isIntegerTy(1)) ||
        (match(I, MatchShiftPattern) &&
         X->getType()->getIntegerBitWidth() == Shift->getZExtValue() + 1)) {
      if (I->getOpcode() != Instruction::Add &&
          I->getOpcode() != Instruction::Sub &&
          I->getOpcode() != Instruction::Or)
        return SelectInfo.end();

      if (I->getOpcode() == Instruction::Or && I->getType()->isIntegerTy(1))
        return SelectInfo.end();

      // Iterate through operands and find dependant on recognised sign
      // extending auxiliary select-like instructions. The operand index does
      // not matter for Add and Or. However, for Sub, we can only safely
      // transform when the operand is second.
      unsigned Idx = I->getOpcode() == Instruction::Sub ? 1 : 0;
      for (; Idx < 2; Idx++) {
````
- **L841 EN**: Separates nearby statements for readability.
  **L841 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L842 EN**: Comment documents: `This check is unnecessary, but it prevents costly access to the`.
  **L842 CN**: 注释说明：`This check is unnecessary, but it prevents costly access to the`。
- **L843 EN**: Comment documents: `SelectInfo map.`.
  **L843 CN**: 注释说明：`SelectInfo map.`。
- **L844 EN**: Begins a conditional branch.
  **L844 CN**: 开始一个条件分支。
- **L845 EN**: Continues logic with `(match(I, MatchShiftPattern) &&`.
  **L845 CN**: 继续处理逻辑：`(match(I, MatchShiftPattern) &&`。
- **L846 EN**: Starts block `X->getType()->getIntegerBitWidth() == Shift->getZExtValue() + 1))`.
  **L846 CN**: 开始代码块 `X->getType()->getIntegerBitWidth() == Shift->getZExtValue() + 1))`。
- **L847 EN**: Begins a conditional branch.
  **L847 CN**: 开始一个条件分支。
- **L848 EN**: Continues logic with `I->getOpcode() != Instruction::Sub &&`.
  **L848 CN**: 继续处理逻辑：`I->getOpcode() != Instruction::Sub &&`。
- **L849 EN**: Continues logic with `I->getOpcode() != Instruction::Or)`.
  **L849 CN**: 继续处理逻辑：`I->getOpcode() != Instruction::Or)`。
- **L850 EN**: Returns `SelectInfo.end()` to the caller.
  **L850 CN**: 向调用者返回 `SelectInfo.end()`。
- **L851 EN**: Separates nearby statements for readability.
  **L851 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L852 EN**: Begins a conditional branch.
  **L852 CN**: 开始一个条件分支。
- **L853 EN**: Returns `SelectInfo.end()` to the caller.
  **L853 CN**: 向调用者返回 `SelectInfo.end()`。
- **L854 EN**: Separates nearby statements for readability.
  **L854 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L855 EN**: Comment documents: `Iterate through operands and find dependant on recognised sign`.
  **L855 CN**: 注释说明：`Iterate through operands and find dependant on recognised sign`。
- **L856 EN**: Comment documents: `extending auxiliary select-like instructions. The operand index does`.
  **L856 CN**: 注释说明：`extending auxiliary select-like instructions. The operand index does`。
- **L857 EN**: Comment documents: `not matter for Add and Or. However, for Sub, we can only safely`.
  **L857 CN**: 注释说明：`not matter for Add and Or. However, for Sub, we can only safely`。
- **L858 EN**: Comment documents: `transform when the operand is second.`.
  **L858 CN**: 注释说明：`transform when the operand is second.`。
- **L859 EN**: Assigns or initializes `unsigned Idx`.
  **L859 CN**: 对 `unsigned Idx` 进行赋值或初始化。
- **L860 EN**: Starts a loop over a sequence or range.
  **L860 CN**: 开始遍历序列或范围的循环。

### Lines 861-880

````cpp
        auto *Op = I->getOperand(Idx);
        auto It = SelectInfo.find(Op);
        if (It != SelectInfo.end() && It->second.IsAuxiliary) {
          Cond = It->second.Cond;
          bool Inverted = It->second.IsInverted;
          return SelectInfo.insert({I, {Cond, false, Inverted, Idx}}).first;
        }
      }
    }
    return SelectInfo.end();
  };

  bool AlreadyProcessed = false;
  BasicBlock::iterator BBIt = BB.begin();
  DenseMap<Value *, SelectLikeInfo>::iterator It;
  while (BBIt != BB.end()) {
    Instruction *I = &*BBIt++;
    if (I->isDebugOrPseudoInst())
      continue;

````
- **L861 EN**: Assigns or initializes `auto *Op`.
  **L861 CN**: 对 `auto *Op` 进行赋值或初始化。
- **L862 EN**: Assigns or initializes `auto It`.
  **L862 CN**: 对 `auto It` 进行赋值或初始化。
- **L863 EN**: Begins a conditional branch.
  **L863 CN**: 开始一个条件分支。
- **L864 EN**: Assigns or initializes `Cond`.
  **L864 CN**: 对 `Cond` 进行赋值或初始化。
- **L865 EN**: Assigns or initializes `bool Inverted`.
  **L865 CN**: 对 `bool Inverted` 进行赋值或初始化。
- **L866 EN**: Returns `SelectInfo.insert({I, {Cond, false, Inverted, Idx}}).first` to the caller.
  **L866 CN**: 向调用者返回 `SelectInfo.insert({I, {Cond, false, Inverted, Idx}}).first`。
- **L867 EN**: Closes the current scope.
  **L867 CN**: 关闭当前作用域。
- **L868 EN**: Closes the current scope.
  **L868 CN**: 关闭当前作用域。
- **L869 EN**: Closes the current scope.
  **L869 CN**: 关闭当前作用域。
- **L870 EN**: Returns `SelectInfo.end()` to the caller.
  **L870 CN**: 向调用者返回 `SelectInfo.end()`。
- **L871 EN**: Closes the current scope.
  **L871 CN**: 关闭当前作用域。
- **L872 EN**: Separates nearby statements for readability.
  **L872 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L873 EN**: Assigns or initializes `bool AlreadyProcessed`.
  **L873 CN**: 对 `bool AlreadyProcessed` 进行赋值或初始化。
- **L874 EN**: Assigns or initializes `BasicBlock::iterator BBIt`.
  **L874 CN**: 对 `BasicBlock::iterator BBIt` 进行赋值或初始化。
- **L875 EN**: Executes statement `DenseMap<Value *, SelectLikeInfo>::iterator It;`.
  **L875 CN**: 执行语句 `DenseMap<Value *, SelectLikeInfo>::iterator It;`。
- **L876 EN**: Starts a while loop controlled by a condition.
  **L876 CN**: 开始一个由条件控制的 while 循环。
- **L877 EN**: Assigns or initializes `Instruction *I`.
  **L877 CN**: 对 `Instruction *I` 进行赋值或初始化。
- **L878 EN**: Begins a conditional branch.
  **L878 CN**: 开始一个条件分支。
- **L879 EN**: Skips to the next loop iteration.
  **L879 CN**: 跳到下一次循环迭代。
- **L880 EN**: Separates nearby statements for readability.
  **L880 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 881-900

````cpp
    if (!AlreadyProcessed)
      It = ProcessSelectInfo(I);
    else
      AlreadyProcessed = false;

    if (It == SelectInfo.end() || It->second.IsAuxiliary)
      continue;

    if (!TTI->shouldTreatInstructionLikeSelect(I))
      continue;

    Value *Cond = It->second.Cond;
    // Vector conditions are not supported.
    if (!Cond->getType()->isIntegerTy(1))
      continue;

    SelectGroup SIGroup = {Cond, {}};
    SIGroup.Selects.emplace_back(I, It->second.IsInverted,
                                 It->second.ConditionIdx);

````
- **L881 EN**: Begins a conditional branch.
  **L881 CN**: 开始一个条件分支。
- **L882 EN**: Assigns or initializes `It`.
  **L882 CN**: 对 `It` 进行赋值或初始化。
- **L883 EN**: Handles the fallback branch.
  **L883 CN**: 处理兜底分支。
- **L884 EN**: Assigns or initializes `AlreadyProcessed`.
  **L884 CN**: 对 `AlreadyProcessed` 进行赋值或初始化。
- **L885 EN**: Separates nearby statements for readability.
  **L885 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L886 EN**: Begins a conditional branch.
  **L886 CN**: 开始一个条件分支。
- **L887 EN**: Skips to the next loop iteration.
  **L887 CN**: 跳到下一次循环迭代。
- **L888 EN**: Separates nearby statements for readability.
  **L888 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L889 EN**: Begins a conditional branch.
  **L889 CN**: 开始一个条件分支。
- **L890 EN**: Skips to the next loop iteration.
  **L890 CN**: 跳到下一次循环迭代。
- **L891 EN**: Separates nearby statements for readability.
  **L891 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L892 EN**: Assigns or initializes `Value *Cond`.
  **L892 CN**: 对 `Value *Cond` 进行赋值或初始化。
- **L893 EN**: Comment documents: `Vector conditions are not supported.`.
  **L893 CN**: 注释说明：`Vector conditions are not supported.`。
- **L894 EN**: Begins a conditional branch.
  **L894 CN**: 开始一个条件分支。
- **L895 EN**: Skips to the next loop iteration.
  **L895 CN**: 跳到下一次循环迭代。
- **L896 EN**: Separates nearby statements for readability.
  **L896 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L897 EN**: Assigns or initializes `SelectGroup SIGroup`.
  **L897 CN**: 对 `SelectGroup SIGroup` 进行赋值或初始化。
- **L898 EN**: Continues logic with `SIGroup.Selects.emplace_back(I, It->second.IsInverted,`.
  **L898 CN**: 继续处理逻辑：`SIGroup.Selects.emplace_back(I, It->second.IsInverted,`。
- **L899 EN**: Executes statement `It->second.ConditionIdx);`.
  **L899 CN**: 执行语句 `It->second.ConditionIdx);`。
- **L900 EN**: Separates nearby statements for readability.
  **L900 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 901-920

````cpp
    // If the select type is not supported, no point optimizing it.
    // Instruction selection will take care of it.
    if (!isSelectKindSupported(SIGroup.Selects.front()))
      continue;

    while (BBIt != BB.end()) {
      Instruction *NI = &*BBIt;
      // Debug/pseudo instructions should be skipped and not prevent the
      // formation of a select group.
      if (NI->isDebugOrPseudoInst()) {
        ++BBIt;
        continue;
      }

      It = ProcessSelectInfo(NI);
      if (It == SelectInfo.end()) {
        AlreadyProcessed = true;
        break;
      }

````
- **L901 EN**: Comment documents: `If the select type is not supported, no point optimizing it.`.
  **L901 CN**: 注释说明：`If the select type is not supported, no point optimizing it.`。
- **L902 EN**: Comment documents: `Instruction selection will take care of it.`.
  **L902 CN**: 注释说明：`Instruction selection will take care of it.`。
- **L903 EN**: Begins a conditional branch.
  **L903 CN**: 开始一个条件分支。
- **L904 EN**: Skips to the next loop iteration.
  **L904 CN**: 跳到下一次循环迭代。
- **L905 EN**: Separates nearby statements for readability.
  **L905 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L906 EN**: Starts a while loop controlled by a condition.
  **L906 CN**: 开始一个由条件控制的 while 循环。
- **L907 EN**: Assigns or initializes `Instruction *NI`.
  **L907 CN**: 对 `Instruction *NI` 进行赋值或初始化。
- **L908 EN**: Comment documents: `Debug/pseudo instructions should be skipped and not prevent the`.
  **L908 CN**: 注释说明：`Debug/pseudo instructions should be skipped and not prevent the`。
- **L909 EN**: Comment documents: `formation of a select group.`.
  **L909 CN**: 注释说明：`formation of a select group.`。
- **L910 EN**: Begins a conditional branch.
  **L910 CN**: 开始一个条件分支。
- **L911 EN**: Executes statement `++BBIt;`.
  **L911 CN**: 执行语句 `++BBIt;`。
- **L912 EN**: Skips to the next loop iteration.
  **L912 CN**: 跳到下一次循环迭代。
- **L913 EN**: Closes the current scope.
  **L913 CN**: 关闭当前作用域。
- **L914 EN**: Separates nearby statements for readability.
  **L914 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L915 EN**: Assigns or initializes `It`.
  **L915 CN**: 对 `It` 进行赋值或初始化。
- **L916 EN**: Begins a conditional branch.
  **L916 CN**: 开始一个条件分支。
- **L917 EN**: Assigns or initializes `AlreadyProcessed`.
  **L917 CN**: 对 `AlreadyProcessed` 进行赋值或初始化。
- **L918 EN**: Breaks out of the current control-flow construct.
  **L918 CN**: 跳出当前控制流结构。
- **L919 EN**: Closes the current scope.
  **L919 CN**: 关闭当前作用域。
- **L920 EN**: Separates nearby statements for readability.
  **L920 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 921-940

````cpp
      // Auxiliary with same condition
      auto [CurrCond, IsAux, IsRev, CondIdx] = It->second;
      if (Cond != CurrCond) {
        AlreadyProcessed = true;
        break;
      }

      if (!IsAux)
        SIGroup.Selects.emplace_back(NI, IsRev, CondIdx);
      ++BBIt;
    }
    LLVM_DEBUG({
      dbgs() << "New Select group (" << SIGroup.Selects.size() << ") with\n";
      for (auto &SI : SIGroup.Selects)
        dbgs() << "  " << *SI.getI() << "\n";
    });

    SIGroups.push_back(SIGroup);
  }
}
````
- **L921 EN**: Comment documents: `Auxiliary with same condition`.
  **L921 CN**: 注释说明：`Auxiliary with same condition`。
- **L922 EN**: Assigns or initializes `auto [CurrCond, IsAux, IsRev, CondIdx]`.
  **L922 CN**: 对 `auto [CurrCond, IsAux, IsRev, CondIdx]` 进行赋值或初始化。
- **L923 EN**: Begins a conditional branch.
  **L923 CN**: 开始一个条件分支。
- **L924 EN**: Assigns or initializes `AlreadyProcessed`.
  **L924 CN**: 对 `AlreadyProcessed` 进行赋值或初始化。
- **L925 EN**: Breaks out of the current control-flow construct.
  **L925 CN**: 跳出当前控制流结构。
- **L926 EN**: Closes the current scope.
  **L926 CN**: 关闭当前作用域。
- **L927 EN**: Separates nearby statements for readability.
  **L927 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L928 EN**: Begins a conditional branch.
  **L928 CN**: 开始一个条件分支。
- **L929 EN**: Executes statement `SIGroup.Selects.emplace_back(NI, IsRev, CondIdx);`.
  **L929 CN**: 执行语句 `SIGroup.Selects.emplace_back(NI, IsRev, CondIdx);`。
- **L930 EN**: Executes statement `++BBIt;`.
  **L930 CN**: 执行语句 `++BBIt;`。
- **L931 EN**: Closes the current scope.
  **L931 CN**: 关闭当前作用域。
- **L932 EN**: Emits debug-only tracing logic.
  **L932 CN**: 发出仅在调试时启用的跟踪逻辑。
- **L933 EN**: Executes statement `dbgs() << "New Select group (" << SIGroup.Selects.size() << ") with\n";`.
  **L933 CN**: 执行语句 `dbgs() << "New Select group (" << SIGroup.Selects.size() << ") with\n";`。
- **L934 EN**: Starts a loop over a sequence or range.
  **L934 CN**: 开始遍历序列或范围的循环。
- **L935 EN**: Executes statement `dbgs() << " " << *SI.getI() << "\n";`.
  **L935 CN**: 执行语句 `dbgs() << " " << *SI.getI() << "\n";`。
- **L936 EN**: Executes statement `});`.
  **L936 CN**: 执行语句 `});`。
- **L937 EN**: Separates nearby statements for readability.
  **L937 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L938 EN**: Executes statement `SIGroups.push_back(SIGroup);`.
  **L938 CN**: 执行语句 `SIGroups.push_back(SIGroup);`。
- **L939 EN**: Closes the current scope.
  **L939 CN**: 关闭当前作用域。
- **L940 EN**: Closes the current scope.
  **L940 CN**: 关闭当前作用域。

### Lines 941-960

````cpp

void SelectOptimizeImpl::findProfitableSIGroupsBase(
    SelectGroups &SIGroups, SelectGroups &ProfSIGroups) {
  for (SelectGroup &ASI : SIGroups) {
    ++NumSelectOptAnalyzed;
    if (isConvertToBranchProfitableBase(ASI))
      ProfSIGroups.push_back(ASI);
  }
}

static void EmitAndPrintRemark(OptimizationRemarkEmitter *ORE,
                               DiagnosticInfoOptimizationBase &Rem) {
  LLVM_DEBUG(dbgs() << Rem.getMsg() << "\n");
  ORE->emit(Rem);
}

void SelectOptimizeImpl::findProfitableSIGroupsInnerLoops(
    const Loop *L, SelectGroups &SIGroups, SelectGroups &ProfSIGroups) {
  NumSelectOptAnalyzed += SIGroups.size();
  // For each select group in an inner-most loop,
````
- **L941 EN**: Separates nearby statements for readability.
  **L941 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L942 EN**: Provides part of the signature for `findProfitableSIGroupsBase`.
  **L942 CN**: 给出 `findProfitableSIGroupsBase` 的一部分签名。
- **L943 EN**: Starts block `SelectGroups &SIGroups, SelectGroups &ProfSIGroups)`.
  **L943 CN**: 开始代码块 `SelectGroups &SIGroups, SelectGroups &ProfSIGroups)`。
- **L944 EN**: Starts a loop over a sequence or range.
  **L944 CN**: 开始遍历序列或范围的循环。
- **L945 EN**: Executes statement `++NumSelectOptAnalyzed;`.
  **L945 CN**: 执行语句 `++NumSelectOptAnalyzed;`。
- **L946 EN**: Begins a conditional branch.
  **L946 CN**: 开始一个条件分支。
- **L947 EN**: Executes statement `ProfSIGroups.push_back(ASI);`.
  **L947 CN**: 执行语句 `ProfSIGroups.push_back(ASI);`。
- **L948 EN**: Closes the current scope.
  **L948 CN**: 关闭当前作用域。
- **L949 EN**: Closes the current scope.
  **L949 CN**: 关闭当前作用域。
- **L950 EN**: Separates nearby statements for readability.
  **L950 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L951 EN**: Provides part of the signature for `EmitAndPrintRemark`.
  **L951 CN**: 给出 `EmitAndPrintRemark` 的一部分签名。
- **L952 EN**: Starts block `DiagnosticInfoOptimizationBase &Rem)`.
  **L952 CN**: 开始代码块 `DiagnosticInfoOptimizationBase &Rem)`。
- **L953 EN**: Emits debug-only tracing logic.
  **L953 CN**: 发出仅在调试时启用的跟踪逻辑。
- **L954 EN**: Executes statement `ORE->emit(Rem);`.
  **L954 CN**: 执行语句 `ORE->emit(Rem);`。
- **L955 EN**: Closes the current scope.
  **L955 CN**: 关闭当前作用域。
- **L956 EN**: Separates nearby statements for readability.
  **L956 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L957 EN**: Provides part of the signature for `findProfitableSIGroupsInnerLoops`.
  **L957 CN**: 给出 `findProfitableSIGroupsInnerLoops` 的一部分签名。
- **L958 EN**: Starts block `const Loop *L, SelectGroups &SIGroups, SelectGroups &ProfSIGroups)`.
  **L958 CN**: 开始代码块 `const Loop *L, SelectGroups &SIGroups, SelectGroups &ProfSIGroups)`。
- **L959 EN**: Assigns or initializes `NumSelectOptAnalyzed +`.
  **L959 CN**: 对 `NumSelectOptAnalyzed +` 进行赋值或初始化。
- **L960 EN**: Comment documents: `For each select group in an inner-most loop,`.
  **L960 CN**: 注释说明：`For each select group in an inner-most loop,`。

### Lines 961-980

````cpp
  // a branch is more preferable than a select/conditional-move if:
  // i) conversion to branches for all the select groups of the loop satisfies
  //    loop-level heuristics including reducing the loop's critical path by
  //    some threshold (see SelectOptimizeImpl::checkLoopHeuristics); and
  // ii) the total cost of the select group is cheaper with a branch compared
  //     to its predicated version. The cost is in terms of latency and the cost
  //     of a select group is the cost of its most expensive select instruction
  //     (assuming infinite resources and thus fully leveraging available ILP).

  DenseMap<const Instruction *, CostInfo> InstCostMap;
  CostInfo LoopCost[2] = {{Scaled64::getZero(), Scaled64::getZero()},
                          {Scaled64::getZero(), Scaled64::getZero()}};
  if (!computeLoopCosts(L, SIGroups, InstCostMap, LoopCost) ||
      !checkLoopHeuristics(L, LoopCost)) {
    return;
  }

  for (SelectGroup &ASI : SIGroups) {
    // Assuming infinite resources, the cost of a group of instructions is the
    // cost of the most expensive instruction of the group.
````
- **L961 EN**: Comment documents: `a branch is more preferable than a select/conditional-move if:`.
  **L961 CN**: 注释说明：`a branch is more preferable than a select/conditional-move if:`。
- **L962 EN**: Comment documents: `i) conversion to branches for all the select groups of the loop satisfie…`.
  **L962 CN**: 注释说明：`i) conversion to branches for all the select groups of the loop satisfie…`。
- **L963 EN**: Comment documents: `loop-level heuristics including reducing the loop's critical path by`.
  **L963 CN**: 注释说明：`loop-level heuristics including reducing the loop's critical path by`。
- **L964 EN**: Comment documents: `some threshold (see SelectOptimizeImpl::checkLoopHeuristics); and`.
  **L964 CN**: 注释说明：`some threshold (see SelectOptimizeImpl::checkLoopHeuristics); and`。
- **L965 EN**: Comment documents: `ii) the total cost of the select group is cheaper with a branch compared`.
  **L965 CN**: 注释说明：`ii) the total cost of the select group is cheaper with a branch compared`。
- **L966 EN**: Comment documents: `to its predicated version. The cost is in terms of latency and the cost`.
  **L966 CN**: 注释说明：`to its predicated version. The cost is in terms of latency and the cost`。
- **L967 EN**: Comment documents: `of a select group is the cost of its most expensive select instruction`.
  **L967 CN**: 注释说明：`of a select group is the cost of its most expensive select instruction`。
- **L968 EN**: Comment documents: `(assuming infinite resources and thus fully leveraging available ILP).`.
  **L968 CN**: 注释说明：`(assuming infinite resources and thus fully leveraging available ILP).`。
- **L969 EN**: Separates nearby statements for readability.
  **L969 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L970 EN**: Executes statement `DenseMap<const Instruction *, CostInfo> InstCostMap;`.
  **L970 CN**: 执行语句 `DenseMap<const Instruction *, CostInfo> InstCostMap;`。
- **L971 EN**: Provides part of the signature for `getZero`.
  **L971 CN**: 给出 `getZero` 的一部分签名。
- **L972 EN**: Declares function or method `getZero`.
  **L972 CN**: 声明函数或方法 `getZero`。
- **L973 EN**: Begins a conditional branch.
  **L973 CN**: 开始一个条件分支。
- **L974 EN**: Starts block `!checkLoopHeuristics(L, LoopCost))`.
  **L974 CN**: 开始代码块 `!checkLoopHeuristics(L, LoopCost))`。
- **L975 EN**: Returns control to the caller.
  **L975 CN**: 将控制流返回给调用者。
- **L976 EN**: Closes the current scope.
  **L976 CN**: 关闭当前作用域。
- **L977 EN**: Separates nearby statements for readability.
  **L977 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L978 EN**: Starts a loop over a sequence or range.
  **L978 CN**: 开始遍历序列或范围的循环。
- **L979 EN**: Comment documents: `Assuming infinite resources, the cost of a group of instructions is the`.
  **L979 CN**: 注释说明：`Assuming infinite resources, the cost of a group of instructions is the`。
- **L980 EN**: Comment documents: `cost of the most expensive instruction of the group.`.
  **L980 CN**: 注释说明：`cost of the most expensive instruction of the group.`。

### Lines 981-1000

````cpp
    Scaled64 SelectCost = Scaled64::getZero(), BranchCost = Scaled64::getZero();
    for (SelectLike &SI : ASI.Selects) {
      const auto &ICM = InstCostMap[SI.getI()];
      SelectCost = std::max(SelectCost, ICM.PredCost);
      BranchCost = std::max(BranchCost, ICM.NonPredCost);
    }
    if (BranchCost < SelectCost) {
      OptimizationRemark OR(DEBUG_TYPE, "SelectOpti",
                            ASI.Selects.front().getI());
      OR << "Profitable to convert to branch (loop analysis). BranchCost="
         << BranchCost.toString() << ", SelectCost=" << SelectCost.toString()
         << ". ";
      EmitAndPrintRemark(ORE, OR);
      ++NumSelectConvertedLoop;
      ProfSIGroups.push_back(ASI);
    } else {
      OptimizationRemarkMissed ORmiss(DEBUG_TYPE, "SelectOpti",
                                      ASI.Selects.front().getI());
      ORmiss << "Select is more profitable (loop analysis). BranchCost="
             << BranchCost.toString()
````
- **L981 EN**: Declares function or method `getZero`.
  **L981 CN**: 声明函数或方法 `getZero`。
- **L982 EN**: Starts a loop over a sequence or range.
  **L982 CN**: 开始遍历序列或范围的循环。
- **L983 EN**: Assigns or initializes `const auto &ICM`.
  **L983 CN**: 对 `const auto &ICM` 进行赋值或初始化。
- **L984 EN**: Declares function or method `max`.
  **L984 CN**: 声明函数或方法 `max`。
- **L985 EN**: Declares function or method `max`.
  **L985 CN**: 声明函数或方法 `max`。
- **L986 EN**: Closes the current scope.
  **L986 CN**: 关闭当前作用域。
- **L987 EN**: Begins a conditional branch.
  **L987 CN**: 开始一个条件分支。
- **L988 EN**: Provides part of the signature for `OR`.
  **L988 CN**: 给出 `OR` 的一部分签名。
- **L989 EN**: Executes statement `ASI.Selects.front().getI());`.
  **L989 CN**: 执行语句 `ASI.Selects.front().getI());`。
- **L990 EN**: Continues logic with `OR << "Profitable to convert to branch (loop analysis). BranchCost="`.
  **L990 CN**: 继续处理逻辑：`OR << "Profitable to convert to branch (loop analysis). BranchCost="`。
- **L991 EN**: Continues logic with `<< BranchCost.toString() << ", SelectCost=" << SelectCost.toString()`.
  **L991 CN**: 继续处理逻辑：`<< BranchCost.toString() << ", SelectCost=" << SelectCost.toString()`。
- **L992 EN**: Executes statement `<< ". ";`.
  **L992 CN**: 执行语句 `<< ". ";`。
- **L993 EN**: Executes statement `EmitAndPrintRemark(ORE, OR);`.
  **L993 CN**: 执行语句 `EmitAndPrintRemark(ORE, OR);`。
- **L994 EN**: Executes statement `++NumSelectConvertedLoop;`.
  **L994 CN**: 执行语句 `++NumSelectConvertedLoop;`。
- **L995 EN**: Executes statement `ProfSIGroups.push_back(ASI);`.
  **L995 CN**: 执行语句 `ProfSIGroups.push_back(ASI);`。
- **L996 EN**: Starts block `} else`.
  **L996 CN**: 开始代码块 `} else`。
- **L997 EN**: Provides part of the signature for `ORmiss`.
  **L997 CN**: 给出 `ORmiss` 的一部分签名。
- **L998 EN**: Executes statement `ASI.Selects.front().getI());`.
  **L998 CN**: 执行语句 `ASI.Selects.front().getI());`。
- **L999 EN**: Continues logic with `ORmiss << "Select is more profitable (loop analysis). BranchCost="`.
  **L999 CN**: 继续处理逻辑：`ORmiss << "Select is more profitable (loop analysis). BranchCost="`。
- **L1000 EN**: Continues logic with `<< BranchCost.toString()`.
  **L1000 CN**: 继续处理逻辑：`<< BranchCost.toString()`。

### Lines 1001-1020

````cpp
             << ", SelectCost=" << SelectCost.toString() << ". ";
      EmitAndPrintRemark(ORE, ORmiss);
    }
  }
}

bool SelectOptimizeImpl::isConvertToBranchProfitableBase(
    const SelectGroup &ASI) {
  const SelectLike &SI = ASI.Selects.front();
  LLVM_DEBUG(dbgs() << "Analyzing select group containing " << *SI.getI()
                    << "\n");
  OptimizationRemark OR(DEBUG_TYPE, "SelectOpti", SI.getI());
  OptimizationRemarkMissed ORmiss(DEBUG_TYPE, "SelectOpti", SI.getI());

  // Skip cold basic blocks. Better to optimize for size for cold blocks.
  if (PSI->isColdBlock(SI.getI()->getParent(), BFI)) {
    ++NumSelectColdBB;
    ORmiss << "Not converted to branch because of cold basic block. ";
    EmitAndPrintRemark(ORE, ORmiss);
    return false;
````
- **L1001 EN**: Assigns or initializes `<< ", SelectCost`.
  **L1001 CN**: 对 `<< ", SelectCost` 进行赋值或初始化。
- **L1002 EN**: Executes statement `EmitAndPrintRemark(ORE, ORmiss);`.
  **L1002 CN**: 执行语句 `EmitAndPrintRemark(ORE, ORmiss);`。
- **L1003 EN**: Closes the current scope.
  **L1003 CN**: 关闭当前作用域。
- **L1004 EN**: Closes the current scope.
  **L1004 CN**: 关闭当前作用域。
- **L1005 EN**: Closes the current scope.
  **L1005 CN**: 关闭当前作用域。
- **L1006 EN**: Separates nearby statements for readability.
  **L1006 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1007 EN**: Provides part of the signature for `isConvertToBranchProfitableBase`.
  **L1007 CN**: 给出 `isConvertToBranchProfitableBase` 的一部分签名。
- **L1008 EN**: Starts block `const SelectGroup &ASI)`.
  **L1008 CN**: 开始代码块 `const SelectGroup &ASI)`。
- **L1009 EN**: Assigns or initializes `const SelectLike &SI`.
  **L1009 CN**: 对 `const SelectLike &SI` 进行赋值或初始化。
- **L1010 EN**: Emits debug-only tracing logic.
  **L1010 CN**: 发出仅在调试时启用的跟踪逻辑。
- **L1011 EN**: Executes statement `<< "\n");`.
  **L1011 CN**: 执行语句 `<< "\n");`。
- **L1012 EN**: Declares function or method `OR`.
  **L1012 CN**: 声明函数或方法 `OR`。
- **L1013 EN**: Declares function or method `ORmiss`.
  **L1013 CN**: 声明函数或方法 `ORmiss`。
- **L1014 EN**: Separates nearby statements for readability.
  **L1014 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1015 EN**: Comment documents: `Skip cold basic blocks. Better to optimize for size for cold blocks.`.
  **L1015 CN**: 注释说明：`Skip cold basic blocks. Better to optimize for size for cold blocks.`。
- **L1016 EN**: Begins a conditional branch.
  **L1016 CN**: 开始一个条件分支。
- **L1017 EN**: Executes statement `++NumSelectColdBB;`.
  **L1017 CN**: 执行语句 `++NumSelectColdBB;`。
- **L1018 EN**: Executes statement `ORmiss << "Not converted to branch because of cold basic block. ";`.
  **L1018 CN**: 执行语句 `ORmiss << "Not converted to branch because of cold basic block. ";`。
- **L1019 EN**: Executes statement `EmitAndPrintRemark(ORE, ORmiss);`.
  **L1019 CN**: 执行语句 `EmitAndPrintRemark(ORE, ORmiss);`。
- **L1020 EN**: Returns `false` to the caller.
  **L1020 CN**: 向调用者返回 `false`。

### Lines 1021-1040

````cpp
  }

  // If unpredictable, branch form is less profitable.
  if (SI.getI()->getMetadata(LLVMContext::MD_unpredictable)) {
    ++NumSelectUnPred;
    ORmiss << "Not converted to branch because of unpredictable branch. ";
    EmitAndPrintRemark(ORE, ORmiss);
    return false;
  }

  // If highly predictable, branch form is more profitable, unless a
  // predictable select is inexpensive in the target architecture.
  if (isSelectHighlyPredictable(SI) && TLI->isPredictableSelectExpensive()) {
    ++NumSelectConvertedHighPred;
    OR << "Converted to branch because of highly predictable branch. ";
    EmitAndPrintRemark(ORE, OR);
    return true;
  }

  // Look for expensive instructions in the cold operand's (if any) dependence
````
- **L1021 EN**: Closes the current scope.
  **L1021 CN**: 关闭当前作用域。
- **L1022 EN**: Separates nearby statements for readability.
  **L1022 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1023 EN**: Comment documents: `If unpredictable, branch form is less profitable.`.
  **L1023 CN**: 注释说明：`If unpredictable, branch form is less profitable.`。
- **L1024 EN**: Begins a conditional branch.
  **L1024 CN**: 开始一个条件分支。
- **L1025 EN**: Executes statement `++NumSelectUnPred;`.
  **L1025 CN**: 执行语句 `++NumSelectUnPred;`。
- **L1026 EN**: Executes statement `ORmiss << "Not converted to branch because of unpredictable branch. ";`.
  **L1026 CN**: 执行语句 `ORmiss << "Not converted to branch because of unpredictable branch. ";`。
- **L1027 EN**: Executes statement `EmitAndPrintRemark(ORE, ORmiss);`.
  **L1027 CN**: 执行语句 `EmitAndPrintRemark(ORE, ORmiss);`。
- **L1028 EN**: Returns `false` to the caller.
  **L1028 CN**: 向调用者返回 `false`。
- **L1029 EN**: Closes the current scope.
  **L1029 CN**: 关闭当前作用域。
- **L1030 EN**: Separates nearby statements for readability.
  **L1030 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1031 EN**: Comment documents: `If highly predictable, branch form is more profitable, unless a`.
  **L1031 CN**: 注释说明：`If highly predictable, branch form is more profitable, unless a`。
- **L1032 EN**: Comment documents: `predictable select is inexpensive in the target architecture.`.
  **L1032 CN**: 注释说明：`predictable select is inexpensive in the target architecture.`。
- **L1033 EN**: Begins a conditional branch.
  **L1033 CN**: 开始一个条件分支。
- **L1034 EN**: Executes statement `++NumSelectConvertedHighPred;`.
  **L1034 CN**: 执行语句 `++NumSelectConvertedHighPred;`。
- **L1035 EN**: Executes statement `OR << "Converted to branch because of highly predictable branch. ";`.
  **L1035 CN**: 执行语句 `OR << "Converted to branch because of highly predictable branch. ";`。
- **L1036 EN**: Executes statement `EmitAndPrintRemark(ORE, OR);`.
  **L1036 CN**: 执行语句 `EmitAndPrintRemark(ORE, OR);`。
- **L1037 EN**: Returns `true` to the caller.
  **L1037 CN**: 向调用者返回 `true`。
- **L1038 EN**: Closes the current scope.
  **L1038 CN**: 关闭当前作用域。
- **L1039 EN**: Separates nearby statements for readability.
  **L1039 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1040 EN**: Comment documents: `Look for expensive instructions in the cold operand's (if any) dependenc…`.
  **L1040 CN**: 注释说明：`Look for expensive instructions in the cold operand's (if any) dependenc…`。

### Lines 1041-1060

````cpp
  // slice of any of the selects in the group.
  if (hasExpensiveColdOperand(ASI)) {
    ++NumSelectConvertedExpColdOperand;
    OR << "Converted to branch because of expensive cold operand.";
    EmitAndPrintRemark(ORE, OR);
    return true;
  }

  // If latch has a select group with several elements, it is usually profitable
  // to convert it to branches. We let `optimizeSelectsInnerLoops` decide if
  // conversion is profitable for innermost loops.
  auto *BB = SI.getI()->getParent();
  auto *L = LI->getLoopFor(BB);
  if (L && !L->isInnermost() && L->getLoopLatch() == BB &&
      ASI.Selects.size() >= 3) {
    OR << "Converted to branch because select group in the latch block is big.";
    EmitAndPrintRemark(ORE, OR);
    return true;
  }

````
- **L1041 EN**: Comment documents: `slice of any of the selects in the group.`.
  **L1041 CN**: 注释说明：`slice of any of the selects in the group.`。
- **L1042 EN**: Begins a conditional branch.
  **L1042 CN**: 开始一个条件分支。
- **L1043 EN**: Executes statement `++NumSelectConvertedExpColdOperand;`.
  **L1043 CN**: 执行语句 `++NumSelectConvertedExpColdOperand;`。
- **L1044 EN**: Executes statement `OR << "Converted to branch because of expensive cold operand.";`.
  **L1044 CN**: 执行语句 `OR << "Converted to branch because of expensive cold operand.";`。
- **L1045 EN**: Executes statement `EmitAndPrintRemark(ORE, OR);`.
  **L1045 CN**: 执行语句 `EmitAndPrintRemark(ORE, OR);`。
- **L1046 EN**: Returns `true` to the caller.
  **L1046 CN**: 向调用者返回 `true`。
- **L1047 EN**: Closes the current scope.
  **L1047 CN**: 关闭当前作用域。
- **L1048 EN**: Separates nearby statements for readability.
  **L1048 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1049 EN**: Comment documents: `If latch has a select group with several elements, it is usually profita…`.
  **L1049 CN**: 注释说明：`If latch has a select group with several elements, it is usually profita…`。
- **L1050 EN**: Comment documents: `to convert it to branches. We let 'optimizeSelectsInnerLoops' decide if`.
  **L1050 CN**: 注释说明：`to convert it to branches. We let 'optimizeSelectsInnerLoops' decide if`。
- **L1051 EN**: Comment documents: `conversion is profitable for innermost loops.`.
  **L1051 CN**: 注释说明：`conversion is profitable for innermost loops.`。
- **L1052 EN**: Assigns or initializes `auto *BB`.
  **L1052 CN**: 对 `auto *BB` 进行赋值或初始化。
- **L1053 EN**: Assigns or initializes `auto *L`.
  **L1053 CN**: 对 `auto *L` 进行赋值或初始化。
- **L1054 EN**: Begins a conditional branch.
  **L1054 CN**: 开始一个条件分支。
- **L1055 EN**: Starts block `ASI.Selects.size() >= 3)`.
  **L1055 CN**: 开始代码块 `ASI.Selects.size() >= 3)`。
- **L1056 EN**: Executes statement `OR << "Converted to branch because select group in the latch block is bi…`.
  **L1056 CN**: 执行语句 `OR << "Converted to branch because select group in the latch block is bi…`。
- **L1057 EN**: Executes statement `EmitAndPrintRemark(ORE, OR);`.
  **L1057 CN**: 执行语句 `EmitAndPrintRemark(ORE, OR);`。
- **L1058 EN**: Returns `true` to the caller.
  **L1058 CN**: 向调用者返回 `true`。
- **L1059 EN**: Closes the current scope.
  **L1059 CN**: 关闭当前作用域。
- **L1060 EN**: Separates nearby statements for readability.
  **L1060 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 1061-1080

````cpp
  ORmiss << "Not profitable to convert to branch (base heuristic).";
  EmitAndPrintRemark(ORE, ORmiss);
  return false;
}

static InstructionCost divideNearest(InstructionCost Numerator,
                                     uint64_t Denominator) {
  return (Numerator + (Denominator / 2)) / Denominator;
}

static bool extractBranchWeights(const SelectOptimizeImpl::SelectLike SI,
                                 uint64_t &TrueVal, uint64_t &FalseVal) {
  if (isa<SelectInst>(SI.getI()))
    return extractBranchWeights(*SI.getI(), TrueVal, FalseVal);
  return false;
}

bool SelectOptimizeImpl::hasExpensiveColdOperand(const SelectGroup &ASI) {
  bool ColdOperand = false;
  uint64_t TrueWeight, FalseWeight, TotalWeight;
````
- **L1061 EN**: Executes statement `ORmiss << "Not profitable to convert to branch (base heuristic).";`.
  **L1061 CN**: 执行语句 `ORmiss << "Not profitable to convert to branch (base heuristic).";`。
- **L1062 EN**: Executes statement `EmitAndPrintRemark(ORE, ORmiss);`.
  **L1062 CN**: 执行语句 `EmitAndPrintRemark(ORE, ORmiss);`。
- **L1063 EN**: Returns `false` to the caller.
  **L1063 CN**: 向调用者返回 `false`。
- **L1064 EN**: Closes the current scope.
  **L1064 CN**: 关闭当前作用域。
- **L1065 EN**: Separates nearby statements for readability.
  **L1065 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1066 EN**: Provides part of the signature for `divideNearest`.
  **L1066 CN**: 给出 `divideNearest` 的一部分签名。
- **L1067 EN**: Starts block `uint64_t Denominator)`.
  **L1067 CN**: 开始代码块 `uint64_t Denominator)`。
- **L1068 EN**: Returns `(Numerator + (Denominator / 2)) / Denominator` to the caller.
  **L1068 CN**: 向调用者返回 `(Numerator + (Denominator / 2)) / Denominator`。
- **L1069 EN**: Closes the current scope.
  **L1069 CN**: 关闭当前作用域。
- **L1070 EN**: Separates nearby statements for readability.
  **L1070 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1071 EN**: Provides part of the signature for `extractBranchWeights`.
  **L1071 CN**: 给出 `extractBranchWeights` 的一部分签名。
- **L1072 EN**: Starts block `uint64_t &TrueVal, uint64_t &FalseVal)`.
  **L1072 CN**: 开始代码块 `uint64_t &TrueVal, uint64_t &FalseVal)`。
- **L1073 EN**: Begins a conditional branch.
  **L1073 CN**: 开始一个条件分支。
- **L1074 EN**: Returns `extractBranchWeights(*SI.getI(), TrueVal, FalseVal)` to the caller.
  **L1074 CN**: 向调用者返回 `extractBranchWeights(*SI.getI(), TrueVal, FalseVal)`。
- **L1075 EN**: Returns `false` to the caller.
  **L1075 CN**: 向调用者返回 `false`。
- **L1076 EN**: Closes the current scope.
  **L1076 CN**: 关闭当前作用域。
- **L1077 EN**: Separates nearby statements for readability.
  **L1077 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1078 EN**: Begins the definition of `hasExpensiveColdOperand`.
  **L1078 CN**: 开始定义 `hasExpensiveColdOperand`。
- **L1079 EN**: Assigns or initializes `bool ColdOperand`.
  **L1079 CN**: 对 `bool ColdOperand` 进行赋值或初始化。
- **L1080 EN**: Executes statement `uint64_t TrueWeight, FalseWeight, TotalWeight;`.
  **L1080 CN**: 执行语句 `uint64_t TrueWeight, FalseWeight, TotalWeight;`。

### Lines 1081-1100

````cpp
  if (extractBranchWeights(ASI.Selects.front(), TrueWeight, FalseWeight)) {
    uint64_t MinWeight = std::min(TrueWeight, FalseWeight);
    TotalWeight = TrueWeight + FalseWeight;
    // Is there a path with frequency <ColdOperandThreshold% (default:20%) ?
    ColdOperand = TotalWeight * ColdOperandThreshold > 100 * MinWeight;
  } else if (PSI->hasProfileSummary()) {
    OptimizationRemarkMissed ORmiss(DEBUG_TYPE, "SelectOpti",
                                    ASI.Selects.front().getI());
    ORmiss << "Profile data available but missing branch-weights metadata for "
              "select instruction. ";
    EmitAndPrintRemark(ORE, ORmiss);
  }
  if (!ColdOperand)
    return false;
  // Check if the cold path's dependence slice is expensive for any of the
  // selects of the group.
  for (SelectLike SI : ASI.Selects) {
    Instruction *ColdI = nullptr;
    uint64_t HotWeight;
    if (TrueWeight < FalseWeight) {
````
- **L1081 EN**: Begins a conditional branch.
  **L1081 CN**: 开始一个条件分支。
- **L1082 EN**: Declares function or method `min`.
  **L1082 CN**: 声明函数或方法 `min`。
- **L1083 EN**: Assigns or initializes `TotalWeight`.
  **L1083 CN**: 对 `TotalWeight` 进行赋值或初始化。
- **L1084 EN**: Comment documents: `Is there a path with frequency <ColdOperandThreshold% (default:20%) ?`.
  **L1084 CN**: 注释说明：`Is there a path with frequency <ColdOperandThreshold% (default:20%) ?`。
- **L1085 EN**: Assigns or initializes `ColdOperand`.
  **L1085 CN**: 对 `ColdOperand` 进行赋值或初始化。
- **L1086 EN**: Starts block `} else if (PSI->hasProfileSummary())`.
  **L1086 CN**: 开始代码块 `} else if (PSI->hasProfileSummary())`。
- **L1087 EN**: Provides part of the signature for `ORmiss`.
  **L1087 CN**: 给出 `ORmiss` 的一部分签名。
- **L1088 EN**: Executes statement `ASI.Selects.front().getI());`.
  **L1088 CN**: 执行语句 `ASI.Selects.front().getI());`。
- **L1089 EN**: Continues logic with `ORmiss << "Profile data available but missing branch-weights metadata fo…`.
  **L1089 CN**: 继续处理逻辑：`ORmiss << "Profile data available but missing branch-weights metadata fo…`。
- **L1090 EN**: Executes statement `"select instruction. ";`.
  **L1090 CN**: 执行语句 `"select instruction. ";`。
- **L1091 EN**: Executes statement `EmitAndPrintRemark(ORE, ORmiss);`.
  **L1091 CN**: 执行语句 `EmitAndPrintRemark(ORE, ORmiss);`。
- **L1092 EN**: Closes the current scope.
  **L1092 CN**: 关闭当前作用域。
- **L1093 EN**: Begins a conditional branch.
  **L1093 CN**: 开始一个条件分支。
- **L1094 EN**: Returns `false` to the caller.
  **L1094 CN**: 向调用者返回 `false`。
- **L1095 EN**: Comment documents: `Check if the cold path's dependence slice is expensive for any of the`.
  **L1095 CN**: 注释说明：`Check if the cold path's dependence slice is expensive for any of the`。
- **L1096 EN**: Comment documents: `selects of the group.`.
  **L1096 CN**: 注释说明：`selects of the group.`。
- **L1097 EN**: Starts a loop over a sequence or range.
  **L1097 CN**: 开始遍历序列或范围的循环。
- **L1098 EN**: Assigns or initializes `Instruction *ColdI`.
  **L1098 CN**: 对 `Instruction *ColdI` 进行赋值或初始化。
- **L1099 EN**: Executes statement `uint64_t HotWeight;`.
  **L1099 CN**: 执行语句 `uint64_t HotWeight;`。
- **L1100 EN**: Begins a conditional branch.
  **L1100 CN**: 开始一个条件分支。

### Lines 1101-1120

````cpp
      ColdI = dyn_cast_or_null<Instruction>(SI.getTrueValue());
      HotWeight = FalseWeight;
    } else {
      ColdI = dyn_cast_or_null<Instruction>(SI.getFalseValue());
      HotWeight = TrueWeight;
    }
    if (ColdI) {
      std::stack<Instruction *> ColdSlice;
      getExclBackwardsSlice(ColdI, ColdSlice, SI.getI());
      InstructionCost SliceCost = 0;
      while (!ColdSlice.empty()) {
        SliceCost += TTI->getInstructionCost(ColdSlice.top(),
                                             TargetTransformInfo::TCK_Latency);
        ColdSlice.pop();
      }
      // The colder the cold value operand of the select is the more expensive
      // the cmov becomes for computing the cold value operand every time. Thus,
      // the colder the cold operand is the more its cost counts.
      // Get nearest integer cost adjusted for coldness.
      InstructionCost AdjSliceCost =
````
- **L1101 EN**: Assigns or initializes `ColdI`.
  **L1101 CN**: 对 `ColdI` 进行赋值或初始化。
- **L1102 EN**: Assigns or initializes `HotWeight`.
  **L1102 CN**: 对 `HotWeight` 进行赋值或初始化。
- **L1103 EN**: Starts block `} else`.
  **L1103 CN**: 开始代码块 `} else`。
- **L1104 EN**: Assigns or initializes `ColdI`.
  **L1104 CN**: 对 `ColdI` 进行赋值或初始化。
- **L1105 EN**: Assigns or initializes `HotWeight`.
  **L1105 CN**: 对 `HotWeight` 进行赋值或初始化。
- **L1106 EN**: Closes the current scope.
  **L1106 CN**: 关闭当前作用域。
- **L1107 EN**: Begins a conditional branch.
  **L1107 CN**: 开始一个条件分支。
- **L1108 EN**: Executes statement `std::stack<Instruction *> ColdSlice;`.
  **L1108 CN**: 执行语句 `std::stack<Instruction *> ColdSlice;`。
- **L1109 EN**: Executes statement `getExclBackwardsSlice(ColdI, ColdSlice, SI.getI());`.
  **L1109 CN**: 执行语句 `getExclBackwardsSlice(ColdI, ColdSlice, SI.getI());`。
- **L1110 EN**: Assigns or initializes `InstructionCost SliceCost`.
  **L1110 CN**: 对 `InstructionCost SliceCost` 进行赋值或初始化。
- **L1111 EN**: Starts a while loop controlled by a condition.
  **L1111 CN**: 开始一个由条件控制的 while 循环。
- **L1112 EN**: Continues logic with `SliceCost += TTI->getInstructionCost(ColdSlice.top(),`.
  **L1112 CN**: 继续处理逻辑：`SliceCost += TTI->getInstructionCost(ColdSlice.top(),`。
- **L1113 EN**: Executes statement `TargetTransformInfo::TCK_Latency);`.
  **L1113 CN**: 执行语句 `TargetTransformInfo::TCK_Latency);`。
- **L1114 EN**: Executes statement `ColdSlice.pop();`.
  **L1114 CN**: 执行语句 `ColdSlice.pop();`。
- **L1115 EN**: Closes the current scope.
  **L1115 CN**: 关闭当前作用域。
- **L1116 EN**: Comment documents: `The colder the cold value operand of the select is the more expensive`.
  **L1116 CN**: 注释说明：`The colder the cold value operand of the select is the more expensive`。
- **L1117 EN**: Comment documents: `the cmov becomes for computing the cold value operand every time. Thus,`.
  **L1117 CN**: 注释说明：`the cmov becomes for computing the cold value operand every time. Thus,`。
- **L1118 EN**: Comment documents: `the colder the cold operand is the more its cost counts.`.
  **L1118 CN**: 注释说明：`the colder the cold operand is the more its cost counts.`。
- **L1119 EN**: Comment documents: `Get nearest integer cost adjusted for coldness.`.
  **L1119 CN**: 注释说明：`Get nearest integer cost adjusted for coldness.`。
- **L1120 EN**: Continues logic with `InstructionCost AdjSliceCost =`.
  **L1120 CN**: 继续处理逻辑：`InstructionCost AdjSliceCost =`。

### Lines 1121-1140

````cpp
          divideNearest(SliceCost * HotWeight, TotalWeight);
      if (AdjSliceCost >=
          ColdOperandMaxCostMultiplier * TargetTransformInfo::TCC_Expensive)
        return true;
    }
  }
  return false;
}

// Check if it is safe to move LoadI next to the SI.
// Conservatively assume it is safe only if there is no instruction
// modifying memory in-between the load and the select instruction.
static bool isSafeToSinkLoad(Instruction *LoadI, Instruction *SI) {
  // Assume loads from different basic blocks are unsafe to move.
  if (LoadI->getParent() != SI->getParent())
    return false;
  auto It = LoadI->getIterator();
  while (&*It != SI) {
    if (It->mayWriteToMemory())
      return false;
````
- **L1121 EN**: Executes statement `divideNearest(SliceCost * HotWeight, TotalWeight);`.
  **L1121 CN**: 执行语句 `divideNearest(SliceCost * HotWeight, TotalWeight);`。
- **L1122 EN**: Begins a conditional branch.
  **L1122 CN**: 开始一个条件分支。
- **L1123 EN**: Continues logic with `ColdOperandMaxCostMultiplier * TargetTransformInfo::TCC_Expensive)`.
  **L1123 CN**: 继续处理逻辑：`ColdOperandMaxCostMultiplier * TargetTransformInfo::TCC_Expensive)`。
- **L1124 EN**: Returns `true` to the caller.
  **L1124 CN**: 向调用者返回 `true`。
- **L1125 EN**: Closes the current scope.
  **L1125 CN**: 关闭当前作用域。
- **L1126 EN**: Closes the current scope.
  **L1126 CN**: 关闭当前作用域。
- **L1127 EN**: Returns `false` to the caller.
  **L1127 CN**: 向调用者返回 `false`。
- **L1128 EN**: Closes the current scope.
  **L1128 CN**: 关闭当前作用域。
- **L1129 EN**: Separates nearby statements for readability.
  **L1129 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1130 EN**: Comment documents: `Check if it is safe to move LoadI next to the SI.`.
  **L1130 CN**: 注释说明：`Check if it is safe to move LoadI next to the SI.`。
- **L1131 EN**: Comment documents: `Conservatively assume it is safe only if there is no instruction`.
  **L1131 CN**: 注释说明：`Conservatively assume it is safe only if there is no instruction`。
- **L1132 EN**: Comment documents: `modifying memory in-between the load and the select instruction.`.
  **L1132 CN**: 注释说明：`modifying memory in-between the load and the select instruction.`。
- **L1133 EN**: Begins the definition of `isSafeToSinkLoad`.
  **L1133 CN**: 开始定义 `isSafeToSinkLoad`。
- **L1134 EN**: Comment documents: `Assume loads from different basic blocks are unsafe to move.`.
  **L1134 CN**: 注释说明：`Assume loads from different basic blocks are unsafe to move.`。
- **L1135 EN**: Begins a conditional branch.
  **L1135 CN**: 开始一个条件分支。
- **L1136 EN**: Returns `false` to the caller.
  **L1136 CN**: 向调用者返回 `false`。
- **L1137 EN**: Assigns or initializes `auto It`.
  **L1137 CN**: 对 `auto It` 进行赋值或初始化。
- **L1138 EN**: Starts a while loop controlled by a condition.
  **L1138 CN**: 开始一个由条件控制的 while 循环。
- **L1139 EN**: Begins a conditional branch.
  **L1139 CN**: 开始一个条件分支。
- **L1140 EN**: Returns `false` to the caller.
  **L1140 CN**: 向调用者返回 `false`。

### Lines 1141-1160

````cpp
    It++;
  }
  return true;
}

// For a given source instruction, collect its backwards dependence slice
// consisting of instructions exclusively computed for the purpose of producing
// the operands of the source instruction. As an approximation
// (sufficiently-accurate in practice), we populate this set with the
// instructions of the backwards dependence slice that only have one-use and
// form an one-use chain that leads to the source instruction.
void SelectOptimizeImpl::getExclBackwardsSlice(Instruction *I,
                                               std::stack<Instruction *> &Slice,
                                               Instruction *SI,
                                               bool ForSinking) {
  SmallPtrSet<Instruction *, 2> Visited;
  std::queue<Instruction *> Worklist;
  Worklist.push(I);
  while (!Worklist.empty()) {
    Instruction *II = Worklist.front();
````
- **L1141 EN**: Executes statement `It++;`.
  **L1141 CN**: 执行语句 `It++;`。
- **L1142 EN**: Closes the current scope.
  **L1142 CN**: 关闭当前作用域。
- **L1143 EN**: Returns `true` to the caller.
  **L1143 CN**: 向调用者返回 `true`。
- **L1144 EN**: Closes the current scope.
  **L1144 CN**: 关闭当前作用域。
- **L1145 EN**: Separates nearby statements for readability.
  **L1145 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1146 EN**: Comment documents: `For a given source instruction, collect its backwards dependence slice`.
  **L1146 CN**: 注释说明：`For a given source instruction, collect its backwards dependence slice`。
- **L1147 EN**: Comment documents: `consisting of instructions exclusively computed for the purpose of produ…`.
  **L1147 CN**: 注释说明：`consisting of instructions exclusively computed for the purpose of produ…`。
- **L1148 EN**: Comment documents: `the operands of the source instruction. As an approximation`.
  **L1148 CN**: 注释说明：`the operands of the source instruction. As an approximation`。
- **L1149 EN**: Comment documents: `(sufficiently-accurate in practice), we populate this set with the`.
  **L1149 CN**: 注释说明：`(sufficiently-accurate in practice), we populate this set with the`。
- **L1150 EN**: Comment documents: `instructions of the backwards dependence slice that only have one-use an…`.
  **L1150 CN**: 注释说明：`instructions of the backwards dependence slice that only have one-use an…`。
- **L1151 EN**: Comment documents: `form an one-use chain that leads to the source instruction.`.
  **L1151 CN**: 注释说明：`form an one-use chain that leads to the source instruction.`。
- **L1152 EN**: Provides part of the signature for `getExclBackwardsSlice`.
  **L1152 CN**: 给出 `getExclBackwardsSlice` 的一部分签名。
- **L1153 EN**: Continues logic with `std::stack<Instruction *> &Slice,`.
  **L1153 CN**: 继续处理逻辑：`std::stack<Instruction *> &Slice,`。
- **L1154 EN**: Continues logic with `Instruction *SI,`.
  **L1154 CN**: 继续处理逻辑：`Instruction *SI,`。
- **L1155 EN**: Starts block `bool ForSinking)`.
  **L1155 CN**: 开始代码块 `bool ForSinking)`。
- **L1156 EN**: Executes statement `SmallPtrSet<Instruction *, 2> Visited;`.
  **L1156 CN**: 执行语句 `SmallPtrSet<Instruction *, 2> Visited;`。
- **L1157 EN**: Executes statement `std::queue<Instruction *> Worklist;`.
  **L1157 CN**: 执行语句 `std::queue<Instruction *> Worklist;`。
- **L1158 EN**: Executes statement `Worklist.push(I);`.
  **L1158 CN**: 执行语句 `Worklist.push(I);`。
- **L1159 EN**: Starts a while loop controlled by a condition.
  **L1159 CN**: 开始一个由条件控制的 while 循环。
- **L1160 EN**: Assigns or initializes `Instruction *II`.
  **L1160 CN**: 对 `Instruction *II` 进行赋值或初始化。

### Lines 1161-1180

````cpp
    Worklist.pop();

    // Avoid cycles.
    if (!Visited.insert(II).second)
      continue;

    if (!II->hasOneUse())
      continue;

    // Cannot soundly sink instructions with side-effects.
    // Terminator or phi instructions cannot be sunk.
    // Avoid sinking other select instructions (should be handled separetely).
    if (ForSinking && (II->isTerminator() || II->mayHaveSideEffects() ||
                       isa<SelectInst>(II) || isa<PHINode>(II)))
      continue;

    // Avoid sinking loads in order not to skip state-modifying instructions,
    // that may alias with the loaded address.
    // Only allow sinking of loads within the same basic block that are
    // conservatively proven to be safe.
````
- **L1161 EN**: Executes statement `Worklist.pop();`.
  **L1161 CN**: 执行语句 `Worklist.pop();`。
- **L1162 EN**: Separates nearby statements for readability.
  **L1162 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1163 EN**: Comment documents: `Avoid cycles.`.
  **L1163 CN**: 注释说明：`Avoid cycles.`。
- **L1164 EN**: Begins a conditional branch.
  **L1164 CN**: 开始一个条件分支。
- **L1165 EN**: Skips to the next loop iteration.
  **L1165 CN**: 跳到下一次循环迭代。
- **L1166 EN**: Separates nearby statements for readability.
  **L1166 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1167 EN**: Begins a conditional branch.
  **L1167 CN**: 开始一个条件分支。
- **L1168 EN**: Skips to the next loop iteration.
  **L1168 CN**: 跳到下一次循环迭代。
- **L1169 EN**: Separates nearby statements for readability.
  **L1169 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1170 EN**: Comment documents: `Cannot soundly sink instructions with side-effects.`.
  **L1170 CN**: 注释说明：`Cannot soundly sink instructions with side-effects.`。
- **L1171 EN**: Comment documents: `Terminator or phi instructions cannot be sunk.`.
  **L1171 CN**: 注释说明：`Terminator or phi instructions cannot be sunk.`。
- **L1172 EN**: Comment documents: `Avoid sinking other select instructions (should be handled separetely).`.
  **L1172 CN**: 注释说明：`Avoid sinking other select instructions (should be handled separetely).`。
- **L1173 EN**: Begins a conditional branch.
  **L1173 CN**: 开始一个条件分支。
- **L1174 EN**: Continues logic with `isa<SelectInst>(II) || isa<PHINode>(II)))`.
  **L1174 CN**: 继续处理逻辑：`isa<SelectInst>(II) || isa<PHINode>(II)))`。
- **L1175 EN**: Skips to the next loop iteration.
  **L1175 CN**: 跳到下一次循环迭代。
- **L1176 EN**: Separates nearby statements for readability.
  **L1176 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1177 EN**: Comment documents: `Avoid sinking loads in order not to skip state-modifying instructions,`.
  **L1177 CN**: 注释说明：`Avoid sinking loads in order not to skip state-modifying instructions,`。
- **L1178 EN**: Comment documents: `that may alias with the loaded address.`.
  **L1178 CN**: 注释说明：`that may alias with the loaded address.`。
- **L1179 EN**: Comment documents: `Only allow sinking of loads within the same basic block that are`.
  **L1179 CN**: 注释说明：`Only allow sinking of loads within the same basic block that are`。
- **L1180 EN**: Comment documents: `conservatively proven to be safe.`.
  **L1180 CN**: 注释说明：`conservatively proven to be safe.`。

### Lines 1181-1200

````cpp
    if (ForSinking && II->mayReadFromMemory() && !isSafeToSinkLoad(II, SI))
      continue;

    // Avoid considering instructions with less frequency than the source
    // instruction (i.e., avoid colder code regions of the dependence slice).
    if (BFI->getBlockFreq(II->getParent()) < BFI->getBlockFreq(I->getParent()))
      continue;

    // Eligible one-use instruction added to the dependence slice.
    Slice.push(II);

    // Explore all the operands of the current instruction to expand the slice.
    for (Value *Op : II->operand_values())
      if (auto *OpI = dyn_cast<Instruction>(Op))
        Worklist.push(OpI);
  }
}

bool SelectOptimizeImpl::isSelectHighlyPredictable(const SelectLike SI) {
  uint64_t TrueWeight, FalseWeight;
````
- **L1181 EN**: Begins a conditional branch.
  **L1181 CN**: 开始一个条件分支。
- **L1182 EN**: Skips to the next loop iteration.
  **L1182 CN**: 跳到下一次循环迭代。
- **L1183 EN**: Separates nearby statements for readability.
  **L1183 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1184 EN**: Comment documents: `Avoid considering instructions with less frequency than the source`.
  **L1184 CN**: 注释说明：`Avoid considering instructions with less frequency than the source`。
- **L1185 EN**: Comment documents: `instruction (i.e., avoid colder code regions of the dependence slice).`.
  **L1185 CN**: 注释说明：`instruction (i.e., avoid colder code regions of the dependence slice).`。
- **L1186 EN**: Begins a conditional branch.
  **L1186 CN**: 开始一个条件分支。
- **L1187 EN**: Skips to the next loop iteration.
  **L1187 CN**: 跳到下一次循环迭代。
- **L1188 EN**: Separates nearby statements for readability.
  **L1188 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1189 EN**: Comment documents: `Eligible one-use instruction added to the dependence slice.`.
  **L1189 CN**: 注释说明：`Eligible one-use instruction added to the dependence slice.`。
- **L1190 EN**: Executes statement `Slice.push(II);`.
  **L1190 CN**: 执行语句 `Slice.push(II);`。
- **L1191 EN**: Separates nearby statements for readability.
  **L1191 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1192 EN**: Comment documents: `Explore all the operands of the current instruction to expand the slice.`.
  **L1192 CN**: 注释说明：`Explore all the operands of the current instruction to expand the slice.`。
- **L1193 EN**: Starts a loop over a sequence or range.
  **L1193 CN**: 开始遍历序列或范围的循环。
- **L1194 EN**: Begins a conditional branch.
  **L1194 CN**: 开始一个条件分支。
- **L1195 EN**: Executes statement `Worklist.push(OpI);`.
  **L1195 CN**: 执行语句 `Worklist.push(OpI);`。
- **L1196 EN**: Closes the current scope.
  **L1196 CN**: 关闭当前作用域。
- **L1197 EN**: Closes the current scope.
  **L1197 CN**: 关闭当前作用域。
- **L1198 EN**: Separates nearby statements for readability.
  **L1198 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1199 EN**: Begins the definition of `isSelectHighlyPredictable`.
  **L1199 CN**: 开始定义 `isSelectHighlyPredictable`。
- **L1200 EN**: Executes statement `uint64_t TrueWeight, FalseWeight;`.
  **L1200 CN**: 执行语句 `uint64_t TrueWeight, FalseWeight;`。

### Lines 1201-1220

````cpp
  if (extractBranchWeights(SI, TrueWeight, FalseWeight)) {
    uint64_t Max = std::max(TrueWeight, FalseWeight);
    uint64_t Sum = TrueWeight + FalseWeight;
    if (Sum != 0) {
      auto Probability = BranchProbability::getBranchProbability(Max, Sum);
      if (Probability > TTI->getPredictableBranchThreshold())
        return true;
    }
  }
  return false;
}

bool SelectOptimizeImpl::checkLoopHeuristics(const Loop *L,
                                             const CostInfo LoopCost[2]) {
  // Loop-level checks to determine if a non-predicated version (with branches)
  // of the loop is more profitable than its predicated version.

  if (DisableLoopLevelHeuristics)
    return true;

````
- **L1201 EN**: Begins a conditional branch.
  **L1201 CN**: 开始一个条件分支。
- **L1202 EN**: Declares function or method `max`.
  **L1202 CN**: 声明函数或方法 `max`。
- **L1203 EN**: Assigns or initializes `uint64_t Sum`.
  **L1203 CN**: 对 `uint64_t Sum` 进行赋值或初始化。
- **L1204 EN**: Begins a conditional branch.
  **L1204 CN**: 开始一个条件分支。
- **L1205 EN**: Declares function or method `getBranchProbability`.
  **L1205 CN**: 声明函数或方法 `getBranchProbability`。
- **L1206 EN**: Begins a conditional branch.
  **L1206 CN**: 开始一个条件分支。
- **L1207 EN**: Returns `true` to the caller.
  **L1207 CN**: 向调用者返回 `true`。
- **L1208 EN**: Closes the current scope.
  **L1208 CN**: 关闭当前作用域。
- **L1209 EN**: Closes the current scope.
  **L1209 CN**: 关闭当前作用域。
- **L1210 EN**: Returns `false` to the caller.
  **L1210 CN**: 向调用者返回 `false`。
- **L1211 EN**: Closes the current scope.
  **L1211 CN**: 关闭当前作用域。
- **L1212 EN**: Separates nearby statements for readability.
  **L1212 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1213 EN**: Provides part of the signature for `checkLoopHeuristics`.
  **L1213 CN**: 给出 `checkLoopHeuristics` 的一部分签名。
- **L1214 EN**: Starts block `const CostInfo LoopCost[2])`.
  **L1214 CN**: 开始代码块 `const CostInfo LoopCost[2])`。
- **L1215 EN**: Comment documents: `Loop-level checks to determine if a non-predicated version (with branche…`.
  **L1215 CN**: 注释说明：`Loop-level checks to determine if a non-predicated version (with branche…`。
- **L1216 EN**: Comment documents: `of the loop is more profitable than its predicated version.`.
  **L1216 CN**: 注释说明：`of the loop is more profitable than its predicated version.`。
- **L1217 EN**: Separates nearby statements for readability.
  **L1217 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1218 EN**: Begins a conditional branch.
  **L1218 CN**: 开始一个条件分支。
- **L1219 EN**: Returns `true` to the caller.
  **L1219 CN**: 向调用者返回 `true`。
- **L1220 EN**: Separates nearby statements for readability.
  **L1220 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 1221-1240

````cpp
  OptimizationRemarkMissed ORmissL(DEBUG_TYPE, "SelectOpti",
                                   &*L->getHeader()->getFirstNonPHIIt());

  if (LoopCost[0].NonPredCost > LoopCost[0].PredCost ||
      LoopCost[1].NonPredCost >= LoopCost[1].PredCost) {
    ORmissL << "No select conversion in the loop due to no reduction of loop's "
               "critical path. ";
    EmitAndPrintRemark(ORE, ORmissL);
    return false;
  }

  Scaled64 Gain[2] = {LoopCost[0].PredCost - LoopCost[0].NonPredCost,
                      LoopCost[1].PredCost - LoopCost[1].NonPredCost};

  // Profitably converting to branches need to reduce the loop's critical path
  // by at least some threshold (absolute gain of GainCycleThreshold cycles and
  // relative gain of 12.5%).
  if (Gain[1] < Scaled64::get(GainCycleThreshold) ||
      Gain[1] * Scaled64::get(GainRelativeThreshold) < LoopCost[1].PredCost) {
    Scaled64 RelativeGain = Scaled64::get(100) * Gain[1] / LoopCost[1].PredCost;
````
- **L1221 EN**: Provides part of the signature for `ORmissL`.
  **L1221 CN**: 给出 `ORmissL` 的一部分签名。
- **L1222 EN**: Executes statement `&*L->getHeader()->getFirstNonPHIIt());`.
  **L1222 CN**: 执行语句 `&*L->getHeader()->getFirstNonPHIIt());`。
- **L1223 EN**: Separates nearby statements for readability.
  **L1223 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1224 EN**: Begins a conditional branch.
  **L1224 CN**: 开始一个条件分支。
- **L1225 EN**: Starts block `LoopCost[1].NonPredCost >= LoopCost[1].PredCost)`.
  **L1225 CN**: 开始代码块 `LoopCost[1].NonPredCost >= LoopCost[1].PredCost)`。
- **L1226 EN**: Continues logic with `ORmissL << "No select conversion in the loop due to no reduction of loop…`.
  **L1226 CN**: 继续处理逻辑：`ORmissL << "No select conversion in the loop due to no reduction of loop…`。
- **L1227 EN**: Executes statement `"critical path. ";`.
  **L1227 CN**: 执行语句 `"critical path. ";`。
- **L1228 EN**: Executes statement `EmitAndPrintRemark(ORE, ORmissL);`.
  **L1228 CN**: 执行语句 `EmitAndPrintRemark(ORE, ORmissL);`。
- **L1229 EN**: Returns `false` to the caller.
  **L1229 CN**: 向调用者返回 `false`。
- **L1230 EN**: Closes the current scope.
  **L1230 CN**: 关闭当前作用域。
- **L1231 EN**: Separates nearby statements for readability.
  **L1231 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1232 EN**: Continues logic with `Scaled64 Gain[2] = {LoopCost[0].PredCost - LoopCost[0].NonPredCost,`.
  **L1232 CN**: 继续处理逻辑：`Scaled64 Gain[2] = {LoopCost[0].PredCost - LoopCost[0].NonPredCost,`。
- **L1233 EN**: Executes statement `LoopCost[1].PredCost - LoopCost[1].NonPredCost};`.
  **L1233 CN**: 执行语句 `LoopCost[1].PredCost - LoopCost[1].NonPredCost};`。
- **L1234 EN**: Separates nearby statements for readability.
  **L1234 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1235 EN**: Comment documents: `Profitably converting to branches need to reduce the loop's critical pat…`.
  **L1235 CN**: 注释说明：`Profitably converting to branches need to reduce the loop's critical pat…`。
- **L1236 EN**: Comment documents: `by at least some threshold (absolute gain of GainCycleThreshold cycles a…`.
  **L1236 CN**: 注释说明：`by at least some threshold (absolute gain of GainCycleThreshold cycles a…`。
- **L1237 EN**: Comment documents: `relative gain of 12.5%).`.
  **L1237 CN**: 注释说明：`relative gain of 12.5%).`。
- **L1238 EN**: Begins a conditional branch.
  **L1238 CN**: 开始一个条件分支。
- **L1239 EN**: Begins the definition of `get`.
  **L1239 CN**: 开始定义 `get`。
- **L1240 EN**: Declares function or method `get`.
  **L1240 CN**: 声明函数或方法 `get`。

### Lines 1241-1260

````cpp
    ORmissL << "No select conversion in the loop due to small reduction of "
               "loop's critical path. Gain="
            << Gain[1].toString()
            << ", RelativeGain=" << RelativeGain.toString() << "%. ";
    EmitAndPrintRemark(ORE, ORmissL);
    return false;
  }

  // If the loop's critical path involves loop-carried dependences, the gradient
  // of the gain needs to be at least GainGradientThreshold% (defaults to 25%).
  // This check ensures that the latency reduction for the loop's critical path
  // keeps decreasing with sufficient rate beyond the two analyzed loop
  // iterations.
  if (Gain[1] > Gain[0]) {
    Scaled64 GradientGain = Scaled64::get(100) * (Gain[1] - Gain[0]) /
                            (LoopCost[1].PredCost - LoopCost[0].PredCost);
    if (GradientGain < Scaled64::get(GainGradientThreshold)) {
      ORmissL << "No select conversion in the loop due to small gradient gain. "
                 "GradientGain="
              << GradientGain.toString() << "%. ";
````
- **L1241 EN**: Continues logic with `ORmissL << "No select conversion in the loop due to small reduction of "`.
  **L1241 CN**: 继续处理逻辑：`ORmissL << "No select conversion in the loop due to small reduction of "`。
- **L1242 EN**: Continues logic with `"loop's critical path. Gain="`.
  **L1242 CN**: 继续处理逻辑：`"loop's critical path. Gain="`。
- **L1243 EN**: Continues logic with `<< Gain[1].toString()`.
  **L1243 CN**: 继续处理逻辑：`<< Gain[1].toString()`。
- **L1244 EN**: Assigns or initializes `<< ", RelativeGain`.
  **L1244 CN**: 对 `<< ", RelativeGain` 进行赋值或初始化。
- **L1245 EN**: Executes statement `EmitAndPrintRemark(ORE, ORmissL);`.
  **L1245 CN**: 执行语句 `EmitAndPrintRemark(ORE, ORmissL);`。
- **L1246 EN**: Returns `false` to the caller.
  **L1246 CN**: 向调用者返回 `false`。
- **L1247 EN**: Closes the current scope.
  **L1247 CN**: 关闭当前作用域。
- **L1248 EN**: Separates nearby statements for readability.
  **L1248 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1249 EN**: Comment documents: `If the loop's critical path involves loop-carried dependences, the gradi…`.
  **L1249 CN**: 注释说明：`If the loop's critical path involves loop-carried dependences, the gradi…`。
- **L1250 EN**: Comment documents: `of the gain needs to be at least GainGradientThreshold% (defaults to 25%…`.
  **L1250 CN**: 注释说明：`of the gain needs to be at least GainGradientThreshold% (defaults to 25%…`。
- **L1251 EN**: Comment documents: `This check ensures that the latency reduction for the loop's critical pa…`.
  **L1251 CN**: 注释说明：`This check ensures that the latency reduction for the loop's critical pa…`。
- **L1252 EN**: Comment documents: `keeps decreasing with sufficient rate beyond the two analyzed loop`.
  **L1252 CN**: 注释说明：`keeps decreasing with sufficient rate beyond the two analyzed loop`。
- **L1253 EN**: Comment documents: `iterations.`.
  **L1253 CN**: 注释说明：`iterations.`。
- **L1254 EN**: Begins a conditional branch.
  **L1254 CN**: 开始一个条件分支。
- **L1255 EN**: Provides part of the signature for `get`.
  **L1255 CN**: 给出 `get` 的一部分签名。
- **L1256 EN**: Executes statement `(LoopCost[1].PredCost - LoopCost[0].PredCost);`.
  **L1256 CN**: 执行语句 `(LoopCost[1].PredCost - LoopCost[0].PredCost);`。
- **L1257 EN**: Begins a conditional branch.
  **L1257 CN**: 开始一个条件分支。
- **L1258 EN**: Continues logic with `ORmissL << "No select conversion in the loop due to small gradient gain.…`.
  **L1258 CN**: 继续处理逻辑：`ORmissL << "No select conversion in the loop due to small gradient gain.…`。
- **L1259 EN**: Continues logic with `"GradientGain="`.
  **L1259 CN**: 继续处理逻辑：`"GradientGain="`。
- **L1260 EN**: Executes statement `<< GradientGain.toString() << "%. ";`.
  **L1260 CN**: 执行语句 `<< GradientGain.toString() << "%. ";`。

### Lines 1261-1280

````cpp
      EmitAndPrintRemark(ORE, ORmissL);
      return false;
    }
  }
  // If the gain decreases it is not profitable to convert.
  else if (Gain[1] < Gain[0]) {
    ORmissL
        << "No select conversion in the loop due to negative gradient gain. ";
    EmitAndPrintRemark(ORE, ORmissL);
    return false;
  }

  // Non-predicated version of the loop is more profitable than its
  // predicated version.
  return true;
}

// Computes instruction and loop-critical-path costs for both the predicated
// and non-predicated version of the given loop.
// Returns false if unable to compute these costs due to invalid cost of loop
````
- **L1261 EN**: Executes statement `EmitAndPrintRemark(ORE, ORmissL);`.
  **L1261 CN**: 执行语句 `EmitAndPrintRemark(ORE, ORmissL);`。
- **L1262 EN**: Returns `false` to the caller.
  **L1262 CN**: 向调用者返回 `false`。
- **L1263 EN**: Closes the current scope.
  **L1263 CN**: 关闭当前作用域。
- **L1264 EN**: Closes the current scope.
  **L1264 CN**: 关闭当前作用域。
- **L1265 EN**: Comment documents: `If the gain decreases it is not profitable to convert.`.
  **L1265 CN**: 注释说明：`If the gain decreases it is not profitable to convert.`。
- **L1266 EN**: Checks an alternate conditional path.
  **L1266 CN**: 检查一个备用条件分支。
- **L1267 EN**: Continues logic with `ORmissL`.
  **L1267 CN**: 继续处理逻辑：`ORmissL`。
- **L1268 EN**: Executes statement `<< "No select conversion in the loop due to negative gradient gain. ";`.
  **L1268 CN**: 执行语句 `<< "No select conversion in the loop due to negative gradient gain. ";`。
- **L1269 EN**: Executes statement `EmitAndPrintRemark(ORE, ORmissL);`.
  **L1269 CN**: 执行语句 `EmitAndPrintRemark(ORE, ORmissL);`。
- **L1270 EN**: Returns `false` to the caller.
  **L1270 CN**: 向调用者返回 `false`。
- **L1271 EN**: Closes the current scope.
  **L1271 CN**: 关闭当前作用域。
- **L1272 EN**: Separates nearby statements for readability.
  **L1272 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1273 EN**: Comment documents: `Non-predicated version of the loop is more profitable than its`.
  **L1273 CN**: 注释说明：`Non-predicated version of the loop is more profitable than its`。
- **L1274 EN**: Comment documents: `predicated version.`.
  **L1274 CN**: 注释说明：`predicated version.`。
- **L1275 EN**: Returns `true` to the caller.
  **L1275 CN**: 向调用者返回 `true`。
- **L1276 EN**: Closes the current scope.
  **L1276 CN**: 关闭当前作用域。
- **L1277 EN**: Separates nearby statements for readability.
  **L1277 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1278 EN**: Comment documents: `Computes instruction and loop-critical-path costs for both the predicate…`.
  **L1278 CN**: 注释说明：`Computes instruction and loop-critical-path costs for both the predicate…`。
- **L1279 EN**: Comment documents: `and non-predicated version of the given loop.`.
  **L1279 CN**: 注释说明：`and non-predicated version of the given loop.`。
- **L1280 EN**: Comment documents: `Returns false if unable to compute these costs due to invalid cost of lo…`.
  **L1280 CN**: 注释说明：`Returns false if unable to compute these costs due to invalid cost of lo…`。

### Lines 1281-1300

````cpp
// instruction(s).
bool SelectOptimizeImpl::computeLoopCosts(
    const Loop *L, const SelectGroups &SIGroups,
    DenseMap<const Instruction *, CostInfo> &InstCostMap, CostInfo *LoopCost) {
  LLVM_DEBUG(dbgs() << "Calculating Latency / IPredCost / INonPredCost of loop "
                    << L->getHeader()->getName() << "\n");
  const auto SImap = getSImap(SIGroups);
  const auto SGmap = getSGmap(SIGroups);
  // Compute instruction and loop-critical-path costs across two iterations for
  // both predicated and non-predicated version.
  const unsigned Iterations = 2;
  for (unsigned Iter = 0; Iter < Iterations; ++Iter) {
    // Cost of the loop's critical path.
    CostInfo &MaxCost = LoopCost[Iter];
    for (BasicBlock *BB : L->getBlocks()) {
      for (const Instruction &I : *BB) {
        if (I.isDebugOrPseudoInst())
          continue;
        // Compute the predicated and non-predicated cost of the instruction.
        Scaled64 IPredCost = Scaled64::getZero(),
````
- **L1281 EN**: Comment documents: `instruction(s).`.
  **L1281 CN**: 注释说明：`instruction(s).`。
- **L1282 EN**: Provides part of the signature for `computeLoopCosts`.
  **L1282 CN**: 给出 `computeLoopCosts` 的一部分签名。
- **L1283 EN**: Continues logic with `const Loop *L, const SelectGroups &SIGroups,`.
  **L1283 CN**: 继续处理逻辑：`const Loop *L, const SelectGroups &SIGroups,`。
- **L1284 EN**: Starts block `DenseMap<const Instruction *, CostInfo> &InstCostMap, CostInfo *LoopCost…`.
  **L1284 CN**: 开始代码块 `DenseMap<const Instruction *, CostInfo> &InstCostMap, CostInfo *LoopCost…`。
- **L1285 EN**: Emits debug-only tracing logic.
  **L1285 CN**: 发出仅在调试时启用的跟踪逻辑。
- **L1286 EN**: Executes statement `<< L->getHeader()->getName() << "\n");`.
  **L1286 CN**: 执行语句 `<< L->getHeader()->getName() << "\n");`。
- **L1287 EN**: Assigns or initializes `const auto SImap`.
  **L1287 CN**: 对 `const auto SImap` 进行赋值或初始化。
- **L1288 EN**: Assigns or initializes `const auto SGmap`.
  **L1288 CN**: 对 `const auto SGmap` 进行赋值或初始化。
- **L1289 EN**: Comment documents: `Compute instruction and loop-critical-path costs across two iterations f…`.
  **L1289 CN**: 注释说明：`Compute instruction and loop-critical-path costs across two iterations f…`。
- **L1290 EN**: Comment documents: `both predicated and non-predicated version.`.
  **L1290 CN**: 注释说明：`both predicated and non-predicated version.`。
- **L1291 EN**: Assigns or initializes `const unsigned Iterations`.
  **L1291 CN**: 对 `const unsigned Iterations` 进行赋值或初始化。
- **L1292 EN**: Starts a loop over a sequence or range.
  **L1292 CN**: 开始遍历序列或范围的循环。
- **L1293 EN**: Comment documents: `Cost of the loop's critical path.`.
  **L1293 CN**: 注释说明：`Cost of the loop's critical path.`。
- **L1294 EN**: Assigns or initializes `CostInfo &MaxCost`.
  **L1294 CN**: 对 `CostInfo &MaxCost` 进行赋值或初始化。
- **L1295 EN**: Starts a loop over a sequence or range.
  **L1295 CN**: 开始遍历序列或范围的循环。
- **L1296 EN**: Starts a loop over a sequence or range.
  **L1296 CN**: 开始遍历序列或范围的循环。
- **L1297 EN**: Begins a conditional branch.
  **L1297 CN**: 开始一个条件分支。
- **L1298 EN**: Skips to the next loop iteration.
  **L1298 CN**: 跳到下一次循环迭代。
- **L1299 EN**: Comment documents: `Compute the predicated and non-predicated cost of the instruction.`.
  **L1299 CN**: 注释说明：`Compute the predicated and non-predicated cost of the instruction.`。
- **L1300 EN**: Provides part of the signature for `getZero`.
  **L1300 CN**: 给出 `getZero` 的一部分签名。

### Lines 1301-1320

````cpp
                 INonPredCost = Scaled64::getZero();

        // Assume infinite resources that allow to fully exploit the available
        // instruction-level parallelism.
        // InstCost = InstLatency + max(Op1Cost, Op2Cost, … OpNCost)
        for (const Use &U : I.operands()) {
          auto UI = dyn_cast<Instruction>(U.get());
          if (!UI)
            continue;
          if (auto It = InstCostMap.find(UI); It != InstCostMap.end()) {
            IPredCost = std::max(IPredCost, It->second.PredCost);
            INonPredCost = std::max(INonPredCost, It->second.NonPredCost);
          }
        }
        auto ILatency = computeInstCost(&I);
        if (!ILatency) {
          OptimizationRemarkMissed ORmissL(DEBUG_TYPE, "SelectOpti", &I);
          ORmissL << "Invalid instruction cost preventing analysis and "
                     "optimization of the inner-most loop containing this "
                     "instruction. ";
````
- **L1301 EN**: Declares function or method `getZero`.
  **L1301 CN**: 声明函数或方法 `getZero`。
- **L1302 EN**: Separates nearby statements for readability.
  **L1302 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1303 EN**: Comment documents: `Assume infinite resources that allow to fully exploit the available`.
  **L1303 CN**: 注释说明：`Assume infinite resources that allow to fully exploit the available`。
- **L1304 EN**: Comment documents: `instruction-level parallelism.`.
  **L1304 CN**: 注释说明：`instruction-level parallelism.`。
- **L1305 EN**: Comment documents: `InstCost = InstLatency + max(Op1Cost, Op2Cost, … OpNCost)`.
  **L1305 CN**: 注释说明：`InstCost = InstLatency + max(Op1Cost, Op2Cost, … OpNCost)`。
- **L1306 EN**: Starts a loop over a sequence or range.
  **L1306 CN**: 开始遍历序列或范围的循环。
- **L1307 EN**: Assigns or initializes `auto UI`.
  **L1307 CN**: 对 `auto UI` 进行赋值或初始化。
- **L1308 EN**: Begins a conditional branch.
  **L1308 CN**: 开始一个条件分支。
- **L1309 EN**: Skips to the next loop iteration.
  **L1309 CN**: 跳到下一次循环迭代。
- **L1310 EN**: Begins a conditional branch.
  **L1310 CN**: 开始一个条件分支。
- **L1311 EN**: Declares function or method `max`.
  **L1311 CN**: 声明函数或方法 `max`。
- **L1312 EN**: Declares function or method `max`.
  **L1312 CN**: 声明函数或方法 `max`。
- **L1313 EN**: Closes the current scope.
  **L1313 CN**: 关闭当前作用域。
- **L1314 EN**: Closes the current scope.
  **L1314 CN**: 关闭当前作用域。
- **L1315 EN**: Assigns or initializes `auto ILatency`.
  **L1315 CN**: 对 `auto ILatency` 进行赋值或初始化。
- **L1316 EN**: Begins a conditional branch.
  **L1316 CN**: 开始一个条件分支。
- **L1317 EN**: Declares function or method `ORmissL`.
  **L1317 CN**: 声明函数或方法 `ORmissL`。
- **L1318 EN**: Continues logic with `ORmissL << "Invalid instruction cost preventing analysis and "`.
  **L1318 CN**: 继续处理逻辑：`ORmissL << "Invalid instruction cost preventing analysis and "`。
- **L1319 EN**: Continues logic with `"optimization of the inner-most loop containing this "`.
  **L1319 CN**: 继续处理逻辑：`"optimization of the inner-most loop containing this "`。
- **L1320 EN**: Executes statement `"instruction. ";`.
  **L1320 CN**: 执行语句 `"instruction. ";`。

### Lines 1321-1340

````cpp
          EmitAndPrintRemark(ORE, ORmissL);
          return false;
        }
        IPredCost += Scaled64::get(*ILatency);
        INonPredCost += Scaled64::get(*ILatency);

        // For a select that can be converted to branch,
        // compute its cost as a branch (non-predicated cost).
        //
        // BranchCost = PredictedPathCost + MispredictCost
        // PredictedPathCost = TrueOpCost * TrueProb + FalseOpCost * FalseProb
        // MispredictCost = max(MispredictPenalty, CondCost) * MispredictRate
        if (auto It = SImap.find(&I); It != SImap.end()) {
          auto SI = It->second;
          const auto *SG = SGmap.at(&I);
          Scaled64 TrueOpCost = SI.getOpCostOnBranch(true, InstCostMap, TTI);
          Scaled64 FalseOpCost = SI.getOpCostOnBranch(false, InstCostMap, TTI);
          Scaled64 PredictedPathCost =
              getPredictedPathCost(TrueOpCost, FalseOpCost, SI);

````
- **L1321 EN**: Executes statement `EmitAndPrintRemark(ORE, ORmissL);`.
  **L1321 CN**: 执行语句 `EmitAndPrintRemark(ORE, ORmissL);`。
- **L1322 EN**: Returns `false` to the caller.
  **L1322 CN**: 向调用者返回 `false`。
- **L1323 EN**: Closes the current scope.
  **L1323 CN**: 关闭当前作用域。
- **L1324 EN**: Declares function or method `get`.
  **L1324 CN**: 声明函数或方法 `get`。
- **L1325 EN**: Declares function or method `get`.
  **L1325 CN**: 声明函数或方法 `get`。
- **L1326 EN**: Separates nearby statements for readability.
  **L1326 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1327 EN**: Comment documents: `For a select that can be converted to branch,`.
  **L1327 CN**: 注释说明：`For a select that can be converted to branch,`。
- **L1328 EN**: Comment documents: `compute its cost as a branch (non-predicated cost).`.
  **L1328 CN**: 注释说明：`compute its cost as a branch (non-predicated cost).`。
- **L1329 EN**: Continues the surrounding comment block.
  **L1329 CN**: 延续周围的注释块。
- **L1330 EN**: Comment documents: `BranchCost = PredictedPathCost + MispredictCost`.
  **L1330 CN**: 注释说明：`BranchCost = PredictedPathCost + MispredictCost`。
- **L1331 EN**: Comment documents: `PredictedPathCost = TrueOpCost * TrueProb + FalseOpCost * FalseProb`.
  **L1331 CN**: 注释说明：`PredictedPathCost = TrueOpCost * TrueProb + FalseOpCost * FalseProb`。
- **L1332 EN**: Comment documents: `MispredictCost = max(MispredictPenalty, CondCost) * MispredictRate`.
  **L1332 CN**: 注释说明：`MispredictCost = max(MispredictPenalty, CondCost) * MispredictRate`。
- **L1333 EN**: Begins a conditional branch.
  **L1333 CN**: 开始一个条件分支。
- **L1334 EN**: Assigns or initializes `auto SI`.
  **L1334 CN**: 对 `auto SI` 进行赋值或初始化。
- **L1335 EN**: Assigns or initializes `const auto *SG`.
  **L1335 CN**: 对 `const auto *SG` 进行赋值或初始化。
- **L1336 EN**: Assigns or initializes `Scaled64 TrueOpCost`.
  **L1336 CN**: 对 `Scaled64 TrueOpCost` 进行赋值或初始化。
- **L1337 EN**: Assigns or initializes `Scaled64 FalseOpCost`.
  **L1337 CN**: 对 `Scaled64 FalseOpCost` 进行赋值或初始化。
- **L1338 EN**: Continues logic with `Scaled64 PredictedPathCost =`.
  **L1338 CN**: 继续处理逻辑：`Scaled64 PredictedPathCost =`。
- **L1339 EN**: Executes statement `getPredictedPathCost(TrueOpCost, FalseOpCost, SI);`.
  **L1339 CN**: 执行语句 `getPredictedPathCost(TrueOpCost, FalseOpCost, SI);`。
- **L1340 EN**: Separates nearby statements for readability.
  **L1340 CN**: 空行，用于分隔相邻语句并提升可读性。

### Lines 1341-1360

````cpp
          Scaled64 CondCost = Scaled64::getZero();
          if (auto *CI = dyn_cast<Instruction>(SG->Condition))
            if (auto It = InstCostMap.find(CI); It != InstCostMap.end())
              CondCost = It->second.NonPredCost;
          Scaled64 MispredictCost = getMispredictionCost(SI, CondCost);

          INonPredCost = PredictedPathCost + MispredictCost;
        }
        LLVM_DEBUG(dbgs() << " " << ILatency << "/" << IPredCost << "/"
                          << INonPredCost << " for " << I << "\n");

        InstCostMap[&I] = {IPredCost, INonPredCost};
        MaxCost.PredCost = std::max(MaxCost.PredCost, IPredCost);
        MaxCost.NonPredCost = std::max(MaxCost.NonPredCost, INonPredCost);
      }
    }
    LLVM_DEBUG(dbgs() << "Iteration " << Iter + 1
                      << " MaxCost = " << MaxCost.PredCost << " "
                      << MaxCost.NonPredCost << "\n");
  }
````
- **L1341 EN**: Declares function or method `getZero`.
  **L1341 CN**: 声明函数或方法 `getZero`。
- **L1342 EN**: Begins a conditional branch.
  **L1342 CN**: 开始一个条件分支。
- **L1343 EN**: Begins a conditional branch.
  **L1343 CN**: 开始一个条件分支。
- **L1344 EN**: Assigns or initializes `CondCost`.
  **L1344 CN**: 对 `CondCost` 进行赋值或初始化。
- **L1345 EN**: Assigns or initializes `Scaled64 MispredictCost`.
  **L1345 CN**: 对 `Scaled64 MispredictCost` 进行赋值或初始化。
- **L1346 EN**: Separates nearby statements for readability.
  **L1346 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1347 EN**: Assigns or initializes `INonPredCost`.
  **L1347 CN**: 对 `INonPredCost` 进行赋值或初始化。
- **L1348 EN**: Closes the current scope.
  **L1348 CN**: 关闭当前作用域。
- **L1349 EN**: Emits debug-only tracing logic.
  **L1349 CN**: 发出仅在调试时启用的跟踪逻辑。
- **L1350 EN**: Executes statement `<< INonPredCost << " for " << I << "\n");`.
  **L1350 CN**: 执行语句 `<< INonPredCost << " for " << I << "\n");`。
- **L1351 EN**: Separates nearby statements for readability.
  **L1351 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1352 EN**: Assigns or initializes `InstCostMap[&I]`.
  **L1352 CN**: 对 `InstCostMap[&I]` 进行赋值或初始化。
- **L1353 EN**: Declares function or method `max`.
  **L1353 CN**: 声明函数或方法 `max`。
- **L1354 EN**: Declares function or method `max`.
  **L1354 CN**: 声明函数或方法 `max`。
- **L1355 EN**: Closes the current scope.
  **L1355 CN**: 关闭当前作用域。
- **L1356 EN**: Closes the current scope.
  **L1356 CN**: 关闭当前作用域。
- **L1357 EN**: Emits debug-only tracing logic.
  **L1357 CN**: 发出仅在调试时启用的跟踪逻辑。
- **L1358 EN**: Continues logic with `<< " MaxCost = " << MaxCost.PredCost << " "`.
  **L1358 CN**: 继续处理逻辑：`<< " MaxCost = " << MaxCost.PredCost << " "`。
- **L1359 EN**: Executes statement `<< MaxCost.NonPredCost << "\n");`.
  **L1359 CN**: 执行语句 `<< MaxCost.NonPredCost << "\n");`。
- **L1360 EN**: Closes the current scope.
  **L1360 CN**: 关闭当前作用域。

### Lines 1361-1380

````cpp
  return true;
}

SmallDenseMap<const Instruction *, SelectOptimizeImpl::SelectLike, 2>
SelectOptimizeImpl::getSImap(const SelectGroups &SIGroups) {
  SmallDenseMap<const Instruction *, SelectLike, 2> SImap;
  for (const SelectGroup &ASI : SIGroups)
    for (const SelectLike &SI : ASI.Selects)
      SImap.try_emplace(SI.getI(), SI);
  return SImap;
}

SmallDenseMap<const Instruction *, const SelectOptimizeImpl::SelectGroup *, 2>
SelectOptimizeImpl::getSGmap(const SelectGroups &SIGroups) {
  SmallDenseMap<const Instruction *, const SelectGroup *, 2> SImap;
  for (const SelectGroup &ASI : SIGroups)
    for (const SelectLike &SI : ASI.Selects)
      SImap.try_emplace(SI.getI(), &ASI);
  return SImap;
}
````
- **L1361 EN**: Returns `true` to the caller.
  **L1361 CN**: 向调用者返回 `true`。
- **L1362 EN**: Closes the current scope.
  **L1362 CN**: 关闭当前作用域。
- **L1363 EN**: Separates nearby statements for readability.
  **L1363 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1364 EN**: Continues logic with `SmallDenseMap<const Instruction *, SelectOptimizeImpl::SelectLike, 2>`.
  **L1364 CN**: 继续处理逻辑：`SmallDenseMap<const Instruction *, SelectOptimizeImpl::SelectLike, 2>`。
- **L1365 EN**: Begins the definition of `getSImap`.
  **L1365 CN**: 开始定义 `getSImap`。
- **L1366 EN**: Executes statement `SmallDenseMap<const Instruction *, SelectLike, 2> SImap;`.
  **L1366 CN**: 执行语句 `SmallDenseMap<const Instruction *, SelectLike, 2> SImap;`。
- **L1367 EN**: Starts a loop over a sequence or range.
  **L1367 CN**: 开始遍历序列或范围的循环。
- **L1368 EN**: Starts a loop over a sequence or range.
  **L1368 CN**: 开始遍历序列或范围的循环。
- **L1369 EN**: Executes statement `SImap.try_emplace(SI.getI(), SI);`.
  **L1369 CN**: 执行语句 `SImap.try_emplace(SI.getI(), SI);`。
- **L1370 EN**: Returns `SImap` to the caller.
  **L1370 CN**: 向调用者返回 `SImap`。
- **L1371 EN**: Closes the current scope.
  **L1371 CN**: 关闭当前作用域。
- **L1372 EN**: Separates nearby statements for readability.
  **L1372 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1373 EN**: Continues logic with `SmallDenseMap<const Instruction *, const SelectOptimizeImpl::SelectGroup…`.
  **L1373 CN**: 继续处理逻辑：`SmallDenseMap<const Instruction *, const SelectOptimizeImpl::SelectGroup…`。
- **L1374 EN**: Begins the definition of `getSGmap`.
  **L1374 CN**: 开始定义 `getSGmap`。
- **L1375 EN**: Executes statement `SmallDenseMap<const Instruction *, const SelectGroup *, 2> SImap;`.
  **L1375 CN**: 执行语句 `SmallDenseMap<const Instruction *, const SelectGroup *, 2> SImap;`。
- **L1376 EN**: Starts a loop over a sequence or range.
  **L1376 CN**: 开始遍历序列或范围的循环。
- **L1377 EN**: Starts a loop over a sequence or range.
  **L1377 CN**: 开始遍历序列或范围的循环。
- **L1378 EN**: Executes statement `SImap.try_emplace(SI.getI(), &ASI);`.
  **L1378 CN**: 执行语句 `SImap.try_emplace(SI.getI(), &ASI);`。
- **L1379 EN**: Returns `SImap` to the caller.
  **L1379 CN**: 向调用者返回 `SImap`。
- **L1380 EN**: Closes the current scope.
  **L1380 CN**: 关闭当前作用域。

### Lines 1381-1400

````cpp

std::optional<uint64_t>
SelectOptimizeImpl::computeInstCost(const Instruction *I) {
  InstructionCost ICost =
      TTI->getInstructionCost(I, TargetTransformInfo::TCK_Latency);
  if (ICost.isValid())
    return std::optional<uint64_t>(ICost.getValue());
  return std::nullopt;
}

ScaledNumber<uint64_t>
SelectOptimizeImpl::getMispredictionCost(const SelectLike SI,
                                         const Scaled64 CondCost) {
  uint64_t MispredictPenalty = TSchedModel.getMCSchedModel()->MispredictPenalty;

  // Account for the default misprediction rate when using a branch
  // (conservatively set to 25% by default).
  uint64_t MispredictRate = MispredictDefaultRate;
  // If the select condition is obviously predictable, then the misprediction
  // rate is zero.
````
- **L1381 EN**: Separates nearby statements for readability.
  **L1381 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1382 EN**: Continues logic with `std::optional<uint64_t>`.
  **L1382 CN**: 继续处理逻辑：`std::optional<uint64_t>`。
- **L1383 EN**: Begins the definition of `computeInstCost`.
  **L1383 CN**: 开始定义 `computeInstCost`。
- **L1384 EN**: Continues logic with `InstructionCost ICost =`.
  **L1384 CN**: 继续处理逻辑：`InstructionCost ICost =`。
- **L1385 EN**: Executes statement `TTI->getInstructionCost(I, TargetTransformInfo::TCK_Latency);`.
  **L1385 CN**: 执行语句 `TTI->getInstructionCost(I, TargetTransformInfo::TCK_Latency);`。
- **L1386 EN**: Begins a conditional branch.
  **L1386 CN**: 开始一个条件分支。
- **L1387 EN**: Returns `std::optional<uint64_t>(ICost.getValue())` to the caller.
  **L1387 CN**: 向调用者返回 `std::optional<uint64_t>(ICost.getValue())`。
- **L1388 EN**: Returns `std::nullopt` to the caller.
  **L1388 CN**: 向调用者返回 `std::nullopt`。
- **L1389 EN**: Closes the current scope.
  **L1389 CN**: 关闭当前作用域。
- **L1390 EN**: Separates nearby statements for readability.
  **L1390 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1391 EN**: Continues logic with `ScaledNumber<uint64_t>`.
  **L1391 CN**: 继续处理逻辑：`ScaledNumber<uint64_t>`。
- **L1392 EN**: Provides part of the signature for `getMispredictionCost`.
  **L1392 CN**: 给出 `getMispredictionCost` 的一部分签名。
- **L1393 EN**: Starts block `const Scaled64 CondCost)`.
  **L1393 CN**: 开始代码块 `const Scaled64 CondCost)`。
- **L1394 EN**: Assigns or initializes `uint64_t MispredictPenalty`.
  **L1394 CN**: 对 `uint64_t MispredictPenalty` 进行赋值或初始化。
- **L1395 EN**: Separates nearby statements for readability.
  **L1395 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1396 EN**: Comment documents: `Account for the default misprediction rate when using a branch`.
  **L1396 CN**: 注释说明：`Account for the default misprediction rate when using a branch`。
- **L1397 EN**: Comment documents: `(conservatively set to 25% by default).`.
  **L1397 CN**: 注释说明：`(conservatively set to 25% by default).`。
- **L1398 EN**: Assigns or initializes `uint64_t MispredictRate`.
  **L1398 CN**: 对 `uint64_t MispredictRate` 进行赋值或初始化。
- **L1399 EN**: Comment documents: `If the select condition is obviously predictable, then the misprediction`.
  **L1399 CN**: 注释说明：`If the select condition is obviously predictable, then the misprediction`。
- **L1400 EN**: Comment documents: `rate is zero.`.
  **L1400 CN**: 注释说明：`rate is zero.`。

### Lines 1401-1420

````cpp
  if (isSelectHighlyPredictable(SI))
    MispredictRate = 0;

  // CondCost is included to account for cases where the computation of the
  // condition is part of a long dependence chain (potentially loop-carried)
  // that would delay detection of a misprediction and increase its cost.
  Scaled64 MispredictCost =
      std::max(Scaled64::get(MispredictPenalty), CondCost) *
      Scaled64::get(MispredictRate);
  MispredictCost /= Scaled64::get(100);

  return MispredictCost;
}

// Returns the cost of a branch when the prediction is correct.
// TrueCost * TrueProbability + FalseCost * FalseProbability.
ScaledNumber<uint64_t>
SelectOptimizeImpl::getPredictedPathCost(Scaled64 TrueCost, Scaled64 FalseCost,
                                         const SelectLike SI) {
  Scaled64 PredPathCost;
````
- **L1401 EN**: Begins a conditional branch.
  **L1401 CN**: 开始一个条件分支。
- **L1402 EN**: Assigns or initializes `MispredictRate`.
  **L1402 CN**: 对 `MispredictRate` 进行赋值或初始化。
- **L1403 EN**: Separates nearby statements for readability.
  **L1403 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1404 EN**: Comment documents: `CondCost is included to account for cases where the computation of the`.
  **L1404 CN**: 注释说明：`CondCost is included to account for cases where the computation of the`。
- **L1405 EN**: Comment documents: `condition is part of a long dependence chain (potentially loop-carried)`.
  **L1405 CN**: 注释说明：`condition is part of a long dependence chain (potentially loop-carried)`。
- **L1406 EN**: Comment documents: `that would delay detection of a misprediction and increase its cost.`.
  **L1406 CN**: 注释说明：`that would delay detection of a misprediction and increase its cost.`。
- **L1407 EN**: Continues logic with `Scaled64 MispredictCost =`.
  **L1407 CN**: 继续处理逻辑：`Scaled64 MispredictCost =`。
- **L1408 EN**: Provides part of the signature for `max`.
  **L1408 CN**: 给出 `max` 的一部分签名。
- **L1409 EN**: Declares function or method `get`.
  **L1409 CN**: 声明函数或方法 `get`。
- **L1410 EN**: Declares function or method `get`.
  **L1410 CN**: 声明函数或方法 `get`。
- **L1411 EN**: Separates nearby statements for readability.
  **L1411 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1412 EN**: Returns `MispredictCost` to the caller.
  **L1412 CN**: 向调用者返回 `MispredictCost`。
- **L1413 EN**: Closes the current scope.
  **L1413 CN**: 关闭当前作用域。
- **L1414 EN**: Separates nearby statements for readability.
  **L1414 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1415 EN**: Comment documents: `Returns the cost of a branch when the prediction is correct.`.
  **L1415 CN**: 注释说明：`Returns the cost of a branch when the prediction is correct.`。
- **L1416 EN**: Comment documents: `TrueCost * TrueProbability + FalseCost * FalseProbability.`.
  **L1416 CN**: 注释说明：`TrueCost * TrueProbability + FalseCost * FalseProbability.`。
- **L1417 EN**: Continues logic with `ScaledNumber<uint64_t>`.
  **L1417 CN**: 继续处理逻辑：`ScaledNumber<uint64_t>`。
- **L1418 EN**: Provides part of the signature for `getPredictedPathCost`.
  **L1418 CN**: 给出 `getPredictedPathCost` 的一部分签名。
- **L1419 EN**: Starts block `const SelectLike SI)`.
  **L1419 CN**: 开始代码块 `const SelectLike SI)`。
- **L1420 EN**: Executes statement `Scaled64 PredPathCost;`.
  **L1420 CN**: 执行语句 `Scaled64 PredPathCost;`。

### Lines 1421-1440

````cpp
  uint64_t TrueWeight, FalseWeight;
  if (extractBranchWeights(SI, TrueWeight, FalseWeight)) {
    uint64_t SumWeight = TrueWeight + FalseWeight;
    if (SumWeight != 0) {
      PredPathCost = TrueCost * Scaled64::get(TrueWeight) +
                     FalseCost * Scaled64::get(FalseWeight);
      PredPathCost /= Scaled64::get(SumWeight);
      return PredPathCost;
    }
  }
  // Without branch weight metadata, we assume 75% for the one path and 25% for
  // the other, and pick the result with the biggest cost.
  PredPathCost = std::max(TrueCost * Scaled64::get(3) + FalseCost,
                          FalseCost * Scaled64::get(3) + TrueCost);
  PredPathCost /= Scaled64::get(4);
  return PredPathCost;
}

bool SelectOptimizeImpl::isSelectKindSupported(const SelectLike SI) {
  TargetLowering::SelectSupportKind SelectKind;
````
- **L1421 EN**: Executes statement `uint64_t TrueWeight, FalseWeight;`.
  **L1421 CN**: 执行语句 `uint64_t TrueWeight, FalseWeight;`。
- **L1422 EN**: Begins a conditional branch.
  **L1422 CN**: 开始一个条件分支。
- **L1423 EN**: Assigns or initializes `uint64_t SumWeight`.
  **L1423 CN**: 对 `uint64_t SumWeight` 进行赋值或初始化。
- **L1424 EN**: Begins a conditional branch.
  **L1424 CN**: 开始一个条件分支。
- **L1425 EN**: Provides part of the signature for `get`.
  **L1425 CN**: 给出 `get` 的一部分签名。
- **L1426 EN**: Declares function or method `get`.
  **L1426 CN**: 声明函数或方法 `get`。
- **L1427 EN**: Declares function or method `get`.
  **L1427 CN**: 声明函数或方法 `get`。
- **L1428 EN**: Returns `PredPathCost` to the caller.
  **L1428 CN**: 向调用者返回 `PredPathCost`。
- **L1429 EN**: Closes the current scope.
  **L1429 CN**: 关闭当前作用域。
- **L1430 EN**: Closes the current scope.
  **L1430 CN**: 关闭当前作用域。
- **L1431 EN**: Comment documents: `Without branch weight metadata, we assume 75% for the one path and 25% f…`.
  **L1431 CN**: 注释说明：`Without branch weight metadata, we assume 75% for the one path and 25% f…`。
- **L1432 EN**: Comment documents: `the other, and pick the result with the biggest cost.`.
  **L1432 CN**: 注释说明：`the other, and pick the result with the biggest cost.`。
- **L1433 EN**: Provides part of the signature for `max`.
  **L1433 CN**: 给出 `max` 的一部分签名。
- **L1434 EN**: Declares function or method `get`.
  **L1434 CN**: 声明函数或方法 `get`。
- **L1435 EN**: Declares function or method `get`.
  **L1435 CN**: 声明函数或方法 `get`。
- **L1436 EN**: Returns `PredPathCost` to the caller.
  **L1436 CN**: 向调用者返回 `PredPathCost`。
- **L1437 EN**: Closes the current scope.
  **L1437 CN**: 关闭当前作用域。
- **L1438 EN**: Separates nearby statements for readability.
  **L1438 CN**: 空行，用于分隔相邻语句并提升可读性。
- **L1439 EN**: Begins the definition of `isSelectKindSupported`.
  **L1439 CN**: 开始定义 `isSelectKindSupported`。
- **L1440 EN**: Executes statement `TargetLowering::SelectSupportKind SelectKind;`.
  **L1440 CN**: 执行语句 `TargetLowering::SelectSupportKind SelectKind;`。

### Lines 1441-1446

````cpp
  if (SI.getType()->isVectorTy())
    SelectKind = TargetLowering::ScalarCondVectorVal;
  else
    SelectKind = TargetLowering::ScalarValSelect;
  return TLI->isSelectSupported(SelectKind);
}
````
- **L1441 EN**: Begins a conditional branch.
  **L1441 CN**: 开始一个条件分支。
- **L1442 EN**: Assigns or initializes `SelectKind`.
  **L1442 CN**: 对 `SelectKind` 进行赋值或初始化。
- **L1443 EN**: Handles the fallback branch.
  **L1443 CN**: 处理兜底分支。
- **L1444 EN**: Assigns or initializes `SelectKind`.
  **L1444 CN**: 对 `SelectKind` 进行赋值或初始化。
- **L1445 EN**: Returns `TLI->isSelectSupported(SelectKind)` to the caller.
  **L1445 CN**: 向调用者返回 `TLI->isSelectSupported(SelectKind)`。
- **L1446 EN**: Closes the current scope.
  **L1446 CN**: 关闭当前作用域。

## Key Concepts / 关键概念
- **Instruction scheduling** / **指令调度**
- **Register management** / **寄存器管理**
- **Register pressure tracking** / **寄存器压力跟踪**
- **Stack frame management** / **栈帧管理**
- **Control-flow updates** / **控制流更新**
- **Debug information handling** / **调试信息处理**
- **SSA/PHI lowering** / **SSA/PHI 降低**
- **Target-specific hooks** / **目标相关钩子**

## Dependencies / 依赖关系
- **LLVM headers / LLVM 头文件**: `llvm/CodeGen/SelectOptimize.h`, `llvm/ADT/SetVector.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/Analysis/BlockFrequencyInfo.h`, `llvm/Analysis/BranchProbabilityInfo.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/ProfileSummaryInfo.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/CodeGen/Passes.h`, `llvm/CodeGen/TargetLowering.h`, `llvm/CodeGen/TargetPassConfig.h`, `llvm/CodeGen/TargetSchedule.h`, `llvm/CodeGen/TargetSubtargetInfo.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instruction.h`, `llvm/IR/PatternMatch.h`, `llvm/IR/ProfDataUtils.h`, `llvm/InitializePasses.h`, `llvm/Pass.h`, `llvm/Support/ScaledNumber.h`, and 2 more / 以及另外 2 个
- **System headers / 系统头文件**: `algorithm`, `queue`, `stack`
- **Primary dependency domains / 主要依赖域**: CodeGen internals, LLVM support utilities, and C++ runtime helpers. / CodeGen 内部组件、LLVM 支持工具以及 C++ 运行时辅助设施。
