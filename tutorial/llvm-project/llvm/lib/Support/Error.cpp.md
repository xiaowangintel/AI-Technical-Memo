# Error.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/Error.cpp`
- Repository: `llvm-project`
- Purpose (EN): FIXME: This class is only here to support the transition to llvm::Error.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `Error` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===----- lib/Support/Error.cpp - Error and associated utilities ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/Error.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/ADT/Twine.h"
#include "llvm/Support/ErrorHandling.h"
#include <system_error>

using namespace llvm;

namespace {

enum class ErrorErrorCode : int {
  MultipleErrors = 1,
  FileError,
  InconvertibleError
};

// FIXME: This class is only here to support the transition to llvm::Error. It
// will be removed once this transition is complete. Clients should prefer to
// deal with the Error value directly, rather than converting to error_code.
class ErrorErrorCategory : public std::error_category {
public:
  const char *name() const noexcept override { return "Error"; }

  std::string message(int condition) const override {
    switch (static_cast<ErrorErrorCode>(condition)) {
    case ErrorErrorCode::MultipleErrors:
      return "Multiple errors";
    case ErrorErrorCode::InconvertibleError:
      return "Inconvertible error value. An error has occurred that could not "
             "be converted to a known std::error_code. Please file a bug.";
    case ErrorErrorCode::FileError:
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 6 direct dependencies, including `llvm/Support/Error.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/Twine.h`.
  CN: 引入了 6 个直接依赖，其中包括 `llvm/Support/Error.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/Twine.h`。
- EN: This range defines or extends data types such as `ErrorErrorCode`, `is`, `ErrorErrorCategory`.
  CN: 这一段定义或扩展了 `ErrorErrorCode`, `is`, `ErrorErrorCategory` 等数据类型。

### Lines 41-80

```cpp
      return "A file error occurred.";
    }
    llvm_unreachable("Unhandled error code");
  }
};
}

static ErrorErrorCategory &getErrorErrorCat() {
  static ErrorErrorCategory ErrorErrorCat;
  return ErrorErrorCat;
}

void ErrorInfoBase::anchor() {}
char ErrorInfoBase::ID = 0;
char ErrorList::ID = 0;
void ECError::anchor() {}
char ECError::ID = 0;
char StringError::ID = 0;
char FileError::ID = 0;

void llvm::logAllUnhandledErrors(Error E, raw_ostream &OS, Twine ErrorBanner) {
  if (!E)
    return;
  OS << ErrorBanner;
  handleAllErrors(std::move(E), [&](const ErrorInfoBase &EI) {
    EI.log(OS);
    OS << "\n";
  });
}

/// Write all error messages (if any) in E to a string. The newline character
/// is used to separate error messages.
std::string llvm::toString(Error E) {
  SmallVector<std::string, 2> Errors;
  handleAllErrors(std::move(E), [&Errors](const ErrorInfoBase &EI) {
    Errors.push_back(EI.message());
  });
  return join(Errors.begin(), Errors.end(), "\n");
}

```
- EN: This section centers on `llvm_unreachable`, `anchor`, `logAllUnhandledErrors` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `llvm_unreachable`, `anchor`, `logAllUnhandledErrors` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 81-120

```cpp
std::string llvm::toStringWithoutConsuming(const Error &E) {
  SmallVector<std::string, 2> Errors;
  visitErrors(E, [&Errors](const ErrorInfoBase &EI) {
    Errors.push_back(EI.message());
  });
  return join(Errors.begin(), Errors.end(), "\n");
}

std::error_code ErrorList::convertToErrorCode() const {
  return std::error_code(static_cast<int>(ErrorErrorCode::MultipleErrors),
                         getErrorErrorCat());
}

std::error_code llvm::inconvertibleErrorCode() {
  return std::error_code(static_cast<int>(ErrorErrorCode::InconvertibleError),
                         getErrorErrorCat());
}

std::error_code FileError::convertToErrorCode() const {
  std::error_code NestedEC = Err->convertToErrorCode();
  if (NestedEC == inconvertibleErrorCode())
    return std::error_code(static_cast<int>(ErrorErrorCode::FileError),
                           getErrorErrorCat());
  return NestedEC;
}

Error llvm::errorCodeToError(std::error_code EC) {
  if (!EC)
    return Error::success();
  return Error(std::make_unique<ECError>(ECError(EC)));
}

std::error_code llvm::errorToErrorCode(Error Err) {
  std::error_code EC;
  handleAllErrors(std::move(Err), [&](const ErrorInfoBase &EI) {
    EC = EI.convertToErrorCode();
  });
  if (EC == inconvertibleErrorCode())
    report_fatal_error(Twine(EC.message()));
  return EC;
```
- EN: This section centers on `toStringWithoutConsuming`, `visitErrors`, `join` and walks over records and applies per-item processing.
  CN: 这一段主要围绕 `toStringWithoutConsuming`, `visitErrors`, `join` 等符号展开，负责遍历记录并执行逐项处理。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-160

```cpp
}

#if LLVM_ENABLE_ABI_BREAKING_CHECKS
void Error::fatalUncheckedError() const {
  dbgs() << "Program aborted due to an unhandled Error:\n";
  if (getPtr()) {
    getPtr()->log(dbgs());
    dbgs() << "\n";
  }else
    dbgs() << "Error value was Success. (Note: Success values must still be "
              "checked prior to being destroyed).\n";
  abort();
}
#endif

StringError::StringError(std::error_code EC, const Twine &S)
    : Msg(S.str()), EC(EC) {}

StringError::StringError(const Twine &S, std::error_code EC)
    : Msg(S.str()), EC(EC), PrintMsgOnly(true) {}

StringError::StringError(std::string &&S, std::error_code EC, bool PrintMsgOnly)
    : Msg(std::move(S)), EC(EC), PrintMsgOnly(PrintMsgOnly) {}

void StringError::log(raw_ostream &OS) const {
  if (PrintMsgOnly) {
    OS << Msg;
  } else {
    OS << EC.message();
    if (!Msg.empty())
      OS << (" " + Msg);
  }
}

std::error_code StringError::convertToErrorCode() const {
  return EC;
}

Error llvm::createStringError(std::string &&Msg, std::error_code EC) {
  return make_error<StringError>(std::move(Msg), EC, true);
```
- EN: This section centers on `fatalUncheckedError`, `getPtr`, `abort` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `fatalUncheckedError`, `getPtr`, `abort` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 161-200

```cpp
}

void llvm::report_fatal_error(Error Err, bool GenCrashDiag) {
  assert(Err && "report_fatal_error called with success value");
  std::string ErrMsg;
  {
    raw_string_ostream ErrStream(ErrMsg);
    logAllUnhandledErrors(std::move(Err), ErrStream);
  }
  report_fatal_error(Twine(ErrMsg), GenCrashDiag);
}

void llvm::reportFatalInternalError(Error Err) {
  report_fatal_error(std::move(Err), /*GenCrashDiag=*/true);
}

void llvm::reportFatalUsageError(Error Err) {
  report_fatal_error(std::move(Err), /*GenCrashDiag=*/false);
}

LLVMErrorTypeId LLVMGetErrorTypeId(LLVMErrorRef Err) {
  return reinterpret_cast<ErrorInfoBase *>(Err)->dynamicClassID();
}

void LLVMConsumeError(LLVMErrorRef Err) { consumeError(unwrap(Err)); }

void LLVMCantFail(LLVMErrorRef Err) {
  cantFail(unwrap(Err));
}

char *LLVMGetErrorMessage(LLVMErrorRef Err) {
  std::string Tmp = toString(unwrap(Err));
  char *ErrMsg = new char[Tmp.size() + 1];
  memcpy(ErrMsg, Tmp.data(), Tmp.size());
  ErrMsg[Tmp.size()] = '\0';
  return ErrMsg;
}

void LLVMDisposeErrorMessage(char *ErrMsg) { delete[] ErrMsg; }

```
- EN: This section centers on `report_fatal_error`, `assert`, `ErrStream` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `report_fatal_error`, `assert`, `ErrStream` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code propagates LLVM-style errors and invariants and returns the resulting value to its callers.
  CN: 在这一段中，代码传播 LLVM 风格的错误处理并维护不变量，并将结果返回给调用方。

### Lines 201-207

```cpp
LLVMErrorTypeId LLVMGetStringErrorTypeId() {
  return reinterpret_cast<void *>(&StringError::ID);
}

LLVMErrorRef LLVMCreateStringError(const char *ErrMsg) {
  return wrap(make_error<StringError>(ErrMsg, inconvertibleErrorCode()));
}
```
- EN: This section centers on `LLVMGetStringErrorTypeId`, `LLVMCreateStringError`, `wrap` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `LLVMGetStringErrorTypeId`, `LLVMCreateStringError`, `wrap` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code propagates LLVM-style errors and invariants and returns the resulting value to its callers.
  CN: 在这一段中，代码传播 LLVM 风格的错误处理并维护不变量，并将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `ErrorErrorCode`, `is`, `ErrorErrorCategory`, `llvm_unreachable`, `anchor`, `logAllUnhandledErrors`, `handleAllErrors` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Formatting, emission, and outward serialization. / 重点关注格式化、输出与序列化。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/Error.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/Twine.h`, `llvm/Support/ErrorHandling.h`
- Standard library / 标准库: `system_error`
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `ErrorErrorCode`, `is`, `ErrorErrorCategory`, `llvm_unreachable`, `anchor`, `logAllUnhandledErrors`, `handleAllErrors`, `toString`
