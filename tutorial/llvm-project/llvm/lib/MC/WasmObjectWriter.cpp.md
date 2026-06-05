# WasmObjectWriter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/WasmObjectWriter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Wasm object file writer information.
  - **CN**: 实现 MC 层中某一特定目标文件格式的写出器支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32
```cpp
//===- lib/MC/WasmObjectWriter.cpp - Wasm File Writer ---------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements Wasm object file writer information.
//
//===----------------------------------------------------------------------===//

#include "llvm/ADT/STLExtras.h"
#include "llvm/BinaryFormat/Wasm.h"
#include "llvm/BinaryFormat/WasmTraits.h"
#include "llvm/Config/llvm-config.h"
#include "llvm/MC/MCAsmBackend.h"
#include "llvm/MC/MCAssembler.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/MC/MCSectionWasm.h"
#include "llvm/MC/MCSymbolWasm.h"
#include "llvm/MC/MCValue.h"
#include "llvm/MC/MCWasmObjectWriter.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/EndianStream.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/LEB128.h"
#include <vector>

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ADT/STLExtras.h`, `llvm/BinaryFormat/Wasm.h`, `llvm/BinaryFormat/WasmTraits.h`, `llvm/Config/llvm-config.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ADT/STLExtras.h`, `llvm/BinaryFormat/Wasm.h`, `llvm/BinaryFormat/WasmTraits.h`, `llvm/Config/llvm-config.h`。

### Lines 33-55
```cpp
using namespace llvm;

#define DEBUG_TYPE "mc"

namespace {

// When we create the indirect function table we start at 1, so that there is
// and empty slot at 0 and therefore calling a null function pointer will trap.
static const uint32_t InitialTableOffset = 1;

// For patching purposes, we need to remember where each section starts, both
// for patching up the section size field, and for patching up references to
// locations within the section.
struct SectionBookkeeping {
  // Where the size of the section is written.
  uint64_t SizeOffset;
  // Where the section header ends (without custom section name).
  uint64_t PayloadOffset;
  // Where the contents of the section starts.
  uint64_t ContentsOffset;
  uint32_t Index;
};

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 56-75
```cpp
// A wasm data segment.  A wasm binary contains only a single data section
// but that can contain many segments, each with their own virtual location
// in memory.  Each MCSection data created by llvm is modeled as its own
// wasm data segment.
struct WasmDataSegment {
  MCSectionWasm *Section;
  StringRef Name;
  uint32_t InitFlags;
  uint64_t Offset;
  uint32_t Alignment;
  uint32_t LinkingFlags;
  SmallVector<char, 4> Data;
};

// A wasm function to be written into the function section.
struct WasmFunction {
  uint32_t SigIndex;
  MCSection *Section;
};

```
- **EN**: Introduces declarations for `WasmDataSegment`, `WasmFunction`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `WasmDataSegment`, `WasmFunction` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 76-97
```cpp
// A wasm global to be written into the global section.
struct WasmGlobal {
  wasm::WasmGlobalType Type;
  uint64_t InitialValue;
};

// Information about a single item which is part of a COMDAT.  For each data
// segment or function which is in the COMDAT, there is a corresponding
// WasmComdatEntry.
struct WasmComdatEntry {
  unsigned Kind;
  uint32_t Index;
};

// Information about a single relocation.
struct WasmRelocationEntry {
  uint64_t Offset;                   // Where is the relocation.
  const MCSymbolWasm *Symbol;        // The symbol to relocate with.
  int64_t Addend;                    // A value to add to the symbol.
  unsigned Type;                     // The type of the relocation.
  const MCSectionWasm *FixupSection; // The section the relocation is targeting.

```
- **EN**: Introduces declarations for `WasmGlobal`, `WasmComdatEntry`, `WasmRelocationEntry`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `WasmGlobal`, `WasmComdatEntry`, `WasmRelocationEntry` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 98-118
```cpp
  WasmRelocationEntry(uint64_t Offset, const MCSymbolWasm *Symbol,
                      int64_t Addend, unsigned Type,
                      const MCSectionWasm *FixupSection)
      : Offset(Offset), Symbol(Symbol), Addend(Addend), Type(Type),
        FixupSection(FixupSection) {}

  bool hasAddend() const { return wasm::relocTypeHasAddend(Type); }

  void print(raw_ostream &Out) const {
    Out << wasm::relocTypetoString(Type) << " Off=" << Offset
        << ", Sym=" << *Symbol << ", Addend=" << Addend
        << ", FixupSection=" << FixupSection->getName();
  }

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
  LLVM_DUMP_METHOD void dump() const { print(dbgs()); }
#endif
};

static const uint32_t InvalidIndex = -1;

```
- **EN**: Implements logic around `WasmRelocationEntry`, `Offset`, `FixupSection`, `hasAddend`, and 4 more symbols; this block handles relocation, fixup, or symbol-resolution work; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `WasmRelocationEntry`, `Offset`, `FixupSection`, `hasAddend`, and 4 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 119-147
```cpp
struct WasmCustomSection {

  StringRef Name;
  MCSectionWasm *Section;

  uint32_t OutputContentsOffset = 0;
  uint32_t OutputIndex = InvalidIndex;

  WasmCustomSection(StringRef Name, MCSectionWasm *Section)
      : Name(Name), Section(Section) {}
};

#if !defined(NDEBUG)
raw_ostream &operator<<(raw_ostream &OS, const WasmRelocationEntry &Rel) {
  Rel.print(OS);
  return OS;
}
#endif

// Write Value as an (unsigned) LEB value at offset Offset in Stream, padded
// to allow patching.
template <typename T, int W>
void writePatchableULEB(raw_pwrite_stream &Stream, T Value, uint64_t Offset) {
  uint8_t Buffer[W];
  unsigned SizeLen = encodeULEB128(Value, Buffer, W);
  assert(SizeLen == W);
  Stream.pwrite((char *)Buffer, SizeLen, Offset);
}

```
- **EN**: Introduces declarations for `WasmCustomSection`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `WasmCustomSection` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 148-167
```cpp
// Write Value as an signed LEB value at offset Offset in Stream, padded
// to allow patching.
template <typename T, int W>
void writePatchableSLEB(raw_pwrite_stream &Stream, T Value, uint64_t Offset) {
  uint8_t Buffer[W];
  unsigned SizeLen = encodeSLEB128(Value, Buffer, W);
  assert(SizeLen == W);
  Stream.pwrite((char *)Buffer, SizeLen, Offset);
}

static void writePatchableU32(raw_pwrite_stream &Stream, uint32_t Value,
                              uint64_t Offset) {
  writePatchableULEB<uint32_t, 5>(Stream, Value, Offset);
}

static void writePatchableS32(raw_pwrite_stream &Stream, int32_t Value,
                              uint64_t Offset) {
  writePatchableSLEB<int32_t, 5>(Stream, Value, Offset);
}

```
- **EN**: Implements logic around `writePatchableSLEB`, `encodeSLEB128`, `assert`, `pwrite`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `writePatchableSLEB`, `encodeSLEB128`, `assert`, `pwrite`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 168-192
```cpp
static void writePatchableU64(raw_pwrite_stream &Stream, uint64_t Value,
                              uint64_t Offset) {
  writePatchableSLEB<uint64_t, 10>(Stream, Value, Offset);
}

static void writePatchableS64(raw_pwrite_stream &Stream, int64_t Value,
                              uint64_t Offset) {
  writePatchableSLEB<int64_t, 10>(Stream, Value, Offset);
}

// Write Value as a plain integer value at offset Offset in Stream.
static void patchI32(raw_pwrite_stream &Stream, uint32_t Value,
                     uint64_t Offset) {
  uint8_t Buffer[4];
  support::endian::write32le(Buffer, Value);
  Stream.pwrite((char *)Buffer, sizeof(Buffer), Offset);
}

static void patchI64(raw_pwrite_stream &Stream, uint64_t Value,
                     uint64_t Offset) {
  uint8_t Buffer[8];
  support::endian::write64le(Buffer, Value);
  Stream.pwrite((char *)Buffer, sizeof(Buffer), Offset);
}

```
- **EN**: Implements logic around `writePatchableU64`, `writePatchableS64`, `patchI32`, `write32le`, and 3 more symbols; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `writePatchableU64`, `writePatchableS64`, `patchI32`, `write32le`, and 3 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 193-220
```cpp
bool isDwoSection(const MCSection &Sec) {
  return Sec.getName().ends_with(".dwo");
}

class WasmObjectWriter : public MCObjectWriter {
  support::endian::Writer *W = nullptr;

  /// The target specific Wasm writer instance.
  std::unique_ptr<MCWasmObjectTargetWriter> TargetObjectWriter;

  // Relocations for fixing up references in the code section.
  std::vector<WasmRelocationEntry> CodeRelocations;
  // Relocations for fixing up references in the data section.
  std::vector<WasmRelocationEntry> DataRelocations;

  // Index values to use for fixing up call_indirect type indices.
  // Maps function symbols to the index of the type of the function
  DenseMap<const MCSymbolWasm *, uint32_t> TypeIndices;
  // Maps function symbols to the table element index space. Used
  // for TABLE_INDEX relocation types (i.e. address taken functions).
  DenseMap<const MCSymbolWasm *, uint32_t> TableIndices;
  // Maps function/global/table symbols to the
  // function/global/table/tag/section index space.
  DenseMap<const MCSymbolWasm *, uint32_t> WasmIndices;
  DenseMap<const MCSymbolWasm *, uint32_t> GOTIndices;
  // Maps data symbols to the Wasm segment and offset/size with the segment.
  DenseMap<const MCSymbolWasm *, wasm::WasmDataReference> DataLocations;

```
- **EN**: Introduces declarations for `WasmObjectWriter`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `WasmObjectWriter` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 221-241
```cpp
  // Stores output data (index, relocations, content offset) for custom
  // section.
  std::vector<WasmCustomSection> CustomSections;
  std::unique_ptr<WasmCustomSection> ProducersSection;
  std::unique_ptr<WasmCustomSection> TargetFeaturesSection;
  // Relocations for fixing up references in the custom sections.
  DenseMap<const MCSectionWasm *, std::vector<WasmRelocationEntry>>
      CustomSectionsRelocations;

  // Map from section to defining function symbol.
  DenseMap<const MCSection *, const MCSymbol *> SectionFunctions;

