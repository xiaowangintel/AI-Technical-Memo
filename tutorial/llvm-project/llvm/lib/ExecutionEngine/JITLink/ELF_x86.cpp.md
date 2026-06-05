# ELF_x86.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/JITLink/ELF_x86.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements JIT linker implementation for ELF/x86.
  - **CN**: 实现面向即时编译代码的 JITLink 图链接、重定位处理以及特定文件格式的链接图 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===--------- ELF_x86.cpp - JIT linker implementation for ELF/x86 --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-20
```cpp
//
// ELF/x86 jit-link implementation.
//
//===----------------------------------------------------------------------===//

#include "llvm/ExecutionEngine/JITLink/ELF_x86.h"
#include "DefineExternalSectionStartAndEndSymbols.h"
#include "ELFLinkGraphBuilder.h"
#include "JITLinkGeneric.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/ExecutionEngine/JITLink/x86.h"
#include "llvm/Object/ELFObjectFile.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/JITLink/ELF_x86.h`, `DefineExternalSectionStartAndEndSymbols.h`, `ELFLinkGraphBuilder.h`, `JITLinkGeneric.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/JITLink/ELF_x86.h`, `DefineExternalSectionStartAndEndSymbols.h`, `ELFLinkGraphBuilder.h`, `JITLinkGeneric.h`。

### Lines 21-28
```cpp
#define DEBUG_TYPE "jitlink"

using namespace llvm;
using namespace llvm::jitlink;

namespace {
constexpr StringRef ELFGOTSymbolName = "_GLOBAL_OFFSET_TABLE_";

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 29-38
```cpp
Error buildTables_ELF_x86(LinkGraph &G) {
  LLVM_DEBUG(dbgs() << "Visiting edges in graph:\n");

  x86::GOTTableManager GOT;
  x86::PLTTableManager PLT(GOT);
  visitExistingEdges(G, GOT, PLT);
  return Error::success();
}
} // namespace

```
- **EN**: Implements logic around `buildTables_ELF_x86`, `PLT`, `visitExistingEdges`, `success`; this block manipulates JITLink graph structures or link-time passes; executes or prepares runtime behavior for LLVM IR/JIT code; returns subsystem-specific computed results.
- **CN**: 围绕 `buildTables_ELF_x86`, `PLT`, `visitExistingEdges`, `success` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，执行或准备 LLVM IR/JIT 代码的运行时行为，返回子系统相关的计算结果。

### Lines 39-51
```cpp
namespace llvm::jitlink {

class ELFJITLinker_x86 : public JITLinker<ELFJITLinker_x86> {
  friend class JITLinker<ELFJITLinker_x86>;

public:
  ELFJITLinker_x86(std::unique_ptr<JITLinkContext> Ctx,
                   std::unique_ptr<LinkGraph> G, PassConfiguration PassConfig)
      : JITLinker(std::move(Ctx), std::move(G), std::move(PassConfig)) {
    getPassConfig().PostAllocationPasses.push_back(
        [this](LinkGraph &G) { return getOrCreateGOTSymbol(G); });
  }

```
- **EN**: Introduces declarations for `llvm::jitlink`, `ELFJITLinker_x86`, `JITLinker`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm::jitlink`, `ELFJITLinker_x86`, `JITLinker` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 52-65
```cpp
private:
  Symbol *GOTSymbol = nullptr;

  Error getOrCreateGOTSymbol(LinkGraph &G) {
    auto DefineExternalGOTSymbolIfPresent =
        createDefineExternalSectionStartAndEndSymbolsPass(
            [&](LinkGraph &LG, Symbol &Sym) -> SectionRangeSymbolDesc {
              if (Sym.getName() != nullptr &&
                  *Sym.getName() == ELFGOTSymbolName)
                if (auto *GOTSection = G.findSectionByName(
                        x86::GOTTableManager::getSectionName())) {
                  GOTSymbol = &Sym;
                  return {*GOTSection, true};
                }
```
- **EN**: Implements logic around `getOrCreateGOTSymbol`, `createDefineExternalSectionStartAndEndSymbolsPass`, `getName`, `getSectionName`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `getOrCreateGOTSymbol`, `createDefineExternalSectionStartAndEndSymbolsPass`, `getName`, `getSectionName` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 66-73
```cpp
              return {};
            });

    // Try to attach _GLOBAL_OFFSET_TABLE_ to the GOT if it's defined as an
    // external.
    if (auto Err = DefineExternalGOTSymbolIfPresent(G))
      return Err;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 74-83
```cpp
    // If we succeeded then we're done.
    if (GOTSymbol)
      return Error::success();

