# PointerAuthCFIAnalyzer.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/PointerAuthCFIAnalyzer.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Passes/PointerAuthCFIAnalyzer.cpp This file implements the PointerAuthCFIAnalyzer class. Three CFIs have an influence on the RA State of an instruction NegateRAState flips the RA State,. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：bolt/Passes/PointerAuthCFIAnalyzer.cpp This file implements the PointerAuthCFIAnalyzer class. Three CFIs have an influence on the RA State of an instruction NegateRAState flips the RA State,。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```cpp
//===- bolt/Passes/PointerAuthCFIAnalyzer.cpp -----------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the PointerAuthCFIAnalyzer class.
// Three CFIs have an influence on the RA State of an instruction:
// - NegateRAState flips the RA State,
// - RememberState pushes the RA State to a stack,
// - RestoreState pops the RA State from the stack.
// These are saved as MCAnnotations on instructions they refer to at CFI
// reading (in CFIReaderWriter::fillCFIInfoFor). In this pass, we can work out
// the RA State of each instruction, and save it as new MCAnnotations. The new
// annotations are Signing, Signed, Authenticating and Unsigned. After
// optimizations, .cfi_negate_ra_state CFIs are added to the places where the
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 19-28

```cpp
// state changes in PointerAuthCFIFixup.
//
//===----------------------------------------------------------------------===//
#include "bolt/Passes/PointerAuthCFIAnalyzer.h"
#include "bolt/Core/BinaryFunction.h"
#include "bolt/Core/ParallelUtilities.h"
#include <cstdlib>
#include <optional>
#include <stack>
```

- EN: Pulls in 6 header(s) from local project, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 6 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 29-37

```cpp
using namespace llvm;

namespace opts {
extern llvm::cl::opt<unsigned> Verbosity;
} // namespace opts

namespace llvm {
namespace bolt {
```

- EN: Works inside namespace scope `llvm`, `opts`, `bolt` to organize symbols. Notable symbols here include `llvm`, `opts`, `bolt`.
- CN: 这里位于命名空间 `llvm`, `opts`, `bolt` 中，用于组织符号作用域。这里较值得关注的符号包括 `llvm`, `opts`, `bolt`。

### Lines 38-55

```cpp
bool PointerAuthCFIAnalyzer::runOnFunction(BinaryFunction &BF) {

  BinaryContext &BC = BF.getBinaryContext();

  for (const BinaryBasicBlock &BB : BF) {
    for (const MCInst &Inst : BB) {
      if ((BC.MIB->isPSignOnLR(Inst) ||
           (BC.MIB->isPAuthOnLR(Inst) && !BC.MIB->isPAuthAndRet(Inst))) &&
          !BC.MIB->hasNegateRAState(Inst)) {
        // Not all functions have .cfi_negate_ra_state in them. But if one does,
        // we expect psign/pauth instructions to have the hasNegateRAState
        // annotation.
        if (opts::Verbosity >= 1)
          BC.outs() << "BOLT-INFO: inconsistent RAStates in function "
                    << BF.getPrintName()
                    << ": ptr sign/auth inst without .cfi_negate_ra_state\n";
        std::lock_guard<std::mutex> Lock(IgnoreMutex);
        BF.setIgnored();
```

- EN: Declares or implements routines including `runOnFunction`, `hasNegateRAState`, `Lock`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `runOnFunction`, `hasNegateRAState`, `Lock`.
- CN: 这里声明或实现函数，例如 `runOnFunction`, `hasNegateRAState`, `Lock`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `runOnFunction`, `hasNegateRAState`, `Lock`。

### Lines 56-64

