# StringTableBuilder.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/StringTableBuilder.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements String table building utility.
  - **CN**: 实现该文件在 LLVM 中对应的子系统逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- StringTableBuilder.cpp - String table building utility -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-21
```cpp

#include "llvm/MC/StringTableBuilder.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/CachedHashString.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/BinaryFormat/COFF.h"
#include "llvm/Support/Endian.h"
#include "llvm/Support/MathExtras.h"
#include "llvm/Support/raw_ostream.h"
#include <cassert>
#include <cstddef>
#include <cstdint>
#include <cstring>
```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/StringTableBuilder.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/CachedHashString.h`, `llvm/ADT/SmallString.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/StringTableBuilder.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/CachedHashString.h`, `llvm/ADT/SmallString.h`。

### Lines 22-28
```cpp
#include <utility>
#include <vector>

using namespace llvm;

StringTableBuilder::~StringTableBuilder() = default;

```
- **EN**: Pulls in the headers needed for this implementation, including `utility`, `vector`.
- **CN**: 引入该实现所需的头文件，其中包括 `utility`, `vector`。

### Lines 29-42
```cpp
void StringTableBuilder::initSize() {
  // Account for leading bytes in table so that offsets returned from add are
  // correct.
  switch (K) {
  case RAW:
  case DWARF:
    Size = 0;
    break;
  case MachOLinked:
  case MachO64Linked:
    Size = 2;
    break;
  case MachO:
  case MachO64:
```
- **EN**: Implements logic around `initSize`; this block uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `initSize` 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 43-55
```cpp
  case ELF:
  case DXContainer:
    // Start the table with a NUL byte.
    Size = 1;
    break;
  case XCOFF:
  case WinCOFF:
    // Make room to write the table size later.
    Size = 4;
    break;
  }
}

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 56-68
```cpp
StringTableBuilder::StringTableBuilder(Kind K, Align Alignment)
    : K(K), Alignment(Alignment) {
  initSize();
}

void StringTableBuilder::write(raw_ostream &OS) const {
  assert(isFinalized());
  SmallString<0> Data;
  Data.resize(getSize());
  write((uint8_t *)Data.data());
  OS << Data;
}

```
- **EN**: Implements logic around `StringTableBuilder`, `K`, `initSize`, `write`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `StringTableBuilder`, `K`, `initSize`, `write`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 69-82
```cpp
using StringPair = std::pair<CachedHashStringRef, size_t>;

void StringTableBuilder::write(uint8_t *Buf) const {
  assert(isFinalized());
  for (const StringPair &P : StringIndexMap) {
    StringRef Data = P.first.val();
    if (!Data.empty())
      memcpy(Buf + P.second, Data.data(), Data.size());
  }
  // The COFF formats store the size of the string table in the first 4 bytes.
  // For Windows, the format is little-endian; for AIX, it is big-endian.
  if (K == WinCOFF)
    support::endian::write32le(Buf, Size);
  else if (K == XCOFF)
```
- **EN**: Implements logic around `write`, `assert`, `val`, `memcpy`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `write`, `assert`, `val`, `memcpy`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 83-93
```cpp
    support::endian::write32be(Buf, Size);
}

// Returns the character at Pos from end of a string.
static int charTailAt(StringPair *P, size_t Pos) {
  StringRef S = P->first.val();
  if (Pos >= S.size())
    return -1;
  return (unsigned char)S[S.size() - Pos - 1];
}

```
- **EN**: Implements logic around `write32be`, `charTailAt`, `val`; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `write32be`, `charTailAt`, `val` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 94-100
```cpp
// Three-way radix quicksort. This is much faster than std::sort with strcmp
// because it does not compare characters that we already know the same.
static void multikeySort(MutableArrayRef<StringPair *> Vec, int Pos) {
tailcall:
  if (Vec.size() <= 1)
    return;

```
- **EN**: Implements logic around `multikeySort`.
- **CN**: 围绕 `multikeySort` 实现具体逻辑。

