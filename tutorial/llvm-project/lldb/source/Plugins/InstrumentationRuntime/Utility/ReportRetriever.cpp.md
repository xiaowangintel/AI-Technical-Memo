# ReportRetriever.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/InstrumentationRuntime/Utility/ReportRetriever.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `ReportRetriever`.
  - **CN**: 实现与 `ReportRetriever` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- ReportRetriever.cpp -----------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/LLDB file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/LLDB 标准文件头、许可证声明以及文件的高层说明。

### Lines 8-18
```cpp

#include "ReportRetriever.h"
#include "Utility.h"

#include "lldb/Breakpoint/StoppointCallbackContext.h"
#include "lldb/Core/Debugger.h"
#include "lldb/Core/Module.h"
#include "lldb/Expression/UserExpression.h"
#include "lldb/Target/InstrumentationRuntimeStopInfo.h"
#include "lldb/ValueObject/ValueObject.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `ReportRetriever.h`, `Utility.h`, `lldb/Breakpoint/StoppointCallbackContext.h`, `lldb/Core/Debugger.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `ReportRetriever.h`, `Utility.h`, `lldb/Breakpoint/StoppointCallbackContext.h`, `lldb/Core/Debugger.h`。

### Lines 19-32
```cpp
using namespace lldb;
using namespace lldb_private;

const char *address_sanitizer_retrieve_report_data_prefix = R"(
extern "C"
{
int __asan_report_present();
void *__asan_get_report_pc();
void *__asan_get_report_bp();
void *__asan_get_report_sp();
void *__asan_get_report_address();
const char *__asan_get_report_description();
int __asan_get_report_access_type();
size_t __asan_get_report_access_size();
```
- **EN**: Implements logic around `__asan_report_present`, `__asan_get_report_pc`, `__asan_get_report_bp`, `__asan_get_report_sp`, and 4 more symbols.
- **CN**: 围绕 `__asan_report_present`, `__asan_get_report_pc`, `__asan_get_report_bp`, `__asan_get_report_sp`, and 4 more symbols 实现具体逻辑。

### Lines 33-46
```cpp
}
)";

const char *address_sanitizer_retrieve_report_data_command = R"(
struct {
    int present;
    int access_type;
    void *pc;
    void *bp;
    void *sp;
    void *address;
    size_t access_size;
    const char *description;
} t;
```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 47-58
```cpp

t.present = __asan_report_present();
t.access_type = __asan_get_report_access_type();
t.pc = __asan_get_report_pc();
t.bp = __asan_get_report_bp();
t.sp = __asan_get_report_sp();
t.address = __asan_get_report_address();
t.access_size = __asan_get_report_access_size();
t.description = __asan_get_report_description();
t
)";

```
- **EN**: Implements logic around `__asan_report_present`, `__asan_get_report_access_type`, `__asan_get_report_pc`, `__asan_get_report_bp`, and 4 more symbols; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `__asan_report_present`, `__asan_get_report_access_type`, `__asan_get_report_pc`, `__asan_get_report_bp`, and 4 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 59-66
```cpp
StructuredData::ObjectSP
ReportRetriever::RetrieveReportData(const ProcessSP process_sp) {
  if (!process_sp)
    return StructuredData::ObjectSP();

  ThreadSP thread_sp =
      process_sp->GetThreadList().GetExpressionExecutionThread();

```
- **EN**: Implements logic around `RetrieveReportData`, `ObjectSP`, `GetThreadList`.
- **CN**: 围绕 `RetrieveReportData`, `ObjectSP`, `GetThreadList` 实现具体逻辑。

### Lines 67-75
```cpp
  if (!thread_sp)
    return StructuredData::ObjectSP();

  StackFrameSP frame_sp =
      thread_sp->GetSelectedFrame(DoNoSelectMostRelevantFrame);

  if (!frame_sp)
    return StructuredData::ObjectSP();

```
- **EN**: Implements logic around `ObjectSP`, `GetSelectedFrame`.
- **CN**: 围绕 `ObjectSP`, `GetSelectedFrame` 实现具体逻辑。

