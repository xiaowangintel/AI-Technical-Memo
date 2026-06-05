# DeltaTree.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/DeltaTree.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements the DeltaTree and related classes.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `DeltaTree` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60

```cpp
//===- DeltaTree.cpp - B-Tree for Rewrite Delta tracking ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the DeltaTree and related classes.
//
//===----------------------------------------------------------------------===//

#include "llvm/ADT/DeltaTree.h"
#include "llvm/Support/Casting.h"
#include <cassert>
#include <cstring>

using namespace llvm;

/// The DeltaTree class is a multiway search tree (BTree) structure with some
/// fancy features.  B-Trees are generally more memory and cache efficient
/// than binary trees, because they store multiple keys/values in each node.
///
/// DeltaTree implements a key/value mapping from FileIndex to Delta, allowing
/// fast lookup by FileIndex.  However, an added (important) bonus is that it
/// can also efficiently tell us the full accumulated delta for a specific
/// file offset as well, without traversing the whole tree.
///
/// The nodes of the tree are made up of instances of two classes:
/// DeltaTreeNode and DeltaTreeInteriorNode.  The later subclasses the
/// former and adds children pointers.  Each node knows the full delta of all
/// entries (recursively) contained inside of it, which allows us to get the
/// full delta implied by a whole subtree in constant time.

namespace {

/// SourceDelta - As code in the original input buffer is added and deleted,
/// SourceDelta records are used to keep track of how the input SourceLocation
/// object is mapped into the output buffer.
struct SourceDelta {
  unsigned FileLoc;
  int Delta;

  static SourceDelta get(unsigned Loc, int D) {
    SourceDelta Delta;
    Delta.FileLoc = Loc;
    Delta.Delta = D;
    return Delta;
  }
};

/// DeltaTreeNode - The common part of all nodes.
///
class DeltaTreeNode {
public:
  struct InsertResult {
    DeltaTreeNode *LHS, *RHS;
    SourceDelta Split;
  };

```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 4 direct dependencies, including `llvm/ADT/DeltaTree.h`, `llvm/Support/Casting.h`, `cassert`, `cstring`.
  CN: 引入了 4 个直接依赖，其中包括 `llvm/ADT/DeltaTree.h`, `llvm/Support/Casting.h`, `cassert`, `cstring`。
- EN: This section centers on `get` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `get` 等符号展开，负责查询现有状态并返回计算出的元数据。

### Lines 61-120

```cpp
private:
  friend class DeltaTreeInteriorNode;

  /// WidthFactor - This controls the number of K/V slots held in the BTree:
  /// how wide it is.  Each level of the BTree is guaranteed to have at least
  /// WidthFactor-1 K/V pairs (except the root) and may have at most
  /// 2*WidthFactor-1 K/V pairs.
  enum { WidthFactor = 8 };

  /// Values - This tracks the SourceDelta's currently in this node.
  SourceDelta Values[2 * WidthFactor - 1];

  /// NumValuesUsed - This tracks the number of values this node currently
  /// holds.
  unsigned char NumValuesUsed = 0;

  /// IsLeaf - This is true if this is a leaf of the btree.  If false, this is
  /// an interior node, and is actually an instance of DeltaTreeInteriorNode.
  bool IsLeaf;

  /// FullDelta - This is the full delta of all the values in this node and
  /// all children nodes.
  int FullDelta = 0;

public:
  DeltaTreeNode(bool isLeaf = true) : IsLeaf(isLeaf) {}

  bool isLeaf() const { return IsLeaf; }
  int getFullDelta() const { return FullDelta; }
  bool isFull() const { return NumValuesUsed == 2 * WidthFactor - 1; }

  unsigned getNumValuesUsed() const { return NumValuesUsed; }

  const SourceDelta &getValue(unsigned i) const {
    assert(i < NumValuesUsed && "Invalid value #");
    return Values[i];
  }

  SourceDelta &getValue(unsigned i) {
    assert(i < NumValuesUsed && "Invalid value #");
    return Values[i];
  }

  /// DoInsertion - Do an insertion of the specified FileIndex/Delta pair into
  /// this node.  If insertion is easy, do it and return false.  Otherwise,
  /// split the node, populate InsertRes with info about the split, and return
  /// true.
  bool DoInsertion(unsigned FileIndex, int Delta, InsertResult *InsertRes);

  void DoSplit(InsertResult &InsertRes);

  /// RecomputeFullDeltaLocally - Recompute the FullDelta field by doing a
  /// local walk over our contained deltas.
  void RecomputeFullDeltaLocally();

  void Destroy();
};

/// DeltaTreeInteriorNode - When isLeaf = false, a node has child pointers.
/// This class tracks them.
```
- EN: This section centers on `DeltaTreeNode`, `isLeaf`, `getFullDelta` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `DeltaTreeNode`, `isLeaf`, `getFullDelta` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code propagates LLVM-style errors and invariants and returns the resulting value to its callers.
  CN: 在这一段中，代码传播 LLVM 风格的错误处理并维护不变量，并将结果返回给调用方。

