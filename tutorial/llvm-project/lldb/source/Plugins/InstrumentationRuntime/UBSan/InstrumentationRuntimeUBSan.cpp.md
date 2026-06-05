# InstrumentationRuntimeUBSan.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/InstrumentationRuntime/UBSan/InstrumentationRuntimeUBSan.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `InstrumentationRuntimeUBSan`.
  - **CN**: 实现与 `InstrumentationRuntimeUBSan` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```cpp
//===-- InstrumentationRuntimeUBSan.cpp -----------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "InstrumentationRuntimeUBSan.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `InstrumentationRuntimeUBSan.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `InstrumentationRuntimeUBSan.h`。

### Lines 11-30
```cpp
#include "Plugins/Process/Utility/HistoryThread.h"
#include "lldb/Breakpoint/StoppointCallbackContext.h"
#include "lldb/Core/Debugger.h"
#include "lldb/Core/Module.h"
#include "lldb/Core/PluginInterface.h"
#include "lldb/Core/PluginManager.h"
#include "lldb/Expression/UserExpression.h"
#include "lldb/Host/StreamFile.h"
#include "lldb/Interpreter/CommandReturnObject.h"
#include "lldb/Symbol/Symbol.h"
#include "lldb/Symbol/SymbolContext.h"
#include "lldb/Symbol/Variable.h"
#include "lldb/Symbol/VariableList.h"
#include "lldb/Target/InstrumentationRuntimeStopInfo.h"
#include "lldb/Target/SectionLoadList.h"
#include "lldb/Target/StopInfo.h"
#include "lldb/Target/Target.h"
#include "lldb/Target/Thread.h"
#include "lldb/Utility/RegularExpression.h"
#include "lldb/Utility/Stream.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `Plugins/Process/Utility/HistoryThread.h`, `lldb/Breakpoint/StoppointCallbackContext.h`, `lldb/Core/Debugger.h`, `lldb/Core/Module.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `Plugins/Process/Utility/HistoryThread.h`, `lldb/Breakpoint/StoppointCallbackContext.h`, `lldb/Core/Debugger.h`, `lldb/Core/Module.h`。

### Lines 31-40
```cpp
#include "lldb/ValueObject/ValueObject.h"
#include <cctype>

#include <memory>

using namespace lldb;
using namespace lldb_private;

LLDB_PLUGIN_DEFINE(InstrumentationRuntimeUBSan)

```
- **EN**: Pulls in the headers needed by this translation unit, including `lldb/ValueObject/ValueObject.h`, `cctype`, `memory`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `lldb/ValueObject/ValueObject.h`, `cctype`, `memory`。

### Lines 41-54
```cpp
InstrumentationRuntimeUBSan::~InstrumentationRuntimeUBSan() { Deactivate(); }

lldb::InstrumentationRuntimeSP
InstrumentationRuntimeUBSan::CreateInstance(const lldb::ProcessSP &process_sp) {
  return InstrumentationRuntimeSP(new InstrumentationRuntimeUBSan(process_sp));
}

void InstrumentationRuntimeUBSan::Initialize() {
  PluginManager::RegisterPlugin(
      GetPluginNameStatic(),
      "UndefinedBehaviorSanitizer instrumentation runtime plugin.",
      CreateInstance, GetTypeStatic);
}

```
- **EN**: Implements logic around `~InstrumentationRuntimeUBSan`, `CreateInstance`, `InstrumentationRuntimeSP`, `Initialize`, and 2 more symbols; this block registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `~InstrumentationRuntimeUBSan`, `CreateInstance`, `InstrumentationRuntimeSP`, `Initialize`, and 2 more symbols 实现具体逻辑；该代码块注册命令、插件或面向解释器的扩展点。