### Lines 76-85
```cpp
  EvaluateExpressionOptions options;
  options.SetUnwindOnError(true);
  options.SetTryAllThreads(true);
  options.SetStopOthers(true);
  options.SetIgnoreBreakpoints(true);
  options.SetTimeout(process_sp->GetUtilityExpressionTimeout());
  options.SetPrefix(address_sanitizer_retrieve_report_data_prefix);
  options.SetAutoApplyFixIts(false);
  options.SetLanguage(eLanguageTypeC);

```
- **EN**: Implements logic around `SetUnwindOnError`, `SetTryAllThreads`, `SetStopOthers`, `SetIgnoreBreakpoints`, and 4 more symbols.
- **CN**: 围绕 `SetUnwindOnError`, `SetTryAllThreads`, `SetStopOthers`, `SetIgnoreBreakpoints`, and 4 more symbols 实现具体逻辑。

### Lines 86-99
```cpp
  if (auto [m, _] = GetPreferredAsanModule(process_sp->GetTarget()); m) {
    SymbolContextList sc_list;
    sc_list.Append(SymbolContext(std::move(m)));
    options.SetPreferredSymbolContexts(std::move(sc_list));
  }

  ValueObjectSP return_value_sp;
  ExecutionContext exe_ctx;
  frame_sp->CalculateExecutionContext(exe_ctx);
  ExpressionResults result = UserExpression::Evaluate(
      exe_ctx, options, address_sanitizer_retrieve_report_data_command, "",
      return_value_sp);
  if (result != eExpressionCompleted) {
    StreamString ss;
```
- **EN**: Implements logic around `GetPreferredAsanModule`, `Append`, `SetPreferredSymbolContexts`, `CalculateExecutionContext`, and 1 more symbols; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `GetPreferredAsanModule`, `Append`, `SetPreferredSymbolContexts`, `CalculateExecutionContext`, and 1 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 100-107
```cpp
    ss << "cannot evaluate AddressSanitizer expression:\n";
    if (return_value_sp)
      ss << return_value_sp->GetError().AsCString();
    Debugger::ReportWarning(ss.GetString().str(),
                            process_sp->GetTarget().GetDebugger().GetID());
    return StructuredData::ObjectSP();
  }

```
- **EN**: Implements logic around `GetError`, `ReportWarning`, `GetTarget`, `ObjectSP`; this block coordinates debugger runtime objects, events, or asynchronous control flow; supports expression parsing, wrapping, or debug-time code generation.
- **CN**: 围绕 `GetError`, `ReportWarning`, `GetTarget`, `ObjectSP` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并支持表达式解析、包装或调试期代码生成。

