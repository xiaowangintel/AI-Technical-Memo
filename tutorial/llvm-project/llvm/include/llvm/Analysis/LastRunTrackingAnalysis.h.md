# LastRunTrackingAnalysis.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/Analysis/LastRunTrackingAnalysis.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**: This header declares Avoid running redundant pass within LLVM's analysis interfaces and cached program facts layer. / 该头文件在 LLVM 的分析接口与缓存的程序事实层中声明 LastRunTrackingAnalysis 相关接口、类型与辅助能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
//===- LastRunTrackingAnalysis.h - Avoid running redundant pass -*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This is an analysis pass to track a set of passes that have been run, so that
// we can avoid running a pass again if there is no change since the last run of
// the pass.
//
// In this analysis we track a set of passes S for each function with the
// following transition rules:
//   1. If pass P makes changes, set S = {P}.
//   2. If pass P doesn't make changes, set S = S + {P}.
//
// Before running a pass P which satisfies P(P(x)) == P(x), we check if P is in
// S. If so, we skip this pass since we know that there will be no change.
//
```

- **L1**: Banner comment that marks a major file section boundary. / 横幅注释，用于标记文件中的主要分节边界。
- **L2**: Separator comment used to visually divide nearby sections. / 分隔性注释，用于在视觉上划分附近的小节。
- **L3**: Comment documents the nearby API, invariant, or algorithmic intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 这行注释说明了附近 API、不变量或算法意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment documents the nearby API, invariant, or algorithmic intent: `See https://llvm.org/LICENSE.txt for license information.`. / 这行注释说明了附近 API、不变量或算法意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment documents the nearby API, invariant, or algorithmic intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 这行注释说明了附近 API、不变量或算法意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used to visually divide nearby sections. / 分隔性注释，用于在视觉上划分附近的小节。
- **L7**: Banner comment that marks a major file section boundary. / 横幅注释，用于标记文件中的主要分节边界。
- **L8**: Separator comment used to visually divide nearby sections. / 分隔性注释，用于在视觉上划分附近的小节。
- **L9**: Comment documents the nearby API, invariant, or algorithmic intent: `This is an analysis pass to track a set of passes that have been run, so that`. / 这行注释说明了附近 API、不变量或算法意图：`This is an analysis pass to track a set of passes that have been run, so that`。
- **L10**: Comment documents the nearby API, invariant, or algorithmic intent: `we can avoid running a pass again if there is no change since the last run of`. / 这行注释说明了附近 API、不变量或算法意图：`we can avoid running a pass again if there is no change since the last run of`。
- **L11**: Comment documents the nearby API, invariant, or algorithmic intent: `the pass.`. / 这行注释说明了附近 API、不变量或算法意图：`the pass.`。
- **L12**: Separator comment used to visually divide nearby sections. / 分隔性注释，用于在视觉上划分附近的小节。
- **L13**: Comment documents the nearby API, invariant, or algorithmic intent: `In this analysis we track a set of passes S for each function with the`. / 这行注释说明了附近 API、不变量或算法意图：`In this analysis we track a set of passes S for each function with the`。
- **L14**: Comment documents the nearby API, invariant, or algorithmic intent: `following transition rules:`. / 这行注释说明了附近 API、不变量或算法意图：`following transition rules:`。
- **L15**: Comment documents the nearby API, invariant, or algorithmic intent: `1. If pass P makes changes, set S {P}.`. / 这行注释说明了附近 API、不变量或算法意图：`1. If pass P makes changes, set S {P}.`。
- **L16**: Comment documents the nearby API, invariant, or algorithmic intent: `2. If pass P doesn't make changes, set S S + {P}.`. / 这行注释说明了附近 API、不变量或算法意图：`2. If pass P doesn't make changes, set S S + {P}.`。
- **L17**: Separator comment used to visually divide nearby sections. / 分隔性注释，用于在视觉上划分附近的小节。
- **L18**: Comment documents the nearby API, invariant, or algorithmic intent: `Before running a pass P which satisfies P(P(x)) P(x), we check if P is in`. / 这行注释说明了附近 API、不变量或算法意图：`Before running a pass P which satisfies P(P(x)) P(x), we check if P is in`。
- **L19**: Comment documents the nearby API, invariant, or algorithmic intent: `S. If so, we skip this pass since we know that there will be no change.`. / 这行注释说明了附近 API、不变量或算法意图：`S. If so, we skip this pass since we know that there will be no change.`。
- **L20**: Separator comment used to visually divide nearby sections. / 分隔性注释，用于在视觉上划分附近的小节。

