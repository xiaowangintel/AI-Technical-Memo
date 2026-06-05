# X86MCSymbolizer.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Target/X86/X86MCSymbolizer.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Target/X86/X86MCSymbolizer.cpp. It also sits in code that implements target-specific BOLT behavior. / 该文件实现目标架构相关的 BOLT 行为。 源码头部说明其职责是：bolt/Target/X86/X86MCSymbolizer.cpp。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
//===- bolt/Target/X86/X86MCSymbolizer.cpp --------------------------------===//
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
#include "X86MCSymbolizer.h"
#include "MCTargetDesc/X86BaseInfo.h"
#include "bolt/Core/BinaryContext.h"
#include "bolt/Core/BinaryFunction.h"
#include "bolt/Core/MCPlusBuilder.h"
#include "bolt/Core/Relocation.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCRegisterInfo.h"
```

- EN: Pulls in 8 header(s) from local project, LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 8 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 18-30

```cpp
#define DEBUG_TYPE "bolt-symbolizer"

namespace llvm {
namespace bolt {

X86MCSymbolizer::~X86MCSymbolizer() {}

bool X86MCSymbolizer::tryAddingSymbolicOperand(
    MCInst &Inst, raw_ostream &CStream, int64_t Value, uint64_t InstAddress,
    bool IsBranch, uint64_t ImmOffset, uint64_t ImmSize, uint64_t InstSize) {
  if (IsBranch)
    return false;
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `bolt` to organize symbols. Declares or implements routines including `X86MCSymbolizer`. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `X86MCSymbolizer`。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。

### Lines 31-40

```cpp
  // Ignore implicit operands.
  if (ImmSize == 0)
    return false;

  BinaryContext &BC = Function.getBinaryContext();
  MCContext *Ctx = BC.Ctx.get();

  if (BC.MIB->isBranch(Inst) || BC.MIB->isCall(Inst))
    return false;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 41-49

```cpp
  /// Add symbolic operand to the instruction with an optional addend.
  auto addOperand = [&](const MCSymbol *Symbol, uint64_t Addend) {
    const MCExpr *Expr = MCSymbolRefExpr::create(Symbol, *Ctx);
    if (Addend)
      Expr = MCBinaryExpr::createAdd(Expr, MCConstantExpr::create(Addend, *Ctx),
                                     *Ctx);
    Inst.addOperand(MCOperand::createExpr(Expr));
  };
```

- EN: Declares or implements routines including `create`, `createAdd`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `create`, `createAdd`.
- CN: 这里声明或实现函数，例如 `create`, `createAdd`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `create`, `createAdd`。

### Lines 50-57

```cpp
  // Check if the operand being added is a displacement part of a compound
  // memory operand that uses PC-relative addressing. If it is, try to symbolize
  // it without relocations. Return true on success, false otherwise.
  auto processPCRelOperandNoRel = [&]() {
    const int MemOp = BC.MIB->getMemoryOperandNo(Inst);
    if (MemOp == -1)
      return false;
```

- EN: Declares or implements routines including `getMemoryOperandNo`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getMemoryOperandNo`.
- CN: 这里声明或实现函数，例如 `getMemoryOperandNo`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getMemoryOperandNo`。

### Lines 58-65

```cpp
    const unsigned DispOp = MemOp + X86::AddrDisp;
    if (Inst.getNumOperands() != DispOp)
      return false;

    const MCOperand &Base = Inst.getOperand(MemOp + X86::AddrBaseReg);
    if (Base.getReg() != BC.MRI->getProgramCounter())
      return false;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 66-73

```cpp
    const MCOperand &Scale = Inst.getOperand(MemOp + X86::AddrScaleAmt);
    const MCOperand &Index = Inst.getOperand(MemOp + X86::AddrIndexReg);
    if (Scale.getImm() != 0 && Index.getReg() != MCRegister::NoRegister)
      return false;

    const MCSymbol *TargetSymbol;
    uint64_t TargetOffset;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 74-85

```cpp
    if (!CreateNewSymbols) {
      if (BinaryData *BD = BC.getBinaryDataContainingAddress(Value)) {
        TargetSymbol = BD->getSymbol();
        TargetOffset = Value - BD->getAddress();
      } else {
        return false;
      }
    } else {
      std::tie(TargetSymbol, TargetOffset) =
          BC.handleAddressRef(Value, Function, /*IsPCRel=*/true);
    }
```

- EN: Declares or implements routines including `getSymbol`, `getAddress`, `tie`. Notable symbols here include `getSymbol`, `getAddress`, `tie`.
- CN: 这里声明或实现函数，例如 `getSymbol`, `getAddress`, `tie`。这里较值得关注的符号包括 `getSymbol`, `getAddress`, `tie`。

### Lines 86-103

```cpp
    addOperand(TargetSymbol, TargetOffset);

    return true;
  };

  // Check for GOTPCRELX relocations first. Because these relocations allow the
  // linker to modify the instruction, we have to check the offset range
  // corresponding to the instruction, not the offset of the operand.
  // Note that if there is GOTPCRELX relocation against the instruction, there
  // will be no other relocation in this range, since GOTPCRELX applies only to
  // certain instruction types.
  const uint64_t InstOffset = InstAddress - Function.getAddress();
  const Relocation *Relocation =
      Function.getRelocationInRange(InstOffset, InstOffset + InstSize);
  if (Relocation && Relocation::isX86GOTPCRELX(Relocation->Type)) {
    // If the operand is PC-relative, convert it without using the relocation
    // information. For GOTPCRELX, it is safe to use the absolute address
    // instead of extracting the addend from the relocation, as non-standard
```

- EN: Declares or implements routines including `addOperand`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `addOperand`.
- CN: 这里声明或实现函数，例如 `addOperand`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `addOperand`。

### Lines 104-113

```cpp
    // forms will be rejected by linker conversion process and the operand
    // will always reference GOT which we don't rewrite.
    if (processPCRelOperandNoRel())
      return true;

    // The linker converted the PC-relative address to an absolute one.
    // Symbolize this address.
    if (CreateNewSymbols)
      BC.handleAddressRef(Value, Function, /*IsPCRel=*/false);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 114-122

```cpp
    const BinaryData *Target = BC.getBinaryDataAtAddress(Value);
    if (!Target) {
      assert(!CreateNewSymbols &&
             "BinaryData should exist at converted GOTPCRELX destination");
      return false;
    }

    addOperand(Target->getSymbol(), /*Addend=*/0);
```

- EN: Declares or implements routines including `addOperand`. Notable symbols here include `addOperand`.
- CN: 这里声明或实现函数，例如 `addOperand`。这里较值得关注的符号包括 `addOperand`。

### Lines 123-132

```cpp
    return true;
  }

  // Check for relocations against the operand.
  if (!Relocation || Relocation->Offset != InstOffset + ImmOffset)
    Relocation = Function.getRelocationAt(InstOffset + ImmOffset);

  if (!Relocation)
    return processPCRelOperandNoRel();
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 133-147

```cpp
  // GOTPC64 is special because the X86 Assembler doesn't know how to emit
  // a PC-relative 8-byte fixup, which is what we need to cover this. The
  // only way to do this is to use the symbol name _GLOBAL_OFFSET_TABLE_.
  if (Relocation::isX86GOTPC64(Relocation->Type)) {
    auto PairOrErr = handleGOTPC64(*Relocation, InstAddress);
    if (auto E = PairOrErr.takeError()) {
      Function.setSimple(false);
      BC.logBOLTErrorsAndQuitOnFatal(std::move(E));
      return false;
    }
    auto [Sym, Addend] = *PairOrErr;
    addOperand(Sym, Addend);
    return true;
  }
```

- EN: Declares or implements routines including `handleGOTPC64`, `addOperand`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `handleGOTPC64`, `addOperand`.
- CN: 这里声明或实现函数，例如 `handleGOTPC64`, `addOperand`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `handleGOTPC64`, `addOperand`。

### Lines 148-155

```cpp
  uint64_t SymbolValue = Relocation->Value - Relocation->Addend;
  if (Relocation->isPCRelative())
    SymbolValue += InstAddress + ImmOffset;

  // Process reference to the symbol.
  if (CreateNewSymbols)
    BC.handleAddressRef(SymbolValue, Function, Relocation->isPCRelative());
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 156-163

```cpp
  uint64_t Addend = Relocation->Addend;
  // Real addend for pc-relative targets is adjusted with a delta from
  // the relocation placement to the next instruction.
  if (Relocation->isPCRelative())
    Addend += InstOffset + InstSize - Relocation->Offset;

  addOperand(Relocation->Symbol, Addend);
```

- EN: Declares or implements routines including `addOperand`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `addOperand`.
- CN: 这里声明或实现函数，例如 `addOperand`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `addOperand`。

### Lines 164-181

```cpp
  return true;
}

Expected<std::pair<MCSymbol *, uint64_t>>
X86MCSymbolizer::handleGOTPC64(const Relocation &R, uint64_t InstrAddr) {
  BinaryContext &BC = Function.getBinaryContext();
  const BinaryData *GOTSymBD = BC.getGOTSymbol();
  if (!GOTSymBD || !GOTSymBD->getAddress()) {
    // This error is pretty serious but we can't kill the disassembler
    // because of it, so don't make it fatal. Log it and warn the user.
    return createNonFatalBOLTError(
        "R_X86_GOTPC64 relocation is present but we did not detect "
        "a valid  _GLOBAL_OFFSET_TABLE_ in symbol table\n");
  }
  // R_X86_GOTPC64 are not relative to the Reloc nor end of instruction,
  // but the start of the MOVABSQ instruction. So the Target Address is
  // whatever is encoded in the original operand when we disassembled
  // the binary (here, R.Value) plus MOVABSQ address (InstrAddr).
```

- EN: Declares or implements routines including `handleGOTPC64`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `handleGOTPC64`.
- CN: 这里声明或实现函数，例如 `handleGOTPC64`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `handleGOTPC64`。

### Lines 182-192

```cpp
  // Here we extract the intended Addend by subtracting the real
  // GOT addr.
  const int64_t Addend = R.Value + InstrAddr - GOTSymBD->getAddress();
  return std::make_pair(BC.Ctx->getOrCreateSymbol("_GLOBAL_OFFSET_TABLE_"),
                        Addend);
}

void X86MCSymbolizer::tryAddingPcLoadReferenceComment(raw_ostream &CStream,
                                                      int64_t Value,
                                                      uint64_t Address) {}
```

- EN: Declares or implements routines including `getAddress`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getAddress`.
- CN: 这里声明或实现函数，例如 `getAddress`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getAddress`。

### Lines 193-194

```cpp
} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `X86MCSymbolizer`: function or method entry point / 函数或方法入口
- `create`: function or method entry point / 函数或方法入口
- `createAdd`: function or method entry point / 函数或方法入口
- `getMemoryOperandNo`: function or method entry point / 函数或方法入口
- `getSymbol`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域
- `bolt`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Core/BinaryContext.h`, `bolt/Core/BinaryFunction.h`, `bolt/Core/MCPlusBuilder.h`, `bolt/Core/Relocation.h`
- LLVM headers / LLVM 头文件: `llvm/MC/MCInst.h`, `llvm/MC/MCRegisterInfo.h`
- System headers / 系统头文件: `X86MCSymbolizer.h`, `MCTargetDesc/X86BaseInfo.h`
- Directory context / 目录上下文: `bolt/lib/Target/X86` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Target/X86` 下的相邻文件通常与本文件协作组成对应子系统
