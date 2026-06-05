# COFFLinkGraphBuilder.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/JITLink/COFFLinkGraphBuilder.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements COFF LinkGraph builder.
  - **CN**: 实现面向即时编译代码的 JITLink 图链接、重定位处理以及特定文件格式的链接图 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===----- COFFLinkGraphBuilder.h - COFF LinkGraph builder ----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-15
```cpp
//
// Generic COFF LinkGraph building code.
//
//===----------------------------------------------------------------------===//

#ifndef LIB_EXECUTIONENGINE_JITLINK_COFFLINKGRAPHBUILDER_H
#define LIB_EXECUTIONENGINE_JITLINK_COFFLINKGRAPHBUILDER_H

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 16-23
```cpp
#include "llvm/ADT/DenseMap.h"
#include "llvm/ExecutionEngine/JITLink/JITLink.h"
#include "llvm/Object/COFF.h"

#include "COFFDirectiveParser.h"
#include "EHFrameSupportImpl.h"
#include "JITLinkGeneric.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ADT/DenseMap.h`, `llvm/ExecutionEngine/JITLink/JITLink.h`, `llvm/Object/COFF.h`, `COFFDirectiveParser.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ADT/DenseMap.h`, `llvm/ExecutionEngine/JITLink/JITLink.h`, `llvm/Object/COFF.h`, `COFFDirectiveParser.h`。

### Lines 24-33
```cpp
#define DEBUG_TYPE "jitlink"

namespace llvm {
namespace jitlink {

class COFFLinkGraphBuilder {
public:
  virtual ~COFFLinkGraphBuilder();
  Expected<std::unique_ptr<LinkGraph>> buildGraph();

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 34-42
```cpp
protected:
  using COFFSectionIndex = int32_t;
  using COFFSymbolIndex = int32_t;

  COFFLinkGraphBuilder(const object::COFFObjectFile &Obj,
                       std::shared_ptr<orc::SymbolStringPool> SSP, Triple TT,
                       SubtargetFeatures Features,
                       LinkGraph::GetEdgeKindNameFunction GetEdgeKindName);

```
- **EN**: Implements logic around `COFFLinkGraphBuilder`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `COFFLinkGraphBuilder` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 43-51
```cpp
  LinkGraph &getGraph() const { return *G; }

  const object::COFFObjectFile &getObject() const { return Obj; }

  virtual Error addRelocations() = 0;

  Error graphifySections();
  Error graphifySymbols();

```
- **EN**: Implements logic around `getGraph`, `getObject`, `addRelocations`, `graphifySections`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getGraph`, `getObject`, `addRelocations`, `graphifySections`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 52-59
```cpp
  void setGraphSymbol(COFFSectionIndex SecIndex, COFFSymbolIndex SymIndex,
                      Symbol &Sym) {
    assert(!GraphSymbols[SymIndex] && "Duplicate symbol at index");
    GraphSymbols[SymIndex] = &Sym;
    if (!COFF::isReservedSectionNumber(SecIndex))
      SymbolSets[SecIndex].insert({Sym.getOffset(), &Sym});
  }

```
- **EN**: Implements logic around `setGraphSymbol`, `assert`, `insert`.
- **CN**: 围绕 `setGraphSymbol`, `assert`, `insert` 实现具体逻辑。

### Lines 60-66
```cpp
  Symbol *getGraphSymbol(COFFSymbolIndex SymIndex) const {
    if (SymIndex < 0 ||
        SymIndex >= static_cast<COFFSymbolIndex>(GraphSymbols.size()))
      return nullptr;
    return GraphSymbols[SymIndex];
  }

```
- **EN**: Implements logic around `getGraphSymbol`, `static_cast<COFFSymbolIndex>`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getGraphSymbol`, `static_cast<COFFSymbolIndex>` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 67-79
```cpp
  void setGraphBlock(COFFSectionIndex SecIndex, Block *B) {
    assert(!GraphBlocks[SecIndex] && "Duplicate section at index");
    assert(!COFF::isReservedSectionNumber(SecIndex) && "Invalid section index");
    GraphBlocks[SecIndex] = B;
  }

  Block *getGraphBlock(COFFSectionIndex SecIndex) const {
    if (SecIndex <= 0 ||
        SecIndex >= static_cast<COFFSectionIndex>(GraphSymbols.size()))
      return nullptr;
    return GraphBlocks[SecIndex];
  }

```
- **EN**: Implements logic around `setGraphBlock`, `assert`, `getGraphBlock`, `static_cast<COFFSectionIndex>`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `setGraphBlock`, `assert`, `getGraphBlock`, `static_cast<COFFSectionIndex>` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 80-89
```cpp
  Symbol &addImageBaseSymbol(StringRef Name = "__ImageBase") {
    auto &ImageBase = G->addExternalSymbol(G->intern(Name), 0, true);
    ImageBase.setLive(true);
    return ImageBase;
  }

  object::COFFObjectFile::section_iterator_range sections() const {
    return Obj.sections();
  }

