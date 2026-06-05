# ELF_hexagon.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/JITLink/ELF_hexagon.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements JIT linker for ELF/hexagon.
  - **CN**: 实现面向即时编译代码的 JITLink 图链接、重定位处理以及特定文件格式的链接图 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===------ ELF_hexagon.cpp - JIT linker for ELF/hexagon ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-19
```cpp
//
// ELF/hexagon jit-link implementation.
//
//===----------------------------------------------------------------------===//

#include "llvm/ExecutionEngine/JITLink/ELF_hexagon.h"
#include "ELFLinkGraphBuilder.h"
#include "JITLinkGeneric.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/ExecutionEngine/JITLink/hexagon.h"
#include "llvm/Object/ELFObjectFile.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/JITLink/ELF_hexagon.h`, `ELFLinkGraphBuilder.h`, `JITLinkGeneric.h`, `llvm/BinaryFormat/ELF.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/JITLink/ELF_hexagon.h`, `ELFLinkGraphBuilder.h`, `JITLinkGeneric.h`, `llvm/BinaryFormat/ELF.h`。

### Lines 20-26
```cpp
#define DEBUG_TYPE "jitlink"

using namespace llvm;
using namespace llvm::jitlink;

namespace {

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 27-35
```cpp
class ELFJITLinker_hexagon : public JITLinker<ELFJITLinker_hexagon> {
  friend class JITLinker<ELFJITLinker_hexagon>;

public:
  ELFJITLinker_hexagon(std::unique_ptr<JITLinkContext> Ctx,
                       std::unique_ptr<LinkGraph> G,
                       PassConfiguration PassConfig)
      : JITLinker(std::move(Ctx), std::move(G), std::move(PassConfig)) {}

```
- **EN**: Introduces declarations for `ELFJITLinker_hexagon`, `JITLinker`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `ELFJITLinker_hexagon`, `JITLinker` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 36-46
```cpp
private:
  Error applyFixup(LinkGraph &G, Block &B, const Edge &E) const {
    return hexagon::applyFixup(G, B, E);
  }
};

