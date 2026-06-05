# MCInstPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/MCInstPrinter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Convert an MCInst to target assembly syntax.
  - **CN**: 实现 MCInst 打印支持，以汇编语法呈现已解码的指令。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- MCInstPrinter.cpp - Convert an MCInst to target assembly syntax ----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-21
```cpp

#include "llvm/MC/MCInstPrinter.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/Format.h"
#include "llvm/Support/raw_ostream.h"
#include <cinttypes>
```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCInstPrinter.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/StringRef.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCInstPrinter.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/StringRef.h`。

### Lines 22-32
```cpp
#include <cstdint>

using namespace llvm;

void llvm::dumpBytes(ArrayRef<uint8_t> Bytes, raw_ostream &OS) {
  static const char HexRep[] = "0123456789abcdef";
  ListSeparator LS(" ");
  for (char Byte : Bytes)
    OS << LS << HexRep[(Byte & 0xF0) >> 4] << HexRep[Byte & 0xF];
}

```
- **EN**: Pulls in the headers needed for this implementation, including `cstdint`.
- **CN**: 引入该实现所需的头文件，其中包括 `cstdint`。

### Lines 33-40
```cpp
MCInstPrinter::~MCInstPrinter() = default;

/// getOpcodeName - Return the name of the specified opcode enum (e.g.
/// "MOV32ri") or empty if we can't resolve it.
StringRef MCInstPrinter::getOpcodeName(unsigned Opcode) const {
  return MII.getName(Opcode);
}

```
- **EN**: Implements logic around `~MCInstPrinter`, `getOpcodeName`, `getName`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `~MCInstPrinter`, `getOpcodeName`, `getName` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 41-54
```cpp
void MCInstPrinter::printRegName(raw_ostream &OS, MCRegister Reg) {
  llvm_unreachable("Target should implement this");
}

void MCInstPrinter::printAnnotation(raw_ostream &OS, StringRef Annot) {
  if (!Annot.empty()) {
    if (CommentStream) {
      (*CommentStream) << Annot;
      // By definition (see MCInstPrinter.h), CommentStream must end with
      // a newline after each comment.
      if (Annot.back() != '\n')
        (*CommentStream) << '\n';
    } else
      OS << " " << MAI.getCommentString() << " " << Annot;
```
- **EN**: Implements logic around `printRegName`, `llvm_unreachable`, `printAnnotation`, `getCommentString`.
- **CN**: 围绕 `printRegName`, `llvm_unreachable`, `printAnnotation`, `getCommentString` 实现具体逻辑。

### Lines 55-68
```cpp
  }
}

