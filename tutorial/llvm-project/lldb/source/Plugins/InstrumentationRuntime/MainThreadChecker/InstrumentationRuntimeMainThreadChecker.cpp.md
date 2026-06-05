# InstrumentationRuntimeMainThreadChecker.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/InstrumentationRuntime/MainThreadChecker/InstrumentationRuntimeMainThreadChecker.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `InstrumentationRuntimeMainThreadChecker`.
  - **CN**: 实现与 `InstrumentationRuntimeMainThreadChecker` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- InstrumentationRuntimeMainThreadChecker.cpp -----------------------===//
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

#include "InstrumentationRuntimeMainThreadChecker.h"

#include "Plugins/Process/Utility/HistoryThread.h"
#include "lldb/Breakpoint/StoppointCallbackContext.h"
#include "lldb/Core/Module.h"
#include "lldb/Core/PluginManager.h"
#include "lldb/Symbol/Symbol.h"
#include "lldb/Symbol/SymbolContext.h"
#include "lldb/Symbol/Variable.h"
#include "lldb/Symbol/VariableList.h"
#include "lldb/Target/InstrumentationRuntimeStopInfo.h"
#include "lldb/Target/RegisterContext.h"
#include "lldb/Target/SectionLoadList.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `InstrumentationRuntimeMainThreadChecker.h`, `Plugins/Process/Utility/HistoryThread.h`, `lldb/Breakpoint/StoppointCallbackContext.h`, `lldb/Core/Module.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `InstrumentationRuntimeMainThreadChecker.h`, `Plugins/Process/Utility/HistoryThread.h`, `lldb/Breakpoint/StoppointCallbackContext.h`, `lldb/Core/Module.h`。

### Lines 22-28
```cpp
#include "lldb/Target/StopInfo.h"
#include "lldb/Target/Target.h"
#include "lldb/Target/Thread.h"
#include "lldb/Utility/RegularExpression.h"

#include <memory>

```
- **EN**: Pulls in the headers needed by this translation unit, including `lldb/Target/StopInfo.h`, `lldb/Target/Target.h`, `lldb/Target/Thread.h`, `lldb/Utility/RegularExpression.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `lldb/Target/StopInfo.h`, `lldb/Target/Target.h`, `lldb/Target/Thread.h`, `lldb/Utility/RegularExpression.h`。

### Lines 29-38
```cpp
using namespace lldb;
using namespace lldb_private;

LLDB_PLUGIN_DEFINE(InstrumentationRuntimeMainThreadChecker)

InstrumentationRuntimeMainThreadChecker::
    ~InstrumentationRuntimeMainThreadChecker() {
  Deactivate();
}

```
- **EN**: Implements logic around `LLDB_PLUGIN_DEFINE`, `~InstrumentationRuntimeMainThreadChecker`, `Deactivate`.
- **CN**: 围绕 `LLDB_PLUGIN_DEFINE`, `~InstrumentationRuntimeMainThreadChecker`, `Deactivate` 实现具体逻辑。

### Lines 39-45
```cpp
lldb::InstrumentationRuntimeSP
InstrumentationRuntimeMainThreadChecker::CreateInstance(
    const lldb::ProcessSP &process_sp) {
  return InstrumentationRuntimeSP(
      new InstrumentationRuntimeMainThreadChecker(process_sp));
}

```
- **EN**: Implements logic around `CreateInstance`, `InstrumentationRuntimeSP`, `InstrumentationRuntimeMainThreadChecker`.
- **CN**: 围绕 `CreateInstance`, `InstrumentationRuntimeSP`, `InstrumentationRuntimeMainThreadChecker` 实现具体逻辑。

### Lines 46-52
```cpp
void InstrumentationRuntimeMainThreadChecker::Initialize() {
  PluginManager::RegisterPlugin(
      GetPluginNameStatic(),
      "MainThreadChecker instrumentation runtime plugin.", CreateInstance,
      GetTypeStatic);
}

