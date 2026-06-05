# LowerAllowCheckPass.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/Transforms/Instrumentation/LowerAllowCheckPass.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**: Implements the LLVM transform logic, helper routines, and pass plumbing for LowerAllowCheckPass. / 该文件位于 `Transforms/Instrumentation`，主要实现 `LowerAllowCheckPass` 相关的 LLVM 变换逻辑、辅助流程以及 pass 接线代码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
//===- LowerAllowCheckPass.cpp ----------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Instrumentation/LowerAllowCheckPass.h"

#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/Analysis/OptimizationRemarkEmitter.h"
#include "llvm/Analysis/ProfileSummaryInfo.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/DiagnosticInfo.h"
#include "llvm/IR/InstIterator.h"
#include "llvm/IR/Instructions.h"
```

- **L1**: Banner comment marking a file section boundary. / 横幅注释，用于标记文件分节。
- **L2**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L3**: Comment documents the nearby logic or transformation intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑或变换意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment documents the nearby logic or transformation intent: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑或变换意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment documents the nearby logic or transformation intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑或变换意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L7**: Banner comment marking a file section boundary. / 横幅注释，用于标记文件分节。
- **L8**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L9**: Includes "llvm/Transforms/Instrumentation/LowerAllowCheckPass.h" to access transform-specific declarations. / 引入 "llvm/Transforms/Instrumentation/LowerAllowCheckPass.h" 以使用变换相关声明。
- **L10**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L11**: Includes "llvm/ADT/SmallVector.h" to access LLVM ADT data structures/utilities. / 引入 "llvm/ADT/SmallVector.h" 以使用LLVM ADT 数据结构/工具。
- **L12**: Includes "llvm/ADT/Statistic.h" to access LLVM ADT data structures/utilities. / 引入 "llvm/ADT/Statistic.h" 以使用LLVM ADT 数据结构/工具。
- **L13**: Includes "llvm/ADT/StringExtras.h" to access LLVM ADT data structures/utilities. / 引入 "llvm/ADT/StringExtras.h" 以使用LLVM ADT 数据结构/工具。
- **L14**: Includes "llvm/ADT/StringRef.h" to access LLVM ADT data structures/utilities. / 引入 "llvm/ADT/StringRef.h" 以使用LLVM ADT 数据结构/工具。
- **L15**: Includes "llvm/Analysis/OptimizationRemarkEmitter.h" to access analysis interfaces and cached results. / 引入 "llvm/Analysis/OptimizationRemarkEmitter.h" 以使用分析接口与缓存结果。
- **L16**: Includes "llvm/Analysis/ProfileSummaryInfo.h" to access analysis interfaces and cached results. / 引入 "llvm/Analysis/ProfileSummaryInfo.h" 以使用分析接口与缓存结果。
- **L17**: Includes "llvm/IR/Constants.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/Constants.h" 以使用LLVM IR 核心类型与构造工具。
- **L18**: Includes "llvm/IR/DiagnosticInfo.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/DiagnosticInfo.h" 以使用LLVM IR 核心类型与构造工具。
- **L19**: Includes "llvm/IR/InstIterator.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/InstIterator.h" 以使用LLVM IR 核心类型与构造工具。
- **L20**: Includes "llvm/IR/Instructions.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/Instructions.h" 以使用LLVM IR 核心类型与构造工具。

### Lines 21-40

```cpp
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/Intrinsics.h"
#include "llvm/IR/Metadata.h"
#include "llvm/IR/Module.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/RandomNumberGenerator.h"
#include <memory>
#include <optional>
#include <random>

using namespace llvm;

#define DEBUG_TYPE "lower-allow-check"

static cl::opt<int>
    HotPercentileCutoff("lower-allow-check-percentile-cutoff-hot",
                        cl::desc("Hot percentile cutoff."));

