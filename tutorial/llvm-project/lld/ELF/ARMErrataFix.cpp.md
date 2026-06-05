# ARMErrataFix.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `lld/ELF/ARMErrataFix.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: ARMErrataFix.cpp This file implements Section Patching for the purpose of working around the Cortex-a8 erratum 657417 "A 32bit branch instruction that spans 2 4K regions can result in an incorrect instruction fetch or processor deadlock." The. It also sits in code that implements ELF linking components in LLD. / 该文件实现 LLD 中的 ELF 链接组件。 源码头部说明其职责是：ARMErrataFix.cpp This file implements Section Patching for the purpose of working around the Cortex-a8 erratum 657417 "A 32bit branch instruction that spans 2 4K regions can result in an incorrect instruction fetch or processor deadlock." The。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15

```cpp
//===- ARMErrataFix.cpp ---------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
// This file implements Section Patching for the purpose of working around the
// Cortex-a8 erratum 657417 "A 32bit branch instruction that spans 2 4K regions
// can result in an incorrect instruction fetch or processor deadlock." The
// erratum affects all but r1p7, r2p5, r2p6, r3p1 and r3p2 revisions of the
// Cortex-A8. A high level description of the patching technique is given in
// the opening comment of AArch64ErrataFix.cpp.
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 16-26

```cpp
#include "ARMErrataFix.h"
#include "InputFiles.h"
#include "LinkerScript.h"
#include "OutputSections.h"
#include "Relocations.h"
#include "Symbols.h"
#include "SyntheticSections.h"
#include "Target.h"
#include "llvm/Support/Endian.h"
#include <algorithm>
```

- EN: Pulls in 10 header(s) from LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 10 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 27-34

```cpp
using namespace llvm;
using namespace llvm::ELF;
using namespace llvm::object;
using namespace llvm::support;
using namespace llvm::support::endian;
using namespace lld;
using namespace lld::elf;
```

- EN: Works inside namespace scope `llvm`, `lld` to organize symbols. Notable symbols here include `llvm`, `lld`.
- CN: 这里位于命名空间 `llvm`, `lld` 中，用于组织符号作用域。这里较值得关注的符号包括 `llvm`, `lld`。

### Lines 35-52

```cpp
// The documented title for Erratum 657417 is:
// "A 32bit branch instruction that spans two 4K regions can result in an
// incorrect instruction fetch or processor deadlock". Graphically using a
// 32-bit B.w instruction encoded as a pair of halfwords 0xf7fe 0xbfff
// xxxxxx000 // Memory region 1 start
// target:
// ...
// xxxxxxffe f7fe // First halfword of branch to target:
// xxxxxx000 // Memory region 2 start
// xxxxxx002 bfff // Second halfword of branch to target:
//
// The specific trigger conditions that can be detected at link time are:
// - There is a 32-bit Thumb-2 branch instruction with an address of the form
//   xxxxxxFFE. The first 2 bytes of the instruction are in 4KiB region 1, the
//   second 2 bytes are in region 2.
// - The branch instruction is one of BLX, BL, B.w BCC.w
// - The instruction preceding the branch is a 32-bit non-branch instruction.
// - The target of the branch is in region 1.
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 53-68

```cpp
//
// The linker mitigation for the fix is to redirect any branch that meets the
// erratum conditions to a patch section containing a branch to the target.
//
// As adding patch sections may move branches onto region boundaries the patch
// must iterate until no more patches are added.
//
// Example, before:
// 00000FFA func: NOP.w      // 32-bit Thumb function
// 00000FFE       B.W func   // 32-bit branch spanning 2 regions, dest in 1st.
// Example, after:
// 00000FFA func: NOP.w      // 32-bit Thumb function
// 00000FFE       B.w __CortexA8657417_00000FFE
// 00001002       2 - bytes padding
// 00001004 __CortexA8657417_00000FFE: B.w func
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 69-77

```cpp
class elf::Patch657417Section final : public SyntheticSection {
public:
  Patch657417Section(Ctx &, InputSection *p, uint64_t off, uint32_t instr,
                     bool isARM);

  void writeTo(uint8_t *buf) override;

