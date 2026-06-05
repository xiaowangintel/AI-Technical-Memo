# MLIRServer.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Tools/mlir-lsp-server/MLIRServer.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements reusable command-line tool support built on top of MLIR libraries.
  - **CN**: 实现构建于 MLIR 库之上的可复用命令行工具支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-27
```cpp
//===- MLIRServer.cpp - MLIR Generic Language Server ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "MLIRServer.h"
#include "Protocol.h"
#include "mlir/AsmParser/AsmParser.h"
#include "mlir/AsmParser/AsmParserState.h"
#include "mlir/AsmParser/CodeComplete.h"
#include "mlir/Bytecode/BytecodeWriter.h"
#include "mlir/IR/Operation.h"
#include "mlir/Interfaces/FunctionInterfaces.h"
#include "mlir/Parser/Parser.h"
#include "mlir/Support/ToolUtilities.h"
#include "mlir/Tools/lsp-server-support/SourceMgrUtils.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/Support/Base64.h"
#include "llvm/Support/FileSystem.h"
#include "llvm/Support/LSP/Logging.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/SourceMgr.h"
#include <optional>

```
- **EN**: Pulls in the declarations needed by this translation unit, including `MLIRServer.h`, `Protocol.h`, `mlir/AsmParser/AsmParser.h`, `mlir/AsmParser/AsmParserState.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `MLIRServer.h`, `Protocol.h`, `mlir/AsmParser/AsmParser.h`, `mlir/AsmParser/AsmParserState.h`。

### Lines 28-55
```cpp
using namespace mlir;

/// Returns the range of a lexical token given a SMLoc corresponding to the
/// start of an token location. The range is computed heuristically, and
/// supports identifier-like tokens, strings, etc.
static SMRange convertTokenLocToRange(SMLoc loc) {
  return lsp::convertTokenLocToRange(loc, "$-.");
}

/// Returns a language server location from the given MLIR file location.
/// `uriScheme` is the scheme to use when building new uris.
static std::optional<lsp::Location>
getLocationFromLoc(StringRef uriScheme, FileLineColLoc loc,
                   StringRef workspaceRoot) {
  StringRef filename = loc.getFilename();
  SmallString<128> absPath;
  // Always make the path absolute. Skip paths that start with a separator:
  // prevents incorrect resolution of virtual paths used in tests on Windows.
  if (!llvm::sys::path::is_absolute(filename) && !filename.starts_with("/") &&
      !filename.starts_with("\\")) {
    if (!workspaceRoot.empty())
      llvm::sys::path::append(absPath, workspaceRoot, filename);
    else
      absPath = filename;
    llvm::sys::fs::make_absolute(absPath);
    filename = absPath;
  }

```
- **EN**: Implements logic around `convertTokenLocToRange`, `getLocationFromLoc`, `getFilename`, `is_absolute`, and 4 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `convertTokenLocToRange`、`getLocationFromLoc`、`getFilename`、`is_absolute` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 56-73
```cpp
  llvm::Expected<lsp::URIForFile> sourceURI =
      lsp::URIForFile::fromFile(filename, uriScheme);
  if (!sourceURI) {
    llvm::lsp::Logger::error("Failed to create URI for file `{0}`: {1}",
                             filename, llvm::toString(sourceURI.takeError()));
    return std::nullopt;
  }

  lsp::Position position;
  position.line = loc.getLine() - 1;
  position.character = loc.getColumn() ? loc.getColumn() - 1 : 0;
  return lsp::Location{*sourceURI, lsp::Range(position)};
}

/// Returns a language server location from the given MLIR location, or
/// std::nullopt if one couldn't be created. `uriScheme` is the scheme to use
/// when building new uris. `uri` is an optional additional filter that, when
/// present, is used to filter sub locations that do not share the same uri.
```
- **EN**: Implements logic around `fromFile`, `error`, `toString`, `getLine`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `fromFile`、`error`、`toString`、`getLine` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 74-105
```cpp
static std::optional<lsp::Location>
getLocationFromLoc(llvm::SourceMgr &sourceMgr, Location loc,
                   StringRef uriScheme, StringRef workspaceRoot,
                   const lsp::URIForFile *uri = nullptr) {
  std::optional<lsp::Location> location;
  loc->walk([&](Location nestedLoc) {
    auto fileLoc = dyn_cast<FileLineColLoc>(nestedLoc);
    if (!fileLoc)
      return WalkResult::advance();

    std::optional<lsp::Location> sourceLoc =
        getLocationFromLoc(uriScheme, fileLoc, workspaceRoot);
    if (sourceLoc && (!uri || sourceLoc->uri == *uri)) {
      location = *sourceLoc;
      SMLoc loc = sourceMgr.FindLocForLineAndColumn(
          sourceMgr.getMainFileID(), fileLoc.getLine(), fileLoc.getColumn());

      // Use range of potential identifier starting at location, else length 1
      // range.
      location->range.end.character += 1;
      if (std::optional<SMRange> range = convertTokenLocToRange(loc)) {
        auto lineCol = sourceMgr.getLineAndColumn(range->End);
        location->range.end.character =
            std::max(fileLoc.getColumn() + 1, lineCol.second - 1);
      }
      return WalkResult::interrupt();
    }
    return WalkResult::advance();
  });
  return location;
}

```
- **EN**: Implements logic around `getLocationFromLoc`, `walk`, `dyn_cast`, `advance`, and 6 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getLocationFromLoc`、`walk`、`dyn_cast`、`advance` 等另外 6 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 106-125
```cpp
/// Collect all of the locations from the given MLIR location that are not
/// contained within the given URI.
static void collectLocationsFromLoc(Location loc,
                                    std::vector<lsp::Location> &locations,
                                    const lsp::URIForFile &uri,
                                    StringRef workspaceRoot) {
  SetVector<Location> visitedLocs;
  loc->walk([&](Location nestedLoc) {
    FileLineColLoc fileLoc = dyn_cast<FileLineColLoc>(nestedLoc);
    if (!fileLoc || !visitedLocs.insert(nestedLoc))
      return WalkResult::advance();

    std::optional<lsp::Location> sourceLoc =
        getLocationFromLoc(uri.scheme(), fileLoc, workspaceRoot);
    if (sourceLoc && sourceLoc->uri != uri)
      locations.push_back(*sourceLoc);
    return WalkResult::advance();
  });
}

```
- **EN**: Implements logic around `collectLocationsFromLoc`, `walk`, `dyn_cast`, `insert`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `collectLocationsFromLoc`、`walk`、`dyn_cast`、`insert` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 126-145
```cpp
/// Returns true if the given range contains the given source location. Note
/// that this has slightly different behavior than SMRange because it is
/// inclusive of the end location.
static bool contains(SMRange range, SMLoc loc) {
  return range.Start.getPointer() <= loc.getPointer() &&
         loc.getPointer() <= range.End.getPointer();
}

/// Returns true if the given location is contained by the definition or one of
/// the uses of the given SMDefinition. If provided, `overlappedRange` is set to
/// the range within `def` that the provided `loc` overlapped with.
static bool isDefOrUse(const AsmParserState::SMDefinition &def, SMLoc loc,
                       SMRange *overlappedRange = nullptr) {
  // Check the main definition.
  if (contains(def.loc, loc)) {
    if (overlappedRange)
      *overlappedRange = def.loc;
    return true;
  }

```
- **EN**: Implements logic around `contains`, `getPointer`, `isDefOrUse`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `contains`、`getPointer`、`isDefOrUse` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 146-168
```cpp
  // Check the uses.
  const auto *useIt = llvm::find_if(
      def.uses, [&](const SMRange &range) { return contains(range, loc); });
  if (useIt != def.uses.end()) {
    if (overlappedRange)
      *overlappedRange = *useIt;
    return true;
  }
  return false;
}

/// Given a location pointing to a result, return the result number it refers
/// to or std::nullopt if it refers to all of the results.
static std::optional<unsigned> getResultNumberFromLoc(SMLoc loc) {
  // Skip all of the identifier characters.
  auto isIdentifierChar = [](char c) {
    return isalnum(c) || c == '%' || c == '$' || c == '.' || c == '_' ||
           c == '-';
  };
  const char *curPtr = loc.getPointer();
  while (isIdentifierChar(*curPtr))
    ++curPtr;

```
- **EN**: Implements logic around `find_if`, `contains`, `end`, `getResultNumberFromLoc`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `find_if`、`contains`、`end`、`getResultNumberFromLoc` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 169-192
```cpp
  // Check to see if this location indexes into the result group, via `#`. If it
  // doesn't, we can't extract a sub result number.
  if (*curPtr != '#')
    return std::nullopt;

  // Compute the sub result number from the remaining portion of the string.
  const char *numberStart = ++curPtr;
  while (llvm::isDigit(*curPtr))
    ++curPtr;
  StringRef numberStr(numberStart, curPtr - numberStart);
  unsigned resultNumber = 0;
  return numberStr.consumeInteger(10, resultNumber) ? std::optional<unsigned>()
                                                    : resultNumber;
}

/// Given a source location range, return the text covered by the given range.
/// If the range is invalid, returns std::nullopt.
static std::optional<StringRef> getTextFromRange(SMRange range) {
  if (!range.isValid())
    return std::nullopt;
  const char *startPtr = range.Start.getPointer();
  return StringRef(startPtr, range.End.getPointer() - startPtr);
}

