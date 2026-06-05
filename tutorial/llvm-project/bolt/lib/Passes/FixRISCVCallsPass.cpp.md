# FixRISCVCallsPass.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/FixRISCVCallsPass.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: *- C++ -*. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：*- C++ -*。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
//===- bolt/Passes/FixRISCVCallsPass.cpp ------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 9-18

```cpp
#include "bolt/Passes/FixRISCVCallsPass.h"
#include "bolt/Core/ParallelUtilities.h"

#include <iterator>

using namespace llvm;

namespace llvm {
namespace bolt {
```

- EN: Pulls in 3 header(s) from local project, system dependencies needed by this range. Works inside namespace scope `llvm`, `bolt` to organize symbols. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `llvm`, `bolt`.
- CN: 这里引入 3 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `llvm`, `bolt`。

### Lines 19-29

```cpp
void FixRISCVCallsPass::runOnFunction(BinaryFunction &BF) {
  auto &BC = BF.getBinaryContext();
  auto &MIB = BC.MIB;
  auto *Ctx = BC.Ctx.get();

  for (auto &BB : BF) {
    for (auto II = BB.begin(); II != BB.end();) {
      if (MIB->isCall(*II) && !MIB->isIndirectCall(*II)) {
        auto *Target = MIB->getTargetSymbol(*II);
        assert(Target && "Cannot find call target");
```

- EN: Declares or implements routines including `runOnFunction`, `getTargetSymbol`, `assert`. Notable symbols here include `runOnFunction`, `getTargetSymbol`, `assert`.
- CN: 这里声明或实现函数，例如 `runOnFunction`, `getTargetSymbol`, `assert`。这里较值得关注的符号包括 `runOnFunction`, `getTargetSymbol`, `assert`。

### Lines 30-37

```cpp
        MCInst OldCall = *II;
        auto L = BC.scopeLock();

        if (MIB->isTailCall(*II))
          MIB->createTailCall(*II, Target, Ctx);
        else
          MIB->createCall(*II, Target, Ctx);
```

- EN: Declares or implements routines including `createTailCall`, `createCall`. Notable symbols here include `createTailCall`, `createCall`.
- CN: 这里声明或实现函数，例如 `createTailCall`, `createCall`。这里较值得关注的符号包括 `createTailCall`, `createCall`。

### Lines 38-47

```cpp
        MIB->moveAnnotations(std::move(OldCall), *II);
        ++II;
        continue;
      }

      auto NextII = std::next(II);

      if (NextII == BB.end())
        break;
```

- EN: Declares or implements routines including `moveAnnotations`, `next`. Notable symbols here include `moveAnnotations`, `next`.
- CN: 这里声明或实现函数，例如 `moveAnnotations`, `next`。这里较值得关注的符号包括 `moveAnnotations`, `next`。

### Lines 48-56

```cpp
      if (MIB->isRISCVCall(*II, *NextII)) {
        auto *Target = MIB->getTargetSymbol(*II);
        assert(Target && "Cannot find call target");

        MCInst OldCall = *NextII;
        auto L = BC.scopeLock();

        MIB->createNoop(*II);
```

- EN: Declares or implements routines including `getTargetSymbol`, `assert`, `createNoop`. Notable symbols here include `getTargetSymbol`, `assert`, `createNoop`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`, `assert`, `createNoop`。这里较值得关注的符号包括 `getTargetSymbol`, `assert`, `createNoop`。

### Lines 57-67

```cpp
        if (MIB->isTailCall(*NextII))
          MIB->createTailCall(*NextII, Target, Ctx);
        else
          MIB->createCall(*NextII, Target, Ctx);

        MIB->moveAnnotations(std::move(OldCall), *NextII);

        II = std::next(NextII);
        continue;
      }
```

- EN: Declares or implements routines including `createTailCall`, `createCall`, `moveAnnotations`, `next`. Notable symbols here include `createTailCall`, `createCall`, `moveAnnotations`, `next`.
- CN: 这里声明或实现函数，例如 `createTailCall`, `createCall`, `moveAnnotations`, `next`。这里较值得关注的符号包括 `createTailCall`, `createCall`, `moveAnnotations`, `next`。

### Lines 68-76

```cpp
      ++II;
    }
  }
}

Error FixRISCVCallsPass::runOnFunctions(BinaryContext &BC) {
  if (!BC.isRISCV() || !BC.HasRelocations)
    return Error::success();
```

- EN: Declares or implements routines including `runOnFunctions`. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 77-84

```cpp
  ParallelUtilities::WorkFuncTy WorkFun = [&](BinaryFunction &BF) {
    runOnFunction(BF);
  };

  ParallelUtilities::runOnEachFunction(
      BC, ParallelUtilities::SchedulingPolicy::SP_INST_LINEAR, WorkFun, nullptr,
      "FixRISCVCalls");
```

- EN: Declares or implements routines including `runOnFunction`. Notable symbols here include `runOnFunction`.
- CN: 这里声明或实现函数，例如 `runOnFunction`。这里较值得关注的符号包括 `runOnFunction`。

### Lines 85-89

```cpp
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
- `assert`: function or method entry point / 函数或方法入口
- `createTailCall`: function or method entry point / 函数或方法入口
- `createCall`: function or method entry point / 函数或方法入口
- `llvm`: namespace scope / 命名空间作用域
- `bolt`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/FixRISCVCallsPass.h`, `bolt/Core/ParallelUtilities.h`
- System headers / 系统头文件: `iterator`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
