# FoldingSet.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/FoldingSet.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements a hash set that can be used to remove duplication of nodes in a graph.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `FoldingSet` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60

```cpp
//===-- Support/FoldingSet.cpp - Uniquing Hash Set --------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements a hash set that can be used to remove duplication of
// nodes in a graph.
//
//===----------------------------------------------------------------------===//

#include "llvm/ADT/FoldingSet.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/Support/Allocator.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/MathExtras.h"
#include "llvm/Support/SwapByteOrder.h"
#include <cassert>
#include <cstring>
using namespace llvm;

//===----------------------------------------------------------------------===//
// FoldingSetNodeIDRef Implementation

bool FoldingSetNodeIDRef::operator==(FoldingSetNodeIDRef RHS) const {
  if (Size != RHS.Size) return false;
  return memcmp(Data, RHS.Data, Size*sizeof(*Data)) == 0;
}

/// Used to compare the "ordering" of two nodes as defined by the
/// profiled bits and their ordering defined by memcmp().
bool FoldingSetNodeIDRef::operator<(FoldingSetNodeIDRef RHS) const {
  if (Size != RHS.Size)
    return Size < RHS.Size;
  return memcmp(Data, RHS.Data, Size*sizeof(*Data)) < 0;
}

//===----------------------------------------------------------------------===//
// FoldingSetNodeID Implementation

/// Add* - Add various data types to Bit data.
///
void FoldingSetNodeID::AddString(StringRef String) {
  unsigned Size =  String.size();

  unsigned NumInserts = 1 + divideCeil(Size, 4);
  Bits.reserve(Bits.size() + NumInserts);

  Bits.push_back(Size);
  if (!Size) return;

  unsigned Units = Size / 4;
  unsigned Pos = 0;
  const unsigned *Base = (const unsigned*) String.data();

  // If the string is aligned do a bulk transfer.
  if (!((intptr_t)Base & 3)) {
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 9 direct dependencies, including `llvm/ADT/FoldingSet.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/StringRef.h`, `llvm/Support/Allocator.h`.
  CN: 引入了 9 个直接依赖，其中包括 `llvm/ADT/FoldingSet.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/StringRef.h`, `llvm/Support/Allocator.h`。
- EN: This section centers on `AddString` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `AddString` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 61-120

```cpp
    Bits.append(Base, Base + Units);
    Pos = (Units + 1) * 4;
  } else {
    // Otherwise do it the hard way.
    // To be compatible with above bulk transfer, we need to take endianness
    // into account.
    static_assert(sys::IsBigEndianHost || sys::IsLittleEndianHost,
                  "Unexpected host endianness");
    if (sys::IsBigEndianHost) {
      for (Pos += 4; Pos <= Size; Pos += 4) {
        unsigned V = ((unsigned char)String[Pos - 4] << 24) |
                     ((unsigned char)String[Pos - 3] << 16) |
                     ((unsigned char)String[Pos - 2] << 8) |
                      (unsigned char)String[Pos - 1];
        Bits.push_back(V);
      }
    } else {  // Little-endian host
      for (Pos += 4; Pos <= Size; Pos += 4) {
        unsigned V = ((unsigned char)String[Pos - 1] << 24) |
                     ((unsigned char)String[Pos - 2] << 16) |
                     ((unsigned char)String[Pos - 3] << 8) |
                      (unsigned char)String[Pos - 4];
        Bits.push_back(V);
      }
    }
  }

  // With the leftover bits.
  unsigned V = 0;
  // Pos will have overshot size by 4 - #bytes left over.
  // No need to take endianness into account here - this is always executed.
  switch (Pos - Size) {
  case 1: V = (V << 8) | (unsigned char)String[Size - 3]; [[fallthrough]];
  case 2: V = (V << 8) | (unsigned char)String[Size - 2]; [[fallthrough]];
  case 3: V = (V << 8) | (unsigned char)String[Size - 1]; break;
  default: return; // Nothing left.
  }

  Bits.push_back(V);
}

// AddNodeID - Adds the Bit data of another ID to *this.
void FoldingSetNodeID::AddNodeID(const FoldingSetNodeID &ID) {
  Bits.append(ID.Bits.begin(), ID.Bits.end());
}

/// operator== - Used to compare two nodes to each other.
///
bool FoldingSetNodeID::operator==(const FoldingSetNodeID &RHS) const {
  return *this == FoldingSetNodeIDRef(RHS.Bits.data(), RHS.Bits.size());
}

/// operator== - Used to compare two nodes to each other.
///
bool FoldingSetNodeID::operator==(FoldingSetNodeIDRef RHS) const {
  return FoldingSetNodeIDRef(Bits.data(), Bits.size()) == RHS;
}