```
- **EN**: Implements logic around `isDigit`, `numberStr`, `consumeInteger`, `getTextFromRange`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `isDigit`、`numberStr`、`consumeInteger`、`getTextFromRange` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 193-210
```cpp
/// Given a block and source location, print the source name of the block to the
/// given output stream.
static void printDefBlockName(raw_ostream &os, Block *block, SMRange loc = {}) {
  // Try to extract a name from the source location.
  std::optional<StringRef> text = getTextFromRange(loc);
  if (text && text->starts_with("^")) {
    os << *text;
    return;
  }

  // Otherwise, we don't have a name so print the block number.
  os << "<Block #" << block->computeBlockNumber() << ">";
}
static void printDefBlockName(raw_ostream &os,
                              const AsmParserState::BlockDefinition &def) {
  printDefBlockName(os, def.block, def.definition.loc);
}

```
- **EN**: Implements logic around `printDefBlockName`, `getTextFromRange`, `starts_with`, `computeBlockNumber`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `printDefBlockName`、`getTextFromRange`、`starts_with`、`computeBlockNumber` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 211-231
```cpp
/// Convert the given MLIR diagnostic to the LSP form.
static lsp::Diagnostic getLspDiagnoticFromDiag(llvm::SourceMgr &sourceMgr,
                                               Diagnostic &diag,
                                               const lsp::URIForFile &uri,
                                               StringRef workspaceRoot) {
  lsp::Diagnostic lspDiag;
  lspDiag.source = "mlir";

  // Note: Right now all of the diagnostics are treated as parser issues, but
  // some are parser and some are verifier.
  lspDiag.category = "Parse Error";

  // Try to grab a file location for this diagnostic.
  // TODO: For simplicity, we just grab the first one. It may be likely that we
  // will need a more interesting heuristic here.'
  StringRef uriScheme = uri.scheme();
  std::optional<lsp::Location> lspLocation = getLocationFromLoc(
      sourceMgr, diag.getLocation(), uriScheme, workspaceRoot, &uri);
  if (lspLocation)
    lspDiag.range = lspLocation->range;

```
- **EN**: Implements logic around `getLspDiagnoticFromDiag`, `scheme`, `getLocationFromLoc`, `getLocation`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `getLspDiagnoticFromDiag`、`scheme`、`getLocationFromLoc`、`getLocation` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 232-261
```cpp
  // Convert the severity for the diagnostic.
  switch (diag.getSeverity()) {
  case mlir::DiagnosticSeverity::Note:
    llvm_unreachable("expected notes to be handled separately");
  case mlir::DiagnosticSeverity::Warning:
    lspDiag.severity = llvm::lsp::DiagnosticSeverity::Warning;
    break;
  case mlir::DiagnosticSeverity::Error:
    lspDiag.severity = llvm::lsp::DiagnosticSeverity::Error;
    break;
  case mlir::DiagnosticSeverity::Remark:
    lspDiag.severity = llvm::lsp::DiagnosticSeverity::Information;
    break;
  }
  lspDiag.message = diag.str();

  // Attach any notes to the main diagnostic as related information.
  std::vector<llvm::lsp::DiagnosticRelatedInformation> relatedDiags;
  for (Diagnostic &note : diag.getNotes()) {
    lsp::Location noteLoc;
    if (std::optional<lsp::Location> loc = getLocationFromLoc(
            sourceMgr, note.getLocation(), uriScheme, workspaceRoot))
      noteLoc = *loc;
    else
      noteLoc.uri = uri;
    relatedDiags.emplace_back(noteLoc, note.str());
  }
  if (!relatedDiags.empty())
    lspDiag.relatedInformation = std::move(relatedDiags);

```
- **EN**: Implements logic around `getSeverity`, `llvm_unreachable`, `str`, `getNotes`, and 5 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects.
- **CN**: 围绕 `getSeverity`、`llvm_unreachable`、`str`、`getNotes` 等另外 5 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象。

### Lines 262-279
```cpp
  return lspDiag;
}

//===----------------------------------------------------------------------===//
// MLIRDocument
//===----------------------------------------------------------------------===//

namespace {
/// This class represents all of the information pertaining to a specific MLIR
/// document.
struct MLIRDocument {
  MLIRDocument(MLIRContext &context, const lsp::URIForFile &uri,
               StringRef contents, StringRef workspaceRoot,
               std::vector<lsp::Diagnostic> &diagnostics);
  MLIRDocument(const MLIRDocument &) = delete;
  MLIRDocument &operator=(const MLIRDocument &) = delete;

  //===--------------------------------------------------------------------===//
```
- **EN**: Introduces declarations for `MLIRDocument`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `MLIRDocument` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 280-305
```cpp
  // Definitions and References
  //===--------------------------------------------------------------------===//

  void getLocationsOf(const lsp::URIForFile &uri, const lsp::Position &defPos,
                      std::vector<lsp::Location> &locations);
  void findReferencesOf(const lsp::URIForFile &uri, const lsp::Position &pos,
                        std::vector<lsp::Location> &references);

  //===--------------------------------------------------------------------===//
  // Hover
  //===--------------------------------------------------------------------===//

  std::optional<lsp::Hover> findHover(const lsp::URIForFile &uri,
                                      const lsp::Position &hoverPos);
  std::optional<lsp::Hover>
  buildHoverForOperation(SMRange hoverRange,
                         const AsmParserState::OperationDefinition &op);
  lsp::Hover buildHoverForOperationResult(SMRange hoverRange, Operation *op,
                                          unsigned resultStart,
                                          unsigned resultEnd, SMLoc posLoc);
  lsp::Hover buildHoverForBlock(SMRange hoverRange,
                                const AsmParserState::BlockDefinition &block);
  lsp::Hover
  buildHoverForBlockArgument(SMRange hoverRange, BlockArgument arg,
                             const AsmParserState::BlockDefinition &block);

```
- **EN**: Implements logic around `getLocationsOf`, `findReferencesOf`, `findHover`, `buildHoverForOperation`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getLocationsOf`、`findReferencesOf`、`findHover`、`buildHoverForOperation` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 306-323
```cpp
  lsp::Hover buildHoverForAttributeAlias(
      SMRange hoverRange, const AsmParserState::AttributeAliasDefinition &attr);
  lsp::Hover
  buildHoverForTypeAlias(SMRange hoverRange,
                         const AsmParserState::TypeAliasDefinition &type);

  //===--------------------------------------------------------------------===//
  // Document Symbols
  //===--------------------------------------------------------------------===//

  void findDocumentSymbols(std::vector<lsp::DocumentSymbol> &symbols);
  void findDocumentSymbols(Operation *op,
                           std::vector<lsp::DocumentSymbol> &symbols);

  //===--------------------------------------------------------------------===//
  // Code Completion
  //===--------------------------------------------------------------------===//

```
- **EN**: Implements logic around `buildHoverForAttributeAlias`, `buildHoverForTypeAlias`, `findDocumentSymbols`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `buildHoverForAttributeAlias`、`buildHoverForTypeAlias`、`findDocumentSymbols` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 324-342
```cpp
  lsp::CompletionList getCodeCompletion(const lsp::URIForFile &uri,
                                        const lsp::Position &completePos,
                                        const DialectRegistry &registry);

  //===--------------------------------------------------------------------===//
  // Code Action
  //===--------------------------------------------------------------------===//

  void getCodeActionForDiagnostic(const lsp::URIForFile &uri,
                                  lsp::Position &pos, StringRef severity,
                                  StringRef message,
                                  std::vector<llvm::lsp::TextEdit> &edits);

  //===--------------------------------------------------------------------===//
  // Bytecode
  //===--------------------------------------------------------------------===//

  llvm::Expected<lsp::MLIRConvertBytecodeResult> convertToBytecode();

```
- **EN**: Implements logic around `getCodeCompletion`, `getCodeActionForDiagnostic`, `convertToBytecode`.
- **CN**: 围绕 `getCodeCompletion`、`getCodeActionForDiagnostic`、`convertToBytecode` 实现具体逻辑。

### Lines 343-360
```cpp
  //===--------------------------------------------------------------------===//
  // Fields
  //===--------------------------------------------------------------------===//

  /// The high level parser state used to find definitions and references within
  /// the source file.
  AsmParserState asmState;

  /// The container for the IR parsed from the input file.
  Block parsedIR;

  /// A collection of external resources, which we want to propagate up to the
  /// user.
  FallbackAsmResourceMap fallbackResourceMap;

  /// The source manager containing the contents of the input file.
  llvm::SourceMgr sourceMgr;

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 361-382
```cpp
  /// The workspace root of the server.
  std::string workspaceRoot;
};
} // namespace

