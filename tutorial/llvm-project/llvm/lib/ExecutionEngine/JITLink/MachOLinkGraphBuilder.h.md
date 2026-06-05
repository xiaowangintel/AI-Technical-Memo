# MachOLinkGraphBuilder.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/JITLink/MachOLinkGraphBuilder.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements MachO LinkGraph builder.
  - **CN**: 实现面向即时编译代码的 JITLink 图链接、重定位处理以及特定文件格式的链接图 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===----- MachOLinkGraphBuilder.h - MachO LinkGraph builder ----*- C++ -*-===//
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
// Generic MachO LinkGraph building code.
//
//===----------------------------------------------------------------------===//

#ifndef LIB_EXECUTIONENGINE_JITLINK_MACHOLINKGRAPHBUILDER_H
#define LIB_EXECUTIONENGINE_JITLINK_MACHOLINKGRAPHBUILDER_H

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 16-23
```cpp
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/StringMap.h"
#include "llvm/ExecutionEngine/JITLink/JITLink.h"
#include "llvm/Object/MachO.h"

#include "EHFrameSupportImpl.h"
#include "JITLinkGeneric.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ADT/DenseMap.h`, `llvm/ADT/StringMap.h`, `llvm/ExecutionEngine/JITLink/JITLink.h`, `llvm/Object/MachO.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ADT/DenseMap.h`, `llvm/ADT/StringMap.h`, `llvm/ExecutionEngine/JITLink/JITLink.h`, `llvm/Object/MachO.h`。

### Lines 24-31
```cpp
namespace llvm {
namespace jitlink {

class MachOLinkGraphBuilder {
public:
  virtual ~MachOLinkGraphBuilder();
  Expected<std::unique_ptr<LinkGraph>> buildGraph();

```
- **EN**: Introduces declarations for `llvm`, `jitlink`, `MachOLinkGraphBuilder`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `jitlink`, `MachOLinkGraphBuilder` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 32-45
```cpp
protected:

  struct NormalizedSymbol {
    friend class MachOLinkGraphBuilder;

  private:
    NormalizedSymbol(std::optional<StringRef> Name, uint64_t Value,
                     uint8_t Type, uint8_t Sect, uint16_t Desc, Linkage L,
                     Scope S)
        : Name(Name), Value(Value), Type(Type), Sect(Sect), Desc(Desc), L(L),
          S(S) {
      assert((!Name || !Name->empty()) && "Name must be none or non-empty");
    }

```
- **EN**: Introduces declarations for `NormalizedSymbol`, `MachOLinkGraphBuilder`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `NormalizedSymbol`, `MachOLinkGraphBuilder` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 46-59
```cpp
  public:
    NormalizedSymbol(const NormalizedSymbol &) = delete;
    NormalizedSymbol &operator=(const NormalizedSymbol &) = delete;
    NormalizedSymbol(NormalizedSymbol &&) = delete;
    NormalizedSymbol &operator=(NormalizedSymbol &&) = delete;

    std::optional<StringRef> Name;
    uint64_t Value = 0;
    uint8_t Type = 0;
    uint8_t Sect = 0;
    uint16_t Desc = 0;
    Linkage L = Linkage::Strong;
    Scope S = Scope::Default;
    Symbol *GraphSymbol = nullptr;
```
- **EN**: Implements logic around `NormalizedSymbol`.
- **CN**: 围绕 `NormalizedSymbol` 实现具体逻辑。

### Lines 60-66
```cpp
  };

  // Normalized section representation. Section and segment names are guaranteed
  // to be null-terminated, hence the extra bytes on SegName and SectName.
  class NormalizedSection {
    friend class MachOLinkGraphBuilder;

```
- **EN**: Introduces declarations for `NormalizedSection`, `MachOLinkGraphBuilder`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `NormalizedSection`, `MachOLinkGraphBuilder` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 67-80
```cpp
  private:
    NormalizedSection() = default;

  public:
    char SectName[17];
    char SegName[17];
    orc::ExecutorAddr Address;
    uint64_t Size = 0;
    uint64_t Alignment = 0;
    uint32_t Flags = 0;
    const char *Data = nullptr;
    Section *GraphSection = nullptr;
    std::map<orc::ExecutorAddr, Symbol *> CanonicalSymbols;
  };
```
- **EN**: Implements logic around `NormalizedSection`.
- **CN**: 围绕 `NormalizedSection` 实现具体逻辑。

