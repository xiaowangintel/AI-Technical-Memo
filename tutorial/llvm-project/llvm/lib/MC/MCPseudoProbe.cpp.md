# MCPseudoProbe.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/MCPseudoProbe.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Pseudo probe encoding support.
  - **CN**: 实现该文件在 LLVM 中对应的子系统逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24
```cpp
//===- lib/MC/MCPseudoProbe.cpp - Pseudo probe encoding support ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/MC/MCPseudoProbe.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/IR/PseudoProbe.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCAssembler.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCObjectFileInfo.h"
#include "llvm/MC/MCObjectStreamer.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/Support/Endian.h"
#include "llvm/Support/Error.h"
#include "llvm/Support/LEB128.h"
#include "llvm/Support/MD5.h"
#include "llvm/Support/WithColor.h"
#include "llvm/Support/raw_ostream.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCPseudoProbe.h`, `llvm/ADT/STLExtras.h`, `llvm/IR/PseudoProbe.h`, `llvm/MC/MCAsmInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCPseudoProbe.h`, `llvm/ADT/STLExtras.h`, `llvm/IR/PseudoProbe.h`, `llvm/MC/MCAsmInfo.h`。

### Lines 25-39
```cpp
#include <algorithm>
#include <cassert>
#include <limits>
#include <sstream>
#include <vector>

#define DEBUG_TYPE "mcpseudoprobe"

using namespace llvm;
using namespace support;

#ifndef NDEBUG
int MCPseudoProbeTable::DdgPrintIndent = 0;
#endif

```
- **EN**: Pulls in the headers needed for this implementation, including `algorithm`, `cassert`, `limits`, `sstream`.
- **CN**: 引入该实现所需的头文件，其中包括 `algorithm`, `cassert`, `limits`, `sstream`。

### Lines 40-51
```cpp
static const MCExpr *buildSymbolDiff(MCObjectStreamer *MCOS, const MCSymbol *A,
                                     const MCSymbol *B) {
  MCContext &Context = MCOS->getContext();
  const MCExpr *ARef = MCSymbolRefExpr::create(A, Context);
  const MCExpr *BRef = MCSymbolRefExpr::create(B, Context);
  const MCExpr *AddrDelta =
      MCBinaryExpr::create(MCBinaryExpr::Sub, ARef, BRef, Context);
  return AddrDelta;
}

uint64_t MCDecodedPseudoProbe::getGuid() const { return InlineTree->Guid; }

```
- **EN**: Implements logic around `buildSymbolDiff`, `getContext`, `create`, `getGuid`; this block decodes machine-code bytes into symbolic instruction form; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `buildSymbolDiff`, `getContext`, `create`, `getGuid` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 52-75
```cpp
void MCPseudoProbe::emit(MCObjectStreamer *MCOS,
                         const MCPseudoProbe *LastProbe) const {
  bool IsSentinel = isSentinelProbe(getAttributes());
  assert((LastProbe || IsSentinel) &&
         "Last probe should not be null for non-sentinel probes");

  // Emit Index
  MCOS->emitULEB128IntValue(Index);
  // Emit Type and the flag:
  // Type (bit 0 to 3), with bit 4 to 6 for attributes.
  // Flag (bit 7, 0 - code address, 1 - address delta). This indicates whether
  // the following field is a symbolic code address or an address delta.
  // Emit FS discriminator
  assert(Type <= 0xF && "Probe type too big to encode, exceeding 15");
  auto NewAttributes = Attributes;
  if (Discriminator)
    NewAttributes |= (uint32_t)PseudoProbeAttributes::HasDiscriminator;
  assert(NewAttributes <= 0x7 &&
         "Probe attributes too big to encode, exceeding 7");
  uint8_t PackedType = Type | (NewAttributes << 4);
  uint8_t Flag =
      !IsSentinel ? ((int8_t)MCPseudoProbeFlag::AddressDelta << 7) : 0;
  MCOS->emitInt8(Flag | PackedType);

```
- **EN**: Implements logic around `emit`, `isSentinelProbe`, `assert`, `emitULEB128IntValue`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `emit`, `isSentinelProbe`, `assert`, `emitULEB128IntValue`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 76-92
```cpp
  if (!IsSentinel) {
    // Emit the delta between the address label and LastProbe.
    const MCExpr *AddrDelta =
        buildSymbolDiff(MCOS, Label, LastProbe->getLabel());
    int64_t Delta;
    if (AddrDelta->evaluateAsAbsolute(Delta, MCOS->getAssemblerPtr())) {
      MCOS->emitSLEB128IntValue(Delta);
    } else {
      auto *F = MCOS->getCurrentFragment();
      F->makeLEB(true, AddrDelta);
      MCOS->newFragment();
    }
  } else {
    // Emit the GUID of the split function that the sentinel probe represents.
    MCOS->emitInt64(Guid);
  }

```
- **EN**: Implements logic around `buildSymbolDiff`, `emitSLEB128IntValue`, `getCurrentFragment`, `makeLEB`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `buildSymbolDiff`, `emitSLEB128IntValue`, `getCurrentFragment`, `makeLEB`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 93-106
```cpp
  if (Discriminator)
    MCOS->emitULEB128IntValue(Discriminator);

  LLVM_DEBUG({
    dbgs().indent(MCPseudoProbeTable::DdgPrintIndent);
    dbgs() << "Probe: " << Index << "\n";
  });
}

void MCPseudoProbeInlineTree::addPseudoProbe(
    const MCPseudoProbe &Probe, const MCPseudoProbeInlineStack &InlineStack) {
  // The function should not be called on the root.
  assert(isRoot() && "Should only be called on root");

```
- **EN**: Implements logic around `emitULEB128IntValue`, `dbgs`, `addPseudoProbe`, `assert`.
- **CN**: 围绕 `emitULEB128IntValue`, `dbgs`, `addPseudoProbe`, `assert` 实现具体逻辑。

