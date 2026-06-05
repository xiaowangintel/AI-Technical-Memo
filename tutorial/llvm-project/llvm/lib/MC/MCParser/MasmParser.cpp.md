# MasmParser.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/MCParser/MasmParser.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements LLVM MC assembly parsing, directive handling, expression parsing, and streamer-facing parser utilities.
  - **CN**: 实现 LLVM MC 汇编解析、伪指令处理、表达式解析以及面向 streamer 的解析辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40
```cpp
//===- AsmParser.cpp - Parser for Assembly Files --------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This class implements the parser for assembly files.
//
//===----------------------------------------------------------------------===//

#include "llvm/ADT/APFloat.h"
#include "llvm/ADT/APInt.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/BitVector.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/ADT/StringMap.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/StringSwitch.h"
#include "llvm/ADT/Twine.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCCodeView.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCDirectives.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInstPrinter.h"
#include "llvm/MC/MCInstrDesc.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCParser/AsmCond.h"
#include "llvm/MC/MCParser/AsmLexer.h"
#include "llvm/MC/MCParser/MCAsmParser.h"
#include "llvm/MC/MCParser/MCAsmParserExtension.h"
#include "llvm/MC/MCParser/MCParsedAsmOperand.h"
#include "llvm/MC/MCParser/MCTargetAsmParser.h"
#include "llvm/MC/MCSection.h"
#include "llvm/MC/MCStreamer.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ADT/APFloat.h`, `llvm/ADT/APInt.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/BitVector.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ADT/APFloat.h`, `llvm/ADT/APInt.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/BitVector.h`。

### Lines 41-69
```cpp
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCSymbolCOFF.h"
#include "llvm/MC/MCTargetOptions.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/Format.h"
#include "llvm/Support/MD5.h"
#include "llvm/Support/MathExtras.h"
#include "llvm/Support/MemoryBuffer.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/SMLoc.h"
#include "llvm/Support/SourceMgr.h"
#include "llvm/Support/raw_ostream.h"
#include <algorithm>
#include <cassert>
#include <climits>
#include <cstddef>
#include <cstdint>
#include <ctime>
#include <deque>
#include <memory>
#include <optional>
#include <sstream>
#include <string>
#include <tuple>
#include <utility>
#include <vector>

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/MCSymbolCOFF.h`, `llvm/MC/MCTargetOptions.h`, `llvm/Support/Casting.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/MCSymbolCOFF.h`, `llvm/MC/MCTargetOptions.h`, `llvm/Support/Casting.h`。

### Lines 70-89
```cpp
using namespace llvm;

namespace {

/// Helper types for tracking macro definitions.
typedef std::vector<AsmToken> MCAsmMacroArgument;
typedef std::vector<MCAsmMacroArgument> MCAsmMacroArguments;

/// Helper class for storing information about an active macro instantiation.
struct MacroInstantiation {
  /// The location of the instantiation.
  SMLoc InstantiationLoc;

  /// The buffer where parsing should resume upon instantiation completion.
  unsigned ExitBuffer;

  /// The location where parsing should resume upon instantiation completion.
  SMLoc ExitLoc;

  /// The depth of TheCondStack at the start of the instantiation.
```
- **EN**: Introduces declarations for `llvm`, `for`, `MacroInstantiation`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `for`, `MacroInstantiation` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 90-112
```cpp
  size_t CondStackDepth;
};

struct ParseStatementInfo {
  /// The parsed operands from the last parsed statement.
  SmallVector<std::unique_ptr<MCParsedAsmOperand>, 8> ParsedOperands;

  /// The opcode from the last parsed instruction.
  unsigned Opcode = ~0U;

  /// Was there an error parsing the inline assembly?
  bool ParseError = false;

  /// The value associated with a macro exit.
  std::optional<std::string> ExitValue;

  SmallVectorImpl<AsmRewrite> *AsmRewrites = nullptr;

  ParseStatementInfo() = delete;
  ParseStatementInfo(SmallVectorImpl<AsmRewrite> *rewrites)
      : AsmRewrites(rewrites) {}
};

```
- **EN**: Introduces declarations for `ParseStatementInfo`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `ParseStatementInfo` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 113-133
```cpp
enum FieldType {
  FT_INTEGRAL, // Initializer: integer expression, stored as an MCExpr.
  FT_REAL,     // Initializer: real number, stored as an APInt.
  FT_STRUCT    // Initializer: struct initializer, stored recursively.
};

struct FieldInfo;
struct StructInfo {
  StringRef Name;
  bool IsUnion = false;
  bool Initializable = true;
  unsigned Alignment = 0;
  unsigned AlignmentSize = 0;
  unsigned NextOffset = 0;
  unsigned Size = 0;
  std::vector<FieldInfo> Fields;
  StringMap<size_t> FieldsByName;

  FieldInfo &addField(StringRef FieldName, FieldType FT,
                      unsigned FieldAlignmentSize);

```
- **EN**: Introduces declarations for `FieldType`, `initializer`, `FieldInfo`, `StructInfo`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `FieldType`, `initializer`, `FieldInfo`, `StructInfo` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 134-153
```cpp
  StructInfo() = default;
  StructInfo(StringRef StructName, bool Union, unsigned AlignmentValue);
};

// FIXME: This should probably use a class hierarchy, raw pointers between the
// objects, and dynamic type resolution instead of a union. On the other hand,
// ownership then becomes much more complicated; the obvious thing would be to
// use BumpPtrAllocator, but the lack of a destructor makes that messy.

struct StructInitializer;
struct IntFieldInfo {
  SmallVector<const MCExpr *, 1> Values;

  IntFieldInfo() = default;
  IntFieldInfo(const SmallVector<const MCExpr *, 1> &V) { Values = V; }
  IntFieldInfo(SmallVector<const MCExpr *, 1> &&V) { Values = std::move(V); }
};
struct RealFieldInfo {
  SmallVector<APInt, 1> AsIntValues;

```
- **EN**: Introduces declarations for `hierarchy`, `StructInitializer`, `IntFieldInfo`, `RealFieldInfo`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `hierarchy`, `StructInitializer`, `IntFieldInfo`, `RealFieldInfo` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 154-174
```cpp
  RealFieldInfo() = default;
  RealFieldInfo(const SmallVector<APInt, 1> &V) { AsIntValues = V; }
  RealFieldInfo(SmallVector<APInt, 1> &&V) { AsIntValues = std::move(V); }
};
struct StructFieldInfo {
  std::vector<StructInitializer> Initializers;
  StructInfo Structure;

  StructFieldInfo() = default;
  StructFieldInfo(std::vector<StructInitializer> V, StructInfo S);
};

class FieldInitializer {
public:
  FieldType FT;
  union {
    IntFieldInfo IntInfo;
    RealFieldInfo RealInfo;
    StructFieldInfo StructInfo;
  };

```
- **EN**: Introduces declarations for `StructFieldInfo`, `FieldInitializer`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `StructFieldInfo`, `FieldInitializer` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 175-197
```cpp
  ~FieldInitializer();
  FieldInitializer(FieldType FT);

  FieldInitializer(SmallVector<const MCExpr *, 1> &&Values);
  FieldInitializer(SmallVector<APInt, 1> &&AsIntValues);
  FieldInitializer(std::vector<StructInitializer> &&Initializers,
                   struct StructInfo Structure);

  FieldInitializer(const FieldInitializer &Initializer);
  FieldInitializer(FieldInitializer &&Initializer);

  FieldInitializer &operator=(const FieldInitializer &Initializer);
  FieldInitializer &operator=(FieldInitializer &&Initializer);
};

struct StructInitializer {
  std::vector<FieldInitializer> FieldInitializers;
};

struct FieldInfo {
  // Offset of the field within the containing STRUCT.
  unsigned Offset = 0;

```
- **EN**: Introduces declarations for `StructInfo`, `StructInitializer`, `FieldInfo`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `StructInfo`, `StructInitializer`, `FieldInfo` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 198-217
```cpp
  // Total size of the field (= LengthOf * Type).
  unsigned SizeOf = 0;

  // Number of elements in the field (1 if scalar, >1 if an array).
  unsigned LengthOf = 0;

  // Size of a single entry in this field, in bytes ("type" in MASM standards).
  unsigned Type = 0;

  FieldInitializer Contents;

  FieldInfo(FieldType FT) : Contents(FT) {}
};

StructFieldInfo::StructFieldInfo(std::vector<StructInitializer> V,
                                 StructInfo S) {
  Initializers = std::move(V);
  Structure = std::move(S);
}

```
- **EN**: Implements logic around `FieldInfo`, `StructFieldInfo`, `move`.
- **CN**: 围绕 `FieldInfo`, `StructFieldInfo`, `move` 实现具体逻辑。

### Lines 218-250
```cpp
StructInfo::StructInfo(StringRef StructName, bool Union,
                       unsigned AlignmentValue)
    : Name(StructName), IsUnion(Union), Alignment(AlignmentValue) {}

FieldInfo &StructInfo::addField(StringRef FieldName, FieldType FT,
                                unsigned FieldAlignmentSize) {
  if (!FieldName.empty())
    FieldsByName[FieldName.lower()] = Fields.size();
  Fields.emplace_back(FT);
  FieldInfo &Field = Fields.back();
  Field.Offset =
      llvm::alignTo(NextOffset, std::min(Alignment, FieldAlignmentSize));
  if (!IsUnion) {
    NextOffset = std::max(NextOffset, Field.Offset);
  }
  AlignmentSize = std::max(AlignmentSize, FieldAlignmentSize);
  return Field;
}

FieldInitializer::~FieldInitializer() {
  switch (FT) {
  case FT_INTEGRAL:
    IntInfo.~IntFieldInfo();
    break;
  case FT_REAL:
    RealInfo.~RealFieldInfo();
    break;
  case FT_STRUCT:
    StructInfo.~StructFieldInfo();
    break;
  }
}

```
- **EN**: Implements logic around `StructInfo`, `Name`, `addField`, `lower`, and 8 more symbols; this block uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `StructInfo`, `Name`, `addField`, `lower`, and 8 more symbols 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 251-274
```cpp
FieldInitializer::FieldInitializer(FieldType FT) : FT(FT) {
  switch (FT) {
  case FT_INTEGRAL:
    new (&IntInfo) IntFieldInfo();
    break;
  case FT_REAL:
    new (&RealInfo) RealFieldInfo();
    break;
  case FT_STRUCT:
    new (&StructInfo) StructFieldInfo();
    break;
  }
}

FieldInitializer::FieldInitializer(SmallVector<const MCExpr *, 1> &&Values)
    : FT(FT_INTEGRAL) {
  new (&IntInfo) IntFieldInfo(std::move(Values));
}

FieldInitializer::FieldInitializer(SmallVector<APInt, 1> &&AsIntValues)
    : FT(FT_REAL) {
  new (&RealInfo) RealFieldInfo(std::move(AsIntValues));
}

```
- **EN**: Implements logic around `FieldInitializer`, `new`, `FT`; this block uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `FieldInitializer`, `new`, `FT` 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 275-295
```cpp
FieldInitializer::FieldInitializer(
    std::vector<StructInitializer> &&Initializers, struct StructInfo Structure)
    : FT(FT_STRUCT) {
  new (&StructInfo) StructFieldInfo(std::move(Initializers), Structure);
}

FieldInitializer::FieldInitializer(const FieldInitializer &Initializer)
    : FT(Initializer.FT) {
  switch (FT) {
  case FT_INTEGRAL:
    new (&IntInfo) IntFieldInfo(Initializer.IntInfo);
    break;
  case FT_REAL:
    new (&RealInfo) RealFieldInfo(Initializer.RealInfo);
    break;
  case FT_STRUCT:
    new (&StructInfo) StructFieldInfo(Initializer.StructInfo);
    break;
  }
}

```
- **EN**: Introduces declarations for `StructInfo`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `StructInfo` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 296-335
```cpp
FieldInitializer::FieldInitializer(FieldInitializer &&Initializer)
    : FT(Initializer.FT) {
  switch (FT) {
  case FT_INTEGRAL:
    new (&IntInfo) IntFieldInfo(Initializer.IntInfo);
    break;
  case FT_REAL:
    new (&RealInfo) RealFieldInfo(Initializer.RealInfo);
    break;
  case FT_STRUCT:
    new (&StructInfo) StructFieldInfo(Initializer.StructInfo);
    break;
  }
}

FieldInitializer &
FieldInitializer::operator=(const FieldInitializer &Initializer) {
  if (FT != Initializer.FT) {
    switch (FT) {
    case FT_INTEGRAL:
      IntInfo.~IntFieldInfo();
      break;
    case FT_REAL:
      RealInfo.~RealFieldInfo();
      break;
    case FT_STRUCT:
      StructInfo.~StructFieldInfo();
      break;
    }
  }
  FT = Initializer.FT;
  switch (FT) {
  case FT_INTEGRAL:
    IntInfo = Initializer.IntInfo;
    break;
  case FT_REAL:
    RealInfo = Initializer.RealInfo;
    break;
  case FT_STRUCT:
    StructInfo = Initializer.StructInfo;
```
- **EN**: Implements logic around `FieldInitializer`, `FT`, `new`, `~IntFieldInfo`, and 2 more symbols; this block uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `FieldInitializer`, `FT`, `new`, `~IntFieldInfo`, and 2 more symbols 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 336-369
```cpp
    break;
  }
  return *this;
}

FieldInitializer &FieldInitializer::operator=(FieldInitializer &&Initializer) {
  if (FT != Initializer.FT) {
    switch (FT) {
    case FT_INTEGRAL:
      IntInfo.~IntFieldInfo();
      break;
    case FT_REAL:
      RealInfo.~RealFieldInfo();
      break;
    case FT_STRUCT:
      StructInfo.~StructFieldInfo();
      break;
    }
  }
  FT = Initializer.FT;
  switch (FT) {
  case FT_INTEGRAL:
    IntInfo = Initializer.IntInfo;
    break;
  case FT_REAL:
    RealInfo = Initializer.RealInfo;
    break;
  case FT_STRUCT:
    StructInfo = Initializer.StructInfo;
    break;
  }
  return *this;
}

```
- **EN**: Implements logic around `~IntFieldInfo`, `~RealFieldInfo`, `~StructFieldInfo`; this block uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `~IntFieldInfo`, `~RealFieldInfo`, `~StructFieldInfo` 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 370-390
```cpp
/// The concrete assembly parser instance.
// Note that this is a full MCAsmParser, not an MCAsmParserExtension!
// It's a peer of AsmParser, not of COFFAsmParser, WasmAsmParser, etc.
class MasmParser : public MCAsmParser {
private:
  SourceMgr::DiagHandlerTy SavedDiagHandler;
  void *SavedDiagContext;
  std::unique_ptr<MCAsmParserExtension> PlatformParser;

  /// This is the current buffer index we're lexing from as managed by the
  /// SourceMgr object.
  unsigned CurBuffer;

  /// time of assembly
  struct tm TM;

  BitVector EndStatementAtEOFStack;

  AsmCond TheCondState;
  std::vector<AsmCond> TheCondStack;

```
- **EN**: Introduces declarations for `MasmParser`, `tm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `MasmParser`, `tm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 391-410
```cpp
  /// maps directive names to handler methods in parser
  /// extensions. Extensions register themselves in this map by calling
  /// addDirectiveHandler.
  StringMap<ExtensionDirectiveHandler> ExtensionDirectiveMap;

  /// maps assembly-time variable names to variables.
  struct Variable {
    enum RedefinableKind { NOT_REDEFINABLE, WARN_ON_REDEFINITION, REDEFINABLE };

    StringRef Name;
    RedefinableKind Redefinable = REDEFINABLE;
    bool IsText = false;
    std::string TextValue;
  };
  StringMap<Variable> Variables;

  /// Stack of active struct definitions.
  SmallVector<StructInfo, 1> StructInProgress;

  /// Maps struct tags to struct definitions.
```
- **EN**: Introduces declarations for `Variable`, `RedefinableKind`, `definitions`, `tags`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `Variable`, `RedefinableKind`, `definitions`, `tags` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 411-434
```cpp
  StringMap<StructInfo> Structs;

  /// Maps data location names to types.
  StringMap<AsmTypeInfo> KnownType;

  /// Stack of active macro instantiations.
  std::vector<MacroInstantiation*> ActiveMacros;

  /// List of bodies of anonymous macros.
  std::deque<MCAsmMacro> MacroLikeBodies;

  /// Keeps track of how many .macro's have been instantiated.
  unsigned NumOfMacroInstantiations;

  /// The values from the last parsed cpp hash file line comment if any.
  struct CppHashInfoTy {
    StringRef Filename;
    int64_t LineNumber;
    SMLoc Loc;
    unsigned Buf;
    CppHashInfoTy() : LineNumber(0), Buf(0) {}
  };
  CppHashInfoTy CppHashInfo;

```
- **EN**: Introduces declarations for `CppHashInfoTy`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `CppHashInfoTy` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 435-460
```cpp
  /// The filename from the first cpp hash file line comment, if any.
  StringRef FirstCppHashFilename;

  /// List of forward directional labels for diagnosis at the end.
  SmallVector<std::tuple<SMLoc, CppHashInfoTy, MCSymbol *>, 4> DirLabels;

  /// AssemblerDialect. ~OU means unset value and use value provided by MAI.
  /// Defaults to 1U, meaning Intel.
  unsigned AssemblerDialect = 1U;

  /// Are we parsing ms-style inline assembly?
  bool ParsingMSInlineAsm = false;

  // Current <...> expression depth.
  unsigned AngleBracketDepth = 0U;

  // Number of locals defined.
  uint16_t LocalCounter = 0;

public:
  MasmParser(SourceMgr &SM, MCContext &Ctx, MCStreamer &Out,
             const MCAsmInfo &MAI, struct tm TM, unsigned CB = 0);
  MasmParser(const MasmParser &) = delete;
  MasmParser &operator=(const MasmParser &) = delete;
  ~MasmParser() override;

```
- **EN**: Introduces declarations for `tm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `tm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 461-485
```cpp
  bool Run(bool NoInitialTextSection, bool NoFinalize = false) override;

  void addDirectiveHandler(StringRef Directive,
                           ExtensionDirectiveHandler Handler) override {
    ExtensionDirectiveMap[Directive] = std::move(Handler);
    DirectiveKindMap.try_emplace(Directive, DK_HANDLER_DIRECTIVE);
  }

  void addAliasForDirective(StringRef Directive, StringRef Alias) override {
    DirectiveKindMap[Directive] = DirectiveKindMap[Alias];
  }

  /// @name MCAsmParser Interface
  /// {

  unsigned getAssemblerDialect() override {
    if (AssemblerDialect == ~0U)
      return MAI.getAssemblerDialect();
    else
      return AssemblerDialect;
  }
  void setAssemblerDialect(unsigned i) override {
    AssemblerDialect = i;
  }

```
- **EN**: Implements logic around `Run`, `addDirectiveHandler`, `move`, `try_emplace`, and 3 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `Run`, `addDirectiveHandler`, `move`, `try_emplace`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 486-505
```cpp
  void Note(SMLoc L, const Twine &Msg, SMRange Range = {}) override;
  bool Warning(SMLoc L, const Twine &Msg, SMRange Range = {}) override;
  bool printError(SMLoc L, const Twine &Msg, SMRange Range = {}) override;

  enum ExpandKind { ExpandMacros, DoNotExpandMacros };
  const AsmToken &Lex(ExpandKind ExpandNextToken);
  const AsmToken &Lex() override { return Lex(ExpandMacros); }

  void setParsingMSInlineAsm(bool V) override {
    ParsingMSInlineAsm = V;
    // When parsing MS inline asm, we must lex 0b1101 and 0ABCH as binary and
    // hex integer literals.
    Lexer.setLexMasmIntegers(V);
  }
  bool isParsingMSInlineAsm() override { return ParsingMSInlineAsm; }

  bool isParsingMasm() const override { return true; }

  bool defineMacro(StringRef Name, StringRef Value) override;

```
- **EN**: Introduces declarations for `ExpandKind`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `ExpandKind` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 506-526
```cpp
  bool lookUpField(StringRef Name, AsmFieldInfo &Info) const override;
  bool lookUpField(StringRef Base, StringRef Member,
                   AsmFieldInfo &Info) const override;

  bool lookUpType(StringRef Name, AsmTypeInfo &Info) const override;

  bool parseMSInlineAsm(std::string &AsmString, unsigned &NumOutputs,
                        unsigned &NumInputs,
                        SmallVectorImpl<std::pair<void *, bool>> &OpDecls,
                        SmallVectorImpl<std::string> &Constraints,
                        SmallVectorImpl<std::string> &Clobbers,
                        const MCInstrInfo *MII, MCInstPrinter *IP,
                        MCAsmParserSemaCallback &SI) override;

  bool parseExpression(const MCExpr *&Res);
  bool parseExpression(const MCExpr *&Res, SMLoc &EndLoc) override;
  bool parsePrimaryExpr(const MCExpr *&Res, SMLoc &EndLoc,
                        AsmTypeInfo *TypeInfo) override;
  bool parseParenExpression(const MCExpr *&Res, SMLoc &EndLoc) override;
  bool parseAbsoluteExpression(int64_t &Res) override;

```
- **EN**: Implements logic around `lookUpField`, `lookUpType`, `parseMSInlineAsm`, `parseExpression`, and 3 more symbols; this block parses assembly syntax or operands.
- **CN**: 围绕 `lookUpField`, `lookUpType`, `parseMSInlineAsm`, `parseExpression`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 527-547
```cpp
  /// Parse a floating point expression using the float \p Semantics
  /// and set \p Res to the value.
  bool parseRealValue(const fltSemantics &Semantics, APInt &Res);

  /// Parse an identifier or string (as a quoted identifier)
  /// and set \p Res to the identifier contents.
  enum IdentifierPositionKind { StandardPosition, StartOfStatement };
  bool parseIdentifier(StringRef &Res, IdentifierPositionKind Position);
  bool parseIdentifier(StringRef &Res) override {
    return parseIdentifier(Res, StandardPosition);
  }
  void eatToEndOfStatement() override;

  bool checkForValidSection() override;

  /// }

private:
  bool expandMacros();
  const AsmToken peekTok(bool ShouldSkipSpace = true);

```
- **EN**: Introduces declarations for `IdentifierPositionKind`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `IdentifierPositionKind` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 548-568
```cpp
  bool parseStatement(ParseStatementInfo &Info,
                      MCAsmParserSemaCallback *SI);
  bool parseCurlyBlockScope(SmallVectorImpl<AsmRewrite>& AsmStrRewrites);
  bool parseCppHashLineFilenameComment(SMLoc L);

  bool expandMacro(raw_svector_ostream &OS, StringRef Body,
                   ArrayRef<MCAsmMacroParameter> Parameters,
                   ArrayRef<MCAsmMacroArgument> A,
                   const std::vector<std::string> &Locals, SMLoc L);

  /// Are we inside a macro instantiation?
  bool isInsideMacroInstantiation() {return !ActiveMacros.empty();}

  /// Handle entry to macro instantiation.
  ///
  /// \param M The macro.
  /// \param NameLoc Instantiation location.
  bool handleMacroEntry(
      const MCAsmMacro *M, SMLoc NameLoc,
      AsmToken::TokenKind ArgumentEndTok = AsmToken::EndOfStatement);

```
- **EN**: Implements logic around `parseStatement`, `parseCurlyBlockScope`, `parseCppHashLineFilenameComment`, `expandMacro`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes; parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `parseStatement`, `parseCurlyBlockScope`, `parseCppHashLineFilenameComment`, `expandMacro`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 569-589
```cpp
  /// Handle invocation of macro function.
  ///
  /// \param M The macro.
  /// \param NameLoc Invocation location.
  bool handleMacroInvocation(const MCAsmMacro *M, SMLoc NameLoc);

  /// Handle exit from macro instantiation.
  void handleMacroExit();

  /// Extract AsmTokens for a macro argument.
  bool
  parseMacroArgument(const MCAsmMacroParameter *MP, MCAsmMacroArgument &MA,
                     AsmToken::TokenKind EndTok = AsmToken::EndOfStatement);

  /// Parse all macro arguments for a given macro.
  bool
  parseMacroArguments(const MCAsmMacro *M, MCAsmMacroArguments &A,
                      AsmToken::TokenKind EndTok = AsmToken::EndOfStatement);

  void printMacroInstantiations();

```
- **EN**: Implements logic around `handleMacroInvocation`, `handleMacroExit`, `parseMacroArgument`, `parseMacroArguments`, and 1 more symbols; this block parses assembly syntax or operands.
- **CN**: 围绕 `handleMacroInvocation`, `handleMacroExit`, `parseMacroArgument`, `parseMacroArguments`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 590-609
```cpp
  bool expandStatement(SMLoc Loc);

  void printMessage(SMLoc Loc, SourceMgr::DiagKind Kind, const Twine &Msg,
                    SMRange Range = {}) const {
    ArrayRef<SMRange> Ranges(Range);
    SrcMgr.PrintMessage(Loc, Kind, Msg, Ranges);
  }
  static void DiagHandler(const SMDiagnostic &Diag, void *Context);

  bool lookUpField(const StructInfo &Structure, StringRef Member,
                   AsmFieldInfo &Info) const;

  /// Enter the specified file. This returns true on failure.
  bool enterIncludeFile(const std::string &Filename);

  /// Reset the current lexer position to that given by \p Loc. The
  /// current token is not set; clients should ensure Lex() is called
  /// subsequently.
  ///
  /// \param InBuffer If not 0, should be the known buffer id that contains the
```
- **EN**: Implements logic around `expandStatement`, `printMessage`, `Ranges`, `PrintMessage`, and 3 more symbols; this block parses assembly syntax or operands.
- **CN**: 围绕 `expandStatement`, `printMessage`, `Ranges`, `PrintMessage`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 610-630
```cpp
  /// location.
  void jumpToLoc(SMLoc Loc, unsigned InBuffer = 0,
                 bool EndStatementAtEOF = true);

  /// Parse up to a token of kind \p EndTok and return the contents from the
  /// current token up to (but not including) this token; the current token on
  /// exit will be either this kind or EOF. Reads through instantiated macro
  /// functions and text macros.
  SmallVector<StringRef, 1> parseStringRefsTo(AsmToken::TokenKind EndTok);
  std::string parseStringTo(AsmToken::TokenKind EndTok);

  /// Parse up to the end of statement and return the contents from the current
  /// token until the end of the statement; the current token on exit will be
  /// either the EndOfStatement or EOF.
  StringRef parseStringToEndOfStatement() override;

  bool parseTextItem(std::string &Data);

  unsigned getBinOpPrecedence(AsmToken::TokenKind K,
                              MCBinaryExpr::Opcode &Kind);

```
- **EN**: Implements logic around `jumpToLoc`, `parseStringRefsTo`, `parseStringTo`, `parseStringToEndOfStatement`, and 2 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `jumpToLoc`, `parseStringRefsTo`, `parseStringTo`, `parseStringToEndOfStatement`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 631-670
```cpp
  bool parseBinOpRHS(unsigned Precedence, const MCExpr *&Res, SMLoc &EndLoc);
  bool parseParenExpr(const MCExpr *&Res, SMLoc &EndLoc);
  bool parseBracketExpr(const MCExpr *&Res, SMLoc &EndLoc);

  // Generic (target and platform independent) directive parsing.
  enum DirectiveKind {
    DK_NO_DIRECTIVE, // Placeholder
    DK_HANDLER_DIRECTIVE,
    DK_ASSIGN,
    DK_EQU,
    DK_TEXTEQU,
    DK_ASCII,
    DK_ASCIZ,
    DK_STRING,
    DK_BYTE,
    DK_SBYTE,
    DK_WORD,
    DK_SWORD,
    DK_DWORD,
    DK_SDWORD,
    DK_FWORD,
    DK_QWORD,
    DK_SQWORD,
    DK_DB,
    DK_DD,
    DK_DF,
    DK_DQ,
    DK_DW,
    DK_REAL4,
    DK_REAL8,
    DK_REAL10,
    DK_ALIGN,
    DK_EVEN,
    DK_ORG,
    DK_ENDR,
    DK_EXTERN,
    DK_PUBLIC,
    DK_COMM,
    DK_COMMENT,
    DK_INCLUDE,
```
- **EN**: Introduces declarations for `DirectiveKind`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `DirectiveKind` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 671-697
```cpp
    DK_REPEAT,
    DK_WHILE,
    DK_FOR,
    DK_FORC,
    DK_IF,
    DK_IFE,
    DK_IFB,
    DK_IFNB,
    DK_IFDEF,
    DK_IFNDEF,
    DK_IFDIF,
    DK_IFDIFI,
    DK_IFIDN,
    DK_IFIDNI,
    DK_ELSEIF,
    DK_ELSEIFE,
    DK_ELSEIFB,
    DK_ELSEIFNB,
    DK_ELSEIFDEF,
    DK_ELSEIFNDEF,
    DK_ELSEIFDIF,
    DK_ELSEIFDIFI,
    DK_ELSEIFIDN,
    DK_ELSEIFIDNI,
    DK_ELSE,
    DK_ENDIF,

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 698-725
```cpp
    DK_MACRO,
    DK_EXITM,
    DK_ENDM,
    DK_PURGE,
    DK_ERR,
    DK_ERRB,
    DK_ERRNB,
    DK_ERRDEF,
    DK_ERRNDEF,
    DK_ERRDIF,
    DK_ERRDIFI,
    DK_ERRIDN,
    DK_ERRIDNI,
    DK_ERRE,
    DK_ERRNZ,
    DK_ECHO,
    DK_STRUCT,
    DK_UNION,
    DK_ENDS,
    DK_END,
    DK_PUSHFRAME,
    DK_PUSHREG,
    DK_SAVEREG,
    DK_SAVEXMM128,
    DK_SETFRAME,
    DK_RADIX,
  };

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 726-753
```cpp
  /// Maps directive name --> DirectiveKind enum, for directives parsed by this
  /// class.
  StringMap<DirectiveKind> DirectiveKindMap;

  bool isMacroLikeDirective();

  // Generic (target and platform independent) directive parsing.
  enum BuiltinSymbol {
    BI_NO_SYMBOL, // Placeholder
    BI_DATE,
    BI_TIME,
    BI_VERSION,
    BI_FILECUR,
    BI_FILENAME,
    BI_LINE,
    BI_CURSEG,
    BI_CPU,
    BI_INTERFACE,
    BI_CODE,
    BI_DATA,
    BI_FARDATA,
    BI_WORDSIZE,
    BI_CODESIZE,
    BI_DATASIZE,
    BI_MODEL,
    BI_STACK,
  };

```
- **EN**: Introduces declarations for `BuiltinSymbol`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `BuiltinSymbol` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 754-775
```cpp
  /// Maps builtin name --> BuiltinSymbol enum, for builtins handled by this
  /// class.
  StringMap<BuiltinSymbol> BuiltinSymbolMap;

  const MCExpr *evaluateBuiltinValue(BuiltinSymbol Symbol, SMLoc StartLoc);

  std::optional<std::string> evaluateBuiltinTextMacro(BuiltinSymbol Symbol,
                                                      SMLoc StartLoc);

  // Generic (target and platform independent) directive parsing.
  enum BuiltinFunction {
    BI_NO_FUNCTION, // Placeholder
    BI_CATSTR,
  };

  /// Maps builtin name --> BuiltinFunction enum, for builtins handled by this
  /// class.
  StringMap<BuiltinFunction> BuiltinFunctionMap;

  bool evaluateBuiltinMacroFunction(BuiltinFunction Function, StringRef Name,
                                    std::string &Res);

```
- **EN**: Introduces declarations for `BuiltinFunction`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `BuiltinFunction` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 776-805
```cpp
  // ".ascii", ".asciz", ".string"
  bool parseDirectiveAscii(StringRef IDVal, bool ZeroTerminated);

  // "byte", "word", ...
  bool emitIntValue(const MCExpr *Value, unsigned Size);
  bool parseScalarInitializer(unsigned Size,
                              SmallVectorImpl<const MCExpr *> &Values,
                              unsigned StringPadLength = 0);
  bool parseScalarInstList(
      unsigned Size, SmallVectorImpl<const MCExpr *> &Values,
      const AsmToken::TokenKind EndToken = AsmToken::EndOfStatement);
  bool emitIntegralValues(unsigned Size, unsigned *Count = nullptr);
  bool addIntegralField(StringRef Name, unsigned Size);
  bool parseDirectiveValue(StringRef IDVal, unsigned Size);
  bool parseDirectiveNamedValue(StringRef TypeName, unsigned Size,
                                StringRef Name, SMLoc NameLoc);

  // "real4", "real8", "real10"
  bool emitRealValues(const fltSemantics &Semantics, unsigned *Count = nullptr);
  bool addRealField(StringRef Name, const fltSemantics &Semantics, size_t Size);
  bool parseDirectiveRealValue(StringRef IDVal, const fltSemantics &Semantics,
                               size_t Size);
  bool parseRealInstList(
      const fltSemantics &Semantics, SmallVectorImpl<APInt> &Values,
      const AsmToken::TokenKind EndToken = AsmToken::EndOfStatement);
  bool parseDirectiveNamedRealValue(StringRef TypeName,
                                    const fltSemantics &Semantics,
                                    unsigned Size, StringRef Name,
                                    SMLoc NameLoc);

```
- **EN**: Implements logic around `parseDirectiveAscii`, `emitIntValue`, `parseScalarInitializer`, `parseScalarInstList`, and 9 more symbols; this block parses assembly syntax or operands.
- **CN**: 围绕 `parseDirectiveAscii`, `emitIntValue`, `parseScalarInitializer`, `parseScalarInstList`, and 9 more symbols 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 806-826
```cpp
  bool parseOptionalAngleBracketOpen();
  bool parseAngleBracketClose(const Twine &Msg = "expected '>'");

  bool parseFieldInitializer(const FieldInfo &Field,
                             FieldInitializer &Initializer);
  bool parseFieldInitializer(const FieldInfo &Field,
                             const IntFieldInfo &Contents,
                             FieldInitializer &Initializer);
  bool parseFieldInitializer(const FieldInfo &Field,
                             const RealFieldInfo &Contents,
                             FieldInitializer &Initializer);
  bool parseFieldInitializer(const FieldInfo &Field,
                             const StructFieldInfo &Contents,
                             FieldInitializer &Initializer);

  bool parseStructInitializer(const StructInfo &Structure,
                              StructInitializer &Initializer);
  bool parseStructInstList(
      const StructInfo &Structure, std::vector<StructInitializer> &Initializers,
      const AsmToken::TokenKind EndToken = AsmToken::EndOfStatement);

```
- **EN**: Implements logic around `parseOptionalAngleBracketOpen`, `parseAngleBracketClose`, `parseFieldInitializer`, `parseStructInitializer`, and 1 more symbols; this block parses assembly syntax or operands.
- **CN**: 围绕 `parseOptionalAngleBracketOpen`, `parseAngleBracketClose`, `parseFieldInitializer`, `parseStructInitializer`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 827-846
```cpp
  bool emitFieldValue(const FieldInfo &Field);
  bool emitFieldValue(const FieldInfo &Field, const IntFieldInfo &Contents);
  bool emitFieldValue(const FieldInfo &Field, const RealFieldInfo &Contents);
  bool emitFieldValue(const FieldInfo &Field, const StructFieldInfo &Contents);

  bool emitFieldInitializer(const FieldInfo &Field,
                            const FieldInitializer &Initializer);
  bool emitFieldInitializer(const FieldInfo &Field,
                            const IntFieldInfo &Contents,
                            const IntFieldInfo &Initializer);
  bool emitFieldInitializer(const FieldInfo &Field,
                            const RealFieldInfo &Contents,
                            const RealFieldInfo &Initializer);
  bool emitFieldInitializer(const FieldInfo &Field,
                            const StructFieldInfo &Contents,
                            const StructFieldInfo &Initializer);

  bool emitStructInitializer(const StructInfo &Structure,
                             const StructInitializer &Initializer);

```
- **EN**: Implements logic around `emitFieldValue`, `emitFieldInitializer`, `emitStructInitializer`.
- **CN**: 围绕 `emitFieldValue`, `emitFieldInitializer`, `emitStructInitializer` 实现具体逻辑。

