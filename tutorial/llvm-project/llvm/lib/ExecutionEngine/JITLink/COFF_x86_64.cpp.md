# COFF_x86_64.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/JITLink/COFF_x86_64.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements JIT linker implementation for COFF/x86_64.
  - **CN**: 实现面向即时编译代码的 JITLink 图链接、重定位处理以及特定文件格式的链接图 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===----- COFF_x86_64.cpp - JIT linker implementation for COFF/x86_64 ----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// COFF/x86_64 jit-link implementation.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 12-22
```cpp

#include "llvm/ExecutionEngine/JITLink/COFF_x86_64.h"
#include "COFFLinkGraphBuilder.h"
#include "JITLinkGeneric.h"
#include "SEHFrameSupport.h"
#include "llvm/BinaryFormat/COFF.h"
#include "llvm/ExecutionEngine/JITLink/COFF.h"
#include "llvm/ExecutionEngine/JITLink/x86_64.h"
#include "llvm/Object/COFF.h"
#include "llvm/Support/Endian.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/JITLink/COFF_x86_64.h`, `COFFLinkGraphBuilder.h`, `JITLinkGeneric.h`, `SEHFrameSupport.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/JITLink/COFF_x86_64.h`, `COFFLinkGraphBuilder.h`, `JITLinkGeneric.h`, `SEHFrameSupport.h`。

### Lines 23-37
```cpp
#define DEBUG_TYPE "jitlink"

using namespace llvm;
using namespace llvm::jitlink;

namespace {

enum EdgeKind_coff_x86_64 : Edge::Kind {
  PCRel32 = x86_64::FirstPlatformRelocation,
  Pointer32NB,
  Pointer64,
  SectionIdx16,
  SecRel32,
};

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 38-46
```cpp
class COFFJITLinker_x86_64 : public JITLinker<COFFJITLinker_x86_64> {
  friend class JITLinker<COFFJITLinker_x86_64>;

public:
  COFFJITLinker_x86_64(std::unique_ptr<JITLinkContext> Ctx,
                       std::unique_ptr<LinkGraph> G,
                       PassConfiguration PassConfig)
      : JITLinker(std::move(Ctx), std::move(G), std::move(PassConfig)) {}

```
- **EN**: Introduces declarations for `COFFJITLinker_x86_64`, `JITLinker`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `COFFJITLinker_x86_64`, `JITLinker` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 47-57
```cpp
private:
  Error applyFixup(LinkGraph &G, Block &B, const Edge &E) const {
    return x86_64::applyFixup(G, B, E, nullptr);
  }
};

class COFFLinkGraphBuilder_x86_64 : public COFFLinkGraphBuilder {
private:
  Error addRelocations() override {
    LLVM_DEBUG(dbgs() << "Processing relocations:\n");

```
- **EN**: Introduces declarations for `COFFLinkGraphBuilder_x86_64`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `COFFLinkGraphBuilder_x86_64` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 58-75
```cpp
    for (const auto &RelSect : sections())
      if (Error Err = COFFLinkGraphBuilder::forEachRelocation(
              RelSect, this, &COFFLinkGraphBuilder_x86_64::addSingleRelocation))
        return Err;

    return Error::success();
  }

