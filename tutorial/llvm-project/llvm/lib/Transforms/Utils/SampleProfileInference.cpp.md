# SampleProfileInference.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/SampleProfileInference.cpp` | `llvm/lib/Transforms/Utils/SampleProfileInference.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements adjust sample profiles in the IR within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 SampleProfileInference 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-89

```cpp
//===- SampleProfileInference.cpp - Adjust sample profiles in the IR ------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements a profile inference algorithm. Given an incomplete and
// possibly imprecise block counts, the algorithm reconstructs realistic block
// and edge counts that satisfy flow conservation rules, while minimally modify
// input block counts.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/SampleProfileInference.h"
#include "llvm/ADT/BitVector.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include <queue>
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
/// approach. The algorithm applies a number of augmentation iterations in which
/// flow is sent along paths of positive capacity from the source to the sink.
/// The worst-case time complexity of the implementation is O(v(f)*m*n), where
/// where m is the number of edges, n is the number of vertices, and v(f) is the
/// value of the maximum flow. However, the observed running time on typical
/// instances is sub-quadratic, that is, o(n^2).
///
/// The input is a set of edges with specified costs and capacities, and a pair
/// of nodes (source and sink). The output is the flow along each edge of the
/// minimum total cost respecting the given edge capacities.
class MinCostMaxFlow {
public:
  MinCostMaxFlow(const ProfiParams &Params) : Params(Params) {}

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 90-181

```cpp
  // Initialize algorithm's data structures for a network of a given size.
  void initialize(uint64_t NodeCount, uint64_t SourceNode, uint64_t SinkNode) {
    Source = SourceNode;
    Target = SinkNode;

    Nodes = std::vector<Node>(NodeCount);
    Edges = std::vector<std::vector<Edge>>(NodeCount, std::vector<Edge>());
    if (Params.EvenFlowDistribution)
      AugmentingEdges =
          std::vector<std::vector<Edge *>>(NodeCount, std::vector<Edge *>());
  }

  // Run the algorithm.
  int64_t run() {
    LLVM_DEBUG(dbgs() << "Starting profi for " << Nodes.size() << " nodes\n");

    // Iteratively find an augmentation path/dag in the network and send the
    // flow along its edges
    size_t AugmentationIters = applyFlowAugmentation();

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    return Flow;
  }

  /// Get the total flow between a pair of nodes.
  int64_t getFlow(uint64_t Src, uint64_t Dst) const {
    int64_t Flow = 0;
    for (const auto &Edge : Edges[Src]) {
      if (Edge.Dst == Dst) {
        Flow += Edge.Flow;
      }
    }
    return Flow;
  }

