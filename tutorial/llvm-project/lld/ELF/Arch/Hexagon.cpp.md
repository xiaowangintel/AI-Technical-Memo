# Hexagon.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `lld/ELF/Arch/Hexagon.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: Hexagon.cpp. It also sits in code that implements ELF linking components in LLD. / 该文件实现 LLD 中的 ELF 链接组件。 源码头部说明其职责是：Hexagon.cpp。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
//===-- Hexagon.cpp -------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 9-25

```cpp
#include "InputFiles.h"
#include "OutputSections.h"
#include "RelocScan.h"
#include "SymbolTable.h"
#include "Symbols.h"
#include "SyntheticSections.h"
#include "Target.h"
#include "Thunks.h"
#include "lld/Common/ErrorHandler.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/Support/ELFAttributes.h"
#include "llvm/Support/Endian.h"
#include "llvm/Support/HexagonAttributeParser.h"
#include "llvm/Support/HexagonAttributes.h"
#include "llvm/Support/LEB128.h"
```

- EN: Pulls in 16 header(s) from local project, LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 16 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 26-42

```cpp
using namespace llvm;
using namespace llvm::object;
using namespace llvm::support::endian;
using namespace llvm::ELF;
using namespace lld;
using namespace lld::elf;

namespace {
class Hexagon final : public TargetInfo {
public:
  Hexagon(Ctx &);
  uint32_t calcEFlags() const override;
  RelExpr getRelExpr(RelType type, const Symbol &s,
                     const uint8_t *loc) const override;
  RelType getDynRel(RelType type) const override;
  int64_t getImplicitAddend(const uint8_t *buf, RelType type) const override;
  template <class ELFT, class RelTy>
```

- EN: Works inside namespace scope `llvm`, `lld` to organize symbols. Introduces type definitions such as `Hexagon`, `ELFT`, `RelTy`. Declares or implements routines including `Hexagon`, `calcEFlags`, `getDynRel`, `getImplicitAddend`. Notable symbols here include `Hexagon`, `ELFT`, `RelTy`, `calcEFlags`, `getDynRel`, `getImplicitAddend`.
- CN: 这里位于命名空间 `llvm`, `lld` 中，用于组织符号作用域。这里引入类型定义，例如 `Hexagon`, `ELFT`, `RelTy`。这里声明或实现函数，例如 `Hexagon`, `calcEFlags`, `getDynRel`, `getImplicitAddend`。这里较值得关注的符号包括 `Hexagon`, `ELFT`, `RelTy`, `calcEFlags`, `getDynRel`, `getImplicitAddend`。

### Lines 43-60

```cpp
  void scanSectionImpl(InputSectionBase &sec, Relocs<RelTy> rels);
  void scanSection(InputSectionBase &sec) override {
    elf::scanSection1<Hexagon, ELF32LE>(*this, sec);
  }
  void finalizeRelocScan() override;
  bool needsThunk(RelExpr expr, RelType type, const InputFile *file,
                  uint64_t branchAddr, const Symbol &s,
                  int64_t a) const override;
  uint32_t getThunkSectionSpacing() const override;
  bool inBranchRange(RelType type, uint64_t src, uint64_t dst) const override;
  void relocate(uint8_t *loc, const Relocation &rel,
                uint64_t val) const override;
  void writePltHeader(uint8_t *buf) const override;
  void writePlt(uint8_t *buf, const Symbol &sym,
                uint64_t pltEntryAddr) const override;
};
} // namespace
```

- EN: Declares or implements routines including `scanSectionImpl`, `scanSection`, `finalizeRelocScan`, `getThunkSectionSpacing`, `inBranchRange`, and 1 more. Notable symbols here include `scanSectionImpl`, `scanSection`, `finalizeRelocScan`, `getThunkSectionSpacing`, `inBranchRange`, `writePltHeader`.
- CN: 这里声明或实现函数，例如 `scanSectionImpl`, `scanSection`, `finalizeRelocScan`, `getThunkSectionSpacing`, `inBranchRange`, and 1 more。这里较值得关注的符号包括 `scanSectionImpl`, `scanSection`, `finalizeRelocScan`, `getThunkSectionSpacing`, `inBranchRange`, `writePltHeader`。

### Lines 61-71

```cpp
Hexagon::Hexagon(Ctx &ctx) : TargetInfo(ctx) {
  pltRel = R_HEX_JMP_SLOT;
  relativeRel = R_HEX_RELATIVE;
  gotRel = R_HEX_GLOB_DAT;
  symbolicRel = R_HEX_32;

  gotBaseSymInGotPlt = true;
  // The zero'th GOT entry is reserved for the address of _DYNAMIC.  The
  // next 3 are reserved for the dynamic loader.
  gotPltHeaderEntriesNum = 4;
```

- EN: Declares or implements routines including `Hexagon`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `Hexagon`.
- CN: 这里声明或实现函数，例如 `Hexagon`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `Hexagon`。

### Lines 72-80

```cpp
  pltEntrySize = 16;
  pltHeaderSize = 32;

  // Hexagon Linux uses 64K pages by default.
  defaultMaxPageSize = 0x10000;
  tlsGotRel = R_HEX_TPREL_32;
  tlsModuleIndexRel = R_HEX_DTPMOD_32;
  tlsOffsetRel = R_HEX_DTPREL_32;
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 81-95

```cpp
  needsThunks = true;
}