### Lines 107-125
```cpp
  // When it comes here, the input look like:
  //    Probe: GUID of C, ...
  //    InlineStack: [88, A], [66, B]
  // which means, Function A inlines function B at call site with a probe id of
  // 88, and B inlines C at probe 66. The tri-tree expects a tree path like {[0,
  // A], [88, B], [66, C]} to locate the tree node where the probe should be
  // added. Note that the edge [0, A] means A is the top-level function we are
  // emitting probes for.

  // Make a [0, A] edge.
  // An empty inline stack means the function that the probe originates from
  // is a top-level function.
  InlineSite Top;
  if (InlineStack.empty()) {
    Top = InlineSite(Probe.getGuid(), 0);
  } else {
    Top = InlineSite(std::get<0>(InlineStack.front()), 0);
  }

```
- **EN**: Implements logic around `InlineSite`.
- **CN**: 围绕 `InlineSite` 实现具体逻辑。

### Lines 126-141
```cpp
  auto *Cur = getOrAddNode(Top);

  // Make interior edges by walking the inline stack. Once it's done, Cur should
  // point to the node that the probe originates from.
  if (!InlineStack.empty()) {
    auto Iter = InlineStack.begin();
    auto Index = std::get<1>(*Iter);
    Iter++;
    for (; Iter != InlineStack.end(); Iter++) {
      // Make an edge by using the previous probe id and current GUID.
      Cur = Cur->getOrAddNode(InlineSite(std::get<0>(*Iter), Index));
      Index = std::get<1>(*Iter);
    }
    Cur = Cur->getOrAddNode(InlineSite(Probe.getGuid(), Index));
  }

```
- **EN**: Implements logic around `getOrAddNode`, `begin`, `get<1>`.
- **CN**: 围绕 `getOrAddNode`, `begin`, `get<1>` 实现具体逻辑。

### Lines 142-153
```cpp
  Cur->Probes.push_back(Probe);
}

void MCPseudoProbeInlineTree::emit(MCObjectStreamer *MCOS,
                                   const MCPseudoProbe *&LastProbe) {
  LLVM_DEBUG({
    dbgs().indent(MCPseudoProbeTable::DdgPrintIndent);
    dbgs() << "Group [\n";
    MCPseudoProbeTable::DdgPrintIndent += 2;
  });
  assert(!isRoot() && "Root should be handled separately");

```
- **EN**: Implements logic around `push_back`, `emit`, `dbgs`, `assert`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `push_back`, `emit`, `dbgs`, `assert` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 154-171
```cpp
  // Emit probes grouped by GUID.
  LLVM_DEBUG({
    dbgs().indent(MCPseudoProbeTable::DdgPrintIndent);
    dbgs() << "GUID: " << Guid << "\n";
  });
  // Emit Guid
  MCOS->emitInt64(Guid);
  // Emit number of probes in this node, including a sentinel probe for
  // top-level functions if needed.
  bool NeedSentinel = false;
  if (Parent->isRoot()) {
    assert(isSentinelProbe(LastProbe->getAttributes()) &&
           "Starting probe of a top-level function should be a sentinel probe");
    // The main body of a split function doesn't need a sentinel probe.
    if (LastProbe->getGuid() != Guid)
      NeedSentinel = true;
  }

```
- **EN**: Implements logic around `dbgs`, `emitInt64`, `assert`.
- **CN**: 围绕 `dbgs`, `emitInt64`, `assert` 实现具体逻辑。

### Lines 172-184
```cpp
  MCOS->emitULEB128IntValue(Probes.size() + NeedSentinel);
  // Emit number of direct inlinees
  MCOS->emitULEB128IntValue(Children.size());
  // Emit sentinel probe for top-level functions
  if (NeedSentinel)
    LastProbe->emit(MCOS, nullptr);

  // Emit probes in this group
  for (const auto &Probe : Probes) {
    Probe.emit(MCOS, LastProbe);
    LastProbe = &Probe;
  }

```
- **EN**: Implements logic around `emitULEB128IntValue`, `emit`.
- **CN**: 围绕 `emitULEB128IntValue`, `emit` 实现具体逻辑。

### Lines 185-203
```cpp
  // Emit sorted descendant. InlineSite is unique for each pair, so there will
  // be no ordering of Inlinee based on MCPseudoProbeInlineTree*
  using InlineeType = std::pair<InlineSite, MCPseudoProbeInlineTree *>;
  std::vector<InlineeType> Inlinees;
  for (const auto &Child : Children)
    Inlinees.emplace_back(Child.first, Child.second.get());
  llvm::sort(Inlinees, llvm::less_first());

  for (const auto &Inlinee : Inlinees) {
    // Emit probe index
    MCOS->emitULEB128IntValue(std::get<1>(Inlinee.first));
    LLVM_DEBUG({
      dbgs().indent(MCPseudoProbeTable::DdgPrintIndent);
      dbgs() << "InlineSite: " << std::get<1>(Inlinee.first) << "\n";
    });
    // Emit the group
    Inlinee.second->emit(MCOS, LastProbe);
  }

```
- **EN**: Implements logic around `emplace_back`, `sort`, `emitULEB128IntValue`, `dbgs`, and 1 more symbols.
- **CN**: 围绕 `emplace_back`, `sort`, `emitULEB128IntValue`, `dbgs`, and 1 more symbols 实现具体逻辑。

