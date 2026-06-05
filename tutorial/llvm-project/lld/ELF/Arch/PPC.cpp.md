# PPC.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `lld/ELF/Arch/PPC.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: PPC.cpp. It also sits in code that implements ELF linking components in LLD. / 该文件实现 LLD 中的 ELF 链接组件。 源码头部说明其职责是：PPC.cpp。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
//===- PPC.cpp ------------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 9-16

```cpp
#include "InputFiles.h"
#include "OutputSections.h"
#include "RelocScan.h"
#include "Symbols.h"
#include "SyntheticSections.h"
#include "Target.h"
#include "Thunks.h"
```

- EN: Pulls in 7 header(s) from system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 7 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 17-25

```cpp
using namespace llvm;
using namespace llvm::support::endian;
using namespace llvm::ELF;
using namespace lld;
using namespace lld::elf;

// Undefine the macro predefined by GCC powerpc32.
#undef PPC
```

- EN: Works inside namespace scope `llvm`, `lld` to organize symbols. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `llvm`, `lld`.
- CN: 这里位于命名空间 `llvm`, `lld` 中，用于组织符号作用域。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `llvm`, `lld`。

### Lines 26-43

```cpp
namespace {
class PPC final : public TargetInfo {
public:
  PPC(Ctx &);
  void initTargetSpecificSections() override;
  RelExpr getRelExpr(RelType type, const Symbol &s,
                     const uint8_t *loc) const override;
  RelType getDynRel(RelType type) const override;
  int64_t getImplicitAddend(const uint8_t *buf, RelType type) const override;
  void writeGotHeader(uint8_t *buf) const override;
  void writePltHeader(uint8_t *buf) const override {
    llvm_unreachable("should call writePPC32GlinkSection() instead");
  }
  void writePlt(uint8_t *buf, const Symbol &sym,
                uint64_t pltEntryAddr) const override {
    llvm_unreachable("should call writePPC32GlinkSection() instead");
  }
  void writeIplt(uint8_t *buf, const Symbol &sym,
```

- EN: Introduces type definitions such as `PPC`. Declares or implements routines including `PPC`, `initTargetSpecificSections`, `getDynRel`, `getImplicitAddend`, `writeGotHeader`, and 2 more. Notable symbols here include `PPC`, `initTargetSpecificSections`, `getDynRel`, `getImplicitAddend`, `writeGotHeader`, `writePltHeader`.
- CN: 这里引入类型定义，例如 `PPC`。这里声明或实现函数，例如 `PPC`, `initTargetSpecificSections`, `getDynRel`, `getImplicitAddend`, `writeGotHeader`, and 2 more。这里较值得关注的符号包括 `PPC`, `initTargetSpecificSections`, `getDynRel`, `getImplicitAddend`, `writeGotHeader`, `writePltHeader`。

### Lines 44-61

```cpp
                 uint64_t pltEntryAddr) const override;
  void writeGotPlt(uint8_t *buf, const Symbol &s) const override;
  template <class ELFT, class RelTy>
  void scanSectionImpl(InputSectionBase &, Relocs<RelTy>);
  void scanSection(InputSectionBase &) override;
  bool needsThunk(RelExpr expr, RelType relocType, const InputFile *file,
                  uint64_t branchAddr, const Symbol &s,
                  int64_t a) const override;
  uint32_t getThunkSectionSpacing() const override;
  bool inBranchRange(RelType type, uint64_t src, uint64_t dst) const override;
  void relocate(uint8_t *loc, const Relocation &rel,
                uint64_t val) const override;
private:
  void relaxTlsGdToIe(uint8_t *loc, const Relocation &rel, uint64_t val) const;
  void relaxTlsGdToLe(uint8_t *loc, const Relocation &rel, uint64_t val) const;
  void relaxTlsLdToLe(uint8_t *loc, const Relocation &rel, uint64_t val) const;
  void relaxTlsIeToLe(uint8_t *loc, const Relocation &rel, uint64_t val) const;
};
```

- EN: Introduces type definitions such as `ELFT`, `RelTy`. Declares or implements routines including `writeGotPlt`, `scanSectionImpl`, `scanSection`, `getThunkSectionSpacing`, `inBranchRange`, and 4 more. Notable symbols here include `ELFT`, `RelTy`, `writeGotPlt`, `scanSectionImpl`, `scanSection`, `getThunkSectionSpacing`.
- CN: 这里引入类型定义，例如 `ELFT`, `RelTy`。这里声明或实现函数，例如 `writeGotPlt`, `scanSectionImpl`, `scanSection`, `getThunkSectionSpacing`, `inBranchRange`, and 4 more。这里较值得关注的符号包括 `ELFT`, `RelTy`, `writeGotPlt`, `scanSectionImpl`, `scanSection`, `getThunkSectionSpacing`。

### Lines 62-75

```cpp

// Used to compute outSecOff of .got2 in each object file. This is needed to
// synthesize PLT entries for PPC32 Secure PLT ABI.
struct Got2Section : SyntheticSection {
  Got2Section(Ctx &ctx)
      : SyntheticSection(ctx, ".got2", SHT_PROGBITS, SHF_ALLOC | SHF_WRITE, 4) {
  }
  bool isNeeded() const override;
  size_t getSize() const override { return 0; }
  void writeTo(uint8_t *buf) override {}
  void finalizeContents() override;
};
} // namespace
```

