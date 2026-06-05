# MetaRenamer.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/MetaRenamer.cpp` | `llvm/lib/Transforms/Utils/MetaRenamer.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements rename everything with metasyntatic names within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 MetaRenamer 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

```cpp
//===- MetaRenamer.cpp - Rename everything with metasyntatic names --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass renames everything with metasyntatic names. The intent is to use
// this pass after llvm-reduce reduction to conceal the nature of the original
// program.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/MetaRenamer.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/Twine.h"
#include "llvm/Analysis/TargetLibraryInfo.h"
#include "llvm/IR/Argument.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/DerivedTypes.h"
```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetLibraryInfo.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 这一段中的合法性、收益性或规范形态检查依赖 TargetLibraryInfo 等分析结果。

### Lines 25-49

```cpp
#include "llvm/IR/Function.h"
#include "llvm/IR/GlobalAlias.h"
#include "llvm/IR/GlobalVariable.h"
#include "llvm/IR/Instruction.h"
#include "llvm/IR/InstIterator.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/PassManager.h"
#include "llvm/IR/Type.h"
#include "llvm/IR/TypeFinder.h"
#include "llvm/Support/CommandLine.h"

using namespace llvm;

static cl::opt<std::string> RenameExcludeFunctionPrefixes(
    "rename-exclude-function-prefixes",
    cl::desc("Prefixes for functions that don't need to be renamed, separated "
             "by a comma"),
    cl::Hidden);

static cl::opt<std::string> RenameExcludeAliasPrefixes(
    "rename-exclude-alias-prefixes",
    cl::desc("Prefixes for aliases that don't need to be renamed, separated "
             "by a comma"),
    cl::Hidden);

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 50-73

```cpp
static cl::opt<std::string> RenameExcludeGlobalPrefixes(
    "rename-exclude-global-prefixes",
    cl::desc(
        "Prefixes for global values that don't need to be renamed, separated "
        "by a comma"),
    cl::Hidden);

static cl::opt<std::string> RenameExcludeStructPrefixes(
    "rename-exclude-struct-prefixes",
    cl::desc("Prefixes for structs that don't need to be renamed, separated "
             "by a comma"),
    cl::Hidden);

static cl::opt<bool>
    RenameOnlyInst("rename-only-inst", cl::init(false),
                   cl::desc("only rename the instructions in the function"),
                   cl::Hidden);

static const char *const metaNames[] = {
  // See http://en.wikipedia.org/wiki/Metasyntactic_variable
  "foo", "bar", "baz", "quux", "barney", "snork", "zot", "blam", "hoge",
  "wibble", "wobble", "widget", "wombat", "ham", "eggs", "pluto", "spam"
};

```
- EN: This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 74-98

```cpp
namespace {
// This PRNG is from the ISO C spec. It is intentionally simple and
// unsuitable for cryptographic use. We're just looking for enough
// variety to surprise and delight users.
struct PRNG {
  unsigned long next;

  void srand(unsigned int seed) { next = seed; }

  int rand() {
    next = next * 1103515245 + 12345;
    return (unsigned int)(next / 65536) % 32768;
  }
};

struct Renamer {
  Renamer(unsigned int seed) { prng.srand(seed); }

  const char *newName() {
    return metaNames[prng.rand() % std::size(metaNames)];
  }

  PRNG prng;
};

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include PRNG, rand, Renamer, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 PRNG, rand, Renamer，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 99-124

```cpp
static void
parseExcludedPrefixes(StringRef PrefixesStr,
                      SmallVectorImpl<StringRef> &ExcludedPrefixes) {
  for (;;) {
    auto PrefixesSplit = PrefixesStr.split(',');
    if (PrefixesSplit.first.empty())
      break;
    ExcludedPrefixes.push_back(PrefixesSplit.first);
    PrefixesStr = PrefixesSplit.second;
  }
}

void MetaRenameOnlyInstructions(Function &F) {
  for (auto &I : instructions(F))
    if (!I.getType()->isVoidTy() && I.getName().empty())
      I.setName(I.getOpcodeName());
}

void MetaRename(Function &F) {
  for (Argument &Arg : F.args())
    if (!Arg.getType()->isVoidTy())
      Arg.setName("arg");

  for (auto &BB : F) {
    BB.setName("bb");

```
- EN: Core entities appearing here include MetaRenameOnlyInstructions, MetaRename, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 MetaRenameOnlyInstructions, MetaRename，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 125-150

```cpp
    for (auto &I : BB)
      if (!I.getType()->isVoidTy())
        I.setName(I.getOpcodeName());
  }
}

void MetaRename(Module &M,
                function_ref<TargetLibraryInfo &(Function &)> GetTLI) {
  // Seed our PRNG with simple additive sum of ModuleID. We're looking to
  // simply avoid always having the same function names, and we need to
  // remain deterministic.
  unsigned int randSeed = 0;
  for (auto C : M.getModuleIdentifier())
    randSeed += C;

  Renamer renamer(randSeed);

  SmallVector<StringRef, 8> ExcludedAliasesPrefixes;
  SmallVector<StringRef, 8> ExcludedGlobalsPrefixes;
  SmallVector<StringRef, 8> ExcludedStructsPrefixes;
  SmallVector<StringRef, 8> ExcludedFuncPrefixes;
  parseExcludedPrefixes(RenameExcludeAliasPrefixes, ExcludedAliasesPrefixes);
  parseExcludedPrefixes(RenameExcludeGlobalPrefixes, ExcludedGlobalsPrefixes);
  parseExcludedPrefixes(RenameExcludeStructPrefixes, ExcludedStructsPrefixes);
  parseExcludedPrefixes(RenameExcludeFunctionPrefixes, ExcludedFuncPrefixes);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetLibraryInfo.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 TargetLibraryInfo 等分析结果。

### Lines 151-176

```cpp
  auto IsNameExcluded = [](StringRef &Name,
                           SmallVectorImpl<StringRef> &ExcludedPrefixes) {
    return any_of(ExcludedPrefixes,
                  [&Name](auto &Prefix) { return Name.starts_with(Prefix); });
  };