    // Otherwise look for a GOT section: If it already has a start symbol we'll
    // record it, otherwise we'll create our own.
    // If there's a GOT section but we didn't find an external GOT symbol...
    if (auto *GOTSection =
            G.findSectionByName(x86::GOTTableManager::getSectionName())) {

```
- **EN**: Implements logic around `success`, `findSectionByName`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `findSectionByName` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 84-90
```cpp
      // Check for an existing defined symbol.
      for (auto *Sym : GOTSection->symbols())
        if (Sym->getName() != nullptr && *Sym->getName() == ELFGOTSymbolName) {
          GOTSymbol = Sym;
          return Error::success();
        }

```
- **EN**: Implements logic around `success`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `success` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 91-104
```cpp
      // If there's no defined symbol then create one.
      SectionRange SR(*GOTSection);

      if (SR.empty()) {
        GOTSymbol =
            &G.addAbsoluteSymbol(ELFGOTSymbolName, orc::ExecutorAddr(), 0,
                                 Linkage::Strong, Scope::Local, true);
      } else {
        GOTSymbol =
            &G.addDefinedSymbol(*SR.getFirstBlock(), 0, ELFGOTSymbolName, 0,
                                Linkage::Strong, Scope::Local, false, true);
      }
    }

```
- **EN**: Implements logic around `SR`, `addAbsoluteSymbol`, `addDefinedSymbol`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `SR`, `addAbsoluteSymbol`, `addDefinedSymbol` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 105-112
```cpp
    return Error::success();
  }

  Error applyFixup(LinkGraph &G, Block &B, const Edge &E) const {
    return x86::applyFixup(G, B, E, GOTSymbol);
  }
};

```
- **EN**: Implements logic around `success`, `applyFixup`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `applyFixup` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 113-126
```cpp
class ELFLinkGraphBuilder_x86 : public ELFLinkGraphBuilder<object::ELF32LE> {
private:
  using ELFT = object::ELF32LE;

  Expected<x86::EdgeKind_x86> getRelocationKind(const uint32_t Type) {
    switch (Type) {
    case ELF::R_386_32:
      return x86::Pointer32;
    case ELF::R_386_PC32:
      return x86::PCRel32;
    case ELF::R_386_16:
      return x86::Pointer16;
    case ELF::R_386_PC16:
      return x86::PCRel16;
```
- **EN**: Introduces declarations for `ELFLinkGraphBuilder_x86`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `ELFLinkGraphBuilder_x86` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 127-139
```cpp
    case ELF::R_386_GOT32:
      return x86::RequestGOTAndTransformToDelta32FromGOT;
    case ELF::R_386_GOT32X:
      // TODO: Add a relaxable edge kind and update relaxation optimization.
      return x86::RequestGOTAndTransformToDelta32FromGOT;
    case ELF::R_386_GOTPC:
      return x86::Delta32;
    case ELF::R_386_GOTOFF:
      return x86::Delta32FromGOT;
    case ELF::R_386_PLT32:
      return x86::BranchPCRel32;
    }

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 140-149
```cpp
    return make_error<JITLinkError>(
        "In " + G->getName() + ": Unsupported x86 relocation type " +
        object::getELFRelocationTypeName(ELF::EM_386, Type));
  }

  Error addRelocations() override {
    LLVM_DEBUG(dbgs() << "Adding relocations\n");
    using Base = ELFLinkGraphBuilder<ELFT>;
    using Self = ELFLinkGraphBuilder_x86;

```
- **EN**: Implements logic around `make_error<JITLinkError>`, `getName`, `getELFRelocationTypeName`, `addRelocations`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `getName`, `getELFRelocationTypeName`, `addRelocations` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 150-156
```cpp
    for (const auto &RelSect : Base::Sections) {
      // Validate the section to read relocation entries from.
      if (RelSect.sh_type == ELF::SHT_RELA)
        return make_error<StringError>(
            "No SHT_RELA in valid x86 ELF object files",
            inconvertibleErrorCode());

```
- **EN**: Implements logic around `make_error<StringError>`, `inconvertibleErrorCode`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<StringError>`, `inconvertibleErrorCode` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 157-164
```cpp
      if (Error Err = Base::forEachRelRelocation(RelSect, this,
                                                 &Self::addSingleRelocation))
        return Err;
    }