/// Used to compare the "ordering" of two nodes as defined by the
/// profiled bits and their ordering defined by memcmp().
```
- EN: This section centers on `static_assert`, `AddNodeID` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `static_assert`, `AddNodeID` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and iterates over collections, ranges, or records.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并遍历集合、区间或记录。

### Lines 121-180

```cpp
bool FoldingSetNodeID::operator<(const FoldingSetNodeID &RHS) const {
  return *this < FoldingSetNodeIDRef(RHS.Bits.data(), RHS.Bits.size());
}

bool FoldingSetNodeID::operator<(FoldingSetNodeIDRef RHS) const {
  return FoldingSetNodeIDRef(Bits.data(), Bits.size()) < RHS;
}

/// Intern - Copy this node's data to a memory region allocated from the
/// given allocator and return a FoldingSetNodeIDRef describing the
/// interned data.
FoldingSetNodeIDRef
FoldingSetNodeID::Intern(BumpPtrAllocator &Allocator) const {
  unsigned *New = Allocator.Allocate<unsigned>(Bits.size());
  llvm::uninitialized_copy(Bits, New);
  return FoldingSetNodeIDRef(New, Bits.size());
}

//===----------------------------------------------------------------------===//
/// Helper functions for FoldingSetBase.

/// GetNextPtr - In order to save space, each bucket is a
/// singly-linked-list. In order to make deletion more efficient, we make
/// the list circular, so we can delete a node without computing its hash.
/// The problem with this is that the start of the hash buckets are not
/// Nodes. If NextInBucketPtr is a bucket pointer, this method returns null:
/// use GetBucketPtr when this happens.
static FoldingSetBase::Node *GetNextPtr(void *NextInBucketPtr) {
  // The low bit is set if this is the pointer back to the bucket.
  if (reinterpret_cast<intptr_t>(NextInBucketPtr) & 1)
    return nullptr;

  return static_cast<FoldingSetBase::Node*>(NextInBucketPtr);
}


/// testing.
static void **GetBucketPtr(void *NextInBucketPtr) {
  intptr_t Ptr = reinterpret_cast<intptr_t>(NextInBucketPtr);
  assert((Ptr & 1) && "Not a bucket pointer");
  return reinterpret_cast<void**>(Ptr & ~intptr_t(1));
}

/// GetBucketFor - Hash the specified node ID and return the hash bucket for
/// the specified ID.
static void **GetBucketFor(unsigned Hash, void **Buckets, unsigned NumBuckets) {
  // NumBuckets is always a power of 2.
  unsigned BucketNum = Hash & (NumBuckets-1);
  return Buckets + BucketNum;
}

/// AllocateBuckets - Allocated initialized bucket memory.
static void **AllocateBuckets(unsigned NumBuckets) {
  void **Buckets = static_cast<void**>(safe_calloc(NumBuckets + 1,
                                                   sizeof(void*)));
  // Set the very last bucket to be a non-null "pointer".
  Buckets[NumBuckets] = reinterpret_cast<void*>(-1);
  return Buckets;
}

```
- EN: This section centers on `FoldingSetNodeIDRef`, `Intern`, `uninitialized_copy` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `FoldingSetNodeIDRef`, `Intern`, `uninitialized_copy` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 181-240

```cpp
//===----------------------------------------------------------------------===//
// FoldingSetBase Implementation

FoldingSetBase::FoldingSetBase(unsigned Log2InitSize) {
  assert(5 < Log2InitSize && Log2InitSize < 32 &&
         "Initial hash table size out of range");
  NumBuckets = 1 << Log2InitSize;
  Buckets = AllocateBuckets(NumBuckets);
  NumNodes = 0;
}

FoldingSetBase::FoldingSetBase(FoldingSetBase &&Arg)
    : Buckets(Arg.Buckets), NumBuckets(Arg.NumBuckets), NumNodes(Arg.NumNodes) {
  Arg.Buckets = nullptr;
  Arg.NumBuckets = 0;
  Arg.NumNodes = 0;
}

FoldingSetBase &FoldingSetBase::operator=(FoldingSetBase &&RHS) {
  free(Buckets); // This may be null if the set is in a moved-from state.
  Buckets = RHS.Buckets;
  NumBuckets = RHS.NumBuckets;
  NumNodes = RHS.NumNodes;
  RHS.Buckets = nullptr;
  RHS.NumBuckets = 0;
  RHS.NumNodes = 0;
  return *this;
}

FoldingSetBase::~FoldingSetBase() {
  free(Buckets);
}

void FoldingSetBase::clear() {
  // Set all but the last bucket to null pointers.
  memset(Buckets, 0, NumBuckets*sizeof(void*));

  // Set the very last bucket to be a non-null "pointer".
  Buckets[NumBuckets] = reinterpret_cast<void*>(-1);

  // Reset the node count to zero.
  NumNodes = 0;
}