### Lines 55-71
```cpp
void InstrumentationRuntimeUBSan::Terminate() {
  PluginManager::UnregisterPlugin(CreateInstance);
}

lldb::InstrumentationRuntimeType InstrumentationRuntimeUBSan::GetTypeStatic() {
  return eInstrumentationRuntimeTypeUndefinedBehaviorSanitizer;
}

static const char *ub_sanitizer_retrieve_report_data_prefix = R"(
extern "C" {
void
__ubsan_get_current_report_data(const char **OutIssueKind,
    const char **OutMessage, const char **OutFilename, unsigned *OutLine,
    unsigned *OutCol, char **OutMemoryAddr);
}
)";

```
- **EN**: Implements logic around `Terminate`, `UnregisterPlugin`, `GetTypeStatic`, `__ubsan_get_current_report_data`.
- **CN**: 围绕 `Terminate`, `UnregisterPlugin`, `GetTypeStatic`, `__ubsan_get_current_report_data` 实现具体逻辑。

### Lines 72-81
```cpp
static const char *ub_sanitizer_retrieve_report_data_command = R"(
struct {
  const char *issue_kind;
  const char *message;
  const char *filename;
  unsigned line;
  unsigned col;
  char *memory_addr;
} t;

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 82-93
```cpp
__ubsan_get_current_report_data(&t.issue_kind, &t.message, &t.filename, &t.line,
                                &t.col, &t.memory_addr);
t;
)";

static addr_t RetrieveUnsigned(ValueObjectSP return_value_sp,
                               ProcessSP process_sp,
                               const std::string &expression_path) {
  return return_value_sp->GetValueForExpressionPath(expression_path.c_str())
      ->GetValueAsUnsigned(0);
}

```
- **EN**: Implements logic around `__ubsan_get_current_report_data`, `RetrieveUnsigned`, `GetValueForExpressionPath`, `GetValueAsUnsigned`.
- **CN**: 围绕 `__ubsan_get_current_report_data`, `RetrieveUnsigned`, `GetValueForExpressionPath`, `GetValueAsUnsigned` 实现具体逻辑。

### Lines 94-103
```cpp
static std::string RetrieveString(ValueObjectSP return_value_sp,
                                  ProcessSP process_sp,
                                  const std::string &expression_path) {
  addr_t ptr = RetrieveUnsigned(return_value_sp, process_sp, expression_path);
  std::string str;
  Status error;
  process_sp->ReadCStringFromMemory(ptr, str, error);
  return str;
}

