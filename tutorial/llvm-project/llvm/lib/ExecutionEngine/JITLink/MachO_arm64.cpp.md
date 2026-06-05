# MachO_arm64.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/JITLink/MachO_arm64.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements JIT linker implementation for MachO/arm64.
  - **CN**: 实现面向即时编译代码的 JITLink 图链接、重定位处理以及特定文件格式的链接图 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===---- MachO_arm64.cpp - JIT linker implementation for MachO/arm64 -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// MachO/arm64 jit-link implementation.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 13-26
```cpp
#include "llvm/ExecutionEngine/JITLink/MachO_arm64.h"
#include "llvm/ExecutionEngine/JITLink/DWARFRecordSectionSplitter.h"
#include "llvm/ExecutionEngine/JITLink/aarch64.h"
#include "llvm/ExecutionEngine/Orc/Shared/MachOObjectFormat.h"

#include "CompactUnwindSupport.h"
#include "DefineExternalSectionStartAndEndSymbols.h"
#include "MachOLinkGraphBuilder.h"

#define DEBUG_TYPE "jitlink"

using namespace llvm;
using namespace llvm::jitlink;

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/JITLink/MachO_arm64.h`, `llvm/ExecutionEngine/JITLink/DWARFRecordSectionSplitter.h`, `llvm/ExecutionEngine/JITLink/aarch64.h`, `llvm/ExecutionEngine/Orc/Shared/MachOObjectFormat.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/JITLink/MachO_arm64.h`, `llvm/ExecutionEngine/JITLink/DWARFRecordSectionSplitter.h`, `llvm/ExecutionEngine/JITLink/aarch64.h`, `llvm/ExecutionEngine/Orc/Shared/MachOObjectFormat.h`。