```
- **EN**: Implements logic around `addImageBaseSymbol`, `addExternalSymbol`, `setLive`, `sections`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `addImageBaseSymbol`, `addExternalSymbol`, `setLive`, `sections` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 90-99
```cpp
  /// Traverse all matching relocation records in the given section. The handler
  /// function Func should be callable with this signature:
  ///   Error(const object::RelocationRef&,
  ///         const object::SectionRef&, Section &)
  ///
  template <typename RelocHandlerFunction>
  Error forEachRelocation(const object::SectionRef &RelSec,
                          RelocHandlerFunction &&Func,
                          bool ProcessDebugSections = false);

```
- **EN**: Implements logic around `forEachRelocation`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `forEachRelocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 100-113
```cpp
  /// Traverse all matching relocation records in the given section. Convenience
  /// wrapper to allow passing a member function for the handler.
  ///
  template <typename ClassT, typename RelocHandlerMethod>
  Error forEachRelocation(const object::SectionRef &RelSec, ClassT *Instance,
                          RelocHandlerMethod &&Method,
                          bool ProcessDebugSections = false) {
    return forEachRelocation(
        RelSec,
        [Instance, Method](const auto &Rel, const auto &Target, auto &GS) {
          return (Instance->*Method)(Rel, Target, GS);
        },
        ProcessDebugSections);
  }
```
- **EN**: Implements logic around `forEachRelocation`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `forEachRelocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 114-124
```cpp

private:
  // Pending comdat symbol export that is initiated by the first symbol of
  // COMDAT sequence.
  struct ComdatExportRequest {
    COFFSymbolIndex SymbolIndex;
    jitlink::Linkage Linkage;
    orc::ExecutorAddrDiff Size;
  };
  std::vector<std::optional<ComdatExportRequest>> PendingComdatExports;

```
- **EN**: Introduces declarations for `ComdatExportRequest`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `ComdatExportRequest` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 125-134
```cpp
  // This represents a pending request to create a weak external symbol with a
  // name.
  struct WeakExternalRequest {
    COFFSymbolIndex Alias;
    COFFSymbolIndex Target;
    uint32_t Characteristics;
    StringRef SymbolName;
  };
  std::vector<WeakExternalRequest> WeakExternalRequests;

```
- **EN**: Introduces declarations for `WeakExternalRequest`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `WeakExternalRequest` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 135-141
```cpp
  // Per COFF section jitlink symbol set sorted by offset.
  // Used for calculating implicit size of defined symbols.
  using SymbolSet = std::set<std::pair<orc::ExecutorAddrDiff, Symbol *>>;
  std::vector<SymbolSet> SymbolSets;

  Section &getCommonSection();

```
- **EN**: Implements logic around `getCommonSection`.
- **CN**: 围绕 `getCommonSection` 实现具体逻辑。

### Lines 142-155
```cpp
  Symbol *createExternalSymbol(COFFSymbolIndex SymIndex,
                               orc::SymbolStringPtr SymbolName,
                               object::COFFSymbolRef Symbol,
                               const object::coff_section *Section);
  Expected<Symbol *> createAliasSymbol(orc::SymbolStringPtr SymbolName,
                                       Linkage L, Scope S, Symbol &Target);
  Expected<Symbol *> createDefinedSymbol(COFFSymbolIndex SymIndex,
                                         orc::SymbolStringPtr SymbolName,
                                         object::COFFSymbolRef Symbol,
                                         const object::coff_section *Section);
  Expected<Symbol *> createCOMDATExportRequest(
      COFFSymbolIndex SymIndex, object::COFFSymbolRef Symbol,
      const object::coff_aux_section_definition *Definition);
  Expected<Symbol *> exportCOMDATSymbol(COFFSymbolIndex SymIndex,
```
- **EN**: Implements logic around `createExternalSymbol`, `createAliasSymbol`, `createDefinedSymbol`, `createCOMDATExportRequest`, and 1 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `createExternalSymbol`, `createAliasSymbol`, `createDefinedSymbol`, `createCOMDATExportRequest`, and 1 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 156-163
```cpp
                                        orc::SymbolStringPtr SymbolName,
                                        object::COFFSymbolRef Symbol);

  Error handleDirectiveSection(StringRef Str);
  Error flushWeakAliasRequests();
  Error handleAlternateNames();
  Error calculateImplicitSizeOfSymbols();