```
- **EN**: Implements logic around `RetrieveString`, `RetrieveUnsigned`, `ReadCStringFromMemory`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `RetrieveString`, `RetrieveUnsigned`, `ReadCStringFromMemory` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 104-115
```cpp
StructuredData::ObjectSP InstrumentationRuntimeUBSan::RetrieveReportData(
    ExecutionContextRef exe_ctx_ref) {
  ProcessSP process_sp = GetProcessSP();
  if (!process_sp)
    return StructuredData::ObjectSP();

  ThreadSP thread_sp = exe_ctx_ref.GetThreadSP();
  StackFrameSP frame_sp =
      thread_sp->GetSelectedFrame(DoNoSelectMostRelevantFrame);
  ModuleSP runtime_module_sp = GetRuntimeModuleSP();
  Target &target = process_sp->GetTarget();

```
- **EN**: Implements logic around `RetrieveReportData`, `GetProcessSP`, `ObjectSP`, `GetThreadSP`, and 3 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `RetrieveReportData`, `GetProcessSP`, `ObjectSP`, `GetThreadSP`, and 3 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 116-128
```cpp
  if (!frame_sp)
    return StructuredData::ObjectSP();

  EvaluateExpressionOptions options;
  options.SetUnwindOnError(true);
  options.SetTryAllThreads(true);
  options.SetStopOthers(true);
  options.SetIgnoreBreakpoints(true);
  options.SetTimeout(process_sp->GetUtilityExpressionTimeout());
  options.SetPrefix(ub_sanitizer_retrieve_report_data_prefix);
  options.SetAutoApplyFixIts(false);
  options.SetLanguage(eLanguageTypeC);

```
- **EN**: Implements logic around `ObjectSP`, `SetUnwindOnError`, `SetTryAllThreads`, `SetStopOthers`, and 5 more symbols.
- **CN**: 围绕 `ObjectSP`, `SetUnwindOnError`, `SetTryAllThreads`, `SetStopOthers`, and 5 more symbols 实现具体逻辑。

### Lines 129-144
```cpp
  ValueObjectSP main_value;
  ExecutionContext exe_ctx;
  frame_sp->CalculateExecutionContext(exe_ctx);
  ExpressionResults result = UserExpression::Evaluate(
      exe_ctx, options, ub_sanitizer_retrieve_report_data_command, "",
      main_value);
  if (result != eExpressionCompleted) {
    StreamString ss;
    ss << "cannot evaluate UndefinedBehaviorSanitizer expression:\n";
    if (main_value)
      ss << main_value->GetError().AsCString();
    Debugger::ReportWarning(ss.GetString().str(),
                            process_sp->GetTarget().GetDebugger().GetID());
    return StructuredData::ObjectSP();
  }

```
- **EN**: Implements logic around `CalculateExecutionContext`, `Evaluate`, `GetError`, `ReportWarning`, and 2 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; supports expression parsing, wrapping, or debug-time code generation.
- **CN**: 围绕 `CalculateExecutionContext`, `Evaluate`, `GetError`, `ReportWarning`, and 2 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并支持表达式解析、包装或调试期代码生成。

### Lines 145-157
```cpp
  // Gather the PCs of the user frames in the backtrace.
  StructuredData::Array *trace = new StructuredData::Array();
  auto trace_sp = StructuredData::ObjectSP(trace);
  for (unsigned I = 0; I < thread_sp->GetStackFrameCount(); ++I) {
    const Address FCA = thread_sp->GetStackFrameAtIndex(I)
                            ->GetFrameCodeAddressForSymbolication();
    if (FCA.GetModule() == runtime_module_sp) // Skip PCs from the runtime.
      continue;

    lldb::addr_t PC = FCA.GetLoadAddress(&target);
    trace->AddIntegerItem(PC);
  }

```
- **EN**: Implements logic around `Array`, `ObjectSP`, `GetStackFrameCount`, `GetStackFrameAtIndex`, and 4 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `Array`, `ObjectSP`, `GetStackFrameCount`, `GetStackFrameAtIndex`, and 4 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址。

### Lines 158-177
```cpp
  std::string IssueKind = RetrieveString(main_value, process_sp, ".issue_kind");
  std::string ErrMessage = RetrieveString(main_value, process_sp, ".message");
  std::string Filename = RetrieveString(main_value, process_sp, ".filename");
  unsigned Line = RetrieveUnsigned(main_value, process_sp, ".line");
  unsigned Col = RetrieveUnsigned(main_value, process_sp, ".col");
  uintptr_t MemoryAddr =
      RetrieveUnsigned(main_value, process_sp, ".memory_addr");

  auto *d = new StructuredData::Dictionary();
  auto dict_sp = StructuredData::ObjectSP(d);
  d->AddStringItem("instrumentation_class", "UndefinedBehaviorSanitizer");
  d->AddStringItem("description", IssueKind);
  d->AddStringItem("summary", ErrMessage);
  d->AddStringItem("filename", Filename);
  d->AddIntegerItem("line", Line);
  d->AddIntegerItem("col", Col);
  d->AddIntegerItem("memory_address", MemoryAddr);
  d->AddIntegerItem("tid", thread_sp->GetID());
  d->AddItem("trace", trace_sp);
  return dict_sp;
```
- **EN**: Implements logic around `RetrieveString`, `RetrieveUnsigned`, `Dictionary`, `ObjectSP`, and 3 more symbols; this block controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `RetrieveString`, `RetrieveUnsigned`, `Dictionary`, `ObjectSP`, and 3 more symbols 实现具体逻辑；该代码块控制调试器侧的值格式化或 synthetic 展示。

### Lines 178-197
```cpp
}