MLIRDocument::MLIRDocument(MLIRContext &context, const lsp::URIForFile &uri,
                           StringRef contents, StringRef workspaceRoot,
                           std::vector<lsp::Diagnostic> &diagnostics)
    : workspaceRoot(workspaceRoot.str()) {
  ScopedDiagnosticHandler handler(&context, [&](Diagnostic &diag) {
    diagnostics.push_back(
        getLspDiagnoticFromDiag(sourceMgr, diag, uri, workspaceRoot));
  });

  // Try to parsed the given IR string.
  auto memBuffer = llvm::MemoryBuffer::getMemBufferCopy(contents, uri.file());
  if (!memBuffer) {
    llvm::lsp::Logger::error("Failed to create memory buffer for file",
                             uri.file());
    return;
  }

```
- **EN**: Implements logic around `workspaceRoot`, `handler`, `push_back`, `getLspDiagnoticFromDiag`, and 3 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; parses or prints textual MLIR representations.
- **CN**: 围绕 `workspaceRoot`、`handler`、`push_back`、`getLspDiagnoticFromDiag` 等另外 3 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并解析或打印文本形式的 MLIR 表示。

### Lines 383-403
```cpp
  ParserConfig config(&context, /*verifyAfterParse=*/true,
                      &fallbackResourceMap);
  sourceMgr.AddNewSourceBuffer(std::move(memBuffer), SMLoc());
  if (failed(parseAsmSourceFile(sourceMgr, &parsedIR, config, &asmState))) {
    // If parsing failed, clear out any of the current state.
    parsedIR.clear();
    asmState = AsmParserState();
    fallbackResourceMap = FallbackAsmResourceMap();
    return;
  }
}

//===----------------------------------------------------------------------===//
// MLIRDocument: Definitions and References
//===----------------------------------------------------------------------===//

void MLIRDocument::getLocationsOf(const lsp::URIForFile &uri,
                                  const lsp::Position &defPos,
                                  std::vector<lsp::Location> &locations) {
  SMLoc posLoc = defPos.getAsSMLoc(sourceMgr);

```
- **EN**: Implements logic around `config`, `AddNewSourceBuffer`, `failed`, `clear`, and 4 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `config`、`AddNewSourceBuffer`、`failed`、`clear` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 404-429
```cpp
  // Functor used to check if an SM definition contains the position.
  auto containsPosition = [&](const AsmParserState::SMDefinition &def) {
    if (!isDefOrUse(def, posLoc))
      return false;
    locations.emplace_back(uri, sourceMgr, def.loc);
    return true;
  };

  // Check all definitions related to operations.
  for (const AsmParserState::OperationDefinition &op : asmState.getOpDefs()) {
    if (contains(op.loc, posLoc))
      return collectLocationsFromLoc(op.op->getLoc(), locations, uri,
                                     workspaceRoot);
    for (const auto &result : op.resultGroups)
      if (containsPosition(result.definition))
        return collectLocationsFromLoc(op.op->getLoc(), locations, uri,
                                       workspaceRoot);
    for (const auto &symUse : op.symbolUses) {
      if (contains(symUse, posLoc)) {
        locations.emplace_back(uri, sourceMgr, op.loc);
        return collectLocationsFromLoc(op.op->getLoc(), locations, uri,
                                       workspaceRoot);
      }
    }
  }

```
- **EN**: Implements logic around `isDefOrUse`, `emplace_back`, `getOpDefs`, `contains`, and 2 more symbols.
- **CN**: 围绕 `isDefOrUse`、`emplace_back`、`getOpDefs`、`contains` 等另外 2 个符号 实现具体逻辑。

### Lines 430-451
```cpp
  // Check all definitions related to blocks.
  for (const AsmParserState::BlockDefinition &block : asmState.getBlockDefs()) {
    if (containsPosition(block.definition))
      return;
    for (const AsmParserState::SMDefinition &arg : block.arguments)
      if (containsPosition(arg))
        return;
  }

  // Check all alias definitions.
  for (const AsmParserState::AttributeAliasDefinition &attr :
       asmState.getAttributeAliasDefs()) {
    if (containsPosition(attr.definition))
      return;
  }
  for (const AsmParserState::TypeAliasDefinition &type :
       asmState.getTypeAliasDefs()) {
    if (containsPosition(type.definition))
      return;
  }
}

```
- **EN**: Implements logic around `getBlockDefs`, `containsPosition`, `getAttributeAliasDefs`, `getTypeAliasDefs`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getBlockDefs`、`containsPosition`、`getAttributeAliasDefs`、`getTypeAliasDefs` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 452-486
```cpp
void MLIRDocument::findReferencesOf(const lsp::URIForFile &uri,
                                    const lsp::Position &pos,
                                    std::vector<lsp::Location> &references) {
  // Functor used to append all of the definitions/uses of the given SM
  // definition to the reference list.
  auto appendSMDef = [&](const AsmParserState::SMDefinition &def) {
    references.emplace_back(uri, sourceMgr, def.loc);
    for (const SMRange &use : def.uses)
      references.emplace_back(uri, sourceMgr, use);
  };

  SMLoc posLoc = pos.getAsSMLoc(sourceMgr);

  // Check all definitions related to operations.
  for (const AsmParserState::OperationDefinition &op : asmState.getOpDefs()) {
    if (contains(op.loc, posLoc)) {
      for (const auto &result : op.resultGroups)
        appendSMDef(result.definition);
      for (const auto &symUse : op.symbolUses)
        if (contains(symUse, posLoc))
          references.emplace_back(uri, sourceMgr, symUse);
      return;
    }
    for (const auto &result : op.resultGroups)
      if (isDefOrUse(result.definition, posLoc))
        return appendSMDef(result.definition);
    for (const auto &symUse : op.symbolUses) {
      if (!contains(symUse, posLoc))
        continue;
      for (const auto &symUse : op.symbolUses)
        references.emplace_back(uri, sourceMgr, symUse);
      return;
    }
  }

```
- **EN**: Implements logic around `findReferencesOf`, `emplace_back`, `getAsSMLoc`, `getOpDefs`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `findReferencesOf`、`emplace_back`、`getAsSMLoc`、`getOpDefs` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 487-509
```cpp
  // Check all definitions related to blocks.
  for (const AsmParserState::BlockDefinition &block : asmState.getBlockDefs()) {
    if (isDefOrUse(block.definition, posLoc))
      return appendSMDef(block.definition);

    for (const AsmParserState::SMDefinition &arg : block.arguments)
      if (isDefOrUse(arg, posLoc))
        return appendSMDef(arg);
  }

  // Check all alias definitions.
  for (const AsmParserState::AttributeAliasDefinition &attr :
       asmState.getAttributeAliasDefs()) {
    if (isDefOrUse(attr.definition, posLoc))
      return appendSMDef(attr.definition);
  }
  for (const AsmParserState::TypeAliasDefinition &type :
       asmState.getTypeAliasDefs()) {
    if (isDefOrUse(type.definition, posLoc))
      return appendSMDef(type.definition);
  }
}

```
- **EN**: Implements logic around `getBlockDefs`, `isDefOrUse`, `appendSMDef`, `getAttributeAliasDefs`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getBlockDefs`、`isDefOrUse`、`appendSMDef`、`getAttributeAliasDefs` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 510-530
```cpp
//===----------------------------------------------------------------------===//
// MLIRDocument: Hover
//===----------------------------------------------------------------------===//

std::optional<lsp::Hover>
MLIRDocument::findHover(const lsp::URIForFile &uri,
                        const lsp::Position &hoverPos) {
  SMLoc posLoc = hoverPos.getAsSMLoc(sourceMgr);
  SMRange hoverRange;

  // Check for Hovers on operations and results.
  for (const AsmParserState::OperationDefinition &op : asmState.getOpDefs()) {
    // Check if the position points at this operation.
    if (contains(op.loc, posLoc))
      return buildHoverForOperation(op.loc, op);

    // Check if the position points at the symbol name.
    for (auto &use : op.symbolUses)
      if (contains(use, posLoc))
        return buildHoverForOperation(use, op);

```
- **EN**: Implements logic around `findHover`, `getAsSMLoc`, `getOpDefs`, `contains`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `findHover`、`getAsSMLoc`、`getOpDefs`、`contains` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 531-550
```cpp
    // Check if the position points at a result group.
    for (unsigned i = 0, e = op.resultGroups.size(); i < e; ++i) {
      const auto &result = op.resultGroups[i];
      if (!isDefOrUse(result.definition, posLoc, &hoverRange))
        continue;

      // Get the range of results covered by the over position.
      unsigned resultStart = result.startIndex;
      unsigned resultEnd = (i == e - 1) ? op.op->getNumResults()
                                        : op.resultGroups[i + 1].startIndex;
      return buildHoverForOperationResult(hoverRange, op.op, resultStart,
                                          resultEnd, posLoc);
    }
  }

  // Check to see if the hover is over a block argument.
  for (const AsmParserState::BlockDefinition &block : asmState.getBlockDefs()) {
    if (isDefOrUse(block.definition, posLoc, &hoverRange))
      return buildHoverForBlock(hoverRange, block);

```
- **EN**: Implements logic around `size`, `isDefOrUse`, `getNumResults`, `buildHoverForOperationResult`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `size`、`isDefOrUse`、`getNumResults`、`buildHoverForOperationResult` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 551-571
```cpp
    for (const auto &arg : llvm::enumerate(block.arguments)) {
      if (!isDefOrUse(arg.value(), posLoc, &hoverRange))
        continue;

      return buildHoverForBlockArgument(
          hoverRange, block.block->getArgument(arg.index()), block);
    }
  }

  // Check to see if the hover is over an alias.
  for (const AsmParserState::AttributeAliasDefinition &attr :
       asmState.getAttributeAliasDefs()) {
    if (isDefOrUse(attr.definition, posLoc, &hoverRange))
      return buildHoverForAttributeAlias(hoverRange, attr);
  }
  for (const AsmParserState::TypeAliasDefinition &type :
       asmState.getTypeAliasDefs()) {
    if (isDefOrUse(type.definition, posLoc, &hoverRange))
      return buildHoverForTypeAlias(hoverRange, type);
  }

```
- **EN**: Implements logic around `enumerate`, `isDefOrUse`, `buildHoverForBlockArgument`, `getArgument`, and 4 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `enumerate`、`isDefOrUse`、`buildHoverForBlockArgument`、`getArgument` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 572-593
```cpp
  return std::nullopt;
}

