# StringMap.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/StringMap.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements the StringMap class.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `StringMap` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===--- StringMap.cpp - String Hash table map implementation -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the StringMap class.
//
//===----------------------------------------------------------------------===//

#include "llvm/ADT/StringMap.h"
#include "llvm/Support/MathExtras.h"
#include "llvm/Support/ReverseIteration.h"
#include "llvm/Support/xxhash.h"

using namespace llvm;

/// Returns the number of buckets to allocate to ensure that the DenseMap can
/// accommodate \p NumEntries without need to grow().
static inline unsigned getMinBucketToReserveForEntries(unsigned NumEntries) {
  // Ensure that "NumEntries * 4 < NumBuckets * 3"
  if (NumEntries == 0)
    return 0;
  // +1 is required because of the strict equality.
  // For example if NumEntries is 48, we need to return 401.
  return NextPowerOf2(NumEntries * 4 / 3 + 1);
}

static inline StringMapEntryBase **createTable(unsigned NewNumBuckets) {
  auto **Table = static_cast<StringMapEntryBase **>(safe_calloc(
      NewNumBuckets + 1, sizeof(StringMapEntryBase **) + sizeof(unsigned)));

  // Allocate one extra bucket, set it to look filled so the iterators stop at
  // end.
  Table[NewNumBuckets] = (StringMapEntryBase *)2;
  return Table;
}

```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 4 direct dependencies, including `llvm/ADT/StringMap.h`, `llvm/Support/MathExtras.h`, `llvm/Support/ReverseIteration.h`, `llvm/Support/xxhash.h`.
  CN: 引入了 4 个直接依赖，其中包括 `llvm/ADT/StringMap.h`, `llvm/Support/MathExtras.h`, `llvm/Support/ReverseIteration.h`, `llvm/Support/xxhash.h`。
- EN: This section centers on `getMinBucketToReserveForEntries`, `NextPowerOf2` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getMinBucketToReserveForEntries`, `NextPowerOf2` 等符号展开，负责查询现有状态并返回计算出的元数据。

### Lines 41-80

```cpp
static inline unsigned *getHashTable(StringMapEntryBase **TheTable,
                                     unsigned NumBuckets) {
  return reinterpret_cast<unsigned *>(TheTable + NumBuckets + 1);
}

uint32_t StringMapImpl::hash(StringRef Key) { return xxh3_64bits(Key); }

StringMapImpl::StringMapImpl(unsigned InitSize, unsigned itemSize)
    : ItemSize(itemSize) {
  // If a size is specified, initialize the table with that many buckets.
  if (InitSize) {
    // The table will grow when the number of entries reach 3/4 of the number of
    // buckets. To guarantee that "InitSize" number of entries can be inserted
    // in the table without growing, we allocate just what is needed here.
    init(getMinBucketToReserveForEntries(InitSize));
  }
}

void StringMapImpl::init(unsigned InitSize) {
  assert((InitSize & (InitSize - 1)) == 0 &&
         "Init Size must be a power of 2 or zero!");

  unsigned NewNumBuckets = InitSize ? InitSize : 16;
  NumItems = 0;
  NumTombstones = 0;

  TheTable = createTable(NewNumBuckets);

  // Set the member only if TheTable was successfully allocated
  NumBuckets = NewNumBuckets;
}

/// LookupBucketFor - Look up the bucket that the specified string should end
/// up in.  If it already exists as a key in the map, the Item pointer for the
/// specified bucket will be non-null.  Otherwise, it will be null.  In either
/// case, the FullHashValue field of the bucket will be set to the hash value
/// of the string.
unsigned StringMapImpl::LookupBucketFor(StringRef Name,
                                        uint32_t FullHashValue) {
#ifdef EXPENSIVE_CHECKS
```
- EN: This section centers on `hash`, `StringMapImpl`, `init` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `hash`, `StringMapImpl`, `init` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 81-120

