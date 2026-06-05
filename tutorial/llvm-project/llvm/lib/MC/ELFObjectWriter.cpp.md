# ELFObjectWriter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/ELFObjectWriter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements ELF object file writer information.
  - **CN**: 实现 MC 层中某一特定目标文件格式的写出器支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32
```cpp
//===- lib/MC/ELFObjectWriter.cpp - ELF File Writer -----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements ELF object file writer information.
//
//===----------------------------------------------------------------------===//

#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/Twine.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/MC/MCAsmBackend.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCAssembler.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCELFExtras.h"
#include "llvm/MC/MCELFObjectWriter.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCFixup.h"
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/MC/MCSection.h"
#include "llvm/MC/MCSectionELF.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ADT/ArrayRef.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ADT/ArrayRef.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`。

### Lines 33-55
```cpp
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/MCSymbolELF.h"
#include "llvm/MC/MCTargetOptions.h"
#include "llvm/MC/MCValue.h"
#include "llvm/MC/StringTableBuilder.h"
#include "llvm/Support/Alignment.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/Compression.h"
#include "llvm/Support/Endian.h"
#include "llvm/Support/EndianStream.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/LEB128.h"
#include "llvm/Support/SMLoc.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/TargetParser/Host.h"
#include <cassert>
#include <cstddef>
#include <cstdint>
#include <memory>
#include <string>
#include <utility>
#include <vector>

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCSymbol.h`, `llvm/MC/MCSymbolELF.h`, `llvm/MC/MCTargetOptions.h`, `llvm/MC/MCValue.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCSymbol.h`, `llvm/MC/MCSymbolELF.h`, `llvm/MC/MCTargetOptions.h`, `llvm/MC/MCValue.h`。

### Lines 56-78
```cpp
using namespace llvm;

#define DEBUG_TYPE "elf-object-writer"

namespace {
namespace stats {

STATISTIC(ELFHeaderBytes, "Total size of ELF headers");
STATISTIC(SectionHeaderBytes, "Total size of section headers table");
STATISTIC(AllocTextBytes, "Total size of SHF_ALLOC text sections");
STATISTIC(AllocROBytes, "Total size of SHF_ALLOC readonly sections");
STATISTIC(AllocRWBytes, "Total size of SHF_ALLOC read-write sections");
STATISTIC(StrtabBytes, "Total size of SHT_STRTAB sections");
STATISTIC(SymtabBytes, "Total size of SHT_SYMTAB sections");
STATISTIC(RelocationBytes, "Total size of relocation sections");
STATISTIC(DynsymBytes, "Total size of SHT_DYNSYM sections");
STATISTIC(
    DebugBytes,
    "Total size of debug info sections (not including those written to .dwo)");
STATISTIC(UnwindBytes, "Total size of unwind sections");
STATISTIC(OtherBytes, "Total size of uncategorized sections");
STATISTIC(DwoBytes, "Total size of sections written to .dwo file");

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 79-96
```cpp
} // namespace stats

struct ELFWriter;

bool isDwoSection(const MCSectionELF &Sec) {
  return Sec.getName().ends_with(".dwo");
}

class SymbolTableWriter {
  ELFWriter &EWriter;
  bool Is64Bit;

  // indexes we are going to write to .symtab_shndx.
  std::vector<uint32_t> ShndxIndexes;

  // The numbel of symbols written so far.
  unsigned NumWritten;

```
- **EN**: Introduces declarations for `stats`, `ELFWriter`, `SymbolTableWriter`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `stats`, `ELFWriter`, `SymbolTableWriter` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 97-114
```cpp
  void createSymtabShndx();

  template <typename T> void write(T Value);

public:
  SymbolTableWriter(ELFWriter &EWriter, bool Is64Bit);

  void writeSymbol(uint32_t name, uint8_t info, uint64_t value, uint64_t size,
                   uint8_t other, uint32_t shndx, bool Reserved);

  ArrayRef<uint32_t> getShndxIndexes() const { return ShndxIndexes; }
};

struct ELFWriter {
  MCAssembler &Asm;
  ELFObjectWriter &OWriter;
  support::endian::Writer W;

```
- **EN**: Introduces declarations for `ELFWriter`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `ELFWriter` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 115-131
```cpp
  enum DwoMode {
    AllSections,
    NonDwoOnly,
    DwoOnly,
  } Mode;

  uint64_t symbolValue(const MCSymbol &Sym);
  bool isInSymtab(const MCSymbolELF &Symbol);

  /// Helper struct for containing some precomputed information on symbols.
  struct ELFSymbolData {
    const MCSymbolELF *Symbol;
    StringRef Name;
    uint32_t SectionIndex;
    uint32_t Order;
  };

```
- **EN**: Introduces declarations for `DwoMode`, `for`, `ELFSymbolData`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `DwoMode`, `for`, `ELFSymbolData` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 132-150
```cpp
  /// @}
  /// @name Symbol Table Data
  /// @{

  StringTableBuilder StrTabBuilder{StringTableBuilder::ELF};

  /// @}

  // This holds the symbol table index of the last local symbol.
  unsigned LastLocalSymbolIndex = ~0u;
  // This holds the .strtab section index.
  unsigned StringTableIndex = ~0u;
  // This holds the .symtab section index.
  unsigned SymbolTableIndex = ~0u;

  // Sections in the order they are to be output in the section table.
  std::vector<MCSectionELF *> SectionTable;
  unsigned addToSectionTable(MCSectionELF *Sec);

```
- **EN**: Implements logic around `addToSectionTable`; this block updates MC section or symbol state.
- **CN**: 围绕 `addToSectionTable` 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 151-167
```cpp
  // TargetObjectWriter wrappers.
  bool is64Bit() const;

  uint64_t align(Align Alignment);

  bool maybeWriteCompression(uint32_t ChType, uint64_t Size,
                             SmallVectorImpl<uint8_t> &CompressedContents,
                             Align Alignment);

public:
  ELFWriter(MCAssembler &Asm, ELFObjectWriter &OWriter, raw_pwrite_stream &OS,
            bool IsLittleEndian, DwoMode Mode)
      : Asm(Asm), OWriter(OWriter),
        W(OS,
          IsLittleEndian ? llvm::endianness::little : llvm::endianness::big),
        Mode(Mode) {}

```
- **EN**: Implements logic around `is64Bit`, `align`, `maybeWriteCompression`, `ELFWriter`, and 3 more symbols; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `is64Bit`, `align`, `maybeWriteCompression`, `ELFWriter`, and 3 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 168-185
```cpp
  MCContext &getContext() const { return Asm.getContext(); }

  void writeWord(uint64_t Word) {
    if (is64Bit())
      W.write<uint64_t>(Word);
    else
      W.write<uint32_t>(Word);
  }

  template <typename T> void write(T Val) {
    W.write(Val);
  }

  void writeHeader();

