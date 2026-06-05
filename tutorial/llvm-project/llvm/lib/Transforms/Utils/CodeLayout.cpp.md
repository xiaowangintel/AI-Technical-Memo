# CodeLayout.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/CodeLayout.cpp` | `llvm/lib/Transforms/Utils/CodeLayout.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements implementation of code layout algorithms within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 CodeLayout 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-96

```cpp
//===- CodeLayout.cpp - Implementation of code layout algorithms ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// The file implements "cache-aware" layout algorithms of basic blocks and
// functions in a binary.
//
// The algorithm tries to find a layout of nodes (basic blocks) of a given CFG
// optimizing jump locality and thus processor I-cache utilization. This is
// achieved via increasing the number of fall-through jumps and co-locating
// frequently executed nodes together. The name follows the underlying
// optimization problem, Extended-TSP, which is a generalization of classical
// (maximum) Traveling Salesmen Problem.
//
// The algorithm is a greedy heuristic that works with chains (ordered lists)
// of basic blocks. Initially all chains are isolated basic blocks. On every
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    cl::desc("The weight of unconditional backward jumps for ExtTSP value"));

static cl::opt<double> FallthroughWeightCond(
    "ext-tsp-fallthrough-weight-cond", cl::ReallyHidden, cl::init(1.0),
    cl::desc("The weight of conditional fallthrough jumps for ExtTSP value"));

static cl::opt<double> FallthroughWeightUncond(
    "ext-tsp-fallthrough-weight-uncond", cl::ReallyHidden, cl::init(1.05),
    cl::desc("The weight of unconditional fallthrough jumps for ExtTSP value"));

static cl::opt<unsigned> ForwardDistance(
    "ext-tsp-forward-distance", cl::ReallyHidden, cl::init(1024),
    cl::desc("The maximum distance (in bytes) of a forward jump for ExtTSP"));

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 97-191

```cpp
static cl::opt<unsigned> BackwardDistance(
    "ext-tsp-backward-distance", cl::ReallyHidden, cl::init(640),
    cl::desc("The maximum distance (in bytes) of a backward jump for ExtTSP"));

// The maximum size of a chain created by the algorithm. The size is bounded
// so that the algorithm can efficiently process extremely large instances.
static cl::opt<unsigned>
    MaxChainSize("ext-tsp-max-chain-size", cl::ReallyHidden, cl::init(512),
                 cl::desc("The maximum size of a chain to create"));

// The maximum size of a chain for splitting. Larger values of the threshold
// may yield better quality at the cost of worsen run-time.
static cl::opt<unsigned> ChainSplitThreshold(
    "ext-tsp-chain-split-threshold", cl::ReallyHidden, cl::init(128),
    cl::desc("The maximum size of a chain to apply splitting"));

// The maximum ratio between densities of two chains for merging.
static cl::opt<double> MaxMergeDensityRatio(
    "ext-tsp-max-merge-density-ratio", cl::ReallyHidden, cl::init(100),
    cl::desc("The maximum ratio between densities of two chains for merging"));
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

/// The gain of merging two chains, that is, the Ext-TSP score of the merge
/// together with the corresponding merge 'type' and 'offset'.
struct MergeGainT {
  explicit MergeGainT() = default;
  explicit MergeGainT(double Score, size_t MergeOffset, MergeTypeT MergeType)
      : Score(Score), MergeOffset(MergeOffset), MergeType(MergeType) {}

  double score() const { return Score; }

  size_t mergeOffset() const { return MergeOffset; }