  size_t getSize() const override { return 4; }
```

- EN: Introduces type definitions such as `elf`. Declares or implements routines including `writeTo`, `getSize`. Notable symbols here include `elf`, `writeTo`, `getSize`.
- CN: 这里引入类型定义，例如 `elf`。这里声明或实现函数，例如 `writeTo`, `getSize`。这里较值得关注的符号包括 `elf`, `writeTo`, `getSize`。

### Lines 78-95

```cpp
  // Get the virtual address of the branch instruction at patcheeOffset.
  uint64_t getBranchAddr() const;

  static bool classof(const SectionBase *d) {
    return d->kind() == InputSectionBase::Synthetic && d->name ==".text.patch";
  }

  // The Section we are patching.
  const InputSection *patchee;
  // The offset of the instruction in the Patchee section we are patching.
  uint64_t patcheeOffset;
  // A label for the start of the Patch that we can use as a relocation target.
  Symbol *patchSym;
  // A decoding of the branch instruction at patcheeOffset.
  uint32_t instr;
  // True If the patch is to be written in ARM state, otherwise the patch will
  // be written in Thumb state.
  bool isARM;
```

- EN: Declares or implements routines including `getBranchAddr`, `classof`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getBranchAddr`, `classof`.
- CN: 这里声明或实现函数，例如 `getBranchAddr`, `classof`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getBranchAddr`, `classof`。

### Lines 96-110

```cpp
};

// Return true if the half-word, when taken as the first of a pair of halfwords
// is the first half of a 32-bit instruction.
// Reference from ARM Architecture Reference Manual ARMv7-A and ARMv7-R edition
// section A6.3: 32-bit Thumb instruction encoding
// |             HW1                   |               HW2                |
// | 1 1 1 | op1 (2) | op2 (7) | x (4) |op|           x (15)              |
// With op1 == 0b00, a 16-bit instruction is encoded.
//
// We test only the first halfword, looking for op != 0b00.
static bool is32bitInstruction(uint16_t hw) {
  return (hw & 0xe000) == 0xe000 && (hw & 0x1800) != 0x0000;
}
```

- EN: Declares or implements routines including `is32bitInstruction`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `is32bitInstruction`.
- CN: 这里声明或实现函数，例如 `is32bitInstruction`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `is32bitInstruction`。

### Lines 111-119

```cpp
// Reference from ARM Architecture Reference Manual ARMv7-A and ARMv7-R edition
// section A6.3.4 Branches and miscellaneous control.
// |             HW1              |               HW2                |
// | 1 1 1 | 1 0 | op (7) | x (4) | 1 | op1 (3) | op2 (4) | imm8 (8) |
// op1 == 0x0 op != x111xxx | Conditional branch (Bcc.W)
// op1 == 0x1               | Branch (B.W)
// op1 == 1x0               | Branch with Link and Exchange (BLX.w)
// op1 == 1x1               | Branch with Link (BL.W)
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 120-128

```cpp
static bool isBcc(uint32_t instr) {
  return (instr & 0xf800d000) == 0xf0008000 &&
         (instr & 0x03800000) != 0x03800000;
}

static bool isB(uint32_t instr) { return (instr & 0xf800d000) == 0xf0009000; }

static bool isBLX(uint32_t instr) { return (instr & 0xf800d000) == 0xf000c000; }
```

- EN: Declares or implements routines including `isBcc`, `isB`, `isBLX`. Notable symbols here include `isBcc`, `isB`, `isBLX`.
- CN: 这里声明或实现函数，例如 `isBcc`, `isB`, `isBLX`。这里较值得关注的符号包括 `isBcc`, `isB`, `isBLX`。

### Lines 129-146

```cpp
static bool isBL(uint32_t instr) { return (instr & 0xf800d000) == 0xf000d000; }

static bool is32bitBranch(uint32_t instr) {
  return isBcc(instr) || isB(instr) || isBL(instr) || isBLX(instr);
}

Patch657417Section::Patch657417Section(Ctx &ctx, InputSection *p, uint64_t off,
                                       uint32_t instr, bool isARM)
    : SyntheticSection(ctx, ".text.patch", SHT_PROGBITS,
                       SHF_ALLOC | SHF_EXECINSTR, 4),
      patchee(p), patcheeOffset(off), instr(instr), isARM(isARM) {
  parent = p->getParent();
  patchSym = addSyntheticLocal(
      ctx, ctx.saver.save("__CortexA8657417_" + utohexstr(getBranchAddr())),
      STT_FUNC, isARM ? 0 : 1, getSize(), *this);
  addSyntheticLocal(ctx, ctx.saver.save(isARM ? "$a" : "$t"), STT_NOTYPE, 0, 0,
                    *this);
}
```