### Lines 101-114
```cpp
  // Partition items so that items in [0, I) are greater than the pivot,
  // [I, J) are the same as the pivot, and [J, Vec.size()) are less than
  // the pivot.
  int Pivot = charTailAt(Vec[0], Pos);
  size_t I = 0;
  size_t J = Vec.size();
  for (size_t K = 1; K < J;) {
    int C = charTailAt(Vec[K], Pos);
    if (C > Pivot)
      std::swap(Vec[I++], Vec[K++]);
    else if (C < Pivot)
      std::swap(Vec[--J], Vec[K]);
    else
      K++;
```
- **EN**: Implements logic around `charTailAt`, `size`, `swap`.
- **CN**: 围绕 `charTailAt`, `size`, `swap` 实现具体逻辑。

### Lines 115-128
```cpp
  }

  multikeySort(Vec.slice(0, I), Pos);
  multikeySort(Vec.slice(J), Pos);

  // multikeySort(Vec.slice(I, J - I), Pos + 1), but with
  // tail call optimization.
  if (Pivot != -1) {
    Vec = Vec.slice(I, J - I);
    ++Pos;
    goto tailcall;
  }
}

```
- **EN**: Implements logic around `multikeySort`, `slice`.
- **CN**: 围绕 `multikeySort`, `slice` 实现具体逻辑。

### Lines 129-137
```cpp
void StringTableBuilder::finalize() {
  assert(K != DWARF);
  finalizeStringTable(/*Optimize=*/true);
}

void StringTableBuilder::finalizeInOrder() {
  finalizeStringTable(/*Optimize=*/false);
}

```
- **EN**: Implements logic around `finalize`, `assert`, `finalizeStringTable`, `finalizeInOrder`.
- **CN**: 围绕 `finalize`, `assert`, `finalizeStringTable`, `finalizeInOrder` 实现具体逻辑。

### Lines 138-146
```cpp
void StringTableBuilder::finalizeStringTable(bool Optimize) {
  Finalized = true;

  if (Optimize && StringIndexMap.size()) {
    std::vector<StringPair *> Strings;
    Strings.reserve(StringIndexMap.size());
    for (StringPair &P : StringIndexMap)
      Strings.push_back(&P);

```
- **EN**: Implements logic around `finalizeStringTable`, `reserve`, `push_back`.
- **CN**: 围绕 `finalizeStringTable`, `reserve`, `push_back` 实现具体逻辑。

### Lines 147-160
```cpp
    size_t RangeBegin = 0;
    MutableArrayRef<StringPair *> StringsRef(Strings);
    if (StringPriorityMap.size()) {
      llvm::sort(Strings,
                 [&](const StringPair *LHS, const StringPair *RHS) -> bool {
                   return StringPriorityMap.lookup(LHS->first) >
                          StringPriorityMap.lookup(RHS->first);
                 });
      uint8_t RangePriority = StringPriorityMap.lookup(Strings[0]->first);
      for (size_t I = 1, E = Strings.size(); I != E && RangePriority; ++I) {
        uint8_t Priority = StringPriorityMap.lookup(Strings[I]->first);
        if (Priority != RangePriority) {
          multikeySort(StringsRef.slice(RangeBegin, I - RangeBegin), 0);
          RangePriority = Priority;
```
- **EN**: Implements logic around `StringsRef`, `sort`, `lookup`, `multikeySort`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `StringsRef`, `sort`, `lookup`, `multikeySort` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 161-167
```cpp
          RangeBegin = I;
        }
      }
    }
    multikeySort(StringsRef.slice(RangeBegin), 0);
    initSize();

```
- **EN**: Implements logic around `multikeySort`, `initSize`.
- **CN**: 围绕 `multikeySort`, `initSize` 实现具体逻辑。

