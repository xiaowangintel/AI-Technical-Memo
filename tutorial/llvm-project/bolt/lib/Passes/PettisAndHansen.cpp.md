# PettisAndHansen.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/PettisAndHansen.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Passes/PettisAndHansen.cpp The file implements Pettis and Hansen code-layout algorithm.. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：bolt/Passes/PettisAndHansen.cpp The file implements Pettis and Hansen code-layout algorithm.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Passes/PettisAndHansen.cpp ------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// The file implements Pettis and Hansen code-layout algorithm.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-21

```cpp
#include "bolt/Passes/HFSort.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/Format.h"
#include "llvm/Support/raw_ostream.h"
#include <set>
#include <unordered_map>

#define DEBUG_TYPE "hfsort"
```

- EN: Pulls in 6 header(s) from local project, LLVM, system dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 6 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。这里的注释记录了设计假设、不变量或使用说明。

### Lines 22-34

```cpp
namespace llvm {
namespace bolt {

using NodeId = CallGraph::NodeId;
using Arc = CallGraph::Arc;
using Node = CallGraph::Node;

namespace {
class ClusterArc {
public:
  ClusterArc(Cluster *Ca, Cluster *Cb, double W = 0)
      : C1(std::min(Ca, Cb)), C2(std::max(Ca, Cb)), Weight(W) {}
```

- EN: Works inside namespace scope `llvm`, `bolt` to organize symbols. Introduces type definitions such as `ClusterArc`. Declares or implements routines including `ClusterArc`, `C1`. Notable symbols here include `ClusterArc`, `C1`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里引入类型定义，例如 `ClusterArc`。这里声明或实现函数，例如 `ClusterArc`, `C1`。这里较值得关注的符号包括 `ClusterArc`, `C1`, `llvm`, `bolt`。

### Lines 35-43

```cpp
  friend bool operator==(const ClusterArc &Lhs, const ClusterArc &Rhs) {
    return Lhs.C1 == Rhs.C1 && Lhs.C2 == Rhs.C2;
  }

  Cluster *const C1;
  Cluster *const C2;
  mutable double Weight;
};
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 44-51

```cpp
class ClusterArcHash {
public:
  int64_t operator()(const ClusterArc &Arc) const {
    std::hash<int64_t> Hasher;
    return hashCombine(Hasher(int64_t(Arc.C1)), int64_t(Arc.C2));
  }
};
```

- EN: Introduces type definitions such as `ClusterArcHash`. Declares or implements routines including `operator`. Notable symbols here include `ClusterArcHash`, `operator`.
- CN: 这里引入类型定义，例如 `ClusterArcHash`。这里声明或实现函数，例如 `operator`。这里较值得关注的符号包括 `ClusterArcHash`, `operator`。

### Lines 52-59

```cpp
using ClusterArcSet = std::unordered_set<ClusterArc, ClusterArcHash>;

