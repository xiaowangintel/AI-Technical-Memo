# ModuleSummaryAnalysis.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/Analysis/ModuleSummaryAnalysis.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**: This header declares Module summary index builder within LLVM's analysis interfaces and cached program facts layer. / 该头文件在 LLVM 的分析接口与缓存的程序事实层中声明 ModuleSummaryAnalysis 相关接口、类型与辅助能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
//===- ModuleSummaryAnalysis.h - Module summary index builder ---*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
/// \file
/// This is the interface to build a ModuleSummaryIndex for a module.
///
//===----------------------------------------------------------------------===//

#ifndef LLVM_ANALYSIS_MODULESUMMARYANALYSIS_H
#define LLVM_ANALYSIS_MODULESUMMARYANALYSIS_H

#include "llvm/IR/ModuleSummaryIndex.h"
#include "llvm/IR/PassManager.h"
#include "llvm/Pass.h"
#include "llvm/Support/Compiler.h"
#include <functional>
```

- **L1**: Banner comment that marks a major file section boundary. / 横幅注释，用于标记文件中的主要分节边界。
- **L2**: Separator comment used to visually divide nearby sections. / 分隔性注释，用于在视觉上划分附近的小节。
- **L3**: Comment documents the nearby API, invariant, or algorithmic intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 这行注释说明了附近 API、不变量或算法意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment documents the nearby API, invariant, or algorithmic intent: `See https://llvm.org/LICENSE.txt for license information.`. / 这行注释说明了附近 API、不变量或算法意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment documents the nearby API, invariant, or algorithmic intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 这行注释说明了附近 API、不变量或算法意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used to visually divide nearby sections. / 分隔性注释，用于在视觉上划分附近的小节。
- **L7**: Banner comment that marks a major file section boundary. / 横幅注释，用于标记文件中的主要分节边界。
- **L8**: Comment documents the nearby API, invariant, or algorithmic intent: `\file`. / 这行注释说明了附近 API、不变量或算法意图：`\file`。
- **L9**: Comment documents the nearby API, invariant, or algorithmic intent: `This is the interface to build a ModuleSummaryIndex for a module.`. / 这行注释说明了附近 API、不变量或算法意图：`This is the interface to build a ModuleSummaryIndex for a module.`。
- **L10**: Separator comment used to visually divide nearby sections. / 分隔性注释，用于在视觉上划分附近的小节。
- **L11**: Banner comment that marks a major file section boundary. / 横幅注释，用于标记文件中的主要分节边界。
- **L12**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L13**: Starts a preprocessor guard or conditional branch keyed by `LLVM_ANALYSIS_MODULESUMMARYANALYSIS_H`. / 开始一个由 `LLVM_ANALYSIS_MODULESUMMARYANALYSIS_H` 控制的预处理保护或条件分支。
- **L14**: Defines macro `LLVM_ANALYSIS_MODULESUMMARYANALYSIS_H` for later conditional compilation, generated entries, or annotations. / 定义宏 `LLVM_ANALYSIS_MODULESUMMARYANALYSIS_H`，供后续条件编译、生成条目或注解使用。
- **L15**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L16**: Includes `llvm/IR/ModuleSummaryIndex.h` to access LLVM IR core types and helper APIs. / 引入 `llvm/IR/ModuleSummaryIndex.h` 以使用LLVM IR 核心类型与辅助 API。
- **L17**: Includes `llvm/IR/PassManager.h` to access LLVM IR core types and helper APIs. / 引入 `llvm/IR/PassManager.h` 以使用LLVM IR 核心类型与辅助 API。
- **L18**: Includes `llvm/Pass.h` to access standard or external library facilities. / 引入 `llvm/Pass.h` 以使用标准库或外部库能力。
- **L19**: Includes `llvm/Support/Compiler.h` to access LLVM support-library utilities. / 引入 `llvm/Support/Compiler.h` 以使用LLVM 支持库工具。
- **L20**: Includes `functional` to access standard or external library facilities. / 引入 `functional` 以使用标准库或外部库能力。