  void writeSymbol(SymbolTableWriter &Writer, uint32_t StringIndex,
                   ELFSymbolData &MSD);

```
- **EN**: Implements logic around `getContext`, `writeWord`, `write<uint64_t>`, `write<uint32_t>`, and 3 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getContext`, `writeWord`, `write<uint64_t>`, `write<uint32_t>`, and 3 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 186-201
```cpp
  // Map from a signature symbol to the group section index
  using RevGroupMapTy = DenseMap<const MCSymbol *, unsigned>;

  /// Compute the symbol table data
  ///
  /// \param Asm - The assembler.
  /// \param RevGroupMap - Maps a signature symbol to the group section.
  void computeSymbolTable(const RevGroupMapTy &RevGroupMap);

  void writeAddrsigSection();

  MCSectionELF *createRelocationSection(MCContext &Ctx,
                                        const MCSectionELF &Sec);

  void writeSectionHeaders();

```
- **EN**: Implements logic around `computeSymbolTable`, `writeAddrsigSection`, `createRelocationSection`, `writeSectionHeaders`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `computeSymbolTable`, `writeAddrsigSection`, `createRelocationSection`, `writeSectionHeaders` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 202-223
```cpp
  void writeSectionData(MCSection &Sec);

  void writeSectionHeaderEntry(uint32_t Name, uint32_t Type, uint64_t Flags,
                               uint64_t Address, uint64_t Offset, uint64_t Size,
                               uint32_t Link, uint32_t Info,
                               MaybeAlign Alignment, uint64_t EntrySize);

  void writeRelocations(const MCSectionELF &Sec);

  uint64_t writeObject();
  void writeSectionHeader(uint32_t GroupSymbolIndex, uint64_t Offset,
                          uint64_t Size, const MCSectionELF &Section);
};
} // end anonymous namespace

uint64_t ELFWriter::align(Align Alignment) {
  uint64_t Offset = W.OS.tell();
  uint64_t NewOffset = alignTo(Offset, Alignment);
  W.OS.write_zeros(NewOffset - Offset);
  return NewOffset;
}

```
- **EN**: Implements logic around `writeSectionData`, `writeSectionHeaderEntry`, `writeRelocations`, `writeObject`, and 5 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `writeSectionData`, `writeSectionHeaderEntry`, `writeRelocations`, `writeObject`, and 5 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 224-240
```cpp
unsigned ELFWriter::addToSectionTable(MCSectionELF *Sec) {
  SectionTable.push_back(Sec);
  StrTabBuilder.add(Sec->getName());
  return SectionTable.size();
}

void SymbolTableWriter::createSymtabShndx() {
  if (!ShndxIndexes.empty())
    return;

  ShndxIndexes.resize(NumWritten);
}

template <typename T> void SymbolTableWriter::write(T Value) {
  EWriter.write(Value);
}

```
- **EN**: Implements logic around `addToSectionTable`, `push_back`, `add`, `size`, and 3 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `addToSectionTable`, `push_back`, `add`, `size`, and 3 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 241-258
```cpp
SymbolTableWriter::SymbolTableWriter(ELFWriter &EWriter, bool Is64Bit)
    : EWriter(EWriter), Is64Bit(Is64Bit), NumWritten(0) {}

void SymbolTableWriter::writeSymbol(uint32_t name, uint8_t info, uint64_t value,
                                    uint64_t size, uint8_t other,
                                    uint32_t shndx, bool Reserved) {
  bool LargeIndex = shndx >= ELF::SHN_LORESERVE && !Reserved;

  if (LargeIndex)
    createSymtabShndx();

  if (!ShndxIndexes.empty()) {
    if (LargeIndex)
      ShndxIndexes.push_back(shndx);
    else
      ShndxIndexes.push_back(0);
  }

```
- **EN**: Implements logic around `SymbolTableWriter`, `EWriter`, `writeSymbol`, `createSymtabShndx`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `SymbolTableWriter`, `EWriter`, `writeSymbol`, `createSymtabShndx`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 259-276
```cpp
  uint16_t Index = LargeIndex ? uint16_t(ELF::SHN_XINDEX) : shndx;

  if (Is64Bit) {
    write(name);  // st_name
    write(info);  // st_info
    write(other); // st_other
    write(Index); // st_shndx
    write(value); // st_value
    write(size);  // st_size
  } else {
    write(name);            // st_name
    write(uint32_t(value)); // st_value
    write(uint32_t(size));  // st_size
    write(info);            // st_info
    write(other);           // st_other
    write(Index);           // st_shndx
  }

```
- **EN**: Implements logic around `uint16_t`, `write`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `uint16_t`, `write` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 277-293
```cpp
  ++NumWritten;
}

bool ELFWriter::is64Bit() const {
  return OWriter.TargetObjectWriter->is64Bit();
}

// Emit the ELF header.
void ELFWriter::writeHeader() {
  // ELF Header
  // ----------
  //
  // Note
  // ----
  // emitWord method behaves differently for ELF32 and ELF64, writing
  // 4 bytes in the former and 8 in the latter.

```
- **EN**: Implements logic around `is64Bit`, `writeHeader`; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `is64Bit`, `writeHeader` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 294-312
```cpp
  W.OS << ELF::ElfMagic; // e_ident[EI_MAG0] to e_ident[EI_MAG3]

  W.OS << char(is64Bit() ? ELF::ELFCLASS64 : ELF::ELFCLASS32); // e_ident[EI_CLASS]

  // e_ident[EI_DATA]
  W.OS << char(W.Endian == llvm::endianness::little ? ELF::ELFDATA2LSB
                                                    : ELF::ELFDATA2MSB);

  W.OS << char(ELF::EV_CURRENT);        // e_ident[EI_VERSION]
  // e_ident[EI_OSABI]
  uint8_t OSABI = OWriter.TargetObjectWriter->getOSABI();
  W.OS << char(OSABI == ELF::ELFOSABI_NONE && OWriter.seenGnuAbi()
                   ? int(ELF::ELFOSABI_GNU)
                   : OSABI);
  // e_ident[EI_ABIVERSION]
  W.OS << char(OWriter.OverrideABIVersion
                   ? *OWriter.OverrideABIVersion
                   : OWriter.TargetObjectWriter->getABIVersion());

```
- **EN**: Implements logic around `char`, `getOSABI`, `int`, `getABIVersion`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `char`, `getOSABI`, `int`, `getABIVersion` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 313-330
```cpp
  W.OS.write_zeros(ELF::EI_NIDENT - ELF::EI_PAD);

  W.write<uint16_t>(ELF::ET_REL);             // e_type

  W.write<uint16_t>(OWriter.TargetObjectWriter->getEMachine()); // e_machine = target

  W.write<uint32_t>(ELF::EV_CURRENT);         // e_version
  writeWord(0); // e_entry, no entry point in .o file
  writeWord(0); // e_phoff, no program header for .o
  writeWord(0); // e_shoff = sec hdr table off in bytes

  // e_flags = whatever the target wants
  W.write<uint32_t>(OWriter.getELFHeaderEFlags());

  // e_ehsize = ELF header size
  W.write<uint16_t>(is64Bit() ? sizeof(ELF::Elf64_Ehdr)
                              : sizeof(ELF::Elf32_Ehdr));

```
- **EN**: Implements logic around `write_zeros`, `write<uint16_t>`, `write<uint32_t>`, `writeWord`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `write_zeros`, `write<uint16_t>`, `write<uint32_t>`, `writeWord` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 331-349
```cpp
  W.write<uint16_t>(0);                  // e_phentsize = prog header entry size
  W.write<uint16_t>(0);                  // e_phnum = # prog header entries = 0

  // e_shentsize = Section header entry size
  W.write<uint16_t>(is64Bit() ? sizeof(ELF::Elf64_Shdr)
                              : sizeof(ELF::Elf32_Shdr));

  // e_shnum     = # of section header ents
  W.write<uint16_t>(0);

  // e_shstrndx  = Section # of '.strtab'
  assert(StringTableIndex < ELF::SHN_LORESERVE);
  W.write<uint16_t>(StringTableIndex);
}

uint64_t ELFWriter::symbolValue(const MCSymbol &Sym) {
  if (Sym.isCommon())
    return Sym.getCommonAlignment()->value();

```
- **EN**: Implements logic around `write<uint16_t>`, `assert`, `symbolValue`, `getCommonAlignment`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `write<uint16_t>`, `assert`, `symbolValue`, `getCommonAlignment` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 350-381
```cpp
  uint64_t Res;
  if (!Asm.getSymbolOffset(Sym, Res))
    return 0;

  if (Asm.isThumbFunc(&Sym))
    Res |= 1;

  return Res;
}

static uint8_t mergeTypeForSet(uint8_t origType, uint8_t newType) {
  uint8_t Type = newType;

  // Propagation rules:
  // IFUNC > FUNC > OBJECT > NOTYPE
  // TLS_OBJECT > OBJECT > NOTYPE
  //
  // dont let the new type degrade the old type
  switch (origType) {
  default:
    break;
  case ELF::STT_GNU_IFUNC:
    if (Type == ELF::STT_FUNC || Type == ELF::STT_OBJECT ||
        Type == ELF::STT_NOTYPE || Type == ELF::STT_TLS)
      Type = ELF::STT_GNU_IFUNC;
    break;
  case ELF::STT_FUNC:
    if (Type == ELF::STT_OBJECT || Type == ELF::STT_NOTYPE ||
        Type == ELF::STT_TLS)
      Type = ELF::STT_FUNC;
    break;
  case ELF::STT_OBJECT:
```
- **EN**: Implements logic around `mergeTypeForSet`; this block updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `mergeTypeForSet` 实现具体逻辑；这一段更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 382-408
```cpp
    if (Type == ELF::STT_NOTYPE)
      Type = ELF::STT_OBJECT;
    break;
  case ELF::STT_TLS:
    if (Type == ELF::STT_OBJECT || Type == ELF::STT_NOTYPE ||
        Type == ELF::STT_GNU_IFUNC || Type == ELF::STT_FUNC)
      Type = ELF::STT_TLS;
    break;
  }

  return Type;
}

static bool isIFunc(const MCSymbolELF *Symbol) {
  while (Symbol->getType() != ELF::STT_GNU_IFUNC) {
    const MCSymbolRefExpr *Value;
    if (!Symbol->isVariable() ||
        !(Value = dyn_cast<MCSymbolRefExpr>(Symbol->getVariableValue())) ||
        Value->getSpecifier() ||
        mergeTypeForSet(Symbol->getType(), ELF::STT_GNU_IFUNC) !=
            ELF::STT_GNU_IFUNC)
      return false;
    Symbol = &static_cast<const MCSymbolELF &>(Value->getSymbol());
  }
  return true;
}

```
- **EN**: Implements logic around `isIFunc`, `dyn_cast<MCSymbolRefExpr>`, `getSpecifier`, `mergeTypeForSet`, and 1 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `isIFunc`, `dyn_cast<MCSymbolRefExpr>`, `getSpecifier`, `mergeTypeForSet`, and 1 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 409-427
```cpp
void ELFWriter::writeSymbol(SymbolTableWriter &Writer, uint32_t StringIndex,
                            ELFSymbolData &MSD) {
  auto &Symbol = static_cast<const MCSymbolELF &>(*MSD.Symbol);
  auto *Base = static_cast<const MCSymbolELF *>(Asm.getBaseSymbol(Symbol));

  // This has to be in sync with when computeSymbolTable uses SHN_ABS or
  // SHN_COMMON.
  bool IsReserved = !Base || Symbol.isCommon();

  // Binding and Type share the same byte as upper and lower nibbles
  uint8_t Binding = Symbol.getBinding();
  uint8_t Type = Symbol.getType();
  if (isIFunc(&Symbol))
    Type = ELF::STT_GNU_IFUNC;
  if (Base) {
    Type = mergeTypeForSet(Type, Base->getType());
  }
  uint8_t Info = (Binding << 4) | Type;

```
- **EN**: Implements logic around `writeSymbol`, `getBaseSymbol`, `isCommon`, `getBinding`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `writeSymbol`, `getBaseSymbol`, `isCommon`, `getBinding`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 428-456
```cpp
  // Other and Visibility share the same byte with Visibility using the lower
  // 2 bits
  uint8_t Visibility = Symbol.getVisibility();
  uint8_t Other = Symbol.getOther() | Visibility;

  uint64_t Value = symbolValue(*MSD.Symbol);
  uint64_t Size = 0;

  const MCExpr *ESize = MSD.Symbol->getSize();
  if (!ESize && Base) {
    // For expressions like .set y, x+1, if y's size is unset, inherit from x.
    ESize = Base->getSize();

    // For `.size x, 2; y = x; .size y, 1; z = y; z1 = z; .symver y, y@v1`, z,
    // z1, and y@v1's st_size equals y's. However, `Base` is `x` which will give
    // us 2. Follow the MCSymbolRefExpr assignment chain, which covers most
    // needs. MCBinaryExpr is not handled.
    const MCSymbolELF *Sym = &Symbol;
    while (Sym->isVariable()) {
      if (auto *Expr = dyn_cast<MCSymbolRefExpr>(Sym->getVariableValue())) {
        Sym = static_cast<const MCSymbolELF *>(&Expr->getSymbol());
        if (!Sym->getSize())
          continue;
        ESize = Sym->getSize();
      }
      break;
    }
  }

```
- **EN**: Implements logic around `getVisibility`, `getOther`, `symbolValue`, `getSize`, and 1 more symbols; this block updates MC section or symbol state.
- **CN**: 围绕 `getVisibility`, `getOther`, `symbolValue`, `getSize`, and 1 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 457-472
```cpp
  if (ESize) {
    int64_t Res;
    if (!ESize->evaluateKnownAbsolute(Res, Asm))
      report_fatal_error("Size expression must be absolute.");
    Size = Res;
  }

  // Write out the symbol table entry
  Writer.writeSymbol(StringIndex, Info, Value, Size, Other, MSD.SectionIndex,
                     IsReserved);
}

bool ELFWriter::isInSymtab(const MCSymbolELF &Symbol) {
  if (Symbol.isUsedInReloc() || Symbol.isSignature())
    return true;

```
- **EN**: Implements logic around `report_fatal_error`, `writeSymbol`, `isInSymtab`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `report_fatal_error`, `writeSymbol`, `isInSymtab` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 473-492
```cpp
  if (OWriter.Renames.count(&Symbol))
    return false;

  if (Symbol.isVariable()) {
    const MCExpr *Expr = Symbol.getVariableValue();
    // Target Expressions that are always inlined do not appear in the symtab
    if (const auto *T = dyn_cast<MCTargetExpr>(Expr))
      if (T->inlineAssignedExpr())
        return false;
    // The .weakref alias does not appear in the symtab.
    if (Symbol.isWeakref())
      return false;

    if (Symbol.isUndefined()) {
      // FIXME: this is here just to diagnose the case of a var = commmon_sym.
      Asm.getBaseSymbol(Symbol);
      return false;
    }
  }

```
- **EN**: Implements logic around `getVariableValue`, `getBaseSymbol`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getVariableValue`, `getBaseSymbol` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 493-509
```cpp
  if (Symbol.isTemporary())
    return false;