### Lines 21-40

```cpp
// Notes:
//   1. Some transform passes have parameters that may vary in the optimization
//   pipeline. We should check if parameters in current run is compatible with
//   that in the last run.
//   2. This pass only tracks at the module/function level. Loop passes are not
//   supported for now.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_ANALYSIS_LASTRUNTRACKINGANALYSIS_H
#define LLVM_ANALYSIS_LASTRUNTRACKINGANALYSIS_H

#include "llvm/ADT/DenseMap.h"
#include "llvm/IR/PassManager.h"
#include "llvm/Support/Compiler.h"
#include <functional>

namespace llvm {

/// This class is used to track the last run of a set of module/function passes.
```

- **L21**: Comment documents the nearby API, invariant, or algorithmic intent: `Notes:`. / 这行注释说明了附近 API、不变量或算法意图：`Notes:`。
- **L22**: Comment documents the nearby API, invariant, or algorithmic intent: `1. Some transform passes have parameters that may vary in the optimization`. / 这行注释说明了附近 API、不变量或算法意图：`1. Some transform passes have parameters that may vary in the optimization`。
- **L23**: Comment documents the nearby API, invariant, or algorithmic intent: `pipeline. We should check if parameters in current run is compatible with`. / 这行注释说明了附近 API、不变量或算法意图：`pipeline. We should check if parameters in current run is compatible with`。
- **L24**: Comment documents the nearby API, invariant, or algorithmic intent: `that in the last run.`. / 这行注释说明了附近 API、不变量或算法意图：`that in the last run.`。
- **L25**: Comment documents the nearby API, invariant, or algorithmic intent: `2. This pass only tracks at the module/function level. Loop passes are not`. / 这行注释说明了附近 API、不变量或算法意图：`2. This pass only tracks at the module/function level. Loop passes are not`。
- **L26**: Comment documents the nearby API, invariant, or algorithmic intent: `supported for now.`. / 这行注释说明了附近 API、不变量或算法意图：`supported for now.`。
- **L27**: Separator comment used to visually divide nearby sections. / 分隔性注释，用于在视觉上划分附近的小节。
- **L28**: Banner comment that marks a major file section boundary. / 横幅注释，用于标记文件中的主要分节边界。
- **L29**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L30**: Starts a preprocessor guard or conditional branch keyed by `LLVM_ANALYSIS_LASTRUNTRACKINGANALYSIS_H`. / 开始一个由 `LLVM_ANALYSIS_LASTRUNTRACKINGANALYSIS_H` 控制的预处理保护或条件分支。
- **L31**: Defines macro `LLVM_ANALYSIS_LASTRUNTRACKINGANALYSIS_H` for later conditional compilation, generated entries, or annotations. / 定义宏 `LLVM_ANALYSIS_LASTRUNTRACKINGANALYSIS_H`，供后续条件编译、生成条目或注解使用。
- **L32**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L33**: Includes `llvm/ADT/DenseMap.h` to access LLVM ADT containers and generic helpers. / 引入 `llvm/ADT/DenseMap.h` 以使用LLVM ADT 容器与通用辅助工具。
- **L34**: Includes `llvm/IR/PassManager.h` to access LLVM IR core types and helper APIs. / 引入 `llvm/IR/PassManager.h` 以使用LLVM IR 核心类型与辅助 API。
- **L35**: Includes `llvm/Support/Compiler.h` to access LLVM support-library utilities. / 引入 `llvm/Support/Compiler.h` 以使用LLVM 支持库工具。
- **L36**: Includes `functional` to access standard or external library facilities. / 引入 `functional` 以使用标准库或外部库能力。
- **L37**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L38**: Opens namespace `llvm` to scope the following declarations under the intended API surface. / 打开命名空间 `llvm`，让后续声明归属到预期的 API 作用域中。
- **L39**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L40**: Comment documents the nearby API, invariant, or algorithmic intent: `This class is used to track the last run of a set of module/function passes.`. / 这行注释说明了附近 API、不变量或算法意图：`This class is used to track the last run of a set of module/function passes.`。