- EN: Declares or implements routines including `isBL`, `is32bitBranch`, `patchee`, `getParent`, `getSize`, and 1 more. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isBL`, `is32bitBranch`, `patchee`, `getParent`, `getSize`, `addSyntheticLocal`.
- CN: 这里声明或实现函数，例如 `isBL`, `is32bitBranch`, `patchee`, `getParent`, `getSize`, and 1 more。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isBL`, `is32bitBranch`, `patchee`, `getParent`, `getSize`, `addSyntheticLocal`。

### Lines 147-164

```cpp

uint64_t Patch657417Section::getBranchAddr() const {
  return patchee->getVA(patcheeOffset);
}

// Given a branch instruction instr at sourceAddr work out its destination
// address. This is only used when the branch instruction has no relocation.
static uint64_t getThumbDestAddr(Ctx &ctx, uint64_t sourceAddr,
                                 uint32_t instr) {
  uint8_t buf[4];
  write16le(buf, instr >> 16);
  write16le(buf + 2, instr & 0x0000ffff);
  int64_t offset;
  if (isBcc(instr))
    offset = ctx.target->getImplicitAddend(buf, R_ARM_THM_JUMP19);
  else if (isB(instr))
    offset = ctx.target->getImplicitAddend(buf, R_ARM_THM_JUMP24);
  else
```

- EN: Declares or implements routines including `getBranchAddr`, `write16le`, `getImplicitAddend`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getBranchAddr`, `write16le`, `getImplicitAddend`.
- CN: 这里声明或实现函数，例如 `getBranchAddr`, `write16le`, `getImplicitAddend`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getBranchAddr`, `write16le`, `getImplicitAddend`。

### Lines 165-177

```cpp
    offset = ctx.target->getImplicitAddend(buf, R_ARM_THM_CALL);
  // A BLX instruction from Thumb to Arm may have an address that is
  // not 4-byte aligned. As Arm instructions are always 4-byte aligned
  // the instruction is calculated (from Arm ARM):
  // targetAddress = Align(PC, 4) + imm32
  // where
  //   Align(x, y) = y * (x Div y)
  // which corresponds to alignDown.
  if (isBLX(instr))
    sourceAddr = alignDown(sourceAddr, 4);
  return sourceAddr + offset + 4;
}
```

- EN: Declares or implements routines including `getImplicitAddend`, `alignDown`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getImplicitAddend`, `alignDown`.
- CN: 这里声明或实现函数，例如 `getImplicitAddend`, `alignDown`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getImplicitAddend`, `alignDown`。

### Lines 178-189

```cpp
void Patch657417Section::writeTo(uint8_t *buf) {
  // The base instruction of the patch is always a 32-bit unconditional branch.
  if (isARM)
    write32le(buf, 0xea000000);
  else
    write32le(buf, 0x9000f000);
  // If we have a relocation then apply it.
  if (!relocs().empty()) {
    ctx.target->relocateAlloc(*this, buf);
    return;
  }
```

- EN: Declares or implements routines including `writeTo`, `write32le`, `relocateAlloc`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `writeTo`, `write32le`, `relocateAlloc`.
- CN: 这里声明或实现函数，例如 `writeTo`, `write32le`, `relocateAlloc`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `writeTo`, `write32le`, `relocateAlloc`。

### Lines 190-204

```cpp
  // If we don't have a relocation then we must calculate and write the offset
  // ourselves.
  // Get the destination offset from the addend in the branch instruction.
  // We cannot use the instruction in the patchee section as this will have
  // been altered to point to us!
  uint64_t s = getThumbDestAddr(ctx, getBranchAddr(), instr);
  // A BLX changes the state of the branch in the patch to Arm state, which
  // has a PC Bias of 8, whereas in all other cases the branch is in Thumb
  // state with a PC Bias of 4.
  uint64_t pcBias = isBLX(instr) ? 8 : 4;
  uint64_t p = getVA(pcBias);
  ctx.target->relocateNoSym(buf, isARM ? R_ARM_JUMP24 : R_ARM_THM_JUMP24,
                            s - p);
}
```