### Lines 81-89
```cpp

  using SectionParserFunction = std::function<Error(NormalizedSection &S)>;

  MachOLinkGraphBuilder(const object::MachOObjectFile &Obj,
                        std::shared_ptr<orc::SymbolStringPool> SSP, Triple TT,
                        SubtargetFeatures Features,
                        LinkGraph::GetEdgeKindNameFunction GetEdgeKindName);
  LinkGraph &getGraph() const { return *G; }

```
- **EN**: Implements logic around `function<Error`, `MachOLinkGraphBuilder`, `getGraph`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `function<Error`, `MachOLinkGraphBuilder`, `getGraph` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 90-96
```cpp
  const object::MachOObjectFile &getObject() const { return Obj; }

  void addCustomSectionParser(StringRef SectionName,
                              SectionParserFunction Parse);

  virtual Error addRelocations() = 0;

```
- **EN**: Implements logic around `getObject`, `addCustomSectionParser`, `addRelocations`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getObject`, `addCustomSectionParser`, `addRelocations` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 97-105
```cpp
  /// Create a symbol.
  template <typename... ArgTs>
  NormalizedSymbol &createNormalizedSymbol(ArgTs &&... Args) {
    NormalizedSymbol *Sym = reinterpret_cast<NormalizedSymbol *>(
        Allocator.Allocate<NormalizedSymbol>());
    new (Sym) NormalizedSymbol(std::forward<ArgTs>(Args)...);
    return *Sym;
  }

```
- **EN**: Implements logic around `createNormalizedSymbol`, `Allocate<NormalizedSymbol>`, `new`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `createNormalizedSymbol`, `Allocate<NormalizedSymbol>`, `new` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 106-113
```cpp
  /// Index is zero-based (MachO section indexes are usually one-based) and
  /// assumed to be in-range. Client is responsible for checking.
  NormalizedSection &getSectionByIndex(unsigned Index) {
    auto I = IndexToSection.find(Index);
    assert(I != IndexToSection.end() && "No section recorded at index");
    return I->second;
  }

```
- **EN**: Implements logic around `getSectionByIndex`, `find`, `assert`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getSectionByIndex`, `find`, `assert` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 114-124
```cpp
  /// Try to get the section at the given index. Will return an error if the
  /// given index is out of range, or if no section has been added for the given
  /// index.
  Expected<NormalizedSection &> findSectionByIndex(unsigned Index) {
    auto I = IndexToSection.find(Index);
    if (I == IndexToSection.end())
      return make_error<JITLinkError>("No section recorded for index " +
                                      formatv("{0:d}", Index));
    return I->second;
  }

```
- **EN**: Implements logic around `findSectionByIndex`, `find`, `make_error<JITLinkError>`, `formatv`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `findSectionByIndex`, `find`, `make_error<JITLinkError>`, `formatv` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 125-136
```cpp
  /// Try to get the symbol at the given index. Will return an error if the
  /// given index is out of range, or if no symbol has been added for the given
  /// index.
  Expected<NormalizedSymbol &> findSymbolByIndex(uint64_t Index) {
    auto I = IndexToSymbol.find(Index);
    if (I == IndexToSymbol.end())
      return make_error<JITLinkError>("No symbol at index " +
                                      formatv("{0:d}", Index));
    assert(I->second && "Null symbol at index");
    return *I->second;
  }

```
- **EN**: Implements logic around `findSymbolByIndex`, `find`, `make_error<JITLinkError>`, `formatv`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `findSymbolByIndex`, `find`, `make_error<JITLinkError>`, `formatv`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 137-146
```cpp
  /// Returns the symbol with the highest address not greater than the search
  /// address, or null if no such symbol exists.
  Symbol *getSymbolByAddress(NormalizedSection &NSec,
                             orc::ExecutorAddr Address) {
    auto I = NSec.CanonicalSymbols.upper_bound(Address);
    if (I == NSec.CanonicalSymbols.begin())
      return nullptr;
    return std::prev(I)->second;
  }

```
- **EN**: Implements logic around `getSymbolByAddress`, `upper_bound`, `prev`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getSymbolByAddress`, `upper_bound`, `prev` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 147-158
```cpp
  /// Returns the symbol with the highest address not greater than the search
  /// address, or an error if no such symbol exists.
  Expected<Symbol &> findSymbolByAddress(NormalizedSection &NSec,
                                         orc::ExecutorAddr Address) {
    auto *Sym = getSymbolByAddress(NSec, Address);
    if (Sym)
      if (Address <= Sym->getAddress() + Sym->getSize())
        return *Sym;
    return make_error<JITLinkError>("No symbol covering address " +
                                    formatv("{0:x16}", Address));
  }

```
- **EN**: Implements logic around `findSymbolByAddress`, `getSymbolByAddress`, `make_error<JITLinkError>`, `formatv`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `findSymbolByAddress`, `getSymbolByAddress`, `make_error<JITLinkError>`, `formatv` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 159-165
```cpp
  static Linkage getLinkage(uint16_t Desc);
  static Scope getScope(StringRef Name, uint8_t Type);
  static bool isAltEntry(const NormalizedSymbol &NSym);

  static bool isDebugSection(const NormalizedSection &NSec);
  static bool isZeroFillSection(const NormalizedSection &NSec);