### Lines 41-60

```cpp
/// Invalidation are conservatively handled by the pass manager if a pass
/// doesn't explicitly preserve the result.
/// If we want to skip a pass, we should define a unique ID \p PassID to
/// identify the pass, which is usually a pointer to a static member. If a pass
/// has parameters, they should be stored in a struct \p OptionT with a method
/// bool isCompatibleWith(const OptionT& LastOpt) const to check compatibility.
class LastRunTrackingInfo {
public:
  using PassID = const void *;
  using OptionPtr = const void *;
  // CompatibilityCheckFn is a closure that stores the parameters of last run.
  using CompatibilityCheckFn = std::function<bool(OptionPtr)>;

  /// Check if we should skip a pass.
  /// \param ID The unique ID of the pass.
  /// \param Opt The parameters of the pass. If the pass has no parameters, use
  /// shouldSkip(PassID ID) instead.
  /// \return True if we should skip the pass.
  /// \sa shouldSkip(PassID ID)
  template <typename OptionT>
```

- **L41**: Comment documents the nearby API, invariant, or algorithmic intent: `Invalidation are conservatively handled by the pass manager if a pass`. / 这行注释说明了附近 API、不变量或算法意图：`Invalidation are conservatively handled by the pass manager if a pass`。
- **L42**: Comment documents the nearby API, invariant, or algorithmic intent: `doesn't explicitly preserve the result.`. / 这行注释说明了附近 API、不变量或算法意图：`doesn't explicitly preserve the result.`。
- **L43**: Comment documents the nearby API, invariant, or algorithmic intent: `If we want to skip a pass, we should define a unique ID \p PassID to`. / 这行注释说明了附近 API、不变量或算法意图：`If we want to skip a pass, we should define a unique ID \p PassID to`。
- **L44**: Comment documents the nearby API, invariant, or algorithmic intent: `identify the pass, which is usually a pointer to a static member. If a pass`. / 这行注释说明了附近 API、不变量或算法意图：`identify the pass, which is usually a pointer to a static member. If a pass`。
- **L45**: Comment documents the nearby API, invariant, or algorithmic intent: `has parameters, they should be stored in a struct \p OptionT with a method`. / 这行注释说明了附近 API、不变量或算法意图：`has parameters, they should be stored in a struct \p OptionT with a method`。
- **L46**: Comment documents the nearby API, invariant, or algorithmic intent: `bool isCompatibleWith(const OptionT& LastOpt) const to check compatibility.`. / 这行注释说明了附近 API、不变量或算法意图：`bool isCompatibleWith(const OptionT& LastOpt) const to check compatibility.`。
- **L47**: Declares class `LastRunTrackingInfo`, establishing a named type used by later APIs or implementations. / 声明 class `LastRunTrackingInfo`，建立后续 API 或实现会使用到的命名类型。
- **L48**: Switches to the `public` access section for the members that follow. / 切换到 `public` 访问区段，以约束后续成员的可见性。
- **L49**: Defines type alias `PassID` to present a clearer or more convenient name for an existing type. / 定义类型别名 `PassID`，为已有类型提供更清晰或更方便的名称。
- **L50**: Defines type alias `OptionPtr` to present a clearer or more convenient name for an existing type. / 定义类型别名 `OptionPtr`，为已有类型提供更清晰或更方便的名称。
- **L51**: Comment documents the nearby API, invariant, or algorithmic intent: `CompatibilityCheckFn is a closure that stores the parameters of last run.`. / 这行注释说明了附近 API、不变量或算法意图：`CompatibilityCheckFn is a closure that stores the parameters of last run.`。
- **L52**: Defines type alias `CompatibilityCheckFn` to present a clearer or more convenient name for an existing type. / 定义类型别名 `CompatibilityCheckFn`，为已有类型提供更清晰或更方便的名称。
- **L53**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L54**: Comment documents the nearby API, invariant, or algorithmic intent: `Check if we should skip a pass.`. / 这行注释说明了附近 API、不变量或算法意图：`Check if we should skip a pass.`。
- **L55**: Comment documents the nearby API, invariant, or algorithmic intent: `\param ID The unique ID of the pass.`. / 这行注释说明了附近 API、不变量或算法意图：`\param ID The unique ID of the pass.`。
- **L56**: Comment documents the nearby API, invariant, or algorithmic intent: `\param Opt The parameters of the pass. If the pass has no parameters, use`. / 这行注释说明了附近 API、不变量或算法意图：`\param Opt The parameters of the pass. If the pass has no parameters, use`。
- **L57**: Comment documents the nearby API, invariant, or algorithmic intent: `shouldSkip(PassID ID) instead.`. / 这行注释说明了附近 API、不变量或算法意图：`shouldSkip(PassID ID) instead.`。
- **L58**: Comment documents the nearby API, invariant, or algorithmic intent: `\return True if we should skip the pass.`. / 这行注释说明了附近 API、不变量或算法意图：`\return True if we should skip the pass.`。
- **L59**: Comment documents the nearby API, invariant, or algorithmic intent: `\sa shouldSkip(PassID ID)`. / 这行注释说明了附近 API、不变量或算法意图：`\sa shouldSkip(PassID ID)`。
- **L60**: Begins a template parameter list, making the following declaration generic over types or values. / 开始模板参数列表，使后续声明能够对类型或常量进行泛化。

