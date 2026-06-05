# RegReAssign.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/RegReAssign.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Passes/RegReAssign.cpp This file implements the RegReAssign class.. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：bolt/Passes/RegReAssign.cpp This file implements the RegReAssign class.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Passes/RegReAssign.cpp ----------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the RegReAssign class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-20

```cpp
#include "bolt/Passes/RegReAssign.h"
#include "bolt/Core/BinaryFunctionCallGraph.h"
#include "bolt/Core/MCPlus.h"
#include "bolt/Passes/DataflowAnalysis.h"
#include "bolt/Passes/DataflowInfoManager.h"
#include "bolt/Utils/Utils.h"
#include <numeric>
```

- EN: Pulls in 7 header(s) from local project, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 7 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 21-28

```cpp
#define DEBUG_TYPE "regreassign"

using namespace llvm;

namespace opts {
extern cl::OptionCategory BoltOptCategory;
extern cl::opt<bool> UpdateDebugSections;
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `opts` to organize symbols. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `opts` 中，用于组织符号作用域。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。这里的注释记录了设计假设、不变量或使用说明。

### Lines 29-38

```cpp
static cl::opt<bool> AggressiveReAssign(
    "use-aggr-reg-reassign",
    cl::desc("use register liveness analysis to try to find more opportunities "
             "for -reg-reassign optimization"),
    cl::cat(BoltOptCategory));
}

