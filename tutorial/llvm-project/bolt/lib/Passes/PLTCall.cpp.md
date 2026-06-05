# PLTCall.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/PLTCall.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: PLT call optimization. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：PLT call optimization。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13

```cpp
//===- bolt/Passes/PLTCall.h - PLT call optimization ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the PLTCall class, which replaces calls to PLT entries
// with indirect calls against GOT.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 14-22

```cpp
#include "bolt/Passes/PLTCall.h"
#include "llvm/Support/CommandLine.h"

#define DEBUG_TYPE "bolt-plt"

using namespace llvm;

namespace opts {
```

- EN: Pulls in 2 header(s) from local project, LLVM dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `opts` to organize symbols. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches.
- CN: 这里引入 2 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `opts` 中，用于组织符号作用域。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。

### Lines 23-38

```cpp
extern cl::OptionCategory BoltOptCategory;

static cl::opt<bolt::PLTCall::OptType>
    PLT("plt",
        cl::desc("optimize PLT calls (requires linking with -znow on "
                 "non-x86 architectures)"),
        cl::init(bolt::PLTCall::OT_NONE),
        cl::values(clEnumValN(bolt::PLTCall::OT_NONE, "none",
                              "do not optimize PLT calls"),
                   clEnumValN(bolt::PLTCall::OT_HOT, "hot",
                              "optimize executed (hot) PLT calls"),
                   clEnumValN(bolt::PLTCall::OT_ALL, "all",
                              "optimize all PLT calls")),
        cl::ZeroOrMore, cl::cat(BoltOptCategory));
}
```

- EN: Declares or implements routines including `init`, `executed`, `cat`. Notable symbols here include `init`, `executed`, `cat`.
- CN: 这里声明或实现函数，例如 `init`, `executed`, `cat`。这里较值得关注的符号包括 `init`, `executed`, `cat`。

### Lines 39-51

```cpp
namespace llvm {
namespace bolt {

Error PLTCall::runOnFunctions(BinaryContext &BC) {
  if (opts::PLT == OT_NONE)
    return Error::success();

  uint64_t NumCallsOptimized = 0;
  for (auto &BFI : BC.getBinaryFunctions()) {
    BinaryFunction &Function = BFI.second;
    if (!shouldOptimize(Function))
      continue;
```

- EN: Works inside namespace scope `llvm`, `bolt` to organize symbols. Declares or implements routines including `runOnFunctions`. Notable symbols here include `runOnFunctions`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `runOnFunctions`。这里较值得关注的符号包括 `runOnFunctions`, `llvm`, `bolt`。

### Lines 52-59

```cpp
    if (opts::PLT == OT_HOT &&
        Function.getExecutionCount() == BinaryFunction::COUNT_NO_PROFILE)
      continue;

    for (BinaryBasicBlock &BB : Function) {
      if (opts::PLT == OT_HOT && !BB.getKnownExecutionCount())
        continue;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 60-77

```cpp
      for (auto II = BB.begin(); II != BB.end(); II++) {
        if (!BC.MIB->isCall(*II))
          continue;
        const MCSymbol *CallSymbol = BC.MIB->getTargetSymbol(*II);
        if (!CallSymbol)
          continue;
        const BinaryFunction *CalleeBF = BC.getFunctionForSymbol(CallSymbol);
        if (!CalleeBF || !CalleeBF->isPLTFunction())
          continue;
        const InstructionListType NewCode = BC.MIB->createIndirectPLTCall(
            std::move(*II), CalleeBF->getPLTSymbol(), BC.Ctx.get());
        II = BB.replaceInstruction(II, NewCode);
        assert(!NewCode.empty() && "PLT Call replacement must be non-empty");
        std::advance(II, NewCode.size() - 1);
        BC.MIB->addAnnotation(*II, "PLTCall", true);
        ++NumCallsOptimized;
      }
    }
```

- EN: Declares or implements routines including `getTargetSymbol`, `move`, `assert`, `advance`, `addAnnotation`. Notable symbols here include `getTargetSymbol`, `move`, `assert`, `advance`, `addAnnotation`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`, `move`, `assert`, `advance`, `addAnnotation`。这里较值得关注的符号包括 `getTargetSymbol`, `move`, `assert`, `advance`, `addAnnotation`。

### Lines 78-92

```cpp
  }

  if (NumCallsOptimized) {
    // On X86-64, PLT optimization does not require -znow because the indirect
    // call through GOT works correctly with lazy binding. At runtime, the
    // resolver will populate the GOT entry on first call just like with a
    // regular PLT call.
    if (!BC.isX86())
      BC.RequiresZNow = true;
    BC.outs() << "BOLT-INFO: " << NumCallsOptimized
              << " PLT calls in the binary were optimized.\n";
  }
  return Error::success();
}
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 93-94

```cpp
} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `init`: function or method entry point / 函数或方法入口
- `executed`: function or method entry point / 函数或方法入口
- `cat`: function or method entry point / 函数或方法入口
- `runOnFunctions`: function or method entry point / 函数或方法入口
- `getTargetSymbol`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域
- `opts`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/PLTCall.h`
- LLVM headers / LLVM 头文件: `llvm/Support/CommandLine.h`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
