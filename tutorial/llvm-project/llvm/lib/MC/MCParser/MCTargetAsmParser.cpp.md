# MCTargetAsmParser.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/MCParser/MCTargetAsmParser.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Target Assembly Parser.
  - **CN**: 实现 LLVM MC 汇编解析、伪指令处理、表达式解析以及面向 streamer 的解析辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- MCTargetAsmParser.cpp - Target Assembly Parser --------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-13
```cpp

#include "llvm/MC/MCParser/MCTargetAsmParser.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCObjectStreamer.h"
#include "llvm/MC/MCRegister.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCParser/MCTargetAsmParser.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCObjectStreamer.h`, `llvm/MC/MCRegister.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCParser/MCTargetAsmParser.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCObjectStreamer.h`, `llvm/MC/MCRegister.h`。

### Lines 14-19
```cpp
using namespace llvm;

MCTargetAsmParser::MCTargetAsmParser(const MCSubtargetInfo &STI,
                                     const MCInstrInfo &MII)
    : STI(&STI), MII(MII) {}

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 20-27
```cpp
MCTargetAsmParser::~MCTargetAsmParser() = default;

MCSubtargetInfo &MCTargetAsmParser::copySTI() {
  MCSubtargetInfo &STICopy = getContext().getSubtargetCopy(getSTI());
  STI = &STICopy;
  // The returned STI will likely be modified. Create a new fragment to prevent
  // mixing STI values within a fragment.
  auto &S = getStreamer();
```
- **EN**: Implements logic around `~MCTargetAsmParser`, `copySTI`, `getContext`, `getStreamer`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `~MCTargetAsmParser`, `copySTI`, `getContext`, `getStreamer` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为。

### Lines 28-32
```cpp
  if (S.isObj() && S.getCurrentFragment())
    static_cast<MCObjectStreamer &>(S).newFragment();
  return STICopy;
}

```
- **EN**: Implements logic around `newFragment`; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `newFragment` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 33-36
```cpp
const MCSubtargetInfo &MCTargetAsmParser::getSTI() const {
  return *STI;
}

```
- **EN**: Implements logic around `getSTI`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `getSTI` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 37-42
```cpp
ParseStatus MCTargetAsmParser::parseDirective(AsmToken DirectiveID) {
  SMLoc StartTokLoc = getTok().getLoc();
  // Delegate to ParseDirective by default for transition period. Once the
  // transition is over, this method should just return NoMatch.
  bool Res = ParseDirective(DirectiveID);

```
- **EN**: Implements logic around `parseDirective`, `getTok`, `ParseDirective`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirective`, `getTok`, `ParseDirective` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 43-46
```cpp
  // Some targets erroneously report success after emitting an error.
  if (getParser().hasPendingError())
    return ParseStatus::Failure;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 47-54
```cpp
  // ParseDirective returns true if there was an error or if the directive is
  // not target-specific. Disambiguate the two cases by comparing position of
  // the lexer before and after calling the method: if no tokens were consumed,
  // there was no match, otherwise there was a failure.
  if (!Res)
    return ParseStatus::Success;
  if (getTok().getLoc() != StartTokLoc)
    return ParseStatus::Failure;
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 55-61
```cpp
  return ParseStatus::NoMatch;
}

bool MCTargetAsmParser::areEqualRegs(const MCParsedAsmOperand &Op1,
                                     const MCParsedAsmOperand &Op2) const {
  return Op1.isReg() && Op2.isReg() && Op1.getReg() == Op2.getReg();
}
```
- **EN**: Implements logic around `areEqualRegs`, `isReg`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `areEqualRegs`, `isReg` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

## Key Concepts / 关键概念

- **Assembly parsing / 汇编解析**:
  - **EN**: Tokenizes assembly text, interprets directives, and builds MC expressions or instructions
  - **CN**: 对汇编文本做词法与语法解析，并构建 MC 表达式或指令
- **Directive handling / 伪指令处理**:
  - **EN**: Interprets assembler directives and maps them to streamer operations
  - **CN**: 解释汇编伪指令并把它们映射为 streamer 操作
- **MC instruction model / MC 指令模型**:
  - **EN**: Uses MCInst/MCOperand abstractions to represent machine instructions generically
  - **CN**: 使用 MCInst/MCOperand 抽象以通用方式表示机器指令
- **Object layout / 目标文件布局**:
  - **EN**: Organizes fragments into sections and computes final offsets
  - **CN**: 把片段组织到节区中并计算最终偏移

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/MC/MCParser/MCTargetAsmParser.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCObjectStreamer.h`, `llvm/MC/MCRegister.h`
- **LLVM subsystems / LLVM 子系统**: MC
