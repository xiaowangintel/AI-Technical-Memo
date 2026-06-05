# MCAssembler.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/MCAssembler.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Assembler Backend Implementation.
  - **CN**: 实现核心 MC 汇编器，负责管理片段、求值 fixup、执行布局并输出目标数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32
```cpp
//===- lib/MC/MCAssembler.cpp - Assembler Backend Implementation ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/MC/MCAssembler.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/Twine.h"
#include "llvm/MC/MCAsmBackend.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCCodeEmitter.h"
#include "llvm/MC/MCCodeView.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCDwarf.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCFixup.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/MC/MCSFrame.h"
#include "llvm/MC/MCSection.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/MCValue.h"
#include "llvm/Support/Alignment.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/EndianStream.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCAssembler.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCAssembler.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`。

### Lines 33-48
```cpp
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/LEB128.h"
#include "llvm/Support/raw_ostream.h"
#include <cassert>
#include <cstdint>
#include <tuple>
#include <utility>

using namespace llvm;

namespace llvm {
class MCSubtargetInfo;
}

#define DEBUG_TYPE "assembler"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/Support/ErrorHandling.h`, `llvm/Support/LEB128.h`, `llvm/Support/raw_ostream.h`, `cassert`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/Support/ErrorHandling.h`, `llvm/Support/LEB128.h`, `llvm/Support/raw_ostream.h`, `cassert`。

### Lines 49-68
```cpp
namespace {
namespace stats {

STATISTIC(EmittedFragments, "Number of emitted assembler fragments - total");
STATISTIC(EmittedRelaxableFragments,
          "Number of emitted assembler fragments - relaxable");
STATISTIC(EmittedDataFragments,
          "Number of emitted assembler fragments - data");
STATISTIC(EmittedAlignFragments,
          "Number of emitted assembler fragments - align");
STATISTIC(EmittedFillFragments,
          "Number of emitted assembler fragments - fill");
STATISTIC(EmittedNopsFragments, "Number of emitted assembler fragments - nops");
STATISTIC(EmittedOrgFragments, "Number of emitted assembler fragments - org");
STATISTIC(Fixups, "Number of fixups");
STATISTIC(FixupEvalForRelax, "Number of fixup evaluations for relaxation");
STATISTIC(ObjectBytes, "Number of emitted object file bytes");
STATISTIC(RelaxationSteps, "Number of assembler layout and relaxation steps");
STATISTIC(RelaxedInstructions, "Number of relaxed instructions");

```
- **EN**: Introduces declarations for `stats`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `stats` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 69-90
```cpp
} // end namespace stats
} // end anonymous namespace

// FIXME FIXME FIXME: There are number of places in this file where we convert
// what is a 64-bit assembler value used for computation into a value in the
// object file, which may truncate it. We should detect that truncation where
// invalid and report errors back.

/* *** */

MCAssembler::MCAssembler(MCContext &Context,
                         std::unique_ptr<MCAsmBackend> Backend,
                         std::unique_ptr<MCCodeEmitter> Emitter,
                         std::unique_ptr<MCObjectWriter> Writer)
    : Context(Context), Backend(std::move(Backend)),
      Emitter(std::move(Emitter)), Writer(std::move(Writer)) {
  if (this->Backend)
    this->Backend->setAssembler(this);
  if (this->Writer)
    this->Writer->setAssembler(this);
}

```
- **EN**: Introduces declarations for `stats`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `stats` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 91-107
```cpp
void MCAssembler::reset() {
  HasLayout = false;
  HasFinalLayout = false;
  RelaxAll = false;
  Sections.clear();
  Symbols.clear();
  ThumbFuncs.clear();

  // reset objects owned by us
  if (getBackendPtr())
    getBackendPtr()->reset();
  if (getEmitterPtr())
    getEmitterPtr()->reset();
  if (Writer)
    Writer->reset();
}

```
- **EN**: Implements logic around `reset`, `clear`, `getBackendPtr`, `getEmitterPtr`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `reset`, `clear`, `getBackendPtr`, `getEmitterPtr` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 108-124
```cpp
bool MCAssembler::registerSection(MCSection &Section) {
  if (Section.isRegistered())
    return false;
  Sections.push_back(&Section);
  Section.setIsRegistered(true);
  return true;
}

bool MCAssembler::isThumbFunc(const MCSymbol *Symbol) const {
  if (ThumbFuncs.count(Symbol))
    return true;

  if (!Symbol->isVariable())
    return false;

  const MCExpr *Expr = Symbol->getVariableValue();

```
- **EN**: Implements logic around `registerSection`, `push_back`, `setIsRegistered`, `isThumbFunc`, and 1 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `registerSection`, `push_back`, `setIsRegistered`, `isThumbFunc`, and 1 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 125-142
```cpp
  MCValue V;
  if (!Expr->evaluateAsRelocatable(V, nullptr))
    return false;

  if (V.getSubSym() || V.getSpecifier())
    return false;

  auto *Sym = V.getAddSym();
  if (!Sym || V.getSpecifier())
    return false;

  if (!isThumbFunc(Sym))
    return false;

  ThumbFuncs.insert(Symbol); // Cache it.
  return true;
}