### Lines 847-872
```cpp
  // User-defined types (structs, unions):
  bool emitStructValues(const StructInfo &Structure, unsigned *Count = nullptr);
  bool addStructField(StringRef Name, const StructInfo &Structure);
  bool parseDirectiveStructValue(const StructInfo &Structure,
                                 StringRef Directive, SMLoc DirLoc);
  bool parseDirectiveNamedStructValue(const StructInfo &Structure,
                                      StringRef Directive, SMLoc DirLoc,
                                      StringRef Name);

  // "=", "equ", "textequ"
  bool parseDirectiveEquate(StringRef IDVal, StringRef Name,
                            DirectiveKind DirKind, SMLoc NameLoc);

  bool parseDirectiveOrg(); // "org"

  bool emitAlignTo(int64_t Alignment);
  bool parseDirectiveAlign();  // "align"
  bool parseDirectiveEven();   // "even"

  // macro directives
  bool parseDirectivePurgeMacro(SMLoc DirectiveLoc);
  bool parseDirectiveExitMacro(SMLoc DirectiveLoc, StringRef Directive,
                               std::string &Value);
  bool parseDirectiveEndMacro(StringRef Directive);
  bool parseDirectiveMacro(StringRef Name, SMLoc NameLoc);

```
- **EN**: Implements logic around `emitStructValues`, `addStructField`, `parseDirectiveStructValue`, `parseDirectiveNamedStructValue`, and 9 more symbols; this block parses assembly syntax or operands.
- **CN**: 围绕 `emitStructValues`, `addStructField`, `parseDirectiveStructValue`, `parseDirectiveNamedStructValue`, and 9 more symbols 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 873-912
```cpp
  bool parseDirectiveStruct(StringRef Directive, DirectiveKind DirKind,
                            StringRef Name, SMLoc NameLoc);
  bool parseDirectiveNestedStruct(StringRef Directive, DirectiveKind DirKind);
  bool parseDirectiveEnds(StringRef Name, SMLoc NameLoc);
  bool parseDirectiveNestedEnds();

  bool parseDirectiveExtern();

  /// Parse a directive like ".globl" which accepts a single symbol (which
  /// should be a label or an external).
  bool parseDirectiveSymbolAttribute(MCSymbolAttr Attr);

  bool parseDirectiveComm(bool IsLocal); // ".comm" and ".lcomm"

  bool parseDirectiveComment(SMLoc DirectiveLoc); // "comment"

  bool parseDirectiveInclude(); // "include"

  // "if" or "ife"
  bool parseDirectiveIf(SMLoc DirectiveLoc, DirectiveKind DirKind);
  // "ifb" or "ifnb", depending on ExpectBlank.
  bool parseDirectiveIfb(SMLoc DirectiveLoc, bool ExpectBlank);
  // "ifidn", "ifdif", "ifidni", or "ifdifi", depending on ExpectEqual and
  // CaseInsensitive.
  bool parseDirectiveIfidn(SMLoc DirectiveLoc, bool ExpectEqual,
                           bool CaseInsensitive);
  // "ifdef" or "ifndef", depending on expect_defined
  bool parseDirectiveIfdef(SMLoc DirectiveLoc, bool expect_defined);
  // "elseif" or "elseife"
  bool parseDirectiveElseIf(SMLoc DirectiveLoc, DirectiveKind DirKind);
  // "elseifb" or "elseifnb", depending on ExpectBlank.
  bool parseDirectiveElseIfb(SMLoc DirectiveLoc, bool ExpectBlank);
  // ".elseifdef" or ".elseifndef", depending on expect_defined
  bool parseDirectiveElseIfdef(SMLoc DirectiveLoc, bool expect_defined);
  // "elseifidn", "elseifdif", "elseifidni", or "elseifdifi", depending on
  // ExpectEqual and CaseInsensitive.
  bool parseDirectiveElseIfidn(SMLoc DirectiveLoc, bool ExpectEqual,
                               bool CaseInsensitive);
  bool parseDirectiveElse(SMLoc DirectiveLoc);   // "else"
  bool parseDirectiveEndIf(SMLoc DirectiveLoc);  // "endif"
```
- **EN**: Implements logic around `parseDirectiveStruct`, `parseDirectiveNestedStruct`, `parseDirectiveEnds`, `parseDirectiveNestedEnds`, and 15 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state.
- **CN**: 围绕 `parseDirectiveStruct`, `parseDirectiveNestedStruct`, `parseDirectiveEnds`, `parseDirectiveNestedEnds`, and 15 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态。

### Lines 913-933
```cpp
  bool parseEscapedString(std::string &Data) override;
  bool parseAngleBracketString(std::string &Data) override;

  // Macro-like directives
  MCAsmMacro *parseMacroLikeBody(SMLoc DirectiveLoc);
  void instantiateMacroLikeBody(MCAsmMacro *M, SMLoc DirectiveLoc,
                                raw_svector_ostream &OS);
  void instantiateMacroLikeBody(MCAsmMacro *M, SMLoc DirectiveLoc,
                                SMLoc ExitLoc, raw_svector_ostream &OS);
  bool parseDirectiveRepeat(SMLoc DirectiveLoc, StringRef Directive);
  bool parseDirectiveFor(SMLoc DirectiveLoc, StringRef Directive);
  bool parseDirectiveForc(SMLoc DirectiveLoc, StringRef Directive);
  bool parseDirectiveWhile(SMLoc DirectiveLoc);

  // "_emit" or "__emit"
  bool parseDirectiveMSEmit(SMLoc DirectiveLoc, ParseStatementInfo &Info,
                            size_t Len);

  // "align"
  bool parseDirectiveMSAlign(SMLoc DirectiveLoc, ParseStatementInfo &Info);

```
- **EN**: Implements logic around `parseEscapedString`, `parseAngleBracketString`, `parseMacroLikeBody`, `instantiateMacroLikeBody`, and 6 more symbols; this block parses assembly syntax or operands.
- **CN**: 围绕 `parseEscapedString`, `parseAngleBracketString`, `parseMacroLikeBody`, `instantiateMacroLikeBody`, and 6 more symbols 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 934-955
```cpp
  // "end"
  bool parseDirectiveEnd(SMLoc DirectiveLoc);

  // ".err"
  bool parseDirectiveError(SMLoc DirectiveLoc);
  // ".errb" or ".errnb", depending on ExpectBlank.
  bool parseDirectiveErrorIfb(SMLoc DirectiveLoc, bool ExpectBlank);
  // ".errdef" or ".errndef", depending on ExpectBlank.
  bool parseDirectiveErrorIfdef(SMLoc DirectiveLoc, bool ExpectDefined);
  // ".erridn", ".errdif", ".erridni", or ".errdifi", depending on ExpectEqual
  // and CaseInsensitive.
  bool parseDirectiveErrorIfidn(SMLoc DirectiveLoc, bool ExpectEqual,
                                bool CaseInsensitive);
  // ".erre" or ".errnz", depending on ExpectZero.
  bool parseDirectiveErrorIfe(SMLoc DirectiveLoc, bool ExpectZero);

  // ".radix"
  bool parseDirectiveRadix(SMLoc DirectiveLoc);

  // "echo"
  bool parseDirectiveEcho(SMLoc DirectiveLoc);

```
- **EN**: Implements logic around `parseDirectiveEnd`, `parseDirectiveError`, `parseDirectiveErrorIfb`, `parseDirectiveErrorIfdef`, and 4 more symbols; this block parses assembly syntax or operands.
- **CN**: 围绕 `parseDirectiveEnd`, `parseDirectiveError`, `parseDirectiveErrorIfb`, `parseDirectiveErrorIfdef`, and 4 more symbols 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 956-982
```cpp
  void initializeDirectiveKindMap();
  void initializeBuiltinSymbolMaps();
};

} // end anonymous namespace

namespace llvm {

extern cl::opt<unsigned> AsmMacroMaxNestingDepth;

} // end namespace llvm

enum { DEFAULT_ADDRSPACE = 0 };

MasmParser::MasmParser(SourceMgr &SM, MCContext &Ctx, MCStreamer &Out,
                       const MCAsmInfo &MAI, struct tm TM, unsigned CB)
    : MCAsmParser(Ctx, Out, SM, MAI), CurBuffer(CB ? CB : SM.getMainFileID()),
      TM(TM) {
  HadError = false;
  // Save the old handler.
  SavedDiagHandler = SrcMgr.getDiagHandler();
  SavedDiagContext = SrcMgr.getDiagContext();
  // Set our own handler which calls the saved handler.
  SrcMgr.setDiagHandler(DiagHandler, this);
  Lexer.setBuffer(SrcMgr.getMemoryBuffer(CurBuffer)->getBuffer());
  EndStatementAtEOFStack.push_back(true);

```
- **EN**: Introduces declarations for `llvm`, `tm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `tm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 983-1003
```cpp
  // Initialize the platform / file format parser.
  switch (Ctx.getObjectFileType()) {
  case MCContext::IsCOFF:
    PlatformParser.reset(createCOFFMasmParser());
    break;
  default:
    report_fatal_error("llvm-ml currently supports only COFF output.");
    break;
  }

  initializeDirectiveKindMap();
  PlatformParser->Initialize(*this);
  initializeBuiltinSymbolMaps();

  NumOfMacroInstantiations = 0;
}

MasmParser::~MasmParser() {
  assert((HadError || ActiveMacros.empty()) &&
         "Unexpected active macro instantiation!");

```
- **EN**: Implements logic around `reset`, `report_fatal_error`, `initializeDirectiveKindMap`, `Initialize`, and 3 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `reset`, `report_fatal_error`, `initializeDirectiveKindMap`, `Initialize`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 1004-1024
```cpp
  // Restore the saved diagnostics handler and context for use during
  // finalization.
  SrcMgr.setDiagHandler(SavedDiagHandler, SavedDiagContext);
}

void MasmParser::printMacroInstantiations() {
  // Print the active macro instantiation stack.
  for (std::vector<MacroInstantiation *>::const_reverse_iterator
           it = ActiveMacros.rbegin(),
           ie = ActiveMacros.rend();
       it != ie; ++it)
    printMessage((*it)->InstantiationLoc, SourceMgr::DK_Note,
                 "while in macro instantiation");
}

void MasmParser::Note(SMLoc L, const Twine &Msg, SMRange Range) {
  printPendingErrors();
  printMessage(L, SourceMgr::DK_Note, Msg, Range);
  printMacroInstantiations();
}

```
- **EN**: Implements logic around `setDiagHandler`, `printMacroInstantiations`, `rbegin`, `rend`, and 3 more symbols; this block parses assembly syntax or operands.
- **CN**: 围绕 `setDiagHandler`, `printMacroInstantiations`, `rbegin`, `rend`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 1025-1048
```cpp
bool MasmParser::Warning(SMLoc L, const Twine &Msg, SMRange Range) {
  if (getTargetParser().getTargetOptions().MCNoWarn)
    return false;
  if (getTargetParser().getTargetOptions().MCFatalWarnings)
    return Error(L, Msg, Range);
  printMessage(L, SourceMgr::DK_Warning, Msg, Range);
  printMacroInstantiations();
  return false;
}

bool MasmParser::printError(SMLoc L, const Twine &Msg, SMRange Range) {
  HadError = true;
  printMessage(L, SourceMgr::DK_Error, Msg, Range);
  printMacroInstantiations();
  return true;
}

bool MasmParser::enterIncludeFile(const std::string &Filename) {
  std::string IncludedFile;
  unsigned NewBuf =
      SrcMgr.AddIncludeFile(Filename, Lexer.getLoc(), IncludedFile);
  if (!NewBuf)
    return true;

```
- **EN**: Implements logic around `Warning`, `Error`, `printMessage`, `printMacroInstantiations`, and 3 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `Warning`, `Error`, `printMessage`, `printMacroInstantiations`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 1049-1078
```cpp
  CurBuffer = NewBuf;
  Lexer.setBuffer(SrcMgr.getMemoryBuffer(CurBuffer)->getBuffer());
  EndStatementAtEOFStack.push_back(true);
  return false;
}

void MasmParser::jumpToLoc(SMLoc Loc, unsigned InBuffer,
                           bool EndStatementAtEOF) {
  CurBuffer = InBuffer ? InBuffer : SrcMgr.FindBufferContainingLoc(Loc);
  Lexer.setBuffer(SrcMgr.getMemoryBuffer(CurBuffer)->getBuffer(),
                  Loc.getPointer(), EndStatementAtEOF);
}

bool MasmParser::expandMacros() {
  const AsmToken &Tok = getTok();
  const std::string IDLower = Tok.getIdentifier().lower();

  const llvm::MCAsmMacro *M = getContext().lookupMacro(IDLower);
  if (M && M->IsFunction && peekTok().is(AsmToken::LParen)) {
    // This is a macro function invocation; expand it in place.
    const SMLoc MacroLoc = Tok.getLoc();
    const StringRef MacroId = Tok.getIdentifier();
    Lexer.Lex();
    if (handleMacroInvocation(M, MacroLoc)) {
      Lexer.UnLex(AsmToken(AsmToken::Error, MacroId));
      Lexer.Lex();
    }
    return false;
  }

```
- **EN**: Implements logic around `setBuffer`, `push_back`, `jumpToLoc`, `FindBufferContainingLoc`, and 8 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `setBuffer`, `push_back`, `jumpToLoc`, `FindBufferContainingLoc`, and 8 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 1079-1100
```cpp
  std::optional<std::string> ExpandedValue;

  if (auto BuiltinIt = BuiltinSymbolMap.find(IDLower);
      BuiltinIt != BuiltinSymbolMap.end()) {
    ExpandedValue =
        evaluateBuiltinTextMacro(BuiltinIt->getValue(), Tok.getLoc());
  } else if (auto BuiltinFuncIt = BuiltinFunctionMap.find(IDLower);
             BuiltinFuncIt != BuiltinFunctionMap.end()) {
    StringRef Name;
    if (parseIdentifier(Name)) {
      return true;
    }
    std::string Res;
    if (evaluateBuiltinMacroFunction(BuiltinFuncIt->getValue(), Name, Res)) {
      return true;
    }
    ExpandedValue = Res;
  } else if (auto VarIt = Variables.find(IDLower);
             VarIt != Variables.end() && VarIt->getValue().IsText) {
    ExpandedValue = VarIt->getValue().TextValue;
  }

```
- **EN**: Implements logic around `end`, `evaluateBuiltinTextMacro`, `getValue`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `end`, `evaluateBuiltinTextMacro`, `getValue` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1101-1120
```cpp
  if (!ExpandedValue)
    return true;
  std::unique_ptr<MemoryBuffer> Instantiation =
      MemoryBuffer::getMemBufferCopy(*ExpandedValue, "<instantiation>");

  // Jump to the macro instantiation and prime the lexer.
  CurBuffer =
      SrcMgr.AddNewSourceBuffer(std::move(Instantiation), Tok.getEndLoc());
  Lexer.setBuffer(SrcMgr.getMemoryBuffer(CurBuffer)->getBuffer(), nullptr,
                  /*EndStatementAtEOF=*/false);
  EndStatementAtEOFStack.push_back(false);
  Lexer.Lex();
  return false;
}

const AsmToken &MasmParser::Lex(ExpandKind ExpandNextToken) {
  if (Lexer.getTok().is(AsmToken::Error))
    Error(Lexer.getErrLoc(), Lexer.getErr());
  bool StartOfStatement = false;

```
- **EN**: Implements logic around `getMemBufferCopy`, `AddNewSourceBuffer`, `setBuffer`, `push_back`, and 2 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `getMemBufferCopy`, `AddNewSourceBuffer`, `setBuffer`, `push_back`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 1121-1148
```cpp
  // if it's a end of statement with a comment in it
  if (getTok().is(AsmToken::EndOfStatement)) {
    // if this is a line comment output it.
    if (!getTok().getString().empty() && getTok().getString().front() != '\n' &&
        getTok().getString().front() != '\r' && MAI.preserveAsmComments())
      Out.addExplicitComment(Twine(getTok().getString()));
    StartOfStatement = true;
  }

  const AsmToken *tok = &Lexer.Lex();

  while (ExpandNextToken == ExpandMacros && tok->is(AsmToken::Identifier)) {
    if (StartOfStatement) {
      AsmToken NextTok;
      MutableArrayRef<AsmToken> Buf(NextTok);
      size_t ReadCount = Lexer.peekTokens(Buf);
      if (ReadCount && NextTok.is(AsmToken::Identifier) &&
          (NextTok.getString().equals_insensitive("equ") ||
           NextTok.getString().equals_insensitive("textequ"))) {
        // This looks like an EQU or TEXTEQU directive; don't expand the
        // identifier, allowing for redefinitions.
        break;
      }
    }
    if (expandMacros())
      break;
  }

```
- **EN**: Implements logic around `getTok`, `addExplicitComment`, `Lex`, `Buf`, and 2 more symbols; this block parses assembly syntax or operands.
- **CN**: 围绕 `getTok`, `addExplicitComment`, `Lex`, `Buf`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 1149-1176
```cpp
  // Parse comments here to be deferred until end of next statement.
  while (tok->is(AsmToken::Comment)) {
    if (MAI.preserveAsmComments())
      Out.addExplicitComment(Twine(tok->getString()));
    tok = &Lexer.Lex();
  }

  // Recognize and bypass line continuations.
  while (tok->is(AsmToken::BackSlash) &&
         peekTok().is(AsmToken::EndOfStatement)) {
    // Eat both the backslash and the end of statement.
    Lexer.Lex();
    tok = &Lexer.Lex();
  }

  if (tok->is(AsmToken::Eof)) {
    // If this is the end of an included file, pop the parent file off the
    // include stack.
    SMLoc ParentIncludeLoc = SrcMgr.getParentIncludeLoc(CurBuffer);
    if (ParentIncludeLoc != SMLoc()) {
      EndStatementAtEOFStack.pop_back();
      jumpToLoc(ParentIncludeLoc, 0, EndStatementAtEOFStack.back());
      return Lex();
    }
    EndStatementAtEOFStack.pop_back();
    assert(EndStatementAtEOFStack.empty());
  }

```
- **EN**: Implements logic around `addExplicitComment`, `Lex`, `peekTok`, `getParentIncludeLoc`, and 3 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `addExplicitComment`, `Lex`, `peekTok`, `getParentIncludeLoc`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 1177-1198
```cpp
  return *tok;
}

const AsmToken MasmParser::peekTok(bool ShouldSkipSpace) {
  AsmToken Tok;

  MutableArrayRef<AsmToken> Buf(Tok);
  size_t ReadCount = Lexer.peekTokens(Buf, ShouldSkipSpace);

  if (ReadCount == 0) {
    // If this is the end of an included file, pop the parent file off the
    // include stack.
    SMLoc ParentIncludeLoc = SrcMgr.getParentIncludeLoc(CurBuffer);
    if (ParentIncludeLoc != SMLoc()) {
      EndStatementAtEOFStack.pop_back();
      jumpToLoc(ParentIncludeLoc, 0, EndStatementAtEOFStack.back());
      return peekTok(ShouldSkipSpace);
    }
    EndStatementAtEOFStack.pop_back();
    assert(EndStatementAtEOFStack.empty());
  }

```
- **EN**: Implements logic around `peekTok`, `Buf`, `peekTokens`, `getParentIncludeLoc`, and 3 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `peekTok`, `Buf`, `peekTokens`, `getParentIncludeLoc`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 1199-1221
```cpp
  assert(ReadCount == 1);
  return Tok;
}

bool MasmParser::Run(bool NoInitialTextSection, bool NoFinalize) {
  // Create the initial section, if requested.
  if (!NoInitialTextSection)
    Out.initSections(getTargetParser().getSTI());

  // Prime the lexer.
  Lex();

  HadError = false;
  AsmCond StartingCondState = TheCondState;
  SmallVector<AsmRewrite, 4> AsmStrRewrites;

  // While we have input, parse each statement.
  while (Lexer.isNot(AsmToken::Eof) ||
         SrcMgr.getParentIncludeLoc(CurBuffer) != SMLoc()) {
    // Skip through the EOF at the end of an inclusion.
    if (Lexer.is(AsmToken::Eof))
      Lex();

```
- **EN**: Implements logic around `assert`, `Run`, `initSections`, `Lex`, and 1 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `assert`, `Run`, `initSections`, `Lex`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1222-1243
```cpp
    ParseStatementInfo Info(&AsmStrRewrites);
    bool HasError = parseStatement(Info, nullptr);

    // If we have a Lexer Error we are on an Error Token. Load in Lexer Error
    // for printing ErrMsg via Lex() only if no (presumably better) parser error
    // exists.
    if (HasError && !hasPendingError() && Lexer.getTok().is(AsmToken::Error))
      Lex();

    // parseStatement returned true so may need to emit an error.
    printPendingErrors();

    // Skipping to the next line if needed.
    if (HasError && !getLexer().justConsumedEOL())
      eatToEndOfStatement();
  }

  printPendingErrors();

  // All errors should have been emitted.
  assert(!hasPendingError() && "unexpected error from parseStatement");

```
- **EN**: Implements logic around `Info`, `parseStatement`, `Lex`, `printPendingErrors`, and 2 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `Info`, `parseStatement`, `Lex`, `printPendingErrors`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为。

### Lines 1244-1264
```cpp
  if (TheCondState.TheCond != StartingCondState.TheCond ||
      TheCondState.Ignore != StartingCondState.Ignore)
    printError(getTok().getLoc(), "unmatched .ifs or .elses");

  // Check to see that all assembler local symbols were actually defined.
  // Targets that don't do subsections via symbols may not want this, though,
  // so conservatively exclude them. Only do this if we're finalizing, though,
  // as otherwise we won't necessarily have seen everything yet.
  if (!NoFinalize) {
    // Temporary symbols like the ones for directional jumps don't go in the
    // symbol table. They also need to be diagnosed in all (final) cases.
    for (std::tuple<SMLoc, CppHashInfoTy, MCSymbol *> &LocSym : DirLabels) {
      if (std::get<2>(LocSym)->isUndefined()) {
        // Reset the state of any "# line file" directives we've seen to the
        // context as it was at the diagnostic site.
        CppHashInfo = std::get<1>(LocSym);
        printError(std::get<0>(LocSym), "directional label undefined");
      }
    }
  }

```
- **EN**: Implements logic around `printError`, `get<1>`; this block parses assembly syntax or operands; updates MC section or symbol state.
- **CN**: 围绕 `printError`, `get<1>` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态。

### Lines 1265-1291
```cpp
  // Finalize the output stream if there are no errors and if the client wants
  // us to.
  if (!HadError && !NoFinalize)
    Out.finish(Lexer.getLoc());

  return HadError || getContext().hadError();
}

bool MasmParser::checkForValidSection() {
  if (!ParsingMSInlineAsm && !(getStreamer().getCurrentFragment() &&
                               getStreamer().getCurrentSectionOnly())) {
    Out.initSections(getTargetParser().getSTI());
    return Error(getTok().getLoc(),
                 "expected section directive before assembly directive");
  }
  return false;
}

/// Throw away the rest of the line for testing purposes.
void MasmParser::eatToEndOfStatement() {
  while (Lexer.isNot(AsmToken::EndOfStatement)) {
    if (Lexer.is(AsmToken::Eof)) {
      SMLoc ParentIncludeLoc = SrcMgr.getParentIncludeLoc(CurBuffer);
      if (ParentIncludeLoc == SMLoc()) {
        break;
      }

```
- **EN**: Implements logic around `finish`, `getContext`, `checkForValidSection`, `getStreamer`, and 4 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `finish`, `getContext`, `checkForValidSection`, `getStreamer`, and 4 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1292-1315
```cpp
      EndStatementAtEOFStack.pop_back();
      jumpToLoc(ParentIncludeLoc, 0, EndStatementAtEOFStack.back());
    }

    Lexer.Lex();
  }

  // Eat EOL.
  if (Lexer.is(AsmToken::EndOfStatement))
    Lexer.Lex();
}

SmallVector<StringRef, 1>
MasmParser::parseStringRefsTo(AsmToken::TokenKind EndTok) {
  SmallVector<StringRef, 1> Refs;
  const char *Start = getTok().getLoc().getPointer();
  while (Lexer.isNot(EndTok)) {
    if (Lexer.is(AsmToken::Eof)) {
      SMLoc ParentIncludeLoc = SrcMgr.getParentIncludeLoc(CurBuffer);
      if (ParentIncludeLoc == SMLoc()) {
        break;
      }
      Refs.emplace_back(Start, getTok().getLoc().getPointer() - Start);

```
- **EN**: Implements logic around `pop_back`, `jumpToLoc`, `Lex`, `parseStringRefsTo`, and 3 more symbols; this block parses assembly syntax or operands.
- **CN**: 围绕 `pop_back`, `jumpToLoc`, `Lex`, `parseStringRefsTo`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 1316-1336
```cpp
      EndStatementAtEOFStack.pop_back();
      jumpToLoc(ParentIncludeLoc, 0, EndStatementAtEOFStack.back());
      Lexer.Lex();
      Start = getTok().getLoc().getPointer();
    } else {
      Lexer.Lex();
    }
  }
  Refs.emplace_back(Start, getTok().getLoc().getPointer() - Start);
  return Refs;
}

std::string MasmParser::parseStringTo(AsmToken::TokenKind EndTok) {
  SmallVector<StringRef, 1> Refs = parseStringRefsTo(EndTok);
  std::string Str;
  for (StringRef S : Refs) {
    Str.append(S.str());
  }
  return Str;
}

```
- **EN**: Implements logic around `pop_back`, `jumpToLoc`, `Lex`, `getTok`, and 4 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `pop_back`, `jumpToLoc`, `Lex`, `getTok`, and 4 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 1337-1358
```cpp
StringRef MasmParser::parseStringToEndOfStatement() {
  const char *Start = getTok().getLoc().getPointer();

  while (Lexer.isNot(AsmToken::EndOfStatement) && Lexer.isNot(AsmToken::Eof))
    Lexer.Lex();

  const char *End = getTok().getLoc().getPointer();
  return StringRef(Start, End - Start);
}

/// Parse a paren expression and return it.
/// NOTE: This assumes the leading '(' has already been consumed.
///
/// parenexpr ::= expr)
///
bool MasmParser::parseParenExpr(const MCExpr *&Res, SMLoc &EndLoc) {
  if (parseExpression(Res))
    return true;
  EndLoc = Lexer.getTok().getEndLoc();
  return parseRParen();
}

```
- **EN**: Implements logic around `parseStringToEndOfStatement`, `getTok`, `Lex`, `StringRef`, and 2 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseStringToEndOfStatement`, `getTok`, `Lex`, `StringRef`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 1359-1378
```cpp
/// Parse a bracket expression and return it.
/// NOTE: This assumes the leading '[' has already been consumed.
///
/// bracketexpr ::= expr]
///
bool MasmParser::parseBracketExpr(const MCExpr *&Res, SMLoc &EndLoc) {
  if (parseExpression(Res))
    return true;
  EndLoc = getTok().getEndLoc();
  if (parseToken(AsmToken::RBrac, "expected ']' in brackets expression"))
    return true;
  return false;
}

/// Parse a primary expression and return it.
///  primaryexpr ::= (parenexpr
///  primaryexpr ::= symbol
///  primaryexpr ::= number
///  primaryexpr ::= '.'
///  primaryexpr ::= ~,+,-,'not' primaryexpr
```
- **EN**: Implements logic around `parseBracketExpr`, `getTok`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseBracketExpr`, `getTok` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1379-1418
```cpp
///  primaryexpr ::= string
///          (a string is interpreted as a 64-bit number in big-endian base-256)
bool MasmParser::parsePrimaryExpr(const MCExpr *&Res, SMLoc &EndLoc,
                                  AsmTypeInfo *TypeInfo) {
  SMLoc FirstTokenLoc = getLexer().getLoc();
  AsmToken::TokenKind FirstTokenKind = Lexer.getKind();
  switch (FirstTokenKind) {
  default:
    return TokError("unknown token in expression");
  // If we have an error assume that we've already handled it.
  case AsmToken::Error:
    return true;
  case AsmToken::Exclaim:
    Lex(); // Eat the operator.
    if (parsePrimaryExpr(Res, EndLoc, nullptr))
      return true;
    Res = MCUnaryExpr::createLNot(Res, getContext(), FirstTokenLoc);
    return false;
  case AsmToken::Dollar:
  case AsmToken::At:
  case AsmToken::Identifier: {
    StringRef Identifier;
    if (parseIdentifier(Identifier)) {
      // We may have failed but $ may be a valid token.
      if (getTok().is(AsmToken::Dollar)) {
        if (Lexer.getMAI().getDollarIsPC()) {
          Lex();
          // This is a '$' reference, which references the current PC.  Emit a
          // temporary label to the streamer and refer to it.
          MCSymbol *Sym = Ctx.createTempSymbol();
          Out.emitLabel(Sym);
          Res = MCSymbolRefExpr::create(Sym, getContext());
          EndLoc = FirstTokenLoc;
          return false;
        }
        return Error(FirstTokenLoc, "invalid token in expression");
      }
    }
    // Parse named bitwise negation.
    if (Identifier.equals_insensitive("not")) {
```
- **EN**: Implements logic around `parsePrimaryExpr`, `getLexer`, `getKind`, `TokError`, and 6 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `parsePrimaryExpr`, `getLexer`, `getKind`, `TokError`, and 6 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 1419-1441
```cpp
      if (parsePrimaryExpr(Res, EndLoc, nullptr))
        return true;
      Res = MCUnaryExpr::createNot(Res, getContext(), FirstTokenLoc);
      return false;
    }
    // Parse directional local label references.
    if (Identifier.equals_insensitive("@b") ||
        Identifier.equals_insensitive("@f")) {
      bool Before = Identifier.equals_insensitive("@b");
      MCSymbol *Sym = getContext().getDirectionalLocalSymbol(0, Before);
      if (Before && Sym->isUndefined())
        return Error(FirstTokenLoc, "Expected @@ label before @B reference");
      Res = MCSymbolRefExpr::create(Sym, getContext());
      return false;
    }

    EndLoc = SMLoc::getFromPointer(Identifier.end());

    // This is a symbol reference.
    StringRef SymbolName = Identifier;
    if (SymbolName.empty())
      return Error(getLexer().getLoc(), "expected a symbol reference");

```
- **EN**: Implements logic around `createNot`, `equals_insensitive`, `getContext`, `Error`, and 2 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `createNot`, `equals_insensitive`, `getContext`, `Error`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1442-1473
```cpp
    // Find the field offset if used.
    AsmFieldInfo Info;
    auto Split = SymbolName.split('.');
    if (Split.second.empty()) {
    } else {
      SymbolName = Split.first;
      if (lookUpField(SymbolName, Split.second, Info)) {
        std::pair<StringRef, StringRef> BaseMember = Split.second.split('.');
        StringRef Base = BaseMember.first, Member = BaseMember.second;
        lookUpField(Base, Member, Info);
      } else if (Structs.count(SymbolName.lower())) {
        // This is actually a reference to a field offset.
        Res = MCConstantExpr::create(Info.Offset, getContext());
        return false;
      }
    }

    MCSymbol *Sym = getContext().getInlineAsmLabel(SymbolName);
    if (!Sym) {
      // If this is a built-in numeric value, treat it as a constant.
      auto BuiltinIt = BuiltinSymbolMap.find(SymbolName.lower());
      const BuiltinSymbol Symbol = (BuiltinIt == BuiltinSymbolMap.end())
                                       ? BI_NO_SYMBOL
                                       : BuiltinIt->getValue();
      if (Symbol != BI_NO_SYMBOL) {
        const MCExpr *Value = evaluateBuiltinValue(Symbol, FirstTokenLoc);
        if (Value) {
          Res = Value;
          return false;
        }
      }

```
- **EN**: Implements logic around `split`, `lookUpField`, `create`, `getContext`, and 4 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `split`, `lookUpField`, `create`, `getContext`, and 4 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1474-1494
```cpp
      // Variables use case-insensitive symbol names; if this is a variable, we
      // find the symbol using its canonical name.
      auto VarIt = Variables.find(SymbolName.lower());
      if (VarIt != Variables.end())
        SymbolName = VarIt->second.Name;
      Sym = getContext().parseSymbol(SymbolName);
    }

    // If this is an absolute variable reference, substitute it now to preserve
    // semantics in the face of reassignment.
    if (Sym->isVariable()) {
      auto V = Sym->getVariableValue();
      bool DoInline = isa<MCConstantExpr>(V);
      if (auto TV = dyn_cast<MCTargetExpr>(V))
        DoInline = TV->inlineAssignedExpr();
      if (DoInline) {
        Res = Sym->getVariableValue();
        return false;
      }
    }

```
- **EN**: Implements logic around `find`, `getContext`, `getVariableValue`, `isa<MCConstantExpr>`, and 1 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `find`, `getContext`, `getVariableValue`, `isa<MCConstantExpr>`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1495-1534
```cpp
    // Otherwise create a symbol ref.
    const MCExpr *SymRef =
        MCSymbolRefExpr::create(Sym, getContext(), FirstTokenLoc);
    if (Info.Offset) {
      Res = MCBinaryExpr::create(
          MCBinaryExpr::Add, SymRef,
          MCConstantExpr::create(Info.Offset, getContext()), getContext());
    } else {
      Res = SymRef;
    }
    if (TypeInfo) {
      if (Info.Type.Name.empty()) {
        auto TypeIt = KnownType.find(Identifier.lower());
        if (TypeIt != KnownType.end()) {
          Info.Type = TypeIt->second;
        }
      }

      *TypeInfo = Info.Type;
    }
    return false;
  }
  case AsmToken::BigNum:
    return TokError("literal value out of range for directive");
  case AsmToken::Integer: {
    int64_t IntVal = getTok().getIntVal();
    Res = MCConstantExpr::create(IntVal, getContext());
    EndLoc = Lexer.getTok().getEndLoc();
    Lex(); // Eat token.
    return false;
  }
  case AsmToken::String: {
    // MASM strings (used as constants) are interpreted as big-endian base-256.
    SMLoc ValueLoc = getTok().getLoc();
    std::string Value;
    if (parseEscapedString(Value))
      return true;
    if (Value.size() > 8)
      return Error(ValueLoc, "literal value out of range");
    uint64_t IntValue = 0;
```
- **EN**: Implements logic around `create`, `find`, `TokError`, `getTok`, and 2 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `create`, `find`, `TokError`, `getTok`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1535-1574
```cpp
    for (const unsigned char CharVal : Value)
      IntValue = (IntValue << 8) | CharVal;
    Res = MCConstantExpr::create(IntValue, getContext());
    return false;
  }
  case AsmToken::Real: {
    APFloat RealVal(APFloat::IEEEdouble(), getTok().getString());
    uint64_t IntVal = RealVal.bitcastToAPInt().getZExtValue();
    Res = MCConstantExpr::create(IntVal, getContext());
    EndLoc = Lexer.getTok().getEndLoc();
    Lex(); // Eat token.
    return false;
  }
  case AsmToken::Dot: {
    // This is a '.' reference, which references the current PC.  Emit a
    // temporary label to the streamer and refer to it.
    MCSymbol *Sym = Ctx.createTempSymbol();
    Out.emitLabel(Sym);
    Res = MCSymbolRefExpr::create(Sym, getContext());
    EndLoc = Lexer.getTok().getEndLoc();
    Lex(); // Eat identifier.
    return false;
  }
  case AsmToken::LParen:
    Lex(); // Eat the '('.
    return parseParenExpr(Res, EndLoc);
  case AsmToken::LBrac:
    if (!PlatformParser->HasBracketExpressions())
      return TokError("brackets expression not supported on this target");
    Lex(); // Eat the '['.
    return parseBracketExpr(Res, EndLoc);
  case AsmToken::Minus:
    Lex(); // Eat the operator.
    if (parsePrimaryExpr(Res, EndLoc, nullptr))
      return true;
    Res = MCUnaryExpr::createMinus(Res, getContext(), FirstTokenLoc);
    return false;
  case AsmToken::Plus:
    Lex(); // Eat the operator.
    if (parsePrimaryExpr(Res, EndLoc, nullptr))
```
- **EN**: Implements logic around `create`, `RealVal`, `bitcastToAPInt`, `getTok`, and 7 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `create`, `RealVal`, `bitcastToAPInt`, `getTok`, and 7 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1575-1594
```cpp
      return true;
    Res = MCUnaryExpr::createPlus(Res, getContext(), FirstTokenLoc);
    return false;
  case AsmToken::Tilde:
    Lex(); // Eat the operator.
    if (parsePrimaryExpr(Res, EndLoc, nullptr))
      return true;
    Res = MCUnaryExpr::createNot(Res, getContext(), FirstTokenLoc);
    return false;
  }
}

bool MasmParser::parseExpression(const MCExpr *&Res) {
  SMLoc EndLoc;
  return parseExpression(Res, EndLoc);
}

/// This function checks if the next token is <string> type or arithmetic.
/// string that begin with character '<' must end with character '>'.
/// otherwise it is arithmetics.
```
- **EN**: Implements logic around `createPlus`, `Lex`, `createNot`, `parseExpression`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `createPlus`, `Lex`, `createNot`, `parseExpression` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 1595-1614
```cpp
/// If the function returns a 'true' value,
/// the End argument will be filled with the last location pointed to the '>'
/// character.
static bool isAngleBracketString(SMLoc &StrLoc, SMLoc &EndLoc) {
  assert((StrLoc.getPointer() != nullptr) &&
         "Argument to the function cannot be a NULL value");
  const char *CharPtr = StrLoc.getPointer();
  while ((*CharPtr != '>') && (*CharPtr != '\n') && (*CharPtr != '\r') &&
         (*CharPtr != '\0')) {
    if (*CharPtr == '!')
      CharPtr++;
    CharPtr++;
  }
  if (*CharPtr == '>') {
    EndLoc = StrLoc.getFromPointer(CharPtr + 1);
    return true;
  }
  return false;
}

```
- **EN**: Implements logic around `isAngleBracketString`, `assert`, `getPointer`, `getFromPointer`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `isAngleBracketString`, `assert`, `getPointer`, `getFromPointer` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 1615-1634
```cpp
/// creating a string without the escape characters '!'.
static std::string angleBracketString(StringRef BracketContents) {
  std::string Res;
  for (size_t Pos = 0; Pos < BracketContents.size(); Pos++) {
    if (BracketContents[Pos] == '!')
      Pos++;
    Res += BracketContents[Pos];
  }
  return Res;
}

/// Parse an expression and return it.
///
///  expr ::= expr &&,|| expr               -> lowest.
///  expr ::= expr |,^,&,! expr
///  expr ::= expr ==,!=,<>,<,<=,>,>= expr
///  expr ::= expr <<,>> expr
///  expr ::= expr +,- expr
///  expr ::= expr *,/,% expr               -> highest.
///  expr ::= primaryexpr
```
- **EN**: Implements logic around `angleBracketString`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `angleBracketString` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 1635-1656
```cpp
///
bool MasmParser::parseExpression(const MCExpr *&Res, SMLoc &EndLoc) {
  // Parse the expression.
  Res = nullptr;
  if (getTargetParser().parsePrimaryExpr(Res, EndLoc) ||
      parseBinOpRHS(1, Res, EndLoc))
    return true;

  // Try to constant fold it up front, if possible. Do not exploit
  // assembler here.
  int64_t Value;
  if (Res->evaluateAsAbsolute(Value))
    Res = MCConstantExpr::create(Value, getContext());

  return false;
}

bool MasmParser::parseParenExpression(const MCExpr *&Res, SMLoc &EndLoc) {
  Res = nullptr;
  return parseParenExpr(Res, EndLoc) || parseBinOpRHS(1, Res, EndLoc);
}

```
- **EN**: Implements logic around `parseExpression`, `parseBinOpRHS`, `create`, `parseParenExpression`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseExpression`, `parseBinOpRHS`, `create`, `parseParenExpression`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 1657-1677
```cpp
bool MasmParser::parseAbsoluteExpression(int64_t &Res) {
  const MCExpr *Expr;

  SMLoc StartLoc = Lexer.getLoc();
  if (parseExpression(Expr))
    return true;

  if (!Expr->evaluateAsAbsolute(Res, getStreamer().getAssemblerPtr()))
    return Error(StartLoc, "expected absolute expression");

  return false;
}

