# DataExtractor.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/DataExtractor.cpp`
- Repository: `llvm-project`
- Purpose (EN): E = createStringError(
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `DataExtractor` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===-- DataExtractor.cpp -------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/DataExtractor.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/Support/Errc.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/LEB128.h"
#include "llvm/Support/SwapByteOrder.h"

using namespace llvm;

bool DataExtractor::prepareRead(uint64_t Offset, uint64_t Size,
                                Error *E) const {
  if (isValidOffsetForDataOfSize(Offset, Size))
    return true;
  if (E) {
    if (Offset <= Data.size())
      *E = createStringError(
          errc::illegal_byte_sequence,
          "unexpected end of data at offset 0x%zx while reading [0x%" PRIx64
          ", 0x%" PRIx64 ")",
          Data.size(), Offset, Offset + Size);
    else
      *E = createStringError(errc::invalid_argument,
                             "offset 0x%" PRIx64
                             " is beyond the end of data at 0x%zx",
                             Offset, Data.size());
  }
  return false;
}

static bool isError(Error *E) { return E && *E; }

template <typename T>
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 6 direct dependencies, including `llvm/Support/DataExtractor.h`, `llvm/ADT/StringExtras.h`, `llvm/Support/Errc.h`, `llvm/Support/ErrorHandling.h`.
  CN: 引入了 6 个直接依赖，其中包括 `llvm/Support/DataExtractor.h`, `llvm/ADT/StringExtras.h`, `llvm/Support/Errc.h`, `llvm/Support/ErrorHandling.h`。
- EN: This section centers on `prepareRead`, `isError` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `prepareRead`, `isError` 等符号展开，负责解析输入并把原始数据转换成结构化形式。

### Lines 41-80

```cpp
T DataExtractor::getU(uint64_t *offset_ptr, Error *Err) const {
  ErrorAsOutParameter ErrAsOut(Err);
  T val = 0;
  if (isError(Err))
    return val;

  uint64_t offset = *offset_ptr;
  if (!prepareRead(offset, sizeof(T), Err))
    return val;
  std::memcpy(&val, &Data.data()[offset], sizeof(val));
  if (sys::IsLittleEndianHost != IsLittleEndian)
    sys::swapByteOrder(val);

  // Advance the offset
  *offset_ptr += sizeof(val);
  return val;
}

template <typename T>
T *DataExtractor::getUs(uint64_t *offset_ptr, T *dst, uint32_t count,
                        Error *Err) const {
  ErrorAsOutParameter ErrAsOut(Err);
  if (isError(Err))
    return nullptr;

  uint64_t offset = *offset_ptr;

  if (!prepareRead(offset, sizeof(*dst) * count, Err))
    return nullptr;
  for (T *value_ptr = dst, *end = dst + count; value_ptr != end;
       ++value_ptr, offset += sizeof(*dst))
    *value_ptr = getU<T>(offset_ptr, Err);
  // Advance the offset
  *offset_ptr = offset;
  // Return a non-NULL pointer to the converted data as an indicator of
  // success
  return dst;
}

uint8_t DataExtractor::getU8(uint64_t *offset_ptr, llvm::Error *Err) const {
```
- EN: This section centers on `getU`, `ErrAsOut`, `memcpy` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getU`, `ErrAsOut`, `memcpy` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 81-120

```cpp
  return getU<uint8_t>(offset_ptr, Err);
}

uint8_t *DataExtractor::getU8(uint64_t *offset_ptr, uint8_t *dst,
                              uint32_t count) const {
  return getUs<uint8_t>(offset_ptr, dst, count, nullptr);
}

uint8_t *DataExtractor::getU8(Cursor &C, uint8_t *Dst, uint32_t Count) const {
  return getUs<uint8_t>(&C.Offset, Dst, Count, &C.Err);
}

uint16_t DataExtractor::getU16(uint64_t *offset_ptr, llvm::Error *Err) const {
  return getU<uint16_t>(offset_ptr, Err);
}

uint16_t *DataExtractor::getU16(uint64_t *offset_ptr, uint16_t *dst,
                                uint32_t count) const {
  return getUs<uint16_t>(offset_ptr, dst, count, nullptr);
}

uint32_t DataExtractor::getU24(uint64_t *OffsetPtr, Error *Err) const {
  uint24_t ExtractedVal = getU<uint24_t>(OffsetPtr, Err);
  // The 3 bytes are in the correct byte order for the host.
  return ExtractedVal.getAsUint32(sys::IsLittleEndianHost);
}

uint32_t DataExtractor::getU32(uint64_t *offset_ptr, llvm::Error *Err) const {
  return getU<uint32_t>(offset_ptr, Err);
}

uint32_t *DataExtractor::getU32(uint64_t *offset_ptr, uint32_t *dst,
                                uint32_t count) const {
  return getUs<uint32_t>(offset_ptr, dst, count, nullptr);
}

uint64_t DataExtractor::getU64(uint64_t *offset_ptr, llvm::Error *Err) const {
  return getU<uint64_t>(offset_ptr, Err);
}

```
- EN: This section centers on `getU16`, `getU24`, `getU32` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getU16`, `getU24`, `getU32` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code propagates LLVM-style errors and invariants and returns the resulting value to its callers.
  CN: 在这一段中，代码传播 LLVM 风格的错误处理并维护不变量，并将结果返回给调用方。

### Lines 121-160