```
- **EN**: Implements logic around `getAddSym`, `insert`; this block handles relocation, fixup, or symbol-resolution work; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getAddSym`, `insert` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 143-161
```cpp
bool MCAssembler::evaluateFixup(const MCFragment &F, MCFixup &Fixup,
                                MCValue &Target, uint64_t &Value,
                                bool RecordReloc, uint8_t *Data) const {
  if (RecordReloc)
    ++stats::Fixups;

  // FIXME: This code has some duplication with recordRelocation. We should
  // probably merge the two into a single callback that tries to evaluate a
  // fixup and records a relocation if one is needed.

  // On error claim to have completely evaluated the fixup, to prevent any
  // further processing from being done.
  const MCExpr *Expr = Fixup.getValue();
  Value = 0;
  if (!Expr->evaluateAsRelocatable(Target, this)) {
    reportError(Fixup.getLoc(), "expected relocatable expression");
    return true;
  }

```
- **EN**: Implements logic around `evaluateFixup`, `getValue`, `reportError`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `evaluateFixup`, `getValue`, `reportError` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 162-193
```cpp
  bool IsResolved = false;
  if (auto State = getBackend().evaluateFixup(F, Fixup, Target, Value)) {
    IsResolved = *State;
  } else {
    const MCSymbol *Add = Target.getAddSym();
    const MCSymbol *Sub = Target.getSubSym();
    Value += Target.getConstant();
    if (Add && Add->isDefined())
      Value += getSymbolOffset(*Add);
    if (Sub && Sub->isDefined())
      Value -= getSymbolOffset(*Sub);

    if (Fixup.isPCRel()) {
      Value -= getFragmentOffset(F) + Fixup.getOffset();
      // During relaxation, F's offset is already updated but forward reference
      // targets are stale. Add Stretch so that the displacement equals
      // target_old - source_old, preventing premature relaxation.
      if (Stretch) {
        assert(!RecordReloc &&
               "Stretch should only be applied during relaxation");
        MCFragment *AF = Add ? Add->getFragment() : nullptr;
        if (AF && AF->getLayoutOrder() > F.getLayoutOrder())
          Value += Stretch;
        MCFragment *SF = Sub ? Sub->getFragment() : nullptr;
        if (SF && SF->getLayoutOrder() > F.getLayoutOrder())
          Value -= Stretch;
      }
      if (Add && !Sub && !Add->isUndefined() && !Add->isAbsolute()) {
        IsResolved = getWriter().isSymbolRefDifferenceFullyResolvedImpl(
            *Add, F, false, true);
      }
    } else {
```
- **EN**: Implements logic around `getAddSym`, `getSubSym`, `getConstant`, `getSymbolOffset`, and 4 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `getAddSym`, `getSubSym`, `getConstant`, `getSymbolOffset`, and 4 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 194-225
```cpp
      IsResolved = Target.isAbsolute();
    }
  }

  if (!RecordReloc)
    return IsResolved;

  if (IsResolved && mc::isRelocRelocation(Fixup.getKind()))
    IsResolved = false;
  getBackend().applyFixup(F, Fixup, Target, Data, Value, IsResolved);
  return true;
}

uint64_t MCAssembler::computeFragmentSize(const MCFragment &F) const {
  assert(getBackendPtr() && "Requires assembler backend");
  switch (F.getKind()) {
  case MCFragment::FT_Data:
  case MCFragment::FT_Relaxable:
  case MCFragment::FT_Align:
  case MCFragment::FT_LEB:
  case MCFragment::FT_Dwarf:
  case MCFragment::FT_DwarfFrame:
  case MCFragment::FT_SFrame:
  case MCFragment::FT_CVInlineLines:
  case MCFragment::FT_CVDefRange:
    return F.getSize();
  case MCFragment::FT_Fill: {
    auto &FF = static_cast<const MCFillFragment &>(F);
    int64_t NumValues = 0;
    if (!FF.getNumValues().evaluateKnownAbsolute(NumValues, *this)) {
      recordError(FF.getLoc(), "expected assembly-time absolute expression");
      return 0;
```
- **EN**: Implements logic around `isAbsolute`, `getBackend`, `computeFragmentSize`, `assert`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `isAbsolute`, `getBackend`, `computeFragmentSize`, `assert`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 226-243
```cpp
    }
    int64_t Size = NumValues * FF.getValueSize();
    if (Size < 0) {
      recordError(FF.getLoc(), "invalid number of bytes");
      return 0;
    }
    return Size;
  }

  case MCFragment::FT_PrefAlign:
    return F.getSize();

  case MCFragment::FT_Nops:
    return cast<MCNopsFragment>(F).getNumBytes();

  case MCFragment::FT_BoundaryAlign:
    return cast<MCBoundaryAlignFragment>(F).getSize();

```
- **EN**: Implements logic around `getValueSize`, `recordError`, `getSize`, `cast<MCNopsFragment>`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `getValueSize`, `recordError`, `getSize`, `cast<MCNopsFragment>`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 244-275
```cpp
  case MCFragment::FT_SymbolId:
    return 4;

  case MCFragment::FT_Org: {
    const MCOrgFragment &OF = cast<MCOrgFragment>(F);
    MCValue Value;
    if (!OF.getOffset().evaluateAsValue(Value, *this)) {
      recordError(OF.getLoc(), "expected assembly-time absolute expression");
      return 0;
    }

    uint64_t FragmentOffset = getFragmentOffset(OF);
    int64_t TargetLocation = Value.getConstant();
    if (const auto *SA = Value.getAddSym()) {
      uint64_t Val;
      if (!getSymbolOffset(*SA, Val)) {
        recordError(OF.getLoc(), "expected absolute expression");
        return 0;
      }
      TargetLocation += Val;
    }
    int64_t Size = TargetLocation - FragmentOffset;
    if (Size < 0 || Size >= 0x40000000) {
      recordError(OF.getLoc(), "invalid .org offset '" + Twine(TargetLocation) +
                                   "' (at offset '" + Twine(FragmentOffset) +
                                   "')");
      return 0;
    }
    return Size;
  }
  }

```
- **EN**: Implements logic around `cast<MCOrgFragment>`, `recordError`, `getFragmentOffset`, `getConstant`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `cast<MCOrgFragment>`, `recordError`, `getFragmentOffset`, `getConstant`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 276-291
```cpp
  llvm_unreachable("invalid fragment kind");
}

// Simple getSymbolOffset helper for the non-variable case.
static bool getLabelOffset(const MCAssembler &Asm, const MCSymbol &S,
                           bool ReportError, uint64_t &Val) {
  if (!S.getFragment()) {
    if (ReportError)
      reportFatalUsageError("cannot evaluate undefined symbol '" + S.getName() +
                            "'");
    return false;
  }
  Val = Asm.getFragmentOffset(*S.getFragment()) + S.getOffset();
  return true;
}

```
- **EN**: Implements logic around `llvm_unreachable`, `getLabelOffset`, `reportFatalUsageError`, `getFragmentOffset`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `llvm_unreachable`, `getLabelOffset`, `reportFatalUsageError`, `getFragmentOffset` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 292-316
```cpp
static bool getSymbolOffsetImpl(const MCAssembler &Asm, const MCSymbol &S,
                                bool ReportError, uint64_t &Val) {
  if (!S.isVariable())
    return getLabelOffset(Asm, S, ReportError, Val);

  // If SD is a variable, evaluate it.
  MCValue Target;
  if (!S.getVariableValue()->evaluateAsValue(Target, Asm))
    reportFatalUsageError("cannot evaluate equated symbol '" + S.getName() +
                          "'");

  uint64_t Offset = Target.getConstant();

  const MCSymbol *A = Target.getAddSym();
  if (A) {
    uint64_t ValA;
    // FIXME: On most platforms, `Target`'s component symbols are labels from
    // having been simplified during evaluation, but on Mach-O they can be
    // variables due to PR19203. This, and the line below for `B` can be
    // restored to call `getLabelOffset` when PR19203 is fixed.
    if (!getSymbolOffsetImpl(Asm, *A, ReportError, ValA))
      return false;
    Offset += ValA;
  }

```
- **EN**: Implements logic around `getSymbolOffsetImpl`, `getLabelOffset`, `reportFatalUsageError`, `getConstant`, and 1 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getSymbolOffsetImpl`, `getLabelOffset`, `reportFatalUsageError`, `getConstant`, and 1 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 317-332
```cpp
  const MCSymbol *B = Target.getSubSym();
  if (B) {
    uint64_t ValB;
    if (!getSymbolOffsetImpl(Asm, *B, ReportError, ValB))
      return false;
    Offset -= ValB;
  }

  Val = Offset;
  return true;
}

bool MCAssembler::getSymbolOffset(const MCSymbol &S, uint64_t &Val) const {
  return getSymbolOffsetImpl(*this, S, false, Val);
}