static unsigned getGNUBinOpPrecedence(AsmToken::TokenKind K,
                                      MCBinaryExpr::Opcode &Kind,
                                      bool ShouldUseLogicalShr,
                                      bool EndExpressionAtGreater) {
  switch (K) {
  default:
    return 0; // not a binop.

```
- **EN**: Implements logic around `parseAbsoluteExpression`, `getLoc`, `Error`, `getGNUBinOpPrecedence`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `parseAbsoluteExpression`, `getLoc`, `Error`, `getGNUBinOpPrecedence` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 1678-1708
```cpp
  // Lowest Precedence: &&, ||
  case AsmToken::AmpAmp:
    Kind = MCBinaryExpr::LAnd;
    return 2;
  case AsmToken::PipePipe:
    Kind = MCBinaryExpr::LOr;
    return 1;

  // Low Precedence: ==, !=, <>, <, <=, >, >=
  case AsmToken::EqualEqual:
    Kind = MCBinaryExpr::EQ;
    return 3;
  case AsmToken::ExclaimEqual:
  case AsmToken::LessGreater:
    Kind = MCBinaryExpr::NE;
    return 3;
  case AsmToken::Less:
    Kind = MCBinaryExpr::LT;
    return 3;
  case AsmToken::LessEqual:
    Kind = MCBinaryExpr::LTE;
    return 3;
  case AsmToken::Greater:
    if (EndExpressionAtGreater)
      return 0;
    Kind = MCBinaryExpr::GT;
    return 3;
  case AsmToken::GreaterEqual:
    Kind = MCBinaryExpr::GTE;
    return 3;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 1709-1748
```cpp
  // Low Intermediate Precedence: +, -
  case AsmToken::Plus:
    Kind = MCBinaryExpr::Add;
    return 4;
  case AsmToken::Minus:
    Kind = MCBinaryExpr::Sub;
    return 4;

  // High Intermediate Precedence: |, &, ^
  case AsmToken::Pipe:
    Kind = MCBinaryExpr::Or;
    return 5;
  case AsmToken::Caret:
    Kind = MCBinaryExpr::Xor;
    return 5;
  case AsmToken::Amp:
    Kind = MCBinaryExpr::And;
    return 5;

  // Highest Precedence: *, /, %, <<, >>
  case AsmToken::Star:
    Kind = MCBinaryExpr::Mul;
    return 6;
  case AsmToken::Slash:
    Kind = MCBinaryExpr::Div;
    return 6;
  case AsmToken::Percent:
    Kind = MCBinaryExpr::Mod;
    return 6;
  case AsmToken::LessLess:
    Kind = MCBinaryExpr::Shl;
    return 6;
  case AsmToken::GreaterGreater:
    if (EndExpressionAtGreater)
      return 0;
    Kind = ShouldUseLogicalShr ? MCBinaryExpr::LShr : MCBinaryExpr::AShr;
    return 6;
  }
}

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 1749-1781
```cpp
unsigned MasmParser::getBinOpPrecedence(AsmToken::TokenKind K,
                                        MCBinaryExpr::Opcode &Kind) {
  bool ShouldUseLogicalShr = MAI.shouldUseLogicalShr();
  return getGNUBinOpPrecedence(K, Kind, ShouldUseLogicalShr,
                               AngleBracketDepth > 0);
}

/// Parse all binary operators with precedence >= 'Precedence'.
/// Res contains the LHS of the expression on input.
bool MasmParser::parseBinOpRHS(unsigned Precedence, const MCExpr *&Res,
                               SMLoc &EndLoc) {
  SMLoc StartLoc = Lexer.getLoc();
  while (true) {
    AsmToken::TokenKind TokKind = Lexer.getKind();
    if (Lexer.getKind() == AsmToken::Identifier) {
      TokKind = StringSwitch<AsmToken::TokenKind>(Lexer.getTok().getString())
                    .CaseLower("and", AsmToken::Amp)
                    .CaseLower("not", AsmToken::Exclaim)
                    .CaseLower("or", AsmToken::Pipe)
                    .CaseLower("xor", AsmToken::Caret)
                    .CaseLower("shl", AsmToken::LessLess)
                    .CaseLower("shr", AsmToken::GreaterGreater)
                    .CaseLower("eq", AsmToken::EqualEqual)
                    .CaseLower("ne", AsmToken::ExclaimEqual)
                    .CaseLower("lt", AsmToken::Less)
                    .CaseLower("le", AsmToken::LessEqual)
                    .CaseLower("gt", AsmToken::Greater)
                    .CaseLower("ge", AsmToken::GreaterEqual)
                    .Default(TokKind);
    }
    MCBinaryExpr::Opcode Kind = MCBinaryExpr::Add;
    unsigned TokPrec = getBinOpPrecedence(TokKind, Kind);

```
- **EN**: Implements logic around `getBinOpPrecedence`, `shouldUseLogicalShr`, `getGNUBinOpPrecedence`, `parseBinOpRHS`, and 5 more symbols; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getBinOpPrecedence`, `shouldUseLogicalShr`, `getGNUBinOpPrecedence`, `parseBinOpRHS`, and 5 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 1782-1805
```cpp
    // If the next token is lower precedence than we are allowed to eat, return
    // successfully with what we ate already.
    if (TokPrec < Precedence)
      return false;

    Lex();

    // Eat the next primary expression.
    const MCExpr *RHS;
    if (getTargetParser().parsePrimaryExpr(RHS, EndLoc))
      return true;

    // If BinOp binds less tightly with RHS than the operator after RHS, let
    // the pending operator take RHS as its LHS.
    MCBinaryExpr::Opcode Dummy;
    unsigned NextTokPrec = getBinOpPrecedence(Lexer.getKind(), Dummy);
    if (TokPrec < NextTokPrec && parseBinOpRHS(TokPrec + 1, RHS, EndLoc))
      return true;

    // Merge LHS and RHS according to operator.
    Res = MCBinaryExpr::create(Kind, Res, RHS, getContext(), StartLoc);
  }
}

```
- **EN**: Implements logic around `Lex`, `getBinOpPrecedence`, `create`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `Lex`, `getBinOpPrecedence`, `create` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 1806-1825
```cpp
/// ParseStatement:
///   ::= % statement
///   ::= EndOfStatement
///   ::= Label* Directive ...Operands... EndOfStatement
///   ::= Label* Identifier OperandList* EndOfStatement
bool MasmParser::parseStatement(ParseStatementInfo &Info,
                                MCAsmParserSemaCallback *SI) {
  assert(!hasPendingError() && "parseStatement started with pending error");
  // Eat initial spaces and comments.
  while (Lexer.is(AsmToken::Space))
    Lex();
  if (Lexer.is(AsmToken::EndOfStatement)) {
    // If this is a line comment we can drop it safely.
    if (getTok().getString().empty() || getTok().getString().front() == '\r' ||
        getTok().getString().front() == '\n')
      Out.addBlankLine();
    Lex();
    return false;
  }

```
- **EN**: Implements logic around `parseStatement`, `assert`, `Lex`, `getTok`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseStatement`, `assert`, `Lex`, `getTok`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 1826-1858
```cpp
  // If preceded by an expansion operator, first expand all text macros and
  // macro functions.
  if (getTok().is(AsmToken::Percent)) {
    SMLoc ExpansionLoc = getTok().getLoc();
    if (parseToken(AsmToken::Percent) || expandStatement(ExpansionLoc))
      return true;
  }

  // Statements always start with an identifier, unless we're dealing with a
  // processor directive (.386, .686, etc.) that lexes as a real.
  AsmToken ID = getTok();
  SMLoc IDLoc = ID.getLoc();
  StringRef IDVal;
  if (Lexer.is(AsmToken::HashDirective))
    return parseCppHashLineFilenameComment(IDLoc);
  if (Lexer.is(AsmToken::Dot)) {
    // Treat '.' as a valid identifier in this context.
    Lex();
    IDVal = ".";
  } else if (Lexer.is(AsmToken::Real)) {
    // Treat ".<number>" as a valid identifier in this context.
    IDVal = getTok().getString();
    Lex(); // always eat a token
    if (!IDVal.starts_with("."))
      return Error(IDLoc, "unexpected token at start of statement");
  } else if (parseIdentifier(IDVal, StartOfStatement)) {
    if (!TheCondState.Ignore) {
      Lex(); // always eat a token
      return Error(IDLoc, "unexpected token at start of statement");
    }
    IDVal = "";
  }

```
- **EN**: Implements logic around `getTok`, `getLoc`, `parseCppHashLineFilenameComment`, `Lex`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `getTok`, `getLoc`, `parseCppHashLineFilenameComment`, `Lex`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 1859-1883
```cpp
  // Handle conditional assembly here before checking for skipping.  We
  // have to do this so that .endif isn't skipped in a ".if 0" block for
  // example.
  StringMap<DirectiveKind>::const_iterator DirKindIt =
      DirectiveKindMap.find(IDVal.lower());
  DirectiveKind DirKind = (DirKindIt == DirectiveKindMap.end())
                              ? DK_NO_DIRECTIVE
                              : DirKindIt->getValue();
  switch (DirKind) {
  default:
    break;
  case DK_IF:
  case DK_IFE:
    return parseDirectiveIf(IDLoc, DirKind);
  case DK_IFB:
    return parseDirectiveIfb(IDLoc, true);
  case DK_IFNB:
    return parseDirectiveIfb(IDLoc, false);
  case DK_IFDEF:
    return parseDirectiveIfdef(IDLoc, true);
  case DK_IFNDEF:
    return parseDirectiveIfdef(IDLoc, false);
  case DK_IFDIF:
    return parseDirectiveIfidn(IDLoc, /*ExpectEqual=*/false,
                               /*CaseInsensitive=*/false);
```
- **EN**: Implements logic around `find`, `end`, `getValue`, `parseDirectiveIf`, and 3 more symbols; this block manipulates JITLink graph structures or link-time passes; parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `find`, `end`, `getValue`, `parseDirectiveIf`, and 3 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 1884-1906
```cpp
  case DK_IFDIFI:
    return parseDirectiveIfidn(IDLoc, /*ExpectEqual=*/false,
                               /*CaseInsensitive=*/true);
  case DK_IFIDN:
    return parseDirectiveIfidn(IDLoc, /*ExpectEqual=*/true,
                               /*CaseInsensitive=*/false);
  case DK_IFIDNI:
    return parseDirectiveIfidn(IDLoc, /*ExpectEqual=*/true,
                               /*CaseInsensitive=*/true);
  case DK_ELSEIF:
  case DK_ELSEIFE:
    return parseDirectiveElseIf(IDLoc, DirKind);
  case DK_ELSEIFB:
    return parseDirectiveElseIfb(IDLoc, true);
  case DK_ELSEIFNB:
    return parseDirectiveElseIfb(IDLoc, false);
  case DK_ELSEIFDEF:
    return parseDirectiveElseIfdef(IDLoc, true);
  case DK_ELSEIFNDEF:
    return parseDirectiveElseIfdef(IDLoc, false);
  case DK_ELSEIFDIF:
    return parseDirectiveElseIfidn(IDLoc, /*ExpectEqual=*/false,
                                   /*CaseInsensitive=*/false);
```
- **EN**: Implements logic around `parseDirectiveIfidn`, `parseDirectiveElseIf`, `parseDirectiveElseIfb`, `parseDirectiveElseIfdef`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveIfidn`, `parseDirectiveElseIf`, `parseDirectiveElseIfb`, `parseDirectiveElseIfdef`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 1907-1928
```cpp
  case DK_ELSEIFDIFI:
    return parseDirectiveElseIfidn(IDLoc, /*ExpectEqual=*/false,
                                   /*CaseInsensitive=*/true);
  case DK_ELSEIFIDN:
    return parseDirectiveElseIfidn(IDLoc, /*ExpectEqual=*/true,
                                   /*CaseInsensitive=*/false);
  case DK_ELSEIFIDNI:
    return parseDirectiveElseIfidn(IDLoc, /*ExpectEqual=*/true,
                                   /*CaseInsensitive=*/true);
  case DK_ELSE:
    return parseDirectiveElse(IDLoc);
  case DK_ENDIF:
    return parseDirectiveEndIf(IDLoc);
  }

  // Ignore the statement if in the middle of inactive conditional
  // (e.g. ".if 0").
  if (TheCondState.Ignore) {
    eatToEndOfStatement();
    return false;
  }

```
- **EN**: Implements logic around `parseDirectiveElseIfidn`, `parseDirectiveElse`, `parseDirectiveEndIf`, `eatToEndOfStatement`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveElseIfidn`, `parseDirectiveElse`, `parseDirectiveEndIf`, `eatToEndOfStatement` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 1929-1966
```cpp
  // FIXME: Recurse on local labels?

  // Check for a label.
  //   ::= identifier ':'
  //   ::= number ':'
  if (Lexer.is(AsmToken::Colon) && getTargetParser().isLabel(ID)) {
    if (checkForValidSection())
      return true;

    // identifier ':'   -> Label.
    Lex();

    // Diagnose attempt to use '.' as a label.
    if (IDVal == ".")
      return Error(IDLoc, "invalid use of pseudo-symbol '.' as a label");

    // Diagnose attempt to use a variable as a label.
    //
    // FIXME: Diagnostics. Note the location of the definition as a label.
    // FIXME: This doesn't diagnose assignment to a symbol which has been
    // implicitly marked as external.
    MCSymbol *Sym;
    if (ParsingMSInlineAsm && SI) {
      StringRef RewrittenLabel =
          SI->LookupInlineAsmLabel(IDVal, getSourceManager(), IDLoc, true);
      assert(!RewrittenLabel.empty() &&
             "We should have an internal name here.");
      Info.AsmRewrites->emplace_back(AOK_Label, IDLoc, IDVal.size(),
                                     RewrittenLabel);
      IDVal = RewrittenLabel;
    }
    // Handle directional local labels
    if (IDVal == "@@") {
      Sym = Ctx.createDirectionalLocalSymbol(0);
    } else {
      Sym = getContext().parseSymbol(IDVal);
    }

```
- **EN**: Implements logic around `Lex`, `Error`, `LookupInlineAsmLabel`, `assert`, and 3 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `Lex`, `Error`, `LookupInlineAsmLabel`, `assert`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1967-1988
```cpp
    // End of Labels should be treated as end of line for lexing
    // purposes but that information is not available to the Lexer who
    // does not understand Labels. This may cause us to see a Hash
    // here instead of a preprocessor line comment.
    if (getTok().is(AsmToken::Hash)) {
      std::string CommentStr = parseStringTo(AsmToken::EndOfStatement);
      Lexer.Lex();
      Lexer.UnLex(AsmToken(AsmToken::EndOfStatement, CommentStr));
    }

    // Consume any end of statement token, if present, to avoid spurious
    // addBlankLine calls().
    if (getTok().is(AsmToken::EndOfStatement)) {
      Lex();
    }

    // Emit the label.
    if (!getTargetParser().isParsingMSInlineAsm())
      Out.emitLabel(Sym, IDLoc);
    return false;
  }

```
- **EN**: Implements logic around `parseStringTo`, `Lex`, `UnLex`, `emitLabel`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseStringTo`, `Lex`, `UnLex`, `emitLabel` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 1989-2010
```cpp
  // If macros are enabled, check to see if this is a macro instantiation.
  if (const MCAsmMacro *M = getContext().lookupMacro(IDVal.lower())) {
    AsmToken::TokenKind ArgumentEndTok = parseOptionalToken(AsmToken::LParen)
                                             ? AsmToken::RParen
                                             : AsmToken::EndOfStatement;
    return handleMacroEntry(M, IDLoc, ArgumentEndTok);
  }

  // Otherwise, we have a normal instruction or directive.

  if (DirKind != DK_NO_DIRECTIVE) {
    // There are several entities interested in parsing directives:
    //
    // 1. Asm parser extensions. For example, platform-specific parsers
    //    (like the ELF parser) register themselves as extensions.
    // 2. The target-specific assembly parser. Some directives are target
    //    specific or may potentially behave differently on certain targets.
    // 3. The generic directive parser implemented by this class. These are
    //    all the directives that behave in a target and platform independent
    //    manner, or at least have a default behavior that's shared between
    //    all targets and platforms.

```
- **EN**: Implements logic around `parseOptionalToken`, `handleMacroEntry`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseOptionalToken`, `handleMacroEntry` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 2011-2036
```cpp
    // Special-case handling of structure-end directives at higher priority,
    // since ENDS is overloaded as a segment-end directive.
    if (IDVal.equals_insensitive("ends") && StructInProgress.size() > 1 &&
        getTok().is(AsmToken::EndOfStatement)) {
      return parseDirectiveNestedEnds();
    }

    // First, check the extension directive map to see if any extension has
    // registered itself to parse this directive.
    std::pair<MCAsmParserExtension *, DirectiveHandler> Handler =
        ExtensionDirectiveMap.lookup(IDVal.lower());
    if (Handler.first)
      return (*Handler.second)(Handler.first, IDVal, IDLoc);

    // Next, let the target-specific assembly parser try.
    if (ID.isNot(AsmToken::Identifier))
      return false;

    ParseStatus TPDirectiveReturn = getTargetParser().parseDirective(ID);
    assert(TPDirectiveReturn.isFailure() == hasPendingError() &&
           "Should only return Failure iff there was an error");
    if (TPDirectiveReturn.isFailure())
      return true;
    if (TPDirectiveReturn.isSuccess())
      return false;

```
- **EN**: Implements logic around `getTok`, `parseDirectiveNestedEnds`, `lookup`, `getTargetParser`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `getTok`, `parseDirectiveNestedEnds`, `lookup`, `getTargetParser`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 2037-2076
```cpp
    // Finally, if no one else is interested in this directive, it must be
    // generic and familiar to this class.
    switch (DirKind) {
    default:
      break;
    case DK_ASCII:
      return parseDirectiveAscii(IDVal, false);
    case DK_ASCIZ:
    case DK_STRING:
      return parseDirectiveAscii(IDVal, true);
    case DK_BYTE:
    case DK_SBYTE:
    case DK_DB:
      return parseDirectiveValue(IDVal, 1);
    case DK_WORD:
    case DK_SWORD:
    case DK_DW:
      return parseDirectiveValue(IDVal, 2);
    case DK_DWORD:
    case DK_SDWORD:
    case DK_DD:
      return parseDirectiveValue(IDVal, 4);
    case DK_FWORD:
    case DK_DF:
      return parseDirectiveValue(IDVal, 6);
    case DK_QWORD:
    case DK_SQWORD:
    case DK_DQ:
      return parseDirectiveValue(IDVal, 8);
    case DK_REAL4:
      return parseDirectiveRealValue(IDVal, APFloat::IEEEsingle(), 4);
    case DK_REAL8:
      return parseDirectiveRealValue(IDVal, APFloat::IEEEdouble(), 8);
    case DK_REAL10:
      return parseDirectiveRealValue(IDVal, APFloat::x87DoubleExtended(), 10);
    case DK_STRUCT:
    case DK_UNION:
      return parseDirectiveNestedStruct(IDVal, DirKind);
    case DK_ENDS:
      return parseDirectiveNestedEnds();
```
- **EN**: Implements logic around `parseDirectiveAscii`, `parseDirectiveValue`, `parseDirectiveRealValue`, `parseDirectiveNestedStruct`, and 1 more symbols; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveAscii`, `parseDirectiveValue`, `parseDirectiveRealValue`, `parseDirectiveNestedStruct`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 2077-2116
```cpp
    case DK_ALIGN:
      return parseDirectiveAlign();
    case DK_EVEN:
      return parseDirectiveEven();
    case DK_ORG:
      return parseDirectiveOrg();
    case DK_EXTERN:
      return parseDirectiveExtern();
    case DK_PUBLIC:
      return parseDirectiveSymbolAttribute(MCSA_Global);
    case DK_COMM:
      return parseDirectiveComm(/*IsLocal=*/false);
    case DK_COMMENT:
      return parseDirectiveComment(IDLoc);
    case DK_INCLUDE:
      return parseDirectiveInclude();
    case DK_REPEAT:
      return parseDirectiveRepeat(IDLoc, IDVal);
    case DK_WHILE:
      return parseDirectiveWhile(IDLoc);
    case DK_FOR:
      return parseDirectiveFor(IDLoc, IDVal);
    case DK_FORC:
      return parseDirectiveForc(IDLoc, IDVal);
    case DK_EXITM:
      Info.ExitValue = "";
      return parseDirectiveExitMacro(IDLoc, IDVal, *Info.ExitValue);
    case DK_ENDM:
      Info.ExitValue = "";
      return parseDirectiveEndMacro(IDVal);
    case DK_PURGE:
      return parseDirectivePurgeMacro(IDLoc);
    case DK_END:
      return parseDirectiveEnd(IDLoc);
    case DK_ERR:
      return parseDirectiveError(IDLoc);
    case DK_ERRB:
      return parseDirectiveErrorIfb(IDLoc, true);
    case DK_ERRNB:
      return parseDirectiveErrorIfb(IDLoc, false);
```
- **EN**: Implements logic around `parseDirectiveAlign`, `parseDirectiveEven`, `parseDirectiveOrg`, `parseDirectiveExtern`, and 14 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveAlign`, `parseDirectiveEven`, `parseDirectiveOrg`, `parseDirectiveExtern`, and 14 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 2117-2142
```cpp
    case DK_ERRDEF:
      return parseDirectiveErrorIfdef(IDLoc, true);
    case DK_ERRNDEF:
      return parseDirectiveErrorIfdef(IDLoc, false);
    case DK_ERRDIF:
      return parseDirectiveErrorIfidn(IDLoc, /*ExpectEqual=*/false,
                                      /*CaseInsensitive=*/false);
    case DK_ERRDIFI:
      return parseDirectiveErrorIfidn(IDLoc, /*ExpectEqual=*/false,
                                      /*CaseInsensitive=*/true);
    case DK_ERRIDN:
      return parseDirectiveErrorIfidn(IDLoc, /*ExpectEqual=*/true,
                                      /*CaseInsensitive=*/false);
    case DK_ERRIDNI:
      return parseDirectiveErrorIfidn(IDLoc, /*ExpectEqual=*/true,
                                      /*CaseInsensitive=*/true);
    case DK_ERRE:
      return parseDirectiveErrorIfe(IDLoc, true);
    case DK_ERRNZ:
      return parseDirectiveErrorIfe(IDLoc, false);
    case DK_RADIX:
      return parseDirectiveRadix(IDLoc);
    case DK_ECHO:
      return parseDirectiveEcho(IDLoc);
    }

```
- **EN**: Implements logic around `parseDirectiveErrorIfdef`, `parseDirectiveErrorIfidn`, `parseDirectiveErrorIfe`, `parseDirectiveRadix`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveErrorIfdef`, `parseDirectiveErrorIfidn`, `parseDirectiveErrorIfe`, `parseDirectiveRadix`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 2143-2167
```cpp
    return Error(IDLoc, "unknown directive");
  }

  // We also check if this is allocating memory with user-defined type.
  auto IDIt = Structs.find(IDVal.lower());
  if (IDIt != Structs.end())
    return parseDirectiveStructValue(/*Structure=*/IDIt->getValue(), IDVal,
                                     IDLoc);

  // Non-conditional Microsoft directives sometimes follow their first argument.
  const AsmToken nextTok = getTok();
  const StringRef nextVal = nextTok.getString();
  const SMLoc nextLoc = nextTok.getLoc();

  const AsmToken afterNextTok = peekTok();

  // There are several entities interested in parsing infix directives:
  //
  // 1. Asm parser extensions. For example, platform-specific parsers
  //    (like the ELF parser) register themselves as extensions.
  // 2. The generic directive parser implemented by this class. These are
  //    all the directives that behave in a target and platform independent
  //    manner, or at least have a default behavior that's shared between
  //    all targets and platforms.

```
- **EN**: Implements logic around `Error`, `find`, `parseDirectiveStructValue`, `getTok`, and 3 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `Error`, `find`, `parseDirectiveStructValue`, `getTok`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 2168-2207
```cpp
  getTargetParser().flushPendingInstructions(getStreamer());

  // Special-case handling of structure-end directives at higher priority, since
  // ENDS is overloaded as a segment-end directive.
  if (nextVal.equals_insensitive("ends") && StructInProgress.size() == 1) {
    Lex();
    return parseDirectiveEnds(IDVal, IDLoc);
  }

  // First, check the extension directive map to see if any extension has
  // registered itself to parse this directive.
  std::pair<MCAsmParserExtension *, DirectiveHandler> Handler =
      ExtensionDirectiveMap.lookup(nextVal.lower());
  if (Handler.first) {
    Lex();
    Lexer.UnLex(ID);
    return (*Handler.second)(Handler.first, nextVal, nextLoc);
  }

  // If no one else is interested in this directive, it must be
  // generic and familiar to this class.
  DirKindIt = DirectiveKindMap.find(nextVal.lower());
  DirKind = (DirKindIt == DirectiveKindMap.end())
                ? DK_NO_DIRECTIVE
                : DirKindIt->getValue();
  switch (DirKind) {
  default:
    break;
  case DK_ASSIGN:
  case DK_EQU:
  case DK_TEXTEQU:
    Lex();
    return parseDirectiveEquate(nextVal, IDVal, DirKind, IDLoc);
  case DK_BYTE:
    if (afterNextTok.is(AsmToken::Identifier) &&
        afterNextTok.getString().equals_insensitive("ptr")) {
      // Size directive; part of an instruction.
      break;
    }
    [[fallthrough]];
```
- **EN**: Implements logic around `getTargetParser`, `Lex`, `parseDirectiveEnds`, `lookup`, and 6 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getTargetParser`, `Lex`, `parseDirectiveEnds`, `lookup`, and 6 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 2208-2247
```cpp
  case DK_SBYTE:
  case DK_DB:
    Lex();
    return parseDirectiveNamedValue(nextVal, 1, IDVal, IDLoc);
  case DK_WORD:
    if (afterNextTok.is(AsmToken::Identifier) &&
        afterNextTok.getString().equals_insensitive("ptr")) {
      // Size directive; part of an instruction.
      break;
    }
    [[fallthrough]];
  case DK_SWORD:
  case DK_DW:
    Lex();
    return parseDirectiveNamedValue(nextVal, 2, IDVal, IDLoc);
  case DK_DWORD:
    if (afterNextTok.is(AsmToken::Identifier) &&
        afterNextTok.getString().equals_insensitive("ptr")) {
      // Size directive; part of an instruction.
      break;
    }
    [[fallthrough]];
  case DK_SDWORD:
  case DK_DD:
    Lex();
    return parseDirectiveNamedValue(nextVal, 4, IDVal, IDLoc);
  case DK_FWORD:
    if (afterNextTok.is(AsmToken::Identifier) &&
        afterNextTok.getString().equals_insensitive("ptr")) {
      // Size directive; part of an instruction.
      break;
    }
    [[fallthrough]];
  case DK_DF:
    Lex();
    return parseDirectiveNamedValue(nextVal, 6, IDVal, IDLoc);
  case DK_QWORD:
    if (afterNextTok.is(AsmToken::Identifier) &&
        afterNextTok.getString().equals_insensitive("ptr")) {
      // Size directive; part of an instruction.
```
- **EN**: Implements logic around `Lex`, `parseDirectiveNamedValue`, `getString`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `Lex`, `parseDirectiveNamedValue`, `getString` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 2248-2278
```cpp
      break;
    }
    [[fallthrough]];
  case DK_SQWORD:
  case DK_DQ:
    Lex();
    return parseDirectiveNamedValue(nextVal, 8, IDVal, IDLoc);
  case DK_REAL4:
    Lex();
    return parseDirectiveNamedRealValue(nextVal, APFloat::IEEEsingle(), 4,
                                        IDVal, IDLoc);
  case DK_REAL8:
    Lex();
    return parseDirectiveNamedRealValue(nextVal, APFloat::IEEEdouble(), 8,
                                        IDVal, IDLoc);
  case DK_REAL10:
    Lex();
    return parseDirectiveNamedRealValue(nextVal, APFloat::x87DoubleExtended(),
                                        10, IDVal, IDLoc);
  case DK_STRUCT:
  case DK_UNION:
    Lex();
    return parseDirectiveStruct(nextVal, DirKind, IDVal, IDLoc);
  case DK_ENDS:
    Lex();
    return parseDirectiveEnds(IDVal, IDLoc);
  case DK_MACRO:
    Lex();
    return parseDirectiveMacro(IDVal, IDLoc);
  }

```
- **EN**: Implements logic around `Lex`, `parseDirectiveNamedValue`, `parseDirectiveNamedRealValue`, `parseDirectiveStruct`, and 2 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `Lex`, `parseDirectiveNamedValue`, `parseDirectiveNamedRealValue`, `parseDirectiveStruct`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 2279-2300
```cpp
  // Finally, we check if this is allocating a variable with user-defined type.
  auto NextIt = Structs.find(nextVal.lower());
  if (NextIt != Structs.end()) {
    Lex();
    return parseDirectiveNamedStructValue(/*Structure=*/NextIt->getValue(),
                                          nextVal, nextLoc, IDVal);
  }

  // __asm _emit or __asm __emit
  if (ParsingMSInlineAsm && (IDVal == "_emit" || IDVal == "__emit" ||
                             IDVal == "_EMIT" || IDVal == "__EMIT"))
    return parseDirectiveMSEmit(IDLoc, Info, IDVal.size());

  // __asm align
  if (ParsingMSInlineAsm && (IDVal == "align" || IDVal == "ALIGN"))
    return parseDirectiveMSAlign(IDLoc, Info);

  if (ParsingMSInlineAsm && (IDVal == "even" || IDVal == "EVEN"))
    Info.AsmRewrites->emplace_back(AOK_EVEN, IDLoc, 4);
  if (checkForValidSection())
    return true;

```
- **EN**: Implements logic around `find`, `Lex`, `parseDirectiveNamedStructValue`, `parseDirectiveMSEmit`, and 2 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `find`, `Lex`, `parseDirectiveNamedStructValue`, `parseDirectiveMSEmit`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 2301-2322
```cpp
  // Canonicalize the opcode to lower case.
  std::string OpcodeStr = IDVal.lower();
  ParseInstructionInfo IInfo(Info.AsmRewrites);
  bool ParseHadError = getTargetParser().parseInstruction(IInfo, OpcodeStr, ID,
                                                          Info.ParsedOperands);
  Info.ParseError = ParseHadError;

  // Dump the parsed representation, if requested.
  if (getShowParsedOperands()) {
    SmallString<256> Str;
    raw_svector_ostream OS(Str);
    OS << "parsed instruction: [";
    for (unsigned i = 0; i != Info.ParsedOperands.size(); ++i) {
      if (i != 0)
        OS << ", ";
      Info.ParsedOperands[i]->print(OS, MAI);
    }
    OS << "]";

    printMessage(IDLoc, SourceMgr::DK_Note, OS.str());
  }

```
- **EN**: Implements logic around `lower`, `IInfo`, `getTargetParser`, `OS`, and 2 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `lower`, `IInfo`, `getTargetParser`, `OS`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为。

### Lines 2323-2344
```cpp
  // Fail even if ParseInstruction erroneously returns false.
  if (hasPendingError() || ParseHadError)
    return true;

  // If parsing succeeded, match the instruction.
  if (!ParseHadError) {
    uint64_t ErrorInfo;
    if (getTargetParser().matchAndEmitInstruction(
            IDLoc, Info.Opcode, Info.ParsedOperands, Out, ErrorInfo,
            getTargetParser().isParsingMSInlineAsm()))
      return true;
  }
  return false;
}

// Parse and erase curly braces marking block start/end.
bool MasmParser::parseCurlyBlockScope(
    SmallVectorImpl<AsmRewrite> &AsmStrRewrites) {
  // Identify curly brace marking block start/end.
  if (Lexer.isNot(AsmToken::LCurly) && Lexer.isNot(AsmToken::RCurly))
    return false;

```
- **EN**: Implements logic around `getTargetParser`, `parseCurlyBlockScope`; this block manipulates JITLink graph structures or link-time passes; parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `getTargetParser`, `parseCurlyBlockScope` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 2345-2370
```cpp
  SMLoc StartLoc = Lexer.getLoc();
  Lex(); // Eat the brace.
  if (Lexer.is(AsmToken::EndOfStatement))
    Lex(); // Eat EndOfStatement following the brace.

  // Erase the block start/end brace from the output asm string.
  AsmStrRewrites.emplace_back(AOK_Skip, StartLoc, Lexer.getLoc().getPointer() -
                                                  StartLoc.getPointer());
  return true;
}

/// parseCppHashLineFilenameComment as this:
///   ::= # number "filename"
bool MasmParser::parseCppHashLineFilenameComment(SMLoc L) {
  Lex(); // Eat the hash token.
  // Lexer only ever emits HashDirective if it fully formed if it's
  // done the checking already so this is an internal error.
  assert(getTok().is(AsmToken::Integer) &&
         "Lexing Cpp line comment: Expected Integer");
  int64_t LineNumber = getTok().getIntVal();
  Lex();
  assert(getTok().is(AsmToken::String) &&
         "Lexing Cpp line comment: Expected String");
  StringRef Filename = getTok().getString();
  Lex();

```
- **EN**: Implements logic around `getLoc`, `Lex`, `emplace_back`, `getPointer`, and 3 more symbols; this block manipulates JITLink graph structures or link-time passes; parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `getLoc`, `Lex`, `emplace_back`, `getPointer`, and 3 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 2371-2390
```cpp
  // Get rid of the enclosing quotes.
  Filename = Filename.substr(1, Filename.size() - 2);

  // Save the SMLoc, Filename and LineNumber for later use by diagnostics
  // and possibly DWARF file info.
  CppHashInfo.Loc = L;
  CppHashInfo.Filename = Filename;
  CppHashInfo.LineNumber = LineNumber;
  CppHashInfo.Buf = CurBuffer;
  if (FirstCppHashFilename.empty())
    FirstCppHashFilename = Filename;
  return false;
}

/// will use the last parsed cpp hash line filename comment
/// for the Filename and LineNo if any in the diagnostic.
void MasmParser::DiagHandler(const SMDiagnostic &Diag, void *Context) {
  const MasmParser *Parser = static_cast<const MasmParser *>(Context);
  raw_ostream &OS = errs();

```
- **EN**: Implements logic around `substr`, `DiagHandler`, `errs`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `substr`, `DiagHandler`, `errs` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 2391-2417
```cpp
  const SourceMgr &DiagSrcMgr = *Diag.getSourceMgr();
  SMLoc DiagLoc = Diag.getLoc();
  unsigned DiagBuf = DiagSrcMgr.FindBufferContainingLoc(DiagLoc);
  unsigned CppHashBuf =
      Parser->SrcMgr.FindBufferContainingLoc(Parser->CppHashInfo.Loc);

  // Like SourceMgr::printMessage() we need to print the include stack if any
  // before printing the message.
  unsigned DiagCurBuffer = DiagSrcMgr.FindBufferContainingLoc(DiagLoc);
  if (!Parser->SavedDiagHandler && DiagCurBuffer &&
      DiagCurBuffer != DiagSrcMgr.getMainFileID()) {
    SMLoc ParentIncludeLoc = DiagSrcMgr.getParentIncludeLoc(DiagCurBuffer);
    DiagSrcMgr.PrintIncludeStack(ParentIncludeLoc, OS);
  }

  // If we have not parsed a cpp hash line filename comment or the source
  // manager changed or buffer changed (like in a nested include) then just
  // print the normal diagnostic using its Filename and LineNo.
  if (!Parser->CppHashInfo.LineNumber || &DiagSrcMgr != &Parser->SrcMgr ||
      DiagBuf != CppHashBuf) {
    if (Parser->SavedDiagHandler)
      Parser->SavedDiagHandler(Diag, Parser->SavedDiagContext);
    else
      Diag.print(nullptr, OS);
    return;
  }

```
- **EN**: Implements logic around `getSourceMgr`, `getLoc`, `FindBufferContainingLoc`, `getMainFileID`, and 4 more symbols; this block parses assembly syntax or operands.
- **CN**: 围绕 `getSourceMgr`, `getLoc`, `FindBufferContainingLoc`, `getMainFileID`, and 4 more symbols 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 2418-2438
```cpp
  // Use the CppHashFilename and calculate a line number based on the
  // CppHashInfo.Loc and CppHashInfo.LineNumber relative to this Diag's SMLoc
  // for the diagnostic.
  const std::string &Filename = std::string(Parser->CppHashInfo.Filename);

  int DiagLocLineNo = DiagSrcMgr.FindLineNumber(DiagLoc, DiagBuf);
  int CppHashLocLineNo =
      Parser->SrcMgr.FindLineNumber(Parser->CppHashInfo.Loc, CppHashBuf);
  int LineNo =
      Parser->CppHashInfo.LineNumber - 1 + (DiagLocLineNo - CppHashLocLineNo);

  SMDiagnostic NewDiag(*Diag.getSourceMgr(), Diag.getLoc(), Filename, LineNo,
                       Diag.getColumnNo(), Diag.getKind(), Diag.getMessage(),
                       Diag.getLineContents(), Diag.getRanges());

  if (Parser->SavedDiagHandler)
    Parser->SavedDiagHandler(NewDiag, Parser->SavedDiagContext);
  else
    NewDiag.print(nullptr, OS);
}

```
- **EN**: Implements logic around `string`, `FindLineNumber`, `NewDiag`, `getColumnNo`, and 3 more symbols; this block parses assembly syntax or operands.
- **CN**: 围绕 `string`, `FindLineNumber`, `NewDiag`, `getColumnNo`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 2439-2462
```cpp
// This is similar to the IsIdentifierChar function in AsmLexer.cpp, but does
// not accept '.'.
static bool isMacroParameterChar(char C) {
  return isAlnum(C) || C == '_' || C == '$' || C == '@' || C == '?';
}

bool MasmParser::expandMacro(raw_svector_ostream &OS, StringRef Body,
                             ArrayRef<MCAsmMacroParameter> Parameters,
                             ArrayRef<MCAsmMacroArgument> A,
                             const std::vector<std::string> &Locals, SMLoc L) {
  unsigned NParameters = Parameters.size();
  if (NParameters != A.size())
    return Error(L, "Wrong number of arguments");
  StringMap<std::string> LocalSymbols;
  std::string Name;
  Name.reserve(6);
  for (StringRef Local : Locals) {
    raw_string_ostream LocalName(Name);
    LocalName << "??"
              << format_hex_no_prefix(LocalCounter++, 4, /*Upper=*/true);
    LocalSymbols.insert({Local, Name});
    Name.clear();
  }

```
- **EN**: Implements logic around `isMacroParameterChar`, `isAlnum`, `expandMacro`, `size`, and 6 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `isMacroParameterChar`, `isAlnum`, `expandMacro`, `size`, and 6 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 2463-2502
```cpp
  std::optional<char> CurrentQuote;
  while (!Body.empty()) {
    // Scan for the next substitution.
    std::size_t End = Body.size(), Pos = 0;
    std::size_t IdentifierPos = End;
    for (; Pos != End; ++Pos) {
      // Find the next possible macro parameter, including preceding a '&'
      // inside quotes.
      if (Body[Pos] == '&')
        break;
      if (isMacroParameterChar(Body[Pos])) {
        if (!CurrentQuote)
          break;
        if (IdentifierPos == End)
          IdentifierPos = Pos;
      } else {
        IdentifierPos = End;
      }

      // Track quotation status
      if (!CurrentQuote) {
        if (Body[Pos] == '\'' || Body[Pos] == '"')
          CurrentQuote = Body[Pos];
      } else if (Body[Pos] == CurrentQuote) {
        if (Pos + 1 != End && Body[Pos + 1] == CurrentQuote) {
          // Escaped quote, and quotes aren't identifier chars; skip
          ++Pos;
          continue;
        } else {
          CurrentQuote.reset();
        }
      }
    }
    if (IdentifierPos != End) {
      // We've recognized an identifier before an apostrophe inside quotes;
      // check once to see if we can expand it.
      Pos = IdentifierPos;
      IdentifierPos = End;
    }

```
- **EN**: Implements logic around `size`, `reset`.
- **CN**: 围绕 `size`, `reset` 实现具体逻辑。

### Lines 2503-2523
```cpp
    // Add the prefix.
    OS << Body.slice(0, Pos);

    // Check if we reached the end.
    if (Pos == End)
      break;

    unsigned I = Pos;
    bool InitialAmpersand = (Body[I] == '&');
    if (InitialAmpersand) {
      ++I;
      ++Pos;
    }
    while (I < End && isMacroParameterChar(Body[I]))
      ++I;

    const char *Begin = Body.data() + Pos;
    StringRef Argument(Begin, I - Pos);
    const std::string ArgumentLower = Argument.lower();
    unsigned Index = 0;

```
- **EN**: Implements logic around `slice`, `data`, `Argument`, `lower`.
- **CN**: 围绕 `slice`, `data`, `Argument`, `lower` 实现具体逻辑。

### Lines 2524-2552
```cpp
    for (; Index < NParameters; ++Index)
      if (Parameters[Index].Name.equals_insensitive(ArgumentLower))
        break;

    if (Index == NParameters) {
      if (InitialAmpersand)
        OS << '&';
      auto it = LocalSymbols.find(ArgumentLower);
      if (it != LocalSymbols.end())
        OS << it->second;
      else
        OS << Argument;
      Pos = I;
    } else {
      for (const AsmToken &Token : A[Index]) {
        // In MASM, you can write '%expr'.
        // The prefix '%' evaluates the expression 'expr'
        // and uses the result as a string (e.g. replace %(1+2) with the
        // string "3").
        // Here, we identify the integer token which is the result of the
        // absolute expression evaluation and replace it with its string
        // representation.
        if (Token.getString().front() == '%' && Token.is(AsmToken::Integer))
          // Emit an integer value to the buffer.
          OS << Token.getIntVal();
        else
          OS << Token.getString();
      }

```
- **EN**: Implements logic around `find`, `getIntVal`, `getString`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `find`, `getIntVal`, `getString` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 2553-2577
```cpp
      Pos += Argument.size();
      if (Pos < End && Body[Pos] == '&') {
        ++Pos;
      }
    }
    // Update the scan point.
    Body = Body.substr(Pos);
  }

  return false;
}

bool MasmParser::parseMacroArgument(const MCAsmMacroParameter *MP,
                                    MCAsmMacroArgument &MA,
                                    AsmToken::TokenKind EndTok) {
  if (MP && MP->Vararg) {
    if (Lexer.isNot(EndTok)) {
      SmallVector<StringRef, 1> Str = parseStringRefsTo(EndTok);
      for (StringRef S : Str) {
        MA.emplace_back(AsmToken::String, S);
      }
    }
    return false;
  }

```
- **EN**: Implements logic around `size`, `substr`, `parseMacroArgument`, `parseStringRefsTo`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `size`, `substr`, `parseMacroArgument`, `parseStringRefsTo`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 2578-2597
```cpp
  SMLoc StrLoc = Lexer.getLoc(), EndLoc;
  if (Lexer.is(AsmToken::Less) && isAngleBracketString(StrLoc, EndLoc)) {
    const char *StrChar = StrLoc.getPointer() + 1;
    const char *EndChar = EndLoc.getPointer() - 1;
    jumpToLoc(EndLoc, CurBuffer, EndStatementAtEOFStack.back());
    /// Eat from '<' to '>'.
    Lex();
    MA.emplace_back(AsmToken::String, StringRef(StrChar, EndChar - StrChar));
    return false;
  }

  unsigned ParenLevel = 0;

  while (true) {
    if (Lexer.is(AsmToken::Eof) || Lexer.is(AsmToken::Equal))
      return TokError("unexpected token");

    if (ParenLevel == 0 && Lexer.is(AsmToken::Comma))
      break;

```
- **EN**: Implements logic around `getLoc`, `getPointer`, `jumpToLoc`, `Lex`, and 2 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `getLoc`, `getPointer`, `jumpToLoc`, `Lex`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 2598-2627
```cpp
    // handleMacroEntry relies on not advancing the lexer here
    // to be able to fill in the remaining default parameter values
    if (Lexer.is(EndTok) && (EndTok != AsmToken::RParen || ParenLevel == 0))
      break;

    // Adjust the current parentheses level.
    if (Lexer.is(AsmToken::LParen))
      ++ParenLevel;
    else if (Lexer.is(AsmToken::RParen) && ParenLevel)
      --ParenLevel;

    // Append the token to the current argument list.
    MA.push_back(getTok());
    Lex();
  }

  if (ParenLevel != 0)
    return TokError("unbalanced parentheses in argument");

  if (MA.empty() && MP) {
    if (MP->Required) {
      return TokError("missing value for required parameter '" + MP->Name +
                      "'");
    } else {
      MA = MP->Value;
    }
  }
  return false;
}

```
- **EN**: Implements logic around `push_back`, `Lex`, `TokError`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `push_back`, `Lex`, `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 2628-2650
```cpp
// Parse the macro instantiation arguments.
bool MasmParser::parseMacroArguments(const MCAsmMacro *M,
                                     MCAsmMacroArguments &A,
                                     AsmToken::TokenKind EndTok) {
  const unsigned NParameters = M ? M->Parameters.size() : 0;
  bool NamedParametersFound = false;
  SmallVector<SMLoc, 4> FALocs;

  A.resize(NParameters);
  FALocs.resize(NParameters);

  // Parse two kinds of macro invocations:
  // - macros defined without any parameters accept an arbitrary number of them
  // - macros defined with parameters accept at most that many of them
  for (unsigned Parameter = 0; !NParameters || Parameter < NParameters;
       ++Parameter) {
    SMLoc IDLoc = Lexer.getLoc();
    MCAsmMacroParameter FA;

    if (Lexer.is(AsmToken::Identifier) && peekTok().is(AsmToken::Equal)) {
      if (parseIdentifier(FA.Name))
        return Error(IDLoc, "invalid argument identifier for formal argument");

```
- **EN**: Implements logic around `parseMacroArguments`, `size`, `resize`, `getLoc`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseMacroArguments`, `size`, `resize`, `getLoc`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 2651-2679
```cpp
      if (Lexer.isNot(AsmToken::Equal))
        return TokError("expected '=' after formal parameter identifier");

      Lex();

      NamedParametersFound = true;
    }

    if (NamedParametersFound && FA.Name.empty())
      return Error(IDLoc, "cannot mix positional and keyword arguments");

    unsigned PI = Parameter;
    if (!FA.Name.empty()) {
      assert(M && "expected macro to be defined");
      unsigned FAI = 0;
      for (FAI = 0; FAI < NParameters; ++FAI)
        if (M->Parameters[FAI].Name == FA.Name)
          break;

      if (FAI >= NParameters) {
        return Error(IDLoc, "parameter named '" + FA.Name +
                                "' does not exist for macro '" + M->Name + "'");
      }
      PI = FAI;
    }
    const MCAsmMacroParameter *MP = nullptr;
    if (M && PI < NParameters)
      MP = &M->Parameters[PI];

```
- **EN**: Implements logic around `TokError`, `Lex`, `Error`, `assert`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `Lex`, `Error`, `assert` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 2680-2703
```cpp
    SMLoc StrLoc = Lexer.getLoc();
    SMLoc EndLoc;
    if (Lexer.is(AsmToken::Percent)) {
      const MCExpr *AbsoluteExp;
      int64_t Value;
      /// Eat '%'.
      Lex();
      if (parseExpression(AbsoluteExp, EndLoc))
        return false;
      if (!AbsoluteExp->evaluateAsAbsolute(Value,
                                           getStreamer().getAssemblerPtr()))
        return Error(StrLoc, "expected absolute expression");
      const char *StrChar = StrLoc.getPointer();
      const char *EndChar = EndLoc.getPointer();
      AsmToken newToken(AsmToken::Integer,
                        StringRef(StrChar, EndChar - StrChar), Value);
      FA.Value.push_back(newToken);
    } else if (parseMacroArgument(MP, FA.Value, EndTok)) {
      if (M)
        return addErrorSuffix(" in '" + M->Name + "' macro");
      else
        return true;
    }

```
- **EN**: Implements logic around `getLoc`, `Lex`, `getStreamer`, `Error`, and 5 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `getLoc`, `Lex`, `getStreamer`, `Error`, and 5 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 2704-2729
```cpp
    if (!FA.Value.empty()) {
      if (A.size() <= PI)
        A.resize(PI + 1);
      A[PI] = FA.Value;

      if (FALocs.size() <= PI)
        FALocs.resize(PI + 1);

      FALocs[PI] = Lexer.getLoc();
    }

    // At the end of the statement, fill in remaining arguments that have
    // default values. If there aren't any, then the next argument is
    // required but missing
    if (Lexer.is(EndTok)) {
      bool Failure = false;
      for (unsigned FAI = 0; FAI < NParameters; ++FAI) {
        if (A[FAI].empty()) {
          if (M->Parameters[FAI].Required) {
            Error(FALocs[FAI].isValid() ? FALocs[FAI] : Lexer.getLoc(),
                  "missing value for required parameter "
                  "'" +
                      M->Parameters[FAI].Name + "' in macro '" + M->Name + "'");
            Failure = true;
          }

```
- **EN**: Implements logic around `resize`, `getLoc`, `Error`; this block parses assembly syntax or operands.
- **CN**: 围绕 `resize`, `getLoc`, `Error` 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 2730-2757
```cpp
          if (!M->Parameters[FAI].Value.empty())
            A[FAI] = M->Parameters[FAI].Value;
        }
      }
      return Failure;
    }

    if (Lexer.is(AsmToken::Comma))
      Lex();
  }

  return TokError("too many positional arguments");
}