std::optional<lsp::Hover> MLIRDocument::buildHoverForOperation(
    SMRange hoverRange, const AsmParserState::OperationDefinition &op) {
  lsp::Hover hover(lsp::Range(sourceMgr, hoverRange));
  llvm::raw_string_ostream os(hover.contents.value);

  // Add the operation name to the hover.
  os << "\"" << op.op->getName() << "\"";
  if (SymbolOpInterface symbol = dyn_cast<SymbolOpInterface>(op.op))
    os << " : " << symbol.getVisibility() << " @" << symbol.getName() << "";
  os << "\n\n";

  os << "Generic Form:\n\n```mlir\n";

  op.op->print(os, OpPrintingFlags()
                       .printGenericOpForm()
                       .elideLargeElementsAttrs()
                       .skipRegions());
  os << "\n```\n";

```
- **EN**: Implements logic around `buildHoverForOperation`, `hover`, `os`, `getName`, and 6 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `buildHoverForOperation`、`hover`、`os`、`getName` 等另外 6 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 594-616
```cpp
  return hover;
}

lsp::Hover MLIRDocument::buildHoverForOperationResult(SMRange hoverRange,
                                                      Operation *op,
                                                      unsigned resultStart,
                                                      unsigned resultEnd,
                                                      SMLoc posLoc) {
  lsp::Hover hover(lsp::Range(sourceMgr, hoverRange));
  llvm::raw_string_ostream os(hover.contents.value);

  // Add the parent operation name to the hover.
  os << "Operation: \"" << op->getName() << "\"\n\n";

  // Check to see if the location points to a specific result within the
  // group.
  if (std::optional<unsigned> resultNumber = getResultNumberFromLoc(posLoc)) {
    if ((resultStart + *resultNumber) < resultEnd) {
      resultStart += *resultNumber;
      resultEnd = resultStart + 1;
    }
  }

```
- **EN**: Implements logic around `buildHoverForOperationResult`, `hover`, `os`, `getName`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `buildHoverForOperationResult`、`hover`、`os`、`getName` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 617-637
```cpp
  // Add the range of results and their types to the hover info.
  if ((resultStart + 1) == resultEnd) {
    os << "Result #" << resultStart << "\n\n"
       << "Type: `" << op->getResult(resultStart).getType() << "`\n\n";
  } else {
    os << "Result #[" << resultStart << ", " << (resultEnd - 1) << "]\n\n"
       << "Types: ";
    llvm::interleaveComma(
        op->getResults().slice(resultStart, resultEnd), os,
        [&](Value result) { os << "`" << result.getType() << "`"; });
  }

  return hover;
}

lsp::Hover
MLIRDocument::buildHoverForBlock(SMRange hoverRange,
                                 const AsmParserState::BlockDefinition &block) {
  lsp::Hover hover(lsp::Range(sourceMgr, hoverRange));
  llvm::raw_string_ostream os(hover.contents.value);

```
- **EN**: Implements logic around `getResult`, `interleaveComma`, `getResults`, `getType`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getResult`、`interleaveComma`、`getResults`、`getType` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 638-660
```cpp
  // Print the given block to the hover output stream.
  auto printBlockToHover = [&](Block *newBlock) {
    if (const auto *def = asmState.getBlockDef(newBlock))
      printDefBlockName(os, *def);
    else
      printDefBlockName(os, newBlock);
  };

  // Display the parent operation, block number, predecessors, and successors.
  os << "Operation: \"" << block.block->getParentOp()->getName() << "\"\n\n"
     << "Block #" << block.block->computeBlockNumber() << "\n\n";
  if (!block.block->hasNoPredecessors()) {
    os << "Predecessors: ";
    llvm::interleaveComma(block.block->getPredecessors(), os,
                          printBlockToHover);
    os << "\n\n";
  }
  if (!block.block->hasNoSuccessors()) {
    os << "Successors: ";
    llvm::interleaveComma(block.block->getSuccessors(), os, printBlockToHover);
    os << "\n\n";
  }

```
- **EN**: Implements logic around `getBlockDef`, `printDefBlockName`, `getParentOp`, `computeBlockNumber`, and 3 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `getBlockDef`、`printDefBlockName`、`getParentOp`、`computeBlockNumber` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 661-679
```cpp
  return hover;
}

lsp::Hover MLIRDocument::buildHoverForBlockArgument(
    SMRange hoverRange, BlockArgument arg,
    const AsmParserState::BlockDefinition &block) {
  lsp::Hover hover(lsp::Range(sourceMgr, hoverRange));
  llvm::raw_string_ostream os(hover.contents.value);

  // Display the parent operation, block, the argument number, and the type.
  os << "Operation: \"" << block.block->getParentOp()->getName() << "\"\n\n"
     << "Block: ";
  printDefBlockName(os, block);
  os << "\n\nArgument #" << arg.getArgNumber() << "\n\n"
     << "Type: `" << arg.getType() << "`\n\n";

  return hover;
}

```
- **EN**: Implements logic around `buildHoverForBlockArgument`, `hover`, `os`, `getParentOp`, and 3 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `buildHoverForBlockArgument`、`hover`、`os`、`getParentOp` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 680-698
```cpp
lsp::Hover MLIRDocument::buildHoverForAttributeAlias(
    SMRange hoverRange, const AsmParserState::AttributeAliasDefinition &attr) {
  lsp::Hover hover(lsp::Range(sourceMgr, hoverRange));
  llvm::raw_string_ostream os(hover.contents.value);

  os << "Attribute Alias: \"" << attr.name << "\n\n";
  os << "Value: ```mlir\n" << attr.value << "\n```\n\n";

  return hover;
}

lsp::Hover MLIRDocument::buildHoverForTypeAlias(
    SMRange hoverRange, const AsmParserState::TypeAliasDefinition &type) {
  lsp::Hover hover(lsp::Range(sourceMgr, hoverRange));
  llvm::raw_string_ostream os(hover.contents.value);

  os << "Type Alias: \"" << type.name << "\n\n";
  os << "Value: ```mlir\n" << type.value << "\n```\n\n";

```
- **EN**: Implements logic around `buildHoverForAttributeAlias`, `hover`, `os`, `buildHoverForTypeAlias`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `buildHoverForAttributeAlias`、`hover`、`os`、`buildHoverForTypeAlias` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 699-727
```cpp
  return hover;
}

//===----------------------------------------------------------------------===//
// MLIRDocument: Document Symbols
//===----------------------------------------------------------------------===//

void MLIRDocument::findDocumentSymbols(
    std::vector<lsp::DocumentSymbol> &symbols) {
  for (Operation &op : parsedIR)
    findDocumentSymbols(&op, symbols);
}

void MLIRDocument::findDocumentSymbols(
    Operation *op, std::vector<lsp::DocumentSymbol> &symbols) {
  std::vector<lsp::DocumentSymbol> *childSymbols = &symbols;

  // Check for the source information of this operation.
  if (const AsmParserState::OperationDefinition *def = asmState.getOpDef(op)) {
    // If this operation defines a symbol, record it.
    if (SymbolOpInterface symbol = dyn_cast<SymbolOpInterface>(op)) {
      symbols.emplace_back(symbol.getName(),
                           isa<FunctionOpInterface>(op)
                               ? llvm::lsp::SymbolKind::Function
                               : llvm::lsp::SymbolKind::Class,
                           lsp::Range(sourceMgr, def->scopeLoc),
                           lsp::Range(sourceMgr, def->loc));
      childSymbols = &symbols.back().children;

```
- **EN**: Implements logic around `findDocumentSymbols`, `getOpDef`, `dyn_cast`, `emplace_back`, and 3 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `findDocumentSymbols`、`getOpDef`、`dyn_cast`、`emplace_back` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 728-745
```cpp
    } else if (op->hasTrait<OpTrait::SymbolTable>()) {
      // Otherwise, if this is a symbol table push an anonymous document symbol.
      symbols.emplace_back("<" + op->getName().getStringRef() + ">",
                           llvm::lsp::SymbolKind::Namespace,
                           llvm::lsp::Range(sourceMgr, def->scopeLoc),
                           llvm::lsp::Range(sourceMgr, def->loc));
      childSymbols = &symbols.back().children;
    }
  }

  // Recurse into the regions of this operation.
  if (!op->getNumRegions())
    return;
  for (Region &region : op->getRegions())
    for (Operation &childOp : region.getOps())
      findDocumentSymbols(&childOp, *childSymbols);
}

```
- **EN**: Implements logic around `SymbolTable>`, `emplace_back`, `Range`, `back`, and 4 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `SymbolTable>`、`emplace_back`、`Range`、`back` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 746-769
```cpp
//===----------------------------------------------------------------------===//
// MLIRDocument: Code Completion
//===----------------------------------------------------------------------===//

namespace {
class LSPCodeCompleteContext : public AsmParserCodeCompleteContext {
public:
  LSPCodeCompleteContext(SMLoc completeLoc, lsp::CompletionList &completionList,
                         MLIRContext *ctx)
      : AsmParserCodeCompleteContext(completeLoc),
        completionList(completionList), ctx(ctx) {}

  /// Signal code completion for a dialect name, with an optional prefix.
  void completeDialectName(StringRef prefix) final {
    for (StringRef dialect : ctx->getAvailableDialects()) {
      llvm::lsp::CompletionItem item(prefix + dialect,
                                     llvm::lsp::CompletionItemKind::Module,
                                     /*sortText=*/"3");
      item.detail = "dialect";
      completionList.items.emplace_back(item);
    }
  }
  using AsmParserCodeCompleteContext::completeDialectName;

```
- **EN**: Introduces declarations for `LSPCodeCompleteContext`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `LSPCodeCompleteContext` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 770-788
```cpp
  /// Signal code completion for an operation name within the given dialect.
  void completeOperationName(StringRef dialectName) final {
    Dialect *dialect = ctx->getOrLoadDialect(dialectName);
    if (!dialect)
      return;

    for (const auto &op : ctx->getRegisteredOperations()) {
      if (&op.getDialect() != dialect)
        continue;

      llvm::lsp::CompletionItem item(
          op.getStringRef().drop_front(dialectName.size() + 1),
          llvm::lsp::CompletionItemKind::Field,
          /*sortText=*/"1");
      item.detail = "operation";
      completionList.items.emplace_back(item);
    }
  }

```
- **EN**: Implements logic around `completeOperationName`, `getOrLoadDialect`, `getRegisteredOperations`, `getDialect`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `completeOperationName`、`getOrLoadDialect`、`getRegisteredOperations`、`getDialect` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 789-808
```cpp
  /// Append the given SSA value as a code completion result for SSA value
  /// completions.
  void appendSSAValueCompletion(StringRef name, std::string typeData) final {
    // Check if we need to insert the `%` or not.
    bool stripPrefix = getCodeCompleteLoc().getPointer()[-1] == '%';

    llvm::lsp::CompletionItem item(name,
                                   llvm::lsp::CompletionItemKind::Variable);
    if (stripPrefix)
      item.insertText = name.drop_front(1).str();
    item.detail = std::move(typeData);
    completionList.items.emplace_back(item);
  }

  /// Append the given block as a code completion result for block name
  /// completions.
  void appendBlockCompletion(StringRef name) final {
    // Check if we need to insert the `^` or not.
    bool stripPrefix = getCodeCompleteLoc().getPointer()[-1] == '^';

```
- **EN**: Implements logic around `appendSSAValueCompletion`, `getCodeCompleteLoc`, `item`, `drop_front`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `appendSSAValueCompletion`、`getCodeCompleteLoc`、`item`、`drop_front` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 809-826
```cpp
    llvm::lsp::CompletionItem item(name, llvm::lsp::CompletionItemKind::Field);
    if (stripPrefix)
      item.insertText = name.drop_front(1).str();
    completionList.items.emplace_back(item);
  }

  /// Signal a completion for the given expected token.
  void completeExpectedTokens(ArrayRef<StringRef> tokens, bool optional) final {
    for (StringRef token : tokens) {
      llvm::lsp::CompletionItem item(token,
                                     llvm::lsp::CompletionItemKind::Keyword,
                                     /*sortText=*/"0");
      item.detail = optional ? "optional" : "";
      completionList.items.emplace_back(item);
    }
  }

  /// Signal a completion for an attribute.
```
- **EN**: Implements logic around `item`, `drop_front`, `emplace_back`, `completeExpectedTokens`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `item`、`drop_front`、`emplace_back`、`completeExpectedTokens` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 827-851
```cpp
  void completeAttribute(const llvm::StringMap<Attribute> &aliases) override {
    appendSimpleCompletions({"affine_set", "affine_map", "dense",
                             "dense_resource", "false", "loc", "sparse", "true",
                             "unit"},
                            llvm::lsp::CompletionItemKind::Field,
                            /*sortText=*/"1");

    completeDialectName("#");
    completeAliases(aliases, "#");
  }
  void completeDialectAttributeOrAlias(
      const llvm::StringMap<Attribute> &aliases) override {
    completeDialectName();
    completeAliases(aliases);
  }

  /// Signal a completion for a type.
  void completeType(const llvm::StringMap<Type> &aliases) override {
    // Handle the various builtin types.
    appendSimpleCompletions({"memref", "tensor", "complex", "tuple", "vector",
                             "bf16", "f16", "f32", "f64", "f80", "f128",
                             "index", "none"},
                            llvm::lsp::CompletionItemKind::Field,
                            /*sortText=*/"1");

```
- **EN**: Implements logic around `completeAttribute`, `appendSimpleCompletions`, `completeDialectName`, `completeAliases`, and 2 more symbols; this block manipulates core MLIR IR objects; bridges MLIR-produced code with runtime entry points.
- **CN**: 围绕 `completeAttribute`、`appendSimpleCompletions`、`completeDialectName`、`completeAliases` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并在 MLIR 生成代码与运行时入口点之间建立桥接。

### Lines 852-870
```cpp
    // Handle the builtin integer types.
    for (StringRef type : {"i", "si", "ui"}) {
      llvm::lsp::CompletionItem item(type + "<N>",
                                     llvm::lsp::CompletionItemKind::Field,
                                     /*sortText=*/"1");
      item.insertText = type.str();
      completionList.items.emplace_back(item);
    }

    // Insert completions for dialect types and aliases.
    completeDialectName("!");
    completeAliases(aliases, "!");
  }
  void
  completeDialectTypeOrAlias(const llvm::StringMap<Type> &aliases) override {
    completeDialectName();
    completeAliases(aliases);
  }

```
- **EN**: Implements logic around `item`, `str`, `emplace_back`, `completeDialectName`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `item`、`str`、`emplace_back`、`completeDialectName` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 871-891
```cpp
  /// Add completion results for the given set of aliases.
  template <typename T>
  void completeAliases(const llvm::StringMap<T> &aliases,
                       StringRef prefix = "") {
    for (const auto &alias : aliases) {
      llvm::lsp::CompletionItem item(prefix + alias.getKey(),
                                     llvm::lsp::CompletionItemKind::Field,
                                     /*sortText=*/"2");
      llvm::raw_string_ostream(item.detail) << "alias: " << alias.getValue();
      completionList.items.emplace_back(item);
    }
  }

  /// Add a set of simple completions that all have the same kind.
  void appendSimpleCompletions(ArrayRef<StringRef> completions,
                               llvm::lsp::CompletionItemKind kind,
                               StringRef sortText = "") {
    for (StringRef completion : completions)
      completionList.items.emplace_back(completion, kind, sortText);
  }

```
- **EN**: Implements logic around `completeAliases`, `item`, `raw_string_ostream`, `emplace_back`, and 1 more symbols.
- **CN**: 围绕 `completeAliases`、`item`、`raw_string_ostream`、`emplace_back` 等另外 1 个符号 实现具体逻辑。

### Lines 892-913
```cpp
private:
  lsp::CompletionList &completionList;
  MLIRContext *ctx;
};
} // namespace

lsp::CompletionList
MLIRDocument::getCodeCompletion(const lsp::URIForFile &uri,
                                const lsp::Position &completePos,
                                const DialectRegistry &registry) {
  SMLoc posLoc = completePos.getAsSMLoc(sourceMgr);
  if (!posLoc.isValid())
    return lsp::CompletionList();

  // To perform code completion, we run another parse of the module with the
  // code completion context provided.
  MLIRContext tmpContext(registry, MLIRContext::Threading::DISABLED);
  tmpContext.allowUnregisteredDialects();
  lsp::CompletionList completionList;
  LSPCodeCompleteContext lspCompleteContext(posLoc, completionList,
                                            &tmpContext);

```
- **EN**: Implements logic around `getCodeCompletion`, `getAsSMLoc`, `isValid`, `CompletionList`, and 3 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `getCodeCompletion`、`getAsSMLoc`、`isValid`、`CompletionList` 等另外 3 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 914-933
```cpp
  Block tmpIR;
  AsmParserState tmpState;
  (void)parseAsmSourceFile(sourceMgr, &tmpIR, &tmpContext, &tmpState,
                           &lspCompleteContext);
  return completionList;
}

//===----------------------------------------------------------------------===//
// MLIRDocument: Code Action
//===----------------------------------------------------------------------===//

void MLIRDocument::getCodeActionForDiagnostic(
    const lsp::URIForFile &uri, lsp::Position &pos, StringRef severity,
    StringRef message, std::vector<llvm::lsp::TextEdit> &edits) {
  // Ignore diagnostics that print the current operation. These are always
  // enabled for the language server, but not generally during normal
  // parsing/verification.
  if (message.starts_with("see current operation: "))
    return;

```
- **EN**: Implements logic around `parseAsmSourceFile`, `getCodeActionForDiagnostic`, `starts_with`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `parseAsmSourceFile`、`getCodeActionForDiagnostic`、`starts_with` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 934-956
```cpp
  // Get the start of the line containing the diagnostic.
  const auto &buffer = sourceMgr.getBufferInfo(sourceMgr.getMainFileID());
  const char *lineStart = buffer.getPointerForLineNumber(pos.line + 1);
  if (!lineStart)
    return;
  StringRef line(lineStart, pos.character);

  // Add a text edit for adding an expected-* diagnostic check for this
  // diagnostic.
  llvm::lsp::TextEdit edit;
  edit.range = lsp::Range(lsp::Position(pos.line, 0));

  // Use the indent of the current line for the expected-* diagnostic.
  size_t indent = line.find_first_not_of(' ');
  if (indent == StringRef::npos)
    indent = line.size();

  edit.newText.append(indent, ' ');
  llvm::raw_string_ostream(edit.newText)
      << "// expected-" << severity << " @below {{" << message << "}}\n";
  edits.emplace_back(std::move(edit));
}

```
- **EN**: Implements logic around `getBufferInfo`, `getPointerForLineNumber`, `line`, `Range`, and 5 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `getBufferInfo`、`getPointerForLineNumber`、`line`、`Range` 等另外 5 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 957-976
```cpp
//===----------------------------------------------------------------------===//
// MLIRDocument: Bytecode
//===----------------------------------------------------------------------===//

llvm::Expected<lsp::MLIRConvertBytecodeResult>
MLIRDocument::convertToBytecode() {
  // TODO: We currently require a single top-level operation, but this could
  // conceptually be relaxed.
  if (!llvm::hasSingleElement(parsedIR)) {
    if (parsedIR.empty()) {
      return llvm::make_error<llvm::lsp::LSPError>(
          "expected a single and valid top-level operation, please ensure "
          "there are no errors",
          llvm::lsp::ErrorCode::RequestFailed);
    }
    return llvm::make_error<llvm::lsp::LSPError>(
        "expected a single top-level operation",
        llvm::lsp::ErrorCode::RequestFailed);
  }

```
- **EN**: Implements logic around `convertToBytecode`, `hasSingleElement`, `empty`, `LSPError>`; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `convertToBytecode`、`hasSingleElement`、`empty`、`LSPError>` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 977-994
```cpp
  lsp::MLIRConvertBytecodeResult result;
  {
    BytecodeWriterConfig writerConfig(fallbackResourceMap);

    std::string rawBytecodeBuffer;
    llvm::raw_string_ostream os(rawBytecodeBuffer);
    // No desired bytecode version set, so no need to check for error.
    (void)writeBytecodeToFile(&parsedIR.front(), os, writerConfig);
    result.output = llvm::encodeBase64(rawBytecodeBuffer);
  }
  return result;
}

//===----------------------------------------------------------------------===//
// MLIRTextFileChunk
//===----------------------------------------------------------------------===//

namespace {
```
- **EN**: Implements logic around `writerConfig`, `os`, `writeBytecodeToFile`, `encodeBase64`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `writerConfig`、`os`、`writeBytecodeToFile`、`encodeBase64` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 995-1013
```cpp
/// This class represents a single chunk of an MLIR text file.
struct MLIRTextFileChunk {
  MLIRTextFileChunk(MLIRContext &context, uint64_t lineOffset,
                    const lsp::URIForFile &uri, StringRef contents,
                    StringRef workspaceRoot,
                    std::vector<lsp::Diagnostic> &diagnostics)
      : lineOffset(lineOffset),
        document(context, uri, contents, workspaceRoot, diagnostics) {}

  /// Adjust the line number of the given range to anchor at the beginning of
  /// the file, instead of the beginning of this chunk.
  void adjustLocForChunkOffset(lsp::Range &range) {
    adjustLocForChunkOffset(range.start);
    adjustLocForChunkOffset(range.end);
  }
  /// Adjust the line number of the given position to anchor at the beginning of
  /// the file, instead of the beginning of this chunk.
  void adjustLocForChunkOffset(lsp::Position &pos) { pos.line += lineOffset; }

```
- **EN**: Introduces declarations for `MLIRTextFileChunk`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `MLIRTextFileChunk` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 1014-1033
```cpp
  /// The line offset of this chunk from the beginning of the file.
  uint64_t lineOffset;
  /// The document referred to by this chunk.
  MLIRDocument document;
};
} // namespace

//===----------------------------------------------------------------------===//
// MLIRTextFile
//===----------------------------------------------------------------------===//

namespace {
/// This class represents a text file containing one or more MLIR documents.
class MLIRTextFile {
public:
  MLIRTextFile(const lsp::URIForFile &uri, StringRef fileContents,
               int64_t version, lsp::DialectRegistryFn registryFn,
               StringRef workspaceRoot,
               std::vector<lsp::Diagnostic> &diagnostics);

```
- **EN**: Introduces declarations for `MLIRTextFile`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `MLIRTextFile` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 1034-1054
```cpp
  /// Return the current version of this text file.
  int64_t getVersion() const { return version; }

  //===--------------------------------------------------------------------===//
  // LSP Queries
  //===--------------------------------------------------------------------===//

  void getLocationsOf(const lsp::URIForFile &uri, lsp::Position defPos,
                      std::vector<lsp::Location> &locations);
  void findReferencesOf(const lsp::URIForFile &uri, lsp::Position pos,
                        std::vector<lsp::Location> &references);
  std::optional<lsp::Hover> findHover(const lsp::URIForFile &uri,
                                      lsp::Position hoverPos);
  void findDocumentSymbols(std::vector<lsp::DocumentSymbol> &symbols);
  lsp::CompletionList getCodeCompletion(const lsp::URIForFile &uri,
                                        lsp::Position completePos);
  void getCodeActions(const lsp::URIForFile &uri, const lsp::Range &pos,
                      const lsp::CodeActionContext &context,
                      std::vector<lsp::CodeAction> &actions);
  llvm::Expected<lsp::MLIRConvertBytecodeResult> convertToBytecode();

```
- **EN**: Implements logic around `getVersion`, `getLocationsOf`, `findReferencesOf`, `findHover`, and 4 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getVersion`、`getLocationsOf`、`findReferencesOf`、`findHover` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 1055-1072
```cpp
private:
  /// Find the MLIR document that contains the given position, and update the
  /// position to be anchored at the start of the found chunk instead of the
  /// beginning of the file.
  MLIRTextFileChunk &getChunkFor(lsp::Position &pos);

  /// The context used to hold the state contained by the parsed document.
  MLIRContext context;

  /// The full string contents of the file.
  std::string contents;

  /// The version of this file.
  int64_t version;

  /// The number of lines in the file.
  int64_t totalNumLines = 0;

```
- **EN**: Implements logic around `getChunkFor`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `getChunkFor` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 1073-1093
```cpp
  /// The chunks of this file. The order of these chunks is the order in which
  /// they appear in the text file.
  std::vector<std::unique_ptr<MLIRTextFileChunk>> chunks;
};
} // namespace

