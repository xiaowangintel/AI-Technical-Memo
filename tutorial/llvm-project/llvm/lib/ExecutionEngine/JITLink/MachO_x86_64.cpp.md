# MachO_x86_64.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/JITLink/MachO_x86_64.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements JITLink graph-based linking, relocation handling, and format-specific link graph passes for just-in-time compiled code.
  - **CN**: 实现面向即时编译代码的 JITLink 图链接、重定位处理以及特定文件格式的链接图 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===---- MachO_x86_64.cpp -JIT linker implementation for MachO/x86-64 ----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// MachO/x86-64 jit-link implementation.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 13-26
```cpp
#include "llvm/ExecutionEngine/JITLink/MachO_x86_64.h"
#include "llvm/ExecutionEngine/JITLink/DWARFRecordSectionSplitter.h"
#include "llvm/ExecutionEngine/JITLink/x86_64.h"
#include "llvm/ExecutionEngine/Orc/Shared/MachOObjectFormat.h"

#include "CompactUnwindSupport.h"
#include "DefineExternalSectionStartAndEndSymbols.h"
#include "MachOLinkGraphBuilder.h"

#define DEBUG_TYPE "jitlink"

using namespace llvm;
using namespace llvm::jitlink;

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/JITLink/MachO_x86_64.h`, `llvm/ExecutionEngine/JITLink/DWARFRecordSectionSplitter.h`, `llvm/ExecutionEngine/JITLink/x86_64.h`, `llvm/ExecutionEngine/Orc/Shared/MachOObjectFormat.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/JITLink/MachO_x86_64.h`, `llvm/ExecutionEngine/JITLink/DWARFRecordSectionSplitter.h`, `llvm/ExecutionEngine/JITLink/x86_64.h`, `llvm/ExecutionEngine/Orc/Shared/MachOObjectFormat.h`。