### Lines 27-50
```cpp
namespace {

class MachOLinkGraphBuilder_arm64 : public MachOLinkGraphBuilder {
public:
  MachOLinkGraphBuilder_arm64(const object::MachOObjectFile &Obj,
                              std::shared_ptr<orc::SymbolStringPool> SSP,
                              SubtargetFeatures Features)
      : MachOLinkGraphBuilder(Obj, std::move(SSP), getObjectTriple(Obj),
                              std::move(Features), aarch64::getEdgeKindName),
        NumSymbols(Obj.getSymtabLoadCommand().nsyms) {}

private:
  enum MachOARM64RelocationKind : Edge::Kind {
    MachOBranch26 = Edge::FirstRelocation,
    MachOPointer32,
    MachOPointer64,
    MachOPointer64Anon,
    MachOPointer64Authenticated,
    MachOPage21,
    MachOPageOffset12,
    MachOGOTPage21,
    MachOGOTPageOffset12,
    MachOTLVPage21,
    MachOTLVPageOffset12,
```
- **EN**: Introduces declarations for `MachOLinkGraphBuilder_arm64`, `MachOARM64RelocationKind`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `MachOLinkGraphBuilder_arm64`, `MachOARM64RelocationKind` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 51-71
```cpp
    MachOPointerToGOT,
    MachOPairedAddend,
    MachOLDRLiteral19,
    MachODelta32,
    MachODelta64,
    MachONegDelta32,
    MachONegDelta64,
  };

  static Triple getObjectTriple(const object::MachOObjectFile &Obj) {
    // Get the CPU sub-type from the header.
    // jitLink_MachO should already have validated that the buffer is big enough
    // to cover a mach_header64 so this is safe.
    uint32_t CPUSubType =
        *(const support::ulittle32_t *)(Obj.getData().data() + 8);
    CPUSubType &= ~MachO::CPU_SUBTYPE_MASK;
    if (CPUSubType == MachO::CPU_SUBTYPE_ARM64E)
      return Triple("arm64e-apple-darwin");
    return Triple("arm64-apple-darwin");
  }

```
- **EN**: Implements logic around `getObjectTriple`, `getData`, `Triple`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getObjectTriple`, `getData`, `Triple` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 72-95
```cpp
  static Expected<MachOARM64RelocationKind>
  getRelocationKind(const MachO::relocation_info &RI) {
    switch (RI.r_type) {
    case MachO::ARM64_RELOC_UNSIGNED:
      if (!RI.r_pcrel) {
        if (RI.r_length == 3)
          return RI.r_extern ? MachOPointer64 : MachOPointer64Anon;
        else if (RI.r_length == 2)
          return MachOPointer32;
      }
      break;
    case MachO::ARM64_RELOC_SUBTRACTOR:
      // SUBTRACTOR must be non-pc-rel, extern, with length 2 or 3.
      // Initially represent SUBTRACTOR relocations with 'Delta<W>'.
      // They may be turned into NegDelta<W> by parsePairRelocation.
      if (!RI.r_pcrel && RI.r_extern) {
        if (RI.r_length == 2)
          return MachODelta32;
        else if (RI.r_length == 3)
          return MachODelta64;
      }
      break;
    case MachO::ARM64_RELOC_BRANCH26:
      if (RI.r_pcrel && RI.r_extern && RI.r_length == 2)
```
- **EN**: Implements logic around `getRelocationKind`; this block handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getRelocationKind` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 96-119
```cpp
        return MachOBranch26;
      break;
    case MachO::ARM64_RELOC_PAGE21:
      if (RI.r_pcrel && RI.r_extern && RI.r_length == 2)
        return MachOPage21;
      break;
    case MachO::ARM64_RELOC_PAGEOFF12:
      if (!RI.r_pcrel && RI.r_extern && RI.r_length == 2)
        return MachOPageOffset12;
      break;
    case MachO::ARM64_RELOC_GOT_LOAD_PAGE21:
      if (RI.r_pcrel && RI.r_extern && RI.r_length == 2)
        return MachOGOTPage21;
      break;
    case MachO::ARM64_RELOC_GOT_LOAD_PAGEOFF12:
      if (!RI.r_pcrel && RI.r_extern && RI.r_length == 2)
        return MachOGOTPageOffset12;
      break;
    case MachO::ARM64_RELOC_POINTER_TO_GOT:
      if (RI.r_pcrel && RI.r_extern && RI.r_length == 2)
        return MachOPointerToGOT;
      break;
    case MachO::ARM64_RELOC_ADDEND:
      if (!RI.r_pcrel && !RI.r_extern && RI.r_length == 2)
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 120-135
```cpp
        return MachOPairedAddend;
      break;
    case MachO::ARM64_RELOC_AUTHENTICATED_POINTER:
      if (!RI.r_pcrel && RI.r_extern && RI.r_length == 3)
        return MachOPointer64Authenticated;
      break;
    case MachO::ARM64_RELOC_TLVP_LOAD_PAGE21:
      if (RI.r_pcrel && RI.r_extern && RI.r_length == 2)
        return MachOTLVPage21;
      break;
    case MachO::ARM64_RELOC_TLVP_LOAD_PAGEOFF12:
      if (!RI.r_pcrel && RI.r_extern && RI.r_length == 2)
        return MachOTLVPageOffset12;
      break;
    }

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 136-147
```cpp
    return make_error<JITLinkError>(
        "Unsupported arm64 relocation: address=" +
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

### Lines 148-163
```cpp
  // Parses paired SUBTRACTOR/UNSIGNED relocations and, on success,
  // returns the edge kind and addend to be used.
  Expected<PairRelocInfo>
  parsePairRelocation(Block &BlockToFix, Edge::Kind SubtractorKind,
                      const MachO::relocation_info &SubRI,
                      orc::ExecutorAddr FixupAddress, const char *FixupContent,
                      object::relocation_iterator &UnsignedRelItr,
                      object::relocation_iterator &RelEnd) {
    using namespace support;

    assert(((SubtractorKind == MachODelta32 && SubRI.r_length == 2) ||
            (SubtractorKind == MachODelta64 && SubRI.r_length == 3)) &&
           "Subtractor kind should match length");
    assert(SubRI.r_extern && "SUBTRACTOR reloc symbol should be extern");
    assert(!SubRI.r_pcrel && "SUBTRACTOR reloc should not be PCRel");

```
- **EN**: Introduces declarations for `support`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `support` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 164-177
```cpp
    if (UnsignedRelItr == RelEnd)
      return make_error<JITLinkError>("arm64 SUBTRACTOR without paired "
                                      "UNSIGNED relocation");

    auto UnsignedRI = getRelocationInfo(UnsignedRelItr);

    if (SubRI.r_address != UnsignedRI.r_address)
      return make_error<JITLinkError>("arm64 SUBTRACTOR and paired UNSIGNED "
                                      "point to different addresses");

    if (SubRI.r_length != UnsignedRI.r_length)
      return make_error<JITLinkError>("length of arm64 SUBTRACTOR and paired "
                                      "UNSIGNED reloc must match");

```
- **EN**: Implements logic around `make_error<JITLinkError>`, `getRelocationInfo`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `getRelocationInfo` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 178-190
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

### Lines 191-208
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

### Lines 209-232
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

### Lines 233-248
```cpp
                                        "of their alt-entry groups)");
      }
    }

    if (FixingFromSymbol) {
      TargetSymbol = ToSymbol;
      DeltaKind = (SubRI.r_length == 3) ? aarch64::Delta64 : aarch64::Delta32;
      Addend = FixupValue + (FixupAddress - FromSymbol->getAddress());
      // FIXME: handle extern 'from'.
    } else {
      TargetSymbol = &*FromSymbol;
      DeltaKind =
          (SubRI.r_length == 3) ? aarch64::NegDelta64 : aarch64::NegDelta32;
      Addend = FixupValue - (FixupAddress - ToSymbol->getAddress());
    }

