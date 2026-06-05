# EHFrameSupport.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/JITLink/EHFrameSupport.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements JITLink eh-frame utils.
  - **CN**: 实现面向即时编译代码的 JITLink 图链接、重定位处理以及特定文件格式的链接图 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
//===-------- JITLink_EHFrameSupport.cpp - JITLink eh-frame utils ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "EHFrameSupportImpl.h"

#include "llvm/BinaryFormat/Dwarf.h"
#include "llvm/Config/config.h"
#include "llvm/ExecutionEngine/Orc/TargetProcess/RegisterEHFrames.h"
#include "llvm/Support/DynamicLibrary.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `EHFrameSupportImpl.h`, `llvm/BinaryFormat/Dwarf.h`, `llvm/Config/config.h`, `llvm/ExecutionEngine/Orc/TargetProcess/RegisterEHFrames.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `EHFrameSupportImpl.h`, `llvm/BinaryFormat/Dwarf.h`, `llvm/Config/config.h`, `llvm/ExecutionEngine/Orc/TargetProcess/RegisterEHFrames.h`。

### Lines 16-28
```cpp
#define DEBUG_TYPE "jitlink"

namespace llvm {
namespace jitlink {

EHFrameEdgeFixer::EHFrameEdgeFixer(StringRef EHFrameSectionName,
                                   unsigned PointerSize, Edge::Kind Pointer32,
                                   Edge::Kind Pointer64, Edge::Kind Delta32,
                                   Edge::Kind Delta64, Edge::Kind NegDelta32)
    : EHFrameSectionName(EHFrameSectionName), PointerSize(PointerSize),
      Pointer32(Pointer32), Pointer64(Pointer64), Delta32(Delta32),
      Delta64(Delta64), NegDelta32(NegDelta32) {}

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 29-44
```cpp
Error EHFrameEdgeFixer::operator()(LinkGraph &G) {
  auto *EHFrame = G.findSectionByName(EHFrameSectionName);

  if (!EHFrame) {
    LLVM_DEBUG({
      dbgs() << "EHFrameEdgeFixer: No " << EHFrameSectionName
             << " section in \"" << G.getName() << "\". Nothing to do.\n";
    });
    return Error::success();
  }

  // Check that we support the graph's pointer size.
  if (G.getPointerSize() != 4 && G.getPointerSize() != 8)
    return make_error<JITLinkError>(
        "EHFrameEdgeFixer only supports 32 and 64 bit targets");

```
- **EN**: Implements logic around `operator`, `findSectionByName`, `dbgs`, `getName`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `operator`, `findSectionByName`, `dbgs`, `getName`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 45-68
```cpp
  LLVM_DEBUG({
    dbgs() << "EHFrameEdgeFixer: Processing " << EHFrameSectionName << " in \""
           << G.getName() << "\"...\n";
  });

  ParseContext PC(G);

  // Build a map of all blocks and symbols in the text sections. We will use
  // these for finding / building edge targets when processing FDEs.
  for (auto &Sec : G.sections()) {
    // Just record the most-canonical symbol (for eh-frame purposes) at each
    // address.
    for (auto *Sym : Sec.symbols()) {
      auto &CurSym = PC.AddrToSym[Sym->getAddress()];
      if (!CurSym || (std::make_tuple(Sym->getLinkage(), Sym->getScope(),
                                      !Sym->hasName(), Sym->getName()) <
                      std::make_tuple(CurSym->getLinkage(), CurSym->getScope(),
                                      !CurSym->hasName(), CurSym->getName())))
        CurSym = Sym;
    }
    if (auto Err = PC.AddrToBlock.addBlocks(Sec.blocks(),
                                            BlockAddressMap::includeNonNull))
      return Err;
  }
```
- **EN**: Implements logic around `dbgs`, `getName`, `PC`, `getAddress`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `dbgs`, `getName`, `PC`, `getAddress`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 69-82
```cpp

  // Sort eh-frame blocks into address order to ensure we visit CIEs before
  // their child FDEs.
  std::vector<Block *> EHFrameBlocks;
  llvm::append_range(EHFrameBlocks, EHFrame->blocks());
  llvm::sort(EHFrameBlocks, [](const Block *LHS, const Block *RHS) {
    return LHS->getAddress() < RHS->getAddress();
  });

  // Loop over the blocks in address order.
  for (auto *B : EHFrameBlocks)
    if (auto Err = processBlock(PC, *B))
      return Err;

```
- **EN**: Implements logic around `append_range`, `sort`, `getAddress`; this block manipulates JITLink graph structures or link-time passes; executes or prepares runtime behavior for LLVM IR/JIT code; returns subsystem-specific computed results.
- **CN**: 围绕 `append_range`, `sort`, `getAddress` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，执行或准备 LLVM IR/JIT 代码的运行时行为，返回子系统相关的计算结果。

### Lines 83-96
```cpp
  return Error::success();
}