static std::string GetStopReasonDescription(StructuredData::ObjectSP report) {
  llvm::StringRef stop_reason_description_ref;
  report->GetAsDictionary()->GetValueForKeyAsString(
      "description", stop_reason_description_ref);
  std::string stop_reason_description =
      std::string(stop_reason_description_ref);

  if (!stop_reason_description.size()) {
    stop_reason_description = "Undefined behavior detected";
  } else {
    stop_reason_description[0] = toupper(stop_reason_description[0]);
    for (unsigned I = 1; I < stop_reason_description.size(); ++I)
      if (stop_reason_description[I] == '-')
        stop_reason_description[I] = ' ';
  }
  return stop_reason_description;
}

```
- **EN**: Implements logic around `GetStopReasonDescription`, `GetAsDictionary`, `string`, `size`, and 1 more symbols.
- **CN**: 围绕 `GetStopReasonDescription`, `GetAsDictionary`, `string`, `size`, and 1 more symbols 实现具体逻辑。

### Lines 198-207
```cpp
bool InstrumentationRuntimeUBSan::NotifyBreakpointHit(
    void *baton, StoppointCallbackContext *context, user_id_t break_id,
    user_id_t break_loc_id) {
  assert(baton && "null baton");
  if (!baton)
    return false; ///< false => resume execution.

  InstrumentationRuntimeUBSan *const instance =
      static_cast<InstrumentationRuntimeUBSan *>(baton);

```
- **EN**: Implements logic around `NotifyBreakpointHit`, `assert`.
- **CN**: 围绕 `NotifyBreakpointHit`, `assert` 实现具体逻辑。

### Lines 208-219
```cpp
  ProcessSP process_sp = instance->GetProcessSP();
  ThreadSP thread_sp = context->exe_ctx_ref.GetThreadSP();
  if (!process_sp || !thread_sp ||
      process_sp != context->exe_ctx_ref.GetProcessSP())
    return false;

  if (process_sp->GetModIDRef().IsLastResumeForUserExpression())
    return false;

  StructuredData::ObjectSP report =
      instance->RetrieveReportData(context->exe_ctx_ref);

```
- **EN**: Implements logic around `GetProcessSP`, `GetThreadSP`, `GetModIDRef`, `RetrieveReportData`.
- **CN**: 围绕 `GetProcessSP`, `GetThreadSP`, `GetModIDRef`, `RetrieveReportData` 实现具体逻辑。

### Lines 220-229
```cpp
  if (report) {
    thread_sp->SetStopInfo(
        InstrumentationRuntimeStopInfo::CreateStopReasonWithInstrumentationData(
            *thread_sp, GetStopReasonDescription(report), report));
    return true;
  }

  return false;
}

```
- **EN**: Implements logic around `SetStopInfo`, `CreateStopReasonWithInstrumentationData`.
- **CN**: 围绕 `SetStopInfo`, `CreateStopReasonWithInstrumentationData` 实现具体逻辑。

### Lines 230-243
```cpp
const RegularExpression &
InstrumentationRuntimeUBSan::GetPatternForRuntimeLibrary() {
  static RegularExpression regex(llvm::StringRef("libclang_rt\\.(a|t|ub)san_"));
  return regex;
}

bool InstrumentationRuntimeUBSan::CheckIfRuntimeIsValid(
    const lldb::ModuleSP module_sp) {
  static ConstString ubsan_test_sym("__ubsan_on_report");
  const Symbol *symbol = module_sp->FindFirstSymbolWithNameAndType(
      ubsan_test_sym, lldb::eSymbolTypeAny);
  return symbol != nullptr;
}