  return Symbol.getType() != ELF::STT_SECTION;
}

void ELFWriter::computeSymbolTable(const RevGroupMapTy &RevGroupMap) {
  MCContext &Ctx = Asm.getContext();
  SymbolTableWriter Writer(*this, is64Bit());

  // Symbol table
  unsigned EntrySize = is64Bit() ? ELF::SYMENTRY_SIZE64 : ELF::SYMENTRY_SIZE32;
  MCSectionELF *SymtabSection =
      Ctx.getELFSection(".symtab", ELF::SHT_SYMTAB, 0, EntrySize);
  SymtabSection->setAlignment(is64Bit() ? Align(8) : Align(4));
  SymbolTableIndex = addToSectionTable(SymtabSection);

```
- **EN**: Implements logic around `getType`, `computeSymbolTable`, `getContext`, `Writer`, and 4 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getType`, `computeSymbolTable`, `getContext`, `Writer`, and 4 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 510-528
```cpp
  uint64_t SecStart = align(SymtabSection->getAlign());

  // The first entry is the undefined symbol entry.
  Writer.writeSymbol(0, 0, 0, 0, 0, 0, false);

  std::vector<ELFSymbolData> LocalSymbolData;
  std::vector<ELFSymbolData> ExternalSymbolData;
  MutableArrayRef<std::pair<std::string, size_t>> FileNames =
      OWriter.getFileNames();
  for (const std::pair<std::string, size_t> &F : FileNames)
    StrTabBuilder.add(F.first);

  // Add the data for the symbols.
  bool HasLargeSectionIndex = false;
  for (auto It : llvm::enumerate(Asm.symbols())) {
    auto &Symbol = static_cast<const MCSymbolELF &>(It.value());
    if (!isInSymtab(Symbol))
      continue;

```
- **EN**: Implements logic around `align`, `writeSymbol`, `getFileNames`, `add`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `align`, `writeSymbol`, `getFileNames`, `add`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 529-560
```cpp
    if (Symbol.isTemporary() && Symbol.isUndefined()) {
      Ctx.reportError(SMLoc(), "Undefined temporary symbol " + Symbol.getName());
      continue;
    }

    ELFSymbolData MSD;
    MSD.Symbol = static_cast<const MCSymbolELF *>(&Symbol);
    MSD.Order = It.index();

    bool Local = Symbol.getBinding() == ELF::STB_LOCAL;
    assert(Local || !Symbol.isTemporary());

    if (Symbol.isAbsolute()) {
      MSD.SectionIndex = ELF::SHN_ABS;
    } else if (Symbol.isCommon()) {
      auto Shndx = Symbol.getIndex();
      if (!Shndx) {
        assert(!Local);
        Shndx = ELF::SHN_COMMON;
      }
      MSD.SectionIndex = Shndx;
    } else if (Symbol.isUndefined()) {
      if (Symbol.isSignature() && !Symbol.isUsedInReloc()) {
        MSD.SectionIndex = RevGroupMap.lookup(&Symbol);
        if (MSD.SectionIndex >= ELF::SHN_LORESERVE)
          HasLargeSectionIndex = true;
      } else {
        MSD.SectionIndex = ELF::SHN_UNDEF;
      }
    } else {
      const MCSectionELF &Section =
          static_cast<const MCSectionELF &>(Symbol.getSection());
```
- **EN**: Implements logic around `reportError`, `index`, `getBinding`, `assert`, and 3 more symbols; this block handles relocation, fixup, or symbol-resolution work; updates MC section or symbol state.
- **CN**: 围绕 `reportError`, `index`, `getBinding`, `assert`, and 3 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，更新 MC 节区或符号状态。

### Lines 561-577
```cpp
      assert(Section.isRegistered());
      if (Mode == NonDwoOnly && isDwoSection(Section))
        continue;
      MSD.SectionIndex = Section.getOrdinal();
      assert(MSD.SectionIndex && "Invalid section index!");
      if (MSD.SectionIndex >= ELF::SHN_LORESERVE)
        HasLargeSectionIndex = true;
    }

    // Temporary symbols generated for certain assembler features (.eh_frame,
    // .debug_line) of an empty name may be referenced by relocations due to
    // linker relaxation. Rename them to ".L0 " to match the gas fake label name
    // and allow ld/objcopy --discard-locals to discard such symbols.
    StringRef Name = Symbol.getName();
    if (Name.empty())
      Name = ".L0 ";

```
- **EN**: Implements logic around `assert`, `getOrdinal`, `getName`; this block handles relocation, fixup, or symbol-resolution work; updates MC section or symbol state.
- **CN**: 围绕 `assert`, `getOrdinal`, `getName` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，更新 MC 节区或符号状态。

### Lines 578-599
```cpp
    // Sections have their own string table
    if (Symbol.getType() != ELF::STT_SECTION) {
      MSD.Name = Name;
      StrTabBuilder.add(Name);
    }

    if (Local)
      LocalSymbolData.push_back(MSD);
    else
      ExternalSymbolData.push_back(MSD);
  }

  // This holds the .symtab_shndx section index.
  unsigned SymtabShndxSectionIndex = 0;

  if (HasLargeSectionIndex) {
    MCSectionELF *SymtabShndxSection =
        Ctx.getELFSection(".symtab_shndx", ELF::SHT_SYMTAB_SHNDX, 0, 4);
    SymtabShndxSectionIndex = addToSectionTable(SymtabShndxSection);
    SymtabShndxSection->setAlignment(Align(4));
  }

```
- **EN**: Implements logic around `add`, `push_back`, `getELFSection`, `addToSectionTable`, and 1 more symbols; this block updates MC section or symbol state.
- **CN**: 围绕 `add`, `push_back`, `getELFSection`, `addToSectionTable`, and 1 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 600-617
```cpp
  StrTabBuilder.finalize();

  // Make the first STT_FILE precede previous local symbols.
  unsigned Index = 1;
  auto FileNameIt = FileNames.begin();
  if (!FileNames.empty())
    FileNames[0].second = 0;

  for (ELFSymbolData &MSD : LocalSymbolData) {
    // Emit STT_FILE symbols before their associated local symbols.
    for (; FileNameIt != FileNames.end() && FileNameIt->second <= MSD.Order;
         ++FileNameIt) {
      Writer.writeSymbol(StrTabBuilder.getOffset(FileNameIt->first),
                         ELF::STT_FILE | ELF::STB_LOCAL, 0, 0, ELF::STV_DEFAULT,
                         ELF::SHN_ABS, true);
      ++Index;
    }

```
- **EN**: Implements logic around `finalize`, `begin`, `writeSymbol`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `finalize`, `begin`, `writeSymbol` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 618-633
```cpp
    unsigned StringIndex = MSD.Symbol->getType() == ELF::STT_SECTION
                               ? 0
                               : StrTabBuilder.getOffset(MSD.Name);
    MSD.Symbol->setIndex(Index++);
    writeSymbol(Writer, StringIndex, MSD);
  }
  for (; FileNameIt != FileNames.end(); ++FileNameIt) {
    Writer.writeSymbol(StrTabBuilder.getOffset(FileNameIt->first),
                       ELF::STT_FILE | ELF::STB_LOCAL, 0, 0, ELF::STV_DEFAULT,
                       ELF::SHN_ABS, true);
    ++Index;
  }

  // Write the symbol table entries.
  LastLocalSymbolIndex = Index;

```
- **EN**: Implements logic around `getType`, `getOffset`, `setIndex`, `writeSymbol`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `getType`, `getOffset`, `setIndex`, `writeSymbol` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 634-650
```cpp
  for (ELFSymbolData &MSD : ExternalSymbolData) {
    unsigned StringIndex = StrTabBuilder.getOffset(MSD.Name);
    MSD.Symbol->setIndex(Index++);
    writeSymbol(Writer, StringIndex, MSD);
    assert(MSD.Symbol->getBinding() != ELF::STB_LOCAL);
  }

  uint64_t SecEnd = W.OS.tell();
  SymtabSection->setOffsets(SecStart, SecEnd);

  ArrayRef<uint32_t> ShndxIndexes = Writer.getShndxIndexes();
  if (ShndxIndexes.empty()) {
    assert(SymtabShndxSectionIndex == 0);
    return;
  }
  assert(SymtabShndxSectionIndex != 0);

```
- **EN**: Implements logic around `getOffset`, `setIndex`, `writeSymbol`, `assert`, and 3 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `getOffset`, `setIndex`, `writeSymbol`, `assert`, and 3 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 651-669
```cpp
  SecStart = W.OS.tell();
  MCSectionELF *SymtabShndxSection = SectionTable[SymtabShndxSectionIndex - 1];
  for (uint32_t Index : ShndxIndexes)
    write(Index);
  SecEnd = W.OS.tell();
  SymtabShndxSection->setOffsets(SecStart, SecEnd);
}

void ELFWriter::writeAddrsigSection() {
  for (const MCSymbol *Sym : OWriter.getAddrsigSyms())
    if (Sym->getIndex() != 0)
      encodeULEB128(Sym->getIndex(), W.OS);
}

MCSectionELF *ELFWriter::createRelocationSection(MCContext &Ctx,
                                                 const MCSectionELF &Sec) {
  if (OWriter.Relocations[&Sec].empty())
    return nullptr;

```
- **EN**: Implements logic around `tell`, `write`, `setOffsets`, `writeAddrsigSection`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `tell`, `write`, `setOffsets`, `writeAddrsigSection`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 670-689
```cpp
  unsigned Flags = ELF::SHF_INFO_LINK;
  if (Sec.getFlags() & ELF::SHF_GROUP)
    Flags = ELF::SHF_GROUP;

  const StringRef SectionName = Sec.getName();
  const MCTargetOptions &TO = Ctx.getTargetOptions();
  if (TO.Crel) {
    MCSectionELF *RelaSection =
        Ctx.createELFRelSection(".crel" + SectionName, ELF::SHT_CREL, Flags,
                                /*EntrySize=*/1, Sec.getGroup(), &Sec);
    return RelaSection;
  }

  const bool Rela = OWriter.usesRela(TO, Sec);
  unsigned EntrySize;
  if (Rela)
    EntrySize = is64Bit() ? sizeof(ELF::Elf64_Rela) : sizeof(ELF::Elf32_Rela);
  else
    EntrySize = is64Bit() ? sizeof(ELF::Elf64_Rel) : sizeof(ELF::Elf32_Rel);

```
- **EN**: Implements logic around `getName`, `getTargetOptions`, `createELFRelSection`, `getGroup`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getName`, `getTargetOptions`, `createELFRelSection`, `getGroup`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 690-721
```cpp
  MCSectionELF *RelaSection =
      Ctx.createELFRelSection(((Rela ? ".rela" : ".rel") + SectionName),
                              Rela ? ELF::SHT_RELA : ELF::SHT_REL, Flags,
                              EntrySize, Sec.getGroup(), &Sec);
  RelaSection->setAlignment(is64Bit() ? Align(8) : Align(4));
  return RelaSection;
}