### Lines 21-40

```cpp
#include <optional>

namespace llvm {

class BlockFrequencyInfo;
class Function;
class Module;
class ProfileSummaryInfo;
class StackSafetyInfo;

/// Direct function to compute a \c ModuleSummaryIndex from a given module.
///
/// If operating within a pass manager which has defined ways to compute the \c
/// BlockFrequencyInfo for a given function, that can be provided via
/// a std::function callback. Otherwise, this routine will manually construct
/// that information.
LLVM_ABI ModuleSummaryIndex buildModuleSummaryIndex(
    const Module &M,
    std::function<BlockFrequencyInfo *(const Function &F)> GetBFICallback,
    ProfileSummaryInfo *PSI,
```

- **L21**: Includes `optional` to access standard or external library facilities. / 引入 `optional` 以使用标准库或外部库能力。
- **L22**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L23**: Opens namespace `llvm` to scope the following declarations under the intended API surface. / 打开命名空间 `llvm`，让后续声明归属到预期的 API 作用域中。
- **L24**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L25**: Declares class `BlockFrequencyInfo`, establishing a named type used by later APIs or implementations. / 声明 class `BlockFrequencyInfo`，建立后续 API 或实现会使用到的命名类型。
- **L26**: Declares class `Function`, establishing a named type used by later APIs or implementations. / 声明 class `Function`，建立后续 API 或实现会使用到的命名类型。
- **L27**: Declares class `Module`, establishing a named type used by later APIs or implementations. / 声明 class `Module`，建立后续 API 或实现会使用到的命名类型。
- **L28**: Declares class `ProfileSummaryInfo`, establishing a named type used by later APIs or implementations. / 声明 class `ProfileSummaryInfo`，建立后续 API 或实现会使用到的命名类型。
- **L29**: Declares class `StackSafetyInfo`, establishing a named type used by later APIs or implementations. / 声明 class `StackSafetyInfo`，建立后续 API 或实现会使用到的命名类型。
- **L30**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L31**: Comment documents the nearby API, invariant, or algorithmic intent: `Direct function to compute a \c ModuleSummaryIndex from a given module.`. / 这行注释说明了附近 API、不变量或算法意图：`Direct function to compute a \c ModuleSummaryIndex from a given module.`。
- **L32**: Separator comment used to visually divide nearby sections. / 分隔性注释，用于在视觉上划分附近的小节。
- **L33**: Comment documents the nearby API, invariant, or algorithmic intent: `If operating within a pass manager which has defined ways to compute the \c`. / 这行注释说明了附近 API、不变量或算法意图：`If operating within a pass manager which has defined ways to compute the \c`。
- **L34**: Comment documents the nearby API, invariant, or algorithmic intent: `BlockFrequencyInfo for a given function, that can be provided via`. / 这行注释说明了附近 API、不变量或算法意图：`BlockFrequencyInfo for a given function, that can be provided via`。
- **L35**: Comment documents the nearby API, invariant, or algorithmic intent: `a std::function callback. Otherwise, this routine will manually construct`. / 这行注释说明了附近 API、不变量或算法意图：`a std::function callback. Otherwise, this routine will manually construct`。
- **L36**: Comment documents the nearby API, invariant, or algorithmic intent: `that information.`. / 这行注释说明了附近 API、不变量或算法意图：`that information.`。
- **L37**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L38**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L39**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L40**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。

### Lines 41-60

```cpp
    std::function<const StackSafetyInfo *(const Function &F)> GetSSICallback =
        [](const Function &F) -> const StackSafetyInfo * { return nullptr; });

/// Analysis pass to provide the ModuleSummaryIndex object.
class ModuleSummaryIndexAnalysis
    : public AnalysisInfoMixin<ModuleSummaryIndexAnalysis> {
  friend AnalysisInfoMixin<ModuleSummaryIndexAnalysis>;

  LLVM_ABI static AnalysisKey Key;

public:
  using Result = ModuleSummaryIndex;

  LLVM_ABI Result run(Module &M, ModuleAnalysisManager &AM);
};

/// Legacy wrapper pass to provide the ModuleSummaryIndex object.
class LLVM_ABI ModuleSummaryIndexWrapperPass : public ModulePass {
  std::optional<ModuleSummaryIndex> Index;

```