### Lines 27-50
```cpp
namespace {

class MachOLinkGraphBuilder_x86_64 : public MachOLinkGraphBuilder {
public:
  MachOLinkGraphBuilder_x86_64(const object::MachOObjectFile &Obj,
                               std::shared_ptr<orc::SymbolStringPool> SSP,
                               SubtargetFeatures Features)
      : MachOLinkGraphBuilder(Obj, std::move(SSP),
                              Triple("x86_64-apple-darwin"),
                              std::move(Features), x86_64::getEdgeKindName) {}

private:
  enum MachONormalizedRelocationType : unsigned {
    MachOBranch32,
    MachOPointer32,
    MachOPointer64,
    MachOPointer64Anon,
    MachOPCRel32,
    MachOPCRel32Minus1,
    MachOPCRel32Minus2,
    MachOPCRel32Minus4,
    MachOPCRel32Anon,
    MachOPCRel32Minus1Anon,
    MachOPCRel32Minus2Anon,
```
- **EN**: Introduces declarations for `MachOLinkGraphBuilder_x86_64`, `MachONormalizedRelocationType`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `MachOLinkGraphBuilder_x86_64`, `MachONormalizedRelocationType` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 51-74
```cpp
    MachOPCRel32Minus4Anon,
    MachOPCRel32GOTLoad,
    MachOPCRel32GOT,
    MachOPCRel32TLV,
    MachOSubtractor32,
    MachOSubtractor64,
  };

  static Expected<MachONormalizedRelocationType>
  getRelocKind(const MachO::relocation_info &RI) {
    switch (RI.r_type) {
    case MachO::X86_64_RELOC_UNSIGNED:
      if (!RI.r_pcrel) {
        if (RI.r_length == 3)
          return RI.r_extern ? MachOPointer64 : MachOPointer64Anon;
        else if (RI.r_extern && RI.r_length == 2)
          return MachOPointer32;
      }
      break;
    case MachO::X86_64_RELOC_SIGNED:
      if (RI.r_pcrel && RI.r_length == 2)
        return RI.r_extern ? MachOPCRel32 : MachOPCRel32Anon;
      break;
    case MachO::X86_64_RELOC_BRANCH:
```
- **EN**: Implements logic around `getRelocKind`; this block handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getRelocKind` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 75-98
```cpp
      if (RI.r_pcrel && RI.r_extern && RI.r_length == 2)
        return MachOBranch32;
      break;
    case MachO::X86_64_RELOC_GOT_LOAD:
      if (RI.r_pcrel && RI.r_extern && RI.r_length == 2)
        return MachOPCRel32GOTLoad;
      break;
    case MachO::X86_64_RELOC_GOT:
      if (RI.r_pcrel && RI.r_extern && RI.r_length == 2)
        return MachOPCRel32GOT;
      break;
    case MachO::X86_64_RELOC_SUBTRACTOR:
      if (!RI.r_pcrel && RI.r_extern) {
        if (RI.r_length == 2)
          return MachOSubtractor32;
        else if (RI.r_length == 3)
          return MachOSubtractor64;
      }
      break;
    case MachO::X86_64_RELOC_SIGNED_1:
      if (RI.r_pcrel && RI.r_length == 2)
        return RI.r_extern ? MachOPCRel32Minus1 : MachOPCRel32Minus1Anon;
      break;
    case MachO::X86_64_RELOC_SIGNED_2:
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 99-111
```cpp
      if (RI.r_pcrel && RI.r_length == 2)
        return RI.r_extern ? MachOPCRel32Minus2 : MachOPCRel32Minus2Anon;
      break;
    case MachO::X86_64_RELOC_SIGNED_4:
      if (RI.r_pcrel && RI.r_length == 2)
        return RI.r_extern ? MachOPCRel32Minus4 : MachOPCRel32Minus4Anon;
      break;
    case MachO::X86_64_RELOC_TLV:
      if (RI.r_pcrel && RI.r_extern && RI.r_length == 2)
        return MachOPCRel32TLV;
      break;
    }

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 112-123
```cpp
    return make_error<JITLinkError>(
        "Unsupported x86-64 relocation: address=" +
        formatv("{0:x8}", RI.r_address) +
        ", symbolnum=" + formatv("{0:x6}", RI.r_symbolnum) +
        ", kind=" + formatv("{0:x1}", RI.r_type) +
        ", pc_rel=" + (RI.r_pcrel ? "true" : "false") +
        ", extern=" + (RI.r_extern ? "true" : "false") +
        ", length=" + formatv("{0:d}", RI.r_length));
  }

  using PairRelocInfo = std::tuple<Edge::Kind, Symbol *, uint64_t>;