```
- **EN**: Implements logic around `getSubSym`, `getSymbolOffset`, `getSymbolOffsetImpl`; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getSubSym`, `getSymbolOffset`, `getSymbolOffsetImpl` 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 333-350
```cpp
uint64_t MCAssembler::getSymbolOffset(const MCSymbol &S) const {
  uint64_t Val;
  getSymbolOffsetImpl(*this, S, true, Val);
  return Val;
}

const MCSymbol *MCAssembler::getBaseSymbol(const MCSymbol &Symbol) const {
  assert(HasLayout);
  if (!Symbol.isVariable())
    return &Symbol;

  const MCExpr *Expr = Symbol.getVariableValue();
  MCValue Value;
  if (!Expr->evaluateAsValue(Value, *this)) {
    reportError(Expr->getLoc(), "expression could not be evaluated");
    return nullptr;
  }

```
- **EN**: Implements logic around `getSymbolOffset`, `getSymbolOffsetImpl`, `getBaseSymbol`, `assert`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getSymbolOffset`, `getSymbolOffsetImpl`, `getBaseSymbol`, `assert`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 351-369
```cpp
  const MCSymbol *SymB = Value.getSubSym();
  if (SymB) {
    reportError(Expr->getLoc(),
                Twine("symbol '") + SymB->getName() +
                    "' could not be evaluated in a subtraction expression");
    return nullptr;
  }

  const MCSymbol *A = Value.getAddSym();
  if (!A)
    return nullptr;

  const MCSymbol &ASym = *A;
  if (ASym.isCommon()) {
    reportError(Expr->getLoc(), "Common symbol '" + ASym.getName() +
                                    "' cannot be used in assignment expr");
    return nullptr;
  }

```
- **EN**: Implements logic around `getSubSym`, `reportError`, `Twine`, `getAddSym`; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getSubSym`, `reportError`, `Twine`, `getAddSym` 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 370-385
```cpp
  return &ASym;
}

uint64_t MCAssembler::getSectionAddressSize(const MCSection &Sec) const {
  const MCFragment &F = *Sec.curFragList()->Tail;
  assert(HasLayout && F.getKind() == MCFragment::FT_Data);
  return getFragmentOffset(F) + F.getSize();
}

uint64_t MCAssembler::getSectionFileSize(const MCSection &Sec) const {
  // Virtual sections have no file size.
  if (Sec.isBssSection())
    return 0;
  return getSectionAddressSize(Sec);
}

```
- **EN**: Implements logic around `getSectionAddressSize`, `curFragList`, `assert`, `getFragmentOffset`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getSectionAddressSize`, `curFragList`, `assert`, `getFragmentOffset`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 386-404
```cpp
bool MCAssembler::registerSymbol(const MCSymbol &Symbol) {
  bool Changed = !Symbol.isRegistered();
  if (Changed) {
    Symbol.setIsRegistered(true);
    Symbols.push_back(&Symbol);
  }
  return Changed;
}

void MCAssembler::addRelocDirective(RelocDirective RD) {
  relocDirectives.push_back(RD);
}