static cl::opt<float>
    RandomRate("lower-allow-check-random-rate",
```

- **L21**: Includes "llvm/IR/IntrinsicInst.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/IntrinsicInst.h" 以使用LLVM IR 核心类型与构造工具。
- **L22**: Includes "llvm/IR/Intrinsics.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/Intrinsics.h" 以使用LLVM IR 核心类型与构造工具。
- **L23**: Includes "llvm/IR/Metadata.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/Metadata.h" 以使用LLVM IR 核心类型与构造工具。
- **L24**: Includes "llvm/IR/Module.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/Module.h" 以使用LLVM IR 核心类型与构造工具。
- **L25**: Includes "llvm/Support/Debug.h" to access support-library helpers. / 引入 "llvm/Support/Debug.h" 以使用Support 库辅助功能。
- **L26**: Includes "llvm/Support/RandomNumberGenerator.h" to access support-library helpers. / 引入 "llvm/Support/RandomNumberGenerator.h" 以使用Support 库辅助功能。
- **L27**: Includes <memory> to access supporting declarations. / 引入 <memory> 以使用所需的辅助声明。
- **L28**: Includes <optional> to access supporting declarations. / 引入 <optional> 以使用所需的辅助声明。
- **L29**: Includes <random> to access supporting declarations. / 引入 <random> 以使用所需的辅助声明。
- **L30**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L31**: Brings namespace `llvm` into the local scope. / 将命名空间 `llvm` 引入当前作用域。
- **L32**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L33**: Defines macro `DEBUG_TYPE` for later conditional logic, flags, or diagnostics. / 定义宏 `DEBUG_TYPE`，供后续条件逻辑、标志位或诊断使用。
- **L34**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L35**: Declares a command-line option or tunable parameter: `static cl::opt<int>`. / 声明一个命令行选项或可调参数：`static cl::opt<int>`。
- **L36**: Continues a multi-line argument list or initializer: `HotPercentileCutoff("lower-allow-check-percentile-cutoff-hot",`. / 继续一个多行参数列表或初始化器：`HotPercentileCutoff("lower-allow-check-percentile-cutoff-hot",`。
- **L37**: Executes call or statement centered on `cl::desc`. / 执行以 `cl::desc` 为核心的调用或语句。
- **L38**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L39**: Declares a command-line option or tunable parameter: `static cl::opt<float>`. / 声明一个命令行选项或可调参数：`static cl::opt<float>`。
- **L40**: Continues a multi-line argument list or initializer: `RandomRate("lower-allow-check-random-rate",`. / 继续一个多行参数列表或初始化器：`RandomRate("lower-allow-check-random-rate",`。

### Lines 41-60

```cpp
               cl::desc("Probability value in the range [0.0, 1.0] of "
                        "unconditional pseudo-random checks."));

STATISTIC(NumChecksTotal, "Number of checks");
STATISTIC(NumChecksRemoved, "Number of removed checks");

struct RemarkInfo {
  ore::NV Kind;
  ore::NV F;
  ore::NV BB;
  explicit RemarkInfo(IntrinsicInst *II)
      : Kind("Kind", II->getArgOperand(0)),
        F("Function", II->getParent()->getParent()),
        BB("Block", II->getParent()->getName()) {}
};

static void emitRemark(IntrinsicInst *II, OptimizationRemarkEmitter &ORE,
                       bool Removed) {
  if (Removed) {
    ORE.emit([&]() {
```

- **L41**: Continues the surrounding expression or declaration: `cl::desc("Probability value in the range [0.0, 1.0] of "`. / 继续构造周围的表达式或声明：`cl::desc("Probability value in the range [0.0, 1.0] of "`。
- **L42**: Executes a standalone statement or declaration: `"unconditional pseudo-random checks."));`. / 执行一条独立语句或声明：`"unconditional pseudo-random checks."));`。
- **L43**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L44**: Registers LLVM statistic counter `NumChecksTotal`. / 注册 LLVM 统计计数器 `NumChecksTotal`。
- **L45**: Registers LLVM statistic counter `NumChecksRemoved`. / 注册 LLVM 统计计数器 `NumChecksRemoved`。
- **L46**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L47**: Declares struct `RemarkInfo`. / 声明 struct `RemarkInfo`。
- **L48**: Executes a standalone statement or declaration: `ore::NV Kind;`. / 执行一条独立语句或声明：`ore::NV Kind;`。
- **L49**: Executes a standalone statement or declaration: `ore::NV F;`. / 执行一条独立语句或声明：`ore::NV F;`。
- **L50**: Executes a standalone statement or declaration: `ore::NV BB;`. / 执行一条独立语句或声明：`ore::NV BB;`。
- **L51**: Continues the surrounding expression or declaration: `explicit RemarkInfo(IntrinsicInst *II)`. / 继续构造周围的表达式或声明：`explicit RemarkInfo(IntrinsicInst *II)`。
- **L52**: Continues a multi-line argument list or initializer: `: Kind("Kind", II->getArgOperand(0)),`. / 继续一个多行参数列表或初始化器：`: Kind("Kind", II->getArgOperand(0)),`。
- **L53**: Continues a multi-line argument list or initializer: `F("Function", II->getParent()->getParent()),`. / 继续一个多行参数列表或初始化器：`F("Function", II->getParent()->getParent()),`。
- **L54**: Continues the surrounding expression or declaration: `BB("Block", II->getParent()->getName()) {}`. / 继续构造周围的表达式或声明：`BB("Block", II->getParent()->getName()) {}`。
- **L55**: Closes the current declaration scope, such as a class or struct. / 结束当前声明作用域，例如类或结构体。
- **L56**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L57**: Continues a multi-line argument list or initializer: `static void emitRemark(IntrinsicInst *II, OptimizationRemarkEmitter &ORE,`. / 继续一个多行参数列表或初始化器：`static void emitRemark(IntrinsicInst *II, OptimizationRemarkEmitter &ORE,`。
- **L58**: Continues the surrounding expression or declaration: `bool Removed) {`. / 继续构造周围的表达式或声明：`bool Removed) {`。
- **L59**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L60**: Starts a function, method, or lambda body: `ORE.emit([&]() {`. / 开始一个函数、方法或 lambda 的主体：`ORE.emit([&]() {`。

### Lines 61-80

```cpp
      RemarkInfo Info(II);
      return OptimizationRemark(DEBUG_TYPE, "Removed", II)
             << "Removed check: Kind=" << Info.Kind << " F=" << Info.F
             << " BB=" << Info.BB;
    });
  } else {
    ORE.emit([&]() {
      RemarkInfo Info(II);
      return OptimizationRemarkMissed(DEBUG_TYPE, "Allowed", II)
             << "Allowed check: Kind=" << Info.Kind << " F=" << Info.F
             << " BB=" << Info.BB;
    });
  }
}

static bool lowerAllowChecks(Function &F, FunctionAnalysisManager &AM,
                             const LowerAllowCheckPass::Options &Opts) {
  // Lazy analysis getters.
  auto GetBFI = [&AM, &F, BFI = (BlockFrequencyInfo *)nullptr]() mutable
      -> const BlockFrequencyInfo & {
```

- **L61**: Executes call or statement centered on `Info`. / 执行以 `Info` 为核心的调用或语句。
- **L62**: Sets the debug logging category used by LLVM debug output. / 设置 LLVM 调试输出使用的调试类别。
- **L63**: Continues the surrounding expression or declaration: `<< "Removed check: Kind=" << Info.Kind << " F=" << Info.F`. / 继续构造周围的表达式或声明：`<< "Removed check: Kind=" << Info.Kind << " F=" << Info.F`。
- **L64**: Executes a standalone statement or declaration: `<< " BB=" << Info.BB;`. / 执行一条独立语句或声明：`<< " BB=" << Info.BB;`。
- **L65**: Executes a standalone statement or declaration: `});`. / 执行一条独立语句或声明：`});`。
- **L66**: Continues the surrounding expression or declaration: `} else {`. / 继续构造周围的表达式或声明：`} else {`。
- **L67**: Starts a function, method, or lambda body: `ORE.emit([&]() {`. / 开始一个函数、方法或 lambda 的主体：`ORE.emit([&]() {`。
- **L68**: Executes call or statement centered on `Info`. / 执行以 `Info` 为核心的调用或语句。
- **L69**: Sets the debug logging category used by LLVM debug output. / 设置 LLVM 调试输出使用的调试类别。
- **L70**: Continues the surrounding expression or declaration: `<< "Allowed check: Kind=" << Info.Kind << " F=" << Info.F`. / 继续构造周围的表达式或声明：`<< "Allowed check: Kind=" << Info.Kind << " F=" << Info.F`。
- **L71**: Executes a standalone statement or declaration: `<< " BB=" << Info.BB;`. / 执行一条独立语句或声明：`<< " BB=" << Info.BB;`。
- **L72**: Executes a standalone statement or declaration: `});`. / 执行一条独立语句或声明：`});`。
- **L73**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L74**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L75**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L76**: Continues a multi-line argument list or initializer: `static bool lowerAllowChecks(Function &F, FunctionAnalysisManager &AM,`. / 继续一个多行参数列表或初始化器：`static bool lowerAllowChecks(Function &F, FunctionAnalysisManager &AM,`。
- **L77**: Continues the surrounding expression or declaration: `const LowerAllowCheckPass::Options &Opts) {`. / 继续构造周围的表达式或声明：`const LowerAllowCheckPass::Options &Opts) {`。
- **L78**: Comment documents the nearby logic or transformation intent: `Lazy analysis getters.`. / 注释说明了附近代码的逻辑或变换意图：`Lazy analysis getters.`。
- **L79**: Continues the surrounding expression or declaration: `auto GetBFI = [&AM, &F, BFI = (BlockFrequencyInfo *)nullptr]() mutable`. / 继续构造周围的表达式或声明：`auto GetBFI = [&AM, &F, BFI = (BlockFrequencyInfo *)nullptr]() mutable`。
- **L80**: Continues the surrounding expression or declaration: `-> const BlockFrequencyInfo & {`. / 继续构造周围的表达式或声明：`-> const BlockFrequencyInfo & {`。

### Lines 81-100

```cpp
    if (!BFI)
      BFI = &AM.getResult<BlockFrequencyAnalysis>(F);
    return *BFI;
  };
  auto GetPSI = [&AM, &F, PSI = std::optional<ProfileSummaryInfo *>()]() mutable
      -> const ProfileSummaryInfo * {
    if (!PSI.has_value()) {
      auto &MAMProxy = AM.getResult<ModuleAnalysisManagerFunctionProxy>(F);
      PSI = MAMProxy.getCachedResult<ProfileSummaryAnalysis>(*F.getParent());
    }
    return *PSI;
  };
  auto GetORE = [&AM, &F, ORE = (OptimizationRemarkEmitter *)nullptr]() mutable
      -> OptimizationRemarkEmitter & {
    if (!ORE)
      ORE = &AM.getResult<OptimizationRemarkEmitterAnalysis>(F);
    return *ORE;
  };

  // List of intrinsics and the constant value they should be lowered to.
```

- **L81**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L82**: Executes call or statement centered on `&AM.getResult<BlockFrequencyAnalysis>`. / 执行以 `&AM.getResult<BlockFrequencyAnalysis>` 为核心的调用或语句。
- **L83**: Returns from the current function with `*BFI`. / 以 `*BFI` 从当前函数返回。
- **L84**: Closes the current declaration scope, such as a class or struct. / 结束当前声明作用域，例如类或结构体。
- **L85**: Continues the surrounding expression or declaration: `auto GetPSI = [&AM, &F, PSI = std::optional<ProfileSummaryInfo *>()]() mutable`. / 继续构造周围的表达式或声明：`auto GetPSI = [&AM, &F, PSI = std::optional<ProfileSummaryInfo *>()]() mutable`。
- **L86**: Continues the surrounding expression or declaration: `-> const ProfileSummaryInfo * {`. / 继续构造周围的表达式或声明：`-> const ProfileSummaryInfo * {`。
- **L87**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L88**: Executes call or statement centered on `AM.getResult<ModuleAnalysisManagerFunctionProxy>`. / 执行以 `AM.getResult<ModuleAnalysisManagerFunctionProxy>` 为核心的调用或语句。
- **L89**: Executes call or statement centered on `MAMProxy.getCachedResult<ProfileSummaryAnalysis>`. / 执行以 `MAMProxy.getCachedResult<ProfileSummaryAnalysis>` 为核心的调用或语句。
- **L90**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L91**: Returns from the current function with `*PSI`. / 以 `*PSI` 从当前函数返回。
- **L92**: Closes the current declaration scope, such as a class or struct. / 结束当前声明作用域，例如类或结构体。
- **L93**: Continues the surrounding expression or declaration: `auto GetORE = [&AM, &F, ORE = (OptimizationRemarkEmitter *)nullptr]() mutable`. / 继续构造周围的表达式或声明：`auto GetORE = [&AM, &F, ORE = (OptimizationRemarkEmitter *)nullptr]() mutable`。
- **L94**: Continues the surrounding expression or declaration: `-> OptimizationRemarkEmitter & {`. / 继续构造周围的表达式或声明：`-> OptimizationRemarkEmitter & {`。
- **L95**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L96**: Executes call or statement centered on `&AM.getResult<OptimizationRemarkEmitterAnalysis>`. / 执行以 `&AM.getResult<OptimizationRemarkEmitterAnalysis>` 为核心的调用或语句。
- **L97**: Returns from the current function with `*ORE`. / 以 `*ORE` 从当前函数返回。
- **L98**: Closes the current declaration scope, such as a class or struct. / 结束当前声明作用域，例如类或结构体。
- **L99**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L100**: Comment documents the nearby logic or transformation intent: `List of intrinsics and the constant value they should be lowered to.`. / 注释说明了附近代码的逻辑或变换意图：`List of intrinsics and the constant value they should be lowered to.`。

### Lines 101-120

```cpp
  SmallVector<std::pair<IntrinsicInst *, bool>, 16> ReplaceWithValue;
  std::unique_ptr<RandomNumberGenerator> Rng;

  auto GetRng = [&]() -> RandomNumberGenerator & {
    if (!Rng)
      Rng = F.getParent()->createRNG(F.getName());
    return *Rng;
  };

  auto GetCutoff = [&](const IntrinsicInst *II) -> unsigned {
    if (HotPercentileCutoff.getNumOccurrences())
      return HotPercentileCutoff;
    else if (II->getIntrinsicID() == Intrinsic::allow_ubsan_check) {
      auto *Kind = cast<ConstantInt>(II->getArgOperand(0));
      if (Kind->getZExtValue() < Opts.cutoffs.size())
        return Opts.cutoffs[Kind->getZExtValue()];
    } else if (II->getIntrinsicID() == Intrinsic::allow_runtime_check) {
      return Opts.runtime_check;
    }

```

- **L101**: Executes a standalone statement or declaration: `SmallVector<std::pair<IntrinsicInst *, bool>, 16> ReplaceWithValue;`. / 执行一条独立语句或声明：`SmallVector<std::pair<IntrinsicInst *, bool>, 16> ReplaceWithValue;`。
- **L102**: Executes a standalone statement or declaration: `std::unique_ptr<RandomNumberGenerator> Rng;`. / 执行一条独立语句或声明：`std::unique_ptr<RandomNumberGenerator> Rng;`。
- **L103**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L104**: Starts a function, method, or lambda body: `auto GetRng = [&]() -> RandomNumberGenerator & {`. / 开始一个函数、方法或 lambda 的主体：`auto GetRng = [&]() -> RandomNumberGenerator & {`。
- **L105**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L106**: Executes call or statement centered on `F.getParent`. / 执行以 `F.getParent` 为核心的调用或语句。
- **L107**: Returns from the current function with `*Rng`. / 以 `*Rng` 从当前函数返回。
- **L108**: Closes the current declaration scope, such as a class or struct. / 结束当前声明作用域，例如类或结构体。
- **L109**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L110**: Starts a function, method, or lambda body: `auto GetCutoff = [&](const IntrinsicInst *II) -> unsigned {`. / 开始一个函数、方法或 lambda 的主体：`auto GetCutoff = [&](const IntrinsicInst *II) -> unsigned {`。
- **L111**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L112**: Returns from the current function with `HotPercentileCutoff`. / 以 `HotPercentileCutoff` 从当前函数返回。
- **L113**: Starts the alternative branch of the preceding conditional. / 开始前一个条件语句的备选分支。
- **L114**: Executes call or statement centered on `cast<ConstantInt>`. / 执行以 `cast<ConstantInt>` 为核心的调用或语句。
- **L115**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L116**: Returns from the current function with `Opts.cutoffs[Kind->getZExtValue()]`. / 以 `Opts.cutoffs[Kind->getZExtValue()]` 从当前函数返回。
- **L117**: Starts a function, method, or lambda body: `} else if (II->getIntrinsicID() == Intrinsic::allow_runtime_check) {`. / 开始一个函数、方法或 lambda 的主体：`} else if (II->getIntrinsicID() == Intrinsic::allow_runtime_check) {`。
- **L118**: Returns from the current function with `Opts.runtime_check`. / 以 `Opts.runtime_check` 从当前函数返回。
- **L119**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L120**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 121-140

```cpp
    return 0;
  };

  auto ShouldRemoveHot = [&](const BasicBlock &BB, unsigned int cutoff) {
    if (cutoff == 1000000)
      return true;
    const ProfileSummaryInfo *PSI = GetPSI();
    return PSI && PSI->isHotCountNthPercentile(
                      cutoff, GetBFI().getBlockProfileCount(&BB).value_or(0));
  };

  auto ShouldRemoveRandom = [&]() {
    return RandomRate.getNumOccurrences() &&
           !std::bernoulli_distribution(RandomRate)(GetRng());
  };

  auto ShouldRemove = [&](const IntrinsicInst *II) {
    unsigned int cutoff = GetCutoff(II);
    return ShouldRemoveRandom() || ShouldRemoveHot(*(II->getParent()), cutoff);
  };
```

- **L121**: Returns from the current function with `0`. / 以 `0` 从当前函数返回。
- **L122**: Closes the current declaration scope, such as a class or struct. / 结束当前声明作用域，例如类或结构体。
- **L123**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L124**: Starts a function, method, or lambda body: `auto ShouldRemoveHot = [&](const BasicBlock &BB, unsigned int cutoff) {`. / 开始一个函数、方法或 lambda 的主体：`auto ShouldRemoveHot = [&](const BasicBlock &BB, unsigned int cutoff) {`。
- **L125**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L126**: Returns from the current function with `true`. / 以 `true` 从当前函数返回。
- **L127**: Executes call or statement centered on `GetPSI`. / 执行以 `GetPSI` 为核心的调用或语句。
- **L128**: Returns from the current function with `PSI && PSI->isHotCountNthPercentile(`. / 以 `PSI && PSI->isHotCountNthPercentile(` 从当前函数返回。
- **L129**: Executes call or statement centered on `GetBFI`. / 执行以 `GetBFI` 为核心的调用或语句。
- **L130**: Closes the current declaration scope, such as a class or struct. / 结束当前声明作用域，例如类或结构体。
- **L131**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L132**: Starts a function, method, or lambda body: `auto ShouldRemoveRandom = [&]() {`. / 开始一个函数、方法或 lambda 的主体：`auto ShouldRemoveRandom = [&]() {`。
- **L133**: Returns from the current function with `RandomRate.getNumOccurrences() &&`. / 以 `RandomRate.getNumOccurrences() &&` 从当前函数返回。
- **L134**: Executes call or statement centered on `!std::bernoulli_distribution`. / 执行以 `!std::bernoulli_distribution` 为核心的调用或语句。
- **L135**: Closes the current declaration scope, such as a class or struct. / 结束当前声明作用域，例如类或结构体。
- **L136**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L137**: Starts a function, method, or lambda body: `auto ShouldRemove = [&](const IntrinsicInst *II) {`. / 开始一个函数、方法或 lambda 的主体：`auto ShouldRemove = [&](const IntrinsicInst *II) {`。
- **L138**: Initializes variable `cutoff` from the right-hand expression. / 使用右侧表达式初始化变量 `cutoff`。
- **L139**: Returns from the current function with `ShouldRemoveRandom() || ShouldRemoveHot(*(II->getParent()), cutoff)`. / 以 `ShouldRemoveRandom() || ShouldRemoveHot(*(II->getParent()), cutoff)` 从当前函数返回。
- **L140**: Closes the current declaration scope, such as a class or struct. / 结束当前声明作用域，例如类或结构体。

### Lines 141-160

```cpp

  for (Instruction &I : instructions(F)) {
    IntrinsicInst *II = dyn_cast<IntrinsicInst>(&I);
    if (!II)
      continue;
    auto ID = II->getIntrinsicID();
    switch (ID) {
    case Intrinsic::allow_ubsan_check:
    case Intrinsic::allow_runtime_check: {
      bool ToRemove = ShouldRemove(II);

      ReplaceWithValue.push_back({
          II,
          !ToRemove,
      });
      emitRemark(II, GetORE(), ToRemove);
      break;
    }
    case Intrinsic::allow_sanitize_address:
      ReplaceWithValue.push_back(
```

- **L141**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L142**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L143**: Executes call or statement centered on `dyn_cast<IntrinsicInst>`. / 执行以 `dyn_cast<IntrinsicInst>` 为核心的调用或语句。
- **L144**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L145**: Skips to the next loop iteration. / 跳到下一次循环迭代。
- **L146**: Initializes variable `ID` from the right-hand expression. / 使用右侧表达式初始化变量 `ID`。
- **L147**: Begins a `switch` control-flow statement and evaluates its condition. / 开始 `switch` 控制流语句并计算其条件。
- **L148**: Introduces a switch dispatch label: `case Intrinsic::allow_ubsan_check:`. / 引入一个 switch 分发标签：`case Intrinsic::allow_ubsan_check:`。
- **L149**: Introduces a switch dispatch label: `case Intrinsic::allow_runtime_check: {`. / 引入一个 switch 分发标签：`case Intrinsic::allow_runtime_check: {`。
- **L150**: Initializes variable `ToRemove` from the right-hand expression. / 使用右侧表达式初始化变量 `ToRemove`。
- **L151**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L152**: Starts a function, method, or lambda body: `ReplaceWithValue.push_back({`. / 开始一个函数、方法或 lambda 的主体：`ReplaceWithValue.push_back({`。
- **L153**: Continues a multi-line argument list or initializer: `II,`. / 继续一个多行参数列表或初始化器：`II,`。
- **L154**: Continues a multi-line argument list or initializer: `!ToRemove,`. / 继续一个多行参数列表或初始化器：`!ToRemove,`。
- **L155**: Executes a standalone statement or declaration: `});`. / 执行一条独立语句或声明：`});`。
- **L156**: Executes call or statement centered on `emitRemark`. / 执行以 `emitRemark` 为核心的调用或语句。
- **L157**: Exits the nearest loop or switch statement. / 退出最近的循环或 switch 语句。
- **L158**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L159**: Introduces a switch dispatch label: `case Intrinsic::allow_sanitize_address:`. / 引入一个 switch 分发标签：`case Intrinsic::allow_sanitize_address:`。
- **L160**: Continues the surrounding expression or declaration: `ReplaceWithValue.push_back(`. / 继续构造周围的表达式或声明：`ReplaceWithValue.push_back(`。

### Lines 161-180

```cpp
          {II, F.hasFnAttribute(Attribute::SanitizeAddress)});
      break;
    case Intrinsic::allow_sanitize_thread:
      ReplaceWithValue.push_back(
          {II, F.hasFnAttribute(Attribute::SanitizeThread)});
      break;
    case Intrinsic::allow_sanitize_memory:
      ReplaceWithValue.push_back(
          {II, F.hasFnAttribute(Attribute::SanitizeMemory)});
      break;
    case Intrinsic::allow_sanitize_hwaddress:
      ReplaceWithValue.push_back(
          {II, F.hasFnAttribute(Attribute::SanitizeHWAddress)});
      break;
    default:
      break;
    }
  }

  for (auto [I, V] : ReplaceWithValue) {
```

- **L161**: Executes call or statement centered on `F.hasFnAttribute`. / 执行以 `F.hasFnAttribute` 为核心的调用或语句。
- **L162**: Exits the nearest loop or switch statement. / 退出最近的循环或 switch 语句。
- **L163**: Introduces a switch dispatch label: `case Intrinsic::allow_sanitize_thread:`. / 引入一个 switch 分发标签：`case Intrinsic::allow_sanitize_thread:`。
- **L164**: Continues the surrounding expression or declaration: `ReplaceWithValue.push_back(`. / 继续构造周围的表达式或声明：`ReplaceWithValue.push_back(`。
- **L165**: Executes call or statement centered on `F.hasFnAttribute`. / 执行以 `F.hasFnAttribute` 为核心的调用或语句。
- **L166**: Exits the nearest loop or switch statement. / 退出最近的循环或 switch 语句。
- **L167**: Introduces a switch dispatch label: `case Intrinsic::allow_sanitize_memory:`. / 引入一个 switch 分发标签：`case Intrinsic::allow_sanitize_memory:`。
- **L168**: Continues the surrounding expression or declaration: `ReplaceWithValue.push_back(`. / 继续构造周围的表达式或声明：`ReplaceWithValue.push_back(`。
- **L169**: Executes call or statement centered on `F.hasFnAttribute`. / 执行以 `F.hasFnAttribute` 为核心的调用或语句。
- **L170**: Exits the nearest loop or switch statement. / 退出最近的循环或 switch 语句。
- **L171**: Introduces a switch dispatch label: `case Intrinsic::allow_sanitize_hwaddress:`. / 引入一个 switch 分发标签：`case Intrinsic::allow_sanitize_hwaddress:`。
- **L172**: Continues the surrounding expression or declaration: `ReplaceWithValue.push_back(`. / 继续构造周围的表达式或声明：`ReplaceWithValue.push_back(`。
- **L173**: Executes call or statement centered on `F.hasFnAttribute`. / 执行以 `F.hasFnAttribute` 为核心的调用或语句。
- **L174**: Exits the nearest loop or switch statement. / 退出最近的循环或 switch 语句。
- **L175**: Introduces a switch dispatch label: `default:`. / 引入一个 switch 分发标签：`default:`。
- **L176**: Exits the nearest loop or switch statement. / 退出最近的循环或 switch 语句。
- **L177**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L178**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L179**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L180**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。

### Lines 181-200

```cpp
    ++NumChecksTotal;
    if (!V) // If the final value is false, the check is considered removed.
      ++NumChecksRemoved;
    I->replaceAllUsesWith(ConstantInt::getBool(I->getType(), V));
    I->eraseFromParent();
  }

  return !ReplaceWithValue.empty();
}

PreservedAnalyses LowerAllowCheckPass::run(Function &F,
                                           FunctionAnalysisManager &AM) {
  if (F.isDeclaration())
    return PreservedAnalyses::all();

  return lowerAllowChecks(F, AM, Opts)
             // We do not change the CFG, we only replace the intrinsics with
             // true or false.
             ? PreservedAnalyses::none().preserveSet<CFGAnalyses>()
             : PreservedAnalyses::all();
```

- **L181**: Executes a standalone statement or declaration: `++NumChecksTotal;`. / 执行一条独立语句或声明：`++NumChecksTotal;`。
- **L182**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L183**: Executes a standalone statement or declaration: `++NumChecksRemoved;`. / 执行一条独立语句或声明：`++NumChecksRemoved;`。
- **L184**: Executes call or statement centered on `I->replaceAllUsesWith`. / 执行以 `I->replaceAllUsesWith` 为核心的调用或语句。
- **L185**: Executes call or statement centered on `I->eraseFromParent`. / 执行以 `I->eraseFromParent` 为核心的调用或语句。
- **L186**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L187**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L188**: Returns from the current function with `!ReplaceWithValue.empty()`. / 以 `!ReplaceWithValue.empty()` 从当前函数返回。
- **L189**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L190**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L191**: Continues a multi-line argument list or initializer: `PreservedAnalyses LowerAllowCheckPass::run(Function &F,`. / 继续一个多行参数列表或初始化器：`PreservedAnalyses LowerAllowCheckPass::run(Function &F,`。
- **L192**: Continues the surrounding expression or declaration: `FunctionAnalysisManager &AM) {`. / 继续构造周围的表达式或声明：`FunctionAnalysisManager &AM) {`。
- **L193**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L194**: Returns from the current function with `PreservedAnalyses::all()`. / 以 `PreservedAnalyses::all()` 从当前函数返回。
- **L195**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L196**: Returns from the current function with `lowerAllowChecks(F, AM, Opts)`. / 以 `lowerAllowChecks(F, AM, Opts)` 从当前函数返回。
- **L197**: Comment documents the nearby logic or transformation intent: `We do not change the CFG, we only replace the intrinsics with`. / 注释说明了附近代码的逻辑或变换意图：`We do not change the CFG, we only replace the intrinsics with`。
- **L198**: Comment documents the nearby logic or transformation intent: `true or false.`. / 注释说明了附近代码的逻辑或变换意图：`true or false.`。
- **L199**: Continues the surrounding expression or declaration: `? PreservedAnalyses::none().preserveSet<CFGAnalyses>()`. / 继续构造周围的表达式或声明：`? PreservedAnalyses::none().preserveSet<CFGAnalyses>()`。
- **L200**: Executes call or statement centered on `PreservedAnalyses::all`. / 执行以 `PreservedAnalyses::all` 为核心的调用或语句。

### Lines 201-220

```cpp
}

bool LowerAllowCheckPass::IsRequested() {
  return RandomRate.getNumOccurrences() ||
         HotPercentileCutoff.getNumOccurrences();
}

void LowerAllowCheckPass::printPipeline(
    raw_ostream &OS, function_ref<StringRef(StringRef)> MapClassName2PassName) {
  static_cast<PassInfoMixin<LowerAllowCheckPass> *>(this)->printPipeline(
      OS, MapClassName2PassName);
  OS << "<";

  // Format is <cutoffs[0,1,2]=70000;cutoffs[5,6,8]=90000>
  // but it's equally valid to specify
  //   cutoffs[0]=70000;cutoffs[1]=70000;cutoffs[2]=70000;cutoffs[5]=90000;...
  // and that's what we do here. It is verbose but valid and easy to verify
  // correctness.
  // TODO: print shorter output by combining adjacent runs, etc.
  int i = 0;
```

- **L201**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L202**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L203**: Starts a function, method, or lambda body: `bool LowerAllowCheckPass::IsRequested() {`. / 开始一个函数、方法或 lambda 的主体：`bool LowerAllowCheckPass::IsRequested() {`。
- **L204**: Returns from the current function with `RandomRate.getNumOccurrences() ||`. / 以 `RandomRate.getNumOccurrences() ||` 从当前函数返回。
- **L205**: Executes call or statement centered on `HotPercentileCutoff.getNumOccurrences`. / 执行以 `HotPercentileCutoff.getNumOccurrences` 为核心的调用或语句。
- **L206**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L207**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L208**: Continues the surrounding expression or declaration: `void LowerAllowCheckPass::printPipeline(`. / 继续构造周围的表达式或声明：`void LowerAllowCheckPass::printPipeline(`。
- **L209**: Starts a function, method, or lambda body: `raw_ostream &OS, function_ref<StringRef(StringRef)> MapClassName2PassName) {`. / 开始一个函数、方法或 lambda 的主体：`raw_ostream &OS, function_ref<StringRef(StringRef)> MapClassName2PassName) {`。
- **L210**: Continues the surrounding expression or declaration: `static_cast<PassInfoMixin<LowerAllowCheckPass> *>(this)->printPipeline(`. / 继续构造周围的表达式或声明：`static_cast<PassInfoMixin<LowerAllowCheckPass> *>(this)->printPipeline(`。
- **L211**: Executes a standalone statement or declaration: `OS, MapClassName2PassName);`. / 执行一条独立语句或声明：`OS, MapClassName2PassName);`。
- **L212**: Executes a standalone statement or declaration: `OS << "<";`. / 执行一条独立语句或声明：`OS << "<";`。
- **L213**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L214**: Comment documents the nearby logic or transformation intent: `Format is <cutoffs[0,1,2]=70000;cutoffs[5,6,8]=90000>`. / 注释说明了附近代码的逻辑或变换意图：`Format is <cutoffs[0,1,2]=70000;cutoffs[5,6,8]=90000>`。
- **L215**: Comment documents the nearby logic or transformation intent: `but it's equally valid to specify`. / 注释说明了附近代码的逻辑或变换意图：`but it's equally valid to specify`。
- **L216**: Comment documents the nearby logic or transformation intent: `cutoffs[0]=70000;cutoffs[1]=70000;cutoffs[2]=70000;cutoffs[5]=90000;...`. / 注释说明了附近代码的逻辑或变换意图：`cutoffs[0]=70000;cutoffs[1]=70000;cutoffs[2]=70000;cutoffs[5]=90000;...`。
- **L217**: Comment documents the nearby logic or transformation intent: `and that's what we do here. It is verbose but valid and easy to verify`. / 注释说明了附近代码的逻辑或变换意图：`and that's what we do here. It is verbose but valid and easy to verify`。
- **L218**: Comment documents the nearby logic or transformation intent: `correctness.`. / 注释说明了附近代码的逻辑或变换意图：`correctness.`。
- **L219**: Comment records a pending task or caution: `TODO: print shorter output by combining adjacent runs, etc.`. / 注释记录了待办事项或注意点：`TODO: print shorter output by combining adjacent runs, etc.`。
- **L220**: Initializes variable `i` from the right-hand expression. / 使用右侧表达式初始化变量 `i`。

### Lines 221-231

```cpp
  ListSeparator LS(";");
  for (unsigned int cutoff : Opts.cutoffs) {
    if (cutoff > 0)
      OS << LS << "cutoffs[" << i << "]=" << cutoff;
    i++;
  }
  if (Opts.runtime_check)
    OS << LS << "runtime_check=" << Opts.runtime_check;

  OS << '>';
}
```

- **L221**: Executes call or statement centered on `LS`. / 执行以 `LS` 为核心的调用或语句。
- **L222**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L223**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L224**: Executes a standalone statement or declaration: `OS << LS << "cutoffs[" << i << "]=" << cutoff;`. / 执行一条独立语句或声明：`OS << LS << "cutoffs[" << i << "]=" << cutoff;`。
- **L225**: Executes a standalone statement or declaration: `i++;`. / 执行一条独立语句或声明：`i++;`。
- **L226**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L227**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L228**: Executes a standalone statement or declaration: `OS << LS << "runtime_check=" << Opts.runtime_check;`. / 执行一条独立语句或声明：`OS << LS << "runtime_check=" << Opts.runtime_check;`。
- **L229**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L230**: Executes a standalone statement or declaration: `OS << '>';`. / 执行一条独立语句或声明：`OS << '>';`。
- **L231**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **Instrumentation transform pipeline / Instrumentation 变换流水线**
- **Function-level traversal and updates / 函数级遍历与更新**
- **Module-wide coordination / 模块范围的协调**
- **Pass statistics and instrumentation / Pass 统计与观测**
- **Analysis preservation reporting / 分析保持情况报告**
- **Optimization remarks and diagnostics / 优化备注与诊断**
- **New PM pass wrapper structure / 新 PM 的 pass 包装结构**

## Dependencies / 依赖关系

- `llvm/Transforms/Instrumentation/LowerAllowCheckPass.h`: Provides transform-specific declarations. / 提供变换相关声明。
- `llvm/ADT/SmallVector.h`: Provides LLVM ADT data structures/utilities. / 提供LLVM ADT 数据结构/工具。
- `llvm/ADT/Statistic.h`: Provides LLVM ADT data structures/utilities. / 提供LLVM ADT 数据结构/工具。
- `llvm/ADT/StringExtras.h`: Provides LLVM ADT data structures/utilities. / 提供LLVM ADT 数据结构/工具。
- `llvm/ADT/StringRef.h`: Provides LLVM ADT data structures/utilities. / 提供LLVM ADT 数据结构/工具。
- `llvm/Analysis/OptimizationRemarkEmitter.h`: Provides analysis interfaces and cached results. / 提供分析接口与缓存结果。
- `llvm/Analysis/ProfileSummaryInfo.h`: Provides analysis interfaces and cached results. / 提供分析接口与缓存结果。
- `llvm/IR/Constants.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/DiagnosticInfo.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/InstIterator.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/Instructions.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/IntrinsicInst.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/Intrinsics.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/Metadata.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/Module.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/Support/Debug.h`: Provides support-library helpers. / 提供Support 库辅助功能。
- `llvm/Support/RandomNumberGenerator.h`: Provides support-library helpers. / 提供Support 库辅助功能。
- `memory`: Provides supporting declarations. / 提供所需的辅助声明。
- `optional`: Provides supporting declarations. / 提供所需的辅助声明。
- `random`: Provides supporting declarations. / 提供所需的辅助声明。