// Include the debug info compression header.
bool ELFWriter::maybeWriteCompression(
    uint32_t ChType, uint64_t Size,
    SmallVectorImpl<uint8_t> &CompressedContents, Align Alignment) {
  uint64_t HdrSize =
      is64Bit() ? sizeof(ELF::Elf64_Chdr) : sizeof(ELF::Elf32_Chdr);
  if (Size <= HdrSize + CompressedContents.size())
    return false;
  // Platform specific header is followed by compressed data.
  if (is64Bit()) {
    // Write Elf64_Chdr header.
    write(static_cast<ELF::Elf64_Word>(ChType));
    write(static_cast<ELF::Elf64_Word>(0)); // ch_reserved field.
    write(static_cast<ELF::Elf64_Xword>(Size));
    write(static_cast<ELF::Elf64_Xword>(Alignment.value()));
  } else {
    // Write Elf32_Chdr header otherwise.
    write(static_cast<ELF::Elf32_Word>(ChType));
    write(static_cast<ELF::Elf32_Word>(Size));
    write(static_cast<ELF::Elf32_Word>(Alignment.value()));
  }
  return true;
}

```
- **EN**: Implements logic around `createELFRelSection`, `getGroup`, `setAlignment`, `maybeWriteCompression`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `createELFRelSection`, `getGroup`, `setAlignment`, `maybeWriteCompression`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 722-740
```cpp
void ELFWriter::writeSectionData(MCSection &Sec) {
  MCSectionELF &Section = static_cast<MCSectionELF &>(Sec);
  StringRef SectionName = Section.getName();
  auto &Ctx = Asm.getContext();
  const DebugCompressionType CompressionType =
      Ctx.getTargetOptions().CompressDebugSections;
  if (CompressionType == DebugCompressionType::None ||
      !SectionName.starts_with(".debug_")) {
    Asm.writeSectionData(W.OS, &Section);
    return;
  }

  SmallVector<char, 128> UncompressedData;
  raw_svector_ostream VecOS(UncompressedData);
  Asm.writeSectionData(VecOS, &Section);
  ArrayRef<uint8_t> Uncompressed =
      ArrayRef(reinterpret_cast<uint8_t *>(UncompressedData.data()),
               UncompressedData.size());

```
- **EN**: Implements logic around `writeSectionData`, `getName`, `getContext`, `getTargetOptions`, and 4 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `writeSectionData`, `getName`, `getContext`, `getTargetOptions`, and 4 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 741-760
```cpp
  SmallVector<uint8_t, 128> Compressed;
  uint32_t ChType;
  switch (CompressionType) {
  case DebugCompressionType::None:
    llvm_unreachable("has been handled");
  case DebugCompressionType::Zlib:
    ChType = ELF::ELFCOMPRESS_ZLIB;
    break;
  case DebugCompressionType::Zstd:
    ChType = ELF::ELFCOMPRESS_ZSTD;
    break;
  }
  compression::compress(compression::Params(CompressionType), Uncompressed,
                        Compressed);
  if (!maybeWriteCompression(ChType, UncompressedData.size(), Compressed,
                             Sec.getAlign())) {
    W.OS << UncompressedData;
    return;
  }

```
- **EN**: Implements logic around `llvm_unreachable`, `compress`, `getAlign`; this block drives emission, layout, or binary encoding behavior; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `llvm_unreachable`, `compress`, `getAlign` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 761-785
```cpp
  Section.setFlags(Section.getFlags() | ELF::SHF_COMPRESSED);
  // Alignment field should reflect the requirements of
  // the compressed section header.
  Section.setAlignment(is64Bit() ? Align(8) : Align(4));
  W.OS << toStringRef(Compressed);
}

