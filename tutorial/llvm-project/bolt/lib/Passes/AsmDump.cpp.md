# AsmDump.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/AsmDump.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: Dump BinaryFunction into assembly. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：Dump BinaryFunction into assembly。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Passes/AsmDump.cpp - Dump BinaryFunction into assembly --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the AsmDumpPass class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-20

```cpp
#include "bolt/Passes/AsmDump.h"
#include "llvm/CodeGen/AsmPrinter.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/FileSystem.h"
#include "llvm/Support/Path.h"
#include "llvm/Target/TargetMachine.h"
#include <unordered_set>
```

- EN: Pulls in 7 header(s) from local project, LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 7 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 21-29

```cpp
#define DEBUG_TYPE "asm-dump"

using namespace llvm;

namespace opts {
extern bool shouldPrint(const bolt::BinaryFunction &Function);
extern cl::OptionCategory BoltCategory;
extern cl::opt<unsigned> Verbosity;
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `opts` to organize symbols. Declares or implements routines including `shouldPrint`. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `opts` 中，用于组织符号作用域。这里声明或实现函数，例如 `shouldPrint`。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。

### Lines 30-38

```cpp
cl::opt<std::string> AsmDump("asm-dump",
                             cl::desc("dump function into assembly"),
                             cl::value_desc("dump folder"), cl::ValueOptional,
                             cl::Hidden, cl::cat(BoltCategory));
} // end namespace opts