- EN: Introduces type definitions such as `Got2Section`. Declares or implements routines including `Got2Section`, `SyntheticSection`, `isNeeded`, `getSize`, `writeTo`, and 1 more. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `Got2Section`, `SyntheticSection`, `isNeeded`, `getSize`, `writeTo`, `finalizeContents`.
- CN: 这里引入类型定义，例如 `Got2Section`。这里声明或实现函数，例如 `Got2Section`, `SyntheticSection`, `isNeeded`, `getSize`, `writeTo`, and 1 more。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `Got2Section`, `SyntheticSection`, `isNeeded`, `getSize`, `writeTo`, `finalizeContents`。

### Lines 76-86

```cpp
static uint16_t lo(uint32_t v) { return v; }
static uint16_t ha(uint32_t v) { return (v + 0x8000) >> 16; }

static uint32_t readFromHalf16(Ctx &ctx, const uint8_t *loc) {
  return read32(ctx, ctx.arg.isLE ? loc : loc - 2);
}

static void writeFromHalf16(Ctx &ctx, uint8_t *loc, uint32_t insn) {
  write32(ctx, ctx.arg.isLE ? loc : loc - 2, insn);
}
```

- EN: Declares or implements routines including `lo`, `ha`, `readFromHalf16`, `writeFromHalf16`, `write32`. Notable symbols here include `lo`, `ha`, `readFromHalf16`, `writeFromHalf16`, `write32`.
- CN: 这里声明或实现函数，例如 `lo`, `ha`, `readFromHalf16`, `writeFromHalf16`, `write32`。这里较值得关注的符号包括 `lo`, `ha`, `readFromHalf16`, `writeFromHalf16`, `write32`。

### Lines 87-99

```cpp
void elf::writePPC32GlinkSection(Ctx &ctx, uint8_t *buf, size_t numEntries) {
  // Create canonical PLT entries for non-PIE code. Compilers don't generate
  // non-GOT-non-PLT relocations referencing external functions for -fpie/-fPIE.
  uint32_t glink = ctx.in.plt->getVA(); // VA of .glink
  if (!ctx.arg.isPic) {
    for (const Symbol *sym :
         cast<PPC32GlinkSection>(*ctx.in.plt).canonical_plts) {
      writePPC32PltCallStub(ctx, buf, sym->getGotPltVA(ctx), nullptr, 0);
      buf += 16;
      glink += 16;
    }
  }
```

- EN: Declares or implements routines including `writePPC32GlinkSection`, `getVA`, `writePPC32PltCallStub`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `writePPC32GlinkSection`, `getVA`, `writePPC32PltCallStub`.
- CN: 这里声明或实现函数，例如 `writePPC32GlinkSection`, `getVA`, `writePPC32PltCallStub`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `writePPC32GlinkSection`, `getVA`, `writePPC32PltCallStub`。

### Lines 100-108