### Lines 108-121
```cpp
  int present = return_value_sp->GetValueForExpressionPath(".present")
                    ->GetValueAsUnsigned(0);
  if (present != 1)
    return StructuredData::ObjectSP();

  addr_t pc =
      return_value_sp->GetValueForExpressionPath(".pc")->GetValueAsUnsigned(0);
  addr_t bp =
      return_value_sp->GetValueForExpressionPath(".bp")->GetValueAsUnsigned(0);
  addr_t sp =
      return_value_sp->GetValueForExpressionPath(".sp")->GetValueAsUnsigned(0);
  addr_t address = return_value_sp->GetValueForExpressionPath(".address")
                       ->GetValueAsUnsigned(0);
  addr_t access_type =
```
- **EN**: Implements logic around `GetValueForExpressionPath`, `GetValueAsUnsigned`, `ObjectSP`; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `GetValueForExpressionPath`, `GetValueAsUnsigned`, `ObjectSP` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 122-133
```cpp
      return_value_sp->GetValueForExpressionPath(".access_type")
          ->GetValueAsUnsigned(0);
  addr_t access_size =
      return_value_sp->GetValueForExpressionPath(".access_size")
          ->GetValueAsUnsigned(0);
  addr_t description_ptr =
      return_value_sp->GetValueForExpressionPath(".description")
          ->GetValueAsUnsigned(0);
  std::string description;
  Status error;
  process_sp->ReadCStringFromMemory(description_ptr, description, error);

```
- **EN**: Implements logic around `GetValueForExpressionPath`, `GetValueAsUnsigned`, `ReadCStringFromMemory`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `GetValueForExpressionPath`, `GetValueAsUnsigned`, `ReadCStringFromMemory` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 134-147
```cpp
  auto dict = std::make_shared<StructuredData::Dictionary>();
  if (!dict)
    return StructuredData::ObjectSP();

  dict->AddStringItem("instrumentation_class", "AddressSanitizer");
  dict->AddStringItem("stop_type", "fatal_error");
  dict->AddIntegerItem("pc", pc);
  dict->AddIntegerItem("bp", bp);
  dict->AddIntegerItem("sp", sp);
  dict->AddIntegerItem("address", address);
  dict->AddIntegerItem("access_type", access_type);
  dict->AddIntegerItem("access_size", access_size);
  dict->AddStringItem("description", description);

```
- **EN**: Implements logic around `Dictionary>`, `ObjectSP`, `AddStringItem`, `AddIntegerItem`; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `Dictionary>`, `ObjectSP`, `AddStringItem`, `AddIntegerItem` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 148-161
```cpp
  return StructuredData::ObjectSP(dict);
}

std::string
ReportRetriever::FormatDescription(StructuredData::ObjectSP report) {
  std::string description = std::string(report->GetAsDictionary()
                                            ->GetValueForKey("description")
                                            ->GetAsString()
                                            ->GetValue());
  return llvm::StringSwitch<std::string>(description)
      .Case("heap-use-after-free", "Use of deallocated memory")
      .Case("heap-buffer-overflow", "Heap buffer overflow")
      .Case("stack-buffer-underflow", "Stack buffer underflow")
      .Case("initialization-order-fiasco", "Initialization order problem")
```
- **EN**: Implements logic around `ObjectSP`, `FormatDescription`, `string`, `GetValueForKey`, and 4 more symbols; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `ObjectSP`, `FormatDescription`, `string`, `GetValueForKey`, and 4 more symbols 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 162-175
```cpp
      .Case("stack-buffer-overflow", "Stack buffer overflow")
      .Case("stack-use-after-return", "Use of stack memory after return")
      .Case("use-after-poison", "Use of poisoned memory")
      .Case("container-overflow", "Container overflow")
      .Case("stack-use-after-scope", "Use of out-of-scope stack memory")
      .Case("global-buffer-overflow", "Global buffer overflow")
      .Case("unknown-crash", "Invalid memory access")
      .Case("stack-overflow", "Stack space exhausted")
      .Case("null-deref", "Dereference of null pointer")
      .Case("wild-jump", "Jump to non-executable address")
      .Case("wild-addr-write", "Write through wild pointer")
      .Case("wild-addr-read", "Read from wild pointer")
      .Case("wild-addr", "Access through wild pointer")
      .Case("signal", "Deadly signal")
```
- **EN**: Implements logic around `Case`; this block maps executable state back to modules, symbols, sections, or addresses; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `Case` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 176-189
```cpp
      .Case("double-free", "Deallocation of freed memory")
      .Case("new-delete-type-mismatch",
            "Deallocation size different from allocation size")
      .Case("bad-free", "Deallocation of non-allocated memory")
      .Case("alloc-dealloc-mismatch",
            "Mismatch between allocation and deallocation APIs")
      .Case("bad-malloc_usable_size", "Invalid argument to malloc_usable_size")
      .Case("bad-__sanitizer_get_allocated_size",
            "Invalid argument to __sanitizer_get_allocated_size")
      .Case("param-overlap",
            "Call to function disallowing overlapping memory ranges")
      .Case("negative-size-param", "Negative size used when accessing memory")
      .Case("bad-__sanitizer_annotate_contiguous_container",
            "Invalid argument to __sanitizer_annotate_contiguous_container")
```
- **EN**: Implements logic around `Case`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `Case` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 190-197
```cpp
      .Case("odr-violation", "Symbol defined in multiple translation units")
      .Case(
          "invalid-pointer-pair",
          "Comparison or arithmetic on pointers from different memory regions")
      // for unknown report codes just show the code
      .Default("AddressSanitizer detected: " + description);
}

```
- **EN**: Implements logic around `Case`, `Default`; this block maps executable state back to modules, symbols, sections, or addresses; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `Case`, `Default` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 198-205
```cpp
bool ReportRetriever::NotifyBreakpointHit(ProcessSP process_sp,
                                          StoppointCallbackContext *context,
                                          user_id_t break_id,
                                          user_id_t break_loc_id) {
  // Make sure this is the right process
  if (!process_sp || process_sp != context->exe_ctx_ref.GetProcessSP())
    return false;

```
- **EN**: Implements logic around `NotifyBreakpointHit`, `GetProcessSP`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `NotifyBreakpointHit`, `GetProcessSP` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 206-215
```cpp
  if (process_sp->GetModIDRef().IsLastResumeForUserExpression())
    return false;

  StructuredData::ObjectSP report = RetrieveReportData(process_sp);
  if (!report || report->GetType() != lldb::eStructuredDataTypeDictionary) {
    LLDB_LOGF(GetLog(LLDBLog::InstrumentationRuntime),
              "ReportRetriever::RetrieveReportData() failed");
    return false;
  }

```
- **EN**: Implements logic around `GetModIDRef`, `RetrieveReportData`, `GetType`, `LLDB_LOGF`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `GetModIDRef`, `RetrieveReportData`, `GetType`, `LLDB_LOGF` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 216-222
```cpp
  std::string description = FormatDescription(report);

  if (ThreadSP thread_sp = context->exe_ctx_ref.GetThreadSP())
    thread_sp->SetStopInfo(
        InstrumentationRuntimeStopInfo::CreateStopReasonWithInstrumentationData(
            *thread_sp, description, report));

```
- **EN**: Implements logic around `FormatDescription`, `GetThreadSP`, `SetStopInfo`, `CreateStopReasonWithInstrumentationData`.
- **CN**: 围绕 `FormatDescription`, `GetThreadSP`, `SetStopInfo`, `CreateStopReasonWithInstrumentationData` 实现具体逻辑。