```
- **EN**: Implements logic around `getLinkage`, `getScope`, `isAltEntry`, `isDebugSection`, and 1 more symbols.
- **CN**: 围绕 `getLinkage`, `getScope`, `isAltEntry`, `isDebugSection`, and 1 more symbols 实现具体逻辑。

### Lines 166-179
```cpp
  MachO::relocation_info
  getRelocationInfo(const object::relocation_iterator RelItr) {
    MachO::any_relocation_info ARI =
        getObject().getRelocation(RelItr->getRawDataRefImpl());
    MachO::relocation_info RI;
    RI.r_address = ARI.r_word0;
    RI.r_symbolnum = ARI.r_word1 & 0xffffff;
    RI.r_pcrel = (ARI.r_word1 >> 24) & 1;
    RI.r_length = (ARI.r_word1 >> 25) & 3;
    RI.r_extern = (ARI.r_word1 >> 27) & 1;
    RI.r_type = (ARI.r_word1 >> 28);
    return RI;
  }

```
- **EN**: Implements logic around `getRelocationInfo`, `getObject`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getRelocationInfo`, `getObject` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 180-193
```cpp
private:
  static unsigned getPointerSize(const object::MachOObjectFile &Obj);
  static llvm::endianness getEndianness(const object::MachOObjectFile &Obj);

  void setCanonicalSymbol(NormalizedSection &NSec, Symbol &Sym) {
    auto *&CanonicalSymEntry = NSec.CanonicalSymbols[Sym.getAddress()];
    // There should be no symbol at this address, or, if there is,
    // it should be a zero-sized symbol from an empty section (which
    // we can safely override).
    assert((!CanonicalSymEntry || CanonicalSymEntry->getSize() == 0) &&
           "Duplicate canonical symbol at address");
    CanonicalSymEntry = &Sym;
  }

```
- **EN**: Implements logic around `getPointerSize`, `getEndianness`, `setCanonicalSymbol`, `getAddress`, and 1 more symbols.
- **CN**: 围绕 `getPointerSize`, `getEndianness`, `setCanonicalSymbol`, `getAddress`, and 1 more symbols 实现具体逻辑。

### Lines 194-202
```cpp
  Section &getCommonSection();
  void addSectionStartSymAndBlock(unsigned SecIndex, Section &GraphSec,
                                  orc::ExecutorAddr Address, const char *Data,
                                  orc::ExecutorAddrDiff Size,
                                  uint32_t Alignment, bool IsLive);

  Error createNormalizedSections();
  Error createNormalizedSymbols();

```
- **EN**: Implements logic around `getCommonSection`, `addSectionStartSymAndBlock`, `createNormalizedSections`, `createNormalizedSymbols`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `getCommonSection`, `addSectionStartSymAndBlock`, `createNormalizedSections`, `createNormalizedSymbols` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 203-209
```cpp
  /// Create graph blocks and symbols for externals, absolutes, commons and
  /// all defined symbols in sections without custom parsers.
  Error graphifyRegularSymbols();

  /// Create and return a graph symbol for the given normalized symbol.
  ///
  /// NSym's GraphSymbol member will be updated to point at the newly created
```
- **EN**: Implements logic around `graphifyRegularSymbols`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `graphifyRegularSymbols` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 210-217
```cpp
  /// symbol.
  Symbol &createStandardGraphSymbol(NormalizedSymbol &Sym, Block &B,
                                    size_t Size, bool IsText,
                                    bool IsNoDeadStrip, bool IsCanonical);

  /// Create graph blocks and symbols for all sections.
  Error graphifySectionsWithCustomParsers();

```
- **EN**: Implements logic around `createStandardGraphSymbol`, `graphifySectionsWithCustomParsers`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `createStandardGraphSymbol`, `graphifySectionsWithCustomParsers` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 218-225
```cpp
  /// Graphify cstring section.
  Error graphifyCStringSection(NormalizedSection &NSec,
                               std::vector<NormalizedSymbol *> NSyms);

  // Put the BumpPtrAllocator first so that we don't free any of the underlying
  // memory until the Symbol/Addressable destructors have been run.
  BumpPtrAllocator Allocator;

```
- **EN**: Implements logic around `graphifyCStringSection`.
- **CN**: 围绕 `graphifyCStringSection` 实现具体逻辑。

### Lines 226-232
```cpp
  const object::MachOObjectFile &Obj;
  std::unique_ptr<LinkGraph> G;

  bool SubsectionsViaSymbols = false;
  DenseMap<unsigned, NormalizedSection> IndexToSection;
  Section *CommonSection = nullptr;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 233-239
```cpp
  DenseMap<uint32_t, NormalizedSymbol *> IndexToSymbol;
  StringMap<SectionParserFunction> CustomSectionParserFunctions;
};

} // end namespace jitlink
} // end namespace llvm

```
- **EN**: Introduces declarations for `jitlink`, `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `jitlink`, `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 240-240
```cpp
#endif // LIB_EXECUTIONENGINE_JITLINK_MACHOLINKGRAPHBUILDER_H
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

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

- **Direct includes / 直接包含**: `llvm/ADT/DenseMap.h`, `llvm/ADT/StringMap.h`, `llvm/ExecutionEngine/JITLink/JITLink.h`, `llvm/Object/MachO.h`, `EHFrameSupportImpl.h`, `JITLinkGeneric.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Object