```cpp
  // On PPC Secure PLT ABI, bl foo@plt jumps to a call stub, which loads an
  // absolute address from a specific .plt slot (usually called .got.plt on
  // other targets) and jumps there.
  //
  // a) With immediate binding (BIND_NOW), the .plt entry is resolved at load
  // time. The .glink section is not used.
  // b) With lazy binding, the .plt entry points to a `b PLTresolve`
  // instruction in .glink, filled in by PPC::writeGotPlt().
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 109-126

```cpp
  // Write N `b PLTresolve` first.
  for (size_t i = 0; i != numEntries; ++i)
    write32(ctx, buf + 4 * i, 0x48000000 | 4 * (numEntries - i));
  buf += 4 * numEntries;

  // Then write PLTresolve(), which has two forms: PIC and non-PIC. PLTresolve()
  // computes the PLT index (by computing the distance from the landing b to
  // itself) and calls _dl_runtime_resolve() (in glibc).
  uint32_t got = ctx.in.got->getVA();
  const uint8_t *end = buf + 64;
  if (ctx.arg.isPic) {
    uint32_t afterBcl = 4 * ctx.in.plt->getNumEntries() + 12;
    uint32_t gotBcl = got + 4 - (glink + afterBcl);
    write32(ctx, buf + 0,
            0x3d6b0000 | ha(afterBcl)); // addis r11,r11,1f-glink@ha
    write32(ctx, buf + 4, 0x7c0802a6);  // mflr r0
    write32(ctx, buf + 8, 0x429f0005);  // bcl 20,30,.+4
    write32(ctx, buf + 12,
```

- EN: Declares or implements routines including `write32`, `getVA`, `getNumEntries`, `ha`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `write32`, `getVA`, `getNumEntries`, `ha`.
- CN: 这里声明或实现函数，例如 `write32`, `getVA`, `getNumEntries`, `ha`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `write32`, `getVA`, `getNumEntries`, `ha`。

### Lines 127-144

```cpp
            0x396b0000 | lo(afterBcl)); // 1: addi r11,r11,1b-glink@l
    write32(ctx, buf + 16, 0x7d8802a6); // mflr r12
    write32(ctx, buf + 20, 0x7c0803a6); // mtlr r0
    write32(ctx, buf + 24, 0x7d6c5850); // sub r11,r11,r12
    write32(ctx, buf + 28, 0x3d8c0000 | ha(gotBcl)); // addis 12,12,GOT+4-1b@ha
    if (ha(gotBcl) == ha(gotBcl + 4)) {
      write32(ctx, buf + 32,
              0x800c0000 | lo(gotBcl)); // lwz r0,r12,GOT+4-1b@l(r12)
      write32(ctx, buf + 36,
              0x818c0000 | lo(gotBcl + 4)); // lwz r12,r12,GOT+8-1b@l(r12)
    } else {
      write32(ctx, buf + 32,
              0x840c0000 | lo(gotBcl));       // lwzu r0,r12,GOT+4-1b@l(r12)
      write32(ctx, buf + 36, 0x818c0000 | 4); // lwz r12,r12,4(r12)
    }
    write32(ctx, buf + 40, 0x7c0903a6); // mtctr 0
    write32(ctx, buf + 44, 0x7c0b5a14); // add r0,11,11
    write32(ctx, buf + 48, 0x7d605a14); // add r11,0,11
```

- EN: Declares or implements routines including `lo`, `write32`. Notable symbols here include `lo`, `write32`.
- CN: 这里声明或实现函数，例如 `lo`, `write32`。这里较值得关注的符号包括 `lo`, `write32`。

### Lines 145-162

```cpp
    write32(ctx, buf + 52, 0x4e800420); // bctr
    buf += 56;
  } else {
    write32(ctx, buf + 0, 0x3d800000 | ha(got + 4)); // lis     r12,GOT+4@ha
    write32(ctx, buf + 4, 0x3d6b0000 | ha(-glink)); // addis   r11,r11,-glink@ha
    if (ha(got + 4) == ha(got + 8))
      write32(ctx, buf + 8, 0x800c0000 | lo(got + 4)); // lwz r0,GOT+4@l(r12)
    else
      write32(ctx, buf + 8, 0x840c0000 | lo(got + 4)); // lwzu r0,GOT+4@l(r12)
    write32(ctx, buf + 12, 0x396b0000 | lo(-glink)); // addi    r11,r11,-glink@l
    write32(ctx, buf + 16, 0x7c0903a6);              // mtctr   r0
    write32(ctx, buf + 20, 0x7c0b5a14);              // add     r0,r11,r11
    if (ha(got + 4) == ha(got + 8))
      write32(ctx, buf + 24, 0x818c0000 | lo(got + 8)); // lwz r12,GOT+8@l(r12)
    else
      write32(ctx, buf + 24, 0x818c0000 | 4); // lwz r12,4(r12)
    write32(ctx, buf + 28, 0x7d605a14);       // add     r11,r0,r11
    write32(ctx, buf + 32, 0x4e800420);       // bctr
```

- EN: Declares or implements routines including `write32`. Notable symbols here include `write32`.
- CN: 这里声明或实现函数，例如 `write32`。这里较值得关注的符号包括 `write32`。

### Lines 163-170

```cpp
    buf += 36;
  }

  // Pad with nop. They should not be executed.
  for (; buf < end; buf += 4)
    write32(ctx, buf, 0x60000000);
}
```

- EN: Declares or implements routines including `write32`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `write32`.
- CN: 这里声明或实现函数，例如 `write32`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `write32`。

### Lines 171-183

```cpp
PPC::PPC(Ctx &ctx) : TargetInfo(ctx) {
  copyRel = R_PPC_COPY;
  gotRel = R_PPC_GLOB_DAT;
  pltRel = R_PPC_JMP_SLOT;
  relativeRel = R_PPC_RELATIVE;
  iRelativeRel = R_PPC_IRELATIVE;
  symbolicRel = R_PPC_ADDR32;
  gotHeaderEntriesNum = 3;
  gotPltHeaderEntriesNum = 0;
  pltHeaderSize = 0;
  pltEntrySize = 4;
  ipltEntrySize = 16;
```

- EN: Declares or implements routines including `PPC`. Notable symbols here include `PPC`.
- CN: 这里声明或实现函数，例如 `PPC`。这里较值得关注的符号包括 `PPC`。

### Lines 184-192

```cpp
  needsThunks = true;

  tlsModuleIndexRel = R_PPC_DTPMOD32;
  tlsOffsetRel = R_PPC_DTPREL32;
  tlsGotRel = R_PPC_TPREL32;

  defaultMaxPageSize = 65536;
  defaultImageBase = 0x10000000;
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 193-200

```cpp
  write32(ctx, trapInstr.data(), 0x7fe00008);
}

void PPC::initTargetSpecificSections() {
  ctx.in.ppc32Got2 = std::make_unique<Got2Section>(ctx);
  ctx.inputSections.push_back(ctx.in.ppc32Got2.get());
}
```

- EN: Declares or implements routines including `write32`, `initTargetSpecificSections`. Notable symbols here include `write32`, `initTargetSpecificSections`.
- CN: 这里声明或实现函数，例如 `write32`, `initTargetSpecificSections`。这里较值得关注的符号包括 `write32`, `initTargetSpecificSections`。

### Lines 201-214

```cpp
void PPC::writeIplt(uint8_t *buf, const Symbol &sym,
                    uint64_t /*pltEntryAddr*/) const {
  // In -pie or -shared mode, assume r30 points to .got2+0x8000, and use a
  // .got2.plt_pic32. thunk.
  writePPC32PltCallStub(ctx, buf, sym.getGotPltVA(ctx), sym.file, 0x8000);
}

void PPC::writeGotHeader(uint8_t *buf) const {
  // _GLOBAL_OFFSET_TABLE_[0] = _DYNAMIC
  // glibc stores _dl_runtime_resolve in _GLOBAL_OFFSET_TABLE_[1],
  // link_map in _GLOBAL_OFFSET_TABLE_[2].
  write32(ctx, buf, ctx.mainPart->dynamic->getVA());
}
```

- EN: Declares or implements routines including `writePPC32PltCallStub`, `writeGotHeader`, `write32`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `writePPC32PltCallStub`, `writeGotHeader`, `write32`.
- CN: 这里声明或实现函数，例如 `writePPC32PltCallStub`, `writeGotHeader`, `write32`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `writePPC32PltCallStub`, `writeGotHeader`, `write32`。

### Lines 215-231

```cpp
void PPC::writeGotPlt(uint8_t *buf, const Symbol &s) const {
  // Address of the symbol resolver stub in .glink .
  write32(ctx, buf,
          ctx.in.plt->getVA() + ctx.in.plt->headerSize + 4 * s.getPltIdx(ctx));
}

bool PPC::needsThunk(RelExpr expr, RelType type, const InputFile *file,
                     uint64_t branchAddr, const Symbol &s, int64_t a) const {
  if (type != R_PPC_LOCAL24PC && type != R_PPC_REL24 && type != R_PPC_PLTREL24)
    return false;
  if (s.isInPlt(ctx))
    return true;
  if (s.isUndefWeak())
    return false;
  return !PPC::inBranchRange(type, branchAddr, s.getVA(ctx, a));
}
```

- EN: Declares or implements routines including `writeGotPlt`, `getVA`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `writeGotPlt`, `getVA`.
- CN: 这里声明或实现函数，例如 `writeGotPlt`, `getVA`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `writeGotPlt`, `getVA`。

### Lines 232-240

```cpp
uint32_t PPC::getThunkSectionSpacing() const { return 0x2000000; }

bool PPC::inBranchRange(RelType type, uint64_t src, uint64_t dst) const {
  uint64_t offset = dst - src;
  if (type == R_PPC_LOCAL24PC || type == R_PPC_REL24 || type == R_PPC_PLTREL24)
    return isInt<26>(offset);
  llvm_unreachable("unsupported relocation type used in branch");
}
```

- EN: Declares or implements routines including `getThunkSectionSpacing`, `inBranchRange`, `llvm_unreachable`. Notable symbols here include `getThunkSectionSpacing`, `inBranchRange`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `getThunkSectionSpacing`, `inBranchRange`, `llvm_unreachable`。这里较值得关注的符号包括 `getThunkSectionSpacing`, `inBranchRange`, `llvm_unreachable`。

### Lines 241-258

```cpp
// Only needed to support relocations used by relocateNonAlloc and
// preprocessRelocs.
RelExpr PPC::getRelExpr(RelType type, const Symbol &s,
                        const uint8_t *loc) const {
  switch (type) {
  case R_PPC_NONE:
    return R_NONE;
  case R_PPC_ADDR32:
    return R_ABS;
  case R_PPC_DTPREL32:
    return R_DTPREL;
  case R_PPC_REL32:
    return R_PC;
  default:
    Err(ctx) << getErrorLoc(ctx, loc) << "unknown relocation (" << type.v
             << ") against symbol " << &s;
    return R_NONE;
  }
```

- EN: Declares or implements routines including `Err`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `Err`.
- CN: 这里声明或实现函数，例如 `Err`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `Err`。

### Lines 259-266

```cpp
}

RelType PPC::getDynRel(RelType type) const {
  if (type == R_PPC_ADDR32)
    return type;
  return R_PPC_NONE;
}
```

- EN: Declares or implements routines including `getDynRel`. Notable symbols here include `getDynRel`.
- CN: 这里声明或实现函数，例如 `getDynRel`。这里较值得关注的符号包括 `getDynRel`。

### Lines 267-284

```cpp
int64_t PPC::getImplicitAddend(const uint8_t *buf, RelType type) const {
  switch (type) {
  case R_PPC_NONE:
  case R_PPC_GLOB_DAT:
  case R_PPC_JMP_SLOT:
    return 0;
  case R_PPC_ADDR32:
  case R_PPC_REL32:
  case R_PPC_RELATIVE:
  case R_PPC_IRELATIVE:
  case R_PPC_DTPMOD32:
  case R_PPC_DTPREL32:
  case R_PPC_TPREL32:
    return SignExtend64<32>(read32(ctx, buf));
  default:
    InternalErr(ctx, buf) << "cannot read addend for relocation " << type;
    return 0;
  }
```

- EN: Declares or implements routines including `getImplicitAddend`, `InternalErr`. Notable symbols here include `getImplicitAddend`, `InternalErr`.
- CN: 这里声明或实现函数，例如 `getImplicitAddend`, `InternalErr`。这里较值得关注的符号包括 `getImplicitAddend`, `InternalErr`。

### Lines 285-302

```cpp
}

template <class ELFT, class RelTy>
void PPC::scanSectionImpl(InputSectionBase &sec, Relocs<RelTy> rels) {
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
```

- EN: Introduces type definitions such as `ELFT`, `RelTy`. Declares or implements routines including `scanSectionImpl`, `rs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `ELFT`, `RelTy`, `scanSectionImpl`, `rs`.
- CN: 这里引入类型定义，例如 `ELFT`, `RelTy`。这里声明或实现函数，例如 `scanSectionImpl`, `rs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `ELFT`, `RelTy`, `scanSectionImpl`, `rs`。

### Lines 303-316

```cpp
    // the switch to reach rs.process(). Types that need special handling
    // (fast-path helpers, TLS) call a handler and use `continue`.
    switch (type) {
    case R_PPC_NONE:
      continue;
    // Absolute relocations:
    case R_PPC_ADDR16_HA:
    case R_PPC_ADDR16_HI:
    case R_PPC_ADDR16_LO:
    case R_PPC_ADDR24:
    case R_PPC_ADDR32:
      expr = R_ABS;
      break;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 317-325

```cpp
    // PC-relative relocations:
    case R_PPC_REL14:
    case R_PPC_REL32:
    case R_PPC_REL16_LO:
    case R_PPC_REL16_HI:
    case R_PPC_REL16_HA:
      rs.processR_PC(type, offset, addend, sym);
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 326-343

```cpp
    // GOT-generating relocation:
    case R_PPC_GOT16:
      expr = R_GOT_OFF;
      break;

    // PLT-generating relocations:
    case R_PPC_LOCAL24PC:
    case R_PPC_REL24:
      rs.processR_PLT_PC(type, offset, addend, sym);
      continue;
    case R_PPC_PLTREL24:
      ctx.in.got->hasGotOffRel.store(true, std::memory_order_relaxed);
      if (LLVM_UNLIKELY(sym.isGnuIFunc())) {
        rs.process(RE_PPC32_PLTREL, type, offset, sym, addend);
      } else if (sym.isPreemptible) {
        sym.setFlags(NEEDS_PLT);
        sec.addReloc({RE_PPC32_PLTREL, type, offset, addend, &sym});
      } else {
```

- EN: Declares or implements routines including `if`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `if`.
- CN: 这里声明或实现函数，例如 `if`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `if`。

### Lines 344-352

```cpp
        // The 0x8000 bit of r_addend selects call stub type; mask it for direct
        // calls.
        addend &= ~0x8000;
        rs.processAux(R_PC, type, offset, sym, addend);
      }
      continue;

    // TLS relocations:
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 353-362

```cpp
    // TLS LE:
    case R_PPC_TPREL16:
    case R_PPC_TPREL16_HA:
    case R_PPC_TPREL16_LO:
    case R_PPC_TPREL16_HI:
      if (rs.checkTlsLe(offset, sym, type))
        continue;
      expr = R_TPREL;
      break;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 363-371

```cpp
    // TLS IE:
    case R_PPC_GOT_TPREL16:
      rs.handleTlsIe(R_GOT_OFF, type, offset, addend, sym);
      continue;
    case R_PPC_TLS:
      if (!ctx.arg.shared && !sym.isPreemptible)
        sec.addReloc({R_TPREL, type, offset, addend, &sym});
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 372-385

```cpp
    // TLS GD:
    case R_PPC_GOT_TLSGD16:
      rs.handleTlsGd(R_TLSGD_GOT, R_GOT_OFF, R_TPREL, type, offset, addend,
                     sym);
      continue;
    case R_PPC_TLSGD:
    case R_PPC_TLSLD:
      if (!ctx.arg.shared) {
        sec.addReloc({sym.isPreemptible ? R_GOT_OFF : R_TPREL, type, offset,
                      addend, &sym});
        ++it; // Skip REL24
      }
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 386-397

```cpp
    // TLS LD:
    case R_PPC_GOT_TLSLD16:
      rs.handleTlsLd(R_TLSLD_GOT, type, offset, addend, sym);
      continue;
    case R_PPC_DTPREL16:
    case R_PPC_DTPREL16_HA:
    case R_PPC_DTPREL16_HI:
    case R_PPC_DTPREL16_LO:
    case R_PPC_DTPREL32:
      sec.addReloc({R_DTPREL, type, offset, addend, &sym});
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 398-407

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

### Lines 408-425

```cpp
void PPC::scanSection(InputSectionBase &sec) {
  if (ctx.arg.isLE)
    elf::scanSection1<PPC, ELF32LE>(*this, sec);
  else
    elf::scanSection1<PPC, ELF32BE>(*this, sec);
}

static std::pair<RelType, uint64_t> fromDTPREL(RelType type, uint64_t val) {
  uint64_t dtpBiasedVal = val - 0x8000;
  switch (type) {
  case R_PPC_DTPREL16:
    return {R_PPC64_ADDR16, dtpBiasedVal};
  case R_PPC_DTPREL16_HA:
    return {R_PPC_ADDR16_HA, dtpBiasedVal};
  case R_PPC_DTPREL16_HI:
    return {R_PPC_ADDR16_HI, dtpBiasedVal};
  case R_PPC_DTPREL16_LO:
    return {R_PPC_ADDR16_LO, dtpBiasedVal};
```

- EN: Declares or implements routines including `scanSection`, `fromDTPREL`. Notable symbols here include `scanSection`, `fromDTPREL`.
- CN: 这里声明或实现函数，例如 `scanSection`, `fromDTPREL`。这里较值得关注的符号包括 `scanSection`, `fromDTPREL`。

### Lines 426-443

```cpp
  case R_PPC_DTPREL32:
    return {R_PPC_ADDR32, dtpBiasedVal};
  default:
    return {type, val};
  }
}

void PPC::relocate(uint8_t *loc, const Relocation &rel, uint64_t val) const {
  RelType newType;
  std::tie(newType, val) = fromDTPREL(rel.type, val);
  switch (newType) {
  case R_PPC_ADDR16:
    checkIntUInt(ctx, loc, val, 16, rel);
    write16(ctx, loc, val);
    break;
  case R_PPC_GOT16:
  case R_PPC_TPREL16:
    checkInt(ctx, loc, val, 16, rel);
```

- EN: Declares or implements routines including `relocate`, `tie`, `checkIntUInt`, `write16`, `checkInt`. Notable symbols here include `relocate`, `tie`, `checkIntUInt`, `write16`, `checkInt`.
- CN: 这里声明或实现函数，例如 `relocate`, `tie`, `checkIntUInt`, `write16`, `checkInt`。这里较值得关注的符号包括 `relocate`, `tie`, `checkIntUInt`, `write16`, `checkInt`。

### Lines 444-461

```cpp
    write16(ctx, loc, val);
    break;
  case R_PPC_GOT_TLSGD16:
    if (rel.expr == R_TPREL)
      relaxTlsGdToLe(loc, rel, val);
    else if (rel.expr == R_GOT_OFF)
      relaxTlsGdToIe(loc, rel, val);
    else {
      checkInt(ctx, loc, val, 16, rel);
      write16(ctx, loc, val);
    }
    break;
  case R_PPC_GOT_TLSLD16:
    if (rel.expr == R_TPREL)
      relaxTlsLdToLe(loc, rel, val);
    else {
      checkInt(ctx, loc, val, 16, rel);
      write16(ctx, loc, val);
```

- EN: Declares or implements routines including `write16`, `relaxTlsGdToLe`, `relaxTlsGdToIe`, `checkInt`, `relaxTlsLdToLe`. Notable symbols here include `write16`, `relaxTlsGdToLe`, `relaxTlsGdToIe`, `checkInt`, `relaxTlsLdToLe`.
- CN: 这里声明或实现函数，例如 `write16`, `relaxTlsGdToLe`, `relaxTlsGdToIe`, `checkInt`, `relaxTlsLdToLe`。这里较值得关注的符号包括 `write16`, `relaxTlsGdToLe`, `relaxTlsGdToIe`, `checkInt`, `relaxTlsLdToLe`。

### Lines 462-479

```cpp
    }
    break;
  case R_PPC_GOT_TPREL16:
    if (rel.expr == R_TPREL)
      relaxTlsIeToLe(loc, rel, val);
    else {
      checkInt(ctx, loc, val, 16, rel);
      write16(ctx, loc, val);
    }
    break;
  case R_PPC_ADDR16_HA:
  case R_PPC_DTPREL16_HA:
  case R_PPC_GOT_TLSGD16_HA:
  case R_PPC_GOT_TLSLD16_HA:
  case R_PPC_GOT_TPREL16_HA:
  case R_PPC_REL16_HA:
  case R_PPC_TPREL16_HA:
    write16(ctx, loc, ha(val));
```

- EN: Declares or implements routines including `relaxTlsIeToLe`, `checkInt`, `write16`. Notable symbols here include `relaxTlsIeToLe`, `checkInt`, `write16`.
- CN: 这里声明或实现函数，例如 `relaxTlsIeToLe`, `checkInt`, `write16`。这里较值得关注的符号包括 `relaxTlsIeToLe`, `checkInt`, `write16`。

### Lines 480-497

```cpp
    break;
  case R_PPC_ADDR16_HI:
  case R_PPC_DTPREL16_HI:
  case R_PPC_GOT_TLSGD16_HI:
  case R_PPC_GOT_TLSLD16_HI:
  case R_PPC_GOT_TPREL16_HI:
  case R_PPC_REL16_HI:
  case R_PPC_TPREL16_HI:
    write16(ctx, loc, val >> 16);
    break;
  case R_PPC_ADDR16_LO:
  case R_PPC_DTPREL16_LO:
  case R_PPC_GOT_TLSGD16_LO:
  case R_PPC_GOT_TLSLD16_LO:
  case R_PPC_GOT_TPREL16_LO:
  case R_PPC_REL16_LO:
  case R_PPC_TPREL16_LO:
    write16(ctx, loc, val);
```

- EN: Declares or implements routines including `write16`. Notable symbols here include `write16`.
- CN: 这里声明或实现函数，例如 `write16`。这里较值得关注的符号包括 `write16`。

### Lines 498-515

```cpp
    break;
  case R_PPC_ADDR32:
  case R_PPC_REL32:
    write32(ctx, loc, val);
    break;
  case R_PPC_REL14: {
    uint32_t mask = 0x0000FFFC;
    checkInt(ctx, loc, val, 16, rel);
    checkAlignment(ctx, loc, val, 4, rel);
    write32(ctx, loc, (read32(ctx, loc) & ~mask) | (val & mask));
    break;
  }
  case R_PPC_ADDR24:
  case R_PPC_REL24:
  case R_PPC_LOCAL24PC:
  case R_PPC_PLTREL24: {
    uint32_t mask = 0x03FFFFFC;
    checkInt(ctx, loc, val, 26, rel);
```

- EN: Declares or implements routines including `write32`, `checkInt`, `checkAlignment`. Notable symbols here include `write32`, `checkInt`, `checkAlignment`.
- CN: 这里声明或实现函数，例如 `write32`, `checkInt`, `checkAlignment`。这里较值得关注的符号包括 `write32`, `checkInt`, `checkAlignment`。

### Lines 516-533

```cpp
    checkAlignment(ctx, loc, val, 4, rel);
    write32(ctx, loc, (read32(ctx, loc) & ~mask) | (val & mask));
    break;
  }
  case R_PPC_TLSGD:
    if (rel.expr == R_TPREL)
      relaxTlsGdToLe(loc, rel, val);
    else if (rel.expr == R_GOT_OFF)
      relaxTlsGdToIe(loc, rel, val);
    break;
  case R_PPC_TLSLD:
    if (rel.expr == R_TPREL)
      relaxTlsLdToLe(loc, rel, val);
    break;
  case R_PPC_TLS:
    if (rel.expr == R_TPREL)
      relaxTlsIeToLe(loc, rel, val);
    break;
```

- EN: Declares or implements routines including `checkAlignment`, `write32`, `relaxTlsGdToLe`, `relaxTlsGdToIe`, `relaxTlsLdToLe`, and 1 more. Notable symbols here include `checkAlignment`, `write32`, `relaxTlsGdToLe`, `relaxTlsGdToIe`, `relaxTlsLdToLe`, `relaxTlsIeToLe`.
- CN: 这里声明或实现函数，例如 `checkAlignment`, `write32`, `relaxTlsGdToLe`, `relaxTlsGdToIe`, `relaxTlsLdToLe`, and 1 more。这里较值得关注的符号包括 `checkAlignment`, `write32`, `relaxTlsGdToLe`, `relaxTlsGdToIe`, `relaxTlsLdToLe`, `relaxTlsIeToLe`。

### Lines 534-551

```cpp
  default:
    llvm_unreachable("unknown relocation");
  }
}

void PPC::relaxTlsGdToIe(uint8_t *loc, const Relocation &rel,
                         uint64_t val) const {
  switch (rel.type) {
  case R_PPC_GOT_TLSGD16: {
    // addi rT, rA, x@got@tlsgd --> lwz rT, x@got@tprel(rA)
    uint32_t insn = readFromHalf16(ctx, loc);
    writeFromHalf16(ctx, loc, 0x80000000 | (insn & 0x03ff0000));
    relocateNoSym(loc, R_PPC_GOT_TPREL16, val);
    break;
  }
  case R_PPC_TLSGD:
    // bl __tls_get_addr(x@tldgd) --> add r3, r3, r2
    write32(ctx, loc, 0x7c631214);
```

- EN: Declares or implements routines including `llvm_unreachable`, `readFromHalf16`, `writeFromHalf16`, `relocateNoSym`, `write32`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `llvm_unreachable`, `readFromHalf16`, `writeFromHalf16`, `relocateNoSym`, `write32`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`, `readFromHalf16`, `writeFromHalf16`, `relocateNoSym`, `write32`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `llvm_unreachable`, `readFromHalf16`, `writeFromHalf16`, `relocateNoSym`, `write32`。

### Lines 552-569

```cpp
    break;
  default:
    llvm_unreachable("unsupported relocation for TLS GD to IE relaxation");
  }
}

void PPC::relaxTlsGdToLe(uint8_t *loc, const Relocation &rel,
                         uint64_t val) const {
  switch (rel.type) {
  case R_PPC_GOT_TLSGD16:
    // addi r3, r31, x@got@tlsgd --> addis r3, r2, x@tprel@ha
    writeFromHalf16(ctx, loc, 0x3c620000 | ha(val));
    break;
  case R_PPC_TLSGD:
    // bl __tls_get_addr(x@tldgd) --> add r3, r3, x@tprel@l
    write32(ctx, loc, 0x38630000 | lo(val));
    break;
  default:
```

- EN: Declares or implements routines including `llvm_unreachable`, `writeFromHalf16`, `write32`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `llvm_unreachable`, `writeFromHalf16`, `write32`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`, `writeFromHalf16`, `write32`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `llvm_unreachable`, `writeFromHalf16`, `write32`。

### Lines 570-587

```cpp
    llvm_unreachable("unsupported relocation for TLS GD to LE relaxation");
  }
}

void PPC::relaxTlsLdToLe(uint8_t *loc, const Relocation &rel,
                         uint64_t val) const {
  switch (rel.type) {
  case R_PPC_GOT_TLSLD16:
    // addi r3, rA, x@got@tlsgd --> addis r3, r2, 0
    writeFromHalf16(ctx, loc, 0x3c620000);
    break;
  case R_PPC_TLSLD:
    // r3+x@dtprel computes r3+x-0x8000, while we want it to compute r3+x@tprel
    // = r3+x-0x7000, so add 4096 to r3.
    // bl __tls_get_addr(x@tlsld) --> addi r3, r3, 4096
    write32(ctx, loc, 0x38631000);
    break;
  default:
```

- EN: Declares or implements routines including `llvm_unreachable`, `writeFromHalf16`, `write32`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `llvm_unreachable`, `writeFromHalf16`, `write32`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`, `writeFromHalf16`, `write32`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `llvm_unreachable`, `writeFromHalf16`, `write32`。

### Lines 588-605

```cpp
    llvm_unreachable("unsupported relocation for TLS LD to LE relaxation");
  }
}

void PPC::relaxTlsIeToLe(uint8_t *loc, const Relocation &rel,
                         uint64_t val) const {
  switch (rel.type) {
  case R_PPC_GOT_TPREL16: {
    // lwz rT, x@got@tprel(rA) --> addis rT, r2, x@tprel@ha
    uint32_t rt = readFromHalf16(ctx, loc) & 0x03e00000;
    writeFromHalf16(ctx, loc, 0x3c020000 | rt | ha(val));
    break;
  }
  case R_PPC_TLS: {
    uint32_t insn = read32(ctx, loc);
    if (insn >> 26 != 31)
      ErrAlways(ctx) << "unrecognized instruction for IE to LE R_PPC_TLS";
    // addi rT, rT, x@tls --> addi rT, rT, x@tprel@l
```

- EN: Declares or implements routines including `llvm_unreachable`, `readFromHalf16`, `writeFromHalf16`, `read32`, `ErrAlways`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `llvm_unreachable`, `readFromHalf16`, `writeFromHalf16`, `read32`, `ErrAlways`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`, `readFromHalf16`, `writeFromHalf16`, `read32`, `ErrAlways`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `llvm_unreachable`, `readFromHalf16`, `writeFromHalf16`, `read32`, `ErrAlways`。

### Lines 606-620

```cpp
    unsigned secondaryOp = (read32(ctx, loc) & 0x000007fe) >> 1;
    uint32_t dFormOp = getPPCDFormOp(secondaryOp);
    if (dFormOp == 0) { // Expecting a DS-Form instruction.
      dFormOp = getPPCDSFormOp(secondaryOp);
      if (dFormOp == 0)
        ErrAlways(ctx) << "unrecognized instruction for IE to LE R_PPC_TLS";
    }
    write32(ctx, loc, (dFormOp | (insn & 0x03ff0000) | lo(val)));
    break;
  }
  default:
    llvm_unreachable("unsupported relocation for TLS IE to LE relaxation");
  }
}
```

- EN: Declares or implements routines including `getPPCDFormOp`, `getPPCDSFormOp`, `ErrAlways`, `write32`, `llvm_unreachable`. Notable symbols here include `getPPCDFormOp`, `getPPCDSFormOp`, `ErrAlways`, `write32`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `getPPCDFormOp`, `getPPCDSFormOp`, `ErrAlways`, `write32`, `llvm_unreachable`。这里较值得关注的符号包括 `getPPCDFormOp`, `getPPCDSFormOp`, `ErrAlways`, `write32`, `llvm_unreachable`。

### Lines 621-631

```cpp
void elf::setPPCTargetInfo(Ctx &ctx) { ctx.target.reset(new PPC(ctx)); }