/// Write the fragment \p F to the output file.
static void writeFragment(raw_ostream &OS, const MCAssembler &Asm,
                          const MCFragment &F) {
  // FIXME: Embed in fragments instead?
  uint64_t FragmentSize = Asm.computeFragmentSize(F);

```
- **EN**: Implements logic around `registerSymbol`, `isRegistered`, `setIsRegistered`, `push_back`, and 3 more symbols; this block handles relocation, fixup, or symbol-resolution work; parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `registerSymbol`, `isRegistered`, `setIsRegistered`, `push_back`, and 3 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 405-431
```cpp
  llvm::endianness Endian = Asm.getBackend().Endian;

  // This variable (and its dummy usage) is to participate in the assert at
  // the end of the function.
  uint64_t Start = OS.tell();
  (void) Start;

  ++stats::EmittedFragments;

  switch (F.getKind()) {
  case MCFragment::FT_Data:
  case MCFragment::FT_Relaxable:
  case MCFragment::FT_LEB:
  case MCFragment::FT_Dwarf:
  case MCFragment::FT_DwarfFrame:
  case MCFragment::FT_SFrame:
  case MCFragment::FT_CVInlineLines:
  case MCFragment::FT_CVDefRange: {
    if (F.getKind() == MCFragment::FT_Data)
      ++stats::EmittedDataFragments;
    else if (F.getKind() == MCFragment::FT_Relaxable)
      ++stats::EmittedRelaxableFragments;
    const auto &EF = cast<MCFragment>(F);
    OS << StringRef(EF.getContents().data(), EF.getContents().size());
    OS << StringRef(EF.getVarContents().data(), EF.getVarContents().size());
  } break;

```
- **EN**: Implements logic around `getBackend`, `tell`, `cast<MCFragment>`, `StringRef`; this block drives emission, layout, or binary encoding behavior; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `getBackend`, `tell`, `cast<MCFragment>`, `StringRef` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 432-463
```cpp
  case MCFragment::FT_Align: {
    ++stats::EmittedAlignFragments;
    OS << StringRef(F.getContents().data(), F.getContents().size());
    assert(F.getAlignFillLen() &&
           "Invalid virtual align in concrete fragment!");

    uint64_t Count = (FragmentSize - F.getFixedSize()) / F.getAlignFillLen();
    assert((FragmentSize - F.getFixedSize()) % F.getAlignFillLen() == 0 &&
           "computeFragmentSize computed size is incorrect");

    // In the nops mode, call the backend hook to write `Count` nops.
    if (F.hasAlignEmitNops()) {
      if (!Asm.getBackend().writeNopData(OS, Count, F.getSubtargetInfo()))
        reportFatalInternalError("unable to write nop sequence of " +
                                 Twine(Count) + " bytes");
    } else {
      // Otherwise, write out in multiples of the value size.
      for (uint64_t i = 0; i != Count; ++i) {
        switch (F.getAlignFillLen()) {
        default:
          llvm_unreachable("Invalid size!");
        case 1:
          OS << char(F.getAlignFill());
          break;
        case 2:
          support::endian::write<uint16_t>(OS, F.getAlignFill(), Endian);
          break;
        case 4:
          support::endian::write<uint32_t>(OS, F.getAlignFill(), Endian);
          break;
        case 8:
          support::endian::write<uint64_t>(OS, F.getAlignFill(), Endian);
```
- **EN**: Implements logic around `StringRef`, `assert`, `getFixedSize`, `reportFatalInternalError`, and 6 more symbols; this block drives emission, layout, or binary encoding behavior; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `StringRef`, `assert`, `getFixedSize`, `reportFatalInternalError`, and 6 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 464-486
```cpp
          break;
        }
      }
    }
  } break;

  case MCFragment::FT_PrefAlign: {
    OS << StringRef(F.getContents().data(), F.getContents().size());
    uint64_t PadSize = FragmentSize - F.getContents().size();
    if (F.getPrefAlignEmitNops()) {
      if (!Asm.getBackend().writeNopData(OS, PadSize, F.getSubtargetInfo()))
        reportFatalInternalError("unable to write nop sequence of " +
                                 Twine(PadSize) + " bytes");
    } else if (F.getPrefAlignFill() == 0) {
      OS.write_zeros(PadSize);
    } else {
      char B = char(F.getPrefAlignFill());
      for (uint64_t I = 0; I < PadSize; ++I)
        OS << B;
    }
    break;
  }

```
- **EN**: Implements logic around `StringRef`, `getContents`, `reportFatalInternalError`, `Twine`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `StringRef`, `getContents`, `reportFatalInternalError`, `Twine`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 487-503
```cpp
  case MCFragment::FT_Fill: {
    ++stats::EmittedFillFragments;
    const MCFillFragment &FF = cast<MCFillFragment>(F);
    uint64_t V = FF.getValue();
    unsigned VSize = FF.getValueSize();
    const unsigned MaxChunkSize = 16;
    char Data[MaxChunkSize];
    assert(0 < VSize && VSize <= MaxChunkSize && "Illegal fragment fill size");
    // Duplicate V into Data as byte vector to reduce number of
    // writes done. As such, do endian conversion here.
    for (unsigned I = 0; I != VSize; ++I) {
      unsigned index = Endian == llvm::endianness::little ? I : (VSize - I - 1);
      Data[I] = uint8_t(V >> (index * 8));
    }
    for (unsigned I = VSize; I < MaxChunkSize; ++I)
      Data[I] = Data[I - VSize];

```
- **EN**: Implements logic around `cast<MCFillFragment>`, `getValue`, `getValueSize`, `assert`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `cast<MCFillFragment>`, `getValue`, `getValueSize`, `assert`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 504-520
```cpp
    // Set to largest multiple of VSize in Data.
    const unsigned NumPerChunk = MaxChunkSize / VSize;
    // Set ChunkSize to largest multiple of VSize in Data
    const unsigned ChunkSize = VSize * NumPerChunk;

    // Do copies by chunk.
    StringRef Ref(Data, ChunkSize);
    for (uint64_t I = 0, E = FragmentSize / ChunkSize; I != E; ++I)
      OS << Ref;

    // do remainder if needed.
    unsigned TrailingCount = FragmentSize % ChunkSize;
    if (TrailingCount)
      OS.write(Data, TrailingCount);
    break;
  }

```
- **EN**: Implements logic around `Ref`, `write`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `Ref`, `write` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 521-542
```cpp
  case MCFragment::FT_Nops: {
    ++stats::EmittedNopsFragments;
    const MCNopsFragment &NF = cast<MCNopsFragment>(F);

    int64_t NumBytes = NF.getNumBytes();
    int64_t ControlledNopLength = NF.getControlledNopLength();
    int64_t MaximumNopLength =
        Asm.getBackend().getMaximumNopSize(*NF.getSubtargetInfo());

    assert(NumBytes > 0 && "Expected positive NOPs fragment size");
    assert(ControlledNopLength >= 0 && "Expected non-negative NOP size");

    if (ControlledNopLength > MaximumNopLength) {
      Asm.reportError(NF.getLoc(), "illegal NOP size " +
                                       std::to_string(ControlledNopLength) +
                                       ". (expected within [0, " +
                                       std::to_string(MaximumNopLength) + "])");
      // Clamp the NOP length as reportError does not stop the execution
      // immediately.
      ControlledNopLength = MaximumNopLength;
    }

```
- **EN**: Implements logic around `cast<MCNopsFragment>`, `getNumBytes`, `getControlledNopLength`, `getBackend`, and 3 more symbols; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `cast<MCNopsFragment>`, `getNumBytes`, `getControlledNopLength`, `getBackend`, and 3 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 543-561
```cpp
    // Use maximum value if the size of each NOP is not specified
    if (!ControlledNopLength)
      ControlledNopLength = MaximumNopLength;

    while (NumBytes) {
      uint64_t NumBytesToEmit =
          (uint64_t)std::min(NumBytes, ControlledNopLength);
      assert(NumBytesToEmit && "try to emit empty NOP instruction");
      if (!Asm.getBackend().writeNopData(OS, NumBytesToEmit,
                                         NF.getSubtargetInfo())) {
        report_fatal_error("unable to write nop sequence of the remaining " +
                           Twine(NumBytesToEmit) + " bytes");
        break;
      }
      NumBytes -= NumBytesToEmit;
    }
    break;
  }

```
- **EN**: Implements logic around `min`, `assert`, `getSubtargetInfo`, `report_fatal_error`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `min`, `assert`, `getSubtargetInfo`, `report_fatal_error`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 562-579
```cpp
  case MCFragment::FT_BoundaryAlign: {
    const MCBoundaryAlignFragment &BF = cast<MCBoundaryAlignFragment>(F);
    if (!Asm.getBackend().writeNopData(OS, FragmentSize, BF.getSubtargetInfo()))
      report_fatal_error("unable to write nop sequence of " +
                         Twine(FragmentSize) + " bytes");
    break;
  }

  case MCFragment::FT_SymbolId: {
    const MCSymbolIdFragment &SF = cast<MCSymbolIdFragment>(F);
    support::endian::write<uint32_t>(OS, SF.getSymbol()->getIndex(), Endian);
    break;
  }

  case MCFragment::FT_Org: {
    ++stats::EmittedOrgFragments;
    const MCOrgFragment &OF = cast<MCOrgFragment>(F);

```
- **EN**: Implements logic around `cast<MCBoundaryAlignFragment>`, `report_fatal_error`, `Twine`, `cast<MCSymbolIdFragment>`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `cast<MCBoundaryAlignFragment>`, `report_fatal_error`, `Twine`, `cast<MCSymbolIdFragment>`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 580-595
```cpp
    for (uint64_t i = 0, e = FragmentSize; i != e; ++i)
      OS << char(OF.getValue());

    break;
  }

  }

  assert(OS.tell() - Start == FragmentSize &&
         "The stream should advance by fragment size");
}

void MCAssembler::writeSectionData(raw_ostream &OS,
                                   const MCSection *Sec) const {
  assert(getBackendPtr() && "Expected assembler backend");

```
- **EN**: Implements logic around `char`, `assert`, `writeSectionData`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `char`, `assert`, `writeSectionData` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 596-627
```cpp
  if (Sec->isBssSection()) {
    assert(getSectionFileSize(*Sec) == 0 && "Invalid size for section!");

    // Ensure no fixups or non-zero bytes are written to BSS sections, catching
    // errors in both input assembly code and MCStreamer API usage. Location is
    // not tracked for efficiency.
    auto Fn = [](char c) { return c != 0; };
    for (const MCFragment &F : *Sec) {
      bool HasNonZero = false;
      switch (F.getKind()) {
      default:
        reportFatalInternalError("BSS section '" + Sec->getName() +
                                 "' contains invalid fragment");
        break;
      case MCFragment::FT_Data:
      case MCFragment::FT_Relaxable:
        HasNonZero =
            any_of(F.getContents(), Fn) || any_of(F.getVarContents(), Fn);
        break;
      case MCFragment::FT_Align:
        // Disallowed for API usage. AsmParser changes non-zero fill values to
        // 0.
        assert(F.getAlignFill() == 0 && "Invalid align in virtual section!");
        break;
      case MCFragment::FT_PrefAlign:
        assert(!F.getPrefAlignEmitNops() && F.getPrefAlignFill() == 0 &&
               "Invalid align in BSS");
        break;
      case MCFragment::FT_Fill:
        HasNonZero = cast<MCFillFragment>(F).getValue() != 0;
        break;
      case MCFragment::FT_Org:
```
- **EN**: Implements logic around `assert`, `reportFatalInternalError`, `any_of`, `cast<MCFillFragment>`; this block handles relocation, fixup, or symbol-resolution work; parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `assert`, `reportFatalInternalError`, `any_of`, `cast<MCFillFragment>` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 628-645
```cpp
        HasNonZero = cast<MCOrgFragment>(F).getValue() != 0;
        break;
      }
      if (HasNonZero) {
        reportError(SMLoc(), "BSS section '" + Sec->getName() +
                                 "' cannot have non-zero bytes");
        break;
      }
      if (F.getFixups().size() || F.getVarFixups().size()) {
        reportError(SMLoc(),
                    "BSS section '" + Sec->getName() + "' cannot have fixups");
        break;
      }
    }

    return;
  }

```
- **EN**: Implements logic around `cast<MCOrgFragment>`, `reportError`, `getName`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `cast<MCOrgFragment>`, `reportError`, `getName` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 646-663
```cpp
  uint64_t Start = OS.tell();
  (void)Start;

  for (const MCFragment &F : *Sec)
    writeFragment(OS, *this, F);

  flushPendingErrors();
  assert(getContext().hadError() ||
         OS.tell() - Start == getSectionAddressSize(*Sec));
}