void FoldingSetBase::GrowBucketCount(unsigned NewBucketCount,
                                     const FoldingSetInfo &Info) {
  assert((NewBucketCount > NumBuckets) &&
         "Can't shrink a folding set with GrowBucketCount");
  assert(isPowerOf2_32(NewBucketCount) && "Bad bucket count!");
  void **OldBuckets = Buckets;
  unsigned OldNumBuckets = NumBuckets;

  // Clear out new buckets.
  Buckets = AllocateBuckets(NewBucketCount);
  // Set NumBuckets only if allocation of new buckets was successful.
  NumBuckets = NewBucketCount;
  NumNodes = 0;

  // Walk the old buckets, rehashing nodes into their new place.
  FoldingSetNodeID TempID;
```
- EN: This section centers on `FoldingSetBase`, `assert`, `free` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `FoldingSetBase`, `assert`, `free` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code propagates LLVM-style errors and invariants and returns the resulting value to its callers.
  CN: 在这一段中，代码传播 LLVM 风格的错误处理并维护不变量，并将结果返回给调用方。

### Lines 241-300

```cpp
  for (unsigned i = 0; i != OldNumBuckets; ++i) {
    void *Probe = OldBuckets[i];
    if (!Probe) continue;
    while (Node *NodeInBucket = GetNextPtr(Probe)) {
      // Figure out the next link, remove NodeInBucket from the old link.
      Probe = NodeInBucket->getNextInBucket();
      NodeInBucket->SetNextInBucket(nullptr);

      // Insert the node into the new bucket, after recomputing the hash.
      InsertNode(NodeInBucket,
                 GetBucketFor(Info.ComputeNodeHash(this, NodeInBucket, TempID),
                              Buckets, NumBuckets),
                 Info);
      TempID.clear();
    }
  }

  free(OldBuckets);
}

/// GrowHashTable - Double the size of the hash table and rehash everything.
///
void FoldingSetBase::GrowHashTable(const FoldingSetInfo &Info) {
  GrowBucketCount(NumBuckets * 2, Info);
}

void FoldingSetBase::reserve(unsigned EltCount, const FoldingSetInfo &Info) {
  // This will give us somewhere between EltCount / 2 and
  // EltCount buckets.  This puts us in the load factor
  // range of 1.0 - 2.0.
  if(EltCount < capacity())
    return;
  GrowBucketCount(llvm::bit_floor(EltCount), Info);
}

/// FindNodeOrInsertPos - Look up the node specified by ID.  If it exists,
/// return it.  If not, return the insertion token that will make insertion
/// faster.
FoldingSetBase::Node *FoldingSetBase::FindNodeOrInsertPos(
    const FoldingSetNodeID &ID, void *&InsertPos, const FoldingSetInfo &Info) {
  unsigned IDHash = ID.ComputeHash();
  void **Bucket = GetBucketFor(IDHash, Buckets, NumBuckets);
  void *Probe = *Bucket;

  InsertPos = nullptr;

  FoldingSetNodeID TempID;
  while (Node *NodeInBucket = GetNextPtr(Probe)) {
    if (Info.NodeEquals(this, NodeInBucket, ID, IDHash, TempID))
      return NodeInBucket;
    TempID.clear();

    Probe = NodeInBucket->getNextInBucket();
  }

  // Didn't find the node, return null with the bucket as the InsertPos.
  InsertPos = Bucket;
  return nullptr;
}

```
- EN: This section centers on `InsertNode`, `free`, `GrowHashTable` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `InsertNode`, `free`, `GrowHashTable` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 301-360

```cpp
/// InsertNode - Insert the specified node into the folding set, knowing that it
/// is not already in the map.  InsertPos must be obtained from
/// FindNodeOrInsertPos.
void FoldingSetBase::InsertNode(Node *N, void *InsertPos,
                                const FoldingSetInfo &Info) {
  assert(!N->getNextInBucket());
  // Do we need to grow the hashtable?
  if (NumNodes+1 > capacity()) {
    GrowHashTable(Info);
    FoldingSetNodeID TempID;
    InsertPos = GetBucketFor(Info.ComputeNodeHash(this, N, TempID), Buckets,
                             NumBuckets);
  }

  ++NumNodes;

  /// The insert position is actually a bucket pointer.
  void **Bucket = static_cast<void**>(InsertPos);

  void *Next = *Bucket;

  // If this is the first insertion into this bucket, its next pointer will be
  // null.  Pretend as if it pointed to itself, setting the low bit to indicate
  // that it is a pointer to the bucket.
  if (!Next)
    Next = reinterpret_cast<void*>(reinterpret_cast<intptr_t>(Bucket)|1);

  // Set the node's next pointer, and make the bucket point to the node.
  N->SetNextInBucket(Next);
  *Bucket = N;
}