- EN: Declares or implements routines including `getThumbDestAddr`, `isBLX`, `getVA`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getThumbDestAddr`, `isBLX`, `getVA`.
- CN: 这里声明或实现函数，例如 `getThumbDestAddr`, `isBLX`, `getVA`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getThumbDestAddr`, `isBLX`, `getVA`。

### Lines 205-222

```cpp
// Given a branch instruction spanning two 4KiB regions, at offset off from the
// start of isec, return true if the destination of the branch is within the
// first of the two 4Kib regions.
static bool branchDestInFirstRegion(Ctx &ctx, const InputSection *isec,
                                    uint64_t off, uint32_t instr,
                                    const Relocation *r) {
  uint64_t sourceAddr = isec->getVA(0) + off;
  assert((sourceAddr & 0xfff) == 0xffe);
  uint64_t destAddr;
  // If there is a branch relocation at the same offset we must use this to
  // find the destination address as the branch could be indirected via a thunk
  // or the PLT.
  if (r) {
    uint64_t dst =
        r->expr == R_PLT_PC ? r->sym->getPltVA(ctx) : r->sym->getVA(ctx);
    // Account for Thumb PC bias, usually cancelled to 0 by addend of -4.
    destAddr = dst + r->addend + 4;
  } else {
```

- EN: Declares or implements routines including `getVA`, `assert`, `getPltVA`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getVA`, `assert`, `getPltVA`.
- CN: 这里声明或实现函数，例如 `getVA`, `assert`, `getPltVA`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getVA`, `assert`, `getPltVA`。

### Lines 223-230

```cpp
    // If there is no relocation, we must have an intra-section branch
    // We must extract the offset from the addend manually.
    destAddr = getThumbDestAddr(ctx, sourceAddr, instr);
  }

  return (destAddr & 0xfffff000) == (sourceAddr & 0xfffff000);
}
```

- EN: Declares or implements routines including `getThumbDestAddr`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getThumbDestAddr`.
- CN: 这里声明或实现函数，例如 `getThumbDestAddr`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getThumbDestAddr`。

### Lines 231-244

```cpp
// Return true if a branch can reach a patch section placed after isec.
// The Bcc.w instruction has a range of 1 MiB, all others have 16 MiB.
static bool patchInRange(Ctx &ctx, const InputSection *isec, uint64_t off,
                         uint32_t instr) {

  // We need the branch at source to reach a patch section placed immediately
  // after isec. As there can be more than one patch in the patch section we
  // add 0x100 as contingency to account for worst case of 1 branch every 4KiB
  // for a 1 MiB range.
  return ctx.target->inBranchRange(
      isBcc(instr) ? R_ARM_THM_JUMP19 : R_ARM_THM_JUMP24, isec->getVA(off),
      isec->getVA() + isec->getSize() + 0x100);
}
```

- EN: Declares or implements routines including `isBcc`, `getVA`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isBcc`, `getVA`.
- CN: 这里声明或实现函数，例如 `isBcc`, `getVA`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isBcc`, `getVA`。

### Lines 245-253

```cpp
struct ScanResult {
  // Offset of branch within its InputSection.
  uint64_t off;
  // Cached decoding of the branch instruction.
  uint32_t instr;
  // Branch relocation at off. Will be nullptr if no relocation exists.
  Relocation *rel;
};
```

- EN: Introduces type definitions such as `ScanResult`. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `ScanResult`.
- CN: 这里引入类型定义，例如 `ScanResult`。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `ScanResult`。

### Lines 254-271

```cpp
// Detect the erratum sequence, returning the offset of the branch instruction
// and a decoding of the branch. If the erratum sequence is not found then
// return an offset of 0 for the branch. 0 is a safe value to use for no patch
// as there must be at least one 32-bit non-branch instruction before the
// branch so the minimum offset for a patch is 4.
static ScanResult scanCortexA8Errata657417(InputSection *isec, uint64_t &off,
                                           uint64_t limit) {
  Ctx &ctx = isec->getCtx();
  uint64_t isecAddr = isec->getVA(0);
  // Advance Off so that (isecAddr + off) modulo 0x1000 is at least 0xffa. We
  // need to check for a 32-bit instruction immediately before a 32-bit branch
  // at 0xffe modulo 0x1000.
  off = alignTo(isecAddr + off, 0x1000, 0xffa) - isecAddr;
  if (off >= limit || limit - off < 8) {
    // Need at least 2 4-byte sized instructions to trigger erratum.
    off = limit;
    return {0, 0, nullptr};
  }
```