### Lines 204-227
```cpp
  LLVM_DEBUG({
    MCPseudoProbeTable::DdgPrintIndent -= 2;
    dbgs().indent(MCPseudoProbeTable::DdgPrintIndent);
    dbgs() << "]\n";
  });
}

void MCPseudoProbeSections::emit(MCObjectStreamer *MCOS) {
  MCContext &Ctx = MCOS->getContext();
  SmallVector<std::pair<MCSymbol *, MCPseudoProbeInlineTree *>> Vec;
  Vec.reserve(MCProbeDivisions.size());
  for (auto &ProbeSec : MCProbeDivisions)
    Vec.emplace_back(ProbeSec.first, &ProbeSec.second);
  for (auto I : llvm::enumerate(MCOS->getAssembler()))
    I.value().setOrdinal(I.index());
  llvm::sort(Vec, [](auto A, auto B) {
    return A.first->getSection().getOrdinal() <
           B.first->getSection().getOrdinal();
  });
  for (auto [FuncSym, RootPtr] : Vec) {
    const auto &Root = *RootPtr;
    if (auto *S = Ctx.getObjectFileInfo()->getPseudoProbeSection(
            FuncSym->getSection())) {
      // Switch to the .pseudoprobe section or a comdat group.
```
- **EN**: Implements logic around `dbgs`, `emit`, `getContext`, `reserve`, and 4 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `dbgs`, `emit`, `getContext`, `reserve`, and 4 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 228-251
```cpp
      MCOS->switchSection(S);
      // Emit probes grouped by GUID.
      // Emit sorted descendant. InlineSite is unique for each pair, so there
      // will be no ordering of Inlinee based on MCPseudoProbeInlineTree*
      using InlineeType = std::pair<InlineSite, MCPseudoProbeInlineTree *>;
      std::vector<InlineeType> Inlinees;
      for (const auto &Child : Root.getChildren())
        Inlinees.emplace_back(Child.first, Child.second.get());
      llvm::sort(Inlinees, llvm::less_first());

      for (const auto &Inlinee : Inlinees) {
        // Emit the group guarded by a sentinel probe.
        MCPseudoProbe SentinelProbe(
            const_cast<MCSymbol *>(FuncSym), MD5Hash(FuncSym->getName()),
            (uint32_t)PseudoProbeReservedId::Invalid,
            (uint32_t)PseudoProbeType::Block,
            (uint32_t)PseudoProbeAttributes::Sentinel, 0);
        const MCPseudoProbe *Probe = &SentinelProbe;
        Inlinee.second->emit(MCOS, Probe);
      }
    }
  }
}

```
- **EN**: Implements logic around `switchSection`, `emplace_back`, `sort`, `SentinelProbe`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `switchSection`, `emplace_back`, `sort`, `SentinelProbe`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 252-264
```cpp
//
// This emits the pseudo probe tables.
//
void MCPseudoProbeTable::emit(MCObjectStreamer *MCOS) {
  MCContext &Ctx = MCOS->getContext();
  auto &ProbeTable = Ctx.getMCPseudoProbeTable();

  // Bail out early so we don't switch to the pseudo_probe section needlessly
  // and in doing so create an unnecessary (if empty) section.
  auto &ProbeSections = ProbeTable.getProbeSections();
  if (ProbeSections.empty())
    return;

```
- **EN**: Implements logic around `emit`, `getContext`, `getMCPseudoProbeTable`, `getProbeSections`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `emit`, `getContext`, `getMCPseudoProbeTable`, `getProbeSections` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 265-278
```cpp
  LLVM_DEBUG(MCPseudoProbeTable::DdgPrintIndent = 0);

  // Put out the probe.
  ProbeSections.emit(MCOS);
}

static StringRef getProbeFNameForGUID(const GUIDProbeFunctionMap &GUID2FuncMAP,
                                      uint64_t GUID) {
  auto It = GUID2FuncMAP.find(GUID);
  assert(It != GUID2FuncMAP.end() &&
         "Probe function must exist for a valid GUID");
  return It->FuncName;
}

```
- **EN**: Implements logic around `emit`, `getProbeFNameForGUID`, `find`, `assert`; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `emit`, `getProbeFNameForGUID`, `find`, `assert` 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 279-300
```cpp
void MCPseudoProbeFuncDesc::print(raw_ostream &OS) {
  OS << "GUID: " << FuncGUID << " Name: " << FuncName << "\n";
  OS << "Hash: " << FuncHash << "\n";
}

void MCDecodedPseudoProbe::getInlineContext(
    SmallVectorImpl<MCPseudoProbeFrameLocation> &ContextStack,
    const GUIDProbeFunctionMap &GUID2FuncMAP) const {
  uint32_t Begin = ContextStack.size();
  MCDecodedPseudoProbeInlineTree *Cur = InlineTree;
  // It will add the string of each node's inline site during iteration.
  // Note that it won't include the probe's belonging function(leaf location)
  while (Cur->hasInlineSite()) {
    StringRef FuncName = getProbeFNameForGUID(GUID2FuncMAP, Cur->Parent->Guid);
    ContextStack.emplace_back(MCPseudoProbeFrameLocation(
        FuncName, std::get<1>(Cur->getInlineSite())));
    Cur = static_cast<MCDecodedPseudoProbeInlineTree *>(Cur->Parent);
  }
  // Make the ContextStack in caller-callee order
  std::reverse(ContextStack.begin() + Begin, ContextStack.end());
}

```
- **EN**: Implements logic around `print`, `getInlineContext`, `size`, `getProbeFNameForGUID`, and 3 more symbols; this block decodes machine-code bytes into symbolic instruction form.
- **CN**: 围绕 `print`, `getInlineContext`, `size`, `getProbeFNameForGUID`, and 3 more symbols 实现具体逻辑；这一段把机器码字节解码为符号化指令形式。