    return Error::success();
  }

```
- **EN**: Implements logic around `success`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `success` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 165-171
```cpp
  Error addSingleRelocation(const typename ELFT::Rel &Rel,
                            const typename ELFT::Shdr &FixupSection,
                            Block &BlockToFix) {
    using Base = ELFLinkGraphBuilder<ELFT>;

    auto ELFReloc = Rel.getType(false);

```
- **EN**: Implements logic around `addSingleRelocation`, `getType`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `addSingleRelocation`, `getType` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作。

### Lines 172-180
```cpp
    // R_386_NONE is a no-op.
    if (LLVM_UNLIKELY(ELFReloc == ELF::R_386_NONE))
      return Error::success();

    uint32_t SymbolIndex = Rel.getSymbol(false);
    auto ObjSymbol = Base::Obj.getRelocationSymbol(Rel, Base::SymTabSec);
    if (!ObjSymbol)
      return ObjSymbol.takeError();

```
- **EN**: Implements logic around `success`, `getSymbol`, `getRelocationSymbol`, `takeError`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `getSymbol`, `getRelocationSymbol`, `takeError` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 181-189
```cpp
    Symbol *GraphSymbol = Base::getGraphSymbol(SymbolIndex);
    if (!GraphSymbol)
      return make_error<StringError>(
          formatv("Could not find symbol at given index, did you add it to "
                  "JITSymbolTable? index: {0}, shndx: {1} Size of table: {2}",
                  SymbolIndex, (*ObjSymbol)->st_shndx,
                  Base::GraphSymbols.size()),
          inconvertibleErrorCode());

```
- **EN**: Implements logic around `getGraphSymbol`, `make_error<StringError>`, `formatv`, `size`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getGraphSymbol`, `make_error<StringError>`, `formatv`, `size`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 190-196
```cpp
    Expected<x86::EdgeKind_x86> Kind = getRelocationKind(ELFReloc);
    if (!Kind)
      return Kind.takeError();

    auto FixupAddress = orc::ExecutorAddr(FixupSection.sh_addr) + Rel.r_offset;
    int64_t Addend = 0;

```
- **EN**: Implements logic around `getRelocationKind`, `takeError`, `ExecutorAddr`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getRelocationKind`, `takeError`, `ExecutorAddr` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 197-210
```cpp
    switch (*Kind) {
    case x86::Pointer32:
    case x86::PCRel32:
    case x86::RequestGOTAndTransformToDelta32FromGOT:
    case x86::Delta32:
    case x86::Delta32FromGOT:
    case x86::BranchPCRel32:
    case x86::BranchPCRel32ToPtrJumpStub:
    case x86::BranchPCRel32ToPtrJumpStubBypassable: {
      const char *FixupContent = BlockToFix.getContent().data() +
                                 (FixupAddress - BlockToFix.getAddress());
      Addend = *(const support::little32_t *)FixupContent;
      break;
    }
```
- **EN**: Implements logic around `getContent`, `getAddress`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `getContent`, `getAddress` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 211-219
```cpp
    case x86::Pointer16:
    case x86::PCRel16: {
      const char *FixupContent = BlockToFix.getContent().data() +
                                 (FixupAddress - BlockToFix.getAddress());
      Addend = *(const support::little16_t *)FixupContent;
      break;
    }
    }

```
- **EN**: Implements logic around `getContent`, `getAddress`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `getContent`, `getAddress` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作。

### Lines 220-227
```cpp
    Edge::OffsetT Offset = FixupAddress - BlockToFix.getAddress();
    Edge GE(*Kind, Offset, *GraphSymbol, Addend);
    LLVM_DEBUG({
      dbgs() << "    ";
      printEdge(dbgs(), BlockToFix, GE, x86::getEdgeKindName(*Kind));
      dbgs() << "\n";
    });

```
- **EN**: Implements logic around `getAddress`, `GE`, `dbgs`, `printEdge`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `getAddress`, `GE`, `dbgs`, `printEdge` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作。

### Lines 228-240
```cpp
    BlockToFix.addEdge(std::move(GE));
    return Error::success();
  }