  DenseMap<wasm::WasmSignature, uint32_t> SignatureIndices;
  SmallVector<wasm::WasmSignature, 4> Signatures;
  SmallVector<WasmDataSegment, 4> DataSegments;
  unsigned NumFunctionImports = 0;
  unsigned NumGlobalImports = 0;
  unsigned NumTableImports = 0;
  unsigned NumTagImports = 0;
  uint32_t SectionCount = 0;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 242-263
```cpp
  enum class DwoMode {
    AllSections,
    NonDwoOnly,
    DwoOnly,
  };
  bool IsSplitDwarf = false;
  raw_pwrite_stream *OS = nullptr;
  raw_pwrite_stream *DwoOS = nullptr;

  // TargetObjectWriter wranppers.
  bool is64Bit() const { return TargetObjectWriter->is64Bit(); }
  bool isEmscripten() const { return TargetObjectWriter->isEmscripten(); }

  void startSection(SectionBookkeeping &Section, unsigned SectionId);
  void startCustomSection(SectionBookkeeping &Section, StringRef Name);
  void endSection(SectionBookkeeping &Section);

public:
  WasmObjectWriter(std::unique_ptr<MCWasmObjectTargetWriter> MOTW,
                   raw_pwrite_stream &OS_)
      : TargetObjectWriter(std::move(MOTW)), OS(&OS_) {}

```
- **EN**: Introduces declarations for `DwoMode`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `DwoMode` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 264-291
```cpp
  WasmObjectWriter(std::unique_ptr<MCWasmObjectTargetWriter> MOTW,
                   raw_pwrite_stream &OS_, raw_pwrite_stream &DwoOS_)
      : TargetObjectWriter(std::move(MOTW)), IsSplitDwarf(true), OS(&OS_),
        DwoOS(&DwoOS_) {}

private:
  void reset() override {
    CodeRelocations.clear();
    DataRelocations.clear();
    TypeIndices.clear();
    WasmIndices.clear();
    GOTIndices.clear();
    TableIndices.clear();
    DataLocations.clear();
    CustomSections.clear();
    ProducersSection.reset();
    TargetFeaturesSection.reset();
    CustomSectionsRelocations.clear();
    SignatureIndices.clear();
    Signatures.clear();
    DataSegments.clear();
    SectionFunctions.clear();
    NumFunctionImports = 0;
    NumGlobalImports = 0;
    NumTableImports = 0;
    MCObjectWriter::reset();
  }

```
- **EN**: Implements logic around `WasmObjectWriter`, `TargetObjectWriter`, `DwoOS`, `reset`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `WasmObjectWriter`, `TargetObjectWriter`, `DwoOS`, `reset`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 292-316
```cpp
  void writeHeader(const MCAssembler &Asm);

  void recordRelocation(const MCFragment &F, const MCFixup &Fixup,
                        MCValue Target, uint64_t &FixedValue) override;

  void executePostLayoutBinding() override;
  void prepareImports(SmallVectorImpl<wasm::WasmImport> &Imports,
                      MCAssembler &Asm);
  uint64_t writeObject() override;

  uint64_t writeOneObject(MCAssembler &Asm, DwoMode Mode);

  void writeString(const StringRef Str) {
    encodeULEB128(Str.size(), W->OS);
    W->OS << Str;
  }

  void writeStringWithAlignment(const StringRef Str, unsigned Alignment);

  void writeI32(int32_t val) {
    char Buffer[4];
    support::endian::write32le(Buffer, val);
    W->OS.write(Buffer, sizeof(Buffer));
  }

```
- **EN**: Implements logic around `writeHeader`, `recordRelocation`, `executePostLayoutBinding`, `prepareImports`, and 8 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `writeHeader`, `recordRelocation`, `executePostLayoutBinding`, `prepareImports`, and 8 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为。

### Lines 317-348
```cpp
  void writeI64(int64_t val) {
    char Buffer[8];
    support::endian::write64le(Buffer, val);
    W->OS.write(Buffer, sizeof(Buffer));
  }

  void writeValueType(wasm::ValType Ty) { W->OS << static_cast<char>(Ty); }

  void writeTypeSection(ArrayRef<wasm::WasmSignature> Signatures);
  void writeImportSection(ArrayRef<wasm::WasmImport> Imports, uint64_t DataSize,
                          uint32_t NumElements);
  void writeFunctionSection(ArrayRef<WasmFunction> Functions);
  void writeExportSection(ArrayRef<wasm::WasmExport> Exports);
  void writeElemSection(const MCSymbolWasm *IndirectFunctionTable,
                        ArrayRef<uint32_t> TableElems);
  void writeDataCountSection();
  uint32_t writeCodeSection(const MCAssembler &Asm,
                            ArrayRef<WasmFunction> Functions);
  uint32_t writeDataSection(const MCAssembler &Asm);
  void writeTagSection(ArrayRef<uint32_t> TagTypes);
  void writeGlobalSection(ArrayRef<wasm::WasmGlobal> Globals);
  void writeTableSection(ArrayRef<wasm::WasmTable> Tables);
  void writeRelocSection(uint32_t SectionIndex, StringRef Name,
                         std::vector<WasmRelocationEntry> &Relocations);
  void writeLinkingMetaDataSection(
      ArrayRef<wasm::WasmSymbolInfo> SymbolInfos,
      ArrayRef<std::pair<uint16_t, uint32_t>> InitFuncs,
      const std::map<StringRef, std::vector<WasmComdatEntry>> &Comdats);
  void writeCustomSection(WasmCustomSection &CustomSection,
                          const MCAssembler &Asm);
  void writeCustomRelocSections();

```
- **EN**: Implements logic around `writeI64`, `write64le`, `write`, `writeValueType`, and 15 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `writeI64`, `write64le`, `write`, `writeValueType`, and 15 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 349-368
```cpp
  uint64_t getProvisionalValue(const MCAssembler &Asm,
                               const WasmRelocationEntry &RelEntry);
  void applyRelocations(ArrayRef<WasmRelocationEntry> Relocations,
                        uint64_t ContentsOffset, const MCAssembler &Asm);

  uint32_t getRelocationIndexValue(const WasmRelocationEntry &RelEntry);
  uint32_t getFunctionType(const MCSymbolWasm &Symbol);
  uint32_t getTagType(const MCSymbolWasm &Symbol);
  void registerFunctionType(const MCSymbolWasm &Symbol);
  void registerTagType(const MCSymbolWasm &Symbol);
};

} // end anonymous namespace

// Write out a section header and a patchable section size field.
void WasmObjectWriter::startSection(SectionBookkeeping &Section,
                                    unsigned SectionId) {
  LLVM_DEBUG(dbgs() << "startSection " << SectionId << "\n");
  W->OS << char(SectionId);

```
- **EN**: Implements logic around `getProvisionalValue`, `applyRelocations`, `getRelocationIndexValue`, `getFunctionType`, and 5 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `getProvisionalValue`, `applyRelocations`, `getRelocationIndexValue`, `getFunctionType`, and 5 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 369-393
```cpp
  Section.SizeOffset = W->OS.tell();

  // The section size. We don't know the size yet, so reserve enough space
  // for any 32-bit value; we'll patch it later.
  encodeULEB128(0, W->OS, 5);

  // The position where the section starts, for measuring its size.
  Section.ContentsOffset = W->OS.tell();
  Section.PayloadOffset = W->OS.tell();
  Section.Index = SectionCount++;
}

// Write a string with extra paddings for trailing alignment
// TODO: support alignment at asm and llvm level?
void WasmObjectWriter::writeStringWithAlignment(const StringRef Str,
                                                unsigned Alignment) {

  // Calculate the encoded size of str length and add pads based on it and
  // alignment.
  raw_null_ostream NullOS;
  uint64_t StrSizeLength = encodeULEB128(Str.size(), NullOS);
  uint64_t Offset = W->OS.tell() + StrSizeLength + Str.size();
  uint64_t Paddings = offsetToAlignment(Offset, Align(Alignment));
  Offset += Paddings;

```
- **EN**: Implements logic around `tell`, `encodeULEB128`, `writeStringWithAlignment`, `offsetToAlignment`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `tell`, `encodeULEB128`, `writeStringWithAlignment`, `offsetToAlignment` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 394-418
```cpp
  // LEB128 greater than 5 bytes is invalid
  assert((StrSizeLength + Paddings) <= 5 && "too long string to align");

  encodeSLEB128(Str.size(), W->OS, StrSizeLength + Paddings);
  W->OS << Str;

  assert(W->OS.tell() == Offset && "invalid padding");
}

void WasmObjectWriter::startCustomSection(SectionBookkeeping &Section,
                                          StringRef Name) {
  LLVM_DEBUG(dbgs() << "startCustomSection " << Name << "\n");
  startSection(Section, wasm::WASM_SEC_CUSTOM);

  // The position where the section header ends, for measuring its size.
  Section.PayloadOffset = W->OS.tell();

  // Custom sections in wasm also have a string identifier.
  if (Name != "__clangast") {
    writeString(Name);
  } else {
    // The on-disk hashtable in clangast needs to be aligned by 4 bytes.
    writeStringWithAlignment(Name, 4);
  }

```
- **EN**: Implements logic around `assert`, `encodeSLEB128`, `startCustomSection`, `startSection`, and 3 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `assert`, `encodeSLEB128`, `startCustomSection`, `startSection`, and 3 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 419-443
```cpp
  // The position where the custom section starts.
  Section.ContentsOffset = W->OS.tell();
}

// Now that the section is complete and we know how big it is, patch up the
// section size field at the start of the section.
void WasmObjectWriter::endSection(SectionBookkeeping &Section) {
  uint64_t Size = W->OS.tell();
  // /dev/null doesn't support seek/tell and can report offset of 0.
  // Simply skip this patching in that case.
  if (!Size)
    return;

  Size -= Section.PayloadOffset;
  if (uint32_t(Size) != Size)
    report_fatal_error("section size does not fit in a uint32_t");

  LLVM_DEBUG(dbgs() << "endSection size=" << Size << "\n");

  // Write the final section size to the payload_len field, which follows
  // the section id byte.
  writePatchableU32(static_cast<raw_pwrite_stream &>(W->OS), Size,
                    Section.SizeOffset);
}

```
- **EN**: Implements logic around `tell`, `endSection`, `report_fatal_error`, `writePatchableU32`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `tell`, `endSection`, `report_fatal_error`, `writePatchableU32` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 444-476
```cpp
// Emit the Wasm header.
void WasmObjectWriter::writeHeader(const MCAssembler &Asm) {
  W->OS.write(wasm::WasmMagic, sizeof(wasm::WasmMagic));
  W->write<uint32_t>(wasm::WasmVersion);
}

void WasmObjectWriter::executePostLayoutBinding() {
  // Some compilation units require the indirect function table to be present
  // but don't explicitly reference it.  This is the case for call_indirect
  // without the reference-types feature, and also function bitcasts in all
  // cases.  In those cases the __indirect_function_table has the
  // WASM_SYMBOL_NO_STRIP attribute.  Here we make sure this symbol makes it to
  // the assembler, if needed.
  if (auto *Sym = Asm->getContext().lookupSymbol("__indirect_function_table")) {
    const auto *WasmSym = static_cast<const MCSymbolWasm *>(Sym);
    if (WasmSym->isNoStrip())
      Asm->registerSymbol(*Sym);
  }

  // Build a map of sections to the function that defines them, for use
  // in recordRelocation.
  for (const MCSymbol &S : Asm->symbols()) {
    const auto &WS = static_cast<const MCSymbolWasm &>(S);
    if (WS.isDefined() && WS.isFunction() && !WS.isVariable()) {
      const auto &Sec = static_cast<const MCSectionWasm &>(S.getSection());
      auto Pair = SectionFunctions.insert(std::make_pair(&Sec, &S));
      if (!Pair.second)
        report_fatal_error("section already has a defining function: " +
                           Sec.getName());
    }
  }
}

```
- **EN**: Implements logic around `writeHeader`, `write`, `write<uint32_t>`, `executePostLayoutBinding`, and 5 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `writeHeader`, `write`, `write<uint32_t>`, `executePostLayoutBinding`, and 5 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 477-499
```cpp
void WasmObjectWriter::recordRelocation(const MCFragment &F,
                                        const MCFixup &Fixup, MCValue Target,
                                        uint64_t &FixedValue) {
  // The WebAssembly backend should never generate FKF_IsPCRel fixups
  assert(!Fixup.isPCRel());

  const auto &FixupSection = static_cast<MCSectionWasm &>(*F.getParent());
  uint64_t C = Target.getConstant();
  uint64_t FixupOffset = Asm->getFragmentOffset(F) + Fixup.getOffset();
  MCContext &Ctx = getContext();
  bool IsLocRel = false;

  if (const auto *RefB = Target.getSubSym()) {
    auto &SymB = static_cast<const MCSymbolWasm &>(*RefB);

    if (FixupSection.isText()) {
      Ctx.reportError(Fixup.getLoc(),
                      Twine("symbol '") + SymB.getName() +
                          "' unsupported subtraction expression used in "
                          "relocation in code section.");
      return;
    }

```
- **EN**: Implements logic around `recordRelocation`, `assert`, `getParent`, `getConstant`, and 4 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `recordRelocation`, `assert`, `getParent`, `getConstant`, and 4 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 500-519
```cpp
    if (SymB.isUndefined()) {
      Ctx.reportError(Fixup.getLoc(),
                      Twine("symbol '") + SymB.getName() +
                          "' can not be undefined in a subtraction expression");
      return;
    }
    const MCSection &SecB = SymB.getSection();
    if (&SecB != &FixupSection) {
      Ctx.reportError(Fixup.getLoc(),
                      Twine("symbol '") + SymB.getName() +
                          "' can not be placed in a different section");
      return;
    }
    IsLocRel = true;
    C += FixupOffset - Asm->getSymbolOffset(SymB);
  }

  // We either rejected the fixup or folded B into C at this point.
  auto *SymA = static_cast<const MCSymbolWasm *>(Target.getAddSym());

```
- **EN**: Implements logic around `reportError`, `Twine`, `getSection`, `getSymbolOffset`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; updates MC section or symbol state.
- **CN**: 围绕 `reportError`, `Twine`, `getSection`, `getSymbolOffset`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，更新 MC 节区或符号状态。

### Lines 520-549
```cpp
  // The .init_array isn't translated as data, so don't do relocations in it.
  if (FixupSection.getName().starts_with(".init_array")) {
    SymA->setUsedInInitArray();
    return;
  }

  // Put any constant offset in an addend. Offsets can be negative, and
  // LLVM expects wrapping, in contrast to wasm's immediates which can't
  // be negative and don't wrap.
  FixedValue = 0;

  unsigned Type;
  if (mc::isRelocRelocation(Fixup.getKind()))
    Type = Fixup.getKind() - FirstLiteralRelocationKind;
  else
    Type =
        TargetObjectWriter->getRelocType(Target, Fixup, FixupSection, IsLocRel);

  // Absolute offset within a section or a function.
  // Currently only supported for metadata sections.
  // See: test/MC/WebAssembly/blockaddress.ll
  if ((Type == wasm::R_WASM_FUNCTION_OFFSET_I32 ||
       Type == wasm::R_WASM_FUNCTION_OFFSET_I64 ||
       Type == wasm::R_WASM_SECTION_OFFSET_I32) &&
      SymA->isDefined()) {
    // SymA can be a temp data symbol that represents a function (in which case
    // it needs to be replaced by the section symbol), [XXX and it apparently
    // later gets changed again to a func symbol?] or it can be a real
    // function symbol, in which case it can be left as-is.

```
- **EN**: Implements logic around `setUsedInInitArray`, `getKind`, `getRelocType`, `isDefined`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `setUsedInInitArray`, `getKind`, `getRelocType`, `isDefined` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 550-570
```cpp
    if (!FixupSection.isMetadata())
      report_fatal_error("relocations for function or section offsets are "
                         "only supported in metadata sections");

    const MCSymbol *SectionSymbol = nullptr;
    const MCSection &SecA = SymA->getSection();
    if (SecA.isText()) {
      auto SecSymIt = SectionFunctions.find(&SecA);
      if (SecSymIt == SectionFunctions.end())
        report_fatal_error("section doesn\'t have defining symbol");
      SectionSymbol = SecSymIt->second;
    } else {
      SectionSymbol = SecA.getBeginSymbol();
    }
    if (!SectionSymbol)
      report_fatal_error("section symbol is required for relocation");

    C += Asm->getSymbolOffset(*SymA);
    SymA = static_cast<const MCSymbolWasm *>(SectionSymbol);
  }

```
- **EN**: Implements logic around `report_fatal_error`, `getSection`, `find`, `getBeginSymbol`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; updates MC section or symbol state.
- **CN**: 围绕 `report_fatal_error`, `getSection`, `find`, `getBeginSymbol`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，更新 MC 节区或符号状态。

### Lines 571-591
```cpp
  if (Type == wasm::R_WASM_TABLE_INDEX_REL_SLEB ||
      Type == wasm::R_WASM_TABLE_INDEX_REL_SLEB64 ||
      Type == wasm::R_WASM_TABLE_INDEX_SLEB ||
      Type == wasm::R_WASM_TABLE_INDEX_SLEB64 ||
      Type == wasm::R_WASM_TABLE_INDEX_I32 ||
      Type == wasm::R_WASM_TABLE_INDEX_I64) {
    // TABLE_INDEX relocs implicitly use the default indirect function table.
    // We require the function table to have already been defined.
    auto TableName = "__indirect_function_table";
    auto *Sym = static_cast<MCSymbolWasm *>(Ctx.lookupSymbol(TableName));
    if (!Sym) {
      report_fatal_error("missing indirect function table symbol");
    } else {
      if (!Sym->isFunctionTable())
        report_fatal_error("__indirect_function_table symbol has wrong type");
      // Ensure that __indirect_function_table reaches the output.
      Sym->setNoStrip();
      Asm->registerSymbol(*Sym);
    }
  }

```
- **EN**: Implements logic around `lookupSymbol`, `report_fatal_error`, `setNoStrip`, `registerSymbol`; this block handles relocation, fixup, or symbol-resolution work; updates MC section or symbol state.
- **CN**: 围绕 `lookupSymbol`, `report_fatal_error`, `setNoStrip`, `registerSymbol` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，更新 MC 节区或符号状态。

### Lines 592-615
```cpp
  // Relocation other than R_WASM_TYPE_INDEX_LEB are required to be
  // against a named symbol.
  if (Type != wasm::R_WASM_TYPE_INDEX_LEB) {
    if (SymA->getName().empty())
      report_fatal_error("relocations against un-named temporaries are not yet "
                         "supported by wasm");

    SymA->setUsedInReloc();
  }

  WasmRelocationEntry Rec(FixupOffset, SymA, C, Type, &FixupSection);
  LLVM_DEBUG(dbgs() << "WasmReloc: " << Rec << "\n");

  if (FixupSection.isWasmData()) {
    DataRelocations.push_back(Rec);
  } else if (FixupSection.isText()) {
    CodeRelocations.push_back(Rec);
  } else if (FixupSection.isMetadata()) {
    CustomSectionsRelocations[&FixupSection].push_back(Rec);
  } else {
    llvm_unreachable("unexpected section type");
  }
}

```
- **EN**: Implements logic around `report_fatal_error`, `setUsedInReloc`, `Rec`, `push_back`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; updates MC section or symbol state.
- **CN**: 围绕 `report_fatal_error`, `setUsedInReloc`, `Rec`, `push_back`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，更新 MC 节区或符号状态。

### Lines 616-655
```cpp
// Compute a value to write into the code at the location covered
// by RelEntry. This value isn't used by the static linker; it just serves
// to make the object format more readable and more likely to be directly
// useable.
uint64_t
WasmObjectWriter::getProvisionalValue(const MCAssembler &Asm,
                                      const WasmRelocationEntry &RelEntry) {
  if ((RelEntry.Type == wasm::R_WASM_GLOBAL_INDEX_LEB ||
       RelEntry.Type == wasm::R_WASM_GLOBAL_INDEX_I32) &&
      !RelEntry.Symbol->isGlobal()) {
    assert(GOTIndices.contains(RelEntry.Symbol) && "symbol not found in GOT");
    return GOTIndices[RelEntry.Symbol];
  }

  switch (RelEntry.Type) {
  case wasm::R_WASM_TABLE_INDEX_REL_SLEB:
  case wasm::R_WASM_TABLE_INDEX_REL_SLEB64:
  case wasm::R_WASM_TABLE_INDEX_SLEB:
  case wasm::R_WASM_TABLE_INDEX_SLEB64:
  case wasm::R_WASM_TABLE_INDEX_I32:
  case wasm::R_WASM_TABLE_INDEX_I64: {
    // Provisional value is table address of the resolved symbol itself
    auto *Base =
        static_cast<const MCSymbolWasm *>(Asm.getBaseSymbol(*RelEntry.Symbol));
    assert(Base->isFunction());
    if (RelEntry.Type == wasm::R_WASM_TABLE_INDEX_REL_SLEB ||
        RelEntry.Type == wasm::R_WASM_TABLE_INDEX_REL_SLEB64)
      return TableIndices[Base] - InitialTableOffset;
    else
      return TableIndices[Base];
  }
  case wasm::R_WASM_TYPE_INDEX_LEB:
    // Provisional value is same as the index
    return getRelocationIndexValue(RelEntry);
  case wasm::R_WASM_FUNCTION_INDEX_LEB:
  case wasm::R_WASM_FUNCTION_INDEX_I32:
  case wasm::R_WASM_GLOBAL_INDEX_LEB:
  case wasm::R_WASM_GLOBAL_INDEX_I32:
  case wasm::R_WASM_TAG_INDEX_LEB:
  case wasm::R_WASM_TABLE_NUMBER_LEB:
```
- **EN**: Implements logic around `getProvisionalValue`, `isGlobal`, `assert`, `getBaseSymbol`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getProvisionalValue`, `isGlobal`, `assert`, `getBaseSymbol`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 656-693
```cpp
    // Provisional value is function/global/tag Wasm index
    assert(WasmIndices.contains(RelEntry.Symbol) &&
           "symbol not found in wasm index space");
    return WasmIndices[RelEntry.Symbol];
  case wasm::R_WASM_FUNCTION_OFFSET_I32:
  case wasm::R_WASM_FUNCTION_OFFSET_I64:
  case wasm::R_WASM_SECTION_OFFSET_I32: {
    if (!RelEntry.Symbol->isDefined())
      return 0;
    const auto &Section =
        static_cast<const MCSectionWasm &>(RelEntry.Symbol->getSection());
    return Section.getSectionOffset() + RelEntry.Addend;
  }
  case wasm::R_WASM_MEMORY_ADDR_LEB:
  case wasm::R_WASM_MEMORY_ADDR_LEB64:
  case wasm::R_WASM_MEMORY_ADDR_SLEB:
  case wasm::R_WASM_MEMORY_ADDR_SLEB64:
  case wasm::R_WASM_MEMORY_ADDR_REL_SLEB:
  case wasm::R_WASM_MEMORY_ADDR_REL_SLEB64:
  case wasm::R_WASM_MEMORY_ADDR_I32:
  case wasm::R_WASM_MEMORY_ADDR_I64:
  case wasm::R_WASM_MEMORY_ADDR_TLS_SLEB:
  case wasm::R_WASM_MEMORY_ADDR_TLS_SLEB64:
  case wasm::R_WASM_MEMORY_ADDR_LOCREL_I32: {
    // Provisional value is address of the global plus the offset
    // For undefined symbols, use zero
    if (!RelEntry.Symbol->isDefined())
      return 0;
    const wasm::WasmDataReference &SymRef = DataLocations[RelEntry.Symbol];
    const WasmDataSegment &Segment = DataSegments[SymRef.Segment];
    // Ignore overflow. LLVM allows address arithmetic to silently wrap.
    return Segment.Offset + SymRef.Offset + RelEntry.Addend;
  }
  default:
    llvm_unreachable("invalid relocation type");
  }
}