```cpp
  assert(FullHashValue == hash(Name));
#endif
  // Hash table unallocated so far?
  if (NumBuckets == 0)
    init(16);
  if constexpr (shouldReverseIterate())
    FullHashValue = ~FullHashValue;
  unsigned BucketNo = FullHashValue & (NumBuckets - 1);
  unsigned *HashTable = getHashTable(TheTable, NumBuckets);

  unsigned ProbeAmt = 1;
  int FirstTombstone = -1;
  while (true) {
    StringMapEntryBase *BucketItem = TheTable[BucketNo];
    // If we found an empty bucket, this key isn't in the table yet, return it.
    if (LLVM_LIKELY(!BucketItem)) {
      // If we found a tombstone, we want to reuse the tombstone instead of an
      // empty bucket.  This reduces probing.
      if (FirstTombstone != -1) {
        HashTable[FirstTombstone] = FullHashValue;
        return FirstTombstone;
      }

      HashTable[BucketNo] = FullHashValue;
      return BucketNo;
    }

    if (BucketItem == getTombstoneVal()) {
      // Skip over tombstones.  However, remember the first one we see.
      if (FirstTombstone == -1)
        FirstTombstone = BucketNo;
    } else if (LLVM_LIKELY(HashTable[BucketNo] == FullHashValue)) {
      // If the full hash value matches, check deeply for a match.  The common
      // case here is that we are only looking at the buckets (for item info
      // being non-null and for the full hash value) not at the items.  This
      // is important for cache locality.

      // Do the comparison like this because Name isn't necessarily
      // null-terminated!
      char *ItemStr = (char *)BucketItem + ItemSize;
```
- EN: This section centers on `assert` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `assert` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 121-160

```cpp
      if (Name == StringRef(ItemStr, BucketItem->getKeyLength())) {
        // We found a match!
        return BucketNo;
      }
    }

    // Okay, we didn't find the item.  Probe to the next bucket.
    BucketNo = (BucketNo + ProbeAmt) & (NumBuckets - 1);

    // Use quadratic probing, it has fewer clumping artifacts than linear
    // probing and has good cache behavior in the common case.
    ++ProbeAmt;
  }
}

/// FindKey - Look up the bucket that contains the specified key. If it exists
/// in the map, return the bucket number of the key.  Otherwise return -1.
/// This does not modify the map.
int StringMapImpl::FindKey(StringRef Key, uint32_t FullHashValue) const {
  if (NumBuckets == 0)
    return -1; // Really empty table?
#ifdef EXPENSIVE_CHECKS
  assert(FullHashValue == hash(Key));
#endif
  if constexpr (shouldReverseIterate())
    FullHashValue = ~FullHashValue;
  unsigned BucketNo = FullHashValue & (NumBuckets - 1);
  unsigned *HashTable = getHashTable(TheTable, NumBuckets);

  unsigned ProbeAmt = 1;
  while (true) {
    StringMapEntryBase *BucketItem = TheTable[BucketNo];
    // If we found an empty bucket, this key isn't in the table yet, return.
    if (LLVM_LIKELY(!BucketItem))
      return -1;

    if (BucketItem == getTombstoneVal()) {
      // Ignore tombstones.
    } else if (LLVM_LIKELY(HashTable[BucketNo] == FullHashValue)) {
      // If the full hash value matches, check deeply for a match.  The common
```
- EN: This section centers on `FindKey`, `assert` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `FindKey`, `assert` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 161-200

