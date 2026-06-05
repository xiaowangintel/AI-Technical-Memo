# PatchEntries.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/PatchEntries.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Passes/PatchEntries.cpp - Pass for patching function entries This file implements the PatchEntries class that is used for patching the original function entry points. This ensures that only the new/optimized code executes and that the old code is never used. This is necessary due to. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：bolt/Passes/PatchEntries.cpp - Pass for patching function entries This file implements the PatchEntries class that is used for patching the original function entry points. This ensures that only the new/optimized code executes and that the old code is never used. This is necessary due to。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```cpp
//===- bolt/Passes/PatchEntries.cpp - Pass for patching function entries --===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the PatchEntries class that is used for patching the
// original function entry points. This ensures that only the new/optimized code
// executes and that the old code is never used. This is necessary due to
// current BOLT limitations of not being able to duplicate all function's
// associated metadata (e.g., .eh_frame, exception ranges, debug info,
// jump-tables).
//
// NOTE: A successful run of 'scanExternalRefs' can relax this requirement as
// it also ensures that old code is never executed.
//
```

- EN: Introduces type definitions such as `that`. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `that`.
- CN: 这里引入类型定义，例如 `that`。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `that`。

### Lines 19-26

```cpp
//===----------------------------------------------------------------------===//

#include "bolt/Passes/PatchEntries.h"
#include "bolt/Utils/CommandLineOpts.h"
#include "bolt/Utils/NameResolver.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/Support/CommandLine.h"
```

- EN: Pulls in 5 header(s) from local project, LLVM dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 5 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 27-34

```cpp
namespace opts {
extern llvm::cl::OptionCategory BoltCategory;
extern llvm::cl::opt<unsigned> Verbosity;
} // namespace opts

namespace llvm {
namespace bolt {
```

- EN: Works inside namespace scope `opts`, `llvm`, `bolt` to organize symbols. Notable symbols here include `opts`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `opts`, `llvm`, `bolt` 中，用于组织符号作用域。这里较值得关注的符号包括 `opts`, `llvm`, `bolt`。

### Lines 35-43

```cpp
Error PatchEntries::runOnFunctions(BinaryContext &BC) {
  if (!opts::ForcePatch) {
    // Mark the binary for patching if we did not create external references
    // for original code in any of functions we are not going to emit.
    auto needsPatching = [&](const BinaryFunction &BF) {
      // FIXME: keep compatibility for NFC testing.
      if (BF.isFolded())
        return false;
```

- EN: Declares or implements routines including `runOnFunctions`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 44-55

```cpp
      // Patching is always needed if explicitly requested.
      if (BF.needsPatch())
        return true;

      return !BC.shouldEmit(BF) && !BF.hasExternalRefRelocations();
    };

    if (!llvm::any_of(llvm::make_second_range(BC.getBinaryFunctions()),
                      needsPatching))
      return Error::success();
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 56-66

```cpp
  if (opts::Verbosity >= 1)
    BC.outs() << "BOLT-INFO: patching entries in original code\n";

  // Calculate the size of the patch.
  static size_t PatchSize = 0;
  if (!PatchSize) {
    InstructionListType Seq;
    BC.MIB->createLongTailCall(Seq, BC.Ctx->createTempSymbol(), BC.Ctx.get());
    PatchSize = BC.computeCodeSize(Seq.begin(), Seq.end());
  }
```

- EN: Declares or implements routines including `createLongTailCall`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `createLongTailCall`.
- CN: 这里声明或实现函数，例如 `createLongTailCall`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `createLongTailCall`。

### Lines 67-78

```cpp
  for (auto &BFI : BC.getBinaryFunctions()) {
    BinaryFunction &Function = BFI.second;

    // Patch original code only for functions that will be emitted.
    if (!BC.shouldEmit(Function))
      continue;

    // Check if we can skip patching the function.
    if (!opts::ForcePatch && !Function.hasEHRanges() &&
        !Function.needsPatch() && Function.getSize() < PatchThreshold)
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 79-93

```cpp
    // List of patches for function entries. We either successfully patch
    // all entries or, if we cannot patch one or more, do no patch any and
    // mark the function as ignorable.
    std::vector<Patch> PendingPatches;

    uint64_t NextValidByte = 0; // offset of the byte past the last patch
    bool Success = Function.forEachEntryPoint([&](uint64_t Offset,
                                                  const MCSymbol *Symbol) {
      if (Offset < NextValidByte) {
        if (opts::Verbosity >= 1)
          BC.outs() << "BOLT-INFO: unable to patch entry point in " << Function
                    << " at offset 0x" << Twine::utohexstr(Offset) << '\n';
        return false;
      }
```

- EN: Declares or implements routines including `utohexstr`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `utohexstr`.
- CN: 这里声明或实现函数，例如 `utohexstr`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `utohexstr`。

### Lines 94-103

```cpp
      PendingPatches.emplace_back(
          Patch{Symbol, Function.getAddress() + Offset});
      NextValidByte = Offset + PatchSize;
      if (NextValidByte > Function.getMaxSize()) {
        if (opts::Verbosity >= 1)
          BC.outs() << "BOLT-INFO: function " << Function
                    << " too small to patch its entry point\n";
        return false;
      }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 104-115

```cpp
      return true;
    });

    if (!Success) {
      // If the original function entries cannot be patched, then we cannot
      // safely emit new function body.
      BC.errs() << "BOLT-WARNING: failed to patch entries in " << Function
                << ". The function will not be optimized\n";
      Function.setIgnored();
      continue;
    }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 116-126

```cpp
    for (Patch &Patch : PendingPatches) {
      // Add instruction patch to the binary.
      InstructionListType Instructions;
      BC.MIB->createLongTailCall(Instructions, Patch.Symbol, BC.Ctx.get());
      BinaryFunction *PatchFunction = BC.createInstructionPatch(
          Patch.Address, Instructions,
          NameResolver::append(Patch.Symbol->getName(), ".org.0"));
      if (BC.usesBTI())
        BC.MIB->applyBTIFixupToSymbol(BC, Patch.Symbol,
                                      *(Instructions.end() - 1));
```

- EN: Declares or implements routines including `createLongTailCall`, `append`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `createLongTailCall`, `append`.
- CN: 这里声明或实现函数，例如 `createLongTailCall`, `append`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `createLongTailCall`, `append`。

### Lines 127-136

```cpp
      // Verify the size requirements.
      uint64_t HotSize, ColdSize;
      std::tie(HotSize, ColdSize) = BC.calculateEmittedSize(*PatchFunction);
      assert(!ColdSize && "unexpected cold code");
      assert(HotSize <= PatchSize && "max patch size exceeded");
    }
  }
  return Error::success();
}
```

- EN: Declares or implements routines including `tie`, `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `tie`, `assert`.
- CN: 这里声明或实现函数，例如 `tie`, `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `tie`, `assert`。

### Lines 137-138

```cpp
} // end namespace bolt
} // end namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `that`: class or struct interface / 类或结构体接口
- `runOnFunctions`: function or method entry point / 函数或方法入口
- `createLongTailCall`: function or method entry point / 函数或方法入口
- `utohexstr`: function or method entry point / 函数或方法入口
- `append`: function or method entry point / 函数或方法入口
- `tie`: function or method entry point / 函数或方法入口
- `opts`: namespace scope / 命名空间作用域
- `llvm`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/PatchEntries.h`, `bolt/Utils/CommandLineOpts.h`, `bolt/Utils/NameResolver.h`
- LLVM headers / LLVM 头文件: `llvm/ADT/STLExtras.h`, `llvm/Support/CommandLine.h`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