bool Got2Section::isNeeded() const {
  for (SectionCommand *cmd : getParent()->commands)
    if (auto *isd = dyn_cast<InputSectionDescription>(cmd))
      for (InputSection *isec : isd->sections)
        if (isec != this)
          return true;
  return false;
}
```

- EN: Declares or implements routines including `setPPCTargetInfo`, `isNeeded`. Notable symbols here include `setPPCTargetInfo`, `isNeeded`.
- CN: 这里声明或实现函数，例如 `setPPCTargetInfo`, `isNeeded`。这里较值得关注的符号包括 `setPPCTargetInfo`, `isNeeded`。

### Lines 632-645

```cpp
void Got2Section::finalizeContents() {
  // PPC32 may create multiple GOT sections for -fPIC/-fPIE, one per file in
  // .got2 . This function computes outSecOff of each .got2 to be used in
  // PPC32PltCallStub::writeTo(). The purpose of this empty synthetic section is
  // to collect input sections named ".got2".
  for (SectionCommand *cmd : getParent()->commands)
    if (auto *isd = dyn_cast<InputSectionDescription>(cmd)) {
      for (InputSection *isec : isd->sections) {
        // isec->file may be nullptr for MergeSyntheticSection.
        if (isec != this && isec->file)
          isec->file->ppc32Got2 = isec;
      }
    }
}
```

- EN: Declares or implements routines including `finalizeContents`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `finalizeContents`.
- CN: 这里声明或实现函数，例如 `finalizeContents`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `finalizeContents`。

## Key Concepts / 关键概念

- `PPC`: class or struct interface / 类或结构体接口
- `ELFT`: class or struct interface / 类或结构体接口
- `RelTy`: class or struct interface / 类或结构体接口
- `Got2Section`: class or struct interface / 类或结构体接口
- `PPC`: function or method entry point / 函数或方法入口
- `initTargetSpecificSections`: function or method entry point / 函数或方法入口
- `getDynRel`: function or method entry point / 函数或方法入口
- `getImplicitAddend`: function or method entry point / 函数或方法入口

## Dependencies / 依赖关系

- System headers / 系统头文件: `InputFiles.h`, `OutputSections.h`, `RelocScan.h`, `Symbols.h`, `SyntheticSections.h`, `Target.h`, `Thunks.h`
- Directory context / 目录上下文: `lld/ELF/Arch` neighbors usually cooperate with this file to provide the surrounding subsystem / `lld/ELF/Arch` 下的相邻文件通常与本文件协作组成对应子系统