```cpp
      // case here is that we are only looking at the buckets (for item info
      // being non-null and for the full hash value) not at the items.  This
      // is important for cache locality.

      // Do the comparison like this because NameStart isn't necessarily
      // null-terminated!
      char *ItemStr = (char *)BucketItem + ItemSize;
      if (Key == StringRef(ItemStr, BucketItem->getKeyLength())) {
        // We found a match!
        return BucketNo;
      }
    }

    // Okay, we didn't find the item.  Probe to the next bucket.
    BucketNo = (BucketNo + ProbeAmt) & (NumBuckets - 1);

    // Use quadratic probing, it has fewer clumping artifacts than linear
    // probing and has good cache behavior in the common case.
    ++ProbeAmt;
  }
}

/// RemoveKey - Remove the specified StringMapEntry from the table, but do not
/// delete it.  This aborts if the value isn't in the table.
void StringMapImpl::RemoveKey(StringMapEntryBase *V) {
  const char *VStr = (char *)V + ItemSize;
  StringMapEntryBase *V2 = RemoveKey(StringRef(VStr, V->getKeyLength()));
  (void)V2;
  assert(V == V2 && "Didn't find key?");
}

/// RemoveKey - Remove the StringMapEntry for the specified key from the
/// table, returning it.  If the key is not in the table, this returns null.
StringMapEntryBase *StringMapImpl::RemoveKey(StringRef Key) {
  int Bucket = FindKey(Key);
  if (Bucket == -1)
    return nullptr;

  StringMapEntryBase *Result = TheTable[Bucket];
  TheTable[Bucket] = getTombstoneVal();
```
- EN: This section centers on `RemoveKey`, `assert` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `RemoveKey`, `assert` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 201-240

```cpp
  --NumItems;
  ++NumTombstones;
  assert(NumItems + NumTombstones <= NumBuckets);

  return Result;
}

/// RehashTable - Grow the table, redistributing values into the buckets with
/// the appropriate mod-of-hashtable-size.
unsigned StringMapImpl::RehashTable(unsigned BucketNo) {
  unsigned NewSize;
  // If the hash table is now more than 3/4 full, or if fewer than 1/8 of
  // the buckets are empty (meaning that many are filled with tombstones),
  // grow/rehash the table.
  if (LLVM_UNLIKELY(NumItems * 4 > NumBuckets * 3)) {
    NewSize = NumBuckets * 2;
  } else if (LLVM_UNLIKELY(NumBuckets - (NumItems + NumTombstones) <=
                           NumBuckets / 8)) {
    NewSize = NumBuckets;
  } else {
    return BucketNo;
  }

  unsigned NewBucketNo = BucketNo;
  auto **NewTableArray = createTable(NewSize);
  unsigned *NewHashArray = getHashTable(NewTableArray, NewSize);
  unsigned *HashTable = getHashTable(TheTable, NumBuckets);

  // Rehash all the items into their new buckets.  Luckily :) we already have
  // the hash values available, so we don't have to rehash any strings.
  for (unsigned I = 0, E = NumBuckets; I != E; ++I) {
    StringMapEntryBase *Bucket = TheTable[I];
    if (Bucket && Bucket != getTombstoneVal()) {
      // If the bucket is not available, probe for a spot.
      unsigned FullHash = HashTable[I];
      unsigned NewBucket = FullHash & (NewSize - 1);
      if (NewTableArray[NewBucket]) {
        unsigned ProbeSize = 1;
        do {
          NewBucket = (NewBucket + ProbeSize++) & (NewSize - 1);
```
- EN: This section centers on `assert`, `RehashTable` and performs utility computation and state updates.
  CN: 这一段主要围绕 `assert`, `RehashTable` 等符号展开，负责执行工具性计算并更新状态。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 241-258

```cpp
        } while (NewTableArray[NewBucket]);
      }

      // Finally found a slot.  Fill it in.
      NewTableArray[NewBucket] = Bucket;
      NewHashArray[NewBucket] = FullHash;
      if (I == BucketNo)
        NewBucketNo = NewBucket;
    }
  }

  free(TheTable);

  TheTable = NewTableArray;
  NumBuckets = NewSize;
  NumTombstones = 0;
  return NewBucketNo;
}
```
- EN: This section centers on `free` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `free` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `getMinBucketToReserveForEntries`, `NextPowerOf2`, `hash`, `StringMapImpl` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/ADT/StringMap.h`, `llvm/Support/MathExtras.h`, `llvm/Support/ReverseIteration.h`, `llvm/Support/xxhash.h`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `getMinBucketToReserveForEntries`, `NextPowerOf2`, `hash`, `StringMapImpl`, `init`
