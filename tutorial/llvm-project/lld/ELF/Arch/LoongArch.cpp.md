# LoongArch.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `lld/ELF/Arch/LoongArch.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: LoongArch.cpp. It also sits in code that implements ELF linking components in LLD. / 该文件实现 LLD 中的 ELF 链接组件。 源码头部说明其职责是：LoongArch.cpp。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
//===- LoongArch.cpp ------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 9-17

```cpp
#include "InputFiles.h"
#include "OutputSections.h"
#include "RelocScan.h"
#include "Symbols.h"
#include "SyntheticSections.h"
#include "Target.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/Support/LEB128.h"
```

- EN: Pulls in 8 header(s) from LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 8 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 18-35

```cpp
using namespace llvm;
using namespace llvm::object;
using namespace llvm::support::endian;
using namespace llvm::ELF;
using namespace lld;
using namespace lld::elf;

namespace {
class LoongArch final : public TargetInfo {
public:
  LoongArch(Ctx &);
  uint32_t calcEFlags() const override;
  int64_t getImplicitAddend(const uint8_t *buf, RelType type) const override;
  void writeGotPlt(uint8_t *buf, const Symbol &s) const override;
  void writeIgotPlt(uint8_t *buf, const Symbol &s) const override;
  void writePltHeader(uint8_t *buf) const override;
  void writePlt(uint8_t *buf, const Symbol &sym,
                uint64_t pltEntryAddr) const override;
```

- EN: Works inside namespace scope `llvm`, `lld` to organize symbols. Introduces type definitions such as `LoongArch`. Declares or implements routines including `LoongArch`, `calcEFlags`, `getImplicitAddend`, `writeGotPlt`, `writeIgotPlt`, and 1 more. Notable symbols here include `LoongArch`, `calcEFlags`, `getImplicitAddend`, `writeGotPlt`, `writeIgotPlt`, `writePltHeader`.
- CN: 这里位于命名空间 `llvm`, `lld` 中，用于组织符号作用域。这里引入类型定义，例如 `LoongArch`。这里声明或实现函数，例如 `LoongArch`, `calcEFlags`, `getImplicitAddend`, `writeGotPlt`, `writeIgotPlt`, and 1 more。这里较值得关注的符号包括 `LoongArch`, `calcEFlags`, `getImplicitAddend`, `writeGotPlt`, `writeIgotPlt`, `writePltHeader`。

### Lines 36-53

```cpp
  RelType getDynRel(RelType type) const override;
  RelExpr getRelExpr(RelType type, const Symbol &s,
                     const uint8_t *loc) const override;
  bool usesOnlyLowPageBits(RelType type) const override;
  template <class ELFT, class RelTy>
  void scanSectionImpl(InputSectionBase &, Relocs<RelTy>);
  void scanSection(InputSectionBase &sec) override {
    if (ctx.arg.is64)
      elf::scanSection1<LoongArch, ELF64LE>(*this, sec);
    else
      elf::scanSection1<LoongArch, ELF32LE>(*this, sec);
  }
  void relocate(uint8_t *loc, const Relocation &rel,
                uint64_t val) const override;
  bool relaxOnce(int pass) const override;
  bool synthesizeAlign(uint64_t &dot, InputSection *sec) override;
  void relocateAlloc(InputSection &sec, uint8_t *buf) const override;
  void finalizeRelax(int passes) const override;
```

- EN: Introduces type definitions such as `ELFT`, `RelTy`. Declares or implements routines including `getDynRel`, `usesOnlyLowPageBits`, `scanSectionImpl`, `scanSection`, `relaxOnce`, and 3 more. Notable symbols here include `ELFT`, `RelTy`, `getDynRel`, `usesOnlyLowPageBits`, `scanSectionImpl`, `scanSection`.
- CN: 这里引入类型定义，例如 `ELFT`, `RelTy`。这里声明或实现函数，例如 `getDynRel`, `usesOnlyLowPageBits`, `scanSectionImpl`, `scanSection`, `relaxOnce`, and 3 more。这里较值得关注的符号包括 `ELFT`, `RelTy`, `getDynRel`, `usesOnlyLowPageBits`, `scanSectionImpl`, `scanSection`。

### Lines 54-66

```cpp

private:
  void tlsdescToIe(uint8_t *loc, const Relocation &rel, uint64_t val) const;
  void tlsdescToLe(uint8_t *loc, const Relocation &rel, uint64_t val) const;
  bool tryGotToPCRel(uint8_t *loc, const Relocation &rHi20,
                     const Relocation &rLo12, uint64_t secAddr) const;
  template <class ELFT, class RelTy>
  bool synthesizeAlignForInput(uint64_t &dot, InputSection *sec,
                               Relocs<RelTy> rels);
  template <class ELFT, class RelTy>
  void finalizeSynthesizeAligns(uint64_t &dot, InputSection *sec,
                                Relocs<RelTy> rels);
  template <class ELFT>
```

- EN: Introduces type definitions such as `ELFT`, `RelTy`. Declares or implements routines including `tlsdescToIe`, `tlsdescToLe`. Notable symbols here include `ELFT`, `RelTy`, `tlsdescToIe`, `tlsdescToLe`.
- CN: 这里引入类型定义，例如 `ELFT`, `RelTy`。这里声明或实现函数，例如 `tlsdescToIe`, `tlsdescToLe`。这里较值得关注的符号包括 `ELFT`, `RelTy`, `tlsdescToIe`, `tlsdescToLe`。

### Lines 67-75

```cpp
  bool synthesizeAlignAux(uint64_t &dot, InputSection *sec);

  // The following two variables are used by synthesized ALIGN relocations.
  InputSection *baseSec = nullptr;
  // r_offset and r_addend pairs.
  SmallVector<std::pair<uint64_t, uint64_t>, 0> synthesizedAligns;
};
} // end anonymous namespace
```

- EN: Declares or implements routines including `synthesizeAlignAux`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `synthesizeAlignAux`.
- CN: 这里声明或实现函数，例如 `synthesizeAlignAux`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `synthesizeAlignAux`。

### Lines 76-93

```cpp
namespace {
enum Op {
  SUB_W = 0x00110000,
  SUB_D = 0x00118000,
  BREAK = 0x002a0000,
  SRLI_W = 0x00448000,
  SRLI_D = 0x00450000,
  ADDI_W = 0x02800000,
  ADDI_D = 0x02c00000,
  ANDI = 0x03400000,
  ORI = 0x03800000,
  LU12I_W = 0x14000000,
  PCADDI = 0x18000000,
  PCADDU12I = 0x1c000000,
  PCALAU12I = 0x1a000000,
  LD_W = 0x28800000,
  LD_D = 0x28c00000,
  JIRL = 0x4c000000,
```

- EN: Defines enumerations such as `Op` to encode states or modes. Notable symbols here include `Op`.
- CN: 这里定义枚举 `Op`，用于表达状态或模式。这里较值得关注的符号包括 `Op`。

### Lines 94-109

```cpp
  B = 0x50000000,
  BL = 0x54000000,
};

enum Reg {
  R_ZERO = 0,
  R_RA = 1,
  R_TP = 2,
  R_A0 = 4,
  R_T0 = 12,
  R_T1 = 13,
  R_T2 = 14,
  R_T3 = 15,
};
} // namespace
```

- EN: Defines enumerations such as `Reg` to encode states or modes. Notable symbols here include `Reg`.
- CN: 这里定义枚举 `Reg`，用于表达状态或模式。这里较值得关注的符号包括 `Reg`。

### Lines 110-124

```cpp
// Mask out the input's lowest 12 bits for use with `pcalau12i`, in sequences
// like `pcalau12i + addi.[wd]` or `pcalau12i + {ld,st}.*` where the `pcalau12i`
// produces a PC-relative intermediate value with the lowest 12 bits zeroed (the
// "page") for the next instruction to add in the "page offset". (`pcalau12i`
// stands for something like "PC ALigned Add Upper that starts from the 12th
// bit, Immediate".)
//
// Here a "page" is in fact just another way to refer to the 12-bit range
// allowed by the immediate field of the addi/ld/st instructions, and not
// related to the system or the kernel's actual page size. The semantics happen
// to match the AArch64 `adrp`, so the concept of "page" is borrowed here.
static uint64_t getLoongArchPage(uint64_t p) {
  return p & ~static_cast<uint64_t>(0xfff);
}
```

- EN: Declares or implements routines including `getLoongArchPage`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getLoongArchPage`.
- CN: 这里声明或实现函数，例如 `getLoongArchPage`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getLoongArchPage`。

### Lines 125-142

```cpp
static uint32_t lo12(uint32_t val) { return val & 0xfff; }

// Calculate the adjusted page delta between dest and PC.
uint64_t elf::getLoongArchPageDelta(uint64_t dest, uint64_t pc, RelType type) {
  // Note that if the sequence being relocated is `pcalau12i + addi.d + lu32i.d
  // + lu52i.d`, they must be adjacent so that we can infer the PC of
  // `pcalau12i` when calculating the page delta for the other two instructions
  // (lu32i.d and lu52i.d). Compensate all the sign-extensions is a bit
  // complicated. Just use psABI recommended algorithm.
  uint64_t pcalau12i_pc;
  switch (type) {
  case R_LARCH_PCALA64_LO20:
  case R_LARCH_GOT64_PC_LO20:
  case R_LARCH_TLS_IE64_PC_LO20:
  case R_LARCH_TLS_DESC64_PC_LO20:
    pcalau12i_pc = pc - 8;
    break;
  case R_LARCH_PCALA64_HI12:
```

- EN: Declares or implements routines including `lo12`, `getLoongArchPageDelta`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `lo12`, `getLoongArchPageDelta`.
- CN: 这里声明或实现函数，例如 `lo12`, `getLoongArchPageDelta`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `lo12`, `getLoongArchPageDelta`。

### Lines 143-159

```cpp
  case R_LARCH_GOT64_PC_HI12:
  case R_LARCH_TLS_IE64_PC_HI12:
  case R_LARCH_TLS_DESC64_PC_HI12:
    pcalau12i_pc = pc - 12;
    break;
  default:
    pcalau12i_pc = pc;
    break;
  }
  uint64_t result = getLoongArchPage(dest) - getLoongArchPage(pcalau12i_pc);
  if (dest & 0x800)
    result += 0x1000 - 0x1'0000'0000;
  if (result & 0x8000'0000)
    result += 0x1'0000'0000;
  return result;
}
```

- EN: Declares or implements routines including `getLoongArchPage`. Notable symbols here include `getLoongArchPage`.
- CN: 这里声明或实现函数，例如 `getLoongArchPage`。这里较值得关注的符号包括 `getLoongArchPage`。

### Lines 160-170

```cpp
static uint32_t hi20(uint32_t val) { return (val + 0x800) >> 12; }

static uint32_t insn(uint32_t op, uint32_t d, uint32_t j, uint32_t k) {
  return op | d | (j << 5) | (k << 10);
}

// Extract bits v[begin:end], where range is inclusive.
static uint32_t extractBits(uint64_t v, uint32_t begin, uint32_t end) {
  return begin == 63 ? v >> end : (v & ((1ULL << (begin + 1)) - 1)) >> end;
}
```

- EN: Declares or implements routines including `hi20`, `insn`, `extractBits`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `hi20`, `insn`, `extractBits`.
- CN: 这里声明或实现函数，例如 `hi20`, `insn`, `extractBits`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `hi20`, `insn`, `extractBits`。

### Lines 171-180

```cpp
static uint32_t getD5(uint64_t v) { return extractBits(v, 4, 0); }

static uint32_t getJ5(uint64_t v) { return extractBits(v, 9, 5); }

static uint32_t setD5k16(uint32_t insn, uint32_t imm) {
  uint32_t immLo = extractBits(imm, 15, 0);
  uint32_t immHi = extractBits(imm, 20, 16);
  return (insn & 0xfc0003e0) | (immLo << 10) | immHi;
}
```

- EN: Declares or implements routines including `getD5`, `getJ5`, `setD5k16`, `extractBits`. Notable symbols here include `getD5`, `getJ5`, `setD5k16`, `extractBits`.
- CN: 这里声明或实现函数，例如 `getD5`, `getJ5`, `setD5k16`, `extractBits`。这里较值得关注的符号包括 `getD5`, `getJ5`, `setD5k16`, `extractBits`。

### Lines 181-190

```cpp
static uint32_t setD10k16(uint32_t insn, uint32_t imm) {
  uint32_t immLo = extractBits(imm, 15, 0);
  uint32_t immHi = extractBits(imm, 25, 16);
  return (insn & 0xfc000000) | (immLo << 10) | immHi;
}

static uint32_t setJ20(uint32_t insn, uint32_t imm) {
  return (insn & 0xfe00001f) | (extractBits(imm, 19, 0) << 5);
}
```

- EN: Declares or implements routines including `setD10k16`, `extractBits`, `setJ20`. Notable symbols here include `setD10k16`, `extractBits`, `setJ20`.
- CN: 这里声明或实现函数，例如 `setD10k16`, `extractBits`, `setJ20`。这里较值得关注的符号包括 `setD10k16`, `extractBits`, `setJ20`。

### Lines 191-198

```cpp
static uint32_t setJ5(uint32_t insn, uint32_t imm) {
  return (insn & 0xfffffc1f) | (extractBits(imm, 4, 0) << 5);
}

static uint32_t setK12(uint32_t insn, uint32_t imm) {
  return (insn & 0xffc003ff) | (extractBits(imm, 11, 0) << 10);
}
```

- EN: Declares or implements routines including `setJ5`, `setK12`. Notable symbols here include `setJ5`, `setK12`.
- CN: 这里声明或实现函数，例如 `setJ5`, `setK12`。这里较值得关注的符号包括 `setJ5`, `setK12`。

### Lines 199-206

```cpp
static uint32_t setK16(uint32_t insn, uint32_t imm) {
  return (insn & 0xfc0003ff) | (extractBits(imm, 15, 0) << 10);
}