void MCAssembler::layout() {
  assert(getBackendPtr() && "Expected assembler backend");
  DEBUG_WITH_TYPE("mc-dump-pre", {
    errs() << "assembler backend - pre-layout\n--\n";
    dump();
  });

```
- **EN**: Implements logic around `tell`, `writeFragment`, `flushPendingErrors`, `assert`, and 4 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `tell`, `writeFragment`, `flushPendingErrors`, `assert`, and 4 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 664-681
```cpp
  // Assign section ordinals.
  unsigned SectionIndex = 0;
  for (MCSection &Sec : *this) {
    Sec.setOrdinal(SectionIndex++);

    // Chain together fragments from all subsections.
    if (Sec.Subsections.size() > 1) {
      MCFragment Dummy;
      MCFragment *Tail = &Dummy;
      for (auto &[_, List] : Sec.Subsections) {
        assert(List.Head);
        Tail->Next = List.Head;
        Tail = List.Tail;
      }
      Sec.Subsections.clear();
      Sec.Subsections.push_back({0u, {Dummy.getNext(), Tail}});
      Sec.CurFragList = &Sec.Subsections[0].second;

```
- **EN**: Implements logic around `setOrdinal`, `assert`, `clear`, `push_back`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `setOrdinal`, `assert`, `clear`, `push_back` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 682-702
```cpp
      unsigned FragmentIndex = 0;
      for (MCFragment &Frag : Sec)
        Frag.setLayoutOrder(FragmentIndex++);
    }
  }

  // Layout until everything fits.
  this->HasLayout = true;
  for (MCSection &Sec : *this)
    layoutSection(Sec);
  unsigned FirstStable = Sections.size();
  while ((FirstStable = relaxOnce(FirstStable)) > 0)
    if (getContext().hadError())
      return;

  // Some targets might want to adjust fragment offsets. If so, perform another
  // layout iteration.
  if (getBackend().finishLayout())
    for (MCSection &Sec : *this)
      layoutSection(Sec);

```
- **EN**: Implements logic around `setLayoutOrder`, `layoutSection`, `size`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `setLayoutOrder`, `layoutSection`, `size` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 703-729
```cpp
  flushPendingErrors();

  DEBUG_WITH_TYPE("mc-dump", {
      errs() << "assembler backend - final-layout\n--\n";
      dump(); });

  // Allow the object writer a chance to perform post-layout binding (for
  // example, to set the index fields in the symbol data).
  getWriter().executePostLayoutBinding();

  // Fragment sizes are finalized. For RISC-V linker relaxation, this flag
  // helps check whether a PC-relative fixup is fully resolved.
  this->HasFinalLayout = true;

  // Stores the current .reloc group for each fragment.
  //
  // A .reloc group is a consecutive sequence of .reloc relocations that have
  // an offset <= the first relocation's offset. A relocation with offset > the
  // first relocation's offset starts a new group. Relocation groups are
  // inserted in offset order using the offset of the first relocation, but the
  // source ordering of relocations within the group is preserved.
  DenseMap<MCFragment *, std::vector<MCFixup>> RelocGroups;
  auto DrainRelocGroup = [](MCFragment *F, std::vector<MCFixup> &Group) {
    F->insertRelocFixups(Group);
    Group.clear();
  };

```
- **EN**: Implements logic around `flushPendingErrors`, `DEBUG_WITH_TYPE`, `errs`, `dump`, and 3 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `flushPendingErrors`, `DEBUG_WITH_TYPE`, `errs`, `dump`, and 3 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 730-746
```cpp
  // Resolve .reloc offsets and add fixups.
  for (auto &PF : relocDirectives) {
    MCValue Res;
    auto &O = PF.Offset;
    if (!O.evaluateAsValue(Res, *this)) {
      getContext().reportError(O.getLoc(), ".reloc offset is not relocatable");
      continue;
    }
    auto *Sym = Res.getAddSym();
    auto *F = Sym ? Sym->getFragment() : nullptr;
    auto *Sec = F ? F->getParent() : nullptr;
    if (Res.getSubSym() || !Sec) {
      getContext().reportError(O.getLoc(),
                               ".reloc offset is not relative to a section");
      continue;
    }

```
- **EN**: Implements logic around `getContext`, `getAddSym`, `getFragment`, `getParent`; this block handles relocation, fixup, or symbol-resolution work; parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `getContext`, `getAddSym`, `getFragment`, `getParent` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，解析汇编语法或操作数，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 747-771
```cpp
    uint64_t Offset = Sym ? Sym->getOffset() + Res.getConstant() : 0;
    auto Fixup = MCFixup::create(Offset, PF.Expr, PF.Kind);
    auto &Group = RelocGroups[F];
    if (!Group.empty() && Group[0].getOffset() < Offset)
      DrainRelocGroup(F, Group);
    Group.push_back(Fixup);
  }

  for (auto &[F, Group] : RelocGroups)
    DrainRelocGroup(F, Group);

  // Evaluate and apply the fixups, generating relocation entries as necessary.
  for (MCSection &Sec : *this) {
    for (MCFragment &F : Sec) {
      // Process fragments with fixups here.
      auto Contents = F.getContents();
      for (MCFixup &Fixup : F.getFixups()) {
        uint64_t FixedValue;
        MCValue Target;
        assert(mc::isRelocRelocation(Fixup.getKind()) ||
               Fixup.getOffset() <= F.getFixedSize());
        auto *Data =
            reinterpret_cast<uint8_t *>(Contents.data() + Fixup.getOffset());
        evaluateFixup(F, Fixup, Target, FixedValue,
                      /*RecordReloc=*/true, Data);
```
- **EN**: Implements logic around `getOffset`, `create`, `DrainRelocGroup`, `push_back`, and 4 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `getOffset`, `create`, `DrainRelocGroup`, `push_back`, and 4 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 772-789
```cpp
      }
      // In the variable part, fixup offsets are relative to the fixed part's
      // start.
      for (MCFixup &Fixup : F.getVarFixups()) {
        uint64_t FixedValue;
        MCValue Target;
        assert(mc::isRelocRelocation(Fixup.getKind()) ||
               (Fixup.getOffset() >= F.getFixedSize() &&
                Fixup.getOffset() <= F.getSize()));
        auto *Data = reinterpret_cast<uint8_t *>(
            F.getVarContents().data() + (Fixup.getOffset() - F.getFixedSize()));
        evaluateFixup(F, Fixup, Target, FixedValue,
                      /*RecordReloc=*/true, Data);
      }
    }
  }
}

