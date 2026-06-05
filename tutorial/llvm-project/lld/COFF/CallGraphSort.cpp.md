# CallGraphSort.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `lld/COFF/CallGraphSort.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: CallGraphSort.cpp This is based on the ELF port, see ELF/CallGraphSort.cpp for the details about the algorithm.. It also sits in code that implements COFF/PE linking components in LLD. / 该文件实现 LLD 中的 COFF/PE 链接组件。 源码头部说明其职责是：CallGraphSort.cpp This is based on the ELF port, see ELF/CallGraphSort.cpp for the details about the algorithm.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13

```cpp
//===- CallGraphSort.cpp --------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// This is based on the ELF port, see ELF/CallGraphSort.cpp for the details
/// about the algorithm.
///
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 14-21

```cpp
#include "CallGraphSort.h"
#include "COFFLinkerContext.h"
#include "InputFiles.h"
#include "SymbolTable.h"
#include "Symbols.h"

#include <numeric>
```

- EN: Pulls in 6 header(s) from system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 6 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 22-31

```cpp
using namespace llvm;
using namespace lld;
using namespace lld::coff;

namespace {
struct Edge {
  int from;
  uint64_t weight;
};
```

- EN: Works inside namespace scope `llvm`, `lld` to organize symbols. Introduces type definitions such as `Edge`. Notable symbols here include `Edge`, `llvm`, `lld`.
- CN: 这里位于命名空间 `llvm`, `lld` 中，用于组织符号作用域。这里引入类型定义，例如 `Edge`。这里较值得关注的符号包括 `Edge`, `llvm`, `lld`。

### Lines 32-40

```cpp
struct Cluster {
  Cluster(int sec, size_t s) : next(sec), prev(sec), size(s) {}

  double getDensity() const {
    if (size == 0)
      return 0;
    return double(weight) / double(size);
  }
```

- EN: Introduces type definitions such as `Cluster`. Declares or implements routines including `Cluster`, `getDensity`. Notable symbols here include `Cluster`, `getDensity`.
- CN: 这里引入类型定义，例如 `Cluster`。这里声明或实现函数，例如 `Cluster`, `getDensity`。这里较值得关注的符号包括 `Cluster`, `getDensity`。

### Lines 41-48

```cpp
  int next;
  int prev;
  uint64_t size;
  uint64_t weight = 0;
  uint64_t initialWeight = 0;
  Edge bestPred = {-1, 0};
};
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 49-58

```cpp
class CallGraphSort {
public:
  CallGraphSort(COFFLinkerContext &ctx);

  DenseMap<const SectionChunk *, int> run();

private:
  std::vector<Cluster> clusters;
  std::vector<const SectionChunk *> sections;
```

- EN: Introduces type definitions such as `CallGraphSort`. Declares or implements routines including `CallGraphSort`, `run`. Notable symbols here include `CallGraphSort`, `run`.
- CN: 这里引入类型定义，例如 `CallGraphSort`。这里声明或实现函数，例如 `CallGraphSort`, `run`。这里较值得关注的符号包括 `CallGraphSort`, `run`。

### Lines 59-69

```cpp
  COFFLinkerContext &ctx;
};

// Maximum amount the combined cluster density can be worse than the original
// cluster to consider merging.
constexpr int MAX_DENSITY_DEGRADATION = 8;

// Maximum cluster size in bytes.
constexpr uint64_t MAX_CLUSTER_SIZE = 1024 * 1024;
} // end anonymous namespace
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 70-78

```cpp
using SectionPair = std::pair<const SectionChunk *, const SectionChunk *>;

// Take the edge list in Config->CallGraphProfile, resolve symbol names to
// Symbols, and generate a graph between InputSections with the provided
// weights.
CallGraphSort::CallGraphSort(COFFLinkerContext &ctx) : ctx(ctx) {
  const MapVector<SectionPair, uint64_t> &profile = ctx.config.callGraphProfile;
  DenseMap<const SectionChunk *, int> secToCluster;
```

- EN: Declares or implements routines including `CallGraphSort`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `CallGraphSort`.
- CN: 这里声明或实现函数，例如 `CallGraphSort`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `CallGraphSort`。

### Lines 79-87

```cpp
  auto getOrCreateNode = [&](const SectionChunk *isec) -> int {
    auto res = secToCluster.try_emplace(isec, clusters.size());
    if (res.second) {
      sections.push_back(isec);
      clusters.emplace_back(clusters.size(), isec->getSize());
    }
    return res.first->second;
  };
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 88-102

```cpp
  // Create the graph.
  for (const std::pair<SectionPair, uint64_t> &c : profile) {
    const auto *fromSec = cast<SectionChunk>(c.first.first->repl);
    const auto *toSec = cast<SectionChunk>(c.first.second->repl);
    uint64_t weight = c.second;

    // Ignore edges between input sections belonging to different output
    // sections.  This is done because otherwise we would end up with clusters
    // containing input sections that can't actually be placed adjacently in the
    // output.  This messes with the cluster size and density calculations.  We
    // would also end up moving input sections in other output sections without
    // moving them closer to what calls them.
    if (ctx.getOutputSection(fromSec) != ctx.getOutputSection(toSec))
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 103-110

```cpp
    int from = getOrCreateNode(fromSec);
    int to = getOrCreateNode(toSec);

    clusters[to].weight += weight;

    if (from == to)
      continue;
```

- EN: Declares or implements routines including `getOrCreateNode`. Notable symbols here include `getOrCreateNode`.
- CN: 这里声明或实现函数，例如 `getOrCreateNode`。这里较值得关注的符号包括 `getOrCreateNode`。

### Lines 111-121

```cpp
    // Remember the best edge.
    Cluster &toC = clusters[to];
    if (toC.bestPred.from == -1 || toC.bestPred.weight < weight) {
      toC.bestPred.from = from;
      toC.bestPred.weight = weight;
    }
  }
  for (Cluster &c : clusters)
    c.initialWeight = c.weight;
}
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 122-138

