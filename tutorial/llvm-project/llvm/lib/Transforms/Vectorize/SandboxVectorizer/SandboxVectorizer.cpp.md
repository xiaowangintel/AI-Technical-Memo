# SandboxVectorizer.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/SandboxVectorizer/SandboxVectorizer.cpp` | `llvm/lib/Transforms/Vectorize/SandboxVectorizer/SandboxVectorizer.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements vectorizer based on Sandbox IR within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 SandboxVectorizer 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-17

```cpp
//===- SandboxVectorizer.cpp - Vectorizer based on Sandbox IR -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Vectorize/SandboxVectorizer/SandboxVectorizer.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/IR/Module.h"
#include "llvm/SandboxIR/Constant.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Regex.h"
#include "llvm/Transforms/Vectorize/SandboxVectorizer/Debug.h"
#include "llvm/Transforms/Vectorize/SandboxVectorizer/SandboxVectorizerPassBuilder.h"

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 18-31

```cpp
using namespace llvm;

static cl::opt<bool>
    PrintPassPipeline("sbvec-print-pass-pipeline", cl::init(false), cl::Hidden,
                      cl::desc("Prints the pass pipeline and returns."));

/// A magic string for the default pass pipeline.
static const char *DefaultPipelineMagicStr = "*";

static cl::opt<std::string> UserDefinedPassPipeline(
    "sbvec-passes", cl::init(DefaultPipelineMagicStr), cl::Hidden,
    cl::desc("Comma-separated list of vectorizer passes. If not set "
             "we run the predefined pipeline."));

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 32-47

```cpp
// This option is useful for bisection debugging.
// For example you may use it to figure out which filename is the one causing a
// miscompile. You can specify a regex for the filename like: "/[a-m][^/]*"
// which will enable any file name starting with 'a' to 'm' and disable the
// rest. If the miscompile goes away, then we try "/[n-z][^/]*" for the other
// half of the range, from 'n' to 'z'. If we can reproduce the miscompile then
// we can keep looking in [n-r] and [s-z] and so on, in a binary-search fashion.
//
// Please note that we are using [^/]* and not .* to make sure that we are
// matching the actual filename and not some other directory in the path.
cl::opt<std::string> AllowFiles(
    "sbvec-allow-files", cl::init(".*"), cl::Hidden,
    cl::desc("Run the vectorizer only on file paths that match any in the "
             "list of comma-separated regex's."));
static constexpr char AllowFilesDelim = ',';

```
- EN: This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 48-66

```cpp
SandboxVectorizerPass::SandboxVectorizerPass() : FPM("fpm") {
  if (UserDefinedPassPipeline == DefaultPipelineMagicStr) {
    // TODO: Add passes to the default pipeline. It currently contains:
    //       - Seed collection, which creates seed regions and runs the pipeline
    //         - Bottom-up Vectorizer pass that starts from a seed
    //         - Load-Store Vectorizer pass for load-store chains
    //         - Accept or revert IR state pass
    FPM.setPassPipeline(
        "seed-collection<tr-save,bottom-up-vec,load-store-vec,tr-accept-or-"
        "revert>",
        sandboxir::SandboxVectorizerPassBuilder::createFunctionPass);
  } else {
    // Create the user-defined pipeline.
    FPM.setPassPipeline(
        UserDefinedPassPipeline,
        sandboxir::SandboxVectorizerPassBuilder::createFunctionPass);
  }
}

```
- EN: Core entities appearing here include SandboxVectorizerPass, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 SandboxVectorizerPass，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 67-81

```cpp
SandboxVectorizerPass::SandboxVectorizerPass(SandboxVectorizerPass &&) =
    default;

SandboxVectorizerPass::~SandboxVectorizerPass() = default;

PreservedAnalyses SandboxVectorizerPass::run(Function &F,
                                             FunctionAnalysisManager &AM) {
  TTI = &AM.getResult<TargetIRAnalysis>(F);
  AA = &AM.getResult<AAManager>(F);
  SE = &AM.getResult<ScalarEvolutionAnalysis>(F);

  bool Changed = runImpl(F);
  if (!Changed)
    return PreservedAnalyses::all();

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AAManager, ScalarEvolution.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AAManager, ScalarEvolution 等分析结果。

### Lines 82-96

```cpp
  PreservedAnalyses PA;
  PA.preserveSet<CFGAnalyses>();
  return PA;
}