```
- **EN**: Implements logic around `assert`, `getSection`, `getSectionOffset`, `llvm_unreachable`; this block handles relocation, fixup, or symbol-resolution work; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `assert`, `getSection`, `getSectionOffset`, `llvm_unreachable` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 694-726
```cpp
static void addData(SmallVectorImpl<char> &DataBytes,
                    MCSectionWasm &DataSection) {
  LLVM_DEBUG(errs() << "addData: " << DataSection.getName() << "\n");

  DataBytes.resize(alignTo(DataBytes.size(), DataSection.getAlign()));

  for (const MCFragment &Frag : DataSection) {
    if (Frag.hasInstructions())
      report_fatal_error("only data supported in data sections");

    llvm::append_range(DataBytes, Frag.getContents());
    if (Frag.getKind() == MCFragment::FT_Align) {
      if (Frag.getAlignFillLen() != 1)
        report_fatal_error("only byte values supported for alignment");
      // If nops are requested, use zeros, as this is the data section.
      uint8_t Value = Frag.hasAlignEmitNops() ? 0 : Frag.getAlignFill();
      uint64_t Size =
          std::min<uint64_t>(alignTo(DataBytes.size(), Frag.getAlignment()),
                             DataBytes.size() + Frag.getAlignMaxBytesToEmit());
      DataBytes.resize(Size, Value);
    } else if (auto *Fill = dyn_cast<MCFillFragment>(&Frag)) {
      int64_t NumValues;
      if (!Fill->getNumValues().evaluateAsAbsolute(NumValues))
        llvm_unreachable("The fill should be an assembler constant");
      DataBytes.insert(DataBytes.end(), Fill->getValueSize() * NumValues,
                       Fill->getValue());
    } else if (Frag.getKind() == MCFragment::FT_LEB) {
      llvm::append_range(DataBytes, Frag.getVarContents());
    } else {
      assert(Frag.getKind() == MCFragment::FT_Data);
    }
  }

```
- **EN**: Implements logic around `addData`, `resize`, `report_fatal_error`, `append_range`, and 7 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `addData`, `resize`, `report_fatal_error`, `append_range`, and 7 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 727-753
```cpp
  LLVM_DEBUG(dbgs() << "addData -> " << DataBytes.size() << "\n");
}

uint32_t
WasmObjectWriter::getRelocationIndexValue(const WasmRelocationEntry &RelEntry) {
  if (RelEntry.Type == wasm::R_WASM_TYPE_INDEX_LEB) {
    auto It = TypeIndices.find(RelEntry.Symbol);
    if (It == TypeIndices.end())
      report_fatal_error("symbol not found in type index space: " +
                         RelEntry.Symbol->getName());
    return It->second;
  }

  return RelEntry.Symbol->getIndex();
}