- EN: Declares or implements routines including `getCtx`, `getVA`, `alignTo`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getCtx`, `getVA`, `alignTo`.
- CN: 这里声明或实现函数，例如 `getCtx`, `getVA`, `alignTo`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getCtx`, `getVA`, `alignTo`。

### Lines 272-289

```cpp

  ScanResult scanRes = {0, 0, nullptr};
  const uint8_t *buf = isec->content().begin();
  // ARMv7-A Thumb 32-bit instructions are encoded 2 consecutive
  // little-endian halfwords.
  const ulittle16_t *instBuf = reinterpret_cast<const ulittle16_t *>(buf + off);
  uint16_t hw11 = *instBuf++;
  uint16_t hw12 = *instBuf++;
  uint16_t hw21 = *instBuf++;
  uint16_t hw22 = *instBuf++;
  if (is32bitInstruction(hw11) && is32bitInstruction(hw21)) {
    uint32_t instr1 = (hw11 << 16) | hw12;
    uint32_t instr2 = (hw21 << 16) | hw22;
    if (!is32bitBranch(instr1) && is32bitBranch(instr2)) {
      // Find a relocation for the branch if it exists. This will be used
      // to determine the target.
      uint64_t branchOff = off + 4;
      auto relIt = llvm::find_if(isec->relocs(), [=](const Relocation &r) {
```

- EN: Declares or implements routines including `content`, `find_if`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `content`, `find_if`.
- CN: 这里声明或实现函数，例如 `content`, `find_if`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `content`, `find_if`。

### Lines 290-307

```cpp
        return r.offset == branchOff &&
               (r.type == R_ARM_THM_JUMP19 || r.type == R_ARM_THM_JUMP24 ||
                r.type == R_ARM_THM_CALL);
      });
      if (relIt != isec->relocs().end())
        scanRes.rel = &(*relIt);
      if (branchDestInFirstRegion(ctx, isec, branchOff, instr2, scanRes.rel)) {
        if (patchInRange(ctx, isec, branchOff, instr2)) {
          scanRes.off = branchOff;
          scanRes.instr = instr2;
        } else {
          Warn(ctx) << isec->file
                    << ": skipping cortex-a8 657417 erratum sequence, section "
                    << isec->name << " is too large to patch";
        }
      }
    }
  }
```

- EN: Declares or implements routines including `Warn`. Notable symbols here include `Warn`.
- CN: 这里声明或实现函数，例如 `Warn`。这里较值得关注的符号包括 `Warn`。

### Lines 308-318

```cpp
  off += 0x1000;
  return scanRes;
}

void ARMErr657417Patcher::init() {
  // The Arm ABI permits a mix of ARM, Thumb and Data in the same
  // InputSection. We must only scan Thumb instructions to avoid false
  // matches. We use the mapping symbols in the InputObjects to identify this
  // data, caching the results in sectionMap so we don't have to recalculate
  // it each pass.
```

- EN: Declares or implements routines including `init`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `init`.
- CN: 这里声明或实现函数，例如 `init`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `init`。

### Lines 319-333