```
- **EN**: Implements logic around `assert`, `getOffset`, `getVarContents`, `evaluateFixup`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `assert`, `getOffset`, `getVarContents`, `evaluateFixup` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 790-820
```cpp
void MCAssembler::Finish() {
  layout();

  // Write the object file if there is no error. The output would be discarded
  // anyway, and this avoids wasting time writing large files (e.g. when testing
  // fixup overflow with `.space 0x80000000`).
  if (!getContext().hadError())
    stats::ObjectBytes += getWriter().writeObject();

  HasLayout = false;
  assert(PendingErrors.empty());
}

void MCAssembler::relaxAlign(MCFragment &F) {
  uint64_t Offset = F.Offset + F.getFixedSize();
  unsigned Size = offsetToAlignment(Offset, F.getAlignment());
  bool AlignFixup = false;
  if (F.hasAlignEmitNops()) {
    AlignFixup = getBackend().relaxAlign(F, Size);
    if (!AlignFixup)
      while (Size % getBackend().getMinimumNopSize())
        Size += F.getAlignment().value();
  }
  if (!AlignFixup && Size > F.getAlignMaxBytesToEmit())
    Size = 0;
  F.VarContentStart = F.getFixedSize();
  F.VarContentEnd = F.VarContentStart + Size;
  if (F.VarContentEnd > F.getParent()->ContentStorage.size())
    F.getParent()->ContentStorage.resize(F.VarContentEnd);
}

```
- **EN**: Implements logic around `Finish`, `layout`, `getWriter`, `assert`, and 6 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `Finish`, `layout`, `getWriter`, `assert`, and 6 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为。

### Lines 821-852
```cpp
// Compute the body size by walking forward from F to the End symbol and
// summing fragment sizes. This avoids depending on stale layout offsets.
void MCAssembler::relaxPrefAlign(MCFragment &F) {
  uint64_t RawStart = F.Offset + F.getFixedSize();
  const MCSymbol &End = F.getPrefAlignEnd();
  if (!End.getFragment() || End.getFragment()->getParent() != F.getParent()) {
    recordError(SMLoc(), ".prefalign end symbol '" + End.getName() +
                             "' must be in the current section");
    return;
  }
  const MCFragment *EndFrag = End.getFragment();
  if (EndFrag->getLayoutOrder() <= F.getLayoutOrder())
    return;
  uint64_t BodySize = End.getOffset();
  for (auto *Cur = F.getNext(); Cur != EndFrag; Cur = Cur->getNext())
    BodySize += computeFragmentSize(*Cur);
  // Intervening FT_Align's padding depends on where this prefalign lands, so
  // `BodySize` depends on this prefalign's own padding and may not reach a
  // fixed point. Break the cycle with a monotone value.
  Align NewAlign =
      std::min(Align(llvm::bit_ceil(BodySize)), F.getPrefAlignPreferred());
  NewAlign = std::max(NewAlign, F.getPrefAlignComputed());
  F.setPrefAlignComputed(NewAlign);
  uint64_t NewPadSize = offsetToAlignment(RawStart, NewAlign);
  F.VarContentStart = F.getFixedSize();
  F.VarContentEnd = F.VarContentStart + NewPadSize;
  if (F.VarContentEnd > F.getParent()->ContentStorage.size())
    F.getParent()->ContentStorage.resize(F.VarContentEnd);
  // Update the maximum alignment on the current section if necessary, similar
  // to MCObjectStreamer::emitValueToAlignment.
  F.getParent()->ensureMinAlignment(NewAlign);
}
```
- **EN**: Implements logic around `relaxPrefAlign`, `getFixedSize`, `getPrefAlignEnd`, `recordError`, and 8 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `relaxPrefAlign`, `getFixedSize`, `getPrefAlignEnd`, `recordError`, and 8 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 853-874
```cpp

bool MCAssembler::fixupNeedsRelaxation(const MCFragment &F,
                                       const MCFixup &Fixup) const {
  ++stats::FixupEvalForRelax;
  MCValue Target;
  uint64_t Value;
  bool Resolved = evaluateFixup(F, const_cast<MCFixup &>(Fixup), Target, Value,
                                /*RecordReloc=*/false, {});
  return getBackend().fixupNeedsRelaxationAdvanced(F, Fixup, Target, Value,
                                                   Resolved);
}

void MCAssembler::relaxInstruction(MCFragment &F) {
  assert(getEmitterPtr() &&
         "Expected CodeEmitter defined for relaxInstruction");
  // If this inst doesn't ever need relaxation, ignore it. This occurs when we
  // are intentionally pushing out inst fragments, or because we relaxed a
  // previous instruction to one that doesn't need relaxation.
  if (!getBackend().mayNeedRelaxation(F.getOpcode(), F.getOperands(),
                                      *F.getSubtargetInfo()))
    return;

```
- **EN**: Implements logic around `fixupNeedsRelaxation`, `evaluateFixup`, `getBackend`, `relaxInstruction`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work; parses assembly syntax or operands; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `fixupNeedsRelaxation`, `evaluateFixup`, `getBackend`, `relaxInstruction`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，解析汇编语法或操作数，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 875-897
```cpp
  bool DoRelax = false;
  for (const MCFixup &Fixup : F.getVarFixups())
    if ((DoRelax = fixupNeedsRelaxation(F, Fixup)))
      break;
  if (!DoRelax)
    return;

  ++stats::RelaxedInstructions;

  // TODO Refactor relaxInstruction to accept MCFragment and remove
  // `setInst`.
  MCInst Relaxed = F.getInst();
  getBackend().relaxInstruction(Relaxed, *F.getSubtargetInfo());

  // Encode the new instruction.
  F.setInst(Relaxed);
  SmallVector<char, 16> Data;
  SmallVector<MCFixup, 1> Fixups;
  getEmitter().encodeInstruction(Relaxed, Data, Fixups, *F.getSubtargetInfo());
  F.setVarContents(Data);
  F.setVarFixups(Fixups);
}