namespace llvm {
namespace bolt {
```

- EN: Works inside namespace scope `opts`, `llvm`, `bolt` to organize symbols. Declares or implements routines including `desc`, `value_desc`, `cat`. Notable symbols here include `desc`, `value_desc`, `cat`, `opts`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `opts`, `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `desc`, `value_desc`, `cat`。这里较值得关注的符号包括 `desc`, `value_desc`, `cat`, `opts`, `llvm`, `bolt`。

### Lines 39-49

```cpp
void dumpCFI(const BinaryFunction &BF, const MCInst &Instr, AsmPrinter &MAP) {
  const MCCFIInstruction *CFIInstr = BF.getCFIFor(Instr);
  switch (CFIInstr->getOperation()) {
  // Skip unsupported CFI instructions.
  case MCCFIInstruction::OpRememberState:
  case MCCFIInstruction::OpRestoreState:
    if (opts::Verbosity >= 2)
      BF.getBinaryContext().errs()
          << "BOLT-WARNING: AsmDump: skipping unsupported CFI instruction in "
          << BF << ".\n";
```

- EN: Declares or implements routines including `dumpCFI`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `dumpCFI`.
- CN: 这里声明或实现函数，例如 `dumpCFI`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `dumpCFI`。

### Lines 50-57

```cpp
    return;

  default:
    // Emit regular CFI instructions.
    MAP.emitCFIInstruction(*CFIInstr);
  }
}
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 58-72

```cpp
void dumpTargetFunctionStub(raw_ostream &OS, const BinaryContext &BC,
                            const MCSymbol *CalleeSymb,
                            const BinarySection *&LastCS) {
  const BinaryFunction *CalleeFunc = BC.getFunctionForSymbol(CalleeSymb);
  if (!CalleeFunc || CalleeFunc->isPLTFunction())
    return;

  if (CalleeFunc->getOriginSection() != LastCS) {
    OS << ".section " << CalleeFunc->getOriginSectionName() << '\n';
    LastCS = CalleeFunc->getOriginSection();
  }
  StringRef CalleeName = CalleeFunc->getOneName();
  OS << ".set \"" << CalleeName << "\", 0\n";
}
```

- EN: Declares or implements routines including `getOriginSectionName`, `getOriginSection`, `getOneName`. Notable symbols here include `getOriginSectionName`, `getOriginSection`, `getOneName`.
- CN: 这里声明或实现函数，例如 `getOriginSectionName`, `getOriginSection`, `getOneName`。这里较值得关注的符号包括 `getOriginSectionName`, `getOriginSection`, `getOneName`。

### Lines 73-84

```cpp
void dumpJumpTableSymbols(raw_ostream &OS, const JumpTable *JT, AsmPrinter &MAP,
                          const BinarySection *&LastBS) {
  if (&JT->getSection() != LastBS) {
    OS << ".section " << JT->getSectionName() << '\n';
    LastBS = &JT->getSection();
  }
  OS << "\"" << JT->getName() << "\":\n";
  for (MCSymbol *JTEntry : JT->Entries)
    MAP.OutStreamer->emitSymbolValue(JTEntry, JT->OutputEntrySize);
  OS << '\n';
}
```

- EN: Declares or implements routines including `getSectionName`, `getSection`, `getName`, `emitSymbolValue`. Notable symbols here include `getSectionName`, `getSection`, `getName`, `emitSymbolValue`.
- CN: 这里声明或实现函数，例如 `getSectionName`, `getSection`, `getName`, `emitSymbolValue`。这里较值得关注的符号包括 `getSectionName`, `getSection`, `getName`, `emitSymbolValue`。

### Lines 85-96

```cpp
void dumpBinaryDataSymbols(raw_ostream &OS, const BinaryData *BD,
                           const BinarySection *&LastBS) {
  if (BD->isJumpTable())
    return;
  if (&BD->getSection() != LastBS) {
    OS << ".section " << BD->getSectionName() << '\n';
    LastBS = &BD->getSection();
  }
  OS << "\"" << BD->getName() << "\": ";
  OS << '\n';
}
```

- EN: Declares or implements routines including `getSectionName`, `getSection`, `getName`. Notable symbols here include `getSectionName`, `getSection`, `getName`.
- CN: 这里声明或实现函数，例如 `getSectionName`, `getSection`, `getName`。这里较值得关注的符号包括 `getSectionName`, `getSection`, `getName`。

### Lines 97-110

```cpp
void dumpFunction(const BinaryFunction &BF) {
  const BinaryContext &BC = BF.getBinaryContext();
  if (!opts::shouldPrint(BF))
    return;

  // Make sure the new directory exists, creating it if necessary.
  if (!opts::AsmDump.empty()) {
    if (std::error_code EC = sys::fs::create_directories(opts::AsmDump)) {
      BC.errs() << "BOLT-ERROR: could not create directory '" << opts::AsmDump
                << "': " << EC.message() << '\n';
      return;
    }
  }
```

- EN: Declares or implements routines including `dumpFunction`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `dumpFunction`.
- CN: 这里声明或实现函数，例如 `dumpFunction`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `dumpFunction`。

### Lines 111-119

```cpp
  std::string PrintName = BF.getPrintName();
  llvm::replace(PrintName, '/', '-');
  std::string Filename =
      opts::AsmDump.empty()
          ? (PrintName + ".s")
          : (opts::AsmDump + sys::path::get_separator() + PrintName + ".s")
                .str();
  BC.outs() << "BOLT-INFO: Dumping function assembly to " << Filename << "\n";
```

- EN: Declares or implements routines including `replace`. Notable symbols here include `replace`.
- CN: 这里声明或实现函数，例如 `replace`。这里较值得关注的符号包括 `replace`。

### Lines 120-128

```cpp
  std::error_code EC;
  raw_fd_ostream OS(Filename, EC, sys::fs::OF_None);
  if (EC) {
    BC.errs() << "BOLT-ERROR: " << EC.message() << ", unable to open "
              << Filename << " for output.\n";
    return;
  }
  OS.SetUnbuffered();
```

- EN: Declares or implements routines including `OS`. Notable symbols here include `OS`.
- CN: 这里声明或实现函数，例如 `OS`。这里较值得关注的符号包括 `OS`。

### Lines 129-146

```cpp
  // Create local MC context to isolate the effect of ephemeral assembly
  // emission.
  BinaryContext::IndependentCodeEmitter MCEInstance =
      BC.createIndependentMCCodeEmitter();
  MCContext *LocalCtx = MCEInstance.LocalCtx.get();
  std::unique_ptr<MCAsmBackend> MAB(
      BC.TheTarget->createMCAsmBackend(*BC.STI, *BC.MRI, MCTargetOptions()));
  int AsmPrinterVariant = BC.AsmInfo->getAssemblerDialect();
  std::unique_ptr<MCInstPrinter> InstructionPrinter(
      BC.TheTarget->createMCInstPrinter(*BC.TheTriple, AsmPrinterVariant,
                                        *BC.AsmInfo, *BC.MII, *BC.MRI));
  auto FOut = std::make_unique<formatted_raw_ostream>(OS);
  FOut->SetUnbuffered();
  std::unique_ptr<MCStreamer> AsmStreamer(createAsmStreamer(
      *LocalCtx, std::move(FOut), std::move(InstructionPrinter),
      std::move(MCEInstance.MCE), std::move(MAB)));
  AsmStreamer->initSections(*BC.STI);
  std::unique_ptr<TargetMachine> TM(BC.TheTarget->createTargetMachine(
```

- EN: Declares or implements routines including `createMCAsmBackend`, `getAssemblerDialect`, `SetUnbuffered`, `move`, `initSections`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `createMCAsmBackend`, `getAssemblerDialect`, `SetUnbuffered`, `move`, `initSections`.
- CN: 这里声明或实现函数，例如 `createMCAsmBackend`, `getAssemblerDialect`, `SetUnbuffered`, `move`, `initSections`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `createMCAsmBackend`, `getAssemblerDialect`, `SetUnbuffered`, `move`, `initSections`。

### Lines 147-155

```cpp
      *BC.TheTriple, "", "", TargetOptions(), std::nullopt));
  std::unique_ptr<AsmPrinter> MAP(
      BC.TheTarget->createAsmPrinter(*TM, std::move(AsmStreamer)));

  StringRef FunctionName = BF.getOneName();
  OS << "  .globl " << FunctionName << '\n';
  OS << "  .type " << FunctionName << ", %function\n";
  OS << FunctionName << ":\n";
```

- EN: Declares or implements routines including `createAsmPrinter`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `createAsmPrinter`.
- CN: 这里声明或实现函数，例如 `createAsmPrinter`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `createAsmPrinter`。

### Lines 156-166

```cpp
  // FDATA for the entry point
  if (uint64_t EntryExecCount = BF.getKnownExecutionCount())
    OS << "# FDATA: 0 [unknown] 0 "
       << "1 " << FunctionName << " 0 "
       << "0 " << EntryExecCount << '\n';

  // Binary data references from the function.
  std::unordered_set<const BinaryData *> BDReferences;
  // Function references from the function (to avoid constructing call graph).
  std::unordered_set<const MCSymbol *> CallReferences;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 167-181

```cpp
  MAP->OutStreamer->emitCFIStartProc(/*IsSimple=*/false);
  for (const BinaryBasicBlock *BB : BF.getLayout().blocks()) {
    OS << BB->getName() << ": \n";

    const std::string BranchLabel = Twine(BB->getName(), "_br").str();
    const MCInst *LastInst = BB->getLastNonPseudoInstr();

    for (const MCInst &Instr : *BB) {
      // Dump pseudo instructions (CFI)
      if (BC.MIB->isPseudo(Instr)) {
        if (BC.MIB->isCFI(Instr))
          dumpCFI(BF, Instr, *MAP);
        continue;
      }
```

- EN: Declares or implements routines including `emitCFIStartProc`, `getName`, `Twine`, `getLastNonPseudoInstr`, `dumpCFI`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `emitCFIStartProc`, `getName`, `Twine`, `getLastNonPseudoInstr`, `dumpCFI`.
- CN: 这里声明或实现函数，例如 `emitCFIStartProc`, `getName`, `Twine`, `getLastNonPseudoInstr`, `dumpCFI`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `emitCFIStartProc`, `getName`, `Twine`, `getLastNonPseudoInstr`, `dumpCFI`。

### Lines 182-197

```cpp
      // Analyze symbol references (data, functions) from the instruction.
      bool IsCall = BC.MIB->isCall(Instr);
      for (const MCOperand &Operand : MCPlus::primeOperands(Instr)) {
        if (Operand.isExpr() &&
            Operand.getExpr()->getKind() == MCExpr::SymbolRef) {
          std::pair<const MCSymbol *, uint64_t> TSI =
              BC.MIB->getTargetSymbolInfo(Operand.getExpr());
          const MCSymbol *Symbol = TSI.first;
          if (IsCall)
            CallReferences.insert(Symbol);
          else if (const BinaryData *BD =
                       BC.getBinaryDataByName(Symbol->getName()))
            BDReferences.insert(BD);
        }
      }
```

- EN: Declares or implements routines including `isCall`, `getTargetSymbolInfo`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isCall`, `getTargetSymbolInfo`.
- CN: 这里声明或实现函数，例如 `isCall`, `getTargetSymbolInfo`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isCall`, `getTargetSymbolInfo`。

### Lines 198-210

```cpp
      if (&Instr == LastInst && (BB->succ_size() || IsCall))
        OS << BranchLabel << ":\n";

      BC.InstPrinter->printInst(&Instr, 0, "", *BC.STI, OS);
      OS << '\n';
    }

    // Dump profile data in FDATA format (as parsed by link_fdata).
    for (const BinaryBasicBlock *Succ : BB->successors()) {
      const BinaryBasicBlock::BinaryBranchInfo BI = BB->getBranchInfo(*Succ);
      if (!BI.MispredictedCount && !BI.Count)
        continue;
```

- EN: Declares or implements routines including `printInst`, `getBranchInfo`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `printInst`, `getBranchInfo`.
- CN: 这里声明或实现函数，例如 `printInst`, `getBranchInfo`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `printInst`, `getBranchInfo`。

### Lines 211-219

```cpp
      OS << "# FDATA: 1 " << FunctionName << " #" << BranchLabel << "# "
         << "1 " << FunctionName << " #" << Succ->getName() << "# "
         << BI.MispredictedCount << " " << BI.Count << '\n';
    }

    OS << '\n';
  }
  MAP->OutStreamer->emitCFIEndProc();
```

- EN: Declares or implements routines including `getName`, `emitCFIEndProc`. Notable symbols here include `getName`, `emitCFIEndProc`.
- CN: 这里声明或实现函数，例如 `getName`, `emitCFIEndProc`。这里较值得关注的符号包括 `getName`, `emitCFIEndProc`。

### Lines 220-231

```cpp
  OS << ".size " << FunctionName << ", .-" << FunctionName << '\n';

  const BinarySection *LastSection = BF.getOriginSection();
  // Print stubs for all target functions.
  for (const MCSymbol *CalleeSymb : CallReferences)
    dumpTargetFunctionStub(OS, BC, CalleeSymb, LastSection);

  OS << "# Jump tables\n";
  // Print all jump tables.
  for (auto &JTI : BF.jumpTables())
    dumpJumpTableSymbols(OS, JTI.second, *MAP, LastSection);
```

- EN: Declares or implements routines including `dumpTargetFunctionStub`, `dumpJumpTableSymbols`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `dumpTargetFunctionStub`, `dumpJumpTableSymbols`.
- CN: 这里声明或实现函数，例如 `dumpTargetFunctionStub`, `dumpJumpTableSymbols`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `dumpTargetFunctionStub`, `dumpJumpTableSymbols`。

### Lines 232-243

```cpp
  OS << "# BinaryData\n";
  // Print data references.
  for (const BinaryData *BD : BDReferences)
    dumpBinaryDataSymbols(OS, BD, LastSection);
}

Error AsmDumpPass::runOnFunctions(BinaryContext &BC) {
  for (const auto &BFIt : BC.getBinaryFunctions())
    dumpFunction(BFIt.second);
  return Error::success();
}
```

- EN: Declares or implements routines including `dumpBinaryDataSymbols`, `runOnFunctions`, `dumpFunction`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `dumpBinaryDataSymbols`, `runOnFunctions`, `dumpFunction`.
- CN: 这里声明或实现函数，例如 `dumpBinaryDataSymbols`, `runOnFunctions`, `dumpFunction`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `dumpBinaryDataSymbols`, `runOnFunctions`, `dumpFunction`。

### Lines 244-245

```cpp
} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `shouldPrint`: function or method entry point / 函数或方法入口
- `desc`: function or method entry point / 函数或方法入口
- `value_desc`: function or method entry point / 函数或方法入口
- `cat`: function or method entry point / 函数或方法入口
- `dumpCFI`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域
- `opts`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/AsmDump.h`
- LLVM headers / LLVM 头文件: `llvm/CodeGen/AsmPrinter.h`, `llvm/MC/TargetRegistry.h`, `llvm/Support/FileSystem.h`, `llvm/Support/Path.h`, `llvm/Target/TargetMachine.h`
- System headers / 系统头文件: `unordered_set`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