  MergeTypeT mergeType() const { return MergeType; }

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. Core entities appearing here include MergeTypeT, MergeGainT, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 此处出现的核心实体包括 MergeTypeT, MergeGainT，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 192-287

```cpp
  void setMergeType(MergeTypeT Ty) { MergeType = Ty; }

  // Returns 'true' iff Other is preferred over this.
  bool operator<(const MergeGainT &Other) const {
    return (Other.Score > EPS && Other.Score > Score + EPS);
  }

  // Update the current gain if Other is preferred over this.
  void updateIfLessThan(const MergeGainT &Other) {
    if (*this < Other)
      *this = Other;
  }

private:
  double Score{-1.0};
  size_t MergeOffset{0};
  MergeTypeT MergeType{MergeTypeT::X_Y};
};

struct JumpT;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  uint64_t ExecutionCount{0};
  // Whether the jump corresponds to a conditional branch.
  bool IsConditional{false};
  // The offset of the jump from the source node.
  uint64_t Offset{0};
};

/// A chain (ordered sequence) of nodes in the graph.
struct ChainT {
  ChainT(const ChainT &) = delete;
  ChainT(ChainT &&) = default;
  ChainT &operator=(const ChainT &) = delete;
  ChainT &operator=(ChainT &&) = default;

```
- EN: Core entities appearing here include updateIfLessThan, JumpT, ChainT, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 updateIfLessThan, JumpT, ChainT，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 288-382

```cpp
  explicit ChainT(uint64_t Id, NodeT *Node)
      : Id(Id), ExecutionCount(Node->ExecutionCount), Size(Node->Size),
        Nodes(1, Node) {}

  size_t numBlocks() const { return Nodes.size(); }

  double density() const { return ExecutionCount / Size; }

  bool isEntry() const { return Nodes[0]->Index == 0; }

  bool isCold() const {
    for (NodeT *Node : Nodes) {
      if (Node->ExecutionCount > 0)
        return false;
    }
    return true;
  }

  ChainEdge *getEdge(ChainT *Other) const {
    for (const auto &[Chain, ChainEdge] : Edges) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
struct ChainEdge {
  ChainEdge(const ChainEdge &) = delete;
  ChainEdge(ChainEdge &&) = default;
  ChainEdge &operator=(const ChainEdge &) = delete;
  ChainEdge &operator=(ChainEdge &&) = delete;

  explicit ChainEdge(JumpT *Jump)
      : SrcChain(Jump->Source->CurChain), DstChain(Jump->Target->CurChain),
        Jumps(1, Jump) {}

  ChainT *srcChain() const { return SrcChain; }

  ChainT *dstChain() const { return DstChain; }

```
- EN: Core entities appearing here include isCold, removeEdge, addEdge, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 isCold, removeEdge, addEdge，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 383-472

```cpp
  bool isSelfEdge() const { return SrcChain == DstChain; }

  const std::vector<JumpT *> &jumps() const { return Jumps; }

  void appendJump(JumpT *Jump) { Jumps.push_back(Jump); }

  void moveJumps(ChainEdge *Other) {
    llvm::append_range(Jumps, Other->Jumps);
    Other->Jumps.clear();
    Other->Jumps.shrink_to_fit();
  }

  void changeEndpoint(ChainT *From, ChainT *To) {
    if (From == SrcChain)
      SrcChain = To;
    if (From == DstChain)
      DstChain = To;
  }

  bool hasCachedMergeGain(ChainT *Src, ChainT *Dst) const {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
uint64_t NodeT::outCount() const {
  uint64_t Count = 0;
  for (JumpT *Jump : OutJumps)
    Count += Jump->ExecutionCount;
  return Count;
}

uint64_t NodeT::inCount() const {
  uint64_t Count = 0;
  for (JumpT *Jump : InJumps)
    Count += Jump->ExecutionCount;
  return Count;
}

```
- EN: Core entities appearing here include moveJumps, changeEndpoint, hasCachedMergeGain, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 moveJumps, changeEndpoint, hasCachedMergeGain，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 473-571

```cpp
void ChainT::mergeEdges(ChainT *Other) {
  // Update edges adjacent to chain Other.
  for (const auto &[DstChain, DstEdge] : Other->Edges) {
    ChainT *TargetChain = DstChain == Other ? this : DstChain;
    ChainEdge *CurEdge = getEdge(TargetChain);
    if (CurEdge == nullptr) {
      DstEdge->changeEndpoint(Other, this);
      this->addEdge(TargetChain, DstEdge);
      if (DstChain != this && DstChain != Other)
        DstChain->addEdge(this, DstEdge);
    } else {
      CurEdge->moveJumps(DstEdge);
    }
    // Cleanup leftover edge.
    if (DstChain != Other)
      DstChain->removeEdge(Other);
  }
}

using NodeIter = std::vector<NodeT *>::const_iterator;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
/// If MergeType == 0, then the result is a concatenation of two chains.
/// Otherwise, the first chain is cut into two sub-chains at the offset,
/// and merged using all possible ways of concatenating three chains.
MergedNodesT mergeNodes(const std::vector<NodeT *> &X,
                        const std::vector<NodeT *> &Y, size_t MergeOffset,
                        MergeTypeT MergeType) {
  // Split the first chain, X, into X1 and X2.
  NodeIter BeginX1 = X.begin();
  NodeIter EndX1 = X.begin() + MergeOffset;
  NodeIter BeginX2 = X.begin() + MergeOffset;
  NodeIter EndX2 = X.end();
  NodeIter BeginY = Y.begin();
  NodeIter EndY = Y.end();

```
- EN: Core entities appearing here include mergeEdges, MergedNodesT, h, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 mergeEdges, MergedNodesT, h，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 572-668

```cpp
  // Construct a new chain from the three existing ones.
  switch (MergeType) {
  case MergeTypeT::X_Y:
    return MergedNodesT(BeginX1, EndX2, BeginY, EndY);
  case MergeTypeT::Y_X:
    return MergedNodesT(BeginY, EndY, BeginX1, EndX2);
  case MergeTypeT::X1_Y_X2:
    return MergedNodesT(BeginX1, EndX1, BeginY, EndY, BeginX2, EndX2);
  case MergeTypeT::Y_X2_X1:
    return MergedNodesT(BeginY, EndY, BeginX2, EndX2, BeginX1, EndX1);
  case MergeTypeT::X2_X1_Y:
    return MergedNodesT(BeginX2, EndX2, BeginX1, EndX1, BeginY, EndY);
  }
  llvm_unreachable("unexpected chain merge type");
}

/// The implementation of the ExtTSP algorithm.
class ExtTSPImpl {
public:
  ExtTSPImpl(ArrayRef<uint64_t> NodeSizes, ArrayRef<uint64_t> NodeCounts,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      Jump.IsConditional = OutDegree[Jump.Source->Index] > 1;
    }

    // Initialize chains.
    AllChains.reserve(NumNodes);
    HotChains.reserve(NumNodes);
    for (NodeT &Node : AllNodes) {
      // Create a chain.
      AllChains.emplace_back(Node.Index, &Node);
      Node.CurChain = &AllChains.back();
      if (Node.ExecutionCount > 0)
        HotChains.push_back(&AllChains.back());
    }

```
- EN: Core entities appearing here include ExtTSPImpl, NumNodes, run, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 ExtTSPImpl, NumNodes, run，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 669-776

```cpp
    // Initialize chain edges.
    AllEdges.reserve(AllJumps.size());
    for (NodeT &PredNode : AllNodes) {
      for (JumpT *Jump : PredNode.OutJumps) {
        assert(Jump->ExecutionCount > 0 && "incorrectly initialized jump");
        NodeT *SuccNode = Jump->Target;
        ChainEdge *CurEdge = PredNode.CurChain->getEdge(SuccNode->CurChain);
        // This edge is already present in the graph.
        if (CurEdge != nullptr) {
          assert(SuccNode->CurChain->getEdge(PredNode.CurChain) != nullptr);
          CurEdge->appendJump(Jump);
          continue;
        }
        // This is a new edge.
        AllEdges.emplace_back(Jump);
        PredNode.CurChain->addEdge(SuccNode->CurChain, &AllEdges.back());
        SuccNode->CurChain->addEdge(PredNode.CurChain, &AllEdges.back());
      }
    }
  }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          // Don't merge the chains if they have vastly different densities.
          // Skip the merge if the ratio between the densities exceeds
          // MaxMergeDensityRatio. Smaller values of the option result in fewer
          // merges, and hence, more chains.
          const double ChainPredDensity = ChainPred->density();
          const double ChainSuccDensity = ChainSucc->density();
          assert(ChainPredDensity > 0.0 && ChainSuccDensity > 0.0 &&
                 "incorrectly computed chain densities");
          auto [MinDensity, MaxDensity] =
              std::minmax(ChainPredDensity, ChainSuccDensity);
          const double Ratio = MaxDensity / MinDensity;
          if (Ratio > MaxMergeDensityRatio)
            continue;

```
- EN: Core entities appearing here include mergeForcedPairs, mergeChainPairs, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 mergeForcedPairs, mergeChainPairs，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 777-882

```cpp
          // Compute the gain of merging the two chains.
          MergeGainT CurGain = getBestMergeGain(ChainPred, ChainSucc, Edge);
          if (CurGain.score() <= EPS)
            continue;

          if (BestGain < CurGain ||
              (std::abs(CurGain.score() - BestGain.score()) < EPS &&
               compareChainPairs(ChainPred, ChainSucc, BestChainPred,
                                 BestChainSucc))) {
            BestGain = CurGain;
            BestChainPred = ChainPred;
            BestChainSucc = ChainSucc;
          }
        }
      }

      // Stop merging when there is no improvement.
      if (BestGain.score() <= EPS)
        break;

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      if (Offset == 0 || Offset == ChainPred->Nodes.size())
        return;
      // Skip merging if it breaks Forced successors.
      NodeT *Node = ChainPred->Nodes[Offset - 1];
      if (Node->ForcedSucc != nullptr)
        return;
      // Apply the merge, compute the corresponding gain, and update the best
      // value, if the merge is beneficial.
      for (const MergeTypeT &MergeType : MergeTypes) {
        Gain.updateIfLessThan(
            computeMergeGain(ChainPred, ChainSucc, Jumps, Offset, MergeType));
      }
    };

```
- EN: Core entities appearing here include mergeColdChains, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 mergeColdChains，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 883-978

```cpp
    // Try to concatenate two chains w/o splitting.
    Gain.updateIfLessThan(
        computeMergeGain(ChainPred, ChainSucc, Jumps, 0, MergeTypeT::X_Y));

    // Attach (a part of) ChainPred before the first node of ChainSucc.
    for (JumpT *Jump : ChainSucc->Nodes.front()->InJumps) {
      const NodeT *SrcBlock = Jump->Source;
      if (SrcBlock->CurChain != ChainPred)
        continue;
      size_t Offset = SrcBlock->CurIndex + 1;
      tryChainMerging(Offset, {MergeTypeT::X1_Y_X2, MergeTypeT::X2_X1_Y});
    }

    // Attach (a part of) ChainPred after the last node of ChainSucc.
    for (JumpT *Jump : ChainSucc->Nodes.back()->OutJumps) {
      const NodeT *DstBlock = Jump->Target;
      if (DstBlock->CurChain != ChainPred)
        continue;
      size_t Offset = DstBlock->CurIndex;
      tryChainMerging(Offset, {MergeTypeT::X1_Y_X2, MergeTypeT::Y_X2_X1});
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    if (SelfEdge != nullptr) {
      MergedNodes = MergedNodesT(Into->Nodes.begin(), Into->Nodes.end());
      MergedJumpsT MergedJumps(&SelfEdge->jumps());
      Into->Score = extTSPScore(MergedNodes, MergedJumps);
    }

    // Remove the chain from the list of active chains.
    llvm::erase(HotChains, From);

    // Invalidate caches.
    for (auto EdgeIt : Into->Edges)
      EdgeIt.second->invalidateCache();
  }

```
- EN: This region continues the CodeLayout implementation with local helper logic centered on Try, Gain, ChainPred, ChainSucc.
- CN: 这一段延续了 CodeLayout 的主体实现，围绕 Try, Gain, ChainPred, ChainSucc 等局部辅助逻辑展开。

### Lines 979-1081

```cpp
  /// Concatenate all chains into the final order.
  std::vector<uint64_t> concatChains() {
    // Collect non-empty chains.
    std::vector<const ChainT *> SortedChains;
    for (ChainT &Chain : AllChains) {
      if (!Chain.Nodes.empty())
        SortedChains.push_back(&Chain);
    }

    // Sorting chains by density in the decreasing order.
    std::sort(SortedChains.begin(), SortedChains.end(),
              [&](const ChainT *L, const ChainT *R) {
                // Place the entry point at the beginning of the order.
                if (L->isEntry() != R->isEntry())
                  return L->isEntry();

                // Compare by density and break ties by chain identifiers.
                return std::make_tuple(-L->density(), L->Id) <
                       std::make_tuple(-R->density(), R->Id);
              });
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      if (ExecutionCount > 0)
        TotalSize += Size;
    }

    // Initialize jumps between the nodes.
    SuccNodes.resize(NumNodes);
    PredNodes.resize(NumNodes);
    AllJumps.reserve(EdgeCounts.size());
    for (size_t I = 0; I < EdgeCounts.size(); I++) {
      auto [Pred, Succ, Count] = EdgeCounts[I];
      // Ignore recursive calls.
      if (Pred == Succ)
        continue;

```
- EN: Core entities appearing here include concatChains, CDSortImpl, Config, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 concatChains, CDSortImpl, Config，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 1082-1179

```cpp
      SuccNodes[Pred].push_back(Succ);
      PredNodes[Succ].push_back(Pred);
      if (Count > 0) {
        NodeT &PredNode = AllNodes[Pred];
        NodeT &SuccNode = AllNodes[Succ];
        AllJumps.emplace_back(&PredNode, &SuccNode, Count);
        AllJumps.back().Offset = EdgeOffsets[I];
        SuccNode.InJumps.push_back(&AllJumps.back());
        PredNode.OutJumps.push_back(&AllJumps.back());
        // Adjust execution counts.
        PredNode.ExecutionCount = std::max(PredNode.ExecutionCount, Count);
        SuccNode.ExecutionCount = std::max(SuccNode.ExecutionCount, Count);
      }
    }

    // Initialize chains.
    AllChains.reserve(NumNodes);
    for (NodeT &Node : AllNodes) {
      // Adjust execution counts.
      Node.ExecutionCount = std::max(Node.ExecutionCount, Node.inCount());
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      ChainT *BestDstChain = BestEdge->dstChain();

      // Remove outdated edges from the queue.
      for (const auto &[_, ChainEdge] : BestSrcChain->Edges)
        Queue.erase(ChainEdge);
      for (const auto &[_, ChainEdge] : BestDstChain->Edges)
        Queue.erase(ChainEdge);

      // Merge the best pair of chains.
      MergeGainT BestGain = BestEdge->getMergeGain();
      mergeChains(BestSrcChain, BestDstChain, BestGain.mergeOffset(),
                  BestGain.mergeType());
      --NumActiveChains;

```
- EN: Core entities appearing here include mergeChainPairs, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 mergeChainPairs，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1180-1279

```cpp
      // Insert newly created edges into the queue.
      for (const auto &[_, Edge] : BestSrcChain->Edges) {
        // Ignore loop edges.
        if (Edge->isSelfEdge())
          continue;
        if (Edge->srcChain()->numBlocks() + Edge->dstChain()->numBlocks() >
            Config.MaxChainSize)
          continue;

        // Compute the gain of merging the two chains.
        MergeGainT Gain = getBestMergeGain(Edge);
        Edge->setMergeGain(Gain);

        if (Edge->gain() > EPS)
          Queue.insert(Edge);
      }
    }

    LLVM_DEBUG(dbgs() << "Cache-directed function sorting reduced the number"
                      << " of chains from " << NumNodes << " to "
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

    return MergeGainT(GainScore, MergeOffset, MergeType);
  }

  /// Compute the change of the frequency locality after merging the chains.
  double freqBasedLocalityGain(ChainT *ChainPred, ChainT *ChainSucc) const {
    auto missProbability = [&](double ChainDensity) {
      double PageSamples = ChainDensity * Config.CacheSize;
      if (PageSamples >= TotalSamples)
        return 0.0;
      double P = PageSamples / TotalSamples;
      return pow(1.0 - P, static_cast<double>(Config.CacheEntries));
    };

```
- EN: Core entities appearing here include getBestMergeGain, freqBasedLocalityGain, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 getBestMergeGain, freqBasedLocalityGain，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 1280-1375

```cpp
    // Cache misses on the chains before merging.
    double CurScore =
        ChainPred->ExecutionCount * missProbability(ChainPred->density()) +
        ChainSucc->ExecutionCount * missProbability(ChainSucc->density());

    // Cache misses on the merged chain
    double MergedCounts = ChainPred->ExecutionCount + ChainSucc->ExecutionCount;
    double MergedSize = ChainPred->Size + ChainSucc->Size;
    double MergedDensity = MergedCounts / MergedSize;
    double NewScore = MergedCounts * missProbability(MergedDensity);

    return CurScore - NewScore;
  }

  /// Compute the distance locality for a jump / call.
  double distScore(uint64_t SrcAddr, uint64_t DstAddr, uint64_t Count) const {
    uint64_t Dist = SrcAddr <= DstAddr ? DstAddr - SrcAddr : SrcAddr - DstAddr;
    double D = Dist == 0 ? 0.1 : static_cast<double>(Dist);
    return static_cast<double>(Count) * std::pow(D, -Config.DistancePower);
  }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                // Compare by density and break ties by chain identifiers.
                return std::make_tuple(-DL, L->Id) <
                       std::make_tuple(-DR, R->Id);
              });

    // Collect the nodes in the order specified by their chains.
    std::vector<uint64_t> Order;
    Order.reserve(NumNodes);
    for (const ChainT *Chain : SortedChains)
      for (NodeT *Node : Chain->Nodes)
        Order.push_back(Node->Index);
    return Order;
  }

```
- EN: Core entities appearing here include distScore, concatChains, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 distScore, concatChains，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 1376-1470

```cpp
private:
  /// Config for the algorithm.
  const CDSortConfig Config;

  /// The number of nodes in the graph.
  const size_t NumNodes;

  /// Successors of each node.
  std::vector<std::vector<uint64_t>> SuccNodes;

  /// Predecessors of each node.
  std::vector<std::vector<uint64_t>> PredNodes;

  /// All nodes (functions) in the graph.
  std::vector<NodeT> AllNodes;

  /// All jumps (function calls) between the nodes.
  std::vector<JumpT> AllJumps;

  /// All chains of nodes.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
std::vector<uint64_t> codelayout::computeCacheDirectedLayout(
    const CDSortConfig &Config, ArrayRef<uint64_t> FuncSizes,
    ArrayRef<uint64_t> FuncCounts, ArrayRef<EdgeCount> CallCounts,
    ArrayRef<uint64_t> CallOffsets) {
  // Verify correctness of the input data.
  assert(FuncCounts.size() == FuncSizes.size() && "Incorrect input");

  // Apply the reordering algorithm.
  CDSortImpl Alg(Config, FuncSizes, FuncCounts, CallCounts, CallOffsets);
  std::vector<uint64_t> Result = Alg.run();
  assert(Result.size() == FuncSizes.size() && "Incorrect size of layout");
  return Result;
}

```
- EN: This region continues the CodeLayout implementation with local helper logic centered on Config, CDSortConfig, The, NumNodes.
- CN: 这一段延续了 CodeLayout 的主体实现，围绕 Config, CDSortConfig, The, NumNodes 等局部辅助逻辑展开。

### Lines 1471-1488

```cpp
std::vector<uint64_t> codelayout::computeCacheDirectedLayout(
    ArrayRef<uint64_t> FuncSizes, ArrayRef<uint64_t> FuncCounts,
    ArrayRef<EdgeCount> CallCounts, ArrayRef<uint64_t> CallOffsets) {
  CDSortConfig Config;
  // Populate the config from the command-line options.
  if (CacheEntries.getNumOccurrences() > 0)
    Config.CacheEntries = CacheEntries;
  if (CacheSize.getNumOccurrences() > 0)
    Config.CacheSize = CacheSize;
  if (CDMaxChainSize.getNumOccurrences() > 0)
    Config.MaxChainSize = CDMaxChainSize;
  if (DistancePower.getNumOccurrences() > 0)
    Config.DistancePower = DistancePower;
  if (FrequencyScale.getNumOccurrences() > 0)
    Config.FrequencyScale = FrequencyScale;
  return computeCacheDirectedLayout(Config, FuncSizes, FuncCounts, CallCounts,
                                    CallOffsets);
}
```
- EN: This region continues the CodeLayout implementation with local helper logic centered on ArrayRef, FuncSizes, FuncCounts, EdgeCount.
- CN: 这一段延续了 CodeLayout 的主体实现，围绕 ArrayRef, FuncSizes, FuncCounts, EdgeCount 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `MergeTypeT, MergeGainT, JumpT, ChainT, ChainEdge, NodeT, MergedNodesT, MergedJumpsT` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`MergeTypeT, MergeGainT, JumpT, ChainT, ChainEdge, NodeT, MergedNodesT, MergedJumpsT` 构成该文件对外 API 与主要实现挂钩。
- EN: Tuning surface: command-line knobs such as `ForwardWeightCond, ForwardWeightUncond, BackwardWeightCond, BackwardWeightUncond, FallthroughWeightCond` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `ForwardWeightCond, ForwardWeightUncond, BackwardWeightCond, BackwardWeightUncond, FallthroughWeightCond` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/Transforms/Utils/CodeLayout.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/Transforms/Utils/CodeLayout.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/CommandLine.h`, `llvm/Support/Debug.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `cmath`, `set` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`cmath`, `set` 为 LLVM API 之外的 C++ 语言工具提供支持。