  Error addSingleRelocation(const object::RelocationRef &Rel,
                            const object::SectionRef &FixupSect,
                            Block &BlockToFix) {
    const object::coff_relocation *COFFRel = getObject().getCOFFRelocation(Rel);
    auto SymbolIt = Rel.getSymbol();
    if (SymbolIt == getObject().symbol_end()) {
      return make_error<StringError>(
          formatv("Invalid symbol index in relocation entry. "
                  "index: {0}, section: {1}",
                  COFFRel->SymbolTableIndex, FixupSect.getIndex()),
```
- **EN**: Implements logic around `success`, `addSingleRelocation`, `getObject`, `getSymbol`, and 3 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `addSingleRelocation`, `getObject`, `getSymbol`, and 3 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 76-89
```cpp
          inconvertibleErrorCode());
    }

    object::COFFSymbolRef COFFSymbol = getObject().getCOFFSymbol(*SymbolIt);
    COFFSymbolIndex SymIndex = getObject().getSymbolIndex(COFFSymbol);

    Symbol *GraphSymbol = getGraphSymbol(SymIndex);
    if (!GraphSymbol)
      return make_error<StringError>(
          formatv("Could not find symbol at given index, did you add it to "
                  "JITSymbolTable? index: {0}, section: {1}",
                  SymIndex, FixupSect.getIndex()),
          inconvertibleErrorCode());

```
- **EN**: Implements logic around `inconvertibleErrorCode`, `getObject`, `getGraphSymbol`, `make_error<StringError>`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `inconvertibleErrorCode`, `getObject`, `getGraphSymbol`, `make_error<StringError>`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 90-98
```cpp
    int64_t Addend = 0;
    orc::ExecutorAddr FixupAddress =
        orc::ExecutorAddr(FixupSect.getAddress()) + Rel.getOffset();
    Edge::OffsetT Offset = FixupAddress - BlockToFix.getAddress();

    Edge::Kind Kind = Edge::Invalid;
    const char *FixupPtr = BlockToFix.getContent().data() + Offset;
    Symbol *ImageBase = GetImageBaseSymbol()(getGraph());

```
- **EN**: Implements logic around `ExecutorAddr`, `getAddress`, `getContent`, `GetImageBaseSymbol`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `ExecutorAddr`, `getAddress`, `getContent`, `GetImageBaseSymbol` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作。

### Lines 99-116
```cpp
    switch (Rel.getType()) {
    case COFF::RelocationTypeAMD64::IMAGE_REL_AMD64_ADDR32NB: {
      if (!ImageBase)
        ImageBase = &addImageBaseSymbol();
      Kind = EdgeKind_coff_x86_64::Pointer32NB;
      Addend = *reinterpret_cast<const support::little32_t *>(FixupPtr);
      break;
    }
    case COFF::RelocationTypeAMD64::IMAGE_REL_AMD64_REL32: {
      Kind = EdgeKind_coff_x86_64::PCRel32;
      Addend = *reinterpret_cast<const support::little32_t *>(FixupPtr);
      break;
    }
    case COFF::RelocationTypeAMD64::IMAGE_REL_AMD64_REL32_1: {
      Kind = EdgeKind_coff_x86_64::PCRel32;
      Addend = *reinterpret_cast<const support::little32_t *>(FixupPtr);
      Addend -= 1;
      break;
```
- **EN**: Implements logic around `addImageBaseSymbol`; this block handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `addImageBaseSymbol` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 117-134
```cpp
    }
    case COFF::RelocationTypeAMD64::IMAGE_REL_AMD64_REL32_2: {
      Kind = EdgeKind_coff_x86_64::PCRel32;
      Addend = *reinterpret_cast<const support::little32_t *>(FixupPtr);
      Addend -= 2;
      break;
    }
    case COFF::RelocationTypeAMD64::IMAGE_REL_AMD64_REL32_3: {
      Kind = EdgeKind_coff_x86_64::PCRel32;
      Addend = *reinterpret_cast<const support::little32_t *>(FixupPtr);
      Addend -= 3;
      break;
    }
    case COFF::RelocationTypeAMD64::IMAGE_REL_AMD64_REL32_4: {
      Kind = EdgeKind_coff_x86_64::PCRel32;
      Addend = *reinterpret_cast<const support::little32_t *>(FixupPtr);
      Addend -= 4;
      break;
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 135-152
```cpp
    }
    case COFF::RelocationTypeAMD64::IMAGE_REL_AMD64_REL32_5: {
      Kind = EdgeKind_coff_x86_64::PCRel32;
      Addend = *reinterpret_cast<const support::little32_t *>(FixupPtr);
      Addend -= 5;
      break;
    }
    case COFF::RelocationTypeAMD64::IMAGE_REL_AMD64_ADDR64: {
      Kind = EdgeKind_coff_x86_64::Pointer64;
      Addend = *reinterpret_cast<const support::little64_t *>(FixupPtr);
      break;
    }
    case COFF::RelocationTypeAMD64::IMAGE_REL_AMD64_SECTION: {
      Kind = EdgeKind_coff_x86_64::SectionIdx16;
      Addend = *reinterpret_cast<const support::little16_t *>(FixupPtr);
      uint64_t SectionIdx = 0;
      if (COFFSymbol.isAbsolute())
        SectionIdx = getObject().getNumberOfSections() + 1;
```
- **EN**: Implements logic around `getObject`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `getObject` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 153-170
```cpp
      else
        SectionIdx = COFFSymbol.getSectionNumber();

      auto *AbsSym = &getGraph().addAbsoluteSymbol(
          "secidx", orc::ExecutorAddr(SectionIdx), 2, Linkage::Strong,
          Scope::Local, false);
      GraphSymbol = AbsSym;
      break;
    }
    case COFF::RelocationTypeAMD64::IMAGE_REL_AMD64_SECREL: {
      // FIXME: SECREL to external symbol should be handled
      if (!GraphSymbol->isDefined())
        return Error::success();
      Kind = EdgeKind_coff_x86_64::SecRel32;
      Addend = *reinterpret_cast<const support::little32_t *>(FixupPtr);
      break;
    }
    default: {
```
- **EN**: Implements logic around `getSectionNumber`, `getGraph`, `ExecutorAddr`, `success`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getSectionNumber`, `getGraph`, `ExecutorAddr`, `success` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 171-182
```cpp
      return make_error<JITLinkError>("Unsupported x86_64 relocation:" +
                                      formatv("{0:d}", Rel.getType()));
    }
    };

    Edge GE(Kind, Offset, *GraphSymbol, Addend);
    LLVM_DEBUG({
      dbgs() << "    ";
      printEdge(dbgs(), BlockToFix, GE, getCOFFX86RelocationKindName(Kind));
      dbgs() << "\n";
    });

```
- **EN**: Implements logic around `make_error<JITLinkError>`, `formatv`, `GE`, `dbgs`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `formatv`, `GE`, `dbgs`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 183-196
```cpp
    BlockToFix.addEdge(std::move(GE));

    return Error::success();
  }

public:
  COFFLinkGraphBuilder_x86_64(const object::COFFObjectFile &Obj,
                              std::shared_ptr<orc::SymbolStringPool> SSP,
                              const Triple T, const SubtargetFeatures Features)
      : COFFLinkGraphBuilder(Obj, std::move(SSP), std::move(T),
                             std::move(Features),
                             getCOFFX86RelocationKindName) {}
};