  // Leave library functions alone because their presence or absence could
  // affect the behavior of other passes.
  auto ExcludeLibFuncs = [&](Function &F) {
    LibFunc Tmp;
    StringRef Name = F.getName();
    return F.isIntrinsic() || (!Name.empty() && Name[0] == 1) ||
           GetTLI(F).getLibFunc(F, Tmp) ||
           IsNameExcluded(Name, ExcludedFuncPrefixes);
  };

  if (RenameOnlyInst) {
    // Rename all functions
    for (auto &F : M) {
      if (ExcludeLibFuncs(F))
        continue;
      MetaRenameOnlyInstructions(F);
    }
    return;
  }

```
- EN: This region continues the MetaRenamer implementation with local helper logic centered on IsNameExcluded, StringRef, Name, SmallVectorImpl.
- CN: 这一段延续了 MetaRenamer 的主体实现，围绕 IsNameExcluded, StringRef, Name, SmallVectorImpl 等局部辅助逻辑展开。

### Lines 177-200

```cpp
  // Rename all aliases
  for (GlobalAlias &GA : M.aliases()) {
    StringRef Name = GA.getName();
    if (Name.starts_with("llvm.") || (!Name.empty() && Name[0] == 1) ||
        IsNameExcluded(Name, ExcludedAliasesPrefixes))
      continue;

    GA.setName("alias");
  }

  // Rename all global variables
  for (GlobalVariable &GV : M.globals()) {
    StringRef Name = GV.getName();
    if (Name.starts_with("llvm.") || (!Name.empty() && Name[0] == 1) ||
        IsNameExcluded(Name, ExcludedGlobalsPrefixes))
      continue;

    GV.setName("global");
  }

  // Rename all struct types
  TypeFinder StructTypes;
  StructTypes.run(M, true);
  for (StructType *STy : StructTypes) {
```
- EN: This region continues the MetaRenamer implementation with local helper logic centered on Rename, GlobalAlias, StringRef, Name.
- CN: 这一段延续了 MetaRenamer 的主体实现，围绕 Rename, GlobalAlias, StringRef, Name 等局部辅助逻辑展开。

### Lines 201-226

```cpp
    StringRef Name = STy->getName();
    if (STy->isLiteral() || Name.empty() ||
        IsNameExcluded(Name, ExcludedStructsPrefixes))
      continue;

    SmallString<128> NameStorage;
    STy->setName(
        (Twine("struct.") + renamer.newName()).toStringRef(NameStorage));
  }

  // Rename all functions
  for (auto &F : M) {
    if (ExcludeLibFuncs(F))
      continue;

    // Leave @main alone. The output of -metarenamer might be passed to
    // lli for execution and the latter needs a main entry point.
    if (F.getName() != "main")
      F.setName(renamer.newName());

    MetaRename(F);
  }
}

} // end anonymous namespace

```
- EN: This region continues the MetaRenamer implementation with local helper logic centered on StringRef, Name, STy, IsNameExcluded.
- CN: 这一段延续了 MetaRenamer 的主体实现，围绕 StringRef, Name, STy, IsNameExcluded 等局部辅助逻辑展开。

### Lines 227-236

```cpp
PreservedAnalyses MetaRenamerPass::run(Module &M, ModuleAnalysisManager &AM) {
  FunctionAnalysisManager &FAM =
      AM.getResult<FunctionAnalysisManagerModuleProxy>(M).getManager();
  auto GetTLI = [&FAM](Function &F) -> TargetLibraryInfo & {
    return FAM.getResult<TargetLibraryAnalysis>(F);
  };
  MetaRename(M, GetTLI);

  return PreservedAnalyses::all();
}
```
- EN: Core entities appearing here include run, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetLibraryInfo.
- CN: 此处出现的核心实体包括 run，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetLibraryInfo 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `PRNG, Renamer, rand, MetaRenameOnlyInstructions, MetaRename, run` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`PRNG, Renamer, rand, MetaRenameOnlyInstructions, MetaRename, run` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `TargetLibraryInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `TargetLibraryInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `RenameExcludeFunctionPrefixes, RenameExcludeAliasPrefixes, RenameExcludeGlobalPrefixes, RenameExcludeStructPrefixes, RenameOnlyInst` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `RenameExcludeFunctionPrefixes, RenameExcludeAliasPrefixes, RenameExcludeGlobalPrefixes, RenameExcludeStructPrefixes, RenameOnlyInst` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/TargetLibraryInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/TargetLibraryInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Argument.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Function.h`, `llvm/IR/GlobalAlias.h`, `llvm/IR/GlobalVariable.h`, `llvm/IR/Instruction.h`, `llvm/IR/InstIterator.h`, `llvm/IR/Module.h`, `llvm/IR/PassManager.h`, `llvm/IR/Type.h`, `llvm/IR/TypeFinder.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Argument.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Function.h`, `llvm/IR/GlobalAlias.h`, `llvm/IR/GlobalVariable.h`, `llvm/IR/Instruction.h`, `llvm/IR/InstIterator.h`, `llvm/IR/Module.h`, `llvm/IR/PassManager.h`, `llvm/IR/Type.h`, `llvm/IR/TypeFinder.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/CommandLine.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/CommandLine.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `TargetLibraryInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`TargetLibraryInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