```
- **EN**: Implements logic around `GetPatternForRuntimeLibrary`, `regex`, `CheckIfRuntimeIsValid`, `ubsan_test_sym`, and 1 more symbols; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `GetPatternForRuntimeLibrary`, `regex`, `CheckIfRuntimeIsValid`, `ubsan_test_sym`, and 1 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 244-254
```cpp
// FIXME: Factor out all the logic we have in common with the {a,t}san plugins.
void InstrumentationRuntimeUBSan::Activate() {
  if (IsActive())
    return;

  ProcessSP process_sp = GetProcessSP();
  if (!process_sp)
    return;

  ModuleSP runtime_module_sp = GetRuntimeModuleSP();

```
- **EN**: Implements logic around `Activate`, `IsActive`, `GetProcessSP`, `GetRuntimeModuleSP`.
- **CN**: 围绕 `Activate`, `IsActive`, `GetProcessSP`, `GetRuntimeModuleSP` 实现具体逻辑。

### Lines 255-264
```cpp
  ConstString symbol_name("__ubsan_on_report");
  const Symbol *symbol = runtime_module_sp->FindFirstSymbolWithNameAndType(
      symbol_name, eSymbolTypeCode);

  if (symbol == nullptr)
    return;

  if (!symbol->ValueIsAddress() || !symbol->GetAddressRef().IsValid())
    return;

```
- **EN**: Implements logic around `symbol_name`, `FindFirstSymbolWithNameAndType`, `ValueIsAddress`; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `symbol_name`, `FindFirstSymbolWithNameAndType`, `ValueIsAddress` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 265-281
```cpp
  Target &target = process_sp->GetTarget();
  addr_t symbol_address = symbol->GetAddressRef().GetOpcodeLoadAddress(&target);

  if (symbol_address == LLDB_INVALID_ADDRESS)
    return;

  Breakpoint *breakpoint =
      process_sp->GetTarget()
          .CreateBreakpoint(symbol_address, /*internal=*/true,
                            /*hardware=*/false)
          .get();
  const bool sync = false;
  breakpoint->SetCallback(InstrumentationRuntimeUBSan::NotifyBreakpointHit,
                          this, sync);
  breakpoint->SetBreakpointKind("undefined-behavior-sanitizer-report");
  SetBreakpointID(breakpoint->GetID());

```
- **EN**: Implements logic around `GetTarget`, `GetAddressRef`, `CreateBreakpoint`, `get`, and 3 more symbols; this block tracks breakpoint state, stop conditions, or hit-processing policy; coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `GetTarget`, `GetAddressRef`, `CreateBreakpoint`, `get`, and 3 more symbols 实现具体逻辑；该代码块跟踪断点状态、停机条件或命中处理策略，并协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址。

### Lines 282-291
```cpp
  SetActive(true);
}

void InstrumentationRuntimeUBSan::Deactivate() {
  SetActive(false);

  auto BID = GetBreakpointID();
  if (BID == LLDB_INVALID_BREAK_ID)
    return;

```
- **EN**: Implements logic around `SetActive`, `Deactivate`, `GetBreakpointID`.
- **CN**: 围绕 `SetActive`, `Deactivate`, `GetBreakpointID` 实现具体逻辑。

### Lines 292-303
```cpp
  if (ProcessSP process_sp = GetProcessSP()) {
    process_sp->GetTarget().RemoveBreakpointByID(BID);
    SetBreakpointID(LLDB_INVALID_BREAK_ID);
  }
}