```
- **EN**: Implements logic around `addEdge`, `success`, `COFFLinkGraphBuilder_x86_64`, `COFFLinkGraphBuilder`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `addEdge`, `success`, `COFFLinkGraphBuilder_x86_64`, `COFFLinkGraphBuilder`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 197-214
```cpp
class COFFLinkGraphLowering_x86_64 {
public:
  // Lowers COFF x86_64 specific edges to generic x86_64 edges.
  Error operator()(LinkGraph &G) {
    for (auto *B : G.blocks()) {
      for (auto &E : B->edges()) {
        switch (E.getKind()) {
        case EdgeKind_coff_x86_64::Pointer32NB: {
          auto ImageBase = GetImageBase(G);
          assert(ImageBase && "__ImageBase symbol must be defined");
          E.setAddend(E.getAddend() - ImageBase->getAddress().getValue());
          E.setKind(x86_64::Pointer32);
          break;
        }
        case EdgeKind_coff_x86_64::PCRel32: {
          E.setKind(x86_64::PCRel32);
          break;
        }
```
- **EN**: Introduces declarations for `COFFLinkGraphLowering_x86_64`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `COFFLinkGraphLowering_x86_64` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 215-232
```cpp
        case EdgeKind_coff_x86_64::Pointer64: {
          E.setKind(x86_64::Pointer64);
          break;
        }
        case EdgeKind_coff_x86_64::SectionIdx16: {
          E.setKind(x86_64::Pointer16);
          break;
        }
        case EdgeKind_coff_x86_64::SecRel32: {
          E.setAddend(E.getAddend() -
                      getSectionStart(E.getTarget().getSection()).getValue());
          E.setKind(x86_64::Pointer32);
          break;
        }
        default:
          break;
        }
      }
```
- **EN**: Implements logic around `setKind`, `setAddend`, `getSectionStart`.
- **CN**: 围绕 `setKind`, `setAddend`, `getSectionStart` 实现具体逻辑。

### Lines 233-246
```cpp
    }
    return Error::success();
  }

private:
  orc::ExecutorAddr getSectionStart(Section &Sec) {
    auto [It, Inserted] = SectionStartCache.try_emplace(&Sec);
    if (Inserted) {
      SectionRange Range(Sec);
      It->second = Range.getStart();
    }
    return It->second;
  }

```
- **EN**: Implements logic around `success`, `getSectionStart`, `try_emplace`, `Range`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `getSectionStart`, `try_emplace`, `Range`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 247-255
```cpp
  GetImageBaseSymbol GetImageBase;
  DenseMap<Section *, orc::ExecutorAddr> SectionStartCache;
};
} // namespace