### Lines 61-80

```cpp
  bool shouldSkip(PassID ID, const OptionT &Opt) const {
    return shouldSkipImpl(ID, &Opt);
  }
  bool shouldSkip(PassID ID) const { return shouldSkipImpl(ID, nullptr); }

  /// Update the tracking info.
  /// \param ID The unique ID of the pass.
  /// \param Changed Whether the pass makes changes.
  /// \param Opt The parameters of the pass. It must have the same type as the
  /// parameters of the last run. If the pass has no parameters, use
  /// update(PassID ID, bool Changed) instead.
  /// \sa update(PassID ID, bool Changed)
  template <typename OptionT>
  void update(PassID ID, bool Changed, const OptionT &Opt) {
    updateImpl(ID, Changed, [Opt](OptionPtr Ptr) {
      return static_cast<const OptionT *>(Ptr)->isCompatibleWith(Opt);
    });
  }
  void update(PassID ID, bool Changed) {
    updateImpl(ID, Changed, CompatibilityCheckFn{});
```

- **L61**: Introduces the function definition for `shouldSkip`, one of the callable entry points exposed in this scope. / 给出 `shouldSkip` 的函数定义，它是此作用域中的可调用入口之一。
- **L62**: Returns a value or reference to the caller from the current function. / 从当前函数向调用者返回一个值或引用。
- **L63**: Opens or closes a lexical scope that groups the surrounding statements. / 打开或关闭一个词法作用域，用来组织附近的语句。
- **L64**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L65**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L66**: Comment documents the nearby API, invariant, or algorithmic intent: `Update the tracking info.`. / 这行注释说明了附近 API、不变量或算法意图：`Update the tracking info.`。
- **L67**: Comment documents the nearby API, invariant, or algorithmic intent: `\param ID The unique ID of the pass.`. / 这行注释说明了附近 API、不变量或算法意图：`\param ID The unique ID of the pass.`。
- **L68**: Comment documents the nearby API, invariant, or algorithmic intent: `\param Changed Whether the pass makes changes.`. / 这行注释说明了附近 API、不变量或算法意图：`\param Changed Whether the pass makes changes.`。
- **L69**: Comment documents the nearby API, invariant, or algorithmic intent: `\param Opt The parameters of the pass. It must have the same type as the`. / 这行注释说明了附近 API、不变量或算法意图：`\param Opt The parameters of the pass. It must have the same type as the`。
- **L70**: Comment documents the nearby API, invariant, or algorithmic intent: `parameters of the last run. If the pass has no parameters, use`. / 这行注释说明了附近 API、不变量或算法意图：`parameters of the last run. If the pass has no parameters, use`。
- **L71**: Comment documents the nearby API, invariant, or algorithmic intent: `update(PassID ID, bool Changed) instead.`. / 这行注释说明了附近 API、不变量或算法意图：`update(PassID ID, bool Changed) instead.`。
- **L72**: Comment documents the nearby API, invariant, or algorithmic intent: `\sa update(PassID ID, bool Changed)`. / 这行注释说明了附近 API、不变量或算法意图：`\sa update(PassID ID, bool Changed)`。
- **L73**: Begins a template parameter list, making the following declaration generic over types or values. / 开始模板参数列表，使后续声明能够对类型或常量进行泛化。
- **L74**: Introduces the function definition for `update`, one of the callable entry points exposed in this scope. / 给出 `update` 的函数定义，它是此作用域中的可调用入口之一。
- **L75**: Introduces the function definition for `updateImpl`, one of the callable entry points exposed in this scope. / 给出 `updateImpl` 的函数定义，它是此作用域中的可调用入口之一。
- **L76**: Returns a value or reference to the caller from the current function. / 从当前函数向调用者返回一个值或引用。
- **L77**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L78**: Opens or closes a lexical scope that groups the surrounding statements. / 打开或关闭一个词法作用域，用来组织附近的语句。
- **L79**: Introduces the function definition for `update`, one of the callable entry points exposed in this scope. / 给出 `update` 的函数定义，它是此作用域中的可调用入口之一。
- **L80**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。