void ELFWriter::writeSectionHeaderEntry(uint32_t Name, uint32_t Type,
                                        uint64_t Flags, uint64_t Address,
                                        uint64_t Offset, uint64_t Size,
                                        uint32_t Link, uint32_t Info,
                                        MaybeAlign Alignment,
                                        uint64_t EntrySize) {
  W.write<uint32_t>(Name);        // sh_name: index into string table
  W.write<uint32_t>(Type);        // sh_type
  writeWord(Flags);               // sh_flags
  writeWord(Address);             // sh_addr
  writeWord(Offset);              // sh_offset
  writeWord(Size);                // sh_size
  W.write<uint32_t>(Link);        // sh_link
  W.write<uint32_t>(Info);        // sh_info
  writeWord(Alignment ? Alignment->value() : 0); // sh_addralign
  writeWord(EntrySize);                          // sh_entsize
}

```
- **EN**: Implements logic around `setFlags`, `setAlignment`, `toStringRef`, `writeSectionHeaderEntry`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `setFlags`, `setAlignment`, `toStringRef`, `writeSectionHeaderEntry`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 786-803
```cpp
template <bool Is64>
static void encodeCrel(ArrayRef<ELFRelocationEntry> Relocs, raw_ostream &OS) {
  using uint = std::conditional_t<Is64, uint64_t, uint32_t>;
  ELF::encodeCrel<Is64>(OS, Relocs, [&](const ELFRelocationEntry &R) {
    uint32_t SymIdx = R.Symbol ? R.Symbol->getIndex() : 0;
    return ELF::Elf_Crel<Is64>{static_cast<uint>(R.Offset), SymIdx, R.Type,
                               std::make_signed_t<uint>(R.Addend)};
  });
}

void ELFWriter::writeRelocations(const MCSectionELF &Sec) {
  std::vector<ELFRelocationEntry> &Relocs = OWriter.Relocations[&Sec];
  const MCTargetOptions &TO = getContext().getTargetOptions();
  const bool Rela = OWriter.usesRela(TO, Sec);

  // Sort the relocation entries. MIPS needs this.
  OWriter.TargetObjectWriter->sortRelocs(Relocs);

```
- **EN**: Implements logic around `encodeCrel`, `encodeCrel<Is64>`, `getIndex`, `static_cast<uint>`, and 5 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `encodeCrel`, `encodeCrel<Is64>`, `getIndex`, `static_cast<uint>`, and 5 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 804-835
```cpp
  if (OWriter.TargetObjectWriter->getEMachine() == ELF::EM_MIPS) {
    for (const ELFRelocationEntry &Entry : Relocs) {
      uint32_t SymIdx = Entry.Symbol ? Entry.Symbol->getIndex() : 0;
      if (is64Bit()) {
        write(Entry.Offset);
        write(uint32_t(SymIdx));
        write(OWriter.TargetObjectWriter->getRSsym(Entry.Type));
        write(OWriter.TargetObjectWriter->getRType3(Entry.Type));
        write(OWriter.TargetObjectWriter->getRType2(Entry.Type));
        write(OWriter.TargetObjectWriter->getRType(Entry.Type));
        if (Rela)
          write(Entry.Addend);
      } else {
        write(uint32_t(Entry.Offset));
        ELF::Elf32_Rela ERE32;
        ERE32.setSymbolAndType(SymIdx, Entry.Type);
        write(ERE32.r_info);
        if (Rela)
          write(uint32_t(Entry.Addend));
        if (uint32_t RType =
                OWriter.TargetObjectWriter->getRType2(Entry.Type)) {
          write(uint32_t(Entry.Offset));
          ERE32.setSymbolAndType(0, RType);
          write(ERE32.r_info);
          write(uint32_t(0));
        }
        if (uint32_t RType =
                OWriter.TargetObjectWriter->getRType3(Entry.Type)) {
          write(uint32_t(Entry.Offset));
          ERE32.setSymbolAndType(0, RType);
          write(ERE32.r_info);
          write(uint32_t(0));
```
- **EN**: Implements logic around `getIndex`, `write`, `setSymbolAndType`, `getRType2`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `getIndex`, `write`, `setSymbolAndType`, `getRType2`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 836-865
```cpp
        }
      }
    }
  } else if (TO.Crel) {
    if (is64Bit())
      encodeCrel<true>(Relocs, W.OS);
    else
      encodeCrel<false>(Relocs, W.OS);
  } else {
    for (const ELFRelocationEntry &Entry : Relocs) {
      uint32_t Symidx = Entry.Symbol ? Entry.Symbol->getIndex() : 0;
      if (is64Bit()) {
        write(Entry.Offset);
        ELF::Elf64_Rela ERE;
        ERE.setSymbolAndType(Symidx, Entry.Type);
        write(ERE.r_info);
        if (Rela)
          write(Entry.Addend);
      } else {
        write(uint32_t(Entry.Offset));
        ELF::Elf32_Rela ERE;
        ERE.setSymbolAndType(Symidx, Entry.Type);
        write(ERE.r_info);
        if (Rela)
          write(uint32_t(Entry.Addend));
      }
    }
  }
}

