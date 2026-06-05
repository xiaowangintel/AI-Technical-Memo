# CodeGenCoverage.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/CodeGenCoverage.cpp`
- Repository: `llvm-project`
- Purpose (EN): / \file / This file implements the CodeGenCoverage class.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `CodeGenCoverage` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
//===- lib/Support/CodeGenCoverage.cpp -------------------------------------==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
/// \file
/// This file implements the CodeGenCoverage class.
//===----------------------------------------------------------------------===//

#include "llvm/Support/CodeGenCoverage.h"

#include "llvm/Support/Endian.h"
#include "llvm/Support/FileSystem.h"
#include "llvm/Support/MemoryBuffer.h"
#include "llvm/Support/Mutex.h"
#include "llvm/Support/Process.h"
#include "llvm/Support/ScopedPrinter.h"
#include "llvm/Support/ToolOutputFile.h"
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 8 direct dependencies, including `llvm/Support/CodeGenCoverage.h`, `llvm/Support/Endian.h`, `llvm/Support/FileSystem.h`, `llvm/Support/MemoryBuffer.h`.
  CN: 引入了 8 个直接依赖，其中包括 `llvm/Support/CodeGenCoverage.h`, `llvm/Support/Endian.h`, `llvm/Support/FileSystem.h`, `llvm/Support/MemoryBuffer.h`。

### Lines 21-40

```cpp

using namespace llvm;

CodeGenCoverage::CodeGenCoverage() = default;

void CodeGenCoverage::setCovered(uint64_t RuleID) {
  if (RuleCoverage.size() <= RuleID)
    RuleCoverage.resize(RuleID + 1, false);
  RuleCoverage[RuleID] = true;
}

bool CodeGenCoverage::isCovered(uint64_t RuleID) const {
  if (RuleCoverage.size() <= RuleID)
    return false;
  return RuleCoverage[RuleID];
}

iterator_range<CodeGenCoverage::const_covered_iterator>
CodeGenCoverage::covered() const {
  return RuleCoverage.set_bits();
```
- EN: This section centers on `setCovered`, `isCovered`, `covered` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `setCovered`, `isCovered`, `covered` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 41-60

```cpp
}

bool CodeGenCoverage::parse(MemoryBuffer &Buffer, StringRef BackendName) {
  const char *CurPtr = Buffer.getBufferStart();

  while (CurPtr != Buffer.getBufferEnd()) {
    // Read the backend name from the input.
    const char *LexedBackendName = CurPtr;
    while (*CurPtr++ != 0)
      ;
    if (CurPtr == Buffer.getBufferEnd())
      return false; // Data is invalid, expected rule id's to follow.

    bool IsForThisBackend = BackendName == LexedBackendName;
    while (CurPtr != Buffer.getBufferEnd()) {
      if (std::distance(CurPtr, Buffer.getBufferEnd()) < 8)
        return false; // Data is invalid. Not enough bytes for another rule id.

      uint64_t RuleID =
          support::endian::read64(CurPtr, llvm::endianness::native);
```
- EN: This section centers on `parse`, `read64` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `parse`, `read64` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 61-80

```cpp
      CurPtr += 8;

      // ~0ull terminates the rule id list.
      if (RuleID == ~0ull)
        break;

      // Anything else, is recorded or ignored depending on whether it's
      // intended for the backend we're interested in.
      if (IsForThisBackend)
        setCovered(RuleID);
    }
  }

  return true;
}

bool CodeGenCoverage::emit(StringRef CoveragePrefix,
                           StringRef BackendName) const {
  if (!CoveragePrefix.empty() && !RuleCoverage.empty()) {
    static sys::SmartMutex<true> OutputMutex;
```
- EN: This section centers on `emit` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `emit` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 81-100

```cpp
    sys::SmartScopedLock<true> Lock(OutputMutex);

    // We can handle locking within a process easily enough but we don't want to
    // manage it between multiple processes. Use the process ID to ensure no
    // more than one process is ever writing to the same file at the same time.
    std::string Pid = llvm::to_string(sys::Process::getProcessId());

    std::string CoverageFilename = (CoveragePrefix + Pid).str();

    std::error_code EC;
    sys::fs::OpenFlags OpenFlags = sys::fs::OF_Append;
    std::unique_ptr<ToolOutputFile> CoverageFile =
        std::make_unique<ToolOutputFile>(CoverageFilename, EC, OpenFlags);
    if (EC)
      return false;

    uint64_t Zero = 0;
    uint64_t InvZero = ~0ull;
    CoverageFile->os() << BackendName;
    CoverageFile->os().write((const char *)&Zero, sizeof(unsigned char));
```
- EN: This section centers on `Lock` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `Lock` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 101-111

```cpp
    for (uint64_t I : RuleCoverage.set_bits())
      CoverageFile->os().write((const char *)&I, sizeof(uint64_t));
    CoverageFile->os().write((const char *)&InvZero, sizeof(uint64_t));

    CoverageFile->keep();
  }

  return true;
}

void CodeGenCoverage::reset() { RuleCoverage.resize(0); }
```
- EN: This section centers on `reset` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `reset` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and returns the resulting value to its callers.
  CN: 在这一段中，代码遍历集合、区间或记录，并将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `setCovered`, `isCovered`, `covered`, `parse` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Parsing, decoding, and structural validation. / 重点关注解析、解码与结构校验。
- Integration / 集成关系: Connects file-local logic to adjacent LLVM headers, support types, and subsystems. / 将文件内逻辑连接到相邻的 LLVM 头文件、支撑类型与子系统。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/CodeGenCoverage.h`, `llvm/Support/Endian.h`, `llvm/Support/FileSystem.h`, `llvm/Support/MemoryBuffer.h`, `llvm/Support/Mutex.h`, `llvm/Support/Process.h`, `llvm/Support/ScopedPrinter.h`, `llvm/Support/ToolOutputFile.h`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `setCovered`, `isCovered`, `covered`, `parse`, `read64`