```cpp
uint64_t *DataExtractor::getU64(uint64_t *offset_ptr, uint64_t *dst,
                                uint32_t count) const {
  return getUs<uint64_t>(offset_ptr, dst, count, nullptr);
}

uint64_t DataExtractor::getUnsigned(uint64_t *offset_ptr, uint32_t byte_size,
                                    llvm::Error *Err) const {
  switch (byte_size) {
  case 1:
    return getU8(offset_ptr, Err);
  case 2:
    return getU16(offset_ptr, Err);
  case 4:
    return getU32(offset_ptr, Err);
  case 8:
    return getU64(offset_ptr, Err);
  }

  // For any other byte size, read the bytes and swap/shift if necessary.
  ErrorAsOutParameter ErrAsOut(Err);
  uint64_t val = 0;
  if (isError(Err))
    return val;
  uint64_t offset = *offset_ptr;
  if (!prepareRead(offset, byte_size, Err))
    return val;
  // Copy into the least significant bytes of val regardless of host
  // endianness.
  std::memcpy(reinterpret_cast<char *>(&val) +
                  (sys::IsLittleEndianHost ? 0 : 8 - byte_size),
              &Data.data()[offset], byte_size);
  // Swap the least significant bytes of val if endianness doesn't match.
  if (sys::IsLittleEndianHost != IsLittleEndian)
    val = sys::getSwappedBytes(val) >> (8 * (8 - byte_size));

  *offset_ptr += byte_size;
  return val;
}

int64_t
```
- EN: This section centers on `getUnsigned`, `getU8`, `getU16` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getUnsigned`, `getU8`, `getU16` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and checks conditions and handles edge cases.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并检查条件并处理边界情况。

### Lines 161-200

```cpp
DataExtractor::getSigned(uint64_t *offset_ptr, uint32_t byte_size) const {
  switch (byte_size) {
  case 1:
    return getS8(offset_ptr);
  case 2:
    return getS16(offset_ptr);
  case 4:
    return getS32(offset_ptr);
  case 8:
    return getS64(offset_ptr);
  }
  llvm_unreachable("getSigned unhandled case!");
}

StringRef DataExtractor::getCStrRef(uint64_t *OffsetPtr, Error *Err) const {
  ErrorAsOutParameter ErrAsOut(Err);
  if (isError(Err))
    return StringRef();

  uint64_t Start = *OffsetPtr;
  StringRef::size_type Pos = Data.find('\0', Start);
  if (Pos != StringRef::npos) {
    *OffsetPtr = Pos + 1;
    return StringRef(Data.data() + Start, Pos - Start);
  }
  if (Err)
    *Err = createStringError(errc::illegal_byte_sequence,
                             "no null terminated string at offset 0x%" PRIx64,
                             Start);
  return StringRef();
}

StringRef DataExtractor::getFixedLengthString(uint64_t *OffsetPtr,
                                              uint64_t Length,
                                              StringRef TrimChars) const {
  StringRef Bytes(getBytes(OffsetPtr, Length));
  return Bytes.trim(TrimChars);
}

StringRef DataExtractor::getBytes(uint64_t *OffsetPtr, uint64_t Length,
```
- EN: This section centers on `getSigned`, `getS8`, `getS16` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getSigned`, `getS8`, `getS16` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and checks conditions and handles edge cases.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并检查条件并处理边界情况。

### Lines 201-240

```cpp
                                  Error *Err) const {
  ErrorAsOutParameter ErrAsOut(Err);
  if (isError(Err))
    return StringRef();

  if (!prepareRead(*OffsetPtr, Length, Err))
    return StringRef();

  StringRef Result = Data.substr(*OffsetPtr, Length);
  *OffsetPtr += Length;
  return Result;
}

template <typename T>
static T getLEB128(StringRef Data, uint64_t *OffsetPtr, Error *Err,
                   T (&Decoder)(const uint8_t *p, unsigned *n,
                                const uint8_t *end, const char **error)) {
  ArrayRef<uint8_t> Bytes = arrayRefFromStringRef(Data);
  assert(*OffsetPtr <= Bytes.size());
  ErrorAsOutParameter ErrAsOut(Err);
  if (isError(Err))
    return T();

  const char *error = nullptr;
  unsigned bytes_read;
  T result =
      Decoder(Bytes.data() + *OffsetPtr, &bytes_read, Bytes.end(), &error);
  if (error) {
    if (Err)
      *Err = createStringError(errc::illegal_byte_sequence,
                               "unable to decode LEB128 at offset 0x%8.8" PRIx64
                               ": %s",
                               *OffsetPtr, error);
    return T();
  }
  *OffsetPtr += bytes_read;
  return result;
}

uint64_t DataExtractor::getULEB128(uint64_t *offset_ptr, Error *Err) const {
```
- EN: This section centers on `ErrAsOut`, `getLEB128`, `assert` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `ErrAsOut`, `getLEB128`, `assert` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 241-255

```cpp
  return getLEB128(Data, offset_ptr, Err, decodeULEB128);
}

int64_t DataExtractor::getSLEB128(uint64_t *offset_ptr, Error *Err) const {
  return getLEB128(Data, offset_ptr, Err, decodeSLEB128);
}

void DataExtractor::skip(Cursor &C, uint64_t Length) const {
  ErrorAsOutParameter ErrAsOut(C.Err);
  if (isError(&C.Err))
    return;

  if (prepareRead(C.Offset, Length, &C.Err))
    C.Offset += Length;
}
```
- EN: This section centers on `getLEB128`, `getSLEB128`, `skip` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getLEB128`, `getSLEB128`, `skip` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `prepareRead`, `isError`, `getU`, `ErrAsOut` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Parsing, decoding, and structural validation. / 重点关注解析、解码与结构校验。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/DataExtractor.h`, `llvm/ADT/StringExtras.h`, `llvm/Support/Errc.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/LEB128.h`, `llvm/Support/SwapByteOrder.h`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `prepareRead`, `isError`, `getU`, `ErrAsOut`, `memcpy`
