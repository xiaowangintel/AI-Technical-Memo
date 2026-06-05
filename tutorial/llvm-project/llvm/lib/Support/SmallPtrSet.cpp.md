# SmallPtrSet.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/SmallPtrSet.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements the SmallPtrSet class.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `SmallPtrSet` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===- llvm/ADT/SmallPtrSet.cpp - 'Normally small' pointer set ------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the SmallPtrSet class.  See SmallPtrSet.h for an
// overview of the algorithm.
//
//===----------------------------------------------------------------------===//

#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/ADT/DenseMapInfo.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/Support/MathExtras.h"
#include "llvm/Support/MemAlloc.h"
#include <algorithm>
#include <cassert>
#include <cstdlib>

using namespace llvm;

void SmallPtrSetImplBase::shrink_and_clear() {
  assert(!isSmall() && "Can't shrink a small set!");
  free(CurArray);

  // Reduce the number of buckets.
  unsigned Size = size();
  CurArraySize = Size > 16 ? 1 << (Log2_32_Ceil(Size) + 1) : 32;
  NumEntries = NumTombstones = 0;

  // Install the new array.  Clear all the buckets to empty.
  CurArray = (const void**)safe_malloc(sizeof(void*) * CurArraySize);

  memset(CurArray, -1, CurArraySize*sizeof(void*));
}

std::pair<const void *const *, bool>
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 8 direct dependencies, including `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/DenseMapInfo.h`, `llvm/ADT/STLExtras.h`, `llvm/Support/MathExtras.h`.
  CN: 引入了 8 个直接依赖，其中包括 `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/DenseMapInfo.h`, `llvm/ADT/STLExtras.h`, `llvm/Support/MathExtras.h`。
- EN: This section centers on `shrink_and_clear`, `assert`, `free` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `shrink_and_clear`, `assert`, `free` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 41-80

```cpp
SmallPtrSetImplBase::insert_imp_big(const void *Ptr) {
  if (LLVM_UNLIKELY(size() * 4 >= CurArraySize * 3)) {
    // If more than 3/4 of the array is full, grow.
    Grow(CurArraySize < 64 ? 128 : CurArraySize * 2);
  } else if (LLVM_UNLIKELY(CurArraySize - NumEntries - NumTombstones <
                           CurArraySize / 8)) {
    // If fewer of 1/8 of the array is empty (meaning that many are filled with
    // tombstones), rehash.
    Grow(CurArraySize);
  }

  // Okay, we know we have space.  Find a hash bucket.
  const void **Bucket = const_cast<const void**>(FindBucketFor(Ptr));
  if (*Bucket == Ptr)
    return {Bucket, false}; // Already inserted, good.

  // Otherwise, insert it!
  if (*Bucket == getTombstoneMarker())
    --NumTombstones;
  ++NumEntries;
  *Bucket = Ptr;
  incrementEpoch();
  return {Bucket, true};
}

const void *const *SmallPtrSetImplBase::doFind(const void *Ptr) const {
  unsigned BucketNo =
      DenseMapInfo<void *>::getHashValue(Ptr) & (CurArraySize - 1);
  unsigned ProbeAmt = 1;
  while (true) {
    const void *const *Bucket = CurArray + BucketNo;
    if (LLVM_LIKELY(*Bucket == Ptr))
      return Bucket;
    if (LLVM_LIKELY(*Bucket == getEmptyMarker()))
      return nullptr;

    // Otherwise, it's a hash collision or a tombstone, continue quadratic
    // probing.
    BucketNo += ProbeAmt++;
    BucketNo &= CurArraySize - 1;
```
- EN: This section centers on `insert_imp_big`, `Grow`, `incrementEpoch` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `insert_imp_big`, `Grow`, `incrementEpoch` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 81-120

