# SuffixTree.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/SuffixTree.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements the Suffix Tree class.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `SuffixTree` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===- llvm/Support/SuffixTree.cpp - Implement Suffix Tree ------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the Suffix Tree class.
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/SuffixTree.h"
#include "llvm/Support/Allocator.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/SuffixTreeNode.h"

using namespace llvm;

/// \returns the number of elements in the substring associated with \p N.
static size_t numElementsInSubstring(const SuffixTreeNode *N) {
  assert(N && "Got a null node?");
  if (auto *Internal = dyn_cast<SuffixTreeInternalNode>(N))
    if (Internal->isRoot())
      return 0;
  return N->getEndIdx() - N->getStartIdx() + 1;
}

SuffixTree::SuffixTree(const ArrayRef<unsigned> &Str,
                       bool OutlinerLeafDescendants)
    : Str(Str), OutlinerLeafDescendants(OutlinerLeafDescendants) {
  Root = insertRoot();
  Active.Node = Root;

  // Keep track of the number of suffixes we have to add of the current
  // prefix.
  unsigned SuffixesToAdd = 0;

  // Construct the suffix tree iteratively on each prefix of the string.
  // PfxEndIdx is the end index of the current prefix.
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 4 direct dependencies, including `llvm/Support/SuffixTree.h`, `llvm/Support/Allocator.h`, `llvm/Support/Casting.h`, `llvm/Support/SuffixTreeNode.h`.
  CN: 引入了 4 个直接依赖，其中包括 `llvm/Support/SuffixTree.h`, `llvm/Support/Allocator.h`, `llvm/Support/Casting.h`, `llvm/Support/SuffixTreeNode.h`。
- EN: This section centers on `numElementsInSubstring`, `assert`, `SuffixTree` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `numElementsInSubstring`, `assert`, `SuffixTree` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 41-80

```cpp
  // End is one past the last element in the string.
  for (unsigned PfxEndIdx = 0, End = Str.size(); PfxEndIdx < End; PfxEndIdx++) {
    SuffixesToAdd++;
    LeafEndIdx = PfxEndIdx; // Extend each of the leaves.
    SuffixesToAdd = extend(PfxEndIdx, SuffixesToAdd);
  }

  // Set the suffix indices of each leaf.
  assert(Root && "Root node can't be nullptr!");
  setSuffixIndices();

  // Collect all leaf nodes of the suffix tree. And for each internal node,
  // record the range of leaf nodes that are descendants of it.
  if (OutlinerLeafDescendants)
    setLeafNodes();
}

SuffixTreeNode *SuffixTree::insertLeaf(SuffixTreeInternalNode &Parent,
                                       unsigned StartIdx, unsigned Edge) {
  assert(StartIdx <= LeafEndIdx && "String can't start after it ends!");
  auto *N = new (LeafNodeAllocator.Allocate())
      SuffixTreeLeafNode(StartIdx, &LeafEndIdx);
  Parent.Children[Edge] = N;
  return N;
}

SuffixTreeInternalNode *
SuffixTree::insertInternalNode(SuffixTreeInternalNode *Parent,
                               unsigned StartIdx, unsigned EndIdx,
                               unsigned Edge) {
  assert(StartIdx <= EndIdx && "String can't start after it ends!");
  assert(!(!Parent && StartIdx != SuffixTreeNode::EmptyIdx) &&
         "Non-root internal nodes must have parents!");
  auto *N = new (InternalNodeAllocator.Allocate())
      SuffixTreeInternalNode(StartIdx, EndIdx, Root);
  if (Parent)
    Parent->Children[Edge] = N;
  return N;
}

```
- EN: This section centers on `assert`, `setSuffixIndices`, `SuffixTreeLeafNode` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `assert`, `setSuffixIndices`, `SuffixTreeLeafNode` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 81-120

```cpp
SuffixTreeInternalNode *SuffixTree::insertRoot() {
  return insertInternalNode(/*Parent = */ nullptr, SuffixTreeNode::EmptyIdx,
                            SuffixTreeNode::EmptyIdx, /*Edge = */ 0);
}

void SuffixTree::setSuffixIndices() {
  // List of nodes we need to visit along with the current length of the
  // string.
  SmallVector<std::pair<SuffixTreeNode *, unsigned>> ToVisit;

  // Current node being visited.
  SuffixTreeNode *CurrNode = Root;

  // Sum of the lengths of the nodes down the path to the current one.
  unsigned CurrNodeLen = 0;
  ToVisit.push_back({CurrNode, CurrNodeLen});
  while (!ToVisit.empty()) {
    std::tie(CurrNode, CurrNodeLen) = ToVisit.pop_back_val();
    // Length of the current node from the root down to here.
    CurrNode->setConcatLen(CurrNodeLen);
    if (auto *InternalNode = dyn_cast<SuffixTreeInternalNode>(CurrNode))
      for (auto &ChildPair : InternalNode->Children) {
        assert(ChildPair.second && "Node had a null child!");
        ToVisit.push_back(
            {ChildPair.second,
             CurrNodeLen + numElementsInSubstring(ChildPair.second)});
      }
    // No children, so we are at the end of the string.
    if (auto *LeafNode = dyn_cast<SuffixTreeLeafNode>(CurrNode))
      LeafNode->setSuffixIdx(Str.size() - CurrNodeLen);
  }
}

void SuffixTree::setLeafNodes() {
  // A stack that keeps track of nodes to visit for post-order DFS traversal.
  SmallVector<SuffixTreeNode *> ToVisit;
  ToVisit.push_back(Root);

  // This keeps track of the index of the next leaf node to be added to
  // the LeafNodes vector of the suffix tree.
```
- EN: This section centers on `insertInternalNode`, `setSuffixIndices`, `tie` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `insertInternalNode`, `setSuffixIndices`, `tie` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 121-160

```cpp
  unsigned LeafCounter = 0;

  // This keeps track of nodes whose children have been added to the stack.
  // The value is a pair, representing a node's first and last children.
  DenseMap<SuffixTreeInternalNode *,
           std::pair<SuffixTreeNode *, SuffixTreeNode *>>
      ChildrenMap;

  // Traverse the tree in post-order.
  while (!ToVisit.empty()) {
    SuffixTreeNode *CurrNode = ToVisit.pop_back_val();
    if (auto *CurrInternalNode = dyn_cast<SuffixTreeInternalNode>(CurrNode)) {
      // The current node is an internal node.
      auto I = ChildrenMap.find(CurrInternalNode);
      if (I == ChildrenMap.end()) {
        // This is the first time we visit this node.
        // Its children have not been added to the stack yet.
        // We add current node back, and add its children to the stack.
        // We keep track of the first and last children of the current node.
        auto J = CurrInternalNode->Children.begin();
        if (J != CurrInternalNode->Children.end()) {
          ToVisit.push_back(CurrNode);
          SuffixTreeNode *FirstChild = J->second;
          SuffixTreeNode *LastChild = nullptr;
          for (; J != CurrInternalNode->Children.end(); ++J) {
            LastChild = J->second;
            ToVisit.push_back(LastChild);
          }
          ChildrenMap[CurrInternalNode] = {FirstChild, LastChild};
        }
      } else {
        // This is the second time we visit this node.
        // All of its children have already been processed.
        // Now, we can set its LeftLeafIdx and RightLeafIdx;
        auto [FirstChild, LastChild] = I->second;
        // Get the first child to use its RightLeafIdx.
        // The first child is the first one added to the stack, so it is
        // the last one to be processed. Hence, the leaf descendants
        // of the first child are assigned the largest index numbers.
        CurrNode->setRightLeafIdx(FirstChild->getRightLeafIdx());
```
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 161-200

```cpp
        // Get the last child to use its LeftLeafIdx.
        CurrNode->setLeftLeafIdx(LastChild->getLeftLeafIdx());
        assert(CurrNode->getLeftLeafIdx() <= CurrNode->getRightLeafIdx() &&
               "LeftLeafIdx should not be larger than RightLeafIdx");
      }
    } else {
      // The current node is a leaf node.
      // We can simply set its LeftLeafIdx and RightLeafIdx.
      CurrNode->setLeftLeafIdx(LeafCounter);
      CurrNode->setRightLeafIdx(LeafCounter);
      ++LeafCounter;
      auto *CurrLeafNode = cast<SuffixTreeLeafNode>(CurrNode);
      LeafNodes.push_back(CurrLeafNode);
    }
  }
}

unsigned SuffixTree::extend(unsigned EndIdx, unsigned SuffixesToAdd) {
  SuffixTreeInternalNode *NeedsLink = nullptr;

  while (SuffixesToAdd > 0) {

    // Are we waiting to add anything other than just the last character?
    if (Active.Len == 0) {
      // If not, then say the active index is the end index.
      Active.Idx = EndIdx;
    }

    assert(Active.Idx <= EndIdx && "Start index can't be after end index!");

    // The first character in the current substring we're looking at.
    unsigned FirstChar = Str[Active.Idx];

    // Have we inserted anything starting with FirstChar at the current node?
    if (auto It = Active.Node->Children.find(FirstChar);
        It == Active.Node->Children.end()) {
      // If not, then we can just insert a leaf and move to the next step.
      insertLeaf(*Active.Node, EndIdx, FirstChar);

      // The active node is an internal node, and we visited it, so it must
```
- EN: This section centers on `assert`, `extend`, `insertLeaf` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `assert`, `extend`, `insertLeaf` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 201-240

```cpp
      // need a link if it doesn't have one.
      if (NeedsLink) {
        NeedsLink->setLink(Active.Node);
        NeedsLink = nullptr;
      }
    } else {
      // There's a match with FirstChar, so look for the point in the tree to
      // insert a new node.
      SuffixTreeNode *NextNode = It->second;

      unsigned SubstringLen = numElementsInSubstring(NextNode);

      // Is the current suffix we're trying to insert longer than the size of
      // the child we want to move to?
      if (Active.Len >= SubstringLen) {
        // If yes, then consume the characters we've seen and move to the next
        // node.
        assert(isa<SuffixTreeInternalNode>(NextNode) &&
               "Expected an internal node?");
        Active.Idx += SubstringLen;
        Active.Len -= SubstringLen;
        Active.Node = cast<SuffixTreeInternalNode>(NextNode);
        continue;
      }

      // Otherwise, the suffix we're trying to insert must be contained in the
      // next node we want to move to.
      unsigned LastChar = Str[EndIdx];

      // Is the string we're trying to insert a substring of the next node?
      if (Str[NextNode->getStartIdx() + Active.Len] == LastChar) {
        // If yes, then we're done for this step. Remember our insertion point
        // and move to the next end index. At this point, we have an implicit
        // suffix tree.
        if (NeedsLink && !Active.Node->isRoot()) {
          NeedsLink->setLink(Active.Node);
          NeedsLink = nullptr;
        }

        Active.Len++;
```
- EN: This section centers on `assert` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `assert` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 241-280

```cpp
        break;
      }

      // The string we're trying to insert isn't a substring of the next node,
      // but matches up to a point. Split the node.
      //
      // For example, say we ended our search at a node n and we're trying to
      // insert ABD. Then we'll create a new node s for AB, reduce n to just
      // representing C, and insert a new leaf node l to represent d. This
      // allows us to ensure that if n was a leaf, it remains a leaf.
      //
      //   | ABC  ---split--->  | AB
      //   n                    s
      //                     C / \ D
      //                      n   l

      // The node s from the diagram
      SuffixTreeInternalNode *SplitNode = insertInternalNode(
          Active.Node, NextNode->getStartIdx(),
          NextNode->getStartIdx() + Active.Len - 1, FirstChar);

      // Insert the new node representing the new substring into the tree as
      // a child of the split node. This is the node l from the diagram.
      insertLeaf(*SplitNode, EndIdx, LastChar);

      // Make the old node a child of the split node and update its start
      // index. This is the node n from the diagram.
      NextNode->incrementStartIdx(Active.Len);
      SplitNode->Children[Str[NextNode->getStartIdx()]] = NextNode;

      // SplitNode is an internal node, update the suffix link.
      if (NeedsLink)
        NeedsLink->setLink(SplitNode);

      NeedsLink = SplitNode;
    }

    // We've added something new to the tree, so there's one less suffix to
    // add.
    SuffixesToAdd--;
```
- EN: This section centers on `insertLeaf` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `insertLeaf` 等符号展开，负责实现局部控制流程与状态维护。
- EN: This range checks conditions and handles edge cases.
  CN: 这一段检查条件并处理边界情况。

### Lines 281-320

```cpp

    if (Active.Node->isRoot()) {
      if (Active.Len > 0) {
        Active.Len--;
        Active.Idx = EndIdx - SuffixesToAdd + 1;
      }
    } else {
      // Start the next phase at the next smallest suffix.
      Active.Node = Active.Node->getLink();
    }
  }

  return SuffixesToAdd;
}

void SuffixTree::RepeatedSubstringIterator::advance() {
  // Clear the current state. If we're at the end of the range, then this
  // is the state we want to be in.
  RS = RepeatedSubstring();
  N = nullptr;

  // Each leaf node represents a repeat of a string.
  SmallVector<unsigned> RepeatedSubstringStarts;

  // Continue visiting nodes until we find one which repeats more than once.
  while (!InternalNodesToVisit.empty()) {
    RepeatedSubstringStarts.clear();
    auto *Curr = InternalNodesToVisit.pop_back_val();

    // Keep track of the length of the string associated with the node. If
    // it's too short, we'll quit.
    unsigned Length = Curr->getConcatLen();

    // Iterate over each child, saving internal nodes for visiting.
    // Internal nodes represent individual strings, which may repeat.
    for (auto &ChildPair : Curr->Children)
      // Save all of this node's children for processing.
      if (auto *InternalChild =
              dyn_cast<SuffixTreeInternalNode>(ChildPair.second))
        InternalNodesToVisit.push_back(InternalChild);
```
- EN: This section centers on `advance` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `advance` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 321-355

```cpp

    // If length of repeated substring is below threshold, then skip it.
    if (Length < MinLength)
      continue;

    // The root never represents a repeated substring. If we're looking at
    // that, then skip it.
    if (Curr->isRoot())
      continue;

    // Collect leaf children or leaf descendants by OutlinerLeafDescendants.
    if (OutlinerLeafDescendants) {
      for (unsigned I = Curr->getLeftLeafIdx(); I <= Curr->getRightLeafIdx();
           ++I)
        RepeatedSubstringStarts.push_back(LeafNodes[I]->getSuffixIdx());
    } else {
      for (auto &ChildPair : Curr->Children)
        if (auto *Leaf = dyn_cast<SuffixTreeLeafNode>(ChildPair.second))
          RepeatedSubstringStarts.push_back(Leaf->getSuffixIdx());
    }

    // Do we have any repeated substrings?
    if (RepeatedSubstringStarts.size() < 2)
      continue;

    // Yes. Update the state to reflect this, and then bail out.
    N = Curr;
    RS.Length = Length;
    llvm::append_range(RS.StartIndices, RepeatedSubstringStarts);
    break;
  }
  // At this point, either NewRS is an empty RepeatedSubstring, or it was
  // set in the above loop. Similarly, N is either nullptr, or the node
  // associated with NewRS.
}
```
- EN: This section centers on `append_range` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `append_range` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `numElementsInSubstring`, `assert`, `SuffixTree`, `setSuffixIndices` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/SuffixTree.h`, `llvm/Support/Allocator.h`, `llvm/Support/Casting.h`, `llvm/Support/SuffixTreeNode.h`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `numElementsInSubstring`, `assert`, `SuffixTree`, `setSuffixIndices`, `SuffixTreeLeafNode`
