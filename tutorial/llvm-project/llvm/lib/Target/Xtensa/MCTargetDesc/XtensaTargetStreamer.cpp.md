# XtensaTargetStreamer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/MCTargetDesc/XtensaTargetStreamer.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Provides MC-layer target descriptions such as fixups, asm info, code emission, object writing, and target registration.
  - **CN**: 提供 MC 层的目标描述，例如 fixup、汇编信息、编码发射、目标文件写出以及目标注册。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- XtensaTargetStreamer.cpp - Xtensa Target Streamer Methods ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-12
```cpp
//
// This file provides Xtensa specific target streamer methods.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 13-22
```cpp
#include "XtensaTargetStreamer.h"
#include "XtensaInstPrinter.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/MC/MCAssembler.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCObjectFileInfo.h"
#include "llvm/MC/MCSectionELF.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/FormattedStream.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `XtensaTargetStreamer.h`, `XtensaInstPrinter.h`, `llvm/BinaryFormat/ELF.h`, `llvm/MC/MCAssembler.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `XtensaTargetStreamer.h`, `XtensaInstPrinter.h`, `llvm/BinaryFormat/ELF.h`, `llvm/MC/MCAssembler.h`。

### Lines 23-30
```cpp
using namespace llvm;

static std::string getLiteralSectionName(StringRef CSectionName) {
  std::size_t Pos = CSectionName.find(".text");
  std::string SectionName;
  if (Pos != std::string::npos) {
    SectionName = CSectionName.substr(0, Pos);

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 31-36
```cpp
    if (Pos > 0)
      SectionName += ".text";

    CSectionName = CSectionName.drop_front(Pos);
    CSectionName.consume_front(".text");

```
- **EN**: Implements logic around `drop_front`, `consume_front`; this block applies conditional target rules.
- **CN**: 围绕 `drop_front`, `consume_front` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 37-45
```cpp
    SectionName += ".literal";
    SectionName += CSectionName;
  } else {
    SectionName = CSectionName;
    SectionName += ".literal";
  }
  return SectionName;
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 46-52
```cpp
XtensaTargetStreamer::XtensaTargetStreamer(MCStreamer &S)
    : MCTargetStreamer(S) {}

XtensaTargetAsmStreamer::XtensaTargetAsmStreamer(MCStreamer &S,
                                                 formatted_raw_ostream &OS)
    : XtensaTargetStreamer(S), OS(OS) {}

```
- **EN**: Implements logic around `XtensaTargetStreamer`, `MCTargetStreamer`, `XtensaTargetAsmStreamer`.
- **CN**: 围绕 `XtensaTargetStreamer`, `MCTargetStreamer`, `XtensaTargetAsmStreamer` 实现具体逻辑。

### Lines 53-57
```cpp
void XtensaTargetAsmStreamer::emitLiteral(MCSymbol *LblSym, const MCExpr *Value,
                                          bool SwitchLiteralSection, SMLoc L) {
  SmallString<60> Str;
  raw_svector_ostream LiteralStr(Str);

```
- **EN**: Implements logic around `emitLiteral`, `LiteralStr`; this block uses `switch`-based dispatch.
- **CN**: 围绕 `emitLiteral`, `LiteralStr` 实现具体逻辑；这一段使用 `switch` 分派。

### Lines 58-67
```cpp
  LiteralStr << "\t.literal " << LblSym->getName() << ", ";

  if (auto CE = dyn_cast<MCConstantExpr>(Value)) {
    LiteralStr << CE->getValue() << "\n";
  } else if (auto SRE = dyn_cast<MCSymbolRefExpr>(Value)) {
    const MCSymbol &Sym = SRE->getSymbol();
    LiteralStr << Sym.getName() << "\n";
  } else {
    llvm_unreachable("unexpected constant pool entry type");
  }
```
- **EN**: Implements logic around `getName`, `getValue`, `getSymbol`, `llvm_unreachable`; this block applies conditional target rules.
- **CN**: 围绕 `getName`, `getValue`, `getSymbol`, `llvm_unreachable` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 68-75
```cpp

  OS << LiteralStr.str();
}

void XtensaTargetAsmStreamer::emitLiteralPosition() {
  OS << "\t.literal_position\n";
}

```
- **EN**: Implements logic around `str`, `emitLiteralPosition`.
- **CN**: 围绕 `str`, `emitLiteralPosition` 实现具体逻辑。

