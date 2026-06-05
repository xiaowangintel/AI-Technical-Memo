# RISCVISAUtils.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/RISCVISAUtils.cpp`
- Repository: `llvm-project`
- Purpose (EN): Utilities shared by TableGen and RISCVISAInfo.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `RISCVISAUtils` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
//===-- RISCVISAUtils.cpp - RISC-V ISA Utilities --------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Utilities shared by TableGen and RISCVISAInfo.
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/RISCVISAUtils.h"
#include "llvm/ADT/StringExtras.h"
#include <cassert>

using namespace llvm;

// We rank extensions in the following order:
// -Single letter extensions in canonical order.
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 3 direct dependencies, including `llvm/Support/RISCVISAUtils.h`, `llvm/ADT/StringExtras.h`, `cassert`.
  CN: 引入了 3 个直接依赖，其中包括 `llvm/Support/RISCVISAUtils.h`, `llvm/ADT/StringExtras.h`, `cassert`。
- EN: This range propagates LLVM-style errors and invariants.
  CN: 这一段传播 LLVM 风格的错误处理并维护不变量。

### Lines 21-40

```cpp
// -Unknown single letter extensions in alphabetical order.
// -Multi-letter extensions starting with 'z' sorted by canonical order of
//  the second letter then sorted alphabetically.
// -Multi-letter extensions starting with 's' in alphabetical order.
// -(TODO) Multi-letter extensions starting with 'zxm' in alphabetical order.
// -X extensions in alphabetical order.
// -Unknown multi-letter extensions in alphabetical order.
// These flags are used to indicate the category. The first 6 bits store the
// single letter extension rank for single letter and multi-letter extensions
// starting with 'z'.
enum RankFlags {
  RF_Z_EXTENSION = 1 << 6,
  RF_S_EXTENSION = 2 << 6,
  RF_X_EXTENSION = 3 << 6,
  RF_UNKNOWN_MULTILETTER_EXTENSION = 4 << 6,
};

// Get the rank for single-letter extension, lower value meaning higher
// priority.
static unsigned singleLetterExtensionRank(char Ext) {
```
- EN: This section centers on `singleLetterExtensionRank` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `singleLetterExtensionRank` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 41-60

```cpp
  assert(isLower(Ext));
  switch (Ext) {
  case 'i':
    return 0;
  case 'e':
    return 1;
  }

  size_t Pos = RISCVISAUtils::AllStdExts.find(Ext);
  if (Pos != StringRef::npos)
    return Pos + 2; // Skip 'e' and 'i' from above.

  // If we got an unknown extension letter, then give it an alphabetical
  // order, but after all known standard extensions.
  return 2 + RISCVISAUtils::AllStdExts.size() + (Ext - 'a');
}

// Get the rank for multi-letter extension, lower value meaning higher
// priority/order in canonical order.
static unsigned getExtensionRank(const std::string &ExtName) {
```
- EN: This section centers on `assert`, `getExtensionRank` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `assert`, `getExtensionRank` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and checks conditions and handles edge cases.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并检查条件并处理边界情况。

### Lines 61-80

```cpp
  assert(ExtName.size() >= 1);
  switch (ExtName[0]) {
  case 's':
    return RF_S_EXTENSION;
  case 'z':
    assert(ExtName.size() >= 2);
    // `z` extension must be sorted by canonical order of second letter.
    // e.g. zmx has higher rank than zax.
    return RF_Z_EXTENSION | singleLetterExtensionRank(ExtName[1]);
  case 'x':
    return RF_X_EXTENSION;
  default:
    if (ExtName.size() == 1)
      return singleLetterExtensionRank(ExtName[0]);
    return RF_UNKNOWN_MULTILETTER_EXTENSION;
  }
}

// Compare function for extension.
// Only compare the extension name, ignore version comparison.
```
- EN: This section centers on `assert` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `assert` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and checks conditions and handles edge cases.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并检查条件并处理边界情况。

### Lines 81-92

```cpp
bool llvm::RISCVISAUtils::compareExtension(const std::string &LHS,
                                           const std::string &RHS) {
  unsigned LHSRank = getExtensionRank(LHS);
  unsigned RHSRank = getExtensionRank(RHS);

  // If the ranks differ, pick the lower rank.
  if (LHSRank != RHSRank)
    return LHSRank < RHSRank;

  // If the rank is same, it must be sorted by lexicographic order.
  return LHS < RHS;
}
```
- EN: This section centers on `compareExtension` and performs utility computation and state updates.
  CN: 这一段主要围绕 `compareExtension` 等符号展开，负责执行工具性计算并更新状态。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `RankFlags`, `singleLetterExtensionRank`, `assert`, `getExtensionRank`, `compareExtension` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/RISCVISAUtils.h`, `llvm/ADT/StringExtras.h`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: `cassert`
- Related symbols / 相关符号: `RankFlags`, `singleLetterExtensionRank`, `assert`, `getExtensionRank`, `compareExtension`