void orderFuncs(const CallGraph &Cg, Cluster *C1, Cluster *C2) {
  NodeId C1head = C1->targets().front();
  NodeId C1tail = C1->targets().back();
  NodeId C2head = C2->targets().front();
  NodeId C2tail = C2->targets().back();
```

- EN: Declares or implements routines including `orderFuncs`, `targets`. Notable symbols here include `orderFuncs`, `targets`.
- CN: 这里声明或实现函数，例如 `orderFuncs`, `targets`。这里较值得关注的符号包括 `orderFuncs`, `targets`。

### Lines 60-77

```cpp
  double C1headC2head = 0;
  double C1headC2tail = 0;
  double C1tailC2head = 0;
  double C1tailC2tail = 0;

  for (const Arc &Arc : Cg.arcs()) {
    if ((Arc.src() == C1head && Arc.dst() == C2head) ||
        (Arc.dst() == C1head && Arc.src() == C2head))
      C1headC2head += Arc.weight();
    else if ((Arc.src() == C1head && Arc.dst() == C2tail) ||
             (Arc.dst() == C1head && Arc.src() == C2tail))
      C1headC2tail += Arc.weight();
    else if ((Arc.src() == C1tail && Arc.dst() == C2head) ||
             (Arc.dst() == C1tail && Arc.src() == C2head))
      C1tailC2head += Arc.weight();
    else if ((Arc.src() == C1tail && Arc.dst() == C2tail) ||
             (Arc.dst() == C1tail && Arc.src() == C2tail))
      C1tailC2tail += Arc.weight();
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 78-95

```cpp
  }

  const double Max = std::max(std::max(C1headC2head, C1headC2tail),
                              std::max(C1tailC2head, C1tailC2tail));

  if (C1headC2head == Max) {
    // flip C1
    C1->reverseTargets();
  } else if (C1headC2tail == Max) {
    // flip C1 C2
    C1->reverseTargets();
    C2->reverseTargets();
  } else if (C1tailC2tail == Max) {
    // flip C2
    C2->reverseTargets();
  }
}
} // namespace
```

- EN: Declares or implements routines including `max`, `reverseTargets`, `if`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `max`, `reverseTargets`, `if`.
- CN: 这里声明或实现函数，例如 `max`, `reverseTargets`, `if`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `max`, `reverseTargets`, `if`。

### Lines 96-104

```cpp

std::vector<Cluster> pettisAndHansen(const CallGraph &Cg) {
  // indexed by NodeId, keeps its current cluster
  std::vector<Cluster *> FuncCluster(Cg.numNodes(), nullptr);
  std::vector<Cluster> Clusters;
  std::vector<NodeId> Funcs;

  Clusters.reserve(Cg.numNodes());
```

- EN: Declares or implements routines including `pettisAndHansen`, `FuncCluster`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `pettisAndHansen`, `FuncCluster`.
- CN: 这里声明或实现函数，例如 `pettisAndHansen`, `FuncCluster`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `pettisAndHansen`, `FuncCluster`。

### Lines 105-112

```cpp
  for (NodeId F = 0; F < Cg.numNodes(); F++) {
    if (Cg.samples(F) == 0)
      continue;
    Clusters.emplace_back(F, Cg.getNode(F));
    FuncCluster[F] = &Clusters.back();
    Funcs.push_back(F);
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 113-120

```cpp
  ClusterArcSet Carcs;

  auto insertOrInc = [&](Cluster *C1, Cluster *C2, double Weight) {
    auto Res = Carcs.emplace(C1, C2, Weight);
    if (!Res.second)
      Res.first->Weight += Weight;
  };
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 121-129

```cpp
  // Create a std::vector of cluster arcs

  for (const Arc &Arc : Cg.arcs()) {
    if (Arc.weight() == 0)
      continue;

    Cluster *const S = FuncCluster[Arc.src()];
    Cluster *const D = FuncCluster[Arc.dst()];
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 130-139

```cpp
    // ignore if s or d is nullptr

    if (S == nullptr || D == nullptr)
      continue;

    // ignore self-edges

    if (S == D)
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 140-150

```cpp
    insertOrInc(S, D, Arc.weight());
  }

  // Find an arc with max weight and merge its nodes

  while (!Carcs.empty()) {
    auto Maxpos = llvm::max_element(
        Carcs, [&](const ClusterArc &Carc1, const ClusterArc &Carc2) {
          return Carc1.Weight < Carc2.Weight;
        });
```

- EN: Declares or implements routines including `insertOrInc`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `insertOrInc`.
- CN: 这里声明或实现函数，例如 `insertOrInc`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `insertOrInc`。

### Lines 151-159

```cpp
    ClusterArc Max = *Maxpos;
    Carcs.erase(Maxpos);

    Cluster *const C1 = Max.C1;
    Cluster *const C2 = Max.C2;

    if (C1->size() + C2->size() > MaxClusterSize)
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 160-170

```cpp
    if (C1->frozen() || C2->frozen())
      continue;

    // order functions and merge cluster

    orderFuncs(Cg, C1, C2);

    LLVM_DEBUG(dbgs() << format("merging %s -> %s: %.1f\n",
                                C2->toString().c_str(), C1->toString().c_str(),
                                Max.Weight));
```

- EN: Declares or implements routines including `orderFuncs`, `LLVM_DEBUG`, `toString`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `orderFuncs`, `LLVM_DEBUG`, `toString`.
- CN: 这里声明或实现函数，例如 `orderFuncs`, `LLVM_DEBUG`, `toString`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `orderFuncs`, `LLVM_DEBUG`, `toString`。

### Lines 171-180

```cpp
    // update carcs: merge C1arcs to C2arcs

    std::unordered_map<ClusterArc, Cluster *, ClusterArcHash> C2arcs;
    for (const ClusterArc &Carc : Carcs) {
      if (Carc.C1 == C2)
        C2arcs.emplace(Carc, Carc.C2);
      if (Carc.C2 == C2)
        C2arcs.emplace(Carc, Carc.C1);
    }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 181-188

```cpp
    for (auto It : C2arcs) {
      Cluster *const C = It.second;
      ClusterArc const C2arc = It.first;

      insertOrInc(C, C1, C2arc.Weight);
      Carcs.erase(C2arc);
    }
```

- EN: Declares or implements routines including `insertOrInc`. Notable symbols here include `insertOrInc`.
- CN: 这里声明或实现函数，例如 `insertOrInc`。这里较值得关注的符号包括 `insertOrInc`。

### Lines 189-197

```cpp
    // update FuncCluster

    for (NodeId F : C2->targets())
      FuncCluster[F] = C1;

    C1->merge(*C2, Max.Weight);
    C2->clear();
  }
```

- EN: Declares or implements routines including `merge`, `clear`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `merge`, `clear`.
- CN: 这里声明或实现函数，例如 `merge`, `clear`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `merge`, `clear`。

### Lines 198-208

```cpp
  // Return the set of Clusters that are left, which are the ones that
  // didn't get merged.

  std::set<Cluster *> LiveClusters;
  std::vector<Cluster> OutClusters;

  for (NodeId Fid : Funcs)
    LiveClusters.insert(FuncCluster[Fid]);
  for (Cluster *C : LiveClusters)
    OutClusters.push_back(std::move(*C));
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 209-215

```cpp
  llvm::sort(OutClusters, compareClustersDensity);

  return OutClusters;
}

} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Declares or implements routines including `sort`. Notable symbols here include `sort`, `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里声明或实现函数，例如 `sort`。这里较值得关注的符号包括 `sort`, `bolt`, `llvm`。

## Key Concepts / 关键概念

- `ClusterArc`: class or struct interface / 类或结构体接口
- `ClusterArcHash`: class or struct interface / 类或结构体接口
- `ClusterArc`: function or method entry point / 函数或方法入口
- `C1`: function or method entry point / 函数或方法入口
- `operator`: function or method entry point / 函数或方法入口
- `orderFuncs`: function or method entry point / 函数或方法入口
- `targets`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/HFSort.h`
- LLVM headers / LLVM 头文件: `llvm/Support/Debug.h`, `llvm/Support/Format.h`, `llvm/Support/raw_ostream.h`
- System headers / 系统头文件: `set`, `unordered_map`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