```
- **EN**: Implements logic around `Initialize`, `RegisterPlugin`, `GetPluginNameStatic`; this block registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `Initialize`, `RegisterPlugin`, `GetPluginNameStatic` 实现具体逻辑；该代码块注册命令、插件或面向解释器的扩展点。

### Lines 53-61
```cpp
void InstrumentationRuntimeMainThreadChecker::Terminate() {
  PluginManager::UnregisterPlugin(CreateInstance);
}

lldb::InstrumentationRuntimeType
InstrumentationRuntimeMainThreadChecker::GetTypeStatic() {
  return eInstrumentationRuntimeTypeMainThreadChecker;
}

```
- **EN**: Implements logic around `Terminate`, `UnregisterPlugin`, `GetTypeStatic`.
- **CN**: 围绕 `Terminate`, `UnregisterPlugin`, `GetTypeStatic` 实现具体逻辑。

### Lines 62-75
```cpp
const RegularExpression &
InstrumentationRuntimeMainThreadChecker::GetPatternForRuntimeLibrary() {
  static RegularExpression regex(llvm::StringRef("libMainThreadChecker.dylib"));
  return regex;
}

bool InstrumentationRuntimeMainThreadChecker::CheckIfRuntimeIsValid(
    const lldb::ModuleSP module_sp) {
  static ConstString test_sym("__main_thread_checker_on_report");
  const Symbol *symbol =
      module_sp->FindFirstSymbolWithNameAndType(test_sym, lldb::eSymbolTypeAny);
  return symbol != nullptr;
}