```cpp
// It's bad to merge clusters which would degrade the density too much.
static bool isNewDensityBad(Cluster &a, Cluster &b) {
  double newDensity = double(a.weight + b.weight) / double(a.size + b.size);
  return newDensity < a.getDensity() / MAX_DENSITY_DEGRADATION;
}

// Find the leader of V's belonged cluster (represented as an equivalence
// class). We apply union-find path-halving technique (simple to implement) in
// the meantime as it decreases depths and the time complexity.
static int getLeader(std::vector<int> &leaders, int v) {
  while (leaders[v] != v) {
    leaders[v] = leaders[leaders[v]];
    v = leaders[v];
  }
  return v;
}
```

- EN: Declares or implements routines including `isNewDensityBad`, `double`, `getLeader`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isNewDensityBad`, `double`, `getLeader`.
- CN: 这里声明或实现函数，例如 `isNewDensityBad`, `double`, `getLeader`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isNewDensityBad`, `double`, `getLeader`。

### Lines 139-151

```cpp
static void mergeClusters(std::vector<Cluster> &cs, Cluster &into, int intoIdx,
                          Cluster &from, int fromIdx) {
  int tail1 = into.prev, tail2 = from.prev;
  into.prev = tail2;
  cs[tail2].next = intoIdx;
  from.prev = tail1;
  cs[tail1].next = fromIdx;
  into.size += from.size;
  into.weight += from.weight;
  from.size = 0;
  from.weight = 0;
}
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 152-163

```cpp
// Group InputSections into clusters using the Call-Chain Clustering heuristic
// then sort the clusters by density.
DenseMap<const SectionChunk *, int> CallGraphSort::run() {
  std::vector<int> sorted(clusters.size());
  std::vector<int> leaders(clusters.size());

  std::iota(leaders.begin(), leaders.end(), 0);
  std::iota(sorted.begin(), sorted.end(), 0);
  llvm::stable_sort(sorted, [&](int a, int b) {
    return clusters[a].getDensity() > clusters[b].getDensity();
  });
```

- EN: Declares or implements routines including `run`, `sorted`, `leaders`, `iota`, `stable_sort`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `run`, `sorted`, `leaders`, `iota`, `stable_sort`.
- CN: 这里声明或实现函数，例如 `run`, `sorted`, `leaders`, `iota`, `stable_sort`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `run`, `sorted`, `leaders`, `iota`, `stable_sort`。

### Lines 164-172

```cpp
  for (int l : sorted) {
    // The cluster index is the same as the index of its leader here because
    // clusters[L] has not been merged into another cluster yet.
    Cluster &c = clusters[l];

    // Don't consider merging if the edge is unlikely.
    if (c.bestPred.from == -1 || c.bestPred.weight * 10 <= c.initialWeight)
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 173-180

```cpp
    int predL = getLeader(leaders, c.bestPred.from);
    if (l == predL)
      continue;

    Cluster *predC = &clusters[predL];
    if (c.size + predC->size > MAX_CLUSTER_SIZE)
      continue;
```

- EN: Declares or implements routines including `getLeader`. Notable symbols here include `getLeader`.
- CN: 这里声明或实现函数，例如 `getLeader`。这里较值得关注的符号包括 `getLeader`。

### Lines 181-196

```cpp
    if (isNewDensityBad(*predC, c))
      continue;

    leaders[l] = predL;
    mergeClusters(clusters, *predC, predL, c, l);
  }

  // Sort remaining non-empty clusters by density.
  sorted.clear();
  for (int i = 0, e = (int)clusters.size(); i != e; ++i)
    if (clusters[i].size > 0)
      sorted.push_back(i);
  llvm::stable_sort(sorted, [&](int a, int b) {
    return clusters[a].getDensity() > clusters[b].getDensity();
  });
```

- EN: Declares or implements routines including `mergeClusters`, `stable_sort`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `mergeClusters`, `stable_sort`.
- CN: 这里声明或实现函数，例如 `mergeClusters`, `stable_sort`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `mergeClusters`, `stable_sort`。

### Lines 197-214

```cpp
  DenseMap<const SectionChunk *, int> orderMap;
  // Sections will be sorted by increasing order. Absent sections will have
  // priority 0 and be placed at the end of sections.
  int curOrder = INT_MIN;
  for (int leader : sorted) {
    for (int i = leader;;) {
      orderMap[sections[i]] = curOrder++;
      i = clusters[i].next;
      if (i == leader)
        break;
    }
  }
  if (!ctx.config.printSymbolOrder.empty()) {
    std::error_code ec;
    raw_fd_ostream os(ctx.config.printSymbolOrder, ec, sys::fs::OF_None);
    if (ec) {
      Err(ctx) << "cannot open " << ctx.config.printSymbolOrder << ": "
               << ec.message();
```

- EN: Declares or implements routines including `os`, `Err`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `os`, `Err`.
- CN: 这里声明或实现函数，例如 `os`, `Err`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `os`, `Err`。

### Lines 215-222

```cpp
      return orderMap;
    }
    // Print the symbols ordered by C3, in the order of increasing curOrder
    // Instead of sorting all the orderMap, just repeat the loops above.
    for (int leader : sorted)
      for (int i = leader;;) {
        const SectionChunk *sc = sections[i];
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 223-237

```cpp
        // Search all the symbols in the file of the section
        // and find out a DefinedCOFF symbol with name that is within the
        // section.
        for (Symbol *sym : sc->file->getSymbols())
          if (auto *d = dyn_cast_or_null<DefinedCOFF>(sym))
            // Filter out non-COMDAT symbols and section symbols.
            if (d->isCOMDAT && !d->getCOFFSymbol().isSection() &&
                sc == d->getChunk())
              os << sym->getName() << "\n";
        i = clusters[i].next;
        if (i == leader)
          break;
      }
  }