static Expected<size_t> readCFIRecordLength(const Block &B,
                                            BinaryStreamReader &R) {
  uint32_t Length;
  if (auto Err = R.readInteger(Length))
    return std::move(Err);

  // If Length < 0xffffffff then use the regular length field, otherwise
  // read the extended length field.
  if (Length != 0xffffffff)
    return Length;

```
- **EN**: Implements logic around `success`, `readCFIRecordLength`, `move`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `readCFIRecordLength`, `move` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 97-108
```cpp
  uint64_t ExtendedLength;
  if (auto Err = R.readInteger(ExtendedLength))
    return std::move(Err);

  if (ExtendedLength > std::numeric_limits<size_t>::max())
    return make_error<JITLinkError>(
        "In CFI record at " +
        formatv("{0:x}", B.getAddress() + R.getOffset() - 12) +
        ", extended length of " + formatv("{0:x}", ExtendedLength) +
        " exceeds address-range max (" +
        formatv("{0:x}", std::numeric_limits<size_t>::max()));

```
- **EN**: Implements logic around `move`, `make_error<JITLinkError>`, `formatv`, `max`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `move`, `make_error<JITLinkError>`, `formatv`, `max` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 109-120
```cpp
  return ExtendedLength;
}

Error EHFrameEdgeFixer::processBlock(ParseContext &PC, Block &B) {

  LLVM_DEBUG(dbgs() << "  Processing block at " << B.getAddress() << "\n");

  // eh-frame should not contain zero-fill blocks.
  if (B.isZeroFill())
    return make_error<JITLinkError>("Unexpected zero-fill block in " +
                                    EHFrameSectionName + " section");

```
- **EN**: Implements logic around `processBlock`, `make_error<JITLinkError>`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `processBlock`, `make_error<JITLinkError>` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 121-133
```cpp
  if (B.getSize() == 0) {
    LLVM_DEBUG(dbgs() << "    Block is empty. Skipping.\n");
    return Error::success();
  }

  // Find the offsets of any existing edges from this block.
  BlockEdgesInfo BlockEdges;
  for (auto &E : B.edges())
    if (E.isRelocation()) {
      // Check if we already saw more than one relocation at this offset.
      if (BlockEdges.Multiple.contains(E.getOffset()))
        continue;

```
- **EN**: Implements logic around `success`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `success` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 134-147
```cpp
      // Otherwise check if we previously had exactly one relocation at this
      // offset. If so, we now have a second one and move it from the TargetMap
      // into the Multiple set.
      auto [It, Inserted] = BlockEdges.TargetMap.try_emplace(E.getOffset(), E);
      if (!Inserted) {
        BlockEdges.TargetMap.erase(It);
        BlockEdges.Multiple.insert(E.getOffset());
      }
    }

  BinaryStreamReader BlockReader(
      StringRef(B.getContent().data(), B.getContent().size()),
      PC.G.getEndianness());

```
- **EN**: Implements logic around `try_emplace`, `erase`, `insert`, `BlockReader`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `try_emplace`, `erase`, `insert`, `BlockReader`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作。

### Lines 148-164
```cpp
  // Get the record length.
  Expected<size_t> RecordRemaining = readCFIRecordLength(B, BlockReader);
  if (!RecordRemaining)
    return RecordRemaining.takeError();

  // We expect DWARFRecordSectionSplitter to split each CFI record into its own
  // block.
  if (BlockReader.bytesRemaining() != *RecordRemaining)
    return make_error<JITLinkError>("Incomplete CFI record at " +
                                    formatv("{0:x16}", B.getAddress()));

  // Read the CIE delta for this record.
  uint64_t CIEDeltaFieldOffset = BlockReader.getOffset();
  uint32_t CIEDelta;
  if (auto Err = BlockReader.readInteger(CIEDelta))
    return Err;

```
- **EN**: Implements logic around `readCFIRecordLength`, `takeError`, `make_error<JITLinkError>`, `formatv`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `readCFIRecordLength`, `takeError`, `make_error<JITLinkError>`, `formatv`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 165-179
```cpp
  if (CIEDelta == 0) {
    if (auto Err = processCIE(PC, B, CIEDeltaFieldOffset, BlockEdges))
      return Err;
  } else {
    if (auto Err = processFDE(PC, B, CIEDeltaFieldOffset, CIEDelta, BlockEdges))
      return Err;
  }

  return Error::success();
}