```
- **EN**: Implements logic around `GetPatternForRuntimeLibrary`, `regex`, `CheckIfRuntimeIsValid`, `test_sym`, and 1 more symbols; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `GetPatternForRuntimeLibrary`, `regex`, `CheckIfRuntimeIsValid`, `test_sym`, and 1 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 76-82
```cpp
StructuredData::ObjectSP
InstrumentationRuntimeMainThreadChecker::RetrieveReportData(
    ExecutionContextRef exe_ctx_ref) {
  ProcessSP process_sp = GetProcessSP();
  if (!process_sp)
    return StructuredData::ObjectSP();

```
- **EN**: Implements logic around `RetrieveReportData`, `GetProcessSP`, `ObjectSP`.
- **CN**: 围绕 `RetrieveReportData`, `GetProcessSP`, `ObjectSP` 实现具体逻辑。

### Lines 83-91
```cpp
  ThreadSP thread_sp = exe_ctx_ref.GetThreadSP();
  StackFrameSP frame_sp =
      thread_sp->GetSelectedFrame(DoNoSelectMostRelevantFrame);
  ModuleSP runtime_module_sp = GetRuntimeModuleSP();
  Target &target = process_sp->GetTarget();

  if (!frame_sp)
    return StructuredData::ObjectSP();

```
- **EN**: Implements logic around `GetThreadSP`, `GetSelectedFrame`, `GetRuntimeModuleSP`, `GetTarget`, and 1 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetThreadSP`, `GetSelectedFrame`, `GetRuntimeModuleSP`, `GetTarget`, and 1 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 92-99
```cpp
  RegisterContextSP regctx_sp = frame_sp->GetRegisterContext();
  if (!regctx_sp)
    return StructuredData::ObjectSP();

  const RegisterInfo *reginfo = regctx_sp->GetRegisterInfoByName("arg1");
  if (!reginfo)
    return StructuredData::ObjectSP();

```
- **EN**: Implements logic around `GetRegisterContext`, `ObjectSP`, `GetRegisterInfoByName`.
- **CN**: 围绕 `GetRegisterContext`, `ObjectSP`, `GetRegisterInfoByName` 实现具体逻辑。

### Lines 100-109
```cpp
  uint64_t apiname_ptr = regctx_sp->ReadRegisterAsUnsigned(reginfo, 0);
  if (!apiname_ptr)
    return StructuredData::ObjectSP();

  std::string apiName;
  Status read_error;
  target.ReadCStringFromMemory(Address(apiname_ptr), apiName, read_error);
  if (read_error.Fail())
    return StructuredData::ObjectSP();

```
- **EN**: Implements logic around `ReadRegisterAsUnsigned`, `ObjectSP`, `ReadCStringFromMemory`, `Fail`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `ReadRegisterAsUnsigned`, `ObjectSP`, `ReadCStringFromMemory`, `Fail` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址。

### Lines 110-119
```cpp
  std::string className;
  std::string selector;
  if (apiName.substr(0, 2) == "-[") {
    size_t spacePos = apiName.find(' ');
    if (spacePos != std::string::npos) {
      className = apiName.substr(2, spacePos - 2);
      selector = apiName.substr(spacePos + 1, apiName.length() - spacePos - 2);
    }
  }

```
- **EN**: Implements logic around `substr`, `find`.
- **CN**: 围绕 `substr`, `find` 实现具体逻辑。

### Lines 120-129
```cpp
  // Gather the PCs of the user frames in the backtrace.
  StructuredData::Array *trace = new StructuredData::Array();
  auto trace_sp = StructuredData::ObjectSP(trace);
  StackFrameSP responsible_frame;
  for (unsigned I = 0; I < thread_sp->GetStackFrameCount(); ++I) {
    StackFrameSP frame = thread_sp->GetStackFrameAtIndex(I);
    Address addr = frame->GetFrameCodeAddressForSymbolication();
    if (addr.GetModule() == runtime_module_sp) // Skip PCs from the runtime.
      continue;

```
- **EN**: Implements logic around `Array`, `ObjectSP`, `GetStackFrameCount`, `GetStackFrameAtIndex`, and 2 more symbols; this block maps executable state back to modules, symbols, sections, or addresses; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `Array`, `ObjectSP`, `GetStackFrameCount`, `GetStackFrameAtIndex`, and 2 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 130-137
```cpp
    // The first non-runtime frame is responsible for the bug.
    if (!responsible_frame)
      responsible_frame = frame;

    lldb::addr_t PC = addr.GetLoadAddress(&target);
    trace->AddIntegerItem(PC);
  }

```
- **EN**: Implements logic around `GetLoadAddress`, `AddIntegerItem`; this block coordinates debugger runtime objects, events, or asynchronous control flow; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetLoadAddress`, `AddIntegerItem` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 138-150
```cpp
  auto *d = new StructuredData::Dictionary();
  auto dict_sp = StructuredData::ObjectSP(d);
  d->AddStringItem("instrumentation_class", "MainThreadChecker");
  d->AddStringItem("api_name", apiName);
  d->AddStringItem("class_name", className);
  d->AddStringItem("selector", selector);
  d->AddStringItem("description",
                   apiName + " must be used from main thread only");
  d->AddIntegerItem("tid", thread_sp->GetIndexID());
  d->AddItem("trace", trace_sp);
  return dict_sp;
}