```cpp
  // The ABI Section 4.5.5 Mapping symbols; defines local symbols that describe
  // half open intervals [Symbol Value, Next Symbol Value) of code and data
  // within sections. If there is no next symbol then the half open interval is
  // [Symbol Value, End of section). The type, code or data, is determined by
  // the mapping symbol name, $a for Arm code, $t for Thumb code, $d for data.
  auto isArmMapSymbol = [](const Symbol *s) {
    return s->getName() == "$a" || s->getName().starts_with("$a.");
  };
  auto isThumbMapSymbol = [](const Symbol *s) {
    return s->getName() == "$t" || s->getName().starts_with("$t.");
  };
  auto isDataMapSymbol = [](const Symbol *s) {
    return s->getName() == "$d" || s->getName().starts_with("$d.");
  };
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 334-351

```cpp
  // Collect mapping symbols for every executable InputSection.
  for (ELFFileBase *file : ctx.objectFiles) {
    for (Symbol *s : file->getLocalSymbols()) {
      auto *def = dyn_cast<Defined>(s);
      if (!def)
        continue;
      if (!isArmMapSymbol(def) && !isThumbMapSymbol(def) &&
          !isDataMapSymbol(def))
        continue;
      if (auto *sec = dyn_cast_or_null<InputSection>(def->section))
        if (sec->flags & SHF_EXECINSTR)
          sectionMap[sec].push_back(def);
    }
  }
  // For each InputSection make sure the mapping symbols are in sorted in
  // ascending order and are in alternating Thumb, non-Thumb order.
  for (auto &kv : sectionMap) {
    std::vector<const Defined *> &mapSyms = kv.second;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 352-367

```cpp
    llvm::stable_sort(mapSyms, [](const Defined *a, const Defined *b) {
      return a->value < b->value;
    });
    mapSyms.erase(llvm::unique(mapSyms,
                               [=](const Defined *a, const Defined *b) {
                                 return (isThumbMapSymbol(a) ==
                                         isThumbMapSymbol(b));
                               }),
                  mapSyms.end());
    // Always start with a Thumb Mapping Symbol
    if (!mapSyms.empty() && !isThumbMapSymbol(mapSyms.front()))
      mapSyms.erase(mapSyms.begin());
  }
  initialized = true;
}
```

- EN: Declares or implements routines including `stable_sort`, `isThumbMapSymbol`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `stable_sort`, `isThumbMapSymbol`.
- CN: 这里声明或实现函数，例如 `stable_sort`, `isThumbMapSymbol`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `stable_sort`, `isThumbMapSymbol`。

### Lines 368-375

```cpp
void ARMErr657417Patcher::insertPatches(
    InputSectionDescription &isd, std::vector<Patch657417Section *> &patches) {
  uint64_t spacing = 0x100000 - 0x7500;
  uint64_t isecLimit;
  uint64_t prevIsecLimit = isd.sections.front()->outSecOff;
  uint64_t patchUpperBound = prevIsecLimit + spacing;
  uint64_t outSecAddr = isd.sections.front()->getParent()->addr;
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 376-393

```cpp
  // Set the outSecOff of patches to the place where we want to insert them.
  // We use a similar strategy to initial thunk placement, using 1 MiB as the
  // range of the Thumb-2 conditional branch with a contingency accounting for
  // thunk generation.
  auto patchIt = patches.begin();
  auto patchEnd = patches.end();
  for (const InputSection *isec : isd.sections) {
    isecLimit = isec->outSecOff + isec->getSize();
    if (isecLimit > patchUpperBound) {
      for (; patchIt != patchEnd; ++patchIt) {
        if ((*patchIt)->getBranchAddr() - outSecAddr >= prevIsecLimit)
          break;
        (*patchIt)->outSecOff = prevIsecLimit;
      }
      patchUpperBound = prevIsecLimit + spacing;
    }
    prevIsecLimit = isecLimit;
  }
```

- EN: Declares or implements routines including `getSize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getSize`.
- CN: 这里声明或实现函数，例如 `getSize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getSize`。

### Lines 394-411

```cpp
  for (; patchIt != patchEnd; ++patchIt)
    (*patchIt)->outSecOff = isecLimit;

  // Merge all patch sections. We use the outSecOff assigned above to
  // determine the insertion point. This is ok as we only merge into an
  // InputSectionDescription once per pass, and at the end of the pass
  // assignAddresses() will recalculate all the outSecOff values.
  SmallVector<InputSection *, 0> tmp;
  tmp.reserve(isd.sections.size() + patches.size());
  auto mergeCmp = [](const InputSection *a, const InputSection *b) {
    if (a->outSecOff != b->outSecOff)
      return a->outSecOff < b->outSecOff;
    return isa<Patch657417Section>(a) && !isa<Patch657417Section>(b);
  };
  std::merge(isd.sections.begin(), isd.sections.end(), patches.begin(),
             patches.end(), std::back_inserter(tmp), mergeCmp);
  isd.sections = std::move(tmp);
}
```

- EN: Declares or implements routines including `merge`, `move`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `merge`, `move`.
- CN: 这里声明或实现函数，例如 `merge`, `move`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `merge`, `move`。

### Lines 412-429

```cpp

