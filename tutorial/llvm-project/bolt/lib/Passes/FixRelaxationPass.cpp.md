# FixRelaxationPass.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/FixRelaxationPass.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: *- C++ -*. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：*- C++ -*。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
//===- bolt/Passes/FixRelaxationPass.cpp ------------------------*- C++ -*-===//
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
#include "bolt/Passes/FixRelaxationPass.h"
#include "bolt/Core/ParallelUtilities.h"

using namespace llvm;

namespace llvm {
namespace bolt {
```

- EN: Pulls in 2 header(s) from local project dependencies needed by this range. Works inside namespace scope `llvm`, `bolt` to organize symbols. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `llvm`, `bolt`.
- CN: 这里引入 2 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `llvm`, `bolt`。

### Lines 17-33

```cpp
// This function finds ADRP+ADD instruction sequences that originally before
// linker relaxations were ADRP+LDR. We've modified LDR/ADD relocation properly
// during relocation reading, so its targeting right symbol. As for ADRP its
// target is wrong before this pass since we won't be able to recognize and
// properly change R_AARCH64_ADR_GOT_PAGE relocation to
// R_AARCH64_ADR_PREL_PG_HI21 during relocation reading. Now we're searching for
// ADRP+ADD sequences, checking that ADRP points to the GOT-table symbol and the
// target of ADD is another symbol. When found change ADRP symbol reference to
// the ADDs one.
void FixRelaxations::runOnFunction(BinaryFunction &BF) {
  BinaryContext &BC = BF.getBinaryContext();
  for (BinaryBasicBlock &BB : BF) {
    for (auto II = BB.begin(); II != BB.end(); ++II) {
      MCInst &Adrp = *II;
      if (BC.MIB->isPseudo(Adrp) || !BC.MIB->isADRP(Adrp))
        continue;
```

- EN: Declares or implements routines including `runOnFunction`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `runOnFunction`.
- CN: 这里声明或实现函数，例如 `runOnFunction`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `runOnFunction`。

### Lines 34-41

```cpp
      const MCSymbol *AdrpSymbol = BC.MIB->getTargetSymbol(Adrp);
      if (!AdrpSymbol || AdrpSymbol->getName() != "__BOLT_got_zero")
        continue;

      auto NextII = std::next(II);
      if (NextII == BB.end())
        continue;
```

- EN: Declares or implements routines including `getTargetSymbol`, `next`. Notable symbols here include `getTargetSymbol`, `next`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`, `next`。这里较值得关注的符号包括 `getTargetSymbol`, `next`。

### Lines 42-49

```cpp
      const MCInst &Add = *NextII;
      if (!BC.MIB->matchAdrpAddPair(Adrp, Add))
        continue;

      const MCSymbol *Symbol = BC.MIB->getTargetSymbol(Add);
      if (!Symbol || AdrpSymbol == Symbol)
        continue;
```

- EN: Declares or implements routines including `getTargetSymbol`. Notable symbols here include `getTargetSymbol`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`。这里较值得关注的符号包括 `getTargetSymbol`。

### Lines 50-57

```cpp
      auto L = BC.scopeLock();
      const int64_t Addend = BC.MIB->getTargetAddend(Add);
      BC.MIB->setOperandToSymbolRef(Adrp, /*OpNum*/ 1, Symbol, Addend,
                                    BC.Ctx.get(), ELF::R_AARCH64_NONE);
    }
  }
}
```

- EN: Declares or implements routines including `getTargetAddend`. Notable symbols here include `getTargetAddend`.
- CN: 这里声明或实现函数，例如 `getTargetAddend`。这里较值得关注的符号包括 `getTargetAddend`。

### Lines 58-65

```cpp
Error FixRelaxations::runOnFunctions(BinaryContext &BC) {
  if (!BC.isAArch64() || !BC.HasRelocations)
    return Error::success();

  ParallelUtilities::WorkFuncTy WorkFun = [&](BinaryFunction &BF) {
    runOnFunction(BF);
  };
```

- EN: Declares or implements routines including `runOnFunctions`, `runOnFunction`. Notable symbols here include `runOnFunctions`, `runOnFunction`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`, `runOnFunction`。这里较值得关注的符号包括 `runOnFunctions`, `runOnFunction`。

### Lines 66-73

```cpp
  ParallelUtilities::runOnEachFunction(
      BC, ParallelUtilities::SchedulingPolicy::SP_INST_LINEAR, WorkFun, nullptr,
      "FixRelaxations");
  return Error::success();
}

} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Contains local control flow that updates state or selects among execution branches. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `runOnFunction`: function or method entry point / 函数或方法入口
- `getTargetSymbol`: function or method entry point / 函数或方法入口
- `next`: function or method entry point / 函数或方法入口
- `getTargetAddend`: function or method entry point / 函数或方法入口
- `runOnFunctions`: function or method entry point / 函数或方法入口
- `llvm`: namespace scope / 命名空间作用域
- `bolt`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/FixRelaxationPass.h`, `bolt/Core/ParallelUtilities.h`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