class ELFLinkGraphBuilder_hexagon
    : public ELFLinkGraphBuilder<object::ELF32LE> {
private:
  using ELFT = object::ELF32LE;

```
- **EN**: Introduces declarations for `ELFLinkGraphBuilder_hexagon`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `ELFLinkGraphBuilder_hexagon` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 47-60
```cpp
  Expected<hexagon::EdgeKind_hexagon> getRelocationKind(const uint32_t Type) {
    switch (Type) {
    case ELF::R_HEX_32:
      return hexagon::Pointer32;
    case ELF::R_HEX_32_PCREL:
      return hexagon::PCRel32;
    case ELF::R_HEX_B22_PCREL:
    case ELF::R_HEX_PLT_B22_PCREL:
    // PLT and GD_PLT variants are mapped to plain branch edges since JITLink
    // resolves all symbols directly within contiguous JIT memory. When the
    // GOT/PLT stubs builder is added (see TODO in link_ELF_hexagon), these
    // should map to a distinct edge kind that triggers stub generation.
    // GD_PLT does not handle TLS __tls_get_addr calls.
    case ELF::R_HEX_GD_PLT_B22_PCREL:
```
- **EN**: Implements logic around `getRelocationKind`; this block handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getRelocationKind` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 61-74
```cpp
      return hexagon::B22_PCREL;
    case ELF::R_HEX_B15_PCREL:
      return hexagon::B15_PCREL;
    case ELF::R_HEX_B13_PCREL:
      return hexagon::B13_PCREL;
    case ELF::R_HEX_B9_PCREL:
      return hexagon::B9_PCREL;
    case ELF::R_HEX_B7_PCREL:
      return hexagon::B7_PCREL;
    case ELF::R_HEX_HI16:
      return hexagon::HI16;
    case ELF::R_HEX_LO16:
      return hexagon::LO16;
    case ELF::R_HEX_32_6_X:
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 75-88
```cpp
      return hexagon::Word32_6_X;
    case ELF::R_HEX_B32_PCREL_X:
    case ELF::R_HEX_GD_PLT_B32_PCREL_X: // See PLT/GD_PLT note above.
      return hexagon::B32_PCREL_X;
    case ELF::R_HEX_B22_PCREL_X:
    case ELF::R_HEX_GD_PLT_B22_PCREL_X: // See PLT/GD_PLT note above.
      return hexagon::B22_PCREL_X;
    case ELF::R_HEX_B15_PCREL_X:
      return hexagon::B15_PCREL_X;
    case ELF::R_HEX_B13_PCREL_X:
      return hexagon::B13_PCREL_X;
    case ELF::R_HEX_B9_PCREL_X:
      return hexagon::B9_PCREL_X;
    case ELF::R_HEX_B7_PCREL_X:
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 89-102
```cpp
      return hexagon::B7_PCREL_X;
    case ELF::R_HEX_6_X:
      return hexagon::Word6_X;
    case ELF::R_HEX_6_PCREL_X:
      return hexagon::Word6_PCREL_X;
    case ELF::R_HEX_8_X:
      return hexagon::Word8_X;
    case ELF::R_HEX_9_X:
      return hexagon::Word9_X;
    case ELF::R_HEX_10_X:
      return hexagon::Word10_X;
    case ELF::R_HEX_11_X:
      return hexagon::Word11_X;
    case ELF::R_HEX_12_X:
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 103-112
```cpp
      return hexagon::Word12_X;
    case ELF::R_HEX_16_X:
      return hexagon::Word16_X;
    }

    return make_error<JITLinkError>(
        "In " + G->getName() + ": Unsupported Hexagon relocation type " +
        object::getELFRelocationTypeName(ELF::EM_HEXAGON, Type));
  }

```
- **EN**: Implements logic around `make_error<JITLinkError>`, `getName`, `getELFRelocationTypeName`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `getName`, `getELFRelocationTypeName` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 113-124
```cpp
  Error addRelocations() override {
    LLVM_DEBUG(dbgs() << "Adding relocations\n");
    using Base = ELFLinkGraphBuilder<ELFT>;
    using Self = ELFLinkGraphBuilder_hexagon;

    for (const auto &RelSect : Base::Sections) {
      // Hexagon uses SHT_RELA.
      if (RelSect.sh_type == ELF::SHT_REL)
        return make_error<StringError>(
            "Unexpected SHT_REL section in Hexagon ELF object",
            inconvertibleErrorCode());

```
- **EN**: Implements logic around `addRelocations`, `make_error<StringError>`, `inconvertibleErrorCode`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `addRelocations`, `make_error<StringError>`, `inconvertibleErrorCode` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 125-132
```cpp
      if (Error Err = Base::forEachRelaRelocation(RelSect, this,
                                                  &Self::addSingleRelocation))
        return Err;
    }

    return Error::success();
  }

```
- **EN**: Implements logic around `success`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `success` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 133-139
```cpp
  Error addSingleRelocation(const typename ELFT::Rela &Rel,
                            const typename ELFT::Shdr &FixupSection,
                            Block &BlockToFix) {
    using Base = ELFLinkGraphBuilder<ELFT>;

    auto ELFReloc = Rel.getType(false);

```
- **EN**: Implements logic around `addSingleRelocation`, `getType`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `addSingleRelocation`, `getType` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作。

### Lines 140-147
```cpp
    if (LLVM_UNLIKELY(ELFReloc == ELF::R_HEX_NONE))
      return Error::success();

    uint32_t SymbolIndex = Rel.getSymbol(false);
    auto ObjSymbol = Base::Obj.getRelocationSymbol(Rel, Base::SymTabSec);
    if (!ObjSymbol)
      return ObjSymbol.takeError();

```
- **EN**: Implements logic around `success`, `getSymbol`, `getRelocationSymbol`, `takeError`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `getSymbol`, `getRelocationSymbol`, `takeError` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 148-156
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

### Lines 157-163
```cpp
    Expected<hexagon::EdgeKind_hexagon> Kind = getRelocationKind(ELFReloc);
    if (!Kind)
      return Kind.takeError();

    auto FixupAddress = orc::ExecutorAddr(FixupSection.sh_addr) + Rel.r_offset;
    int64_t Addend = Rel.r_addend;

```
- **EN**: Implements logic around `getRelocationKind`, `takeError`, `ExecutorAddr`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getRelocationKind`, `takeError`, `ExecutorAddr` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 164-171
```cpp
    Edge::OffsetT Offset = FixupAddress - BlockToFix.getAddress();
    Edge GE(*Kind, Offset, *GraphSymbol, Addend);
    LLVM_DEBUG({
      dbgs() << "    ";
      printEdge(dbgs(), BlockToFix, GE, hexagon::getEdgeKindName(*Kind));
      dbgs() << "\n";
    });

```
- **EN**: Implements logic around `getAddress`, `GE`, `dbgs`, `printEdge`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `getAddress`, `GE`, `dbgs`, `printEdge` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作。

### Lines 172-185
```cpp
    BlockToFix.addEdge(std::move(GE));
    return Error::success();
  }

public:
  ELFLinkGraphBuilder_hexagon(StringRef FileName,
                              const object::ELFFile<ELFT> &Obj,
                              std::shared_ptr<orc::SymbolStringPool> SSP,
                              Triple TT, SubtargetFeatures Features)
      : ELFLinkGraphBuilder<ELFT>(Obj, std::move(SSP), std::move(TT),
                                  std::move(Features), FileName,
                                  hexagon::getEdgeKindName) {}
};

```
- **EN**: Implements logic around `addEdge`, `success`, `ELFLinkGraphBuilder_hexagon`, `ELFLinkGraphBuilder<ELFT>`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `addEdge`, `success`, `ELFLinkGraphBuilder_hexagon`, `ELFLinkGraphBuilder<ELFT>`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 186-196
```cpp
} // anonymous namespace

namespace llvm::jitlink {

Expected<std::unique_ptr<LinkGraph>> createLinkGraphFromELFObject_hexagon(
    MemoryBufferRef ObjectBuffer, std::shared_ptr<orc::SymbolStringPool> SSP) {
  LLVM_DEBUG({
    dbgs() << "Building jitlink graph for new input "
           << ObjectBuffer.getBufferIdentifier() << "...\n";
  });

```
- **EN**: Introduces declarations for `llvm::jitlink`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm::jitlink` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 197-204
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

### Lines 205-215
```cpp
  assert((*ELFObj)->getArch() == Triple::hexagon &&
         "Only Hexagon is supported");

  auto &ELFObjFile = cast<object::ELFObjectFile<object::ELF32LE>>(**ELFObj);

  return ELFLinkGraphBuilder_hexagon(
             (*ELFObj)->getFileName(), ELFObjFile.getELFFile(), std::move(SSP),
             (*ELFObj)->makeTriple(), std::move(*Features))
      .buildGraph();
}

```
- **EN**: Implements logic around `assert`, `ELF32LE>>`, `ELFLinkGraphBuilder_hexagon`, `getFileName`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `assert`, `ELF32LE>>`, `ELFLinkGraphBuilder_hexagon`, `getFileName`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 216-229
```cpp
void link_ELF_hexagon(std::unique_ptr<LinkGraph> G,
                      std::unique_ptr<JITLinkContext> Ctx) {
  PassConfiguration Config;
  const Triple &TT = G->getTargetTriple();
  if (Ctx->shouldAddDefaultTargetPasses(TT)) {
    // TODO: Add GOT/PLT stubs builder when external symbol support is needed.
    // TODO: Add eh-frame passes when exception handling support is needed.
    if (auto MarkLive = Ctx->getMarkLivePass(TT))
      Config.PrePrunePasses.push_back(std::move(MarkLive));
    else
      Config.PrePrunePasses.push_back(markAllSymbolsLive);
  }
  if (auto Err = Ctx->modifyPassConfig(*G, Config))
    return Ctx->notifyFailed(std::move(Err));
```
- **EN**: Implements logic around `link_ELF_hexagon`, `getTargetTriple`, `push_back`, `notifyFailed`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `link_ELF_hexagon`, `getTargetTriple`, `push_back`, `notifyFailed` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 230-234
```cpp

  ELFJITLinker_hexagon::link(std::move(Ctx), std::move(G), std::move(Config));
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

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/JITLink/ELF_hexagon.h`, `ELFLinkGraphBuilder.h`, `JITLinkGeneric.h`, `llvm/BinaryFormat/ELF.h`, `llvm/ExecutionEngine/JITLink/hexagon.h`, `llvm/Object/ELFObjectFile.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Object, BinaryFormat