```
- **EN**: Implements logic around `getInst`, `getBackend`, `setInst`, `getEmitter`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `getInst`, `getBackend`, `setInst`, `getEmitter`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为。

### Lines 898-929
```cpp
void MCAssembler::relaxLEB(MCFragment &F) {
  unsigned PadTo = F.getVarSize();
  int64_t Value;
  F.clearVarFixups();
  // Use evaluateKnownAbsolute for Mach-O as a hack: .subsections_via_symbols
  // requires that .uleb128 A-B is foldable where A and B reside in different
  // fragments. This is used by __gcc_except_table.
  bool Abs = getWriter().getSubsectionsViaSymbols()
                 ? F.getLEBValue().evaluateKnownAbsolute(Value, *this)
                 : F.getLEBValue().evaluateAsAbsolute(Value, *this);
  if (!Abs) {
    bool Relaxed, UseZeroPad;
    std::tie(Relaxed, UseZeroPad) = getBackend().relaxLEB128(F, Value);
    if (!Relaxed) {
      reportError(F.getLEBValue().getLoc(),
                  Twine(F.isLEBSigned() ? ".s" : ".u") +
                      "leb128 expression is not absolute");
      F.setLEBValue(MCConstantExpr::create(0, Context));
    }
    uint8_t Tmp[10]; // maximum size: ceil(64/7)
    PadTo = std::max(PadTo, encodeULEB128(uint64_t(Value), Tmp));
    if (UseZeroPad)
      Value = 0;
  }
  uint8_t Data[16];
  size_t Size = 0;
  // The compiler can generate EH table assembly that is impossible to assemble
  // without either adding padding to an LEB fragment or adding extra padding
  // to a later alignment fragment. To accommodate such tables, relaxation can
  // only increase an LEB fragment size here, not decrease it. See PR35809.
  if (F.isLEBSigned())
    Size = encodeSLEB128(Value, Data, PadTo);
```
- **EN**: Implements logic around `relaxLEB`, `getVarSize`, `clearVarFixups`, `getWriter`, and 8 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `relaxLEB`, `getVarSize`, `clearVarFixups`, `getWriter`, and 8 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 930-947
```cpp
  else
    Size = encodeULEB128(Value, Data, PadTo);
  F.setVarContents({reinterpret_cast<char *>(Data), Size});
}

/// Check if the branch crosses the boundary.
///
/// \param StartAddr start address of the fused/unfused branch.
/// \param Size size of the fused/unfused branch.
/// \param BoundaryAlignment alignment requirement of the branch.
/// \returns true if the branch cross the boundary.
static bool mayCrossBoundary(uint64_t StartAddr, uint64_t Size,
                             Align BoundaryAlignment) {
  uint64_t EndAddr = StartAddr + Size;
  return (StartAddr >> Log2(BoundaryAlignment)) !=
         ((EndAddr - 1) >> Log2(BoundaryAlignment));
}

```
- **EN**: Implements logic around `encodeULEB128`, `setVarContents`, `mayCrossBoundary`, `Log2`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `encodeULEB128`, `setVarContents`, `mayCrossBoundary`, `Log2` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 948-963
```cpp
/// Check if the branch is against the boundary.
///
/// \param StartAddr start address of the fused/unfused branch.
/// \param Size size of the fused/unfused branch.
/// \param BoundaryAlignment alignment requirement of the branch.
/// \returns true if the branch is against the boundary.
static bool isAgainstBoundary(uint64_t StartAddr, uint64_t Size,
                              Align BoundaryAlignment) {
  uint64_t EndAddr = StartAddr + Size;
  return (EndAddr & (BoundaryAlignment.value() - 1)) == 0;
}

/// Check if the branch needs padding.
///
/// \param StartAddr start address of the fused/unfused branch.
/// \param Size size of the fused/unfused branch.
```
- **EN**: Implements logic around `isAgainstBoundary`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `isAgainstBoundary` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 964-985
```cpp
/// \param BoundaryAlignment alignment requirement of the branch.
/// \returns true if the branch needs padding.
static bool needPadding(uint64_t StartAddr, uint64_t Size,
                        Align BoundaryAlignment) {
  return mayCrossBoundary(StartAddr, Size, BoundaryAlignment) ||
         isAgainstBoundary(StartAddr, Size, BoundaryAlignment);
}

void MCAssembler::relaxBoundaryAlign(MCBoundaryAlignFragment &BF) {
  // BoundaryAlignFragment that doesn't need to align any fragment should not be
  // relaxed.
  if (!BF.getLastFragment())
    return;

  uint64_t AlignedOffset = getFragmentOffset(BF);
  uint64_t AlignedSize = 0;
  for (const MCFragment *F = BF.getNext();; F = F->getNext()) {
    AlignedSize += computeFragmentSize(*F);
    if (F == BF.getLastFragment())
      break;
  }

```
- **EN**: Implements logic around `needPadding`, `mayCrossBoundary`, `isAgainstBoundary`, `relaxBoundaryAlign`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `needPadding`, `mayCrossBoundary`, `isAgainstBoundary`, `relaxBoundaryAlign`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 986-1010
```cpp
  Align BoundaryAlignment = BF.getAlignment();
  uint64_t NewSize = needPadding(AlignedOffset, AlignedSize, BoundaryAlignment)
                         ? offsetToAlignment(AlignedOffset, BoundaryAlignment)
                         : 0U;
  if (NewSize == BF.getSize())
    return;
  BF.setSize(NewSize);
}

void MCAssembler::relaxDwarfLineAddr(MCFragment &F) {
  if (getBackend().relaxDwarfLineAddr(F))
    return;

  MCContext &Context = getContext();
  int64_t AddrDelta;
  bool Abs = F.getDwarfAddrDelta().evaluateKnownAbsolute(AddrDelta, *this);
  assert(Abs && "We created a line delta with an invalid expression");
  (void)Abs;
  SmallVector<char, 8> Data;
  MCDwarfLineAddr::encode(Context, getDWARFLinetableParams(),
                          F.getDwarfLineDelta(), AddrDelta, Data);
  F.setVarContents(Data);
  F.clearVarFixups();
}

```
- **EN**: Implements logic around `getAlignment`, `needPadding`, `offsetToAlignment`, `setSize`, and 8 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `getAlignment`, `needPadding`, `offsetToAlignment`, `setSize`, and 8 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为。

### Lines 1011-1030
```cpp
void MCAssembler::relaxDwarfCallFrameFragment(MCFragment &F) {
  if (getBackend().relaxDwarfCFA(F))
    return;

  MCContext &Context = getContext();
  int64_t Value;
  bool Abs = F.getDwarfAddrDelta().evaluateAsAbsolute(Value, *this);
  if (!Abs) {
    reportError(F.getDwarfAddrDelta().getLoc(),
                "invalid CFI advance_loc expression");
    F.setDwarfAddrDelta(MCConstantExpr::create(0, Context));
    return;
  }

  SmallVector<char, 8> Data;
  MCDwarfFrameEmitter::encodeAdvanceLoc(Context, Value, Data);
  F.setVarContents(Data);
  F.clearVarFixups();
}

```
- **EN**: Implements logic around `relaxDwarfCallFrameFragment`, `getContext`, `getDwarfAddrDelta`, `reportError`, and 4 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `relaxDwarfCallFrameFragment`, `getContext`, `getDwarfAddrDelta`, `reportError`, and 4 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为。

### Lines 1031-1048
```cpp
void MCAssembler::relaxSFrameFragment(MCFragment &F) {
  assert(F.getKind() == MCFragment::FT_SFrame);
  MCContext &C = getContext();
  int64_t Value;
  bool Abs = F.getSFrameAddrDelta().evaluateAsAbsolute(Value, *this);
  if (!Abs) {
    C.reportError(F.getSFrameAddrDelta().getLoc(),
                  "invalid CFI advance_loc expression in sframe");
    F.setSFrameAddrDelta(MCConstantExpr::create(0, C));
    return;
  }

  SmallVector<char, 4> Data;
  MCSFrameEmitter::encodeFuncOffset(Context, Value, Data, F.getSFrameFDE());
  F.setVarContents(Data);
  F.clearVarFixups();
}