### Lines 121-180

```cpp
class DeltaTreeInteriorNode : public DeltaTreeNode {
  friend class DeltaTreeNode;

  DeltaTreeNode *Children[2 * WidthFactor];

  ~DeltaTreeInteriorNode() {
    for (unsigned i = 0, e = NumValuesUsed + 1; i != e; ++i)
      Children[i]->Destroy();
  }

public:
  DeltaTreeInteriorNode() : DeltaTreeNode(false /*nonleaf*/) {}

  DeltaTreeInteriorNode(const InsertResult &IR)
      : DeltaTreeNode(false /*nonleaf*/) {
    Children[0] = IR.LHS;
    Children[1] = IR.RHS;
    Values[0] = IR.Split;
    FullDelta =
        IR.LHS->getFullDelta() + IR.RHS->getFullDelta() + IR.Split.Delta;
    NumValuesUsed = 1;
  }

  const DeltaTreeNode *getChild(unsigned i) const {
    assert(i < getNumValuesUsed() + 1 && "Invalid child");
    return Children[i];
  }

  DeltaTreeNode *getChild(unsigned i) {
    assert(i < getNumValuesUsed() + 1 && "Invalid child");
    return Children[i];
  }

  static bool classof(const DeltaTreeNode *N) { return !N->isLeaf(); }
};

} // namespace

/// Destroy - A 'virtual' destructor.
void DeltaTreeNode::Destroy() {
  if (isLeaf())
    delete this;
  else
    delete cast<DeltaTreeInteriorNode>(this);
}

/// RecomputeFullDeltaLocally - Recompute the FullDelta field by doing a
/// local walk over our contained deltas.
void DeltaTreeNode::RecomputeFullDeltaLocally() {
  int NewFullDelta = 0;
  for (unsigned i = 0, e = getNumValuesUsed(); i != e; ++i)
    NewFullDelta += Values[i].Delta;
  if (auto *IN = dyn_cast<DeltaTreeInteriorNode>(this))
    for (unsigned i = 0, e = getNumValuesUsed() + 1; i != e; ++i)
      NewFullDelta += IN->getChild(i)->getFullDelta();
  FullDelta = NewFullDelta;
}

/// DoInsertion - Do an insertion of the specified FileIndex/Delta pair into
/// this node.  If insertion is easy, do it and return false.  Otherwise,
```
- EN: This section centers on `~DeltaTreeInteriorNode`, `DeltaTreeInteriorNode`, `assert` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `~DeltaTreeInteriorNode`, `DeltaTreeInteriorNode`, `assert` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 181-240