```
- **EN**: Implements logic around `encodeCrel<true>`, `encodeCrel<false>`, `getIndex`, `write`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `encodeCrel<true>`, `encodeCrel<false>`, `getIndex`, `write`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 866-888
```cpp
void ELFWriter::writeSectionHeader(uint32_t GroupSymbolIndex, uint64_t Offset,
                                   uint64_t Size, const MCSectionELF &Section) {
  uint64_t sh_link = 0;
  uint64_t sh_info = 0;

  switch(Section.getType()) {
  default:
    // Nothing to do.
    break;

  case ELF::SHT_DYNAMIC:
    llvm_unreachable("SHT_DYNAMIC in a relocatable object");

  case ELF::SHT_REL:
  case ELF::SHT_RELA:
  case ELF::SHT_CREL: {
    sh_link = SymbolTableIndex;
    assert(sh_link && ".symtab not found");
    const MCSection *InfoSection = Section.getLinkedToSection();
    sh_info = InfoSection->getOrdinal();
    break;
  }

```
- **EN**: Implements logic around `writeSectionHeader`, `llvm_unreachable`, `assert`, `getLinkedToSection`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `writeSectionHeader`, `llvm_unreachable`, `assert`, `getLinkedToSection`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 889-905
```cpp
  case ELF::SHT_SYMTAB:
    sh_link = StringTableIndex;
    sh_info = LastLocalSymbolIndex;
    break;

  case ELF::SHT_SYMTAB_SHNDX:
  case ELF::SHT_LLVM_CALL_GRAPH_PROFILE:
  case ELF::SHT_LLVM_ADDRSIG:
    sh_link = SymbolTableIndex;
    break;

  case ELF::SHT_GROUP:
    sh_link = SymbolTableIndex;
    sh_info = GroupSymbolIndex;
    break;
  }

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 906-923
```cpp
  if (Section.getFlags() & ELF::SHF_LINK_ORDER) {
    // If the value in the associated metadata is not a definition, Sym will be
    // undefined. Represent this with sh_link=0.
    const MCSymbol *Sym = Section.getLinkedToSymbol();
    if (Sym && Sym->isInSection())
      sh_link = Sym->getSection().getOrdinal();
  }

  writeSectionHeaderEntry(StrTabBuilder.getOffset(Section.getName()),
                          Section.getType(), Section.getFlags(), 0, Offset,
                          Size, sh_link, sh_info, Section.getAlign(),
                          Section.getEntrySize());
}

void ELFWriter::writeSectionHeaders() {
  uint64_t Start = W.OS.tell();
  const unsigned NumSections = SectionTable.size();

```
- **EN**: Implements logic around `getLinkedToSymbol`, `getSection`, `writeSectionHeaderEntry`, `getType`, and 5 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `getLinkedToSymbol`, `getSection`, `writeSectionHeaderEntry`, `getType`, and 5 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 924-944
```cpp
  // Null section first.
  uint64_t FirstSectionSize =
      (NumSections + 1) >= ELF::SHN_LORESERVE ? NumSections + 1 : 0;
  writeSectionHeaderEntry(0, 0, 0, 0, 0, FirstSectionSize, 0, 0, std::nullopt,
                          0);

  for (const MCSectionELF *Section : SectionTable) {
    uint32_t GroupSymbolIndex;
    unsigned Type = Section->getType();
    if (Type != ELF::SHT_GROUP)
      GroupSymbolIndex = 0;
    else
      GroupSymbolIndex = Section->getGroup()->getIndex();

    std::pair<uint64_t, uint64_t> Offsets = Section->getOffsets();
    uint64_t Size;
    if (Type == ELF::SHT_NOBITS)
      Size = Asm.getSectionAddressSize(*Section);
    else
      Size = Offsets.second - Offsets.first;

```
- **EN**: Implements logic around `writeSectionHeaderEntry`, `getType`, `getGroup`, `getOffsets`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `writeSectionHeaderEntry`, `getType`, `getGroup`, `getOffsets`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 945-976
```cpp
    auto SectionHasFlag = [&](uint64_t Flag) -> bool {
      return Section->getFlags() & Flag;
    };

    if (Mode == DwoOnly) {
      stats::DwoBytes += Size;
    } else if (Section->getName().starts_with(".debug")) {
      stats::DebugBytes += Size;
    } else if (Section->getName().starts_with(".eh_frame")) {
      stats::UnwindBytes += Size;
    } else if (SectionHasFlag(ELF::SHF_ALLOC)) {
      if (SectionHasFlag(ELF::SHF_EXECINSTR)) {
        stats::AllocTextBytes += Size;
      } else if (SectionHasFlag(ELF::SHF_WRITE)) {
        stats::AllocRWBytes += Size;
      } else {
        stats::AllocROBytes += Size;
      }
    } else {
      switch (Section->getType()) {
      case ELF::SHT_STRTAB:
        stats::StrtabBytes += Size;
        break;
      case ELF::SHT_SYMTAB:
        stats::SymtabBytes += Size;
        break;
      case ELF::SHT_DYNSYM:
        stats::DynsymBytes += Size;
        break;
      case ELF::SHT_REL:
      case ELF::SHT_RELA:
      case ELF::SHT_CREL:
```
- **EN**: Implements logic around `getFlags`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getFlags` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 977-993
```cpp
        stats::RelocationBytes += Size;
        break;
      default:
        stats::OtherBytes += Size;
        break;
      }
    }

    writeSectionHeader(GroupSymbolIndex, Offsets.first, Size, *Section);
  }

  stats::SectionHeaderBytes += W.OS.tell() - Start;
}

uint64_t ELFWriter::writeObject() {
  uint64_t StartOffset = W.OS.tell();

```
- **EN**: Implements logic around `writeSectionHeader`, `tell`, `writeObject`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `writeSectionHeader`, `tell`, `writeObject` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 994-1017
```cpp
  MCContext &Ctx = getContext();
  MCSectionELF *StrtabSection =
      Ctx.getELFSection(".strtab", ELF::SHT_STRTAB, 0);
  StringTableIndex = addToSectionTable(StrtabSection);

  RevGroupMapTy RevGroupMap;

  // Write out the ELF header ...
  writeHeader();

  stats::ELFHeaderBytes += W.OS.tell() - StartOffset;

  // ... then the sections ...
  SmallVector<std::pair<MCSectionELF *, SmallVector<unsigned>>, 0> Groups;
  // Map from group section index to group
  SmallVector<unsigned, 0> GroupMap;
  SmallVector<MCSectionELF *> Relocations;
  for (MCSection &Sec : Asm) {
    MCSectionELF &Section = static_cast<MCSectionELF &>(Sec);
    if (Mode == NonDwoOnly && isDwoSection(Section))
      continue;
    if (Mode == DwoOnly && !isDwoSection(Section))
      continue;

```
- **EN**: Implements logic around `getContext`, `getELFSection`, `addToSectionTable`, `writeHeader`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `getContext`, `getELFSection`, `addToSectionTable`, `writeHeader`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1018-1037
```cpp
    // Remember the offset into the file for this section.
    const uint64_t SecStart = align(Section.getAlign());

    const MCSymbolELF *SignatureSymbol = Section.getGroup();
    writeSectionData(Section);

    uint64_t SecEnd = W.OS.tell();
    Section.setOffsets(SecStart, SecEnd);

    MCSectionELF *RelSection = createRelocationSection(Ctx, Section);

    unsigned *GroupIdxEntry = nullptr;
    if (SignatureSymbol) {
      GroupIdxEntry = &RevGroupMap[SignatureSymbol];
      if (!*GroupIdxEntry) {
        MCSectionELF *Group =
            Ctx.createELFGroupSection(SignatureSymbol, Section.isComdat());
        *GroupIdxEntry = addToSectionTable(Group);
        Group->setAlignment(Align(4));

```
- **EN**: Implements logic around `align`, `getGroup`, `writeSectionData`, `tell`, and 5 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `align`, `getGroup`, `writeSectionData`, `tell`, and 5 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1038-1057
```cpp
        GroupMap.resize(*GroupIdxEntry + 1);
        GroupMap[*GroupIdxEntry] = Groups.size();
        Groups.emplace_back(Group, SmallVector<unsigned>{});
      }
    }

    Section.setOrdinal(addToSectionTable(&Section));
    if (RelSection) {
      RelSection->setOrdinal(addToSectionTable(RelSection));
      Relocations.push_back(RelSection);
    }

    if (GroupIdxEntry) {
      auto &Members = Groups[GroupMap[*GroupIdxEntry]];
      Members.second.push_back(Section.getOrdinal());
      if (RelSection)
        Members.second.push_back(RelSection->getOrdinal());
    }
  }

```
- **EN**: Implements logic around `resize`, `size`, `emplace_back`, `setOrdinal`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; updates MC section or symbol state.
- **CN**: 围绕 `resize`, `size`, `emplace_back`, `setOrdinal`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，更新 MC 节区或符号状态。

### Lines 1058-1080
```cpp
  for (auto &[Group, Members] : Groups) {
    // Remember the offset into the file for this section.
    const uint64_t SecStart = align(Group->getAlign());

    write(uint32_t(Group->isComdat() ? unsigned(ELF::GRP_COMDAT) : 0));
    W.write<unsigned>(Members);

    uint64_t SecEnd = W.OS.tell();
    Group->setOffsets(SecStart, SecEnd);
  }

  if (Mode == DwoOnly) {
    // dwo files don't have symbol tables or relocations, but they do have
    // string tables.
    StrTabBuilder.finalize();
  } else {
    MCSectionELF *AddrsigSection;
    if (OWriter.getEmitAddrsigSection()) {
      AddrsigSection = Ctx.getELFSection(".llvm_addrsig", ELF::SHT_LLVM_ADDRSIG,
                                         ELF::SHF_EXCLUDE);
      addToSectionTable(AddrsigSection);
    }

```
- **EN**: Implements logic around `align`, `write`, `write<unsigned>`, `tell`, and 4 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `align`, `write`, `write<unsigned>`, `tell`, and 4 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1081-1102
```cpp
    // Compute symbol table information.
    computeSymbolTable(RevGroupMap);

    for (MCSectionELF *RelSection : Relocations) {
      // Remember the offset into the file for this section.
      const uint64_t SecStart = align(RelSection->getAlign());

      writeRelocations(
          static_cast<const MCSectionELF &>(*RelSection->getLinkedToSection()));

      uint64_t SecEnd = W.OS.tell();
      RelSection->setOffsets(SecStart, SecEnd);
    }

    if (OWriter.getEmitAddrsigSection()) {
      uint64_t SecStart = W.OS.tell();
      writeAddrsigSection();
      uint64_t SecEnd = W.OS.tell();
      AddrsigSection->setOffsets(SecStart, SecEnd);
    }
  }

```
- **EN**: Implements logic around `computeSymbolTable`, `align`, `writeRelocations`, `getLinkedToSection`, and 3 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `computeSymbolTable`, `align`, `writeRelocations`, `getLinkedToSection`, and 3 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1103-1119
```cpp
  {
    uint64_t SecStart = W.OS.tell();
    StrTabBuilder.write(W.OS);
    StrtabSection->setOffsets(SecStart, W.OS.tell());
  }

  const uint64_t SectionHeaderOffset = align(is64Bit() ? Align(8) : Align(4));

  // ... then the section header table ...
  writeSectionHeaders();

  uint16_t NumSections = support::endian::byte_swap<uint16_t>(
      (SectionTable.size() + 1 >= ELF::SHN_LORESERVE) ? (uint16_t)ELF::SHN_UNDEF
                                                      : SectionTable.size() + 1,
      W.Endian);
  unsigned NumSectionsOffset;

```
- **EN**: Implements logic around `tell`, `write`, `setOffsets`, `align`, and 3 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `tell`, `write`, `setOffsets`, `align`, and 3 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1120-1136
```cpp
  auto &Stream = static_cast<raw_pwrite_stream &>(W.OS);
  if (is64Bit()) {
    uint64_t Val =
        support::endian::byte_swap<uint64_t>(SectionHeaderOffset, W.Endian);
    Stream.pwrite(reinterpret_cast<char *>(&Val), sizeof(Val),
                  offsetof(ELF::Elf64_Ehdr, e_shoff));
    NumSectionsOffset = offsetof(ELF::Elf64_Ehdr, e_shnum);
  } else {
    uint32_t Val =
        support::endian::byte_swap<uint32_t>(SectionHeaderOffset, W.Endian);
    Stream.pwrite(reinterpret_cast<char *>(&Val), sizeof(Val),
                  offsetof(ELF::Elf32_Ehdr, e_shoff));
    NumSectionsOffset = offsetof(ELF::Elf32_Ehdr, e_shnum);
  }
  Stream.pwrite(reinterpret_cast<char *>(&NumSections), sizeof(NumSections),
                NumSectionsOffset);

```
- **EN**: Implements logic around `byte_swap<uint64_t>`, `pwrite`, `offsetof`, `byte_swap<uint32_t>`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `byte_swap<uint64_t>`, `pwrite`, `offsetof`, `byte_swap<uint32_t>` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1137-1161
```cpp
  return W.OS.tell() - StartOffset;
}

ELFObjectWriter::ELFObjectWriter(std::unique_ptr<MCELFObjectTargetWriter> MOTW,
                                 raw_pwrite_stream &OS, bool IsLittleEndian)
    : TargetObjectWriter(std::move(MOTW)), OS(OS),
      IsLittleEndian(IsLittleEndian) {}
ELFObjectWriter::ELFObjectWriter(std::unique_ptr<MCELFObjectTargetWriter> MOTW,
                                 raw_pwrite_stream &OS,
                                 raw_pwrite_stream &DwoOS, bool IsLittleEndian)
    : TargetObjectWriter(std::move(MOTW)), OS(OS), DwoOS(&DwoOS),
      IsLittleEndian(IsLittleEndian) {}

void ELFObjectWriter::reset() {
  ELFHeaderEFlags = 0;
  SeenGnuAbi = false;
  OverrideABIVersion.reset();
  Relocations.clear();
  Renames.clear();
  Weakrefs.clear();
  Symvers.clear();
  SeenGnuAbi = false;
  MCObjectWriter::reset();
}

```
- **EN**: Implements logic around `tell`, `ELFObjectWriter`, `TargetObjectWriter`, `IsLittleEndian`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `tell`, `ELFObjectWriter`, `TargetObjectWriter`, `IsLittleEndian`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 1162-1179
```cpp
void ELFObjectWriter::setAssembler(MCAssembler *Asm) {
  MCObjectWriter::setAssembler(Asm);
  TargetObjectWriter->setAssembler(Asm);
}

bool ELFObjectWriter::hasRelocationAddend() const {
  return TargetObjectWriter->hasRelocationAddend();
}

void ELFObjectWriter::executePostLayoutBinding() {
  // The presence of symbol versions causes undefined symbols and
  // versions declared with @@@ to be renamed.
  for (const Symver &S : Symvers) {
    StringRef AliasName = S.Name;
    auto &Symbol = static_cast<const MCSymbolELF &>(*S.Sym);
    size_t Pos = AliasName.find('@');
    assert(Pos != StringRef::npos);

```
- **EN**: Implements logic around `setAssembler`, `hasRelocationAddend`, `executePostLayoutBinding`, `find`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `setAssembler`, `hasRelocationAddend`, `executePostLayoutBinding`, `find`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1180-1197
```cpp
    StringRef Prefix = AliasName.substr(0, Pos);
    StringRef Rest = AliasName.substr(Pos);
    StringRef Tail = Rest;
    if (Rest.starts_with("@@@"))
      Tail = Rest.substr(Symbol.isUndefined() ? 2 : 1);

    auto *Alias = static_cast<MCSymbolELF *>(
        Asm->getContext().getOrCreateSymbol(Prefix + Tail));
    Asm->registerSymbol(*Alias);
    const MCExpr *Value = MCSymbolRefExpr::create(&Symbol, Asm->getContext());
    Alias->setVariableValue(Value);

    // Aliases defined with .symvar copy the binding from the symbol they alias.
    // This is the first place we are able to copy this information.
    Alias->setBinding(Symbol.getBinding());
    Alias->setVisibility(Symbol.getVisibility());
    Alias->setOther(Symbol.getOther());

```
- **EN**: Implements logic around `substr`, `getContext`, `registerSymbol`, `create`, and 4 more symbols; this block updates MC section or symbol state.
- **CN**: 围绕 `substr`, `getContext`, `registerSymbol`, `create`, and 4 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 1198-1214
```cpp
    if (!Symbol.isUndefined() && S.KeepOriginalSym)
      continue;

    if (Symbol.isUndefined() && Rest.starts_with("@@") &&
        !Rest.starts_with("@@@")) {
      Asm->getContext().reportError(S.Loc, "default version symbol " +
                                               AliasName + " must be defined");
      continue;
    }

    if (auto It = Renames.find(&Symbol);
        It != Renames.end() && It->second != Alias) {
      Asm->getContext().reportError(S.Loc, Twine("multiple versions for ") +
                                               Symbol.getName());
      continue;
    }

```
- **EN**: Implements logic around `starts_with`, `getContext`, `end`, `getName`; this block updates MC section or symbol state.
- **CN**: 围绕 `starts_with`, `getContext`, `end`, `getName` 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 1215-1242
```cpp
    Renames.insert(std::make_pair(&Symbol, Alias));
  }

  for (const MCSymbol *&Sym : AddrsigSyms) {
    if (const MCSymbol *R =
            Renames.lookup(static_cast<const MCSymbolELF *>(Sym)))
      Sym = R;
    if (Sym->isInSection() && Sym->getName().starts_with(".L"))
      Sym = Sym->getSection().getBeginSymbol();
    Sym->setUsedInReloc();
  }

  // For each `.weakref alias, target`, if the variable `alias` is registered
  // (typically through MCObjectStreamer::visitUsedSymbol), register `target`.
  // If `target` was unregistered before (not directly referenced or defined),
  // make it weak.
  for (const MCSymbol *Alias : Weakrefs) {
    if (!Alias->isRegistered())
      continue;
    auto *Expr = Alias->getVariableValue();
    if (const auto *Inner = dyn_cast<MCSymbolRefExpr>(Expr)) {
      auto &Sym = static_cast<const MCSymbolELF &>(Inner->getSymbol());
      if (Asm->registerSymbol(Sym))
        Sym.setBinding(ELF::STB_WEAK);
    }
  }
}