```
- **EN**: Implements logic around `Dictionary`, `ObjectSP`, `AddStringItem`, `AddIntegerItem`, and 1 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `Dictionary`, `ObjectSP`, `AddStringItem`, `AddIntegerItem`, and 1 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 151-157
```cpp
bool InstrumentationRuntimeMainThreadChecker::NotifyBreakpointHit(
    void *baton, StoppointCallbackContext *context, user_id_t break_id,
    user_id_t break_loc_id) {
  assert(baton && "null baton");
  if (!baton)
    return false; ///< false => resume execution.

```
- **EN**: Implements logic around `NotifyBreakpointHit`, `assert`.
- **CN**: 围绕 `NotifyBreakpointHit`, `assert` 实现具体逻辑。

### Lines 158-166
```cpp
  InstrumentationRuntimeMainThreadChecker *const instance =
      static_cast<InstrumentationRuntimeMainThreadChecker *>(baton);

  ProcessSP process_sp = instance->GetProcessSP();
  ThreadSP thread_sp = context->exe_ctx_ref.GetThreadSP();
  if (!process_sp || !thread_sp ||
      process_sp != context->exe_ctx_ref.GetProcessSP())
    return false;

```
- **EN**: Implements logic around `GetProcessSP`, `GetThreadSP`.
- **CN**: 围绕 `GetProcessSP`, `GetThreadSP` 实现具体逻辑。

### Lines 167-180
```cpp
  if (process_sp->GetModIDRef().IsLastResumeForUserExpression())
    return false;

  StructuredData::ObjectSP report =
      instance->RetrieveReportData(context->exe_ctx_ref);

  if (report) {
    std::string description = std::string(report->GetAsDictionary()
                                              ->GetValueForKey("description")
                                              ->GetAsString()
                                              ->GetValue());
    thread_sp->SetStopInfo(
        InstrumentationRuntimeStopInfo::CreateStopReasonWithInstrumentationData(
            *thread_sp, description, report));
```
- **EN**: Implements logic around `GetModIDRef`, `RetrieveReportData`, `string`, `GetValueForKey`, and 4 more symbols.
- **CN**: 围绕 `GetModIDRef`, `RetrieveReportData`, `string`, `GetValueForKey`, and 4 more symbols 实现具体逻辑。

### Lines 181-190
```cpp
    return true;
  }

  return false;
}