```
- **EN**: Implements logic around `getAddress`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `getAddress` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 249-261
```cpp
    return PairRelocInfo(DeltaKind, TargetSymbol, Addend);
  }

  Error addRelocations() override {
    using namespace support;
    auto &Obj = getObject();

    LLVM_DEBUG(dbgs() << "Processing relocations:\n");

    for (auto &S : Obj.sections()) {

      orc::ExecutorAddr SectionAddress(S.getAddress());

```
- **EN**: Introduces declarations for `support`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `support` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 262-274
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

### Lines 275-287
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

### Lines 288-305
```cpp
      for (auto RelItr = S.relocation_begin(), RelEnd = S.relocation_end();
           RelItr != RelEnd; ++RelItr) {

        MachO::relocation_info RI = getRelocationInfo(RelItr);

        // Validate the relocation kind.
        auto MachORelocKind = getRelocationKind(RI);
        if (!MachORelocKind)
          return MachORelocKind.takeError();

        // Find the address of the value to fix up.
        orc::ExecutorAddr FixupAddress =
            SectionAddress + (uint32_t)RI.r_address;
        LLVM_DEBUG({
          dbgs() << "  " << NSec->SectName << " + "
                 << formatv("{0:x8}", RI.r_address) << ":\n";
        });

```
- **EN**: Implements logic around `getRelocationInfo`, `getRelocationKind`, `takeError`, `dbgs`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getRelocationInfo`, `getRelocationKind`, `takeError`, `dbgs`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 306-319
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
              "Relocation content extends past end of fixup block");

```
- **EN**: Implements logic around `findSymbolByAddress`, `takeError`, `getBlock`, `getAddress`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `findSymbolByAddress`, `takeError`, `getBlock`, `getAddress`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 320-333
```cpp
        Edge::Kind Kind = Edge::Invalid;

        // Get a pointer to the fixup content.
        const char *FixupContent = BlockToFix->getContent().data() +
                                   (FixupAddress - BlockToFix->getAddress());

        // The target symbol and addend will be populated by the switch below.
        Symbol *TargetSymbol = nullptr;
        uint64_t Addend = 0;

        if (*MachORelocKind == MachOPairedAddend) {
          // If this is an Addend relocation then process it and move to the
          // paired reloc.

```
- **EN**: Implements logic around `getContent`, `getAddress`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `getContent`, `getAddress` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 334-345
```cpp
          Addend = SignExtend64(RI.r_symbolnum, 24);

          ++RelItr;
          if (RelItr == RelEnd)
            return make_error<JITLinkError>("Unpaired Addend reloc at " +
                                            formatv("{0:x16}", FixupAddress));
          RI = getRelocationInfo(RelItr);

          MachORelocKind = getRelocationKind(RI);
          if (!MachORelocKind)
            return MachORelocKind.takeError();

```
- **EN**: Implements logic around `SignExtend64`, `make_error<JITLinkError>`, `formatv`, `getRelocationInfo`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `SignExtend64`, `make_error<JITLinkError>`, `formatv`, `getRelocationInfo`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 346-358
```cpp
          if (*MachORelocKind != MachOBranch26 &&
              *MachORelocKind != MachOPage21 &&
              *MachORelocKind != MachOPageOffset12)
            return make_error<JITLinkError>(
                "Invalid relocation pair: Addend + " +
                StringRef(getMachOARM64RelocationKindName(*MachORelocKind)));

          LLVM_DEBUG({
            dbgs() << "    Addend: value = " << formatv("{0:x6}", Addend)
                   << ", pair is "
                   << getMachOARM64RelocationKindName(*MachORelocKind) << "\n";
          });

```
- **EN**: Implements logic around `make_error<JITLinkError>`, `StringRef`, `dbgs`, `getMachOARM64RelocationKindName`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `StringRef`, `dbgs`, `getMachOARM64RelocationKindName` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 359-382
```cpp
          // Find the address of the value to fix up.
          orc::ExecutorAddr PairedFixupAddress =
              SectionAddress + (uint32_t)RI.r_address;
          if (PairedFixupAddress != FixupAddress)
            return make_error<JITLinkError>("Paired relocation points at "
                                            "different target");
        }

        switch (*MachORelocKind) {
        case MachOBranch26: {
          if (auto TargetSymbolOrErr = findSymbolByIndex(RI.r_symbolnum))
            TargetSymbol = TargetSymbolOrErr->GraphSymbol;
          else
            return TargetSymbolOrErr.takeError();
          uint32_t Instr = *(const ulittle32_t *)FixupContent;
          if ((Instr & 0x7fffffff) != 0x14000000)
            return make_error<JITLinkError>("BRANCH26 target is not a B or BL "
                                            "instruction with a zero addend");
          Kind = aarch64::Branch26PCRel;
          break;
        }
        case MachOPointer32:
          if (auto TargetSymbolOrErr = findSymbolByIndex(RI.r_symbolnum))
            TargetSymbol = TargetSymbolOrErr->GraphSymbol;
```
- **EN**: Implements logic around `make_error<JITLinkError>`, `takeError`; this block handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `takeError` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 383-406
```cpp
          else
            return TargetSymbolOrErr.takeError();
          Addend = *(const ulittle32_t *)FixupContent;
          Kind = aarch64::Pointer32;
          break;
        case MachOPointer64:
        case MachOPointer64Authenticated:
          if (auto TargetSymbolOrErr = findSymbolByIndex(RI.r_symbolnum))
            TargetSymbol = TargetSymbolOrErr->GraphSymbol;
          else
            return TargetSymbolOrErr.takeError();
          Addend = *(const ulittle64_t *)FixupContent;
          Kind = *MachORelocKind == MachOPointer64
                     ? aarch64::Pointer64
                     : aarch64::Pointer64Authenticated;
          break;
        case MachOPointer64Anon: {
          orc::ExecutorAddr TargetAddress(*(const ulittle64_t *)FixupContent);
          auto TargetNSec = findSectionByIndex(RI.r_symbolnum - 1);
          if (!TargetNSec)
            return TargetNSec.takeError();
          if (auto TargetSymbolOrErr =
                  findSymbolByAddress(*TargetNSec, TargetAddress))
            TargetSymbol = &*TargetSymbolOrErr;
```
- **EN**: Implements logic around `takeError`, `TargetAddress`, `findSectionByIndex`, `findSymbolByAddress`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `takeError`, `TargetAddress`, `findSectionByIndex`, `findSymbolByAddress` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 407-425
```cpp
          else
            return TargetSymbolOrErr.takeError();
          Addend = TargetAddress - TargetSymbol->getAddress();
          Kind = aarch64::Pointer64;
          break;
        }
        case MachOPage21:
        case MachOGOTPage21:
        case MachOTLVPage21: {
          if (auto TargetSymbolOrErr = findSymbolByIndex(RI.r_symbolnum))
            TargetSymbol = TargetSymbolOrErr->GraphSymbol;
          else
            return TargetSymbolOrErr.takeError();
          uint32_t Instr = *(const ulittle32_t *)FixupContent;
          if ((Instr & 0xffffffe0) != 0x90000000)
            return make_error<JITLinkError>("PAGE21/GOTPAGE21 target is not an "
                                            "ADRP instruction with a zero "
                                            "addend");

```
- **EN**: Implements logic around `takeError`, `getAddress`, `make_error<JITLinkError>`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `takeError`, `getAddress`, `make_error<JITLinkError>` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 426-449
```cpp
          if (*MachORelocKind == MachOPage21) {
            Kind = aarch64::Page21;
          } else if (*MachORelocKind == MachOGOTPage21) {
            Kind = aarch64::RequestGOTAndTransformToPage21;
          } else if (*MachORelocKind == MachOTLVPage21) {
            Kind = aarch64::RequestTLVPAndTransformToPage21;
          }
          break;
        }
        case MachOPageOffset12: {
          if (auto TargetSymbolOrErr = findSymbolByIndex(RI.r_symbolnum))
            TargetSymbol = TargetSymbolOrErr->GraphSymbol;
          else
            return TargetSymbolOrErr.takeError();
          uint32_t Instr = *(const ulittle32_t *)FixupContent;
          uint32_t EncodedAddend = (Instr & 0x003FFC00) >> 10;
          if (EncodedAddend != 0)
            return make_error<JITLinkError>("GOTPAGEOFF12 target has non-zero "
                                            "encoded addend");
          Kind = aarch64::PageOffset12;
          break;
        }
        case MachOGOTPageOffset12:
        case MachOTLVPageOffset12: {
```
- **EN**: Implements logic around `takeError`, `make_error<JITLinkError>`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `takeError`, `make_error<JITLinkError>` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 450-472
```cpp
          if (auto TargetSymbolOrErr = findSymbolByIndex(RI.r_symbolnum))
            TargetSymbol = TargetSymbolOrErr->GraphSymbol;
          else
            return TargetSymbolOrErr.takeError();
          uint32_t Instr = *(const ulittle32_t *)FixupContent;
          if ((Instr & 0xfffffc00) != 0xf9400000)
            return make_error<JITLinkError>("GOTPAGEOFF12 target is not an LDR "
                                            "immediate instruction with a zero "
                                            "addend");

          if (*MachORelocKind == MachOGOTPageOffset12) {
            Kind = aarch64::RequestGOTAndTransformToPageOffset12;
          } else if (*MachORelocKind == MachOTLVPageOffset12) {
            Kind = aarch64::RequestTLVPAndTransformToPageOffset12;
          }
          break;
        }
        case MachOPointerToGOT:
          if (auto TargetSymbolOrErr = findSymbolByIndex(RI.r_symbolnum))
            TargetSymbol = TargetSymbolOrErr->GraphSymbol;
          else
            return TargetSymbolOrErr.takeError();

```
- **EN**: Implements logic around `takeError`, `make_error<JITLinkError>`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `takeError`, `make_error<JITLinkError>` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 473-495
```cpp
          Kind = aarch64::RequestGOTAndTransformToDelta32;
          break;
        case MachODelta32:
        case MachODelta64: {
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
        default:
          llvm_unreachable("Special relocation kind should not appear in "
                           "mach-o file");
        }

```
- **EN**: Implements logic around `parsePairRelocation`, `takeError`, `tie`, `assert`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `parsePairRelocation`, `takeError`, `tie`, `assert`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 496-509
```cpp
        LLVM_DEBUG({
          dbgs() << "    ";
          Edge GE(Kind, FixupAddress - BlockToFix->getAddress(), *TargetSymbol,
                  Addend);
          printEdge(dbgs(), *BlockToFix, GE, aarch64::getEdgeKindName(Kind));
          dbgs() << "\n";
        });
        BlockToFix->addEdge(Kind, FixupAddress - BlockToFix->getAddress(),
                            *TargetSymbol, Addend);
      }
    }
    return Error::success();
  }