- **L41**: Continues building or assigning `GetSSICallback` as part of a larger multi-line declaration or expression. / 作为更大规模多行声明或表达式的一部分，继续构造或赋值 `GetSSICallback`。
- **L42**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L43**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L44**: Comment documents the nearby API, invariant, or algorithmic intent: `Analysis pass to provide the ModuleSummaryIndex object.`. / 这行注释说明了附近 API、不变量或算法意图：`Analysis pass to provide the ModuleSummaryIndex object.`。
- **L45**: Declares class `ModuleSummaryIndexAnalysis`, establishing a named type used by later APIs or implementations. / 声明 class `ModuleSummaryIndexAnalysis`，建立后续 API 或实现会使用到的命名类型。
- **L46**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L47**: Declares a friend relationship so another type or function can access non-public members. / 声明友元关系，使另一个类型或函数可以访问非公有成员。
- **L48**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L49**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L50**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L51**: Switches to the `public` access section for the members that follow. / 切换到 `public` 访问区段，以约束后续成员的可见性。
- **L52**: Defines type alias `Result` to present a clearer or more convenient name for an existing type. / 定义类型别名 `Result`，为已有类型提供更清晰或更方便的名称。
- **L53**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L54**: Introduces the function declaration for `run`, one of the callable entry points exposed in this scope. / 给出 `run` 的函数声明，它是此作用域中的可调用入口之一。
- **L55**: Ends the current type or aggregate definition. / 结束当前的类型或聚合定义。
- **L56**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L57**: Comment documents the nearby API, invariant, or algorithmic intent: `Legacy wrapper pass to provide the ModuleSummaryIndex object.`. / 这行注释说明了附近 API、不变量或算法意图：`Legacy wrapper pass to provide the ModuleSummaryIndex object.`。
- **L58**: Declares class `LLVM_ABI`, establishing a named type used by later APIs or implementations. / 声明 class `LLVM_ABI`，建立后续 API 或实现会使用到的命名类型。
- **L59**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L60**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 61-80

```cpp
public:
  static char ID;

  ModuleSummaryIndexWrapperPass();

  /// Get the index built by pass
  ModuleSummaryIndex &getIndex() { return *Index; }
  const ModuleSummaryIndex &getIndex() const { return *Index; }

  bool runOnModule(Module &M) override;
  bool doFinalization(Module &M) override;
  void getAnalysisUsage(AnalysisUsage &AU) const override;
};

//===--------------------------------------------------------------------===//
//
// createModuleSummaryIndexWrapperPass - This pass builds a ModuleSummaryIndex
// object for the module, to be written to bitcode or LLVM assembly.
//
LLVM_ABI ModulePass *createModuleSummaryIndexWrapperPass();
```

- **L61**: Switches to the `public` access section for the members that follow. / 切换到 `public` 访问区段，以约束后续成员的可见性。
- **L62**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L63**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L64**: Introduces the function declaration for `ModuleSummaryIndexWrapperPass`, one of the callable entry points exposed in this scope. / 给出 `ModuleSummaryIndexWrapperPass` 的函数声明，它是此作用域中的可调用入口之一。
- **L65**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L66**: Comment documents the nearby API, invariant, or algorithmic intent: `Get the index built by pass`. / 这行注释说明了附近 API、不变量或算法意图：`Get the index built by pass`。
- **L67**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L68**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L69**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L70**: Introduces the function declaration for `runOnModule`, one of the callable entry points exposed in this scope. / 给出 `runOnModule` 的函数声明，它是此作用域中的可调用入口之一。
- **L71**: Introduces the function declaration for `doFinalization`, one of the callable entry points exposed in this scope. / 给出 `doFinalization` 的函数声明，它是此作用域中的可调用入口之一。
- **L72**: Introduces the function declaration for `getAnalysisUsage`, one of the callable entry points exposed in this scope. / 给出 `getAnalysisUsage` 的函数声明，它是此作用域中的可调用入口之一。
- **L73**: Ends the current type or aggregate definition. / 结束当前的类型或聚合定义。
- **L74**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L75**: Banner comment that marks a major file section boundary. / 横幅注释，用于标记文件中的主要分节边界。
- **L76**: Separator comment used to visually divide nearby sections. / 分隔性注释，用于在视觉上划分附近的小节。
- **L77**: Comment documents the nearby API, invariant, or algorithmic intent: `createModuleSummaryIndexWrapperPass - This pass builds a ModuleSummaryIndex`. / 这行注释说明了附近 API、不变量或算法意图：`createModuleSummaryIndexWrapperPass - This pass builds a ModuleSummaryIndex`。
- **L78**: Comment documents the nearby API, invariant, or algorithmic intent: `object for the module, to be written to bitcode or LLVM assembly.`. / 这行注释说明了附近 API、不变量或算法意图：`object for the module, to be written to bitcode or LLVM assembly.`。
- **L79**: Separator comment used to visually divide nearby sections. / 分隔性注释，用于在视觉上划分附近的小节。
- **L80**: Introduces the function declaration for `createModuleSummaryIndexWrapperPass`, one of the callable entry points exposed in this scope. / 给出 `createModuleSummaryIndexWrapperPass` 的函数声明，它是此作用域中的可调用入口之一。

### Lines 81-100

```cpp

/// Legacy wrapper pass to provide the ModuleSummaryIndex object.
class LLVM_ABI ImmutableModuleSummaryIndexWrapperPass : public ImmutablePass {
  const ModuleSummaryIndex *Index;

public:
  static char ID;

  ImmutableModuleSummaryIndexWrapperPass(
      const ModuleSummaryIndex *Index = nullptr);
  const ModuleSummaryIndex *getIndex() const { return Index; }
  void getAnalysisUsage(AnalysisUsage &AU) const override;
};

//===--------------------------------------------------------------------===//
//
// ImmutableModuleSummaryIndexWrapperPass - This pass wrap provided
// ModuleSummaryIndex object for the module, to be used by other passes.
//
LLVM_ABI ImmutablePass *
```

- **L81**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L82**: Comment documents the nearby API, invariant, or algorithmic intent: `Legacy wrapper pass to provide the ModuleSummaryIndex object.`. / 这行注释说明了附近 API、不变量或算法意图：`Legacy wrapper pass to provide the ModuleSummaryIndex object.`。
- **L83**: Declares class `LLVM_ABI`, establishing a named type used by later APIs or implementations. / 声明 class `LLVM_ABI`，建立后续 API 或实现会使用到的命名类型。
- **L84**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L85**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L86**: Switches to the `public` access section for the members that follow. / 切换到 `public` 访问区段，以约束后续成员的可见性。
- **L87**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L88**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L89**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L90**: Initializes or assigns `Index` using the expression on the right-hand side. / 使用右侧表达式初始化或赋值 `Index`。
- **L91**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L92**: Introduces the function declaration for `getAnalysisUsage`, one of the callable entry points exposed in this scope. / 给出 `getAnalysisUsage` 的函数声明，它是此作用域中的可调用入口之一。
- **L93**: Ends the current type or aggregate definition. / 结束当前的类型或聚合定义。
- **L94**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L95**: Banner comment that marks a major file section boundary. / 横幅注释，用于标记文件中的主要分节边界。
- **L96**: Separator comment used to visually divide nearby sections. / 分隔性注释，用于在视觉上划分附近的小节。
- **L97**: Comment documents the nearby API, invariant, or algorithmic intent: `ImmutableModuleSummaryIndexWrapperPass - This pass wrap provided`. / 这行注释说明了附近 API、不变量或算法意图：`ImmutableModuleSummaryIndexWrapperPass - This pass wrap provided`。
- **L98**: Comment documents the nearby API, invariant, or algorithmic intent: `ModuleSummaryIndex object for the module, to be used by other passes.`. / 这行注释说明了附近 API、不变量或算法意图：`ModuleSummaryIndex object for the module, to be used by other passes.`。
- **L99**: Separator comment used to visually divide nearby sections. / 分隔性注释，用于在视觉上划分附近的小节。
- **L100**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。