namespace llvm {
namespace bolt {
```

- EN: Works inside namespace scope `llvm`, `bolt` to organize symbols. Declares or implements routines including `cat`. Notable symbols here include `cat`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `cat`。这里较值得关注的符号包括 `cat`, `llvm`, `bolt`。

### Lines 39-50

```cpp
void RegReAssign::swap(BinaryFunction &Function, MCPhysReg A, MCPhysReg B) {
  BinaryContext &BC = Function.getBinaryContext();
  const BitVector &AliasA = BC.MIB->getAliases(A, false);
  const BitVector &AliasB = BC.MIB->getAliases(B, false);

  // Regular instructions
  for (BinaryBasicBlock &BB : Function) {
    for (MCInst &Inst : BB) {
      for (MCOperand &Operand : MCPlus::primeOperands(Inst)) {
        if (!Operand.isReg())
          continue;
```

- EN: Declares or implements routines including `swap`, `getAliases`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `swap`, `getAliases`.
- CN: 这里声明或实现函数，例如 `swap`, `getAliases`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `swap`, `getAliases`。

### Lines 51-66

```cpp
        unsigned Reg = Operand.getReg();
        if (AliasA.test(Reg)) {
          Operand.setReg(BC.MIB->getAliasSized(B, BC.MIB->getRegSize(Reg)));
          --StaticBytesSaved;
          DynBytesSaved -= BB.getKnownExecutionCount();
          continue;
        }
        if (!AliasB.test(Reg))
          continue;
        Operand.setReg(BC.MIB->getAliasSized(A, BC.MIB->getRegSize(Reg)));
        ++StaticBytesSaved;
        DynBytesSaved += BB.getKnownExecutionCount();
      }
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 67-77

```cpp
  // CFI
  DenseSet<const MCCFIInstruction *> Changed;
  for (BinaryBasicBlock &BB : Function) {
    for (MCInst &Inst : BB) {
      if (!BC.MIB->isCFI(Inst))
        continue;
      const MCCFIInstruction *CFI = Function.getCFIFor(Inst);
      if (Changed.count(CFI))
        continue;
      Changed.insert(CFI);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 78-95

```cpp
      switch (CFI->getOperation()) {
      case MCCFIInstruction::OpRegister: {
        const unsigned CFIReg2 = CFI->getRegister2();
        const MCPhysReg Reg2 = *BC.MRI->getLLVMRegNum(CFIReg2, /*isEH=*/false);
        if (AliasA.test(Reg2)) {
          Function.setCFIFor(
              Inst, MCCFIInstruction::createRegister(
                        nullptr, CFI->getRegister(),
                        BC.MRI->getDwarfRegNum(
                            BC.MIB->getAliasSized(B, BC.MIB->getRegSize(Reg2)),
                            false)));
        } else if (AliasB.test(Reg2)) {
          Function.setCFIFor(
              Inst, MCCFIInstruction::createRegister(
                        nullptr, CFI->getRegister(),
                        BC.MRI->getDwarfRegNum(
                            BC.MIB->getAliasSized(A, BC.MIB->getRegSize(Reg2)),
                            false)));
```

- EN: Declares or implements routines including `getRegister2`, `getLLVMRegNum`, `getRegister`, `getAliasSized`, `if`. Notable symbols here include `getRegister2`, `getLLVMRegNum`, `getRegister`, `getAliasSized`, `if`.
- CN: 这里声明或实现函数，例如 `getRegister2`, `getLLVMRegNum`, `getRegister`, `getAliasSized`, `if`。这里较值得关注的符号包括 `getRegister2`, `getLLVMRegNum`, `getRegister`, `getAliasSized`, `if`。

### Lines 96-113

```cpp
        }
      }
      [[fallthrough]];
      case MCCFIInstruction::OpUndefined:
      case MCCFIInstruction::OpDefCfa:
      case MCCFIInstruction::OpOffset:
      case MCCFIInstruction::OpRestore:
      case MCCFIInstruction::OpSameValue:
      case MCCFIInstruction::OpDefCfaRegister:
      case MCCFIInstruction::OpRelOffset:
      case MCCFIInstruction::OpEscape: {
        unsigned CFIReg;
        if (CFI->getOperation() != MCCFIInstruction::OpEscape) {
          CFIReg = CFI->getRegister();
        } else {
          std::optional<uint8_t> Reg =
              readDWARFExpressionTargetReg(CFI->getValues());
          // Handle DW_CFA_def_cfa_expression
```

- EN: Declares or implements routines including `getRegister`, `readDWARFExpressionTargetReg`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getRegister`, `readDWARFExpressionTargetReg`.
- CN: 这里声明或实现函数，例如 `getRegister`, `readDWARFExpressionTargetReg`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getRegister`, `readDWARFExpressionTargetReg`。

### Lines 114-131

```cpp
          if (!Reg)
            break;
          CFIReg = *Reg;
        }
        const MCPhysReg Reg = *BC.MRI->getLLVMRegNum(CFIReg, /*isEH=*/false);
        if (AliasA.test(Reg))
          Function.mutateCFIRegisterFor(
              Inst,
              BC.MRI->getDwarfRegNum(
                  BC.MIB->getAliasSized(B, BC.MIB->getRegSize(Reg)), false));
        else if (AliasB.test(Reg))
          Function.mutateCFIRegisterFor(
              Inst,
              BC.MRI->getDwarfRegNum(
                  BC.MIB->getAliasSized(A, BC.MIB->getRegSize(Reg)), false));
        break;
      }
      default:
```

- EN: Declares or implements routines including `getLLVMRegNum`, `getAliasSized`. Notable symbols here include `getLLVMRegNum`, `getAliasSized`.
- CN: 这里声明或实现函数，例如 `getLLVMRegNum`, `getAliasSized`。这里较值得关注的符号包括 `getLLVMRegNum`, `getAliasSized`。

### Lines 132-142

```cpp
        break;
      }
    }
  }
}

void RegReAssign::rankRegisters(BinaryFunction &Function) {
  BinaryContext &BC = Function.getBinaryContext();
  std::fill(RegScore.begin(), RegScore.end(), 0);
  std::fill(RankedRegs.begin(), RankedRegs.end(), 0);
```

- EN: Declares or implements routines including `rankRegisters`, `fill`. Notable symbols here include `rankRegisters`, `fill`.
- CN: 这里声明或实现函数，例如 `rankRegisters`, `fill`。这里较值得关注的符号包括 `rankRegisters`, `fill`。

### Lines 143-155

```cpp
  auto countRegScore = [&](BinaryBasicBlock &BB) {
    for (MCInst &Inst : BB) {
      const bool CannotUseREX = BC.MIB->cannotUseREX(Inst);
      const MCInstrDesc &Desc = BC.MII->get(Inst.getOpcode());

      // Disallow substitutions involving regs in implicit uses lists
      for (MCPhysReg ImplicitUse : Desc.implicit_uses()) {
        const size_t RegEC =
            BC.MIB->getAliases(ImplicitUse, false).find_first();
        RegScore[RegEC] =
            std::numeric_limits<decltype(RegScore)::value_type>::min();
      }
```

- EN: Declares or implements routines including `cannotUseREX`, `get`, `getAliases`, `decltype`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `cannotUseREX`, `get`, `getAliases`, `decltype`.
- CN: 这里声明或实现函数，例如 `cannotUseREX`, `get`, `getAliases`, `decltype`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `cannotUseREX`, `get`, `getAliases`, `decltype`。

### Lines 156-163

```cpp
      // Disallow substitutions involving regs in implicit defs lists
      for (MCPhysReg ImplicitDef : Desc.implicit_defs()) {
        const size_t RegEC =
            BC.MIB->getAliases(ImplicitDef, false).find_first();
        RegScore[RegEC] =
            std::numeric_limits<decltype(RegScore)::value_type>::min();
      }
```

- EN: Declares or implements routines including `getAliases`, `decltype`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getAliases`, `decltype`.
- CN: 这里声明或实现函数，例如 `getAliases`, `decltype`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getAliases`, `decltype`。

### Lines 164-171

```cpp
      for (int I = 0, E = MCPlus::getNumPrimeOperands(Inst); I != E; ++I) {
        const MCOperand &Operand = Inst.getOperand(I);
        if (!Operand.isReg())
          continue;

        if (Desc.getOperandConstraint(I, MCOI::TIED_TO) != -1)
          continue;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 172-189

```cpp
        unsigned Reg = Operand.getReg();
        size_t RegEC = BC.MIB->getAliases(Reg, false).find_first();
        if (RegEC == 0)
          continue;

        // Disallow substitutions involving regs in instrs that cannot use REX
        // The relationship of X86 registers is shown in the diagram. BL and BH
        // do not have a direct alias relationship. However, if the BH register
        // cannot be swapped, then the BX/EBX/RBX registers cannot be swapped as
        // well, which means that BL register also cannot be swapped. Therefore,
        // in the presence of BX/EBX/RBX registers, BL and BH have an alias
        // relationship.
        // ┌─────────────────┐
        // │  RBX            │
        // ├─────┬───────────┤
        // │     │  EBX      │
        // ├─────┴──┬────────┤
        // │        │   BX   │
```

- EN: Declares or implements routines including `getAliases`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getAliases`.
- CN: 这里声明或实现函数，例如 `getAliases`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getAliases`。

### Lines 190-199

```cpp
        // ├────────┼───┬────┤
        // │        │BH │BL  │
        // └────────┴───┴────┘
        if (CannotUseREX) {
          RegScore[RegEC] =
              std::numeric_limits<decltype(RegScore)::value_type>::min();
          RegScore[BC.MIB->getAliasSized(Reg, 1)] = RegScore[RegEC];
          continue;
        }
```

- EN: Declares or implements routines including `decltype`, `getAliasSized`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `decltype`, `getAliasSized`.
- CN: 这里声明或实现函数，例如 `decltype`, `getAliasSized`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `decltype`, `getAliasSized`。

### Lines 200-207

```cpp
        // Unsupported substitution, cannot swap BH with R* regs, bail
        if (BC.MIB->isUpper8BitReg(Reg) && ClassicCSR.test(Reg)) {
          RegScore[RegEC] =
              std::numeric_limits<decltype(RegScore)::value_type>::min();
          RegScore[BC.MIB->getAliasSized(Reg, 1)] = RegScore[RegEC];
          continue;
        }
```

- EN: Declares or implements routines including `decltype`, `getAliasSized`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `decltype`, `getAliasSized`.
- CN: 这里声明或实现函数，例如 `decltype`, `getAliasSized`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `decltype`, `getAliasSized`。

### Lines 208-219

```cpp
        RegScore[RegEC] += BB.getKnownExecutionCount();
      }
    }
  };
  for (BinaryBasicBlock &BB : Function)
    countRegScore(BB);

  for (BinaryFunction *ChildFrag : Function.getFragments()) {
    for (BinaryBasicBlock &BB : *ChildFrag)
      countRegScore(BB);
  }
```

- EN: Declares or implements routines including `countRegScore`. Notable symbols here include `countRegScore`.
- CN: 这里声明或实现函数，例如 `countRegScore`。这里较值得关注的符号包括 `countRegScore`。

### Lines 220-236

```cpp
  std::iota(RankedRegs.begin(), RankedRegs.end(), 0); // 0, 1, 2, 3...
  llvm::sort(RankedRegs,
             [&](size_t A, size_t B) { return RegScore[A] > RegScore[B]; });

  LLVM_DEBUG({
    for (size_t Reg : RankedRegs) {
      if (RegScore[Reg] == 0)
        continue;
      dbgs() << Reg << " ";
      if (RegScore[Reg] > 0)
        dbgs() << BC.MRI->getName(Reg) << ": " << RegScore[Reg] << "\n";
      else
        dbgs() << BC.MRI->getName(Reg) << ": (blacklisted)\n";
    }
  });
}
```

- EN: Declares or implements routines including `iota`, `dbgs`. Notable symbols here include `iota`, `dbgs`.
- CN: 这里声明或实现函数，例如 `iota`, `dbgs`。这里较值得关注的符号包括 `iota`, `dbgs`。

### Lines 237-251

```cpp
void RegReAssign::aggressivePassOverFunction(BinaryFunction &Function) {
  BinaryContext &BC = Function.getBinaryContext();
  rankRegisters(Function);

  // If there is a situation where function:
  //   A() -> A.cold()
  //   A.localalias() -> A.cold()
  // simply swapping these two calls can cause issues.
  for (BinaryFunction *ChildFrag : Function.getFragments()) {
    if (ChildFrag->getParentFragments()->size() > 1)
      return;
    if (ChildFrag->empty())
      return;
  }
```

- EN: Declares or implements routines including `aggressivePassOverFunction`, `rankRegisters`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `aggressivePassOverFunction`, `rankRegisters`.
- CN: 这里声明或实现函数，例如 `aggressivePassOverFunction`, `rankRegisters`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `aggressivePassOverFunction`, `rankRegisters`。

### Lines 252-269

```cpp
  // Bail early if our registers are all black listed, before running expensive
  // analysis passes
  bool Bail = true;
  int64_t LowScoreClassic = std::numeric_limits<int64_t>::max();
  for (int J : ClassicRegs.set_bits()) {
    if (RegScore[J] <= 0)
      continue;
    Bail = false;
    if (RegScore[J] < LowScoreClassic)
      LowScoreClassic = RegScore[J];
  }
  if (Bail)
    return;
  BitVector Extended = ClassicRegs;
  Extended.flip();
  Extended &= GPRegs;
  Bail = true;
  int64_t HighScoreExtended = 0;
```

- EN: Declares or implements routines including `max`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `max`.
- CN: 这里声明或实现函数，例如 `max`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `max`。

### Lines 270-281

```cpp
  for (int J : Extended.set_bits()) {
    if (RegScore[J] <= 0)
      continue;
    Bail = false;
    if (RegScore[J] > HighScoreExtended)
      HighScoreExtended = RegScore[J];
  }
  // Also bail early if there is no profitable substitution even if we assume
  // all registers can be exchanged
  if (Bail || (LowScoreClassic << 1) >= HighScoreExtended)
    return;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 282-290

```cpp
  // -- expensive pass -- determine all regs alive during func start
  DataflowInfoManager Info(Function, RA.get(), nullptr);
  BitVector AliveAtStart = *Info.getLivenessAnalysis().getStateAt(
      ProgramPoint::getFirstPointAt(*Function.begin()));
  for (BinaryBasicBlock &BB : Function)
    if (BB.pred_size() == 0)
      AliveAtStart |= *Info.getLivenessAnalysis().getStateAt(
          ProgramPoint::getFirstPointAt(BB));
```

- EN: Declares or implements routines including `Info`, `getFirstPointAt`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `Info`, `getFirstPointAt`.
- CN: 这里声明或实现函数，例如 `Info`, `getFirstPointAt`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `Info`, `getFirstPointAt`。

### Lines 291-305

```cpp
  // Mark frame pointer alive because of CFI
  AliveAtStart |= BC.MIB->getAliases(BC.MIB->getFramePointer(), false);
  // Never touch return registers
  BC.MIB->getDefaultLiveOut(AliveAtStart);

  // Try swapping more profitable options first
  auto Begin = RankedRegs.begin();
  auto End = std::prev(RankedRegs.end());
  while (Begin != End) {
    MCPhysReg ClassicReg = *End;
    if (!ClassicRegs[ClassicReg] || RegScore[ClassicReg] <= 0) {
      --End;
      continue;
    }
```

- EN: Declares or implements routines including `getAliases`, `getDefaultLiveOut`, `prev`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getAliases`, `getDefaultLiveOut`, `prev`.
- CN: 这里声明或实现函数，例如 `getAliases`, `getDefaultLiveOut`, `prev`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getAliases`, `getDefaultLiveOut`, `prev`。

### Lines 306-318

```cpp
    MCPhysReg ExtReg = *Begin;
    if (!Extended[ExtReg] || RegScore[ExtReg] <= 0) {
      ++Begin;
      continue;
    }

    if (RegScore[ClassicReg] << 1 >= RegScore[ExtReg]) {
      LLVM_DEBUG(dbgs() << " Ending at " << BC.MRI->getName(ClassicReg)
                        << " with " << BC.MRI->getName(ExtReg)
                        << " because exchange is not profitable\n");
      break;
    }
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `getName`. Notable symbols here include `LLVM_DEBUG`, `getName`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `getName`。这里较值得关注的符号包括 `LLVM_DEBUG`, `getName`。

### Lines 319-333

```cpp
    if (AliveAtStart.anyCommon(BC.MIB->getAliases(ClassicReg))) {
      LLVM_DEBUG(dbgs() << " Bailed on " << BC.MRI->getName(ClassicReg)
                        << " with " << BC.MRI->getName(ExtReg)
                        << " because classic reg is alive\n");
      --End;
      continue;
    }
    if (AliveAtStart.anyCommon(BC.MIB->getAliases(ExtReg))) {
      LLVM_DEBUG(dbgs() << " Bailed on " << BC.MRI->getName(ClassicReg)
                        << " with " << BC.MRI->getName(ExtReg)
                        << " because extended reg is alive\n");
      ++Begin;
      continue;
    }
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `getName`. Notable symbols here include `LLVM_DEBUG`, `getName`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `getName`。这里较值得关注的符号包括 `LLVM_DEBUG`, `getName`。

### Lines 334-349

```cpp
    // Opportunity detected. Swap.
    LLVM_DEBUG(dbgs() << "\n ** Swapping " << BC.MRI->getName(ClassicReg)
                      << " with " << BC.MRI->getName(ExtReg) << "\n\n");
    swap(Function, ClassicReg, ExtReg);
    FuncsChanged.insert(&Function);
    for (BinaryFunction *ChildFrag : Function.getFragments()) {
      swap(*ChildFrag, ClassicReg, ExtReg);
      FuncsChanged.insert(ChildFrag);
    }
    ++Begin;
    if (Begin == End)
      break;
    --End;
  }
}
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `getName`, `swap`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`, `getName`, `swap`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `getName`, `swap`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`, `getName`, `swap`。

### Lines 350-360

```cpp
bool RegReAssign::conservativePassOverFunction(BinaryFunction &Function) {
  BinaryContext &BC = Function.getBinaryContext();
  rankRegisters(Function);

  for (BinaryFunction *ChildFrag : Function.getFragments()) {
    if (ChildFrag->getParentFragments()->size() > 1)
      return false;
    if (ChildFrag->empty())
      return false;
  }
```

- EN: Declares or implements routines including `conservativePassOverFunction`, `rankRegisters`. Notable symbols here include `conservativePassOverFunction`, `rankRegisters`.
- CN: 这里声明或实现函数，例如 `conservativePassOverFunction`, `rankRegisters`。这里较值得关注的符号包括 `conservativePassOverFunction`, `rankRegisters`。

### Lines 361-370

```cpp
  // Try swapping R12, R13, R14 or R15 with RBX (we work with all callee-saved
  // regs except RBP)
  MCPhysReg Candidate = 0;
  for (int J : ExtendedCSR.set_bits())
    if (RegScore[J] > RegScore[Candidate])
      Candidate = J;

  if (!Candidate || RegScore[Candidate] < 0)
    return false;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 371-378

```cpp
  // Check if our classic callee-saved reg (RBX is the only one) has lower
  // score / utilization rate
  MCPhysReg RBX = 0;
  for (int I : ClassicCSR.set_bits()) {
    int64_t ScoreRBX = RegScore[I];
    if (ScoreRBX <= 0)
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 379-394

```cpp
    if (RegScore[Candidate] > (ScoreRBX + 10))
      RBX = I;
  }

  if (!RBX)
    return false;

  // The high 8 bits of the register will never be swapped. To prevent the high
  // 8 bits from being swapped incorrectly, we should switched to swapping the
  // low 8 bits of the register instead.
  if (BC.MIB->isUpper8BitReg(RBX)) {
    RBX = BC.MIB->getAliasSized(RBX, 1);
    if (RegScore[RBX] < 0 || RegScore[RBX] > RegScore[Candidate])
      return false;
  }
```

- EN: Declares or implements routines including `getAliasSized`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getAliasSized`.
- CN: 这里声明或实现函数，例如 `getAliasSized`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getAliasSized`。

### Lines 395-406

```cpp
  LLVM_DEBUG(dbgs() << "\n ** Swapping " << BC.MRI->getName(RBX) << " with "
                    << BC.MRI->getName(Candidate) << "\n\n");
  (void)BC;
  swap(Function, RBX, Candidate);
  FuncsChanged.insert(&Function);
  for (BinaryFunction *ChildFrag : Function.getFragments()) {
    swap(*ChildFrag, RBX, Candidate);
    FuncsChanged.insert(ChildFrag);
  }
  return true;
}
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `getName`, `swap`. Notable symbols here include `LLVM_DEBUG`, `getName`, `swap`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `getName`, `swap`。这里较值得关注的符号包括 `LLVM_DEBUG`, `getName`, `swap`。

### Lines 407-416

```cpp
void RegReAssign::setupAggressivePass(BinaryContext &BC,
                                      std::map<uint64_t, BinaryFunction> &BFs) {
  setupConservativePass(BC, BFs);
  CG.reset(new BinaryFunctionCallGraph(buildCallGraph(BC)));
  RA.reset(new RegAnalysis(BC, &BFs, &*CG));

  GPRegs = BitVector(BC.MRI->getNumRegs(), false);
  BC.MIB->getGPRegs(GPRegs);
}
```

- EN: Declares or implements routines including `setupConservativePass`, `BitVector`, `getGPRegs`. Notable symbols here include `setupConservativePass`, `BitVector`, `getGPRegs`.
- CN: 这里声明或实现函数，例如 `setupConservativePass`, `BitVector`, `getGPRegs`。这里较值得关注的符号包括 `setupConservativePass`, `BitVector`, `getGPRegs`。

### Lines 417-434

```cpp
void RegReAssign::setupConservativePass(
    BinaryContext &BC, std::map<uint64_t, BinaryFunction> &BFs) {
  // Set up constant bitvectors used throughout this analysis
  ClassicRegs = BitVector(BC.MRI->getNumRegs(), false);
  CalleeSaved = BitVector(BC.MRI->getNumRegs(), false);
  ClassicCSR = BitVector(BC.MRI->getNumRegs(), false);
  ExtendedCSR = BitVector(BC.MRI->getNumRegs(), false);
  // Never consider the frame pointer
  BC.MIB->getClassicGPRegs(ClassicRegs);
  ClassicRegs.flip();
  ClassicRegs |= BC.MIB->getAliases(BC.MIB->getFramePointer(), false);
  ClassicRegs.flip();
  BC.MIB->getCalleeSavedRegs(CalleeSaved);
  ClassicCSR |= ClassicRegs;
  ClassicCSR &= CalleeSaved;
  BC.MIB->getClassicGPRegs(ClassicRegs);
  ExtendedCSR |= ClassicRegs;
  ExtendedCSR.flip();
```

- EN: Declares or implements routines including `BitVector`, `getClassicGPRegs`, `getAliases`, `getCalleeSavedRegs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `BitVector`, `getClassicGPRegs`, `getAliases`, `getCalleeSavedRegs`.
- CN: 这里声明或实现函数，例如 `BitVector`, `getClassicGPRegs`, `getAliases`, `getCalleeSavedRegs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `BitVector`, `getClassicGPRegs`, `getAliases`, `getCalleeSavedRegs`。

### Lines 435-450

```cpp
  ExtendedCSR &= CalleeSaved;

  LLVM_DEBUG({
    RegStatePrinter P(BC);
    dbgs() << "Starting register reassignment\nClassicRegs: ";
    P.print(dbgs(), ClassicRegs);
    dbgs() << "\nCalleeSaved: ";
    P.print(dbgs(), CalleeSaved);
    dbgs() << "\nClassicCSR: ";
    P.print(dbgs(), ClassicCSR);
    dbgs() << "\nExtendedCSR: ";
    P.print(dbgs(), ExtendedCSR);
    dbgs() << "\n";
  });
}
```

- EN: Declares or implements routines including `P`, `dbgs`. Notable symbols here include `P`, `dbgs`.
- CN: 这里声明或实现函数，例如 `P`, `dbgs`。这里较值得关注的符号包括 `P`, `dbgs`。

### Lines 451-459

```cpp
Error RegReAssign::runOnFunctions(BinaryContext &BC) {
  if (!BC.isX86()) {
    BC.errs() << "BOLT-ERROR: reg-reassign is specific to X86\n";
    exit(1);
  }

  RegScore = std::vector<int64_t>(BC.MRI->getNumRegs(), 0);
  RankedRegs = std::vector<size_t>(BC.MRI->getNumRegs(), 0);
```

- EN: Declares or implements routines including `runOnFunctions`, `exit`. Notable symbols here include `runOnFunctions`, `exit`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`, `exit`。这里较值得关注的符号包括 `runOnFunctions`, `exit`。

### Lines 460-467

```cpp
  if (opts::AggressiveReAssign)
    setupAggressivePass(BC, BC.getBinaryFunctions());
  else
    setupConservativePass(BC, BC.getBinaryFunctions());

  for (auto &I : BC.getBinaryFunctions()) {
    BinaryFunction &Function = I.second;
```

- EN: Declares or implements routines including `setupAggressivePass`, `setupConservativePass`. Notable symbols here include `setupAggressivePass`, `setupConservativePass`.
- CN: 这里声明或实现函数，例如 `setupAggressivePass`, `setupConservativePass`。这里较值得关注的符号包括 `setupAggressivePass`, `setupConservativePass`。

### Lines 468-482

```cpp
    if (!Function.isSimple() || Function.isIgnored() || Function.isFragment())
      continue;

    LLVM_DEBUG(dbgs() << "====================================\n");
    LLVM_DEBUG(dbgs() << " - " << Function.getPrintName() << "\n");
    if (!conservativePassOverFunction(Function) && opts::AggressiveReAssign) {
      aggressivePassOverFunction(Function);
      LLVM_DEBUG({
        if (FuncsChanged.count(&Function))
          dbgs() << "Aggressive pass successful on " << Function.getPrintName()
                 << "\n";
      });
    }
  }
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `aggressivePassOverFunction`, `dbgs`. Notable symbols here include `LLVM_DEBUG`, `aggressivePassOverFunction`, `dbgs`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `aggressivePassOverFunction`, `dbgs`。这里较值得关注的符号包括 `LLVM_DEBUG`, `aggressivePassOverFunction`, `dbgs`。

### Lines 483-499

```cpp
  if (FuncsChanged.empty()) {
    BC.outs() << "BOLT-INFO: Reg Reassignment Pass: no changes were made.\n";
    return Error::success();
  }
  if (opts::UpdateDebugSections)
    BC.outs()
        << "BOLT-WARNING: You used -reg-reassign and -update-debug-sections."
        << " Some registers were changed but associated AT_LOCATION for "
        << "impacted variables were NOT updated! This operation is "
        << "currently unsupported by BOLT.\n";
  BC.outs() << "BOLT-INFO: Reg Reassignment Pass Stats:\n";
  BC.outs() << "\t   " << FuncsChanged.size() << " functions affected.\n";
  BC.outs() << "\t   " << StaticBytesSaved << " static bytes saved.\n";
  BC.outs() << "\t   " << DynBytesSaved << " dynamic bytes saved.\n";
  return Error::success();
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 500-501

```cpp
} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `cat`: function or method entry point / 函数或方法入口
- `swap`: function or method entry point / 函数或方法入口
- `getAliases`: function or method entry point / 函数或方法入口
- `getRegister2`: function or method entry point / 函数或方法入口
- `getLLVMRegNum`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域
- `opts`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/RegReAssign.h`, `bolt/Core/BinaryFunctionCallGraph.h`, `bolt/Core/MCPlus.h`, `bolt/Passes/DataflowAnalysis.h`, `bolt/Passes/DataflowInfoManager.h`, `bolt/Utils/Utils.h`
- System headers / 系统头文件: `numeric`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