```
- **EN**: Implements logic around `handleDirectiveSection`, `flushWeakAliasRequests`, `handleAlternateNames`, `calculateImplicitSizeOfSymbols`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `handleDirectiveSection`, `flushWeakAliasRequests`, `handleAlternateNames`, `calculateImplicitSizeOfSymbols` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 164-176
```cpp
  static uint64_t getSectionAddress(const object::COFFObjectFile &Obj,
                                    const object::coff_section *Section);
  static uint64_t getSectionSize(const object::COFFObjectFile &Obj,
                                 const object::coff_section *Section);
  static bool isComdatSection(const object::coff_section *Section);
  static unsigned getPointerSize(const object::COFFObjectFile &Obj);
  static llvm::endianness getEndianness(const object::COFFObjectFile &Obj);
  static StringRef getDLLImportStubPrefix() { return "__imp_"; }
  static StringRef getDirectiveSectionName() { return ".drectve"; }
  StringRef getCOFFSectionName(COFFSectionIndex SectionIndex,
                               const object::coff_section *Sec,
                               object::COFFSymbolRef Sym);

```
- **EN**: Implements logic around `getSectionAddress`, `getSectionSize`, `isComdatSection`, `getPointerSize`, and 4 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getSectionAddress`, `getSectionSize`, `isComdatSection`, `getPointerSize`, and 4 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 177-184
```cpp
  const object::COFFObjectFile &Obj;
  std::unique_ptr<LinkGraph> G;
  COFFDirectiveParser DirectiveParser;

  Section *CommonSection = nullptr;
  std::vector<Block *> GraphBlocks;
  std::vector<Symbol *> GraphSymbols;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 185-194
```cpp
  DenseMap<orc::SymbolStringPtr, orc::SymbolStringPtr> AlternateNames;
  DenseMap<orc::SymbolStringPtr, Symbol *> ExternalSymbols;
  DenseMap<orc::SymbolStringPtr, Symbol *> DefinedSymbols;
};

template <typename RelocHandlerFunction>
Error COFFLinkGraphBuilder::forEachRelocation(const object::SectionRef &RelSec,
                                              RelocHandlerFunction &&Func,
                                              bool ProcessDebugSections) {

```
- **EN**: Implements logic around `forEachRelocation`; this block coordinates ORC symbol lookup or materialization state; manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `forEachRelocation` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作。

### Lines 195-201
```cpp
  auto COFFRelSect = Obj.getCOFFSection(RelSec);

  // Target sections have names in valid COFF object files.
  Expected<StringRef> Name = Obj.getSectionName(COFFRelSect);
  if (!Name)
    return Name.takeError();

```
- **EN**: Implements logic around `getCOFFSection`, `getSectionName`, `takeError`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getCOFFSection`, `getSectionName`, `takeError` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 202-213
```cpp
  // Skip the unhandled metadata sections.
  if (*Name == ".voltbl")
    return Error::success();
  LLVM_DEBUG(dbgs() << "  " << *Name << ":\n");

  // Lookup the link-graph node corresponding to the target section name.
  auto *BlockToFix = getGraphBlock(RelSec.getIndex() + 1);
  if (!BlockToFix)
    return make_error<StringError>(
        "Referencing a section that wasn't added to the graph: " + *Name,
        inconvertibleErrorCode());

```
- **EN**: Implements logic around `success`, `getGraphBlock`, `make_error<StringError>`, `inconvertibleErrorCode`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `getGraphBlock`, `make_error<StringError>`, `inconvertibleErrorCode` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 214-222
```cpp
  // Let the callee process relocation entries one by one.
  for (const auto &R : RelSec.relocations())
    if (Error Err = Func(R, RelSec, *BlockToFix))
      return Err;

  LLVM_DEBUG(dbgs() << "\n");
  return Error::success();
}

```
- **EN**: Implements logic around `success`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `success` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 223-226
```cpp
} // end namespace jitlink
} // end namespace llvm

#endif // LIB_EXECUTIONENGINE_JITLINK_COFFLINKGRAPHBUILDER_H
```
- **EN**: Introduces declarations for `jitlink`, `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `jitlink`, `llvm` 等声明，定义本文件后续使用的数据结构或接口。

## Key Concepts / 关键概念

- **JITLink graph linking / JITLink 图链接**:
  - **EN**: Represents object code as graphs of blocks and edges so passes can rewrite relocations before final linking
  - **CN**: 把目标代码表示为块与边构成的图，使 pass 能在最终链接前重写重定位
- **Link graphs / 链接图**:
  - **EN**: Represents atoms of code/data and relocation edges explicitly for JIT-time rewriting
  - **CN**: 显式表示代码/数据原子及其重定位边，以便在 JIT 期重写
- **Relocations and fixups / 重定位与 fixup**:
  - **EN**: Bridges symbolic references to concrete addresses or relocation records
  - **CN**: 把符号引用连接到具体地址或重定位记录
- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ADT/DenseMap.h`, `llvm/ExecutionEngine/JITLink/JITLink.h`, `llvm/Object/COFF.h`, `COFFDirectiveParser.h`, `EHFrameSupportImpl.h`, `JITLinkGeneric.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Object
