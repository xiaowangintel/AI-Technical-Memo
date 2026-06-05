# HFSort.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/HFSort.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: Cluster functions by hotness. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：Cluster functions by hotness。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13

```cpp
//===- bolt/Passes/HFSort.cpp - Cluster functions by hotness --------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Implementation of HFSort algorithm for function ordering:
// https://research.fb.com/wp-content/uploads/2017/01/cgo2017-hfsort-final1.pdf
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 14-22

```cpp
#include "bolt/Passes/HFSort.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/Format.h"
#include "llvm/Support/raw_ostream.h"
#include <unordered_set>

#define DEBUG_TYPE "hfsort"
```

- EN: Pulls in 6 header(s) from local project, LLVM, system dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 6 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。这里的注释记录了设计假设、不变量或使用说明。

### Lines 23-33

```cpp
namespace opts {
extern llvm::cl::opt<unsigned> Verbosity;
}

namespace llvm {
namespace bolt {

using NodeId = CallGraph::NodeId;
using Arc = CallGraph::Arc;
using Node = CallGraph::Node;
```

- EN: Works inside namespace scope `opts`, `llvm`, `bolt` to organize symbols. Notable symbols here include `opts`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `opts`, `llvm`, `bolt` 中，用于组织符号作用域。这里较值得关注的符号包括 `opts`, `llvm`, `bolt`。

### Lines 34-44

```cpp
namespace {

// The number of pages to reserve for the functions with highest
// density (samples / size).  The functions put in these pages are not
// considered for clustering.
constexpr uint32_t FrozenPages = 0;

// The minimum approximate probability of a callee being called from a
// particular arc to consider merging with the caller's cluster.
constexpr double MinArcProbability = 0.1;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 45-52

```cpp
// This is a factor to determine by how much a caller cluster is
// willing to degrade it's density by merging a callee.
constexpr int CallerDegradeFactor = 8;

} // namespace

////////////////////////////////////////////////////////////////////////////////
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 53-69

```cpp
Cluster::Cluster(NodeId Id, const Node &Func)
    : Samples(Func.samples()), Size(Func.size()),
      Density((double)Samples / Size) {
  Targets.push_back(Id);
}

Cluster::Cluster(const std::vector<NodeId> &Nodes, const CallGraph &Cg) {
  Samples = 0;
  Size = 0;
  for (NodeId TargetId : Nodes) {
    Targets.push_back(TargetId);
    Samples += Cg.samples(TargetId);
    Size += Cg.size(TargetId);
  }
  Density = (double)Samples / Size;
}
```

- EN: Declares or implements routines including `Cluster`, `Samples`, `Density`. Notable symbols here include `Cluster`, `Samples`, `Density`.
- CN: 这里声明或实现函数，例如 `Cluster`, `Samples`, `Density`。这里较值得关注的符号包括 `Cluster`, `Samples`, `Density`。

### Lines 70-84

```cpp
std::string Cluster::toString() const {
  std::string Str;
  raw_string_ostream CS(Str);
  bool PrintComma = false;
  CS << "funcs = [";
  for (const NodeId &Target : Targets) {
    if (PrintComma)
      CS << ", ";
    CS << Target;
    PrintComma = true;
  }
  CS << "]";
  return CS.str();
}
```

- EN: Declares or implements routines including `toString`, `CS`. Notable symbols here include `toString`, `CS`.
- CN: 这里声明或实现函数，例如 `toString`, `CS`。这里较值得关注的符号包括 `toString`, `CS`。

### Lines 85-102

```cpp
namespace {

void freezeClusters(const CallGraph &Cg, std::vector<Cluster> &Clusters) {
  uint32_t TotalSize = 0;
  llvm::sort(Clusters, compareClustersDensity);
  for (Cluster &C : Clusters) {
    uint32_t NewSize = TotalSize + C.size();
    if (NewSize > FrozenPages * HugePageSize)
      break;
    C.freeze();
    TotalSize = NewSize;
    LLVM_DEBUG(NodeId Fid = C.target(0);
               dbgs() << format(
                   "freezing cluster for func %d, size = %u, samples = %lu)\n",
                   Fid, Cg.size(Fid), Cg.samples(Fid)););
  }
}
```

- EN: Declares or implements routines including `freezeClusters`, `sort`, `LLVM_DEBUG`, `dbgs`. Notable symbols here include `freezeClusters`, `sort`, `LLVM_DEBUG`, `dbgs`.
- CN: 这里声明或实现函数，例如 `freezeClusters`, `sort`, `LLVM_DEBUG`, `dbgs`。这里较值得关注的符号包括 `freezeClusters`, `sort`, `LLVM_DEBUG`, `dbgs`。

### Lines 103-113

```cpp
} // namespace

void Cluster::reverseTargets() { std::reverse(Targets.begin(), Targets.end()); }

void Cluster::merge(const Cluster &Other, const double Aw) {
  Targets.insert(Targets.end(), Other.Targets.begin(), Other.Targets.end());
  Size += Other.Size;
  Samples += Other.Samples;
  Density = (double)Samples / Size;
}
```

- EN: Works inside namespace scope `void` to organize symbols. Declares or implements routines including `reverseTargets`, `merge`. Notable symbols here include `reverseTargets`, `merge`, `void`.
- CN: 这里位于命名空间 `void` 中，用于组织符号作用域。这里声明或实现函数，例如 `reverseTargets`, `merge`。这里较值得关注的符号包括 `reverseTargets`, `merge`, `void`。

### Lines 114-121

```cpp
void Cluster::merge(const Cluster &Other,
                    const std::vector<CallGraph::NodeId> &Targets_) {
  Targets = Targets_;
  Size += Other.Size;
  Samples += Other.Samples;
  Density = (double)Samples / Size;
}
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 122-130

```cpp
void Cluster::clear() {
  Id = -1u;
  Size = 0;
  Samples = 0;
  Density = 0.0;
  Targets.clear();
  Frozen = false;
}
```

- EN: Declares or implements routines including `clear`. Notable symbols here include `clear`.
- CN: 这里声明或实现函数，例如 `clear`。这里较值得关注的符号包括 `clear`。

### Lines 131-138

```cpp
std::vector<Cluster> clusterize(const CallGraph &Cg) {
  std::vector<NodeId> SortedFuncs;

  // indexed by NodeId, keeps it's current cluster
  std::vector<Cluster *> FuncCluster(Cg.numNodes(), nullptr);
  std::vector<Cluster> Clusters;
  Clusters.reserve(Cg.numNodes());
```

- EN: Declares or implements routines including `clusterize`, `FuncCluster`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `clusterize`, `FuncCluster`.
- CN: 这里声明或实现函数，例如 `clusterize`, `FuncCluster`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `clusterize`, `FuncCluster`。

### Lines 139-147

```cpp
  for (NodeId F = 0; F < Cg.numNodes(); F++) {
    if (Cg.samples(F) == 0)
      continue;
    Clusters.emplace_back(F, Cg.getNode(F));
    SortedFuncs.push_back(F);
  }

  freezeClusters(Cg, Clusters);
```

- EN: Declares or implements routines including `freezeClusters`. Notable symbols here include `freezeClusters`.
- CN: 这里声明或实现函数，例如 `freezeClusters`。这里较值得关注的符号包括 `freezeClusters`。

### Lines 148-159

```cpp
  // The size and order of Clusters is fixed until we reshuffle it immediately
  // before returning.
  for (Cluster &Cluster : Clusters)
    FuncCluster[Cluster.targets().front()] = &Cluster;

  llvm::sort(SortedFuncs, [&](const NodeId F1, const NodeId F2) {
    const CallGraph::Node &Func1 = Cg.getNode(F1);
    const CallGraph::Node &Func2 = Cg.getNode(F2);
    return Func1.samples() * Func2.size() > // TODO: is this correct?
           Func2.samples() * Func1.size();
  });
```

- EN: Declares or implements routines including `sort`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `sort`.
- CN: 这里声明或实现函数，例如 `sort`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `sort`。

### Lines 160-170

```cpp
  // Process each function, and consider merging its cluster with the
  // one containing its most likely predecessor.
  for (const NodeId Fid : SortedFuncs) {
    Cluster *Cluster = FuncCluster[Fid];
    if (Cluster->frozen())
      continue;

    // Find best predecessor.
    NodeId BestPred = CallGraph::InvalidId;
    double BestProb = 0;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 171-179

```cpp
    for (const NodeId Src : Cg.predecessors(Fid)) {
      const Arc &Arc = *Cg.findArc(Src, Fid);
      if (BestPred == CallGraph::InvalidId ||
          Arc.normalizedWeight() > BestProb) {
        BestPred = Arc.src();
        BestProb = Arc.normalizedWeight();
      }
    }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 180-187

```cpp
    // Check if the merge is good for the callee.
    //   Don't merge if the probability of getting to the callee from the
    //   caller is too low.
    if (BestProb < MinArcProbability)
      continue;

    assert(BestPred != CallGraph::InvalidId);
```

- EN: Declares or implements routines including `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`。

### Lines 188-195

```cpp
    class Cluster *PredCluster = FuncCluster[BestPred];

    // Skip if no predCluster (predecessor w/ no samples), or if same
    // as cluster, of it's frozen.
    if (PredCluster == nullptr || PredCluster == Cluster ||
        PredCluster->frozen())
      continue;
```

- EN: Introduces type definitions such as `Cluster`. Declares or implements routines including `frozen`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `Cluster`, `frozen`.
- CN: 这里引入类型定义，例如 `Cluster`。这里声明或实现函数，例如 `frozen`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `Cluster`, `frozen`。

### Lines 196-209

```cpp
    // Skip if merged cluster would be bigger than the threshold.
    if (Cluster->size() + PredCluster->size() > MaxClusterSize)
      continue;

    // Check if the merge is good for the caller.
    //   Don't merge if the caller's density is significantly better
    //   than the density resulting from the merge.
    const double NewDensity =
        ((double)PredCluster->samples() + Cluster->samples()) /
        (PredCluster->size() + Cluster->size());
    if (PredCluster->density() > NewDensity * CallerDegradeFactor) {
      continue;
    }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 210-218

```cpp
    LLVM_DEBUG(if (opts::Verbosity > 1) {
      dbgs() << format("merging %s -> %s: %u\n",
                       PredCluster->toString().c_str(),
                       Cluster->toString().c_str(), Cg.samples(Fid));
    });