```
- EN: Core entities appearing here include initialize, run, addEdge, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 initialize, run, addEdge，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 182-268

```cpp
private:
  /// Iteratively find an augmentation path/dag in the network and send the
  /// flow along its edges. The method returns the number of applied iterations.
  size_t applyFlowAugmentation() {
    size_t AugmentationIters = 0;
    while (findAugmentingPath()) {
      uint64_t PathCapacity = computeAugmentingPathCapacity();
      while (PathCapacity > 0) {
        bool Progress = false;
        if (Params.EvenFlowDistribution) {
          // Identify node/edge candidates for augmentation
          identifyShortestEdges(PathCapacity);

          // Find an augmenting DAG
          auto AugmentingOrder = findAugmentingDAG();

          // Apply the DAG augmentation
          Progress = augmentFlowAlongDAG(AugmentingOrder);
          PathCapacity = computeAugmentingPathCapacity();
        }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      // allows to prune the search-space of the path-finding algorithm using
      // the following early-stop criteria:
      // -- If we find a path with zero-distance from Source to Target, stop the
      //    search, as the path is the shortest since Dist[Source, Target] >= 0;
      // -- If we have Dist[Source, V] > Dist[Source, Target], then do not
      //    process node V, as it is guaranteed _not_ to be on a shortest path
      //    from Source to Target; it follows from inequalities
      //    Dist[Source, Target] >= Dist[Source, V] + Dist[V, Target]
      //                         >= Dist[Source, V]
      if (!Params.EvenFlowDistribution && Nodes[Target].Distance == 0)
        break;
      if (Nodes[Src].Distance > Nodes[Target].Distance)
        continue;

```
- EN: Core entities appearing here include applyFlowAugmentation, computeAugmentingPathCapacity, findAugmentingPath, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 applyFlowAugmentation, computeAugmentingPathCapacity, findAugmentingPath，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 269-350

```cpp
      // Process adjacent edges
      for (uint64_t EdgeIdx = 0; EdgeIdx < Edges[Src].size(); EdgeIdx++) {
        auto &Edge = Edges[Src][EdgeIdx];
        if (Edge.Flow < Edge.Capacity) {
          uint64_t Dst = Edge.Dst;
          int64_t NewDistance = Nodes[Src].Distance + Edge.Cost;
          if (Nodes[Dst].Distance > NewDistance) {
            // Update the distance and the parent node/edge
            Nodes[Dst].Distance = NewDistance;
            Nodes[Dst].ParentNode = Src;
            Nodes[Dst].ParentEdgeIndex = EdgeIdx;
            // Add the node to the queue, if it is not there yet
            if (!Nodes[Dst].Taken) {
              Queue.push(Dst);
              Nodes[Dst].Taken = true;
            }
          }
        }
      }
    }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

    // Phase 1: Start DFS traversal from Source
    Stack.emplace(Source, 0);
    Nodes[Source].Discovery = ++Time;
    while (!Stack.empty()) {
      auto NodeIdx = Stack.top().first;
      auto EdgeIdx = Stack.top().second;

      // If we haven't scanned all edges out of NodeIdx, continue scanning
      if (EdgeIdx < Edges[NodeIdx].size()) {
        auto &Edge = Edges[NodeIdx][EdgeIdx];
        auto &Dst = Nodes[Edge.Dst];
        Stack.top().second++;

```
- EN: Core entities appearing here include augmentFlowAlongPath, findAugmentingDAG, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 augmentFlowAlongPath, findAugmentingDAG，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 351-435

```cpp
        if (Edge.OnShortestPath) {
          // If we haven't seen Edge.Dst so far, continue DFS search there
          if (Dst.Discovery == 0 && Dst.NumCalls < MaxDfsCalls) {
            Dst.Discovery = ++Time;
            Stack.emplace(Edge.Dst, 0);
            Dst.NumCalls++;
          } else if (Dst.Taken && Dst.Finish != 0) {
            // Else, if Edge.Dst already have a path to Target, so that NodeIdx
            Nodes[NodeIdx].Taken = true;
          }
        }
      } else {
        // If we are done scanning all edge out of NodeIdx
        Stack.pop();
        // If we haven't found a path from NodeIdx to Target, forget about it
        if (!Nodes[NodeIdx].Taken) {
          Nodes[NodeIdx].Discovery = 0;
        } else {
          // If we have found a path from NodeIdx to Target, then finish NodeIdx
          // and propagate Taken flag to DFS parent unless at the Source
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      uint64_t Degree = AugmentingEdges[Src].size();
      for (auto &Edge : AugmentingEdges[Src]) {
        double EdgeFlow = Nodes[Src].FracFlow / Degree;
        Nodes[Edge->Dst].FracFlow += EdgeFlow;
        if (Edge->Capacity == INF)
          continue;
        uint64_t MaxIntFlow = double(Edge->Capacity - Edge->Flow) / EdgeFlow;
        MaxFlowAmount = std::min(MaxFlowAmount, MaxIntFlow);
      }
    }
    // Stop early if we cannot send any (integral) flow from Source to Target
    if (MaxFlowAmount == 0)
      return false;

```
- EN: Core entities appearing here include augmentFlowAlongDAG, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 augmentFlowAlongDAG，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 436-525

```cpp
    // Phase 2: Send an integral flow of MaxFlowAmount
    Nodes[Source].IntFlow = MaxFlowAmount;
    for (uint64_t Src : AugmentingOrder) {
      if (Src == Target)
        break;
      // Distribute flow evenly among successors of Src, rounding up to make
      // sure all flow is sent
      uint64_t Degree = AugmentingEdges[Src].size();
      // We are guaranteeed that Node[Src].IntFlow <= SuccFlow * Degree
      uint64_t SuccFlow = (Nodes[Src].IntFlow + Degree - 1) / Degree;
      for (auto &Edge : AugmentingEdges[Src]) {
        uint64_t Dst = Edge->Dst;
        uint64_t EdgeFlow = std::min(Nodes[Src].IntFlow, SuccFlow);
        EdgeFlow = std::min(EdgeFlow, uint64_t(Edge->Capacity - Edge->Flow));
        Nodes[Dst].IntFlow += EdgeFlow;
        Nodes[Src].IntFlow -= EdgeFlow;
        Edge->AugmentedFlow += EdgeFlow;
      }
    }
    assert(Nodes[Target].IntFlow <= MaxFlowAmount);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        uint64_t Dst = Edge.Dst;
        Edge.OnShortestPath =
            Src != Target && Dst != Source &&
            Nodes[Dst].Distance <= Nodes[Target].Distance &&
            Nodes[Dst].Distance == Nodes[Src].Distance + Edge.Cost &&
            Edge.Capacity > Edge.Flow &&
            uint64_t(Edge.Capacity - Edge.Flow) >= MinCapacity;
      }
    }
  }

  /// Maximum number of DFS iterations for DAG finding.
  static constexpr uint64_t MaxDfsCalls = 10;

```
- EN: Core entities appearing here include identifyShortestEdges, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 identifyShortestEdges，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 526-617

```cpp
  /// A node in a flow network.
  struct Node {
    /// The cost of the cheapest path from the source to the current node.
    int64_t Distance;
    /// The node preceding the current one in the path.
    uint64_t ParentNode;
    /// The index of the edge between ParentNode and the current node.
    uint64_t ParentEdgeIndex;
    /// An indicator of whether the current node is in a queue.
    bool Taken;

    /// Data fields utilized in DAG-augmentation:
    /// Fractional flow.
    double FracFlow;
    /// Integral flow.
    uint64_t IntFlow;
    /// Discovery time.
    uint64_t Discovery;
    /// Finish time.
    uint64_t Finish;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  /// Apply the post-processing.
  void run() {
    if (Params.JoinIslands) {
      // Adjust the flow to get rid of isolated components
      joinIsolatedComponents();
    }

    if (Params.RebalanceUnknown) {
      // Rebalance the flow inside unknown subgraphs
      rebalanceUnknownSubgraphs();
    }
  }

```
- EN: Core entities appearing here include Node, Edge, FlowAdjuster, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 Node, Edge, FlowAdjuster，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 618-705

```cpp
private:
  void joinIsolatedComponents() {
    // Find blocks that are reachable from the source
    auto Visited = BitVector(NumBlocks(), false);
    findReachable(Func.Entry, Visited);

    // Iterate over all non-reachable blocks and adjust their weights
    for (uint64_t I = 0; I < NumBlocks(); I++) {
      auto &Block = Func.Blocks[I];
      if (Block.Flow > 0 && !Visited[I]) {
        // Find a path from the entry to an exit passing through the block I
        auto Path = findShortestPath(I);
        // Increase the flow along the path
        assert(Path.size() > 0 && Path[0]->Source == Func.Entry &&
               "incorrectly computed path adjusting control flow");
        Func.Blocks[Func.Entry].Flow += 1;
        for (auto &Jump : Path) {
          Jump->Flow += 1;
          Func.Blocks[Jump->Target].Flow += 1;
          // Update reachability
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    auto Parent = std::vector<FlowJump *>(NumBlocks(), nullptr);
    Distance[Source] = 0;
    std::set<std::pair<uint64_t, uint64_t>> Queue;
    Queue.insert(std::make_pair(Distance[Source], Source));

    // Run the Dijkstra algorithm
    while (!Queue.empty()) {
      uint64_t Src = Queue.begin()->second;
      Queue.erase(Queue.begin());
      // If we found a solution, quit early
      if (Src == Target ||
          (Func.Blocks[Src].isExit() && Target == AnyExitBlock))
        break;

```
- EN: Core entities appearing here include joinIsolatedComponents, findReachable, findShortestPath, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 joinIsolatedComponents, findReachable, findShortestPath，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 706-794

```cpp
      for (auto *Jump : Func.Blocks[Src].SuccJumps) {
        uint64_t Dst = Jump->Target;
        int64_t JumpDist = jumpDistance(Jump);
        if (Distance[Dst] > Distance[Src] + JumpDist) {
          Queue.erase(std::make_pair(Distance[Dst], Dst));

          Distance[Dst] = Distance[Src] + JumpDist;
          Parent[Dst] = Jump;

          Queue.insert(std::make_pair(Distance[Dst], Dst));
        }
      }
    }
    // If Target is not provided, find the closest exit block
    if (Target == AnyExitBlock) {
      for (uint64_t I = 0; I < NumBlocks(); I++) {
        if (Func.Blocks[I].isExit() && Parent[I] != nullptr) {
          if (Target == AnyExitBlock || Distance[Target] > Distance[I]) {
            Target = I;
          }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      std::vector<FlowBlock *> KnownDstBlocks;
      findUnknownSubgraph(&SrcBlock, KnownDstBlocks, UnknownBlocks);

      // Verify if rebalancing of the subgraph is feasible. If the search is
      // successful, find the unique destination block (which can be null)
      FlowBlock *DstBlock = nullptr;
      if (!canRebalanceSubgraph(&SrcBlock, KnownDstBlocks, UnknownBlocks,
                                DstBlock))
        continue;

      // We cannot rebalance subgraphs containing cycles among unknown blocks
      if (!isAcyclicSubgraph(&SrcBlock, DstBlock, UnknownBlocks))
        continue;

```
- EN: Core entities appearing here include jumpDistance, rebalanceUnknownSubgraphs, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 jumpDistance, rebalanceUnknownSubgraphs，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 795-891

```cpp
      // Rebalance the flow
      rebalanceUnknownSubgraph(&SrcBlock, DstBlock, UnknownBlocks);
    }
  }

  /// Verify if rebalancing rooted at a given block is possible.
  bool canRebalanceAtRoot(const FlowBlock *SrcBlock) {
    // Do not attempt to find unknown subgraphs from an unknown or a
    // zero-flow block
    if (SrcBlock->HasUnknownWeight || SrcBlock->Flow == 0)
      return false;

    // Do not attempt to process subgraphs from a block w/o unknown sucessors
    bool HasUnknownSuccs = false;
    for (auto *Jump : SrcBlock->SuccJumps) {
      if (Func.Blocks[Jump->Target].HasUnknownWeight) {
        HasUnknownSuccs = true;
        break;
      }
    }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          return false;
        continue;
      }
      size_t NumIgnoredJumps = 0;
      for (auto *Jump : Block->SuccJumps) {
        if (ignoreJump(SrcBlock, DstBlock, Jump))
          NumIgnoredJumps++;
      }
      // If there is a non-sink block in UnknownBlocks with all jumps ignored,
      // then we can't rebalance
      if (NumIgnoredJumps == Block->SuccJumps.size())
        return false;
    }

```
- EN: Core entities appearing here include canRebalanceAtRoot, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 canRebalanceAtRoot，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 892-982

```cpp
    return true;
  }

  /// Decide whether the Jump is ignored while processing an unknown subgraphs
  /// rooted at basic block SrcBlock with the destination block, DstBlock.
  bool ignoreJump(const FlowBlock *SrcBlock, const FlowBlock *DstBlock,
                  const FlowJump *Jump) {
    // Ignore unlikely jumps with zero flow
    if (Jump->IsUnlikely && Jump->Flow == 0)
      return true;

    auto JumpSource = &Func.Blocks[Jump->Source];
    auto JumpTarget = &Func.Blocks[Jump->Target];

    // Do not ignore jumps coming into DstBlock
    if (DstBlock != nullptr && JumpTarget == DstBlock)
      return false;

    // Ignore jumps out of SrcBlock to known blocks
    if (!JumpTarget->HasUnknownWeight && JumpSource == SrcBlock)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // of all blocks
    if (UnknownBlocks.size() != AcyclicOrder.size())
      return false;
    UnknownBlocks = std::move(AcyclicOrder);
    return true;
  }

  /// Rebalance a given subgraph rooted at SrcBlock, ending at DstBlock and
  /// having UnknownBlocks intermediate blocks.
  void rebalanceUnknownSubgraph(const FlowBlock *SrcBlock,
                                const FlowBlock *DstBlock,
                                const std::vector<FlowBlock *> &UnknownBlocks) {
    assert(SrcBlock->Flow > 0 && "zero-flow block in unknown subgraph");

```
- EN: This region continues the SampleProfileInference implementation with local helper logic centered on Decide, Jump, SrcBlock, DstBlock.
- CN: 这一段延续了 SampleProfileInference 的主体实现，围绕 Decide, Jump, SrcBlock, DstBlock 等局部辅助逻辑展开。

### Lines 983-1071

```cpp
    // Ditribute flow from the source block
    uint64_t BlockFlow = 0;
    // SrcBlock's flow is the sum of outgoing flows along non-ignored jumps
    for (auto *Jump : SrcBlock->SuccJumps) {
      if (ignoreJump(SrcBlock, DstBlock, Jump))
        continue;
      BlockFlow += Jump->Flow;
    }
    rebalanceBlock(SrcBlock, DstBlock, SrcBlock, BlockFlow);

    // Ditribute flow from the remaining blocks
    for (auto *Block : UnknownBlocks) {
      assert(Block->HasUnknownWeight && "incorrect unknown subgraph");
      uint64_t BlockFlow = 0;
      // Block's flow is the sum of incoming flows
      for (auto *Jump : Block->PredJumps) {
        BlockFlow += Jump->Flow;
      }
      Block->Flow = BlockFlow;
      rebalanceBlock(SrcBlock, DstBlock, Block, BlockFlow);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  uint64_t NumBlocks = Func.Blocks.size();
  assert(NumBlocks > 1 && "Too few blocks in a function");
  uint64_t NumJumps = Func.Jumps.size();
  assert(NumJumps > 0 && "Too few jumps in a function");

  // Introducing dummy source/sink pairs to allow flow circulation.
  // The nodes corresponding to blocks of the function have indices in
  // the range [0 .. 2 * NumBlocks); the dummy sources/sinks are indexed by the
  // next four values.
  uint64_t S = 2 * NumBlocks;
  uint64_t T = S + 1;
  uint64_t S1 = S + 2;
  uint64_t T1 = S + 3;

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1072-1160

```cpp
  Network.initialize(2 * NumBlocks + 4, S1, T1);

  // Initialize nodes of the flow network
  for (uint64_t B = 0; B < NumBlocks; B++) {
    auto &Block = Func.Blocks[B];

    // Split every block into two auxiliary nodes to allow
    // increase/reduction of the block count.
    uint64_t Bin = 2 * B;
    uint64_t Bout = 2 * B + 1;

    // Edges from S and to T
    if (Block.isEntry()) {
      Network.addEdge(S, Bin, 0);
    } else if (Block.isExit()) {
      Network.addEdge(Bout, T, 0);
    }

    // Assign costs for increasing/decreasing the block counts
    auto [AuxCostInc, AuxCostDec] = assignBlockCosts(Params, Block);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      CostInc = Params.CostBlockEntryInc;
      CostDec = Params.CostBlockEntryDec;
    }
  }
  return std::make_pair(CostInc, CostDec);
}

/// Assign costs for increasing/decreasing the jump counts.
std::pair<int64_t, int64_t> assignJumpCosts(const ProfiParams &Params,
                                            const FlowJump &Jump) {
  // Modifying the weight of an unlikely jump is expensive
  if (Jump.IsUnlikely)
    return std::make_pair(Params.CostUnlikely, Params.CostUnlikely);

```
- EN: This region continues the SampleProfileInference implementation with local helper logic centered on Network, NumBlocks, Initialize, Block.
- CN: 这一段延续了 SampleProfileInference 的主体实现，围绕 Network, NumBlocks, Initialize, Block 等局部辅助逻辑展开。

### Lines 1161-1254

```cpp
  // Assign default values for the costs
  int64_t CostInc = Params.CostJumpInc;
  int64_t CostDec = Params.CostJumpDec;
  // Update the costs depending on the block metadata
  if (Jump.Source + 1 == Jump.Target) {
    // Adjusting the fall-through branch
    CostInc = Params.CostJumpFTInc;
    CostDec = Params.CostJumpFTDec;
  }
  if (Jump.HasUnknownWeight) {
    // The cost is different for fall-through and non-fall-through branches
    if (Jump.Source + 1 == Jump.Target)
      CostInc = Params.CostJumpUnknownFTInc;
    else
      CostInc = Params.CostJumpUnknownInc;
    CostDec = 0;
  }
  return std::make_pair(CostInc, CostDec);
}

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
           "a block cannot be an entry and an exit");
  }
  // Verify input block weights
  for (auto &Block : Func.Blocks) {
    assert((!Block.HasUnknownWeight || Block.Weight == 0 || Block.isEntry()) &&
           "non-zero weight of a block w/o weight except for an entry");
  }
  // Verify input jump weights
  for (auto &Jump : Func.Jumps) {
    assert((!Jump.HasUnknownWeight || Jump.Weight == 0) &&
           "non-zero weight of a jump w/o weight");
  }
}

```
- EN: Core entities appearing here include verifyInput, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 verifyInput，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 1255-1343

```cpp
/// Verify that the computed flow values satisfy flow conservation rules.
void verifyOutput(const FlowFunction &Func) {
  const uint64_t NumBlocks = Func.Blocks.size();
  auto InFlow = std::vector<uint64_t>(NumBlocks, 0);
  auto OutFlow = std::vector<uint64_t>(NumBlocks, 0);
  for (const auto &Jump : Func.Jumps) {
    InFlow[Jump.Target] += Jump.Flow;
    OutFlow[Jump.Source] += Jump.Flow;
  }

  uint64_t TotalInFlow = 0;
  uint64_t TotalOutFlow = 0;
  for (uint64_t I = 0; I < NumBlocks; I++) {
    auto &Block = Func.Blocks[I];
    if (Block.isEntry()) {
      TotalInFlow += Block.Flow;
      assert(Block.Flow == OutFlow[I] && "incorrectly computed control flow");
    } else if (Block.isExit()) {
      TotalOutFlow += Block.Flow;
      assert(Block.Flow == InFlow[I] && "incorrectly computed control flow");
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    if (Jump.Weight > 0)
      HasSamples = true;
    Jump.Flow = Jump.Weight;
  }

  // Quit early for functions with a single block or ones w/o samples
  if (Func.Blocks.size() <= 1 || !HasSamples)
    return;

#ifndef NDEBUG
  // Verify the input data
  verifyInput(Func);
#endif

```
- EN: Core entities appearing here include verifyOutput, applyFlowInference, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 verifyOutput, applyFlowInference，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 1344-1377

```cpp
  // Create and apply an inference network model
  auto InferenceNetwork = MinCostMaxFlow(Params);
  initializeNetwork(Params, InferenceNetwork, Func);
  InferenceNetwork.run();

  // Extract flow values for every block and every edge
  extractWeights(Params, InferenceNetwork, Func);

  // Post-processing adjustments to the flow
  auto Adjuster = FlowAdjuster(Params, Func);
  Adjuster.run();

#ifndef NDEBUG
  // Verify the result
  verifyOutput(Func);
#endif
}

/// Apply the profile inference algorithm for a given flow function
void llvm::applyFlowInference(FlowFunction &Func) {
  ProfiParams Params;
  // Set the params from the command-line flags.
  Params.EvenFlowDistribution = SampleProfileEvenFlowDistribution;
  Params.RebalanceUnknown = SampleProfileRebalanceUnknown;
  Params.JoinIslands = SampleProfileJoinIslands;
  Params.CostBlockInc = SampleProfileProfiCostBlockInc;
  Params.CostBlockDec = SampleProfileProfiCostBlockDec;
  Params.CostBlockEntryInc = SampleProfileProfiCostBlockEntryInc;
  Params.CostBlockEntryDec = SampleProfileProfiCostBlockEntryDec;
  Params.CostBlockZeroInc = SampleProfileProfiCostBlockZeroInc;
  Params.CostBlockUnknownInc = SampleProfileProfiCostBlockUnknownInc;

  applyFlowInference(Params, Func);
}
```
- EN: Core entities appearing here include applyFlowInference, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 applyFlowInference，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `MinCostMaxFlow, Node, Edge, FlowAdjuster, initialize, run, addEdge, getFlow` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`MinCostMaxFlow, Node, Edge, FlowAdjuster, initialize, run, addEdge, getFlow` 构成该文件对外 API 与主要实现挂钩。
- EN: Tuning surface: command-line knobs such as `SampleProfileEvenFlowDistribution, SampleProfileRebalanceUnknown, SampleProfileJoinIslands, SampleProfileProfiCostBlockInc, SampleProfileProfiCostBlockDec` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `SampleProfileEvenFlowDistribution, SampleProfileRebalanceUnknown, SampleProfileJoinIslands, SampleProfileProfiCostBlockInc, SampleProfileProfiCostBlockDec` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/Transforms/Utils/SampleProfileInference.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/Transforms/Utils/SampleProfileInference.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/ADT/BitVector.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/ADT/BitVector.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `queue`, `set`, `stack`, `unordered_set` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`queue`, `set`, `stack`, `unordered_set` 为 LLVM API 之外的 C++ 语言工具提供支持。
