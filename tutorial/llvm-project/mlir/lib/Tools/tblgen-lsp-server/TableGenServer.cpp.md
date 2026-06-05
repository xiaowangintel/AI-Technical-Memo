# TableGenServer.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Tools/tblgen-lsp-server/TableGenServer.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements reusable command-line tool support built on top of MLIR libraries.
  - **CN**: 实现构建于 MLIR 库之上的可复用命令行工具支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24
```cpp
//===- TableGenServer.cpp - TableGen Language Server ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "TableGenServer.h"

#include "mlir/Support/IndentedOstream.h"
#include "mlir/Tools/lsp-server-support/CompilationDatabase.h"
#include "mlir/Tools/lsp-server-support/SourceMgrUtils.h"
#include "llvm/ADT/IntervalMap.h"
#include "llvm/ADT/PointerUnion.h"
#include "llvm/ADT/StringMap.h"
#include "llvm/Support/LSP/Logging.h"
#include "llvm/Support/LSP/Protocol.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/VirtualFileSystem.h"
#include "llvm/TableGen/Parser.h"
#include "llvm/TableGen/Record.h"
#include <optional>

```
- **EN**: Pulls in the declarations needed by this translation unit, including `TableGenServer.h`, `mlir/Support/IndentedOstream.h`, `mlir/Tools/lsp-server-support/CompilationDatabase.h`, `mlir/Tools/lsp-server-support/SourceMgrUtils.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `TableGenServer.h`, `mlir/Support/IndentedOstream.h`, `mlir/Tools/lsp-server-support/CompilationDatabase.h`, `mlir/Tools/lsp-server-support/SourceMgrUtils.h`。

### Lines 25-38
```cpp
using namespace mlir;
using llvm::Record;
using llvm::RecordKeeper;
using llvm::RecordVal;
using llvm::SourceMgr;

/// Returns the range of a lexical token given a SMLoc corresponding to the
/// start of an token location. The range is computed heuristically, and
/// supports identifier-like tokens, strings, etc.
static SMRange convertTokenLocToRange(SMLoc loc) {
  return lsp::convertTokenLocToRange(loc, "$");
}

/// Returns a language server uri for the given source location. `mainFileURI`
```
- **EN**: Implements logic around `convertTokenLocToRange`; this block manipulates core MLIR IR objects; processes TableGen records or generates derived code.
- **CN**: 围绕 `convertTokenLocToRange` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并处理 TableGen 记录或生成派生代码。

### Lines 39-55
```cpp
/// corresponds to the uri for the main file of the source manager.
static llvm::lsp::URIForFile
getURIFromLoc(const SourceMgr &mgr, SMLoc loc,
              const llvm::lsp::URIForFile &mainFileURI) {
  int bufferId = mgr.FindBufferContainingLoc(loc);
  if (bufferId == 0 || bufferId == static_cast<int>(mgr.getMainFileID()))
    return mainFileURI;
  llvm::Expected<llvm::lsp::URIForFile> fileForLoc =
      llvm::lsp::URIForFile::fromFile(
          mgr.getBufferInfo(bufferId).Buffer->getBufferIdentifier());
  if (fileForLoc)
    return *fileForLoc;
  llvm::lsp::Logger::error("Failed to create URI for include file: {0}",
                           llvm::toString(fileForLoc.takeError()));
  return mainFileURI;
}

```
- **EN**: Implements logic around `getURIFromLoc`, `FindBufferContainingLoc`, `static_cast`, `fromFile`, and 3 more symbols.
- **CN**: 围绕 `getURIFromLoc`、`FindBufferContainingLoc`、`static_cast`、`fromFile` 等另外 3 个符号 实现具体逻辑。

### Lines 56-69
```cpp
/// Returns a language server location from the given source range.
static llvm::lsp::Location
getLocationFromLoc(SourceMgr &mgr, SMRange loc,
                   const llvm::lsp::URIForFile &uri) {
  return llvm::lsp::Location(getURIFromLoc(mgr, loc.Start, uri),
                             llvm::lsp::Range(mgr, loc));
}
static llvm::lsp::Location
getLocationFromLoc(SourceMgr &mgr, SMLoc loc,
                   const llvm::lsp::URIForFile &uri) {
  return getLocationFromLoc(mgr, convertTokenLocToRange(loc), uri);
}

/// Convert the given TableGen diagnostic to the LSP form.
```
- **EN**: Implements logic around `getLocationFromLoc`, `Location`, `Range`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; processes TableGen records or generates derived code.
- **CN**: 围绕 `getLocationFromLoc`、`Location`、`Range` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并处理 TableGen 记录或生成派生代码。

### Lines 70-84
```cpp
static std::optional<llvm::lsp::Diagnostic>
getLspDiagnoticFromDiag(const llvm::SMDiagnostic &diag,
                        const llvm::lsp::URIForFile &uri) {
  auto *sourceMgr = const_cast<SourceMgr *>(diag.getSourceMgr());
  if (!sourceMgr || !diag.getLoc().isValid())
    return std::nullopt;

  llvm::lsp::Diagnostic lspDiag;
  lspDiag.source = "tablegen";
  lspDiag.category = "Parse Error";

  // Try to grab a file location for this diagnostic.
  llvm::lsp::Location loc = getLocationFromLoc(*sourceMgr, diag.getLoc(), uri);
  lspDiag.range = loc.range;

```
- **EN**: Implements logic around `getLspDiagnoticFromDiag`, `getSourceMgr`, `getLoc`, `getLocationFromLoc`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; parses or prints textual MLIR representations; processes TableGen records or generates derived code.
- **CN**: 围绕 `getLspDiagnoticFromDiag`、`getSourceMgr`、`getLoc`、`getLocationFromLoc` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示，并处理 TableGen 记录或生成派生代码。

### Lines 85-106
```cpp
  // Skip diagnostics that weren't emitted within the main file.
  if (loc.uri != uri)
    return std::nullopt;

  // Convert the severity for the diagnostic.
  switch (diag.getKind()) {
  case SourceMgr::DK_Warning:
    lspDiag.severity = llvm::lsp::DiagnosticSeverity::Warning;
    break;
  case SourceMgr::DK_Error:
    lspDiag.severity = llvm::lsp::DiagnosticSeverity::Error;
    break;
  case SourceMgr::DK_Note:
    // Notes are emitted separately from the main diagnostic, so we just treat
    // them as remarks given that we can't determine the diagnostic to relate
    // them to.
  case SourceMgr::DK_Remark:
    lspDiag.severity = llvm::lsp::DiagnosticSeverity::Information;
    break;
  }
  lspDiag.message = diag.getMessage().str();

```
- **EN**: Implements logic around `getKind`, `getMessage`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `getKind`、`getMessage` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 107-129
```cpp
  return lspDiag;
}