uint32_t Hexagon::calcEFlags() const {
  // The architecture revision must always be equal to or greater than
  // greatest revision in the list of inputs.
  std::optional<uint32_t> ret;
  for (InputFile *f : ctx.objectFiles) {
    uint32_t eflags = cast<ObjFile<ELF32LE>>(f)->getObj().getHeader().e_flags;
    if (!ret || eflags > *ret)
      ret = eflags;
  }
  return ret.value_or(/* Default Arch Rev: */ EF_HEXAGON_MACH_V68);
}
```

- EN: Declares or implements routines including `calcEFlags`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `calcEFlags`.
- CN: 这里声明或实现函数，例如 `calcEFlags`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `calcEFlags`。

### Lines 96-110

```cpp
static uint32_t applyMask(uint32_t mask, uint32_t data) {
  uint32_t result = 0;
  size_t off = 0;

  for (size_t bit = 0; bit != 32; ++bit) {
    uint32_t valBit = (data >> off) & 1;
    uint32_t maskBit = (mask >> bit) & 1;
    if (maskBit) {
      result |= (valBit << bit);
      ++off;
    }
  }
  return result;
}
```

- EN: Declares or implements routines including `applyMask`. Notable symbols here include `applyMask`.
- CN: 这里声明或实现函数，例如 `applyMask`。这里较值得关注的符号包括 `applyMask`。

### Lines 111-128

```cpp
// Only needed to support relocations used by relocateNonAlloc and
// preprocessRelocs.
RelExpr Hexagon::getRelExpr(RelType type, const Symbol &s,
                            const uint8_t *loc) const {
  switch (type) {
  case R_HEX_NONE:
    return R_NONE;
  case R_HEX_32:
    return R_ABS;
  case R_HEX_32_PCREL:
    return R_PC;
  default:
    Err(ctx) << getErrorLoc(ctx, loc) << "unknown relocation (" << type.v
             << ") against symbol " << &s;
    return R_NONE;
  }
}
```

- EN: Declares or implements routines including `Err`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `Err`.
- CN: 这里声明或实现函数，例如 `Err`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `Err`。

### Lines 129-146

```cpp
template <class ELFT, class RelTy>
void Hexagon::scanSectionImpl(InputSectionBase &sec, Relocs<RelTy> rels) {
  RelocScan rs(ctx, &sec);
  sec.relocations.reserve(rels.size());
  for (auto it = rels.begin(); it != rels.end(); ++it) {
    const RelTy &rel = *it;
    uint32_t symIdx = rel.getSymbol(false);
    Symbol &sym = sec.getFile<ELFT>()->getSymbol(symIdx);
    uint64_t offset = rel.r_offset;
    RelType type = rel.getType(false);
    if (sym.isUndefined() && symIdx != 0 &&
        rs.maybeReportUndefined(cast<Undefined>(sym), offset))
      continue;
    int64_t addend = rs.getAddend<ELFT>(rel, type);
    RelExpr expr;
    // Relocation types that only need a RelExpr set `expr` and break out of
    // the switch to reach rs.process(). Types that need special handling
    // (fast-path helpers, TLS) call a handler and use `continue`.
```

- EN: Introduces type definitions such as `ELFT`, `RelTy`. Declares or implements routines including `scanSectionImpl`, `rs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `ELFT`, `RelTy`, `scanSectionImpl`, `rs`.
- CN: 这里引入类型定义，例如 `ELFT`, `RelTy`。这里声明或实现函数，例如 `scanSectionImpl`, `rs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `ELFT`, `RelTy`, `scanSectionImpl`, `rs`。

### Lines 147-164

```cpp
    switch (type) {
    case R_HEX_NONE:
      continue;

    // Absolute relocations:
    case R_HEX_6_X:
    case R_HEX_8_X:
    case R_HEX_9_X:
    case R_HEX_10_X:
    case R_HEX_11_X:
    case R_HEX_12_X:
    case R_HEX_16_X:
    case R_HEX_32:
    case R_HEX_32_6_X:
    case R_HEX_HI16:
    case R_HEX_LO16:
    case R_HEX_DTPREL_32:
      expr = R_ABS;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 165-175

```cpp
      break;

    // PC-relative relocations:
    case R_HEX_B9_PCREL:
    case R_HEX_B13_PCREL:
    case R_HEX_B15_PCREL:
    case R_HEX_6_PCREL_X:
    case R_HEX_32_PCREL:
      rs.processR_PC(type, offset, addend, sym);
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 176-193

```cpp
    // PLT-generating relocations:
    case R_HEX_B9_PCREL_X:
    case R_HEX_B15_PCREL_X:
    case R_HEX_B22_PCREL:
    case R_HEX_PLT_B22_PCREL:
    case R_HEX_B22_PCREL_X:
    case R_HEX_B32_PCREL_X:
      rs.processR_PLT_PC(type, offset, addend, sym);
      continue;
    case R_HEX_GD_PLT_B22_PCREL:
    case R_HEX_GD_PLT_B22_PCREL_X:
    case R_HEX_GD_PLT_B32_PCREL_X:
      // GD PLT: call foo@GDPLT becomes call __tls_get_addr.
      // Record R_PLT_PC on the TLS symbol; finalizeRelocScan (called
      // single-threaded after scanning) will create __tls_get_addr and
      // rebind these relocations.  We cannot access the symbol table here
      // because scanSectionImpl runs in parallel.
      sec.addReloc({R_PLT_PC, type, offset, addend, &sym});
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 194-203

```cpp
      continue;

    // GOT-generating relocations:
    case R_HEX_GOT_11_X:
    case R_HEX_GOT_16_X:
    case R_HEX_GOT_32_6_X:
      ctx.in.gotPlt->hasGotPltOffRel.store(true, std::memory_order_relaxed);
      expr = R_GOTPLT;
      break;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 204-213

```cpp
    // GOTREL relocations:
    case R_HEX_GOTREL_11_X:
    case R_HEX_GOTREL_16_X:
    case R_HEX_GOTREL_32_6_X:
    case R_HEX_GOTREL_HI16:
    case R_HEX_GOTREL_LO16:
      ctx.in.gotPlt->hasGotPltOffRel.store(true, std::memory_order_relaxed);
      expr = R_GOTPLTREL;
      break;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 214-231

```cpp
    // TLS relocations:
    case R_HEX_TPREL_11_X:
    case R_HEX_TPREL_16:
    case R_HEX_TPREL_16_X:
    case R_HEX_TPREL_32_6_X:
    case R_HEX_TPREL_HI16:
    case R_HEX_TPREL_LO16:
      if (rs.checkTlsLe(offset, sym, type))
        continue;
      expr = R_TPREL;
      break;
    case R_HEX_IE_32_6_X:
    case R_HEX_IE_16_X:
    case R_HEX_IE_HI16:
    case R_HEX_IE_LO16:
      // There is no IE to LE optimization.
      rs.handleTlsIe<false>(R_GOT, type, offset, addend, sym);
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 232-247

```cpp
    case R_HEX_IE_GOT_11_X:
    case R_HEX_IE_GOT_16_X:
    case R_HEX_IE_GOT_32_6_X:
    case R_HEX_IE_GOT_HI16:
    case R_HEX_IE_GOT_LO16:
      ctx.in.gotPlt->hasGotPltOffRel.store(true, std::memory_order_relaxed);
      rs.handleTlsIe<false>(R_GOTPLT, type, offset, addend, sym);
      continue;
    case R_HEX_GD_GOT_11_X:
    case R_HEX_GD_GOT_16_X:
    case R_HEX_GD_GOT_32_6_X:
      sym.setFlags(NEEDS_TLSGD);
      ctx.in.gotPlt->hasGotPltOffRel.store(true, std::memory_order_relaxed);
      sec.addReloc({R_TLSGD_GOTPLT, type, offset, addend, &sym});
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 248-257

```cpp
    default:
      Err(ctx) << getErrorLoc(ctx, sec.content().data() + offset)
               << "unknown relocation (" << type.v << ") against symbol "
               << &sym;
      continue;
    }
    rs.process(expr, type, offset, sym, addend);
  }
}
```

- EN: Declares or implements routines including `Err`, `relocation`. Notable symbols here include `Err`, `relocation`.
- CN: 这里声明或实现函数，例如 `Err`, `relocation`。这里较值得关注的符号包括 `Err`, `relocation`。

### Lines 258-275

```cpp
// There are (arguably too) many relocation masks for the DSP's
// R_HEX_6_X type.  The table below is used to select the correct mask
// for the given instruction.
struct InstructionMask {
  uint32_t cmpMask;
  uint32_t relocMask;
};
static const InstructionMask r6[] = {
    {0x38000000, 0x0000201f}, {0x39000000, 0x0000201f},
    {0x3e000000, 0x00001f80}, {0x3f000000, 0x00001f80},
    {0x40000000, 0x000020f8}, {0x41000000, 0x000007e0},
    {0x42000000, 0x000020f8}, {0x43000000, 0x000007e0},
    {0x44000000, 0x000020f8}, {0x45000000, 0x000007e0},
    {0x46000000, 0x000020f8}, {0x47000000, 0x000007e0},
    {0x6a000000, 0x00001f80}, {0x7c000000, 0x001f2000},
    {0x9a000000, 0x00000f60}, {0x9b000000, 0x00000f60},
    {0x9c000000, 0x00000f60}, {0x9d000000, 0x00000f60},
    {0x9f000000, 0x001f0100}, {0xab000000, 0x0000003f},
```

- EN: Introduces type definitions such as `InstructionMask`. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `InstructionMask`.
- CN: 这里引入类型定义，例如 `InstructionMask`。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `InstructionMask`。

### Lines 276-288

```cpp
    {0xad000000, 0x0000003f}, {0xaf000000, 0x00030078},
    {0xd7000000, 0x006020e0}, {0xd8000000, 0x006020e0},
    {0xdb000000, 0x006020e0}, {0xdf000000, 0x006020e0}};

constexpr uint32_t instParsePacketEnd = 0x0000c000;

static bool isDuplex(uint32_t insn) {
  // Duplex forms have a fixed mask and parse bits 15:14 are always
  // zero.  Non-duplex insns will always have at least one bit set in the
  // parse field.
  return (instParsePacketEnd & insn) == 0;
}
```

- EN: Declares or implements routines including `isDuplex`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isDuplex`.
- CN: 这里声明或实现函数，例如 `isDuplex`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isDuplex`。

### Lines 289-296

```cpp
static uint32_t findMaskR6(Ctx &ctx, uint32_t insn) {
  if (isDuplex(insn))
    return 0x03f00000;

  for (InstructionMask i : r6)
    if ((0xff000000 & insn) == i.cmpMask)
      return i.relocMask;
```

- EN: Declares or implements routines including `findMaskR6`. Notable symbols here include `findMaskR6`.
- CN: 这里声明或实现函数，例如 `findMaskR6`。这里较值得关注的符号包括 `findMaskR6`。

### Lines 297-311

```cpp
  Err(ctx) << "unrecognized instruction for 6_X relocation: 0x"
           << utohexstr(insn, true);
  return 0;
}

static uint32_t findMaskR8(uint32_t insn) {
  if (isDuplex(insn))
    return 0x03f00000;
  if ((0xff000000 & insn) == 0xde000000)
    return 0x00e020e8;
  if ((0xff000000 & insn) == 0x3c000000)
    return 0x0000207f;
  return 0x00001fe0;
}
```

- EN: Declares or implements routines including `Err`, `utohexstr`, `findMaskR8`. Notable symbols here include `Err`, `utohexstr`, `findMaskR8`.
- CN: 这里声明或实现函数，例如 `Err`, `utohexstr`, `findMaskR8`。这里较值得关注的符号包括 `Err`, `utohexstr`, `findMaskR8`。

### Lines 312-319

```cpp
static uint32_t findMaskR11(uint32_t insn) {
  if (isDuplex(insn))
    return 0x03f00000;
  if ((0xff000000 & insn) == 0xa1000000)
    return 0x060020ff;
  return 0x06003fe0;
}
```

- EN: Declares or implements routines including `findMaskR11`. Notable symbols here include `findMaskR11`.
- CN: 这里声明或实现函数，例如 `findMaskR11`。这里较值得关注的符号包括 `findMaskR11`。

### Lines 320-335

```cpp
static uint32_t findMaskR16(Ctx &ctx, uint32_t insn) {
  if (isDuplex(insn))
    return 0x03f00000;

  // Clear the end-packet-parse bits:
  insn = insn & ~instParsePacketEnd;

  if ((0xff000000 & insn) == 0x48000000)
    return 0x061f20ff;
  if ((0xff000000 & insn) == 0x49000000)
    return 0x061f3fe0;
  if ((0xff000000 & insn) == 0x78000000)
    return 0x00df3fe0;
  if ((0xff000000 & insn) == 0xb0000000)
    return 0x0fe03fe0;
```

- EN: Declares or implements routines including `findMaskR16`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `findMaskR16`.
- CN: 这里声明或实现函数，例如 `findMaskR16`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `findMaskR16`。

### Lines 336-344

```cpp
  if ((0xff802000 & insn) == 0x74000000)
    return 0x00001fe0;
  if ((0xff802000 & insn) == 0x74002000)
    return 0x00001fe0;
  if ((0xff802000 & insn) == 0x74800000)
    return 0x00001fe0;
  if ((0xff802000 & insn) == 0x74802000)
    return 0x00001fe0;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 345-352

```cpp
  for (InstructionMask i : r6)
    if ((0xff000000 & insn) == i.cmpMask)
      return i.relocMask;

  Err(ctx) << "unrecognized instruction for 16_X type: 0x" << utohexstr(insn);
  return 0;
}
```

- EN: Declares or implements routines including `Err`. Notable symbols here include `Err`.
- CN: 这里声明或实现函数，例如 `Err`。这里较值得关注的符号包括 `Err`。

### Lines 353-370

```cpp
static void or32le(uint8_t *p, int32_t v) { write32le(p, read32le(p) | v); }

bool Hexagon::inBranchRange(RelType type, uint64_t src, uint64_t dst) const {
  int64_t offset = dst - src;
  switch (type) {
  case llvm::ELF::R_HEX_B22_PCREL:
  case llvm::ELF::R_HEX_PLT_B22_PCREL:
  case llvm::ELF::R_HEX_GD_PLT_B22_PCREL:
  case llvm::ELF::R_HEX_LD_PLT_B22_PCREL:
    return llvm::isInt<22>(offset >> 2);
  case llvm::ELF::R_HEX_B15_PCREL:
    return llvm::isInt<15>(offset >> 2);
    break;
  case llvm::ELF::R_HEX_B13_PCREL:
    return llvm::isInt<13>(offset >> 2);
    break;
  case llvm::ELF::R_HEX_B9_PCREL:
    return llvm::isInt<9>(offset >> 2);
```

- EN: Declares or implements routines including `or32le`, `inBranchRange`. Notable symbols here include `or32le`, `inBranchRange`.
- CN: 这里声明或实现函数，例如 `or32le`, `inBranchRange`。这里较值得关注的符号包括 `or32le`, `inBranchRange`。

### Lines 371-388

```cpp
  default:
    return true;
  }
  llvm_unreachable("unsupported relocation");
}

bool Hexagon::needsThunk(RelExpr expr, RelType type, const InputFile *file,
                         uint64_t branchAddr, const Symbol &s,
                         int64_t a) const {
  // Undefined weak symbols without PLT entries resolve to address zero.
  // Thunks are not needed since the branch target is fixed.
  if (s.isUndefined() && !s.isInPlt(ctx))
    return false;
  switch (type) {
  case R_HEX_B22_PCREL:
  case R_HEX_PLT_B22_PCREL:
  case R_HEX_GD_PLT_B22_PCREL:
  case R_HEX_LD_PLT_B22_PCREL:
```

- EN: Declares or implements routines including `llvm_unreachable`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `llvm_unreachable`。

### Lines 389-399

```cpp
  case R_HEX_B15_PCREL:
  case R_HEX_B13_PCREL:
  case R_HEX_B9_PCREL: {
    uint64_t dst = expr == R_PLT_PC ? s.getPltVA(ctx) : s.getVA(ctx, a);
    return !ctx.target->inBranchRange(type, branchAddr, dst);
  }
  default:
    return false;
  }
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 400-417

```cpp
uint32_t Hexagon::getThunkSectionSpacing() const {
  // B22_PCREL has a range of +/- 8 MiB (22-bit signed offset * 4).
  // Pre-create ThunkSections at intervals below this to leave room for
  // thunk growth.
  return 0x800000 - 0x30000;
}

void Hexagon::relocate(uint8_t *loc, const Relocation &rel,
                       uint64_t val) const {
  switch (rel.type) {
  case R_HEX_NONE:
    break;
  case R_HEX_6_PCREL_X:
  case R_HEX_6_X:
    or32le(loc, applyMask(findMaskR6(ctx, read32le(loc)), val));
    break;
  case R_HEX_8_X:
    or32le(loc, applyMask(findMaskR8(read32le(loc)), val));
```

- EN: Declares or implements routines including `getThunkSectionSpacing`, `or32le`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getThunkSectionSpacing`, `or32le`.
- CN: 这里声明或实现函数，例如 `getThunkSectionSpacing`, `or32le`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getThunkSectionSpacing`, `or32le`。

### Lines 418-435

```cpp
    break;
  case R_HEX_9_X:
    or32le(loc, applyMask(0x00003fe0, val & 0x3f));
    break;
  case R_HEX_10_X:
    or32le(loc, applyMask(0x00203fe0, val & 0x3f));
    break;
  case R_HEX_11_X:
  case R_HEX_GD_GOT_11_X:
  case R_HEX_IE_GOT_11_X:
  case R_HEX_GOT_11_X:
  case R_HEX_GOTREL_11_X:
  case R_HEX_TPREL_11_X:
    or32le(loc, applyMask(findMaskR11(read32le(loc)), val & 0x3f));
    break;
  case R_HEX_12_X:
    or32le(loc, applyMask(0x000007e0, val));
    break;
```

- EN: Declares or implements routines including `or32le`. Notable symbols here include `or32le`.
- CN: 这里声明或实现函数，例如 `or32le`。这里较值得关注的符号包括 `or32le`。

### Lines 436-453

```cpp
  case R_HEX_16_X: // These relocs only have 6 effective bits.
  case R_HEX_IE_16_X:
  case R_HEX_IE_GOT_16_X:
  case R_HEX_GD_GOT_16_X:
  case R_HEX_GOT_16_X:
  case R_HEX_GOTREL_16_X:
  case R_HEX_TPREL_16_X:
    or32le(loc, applyMask(findMaskR16(ctx, read32le(loc)), val & 0x3f));
    break;
  case R_HEX_TPREL_16:
    or32le(loc, applyMask(findMaskR16(ctx, read32le(loc)), val & 0xffff));
    break;
  case R_HEX_32:
  case R_HEX_32_PCREL:
  case R_HEX_DTPREL_32:
    or32le(loc, val);
    break;
  case R_HEX_32_6_X:
```

- EN: Declares or implements routines including `or32le`. Notable symbols here include `or32le`.
- CN: 这里声明或实现函数，例如 `or32le`。这里较值得关注的符号包括 `or32le`。

### Lines 454-471

```cpp
  case R_HEX_GD_GOT_32_6_X:
  case R_HEX_GOT_32_6_X:
  case R_HEX_GOTREL_32_6_X:
  case R_HEX_IE_GOT_32_6_X:
  case R_HEX_IE_32_6_X:
  case R_HEX_TPREL_32_6_X:
    or32le(loc, applyMask(0x0fff3fff, val >> 6));
    break;
  case R_HEX_B9_PCREL:
    checkInt(ctx, loc, val, 11, rel);
    or32le(loc, applyMask(0x003000fe, val >> 2));
    break;
  case R_HEX_B9_PCREL_X:
    or32le(loc, applyMask(0x003000fe, val & 0x3f));
    break;
  case R_HEX_B13_PCREL:
    checkInt(ctx, loc, val, 15, rel);
    or32le(loc, applyMask(0x00202ffe, val >> 2));
```

- EN: Declares or implements routines including `or32le`, `checkInt`. Notable symbols here include `or32le`, `checkInt`.
- CN: 这里声明或实现函数，例如 `or32le`, `checkInt`。这里较值得关注的符号包括 `or32le`, `checkInt`。

### Lines 472-489

```cpp
    break;
  case R_HEX_B15_PCREL:
    checkInt(ctx, loc, val, 17, rel);
    or32le(loc, applyMask(0x00df20fe, val >> 2));
    break;
  case R_HEX_B15_PCREL_X:
    or32le(loc, applyMask(0x00df20fe, val & 0x3f));
    break;
  case R_HEX_B22_PCREL:
  case R_HEX_GD_PLT_B22_PCREL:
  case R_HEX_PLT_B22_PCREL:
    checkInt(ctx, loc, val, 24, rel);
    or32le(loc, applyMask(0x1ff3ffe, val >> 2));
    break;
  case R_HEX_B22_PCREL_X:
  case R_HEX_GD_PLT_B22_PCREL_X:
    or32le(loc, applyMask(0x1ff3ffe, val & 0x3f));
    break;
```

- EN: Declares or implements routines including `checkInt`, `or32le`. Notable symbols here include `checkInt`, `or32le`.
- CN: 这里声明或实现函数，例如 `checkInt`, `or32le`。这里较值得关注的符号包括 `checkInt`, `or32le`。

### Lines 490-507

```cpp
  case R_HEX_B32_PCREL_X:
  case R_HEX_GD_PLT_B32_PCREL_X:
    or32le(loc, applyMask(0x0fff3fff, val >> 6));
    break;
  case R_HEX_GOTREL_HI16:
  case R_HEX_HI16:
  case R_HEX_IE_GOT_HI16:
  case R_HEX_IE_HI16:
  case R_HEX_TPREL_HI16:
    or32le(loc, applyMask(0x00c03fff, val >> 16));
    break;
  case R_HEX_GOTREL_LO16:
  case R_HEX_LO16:
  case R_HEX_IE_GOT_LO16:
  case R_HEX_IE_LO16:
  case R_HEX_TPREL_LO16:
    or32le(loc, applyMask(0x00c03fff, val));
    break;
```

- EN: Declares or implements routines including `or32le`. Notable symbols here include `or32le`.
- CN: 这里声明或实现函数，例如 `or32le`。这里较值得关注的符号包括 `or32le`。

### Lines 508-525

```cpp
  default:
    llvm_unreachable("unknown relocation");
  }
}

void Hexagon::writePltHeader(uint8_t *buf) const {
  const uint8_t pltData[] = {
      0x00, 0x40, 0x00, 0x00, // { immext (#0)
      0x1c, 0xc0, 0x49, 0x6a, //   r28 = add (pc, ##GOT0@PCREL) } # @GOT0
      0x0e, 0x42, 0x9c, 0xe2, // { r14 -= add (r28, #16)  # offset of GOTn
      0x4f, 0x40, 0x9c, 0x91, //   r15 = memw (r28 + #8)  # object ID at GOT2
      0x3c, 0xc0, 0x9c, 0x91, //   r28 = memw (r28 + #4) }# dynamic link at GOT1
      0x0e, 0x42, 0x0e, 0x8c, // { r14 = asr (r14, #2)    # index of PLTn
      0x00, 0xc0, 0x9c, 0x52, //   jumpr r28 }            # call dynamic linker
      0x0c, 0xdb, 0x00, 0x54, // trap0(#0xdb) # bring plt0 into 16byte alignment
  };
  memcpy(buf, pltData, sizeof(pltData));
```

- EN: Declares or implements routines including `llvm_unreachable`, `writePltHeader`, `immext`, `add`, `memw`, and 3 more. Notable symbols here include `llvm_unreachable`, `writePltHeader`, `immext`, `add`, `memw`, `asr`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`, `writePltHeader`, `immext`, `add`, `memw`, and 3 more。这里较值得关注的符号包括 `llvm_unreachable`, `writePltHeader`, `immext`, `add`, `memw`, `asr`。

### Lines 526-541

```cpp
  // Offset from PLT0 to the GOT.
  uint64_t off = ctx.in.gotPlt->getVA() - ctx.in.plt->getVA();
  relocateNoSym(buf, R_HEX_B32_PCREL_X, off);
  relocateNoSym(buf + 4, R_HEX_6_PCREL_X, off);
}

void Hexagon::writePlt(uint8_t *buf, const Symbol &sym,
                       uint64_t pltEntryAddr) const {
  const uint8_t inst[] = {
      0x00, 0x40, 0x00, 0x00, // { immext (#0)
      0x0e, 0xc0, 0x49, 0x6a, //   r14 = add (pc, ##GOTn@PCREL) }
      0x1c, 0xc0, 0x8e, 0x91, // r28 = memw (r14)
      0x00, 0xc0, 0x9c, 0x52, // jumpr r28
  };
  memcpy(buf, inst, sizeof(inst));
```

- EN: Declares or implements routines including `getVA`, `relocateNoSym`, `immext`, `add`, `memw`, and 1 more. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getVA`, `relocateNoSym`, `immext`, `add`, `memw`, `memcpy`.
- CN: 这里声明或实现函数，例如 `getVA`, `relocateNoSym`, `immext`, `add`, `memw`, and 1 more。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getVA`, `relocateNoSym`, `immext`, `add`, `memw`, `memcpy`。

### Lines 542-552

```cpp
  uint64_t gotPltEntryAddr = sym.getGotPltVA(ctx);
  relocateNoSym(buf, R_HEX_B32_PCREL_X, gotPltEntryAddr - pltEntryAddr);
  relocateNoSym(buf + 4, R_HEX_6_PCREL_X, gotPltEntryAddr - pltEntryAddr);
}

RelType Hexagon::getDynRel(RelType type) const {
  if (type == R_HEX_32)
    return type;
  return R_HEX_NONE;
}
```

- EN: Declares or implements routines including `relocateNoSym`, `getDynRel`. Notable symbols here include `relocateNoSym`, `getDynRel`.
- CN: 这里声明或实现函数，例如 `relocateNoSym`, `getDynRel`。这里较值得关注的符号包括 `relocateNoSym`, `getDynRel`。

### Lines 553-570

```cpp
int64_t Hexagon::getImplicitAddend(const uint8_t *buf, RelType type) const {
  switch (type) {
  case R_HEX_NONE:
  case R_HEX_GLOB_DAT:
  case R_HEX_JMP_SLOT:
    return 0;
  case R_HEX_32:
  case R_HEX_RELATIVE:
  case R_HEX_DTPMOD_32:
  case R_HEX_DTPREL_32:
  case R_HEX_TPREL_32:
    return SignExtend64<32>(read32(ctx, buf));
  default:
    InternalErr(ctx, buf) << "cannot read addend for relocation " << type;
    return 0;
  }
}
```

- EN: Declares or implements routines including `getImplicitAddend`, `InternalErr`. Notable symbols here include `getImplicitAddend`, `InternalErr`.
- CN: 这里声明或实现函数，例如 `getImplicitAddend`, `InternalErr`。这里较值得关注的符号包括 `getImplicitAddend`, `InternalErr`。

### Lines 571-580

```cpp
namespace {
class HexagonAttributesSection final : public SyntheticSection {
public:
  HexagonAttributesSection(Ctx &ctx)
      : SyntheticSection(ctx, ".hexagon.attributes", SHT_HEXAGON_ATTRIBUTES, 0,
                         1) {}

  size_t getSize() const override { return size; }
  void writeTo(uint8_t *buf) override;
```

- EN: Introduces type definitions such as `HexagonAttributesSection`. Declares or implements routines including `HexagonAttributesSection`, `getSize`, `writeTo`. Notable symbols here include `HexagonAttributesSection`, `getSize`, `writeTo`.
- CN: 这里引入类型定义，例如 `HexagonAttributesSection`。这里声明或实现函数，例如 `HexagonAttributesSection`, `getSize`, `writeTo`。这里较值得关注的符号包括 `HexagonAttributesSection`, `getSize`, `writeTo`。

### Lines 581-593

```cpp
  static constexpr StringRef vendor = "hexagon";
  DenseMap<unsigned, unsigned> intAttr;
  size_t size = 0;
};
} // namespace

static HexagonAttributesSection *
mergeAttributesSection(Ctx &ctx,
                       const SmallVector<InputSectionBase *, 0> &sections) {
  ctx.in.hexagonAttributes = std::make_unique<HexagonAttributesSection>(ctx);
  auto &merged =
      static_cast<HexagonAttributesSection &>(*ctx.in.hexagonAttributes);
```

- EN: Works inside namespace scope `static` to organize symbols. Notable symbols here include `static`.
- CN: 这里位于命名空间 `static` 中，用于组织符号作用域。这里较值得关注的符号包括 `static`。

### Lines 594-611

```cpp
  // Collect all tags values from attributes section.
  const auto &attributesTags = HexagonAttrs::getHexagonAttributeTags();
  for (const InputSectionBase *sec : sections) {
    HexagonAttributeParser parser;
    if (Error e = parser.parse(sec->content(), llvm::endianness::little))
      Warn(ctx) << sec << ": " << std::move(e);
    for (const auto &tag : attributesTags) {
      switch (HexagonAttrs::AttrType(tag.attr)) {
      case HexagonAttrs::ARCH:
      case HexagonAttrs::HVXARCH:
        if (auto i = parser.getAttributeValue(tag.attr)) {
          auto r = merged.intAttr.try_emplace(tag.attr, *i);
          if (!r.second)
            if (r.first->second < *i)
              r.first->second = *i;
        }
        continue;
```

- EN: Declares or implements routines including `getHexagonAttributeTags`, `Warn`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getHexagonAttributeTags`, `Warn`.
- CN: 这里声明或实现函数，例如 `getHexagonAttributeTags`, `Warn`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getHexagonAttributeTags`, `Warn`。

### Lines 612-627

```cpp
      case HexagonAttrs::HVXIEEEFP:
      case HexagonAttrs::HVXQFLOAT:
      case HexagonAttrs::ZREG:
      case HexagonAttrs::AUDIO:
      case HexagonAttrs::CABAC:
        if (auto i = parser.getAttributeValue(tag.attr)) {
          auto r = merged.intAttr.try_emplace(tag.attr, *i);
          if (!r.second && r.first->second != *i) {
            r.first->second |= *i;
          }
        }
        continue;
      }
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 628-637

```cpp
  // The total size of headers: format-version [ <section-length> "vendor-name"
  // [ <file-tag> <size>.
  size_t size = 5 + merged.vendor.size() + 1 + 5;
  for (auto &attr : merged.intAttr)
    if (attr.second != 0)
      size += getULEB128Size(attr.first) + getULEB128Size(attr.second);
  merged.size = size;
  return &merged;
}
```

- EN: Declares or implements routines including `getULEB128Size`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getULEB128Size`.
- CN: 这里声明或实现函数，例如 `getULEB128Size`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getULEB128Size`。

### Lines 638-647

```cpp
void HexagonAttributesSection::writeTo(uint8_t *buf) {
  const size_t size = getSize();
  uint8_t *const end = buf + size;
  *buf = ELFAttrs::Format_Version;
  write32(ctx, buf + 1, size - 1);
  buf += 5;

  memcpy(buf, vendor.data(), vendor.size());
  buf += vendor.size() + 1;
```

- EN: Declares or implements routines including `writeTo`, `getSize`, `write32`, `memcpy`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `writeTo`, `getSize`, `write32`, `memcpy`.
- CN: 这里声明或实现函数，例如 `writeTo`, `getSize`, `write32`, `memcpy`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `writeTo`, `getSize`, `write32`, `memcpy`。

### Lines 648-659

```cpp
  *buf = ELFAttrs::File;
  write32(ctx, buf + 1, end - buf);
  buf += 5;

  for (auto &attr : intAttr) {
    if (attr.second == 0)
      continue;
    buf += encodeULEB128(attr.first, buf);
    buf += encodeULEB128(attr.second, buf);
  }
}
```

- EN: Declares or implements routines including `write32`, `encodeULEB128`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `write32`, `encodeULEB128`.
- CN: 这里声明或实现函数，例如 `write32`, `encodeULEB128`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `write32`, `encodeULEB128`。

### Lines 660-668

```cpp
void elf::mergeHexagonAttributesSections(Ctx &ctx) {
  // Find the first input SHT_HEXAGON_ATTRIBUTES; return if not found.
  size_t place =
      llvm::find_if(ctx.inputSections,
                    [](auto *s) { return s->type == SHT_HEXAGON_ATTRIBUTES; }) -
      ctx.inputSections.begin();
  if (place == ctx.inputSections.size())
    return;
```

- EN: Declares or implements routines including `mergeHexagonAttributesSections`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `mergeHexagonAttributesSections`.
- CN: 这里声明或实现函数，例如 `mergeHexagonAttributesSections`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `mergeHexagonAttributesSections`。

### Lines 669-677

```cpp
  // Extract all SHT_HEXAGON_ATTRIBUTES sections into `sections`.
  SmallVector<InputSectionBase *, 0> sections;
  llvm::erase_if(ctx.inputSections, [&](InputSectionBase *s) {
    if (s->type != SHT_HEXAGON_ATTRIBUTES)
      return false;
    sections.push_back(s);
    return true;
  });
```

- EN: Declares or implements routines including `erase_if`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `erase_if`.
- CN: 这里声明或实现函数，例如 `erase_if`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `erase_if`。

### Lines 678-693

```cpp
  // Add the merged section.
  ctx.inputSections.insert(ctx.inputSections.begin() + place,
                           mergeAttributesSection(ctx, sections));
}

static bool isGDPLT(RelType type) {
  switch (type) {
  case R_HEX_GD_PLT_B22_PCREL:
  case R_HEX_GD_PLT_B22_PCREL_X:
  case R_HEX_GD_PLT_B32_PCREL_X:
    return true;
  default:
    return false;
  }
}
```

- EN: Declares or implements routines including `mergeAttributesSection`, `isGDPLT`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `mergeAttributesSection`, `isGDPLT`.
- CN: 这里声明或实现函数，例如 `mergeAttributesSection`, `isGDPLT`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `mergeAttributesSection`, `isGDPLT`。

### Lines 694-711

```cpp
void Hexagon::finalizeRelocScan() {
  Symbol *tga = nullptr;

  // Scan for R_HEX_GD_PLT_* relocations (recorded as R_PLT_PC by
  // scanSectionImpl) and rebind them to __tls_get_addr.
  for (ELFFileBase *f : ctx.objectFiles) {
    for (InputSectionBase *s : f->getSections()) {
      auto *isec = dyn_cast_or_null<InputSection>(s);
      if (!isec || !isec->isLive())
        continue;
      for (Relocation &rel : isec->relocs()) {
        if (rel.expr != R_PLT_PC || !isGDPLT(rel.type))
          continue;
        if (!tga) {
          tga = ctx.symtab->addSymbol(Undefined{ctx.internalFile,
                                                "__tls_get_addr", STB_GLOBAL,
                                                STV_DEFAULT, STT_FUNC});
          tga->isUsedInRegularObj = true;
```

- EN: Declares or implements routines including `finalizeRelocScan`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `finalizeRelocScan`.
- CN: 这里声明或实现函数，例如 `finalizeRelocScan`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `finalizeRelocScan`。

### Lines 712-720

```cpp
          tga->isPreemptible = true;
          tga->setFlags(NEEDS_PLT | USED);
        }
        rel.sym = tga;
      }
    }
  }
}
```

- EN: Declares or implements routines including `setFlags`. Notable symbols here include `setFlags`.
- CN: 这里声明或实现函数，例如 `setFlags`。这里较值得关注的符号包括 `setFlags`。

### Lines 721-721

```cpp
void elf::setHexagonTargetInfo(Ctx &ctx) { ctx.target.reset(new Hexagon(ctx)); }
```

- EN: Declares or implements routines including `setHexagonTargetInfo`. Notable symbols here include `setHexagonTargetInfo`.
- CN: 这里声明或实现函数，例如 `setHexagonTargetInfo`。这里较值得关注的符号包括 `setHexagonTargetInfo`。

## Key Concepts / 关键概念

- `Hexagon`: class or struct interface / 类或结构体接口
- `ELFT`: class or struct interface / 类或结构体接口
- `RelTy`: class or struct interface / 类或结构体接口
- `InstructionMask`: class or struct interface / 类或结构体接口
- `Hexagon`: function or method entry point / 函数或方法入口
- `calcEFlags`: function or method entry point / 函数或方法入口
- `getDynRel`: function or method entry point / 函数或方法入口
- `getImplicitAddend`: function or method entry point / 函数或方法入口

## Dependencies / 依赖关系

- Local headers / 本地头文件: `lld/Common/ErrorHandler.h`
- LLVM headers / LLVM 头文件: `llvm/ADT/SmallVector.h`, `llvm/BinaryFormat/ELF.h`, `llvm/Support/ELFAttributes.h`, `llvm/Support/Endian.h`, `llvm/Support/HexagonAttributeParser.h`, `llvm/Support/HexagonAttributes.h`, `llvm/Support/LEB128.h`
- System headers / 系统头文件: `InputFiles.h`, `OutputSections.h`, `RelocScan.h`, `SymbolTable.h`, `Symbols.h`, `SyntheticSections.h`, `Target.h`, `Thunks.h`
- Directory context / 目录上下文: `lld/ELF/Arch` neighbors usually cooperate with this file to provide the surrounding subsystem / `lld/ELF/Arch` 下的相邻文件通常与本文件协作组成对应子系统
