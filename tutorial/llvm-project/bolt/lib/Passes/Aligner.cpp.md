# Aligner.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/Aligner.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: Pass for optimal code alignment. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：Pass for optimal code alignment。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Passes/Aligner.cpp - Pass for optimal code alignment ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the AlignerPass class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-21

```cpp
#include "bolt/Passes/Aligner.h"
#include "bolt/Core/ParallelUtilities.h"

#define DEBUG_TYPE "bolt-aligner"

using namespace llvm;

namespace opts {
```

- EN: Pulls in 2 header(s) from local project dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `opts` to organize symbols. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches.
- CN: 这里引入 2 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `opts` 中，用于组织符号作用域。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。

### Lines 22-32

```cpp
extern cl::OptionCategory BoltOptCategory;

extern cl::opt<bool> AlignBlocks;
extern cl::opt<bool> PreserveBlocksAlignment;
extern cl::opt<unsigned> AlignFunctions;

static cl::opt<unsigned> AlignBlocksMinSize(
    "align-blocks-min-size",
    cl::desc("minimal size of the basic block that should be aligned"),
    cl::init(0), cl::ZeroOrMore, cl::Hidden, cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `init`. Notable symbols here include `desc`, `init`.
- CN: 这里声明或实现函数，例如 `desc`, `init`。这里较值得关注的符号包括 `desc`, `init`。

### Lines 33-41

```cpp
static cl::opt<unsigned> AlignBlocksThreshold(
    "align-blocks-threshold",
    cl::desc(
        "align only blocks with frequency larger than containing function "
        "execution frequency specified in percent. E.g. 1000 means aligning "
        "blocks that are 10 times more frequently executed than the "
        "containing function."),
    cl::init(800), cl::Hidden, cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `init`. Notable symbols here include `init`.
- CN: 这里声明或实现函数，例如 `init`。这里较值得关注的符号包括 `init`。

### Lines 42-51

```cpp
static cl::opt<unsigned> AlignFunctionsMaxBytes(
    "align-functions-max-bytes",
    cl::desc("maximum number of bytes to use to align functions"), cl::init(32),
    cl::cat(BoltOptCategory));

static cl::opt<unsigned>
    BlockAlignment("block-alignment",
                   cl::desc("boundary to use for alignment of basic blocks"),
                   cl::init(16), cl::ZeroOrMore, cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `cat`, `init`. Notable symbols here include `desc`, `cat`, `init`.
- CN: 这里声明或实现函数，例如 `desc`, `cat`, `init`。这里较值得关注的符号包括 `desc`, `cat`, `init`。

### Lines 52-61

```cpp
static cl::opt<bool>
    UseCompactAligner("use-compact-aligner",
                      cl::desc("Use compact approach for aligning functions"),
                      cl::init(true), cl::cat(BoltOptCategory));

} // end namespace opts

namespace llvm {
namespace bolt {
```

- EN: Works inside namespace scope `opts`, `llvm`, `bolt` to organize symbols. Declares or implements routines including `desc`, `init`. Notable symbols here include `desc`, `init`, `opts`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `opts`, `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `desc`, `init`。这里较值得关注的符号包括 `desc`, `init`, `opts`, `llvm`, `bolt`。

### Lines 62-69

```cpp
// Align function to the specified byte-boundary (typically, 64) offsetting
// the function by not more than the corresponding value
static void alignMaxBytes(BinaryFunction &Function) {
  Function.setAlignment(opts::AlignFunctions);
  Function.setMaxAlignmentBytes(opts::AlignFunctionsMaxBytes);
  Function.setMaxColdAlignmentBytes(opts::AlignFunctionsMaxBytes);
}
```

- EN: Declares or implements routines including `alignMaxBytes`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `alignMaxBytes`.
- CN: 这里声明或实现函数，例如 `alignMaxBytes`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `alignMaxBytes`。

### Lines 70-79

```cpp
// Align function to the specified byte-boundary (typically, 64) offsetting
// the function by not more than the minimum over
// -- the size of the function
// -- the specified number of bytes
static void alignCompact(BinaryFunction &Function,
                         const MCCodeEmitter *Emitter) {
  const BinaryContext &BC = Function.getBinaryContext();
  size_t HotSize = 0;
  size_t ColdSize = 0;
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 80-92

```cpp
  // On AArch64, larger cold code size may lead to more veneers and higher
  // potential overhead for hot code. Minimize the cold code size.
  if (!Function.hasProfile() && BC.isAArch64()) {
    Function.setAlignment(Function.getMinAlignment());
    return;
  }