MLIRTextFile::MLIRTextFile(const lsp::URIForFile &uri, StringRef fileContents,
                           int64_t version, lsp::DialectRegistryFn registryFn,
                           StringRef workspaceRoot,
                           std::vector<lsp::Diagnostic> &diagnostics)
    : context(registryFn(uri), MLIRContext::Threading::DISABLED),
      contents(fileContents.str()), version(version) {
  context.allowUnregisteredDialects();

  // Split the file into separate MLIR documents.
  SmallVector<StringRef, 8> subContents;
  StringRef(contents).split(subContents, kDefaultSplitMarker);
  chunks.emplace_back(std::make_unique<MLIRTextFileChunk>(
      context, /*lineOffset=*/0, uri, subContents.front(), workspaceRoot,
      diagnostics));

```
- **EN**: Implements logic around `context`, `contents`, `allowUnregisteredDialects`, `StringRef`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `context`、`contents`、`allowUnregisteredDialects`、`StringRef` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 1094-1117
```cpp
  uint64_t lineOffset = subContents.front().count('\n');
  for (StringRef docContents : llvm::drop_begin(subContents)) {
    unsigned currentNumDiags = diagnostics.size();
    auto chunk = std::make_unique<MLIRTextFileChunk>(
        context, lineOffset, uri, docContents, workspaceRoot, diagnostics);
    lineOffset += docContents.count('\n');

    // Adjust locations used in diagnostics to account for the offset from the
    // beginning of the file.
    for (lsp::Diagnostic &diag :
         llvm::drop_begin(diagnostics, currentNumDiags)) {
      chunk->adjustLocForChunkOffset(diag.range);

      if (!diag.relatedInformation)
        continue;
      for (auto &it : *diag.relatedInformation)
        if (it.location.uri == uri)
          chunk->adjustLocForChunkOffset(it.location.range);
    }
    chunks.emplace_back(std::move(chunk));
  }
  totalNumLines = lineOffset;
}

```
- **EN**: Implements logic around `front`, `drop_begin`, `size`, `make_unique`, and 3 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects.
- **CN**: 围绕 `front`、`drop_begin`、`size`、`make_unique` 等另外 3 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象。

### Lines 1118-1137
```cpp
void MLIRTextFile::getLocationsOf(const lsp::URIForFile &uri,
                                  lsp::Position defPos,
                                  std::vector<lsp::Location> &locations) {
  MLIRTextFileChunk &chunk = getChunkFor(defPos);
  chunk.document.getLocationsOf(uri, defPos, locations);

  // Adjust any locations within this file for the offset of this chunk.
  if (chunk.lineOffset == 0)
    return;
  for (lsp::Location &loc : locations)
    if (loc.uri == uri)
      chunk.adjustLocForChunkOffset(loc.range);
}