```
- **EN**: Implements logic around `make_error<JITLinkError>`, `formatv`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `formatv` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 124-138
```cpp
  // Parses paired SUBTRACTOR/UNSIGNED relocations and, on success,
  // returns the edge kind and addend to be used.
  Expected<PairRelocInfo> parsePairRelocation(
      Block &BlockToFix, MachONormalizedRelocationType SubtractorKind,
      const MachO::relocation_info &SubRI, orc::ExecutorAddr FixupAddress,
      const char *FixupContent, object::relocation_iterator &UnsignedRelItr,
      object::relocation_iterator &RelEnd) {
    using namespace support;

    assert(((SubtractorKind == MachOSubtractor32 && SubRI.r_length == 2) ||
            (SubtractorKind == MachOSubtractor64 && SubRI.r_length == 3)) &&
           "Subtractor kind should match length");
    assert(SubRI.r_extern && "SUBTRACTOR reloc symbol should be extern");
    assert(!SubRI.r_pcrel && "SUBTRACTOR reloc should not be PCRel");

```
- **EN**: Introduces declarations for `support`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `support` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 139-152
```cpp
    if (UnsignedRelItr == RelEnd)
      return make_error<JITLinkError>("x86_64 SUBTRACTOR without paired "
                                      "UNSIGNED relocation");

    auto UnsignedRI = getRelocationInfo(UnsignedRelItr);

    if (SubRI.r_address != UnsignedRI.r_address)
      return make_error<JITLinkError>("x86_64 SUBTRACTOR and paired UNSIGNED "
                                      "point to different addresses");

    if (SubRI.r_length != UnsignedRI.r_length)
      return make_error<JITLinkError>("length of x86_64 SUBTRACTOR and paired "
                                      "UNSIGNED reloc must match");

```
- **EN**: Implements logic around `make_error<JITLinkError>`, `getRelocationInfo`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `getRelocationInfo` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 153-165
```cpp
    Symbol *FromSymbol;
    if (auto FromSymbolOrErr = findSymbolByIndex(SubRI.r_symbolnum))
      FromSymbol = FromSymbolOrErr->GraphSymbol;
    else
      return FromSymbolOrErr.takeError();

    // Read the current fixup value.
    uint64_t FixupValue = 0;
    if (SubRI.r_length == 3)
      FixupValue = *(const little64_t *)FixupContent;
    else
      FixupValue = *(const little32_t *)FixupContent;

```
- **EN**: Implements logic around `takeError`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `takeError` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 166-183
```cpp
    // Find 'ToSymbol' using symbol number or address, depending on whether the
    // paired UNSIGNED relocation is extern.
    Symbol *ToSymbol = nullptr;
    if (UnsignedRI.r_extern) {
      // Find target symbol by symbol index.
      if (auto ToSymbolOrErr = findSymbolByIndex(UnsignedRI.r_symbolnum))
        ToSymbol = ToSymbolOrErr->GraphSymbol;
      else
        return ToSymbolOrErr.takeError();
    } else {
      auto ToSymbolSec = findSectionByIndex(UnsignedRI.r_symbolnum - 1);
      if (!ToSymbolSec)
        return ToSymbolSec.takeError();
      ToSymbol = getSymbolByAddress(*ToSymbolSec, ToSymbolSec->Address);
      assert(ToSymbol && "No symbol for section");
      FixupValue -= ToSymbol->getAddress().getValue();
    }

```
- **EN**: Implements logic around `takeError`, `findSectionByIndex`, `getSymbolByAddress`, `assert`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `takeError`, `findSectionByIndex`, `getSymbolByAddress`, `assert`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 184-207
```cpp
    Edge::Kind DeltaKind;
    Symbol *TargetSymbol;
    uint64_t Addend;

    bool FixingFromSymbol = true;
    if (&BlockToFix == &FromSymbol->getAddressable()) {
      if (LLVM_UNLIKELY(&BlockToFix == &ToSymbol->getAddressable())) {
        // From and To are symbols in the same block. Decide direction by offset
        // instead.
        if (ToSymbol->getAddress() > FixupAddress)
          FixingFromSymbol = true;
        else if (FromSymbol->getAddress() > FixupAddress)
          FixingFromSymbol = false;
        else
          FixingFromSymbol = FromSymbol->getAddress() >= ToSymbol->getAddress();
      } else
        FixingFromSymbol = true;
    } else {
      if (&BlockToFix == &ToSymbol->getAddressable())
        FixingFromSymbol = false;
      else {
        // BlockToFix was neither FromSymbol nor ToSymbol.
        return make_error<JITLinkError>("SUBTRACTOR relocation must fix up "
                                        "either 'A' or 'B' (or a symbol in one "
```
- **EN**: Implements logic around `getAddress`, `make_error<JITLinkError>`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getAddress`, `make_error<JITLinkError>` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 208-223
```cpp
                                        "of their alt-entry groups)");
      }
    }

    if (FixingFromSymbol) {
      TargetSymbol = ToSymbol;
      DeltaKind = (SubRI.r_length == 3) ? x86_64::Delta64 : x86_64::Delta32;
      Addend = FixupValue + (FixupAddress - FromSymbol->getAddress());
      // FIXME: handle extern 'from'.
    } else {
      TargetSymbol = FromSymbol;
      DeltaKind =
          (SubRI.r_length == 3) ? x86_64::NegDelta64 : x86_64::NegDelta32;
      Addend = FixupValue - (FixupAddress - ToSymbol->getAddress());
    }

```
- **EN**: Implements logic around `getAddress`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `getAddress` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 224-236
```cpp
    return PairRelocInfo(DeltaKind, TargetSymbol, Addend);
  }

  Error addRelocations() override {
    using namespace support;
    auto &Obj = getObject();

    LLVM_DEBUG(dbgs() << "Processing relocations:\n");

    for (const auto &S : Obj.sections()) {

      orc::ExecutorAddr SectionAddress(S.getAddress());

```
- **EN**: Introduces declarations for `support`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `support` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 237-249
```cpp
      // Skip relocations virtual sections.
      if (S.isVirtual()) {
        if (!S.relocations().empty())
          return make_error<JITLinkError>("Virtual section contains "
                                          "relocations");
        continue;
      }

      auto NSec =
          findSectionByIndex(Obj.getSectionIndex(S.getRawDataRefImpl()));
      if (!NSec)
        return NSec.takeError();

```
- **EN**: Implements logic around `make_error<JITLinkError>`, `findSectionByIndex`, `takeError`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `findSectionByIndex`, `takeError` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 250-262
```cpp
      // Skip relocations for MachO sections without corresponding graph
      // sections.
      {
        if (!NSec->GraphSection) {
          LLVM_DEBUG({
            dbgs() << "  Skipping relocations for MachO section "
                   << NSec->SegName << "/" << NSec->SectName
                   << " which has no associated graph section\n";
          });
          continue;
        }
      }

```
- **EN**: Implements logic around `dbgs`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `dbgs` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 263-276
```cpp
      // Add relocations for section.
      for (auto RelItr = S.relocation_begin(), RelEnd = S.relocation_end();
           RelItr != RelEnd; ++RelItr) {

        MachO::relocation_info RI = getRelocationInfo(RelItr);

        // Find the address of the value to fix up.
        auto FixupAddress = SectionAddress + (uint32_t)RI.r_address;

        LLVM_DEBUG({
          dbgs() << "  " << NSec->SectName << " + "
                 << formatv("{0:x8}", RI.r_address) << ":\n";
        });

```
- **EN**: Implements logic around `getRelocationInfo`, `dbgs`, `formatv`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `getRelocationInfo`, `dbgs`, `formatv` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 277-290
```cpp
        // Find the block that the fixup points to.
        Block *BlockToFix = nullptr;
        {
          auto SymbolToFixOrErr = findSymbolByAddress(*NSec, FixupAddress);
          if (!SymbolToFixOrErr)
            return SymbolToFixOrErr.takeError();
          BlockToFix = &SymbolToFixOrErr->getBlock();
        }

        if (FixupAddress + orc::ExecutorAddrDiff(1ULL << RI.r_length) >
            BlockToFix->getAddress() + BlockToFix->getContent().size())
          return make_error<JITLinkError>(
              "Relocation extends past end of fixup block");

```
- **EN**: Implements logic around `findSymbolByAddress`, `takeError`, `getBlock`, `getAddress`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `findSymbolByAddress`, `takeError`, `getBlock`, `getAddress`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 291-305
```cpp
        // Get a pointer to the fixup content.
        const char *FixupContent = BlockToFix->getContent().data() +
                                   (FixupAddress - BlockToFix->getAddress());

        size_t FixupOffset = FixupAddress - BlockToFix->getAddress();

        // The target symbol and addend will be populated by the switch below.
        Symbol *TargetSymbol = nullptr;
        uint64_t Addend = 0;

        // Validate the relocation kind.
        auto MachORelocKind = getRelocKind(RI);
        if (!MachORelocKind)
          return MachORelocKind.takeError();

```
- **EN**: Implements logic around `getContent`, `getAddress`, `getRelocKind`, `takeError`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getContent`, `getAddress`, `getRelocKind`, `takeError` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 306-329
```cpp
        Edge::Kind Kind = Edge::Invalid;

        switch (*MachORelocKind) {
        case MachOBranch32:
          if (auto TargetSymbolOrErr = findSymbolByIndex(RI.r_symbolnum))
            TargetSymbol = TargetSymbolOrErr->GraphSymbol;
          else
            return TargetSymbolOrErr.takeError();
          Addend = *(const little32_t *)FixupContent;
          Kind = x86_64::BranchPCRel32;
          break;
        case MachOPCRel32:
          if (auto TargetSymbolOrErr = findSymbolByIndex(RI.r_symbolnum))
            TargetSymbol = TargetSymbolOrErr->GraphSymbol;
          else
            return TargetSymbolOrErr.takeError();
          Addend = *(const little32_t *)FixupContent - 4;
          Kind = x86_64::Delta32;
          break;
        case MachOPCRel32GOTLoad:
          if (auto TargetSymbolOrErr = findSymbolByIndex(RI.r_symbolnum))
            TargetSymbol = TargetSymbolOrErr->GraphSymbol;
          else
            return TargetSymbolOrErr.takeError();
```
- **EN**: Implements logic around `takeError`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `takeError` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 330-353
```cpp
          Addend = *(const little32_t *)FixupContent;
          Kind = x86_64::RequestGOTAndTransformToPCRel32GOTLoadREXRelaxable;
          if (FixupOffset < 3)
            return make_error<JITLinkError>("GOTLD at invalid offset " +
                                            formatv("{0}", FixupOffset));
          break;
        case MachOPCRel32GOT:
          if (auto TargetSymbolOrErr = findSymbolByIndex(RI.r_symbolnum))
            TargetSymbol = TargetSymbolOrErr->GraphSymbol;
          else
            return TargetSymbolOrErr.takeError();
          Addend = *(const little32_t *)FixupContent - 4;
          Kind = x86_64::RequestGOTAndTransformToDelta32;
          break;
        case MachOPCRel32TLV:
          if (auto TargetSymbolOrErr = findSymbolByIndex(RI.r_symbolnum))
            TargetSymbol = TargetSymbolOrErr->GraphSymbol;
          else
            return TargetSymbolOrErr.takeError();
          Addend = *(const little32_t *)FixupContent;
          Kind = x86_64::RequestTLVPAndTransformToPCRel32TLVPLoadREXRelaxable;
          if (FixupOffset < 3)
            return make_error<JITLinkError>("TLV at invalid offset " +
                                            formatv("{0}", FixupOffset));
```
- **EN**: Implements logic around `make_error<JITLinkError>`, `formatv`, `takeError`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `formatv`, `takeError` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 354-377
```cpp
          break;
        case MachOPointer32:
          if (auto TargetSymbolOrErr = findSymbolByIndex(RI.r_symbolnum))
            TargetSymbol = TargetSymbolOrErr->GraphSymbol;
          else
            return TargetSymbolOrErr.takeError();
          Addend = *(const ulittle32_t *)FixupContent;
          Kind = x86_64::Pointer32;
          break;
        case MachOPointer64:
          if (auto TargetSymbolOrErr = findSymbolByIndex(RI.r_symbolnum))
            TargetSymbol = TargetSymbolOrErr->GraphSymbol;
          else
            return TargetSymbolOrErr.takeError();
          Addend = *(const ulittle64_t *)FixupContent;
          Kind = x86_64::Pointer64;
          break;
        case MachOPointer64Anon: {
          orc::ExecutorAddr TargetAddress(*(const ulittle64_t *)FixupContent);
          auto TargetNSec = findSectionByIndex(RI.r_symbolnum - 1);
          if (!TargetNSec)
            return TargetNSec.takeError();
          if (auto TargetSymbolOrErr =
                  findSymbolByAddress(*TargetNSec, TargetAddress))
```
- **EN**: Implements logic around `takeError`, `TargetAddress`, `findSectionByIndex`, `findSymbolByAddress`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `takeError`, `TargetAddress`, `findSectionByIndex`, `findSymbolByAddress` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 378-401
```cpp
            TargetSymbol = &*TargetSymbolOrErr;
          else
            return TargetSymbolOrErr.takeError();
          Addend = TargetAddress - TargetSymbol->getAddress();
          Kind = x86_64::Pointer64;
          break;
        }
        case MachOPCRel32Minus1:
        case MachOPCRel32Minus2:
        case MachOPCRel32Minus4:
          if (auto TargetSymbolOrErr = findSymbolByIndex(RI.r_symbolnum))
            TargetSymbol = TargetSymbolOrErr->GraphSymbol;
          else
            return TargetSymbolOrErr.takeError();
          Addend = *(const little32_t *)FixupContent - 4;
          Kind = x86_64::Delta32;
          break;
        case MachOPCRel32Anon: {
          orc::ExecutorAddr TargetAddress(FixupAddress + 4 +
                                          *(const little32_t *)FixupContent);
          auto TargetNSec = findSectionByIndex(RI.r_symbolnum - 1);
          if (!TargetNSec)
            return TargetNSec.takeError();
          if (auto TargetSymbolOrErr =
```
- **EN**: Implements logic around `takeError`, `getAddress`, `TargetAddress`, `findSectionByIndex`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `takeError`, `getAddress`, `TargetAddress`, `findSectionByIndex` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 402-425
```cpp
                  findSymbolByAddress(*TargetNSec, TargetAddress))
            TargetSymbol = &*TargetSymbolOrErr;
          else
            return TargetSymbolOrErr.takeError();
          Addend = TargetAddress - TargetSymbol->getAddress() - 4;
          Kind = x86_64::Delta32;
          break;
        }
        case MachOPCRel32Minus1Anon:
        case MachOPCRel32Minus2Anon:
        case MachOPCRel32Minus4Anon: {
          orc::ExecutorAddrDiff Delta =
              4 + orc::ExecutorAddrDiff(
                      1ULL << (*MachORelocKind - MachOPCRel32Minus1Anon));
          orc::ExecutorAddr TargetAddress =
              FixupAddress + Delta + *(const little32_t *)FixupContent;
          auto TargetNSec = findSectionByIndex(RI.r_symbolnum - 1);
          if (!TargetNSec)
            return TargetNSec.takeError();
          if (auto TargetSymbolOrErr =
                  findSymbolByAddress(*TargetNSec, TargetAddress))
            TargetSymbol = &*TargetSymbolOrErr;
          else
            return TargetSymbolOrErr.takeError();
```
- **EN**: Implements logic around `findSymbolByAddress`, `takeError`, `getAddress`, `ExecutorAddrDiff`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `findSymbolByAddress`, `takeError`, `getAddress`, `ExecutorAddrDiff`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 426-447
```cpp
          Addend = TargetAddress - TargetSymbol->getAddress() - Delta;
          Kind = x86_64::Delta32;
          break;
        }
        case MachOSubtractor32:
        case MachOSubtractor64: {
          // We use Delta32/Delta64 to represent SUBTRACTOR relocations.
          // parsePairRelocation handles the paired reloc, and returns the
          // edge kind to be used (either Delta32/Delta64, or
          // NegDelta32/NegDelta64, depending on the direction of the
          // subtraction) along with the addend.
          auto PairInfo =
              parsePairRelocation(*BlockToFix, *MachORelocKind, RI,
                                  FixupAddress, FixupContent, ++RelItr, RelEnd);
          if (!PairInfo)
            return PairInfo.takeError();
          std::tie(Kind, TargetSymbol, Addend) = *PairInfo;
          assert(TargetSymbol && "No target symbol from parsePairRelocation?");
          break;
        }
        }

```
- **EN**: Implements logic around `getAddress`, `parsePairRelocation`, `takeError`, `tie`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getAddress`, `parsePairRelocation`, `takeError`, `tie`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 448-462
```cpp
        LLVM_DEBUG({
          dbgs() << "    ";
          Edge GE(Kind, FixupAddress - BlockToFix->getAddress(), *TargetSymbol,
                  Addend);
          printEdge(dbgs(), *BlockToFix, GE, x86_64::getEdgeKindName(Kind));
          dbgs() << "\n";
        });
        BlockToFix->addEdge(Kind, FixupAddress - BlockToFix->getAddress(),
                            *TargetSymbol, Addend);
      }
    }
    return Error::success();
  }
};

```
- **EN**: Implements logic around `dbgs`, `GE`, `printEdge`, `addEdge`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `dbgs`, `GE`, `printEdge`, `addEdge`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 463-474
```cpp
Error buildGOTAndStubs_MachO_x86_64(LinkGraph &G) {
  x86_64::GOTTableManager GOT(G);
  x86_64::PLTTableManager PLT(G, GOT);
  visitExistingEdges(G, GOT, PLT);
  return Error::success();
}

} // namespace

namespace llvm {
namespace jitlink {

```
- **EN**: Introduces declarations for `llvm`, `jitlink`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `jitlink` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 475-489
```cpp
class MachOJITLinker_x86_64 : public JITLinker<MachOJITLinker_x86_64> {
  friend class JITLinker<MachOJITLinker_x86_64>;

public:
  MachOJITLinker_x86_64(std::unique_ptr<JITLinkContext> Ctx,
                        std::unique_ptr<LinkGraph> G,
                        PassConfiguration PassConfig)
      : JITLinker(std::move(Ctx), std::move(G), std::move(PassConfig)) {}

private:
  Error applyFixup(LinkGraph &G, Block &B, const Edge &E) const {
    return x86_64::applyFixup(G, B, E, nullptr);
  }
};

```
- **EN**: Introduces declarations for `MachOJITLinker_x86_64`, `JITLinker`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `MachOJITLinker_x86_64`, `JITLinker` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 490-504
```cpp
Expected<std::unique_ptr<LinkGraph>> createLinkGraphFromMachOObject_x86_64(
    MemoryBufferRef ObjectBuffer, std::shared_ptr<orc::SymbolStringPool> SSP) {
  auto MachOObj = object::ObjectFile::createMachOObjectFile(ObjectBuffer);
  if (!MachOObj)
    return MachOObj.takeError();

  auto Features = (*MachOObj)->getFeatures();
  if (!Features)
    return Features.takeError();

  return MachOLinkGraphBuilder_x86_64(**MachOObj, std::move(SSP),
                                      std::move(*Features))
      .buildGraph();
}

```
- **EN**: Implements logic around `createLinkGraphFromMachOObject_x86_64`, `createMachOObjectFile`, `takeError`, `getFeatures`, and 3 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `createLinkGraphFromMachOObject_x86_64`, `createMachOObjectFile`, `takeError`, `getFeatures`, and 3 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 505-516
```cpp
struct CompactUnwindTraits_MachO_x86_64
    : public CompactUnwindTraits<CompactUnwindTraits_MachO_x86_64,
                                 /* PointerSize = */ 8> {
  // FIXME: Reinstate once we no longer need the MSVC workaround. See
  //        FIXME for CompactUnwindTraits in CompactUnwindSupport.h.
  // constexpr static size_t PointerSize = 8;

  constexpr static endianness Endianness = endianness::little;

  constexpr static uint32_t EncodingModeMask = 0x0f000000;
  constexpr static uint32_t DWARFSectionOffsetMask = 0x00ffffff;

```
- **EN**: Introduces declarations for `CompactUnwindTraits_MachO_x86_64`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `CompactUnwindTraits_MachO_x86_64` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 517-540
```cpp
  constexpr static uint32_t EBPFrameMode = 0x01000000;
  constexpr static uint32_t StackImmediateMode = 0x02000000;
  constexpr static uint32_t StackIndirectMode = 0x03000000;
  constexpr static uint32_t DWARFMode = 0x04000000;

  using GOTManager = x86_64::GOTTableManager;

  static bool encodingSpecifiesDWARF(uint32_t Encoding) {
    return (Encoding & EncodingModeMask) == DWARFMode;
  }

  static bool encodingCanBeMerged(uint32_t Encoding) {
    switch (Encoding & EncodingModeMask) {
    case EBPFrameMode:
    case StackImmediateMode:
      return true;
    case StackIndirectMode:
    case DWARFMode:
    default:
      return false;
    }
  }
};

```
- **EN**: Implements logic around `encodingSpecifiesDWARF`, `encodingCanBeMerged`; this block uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `encodingSpecifiesDWARF`, `encodingCanBeMerged` 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 541-552
```cpp
void link_MachO_x86_64(std::unique_ptr<LinkGraph> G,
                       std::unique_ptr<JITLinkContext> Ctx) {

  PassConfiguration Config;

  if (Ctx->shouldAddDefaultTargetPasses(G->getTargetTriple())) {
    // Add a mark-live pass.
    if (auto MarkLive = Ctx->getMarkLivePass(G->getTargetTriple()))
      Config.PrePrunePasses.push_back(std::move(MarkLive));
    else
      Config.PrePrunePasses.push_back(markAllSymbolsLive);

```
- **EN**: Implements logic around `link_MachO_x86_64`, `push_back`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `link_MachO_x86_64`, `push_back` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 553-567
```cpp
    // Add eh-frame passes.
    Config.PrePrunePasses.push_back(createEHFrameSplitterPass_MachO_x86_64());
    Config.PrePrunePasses.push_back(createEHFrameEdgeFixerPass_MachO_x86_64());

    // Create a compact-unwind manager for use in passes below.
    auto CompactUnwindMgr = std::make_shared<
        CompactUnwindManager<CompactUnwindTraits_MachO_x86_64>>(
        orc::MachOCompactUnwindSectionName, orc::MachOUnwindInfoSectionName,
        orc::MachOEHFrameSectionName);

    // Add compact unwind prepare pass.
    Config.PrePrunePasses.push_back([CompactUnwindMgr](LinkGraph &G) {
      return CompactUnwindMgr->prepareForPrune(G);
    });

```
- **EN**: Implements logic around `push_back`, `CompactUnwindManager<CompactUnwindTraits_MachO_x86_64>>`, `prepareForPrune`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `push_back`, `CompactUnwindManager<CompactUnwindTraits_MachO_x86_64>>`, `prepareForPrune` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 568-580
```cpp
    // Resolve any external section start / end symbols.
    Config.PostAllocationPasses.push_back(
        createDefineExternalSectionStartAndEndSymbolsPass(
            identifyMachOSectionStartAndEndSymbols));

    // Add an in-place GOT/Stubs pass.
    Config.PostPrunePasses.push_back(buildGOTAndStubs_MachO_x86_64);

    // Reserve space for unwind-info.
    Config.PostPrunePasses.push_back([CompactUnwindMgr](LinkGraph &G) {
      return CompactUnwindMgr->processAndReserveUnwindInfo(G);
    });

```
- **EN**: Implements logic around `push_back`, `createDefineExternalSectionStartAndEndSymbolsPass`, `processAndReserveUnwindInfo`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `push_back`, `createDefineExternalSectionStartAndEndSymbolsPass`, `processAndReserveUnwindInfo` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 581-592
```cpp
    // Translate compact-unwind to unwind-info.
    Config.PreFixupPasses.push_back([CompactUnwindMgr](LinkGraph &G) {
      return CompactUnwindMgr->writeUnwindInfo(G);
    });

    // Add GOT/Stubs optimizer pass.
    Config.PreFixupPasses.push_back(x86_64::optimizeGOTAndStubAccesses);
  }

  if (auto Err = Ctx->modifyPassConfig(*G, Config))
    return Ctx->notifyFailed(std::move(Err));

```
- **EN**: Implements logic around `push_back`, `writeUnwindInfo`, `notifyFailed`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `push_back`, `writeUnwindInfo`, `notifyFailed` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 593-606
```cpp
  // Construct a JITLinker and run the link function.
  MachOJITLinker_x86_64::link(std::move(Ctx), std::move(G), std::move(Config));
}

LinkGraphPassFunction createEHFrameSplitterPass_MachO_x86_64() {
  return DWARFRecordSectionSplitter(orc::MachOEHFrameSectionName);
}

LinkGraphPassFunction createEHFrameEdgeFixerPass_MachO_x86_64() {
  return EHFrameEdgeFixer(orc::MachOEHFrameSectionName, x86_64::PointerSize,
                          x86_64::Pointer32, x86_64::Pointer64, x86_64::Delta32,
                          x86_64::Delta64, x86_64::NegDelta32);
}

```
- **EN**: Implements logic around `link`, `createEHFrameSplitterPass_MachO_x86_64`, `DWARFRecordSectionSplitter`, `createEHFrameEdgeFixerPass_MachO_x86_64`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `link`, `createEHFrameSplitterPass_MachO_x86_64`, `DWARFRecordSectionSplitter`, `createEHFrameEdgeFixerPass_MachO_x86_64`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 607-608
```cpp
} // end namespace jitlink
} // end namespace llvm
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

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/JITLink/MachO_x86_64.h`, `llvm/ExecutionEngine/JITLink/DWARFRecordSectionSplitter.h`, `llvm/ExecutionEngine/JITLink/x86_64.h`, `llvm/ExecutionEngine/Orc/Shared/MachOObjectFormat.h`, `CompactUnwindSupport.h`, `DefineExternalSectionStartAndEndSymbols.h`, `MachOLinkGraphBuilder.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine
