# AutoConvert.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/AutoConvert.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains functions used for auto conversion between ASCII/EBCDIC codepages specific to z/OS.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `AutoConvert` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===- AutoConvert.cpp - Auto conversion between ASCII/EBCDIC -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains functions used for auto conversion between
// ASCII/EBCDIC codepages specific to z/OS.
//
//===----------------------------------------------------------------------===//

#ifdef __MVS__

#include "llvm/Support/AutoConvert.h"
#include <cassert>
#include <fcntl.h>
#include <sys/stat.h>
#include <unistd.h>

using namespace llvm;

static int savedStdHandleAutoConversionMode[3] = {-1, -1, -1};

int disablezOSAutoConversion(int FD) {
  static const struct f_cnvrt Convert = {
      SETCVTOFF, // cvtcmd
      0,         // pccsid
      0,         // fccsid
  };

  return fcntl(FD, F_CONTROL_CVT, &Convert);
}

int restorezOSStdHandleAutoConversion(int FD) {
  assert(FD == STDIN_FILENO || FD == STDOUT_FILENO || FD == STDERR_FILENO);
  if (savedStdHandleAutoConversionMode[FD] == -1)
    return 0;
  struct f_cnvrt Cvt = {
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 5 direct dependencies, including `llvm/Support/AutoConvert.h`, `cassert`, `fcntl.h`, `sys/stat.h`.
  CN: 引入了 5 个直接依赖，其中包括 `llvm/Support/AutoConvert.h`, `cassert`, `fcntl.h`, `sys/stat.h`。
- EN: This section centers on `disablezOSAutoConversion`, `fcntl`, `restorezOSStdHandleAutoConversion` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `disablezOSAutoConversion`, `fcntl`, `restorezOSStdHandleAutoConversion` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 41-80

```cpp
      savedStdHandleAutoConversionMode[FD], // cvtcmd
      0,                                    // pccsid
      0,                                    // fccsid
  };
  return (fcntl(FD, F_CONTROL_CVT, &Cvt));
}

int enablezOSAutoConversion(int FD) {
  struct f_cnvrt Query = {
      QUERYCVT, // cvtcmd
      0,        // pccsid
      0,        // fccsid
  };

  if (fcntl(FD, F_CONTROL_CVT, &Query) == -1)
    return -1;

  // We don't need conversion for UTF-8 tagged files.
  // TODO: Remove the assumption of ISO8859-1 = UTF-8 here when we fully resolve
  // problems related to UTF-8 tagged source files.
  // When the pccsid is not ISO8859-1, autoconversion is still needed.
  if (Query.pccsid == CCSID_ISO8859_1 &&
      (Query.fccsid == CCSID_UTF_8 || Query.fccsid == CCSID_ISO8859_1))
    return 0;

  // Save the state of std handles before we make changes to it.
  if ((FD == STDIN_FILENO || FD == STDOUT_FILENO || FD == STDERR_FILENO) &&
      savedStdHandleAutoConversionMode[FD] == -1)
    savedStdHandleAutoConversionMode[FD] = Query.cvtcmd;

  if (FD == STDOUT_FILENO || FD == STDERR_FILENO)
    Query.cvtcmd = SETCVTON;
  else
    Query.cvtcmd = SETCVTALL;

  Query.pccsid =
      (FD == STDIN_FILENO || FD == STDOUT_FILENO || FD == STDERR_FILENO)
          ? 0
          : CCSID_UTF_8;
  // Assume untagged files to be IBM-1047 encoded.
```
- EN: This section centers on `enablezOSAutoConversion` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `enablezOSAutoConversion` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 81-120

```cpp
  Query.fccsid = (Query.fccsid == FT_UNTAGGED) ? CCSID_IBM_1047 : Query.fccsid;
  return fcntl(FD, F_CONTROL_CVT, &Query);
}

std::error_code llvm::setzOSFileTag(int FD, int CCSID, bool Text) {
  assert((!Text || (CCSID != FT_UNTAGGED && CCSID != FT_BINARY)) &&
         "FT_UNTAGGED and FT_BINARY are not allowed for text files");
  struct file_tag Tag;
  Tag.ft_ccsid = CCSID;
  Tag.ft_txtflag = Text;
  Tag.ft_deferred = 0;
  Tag.ft_rsvflags = 0;

  if (fcntl(FD, F_SETTAG, &Tag) == -1) {
    if (errno == ENOSYS)
      // Some file systems do not support filetags.
      // Ignore ENOSYS error to allow compilation.
      errno = 0;
    else
      return errnoAsErrorCode();
  }
  return std::error_code();
}

ErrorOr<__ccsid_t> llvm::getzOSFileTag(const Twine &FileName, const int FD) {
  // If we have a file descriptor, use it to find out file tagging. Otherwise we
  // need to use stat() with the file path.
  if (FD != -1) {
    struct f_cnvrt Query = {
        QUERYCVT, // cvtcmd
        0,        // pccsid
        0,        // fccsid
    };
    if (fcntl(FD, F_CONTROL_CVT, &Query) == -1)
      return std::error_code(errno, std::generic_category());
    return Query.fccsid;
  }
  struct stat Attr;
  if (stat(FileName.str().c_str(), &Attr) == -1)
    return std::error_code(errno, std::generic_category());
```
- EN: This section centers on `fcntl`, `setzOSFileTag`, `assert` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `fcntl`, `setzOSFileTag`, `assert` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-151

```cpp
  return Attr.st_tag.ft_ccsid;
}

ErrorOr<bool> llvm::needzOSConversion(const Twine &FileName, const int FD) {
  ErrorOr<__ccsid_t> Ccsid = getzOSFileTag(FileName, FD);
  if (std::error_code EC = Ccsid.getError())
    return EC;
  // We don't need conversion for UTF-8 tagged files or binary files.
  // TODO: Remove the assumption of ISO8859-1 = UTF-8 here when we fully resolve
  // problems related to UTF-8 tagged source files.
  switch (*Ccsid) {
  case CCSID_UTF_8:
  case CCSID_ISO8859_1:
  case FT_BINARY:
    return false;
  default:
    return true;
  }
}

std::error_code llvm::copyFileTagAttributes(const std::string &Source,
                                            const int DestinationFD) {
  struct stat SourceAttributes;
  if (stat(Source.c_str(), &SourceAttributes) == -1)
    return std::error_code(errno, std::generic_category());

  return setzOSFileTag(DestinationFD, SourceAttributes.st_tag.ft_ccsid,
                       SourceAttributes.st_tag.ft_txtflag);
}

#endif //__MVS__
```
- EN: This section centers on `needzOSConversion`, `copyFileTagAttributes`, `setzOSFileTag` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `needzOSConversion`, `copyFileTagAttributes`, `setzOSFileTag` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and checks conditions and handles edge cases.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并检查条件并处理边界情况。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `f_cnvrt`, `file_tag`, `stat`, `disablezOSAutoConversion`, `fcntl`, `restorezOSStdHandleAutoConversion`, `assert` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/AutoConvert.h`
- Standard library / 标准库: `fcntl.h`, `unistd.h`
- Other/system headers / 其他或系统头文件: `cassert`, `sys/stat.h`
- Related symbols / 相关符号: `f_cnvrt`, `file_tag`, `stat`, `disablezOSAutoConversion`, `fcntl`, `restorezOSStdHandleAutoConversion`, `assert`, `enablezOSAutoConversion`