static bool isJirl(uint32_t insn) {
  return (insn & 0xfc000000) == JIRL;
}
```

- EN: Declares or implements routines including `setK16`, `isJirl`. Notable symbols here include `setK16`, `isJirl`.
- CN: 这里声明或实现函数，例如 `setK16`, `isJirl`。这里较值得关注的符号包括 `setK16`, `isJirl`。

### Lines 207-217

```cpp
static void handleUleb128(Ctx &ctx, uint8_t *loc, uint64_t val) {
  const uint32_t maxcount = 1 + 64 / 7;
  uint32_t count;
  const char *error = nullptr;
  uint64_t orig = decodeULEB128(loc, &count, nullptr, &error);
  if (count > maxcount || (count == maxcount && error))
    Err(ctx) << getErrorLoc(ctx, loc) << "extra space for uleb128";
  uint64_t mask = count < maxcount ? (1ULL << 7 * count) - 1 : -1ULL;
  encodeULEB128((orig + val) & mask, loc, count);
}
```

- EN: Declares or implements routines including `handleUleb128`, `decodeULEB128`, `Err`, `encodeULEB128`. Notable symbols here include `handleUleb128`, `decodeULEB128`, `Err`, `encodeULEB128`.
- CN: 这里声明或实现函数，例如 `handleUleb128`, `decodeULEB128`, `Err`, `encodeULEB128`。这里较值得关注的符号包括 `handleUleb128`, `decodeULEB128`, `Err`, `encodeULEB128`。

### Lines 218-230

```cpp
LoongArch::LoongArch(Ctx &ctx) : TargetInfo(ctx) {
  // The LoongArch ISA itself does not have a limit on page sizes. According to
  // the ISA manual, the PS (page size) field in MTLB entries and CSR.STLBPS is
  // 6 bits wide, meaning the maximum page size is 2^63 which is equivalent to
  // "unlimited".
  // However, practically the maximum usable page size is constrained by the
  // kernel implementation, and 64KiB is the biggest non-huge page size
  // supported by Linux as of v6.4. The most widespread page size in use,
  // though, is 16KiB.
  defaultCommonPageSize = 16384;
  defaultMaxPageSize = 65536;
  write32le(trapInstr.data(), BREAK); // break 0
```

- EN: Declares or implements routines including `LoongArch`, `write32le`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LoongArch`, `write32le`.
- CN: 这里声明或实现函数，例如 `LoongArch`, `write32le`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LoongArch`, `write32le`。

### Lines 231-248

```cpp
  copyRel = R_LARCH_COPY;
  pltRel = R_LARCH_JUMP_SLOT;
  relativeRel = R_LARCH_RELATIVE;
  iRelativeRel = R_LARCH_IRELATIVE;

  if (ctx.arg.is64) {
    symbolicRel = R_LARCH_64;
    tlsModuleIndexRel = R_LARCH_TLS_DTPMOD64;
    tlsOffsetRel = R_LARCH_TLS_DTPREL64;
    tlsGotRel = R_LARCH_TLS_TPREL64;
    tlsDescRel = R_LARCH_TLS_DESC64;
  } else {
    symbolicRel = R_LARCH_32;
    tlsModuleIndexRel = R_LARCH_TLS_DTPMOD32;
    tlsOffsetRel = R_LARCH_TLS_DTPREL32;
    tlsGotRel = R_LARCH_TLS_TPREL32;
    tlsDescRel = R_LARCH_TLS_DESC32;
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 249-259

```cpp

  gotRel = symbolicRel;

  // .got.plt[0] = _dl_runtime_resolve, .got.plt[1] = link_map
  gotPltHeaderEntriesNum = 2;

  pltHeaderSize = 32;
  pltEntrySize = 16;
  ipltEntrySize = 16;
}
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 260-270

```cpp
static uint32_t getEFlags(Ctx &ctx, const InputFile *f) {
  if (ctx.arg.is64)
    return cast<ObjFile<ELF64LE>>(f)->getObj().getHeader().e_flags;
  return cast<ObjFile<ELF32LE>>(f)->getObj().getHeader().e_flags;
}

static bool inputFileHasCode(const InputFile *f) {
  for (const auto *sec : f->getSections())
    if (sec && sec->flags & SHF_EXECINSTR)
      return true;
```

- EN: Declares or implements routines including `getEFlags`, `inputFileHasCode`. Notable symbols here include `getEFlags`, `inputFileHasCode`.
- CN: 这里声明或实现函数，例如 `getEFlags`, `inputFileHasCode`。这里较值得关注的符号包括 `getEFlags`, `inputFileHasCode`。

### Lines 271-279

```cpp
  return false;
}

uint32_t LoongArch::calcEFlags() const {
  // If there are only binary input files (from -b binary), use a
  // value of 0 for the ELF header flags.
  if (ctx.objectFiles.empty())
    return 0;
```

- EN: Declares or implements routines including `calcEFlags`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `calcEFlags`.
- CN: 这里声明或实现函数，例如 `calcEFlags`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `calcEFlags`。

### Lines 280-288

```cpp
  uint32_t target = 0;
  const InputFile *targetFile;
  for (const InputFile *f : ctx.objectFiles) {
    // Do not enforce ABI compatibility if the input file does not contain code.
    // This is useful for allowing linkage with data-only object files produced
    // with tools like objcopy, that have zero e_flags.
    if (!inputFileHasCode(f))
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 289-301

```cpp
    // Take the first non-zero e_flags as the reference.
    uint32_t flags = getEFlags(ctx, f);
    if (target == 0 && flags != 0) {
      target = flags;
      targetFile = f;
    }

    if ((flags & EF_LOONGARCH_ABI_MODIFIER_MASK) !=
        (target & EF_LOONGARCH_ABI_MODIFIER_MASK))
      ErrAlways(ctx) << f
                     << ": cannot link object files with different ABI from "
                     << targetFile;
```

- EN: Declares or implements routines including `getEFlags`, `ErrAlways`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getEFlags`, `ErrAlways`.
- CN: 这里声明或实现函数，例如 `getEFlags`, `ErrAlways`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getEFlags`, `ErrAlways`。

### Lines 302-319

```cpp
    // We cannot process psABI v1.x / object ABI v0 files (containing stack
    // relocations), unlike ld.bfd.
    //
    // Instead of blindly accepting every v0 object and only failing at
    // relocation processing time, just disallow interlink altogether. We
    // don't expect significant usage of object ABI v0 in the wild (the old
    // world may continue using object ABI v0 for a while, but as it's not
    // binary-compatible with the upstream i.e. new-world ecosystem, it's not
    // being considered here).
    //
    // There are briefly some new-world systems with object ABI v0 binaries too.
    // It is because these systems were built before the new ABI was finalized.
    // These are not supported either due to the extremely small number of them,
    // and the few impacted users are advised to simply rebuild world or
    // reinstall a recent system.
    if ((flags & EF_LOONGARCH_OBJABI_MASK) != EF_LOONGARCH_OBJABI_V1)
      ErrAlways(ctx) << f << ": unsupported object file ABI version";
  }
```

- EN: Declares or implements routines including `ErrAlways`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `ErrAlways`.
- CN: 这里声明或实现函数，例如 `ErrAlways`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `ErrAlways`。

### Lines 320-337

```cpp

  return target;
}

int64_t LoongArch::getImplicitAddend(const uint8_t *buf, RelType type) const {
  switch (type) {
  default:
    InternalErr(ctx, buf) << "cannot read addend for relocation " << type;
    return 0;
  case R_LARCH_32:
  case R_LARCH_TLS_DTPMOD32:
  case R_LARCH_TLS_DTPREL32:
  case R_LARCH_TLS_TPREL32:
    return SignExtend64<32>(read32le(buf));
  case R_LARCH_64:
  case R_LARCH_TLS_DTPMOD64:
  case R_LARCH_TLS_DTPREL64:
  case R_LARCH_TLS_TPREL64:
```

- EN: Declares or implements routines including `getImplicitAddend`, `InternalErr`. Notable symbols here include `getImplicitAddend`, `InternalErr`.
- CN: 这里声明或实现函数，例如 `getImplicitAddend`, `InternalErr`。这里较值得关注的符号包括 `getImplicitAddend`, `InternalErr`。

### Lines 338-352

```cpp
    return read64le(buf);
  case R_LARCH_RELATIVE:
  case R_LARCH_IRELATIVE:
    return ctx.arg.is64 ? read64le(buf) : read32le(buf);
  case R_LARCH_NONE:
  case R_LARCH_JUMP_SLOT:
    // These relocations are defined as not having an implicit addend.
    return 0;
  case R_LARCH_TLS_DESC32:
    return read32le(buf + 4);
  case R_LARCH_TLS_DESC64:
    return read64le(buf + 8);
  }
}
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 353-368

```cpp
void LoongArch::writeGotPlt(uint8_t *buf, const Symbol &s) const {
  if (ctx.arg.is64)
    write64le(buf, ctx.in.plt->getVA());
  else
    write32le(buf, ctx.in.plt->getVA());
}

void LoongArch::writeIgotPlt(uint8_t *buf, const Symbol &s) const {
  if (ctx.arg.writeAddends) {
    if (ctx.arg.is64)
      write64le(buf, s.getVA(ctx));
    else
      write32le(buf, s.getVA(ctx));
  }
}
```

- EN: Declares or implements routines including `writeGotPlt`, `write64le`, `write32le`, `writeIgotPlt`. Notable symbols here include `writeGotPlt`, `write64le`, `write32le`, `writeIgotPlt`.
- CN: 这里声明或实现函数，例如 `writeGotPlt`, `write64le`, `write32le`, `writeIgotPlt`。这里较值得关注的符号包括 `writeGotPlt`, `write64le`, `write32le`, `writeIgotPlt`。

### Lines 369-386

```cpp
void LoongArch::writePltHeader(uint8_t *buf) const {
  // The LoongArch PLT is currently structured just like that of RISCV.
  // Annoyingly, this means the PLT is still using `pcaddu12i` to perform
  // PC-relative addressing (because `pcaddu12i` is the same as RISCV `auipc`),
  // in contrast to the AArch64-like page-offset scheme with `pcalau12i` that
  // is used everywhere else involving PC-relative operations in the LoongArch
  // ELF psABI v2.00.
  //
  // The `pcrel_{hi20,lo12}` operators are illustrative only and not really
  // supported by LoongArch assemblers.
  //
  //   pcaddu12i $t2, %pcrel_hi20(.got.plt)
  //   sub.[wd]  $t1, $t1, $t3
  //   ld.[wd]   $t3, $t2, %pcrel_lo12(.got.plt)  ; t3 = _dl_runtime_resolve
  //   addi.[wd] $t1, $t1, -pltHeaderSize-12      ; t1 = &.plt[i] - &.plt[0]
  //   addi.[wd] $t0, $t2, %pcrel_lo12(.got.plt)
  //   srli.[wd] $t1, $t1, (is64?1:2)             ; t1 = &.got.plt[i] - &.got.plt[0]
  //   ld.[wd]   $t0, $t0, Wordsize               ; t0 = link_map
```

- EN: Declares or implements routines including `writePltHeader`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `writePltHeader`.
- CN: 这里声明或实现函数，例如 `writePltHeader`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `writePltHeader`。

### Lines 387-403

```cpp
  //   jr        $t3
  uint32_t offset = ctx.in.gotPlt->getVA() - ctx.in.plt->getVA();
  uint32_t sub = ctx.arg.is64 ? SUB_D : SUB_W;
  uint32_t ld = ctx.arg.is64 ? LD_D : LD_W;
  uint32_t addi = ctx.arg.is64 ? ADDI_D : ADDI_W;
  uint32_t srli = ctx.arg.is64 ? SRLI_D : SRLI_W;
  write32le(buf + 0, insn(PCADDU12I, R_T2, hi20(offset), 0));
  write32le(buf + 4, insn(sub, R_T1, R_T1, R_T3));
  write32le(buf + 8, insn(ld, R_T3, R_T2, lo12(offset)));
  write32le(buf + 12,
            insn(addi, R_T1, R_T1, lo12(-ctx.target->pltHeaderSize - 12)));
  write32le(buf + 16, insn(addi, R_T0, R_T2, lo12(offset)));
  write32le(buf + 20, insn(srli, R_T1, R_T1, ctx.arg.is64 ? 1 : 2));
  write32le(buf + 24, insn(ld, R_T0, R_T0, ctx.arg.wordsize));
  write32le(buf + 28, insn(JIRL, R_ZERO, R_T3, 0));
}
```

- EN: Declares or implements routines including `getVA`, `write32le`, `insn`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getVA`, `write32le`, `insn`.
- CN: 这里声明或实现函数，例如 `getVA`, `write32le`, `insn`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getVA`, `write32le`, `insn`。

### Lines 404-420

```cpp
void LoongArch::writePlt(uint8_t *buf, const Symbol &sym,
                     uint64_t pltEntryAddr) const {
  // See the comment in writePltHeader for reason why pcaddu12i is used instead
  // of the pcalau12i that's more commonly seen in the ELF psABI v2.0 days.
  //
  //   pcaddu12i $t3, %pcrel_hi20(f@.got.plt)
  //   ld.[wd]   $t3, $t3, %pcrel_lo12(f@.got.plt)
  //   jirl      $t1, $t3, 0
  //   nop
  uint32_t offset = sym.getGotPltVA(ctx) - pltEntryAddr;
  write32le(buf + 0, insn(PCADDU12I, R_T3, hi20(offset), 0));
  write32le(buf + 4,
            insn(ctx.arg.is64 ? LD_D : LD_W, R_T3, R_T3, lo12(offset)));
  write32le(buf + 8, insn(JIRL, R_T1, R_T3, 0));
  write32le(buf + 12, insn(ANDI, R_ZERO, R_ZERO, 0));
}
```

- EN: Declares or implements routines including `write32le`, `insn`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `write32le`, `insn`.
- CN: 这里声明或实现函数，例如 `write32le`, `insn`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `write32le`, `insn`。

### Lines 421-438

```cpp
RelType LoongArch::getDynRel(RelType type) const {
  return type == ctx.target->symbolicRel ? type
                                         : static_cast<RelType>(R_LARCH_NONE);
}

// Used by relocateNonAlloc(), scanEhSection(), and the extreme code model
// fallback in relocateAlloc(). For alloc sections, scanSectionImpl() is the
// primary relocation classifier.
RelExpr LoongArch::getRelExpr(const RelType type, const Symbol &s,
                              const uint8_t *loc) const {
  switch (type) {
  case R_LARCH_NONE:
    return R_NONE;
  case R_LARCH_32:
  case R_LARCH_64:
    return R_ABS;
  case R_LARCH_ADD6:
  case R_LARCH_ADD8:
```

- EN: Declares or implements routines including `getDynRel`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getDynRel`.
- CN: 这里声明或实现函数，例如 `getDynRel`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getDynRel`。

### Lines 439-456

```cpp
  case R_LARCH_ADD16:
  case R_LARCH_ADD32:
  case R_LARCH_ADD64:
  case R_LARCH_ADD_ULEB128:
  case R_LARCH_SUB6:
  case R_LARCH_SUB8:
  case R_LARCH_SUB16:
  case R_LARCH_SUB32:
  case R_LARCH_SUB64:
  case R_LARCH_SUB_ULEB128:
    // The LoongArch add/sub relocs behave like the RISCV counterparts; reuse
    // the RelExpr to avoid code duplication.
    return RE_RISCV_ADD;
  case R_LARCH_32_PCREL:
  case R_LARCH_64_PCREL:
  case R_LARCH_PCREL20_S2:
  case R_LARCH_PCADD_HI20:
    return R_PC;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 457-474

```cpp
  default:
    Err(ctx) << getErrorLoc(ctx, loc) << "unknown relocation (" << type.v
             << ") against symbol " << &s;
    return R_NONE;
  }
}

bool LoongArch::usesOnlyLowPageBits(RelType type) const {
  switch (type) {
  default:
    return false;
  case R_LARCH_PCALA_LO12:
  case R_LARCH_GOT_LO12:
  case R_LARCH_GOT_PC_LO12:
  case R_LARCH_TLS_IE_PC_LO12:
  case R_LARCH_TLS_DESC_LO12:
  case R_LARCH_TLS_DESC_PC_LO12:
    return true;
```

- EN: Declares or implements routines including `Err`, `usesOnlyLowPageBits`. Notable symbols here include `Err`, `usesOnlyLowPageBits`.
- CN: 这里声明或实现函数，例如 `Err`, `usesOnlyLowPageBits`。这里较值得关注的符号包括 `Err`, `usesOnlyLowPageBits`。

### Lines 475-492

```cpp
  }
}

template <class ELFT, class RelTy>
void LoongArch::scanSectionImpl(InputSectionBase &sec, Relocs<RelTy> rels) {
  RelocScan rs(ctx, &sec);
  sec.relocations.reserve(rels.size());
  for (auto it = rels.begin(); it != rels.end(); ++it) {
    RelType type = it->getType(false);
    uint32_t symIndex = it->getSymbol(false);
    Symbol &sym = sec.getFile<ELFT>()->getSymbol(symIndex);
    uint64_t offset = it->r_offset;
    if (sym.isUndefined() && symIndex != 0 &&
        rs.maybeReportUndefined(cast<Undefined>(sym), offset))
      continue;
    int64_t addend = rs.getAddend<ELFT>(*it, type);
    RelExpr expr;
    // Relocation types that only need a RelExpr set `expr` and break out of
```

- EN: Introduces type definitions such as `ELFT`, `RelTy`. Declares or implements routines including `scanSectionImpl`, `rs`, `getType`, `getSymbol`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `ELFT`, `RelTy`, `scanSectionImpl`, `rs`, `getType`, `getSymbol`.
- CN: 这里引入类型定义，例如 `ELFT`, `RelTy`。这里声明或实现函数，例如 `scanSectionImpl`, `rs`, `getType`, `getSymbol`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `ELFT`, `RelTy`, `scanSectionImpl`, `rs`, `getType`, `getSymbol`。

### Lines 493-500

```cpp
    // the switch to reach rs.process(). Types that need special handling
    // (fast-path helpers, TLS) call a handler and use `continue`.
    switch (type) {
    case R_LARCH_NONE:
    case R_LARCH_MARK_LA:
    case R_LARCH_MARK_PCREL:
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 501-510

```cpp
    // Absolute relocations:
    case R_LARCH_32:
    case R_LARCH_64:
    case R_LARCH_ABS_HI20:
    case R_LARCH_ABS_LO12:
    case R_LARCH_ABS64_LO20:
    case R_LARCH_ABS64_HI12:
      expr = R_ABS;
      break;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 511-525

```cpp
    case R_LARCH_PCALA_LO12:
      // R_LARCH_PCALA_LO12 on JIRL is used for function calls (glibc 2.37).
      expr = isJirl(read32le(sec.content().data() + offset)) ? R_PLT : R_ABS;
      break;

    // PC-indirect relocations (lo12 paired with a preceding hi20 pcadd):
    case R_LARCH_PCADD_LO12:
    case R_LARCH_GOT_PCADD_LO12:
    case R_LARCH_TLS_IE_PCADD_LO12:
    case R_LARCH_TLS_LD_PCADD_LO12:
    case R_LARCH_TLS_GD_PCADD_LO12:
    case R_LARCH_TLS_DESC_PCADD_LO12:
      expr = RE_LOONGARCH_PC_INDIRECT;
      break;
```

- EN: Declares or implements routines including `isJirl`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isJirl`.
- CN: 这里声明或实现函数，例如 `isJirl`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isJirl`。

### Lines 526-533

```cpp
    // PC-relative relocations:
    case R_LARCH_32_PCREL:
    case R_LARCH_64_PCREL:
    case R_LARCH_PCREL20_S2:
    case R_LARCH_PCADD_HI20:
      rs.processR_PC(type, offset, addend, sym);
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 534-542

```cpp
    // PLT-generating relocations:
    case R_LARCH_B16:
    case R_LARCH_B21:
    case R_LARCH_B26:
    case R_LARCH_CALL30:
    case R_LARCH_CALL36:
      rs.processR_PLT_PC(type, offset, addend, sym);
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 543-560

```cpp
    // Page-PC relocations:
    case R_LARCH_PCALA_HI20:
      // Why not RE_LOONGARCH_PAGE_PC, majority of references don't go through
      // PLT anyway so why waste time checking only to get everything relaxed
      // back to it?
      //
      // This is again due to the R_LARCH_PCALA_LO12 on JIRL case, where we want
      // both the HI20 and LO12 to potentially refer to the PLT. But in reality
      // the HI20 reloc appears earlier, and the relocs don't contain enough
      // information to let us properly resolve semantics per symbol.
      // Unlike RISCV, our LO12 relocs *do not* point to their corresponding
      // HI20 relocs, hence it is nearly impossible to 100% accurately determine
      // each HI20's "flavor" without taking big performance hits, in the
      // presence of edge cases (e.g. HI20 without pairing LO12; paired LO12
      // placed so far apart that relationship is not certain anymore), and
      // programmer mistakes (e.g. as outlined in
      // https://github.com/loongson/la-abi-specs/pull/3).
      //
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 561-576

```cpp
      // Ideally we would scan in an extra pass for all LO12s on JIRL, then mark
      // every HI20 reloc referring to the same symbol differently; this is not
      // feasible with the current function signature of getRelExpr that doesn't
      // allow for such inter-pass state.
      //
      // So, unfortunately we have to again workaround this quirk the same way
      // as BFD: assuming every R_LARCH_PCALA_HI20 is potentially PLT-needing,
      // only relaxing back to RE_LOONGARCH_PAGE_PC if it's known not so at a
      // later stage.
      expr = RE_LOONGARCH_PLT_PAGE_PC;
      break;
    case R_LARCH_PCALA64_LO20:
    case R_LARCH_PCALA64_HI12:
      expr = RE_LOONGARCH_PAGE_PC;
      break;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 577-594

```cpp
    // GOT-generating relocations:
    case R_LARCH_GOT_PC_HI20:
    case R_LARCH_GOT64_PC_LO20:
    case R_LARCH_GOT64_PC_HI12:
      expr = RE_LOONGARCH_GOT_PAGE_PC;
      break;
    case R_LARCH_GOT_PCADD_HI20:
      expr = R_GOT_PC;
      break;
    case R_LARCH_GOT_PC_LO12:
      expr = RE_LOONGARCH_GOT;
      break;
    case R_LARCH_GOT_HI20:
    case R_LARCH_GOT_LO12:
    case R_LARCH_GOT64_LO20:
    case R_LARCH_GOT64_HI12:
      expr = R_GOT;
      break;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 595-612

```cpp

    // DTPREL relocations:
    case R_LARCH_TLS_DTPREL32:
    case R_LARCH_TLS_DTPREL64:
      expr = R_DTPREL;
      break;

    // TLS LE relocations:
    case R_LARCH_TLS_TPREL32:
    case R_LARCH_TLS_TPREL64:
    case R_LARCH_TLS_LE_HI20:
    case R_LARCH_TLS_LE_HI20_R:
    case R_LARCH_TLS_LE_LO12:
    case R_LARCH_TLS_LE_LO12_R:
    case R_LARCH_TLS_LE64_LO20:
    case R_LARCH_TLS_LE64_HI12:
      if (rs.checkTlsLe(offset, sym, type))
        continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 613-630

```cpp
      expr = R_TPREL;
      break;
    // TLS IE relocations (optimizable to LE in non-extreme code model):
    case R_LARCH_TLS_IE_PC_HI20:
      rs.handleTlsIe(RE_LOONGARCH_GOT_PAGE_PC, type, offset, addend, sym);
      continue;
    case R_LARCH_TLS_IE_PC_LO12:
      rs.handleTlsIe(RE_LOONGARCH_GOT, type, offset, addend, sym);
      continue;
    // TLS IE relocations (extreme code model, no IE->LE optimization):
    case R_LARCH_TLS_IE64_PC_LO20:
    case R_LARCH_TLS_IE64_PC_HI12:
      rs.handleTlsIe<false>(RE_LOONGARCH_GOT_PAGE_PC, type, offset, addend,
                            sym);
      continue;
    // TLS IE relocations (pcadd/absolute, no IE->LE optimization):
    case R_LARCH_TLS_IE_PCADD_HI20:
      rs.handleTlsIe<false>(R_GOT_PC, type, offset, addend, sym);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 631-648

```cpp
      continue;
    case R_LARCH_TLS_IE_HI20:
    case R_LARCH_TLS_IE_LO12:
    case R_LARCH_TLS_IE64_LO20:
    case R_LARCH_TLS_IE64_HI12:
      rs.handleTlsIe<false>(R_GOT, type, offset, addend, sym);
      continue;
    // TLS GD/LD relocations (no GD/LD->IE/LE optimization):
    case R_LARCH_TLS_LD_PC_HI20:
    case R_LARCH_TLS_GD_PC_HI20:
      sym.setFlags(NEEDS_TLSGD);
      sec.addReloc({RE_LOONGARCH_TLSGD_PAGE_PC, type, offset, addend, &sym});
      continue;
    case R_LARCH_TLS_LD_HI20:
      ctx.needsTlsLd.store(true, std::memory_order_relaxed);
      sec.addReloc({R_TLSLD_GOT, type, offset, addend, &sym});
      continue;
    case R_LARCH_TLS_GD_HI20:
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 649-662

```cpp
      sym.setFlags(NEEDS_TLSGD);
      sec.addReloc({R_TLSGD_GOT, type, offset, addend, &sym});
      continue;
    case R_LARCH_TLS_LD_PCREL20_S2:
    case R_LARCH_TLS_LD_PCADD_HI20:
      ctx.needsTlsLd.store(true, std::memory_order_relaxed);
      sec.addReloc({R_TLSLD_PC, type, offset, addend, &sym});
      continue;
    case R_LARCH_TLS_GD_PCREL20_S2:
    case R_LARCH_TLS_GD_PCADD_HI20:
      sym.setFlags(NEEDS_TLSGD);
      sec.addReloc({R_TLSGD_PC, type, offset, addend, &sym});
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 663-680

```cpp
    // TLSDESC relocations (optimizable to IE/LE in non-extreme code model):
    case R_LARCH_TLS_DESC_PC_HI20:
      rs.handleTlsDesc(RE_LOONGARCH_TLSDESC_PAGE_PC, RE_LOONGARCH_GOT_PAGE_PC,
                       type, offset, addend, sym);
      continue;
    case R_LARCH_TLS_DESC_PC_LO12:
    case R_LARCH_TLS_DESC_LD:
      rs.handleTlsDesc(R_TLSDESC, RE_LOONGARCH_GOT_PAGE_PC, type, offset,
                       addend, sym);
      continue;
    case R_LARCH_TLS_DESC_PCREL20_S2:
      rs.handleTlsDesc(R_TLSDESC_PC, RE_LOONGARCH_GOT_PAGE_PC, type, offset,
                       addend, sym);
      continue;
    case R_LARCH_TLS_DESC_CALL:
      if (!ctx.arg.shared)
        sec.addReloc(
            {sym.isPreemptible ? R_GOT : R_TPREL, type, offset, addend, &sym});
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 681-697

```cpp
      continue;
    // TLSDESC relocations (extreme code model, no optimization):
    case R_LARCH_TLS_DESC64_PC_LO20:
    case R_LARCH_TLS_DESC64_PC_HI12:
      sym.setFlags(NEEDS_TLSDESC);
      sec.addReloc({RE_LOONGARCH_TLSDESC_PAGE_PC, type, offset, addend, &sym});
      continue;
    // TLSDESC relocations (absolute/pcadd, no optimization):
    case R_LARCH_TLS_DESC_HI20:
    case R_LARCH_TLS_DESC_LO12:
    case R_LARCH_TLS_DESC64_LO20:
    case R_LARCH_TLS_DESC64_HI12:
    case R_LARCH_TLS_DESC_PCADD_HI20:
      sym.setFlags(NEEDS_TLSDESC);
      sec.addReloc({R_TLSDESC, type, offset, addend, &sym});
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 698-707

```cpp
    // Relaxation hints:
    case R_LARCH_TLS_LE_ADD_R:
    case R_LARCH_RELAX:
      if (ctx.arg.relax)
        sec.addReloc({R_RELAX_HINT, type, offset, addend, &sym});
      continue;
    case R_LARCH_ALIGN:
      sec.addReloc({R_RELAX_HINT, type, offset, addend, &sym});
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 708-723

```cpp
    // Misc relocations:
    case R_LARCH_ADD6:
    case R_LARCH_ADD8:
    case R_LARCH_ADD16:
    case R_LARCH_ADD32:
    case R_LARCH_ADD64:
    case R_LARCH_ADD_ULEB128:
    case R_LARCH_SUB6:
    case R_LARCH_SUB8:
    case R_LARCH_SUB16:
    case R_LARCH_SUB32:
    case R_LARCH_SUB64:
    case R_LARCH_SUB_ULEB128:
      expr = RE_RISCV_ADD;
      break;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 724-732

```cpp
    default:
      Err(ctx) << getErrorLoc(ctx, sec.content().data() + offset)
               << "unknown relocation (" << type.v << ") against symbol "
               << &sym;
      continue;
    }
    rs.process(expr, type, offset, sym, addend);
  }
```

- EN: Declares or implements routines including `Err`, `relocation`. Notable symbols here include `Err`, `relocation`.
- CN: 这里声明或实现函数，例如 `Err`, `relocation`。这里较值得关注的符号包括 `Err`, `relocation`。

### Lines 733-750

```cpp
  llvm::stable_sort(sec.relocs(),
                    [](const Relocation &lhs, const Relocation &rhs) {
                      return lhs.offset < rhs.offset;
                    });
}

void LoongArch::relocate(uint8_t *loc, const Relocation &rel,
                         uint64_t val) const {
  switch (rel.type) {
  case R_LARCH_32_PCREL:
    checkInt(ctx, loc, val, 32, rel);
    [[fallthrough]];
  case R_LARCH_32:
  case R_LARCH_TLS_DTPREL32:
    write32le(loc, val);
    return;
  case R_LARCH_64:
  case R_LARCH_TLS_DTPREL64:
```

- EN: Declares or implements routines including `stable_sort`, `checkInt`, `write32le`. Notable symbols here include `stable_sort`, `checkInt`, `write32le`.
- CN: 这里声明或实现函数，例如 `stable_sort`, `checkInt`, `write32le`。这里较值得关注的符号包括 `stable_sort`, `checkInt`, `write32le`。

### Lines 751-764

```cpp
  case R_LARCH_64_PCREL:
    write64le(loc, val);
    return;

  // Relocs intended for `pcaddi`.
  case R_LARCH_PCREL20_S2:
  case R_LARCH_TLS_LD_PCREL20_S2:
  case R_LARCH_TLS_GD_PCREL20_S2:
  case R_LARCH_TLS_DESC_PCREL20_S2:
    checkInt(ctx, loc, val, 22, rel);
    checkAlignment(ctx, loc, val, 4, rel);
    write32le(loc, setJ20(read32le(loc), val >> 2));
    return;
```

- EN: Declares or implements routines including `write64le`, `checkInt`, `checkAlignment`, `write32le`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `write64le`, `checkInt`, `checkAlignment`, `write32le`.
- CN: 这里声明或实现函数，例如 `write64le`, `checkInt`, `checkAlignment`, `write32le`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `write64le`, `checkInt`, `checkAlignment`, `write32le`。

### Lines 765-776

```cpp
  case R_LARCH_B16:
    checkInt(ctx, loc, val, 18, rel);
    checkAlignment(ctx, loc, val, 4, rel);
    write32le(loc, setK16(read32le(loc), val >> 2));
    return;

  case R_LARCH_B21:
    checkInt(ctx, loc, val, 23, rel);
    checkAlignment(ctx, loc, val, 4, rel);
    write32le(loc, setD5k16(read32le(loc), val >> 2));
    return;
```

- EN: Declares or implements routines including `checkInt`, `checkAlignment`, `write32le`. Notable symbols here include `checkInt`, `checkAlignment`, `write32le`.
- CN: 这里声明或实现函数，例如 `checkInt`, `checkAlignment`, `write32le`。这里较值得关注的符号包括 `checkInt`, `checkAlignment`, `write32le`。

### Lines 777-794

```cpp
  case R_LARCH_B26:
    checkInt(ctx, loc, val, 28, rel);
    checkAlignment(ctx, loc, val, 4, rel);
    write32le(loc, setD10k16(read32le(loc), val >> 2));
    return;

  case R_LARCH_CALL30: {
    // This relocation is designed for adjacent pcaddu12i+jirl pairs that
    // are patched in one time.
    // The relocation range is [-2G, +2G) (of course must be 4-byte aligned).
    checkInt(ctx, loc, val, 32, rel);
    checkAlignment(ctx, loc, val, 4, rel);
    // Although jirl adds the immediate as a signed value, it is always positive
    // in this case, so no adjustment is needed, unlike CALL36.
    uint32_t hi20 = extractBits(val, 31, 12);
    // Despite the name, the lower part is actually 12 bits with 4-byte aligned.
    uint32_t lo10 = extractBits(val, 11, 2);
    write32le(loc, setJ20(read32le(loc), hi20));
```

- EN: Declares or implements routines including `checkInt`, `checkAlignment`, `write32le`, `extractBits`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `checkInt`, `checkAlignment`, `write32le`, `extractBits`.
- CN: 这里声明或实现函数，例如 `checkInt`, `checkAlignment`, `write32le`, `extractBits`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `checkInt`, `checkAlignment`, `write32le`, `extractBits`。

### Lines 795-812

```cpp
    write32le(loc + 4, setK16(read32le(loc + 4), lo10));
    return;
  }

  case R_LARCH_CALL36: {
    // This relocation is designed for adjacent pcaddu18i+jirl pairs that
    // are patched in one time. Because of sign extension of these insns'
    // immediate fields, the relocation range is [-128G - 0x20000, +128G -
    // 0x20000) (of course must be 4-byte aligned).
    if (((int64_t)val + 0x20000) != llvm::SignExtend64(val + 0x20000, 38))
      reportRangeError(ctx, loc, rel, Twine(val), llvm::minIntN(38) - 0x20000,
                       llvm::maxIntN(38) - 0x20000);
    checkAlignment(ctx, loc, val, 4, rel);
    // Since jirl performs sign extension on the offset immediate, adds (1<<17)
    // to original val to get the correct hi20.
    uint32_t hi20 = extractBits(val + (1 << 17), 37, 18);
    // Despite the name, the lower part is actually 18 bits with 4-byte aligned.
    uint32_t lo16 = extractBits(val, 17, 2);
```

- EN: Declares or implements routines including `write32le`, `reportRangeError`, `maxIntN`, `checkAlignment`, `extractBits`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `write32le`, `reportRangeError`, `maxIntN`, `checkAlignment`, `extractBits`.
- CN: 这里声明或实现函数，例如 `write32le`, `reportRangeError`, `maxIntN`, `checkAlignment`, `extractBits`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `write32le`, `reportRangeError`, `maxIntN`, `checkAlignment`, `extractBits`。

### Lines 813-830

```cpp
    write32le(loc, setJ20(read32le(loc), hi20));
    write32le(loc + 4, setK16(read32le(loc + 4), lo16));
    return;
  }

  // Relocs intended for `addi`, `ld` or `st`.
  case R_LARCH_PCALA_LO12:
    // We have to again inspect the insn word to handle the R_LARCH_PCALA_LO12
    // on JIRL case: firstly JIRL wants its immediate's 2 lowest zeroes
    // removed by us (in contrast to regular R_LARCH_PCALA_LO12), secondly
    // its immediate slot width is different too (16, not 12).
    // In this case, process like an R_LARCH_B16, but without overflow checking
    // and only taking the value's lowest 12 bits.
    if (isJirl(read32le(loc))) {
      checkAlignment(ctx, loc, val, 4, rel);
      val = SignExtend64<12>(val);
      write32le(loc, setK16(read32le(loc), val >> 2));
      return;
```

- EN: Declares or implements routines including `write32le`, `checkAlignment`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `write32le`, `checkAlignment`.
- CN: 这里声明或实现函数，例如 `write32le`, `checkAlignment`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `write32le`, `checkAlignment`。

### Lines 831-848

```cpp
    }
    [[fallthrough]];
  case R_LARCH_ABS_LO12:
  case R_LARCH_GOT_PC_LO12:
  case R_LARCH_GOT_LO12:
  case R_LARCH_TLS_LE_LO12:
  case R_LARCH_TLS_IE_PC_LO12:
  case R_LARCH_TLS_IE_LO12:
  case R_LARCH_TLS_LE_LO12_R:
  case R_LARCH_TLS_DESC_PC_LO12:
  case R_LARCH_TLS_DESC_LO12:
  case R_LARCH_PCADD_LO12:
  case R_LARCH_GOT_PCADD_LO12:
  case R_LARCH_TLS_IE_PCADD_LO12:
  case R_LARCH_TLS_LD_PCADD_LO12:
  case R_LARCH_TLS_GD_PCADD_LO12:
  case R_LARCH_TLS_DESC_PCADD_LO12:
    write32le(loc, setK12(read32le(loc), extractBits(val, 11, 0)));
```

- EN: Declares or implements routines including `write32le`. Notable symbols here include `write32le`.
- CN: 这里声明或实现函数，例如 `write32le`。这里较值得关注的符号包括 `write32le`。

### Lines 849-866

```cpp
    return;

  // Relocs intended for `lu12i.w` or `pcalau12i`.
  case R_LARCH_ABS_HI20:
  case R_LARCH_PCALA_HI20:
  case R_LARCH_GOT_PC_HI20:
  case R_LARCH_GOT_HI20:
  case R_LARCH_TLS_LE_HI20:
  case R_LARCH_TLS_IE_PC_HI20:
  case R_LARCH_TLS_IE_HI20:
  case R_LARCH_TLS_LD_PC_HI20:
  case R_LARCH_TLS_LD_HI20:
  case R_LARCH_TLS_GD_PC_HI20:
  case R_LARCH_TLS_GD_HI20:
  case R_LARCH_TLS_DESC_PC_HI20:
  case R_LARCH_TLS_DESC_HI20:
    write32le(loc, setJ20(read32le(loc), extractBits(val, 31, 12)));
    return;
```

- EN: Declares or implements routines including `write32le`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `write32le`.
- CN: 这里声明或实现函数，例如 `write32le`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `write32le`。

### Lines 867-881

```cpp
  case R_LARCH_PCADD_HI20:
  case R_LARCH_GOT_PCADD_HI20:
  case R_LARCH_TLS_IE_PCADD_HI20:
  case R_LARCH_TLS_LD_PCADD_HI20:
  case R_LARCH_TLS_GD_PCADD_HI20:
  case R_LARCH_TLS_DESC_PCADD_HI20: {
    uint64_t hi = val + 0x800;
    checkInt(ctx, loc, val, 32, rel);
    write32le(loc, setJ20(read32le(loc), extractBits(hi, 31, 12)));
    return;
  }
  case R_LARCH_TLS_LE_HI20_R:
    write32le(loc, setJ20(read32le(loc), extractBits(val + 0x800, 31, 12)));
    return;
```

- EN: Declares or implements routines including `checkInt`, `write32le`. Notable symbols here include `checkInt`, `write32le`.
- CN: 这里声明或实现函数，例如 `checkInt`, `write32le`。这里较值得关注的符号包括 `checkInt`, `write32le`。

### Lines 882-894

```cpp
  // Relocs intended for `lu32i.d`.
  case R_LARCH_ABS64_LO20:
  case R_LARCH_PCALA64_LO20:
  case R_LARCH_GOT64_PC_LO20:
  case R_LARCH_GOT64_LO20:
  case R_LARCH_TLS_LE64_LO20:
  case R_LARCH_TLS_IE64_PC_LO20:
  case R_LARCH_TLS_IE64_LO20:
  case R_LARCH_TLS_DESC64_PC_LO20:
  case R_LARCH_TLS_DESC64_LO20:
    write32le(loc, setJ20(read32le(loc), extractBits(val, 51, 32)));
    return;
```

- EN: Declares or implements routines including `write32le`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `write32le`.
- CN: 这里声明或实现函数，例如 `write32le`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `write32le`。

### Lines 895-907

```cpp
  // Relocs intended for `lu52i.d`.
  case R_LARCH_ABS64_HI12:
  case R_LARCH_PCALA64_HI12:
  case R_LARCH_GOT64_PC_HI12:
  case R_LARCH_GOT64_HI12:
  case R_LARCH_TLS_LE64_HI12:
  case R_LARCH_TLS_IE64_PC_HI12:
  case R_LARCH_TLS_IE64_HI12:
  case R_LARCH_TLS_DESC64_PC_HI12:
  case R_LARCH_TLS_DESC64_HI12:
    write32le(loc, setK12(read32le(loc), extractBits(val, 63, 52)));
    return;
```

- EN: Declares or implements routines including `write32le`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `write32le`.
- CN: 这里声明或实现函数，例如 `write32le`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `write32le`。

### Lines 908-925

```cpp
  case R_LARCH_ADD6:
    *loc = (*loc & 0xc0) | ((*loc + val) & 0x3f);
    return;
  case R_LARCH_ADD8:
    *loc += val;
    return;
  case R_LARCH_ADD16:
    write16le(loc, read16le(loc) + val);
    return;
  case R_LARCH_ADD32:
    write32le(loc, read32le(loc) + val);
    return;
  case R_LARCH_ADD64:
    write64le(loc, read64le(loc) + val);
    return;
  case R_LARCH_ADD_ULEB128:
    handleUleb128(ctx, loc, val);
    return;
```

- EN: Declares or implements routines including `write16le`, `write32le`, `write64le`, `handleUleb128`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `write16le`, `write32le`, `write64le`, `handleUleb128`.
- CN: 这里声明或实现函数，例如 `write16le`, `write32le`, `write64le`, `handleUleb128`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `write16le`, `write32le`, `write64le`, `handleUleb128`。

### Lines 926-943

```cpp
  case R_LARCH_SUB6:
    *loc = (*loc & 0xc0) | ((*loc - val) & 0x3f);
    return;
  case R_LARCH_SUB8:
    *loc -= val;
    return;
  case R_LARCH_SUB16:
    write16le(loc, read16le(loc) - val);
    return;
  case R_LARCH_SUB32:
    write32le(loc, read32le(loc) - val);
    return;
  case R_LARCH_SUB64:
    write64le(loc, read64le(loc) - val);
    return;
  case R_LARCH_SUB_ULEB128:
    handleUleb128(ctx, loc, -val);
    return;
```

- EN: Declares or implements routines including `write16le`, `write32le`, `write64le`, `handleUleb128`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `write16le`, `write32le`, `write64le`, `handleUleb128`.
- CN: 这里声明或实现函数，例如 `write16le`, `write32le`, `write64le`, `handleUleb128`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `write16le`, `write32le`, `write64le`, `handleUleb128`。

### Lines 944-953

```cpp

  case R_LARCH_MARK_LA:
  case R_LARCH_MARK_PCREL:
    // no-op
    return;

  case R_LARCH_TLS_LE_ADD_R:
  case R_LARCH_RELAX:
    return; // Ignored (for now)
```

- EN: Declares or implements routines including `Ignored`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `Ignored`.
- CN: 这里声明或实现函数，例如 `Ignored`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `Ignored`。

### Lines 954-962

```cpp
  case R_LARCH_TLS_DESC_LD:
    return; // nothing to do.
  case R_LARCH_TLS_DESC32:
    write32le(loc + 4, val);
    return;
  case R_LARCH_TLS_DESC64:
    write64le(loc + 8, val);
    return;
```

- EN: Declares or implements routines including `write32le`, `write64le`. Notable symbols here include `write32le`, `write64le`.
- CN: 这里声明或实现函数，例如 `write32le`, `write64le`。这里较值得关注的符号包括 `write32le`, `write64le`。

### Lines 963-980

```cpp
  default:
    llvm_unreachable("unknown relocation");
  }
}

// If the section alignment is > 4, advance `dot` to insert NOPs and synthesize
// an ALIGN relocation. Otherwise, return false to use default handling.
template <class ELFT, class RelTy>
bool LoongArch::synthesizeAlignForInput(uint64_t &dot, InputSection *sec,
                                        Relocs<RelTy> rels) {
  if (!baseSec) {
    // Record the first input section with RELAX relocations. We will synthesize
    // ALIGN relocations here.
    for (auto rel : rels) {
      if (rel.getType(false) == R_LARCH_RELAX) {
        baseSec = sec;
        break;
      }
```

- EN: Introduces type definitions such as `ELFT`, `RelTy`. Declares or implements routines including `llvm_unreachable`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `ELFT`, `RelTy`, `llvm_unreachable`.
- CN: 这里引入类型定义，例如 `ELFT`, `RelTy`。这里声明或实现函数，例如 `llvm_unreachable`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `ELFT`, `RelTy`, `llvm_unreachable`。

### Lines 981-997

```cpp
    }
  } else if (sec->addralign > 4) {
    // If the alignment is > 4 and the section does not start with an ALIGN
    // relocation, synthesize one.
    bool hasAlignRel = llvm::any_of(rels, [](const RelTy &rel) {
      return rel.r_offset == 0 && rel.getType(false) == R_LARCH_ALIGN;
    });
    if (!hasAlignRel) {
      synthesizedAligns.emplace_back(dot - baseSec->getVA(),
                                     sec->addralign - 4);
      dot += sec->addralign - 4;
      return true;
    }
  }
  return false;
}
```

- EN: Declares or implements routines including `if`, `any_of`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `if`, `any_of`.
- CN: 这里声明或实现函数，例如 `if`, `any_of`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `if`, `any_of`。

### Lines 998-1010

```cpp
// Finalize the relocation section by appending synthesized ALIGN relocations
// after processing all input sections.
template <class ELFT, class RelTy>
void LoongArch::finalizeSynthesizeAligns(uint64_t &dot, InputSection *sec,
                                         Relocs<RelTy> rels) {
  auto *f = cast<ObjFile<ELFT>>(baseSec->file);
  auto shdr = f->template getELFShdrs<ELFT>()[baseSec->relSecIdx];
  // Create a copy of InputSection.
  sec = make<InputSection>(*f, shdr, baseSec->name);
  auto *baseRelSec = cast<InputSection>(f->getSections()[baseSec->relSecIdx]);
  *sec = *baseRelSec;
  baseSec = nullptr;
```

- EN: Introduces type definitions such as `ELFT`, `RelTy`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `ELFT`, `RelTy`.
- CN: 这里引入类型定义，例如 `ELFT`, `RelTy`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `ELFT`, `RelTy`。

### Lines 1011-1028

```cpp
  // Allocate buffer for original and synthesized relocations in RELA format.
  // If CREL is used, OutputSection::finalizeNonAllocCrel will convert RELA to
  // CREL.
  auto newSize = rels.size() + synthesizedAligns.size();
  auto *relas = makeThreadLocalN<typename ELFT::Rela>(newSize);
  sec->size = newSize * sizeof(typename ELFT::Rela);
  sec->content_ = reinterpret_cast<uint8_t *>(relas);
  sec->type = SHT_RELA;
  // Copy original relocations to the new buffer, potentially converting CREL to
  // RELA.
  for (auto [i, r] : llvm::enumerate(rels)) {
    relas[i].r_offset = r.r_offset;
    relas[i].setSymbolAndType(r.getSymbol(0), r.getType(0), false);
    if constexpr (RelTy::HasAddend)
      relas[i].r_addend = r.r_addend;
  }
  // Append synthesized ALIGN relocations to the buffer.
  for (auto [i, r] : llvm::enumerate(synthesizedAligns)) {
```

- EN: Declares or implements routines including `sizeof`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `sizeof`.
- CN: 这里声明或实现函数，例如 `sizeof`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `sizeof`。

### Lines 1029-1046

```cpp
    auto &rela = relas[rels.size() + i];
    rela.r_offset = r.first;
    rela.setSymbolAndType(0, R_LARCH_ALIGN, false);
    rela.r_addend = r.second;
  }
  synthesizedAligns.clear();
  // Replace the old relocation section with the new one in the output section.
  // addOrphanSections ensures that the output relocation section is processed
  // after osec.
  for (SectionCommand *cmd : sec->getParent()->commands) {
    auto *isd = dyn_cast<InputSectionDescription>(cmd);
    if (!isd)
      continue;
    for (auto *&isec : isd->sections)
      if (isec == baseRelSec)
        isec = sec;
  }
}
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1047-1058

```cpp

template <class ELFT>
bool LoongArch::synthesizeAlignAux(uint64_t &dot, InputSection *sec) {
  bool ret = false;
  if (sec) {
    invokeOnRelocs(*sec, ret = synthesizeAlignForInput<ELFT>, dot, sec);
  } else if (baseSec) {
    invokeOnRelocs(*baseSec, finalizeSynthesizeAligns<ELFT>, dot, sec);
  }
  return ret;
}
```

- EN: Introduces type definitions such as `ELFT`. Declares or implements routines including `synthesizeAlignAux`, `invokeOnRelocs`, `if`. Notable symbols here include `ELFT`, `synthesizeAlignAux`, `invokeOnRelocs`, `if`.
- CN: 这里引入类型定义，例如 `ELFT`。这里声明或实现函数，例如 `synthesizeAlignAux`, `invokeOnRelocs`, `if`。这里较值得关注的符号包括 `ELFT`, `synthesizeAlignAux`, `invokeOnRelocs`, `if`。

### Lines 1059-1076

```cpp
// Without linker relaxation enabled for a particular relocatable file or
// section, the assembler will not generate R_LARCH_ALIGN relocations for
// alignment directives. This becomes problematic in a two-stage linking
// process: ld -r a.o b.o -o ab.o; ld ab.o -o ab. This function synthesizes an
// R_LARCH_ALIGN relocation at section start when needed.
//
// When called with an input section (`sec` is not null): If the section
// alignment is > 4, advance `dot` to insert NOPs and synthesize an ALIGN
// relocation.
//
// When called after all input sections are processed (`sec` is null): The
// output relocation section is updated with all the newly synthesized ALIGN
// relocations.
bool LoongArch::synthesizeAlign(uint64_t &dot, InputSection *sec) {
  assert(ctx.arg.relocatable);
  if (ctx.arg.is64)
    return synthesizeAlignAux<ELF64LE>(dot, sec);
  return synthesizeAlignAux<ELF32LE>(dot, sec);
```

- EN: Declares or implements routines including `synthesizeAlign`, `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `synthesizeAlign`, `assert`.
- CN: 这里声明或实现函数，例如 `synthesizeAlign`, `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `synthesizeAlign`, `assert`。

### Lines 1077-1087

```cpp
}

static bool relaxable(ArrayRef<Relocation> relocs, size_t i) {
  return i + 1 < relocs.size() && relocs[i + 1].type == R_LARCH_RELAX;
}

static bool isPairRelaxable(ArrayRef<Relocation> relocs, size_t i) {
  return relaxable(relocs, i) && relaxable(relocs, i + 2) &&
         relocs[i].offset + 4 == relocs[i + 2].offset;
}
```

- EN: Declares or implements routines including `relaxable`, `isPairRelaxable`. Notable symbols here include `relaxable`, `isPairRelaxable`.
- CN: 这里声明或实现函数，例如 `relaxable`, `isPairRelaxable`。这里较值得关注的符号包括 `relaxable`, `isPairRelaxable`。

### Lines 1088-1105

```cpp
// Relax code sequence.
// From:
//   pcalau12i     $a0, %pc_hi20(sym) | %ld_pc_hi20(sym)  | %gd_pc_hi20(sym)
//                    | %desc_pc_hi20(sym)
//   addi.w/d $a0, $a0, %pc_lo12(sym) | %got_pc_lo12(sym) | %got_pc_lo12(sym)
//                    | %desc_pc_lo12(sym)
// To:
//   pcaddi        $a0, %pc_lo12(sym) | %got_pc_lo12(sym) | %got_pc_lo12(sym)
//                    | %desc_pcrel_20(sym)
//
// From:
//   pcalau12i $a0, %got_pc_hi20(sym_got)
//   ld.w/d $a0, $a0, %got_pc_lo12(sym_got)
// To:
//   pcaddi $a0, %got_pc_hi20(sym_got)
static void relaxPCHi20Lo12(Ctx &ctx, const InputSection &sec, size_t i,
                            uint64_t loc, Relocation &rHi20, Relocation &rLo12,
                            uint32_t &remove) {
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 1106-1118

```cpp
  // check if the relocations are relaxable sequences.
  if (!((rHi20.type == R_LARCH_PCALA_HI20 &&
         rLo12.type == R_LARCH_PCALA_LO12) ||
        (rHi20.type == R_LARCH_GOT_PC_HI20 &&
         rLo12.type == R_LARCH_GOT_PC_LO12) ||
        (rHi20.type == R_LARCH_TLS_GD_PC_HI20 &&
         rLo12.type == R_LARCH_GOT_PC_LO12) ||
        (rHi20.type == R_LARCH_TLS_LD_PC_HI20 &&
         rLo12.type == R_LARCH_GOT_PC_LO12) ||
        (rHi20.type == R_LARCH_TLS_DESC_PC_HI20 &&
         rLo12.type == R_LARCH_TLS_DESC_PC_LO12)))
    return;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1119-1132

```cpp
  // GOT references to absolute symbols can't be relaxed to use pcaddi in
  // position-independent code, because these instructions produce a relative
  // address.
  // Meanwhile skip undefined, preemptible and STT_GNU_IFUNC symbols, because
  // these symbols may be resolve in runtime.
  // Moreover, relaxation can only occur if the addends of both relocations are
  // zero for GOT references.
  if (rHi20.type == R_LARCH_GOT_PC_HI20 &&
      (!rHi20.sym || rHi20.sym != rLo12.sym || !rHi20.sym->isDefined() ||
       rHi20.sym->isPreemptible || rHi20.sym->isGnuIFunc() ||
       (ctx.arg.isPic && !cast<Defined>(*rHi20.sym).section) ||
       rHi20.addend != 0 || rLo12.addend != 0))
    return;
```

- EN: Declares or implements routines including `isGnuIFunc`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isGnuIFunc`.
- CN: 这里声明或实现函数，例如 `isGnuIFunc`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isGnuIFunc`。

### Lines 1133-1150

```cpp
  uint64_t dest = 0;
  if (rHi20.expr == RE_LOONGARCH_PLT_PAGE_PC)
    dest = rHi20.sym->getPltVA(ctx);
  else if (rHi20.expr == RE_LOONGARCH_PAGE_PC ||
           rHi20.expr == RE_LOONGARCH_GOT_PAGE_PC)
    dest = rHi20.sym->getVA(ctx);
  else if (rHi20.expr == RE_LOONGARCH_TLSGD_PAGE_PC)
    dest = ctx.in.got->getGlobalDynAddr(*rHi20.sym);
  else if (rHi20.expr == RE_LOONGARCH_TLSDESC_PAGE_PC)
    dest = ctx.in.got->getTlsDescAddr(*rHi20.sym);
  else {
    Err(ctx) << getErrorLoc(ctx, (const uint8_t *)loc) << "unknown expr ("
             << rHi20.expr << ") against symbol " << rHi20.sym
             << "in relaxPCHi20Lo12";
    return;
  }
  dest += rHi20.addend;
```

- EN: Declares or implements routines including `getPltVA`, `getVA`, `getGlobalDynAddr`, `getTlsDescAddr`, `Err`. Notable symbols here include `getPltVA`, `getVA`, `getGlobalDynAddr`, `getTlsDescAddr`, `Err`.
- CN: 这里声明或实现函数，例如 `getPltVA`, `getVA`, `getGlobalDynAddr`, `getTlsDescAddr`, `Err`。这里较值得关注的符号包括 `getPltVA`, `getVA`, `getGlobalDynAddr`, `getTlsDescAddr`, `Err`。

### Lines 1151-1168

```cpp
  const int64_t displace = dest - loc;
  // Check if the displace aligns 4 bytes or exceeds the range of pcaddi.
  if ((displace & 0x3) != 0 || !isInt<22>(displace))
    return;

  // Note: If we can ensure that the .o files generated by LLVM only contain
  // relaxable instruction sequences with R_LARCH_RELAX, then we do not need to
  // decode instructions. The relaxable instruction sequences imply the
  // following constraints:
  // * For relocation pairs related to got_pc, the opcodes of instructions
  // must be pcalau12i + ld.w/d. In other cases, the opcodes must be pcalau12i +
  // addi.w/d.
  // * The destination register of pcalau12i is guaranteed to be used only by
  // the immediately following instruction.
  const uint32_t currInsn = read32le(sec.content().data() + rHi20.offset);
  const uint32_t nextInsn = read32le(sec.content().data() + rLo12.offset);
  // Check if use the same register.
  if (getD5(currInsn) != getJ5(nextInsn) || getJ5(nextInsn) != getD5(nextInsn))
```

- EN: Declares or implements routines including `read32le`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `read32le`.
- CN: 这里声明或实现函数，例如 `read32le`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `read32le`。

### Lines 1169-1183

```cpp
    return;

  sec.relaxAux->relocTypes[i] = R_LARCH_RELAX;
  if (rHi20.type == R_LARCH_TLS_GD_PC_HI20)
    sec.relaxAux->relocTypes[i + 2] = R_LARCH_TLS_GD_PCREL20_S2;
  else if (rHi20.type == R_LARCH_TLS_LD_PC_HI20)
    sec.relaxAux->relocTypes[i + 2] = R_LARCH_TLS_LD_PCREL20_S2;
  else if (rHi20.type == R_LARCH_TLS_DESC_PC_HI20)
    sec.relaxAux->relocTypes[i + 2] = R_LARCH_TLS_DESC_PCREL20_S2;
  else
    sec.relaxAux->relocTypes[i + 2] = R_LARCH_PCREL20_S2;
  sec.relaxAux->writes.push_back(insn(PCADDI, getD5(nextInsn), 0, 0));
  remove = 4;
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1184-1199

```cpp
// Relax code sequence.
// From:
//   la32r:
//     pcaddu12i $ra, %call30(foo)
//     jirl $ra, $ra, 0
//   la32s/la64:
//     pcaddu18i $ra, %call36(foo)
//     jirl $ra, $ra, 0
// To:
//   b/bl foo
static void relaxMediumCall(Ctx &ctx, const InputSection &sec, size_t i,
                            uint64_t loc, Relocation &r, uint32_t &remove) {
  const uint64_t dest =
      (r.expr == R_PLT_PC ? r.sym->getPltVA(ctx) : r.sym->getVA(ctx)) +
      r.addend;
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 1200-1217

```cpp
  const int64_t displace = dest - loc;
  // Check if the displace aligns 4 bytes or exceeds the range of b[l].
  if ((displace & 0x3) != 0 || !isInt<28>(displace))
    return;

  const uint32_t nextInsn = read32le(sec.content().data() + r.offset + 4);
  if (getD5(nextInsn) == R_RA) {
    // convert jirl to bl
    sec.relaxAux->relocTypes[i] = R_LARCH_B26;
    sec.relaxAux->writes.push_back(insn(BL, 0, 0, 0));
    remove = 4;
  } else if (getD5(nextInsn) == R_ZERO) {
    // convert jirl to b
    sec.relaxAux->relocTypes[i] = R_LARCH_B26;
    sec.relaxAux->writes.push_back(insn(B, 0, 0, 0));
    remove = 4;
  }
}
```

- EN: Declares or implements routines including `read32le`, `if`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `read32le`, `if`.
- CN: 这里声明或实现函数，例如 `read32le`, `if`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `read32le`, `if`。

### Lines 1218-1235

```cpp

// Relax code sequence.
// From:
//   lu12i.w $rd, %le_hi20_r(sym)
//   add.w/d $rd, $rd, $tp, %le_add_r(sym)
//   addi/ld/st.w/d $rd, $rd, %le_lo12_r(sym)
// To:
//   addi/ld/st.w/d $rd, $tp, %le_lo12_r(sym)
static void relaxTlsLe(Ctx &ctx, const InputSection &sec, size_t i,
                       uint64_t loc, Relocation &r, uint32_t &remove) {
  uint64_t val = r.sym->getVA(ctx, r.addend);
  // Check if the val exceeds the range of addi/ld/st.
  if (!isInt<12>(val))
    return;
  uint32_t currInsn = read32le(sec.content().data() + r.offset);
  switch (r.type) {
  case R_LARCH_TLS_LE_HI20_R:
  case R_LARCH_TLS_LE_ADD_R:
```

- EN: Declares or implements routines including `getVA`, `read32le`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getVA`, `read32le`.
- CN: 这里声明或实现函数，例如 `getVA`, `read32le`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getVA`, `read32le`。

### Lines 1236-1245

```cpp
    sec.relaxAux->relocTypes[i] = R_LARCH_RELAX;
    remove = 4;
    break;
  case R_LARCH_TLS_LE_LO12_R:
    sec.relaxAux->writes.push_back(setJ5(currInsn, R_TP));
    sec.relaxAux->relocTypes[i] = R_LARCH_TLS_LE_LO12_R;
    break;
  }
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1246-1253

```cpp
static bool relax(Ctx &ctx, InputSection &sec) {
  const uint64_t secAddr = sec.getVA();
  const MutableArrayRef<Relocation> relocs = sec.relocs();
  auto &aux = *sec.relaxAux;
  bool changed = false;
  ArrayRef<SymbolAnchor> sa = ArrayRef(aux.anchors);
  uint64_t delta = 0;
```

- EN: Declares or implements routines including `relax`, `ArrayRef`. Notable symbols here include `relax`, `ArrayRef`.
- CN: 这里声明或实现函数，例如 `relax`, `ArrayRef`。这里较值得关注的符号包括 `relax`, `ArrayRef`。

### Lines 1254-1271

```cpp
  std::fill_n(aux.relocTypes.get(), relocs.size(), R_LARCH_NONE);
  aux.writes.clear();
  for (auto [i, r] : llvm::enumerate(relocs)) {
    const uint64_t loc = secAddr + r.offset - delta;
    uint32_t &cur = aux.relocDeltas[i], remove = 0;
    switch (r.type) {
    case R_LARCH_ALIGN: {
      const uint64_t addend =
          r.sym->isUndefined() ? Log2_64(r.addend) + 1 : r.addend;
      const uint64_t allBytes = (1ULL << (addend & 0xff)) - 4;
      const uint64_t align = 1ULL << (addend & 0xff);
      const uint64_t maxBytes = addend >> 8;
      const uint64_t off = loc & (align - 1);
      const uint64_t curBytes = off == 0 ? 0 : align - off;
      // All bytes beyond the alignment boundary should be removed.
      // If emit bytes more than max bytes to emit, remove all.
      if (maxBytes != 0 && curBytes > maxBytes)
        remove = allBytes;
```

- EN: Declares or implements routines including `fill_n`, `isUndefined`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `fill_n`, `isUndefined`.
- CN: 这里声明或实现函数，例如 `fill_n`, `isUndefined`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `fill_n`, `isUndefined`。

### Lines 1272-1289

```cpp
      else
        remove = allBytes - curBytes;
      // If we can't satisfy this alignment, we've found a bad input.
      if (LLVM_UNLIKELY(static_cast<int32_t>(remove) < 0)) {
        Err(ctx) << getErrorLoc(ctx, (const uint8_t *)loc)
                 << "insufficient padding bytes for " << r.type << ": "
                 << allBytes << " bytes available for "
                 << "requested alignment of " << align << " bytes";
        remove = 0;
      }
      break;
    }
    case R_LARCH_PCALA_HI20:
    case R_LARCH_GOT_PC_HI20:
    case R_LARCH_TLS_GD_PC_HI20:
    case R_LARCH_TLS_LD_PC_HI20:
      // The overflow check for i+2 will be carried out in isPairRelaxable.
      if (isPairRelaxable(relocs, i))
```

- EN: Declares or implements routines including `Err`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `Err`.
- CN: 这里声明或实现函数，例如 `Err`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `Err`。

### Lines 1290-1307

```cpp
        relaxPCHi20Lo12(ctx, sec, i, loc, r, relocs[i + 2], remove);
      break;
    case R_LARCH_TLS_DESC_PC_HI20:
      if (r.expr == RE_LOONGARCH_GOT_PAGE_PC || r.expr == R_TPREL) {
        if (relaxable(relocs, i))
          remove = 4;
      } else if (isPairRelaxable(relocs, i))
        relaxPCHi20Lo12(ctx, sec, i, loc, r, relocs[i + 2], remove);
      break;
    case R_LARCH_CALL30:
    case R_LARCH_CALL36:
      if (relaxable(relocs, i))
        relaxMediumCall(ctx, sec, i, loc, r, remove);
      break;
    case R_LARCH_TLS_LE_HI20_R:
    case R_LARCH_TLS_LE_ADD_R:
    case R_LARCH_TLS_LE_LO12_R:
      if (relaxable(relocs, i))
```

- EN: Declares or implements routines including `relaxPCHi20Lo12`, `if`, `relaxMediumCall`. Notable symbols here include `relaxPCHi20Lo12`, `if`, `relaxMediumCall`.
- CN: 这里声明或实现函数，例如 `relaxPCHi20Lo12`, `if`, `relaxMediumCall`。这里较值得关注的符号包括 `relaxPCHi20Lo12`, `if`, `relaxMediumCall`。

### Lines 1308-1325

```cpp
        relaxTlsLe(ctx, sec, i, loc, r, remove);
      break;
    case R_LARCH_TLS_IE_PC_HI20:
      if (relaxable(relocs, i) && r.expr == R_TPREL &&
          isUInt<12>(r.sym->getVA(ctx, r.addend)))
        remove = 4;
      break;
    case R_LARCH_TLS_DESC_PC_LO12:
      if (relaxable(relocs, i) &&
          (r.expr == RE_LOONGARCH_GOT_PAGE_PC || r.expr == R_TPREL))
        remove = 4;
      break;
    case R_LARCH_TLS_DESC_LD:
      if (relaxable(relocs, i) && r.expr == R_TPREL &&
          isUInt<12>(r.sym->getVA(ctx, r.addend)))
        remove = 4;
      break;
    }
```

- EN: Declares or implements routines including `relaxTlsLe`. Notable symbols here include `relaxTlsLe`.
- CN: 这里声明或实现函数，例如 `relaxTlsLe`。这里较值得关注的符号包括 `relaxTlsLe`。

### Lines 1326-1342

```cpp

    // For all anchors whose offsets are <= r.offset, they are preceded by
    // the previous relocation whose `relocDeltas` value equals `delta`.
    // Decrease their st_value and update their st_size.
    for (; sa.size() && sa[0].offset <= r.offset; sa = sa.slice(1)) {
      if (sa[0].end)
        sa[0].d->size = sa[0].offset - delta - sa[0].d->value;
      else
        sa[0].d->value = sa[0].offset - delta;
    }
    delta += remove;
    if (delta != cur) {
      cur = delta;
      changed = true;
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1343-1355

```cpp
  for (const SymbolAnchor &a : sa) {
    if (a.end)
      a.d->size = a.offset - delta - a.d->value;
    else
      a.d->value = a.offset - delta;
  }
  // Inform assignAddresses that the size has changed.
  if (!isUInt<32>(delta))
    Fatal(ctx) << "section size decrease is too large: " << delta;
  sec.bytesDropped = delta;
  return changed;
}
```

- EN: Declares or implements routines including `Fatal`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `Fatal`.
- CN: 这里声明或实现函数，例如 `Fatal`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `Fatal`。

### Lines 1356-1370

```cpp
// Convert TLS IE to LE in the normal or medium code model.
// Original code sequence:
//  * pcalau12i $a0, %ie_pc_hi20(sym)
//  * ld.d      $a0, $a0, %ie_pc_lo12(sym)
//
// The code sequence converted is as follows:
//  * lu12i.w   $a0, %le_hi20(sym)      # le_hi20 != 0, otherwise NOP
//  * ori       $a0, src, %le_lo12(sym) # le_hi20 != 0, src = $a0,
//                                      # otherwise,    src = $zero
//
// When relaxation enables, redundant NOPs can be removed.
static void tlsIeToLe(uint8_t *loc, const Relocation &rel, uint64_t val) {
  assert(isInt<32>(val) &&
         "val exceeds the range of medium code model in tlsIeToLe");
```

- EN: Declares or implements routines including `tlsIeToLe`, `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `tlsIeToLe`, `assert`.
- CN: 这里声明或实现函数，例如 `tlsIeToLe`, `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `tlsIeToLe`, `assert`。

### Lines 1371-1388

```cpp
  bool isUInt12 = isUInt<12>(val);
  const uint32_t currInsn = read32le(loc);
  switch (rel.type) {
  case R_LARCH_TLS_IE_PC_HI20:
    if (isUInt12)
      write32le(loc, insn(ANDI, R_ZERO, R_ZERO, 0)); // nop
    else
      write32le(loc, insn(LU12I_W, getD5(currInsn), extractBits(val, 31, 12),
                          0)); // lu12i.w $a0, %le_hi20
    break;
  case R_LARCH_TLS_IE_PC_LO12:
    if (isUInt12)
      write32le(loc, insn(ORI, getD5(currInsn), R_ZERO,
                          val)); // ori $a0, $zero, %le_lo12
    else
      write32le(loc, insn(ORI, getD5(currInsn), getJ5(currInsn),
                          lo12(val))); // ori $a0, $a0, %le_lo12
    break;
```

- EN: Declares or implements routines including `read32le`, `write32le`, `lo12`. Notable symbols here include `read32le`, `write32le`, `lo12`.
- CN: 这里声明或实现函数，例如 `read32le`, `write32le`, `lo12`。这里较值得关注的符号包括 `read32le`, `write32le`, `lo12`。

### Lines 1389-1406

```cpp
  }
}

// Convert TLSDESC GD/LD to IE.
// In normal or medium code model, there are two forms of code sequences:
//  * pcalau12i  $a0, %desc_pc_hi20(sym_desc)
//  * addi.d     $a0, $a0, %desc_pc_lo12(sym_desc)
//  * ld.d       $ra, $a0, %desc_ld(sym_desc)
//  * jirl       $ra, $ra, %desc_call(sym_desc)
//  ------
//  * pcaddi $a0, %desc_pcrel_20(a)
//  * load $ra, $a0, %desc_ld(a)
//  * jirl $ra, $ra, %desc_call(a)
//
// The code sequence obtained is as follows:
//  * pcalau12i $a0, %ie_pc_hi20(sym_ie)
//  * ld.[wd]   $a0, $a0, %ie_pc_lo12(sym_ie)
//
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 1407-1424

```cpp
// Simplicity, whether tlsdescToIe or tlsdescToLe, we always tend to convert the
// preceding instructions to NOPs, due to both forms of code sequence
// (corresponding to relocation combinations:
// R_LARCH_TLS_DESC_PC_HI20+R_LARCH_TLS_DESC_PC_LO12 and
// R_LARCH_TLS_DESC_PCREL20_S2) have same process.
//
// When relaxation enables, redundant NOPs can be removed.
void LoongArch::tlsdescToIe(uint8_t *loc, const Relocation &rel,
                            uint64_t val) const {
  switch (rel.type) {
  case R_LARCH_TLS_DESC_PC_HI20:
  case R_LARCH_TLS_DESC_PC_LO12:
  case R_LARCH_TLS_DESC_PCREL20_S2:
    write32le(loc, insn(ANDI, R_ZERO, R_ZERO, 0)); // nop
    break;
  case R_LARCH_TLS_DESC_LD:
    write32le(loc, insn(PCALAU12I, R_A0, 0, 0)); // pcalau12i $a0, %ie_pc_hi20
    relocateNoSym(loc, R_LARCH_TLS_IE_PC_HI20, val);
```

- EN: Declares or implements routines including `write32le`, `relocateNoSym`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `write32le`, `relocateNoSym`.
- CN: 这里声明或实现函数，例如 `write32le`, `relocateNoSym`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `write32le`, `relocateNoSym`。

### Lines 1425-1435

```cpp
    break;
  case R_LARCH_TLS_DESC_CALL:
    write32le(loc, insn(ctx.arg.is64 ? LD_D : LD_W, R_A0, R_A0,
                        0)); // ld.[wd] $a0, $a0, %ie_pc_lo12
    relocateNoSym(loc, R_LARCH_TLS_IE_PC_LO12, val);
    break;
  default:
    llvm_unreachable("unsupported relocation for TLSDESC to IE");
  }
}
```

- EN: Declares or implements routines including `relocateNoSym`, `llvm_unreachable`. Notable symbols here include `relocateNoSym`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `relocateNoSym`, `llvm_unreachable`。这里较值得关注的符号包括 `relocateNoSym`, `llvm_unreachable`。

### Lines 1436-1446

```cpp
// Convert TLSDESC GD/LD to LE.
// The code sequence obtained in the normal or medium code model is as follows:
//  * lu12i.w   $a0, %le_hi20(sym)      # le_hi20 != 0, otherwise NOP
//  * ori       $a0, src, %le_lo12(sym) # le_hi20 != 0, src = $a0,
//                                      # otherwise,    src = $zero
// See the comment in tlsdescToIe for detailed information.
void LoongArch::tlsdescToLe(uint8_t *loc, const Relocation &rel,
                            uint64_t val) const {
  assert(isInt<32>(val) &&
         "val exceeds the range of medium code model in tlsdescToLe");
```

- EN: Declares or implements routines including `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`。

### Lines 1447-1464

```cpp
  bool isUInt12 = isUInt<12>(val);
  switch (rel.type) {
  case R_LARCH_TLS_DESC_PC_HI20:
  case R_LARCH_TLS_DESC_PC_LO12:
  case R_LARCH_TLS_DESC_PCREL20_S2:
    write32le(loc, insn(ANDI, R_ZERO, R_ZERO, 0)); // nop
    break;
  case R_LARCH_TLS_DESC_LD:
    if (isUInt12)
      write32le(loc, insn(ANDI, R_ZERO, R_ZERO, 0)); // nop
    else
      write32le(loc, insn(LU12I_W, R_A0, extractBits(val, 31, 12),
                          0)); // lu12i.w $a0, %le_hi20
    break;
  case R_LARCH_TLS_DESC_CALL:
    if (isUInt12)
      write32le(loc, insn(ORI, R_A0, R_ZERO, val)); // ori $a0, $zero, %le_lo12
    else
```

- EN: Declares or implements routines including `write32le`. Notable symbols here include `write32le`.
- CN: 这里声明或实现函数，例如 `write32le`。这里较值得关注的符号包括 `write32le`。

### Lines 1465-1472

```cpp
      write32le(loc,
                insn(ORI, R_A0, R_A0, lo12(val))); // ori $a0, $a0, %le_lo12
    break;
  default:
    llvm_unreachable("unsupported relocation for TLSDESC to LE");
  }
}
```

- EN: Declares or implements routines including `insn`, `llvm_unreachable`. Notable symbols here include `insn`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `insn`, `llvm_unreachable`。这里较值得关注的符号包括 `insn`, `llvm_unreachable`。

### Lines 1473-1489

```cpp
// Try GOT indirection to PC relative optimization.
// From:
//  * pcalau12i $a0, %got_pc_hi20(sym_got)
//  * ld.w/d    $a0, $a0, %got_pc_lo12(sym_got)
// To:
//  * pcalau12i $a0, %pc_hi20(sym)
//  * addi.w/d  $a0, $a0, %pc_lo12(sym)
//
// Note: Althouth the optimization has been performed, the GOT entries still
// exists, similarly to AArch64. Eliminating the entries will increase code
// complexity.
bool LoongArch::tryGotToPCRel(uint8_t *loc, const Relocation &rHi20,
                              const Relocation &rLo12, uint64_t secAddr) const {
  // Check if the relocations apply to consecutive instructions.
  if (rHi20.offset + 4 != rLo12.offset)
    return false;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1490-1501

```cpp
  // Check if the relocations reference the same symbol and skip undefined,
  // preemptible and STT_GNU_IFUNC symbols.
  if (!rHi20.sym || rHi20.sym != rLo12.sym || !rHi20.sym->isDefined() ||
      rHi20.sym->isPreemptible || rHi20.sym->isGnuIFunc())
    return false;

  // GOT references to absolute symbols can't be relaxed to use PCALAU12I/ADDI
  // in position-independent code because these instructions produce a relative
  // address.
  if ((ctx.arg.isPic && !cast<Defined>(*rHi20.sym).section))
    return false;
```

- EN: Declares or implements routines including `isGnuIFunc`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isGnuIFunc`.
- CN: 这里声明或实现函数，例如 `isGnuIFunc`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isGnuIFunc`。

### Lines 1502-1514

```cpp
  // Check if the addends of the both relocations are zero.
  if (rHi20.addend != 0 || rLo12.addend != 0)
    return false;

  const uint32_t currInsn = read32le(loc);
  const uint32_t nextInsn = read32le(loc + 4);
  const uint32_t ldOpcode = ctx.arg.is64 ? LD_D : LD_W;
  // Check if the first instruction is PCALAU12I and the second instruction is
  // LD.
  if ((currInsn & 0xfe000000) != PCALAU12I ||
      (nextInsn & 0xffc00000) != ldOpcode)
    return false;
```

- EN: Declares or implements routines including `read32le`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `read32le`.
- CN: 这里声明或实现函数，例如 `read32le`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `read32le`。

### Lines 1515-1528

```cpp
  // Check if use the same register.
  if (getD5(currInsn) != getJ5(nextInsn) || getJ5(nextInsn) != getD5(nextInsn))
    return false;

  Symbol &sym = *rHi20.sym;
  uint64_t symLocal = sym.getVA(ctx);
  const int64_t displace = symLocal - getLoongArchPage(secAddr + rHi20.offset);
  // Check if the symbol address is in
  // [(PC & ~0xfff) - 2GiB - 0x800, (PC & ~0xfff) + 2GiB - 0x800).
  const int64_t underflow = -0x80000000LL - 0x800;
  const int64_t overflow = 0x80000000LL - 0x800;
  if (!(displace >= underflow && displace < overflow))
    return false;
```

- EN: Declares or implements routines including `getLoongArchPage`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getLoongArchPage`.
- CN: 这里声明或实现函数，例如 `getLoongArchPage`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getLoongArchPage`。

### Lines 1529-1544

```cpp
  Relocation newRHi20 = {RE_LOONGARCH_PAGE_PC, R_LARCH_PCALA_HI20, rHi20.offset,
                         rHi20.addend, &sym};
  Relocation newRLo12 = {R_ABS, R_LARCH_PCALA_LO12, rLo12.offset, rLo12.addend,
                         &sym};
  uint64_t pageDelta =
      getLoongArchPageDelta(symLocal, secAddr + rHi20.offset, rHi20.type);
  // pcalau12i $a0, %pc_hi20
  write32le(loc, insn(PCALAU12I, getD5(currInsn), 0, 0));
  relocate(loc, newRHi20, pageDelta);
  // addi.w/d $a0, $a0, %pc_lo12
  write32le(loc + 4, insn(ctx.arg.is64 ? ADDI_D : ADDI_W, getD5(nextInsn),
                          getJ5(nextInsn), 0));
  relocate(loc + 4, newRLo12, SignExtend64(symLocal, 64));
  return true;
}
```

- EN: Declares or implements routines including `getLoongArchPageDelta`, `write32le`, `relocate`, `getJ5`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getLoongArchPageDelta`, `write32le`, `relocate`, `getJ5`.
- CN: 这里声明或实现函数，例如 `getLoongArchPageDelta`, `write32le`, `relocate`, `getJ5`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getLoongArchPageDelta`, `write32le`, `relocate`, `getJ5`。

### Lines 1545-1552

```cpp
// During TLSDESC to IE, the converted code sequence always includes an
// instruction related to the Lo12 relocation (ld.[wd]). To obtain correct val
// in `getRelocTargetVA`, expr of this instruction should be adjusted to R_GOT,
// while expr of other instructions related to the Hi20 relocation (pcalau12i)
// should be adjusted to RE_LOONGARCH_GOT_PAGE_PC. Specifically, in the normal
// or medium code model, the instruction with relocation R_LARCH_TLS_DESC_CALL
// is the candidate of Lo12 relocation.
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1553-1570

```cpp
static bool pairForGotRels(ArrayRef<Relocation> relocs) {
  // Check if R_LARCH_GOT_PC_HI20 and R_LARCH_GOT_PC_LO12 always appear in
  // pairs.
  size_t i = 0;
  const size_t size = relocs.size();
  for (; i != size; ++i) {
    if (relocs[i].type == R_LARCH_GOT_PC_HI20) {
      if (i + 1 < size && relocs[i + 1].type == R_LARCH_GOT_PC_LO12) {
        ++i;
        continue;
      }
      if (relaxable(relocs, i) && i + 2 < size &&
          relocs[i + 2].type == R_LARCH_GOT_PC_LO12) {
        i += 2;
        continue;
      }
      break;
    } else if (relocs[i].type == R_LARCH_GOT_PC_LO12) {
```

- EN: Declares or implements routines including `pairForGotRels`, `if`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `pairForGotRels`, `if`.
- CN: 这里声明或实现函数，例如 `pairForGotRels`, `if`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `pairForGotRels`, `if`。

### Lines 1571-1588

```cpp
      break;
    }
  }
  return i == size;
}

void LoongArch::relocateAlloc(InputSection &sec, uint8_t *buf) const {
  const unsigned bits = ctx.arg.is64 ? 64 : 32;
  uint64_t secAddr = sec.getOutputSection()->addr + sec.outSecOff;
  bool isExtreme = false;
  const MutableArrayRef<Relocation> relocs = sec.relocs();
  const bool isPairForGotRels = pairForGotRels(relocs);
  for (size_t i = 0, size = relocs.size(); i != size; ++i) {
    Relocation &rel = relocs[i];
    if (rel.expr == R_RELAX_HINT)
      continue;
    uint8_t *loc = buf + rel.offset;
    uint64_t val = SignExtend64(
```

- EN: Declares or implements routines including `relocateAlloc`, `pairForGotRels`. Notable symbols here include `relocateAlloc`, `pairForGotRels`.
- CN: 这里声明或实现函数，例如 `relocateAlloc`, `pairForGotRels`。这里较值得关注的符号包括 `relocateAlloc`, `pairForGotRels`。

### Lines 1589-1606

```cpp
        sec.getRelocTargetVA(ctx, rel, secAddr + rel.offset), bits);
    switch (rel.type) {
    case R_LARCH_TLS_IE_PC_HI20:
    case R_LARCH_TLS_IE_PC_LO12:
      // IE to LE. Not supported in extreme code model.
      if (rel.expr != R_TPREL)
        break;
      if (rel.type == R_LARCH_TLS_IE_PC_HI20)
        isExtreme =
            i + 2 < size && relocs[i + 2].type == R_LARCH_TLS_IE64_PC_LO20;
      if (isExtreme) {
        rel.expr = getRelExpr(rel.type, *rel.sym, loc);
        val = SignExtend64(sec.getRelocTargetVA(ctx, rel, secAddr + rel.offset),
                           bits);
        break;
      }
      if (relaxable(relocs, i) && rel.type == R_LARCH_TLS_IE_PC_HI20 &&
          isUInt<12>(val))
```

- EN: Declares or implements routines including `getRelExpr`, `SignExtend64`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getRelExpr`, `SignExtend64`.
- CN: 这里声明或实现函数，例如 `getRelExpr`, `SignExtend64`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getRelExpr`, `SignExtend64`。

### Lines 1607-1624

```cpp
        continue;
      tlsIeToLe(loc, rel, val);
      continue;

    case R_LARCH_TLS_DESC_PC_HI20:
    case R_LARCH_TLS_DESC_PC_LO12:
    case R_LARCH_TLS_DESC_LD:
    case R_LARCH_TLS_DESC_PCREL20_S2:
      // TLSDESC to LE/IE. Not supported in extreme code model.
      if (rel.expr != R_TPREL && rel.expr != RE_LOONGARCH_GOT_PAGE_PC)
        break;
      if (rel.type == R_LARCH_TLS_DESC_PC_HI20)
        isExtreme =
            i + 2 < size && relocs[i + 2].type == R_LARCH_TLS_DESC64_PC_LO20;
      if (isExtreme) {
        rel.expr = getRelExpr(rel.type, *rel.sym, loc);
        val = SignExtend64(sec.getRelocTargetVA(ctx, rel, secAddr + rel.offset),
                           bits);
```

- EN: Declares or implements routines including `tlsIeToLe`, `getRelExpr`, `SignExtend64`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `tlsIeToLe`, `getRelExpr`, `SignExtend64`.
- CN: 这里声明或实现函数，例如 `tlsIeToLe`, `getRelExpr`, `SignExtend64`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `tlsIeToLe`, `getRelExpr`, `SignExtend64`。

### Lines 1625-1639

```cpp
        break;
      }
      if (relaxable(relocs, i) && (rel.type == R_LARCH_TLS_DESC_PC_HI20 ||
                                   rel.type == R_LARCH_TLS_DESC_PC_LO12))
        continue;
      if (rel.expr == R_TPREL) {
        if (relaxable(relocs, i) && rel.type == R_LARCH_TLS_DESC_LD &&
            isUInt<12>(val))
          continue;
        tlsdescToLe(loc, rel, val);
      } else {
        tlsdescToIe(loc, rel, val);
      }
      continue;
```

- EN: Declares or implements routines including `tlsdescToLe`, `tlsdescToIe`. Notable symbols here include `tlsdescToLe`, `tlsdescToIe`.
- CN: 这里声明或实现函数，例如 `tlsdescToLe`, `tlsdescToIe`。这里较值得关注的符号包括 `tlsdescToLe`, `tlsdescToIe`。

### Lines 1640-1648

```cpp
    case R_LARCH_TLS_DESC_CALL:
      if (isExtreme)
        continue;
      if (rel.expr == R_TPREL)
        tlsdescToLe(loc, rel, val);
      else
        tlsdescToIe(loc, rel, val);
      continue;
```

- EN: Declares or implements routines including `tlsdescToLe`, `tlsdescToIe`. Notable symbols here include `tlsdescToLe`, `tlsdescToIe`.
- CN: 这里声明或实现函数，例如 `tlsdescToLe`, `tlsdescToIe`。这里较值得关注的符号包括 `tlsdescToLe`, `tlsdescToIe`。

### Lines 1649-1664

```cpp
    case R_LARCH_GOT_PC_HI20:
      // GOT indirection to PC relative optimization in normal or medium code
      // model, whether or not with R_LARCH_RELAX. If the code sequence can be
      // relaxed to a single pcaddi, the first instruction will be removed and
      // it will not reach here.
      if (isPairForGotRels) {
        bool isRelax = relaxable(relocs, i);
        const Relocation lo12Rel = isRelax ? relocs[i + 2] : relocs[i + 1];
        if (lo12Rel.type == R_LARCH_GOT_PC_LO12 &&
            tryGotToPCRel(loc, rel, lo12Rel, secAddr)) {
          i += isRelax ? 2 : 1;
          continue;
        }
      }
      break;
```

- EN: Declares or implements routines including `relaxable`, `tryGotToPCRel`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `relaxable`, `tryGotToPCRel`.
- CN: 这里声明或实现函数，例如 `relaxable`, `tryGotToPCRel`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `relaxable`, `tryGotToPCRel`。

### Lines 1665-1682

```cpp
    default:
      break;
    }
    relocate(loc, rel, val);
  }
}

// When relaxing just R_LARCH_ALIGN, relocDeltas is usually changed only once in
// the absence of a linker script. For call and load/store R_LARCH_RELAX, code
// shrinkage may reduce displacement and make more relocations eligible for
// relaxation. Code shrinkage may increase displacement to a call/load/store
// target at a higher fixed address, invalidating an earlier relaxation. Any
// change in section sizes can have cascading effect and require another
// relaxation pass.
bool LoongArch::relaxOnce(int pass) const {
  if (pass == 0)
    initSymbolAnchors(ctx);
```

- EN: Declares or implements routines including `relocate`, `relaxOnce`, `initSymbolAnchors`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `relocate`, `relaxOnce`, `initSymbolAnchors`.
- CN: 这里声明或实现函数，例如 `relocate`, `relaxOnce`, `initSymbolAnchors`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `relocate`, `relaxOnce`, `initSymbolAnchors`。

### Lines 1683-1694

```cpp
  SmallVector<InputSection *, 0> storage;
  bool changed = false;
  for (OutputSection *osec : ctx.outputSections) {
    if (!(osec->flags & SHF_EXECINSTR))
      continue;
    for (InputSection *sec : getInputSections(*osec, storage))
      if (sec->relaxAux)
        changed |= relax(ctx, *sec);
  }
  return changed;
}
```

- EN: Declares or implements routines including `relax`. Notable symbols here include `relax`.
- CN: 这里声明或实现函数，例如 `relax`。这里较值得关注的符号包括 `relax`。

### Lines 1695-1707

```cpp
void LoongArch::finalizeRelax(int passes) const {
  Log(ctx) << "relaxation passes: " << passes;
  SmallVector<InputSection *, 0> storage;
  for (OutputSection *osec : ctx.outputSections) {
    if (!(osec->flags & SHF_EXECINSTR))
      continue;
    for (InputSection *sec : getInputSections(*osec, storage)) {
      if (!sec->relaxAux)
        continue;
      RelaxAux &aux = *sec->relaxAux;
      if (!aux.relocDeltas)
        continue;
```

- EN: Declares or implements routines including `finalizeRelax`, `Log`. Notable symbols here include `finalizeRelax`, `Log`.
- CN: 这里声明或实现函数，例如 `finalizeRelax`, `Log`。这里较值得关注的符号包括 `finalizeRelax`, `Log`。

### Lines 1708-1718

```cpp
      MutableArrayRef<Relocation> rels = sec->relocs();
      ArrayRef<uint8_t> old = sec->content();
      size_t newSize = old.size() - aux.relocDeltas[rels.size() - 1];
      size_t writesIdx = 0;
      uint8_t *p = ctx.bAlloc.Allocate<uint8_t>(newSize);
      uint64_t offset = 0;
      int64_t delta = 0;
      sec->content_ = p;
      sec->size = newSize;
      sec->bytesDropped = 0;
```

- EN: Declares or implements routines including `relocs`, `content`. Notable symbols here include `relocs`, `content`.
- CN: 这里声明或实现函数，例如 `relocs`, `content`。这里较值得关注的符号包括 `relocs`, `content`。

### Lines 1719-1726

```cpp
      // Update section content: remove NOPs for R_LARCH_ALIGN and rewrite
      // instructions for relaxed relocations.
      for (size_t i = 0, e = rels.size(); i != e; ++i) {
        uint32_t remove = aux.relocDeltas[i] - delta;
        delta = aux.relocDeltas[i];
        if (remove == 0 && aux.relocTypes[i] == R_LARCH_NONE)
          continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1727-1744

```cpp
        // Copy from last location to the current relocated location.
        Relocation &r = rels[i];
        uint64_t size = r.offset - offset;
        memcpy(p, old.data() + offset, size);
        p += size;

        int64_t skip = 0;
        if (RelType newType = aux.relocTypes[i]) {
          switch (newType) {
          case R_LARCH_RELAX:
            break;
          case R_LARCH_PCREL20_S2:
            skip = 4;
            write32le(p, aux.writes[writesIdx++]);
            // RelExpr is needed for relocating.
            r.expr = r.sym->hasFlag(NEEDS_PLT) ? R_PLT_PC : R_PC;
            break;
          case R_LARCH_B26:
```

- EN: Declares or implements routines including `memcpy`, `write32le`, `hasFlag`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `memcpy`, `write32le`, `hasFlag`.
- CN: 这里声明或实现函数，例如 `memcpy`, `write32le`, `hasFlag`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `memcpy`, `write32le`, `hasFlag`。

### Lines 1745-1762

```cpp
          case R_LARCH_TLS_LE_LO12_R:
            skip = 4;
            write32le(p, aux.writes[writesIdx++]);
            break;
          case R_LARCH_TLS_GD_PCREL20_S2:
            // Note: R_LARCH_TLS_LD_PCREL20_S2 must also use R_TLSGD_PC instead
            // of R_TLSLD_PC due to historical reasons. In fact, right now TLSLD
            // behaves exactly like TLSGD on LoongArch.
            //
            // This reason has also been mentioned in mold commit:
            // https://github.com/rui314/mold/commit/5dfa1cf07c03bd57cb3d493b652ef22441bcd71c
          case R_LARCH_TLS_LD_PCREL20_S2:
            skip = 4;
            write32le(p, aux.writes[writesIdx++]);
            r.expr = R_TLSGD_PC;
            break;
          case R_LARCH_TLS_DESC_PCREL20_S2:
            skip = 4;
```

- EN: Declares or implements routines including `write32le`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `write32le`.
- CN: 这里声明或实现函数，例如 `write32le`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `write32le`。

### Lines 1763-1770

```cpp
            write32le(p, aux.writes[writesIdx++]);
            r.expr = R_TLSDESC_PC;
            break;
          default:
            llvm_unreachable("unsupported type");
          }
        }
```

- EN: Declares or implements routines including `write32le`, `llvm_unreachable`. Notable symbols here include `write32le`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `write32le`, `llvm_unreachable`。这里较值得关注的符号包括 `write32le`, `llvm_unreachable`。

### Lines 1771-1788

```cpp
        p += skip;
        offset = r.offset + skip + remove;
      }
      memcpy(p, old.data() + offset, old.size() - offset);

      // Subtract the previous relocDeltas value from the relocation offset.
      // For a pair of R_LARCH_XXX/R_LARCH_RELAX with the same offset, decrease
      // their r_offset by the same delta.
      delta = 0;
      for (size_t i = 0, e = rels.size(); i != e;) {
        uint64_t cur = rels[i].offset;
        do {
          rels[i].offset -= delta;
          if (aux.relocTypes[i] != R_LARCH_NONE)
            rels[i].type = aux.relocTypes[i];
        } while (++i != e && rels[i].offset == cur);
        delta = aux.relocDeltas[i - 1];
      }
```

- EN: Declares or implements routines including `memcpy`, `while`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `memcpy`, `while`.
- CN: 这里声明或实现函数，例如 `memcpy`, `while`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `memcpy`, `while`。

### Lines 1789-1795

```cpp
    }
  }
}

void elf::setLoongArchTargetInfo(Ctx &ctx) {
  ctx.target.reset(new LoongArch(ctx));
}
```

- EN: Declares or implements routines including `setLoongArchTargetInfo`. Notable symbols here include `setLoongArchTargetInfo`.
- CN: 这里声明或实现函数，例如 `setLoongArchTargetInfo`。这里较值得关注的符号包括 `setLoongArchTargetInfo`。

## Key Concepts / 关键概念

- `LoongArch`: class or struct interface / 类或结构体接口
- `ELFT`: class or struct interface / 类或结构体接口
- `RelTy`: class or struct interface / 类或结构体接口
- `Op`: enumeration of modes or states / 模式或状态枚举
- `Reg`: enumeration of modes or states / 模式或状态枚举
- `LoongArch`: function or method entry point / 函数或方法入口
- `calcEFlags`: function or method entry point / 函数或方法入口
- `getImplicitAddend`: function or method entry point / 函数或方法入口

## Dependencies / 依赖关系

- LLVM headers / LLVM 头文件: `llvm/BinaryFormat/ELF.h`, `llvm/Support/LEB128.h`
- System headers / 系统头文件: `InputFiles.h`, `OutputSections.h`, `RelocScan.h`, `Symbols.h`, `SyntheticSections.h`, `Target.h`
- Directory context / 目录上下文: `lld/ELF/Arch` neighbors usually cooperate with this file to provide the surrounding subsystem / `lld/ELF/Arch` 下的相邻文件通常与本文件协作组成对应子系统