lldb::ThreadCollectionSP
InstrumentationRuntimeUBSan::GetBacktracesFromExtendedStopInfo(
    StructuredData::ObjectSP info) {
  ThreadCollectionSP threads;
  threads = std::make_shared<ThreadCollection>();

```
- **EN**: Implements logic around `GetProcessSP`, `GetTarget`, `SetBreakpointID`, `GetBacktracesFromExtendedStopInfo`, and 1 more symbols.
- **CN**: 围绕 `GetProcessSP`, `GetTarget`, `SetBreakpointID`, `GetBacktracesFromExtendedStopInfo`, and 1 more symbols 实现具体逻辑。

### Lines 304-316
```cpp
  ProcessSP process_sp = GetProcessSP();

  if (info->GetObjectForDotSeparatedPath("instrumentation_class")
          ->GetStringValue() != "UndefinedBehaviorSanitizer")
    return threads;

  std::vector<lldb::addr_t> PCs;
  auto trace = info->GetObjectForDotSeparatedPath("trace")->GetAsArray();
  trace->ForEach([&PCs](StructuredData::Object *PC) -> bool {
    PCs.push_back(PC->GetUnsignedIntegerValue());
    return true;
  });

```
- **EN**: Implements logic around `GetProcessSP`, `GetObjectForDotSeparatedPath`, `GetStringValue`, `ForEach`, and 1 more symbols.
- **CN**: 围绕 `GetProcessSP`, `GetObjectForDotSeparatedPath`, `GetStringValue`, `ForEach`, and 1 more symbols 实现具体逻辑。

### Lines 317-332
```cpp
  if (PCs.empty())
    return threads;

  StructuredData::ObjectSP thread_id_obj =
      info->GetObjectForDotSeparatedPath("tid");
  lldb::tid_t tid =
      thread_id_obj ? thread_id_obj->GetUnsignedIntegerValue() : 0;

  // We gather symbolication addresses above, so no need for HistoryThread to
  // try to infer the call addresses.
  auto pc_type = HistoryPCType::Calls;
  ThreadSP new_thread_sp =
      std::make_shared<HistoryThread>(*process_sp, tid, PCs, pc_type);
  std::string stop_reason_description = GetStopReasonDescription(info);
  new_thread_sp->SetName(stop_reason_description.c_str());

```
- **EN**: Implements logic around `empty`, `GetObjectForDotSeparatedPath`, `GetUnsignedIntegerValue`, `make_shared`, and 2 more symbols.
- **CN**: 围绕 `empty`, `GetObjectForDotSeparatedPath`, `GetUnsignedIntegerValue`, `make_shared`, and 2 more symbols 实现具体逻辑。

### Lines 333-339
```cpp
  // Save this in the Process' ExtendedThreadList so a strong pointer retains
  // the object
  process_sp->GetExtendedThreadList().AddThread(new_thread_sp);
  threads->AddThread(new_thread_sp);

  return threads;
}
```
- **EN**: Implements logic around `GetExtendedThreadList`, `AddThread`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetExtendedThreadList`, `AddThread` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

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
- **OS integration / 操作系统集成**:
  - **EN**: Bridges LLDB abstractions to host files, sockets, terminals, and native process facilities.
  - **CN**: 将 LLDB 抽象连接到主机文件、套接字、终端与本地进程设施。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `InstrumentationRuntimeUBSan.h`, `Plugins/Process/Utility/HistoryThread.h`, `lldb/Breakpoint/StoppointCallbackContext.h`, `lldb/Core/Debugger.h`, `lldb/Core/Module.h`, `lldb/Core/PluginInterface.h`, `lldb/Core/PluginManager.h`, `lldb/Expression/UserExpression.h`, `lldb/Host/StreamFile.h`, `lldb/Interpreter/CommandReturnObject.h` ... (+12 more)
- **Standard-library headers / 标准库头文件**: `<cctype>`, `<memory>`
- **Subsystem categories / 子系统类别**: target, process, and thread control / 目标、进程与线程控制 (5), LLDB core debugger abstractions / LLDB 核心调试抽象 (4), symbol and debug-info abstractions / 符号与调试信息抽象 (4), shared LLDB utility classes / 共享 LLDB 工具类 (2), breakpoint-management infrastructure / 断点管理基础设施 (1), expression parsing and evaluation support / 表达式解析与求值支持 (1), host OS abstraction helpers / 主机操作系统抽象辅助组件 (1), command interpreter support / 命令解释器支持 (1)