```

- EN: Declares or implements routines including `getChunk`, `getName`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getChunk`, `getName`.
- CN: 这里声明或实现函数，例如 `getChunk`, `getName`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getChunk`, `getName`。

### Lines 238-249

```cpp
  return orderMap;
}

// Sort sections by the profile data provided by  /call-graph-ordering-file
//
// This first builds a call graph based on the profile data then merges sections
// according to the C³ heuristic. All clusters are then sorted by a density
// metric to further improve locality.
DenseMap<const SectionChunk *, int>
coff::computeCallGraphProfileOrder(COFFLinkerContext &ctx) {
  return CallGraphSort(ctx).run();
}
```

- EN: Declares or implements routines including `computeCallGraphProfileOrder`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `computeCallGraphProfileOrder`.
- CN: 这里声明或实现函数，例如 `computeCallGraphProfileOrder`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `computeCallGraphProfileOrder`。

## Key Concepts / 关键概念

- `Edge`: class or struct interface / 类或结构体接口
- `Cluster`: class or struct interface / 类或结构体接口
- `CallGraphSort`: class or struct interface / 类或结构体接口
- `Cluster`: function or method entry point / 函数或方法入口
- `getDensity`: function or method entry point / 函数或方法入口
- `CallGraphSort`: function or method entry point / 函数或方法入口
- `run`: function or method entry point / 函数或方法入口
- `getOrCreateNode`: function or method entry point / 函数或方法入口

## Dependencies / 依赖关系

- System headers / 系统头文件: `CallGraphSort.h`, `COFFLinkerContext.h`, `InputFiles.h`, `SymbolTable.h`, `Symbols.h`, `numeric`
- Directory context / 目录上下文: `lld/COFF` neighbors usually cooperate with this file to provide the surrounding subsystem / `lld/COFF` 下的相邻文件通常与本文件协作组成对应子系统