### Lines 301-313
```cpp
std::string MCDecodedPseudoProbe::getInlineContextStr(
    const GUIDProbeFunctionMap &GUID2FuncMAP) const {
  std::ostringstream OContextStr;
  SmallVector<MCPseudoProbeFrameLocation, 16> ContextStack;
  getInlineContext(ContextStack, GUID2FuncMAP);
  for (auto &Cxt : ContextStack) {
    if (OContextStr.str().size())
      OContextStr << " @ ";
    OContextStr << Cxt.first.str() << ":" << Cxt.second;
  }
  return OContextStr.str();
}

```
- **EN**: Implements logic around `getInlineContextStr`, `getInlineContext`, `str`; this block decodes machine-code bytes into symbolic instruction form; returns subsystem-specific computed results.
- **CN**: 围绕 `getInlineContextStr`, `getInlineContext`, `str` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，返回子系统相关的计算结果。

### Lines 314-337
```cpp
static const char *PseudoProbeTypeStr[3] = {"Block", "IndirectCall",
                                            "DirectCall"};

void MCDecodedPseudoProbe::print(raw_ostream &OS,
                                 const GUIDProbeFunctionMap &GUID2FuncMAP,
                                 bool ShowName) const {
  OS << "FUNC: ";
  if (ShowName) {
    StringRef FuncName = getProbeFNameForGUID(GUID2FuncMAP, getGuid());
    OS << FuncName.str() << " ";
  } else {
    OS << getGuid() << " ";
  }
  OS << "Index: " << Index << "  ";
  if (Discriminator)
    OS << "Discriminator: " << Discriminator << "  ";
  OS << "Type: " << PseudoProbeTypeStr[static_cast<uint8_t>(Type)] << "  ";
  std::string InlineContextStr = getInlineContextStr(GUID2FuncMAP);
  if (InlineContextStr.size()) {
    OS << "Inlined: @ ";
    OS << InlineContextStr;
  }
  OS << "\n";
}
```
- **EN**: Implements logic around `print`, `getProbeFNameForGUID`, `str`, `getGuid`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes; decodes machine-code bytes into symbolic instruction form.
- **CN**: 围绕 `print`, `getProbeFNameForGUID`, `str`, `getGuid`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，把机器码字节解码为符号化指令形式。

### Lines 338-356
```cpp

template <typename T> ErrorOr<T> MCPseudoProbeDecoder::readUnencodedNumber() {
  if (Data + sizeof(T) > End) {
    return std::error_code();
  }
  T Val = endian::readNext<T, llvm::endianness::little>(Data);
  return ErrorOr<T>(Val);
}

template <typename T> ErrorOr<T> MCPseudoProbeDecoder::readUnsignedNumber() {
  unsigned NumBytesRead = 0;
  uint64_t Val = decodeULEB128(Data, &NumBytesRead);
  if (Val > std::numeric_limits<T>::max() || (Data + NumBytesRead > End)) {
    return std::error_code();
  }
  Data += NumBytesRead;
  return ErrorOr<T>(static_cast<T>(Val));
}

```
- **EN**: Implements logic around `readUnencodedNumber`, `error_code`, `little>`, `ErrorOr<T>`, and 2 more symbols; this block decodes machine-code bytes into symbolic instruction form; returns subsystem-specific computed results.
- **CN**: 围绕 `readUnencodedNumber`, `error_code`, `little>`, `ErrorOr<T>`, and 2 more symbols 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，返回子系统相关的计算结果。

### Lines 357-375
```cpp
template <typename T> ErrorOr<T> MCPseudoProbeDecoder::readSignedNumber() {
  unsigned NumBytesRead = 0;
  int64_t Val = decodeSLEB128(Data, &NumBytesRead);
  if (Val > std::numeric_limits<T>::max() || (Data + NumBytesRead > End)) {
    return std::error_code();
  }
  Data += NumBytesRead;
  return ErrorOr<T>(static_cast<T>(Val));
}

ErrorOr<StringRef> MCPseudoProbeDecoder::readString(uint32_t Size) {
  StringRef Str(reinterpret_cast<const char *>(Data), Size);
  if (Data + Size > End) {
    return std::error_code();
  }
  Data += Size;
  return ErrorOr<StringRef>(Str);
}

```
- **EN**: Implements logic around `readSignedNumber`, `decodeSLEB128`, `error_code`, `ErrorOr<T>`, and 3 more symbols; this block decodes machine-code bytes into symbolic instruction form; returns subsystem-specific computed results.
- **CN**: 围绕 `readSignedNumber`, `decodeSLEB128`, `error_code`, `ErrorOr<T>`, and 3 more symbols 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，返回子系统相关的计算结果。