```
- **EN**: Implements logic around `insert`, `lookup`, `getSection`, `setUsedInReloc`, and 3 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `insert`, `lookup`, `getSection`, `setUsedInReloc`, and 3 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1243-1268
```cpp
// It is always valid to create a relocation with a symbol. It is preferable
// to use a relocation with a section if that is possible. Using the section
// allows us to omit some local symbols from the symbol table.
bool ELFObjectWriter::useSectionSymbol(const MCValue &Val,
                                       const MCSymbolELF *Sym, uint64_t C,
                                       unsigned Type) const {
  // Keep symbol type for a local ifunc because it may result in an IRELATIVE
  // reloc that the dynamic loader will use to resolve the address at startup
  // time.
  if (Sym->getType() == ELF::STT_GNU_IFUNC)
    return false;

  // If a relocation points to a mergeable section, we have to be careful.
  // If the offset is zero, a relocation with the section will encode the
  // same information. With a non-zero offset, the situation is different.
  // For example, a relocation can point 42 bytes past the end of a string.
  // If we change such a relocation to use the section, the linker would think
  // that it pointed to another string and subtracting 42 at runtime will
  // produce the wrong value.
  if (Sym->isInSection()) {
    auto &Sec = static_cast<const MCSectionELF &>(Sym->getSection());
    unsigned Flags = Sec.getFlags();
    if (Flags & ELF::SHF_MERGE) {
      if (C != 0)
        return false;

```
- **EN**: Implements logic around `useSectionSymbol`, `getSection`, `getFlags`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `useSectionSymbol`, `getSection`, `getFlags` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1269-1286
```cpp
      // gold<2.34 incorrectly ignored the addend for R_386_GOTOFF (9)
      // (http://sourceware.org/PR16794).
      if (TargetObjectWriter->getEMachine() == ELF::EM_386 &&
          Type == ELF::R_386_GOTOFF)
        return false;

      // ld.lld handles R_MIPS_HI16/R_MIPS_LO16 separately, not as a whole, so
      // it doesn't know that an R_MIPS_HI16 with implicit addend 1 and an
      // R_MIPS_LO16 with implicit addend -32768 represents 32768, which is in
      // range of a MergeInputSection. We could introduce a new RelExpr member
      // (like R_RISCV_PC_INDIRECT for R_RISCV_PCREL_HI20 / R_RISCV_PCREL_LO12)
      // but the complexity is unnecessary given that GNU as keeps the original
      // symbol for this case as well.
      if (TargetObjectWriter->getEMachine() == ELF::EM_MIPS &&
          !hasRelocationAddend())
        return false;
    }

```
- **EN**: Implements logic around `hasRelocationAddend`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `hasRelocationAddend` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1287-1311
```cpp
    // Most TLS relocations use a got, so they need the symbol. Even those that
    // are just an offset (@tpoff), require a symbol in gold versions before
    // 5efeedf61e4fe720fd3e9a08e6c91c10abb66d42 (2014-09-26) which fixed
    // http://sourceware.org/PR16773.
    if (Flags & ELF::SHF_TLS)
      return false;
  }

  return !TargetObjectWriter->needsRelocateWithSymbol(Val, Type);
}

bool ELFObjectWriter::checkRelocation(SMLoc Loc, const MCSectionELF *From,
                                      const MCSectionELF *To) {
  if (isDwoSection(*From)) {
    getContext().reportError(Loc, "A dwo section may not contain relocations");
    return false;
  }
  if (To && isDwoSection(*To)) {
    getContext().reportError(Loc,
                             "A relocation may not refer to a dwo section");
    return false;
  }
  return true;
}

```
- **EN**: Implements logic around `needsRelocateWithSymbol`, `checkRelocation`, `getContext`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `needsRelocateWithSymbol`, `checkRelocation`, `getContext` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1312-1337
```cpp
void ELFObjectWriter::recordRelocation(const MCFragment &F,
                                       const MCFixup &Fixup, MCValue Target,
                                       uint64_t &FixedValue) {
  auto &Section = static_cast<const MCSectionELF &>(*F.getParent());
  MCContext &Ctx = getContext();

  auto *SymA = static_cast<const MCSymbolELF *>(Target.getAddSym());
  const MCSectionELF *SecA =
      (SymA && SymA->isInSection())
          ? static_cast<const MCSectionELF *>(&SymA->getSection())
          : nullptr;
  if (DwoOS && !checkRelocation(Fixup.getLoc(), &Section, SecA))
    return;

  bool IsPCRel = Fixup.isPCRel();
  uint64_t FixupOffset = Asm->getFragmentOffset(F) + Fixup.getOffset();
  uint64_t Addend = Target.getConstant();
  if (auto *RefB = Target.getSubSym()) {
    auto &SymB = static_cast<const MCSymbolELF &>(*RefB);
    if (SymB.isUndefined()) {
      Ctx.reportError(Fixup.getLoc(),
                      Twine("symbol '") + SymB.getName() +
                          "' can not be undefined in a subtraction expression");
      return;
    }

```
- **EN**: Implements logic around `recordRelocation`, `getParent`, `getContext`, `getAddSym`, and 7 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `recordRelocation`, `getParent`, `getContext`, `getAddSym`, and 7 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1338-1356
```cpp
    assert(!SymB.isAbsolute() && "Should have been folded");
    const MCSection &SecB = SymB.getSection();
    if (&SecB != &Section) {
      Ctx.reportError(Fixup.getLoc(),
                      "Cannot represent a difference across sections");
      return;
    }

    assert(!IsPCRel && "should have been folded");
    IsPCRel = true;
    Addend += FixupOffset - Asm->getSymbolOffset(SymB);
  }

  unsigned Type;
  if (mc::isRelocRelocation(Fixup.getKind()))
    Type = Fixup.getKind() - FirstLiteralRelocationKind;
  else
    Type = TargetObjectWriter->getRelocType(Fixup, Target, IsPCRel);

```
- **EN**: Implements logic around `assert`, `getSection`, `reportError`, `getSymbolOffset`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `assert`, `getSection`, `reportError`, `getSymbolOffset`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1357-1379
```cpp
  // Convert SymA to an STT_SECTION symbol if it's defined, local, and meets
  // specific conditions, unless it's a .reloc directive, which disables
  // STT_SECTION adjustment.
  const MCTargetOptions &TO = Ctx.getTargetOptions();
  bool UseSectionSym = SymA && SymA->getBinding() == ELF::STB_LOCAL &&
                       !SymA->isUndefined() &&
                       !mc::isRelocRelocation(Fixup.getKind());
  if (UseSectionSym) {
    auto RSS = TO.RelocSectionSym;
    UseSectionSym = RSS == RelocSectionSymType::All ||
                    (RSS == RelocSectionSymType::Internal &&
                     SymA->getName().starts_with(
                         Ctx.getAsmInfo().getInternalSymbolPrefix()));
  }
  if (UseSectionSym && useSectionSymbol(Target, SymA, Addend, Type)) {
    Addend += Asm->getSymbolOffset(*SymA);
    SymA = static_cast<const MCSymbolELF *>(SecA->getBeginSymbol());
  } else if (const MCSymbolELF *R = Renames.lookup(SymA)) {
    SymA = R;
  }
  if (SymA)
    SymA->setUsedInReloc();

```
- **EN**: Implements logic around `getTargetOptions`, `getBinding`, `isUndefined`, `isRelocRelocation`, and 5 more symbols; this block handles relocation, fixup, or symbol-resolution work; parses assembly syntax or operands; updates MC section or symbol state.
- **CN**: 围绕 `getTargetOptions`, `getBinding`, `isUndefined`, `isRelocRelocation`, and 5 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，解析汇编语法或操作数，更新 MC 节区或符号状态。

### Lines 1380-1402
```cpp
  FixedValue = usesRela(TO, Section) ? 0 : Addend;
  Relocations[&Section].emplace_back(FixupOffset, SymA, Type, Addend);
}

bool ELFObjectWriter::usesRela(const MCTargetOptions &TO,
                               const MCSectionELF &Sec) const {
  return (hasRelocationAddend() &&
          Sec.getType() != ELF::SHT_LLVM_CALL_GRAPH_PROFILE) ||
         TO.Crel;
}

bool ELFObjectWriter::isSymbolRefDifferenceFullyResolvedImpl(
    const MCSymbol &SA, const MCFragment &FB, bool InSet, bool IsPCRel) const {
  auto &SymA = static_cast<const MCSymbolELF &>(SA);
  if (IsPCRel) {
    assert(!InSet);
    if (SymA.getBinding() != ELF::STB_LOCAL ||
        SymA.getType() == ELF::STT_GNU_IFUNC)
      return false;
  }
  return &SymA.getSection() == FB.getParent();
}

```
- **EN**: Implements logic around `usesRela`, `emplace_back`, `getType`, `isSymbolRefDifferenceFullyResolvedImpl`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `usesRela`, `emplace_back`, `getType`, `isSymbolRefDifferenceFullyResolvedImpl`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1403-1412
```cpp
uint64_t ELFObjectWriter::writeObject() {
  uint64_t Size =
      ELFWriter(*Asm, *this, OS, IsLittleEndian,
                DwoOS ? ELFWriter::NonDwoOnly : ELFWriter::AllSections)
          .writeObject();
  if (DwoOS)
    Size += ELFWriter(*Asm, *this, *DwoOS, IsLittleEndian, ELFWriter::DwoOnly)
                .writeObject();
  return Size;
}
```
- **EN**: Implements logic around `writeObject`, `ELFWriter`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `writeObject`, `ELFWriter` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

## Key Concepts / 关键概念

- **Relocations and fixups / 重定位与 fixup**:
  - **EN**: Bridges symbolic references to concrete addresses or relocation records
  - **CN**: 把符号引用连接到具体地址或重定位记录
- **Directive handling / 伪指令处理**:
  - **EN**: Interprets assembler directives and maps them to streamer operations
  - **CN**: 解释汇编伪指令并把它们映射为 streamer 操作
- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据
- **Object layout / 目标文件布局**:
  - **EN**: Organizes fragments into sections and computes final offsets
  - **CN**: 把片段组织到节区中并计算最终偏移

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ADT/ArrayRef.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h`, `llvm/BinaryFormat/ELF.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCAssembler.h` ... (+30 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support, BinaryFormat, Target/TargetParser