// Given a branch instruction described by ScanRes redirect it to a patch
// section containing an unconditional branch instruction to the target.
// Ensure that this patch section is 4-byte aligned so that the branch cannot
// span two 4 KiB regions. Place the patch section so that it is always after
// isec so the branch we are patching always goes forwards.
static void implementPatch(ScanResult sr, InputSection *isec,
                           std::vector<Patch657417Section *> &patches) {
  Ctx &ctx = isec->getCtx();
  Log(ctx) << "detected cortex-a8-657419 erratum sequence starting at " <<
      utohexstr(isec->getVA(sr.off)) << " in unpatched output";
  Patch657417Section *psec;
  // We have two cases to deal with.
  // Case 1. There is a relocation at patcheeOffset to a symbol. The
  // unconditional branch in the patch must have a relocation so that any
  // further redirection via the PLT or a Thunk happens as normal. At
  // patcheeOffset we redirect the existing relocation to a Symbol defined at
  // the start of the patch section.
```

- EN: Declares or implements routines including `getCtx`, `Log`, `utohexstr`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getCtx`, `Log`, `utohexstr`.
- CN: 这里声明或实现函数，例如 `getCtx`, `Log`, `utohexstr`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getCtx`, `Log`, `utohexstr`。

### Lines 430-438

```cpp
  //
  // Case 2. There is no relocation at patcheeOffset. We are unlikely to have
  // a symbol that we can use as a target for a relocation in the patch section.
  // Luckily we know that the destination cannot be indirected via the PLT or
  // a Thunk so we can just write the destination directly.
  if (sr.rel) {
    // Case 1. We have an existing relocation to redirect to patch and a
    // Symbol target.
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 439-456

```cpp
    // Create a branch relocation for the unconditional branch in the patch.
    // This can be redirected via the PLT or Thunks.
    RelType patchRelType = R_ARM_THM_JUMP24;
    int64_t patchRelAddend = sr.rel->addend;
    bool destIsARM = false;
    if (isBL(sr.instr) || isBLX(sr.instr)) {
      // The final target of the branch may be ARM or Thumb, if the target
      // is ARM then we write the patch in ARM state to avoid a state change
      // Thunk from the patch to the target.
      uint64_t dstSymAddr = (sr.rel->expr == R_PLT_PC)
                                ? sr.rel->sym->getPltVA(ctx)
                                : sr.rel->sym->getVA(ctx);
      destIsARM = (dstSymAddr & 1) == 0;
    }
    psec = make<Patch657417Section>(ctx, isec, sr.off, sr.instr, destIsARM);
    if (destIsARM) {
      // The patch will be in ARM state. Use an ARM relocation and account for
      // the larger ARM PC-bias of 8 rather than Thumb's 4.
```

- EN: Declares or implements routines including `getPltVA`, `getVA`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getPltVA`, `getVA`.
- CN: 这里声明或实现函数，例如 `getPltVA`, `getVA`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getPltVA`, `getVA`。

### Lines 457-469

```cpp
      patchRelType = R_ARM_JUMP24;
      patchRelAddend -= 4;
    }
    psec->addReloc(
        Relocation{sr.rel->expr, patchRelType, 0, patchRelAddend, sr.rel->sym});
    // Redirect the existing branch relocation to the patch.
    sr.rel->expr = R_PC;
    sr.rel->addend = -4;
    sr.rel->sym = psec->patchSym;
  } else {
    // Case 2. We do not have a relocation to the patch. Add a relocation of the
    // appropriate type to the patch at patcheeOffset.
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 470-484

```cpp
    // The destination is ARM if we have a BLX.
    psec =
        make<Patch657417Section>(ctx, isec, sr.off, sr.instr, isBLX(sr.instr));
    RelType type;
    if (isBcc(sr.instr))
      type = R_ARM_THM_JUMP19;
    else if (isB(sr.instr))
      type = R_ARM_THM_JUMP24;
    else
      type = R_ARM_THM_CALL;
    isec->addReloc(Relocation{R_PC, type, sr.off, -4, psec->patchSym});
  }
  patches.push_back(psec);
}
```

- EN: Declares or implements routines including `addReloc`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `addReloc`.
- CN: 这里声明或实现函数，例如 `addReloc`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `addReloc`。

### Lines 485-502

```cpp
// Scan all the instructions in InputSectionDescription, for each instance of
// the erratum sequence create a Patch657417Section. We return the list of
// Patch657417Sections that need to be applied to the InputSectionDescription.
std::vector<Patch657417Section *>
ARMErr657417Patcher::patchInputSectionDescription(
    InputSectionDescription &isd) {
  std::vector<Patch657417Section *> patches;
  for (InputSection *isec : isd.sections) {
    // LLD doesn't use the erratum sequence in SyntheticSections.
    if (isa<SyntheticSection>(isec))
      continue;
    // Use sectionMap to make sure we only scan Thumb code and not Arm or inline
    // data. We have already sorted mapSyms in ascending order and removed
    // consecutive mapping symbols of the same type. Our range of executable
    // instructions to scan is therefore [thumbSym->value, nonThumbSym->value)
    // or [thumbSym->value, section size).
    std::vector<const Defined *> &mapSyms = sectionMap[isec];
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 503-520

```cpp
    auto thumbSym = mapSyms.begin();
    while (thumbSym != mapSyms.end()) {
      auto nonThumbSym = std::next(thumbSym);
      uint64_t off = (*thumbSym)->value;
      uint64_t limit = nonThumbSym == mapSyms.end() ? isec->content().size()
                                                    : (*nonThumbSym)->value;

      while (off < limit) {
        ScanResult sr = scanCortexA8Errata657417(isec, off, limit);
        if (sr.off)
          implementPatch(sr, isec, patches);
      }
      if (nonThumbSym == mapSyms.end())
        break;
      thumbSym = std::next(nonThumbSym);
    }
  }
  return patches;
```

- EN: Declares or implements routines including `next`, `scanCortexA8Errata657417`, `implementPatch`. Notable symbols here include `next`, `scanCortexA8Errata657417`, `implementPatch`.
- CN: 这里声明或实现函数，例如 `next`, `scanCortexA8Errata657417`, `implementPatch`。这里较值得关注的符号包括 `next`, `scanCortexA8Errata657417`, `implementPatch`。

### Lines 521-538

```cpp
}

bool ARMErr657417Patcher::createFixes() {
  if (!initialized)
    init();

  bool addressesChanged = false;
  for (OutputSection *os : ctx.outputSections) {
    if (!(os->flags & SHF_ALLOC) || !(os->flags & SHF_EXECINSTR))
      continue;
    for (SectionCommand *cmd : os->commands)
      if (auto *isd = dyn_cast<InputSectionDescription>(cmd)) {
        std::vector<Patch657417Section *> patches =
            patchInputSectionDescription(*isd);
        if (!patches.empty()) {
          insertPatches(*isd, patches);
          addressesChanged = true;
        }
```

- EN: Declares or implements routines including `createFixes`, `init`, `patchInputSectionDescription`, `insertPatches`. Notable symbols here include `createFixes`, `init`, `patchInputSectionDescription`, `insertPatches`.
- CN: 这里声明或实现函数，例如 `createFixes`, `init`, `patchInputSectionDescription`, `insertPatches`。这里较值得关注的符号包括 `createFixes`, `init`, `patchInputSectionDescription`, `insertPatches`。

### Lines 539-542

```cpp
      }
  }
  return addressesChanged;
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

## Key Concepts / 关键概念

- `elf`: class or struct interface / 类或结构体接口
- `ScanResult`: class or struct interface / 类或结构体接口
- `writeTo`: function or method entry point / 函数或方法入口
- `getSize`: function or method entry point / 函数或方法入口
- `getBranchAddr`: function or method entry point / 函数或方法入口
- `classof`: function or method entry point / 函数或方法入口
- `is32bitInstruction`: function or method entry point / 函数或方法入口
- `llvm`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- LLVM headers / LLVM 头文件: `llvm/Support/Endian.h`
- System headers / 系统头文件: `ARMErrataFix.h`, `InputFiles.h`, `LinkerScript.h`, `OutputSections.h`, `Relocations.h`, `Symbols.h`, `SyntheticSections.h`, `Target.h`, `algorithm`
- Directory context / 目录上下文: `lld/ELF` neighbors usually cooperate with this file to provide the surrounding subsystem / `lld/ELF` 下的相邻文件通常与本文件协作组成对应子系统
