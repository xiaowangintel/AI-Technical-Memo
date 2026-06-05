# LFIAsmParser.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/MCParser/LFIAsmParser.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements LLVM MC assembly parsing, directive handling, expression parsing, and streamer-facing parser utilities.
  - **CN**: 实现 LLVM MC 汇编解析、伪指令处理、表达式解析以及面向 streamer 的解析辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===----------------------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-11
```cpp
///
/// \file
/// LFI assembly parser.
///
```
- **EN**: Documents the next declaration group or explains a subsystem-specific rule.
- **CN**: 为后续声明分组提供说明，或解释某个子系统的专用规则。

### Lines 12-16
```cpp
//===----------------------------------------------------------------------===//

#include "llvm/MC/MCLFIRewriter.h"
#include "llvm/MC/MCParser/MCAsmParserExtension.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCLFIRewriter.h`, `llvm/MC/MCParser/MCAsmParserExtension.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCLFIRewriter.h`, `llvm/MC/MCParser/MCAsmParserExtension.h`。

### Lines 17-24
```cpp
using namespace llvm;

namespace {
class LFIAsmParser : public MCAsmParserExtension {
  MCLFIRewriter *Rewriter;
  template <bool (LFIAsmParser::*HandlerMethod)(StringRef, SMLoc)>
  void addDirectiveHandler(StringRef Directive) {
    MCAsmParser::ExtensionDirectiveHandler Handler =
```
- **EN**: Introduces declarations for `llvm`, `LFIAsmParser`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `LFIAsmParser` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 25-29
```cpp
        std::make_pair(this, HandleDirective<LFIAsmParser, HandlerMethod>);

    getParser().addDirectiveHandler(Directive, Handler);
  }

```
- **EN**: Implements logic around `make_pair`, `getParser`; this block parses assembly syntax or operands.
- **CN**: 围绕 `make_pair`, `getParser` 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 30-37
```cpp
public:
  LFIAsmParser(MCLFIRewriter *Exp) : Rewriter(Exp) {}
  void Initialize(MCAsmParser &Parser) override {
    // Call the base implementation.
    MCAsmParserExtension::Initialize(Parser);
    addDirectiveHandler<&LFIAsmParser::parseRewriteDisable>(
        ".lfi_rewrite_disable");
    addDirectiveHandler<&LFIAsmParser::parseRewriteEnable>(
```
- **EN**: Implements logic around `LFIAsmParser`, `Initialize`, `parseRewriteDisable>`, `parseRewriteEnable>`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `LFIAsmParser`, `Initialize`, `parseRewriteDisable>`, `parseRewriteEnable>` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为。

### Lines 38-41
```cpp
        ".lfi_rewrite_enable");
  }

  /// ::= {.lfi_rewrite_disable}
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 42-47
```cpp
  bool parseRewriteDisable(StringRef Directive, SMLoc Loc) {
    getParser().checkForValidSection();
    if (getLexer().isNot(AsmToken::EndOfStatement))
      return TokError("unexpected token");
    Lex();

```
- **EN**: Implements logic around `parseRewriteDisable`, `getParser`, `TokError`, `Lex`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseRewriteDisable`, `getParser`, `TokError`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 48-52
```cpp
    Rewriter->disable();

    return false;
  }

```
- **EN**: Implements logic around `disable`; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `disable` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 53-59
```cpp
  /// ::= {.lfi_rewrite_enable}
  bool parseRewriteEnable(StringRef Directive, SMLoc Loc) {
    getParser().checkForValidSection();
    if (getLexer().isNot(AsmToken::EndOfStatement))
      return TokError("unexpected token");
    Lex();

```
- **EN**: Implements logic around `parseRewriteEnable`, `getParser`, `TokError`, `Lex`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseRewriteEnable`, `getParser`, `TokError`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 60-66
```cpp
    Rewriter->enable();

    return false;
  }
};
} // namespace

```
- **EN**: Implements logic around `enable`; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `enable` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 67-69
```cpp
MCAsmParserExtension *llvm::createLFIAsmParser(MCLFIRewriter *Exp) {
  return new LFIAsmParser(Exp);
}
```
- **EN**: Implements logic around `createLFIAsmParser`, `LFIAsmParser`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `createLFIAsmParser`, `LFIAsmParser` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

## Key Concepts / 关键概念

- **Assembly parsing / 汇编解析**:
  - **EN**: Tokenizes assembly text, interprets directives, and builds MC expressions or instructions
  - **CN**: 对汇编文本做词法与语法解析，并构建 MC 表达式或指令
- **Directive handling / 伪指令处理**:
  - **EN**: Interprets assembler directives and maps them to streamer operations
  - **CN**: 解释汇编伪指令并把它们映射为 streamer 操作
- **Object layout / 目标文件布局**:
  - **EN**: Organizes fragments into sections and computes final offsets
  - **CN**: 把片段组织到节区中并计算最终偏移

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/MC/MCLFIRewriter.h`, `llvm/MC/MCParser/MCAsmParserExtension.h`
- **LLVM subsystems / LLVM 子系统**: MC