```cpp
  }
}

const void *const *SmallPtrSetImplBase::FindBucketFor(const void *Ptr) const {
  unsigned Bucket = DenseMapInfo<void *>::getHashValue(Ptr) & (CurArraySize-1);
  unsigned ArraySize = CurArraySize;
  unsigned ProbeAmt = 1;
  const void *const *Array = CurArray;
  const void *const *Tombstone = nullptr;
  while (true) {
    // If we found an empty bucket, the pointer doesn't exist in the set.
    // Return a tombstone if we've seen one so far, or the empty bucket if
    // not.
    if (LLVM_LIKELY(Array[Bucket] == getEmptyMarker()))
      return Tombstone ? Tombstone : Array+Bucket;

    // Found Ptr's bucket?
    if (LLVM_LIKELY(Array[Bucket] == Ptr))
      return Array+Bucket;

    // If this is a tombstone, remember it.  If Ptr ends up not in the set, we
    // prefer to return it than something that would require more probing.
    if (Array[Bucket] == getTombstoneMarker() && !Tombstone)
      Tombstone = Array+Bucket;  // Remember the first tombstone found.

    // It's a hash collision or a tombstone. Reprobe.
    Bucket = (Bucket + ProbeAmt++) & (ArraySize-1);
  }
}

/// Grow - Allocate a larger backing store for the buckets and move it over.
///
void SmallPtrSetImplBase::Grow(unsigned NewSize) {
  auto OldBuckets = buckets();
  bool WasSmall = isSmall();

  // Install the new array.  Clear all the buckets to empty.
  const void **NewBuckets = (const void**) safe_malloc(sizeof(void*) * NewSize);

  // Reset member only if memory was allocated successfully
```
- EN: This section centers on `Grow` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `Grow` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 121-160

```cpp
  CurArray = NewBuckets;
  CurArraySize = NewSize;
  memset(CurArray, -1, NewSize*sizeof(void*));

  // Copy over all valid entries.
  for (const void *&Bucket : OldBuckets) {
    // Copy over the element if it is valid.
    if (Bucket != getTombstoneMarker() && Bucket != getEmptyMarker())
      *const_cast<void **>(FindBucketFor(Bucket)) = const_cast<void *>(Bucket);
  }

  if (!WasSmall)
    free(OldBuckets.begin());
  NumTombstones = 0;
  IsSmall = false;
}

SmallPtrSetImplBase::SmallPtrSetImplBase(const void **SmallStorage,
                                         const SmallPtrSetImplBase &that) {
  IsSmall = that.isSmall();
  if (IsSmall) {
    // If we're becoming small, prepare to insert into our stack space
    CurArray = SmallStorage;
  } else {
    // Otherwise, allocate new heap space (unless we were the same size)
    CurArray = (const void**)safe_malloc(sizeof(void*) * that.CurArraySize);
  }

  // Copy over the that array.
  copyHelper(that);
}

SmallPtrSetImplBase::SmallPtrSetImplBase(const void **SmallStorage,
                                         unsigned SmallSize,
                                         const void **RHSSmallStorage,
                                         SmallPtrSetImplBase &&that) {
  moveHelper(SmallStorage, SmallSize, RHSSmallStorage, std::move(that));
}

void SmallPtrSetImplBase::copyFrom(const void **SmallStorage,
```
- EN: This section centers on `memset`, `SmallPtrSetImplBase`, `copyHelper` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `memset`, `SmallPtrSetImplBase`, `copyHelper` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 161-200

```cpp
                                   const SmallPtrSetImplBase &RHS) {
  assert(&RHS != this && "Self-copy should be handled by the caller.");

  if (isSmall() && RHS.isSmall())
    assert(CurArraySize == RHS.CurArraySize &&
           "Cannot assign sets with different small sizes");

  // If we're becoming small, prepare to insert into our stack space
  if (RHS.isSmall()) {
    if (!isSmall())
      free(CurArray);
    CurArray = SmallStorage;
    IsSmall = true;
    // Otherwise, allocate new heap space (unless we were the same size)
  } else if (CurArraySize != RHS.CurArraySize) {
    if (isSmall())
      CurArray = (const void**)safe_malloc(sizeof(void*) * RHS.CurArraySize);
    else {
      const void **T = (const void**)safe_realloc(CurArray,
                                             sizeof(void*) * RHS.CurArraySize);
      CurArray = T;
    }
    IsSmall = false;
  }

  copyHelper(RHS);
}

void SmallPtrSetImplBase::copyHelper(const SmallPtrSetImplBase &RHS) {
  // Copy over the new array size
  CurArraySize = RHS.CurArraySize;

  // Copy over the contents from the other set
  llvm::copy(RHS.buckets(), CurArray);

  NumEntries = RHS.NumEntries;
  NumTombstones = RHS.NumTombstones;
}

void SmallPtrSetImplBase::moveFrom(const void **SmallStorage,
```
- EN: This section centers on `assert`, `copyHelper`, `copy` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `assert`, `copyHelper`, `copy` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 201-240

