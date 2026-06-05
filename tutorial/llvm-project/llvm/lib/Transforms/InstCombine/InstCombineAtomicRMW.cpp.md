# InstCombineAtomicRMW.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/Transforms/InstCombine/InstCombineAtomicRMW.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**: This file implements the visit functions for atomic rmw instructions. / 该文件位于 `Transforms/InstCombine`，主要实现 `InstCombineAtomicRMW` 相关的 LLVM 变换逻辑、辅助流程以及 pass 接线代码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
//===- InstCombineAtomicRMW.cpp -------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the visit functions for atomic rmw instructions.
//
//===----------------------------------------------------------------------===//

#include "InstCombineInternal.h"
#include "llvm/IR/Instructions.h"

using namespace llvm;
using namespace PatternMatch;

/// Return true if and only if the given instruction does not modify the memory
/// location referenced.  Note that an idemptent atomicrmw may still have
```

- **L1**: Banner comment marking a file section boundary. / 横幅注释，用于标记文件分节。
- **L2**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L3**: Comment documents the nearby logic or transformation intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑或变换意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment documents the nearby logic or transformation intent: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑或变换意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment documents the nearby logic or transformation intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑或变换意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L7**: Banner comment marking a file section boundary. / 横幅注释，用于标记文件分节。
- **L8**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L9**: Comment documents the nearby logic or transformation intent: `This file implements the visit functions for atomic rmw instructions.`. / 注释说明了附近代码的逻辑或变换意图：`This file implements the visit functions for atomic rmw instructions.`。
- **L10**: Separator comment used to visually break up sections. / 分隔性注释，用于在视觉上划分小节。
- **L11**: Banner comment marking a file section boundary. / 横幅注释，用于标记文件分节。
- **L12**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L13**: Includes "InstCombineInternal.h" to access local declarations used by this file. / 引入 "InstCombineInternal.h" 以使用本文件使用的本地声明。
- **L14**: Includes "llvm/IR/Instructions.h" to access LLVM IR core types and builders. / 引入 "llvm/IR/Instructions.h" 以使用LLVM IR 核心类型与构造工具。
- **L15**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L16**: Brings namespace `llvm` into the local scope. / 将命名空间 `llvm` 引入当前作用域。
- **L17**: Brings namespace `PatternMatch` into the local scope. / 将命名空间 `PatternMatch` 引入当前作用域。
- **L18**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L19**: Comment documents the nearby logic or transformation intent: `Return true if and only if the given instruction does not modify the memory`. / 注释说明了附近代码的逻辑或变换意图：`Return true if and only if the given instruction does not modify the memory`。
- **L20**: Comment documents the nearby logic or transformation intent: `location referenced.  Note that an idemptent atomicrmw may still have`. / 注释说明了附近代码的逻辑或变换意图：`location referenced.  Note that an idemptent atomicrmw may still have`。

### Lines 21-40

```cpp
/// ordering effects on nearby instructions, or be volatile.
/// TODO: Common w/ the version in AtomicExpandPass, and change the term used.
/// Idemptotent is confusing in this context.
static bool isIdempotentRMW(AtomicRMWInst &RMWI) {
  if (auto CF = dyn_cast<ConstantFP>(RMWI.getValOperand()))
    switch(RMWI.getOperation()) {
    case AtomicRMWInst::FAdd: // -0.0
      return CF->isZero() && CF->isNegative();
    case AtomicRMWInst::FSub: // +0.0
      return CF->isZero() && !CF->isNegative();
    default:
      return false;
    };

  auto C = dyn_cast<ConstantInt>(RMWI.getValOperand());
  if(!C)
    return false;

  switch(RMWI.getOperation()) {
    case AtomicRMWInst::Add:
```

- **L21**: Comment documents the nearby logic or transformation intent: `ordering effects on nearby instructions, or be volatile.`. / 注释说明了附近代码的逻辑或变换意图：`ordering effects on nearby instructions, or be volatile.`。
- **L22**: Comment records a pending task or caution: `TODO: Common w/ the version in AtomicExpandPass, and change the term used.`. / 注释记录了待办事项或注意点：`TODO: Common w/ the version in AtomicExpandPass, and change the term used.`。
- **L23**: Comment documents the nearby logic or transformation intent: `Idemptotent is confusing in this context.`. / 注释说明了附近代码的逻辑或变换意图：`Idemptotent is confusing in this context.`。
- **L24**: Starts a function, method, or lambda body: `static bool isIdempotentRMW(AtomicRMWInst &RMWI) {`. / 开始一个函数、方法或 lambda 的主体：`static bool isIdempotentRMW(AtomicRMWInst &RMWI) {`。
- **L25**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L26**: Begins a `switch` control-flow statement and evaluates its condition. / 开始 `switch` 控制流语句并计算其条件。
- **L27**: Introduces a switch dispatch label: `case AtomicRMWInst::FAdd: // -0.0`. / 引入一个 switch 分发标签：`case AtomicRMWInst::FAdd: // -0.0`。
- **L28**: Returns from the current function with `CF->isZero() && CF->isNegative()`. / 以 `CF->isZero() && CF->isNegative()` 从当前函数返回。
- **L29**: Introduces a switch dispatch label: `case AtomicRMWInst::FSub: // +0.0`. / 引入一个 switch 分发标签：`case AtomicRMWInst::FSub: // +0.0`。
- **L30**: Returns from the current function with `CF->isZero() && !CF->isNegative()`. / 以 `CF->isZero() && !CF->isNegative()` 从当前函数返回。
- **L31**: Introduces a switch dispatch label: `default:`. / 引入一个 switch 分发标签：`default:`。
- **L32**: Returns from the current function with `false`. / 以 `false` 从当前函数返回。
- **L33**: Closes the current declaration scope, such as a class or struct. / 结束当前声明作用域，例如类或结构体。
- **L34**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L35**: Initializes variable `C` from the right-hand expression. / 使用右侧表达式初始化变量 `C`。
- **L36**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L37**: Returns from the current function with `false`. / 以 `false` 从当前函数返回。
- **L38**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L39**: Begins a `switch` control-flow statement and evaluates its condition. / 开始 `switch` 控制流语句并计算其条件。
- **L40**: Introduces a switch dispatch label: `case AtomicRMWInst::Add:`. / 引入一个 switch 分发标签：`case AtomicRMWInst::Add:`。

### Lines 41-60

```cpp
    case AtomicRMWInst::Sub:
    case AtomicRMWInst::Or:
    case AtomicRMWInst::Xor:
      return C->isZero();
    case AtomicRMWInst::And:
      return C->isMinusOne();
    case AtomicRMWInst::Min:
      return C->isMaxValue(true);
    case AtomicRMWInst::Max:
      return C->isMinValue(true);
    case AtomicRMWInst::UMin:
      return C->isMaxValue(false);
    case AtomicRMWInst::UMax:
      return C->isMinValue(false);
    default:
      return false;
  }
}

/// Return true if the given instruction always produces a value in memory
```

- **L41**: Introduces a switch dispatch label: `case AtomicRMWInst::Sub:`. / 引入一个 switch 分发标签：`case AtomicRMWInst::Sub:`。
- **L42**: Introduces a switch dispatch label: `case AtomicRMWInst::Or:`. / 引入一个 switch 分发标签：`case AtomicRMWInst::Or:`。
- **L43**: Introduces a switch dispatch label: `case AtomicRMWInst::Xor:`. / 引入一个 switch 分发标签：`case AtomicRMWInst::Xor:`。
- **L44**: Returns from the current function with `C->isZero()`. / 以 `C->isZero()` 从当前函数返回。
- **L45**: Introduces a switch dispatch label: `case AtomicRMWInst::And:`. / 引入一个 switch 分发标签：`case AtomicRMWInst::And:`。
- **L46**: Returns from the current function with `C->isMinusOne()`. / 以 `C->isMinusOne()` 从当前函数返回。
- **L47**: Introduces a switch dispatch label: `case AtomicRMWInst::Min:`. / 引入一个 switch 分发标签：`case AtomicRMWInst::Min:`。
- **L48**: Returns from the current function with `C->isMaxValue(true)`. / 以 `C->isMaxValue(true)` 从当前函数返回。
- **L49**: Introduces a switch dispatch label: `case AtomicRMWInst::Max:`. / 引入一个 switch 分发标签：`case AtomicRMWInst::Max:`。
- **L50**: Returns from the current function with `C->isMinValue(true)`. / 以 `C->isMinValue(true)` 从当前函数返回。
- **L51**: Introduces a switch dispatch label: `case AtomicRMWInst::UMin:`. / 引入一个 switch 分发标签：`case AtomicRMWInst::UMin:`。
- **L52**: Returns from the current function with `C->isMaxValue(false)`. / 以 `C->isMaxValue(false)` 从当前函数返回。
- **L53**: Introduces a switch dispatch label: `case AtomicRMWInst::UMax:`. / 引入一个 switch 分发标签：`case AtomicRMWInst::UMax:`。
- **L54**: Returns from the current function with `C->isMinValue(false)`. / 以 `C->isMinValue(false)` 从当前函数返回。
- **L55**: Introduces a switch dispatch label: `default:`. / 引入一个 switch 分发标签：`default:`。
- **L56**: Returns from the current function with `false`. / 以 `false` 从当前函数返回。
- **L57**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L58**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L59**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L60**: Comment documents the nearby logic or transformation intent: `Return true if the given instruction always produces a value in memory`. / 注释说明了附近代码的逻辑或变换意图：`Return true if the given instruction always produces a value in memory`。

### Lines 61-80

```cpp
/// equivalent to its value operand.
static bool isSaturating(AtomicRMWInst &RMWI) {
  if (auto CF = dyn_cast<ConstantFP>(RMWI.getValOperand()))
    switch (RMWI.getOperation()) {
    case AtomicRMWInst::FMax:
      // maxnum(x, +inf) -> +inf
      return !CF->isNegative() && CF->isInfinity();
    case AtomicRMWInst::FMin:
      // minnum(x, -inf) -> +inf
      return CF->isNegative() && CF->isInfinity();
    case AtomicRMWInst::FAdd:
    case AtomicRMWInst::FSub:
      return CF->isNaN();
    default:
      return false;
    };

  auto C = dyn_cast<ConstantInt>(RMWI.getValOperand());
  if(!C)
    return false;
```

- **L61**: Comment documents the nearby logic or transformation intent: `equivalent to its value operand.`. / 注释说明了附近代码的逻辑或变换意图：`equivalent to its value operand.`。
- **L62**: Starts a function, method, or lambda body: `static bool isSaturating(AtomicRMWInst &RMWI) {`. / 开始一个函数、方法或 lambda 的主体：`static bool isSaturating(AtomicRMWInst &RMWI) {`。
- **L63**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L64**: Begins a `switch` control-flow statement and evaluates its condition. / 开始 `switch` 控制流语句并计算其条件。
- **L65**: Introduces a switch dispatch label: `case AtomicRMWInst::FMax:`. / 引入一个 switch 分发标签：`case AtomicRMWInst::FMax:`。
- **L66**: Comment documents the nearby logic or transformation intent: `maxnum(x, +inf) -> +inf`. / 注释说明了附近代码的逻辑或变换意图：`maxnum(x, +inf) -> +inf`。
- **L67**: Returns from the current function with `!CF->isNegative() && CF->isInfinity()`. / 以 `!CF->isNegative() && CF->isInfinity()` 从当前函数返回。
- **L68**: Introduces a switch dispatch label: `case AtomicRMWInst::FMin:`. / 引入一个 switch 分发标签：`case AtomicRMWInst::FMin:`。
- **L69**: Comment documents the nearby logic or transformation intent: `minnum(x, -inf) -> +inf`. / 注释说明了附近代码的逻辑或变换意图：`minnum(x, -inf) -> +inf`。
- **L70**: Returns from the current function with `CF->isNegative() && CF->isInfinity()`. / 以 `CF->isNegative() && CF->isInfinity()` 从当前函数返回。
- **L71**: Introduces a switch dispatch label: `case AtomicRMWInst::FAdd:`. / 引入一个 switch 分发标签：`case AtomicRMWInst::FAdd:`。
- **L72**: Introduces a switch dispatch label: `case AtomicRMWInst::FSub:`. / 引入一个 switch 分发标签：`case AtomicRMWInst::FSub:`。
- **L73**: Returns from the current function with `CF->isNaN()`. / 以 `CF->isNaN()` 从当前函数返回。
- **L74**: Introduces a switch dispatch label: `default:`. / 引入一个 switch 分发标签：`default:`。
- **L75**: Returns from the current function with `false`. / 以 `false` 从当前函数返回。
- **L76**: Closes the current declaration scope, such as a class or struct. / 结束当前声明作用域，例如类或结构体。
- **L77**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L78**: Initializes variable `C` from the right-hand expression. / 使用右侧表达式初始化变量 `C`。
- **L79**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L80**: Returns from the current function with `false`. / 以 `false` 从当前函数返回。

### Lines 81-100

```cpp

  switch(RMWI.getOperation()) {
  default:
    return false;
  case AtomicRMWInst::Xchg:
    return true;
  case AtomicRMWInst::Or:
    return C->isAllOnesValue();
  case AtomicRMWInst::And:
    return C->isZero();
  case AtomicRMWInst::Min:
    return C->isMinValue(true);
  case AtomicRMWInst::Max:
    return C->isMaxValue(true);
  case AtomicRMWInst::UMin:
    return C->isMinValue(false);
  case AtomicRMWInst::UMax:
    return C->isMaxValue(false);
  };
}
```

- **L81**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L82**: Begins a `switch` control-flow statement and evaluates its condition. / 开始 `switch` 控制流语句并计算其条件。
- **L83**: Introduces a switch dispatch label: `default:`. / 引入一个 switch 分发标签：`default:`。
- **L84**: Returns from the current function with `false`. / 以 `false` 从当前函数返回。
- **L85**: Introduces a switch dispatch label: `case AtomicRMWInst::Xchg:`. / 引入一个 switch 分发标签：`case AtomicRMWInst::Xchg:`。
- **L86**: Returns from the current function with `true`. / 以 `true` 从当前函数返回。
- **L87**: Introduces a switch dispatch label: `case AtomicRMWInst::Or:`. / 引入一个 switch 分发标签：`case AtomicRMWInst::Or:`。
- **L88**: Returns from the current function with `C->isAllOnesValue()`. / 以 `C->isAllOnesValue()` 从当前函数返回。
- **L89**: Introduces a switch dispatch label: `case AtomicRMWInst::And:`. / 引入一个 switch 分发标签：`case AtomicRMWInst::And:`。
- **L90**: Returns from the current function with `C->isZero()`. / 以 `C->isZero()` 从当前函数返回。
- **L91**: Introduces a switch dispatch label: `case AtomicRMWInst::Min:`. / 引入一个 switch 分发标签：`case AtomicRMWInst::Min:`。
- **L92**: Returns from the current function with `C->isMinValue(true)`. / 以 `C->isMinValue(true)` 从当前函数返回。
- **L93**: Introduces a switch dispatch label: `case AtomicRMWInst::Max:`. / 引入一个 switch 分发标签：`case AtomicRMWInst::Max:`。
- **L94**: Returns from the current function with `C->isMaxValue(true)`. / 以 `C->isMaxValue(true)` 从当前函数返回。
- **L95**: Introduces a switch dispatch label: `case AtomicRMWInst::UMin:`. / 引入一个 switch 分发标签：`case AtomicRMWInst::UMin:`。
- **L96**: Returns from the current function with `C->isMinValue(false)`. / 以 `C->isMinValue(false)` 从当前函数返回。
- **L97**: Introduces a switch dispatch label: `case AtomicRMWInst::UMax:`. / 引入一个 switch 分发标签：`case AtomicRMWInst::UMax:`。
- **L98**: Returns from the current function with `C->isMaxValue(false)`. / 以 `C->isMaxValue(false)` 从当前函数返回。
- **L99**: Closes the current declaration scope, such as a class or struct. / 结束当前声明作用域，例如类或结构体。
- **L100**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。

### Lines 101-120

```cpp

Instruction *InstCombinerImpl::visitAtomicRMWInst(AtomicRMWInst &RMWI) {

  // Volatile RMWs perform a load and a store, we cannot replace this by just a
  // load or just a store. We chose not to canonicalize out of general paranoia
  // about user expectations around volatile.
  if (RMWI.isVolatile())
    return nullptr;

  // Any atomicrmw op which produces a known result in memory can be
  // replaced w/an atomicrmw xchg.
  if (isSaturating(RMWI) &&
      RMWI.getOperation() != AtomicRMWInst::Xchg) {
    RMWI.setOperation(AtomicRMWInst::Xchg);
    return &RMWI;
  }

  assert(RMWI.getOrdering() != AtomicOrdering::NotAtomic &&
         RMWI.getOrdering() != AtomicOrdering::Unordered &&
         "AtomicRMWs don't make sense with Unordered or NotAtomic");
```

- **L101**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L102**: Starts a function, method, or lambda body: `Instruction *InstCombinerImpl::visitAtomicRMWInst(AtomicRMWInst &RMWI) {`. / 开始一个函数、方法或 lambda 的主体：`Instruction *InstCombinerImpl::visitAtomicRMWInst(AtomicRMWInst &RMWI) {`。
- **L103**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L104**: Comment documents the nearby logic or transformation intent: `Volatile RMWs perform a load and a store, we cannot replace this by just a`. / 注释说明了附近代码的逻辑或变换意图：`Volatile RMWs perform a load and a store, we cannot replace this by just a`。
- **L105**: Comment documents the nearby logic or transformation intent: `load or just a store. We chose not to canonicalize out of general paranoia`. / 注释说明了附近代码的逻辑或变换意图：`load or just a store. We chose not to canonicalize out of general paranoia`。
- **L106**: Comment documents the nearby logic or transformation intent: `about user expectations around volatile.`. / 注释说明了附近代码的逻辑或变换意图：`about user expectations around volatile.`。
- **L107**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L108**: Returns from the current function with `nullptr`. / 以 `nullptr` 从当前函数返回。
- **L109**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L110**: Comment documents the nearby logic or transformation intent: `Any atomicrmw op which produces a known result in memory can be`. / 注释说明了附近代码的逻辑或变换意图：`Any atomicrmw op which produces a known result in memory can be`。
- **L111**: Comment documents the nearby logic or transformation intent: `replaced w/an atomicrmw xchg.`. / 注释说明了附近代码的逻辑或变换意图：`replaced w/an atomicrmw xchg.`。
- **L112**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L113**: Starts a function, method, or lambda body: `RMWI.getOperation() != AtomicRMWInst::Xchg) {`. / 开始一个函数、方法或 lambda 的主体：`RMWI.getOperation() != AtomicRMWInst::Xchg) {`。
- **L114**: Executes call or statement centered on `RMWI.setOperation`. / 执行以 `RMWI.setOperation` 为核心的调用或语句。
- **L115**: Returns from the current function with `&RMWI`. / 以 `&RMWI` 从当前函数返回。
- **L116**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L117**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L118**: Checks an internal invariant in debug builds. / 在调试构建中检查内部不变式。
- **L119**: Continues the surrounding expression or declaration: `RMWI.getOrdering() != AtomicOrdering::Unordered &&`. / 继续构造周围的表达式或声明：`RMWI.getOrdering() != AtomicOrdering::Unordered &&`。
- **L120**: Executes a standalone statement or declaration: `"AtomicRMWs don't make sense with Unordered or NotAtomic");`. / 执行一条独立语句或声明：`"AtomicRMWs don't make sense with Unordered or NotAtomic");`。

### Lines 121-140

```cpp

  // Canonicalize atomicrmw add(ptr, neg(X)) -> atomicrmw sub(ptr, X)
  //              atomicrmw sub(ptr, neg(X)) -> atomicrmw add(ptr, X)
  // old + (-X) == old - X and old - (-X) == old + X; the returned old value
  // is identical in both cases. We match strictly on `sub 0, X` (negation) to
  // avoid infinite loops: a general negation of `sub A, B` yields `sub B, A`,
  // which would infinitely be negated back on the next iteration.
  auto Op = RMWI.getOperation();
  if (Op == AtomicRMWInst::Add || Op == AtomicRMWInst::Sub) {
    Value *Val = RMWI.getValOperand();
    Value *X;
    if (match(Val, m_Neg(m_Value(X)))) {
      RMWI.setOperation(Op == AtomicRMWInst::Add ? AtomicRMWInst::Sub
                                                 : AtomicRMWInst::Add);
      return replaceOperand(RMWI, 1, X);
    }
  }

  if (!isIdempotentRMW(RMWI))
    return nullptr;
```

- **L121**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L122**: Comment documents the nearby logic or transformation intent: `Canonicalize atomicrmw add(ptr, neg(X)) -> atomicrmw sub(ptr, X)`. / 注释说明了附近代码的逻辑或变换意图：`Canonicalize atomicrmw add(ptr, neg(X)) -> atomicrmw sub(ptr, X)`。
- **L123**: Comment documents the nearby logic or transformation intent: `atomicrmw sub(ptr, neg(X)) -> atomicrmw add(ptr, X)`. / 注释说明了附近代码的逻辑或变换意图：`atomicrmw sub(ptr, neg(X)) -> atomicrmw add(ptr, X)`。
- **L124**: Comment documents the nearby logic or transformation intent: `old + (-X) == old - X and old - (-X) == old + X; the returned old value`. / 注释说明了附近代码的逻辑或变换意图：`old + (-X) == old - X and old - (-X) == old + X; the returned old value`。
- **L125**: Comment documents the nearby logic or transformation intent: `is identical in both cases. We match strictly on `sub 0, X` (negation) to`. / 注释说明了附近代码的逻辑或变换意图：`is identical in both cases. We match strictly on `sub 0, X` (negation) to`。
- **L126**: Comment documents the nearby logic or transformation intent: `avoid infinite loops: a general negation of `sub A, B` yields `sub B, A`,`. / 注释说明了附近代码的逻辑或变换意图：`avoid infinite loops: a general negation of `sub A, B` yields `sub B, A`,`。
- **L127**: Comment documents the nearby logic or transformation intent: `which would infinitely be negated back on the next iteration.`. / 注释说明了附近代码的逻辑或变换意图：`which would infinitely be negated back on the next iteration.`。
- **L128**: Initializes variable `Op` from the right-hand expression. / 使用右侧表达式初始化变量 `Op`。
- **L129**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L130**: Executes call or statement centered on `RMWI.getValOperand`. / 执行以 `RMWI.getValOperand` 为核心的调用或语句。
- **L131**: Executes a standalone statement or declaration: `Value *X;`. / 执行一条独立语句或声明：`Value *X;`。
- **L132**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L133**: Continues the surrounding expression or declaration: `RMWI.setOperation(Op == AtomicRMWInst::Add ? AtomicRMWInst::Sub`. / 继续构造周围的表达式或声明：`RMWI.setOperation(Op == AtomicRMWInst::Add ? AtomicRMWInst::Sub`。
- **L134**: Executes a standalone statement or declaration: `: AtomicRMWInst::Add);`. / 执行一条独立语句或声明：`: AtomicRMWInst::Add);`。
- **L135**: Returns from the current function with `replaceOperand(RMWI, 1, X)`. / 以 `replaceOperand(RMWI, 1, X)` 从当前函数返回。
- **L136**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L137**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L138**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L139**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L140**: Returns from the current function with `nullptr`. / 以 `nullptr` 从当前函数返回。

### Lines 141-157

```cpp

  // We chose to canonicalize all idempotent operations to an single
  // operation code and constant.  This makes it easier for the rest of the
  // optimizer to match easily.  The choices of or w/0 and fadd w/-0.0 are
  // arbitrary.
  if (RMWI.getType()->isIntegerTy() &&
      RMWI.getOperation() != AtomicRMWInst::Or) {
    RMWI.setOperation(AtomicRMWInst::Or);
    return replaceOperand(RMWI, 1, ConstantInt::get(RMWI.getType(), 0));
  } else if (RMWI.getType()->isFloatingPointTy() &&
             RMWI.getOperation() != AtomicRMWInst::FAdd) {
    RMWI.setOperation(AtomicRMWInst::FAdd);
    return replaceOperand(RMWI, 1, ConstantFP::getNegativeZero(RMWI.getType()));
  }

  return nullptr;
}
```

- **L141**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L142**: Comment documents the nearby logic or transformation intent: `We chose to canonicalize all idempotent operations to an single`. / 注释说明了附近代码的逻辑或变换意图：`We chose to canonicalize all idempotent operations to an single`。
- **L143**: Comment documents the nearby logic or transformation intent: `operation code and constant.  This makes it easier for the rest of the`. / 注释说明了附近代码的逻辑或变换意图：`operation code and constant.  This makes it easier for the rest of the`。
- **L144**: Comment documents the nearby logic or transformation intent: `optimizer to match easily.  The choices of or w/0 and fadd w/-0.0 are`. / 注释说明了附近代码的逻辑或变换意图：`optimizer to match easily.  The choices of or w/0 and fadd w/-0.0 are`。
- **L145**: Comment documents the nearby logic or transformation intent: `arbitrary.`. / 注释说明了附近代码的逻辑或变换意图：`arbitrary.`。
- **L146**: Begins a `if` control-flow statement and evaluates its condition. / 开始 `if` 控制流语句并计算其条件。
- **L147**: Starts a function, method, or lambda body: `RMWI.getOperation() != AtomicRMWInst::Or) {`. / 开始一个函数、方法或 lambda 的主体：`RMWI.getOperation() != AtomicRMWInst::Or) {`。
- **L148**: Executes call or statement centered on `RMWI.setOperation`. / 执行以 `RMWI.setOperation` 为核心的调用或语句。
- **L149**: Returns from the current function with `replaceOperand(RMWI, 1, ConstantInt::get(RMWI.getType(), 0))`. / 以 `replaceOperand(RMWI, 1, ConstantInt::get(RMWI.getType(), 0))` 从当前函数返回。
- **L150**: Continues the surrounding expression or declaration: `} else if (RMWI.getType()->isFloatingPointTy() &&`. / 继续构造周围的表达式或声明：`} else if (RMWI.getType()->isFloatingPointTy() &&`。
- **L151**: Starts a function, method, or lambda body: `RMWI.getOperation() != AtomicRMWInst::FAdd) {`. / 开始一个函数、方法或 lambda 的主体：`RMWI.getOperation() != AtomicRMWInst::FAdd) {`。
- **L152**: Executes call or statement centered on `RMWI.setOperation`. / 执行以 `RMWI.setOperation` 为核心的调用或语句。
- **L153**: Returns from the current function with `replaceOperand(RMWI, 1, ConstantFP::getNegativeZero(RMWI.getType()))`. / 以 `replaceOperand(RMWI, 1, ConstantFP::getNegativeZero(RMWI.getType()))` 从当前函数返回。
- **L154**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L155**: Blank line that separates nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L156**: Returns from the current function with `nullptr`. / 以 `nullptr` 从当前函数返回。
- **L157**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **InstCombine transform pipeline / InstCombine 变换流水线**
- **Pattern matching over LLVM IR / 针对 LLVM IR 的模式匹配**

## Dependencies / 依赖关系

- `InstCombineInternal.h`: Provides local declarations used by this file. / 提供本文件使用的本地声明。
- `llvm/IR/Instructions.h`: Provides LLVM IR core types and builders. / 提供LLVM IR 核心类型与构造工具。