### Lines 168-178
```cpp
    StringRef Previous;
    for (StringPair *P : Strings) {
      StringRef S = P->first.val();
      if (Previous.ends_with(S)) {
        size_t Pos = Size - S.size() - (K != RAW);
        if (isAligned(Alignment, Pos)) {
          P->second = Pos;
          continue;
        }
      }

```
- **EN**: Implements logic around `val`, `size`.
- **CN**: 围绕 `val`, `size` 实现具体逻辑。

### Lines 179-188
```cpp
      Size = alignTo(Size, Alignment);
      P->second = Size;

      Size += S.size();
      if (K != RAW)
        ++Size;
      Previous = S;
    }
  }

```
- **EN**: Implements logic around `alignTo`, `size`.
- **CN**: 围绕 `alignTo`, `size` 实现具体逻辑。

### Lines 189-199
```cpp
  if (K == MachO || K == MachOLinked || K == DXContainer)
    Size = alignTo(Size, 4); // Pad to multiple of 4.
  if (K == MachO64 || K == MachO64Linked)
    Size = alignTo(Size, 8); // Pad to multiple of 8.

  // According to ld64 the string table of a final linked Mach-O binary starts
  // with " ", i.e. the first byte is ' ' and the second byte is zero. In
  // 'initSize()' we reserved the first two bytes for holding this string.
  if (K == MachOLinked || K == MachO64Linked)
    StringIndexMap[CachedHashStringRef(" ")] = 0;

```
- **EN**: Implements logic around `alignTo`, `CachedHashStringRef`.
- **CN**: 围绕 `alignTo`, `CachedHashStringRef` 实现具体逻辑。

### Lines 200-207
```cpp
  // The first byte in an ELF string table must be null, according to the ELF
  // specification. In 'initSize()' we reserved the first byte to hold null for
  // this purpose and here we actually add the string to allow 'getOffset()' to
  // be called on an empty string.
  if (K == ELF || K == DXContainer)
    StringIndexMap[CachedHashStringRef("")] = 0;
}

```
- **EN**: Implements logic around `CachedHashStringRef`.
- **CN**: 围绕 `CachedHashStringRef` 实现具体逻辑。

### Lines 208-219
```cpp
void StringTableBuilder::clear() {
  Finalized = false;
  StringIndexMap.clear();
}

size_t StringTableBuilder::getOffset(CachedHashStringRef S) const {
  assert(isFinalized());
  auto I = StringIndexMap.find(S);
  assert(I != StringIndexMap.end() && "String is not in table!");
  return I->second;
}

```
- **EN**: Implements logic around `clear`, `getOffset`, `assert`, `find`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `clear`, `getOffset`, `assert`, `find` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 220-227
```cpp
size_t StringTableBuilder::add(CachedHashStringRef S, uint8_t Priority) {
  if (K == WinCOFF)
    assert(S.size() > COFF::NameSize && "Short string in COFF string table!");

  assert(!isFinalized());
  if (Priority)
    StringPriorityMap[S] = std::max(Priority, StringPriorityMap[S]);

```
- **EN**: Implements logic around `add`, `assert`, `max`.
- **CN**: 围绕 `add`, `assert`, `max` 实现具体逻辑。

### Lines 228-235
```cpp
  auto P = StringIndexMap.try_emplace(S);
  if (P.second) {
    size_t Start = alignTo(Size, Alignment);
    P.first->second = Start;
    Size = Start + S.size() + (K != RAW);
  }
  return P.first->second;
}
```
- **EN**: Implements logic around `try_emplace`, `alignTo`, `size`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `try_emplace`, `alignTo`, `size` 实现具体逻辑；这一段返回子系统相关的计算结果。

## Key Concepts / 关键概念

- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/MC/StringTableBuilder.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/CachedHashString.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/StringRef.h`, `llvm/BinaryFormat/COFF.h`, `llvm/Support/Endian.h`, `llvm/Support/MathExtras.h`, `llvm/Support/raw_ostream.h`, `cassert`, `cstddef`, `cstdint` ... (+3 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support, BinaryFormat
