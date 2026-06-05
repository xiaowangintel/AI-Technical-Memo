# AArch64RelaxationPass.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/AArch64RelaxationPass.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Passes/AArch64RelaxationPass.cpp This file implements the AArch64RelaxationPass class.. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：bolt/Passes/AArch64RelaxationPass.cpp This file implements the AArch64RelaxationPass class.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Passes/AArch64RelaxationPass.cpp ------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the AArch64RelaxationPass class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-22

```cpp
#include "bolt/Passes/AArch64RelaxationPass.h"
#include "bolt/Core/ParallelUtilities.h"
#include "bolt/Utils/CommandLineOpts.h"
#include <iterator>

using namespace llvm;

namespace opts {
extern cl::OptionCategory BoltCategory;
```

- EN: Pulls in 4 header(s) from local project, system dependencies needed by this range. Works inside namespace scope `llvm`, `opts` to organize symbols. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `llvm`, `opts`.
- CN: 这里引入 4 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里位于命名空间 `llvm`, `opts` 中，用于组织符号作用域。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `llvm`, `opts`。

### Lines 23-31

```cpp
static cl::opt<bool> AArch64PassOpt(
    "aarch64-relaxation",
    cl::desc("Replace ARM non-local ADR/LDR instructions with ADRP"),
    cl::init(true), cl::cat(BoltCategory), cl::ReallyHidden);
} // namespace opts

namespace llvm {
namespace bolt {
```

- EN: Works inside namespace scope `opts`, `llvm`, `bolt` to organize symbols. Declares or implements routines including `desc`, `init`. Notable symbols here include `desc`, `init`, `opts`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `opts`, `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `desc`, `init`。这里较值得关注的符号包括 `desc`, `init`, `opts`, `llvm`, `bolt`。

### Lines 32-41

```cpp
// We don't exit directly from runOnFunction since it would call ThreadPool
// destructor which might result in internal assert if we're not finished
// creating async jobs on the moment of exit. So we're finishing all parallel
// jobs and checking the exit flag after it.
static bool PassFailed = false;

void AArch64RelaxationPass::runOnFunction(BinaryFunction &BF) {
  if (PassFailed)
    return;
```

- EN: Declares or implements routines including `runOnFunction`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `runOnFunction`.
- CN: 这里声明或实现函数，例如 `runOnFunction`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `runOnFunction`。

### Lines 42-51

```cpp
  BinaryContext &BC = BF.getBinaryContext();
  for (BinaryBasicBlock &BB : BF) {
    for (auto It = BB.begin(); It != BB.end(); ++It) {
      MCInst &Inst = *It;
      bool IsADR = BC.MIB->isADR(Inst);

      // TODO: Handle other types of LDR (literal, PC-relative) instructions.
      if (!IsADR && !BC.MIB->isLoadLiteralGPR(Inst))
        continue;
```

- EN: Declares or implements routines including `isADR`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isADR`.
- CN: 这里声明或实现函数，例如 `isADR`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isADR`。

### Lines 52-61

```cpp
      const MCSymbol *Symbol = BC.MIB->getTargetSymbol(Inst, IsADR ? 0 : 1);
      if (!Symbol)
        continue;

      if (BF.hasIslandsInfo()) {
        BinaryFunction::IslandInfo &Islands = BF.getIslandInfo();
        if (Islands.Symbols.count(Symbol) || Islands.ProxySymbols.count(Symbol))
          continue;
      }
```

- EN: Declares or implements routines including `getTargetSymbol`. Notable symbols here include `getTargetSymbol`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`。这里较值得关注的符号包括 `getTargetSymbol`。

### Lines 62-69

```cpp
      // Don't relax ADR/LDR if it points to the same function and is in the
      // main fragment and BF initial size is < 1MB.
      const unsigned OneMB = 0x100000;
      if (BF.getSize() < OneMB) {
        BinaryFunction *TargetBF = BC.getFunctionForSymbol(Symbol);
        if (TargetBF == &BF && !BB.isSplit())
          continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 70-84

```cpp
        // No relaxation needed if ADR/LDR references a basic block in the same
        // fragment.
        if (BinaryBasicBlock *TargetBB = BF.getBasicBlockForLabel(Symbol))
          if (BB.getFragmentNum() == TargetBB->getFragmentNum())
            continue;
      }

      InstructionListType AdrpMaterialization;
      {
        auto L = BC.scopeLock();
        AdrpMaterialization =
            IsADR ? BC.MIB->undoAdrpAddRelaxation(Inst, BC.Ctx.get())
                  : BC.MIB->createAdrpLdr(Inst, BC.Ctx.get());
      }
```

- EN: Declares or implements routines including `undoAdrpAddRelaxation`, `createAdrpLdr`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `undoAdrpAddRelaxation`, `createAdrpLdr`.
- CN: 这里声明或实现函数，例如 `undoAdrpAddRelaxation`, `createAdrpLdr`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `undoAdrpAddRelaxation`, `createAdrpLdr`。

### Lines 85-102

```cpp
      if (It != BB.begin() && BC.MIB->isNoop(*std::prev(It))) {
        It = BB.eraseInstruction(std::prev(It));
      } else if (std::next(It) != BB.end() && BC.MIB->isNoop(*std::next(It))) {
        BB.eraseInstruction(std::next(It));
      } else if (!BF.isSimple()) {
        // If the function is not simple, it may contain a jump table undetected
        // by us. This jump table may use an offset from the branch instruction
        // to land in the desired place. If we add new instructions, we
        // invalidate this offset, so we have to rely on linker-inserted NOP to
        // replace it with ADRP, and abort if it is not present.
        auto L = BC.scopeLock();
        BC.errs() << "BOLT-ERROR: cannot relax " << (IsADR ? "ADR" : "LDR")
                  << " in non-simple function " << BF << '\n';
        PassFailed = true;
        return;
      }
      It = BB.replaceInstruction(It, AdrpMaterialization);
    }
```

- EN: Declares or implements routines including `if`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `if`.
- CN: 这里声明或实现函数，例如 `if`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `if`。

### Lines 103-113

```cpp
  }
}

Error AArch64RelaxationPass::runOnFunctions(BinaryContext &BC) {
  if (!opts::AArch64PassOpt || !BC.HasRelocations)
    return Error::success();

  ParallelUtilities::WorkFuncTy WorkFun = [&](BinaryFunction &BF) {
    runOnFunction(BF);
  };
```

- EN: Declares or implements routines including `runOnFunctions`, `runOnFunction`. Notable symbols here include `runOnFunctions`, `runOnFunction`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`, `runOnFunction`。这里较值得关注的符号包括 `runOnFunctions`, `runOnFunction`。

### Lines 114-122

```cpp
  ParallelUtilities::runOnEachFunction(
      BC, ParallelUtilities::SchedulingPolicy::SP_TRIVIAL, WorkFun, nullptr,
      "AArch64RelaxationPass");

  if (PassFailed)
    return createFatalBOLTError("");
  return Error::success();
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 123-124

```cpp
} // end namespace bolt
} // end namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `desc`: function or method entry point / 函数或方法入口
- `init`: function or method entry point / 函数或方法入口
- `runOnFunction`: function or method entry point / 函数或方法入口
- `isADR`: function or method entry point / 函数或方法入口
- `getTargetSymbol`: function or method entry point / 函数或方法入口
- `llvm`: namespace scope / 命名空间作用域
- `opts`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/AArch64RelaxationPass.h`, `bolt/Core/ParallelUtilities.h`, `bolt/Utils/CommandLineOpts.h`
- System headers / 系统头文件: `iterator`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