bool MasmParser::handleMacroEntry(const MCAsmMacro *M, SMLoc NameLoc,
                                  AsmToken::TokenKind ArgumentEndTok) {
  // Arbitrarily limit macro nesting depth (default matches 'as'). We can
  // eliminate this, although we should protect against infinite loops.
  unsigned MaxNestingDepth = AsmMacroMaxNestingDepth;
  if (ActiveMacros.size() == MaxNestingDepth) {
    std::ostringstream MaxNestingDepthError;
    MaxNestingDepthError << "macros cannot be nested more than "
                         << MaxNestingDepth << " levels deep."
                         << " Use -asm-macro-max-nesting-depth to increase "
                            "this limit.";
    return TokError(MaxNestingDepthError.str());
  }

```
- **EN**: Implements logic around `Lex`, `TokError`, `handleMacroEntry`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `Lex`, `TokError`, `handleMacroEntry` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 2758-2777
```cpp
  MCAsmMacroArguments A;
  if (parseMacroArguments(M, A, ArgumentEndTok) || parseToken(ArgumentEndTok))
    return true;

  // Macro instantiation is lexical, unfortunately. We construct a new buffer
  // to hold the macro body with substitutions.
  SmallString<256> Buf;
  StringRef Body = M->Body;
  raw_svector_ostream OS(Buf);

  if (expandMacro(OS, Body, M->Parameters, A, M->Locals, getTok().getLoc()))
    return true;

  // We include the endm in the buffer as our cue to exit the macro
  // instantiation.
  OS << "endm\n";

  std::unique_ptr<MemoryBuffer> Instantiation =
      MemoryBuffer::getMemBufferCopy(OS.str(), "<instantiation>");

```
- **EN**: Implements logic around `OS`, `getMemBufferCopy`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `OS`, `getMemBufferCopy` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 2778-2801
```cpp
  // Create the macro instantiation object and add to the current macro
  // instantiation stack.
  MacroInstantiation *MI = new MacroInstantiation{
      NameLoc, CurBuffer, getTok().getLoc(), TheCondStack.size()};
  ActiveMacros.push_back(MI);

  ++NumOfMacroInstantiations;

  // Jump to the macro instantiation and prime the lexer.
  CurBuffer = SrcMgr.AddNewSourceBuffer(std::move(Instantiation), SMLoc());
  Lexer.setBuffer(SrcMgr.getMemoryBuffer(CurBuffer)->getBuffer());
  EndStatementAtEOFStack.push_back(true);
  Lex();

  return false;
}

void MasmParser::handleMacroExit() {
  // Jump to the token we should return to, and consume it.
  EndStatementAtEOFStack.pop_back();
  jumpToLoc(ActiveMacros.back()->ExitLoc, ActiveMacros.back()->ExitBuffer,
            EndStatementAtEOFStack.back());
  Lex();

```
- **EN**: Implements logic around `getTok`, `push_back`, `AddNewSourceBuffer`, `setBuffer`, and 5 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `getTok`, `push_back`, `AddNewSourceBuffer`, `setBuffer`, and 5 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 2802-2822
```cpp
  // Pop the instantiation entry.
  delete ActiveMacros.back();
  ActiveMacros.pop_back();
}

bool MasmParser::handleMacroInvocation(const MCAsmMacro *M, SMLoc NameLoc) {
  if (!M->IsFunction)
    return Error(NameLoc, "cannot invoke macro procedure as function");

  if (parseToken(AsmToken::LParen, "invoking macro function '" + M->Name +
                                       "' requires arguments in parentheses") ||
      handleMacroEntry(M, NameLoc, AsmToken::RParen))
    return true;

  // Parse all statements in the macro, retrieving the exit value when it ends.
  std::string ExitValue;
  SmallVector<AsmRewrite, 4> AsmStrRewrites;
  while (Lexer.isNot(AsmToken::Eof)) {
    ParseStatementInfo Info(&AsmStrRewrites);
    bool HasError = parseStatement(Info, nullptr);

```
- **EN**: Implements logic around `back`, `pop_back`, `handleMacroInvocation`, `Error`, and 3 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `back`, `pop_back`, `handleMacroInvocation`, `Error`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 2823-2846
```cpp
    if (!HasError && Info.ExitValue) {
      ExitValue = std::move(*Info.ExitValue);
      break;
    }

    // If we have a Lexer Error we are on an Error Token. Load in Lexer Error
    // for printing ErrMsg via Lex() only if no (presumably better) parser error
    // exists.
    if (HasError && !hasPendingError() && Lexer.getTok().is(AsmToken::Error))
      Lex();

    // parseStatement returned true so may need to emit an error.
    printPendingErrors();

    // Skipping to the next line if needed.
    if (HasError && !getLexer().justConsumedEOL())
      eatToEndOfStatement();
  }

  // Exit values may require lexing, unfortunately. We construct a new buffer to
  // hold the exit value.
  std::unique_ptr<MemoryBuffer> MacroValue =
      MemoryBuffer::getMemBufferCopy(ExitValue, "<macro-value>");

```
- **EN**: Implements logic around `move`, `Lex`, `printPendingErrors`, `eatToEndOfStatement`, and 1 more symbols; this block parses assembly syntax or operands.
- **CN**: 围绕 `move`, `Lex`, `printPendingErrors`, `eatToEndOfStatement`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 2847-2870
```cpp
  // Jump from this location to the instantiated exit value, and prime the
  // lexer.
  CurBuffer = SrcMgr.AddNewSourceBuffer(std::move(MacroValue), Lexer.getLoc());
  Lexer.setBuffer(SrcMgr.getMemoryBuffer(CurBuffer)->getBuffer(), nullptr,
                  /*EndStatementAtEOF=*/false);
  EndStatementAtEOFStack.push_back(false);
  Lex();

  return false;
}

/// parseIdentifier:
///   ::= identifier
///   ::= string
bool MasmParser::parseIdentifier(StringRef &Res,
                                 IdentifierPositionKind Position) {
  // The assembler has relaxed rules for accepting identifiers, in particular we
  // allow things like '.globl $foo' and '.def @feat.00', which would normally
  // be separate tokens. At this level, we have already lexed so we cannot
  // (currently) handle this as a context dependent token, instead we detect
  // adjacent tokens and return the combined identifier.
  if (Lexer.is(AsmToken::Dollar) || Lexer.is(AsmToken::At)) {
    SMLoc PrefixLoc = getLexer().getLoc();

```
- **EN**: Implements logic around `AddNewSourceBuffer`, `setBuffer`, `push_back`, `Lex`, and 2 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `AddNewSourceBuffer`, `setBuffer`, `push_back`, `Lex`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 2871-2890
```cpp
    // Consume the prefix character, and check for a following identifier.

    AsmToken nextTok = peekTok(false);

    if (nextTok.isNot(AsmToken::Identifier))
      return true;

    // We have a '$' or '@' followed by an identifier, make sure they are adjacent.
    if (PrefixLoc.getPointer() + 1 != nextTok.getLoc().getPointer())
      return true;

    // eat $ or @
    Lexer.Lex(); // Lexer's Lex guarantees consecutive token.
    // Construct the joined identifier and consume the token.
    Res =
        StringRef(PrefixLoc.getPointer(), getTok().getIdentifier().size() + 1);
    Lex(); // Parser Lex to maintain invariants.
    return false;
  }

```
- **EN**: Implements logic around `peekTok`, `Lex`, `StringRef`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `peekTok`, `Lex`, `StringRef` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 2891-2910
```cpp
  if (Lexer.isNot(AsmToken::Identifier) && Lexer.isNot(AsmToken::String))
    return true;

  Res = getTok().getIdentifier();

  // Consume the identifier token - but if parsing certain directives, avoid
  // lexical expansion of the next token.
  ExpandKind ExpandNextToken = ExpandMacros;
  if (Position == StartOfStatement &&
      StringSwitch<bool>(Res)
          .CaseLower("echo", true)
          .CasesLower({"ifdef", "ifndef", "elseifdef", "elseifndef"}, true)
          .Default(false)) {
    ExpandNextToken = DoNotExpandMacros;
  }
  Lex(ExpandNextToken);

  return false;
}

```
- **EN**: Implements logic around `getTok`, `StringSwitch<bool>`, `CaseLower`, `CasesLower`, and 2 more symbols; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getTok`, `StringSwitch<bool>`, `CaseLower`, `CasesLower`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 2911-2934
```cpp
/// parseDirectiveEquate:
///  ::= name "=" expression
///    | name "equ" expression    (not redefinable)
///    | name "equ" text-list
///    | name "textequ" text-list (redefinability unspecified)
bool MasmParser::parseDirectiveEquate(StringRef IDVal, StringRef Name,
                                      DirectiveKind DirKind, SMLoc NameLoc) {
  auto BuiltinIt = BuiltinSymbolMap.find(Name.lower());
  if (BuiltinIt != BuiltinSymbolMap.end())
    return Error(NameLoc, "cannot redefine a built-in symbol");

  Variable &Var = Variables[Name.lower()];
  if (Var.Name.empty()) {
    Var.Name = Name;
  }

  SMLoc StartLoc = Lexer.getLoc();
  if (DirKind == DK_EQU || DirKind == DK_TEXTEQU) {
    // "equ" and "textequ" both allow text expressions.
    std::string Value;
    std::string TextItem;
    if (!parseTextItem(TextItem)) {
      Value += TextItem;

```
- **EN**: Implements logic around `parseDirectiveEquate`, `find`, `Error`, `lower`, and 1 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveEquate`, `find`, `Error`, `lower`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 2935-2962
```cpp
      // Accept a text-list, not just one text-item.
      auto parseItem = [&]() -> bool {
        if (parseTextItem(TextItem))
          return TokError("expected text item");
        Value += TextItem;
        return false;
      };
      if (parseOptionalToken(AsmToken::Comma) && parseMany(parseItem))
        return addErrorSuffix(" in '" + Twine(IDVal) + "' directive");

      if (!Var.IsText || Var.TextValue != Value) {
        switch (Var.Redefinable) {
        case Variable::NOT_REDEFINABLE:
          return Error(getTok().getLoc(), "invalid variable redefinition");
        case Variable::WARN_ON_REDEFINITION:
          if (Warning(NameLoc, "redefining '" + Name +
                                   "', already defined on the command line")) {
            return true;
          }
          break;
        default:
          break;
        }
      }
      Var.IsText = true;
      Var.TextValue = Value;
      Var.Redefinable = Variable::REDEFINABLE;

```
- **EN**: Implements logic around `TokError`, `addErrorSuffix`, `Error`; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `addErrorSuffix`, `Error` 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 2963-2984
```cpp
      return false;
    }
  }
  if (DirKind == DK_TEXTEQU)
    return TokError("expected <text> in '" + Twine(IDVal) + "' directive");

  // Parse as expression assignment.
  const MCExpr *Expr;
  SMLoc EndLoc;
  if (parseExpression(Expr, EndLoc))
    return addErrorSuffix(" in '" + Twine(IDVal) + "' directive");
  StringRef ExprAsString = StringRef(
      StartLoc.getPointer(), EndLoc.getPointer() - StartLoc.getPointer());

  int64_t Value;
  if (!Expr->evaluateAsAbsolute(Value, getStreamer().getAssemblerPtr())) {
    if (DirKind == DK_ASSIGN)
      return Error(
          StartLoc,
          "expected absolute expression; not all symbols have known values",
          {StartLoc, EndLoc});

```
- **EN**: Implements logic around `TokError`, `addErrorSuffix`, `StringRef`, `getPointer`, and 1 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `addErrorSuffix`, `StringRef`, `getPointer`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 2985-3004
```cpp
    // Not an absolute expression; define as a text replacement.
    if (!Var.IsText || Var.TextValue != ExprAsString) {
      switch (Var.Redefinable) {
      case Variable::NOT_REDEFINABLE:
        return Error(getTok().getLoc(), "invalid variable redefinition");
      case Variable::WARN_ON_REDEFINITION:
        if (Warning(NameLoc, "redefining '" + Name +
                                 "', already defined on the command line")) {
          return true;
        }
        break;
      default:
        break;
      }
    }

    Var.IsText = true;
    Var.TextValue = ExprAsString.str();
    Var.Redefinable = Variable::REDEFINABLE;

```
- **EN**: Implements logic around `Error`, `str`; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `Error`, `str` 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 3005-3027
```cpp
    return false;
  }

  auto *Sym = static_cast<MCSymbolCOFF *>(getContext().parseSymbol(Var.Name));
  const MCConstantExpr *PrevValue =
      Sym->isVariable()
          ? dyn_cast_or_null<MCConstantExpr>(Sym->getVariableValue())
          : nullptr;
  if (Var.IsText || !PrevValue || PrevValue->getValue() != Value) {
    switch (Var.Redefinable) {
    case Variable::NOT_REDEFINABLE:
      return Error(getTok().getLoc(), "invalid variable redefinition");
    case Variable::WARN_ON_REDEFINITION:
      if (Warning(NameLoc, "redefining '" + Name +
                               "', already defined on the command line")) {
        return true;
      }
      break;
    default:
      break;
    }
  }