bool SandboxVectorizerPass::allowFile(const std::string &SrcFilePath) {
  // Iterate over all files in AllowFiles separated by `AllowFilesDelim`.
  size_t DelimPos = 0;
  do {
    size_t LastPos = DelimPos != 0 ? DelimPos + 1 : DelimPos;
    DelimPos = AllowFiles.find(AllowFilesDelim, LastPos);
    auto FileNameToMatch = AllowFiles.substr(LastPos, DelimPos - LastPos);
    if (FileNameToMatch.empty())
      return false;
    // Note: This only runs when debugging so its OK not to reuse the regex.
```
- EN: Core entities appearing here include allowFile, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 allowFile，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 97-113

```cpp
    Regex FileNameRegex(".*" + FileNameToMatch + "$");
    assert(FileNameRegex.isValid() && "Bad regex!");
    if (FileNameRegex.match(SrcFilePath))
      return true;
  } while (DelimPos != std::string::npos);
  return false;
}

bool SandboxVectorizerPass::runImpl(Function &LLVMF) {
  if (Ctx == nullptr)
    Ctx = std::make_unique<sandboxir::Context>(LLVMF.getContext());

  if (PrintPassPipeline) {
    FPM.printPipeline(outs());
    return false;
  }

```
- EN: Core entities appearing here include runImpl, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 runImpl，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 114-128

```cpp
  // This is used for debugging.
  if (LLVM_UNLIKELY(AllowFiles != ".*")) {
    const auto &SrcFilePath = LLVMF.getParent()->getSourceFileName();
    if (!allowFile(SrcFilePath))
      return false;
  }

  // If the target claims to have no vector registers early return.
  if (!TTI->getNumberOfRegisters(TTI->getRegisterClassForType(true))) {
    LLVM_DEBUG(dbgs() << DEBUG_PREFIX
                      << "Target has no vector registers, return.\n");
    return false;
  }
  LLVM_DEBUG(dbgs() << DEBUG_PREFIX << "Analyzing " << LLVMF.getName()
                    << ".\n");
```
- EN: This region continues the SandboxVectorizer implementation with local helper logic centered on This, LLVM_UNLIKELY, AllowFiles, SrcFilePath.
- CN: 这一段延续了 SandboxVectorizer 的主体实现，围绕 This, LLVM_UNLIKELY, AllowFiles, SrcFilePath 等局部辅助逻辑展开。

### Lines 129-143

```cpp
  // Early return if the attribute NoImplicitFloat is used.
  if (LLVMF.hasFnAttribute(Attribute::NoImplicitFloat)) {
    LLVM_DEBUG(dbgs() << DEBUG_PREFIX
                      << "NoImplicitFloat attribute, return.\n");
    return false;
  }

  // Create SandboxIR for LLVMF and run BottomUpVec on it.
  sandboxir::Function &F = *Ctx->createFunction(&LLVMF);
  sandboxir::Analyses A(*AA, *SE, *TTI);
  bool Change = FPM.runOnFunction(F, A);
  // Given that sandboxir::Context `Ctx` is defined at a pass-level scope, the
  // maps from LLVM IR to Sandbox IR may go stale as later passes remove LLVM IR
  // objects. To avoid issues caused by this clear the context's state.
  // NOTE: The alternative would be to define Ctx and FPM within runOnFunction()
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 144-146

```cpp
  Ctx->clear();
  return Change;
}
```
- EN: This region continues the SandboxVectorizer implementation with local helper logic centered on Ctx, Change.
- CN: 这一段延续了 SandboxVectorizer 的主体实现，围绕 Ctx, Change 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `SandboxVectorizerPass, allowFile, runImpl` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`SandboxVectorizerPass, allowFile, runImpl` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AAManager, ScalarEvolution, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AAManager, ScalarEvolution, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `PrintPassPipeline, UserDefinedPassPipeline` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `PrintPassPipeline, UserDefinedPassPipeline` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/TargetTransformInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/TargetTransformInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Module.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/SandboxVectorizer.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/Debug.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/SandboxVectorizerPassBuilder.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Module.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/SandboxVectorizer.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/Debug.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/SandboxVectorizerPassBuilder.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/CommandLine.h`, `llvm/Support/Regex.h`, `llvm/SandboxIR/Constant.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/CommandLine.h`, `llvm/Support/Regex.h`, `llvm/SandboxIR/Constant.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `AAManager`, `ScalarEvolution`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AAManager`, `ScalarEvolution`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
