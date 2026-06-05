# TextEncoding.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/TextEncoding.cpp`
- Repository: `llvm-project`
- Purpose (EN): / \file / This file provides utility classes to convert between different character / encodings.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `TextEncoding` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===-- TextEncoding.cpp - Text encoding conversion class ---------*- C++ -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This file provides utility classes to convert between different character
/// encodings.
///
//===----------------------------------------------------------------------===//

#include "llvm/Support/TextEncoding.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/Support/ConvertEBCDIC.h"
#include <system_error>

#if HAVE_ICU
#if HAVE_WINDOWS_ICU
#include <icu.h>
#else
#include <unicode/ucnv.h>
#endif
#elif HAVE_ICONV
#include <iconv.h>
#endif

using namespace llvm;

// Normalize the charset name with the charset alias matching algorithm proposed
// in https://www.unicode.org/reports/tr22/tr22-8.html#Charset_Alias_Matching.
static void normalizeCharSetName(StringRef CSName,
                                 SmallVectorImpl<char> &Normalized) {
  bool PrevDigit = false;
  for (auto Ch : CSName) {
    if (isAlnum(Ch)) {
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 9 direct dependencies, including `llvm/Support/TextEncoding.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringExtras.h`.
  CN: 引入了 9 个直接依赖，其中包括 `llvm/Support/TextEncoding.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringExtras.h`。
- EN: This section centers on `normalizeCharSetName` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `normalizeCharSetName` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 41-80

```cpp
      Ch = toLower(Ch);
      if (Ch != '0' || PrevDigit) {
        PrevDigit = isDigit(Ch);
        Normalized.push_back(Ch);
      }
    }
  }
}

// Maps the encoding name to enum constant if possible.
static std::optional<TextEncoding> getKnownEncoding(StringRef Name) {
  SmallString<16> Normalized;
  normalizeCharSetName(Name, Normalized);
  if (Normalized.equals("utf8"))
    return TextEncoding::UTF8;
  if (Normalized.equals("ibm1047"))
    return TextEncoding::IBM1047;
  return std::nullopt;
}

[[maybe_unused]] static void HandleOverflow(size_t &Capacity, char *&Output,
                                            size_t &OutputLength,
                                            SmallVectorImpl<char> &Result) {
  // No space left in output buffer. Double the size of the underlying
  // memory in the SmallVectorImpl, adjust pointer and length and continue
  // the conversion.
  Capacity =
      (Capacity < Result.max_size() / 2) ? 2 * Capacity : Result.max_size();
  Result.resize(0);
  Result.resize_for_overwrite(Capacity);
  Output = static_cast<char *>(Result.data());
  OutputLength = Capacity;
}

namespace {
enum ConversionType {
  UTF8ToIBM1047,
  IBM1047ToUTF8,
};

```
- EN: This section centers on `getKnownEncoding`, `normalizeCharSetName` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getKnownEncoding`, `normalizeCharSetName` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and updates helper containers and temporary state.
  CN: 在这一段中，代码检查条件并处理边界情况，并更新辅助容器和临时状态。

### Lines 81-120

```cpp
// Support conversion between EBCDIC 1047 and UTF-8. This class uses
// built-in translation tables that allow for translation between the
// aforementioned encodings. The use of tables for conversion is only
// possible because EBCDIC 1047 is a single-byte, stateless encoding; other
// encodings are not supported.
class TextEncodingConverterTable final
    : public details::TextEncodingConverterImplBase {
  const ConversionType ConvType;

public:
  TextEncodingConverterTable(ConversionType ConvType) : ConvType(ConvType) {}

  std::error_code convertString(StringRef Source,
                                SmallVectorImpl<char> &Result) override;

  void reset() override {}
};

std::error_code
TextEncodingConverterTable::convertString(StringRef Source,
                                          SmallVectorImpl<char> &Result) {
  switch (ConvType) {
  case IBM1047ToUTF8:
    ConverterEBCDIC::convertToUTF8(Source, Result);
    return std::error_code();
  case UTF8ToIBM1047:
    return ConverterEBCDIC::convertToEBCDIC(Source, Result);
  }
  llvm_unreachable("Invalid ConvType!");
  return std::error_code();
}

#if HAVE_ICU
struct UConverterDeleter {
  void operator()(UConverter *Converter) const {
    if (Converter)
      ucnv_close(Converter);
  }
};
using UConverterUniquePtr = std::unique_ptr<UConverter, UConverterDeleter>;
```
- EN: This section centers on `TextEncodingConverterTable`, `convertString`, `convertToUTF8` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `TextEncodingConverterTable`, `convertString`, `convertToUTF8` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and checks conditions and handles edge cases.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并检查条件并处理边界情况。

### Lines 121-160

```cpp

class TextEncodingConverterICU final
    : public details::TextEncodingConverterImplBase {
  UConverterUniquePtr FromConvDesc;
  UConverterUniquePtr ToConvDesc;

public:
  TextEncodingConverterICU(UConverterUniquePtr FromConverter,
                           UConverterUniquePtr ToConverter)
      : FromConvDesc(std::move(FromConverter)),
        ToConvDesc(std::move(ToConverter)) {}

  std::error_code convertString(StringRef Source,
                                SmallVectorImpl<char> &Result) override;

  void reset() override;
};

// TODO: The current implementation discards the partial result and restarts the
// conversion from the beginning if there is a conversion error due to
// insufficient buffer size. In the future, it would better to save the partial
// result and resume the conversion for the remaining string.
// TODO: Improve translation of ICU errors to error_code
std::error_code
TextEncodingConverterICU::convertString(StringRef Source,
                                        SmallVectorImpl<char> &Result) {
  // Setup the input in case it has no backing data.
  size_t InputLength = Source.size();
  const char *In = InputLength ? const_cast<char *>(Source.data()) : "";

  // Setup the output. We directly write into the SmallVector.
  size_t Capacity = Result.capacity();
  size_t OutputLength = Capacity;
  Result.resize_for_overwrite(Capacity);
  char *Output;
  UErrorCode EC = U_ZERO_ERROR;

  ucnv_setToUCallBack(&*FromConvDesc, UCNV_TO_U_CALLBACK_STOP, NULL, NULL, NULL,
                      &EC);
  ucnv_setFromUCallBack(&*ToConvDesc, UCNV_FROM_U_CALLBACK_STOP, NULL, NULL,
```
- EN: This section centers on `TextEncodingConverterICU`, `convertString`, `ucnv_setToUCallBack` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `TextEncodingConverterICU`, `convertString`, `ucnv_setToUCallBack` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code propagates LLVM-style errors and invariants and updates helper containers and temporary state.
  CN: 在这一段中，代码传播 LLVM 风格的错误处理并维护不变量，并更新辅助容器和临时状态。

### Lines 161-200

```cpp
                        NULL, &EC);
  assert(U_SUCCESS(EC));

  do {
    EC = U_ZERO_ERROR;
    const char *Input = In;

    Output = static_cast<char *>(Result.data());
    ucnv_convertEx(&*ToConvDesc, &*FromConvDesc, &Output, Result.end(), &Input,
                   In + InputLength, /*pivotStart=*/NULL,
                   /*pivotSource=*/NULL, /*pivotTarget=*/NULL,
                   /*pivotLimit=*/NULL, /*reset=*/true,
                   /*flush=*/true, &EC);
    if (U_FAILURE(EC)) {
      if (EC == U_BUFFER_OVERFLOW_ERROR) {
        if (Capacity < Result.max_size()) {
          HandleOverflow(Capacity, Output, OutputLength, Result);
          continue;
        } else {
          Result.resize(Output - Result.data());
          return std::error_code(E2BIG, std::generic_category());
        }
      }
      // Some other error occurred.
      Result.resize(Output - Result.data());
      return std::error_code(EILSEQ, std::generic_category());
    }
    break;
  } while (true);

  Result.resize(Output - Result.data());
  return std::error_code();
}

void TextEncodingConverterICU::reset() {
  ucnv_reset(&*FromConvDesc);
  ucnv_reset(&*ToConvDesc);
}

#elif HAVE_ICONV
```
- EN: This section centers on `assert`, `ucnv_convertEx`, `HandleOverflow` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `assert`, `ucnv_convertEx`, `HandleOverflow` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 201-240

```cpp
class TextEncodingConverterIconv final
    : public details::TextEncodingConverterImplBase {
  class UniqueIconvT {
    iconv_t ConvDesc;

  public:
    operator iconv_t() const { return ConvDesc; }
    UniqueIconvT(iconv_t CD) : ConvDesc(CD) {}
    ~UniqueIconvT() {
      if (ConvDesc != (iconv_t)-1) {
        iconv_close(ConvDesc);
        ConvDesc = (iconv_t)-1;
      }
    }
    UniqueIconvT(UniqueIconvT &&Other) : ConvDesc(Other.ConvDesc) {
      Other.ConvDesc = (iconv_t)-1;
    }
    UniqueIconvT &operator=(UniqueIconvT &&Other) {
      if (&Other != this) {
        ConvDesc = Other.ConvDesc;
        Other.ConvDesc = (iconv_t)-1;
      }
      return *this;
    }
  };
  UniqueIconvT ConvDesc;

public:
  TextEncodingConverterIconv(UniqueIconvT ConvDesc)
      : ConvDesc(std::move(ConvDesc)) {}

  std::error_code convertString(StringRef Source,
                                SmallVectorImpl<char> &Result) override;

  void reset() override;
};

// TODO: The current implementation discards the partial result and restarts the
// conversion from the beginning if there is a conversion error due to
// insufficient buffer size. In the future, it would better to save the partial
```
- EN: This section centers on `iconv_t`, `UniqueIconvT`, `~UniqueIconvT` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `iconv_t`, `UniqueIconvT`, `~UniqueIconvT` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and updates helper containers and temporary state.
  CN: 在这一段中，代码检查条件并处理边界情况，并更新辅助容器和临时状态。

### Lines 241-280

```cpp
// result and resume the conversion for the remaining string.
std::error_code
TextEncodingConverterIconv::convertString(StringRef Source,
                                          SmallVectorImpl<char> &Result) {
  // Setup the output. We directly write into the SmallVector.
  size_t Capacity = Result.capacity();
  char *Output = static_cast<char *>(Result.data());
  size_t OutputLength = Capacity;
  Result.resize_for_overwrite(Capacity);

  size_t Ret;
  // Handle errors returned from iconv().
  auto HandleError = [&Capacity, &Output, &OutputLength, &Result,
                      this](size_t Ret) {
    if (Ret == static_cast<size_t>(-1)) {
      // An error occurred. Check if we can gracefully handle it.
      if (errno == E2BIG && Capacity < Result.max_size()) {
        HandleOverflow(Capacity, Output, OutputLength, Result);
        // Reset converter
        reset();
        return std::error_code();
      } else {
        // Some other error occurred.
        Result.resize(Output - Result.data());
        return std::error_code(errno, std::generic_category());
      }
    } else {
      // A positive return value indicates that some characters were converted
      // in a nonreversible way, that is, replaced with a SUB symbol. Returning
      // an error in this case makes sure that both conversion routines behave
      // in the same way.
      return std::make_error_code(std::errc::illegal_byte_sequence);
    }
  };

  do {
    size_t InputLength = Source.size();
    char *Input = const_cast<char *>(Source.data());
    Ret = iconv(ConvDesc, &Input, &InputLength, &Output, &OutputLength);
    if (Ret != 0) {
```
- EN: This section centers on `convertString`, `HandleOverflow`, `reset` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `convertString`, `HandleOverflow`, `reset` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 281-320

```cpp
      if (auto EC = HandleError(Ret))
        return EC;
      continue;
    }
    // Flush the converter
    Ret = iconv(ConvDesc, nullptr, nullptr, &Output, &OutputLength);
    if (Ret != 0) {
      if (auto EC = HandleError(Ret))
        return EC;
      continue;
    }
    break;
  } while (true);

  // Re-adjust size to actual size.
  Result.resize(Output - Result.data());
  return std::error_code();
}

inline void TextEncodingConverterIconv::reset() {
  iconv(ConvDesc, nullptr, nullptr, nullptr, nullptr);
}

#endif // HAVE_ICONV
} // namespace

ErrorOr<TextEncodingConverter>
TextEncodingConverter::create(TextEncoding CPFrom, TextEncoding CPTo) {

  // Text encodings should be distinct.
  if (CPFrom == CPTo)
    return std::make_error_code(std::errc::invalid_argument);

  ConversionType Conversion;
  if (CPFrom == TextEncoding::UTF8 && CPTo == TextEncoding::IBM1047)
    Conversion = UTF8ToIBM1047;
  else if (CPFrom == TextEncoding::IBM1047 && CPTo == TextEncoding::UTF8)
    Conversion = IBM1047ToUTF8;
  else
    return std::make_error_code(std::errc::invalid_argument);
```
- EN: This section centers on `error_code`, `reset`, `iconv` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `error_code`, `reset`, `iconv` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 321-358

```cpp

  return TextEncodingConverter(
      std::make_unique<TextEncodingConverterTable>(Conversion));
}

ErrorOr<TextEncodingConverter> TextEncodingConverter::create(StringRef From,
                                                             StringRef To) {
  std::optional<TextEncoding> FromEncoding = getKnownEncoding(From);
  std::optional<TextEncoding> ToEncoding = getKnownEncoding(To);
  if (FromEncoding && ToEncoding) {
    ErrorOr<TextEncodingConverter> Converter =
        create(*FromEncoding, *ToEncoding);
    if (Converter)
      return Converter;
  }
#if HAVE_ICU
  UErrorCode EC = U_ZERO_ERROR;
  UConverterUniquePtr FromConvDesc(ucnv_open(From.str().c_str(), &EC));
  if (U_FAILURE(EC))
    return std::make_error_code(std::errc::invalid_argument);

  UConverterUniquePtr ToConvDesc(ucnv_open(To.str().c_str(), &EC));
  if (U_FAILURE(EC))
    return std::make_error_code(std::errc::invalid_argument);

  auto Converter = std::make_unique<TextEncodingConverterICU>(
      std::move(FromConvDesc), std::move(ToConvDesc));
  return TextEncodingConverter(std::move(Converter));
#elif HAVE_ICONV
  iconv_t ConvDesc = iconv_open(To.str().c_str(), From.str().c_str());
  if (ConvDesc == (iconv_t)-1)
    return std::make_error_code(std::errc::invalid_argument);
  return TextEncodingConverter(
      std::make_unique<TextEncodingConverterIconv>(ConvDesc));
#else
  return std::make_error_code(std::errc::invalid_argument);
#endif
}
```
- EN: This section centers on `TextEncodingConverter`, `create`, `FromConvDesc` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `TextEncodingConverter`, `create`, `FromConvDesc` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `constant`, `ConversionType`, `uses`, `normalizeCharSetName`, `getKnownEncoding`, `TextEncodingConverterTable`, `convertString` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Formatting, emission, and outward serialization. / 重点关注格式化、输出与序列化。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/TextEncoding.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringExtras.h`, `llvm/Support/ConvertEBCDIC.h`
- Standard library / 标准库: `system_error`
- Other/system headers / 其他或系统头文件: `icu.h`, `unicode/ucnv.h`, `iconv.h`
- Related symbols / 相关符号: `constant`, `ConversionType`, `uses`, `TextEncodingConverterTable`, `UConverterDeleter`, `normalizeCharSetName`, `getKnownEncoding`, `convertString`, `convertToUTF8`