### Lines 101-109

```cpp
createImmutableModuleSummaryIndexWrapperPass(const ModuleSummaryIndex *Index);

/// Returns true if the instruction could have memprof metadata, used to ensure
/// consistency between summary analysis and the ThinLTO backend processing.
LLVM_ABI bool mayHaveMemprofSummary(const CallBase *CB);

} // end namespace llvm

#endif // LLVM_ANALYSIS_MODULESUMMARYANALYSIS_H
```

- **L101**: Introduces the function declaration for `createImmutableModuleSummaryIndexWrapperPass`, one of the callable entry points exposed in this scope. / 给出 `createImmutableModuleSummaryIndexWrapperPass` 的函数声明，它是此作用域中的可调用入口之一。
- **L102**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L103**: Comment documents the nearby API, invariant, or algorithmic intent: `Returns true if the instruction could have memprof metadata, used to ensure`. / 这行注释说明了附近 API、不变量或算法意图：`Returns true if the instruction could have memprof metadata, used to ensure`。
- **L104**: Comment documents the nearby API, invariant, or algorithmic intent: `consistency between summary analysis and the ThinLTO backend processing.`. / 这行注释说明了附近 API、不变量或算法意图：`consistency between summary analysis and the ThinLTO backend processing.`。
- **L105**: Introduces the function declaration for `mayHaveMemprofSummary`, one of the callable entry points exposed in this scope. / 给出 `mayHaveMemprofSummary` 的函数声明，它是此作用域中的可调用入口之一。
- **L106**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L107**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L108**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L109**: Closes the current preprocessor conditional or header guard. / 结束当前的预处理条件块或头文件保护。

## Key Concepts / 关键概念

- EN: Layer: `Analysis` belongs to LLVM's analysis interfaces and cached program facts subsystem.
  - CN: 层次：`Analysis` 属于 LLVM 的分析接口与缓存的程序事实子系统。
- EN: Primary entities: `BlockFrequencyInfo, Function, Module, ProfileSummaryInfo, StackSafetyInfo, ModuleSummaryIndexAnalysis, Result, run` are the main named types, records, or call points surfaced in this file.
  - CN: 核心实体：`BlockFrequencyInfo, Function, Module, ProfileSummaryInfo, StackSafetyInfo, ModuleSummaryIndexAnalysis, Result, run` 是该文件中出现的主要命名类型、记录或调用入口。
- EN: Header role: the file mainly defines reusable declarations, templates, inline helpers, and API contracts.
  - CN: 头文件角色：该文件主要定义可复用的声明、模板、内联辅助函数以及 API 契约。
- EN: Include discipline: a conventional header guard prevents duplicate inclusion across translation units.
  - CN: 包含规范：传统头文件保护可防止同一翻译单元中的重复包含。

## Dependencies / 依赖关系

- EN: Core LLVM interfaces: `llvm/IR/ModuleSummaryIndex.h`, `llvm/IR/PassManager.h`, `llvm/Pass.h` contribute IR objects, record bases, or subsystem declarations that this file builds on.
  - CN: 核心 LLVM 接口：`llvm/IR/ModuleSummaryIndex.h`, `llvm/IR/PassManager.h`, `llvm/Pass.h` 提供了本文件建立其上的 IR 对象、记录基类或子系统声明。
- EN: Utility infrastructure: `llvm/Support/Compiler.h` supply containers, traits, diagnostics, ABI helpers, or portability wrappers.
  - CN: 工具基础设施：`llvm/Support/Compiler.h` 提供了容器、traits、诊断、ABI 辅助或可移植性封装。
- EN: Standard/external headers: `functional`, `optional` provide language-level facilities used alongside LLVM APIs.
  - CN: 标准库/外部头文件：`functional`, `optional` 提供了与 LLVM API 配合使用的语言级能力。