### Lines 81-100

```cpp
  }

private:
  LLVM_ABI bool shouldSkipImpl(PassID ID, OptionPtr Ptr) const;
  LLVM_ABI void updateImpl(PassID ID, bool Changed,
                           CompatibilityCheckFn CheckFn);

  DenseMap<PassID, CompatibilityCheckFn> TrackedPasses;
};

/// A function/module analysis which provides an empty \c LastRunTrackingInfo.
class LastRunTrackingAnalysis final
    : public AnalysisInfoMixin<LastRunTrackingAnalysis> {
  friend AnalysisInfoMixin<LastRunTrackingAnalysis>;
  LLVM_ABI static AnalysisKey Key;

public:
  using Result = LastRunTrackingInfo;
  LastRunTrackingInfo run(Function &F, FunctionAnalysisManager &) {
    return LastRunTrackingInfo();
```

- **L81**: Opens or closes a lexical scope that groups the surrounding statements. / 打开或关闭一个词法作用域，用来组织附近的语句。
- **L82**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L83**: Switches to the `private` access section for the members that follow. / 切换到 `private` 访问区段，以约束后续成员的可见性。
- **L84**: Introduces the function declaration for `shouldSkipImpl`, one of the callable entry points exposed in this scope. / 给出 `shouldSkipImpl` 的函数声明，它是此作用域中的可调用入口之一。
- **L85**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L86**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L87**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L88**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L89**: Ends the current type or aggregate definition. / 结束当前的类型或聚合定义。
- **L90**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L91**: Comment documents the nearby API, invariant, or algorithmic intent: `A function/module analysis which provides an empty \c LastRunTrackingInfo.`. / 这行注释说明了附近 API、不变量或算法意图：`A function/module analysis which provides an empty \c LastRunTrackingInfo.`。
- **L92**: Declares class `LastRunTrackingAnalysis`, establishing a named type used by later APIs or implementations. / 声明 class `LastRunTrackingAnalysis`，建立后续 API 或实现会使用到的命名类型。
- **L93**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L94**: Declares a friend relationship so another type or function can access non-public members. / 声明友元关系，使另一个类型或函数可以访问非公有成员。
- **L95**: Provides a declaration or standalone statement that contributes to the surrounding API or implementation. / 给出一条声明或独立语句，为周围的 API 或实现提供支撑。
- **L96**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L97**: Switches to the `public` access section for the members that follow. / 切换到 `public` 访问区段，以约束后续成员的可见性。
- **L98**: Defines type alias `Result` to present a clearer or more convenient name for an existing type. / 定义类型别名 `Result`，为已有类型提供更清晰或更方便的名称。
- **L99**: Introduces the function definition for `run`, one of the callable entry points exposed in this scope. / 给出 `run` 的函数定义，它是此作用域中的可调用入口之一。
- **L100**: Returns a value or reference to the caller from the current function. / 从当前函数向调用者返回一个值或引用。