```cpp
/// split the node, populate InsertRes with info about the split, and return
/// true.
bool DeltaTreeNode::DoInsertion(unsigned FileIndex, int Delta,
                                InsertResult *InsertRes) {
  // Maintain full delta for this node.
  FullDelta += Delta;

  // Find the insertion point, the first delta whose index is >= FileIndex.
  unsigned i = 0, e = getNumValuesUsed();
  while (i != e && FileIndex > getValue(i).FileLoc)
    ++i;

  // If we found an a record for exactly this file index, just merge this
  // value into the pre-existing record and finish early.
  if (i != e && getValue(i).FileLoc == FileIndex) {
    // NOTE: Delta could drop to zero here.  This means that the delta entry is
    // useless and could be removed.  Supporting erases is more complex than
    // leaving an entry with Delta=0, so we just leave an entry with Delta=0 in
    // the tree.
    Values[i].Delta += Delta;
    return false;
  }

  // Otherwise, we found an insertion point, and we know that the value at the
  // specified index is > FileIndex.  Handle the leaf case first.
  if (isLeaf()) {
    if (!isFull()) {
      // For an insertion into a non-full leaf node, just insert the value in
      // its sorted position.  This requires moving later values over.
      if (i != e)
        memmove(&Values[i + 1], &Values[i], sizeof(Values[0]) * (e - i));
      Values[i] = SourceDelta::get(FileIndex, Delta);
      ++NumValuesUsed;
      return false;
    }

    // Otherwise, if this is leaf is full, split the node at its median, insert
    // the value into one of the children, and return the result.
    assert(InsertRes && "No result location specified");
    DoSplit(*InsertRes);

    if (InsertRes->Split.FileLoc > FileIndex)
      InsertRes->LHS->DoInsertion(FileIndex, Delta, nullptr /*can't fail*/);
    else
      InsertRes->RHS->DoInsertion(FileIndex, Delta, nullptr /*can't fail*/);
    return true;
  }

  // Otherwise, this is an interior node.  Send the request down the tree.
  auto *IN = cast<DeltaTreeInteriorNode>(this);
  if (!IN->Children[i]->DoInsertion(FileIndex, Delta, InsertRes))
    return false; // If there was space in the child, just return.

  // Okay, this split the subtree, producing a new value and two children to
  // insert here.  If this node is non-full, we can just insert it directly.
  if (!isFull()) {
    // Now that we have two nodes and a new element, insert the perclated value
    // into ourself by moving all the later values/children down, then inserting
    // the new one.
    if (i != e)
```
- EN: This section centers on `DoInsertion`, `assert`, `DoSplit` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `DoInsertion`, `assert`, `DoSplit` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 241-300

```cpp
      memmove(&IN->Children[i + 2], &IN->Children[i + 1],
              (e - i) * sizeof(IN->Children[0]));
    IN->Children[i] = InsertRes->LHS;
    IN->Children[i + 1] = InsertRes->RHS;

    if (e != i)
      memmove(&Values[i + 1], &Values[i], (e - i) * sizeof(Values[0]));
    Values[i] = InsertRes->Split;
    ++NumValuesUsed;
    return false;
  }

  // Finally, if this interior node was full and a node is percolated up, split
  // ourself and return that up the chain.  Start by saving all our info to
  // avoid having the split clobber it.
  IN->Children[i] = InsertRes->LHS;
  DeltaTreeNode *SubRHS = InsertRes->RHS;
  SourceDelta SubSplit = InsertRes->Split;

  // Do the split.
  DoSplit(*InsertRes);

  // Figure out where to insert SubRHS/NewSplit.
  DeltaTreeInteriorNode *InsertSide;
  if (SubSplit.FileLoc < InsertRes->Split.FileLoc)
    InsertSide = cast<DeltaTreeInteriorNode>(InsertRes->LHS);
  else
    InsertSide = cast<DeltaTreeInteriorNode>(InsertRes->RHS);

  // We now have a non-empty interior node 'InsertSide' to insert
  // SubRHS/SubSplit into.  Find out where to insert SubSplit.

  // Find the insertion point, the first delta whose index is >SubSplit.FileLoc.
  i = 0;
  e = InsertSide->getNumValuesUsed();
  while (i != e && SubSplit.FileLoc > InsertSide->getValue(i).FileLoc)
    ++i;

  // Now we know that i is the place to insert the split value into.  Insert it
  // and the child right after it.
  if (i != e)
    memmove(&InsertSide->Children[i + 2], &InsertSide->Children[i + 1],
            (e - i) * sizeof(IN->Children[0]));
  InsertSide->Children[i + 1] = SubRHS;

  if (e != i)
    memmove(&InsertSide->Values[i + 1], &InsertSide->Values[i],
            (e - i) * sizeof(Values[0]));
  InsertSide->Values[i] = SubSplit;
  ++InsertSide->NumValuesUsed;
  InsertSide->FullDelta += SubSplit.Delta + SubRHS->getFullDelta();
  return true;
}

/// DoSplit - Split the currently full node (which has 2*WidthFactor-1 values)
/// into two subtrees each with "WidthFactor-1" values and a pivot value.
/// Return the pieces in InsertRes.
void DeltaTreeNode::DoSplit(InsertResult &InsertRes) {
  assert(isFull() && "Why split a non-full node?");

```
- EN: This section centers on `memmove`, `DoSplit`, `assert` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `memmove`, `DoSplit`, `assert` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 301-360

