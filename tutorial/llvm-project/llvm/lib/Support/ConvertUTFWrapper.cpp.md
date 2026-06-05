# ConvertUTFWrapper.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/ConvertUTFWrapper.cpp`
- Repository: `llvm-project`
- Purpose (EN): Copy the character span over.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `ConvertUTFWrapper` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===-- ConvertUTFWrapper.cpp - Wrap ConvertUTF.h with clang data types -----===
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/bit.h"
#include "llvm/Support/ConvertUTF.h"
#include "llvm/Support/ErrorHandling.h"
#include <string>
#include <vector>

namespace llvm {

bool ConvertUTF8toWide(unsigned WideCharWidth, llvm::StringRef Source,
                       char *&ResultPtr, const UTF8 *&ErrorPtr) {
  assert(WideCharWidth == 1 || WideCharWidth == 2 || WideCharWidth == 4);
  ConversionResult result = conversionOK;
  // Copy the character span over.
  if (WideCharWidth == 1) {
    const UTF8 *Pos = reinterpret_cast<const UTF8*>(Source.begin());
    if (!isLegalUTF8String(&Pos, reinterpret_cast<const UTF8*>(Source.end()))) {
      result = sourceIllegal;
      ErrorPtr = Pos;
    } else {
      memcpy(ResultPtr, Source.data(), Source.size());
      ResultPtr += Source.size();
    }
  } else if (WideCharWidth == 2) {
    const UTF8 *sourceStart = (const UTF8*)Source.data();
    // FIXME: Make the type of the result buffer correct instead of
    // using reinterpret_cast.
    UTF16 *targetStart = reinterpret_cast<UTF16 *>(ResultPtr);
    ConversionFlags flags = strictConversion;
    result =
        ConvertUTF8toUTF16(&sourceStart, sourceStart + Source.size(),
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 7 direct dependencies, including `llvm/ADT/ArrayRef.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/bit.h`, `llvm/Support/ConvertUTF.h`.
  CN: 引入了 7 个直接依赖，其中包括 `llvm/ADT/ArrayRef.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/bit.h`, `llvm/Support/ConvertUTF.h`。
- EN: This section centers on `ConvertUTF8toWide`, `assert`, `memcpy` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `ConvertUTF8toWide`, `assert`, `memcpy` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 41-80

```cpp
                           &targetStart, targetStart + Source.size(), flags);
    if (result == conversionOK)
      ResultPtr = reinterpret_cast<char *>(targetStart);
    else
      ErrorPtr = sourceStart;
  } else if (WideCharWidth == 4) {
    const UTF8 *sourceStart = (const UTF8 *)Source.data();
    // FIXME: Make the type of the result buffer correct instead of
    // using reinterpret_cast.
    UTF32 *targetStart = reinterpret_cast<UTF32 *>(ResultPtr);
    ConversionFlags flags = strictConversion;
    result =
        ConvertUTF8toUTF32(&sourceStart, sourceStart + Source.size(),
                           &targetStart, targetStart + Source.size(), flags);
    if (result == conversionOK)
      ResultPtr = reinterpret_cast<char *>(targetStart);
    else
      ErrorPtr = sourceStart;
  }
  assert((result != targetExhausted) &&
         "ConvertUTF8toUTFXX exhausted target buffer");
  return result == conversionOK;
}

bool ConvertCodePointToUTF8(unsigned Source, char *&ResultPtr) {
  const UTF32 *SourceStart = &Source;
  const UTF32 *SourceEnd = SourceStart + 1;
  UTF8 *TargetStart = reinterpret_cast<UTF8 *>(ResultPtr);
  UTF8 *TargetEnd = TargetStart + 4;
  ConversionResult CR = ConvertUTF32toUTF8(
      &SourceStart, SourceEnd, &TargetStart, TargetEnd, strictConversion);
  if (CR != conversionOK)
    return false;

  ResultPtr = reinterpret_cast<char *>(TargetStart);
  return true;
}

bool hasUTF16ByteOrderMark(ArrayRef<char> S) {
  return (S.size() >= 2 && ((S[0] == '\xff' && S[1] == '\xfe') ||
```
- EN: This section centers on `ConvertUTF8toUTF32`, `assert`, `ConvertCodePointToUTF8` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `ConvertUTF8toUTF32`, `assert`, `ConvertCodePointToUTF8` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 81-120

```cpp
                            (S[0] == '\xfe' && S[1] == '\xff')));
}

bool convertUTF16ToUTF8String(ArrayRef<char> SrcBytes, std::string &Out) {
  assert(Out.empty());

  // Error out on an uneven byte count.
  if (SrcBytes.size() % 2)
    return false;

  // Avoid OOB by returning early on empty input.
  if (SrcBytes.empty())
    return true;

  const UTF16 *Src = reinterpret_cast<const UTF16 *>(SrcBytes.begin());
  const UTF16 *SrcEnd = reinterpret_cast<const UTF16 *>(SrcBytes.end());

  assert((uintptr_t)Src % sizeof(UTF16) == 0);

  // Byteswap if necessary.
  std::vector<UTF16> ByteSwapped;
  if (Src[0] == UNI_UTF16_BYTE_ORDER_MARK_SWAPPED) {
    ByteSwapped.insert(ByteSwapped.end(), Src, SrcEnd);
    for (UTF16 &I : ByteSwapped)
      I = llvm::byteswap<uint16_t>(I);
    Src = &ByteSwapped[0];
    SrcEnd = &ByteSwapped[ByteSwapped.size() - 1] + 1;
  }

  // Skip the BOM for conversion.
  if (Src[0] == UNI_UTF16_BYTE_ORDER_MARK_NATIVE)
    Src++;

  // Just allocate enough space up front.  We'll shrink it later.  Allocate
  // enough that we can fit a null terminator without reallocating.
  Out.resize(SrcBytes.size() * UNI_MAX_UTF8_BYTES_PER_CODE_POINT + 1);
  UTF8 *Dst = reinterpret_cast<UTF8 *>(&Out[0]);
  UTF8 *DstEnd = Dst + Out.size();

  ConversionResult CR =
```
- EN: This section centers on `convertUTF16ToUTF8String`, `assert` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `convertUTF16ToUTF8String`, `assert` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 121-160

```cpp
      ConvertUTF16toUTF8(&Src, SrcEnd, &Dst, DstEnd, strictConversion);
  assert(CR != targetExhausted);

  if (CR != conversionOK) {
    Out.clear();
    return false;
  }

  Out.resize(reinterpret_cast<char *>(Dst) - &Out[0]);
  Out.push_back(0);
  Out.pop_back();
  return true;
}

bool convertUTF16ToUTF8String(ArrayRef<UTF16> Src, std::string &Out) {
  return convertUTF16ToUTF8String(
      llvm::ArrayRef<char>(reinterpret_cast<const char *>(Src.data()),
                           Src.size() * sizeof(UTF16)),
      Out);
}

bool convertUTF32ToUTF8String(ArrayRef<char> SrcBytes, std::string &Out) {
  assert(Out.empty());

  // Error out on an uneven byte count.
  if (SrcBytes.size() % 4)
    return false;

  // Avoid OOB by returning early on empty input.
  if (SrcBytes.empty())
    return true;

  const UTF32 *Src = reinterpret_cast<const UTF32 *>(SrcBytes.begin());
  const UTF32 *SrcEnd = reinterpret_cast<const UTF32 *>(SrcBytes.end());

  assert((uintptr_t)Src % sizeof(UTF32) == 0);

  // Byteswap if necessary.
  std::vector<UTF32> ByteSwapped;
  if (Src[0] == UNI_UTF32_BYTE_ORDER_MARK_SWAPPED) {
```
- EN: This section centers on `ConvertUTF16toUTF8`, `assert`, `convertUTF16ToUTF8String` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `ConvertUTF16toUTF8`, `assert`, `convertUTF16ToUTF8String` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 161-200

```cpp
    ByteSwapped.insert(ByteSwapped.end(), Src, SrcEnd);
    for (UTF32 &I : ByteSwapped)
      I = llvm::byteswap<uint32_t>(I);
    Src = &ByteSwapped[0];
    SrcEnd = &ByteSwapped[ByteSwapped.size() - 1] + 1;
  }

  // Skip the BOM for conversion.
  if (Src[0] == UNI_UTF32_BYTE_ORDER_MARK_NATIVE)
    Src++;

  // Just allocate enough space up front.  We'll shrink it later.  Allocate
  // enough that we can fit a null terminator without reallocating.
  Out.resize(SrcBytes.size() * UNI_MAX_UTF8_BYTES_PER_CODE_POINT + 1);
  UTF8 *Dst = reinterpret_cast<UTF8 *>(&Out[0]);
  UTF8 *DstEnd = Dst + Out.size();

  ConversionResult CR =
      ConvertUTF32toUTF8(&Src, SrcEnd, &Dst, DstEnd, strictConversion);
  assert(CR != targetExhausted);

  if (CR != conversionOK) {
    Out.clear();
    return false;
  }

  Out.resize(reinterpret_cast<char *>(Dst) - &Out[0]);
  Out.push_back(0);
  Out.pop_back();
  return true;
}

bool convertUTF32ToUTF8String(ArrayRef<UTF32> Src, std::string &Out) {
  return convertUTF32ToUTF8String(
      llvm::ArrayRef<char>(reinterpret_cast<const char *>(Src.data()),
                           Src.size() * sizeof(UTF32)),
      Out);
}

bool convertUTF8ToUTF16String(StringRef SrcUTF8,
```
- EN: This section centers on `ConvertUTF32toUTF8`, `assert`, `convertUTF32ToUTF8String` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `ConvertUTF32toUTF8`, `assert`, `convertUTF32ToUTF8String` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 201-240

```cpp
                              SmallVectorImpl<UTF16> &DstUTF16) {
  assert(DstUTF16.empty());

  // Avoid OOB by returning early on empty input.
  if (SrcUTF8.empty()) {
    DstUTF16.push_back(0);
    DstUTF16.pop_back();
    return true;
  }

  const UTF8 *Src = reinterpret_cast<const UTF8 *>(SrcUTF8.begin());
  const UTF8 *SrcEnd = reinterpret_cast<const UTF8 *>(SrcUTF8.end());

  // Allocate the same number of UTF-16 code units as UTF-8 code units. Encoding
  // as UTF-16 should always require the same amount or less code units than the
  // UTF-8 encoding.  Allocate one extra byte for the null terminator though,
  // so that someone calling DstUTF16.data() gets a null terminated string.
  // We resize down later so we don't have to worry that this over allocates.
  DstUTF16.resize(SrcUTF8.size()+1);
  UTF16 *Dst = &DstUTF16[0];
  UTF16 *DstEnd = Dst + DstUTF16.size();

  ConversionResult CR =
      ConvertUTF8toUTF16(&Src, SrcEnd, &Dst, DstEnd, strictConversion);
  assert(CR != targetExhausted);

  if (CR != conversionOK) {
    DstUTF16.clear();
    return false;
  }

  DstUTF16.resize(Dst - &DstUTF16[0]);
  DstUTF16.push_back(0);
  DstUTF16.pop_back();
  return true;
}

static_assert(sizeof(wchar_t) == 1 || sizeof(wchar_t) == 2 ||
                  sizeof(wchar_t) == 4,
              "Expected wchar_t to be 1, 2, or 4 bytes");
```
- EN: This section centers on `assert`, `ConvertUTF8toUTF16`, `static_assert` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `assert`, `ConvertUTF8toUTF16`, `static_assert` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 241-280

```cpp

template <typename TResult>
static inline bool ConvertUTF8toWideInternal(llvm::StringRef Source,
                                             TResult &Result) {
  // Even in the case of UTF-16, the number of bytes in a UTF-8 string is
  // at least as large as the number of elements in the resulting wide
  // string, because surrogate pairs take at least 4 bytes in UTF-8.
  Result.resize(Source.size() + 1);
  char *ResultPtr = reinterpret_cast<char *>(&Result[0]);
  const UTF8 *ErrorPtr;
  if (!ConvertUTF8toWide(sizeof(wchar_t), Source, ResultPtr, ErrorPtr)) {
    Result.clear();
    return false;
  }
  Result.resize(reinterpret_cast<wchar_t *>(ResultPtr) - &Result[0]);
  return true;
}

bool ConvertUTF8toWide(llvm::StringRef Source, std::wstring &Result) {
  return ConvertUTF8toWideInternal(Source, Result);
}

bool ConvertUTF8toWide(const char *Source, std::wstring &Result) {
  if (!Source) {
    Result.clear();
    return true;
  }
  return ConvertUTF8toWide(llvm::StringRef(Source), Result);
}

bool convertWideToUTF8(const std::wstring &Source, std::string &Result) {
  if (sizeof(wchar_t) == 1) {
    const UTF8 *Start = reinterpret_cast<const UTF8 *>(Source.data());
    const UTF8 *End =
        reinterpret_cast<const UTF8 *>(Source.data() + Source.size());
    if (!isLegalUTF8String(&Start, End))
      return false;
    Result.resize(Source.size());
    memcpy(&Result[0], Source.data(), Source.size());
    return true;
```
- EN: This section centers on `ConvertUTF8toWideInternal`, `ConvertUTF8toWide`, `convertWideToUTF8` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `ConvertUTF8toWideInternal`, `ConvertUTF8toWide`, `convertWideToUTF8` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 281-318

```cpp
  } else if (sizeof(wchar_t) == 2) {
    return convertUTF16ToUTF8String(
        llvm::ArrayRef<UTF16>(reinterpret_cast<const UTF16 *>(Source.data()),
                              Source.size()),
        Result);
  } else if (sizeof(wchar_t) == 4) {
    const UTF32 *Start = reinterpret_cast<const UTF32 *>(Source.data());
    const UTF32 *End =
        reinterpret_cast<const UTF32 *>(Source.data() + Source.size());
    Result.resize(UNI_MAX_UTF8_BYTES_PER_CODE_POINT * Source.size());
    UTF8 *ResultPtr = reinterpret_cast<UTF8 *>(&Result[0]);
    UTF8 *ResultEnd = reinterpret_cast<UTF8 *>(&Result[0] + Result.size());
    if (ConvertUTF32toUTF8(&Start, End, &ResultPtr, ResultEnd,
                           strictConversion) == conversionOK) {
      Result.resize(reinterpret_cast<char *>(ResultPtr) - &Result[0]);
      return true;
    } else {
      Result.clear();
      return false;
    }
  } else {
    llvm_unreachable(
        "Control should never reach this point; see static_assert further up");
  }
}

bool IsSingleCodeUnitUTF8Codepoint(unsigned V) { return V <= 0x7F; }

bool IsSingleCodeUnitUTF16Codepoint(unsigned V) {
  return V <= 0xD7FF || (V >= 0xE000 && V <= 0xFFFF);
}

bool IsSingleCodeUnitUTF32Codepoint(unsigned V) {
  return V <= 0xD7FF || (V >= 0xE000 && V <= 0x10FFFF);
}

} // end namespace llvm

```
- EN: This section centers on `convertUTF16ToUTF8String`, `IsSingleCodeUnitUTF8Codepoint`, `IsSingleCodeUnitUTF16Codepoint` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `convertUTF16ToUTF8String`, `IsSingleCodeUnitUTF8Codepoint`, `IsSingleCodeUnitUTF16Codepoint` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `ConvertUTF8toWide`, `assert`, `memcpy`, `ConvertUTF8toUTF16` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/ADT/ArrayRef.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/bit.h`, `llvm/Support/ConvertUTF.h`, `llvm/Support/ErrorHandling.h`
- Standard library / 标准库: `string`, `vector`
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `ConvertUTF8toWide`, `assert`, `memcpy`, `ConvertUTF8toUTF16`, `ConvertUTF8toUTF32`
