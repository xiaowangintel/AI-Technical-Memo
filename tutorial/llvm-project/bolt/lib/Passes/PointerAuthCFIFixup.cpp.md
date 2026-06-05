# PointerAuthCFIFixup.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/PointerAuthCFIFixup.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Passes/PointerAuthCFIFixup.cpp This file implements the PointerAuthCFIFixup class. It inserts OpNegateRAState CFIs to places where the state of two consecutive instructions are different.. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：bolt/Passes/PointerAuthCFIFixup.cpp This file implements the PointerAuthCFIFixup class. It inserts OpNegateRAState CFIs to places where the state of two consecutive instructions are different.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```cpp
//===- bolt/Passes/PointerAuthCFIFixup.cpp --------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the PointerAuthCFIFixup class. It inserts
// OpNegateRAState CFIs to places where the state of two consecutive
// instructions are different.
//
//===----------------------------------------------------------------------===//
#include "bolt/Passes/PointerAuthCFIFixup.h"
#include "bolt/Core/BinaryFunction.h"
#include "bolt/Core/ParallelUtilities.h"
#include <cstdlib>
```

- EN: Pulls in 4 header(s) from local project, system dependencies needed by this range. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 4 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 19-29

```cpp
using namespace llvm;

namespace llvm {
namespace bolt {

static bool PassFailed = false;

void PointerAuthCFIFixup::runOnFunction(BinaryFunction &BF) {
  if (PassFailed)
    return;
```

- EN: Works inside namespace scope `llvm`, `bolt` to organize symbols. Declares or implements routines including `runOnFunction`. Notable symbols here include `runOnFunction`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `runOnFunction`。这里较值得关注的符号包括 `runOnFunction`, `llvm`, `bolt`。

### Lines 30-41

```cpp
  BinaryContext &BC = BF.getBinaryContext();

  if (BF.getState() == BinaryFunction::State::Empty)
    return;

  if (BF.getState() != BinaryFunction::State::CFG &&
      BF.getState() != BinaryFunction::State::CFG_Finalized) {
    BC.outs() << "BOLT-INFO: no CFG for " << BF.getPrintName()
              << " in PointerAuthCFIFixup\n";
    return;
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 42-59

```cpp
  inferUnknownStates(BF);

  for (FunctionFragment &FF : BF.getLayout().fragments()) {
    coverFunctionFragmentStart(BF, FF);
    bool FirstIter = true;
    bool PrevRAState = false;
    // As this pass runs after function splitting, we should only check
    // consecutive instructions inside FunctionFragments.
    for (BinaryBasicBlock *BB : FF) {
      for (auto It = BB->begin(); It != BB->end(); ++It) {
        MCInst &Inst = *It;
        if (BC.MIB->isCFI(Inst))
          continue;
        std::optional<bool> RAState = BC.MIB->getRAState(Inst);
        if (!RAState.has_value()) {
          BC.errs() << "BOLT-ERROR: unknown RAState after inferUnknownStates "
                    << " in function " << BF.getPrintName() << "\n";
          PassFailed = true;
```

- EN: Declares or implements routines including `inferUnknownStates`, `coverFunctionFragmentStart`, `getRAState`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `inferUnknownStates`, `coverFunctionFragmentStart`, `getRAState`.
- CN: 这里声明或实现函数，例如 `inferUnknownStates`, `coverFunctionFragmentStart`, `getRAState`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `inferUnknownStates`, `coverFunctionFragmentStart`, `getRAState`。

### Lines 60-76

```cpp
          return;
        }
        if (!FirstIter) {
          // Consecutive instructions with different RAState means we need to
          // add a OpNegateRAState.
          if (*RAState != PrevRAState)
            It = BF.addCFIInstruction(
                BB, It, MCCFIInstruction::createNegateRAState(nullptr));
        } else {
          FirstIter = false;
        }
        PrevRAState = *RAState;
      }
    }
  }
}
```

- EN: Declares or implements routines including `createNegateRAState`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `createNegateRAState`.
- CN: 这里声明或实现函数，例如 `createNegateRAState`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `createNegateRAState`。

### Lines 77-90

```cpp
void PointerAuthCFIFixup::inferUnknownStates(BinaryFunction &BF) {
  BinaryContext &BC = BF.getBinaryContext();

  // Fill in missing RAStates in simple cases (inside BBs).
  for (BinaryBasicBlock &BB : BF) {
    fillUnknownStateInBB(BC, BB);
  }
  // BasicBlocks which are made entirely of "new instructions" (instructions
  // without RAState annotation) are stubs, and do not have correct unwind info.
  // We should iterate in layout order and fill them based on previous known
  // RAState.
  fillUnknownStubs(BF);
}
```

- EN: Declares or implements routines including `inferUnknownStates`, `fillUnknownStateInBB`, `fillUnknownStubs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `inferUnknownStates`, `fillUnknownStateInBB`, `fillUnknownStubs`.
- CN: 这里声明或实现函数，例如 `inferUnknownStates`, `fillUnknownStateInBB`, `fillUnknownStubs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `inferUnknownStates`, `fillUnknownStateInBB`, `fillUnknownStubs`。

### Lines 91-108

```cpp
void PointerAuthCFIFixup::coverFunctionFragmentStart(BinaryFunction &BF,
                                                     FunctionFragment &FF) {
  BinaryContext &BC = BF.getBinaryContext();
  if (FF.empty())
    return;
  // Find the first BB in the FF which has Instructions.
  // BOLT can generate empty BBs at function splitting which are only used as
  // target labels. We should add the negate-ra-state CFI to the first
  // non-empty BB.
  auto *FirstNonEmpty =
      std::find_if(FF.begin(), FF.end(), [](BinaryBasicBlock *BB) {
        // getFirstNonPseudo returns BB.end() if it does not find any
        // Instructions.
        return BB->getFirstNonPseudo() != BB->end();
      });
  // If a function is already split in the input, the first FF can also start
  // with Signed state. This covers that scenario as well.
  auto II = (*FirstNonEmpty)->getFirstNonPseudo();
```

- EN: Declares or implements routines including `find_if`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `find_if`.
- CN: 这里声明或实现函数，例如 `find_if`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `find_if`。

### Lines 109-120

```cpp
  std::optional<bool> RAState = BC.MIB->getRAState(*II);
  if (!RAState.has_value()) {
    BC.errs() << "BOLT-ERROR: unknown RAState after inferUnknownStates "
              << " in function " << BF.getPrintName() << "\n";
    PassFailed = true;
    return;
  }
  if (*RAState)
    BF.addCFIInstruction(*FirstNonEmpty, II,
                         MCCFIInstruction::createNegateRAState(nullptr));
}
```

- EN: Declares or implements routines including `getRAState`, `createNegateRAState`. Notable symbols here include `getRAState`, `createNegateRAState`.
- CN: 这里声明或实现函数，例如 `getRAState`, `createNegateRAState`。这里较值得关注的符号包括 `getRAState`, `createNegateRAState`。

### Lines 121-133

```cpp
std::optional<bool>
PointerAuthCFIFixup::getFirstKnownRAState(BinaryContext &BC,
                                          BinaryBasicBlock &BB) {
  for (const MCInst &Inst : BB) {
    if (BC.MIB->isCFI(Inst))
      continue;
    std::optional<bool> RAState = BC.MIB->getRAState(Inst);
    if (RAState.has_value())
      return RAState;
  }
  return std::nullopt;
}
```

- EN: Declares or implements routines including `getRAState`. Notable symbols here include `getRAState`.
- CN: 这里声明或实现函数，例如 `getRAState`。这里较值得关注的符号包括 `getRAState`。

### Lines 134-142

```cpp
bool PointerAuthCFIFixup::isUnknownBlock(BinaryContext &BC,
                                         BinaryBasicBlock &BB) {
  std::optional<bool> FirstRAState = getFirstKnownRAState(BC, BB);
  return !FirstRAState.has_value();
}

void PointerAuthCFIFixup::fillUnknownStateInBB(BinaryContext &BC,
                                               BinaryBasicBlock &BB) {
```

- EN: Declares or implements routines including `getFirstKnownRAState`. Notable symbols here include `getFirstKnownRAState`.
- CN: 这里声明或实现函数，例如 `getFirstKnownRAState`。这里较值得关注的符号包括 `getFirstKnownRAState`。

### Lines 143-154

```cpp
  auto First = BB.getFirstNonPseudo();
  if (First == BB.end())
    return;
  // If the first instruction has unknown RAState, we should copy the first
  // known RAState.
  std::optional<bool> RAState = BC.MIB->getRAState(*First);
  if (!RAState.has_value()) {
    std::optional<bool> FirstRAState = getFirstKnownRAState(BC, BB);
    if (!FirstRAState.has_value())
      // We fill unknown BBs later.
      return;
```

- EN: Declares or implements routines including `getRAState`, `getFirstKnownRAState`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getRAState`, `getFirstKnownRAState`.
- CN: 这里声明或实现函数，例如 `getRAState`, `getFirstKnownRAState`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getRAState`, `getFirstKnownRAState`。

### Lines 155-165

```cpp
    BC.MIB->setRAState(*First, *FirstRAState);
  }

  // At this point we know the RAState of the first instruction,
  // so we can propagate the RAStates to all subsequent unknown instructions.
  MCInst Prev = *First;
  for (auto It = First + 1; It != BB.end(); ++It) {
    MCInst &Inst = *It;
    if (BC.MIB->isCFI(Inst))
      continue;
```

- EN: Declares or implements routines including `setRAState`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `setRAState`.
- CN: 这里声明或实现函数，例如 `setRAState`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `setRAState`。

### Lines 166-182

```cpp
    // No need to check for nullopt: we only entered this loop after the first
    // instruction had its RAState set, and RAState is always set for the
    // previous instruction in the previous iteration of the loop.
    std::optional<bool> PrevRAState = BC.MIB->getRAState(Prev);

    std::optional<bool> RAState = BC.MIB->getRAState(Inst);
    if (!RAState.has_value()) {
      if (BC.MIB->isPSignOnLR(Prev))
        PrevRAState = true;
      else if (BC.MIB->isPAuthOnLR(Prev))
        PrevRAState = false;
      BC.MIB->setRAState(Inst, *PrevRAState);
    }
    Prev = Inst;
  }
}
```

- EN: Declares or implements routines including `getRAState`, `setRAState`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getRAState`, `setRAState`.
- CN: 这里声明或实现函数，例如 `getRAState`, `setRAState`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getRAState`, `setRAState`。

### Lines 183-195

```cpp
void PointerAuthCFIFixup::markUnknownBlock(BinaryContext &BC,
                                           BinaryBasicBlock &BB, bool State) {
  // If we call this when an Instruction has either kRASigned or kRAUnsigned
  // annotation, setRAState would fail.
  assert(isUnknownBlock(BC, BB) &&
         "markUnknownBlock should only be called on unknown blocks");
  for (MCInst &Inst : BB) {
    if (BC.MIB->isCFI(Inst))
      continue;
    BC.MIB->setRAState(Inst, State);
  }
}
```

- EN: Declares or implements routines including `assert`, `setRAState`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`, `setRAState`.
- CN: 这里声明或实现函数，例如 `assert`, `setRAState`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`, `setRAState`。

### Lines 196-213

```cpp
void PointerAuthCFIFixup::fillUnknownStubs(BinaryFunction &BF) {
  BinaryContext &BC = BF.getBinaryContext();
  bool FirstIter = true;
  MCInst PrevInst;
  for (FunctionFragment &FF : BF.getLayout().fragments()) {
    for (BinaryBasicBlock *BB : FF) {
      if (FirstIter) {
        FirstIter = false;
        if (isUnknownBlock(BC, *BB))
          // If the first BasicBlock is unknown, the function's entry RAState
          // should be used.
          markUnknownBlock(BC, *BB, BF.getInitialRAState());
      } else if (isUnknownBlock(BC, *BB)) {
        // As explained in issue #160989, the unwind info is incorrect for
        // stubs. Indicating the correct RAState without the rest of the unwind
        // info being correct is not useful. Instead, we copy the RAState from
        // the previous instruction.
        std::optional<bool> PrevRAState = BC.MIB->getRAState(PrevInst);
```

- EN: Declares or implements routines including `fillUnknownStubs`, `markUnknownBlock`, `if`, `getRAState`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `fillUnknownStubs`, `markUnknownBlock`, `if`, `getRAState`.
- CN: 这里声明或实现函数，例如 `fillUnknownStubs`, `markUnknownBlock`, `if`, `getRAState`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `fillUnknownStubs`, `markUnknownBlock`, `if`, `getRAState`。

### Lines 214-231

```cpp
        if (!PrevRAState.has_value()) {
          // No non-cfi instruction encountered in the function yet.
          // This means the RAState is the same as at the function entry.
          markUnknownBlock(BC, *BB, BF.getInitialRAState());
          continue;
        }

        if (BC.MIB->isPSignOnLR(PrevInst))
          PrevRAState = true;
        else if (BC.MIB->isPAuthOnLR(PrevInst))
          PrevRAState = false;
        markUnknownBlock(BC, *BB, *PrevRAState);
      }
      // This function iterates on BasicBlocks, so the PrevInst has to be
      // updated to the last instruction of the current BasicBlock. If the
      // BasicBlock is empty, or only has PseudoInstructions, PrevInst will not
      // be updated.
      auto Last = BB->getLastNonPseudo();
```

- EN: Declares or implements routines including `markUnknownBlock`, `getLastNonPseudo`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `markUnknownBlock`, `getLastNonPseudo`.
- CN: 这里声明或实现函数，例如 `markUnknownBlock`, `getLastNonPseudo`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `markUnknownBlock`, `getLastNonPseudo`。

### Lines 232-244

```cpp
      if (Last != BB->rend())
        PrevInst = *Last;
    }
  }
}

Error PointerAuthCFIFixup::runOnFunctions(BinaryContext &BC) {
  std::atomic<uint64_t> FunctionsModified{0};
  ParallelUtilities::WorkFuncTy WorkFun = [&](BinaryFunction &BF) {
    FunctionsModified++;
    runOnFunction(BF);
  };
```

- EN: Declares or implements routines including `runOnFunctions`, `runOnFunction`. Notable symbols here include `runOnFunctions`, `runOnFunction`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`, `runOnFunction`。这里较值得关注的符号包括 `runOnFunctions`, `runOnFunction`。

### Lines 245-252

```cpp
  ParallelUtilities::PredicateTy SkipPredicate = [&](const BinaryFunction &BF) {
    // We can skip functions which did not include negate-ra-state CFIs. This
    // includes code using pac-ret hardening as well, if the binary is
    // compiled with `-fno-exceptions -fno-unwind-tables
    // -fno-asynchronous-unwind-tables`
    return !BF.containedNegateRAState() || BF.isIgnored();
  };
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 253-266

```cpp
  ParallelUtilities::runOnEachFunction(
      BC, ParallelUtilities::SchedulingPolicy::SP_INST_LINEAR, WorkFun,
      SkipPredicate, "PointerAuthCFIFixup");

  BC.outs() << "BOLT-INFO: rewritten pac-ret DWARF info in "
            << FunctionsModified << " out of " << BC.getBinaryFunctions().size()
            << " functions "
            << format("(%.2lf%%).\n", (100.0 * FunctionsModified) /
                                          BC.getBinaryFunctions().size());
  if (PassFailed)
    return createFatalBOLTError("");
  return Error::success();
}
```

- EN: Declares or implements routines including `format`. Notable symbols here include `format`.
- CN: 这里声明或实现函数，例如 `format`。这里较值得关注的符号包括 `format`。

### Lines 267-268

```cpp
} // end namespace bolt
} // end namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `runOnFunction`: function or method entry point / 函数或方法入口
- `inferUnknownStates`: function or method entry point / 函数或方法入口
- `coverFunctionFragmentStart`: function or method entry point / 函数或方法入口
- `getRAState`: function or method entry point / 函数或方法入口
- `createNegateRAState`: function or method entry point / 函数或方法入口
- `llvm`: namespace scope / 命名空间作用域
- `bolt`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/PointerAuthCFIFixup.h`, `bolt/Core/BinaryFunction.h`, `bolt/Core/ParallelUtilities.h`
- System headers / 系统头文件: `cstdlib`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