static bool matchAliasCondition(const MCInst &MI, const MCSubtargetInfo *STI,
                                const MCInstrInfo &MII,
                                const MCRegisterInfo &MRI, unsigned &OpIdx,
                                const AliasMatchingData &M,
                                const AliasPatternCond &C,
                                bool &OrPredicateResult) {
  // Feature tests are special, they don't consume operands.
  if (C.Kind == AliasPatternCond::K_Feature)
    return STI->getFeatureBits().test(C.Value);
  if (C.Kind == AliasPatternCond::K_NegFeature)
    return !STI->getFeatureBits().test(C.Value);
```
- **EN**: Implements logic around `matchAliasCondition`, `getFeatureBits`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `matchAliasCondition`, `getFeatureBits` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 69-82
```cpp
  // For feature tests where just one feature is required in a list, set the
  // predicate result bit to whether the expression will return true, and only
  // return the real result at the end of list marker.
  if (C.Kind == AliasPatternCond::K_OrFeature) {
    OrPredicateResult |= STI->getFeatureBits().test(C.Value);
    return true;
  }
  if (C.Kind == AliasPatternCond::K_OrNegFeature) {
    OrPredicateResult |= !(STI->getFeatureBits().test(C.Value));
    return true;
  }
  if (C.Kind == AliasPatternCond::K_EndOrFeatures) {
    bool Res = OrPredicateResult;
    OrPredicateResult = false;
```
- **EN**: Implements logic around `getFeatureBits`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getFeatureBits` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 83-89
```cpp
    return Res;
  }

  // Get and consume an operand.
  const MCOperand &Opnd = MI.getOperand(OpIdx);
  ++OpIdx;

```
- **EN**: Implements logic around `getOperand`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `getOperand` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 90-103
```cpp
  // Check the specific condition for the operand.
  switch (C.Kind) {
  case AliasPatternCond::K_Imm:
    // Operand must be a specific immediate.
    return Opnd.isImm() && Opnd.getImm() == int32_t(C.Value);
  case AliasPatternCond::K_Reg:
    // Operand must be a specific register.
    return Opnd.isReg() && Opnd.getReg() == C.Value;
  case AliasPatternCond::K_TiedReg:
    // Operand must match the register of another operand.
    return Opnd.isReg() && Opnd.getReg() == MI.getOperand(C.Value).getReg();
  case AliasPatternCond::K_RegClassByHwMode: {
    // Operand must be RegisterByHwMode. Value is RegClassByHwMode index.
    unsigned HwModeId = STI->getHwMode(MCSubtargetInfo::HwMode_RegInfo);
```
- **EN**: Implements logic around `isImm`, `isReg`, `getHwMode`; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `isImm`, `isReg`, `getHwMode` 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 104-117
```cpp
    int16_t RCID = MII.getRegClassByHwModeTable(HwModeId)[C.Value];
    return Opnd.isReg() && MRI.getRegClass(RCID).contains(Opnd.getReg());
  }
  case AliasPatternCond::K_RegClass:
    // Operand must be a register in this class. Value is a register class id.
    return Opnd.isReg() && MRI.getRegClass(C.Value).contains(Opnd.getReg());
  case AliasPatternCond::K_Custom:
    // Operand must match some custom criteria.
    return M.ValidateMCOperand(Opnd, *STI, C.Value);
  case AliasPatternCond::K_Ignore:
    // Operand can be anything.
    return true;
  case AliasPatternCond::K_Feature:
  case AliasPatternCond::K_NegFeature:
```
- **EN**: Introduces declarations for `id`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `id` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 118-125
```cpp
  case AliasPatternCond::K_OrFeature:
  case AliasPatternCond::K_OrNegFeature:
  case AliasPatternCond::K_EndOrFeatures:
    llvm_unreachable("handled earlier");
  }
  llvm_unreachable("invalid kind");
}

```
- **EN**: Implements logic around `llvm_unreachable`.
- **CN**: 围绕 `llvm_unreachable` 实现具体逻辑。

### Lines 126-137
```cpp
const char *MCInstPrinter::matchAliasPatterns(const MCInst *MI,
                                              const MCSubtargetInfo *STI,
                                              const AliasMatchingData &M) {
  // Binary search by opcode. Return false if there are no aliases for this
  // opcode.
  auto It = lower_bound(M.OpToPatterns, MI->getOpcode(),
                        [](const PatternsForOpcode &L, unsigned Opcode) {
                          return L.Opcode < Opcode;
                        });
  if (It == M.OpToPatterns.end() || It->Opcode != MI->getOpcode())
    return nullptr;

```
- **EN**: Implements logic around `matchAliasPatterns`, `lower_bound`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `matchAliasPatterns`, `lower_bound` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 138-146
```cpp
  // Try all patterns for this opcode.
  uint32_t AsmStrOffset = ~0U;
  ArrayRef<AliasPattern> Patterns =
      M.Patterns.slice(It->PatternStart, It->NumPatterns);
  for (const AliasPattern &P : Patterns) {
    // Check operand count first.
    if (MI->getNumOperands() != P.NumOperands)
      return nullptr;

```
- **EN**: Implements logic around `slice`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `slice` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 147-160
```cpp
    // Test all conditions for this pattern.
    ArrayRef<AliasPatternCond> Conds =
        M.PatternConds.slice(P.AliasCondStart, P.NumConds);
    unsigned OpIdx = 0;
    bool OrPredicateResult = false;
    if (llvm::all_of(Conds, [&](const AliasPatternCond &C) {
          return matchAliasCondition(*MI, STI, MII, MRI, OpIdx, M, C,
                                     OrPredicateResult);
        })) {
      // If all conditions matched, use this asm string.
      AsmStrOffset = P.AsmStrOffset;
      break;
    }
  }
```
- **EN**: Implements logic around `slice`, `matchAliasCondition`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `slice`, `matchAliasCondition` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 161-174
```cpp

  // If no alias matched, don't print an alias.
  if (AsmStrOffset == ~0U)
    return nullptr;

  // Go to offset AsmStrOffset and use the null terminated string there. The
  // offset should point to the beginning of an alias string, so it should
  // either be zero or be preceded by a null byte.
  assert(AsmStrOffset < M.AsmStrings.size() &&
         (AsmStrOffset == 0 || M.AsmStrings[AsmStrOffset - 1] == '\0') &&
         "bad asm string offset");
  return M.AsmStrings.data() + AsmStrOffset;
}

```
- **EN**: Implements logic around `assert`, `data`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `assert`, `data` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 175-187
```cpp
// For asm-style hex (e.g. 0ffh) the first digit always has to be a number.
static bool needsLeadingZero(uint64_t Value)
{
  while (Value)
  {
    uint64_t digit = (Value >> 60) & 0xf;
    if (digit != 0)
      return (digit >= 0xa);
    Value <<= 4;
  }
  return false;
}

```
- **EN**: Implements logic around `needsLeadingZero`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `needsLeadingZero` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 188-201
```cpp
format_object<int64_t> MCInstPrinter::formatDec(int64_t Value) const {
  return format("%" PRId64, Value);
}

format_object<int64_t> MCInstPrinter::formatHex(int64_t Value) const {
  switch (PrintHexStyle) {
  case HexStyle::C:
    if (Value < 0) {
      if (Value == std::numeric_limits<int64_t>::min())
        return format<int64_t>("-0x8000000000000000", Value);
      return format("-0x%" PRIx64, -Value);
    }
    return format("0x%" PRIx64, Value);
  case HexStyle::Asm:
```
- **EN**: Implements logic around `formatDec`, `format`, `formatHex`, `format<int64_t>`; this block uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `formatDec`, `format`, `formatHex`, `format<int64_t>` 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 202-215
```cpp
    if (Value < 0) {
      if (Value == std::numeric_limits<int64_t>::min())
        return format<int64_t>("-8000000000000000h", Value);
      if (needsLeadingZero(-(uint64_t)(Value)))
        return format("-0%" PRIx64 "h", -Value);
      return format("-%" PRIx64 "h", -Value);
    }
    if (needsLeadingZero((uint64_t)(Value)))
      return format("0%" PRIx64 "h", Value);
    return format("%" PRIx64 "h", Value);
  }
  llvm_unreachable("unsupported print style");
}

```
- **EN**: Implements logic around `format<int64_t>`, `format`, `llvm_unreachable`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `format<int64_t>`, `format`, `llvm_unreachable` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 216-228
```cpp
format_object<uint64_t> MCInstPrinter::formatHex(uint64_t Value) const {
  switch(PrintHexStyle) {
  case HexStyle::C:
     return format("0x%" PRIx64, Value);
  case HexStyle::Asm:
    if (needsLeadingZero(Value))
      return format("0%" PRIx64 "h", Value);
    else
      return format("%" PRIx64 "h", Value);
  }
  llvm_unreachable("unsupported print style");
}

```
- **EN**: Implements logic around `formatHex`, `format`, `llvm_unreachable`; this block uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `formatHex`, `format`, `llvm_unreachable` 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 229-242
```cpp
MCInstPrinter::WithMarkup MCInstPrinter::markup(raw_ostream &OS, Markup S) {
  return WithMarkup(*this, OS, S, getUseMarkup(), getUseColor());
}

MCInstPrinter::WithMarkup::WithMarkup(MCInstPrinter &IP, raw_ostream &OS,
                                      Markup M, bool EnableMarkup,
                                      bool EnableColor)
    : IP(IP), OS(OS), EnableMarkup(EnableMarkup), EnableColor(EnableColor) {
  if (EnableColor) {
    raw_ostream::Colors Color = raw_ostream::Colors::RESET;
    switch (M) {
    case Markup::Immediate:
      Color = raw_ostream::RED;
      break;
```
- **EN**: Implements logic around `markup`, `WithMarkup`, `IP`; this block uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `markup`, `WithMarkup`, `IP` 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 243-256
```cpp
    case Markup::Register:
      Color = raw_ostream::CYAN;
      break;
    case Markup::Target:
      Color = raw_ostream::YELLOW;
      break;
    case Markup::Memory:
      Color = raw_ostream::GREEN;
      break;
    }
    IP.ColorStack.push_back(Color);
    OS.changeColor(Color);
  }

```
- **EN**: Implements logic around `push_back`, `changeColor`.
- **CN**: 围绕 `push_back`, `changeColor` 实现具体逻辑。

### Lines 257-270
```cpp
  if (EnableMarkup) {
    switch (M) {
    case Markup::Immediate:
      OS << "<imm:";
      break;
    case Markup::Register:
      OS << "<reg:";
      break;
    case Markup::Target:
      OS << "<target:";
      break;
    case Markup::Memory:
      OS << "<mem:";
      break;
```
- **EN**: Implements subsystem-specific case analysis using a `switch` over kinds, opcodes, or states.
- **CN**: 通过对 kind、opcode 或状态执行 `switch` 分析来实现子系统相关逻辑。

### Lines 271-282
```cpp
    }
  }
}

MCInstPrinter::WithMarkup::~WithMarkup() {
  if (EnableMarkup)
    OS << '>';
  if (!EnableColor)
    return;
  IP.ColorStack.pop_back();
  OS << IP.ColorStack.back();
}
```
- **EN**: Implements logic around `~WithMarkup`, `pop_back`, `back`.
- **CN**: 围绕 `~WithMarkup`, `pop_back`, `back` 实现具体逻辑。

## Key Concepts / 关键概念

- **Assembly rendering / 汇编渲染**:
  - **EN**: Prints MC instructions and operands back into textual assembly
  - **CN**: 把 MC 指令与操作数重新打印为文本汇编
- **MC instruction model / MC 指令模型**:
  - **EN**: Uses MCInst/MCOperand abstractions to represent machine instructions generically
  - **CN**: 使用 MCInst/MCOperand 抽象以通用方式表示机器指令

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/MC/MCInstPrinter.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/StringRef.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCRegisterInfo.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/Format.h`, `llvm/Support/raw_ostream.h` ... (+2 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support