### Lines 376-390
```cpp
bool MCPseudoProbeDecoder::buildGUID2FuncDescMap(const uint8_t *Start,
                                                 std::size_t Size,
                                                 bool IsMMapped,
                                                 bool VerboseWarnings) {
  // The pseudo_probe_desc section has a format like:
  // .section .pseudo_probe_desc,"",@progbits
  // .quad -5182264717993193164   // GUID
  // .quad 4294967295             // Hash
  // .uleb 3                      // Name size
  // .ascii "foo"                 // Name
  // .quad -2624081020897602054
  // .quad 174696971957
  // .uleb 34
  // .ascii "main"

```
- **EN**: Implements logic around `buildGUID2FuncDescMap`; this block decodes machine-code bytes into symbolic instruction form; updates MC section or symbol state.
- **CN**: 围绕 `buildGUID2FuncDescMap` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，更新 MC 节区或符号状态。

### Lines 391-402
```cpp
  Data = Start;
  End = Data + Size;

  uint32_t FuncDescCount = 0;
  while (Data < End) {
    // GUID
    if (!readUnencodedNumber<uint64_t>())
      return false;
    // Hash
    if (!readUnencodedNumber<uint64_t>())
      return false;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 403-424
```cpp
    auto ErrorOrNameSize = readUnsignedNumber<uint32_t>();
    if (!ErrorOrNameSize)
      return false;
    // Function name
    if (!readString(*ErrorOrNameSize))
      return false;
    ++FuncDescCount;
  }
  assert(Data == End && "Have unprocessed data in pseudo_probe_desc section");
  GUID2FuncDescMap.reserve(FuncDescCount);

  Data = Start;
  End = Data + Size;
  while (Data < End) {
    uint64_t GUID =
        cantFail(errorOrToExpected(readUnencodedNumber<uint64_t>()));
    uint64_t Hash =
        cantFail(errorOrToExpected(readUnencodedNumber<uint64_t>()));
    uint32_t NameSize =
        cantFail(errorOrToExpected(readUnsignedNumber<uint32_t>()));
    StringRef Name = cantFail(errorOrToExpected(readString(NameSize)));

```
- **EN**: Implements logic around `readUnsignedNumber<uint32_t>`, `assert`, `reserve`, `cantFail`; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `readUnsignedNumber<uint32_t>`, `assert`, `reserve`, `cantFail` 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 425-448
```cpp
    // Initialize PseudoProbeFuncDesc and populate it into GUID2FuncDescMap
    GUID2FuncDescMap.emplace_back(
        GUID, Hash, IsMMapped ? Name : Name.copy(FuncNameAllocator));
  }
  assert(Data == End && "Have unprocessed data in pseudo_probe_desc section");
  assert(GUID2FuncDescMap.size() == FuncDescCount &&
         "Mismatching function description count pre- and post-parsing");
  llvm::stable_sort(GUID2FuncDescMap, [](const auto &LHS, const auto &RHS) {
    return LHS.FuncGUID < RHS.FuncGUID;
  });

  // Detect duplicate GUIDs with different hashes across TUs.
  uint32_t MismatchCount = 0;
  uint64_t LastMismatchGUID = 0;
  for (size_t I = 1; I < GUID2FuncDescMap.size(); ++I) {
    const auto &Prev = GUID2FuncDescMap[I - 1];
    const auto &Curr = GUID2FuncDescMap[I];
    if (Prev.FuncGUID == Curr.FuncGUID && Prev.FuncHash != Curr.FuncHash) {
      if (LastMismatchGUID != Curr.FuncGUID) {
        ++MismatchCount;
        LastMismatchGUID = Curr.FuncGUID;
      }
      if (VerboseWarnings)
        WithColor::warning() << "pseudo probe descriptor for " << Prev.FuncName
```
- **EN**: Implements logic around `emplace_back`, `copy`, `assert`, `stable_sort`, and 1 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `emplace_back`, `copy`, `assert`, `stable_sort`, and 1 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 449-460
```cpp
                             << " has mismatching hash across TUs: "
                             << format_hex(Prev.FuncHash, 18) << " vs "
                             << format_hex(Curr.FuncHash, 18) << "\n";
    }
  }
  if (MismatchCount > 0)
    WithColor::warning() << MismatchCount
                         << " functions have mismatching pseudo probe "
                            "descriptors across translation units.\n";
  return true;
}

