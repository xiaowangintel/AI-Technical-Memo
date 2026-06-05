# ScheduleTreeTransform.h — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/include/polly/ScheduleTreeTransform.h` | `polly/include/polly/ScheduleTreeTransform.h` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Declares public Polly interfaces, passes, analyses, and data structures. | 声明 Polly 的公共接口、Pass、分析能力与数据结构。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

````cpp
//===- polly/ScheduleTreeTransform.h ----------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Make changes to isl's schedule tree data structure.
//
//===----------------------------------------------------------------------===//

````
- **EN**: This block records the standard LLVM file banner and license metadata; mentions schedule-related state that orders statements or iterations; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 给出 LLVM 标准文件头与许可证元数据; 涉及调度相关状态，用于安排语句或迭代顺序; 保留用于解释意图、用法或算法背景的注释.

### Lines 13-24

````cpp
#ifndef POLLY_SCHEDULETREETRANSFORM_H
#define POLLY_SCHEDULETREETRANSFORM_H

#include "polly/Support/ISLTools.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/Support/ErrorHandling.h"
#include "isl/isl-noexceptions.h"
#include <cassert>

namespace polly {
struct BandAttr;

````
- **EN**: This block imports Polly, LLVM-family, ISL, system/standard headers needed by the surrounding code; uses preprocessor directives to control compilation, macros, or include guards; opens or organizes declarations inside a C++ namespace; declares or references types such as `BandAttr`; and continues the surrounding implementation details.
- **CN**: 该代码块 引入周边逻辑所需的 Polly、LLVM-family、ISL、system/standard 头文件; 使用预处理指令控制编译、宏或头文件保护; 在 C++ 命名空间中组织声明或实现; 声明或引用类型，例如 `BandAttr`；并延续周边实现细节。

### Lines 25-40

````cpp
/// This class defines a simple visitor class that may be used for
/// various schedule tree analysis purposes.
template <typename Derived, typename RetTy = void, typename... Args>
struct ScheduleTreeVisitor {
  Derived &getDerived() { return *static_cast<Derived *>(this); }
  const Derived &getDerived() const {
    return *static_cast<const Derived *>(this);
  }

  RetTy visit(isl::schedule_node Node, Args... args) {
    assert(!Node.is_null());
    switch (isl_schedule_node_get_type(Node.get())) {
    case isl_schedule_node_domain:
      assert(isl_schedule_node_n_children(Node.get()) == 1);
      return getDerived().visitDomain(Node.as<isl::schedule_node_domain>(),
                                      std::forward<Args>(args)...);
````
- **EN**: This block declares or references types such as `ScheduleTreeVisitor`; declares or defines routines around `getDerived`, `visit`, `forward<Args>`; contains control flow with 1 switch dispatch(es); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `ScheduleTreeVisitor`; 声明或定义与 `getDerived`, `visit`, `forward<Args>` 相关的例程; 包含控制流结构：1 处分支派发; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 41-56

````cpp
    case isl_schedule_node_band:
      assert(isl_schedule_node_n_children(Node.get()) == 1);
      return getDerived().visitBand(Node.as<isl::schedule_node_band>(),
                                    std::forward<Args>(args)...);
    case isl_schedule_node_sequence:
      assert(isl_schedule_node_n_children(Node.get()) >= 2);
      return getDerived().visitSequence(Node.as<isl::schedule_node_sequence>(),
                                        std::forward<Args>(args)...);
    case isl_schedule_node_set:
      assert(isl_schedule_node_n_children(Node.get()) >= 2);
      return getDerived().visitSet(Node.as<isl::schedule_node_set>(),
                                   std::forward<Args>(args)...);
    case isl_schedule_node_leaf:
      assert(isl_schedule_node_n_children(Node.get()) == 0);
      return getDerived().visitLeaf(Node.as<isl::schedule_node_leaf>(),
                                    std::forward<Args>(args)...);
````
- **EN**: This block declares or defines routines around `forward<Args>`; emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `forward<Args>` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 57-73

````cpp
    case isl_schedule_node_mark:
      assert(isl_schedule_node_n_children(Node.get()) == 1);
      return getDerived().visitMark(Node.as<isl::schedule_node_mark>(),
                                    std::forward<Args>(args)...);
    case isl_schedule_node_extension:
      assert(isl_schedule_node_n_children(Node.get()) == 1);
      return getDerived().visitExtension(
          Node.as<isl::schedule_node_extension>(), std::forward<Args>(args)...);
    case isl_schedule_node_filter:
      assert(isl_schedule_node_n_children(Node.get()) == 1);
      return getDerived().visitFilter(Node.as<isl::schedule_node_filter>(),
                                      std::forward<Args>(args)...);
    default:
      llvm_unreachable("unimplemented schedule node type");
    }
  }

````
- **EN**: This block declares or defines routines around `forward<Args>`, `schedule_node_extension>`; emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `forward<Args>`, `schedule_node_extension>` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 74-88

````cpp
  RetTy visitDomain(isl::schedule_node_domain Domain, Args... args) {
    return getDerived().visitSingleChild(std::move(Domain),
                                         std::forward<Args>(args)...);
  }