```cpp
  // Since this node is full, it contains 2*WidthFactor-1 values.  We move
  // the first 'WidthFactor-1' values to the LHS child (which we leave in this
  // node), propagate one value up, and move the last 'WidthFactor-1' values
  // into the RHS child.

  // Create the new child node.
  DeltaTreeNode *NewNode;
  if (auto *IN = dyn_cast<DeltaTreeInteriorNode>(this)) {
    // If this is an interior node, also move over 'WidthFactor' children
    // into the new node.
    DeltaTreeInteriorNode *New = new DeltaTreeInteriorNode();
    memcpy(&New->Children[0], &IN->Children[WidthFactor],
           WidthFactor * sizeof(IN->Children[0]));
    NewNode = New;
  } else {
    // Just create the new leaf node.
    NewNode = new DeltaTreeNode();
  }

  // Move over the last 'WidthFactor-1' values from here to NewNode.
  memcpy(&NewNode->Values[0], &Values[WidthFactor],
         (WidthFactor - 1) * sizeof(Values[0]));

  // Decrease the number of values in the two nodes.
  NewNode->NumValuesUsed = NumValuesUsed = WidthFactor - 1;

  // Recompute the two nodes' full delta.
  NewNode->RecomputeFullDeltaLocally();
  RecomputeFullDeltaLocally();

  InsertRes.LHS = this;
  InsertRes.RHS = NewNode;
  InsertRes.Split = Values[WidthFactor - 1];
}

//===----------------------------------------------------------------------===//
//                        DeltaTree Implementation
//===----------------------------------------------------------------------===//

// #define VERIFY_TREE

#ifdef VERIFY_TREE
/// VerifyTree - Walk the btree performing assertions on various properties to
/// verify consistency.  This is useful for debugging new changes to the tree.
static void VerifyTree(const DeltaTreeNode *N) {
  const auto *IN = dyn_cast<DeltaTreeInteriorNode>(N);
  if (IN == 0) {
    // Verify leaves, just ensure that FullDelta matches up and the elements
    // are in proper order.
    int FullDelta = 0;
    for (unsigned i = 0, e = N->getNumValuesUsed(); i != e; ++i) {
      if (i)
        assert(N->getValue(i - 1).FileLoc < N->getValue(i).FileLoc);
      FullDelta += N->getValue(i).Delta;
    }
    assert(FullDelta == N->getFullDelta());
    return;
  }

  // Verify interior nodes: Ensure that FullDelta matches up and the
```
- EN: This section centers on `memcpy`, `RecomputeFullDeltaLocally`, `VerifyTree` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `memcpy`, `RecomputeFullDeltaLocally`, `VerifyTree` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 361-420

