# PDLLServer.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Tools/mlir-pdll-lsp-server/PDLLServer.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements reusable command-line tool support built on top of MLIR libraries.
  - **CN**: 实现构建于 MLIR 库之上的可复用命令行工具支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-36
```cpp
//===- PDLLServer.cpp - PDLL Language Server ------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "PDLLServer.h"

#include "Protocol.h"
#include "mlir/IR/BuiltinOps.h"
#include "mlir/Support/ToolUtilities.h"
#include "mlir/Tools/PDLL/AST/Context.h"
#include "mlir/Tools/PDLL/AST/Nodes.h"
#include "mlir/Tools/PDLL/AST/Types.h"
#include "mlir/Tools/PDLL/CodeGen/CPPGen.h"
#include "mlir/Tools/PDLL/CodeGen/MLIRGen.h"
#include "mlir/Tools/PDLL/ODS/Constraint.h"
#include "mlir/Tools/PDLL/ODS/Context.h"
#include "mlir/Tools/PDLL/ODS/Dialect.h"
#include "mlir/Tools/PDLL/ODS/Operation.h"
#include "mlir/Tools/PDLL/Parser/CodeComplete.h"
#include "mlir/Tools/PDLL/Parser/Parser.h"
#include "mlir/Tools/lsp-server-support/CompilationDatabase.h"
#include "mlir/Tools/lsp-server-support/SourceMgrUtils.h"
#include "llvm/ADT/IntervalMap.h"
#include "llvm/ADT/StringMap.h"
#include "llvm/ADT/StringSet.h"
#include "llvm/ADT/TypeSwitch.h"
#include "llvm/Support/FileSystem.h"
#include "llvm/Support/LSP/Logging.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/VirtualFileSystem.h"
#include <optional>

```
- **EN**: Pulls in the declarations needed by this translation unit, including `PDLLServer.h`, `Protocol.h`, `mlir/IR/BuiltinOps.h`, `mlir/Support/ToolUtilities.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `PDLLServer.h`, `Protocol.h`, `mlir/IR/BuiltinOps.h`, `mlir/Support/ToolUtilities.h`。

### Lines 37-57
```cpp
using namespace mlir;
using namespace mlir::pdll;