```
- **EN**: Implements logic around `dbgs`, `GE`, `printEdge`, `addEdge`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `dbgs`, `GE`, `printEdge`, `addEdge`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 510-533
```cpp
  /// Return the string name of the given MachO arm64 edge kind.
  const char *getMachOARM64RelocationKindName(Edge::Kind R) {
    switch (R) {
    case MachOBranch26:
      return "MachOBranch26";
    case MachOPointer64:
      return "MachOPointer64";
    case MachOPointer64Anon:
      return "MachOPointer64Anon";
    case MachOPointer64Authenticated:
      return "MachOPointer64Authenticated";
    case MachOPage21:
      return "MachOPage21";
    case MachOPageOffset12:
      return "MachOPageOffset12";
    case MachOGOTPage21:
      return "MachOGOTPage21";
    case MachOGOTPageOffset12:
      return "MachOGOTPageOffset12";
    case MachOTLVPage21:
      return "MachOTLVPage21";
    case MachOTLVPageOffset12:
      return "MachOTLVPageOffset12";
    case MachOPointerToGOT:
```
- **EN**: Implements logic around `getMachOARM64RelocationKindName`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getMachOARM64RelocationKindName` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 534-551
```cpp
      return "MachOPointerToGOT";
    case MachOPairedAddend:
      return "MachOPairedAddend";
    case MachOLDRLiteral19:
      return "MachOLDRLiteral19";
    case MachODelta32:
      return "MachODelta32";
    case MachODelta64:
      return "MachODelta64";
    case MachONegDelta32:
      return "MachONegDelta32";
    case MachONegDelta64:
      return "MachONegDelta64";
    default:
      return getGenericEdgeKindName(R);
    }
  }

```
- **EN**: Implements logic around `getGenericEdgeKindName`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getGenericEdgeKindName` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 552-568
```cpp
  unsigned NumSymbols = 0;
};

} // namespace

