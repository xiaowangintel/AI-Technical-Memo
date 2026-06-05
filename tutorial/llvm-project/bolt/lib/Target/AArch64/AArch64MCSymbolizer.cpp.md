# AArch64MCSymbolizer.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Target/AArch64/AArch64MCSymbolizer.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Target/AArch64/AArch64MCSymbolizer.cpp. It also sits in code that implements target-specific BOLT behavior. / 该文件实现目标架构相关的 BOLT 行为。 源码头部说明其职责是：bolt/Target/AArch64/AArch64MCSymbolizer.cpp。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
//===- bolt/Target/AArch64/AArch64MCSymbolizer.cpp ------------------------===//
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
#include "AArch64MCSymbolizer.h"
#include "bolt/Core/BinaryContext.h"
#include "bolt/Core/BinaryFunction.h"
#include "bolt/Core/MCPlusBuilder.h"
#include "bolt/Core/Relocation.h"
#include "llvm/MC/MCInst.h"
#include "llvm/Support/Debug.h"
```

- EN: Pulls in 7 header(s) from local project, LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 7 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 17-29

```cpp
#define DEBUG_TYPE "bolt-symbolizer"

namespace llvm {
namespace bolt {

AArch64MCSymbolizer::~AArch64MCSymbolizer() {}

bool AArch64MCSymbolizer::tryAddingSymbolicOperand(
    MCInst &Inst, raw_ostream &CStream, int64_t Value, uint64_t InstAddress,
    bool IsBranch, uint64_t ImmOffset, uint64_t ImmSize, uint64_t InstSize) {
  BinaryContext &BC = Function.getBinaryContext();
  MCContext *Ctx = BC.Ctx.get();
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `bolt` to organize symbols. Declares or implements routines including `AArch64MCSymbolizer`. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `AArch64MCSymbolizer`。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。

### Lines 30-47

```cpp
  // NOTE: the callee may incorrectly set IsBranch.
  if (BC.MIB->isBranch(Inst) || BC.MIB->isCall(Inst))
    return false;

  const uint64_t InstOffset = InstAddress - Function.getAddress();
  const Relocation *Relocation = Function.getRelocationAt(InstOffset);

  /// Add symbolic operand to the instruction with an optional addend.
  auto addOperand = [&](const MCSymbol *Symbol, uint64_t Addend,
                        uint64_t RelType) {
    const MCExpr *Expr = MCSymbolRefExpr::create(Symbol, *Ctx);
    if (Addend)
      Expr = MCBinaryExpr::createAdd(Expr, MCConstantExpr::create(Addend, *Ctx),
                                     *Ctx);
    Inst.addOperand(MCOperand::createExpr(
        BC.MIB->getTargetExprFor(Inst, Expr, *Ctx, RelType)));
  };
```

- EN: Declares or implements routines including `create`, `createAdd`, `getTargetExprFor`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `create`, `createAdd`, `getTargetExprFor`.
- CN: 这里声明或实现函数，例如 `create`, `createAdd`, `getTargetExprFor`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `create`, `createAdd`, `getTargetExprFor`。

### Lines 48-58

```cpp
  if (Relocation) {
    auto AdjustedRel = adjustRelocation(*Relocation, Inst);
    if (AdjustedRel) {
      addOperand(AdjustedRel->Symbol, AdjustedRel->Addend, AdjustedRel->Type);
      return true;
    }

    LLVM_DEBUG(dbgs() << "BOLT-DEBUG: ignoring relocation at 0x"
                      << Twine::utohexstr(InstAddress) << '\n');
  }
```

- EN: Declares or implements routines including `adjustRelocation`, `addOperand`, `LLVM_DEBUG`, `utohexstr`. Notable symbols here include `adjustRelocation`, `addOperand`, `LLVM_DEBUG`, `utohexstr`.
- CN: 这里声明或实现函数，例如 `adjustRelocation`, `addOperand`, `LLVM_DEBUG`, `utohexstr`。这里较值得关注的符号包括 `adjustRelocation`, `addOperand`, `LLVM_DEBUG`, `utohexstr`。

### Lines 59-76

```cpp
  if (!BC.MIB->hasPCRelOperand(Inst))
    return false;

  Value += InstAddress;
  const MCSymbol *TargetSymbol;
  uint64_t TargetOffset;
  if (!CreateNewSymbols) {
    if (BinaryData *BD = BC.getBinaryDataContainingAddress(Value)) {
      TargetSymbol = BD->getSymbol();
      TargetOffset = Value - BD->getAddress();
    } else {
      return false;
    }
  } else {
    std::tie(TargetSymbol, TargetOffset) =
        BC.handleAddressRef(Value, Function, /*IsPCRel*/ true);
  }
```

- EN: Declares or implements routines including `getSymbol`, `getAddress`, `tie`. Notable symbols here include `getSymbol`, `getAddress`, `tie`.
- CN: 这里声明或实现函数，例如 `getSymbol`, `getAddress`, `tie`。这里较值得关注的符号包括 `getSymbol`, `getAddress`, `tie`。

### Lines 77-86

```cpp
  addOperand(TargetSymbol, TargetOffset, 0);

  return true;
}

std::optional<Relocation>
AArch64MCSymbolizer::adjustRelocation(const Relocation &Rel,
                                      const MCInst &Inst) const {
  BinaryContext &BC = Function.getBinaryContext();
```

- EN: Declares or implements routines including `addOperand`. Notable symbols here include `addOperand`.
- CN: 这里声明或实现函数，例如 `addOperand`。这里较值得关注的符号包括 `addOperand`。

### Lines 87-96

```cpp
  // The linker can convert ADRP+ADD and ADRP+LDR instruction sequences into
  // NOP+ADR. After the conversion, the linker might keep the relocations and
  // if we try to symbolize ADR's operand using outdated relocations, we might
  // get unexpected results. Hence, we check for the conversion/relaxation, and
  // ignore the relocation. The symbolization is done based on the PC-relative
  // value of the operand instead.
  if (BC.MIB->isADR(Inst) && (Rel.Type == ELF::R_AARCH64_ADD_ABS_LO12_NC ||
                              Rel.Type == ELF::R_AARCH64_LD64_GOT_LO12_NC))
    return std::nullopt;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 97-114

```cpp
  // The linker might perform TLS relocations relaxations, such as changed TLS
  // access model (e.g. changed global dynamic model to initial exec), thus
  // changing the instructions. The static relocations might be invalid at this
  // point and we don't have to process these relocations anymore. More
  // information could be found by searching elfNN_aarch64_tls_relax in bfd.
  if (BC.MIB->isMOVW(Inst)) {
    switch (Rel.Type) {
    default:
      break;
    case ELF::R_AARCH64_TLSDESC_LD64_LO12:
    case ELF::R_AARCH64_TLSDESC_ADD_LO12:
    case ELF::R_AARCH64_TLSDESC_ADR_PAGE21:
    case ELF::R_AARCH64_TLSIE_LD64_GOTTPREL_LO12_NC:
    case ELF::R_AARCH64_TLSIE_ADR_GOTTPREL_PAGE21:
      return std::nullopt;
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 115-125

```cpp
  if (!Relocation::isGOT(Rel.Type))
    return Rel;

  Relocation AdjustedRel = Rel;
  if (Rel.Type == ELF::R_AARCH64_LD64_GOT_LO12_NC && BC.MIB->isAddXri(Inst)) {
    // The ADRP+LDR sequence was converted into ADRP+ADD. We are looking at the
    // second instruction and have to use the relocation type for ADD.
    AdjustedRel.Type = ELF::R_AARCH64_ADD_ABS_LO12_NC;
    return AdjustedRel;
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 126-142

```cpp
  // ADRP is a special case since the linker can leave the instruction opcode
  // intact and modify only the operand. We are doing our best to detect when
  // such conversion has happened without looking at the next instruction.
  //
  // If we detect that a page referenced by the ADRP cannot belong to GOT, and
  // that it matches the symbol from the relocation, then we can be certain
  // that the linker converted the GOT reference into the local one. Otherwise,
  // we leave the disambiguation resolution to FixRelaxationPass.
  //
  // Note that ADRP relaxation described above cannot happen for TLS relocation.
  // Since TLS relocations may not even have a valid symbol (not supported by
  // BOLT), we explicitly exclude them from the check.
  if (BC.MIB->isADRP(Inst) && Rel.Addend == 0 && !Relocation::isTLS(Rel.Type)) {
    ErrorOr<uint64_t> SymbolValue = BC.getSymbolValue(*Rel.Symbol);
    assert(SymbolValue && "Symbol value should be set");
    const uint64_t SymbolPageAddr = *SymbolValue & ~0xfffULL;
```

- EN: Declares or implements routines including `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`。

### Lines 143-155

```cpp
    if (SymbolPageAddr == Rel.Value &&
        !isPageAddressValidForGOT(SymbolPageAddr)) {
      AdjustedRel.Type = ELF::R_AARCH64_ADR_PREL_PG_HI21;
      return AdjustedRel;
    }
  }

  // For instructions that reference GOT, ignore the referenced symbol and
  // use value at the relocation site. FixRelaxationPass will look at
  // instruction pairs and will perform necessary adjustments.
  AdjustedRel.Symbol = BC.registerNameAtAddress("__BOLT_got_zero", 0, 0, 0);
  AdjustedRel.Addend = Rel.Value;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 156-166

```cpp
  return AdjustedRel;
}

bool AArch64MCSymbolizer::isPageAddressValidForGOT(uint64_t PageAddress) const {
  assert(!(PageAddress & 0xfffULL) && "Page address not aligned at 4KB");

  ErrorOr<BinarySection &> GOT =
      Function.getBinaryContext().getUniqueSectionByName(".got");
  if (!GOT || !GOT->getSize())
    return false;
```

- EN: Declares or implements routines including `isPageAddressValidForGOT`, `assert`. Notable symbols here include `isPageAddressValidForGOT`, `assert`.
- CN: 这里声明或实现函数，例如 `isPageAddressValidForGOT`, `assert`。这里较值得关注的符号包括 `isPageAddressValidForGOT`, `assert`。

### Lines 167-174

```cpp
  const uint64_t GOTFirstPageAddress = GOT->getAddress() & ~0xfffULL;
  const uint64_t GOTLastPageAddress =
      (GOT->getAddress() + GOT->getSize() - 1) & ~0xfffULL;

  return PageAddress >= GOTFirstPageAddress &&
         PageAddress <= GOTLastPageAddress;
}
```

- EN: Declares or implements routines including `getAddress`. Notable symbols here include `getAddress`.
- CN: 这里声明或实现函数，例如 `getAddress`。这里较值得关注的符号包括 `getAddress`。

### Lines 175-180

```cpp
void AArch64MCSymbolizer::tryAddingPcLoadReferenceComment(raw_ostream &CStream,
                                                          int64_t Value,
                                                          uint64_t Address) {}

} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `AArch64MCSymbolizer`: function or method entry point / 函数或方法入口
- `create`: function or method entry point / 函数或方法入口
- `createAdd`: function or method entry point / 函数或方法入口
- `getTargetExprFor`: function or method entry point / 函数或方法入口
- `adjustRelocation`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域
- `bolt`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Core/BinaryContext.h`, `bolt/Core/BinaryFunction.h`, `bolt/Core/MCPlusBuilder.h`, `bolt/Core/Relocation.h`
- LLVM headers / LLVM 头文件: `llvm/MC/MCInst.h`, `llvm/Support/Debug.h`
- System headers / 系统头文件: `AArch64MCSymbolizer.h`
- Directory context / 目录上下文: `bolt/lib/Target/AArch64` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Target/AArch64` 下的相邻文件通常与本文件协作组成对应子系统