/// Returns a language server uri for the given source location. `mainFileURI`
/// corresponds to the uri for the main file of the source manager.
static llvm::lsp::URIForFile
getURIFromLoc(llvm::SourceMgr &mgr, SMRange loc,
              const llvm::lsp::URIForFile &mainFileURI) {
  int bufferId = mgr.FindBufferContainingLoc(loc.Start);
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
- **EN**: Implements logic around `getURIFromLoc`, `FindBufferContainingLoc`, `static_cast`, `fromFile`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getURIFromLoc`、`FindBufferContainingLoc`、`static_cast`、`fromFile` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 58-78
```cpp
/// Returns true if the given location is in the main file of the source
/// manager.
static bool isMainFileLoc(llvm::SourceMgr &mgr, SMRange loc) {
  return mgr.FindBufferContainingLoc(loc.Start) == mgr.getMainFileID();
}

/// Returns a language server location from the given source range.
static llvm::lsp::Location
getLocationFromLoc(llvm::SourceMgr &mgr, SMRange range,
                   const llvm::lsp::URIForFile &uri) {
  return llvm::lsp::Location(getURIFromLoc(mgr, range, uri),
                             llvm::lsp::Range(mgr, range));
}

/// Convert the given MLIR diagnostic to the LSP form.
static std::optional<llvm::lsp::Diagnostic>
getLspDiagnoticFromDiag(llvm::SourceMgr &sourceMgr, const ast::Diagnostic &diag,
                        const llvm::lsp::URIForFile &uri) {
  llvm::lsp::Diagnostic lspDiag;
  lspDiag.source = "pdll";

```
- **EN**: Implements logic around `isMainFileLoc`, `FindBufferContainingLoc`, `getLocationFromLoc`, `Location`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects.
- **CN**: 围绕 `isMainFileLoc`、`FindBufferContainingLoc`、`getLocationFromLoc`、`Location` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象。

### Lines 79-107
```cpp
  // FIXME: Right now all of the diagnostics are treated as parser issues, but
  // some are parser and some are verifier.
  lspDiag.category = "Parse Error";

  // Try to grab a file location for this diagnostic.
  llvm::lsp::Location loc =
      getLocationFromLoc(sourceMgr, diag.getLocation(), uri);
  lspDiag.range = loc.range;

  // Skip diagnostics that weren't emitted within the main file.
  if (loc.uri != uri)
    return std::nullopt;

  // Convert the severity for the diagnostic.
  switch (diag.getSeverity()) {
  case ast::Diagnostic::Severity::DK_Note:
    llvm_unreachable("expected notes to be handled separately");
  case ast::Diagnostic::Severity::DK_Warning:
    lspDiag.severity = llvm::lsp::DiagnosticSeverity::Warning;
    break;
  case ast::Diagnostic::Severity::DK_Error:
    lspDiag.severity = llvm::lsp::DiagnosticSeverity::Error;
    break;
  case ast::Diagnostic::Severity::DK_Remark:
    lspDiag.severity = llvm::lsp::DiagnosticSeverity::Information;
    break;
  }
  lspDiag.message = diag.getMessage().str();

```
- **EN**: Implements logic around `getLocationFromLoc`, `getSeverity`, `llvm_unreachable`, `getMessage`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `getLocationFromLoc`、`getSeverity`、`llvm_unreachable`、`getMessage` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 108-127
```cpp
  // Attach any notes to the main diagnostic as related information.
  std::vector<llvm::lsp::DiagnosticRelatedInformation> relatedDiags;
  for (const ast::Diagnostic &note : diag.getNotes()) {
    relatedDiags.emplace_back(
        getLocationFromLoc(sourceMgr, note.getLocation(), uri),
        note.getMessage().str());
  }
  if (!relatedDiags.empty())
    lspDiag.relatedInformation = std::move(relatedDiags);

  return lspDiag;
}

/// Get or extract the documentation for the given decl.
static std::optional<std::string>
getDocumentationFor(llvm::SourceMgr &sourceMgr, const ast::Decl *decl) {
  // If the decl already had documentation set, use it.
  if (std::optional<StringRef> doc = decl->getDocComment())
    return doc->str();

```
- **EN**: Implements logic around `getNotes`, `emplace_back`, `getLocationFromLoc`, `getMessage`, and 5 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `getNotes`、`emplace_back`、`getLocationFromLoc`、`getMessage` 等另外 5 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 128-153
```cpp
  // If the decl doesn't yet have documentation, try to extract it from the
  // source file.
  return lsp::extractSourceDocComment(sourceMgr, decl->getLoc().Start);
}

//===----------------------------------------------------------------------===//
// PDLIndex
//===----------------------------------------------------------------------===//

namespace {
struct PDLIndexSymbol {
  explicit PDLIndexSymbol(const ast::Decl *definition)
      : definition(definition) {}
  explicit PDLIndexSymbol(const ods::Operation *definition)
      : definition(definition) {}

  /// Return the location of the definition of this symbol.
  SMRange getDefLoc() const {
    if (const ast::Decl *decl =
            llvm::dyn_cast_if_present<const ast::Decl *>(definition)) {
      const ast::Name *declName = decl->getName();
      return declName ? declName->getLoc() : decl->getLoc();
    }
    return cast<const ods::Operation *>(definition)->getLoc();
  }

```
- **EN**: Introduces declarations for `PDLIndexSymbol`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `PDLIndexSymbol` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 154-171
```cpp
  /// The main definition of the symbol.
  PointerUnion<const ast::Decl *, const ods::Operation *> definition;
  /// The set of references to the symbol.
  std::vector<SMRange> references;
};

/// This class provides an index for definitions/uses within a PDL document.
/// It provides efficient lookup of a definition given an input source range.
class PDLIndex {
public:
  PDLIndex() : intervalMap(allocator) {}

  /// Initialize the index with the given ast::Module.
  void initialize(const ast::Module &module, const ods::Context &odsContext);

  /// Lookup a symbol for the given location. Returns nullptr if no symbol could
  /// be found. If provided, `overlappedRange` is set to the range that the
  /// provided `loc` overlapped with.
```
- **EN**: Introduces declarations for `PDLIndex`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `PDLIndex` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 172-190
```cpp
  const PDLIndexSymbol *lookup(SMLoc loc,
                               SMRange *overlappedRange = nullptr) const;

private:
  /// The type of interval map used to store source references. SMRange is
  /// half-open, so we also need to use a half-open interval map.
  using MapT =
      llvm::IntervalMap<const char *, const PDLIndexSymbol *,
                        llvm::IntervalMapImpl::NodeSizer<
                            const char *, const PDLIndexSymbol *>::LeafSize,
                        llvm::IntervalMapHalfOpenInfo<const char *>>;

  /// An allocator for the interval map.
  MapT::Allocator allocator;

  /// An interval map containing a corresponding definition mapped to a source
  /// interval.
  MapT intervalMap;

```
- **EN**: Implements logic around `lookup`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `lookup` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 191-218
```cpp
  /// A mapping between definitions and their corresponding symbol.
  DenseMap<const void *, std::unique_ptr<PDLIndexSymbol>> defToSymbol;
};
} // namespace

void PDLIndex::initialize(const ast::Module &module,
                          const ods::Context &odsContext) {
  auto getOrInsertDef = [&](const auto *def) -> PDLIndexSymbol * {
    auto it = defToSymbol.try_emplace(def, nullptr);
    if (it.second)
      it.first->second = std::make_unique<PDLIndexSymbol>(def);
    return &*it.first->second;
  };
  auto insertDeclRef = [&](PDLIndexSymbol *sym, SMRange refLoc,
                           bool isDef = false) {
    const char *startLoc = refLoc.Start.getPointer();
    const char *endLoc = refLoc.End.getPointer();
    if (!intervalMap.overlaps(startLoc, endLoc)) {
      intervalMap.insert(startLoc, endLoc, sym);
      if (!isDef)
        sym->references.push_back(refLoc);
    }
  };
  auto insertODSOpRef = [&](StringRef opName, SMRange refLoc) {
    const ods::Operation *odsOp = odsContext.lookupOperation(opName);
    if (!odsOp)
      return;

```
- **EN**: Implements logic around `initialize`, `try_emplace`, `make_unique`, `getPointer`, and 4 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `initialize`、`try_emplace`、`make_unique`、`getPointer` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 219-246
```cpp
    PDLIndexSymbol *symbol = getOrInsertDef(odsOp);
    insertDeclRef(symbol, odsOp->getLoc(), /*isDef=*/true);
    insertDeclRef(symbol, refLoc);
  };

  module.walk([&](const ast::Node *node) {
    // Handle references to PDL decls.
    if (const auto *decl = dyn_cast<ast::OpNameDecl>(node)) {
      if (std::optional<StringRef> name = decl->getName())
        insertODSOpRef(*name, decl->getLoc());
    } else if (const ast::Decl *decl = dyn_cast<ast::Decl>(node)) {
      const ast::Name *name = decl->getName();
      if (!name)
        return;
      PDLIndexSymbol *declSym = getOrInsertDef(decl);
      insertDeclRef(declSym, name->getLoc(), /*isDef=*/true);

      if (const auto *varDecl = dyn_cast<ast::VariableDecl>(decl)) {
        // Record references to any constraints.
        for (const auto &it : varDecl->getConstraints())
          insertDeclRef(getOrInsertDef(it.constraint), it.referenceLoc);
      }
    } else if (const auto *expr = dyn_cast<ast::DeclRefExpr>(node)) {
      insertDeclRef(getOrInsertDef(expr->getDecl()), expr->getLoc());
    }
  });
}

```
- **EN**: Implements logic around `getOrInsertDef`, `insertDeclRef`, `walk`, `OpNameDecl>`, and 6 more symbols.
- **CN**: 围绕 `getOrInsertDef`、`insertDeclRef`、`walk`、`OpNameDecl>` 等另外 6 个符号 实现具体逻辑。

### Lines 247-264
```cpp
const PDLIndexSymbol *PDLIndex::lookup(SMLoc loc,
                                       SMRange *overlappedRange) const {
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
// PDLDocument
//===----------------------------------------------------------------------===//

namespace {
```
- **EN**: Implements logic around `lookup`, `find`, `valid`, `SMRange`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `lookup`、`find`、`valid`、`SMRange` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 265-284
```cpp
/// This class represents all of the information pertaining to a specific PDL
/// document.
struct PDLDocument {
  PDLDocument(const llvm::lsp::URIForFile &uri, StringRef contents,
              const std::vector<std::string> &extraDirs,
              std::vector<llvm::lsp::Diagnostic> &diagnostics);
  PDLDocument(const PDLDocument &) = delete;
  PDLDocument &operator=(const PDLDocument &) = delete;

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
- **EN**: Introduces declarations for `PDLDocument`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `PDLDocument` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 285-314
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
  std::optional<llvm::lsp::Hover> findHover(const ast::Decl *decl,
                                            const SMRange &hoverRange);
  llvm::lsp::Hover buildHoverForOpName(const ods::Operation *op,
                                       const SMRange &hoverRange);
  llvm::lsp::Hover buildHoverForVariable(const ast::VariableDecl *varDecl,
                                         const SMRange &hoverRange);
  llvm::lsp::Hover buildHoverForPattern(const ast::PatternDecl *decl,
                                        const SMRange &hoverRange);
  llvm::lsp::Hover
  buildHoverForCoreConstraint(const ast::CoreConstraintDecl *decl,
                              const SMRange &hoverRange);
  template <typename T>
  llvm::lsp::Hover
  buildHoverForUserConstraintOrRewrite(StringRef typeName, const T *decl,
                                       const SMRange &hoverRange);

```
- **EN**: Implements logic around `getDocumentLinks`, `findHover`, `buildHoverForOpName`, `buildHoverForVariable`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getDocumentLinks`、`findHover`、`buildHoverForOpName`、`buildHoverForVariable` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 315-332
```cpp
  //===--------------------------------------------------------------------===//
  // Document Symbols
  //===--------------------------------------------------------------------===//

  void findDocumentSymbols(std::vector<llvm::lsp::DocumentSymbol> &symbols);

  //===--------------------------------------------------------------------===//
  // Code Completion
  //===--------------------------------------------------------------------===//

  llvm::lsp::CompletionList
  getCodeCompletion(const llvm::lsp::URIForFile &uri,
                    const llvm::lsp::Position &completePos);

  //===--------------------------------------------------------------------===//
  // Signature Help
  //===--------------------------------------------------------------------===//

```
- **EN**: Implements logic around `findDocumentSymbols`, `getCodeCompletion`.
- **CN**: 围绕 `findDocumentSymbols`、`getCodeCompletion` 实现具体逻辑。

### Lines 333-352
```cpp
  llvm::lsp::SignatureHelp getSignatureHelp(const llvm::lsp::URIForFile &uri,
                                            const llvm::lsp::Position &helpPos);

  //===--------------------------------------------------------------------===//
  // Inlay Hints
  //===--------------------------------------------------------------------===//

  void getInlayHints(const llvm::lsp::URIForFile &uri,
                     const llvm::lsp::Range &range,
                     std::vector<llvm::lsp::InlayHint> &inlayHints);
  void getInlayHintsFor(const ast::VariableDecl *decl,
                        const llvm::lsp::URIForFile &uri,
                        std::vector<llvm::lsp::InlayHint> &inlayHints);
  void getInlayHintsFor(const ast::CallExpr *expr,
                        const llvm::lsp::URIForFile &uri,
                        std::vector<llvm::lsp::InlayHint> &inlayHints);
  void getInlayHintsFor(const ast::OperationExpr *expr,
                        const llvm::lsp::URIForFile &uri,
                        std::vector<llvm::lsp::InlayHint> &inlayHints);

```
- **EN**: Implements logic around `getSignatureHelp`, `getInlayHints`, `getInlayHintsFor`.
- **CN**: 围绕 `getSignatureHelp`、`getInlayHints`、`getInlayHintsFor` 实现具体逻辑。

### Lines 353-370
```cpp
  /// Add a parameter hint for the given expression using `label`.
  void addParameterHintFor(std::vector<llvm::lsp::InlayHint> &inlayHints,
                           const ast::Expr *expr, StringRef label);

  //===--------------------------------------------------------------------===//
  // PDLL ViewOutput
  //===--------------------------------------------------------------------===//

  void getPDLLViewOutput(raw_ostream &os, lsp::PDLLViewOutputKind kind);

  //===--------------------------------------------------------------------===//
  // Fields
  //===--------------------------------------------------------------------===//

  /// The include directories for this file.
  std::vector<std::string> includeDirs;

  /// The source manager containing the contents of the input file.
```
- **EN**: Implements logic around `addParameterHintFor`, `getPDLLViewOutput`.
- **CN**: 围绕 `addParameterHintFor`、`getPDLLViewOutput` 实现具体逻辑。

### Lines 371-398
```cpp
  llvm::SourceMgr sourceMgr;

  /// The ODS and AST contexts.
  ods::Context odsContext;
  ast::Context astContext;

  /// The parsed AST module, or failure if the file wasn't valid.
  FailureOr<ast::Module *> astModule;

  /// The index of the parsed module.
  PDLIndex index;

  /// The set of includes of the parsed module.
  SmallVector<lsp::SourceMgrInclude> parsedIncludes;
};
} // namespace

PDLDocument::PDLDocument(const llvm::lsp::URIForFile &uri, StringRef contents,
                         const std::vector<std::string> &extraDirs,
                         std::vector<llvm::lsp::Diagnostic> &diagnostics)
    : astContext(odsContext) {
  auto memBuffer = llvm::MemoryBuffer::getMemBufferCopy(contents, uri.file());
  if (!memBuffer) {
    llvm::lsp::Logger::error("Failed to create memory buffer for file",
                             uri.file());
    return;
  }

```
- **EN**: Implements logic around `PDLDocument`, `astContext`, `getMemBufferCopy`, `error`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; parses or prints textual MLIR representations.
- **CN**: 围绕 `PDLDocument`、`astContext`、`getMemBufferCopy`、`error` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并解析或打印文本形式的 MLIR 表示。

### Lines 399-417
```cpp
  // Build the set of include directories for this file.
  llvm::SmallString<32> uriDirectory(uri.file());
  llvm::sys::path::remove_filename(uriDirectory);
  includeDirs.push_back(uriDirectory.str().str());
  llvm::append_range(includeDirs, extraDirs);

  sourceMgr.setIncludeDirs(includeDirs);
  sourceMgr.setVirtualFileSystem(llvm::vfs::getRealFileSystem());
  sourceMgr.AddNewSourceBuffer(std::move(memBuffer), SMLoc());

  astContext.getDiagEngine().setHandlerFn([&](const ast::Diagnostic &diag) {
    if (auto lspDiag = getLspDiagnoticFromDiag(sourceMgr, diag, uri))
      diagnostics.push_back(std::move(*lspDiag));
  });
  astModule = parsePDLLAST(astContext, sourceMgr, /*enableDocumentation=*/true);

  // Initialize the set of parsed includes.
  lsp::gatherIncludeFiles(sourceMgr, parsedIncludes);

```
- **EN**: Implements logic around `uriDirectory`, `remove_filename`, `push_back`, `append_range`, and 7 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; parses or prints textual MLIR representations.
- **CN**: 围绕 `uriDirectory`、`remove_filename`、`push_back`、`append_range` 等另外 7 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并解析或打印文本形式的 MLIR 表示。

### Lines 418-437
```cpp
  // If we failed to parse the module, there is nothing left to initialize.
  if (failed(astModule))
    return;

  // Prepare the AST index with the parsed module.
  index.initialize(**astModule, odsContext);
}

//===----------------------------------------------------------------------===//
// PDLDocument: Definitions and References
//===----------------------------------------------------------------------===//

void PDLDocument::getLocationsOf(const llvm::lsp::URIForFile &uri,
                                 const llvm::lsp::Position &defPos,
                                 std::vector<llvm::lsp::Location> &locations) {
  SMLoc posLoc = defPos.getAsSMLoc(sourceMgr);
  const PDLIndexSymbol *symbol = index.lookup(posLoc);
  if (!symbol)
    return;

```
- **EN**: Implements logic around `failed`, `initialize`, `getLocationsOf`, `getAsSMLoc`, and 1 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `failed`、`initialize`、`getLocationsOf`、`getAsSMLoc` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 438-456
```cpp
  locations.push_back(getLocationFromLoc(sourceMgr, symbol->getDefLoc(), uri));
}

void PDLDocument::findReferencesOf(
    const llvm::lsp::URIForFile &uri, const llvm::lsp::Position &pos,
    std::vector<llvm::lsp::Location> &references) {
  SMLoc posLoc = pos.getAsSMLoc(sourceMgr);
  const PDLIndexSymbol *symbol = index.lookup(posLoc);
  if (!symbol)
    return;

  references.push_back(getLocationFromLoc(sourceMgr, symbol->getDefLoc(), uri));
  for (SMRange refLoc : symbol->references)
    references.push_back(getLocationFromLoc(sourceMgr, refLoc, uri));
}

//===--------------------------------------------------------------------===//
// PDLDocument: Document Links
//===--------------------------------------------------------------------===//
```
- **EN**: Implements logic around `push_back`, `findReferencesOf`, `getAsSMLoc`, `lookup`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `push_back`、`findReferencesOf`、`getAsSMLoc`、`lookup` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 457-478
```cpp

void PDLDocument::getDocumentLinks(
    const llvm::lsp::URIForFile &uri,
    std::vector<llvm::lsp::DocumentLink> &links) {
  for (const lsp::SourceMgrInclude &include : parsedIncludes)
    links.emplace_back(include.range, include.uri);
}

//===----------------------------------------------------------------------===//
// PDLDocument: Hover
//===----------------------------------------------------------------------===//

std::optional<llvm::lsp::Hover>
PDLDocument::findHover(const llvm::lsp::URIForFile &uri,
                       const llvm::lsp::Position &hoverPos) {
  SMLoc posLoc = hoverPos.getAsSMLoc(sourceMgr);

  // Check for a reference to an include.
  for (const lsp::SourceMgrInclude &include : parsedIncludes)
    if (include.range.contains(hoverPos))
      return include.buildHover();

```
- **EN**: Implements logic around `getDocumentLinks`, `emplace_back`, `findHover`, `getAsSMLoc`, and 2 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `getDocumentLinks`、`emplace_back`、`findHover`、`getAsSMLoc` 等另外 2 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 479-498
```cpp
  // Find the symbol at the given location.
  SMRange hoverRange;
  const PDLIndexSymbol *symbol = index.lookup(posLoc, &hoverRange);
  if (!symbol)
    return std::nullopt;

  // Add hover for operation names.
  if (const auto *op =
          llvm::dyn_cast_if_present<const ods::Operation *>(symbol->definition))
    return buildHoverForOpName(op, hoverRange);
  const auto *decl = cast<const ast::Decl *>(symbol->definition);
  return findHover(decl, hoverRange);
}

std::optional<llvm::lsp::Hover>
PDLDocument::findHover(const ast::Decl *decl, const SMRange &hoverRange) {
  // Add hover for variables.
  if (const auto *varDecl = dyn_cast<ast::VariableDecl>(decl))
    return buildHoverForVariable(varDecl, hoverRange);

```
- **EN**: Implements logic around `lookup`, `buildHoverForOpName`, `findHover`, `VariableDecl>`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `lookup`、`buildHoverForOpName`、`findHover`、`VariableDecl>` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 499-517
```cpp
  // Add hover for patterns.
  if (const auto *patternDecl = dyn_cast<ast::PatternDecl>(decl))
    return buildHoverForPattern(patternDecl, hoverRange);

  // Add hover for core constraints.
  if (const auto *cst = dyn_cast<ast::CoreConstraintDecl>(decl))
    return buildHoverForCoreConstraint(cst, hoverRange);

  // Add hover for user constraints.
  if (const auto *cst = dyn_cast<ast::UserConstraintDecl>(decl))
    return buildHoverForUserConstraintOrRewrite("Constraint", cst, hoverRange);

  // Add hover for user rewrites.
  if (const auto *rewrite = dyn_cast<ast::UserRewriteDecl>(decl))
    return buildHoverForUserConstraintOrRewrite("Rewrite", rewrite, hoverRange);

  return std::nullopt;
}

```
- **EN**: Implements logic around `PatternDecl>`, `buildHoverForPattern`, `CoreConstraintDecl>`, `buildHoverForCoreConstraint`, and 3 more symbols.
- **CN**: 围绕 `PatternDecl>`、`buildHoverForPattern`、`CoreConstraintDecl>`、`buildHoverForCoreConstraint` 等另外 3 个符号 实现具体逻辑。

### Lines 518-541
```cpp
llvm::lsp::Hover PDLDocument::buildHoverForOpName(const ods::Operation *op,
                                                  const SMRange &hoverRange) {
  llvm::lsp::Hover hover(llvm::lsp::Range(sourceMgr, hoverRange));
  {
    llvm::raw_string_ostream hoverOS(hover.contents.value);
    hoverOS << "**OpName**: `" << op->getName() << "`\n***\n"
            << op->getSummary() << "\n***\n"
            << op->getDescription();
  }
  return hover;
}

llvm::lsp::Hover
PDLDocument::buildHoverForVariable(const ast::VariableDecl *varDecl,
                                   const SMRange &hoverRange) {
  llvm::lsp::Hover hover(llvm::lsp::Range(sourceMgr, hoverRange));
  {
    llvm::raw_string_ostream hoverOS(hover.contents.value);
    hoverOS << "**Variable**: `" << varDecl->getName().getName() << "`\n***\n"
            << "Type: `" << varDecl->getType() << "`\n";
  }
  return hover;
}

```
- **EN**: Implements logic around `buildHoverForOpName`, `hover`, `hoverOS`, `getName`, and 4 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `buildHoverForOpName`、`hover`、`hoverOS`、`getName` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 542-564
```cpp
llvm::lsp::Hover PDLDocument::buildHoverForPattern(const ast::PatternDecl *decl,
                                                   const SMRange &hoverRange) {
  llvm::lsp::Hover hover(llvm::lsp::Range(sourceMgr, hoverRange));
  {
    llvm::raw_string_ostream hoverOS(hover.contents.value);
    hoverOS << "**Pattern**";
    if (const ast::Name *name = decl->getName())
      hoverOS << ": `" << name->getName() << "`";
    hoverOS << "\n***\n";
    if (std::optional<uint16_t> benefit = decl->getBenefit())
      hoverOS << "Benefit: " << *benefit << "\n";
    if (decl->hasBoundedRewriteRecursion())
      hoverOS << "HasBoundedRewriteRecursion\n";
    hoverOS << "RootOp: `"
            << decl->getRootRewriteStmt()->getRootOpExpr()->getType() << "`\n";

    // Format the documentation for the decl.
    if (std::optional<std::string> doc = getDocumentationFor(sourceMgr, decl))
      hoverOS << "\n" << *doc << "\n";
  }
  return hover;
}

```
- **EN**: Implements logic around `buildHoverForPattern`, `hover`, `hoverOS`, `getName`, and 4 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `buildHoverForPattern`、`hover`、`hoverOS`、`getName` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 565-591
```cpp
llvm::lsp::Hover
PDLDocument::buildHoverForCoreConstraint(const ast::CoreConstraintDecl *decl,
                                         const SMRange &hoverRange) {
  llvm::lsp::Hover hover(llvm::lsp::Range(sourceMgr, hoverRange));
  {
    llvm::raw_string_ostream hoverOS(hover.contents.value);
    hoverOS << "**Constraint**: `";
    TypeSwitch<const ast::Decl *>(decl)
        .Case([&](const ast::AttrConstraintDecl *) { hoverOS << "Attr"; })
        .Case([&](const ast::OpConstraintDecl *opCst) {
          hoverOS << "Op";
          if (std::optional<StringRef> name = opCst->getName())
            hoverOS << "<" << *name << ">";
        })
        .Case([&](const ast::TypeConstraintDecl *) { hoverOS << "Type"; })
        .Case([&](const ast::TypeRangeConstraintDecl *) {
          hoverOS << "TypeRange";
        })
        .Case([&](const ast::ValueConstraintDecl *) { hoverOS << "Value"; })
        .Case([&](const ast::ValueRangeConstraintDecl *) {
          hoverOS << "ValueRange";
        });
    hoverOS << "`\n";
  }
  return hover;
}

```
- **EN**: Implements logic around `buildHoverForCoreConstraint`, `hover`, `hoverOS`, `Case`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `buildHoverForCoreConstraint`、`hover`、`hoverOS`、`Case` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 592-624
```cpp
template <typename T>
llvm::lsp::Hover PDLDocument::buildHoverForUserConstraintOrRewrite(
    StringRef typeName, const T *decl, const SMRange &hoverRange) {
  llvm::lsp::Hover hover(llvm::lsp::Range(sourceMgr, hoverRange));
  {
    llvm::raw_string_ostream hoverOS(hover.contents.value);
    hoverOS << "**" << typeName << "**: `" << decl->getName().getName()
            << "`\n***\n";
    ArrayRef<ast::VariableDecl *> inputs = decl->getInputs();
    if (!inputs.empty()) {
      hoverOS << "Parameters:\n";
      for (const ast::VariableDecl *input : inputs)
        hoverOS << "* " << input->getName().getName() << ": `"
                << input->getType() << "`\n";
      hoverOS << "***\n";
    }
    ast::Type resultType = decl->getResultType();
    if (auto resultTupleTy = dyn_cast<ast::TupleType>(resultType)) {
      if (!resultTupleTy.empty()) {
        hoverOS << "Results:\n";
        for (auto it : llvm::zip(resultTupleTy.getElementNames(),
                                 resultTupleTy.getElementTypes())) {
          StringRef name = std::get<0>(it);
          hoverOS << "* " << (name.empty() ? "" : (name + ": ")) << "`"
                  << std::get<1>(it) << "`\n";
        }
        hoverOS << "***\n";
      }
    } else {
      hoverOS << "Results:\n* `" << resultType << "`\n";
      hoverOS << "***\n";
    }

```
- **EN**: Implements logic around `buildHoverForUserConstraintOrRewrite`, `hover`, `hoverOS`, `getName`, and 8 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `buildHoverForUserConstraintOrRewrite`、`hover`、`hoverOS`、`getName` 等另外 8 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 625-644
```cpp
    // Format the documentation for the decl.
    if (std::optional<std::string> doc = getDocumentationFor(sourceMgr, decl))
      hoverOS << "\n" << *doc << "\n";
  }
  return hover;
}

//===----------------------------------------------------------------------===//
// PDLDocument: Document Symbols
//===----------------------------------------------------------------------===//

void PDLDocument::findDocumentSymbols(
    std::vector<llvm::lsp::DocumentSymbol> &symbols) {
  if (failed(astModule))
    return;

  for (const ast::Decl *decl : (*astModule)->getChildren()) {
    if (!isMainFileLoc(sourceMgr, decl->getLoc()))
      continue;

```
- **EN**: Implements logic around `getDocumentationFor`, `findDocumentSymbols`, `failed`, `getChildren`, and 1 more symbols.
- **CN**: 围绕 `getDocumentationFor`、`findDocumentSymbols`、`failed`、`getChildren` 等另外 1 个符号 实现具体逻辑。

### Lines 645-668
```cpp
    if (const auto *patternDecl = dyn_cast<ast::PatternDecl>(decl)) {
      const ast::Name *name = patternDecl->getName();

      SMRange nameLoc = name ? name->getLoc() : patternDecl->getLoc();
      SMRange bodyLoc(nameLoc.Start, patternDecl->getBody()->getLoc().End);

      symbols.emplace_back(name ? name->getName() : "<pattern>",
                           llvm::lsp::SymbolKind::Class,
                           llvm::lsp::Range(sourceMgr, bodyLoc),
                           llvm::lsp::Range(sourceMgr, nameLoc));
    } else if (const auto *cDecl = dyn_cast<ast::UserConstraintDecl>(decl)) {
      // TODO: Add source information for the code block body.
      SMRange nameLoc = cDecl->getName().getLoc();
      SMRange bodyLoc = nameLoc;

      symbols.emplace_back(cDecl->getName().getName(),
                           llvm::lsp::SymbolKind::Function,
                           llvm::lsp::Range(sourceMgr, bodyLoc),
                           llvm::lsp::Range(sourceMgr, nameLoc));
    } else if (const auto *cDecl = dyn_cast<ast::UserRewriteDecl>(decl)) {
      // TODO: Add source information for the code block body.
      SMRange nameLoc = cDecl->getName().getLoc();
      SMRange bodyLoc = nameLoc;

```
- **EN**: Implements logic around `PatternDecl>`, `getName`, `getLoc`, `bodyLoc`, and 4 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `PatternDecl>`、`getName`、`getLoc`、`bodyLoc` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 669-691
```cpp
      symbols.emplace_back(cDecl->getName().getName(),
                           llvm::lsp::SymbolKind::Function,
                           llvm::lsp::Range(sourceMgr, bodyLoc),
                           llvm::lsp::Range(sourceMgr, nameLoc));
    }
  }
}

//===----------------------------------------------------------------------===//
// PDLDocument: Code Completion
//===----------------------------------------------------------------------===//

namespace {
class LSPCodeCompleteContext : public CodeCompleteContext {
public:
  LSPCodeCompleteContext(SMLoc completeLoc, llvm::SourceMgr &sourceMgr,
                         llvm::lsp::CompletionList &completionList,
                         ods::Context &odsContext,
                         ArrayRef<std::string> includeDirs)
      : CodeCompleteContext(completeLoc), sourceMgr(sourceMgr),
        completionList(completionList), odsContext(odsContext),
        includeDirs(includeDirs) {}

```
- **EN**: Introduces declarations for `LSPCodeCompleteContext`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `LSPCodeCompleteContext` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 692-716
```cpp
  void codeCompleteTupleMemberAccess(ast::TupleType tupleType) final {
    ArrayRef<ast::Type> elementTypes = tupleType.getElementTypes();
    ArrayRef<StringRef> elementNames = tupleType.getElementNames();
    for (unsigned i = 0, e = tupleType.size(); i < e; ++i) {
      // Push back a completion item that uses the result index.
      llvm::lsp::CompletionItem item;
      item.label = llvm::formatv("{0} (field #{0})", i).str();
      item.insertText = Twine(i).str();
      item.filterText = item.sortText = item.insertText;
      item.kind = llvm::lsp::CompletionItemKind::Field;
      item.detail = llvm::formatv("{0}: {1}", i, elementTypes[i]);
      item.insertTextFormat = llvm::lsp::InsertTextFormat::PlainText;
      completionList.items.emplace_back(item);

      // If the element has a name, push back a completion item with that name.
      if (!elementNames[i].empty()) {
        item.label =
            llvm::formatv("{1} (field #{0})", i, elementNames[i]).str();
        item.filterText = item.label;
        item.insertText = elementNames[i].str();
        completionList.items.emplace_back(item);
      }
    }
  }

```
- **EN**: Implements logic around `codeCompleteTupleMemberAccess`, `getElementTypes`, `getElementNames`, `size`, and 5 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `codeCompleteTupleMemberAccess`、`getElementTypes`、`getElementNames`、`size` 等另外 5 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 717-751
```cpp
  void codeCompleteOperationMemberAccess(ast::OperationType opType) final {
    const ods::Operation *odsOp = opType.getODSOperation();
    if (!odsOp)
      return;

    ArrayRef<ods::OperandOrResult> results = odsOp->getResults();
    for (const auto &it : llvm::enumerate(results)) {
      const ods::OperandOrResult &result = it.value();
      const ods::TypeConstraint &constraint = result.getConstraint();

      // Push back a completion item that uses the result index.
      llvm::lsp::CompletionItem item;
      item.label = llvm::formatv("{0} (field #{0})", it.index()).str();
      item.insertText = Twine(it.index()).str();
      item.filterText = item.sortText = item.insertText;
      item.kind = llvm::lsp::CompletionItemKind::Field;
      switch (result.getVariableLengthKind()) {
      case ods::VariableLengthKind::Single:
        item.detail = llvm::formatv("{0}: Value", it.index()).str();
        break;
      case ods::VariableLengthKind::Optional:
        item.detail = llvm::formatv("{0}: Value?", it.index()).str();
        break;
      case ods::VariableLengthKind::Variadic:
        item.detail = llvm::formatv("{0}: ValueRange", it.index()).str();
        break;
      }
      item.documentation = llvm::lsp::MarkupContent{
          llvm::lsp::MarkupKind::Markdown,
          llvm::formatv("{0}\n\n```c++\n{1}\n```\n", constraint.getSummary(),
                        constraint.getCppClass())
              .str()};
      item.insertTextFormat = llvm::lsp::InsertTextFormat::PlainText;
      completionList.items.emplace_back(item);

```
- **EN**: Implements logic around `codeCompleteOperationMemberAccess`, `getODSOperation`, `getResults`, `enumerate`, and 8 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `codeCompleteOperationMemberAccess`、`getODSOperation`、`getResults`、`enumerate` 等另外 8 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 752-769
```cpp
      // If the result has a name, push back a completion item with the result
      // name.
      if (!result.getName().empty()) {
        item.label =
            llvm::formatv("{1} (field #{0})", it.index(), result.getName())
                .str();
        item.filterText = item.label;
        item.insertText = result.getName().str();
        completionList.items.emplace_back(item);
      }
    }
  }

  void codeCompleteOperationAttributeName(StringRef opName) final {
    const ods::Operation *odsOp = odsContext.lookupOperation(opName);
    if (!odsOp)
      return;

```
- **EN**: Implements logic around `getName`, `formatv`, `str`, `emplace_back`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getName`、`formatv`、`str`、`emplace_back` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 770-805
```cpp
    for (const ods::Attribute &attr : odsOp->getAttributes()) {
      const ods::AttributeConstraint &constraint = attr.getConstraint();

      llvm::lsp::CompletionItem item;
      item.label = attr.getName().str();
      item.kind = llvm::lsp::CompletionItemKind::Field;
      item.detail = attr.isOptional() ? "optional" : "";
      item.documentation = llvm::lsp::MarkupContent{
          llvm::lsp::MarkupKind::Markdown,
          llvm::formatv("{0}\n\n```c++\n{1}\n```\n", constraint.getSummary(),
                        constraint.getCppClass())
              .str()};
      item.insertTextFormat = llvm::lsp::InsertTextFormat::PlainText;
      completionList.items.emplace_back(item);
    }
  }

  void codeCompleteConstraintName(ast::Type currentType,
                                  bool allowInlineTypeConstraints,
                                  const ast::DeclScope *scope) final {
    auto addCoreConstraint = [&](StringRef constraint, StringRef mlirType,
                                 StringRef snippetText = "") {
      llvm::lsp::CompletionItem item;
      item.label = constraint.str();
      item.kind = llvm::lsp::CompletionItemKind::Class;
      item.detail = (constraint + " constraint").str();
      item.documentation = llvm::lsp::MarkupContent{
          llvm::lsp::MarkupKind::Markdown,
          ("A single entity core constraint of type `" + mlirType + "`").str()};
      item.sortText = "0";
      item.insertText = snippetText.str();
      item.insertTextFormat = snippetText.empty()
                                  ? llvm::lsp::InsertTextFormat::PlainText
                                  : llvm::lsp::InsertTextFormat::Snippet;
      completionList.items.emplace_back(item);
    };
```
- **EN**: Implements logic around `getAttributes`, `getConstraint`, `getName`, `isOptional`, and 6 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getAttributes`、`getConstraint`、`getName`、`isOptional` 等另外 6 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 806-825
```cpp

    // Insert completions for the core constraints. Some core constraints have
    // additional characteristics, so we may add then even if a type has been
    // inferred.
    if (!currentType) {
      addCoreConstraint("Attr", "mlir::Attribute");
      addCoreConstraint("Op", "mlir::Operation *");
      addCoreConstraint("Value", "mlir::Value");
      addCoreConstraint("ValueRange", "mlir::ValueRange");
      addCoreConstraint("Type", "mlir::Type");
      addCoreConstraint("TypeRange", "mlir::TypeRange");
    }
    if (allowInlineTypeConstraints) {
      /// Attr<Type>.
      if (!currentType || isa<ast::AttributeType>(currentType))
        addCoreConstraint("Attr<type>", "mlir::Attribute", "Attr<$1>");
      /// Value<Type>.
      if (!currentType || isa<ast::ValueType>(currentType))
        addCoreConstraint("Value<type>", "mlir::Value", "Value<$1>");
      /// ValueRange<TypeRange>.
```
- **EN**: Implements logic around `addCoreConstraint`, `AttributeType>`, `ValueType>`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `addCoreConstraint`、`AttributeType>`、`ValueType>` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 826-844
```cpp
      if (!currentType || isa<ast::ValueRangeType>(currentType))
        addCoreConstraint("ValueRange<type>", "mlir::ValueRange",
                          "ValueRange<$1>");
    }

    // If a scope was provided, check it for potential constraints.
    while (scope) {
      for (const ast::Decl *decl : scope->getDecls()) {
        if (const auto *cst = dyn_cast<ast::UserConstraintDecl>(decl)) {
          llvm::lsp::CompletionItem item;
          item.label = cst->getName().getName().str();
          item.kind = llvm::lsp::CompletionItemKind::Interface;
          item.sortText = "2_" + item.label;

          // Skip constraints that are not single-arg. We currently only
          // complete variable constraints.
          if (cst->getInputs().size() != 1)
            continue;

```
- **EN**: Implements logic around `ValueRangeType>`, `addCoreConstraint`, `getDecls`, `UserConstraintDecl>`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `ValueRangeType>`、`addCoreConstraint`、`getDecls`、`UserConstraintDecl>` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 845-867
```cpp
          // Ensure the input type matched the given type.
          ast::Type constraintType = cst->getInputs()[0]->getType();
          if (currentType && !currentType.refineWith(constraintType))
            continue;

          // Format the constraint signature.
          {
            llvm::raw_string_ostream strOS(item.detail);
            strOS << "(";
            llvm::interleaveComma(
                cst->getInputs(), strOS, [&](const ast::VariableDecl *var) {
                  strOS << var->getName().getName() << ": " << var->getType();
                });
            strOS << ") -> " << cst->getResultType();
          }

          // Format the documentation for the constraint.
          if (std::optional<std::string> doc =
                  getDocumentationFor(sourceMgr, cst)) {
            item.documentation = llvm::lsp::MarkupContent{
                llvm::lsp::MarkupKind::Markdown, std::move(*doc)};
          }

```
- **EN**: Implements logic around `getInputs`, `refineWith`, `strOS`, `interleaveComma`, and 4 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getInputs`、`refineWith`、`strOS`、`interleaveComma` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 868-886
```cpp
          completionList.items.emplace_back(item);
        }
      }

      scope = scope->getParentScope();
    }
  }

  void codeCompleteDialectName() final {
    // Code complete known dialects.
    for (const ods::Dialect &dialect : odsContext.getDialects()) {
      llvm::lsp::CompletionItem item;
      item.label = dialect.getName().str();
      item.kind = llvm::lsp::CompletionItemKind::Class;
      item.insertTextFormat = llvm::lsp::InsertTextFormat::PlainText;
      completionList.items.emplace_back(item);
    }
  }

```
- **EN**: Implements logic around `emplace_back`, `getParentScope`, `codeCompleteDialectName`, `getDialects`, and 1 more symbols.
- **CN**: 围绕 `emplace_back`、`getParentScope`、`codeCompleteDialectName`、`getDialects` 等另外 1 个符号 实现具体逻辑。

### Lines 887-918
```cpp
  void codeCompleteOperationName(StringRef dialectName) final {
    const ods::Dialect *dialect = odsContext.lookupDialect(dialectName);
    if (!dialect)
      return;

    for (const auto &it : dialect->getOperations()) {
      const ods::Operation &op = *it.second;

      llvm::lsp::CompletionItem item;
      item.label = op.getName().drop_front(dialectName.size() + 1).str();
      item.kind = llvm::lsp::CompletionItemKind::Field;
      item.insertTextFormat = llvm::lsp::InsertTextFormat::PlainText;
      completionList.items.emplace_back(item);
    }
  }

  void codeCompletePatternMetadata() final {
    auto addSimpleConstraint = [&](StringRef constraint, StringRef desc,
                                   StringRef snippetText = "") {
      llvm::lsp::CompletionItem item;
      item.label = constraint.str();
      item.kind = llvm::lsp::CompletionItemKind::Class;
      item.detail = "pattern metadata";
      item.documentation =
          llvm::lsp::MarkupContent{llvm::lsp::MarkupKind::Markdown, desc.str()};
      item.insertText = snippetText.str();
      item.insertTextFormat = snippetText.empty()
                                  ? llvm::lsp::InsertTextFormat::PlainText
                                  : llvm::lsp::InsertTextFormat::Snippet;
      completionList.items.emplace_back(item);
    };

```
- **EN**: Implements logic around `codeCompleteOperationName`, `lookupDialect`, `getOperations`, `getName`, and 4 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `codeCompleteOperationName`、`lookupDialect`、`getOperations`、`getName` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 919-943
```cpp
    addSimpleConstraint("benefit", "The `benefit` of matching the pattern.",
                        "benefit($1)");
    addSimpleConstraint("recursion",
                        "The pattern properly handles recursive application.");
  }

  void codeCompleteIncludeFilename(StringRef curPath) final {
    // Normalize the path to allow for interacting with the file system
    // utilities.
    SmallString<128> nativeRelDir(llvm::sys::path::convert_to_slash(curPath));
    llvm::sys::path::native(nativeRelDir);

    // Set of already included completion paths.
    StringSet<> seenResults;

    // Functor used to add a single include completion item.
    auto addIncludeCompletion = [&](StringRef path, bool isDirectory) {
      llvm::lsp::CompletionItem item;
      item.label = path.str();
      item.kind = isDirectory ? llvm::lsp::CompletionItemKind::Folder
                              : llvm::lsp::CompletionItemKind::File;
      if (seenResults.insert(item.label).second)
        completionList.items.emplace_back(item);
    };

```
- **EN**: Implements logic around `addSimpleConstraint`, `benefit`, `codeCompleteIncludeFilename`, `nativeRelDir`, and 4 more symbols.
- **CN**: 围绕 `addSimpleConstraint`、`benefit`、`codeCompleteIncludeFilename`、`nativeRelDir` 等另外 4 个符号 实现具体逻辑。

### Lines 944-965
```cpp
    // Process the include directories for this file, adding any potential
    // nested include files or directories.
    for (StringRef includeDir : includeDirs) {
      llvm::SmallString<128> dir = includeDir;
      if (!nativeRelDir.empty())
        llvm::sys::path::append(dir, nativeRelDir);

      std::error_code errorCode;
      for (auto it = llvm::sys::fs::directory_iterator(dir, errorCode),
                e = llvm::sys::fs::directory_iterator();
           !errorCode && it != e; it.increment(errorCode)) {
        StringRef filename = llvm::sys::path::filename(it->path());

        // To know whether a symlink should be treated as file or a directory,
        // we have to stat it. This should be cheap enough as there shouldn't be
        // many symlinks.
        llvm::sys::fs::file_type fileType = it->type();
        if (fileType == llvm::sys::fs::file_type::symlink_file) {
          if (auto fileStatus = it->status())
            fileType = fileStatus->type();
        }

```
- **EN**: Implements logic around `empty`, `append`, `directory_iterator`, `increment`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `empty`、`append`、`directory_iterator`、`increment` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 966-989
```cpp
        switch (fileType) {
        case llvm::sys::fs::file_type::directory_file:
          addIncludeCompletion(filename, /*isDirectory=*/true);
          break;
        case llvm::sys::fs::file_type::regular_file: {
          // Only consider concrete files that can actually be included by PDLL.
          if (filename.ends_with(".pdll") || filename.ends_with(".td"))
            addIncludeCompletion(filename, /*isDirectory=*/false);
          break;
        }
        default:
          break;
        }
      }
    }

    // Sort the completion results to make sure the output is deterministic in
    // the face of different iteration schemes for different platforms.
    llvm::sort(completionList.items, [](const llvm::lsp::CompletionItem &lhs,
                                        const llvm::lsp::CompletionItem &rhs) {
      return lhs.label < rhs.label;
    });
  }

```
- **EN**: Implements logic around `addIncludeCompletion`, `ends_with`, `sort`.
- **CN**: 围绕 `addIncludeCompletion`、`ends_with`、`sort` 实现具体逻辑。

### Lines 990-1012
```cpp
private:
  llvm::SourceMgr &sourceMgr;
  llvm::lsp::CompletionList &completionList;
  ods::Context &odsContext;
  ArrayRef<std::string> includeDirs;
};
} // namespace

llvm::lsp::CompletionList
PDLDocument::getCodeCompletion(const llvm::lsp::URIForFile &uri,
                               const llvm::lsp::Position &completePos) {
  SMLoc posLoc = completePos.getAsSMLoc(sourceMgr);
  if (!posLoc.isValid())
    return llvm::lsp::CompletionList();

  // To perform code completion, we run another parse of the module with the
  // code completion context provided.
  ods::Context tmpODSContext;
  llvm::lsp::CompletionList completionList;
  LSPCodeCompleteContext lspCompleteContext(posLoc, sourceMgr, completionList,
                                            tmpODSContext,
                                            sourceMgr.getIncludeDirs());

```
- **EN**: Implements logic around `getCodeCompletion`, `getAsSMLoc`, `isValid`, `CompletionList`, and 2 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `getCodeCompletion`、`getAsSMLoc`、`isValid`、`CompletionList` 等另外 2 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 1013-1032
```cpp
  ast::Context tmpContext(tmpODSContext);
  (void)parsePDLLAST(tmpContext, sourceMgr, /*enableDocumentation=*/true,
                     &lspCompleteContext);

  return completionList;
}

//===----------------------------------------------------------------------===//
// PDLDocument: Signature Help
//===----------------------------------------------------------------------===//

namespace {
class LSPSignatureHelpContext : public CodeCompleteContext {
public:
  LSPSignatureHelpContext(SMLoc completeLoc, llvm::SourceMgr &sourceMgr,
                          llvm::lsp::SignatureHelp &signatureHelp,
                          ods::Context &odsContext)
      : CodeCompleteContext(completeLoc), sourceMgr(sourceMgr),
        signatureHelp(signatureHelp), odsContext(odsContext) {}

```
- **EN**: Introduces declarations for `LSPSignatureHelpContext`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `LSPSignatureHelpContext` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 1033-1052
```cpp
  void codeCompleteCallSignature(const ast::CallableDecl *callable,
                                 unsigned currentNumArgs) final {
    signatureHelp.activeParameter = currentNumArgs;

    llvm::lsp::SignatureInformation signatureInfo;
    {
      llvm::raw_string_ostream strOS(signatureInfo.label);
      strOS << callable->getName()->getName() << "(";
      auto formatParamFn = [&](const ast::VariableDecl *var) {
        unsigned paramStart = strOS.str().size();
        strOS << var->getName().getName() << ": " << var->getType();
        unsigned paramEnd = strOS.str().size();
        signatureInfo.parameters.emplace_back(llvm::lsp::ParameterInformation{
            StringRef(strOS.str()).slice(paramStart, paramEnd).str(),
            std::make_pair(paramStart, paramEnd), /*paramDoc*/ std::string()});
      };
      llvm::interleaveComma(callable->getInputs(), strOS, formatParamFn);
      strOS << ") -> " << callable->getResultType();
    }

```
- **EN**: Implements logic around `codeCompleteCallSignature`, `strOS`, `getName`, `str`, and 5 more symbols.
- **CN**: 围绕 `codeCompleteCallSignature`、`strOS`、`getName`、`str` 等另外 5 个符号 实现具体逻辑。

### Lines 1053-1071
```cpp
    // Format the documentation for the callable.
    if (std::optional<std::string> doc =
            getDocumentationFor(sourceMgr, callable))
      signatureInfo.documentation = std::move(*doc);

    signatureHelp.signatures.emplace_back(std::move(signatureInfo));
  }

  void
  codeCompleteOperationOperandsSignature(std::optional<StringRef> opName,
                                         unsigned currentNumOperands) final {
    const ods::Operation *odsOp =
        opName ? odsContext.lookupOperation(*opName) : nullptr;
    codeCompleteOperationOperandOrResultSignature(
        opName, odsOp,
        odsOp ? odsOp->getOperands() : ArrayRef<ods::OperandOrResult>(),
        currentNumOperands, "operand", "Value");
  }

```
- **EN**: Implements logic around `getDocumentationFor`, `move`, `emplace_back`, `codeCompleteOperationOperandsSignature`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getDocumentationFor`、`move`、`emplace_back`、`codeCompleteOperationOperandsSignature` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 1072-1094
```cpp
  void codeCompleteOperationResultsSignature(std::optional<StringRef> opName,
                                             unsigned currentNumResults) final {
    const ods::Operation *odsOp =
        opName ? odsContext.lookupOperation(*opName) : nullptr;
    codeCompleteOperationOperandOrResultSignature(
        opName, odsOp,
        odsOp ? odsOp->getResults() : ArrayRef<ods::OperandOrResult>(),
        currentNumResults, "result", "Type");
  }

  void codeCompleteOperationOperandOrResultSignature(
      std::optional<StringRef> opName, const ods::Operation *odsOp,
      ArrayRef<ods::OperandOrResult> values, unsigned currentValue,
      StringRef label, StringRef dataType) {
    signatureHelp.activeParameter = currentValue;

    // If we have ODS information for the operation, add in the ODS signature
    // for the operation. We also verify that the current number of values is
    // not more than what is defined in ODS, as this will result in an error
    // anyways.
    if (odsOp && currentValue < values.size()) {
      llvm::lsp::SignatureInformation signatureInfo;

```
- **EN**: Implements logic around `codeCompleteOperationResultsSignature`, `lookupOperation`, `codeCompleteOperationOperandOrResultSignature`, `getResults`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `codeCompleteOperationResultsSignature`、`lookupOperation`、`codeCompleteOperationOperandOrResultSignature`、`getResults` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 1095-1120
```cpp
      // Build the signature label.
      {
        llvm::raw_string_ostream strOS(signatureInfo.label);
        strOS << "(";
        auto formatFn = [&](const ods::OperandOrResult &value) {
          unsigned paramStart = strOS.str().size();

          strOS << value.getName() << ": ";

          StringRef constraintDoc = value.getConstraint().getSummary();
          std::string paramDoc;
          switch (value.getVariableLengthKind()) {
          case ods::VariableLengthKind::Single:
            strOS << dataType;
            paramDoc = constraintDoc.str();
            break;
          case ods::VariableLengthKind::Optional:
            strOS << dataType << "?";
            paramDoc = ("optional: " + constraintDoc).str();
            break;
          case ods::VariableLengthKind::Variadic:
            strOS << dataType << "Range";
            paramDoc = ("variadic: " + constraintDoc).str();
            break;
          }

```
- **EN**: Implements logic around `strOS`, `str`, `getName`, `getConstraint`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `strOS`、`str`、`getName`、`getConstraint` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 1121-1149
```cpp
          unsigned paramEnd = strOS.str().size();
          signatureInfo.parameters.emplace_back(llvm::lsp::ParameterInformation{
              StringRef(strOS.str()).slice(paramStart, paramEnd).str(),
              std::make_pair(paramStart, paramEnd), paramDoc});
        };
        llvm::interleaveComma(values, strOS, formatFn);
        strOS << ")";
      }
      signatureInfo.documentation =
          llvm::formatv("`op<{0}>` ODS {1} specification", *opName, label)
              .str();
      signatureHelp.signatures.emplace_back(std::move(signatureInfo));
    }

    // If there aren't any arguments yet, we also add the generic signature.
    if (currentValue == 0 && (!odsOp || !values.empty())) {
      llvm::lsp::SignatureInformation signatureInfo;
      signatureInfo.label =
          llvm::formatv("(<{0}s>: {1}Range)", label, dataType).str();
      signatureInfo.documentation =
          ("Generic operation " + label + " specification").str();
      signatureInfo.parameters.emplace_back(llvm::lsp::ParameterInformation{
          StringRef(signatureInfo.label).drop_front().drop_back().str(),
          std::pair<unsigned, unsigned>(1, signatureInfo.label.size() - 1),
          ("All of the " + label + "s of the operation.").str()});
      signatureHelp.signatures.emplace_back(std::move(signatureInfo));
    }
  }

```
- **EN**: Implements logic around `str`, `emplace_back`, `StringRef`, `make_pair`, and 4 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `str`、`emplace_back`、`StringRef`、`make_pair` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 1150-1170
```cpp
private:
  llvm::SourceMgr &sourceMgr;
  llvm::lsp::SignatureHelp &signatureHelp;
  ods::Context &odsContext;
};
} // namespace

llvm::lsp::SignatureHelp
PDLDocument::getSignatureHelp(const llvm::lsp::URIForFile &uri,
                              const llvm::lsp::Position &helpPos) {
  SMLoc posLoc = helpPos.getAsSMLoc(sourceMgr);
  if (!posLoc.isValid())
    return llvm::lsp::SignatureHelp();

  // To perform code completion, we run another parse of the module with the
  // code completion context provided.
  ods::Context tmpODSContext;
  llvm::lsp::SignatureHelp signatureHelp;
  LSPSignatureHelpContext completeContext(posLoc, sourceMgr, signatureHelp,
                                          tmpODSContext);

```
- **EN**: Implements logic around `getSignatureHelp`, `getAsSMLoc`, `isValid`, `SignatureHelp`, and 1 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `getSignatureHelp`、`getAsSMLoc`、`isValid`、`SignatureHelp` 等另外 1 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 1171-1193
```cpp
  ast::Context tmpContext(tmpODSContext);
  (void)parsePDLLAST(tmpContext, sourceMgr, /*enableDocumentation=*/true,
                     &completeContext);

  return signatureHelp;
}

//===----------------------------------------------------------------------===//
// PDLDocument: Inlay Hints
//===----------------------------------------------------------------------===//

/// Returns true if the given name should be added as a hint for `expr`.
static bool shouldAddHintFor(const ast::Expr *expr, StringRef name) {
  if (name.empty())
    return false;

  // If the argument is a reference of the same name, don't add it as a hint.
  if (auto *ref = dyn_cast<ast::DeclRefExpr>(expr)) {
    const ast::Name *declName = ref->getDecl()->getName();
    if (declName && declName->getName() == name)
      return false;
  }

```
- **EN**: Implements logic around `tmpContext`, `parsePDLLAST`, `shouldAddHintFor`, `empty`, and 3 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `tmpContext`、`parsePDLLAST`、`shouldAddHintFor`、`empty` 等另外 3 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 1194-1212
```cpp
  return true;
}

void PDLDocument::getInlayHints(const llvm::lsp::URIForFile &uri,
                                const llvm::lsp::Range &range,
                                std::vector<llvm::lsp::InlayHint> &inlayHints) {
  if (failed(astModule))
    return;
  SMRange rangeLoc = range.getAsSMRange(sourceMgr);
  if (!rangeLoc.isValid())
    return;
  (*astModule)->walk([&](const ast::Node *node) {
    SMRange loc = node->getLoc();

    // Check that the location of this node is within the input range.
    if (!lsp::contains(rangeLoc, loc.Start) &&
        !lsp::contains(rangeLoc, loc.End))
      return;

```
- **EN**: Implements logic around `getInlayHints`, `failed`, `getAsSMRange`, `isValid`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getInlayHints`、`failed`、`getAsSMRange`、`isValid` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 1213-1238
```cpp
    // Handle hints for various types of nodes.
    llvm::TypeSwitch<const ast::Node *>(node)
        .Case<ast::VariableDecl, ast::CallExpr, ast::OperationExpr>(
            [&](const auto *node) {
              this->getInlayHintsFor(node, uri, inlayHints);
            });
  });
}

void PDLDocument::getInlayHintsFor(
    const ast::VariableDecl *decl, const llvm::lsp::URIForFile &uri,
    std::vector<llvm::lsp::InlayHint> &inlayHints) {
  // Check to see if the variable has a constraint list, if it does we don't
  // provide initializer hints.
  if (!decl->getConstraints().empty())
    return;

  // Check to see if the variable has an initializer.
  if (const ast::Expr *expr = decl->getInitExpr()) {
    // Don't add hints for operation expression initialized variables given that
    // the type of the variable is easily inferred by the expression operation
    // name.
    if (isa<ast::OperationExpr>(expr))
      return;
  }

```
- **EN**: Implements logic around `OperationExpr>`, `getInlayHintsFor`, `getConstraints`, `getInitExpr`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `OperationExpr>`、`getInlayHintsFor`、`getConstraints`、`getInitExpr` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 1239-1259
```cpp
  llvm::lsp::InlayHint hint(llvm::lsp::InlayHintKind::Type,
                            llvm::lsp::Position(sourceMgr, decl->getLoc().End));
  {
    llvm::raw_string_ostream labelOS(hint.label);
    labelOS << ": " << decl->getType();
  }

  inlayHints.emplace_back(std::move(hint));
}

void PDLDocument::getInlayHintsFor(
    const ast::CallExpr *expr, const llvm::lsp::URIForFile &uri,
    std::vector<llvm::lsp::InlayHint> &inlayHints) {
  // Try to extract the callable of this call.
  const auto *callableRef = dyn_cast<ast::DeclRefExpr>(expr->getCallableExpr());
  const auto *callable =
      callableRef ? dyn_cast<ast::CallableDecl>(callableRef->getDecl())
                  : nullptr;
  if (!callable)
    return;

```
- **EN**: Implements logic around `hint`, `Position`, `labelOS`, `getType`, and 4 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `hint`、`Position`、`labelOS`、`getType` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 1260-1280
```cpp
  // Add hints for the arguments to the call.
  for (const auto &it : llvm::zip(expr->getArguments(), callable->getInputs()))
    addParameterHintFor(inlayHints, std::get<0>(it),
                        std::get<1>(it)->getName().getName());
}

void PDLDocument::getInlayHintsFor(
    const ast::OperationExpr *expr, const llvm::lsp::URIForFile &uri,
    std::vector<llvm::lsp::InlayHint> &inlayHints) {
  // Check for ODS information.
  ast::OperationType opType = dyn_cast<ast::OperationType>(expr->getType());
  const auto *odsOp = opType ? opType.getODSOperation() : nullptr;

  auto addOpHint = [&](const ast::Expr *valueExpr, StringRef label) {
    // If the value expression used the same location as the operation, don't
    // add a hint. This expression was materialized during parsing.
    if (expr->getLoc().Start == valueExpr->getLoc().Start)
      return;
    addParameterHintFor(inlayHints, valueExpr, label);
  };

```
- **EN**: Implements logic around `zip`, `addParameterHintFor`, `get`, `getInlayHintsFor`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `zip`、`addParameterHintFor`、`get`、`getInlayHintsFor` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 1281-1298
```cpp
  // Functor used to process hints for the operands and results of the
  // operation. They effectively have the same format, and thus can be processed
  // using the same logic.
  auto addOperandOrResultHints = [&](ArrayRef<ast::Expr *> values,
                                     ArrayRef<ods::OperandOrResult> odsValues,
                                     StringRef allValuesName) {
    if (values.empty())
      return;

    // The values should either map to a single range, or be equivalent to the
    // ODS values.
    if (values.size() != odsValues.size()) {
      // Handle the case of a single element that covers the full range.
      if (values.size() == 1)
        return addOpHint(values.front(), allValuesName);
      return;
    }

```
- **EN**: Implements logic around `empty`, `size`, `addOpHint`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `empty`、`size`、`addOpHint` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 1299-1319
```cpp
    for (const auto &it : llvm::zip(values, odsValues))
      addOpHint(std::get<0>(it), std::get<1>(it).getName());
  };

  // Add hints for the operands and results of the operation.
  addOperandOrResultHints(expr->getOperands(),
                          odsOp ? odsOp->getOperands()
                                : ArrayRef<ods::OperandOrResult>(),
                          "operands");
  addOperandOrResultHints(expr->getResultTypes(),
                          odsOp ? odsOp->getResults()
                                : ArrayRef<ods::OperandOrResult>(),
                          "results");
}

void PDLDocument::addParameterHintFor(
    std::vector<llvm::lsp::InlayHint> &inlayHints, const ast::Expr *expr,
    StringRef label) {
  if (!shouldAddHintFor(expr, label))
    return;

```
- **EN**: Implements logic around `zip`, `addOpHint`, `addOperandOrResultHints`, `getOperands`, and 4 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `zip`、`addOpHint`、`addOperandOrResultHints`、`getOperands` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 1320-1340
```cpp
  llvm::lsp::InlayHint hint(
      llvm::lsp::InlayHintKind::Parameter,
      llvm::lsp::Position(sourceMgr, expr->getLoc().Start));
  hint.label = (label + ":").str();
  hint.paddingRight = true;
  inlayHints.emplace_back(std::move(hint));
}

//===----------------------------------------------------------------------===//
// PDLL ViewOutput
//===----------------------------------------------------------------------===//

void PDLDocument::getPDLLViewOutput(raw_ostream &os,
                                    lsp::PDLLViewOutputKind kind) {
  if (failed(astModule))
    return;
  if (kind == lsp::PDLLViewOutputKind::AST) {
    (*astModule)->print(os);
    return;
  }

```
- **EN**: Implements logic around `hint`, `Position`, `str`, `emplace_back`, and 3 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `hint`、`Position`、`str`、`emplace_back` 等另外 3 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 1341-1360
```cpp
  // Generate the MLIR for the ast module. We also capture diagnostics here to
  // show to the user, which may be useful if PDLL isn't capturing constraints
  // expected by PDL.
  MLIRContext mlirContext;
  SourceMgrDiagnosticHandler diagHandler(sourceMgr, &mlirContext, os);
  OwningOpRef<ModuleOp> pdlModule =
      codegenPDLLToMLIR(&mlirContext, astContext, sourceMgr, **astModule);
  if (!pdlModule)
    return;
  if (kind == lsp::PDLLViewOutputKind::MLIR) {
    pdlModule->print(os, OpPrintingFlags().enableDebugInfo());
    return;
  }

  // Otherwise, generate the output for C++.
  assert(kind == lsp::PDLLViewOutputKind::CPP &&
         "unexpected PDLLViewOutputKind");
  codegenPDLLToCPP(**astModule, *pdlModule, os);
}

```
- **EN**: Implements logic around `diagHandler`, `codegenPDLLToMLIR`, `print`, `assert`, and 1 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `diagHandler`、`codegenPDLLToMLIR`、`print`、`assert` 等另外 1 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 1361-1381
```cpp
//===----------------------------------------------------------------------===//
// PDLTextFileChunk
//===----------------------------------------------------------------------===//

namespace {
/// This class represents a single chunk of an PDL text file.
struct PDLTextFileChunk {
  PDLTextFileChunk(uint64_t lineOffset, const llvm::lsp::URIForFile &uri,
                   StringRef contents,
                   const std::vector<std::string> &extraDirs,
                   std::vector<llvm::lsp::Diagnostic> &diagnostics)
      : lineOffset(lineOffset),
        document(uri, contents, extraDirs, diagnostics) {}

  /// Adjust the line number of the given range to anchor at the beginning of
  /// the file, instead of the beginning of this chunk.
  void adjustLocForChunkOffset(llvm::lsp::Range &range) {
    adjustLocForChunkOffset(range.start);
    adjustLocForChunkOffset(range.end);
  }
  /// Adjust the line number of the given position to anchor at the beginning of
```
- **EN**: Introduces declarations for `PDLTextFileChunk`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `PDLTextFileChunk` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 1382-1399
```cpp
  /// the file, instead of the beginning of this chunk.
  void adjustLocForChunkOffset(llvm::lsp::Position &pos) {
    pos.line += lineOffset;
  }

  /// The line offset of this chunk from the beginning of the file.
  uint64_t lineOffset;
  /// The document referred to by this chunk.
  PDLDocument document;
};
} // namespace

//===----------------------------------------------------------------------===//
// PDLTextFile
//===----------------------------------------------------------------------===//

namespace {
/// This class represents a text file containing one or more PDL documents.
```
- **EN**: Implements logic around `adjustLocForChunkOffset`.
- **CN**: 围绕 `adjustLocForChunkOffset` 实现具体逻辑。

### Lines 1400-1418
```cpp
class PDLTextFile {
public:
  PDLTextFile(const llvm::lsp::URIForFile &uri, StringRef fileContents,
              int64_t version, const std::vector<std::string> &extraDirs,
              std::vector<llvm::lsp::Diagnostic> &diagnostics);

  /// Return the current version of this text file.
  int64_t getVersion() const { return version; }

  /// Update the file to the new version using the provided set of content
  /// changes. Returns failure if the update was unsuccessful.
  LogicalResult
  update(const llvm::lsp::URIForFile &uri, int64_t newVersion,
         ArrayRef<llvm::lsp::TextDocumentContentChangeEvent> changes,
         std::vector<llvm::lsp::Diagnostic> &diagnostics);

  //===--------------------------------------------------------------------===//
  // LSP Queries
  //===--------------------------------------------------------------------===//
```
- **EN**: Introduces declarations for `PDLTextFile`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `PDLTextFile` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 1419-1438
```cpp

  void getLocationsOf(const llvm::lsp::URIForFile &uri,
                      llvm::lsp::Position defPos,
                      std::vector<llvm::lsp::Location> &locations);
  void findReferencesOf(const llvm::lsp::URIForFile &uri,
                        llvm::lsp::Position pos,
                        std::vector<llvm::lsp::Location> &references);
  void getDocumentLinks(const llvm::lsp::URIForFile &uri,
                        std::vector<llvm::lsp::DocumentLink> &links);
  std::optional<llvm::lsp::Hover> findHover(const llvm::lsp::URIForFile &uri,
                                            llvm::lsp::Position hoverPos);
  void findDocumentSymbols(std::vector<llvm::lsp::DocumentSymbol> &symbols);
  llvm::lsp::CompletionList getCodeCompletion(const llvm::lsp::URIForFile &uri,
                                              llvm::lsp::Position completePos);
  llvm::lsp::SignatureHelp getSignatureHelp(const llvm::lsp::URIForFile &uri,
                                            llvm::lsp::Position helpPos);
  void getInlayHints(const llvm::lsp::URIForFile &uri, llvm::lsp::Range range,
                     std::vector<llvm::lsp::InlayHint> &inlayHints);
  lsp::PDLLViewOutputResult getPDLLViewOutput(lsp::PDLLViewOutputKind kind);

```
- **EN**: Implements logic around `getLocationsOf`, `findReferencesOf`, `getDocumentLinks`, `findHover`, and 5 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getLocationsOf`、`findReferencesOf`、`getDocumentLinks`、`findHover` 等另外 5 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 1439-1457
```cpp
private:
  using ChunkIterator = llvm::pointee_iterator<
      std::vector<std::unique_ptr<PDLTextFileChunk>>::iterator>;

  /// Initialize the text file from the given file contents.
  void initialize(const llvm::lsp::URIForFile &uri, int64_t newVersion,
                  std::vector<llvm::lsp::Diagnostic> &diagnostics);

  /// Find the PDL document that contains the given position, and update the
  /// position to be anchored at the start of the found chunk instead of the
  /// beginning of the file.
  ChunkIterator getChunkItFor(llvm::lsp::Position &pos);
  PDLTextFileChunk &getChunkFor(llvm::lsp::Position &pos) {
    return *getChunkItFor(pos);
  }

  /// The full string contents of the file.
  std::string contents;

```
- **EN**: Implements logic around `initialize`, `getChunkItFor`, `getChunkFor`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `initialize`、`getChunkItFor`、`getChunkFor` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 1458-1480
```cpp
  /// The version of this file.
  int64_t version = 0;

  /// The number of lines in the file.
  int64_t totalNumLines = 0;

  /// The chunks of this file. The order of these chunks is the order in which
  /// they appear in the text file.
  std::vector<std::unique_ptr<PDLTextFileChunk>> chunks;

  /// The extra set of include directories for this file.
  std::vector<std::string> extraIncludeDirs;
};
} // namespace

PDLTextFile::PDLTextFile(const llvm::lsp::URIForFile &uri,
                         StringRef fileContents, int64_t version,
                         const std::vector<std::string> &extraDirs,
                         std::vector<llvm::lsp::Diagnostic> &diagnostics)
    : contents(fileContents.str()), extraIncludeDirs(extraDirs) {
  initialize(uri, version, diagnostics);
}

```
- **EN**: Implements logic around `PDLTextFile`, `contents`, `initialize`; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `PDLTextFile`、`contents`、`initialize` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 1481-1501
```cpp
LogicalResult
PDLTextFile::update(const llvm::lsp::URIForFile &uri, int64_t newVersion,
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

void PDLTextFile::getLocationsOf(const llvm::lsp::URIForFile &uri,
                                 llvm::lsp::Position defPos,
                                 std::vector<llvm::lsp::Location> &locations) {
  PDLTextFileChunk &chunk = getChunkFor(defPos);
  chunk.document.getLocationsOf(uri, defPos, locations);

```
- **EN**: Implements logic around `update`, `failed`, `error`, `failure`, and 4 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects.
- **CN**: 围绕 `update`、`failed`、`error`、`failure` 等另外 4 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象。

### Lines 1502-1523
```cpp
  // Adjust any locations within this file for the offset of this chunk.
  if (chunk.lineOffset == 0)
    return;
  for (llvm::lsp::Location &loc : locations)
    if (loc.uri == uri)
      chunk.adjustLocForChunkOffset(loc.range);
}

void PDLTextFile::findReferencesOf(
    const llvm::lsp::URIForFile &uri, llvm::lsp::Position pos,
    std::vector<llvm::lsp::Location> &references) {
  PDLTextFileChunk &chunk = getChunkFor(pos);
  chunk.document.findReferencesOf(uri, pos, references);

  // Adjust any locations within this file for the offset of this chunk.
  if (chunk.lineOffset == 0)
    return;
  for (llvm::lsp::Location &loc : references)
    if (loc.uri == uri)
      chunk.adjustLocForChunkOffset(loc.range);
}

```
- **EN**: Implements logic around `adjustLocForChunkOffset`, `findReferencesOf`, `getChunkFor`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `adjustLocForChunkOffset`、`findReferencesOf`、`getChunkFor` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 1524-1545
```cpp
void PDLTextFile::getDocumentLinks(
    const llvm::lsp::URIForFile &uri,
    std::vector<llvm::lsp::DocumentLink> &links) {
  chunks.front()->document.getDocumentLinks(uri, links);
  for (const auto &it : llvm::drop_begin(chunks)) {
    size_t currentNumLinks = links.size();
    it->document.getDocumentLinks(uri, links);

    // Adjust any links within this file to account for the offset of this
    // chunk.
    for (auto &link : llvm::drop_begin(links, currentNumLinks))
      it->adjustLocForChunkOffset(link.range);
  }
}

std::optional<llvm::lsp::Hover>
PDLTextFile::findHover(const llvm::lsp::URIForFile &uri,
                       llvm::lsp::Position hoverPos) {
  PDLTextFileChunk &chunk = getChunkFor(hoverPos);
  std::optional<llvm::lsp::Hover> hoverInfo =
      chunk.document.findHover(uri, hoverPos);

```
- **EN**: Implements logic around `getDocumentLinks`, `front`, `drop_begin`, `size`, and 3 more symbols.
- **CN**: 围绕 `getDocumentLinks`、`front`、`drop_begin`、`size` 等另外 3 个符号 实现具体逻辑。

### Lines 1546-1569
```cpp
  // Adjust any locations within this file for the offset of this chunk.
  if (chunk.lineOffset != 0 && hoverInfo && hoverInfo->range)
    chunk.adjustLocForChunkOffset(*hoverInfo->range);
  return hoverInfo;
}

void PDLTextFile::findDocumentSymbols(
    std::vector<llvm::lsp::DocumentSymbol> &symbols) {
  if (chunks.size() == 1)
    return chunks.front()->document.findDocumentSymbols(symbols);

  // If there are multiple chunks in this file, we create top-level symbols for
  // each chunk.
  for (unsigned i = 0, e = chunks.size(); i < e; ++i) {
    PDLTextFileChunk &chunk = *chunks[i];
    llvm::lsp::Position startPos(chunk.lineOffset);
    llvm::lsp::Position endPos((i == e - 1) ? totalNumLines - 1
                                            : chunks[i + 1]->lineOffset);
    llvm::lsp::DocumentSymbol symbol(
        "<file-split-" + Twine(i) + ">", llvm::lsp::SymbolKind::Namespace,
        /*range=*/llvm::lsp::Range(startPos, endPos),
        /*selectionRange=*/llvm::lsp::Range(startPos));
    chunk.document.findDocumentSymbols(symbol.children);

```
- **EN**: Implements logic around `adjustLocForChunkOffset`, `findDocumentSymbols`, `size`, `front`, and 5 more symbols.
- **CN**: 围绕 `adjustLocForChunkOffset`、`findDocumentSymbols`、`size`、`front` 等另外 5 个符号 实现具体逻辑。

### Lines 1570-1590
```cpp
    // Fixup the locations of document symbols within this chunk.
    if (i != 0) {
      SmallVector<llvm::lsp::DocumentSymbol *> symbolsToFix;
      for (llvm::lsp::DocumentSymbol &childSymbol : symbol.children)
        symbolsToFix.push_back(&childSymbol);

      while (!symbolsToFix.empty()) {
        llvm::lsp::DocumentSymbol *symbol = symbolsToFix.pop_back_val();
        chunk.adjustLocForChunkOffset(symbol->range);
        chunk.adjustLocForChunkOffset(symbol->selectionRange);

        for (llvm::lsp::DocumentSymbol &childSymbol : symbol->children)
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

### Lines 1591-1613
```cpp
llvm::lsp::CompletionList
PDLTextFile::getCodeCompletion(const llvm::lsp::URIForFile &uri,
                               llvm::lsp::Position completePos) {
  PDLTextFileChunk &chunk = getChunkFor(completePos);
  llvm::lsp::CompletionList completionList =
      chunk.document.getCodeCompletion(uri, completePos);

  // Adjust any completion locations.
  for (llvm::lsp::CompletionItem &item : completionList.items) {
    if (item.textEdit)
      chunk.adjustLocForChunkOffset(item.textEdit->range);
    for (llvm::lsp::TextEdit &edit : item.additionalTextEdits)
      chunk.adjustLocForChunkOffset(edit.range);
  }
  return completionList;
}

llvm::lsp::SignatureHelp
PDLTextFile::getSignatureHelp(const llvm::lsp::URIForFile &uri,
                              llvm::lsp::Position helpPos) {
  return getChunkFor(helpPos).document.getSignatureHelp(uri, helpPos);
}

```
- **EN**: Implements logic around `getCodeCompletion`, `getChunkFor`, `adjustLocForChunkOffset`, `getSignatureHelp`.
- **CN**: 围绕 `getCodeCompletion`、`getChunkFor`、`adjustLocForChunkOffset`、`getSignatureHelp` 实现具体逻辑。

### Lines 1614-1636
```cpp
void PDLTextFile::getInlayHints(const llvm::lsp::URIForFile &uri,
                                llvm::lsp::Range range,
                                std::vector<llvm::lsp::InlayHint> &inlayHints) {
  auto startIt = getChunkItFor(range.start);
  auto endIt = getChunkItFor(range.end);

  // Functor used to get the chunks for a given file, and fixup any locations
  auto getHintsForChunk = [&](ChunkIterator chunkIt, llvm::lsp::Range range) {
    size_t currentNumHints = inlayHints.size();
    chunkIt->document.getInlayHints(uri, range, inlayHints);

    // If this isn't the first chunk, update any positions to account for line
    // number differences.
    if (&*chunkIt != &*chunks.front()) {
      for (auto &hint : llvm::drop_begin(inlayHints, currentNumHints))
        chunkIt->adjustLocForChunkOffset(hint.position);
    }
  };
  // Returns the number of lines held by a given chunk.
  auto getNumLines = [](ChunkIterator chunkIt) {
    return (chunkIt + 1)->lineOffset - chunkIt->lineOffset;
  };

```
- **EN**: Implements logic around `getInlayHints`, `getChunkItFor`, `size`, `front`, and 2 more symbols.
- **CN**: 围绕 `getInlayHints`、`getChunkItFor`、`size`、`front` 等另外 2 个符号 实现具体逻辑。

### Lines 1637-1654
```cpp
  // Check if the range is fully within a single chunk.
  if (startIt == endIt)
    return getHintsForChunk(startIt, range);

  // Otherwise, the range is split between multiple chunks. The first chunk
  // has the correct range start, but covers the total document.
  getHintsForChunk(startIt,
                   llvm::lsp::Range(range.start, getNumLines(startIt)));

  // Every chunk in between uses the full document.
  for (++startIt; startIt != endIt; ++startIt)
    getHintsForChunk(startIt, llvm::lsp::Range(0, getNumLines(startIt)));

  // The range for the last chunk starts at the beginning of the document, up
  // through the end of the input range.
  getHintsForChunk(startIt, llvm::lsp::Range(0, range.end));
}

```
- **EN**: Implements logic around `getHintsForChunk`, `Range`.
- **CN**: 围绕 `getHintsForChunk`、`Range` 实现具体逻辑。

### Lines 1655-1676
```cpp
lsp::PDLLViewOutputResult
PDLTextFile::getPDLLViewOutput(lsp::PDLLViewOutputKind kind) {
  lsp::PDLLViewOutputResult result;
  {
    llvm::raw_string_ostream outputOS(result.output);
    llvm::interleave(
        llvm::make_pointee_range(chunks),
        [&](PDLTextFileChunk &chunk) {
          chunk.document.getPDLLViewOutput(outputOS, kind);
        },
        [&] { outputOS << "\n"
                       << kDefaultSplitMarker << "\n\n"; });
  }
  return result;
}

void PDLTextFile::initialize(const llvm::lsp::URIForFile &uri,
                             int64_t newVersion,
                             std::vector<llvm::lsp::Diagnostic> &diagnostics) {
  version = newVersion;
  chunks.clear();

```
- **EN**: Implements logic around `getPDLLViewOutput`, `outputOS`, `interleave`, `make_pointee_range`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `getPDLLViewOutput`、`outputOS`、`interleave`、`make_pointee_range` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 1677-1696
```cpp
  // Split the file into separate PDL documents.
  SmallVector<StringRef, 8> subContents;
  StringRef(contents).split(subContents, kDefaultSplitMarker);
  chunks.emplace_back(std::make_unique<PDLTextFileChunk>(
      /*lineOffset=*/0, uri, subContents.front(), extraIncludeDirs,
      diagnostics));

  uint64_t lineOffset = subContents.front().count('\n');
  for (StringRef docContents : llvm::drop_begin(subContents)) {
    unsigned currentNumDiags = diagnostics.size();
    auto chunk = std::make_unique<PDLTextFileChunk>(
        lineOffset, uri, docContents, extraIncludeDirs, diagnostics);
    lineOffset += docContents.count('\n');

    // Adjust locations used in diagnostics to account for the offset from the
    // beginning of the file.
    for (llvm::lsp::Diagnostic &diag :
         llvm::drop_begin(diagnostics, currentNumDiags)) {
      chunk->adjustLocForChunkOffset(diag.range);

```
- **EN**: Implements logic around `StringRef`, `emplace_back`, `front`, `drop_begin`, and 4 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `StringRef`、`emplace_back`、`front`、`drop_begin` 等另外 4 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 1697-1723
```cpp
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

PDLTextFile::ChunkIterator
PDLTextFile::getChunkItFor(llvm::lsp::Position &pos) {
  if (chunks.size() == 1)
    return chunks.begin();

  // Search for the first chunk with a greater line offset, the previous chunk
  // is the one that contains `pos`.
  auto it = llvm::upper_bound(
      chunks, pos, [](const llvm::lsp::Position &pos, const auto &chunk) {
        return static_cast<uint64_t>(pos.line) < chunk->lineOffset;
      });
  ChunkIterator chunkIt(it == chunks.end() ? (chunks.end() - 1) : --it);
  pos.line -= chunkIt->lineOffset;
  return chunkIt;
}

```
- **EN**: Implements logic around `adjustLocForChunkOffset`, `emplace_back`, `getChunkItFor`, `size`, and 4 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `adjustLocForChunkOffset`、`emplace_back`、`getChunkItFor`、`size` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 1724-1742
```cpp
//===----------------------------------------------------------------------===//
// PDLLServer::Impl
//===----------------------------------------------------------------------===//

struct lsp::PDLLServer::Impl {
  explicit Impl(const Options &options)
      : options(options), compilationDatabase(options.compilationDatabases) {}

  /// PDLL LSP options.
  const Options &options;

  /// The compilation database containing additional information for files
  /// passed to the server.
  lsp::CompilationDatabase compilationDatabase;

  /// The files held by the server, mapped by their URI file name.
  llvm::StringMap<std::unique_ptr<PDLTextFile>> files;
};

```
- **EN**: Introduces declarations for `lsp::PDLLServer::Impl`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `lsp::PDLLServer::Impl` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 1743-1762
```cpp
//===----------------------------------------------------------------------===//
// PDLLServer
//===----------------------------------------------------------------------===//

lsp::PDLLServer::PDLLServer(const Options &options)
    : impl(std::make_unique<Impl>(options)) {}
lsp::PDLLServer::~PDLLServer() = default;

void lsp::PDLLServer::addDocument(
    const URIForFile &uri, StringRef contents, int64_t version,
    std::vector<llvm::lsp::Diagnostic> &diagnostics) {
  // Build the set of additional include directories.
  std::vector<std::string> additionalIncludeDirs = impl->options.extraDirs;
  const auto &fileInfo = impl->compilationDatabase.getFileInfo(uri.file());
  llvm::append_range(additionalIncludeDirs, fileInfo.includeDirs);

  impl->files[uri.file()] = std::make_unique<PDLTextFile>(
      uri, contents, version, additionalIncludeDirs, diagnostics);
}

```
- **EN**: Implements logic around `PDLLServer`, `impl`, `~PDLLServer`, `addDocument`, and 3 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `PDLLServer`、`impl`、`~PDLLServer`、`addDocument` 等另外 3 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 1763-1781
```cpp
void lsp::PDLLServer::updateDocument(
    const URIForFile &uri, ArrayRef<TextDocumentContentChangeEvent> changes,
    int64_t version, std::vector<llvm::lsp::Diagnostic> &diagnostics) {
  // Check that we actually have a document for this uri.
  auto it = impl->files.find(uri.file());
  if (it == impl->files.end())
    return;

  // Try to update the document. If we fail, erase the file from the server. A
  // failed updated generally means we've fallen out of sync somewhere.
  if (failed(it->second->update(uri, version, changes, diagnostics)))
    impl->files.erase(it);
}

std::optional<int64_t> lsp::PDLLServer::removeDocument(const URIForFile &uri) {
  auto it = impl->files.find(uri.file());
  if (it == impl->files.end())
    return std::nullopt;

```
- **EN**: Implements logic around `updateDocument`, `find`, `end`, `failed`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `updateDocument`、`find`、`end`、`failed` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 1782-1802
```cpp
  int64_t version = it->second->getVersion();
  impl->files.erase(it);
  return version;
}

void lsp::PDLLServer::getLocationsOf(
    const URIForFile &uri, const Position &defPos,
    std::vector<llvm::lsp::Location> &locations) {
  auto fileIt = impl->files.find(uri.file());
  if (fileIt != impl->files.end())
    fileIt->second->getLocationsOf(uri, defPos, locations);
}

void lsp::PDLLServer::findReferencesOf(
    const URIForFile &uri, const Position &pos,
    std::vector<llvm::lsp::Location> &references) {
  auto fileIt = impl->files.find(uri.file());
  if (fileIt != impl->files.end())
    fileIt->second->findReferencesOf(uri, pos, references);
}

```
- **EN**: Implements logic around `getVersion`, `erase`, `getLocationsOf`, `find`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getVersion`、`erase`、`getLocationsOf`、`find` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 1803-1824
```cpp
void lsp::PDLLServer::getDocumentLinks(
    const URIForFile &uri, std::vector<DocumentLink> &documentLinks) {
  auto fileIt = impl->files.find(uri.file());
  if (fileIt != impl->files.end())
    return fileIt->second->getDocumentLinks(uri, documentLinks);
}

std::optional<llvm::lsp::Hover>
lsp::PDLLServer::findHover(const URIForFile &uri, const Position &hoverPos) {
  auto fileIt = impl->files.find(uri.file());
  if (fileIt != impl->files.end())
    return fileIt->second->findHover(uri, hoverPos);
  return std::nullopt;
}

void lsp::PDLLServer::findDocumentSymbols(
    const URIForFile &uri, std::vector<DocumentSymbol> &symbols) {
  auto fileIt = impl->files.find(uri.file());
  if (fileIt != impl->files.end())
    fileIt->second->findDocumentSymbols(symbols);
}

```
- **EN**: Implements logic around `getDocumentLinks`, `find`, `end`, `findHover`, and 1 more symbols.
- **CN**: 围绕 `getDocumentLinks`、`find`、`end`、`findHover` 等另外 1 个符号 实现具体逻辑。

### Lines 1825-1842
```cpp
lsp::CompletionList
lsp::PDLLServer::getCodeCompletion(const URIForFile &uri,
                                   const Position &completePos) {
  auto fileIt = impl->files.find(uri.file());
  if (fileIt != impl->files.end())
    return fileIt->second->getCodeCompletion(uri, completePos);
  return CompletionList();
}

llvm::lsp::SignatureHelp
lsp::PDLLServer::getSignatureHelp(const URIForFile &uri,
                                  const Position &helpPos) {
  auto fileIt = impl->files.find(uri.file());
  if (fileIt != impl->files.end())
    return fileIt->second->getSignatureHelp(uri, helpPos);
  return SignatureHelp();
}

```
- **EN**: Implements logic around `getCodeCompletion`, `find`, `end`, `CompletionList`, and 2 more symbols.
- **CN**: 围绕 `getCodeCompletion`、`find`、`end`、`CompletionList` 等另外 2 个符号 实现具体逻辑。

### Lines 1843-1862
```cpp
void lsp::PDLLServer::getInlayHints(const URIForFile &uri, const Range &range,
                                    std::vector<InlayHint> &inlayHints) {
  auto fileIt = impl->files.find(uri.file());
  if (fileIt == impl->files.end())
    return;
  fileIt->second->getInlayHints(uri, range, inlayHints);

  // Drop any duplicated hints that may have cropped up.
  llvm::sort(inlayHints);
  inlayHints.erase(llvm::unique(inlayHints), inlayHints.end());
}

std::optional<lsp::PDLLViewOutputResult>
lsp::PDLLServer::getPDLLViewOutput(const URIForFile &uri,
                                   PDLLViewOutputKind kind) {
  auto fileIt = impl->files.find(uri.file());
  if (fileIt != impl->files.end())
    return fileIt->second->getPDLLViewOutput(kind);
  return std::nullopt;
}
```
- **EN**: Implements logic around `getInlayHints`, `find`, `end`, `sort`, and 2 more symbols.
- **CN**: 围绕 `getInlayHints`、`find`、`end`、`sort` 等另外 2 个符号 实现具体逻辑。

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
- **Streaming output / 流式输出**:
  - **EN**: Text, diagnostics, or generated content are emitted through LLVM stream APIs.
  - **CN**: 通过 LLVM 流 API 输出文本、诊断或生成内容。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `PDLLServer.h`, `Protocol.h`, `mlir/IR/BuiltinOps.h`, `mlir/Support/ToolUtilities.h`, `mlir/Tools/PDLL/AST/Context.h`, `mlir/Tools/PDLL/AST/Nodes.h`, `mlir/Tools/PDLL/AST/Types.h`, `mlir/Tools/PDLL/CodeGen/CPPGen.h`, `mlir/Tools/PDLL/CodeGen/MLIRGen.h`, `mlir/Tools/PDLL/ODS/Constraint.h` ... (+15 more)
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: tooling support declarations / 工具支持声明 (13), LLVM ADT containers and utilities / LLVM ADT 容器与工具 (4), LLVM support-library helpers / LLVM Support 库辅助工具 (4), core MLIR IR abstractions / 核心 MLIR IR 抽象 (1), shared MLIR support helpers / 共享的 MLIR 支持工具 (1)
