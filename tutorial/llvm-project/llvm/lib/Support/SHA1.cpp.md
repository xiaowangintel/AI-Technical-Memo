# SHA1.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/SHA1.cpp`
- Repository: `llvm-project`
- Purpose (EN): This code is taken from public domain (http://oauth.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `SHA1` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//====- SHA1.cpp - Private copy of the SHA1 implementation ---*- C++ -* ======//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This code is taken from public domain
// (http://oauth.googlecode.com/svn/code/c/liboauth/src/sha1.c and
// http://cvsweb.netbsd.org/bsdweb.cgi/src/common/lib/libc/hash/sha1/sha1.c?rev=1.6)
// and modified by wrapping it in a C++ interface for LLVM,
// and removing unnecessary code.
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/SHA1.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/Support/Endian.h"
#include "llvm/Support/SwapByteOrder.h"
#include <string.h>

using namespace llvm;

static inline uint32_t rol(uint32_t Number, int Bits) {
  return (Number << Bits) | (Number >> (32 - Bits));
}

static inline uint32_t blk0(uint32_t *Buf, int I) { return Buf[I]; }

static inline uint32_t blk(uint32_t *Buf, int I) {
  Buf[I & 15] = rol(Buf[(I + 13) & 15] ^ Buf[(I + 8) & 15] ^ Buf[(I + 2) & 15] ^
                        Buf[I & 15],
                    1);
  return Buf[I & 15];
}

static inline void r0(uint32_t &A, uint32_t &B, uint32_t &C, uint32_t &D,
                      uint32_t &E, int I, uint32_t *Buf) {
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 6 direct dependencies, including `llvm/Support/SHA1.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/StringRef.h`, `llvm/Support/Endian.h`.
  CN: 引入了 6 个直接依赖，其中包括 `llvm/Support/SHA1.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/StringRef.h`, `llvm/Support/Endian.h`。
- EN: This section centers on `rol`, `blk0`, `blk` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `rol`, `blk0`, `blk` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 41-80

```cpp
  E += ((B & (C ^ D)) ^ D) + blk0(Buf, I) + 0x5A827999 + rol(A, 5);
  B = rol(B, 30);
}

static inline void r1(uint32_t &A, uint32_t &B, uint32_t &C, uint32_t &D,
                      uint32_t &E, int I, uint32_t *Buf) {
  E += ((B & (C ^ D)) ^ D) + blk(Buf, I) + 0x5A827999 + rol(A, 5);
  B = rol(B, 30);
}

static inline void r2(uint32_t &A, uint32_t &B, uint32_t &C, uint32_t &D,
                      uint32_t &E, int I, uint32_t *Buf) {
  E += (B ^ C ^ D) + blk(Buf, I) + 0x6ED9EBA1 + rol(A, 5);
  B = rol(B, 30);
}

static inline void r3(uint32_t &A, uint32_t &B, uint32_t &C, uint32_t &D,
                      uint32_t &E, int I, uint32_t *Buf) {
  E += (((B | C) & D) | (B & C)) + blk(Buf, I) + 0x8F1BBCDC + rol(A, 5);
  B = rol(B, 30);
}

static inline void r4(uint32_t &A, uint32_t &B, uint32_t &C, uint32_t &D,
                      uint32_t &E, int I, uint32_t *Buf) {
  E += (B ^ C ^ D) + blk(Buf, I) + 0xCA62C1D6 + rol(A, 5);
  B = rol(B, 30);
}

/* code */
#define SHA1_K0 0x5a827999
#define SHA1_K20 0x6ed9eba1
#define SHA1_K40 0x8f1bbcdc
#define SHA1_K60 0xca62c1d6

#define SEED_0 0x67452301
#define SEED_1 0xefcdab89
#define SEED_2 0x98badcfe
#define SEED_3 0x10325476
#define SEED_4 0xc3d2e1f0

```
- EN: This section centers on `r1`, `r2`, `r3` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `r1`, `r2`, `r3` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 81-120

```cpp
void SHA1::init() {
  InternalState.State[0] = SEED_0;
  InternalState.State[1] = SEED_1;
  InternalState.State[2] = SEED_2;
  InternalState.State[3] = SEED_3;
  InternalState.State[4] = SEED_4;
  InternalState.ByteCount = 0;
  InternalState.BufferOffset = 0;
}

void SHA1::hashBlock() {
  uint32_t A = InternalState.State[0];
  uint32_t B = InternalState.State[1];
  uint32_t C = InternalState.State[2];
  uint32_t D = InternalState.State[3];
  uint32_t E = InternalState.State[4];

  // 4 rounds of 20 operations each. Loop unrolled.
  r0(A, B, C, D, E, 0, InternalState.Buffer.L);
  r0(E, A, B, C, D, 1, InternalState.Buffer.L);
  r0(D, E, A, B, C, 2, InternalState.Buffer.L);
  r0(C, D, E, A, B, 3, InternalState.Buffer.L);
  r0(B, C, D, E, A, 4, InternalState.Buffer.L);
  r0(A, B, C, D, E, 5, InternalState.Buffer.L);
  r0(E, A, B, C, D, 6, InternalState.Buffer.L);
  r0(D, E, A, B, C, 7, InternalState.Buffer.L);
  r0(C, D, E, A, B, 8, InternalState.Buffer.L);
  r0(B, C, D, E, A, 9, InternalState.Buffer.L);
  r0(A, B, C, D, E, 10, InternalState.Buffer.L);
  r0(E, A, B, C, D, 11, InternalState.Buffer.L);
  r0(D, E, A, B, C, 12, InternalState.Buffer.L);
  r0(C, D, E, A, B, 13, InternalState.Buffer.L);
  r0(B, C, D, E, A, 14, InternalState.Buffer.L);
  r0(A, B, C, D, E, 15, InternalState.Buffer.L);
  r1(E, A, B, C, D, 16, InternalState.Buffer.L);
  r1(D, E, A, B, C, 17, InternalState.Buffer.L);
  r1(C, D, E, A, B, 18, InternalState.Buffer.L);
  r1(B, C, D, E, A, 19, InternalState.Buffer.L);

  r2(A, B, C, D, E, 20, InternalState.Buffer.L);
```
- EN: This section centers on `init`, `hashBlock`, `r0` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `init`, `hashBlock`, `r0` 等符号展开，负责创建并初始化辅助对象或状态。

### Lines 121-160

```cpp
  r2(E, A, B, C, D, 21, InternalState.Buffer.L);
  r2(D, E, A, B, C, 22, InternalState.Buffer.L);
  r2(C, D, E, A, B, 23, InternalState.Buffer.L);
  r2(B, C, D, E, A, 24, InternalState.Buffer.L);
  r2(A, B, C, D, E, 25, InternalState.Buffer.L);
  r2(E, A, B, C, D, 26, InternalState.Buffer.L);
  r2(D, E, A, B, C, 27, InternalState.Buffer.L);
  r2(C, D, E, A, B, 28, InternalState.Buffer.L);
  r2(B, C, D, E, A, 29, InternalState.Buffer.L);
  r2(A, B, C, D, E, 30, InternalState.Buffer.L);
  r2(E, A, B, C, D, 31, InternalState.Buffer.L);
  r2(D, E, A, B, C, 32, InternalState.Buffer.L);
  r2(C, D, E, A, B, 33, InternalState.Buffer.L);
  r2(B, C, D, E, A, 34, InternalState.Buffer.L);
  r2(A, B, C, D, E, 35, InternalState.Buffer.L);
  r2(E, A, B, C, D, 36, InternalState.Buffer.L);
  r2(D, E, A, B, C, 37, InternalState.Buffer.L);
  r2(C, D, E, A, B, 38, InternalState.Buffer.L);
  r2(B, C, D, E, A, 39, InternalState.Buffer.L);

  r3(A, B, C, D, E, 40, InternalState.Buffer.L);
  r3(E, A, B, C, D, 41, InternalState.Buffer.L);
  r3(D, E, A, B, C, 42, InternalState.Buffer.L);
  r3(C, D, E, A, B, 43, InternalState.Buffer.L);
  r3(B, C, D, E, A, 44, InternalState.Buffer.L);
  r3(A, B, C, D, E, 45, InternalState.Buffer.L);
  r3(E, A, B, C, D, 46, InternalState.Buffer.L);
  r3(D, E, A, B, C, 47, InternalState.Buffer.L);
  r3(C, D, E, A, B, 48, InternalState.Buffer.L);
  r3(B, C, D, E, A, 49, InternalState.Buffer.L);
  r3(A, B, C, D, E, 50, InternalState.Buffer.L);
  r3(E, A, B, C, D, 51, InternalState.Buffer.L);
  r3(D, E, A, B, C, 52, InternalState.Buffer.L);
  r3(C, D, E, A, B, 53, InternalState.Buffer.L);
  r3(B, C, D, E, A, 54, InternalState.Buffer.L);
  r3(A, B, C, D, E, 55, InternalState.Buffer.L);
  r3(E, A, B, C, D, 56, InternalState.Buffer.L);
  r3(D, E, A, B, C, 57, InternalState.Buffer.L);
  r3(C, D, E, A, B, 58, InternalState.Buffer.L);
  r3(B, C, D, E, A, 59, InternalState.Buffer.L);
```
- EN: This section centers on `r2`, `r3` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `r2`, `r3` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 161-200

```cpp

  r4(A, B, C, D, E, 60, InternalState.Buffer.L);
  r4(E, A, B, C, D, 61, InternalState.Buffer.L);
  r4(D, E, A, B, C, 62, InternalState.Buffer.L);
  r4(C, D, E, A, B, 63, InternalState.Buffer.L);
  r4(B, C, D, E, A, 64, InternalState.Buffer.L);
  r4(A, B, C, D, E, 65, InternalState.Buffer.L);
  r4(E, A, B, C, D, 66, InternalState.Buffer.L);
  r4(D, E, A, B, C, 67, InternalState.Buffer.L);
  r4(C, D, E, A, B, 68, InternalState.Buffer.L);
  r4(B, C, D, E, A, 69, InternalState.Buffer.L);
  r4(A, B, C, D, E, 70, InternalState.Buffer.L);
  r4(E, A, B, C, D, 71, InternalState.Buffer.L);
  r4(D, E, A, B, C, 72, InternalState.Buffer.L);
  r4(C, D, E, A, B, 73, InternalState.Buffer.L);
  r4(B, C, D, E, A, 74, InternalState.Buffer.L);
  r4(A, B, C, D, E, 75, InternalState.Buffer.L);
  r4(E, A, B, C, D, 76, InternalState.Buffer.L);
  r4(D, E, A, B, C, 77, InternalState.Buffer.L);
  r4(C, D, E, A, B, 78, InternalState.Buffer.L);
  r4(B, C, D, E, A, 79, InternalState.Buffer.L);

  InternalState.State[0] += A;
  InternalState.State[1] += B;
  InternalState.State[2] += C;
  InternalState.State[3] += D;
  InternalState.State[4] += E;
}

void SHA1::addUncounted(uint8_t Data) {
  if constexpr (sys::IsBigEndianHost)
    InternalState.Buffer.C[InternalState.BufferOffset] = Data;
  else
    InternalState.Buffer.C[InternalState.BufferOffset ^ 3] = Data;

  InternalState.BufferOffset++;
  if (InternalState.BufferOffset == BLOCK_LENGTH) {
    hashBlock();
    InternalState.BufferOffset = 0;
  }
```
- EN: This section centers on `r4`, `addUncounted`, `hashBlock` and performs utility computation and state updates.
  CN: 这一段主要围绕 `r4`, `addUncounted`, `hashBlock` 等符号展开，负责执行工具性计算并更新状态。
- EN: This range checks conditions and handles edge cases.
  CN: 这一段检查条件并处理边界情况。

### Lines 201-240

```cpp
}

void SHA1::writebyte(uint8_t Data) {
  ++InternalState.ByteCount;
  addUncounted(Data);
}

void SHA1::update(ArrayRef<uint8_t> Data) {
  InternalState.ByteCount += Data.size();

  // Finish the current block.
  if (InternalState.BufferOffset > 0) {
    const size_t Remainder = std::min<size_t>(
        Data.size(), BLOCK_LENGTH - InternalState.BufferOffset);
    for (size_t I = 0; I < Remainder; ++I)
      addUncounted(Data[I]);
    Data = Data.drop_front(Remainder);
  }

  // Fast buffer filling for large inputs.
  while (Data.size() >= BLOCK_LENGTH) {
    assert(InternalState.BufferOffset == 0);
    static_assert(BLOCK_LENGTH % 4 == 0);
    constexpr size_t BLOCK_LENGTH_32 = BLOCK_LENGTH / 4;
    for (size_t I = 0; I < BLOCK_LENGTH_32; ++I)
      InternalState.Buffer.L[I] = support::endian::read32be(&Data[I * 4]);
    hashBlock();
    Data = Data.drop_front(BLOCK_LENGTH);
  }

  // Finish the remainder.
  for (uint8_t C : Data)
    addUncounted(C);
}

void SHA1::update(StringRef Str) {
  update(
      ArrayRef<uint8_t>((uint8_t *)const_cast<char *>(Str.data()), Str.size()));
}

```
- EN: This section centers on `writebyte`, `addUncounted`, `update` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `writebyte`, `addUncounted`, `update` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 241-280

```cpp
void SHA1::pad() {
  // Implement SHA-1 padding (fips180-2 5.1.1)

  // Pad with 0x80 followed by 0x00 until the end of the block
  addUncounted(0x80);
  while (InternalState.BufferOffset != 56)
    addUncounted(0x00);

  // Append length in the last 8 bytes
  addUncounted(0); // We're only using 32 bit lengths
  addUncounted(0); // But SHA-1 supports 64 bit lengths
  addUncounted(0); // So zero pad the top bits
  addUncounted(InternalState.ByteCount >> 29); // Shifting to multiply by 8
  addUncounted(InternalState.ByteCount >>
               21); // as SHA-1 supports bitstreams as well as
  addUncounted(InternalState.ByteCount >> 13); // byte.
  addUncounted(InternalState.ByteCount >> 5);
  addUncounted(InternalState.ByteCount << 3);
}

void SHA1::final(std::array<uint32_t, HASH_LENGTH / 4> &HashResult) {
  // Pad to complete the last block
  pad();

  if constexpr (sys::IsBigEndianHost) {
    // Just copy the current state
    for (int i = 0; i < 5; i++) {
      HashResult[i] = InternalState.State[i];
    }
  } else {
    // Swap byte order back
    for (int i = 0; i < 5; i++) {
      HashResult[i] = llvm::byteswap(InternalState.State[i]);
    }
  }
}

std::array<uint8_t, 20> SHA1::final() {
  union {
    std::array<uint32_t, HASH_LENGTH / 4> HashResult;
```
- EN: This section centers on `pad`, `addUncounted`, `final` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `pad`, `addUncounted`, `final` 等符号展开，负责实现局部控制流程与状态维护。
- EN: This range iterates over collections, ranges, or records.
  CN: 这一段遍历集合、区间或记录。

### Lines 281-304

```cpp
    std::array<uint8_t, HASH_LENGTH> ReturnResult;
  };
  static_assert(sizeof(HashResult) == sizeof(ReturnResult));
  final(HashResult);
  return ReturnResult;
}

std::array<uint8_t, 20> SHA1::result() {
  auto StateToRestore = InternalState;

  auto Hash = final();

  // Restore the state
  InternalState = StateToRestore;

  // Return pointer to hash (20 characters)
  return Hash;
}

std::array<uint8_t, 20> SHA1::hash(ArrayRef<uint8_t> Data) {
  SHA1 Hash;
  Hash.update(Data);
  return Hash.final();
}
```
- EN: This section centers on `static_assert`, `final`, `result` and performs utility computation and state updates.
  CN: 这一段主要围绕 `static_assert`, `final`, `result` 等符号展开，负责执行工具性计算并更新状态。
- EN: In this range, the code propagates LLVM-style errors and invariants and returns the resulting value to its callers.
  CN: 在这一段中，代码传播 LLVM 风格的错误处理并维护不变量，并将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `rol`, `blk0`, `blk`, `r0` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/SHA1.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/StringRef.h`, `llvm/Support/Endian.h`, `llvm/Support/SwapByteOrder.h`
- Standard library / 标准库: `string.h`
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `rol`, `blk0`, `blk`, `r0`, `r1`