    for (NodeId F : Cluster->targets())
      FuncCluster[F] = PredCluster;
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `dbgs`, `toString`. Notable symbols here include `LLVM_DEBUG`, `dbgs`, `toString`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `dbgs`, `toString`。这里较值得关注的符号包括 `LLVM_DEBUG`, `dbgs`, `toString`。

### Lines 219-231

```cpp
    PredCluster->merge(*Cluster);
    Cluster->clear();
  }

  // Return the set of Clusters that are left, which are the ones that
  // didn't get merged (so their first func is its original func).
  std::vector<Cluster> SortedClusters;
  std::unordered_set<Cluster *> Visited;
  for (const NodeId Func : SortedFuncs) {
    Cluster *Cluster = FuncCluster[Func];
    if (!Cluster || Visited.count(Cluster) == 1 || Cluster->target(0) != Func)
      continue;
```

- EN: Declares or implements routines including `merge`, `clear`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `merge`, `clear`.
- CN: 这里声明或实现函数，例如 `merge`, `clear`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `merge`, `clear`。

### Lines 232-240

```cpp
    SortedClusters.emplace_back(std::move(*Cluster));
    Visited.insert(Cluster);
  }

  llvm::sort(SortedClusters, compareClustersDensity);

  return SortedClusters;
}
```

- EN: Declares or implements routines including `sort`. Notable symbols here include `sort`.
- CN: 这里声明或实现函数，例如 `sort`。这里较值得关注的符号包括 `sort`。

### Lines 241-250

```cpp
std::vector<Cluster> randomClusters(const CallGraph &Cg) {
  std::vector<Cluster> Clusters;
  Clusters.reserve(Cg.numNodes());

  for (NodeId F = 0; F < Cg.numNodes(); F++) {
    if (Cg.samples(F) == 0)
      continue;
    Clusters.emplace_back(F, Cg.getNode(F));
  }
```

- EN: Declares or implements routines including `randomClusters`. Notable symbols here include `randomClusters`.
- CN: 这里声明或实现函数，例如 `randomClusters`。这里较值得关注的符号包括 `randomClusters`。

### Lines 251-261

```cpp
  llvm::sort(Clusters, [](const Cluster &A, const Cluster &B) {
    return A.size() < B.size();
  });

  auto pickMergeCluster = [&Clusters](const size_t Idx) {
    size_t MaxIdx = Idx + 1;

    while (MaxIdx < Clusters.size() &&
           Clusters[Idx].size() + Clusters[MaxIdx].size() <= MaxClusterSize)
      ++MaxIdx;
```

- EN: Declares or implements routines including `sort`. Notable symbols here include `sort`.
- CN: 这里声明或实现函数，例如 `sort`。这里较值得关注的符号包括 `sort`。

### Lines 262-270

```cpp
    if (MaxIdx - Idx > 1) {
      size_t MergeIdx = (std::rand() % (MaxIdx - Idx - 1)) + Idx + 1;
      assert(Clusters[MergeIdx].size() + Clusters[Idx].size() <=
             MaxClusterSize);
      return MergeIdx;
    }
    return Clusters.size();
  };
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 271-281

```cpp
  size_t Idx = 0;
  while (Idx < Clusters.size()) {
    size_t MergeIdx = pickMergeCluster(Idx);
    if (MergeIdx == Clusters.size()) {
      ++Idx;
    } else {
      Clusters[Idx].merge(Clusters[MergeIdx]);
      Clusters.erase(Clusters.begin() + MergeIdx);
    }
  }
```

- EN: Declares or implements routines including `pickMergeCluster`. Notable symbols here include `pickMergeCluster`.
- CN: 这里声明或实现函数，例如 `pickMergeCluster`。这里较值得关注的符号包括 `pickMergeCluster`。

### Lines 282-286

```cpp
  return Clusters;
}

} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Contains local control flow that updates state or selects among execution branches. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `Cluster`: class or struct interface / 类或结构体接口
- `Cluster`: function or method entry point / 函数或方法入口
- `Samples`: function or method entry point / 函数或方法入口
- `Density`: function or method entry point / 函数或方法入口
- `toString`: function or method entry point / 函数或方法入口
- `CS`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `opts`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/HFSort.h`
- LLVM headers / LLVM 头文件: `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/Format.h`, `llvm/Support/raw_ostream.h`
- System headers / 系统头文件: `unordered_set`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