```
- **EN**: Implements logic around `format_hex`, `warning`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `format_hex`, `warning` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 461-477
```cpp
template <bool IsTopLevelFunc>
bool MCPseudoProbeDecoder::buildAddress2ProbeMap(
    MCDecodedPseudoProbeInlineTree *Cur, uint64_t &LastAddr,
    const Uint64Set &GuidFilter, const Uint64Map &FuncStartAddrs,
    const uint32_t CurChildIndex) {
  // The pseudo_probe section encodes an inline forest and each tree has a
  // format defined in MCPseudoProbe.h

  uint32_t Index = 0;
  if (IsTopLevelFunc) {
    // Use a sequential id for top level inliner.
    Index = CurChildIndex;
  } else {
    // Read inline site for inlinees
    Index = cantFail(errorOrToExpected(readUnsignedNumber<uint32_t>()));
  }

```
- **EN**: Implements logic around `buildAddress2ProbeMap`, `cantFail`; this block decodes machine-code bytes into symbolic instruction form; updates MC section or symbol state.
- **CN**: 围绕 `buildAddress2ProbeMap`, `cantFail` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，更新 MC 节区或符号状态。

### Lines 478-496
```cpp
  // Read guid
  uint64_t Guid = cantFail(errorOrToExpected(readUnencodedNumber<uint64_t>()));

  // Decide if top-level node should be disgarded.
  if (IsTopLevelFunc && !GuidFilter.empty() && !GuidFilter.count(Guid))
    Cur = nullptr;

  // If the incoming node is null, all its children nodes should be disgarded.
  if (Cur) {
    // Switch/add to a new tree node(inlinee)
    Cur->getChildren()[CurChildIndex] =
        MCDecodedPseudoProbeInlineTree(InlineSite(Guid, Index), Cur);
    Cur = &Cur->getChildren()[CurChildIndex];
    if (IsTopLevelFunc && !EncodingIsAddrBased) {
      if (auto V = FuncStartAddrs.lookup(Guid))
        LastAddr = V;
    }
  }

```
- **EN**: Implements logic around `cantFail`, `getChildren`, `MCDecodedPseudoProbeInlineTree`; this block decodes machine-code bytes into symbolic instruction form; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `cantFail`, `getChildren`, `MCDecodedPseudoProbeInlineTree` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 497-520
```cpp
  // Read number of probes in the current node.
  uint32_t NodeCount =
      cantFail(errorOrToExpected(readUnsignedNumber<uint32_t>()));
  uint32_t CurrentProbeCount = 0;
  // Read number of direct inlinees
  uint32_t ChildrenToProcess =
      cantFail(errorOrToExpected(readUnsignedNumber<uint32_t>()));
  // Read all probes in this node
  for (std::size_t I = 0; I < NodeCount; I++) {
    // Read index
    uint32_t Index =
        cantFail(errorOrToExpected(readUnsignedNumber<uint32_t>()));
    // Read type | flag.
    uint8_t Value = cantFail(errorOrToExpected(readUnencodedNumber<uint8_t>()));
    uint8_t Kind = Value & 0xf;
    uint8_t Attr = (Value & 0x70) >> 4;
    // Read address
    uint64_t Addr = 0;
    if (Value & 0x80) {
      int64_t Offset = cantFail(errorOrToExpected(readSignedNumber<int64_t>()));
      Addr = LastAddr + Offset;
    } else {
      Addr = cantFail(errorOrToExpected(readUnencodedNumber<int64_t>()));
      if (isSentinelProbe(Attr)) {
```
- **EN**: Implements logic around `cantFail`.
- **CN**: 围绕 `cantFail` 实现具体逻辑。

### Lines 521-533
```cpp
        // For sentinel probe, the addr field actually stores the GUID of the
        // split function. Convert it to the real address.
        if (auto V = FuncStartAddrs.lookup(Addr))
          Addr = V;
      } else {
        // For now we assume all probe encoding should be either based on
        // leading probe address or function start address.
        // The scheme is for downwards compatibility.
        // TODO: retire this scheme once compatibility is no longer an issue.
        EncodingIsAddrBased = true;
      }
    }

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 534-547
```cpp
    uint32_t Discriminator = 0;
    if (hasDiscriminator(Attr)) {
      Discriminator =
          cantFail(errorOrToExpected(readUnsignedNumber<uint32_t>()));
    }

    if (Cur && !isSentinelProbe(Attr)) {
      PseudoProbeVec.emplace_back(Addr, Index, PseudoProbeType(Kind), Attr,
                                  Discriminator, Cur);
      ++CurrentProbeCount;
    }
    LastAddr = Addr;
  }

```
- **EN**: Implements logic around `cantFail`, `emplace_back`.
- **CN**: 围绕 `cantFail`, `emplace_back` 实现具体逻辑。

### Lines 548-560
```cpp
  if (Cur) {
    Cur->setProbes(
        MutableArrayRef(PseudoProbeVec).take_back(CurrentProbeCount));
    InlineTreeVec.resize(InlineTreeVec.size() + ChildrenToProcess);
    Cur->getChildren() =
        MutableArrayRef(InlineTreeVec).take_back(ChildrenToProcess);
  }
  for (uint32_t I = 0; I < ChildrenToProcess; I++) {
    buildAddress2ProbeMap<false>(Cur, LastAddr, GuidFilter, FuncStartAddrs, I);
  }
  return Cur;
}

```
- **EN**: Implements logic around `setProbes`, `MutableArrayRef`, `resize`, `getChildren`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `setProbes`, `MutableArrayRef`, `resize`, `getChildren`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 561-575
```cpp
template <bool IsTopLevelFunc>
bool MCPseudoProbeDecoder::countRecords(bool &Discard, uint32_t &ProbeCount,
                                        uint32_t &InlinedCount,
                                        const Uint64Set &GuidFilter) {
  if (!IsTopLevelFunc)
    // Read inline site for inlinees
    if (!readUnsignedNumber<uint32_t>())
      return false;

  // Read guid
  auto ErrorOrCurGuid = readUnencodedNumber<uint64_t>();
  if (!ErrorOrCurGuid)
    return false;
  uint64_t Guid = std::move(*ErrorOrCurGuid);

```
- **EN**: Implements logic around `countRecords`, `readUnencodedNumber<uint64_t>`, `move`; this block decodes machine-code bytes into symbolic instruction form; returns subsystem-specific computed results.
- **CN**: 围绕 `countRecords`, `readUnencodedNumber<uint64_t>`, `move` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，返回子系统相关的计算结果。

### Lines 576-590
```cpp
  // Decide if top-level node should be disgarded.
  if (IsTopLevelFunc) {
    Discard = !GuidFilter.empty() && !GuidFilter.count(Guid);
    if (!Discard)
      // Allocate an entry for top-level function record.
      ++InlinedCount;
  }

  // Read number of probes in the current node.
  auto ErrorOrNodeCount = readUnsignedNumber<uint32_t>();
  if (!ErrorOrNodeCount)
    return false;
  uint32_t NodeCount = std::move(*ErrorOrNodeCount);
  uint32_t CurrentProbeCount = 0;

```
- **EN**: Implements logic around `empty`, `readUnsignedNumber<uint32_t>`, `move`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `empty`, `readUnsignedNumber<uint32_t>`, `move` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 591-602
```cpp
  // Read number of direct inlinees
  auto ErrorOrCurChildrenToProcess = readUnsignedNumber<uint32_t>();
  if (!ErrorOrCurChildrenToProcess)
    return false;
  uint32_t ChildrenToProcess = std::move(*ErrorOrCurChildrenToProcess);

  // Read all probes in this node
  for (std::size_t I = 0; I < NodeCount; I++) {
    // Read index
    if (!readUnsignedNumber<uint32_t>())
      return false;

```
- **EN**: Implements logic around `readUnsignedNumber<uint32_t>`, `move`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `readUnsignedNumber<uint32_t>`, `move` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 603-619
```cpp
    // Read type | flag.
    auto ErrorOrValue = readUnencodedNumber<uint8_t>();
    if (!ErrorOrValue)
      return false;
    uint8_t Value = std::move(*ErrorOrValue);

    uint8_t Attr = (Value & 0x70) >> 4;
    if (Value & 0x80) {
      // Offset
      if (!readSignedNumber<int64_t>())
        return false;
    } else {
      // Addr
      if (!readUnencodedNumber<int64_t>())
        return false;
    }

```
- **EN**: Implements logic around `readUnencodedNumber<uint8_t>`, `move`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `readUnencodedNumber<uint8_t>`, `move` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 620-633
```cpp
    if (hasDiscriminator(Attr))
      // Discriminator
      if (!readUnsignedNumber<uint32_t>())
        return false;

    if (!Discard && !isSentinelProbe(Attr))
      ++CurrentProbeCount;
  }

  if (!Discard) {
    ProbeCount += CurrentProbeCount;
    InlinedCount += ChildrenToProcess;
  }

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 634-657
```cpp
  for (uint32_t I = 0; I < ChildrenToProcess; I++)
    if (!countRecords<false>(Discard, ProbeCount, InlinedCount, GuidFilter))
      return false;
  return true;
}

bool MCPseudoProbeDecoder::buildAddress2ProbeMap(
    const uint8_t *Start, std::size_t Size, const Uint64Set &GuidFilter,
    const Uint64Map &FuncStartAddrs) {
  // For function records in the order of their appearance in the encoded data
  // (DFS), count the number of contained probes and inlined function records.
  uint32_t ProbeCount = 0;
  uint32_t InlinedCount = 0;
  uint32_t TopLevelFuncs = 0;
  Data = Start;
  End = Data + Size;
  bool Discard = false;
  while (Data < End) {
    if (!countRecords<true>(Discard, ProbeCount, InlinedCount, GuidFilter))
      return false;
    TopLevelFuncs += !Discard;
  }
  assert(Data == End && "Have unprocessed data in pseudo_probe section");
  PseudoProbeVec.reserve(ProbeCount);
```
- **EN**: Implements logic around `buildAddress2ProbeMap`, `assert`, `reserve`; this block decodes machine-code bytes into symbolic instruction form; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `buildAddress2ProbeMap`, `assert`, `reserve` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 658-676
```cpp
  InlineTreeVec.reserve(InlinedCount);

  // Allocate top-level function records as children of DummyInlineRoot.
  InlineTreeVec.resize(TopLevelFuncs);
  DummyInlineRoot.getChildren() = MutableArrayRef(InlineTreeVec);

  Data = Start;
  End = Data + Size;
  uint64_t LastAddr = 0;
  uint32_t CurChildIndex = 0;
  while (Data < End)
    CurChildIndex += buildAddress2ProbeMap<true>(
        &DummyInlineRoot, LastAddr, GuidFilter, FuncStartAddrs, CurChildIndex);
  assert(Data == End && "Have unprocessed data in pseudo_probe section");
  assert(PseudoProbeVec.size() == ProbeCount &&
         "Mismatching probe count pre- and post-parsing");
  assert(InlineTreeVec.size() == InlinedCount &&
         "Mismatching function records count pre- and post-parsing");

```
- **EN**: Implements logic around `reserve`, `resize`, `getChildren`, `buildAddress2ProbeMap<true>`, and 1 more symbols; this block updates MC section or symbol state.
- **CN**: 围绕 `reserve`, `resize`, `getChildren`, `buildAddress2ProbeMap<true>`, and 1 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 677-693
```cpp
  std::vector<std::pair<uint64_t, uint32_t>> SortedA2P(ProbeCount);
  for (const auto &[I, Probe] : llvm::enumerate(PseudoProbeVec))
    SortedA2P[I] = {Probe.getAddress(), I};
  llvm::sort(SortedA2P);
  Address2ProbesMap.reserve(ProbeCount);
  for (const uint32_t I : llvm::make_second_range(SortedA2P))
    Address2ProbesMap.emplace_back(PseudoProbeVec[I]);
  SortedA2P.clear();
  return true;
}

void MCPseudoProbeDecoder::printGUID2FuncDescMap(raw_ostream &OS) {
  OS << "Pseudo Probe Desc:\n";
  for (auto &I : GUID2FuncDescMap)
    I.print(OS);
}

```
- **EN**: Implements logic around `SortedA2P`, `getAddress`, `sort`, `reserve`, and 4 more symbols; this block decodes machine-code bytes into symbolic instruction form; returns subsystem-specific computed results.
- **CN**: 围绕 `SortedA2P`, `getAddress`, `sort`, `reserve`, and 4 more symbols 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，返回子系统相关的计算结果。

### Lines 694-714
```cpp
void MCPseudoProbeDecoder::printProbeForAddress(raw_ostream &OS,
                                                uint64_t Address) {
  for (const MCDecodedPseudoProbe &Probe : Address2ProbesMap.find(Address)) {
    OS << " [Probe]:\t";
    Probe.print(OS, GUID2FuncDescMap, true);
  }
}

void MCPseudoProbeDecoder::printProbesForAllAddresses(raw_ostream &OS) {
  uint64_t PrevAddress = INT64_MAX;
  for (MCDecodedPseudoProbe &Probe : Address2ProbesMap) {
    uint64_t Address = Probe.getAddress();
    if (Address != PrevAddress) {
      PrevAddress = Address;
      OS << "Address:\t" << Address << '\n';
    }
    OS << " [Probe]:\t";
    Probe.print(OS, GUID2FuncDescMap, true);
  }
}

```
- **EN**: Implements logic around `printProbeForAddress`, `print`, `printProbesForAllAddresses`, `getAddress`; this block decodes machine-code bytes into symbolic instruction form.
- **CN**: 围绕 `printProbeForAddress`, `print`, `printProbesForAllAddresses`, `getAddress` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式。

### Lines 715-738
```cpp
const MCDecodedPseudoProbe *
MCPseudoProbeDecoder::getCallProbeForAddr(uint64_t Address) const {
  const MCDecodedPseudoProbe *CallProbe = nullptr;
  for (const MCDecodedPseudoProbe &Probe : Address2ProbesMap.find(Address)) {
    if (Probe.isCall()) {
      // Disabling the assert and returning first call probe seen so far.
      // Subsequent call probes, if any, are ignored. Due to the the way
      // .pseudo_probe section is decoded, probes of the same-named independent
      // static functions are merged thus multiple call probes may be seen for a
      // callsite. This should only happen to compiler-generated statics, with
      // -funique-internal-linkage-names where user statics get unique names.
      //
      // TODO: re-enable or narrow down the assert to static functions only.
      //
      // assert(!CallProbe &&
      //        "There should be only one call probe corresponding to address "
      //        "which is a callsite.");
      CallProbe = &Probe;
      break;
    }
  }
  return CallProbe;
}

```
- **EN**: Implements logic around `getCallProbeForAddr`; this block decodes machine-code bytes into symbolic instruction form; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getCallProbeForAddr` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 739-759
```cpp
const MCPseudoProbeFuncDesc *
MCPseudoProbeDecoder::getFuncDescForGUID(uint64_t GUID) const {
  auto It = GUID2FuncDescMap.find(GUID);
  assert(It != GUID2FuncDescMap.end() && "Function descriptor doesn't exist");
  return &*It;
}

void MCPseudoProbeDecoder::getInlineContextForProbe(
    const MCDecodedPseudoProbe *Probe,
    SmallVectorImpl<MCPseudoProbeFrameLocation> &InlineContextStack,
    bool IncludeLeaf) const {
  Probe->getInlineContext(InlineContextStack, GUID2FuncDescMap);
  if (!IncludeLeaf)
    return;
  // Note that the context from probe doesn't include leaf frame,
  // hence we need to retrieve and prepend leaf if requested.
  const auto *FuncDesc = getFuncDescForGUID(Probe->getGuid());
  InlineContextStack.emplace_back(
      MCPseudoProbeFrameLocation(FuncDesc->FuncName, Probe->getIndex()));
}

```
- **EN**: Implements logic around `getFuncDescForGUID`, `find`, `assert`, `getInlineContextForProbe`, and 3 more symbols; this block decodes machine-code bytes into symbolic instruction form; returns subsystem-specific computed results.
- **CN**: 围绕 `getFuncDescForGUID`, `find`, `assert`, `getInlineContextForProbe`, and 3 more symbols 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，返回子系统相关的计算结果。

### Lines 760-766
```cpp
const MCPseudoProbeFuncDesc *MCPseudoProbeDecoder::getInlinerDescForProbe(
    const MCDecodedPseudoProbe *Probe) const {
  MCDecodedPseudoProbeInlineTree *InlinerNode = Probe->getInlineTreeNode();
  if (!InlinerNode->hasInlineSite())
    return nullptr;
  return getFuncDescForGUID(InlinerNode->Parent->Guid);
}
```
- **EN**: Implements logic around `getInlinerDescForProbe`, `getInlineTreeNode`, `getFuncDescForGUID`; this block decodes machine-code bytes into symbolic instruction form; returns subsystem-specific computed results.
- **CN**: 围绕 `getInlinerDescForProbe`, `getInlineTreeNode`, `getFuncDescForGUID` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，返回子系统相关的计算结果。

## Key Concepts / 关键概念

- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据
- **Object layout / 目标文件布局**:
  - **EN**: Organizes fragments into sections and computes final offsets
  - **CN**: 把片段组织到节区中并计算最终偏移

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/MC/MCPseudoProbe.h`, `llvm/ADT/STLExtras.h`, `llvm/IR/PseudoProbe.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCAssembler.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCObjectFileInfo.h`, `llvm/MC/MCObjectStreamer.h`, `llvm/MC/MCSymbol.h`, `llvm/Support/Endian.h`, `llvm/Support/Error.h` ... (+9 more)
- **LLVM subsystems / LLVM 子系统**: MC, IR, Support
