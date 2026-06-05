# VeneerElimination.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/VeneerElimination.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Passes/VeneerElimination.cpp This class implements a pass that removes linker-inserted veneers from the code and redirects veneer callers to call to veneers destinations. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：bolt/Passes/VeneerElimination.cpp This class implements a pass that removes linker-inserted veneers from the code and redirects veneer callers to call to veneers destinations。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13

```cpp
//===- bolt/Passes/VeneerElimination.cpp ----------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This class implements a pass that removes linker-inserted veneers from the
// code and redirects veneer callers to call to veneers destinations
//
//===----------------------------------------------------------------------===//
```

- EN: Introduces type definitions such as `implements`. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `implements`.
- CN: 这里引入类型定义，例如 `implements`。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `implements`。

### Lines 14-22

```cpp
#include "bolt/Passes/VeneerElimination.h"
#define DEBUG_TYPE "veneer-elim"

using namespace llvm;

namespace opts {

extern cl::OptionCategory BoltOptCategory;
```

- EN: Pulls in 1 header(s) from local project dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `opts` to organize symbols. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches.
- CN: 这里引入 1 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `opts` 中，用于组织符号作用域。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。

### Lines 23-31

```cpp
static llvm::cl::opt<bool>
    EliminateVeneers("elim-link-veneers",
                     cl::desc("run veneer elimination pass"), cl::init(true),
                     cl::Hidden, cl::cat(BoltOptCategory));
} // namespace opts

namespace llvm {
namespace bolt {
```

- EN: Works inside namespace scope `opts`, `llvm`, `bolt` to organize symbols. Declares or implements routines including `desc`, `cat`. Notable symbols here include `desc`, `cat`, `opts`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `opts`, `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `desc`, `cat`。这里较值得关注的符号包括 `desc`, `cat`, `opts`, `llvm`, `bolt`。

### Lines 32-41

```cpp
Error VeneerElimination::runOnFunctions(BinaryContext &BC) {
  if (!opts::EliminateVeneers || !BC.isAArch64())
    return Error::success();

  std::unordered_map<const MCSymbol *, const MCSymbol *> VeneerDestinations;
  uint64_t NumEliminatedVeneers = 0;
  for (BinaryFunction &BF : llvm::make_second_range(BC.getBinaryFunctions())) {
    if (!BF.isPossibleVeneer())
      continue;
```

- EN: Declares or implements routines including `runOnFunctions`. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 42-57

```cpp
    if (BF.isIgnored())
      continue;

    MCInst &FirstInstruction = *(BF.begin()->begin());
    const MCSymbol *VeneerTargetSymbol = 0;
    uint64_t TargetAddress;
    if (BC.MIB->isTailCall(FirstInstruction)) {
      VeneerTargetSymbol = BC.MIB->getTargetSymbol(FirstInstruction);
    } else if (BC.MIB->matchAbsLongVeneer(BF, TargetAddress)) {
      if (BinaryFunction *TargetBF =
              BC.getBinaryFunctionAtAddress(TargetAddress))
        VeneerTargetSymbol = TargetBF->getSymbol();
    } else if (BC.MIB->hasAnnotation(FirstInstruction, "AArch64Veneer")) {
      VeneerTargetSymbol = BC.MIB->getTargetSymbol(FirstInstruction, 1);
    }
```

- EN: Declares or implements routines including `getTargetSymbol`, `if`, `getSymbol`. Notable symbols here include `getTargetSymbol`, `if`, `getSymbol`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`, `if`, `getSymbol`。这里较值得关注的符号包括 `getTargetSymbol`, `if`, `getSymbol`。

### Lines 58-67

```cpp
    if (!VeneerTargetSymbol)
      continue;

    for (const MCSymbol *Symbol : BF.getSymbols())
      VeneerDestinations[Symbol] = VeneerTargetSymbol;

    NumEliminatedVeneers++;
    BF.setPseudo(true);
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 68-77

```cpp
  BC.outs() << "BOLT-INFO: number of removed linker-inserted veneers: "
            << NumEliminatedVeneers << '\n';

  // Handle veneers to veneers in case they occur
  for (auto &Entry : VeneerDestinations) {
    const MCSymbol *Src = Entry.first;
    const MCSymbol *Dest = Entry.second;
    while (VeneerDestinations.find(Dest) != VeneerDestinations.end())
      Dest = VeneerDestinations[Dest];
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 78-87

```cpp
    VeneerDestinations[Src] = Dest;
  }

  uint64_t VeneerCallers = 0;
  for (BinaryFunction &BF : llvm::make_second_range(BC.getBinaryFunctions())) {
    for (BinaryBasicBlock &BB : BF) {
      for (MCInst &Instr : BB) {
        if (!BC.MIB->isCall(Instr) || BC.MIB->isIndirectCall(Instr))
          continue;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 88-98

```cpp
        const MCSymbol *TargetSymbol = BC.MIB->getTargetSymbol(Instr, 0);
        auto It = VeneerDestinations.find(TargetSymbol);
        if (It == VeneerDestinations.end())
          continue;

        VeneerCallers++;
        BC.MIB->replaceBranchTarget(Instr, It->second, BC.Ctx.get());
      }
    }
  }
```

- EN: Declares or implements routines including `getTargetSymbol`, `replaceBranchTarget`. Notable symbols here include `getTargetSymbol`, `replaceBranchTarget`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`, `replaceBranchTarget`。这里较值得关注的符号包括 `getTargetSymbol`, `replaceBranchTarget`。

### Lines 99-107

```cpp
  LLVM_DEBUG(
      dbgs() << "BOLT-INFO: number of linker-inserted veneers call sites: "
             << VeneerCallers << "\n");
  (void)VeneerCallers;
  return Error::success();
}

} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Declares or implements routines including `dbgs`. Notable symbols here include `dbgs`, `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里声明或实现函数，例如 `dbgs`。这里较值得关注的符号包括 `dbgs`, `bolt`, `llvm`。

## Key Concepts / 关键概念

- `implements`: class or struct interface / 类或结构体接口
- `desc`: function or method entry point / 函数或方法入口
- `cat`: function or method entry point / 函数或方法入口
- `runOnFunctions`: function or method entry point / 函数或方法入口
- `getTargetSymbol`: function or method entry point / 函数或方法入口
- `if`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/VeneerElimination.h`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
