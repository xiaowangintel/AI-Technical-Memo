# ParallelUtilities.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Core/ParallelUtilities.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: Parallel utilities. It also sits in code that implements BOLT core binary-analysis logic. / 该文件实现 BOLT 核心二进制分析逻辑。 源码头部说明其职责是：Parallel utilities。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Core/ParallelUtilities.cpp - Parallel utilities ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Implementation of the class that manages parallel work on BinaryFunctions.
//
//===----------------------------------------------------------------------===//
```

- EN: Introduces type definitions such as `that`. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `that`.
- CN: 这里引入类型定义，例如 `that`。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `that`。

### Lines 13-20

```cpp
#include "bolt/Core/ParallelUtilities.h"
#include "bolt/Core/BinaryContext.h"
#include "bolt/Core/BinaryFunction.h"
#include "llvm/Support/RWMutex.h"
#include "llvm/Support/ThreadPool.h"
#include "llvm/Support/Timer.h"
#include <mutex>
```

- EN: Pulls in 7 header(s) from local project, LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 7 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 21-31

```cpp
#define DEBUG_TYPE "par-utils"

namespace opts {
extern cl::OptionCategory BoltCategory;

cl::opt<unsigned>
ThreadCount("thread-count",
  cl::desc("number of threads"),
  cl::init(hardware_concurrency().compute_thread_count()),
  cl::cat(BoltCategory));
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `opts` to organize symbols. Declares or implements routines including `desc`, `init`, `cat`. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `opts` 中，用于组织符号作用域。这里声明或实现函数，例如 `desc`, `init`, `cat`。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。

### Lines 32-43

```cpp
cl::opt<bool>
NoThreads("no-threads",
  cl::desc("disable multithreading"),
  cl::init(false),
  cl::cat(BoltCategory));

cl::opt<unsigned>
TaskCount("tasks-per-thread",
  cl::desc("number of tasks to be created per thread"),
  cl::init(20),
  cl::cat(BoltCategory));
```

- EN: Declares or implements routines including `desc`, `init`, `cat`. Notable symbols here include `desc`, `init`, `cat`.
- CN: 这里声明或实现函数，例如 `desc`, `init`, `cat`。这里较值得关注的符号包括 `desc`, `init`, `cat`。

### Lines 44-53

```cpp
} // namespace opts

namespace llvm {
namespace bolt {
namespace ParallelUtilities {

namespace {
/// A single thread pool that is used to run parallel tasks
std::unique_ptr<ThreadPoolInterface> ThreadPoolPtr;
```

- EN: Works inside namespace scope `opts`, `llvm`, `bolt`, `ParallelUtilities` to organize symbols. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `opts`, `llvm`, `bolt`, `ParallelUtilities`.
- CN: 这里位于命名空间 `opts`, `llvm`, `bolt`, `ParallelUtilities` 中，用于组织符号作用域。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `opts`, `llvm`, `bolt`, `ParallelUtilities`。

### Lines 54-62

```cpp
unsigned computeCostFor(const BinaryFunction &BF,
                        const PredicateTy &SkipPredicate,
                        const SchedulingPolicy &SchedPolicy) {
  if (SchedPolicy == SchedulingPolicy::SP_TRIVIAL)
    return 1;

  if (SkipPredicate && SkipPredicate(BF))
    return 0;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 63-78

```cpp
  switch (SchedPolicy) {
  case SchedulingPolicy::SP_CONSTANT:
    return 1;
  case SchedulingPolicy::SP_INST_LINEAR:
    return BF.getSize();
  case SchedulingPolicy::SP_INST_QUADRATIC:
    return BF.getSize() * BF.getSize();
  case SchedulingPolicy::SP_BB_LINEAR:
    return BF.size();
  case SchedulingPolicy::SP_BB_QUADRATIC:
    return BF.size() * BF.size();
  default:
    llvm_unreachable("unsupported scheduling policy");
  }
}
```

- EN: Declares or implements routines including `llvm_unreachable`. Notable symbols here include `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`。这里较值得关注的符号包括 `llvm_unreachable`。

### Lines 79-90

```cpp
inline unsigned estimateTotalCost(const BinaryContext &BC,
                                  const PredicateTy &SkipPredicate,
                                  SchedulingPolicy &SchedPolicy) {
  if (SchedPolicy == SchedulingPolicy::SP_TRIVIAL)
    return BC.getBinaryFunctions().size();

  unsigned TotalCost = 0;
  for (auto &BFI : BC.getBinaryFunctions()) {
    const BinaryFunction &BF = BFI.second;
    TotalCost += computeCostFor(BF, SkipPredicate, SchedPolicy);
  }
```

- EN: Declares or implements routines including `computeCostFor`. Notable symbols here include `computeCostFor`.
- CN: 这里声明或实现函数，例如 `computeCostFor`。这里较值得关注的符号包括 `computeCostFor`。

### Lines 91-102

```cpp
  // Switch to trivial scheduling if total estimated work is zero
  if (TotalCost == 0) {
    BC.outs()
        << "BOLT-WARNING: Running parallel work of 0 estimated cost, will "
           "switch to  trivial scheduling.\n";

    SchedPolicy = SP_TRIVIAL;
    TotalCost = BC.getBinaryFunctions().size();
  }
  return TotalCost;
}
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 103-116