```
- **EN**: Implements logic around `relaxSFrameFragment`, `assert`, `getContext`, `getSFrameAddrDelta`, and 5 more symbols; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `relaxSFrameFragment`, `assert`, `getContext`, `getSFrameAddrDelta`, and 5 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为。

### Lines 1049-1080
```cpp
void MCAssembler::relaxFragment(MCFragment &F) {
  switch (F.getKind()) {
  default:
    return;
  case MCFragment::FT_Align:
    relaxAlign(F);
    break;
  case MCFragment::FT_Relaxable:
    assert(!getRelaxAll() && "Did not expect a FT_Relaxable in RelaxAll mode");
    relaxInstruction(F);
    break;
  case MCFragment::FT_LEB:
    relaxLEB(F);
    break;
  case MCFragment::FT_Dwarf:
    relaxDwarfLineAddr(F);
    break;
  case MCFragment::FT_DwarfFrame:
    relaxDwarfCallFrameFragment(F);
    break;
  case MCFragment::FT_SFrame:
    relaxSFrameFragment(F);
    break;
  case MCFragment::FT_BoundaryAlign:
    relaxBoundaryAlign(static_cast<MCBoundaryAlignFragment &>(F));
    break;
  case MCFragment::FT_PrefAlign:
    relaxPrefAlign(F);
    break;
  case MCFragment::FT_CVInlineLines:
    getContext().getCVContext().encodeInlineLineTable(
        *this, static_cast<MCCVInlineLineTableFragment &>(F));
```
- **EN**: Implements logic around `relaxFragment`, `relaxAlign`, `assert`, `relaxInstruction`, and 7 more symbols; this block drives emission, layout, or binary encoding behavior; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `relaxFragment`, `relaxAlign`, `assert`, `relaxInstruction`, and 7 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 1081-1098
```cpp
    break;
  case MCFragment::FT_CVDefRange:
    getContext().getCVContext().encodeDefRange(
        *this, static_cast<MCCVDefRangeFragment &>(F));
    break;
  }
}

void MCAssembler::layoutSection(MCSection &Sec) {
  uint64_t Offset = 0;
  for (MCFragment &F : Sec) {
    F.Offset = Offset;
    if (F.getKind() == MCFragment::FT_Align)
      relaxAlign(F);
    Offset += computeFragmentSize(F);
  }
}

```
- **EN**: Implements logic around `getContext`, `layoutSection`, `relaxAlign`, `computeFragmentSize`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `getContext`, `layoutSection`, `relaxAlign`, `computeFragmentSize` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1099-1122
```cpp
// Fused relaxation and layout: a single forward pass that updates each
// fragment's offset before processing it, so upstream size changes are
// immediately visible.
unsigned MCAssembler::relaxOnce(unsigned FirstStable) {
  uint64_t MaxIterations = 0;
  PendingErrors.clear();
  unsigned Res = 0;
  for (unsigned I = 0; I != FirstStable; ++I) {
    auto &Sec = *Sections[I];
    uint64_t Iters = 0;
    for (;;) {
      bool Changed = false;
      uint64_t Offset = 0;
      for (MCFragment &F : Sec) {
        if (F.Offset != Offset)
          Changed = true;
        Stretch = Offset - F.Offset;
        F.Offset = Offset;
        if (F.getKind() != MCFragment::FT_Data)
          relaxFragment(F);
        Offset += computeFragmentSize(F);
      }
      ++Iters;

```
- **EN**: Implements logic around `relaxOnce`, `clear`, `relaxFragment`, `computeFragmentSize`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `relaxOnce`, `clear`, `relaxFragment`, `computeFragmentSize` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1123-1142
```cpp
      if (!Changed)
        break;
      // If any fragment changed size, it might impact the layout of subsequent
      // sections. Therefore, we must re-evaluate all sections.
      FirstStable = Sections.size();
      Res = I;
      // Assume each iteration finalizes at least one extra fragment. If the
      // layout does not converge after N+1 iterations, bail out.
      if (Iters > Sec.curFragList()->Tail->getLayoutOrder())
        break;
    }
    MaxIterations = std::max(MaxIterations, Iters);
  }
  stats::RelaxationSteps += MaxIterations;
  Stretch = 0;
  // The subsequent relaxOnce call only needs to visit Sections [0,Res) if no
  // change occurred.
  return Res;
}

```
- **EN**: Implements logic around `size`, `max`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `size`, `max` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1143-1168
```cpp
void MCAssembler::reportError(SMLoc L, const Twine &Msg) const {
  getContext().reportError(L, Msg);
}

void MCAssembler::recordError(SMLoc Loc, const Twine &Msg) const {
  PendingErrors.emplace_back(Loc, Msg.str());
}

void MCAssembler::flushPendingErrors() const {
  for (auto &Err : PendingErrors)
    reportError(Err.first, Err.second);
  PendingErrors.clear();
}

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
LLVM_DUMP_METHOD void MCAssembler::dump() const{
  raw_ostream &OS = errs();
  DenseMap<const MCFragment *, SmallVector<const MCSymbol *, 0>> FragToSyms;
  // Scan symbols and build a map of fragments to their corresponding symbols.
  // For variable symbols, we don't want to call their getFragment, which might
  // modify `Fragment`.
  for (const MCSymbol &Sym : symbols())
    if (!Sym.isVariable())
      if (auto *F = Sym.getFragment())
        FragToSyms.try_emplace(F).first->second.push_back(&Sym);

```
- **EN**: Implements logic around `reportError`, `getContext`, `recordError`, `emplace_back`, and 5 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `reportError`, `getContext`, `recordError`, `emplace_back`, and 5 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 1169-1182
```cpp
  OS << "Sections:[";
  for (const MCSection &Sec : *this) {
    OS << '\n';
    Sec.dump(&FragToSyms);
  }
  OS << "\n]\n";
}
#endif

SMLoc MCFixup::getLoc() const {
  if (auto *E = getValue())
    return E->getLoc();
  return {};
}
```
- **EN**: Implements logic around `dump`, `getLoc`; this block handles relocation, fixup, or symbol-resolution work; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `dump`, `getLoc` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，更新 MC 节区或符号状态，返回子系统相关的计算结果。

## Key Concepts / 关键概念

- **Assembler layout / 汇编布局**:
  - **EN**: Owns fragment ordering, relaxation, fixup evaluation, and final object emission
  - **CN**: 管理片段顺序、指令松弛、fixup 求值与最终目标输出
- **Relocations and fixups / 重定位与 fixup**:
  - **EN**: Bridges symbolic references to concrete addresses or relocation records
  - **CN**: 把符号引用连接到具体地址或重定位记录
- **Directive handling / 伪指令处理**:
  - **EN**: Interprets assembler directives and maps them to streamer operations
  - **CN**: 解释汇编伪指令并把它们映射为 streamer 操作
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

- **Direct includes / 直接包含**: `llvm/MC/MCAssembler.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCCodeEmitter.h`, `llvm/MC/MCCodeView.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCDwarf.h` ... (+19 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support