// Apply the portions of the relocation records that we can handle ourselves
// directly.
void WasmObjectWriter::applyRelocations(
    ArrayRef<WasmRelocationEntry> Relocations, uint64_t ContentsOffset,
    const MCAssembler &Asm) {
  auto &Stream = static_cast<raw_pwrite_stream &>(W->OS);
  for (const WasmRelocationEntry &RelEntry : Relocations) {
    uint64_t Offset = ContentsOffset +
                      RelEntry.FixupSection->getSectionOffset() +
                      RelEntry.Offset;

```
- **EN**: Implements logic around `getRelocationIndexValue`, `find`, `report_fatal_error`, `getName`, and 3 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getRelocationIndexValue`, `find`, `report_fatal_error`, `getName`, and 3 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 754-793
```cpp
    LLVM_DEBUG(dbgs() << "applyRelocation: " << RelEntry << "\n");
    uint64_t Value = getProvisionalValue(Asm, RelEntry);

    switch (RelEntry.Type) {
    case wasm::R_WASM_FUNCTION_INDEX_LEB:
    case wasm::R_WASM_TYPE_INDEX_LEB:
    case wasm::R_WASM_GLOBAL_INDEX_LEB:
    case wasm::R_WASM_MEMORY_ADDR_LEB:
    case wasm::R_WASM_TAG_INDEX_LEB:
    case wasm::R_WASM_TABLE_NUMBER_LEB:
      writePatchableU32(Stream, Value, Offset);
      break;
    case wasm::R_WASM_MEMORY_ADDR_LEB64:
      writePatchableU64(Stream, Value, Offset);
      break;
    case wasm::R_WASM_TABLE_INDEX_I32:
    case wasm::R_WASM_MEMORY_ADDR_I32:
    case wasm::R_WASM_FUNCTION_OFFSET_I32:
    case wasm::R_WASM_FUNCTION_INDEX_I32:
    case wasm::R_WASM_SECTION_OFFSET_I32:
    case wasm::R_WASM_GLOBAL_INDEX_I32:
    case wasm::R_WASM_MEMORY_ADDR_LOCREL_I32:
      patchI32(Stream, Value, Offset);
      break;
    case wasm::R_WASM_TABLE_INDEX_I64:
    case wasm::R_WASM_MEMORY_ADDR_I64:
    case wasm::R_WASM_FUNCTION_OFFSET_I64:
      patchI64(Stream, Value, Offset);
      break;
    case wasm::R_WASM_TABLE_INDEX_SLEB:
    case wasm::R_WASM_TABLE_INDEX_REL_SLEB:
    case wasm::R_WASM_MEMORY_ADDR_SLEB:
    case wasm::R_WASM_MEMORY_ADDR_REL_SLEB:
    case wasm::R_WASM_MEMORY_ADDR_TLS_SLEB:
      writePatchableS32(Stream, Value, Offset);
      break;
    case wasm::R_WASM_TABLE_INDEX_SLEB64:
    case wasm::R_WASM_TABLE_INDEX_REL_SLEB64:
    case wasm::R_WASM_MEMORY_ADDR_SLEB64:
    case wasm::R_WASM_MEMORY_ADDR_REL_SLEB64:
```
- **EN**: Implements logic around `getProvisionalValue`, `writePatchableU32`, `writePatchableU64`, `patchI32`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `getProvisionalValue`, `writePatchableU32`, `writePatchableU64`, `patchI32`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 794-822
```cpp
    case wasm::R_WASM_MEMORY_ADDR_TLS_SLEB64:
      writePatchableS64(Stream, Value, Offset);
      break;
    default:
      llvm_unreachable("invalid relocation type");
    }
  }
}

void WasmObjectWriter::writeTypeSection(
    ArrayRef<wasm::WasmSignature> Signatures) {
  if (Signatures.empty())
    return;

  SectionBookkeeping Section;
  startSection(Section, wasm::WASM_SEC_TYPE);

  encodeULEB128(Signatures.size(), W->OS);

  for (const wasm::WasmSignature &Sig : Signatures) {
    W->OS << char(wasm::WASM_TYPE_FUNC);
    encodeULEB128(Sig.Params.size(), W->OS);
    for (wasm::ValType Ty : Sig.Params)
      writeValueType(Ty);
    encodeULEB128(Sig.Returns.size(), W->OS);
    for (wasm::ValType Ty : Sig.Returns)
      writeValueType(Ty);
  }

```
- **EN**: Implements logic around `writePatchableS64`, `llvm_unreachable`, `writeTypeSection`, `startSection`, and 3 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `writePatchableS64`, `llvm_unreachable`, `writeTypeSection`, `startSection`, and 3 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 823-843
```cpp
  endSection(Section);
}

void WasmObjectWriter::writeImportSection(ArrayRef<wasm::WasmImport> Imports,
                                          uint64_t DataSize,
                                          uint32_t NumElements) {
  if (Imports.empty())
    return;

  uint64_t NumPages =
      (DataSize + wasm::WasmDefaultPageSize - 1) / wasm::WasmDefaultPageSize;

  SectionBookkeeping Section;
  startSection(Section, wasm::WASM_SEC_IMPORT);

  encodeULEB128(Imports.size(), W->OS);
  for (const wasm::WasmImport &Import : Imports) {
    writeString(Import.Module);
    writeString(Import.Field);
    W->OS << char(Import.Kind);

```
- **EN**: Implements logic around `endSection`, `writeImportSection`, `startSection`, `encodeULEB128`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `endSection`, `writeImportSection`, `startSection`, `encodeULEB128`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 844-869
```cpp
    switch (Import.Kind) {
    case wasm::WASM_EXTERNAL_FUNCTION:
      encodeULEB128(Import.SigIndex, W->OS);
      break;
    case wasm::WASM_EXTERNAL_GLOBAL:
      W->OS << char(Import.Global.Type);
      W->OS << char(Import.Global.Mutable ? 1 : 0);
      break;
    case wasm::WASM_EXTERNAL_MEMORY:
      encodeULEB128(Import.Memory.Flags, W->OS);
      encodeULEB128(NumPages, W->OS); // initial
      break;
    case wasm::WASM_EXTERNAL_TABLE:
      W->OS << char(Import.Table.ElemType);
      encodeULEB128(Import.Table.Limits.Flags, W->OS);
      encodeULEB128(NumElements, W->OS); // initial
      break;
    case wasm::WASM_EXTERNAL_TAG:
      W->OS << char(0); // Reserved 'attribute' field
      encodeULEB128(Import.SigIndex, W->OS);
      break;
    default:
      llvm_unreachable("unsupported import kind");
    }
  }

```
- **EN**: Implements logic around `encodeULEB128`, `char`, `llvm_unreachable`; this block uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `encodeULEB128`, `char`, `llvm_unreachable` 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 870-890
```cpp
  endSection(Section);
}

void WasmObjectWriter::writeFunctionSection(ArrayRef<WasmFunction> Functions) {
  if (Functions.empty())
    return;

  SectionBookkeeping Section;
  startSection(Section, wasm::WASM_SEC_FUNCTION);

  encodeULEB128(Functions.size(), W->OS);
  for (const WasmFunction &Func : Functions)
    encodeULEB128(Func.SigIndex, W->OS);

  endSection(Section);
}

void WasmObjectWriter::writeTagSection(ArrayRef<uint32_t> TagTypes) {
  if (TagTypes.empty())
    return;

```
- **EN**: Implements logic around `endSection`, `writeFunctionSection`, `startSection`, `encodeULEB128`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `endSection`, `writeFunctionSection`, `startSection`, `encodeULEB128`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 891-930
```cpp
  SectionBookkeeping Section;
  startSection(Section, wasm::WASM_SEC_TAG);

  encodeULEB128(TagTypes.size(), W->OS);
  for (uint32_t Index : TagTypes) {
    W->OS << char(0); // Reserved 'attribute' field
    encodeULEB128(Index, W->OS);
  }

  endSection(Section);
}

void WasmObjectWriter::writeGlobalSection(ArrayRef<wasm::WasmGlobal> Globals) {
  if (Globals.empty())
    return;

  SectionBookkeeping Section;
  startSection(Section, wasm::WASM_SEC_GLOBAL);

  encodeULEB128(Globals.size(), W->OS);
  for (const wasm::WasmGlobal &Global : Globals) {
    encodeULEB128(Global.Type.Type, W->OS);
    W->OS << char(Global.Type.Mutable);
    if (Global.InitExpr.Extended) {
      llvm_unreachable("extected init expressions not supported");
    } else {
      W->OS << char(Global.InitExpr.Inst.Opcode);
      switch (Global.Type.Type) {
      case wasm::WASM_TYPE_I32:
        encodeSLEB128(0, W->OS);
        break;
      case wasm::WASM_TYPE_I64:
        encodeSLEB128(0, W->OS);
        break;
      case wasm::WASM_TYPE_F32:
        writeI32(0);
        break;
      case wasm::WASM_TYPE_F64:
        writeI64(0);
        break;
```
- **EN**: Implements logic around `startSection`, `encodeULEB128`, `char`, `endSection`, and 5 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `startSection`, `encodeULEB128`, `char`, `endSection`, and 5 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 931-950
```cpp
      case wasm::WASM_TYPE_EXTERNREF:
        writeValueType(wasm::ValType::EXTERNREF);
        break;
      default:
        llvm_unreachable("unexpected type");
      }
    }
    W->OS << char(wasm::WASM_OPCODE_END);
  }

  endSection(Section);
}

void WasmObjectWriter::writeTableSection(ArrayRef<wasm::WasmTable> Tables) {
  if (Tables.empty())
    return;

  SectionBookkeeping Section;
  startSection(Section, wasm::WASM_SEC_TABLE);

```
- **EN**: Implements logic around `writeValueType`, `llvm_unreachable`, `char`, `endSection`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `writeValueType`, `llvm_unreachable`, `char`, `endSection`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 951-970
```cpp
  encodeULEB128(Tables.size(), W->OS);
  for (const wasm::WasmTable &Table : Tables) {
    assert(Table.Type.ElemType != wasm::ValType::OTHERREF &&
           "Cannot encode general ref-typed tables");
    encodeULEB128((uint32_t)Table.Type.ElemType, W->OS);
    encodeULEB128(Table.Type.Limits.Flags, W->OS);
    encodeULEB128(Table.Type.Limits.Minimum, W->OS);
    if (Table.Type.Limits.Flags & wasm::WASM_LIMITS_FLAG_HAS_MAX)
      encodeULEB128(Table.Type.Limits.Maximum, W->OS);
  }
  endSection(Section);
}

void WasmObjectWriter::writeExportSection(ArrayRef<wasm::WasmExport> Exports) {
  if (Exports.empty())
    return;

  SectionBookkeeping Section;
  startSection(Section, wasm::WASM_SEC_EXPORT);

```
- **EN**: Implements logic around `encodeULEB128`, `assert`, `endSection`, `writeExportSection`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `encodeULEB128`, `assert`, `endSection`, `writeExportSection`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 971-990
```cpp
  encodeULEB128(Exports.size(), W->OS);
  for (const wasm::WasmExport &Export : Exports) {
    writeString(Export.Name);
    W->OS << char(Export.Kind);
    encodeULEB128(Export.Index, W->OS);
  }

  endSection(Section);
}

void WasmObjectWriter::writeElemSection(
    const MCSymbolWasm *IndirectFunctionTable, ArrayRef<uint32_t> TableElems) {
  if (TableElems.empty())
    return;

  assert(IndirectFunctionTable);

  SectionBookkeeping Section;
  startSection(Section, wasm::WASM_SEC_ELEM);

```
- **EN**: Implements logic around `encodeULEB128`, `writeString`, `char`, `endSection`, and 3 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `encodeULEB128`, `writeString`, `char`, `endSection`, and 3 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 991-1014
```cpp
  encodeULEB128(1, W->OS); // number of "segments"

  assert(WasmIndices.contains(IndirectFunctionTable));
  uint32_t TableNumber = WasmIndices.find(IndirectFunctionTable)->second;
  uint32_t Flags = 0;
  if (TableNumber)
    Flags |= wasm::WASM_ELEM_SEGMENT_HAS_TABLE_NUMBER;
  encodeULEB128(Flags, W->OS);
  if (Flags & wasm::WASM_ELEM_SEGMENT_HAS_TABLE_NUMBER)
    encodeULEB128(TableNumber, W->OS); // the table number

  // init expr for starting offset
  W->OS << char(is64Bit() ? wasm::WASM_OPCODE_I64_CONST
                          : wasm::WASM_OPCODE_I32_CONST);
  encodeSLEB128(InitialTableOffset, W->OS);
  W->OS << char(wasm::WASM_OPCODE_END);

  if (Flags & wasm::WASM_ELEM_SEGMENT_MASK_HAS_ELEM_DESC) {
    // We only write active function table initializers, for which the elem kind
    // is specified to be written as 0x00 and interpreted to mean "funcref".
    const uint8_t ElemKind = 0;
    W->OS << ElemKind;
  }

```
- **EN**: Implements logic around `encodeULEB128`, `assert`, `find`, `char`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `encodeULEB128`, `assert`, `find`, `char`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 1015-1036
```cpp
  encodeULEB128(TableElems.size(), W->OS);
  for (uint32_t Elem : TableElems)
    encodeULEB128(Elem, W->OS);

  endSection(Section);
}

void WasmObjectWriter::writeDataCountSection() {
  if (DataSegments.empty())
    return;

  SectionBookkeeping Section;
  startSection(Section, wasm::WASM_SEC_DATACOUNT);
  encodeULEB128(DataSegments.size(), W->OS);
  endSection(Section);
}

uint32_t WasmObjectWriter::writeCodeSection(const MCAssembler &Asm,
                                            ArrayRef<WasmFunction> Functions) {
  if (Functions.empty())
    return 0;

```
- **EN**: Implements logic around `encodeULEB128`, `endSection`, `writeDataCountSection`, `startSection`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `encodeULEB128`, `endSection`, `writeDataCountSection`, `startSection`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1037-1057
```cpp
  SectionBookkeeping Section;
  startSection(Section, wasm::WASM_SEC_CODE);

  encodeULEB128(Functions.size(), W->OS);

  for (const WasmFunction &Func : Functions) {
    auto *FuncSection = static_cast<MCSectionWasm *>(Func.Section);

    int64_t Size = Asm.getSectionAddressSize(*FuncSection);
    encodeULEB128(Size, W->OS);
    FuncSection->setSectionOffset(W->OS.tell() - Section.ContentsOffset);
    Asm.writeSectionData(W->OS, FuncSection);
  }

  // Apply fixups.
  applyRelocations(CodeRelocations, Section.ContentsOffset, Asm);

  endSection(Section);
  return Section.Index;
}

```
- **EN**: Implements logic around `startSection`, `encodeULEB128`, `getSectionAddressSize`, `setSectionOffset`, and 3 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `startSection`, `encodeULEB128`, `getSectionAddressSize`, `setSectionOffset`, and 3 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1058-1081
```cpp
uint32_t WasmObjectWriter::writeDataSection(const MCAssembler &Asm) {
  if (DataSegments.empty())
    return 0;

  SectionBookkeeping Section;
  startSection(Section, wasm::WASM_SEC_DATA);

  encodeULEB128(DataSegments.size(), W->OS); // count

  for (const WasmDataSegment &Segment : DataSegments) {
    encodeULEB128(Segment.InitFlags, W->OS); // flags
    if (Segment.InitFlags & wasm::WASM_DATA_SEGMENT_HAS_MEMINDEX)
      encodeULEB128(0, W->OS); // memory index
    if ((Segment.InitFlags & wasm::WASM_DATA_SEGMENT_IS_PASSIVE) == 0) {
      W->OS << char(is64Bit() ? wasm::WASM_OPCODE_I64_CONST
                              : wasm::WASM_OPCODE_I32_CONST);
      encodeSLEB128(Segment.Offset, W->OS); // offset
      W->OS << char(wasm::WASM_OPCODE_END);
    }
    encodeULEB128(Segment.Data.size(), W->OS); // size
    Segment.Section->setSectionOffset(W->OS.tell() - Section.ContentsOffset);
    W->OS << Segment.Data; // data
  }

```
- **EN**: Implements logic around `writeDataSection`, `startSection`, `encodeULEB128`, `char`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `writeDataSection`, `startSection`, `encodeULEB128`, `char`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1082-1108
```cpp
  // Apply fixups.
  applyRelocations(DataRelocations, Section.ContentsOffset, Asm);

  endSection(Section);
  return Section.Index;
}

void WasmObjectWriter::writeRelocSection(
    uint32_t SectionIndex, StringRef Name,
    std::vector<WasmRelocationEntry> &Relocs) {
  // See: https://github.com/WebAssembly/tool-conventions/blob/main/Linking.md
  // for descriptions of the reloc sections.

  if (Relocs.empty())
    return;

  // First, ensure the relocations are sorted in offset order.  In general they
  // should already be sorted since `recordRelocation` is called in offset
  // order, but for the code section we combine many MC sections into single
  // wasm section, and this order is determined by the order of Asm.Symbols()
  // not the sections order.
  llvm::stable_sort(
      Relocs, [](const WasmRelocationEntry &A, const WasmRelocationEntry &B) {
        return (A.Offset + A.FixupSection->getSectionOffset()) <
               (B.Offset + B.FixupSection->getSectionOffset());
      });

```
- **EN**: Implements logic around `applyRelocations`, `endSection`, `writeRelocSection`, `stable_sort`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `applyRelocations`, `endSection`, `writeRelocSection`, `stable_sort`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1109-1128
```cpp
  SectionBookkeeping Section;
  startCustomSection(Section, std::string("reloc.") + Name.str());

  encodeULEB128(SectionIndex, W->OS);
  encodeULEB128(Relocs.size(), W->OS);
  for (const WasmRelocationEntry &RelEntry : Relocs) {
    uint64_t Offset =
        RelEntry.Offset + RelEntry.FixupSection->getSectionOffset();
    uint32_t Index = getRelocationIndexValue(RelEntry);

    W->OS << char(RelEntry.Type);
    encodeULEB128(Offset, W->OS);
    encodeULEB128(Index, W->OS);
    if (RelEntry.hasAddend())
      encodeSLEB128(RelEntry.Addend, W->OS);
  }

  endSection(Section);
}

```
- **EN**: Implements logic around `startCustomSection`, `encodeULEB128`, `getSectionOffset`, `getRelocationIndexValue`, and 3 more symbols; this block handles relocation, fixup, or symbol-resolution work; updates MC section or symbol state.
- **CN**: 围绕 `startCustomSection`, `encodeULEB128`, `getSectionOffset`, `getRelocationIndexValue`, and 3 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，更新 MC 节区或符号状态。

### Lines 1129-1168
```cpp
void WasmObjectWriter::writeCustomRelocSections() {
  for (const auto &Sec : CustomSections) {
    auto &Relocations = CustomSectionsRelocations[Sec.Section];
    writeRelocSection(Sec.OutputIndex, Sec.Name, Relocations);
  }
}

void WasmObjectWriter::writeLinkingMetaDataSection(
    ArrayRef<wasm::WasmSymbolInfo> SymbolInfos,
    ArrayRef<std::pair<uint16_t, uint32_t>> InitFuncs,
    const std::map<StringRef, std::vector<WasmComdatEntry>> &Comdats) {
  SectionBookkeeping Section;
  startCustomSection(Section, "linking");
  encodeULEB128(wasm::WasmMetadataVersion, W->OS);

  SectionBookkeeping SubSection;
  if (SymbolInfos.size() != 0) {
    startSection(SubSection, wasm::WASM_SYMBOL_TABLE);
    encodeULEB128(SymbolInfos.size(), W->OS);
    for (const wasm::WasmSymbolInfo &Sym : SymbolInfos) {
      encodeULEB128(Sym.Kind, W->OS);
      encodeULEB128(Sym.Flags, W->OS);
      switch (Sym.Kind) {
      case wasm::WASM_SYMBOL_TYPE_FUNCTION:
      case wasm::WASM_SYMBOL_TYPE_GLOBAL:
      case wasm::WASM_SYMBOL_TYPE_TAG:
      case wasm::WASM_SYMBOL_TYPE_TABLE:
        encodeULEB128(Sym.ElementIndex, W->OS);
        if ((Sym.Flags & wasm::WASM_SYMBOL_UNDEFINED) == 0 ||
            (Sym.Flags & wasm::WASM_SYMBOL_EXPLICIT_NAME) != 0)
          writeString(Sym.Name);
        break;
      case wasm::WASM_SYMBOL_TYPE_DATA:
        writeString(Sym.Name);
        if ((Sym.Flags & wasm::WASM_SYMBOL_UNDEFINED) == 0) {
          encodeULEB128(Sym.DataRef.Segment, W->OS);
          encodeULEB128(Sym.DataRef.Offset, W->OS);
          encodeULEB128(Sym.DataRef.Size, W->OS);
        }
        break;
```
- **EN**: Implements logic around `writeCustomRelocSections`, `writeRelocSection`, `writeLinkingMetaDataSection`, `startCustomSection`, and 3 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `writeCustomRelocSections`, `writeRelocSection`, `writeLinkingMetaDataSection`, `startCustomSection`, and 3 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 1169-1192
```cpp
      case wasm::WASM_SYMBOL_TYPE_SECTION: {
        const uint32_t SectionIndex =
            CustomSections[Sym.ElementIndex].OutputIndex;
        encodeULEB128(SectionIndex, W->OS);
        break;
      }
      default:
        llvm_unreachable("unexpected kind");
      }
    }
    endSection(SubSection);
  }

  if (DataSegments.size()) {
    startSection(SubSection, wasm::WASM_SEGMENT_INFO);
    encodeULEB128(DataSegments.size(), W->OS);
    for (const WasmDataSegment &Segment : DataSegments) {
      writeString(Segment.Name);
      encodeULEB128(Segment.Alignment, W->OS);
      encodeULEB128(Segment.LinkingFlags, W->OS);
    }
    endSection(SubSection);
  }

```
- **EN**: Implements logic around `encodeULEB128`, `llvm_unreachable`, `endSection`, `startSection`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `encodeULEB128`, `llvm_unreachable`, `endSection`, `startSection`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1193-1217
```cpp
  if (!InitFuncs.empty()) {
    startSection(SubSection, wasm::WASM_INIT_FUNCS);
    encodeULEB128(InitFuncs.size(), W->OS);
    for (auto &StartFunc : InitFuncs) {
      encodeULEB128(StartFunc.first, W->OS);  // priority
      encodeULEB128(StartFunc.second, W->OS); // function index
    }
    endSection(SubSection);
  }

  if (Comdats.size()) {
    startSection(SubSection, wasm::WASM_COMDAT_INFO);
    encodeULEB128(Comdats.size(), W->OS);
    for (const auto &C : Comdats) {
      writeString(C.first);
      encodeULEB128(0, W->OS); // flags for future use
      encodeULEB128(C.second.size(), W->OS);
      for (const WasmComdatEntry &Entry : C.second) {
        encodeULEB128(Entry.Kind, W->OS);
        encodeULEB128(Entry.Index, W->OS);
      }
    }
    endSection(SubSection);
  }

```
- **EN**: Implements logic around `startSection`, `encodeULEB128`, `endSection`, `writeString`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `startSection`, `encodeULEB128`, `endSection`, `writeString` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1218-1239
```cpp
  endSection(Section);
}

void WasmObjectWriter::writeCustomSection(WasmCustomSection &CustomSection,
                                          const MCAssembler &Asm) {
  SectionBookkeeping Section;
  auto *Sec = CustomSection.Section;
  startCustomSection(Section, CustomSection.Name);

  Sec->setSectionOffset(W->OS.tell() - Section.ContentsOffset);
  Asm.writeSectionData(W->OS, Sec);

  CustomSection.OutputContentsOffset = Section.ContentsOffset;
  CustomSection.OutputIndex = Section.Index;

  endSection(Section);

  // Apply fixups.
  auto &Relocations = CustomSectionsRelocations[CustomSection.Section];
  applyRelocations(Relocations, CustomSection.OutputContentsOffset, Asm);
}

```
- **EN**: Implements logic around `endSection`, `writeCustomSection`, `startCustomSection`, `setSectionOffset`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `endSection`, `writeCustomSection`, `startCustomSection`, `setSectionOffset`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1240-1261
```cpp
uint32_t WasmObjectWriter::getFunctionType(const MCSymbolWasm &Symbol) {
  assert(Symbol.isFunction());
  assert(TypeIndices.contains(&Symbol));
  return TypeIndices[&Symbol];
}

uint32_t WasmObjectWriter::getTagType(const MCSymbolWasm &Symbol) {
  assert(Symbol.isTag());
  assert(TypeIndices.contains(&Symbol));
  return TypeIndices[&Symbol];
}

void WasmObjectWriter::registerFunctionType(const MCSymbolWasm &Symbol) {
  assert(Symbol.isFunction());

  wasm::WasmSignature S;

  if (auto *Sig = Symbol.getSignature()) {
    S.Returns = Sig->Returns;
    S.Params = Sig->Params;
  }

```
- **EN**: Implements logic around `getFunctionType`, `assert`, `getTagType`, `registerFunctionType`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getFunctionType`, `assert`, `getTagType`, `registerFunctionType` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1262-1282
```cpp
  auto Pair = SignatureIndices.insert(std::make_pair(S, Signatures.size()));
  if (Pair.second)
    Signatures.push_back(S);
  TypeIndices[&Symbol] = Pair.first->second;

  LLVM_DEBUG(dbgs() << "registerFunctionType: " << Symbol
                    << " new:" << Pair.second << "\n");
  LLVM_DEBUG(dbgs() << "  -> type index: " << Pair.first->second << "\n");
}

void WasmObjectWriter::registerTagType(const MCSymbolWasm &Symbol) {
  assert(Symbol.isTag());

  // TODO Currently we don't generate imported exceptions, but if we do, we
  // should have a way of infering types of imported exceptions.
  wasm::WasmSignature S;
  if (auto *Sig = Symbol.getSignature()) {
    S.Returns = Sig->Returns;
    S.Params = Sig->Params;
  }

```
- **EN**: Implements logic around `insert`, `push_back`, `registerTagType`, `assert`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `insert`, `push_back`, `registerTagType`, `assert` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1283-1302
```cpp
  auto Pair = SignatureIndices.insert(std::make_pair(S, Signatures.size()));
  if (Pair.second)
    Signatures.push_back(S);
  TypeIndices[&Symbol] = Pair.first->second;

  LLVM_DEBUG(dbgs() << "registerTagType: " << Symbol << " new:" << Pair.second
                    << "\n");
  LLVM_DEBUG(dbgs() << "  -> type index: " << Pair.first->second << "\n");
}

static bool isInSymtab(const MCSymbolWasm &Sym) {
  if (Sym.isUsedInReloc() || Sym.isUsedInInitArray())
    return true;

  if (Sym.isComdat() && !Sym.isDefined())
    return false;

  if (Sym.isTemporary())
    return false;

```
- **EN**: Implements logic around `insert`, `push_back`, `isInSymtab`; this block handles relocation, fixup, or symbol-resolution work; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `insert`, `push_back`, `isInSymtab` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1303-1324
```cpp
  if (Sym.isSection())
    return false;

  if (Sym.omitFromLinkingSection())
    return false;

  return true;
}

static bool isSectionReferenced(MCAssembler &Asm, MCSectionWasm &Section) {
  StringRef SectionName = Section.getName();

  for (const MCSymbol &S : Asm.symbols()) {
    const auto &WS = static_cast<const MCSymbolWasm &>(S);
    if (WS.isData() && WS.isInSection()) {
      auto &RefSection = static_cast<MCSectionWasm &>(WS.getSection());
      if (RefSection.getName() == SectionName) {
        return true;
      }
    }
  }

```
- **EN**: Implements logic around `isSectionReferenced`, `getName`, `getSection`; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `isSectionReferenced`, `getName`, `getSection` 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1325-1346
```cpp
  return false;
}

void WasmObjectWriter::prepareImports(
    SmallVectorImpl<wasm::WasmImport> &Imports, MCAssembler &Asm) {
  // For now, always emit the memory import, since loads and stores are not
  // valid without it. In the future, we could perhaps be more clever and omit
  // it if there are no loads or stores.
  wasm::WasmImport MemImport;
  MemImport.Module = "env";
  MemImport.Field = "__linear_memory";
  MemImport.Kind = wasm::WASM_EXTERNAL_MEMORY;
  MemImport.Memory.Flags = is64Bit() ? wasm::WASM_LIMITS_FLAG_IS_64
                                     : wasm::WASM_LIMITS_FLAG_NONE;
  Imports.push_back(MemImport);

  // Populate SignatureIndices, and Imports and WasmIndices for undefined
  // symbols.  This must be done before populating WasmIndices for defined
  // symbols.
  for (const MCSymbol &S : Asm.symbols()) {
    const auto &WS = static_cast<const MCSymbolWasm &>(S);

```
- **EN**: Implements logic around `prepareImports`, `is64Bit`, `push_back`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `prepareImports`, `is64Bit`, `push_back` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1347-1377
```cpp
    // Register types for all functions, including those with private linkage
    // (because wasm always needs a type signature).
    if (WS.isFunction()) {
      auto *BS = static_cast<const MCSymbolWasm *>(Asm.getBaseSymbol(S));
      if (!BS)
        report_fatal_error(Twine(S.getName()) +
                           ": absolute addressing not supported!");
      registerFunctionType(*BS);
    }

    if (WS.isTag())
      registerTagType(WS);

    if (WS.isTemporary())
      continue;

    // If the symbol is not defined in this translation unit, import it.
    if (!WS.isDefined() && !WS.isComdat()) {
      if (WS.isFunction()) {
        wasm::WasmImport Import;
        Import.Module = WS.getImportModule();
        Import.Field = WS.getImportName();
        Import.Kind = wasm::WASM_EXTERNAL_FUNCTION;
        Import.SigIndex = getFunctionType(WS);
        Imports.push_back(Import);
        assert(!WasmIndices.contains(&WS));
        WasmIndices[&WS] = NumFunctionImports++;
      } else if (WS.isGlobal()) {
        if (WS.isWeak())
          report_fatal_error("undefined global symbol cannot be weak");

```
- **EN**: Implements logic around `getBaseSymbol`, `report_fatal_error`, `registerFunctionType`, `registerTagType`, and 5 more symbols; this block updates MC section or symbol state.
- **CN**: 围绕 `getBaseSymbol`, `report_fatal_error`, `registerFunctionType`, `registerTagType`, and 5 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 1378-1401
```cpp
        wasm::WasmImport Import;
        Import.Field = WS.getImportName();
        Import.Kind = wasm::WASM_EXTERNAL_GLOBAL;
        Import.Module = WS.getImportModule();
        Import.Global = WS.getGlobalType();
        Imports.push_back(Import);
        assert(!WasmIndices.contains(&WS));
        WasmIndices[&WS] = NumGlobalImports++;
      } else if (WS.isTag()) {
        if (WS.isWeak())
          report_fatal_error("undefined tag symbol cannot be weak");

        wasm::WasmImport Import;
        Import.Module = WS.getImportModule();
        Import.Field = WS.getImportName();
        Import.Kind = wasm::WASM_EXTERNAL_TAG;
        Import.SigIndex = getTagType(WS);
        Imports.push_back(Import);
        assert(!WasmIndices.contains(&WS));
        WasmIndices[&WS] = NumTagImports++;
      } else if (WS.isTable()) {
        if (WS.isWeak())
          report_fatal_error("undefined table symbol cannot be weak");

```
- **EN**: Implements logic around `getImportName`, `getImportModule`, `getGlobalType`, `push_back`, and 3 more symbols; this block updates MC section or symbol state.
- **CN**: 围绕 `getImportName`, `getImportModule`, `getGlobalType`, `push_back`, and 3 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 1402-1432
```cpp
        wasm::WasmImport Import;
        Import.Module = WS.getImportModule();
        Import.Field = WS.getImportName();
        Import.Kind = wasm::WASM_EXTERNAL_TABLE;
        Import.Table = WS.getTableType();
        Imports.push_back(Import);
        assert(!WasmIndices.contains(&WS));
        WasmIndices[&WS] = NumTableImports++;
      }
    }
  }

  // Add imports for GOT globals
  for (const MCSymbol &S : Asm.symbols()) {
    const auto &WS = static_cast<const MCSymbolWasm &>(S);
    if (WS.isUsedInGOT()) {
      wasm::WasmImport Import;
      if (WS.isFunction())
        Import.Module = "GOT.func";
      else
        Import.Module = "GOT.mem";
      Import.Field = WS.getName();
      Import.Kind = wasm::WASM_EXTERNAL_GLOBAL;
      Import.Global = {wasm::WASM_TYPE_I32, true};
      Imports.push_back(Import);
      assert(!GOTIndices.contains(&WS));
      GOTIndices[&WS] = NumGlobalImports++;
    }
  }
}

```
- **EN**: Implements logic around `getImportModule`, `getImportName`, `getTableType`, `push_back`, and 2 more symbols; this block updates MC section or symbol state.
- **CN**: 围绕 `getImportModule`, `getImportName`, `getTableType`, `push_back`, and 2 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 1433-1452
```cpp
uint64_t WasmObjectWriter::writeObject() {
  support::endian::Writer MainWriter(*OS, llvm::endianness::little);
  W = &MainWriter;
  if (IsSplitDwarf) {
    uint64_t TotalSize = writeOneObject(*Asm, DwoMode::NonDwoOnly);
    assert(DwoOS);
    support::endian::Writer DwoWriter(*DwoOS, llvm::endianness::little);
    W = &DwoWriter;
    return TotalSize + writeOneObject(*Asm, DwoMode::DwoOnly);
  } else {
    return writeOneObject(*Asm, DwoMode::AllSections);
  }
}

uint64_t WasmObjectWriter::writeOneObject(MCAssembler &Asm,
                                          DwoMode Mode) {
  uint64_t StartOffset = W->OS.tell();
  SectionCount = 0;
  CustomSections.clear();

```
- **EN**: Implements logic around `writeObject`, `MainWriter`, `writeOneObject`, `assert`, and 3 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `writeObject`, `MainWriter`, `writeOneObject`, `assert`, and 3 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1453-1475
```cpp
  LLVM_DEBUG(dbgs() << "WasmObjectWriter::writeObject\n");

  // Collect information from the available symbols.
  SmallVector<WasmFunction, 4> Functions;
  SmallVector<uint32_t, 4> TableElems;
  SmallVector<wasm::WasmImport, 4> Imports;
  SmallVector<wasm::WasmExport, 4> Exports;
  SmallVector<uint32_t, 2> TagTypes;
  SmallVector<wasm::WasmGlobal, 1> Globals;
  SmallVector<wasm::WasmTable, 1> Tables;
  SmallVector<wasm::WasmSymbolInfo, 4> SymbolInfos;
  SmallVector<std::pair<uint16_t, uint32_t>, 2> InitFuncs;
  std::map<StringRef, std::vector<WasmComdatEntry>> Comdats;
  uint64_t DataSize = 0;
  if (Mode != DwoMode::DwoOnly)
    prepareImports(Imports, Asm);

  // Populate DataSegments and CustomSections, which must be done before
  // populating DataLocations.
  for (MCSection &Sec : Asm) {
    auto &Section = static_cast<MCSectionWasm &>(Sec);
    StringRef SectionName = Section.getName();

```
- **EN**: Implements logic around `prepareImports`, `getName`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `prepareImports`, `getName` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1476-1510
```cpp
    if (Mode == DwoMode::NonDwoOnly && isDwoSection(Sec))
      continue;
    if (Mode == DwoMode::DwoOnly && !isDwoSection(Sec))
      continue;

    LLVM_DEBUG(dbgs() << "Processing Section " << SectionName << "  group "
                      << Section.getGroup() << "\n";);

    // .init_array sections are handled specially elsewhere, include them in
    // data segments if and only if referenced by a symbol.
    if (SectionName.starts_with(".init_array") &&
        !isSectionReferenced(Asm, Section))
      continue;

    // Code is handled separately
    if (Section.isText())
      continue;

    if (Section.isWasmData()) {
      uint32_t SegmentIndex = DataSegments.size();
      DataSize = alignTo(DataSize, Section.getAlign());
      DataSegments.emplace_back();
      WasmDataSegment &Segment = DataSegments.back();
      Segment.Name = SectionName;
      Segment.InitFlags = Section.getPassive()
                              ? (uint32_t)wasm::WASM_DATA_SEGMENT_IS_PASSIVE
                              : 0;
      Segment.Offset = DataSize;
      Segment.Section = &Section;
      addData(Segment.Data, Section);
      Segment.Alignment = Log2(Section.getAlign());
      Segment.LinkingFlags = Section.getSegmentFlags();
      DataSize += Segment.Data.size();
      Section.setSegmentIndex(SegmentIndex);

```
- **EN**: Implements logic around `getGroup`, `isSectionReferenced`, `size`, `alignTo`, and 7 more symbols; this block updates MC section or symbol state.
- **CN**: 围绕 `getGroup`, `isSectionReferenced`, `size`, `alignTo`, and 7 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 1511-1540
```cpp
      if (const MCSymbolWasm *C = Section.getGroup()) {
        Comdats[C->getName()].emplace_back(
            WasmComdatEntry{wasm::WASM_COMDAT_DATA, SegmentIndex});
      }
    } else {
      // Create custom sections
      assert(Section.isMetadata());

      StringRef Name = SectionName;

      // For user-defined custom sections, strip the prefix
      Name.consume_front(".custom_section.");

      auto *Begin = static_cast<MCSymbolWasm *>(Sec.getBeginSymbol());
      if (Begin) {
        assert(!WasmIndices.contains(Begin));
        WasmIndices[Begin] = CustomSections.size();
      }

      // Separate out the producers and target features sections
      if (Name == "producers") {
        ProducersSection = std::make_unique<WasmCustomSection>(Name, &Section);
        continue;
      }
      if (Name == "target_features") {
        TargetFeaturesSection =
            std::make_unique<WasmCustomSection>(Name, &Section);
        continue;
      }

```
- **EN**: Implements logic around `getName`, `assert`, `consume_front`, `getBeginSymbol`, and 2 more symbols; this block updates MC section or symbol state.
- **CN**: 围绕 `getName`, `assert`, `consume_front`, `getBeginSymbol`, and 2 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 1541-1562
```cpp
      // Custom sections can also belong to COMDAT groups. In this case the
      // decriptor's "index" field is the section index (in the final object
      // file), but that is not known until after layout, so it must be fixed up
      // later
      if (const MCSymbolWasm *C = Section.getGroup()) {
        Comdats[C->getName()].emplace_back(
            WasmComdatEntry{wasm::WASM_COMDAT_SECTION,
                            static_cast<uint32_t>(CustomSections.size())});
      }

      CustomSections.emplace_back(Name, &Section);
    }
  }

  if (Mode != DwoMode::DwoOnly) {
    // Populate WasmIndices and DataLocations for defined symbols.
    for (const MCSymbol &S : Asm.symbols()) {
      // Ignore unnamed temporary symbols, which aren't ever exported, imported,
      // or used in relocations.
      if (S.isTemporary() && S.getName().empty())
        continue;

```
- **EN**: Implements logic around `getName`, `static_cast<uint32_t>`, `emplace_back`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `getName`, `static_cast<uint32_t>`, `emplace_back` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1563-1584
```cpp
      const auto &WS = static_cast<const MCSymbolWasm &>(S);
      LLVM_DEBUG(
          dbgs() << "MCSymbol: "
                 << toString(WS.getType().value_or(wasm::WASM_SYMBOL_TYPE_DATA))
                 << " '" << S << "'"
                 << " isDefined=" << S.isDefined() << " isExternal="
                 << WS.isExternal() << " isTemporary=" << S.isTemporary()
                 << " isWeak=" << WS.isWeak() << " isHidden=" << WS.isHidden()
                 << " isVariable=" << WS.isVariable() << "\n");

      if (WS.isVariable())
        continue;
      if (WS.isComdat() && !WS.isDefined())
        continue;

      if (WS.isFunction()) {
        unsigned Index;
        if (WS.isDefined()) {
          if (WS.getOffset() != 0)
            report_fatal_error(
                "function sections must contain one function each");

```
- **EN**: Implements logic around `dbgs`, `toString`, `isDefined`, `isExternal`, and 3 more symbols; this block updates MC section or symbol state.
- **CN**: 围绕 `dbgs`, `toString`, `isDefined`, `isExternal`, and 3 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 1585-1611
```cpp
          // A definition. Write out the function body.
          Index = NumFunctionImports + Functions.size();
          WasmFunction Func;
          Func.SigIndex = getFunctionType(WS);
          Func.Section = &WS.getSection();
          assert(!WasmIndices.contains(&WS));
          WasmIndices[&WS] = Index;
          Functions.push_back(Func);

          auto &Section = static_cast<MCSectionWasm &>(WS.getSection());
          if (const MCSymbolWasm *C = Section.getGroup()) {
            Comdats[C->getName()].emplace_back(
                WasmComdatEntry{wasm::WASM_COMDAT_FUNCTION, Index});
          }

          if (WS.hasExportName()) {
            wasm::WasmExport Export;
            Export.Name = WS.getExportName();
            Export.Kind = wasm::WASM_EXTERNAL_FUNCTION;
            Export.Index = Index;
            Exports.push_back(Export);
          }
        } else {
          // An import; the index was assigned above.
          Index = WasmIndices.find(&WS)->second;
        }

```
- **EN**: Implements logic around `size`, `getFunctionType`, `getSection`, `assert`, and 4 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `size`, `getFunctionType`, `getSection`, `assert`, and 4 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1612-1631
```cpp
        LLVM_DEBUG(dbgs() << "  -> function index: " << Index << "\n");

      } else if (WS.isData()) {
        if (!isInSymtab(WS))
          continue;

        if (!WS.isDefined()) {
          LLVM_DEBUG(dbgs() << "  -> segment index: -1"
                            << "\n");
          continue;
        }

        if (!WS.getSize())
          report_fatal_error("data symbols must have a size set with .size: " +
                             WS.getName());

        int64_t Size = 0;
        if (!WS.getSize()->evaluateAsAbsolute(Size, Asm))
          report_fatal_error(".size expression must be evaluatable");

```
- **EN**: Implements logic around `report_fatal_error`, `getName`; this block updates MC section or symbol state.
- **CN**: 围绕 `report_fatal_error`, `getName` 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 1632-1671
```cpp
        auto &DataSection = static_cast<MCSectionWasm &>(WS.getSection());
        if (!DataSection.isWasmData())
          report_fatal_error("data symbols must live in a data section: " +
                             WS.getName());

        // For each data symbol, export it in the symtab as a reference to the
        // corresponding Wasm data segment.
        wasm::WasmDataReference Ref = wasm::WasmDataReference{
            DataSection.getSegmentIndex(), Asm.getSymbolOffset(WS),
            static_cast<uint64_t>(Size)};
        assert(!DataLocations.contains(&WS));
        DataLocations[&WS] = Ref;
        LLVM_DEBUG(dbgs() << "  -> segment index: " << Ref.Segment << "\n");

      } else if (WS.isGlobal()) {
        // A "true" Wasm global (currently just __stack_pointer)
        if (WS.isDefined()) {
          wasm::WasmGlobal Global;
          Global.Type = WS.getGlobalType();
          Global.Index = NumGlobalImports + Globals.size();
          Global.InitExpr.Extended = false;
          switch (Global.Type.Type) {
          case wasm::WASM_TYPE_I32:
            Global.InitExpr.Inst.Opcode = wasm::WASM_OPCODE_I32_CONST;
            break;
          case wasm::WASM_TYPE_I64:
            Global.InitExpr.Inst.Opcode = wasm::WASM_OPCODE_I64_CONST;
            break;
          case wasm::WASM_TYPE_F32:
            Global.InitExpr.Inst.Opcode = wasm::WASM_OPCODE_F32_CONST;
            break;
          case wasm::WASM_TYPE_F64:
            Global.InitExpr.Inst.Opcode = wasm::WASM_OPCODE_F64_CONST;
            break;
          case wasm::WASM_TYPE_EXTERNREF:
            Global.InitExpr.Inst.Opcode = wasm::WASM_OPCODE_REF_NULL;
            break;
          default:
            llvm_unreachable("unexpected type");
          }
```
- **EN**: Implements logic around `getSection`, `report_fatal_error`, `getName`, `getSegmentIndex`, and 5 more symbols; this block updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `getSection`, `report_fatal_error`, `getName`, `getSegmentIndex`, and 5 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 1672-1707
```cpp
          assert(!WasmIndices.contains(&WS));
          WasmIndices[&WS] = Global.Index;
          Globals.push_back(Global);
        } else {
          // An import; the index was assigned above
          LLVM_DEBUG(dbgs() << "  -> global index: "
                            << WasmIndices.find(&WS)->second << "\n");
        }
      } else if (WS.isTable()) {
        if (WS.isDefined()) {
          wasm::WasmTable Table;
          Table.Index = NumTableImports + Tables.size();
          Table.Type = WS.getTableType();
          assert(!WasmIndices.contains(&WS));
          WasmIndices[&WS] = Table.Index;
          Tables.push_back(Table);
        }
        LLVM_DEBUG(dbgs() << " -> table index: "
                          << WasmIndices.find(&WS)->second << "\n");
      } else if (WS.isTag()) {
        // C++ exception symbol (__cpp_exception) or longjmp symbol
        // (__c_longjmp)
        unsigned Index;
        if (WS.isDefined()) {
          Index = NumTagImports + TagTypes.size();
          uint32_t SigIndex = getTagType(WS);
          assert(!WasmIndices.contains(&WS));
          WasmIndices[&WS] = Index;
          TagTypes.push_back(SigIndex);
        } else {
          // An import; the index was assigned above.
          assert(WasmIndices.contains(&WS));
        }
        LLVM_DEBUG(dbgs() << "  -> tag index: " << WasmIndices.find(&WS)->second
                          << "\n");

```
- **EN**: Implements logic around `assert`, `push_back`, `find`, `size`, and 2 more symbols; this block updates MC section or symbol state.
- **CN**: 围绕 `assert`, `push_back`, `find`, `size`, and 2 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 1708-1728
```cpp
      } else {
        assert(WS.isSection());
      }
    }

    // Populate WasmIndices and DataLocations for aliased symbols.  We need to
    // process these in a separate pass because we need to have processed the
    // target of the alias before the alias itself and the symbols are not
    // necessarily ordered in this way.
    for (const MCSymbol &S : Asm.symbols()) {
      if (!S.isVariable())
        continue;

      assert(S.isDefined());

      const auto *BS = Asm.getBaseSymbol(S);
      if (!BS)
        report_fatal_error(Twine(S.getName()) +
                           ": absolute addressing not supported!");
      const MCSymbolWasm *Base = static_cast<const MCSymbolWasm *>(BS);

```
- **EN**: Implements logic around `assert`, `getBaseSymbol`, `report_fatal_error`; this block updates MC section or symbol state.
- **CN**: 围绕 `assert`, `getBaseSymbol`, `report_fatal_error` 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 1729-1765
```cpp
      // Find the target symbol of this weak alias and export that index
      const auto &WS = static_cast<const MCSymbolWasm &>(S);
      LLVM_DEBUG(dbgs() << WS.getName() << ": weak alias of '" << *Base
                        << "'\n");

      if (Base->isFunction()) {
        assert(WasmIndices.contains(Base));
        uint32_t WasmIndex = WasmIndices.find(Base)->second;
        assert(!WasmIndices.contains(&WS));
        WasmIndices[&WS] = WasmIndex;
        LLVM_DEBUG(dbgs() << "  -> index:" << WasmIndex << "\n");
      } else if (Base->isData()) {
        auto &DataSection = static_cast<MCSectionWasm &>(WS.getSection());
        uint64_t Offset = Asm.getSymbolOffset(S);
        int64_t Size = 0;
        // For data symbol alias we use the size of the base symbol as the
        // size of the alias.  When an offset from the base is involved this
        // can result in a offset + size goes past the end of the data section
        // which out object format doesn't support.  So we must clamp it.
        if (!Base->getSize()->evaluateAsAbsolute(Size, Asm))
          report_fatal_error(".size expression must be evaluatable");
        const WasmDataSegment &Segment =
            DataSegments[DataSection.getSegmentIndex()];
        Size =
            std::min(static_cast<uint64_t>(Size), Segment.Data.size() - Offset);
        wasm::WasmDataReference Ref = wasm::WasmDataReference{
            DataSection.getSegmentIndex(),
            static_cast<uint32_t>(Asm.getSymbolOffset(S)),
            static_cast<uint32_t>(Size)};
        DataLocations[&WS] = Ref;
        LLVM_DEBUG(dbgs() << "  -> index:" << Ref.Segment << "\n");
      } else {
        report_fatal_error("don't yet support global/tag aliases");
      }
    }
  }

```
- **EN**: Implements logic around `assert`, `find`, `getSection`, `getSymbolOffset`, and 4 more symbols; this block updates MC section or symbol state.
- **CN**: 围绕 `assert`, `find`, `getSection`, `getSymbolOffset`, and 4 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 1766-1786
```cpp
  // Finally, populate the symbol table itself, in its "natural" order.
  for (const MCSymbol &S : Asm.symbols()) {
    const auto &WS = static_cast<const MCSymbolWasm &>(S);
    if (!isInSymtab(WS)) {
      WS.setIndex(InvalidIndex);
      continue;
    }
    // In bitcode generated by split-LTO-unit mode in ThinLTO, these lines can
    // appear:
    // module asm ".lto_set_conditional symbolA,symbolA.[moduleId]"
    // ...
    // (Here [moduleId] will be replaced by a real module hash ID)
    //
    // Here the original symbol (symbolA here) has been renamed to the new name
    // created by attaching its module ID, so the original symbol does not
    // appear in the bitcode anymore, and thus not in DataLocations. We should
    // ignore them.
    if (WS.isData() && WS.isDefined() && !DataLocations.contains(&WS))
      continue;
    LLVM_DEBUG(dbgs() << "adding to symtab: " << WS << "\n");

```
- **EN**: Implements logic around `setIndex`; this block updates MC section or symbol state.
- **CN**: 围绕 `setIndex` 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 1787-1808
```cpp
    uint32_t Flags = 0;
    if (WS.isWeak())
      Flags |= wasm::WASM_SYMBOL_BINDING_WEAK;
    if (WS.isHidden())
      Flags |= wasm::WASM_SYMBOL_VISIBILITY_HIDDEN;
    if (!WS.isExternal() && WS.isDefined())
      Flags |= wasm::WASM_SYMBOL_BINDING_LOCAL;
    if (WS.isUndefined())
      Flags |= wasm::WASM_SYMBOL_UNDEFINED;
    if (WS.isNoStrip()) {
      Flags |= wasm::WASM_SYMBOL_NO_STRIP;
      if (isEmscripten()) {
        Flags |= wasm::WASM_SYMBOL_EXPORTED;
      }
    }
    if (WS.hasImportName())
      Flags |= wasm::WASM_SYMBOL_EXPLICIT_NAME;
    if (WS.hasExportName())
      Flags |= wasm::WASM_SYMBOL_EXPORTED;
    if (WS.isTLS())
      Flags |= wasm::WASM_SYMBOL_TLS;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 1809-1848
```cpp
    wasm::WasmSymbolInfo Info;
    Info.Name = WS.getName();
    Info.Kind = WS.getType().value_or(wasm::WASM_SYMBOL_TYPE_DATA);
    Info.Flags = Flags;
    if (!WS.isData()) {
      assert(WasmIndices.contains(&WS));
      Info.ElementIndex = WasmIndices.find(&WS)->second;
    } else if (WS.isDefined()) {
      assert(DataLocations.contains(&WS));
      Info.DataRef = DataLocations.find(&WS)->second;
    }
    WS.setIndex(SymbolInfos.size());
    SymbolInfos.emplace_back(Info);
  }

  {
    auto HandleReloc = [&](const WasmRelocationEntry &Rel) {
      // Functions referenced by a relocation need to put in the table.  This is
      // purely to make the object file's provisional values readable, and is
      // ignored by the linker, which re-calculates the relocations itself.
      if (Rel.Type != wasm::R_WASM_TABLE_INDEX_I32 &&
          Rel.Type != wasm::R_WASM_TABLE_INDEX_I64 &&
          Rel.Type != wasm::R_WASM_TABLE_INDEX_SLEB &&
          Rel.Type != wasm::R_WASM_TABLE_INDEX_SLEB64 &&
          Rel.Type != wasm::R_WASM_TABLE_INDEX_REL_SLEB &&
          Rel.Type != wasm::R_WASM_TABLE_INDEX_REL_SLEB64)
        return;
      assert(Rel.Symbol->isFunction());
      auto *Base =
          static_cast<const MCSymbolWasm *>(Asm.getBaseSymbol(*Rel.Symbol));
      uint32_t FunctionIndex = WasmIndices.find(Base)->second;
      uint32_t TableIndex = TableElems.size() + InitialTableOffset;
      if (TableIndices.try_emplace(Base, TableIndex).second) {
        LLVM_DEBUG(dbgs() << "  -> adding " << Base->getName()
                          << " to table: " << TableIndex << "\n");
        TableElems.push_back(FunctionIndex);
        registerFunctionType(*Base);
      }
    };

```
- **EN**: Implements logic around `getName`, `getType`, `assert`, `find`, and 6 more symbols; this block handles relocation, fixup, or symbol-resolution work; updates MC section or symbol state.
- **CN**: 围绕 `getName`, `getType`, `assert`, `find`, and 6 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，更新 MC 节区或符号状态。

### Lines 1849-1869
```cpp
    for (const WasmRelocationEntry &RelEntry : CodeRelocations)
      HandleReloc(RelEntry);
    for (const WasmRelocationEntry &RelEntry : DataRelocations)
      HandleReloc(RelEntry);
  }

  // Translate .init_array section contents into start functions.
  for (const MCSection &S : Asm) {
    const auto &WS = static_cast<const MCSectionWasm &>(S);
    if (WS.getName().starts_with(".fini_array"))
      report_fatal_error(".fini_array sections are unsupported");
    if (!WS.getName().starts_with(".init_array"))
      continue;
    auto IT = WS.begin();
    if (IT == WS.end())
      continue;
    for (auto *Frag = &*IT; Frag; Frag = Frag->getNext()) {
      if (Frag->hasInstructions() || (Frag->getKind() != MCFragment::FT_Align &&
                                      Frag->getKind() != MCFragment::FT_Data))
        report_fatal_error("only data supported in .init_array section");

```
- **EN**: Implements logic around `HandleReloc`, `report_fatal_error`, `begin`, `getKind`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `HandleReloc`, `report_fatal_error`, `begin`, `getKind` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1870-1898
```cpp
      uint16_t Priority = UINT16_MAX;
      unsigned PrefixLength = strlen(".init_array");
      if (WS.getName().size() > PrefixLength) {
        if (WS.getName()[PrefixLength] != '.')
          report_fatal_error(
              ".init_array section priority should start with '.'");
        if (WS.getName().substr(PrefixLength + 1).getAsInteger(10, Priority))
          report_fatal_error("invalid .init_array section priority");
      }
      assert(llvm::all_of(Frag->getContents(), [](char C) { return !C; }));
      for (const MCFixup &Fixup : Frag->getFixups()) {
        assert(Fixup.getKind() ==
               MCFixup::getDataKindForSize(is64Bit() ? 8 : 4));
        const MCExpr *Expr = Fixup.getValue();
        auto *SymRef = dyn_cast<MCSymbolRefExpr>(Expr);
        if (!SymRef)
          report_fatal_error(
              "fixups in .init_array should be symbol references");
        auto &TargetSym =
            static_cast<const MCSymbolWasm &>(SymRef->getSymbol());
        if (TargetSym.getIndex() == InvalidIndex)
          report_fatal_error("symbols in .init_array should exist in symtab");
        if (!TargetSym.isFunction())
          report_fatal_error("symbols in .init_array should be for functions");
        InitFuncs.push_back(std::make_pair(Priority, TargetSym.getIndex()));
      }
    }
  }

```
- **EN**: Implements logic around `strlen`, `report_fatal_error`, `assert`, `getDataKindForSize`, and 4 more symbols; this block handles relocation, fixup, or symbol-resolution work; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `strlen`, `report_fatal_error`, `assert`, `getDataKindForSize`, and 4 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1899-1921
```cpp
  // Write out the Wasm header.
  writeHeader(Asm);

  uint32_t CodeSectionIndex, DataSectionIndex;
  if (Mode != DwoMode::DwoOnly) {
    writeTypeSection(Signatures);
    writeImportSection(Imports, DataSize, TableElems.size());
    writeFunctionSection(Functions);
    writeTableSection(Tables);
    // Skip the "memory" section; we import the memory instead.
    writeTagSection(TagTypes);
    writeGlobalSection(Globals);
    writeExportSection(Exports);
    const MCSymbol *IndirectFunctionTable =
        getContext().lookupSymbol("__indirect_function_table");
    writeElemSection(static_cast<const MCSymbolWasm *>(IndirectFunctionTable),
                     TableElems);
    writeDataCountSection();

    CodeSectionIndex = writeCodeSection(Asm, Functions);
    DataSectionIndex = writeDataSection(Asm);
  }

```
- **EN**: Implements logic around `writeHeader`, `writeTypeSection`, `writeImportSection`, `writeFunctionSection`, and 9 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `writeHeader`, `writeTypeSection`, `writeImportSection`, `writeFunctionSection`, and 9 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1922-1945
```cpp
  // The Sections in the COMDAT list have placeholder indices (their index among
  // custom sections, rather than among all sections). Fix them up here.
  for (auto &Group : Comdats) {
    for (auto &Entry : Group.second) {
      if (Entry.Kind == wasm::WASM_COMDAT_SECTION) {
        Entry.Index += SectionCount;
      }
    }
  }
  for (auto &CustomSection : CustomSections)
    writeCustomSection(CustomSection, Asm);

  if (Mode != DwoMode::DwoOnly) {
    writeLinkingMetaDataSection(SymbolInfos, InitFuncs, Comdats);

    writeRelocSection(CodeSectionIndex, "CODE", CodeRelocations);
    writeRelocSection(DataSectionIndex, "DATA", DataRelocations);
  }
  writeCustomRelocSections();
  if (ProducersSection)
    writeCustomSection(*ProducersSection, Asm);
  if (TargetFeaturesSection)
    writeCustomSection(*TargetFeaturesSection, Asm);

```
- **EN**: Implements logic around `writeCustomSection`, `writeLinkingMetaDataSection`, `writeRelocSection`, `writeCustomRelocSections`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `writeCustomSection`, `writeLinkingMetaDataSection`, `writeRelocSection`, `writeCustomRelocSections` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1946-1961
```cpp
  // TODO: Translate the .comment section to the output.
  return W->OS.tell() - StartOffset;
}

std::unique_ptr<MCObjectWriter>
llvm::createWasmObjectWriter(std::unique_ptr<MCWasmObjectTargetWriter> MOTW,
                             raw_pwrite_stream &OS) {
  return std::make_unique<WasmObjectWriter>(std::move(MOTW), OS);
}

std::unique_ptr<MCObjectWriter>
llvm::createWasmDwoObjectWriter(std::unique_ptr<MCWasmObjectTargetWriter> MOTW,
                                raw_pwrite_stream &OS,
                                raw_pwrite_stream &DwoOS) {
  return std::make_unique<WasmObjectWriter>(std::move(MOTW), OS, DwoOS);
}
```
- **EN**: Implements logic around `tell`, `createWasmObjectWriter`, `make_unique<WasmObjectWriter>`, `createWasmDwoObjectWriter`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `tell`, `createWasmObjectWriter`, `make_unique<WasmObjectWriter>`, `createWasmDwoObjectWriter` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

## Key Concepts / 关键概念

- **Relocations and fixups / 重定位与 fixup**:
  - **EN**: Bridges symbolic references to concrete addresses or relocation records
  - **CN**: 把符号引用连接到具体地址或重定位记录
- **MC instruction model / MC 指令模型**:
  - **EN**: Uses MCInst/MCOperand abstractions to represent machine instructions generically
  - **CN**: 使用 MCInst/MCOperand 抽象以通用方式表示机器指令
- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据
- **Object layout / 目标文件布局**:
  - **EN**: Organizes fragments into sections and computes final offsets
  - **CN**: 把片段组织到节区中并计算最终偏移

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ADT/STLExtras.h`, `llvm/BinaryFormat/Wasm.h`, `llvm/BinaryFormat/WasmTraits.h`, `llvm/Config/llvm-config.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCAssembler.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCObjectWriter.h`, `llvm/MC/MCSectionWasm.h`, `llvm/MC/MCSymbolWasm.h`, `llvm/MC/MCValue.h` ... (+7 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support, BinaryFormat