```
- **EN**: Implements logic around `getContext`, `isVariable`, `dyn_cast_or_null<MCConstantExpr>`, `Error`; this block parses assembly syntax or operands; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getContext`, `isVariable`, `dyn_cast_or_null<MCConstantExpr>`, `Error` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 3028-3060
```cpp
  Var.IsText = false;
  Var.TextValue.clear();
  Var.Redefinable = (DirKind == DK_ASSIGN) ? Variable::REDEFINABLE
                                           : Variable::NOT_REDEFINABLE;

  Sym->setRedefinable(Var.Redefinable != Variable::NOT_REDEFINABLE);
  Sym->setVariableValue(Expr);
  Sym->setExternal(false);

  return false;
}

bool MasmParser::parseEscapedString(std::string &Data) {
  if (check(getTok().isNot(AsmToken::String), "expected string"))
    return true;

  Data = "";
  char Quote = getTok().getString().front();
  StringRef Str = getTok().getStringContents();
  Data.reserve(Str.size());
  for (size_t i = 0, e = Str.size(); i != e; ++i) {
    Data.push_back(Str[i]);
    if (Str[i] == Quote) {
      // MASM treats doubled delimiting quotes as an escaped delimiting quote.
      // If we're escaping the string's trailing delimiter, we're definitely
      // missing a quotation mark.
      if (i + 1 == Str.size())
        return Error(getTok().getLoc(), "missing quotation mark in string");
      if (Str[i + 1] == Quote)
        ++i;
    }
  }

```
- **EN**: Implements logic around `clear`, `setRedefinable`, `setVariableValue`, `setExternal`, and 5 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `clear`, `setRedefinable`, `setVariableValue`, `setExternal`, and 5 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 3061-3080
```cpp
  Lex();
  return false;
}

bool MasmParser::parseAngleBracketString(std::string &Data) {
  SMLoc EndLoc, StartLoc = getTok().getLoc();
  if (isAngleBracketString(StartLoc, EndLoc)) {
    const char *StartChar = StartLoc.getPointer() + 1;
    const char *EndChar = EndLoc.getPointer() - 1;
    jumpToLoc(EndLoc, CurBuffer, EndStatementAtEOFStack.back());
    // Eat from '<' to '>'.
    Lex();

    Data = angleBracketString(StringRef(StartChar, EndChar - StartChar));
    return false;
  }
  return true;
}

/// textItem ::= textLiteral | textMacroID | % constExpr
```
- **EN**: Implements logic around `Lex`, `parseAngleBracketString`, `getTok`, `getPointer`, and 2 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `Lex`, `parseAngleBracketString`, `getTok`, `getPointer`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 3081-3104
```cpp
bool MasmParser::parseTextItem(std::string &Data) {
  switch (getTok().getKind()) {
  default:
    return true;
  case AsmToken::Percent: {
    int64_t Res;
    if (parseToken(AsmToken::Percent) || parseAbsoluteExpression(Res))
      return true;
    Data = std::to_string(Res);
    return false;
  }
  case AsmToken::Less:
  case AsmToken::LessEqual:
  case AsmToken::LessLess:
  case AsmToken::LessGreater:
    return parseAngleBracketString(Data);
  case AsmToken::Identifier: {
    // This must be a text macro; we need to expand it accordingly.
    StringRef ID;
    SMLoc StartLoc = getTok().getLoc();
    if (parseIdentifier(ID))
      return true;
    Data = ID.str();

```
- **EN**: Implements logic around `parseTextItem`, `to_string`, `parseAngleBracketString`, `getTok`, and 1 more symbols; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `parseTextItem`, `to_string`, `parseAngleBracketString`, `getTok`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 3105-3133
```cpp
    bool Expanded = false;
    while (true) {
      // Try to resolve as a built-in text macro
      auto BuiltinIt = BuiltinSymbolMap.find(ID.lower());
      if (BuiltinIt != BuiltinSymbolMap.end()) {
        std::optional<std::string> BuiltinText =
            evaluateBuiltinTextMacro(BuiltinIt->getValue(), StartLoc);
        if (!BuiltinText) {
          // Not a text macro; break without substituting
          break;
        }
        Data = std::move(*BuiltinText);
        ID = StringRef(Data);
        Expanded = true;
        continue;
      }

      // Try to resolve as a built-in macro function
      auto BuiltinFuncIt = BuiltinFunctionMap.find(ID.lower());
      if (BuiltinFuncIt != BuiltinFunctionMap.end()) {
        Data.clear();
        if (evaluateBuiltinMacroFunction(BuiltinFuncIt->getValue(), ID, Data)) {
          return true;
        }
        ID = StringRef(Data);
        Expanded = true;
        continue;
      }

```
- **EN**: Implements logic around `find`, `evaluateBuiltinTextMacro`, `move`, `StringRef`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `find`, `evaluateBuiltinTextMacro`, `move`, `StringRef`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 3134-3162
```cpp
      // Try to resolve as a variable text macro
      auto VarIt = Variables.find(ID.lower());
      if (VarIt != Variables.end()) {
        const Variable &Var = VarIt->getValue();
        if (!Var.IsText) {
          // Not a text macro; break without substituting
          break;
        }
        Data = Var.TextValue;
        ID = StringRef(Data);
        Expanded = true;
        continue;
      }

      break;
    }

    if (!Expanded) {
      // Not a text macro; not usable in TextItem context. Since we haven't used
      // the token, put it back for better error recovery.
      getLexer().UnLex(AsmToken(AsmToken::Identifier, ID));
      return true;
    }
    return false;
  }
  }
  llvm_unreachable("unhandled token kind");
}

```
- **EN**: Implements logic around `find`, `getValue`, `StringRef`, `getLexer`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `find`, `getValue`, `StringRef`, `getLexer`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 3163-3200
```cpp
/// parseDirectiveAscii:
///   ::= ( .ascii | .asciz | .string ) [ "string" ( , "string" )* ]
bool MasmParser::parseDirectiveAscii(StringRef IDVal, bool ZeroTerminated) {
  auto parseOp = [&]() -> bool {
    std::string Data;
    if (checkForValidSection() || parseEscapedString(Data))
      return true;
    getStreamer().emitBytes(Data);
    if (ZeroTerminated)
      getStreamer().emitBytes(StringRef("\0", 1));
    return false;
  };

  if (parseMany(parseOp))
    return addErrorSuffix(" in '" + Twine(IDVal) + "' directive");
  return false;
}

bool MasmParser::emitIntValue(const MCExpr *Value, unsigned Size) {
  // Special case constant expressions to match code generator.
  if (const MCConstantExpr *MCE = dyn_cast<MCConstantExpr>(Value)) {
    assert(Size <= 8 && "Invalid size");
    int64_t IntValue = MCE->getValue();
    if (!isUIntN(8 * Size, IntValue) && !isIntN(8 * Size, IntValue))
      return Error(MCE->getLoc(), "out of range literal value");
    getStreamer().emitIntValue(IntValue, Size);
  } else {
    const MCSymbolRefExpr *MSE = dyn_cast<MCSymbolRefExpr>(Value);
    if (MSE && MSE->getSymbol().getName() == "?") {
      // ? initializer; treat as 0.
      getStreamer().emitIntValue(0, Size);
    } else {
      getStreamer().emitValue(Value, Size, Value->getLoc());
    }
  }
  return false;
}

```
- **EN**: Implements logic around `parseDirectiveAscii`, `getStreamer`, `addErrorSuffix`, `emitIntValue`, and 4 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveAscii`, `getStreamer`, `addErrorSuffix`, `emitIntValue`, and 4 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 3201-3230
```cpp
bool MasmParser::parseScalarInitializer(unsigned Size,
                                        SmallVectorImpl<const MCExpr *> &Values,
                                        unsigned StringPadLength) {
  if (Size == 1 && getTok().is(AsmToken::String)) {
    std::string Value;
    if (parseEscapedString(Value))
      return true;
    // Treat each character as an initializer.
    for (const unsigned char CharVal : Value)
      Values.push_back(MCConstantExpr::create(CharVal, getContext()));

    // Pad the string with spaces to the specified length.
    for (size_t i = Value.size(); i < StringPadLength; ++i)
      Values.push_back(MCConstantExpr::create(' ', getContext()));
  } else {
    const MCExpr *Value;
    if (parseExpression(Value))
      return true;
    if (getTok().is(AsmToken::Identifier) &&
        getTok().getString().equals_insensitive("dup")) {
      Lex(); // Eat 'dup'.
      const MCConstantExpr *MCE = dyn_cast<MCConstantExpr>(Value);
      if (!MCE)
        return Error(Value->getLoc(),
                     "cannot repeat value a non-constant number of times");
      const int64_t Repetitions = MCE->getValue();
      if (Repetitions < 0)
        return Error(Value->getLoc(),
                     "cannot repeat value a negative number of times");

```
- **EN**: Implements logic around `parseScalarInitializer`, `push_back`, `getTok`, `Lex`, and 3 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseScalarInitializer`, `push_back`, `getTok`, `Lex`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 3231-3253
```cpp
      SmallVector<const MCExpr *, 1> DuplicatedValues;
      if (parseToken(AsmToken::LParen,
                     "parentheses required for 'dup' contents") ||
          parseScalarInstList(Size, DuplicatedValues) || parseRParen())
        return true;

      for (int i = 0; i < Repetitions; ++i)
        Values.append(DuplicatedValues.begin(), DuplicatedValues.end());
    } else {
      Values.push_back(Value);
    }
  }
  return false;
}

bool MasmParser::parseScalarInstList(unsigned Size,
                                     SmallVectorImpl<const MCExpr *> &Values,
                                     const AsmToken::TokenKind EndToken) {
  while (getTok().isNot(EndToken) &&
         (EndToken != AsmToken::Greater ||
          getTok().isNot(AsmToken::GreaterGreater))) {
    parseScalarInitializer(Size, Values);

```
- **EN**: Implements logic around `parseScalarInstList`, `append`, `push_back`, `getTok`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseScalarInstList`, `append`, `push_back`, `getTok`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 3254-3274
```cpp
    // If we see a comma, continue, and allow line continuation.
    if (!parseOptionalToken(AsmToken::Comma))
      break;
    parseOptionalToken(AsmToken::EndOfStatement);
  }
  return false;
}

bool MasmParser::emitIntegralValues(unsigned Size, unsigned *Count) {
  SmallVector<const MCExpr *, 1> Values;
  if (checkForValidSection() || parseScalarInstList(Size, Values))
    return true;

  for (const auto *Value : Values) {
    emitIntValue(Value, Size);
  }
  if (Count)
    *Count = Values.size();
  return false;
}

```
- **EN**: Implements logic around `parseOptionalToken`, `emitIntegralValues`, `emitIntValue`, `size`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseOptionalToken`, `emitIntegralValues`, `emitIntValue`, `size` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 3275-3295
```cpp
// Add a field to the current structure.
bool MasmParser::addIntegralField(StringRef Name, unsigned Size) {
  StructInfo &Struct = StructInProgress.back();
  FieldInfo &Field = Struct.addField(Name, FT_INTEGRAL, Size);
  IntFieldInfo &IntInfo = Field.Contents.IntInfo;

  Field.Type = Size;

  if (parseScalarInstList(Size, IntInfo.Values))
    return true;

  Field.SizeOf = Field.Type * IntInfo.Values.size();
  Field.LengthOf = IntInfo.Values.size();
  const unsigned FieldEnd = Field.Offset + Field.SizeOf;
  if (!Struct.IsUnion) {
    Struct.NextOffset = FieldEnd;
  }
  Struct.Size = std::max(Struct.Size, FieldEnd);
  return false;
}

```
- **EN**: Implements logic around `addIntegralField`, `back`, `addField`, `size`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `addIntegralField`, `back`, `addField`, `size`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 3296-3321
```cpp
/// parseDirectiveValue
///  ::= (byte | word | ... ) [ expression (, expression)* ]
bool MasmParser::parseDirectiveValue(StringRef IDVal, unsigned Size) {
  if (StructInProgress.empty()) {
    // Initialize data value.
    if (emitIntegralValues(Size))
      return addErrorSuffix(" in '" + Twine(IDVal) + "' directive");
  } else if (addIntegralField("", Size)) {
    return addErrorSuffix(" in '" + Twine(IDVal) + "' directive");
  }

  return false;
}

/// parseDirectiveNamedValue
///  ::= name (byte | word | ... ) [ expression (, expression)* ]
bool MasmParser::parseDirectiveNamedValue(StringRef TypeName, unsigned Size,
                                          StringRef Name, SMLoc NameLoc) {
  if (StructInProgress.empty()) {
    // Initialize named data value.
    MCSymbol *Sym = getContext().parseSymbol(Name);
    getStreamer().emitLabel(Sym);
    unsigned Count;
    if (emitIntegralValues(Size, &Count))
      return addErrorSuffix(" in '" + Twine(TypeName) + "' directive");

```
- **EN**: Implements logic around `parseDirectiveValue`, `addErrorSuffix`, `parseDirectiveNamedValue`, `getContext`, and 1 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveValue`, `addErrorSuffix`, `parseDirectiveNamedValue`, `getContext`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 3322-3348
```cpp
    AsmTypeInfo Type;
    Type.Name = TypeName;
    Type.Size = Size * Count;
    Type.ElementSize = Size;
    Type.Length = Count;
    KnownType[Name.lower()] = Type;
  } else if (addIntegralField(Name, Size)) {
    return addErrorSuffix(" in '" + Twine(TypeName) + "' directive");
  }

  return false;
}

bool MasmParser::parseRealValue(const fltSemantics &Semantics, APInt &Res) {
  // We don't truly support arithmetic on floating point expressions, so we
  // have to manually parse unary prefixes.
  bool IsNeg = false;
  SMLoc SignLoc;
  if (getLexer().is(AsmToken::Minus)) {
    SignLoc = getLexer().getLoc();
    Lexer.Lex();
    IsNeg = true;
  } else if (getLexer().is(AsmToken::Plus)) {
    SignLoc = getLexer().getLoc();
    Lexer.Lex();
  }

```
- **EN**: Implements logic around `lower`, `addErrorSuffix`, `parseRealValue`, `getLexer`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `lower`, `addErrorSuffix`, `parseRealValue`, `getLexer`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 3349-3373
```cpp
  if (Lexer.is(AsmToken::Error))
    return TokError(Lexer.getErr());
  if (Lexer.isNot(AsmToken::Integer) && Lexer.isNot(AsmToken::Real) &&
      Lexer.isNot(AsmToken::Identifier))
    return TokError("unexpected token in directive");

  // Convert to an APFloat.
  APFloat Value(Semantics);
  StringRef IDVal = getTok().getString();
  if (getLexer().is(AsmToken::Identifier)) {
    if (IDVal.equals_insensitive("infinity") || IDVal.equals_insensitive("inf"))
      Value = APFloat::getInf(Semantics);
    else if (IDVal.equals_insensitive("nan"))
      Value = APFloat::getNaN(Semantics, false, ~0);
    else if (IDVal.equals_insensitive("?"))
      Value = APFloat::getZero(Semantics);
    else
      return TokError("invalid floating point literal");
  } else if (IDVal.consume_back("r") || IDVal.consume_back("R")) {
    // MASM hexadecimal floating-point literal; no APFloat conversion needed.
    // To match ML64.exe, ignore the initial sign.
    unsigned SizeInBits = Value.getSizeInBits(Semantics);
    if (SizeInBits != (IDVal.size() << 2))
      return TokError("invalid floating point literal");

```
- **EN**: Implements logic around `TokError`, `isNot`, `Value`, `getTok`, and 4 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `isNot`, `Value`, `getTok`, and 4 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 3374-3393
```cpp
    // Consume the numeric token.
    Lex();

    Res = APInt(SizeInBits, IDVal, 16);
    if (SignLoc.isValid())
      return Warning(SignLoc, "MASM-style hex floats ignore explicit sign");
    return false;
  } else if (errorToBool(
                 Value.convertFromString(IDVal, APFloat::rmNearestTiesToEven)
                     .takeError())) {
    return TokError("invalid floating point literal");
  }
  if (IsNeg)
    Value.changeSign();

  // Consume the numeric token.
  Lex();

  Res = Value.bitcastToAPInt();

```
- **EN**: Implements logic around `Lex`, `APInt`, `Warning`, `convertFromString`, and 4 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `Lex`, `APInt`, `Warning`, `convertFromString`, and 4 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 3394-3417
```cpp
  return false;
}