```cpp
        return false;
      }
    }
  }

  bool RAState = BF.getInitialRAState();
  std::stack<bool> RAStateStack;
  RAStateStack.push(RAState);
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 65-82

```cpp
  for (BinaryBasicBlock &BB : BF) {
    for (MCInst &Inst : BB) {
      if (BC.MIB->isCFI(Inst))
        continue;

      if (BC.MIB->isPSignOnLR(Inst)) {
        if (RAState) {
          // RA signing instructions should only follow unsigned RA state.
          if (opts::Verbosity >= 1)
            BC.outs() << "BOLT-INFO: inconsistent RAStates in function "
                      << BF.getPrintName()
                      << ": ptr signing inst encountered in Signed RA state\n";
          std::lock_guard<std::mutex> Lock(IgnoreMutex);
          BF.setIgnored();
          return false;
        }
      } else if (BC.MIB->isPAuthOnLR(Inst)) {
        if (!RAState) {
```

- EN: Declares or implements routines including `Lock`, `if`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `Lock`, `if`.
- CN: 这里声明或实现函数，例如 `Lock`, `if`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `Lock`, `if`。

### Lines 83-94

```cpp
          // RA authenticating instructions should only follow signed RA state.
          if (opts::Verbosity >= 1)
            BC.outs() << "BOLT-INFO: inconsistent RAStates in function "
                      << BF.getPrintName()
                      << ": ptr authenticating inst encountered in Unsigned RA "
                         "state\n";
          std::lock_guard<std::mutex> Lock(IgnoreMutex);
          BF.setIgnored();
          return false;
        }
      }
```

- EN: Declares or implements routines including `Lock`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `Lock`.
- CN: 这里声明或实现函数，例如 `Lock`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `Lock`。

### Lines 95-112

```cpp
      BC.MIB->setRAState(Inst, RAState);

      // Updating RAState. All updates are valid from the next instruction.
      // Because the same instruction can have remember and restore, the order
      // here is relevant. This is the reason to loop over Annotations instead
      // of just checking each in a predefined order.
      for (unsigned int Idx = 0; Idx < Inst.getNumOperands(); Idx++) {
        std::optional<int64_t> Annotation =
            BC.MIB->getAnnotationAtOpIndex(Inst, Idx);
        if (!Annotation)
          continue;
        if (Annotation == MCPlus::MCAnnotation::kNegateState)
          RAState = !RAState;
        else if (Annotation == MCPlus::MCAnnotation::kRememberState)
          RAStateStack.push(RAState);
        else if (Annotation == MCPlus::MCAnnotation::kRestoreState) {
          RAState = RAStateStack.top();
          RAStateStack.pop();
```

- EN: Declares or implements routines including `setRAState`, `getAnnotationAtOpIndex`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `setRAState`, `getAnnotationAtOpIndex`.
- CN: 这里声明或实现函数，例如 `setRAState`, `getAnnotationAtOpIndex`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `setRAState`, `getAnnotationAtOpIndex`。

### Lines 113-127

```cpp
        }
      }
    }
  }
  return true;
}

Error PointerAuthCFIAnalyzer::runOnFunctions(BinaryContext &BC) {
  std::atomic<uint64_t> FunctionsIgnored{0};
  ParallelUtilities::WorkFuncTy WorkFun = [&](BinaryFunction &BF) {
    if (!runOnFunction(BF)) {
      FunctionsIgnored++;
    }
  };
```

- EN: Declares or implements routines including `runOnFunctions`. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 128-135

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

### Lines 136-146

```cpp
  int Total = llvm::count_if(BC.getBinaryFunctions(), [&](auto &P) {
    return P.second.containedNegateRAState() && !P.second.isIgnored();
  });

  if (Total == 0)
    return Error::success();

  ParallelUtilities::runOnEachFunction(
      BC, ParallelUtilities::SchedulingPolicy::SP_INST_LINEAR, WorkFun,
      SkipPredicate, "PointerAuthCFIAnalyzer");
```

- EN: Declares or implements routines including `count_if`. Notable symbols here include `count_if`.
- CN: 这里声明或实现函数，例如 `count_if`。这里较值得关注的符号包括 `count_if`。

### Lines 147-164

```cpp
  float IgnoredPercent = (100.0 * FunctionsIgnored) / Total;
  BC.outs() << "BOLT-INFO: PointerAuthCFIAnalyzer ran on " << Total
            << " functions. Ignored " << FunctionsIgnored << " functions "
            << format("(%.2lf%%)", IgnoredPercent)
            << " because of CFI inconsistencies\n";

  // Errors in the input are expected from two sources:
  // - compilers emitting incorrect CFIs. This happens more frequently with
  //   older compiler versions, but it should not account for a large
  //   percentage.
  // - input binary is using synchronous unwind tables. This means that after
  //   call sites, the unwind CFIs are dropped: the pass sees missing
  //   .cfi_negate_ra_state from autiasp instructions. If this is the case, a
  //   larger percentage of functions will be ignored.
  //
  // This is why the 10% threshold was chosen: we should not warn about
  // synchronous unwind tables if only a few % are ignored.
  if (IgnoredPercent >= 10.0)
```

- EN: Declares or implements routines including `format`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `format`.
- CN: 这里声明或实现函数，例如 `format`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `format`。

### Lines 165-173

```cpp
    BC.outs() << "BOLT-WARNING: PointerAuthCFIAnalyzer only supports "
                 "asynchronous unwind tables. For C compilers, see "
                 "-fasynchronous-unwind-tables.\n";

  return Error::success();
}

} // end namespace bolt
} // end namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Contains local control flow that updates state or selects among execution branches. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `runOnFunction`: function or method entry point / 函数或方法入口
- `hasNegateRAState`: function or method entry point / 函数或方法入口
- `Lock`: function or method entry point / 函数或方法入口
- `if`: function or method entry point / 函数或方法入口
- `setRAState`: function or method entry point / 函数或方法入口
- `llvm`: namespace scope / 命名空间作用域
- `opts`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/PointerAuthCFIAnalyzer.h`, `bolt/Core/BinaryFunction.h`, `bolt/Core/ParallelUtilities.h`
- System headers / 系统头文件: `cstdlib`, `optional`, `stack`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