public:
  ELFLinkGraphBuilder_x86(StringRef FileName, const object::ELFFile<ELFT> &Obj,
                          std::shared_ptr<orc::SymbolStringPool> SSP, Triple TT,
                          SubtargetFeatures Features)
      : ELFLinkGraphBuilder<ELFT>(Obj, std::move(SSP), std::move(TT),
                                  std::move(Features), FileName,
                                  x86::getEdgeKindName) {}
};

```
- **EN**: Implements logic around `addEdge`, `success`, `ELFLinkGraphBuilder_x86`, `ELFLinkGraphBuilder<ELFT>`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `addEdge`, `success`, `ELFLinkGraphBuilder_x86`, `ELFLinkGraphBuilder<ELFT>`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 241-248
```cpp
Expected<std::unique_ptr<LinkGraph>>
createLinkGraphFromELFObject_x86(MemoryBufferRef ObjectBuffer,
                                 std::shared_ptr<orc::SymbolStringPool> SSP) {
  LLVM_DEBUG({
    dbgs() << "Building jitlink graph for new input "
           << ObjectBuffer.getBufferIdentifier() << "...\n";
  });

```
- **EN**: Implements logic around `createLinkGraphFromELFObject_x86`, `dbgs`, `getBufferIdentifier`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `createLinkGraphFromELFObject_x86`, `dbgs`, `getBufferIdentifier` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 249-256
```cpp
  auto ELFObj = object::ObjectFile::createELFObjectFile(ObjectBuffer);
  if (!ELFObj)
    return ELFObj.takeError();

  auto Features = (*ELFObj)->getFeatures();
  if (!Features)
    return Features.takeError();

```
- **EN**: Implements logic around `createELFObjectFile`, `takeError`, `getFeatures`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `createELFObjectFile`, `takeError`, `getFeatures` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 257-267
```cpp
  assert((*ELFObj)->getArch() == Triple::x86 &&
         "Only x86 (little endian) is supported for now");

  auto &ELFObjFile = cast<object::ELFObjectFile<object::ELF32LE>>(**ELFObj);

  return ELFLinkGraphBuilder_x86((*ELFObj)->getFileName(),
                                 ELFObjFile.getELFFile(), std::move(SSP),
                                 (*ELFObj)->makeTriple(), std::move(*Features))
      .buildGraph();
}

```
- **EN**: Implements logic around `assert`, `x86`, `ELF32LE>>`, `ELFLinkGraphBuilder_x86`, and 3 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `assert`, `x86`, `ELF32LE>>`, `ELFLinkGraphBuilder_x86`, and 3 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 268-277
```cpp
void link_ELF_x86(std::unique_ptr<LinkGraph> G,
                  std::unique_ptr<JITLinkContext> Ctx) {
  PassConfiguration Config;
  const Triple &TT = G->getTargetTriple();
  if (Ctx->shouldAddDefaultTargetPasses(TT)) {
    if (auto MarkLive = Ctx->getMarkLivePass(TT))
      Config.PrePrunePasses.push_back(std::move(MarkLive));
    else
      Config.PrePrunePasses.push_back(markAllSymbolsLive);

```
- **EN**: Implements logic around `link_ELF_x86`, `getTargetTriple`, `push_back`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `link_ELF_x86`, `getTargetTriple`, `push_back` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 278-286
```cpp
    // Add an in-place GOT and PLT build pass.
    Config.PostPrunePasses.push_back(buildTables_ELF_x86);

    // Add GOT/Stubs optimizer pass.
    Config.PreFixupPasses.push_back(x86::optimizeGOTAndStubAccesses);
  }
  if (auto Err = Ctx->modifyPassConfig(*G, Config))
    return Ctx->notifyFailed(std::move(Err));

```
- **EN**: Implements logic around `push_back`, `notifyFailed`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `push_back`, `notifyFailed` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 287-290
```cpp
  ELFJITLinker_x86::link(std::move(Ctx), std::move(G), std::move(Config));
}

} // namespace llvm::jitlink
```
- **EN**: Introduces declarations for `llvm::jitlink`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm::jitlink` 等声明，定义本文件后续使用的数据结构或接口。

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

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/JITLink/ELF_x86.h`, `DefineExternalSectionStartAndEndSymbols.h`, `ELFLinkGraphBuilder.h`, `JITLinkGeneric.h`, `llvm/BinaryFormat/ELF.h`, `llvm/ExecutionEngine/JITLink/x86.h`, `llvm/Object/ELFObjectFile.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Object, BinaryFormat