void MLIRTextFile::findReferencesOf(const lsp::URIForFile &uri,
                                    lsp::Position pos,
                                    std::vector<lsp::Location> &references) {
  MLIRTextFileChunk &chunk = getChunkFor(pos);
  chunk.document.findReferencesOf(uri, pos, references);

```
- **EN**: Implements logic around `getLocationsOf`, `getChunkFor`, `adjustLocForChunkOffset`, `findReferencesOf`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getLocationsOf`、`getChunkFor`、`adjustLocForChunkOffset`、`findReferencesOf` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 1138-1156
```cpp
  // Adjust any locations within this file for the offset of this chunk.
  if (chunk.lineOffset == 0)
    return;
  for (lsp::Location &loc : references)
    if (loc.uri == uri)
      chunk.adjustLocForChunkOffset(loc.range);
}

std::optional<lsp::Hover> MLIRTextFile::findHover(const lsp::URIForFile &uri,
                                                  lsp::Position hoverPos) {
  MLIRTextFileChunk &chunk = getChunkFor(hoverPos);
  std::optional<lsp::Hover> hoverInfo = chunk.document.findHover(uri, hoverPos);

  // Adjust any locations within this file for the offset of this chunk.
  if (chunk.lineOffset != 0 && hoverInfo && hoverInfo->range)
    chunk.adjustLocForChunkOffset(*hoverInfo->range);
  return hoverInfo;
}

```
- **EN**: Implements logic around `adjustLocForChunkOffset`, `findHover`, `getChunkFor`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `adjustLocForChunkOffset`、`findHover`、`getChunkFor` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 1157-1174
```cpp
void MLIRTextFile::findDocumentSymbols(
    std::vector<lsp::DocumentSymbol> &symbols) {
  if (chunks.size() == 1)
    return chunks.front()->document.findDocumentSymbols(symbols);

  // If there are multiple chunks in this file, we create top-level symbols for
  // each chunk.
  for (unsigned i = 0, e = chunks.size(); i < e; ++i) {
    MLIRTextFileChunk &chunk = *chunks[i];
    lsp::Position startPos(chunk.lineOffset);
    lsp::Position endPos((i == e - 1) ? totalNumLines - 1
                                      : chunks[i + 1]->lineOffset);
    lsp::DocumentSymbol symbol("<file-split-" + Twine(i) + ">",
                               llvm::lsp::SymbolKind::Namespace,
                               /*range=*/lsp::Range(startPos, endPos),
                               /*selectionRange=*/lsp::Range(startPos));
    chunk.document.findDocumentSymbols(symbol.children);

```
- **EN**: Implements logic around `findDocumentSymbols`, `size`, `front`, `startPos`, and 3 more symbols.
- **CN**: 围绕 `findDocumentSymbols`、`size`、`front`、`startPos` 等另外 3 个符号 实现具体逻辑。

### Lines 1175-1195
```cpp
    // Fixup the locations of document symbols within this chunk.
    if (i != 0) {
      SmallVector<lsp::DocumentSymbol *> symbolsToFix;
      for (lsp::DocumentSymbol &childSymbol : symbol.children)
        symbolsToFix.push_back(&childSymbol);

      while (!symbolsToFix.empty()) {
        lsp::DocumentSymbol *symbol = symbolsToFix.pop_back_val();
        chunk.adjustLocForChunkOffset(symbol->range);
        chunk.adjustLocForChunkOffset(symbol->selectionRange);

        for (lsp::DocumentSymbol &childSymbol : symbol->children)
          symbolsToFix.push_back(&childSymbol);
      }
    }

    // Push the symbol for this chunk.
    symbols.emplace_back(std::move(symbol));
  }
}

```
- **EN**: Implements logic around `push_back`, `empty`, `pop_back_val`, `adjustLocForChunkOffset`, and 1 more symbols.
- **CN**: 围绕 `push_back`、`empty`、`pop_back_val`、`adjustLocForChunkOffset` 等另外 1 个符号 实现具体逻辑。

### Lines 1196-1222
```cpp
lsp::CompletionList MLIRTextFile::getCodeCompletion(const lsp::URIForFile &uri,
                                                    lsp::Position completePos) {
  MLIRTextFileChunk &chunk = getChunkFor(completePos);
  lsp::CompletionList completionList = chunk.document.getCodeCompletion(
      uri, completePos, context.getDialectRegistry());

  // Adjust any completion locations.
  for (llvm::lsp::CompletionItem &item : completionList.items) {
    if (item.textEdit)
      chunk.adjustLocForChunkOffset(item.textEdit->range);
    for (llvm::lsp::TextEdit &edit : item.additionalTextEdits)
      chunk.adjustLocForChunkOffset(edit.range);
  }
  return completionList;
}

void MLIRTextFile::getCodeActions(const lsp::URIForFile &uri,
                                  const lsp::Range &pos,
                                  const lsp::CodeActionContext &context,
                                  std::vector<lsp::CodeAction> &actions) {
  // Create actions for any diagnostics in this file.
  for (auto &diag : context.diagnostics) {
    if (diag.source != "mlir")
      continue;
    lsp::Position diagPos = diag.range.start;
    MLIRTextFileChunk &chunk = getChunkFor(diagPos);

```
- **EN**: Implements logic around `getCodeCompletion`, `getChunkFor`, `getDialectRegistry`, `adjustLocForChunkOffset`, and 1 more symbols.
- **CN**: 围绕 `getCodeCompletion`、`getChunkFor`、`getDialectRegistry`、`adjustLocForChunkOffset` 等另外 1 个符号 实现具体逻辑。

### Lines 1223-1244
```cpp
    // Add a new code action that inserts a "expected" diagnostic check.
    lsp::CodeAction action;
    action.title = "Add expected-* diagnostic checks";
    action.kind = lsp::CodeAction::kQuickFix.str();

    StringRef severity;
    switch (diag.severity) {
    case llvm::lsp::DiagnosticSeverity::Error:
      severity = "error";
      break;
    case llvm::lsp::DiagnosticSeverity::Warning:
      severity = "warning";
      break;
    default:
      continue;
    }

    // Get edits for the diagnostic.
    std::vector<llvm::lsp::TextEdit> edits;
    chunk.document.getCodeActionForDiagnostic(uri, diagPos, severity,
                                              diag.message, edits);

```
- **EN**: Implements logic around `str`, `getCodeActionForDiagnostic`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `str`、`getCodeActionForDiagnostic` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 1245-1263
```cpp
    // Walk the related diagnostics, this is how we encode notes.
    if (diag.relatedInformation) {
      for (auto &noteDiag : *diag.relatedInformation) {
        if (noteDiag.location.uri != uri)
          continue;
        diagPos = noteDiag.location.range.start;
        diagPos.line -= chunk.lineOffset;
        chunk.document.getCodeActionForDiagnostic(uri, diagPos, "note",
                                                  noteDiag.message, edits);
      }
    }
    // Fixup the locations for any edits.
    for (llvm::lsp::TextEdit &edit : edits)
      chunk.adjustLocForChunkOffset(edit.range);

    action.edit.emplace();
    action.edit->changes[uri.uri().str()] = std::move(edits);
    action.diagnostics = {diag};

```
- **EN**: Implements logic around `getCodeActionForDiagnostic`, `adjustLocForChunkOffset`, `emplace`, `uri`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getCodeActionForDiagnostic`、`adjustLocForChunkOffset`、`emplace`、`uri` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 1264-1282
```cpp
    actions.emplace_back(std::move(action));
  }
}

llvm::Expected<lsp::MLIRConvertBytecodeResult>
MLIRTextFile::convertToBytecode() {
  // Bail out if there is more than one chunk, bytecode wants a single module.
  if (chunks.size() != 1) {
    return llvm::make_error<llvm::lsp::LSPError>(
        "unexpected split file, please remove all `// -----`",
        llvm::lsp::ErrorCode::RequestFailed);
  }
  return chunks.front()->document.convertToBytecode();
}

MLIRTextFileChunk &MLIRTextFile::getChunkFor(lsp::Position &pos) {
  if (chunks.size() == 1)
    return *chunks.front();

```
- **EN**: Implements logic around `emplace_back`, `convertToBytecode`, `size`, `LSPError>`, and 2 more symbols.
- **CN**: 围绕 `emplace_back`、`convertToBytecode`、`size`、`LSPError>` 等另外 2 个符号 实现具体逻辑。

### Lines 1283-1300
```cpp
  // Search for the first chunk with a greater line offset, the previous chunk
  // is the one that contains `pos`.
  auto it = llvm::upper_bound(
      chunks, pos, [](const lsp::Position &pos, const auto &chunk) {
        return static_cast<uint64_t>(pos.line) < chunk->lineOffset;
      });
  MLIRTextFileChunk &chunk = it == chunks.end() ? *chunks.back() : **(--it);
  pos.line -= chunk.lineOffset;
  return chunk;
}

//===----------------------------------------------------------------------===//
// MLIRServer::Impl
//===----------------------------------------------------------------------===//

struct lsp::MLIRServer::Impl {
  Impl(lsp::DialectRegistryFn registryFn) : registryFn(registryFn) {}

```
- **EN**: Introduces declarations for `lsp::MLIRServer::Impl`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `lsp::MLIRServer::Impl` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 1301-1319
```cpp
  /// The registry factory for containing dialects that can be recognized in
  /// parsed .mlir files.
  lsp::DialectRegistryFn registryFn;

  /// The files held by the server, mapped by their URI file name.
  llvm::StringMap<std::unique_ptr<MLIRTextFile>> files;

  /// The workspace root of the server.
  std::string workspaceRoot;
};

//===----------------------------------------------------------------------===//
// MLIRServer
//===----------------------------------------------------------------------===//