```cpp
                                   unsigned SmallSize,
                                   const void **RHSSmallStorage,
                                   SmallPtrSetImplBase &&RHS) {
  if (!isSmall())
    free(CurArray);
  moveHelper(SmallStorage, SmallSize, RHSSmallStorage, std::move(RHS));
}

void SmallPtrSetImplBase::moveHelper(const void **SmallStorage,
                                     unsigned SmallSize,
                                     const void **RHSSmallStorage,
                                     SmallPtrSetImplBase &&RHS) {
  assert(&RHS != this && "Self-move should be handled by the caller.");

  if (RHS.isSmall()) {
    // Copy a small RHS rather than moving.
    CurArray = SmallStorage;
    llvm::copy(RHS.small_buckets(), CurArray);
  } else {
    CurArray = RHS.CurArray;
    RHS.CurArray = RHSSmallStorage;
  }

  // Copy the rest of the trivial members.
  CurArraySize = RHS.CurArraySize;
  NumEntries = RHS.NumEntries;
  NumTombstones = RHS.NumTombstones;
  IsSmall = RHS.IsSmall;

  // Make the RHS small and empty.
  RHS.CurArraySize = SmallSize;
  RHS.NumEntries = 0;
  RHS.NumTombstones = 0;
  RHS.IsSmall = true;
}

void SmallPtrSetImplBase::swap(const void **SmallStorage,
                               const void **RHSSmallStorage,
                               SmallPtrSetImplBase &RHS) {
  if (this == &RHS) return;
```
- EN: This section centers on `moveHelper`, `assert`, `copy` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `moveHelper`, `assert`, `copy` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 241-280

```cpp

  // We can only avoid copying elements if neither set is small.
  if (!this->isSmall() && !RHS.isSmall()) {
    std::swap(this->CurArray, RHS.CurArray);
    std::swap(this->CurArraySize, RHS.CurArraySize);
    std::swap(this->NumEntries, RHS.NumEntries);
    std::swap(this->NumTombstones, RHS.NumTombstones);
    return;
  }

  // FIXME: From here on we assume that both sets have the same small size.

  // Both a small, just swap the small elements.
  if (this->isSmall() && RHS.isSmall()) {
    unsigned MinEntries = std::min(this->NumEntries, RHS.NumEntries);
    std::swap_ranges(this->CurArray, this->CurArray + MinEntries, RHS.CurArray);
    if (this->NumEntries > MinEntries) {
      std::copy(this->CurArray + MinEntries, this->CurArray + this->NumEntries,
                RHS.CurArray + MinEntries);
    } else {
      std::copy(RHS.CurArray + MinEntries, RHS.CurArray + RHS.NumEntries,
                this->CurArray + MinEntries);
    }
    assert(this->CurArraySize == RHS.CurArraySize);
    std::swap(this->NumEntries, RHS.NumEntries);
    std::swap(this->NumTombstones, RHS.NumTombstones);
    return;
  }

  // If only one side is small, copy the small elements into the large side and
  // move the pointer from the large side to the small side.
  SmallPtrSetImplBase &SmallSide = this->isSmall() ? *this : RHS;
  SmallPtrSetImplBase &LargeSide = this->isSmall() ? RHS : *this;
  const void **LargeSideInlineStorage =
      this->isSmall() ? RHSSmallStorage : SmallStorage;
  llvm::copy(SmallSide.small_buckets(), LargeSideInlineStorage);
  std::swap(LargeSide.CurArraySize, SmallSide.CurArraySize);
  std::swap(LargeSide.NumEntries, SmallSide.NumEntries);
  std::swap(LargeSide.NumTombstones, SmallSide.NumTombstones);
  SmallSide.CurArray = LargeSide.CurArray;
```
- EN: This section centers on `swap`, `swap_ranges`, `copy` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `swap`, `swap_ranges`, `copy` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 281-284

```cpp
  SmallSide.IsSmall = false;
  LargeSide.CurArray = LargeSideInlineStorage;
  LargeSide.IsSmall = true;
}
```
- EN: This range contains supporting statements, comments, or structural glue code.
  CN: 这一段主要包含辅助语句、注释或结构性胶水代码。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `shrink_and_clear`, `assert`, `free`, `memset` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/DenseMapInfo.h`, `llvm/ADT/STLExtras.h`, `llvm/Support/MathExtras.h`, `llvm/Support/MemAlloc.h`
- Standard library / 标准库: `algorithm`, `cstdlib`
- Other/system headers / 其他或系统头文件: `cassert`
- Related symbols / 相关符号: `shrink_and_clear`, `assert`, `free`, `memset`, `insert_imp_big`