void InstrumentationRuntimeMainThreadChecker::Activate() {
  if (IsActive())
    return;

```
- **EN**: Implements logic around `Activate`, `IsActive`.
- **CN**: 围绕 `Activate`, `IsActive` 实现具体逻辑。

### Lines 191-200
```cpp
  ProcessSP process_sp = GetProcessSP();
  if (!process_sp)
    return;

  ModuleSP runtime_module_sp = GetRuntimeModuleSP();

  ConstString symbol_name("__main_thread_checker_on_report");
  const Symbol *symbol = runtime_module_sp->FindFirstSymbolWithNameAndType(
      symbol_name, eSymbolTypeCode);

```
- **EN**: Implements logic around `GetProcessSP`, `GetRuntimeModuleSP`, `symbol_name`, `FindFirstSymbolWithNameAndType`; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `GetProcessSP`, `GetRuntimeModuleSP`, `symbol_name`, `FindFirstSymbolWithNameAndType` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 201-209
```cpp
  if (symbol == nullptr)
    return;

  if (!symbol->ValueIsAddress() || !symbol->GetAddressRef().IsValid())
    return;

  Target &target = process_sp->GetTarget();
  addr_t symbol_address = symbol->GetAddressRef().GetOpcodeLoadAddress(&target);

```
- **EN**: Implements logic around `ValueIsAddress`, `GetTarget`, `GetAddressRef`; this block coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `ValueIsAddress`, `GetTarget`, `GetAddressRef` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址。

### Lines 210-223
```cpp
  if (symbol_address == LLDB_INVALID_ADDRESS)
    return;

  Breakpoint *breakpoint =
      process_sp->GetTarget()
          .CreateBreakpoint(symbol_address, /*internal=*/true,
                            /*hardware=*/false)
          .get();
  const bool sync = false;
  breakpoint->SetCallback(
      InstrumentationRuntimeMainThreadChecker::NotifyBreakpointHit, this, sync);
  breakpoint->SetBreakpointKind("main-thread-checker-report");
  SetBreakpointID(breakpoint->GetID());

```
- **EN**: Implements logic around `GetTarget`, `CreateBreakpoint`, `get`, `SetCallback`, and 2 more symbols; this block tracks breakpoint state, stop conditions, or hit-processing policy; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetTarget`, `CreateBreakpoint`, `get`, `SetCallback`, and 2 more symbols 实现具体逻辑；该代码块跟踪断点状态、停机条件或命中处理策略，并协调调试器运行时对象、事件或异步控制流。

### Lines 224-233
```cpp
  SetActive(true);
}

void InstrumentationRuntimeMainThreadChecker::Deactivate() {
  SetActive(false);

  auto BID = GetBreakpointID();
  if (BID == LLDB_INVALID_BREAK_ID)
    return;

```
- **EN**: Implements logic around `SetActive`, `Deactivate`, `GetBreakpointID`.
- **CN**: 围绕 `SetActive`, `Deactivate`, `GetBreakpointID` 实现具体逻辑。

### Lines 234-245
```cpp
  if (ProcessSP process_sp = GetProcessSP()) {
    process_sp->GetTarget().RemoveBreakpointByID(BID);
    SetBreakpointID(LLDB_INVALID_BREAK_ID);
  }
}

lldb::ThreadCollectionSP
InstrumentationRuntimeMainThreadChecker::GetBacktracesFromExtendedStopInfo(
    StructuredData::ObjectSP info) {
  ThreadCollectionSP threads;
  threads = std::make_shared<ThreadCollection>();

```
- **EN**: Implements logic around `GetProcessSP`, `GetTarget`, `SetBreakpointID`, `GetBacktracesFromExtendedStopInfo`, and 1 more symbols.
- **CN**: 围绕 `GetProcessSP`, `GetTarget`, `SetBreakpointID`, `GetBacktracesFromExtendedStopInfo`, and 1 more symbols 实现具体逻辑。

### Lines 246-258
```cpp
  ProcessSP process_sp = GetProcessSP();

  if (info->GetObjectForDotSeparatedPath("instrumentation_class")
          ->GetStringValue() != "MainThreadChecker")
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

### Lines 259-266
```cpp
  if (PCs.empty())
    return threads;

  StructuredData::ObjectSP thread_id_obj =
      info->GetObjectForDotSeparatedPath("tid");
  lldb::tid_t tid =
      thread_id_obj ? thread_id_obj->GetUnsignedIntegerValue() : 0;

```
- **EN**: Implements logic around `empty`, `GetObjectForDotSeparatedPath`, `GetUnsignedIntegerValue`.
- **CN**: 围绕 `empty`, `GetObjectForDotSeparatedPath`, `GetUnsignedIntegerValue` 实现具体逻辑。

### Lines 267-277
```cpp
  // We gather symbolication addresses above, so no need for HistoryThread to
  // try to infer the call addresses.
  auto pc_type = HistoryPCType::Calls;
  ThreadSP new_thread_sp =
      std::make_shared<HistoryThread>(*process_sp, tid, PCs, pc_type);

  // Save this in the Process' ExtendedThreadList so a strong pointer retains
  // the object
  process_sp->GetExtendedThreadList().AddThread(new_thread_sp);
  threads->AddThread(new_thread_sp);

```
- **EN**: Implements logic around `make_shared`, `GetExtendedThreadList`, `AddThread`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `make_shared`, `GetExtendedThreadList`, `AddThread` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 278-279
```cpp
  return threads;
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
- **User expressions / 用户表达式**:
  - **EN**: Wraps and evaluates source snippets inside the debuggee context.
  - **CN**: 在被调试进程上下文中包装并求值源码片段。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `InstrumentationRuntimeMainThreadChecker.h`, `Plugins/Process/Utility/HistoryThread.h`, `lldb/Breakpoint/StoppointCallbackContext.h`, `lldb/Core/Module.h`, `lldb/Core/PluginManager.h`, `lldb/Symbol/Symbol.h`, `lldb/Symbol/SymbolContext.h`, `lldb/Symbol/Variable.h`, `lldb/Symbol/VariableList.h`, `lldb/Target/InstrumentationRuntimeStopInfo.h` ... (+6 more)
- **Standard-library headers / 标准库头文件**: `<memory>`
- **Subsystem categories / 子系统类别**: target, process, and thread control / 目标、进程与线程控制 (6), symbol and debug-info abstractions / 符号与调试信息抽象 (4), LLDB core debugger abstractions / LLDB 核心调试抽象 (2), breakpoint-management infrastructure / 断点管理基础设施 (1), shared LLDB utility classes / 共享 LLDB 工具类 (1)