/// Get the base definition of the given record value, or nullptr if one
/// couldn't be found.
static std::pair<const Record *, const RecordVal *>
getBaseValue(const Record *record, const RecordVal *value) {
  if (value->isTemplateArg())
    return {nullptr, nullptr};

  // Find a base value for the field in the super classes of the given record.
  // On success, `record` is updated to the new parent record.
  StringRef valueName = value->getName();
  auto findValueInSupers = [&](const Record *&record) -> const RecordVal * {
    for (const Record *parentRecord : record->getSuperClasses()) {
      if (auto *newBase = parentRecord->getValue(valueName)) {
        record = parentRecord;
        return newBase;
      }
    }
    return nullptr;
  };

```
- **EN**: Implements logic around `getBaseValue`, `isTemplateArg`, `getName`, `getSuperClasses`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getBaseValue`、`isTemplateArg`、`getName`、`getSuperClasses` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 130-144
```cpp
  // Try to find the lowest definition of the record value.
  std::pair<const Record *, const RecordVal *> baseValue = {};
  while (const RecordVal *newBase = findValueInSupers(record))
    baseValue = {record, newBase};

  // Check that the base isn't the same as the current value (e.g. if the value
  // wasn't overridden).
  if (!baseValue.second || baseValue.second->getLoc() == value->getLoc())
    return {nullptr, nullptr};
  return baseValue;
}

//===----------------------------------------------------------------------===//
// TableGenIndex
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `findValueInSupers`, `getLoc`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `findValueInSupers`、`getLoc` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 145-160
```cpp

namespace {
/// This class represents a single symbol definition within a TableGen index. It
/// contains the definition of the symbol, the location of the symbol, and any
/// recorded references.
struct TableGenIndexSymbol {
  TableGenIndexSymbol(const Record *record)
      : definition(record),
        defLoc(convertTokenLocToRange(record->getLoc().front())) {}
  TableGenIndexSymbol(const RecordVal *value)
      : definition(value), defLoc(convertTokenLocToRange(value->getLoc())) {}
  virtual ~TableGenIndexSymbol() = default;

  // The main definition of the symbol.
  PointerUnion<const Record *, const RecordVal *> definition;

```
- **EN**: Introduces declarations for `TableGenIndexSymbol`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `TableGenIndexSymbol` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 161-175
```cpp
  /// The source location of the definition.
  SMRange defLoc;

  /// The source location of the references of the definition.
  SmallVector<SMRange> references;
};
/// This class represents a single record symbol.
struct TableGenRecordSymbol : public TableGenIndexSymbol {
  TableGenRecordSymbol(const Record *record) : TableGenIndexSymbol(record) {}
  ~TableGenRecordSymbol() override = default;

  static bool classof(const TableGenIndexSymbol *symbol) {
    return isa<const Record *>(symbol->definition);
  }

```
- **EN**: Introduces declarations for `TableGenRecordSymbol`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `TableGenRecordSymbol` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 176-189
```cpp
  /// Return the value of this symbol.
  const Record *getValue() const { return cast<const Record *>(definition); }
};
/// This class represents a single record value symbol.
struct TableGenRecordValSymbol : public TableGenIndexSymbol {
  TableGenRecordValSymbol(const Record *record, const RecordVal *value)
      : TableGenIndexSymbol(value), record(record) {}
  ~TableGenRecordValSymbol() override = default;

  static bool classof(const TableGenIndexSymbol *symbol) {
    return isa<const RecordVal *>(symbol->definition);
  }

  /// Return the value of this symbol.
```
- **EN**: Introduces declarations for `TableGenRecordValSymbol`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `TableGenRecordValSymbol` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 190-204
```cpp
  const RecordVal *getValue() const {
    return cast<const RecordVal *>(definition);
  }

  /// The parent record of this symbol.
  const Record *record;
};

/// This class provides an index for definitions/uses within a TableGen
/// document. It provides efficient lookup of a definition given an input source
/// range.
class TableGenIndex {
public:
  TableGenIndex() : intervalMap(allocator) {}

```
- **EN**: Introduces declarations for `TableGenIndex`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `TableGenIndex` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 205-222
```cpp
  /// Initialize the index with the given RecordKeeper.
  void initialize(const RecordKeeper &records);

  /// Lookup a symbol for the given location. Returns nullptr if no symbol could
  /// be found. If provided, `overlappedRange` is set to the range that the
  /// provided `loc` overlapped with.
  const TableGenIndexSymbol *lookup(SMLoc loc,
                                    SMRange *overlappedRange = nullptr) const;

private:
  /// The type of interval map used to store source references. SMRange is
  /// half-open, so we also need to use a half-open interval map.
  using MapT = llvm::IntervalMap<
      const char *, const TableGenIndexSymbol *,
      llvm::IntervalMapImpl::NodeSizer<const char *,
                                       const TableGenIndexSymbol *>::LeafSize,
      llvm::IntervalMapHalfOpenInfo<const char *>>;

```
- **EN**: Implements logic around `initialize`, `lookup`; this block manipulates core MLIR IR objects; processes TableGen records or generates derived code.
- **CN**: 围绕 `initialize`、`lookup` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并处理 TableGen 记录或生成派生代码。

### Lines 223-240
```cpp
  /// Get or insert a symbol for the given record.
  TableGenIndexSymbol *getOrInsertDef(const Record *record) {
    auto it = defToSymbol.try_emplace(record, nullptr);
    if (it.second)
      it.first->second = std::make_unique<TableGenRecordSymbol>(record);
    return &*it.first->second;
  }
  /// Get or insert a symbol for the given record value.
  TableGenIndexSymbol *getOrInsertDef(const Record *record,
                                      const RecordVal *value) {
    auto it = defToSymbol.try_emplace(value, nullptr);
    if (it.second) {
      it.first->second =
          std::make_unique<TableGenRecordValSymbol>(record, value);
    }
    return &*it.first->second;
  }

```
- **EN**: Implements logic around `getOrInsertDef`, `try_emplace`, `make_unique`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getOrInsertDef`、`try_emplace`、`make_unique` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 241-256
```cpp
  /// An allocator for the interval map.
  MapT::Allocator allocator;

  /// An interval map containing a corresponding definition mapped to a source
  /// interval.
  MapT intervalMap;

  /// A mapping between definitions and their corresponding symbol.
  DenseMap<const void *, std::unique_ptr<TableGenIndexSymbol>> defToSymbol;
};
} // namespace

void TableGenIndex::initialize(const RecordKeeper &records) {
  intervalMap.clear();
  defToSymbol.clear();

```
- **EN**: Implements logic around `initialize`, `clear`; this block processes TableGen records or generates derived code.
- **CN**: 围绕 `initialize`、`clear` 实现具体逻辑；该代码块处理 TableGen 记录或生成派生代码。

### Lines 257-274
```cpp
  auto insertRef = [&](TableGenIndexSymbol *sym, SMRange refLoc,
                       bool isDef = false) {
    const char *startLoc = refLoc.Start.getPointer();
    const char *endLoc = refLoc.End.getPointer();

    // If the location we got was empty, try to lex a token from the start
    // location.
    if (startLoc == endLoc) {
      refLoc = convertTokenLocToRange(SMLoc::getFromPointer(startLoc));
      startLoc = refLoc.Start.getPointer();
      endLoc = refLoc.End.getPointer();

      // If the location is still empty, bail on trying to use this reference
      // location.
      if (startLoc == endLoc)
        return;
    }

```
- **EN**: Implements logic around `getPointer`, `convertTokenLocToRange`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getPointer`、`convertTokenLocToRange` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 275-292
```cpp
    // Check to see if a symbol is already attached to this location.
    // IntervalMap doesn't allow overlapping inserts, and we don't really
    // want multiple symbols attached to a source location anyways. This
    // shouldn't really happen in practice, but we should handle it gracefully.
    if (!intervalMap.overlaps(startLoc, endLoc))
      intervalMap.insert(startLoc, endLoc, sym);

    if (!isDef)
      sym->references.push_back(refLoc);
  };
  auto classes =
      llvm::make_pointee_range(llvm::make_second_range(records.getClasses()));
  auto defs =
      llvm::make_pointee_range(llvm::make_second_range(records.getDefs()));
  for (const Record &def : llvm::concat<Record>(classes, defs)) {
    auto *sym = getOrInsertDef(&def);
    insertRef(sym, sym->defLoc, /*isDef=*/true);

```
- **EN**: Implements logic around `overlaps`, `insert`, `push_back`, `make_pointee_range`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `overlaps`、`insert`、`push_back`、`make_pointee_range` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 293-308
```cpp
    // Add references to the definition.
    for (SMLoc loc : def.getLoc().drop_front())
      insertRef(sym, convertTokenLocToRange(loc));
    for (SMRange loc : def.getReferenceLocs())
      insertRef(sym, loc);

    // Add definitions for any values.
    for (const RecordVal &value : def.getValues()) {
      auto *sym = getOrInsertDef(&def, &value);
      insertRef(sym, sym->defLoc, /*isDef=*/true);
      for (SMRange refLoc : value.getReferenceLocs())
        insertRef(sym, refLoc);
    }
  }
}

```
- **EN**: Implements logic around `getLoc`, `insertRef`, `getReferenceLocs`, `getValues`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getLoc`、`insertRef`、`getReferenceLocs`、`getValues` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 309-322
```cpp
const TableGenIndexSymbol *
TableGenIndex::lookup(SMLoc loc, SMRange *overlappedRange) const {
  auto it = intervalMap.find(loc.getPointer());
  if (!it.valid() || loc.getPointer() < it.start())
    return nullptr;

  if (overlappedRange) {
    *overlappedRange = SMRange(SMLoc::getFromPointer(it.start()),
                               SMLoc::getFromPointer(it.stop()));
  }
  return it.value();
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `lookup`, `find`, `valid`, `SMRange`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `lookup`、`find`、`valid`、`SMRange` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 323-337
```cpp
// TableGenTextFile
//===----------------------------------------------------------------------===//

namespace {
/// This class represents a text file containing one or more TableGen documents.
class TableGenTextFile {
public:
  TableGenTextFile(const llvm::lsp::URIForFile &uri, StringRef fileContents,
                   int64_t version,
                   const std::vector<std::string> &extraIncludeDirs,
                   std::vector<llvm::lsp::Diagnostic> &diagnostics);

  /// Return the current version of this text file.
  int64_t getVersion() const { return version; }

```
- **EN**: Introduces declarations for `TableGenTextFile`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `TableGenTextFile` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 338-355
```cpp
  /// Update the file to the new version using the provided set of content
  /// changes. Returns failure if the update was unsuccessful.
  LogicalResult
  update(const llvm::lsp::URIForFile &uri, int64_t newVersion,
         ArrayRef<llvm::lsp::TextDocumentContentChangeEvent> changes,
         std::vector<llvm::lsp::Diagnostic> &diagnostics);

  //===--------------------------------------------------------------------===//
  // Definitions and References
  //===--------------------------------------------------------------------===//

  void getLocationsOf(const llvm::lsp::URIForFile &uri,
                      const llvm::lsp::Position &defPos,
                      std::vector<llvm::lsp::Location> &locations);
  void findReferencesOf(const llvm::lsp::URIForFile &uri,
                        const llvm::lsp::Position &pos,
                        std::vector<llvm::lsp::Location> &references);

```
- **EN**: Implements logic around `update`, `getLocationsOf`, `findReferencesOf`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects.
- **CN**: 围绕 `update`、`getLocationsOf`、`findReferencesOf` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象。

### Lines 356-378
```cpp
  //===--------------------------------------------------------------------===//
  // Document Links
  //===--------------------------------------------------------------------===//

  void getDocumentLinks(const llvm::lsp::URIForFile &uri,
                        std::vector<llvm::lsp::DocumentLink> &links);

  //===--------------------------------------------------------------------===//
  // Hover
  //===--------------------------------------------------------------------===//

  std::optional<llvm::lsp::Hover>
  findHover(const llvm::lsp::URIForFile &uri,
            const llvm::lsp::Position &hoverPos);
  llvm::lsp::Hover buildHoverForRecord(const Record *record,
                                       const SMRange &hoverRange);
  llvm::lsp::Hover buildHoverForTemplateArg(const Record *record,
                                            const RecordVal *value,
                                            const SMRange &hoverRange);
  llvm::lsp::Hover buildHoverForField(const Record *record,
                                      const RecordVal *value,
                                      const SMRange &hoverRange);

```
- **EN**: Implements logic around `getDocumentLinks`, `findHover`, `buildHoverForRecord`, `buildHoverForTemplateArg`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getDocumentLinks`、`findHover`、`buildHoverForRecord`、`buildHoverForTemplateArg` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 379-392
```cpp
private:
  /// Initialize the text file from the given file contents.
  void initialize(const llvm::lsp::URIForFile &uri, int64_t newVersion,
                  std::vector<llvm::lsp::Diagnostic> &diagnostics);

  /// The full string contents of the file.
  std::string contents;

  /// The version of this file.
  int64_t version;

  /// The include directories for this file.
  std::vector<std::string> includeDirs;

```
- **EN**: Implements logic around `initialize`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `initialize` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 393-406
```cpp
  /// The source manager containing the contents of the input file.
  SourceMgr sourceMgr;

  /// The record keeper containing the parsed tablegen constructs.
  std::unique_ptr<RecordKeeper> recordKeeper;

  /// The index of the parsed file.
  TableGenIndex index;

  /// The set of includes of the parsed file.
  SmallVector<lsp::SourceMgrInclude> parsedIncludes;
};
} // namespace

```
- **EN**: Contains supporting implementation details for the surrounding MLIR component.
- **CN**: 包含周边 MLIR 组件所需的辅助实现细节。

### Lines 407-421
```cpp
TableGenTextFile::TableGenTextFile(
    const llvm::lsp::URIForFile &uri, StringRef fileContents, int64_t version,
    const std::vector<std::string> &extraIncludeDirs,
    std::vector<llvm::lsp::Diagnostic> &diagnostics)
    : contents(fileContents.str()), version(version) {
  // Build the set of include directories for this file.
  llvm::SmallString<32> uriDirectory(uri.file());
  llvm::sys::path::remove_filename(uriDirectory);
  includeDirs.push_back(uriDirectory.str().str());
  llvm::append_range(includeDirs, extraIncludeDirs);

  // Initialize the file.
  initialize(uri, version, diagnostics);
}

```
- **EN**: Implements logic around `TableGenTextFile`, `contents`, `uriDirectory`, `remove_filename`, and 3 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `TableGenTextFile`、`contents`、`uriDirectory`、`remove_filename` 等另外 3 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 422-436
```cpp
LogicalResult TableGenTextFile::update(
    const llvm::lsp::URIForFile &uri, int64_t newVersion,
    ArrayRef<llvm::lsp::TextDocumentContentChangeEvent> changes,
    std::vector<llvm::lsp::Diagnostic> &diagnostics) {
  if (failed(llvm::lsp::TextDocumentContentChangeEvent::applyTo(changes,
                                                                contents))) {
    llvm::lsp::Logger::error("Failed to update contents of {0}", uri.file());
    return failure();
  }

  // If the file contents were properly changed, reinitialize the text file.
  initialize(uri, newVersion, diagnostics);
  return success();
}

```
- **EN**: Implements logic around `update`, `failed`, `error`, `failure`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `update`、`failed`、`error`、`failure` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 437-454
```cpp
void TableGenTextFile::initialize(
    const llvm::lsp::URIForFile &uri, int64_t newVersion,
    std::vector<llvm::lsp::Diagnostic> &diagnostics) {
  version = newVersion;
  sourceMgr = SourceMgr();
  recordKeeper = std::make_unique<RecordKeeper>();

  // Build a buffer for this file.
  auto memBuffer = llvm::MemoryBuffer::getMemBuffer(contents, uri.file());
  if (!memBuffer) {
    llvm::lsp::Logger::error("Failed to create memory buffer for file",
                             uri.file());
    return;
  }
  sourceMgr.setIncludeDirs(includeDirs);
  sourceMgr.setVirtualFileSystem(llvm::vfs::getRealFileSystem());
  sourceMgr.AddNewSourceBuffer(std::move(memBuffer), SMLoc());

```
- **EN**: Implements logic around `initialize`, `SourceMgr`, `make_unique`, `getMemBuffer`, and 5 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; processes TableGen records or generates derived code.
- **CN**: 围绕 `initialize`、`SourceMgr`、`make_unique`、`getMemBuffer` 等另外 5 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并处理 TableGen 记录或生成派生代码。

### Lines 455-471
```cpp
  // This class provides a context argument for the SourceMgr diagnostic
  // handler.
  struct DiagHandlerContext {
    std::vector<llvm::lsp::Diagnostic> &diagnostics;
    const llvm::lsp::URIForFile &uri;
  } handlerContext{diagnostics, uri};

  // Set the diagnostic handler for the tablegen source manager.
  sourceMgr.setDiagHandler(
      [](const llvm::SMDiagnostic &diag, void *rawHandlerContext) {
        auto *ctx = reinterpret_cast<DiagHandlerContext *>(rawHandlerContext);
        if (auto lspDiag = getLspDiagnoticFromDiag(diag, ctx->uri))
          ctx->diagnostics.push_back(*lspDiag);
      },
      &handlerContext);
  bool failedToParse = llvm::TableGenParseFile(sourceMgr, *recordKeeper);

```
- **EN**: Introduces declarations for `DiagHandlerContext`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `DiagHandlerContext` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 472-492
```cpp
  // Process all of the include files.
  lsp::gatherIncludeFiles(sourceMgr, parsedIncludes);
  if (failedToParse)
    return;

  // If we successfully parsed the file, we can now build the index.
  index.initialize(*recordKeeper);
}

//===----------------------------------------------------------------------===//
// TableGenTextFile: Definitions and References
//===----------------------------------------------------------------------===//

void TableGenTextFile::getLocationsOf(
    const llvm::lsp::URIForFile &uri, const llvm::lsp::Position &defPos,
    std::vector<llvm::lsp::Location> &locations) {
  SMLoc posLoc = defPos.getAsSMLoc(sourceMgr);
  const TableGenIndexSymbol *symbol = index.lookup(posLoc);
  if (!symbol)
    return;

```
- **EN**: Implements logic around `gatherIncludeFiles`, `initialize`, `getLocationsOf`, `getAsSMLoc`, and 1 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations; processes TableGen records or generates derived code.
- **CN**: 围绕 `gatherIncludeFiles`、`initialize`、`getLocationsOf`、`getAsSMLoc` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示，并处理 TableGen 记录或生成派生代码。

### Lines 493-506
```cpp
  // If this symbol is a record value and the def position is already the def of
  // the symbol, check to see if the value has a base definition. This allows
  // for a "go-to-def" on a "let" to resolve the definition in the base class.
  auto *valSym = dyn_cast<TableGenRecordValSymbol>(symbol);
  if (valSym && lsp::contains(valSym->defLoc, posLoc)) {
    if (auto *val = getBaseValue(valSym->record, valSym->getValue()).second) {
      locations.push_back(getLocationFromLoc(sourceMgr, val->getLoc(), uri));
      return;
    }
  }

  locations.push_back(getLocationFromLoc(sourceMgr, symbol->defLoc, uri));
}

```
- **EN**: Implements logic around `dyn_cast`, `contains`, `getBaseValue`, `push_back`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `dyn_cast`、`contains`、`getBaseValue`、`push_back` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 507-520
```cpp
void TableGenTextFile::findReferencesOf(
    const llvm::lsp::URIForFile &uri, const llvm::lsp::Position &pos,
    std::vector<llvm::lsp::Location> &references) {
  SMLoc posLoc = pos.getAsSMLoc(sourceMgr);
  const TableGenIndexSymbol *symbol = index.lookup(posLoc);
  if (!symbol)
    return;

  references.push_back(getLocationFromLoc(sourceMgr, symbol->defLoc, uri));
  for (SMRange refLoc : symbol->references)
    references.push_back(getLocationFromLoc(sourceMgr, refLoc, uri));
}

//===--------------------------------------------------------------------===//
```
- **EN**: Implements logic around `findReferencesOf`, `getAsSMLoc`, `lookup`, `push_back`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `findReferencesOf`、`getAsSMLoc`、`lookup`、`push_back` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 521-534
```cpp
// TableGenTextFile: Document Links
//===--------------------------------------------------------------------===//

void TableGenTextFile::getDocumentLinks(
    const llvm::lsp::URIForFile &uri,
    std::vector<llvm::lsp::DocumentLink> &links) {
  for (const lsp::SourceMgrInclude &include : parsedIncludes)
    links.emplace_back(include.range, include.uri);
}

//===----------------------------------------------------------------------===//
// TableGenTextFile: Hover
//===----------------------------------------------------------------------===//

```
- **EN**: Implements logic around `getDocumentLinks`, `emplace_back`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `getDocumentLinks`、`emplace_back` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 535-549
```cpp
std::optional<llvm::lsp::Hover>
TableGenTextFile::findHover(const llvm::lsp::URIForFile &uri,
                            const llvm::lsp::Position &hoverPos) {
  // Check for a reference to an include.
  for (const lsp::SourceMgrInclude &include : parsedIncludes)
    if (include.range.contains(hoverPos))
      return include.buildHover();

  // Find the symbol at the given location.
  SMRange hoverRange;
  SMLoc posLoc = hoverPos.getAsSMLoc(sourceMgr);
  const TableGenIndexSymbol *symbol = index.lookup(posLoc, &hoverRange);
  if (!symbol)
    return std::nullopt;

```
- **EN**: Implements logic around `findHover`, `contains`, `buildHover`, `getAsSMLoc`, and 1 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `findHover`、`contains`、`buildHover`、`getAsSMLoc` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 550-569
```cpp
  // Build hover for a Record.
  if (auto *record = dyn_cast<TableGenRecordSymbol>(symbol))
    return buildHoverForRecord(record->getValue(), hoverRange);

  // Build hover for a RecordVal, which is either a template argument or a
  // field.
  auto *recordVal = cast<TableGenRecordValSymbol>(symbol);
  const RecordVal *value = recordVal->getValue();
  if (value->isTemplateArg())
    return buildHoverForTemplateArg(recordVal->record, value, hoverRange);
  return buildHoverForField(recordVal->record, value, hoverRange);
}

llvm::lsp::Hover
TableGenTextFile::buildHoverForRecord(const Record *record,
                                      const SMRange &hoverRange) {
  llvm::lsp::Hover hover(llvm::lsp::Range(sourceMgr, hoverRange));
  {
    llvm::raw_string_ostream hoverOS(hover.contents.value);

```
- **EN**: Implements logic around `dyn_cast`, `buildHoverForRecord`, `cast`, `getValue`, and 5 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `dyn_cast`、`buildHoverForRecord`、`cast`、`getValue` 等另外 5 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 570-591
```cpp
    // Format the type of record this is.
    if (record->isClass()) {
      hoverOS << "**class** `" << record->getName() << "`";
    } else if (record->isAnonymous()) {
      hoverOS << "**anonymous class**";
    } else {
      hoverOS << "**def** `" << record->getName() << "`";
    }
    hoverOS << "\n***\n";

    // Check if this record has summary/description fields. These are often used
    // to hold documentation for the record.
    auto printAndFormatField = [&](StringRef fieldName) {
      // Check that the record actually has the given field, and that it's a
      // string.
      const RecordVal *value = record->getValue(fieldName);
      if (!value || !value->getValue())
        return;
      auto *stringValue = dyn_cast<llvm::StringInit>(value->getValue());
      if (!stringValue)
        return;

```
- **EN**: Implements logic around `isClass`, `getName`, `isAnonymous`, `getValue`, and 1 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `isClass`、`getName`、`isAnonymous`、`getValue` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 592-607
```cpp
      raw_indented_ostream ros(hoverOS);
      ros.printReindented(stringValue->getValue().rtrim(" \t"));
      hoverOS << "\n***\n";
    };
    printAndFormatField("summary");
    printAndFormatField("description");

    // Check for documentation in the source file.
    if (std::optional<std::string> doc =
            lsp::extractSourceDocComment(sourceMgr, record->getLoc().front())) {
      hoverOS << "\n" << *doc << "\n";
    }
  }
  return hover;
}

```
- **EN**: Implements logic around `ros`, `printReindented`, `printAndFormatField`, `extractSourceDocComment`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `ros`、`printReindented`、`printAndFormatField`、`extractSourceDocComment` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 608-621
```cpp
llvm::lsp::Hover TableGenTextFile::buildHoverForTemplateArg(
    const Record *record, const RecordVal *value, const SMRange &hoverRange) {
  llvm::lsp::Hover hover(llvm::lsp::Range(sourceMgr, hoverRange));
  {
    llvm::raw_string_ostream hoverOS(hover.contents.value);
    StringRef name = value->getName().rsplit(':').second;

    hoverOS << "**template arg** `" << name << "`\n***\nType: `";
    value->getType()->print(hoverOS);
    hoverOS << "`\n";
  }
  return hover;
}

```
- **EN**: Implements logic around `buildHoverForTemplateArg`, `hover`, `hoverOS`, `getName`, and 1 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `buildHoverForTemplateArg`、`hover`、`hoverOS`、`getName` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 622-637
```cpp
llvm::lsp::Hover TableGenTextFile::buildHoverForField(
    const Record *record, const RecordVal *value, const SMRange &hoverRange) {
  llvm::lsp::Hover hover(llvm::lsp::Range(sourceMgr, hoverRange));
  {
    llvm::raw_string_ostream hoverOS(hover.contents.value);
    hoverOS << "**field** `" << value->getName() << "`\n***\nType: `";
    value->getType()->print(hoverOS);
    hoverOS << "`\n***\n";

    // Check for documentation in the source file.
    if (std::optional<std::string> doc =
            lsp::extractSourceDocComment(sourceMgr, value->getLoc())) {
      hoverOS << "\n" << *doc << "\n";
      hoverOS << "\n***\n";
    }

```
- **EN**: Implements logic around `buildHoverForField`, `hover`, `hoverOS`, `getName`, and 2 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `buildHoverForField`、`hover`、`hoverOS`、`getName` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 638-651
```cpp
    // Check to see if there is a base value that we can use for
    // documentation.
    auto [baseRecord, baseValue] = getBaseValue(record, value);
    if (baseValue) {
      if (std::optional<std::string> doc =
              lsp::extractSourceDocComment(sourceMgr, baseValue->getLoc())) {
        hoverOS << "\n *From `" << baseRecord->getName() << "`*:\n\n"
                << *doc << "\n";
      }
    }
  }
  return hover;
}

```
- **EN**: Implements logic around `getBaseValue`, `extractSourceDocComment`, `getName`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getBaseValue`、`extractSourceDocComment`、`getName` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 652-666
```cpp
//===----------------------------------------------------------------------===//
// TableGenServer::Impl
//===----------------------------------------------------------------------===//

struct lsp::TableGenServer::Impl {
  explicit Impl(const Options &options)
      : options(options), compilationDatabase(options.compilationDatabases) {}

  /// TableGen LSP options.
  const Options &options;

  /// The compilation database containing additional information for files
  /// passed to the server.
  lsp::CompilationDatabase compilationDatabase;

```
- **EN**: Introduces declarations for `lsp::TableGenServer::Impl`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `lsp::TableGenServer::Impl` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 667-686
```cpp
  /// The files held by the server, mapped by their URI file name.
  llvm::StringMap<std::unique_ptr<TableGenTextFile>> files;
};

//===----------------------------------------------------------------------===//
// TableGenServer
//===----------------------------------------------------------------------===//

lsp::TableGenServer::TableGenServer(const Options &options)
    : impl(std::make_unique<Impl>(options)) {}
lsp::TableGenServer::~TableGenServer() = default;

void lsp::TableGenServer::addDocument(const URIForFile &uri, StringRef contents,
                                      int64_t version,
                                      std::vector<Diagnostic> &diagnostics) {
  // Build the set of additional include directories.
  std::vector<std::string> additionalIncludeDirs = impl->options.extraDirs;
  const auto &fileInfo = impl->compilationDatabase.getFileInfo(uri.file());
  llvm::append_range(additionalIncludeDirs, fileInfo.includeDirs);

```
- **EN**: Implements logic around `TableGenServer`, `impl`, `~TableGenServer`, `addDocument`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `TableGenServer`、`impl`、`~TableGenServer`、`addDocument` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 687-704
```cpp
  impl->files[uri.file()] = std::make_unique<TableGenTextFile>(
      uri, contents, version, additionalIncludeDirs, diagnostics);
}

void lsp::TableGenServer::updateDocument(
    const URIForFile &uri, ArrayRef<TextDocumentContentChangeEvent> changes,
    int64_t version, std::vector<Diagnostic> &diagnostics) {
  // Check that we actually have a document for this uri.
  auto it = impl->files.find(uri.file());
  if (it == impl->files.end())
    return;

  // Try to update the document. If we fail, erase the file from the server. A
  // failed updated generally means we've fallen out of sync somewhere.
  if (failed(it->second->update(uri, version, changes, diagnostics)))
    impl->files.erase(it);
}

```
- **EN**: Implements logic around `file`, `updateDocument`, `find`, `end`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `file`、`updateDocument`、`find`、`end` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 705-723
```cpp
std::optional<int64_t>
lsp::TableGenServer::removeDocument(const URIForFile &uri) {
  auto it = impl->files.find(uri.file());
  if (it == impl->files.end())
    return std::nullopt;

  int64_t version = it->second->getVersion();
  impl->files.erase(it);
  return version;
}

void lsp::TableGenServer::getLocationsOf(const URIForFile &uri,
                                         const Position &defPos,
                                         std::vector<Location> &locations) {
  auto fileIt = impl->files.find(uri.file());
  if (fileIt != impl->files.end())
    fileIt->second->getLocationsOf(uri, defPos, locations);
}

```
- **EN**: Implements logic around `removeDocument`, `find`, `end`, `getVersion`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `removeDocument`、`find`、`end`、`getVersion` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 724-738
```cpp
void lsp::TableGenServer::findReferencesOf(const URIForFile &uri,
                                           const Position &pos,
                                           std::vector<Location> &references) {
  auto fileIt = impl->files.find(uri.file());
  if (fileIt != impl->files.end())
    fileIt->second->findReferencesOf(uri, pos, references);
}

void lsp::TableGenServer::getDocumentLinks(
    const URIForFile &uri, std::vector<DocumentLink> &documentLinks) {
  auto fileIt = impl->files.find(uri.file());
  if (fileIt != impl->files.end())
    return fileIt->second->getDocumentLinks(uri, documentLinks);
}

```
- **EN**: Implements logic around `findReferencesOf`, `find`, `end`, `getDocumentLinks`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `findReferencesOf`、`find`、`end`、`getDocumentLinks` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 739-746
```cpp
std::optional<llvm::lsp::Hover>
lsp::TableGenServer::findHover(const URIForFile &uri,
                               const Position &hoverPos) {
  auto fileIt = impl->files.find(uri.file());
  if (fileIt != impl->files.end())
    return fileIt->second->findHover(uri, hoverPos);
  return std::nullopt;
}
```
- **EN**: Implements logic around `findHover`, `find`, `end`.
- **CN**: 围绕 `findHover`、`find`、`end` 实现具体逻辑。

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
- **TableGen-driven generation / TableGen 驱动生成**:
  - **EN**: Declarative records are converted into generated MLIR declarations or implementation fragments.
  - **CN**: 把声明式记录转换为生成的 MLIR 声明或实现片段。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `TableGenServer.h`, `mlir/Support/IndentedOstream.h`, `mlir/Tools/lsp-server-support/CompilationDatabase.h`, `mlir/Tools/lsp-server-support/SourceMgrUtils.h`, `llvm/ADT/IntervalMap.h`, `llvm/ADT/PointerUnion.h`, `llvm/ADT/StringMap.h`, `llvm/Support/LSP/Logging.h`, `llvm/Support/LSP/Protocol.h`, `llvm/Support/Path.h` ... (+3 more)
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: LLVM support-library helpers / LLVM Support 库辅助工具 (4), LLVM ADT containers and utilities / LLVM ADT 容器与工具 (3), tooling support declarations / 工具支持声明 (2), LLVM TableGen infrastructure / LLVM TableGen 基础设施 (2), shared MLIR support helpers / 共享的 MLIR 支持工具 (1)
