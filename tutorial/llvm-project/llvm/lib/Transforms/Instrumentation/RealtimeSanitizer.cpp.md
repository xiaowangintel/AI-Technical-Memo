# RealtimeSanitizer.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/Transforms/Instrumentation/RealtimeSanitizer.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**: This file is a part of the RealtimeSanitizer, an LLVM transformation for detecting and reporting realtime safety violations. / 该文件位于 `Transforms/Instrumentation`，主要实现 `RealtimeSanitizer` 相关的 LLVM 变换逻辑、辅助流程以及 pass 接线代码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
//===- RealtimeSanitizer.cpp - RealtimeSanitizer instrumentation *- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file is a part of the RealtimeSanitizer, an LLVM transformation for
// detecting and reporting realtime safety violations.
//
// See also: llvm-project/compiler-rt/lib/rtsan/
//
//===----------------------------------------------------------------------===//

#include "llvm/IR/Analysis.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/InstIterator.h"
#include "llvm/IR/Module.h"
#include "llvm/Transforms/Utils/ModuleUtils.h"
```

- **L1**: Banner comment marking a file section boundary. / 横幅注释，用于标记文件分节。
- **L2**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L3**: Comment documents the nearby logic or transformation intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑或变换意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment documents the nearby logic or transformation intent: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑或变换意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment documents the nearby logic or transformation intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑或变换意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L7**: Banner comment marking a file section boundary. / 横幅注释，用于标记文件分节。
- **L8**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L9**: Comment documents the nearby logic or transformation intent: `This file is a part of the RealtimeSanitizer, an LLVM transformation for`. / 注释说明了附近代码的逻辑或变换意图：`This file is a part of the RealtimeSanitizer, an LLVM transformation for`。
- **L10**: Comment documents the nearby logic or transformation intent: `detecting and reporting realtime safety violations.`. / 注释说明了附近代码的逻辑或变换意图：`detecting and reporting realtime safety violations.`。
- **L11**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L12**: Comment documents the nearby logic or transformation intent: `See also: llvm-project/compiler-rt/lib/rtsan/`. / 注释说明了附近代码的逻辑或变换意图：`See also: llvm-project/compiler-rt/lib/rtsan/`。
- **L13**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L14**: Banner comment marking a file section boundary. / 横幅注释，用于标记文件分节。
- **L15**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L16**: Includes "llvm/IR/Analysis.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/Analysis.h" 以使用LLVM IR 核心类型与构造工具。
- **L17**: Includes "llvm/IR/IRBuilder.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/IRBuilder.h" 以使用LLVM IR 核心类型与构造工具。
- **L18**: Includes "llvm/IR/InstIterator.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/InstIterator.h" 以使用LLVM IR 核心类型与构造工具。
- **L19**: Includes "llvm/IR/Module.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/Module.h" 以使用LLVM IR 核心类型与构造工具。
- **L20**: Includes "llvm/Transforms/Utils/ModuleUtils.h" to access shared transform utilities. / 引入 "llvm/Transforms/Utils/ModuleUtils.h" 以使用共享的变换辅助工具。

### Lines 21-40

```cpp

#include "llvm/Demangle/Demangle.h"
#include "llvm/Transforms/Instrumentation/RealtimeSanitizer.h"

using namespace llvm;

const char kRtsanModuleCtorName[] = "rtsan.module_ctor";
const char kRtsanInitName[] = "__rtsan_ensure_initialized";

static SmallVector<Type *> getArgTypes(ArrayRef<Value *> FunctionArgs) {
  SmallVector<Type *> Types;
  for (Value *Arg : FunctionArgs)
    Types.push_back(Arg->getType());
  return Types;
}

static void insertCallBeforeInstruction(Function &Fn, Instruction &Instruction,
                                        const char *FunctionName,
                                        ArrayRef<Value *> FunctionArgs) {
  LLVMContext &Context = Fn.getContext();
```

- **L21**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L22**: Includes "llvm/Demangle/Demangle.h" to access local declarations used by this file. / 引入 "llvm/Demangle/Demangle.h" 以使用本文件使用的本地声明。
- **L23**: Includes "llvm/Transforms/Instrumentation/RealtimeSanitizer.h" to access transform-specific declarations. / 引入 "llvm/Transforms/Instrumentation/RealtimeSanitizer.h" 以使用变换相关声明。
- **L24**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L25**: Brings namespace `llvm` into the local scope. / 将命名空间 `llvm` 引入当前作用域。
- **L26**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L27**: Executes a standalone statement or declaration: `const char kRtsanModuleCtorName[] = "rtsan.module_ctor";`. / 执行一条独立语句或声明：`const char kRtsanModuleCtorName[] = "rtsan.module_ctor";`。
- **L28**: Executes a standalone statement or declaration: `const char kRtsanInitName[] = "__rtsan_ensure_initialized";`. / 执行一条独立语句或声明：`const char kRtsanInitName[] = "__rtsan_ensure_initialized";`。
- **L29**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L30**: Starts a function, method, or lambda body: `static SmallVector<Type *> getArgTypes(ArrayRef<Value *> FunctionArgs) {`. / 开始一个函数、方法或 lambda 的主体：`static SmallVector<Type *> getArgTypes(ArrayRef<Value *> FunctionArgs) {`。
- **L31**: Executes a standalone statement or declaration: `SmallVector<Type *> Types;`. / 执行一条独立语句或声明：`SmallVector<Type *> Types;`。
- **L32**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L33**: Executes call or statement centered on `Types.push_back`. / 执行以 `Types.push_back` 为核心的调用或语句。
- **L34**: Returns from the current function with `Types`. / 以 `Types` 从当前函数返回。
- **L35**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L36**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L37**: Continues a multi-line argument list or initializer: `static void insertCallBeforeInstruction(Function &Fn, Instruction &Instruction,`. / 继续一个多行参数列表或初始化器：`static void insertCallBeforeInstruction(Function &Fn, Instruction &Instruction,`。
- **L38**: Continues a multi-line argument list or initializer: `const char *FunctionName,`. / 继续一个多行参数列表或初始化器：`const char *FunctionName,`。
- **L39**: Continues the surrounding expression or declaration: `ArrayRef<Value *> FunctionArgs) {`. / 继续构造周围的表达式或声明：`ArrayRef<Value *> FunctionArgs) {`。
- **L40**: Executes call or statement centered on `Fn.getContext`. / 执行以 `Fn.getContext` 为核心的调用或语句。

### Lines 41-60

```cpp
  FunctionType *FuncType = FunctionType::get(Type::getVoidTy(Context),
                                             getArgTypes(FunctionArgs), false);
  FunctionCallee Func =
      Fn.getParent()->getOrInsertFunction(FunctionName, FuncType);
  IRBuilder<> Builder{&Instruction};
  Builder.CreateCall(Func, FunctionArgs);
}

static void insertCallAtFunctionEntryPoint(Function &Fn,
                                           const char *InsertFnName,
                                           ArrayRef<Value *> FunctionArgs) {
  insertCallBeforeInstruction(Fn, Fn.front().front(), InsertFnName,
                              FunctionArgs);
}

static void insertCallAtAllFunctionExitPoints(Function &Fn,
                                              const char *InsertFnName,
                                              ArrayRef<Value *> FunctionArgs) {
  for (auto &I : instructions(Fn))
    if (isa<ReturnInst>(&I))
```

- **L41**: Continues a multi-line argument list or initializer: `FunctionType *FuncType = FunctionType::get(Type::getVoidTy(Context),`. / 继续一个多行参数列表或初始化器：`FunctionType *FuncType = FunctionType::get(Type::getVoidTy(Context),`。
- **L42**: Executes call or statement centered on `getArgTypes`. / 执行以 `getArgTypes` 为核心的调用或语句。
- **L43**: Continues the surrounding expression or declaration: `FunctionCallee Func =`. / 继续构造周围的表达式或声明：`FunctionCallee Func =`。
- **L44**: Executes call or statement centered on `Fn.getParent`. / 执行以 `Fn.getParent` 为核心的调用或语句。
- **L45**: Executes a standalone statement or declaration: `IRBuilder<> Builder{&Instruction};`. / 执行一条独立语句或声明：`IRBuilder<> Builder{&Instruction};`。
- **L46**: Executes call or statement centered on `Builder.CreateCall`. / 执行以 `Builder.CreateCall` 为核心的调用或语句。
- **L47**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L48**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L49**: Continues a multi-line argument list or initializer: `static void insertCallAtFunctionEntryPoint(Function &Fn,`. / 继续一个多行参数列表或初始化器：`static void insertCallAtFunctionEntryPoint(Function &Fn,`。
- **L50**: Continues a multi-line argument list or initializer: `const char *InsertFnName,`. / 继续一个多行参数列表或初始化器：`const char *InsertFnName,`。
- **L51**: Continues the surrounding expression or declaration: `ArrayRef<Value *> FunctionArgs) {`. / 继续构造周围的表达式或声明：`ArrayRef<Value *> FunctionArgs) {`。
- **L52**: Continues a multi-line argument list or initializer: `insertCallBeforeInstruction(Fn, Fn.front().front(), InsertFnName,`. / 继续一个多行参数列表或初始化器：`insertCallBeforeInstruction(Fn, Fn.front().front(), InsertFnName,`。
- **L53**: Executes a standalone statement or declaration: `FunctionArgs);`. / 执行一条独立语句或声明：`FunctionArgs);`。
- **L54**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L55**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L56**: Continues a multi-line argument list or initializer: `static void insertCallAtAllFunctionExitPoints(Function &Fn,`. / 继续一个多行参数列表或初始化器：`static void insertCallAtAllFunctionExitPoints(Function &Fn,`。
- **L57**: Continues a multi-line argument list or initializer: `const char *InsertFnName,`. / 继续一个多行参数列表或初始化器：`const char *InsertFnName,`。
- **L58**: Continues the surrounding expression or declaration: `ArrayRef<Value *> FunctionArgs) {`. / 继续构造周围的表达式或声明：`ArrayRef<Value *> FunctionArgs) {`。
- **L59**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L60**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。

### Lines 61-80

```cpp
      insertCallBeforeInstruction(Fn, I, InsertFnName, FunctionArgs);
}

static PreservedAnalyses rtsanPreservedCFGAnalyses() {
  PreservedAnalyses PA;
  PA.preserveSet<CFGAnalyses>();
  return PA;
}

static PreservedAnalyses runSanitizeRealtime(Function &Fn) {
  insertCallAtFunctionEntryPoint(Fn, "__rtsan_realtime_enter", {});
  insertCallAtAllFunctionExitPoints(Fn, "__rtsan_realtime_exit", {});
  return rtsanPreservedCFGAnalyses();
}

static PreservedAnalyses runSanitizeRealtimeBlocking(Function &Fn) {
  IRBuilder<> Builder(&Fn.front().front());
  Value *Name = Builder.CreateGlobalString(demangle(Fn.getName()));
  insertCallAtFunctionEntryPoint(Fn, "__rtsan_notify_blocking_call", {Name});
  return rtsanPreservedCFGAnalyses();
```

- **L61**: Executes call or statement centered on `insertCallBeforeInstruction`. / 执行以 `insertCallBeforeInstruction` 为核心的调用或语句。
- **L62**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L63**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L64**: Starts a function, method, or lambda body: `static PreservedAnalyses rtsanPreservedCFGAnalyses() {`. / 开始一个函数、方法或 lambda 的主体：`static PreservedAnalyses rtsanPreservedCFGAnalyses() {`。
- **L65**: Executes a standalone statement or declaration: `PreservedAnalyses PA;`. / 执行一条独立语句或声明：`PreservedAnalyses PA;`。
- **L66**: Executes call or statement centered on `PA.preserveSet<CFGAnalyses>`. / 执行以 `PA.preserveSet<CFGAnalyses>` 为核心的调用或语句。
- **L67**: Returns from the current function with `PA`. / 以 `PA` 从当前函数返回。
- **L68**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L69**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L70**: Starts a function, method, or lambda body: `static PreservedAnalyses runSanitizeRealtime(Function &Fn) {`. / 开始一个函数、方法或 lambda 的主体：`static PreservedAnalyses runSanitizeRealtime(Function &Fn) {`。
- **L71**: Executes call or statement centered on `insertCallAtFunctionEntryPoint`. / 执行以 `insertCallAtFunctionEntryPoint` 为核心的调用或语句。
- **L72**: Executes call or statement centered on `insertCallAtAllFunctionExitPoints`. / 执行以 `insertCallAtAllFunctionExitPoints` 为核心的调用或语句。
- **L73**: Returns from the current function with `rtsanPreservedCFGAnalyses()`. / 以 `rtsanPreservedCFGAnalyses()` 从当前函数返回。
- **L74**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L75**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L76**: Starts a function, method, or lambda body: `static PreservedAnalyses runSanitizeRealtimeBlocking(Function &Fn) {`. / 开始一个函数、方法或 lambda 的主体：`static PreservedAnalyses runSanitizeRealtimeBlocking(Function &Fn) {`。
- **L77**: Executes call or statement centered on `Builder`. / 执行以 `Builder` 为核心的调用或语句。
- **L78**: Executes call or statement centered on `Builder.CreateGlobalString`. / 执行以 `Builder.CreateGlobalString` 为核心的调用或语句。
- **L79**: Executes call or statement centered on `insertCallAtFunctionEntryPoint`. / 执行以 `insertCallAtFunctionEntryPoint` 为核心的调用或语句。
- **L80**: Returns from the current function with `rtsanPreservedCFGAnalyses()`. / 以 `rtsanPreservedCFGAnalyses()` 从当前函数返回。

### Lines 81-100

```cpp
}

PreservedAnalyses RealtimeSanitizerPass::run(Module &M,
                                             ModuleAnalysisManager &MAM) {
  getOrCreateSanitizerCtorAndInitFunctions(
      M, kRtsanModuleCtorName, kRtsanInitName, /*InitArgTypes=*/{},
      /*InitArgs=*/{},
      // This callback is invoked when the functions are created the first
      // time. Hook them into the global ctors list in that case:
      [&](Function *Ctor, FunctionCallee) { appendToGlobalCtors(M, Ctor, 0); });

  for (Function &F : M) {
    if (F.empty())
      continue;

    if (F.hasFnAttribute(Attribute::SanitizeRealtime))
      runSanitizeRealtime(F);

    if (F.hasFnAttribute(Attribute::SanitizeRealtimeBlocking))
      runSanitizeRealtimeBlocking(F);
```

- **L81**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L82**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L83**: Continues a multi-line argument list or initializer: `PreservedAnalyses RealtimeSanitizerPass::run(Module &M,`. / 继续一个多行参数列表或初始化器：`PreservedAnalyses RealtimeSanitizerPass::run(Module &M,`。
- **L84**: Continues the surrounding expression or declaration: `ModuleAnalysisManager &MAM) {`. / 继续构造周围的表达式或声明：`ModuleAnalysisManager &MAM) {`。
- **L85**: Continues the surrounding expression or declaration: `getOrCreateSanitizerCtorAndInitFunctions(`. / 继续构造周围的表达式或声明：`getOrCreateSanitizerCtorAndInitFunctions(`。
- **L86**: Continues a multi-line argument list or initializer: `M, kRtsanModuleCtorName, kRtsanInitName, /*InitArgTypes=*/{},`. / 继续一个多行参数列表或初始化器：`M, kRtsanModuleCtorName, kRtsanInitName, /*InitArgTypes=*/{},`。
- **L87**: Comment documents the nearby logic or transformation intent: `InitArgs=*/{},`. / 注释说明了附近代码的逻辑或变换意图：`InitArgs=*/{},`。
- **L88**: Comment documents the nearby logic or transformation intent: `This callback is invoked when the functions are created the first`. / 注释说明了附近代码的逻辑或变换意图：`This callback is invoked when the functions are created the first`。
- **L89**: Comment documents the nearby logic or transformation intent: `time. Hook them into the global ctors list in that case:`. / 注释说明了附近代码的逻辑或变换意图：`time. Hook them into the global ctors list in that case:`。
- **L90**: Executes call or statement centered on `[&]`. / 执行以 `[&]` 为核心的调用或语句。
- **L91**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L92**: Begins a `for` control-flow statement and evaluates its condition. / 开始 `for` 控制流语句并计算其条件。
- **L93**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L94**: Skips to the next loop iteration. / 跳到下一次循环迭代。
- **L95**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L96**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L97**: Executes call or statement centered on `runSanitizeRealtime`. / 执行以 `runSanitizeRealtime` 为核心的调用或语句。
- **L98**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L99**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L100**: Executes call or statement centered on `runSanitizeRealtimeBlocking`. / 执行以 `runSanitizeRealtimeBlocking` 为核心的调用或语句。

### Lines 101-104

```cpp
  }

  return PreservedAnalyses::none();
}
```

- **L101**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L102**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L103**: Returns from the current function with `PreservedAnalyses::none()`. / 以 `PreservedAnalyses::none()` 从当前函数返回。
- **L104**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **Instrumentation transform pipeline / Instrumentation 变换流水线**
- **IR construction and rewriting / IR 构造与重写**
- **Function-level traversal and updates / 函数级遍历与更新**
- **Module-wide coordination / 模块范围的协调**
- **Analysis preservation reporting / 分析保持情况报告**

## Dependencies / 依赖关系

- `llvm/IR/Analysis.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/IRBuilder.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/InstIterator.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/IR/Module.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
- `llvm/Transforms/Utils/ModuleUtils.h`: Provides shared transform utilities. / 提供共享的变换辅助工具。
- `llvm/Demangle/Demangle.h`: Provides local declarations used by this file. / 提供本文件使用的本地声明。
- `llvm/Transforms/Instrumentation/RealtimeSanitizer.h`: Provides transform-specific declarations. / 提供变换相关声明。
