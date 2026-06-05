# ValidateMemRefs.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/ValidateMemRefs.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Passes/ValidateMemRefs.cpp. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：bolt/Passes/ValidateMemRefs.cpp。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
//===- bolt/Passes/ValidateMemRefs.cpp ------------------------------------===//
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
#include "bolt/Passes/ValidateMemRefs.h"
#include "bolt/Core/ParallelUtilities.h"

#define DEBUG_TYPE "bolt-memrefs"

namespace opts {
extern llvm::cl::opt<llvm::bolt::JumpTableSupportLevel> JumpTables;
}
```

- EN: Pulls in 2 header(s) from local project dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `opts` to organize symbols. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches.
- CN: 这里引入 2 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `opts` 中，用于组织符号作用域。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。

### Lines 18-31

```cpp
namespace llvm::bolt {

std::atomic<std::uint64_t> ValidateMemRefs::ReplacedReferences{0};

bool ValidateMemRefs::checkAndFixJTReference(BinaryFunction &BF, MCInst &Inst,
                                             uint32_t OperandNum,
                                             const MCSymbol *Sym,
                                             uint64_t Offset) {
  BinaryContext &BC = BF.getBinaryContext();
  auto L = BC.scopeLock();
  BinaryData *BD = BC.getBinaryDataByName(Sym->getName());
  if (!BD)
    return false;
```

- EN: Works inside namespace scope `llvm` to organize symbols. Contains local control flow that updates state or selects among execution branches. Notable symbols here include `llvm`.
- CN: 这里位于命名空间 `llvm` 中，用于组织符号作用域。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里较值得关注的符号包括 `llvm`。

### Lines 32-39

```cpp
  JumpTable *JT = BC.getJumpTableContainingAddress(BD->getAddress());
  if (!JT)
    return false;

  const bool IsLegitAccess = llvm::is_contained(JT->Parents, &BF);
  if (IsLegitAccess)
    return true;
```

- EN: Declares or implements routines including `is_contained`. Notable symbols here include `is_contained`.
- CN: 这里声明或实现函数，例如 `is_contained`。这里较值得关注的符号包括 `is_contained`。

### Lines 40-54

```cpp
  // Accessing a jump table in another function. This is not a
  // legitimate jump table access, we need to replace the reference to
  // the jump table label with a regular rodata reference. Get a
  // non-JT reference by fetching the symbol 1 byte before the JT
  // label.
  MCSymbol *NewSym = BC.getOrCreateGlobalSymbol(BD->getAddress() - 1, "DATAat");
  BC.MIB->setOperandToSymbolRef(Inst, OperandNum, NewSym, Offset + 1, &*BC.Ctx,
                                0);
  LLVM_DEBUG(dbgs() << "BOLT-DEBUG: replaced reference @" << BF.getPrintName()
                    << " from " << BD->getName() << " to " << NewSym->getName()
                    << " + 1\n");
  ++ReplacedReferences;
  return true;
}
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `getName`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`, `getName`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `getName`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`, `getName`。

### Lines 55-64

```cpp
void ValidateMemRefs::runOnFunction(BinaryFunction &BF) {
  MCPlusBuilder *MIB = BF.getBinaryContext().MIB.get();

  for (BinaryBasicBlock &BB : BF) {
    for (MCInst &Inst : BB) {
      for (int I = 0, E = MCPlus::getNumPrimeOperands(Inst); I != E; ++I) {
        const MCOperand &Operand = Inst.getOperand(I);
        if (!Operand.isExpr())
          continue;
```

- EN: Declares or implements routines including `runOnFunction`. Notable symbols here include `runOnFunction`.
- CN: 这里声明或实现函数，例如 `runOnFunction`。这里较值得关注的符号包括 `runOnFunction`。

### Lines 65-74

```cpp
        const auto [Sym, Offset] = MIB->getTargetSymbolInfo(Operand.getExpr());
        if (!Sym)
          continue;

        checkAndFixJTReference(BF, Inst, I, Sym, Offset);
      }
    }
  }
}
```

- EN: Declares or implements routines including `getTargetSymbolInfo`, `checkAndFixJTReference`. Notable symbols here include `getTargetSymbolInfo`, `checkAndFixJTReference`.
- CN: 这里声明或实现函数，例如 `getTargetSymbolInfo`, `checkAndFixJTReference`。这里较值得关注的符号包括 `getTargetSymbolInfo`, `checkAndFixJTReference`。

### Lines 75-82

```cpp
Error ValidateMemRefs::runOnFunctions(BinaryContext &BC) {
  if (!BC.isX86())
    return Error::success();

  // Skip validation if not moving JT
  if (opts::JumpTables == JTS_NONE || opts::JumpTables == JTS_BASIC)
    return Error::success();
```

- EN: Declares or implements routines including `runOnFunctions`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 83-95

```cpp
  ParallelUtilities::WorkFuncWithAllocTy ProcessFunction =
      [&](BinaryFunction &BF, MCPlusBuilder::AllocatorIdTy AllocId) {
        runOnFunction(BF);
      };
  ParallelUtilities::PredicateTy SkipPredicate = [&](const BinaryFunction &BF) {
    return !BF.hasCFG();
  };
  LLVM_DEBUG(dbgs() << "BOLT-DEBUG: starting memrefs validation pass\n");
  ParallelUtilities::runOnEachFunctionWithUniqueAllocId(
      BC, ParallelUtilities::SchedulingPolicy::SP_INST_LINEAR, ProcessFunction,
      SkipPredicate, "validate-mem-refs", /*ForceSequential=*/true);
  LLVM_DEBUG(dbgs() << "BOLT-DEBUG: memrefs validation is concluded\n");
```

- EN: Declares or implements routines including `runOnFunction`, `LLVM_DEBUG`. Notable symbols here include `runOnFunction`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `runOnFunction`, `LLVM_DEBUG`。这里较值得关注的符号包括 `runOnFunction`, `LLVM_DEBUG`。

### Lines 96-103

```cpp
  if (!ReplacedReferences)
    return Error::success();

  BC.outs() << "BOLT-INFO: validate-mem-refs updated " << ReplacedReferences
            << " object references\n";
  return Error::success();
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 104-104

```cpp
} // namespace llvm::bolt
```

- EN: Works inside namespace scope `llvm` to organize symbols. Notable symbols here include `llvm`.
- CN: 这里位于命名空间 `llvm` 中，用于组织符号作用域。这里较值得关注的符号包括 `llvm`。

## Key Concepts / 关键概念

- `is_contained`: function or method entry point / 函数或方法入口
- `LLVM_DEBUG`: function or method entry point / 函数或方法入口
- `getName`: function or method entry point / 函数或方法入口
- `runOnFunction`: function or method entry point / 函数或方法入口
- `getTargetSymbolInfo`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `opts`: namespace scope / 命名空间作用域
- `llvm`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/ValidateMemRefs.h`, `bolt/Core/ParallelUtilities.h`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
