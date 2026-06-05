# NativeFormatting.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/NativeFormatting.cpp`
- Repository: `llvm-project`
- Purpose (EN): --CurPtr = '0' + char(Value % 10);
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `NativeFormatting` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===- NativeFormatting.cpp - Low level formatting helpers -------*- C++-*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/NativeFormatting.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/Support/Format.h"
#include "llvm/Support/raw_ostream.h"

#include <cmath>

#if defined(_WIN32) && !defined(__MINGW32__)
#include <float.h> // For _fpclass in llvm::write_double.
#endif

using namespace llvm;

template<typename T, std::size_t N>
static int format_to_buffer(T Value, char (&Buffer)[N]) {
  char *EndPtr = std::end(Buffer);
  char *CurPtr = EndPtr;

  do {
    *--CurPtr = '0' + char(Value % 10);
    Value /= 10;
  } while (Value);
  return EndPtr - CurPtr;
}

static void writeWithCommas(raw_ostream &S, ArrayRef<char> Buffer) {
  assert(!Buffer.empty());

  ArrayRef<char> ThisGroup;
  int InitialDigits = ((Buffer.size() - 1) % 3) + 1;
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 8 direct dependencies, including `llvm/Support/NativeFormatting.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/StringExtras.h`.
  CN: 引入了 8 个直接依赖，其中包括 `llvm/Support/NativeFormatting.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/StringExtras.h`。
- EN: This section centers on `format_to_buffer`, `writeWithCommas`, `assert` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `format_to_buffer`, `writeWithCommas`, `assert` 等符号展开，负责为下游使用方写出、格式化或序列化结果。

### Lines 41-80

```cpp
  ThisGroup = Buffer.take_front(InitialDigits);
  S.write(ThisGroup.data(), ThisGroup.size());

  Buffer = Buffer.drop_front(InitialDigits);
  assert(Buffer.size() % 3 == 0);
  while (!Buffer.empty()) {
    S << ',';
    ThisGroup = Buffer.take_front(3);
    S.write(ThisGroup.data(), 3);
    Buffer = Buffer.drop_front(3);
  }
}

template <typename T>
static void write_unsigned_impl(raw_ostream &S, T N, size_t MinDigits,
                                IntegerStyle Style, bool IsNegative,
                                bool NonNegativePlus) {
  static_assert(std::is_unsigned_v<T>, "Value is not unsigned!");

  char NumberBuffer[128];
  size_t Len = format_to_buffer(N, NumberBuffer);

  if (IsNegative)
    S << '-';
  else if (NonNegativePlus)
    S << '+';

  if (Len < MinDigits && Style != IntegerStyle::Number) {
    for (size_t I = Len; I < MinDigits; ++I)
      S << '0';
  }

  if (Style == IntegerStyle::Number) {
    writeWithCommas(S, ArrayRef<char>(std::end(NumberBuffer) - Len, Len));
  } else {
    S.write(std::end(NumberBuffer) - Len, Len);
  }
}

template <typename T>
```
- EN: This section centers on `assert`, `write_unsigned_impl`, `static_assert` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `assert`, `write_unsigned_impl`, `static_assert` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 81-120

```cpp
static void write_unsigned(raw_ostream &S, T N, size_t MinDigits,
                           IntegerStyle Style, bool IsNegative = false,
                           bool NonNegativePlus = false) {
  // Output using 32-bit div/mod if possible.
  if (N == static_cast<uint32_t>(N))
    write_unsigned_impl(S, static_cast<uint32_t>(N), MinDigits, Style,
                        IsNegative, NonNegativePlus);
  else
    write_unsigned_impl(S, N, MinDigits, Style, IsNegative, NonNegativePlus);
}

template <typename T>
static void write_signed(raw_ostream &S, T N, size_t MinDigits,
                         IntegerStyle Style, bool NonNegativePlus = false) {
  static_assert(std::is_signed_v<T>, "Value is not signed!");

  using UnsignedT = std::make_unsigned_t<T>;

  if (N >= 0) {
    write_unsigned(S, static_cast<UnsignedT>(N), MinDigits, Style, false,
                   NonNegativePlus);
    return;
  }

  UnsignedT UN = -(UnsignedT)N;
  write_unsigned(S, UN, MinDigits, Style, true, NonNegativePlus);
}

void llvm::write_integer(raw_ostream &S, unsigned int N, size_t MinDigits,
                         IntegerStyle Style, bool NonNegativePlus) {
  write_unsigned(S, N, MinDigits, Style, false, NonNegativePlus);
}

void llvm::write_integer(raw_ostream &S, int N, size_t MinDigits,
                         IntegerStyle Style, bool NonNegativePlus) {
  write_signed(S, N, MinDigits, Style, NonNegativePlus);
}

void llvm::write_integer(raw_ostream &S, unsigned long N, size_t MinDigits,
                         IntegerStyle Style, bool NonNegativePlus) {
```
- EN: This section centers on `write_unsigned`, `write_unsigned_impl`, `write_signed` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `write_unsigned`, `write_unsigned_impl`, `write_signed` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-160

```cpp
  write_unsigned(S, N, MinDigits, Style, false, NonNegativePlus);
}

void llvm::write_integer(raw_ostream &S, long N, size_t MinDigits,
                         IntegerStyle Style, bool NonNegativePlus) {
  write_signed(S, N, MinDigits, Style, NonNegativePlus);
}

void llvm::write_integer(raw_ostream &S, unsigned long long N, size_t MinDigits,
                         IntegerStyle Style, bool NonNegativePlus) {
  write_unsigned(S, N, MinDigits, Style, false, NonNegativePlus);
}

void llvm::write_integer(raw_ostream &S, long long N, size_t MinDigits,
                         IntegerStyle Style, bool NonNegativePlus) {
  write_signed(S, N, MinDigits, Style, NonNegativePlus);
}

void llvm::write_hex(raw_ostream &S, uint64_t N, HexPrintStyle Style,
                     std::optional<size_t> Width) {
  const size_t kMaxWidth = 128u;

  size_t W = std::min(kMaxWidth, Width.value_or(0u));

  unsigned Nibbles = (llvm::bit_width(N) + 3) / 4;
  bool Prefix = (Style == HexPrintStyle::PrefixLower ||
                 Style == HexPrintStyle::PrefixUpper);
  bool Upper =
      (Style == HexPrintStyle::Upper || Style == HexPrintStyle::PrefixUpper);
  unsigned PrefixChars = Prefix ? 2 : 0;
  unsigned NumChars =
      std::max(static_cast<unsigned>(W), std::max(1u, Nibbles) + PrefixChars);

  char NumberBuffer[kMaxWidth];
  ::memset(NumberBuffer, '0', std::size(NumberBuffer));
  if (Prefix)
    NumberBuffer[1] = 'x';
  char *EndPtr = NumberBuffer + NumChars;
  char *CurPtr = EndPtr;
  while (N) {
```
- EN: This section centers on `write_unsigned`, `write_integer`, `write_signed` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `write_unsigned`, `write_integer`, `write_signed` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 161-200

```cpp
    unsigned char x = static_cast<unsigned char>(N) % 16;
    *--CurPtr = hexdigit(x, !Upper);
    N /= 16;
  }

  S.write(NumberBuffer, NumChars);
}

void llvm::write_double(raw_ostream &S, double N, FloatStyle Style,
                        std::optional<size_t> Precision) {
  size_t Prec = Precision.value_or(getDefaultPrecision(Style));

  if (std::isnan(N)) {
    S << "nan";
    return;
  } else if (std::isinf(N)) {
    S << (std::signbit(N) ? "-INF" : "INF");
    return;
  }

  char Letter;
  if (Style == FloatStyle::Exponent)
    Letter = 'e';
  else if (Style == FloatStyle::ExponentUpper)
    Letter = 'E';
  else
    Letter = 'f';

  SmallString<8> Spec;
  llvm::raw_svector_ostream Out(Spec);
  Out << "%." << Prec << Letter;

  if (Style == FloatStyle::Exponent || Style == FloatStyle::ExponentUpper) {
#ifdef _WIN32
// On MSVCRT and compatible, output of %e is incompatible to Posix
// by default. Number of exponent digits should be at least 2. "%+03d"
// FIXME: Implement our formatter to here or Support/Format.h!
#if defined(__MINGW32__)
    // FIXME: It should be generic to C++11.
    if (N == 0.0 && std::signbit(N)) {
```
- EN: This section centers on `write_double`, `Out` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `write_double`, `Out` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 201-240

```cpp
      char NegativeZero[] = "-0.000000e+00";
      if (Style == FloatStyle::ExponentUpper)
        NegativeZero[strlen(NegativeZero) - 4] = 'E';
      S << NegativeZero;
      return;
    }
#else
    int fpcl = _fpclass(N);

    // negative zero
    if (fpcl == _FPCLASS_NZ) {
      char NegativeZero[] = "-0.000000e+00";
      if (Style == FloatStyle::ExponentUpper)
        NegativeZero[strlen(NegativeZero) - 4] = 'E';
      S << NegativeZero;
      return;
    }
#endif

    char buf[32];
    unsigned len;
    len = format(Spec.c_str(), N).snprint(buf, sizeof(buf));
    if (len <= sizeof(buf) - 2) {
      if (len >= 5 && (buf[len - 5] == 'e' || buf[len - 5] == 'E') &&
          buf[len - 3] == '0') {
        int cs = buf[len - 4];
        if (cs == '+' || cs == '-') {
          int c1 = buf[len - 2];
          int c0 = buf[len - 1];
          if (isdigit(static_cast<unsigned char>(c1)) &&
              isdigit(static_cast<unsigned char>(c0))) {
            // Trim leading '0': "...e+012" -> "...e+12\0"
            buf[len - 3] = c1;
            buf[len - 2] = c0;
            buf[--len] = 0;
          }
        }
      }
      S << buf;
      return;
```
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 241-269

```cpp
    }
#endif
  }

  if (Style == FloatStyle::Percent)
    N *= 100.0;

  char Buf[32];
  format(Spec.c_str(), N).snprint(Buf, sizeof(Buf));
  S << Buf;
  if (Style == FloatStyle::Percent)
    S << '%';
}

bool llvm::isPrefixedHexStyle(HexPrintStyle S) {
  return (S == HexPrintStyle::PrefixLower || S == HexPrintStyle::PrefixUpper);
}

size_t llvm::getDefaultPrecision(FloatStyle Style) {
  switch (Style) {
  case FloatStyle::Exponent:
  case FloatStyle::ExponentUpper:
    return 6; // Number of decimal places.
  case FloatStyle::Fixed:
  case FloatStyle::Percent:
    return 2; // Number of decimal places.
  }
  llvm_unreachable("Unknown FloatStyle enum");
}
```
- EN: This section centers on `format`, `isPrefixedHexStyle`, `getDefaultPrecision` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `format`, `isPrefixedHexStyle`, `getDefaultPrecision` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and checks conditions and handles edge cases.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并检查条件并处理边界情况。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `format_to_buffer`, `writeWithCommas`, `assert`, `write_unsigned_impl` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Formatting, emission, and outward serialization. / 重点关注格式化、输出与序列化。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/NativeFormatting.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/StringExtras.h`, `llvm/Support/Format.h`, `llvm/Support/raw_ostream.h`
- Standard library / 标准库: `cmath`
- Other/system headers / 其他或系统头文件: `float.h`
- Related symbols / 相关符号: `format_to_buffer`, `writeWithCommas`, `assert`, `write_unsigned_impl`, `static_assert`
