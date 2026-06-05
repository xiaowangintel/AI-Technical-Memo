# ThreadPlanCallOnFunctionExit.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Target/ThreadPlanCallOnFunctionExit.cpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: Implements LLDB logic for debug targets, processes, threads, breakpoints, memory, and execution control related to `ThreadPlanCallOnFunctionExit` in the `Target` subsystem.
- **Purpose (CN)**: 该文件在 LLDB 的 `Target` 子系统中实现与 `ThreadPlanCallOnFunctionExit` 相关的逻辑，重点覆盖调试目标、进程、线程、断点、内存与执行控制。对应英文说明：Implements LLDB logic for debug targets, processes, threads, breakpoints, memory, and execution control related to `ThreadPlanCallOnFunctionExit` in the `Target` subsystem。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行

````cpp
//===-- ThreadPlanCallOnFunctionExit.cpp ----------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "lldb/Target/ThreadPlanCallOnFunctionExit.h"

using namespace lldb;
using namespace lldb_private;

ThreadPlanCallOnFunctionExit::ThreadPlanCallOnFunctionExit(
    Thread &thread, const Callback &callback)
    : ThreadPlan(ThreadPlanKind::eKindGeneric, "CallOnFunctionExit", thread,
                 eVoteNoOpinion, eVoteNoOpinion // TODO check with Jim on these
                 ),
````
- **L1 EN**: Banner comment marks a file or section boundary.
  **L1 CN**: 横幅注释用于标记文件或章节边界。
- **L2 EN**: Separator comment visually groups nearby code.
  **L2 CN**: 分隔注释用于在视觉上分组附近代码。
- **L3 EN**: Comment explains surrounding design intent or invariants: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明周边设计意图或不变式：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains surrounding design intent or invariants: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明周边设计意图或不变式：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains surrounding design intent or invariants: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明周边设计意图或不变式：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment visually groups nearby code.
  **L6 CN**: 分隔注释用于在视觉上分组附近代码。
- **L7 EN**: Banner comment marks a file or section boundary.
  **L7 CN**: 横幅注释用于标记文件或章节边界。
- **L8 EN**: Blank line separates nearby declarations or logic blocks.
  **L8 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L9 EN**: Includes `lldb/Target/ThreadPlanCallOnFunctionExit.h` so this header can use target/process/thread execution-control facilities.
  **L9 CN**: 引入 `lldb/Target/ThreadPlanCallOnFunctionExit.h`，使该头文件能够使用目标/进程/线程执行控制设施。
- **L10 EN**: Blank line separates nearby declarations or logic blocks.
  **L10 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L11 EN**: Imports namespace `lldb` into the current scope.
  **L11 CN**: 将命名空间 `lldb` 导入当前作用域。
- **L12 EN**: Imports namespace `lldb_private` into the current scope.
  **L12 CN**: 将命名空间 `lldb_private` 导入当前作用域。
- **L13 EN**: Blank line separates nearby declarations or logic blocks.
  **L13 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L14 EN**: Continues logic associated with callable symbol `ThreadPlanCallOnFunctionExit`.
  **L14 CN**: 继续与可调用符号 `ThreadPlanCallOnFunctionExit` 相关的逻辑。
- **L15 EN**: Continues the surrounding declaration or expression: `Thread &thread, const Callback &callback)`.
  **L15 CN**: 继续构造周围的声明或表达式：`Thread &thread, const Callback &callback)`。
- **L16 EN**: Continues a multi-line list, initializer, or aggregate entry: `: ThreadPlan(ThreadPlanKind::eKindGeneric, "CallOnFunctionExit", thread,`.
  **L16 CN**: 继续一个多行列表、初始化器或聚合项：`: ThreadPlan(ThreadPlanKind::eKindGeneric, "CallOnFunctionExit", thread,`。
- **L17 EN**: Continues the surrounding declaration or expression: `eVoteNoOpinion, eVoteNoOpinion // TODO check with Jim on these`.
  **L17 CN**: 继续构造周围的声明或表达式：`eVoteNoOpinion, eVoteNoOpinion // TODO check with Jim on these`。
- **L18 EN**: Continues a multi-line list, initializer, or aggregate entry: `),`.
  **L18 CN**: 继续一个多行列表、初始化器或聚合项：`),`。

### Lines 19-36 / 第 19-36 行

````cpp
      m_callback(callback) {
  // We are not a user-generated plan.
  SetIsControllingPlan(false);
}

void ThreadPlanCallOnFunctionExit::DidPush() {
  // We now want to queue the "step out" thread plan so it executes and
  // completes.

  // Set stop vote to eVoteNo.
  Status status;
  m_step_out_threadplan_sp = GetThread().QueueThreadPlanForStepOut(
      false,             // abort other plans
      nullptr,           // addr_context
      true,              // first instruction
      true,              // stop other threads
      eVoteNo,           // do not say "we're stopping"
      eVoteNoOpinion,    // don't care about run state broadcasting
````
- **L19 EN**: Starts a function, method, lambda, or structured scope: `m_callback(callback) {`.
  **L19 CN**: 开始一个函数、方法、lambda 或结构化作用域：`m_callback(callback) {`。
- **L20 EN**: Comment explains surrounding design intent or invariants: `We are not a user-generated plan.`.
  **L20 CN**: 注释说明周边设计意图或不变式：`We are not a user-generated plan.`。
- **L21 EN**: Declares or invokes callable logic centered on `SetIsControllingPlan`.
  **L21 CN**: 声明或调用以 `SetIsControllingPlan` 为核心的可调用逻辑。
- **L22 EN**: Closes the current lexical scope or body.
  **L22 CN**: 关闭当前词法作用域或代码体。
- **L23 EN**: Blank line separates nearby declarations or logic blocks.
  **L23 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L24 EN**: Starts a function, method, lambda, or structured scope: `void ThreadPlanCallOnFunctionExit::DidPush() {`.
  **L24 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void ThreadPlanCallOnFunctionExit::DidPush() {`。
- **L25 EN**: Comment explains surrounding design intent or invariants: `We now want to queue the "step out" thread plan so it executes and`.
  **L25 CN**: 注释说明周边设计意图或不变式：`We now want to queue the "step out" thread plan so it executes and`。
- **L26 EN**: Comment explains surrounding design intent or invariants: `completes.`.
  **L26 CN**: 注释说明周边设计意图或不变式：`completes.`。
- **L27 EN**: Blank line separates nearby declarations or logic blocks.
  **L27 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L28 EN**: Comment explains surrounding design intent or invariants: `Set stop vote to eVoteNo.`.
  **L28 CN**: 注释说明周边设计意图或不变式：`Set stop vote to eVoteNo.`。
- **L29 EN**: Completes a standalone declaration or statement: `Status status;`.
  **L29 CN**: 完成一条独立声明或语句：`Status status;`。
- **L30 EN**: Continues logic associated with callable symbol `GetThread`.
  **L30 CN**: 继续与可调用符号 `GetThread` 相关的逻辑。
- **L31 EN**: Continues the surrounding declaration or expression: `false,             // abort other plans`.
  **L31 CN**: 继续构造周围的声明或表达式：`false,             // abort other plans`。
- **L32 EN**: Continues the surrounding declaration or expression: `nullptr,           // addr_context`.
  **L32 CN**: 继续构造周围的声明或表达式：`nullptr,           // addr_context`。
- **L33 EN**: Continues the surrounding declaration or expression: `true,              // first instruction`.
  **L33 CN**: 继续构造周围的声明或表达式：`true,              // first instruction`。
- **L34 EN**: Continues the surrounding declaration or expression: `true,              // stop other threads`.
  **L34 CN**: 继续构造周围的声明或表达式：`true,              // stop other threads`。
- **L35 EN**: Continues the surrounding declaration or expression: `eVoteNo,           // do not say "we're stopping"`.
  **L35 CN**: 继续构造周围的声明或表达式：`eVoteNo,           // do not say "we're stopping"`。
- **L36 EN**: Continues the surrounding declaration or expression: `eVoteNoOpinion,    // don't care about run state broadcasting`.
  **L36 CN**: 继续构造周围的声明或表达式：`eVoteNoOpinion,    // don't care about run state broadcasting`。

### Lines 37-54 / 第 37-54 行

````cpp
      0,                 // frame_idx
      status,            // status
      eLazyBoolCalculate // avoid code w/o debinfo
  );
}

// ThreadPlan API

void ThreadPlanCallOnFunctionExit::GetDescription(
    Stream *s, lldb::DescriptionLevel level) {
  if (!s)
    return;
  s->Printf("Running until completion of current function, then making "
            "callback.");
}

bool ThreadPlanCallOnFunctionExit::ValidatePlan(Stream *error) {
  // We'll say we're always good since I don't know what would make this
````
- **L37 EN**: Continues the surrounding declaration or expression: `0,                 // frame_idx`.
  **L37 CN**: 继续构造周围的声明或表达式：`0,                 // frame_idx`。
- **L38 EN**: Continues the surrounding declaration or expression: `status,            // status`.
  **L38 CN**: 继续构造周围的声明或表达式：`status,            // status`。
- **L39 EN**: Continues the surrounding declaration or expression: `eLazyBoolCalculate // avoid code w/o debinfo`.
  **L39 CN**: 继续构造周围的声明或表达式：`eLazyBoolCalculate // avoid code w/o debinfo`。
- **L40 EN**: Completes a standalone declaration or statement: `);`.
  **L40 CN**: 完成一条独立声明或语句：`);`。
- **L41 EN**: Closes the current lexical scope or body.
  **L41 CN**: 关闭当前词法作用域或代码体。
- **L42 EN**: Blank line separates nearby declarations or logic blocks.
  **L42 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L43 EN**: Comment explains surrounding design intent or invariants: `ThreadPlan API`.
  **L43 CN**: 注释说明周边设计意图或不变式：`ThreadPlan API`。
- **L44 EN**: Blank line separates nearby declarations or logic blocks.
  **L44 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L45 EN**: Continues logic associated with callable symbol `GetDescription`.
  **L45 CN**: 继续与可调用符号 `GetDescription` 相关的逻辑。
- **L46 EN**: Continues the surrounding declaration or expression: `Stream *s, lldb::DescriptionLevel level) {`.
  **L46 CN**: 继续构造周围的声明或表达式：`Stream *s, lldb::DescriptionLevel level) {`。
- **L47 EN**: Begins a `if` control-flow statement.
  **L47 CN**: 开始一个 `if` 控制流语句。
- **L48 EN**: Returns from the current function with `void`.
  **L48 CN**: 以 `void` 从当前函数返回。
- **L49 EN**: Continues logic associated with callable symbol `Printf`.
  **L49 CN**: 继续与可调用符号 `Printf` 相关的逻辑。
- **L50 EN**: Completes a standalone declaration or statement: `"callback.");`.
  **L50 CN**: 完成一条独立声明或语句：`"callback.");`。
- **L51 EN**: Closes the current lexical scope or body.
  **L51 CN**: 关闭当前词法作用域或代码体。
- **L52 EN**: Blank line separates nearby declarations or logic blocks.
  **L52 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L53 EN**: Starts a function, method, lambda, or structured scope: `bool ThreadPlanCallOnFunctionExit::ValidatePlan(Stream *error) {`.
  **L53 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool ThreadPlanCallOnFunctionExit::ValidatePlan(Stream *error) {`。
- **L54 EN**: Comment explains surrounding design intent or invariants: `We'll say we're always good since I don't know what would make this`.
  **L54 CN**: 注释说明周边设计意图或不变式：`We'll say we're always good since I don't know what would make this`。

### Lines 55-72 / 第 55-72 行

````cpp
  // invalid.
  return true;
}

bool ThreadPlanCallOnFunctionExit::ShouldStop(Event *event_ptr) {
  // If this is where we find out that an internal stop came in, then: Check if
  // the step-out plan completed.  If it did, then we want to run the callback
  // here (our reason for living...)
  if (m_step_out_threadplan_sp && m_step_out_threadplan_sp->IsPlanComplete()) {
    m_callback();

    // We no longer need the pointer to the step-out thread plan.
    m_step_out_threadplan_sp.reset();

    // Indicate that this plan is done and can be discarded.
    SetPlanComplete();

    // We're done now, but we want to return false so that we don't cause the
````
- **L55 EN**: Comment explains surrounding design intent or invariants: `invalid.`.
  **L55 CN**: 注释说明周边设计意图或不变式：`invalid.`。
- **L56 EN**: Returns from the current function with `true`.
  **L56 CN**: 以 `true` 从当前函数返回。
- **L57 EN**: Closes the current lexical scope or body.
  **L57 CN**: 关闭当前词法作用域或代码体。
- **L58 EN**: Blank line separates nearby declarations or logic blocks.
  **L58 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L59 EN**: Starts a function, method, lambda, or structured scope: `bool ThreadPlanCallOnFunctionExit::ShouldStop(Event *event_ptr) {`.
  **L59 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool ThreadPlanCallOnFunctionExit::ShouldStop(Event *event_ptr) {`。
- **L60 EN**: Comment explains surrounding design intent or invariants: `If this is where we find out that an internal stop came in, then: Check if`.
  **L60 CN**: 注释说明周边设计意图或不变式：`If this is where we find out that an internal stop came in, then: Check if`。
- **L61 EN**: Comment explains surrounding design intent or invariants: `the step-out plan completed.  If it did, then we want to run the callback`.
  **L61 CN**: 注释说明周边设计意图或不变式：`the step-out plan completed.  If it did, then we want to run the callback`。
- **L62 EN**: Comment explains surrounding design intent or invariants: `here (our reason for living...)`.
  **L62 CN**: 注释说明周边设计意图或不变式：`here (our reason for living...)`。
- **L63 EN**: Begins a `if` control-flow statement.
  **L63 CN**: 开始一个 `if` 控制流语句。
- **L64 EN**: Declares or invokes callable logic centered on `m_callback`.
  **L64 CN**: 声明或调用以 `m_callback` 为核心的可调用逻辑。
- **L65 EN**: Blank line separates nearby declarations or logic blocks.
  **L65 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L66 EN**: Comment explains surrounding design intent or invariants: `We no longer need the pointer to the step-out thread plan.`.
  **L66 CN**: 注释说明周边设计意图或不变式：`We no longer need the pointer to the step-out thread plan.`。
- **L67 EN**: Declares or invokes callable logic centered on `m_step_out_threadplan_sp.reset`.
  **L67 CN**: 声明或调用以 `m_step_out_threadplan_sp.reset` 为核心的可调用逻辑。
- **L68 EN**: Blank line separates nearby declarations or logic blocks.
  **L68 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L69 EN**: Comment explains surrounding design intent or invariants: `Indicate that this plan is done and can be discarded.`.
  **L69 CN**: 注释说明周边设计意图或不变式：`Indicate that this plan is done and can be discarded.`。
- **L70 EN**: Declares or invokes callable logic centered on `SetPlanComplete`.
  **L70 CN**: 声明或调用以 `SetPlanComplete` 为核心的可调用逻辑。
- **L71 EN**: Blank line separates nearby declarations or logic blocks.
  **L71 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L72 EN**: Comment explains surrounding design intent or invariants: `We're done now, but we want to return false so that we don't cause the`.
  **L72 CN**: 注释说明周边设计意图或不变式：`We're done now, but we want to return false so that we don't cause the`。

### Lines 73-90 / 第 73-90 行

````cpp
    // thread to really stop.
  }

  return false;
}

bool ThreadPlanCallOnFunctionExit::WillStop() {
  // The code looks like the return value is ignored via ThreadList::
  // ShouldStop(). This is called when we really are going to stop.  We don't
  // care and don't need to do anything here.
  return false;
}

bool ThreadPlanCallOnFunctionExit::DoPlanExplainsStop(Event *event_ptr) {
  // We don't ever explain a stop.  The only stop that is relevant to us
  // directly is the step_out plan we added to do the heavy lifting of getting
  // us past the current method.
  return false;
````
- **L73 EN**: Comment explains surrounding design intent or invariants: `thread to really stop.`.
  **L73 CN**: 注释说明周边设计意图或不变式：`thread to really stop.`。
- **L74 EN**: Closes the current lexical scope or body.
  **L74 CN**: 关闭当前词法作用域或代码体。
- **L75 EN**: Blank line separates nearby declarations or logic blocks.
  **L75 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L76 EN**: Returns from the current function with `false`.
  **L76 CN**: 以 `false` 从当前函数返回。
- **L77 EN**: Closes the current lexical scope or body.
  **L77 CN**: 关闭当前词法作用域或代码体。
- **L78 EN**: Blank line separates nearby declarations or logic blocks.
  **L78 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L79 EN**: Starts a function, method, lambda, or structured scope: `bool ThreadPlanCallOnFunctionExit::WillStop() {`.
  **L79 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool ThreadPlanCallOnFunctionExit::WillStop() {`。
- **L80 EN**: Comment explains surrounding design intent or invariants: `The code looks like the return value is ignored via ThreadList::`.
  **L80 CN**: 注释说明周边设计意图或不变式：`The code looks like the return value is ignored via ThreadList::`。
- **L81 EN**: Comment explains surrounding design intent or invariants: `ShouldStop(). This is called when we really are going to stop.  We don't`.
  **L81 CN**: 注释说明周边设计意图或不变式：`ShouldStop(). This is called when we really are going to stop.  We don't`。
- **L82 EN**: Comment explains surrounding design intent or invariants: `care and don't need to do anything here.`.
  **L82 CN**: 注释说明周边设计意图或不变式：`care and don't need to do anything here.`。
- **L83 EN**: Returns from the current function with `false`.
  **L83 CN**: 以 `false` 从当前函数返回。
- **L84 EN**: Closes the current lexical scope or body.
  **L84 CN**: 关闭当前词法作用域或代码体。
- **L85 EN**: Blank line separates nearby declarations or logic blocks.
  **L85 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L86 EN**: Starts a function, method, lambda, or structured scope: `bool ThreadPlanCallOnFunctionExit::DoPlanExplainsStop(Event *event_ptr) {`.
  **L86 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool ThreadPlanCallOnFunctionExit::DoPlanExplainsStop(Event *event_ptr) {`。
- **L87 EN**: Comment explains surrounding design intent or invariants: `We don't ever explain a stop.  The only stop that is relevant to us`.
  **L87 CN**: 注释说明周边设计意图或不变式：`We don't ever explain a stop.  The only stop that is relevant to us`。
- **L88 EN**: Comment explains surrounding design intent or invariants: `directly is the step_out plan we added to do the heavy lifting of getting`.
  **L88 CN**: 注释说明周边设计意图或不变式：`directly is the step_out plan we added to do the heavy lifting of getting`。
- **L89 EN**: Comment explains surrounding design intent or invariants: `us past the current method.`.
  **L89 CN**: 注释说明周边设计意图或不变式：`us past the current method.`。
- **L90 EN**: Returns from the current function with `false`.
  **L90 CN**: 以 `false` 从当前函数返回。

### Lines 91-97 / 第 91-97 行

````cpp
}

lldb::StateType ThreadPlanCallOnFunctionExit::GetPlanRunState() {
  // This value doesn't matter - we'll never be the top thread plan, so nobody
  // will ask us this question.
  return eStateRunning;
}
````
- **L91 EN**: Closes the current lexical scope or body.
  **L91 CN**: 关闭当前词法作用域或代码体。
- **L92 EN**: Blank line separates nearby declarations or logic blocks.
  **L92 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L93 EN**: Starts a function, method, lambda, or structured scope: `lldb::StateType ThreadPlanCallOnFunctionExit::GetPlanRunState() {`.
  **L93 CN**: 开始一个函数、方法、lambda 或结构化作用域：`lldb::StateType ThreadPlanCallOnFunctionExit::GetPlanRunState() {`。
- **L94 EN**: Comment explains surrounding design intent or invariants: `This value doesn't matter - we'll never be the top thread plan, so nobody`.
  **L94 CN**: 注释说明周边设计意图或不变式：`This value doesn't matter - we'll never be the top thread plan, so nobody`。
- **L95 EN**: Comment explains surrounding design intent or invariants: `will ask us this question.`.
  **L95 CN**: 注释说明周边设计意图或不变式：`will ask us this question.`。
- **L96 EN**: Returns from the current function with `eStateRunning`.
  **L96 CN**: 以 `eStateRunning` 从当前函数返回。
- **L97 EN**: Closes the current lexical scope or body.
  **L97 CN**: 关闭当前词法作用域或代码体。

## Key Concepts / 关键概念

- **Role / 角色**: This file is a implementation unit inside LLDB's **Target** area. / 该文件是 LLDB **Target** 范围内的实现文件。
- **Scale / 规模**: 97 lines with 1 direct includes. / 共 97 行，直接包含 1 个头文件。
- **Subsystem focus / 子系统关注点**: execution-state control, breakpoint/watchpoint modeling, memory and register access. / 执行状态控制、断点/观察点建模、内存与寄存器访问。
- **Visible entry points / 关键入口**: `m_callback`, `SetIsControllingPlan`, `ThreadPlanCallOnFunctionExit::DidPush`, `ThreadPlanCallOnFunctionExit::ValidatePlan`, `ThreadPlanCallOnFunctionExit::ShouldStop`, `reset`, `SetPlanComplete`, `ThreadPlanCallOnFunctionExit::WillStop`, `ThreadPlanCallOnFunctionExit::DoPlanExplainsStop`, `ThreadPlanCallOnFunctionExit::GetPlanRunState`. / 可见的关键入口包括 `m_callback`, `SetIsControllingPlan`, `ThreadPlanCallOnFunctionExit::DidPush`, `ThreadPlanCallOnFunctionExit::ValidatePlan`, `ThreadPlanCallOnFunctionExit::ShouldStop`, `reset`, `SetPlanComplete`, `ThreadPlanCallOnFunctionExit::WillStop`, `ThreadPlanCallOnFunctionExit::DoPlanExplainsStop`, `ThreadPlanCallOnFunctionExit::GetPlanRunState`。
- **Concept / 概念**: Event delivery. / 事件传递。
- **Concept / 概念**: Status and error reporting. / 状态与错误报告。
- **Concept / 概念**: Stream-based formatting/output. / 基于流的格式化/输出。
- **Concept / 概念**: Target abstraction. / 目标抽象。

## Dependencies / 依赖关系

- **LLDB headers / LLDB 头文件**: `lldb/Target/ThreadPlanCallOnFunctionExit.h`.
- **Callable interfaces / 可调用接口**: `m_callback`, `SetIsControllingPlan`, `ThreadPlanCallOnFunctionExit::DidPush`, `ThreadPlanCallOnFunctionExit::ValidatePlan`, `ThreadPlanCallOnFunctionExit::ShouldStop`, `reset`, `SetPlanComplete`, `ThreadPlanCallOnFunctionExit::WillStop`, `ThreadPlanCallOnFunctionExit::DoPlanExplainsStop`, `ThreadPlanCallOnFunctionExit::GetPlanRunState`.