/// RemoveNode - Remove a node from the folding set, returning true if one was
/// removed or false if the node was not in the folding set.
bool FoldingSetBase::RemoveNode(Node *N) {
  // Because each bucket is a circular list, we don't need to compute N's hash
  // to remove it.
  void *Ptr = N->getNextInBucket();
  if (!Ptr) return false;  // Not in folding set.

  --NumNodes;
  N->SetNextInBucket(nullptr);

  // Remember what N originally pointed to, either a bucket or another node.
  void *NodeNextPtr = Ptr;

  // Chase around the list until we find the node (or bucket) which points to N.
  while (true) {
    if (Node *NodeInBucket = GetNextPtr(Ptr)) {
      // Advance pointer.
      Ptr = NodeInBucket->getNextInBucket();

      // We found a node that points to N, change it to point to N's next node,
      // removing N from the list.
      if (Ptr == N) {
        NodeInBucket->SetNextInBucket(NodeNextPtr);
        return true;
      }
    } else {
      void **Bucket = GetBucketPtr(Ptr);
```
- EN: This section centers on `InsertNode`, `assert`, `GrowHashTable` and performs utility computation and state updates.
  CN: 这一段主要围绕 `InsertNode`, `assert`, `GrowHashTable` 等符号展开，负责执行工具性计算并更新状态。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 361-420

```cpp
      Ptr = *Bucket;

      // If we found that the bucket points to N, update the bucket to point to
      // whatever is next.
      if (Ptr == N) {
        *Bucket = NodeNextPtr;
        return true;
      }
    }
  }
}

/// GetOrInsertNode - If there is an existing simple Node exactly
/// equal to the specified node, return it.  Otherwise, insert 'N' and it
/// instead.
FoldingSetBase::Node *
FoldingSetBase::GetOrInsertNode(FoldingSetBase::Node *N,
                                const FoldingSetInfo &Info) {
  FoldingSetNodeID ID;
  Info.GetNodeProfile(this, N, ID);
  void *IP;
  if (Node *E = FindNodeOrInsertPos(ID, IP, Info))
    return E;
  InsertNode(N, IP, Info);
  return N;
}

//===----------------------------------------------------------------------===//
// FoldingSetIteratorImpl Implementation

FoldingSetIteratorImpl::FoldingSetIteratorImpl(void **Bucket) {
  // Skip to the first non-null non-self-cycle bucket.
  while (*Bucket != reinterpret_cast<void*>(-1) &&
         (!*Bucket || !GetNextPtr(*Bucket)))
    ++Bucket;

  NodePtr = static_cast<FoldingSetNode*>(*Bucket);
}

void FoldingSetIteratorImpl::advance() {
  // If there is another link within this bucket, go to it.
  void *Probe = NodePtr->getNextInBucket();

  if (FoldingSetNode *NextNodeInBucket = GetNextPtr(Probe))
    NodePtr = NextNodeInBucket;
  else {
    // Otherwise, this is the last link in this bucket.
    void **Bucket = GetBucketPtr(Probe);

    // Skip to the next non-null non-self-cycle bucket.
    do {
      ++Bucket;
    } while (*Bucket != reinterpret_cast<void*>(-1) &&
             (!*Bucket || !GetNextPtr(*Bucket)));

    NodePtr = static_cast<FoldingSetNode*>(*Bucket);
  }
}

//===----------------------------------------------------------------------===//
```
- EN: This section centers on `GetOrInsertNode`, `InsertNode`, `FoldingSetIteratorImpl` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `GetOrInsertNode`, `InsertNode`, `FoldingSetIteratorImpl` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 421-425

```cpp
// FoldingSetBucketIteratorImpl Implementation

FoldingSetBucketIteratorImpl::FoldingSetBucketIteratorImpl(void **Bucket) {
  Ptr = (!*Bucket || !GetNextPtr(*Bucket)) ? (void*) Bucket : *Bucket;
}
```
- EN: This section centers on `FoldingSetBucketIteratorImpl` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `FoldingSetBucketIteratorImpl` 等符号展开，负责实现局部控制流程与状态维护。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `AddString`, `static_assert`, `AddNodeID`, `FoldingSetNodeIDRef` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/ADT/FoldingSet.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/StringRef.h`, `llvm/Support/Allocator.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/MathExtras.h`, `llvm/Support/SwapByteOrder.h`
- Standard library / 标准库: `cstring`
- Other/system headers / 其他或系统头文件: `cassert`
- Related symbols / 相关符号: `AddString`, `static_assert`, `AddNodeID`, `FoldingSetNodeIDRef`, `Intern`