namespace llvm {
namespace jitlink {

Error buildTables_MachO_arm64(LinkGraph &G) {
  LLVM_DEBUG(dbgs() << "Visiting edges in graph:\n");

  aarch64::GOTTableManager GOT(G);
  aarch64::PLTTableManager PLT(G, GOT);
  visitExistingEdges(G, GOT, PLT);
  return Error::success();
}

```
- **EN**: Introduces declarations for `llvm`, `jitlink`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `jitlink` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 569-582
```cpp
class MachOJITLinker_arm64 : public JITLinker<MachOJITLinker_arm64> {
  friend class JITLinker<MachOJITLinker_arm64>;

public:
  MachOJITLinker_arm64(std::unique_ptr<JITLinkContext> Ctx,
                       std::unique_ptr<LinkGraph> G,
                       PassConfiguration PassConfig)
      : JITLinker(std::move(Ctx), std::move(G), std::move(PassConfig)) {}

private:
  Error applyFixup(LinkGraph &G, Block &B, const Edge &E) const {
    return aarch64::applyFixup(G, B, E, nullptr);
  }

```
- **EN**: Introduces declarations for `MachOJITLinker_arm64`, `JITLinker`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `MachOJITLinker_arm64`, `JITLinker` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 583-595
```cpp
  uint64_t NullValue = 0;
};

Expected<std::unique_ptr<LinkGraph>> createLinkGraphFromMachOObject_arm64(
    MemoryBufferRef ObjectBuffer, std::shared_ptr<orc::SymbolStringPool> SSP) {
  auto MachOObj = object::ObjectFile::createMachOObjectFile(ObjectBuffer);
  if (!MachOObj)
    return MachOObj.takeError();

  auto Features = (*MachOObj)->getFeatures();
  if (!Features)
    return Features.takeError();

```
- **EN**: Implements logic around `createLinkGraphFromMachOObject_arm64`, `createMachOObjectFile`, `takeError`, `getFeatures`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `createLinkGraphFromMachOObject_arm64`, `createMachOObjectFile`, `takeError`, `getFeatures` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 596-608
```cpp
  return MachOLinkGraphBuilder_arm64(**MachOObj, std::move(SSP),
                                     std::move(*Features))
      .buildGraph();
}

static Error applyPACSigningToModInitPointers(LinkGraph &G) {
  assert(G.getTargetTriple().isArm64e() && "PAC signing only valid for arm64e");

  if (auto *ModInitSec = G.findSectionByName("__DATA,__mod_init_func")) {
    for (auto *B : ModInitSec->blocks()) {
      for (auto &E : B->edges()) {
        if (E.getKind() == aarch64::Pointer64) {

```
- **EN**: Implements logic around `MachOLinkGraphBuilder_arm64`, `move`, `buildGraph`, `applyPACSigningToModInitPointers`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `MachOLinkGraphBuilder_arm64`, `move`, `buildGraph`, `applyPACSigningToModInitPointers`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 609-625
```cpp
          // Check that we have room to encode pointer signing bits.
          if (E.getAddend() >> 32)
            return make_error<JITLinkError>(
                "In " + G.getName() + ", __mod_init_func pointer at " +
                formatv("{0:x}", B->getFixupAddress(E).getValue()) +
                " has data in high bits of addend (addend >= 2^32)");

          // Change edge to Pointer64Authenticated, encode signing:
          // key = asia, discriminator = 0, diversity = 0.
          Edge::AddendT SigningBits = 0x1ULL << 63;
          E.setKind(aarch64::Pointer64Authenticated);
          E.setAddend(E.getAddend() | SigningBits);
        }
      }
    }
  }

```
- **EN**: Implements logic around `make_error<JITLinkError>`, `getName`, `formatv`, `addend`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `getName`, `formatv`, `addend`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 626-637
```cpp
  return Error::success();
}

struct CompactUnwindTraits_MachO_arm64
    : public CompactUnwindTraits<CompactUnwindTraits_MachO_arm64,
                                 /* PointerSize = */ 8> {
  // FIXME: Reinstate once we no longer need the MSVC workaround. See
  //        FIXME for CompactUnwindTraits in CompactUnwindSupport.h.
  // constexpr static size_t PointerSize = 8;

  constexpr static endianness Endianness = endianness::little;

```
- **EN**: Introduces declarations for `CompactUnwindTraits_MachO_arm64`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `CompactUnwindTraits_MachO_arm64` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 638-650
```cpp
  constexpr static uint32_t EncodingModeMask = 0x0f000000;
  constexpr static uint32_t DWARFSectionOffsetMask = 0x00ffffff;

  constexpr static uint32_t FramelessMode = 0x02000000;
  constexpr static uint32_t DWARFMode = 0x03000000;
  constexpr static uint32_t FrameMode = 0x04000000;

  using GOTManager = aarch64::GOTTableManager;

  static bool encodingSpecifiesDWARF(uint32_t Encoding) {
    return (Encoding & EncodingModeMask) == DWARFMode;
  }

```
- **EN**: Implements logic around `encodingSpecifiesDWARF`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `encodingSpecifiesDWARF` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 651-662
```cpp
  static bool encodingCanBeMerged(uint32_t Encoding) {
    switch (Encoding & EncodingModeMask) {
    case FramelessMode:
    case FrameMode:
      return true;
    case DWARFMode:
    default:
      return false;
    }
  }
};

```
- **EN**: Implements logic around `encodingCanBeMerged`; this block uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `encodingCanBeMerged` 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 663-674
```cpp
void link_MachO_arm64(std::unique_ptr<LinkGraph> G,
                      std::unique_ptr<JITLinkContext> Ctx) {

  PassConfiguration Config;

  if (Ctx->shouldAddDefaultTargetPasses(G->getTargetTriple())) {
    // Add a mark-live pass.
    if (auto MarkLive = Ctx->getMarkLivePass(G->getTargetTriple()))
      Config.PrePrunePasses.push_back(std::move(MarkLive));
    else
      Config.PrePrunePasses.push_back(markAllSymbolsLive);

```
- **EN**: Implements logic around `link_MachO_arm64`, `push_back`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `link_MachO_arm64`, `push_back` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 675-689
```cpp
    // Add eh-frame passes.
    Config.PrePrunePasses.push_back(createEHFrameSplitterPass_MachO_arm64());
    Config.PrePrunePasses.push_back(createEHFrameEdgeFixerPass_MachO_arm64());

    // Create a compact-unwind manager for use in passes below.
    auto CompactUnwindMgr =
        std::make_shared<CompactUnwindManager<CompactUnwindTraits_MachO_arm64>>(
            orc::MachOCompactUnwindSectionName, orc::MachOUnwindInfoSectionName,
            orc::MachOEHFrameSectionName);

    // Add compact unwind prepare pass.
    Config.PrePrunePasses.push_back([CompactUnwindMgr](LinkGraph &G) {
      return CompactUnwindMgr->prepareForPrune(G);
    });

```
- **EN**: Implements logic around `push_back`, `make_shared<CompactUnwindManager<CompactUnwindTraits_MachO_arm64>>`, `prepareForPrune`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `push_back`, `make_shared<CompactUnwindManager<CompactUnwindTraits_MachO_arm64>>`, `prepareForPrune` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 690-706
```cpp
    // Resolve any external section start / end symbols.
    Config.PostAllocationPasses.push_back(
        createDefineExternalSectionStartAndEndSymbolsPass(
            identifyMachOSectionStartAndEndSymbols));

    // Add an in-place GOT/Stubs pass.
    Config.PostPrunePasses.push_back(buildTables_MachO_arm64);

    // If this is an arm64e graph then add pointer signing passes.
    if (G->getTargetTriple().isArm64e()) {
      Config.PostPrunePasses.push_back(applyPACSigningToModInitPointers);
      Config.PostPrunePasses.push_back(
          aarch64::createEmptyPointerSigningFunction);
      Config.PreFixupPasses.push_back(
          aarch64::lowerPointer64AuthEdgesToSigningFunction);
    }

```
- **EN**: Implements logic around `push_back`, `createDefineExternalSectionStartAndEndSymbolsPass`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `push_back`, `createDefineExternalSectionStartAndEndSymbolsPass` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 707-720
```cpp
    // Reserve unwind-info space.
    Config.PostPrunePasses.push_back([CompactUnwindMgr](LinkGraph &G) {
      return CompactUnwindMgr->processAndReserveUnwindInfo(G);
    });

    // Translate compact-unwind to unwind-info.
    Config.PreFixupPasses.push_back([CompactUnwindMgr](LinkGraph &G) {
      return CompactUnwindMgr->writeUnwindInfo(G);
    });
  }

  if (auto Err = Ctx->modifyPassConfig(*G, Config))
    return Ctx->notifyFailed(std::move(Err));

```
- **EN**: Implements logic around `push_back`, `processAndReserveUnwindInfo`, `writeUnwindInfo`, `notifyFailed`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `push_back`, `processAndReserveUnwindInfo`, `writeUnwindInfo`, `notifyFailed` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 721-735
```cpp
  // Construct a JITLinker and run the link function.
  MachOJITLinker_arm64::link(std::move(Ctx), std::move(G), std::move(Config));
}

LinkGraphPassFunction createEHFrameSplitterPass_MachO_arm64() {
  return DWARFRecordSectionSplitter(orc::MachOEHFrameSectionName);
}

LinkGraphPassFunction createEHFrameEdgeFixerPass_MachO_arm64() {
  return EHFrameEdgeFixer(orc::MachOEHFrameSectionName, aarch64::PointerSize,
                          aarch64::Pointer32, aarch64::Pointer64,
                          aarch64::Delta32, aarch64::Delta64,
                          aarch64::NegDelta32);
}

```
- **EN**: Implements logic around `link`, `createEHFrameSplitterPass_MachO_arm64`, `DWARFRecordSectionSplitter`, `createEHFrameEdgeFixerPass_MachO_arm64`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `link`, `createEHFrameSplitterPass_MachO_arm64`, `DWARFRecordSectionSplitter`, `createEHFrameEdgeFixerPass_MachO_arm64`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 736-737
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

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/JITLink/MachO_arm64.h`, `llvm/ExecutionEngine/JITLink/DWARFRecordSectionSplitter.h`, `llvm/ExecutionEngine/JITLink/aarch64.h`, `llvm/ExecutionEngine/Orc/Shared/MachOObjectFormat.h`, `CompactUnwindSupport.h`, `DefineExternalSectionStartAndEndSymbols.h`, `MachOLinkGraphBuilder.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine
