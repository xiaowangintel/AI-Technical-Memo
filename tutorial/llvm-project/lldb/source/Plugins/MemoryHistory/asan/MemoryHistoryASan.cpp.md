# MemoryHistoryASan.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/MemoryHistory/asan/MemoryHistoryASan.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `MemoryHistoryASan`.
  - **CN**: 实现与 `MemoryHistoryASan` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- MemoryHistoryASan.cpp ---------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/LLDB file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/LLDB 标准文件头、许可证声明以及文件的高层说明。

### Lines 8-21
```cpp

#include "MemoryHistoryASan.h"

#include "lldb/Symbol/SymbolContext.h"
#include "lldb/Target/MemoryHistory.h"

#include "Plugins/InstrumentationRuntime/Utility/Utility.h"
#include "Plugins/Process/Utility/HistoryThread.h"
#include "lldb/Core/Debugger.h"
#include "lldb/Core/Module.h"
#include "lldb/Core/PluginInterface.h"
#include "lldb/Core/PluginManager.h"
#include "lldb/Expression/UserExpression.h"
#include "lldb/Target/ExecutionContext.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `MemoryHistoryASan.h`, `lldb/Symbol/SymbolContext.h`, `lldb/Target/MemoryHistory.h`, `Plugins/InstrumentationRuntime/Utility/Utility.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `MemoryHistoryASan.h`, `lldb/Symbol/SymbolContext.h`, `lldb/Target/MemoryHistory.h`, `Plugins/InstrumentationRuntime/Utility/Utility.h`。

### Lines 22-29
```cpp
#include "lldb/Target/Target.h"
#include "lldb/Target/Thread.h"
#include "lldb/Target/ThreadList.h"
#include "lldb/ValueObject/ValueObject.h"
#include "lldb/lldb-private.h"

#include <sstream>

```
- **EN**: Pulls in the headers needed by this translation unit, including `lldb/Target/Target.h`, `lldb/Target/Thread.h`, `lldb/Target/ThreadList.h`, `lldb/ValueObject/ValueObject.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `lldb/Target/Target.h`, `lldb/Target/Thread.h`, `lldb/Target/ThreadList.h`, `lldb/ValueObject/ValueObject.h`。

### Lines 30-38
```cpp
using namespace lldb;
using namespace lldb_private;

LLDB_PLUGIN_DEFINE(MemoryHistoryASan)

MemoryHistorySP MemoryHistoryASan::CreateInstance(const ProcessSP &process_sp) {
  if (!process_sp.get())
    return nullptr;

```
- **EN**: Implements logic around `LLDB_PLUGIN_DEFINE`, `CreateInstance`, `get`.
- **CN**: 围绕 `LLDB_PLUGIN_DEFINE`, `CreateInstance`, `get` 实现具体逻辑。

### Lines 39-48
```cpp
  Target &target = process_sp->GetTarget();

  for (ModuleSP module_sp : target.GetImages().Modules()) {
    const Symbol *symbol = module_sp->FindFirstSymbolWithNameAndType(
        ConstString("__asan_get_alloc_stack"), lldb::eSymbolTypeAny);

    if (symbol != nullptr)
      return MemoryHistorySP(new MemoryHistoryASan(process_sp));
  }

```
- **EN**: Implements logic around `GetTarget`, `GetImages`, `FindFirstSymbolWithNameAndType`, `ConstString`, and 1 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `GetTarget`, `GetImages`, `FindFirstSymbolWithNameAndType`, `ConstString`, and 1 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址。

### Lines 49-56
```cpp
  return MemoryHistorySP();
}

void MemoryHistoryASan::Initialize() {
  PluginManager::RegisterPlugin(
      GetPluginNameStatic(), "ASan memory history provider.", CreateInstance);
}

```
- **EN**: Implements logic around `MemoryHistorySP`, `Initialize`, `RegisterPlugin`, `GetPluginNameStatic`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `MemoryHistorySP`, `Initialize`, `RegisterPlugin`, `GetPluginNameStatic` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 57-65
```cpp
void MemoryHistoryASan::Terminate() {
  PluginManager::UnregisterPlugin(CreateInstance);
}

MemoryHistoryASan::MemoryHistoryASan(const ProcessSP &process_sp) {
  if (process_sp)
    m_process_wp = process_sp;
}

```
- **EN**: Implements logic around `Terminate`, `UnregisterPlugin`, `MemoryHistoryASan`.
- **CN**: 围绕 `Terminate`, `UnregisterPlugin`, `MemoryHistoryASan` 实现具体逻辑。

### Lines 66-73
```cpp
const char *memory_history_asan_command_prefix = R"(
    extern "C"
    {
        size_t __asan_get_alloc_stack(void *addr, void **trace, size_t size, int *thread_id);
        size_t __asan_get_free_stack(void *addr, void **trace, size_t size, int *thread_id);
    }
)";

```
- **EN**: Implements logic around `__asan_get_alloc_stack`, `__asan_get_free_stack`.
- **CN**: 围绕 `__asan_get_alloc_stack`, `__asan_get_free_stack` 实现具体逻辑。

### Lines 74-80
```cpp
const char *memory_history_asan_command_format =
    R"(
    struct {
        void *alloc_trace[256];
        size_t alloc_count;
        int alloc_tid;

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 81-90
```cpp
        void *free_trace[256];
        size_t free_count;
        int free_tid;
    } t;

    t.alloc_count = __asan_get_alloc_stack((void *)0x%)" PRIx64
    R"(, t.alloc_trace, 256, &t.alloc_tid);
    t.free_count = __asan_get_free_stack((void *)0x%)" PRIx64
    R"(, t.free_trace, 256, &t.free_tid);

```
- **EN**: Implements logic around `__asan_get_alloc_stack`, `__asan_get_free_stack`.
- **CN**: 围绕 `__asan_get_alloc_stack`, `__asan_get_free_stack` 实现具体逻辑。

### Lines 91-100
```cpp
    t;
)";

static void CreateHistoryThreadFromValueObject(
    ProcessSP process_sp, ValueObjectSP return_value_sp, HistoryPCType pc_type,
    const char *type, const char *thread_name, HistoryThreads &result) {
  std::string count_path = "." + std::string(type) + "_count";
  std::string tid_path = "." + std::string(type) + "_tid";
  std::string trace_path = "." + std::string(type) + "_trace";

```
- **EN**: Implements logic around `CreateHistoryThreadFromValueObject`, `string`.
- **CN**: 围绕 `CreateHistoryThreadFromValueObject`, `string` 实现具体逻辑。

### Lines 101-108
```cpp
  ValueObjectSP count_sp =
      return_value_sp->GetValueForExpressionPath(count_path.c_str());
  ValueObjectSP tid_sp =
      return_value_sp->GetValueForExpressionPath(tid_path.c_str());

  if (!count_sp || !tid_sp)
    return;

```
- **EN**: Implements logic around `GetValueForExpressionPath`.
- **CN**: 围绕 `GetValueForExpressionPath` 实现具体逻辑。

### Lines 109-117
```cpp
  int count = count_sp->GetValueAsUnsigned(0);
  lldb::tid_t tid = tid_sp->GetValueAsUnsigned(0) + 1;

  if (count <= 0)
    return;

  ValueObjectSP trace_sp =
      return_value_sp->GetValueForExpressionPath(trace_path.c_str());

```
- **EN**: Implements logic around `GetValueAsUnsigned`, `GetValueForExpressionPath`.
- **CN**: 围绕 `GetValueAsUnsigned`, `GetValueForExpressionPath` 实现具体逻辑。

### Lines 118-128
```cpp
  if (!trace_sp)
    return;

  std::vector<lldb::addr_t> pcs;
  for (int i = 0; i < count; i++) {
    addr_t pc = trace_sp->GetChildAtIndex(i)->GetValueAsUnsigned(0);
    if (pc == 0 || pc == 1 || pc == LLDB_INVALID_ADDRESS)
      continue;
    pcs.push_back(pc);
  }

```
- **EN**: Implements logic around `GetChildAtIndex`, `push_back`.
- **CN**: 围绕 `GetChildAtIndex`, `push_back` 实现具体逻辑。

### Lines 129-140
```cpp
  HistoryThread *history_thread =
      new HistoryThread(*process_sp, tid, pcs, pc_type);
  ThreadSP new_thread_sp(history_thread);
  std::ostringstream thread_name_with_number;
  thread_name_with_number << thread_name << " Thread " << tid;
  history_thread->SetThreadName(thread_name_with_number.str().c_str());
  // Save this in the Process' ExtendedThreadList so a strong pointer retains
  // the object
  process_sp->GetExtendedThreadList().AddThread(new_thread_sp);
  result.push_back(new_thread_sp);
}

```
- **EN**: Implements logic around `HistoryThread`, `new_thread_sp`, `SetThreadName`, `GetExtendedThreadList`, and 1 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `HistoryThread`, `new_thread_sp`, `SetThreadName`, `GetExtendedThreadList`, and 1 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 141-147
```cpp
HistoryThreads MemoryHistoryASan::GetHistoryThreads(lldb::addr_t address) {
  HistoryThreads result;

  ProcessSP process_sp = m_process_wp.lock();
  if (!process_sp)
    return result;

```
- **EN**: Implements logic around `GetHistoryThreads`, `lock`; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `GetHistoryThreads`, `lock` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 148-157
```cpp
  ThreadSP thread_sp =
      process_sp->GetThreadList().GetExpressionExecutionThread();
  if (!thread_sp)
    return result;

  StackFrameSP frame_sp =
      thread_sp->GetSelectedFrame(DoNoSelectMostRelevantFrame);
  if (!frame_sp)
    return result;

```
- **EN**: Implements logic around `GetThreadList`, `GetSelectedFrame`.
- **CN**: 围绕 `GetThreadList`, `GetSelectedFrame` 实现具体逻辑。

### Lines 158-171
```cpp
  ExecutionContext exe_ctx(frame_sp);
  ValueObjectSP return_value_sp;
  StreamString expr;
  expr.Printf(memory_history_asan_command_format, address, address);

  EvaluateExpressionOptions options;
  options.SetUnwindOnError(true);
  options.SetTryAllThreads(true);
  options.SetStopOthers(true);
  options.SetIgnoreBreakpoints(true);
  options.SetTimeout(process_sp->GetUtilityExpressionTimeout());
  options.SetPrefix(memory_history_asan_command_prefix);
  options.SetAutoApplyFixIts(false);
  options.SetLanguage(eLanguageTypeObjC_plus_plus);
```
- **EN**: Implements logic around `exe_ctx`, `Printf`, `SetUnwindOnError`, `SetTryAllThreads`, and 6 more symbols; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `exe_ctx`, `Printf`, `SetUnwindOnError`, `SetTryAllThreads`, and 6 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 172-179
```cpp

  auto [m, pc_type] = GetPreferredAsanModule(process_sp->GetTarget());
  if (m) {
    SymbolContextList sc_list;
    sc_list.Append(SymbolContext(std::move(m)));
    options.SetPreferredSymbolContexts(std::move(sc_list));
  }

```
- **EN**: Implements logic around `GetPreferredAsanModule`, `Append`, `SetPreferredSymbolContexts`; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `GetPreferredAsanModule`, `Append`, `SetPreferredSymbolContexts` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 180-191
```cpp
  ExpressionResults expr_result = UserExpression::Evaluate(
      exe_ctx, options, expr.GetString(), "", return_value_sp);
  if (expr_result != eExpressionCompleted) {
    StreamString ss;
    ss << "cannot evaluate AddressSanitizer expression:\n";
    if (return_value_sp)
      ss << return_value_sp->GetError().AsCString();
    Debugger::ReportWarning(ss.GetString().str(),
                            process_sp->GetTarget().GetDebugger().GetID());
    return result;
  }

```
- **EN**: Implements logic around `Evaluate`, `GetString`, `GetError`, `ReportWarning`, and 1 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; supports expression parsing, wrapping, or debug-time code generation.
- **CN**: 围绕 `Evaluate`, `GetString`, `GetError`, `ReportWarning`, and 1 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并支持表达式解析、包装或调试期代码生成。

### Lines 192-199
```cpp
  if (!return_value_sp)
    return result;

  CreateHistoryThreadFromValueObject(process_sp, return_value_sp, pc_type,
                                     "free", "Memory deallocated by", result);
  CreateHistoryThreadFromValueObject(process_sp, return_value_sp, pc_type,
                                     "alloc", "Memory allocated by", result);

```
- **EN**: Implements logic around `CreateHistoryThreadFromValueObject`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `CreateHistoryThreadFromValueObject` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 200-201
```cpp
  return result;
}
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

## Key Concepts / 关键概念

- **LLDB architecture / LLDB 架构**:
  - **EN**: Explains how LLDB organizes debugger APIs, runtime objects, and platform-specific helpers.
  - **CN**: 说明 LLDB 如何组织调试器 API、运行时对象以及平台相关辅助组件。
- **Stop control / 停机控制**:
  - **EN**: Represents stop reasons, conditions, callbacks, and breakpoint resolution state.
  - **CN**: 表示停机原因、条件、回调以及断点解析状态。
- **Symbolication / 符号化**:
  - **EN**: Relates raw addresses back to modules, sections, symbols, and source locations.
  - **CN**: 将原始地址关联回模块、节区、符号与源码位置。
- **Runtime control / 运行时控制**:
  - **EN**: Models debugger ownership over targets, processes, threads, and event delivery.
  - **CN**: 建模调试器对目标、进程、线程以及事件投递的控制。
- **Value presentation / 值展示**:
  - **EN**: Controls how variables are rendered, summarized, and expanded in debugger views.
  - **CN**: 控制变量在调试器视图中的渲染、摘要与展开方式。
- **User expressions / 用户表达式**:
  - **EN**: Wraps and evaluates source snippets inside the debuggee context.
  - **CN**: 在被调试进程上下文中包装并求值源码片段。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `MemoryHistoryASan.h`, `lldb/Symbol/SymbolContext.h`, `lldb/Target/MemoryHistory.h`, `Plugins/InstrumentationRuntime/Utility/Utility.h`, `Plugins/Process/Utility/HistoryThread.h`, `lldb/Core/Debugger.h`, `lldb/Core/Module.h`, `lldb/Core/PluginInterface.h`, `lldb/Core/PluginManager.h`, `lldb/Expression/UserExpression.h` ... (+6 more)
- **Standard-library headers / 标准库头文件**: `<sstream>`
- **Subsystem categories / 子系统类别**: target, process, and thread control / 目标、进程与线程控制 (5), LLDB core debugger abstractions / LLDB 核心调试抽象 (4), symbol and debug-info abstractions / 符号与调试信息抽象 (1), expression parsing and evaluation support / 表达式解析与求值支持 (1), value inspection and presentation helpers / 值检查与展示辅助逻辑 (1), shared LLDB public/private type declarations / 共享 LLDB 公共/私有类型声明 (1)