```cpp
  // elements are in proper order and the children are in proper order.
  int FullDelta = 0;
  for (unsigned i = 0, e = IN->getNumValuesUsed(); i != e; ++i) {
    const SourceDelta &IVal = N->getValue(i);
    const DeltaTreeNode *IChild = IN->getChild(i);
    if (i)
      assert(IN->getValue(i - 1).FileLoc < IVal.FileLoc);
    FullDelta += IVal.Delta;
    FullDelta += IChild->getFullDelta();

    // The largest value in child #i should be smaller than FileLoc.
    assert(IChild->getValue(IChild->getNumValuesUsed() - 1).FileLoc <
           IVal.FileLoc);

    // The smallest value in child #i+1 should be larger than FileLoc.
    assert(IN->getChild(i + 1)->getValue(0).FileLoc > IVal.FileLoc);
    VerifyTree(IChild);
  }

  FullDelta += IN->getChild(IN->getNumValuesUsed())->getFullDelta();

  assert(FullDelta == N->getFullDelta());
}
#endif // VERIFY_TREE

static DeltaTreeNode *getRoot(void *Root) { return (DeltaTreeNode *)Root; }

DeltaTree::DeltaTree() { Root = new DeltaTreeNode(); }

DeltaTree::DeltaTree(const DeltaTree &RHS) {
  // Currently we only support copying when the RHS is empty.
  assert(getRoot(RHS.Root)->getNumValuesUsed() == 0 &&
         "Can only copy empty tree");
  Root = new DeltaTreeNode();
}

DeltaTree::~DeltaTree() { getRoot(Root)->Destroy(); }

/// getDeltaAt - Return the accumulated delta at the specified file offset.
/// This includes all insertions or delections that occurred *before* the
/// specified file index.
int DeltaTree::getDeltaAt(unsigned FileIndex) const {
  const DeltaTreeNode *Node = getRoot(Root);

  int Result = 0;

  // Walk down the tree.
  while (true) {
    // For all nodes, include any local deltas before the specified file
    // index by summing them up directly.  Keep track of how many were
    // included.
    unsigned NumValsGreater = 0;
    for (unsigned e = Node->getNumValuesUsed(); NumValsGreater != e;
         ++NumValsGreater) {
      const SourceDelta &Val = Node->getValue(NumValsGreater);

      if (Val.FileLoc >= FileIndex)
        break;
      Result += Val.Delta;
    }
```
- EN: This section centers on `assert`, `VerifyTree`, `DeltaTree` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `assert`, `VerifyTree`, `DeltaTree` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 421-465

```cpp

    // If we have an interior node, include information about children and
    // recurse.  Otherwise, if we have a leaf, we're done.
    const auto *IN = dyn_cast<DeltaTreeInteriorNode>(Node);
    if (!IN)
      return Result;

    // Include any children to the left of the values we skipped, all of
    // their deltas should be included as well.
    for (unsigned i = 0; i != NumValsGreater; ++i)
      Result += IN->getChild(i)->getFullDelta();

    // If we found exactly the value we were looking for, break off the
    // search early.  There is no need to search the RHS of the value for
    // partial results.
    if (NumValsGreater != Node->getNumValuesUsed() &&
        Node->getValue(NumValsGreater).FileLoc == FileIndex)
      return Result + IN->getChild(NumValsGreater)->getFullDelta();

    // Otherwise, traverse down the tree.  The selected subtree may be
    // partially included in the range.
    Node = IN->getChild(NumValsGreater);
  }
  // NOT REACHED.
}

/// AddDelta - When a change is made that shifts around the text buffer,
/// this method is used to record that info.  It inserts a delta of 'Delta'
/// into the current DeltaTree at offset FileIndex.
void DeltaTree::AddDelta(unsigned FileIndex, int Delta) {
  assert(Delta && "Adding a noop?");
  DeltaTreeNode *MyRoot = getRoot(Root);

  DeltaTreeNode::InsertResult InsertRes;
  if (MyRoot->DoInsertion(FileIndex, Delta, &InsertRes)) {
    Root = new DeltaTreeInteriorNode(InsertRes);
#ifdef VERIFY_TREE
    MyRoot = Root;
#endif
  }

#ifdef VERIFY_TREE
  VerifyTree(MyRoot);
#endif
}
```
- EN: This section centers on `AddDelta`, `assert`, `VerifyTree` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `AddDelta`, `assert`, `VerifyTree` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `is`, `SourceDelta`, `DeltaTreeNode`, `get`, `isLeaf`, `getFullDelta` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/ADT/DeltaTree.h`, `llvm/Support/Casting.h`
- Standard library / 标准库: `cstring`
- Other/system headers / 其他或系统头文件: `cassert`
- Related symbols / 相关符号: `is`, `SourceDelta`, `DeltaTreeNode`, `InsertResult`, `DeltaTreeInteriorNode`, `get`, `isLeaf`, `getFullDelta`, `isFull`