Error EHFrameEdgeFixer::processCIE(ParseContext &PC, Block &B,
                                   size_t CIEDeltaFieldOffset,
                                   const BlockEdgesInfo &BlockEdges) {

```
- **EN**: Implements logic around `success`, `processCIE`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `processCIE` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 180-191
```cpp
  LLVM_DEBUG(dbgs() << "    Record is CIE\n");

  BinaryStreamReader RecordReader(
      StringRef(B.getContent().data(), B.getContent().size()),
      PC.G.getEndianness());

  // Skip past the CIE delta field: we've already processed this far.
  RecordReader.setOffset(CIEDeltaFieldOffset + 4);

  auto &CIESymbol = PC.G.addAnonymousSymbol(B, 0, B.getSize(), false, false);
  CIEInformation CIEInfo(CIESymbol);

```
- **EN**: Implements logic around `RecordReader`, `StringRef`, `getEndianness`, `setOffset`, and 2 more symbols.
- **CN**: 围绕 `RecordReader`, `StringRef`, `getEndianness`, `setOffset`, and 2 more symbols 实现具体逻辑。

### Lines 192-203
```cpp
  uint8_t Version = 0;
  if (auto Err = RecordReader.readInteger(Version))
    return Err;

  if (Version != 0x01)
    return make_error<JITLinkError>("Bad CIE version " + Twine(Version) +
                                    " (should be 0x01) in eh-frame");

  auto AugInfo = parseAugmentationString(RecordReader);
  if (!AugInfo)
    return AugInfo.takeError();

```
- **EN**: Implements logic around `make_error<JITLinkError>`, `parseAugmentationString`, `takeError`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `parseAugmentationString`, `takeError` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 204-215
```cpp
  // Skip the EH Data field if present.
  if (AugInfo->EHDataFieldPresent)
    if (auto Err = RecordReader.skip(PC.G.getPointerSize()))
      return Err;

  // Read and validate the code alignment factor.
  {
    uint64_t CodeAlignmentFactor = 0;
    if (auto Err = RecordReader.readULEB128(CodeAlignmentFactor))
      return Err;
  }

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 216-228
```cpp
  // Read and validate the data alignment factor.
  {
    int64_t DataAlignmentFactor = 0;
    if (auto Err = RecordReader.readSLEB128(DataAlignmentFactor))
      return Err;
  }

  // Skip the return address register field.
  if (auto Err = RecordReader.skip(1))
    return Err;

  if (AugInfo->AugmentationDataPresent) {

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 229-252
```cpp
    CIEInfo.AugmentationDataPresent = true;

    uint64_t AugmentationDataLength = 0;
    if (auto Err = RecordReader.readULEB128(AugmentationDataLength))
      return Err;

    uint32_t AugmentationDataStartOffset = RecordReader.getOffset();

    uint8_t *NextField = &AugInfo->Fields[0];
    while (uint8_t Field = *NextField++) {
      switch (Field) {
      case 'L':
        CIEInfo.LSDAPresent = true;
        if (auto PE = readPointerEncoding(RecordReader, B, "LSDA"))
          CIEInfo.LSDAEncoding = *PE;
        else
          return PE.takeError();
        break;
      case 'P': {
        auto PersonalityPointerEncoding =
            readPointerEncoding(RecordReader, B, "personality");
        if (!PersonalityPointerEncoding)
          return PersonalityPointerEncoding.takeError();
        if (auto Err =
```
- **EN**: Implements logic around `getOffset`, `takeError`, `readPointerEncoding`; this block uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getOffset`, `takeError`, `readPointerEncoding` 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 253-274
```cpp
                getOrCreateEncodedPointerEdge(
                    PC, BlockEdges, *PersonalityPointerEncoding, RecordReader,
                    B, RecordReader.getOffset(), "personality")
                    .takeError())
          return Err;
        break;
      }
      case 'R':
        if (auto PE = readPointerEncoding(RecordReader, B, "address")) {
          CIEInfo.AddressEncoding = *PE;
          if (CIEInfo.AddressEncoding == dwarf::DW_EH_PE_omit)
            return make_error<JITLinkError>(
                "Invalid address encoding DW_EH_PE_omit in CIE at " +
                formatv("{0:x}", B.getAddress().getValue()));
        } else
          return PE.takeError();
        break;
      default:
        llvm_unreachable("Invalid augmentation string field");
      }
    }

```
- **EN**: Implements logic around `getOrCreateEncodedPointerEdge`, `getOffset`, `takeError`, `make_error<JITLinkError>`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `getOrCreateEncodedPointerEdge`, `getOffset`, `takeError`, `make_error<JITLinkError>`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 275-287
```cpp
    if (RecordReader.getOffset() - AugmentationDataStartOffset >
        AugmentationDataLength)
      return make_error<JITLinkError>("Read past the end of the augmentation "
                                      "data while parsing fields");
  }

  assert(!PC.CIEInfos.count(CIESymbol.getAddress()) &&
         "Multiple CIEs recorded at the same address?");
  PC.CIEInfos[CIESymbol.getAddress()] = std::move(CIEInfo);

  return Error::success();
}

```
- **EN**: Implements logic around `make_error<JITLinkError>`, `assert`, `getAddress`, `success`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `assert`, `getAddress`, `success` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 288-299
```cpp
Error EHFrameEdgeFixer::processFDE(ParseContext &PC, Block &B,
                                   size_t CIEDeltaFieldOffset,
                                   uint32_t CIEDelta,
                                   const BlockEdgesInfo &BlockEdges) {
  LLVM_DEBUG(dbgs() << "    Record is FDE\n");

  orc::ExecutorAddr RecordAddress = B.getAddress();

  BinaryStreamReader RecordReader(
      StringRef(B.getContent().data(), B.getContent().size()),
      PC.G.getEndianness());

```
- **EN**: Implements logic around `processFDE`, `getAddress`, `RecordReader`, `StringRef`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `processFDE`, `getAddress`, `RecordReader`, `StringRef`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 300-313
```cpp
  // Skip past the CIE delta field: we've already read this far.
  RecordReader.setOffset(CIEDeltaFieldOffset + 4);

  auto &FDESymbol = PC.G.addAnonymousSymbol(B, 0, B.getSize(), false, false);

  CIEInformation *CIEInfo = nullptr;

  {
    // Process the CIE pointer field.
    if (BlockEdges.Multiple.contains(CIEDeltaFieldOffset))
      return make_error<JITLinkError>(
          "CIE pointer field already has multiple edges at " +
          formatv("{0:x16}", RecordAddress + CIEDeltaFieldOffset));

```
- **EN**: Implements logic around `setOffset`, `addAnonymousSymbol`, `make_error<JITLinkError>`, `formatv`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `setOffset`, `addAnonymousSymbol`, `make_error<JITLinkError>`, `formatv` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 314-337
```cpp
    auto CIEEdgeItr = BlockEdges.TargetMap.find(CIEDeltaFieldOffset);

    orc::ExecutorAddr CIEAddress =
        RecordAddress + orc::ExecutorAddrDiff(CIEDeltaFieldOffset) -
        orc::ExecutorAddrDiff(CIEDelta);
    if (CIEEdgeItr == BlockEdges.TargetMap.end()) {
      LLVM_DEBUG({
        dbgs() << "        Adding edge at "
               << (RecordAddress + CIEDeltaFieldOffset)
               << " to CIE at: " << CIEAddress << "\n";
      });
      if (auto CIEInfoOrErr = PC.findCIEInfo(CIEAddress))
        CIEInfo = *CIEInfoOrErr;
      else
        return CIEInfoOrErr.takeError();
      assert(CIEInfo->CIESymbol && "CIEInfo has no CIE symbol set");
      B.addEdge(NegDelta32, CIEDeltaFieldOffset, *CIEInfo->CIESymbol, 0);
    } else {
      LLVM_DEBUG({
        dbgs() << "        Already has edge at "
               << (RecordAddress + CIEDeltaFieldOffset) << " to CIE at "
               << CIEAddress << "\n";
      });
      auto &EI = CIEEdgeItr->second;
```
- **EN**: Implements logic around `find`, `ExecutorAddrDiff`, `dbgs`, `takeError`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `find`, `ExecutorAddrDiff`, `dbgs`, `takeError`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 338-349
```cpp
      if (EI.Addend)
        return make_error<JITLinkError>(
            "CIE edge at " +
            formatv("{0:x16}", RecordAddress + CIEDeltaFieldOffset) +
            " has non-zero addend");
      if (auto CIEInfoOrErr = PC.findCIEInfo(EI.Target->getAddress()))
        CIEInfo = *CIEInfoOrErr;
      else
        return CIEInfoOrErr.takeError();
    }
  }

```
- **EN**: Implements logic around `make_error<JITLinkError>`, `formatv`, `takeError`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `formatv`, `takeError` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 350-373
```cpp
  // Process the PC-Begin field.
  LLVM_DEBUG({
    dbgs() << "      Processing PC-begin at "
           << (RecordAddress + RecordReader.getOffset()) << "\n";
  });
  if (auto PCBegin = getOrCreateEncodedPointerEdge(
          PC, BlockEdges, CIEInfo->AddressEncoding, RecordReader, B,
          RecordReader.getOffset(), "PC begin")) {
    assert(*PCBegin && "PC-begin symbol not set");
    if ((*PCBegin)->isDefined()) {
      // Add a keep-alive edge from the FDE target to the FDE to ensure that the
      // FDE is kept alive if its target is.
      LLVM_DEBUG({
        dbgs() << "        Adding keep-alive edge from target at "
               << (*PCBegin)->getBlock().getAddress() << " to FDE at "
               << RecordAddress << "\n";
      });
      (*PCBegin)->getBlock().addEdge(Edge::KeepAlive, 0, FDESymbol, 0);
    } else {
      LLVM_DEBUG({
        dbgs() << "        WARNING: Not adding keep-alive edge to FDE at "
               << RecordAddress << ", which points to "
               << ((*PCBegin)->isExternal() ? "external" : "absolute")
               << " symbol \"" << (*PCBegin)->getName()
```
- **EN**: Implements logic around `dbgs`, `getOffset`, `assert`, `getBlock`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `dbgs`, `getOffset`, `assert`, `getBlock`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 374-389
```cpp
               << "\" -- FDE must be kept alive manually or it will be "
               << "dead stripped.\n";
      });
    }
  } else
    return PCBegin.takeError();

  // Skip over the PC range size field.
  if (auto Err = skipEncodedPointer(CIEInfo->AddressEncoding, RecordReader))
    return Err;

  if (CIEInfo->AugmentationDataPresent) {
    uint64_t AugmentationDataSize;
    if (auto Err = RecordReader.readULEB128(AugmentationDataSize))
      return Err;

```
- **EN**: Implements logic around `takeError`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `takeError` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 390-402
```cpp
    if (CIEInfo->LSDAPresent)
      if (auto Err = getOrCreateEncodedPointerEdge(
                         PC, BlockEdges, CIEInfo->LSDAEncoding, RecordReader, B,
                         RecordReader.getOffset(), "LSDA")
                         .takeError())
        return Err;
  } else {
    LLVM_DEBUG(dbgs() << "        Record does not have LSDA field.\n");
  }

  return Error::success();
}

```
- **EN**: Implements logic around `getOffset`, `takeError`, `success`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `getOffset`, `takeError`, `success` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 403-426
```cpp
Expected<EHFrameEdgeFixer::AugmentationInfo>
EHFrameEdgeFixer::parseAugmentationString(BinaryStreamReader &RecordReader) {
  AugmentationInfo AugInfo;
  uint8_t NextChar;
  uint8_t *NextField = &AugInfo.Fields[0];

  if (auto Err = RecordReader.readInteger(NextChar))
    return std::move(Err);

  while (NextChar != 0) {
    switch (NextChar) {
    case 'z':
      AugInfo.AugmentationDataPresent = true;
      break;
    case 'e':
      if (auto Err = RecordReader.readInteger(NextChar))
        return std::move(Err);
      if (NextChar != 'h')
        return make_error<JITLinkError>("Unrecognized substring e" +
                                        Twine(NextChar) +
                                        " in augmentation string");
      AugInfo.EHDataFieldPresent = true;
      break;
    case 'L':
```
- **EN**: Implements logic around `parseAugmentationString`, `move`, `make_error<JITLinkError>`, `Twine`; this block uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `parseAugmentationString`, `move`, `make_error<JITLinkError>`, `Twine` 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 427-440
```cpp
    case 'P':
    case 'R':
      *NextField++ = NextChar;
      break;
    default:
      return make_error<JITLinkError>("Unrecognized character " +
                                      Twine(NextChar) +
                                      " in augmentation string");
    }

    if (auto Err = RecordReader.readInteger(NextChar))
      return std::move(Err);
  }

```
- **EN**: Implements logic around `make_error<JITLinkError>`, `Twine`, `move`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `Twine`, `move` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 441-452
```cpp
  return std::move(AugInfo);
}

Expected<uint8_t> EHFrameEdgeFixer::readPointerEncoding(BinaryStreamReader &R,
                                                        Block &InBlock,
                                                        const char *FieldName) {
  using namespace dwarf;

  uint8_t PointerEncoding;
  if (auto Err = R.readInteger(PointerEncoding))
    return std::move(Err);

```
- **EN**: Introduces declarations for `dwarf`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `dwarf` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 453-472
```cpp
  bool Supported = true;
  switch (PointerEncoding & 0xf) {
  case DW_EH_PE_uleb128:
  case DW_EH_PE_udata2:
  case DW_EH_PE_sleb128:
  case DW_EH_PE_sdata2:
    Supported = false;
    break;
  }
  if (Supported) {
    switch (PointerEncoding & 0x70) {
    case DW_EH_PE_textrel:
    case DW_EH_PE_datarel:
    case DW_EH_PE_funcrel:
    case DW_EH_PE_aligned:
      Supported = false;
      break;
    }
  }

```
- **EN**: Implements subsystem-specific case analysis using a `switch` over kinds, opcodes, or states.
- **CN**: 通过对 kind、opcode 或状态执行 `switch` 分析来实现子系统相关逻辑。

### Lines 473-485
```cpp
  if (Supported)
    return PointerEncoding;

  return make_error<JITLinkError>("Unsupported pointer encoding " +
                                  formatv("{0:x2}", PointerEncoding) + " for " +
                                  FieldName + "in CFI record at " +
                                  formatv("{0:x16}", InBlock.getAddress()));
}

Error EHFrameEdgeFixer::skipEncodedPointer(uint8_t PointerEncoding,
                                           BinaryStreamReader &RecordReader) {
  using namespace dwarf;

```
- **EN**: Introduces declarations for `dwarf`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `dwarf` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 486-506
```cpp
  // Switch absptr to corresponding udata encoding.
  if ((PointerEncoding & 0xf) == DW_EH_PE_absptr)
    PointerEncoding |= (PointerSize == 8) ? DW_EH_PE_udata8 : DW_EH_PE_udata4;

  switch (PointerEncoding & 0xf) {
  case DW_EH_PE_udata4:
  case DW_EH_PE_sdata4:
    if (auto Err = RecordReader.skip(4))
      return Err;
    break;
  case DW_EH_PE_udata8:
  case DW_EH_PE_sdata8:
    if (auto Err = RecordReader.skip(8))
      return Err;
    break;
  default:
    llvm_unreachable("Unrecognized encoding");
  }
  return Error::success();
}

```
- **EN**: Implements logic around `llvm_unreachable`, `success`; this block uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `llvm_unreachable`, `success` 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 507-530
```cpp
Expected<Symbol *> EHFrameEdgeFixer::getOrCreateEncodedPointerEdge(
    ParseContext &PC, const BlockEdgesInfo &BlockEdges, uint8_t PointerEncoding,
    BinaryStreamReader &RecordReader, Block &BlockToFix,
    size_t PointerFieldOffset, const char *FieldName) {
  using namespace dwarf;

  if (PointerEncoding == DW_EH_PE_omit)
    return nullptr;

  // If there's already an edge here then just skip the encoded pointer and
  // return the edge's target.
  {
    auto EdgeI = BlockEdges.TargetMap.find(PointerFieldOffset);
    if (EdgeI != BlockEdges.TargetMap.end()) {
      LLVM_DEBUG({
        dbgs() << "      Existing edge at "
               << (BlockToFix.getAddress() + PointerFieldOffset) << " to "
               << FieldName << " at " << EdgeI->second.Target->getAddress();
        if (EdgeI->second.Target->hasName())
          dbgs() << " (" << EdgeI->second.Target->getName() << ")";
        dbgs() << "\n";
      });
      if (auto Err = skipEncodedPointer(PointerEncoding, RecordReader))
        return std::move(Err);
```
- **EN**: Introduces declarations for `dwarf`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `dwarf` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 531-542
```cpp
      return EdgeI->second.Target;
    }

    if (BlockEdges.Multiple.contains(PointerFieldOffset))
      return make_error<JITLinkError>("Multiple relocations at offset " +
                                      formatv("{0:x16}", PointerFieldOffset));
  }

  // Switch absptr to corresponding udata encoding.
  if ((PointerEncoding & 0xf) == DW_EH_PE_absptr)
    PointerEncoding |= (PointerSize == 8) ? DW_EH_PE_udata8 : DW_EH_PE_udata4;

```
- **EN**: Implements logic around `make_error<JITLinkError>`, `formatv`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `formatv` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 543-566
```cpp
  // We need to create an edge. Start by reading the field value.
  uint64_t FieldValue;
  bool Is64Bit = false;
  switch (PointerEncoding & 0xf) {
  case DW_EH_PE_udata4: {
    uint32_t Val;
    if (auto Err = RecordReader.readInteger(Val))
      return std::move(Err);
    FieldValue = Val;
    break;
  }
  case DW_EH_PE_sdata4: {
    uint32_t Val;
    if (auto Err = RecordReader.readInteger(Val))
      return std::move(Err);
    FieldValue = Val;
    break;
  }
  case DW_EH_PE_udata8:
  case DW_EH_PE_sdata8:
    Is64Bit = true;
    if (auto Err = RecordReader.readInteger(FieldValue))
      return std::move(Err);
    break;
```
- **EN**: Implements logic around `move`; this block uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `move` 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 567-580
```cpp
  default:
    llvm_unreachable("Unsupported encoding");
  }

  // Find the edge target and edge kind to use.
  orc::ExecutorAddr Target;
  Edge::Kind PtrEdgeKind = Edge::Invalid;
  if ((PointerEncoding & 0x70) == DW_EH_PE_pcrel) {
    Target = BlockToFix.getAddress() + PointerFieldOffset;
    PtrEdgeKind = Is64Bit ? Delta64 : Delta32;
  } else
    PtrEdgeKind = Is64Bit ? Pointer64 : Pointer32;
  Target += FieldValue;

```
- **EN**: Implements logic around `llvm_unreachable`, `getAddress`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `llvm_unreachable`, `getAddress` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 581-595
```cpp
  // Find or create a symbol to point the edge at.
  auto TargetSym = getOrCreateSymbol(PC, Target);
  if (!TargetSym)
    return TargetSym.takeError();
  BlockToFix.addEdge(PtrEdgeKind, PointerFieldOffset, *TargetSym, 0);

  LLVM_DEBUG({
    dbgs() << "      Adding edge at "
           << (BlockToFix.getAddress() + PointerFieldOffset) << " to "
           << FieldName << " at " << TargetSym->getAddress();
    if (TargetSym->hasName())
      dbgs() << " (" << TargetSym->getName() << ")";
    dbgs() << "\n";
  });

```
- **EN**: Implements logic around `getOrCreateSymbol`, `takeError`, `addEdge`, `dbgs`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `getOrCreateSymbol`, `takeError`, `addEdge`, `dbgs`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 596-611
```cpp
  return &*TargetSym;
}

Expected<Symbol &> EHFrameEdgeFixer::getOrCreateSymbol(ParseContext &PC,
                                                       orc::ExecutorAddr Addr) {
  // See whether we have a canonical symbol for the given address already.
  auto CanonicalSymI = PC.AddrToSym.find(Addr);
  if (CanonicalSymI != PC.AddrToSym.end())
    return *CanonicalSymI->second;

  // Otherwise search for a block covering the address and create a new symbol.
  auto *B = PC.AddrToBlock.getBlockCovering(Addr);
  if (!B)
    return make_error<JITLinkError>("No symbol or block covering address " +
                                    formatv("{0:x16}", Addr));

```
- **EN**: Implements logic around `getOrCreateSymbol`, `find`, `getBlockCovering`, `make_error<JITLinkError>`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `getOrCreateSymbol`, `find`, `getBlockCovering`, `make_error<JITLinkError>`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 612-625
```cpp
  auto &S =
      PC.G.addAnonymousSymbol(*B, Addr - B->getAddress(), 0, false, false);
  PC.AddrToSym[S.getAddress()] = &S;
  return S;
}

char EHFrameNullTerminator::NullTerminatorBlockContent[4] = {0, 0, 0, 0};

EHFrameNullTerminator::EHFrameNullTerminator(StringRef EHFrameSectionName)
    : EHFrameSectionName(EHFrameSectionName) {}

Error EHFrameNullTerminator::operator()(LinkGraph &G) {
  auto *EHFrame = G.findSectionByName(EHFrameSectionName);

```
- **EN**: Implements logic around `addAnonymousSymbol`, `getAddress`, `EHFrameNullTerminator`, `EHFrameSectionName`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `addAnonymousSymbol`, `getAddress`, `EHFrameNullTerminator`, `EHFrameSectionName`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 626-640
```cpp
  if (!EHFrame)
    return Error::success();

  LLVM_DEBUG({
    dbgs() << "EHFrameNullTerminator adding null terminator to "
           << EHFrameSectionName << "\n";
  });

  auto &NullTerminatorBlock =
      G.createContentBlock(*EHFrame, NullTerminatorBlockContent,
                           orc::ExecutorAddr(~uint64_t(4)), 1, 0);
  G.addAnonymousSymbol(NullTerminatorBlock, 0, 4, false, true);
  return Error::success();
}

```
- **EN**: Implements logic around `success`, `dbgs`, `createContentBlock`, `ExecutorAddr`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `dbgs`, `createContentBlock`, `ExecutorAddr`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 641-655
```cpp
EHFrameCFIBlockInspector EHFrameCFIBlockInspector::FromEdgeScan(Block &B) {
  if (B.edges_empty())
    return EHFrameCFIBlockInspector(nullptr);
  if (B.edges_size() == 1)
    return EHFrameCFIBlockInspector(&*B.edges().begin());
  SmallVector<Edge *, 3> Es(llvm::make_pointer_range(B.edges()));
  assert(Es.size() >= 2 && Es.size() <= 3 && "Unexpected number of edges");
  llvm::sort(Es, [](const Edge *LHS, const Edge *RHS) {
    return LHS->getOffset() < RHS->getOffset();
  });
  return EHFrameCFIBlockInspector(*Es[0], *Es[1],
                                  Es.size() == 3 ? Es[2] : nullptr);
  return EHFrameCFIBlockInspector(nullptr);
}

```
- **EN**: Implements logic around `FromEdgeScan`, `EHFrameCFIBlockInspector`, `Es`, `assert`, and 3 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `FromEdgeScan`, `EHFrameCFIBlockInspector`, `Es`, `assert`, and 3 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 656-676
```cpp
EHFrameCFIBlockInspector::EHFrameCFIBlockInspector(Edge *PersonalityEdge)
    : PersonalityEdge(PersonalityEdge) {}

EHFrameCFIBlockInspector::EHFrameCFIBlockInspector(Edge &CIEEdge,
                                                   Edge &PCBeginEdge,
                                                   Edge *LSDAEdge)
    : CIEEdge(&CIEEdge), PCBeginEdge(&PCBeginEdge), LSDAEdge(LSDAEdge) {}

Section *getEHFrameSection(LinkGraph &G) {
  const char *EHFrameSectionName = nullptr;
  switch (G.getTargetTriple().getObjectFormat()) {
  case Triple::MachO:
    EHFrameSectionName = "__TEXT,__eh_frame";
    break;
  case Triple::ELF:
    EHFrameSectionName = ".eh_frame";
    break;
  default:
    return nullptr;
  }

```
- **EN**: Implements logic around `EHFrameCFIBlockInspector`, `PersonalityEdge`, `CIEEdge`, `getEHFrameSection`; this block manipulates JITLink graph structures or link-time passes; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `EHFrameCFIBlockInspector`, `PersonalityEdge`, `CIEEdge`, `getEHFrameSection` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 677-685
```cpp
  if (auto *S = G.findSectionByName(EHFrameSectionName))
    if (!S->empty())
      return S;

  return nullptr;
}

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

- **Direct includes / 直接包含**: `EHFrameSupportImpl.h`, `llvm/BinaryFormat/Dwarf.h`, `llvm/Config/config.h`, `llvm/ExecutionEngine/Orc/TargetProcess/RegisterEHFrames.h`, `llvm/Support/DynamicLibrary.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Support, BinaryFormat