  RetTy visitBand(isl::schedule_node_band Band, Args... args) {
    return getDerived().visitSingleChild(std::move(Band),
                                         std::forward<Args>(args)...);
  }

  RetTy visitSequence(isl::schedule_node_sequence Sequence, Args... args) {
    return getDerived().visitMultiChild(std::move(Sequence),
                                        std::forward<Args>(args)...);
  }

````
- **EN**: This block declares or defines routines around `visitDomain`, `forward<Args>`, `visitBand`, `visitSequence`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `visitDomain`, `forward<Args>`, `visitBand`, `visitSequence` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 89-102

````cpp
  RetTy visitSet(isl::schedule_node_set Set, Args... args) {
    return getDerived().visitMultiChild(std::move(Set),
                                        std::forward<Args>(args)...);
  }

  RetTy visitLeaf(isl::schedule_node_leaf Leaf, Args... args) {
    return getDerived().visitNode(std::move(Leaf), std::forward<Args>(args)...);
  }

  RetTy visitMark(isl::schedule_node_mark Mark, Args... args) {
    return getDerived().visitSingleChild(std::move(Mark),
                                         std::forward<Args>(args)...);
  }

````
- **EN**: This block declares or defines routines around `visitSet`, `forward<Args>`, `visitLeaf`, `visitMark`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `visitSet`, `forward<Args>`, `visitLeaf`, `visitMark` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 103-116

````cpp
  RetTy visitExtension(isl::schedule_node_extension Extension, Args... args) {
    return getDerived().visitSingleChild(std::move(Extension),
                                         std::forward<Args>(args)...);
  }

  RetTy visitFilter(isl::schedule_node_filter Filter, Args... args) {
    return getDerived().visitSingleChild(std::move(Filter),
                                         std::forward<Args>(args)...);
  }

  RetTy visitSingleChild(isl::schedule_node Node, Args... args) {
    return getDerived().visitNode(std::move(Node), std::forward<Args>(args)...);
  }

````
- **EN**: This block declares or defines routines around `visitExtension`, `forward<Args>`, `visitFilter`, `visitSingleChild`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `visitExtension`, `forward<Args>`, `visitFilter`, `visitSingleChild` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 117-132

````cpp
  RetTy visitMultiChild(isl::schedule_node Node, Args... args) {
    return getDerived().visitNode(std::move(Node), std::forward<Args>(args)...);
  }

  RetTy visitNode(isl::schedule_node Node, Args... args) {
    llvm_unreachable("Unimplemented other");
  }
};

/// Recursively visit all nodes of a schedule tree.
template <typename Derived, typename RetTy = void, typename... Args>
struct RecursiveScheduleTreeVisitor
    : ScheduleTreeVisitor<Derived, RetTy, Args...> {
  using BaseTy = ScheduleTreeVisitor<Derived, RetTy, Args...>;
  BaseTy &getBase() { return *this; }
  const BaseTy &getBase() const { return *this; }
````
- **EN**: This block declares or references types such as `RecursiveScheduleTreeVisitor`; declares or defines routines around `visitMultiChild`, `visitNode`, `getBase`; emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `RecursiveScheduleTreeVisitor`; 声明或定义与 `visitMultiChild`, `visitNode`, `getBase` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查；并延续周边实现细节。

### Lines 133-148

````cpp
  Derived &getDerived() { return *static_cast<Derived *>(this); }
  const Derived &getDerived() const {
    return *static_cast<const Derived *>(this);
  }

  /// When visiting an entire schedule tree, start at its root node.
  RetTy visit(isl::schedule Schedule, Args... args) {
    return getDerived().visit(Schedule.get_root(), std::forward<Args>(args)...);
  }

  // Necessary to allow overload resolution with the added visit(isl::schedule)
  // overload.
  RetTy visit(isl::schedule_node Node, Args... args) {
    return getBase().visit(Node, std::forward<Args>(args)...);
  }

````
- **EN**: This block declares or defines routines around `getDerived`, `visit`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `getDerived`, `visit` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序；并延续周边实现细节。

### Lines 149-164

````cpp
  /// By default, recursively visit the child nodes.
  RetTy visitNode(isl::schedule_node Node, Args... args) {
    for (unsigned i : rangeIslSize(0, Node.n_children()))
      getDerived().visit(Node.child(i), std::forward<Args>(args)...);
    return RetTy();
  }
};

/// Recursively visit all nodes of a schedule tree while allowing changes.
///
/// The visit methods return an isl::schedule_node that is used to continue
/// visiting the tree. Structural changes such as returning a different node
/// will confuse the visitor.
template <typename Derived, typename... Args>
struct ScheduleNodeRewriter
    : public RecursiveScheduleTreeVisitor<Derived, isl::schedule_node,
````
- **EN**: This block declares or references types such as `ScheduleNodeRewriter`; declares or defines routines around `visitNode`, `getDerived`; contains control flow with 1 loop construct(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `ScheduleNodeRewriter`; 声明或定义与 `visitNode`, `getDerived` 相关的例程; 包含控制流结构：1 处循环; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 165-178

````cpp
                                          Args...> {
  Derived &getDerived() { return *static_cast<Derived *>(this); }
  const Derived &getDerived() const {
    return *static_cast<const Derived *>(this);
  }

  isl::schedule_node visitNode(isl::schedule_node Node, Args... args) {
    return getDerived().visitChildren(Node);
  }

  isl::schedule_node visitChildren(isl::schedule_node Node, Args... args) {
    if (!Node.has_children())
      return Node;

````
- **EN**: This block declares or defines routines around `getDerived`, `visitNode`, `visitChildren`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `getDerived`, `visitNode`, `visitChildren` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 179-192

````cpp
    isl::schedule_node It = Node.first_child();
    while (true) {
      It = getDerived().visit(It, std::forward<Args>(args)...);
      if (!It.has_next_sibling())
        break;
      It = It.next_sibling();
    }
    return It.parent();
  }
};

/// Is this node the marker for its parent band?
bool isBandMark(const isl::schedule_node &Node);

````
- **EN**: This block declares or defines routines around `first_child`, `getDerived`, `next_sibling`, `isBandMark`; contains control flow with 1 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `first_child`, `getDerived`, `next_sibling`, `isBandMark` 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 193-210

````cpp
/// Extract the BandAttr from a band's wrapping marker. Can also pass the band
/// itself and this methods will try to find its wrapping mark. Returns nullptr
/// if the band has not BandAttr.
BandAttr *getBandAttr(isl::schedule_node MarkOrBand);

/// Hoist all domains from extension into the root domain node, such that there
/// are no more extension nodes (which isl does not support for some
/// operations). This assumes that domains added by to extension nodes do not
/// overlap.
isl::schedule hoistExtensionNodes(isl::schedule Sched);

/// Replace the AST band @p BandToUnroll by a sequence of all its iterations.
///
/// The implementation enumerates all points in the partial schedule and creates
/// an ISL sequence node for each point. The number of iterations must be a
/// constant.
isl::schedule applyFullUnroll(isl::schedule_node BandToUnroll);

````
- **EN**: This block declares or defines routines around `getBandAttr`, `hoistExtensionNodes`, `applyFullUnroll`; uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getBandAttr`, `hoistExtensionNodes`, `applyFullUnroll` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序; 保留用于解释意图、用法或算法背景的注释.

### Lines 211-226

````cpp
/// Replace the AST band @p BandToUnroll by a partially unrolled equivalent.
isl::schedule applyPartialUnroll(isl::schedule_node BandToUnroll, int Factor);

/// Loop-distribute the band @p BandToFission as much as possible.
isl::schedule applyMaxFission(isl::schedule_node BandToFission);

/// Build the desired set of partial tile prefixes.
///
/// We build a set of partial tile prefixes, which are prefixes of the vector
/// loop that have exactly VectorWidth iterations.
///
/// 1. Drop all constraints involving the dimension that represents the
///    vector loop.
/// 2. Constrain the last dimension to get a set, which has exactly VectorWidth
///    iterations.
/// 3. Subtract loop domain from it, project out the vector loop dimension and
````
- **EN**: This block declares or defines routines around `applyPartialUnroll`, `applyMaxFission`; uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `applyPartialUnroll`, `applyMaxFission` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序; 保留用于解释意图、用法或算法背景的注释.

### Lines 227-245

````cpp
///    get a set that contains prefixes, which do not have exactly VectorWidth
///    iterations.
/// 4. Project out the vector loop dimension of the set that was build on the
///    first step and subtract the set built on the previous step to get the
///    desired set of prefixes.
///
/// @param ScheduleRange A range of a map, which describes a prefix schedule
///                      relation.
isl::set getPartialTilePrefixes(isl::set ScheduleRange, int VectorWidth);

/// Create an isl::union_set, which describes the isolate option based on
/// IsolateDomain.
///
/// @param IsolateDomain An isl::set whose @p OutDimsNum last dimensions should
///                      belong to the current band node.
/// @param OutDimsNum    A number of dimensions that should belong to
///                      the current band node.
isl::union_set getIsolateOptions(isl::set IsolateDomain, unsigned OutDimsNum);

````
- **EN**: This block declares or defines routines around `getPartialTilePrefixes`, `getIsolateOptions`; uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getPartialTilePrefixes`, `getIsolateOptions` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序; 保留用于解释意图、用法或算法背景的注释.

### Lines 246-265

````cpp
/// Create an isl::union_set, which describes the specified option for the
/// dimension of the current node.
///
/// @param Ctx    An isl::ctx, which is used to create the isl::union_set.
/// @param Option The name of the option.
isl::union_set getDimOptions(isl::ctx Ctx, const char *Option);

/// Tile a schedule node.
///
/// @param Node            The node to tile.
/// @param Identifier      An name that identifies this kind of tiling and
///                        that is used to mark the tiled loops in the
///                        generated AST.
/// @param TileSizes       A vector of tile sizes that should be used for
///                        tiling.
/// @param DefaultTileSize A default tile size that is used for dimensions
///                        that are not covered by the TileSizes vector.
isl::schedule_node tileNode(isl::schedule_node Node, const char *Identifier,
                            llvm::ArrayRef<int> TileSizes, int DefaultTileSize);

````
- **EN**: This block declares or defines routines around `getDimOptions`, `tileNode`; uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getDimOptions`, `tileNode` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序; 保留用于解释意图、用法或算法背景的注释.

### Lines 266-283

````cpp
/// Tile a schedule node and unroll point loops.
///
/// @param Node            The node to register tile.
/// @param TileSizes       A vector of tile sizes that should be used for
///                        tiling.
/// @param DefaultTileSize A default tile size that is used for dimensions
isl::schedule_node applyRegisterTiling(isl::schedule_node Node,
                                       llvm::ArrayRef<int> TileSizes,
                                       int DefaultTileSize);

/// Apply greedy fusion. That is, fuse any loop that is possible to be fused
/// top-down.
///
/// @param Sched  Sched tree to fuse all the loops in.
/// @param Deps   Validity constraints that must be preserved.
isl::schedule applyGreedyFusion(isl::schedule Sched,
                                const isl::union_map &Deps);

````
- **EN**: This block declares or defines routines around `applyRegisterTiling`, `applyGreedyFusion`; uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `applyRegisterTiling`, `applyGreedyFusion` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序; 保留用于解释意图、用法或算法背景的注释.

### Lines 284-286

````cpp
} // namespace polly

#endif // POLLY_SCHEDULETREETRANSFORM_H
````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 涉及调度相关状态，用于安排语句或迭代顺序.

## Key Concepts / 关键概念

- **Schedule construction**
  - **CN**: 调度构建
- **ISL relations and sets**
  - **CN**: ISL 关系与集合
- **Loop transformation**
  - **CN**: 循环变换
- **Template/type wrappers**
  - **CN**: 模板与类型包装
- **Debug and diagnostics**
  - **CN**: 调试与诊断
- **Public interface declarations**
  - **CN**: 公共接口声明

## Dependencies / 依赖关系

- **Polly headers**: `polly/Support/ISLTools.h` — Direct Polly APIs, data structures, or pass interfaces used here.
  **Polly headers（CN）**：`polly/Support/ISLTools.h` —— 此处直接使用的 Polly API、数据结构或 Pass 接口。
- **LLVM-family headers**: `llvm/ADT/ArrayRef.h`, `llvm/Support/ErrorHandling.h` — LLVM/Clang/MLIR infrastructure for IR, passes, utilities, or diagnostics.
  **LLVM-family headers（CN）**：`llvm/ADT/ArrayRef.h`, `llvm/Support/ErrorHandling.h` —— 用于 IR、Pass、工具或诊断的 LLVM/Clang/MLIR 基础设施。
- **ISL headers**: `isl/isl-noexceptions.h` — ISL relation/set primitives that support polyhedral analysis and transformations.
  **ISL headers（CN）**：`isl/isl-noexceptions.h` —— 支撑多面体分析与变换的 ISL 关系/集合原语。
- **System/standard headers**: `cassert` — Standard-library or system declarations required by the implementation.
  **System/standard headers（CN）**：`cassert` —— 实现所需的标准库或系统声明。