namespace llvm {
namespace jitlink {

/// Return the string name of the given COFF x86_64 edge kind.
```
- **EN**: Introduces declarations for `llvm`, `jitlink`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `jitlink` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 256-272
```cpp
const char *getCOFFX86RelocationKindName(Edge::Kind R) {
  switch (R) {
  case PCRel32:
    return "PCRel32";
  case Pointer32NB:
    return "Pointer32NB";
  case Pointer64:
    return "Pointer64";
  case SectionIdx16:
    return "SectionIdx16";
  case SecRel32:
    return "SecRel32";
  default:
    return x86_64::getEdgeKindName(R);
  }
}

```
- **EN**: Implements logic around `getCOFFX86RelocationKindName`, `getEdgeKindName`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getCOFFX86RelocationKindName`, `getEdgeKindName` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 273-283
```cpp
Expected<std::unique_ptr<LinkGraph>> createLinkGraphFromCOFFObject_x86_64(
    MemoryBufferRef ObjectBuffer, std::shared_ptr<orc::SymbolStringPool> SSP) {
  LLVM_DEBUG({
    dbgs() << "Building jitlink graph for new input "
           << ObjectBuffer.getBufferIdentifier() << "...\n";
  });

  auto COFFObj = object::ObjectFile::createCOFFObjectFile(ObjectBuffer);
  if (!COFFObj)
    return COFFObj.takeError();

```
- **EN**: Implements logic around `createLinkGraphFromCOFFObject_x86_64`, `dbgs`, `getBufferIdentifier`, `createCOFFObjectFile`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `createLinkGraphFromCOFFObject_x86_64`, `dbgs`, `getBufferIdentifier`, `createCOFFObjectFile`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 284-293
```cpp
  auto Features = (*COFFObj)->getFeatures();
  if (!Features)
    return Features.takeError();

  return COFFLinkGraphBuilder_x86_64(**COFFObj, std::move(SSP),
                                     (*COFFObj)->makeTriple(),
                                     std::move(*Features))
      .buildGraph();
}

```
- **EN**: Implements logic around `getFeatures`, `takeError`, `COFFLinkGraphBuilder_x86_64`, `makeTriple`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `getFeatures`, `takeError`, `COFFLinkGraphBuilder_x86_64`, `makeTriple`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 294-305
```cpp
void link_COFF_x86_64(std::unique_ptr<LinkGraph> G,
                      std::unique_ptr<JITLinkContext> Ctx) {
  PassConfiguration Config;
  const Triple &TT = G->getTargetTriple();
  if (Ctx->shouldAddDefaultTargetPasses(TT)) {
    // Add a mark-live pass.
    if (auto MarkLive = Ctx->getMarkLivePass(TT)) {
      Config.PrePrunePasses.push_back(std::move(MarkLive));
      Config.PrePrunePasses.push_back(SEHFrameKeepAlivePass(".pdata"));
    } else
      Config.PrePrunePasses.push_back(markAllSymbolsLive);

```
- **EN**: Implements logic around `link_COFF_x86_64`, `getTargetTriple`, `push_back`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `link_COFF_x86_64`, `getTargetTriple`, `push_back` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 306-315
```cpp
    // Add COFF edge lowering passes.
    Config.PreFixupPasses.push_back(COFFLinkGraphLowering_x86_64());
  }

  if (auto Err = Ctx->modifyPassConfig(*G, Config))
    return Ctx->notifyFailed(std::move(Err));

  COFFJITLinker_x86_64::link(std::move(Ctx), std::move(G), std::move(Config));
}

```
- **EN**: Implements logic around `push_back`, `notifyFailed`, `link`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `push_back`, `notifyFailed`, `link` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 316-317
```cpp
} // namespace jitlink
} // namespace llvm
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

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/JITLink/COFF_x86_64.h`, `COFFLinkGraphBuilder.h`, `JITLinkGeneric.h`, `SEHFrameSupport.h`, `llvm/BinaryFormat/COFF.h`, `llvm/ExecutionEngine/JITLink/COFF.h`, `llvm/ExecutionEngine/JITLink/x86_64.h`, `llvm/Object/COFF.h`, `llvm/Support/Endian.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Object, Support, BinaryFormat