  for (const BinaryBasicBlock &BB : Function)
    if (BB.isSplit())
      ColdSize += BC.computeCodeSize(BB.begin(), BB.end(), Emitter);
    else
      HotSize += BC.computeCodeSize(BB.begin(), BB.end(), Emitter);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 93-104

```cpp
  Function.setAlignment(opts::AlignFunctions);
  if (HotSize > 0)
    Function.setMaxAlignmentBytes(
      std::min(size_t(opts::AlignFunctionsMaxBytes), HotSize));

  // using the same option, max-align-bytes, both for cold and hot parts of the
  // functions, as aligning cold functions typically does not affect performance
  if (ColdSize > 0)
    Function.setMaxColdAlignmentBytes(
      std::min(size_t(opts::AlignFunctionsMaxBytes), ColdSize));
}
```

- EN: Declares or implements routines including `min`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `min`.
- CN: 这里声明或实现函数，例如 `min`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `min`。

### Lines 105-117

```cpp
void AlignerPass::alignBlocks(BinaryFunction &Function,
                              const MCCodeEmitter *Emitter) {
  if (!Function.hasValidProfile() || !Function.isSimple())
    return;

  const BinaryContext &BC = Function.getBinaryContext();

  const uint64_t FuncCount =
      std::max<uint64_t>(1, Function.getKnownExecutionCount());
  BinaryBasicBlock *PrevBB = nullptr;
  for (BinaryBasicBlock *BB : Function.getLayout().blocks()) {
    uint64_t Count = BB->getKnownExecutionCount();
```

- EN: Declares or implements routines including `getKnownExecutionCount`. Notable symbols here include `getKnownExecutionCount`.
- CN: 这里声明或实现函数，例如 `getKnownExecutionCount`。这里较值得关注的符号包括 `getKnownExecutionCount`。

### Lines 118-126

```cpp
    if (Count <= FuncCount * opts::AlignBlocksThreshold / 100) {
      PrevBB = BB;
      continue;
    }

    uint64_t FTCount = 0;
    if (PrevBB && PrevBB->getFallthrough() == BB)
      FTCount = PrevBB->getBranchInfo(*BB).Count;
```

- EN: Declares or implements routines including `getBranchInfo`. Notable symbols here include `getBranchInfo`.
- CN: 这里声明或实现函数，例如 `getBranchInfo`。这里较值得关注的符号包括 `getBranchInfo`。

### Lines 127-136

```cpp
    PrevBB = BB;

    if (Count < FTCount * 2)
      continue;

    const uint64_t BlockSize =
        BC.computeCodeSize(BB->begin(), BB->end(), Emitter);
    const uint64_t BytesToUse =
        std::min<uint64_t>(opts::BlockAlignment - 1, BlockSize);
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 137-151

```cpp
    if (opts::AlignBlocksMinSize && BlockSize < opts::AlignBlocksMinSize)
      continue;

    BB->setAlignment(opts::BlockAlignment);
    BB->setAlignmentMaxBytes(BytesToUse);

    // Update stats.
    LLVM_DEBUG(
      std::unique_lock<llvm::sys::RWMutex> Lock(AlignHistogramMtx);
      AlignHistogram[BytesToUse]++;
      AlignedBlocksCount += BB->getKnownExecutionCount();
    );
  }
}
```

- EN: Declares or implements routines including `setAlignment`, `setAlignmentMaxBytes`, `Lock`, `getKnownExecutionCount`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `setAlignment`, `setAlignmentMaxBytes`, `Lock`, `getKnownExecutionCount`.
- CN: 这里声明或实现函数，例如 `setAlignment`, `setAlignmentMaxBytes`, `Lock`, `getKnownExecutionCount`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `setAlignment`, `setAlignmentMaxBytes`, `Lock`, `getKnownExecutionCount`。

### Lines 152-162

```cpp
Error AlignerPass::runOnFunctions(BinaryContext &BC) {
  if (!BC.HasRelocations)
    return Error::success();

  AlignHistogram.resize(opts::BlockAlignment);

  ParallelUtilities::WorkFuncTy WorkFun = [&](BinaryFunction &BF) {
    // Create a separate MCCodeEmitter to allow lock free execution
    BinaryContext::IndependentCodeEmitter Emitter =
        BC.createIndependentMCCodeEmitter();
```

- EN: Declares or implements routines including `runOnFunctions`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 163-171

```cpp
    if (opts::UseCompactAligner)
      alignCompact(BF, Emitter.MCE.get());
    else
      alignMaxBytes(BF);

    if (opts::AlignBlocks && !opts::PreserveBlocksAlignment)
      alignBlocks(BF, Emitter.MCE.get());
  };
```

- EN: Declares or implements routines including `alignCompact`, `alignMaxBytes`, `alignBlocks`. Notable symbols here include `alignCompact`, `alignMaxBytes`, `alignBlocks`.
- CN: 这里声明或实现函数，例如 `alignCompact`, `alignMaxBytes`, `alignBlocks`。这里较值得关注的符号包括 `alignCompact`, `alignMaxBytes`, `alignBlocks`。

### Lines 172-180

```cpp
  ParallelUtilities::runOnEachFunction(
      BC, ParallelUtilities::SchedulingPolicy::SP_TRIVIAL, WorkFun,
      ParallelUtilities::PredicateTy(nullptr), "AlignerPass");

  LLVM_DEBUG(
    dbgs() << "BOLT-DEBUG: max bytes per basic block alignment distribution:\n";
    for (unsigned I = 1; I < AlignHistogram.size(); ++I)
      dbgs() << "  " << I << " : " << AlignHistogram[I] << '\n';
```

- EN: Declares or implements routines including `PredicateTy`, `dbgs`. Notable symbols here include `PredicateTy`, `dbgs`.
- CN: 这里声明或实现函数，例如 `PredicateTy`, `dbgs`。这里较值得关注的符号包括 `PredicateTy`, `dbgs`。

### Lines 181-188

```cpp
    dbgs() << "BOLT-DEBUG: total execution count of aligned blocks: "
           << AlignedBlocksCount << '\n';
  );
  return Error::success();
}

} // end namespace bolt
} // end namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Declares or implements routines including `dbgs`. Notable symbols here include `dbgs`, `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里声明或实现函数，例如 `dbgs`。这里较值得关注的符号包括 `dbgs`, `bolt`, `llvm`。

## Key Concepts / 关键概念

- `desc`: function or method entry point / 函数或方法入口
- `init`: function or method entry point / 函数或方法入口
- `cat`: function or method entry point / 函数或方法入口
- `alignMaxBytes`: function or method entry point / 函数或方法入口
- `min`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域
- `opts`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/Aligner.h`, `bolt/Core/ParallelUtilities.h`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