```cpp
} // namespace

ThreadPoolInterface &getThreadPool(const unsigned ThreadsCount) {
  if (ThreadPoolPtr)
    return *ThreadPoolPtr;

  if (ThreadsCount > 1)
    ThreadPoolPtr = std::make_unique<DefaultThreadPool>(
        llvm::hardware_concurrency(ThreadsCount));
  else
    ThreadPoolPtr = std::make_unique<SingleThreadExecutor>();
  return *ThreadPoolPtr;
}
```

- EN: Works inside namespace scope `ThreadPoolInterface` to organize symbols. Declares or implements routines including `getThreadPool`, `hardware_concurrency`. Notable symbols here include `getThreadPool`, `hardware_concurrency`, `ThreadPoolInterface`.
- CN: 这里位于命名空间 `ThreadPoolInterface` 中，用于组织符号作用域。这里声明或实现函数，例如 `getThreadPool`, `hardware_concurrency`。这里较值得关注的符号包括 `getThreadPool`, `hardware_concurrency`, `ThreadPoolInterface`。

### Lines 117-128

```cpp
void runOnEachFunction(BinaryContext &BC, SchedulingPolicy SchedPolicy,
                       WorkFuncTy WorkFunction, PredicateTy SkipPredicate,
                       std::string LogName, bool ForceSequential,
                       unsigned TasksPerThread) {
  if (BC.getBinaryFunctions().size() == 0)
    return;

  auto runBlock = [&](std::map<uint64_t, BinaryFunction>::iterator BlockBegin,
                      std::map<uint64_t, BinaryFunction>::iterator BlockEnd) {
    Timer T(LogName, LogName);
    LLVM_DEBUG(T.startTimer());
```

- EN: Declares or implements routines including `T`, `LLVM_DEBUG`. Notable symbols here include `T`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `T`, `LLVM_DEBUG`。这里较值得关注的符号包括 `T`, `LLVM_DEBUG`。

### Lines 129-138

```cpp
    for (auto It = BlockBegin; It != BlockEnd; ++It) {
      BinaryFunction &BF = It->second;
      if (SkipPredicate && SkipPredicate(BF))
        continue;

      WorkFunction(BF);
    }
    LLVM_DEBUG(T.stopTimer());
  };
```

- EN: Declares or implements routines including `WorkFunction`, `LLVM_DEBUG`. Notable symbols here include `WorkFunction`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `WorkFunction`, `LLVM_DEBUG`。这里较值得关注的符号包括 `WorkFunction`, `LLVM_DEBUG`。

### Lines 139-149

```cpp
  if (opts::NoThreads || ForceSequential) {
    runBlock(BC.getBinaryFunctions().begin(), BC.getBinaryFunctions().end());
    return;
  }

  // Estimate the overall runtime cost using the scheduling policy
  const unsigned TotalCost = estimateTotalCost(BC, SkipPredicate, SchedPolicy);
  const unsigned BlocksCount = TasksPerThread * opts::ThreadCount;
  const unsigned BlockCost =
      TotalCost > BlocksCount ? TotalCost / BlocksCount : 1;
```