### Lines 223-231
```cpp
  if (StreamSP stream_sp =
          process_sp->GetTarget().GetDebugger().GetAsyncOutputStream())
    stream_sp->Printf("AddressSanitizer report breakpoint hit. Use 'thread "
                      "info -s' to get extended information about the "
                      "report.\n");

  return true; // Return true to stop the target
}

```
- **EN**: Implements logic around `GetTarget`, `Printf`; this block tracks breakpoint state, stop conditions, or hit-processing policy; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetTarget`, `Printf` 实现具体逻辑；该代码块跟踪断点状态、停机条件或命中处理策略，并协调调试器运行时对象、事件或异步控制流。

### Lines 232-240
```cpp
Breakpoint *ReportRetriever::SetupBreakpoint(ModuleSP module_sp,
                                             ProcessSP process_sp,
                                             ConstString symbol_name) {
  if (!module_sp || !process_sp)
    return nullptr;

  const Symbol *symbol =
      module_sp->FindFirstSymbolWithNameAndType(symbol_name, eSymbolTypeCode);

```
- **EN**: Implements logic around `SetupBreakpoint`, `FindFirstSymbolWithNameAndType`; this block tracks breakpoint state, stop conditions, or hit-processing policy; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `SetupBreakpoint`, `FindFirstSymbolWithNameAndType` 实现具体逻辑；该代码块跟踪断点状态、停机条件或命中处理策略，并将可执行状态映射回模块、符号、节区或地址。

### Lines 241-250
```cpp
  if (symbol == nullptr)
    return nullptr;

  if (!symbol->ValueIsAddress() || !symbol->GetAddressRef().IsValid())
    return nullptr;

  const Address &address = symbol->GetAddressRef();
  const bool internal = true;
  const bool hardware = false;

```
- **EN**: Implements logic around `ValueIsAddress`, `GetAddressRef`; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `ValueIsAddress`, `GetAddressRef` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 251-256
```cpp
  Breakpoint *breakpoint = process_sp->GetTarget()
                               .CreateBreakpoint(address, internal, hardware)
                               .get();

  return breakpoint;
}
```
- **EN**: Implements logic around `GetTarget`, `CreateBreakpoint`, `get`; this block tracks breakpoint state, stop conditions, or hit-processing policy; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `GetTarget`, `CreateBreakpoint`, `get` 实现具体逻辑；该代码块跟踪断点状态、停机条件或命中处理策略，并将可执行状态映射回模块、符号、节区或地址。

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

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `ReportRetriever.h`, `Utility.h`, `lldb/Breakpoint/StoppointCallbackContext.h`, `lldb/Core/Debugger.h`, `lldb/Core/Module.h`, `lldb/Expression/UserExpression.h`, `lldb/Target/InstrumentationRuntimeStopInfo.h`, `lldb/ValueObject/ValueObject.h`
- **Subsystem categories / 子系统类别**: LLDB core debugger abstractions / LLDB 核心调试抽象 (2), breakpoint-management infrastructure / 断点管理基础设施 (1), expression parsing and evaluation support / 表达式解析与求值支持 (1), target, process, and thread control / 目标、进程与线程控制 (1), value inspection and presentation helpers / 值检查与展示辅助逻辑 (1)