lsp::MLIRServer::MLIRServer(lsp::DialectRegistryFn registryFn)
    : impl(std::make_unique<Impl>(registryFn)) {}
lsp::MLIRServer::~MLIRServer() = default;

```
- **EN**: Implements logic around `impl`, `~MLIRServer`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `impl`、`~MLIRServer` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 1320-1337
```cpp
void lsp::MLIRServer::addOrUpdateDocument(
    const URIForFile &uri, StringRef contents, int64_t version,
    std::vector<llvm::lsp::Diagnostic> &diagnostics) {
  impl->files[uri.file()] =
      std::make_unique<MLIRTextFile>(uri, contents, version, impl->registryFn,
                                     impl->workspaceRoot, diagnostics);
}

std::optional<int64_t> lsp::MLIRServer::removeDocument(const URIForFile &uri) {
  auto it = impl->files.find(uri.file());
  if (it == impl->files.end())
    return std::nullopt;

  int64_t version = it->second->getVersion();
  impl->files.erase(it);
  return version;
}

```
- **EN**: Implements logic around `addOrUpdateDocument`, `file`, `make_unique`, `removeDocument`, and 4 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `addOrUpdateDocument`、`file`、`make_unique`、`removeDocument` 等另外 4 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 1338-1361
```cpp
void lsp::MLIRServer::getLocationsOf(
    const URIForFile &uri, const Position &defPos,
    std::vector<llvm::lsp::Location> &locations) {
  auto fileIt = impl->files.find(uri.file());
  if (fileIt != impl->files.end())
    fileIt->second->getLocationsOf(uri, defPos, locations);
}

void lsp::MLIRServer::findReferencesOf(
    const URIForFile &uri, const Position &pos,
    std::vector<llvm::lsp::Location> &references) {
  auto fileIt = impl->files.find(uri.file());
  if (fileIt != impl->files.end())
    fileIt->second->findReferencesOf(uri, pos, references);
}

std::optional<lsp::Hover> lsp::MLIRServer::findHover(const URIForFile &uri,
                                                     const Position &hoverPos) {
  auto fileIt = impl->files.find(uri.file());
  if (fileIt != impl->files.end())
    return fileIt->second->findHover(uri, hoverPos);
  return std::nullopt;
}

```
- **EN**: Implements logic around `getLocationsOf`, `find`, `end`, `findReferencesOf`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getLocationsOf`、`find`、`end`、`findReferencesOf` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 1362-1385
```cpp
void lsp::MLIRServer::findDocumentSymbols(
    const URIForFile &uri, std::vector<DocumentSymbol> &symbols) {
  auto fileIt = impl->files.find(uri.file());
  if (fileIt != impl->files.end())
    fileIt->second->findDocumentSymbols(symbols);
}

lsp::CompletionList
lsp::MLIRServer::getCodeCompletion(const URIForFile &uri,
                                   const Position &completePos) {
  auto fileIt = impl->files.find(uri.file());
  if (fileIt != impl->files.end())
    return fileIt->second->getCodeCompletion(uri, completePos);
  return CompletionList();
}

void lsp::MLIRServer::getCodeActions(const URIForFile &uri, const Range &pos,
                                     const CodeActionContext &context,
                                     std::vector<CodeAction> &actions) {
  auto fileIt = impl->files.find(uri.file());
  if (fileIt != impl->files.end())
    fileIt->second->getCodeActions(uri, pos, context, actions);
}

```
- **EN**: Implements logic around `findDocumentSymbols`, `find`, `end`, `getCodeCompletion`, and 2 more symbols.
- **CN**: 围绕 `findDocumentSymbols`、`find`、`end`、`getCodeCompletion` 等另外 2 个符号 实现具体逻辑。

### Lines 1386-1403
```cpp
llvm::Expected<lsp::MLIRConvertBytecodeResult>
lsp::MLIRServer::convertFromBytecode(const URIForFile &uri) {
  MLIRContext tempContext(impl->registryFn(uri));
  tempContext.allowUnregisteredDialects();

  // Collect any errors during parsing.
  std::string errorMsg;
  ScopedDiagnosticHandler diagHandler(
      &tempContext,
      [&](mlir::Diagnostic &diag) { errorMsg += diag.str() + "\n"; });

  // Handling for external resources, which we want to propagate up to the user.
  FallbackAsmResourceMap fallbackResourceMap;

  // Setup the parser config.
  ParserConfig parserConfig(&tempContext, /*verifyAfterParse=*/true,
                            &fallbackResourceMap);

```
- **EN**: Implements logic around `convertFromBytecode`, `tempContext`, `allowUnregisteredDialects`, `diagHandler`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; parses or prints textual MLIR representations.
- **CN**: 围绕 `convertFromBytecode`、`tempContext`、`allowUnregisteredDialects`、`diagHandler` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并解析或打印文本形式的 MLIR 表示。

### Lines 1404-1427
```cpp
  // Try to parse the given source file.
  Block parsedBlock;
  if (failed(parseSourceFile(uri.file(), &parsedBlock, parserConfig))) {
    return llvm::make_error<llvm::lsp::LSPError>(
        "failed to parse bytecode source file: " + errorMsg,
        llvm::lsp::ErrorCode::RequestFailed);
  }

  // TODO: We currently expect a single top-level operation, but this could
  // conceptually be relaxed.
  if (!llvm::hasSingleElement(parsedBlock)) {
    return llvm::make_error<llvm::lsp::LSPError>(
        "expected bytecode to contain a single top-level operation",
        llvm::lsp::ErrorCode::RequestFailed);
  }

  // Print the module to a buffer.
  lsp::MLIRConvertBytecodeResult result;
  {
    // Extract the top-level op so that aliases get printed.
    // FIXME: We should be able to enable aliases without having to do this!
    OwningOpRef<Operation *> topOp = &parsedBlock.front();
    topOp->remove();

```
- **EN**: Implements logic around `failed`, `LSPError>`, `hasSingleElement`, `front`, and 1 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `failed`、`LSPError>`、`hasSingleElement`、`front` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 1428-1447
```cpp
    AsmState state(*topOp, OpPrintingFlags().enableDebugInfo().assumeVerified(),
                   /*locationMap=*/nullptr, &fallbackResourceMap);

    llvm::raw_string_ostream os(result.output);
    topOp->print(os, state);
  }
  return std::move(result);
}

llvm::Expected<lsp::MLIRConvertBytecodeResult>
lsp::MLIRServer::convertToBytecode(const URIForFile &uri) {
  auto fileIt = impl->files.find(uri.file());
  if (fileIt == impl->files.end()) {
    return llvm::make_error<llvm::lsp::LSPError>(
        "language server does not contain an entry for this source file",
        llvm::lsp::ErrorCode::RequestFailed);
  }
  return fileIt->second->convertToBytecode();
}

```
- **EN**: Implements logic around `state`, `os`, `print`, `move`, and 4 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `state`、`os`、`print`、`move` 等另外 4 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 1448-1450
```cpp
void lsp::MLIRServer::setWorkspaceRoot(StringRef root) {
  impl->workspaceRoot = root.str();
}
```
- **EN**: Implements logic around `setWorkspaceRoot`, `str`.
- **CN**: 围绕 `setWorkspaceRoot`、`str` 实现具体逻辑。

## Key Concepts / 关键概念

- **Tooling support / 工具链支持**:
  - **EN**: Builds reusable infrastructure for opt-like tools, translation drivers, and CLIs.
  - **CN**: 为类似 opt 的工具、翻译驱动和命令行程序构建可复用基础设施。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **Structured diagnostics / 结构化诊断**:
  - **EN**: Failures are surfaced through `LogicalResult`, diagnostics, or related reporting helpers.
  - **CN**: 通过 `LogicalResult`、诊断对象或相关报告工具显式传播失败。
- **Bytecode support / Bytecode 支持**:
  - **EN**: The file reads, writes, or describes compact MLIR bytecode forms.
  - **CN**: 该文件读取、写出或描述紧凑的 MLIR bytecode 形式。
- **Runtime interop / 运行时互操作**:
  - **EN**: The code exposes or consumes ABI-level hooks used by generated code at runtime.
  - **CN**: 代码暴露或消费运行时由生成代码使用的 ABI 级钩子。
- **Streaming output / 流式输出**:
  - **EN**: Text, diagnostics, or generated content are emitted through LLVM stream APIs.
  - **CN**: 通过 LLVM 流 API 输出文本、诊断或生成内容。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `MLIRServer.h`, `Protocol.h`, `mlir/AsmParser/AsmParser.h`, `mlir/AsmParser/AsmParserState.h`, `mlir/AsmParser/CodeComplete.h`, `mlir/Bytecode/BytecodeWriter.h`, `mlir/IR/Operation.h`, `mlir/Interfaces/FunctionInterfaces.h`, `mlir/Parser/Parser.h`, `mlir/Support/ToolUtilities.h` ... (+7 more)
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: LLVM support-library helpers / LLVM Support 库辅助工具 (5), MLIR assembly parsing interfaces / MLIR 汇编解析接口 (3), MLIR bytecode serialization APIs / MLIR bytecode 序列化 API (1), core MLIR IR abstractions / 核心 MLIR IR 抽象 (1), MLIR interface declarations / MLIR 接口声明 (1), MLIR parser declarations / MLIR 解析器声明 (1), shared MLIR support helpers / 共享的 MLIR 支持工具 (1), tooling support declarations / 工具支持声明 (1)