- EN: Declares or implements routines including `runBlock`, `estimateTotalCost`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `runBlock`, `estimateTotalCost`.
- CN: 这里声明或实现函数，例如 `runBlock`, `estimateTotalCost`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `runBlock`, `estimateTotalCost`。

### Lines 150-159

```cpp
  // Divide work into blocks of equal cost
  ThreadPoolInterface &Pool = getThreadPool();
  auto BlockBegin = BC.getBinaryFunctions().begin();
  unsigned CurrentCost = 0;

  for (auto It = BC.getBinaryFunctions().begin();
       It != BC.getBinaryFunctions().end(); ++It) {
    BinaryFunction &BF = It->second;
    CurrentCost += computeCostFor(BF, SkipPredicate, SchedPolicy);
```

- EN: Declares or implements routines including `getThreadPool`, `computeCostFor`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getThreadPool`, `computeCostFor`.
- CN: 这里声明或实现函数，例如 `getThreadPool`, `computeCostFor`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getThreadPool`, `computeCostFor`。

### Lines 160-169

```cpp
    if (CurrentCost >= BlockCost) {
      Pool.async(runBlock, BlockBegin, std::next(It));
      BlockBegin = std::next(It);
      CurrentCost = 0;
    }
  }
  Pool.async(runBlock, BlockBegin, BC.getBinaryFunctions().end());
  Pool.wait();
}
```

- EN: Declares or implements routines including `next`. Notable symbols here include `next`.
- CN: 这里声明或实现函数，例如 `next`。这里较值得关注的符号包括 `next`。

### Lines 170-187

```cpp
void runOnEachFunctionWithUniqueAllocId(
    BinaryContext &BC, SchedulingPolicy SchedPolicy,
    WorkFuncWithAllocTy WorkFunction, PredicateTy SkipPredicate,
    std::string LogName, bool ForceSequential, unsigned TasksPerThread) {
  if (BC.getBinaryFunctions().size() == 0)
    return;

  llvm::sys::RWMutex MainLock;
  auto runBlock = [&](std::map<uint64_t, BinaryFunction>::iterator BlockBegin,
                      std::map<uint64_t, BinaryFunction>::iterator BlockEnd,
                      MCPlusBuilder::AllocatorIdTy AllocId) {
    Timer T(LogName, LogName);
    LLVM_DEBUG(T.startTimer());
    std::shared_lock<llvm::sys::RWMutex> Lock(MainLock);
    for (auto It = BlockBegin; It != BlockEnd; ++It) {
      BinaryFunction &BF = It->second;
      if (SkipPredicate && SkipPredicate(BF))
        continue;
```

- EN: Declares or implements routines including `T`, `LLVM_DEBUG`, `Lock`. Notable symbols here include `T`, `LLVM_DEBUG`, `Lock`.
- CN: 这里声明或实现函数，例如 `T`, `LLVM_DEBUG`, `Lock`。这里较值得关注的符号包括 `T`, `LLVM_DEBUG`, `Lock`。

### Lines 188-203

```cpp

      WorkFunction(BF, AllocId);
    }
    LLVM_DEBUG(T.stopTimer());
  };

  unsigned AllocId = 1;
  auto EnsureAllocatorExists = [&BC](unsigned AllocId) {
    if (!BC.MIB->checkAllocatorExists(AllocId)) {
      MCPlusBuilder::AllocatorIdTy Id =
          BC.MIB->initializeNewAnnotationAllocator();
      (void)Id;
      assert(AllocId == Id && "unexpected allocator id created");
    }
  };
```

- EN: Declares or implements routines including `WorkFunction`, `LLVM_DEBUG`, `initializeNewAnnotationAllocator`, `assert`. Notable symbols here include `WorkFunction`, `LLVM_DEBUG`, `initializeNewAnnotationAllocator`, `assert`.
- CN: 这里声明或实现函数，例如 `WorkFunction`, `LLVM_DEBUG`, `initializeNewAnnotationAllocator`, `assert`。这里较值得关注的符号包括 `WorkFunction`, `LLVM_DEBUG`, `initializeNewAnnotationAllocator`, `assert`。

### Lines 204-212

```cpp
  if (opts::NoThreads || ForceSequential) {
    EnsureAllocatorExists(AllocId);
    runBlock(BC.getBinaryFunctions().begin(), BC.getBinaryFunctions().end(),
             AllocId);
    return;
  }
  // This lock is used to postpone task execution
  std::unique_lock<llvm::sys::RWMutex> Lock(MainLock);
```

- EN: Declares or implements routines including `EnsureAllocatorExists`, `runBlock`, `Lock`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `EnsureAllocatorExists`, `runBlock`, `Lock`.
- CN: 这里声明或实现函数，例如 `EnsureAllocatorExists`, `runBlock`, `Lock`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `EnsureAllocatorExists`, `runBlock`, `Lock`。

### Lines 213-227

```cpp
  // Estimate the overall runtime cost using the scheduling policy
  const unsigned TotalCost = estimateTotalCost(BC, SkipPredicate, SchedPolicy);
  const unsigned BlocksCount = TasksPerThread * opts::ThreadCount;
  const unsigned BlockCost =
      TotalCost > BlocksCount ? TotalCost / BlocksCount : 1;

  // Divide work into blocks of equal cost
  ThreadPoolInterface &Pool = getThreadPool();
  auto BlockBegin = BC.getBinaryFunctions().begin();
  unsigned CurrentCost = 0;
  for (auto It = BC.getBinaryFunctions().begin();
       It != BC.getBinaryFunctions().end(); ++It) {
    BinaryFunction &BF = It->second;
    CurrentCost += computeCostFor(BF, SkipPredicate, SchedPolicy);
```

- EN: Declares or implements routines including `estimateTotalCost`, `getThreadPool`, `computeCostFor`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `estimateTotalCost`, `getThreadPool`, `computeCostFor`.
- CN: 这里声明或实现函数，例如 `estimateTotalCost`, `getThreadPool`, `computeCostFor`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `estimateTotalCost`, `getThreadPool`, `computeCostFor`。

### Lines 228-236

```cpp
    if (CurrentCost >= BlockCost) {
      EnsureAllocatorExists(AllocId);
      Pool.async(runBlock, BlockBegin, std::next(It), AllocId);
      AllocId++;
      BlockBegin = std::next(It);
      CurrentCost = 0;
    }
  }
```

- EN: Declares or implements routines including `EnsureAllocatorExists`, `next`. Notable symbols here include `EnsureAllocatorExists`, `next`.
- CN: 这里声明或实现函数，例如 `EnsureAllocatorExists`, `next`。这里较值得关注的符号包括 `EnsureAllocatorExists`, `next`。

### Lines 237-246

```cpp
  EnsureAllocatorExists(AllocId);

  Pool.async(runBlock, BlockBegin, BC.getBinaryFunctions().end(), AllocId);
  Lock.unlock();
  Pool.wait();
}

} // namespace ParallelUtilities
} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `ParallelUtilities`, `bolt`, `llvm` to organize symbols. Declares or implements routines including `EnsureAllocatorExists`. Notable symbols here include `EnsureAllocatorExists`, `ParallelUtilities`, `bolt`, `llvm`.
- CN: 这里位于命名空间 `ParallelUtilities`, `bolt`, `llvm` 中，用于组织符号作用域。这里声明或实现函数，例如 `EnsureAllocatorExists`。这里较值得关注的符号包括 `EnsureAllocatorExists`, `ParallelUtilities`, `bolt`, `llvm`。

## Key Concepts / 关键概念

- `that`: class or struct interface / 类或结构体接口
- `desc`: function or method entry point / 函数或方法入口
- `init`: function or method entry point / 函数或方法入口
- `cat`: function or method entry point / 函数或方法入口
- `llvm_unreachable`: function or method entry point / 函数或方法入口
- `computeCostFor`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `opts`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Core/ParallelUtilities.h`, `bolt/Core/BinaryContext.h`, `bolt/Core/BinaryFunction.h`
- LLVM headers / LLVM 头文件: `llvm/Support/RWMutex.h`, `llvm/Support/ThreadPool.h`, `llvm/Support/Timer.h`
- System headers / 系统头文件: `mutex`
- Directory context / 目录上下文: `bolt/lib/Core` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Core` 下的相邻文件通常与本文件协作组成对应子系统