bool MasmParser::parseRealInstList(const fltSemantics &Semantics,
                                   SmallVectorImpl<APInt> &ValuesAsInt,
                                   const AsmToken::TokenKind EndToken) {
  while (getTok().isNot(EndToken) ||
         (EndToken == AsmToken::Greater &&
          getTok().isNot(AsmToken::GreaterGreater))) {
    const AsmToken NextTok = peekTok();
    if (NextTok.is(AsmToken::Identifier) &&
        NextTok.getString().equals_insensitive("dup")) {
      const MCExpr *Value;
      if (parseExpression(Value) || parseToken(AsmToken::Identifier))
        return true;
      const MCConstantExpr *MCE = dyn_cast<MCConstantExpr>(Value);
      if (!MCE)
        return Error(Value->getLoc(),
                     "cannot repeat value a non-constant number of times");
      const int64_t Repetitions = MCE->getValue();
      if (Repetitions < 0)
        return Error(Value->getLoc(),
                     "cannot repeat value a negative number of times");

```
- **EN**: Implements logic around `parseRealInstList`, `getTok`, `peekTok`, `getString`, and 3 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseRealInstList`, `getTok`, `peekTok`, `getString`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 3418-3438
```cpp
      SmallVector<APInt, 1> DuplicatedValues;
      if (parseToken(AsmToken::LParen,
                     "parentheses required for 'dup' contents") ||
          parseRealInstList(Semantics, DuplicatedValues) || parseRParen())
        return true;

      for (int i = 0; i < Repetitions; ++i)
        ValuesAsInt.append(DuplicatedValues.begin(), DuplicatedValues.end());
    } else {
      APInt AsInt;
      if (parseRealValue(Semantics, AsInt))
        return true;
      ValuesAsInt.push_back(AsInt);
    }

    // Continue if we see a comma. (Also, allow line continuation.)
    if (!parseOptionalToken(AsmToken::Comma))
      break;
    parseOptionalToken(AsmToken::EndOfStatement);
  }

```
- **EN**: Implements logic around `parseRealInstList`, `append`, `push_back`, `parseOptionalToken`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseRealInstList`, `append`, `push_back`, `parseOptionalToken` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 3439-3459
```cpp
  return false;
}

// Initialize real data values.
bool MasmParser::emitRealValues(const fltSemantics &Semantics,
                                unsigned *Count) {
  if (checkForValidSection())
    return true;

  SmallVector<APInt, 1> ValuesAsInt;
  if (parseRealInstList(Semantics, ValuesAsInt))
    return true;

  for (const APInt &AsInt : ValuesAsInt) {
    getStreamer().emitIntValue(AsInt);
  }
  if (Count)
    *Count = ValuesAsInt.size();
  return false;
}

```
- **EN**: Implements logic around `emitRealValues`, `getStreamer`, `size`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `emitRealValues`, `getStreamer`, `size` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 3460-3483
```cpp
// Add a real field to the current struct.
bool MasmParser::addRealField(StringRef Name, const fltSemantics &Semantics,
                              size_t Size) {
  StructInfo &Struct = StructInProgress.back();
  FieldInfo &Field = Struct.addField(Name, FT_REAL, Size);
  RealFieldInfo &RealInfo = Field.Contents.RealInfo;

  Field.SizeOf = 0;

  if (parseRealInstList(Semantics, RealInfo.AsIntValues))
    return true;

  Field.Type = RealInfo.AsIntValues.back().getBitWidth() / 8;
  Field.LengthOf = RealInfo.AsIntValues.size();
  Field.SizeOf = Field.Type * Field.LengthOf;

  const unsigned FieldEnd = Field.Offset + Field.SizeOf;
  if (!Struct.IsUnion) {
    Struct.NextOffset = FieldEnd;
  }
  Struct.Size = std::max(Struct.Size, FieldEnd);
  return false;
}

```
- **EN**: Implements logic around `addRealField`, `back`, `addField`, `size`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `addRealField`, `back`, `addField`, `size`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 3484-3512
```cpp
/// parseDirectiveRealValue
///  ::= (real4 | real8 | real10) [ expression (, expression)* ]
bool MasmParser::parseDirectiveRealValue(StringRef IDVal,
                                         const fltSemantics &Semantics,
                                         size_t Size) {
  if (StructInProgress.empty()) {
    // Initialize data value.
    if (emitRealValues(Semantics))
      return addErrorSuffix(" in '" + Twine(IDVal) + "' directive");
  } else if (addRealField("", Semantics, Size)) {
    return addErrorSuffix(" in '" + Twine(IDVal) + "' directive");
  }
  return false;
}

/// parseDirectiveNamedRealValue
///  ::= name (real4 | real8 | real10) [ expression (, expression)* ]
bool MasmParser::parseDirectiveNamedRealValue(StringRef TypeName,
                                              const fltSemantics &Semantics,
                                              unsigned Size, StringRef Name,
                                              SMLoc NameLoc) {
  if (StructInProgress.empty()) {
    // Initialize named data value.
    MCSymbol *Sym = getContext().parseSymbol(Name);
    getStreamer().emitLabel(Sym);
    unsigned Count;
    if (emitRealValues(Semantics, &Count))
      return addErrorSuffix(" in '" + TypeName + "' directive");

```
- **EN**: Implements logic around `parseDirectiveRealValue`, `addErrorSuffix`, `parseDirectiveNamedRealValue`, `getContext`, and 1 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveRealValue`, `addErrorSuffix`, `parseDirectiveNamedRealValue`, `getContext`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 3513-3539
```cpp
    AsmTypeInfo Type;
    Type.Name = TypeName;
    Type.Size = Size * Count;
    Type.ElementSize = Size;
    Type.Length = Count;
    KnownType[Name.lower()] = Type;
  } else if (addRealField(Name, Semantics, Size)) {
    return addErrorSuffix(" in '" + TypeName + "' directive");
  }
  return false;
}

bool MasmParser::parseOptionalAngleBracketOpen() {
  const AsmToken Tok = getTok();
  if (parseOptionalToken(AsmToken::LessLess)) {
    AngleBracketDepth++;
    Lexer.UnLex(AsmToken(AsmToken::Less, Tok.getString().substr(1)));
    return true;
  } else if (parseOptionalToken(AsmToken::LessGreater)) {
    AngleBracketDepth++;
    Lexer.UnLex(AsmToken(AsmToken::Greater, Tok.getString().substr(1)));
    return true;
  } else if (parseOptionalToken(AsmToken::Less)) {
    AngleBracketDepth++;
    return true;
  }

```
- **EN**: Implements logic around `lower`, `addErrorSuffix`, `parseOptionalAngleBracketOpen`, `getTok`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `lower`, `addErrorSuffix`, `parseOptionalAngleBracketOpen`, `getTok`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 3540-3575
```cpp
  return false;
}

bool MasmParser::parseAngleBracketClose(const Twine &Msg) {
  const AsmToken Tok = getTok();
  if (parseOptionalToken(AsmToken::GreaterGreater)) {
    Lexer.UnLex(AsmToken(AsmToken::Greater, Tok.getString().substr(1)));
  } else if (parseToken(AsmToken::Greater, Msg)) {
    return true;
  }
  AngleBracketDepth--;
  return false;
}

bool MasmParser::parseFieldInitializer(const FieldInfo &Field,
                                       const IntFieldInfo &Contents,
                                       FieldInitializer &Initializer) {
  SMLoc Loc = getTok().getLoc();

  SmallVector<const MCExpr *, 1> Values;
  if (parseOptionalToken(AsmToken::LCurly)) {
    if (Field.LengthOf == 1 && Field.Type > 1)
      return Error(Loc, "Cannot initialize scalar field with array value");
    if (parseScalarInstList(Field.Type, Values, AsmToken::RCurly) ||
        parseToken(AsmToken::RCurly))
      return true;
  } else if (parseOptionalAngleBracketOpen()) {
    if (Field.LengthOf == 1 && Field.Type > 1)
      return Error(Loc, "Cannot initialize scalar field with array value");
    if (parseScalarInstList(Field.Type, Values, AsmToken::Greater) ||
        parseAngleBracketClose())
      return true;
  } else if (Field.LengthOf > 1 && Field.Type > 1) {
    return Error(Loc, "Cannot initialize array field with scalar value");
  } else if (parseScalarInitializer(Field.Type, Values,
                                    /*StringPadLength=*/Field.LengthOf)) {
```
- **EN**: Implements logic around `parseAngleBracketClose`, `getTok`, `UnLex`, `parseFieldInitializer`, and 2 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseAngleBracketClose`, `getTok`, `UnLex`, `parseFieldInitializer`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 3576-3608
```cpp
    return true;
  }

  if (Values.size() > Field.LengthOf) {
    return Error(Loc, "Initializer too long for field; expected at most " +
                          std::to_string(Field.LengthOf) + " elements, got " +
                          std::to_string(Values.size()));
  }
  // Default-initialize all remaining values.
  Values.append(Contents.Values.begin() + Values.size(), Contents.Values.end());

  Initializer = FieldInitializer(std::move(Values));
  return false;
}

bool MasmParser::parseFieldInitializer(const FieldInfo &Field,
                                       const RealFieldInfo &Contents,
                                       FieldInitializer &Initializer) {
  const fltSemantics *Semantics;
  switch (Field.Type) {
  case 4:
    Semantics = &APFloat::IEEEsingle();
    break;
  case 8:
    Semantics = &APFloat::IEEEdouble();
    break;
  case 10:
    Semantics = &APFloat::x87DoubleExtended();
    break;
  default:
    llvm_unreachable("unknown real field type");
  }

```
- **EN**: Implements logic around `Error`, `to_string`, `append`, `FieldInitializer`, and 5 more symbols; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `Error`, `to_string`, `append`, `FieldInitializer`, and 5 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 3609-3631
```cpp
  SMLoc Loc = getTok().getLoc();

  SmallVector<APInt, 1> AsIntValues;
  if (parseOptionalToken(AsmToken::LCurly)) {
    if (Field.LengthOf == 1)
      return Error(Loc, "Cannot initialize scalar field with array value");
    if (parseRealInstList(*Semantics, AsIntValues, AsmToken::RCurly) ||
        parseToken(AsmToken::RCurly))
      return true;
  } else if (parseOptionalAngleBracketOpen()) {
    if (Field.LengthOf == 1)
      return Error(Loc, "Cannot initialize scalar field with array value");
    if (parseRealInstList(*Semantics, AsIntValues, AsmToken::Greater) ||
        parseAngleBracketClose())
      return true;
  } else if (Field.LengthOf > 1) {
    return Error(Loc, "Cannot initialize array field with scalar value");
  } else {
    AsIntValues.emplace_back();
    if (parseRealValue(*Semantics, AsIntValues.back()))
      return true;
  }

```
- **EN**: Implements logic around `getTok`, `Error`, `parseToken`, `parseAngleBracketClose`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `getTok`, `Error`, `parseToken`, `parseAngleBracketClose`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 3632-3670
```cpp
  if (AsIntValues.size() > Field.LengthOf) {
    return Error(Loc, "Initializer too long for field; expected at most " +
                          std::to_string(Field.LengthOf) + " elements, got " +
                          std::to_string(AsIntValues.size()));
  }
  // Default-initialize all remaining values.
  AsIntValues.append(Contents.AsIntValues.begin() + AsIntValues.size(),
                     Contents.AsIntValues.end());

  Initializer = FieldInitializer(std::move(AsIntValues));
  return false;
}

bool MasmParser::parseFieldInitializer(const FieldInfo &Field,
                                       const StructFieldInfo &Contents,
                                       FieldInitializer &Initializer) {
  SMLoc Loc = getTok().getLoc();

  std::vector<StructInitializer> Initializers;
  if (Field.LengthOf > 1) {
    if (parseOptionalToken(AsmToken::LCurly)) {
      if (parseStructInstList(Contents.Structure, Initializers,
                              AsmToken::RCurly) ||
          parseToken(AsmToken::RCurly))
        return true;
    } else if (parseOptionalAngleBracketOpen()) {
      if (parseStructInstList(Contents.Structure, Initializers,
                              AsmToken::Greater) ||
          parseAngleBracketClose())
        return true;
    } else {
      return Error(Loc, "Cannot initialize array field with scalar value");
    }
  } else {
    Initializers.emplace_back();
    if (parseStructInitializer(Contents.Structure, Initializers.back()))
      return true;
  }

```
- **EN**: Implements logic around `Error`, `to_string`, `append`, `end`, and 6 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `Error`, `to_string`, `append`, `end`, and 6 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 3671-3696
```cpp
  if (Initializers.size() > Field.LengthOf) {
    return Error(Loc, "Initializer too long for field; expected at most " +
                          std::to_string(Field.LengthOf) + " elements, got " +
                          std::to_string(Initializers.size()));
  }
  // Default-initialize all remaining values.
  llvm::append_range(Initializers, llvm::drop_begin(Contents.Initializers,
                                                    Initializers.size()));

  Initializer = FieldInitializer(std::move(Initializers), Contents.Structure);
  return false;
}

bool MasmParser::parseFieldInitializer(const FieldInfo &Field,
                                       FieldInitializer &Initializer) {
  switch (Field.Contents.FT) {
  case FT_INTEGRAL:
    return parseFieldInitializer(Field, Field.Contents.IntInfo, Initializer);
  case FT_REAL:
    return parseFieldInitializer(Field, Field.Contents.RealInfo, Initializer);
  case FT_STRUCT:
    return parseFieldInitializer(Field, Field.Contents.StructInfo, Initializer);
  }
  llvm_unreachable("Unhandled FieldType enum");
}

```
- **EN**: Implements logic around `Error`, `to_string`, `append_range`, `size`, and 3 more symbols; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `Error`, `to_string`, `append_range`, `size`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 3697-3732
```cpp
bool MasmParser::parseStructInitializer(const StructInfo &Structure,
                                        StructInitializer &Initializer) {
  const AsmToken FirstToken = getTok();

  std::optional<AsmToken::TokenKind> EndToken;
  if (parseOptionalToken(AsmToken::LCurly)) {
    EndToken = AsmToken::RCurly;
  } else if (parseOptionalAngleBracketOpen()) {
    EndToken = AsmToken::Greater;
    AngleBracketDepth++;
  } else if (FirstToken.is(AsmToken::Identifier) &&
             FirstToken.getString() == "?") {
    // ? initializer; leave EndToken uninitialized to treat as empty.
    if (parseToken(AsmToken::Identifier))
      return true;
  } else {
    return Error(FirstToken.getLoc(), "Expected struct initializer");
  }

  auto &FieldInitializers = Initializer.FieldInitializers;
  size_t FieldIndex = 0;
  if (EndToken) {
    // Initialize all fields with given initializers.
    while (getTok().isNot(*EndToken) && FieldIndex < Structure.Fields.size()) {
      const FieldInfo &Field = Structure.Fields[FieldIndex++];
      if (parseOptionalToken(AsmToken::Comma)) {
        // Empty initializer; use the default and continue. (Also, allow line
        // continuation.)
        FieldInitializers.push_back(Field.Contents);
        parseOptionalToken(AsmToken::EndOfStatement);
        continue;
      }
      FieldInitializers.emplace_back(Field.Contents.FT);
      if (parseFieldInitializer(Field, FieldInitializers.back()))
        return true;

```
- **EN**: Introduces declarations for `initializer`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `initializer` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 3733-3753
```cpp
      // Continue if we see a comma. (Also, allow line continuation.)
      SMLoc CommaLoc = getTok().getLoc();
      if (!parseOptionalToken(AsmToken::Comma))
        break;
      if (FieldIndex == Structure.Fields.size())
        return Error(CommaLoc, "'" + Structure.Name +
                                   "' initializer initializes too many fields");
      parseOptionalToken(AsmToken::EndOfStatement);
    }
  }
  // Default-initialize all remaining fields.
  for (const FieldInfo &Field : llvm::drop_begin(Structure.Fields, FieldIndex))
    FieldInitializers.push_back(Field.Contents);

  if (EndToken) {
    if (*EndToken == AsmToken::Greater)
      return parseAngleBracketClose();

    return parseToken(*EndToken);
  }

```
- **EN**: Implements logic around `getTok`, `Error`, `parseOptionalToken`, `push_back`, and 2 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `getTok`, `Error`, `parseOptionalToken`, `push_back`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 3754-3777
```cpp
  return false;
}

bool MasmParser::parseStructInstList(
    const StructInfo &Structure, std::vector<StructInitializer> &Initializers,
    const AsmToken::TokenKind EndToken) {
  while (getTok().isNot(EndToken) ||
         (EndToken == AsmToken::Greater &&
          getTok().isNot(AsmToken::GreaterGreater))) {
    const AsmToken NextTok = peekTok();
    if (NextTok.is(AsmToken::Identifier) &&
        NextTok.getString().equals_insensitive("dup")) {
      const MCExpr *Value;
      if (parseExpression(Value) || parseToken(AsmToken::Identifier))
        return true;
      const MCConstantExpr *MCE = dyn_cast<MCConstantExpr>(Value);
      if (!MCE)
        return Error(Value->getLoc(),
                     "cannot repeat value a non-constant number of times");
      const int64_t Repetitions = MCE->getValue();
      if (Repetitions < 0)
        return Error(Value->getLoc(),
                     "cannot repeat value a negative number of times");

```
- **EN**: Implements logic around `parseStructInstList`, `getTok`, `peekTok`, `getString`, and 3 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseStructInstList`, `getTok`, `peekTok`, `getString`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 3778-3797
```cpp
      std::vector<StructInitializer> DuplicatedValues;
      if (parseToken(AsmToken::LParen,
                     "parentheses required for 'dup' contents") ||
          parseStructInstList(Structure, DuplicatedValues) || parseRParen())
        return true;

      for (int i = 0; i < Repetitions; ++i)
        llvm::append_range(Initializers, DuplicatedValues);
    } else {
      Initializers.emplace_back();
      if (parseStructInitializer(Structure, Initializers.back()))
        return true;
    }

    // Continue if we see a comma. (Also, allow line continuation.)
    if (!parseOptionalToken(AsmToken::Comma))
      break;
    parseOptionalToken(AsmToken::EndOfStatement);
  }

```
- **EN**: Implements logic around `parseStructInstList`, `append_range`, `emplace_back`, `parseOptionalToken`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseStructInstList`, `append_range`, `emplace_back`, `parseOptionalToken` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 3798-3819
```cpp
  return false;
}

bool MasmParser::emitFieldValue(const FieldInfo &Field,
                                const IntFieldInfo &Contents) {
  // Default-initialize all values.
  for (const MCExpr *Value : Contents.Values) {
    if (emitIntValue(Value, Field.Type))
      return true;
  }
  return false;
}

bool MasmParser::emitFieldValue(const FieldInfo &Field,
                                const RealFieldInfo &Contents) {
  for (const APInt &AsInt : Contents.AsIntValues) {
    getStreamer().emitIntValue(AsInt.getLimitedValue(),
                               AsInt.getBitWidth() / 8);
  }
  return false;
}

```
- **EN**: Implements logic around `emitFieldValue`, `getStreamer`, `getBitWidth`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `emitFieldValue`, `getStreamer`, `getBitWidth` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 3820-3844
```cpp
bool MasmParser::emitFieldValue(const FieldInfo &Field,
                                const StructFieldInfo &Contents) {
  for (const auto &Initializer : Contents.Initializers) {
    size_t Index = 0, Offset = 0;
    for (const auto &SubField : Contents.Structure.Fields) {
      getStreamer().emitZeros(SubField.Offset - Offset);
      Offset = SubField.Offset + SubField.SizeOf;
      emitFieldInitializer(SubField, Initializer.FieldInitializers[Index++]);
    }
  }
  return false;
}

bool MasmParser::emitFieldValue(const FieldInfo &Field) {
  switch (Field.Contents.FT) {
  case FT_INTEGRAL:
    return emitFieldValue(Field, Field.Contents.IntInfo);
  case FT_REAL:
    return emitFieldValue(Field, Field.Contents.RealInfo);
  case FT_STRUCT:
    return emitFieldValue(Field, Field.Contents.StructInfo);
  }
  llvm_unreachable("Unhandled FieldType enum");
}

```
- **EN**: Implements logic around `emitFieldValue`, `getStreamer`, `emitFieldInitializer`, `llvm_unreachable`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `emitFieldValue`, `getStreamer`, `emitFieldInitializer`, `llvm_unreachable` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 3845-3876
```cpp
bool MasmParser::emitFieldInitializer(const FieldInfo &Field,
                                      const IntFieldInfo &Contents,
                                      const IntFieldInfo &Initializer) {
  for (const auto &Value : Initializer.Values) {
    if (emitIntValue(Value, Field.Type))
      return true;
  }
  // Default-initialize all remaining values.
  for (const auto &Value :
           llvm::drop_begin(Contents.Values, Initializer.Values.size())) {
    if (emitIntValue(Value, Field.Type))
      return true;
  }
  return false;
}

bool MasmParser::emitFieldInitializer(const FieldInfo &Field,
                                      const RealFieldInfo &Contents,
                                      const RealFieldInfo &Initializer) {
  for (const auto &AsInt : Initializer.AsIntValues) {
    getStreamer().emitIntValue(AsInt.getLimitedValue(),
                               AsInt.getBitWidth() / 8);
  }
  // Default-initialize all remaining values.
  for (const auto &AsInt :
       llvm::drop_begin(Contents.AsIntValues, Initializer.AsIntValues.size())) {
    getStreamer().emitIntValue(AsInt.getLimitedValue(),
                               AsInt.getBitWidth() / 8);
  }
  return false;
}

```
- **EN**: Implements logic around `emitFieldInitializer`, `drop_begin`, `getStreamer`, `getBitWidth`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `emitFieldInitializer`, `drop_begin`, `getStreamer`, `getBitWidth` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 3877-3908
```cpp
bool MasmParser::emitFieldInitializer(const FieldInfo &Field,
                                      const StructFieldInfo &Contents,
                                      const StructFieldInfo &Initializer) {
  for (const auto &Init : Initializer.Initializers) {
    if (emitStructInitializer(Contents.Structure, Init))
      return true;
  }
  // Default-initialize all remaining values.
  for (const auto &Init : llvm::drop_begin(Contents.Initializers,
                                           Initializer.Initializers.size())) {
    if (emitStructInitializer(Contents.Structure, Init))
      return true;
  }
  return false;
}

bool MasmParser::emitFieldInitializer(const FieldInfo &Field,
                                      const FieldInitializer &Initializer) {
  switch (Field.Contents.FT) {
  case FT_INTEGRAL:
    return emitFieldInitializer(Field, Field.Contents.IntInfo,
                                Initializer.IntInfo);
  case FT_REAL:
    return emitFieldInitializer(Field, Field.Contents.RealInfo,
                                Initializer.RealInfo);
  case FT_STRUCT:
    return emitFieldInitializer(Field, Field.Contents.StructInfo,
                                Initializer.StructInfo);
  }
  llvm_unreachable("Unhandled FieldType enum");
}

```
- **EN**: Implements logic around `emitFieldInitializer`, `size`, `llvm_unreachable`; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `emitFieldInitializer`, `size`, `llvm_unreachable` 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 3909-3936
```cpp
bool MasmParser::emitStructInitializer(const StructInfo &Structure,
                                       const StructInitializer &Initializer) {
  if (!Structure.Initializable)
    return Error(getLexer().getLoc(),
                 "cannot initialize a value of type '" + Structure.Name +
                     "'; 'org' was used in the type's declaration");
  size_t Index = 0, Offset = 0;
  for (const auto &Init : Initializer.FieldInitializers) {
    const auto &Field = Structure.Fields[Index++];
    getStreamer().emitZeros(Field.Offset - Offset);
    Offset = Field.Offset + Field.SizeOf;
    if (emitFieldInitializer(Field, Init))
      return true;
  }
  // Default-initialize all remaining fields.
  for (const auto &Field : llvm::drop_begin(
           Structure.Fields, Initializer.FieldInitializers.size())) {
    getStreamer().emitZeros(Field.Offset - Offset);
    Offset = Field.Offset + Field.SizeOf;
    if (emitFieldValue(Field))
      return true;
  }
  // Add final padding.
  if (Offset != Structure.Size)
    getStreamer().emitZeros(Structure.Size - Offset);
  return false;
}

```
- **EN**: Implements logic around `emitStructInitializer`, `Error`, `getStreamer`, `size`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `emitStructInitializer`, `Error`, `getStreamer`, `size` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 3937-3960
```cpp
// Set data values from initializers.
bool MasmParser::emitStructValues(const StructInfo &Structure,
                                  unsigned *Count) {
  std::vector<StructInitializer> Initializers;
  if (parseStructInstList(Structure, Initializers))
    return true;

  for (const auto &Initializer : Initializers) {
    if (emitStructInitializer(Structure, Initializer))
      return true;
  }

  if (Count)
    *Count = Initializers.size();
  return false;
}

// Declare a field in the current struct.
bool MasmParser::addStructField(StringRef Name, const StructInfo &Structure) {
  StructInfo &OwningStruct = StructInProgress.back();
  FieldInfo &Field =
      OwningStruct.addField(Name, FT_STRUCT, Structure.AlignmentSize);
  StructFieldInfo &StructInfo = Field.Contents.StructInfo;

```
- **EN**: Implements logic around `emitStructValues`, `size`, `addStructField`, `back`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `emitStructValues`, `size`, `addStructField`, `back`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 3961-3980
```cpp
  StructInfo.Structure = Structure;
  Field.Type = Structure.Size;

  if (parseStructInstList(Structure, StructInfo.Initializers))
    return true;

  Field.LengthOf = StructInfo.Initializers.size();
  Field.SizeOf = Field.Type * Field.LengthOf;

  const unsigned FieldEnd = Field.Offset + Field.SizeOf;
  if (!OwningStruct.IsUnion) {
    OwningStruct.NextOffset = FieldEnd;
  }
  OwningStruct.Size = std::max(OwningStruct.Size, FieldEnd);

  return false;
}

/// parseDirectiveStructValue
///  ::= struct-id (<struct-initializer> | {struct-initializer})
```
- **EN**: Implements logic around `size`, `max`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `size`, `max` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 3981-4015
```cpp
///                [, (<struct-initializer> | {struct-initializer})]*
bool MasmParser::parseDirectiveStructValue(const StructInfo &Structure,
                                           StringRef Directive, SMLoc DirLoc) {
  if (StructInProgress.empty()) {
    if (emitStructValues(Structure))
      return true;
  } else if (addStructField("", Structure)) {
    return addErrorSuffix(" in '" + Twine(Directive) + "' directive");
  }

  return false;
}

/// parseDirectiveNamedValue
///  ::= name (byte | word | ... ) [ expression (, expression)* ]
bool MasmParser::parseDirectiveNamedStructValue(const StructInfo &Structure,
                                                StringRef Directive,
                                                SMLoc DirLoc, StringRef Name) {
  if (StructInProgress.empty()) {
    // Initialize named data value.
    MCSymbol *Sym = getContext().parseSymbol(Name);
    getStreamer().emitLabel(Sym);
    unsigned Count;
    if (emitStructValues(Structure, &Count))
      return true;
    AsmTypeInfo Type;
    Type.Name = Structure.Name;
    Type.Size = Structure.Size * Count;
    Type.ElementSize = Structure.Size;
    Type.Length = Count;
    KnownType[Name.lower()] = Type;
  } else if (addStructField(Name, Structure)) {
    return addErrorSuffix(" in '" + Twine(Directive) + "' directive");
  }

```
- **EN**: Implements logic around `parseDirectiveStructValue`, `addErrorSuffix`, `parseDirectiveNamedStructValue`, `getContext`, and 2 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveStructValue`, `addErrorSuffix`, `parseDirectiveNamedStructValue`, `getContext`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 4016-4042
```cpp
  return false;
}

/// parseDirectiveStruct
///  ::= <name> (STRUC | STRUCT | UNION) [fieldAlign] [, NONUNIQUE]
///      (dataDir | generalDir | offsetDir | nestedStruct)+
///      <name> ENDS
////// dataDir = data declaration
////// offsetDir = EVEN, ORG, ALIGN
bool MasmParser::parseDirectiveStruct(StringRef Directive,
                                      DirectiveKind DirKind, StringRef Name,
                                      SMLoc NameLoc) {
  // We ignore NONUNIQUE; we do not support OPTION M510 or OPTION OLDSTRUCTS
  // anyway, so all field accesses must be qualified.
  AsmToken NextTok = getTok();
  int64_t AlignmentValue = 1;
  if (NextTok.isNot(AsmToken::Comma) &&
      NextTok.isNot(AsmToken::EndOfStatement) &&
      parseAbsoluteExpression(AlignmentValue)) {
    return addErrorSuffix(" in alignment value for '" + Twine(Directive) +
                          "' directive");
  }
  if (!isPowerOf2_64(AlignmentValue)) {
    return Error(NextTok.getLoc(), "alignment must be a power of two; was " +
                                       std::to_string(AlignmentValue));
  }

```
- **EN**: Implements logic around `parseDirectiveStruct`, `getTok`, `isNot`, `parseAbsoluteExpression`, and 3 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveStruct`, `getTok`, `isNot`, `parseAbsoluteExpression`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4043-4062
```cpp
  StringRef Qualifier;
  SMLoc QualifierLoc;
  if (parseOptionalToken(AsmToken::Comma)) {
    QualifierLoc = getTok().getLoc();
    if (parseIdentifier(Qualifier))
      return addErrorSuffix(" in '" + Twine(Directive) + "' directive");
    if (!Qualifier.equals_insensitive("nonunique"))
      return Error(QualifierLoc, "Unrecognized qualifier for '" +
                                     Twine(Directive) +
                                     "' directive; expected none or NONUNIQUE");
  }

  if (parseEOL())
    return addErrorSuffix(" in '" + Twine(Directive) + "' directive");

  StructInProgress.emplace_back(Name, DirKind == DK_UNION, AlignmentValue);
  return false;
}

/// parseDirectiveNestedStruct
```
- **EN**: Implements logic around `getTok`, `addErrorSuffix`, `Error`, `Twine`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `getTok`, `addErrorSuffix`, `Error`, `Twine`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4063-4087
```cpp
///  ::= (STRUC | STRUCT | UNION) [name]
///      (dataDir | generalDir | offsetDir | nestedStruct)+
///      ENDS
bool MasmParser::parseDirectiveNestedStruct(StringRef Directive,
                                            DirectiveKind DirKind) {
  if (StructInProgress.empty())
    return TokError("missing name in top-level '" + Twine(Directive) +
                    "' directive");

  StringRef Name;
  if (getTok().is(AsmToken::Identifier)) {
    Name = getTok().getIdentifier();
    parseToken(AsmToken::Identifier);
  }
  if (parseEOL())
    return addErrorSuffix(" in '" + Twine(Directive) + "' directive");

  // Reserve space to ensure Alignment doesn't get invalidated when
  // StructInProgress grows.
  StructInProgress.reserve(StructInProgress.size() + 1);
  StructInProgress.emplace_back(Name, DirKind == DK_UNION,
                                StructInProgress.back().Alignment);
  return false;
}

```
- **EN**: Implements logic around `parseDirectiveNestedStruct`, `TokError`, `getTok`, `parseToken`, and 4 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveNestedStruct`, `TokError`, `getTok`, `parseToken`, and 4 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4088-4108
```cpp
bool MasmParser::parseDirectiveEnds(StringRef Name, SMLoc NameLoc) {
  if (StructInProgress.empty())
    return Error(NameLoc, "ENDS directive without matching STRUC/STRUCT/UNION");
  if (StructInProgress.size() > 1)
    return Error(NameLoc, "unexpected name in nested ENDS directive");
  if (StructInProgress.back().Name.compare_insensitive(Name))
    return Error(NameLoc, "mismatched name in ENDS directive; expected '" +
                              StructInProgress.back().Name + "'");
  StructInfo Structure = StructInProgress.pop_back_val();
  // Pad to make the structure's size divisible by the smaller of its alignment
  // and the size of its largest field.
  Structure.Size = llvm::alignTo(
      Structure.Size, std::min(Structure.Alignment, Structure.AlignmentSize));
  Structs[Name.lower()] = std::move(Structure);

  if (parseEOL())
    return addErrorSuffix(" in ENDS directive");

  return false;
}

```
- **EN**: Implements logic around `parseDirectiveEnds`, `Error`, `back`, `pop_back_val`, and 4 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveEnds`, `Error`, `back`, `pop_back_val`, and 4 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4109-4135
```cpp
bool MasmParser::parseDirectiveNestedEnds() {
  if (StructInProgress.empty())
    return TokError("ENDS directive without matching STRUC/STRUCT/UNION");
  if (StructInProgress.size() == 1)
    return TokError("missing name in top-level ENDS directive");

  if (parseEOL())
    return addErrorSuffix(" in nested ENDS directive");

  StructInfo Structure = StructInProgress.pop_back_val();
  // Pad to make the structure's size divisible by its alignment.
  Structure.Size = llvm::alignTo(Structure.Size, Structure.Alignment);

  StructInfo &ParentStruct = StructInProgress.back();
  if (Structure.Name.empty()) {
    // Anonymous substructures' fields are addressed as if they belong to the
    // parent structure - so we transfer them to the parent here.
    const size_t OldFields = ParentStruct.Fields.size();
    ParentStruct.Fields.insert(
        ParentStruct.Fields.end(),
        std::make_move_iterator(Structure.Fields.begin()),
        std::make_move_iterator(Structure.Fields.end()));
    for (const auto &FieldByName : Structure.FieldsByName) {
      ParentStruct.FieldsByName[FieldByName.getKey()] =
          FieldByName.getValue() + OldFields;
    }

```
- **EN**: Implements logic around `parseDirectiveNestedEnds`, `TokError`, `addErrorSuffix`, `pop_back_val`, and 8 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveNestedEnds`, `TokError`, `addErrorSuffix`, `pop_back_val`, and 8 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4136-4162
```cpp
    unsigned FirstFieldOffset = 0;
    if (!Structure.Fields.empty() && !ParentStruct.IsUnion) {
      FirstFieldOffset = llvm::alignTo(
          ParentStruct.NextOffset,
          std::min(ParentStruct.Alignment, Structure.AlignmentSize));
    }

    if (ParentStruct.IsUnion) {
      ParentStruct.Size = std::max(ParentStruct.Size, Structure.Size);
    } else {
      for (auto &Field : llvm::drop_begin(ParentStruct.Fields, OldFields))
        Field.Offset += FirstFieldOffset;

      const unsigned StructureEnd = FirstFieldOffset + Structure.Size;
      if (!ParentStruct.IsUnion) {
        ParentStruct.NextOffset = StructureEnd;
      }
      ParentStruct.Size = std::max(ParentStruct.Size, StructureEnd);
    }
  } else {
    FieldInfo &Field = ParentStruct.addField(Structure.Name, FT_STRUCT,
                                             Structure.AlignmentSize);
    StructFieldInfo &StructInfo = Field.Contents.StructInfo;
    Field.Type = Structure.Size;
    Field.LengthOf = 1;
    Field.SizeOf = Structure.Size;

```
- **EN**: Implements logic around `alignTo`, `min`, `max`, `addField`.
- **CN**: 围绕 `alignTo`, `min`, `max`, `addField` 实现具体逻辑。

### Lines 4163-4189
```cpp
    const unsigned StructureEnd = Field.Offset + Field.SizeOf;
    if (!ParentStruct.IsUnion) {
      ParentStruct.NextOffset = StructureEnd;
    }
    ParentStruct.Size = std::max(ParentStruct.Size, StructureEnd);

    StructInfo.Structure = Structure;
    StructInfo.Initializers.emplace_back();
    auto &FieldInitializers = StructInfo.Initializers.back().FieldInitializers;
    for (const auto &SubField : Structure.Fields) {
      FieldInitializers.push_back(SubField.Contents);
    }
  }

  return false;
}

/// parseDirectiveOrg
///  ::= org expression
bool MasmParser::parseDirectiveOrg() {
  const MCExpr *Offset;
  SMLoc OffsetLoc = Lexer.getLoc();
  if (checkForValidSection() || parseExpression(Offset))
    return true;
  if (parseEOL())
    return addErrorSuffix(" in 'org' directive");

```
- **EN**: Implements logic around `max`, `emplace_back`, `back`, `push_back`, and 3 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `max`, `emplace_back`, `back`, `push_back`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 4190-4209
```cpp
  if (StructInProgress.empty()) {
    // Not in a struct; change the offset for the next instruction or data
    if (checkForValidSection())
      return addErrorSuffix(" in 'org' directive");

    getStreamer().emitValueToOffset(Offset, 0, OffsetLoc);
  } else {
    // Offset the next field of this struct
    StructInfo &Structure = StructInProgress.back();
    int64_t OffsetRes;
    if (!Offset->evaluateAsAbsolute(OffsetRes, getStreamer().getAssemblerPtr()))
      return Error(OffsetLoc,
                   "expected absolute expression in 'org' directive");
    if (OffsetRes < 0)
      return Error(
          OffsetLoc,
          "expected non-negative value in struct's 'org' directive; was " +
              std::to_string(OffsetRes));
    Structure.NextOffset = static_cast<unsigned>(OffsetRes);

```
- **EN**: Implements logic around `addErrorSuffix`, `getStreamer`, `back`, `Error`, and 2 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `addErrorSuffix`, `getStreamer`, `back`, `Error`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 4210-4229
```cpp
    // ORG-affected structures cannot be initialized
    Structure.Initializable = false;
  }

  return false;
}

bool MasmParser::emitAlignTo(int64_t Alignment) {
  if (StructInProgress.empty()) {
    // Not in a struct; align the next instruction or data
    if (checkForValidSection())
      return true;

    // Check whether we should use optimal code alignment for this align
    // directive.
    const MCSection *Section = getStreamer().getCurrentSectionOnly();
    if (MAI.useCodeAlign(*Section)) {
      getStreamer().emitCodeAlignment(Align(Alignment),
                                      &getTargetParser().getSTI(),
                                      /*MaxBytesToEmit=*/0);
```
- **EN**: Implements logic around `emitAlignTo`, `getStreamer`, `getTargetParser`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `emitAlignTo`, `getStreamer`, `getTargetParser` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 4230-4250
```cpp
    } else {
      // FIXME: Target specific behavior about how the "extra" bytes are filled.
      getStreamer().emitValueToAlignment(Align(Alignment), /*Value=*/0,
                                         /*ValueSize=*/1,
                                         /*MaxBytesToEmit=*/0);
    }
  } else {
    // Align the next field of this struct
    StructInfo &Structure = StructInProgress.back();
    Structure.NextOffset = llvm::alignTo(Structure.NextOffset, Alignment);
  }

  return false;
}

/// parseDirectiveAlign
///  ::= align expression
bool MasmParser::parseDirectiveAlign() {
  SMLoc AlignmentLoc = getLexer().getLoc();
  int64_t Alignment;

```
- **EN**: Implements logic around `getStreamer`, `back`, `alignTo`, `parseDirectiveAlign`, and 1 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `getStreamer`, `back`, `alignTo`, `parseDirectiveAlign`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 4251-4270
```cpp
  // Ignore empty 'align' directives.
  if (getTok().is(AsmToken::EndOfStatement)) {
    return Warning(AlignmentLoc,
                   "align directive with no operand is ignored") &&
           parseEOL();
  }
  if (parseAbsoluteExpression(Alignment) || parseEOL())
    return addErrorSuffix(" in align directive");

  // Always emit an alignment here even if we throw an error.
  bool ReturnVal = false;

  // Reject alignments that aren't either a power of two or zero, for ML.exe
  // compatibility. Alignment of zero is silently rounded up to one.
  if (Alignment == 0)
    Alignment = 1;
  if (!isPowerOf2_64(Alignment))
    ReturnVal |= Error(AlignmentLoc, "alignment must be a power of 2; was " +
                                         std::to_string(Alignment));

```
- **EN**: Implements logic around `Warning`, `parseEOL`, `addErrorSuffix`, `Error`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `Warning`, `parseEOL`, `addErrorSuffix`, `Error`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4271-4290
```cpp
  if (emitAlignTo(Alignment))
    ReturnVal |= addErrorSuffix(" in align directive");

  return ReturnVal;
}

/// parseDirectiveEven
///  ::= even
bool MasmParser::parseDirectiveEven() {
  if (parseEOL() || emitAlignTo(2))
    return addErrorSuffix(" in even directive");

  return false;
}

/// parseDirectiveMacro
/// ::= name macro [parameters]
///     ["LOCAL" identifiers]
///   parameters ::= parameter [, parameter]*
///   parameter ::= name ":" qualifier
```
- **EN**: Implements logic around `addErrorSuffix`, `parseDirectiveEven`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `addErrorSuffix`, `parseDirectiveEven` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4291-4312
```cpp
///   qualifier ::= "req" | "vararg" | "=" macro_argument
bool MasmParser::parseDirectiveMacro(StringRef Name, SMLoc NameLoc) {
  MCAsmMacroParameters Parameters;
  while (getLexer().isNot(AsmToken::EndOfStatement)) {
    if (!Parameters.empty() && Parameters.back().Vararg)
      return Error(Lexer.getLoc(),
                   "Vararg parameter '" + Parameters.back().Name +
                       "' should be last in the list of parameters");

    MCAsmMacroParameter Parameter;
    if (parseIdentifier(Parameter.Name))
      return TokError("expected identifier in 'macro' directive");

    // Emit an error if two (or more) named parameters share the same name.
    for (const MCAsmMacroParameter& CurrParam : Parameters)
      if (CurrParam.Name.equals_insensitive(Parameter.Name))
        return TokError("macro '" + Name + "' has multiple parameters"
                        " named '" + Parameter.Name + "'");

    if (Lexer.is(AsmToken::Colon)) {
      Lex();  // consume ':'

```
- **EN**: Implements logic around `parseDirectiveMacro`, `Error`, `back`, `TokError`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveMacro`, `Error`, `back`, `TokError`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4313-4341
```cpp
      if (parseOptionalToken(AsmToken::Equal)) {
        // Default value
        SMLoc ParamLoc;

        ParamLoc = Lexer.getLoc();
        if (parseMacroArgument(nullptr, Parameter.Value))
          return true;
      } else {
        SMLoc QualLoc;
        StringRef Qualifier;

        QualLoc = Lexer.getLoc();
        if (parseIdentifier(Qualifier))
          return Error(QualLoc, "missing parameter qualifier for "
                                "'" +
                                    Parameter.Name + "' in macro '" + Name +
                                    "'");

        if (Qualifier.equals_insensitive("req"))
          Parameter.Required = true;
        else if (Qualifier.equals_insensitive("vararg"))
          Parameter.Vararg = true;
        else
          return Error(QualLoc,
                       Qualifier + " is not a valid parameter qualifier for '" +
                           Parameter.Name + "' in macro '" + Name + "'");
      }
    }

```
- **EN**: Implements logic around `getLoc`, `Error`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `getLoc`, `Error` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4342-4361
```cpp
    Parameters.push_back(std::move(Parameter));

    if (getLexer().is(AsmToken::Comma))
      Lex();
  }

  // Eat just the end of statement.
  Lexer.Lex();

  std::vector<std::string> Locals;
  if (getTok().is(AsmToken::Identifier) &&
      getTok().getIdentifier().equals_insensitive("local")) {
    Lex(); // Eat the LOCAL directive.

    StringRef ID;
    while (true) {
      if (parseIdentifier(ID))
        return true;
      Locals.push_back(ID.lower());

```
- **EN**: Implements logic around `push_back`, `Lex`, `getTok`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `push_back`, `Lex`, `getTok` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4362-4383
```cpp
      // If we see a comma, continue (and allow line continuation).
      if (!parseOptionalToken(AsmToken::Comma))
        break;
      parseOptionalToken(AsmToken::EndOfStatement);
    }
  }

  // Consuming deferred text, so use Lexer.Lex to ignore Lexing Errors.
  AsmToken EndToken, StartToken = getTok();
  unsigned MacroDepth = 0;
  bool IsMacroFunction = false;
  // Lex the macro definition.
  while (true) {
    // Ignore Lexing errors in macros.
    while (Lexer.is(AsmToken::Error)) {
      Lexer.Lex();
    }

    // Check whether we have reached the end of the file.
    if (getLexer().is(AsmToken::Eof))
      return Error(NameLoc, "no matching 'endm' in definition");

```
- **EN**: Implements logic around `parseOptionalToken`, `getTok`, `Lex`, `Error`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseOptionalToken`, `getTok`, `Lex`, `Error` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4384-4409
```cpp
    // Otherwise, check whether we have reached the 'endm'... and determine if
    // this is a macro function.
    if (getLexer().is(AsmToken::Identifier)) {
      if (getTok().getIdentifier().equals_insensitive("endm")) {
        if (MacroDepth == 0) { // Outermost macro.
          EndToken = getTok();
          Lexer.Lex();
          if (getLexer().isNot(AsmToken::EndOfStatement))
            return TokError("unexpected token in '" + EndToken.getIdentifier() +
                            "' directive");
          break;
        } else {
          // Otherwise we just found the end of an inner macro.
          --MacroDepth;
        }
      } else if (getTok().getIdentifier().equals_insensitive("exitm")) {
        if (MacroDepth == 0 && peekTok().isNot(AsmToken::EndOfStatement)) {
          IsMacroFunction = true;
        }
      } else if (isMacroLikeDirective()) {
        // We allow nested macros. Those aren't instantiated until the
        // outermost macro is expanded so just ignore them for now.
        ++MacroDepth;
      }
    }

```
- **EN**: Implements logic around `getTok`, `Lex`, `TokError`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `getTok`, `Lex`, `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4410-4429
```cpp
    // Otherwise, scan til the end of the statement.
    eatToEndOfStatement();
  }

  if (getContext().lookupMacro(Name.lower())) {
    return Error(NameLoc, "macro '" + Name + "' is already defined");
  }

  const char *BodyStart = StartToken.getLoc().getPointer();
  const char *BodyEnd = EndToken.getLoc().getPointer();
  StringRef Body = StringRef(BodyStart, BodyEnd - BodyStart);
  MCAsmMacro Macro(Name, Body, std::move(Parameters), std::move(Locals),
                   IsMacroFunction);
  DEBUG_WITH_TYPE("asm-macros", dbgs() << "Defining new macro:\n";
                  Macro.dump());
  getContext().defineMacro(Name.lower(), std::move(Macro));
  return false;
}

/// parseDirectiveExitMacro
```
- **EN**: Implements logic around `eatToEndOfStatement`, `Error`, `getLoc`, `StringRef`, and 4 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `eatToEndOfStatement`, `Error`, `getLoc`, `StringRef`, and 4 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4430-4449
```cpp
/// ::= "exitm" [textitem]
bool MasmParser::parseDirectiveExitMacro(SMLoc DirectiveLoc,
                                         StringRef Directive,
                                         std::string &Value) {
  SMLoc EndLoc = getTok().getLoc();
  if (getTok().isNot(AsmToken::EndOfStatement) && parseTextItem(Value))
    return Error(EndLoc,
                 "unable to parse text item in '" + Directive + "' directive");
  eatToEndOfStatement();

  if (!isInsideMacroInstantiation())
    return TokError("unexpected '" + Directive + "' in file, "
                                                 "no current macro definition");

  // Exit all conditionals that are active in the current macro.
  while (TheCondStack.size() != ActiveMacros.back()->CondStackDepth) {
    TheCondState = TheCondStack.back();
    TheCondStack.pop_back();
  }

```
- **EN**: Implements logic around `parseDirectiveExitMacro`, `getTok`, `Error`, `eatToEndOfStatement`, and 3 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveExitMacro`, `getTok`, `Error`, `eatToEndOfStatement`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4450-4472
```cpp
  handleMacroExit();
  return false;
}

/// parseDirectiveEndMacro
/// ::= endm
bool MasmParser::parseDirectiveEndMacro(StringRef Directive) {
  if (getLexer().isNot(AsmToken::EndOfStatement))
    return TokError("unexpected token in '" + Directive + "' directive");

  // If we are inside a macro instantiation, terminate the current
  // instantiation.
  if (isInsideMacroInstantiation()) {
    handleMacroExit();
    return false;
  }

  // Otherwise, this .endmacro is a stray entry in the file; well formed
  // .endmacro directives are handled during the macro definition parsing.
  return TokError("unexpected '" + Directive + "' in file, "
                                               "no current macro definition");
}

```
- **EN**: Implements logic around `handleMacroExit`, `parseDirectiveEndMacro`, `TokError`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `handleMacroExit`, `parseDirectiveEndMacro`, `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4473-4494
```cpp
/// parseDirectivePurgeMacro
/// ::= purge identifier ( , identifier )*
bool MasmParser::parseDirectivePurgeMacro(SMLoc DirectiveLoc) {
  StringRef Name;
  while (true) {
    SMLoc NameLoc;
    if (parseTokenLoc(NameLoc) ||
        check(parseIdentifier(Name), NameLoc,
              "expected identifier in 'purge' directive"))
      return true;

    DEBUG_WITH_TYPE("asm-macros", dbgs()
                                      << "Un-defining macro: " << Name << "\n");
    if (!getContext().lookupMacro(Name.lower()))
      return Error(NameLoc, "macro '" + Name + "' is not defined");
    getContext().undefineMacro(Name.lower());

    if (!parseOptionalToken(AsmToken::Comma))
      break;
    parseOptionalToken(AsmToken::EndOfStatement);
  }

```
- **EN**: Implements logic around `parseDirectivePurgeMacro`, `check`, `DEBUG_WITH_TYPE`, `Error`, and 2 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectivePurgeMacro`, `check`, `DEBUG_WITH_TYPE`, `Error`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4495-4518
```cpp
  return false;
}

bool MasmParser::parseDirectiveExtern() {
  // .extern is the default - but we still need to take any provided type info.
  auto parseOp = [&]() -> bool {
    MCSymbol *Sym;
    SMLoc NameLoc = getTok().getLoc();
    if (parseSymbol(Sym))
      return Error(NameLoc, "expected name");
    if (parseToken(AsmToken::Colon))
      return true;

    StringRef TypeName;
    SMLoc TypeLoc = getTok().getLoc();
    if (parseIdentifier(TypeName))
      return Error(TypeLoc, "expected type");
    if (!TypeName.equals_insensitive("proc")) {
      AsmTypeInfo Type;
      if (lookUpType(TypeName, Type))
        return Error(TypeLoc, "unrecognized type");
      KnownType[Sym->getName().lower()] = Type;
    }

```
- **EN**: Implements logic around `parseDirectiveExtern`, `getTok`, `Error`, `getName`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveExtern`, `getTok`, `Error`, `getName` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 4519-4538
```cpp
    static_cast<MCSymbolCOFF *>(Sym)->setExternal(true);
    getStreamer().emitSymbolAttribute(Sym, MCSA_Extern);

    return false;
  };

  if (parseMany(parseOp))
    return addErrorSuffix(" in directive 'extern'");
  return false;
}

/// parseDirectiveSymbolAttribute
///  ::= { ".globl", ".weak", ... } [ identifier ( , identifier )* ]
bool MasmParser::parseDirectiveSymbolAttribute(MCSymbolAttr Attr) {
  auto parseOp = [&]() -> bool {
    SMLoc Loc = getTok().getLoc();
    MCSymbol *Sym;
    if (parseSymbol(Sym))
      return Error(Loc, "expected identifier");

```
- **EN**: Implements logic around `setExternal`, `getStreamer`, `addErrorSuffix`, `parseDirectiveSymbolAttribute`, and 2 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `setExternal`, `getStreamer`, `addErrorSuffix`, `parseDirectiveSymbolAttribute`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 4539-4558
```cpp
    // Assembler local symbols don't make any sense here. Complain loudly.
    if (Sym->isTemporary())
      return Error(Loc, "non-local symbol required");

    if (!getStreamer().emitSymbolAttribute(Sym, Attr))
      return Error(Loc, "unable to emit symbol attribute");
    return false;
  };

  if (parseMany(parseOp))
    return addErrorSuffix(" in directive");
  return false;
}

/// parseDirectiveComm
///  ::= ( .comm | .lcomm ) identifier , size_expression [ , align_expression ]
bool MasmParser::parseDirectiveComm(bool IsLocal) {
  if (checkForValidSection())
    return true;

```
- **EN**: Implements logic around `Error`, `addErrorSuffix`, `parseDirectiveComm`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `Error`, `addErrorSuffix`, `parseDirectiveComm` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 4559-4580
```cpp
  SMLoc IDLoc = getLexer().getLoc();
  MCSymbol *Sym;
  if (parseSymbol(Sym))
    return TokError("expected identifier in directive");

  if (getLexer().isNot(AsmToken::Comma))
    return TokError("unexpected token in directive");
  Lex();

  int64_t Size;
  SMLoc SizeLoc = getLexer().getLoc();
  if (parseAbsoluteExpression(Size))
    return true;

  int64_t Pow2Alignment = 0;
  SMLoc Pow2AlignmentLoc;
  if (getLexer().is(AsmToken::Comma)) {
    Lex();
    Pow2AlignmentLoc = getLexer().getLoc();
    if (parseAbsoluteExpression(Pow2Alignment))
      return true;

```
- **EN**: Implements logic around `getLexer`, `TokError`, `Lex`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getLexer`, `TokError`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 4581-4602
```cpp
    LCOMM::LCOMMType LCOMM = Lexer.getMAI().getLCOMMDirectiveAlignmentType();
    if (IsLocal && LCOMM == LCOMM::NoAlignment)
      return Error(Pow2AlignmentLoc, "alignment not supported on this target");

    // If this target takes alignments in bytes (not log) validate and convert.
    if ((!IsLocal && Lexer.getMAI().getCOMMDirectiveAlignmentIsInBytes()) ||
        (IsLocal && LCOMM == LCOMM::ByteAlignment)) {
      if (!isPowerOf2_64(Pow2Alignment))
        return Error(Pow2AlignmentLoc, "alignment must be a power of 2");
      Pow2Alignment = Log2_64(Pow2Alignment);
    }
  }

  if (parseEOL())
    return true;

  // NOTE: a size of zero for a .comm should create a undefined symbol
  // but a size of .lcomm creates a bss symbol of size zero.
  if (Size < 0)
    return Error(SizeLoc, "invalid '.comm' or '.lcomm' directive size, can't "
                          "be less than zero");

```
- **EN**: Implements logic around `getMAI`, `Error`, `Log2_64`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getMAI`, `Error`, `Log2_64` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 4603-4624
```cpp
  // NOTE: The alignment in the directive is a power of 2 value, the assembler
  // may internally end up wanting an alignment in bytes.
  // FIXME: Diagnose overflow.
  if (Pow2Alignment < 0)
    return Error(Pow2AlignmentLoc, "invalid '.comm' or '.lcomm' directive "
                                   "alignment, can't be less than zero");

  Sym->redefineIfPossible();
  if (!Sym->isUndefined())
    return Error(IDLoc, "invalid symbol redefinition");

  // Create the Symbol as a common or local common with Size and Pow2Alignment.
  if (IsLocal) {
    getStreamer().emitLocalCommonSymbol(Sym, Size,
                                        Align(1ULL << Pow2Alignment));
    return false;
  }

  getStreamer().emitCommonSymbol(Sym, Size, Align(1ULL << Pow2Alignment));
  return false;
}

```
- **EN**: Implements logic around `Error`, `redefineIfPossible`, `getStreamer`, `Align`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `Error`, `redefineIfPossible`, `getStreamer`, `Align` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 4625-4644
```cpp
/// parseDirectiveComment
///  ::= comment delimiter [[text]]
///              [[text]]
///              [[text]] delimiter [[text]]
bool MasmParser::parseDirectiveComment(SMLoc DirectiveLoc) {
  std::string FirstLine = parseStringTo(AsmToken::EndOfStatement);
  size_t DelimiterEnd = FirstLine.find_first_of("\b\t\v\f\r\x1A ");
  assert(DelimiterEnd != std::string::npos);
  StringRef Delimiter = StringRef(FirstLine).take_front(DelimiterEnd);
  if (Delimiter.empty())
    return Error(DirectiveLoc, "no delimiter in 'comment' directive");
  do {
    if (getTok().is(AsmToken::Eof))
      return Error(DirectiveLoc, "unmatched delimiter in 'comment' directive");
    Lex();  // eat end of statement
  } while (
      !StringRef(parseStringTo(AsmToken::EndOfStatement)).contains(Delimiter));
  return parseEOL();
}

```
- **EN**: Implements logic around `parseDirectiveComment`, `parseStringTo`, `find_first_of`, `assert`, and 4 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveComment`, `parseStringTo`, `find_first_of`, `assert`, and 4 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4645-4666
```cpp
/// parseDirectiveInclude
///  ::= include <filename>
///    | include filename
bool MasmParser::parseDirectiveInclude() {
  // Allow the strings to have escaped octal character sequence.
  std::string Filename;
  SMLoc IncludeLoc = getTok().getLoc();

  if (parseAngleBracketString(Filename))
    Filename = parseStringTo(AsmToken::EndOfStatement);
  if (check(Filename.empty(), "missing filename in 'include' directive") ||
      check(getTok().isNot(AsmToken::EndOfStatement),
            "unexpected token in 'include' directive") ||
      // Attempt to switch the lexer to the included file before consuming the
      // end of statement to avoid losing it when we switch.
      check(enterIncludeFile(Filename), IncludeLoc,
            "Could not find include file '" + Filename + "'"))
    return true;

  return false;
}

```
- **EN**: Implements logic around `parseDirectiveInclude`, `getTok`, `parseStringTo`, `check`; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveInclude`, `getTok`, `parseStringTo`, `check` 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 4667-4688
```cpp
/// parseDirectiveIf
/// ::= .if{,eq,ge,gt,le,lt,ne} expression
bool MasmParser::parseDirectiveIf(SMLoc DirectiveLoc, DirectiveKind DirKind) {
  TheCondStack.push_back(TheCondState);
  TheCondState.TheCond = AsmCond::IfCond;
  if (TheCondState.Ignore) {
    eatToEndOfStatement();
  } else {
    int64_t ExprValue;
    if (parseAbsoluteExpression(ExprValue) || parseEOL())
      return true;

    switch (DirKind) {
    default:
      llvm_unreachable("unsupported directive");
    case DK_IF:
      break;
    case DK_IFE:
      ExprValue = ExprValue == 0;
      break;
    }

```
- **EN**: Implements logic around `parseDirectiveIf`, `push_back`, `eatToEndOfStatement`, `llvm_unreachable`; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveIf`, `push_back`, `eatToEndOfStatement`, `llvm_unreachable` 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 4689-4708
```cpp
    TheCondState.CondMet = ExprValue;
    TheCondState.Ignore = !TheCondState.CondMet;
  }

  return false;
}

/// parseDirectiveIfb
/// ::= .ifb textitem
bool MasmParser::parseDirectiveIfb(SMLoc DirectiveLoc, bool ExpectBlank) {
  TheCondStack.push_back(TheCondState);
  TheCondState.TheCond = AsmCond::IfCond;

  if (TheCondState.Ignore) {
    eatToEndOfStatement();
  } else {
    std::string Str;
    if (parseTextItem(Str))
      return TokError("expected text item parameter for 'ifb' directive");

```
- **EN**: Implements logic around `parseDirectiveIfb`, `push_back`, `eatToEndOfStatement`, `TokError`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveIfb`, `push_back`, `eatToEndOfStatement`, `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4709-4730
```cpp
    if (parseEOL())
      return true;

    TheCondState.CondMet = ExpectBlank == Str.empty();
    TheCondState.Ignore = !TheCondState.CondMet;
  }

  return false;
}

/// parseDirectiveIfidn
///   ::= ifidn textitem, textitem
bool MasmParser::parseDirectiveIfidn(SMLoc DirectiveLoc, bool ExpectEqual,
                                     bool CaseInsensitive) {
  std::string String1, String2;

  if (parseTextItem(String1)) {
    if (ExpectEqual)
      return TokError("expected text item parameter for 'ifidn' directive");
    return TokError("expected text item parameter for 'ifdif' directive");
  }

```
- **EN**: Implements logic around `empty`, `parseDirectiveIfidn`, `TokError`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `empty`, `parseDirectiveIfidn`, `TokError` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4731-4753
```cpp
  if (Lexer.isNot(AsmToken::Comma)) {
    if (ExpectEqual)
      return TokError(
          "expected comma after first string for 'ifidn' directive");
    return TokError("expected comma after first string for 'ifdif' directive");
  }
  Lex();

  if (parseTextItem(String2)) {
    if (ExpectEqual)
      return TokError("expected text item parameter for 'ifidn' directive");
    return TokError("expected text item parameter for 'ifdif' directive");
  }

  TheCondStack.push_back(TheCondState);
  TheCondState.TheCond = AsmCond::IfCond;
  if (CaseInsensitive)
    TheCondState.CondMet =
        ExpectEqual == (StringRef(String1).equals_insensitive(String2));
  else
    TheCondState.CondMet = ExpectEqual == (String1 == String2);
  TheCondState.Ignore = !TheCondState.CondMet;

```
- **EN**: Implements logic around `TokError`, `Lex`, `push_back`, `StringRef`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `Lex`, `push_back`, `StringRef` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4754-4777
```cpp
  return false;
}

/// parseDirectiveIfdef
/// ::= ifdef symbol
///   | ifdef variable
bool MasmParser::parseDirectiveIfdef(SMLoc DirectiveLoc, bool expect_defined) {
  TheCondStack.push_back(TheCondState);
  TheCondState.TheCond = AsmCond::IfCond;

  if (TheCondState.Ignore) {
    eatToEndOfStatement();
  } else {
    bool is_defined = false;
    MCRegister Reg;
    SMLoc StartLoc, EndLoc;
    is_defined =
        getTargetParser().tryParseRegister(Reg, StartLoc, EndLoc).isSuccess();
    if (!is_defined) {
      StringRef Name;
      if (check(parseIdentifier(Name), "expected identifier after 'ifdef'") ||
          parseEOL())
        return true;

```
- **EN**: Implements logic around `parseDirectiveIfdef`, `push_back`, `eatToEndOfStatement`, `getTargetParser`, and 1 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveIfdef`, `push_back`, `eatToEndOfStatement`, `getTargetParser`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 4778-4804
```cpp
      if (BuiltinSymbolMap.contains(Name.lower())) {
        is_defined = true;
      } else if (Variables.contains(Name.lower())) {
        is_defined = true;
      } else {
        MCSymbol *Sym = getContext().lookupSymbol(Name.lower());
        is_defined = (Sym && !Sym->isUndefined());
      }
    }

    TheCondState.CondMet = (is_defined == expect_defined);
    TheCondState.Ignore = !TheCondState.CondMet;
  }

  return false;
}

/// parseDirectiveElseIf
/// ::= elseif expression
bool MasmParser::parseDirectiveElseIf(SMLoc DirectiveLoc,
                                      DirectiveKind DirKind) {
  if (TheCondState.TheCond != AsmCond::IfCond &&
      TheCondState.TheCond != AsmCond::ElseIfCond)
    return Error(DirectiveLoc, "Encountered a .elseif that doesn't follow an"
                               " .if or  an .elseif");
  TheCondState.TheCond = AsmCond::ElseIfCond;

```
- **EN**: Implements logic around `getContext`, `isUndefined`, `parseDirectiveElseIf`, `Error`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getContext`, `isUndefined`, `parseDirectiveElseIf`, `Error` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 4805-4828
```cpp
  bool LastIgnoreState = false;
  if (!TheCondStack.empty())
    LastIgnoreState = TheCondStack.back().Ignore;
  if (LastIgnoreState || TheCondState.CondMet) {
    TheCondState.Ignore = true;
    eatToEndOfStatement();
  } else {
    int64_t ExprValue;
    if (parseAbsoluteExpression(ExprValue))
      return true;

    if (parseEOL())
      return true;

    switch (DirKind) {
    default:
      llvm_unreachable("unsupported directive");
    case DK_ELSEIF:
      break;
    case DK_ELSEIFE:
      ExprValue = ExprValue == 0;
      break;
    }

```
- **EN**: Implements logic around `back`, `eatToEndOfStatement`, `llvm_unreachable`; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `back`, `eatToEndOfStatement`, `llvm_unreachable` 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 4829-4858
```cpp
    TheCondState.CondMet = ExprValue;
    TheCondState.Ignore = !TheCondState.CondMet;
  }

  return false;
}

/// parseDirectiveElseIfb
/// ::= elseifb textitem
bool MasmParser::parseDirectiveElseIfb(SMLoc DirectiveLoc, bool ExpectBlank) {
  if (TheCondState.TheCond != AsmCond::IfCond &&
      TheCondState.TheCond != AsmCond::ElseIfCond)
    return Error(DirectiveLoc, "Encountered an elseif that doesn't follow an"
                               " if or an elseif");
  TheCondState.TheCond = AsmCond::ElseIfCond;

  bool LastIgnoreState = false;
  if (!TheCondStack.empty())
    LastIgnoreState = TheCondStack.back().Ignore;
  if (LastIgnoreState || TheCondState.CondMet) {
    TheCondState.Ignore = true;
    eatToEndOfStatement();
  } else {
    std::string Str;
    if (parseTextItem(Str)) {
      if (ExpectBlank)
        return TokError("expected text item parameter for 'elseifb' directive");
      return TokError("expected text item parameter for 'elseifnb' directive");
    }

```
- **EN**: Implements logic around `parseDirectiveElseIfb`, `Error`, `back`, `eatToEndOfStatement`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveElseIfb`, `Error`, `back`, `eatToEndOfStatement`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4859-4879
```cpp
    if (parseEOL())
      return true;

    TheCondState.CondMet = ExpectBlank == Str.empty();
    TheCondState.Ignore = !TheCondState.CondMet;
  }

  return false;
}

/// parseDirectiveElseIfdef
/// ::= elseifdef symbol
///   | elseifdef variable
bool MasmParser::parseDirectiveElseIfdef(SMLoc DirectiveLoc,
                                         bool expect_defined) {
  if (TheCondState.TheCond != AsmCond::IfCond &&
      TheCondState.TheCond != AsmCond::ElseIfCond)
    return Error(DirectiveLoc, "Encountered an elseif that doesn't follow an"
                               " if or an elseif");
  TheCondState.TheCond = AsmCond::ElseIfCond;

```
- **EN**: Implements logic around `empty`, `parseDirectiveElseIfdef`, `Error`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `empty`, `parseDirectiveElseIfdef`, `Error` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 4880-4908
```cpp
  bool LastIgnoreState = false;
  if (!TheCondStack.empty())
    LastIgnoreState = TheCondStack.back().Ignore;
  if (LastIgnoreState || TheCondState.CondMet) {
    TheCondState.Ignore = true;
    eatToEndOfStatement();
  } else {
    bool is_defined = false;
    MCRegister Reg;
    SMLoc StartLoc, EndLoc;
    is_defined =
        getTargetParser().tryParseRegister(Reg, StartLoc, EndLoc).isSuccess();
    if (!is_defined) {
      StringRef Name;
      if (check(parseIdentifier(Name),
                "expected identifier after 'elseifdef'") ||
          parseEOL())
        return true;

      if (BuiltinSymbolMap.contains(Name.lower())) {
        is_defined = true;
      } else if (Variables.contains(Name.lower())) {
        is_defined = true;
      } else {
        MCSymbol *Sym = getContext().lookupSymbol(Name);
        is_defined = (Sym && !Sym->isUndefined());
      }
    }

```
- **EN**: Implements logic around `back`, `eatToEndOfStatement`, `getTargetParser`, `parseEOL`, and 2 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `back`, `eatToEndOfStatement`, `getTargetParser`, `parseEOL`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 4909-4934
```cpp
    TheCondState.CondMet = (is_defined == expect_defined);
    TheCondState.Ignore = !TheCondState.CondMet;
  }

  return false;
}

/// parseDirectiveElseIfidn
/// ::= elseifidn textitem, textitem
bool MasmParser::parseDirectiveElseIfidn(SMLoc DirectiveLoc, bool ExpectEqual,
                                         bool CaseInsensitive) {
  if (TheCondState.TheCond != AsmCond::IfCond &&
      TheCondState.TheCond != AsmCond::ElseIfCond)
    return Error(DirectiveLoc, "Encountered an elseif that doesn't follow an"
                               " if or an elseif");
  TheCondState.TheCond = AsmCond::ElseIfCond;

  bool LastIgnoreState = false;
  if (!TheCondStack.empty())
    LastIgnoreState = TheCondStack.back().Ignore;
  if (LastIgnoreState || TheCondState.CondMet) {
    TheCondState.Ignore = true;
    eatToEndOfStatement();
  } else {
    std::string String1, String2;

```
- **EN**: Implements logic around `parseDirectiveElseIfidn`, `Error`, `back`, `eatToEndOfStatement`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveElseIfidn`, `Error`, `back`, `eatToEndOfStatement` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4935-4957
```cpp
    if (parseTextItem(String1)) {
      if (ExpectEqual)
        return TokError(
            "expected text item parameter for 'elseifidn' directive");
      return TokError("expected text item parameter for 'elseifdif' directive");
    }

    if (Lexer.isNot(AsmToken::Comma)) {
      if (ExpectEqual)
        return TokError(
            "expected comma after first string for 'elseifidn' directive");
      return TokError(
          "expected comma after first string for 'elseifdif' directive");
    }
    Lex();

    if (parseTextItem(String2)) {
      if (ExpectEqual)
        return TokError(
            "expected text item parameter for 'elseifidn' directive");
      return TokError("expected text item parameter for 'elseifdif' directive");
    }

```
- **EN**: Implements logic around `TokError`, `Lex`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4958-4987
```cpp
    if (CaseInsensitive)
      TheCondState.CondMet =
          ExpectEqual == (StringRef(String1).equals_insensitive(String2));
    else
      TheCondState.CondMet = ExpectEqual == (String1 == String2);
    TheCondState.Ignore = !TheCondState.CondMet;
  }

  return false;
}

/// parseDirectiveElse
/// ::= else
bool MasmParser::parseDirectiveElse(SMLoc DirectiveLoc) {
  if (parseEOL())
    return true;

  if (TheCondState.TheCond != AsmCond::IfCond &&
      TheCondState.TheCond != AsmCond::ElseIfCond)
    return Error(DirectiveLoc, "Encountered an else that doesn't follow an if"
                               " or an elseif");
  TheCondState.TheCond = AsmCond::ElseCond;
  bool LastIgnoreState = false;
  if (!TheCondStack.empty())
    LastIgnoreState = TheCondStack.back().Ignore;
  if (LastIgnoreState || TheCondState.CondMet)
    TheCondState.Ignore = true;
  else
    TheCondState.Ignore = false;

```
- **EN**: Implements logic around `StringRef`, `parseDirectiveElse`, `Error`, `back`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `StringRef`, `parseDirectiveElse`, `Error`, `back` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 4988-5012
```cpp
  return false;
}

/// parseDirectiveEnd
/// ::= end
bool MasmParser::parseDirectiveEnd(SMLoc DirectiveLoc) {
  if (parseEOL())
    return true;

  while (Lexer.isNot(AsmToken::Eof))
    Lexer.Lex();

  return false;
}

/// parseDirectiveError
///   ::= .err [message]
bool MasmParser::parseDirectiveError(SMLoc DirectiveLoc) {
  if (!TheCondStack.empty()) {
    if (TheCondStack.back().Ignore) {
      eatToEndOfStatement();
      return false;
    }
  }

```
- **EN**: Implements logic around `parseDirectiveEnd`, `Lex`, `parseDirectiveError`, `eatToEndOfStatement`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveEnd`, `Lex`, `parseDirectiveError`, `eatToEndOfStatement` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 5013-5034
```cpp
  std::string Message = ".err directive invoked in source file";
  if (Lexer.isNot(AsmToken::EndOfStatement))
    Message = parseStringTo(AsmToken::EndOfStatement);
  Lex();

  return Error(DirectiveLoc, Message);
}

/// parseDirectiveErrorIfb
///   ::= .errb textitem[, message]
bool MasmParser::parseDirectiveErrorIfb(SMLoc DirectiveLoc, bool ExpectBlank) {
  if (!TheCondStack.empty()) {
    if (TheCondStack.back().Ignore) {
      eatToEndOfStatement();
      return false;
    }
  }

  std::string Text;
  if (parseTextItem(Text))
    return Error(getTok().getLoc(), "missing text item in '.errb' directive");

```
- **EN**: Implements logic around `parseStringTo`, `Lex`, `Error`, `parseDirectiveErrorIfb`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseStringTo`, `Lex`, `Error`, `parseDirectiveErrorIfb`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 5035-5058
```cpp
  std::string Message = ".errb directive invoked in source file";
  if (Lexer.isNot(AsmToken::EndOfStatement)) {
    if (parseToken(AsmToken::Comma))
      return addErrorSuffix(" in '.errb' directive");
    Message = parseStringTo(AsmToken::EndOfStatement);
  }
  Lex();

  if (Text.empty() == ExpectBlank)
    return Error(DirectiveLoc, Message);
  return false;
}

/// parseDirectiveErrorIfdef
///   ::= .errdef name[, message]
bool MasmParser::parseDirectiveErrorIfdef(SMLoc DirectiveLoc,
                                          bool ExpectDefined) {
  if (!TheCondStack.empty()) {
    if (TheCondStack.back().Ignore) {
      eatToEndOfStatement();
      return false;
    }
  }

```
- **EN**: Implements logic around `addErrorSuffix`, `parseStringTo`, `Lex`, `Error`, and 2 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `addErrorSuffix`, `parseStringTo`, `Lex`, `Error`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 5059-5078
```cpp
  bool IsDefined = false;
  MCRegister Reg;
  SMLoc StartLoc, EndLoc;
  IsDefined =
      getTargetParser().tryParseRegister(Reg, StartLoc, EndLoc).isSuccess();
  if (!IsDefined) {
    StringRef Name;
    if (check(parseIdentifier(Name), "expected identifier after '.errdef'"))
      return true;

    if (BuiltinSymbolMap.contains(Name.lower())) {
      IsDefined = true;
    } else if (Variables.contains(Name.lower())) {
      IsDefined = true;
    } else {
      MCSymbol *Sym = getContext().lookupSymbol(Name);
      IsDefined = (Sym && !Sym->isUndefined());
    }
  }

```
- **EN**: Implements logic around `getTargetParser`, `getContext`, `isUndefined`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getTargetParser`, `getContext`, `isUndefined` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 5079-5102
```cpp
  std::string Message = ".errdef directive invoked in source file";
  if (Lexer.isNot(AsmToken::EndOfStatement)) {
    if (parseToken(AsmToken::Comma))
      return addErrorSuffix(" in '.errdef' directive");
    Message = parseStringTo(AsmToken::EndOfStatement);
  }
  Lex();

  if (IsDefined == ExpectDefined)
    return Error(DirectiveLoc, Message);
  return false;
}

/// parseDirectiveErrorIfidn
///   ::= .erridn textitem, textitem[, message]
bool MasmParser::parseDirectiveErrorIfidn(SMLoc DirectiveLoc, bool ExpectEqual,
                                          bool CaseInsensitive) {
  if (!TheCondStack.empty()) {
    if (TheCondStack.back().Ignore) {
      eatToEndOfStatement();
      return false;
    }
  }

```
- **EN**: Implements logic around `addErrorSuffix`, `parseStringTo`, `Lex`, `Error`, and 2 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `addErrorSuffix`, `parseStringTo`, `Lex`, `Error`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 5103-5125
```cpp
  std::string String1, String2;

  if (parseTextItem(String1)) {
    if (ExpectEqual)
      return TokError("expected string parameter for '.erridn' directive");
    return TokError("expected string parameter for '.errdif' directive");
  }

  if (Lexer.isNot(AsmToken::Comma)) {
    if (ExpectEqual)
      return TokError(
          "expected comma after first string for '.erridn' directive");
    return TokError(
        "expected comma after first string for '.errdif' directive");
  }
  Lex();

  if (parseTextItem(String2)) {
    if (ExpectEqual)
      return TokError("expected string parameter for '.erridn' directive");
    return TokError("expected string parameter for '.errdif' directive");
  }

```
- **EN**: Implements logic around `TokError`, `Lex`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `TokError`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 5126-5151
```cpp
  std::string Message;
  if (ExpectEqual)
    Message = ".erridn directive invoked in source file";
  else
    Message = ".errdif directive invoked in source file";
  if (Lexer.isNot(AsmToken::EndOfStatement)) {
    if (parseToken(AsmToken::Comma))
      return addErrorSuffix(" in '.erridn' directive");
    Message = parseStringTo(AsmToken::EndOfStatement);
  }
  Lex();

  if (CaseInsensitive)
    TheCondState.CondMet =
        ExpectEqual == (StringRef(String1).equals_insensitive(String2));
  else
    TheCondState.CondMet = ExpectEqual == (String1 == String2);
  TheCondState.Ignore = !TheCondState.CondMet;

  if ((CaseInsensitive &&
       ExpectEqual == StringRef(String1).equals_insensitive(String2)) ||
      (ExpectEqual == (String1 == String2)))
    return Error(DirectiveLoc, Message);
  return false;
}

```
- **EN**: Implements logic around `addErrorSuffix`, `parseStringTo`, `Lex`, `StringRef`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `addErrorSuffix`, `parseStringTo`, `Lex`, `StringRef`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 5152-5173
```cpp
/// parseDirectiveErrorIfe
///   ::= .erre expression[, message]
bool MasmParser::parseDirectiveErrorIfe(SMLoc DirectiveLoc, bool ExpectZero) {
  if (!TheCondStack.empty()) {
    if (TheCondStack.back().Ignore) {
      eatToEndOfStatement();
      return false;
    }
  }

  int64_t ExprValue;
  if (parseAbsoluteExpression(ExprValue))
    return addErrorSuffix(" in '.erre' directive");

  std::string Message = ".erre directive invoked in source file";
  if (Lexer.isNot(AsmToken::EndOfStatement)) {
    if (parseToken(AsmToken::Comma))
      return addErrorSuffix(" in '.erre' directive");
    Message = parseStringTo(AsmToken::EndOfStatement);
  }
  Lex();

```
- **EN**: Implements logic around `parseDirectiveErrorIfe`, `eatToEndOfStatement`, `addErrorSuffix`, `parseStringTo`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveErrorIfe`, `eatToEndOfStatement`, `addErrorSuffix`, `parseStringTo`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 5174-5195
```cpp
  if ((ExprValue == 0) == ExpectZero)
    return Error(DirectiveLoc, Message);
  return false;
}

/// parseDirectiveEndIf
/// ::= .endif
bool MasmParser::parseDirectiveEndIf(SMLoc DirectiveLoc) {
  if (parseEOL())
    return true;

  if ((TheCondState.TheCond == AsmCond::NoCond) || TheCondStack.empty())
    return Error(DirectiveLoc, "Encountered a .endif that doesn't follow "
                               "an .if or .else");
  if (!TheCondStack.empty()) {
    TheCondState = TheCondStack.back();
    TheCondStack.pop_back();
  }

  return false;
}

```
- **EN**: Implements logic around `Error`, `parseDirectiveEndIf`, `back`, `pop_back`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `Error`, `parseDirectiveEndIf`, `back`, `pop_back` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 5196-5235
```cpp
void MasmParser::initializeDirectiveKindMap() {
  DirectiveKindMap["="] = DK_ASSIGN;
  DirectiveKindMap["equ"] = DK_EQU;
  DirectiveKindMap["textequ"] = DK_TEXTEQU;
  // DirectiveKindMap[".ascii"] = DK_ASCII;
  // DirectiveKindMap[".asciz"] = DK_ASCIZ;
  // DirectiveKindMap[".string"] = DK_STRING;
  DirectiveKindMap["byte"] = DK_BYTE;
  DirectiveKindMap["sbyte"] = DK_SBYTE;
  DirectiveKindMap["word"] = DK_WORD;
  DirectiveKindMap["sword"] = DK_SWORD;
  DirectiveKindMap["dword"] = DK_DWORD;
  DirectiveKindMap["sdword"] = DK_SDWORD;
  DirectiveKindMap["fword"] = DK_FWORD;
  DirectiveKindMap["qword"] = DK_QWORD;
  DirectiveKindMap["sqword"] = DK_SQWORD;
  DirectiveKindMap["real4"] = DK_REAL4;
  DirectiveKindMap["real8"] = DK_REAL8;
  DirectiveKindMap["real10"] = DK_REAL10;
  DirectiveKindMap["align"] = DK_ALIGN;
  DirectiveKindMap["even"] = DK_EVEN;
  DirectiveKindMap["org"] = DK_ORG;
  DirectiveKindMap["extern"] = DK_EXTERN;
  DirectiveKindMap["extrn"] = DK_EXTERN;
  DirectiveKindMap["public"] = DK_PUBLIC;
  // DirectiveKindMap[".comm"] = DK_COMM;
  DirectiveKindMap["comment"] = DK_COMMENT;
  DirectiveKindMap["include"] = DK_INCLUDE;
  DirectiveKindMap["repeat"] = DK_REPEAT;
  DirectiveKindMap["rept"] = DK_REPEAT;
  DirectiveKindMap["while"] = DK_WHILE;
  DirectiveKindMap["for"] = DK_FOR;
  DirectiveKindMap["irp"] = DK_FOR;
  DirectiveKindMap["forc"] = DK_FORC;
  DirectiveKindMap["irpc"] = DK_FORC;
  DirectiveKindMap["if"] = DK_IF;
  DirectiveKindMap["ife"] = DK_IFE;
  DirectiveKindMap["ifb"] = DK_IFB;
  DirectiveKindMap["ifnb"] = DK_IFNB;
  DirectiveKindMap["ifdef"] = DK_IFDEF;
```
- **EN**: Implements logic around `initializeDirectiveKindMap`; this block parses assembly syntax or operands.
- **CN**: 围绕 `initializeDirectiveKindMap` 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 5236-5275
```cpp
  DirectiveKindMap["ifndef"] = DK_IFNDEF;
  DirectiveKindMap["ifdif"] = DK_IFDIF;
  DirectiveKindMap["ifdifi"] = DK_IFDIFI;
  DirectiveKindMap["ifidn"] = DK_IFIDN;
  DirectiveKindMap["ifidni"] = DK_IFIDNI;
  DirectiveKindMap["elseif"] = DK_ELSEIF;
  DirectiveKindMap["elseifdef"] = DK_ELSEIFDEF;
  DirectiveKindMap["elseifndef"] = DK_ELSEIFNDEF;
  DirectiveKindMap["elseifdif"] = DK_ELSEIFDIF;
  DirectiveKindMap["elseifidn"] = DK_ELSEIFIDN;
  DirectiveKindMap["else"] = DK_ELSE;
  DirectiveKindMap["end"] = DK_END;
  DirectiveKindMap["endif"] = DK_ENDIF;
  // DirectiveKindMap[".file"] = DK_FILE;
  // DirectiveKindMap[".line"] = DK_LINE;
  // DirectiveKindMap[".loc"] = DK_LOC;
  // DirectiveKindMap[".stabs"] = DK_STABS;
  // DirectiveKindMap[".cv_file"] = DK_CV_FILE;
  // DirectiveKindMap[".cv_func_id"] = DK_CV_FUNC_ID;
  // DirectiveKindMap[".cv_loc"] = DK_CV_LOC;
  // DirectiveKindMap[".cv_linetable"] = DK_CV_LINETABLE;
  // DirectiveKindMap[".cv_inline_linetable"] = DK_CV_INLINE_LINETABLE;
  // DirectiveKindMap[".cv_inline_site_id"] = DK_CV_INLINE_SITE_ID;
  // DirectiveKindMap[".cv_def_range"] = DK_CV_DEF_RANGE;
  // DirectiveKindMap[".cv_string"] = DK_CV_STRING;
  // DirectiveKindMap[".cv_stringtable"] = DK_CV_STRINGTABLE;
  // DirectiveKindMap[".cv_filechecksums"] = DK_CV_FILECHECKSUMS;
  // DirectiveKindMap[".cv_filechecksumoffset"] = DK_CV_FILECHECKSUM_OFFSET;
  // DirectiveKindMap[".cv_fpo_data"] = DK_CV_FPO_DATA;
  // DirectiveKindMap[".cfi_sections"] = DK_CFI_SECTIONS;
  // DirectiveKindMap[".cfi_startproc"] = DK_CFI_STARTPROC;
  // DirectiveKindMap[".cfi_endproc"] = DK_CFI_ENDPROC;
  // DirectiveKindMap[".cfi_def_cfa"] = DK_CFI_DEF_CFA;
  // DirectiveKindMap[".cfi_def_cfa_offset"] = DK_CFI_DEF_CFA_OFFSET;
  // DirectiveKindMap[".cfi_adjust_cfa_offset"] = DK_CFI_ADJUST_CFA_OFFSET;
  // DirectiveKindMap[".cfi_def_cfa_register"] = DK_CFI_DEF_CFA_REGISTER;
  // DirectiveKindMap[".cfi_offset"] = DK_CFI_OFFSET;
  // DirectiveKindMap[".cfi_rel_offset"] = DK_CFI_REL_OFFSET;
  // DirectiveKindMap[".cfi_llvm_register_pair"] = DK_CFI_LLVM_REGISTER_PAIR;
  // DirectiveKindMap[".cfi_llvm_vector_registers"] =
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 5276-5315
```cpp
  //   DK_CFI_LLVM_VECTOR_REGISTERS;
  // DirectiveKindMap[".cfi_llvm_vector_offset"] = DK_CFI_LLVM_VECTOR_OFFSET;
  // DirectiveKindMap[".cfi_personality"] = DK_CFI_PERSONALITY;
  // DirectiveKindMap[".cfi_lsda"] = DK_CFI_LSDA;
  // DirectiveKindMap[".cfi_remember_state"] = DK_CFI_REMEMBER_STATE;
  // DirectiveKindMap[".cfi_restore_state"] = DK_CFI_RESTORE_STATE;
  // DirectiveKindMap[".cfi_same_value"] = DK_CFI_SAME_VALUE;
  // DirectiveKindMap[".cfi_restore"] = DK_CFI_RESTORE;
  // DirectiveKindMap[".cfi_escape"] = DK_CFI_ESCAPE;
  // DirectiveKindMap[".cfi_return_column"] = DK_CFI_RETURN_COLUMN;
  // DirectiveKindMap[".cfi_signal_frame"] = DK_CFI_SIGNAL_FRAME;
  // DirectiveKindMap[".cfi_undefined"] = DK_CFI_UNDEFINED;
  // DirectiveKindMap[".cfi_register"] = DK_CFI_REGISTER;
  // DirectiveKindMap[".cfi_window_save"] = DK_CFI_WINDOW_SAVE;
  // DirectiveKindMap[".cfi_b_key_frame"] = DK_CFI_B_KEY_FRAME;
  // DirectiveKindMap[".cfi_val_offset"] = DK_CFI_VAL_OFFSET;
  DirectiveKindMap["macro"] = DK_MACRO;
  DirectiveKindMap["exitm"] = DK_EXITM;
  DirectiveKindMap["endm"] = DK_ENDM;
  DirectiveKindMap["purge"] = DK_PURGE;
  DirectiveKindMap[".err"] = DK_ERR;
  DirectiveKindMap[".errb"] = DK_ERRB;
  DirectiveKindMap[".errnb"] = DK_ERRNB;
  DirectiveKindMap[".errdef"] = DK_ERRDEF;
  DirectiveKindMap[".errndef"] = DK_ERRNDEF;
  DirectiveKindMap[".errdif"] = DK_ERRDIF;
  DirectiveKindMap[".errdifi"] = DK_ERRDIFI;
  DirectiveKindMap[".erridn"] = DK_ERRIDN;
  DirectiveKindMap[".erridni"] = DK_ERRIDNI;
  DirectiveKindMap[".erre"] = DK_ERRE;
  DirectiveKindMap[".errnz"] = DK_ERRNZ;
  DirectiveKindMap[".pushframe"] = DK_PUSHFRAME;
  DirectiveKindMap[".pushreg"] = DK_PUSHREG;
  DirectiveKindMap[".savereg"] = DK_SAVEREG;
  DirectiveKindMap[".savexmm128"] = DK_SAVEXMM128;
  DirectiveKindMap[".setframe"] = DK_SETFRAME;
  DirectiveKindMap[".radix"] = DK_RADIX;
  DirectiveKindMap["db"] = DK_DB;
  DirectiveKindMap["dd"] = DK_DD;
  DirectiveKindMap["df"] = DK_DF;
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 5316-5339
```cpp
  DirectiveKindMap["dq"] = DK_DQ;
  DirectiveKindMap["dw"] = DK_DW;
  DirectiveKindMap["echo"] = DK_ECHO;
  DirectiveKindMap["struc"] = DK_STRUCT;
  DirectiveKindMap["struct"] = DK_STRUCT;
  DirectiveKindMap["union"] = DK_UNION;
  DirectiveKindMap["ends"] = DK_ENDS;
}

bool MasmParser::isMacroLikeDirective() {
  if (getLexer().is(AsmToken::Identifier)) {
    bool IsMacroLike = StringSwitch<bool>(getTok().getIdentifier())
                           .CasesLower({"repeat", "rept"}, true)
                           .CaseLower("while", true)
                           .CasesLower({"for", "irp"}, true)
                           .CasesLower({"forc", "irpc"}, true)
                           .Default(false);
    if (IsMacroLike)
      return true;
  }
  if (peekTok().is(AsmToken::Identifier) &&
      peekTok().getIdentifier().equals_insensitive("macro"))
    return true;

```
- **EN**: Implements logic around `isMacroLikeDirective`, `StringSwitch<bool>`, `CasesLower`, `CaseLower`, and 2 more symbols; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `isMacroLikeDirective`, `StringSwitch<bool>`, `CasesLower`, `CaseLower`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 5340-5371
```cpp
  return false;
}

MCAsmMacro *MasmParser::parseMacroLikeBody(SMLoc DirectiveLoc) {
  AsmToken EndToken, StartToken = getTok();

  unsigned NestLevel = 0;
  while (true) {
    // Check whether we have reached the end of the file.
    if (getLexer().is(AsmToken::Eof)) {
      printError(DirectiveLoc, "no matching 'endm' in definition");
      return nullptr;
    }

    if (isMacroLikeDirective())
      ++NestLevel;

    // Otherwise, check whether we have reached the endm.
    if (Lexer.is(AsmToken::Identifier) &&
        getTok().getIdentifier().equals_insensitive("endm")) {
      if (NestLevel == 0) {
        EndToken = getTok();
        Lex();
        if (Lexer.isNot(AsmToken::EndOfStatement)) {
          printError(getTok().getLoc(), "unexpected token in 'endm' directive");
          return nullptr;
        }
        break;
      }
      --NestLevel;
    }

```
- **EN**: Implements logic around `parseMacroLikeBody`, `getTok`, `printError`, `Lex`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseMacroLikeBody`, `getTok`, `printError`, `Lex` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 5372-5391
```cpp
    // Otherwise, scan till the end of the statement.
    eatToEndOfStatement();
  }

  const char *BodyStart = StartToken.getLoc().getPointer();
  const char *BodyEnd = EndToken.getLoc().getPointer();
  StringRef Body = StringRef(BodyStart, BodyEnd - BodyStart);

  // We Are Anonymous.
  MacroLikeBodies.emplace_back(StringRef(), Body, MCAsmMacroParameters());
  return &MacroLikeBodies.back();
}

bool MasmParser::expandStatement(SMLoc Loc) {
  std::string Body = parseStringTo(AsmToken::EndOfStatement);
  SMLoc EndLoc = getTok().getLoc();

  MCAsmMacroParameters Parameters;
  MCAsmMacroArguments Arguments;

```
- **EN**: Implements logic around `eatToEndOfStatement`, `getLoc`, `StringRef`, `emplace_back`, and 4 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `eatToEndOfStatement`, `getLoc`, `StringRef`, `emplace_back`, and 4 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 5392-5418
```cpp
  StringMap<std::string> BuiltinValues;
  for (const auto &S : BuiltinSymbolMap) {
    const BuiltinSymbol &Sym = S.getValue();
    if (std::optional<std::string> Text = evaluateBuiltinTextMacro(Sym, Loc)) {
      BuiltinValues[S.getKey().lower()] = std::move(*Text);
    }
  }
  for (const auto &B : BuiltinValues) {
    MCAsmMacroParameter P;
    MCAsmMacroArgument A;
    P.Name = B.getKey();
    P.Required = true;
    A.push_back(AsmToken(AsmToken::String, B.getValue()));

    Parameters.push_back(std::move(P));
    Arguments.push_back(std::move(A));
  }

  for (const auto &V : Variables) {
    const Variable &Var = V.getValue();
    if (Var.IsText) {
      MCAsmMacroParameter P;
      MCAsmMacroArgument A;
      P.Name = Var.Name;
      P.Required = true;
      A.push_back(AsmToken(AsmToken::String, Var.TextValue));

```
- **EN**: Implements logic around `getValue`, `getKey`, `push_back`; this block parses assembly syntax or operands; updates MC section or symbol state.
- **CN**: 围绕 `getValue`, `getKey`, `push_back` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态。

### Lines 5419-5441
```cpp
      Parameters.push_back(std::move(P));
      Arguments.push_back(std::move(A));
    }
  }
  MacroLikeBodies.emplace_back(StringRef(), Body, Parameters);
  MCAsmMacro M = MacroLikeBodies.back();

  // Expand the statement in a new buffer.
  SmallString<80> Buf;
  raw_svector_ostream OS(Buf);
  if (expandMacro(OS, M.Body, M.Parameters, Arguments, M.Locals, EndLoc))
    return true;
  std::unique_ptr<MemoryBuffer> Expansion =
      MemoryBuffer::getMemBufferCopy(OS.str(), "<expansion>");

  // Jump to the expanded statement and prime the lexer.
  CurBuffer = SrcMgr.AddNewSourceBuffer(std::move(Expansion), EndLoc);
  Lexer.setBuffer(SrcMgr.getMemoryBuffer(CurBuffer)->getBuffer());
  EndStatementAtEOFStack.push_back(false);
  Lex();
  return false;
}

```
- **EN**: Implements logic around `push_back`, `emplace_back`, `back`, `OS`, and 4 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `push_back`, `emplace_back`, `back`, `OS`, and 4 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 5442-5466
```cpp
void MasmParser::instantiateMacroLikeBody(MCAsmMacro *M, SMLoc DirectiveLoc,
                                          raw_svector_ostream &OS) {
  instantiateMacroLikeBody(M, DirectiveLoc, /*ExitLoc=*/getTok().getLoc(), OS);
}
void MasmParser::instantiateMacroLikeBody(MCAsmMacro *M, SMLoc DirectiveLoc,
                                          SMLoc ExitLoc,
                                          raw_svector_ostream &OS) {
  OS << "endm\n";

  std::unique_ptr<MemoryBuffer> Instantiation =
      MemoryBuffer::getMemBufferCopy(OS.str(), "<instantiation>");

  // Create the macro instantiation object and add to the current macro
  // instantiation stack.
  MacroInstantiation *MI = new MacroInstantiation{DirectiveLoc, CurBuffer,
                                                  ExitLoc, TheCondStack.size()};
  ActiveMacros.push_back(MI);

  // Jump to the macro instantiation and prime the lexer.
  CurBuffer = SrcMgr.AddNewSourceBuffer(std::move(Instantiation), SMLoc());
  Lexer.setBuffer(SrcMgr.getMemoryBuffer(CurBuffer)->getBuffer());
  EndStatementAtEOFStack.push_back(true);
  Lex();
}

```
- **EN**: Implements logic around `instantiateMacroLikeBody`, `getMemBufferCopy`, `size`, `push_back`, and 3 more symbols; this block parses assembly syntax or operands.
- **CN**: 围绕 `instantiateMacroLikeBody`, `getMemBufferCopy`, `size`, `push_back`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 5467-5489
```cpp
/// parseDirectiveRepeat
///   ::= ("repeat" | "rept") count
///       body
///     endm
bool MasmParser::parseDirectiveRepeat(SMLoc DirectiveLoc, StringRef Dir) {
  const MCExpr *CountExpr;
  SMLoc CountLoc = getTok().getLoc();
  if (parseExpression(CountExpr))
    return true;

  int64_t Count;
  if (!CountExpr->evaluateAsAbsolute(Count, getStreamer().getAssemblerPtr())) {
    return Error(CountLoc, "unexpected token in '" + Dir + "' directive");
  }

  if (check(Count < 0, CountLoc, "Count is negative") || parseEOL())
    return true;

  // Lex the repeat definition.
  MCAsmMacro *M = parseMacroLikeBody(DirectiveLoc);
  if (!M)
    return true;

```
- **EN**: Implements logic around `parseDirectiveRepeat`, `getTok`, `Error`, `parseMacroLikeBody`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveRepeat`, `getTok`, `Error`, `parseMacroLikeBody` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 5490-5512
```cpp
  // Macro instantiation is lexical, unfortunately. We construct a new buffer
  // to hold the macro body with substitutions.
  SmallString<256> Buf;
  raw_svector_ostream OS(Buf);
  while (Count--) {
    if (expandMacro(OS, M->Body, {}, {}, M->Locals, getTok().getLoc()))
      return true;
  }
  instantiateMacroLikeBody(M, DirectiveLoc, OS);

  return false;
}

/// parseDirectiveWhile
/// ::= "while" expression
///       body
///     endm
bool MasmParser::parseDirectiveWhile(SMLoc DirectiveLoc) {
  const MCExpr *CondExpr;
  SMLoc CondLoc = getTok().getLoc();
  if (parseExpression(CondExpr))
    return true;

```
- **EN**: Implements logic around `OS`, `instantiateMacroLikeBody`, `parseDirectiveWhile`, `getTok`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `OS`, `instantiateMacroLikeBody`, `parseDirectiveWhile`, `getTok` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 5513-5532
```cpp
  // Lex the repeat definition.
  MCAsmMacro *M = parseMacroLikeBody(DirectiveLoc);
  if (!M)
    return true;

  // Macro instantiation is lexical, unfortunately. We construct a new buffer
  // to hold the macro body with substitutions.
  SmallString<256> Buf;
  raw_svector_ostream OS(Buf);
  int64_t Condition;
  if (!CondExpr->evaluateAsAbsolute(Condition, getStreamer().getAssemblerPtr()))
    return Error(CondLoc, "expected absolute expression in 'while' directive");
  if (Condition) {
    // Instantiate the macro, then resume at this directive to recheck the
    // condition.
    if (expandMacro(OS, M->Body, {}, {}, M->Locals, getTok().getLoc()))
      return true;
    instantiateMacroLikeBody(M, DirectiveLoc, /*ExitLoc=*/DirectiveLoc, OS);
  }

```
- **EN**: Implements logic around `parseMacroLikeBody`, `OS`, `Error`, `instantiateMacroLikeBody`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `parseMacroLikeBody`, `OS`, `Error`, `instantiateMacroLikeBody` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 5533-5552
```cpp
  return false;
}

/// parseDirectiveFor
/// ::= ("for" | "irp") symbol [":" qualifier], <values>
///       body
///     endm
bool MasmParser::parseDirectiveFor(SMLoc DirectiveLoc, StringRef Dir) {
  MCAsmMacroParameter Parameter;
  MCAsmMacroArguments A;
  if (check(parseIdentifier(Parameter.Name),
            "expected identifier in '" + Dir + "' directive"))
    return true;

  // Parse optional qualifier (default value, or "req")
  if (parseOptionalToken(AsmToken::Colon)) {
    if (parseOptionalToken(AsmToken::Equal)) {
      // Default value
      SMLoc ParamLoc;

```
- **EN**: Implements logic around `parseDirectiveFor`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveFor` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 5553-5575
```cpp
      ParamLoc = Lexer.getLoc();
      if (parseMacroArgument(nullptr, Parameter.Value))
        return true;
    } else {
      SMLoc QualLoc;
      StringRef Qualifier;

      QualLoc = Lexer.getLoc();
      if (parseIdentifier(Qualifier))
        return Error(QualLoc, "missing parameter qualifier for "
                              "'" +
                                  Parameter.Name + "' in '" + Dir +
                                  "' directive");

      if (Qualifier.equals_insensitive("req"))
        Parameter.Required = true;
      else
        return Error(QualLoc,
                     Qualifier + " is not a valid parameter qualifier for '" +
                         Parameter.Name + "' in '" + Dir + "' directive");
    }
  }

```
- **EN**: Implements logic around `getLoc`, `Error`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `getLoc`, `Error` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 5576-5599
```cpp
  if (parseToken(AsmToken::Comma,
                 "expected comma in '" + Dir + "' directive") ||
      parseToken(AsmToken::Less,
                 "values in '" + Dir +
                     "' directive must be enclosed in angle brackets"))
    return true;

  while (true) {
    A.emplace_back();
    if (parseMacroArgument(&Parameter, A.back(), /*EndTok=*/AsmToken::Greater))
      return addErrorSuffix(" in arguments for '" + Dir + "' directive");

    // If we see a comma, continue, and allow line continuation.
    if (!parseOptionalToken(AsmToken::Comma))
      break;
    parseOptionalToken(AsmToken::EndOfStatement);
  }

  if (parseToken(AsmToken::Greater,
                 "values in '" + Dir +
                     "' directive must be enclosed in angle brackets") ||
      parseEOL())
    return true;

```
- **EN**: Implements logic around `parseToken`, `emplace_back`, `addErrorSuffix`, `parseOptionalToken`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseToken`, `emplace_back`, `addErrorSuffix`, `parseOptionalToken`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 5600-5619
```cpp
  // Lex the for definition.
  MCAsmMacro *M = parseMacroLikeBody(DirectiveLoc);
  if (!M)
    return true;

  // Macro instantiation is lexical, unfortunately. We construct a new buffer
  // to hold the macro body with substitutions.
  SmallString<256> Buf;
  raw_svector_ostream OS(Buf);

  for (const MCAsmMacroArgument &Arg : A) {
    if (expandMacro(OS, M->Body, Parameter, Arg, M->Locals, getTok().getLoc()))
      return true;
  }

  instantiateMacroLikeBody(M, DirectiveLoc, OS);

  return false;
}

```
- **EN**: Implements logic around `parseMacroLikeBody`, `OS`, `instantiateMacroLikeBody`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseMacroLikeBody`, `OS`, `instantiateMacroLikeBody` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 5620-5649
```cpp
/// parseDirectiveForc
/// ::= ("forc" | "irpc") symbol, <string>
///       body
///     endm
bool MasmParser::parseDirectiveForc(SMLoc DirectiveLoc, StringRef Directive) {
  MCAsmMacroParameter Parameter;

  std::string Argument;
  if (check(parseIdentifier(Parameter.Name),
            "expected identifier in '" + Directive + "' directive") ||
      parseToken(AsmToken::Comma,
                 "expected comma in '" + Directive + "' directive"))
    return true;
  if (parseAngleBracketString(Argument)) {
    // Match ml64.exe; treat all characters to end of statement as a string,
    // ignoring comment markers, then discard anything following a space (using
    // the C locale).
    Argument = parseStringTo(AsmToken::EndOfStatement);
    if (getTok().is(AsmToken::EndOfStatement))
      Argument += getTok().getString();
    size_t End = 0;
    for (; End < Argument.size(); ++End) {
      if (isSpace(Argument[End]))
        break;
    }
    Argument.resize(End);
  }
  if (parseEOL())
    return true;

```
- **EN**: Implements logic around `parseDirectiveForc`, `parseToken`, `parseStringTo`, `getTok`, and 1 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveForc`, `parseToken`, `parseStringTo`, `getTok`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 5650-5670
```cpp
  // Lex the irpc definition.
  MCAsmMacro *M = parseMacroLikeBody(DirectiveLoc);
  if (!M)
    return true;

  // Macro instantiation is lexical, unfortunately. We construct a new buffer
  // to hold the macro body with substitutions.
  SmallString<256> Buf;
  raw_svector_ostream OS(Buf);

  StringRef Values(Argument);
  for (std::size_t I = 0, End = Values.size(); I != End; ++I) {
    MCAsmMacroArgument Arg;
    Arg.emplace_back(AsmToken::Identifier, Values.substr(I, 1));

    if (expandMacro(OS, M->Body, Parameter, Arg, M->Locals, getTok().getLoc()))
      return true;
  }

  instantiateMacroLikeBody(M, DirectiveLoc, OS);

```
- **EN**: Implements logic around `parseMacroLikeBody`, `OS`, `Values`, `emplace_back`, and 1 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `parseMacroLikeBody`, `OS`, `Values`, `emplace_back`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 5671-5690
```cpp
  return false;
}

bool MasmParser::parseDirectiveMSEmit(SMLoc IDLoc, ParseStatementInfo &Info,
                                      size_t Len) {
  const MCExpr *Value;
  SMLoc ExprLoc = getLexer().getLoc();
  if (parseExpression(Value))
    return true;
  const MCConstantExpr *MCE = dyn_cast<MCConstantExpr>(Value);
  if (!MCE)
    return Error(ExprLoc, "unexpected expression in _emit");
  uint64_t IntValue = MCE->getValue();
  if (!isUInt<8>(IntValue) && !isInt<8>(IntValue))
    return Error(ExprLoc, "literal value out of range for directive");

  Info.AsmRewrites->emplace_back(AOK_Emit, IDLoc, Len);
  return false;
}

```
- **EN**: Implements logic around `parseDirectiveMSEmit`, `getLexer`, `dyn_cast<MCConstantExpr>`, `Error`, and 2 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveMSEmit`, `getLexer`, `dyn_cast<MCConstantExpr>`, `Error`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 5691-5723
```cpp
bool MasmParser::parseDirectiveMSAlign(SMLoc IDLoc, ParseStatementInfo &Info) {
  const MCExpr *Value;
  SMLoc ExprLoc = getLexer().getLoc();
  if (parseExpression(Value))
    return true;
  const MCConstantExpr *MCE = dyn_cast<MCConstantExpr>(Value);
  if (!MCE)
    return Error(ExprLoc, "unexpected expression in align");
  uint64_t IntValue = MCE->getValue();
  if (!isPowerOf2_64(IntValue))
    return Error(ExprLoc, "literal value not a power of two greater then zero");

  Info.AsmRewrites->emplace_back(AOK_Align, IDLoc, 5, Log2_64(IntValue));
  return false;
}

bool MasmParser::parseDirectiveRadix(SMLoc DirectiveLoc) {
  const SMLoc Loc = getLexer().getLoc();
  std::string RadixStringRaw = parseStringTo(AsmToken::EndOfStatement);
  StringRef RadixString = StringRef(RadixStringRaw).trim();
  unsigned Radix;
  if (RadixString.getAsInteger(10, Radix)) {
    return Error(Loc,
                 "radix must be a decimal number in the range 2 to 16; was " +
                     RadixString);
  }
  if (Radix < 2 || Radix > 16)
    return Error(Loc, "radix must be in the range 2 to 16; was " +
                          std::to_string(Radix));
  getLexer().setMasmDefaultRadix(Radix);
  return false;
}

```
- **EN**: Implements logic around `parseDirectiveMSAlign`, `getLexer`, `dyn_cast<MCConstantExpr>`, `Error`, and 6 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveMSAlign`, `getLexer`, `dyn_cast<MCConstantExpr>`, `Error`, and 6 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 5724-5750
```cpp
/// parseDirectiveEcho
///   ::= "echo" message
bool MasmParser::parseDirectiveEcho(SMLoc DirectiveLoc) {
  std::string Message = parseStringTo(AsmToken::EndOfStatement);
  llvm::outs() << Message;
  if (!StringRef(Message).ends_with("\n"))
    llvm::outs() << '\n';
  return false;
}

// We are comparing pointers, but the pointers are relative to a single string.
// Thus, this should always be deterministic.
static int rewritesSort(const AsmRewrite *AsmRewriteA,
                        const AsmRewrite *AsmRewriteB) {
  if (AsmRewriteA->Loc.getPointer() < AsmRewriteB->Loc.getPointer())
    return -1;
  if (AsmRewriteB->Loc.getPointer() < AsmRewriteA->Loc.getPointer())
    return 1;

  // It's possible to have a SizeDirective, Imm/ImmPrefix and an Input/Output
  // rewrite to the same location.  Make sure the SizeDirective rewrite is
  // performed first, then the Imm/ImmPrefix and finally the Input/Output.  This
  // ensures the sort algorithm is stable.
  if (AsmRewritePrecedence[AsmRewriteA->Kind] >
      AsmRewritePrecedence[AsmRewriteB->Kind])
    return -1;

```
- **EN**: Implements logic around `parseDirectiveEcho`, `parseStringTo`, `outs`, `rewritesSort`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `parseDirectiveEcho`, `parseStringTo`, `outs`, `rewritesSort` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 5751-5773
```cpp
  if (AsmRewritePrecedence[AsmRewriteA->Kind] <
      AsmRewritePrecedence[AsmRewriteB->Kind])
    return 1;
  llvm_unreachable("Unstable rewrite sort.");
}

bool MasmParser::defineMacro(StringRef Name, StringRef Value) {
  Variable &Var = Variables[Name.lower()];
  if (Var.Name.empty()) {
    Var.Name = Name;
  } else if (Var.Redefinable == Variable::NOT_REDEFINABLE) {
    return Error(SMLoc(), "invalid variable redefinition");
  } else if (Var.Redefinable == Variable::WARN_ON_REDEFINITION &&
             Warning(SMLoc(), "redefining '" + Name +
                                  "', already defined on the command line")) {
    return true;
  }
  Var.Redefinable = Variable::WARN_ON_REDEFINITION;
  Var.IsText = true;
  Var.TextValue = Value.str();
  return false;
}

```
- **EN**: Implements logic around `llvm_unreachable`, `defineMacro`, `lower`, `Error`, and 2 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `llvm_unreachable`, `defineMacro`, `lower`, `Error`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 5774-5796
```cpp
bool MasmParser::lookUpField(StringRef Name, AsmFieldInfo &Info) const {
  const std::pair<StringRef, StringRef> BaseMember = Name.split('.');
  const StringRef Base = BaseMember.first, Member = BaseMember.second;
  return lookUpField(Base, Member, Info);
}

bool MasmParser::lookUpField(StringRef Base, StringRef Member,
                             AsmFieldInfo &Info) const {
  if (Base.empty())
    return true;

  AsmFieldInfo BaseInfo;
  if (Base.contains('.') && !lookUpField(Base, BaseInfo))
    Base = BaseInfo.Type.Name;

  auto StructIt = Structs.find(Base.lower());
  auto TypeIt = KnownType.find(Base.lower());
  if (TypeIt != KnownType.end()) {
    StructIt = Structs.find(TypeIt->second.Name.lower());
  }
  if (StructIt != Structs.end())
    return lookUpField(StructIt->second, Member, Info);

```
- **EN**: Implements logic around `lookUpField`, `split`, `find`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `lookUpField`, `split`, `find` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 5797-5816
```cpp
  return true;
}

bool MasmParser::lookUpField(const StructInfo &Structure, StringRef Member,
                             AsmFieldInfo &Info) const {
  if (Member.empty()) {
    Info.Type.Name = Structure.Name;
    Info.Type.Size = Structure.Size;
    Info.Type.ElementSize = Structure.Size;
    Info.Type.Length = 1;
    return false;
  }

  std::pair<StringRef, StringRef> Split = Member.split('.');
  const StringRef FieldName = Split.first, FieldMember = Split.second;

  auto StructIt = Structs.find(FieldName.lower());
  if (StructIt != Structs.end())
    return lookUpField(StructIt->second, FieldMember, Info);

```
- **EN**: Implements logic around `lookUpField`, `split`, `find`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `lookUpField`, `split`, `find` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 5817-5837
```cpp
  auto FieldIt = Structure.FieldsByName.find(FieldName.lower());
  if (FieldIt == Structure.FieldsByName.end())
    return true;

  const FieldInfo &Field = Structure.Fields[FieldIt->second];
  if (FieldMember.empty()) {
    Info.Offset += Field.Offset;
    Info.Type.Size = Field.SizeOf;
    Info.Type.ElementSize = Field.Type;
    Info.Type.Length = Field.LengthOf;
    if (Field.Contents.FT == FT_STRUCT)
      Info.Type.Name = Field.Contents.StructInfo.Structure.Name;
    else
      Info.Type.Name = "";
    return false;
  }

  if (Field.Contents.FT != FT_STRUCT)
    return true;
  const StructFieldInfo &StructInfo = Field.Contents.StructInfo;

```
- **EN**: Implements logic around `find`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `find` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 5838-5863
```cpp
  if (lookUpField(StructInfo.Structure, FieldMember, Info))
    return true;

  Info.Offset += Field.Offset;
  return false;
}

bool MasmParser::lookUpType(StringRef Name, AsmTypeInfo &Info) const {
  unsigned Size = StringSwitch<unsigned>(Name)
                      .CasesLower({"byte", "db", "sbyte"}, 1)
                      .CasesLower({"word", "dw", "sword"}, 2)
                      .CasesLower({"dword", "dd", "sdword"}, 4)
                      .CasesLower({"fword", "df"}, 6)
                      .CasesLower({"qword", "dq", "sqword"}, 8)
                      .CaseLower("real4", 4)
                      .CaseLower("real8", 8)
                      .CaseLower("real10", 10)
                      .Default(0);
  if (Size) {
    Info.Name = Name;
    Info.ElementSize = Size;
    Info.Length = 1;
    Info.Size = Size;
    return false;
  }

```
- **EN**: Implements logic around `lookUpType`, `StringSwitch<unsigned>`, `CasesLower`, `CaseLower`, and 1 more symbols; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `lookUpType`, `StringSwitch<unsigned>`, `CasesLower`, `CaseLower`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 5864-5890
```cpp
  auto StructIt = Structs.find(Name.lower());
  if (StructIt != Structs.end()) {
    const StructInfo &Structure = StructIt->second;
    Info.Name = Name;
    Info.ElementSize = Structure.Size;
    Info.Length = 1;
    Info.Size = Structure.Size;
    return false;
  }

  return true;
}

bool MasmParser::parseMSInlineAsm(
    std::string &AsmString, unsigned &NumOutputs, unsigned &NumInputs,
    SmallVectorImpl<std::pair<void *, bool>> &OpDecls,
    SmallVectorImpl<std::string> &Constraints,
    SmallVectorImpl<std::string> &Clobbers, const MCInstrInfo *MII,
    MCInstPrinter *IP, MCAsmParserSemaCallback &SI) {
  SmallVector<void *, 4> InputDecls;
  SmallVector<void *, 4> OutputDecls;
  SmallVector<bool, 4> InputDeclsAddressOf;
  SmallVector<bool, 4> OutputDeclsAddressOf;
  SmallVector<std::string, 4> InputConstraints;
  SmallVector<std::string, 4> OutputConstraints;
  SmallVector<MCRegister, 4> ClobberRegs;

```
- **EN**: Implements logic around `find`, `parseMSInlineAsm`; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `find`, `parseMSInlineAsm` 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 5891-5912
```cpp
  SmallVector<AsmRewrite, 4> AsmStrRewrites;

  // Prime the lexer.
  Lex();

  // While we have input, parse each statement.
  unsigned InputIdx = 0;
  unsigned OutputIdx = 0;
  while (getLexer().isNot(AsmToken::Eof)) {
    // Parse curly braces marking block start/end.
    if (parseCurlyBlockScope(AsmStrRewrites))
      continue;

    ParseStatementInfo Info(&AsmStrRewrites);
    bool StatementErr = parseStatement(Info, &SI);

    if (StatementErr || Info.ParseError) {
      // Emit pending errors if any exist.
      printPendingErrors();
      return true;
    }

```
- **EN**: Implements logic around `Lex`, `Info`, `parseStatement`, `printPendingErrors`; this block manipulates JITLink graph structures or link-time passes; parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `Lex`, `Info`, `parseStatement`, `printPendingErrors` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 5913-5934
```cpp
    // No pending error should exist here.
    assert(!hasPendingError() && "unexpected error from parseStatement");

    if (Info.Opcode == ~0U)
      continue;

    const MCInstrDesc &Desc = MII->get(Info.Opcode);

    // Build the list of clobbers, outputs and inputs.
    for (unsigned i = 1, e = Info.ParsedOperands.size(); i != e; ++i) {
      MCParsedAsmOperand &Operand = *Info.ParsedOperands[i];

      // Register operand.
      if (Operand.isReg() && !Operand.needAddressOf() &&
          !getTargetParser().omitRegisterFromClobberLists(Operand.getReg())) {
        unsigned NumDefs = Desc.getNumDefs();
        // Clobber.
        if (NumDefs && Operand.getMCOperandNum() < NumDefs)
          ClobberRegs.push_back(Operand.getReg());
        continue;
      }

```
- **EN**: Implements logic around `assert`, `get`, `getTargetParser`, `getNumDefs`, and 1 more symbols; this block parses assembly syntax or operands.
- **CN**: 围绕 `assert`, `get`, `getTargetParser`, `getNumDefs`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数。

### Lines 5935-5971
```cpp
      // Expr/Input or Output.
      StringRef SymName = Operand.getSymName();
      if (SymName.empty())
        continue;

      void *OpDecl = Operand.getOpDecl();
      if (!OpDecl)
        continue;

      StringRef Constraint = Operand.getConstraint();
      if (Operand.isImm()) {
        // Offset as immediate.
        if (Operand.isOffsetOfLocal())
          Constraint = "r";
        else
          Constraint = "i";
      }

      bool isOutput = (i == 1) && Desc.mayStore();
      SMLoc Start = SMLoc::getFromPointer(SymName.data());
      if (isOutput) {
        ++InputIdx;
        OutputDecls.push_back(OpDecl);
        OutputDeclsAddressOf.push_back(Operand.needAddressOf());
        OutputConstraints.push_back(("=" + Constraint).str());
        AsmStrRewrites.emplace_back(AOK_Output, Start, SymName.size());
      } else {
        InputDecls.push_back(OpDecl);
        InputDeclsAddressOf.push_back(Operand.needAddressOf());
        InputConstraints.push_back(Constraint.str());
        if (Desc.operands()[i - 1].isBranchTarget())
          AsmStrRewrites.emplace_back(AOK_CallInput, Start, SymName.size());
        else
          AsmStrRewrites.emplace_back(AOK_Input, Start, SymName.size());
      }
    }

```
- **EN**: Implements logic around `getSymName`, `getOpDecl`, `getConstraint`, `mayStore`, and 3 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `getSymName`, `getOpDecl`, `getConstraint`, `mayStore`, and 3 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为。

### Lines 5972-6003
```cpp
    // Consider implicit defs to be clobbers.  Think of cpuid and push.
    llvm::append_range(ClobberRegs, Desc.implicit_defs());
  }

  // Set the number of Outputs and Inputs.
  NumOutputs = OutputDecls.size();
  NumInputs = InputDecls.size();

  // Set the unique clobbers.
  array_pod_sort(ClobberRegs.begin(), ClobberRegs.end());
  ClobberRegs.erase(llvm::unique(ClobberRegs), ClobberRegs.end());
  Clobbers.assign(ClobberRegs.size(), std::string());
  for (unsigned I = 0, E = ClobberRegs.size(); I != E; ++I) {
    raw_string_ostream OS(Clobbers[I]);
    IP->printRegName(OS, ClobberRegs[I]);
  }

  // Merge the various outputs and inputs.  Output are expected first.
  if (NumOutputs || NumInputs) {
    unsigned NumExprs = NumOutputs + NumInputs;
    OpDecls.resize(NumExprs);
    Constraints.resize(NumExprs);
    for (unsigned i = 0; i < NumOutputs; ++i) {
      OpDecls[i] = std::make_pair(OutputDecls[i], OutputDeclsAddressOf[i]);
      Constraints[i] = OutputConstraints[i];
    }
    for (unsigned i = 0, j = NumOutputs; i < NumInputs; ++i, ++j) {
      OpDecls[j] = std::make_pair(InputDecls[i], InputDeclsAddressOf[i]);
      Constraints[j] = InputConstraints[i];
    }
  }

```
- **EN**: Implements logic around `append_range`, `size`, `array_pod_sort`, `erase`, and 5 more symbols.
- **CN**: 围绕 `append_range`, `size`, `array_pod_sort`, `erase`, and 5 more symbols 实现具体逻辑。

### Lines 6004-6025
```cpp
  // Build the IR assembly string.
  std::string AsmStringIR;
  raw_string_ostream OS(AsmStringIR);
  StringRef ASMString =
      SrcMgr.getMemoryBuffer(SrcMgr.getMainFileID())->getBuffer();
  const char *AsmStart = ASMString.begin();
  const char *AsmEnd = ASMString.end();
  array_pod_sort(AsmStrRewrites.begin(), AsmStrRewrites.end(), rewritesSort);
  for (auto I = AsmStrRewrites.begin(), E = AsmStrRewrites.end(); I != E; ++I) {
    const AsmRewrite &AR = *I;
    // Check if this has already been covered by another rewrite...
    if (AR.Done)
      continue;
    AsmRewriteKind Kind = AR.Kind;

    const char *Loc = AR.Loc.getPointer();
    assert(Loc >= AsmStart && "Expected Loc to be at or after Start!");

    // Emit everything up to the immediate/expression.
    if (unsigned Len = Loc - AsmStart)
      OS << StringRef(AsmStart, Len);

```
- **EN**: Implements logic around `OS`, `getMemoryBuffer`, `begin`, `end`, and 4 more symbols; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `OS`, `getMemoryBuffer`, `begin`, `end`, and 4 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 6026-6065
```cpp
    // Skip the original expression.
    if (Kind == AOK_Skip) {
      AsmStart = Loc + AR.Len;
      continue;
    }

    unsigned AdditionalSkip = 0;
    // Rewrite expressions in $N notation.
    switch (Kind) {
    default:
      break;
    case AOK_IntelExpr:
      assert(AR.IntelExp.isValid() && "cannot write invalid intel expression");
      if (AR.IntelExp.NeedBracs)
        OS << "[";
      if (AR.IntelExp.hasBaseReg())
        OS << AR.IntelExp.BaseReg;
      if (AR.IntelExp.hasIndexReg())
        OS << (AR.IntelExp.hasBaseReg() ? " + " : "")
           << AR.IntelExp.IndexReg;
      if (AR.IntelExp.Scale > 1)
        OS << " * $$" << AR.IntelExp.Scale;
      if (AR.IntelExp.hasOffset()) {
        if (AR.IntelExp.hasRegs())
          OS << " + ";
        // Fuse this rewrite with a rewrite of the offset name, if present.
        StringRef OffsetName = AR.IntelExp.OffsetName;
        SMLoc OffsetLoc = SMLoc::getFromPointer(AR.IntelExp.OffsetName.data());
        size_t OffsetLen = OffsetName.size();
        auto rewrite_it = std::find_if(
            I, AsmStrRewrites.end(), [&](const AsmRewrite &FusingAR) {
              return FusingAR.Loc == OffsetLoc && FusingAR.Len == OffsetLen &&
                     (FusingAR.Kind == AOK_Input ||
                      FusingAR.Kind == AOK_CallInput);
            });
        if (rewrite_it == AsmStrRewrites.end()) {
          OS << "offset " << OffsetName;
        } else if (rewrite_it->Kind == AOK_CallInput) {
          OS << "${" << InputIdx++ << ":P}";
          rewrite_it->Done = true;
```
- **EN**: Implements logic around `assert`, `hasBaseReg`, `getFromPointer`, `size`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `assert`, `hasBaseReg`, `getFromPointer`, `size`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 6066-6105
```cpp
        } else {
          OS << '$' << InputIdx++;
          rewrite_it->Done = true;
        }
      }
      if (AR.IntelExp.Imm || AR.IntelExp.emitImm())
        OS << (AR.IntelExp.emitImm() ? "$$" : " + $$") << AR.IntelExp.Imm;
      if (AR.IntelExp.NeedBracs)
        OS << "]";
      break;
    case AOK_Label:
      OS << Ctx.getAsmInfo().getPrivateLabelPrefix() << AR.Label;
      break;
    case AOK_Input:
      OS << '$' << InputIdx++;
      break;
    case AOK_CallInput:
      OS << "${" << InputIdx++ << ":P}";
      break;
    case AOK_Output:
      OS << '$' << OutputIdx++;
      break;
    case AOK_SizeDirective:
      switch (AR.Val) {
      default: break;
      case 8:  OS << "byte ptr "; break;
      case 16: OS << "word ptr "; break;
      case 32: OS << "dword ptr "; break;
      case 64: OS << "qword ptr "; break;
      case 80: OS << "xword ptr "; break;
      case 128: OS << "xmmword ptr "; break;
      case 256: OS << "ymmword ptr "; break;
      }
      break;
    case AOK_Emit:
      OS << ".byte";
      break;
    case AOK_Align: {
      // MS alignment directives are measured in bytes. If the native assembler
      // measures alignment in bytes, we can pass it straight through.
```
- **EN**: Implements logic around `emitImm`, `getAsmInfo`; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `emitImm`, `getAsmInfo` 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 6106-6125
```cpp
      OS << ".align";
      if (getContext().getAsmInfo().getAlignmentIsInBytes())
        break;

      // Alignment is in log2 form, so print that instead and skip the original
      // immediate.
      unsigned Val = AR.Val;
      OS << ' ' << Val;
      assert(Val < 10 && "Expected alignment less then 2^10.");
      AdditionalSkip = (Val < 4) ? 2 : Val < 7 ? 3 : 4;
      break;
    }
    case AOK_EVEN:
      OS << ".even";
      break;
    case AOK_EndOfStatement:
      OS << "\n\t";
      break;
    }

```
- **EN**: Implements logic around `assert`.
- **CN**: 围绕 `assert` 实现具体逻辑。

### Lines 6126-6149
```cpp
    // Skip the original expression.
    AsmStart = Loc + AR.Len + AdditionalSkip;
  }

  // Emit the remainder of the asm string.
  if (AsmStart != AsmEnd)
    OS << StringRef(AsmStart, AsmEnd - AsmStart);

  AsmString = OS.str();
  return false;
}

void MasmParser::initializeBuiltinSymbolMaps() {
  // Numeric built-ins (supported in all versions)
  BuiltinSymbolMap["@version"] = BI_VERSION;
  BuiltinSymbolMap["@line"] = BI_LINE;

  // Text built-ins (supported in all versions)
  BuiltinSymbolMap["@date"] = BI_DATE;
  BuiltinSymbolMap["@time"] = BI_TIME;
  BuiltinSymbolMap["@filecur"] = BI_FILECUR;
  BuiltinSymbolMap["@filename"] = BI_FILENAME;
  BuiltinSymbolMap["@curseg"] = BI_CURSEG;

```
- **EN**: Implements logic around `StringRef`, `str`, `initializeBuiltinSymbolMaps`; this block parses assembly syntax or operands; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `StringRef`, `str`, `initializeBuiltinSymbolMaps` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 6150-6171
```cpp
  // Function built-ins (supported in all versions)
  BuiltinFunctionMap["@catstr"] = BI_CATSTR;

  // Some built-ins exist only for MASM32 (32-bit x86)
  if (getContext().getSubtargetInfo()->getTargetTriple().getArch() ==
      Triple::x86) {
    // Numeric built-ins
    // BuiltinSymbolMap["@cpu"] = BI_CPU;
    // BuiltinSymbolMap["@interface"] = BI_INTERFACE;
    // BuiltinSymbolMap["@wordsize"] = BI_WORDSIZE;
    // BuiltinSymbolMap["@codesize"] = BI_CODESIZE;
    // BuiltinSymbolMap["@datasize"] = BI_DATASIZE;
    // BuiltinSymbolMap["@model"] = BI_MODEL;

    // Text built-ins
    // BuiltinSymbolMap["@code"] = BI_CODE;
    // BuiltinSymbolMap["@data"] = BI_DATA;
    // BuiltinSymbolMap["@fardata?"] = BI_FARDATA;
    // BuiltinSymbolMap["@stack"] = BI_STACK;
  }
}

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 6172-6192
```cpp
const MCExpr *MasmParser::evaluateBuiltinValue(BuiltinSymbol Symbol,
                                               SMLoc StartLoc) {
  switch (Symbol) {
  default:
    return nullptr;
  case BI_VERSION:
    // Match a recent version of ML.EXE.
    return MCConstantExpr::create(1427, getContext());
  case BI_LINE: {
    int64_t Line;
    if (ActiveMacros.empty())
      Line = SrcMgr.FindLineNumber(StartLoc, CurBuffer);
    else
      Line = SrcMgr.FindLineNumber(ActiveMacros.front()->InstantiationLoc,
                                   ActiveMacros.front()->ExitBuffer);
    return MCConstantExpr::create(Line, getContext());
  }
  }
  llvm_unreachable("unhandled built-in symbol");
}

```
- **EN**: Implements logic around `evaluateBuiltinValue`, `create`, `FindLineNumber`, `front`, and 1 more symbols; this block parses assembly syntax or operands; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `evaluateBuiltinValue`, `create`, `FindLineNumber`, `front`, and 1 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 6193-6225
```cpp
std::optional<std::string>
MasmParser::evaluateBuiltinTextMacro(BuiltinSymbol Symbol, SMLoc StartLoc) {
  switch (Symbol) {
  default:
    return {};
  case BI_DATE: {
    // Current local date, formatted MM/DD/YY
    char TmpBuffer[sizeof("mm/dd/yy")];
    const size_t Len = strftime(TmpBuffer, sizeof(TmpBuffer), "%D", &TM);
    return std::string(TmpBuffer, Len);
  }
  case BI_TIME: {
    // Current local time, formatted HH:MM:SS (24-hour clock)
    char TmpBuffer[sizeof("hh:mm:ss")];
    const size_t Len = strftime(TmpBuffer, sizeof(TmpBuffer), "%T", &TM);
    return std::string(TmpBuffer, Len);
  }
  case BI_FILECUR:
    return SrcMgr
        .getMemoryBuffer(
            ActiveMacros.empty() ? CurBuffer : ActiveMacros.front()->ExitBuffer)
        ->getBufferIdentifier()
        .str();
  case BI_FILENAME:
    return sys::path::stem(SrcMgr.getMemoryBuffer(SrcMgr.getMainFileID())
                               ->getBufferIdentifier())
        .upper();
  case BI_CURSEG:
    return getStreamer().getCurrentSectionOnly()->getName().str();
  }
  llvm_unreachable("unhandled built-in symbol");
}

```
- **EN**: Implements logic around `evaluateBuiltinTextMacro`, `strftime`, `string`, `getMemoryBuffer`, and 7 more symbols; this block parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `evaluateBuiltinTextMacro`, `strftime`, `string`, `getMemoryBuffer`, and 7 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 6226-6247
```cpp
bool MasmParser::evaluateBuiltinMacroFunction(BuiltinFunction Function,
                                              StringRef Name,
                                              std::string &Res) {
  if (parseToken(AsmToken::LParen, "invoking macro function '" + Name +
                                       "' requires arguments in parentheses")) {
    return true;
  }

  MCAsmMacroParameters P;
  switch (Function) {
  default:
    return true;
  case BI_CATSTR:
    break;
  }
  MCAsmMacro M(Name, "", P, {}, true);

  MCAsmMacroArguments A;
  if (parseMacroArguments(&M, A, AsmToken::RParen) || parseRParen()) {
    return true;
  }

```
- **EN**: Implements logic around `evaluateBuiltinMacroFunction`, `M`; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `evaluateBuiltinMacroFunction`, `M` 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 6248-6267
```cpp
  switch (Function) {
  default:
    llvm_unreachable("unhandled built-in function");
  case BI_CATSTR: {
    for (const MCAsmMacroArgument &Arg : A) {
      for (const AsmToken &Tok : Arg) {
        if (Tok.is(AsmToken::String)) {
          Res.append(Tok.getStringContents());
        } else {
          Res.append(Tok.getString());
        }
      }
    }
    return false;
  }
  }
  llvm_unreachable("unhandled built-in function");
  return true;
}

```
- **EN**: Implements logic around `llvm_unreachable`, `append`; this block parses assembly syntax or operands; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `llvm_unreachable`, `append` 实现具体逻辑；这一段解析汇编语法或操作数，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 6268-6273
```cpp
/// Create an MCAsmParser instance.
MCAsmParser *llvm::createMCMasmParser(SourceMgr &SM, MCContext &C,
                                      MCStreamer &Out, const MCAsmInfo &MAI,
                                      struct tm TM, unsigned CB) {
  return new MasmParser(SM, C, Out, MAI, TM, CB);
}
```
- **EN**: Introduces declarations for `tm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `tm` 等声明，定义本文件后续使用的数据结构或接口。

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
- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据
- **Object layout / 目标文件布局**:
  - **EN**: Organizes fragments into sections and computes final offsets
  - **CN**: 把片段组织到节区中并计算最终偏移

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ADT/APFloat.h`, `llvm/ADT/APInt.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/BitVector.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/StringMap.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/StringSwitch.h`, `llvm/ADT/Twine.h` ... (+44 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support