### Lines 101-109

```cpp
  }
  LastRunTrackingInfo run(Module &M, ModuleAnalysisManager &) {
    return LastRunTrackingInfo();
  }
};

} // namespace llvm

#endif // LLVM_ANALYSIS_LASTRUNTRACKINGANALYSIS_H
```

- **L101**: Opens or closes a lexical scope that groups the surrounding statements. / 打开或关闭一个词法作用域，用来组织附近的语句。
- **L102**: Introduces the function definition for `run`, one of the callable entry points exposed in this scope. / 给出 `run` 的函数定义，它是此作用域中的可调用入口之一。
- **L103**: Returns a value or reference to the caller from the current function. / 从当前函数向调用者返回一个值或引用。
- **L104**: Opens or closes a lexical scope that groups the surrounding statements. / 打开或关闭一个词法作用域，用来组织附近的语句。
- **L105**: Ends the current type or aggregate definition. / 结束当前的类型或聚合定义。
- **L106**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L107**: Closes namespace `llvm` and returns to the outer scope. / 关闭命名空间 `llvm`，并返回外层作用域。
- **L108**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L109**: Closes the current preprocessor conditional or header guard. / 结束当前的预处理条件块或头文件保护。

## Key Concepts / 关键概念

- EN: Layer: `Analysis` belongs to LLVM's analysis interfaces and cached program facts subsystem.
  - CN: 层次：`Analysis` 属于 LLVM 的分析接口与缓存的程序事实子系统。
- EN: Primary entities: `LastRunTrackingInfo, PassID, OptionPtr, CompatibilityCheckFn, function<bool, shouldSkip, update, updateImpl` are the main named types, records, or call points surfaced in this file.
  - CN: 核心实体：`LastRunTrackingInfo, PassID, OptionPtr, CompatibilityCheckFn, function<bool, shouldSkip, update, updateImpl` 是该文件中出现的主要命名类型、记录或调用入口。
- EN: Header role: the file mainly defines reusable declarations, templates, inline helpers, and API contracts.
  - CN: 头文件角色：该文件主要定义可复用的声明、模板、内联辅助函数以及 API 契约。
- EN: Generic programming: templates make the abstractions reusable across types, policies, or compile-time constants.
  - CN: 泛型编程：模板使这些抽象能够在不同类型、策略或编译期常量之间复用。

## Dependencies / 依赖关系

- EN: Core LLVM interfaces: `llvm/IR/PassManager.h` contribute IR objects, record bases, or subsystem declarations that this file builds on.
  - CN: 核心 LLVM 接口：`llvm/IR/PassManager.h` 提供了本文件建立其上的 IR 对象、记录基类或子系统声明。
- EN: Utility infrastructure: `llvm/ADT/DenseMap.h`, `llvm/Support/Compiler.h` supply containers, traits, diagnostics, ABI helpers, or portability wrappers.
  - CN: 工具基础设施：`llvm/ADT/DenseMap.h`, `llvm/Support/Compiler.h` 提供了容器、traits、诊断、ABI 辅助或可移植性封装。
- EN: Standard/external headers: `functional` provide language-level facilities used alongside LLVM APIs.
  - CN: 标准库/外部头文件：`functional` 提供了与 LLVM API 配合使用的语言级能力。