### Lines 76-82
```cpp
void XtensaTargetAsmStreamer::startLiteralSection(MCSection *BaseSection) {
  emitLiteralPosition();
}

XtensaTargetELFStreamer::XtensaTargetELFStreamer(MCStreamer &S)
    : XtensaTargetStreamer(S) {}

```
- **EN**: Implements logic around `startLiteralSection`, `emitLiteralPosition`, `XtensaTargetELFStreamer`, `XtensaTargetStreamer`.
- **CN**: 围绕 `startLiteralSection`, `emitLiteralPosition`, `XtensaTargetELFStreamer`, `XtensaTargetStreamer` 实现具体逻辑。

### Lines 83-90
```cpp
void XtensaTargetELFStreamer::emitLiteral(MCSymbol *LblSym, const MCExpr *Value,
                                          bool SwitchLiteralSection, SMLoc L) {
  MCStreamer &OutStreamer = getStreamer();
  if (SwitchLiteralSection) {
    MCContext &Context = OutStreamer.getContext();
    auto *CS = static_cast<MCSectionELF *>(OutStreamer.getCurrentSectionOnly());
    std::string SectionName = getLiteralSectionName(CS->getName());

```
- **EN**: Implements logic around `emitLiteral`, `getStreamer`, `getContext`, `getCurrentSectionOnly`, ...; this block uses `switch`-based dispatch; applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `emitLiteral`, `getStreamer`, `getContext`, `getCurrentSectionOnly`, ... 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，工作在 MC 层。

### Lines 91-97
```cpp
    MCSection *ConstSection = Context.getELFSection(
        SectionName, ELF::SHT_PROGBITS, ELF::SHF_EXECINSTR | ELF::SHF_ALLOC);

    OutStreamer.pushSection();
    OutStreamer.switchSection(ConstSection);
  }

```
- **EN**: Implements logic around `getELFSection`, `pushSection`, `switchSection`; this block uses `switch`-based dispatch.
- **CN**: 围绕 `getELFSection`, `pushSection`, `switchSection` 实现具体逻辑；这一段使用 `switch` 分派。

### Lines 98-105
```cpp
  OutStreamer.emitLabel(LblSym, L);
  OutStreamer.emitValue(Value, 4, L);

  if (SwitchLiteralSection) {
    OutStreamer.popSection();
  }
}

```
- **EN**: Implements logic around `emitLabel`, `emitValue`, `popSection`; this block uses `switch`-based dispatch; applies conditional target rules.
- **CN**: 围绕 `emitLabel`, `emitValue`, `popSection` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则。

### Lines 106-110
```cpp
void XtensaTargetELFStreamer::startLiteralSection(MCSection *BaseSection) {
  MCContext &Context = getStreamer().getContext();

  std::string SectionName = getLiteralSectionName(BaseSection->getName());

```
- **EN**: Implements logic around `startLiteralSection`, `getStreamer`, `getLiteralSectionName`; this block works at the MC layer.
- **CN**: 围绕 `startLiteralSection`, `getStreamer`, `getLiteralSectionName` 实现具体逻辑；这一段工作在 MC 层。

### Lines 111-116
```cpp
  MCSection *ConstSection = Context.getELFSection(
      SectionName, ELF::SHT_PROGBITS, ELF::SHF_EXECINSTR | ELF::SHF_ALLOC);

  ConstSection->setAlignment(Align(4));
}

```
- **EN**: Implements logic around `getELFSection`, `setAlignment`.
- **CN**: 围绕 `getELFSection`, `setAlignment` 实现具体逻辑。

### Lines 117-119
```cpp
MCELFStreamer &XtensaTargetELFStreamer::getStreamer() {
  return static_cast<MCELFStreamer &>(Streamer);
}
```
- **EN**: Implements logic around `getStreamer`; this block returns target-specific results.
- **CN**: 围绕 `getStreamer` 实现具体逻辑；这一段返回目标相关结果。

## Key Concepts / 关键概念

- **MC layer / MC 层**:
  - **EN**: Connects the backend to LLVM's MC infrastructure
  - **CN**: 把后端接入 LLVM 的 MC 基础设施

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `XtensaTargetStreamer.h`, `XtensaInstPrinter.h`, `llvm/BinaryFormat/ELF.h`, `llvm/MC/MCAssembler.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCObjectFileInfo.h`, `llvm/MC/MCSectionELF.h`, `llvm/Support/Casting.h`, `llvm/Support/FormattedStream.h`
- **LLVM subsystems / LLVM 子系统**: MC, Support
