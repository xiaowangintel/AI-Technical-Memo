# InstrumentationRuntimeTSan.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/InstrumentationRuntime/TSan/InstrumentationRuntimeTSan.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `InstrumentationRuntimeTSan`.
  - **CN**: 实现与 `InstrumentationRuntimeTSan` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28
```cpp
//===-- InstrumentationRuntimeTSan.cpp ------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "InstrumentationRuntimeTSan.h"

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
```
- **EN**: Pulls in the headers needed by this translation unit, including `InstrumentationRuntimeTSan.h`, `Plugins/Process/Utility/HistoryThread.h`, `lldb/Breakpoint/StoppointCallbackContext.h`, `lldb/Core/Debugger.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `InstrumentationRuntimeTSan.h`, `Plugins/Process/Utility/HistoryThread.h`, `lldb/Breakpoint/StoppointCallbackContext.h`, `lldb/Core/Debugger.h`。

### Lines 29-46
```cpp
#include "lldb/Utility/LLDBLog.h"
#include "lldb/Utility/Log.h"
#include "lldb/Utility/RegularExpression.h"
#include "lldb/Utility/Stream.h"
#include "lldb/ValueObject/ValueObject.h"

#include <memory>

using namespace lldb;
using namespace lldb_private;

LLDB_PLUGIN_DEFINE(InstrumentationRuntimeTSan)

lldb::InstrumentationRuntimeSP
InstrumentationRuntimeTSan::CreateInstance(const lldb::ProcessSP &process_sp) {
  return InstrumentationRuntimeSP(new InstrumentationRuntimeTSan(process_sp));
}

```
- **EN**: Pulls in the headers needed by this translation unit, including `lldb/Utility/LLDBLog.h`, `lldb/Utility/Log.h`, `lldb/Utility/RegularExpression.h`, `lldb/Utility/Stream.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `lldb/Utility/LLDBLog.h`, `lldb/Utility/Log.h`, `lldb/Utility/RegularExpression.h`, `lldb/Utility/Stream.h`。

### Lines 47-60
```cpp
void InstrumentationRuntimeTSan::Initialize() {
  PluginManager::RegisterPlugin(
      GetPluginNameStatic(), "ThreadSanitizer instrumentation runtime plugin.",
      CreateInstance, GetTypeStatic);
}

void InstrumentationRuntimeTSan::Terminate() {
  PluginManager::UnregisterPlugin(CreateInstance);
}

lldb::InstrumentationRuntimeType InstrumentationRuntimeTSan::GetTypeStatic() {
  return eInstrumentationRuntimeTypeThreadSanitizer;
}

```
- **EN**: Implements logic around `Initialize`, `RegisterPlugin`, `GetPluginNameStatic`, `Terminate`, and 2 more symbols; this block registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `Initialize`, `RegisterPlugin`, `GetPluginNameStatic`, `Terminate`, and 2 more symbols 实现具体逻辑；该代码块注册命令、插件或面向解释器的扩展点。

### Lines 61-87
```cpp
InstrumentationRuntimeTSan::~InstrumentationRuntimeTSan() { Deactivate(); }

const char *thread_sanitizer_retrieve_report_data_prefix = R"(
extern "C"
{
    void *__tsan_get_current_report();
    int __tsan_get_report_data(void *report, const char **description, int *count,
                               int *stack_count, int *mop_count, int *loc_count,
                               int *mutex_count, int *thread_count,
                               int *unique_tid_count, void **sleep_trace,
                               unsigned long trace_size);
    int __tsan_get_report_stack(void *report, unsigned long idx, void **trace,
                                unsigned long trace_size);
    int __tsan_get_report_mop(void *report, unsigned long idx, int *tid, void **addr,
                              int *size, int *write, int *atomic, void **trace,
                              unsigned long trace_size);
    int __tsan_get_report_loc(void *report, unsigned long idx, const char **type,
                              void **addr, unsigned long *start, unsigned long *size, int *tid,
                              int *fd, int *suppressable, void **trace,
                              unsigned long trace_size);
    int __tsan_get_report_mutex(void *report, unsigned long idx, unsigned long *mutex_id, void **addr,
                                int *destroyed, void **trace, unsigned long trace_size);
    int __tsan_get_report_thread(void *report, unsigned long idx, int *tid, unsigned long *os_id,
                                 int *running, const char **name, int *parent_tid,
                                 void **trace, unsigned long trace_size);
    int __tsan_get_report_unique_tid(void *report, unsigned long idx, int *tid);

```
- **EN**: Implements logic around `~InstrumentationRuntimeTSan`, `__tsan_get_current_report`, `__tsan_get_report_data`, `__tsan_get_report_stack`, and 5 more symbols.
- **CN**: 围绕 `~InstrumentationRuntimeTSan`, `__tsan_get_current_report`, `__tsan_get_report_data`, `__tsan_get_report_stack`, and 5 more symbols 实现具体逻辑。

### Lines 88-103
```cpp
    // TODO: dlsym won't work on Windows.
    void *dlsym(void* handle, const char* symbol);
    int (*ptr__tsan_get_report_loc_object_type)(void *report, unsigned long idx, const char **object_type);
#if defined(__linux__)
#define RTLD_DEFAULT	((void *) 0)
#else
#define RTLD_DEFAULT	((void *) -2)
#endif
  }
)";

const char *thread_sanitizer_retrieve_report_data_command = R"(

const int REPORT_TRACE_SIZE = 128;
const int REPORT_ARRAY_SIZE = 4;

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

### Lines 104-118
```cpp
struct {
    void *report;
    const char *description;
    int report_count;

    void *sleep_trace[REPORT_TRACE_SIZE];

    int stack_count;
    struct {
        int idx;
        void *trace[REPORT_TRACE_SIZE];
    } stacks[REPORT_ARRAY_SIZE];

    int mop_count;
    struct {
```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 119-141
```cpp
        int idx;
        int tid;
        int size;
        int write;
        int atomic;
        void *addr;
        void *trace[REPORT_TRACE_SIZE];
    } mops[REPORT_ARRAY_SIZE];

    int loc_count;
    struct {
        int idx;
        const char *type;
        void *addr;
        unsigned long start;
        unsigned long size;
        int tid;
        int fd;
        int suppressable;
        void *trace[REPORT_TRACE_SIZE];
        const char *object_type;
    } locs[REPORT_ARRAY_SIZE];

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 142-161
```cpp
    int mutex_count;
    struct {
        int idx;
        unsigned long mutex_id;
        void *addr;
        int destroyed;
        void *trace[REPORT_TRACE_SIZE];
    } mutexes[REPORT_ARRAY_SIZE];

    int thread_count;
    struct {
        int idx;
        int tid;
        unsigned long os_id;
        int running;
        const char *name;
        int parent_tid;
        void *trace[REPORT_TRACE_SIZE];
    } threads[REPORT_ARRAY_SIZE];

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 162-179
```cpp
    int unique_tid_count;
    struct {
        int idx;
        int tid;
    } unique_tids[REPORT_ARRAY_SIZE];
} t = {0};

ptr__tsan_get_report_loc_object_type = (typeof(ptr__tsan_get_report_loc_object_type))(void *)dlsym(RTLD_DEFAULT, "__tsan_get_report_loc_object_type");

t.report = __tsan_get_current_report();
__tsan_get_report_data(t.report, &t.description, &t.report_count, &t.stack_count, &t.mop_count, &t.loc_count, &t.mutex_count, &t.thread_count, &t.unique_tid_count, t.sleep_trace, REPORT_TRACE_SIZE);

if (t.stack_count > REPORT_ARRAY_SIZE) t.stack_count = REPORT_ARRAY_SIZE;
for (int i = 0; i < t.stack_count; i++) {
    t.stacks[i].idx = i;
    __tsan_get_report_stack(t.report, i, t.stacks[i].trace, REPORT_TRACE_SIZE);
}

```
- **EN**: Implements logic around `typeof`, `__tsan_get_current_report`, `__tsan_get_report_data`, `__tsan_get_report_stack`.
- **CN**: 围绕 `typeof`, `__tsan_get_current_report`, `__tsan_get_report_data`, `__tsan_get_report_stack` 实现具体逻辑。

### Lines 180-193
```cpp
if (t.mop_count > REPORT_ARRAY_SIZE) t.mop_count = REPORT_ARRAY_SIZE;
for (int i = 0; i < t.mop_count; i++) {
    t.mops[i].idx = i;
    __tsan_get_report_mop(t.report, i, &t.mops[i].tid, &t.mops[i].addr, &t.mops[i].size, &t.mops[i].write, &t.mops[i].atomic, t.mops[i].trace, REPORT_TRACE_SIZE);
}

if (t.loc_count > REPORT_ARRAY_SIZE) t.loc_count = REPORT_ARRAY_SIZE;
for (int i = 0; i < t.loc_count; i++) {
    t.locs[i].idx = i;
    __tsan_get_report_loc(t.report, i, &t.locs[i].type, &t.locs[i].addr, &t.locs[i].start, &t.locs[i].size, &t.locs[i].tid, &t.locs[i].fd, &t.locs[i].suppressable, t.locs[i].trace, REPORT_TRACE_SIZE);
    if (ptr__tsan_get_report_loc_object_type)
        ptr__tsan_get_report_loc_object_type(t.report, i, &t.locs[i].object_type);
}

```
- **EN**: Implements logic around `__tsan_get_report_mop`, `__tsan_get_report_loc`, `ptr__tsan_get_report_loc_object_type`.
- **CN**: 围绕 `__tsan_get_report_mop`, `__tsan_get_report_loc`, `ptr__tsan_get_report_loc_object_type` 实现具体逻辑。

### Lines 194-211
```cpp
if (t.mutex_count > REPORT_ARRAY_SIZE) t.mutex_count = REPORT_ARRAY_SIZE;
for (int i = 0; i < t.mutex_count; i++) {
    t.mutexes[i].idx = i;
    __tsan_get_report_mutex(t.report, i, &t.mutexes[i].mutex_id, &t.mutexes[i].addr, &t.mutexes[i].destroyed, t.mutexes[i].trace, REPORT_TRACE_SIZE);
}

if (t.thread_count > REPORT_ARRAY_SIZE) t.thread_count = REPORT_ARRAY_SIZE;
for (int i = 0; i < t.thread_count; i++) {
    t.threads[i].idx = i;
    __tsan_get_report_thread(t.report, i, &t.threads[i].tid, &t.threads[i].os_id, &t.threads[i].running, &t.threads[i].name, &t.threads[i].parent_tid, t.threads[i].trace, REPORT_TRACE_SIZE);
}

if (t.unique_tid_count > REPORT_ARRAY_SIZE) t.unique_tid_count = REPORT_ARRAY_SIZE;
for (int i = 0; i < t.unique_tid_count; i++) {
    t.unique_tids[i].idx = i;
    __tsan_get_report_unique_tid(t.report, i, &t.unique_tids[i].tid);
}

```
- **EN**: Implements logic around `__tsan_get_report_mutex`, `__tsan_get_report_thread`, `__tsan_get_report_unique_tid`.
- **CN**: 围绕 `__tsan_get_report_mutex`, `__tsan_get_report_thread`, `__tsan_get_report_unique_tid` 实现具体逻辑。

### Lines 212-231
```cpp
t;
)";

static StructuredData::ArraySP
CreateStackTrace(ValueObjectSP o,
                 const std::string &trace_item_name = ".trace") {
  auto trace_sp = std::make_shared<StructuredData::Array>();
  ValueObjectSP trace_value_object =
      o->GetValueForExpressionPath(trace_item_name.c_str());
  size_t count = trace_value_object->GetNumChildrenIgnoringErrors();
  for (size_t j = 0; j < count; j++) {
    addr_t trace_addr =
        trace_value_object->GetChildAtIndex(j)->GetValueAsUnsigned(0);
    if (trace_addr == 0)
      break;
    trace_sp->AddIntegerItem(trace_addr);
  }
  return trace_sp;
}

```
- **EN**: Implements logic around `CreateStackTrace`, `Array>`, `GetValueForExpressionPath`, `GetNumChildrenIgnoringErrors`, and 2 more symbols.
- **CN**: 围绕 `CreateStackTrace`, `Array>`, `GetValueForExpressionPath`, `GetNumChildrenIgnoringErrors`, and 2 more symbols 实现具体逻辑。

### Lines 232-247
```cpp
static StructuredData::ArraySP ConvertToStructuredArray(
    ValueObjectSP return_value_sp, const std::string &items_name,
    const std::string &count_name,
    std::function<void(const ValueObjectSP &o,
                       const StructuredData::DictionarySP &dict)> const
        &callback) {
  auto array_sp = std::make_shared<StructuredData::Array>();
  unsigned int count =
      return_value_sp->GetValueForExpressionPath(count_name.c_str())
          ->GetValueAsUnsigned(0);
  ValueObjectSP objects =
      return_value_sp->GetValueForExpressionPath(items_name.c_str());
  for (unsigned int i = 0; i < count; i++) {
    ValueObjectSP o = objects->GetChildAtIndex(i);
    auto dict_sp = std::make_shared<StructuredData::Dictionary>();

```
- **EN**: Implements logic around `ConvertToStructuredArray`, `function`, `Array>`, `GetValueForExpressionPath`, and 3 more symbols.
- **CN**: 围绕 `ConvertToStructuredArray`, `function`, `Array>`, `GetValueForExpressionPath`, and 3 more symbols 实现具体逻辑。

### Lines 248-266
```cpp
    callback(o, dict_sp);

    array_sp->AddItem(dict_sp);
  }
  return array_sp;
}

static std::string RetrieveString(ValueObjectSP return_value_sp,
                                  ProcessSP process_sp,
                                  const std::string &expression_path) {
  addr_t ptr =
      return_value_sp->GetValueForExpressionPath(expression_path.c_str())
          ->GetValueAsUnsigned(0);
  std::string str;
  Status error;
  process_sp->ReadCStringFromMemory(ptr, str, error);
  return str;
}

```
- **EN**: Implements logic around `callback`, `AddItem`, `RetrieveString`, `GetValueForExpressionPath`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `callback`, `AddItem`, `RetrieveString`, `GetValueForExpressionPath`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 267-292
```cpp
static void
GetRenumberedThreadIds(ProcessSP process_sp, ValueObjectSP data,
                       std::map<uint64_t, user_id_t> &thread_id_map) {
  ConvertToStructuredArray(
      data, ".threads", ".thread_count",
      [process_sp, &thread_id_map](const ValueObjectSP &o,
                                   const StructuredData::DictionarySP &dict) {
        uint64_t thread_id =
            o->GetValueForExpressionPath(".tid")->GetValueAsUnsigned(0);
        uint64_t thread_os_id =
            o->GetValueForExpressionPath(".os_id")->GetValueAsUnsigned(0);
        user_id_t lldb_user_id = 0;

        bool can_update = true;
        ThreadSP lldb_thread = process_sp->GetThreadList().FindThreadByID(
            thread_os_id, can_update);
        if (lldb_thread) {
          lldb_user_id = lldb_thread->GetIndexID();
        } else {
          // This isn't a live thread anymore.  Ask process to assign a new
          // Index ID (or return an old one if we've already seen this
          // thread_os_id). It will also make sure that no new threads are
          // assigned this Index ID.
          lldb_user_id = process_sp->AssignIndexIDToThread(thread_os_id);
        }

```
- **EN**: Implements logic around `GetRenumberedThreadIds`, `ConvertToStructuredArray`, `GetValueForExpressionPath`, `GetThreadList`, and 2 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetRenumberedThreadIds`, `ConvertToStructuredArray`, `GetValueForExpressionPath`, `GetThreadList`, and 2 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 293-311
```cpp
        thread_id_map[thread_id] = lldb_user_id;
      });
}

static user_id_t Renumber(uint64_t id,
                          std::map<uint64_t, user_id_t> &thread_id_map) {
  auto IT = thread_id_map.find(id);
  if (IT == thread_id_map.end())
    return 0;

  return IT->second;
}

StructuredData::ObjectSP InstrumentationRuntimeTSan::RetrieveReportData(
    ExecutionContextRef exe_ctx_ref) {
  ProcessSP process_sp = GetProcessSP();
  if (!process_sp)
    return StructuredData::ObjectSP();

```
- **EN**: Implements logic around `Renumber`, `find`, `end`, `RetrieveReportData`, and 2 more symbols.
- **CN**: 围绕 `Renumber`, `find`, `end`, `RetrieveReportData`, and 2 more symbols 实现具体逻辑。

### Lines 312-328
```cpp
  ThreadSP thread_sp = exe_ctx_ref.GetThreadSP();
  StackFrameSP frame_sp =
      thread_sp->GetSelectedFrame(DoNoSelectMostRelevantFrame);

  if (!frame_sp)
    return StructuredData::ObjectSP();

  EvaluateExpressionOptions options;
  options.SetUnwindOnError(true);
  options.SetTryAllThreads(true);
  options.SetStopOthers(true);
  options.SetIgnoreBreakpoints(true);
  options.SetTimeout(process_sp->GetUtilityExpressionTimeout());
  options.SetPrefix(thread_sanitizer_retrieve_report_data_prefix);
  options.SetAutoApplyFixIts(false);
  options.SetLanguage(eLanguageTypeC);

```
- **EN**: Implements logic around `GetThreadSP`, `GetSelectedFrame`, `ObjectSP`, `SetUnwindOnError`, and 7 more symbols.
- **CN**: 围绕 `GetThreadSP`, `GetSelectedFrame`, `ObjectSP`, `SetUnwindOnError`, and 7 more symbols 实现具体逻辑。

### Lines 329-344
```cpp
  ValueObjectSP main_value;
  ExecutionContext exe_ctx;
  frame_sp->CalculateExecutionContext(exe_ctx);
  ExpressionResults result = UserExpression::Evaluate(
      exe_ctx, options, thread_sanitizer_retrieve_report_data_command, "",
      main_value);
  if (result != eExpressionCompleted) {
    StreamString ss;
    ss << "cannot evaluate ThreadSanitizer expression:\n";
    if (main_value)
      ss << main_value->GetError().AsCString();
    Debugger::ReportWarning(ss.GetString().str(),
                            process_sp->GetTarget().GetDebugger().GetID());
    return StructuredData::ObjectSP();
  }

```
- **EN**: Implements logic around `CalculateExecutionContext`, `Evaluate`, `GetError`, `ReportWarning`, and 2 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; supports expression parsing, wrapping, or debug-time code generation.
- **CN**: 围绕 `CalculateExecutionContext`, `Evaluate`, `GetError`, `ReportWarning`, and 2 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并支持表达式解析、包装或调试期代码生成。

### Lines 345-370
```cpp
  std::map<uint64_t, user_id_t> thread_id_map;
  GetRenumberedThreadIds(process_sp, main_value, thread_id_map);

  auto dict = std::make_shared<StructuredData::Dictionary>();
  dict->AddStringItem("instrumentation_class", "ThreadSanitizer");
  dict->AddStringItem("issue_type",
                      RetrieveString(main_value, process_sp, ".description"));
  dict->AddIntegerItem("report_count",
                       main_value->GetValueForExpressionPath(".report_count")
                           ->GetValueAsUnsigned(0));
  dict->AddItem("sleep_trace", CreateStackTrace(
                                   main_value, ".sleep_trace"));

  StructuredData::ArraySP stacks = ConvertToStructuredArray(
      main_value, ".stacks", ".stack_count",
      [thread_sp](const ValueObjectSP &o,
                  const StructuredData::DictionarySP &dict) {
        dict->AddIntegerItem(
            "index",
            o->GetValueForExpressionPath(".idx")->GetValueAsUnsigned(0));
        dict->AddItem("trace", CreateStackTrace(o));
        // "stacks" happen on the current thread
        dict->AddIntegerItem("thread_id", thread_sp->GetIndexID());
      });
  dict->AddItem("stacks", stacks);

```
- **EN**: Implements logic around `GetRenumberedThreadIds`, `Dictionary>`, `AddStringItem`, `RetrieveString`, and 5 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetRenumberedThreadIds`, `Dictionary>`, `AddStringItem`, `RetrieveString`, and 5 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 371-398
```cpp
  StructuredData::ArraySP mops = ConvertToStructuredArray(
      main_value, ".mops", ".mop_count",
      [&thread_id_map](const ValueObjectSP &o,
                       const StructuredData::DictionarySP &dict) {
        dict->AddIntegerItem(
            "index",
            o->GetValueForExpressionPath(".idx")->GetValueAsUnsigned(0));
        dict->AddIntegerItem(
            "thread_id",
            Renumber(
                o->GetValueForExpressionPath(".tid")->GetValueAsUnsigned(0),
                thread_id_map));
        dict->AddIntegerItem(
            "size",
            o->GetValueForExpressionPath(".size")->GetValueAsUnsigned(0));
        dict->AddBooleanItem(
            "is_write",
            o->GetValueForExpressionPath(".write")->GetValueAsUnsigned(0));
        dict->AddBooleanItem(
            "is_atomic",
            o->GetValueForExpressionPath(".atomic")->GetValueAsUnsigned(0));
        dict->AddIntegerItem(
            "address",
            o->GetValueForExpressionPath(".addr")->GetValueAsUnsigned(0));
        dict->AddItem("trace", CreateStackTrace(o));
      });
  dict->AddItem("mops", mops);

```
- **EN**: Implements logic around `ConvertToStructuredArray`, `AddIntegerItem`, `GetValueForExpressionPath`, `Renumber`, and 2 more symbols; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `ConvertToStructuredArray`, `AddIntegerItem`, `GetValueForExpressionPath`, `Renumber`, and 2 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 399-426
```cpp
  StructuredData::ArraySP locs = ConvertToStructuredArray(
      main_value, ".locs", ".loc_count",
      [process_sp, &thread_id_map](const ValueObjectSP &o,
                                   const StructuredData::DictionarySP &dict) {
        dict->AddIntegerItem(
            "index",
            o->GetValueForExpressionPath(".idx")->GetValueAsUnsigned(0));
        dict->AddStringItem("type", RetrieveString(o, process_sp, ".type"));
        dict->AddIntegerItem(
            "address",
            o->GetValueForExpressionPath(".addr")->GetValueAsUnsigned(0));
        dict->AddIntegerItem(
            "start",
            o->GetValueForExpressionPath(".start")->GetValueAsUnsigned(0));
        dict->AddIntegerItem(
            "size",
            o->GetValueForExpressionPath(".size")->GetValueAsUnsigned(0));
        dict->AddIntegerItem(
            "thread_id",
            Renumber(
                o->GetValueForExpressionPath(".tid")->GetValueAsUnsigned(0),
                thread_id_map));
        dict->AddIntegerItem(
            "file_descriptor",
            o->GetValueForExpressionPath(".fd")->GetValueAsUnsigned(0));
        dict->AddIntegerItem("suppressable",
                             o->GetValueForExpressionPath(".suppressable")
                                 ->GetValueAsUnsigned(0));
```
- **EN**: Implements logic around `ConvertToStructuredArray`, `AddIntegerItem`, `GetValueForExpressionPath`, `AddStringItem`, and 2 more symbols; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `ConvertToStructuredArray`, `AddIntegerItem`, `GetValueForExpressionPath`, `AddStringItem`, and 2 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 427-451
```cpp
        dict->AddItem("trace", CreateStackTrace(o));
        dict->AddStringItem("object_type",
                            RetrieveString(o, process_sp, ".object_type"));
      });
  dict->AddItem("locs", locs);

  StructuredData::ArraySP mutexes = ConvertToStructuredArray(
      main_value, ".mutexes", ".mutex_count",
      [](const ValueObjectSP &o, const StructuredData::DictionarySP &dict) {
        dict->AddIntegerItem(
            "index",
            o->GetValueForExpressionPath(".idx")->GetValueAsUnsigned(0));
        dict->AddIntegerItem(
            "mutex_id",
            o->GetValueForExpressionPath(".mutex_id")->GetValueAsUnsigned(0));
        dict->AddIntegerItem(
            "address",
            o->GetValueForExpressionPath(".addr")->GetValueAsUnsigned(0));
        dict->AddIntegerItem(
            "destroyed",
            o->GetValueForExpressionPath(".destroyed")->GetValueAsUnsigned(0));
        dict->AddItem("trace", CreateStackTrace(o));
      });
  dict->AddItem("mutexes", mutexes);

```
- **EN**: Implements logic around `AddItem`, `AddStringItem`, `RetrieveString`, `ConvertToStructuredArray`, and 2 more symbols; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `AddItem`, `AddStringItem`, `RetrieveString`, `ConvertToStructuredArray`, and 2 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 452-479
```cpp
  StructuredData::ArraySP threads = ConvertToStructuredArray(
      main_value, ".threads", ".thread_count",
      [process_sp, &thread_id_map](const ValueObjectSP &o,
                                   const StructuredData::DictionarySP &dict) {
        dict->AddIntegerItem(
            "index",
            o->GetValueForExpressionPath(".idx")->GetValueAsUnsigned(0));
        dict->AddIntegerItem(
            "thread_id",
            Renumber(
                o->GetValueForExpressionPath(".tid")->GetValueAsUnsigned(0),
                thread_id_map));
        dict->AddIntegerItem(
            "thread_os_id",
            o->GetValueForExpressionPath(".os_id")->GetValueAsUnsigned(0));
        dict->AddIntegerItem(
            "running",
            o->GetValueForExpressionPath(".running")->GetValueAsUnsigned(0));
        dict->AddStringItem("name", RetrieveString(o, process_sp, ".name"));
        dict->AddIntegerItem(
            "parent_thread_id",
            Renumber(o->GetValueForExpressionPath(".parent_tid")
                         ->GetValueAsUnsigned(0),
                     thread_id_map));
        dict->AddItem("trace", CreateStackTrace(o));
      });
  dict->AddItem("threads", threads);

```
- **EN**: Implements logic around `ConvertToStructuredArray`, `AddIntegerItem`, `GetValueForExpressionPath`, `Renumber`, and 3 more symbols.
- **CN**: 围绕 `ConvertToStructuredArray`, `AddIntegerItem`, `GetValueForExpressionPath`, `Renumber`, and 3 more symbols 实现具体逻辑。

### Lines 480-494
```cpp
  StructuredData::ArraySP unique_tids = ConvertToStructuredArray(
      main_value, ".unique_tids", ".unique_tid_count",
      [&thread_id_map](const ValueObjectSP &o,
                       const StructuredData::DictionarySP &dict) {
        dict->AddIntegerItem(
            "index",
            o->GetValueForExpressionPath(".idx")->GetValueAsUnsigned(0));
        dict->AddIntegerItem(
            "tid",
            Renumber(
                o->GetValueForExpressionPath(".tid")->GetValueAsUnsigned(0),
                thread_id_map));
      });
  dict->AddItem("unique_tids", unique_tids);

```
- **EN**: Implements logic around `ConvertToStructuredArray`, `AddIntegerItem`, `GetValueForExpressionPath`, `Renumber`, and 1 more symbols.
- **CN**: 围绕 `ConvertToStructuredArray`, `AddIntegerItem`, `GetValueForExpressionPath`, `Renumber`, and 1 more symbols 实现具体逻辑。

### Lines 495-522
```cpp
  return dict;
}

std::string
InstrumentationRuntimeTSan::FormatDescription(StructuredData::ObjectSP report) {
  std::string description = std::string(report->GetAsDictionary()
                                            ->GetValueForKey("issue_type")
                                            ->GetAsString()
                                            ->GetValue());

  if (description == "data-race") {
    return "Data race";
  } else if (description == "data-race-vptr") {
    return "Data race on C++ virtual pointer";
  } else if (description == "heap-use-after-free") {
    return "Use of deallocated memory";
  } else if (description == "heap-use-after-free-vptr") {
    return "Use of deallocated C++ virtual pointer";
  } else if (description == "thread-leak") {
    return "Thread leak";
  } else if (description == "locked-mutex-destroy") {
    return "Destruction of a locked mutex";
  } else if (description == "mutex-double-lock") {
    return "Double lock of a mutex";
  } else if (description == "mutex-invalid-access") {
    return "Use of an uninitialized or destroyed mutex";
  } else if (description == "mutex-bad-unlock") {
    return "Unlock of an unlocked mutex (or by a wrong thread)";
```
- **EN**: Implements logic around `FormatDescription`, `string`, `GetValueForKey`, `GetAsString`, and 2 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `FormatDescription`, `string`, `GetValueForKey`, `GetAsString`, and 2 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 523-538
```cpp
  } else if (description == "mutex-bad-read-lock") {
    return "Read lock of a write locked mutex";
  } else if (description == "mutex-bad-read-unlock") {
    return "Read unlock of a write locked mutex";
  } else if (description == "signal-unsafe-call") {
    return "Signal-unsafe call inside a signal handler";
  } else if (description == "errno-in-signal-handler") {
    return "Overwrite of errno in a signal handler";
  } else if (description == "lock-order-inversion") {
    return "Lock order inversion (potential deadlock)";
  } else if (description == "external-race") {
    return "Race on a library object";
  } else if (description == "swift-access-race") {
    return "Swift access race";
  }

```
- **EN**: Implements logic around `inversion`.
- **CN**: 围绕 `inversion` 实现具体逻辑。

### Lines 539-556
```cpp
  // for unknown report codes just show the code
  return description;
}

static std::string Sprintf(const char *format, ...) {
  StreamString s;
  va_list args;
  va_start(args, format);
  s.PrintfVarArg(format, args);
  va_end(args);
  return std::string(s.GetString());
}

static std::string GetSymbolNameFromAddress(ProcessSP process_sp, addr_t addr) {
  lldb_private::Address so_addr;
  if (!process_sp->GetTarget().ResolveLoadAddress(addr, so_addr))
    return "";

```
- **EN**: Implements logic around `Sprintf`, `va_start`, `PrintfVarArg`, `va_end`, and 3 more symbols; this block maps executable state back to modules, symbols, sections, or addresses; controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `Sprintf`, `va_start`, `PrintfVarArg`, `va_end`, and 3 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并控制调试器侧的值格式化或 synthetic 展示。

### Lines 557-570
```cpp
  const lldb_private::Symbol *symbol = so_addr.CalculateSymbolContextSymbol();
  if (!symbol)
    return "";

  std::string sym_name = symbol->GetName().GetCString();
  return sym_name;
}

static void GetSymbolDeclarationFromAddress(ProcessSP process_sp, addr_t addr,
                                            Declaration &decl) {
  lldb_private::Address so_addr;
  if (!process_sp->GetTarget().ResolveLoadAddress(addr, so_addr))
    return;

```
- **EN**: Implements logic around `CalculateSymbolContextSymbol`, `GetName`, `GetSymbolDeclarationFromAddress`, `GetTarget`; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `CalculateSymbolContextSymbol`, `GetName`, `GetSymbolDeclarationFromAddress`, `GetTarget` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 571-585
```cpp
  lldb_private::Symbol *symbol = so_addr.CalculateSymbolContextSymbol();
  if (!symbol)
    return;

  ConstString sym_name = symbol->GetMangled().GetName(Mangled::ePreferMangled);

  ModuleSP module = symbol->CalculateSymbolContextModule();
  if (!module)
    return;

  VariableList var_list;
  module->FindGlobalVariables(sym_name, CompilerDeclContext(), 1U, var_list);
  if (var_list.GetSize() < 1)
    return;

```
- **EN**: Implements logic around `CalculateSymbolContextSymbol`, `GetMangled`, `CalculateSymbolContextModule`, `FindGlobalVariables`, and 1 more symbols; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `CalculateSymbolContextSymbol`, `GetMangled`, `CalculateSymbolContextModule`, `FindGlobalVariables`, and 1 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 586-599
```cpp
  VariableSP var = var_list.GetVariableAtIndex(0);
  decl = var->GetDeclaration();
}

addr_t InstrumentationRuntimeTSan::GetFirstNonInternalFramePc(
    StructuredData::ObjectSP trace, bool skip_one_frame) {
  ProcessSP process_sp = GetProcessSP();
  ModuleSP runtime_module_sp = GetRuntimeModuleSP();

  StructuredData::Array *trace_array = trace->GetAsArray();
  for (size_t i = 0; i < trace_array->GetSize(); i++) {
    if (skip_one_frame && i == 0)
      continue;

```
- **EN**: Implements logic around `GetVariableAtIndex`, `GetDeclaration`, `GetFirstNonInternalFramePc`, `GetProcessSP`, and 3 more symbols.
- **CN**: 围绕 `GetVariableAtIndex`, `GetDeclaration`, `GetFirstNonInternalFramePc`, `GetProcessSP`, and 3 more symbols 实现具体逻辑。

### Lines 600-614
```cpp
    auto maybe_addr = trace_array->GetItemAtIndexAsInteger<addr_t>(i);
    if (!maybe_addr)
      continue;
    addr_t addr = *maybe_addr;

    lldb_private::Address so_addr;
    if (!process_sp->GetTarget().ResolveLoadAddress(addr, so_addr))
      continue;

    if (so_addr.GetModule() == runtime_module_sp)
      continue;

    return addr;
  }

```
- **EN**: Implements logic around `GetItemAtIndexAsInteger`, `GetTarget`, `GetModule`; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `GetItemAtIndexAsInteger`, `GetTarget`, `GetModule` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 615-629
```cpp
  return 0;
}

std::string
InstrumentationRuntimeTSan::GenerateSummary(StructuredData::ObjectSP report) {
  ProcessSP process_sp = GetProcessSP();

  std::string summary = std::string(report->GetAsDictionary()
                                        ->GetValueForKey("description")
                                        ->GetAsString()
                                        ->GetValue());
  bool skip_one_frame =
      report->GetObjectForDotSeparatedPath("issue_type")->GetStringValue() ==
      "external-race";

```
- **EN**: Implements logic around `GenerateSummary`, `GetProcessSP`, `string`, `GetValueForKey`, and 3 more symbols; this block controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `GenerateSummary`, `GetProcessSP`, `string`, `GetValueForKey`, and 3 more symbols 实现具体逻辑；该代码块控制调试器侧的值格式化或 synthetic 展示。

### Lines 630-654
```cpp
  addr_t pc = 0;
  if (report->GetAsDictionary()
          ->GetValueForKey("mops")
          ->GetAsArray()
          ->GetSize() > 0)
    pc = GetFirstNonInternalFramePc(report->GetAsDictionary()
                                        ->GetValueForKey("mops")
                                        ->GetAsArray()
                                        ->GetItemAtIndex(0)
                                        ->GetAsDictionary()
                                        ->GetValueForKey("trace"),
                                    skip_one_frame);

  if (report->GetAsDictionary()
          ->GetValueForKey("stacks")
          ->GetAsArray()
          ->GetSize() > 0)
    pc = GetFirstNonInternalFramePc(report->GetAsDictionary()
                                        ->GetValueForKey("stacks")
                                        ->GetAsArray()
                                        ->GetItemAtIndex(0)
                                        ->GetAsDictionary()
                                        ->GetValueForKey("trace"),
                                    skip_one_frame);

```
- **EN**: Implements logic around `GetAsDictionary`, `GetValueForKey`, `GetAsArray`, `GetSize`, and 2 more symbols.
- **CN**: 围绕 `GetAsDictionary`, `GetValueForKey`, `GetAsArray`, `GetSize`, and 2 more symbols 实现具体逻辑。

### Lines 655-681
```cpp
  if (pc != 0) {
    summary = summary + " in " + GetSymbolNameFromAddress(process_sp, pc);
  }

  if (report->GetAsDictionary()
          ->GetValueForKey("locs")
          ->GetAsArray()
          ->GetSize() > 0) {
    StructuredData::ObjectSP loc = report->GetAsDictionary()
                                       ->GetValueForKey("locs")
                                       ->GetAsArray()
                                       ->GetItemAtIndex(0);
    std::string object_type = std::string(loc->GetAsDictionary()
                                              ->GetValueForKey("object_type")
                                              ->GetAsString()
                                              ->GetValue());
    if (!object_type.empty()) {
      summary = "Race on " + object_type + " object";
    }
    addr_t addr = loc->GetAsDictionary()
                      ->GetValueForKey("address")
                      ->GetUnsignedIntegerValue();
    if (addr == 0)
      addr = loc->GetAsDictionary()
                 ->GetValueForKey("start")
                 ->GetUnsignedIntegerValue();

```
- **EN**: Implements logic around `GetSymbolNameFromAddress`, `GetAsDictionary`, `GetValueForKey`, `GetAsArray`, and 7 more symbols; this block maps executable state back to modules, symbols, sections, or addresses; controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `GetSymbolNameFromAddress`, `GetAsDictionary`, `GetValueForKey`, `GetAsArray`, and 7 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并控制调试器侧的值格式化或 synthetic 展示。

### Lines 682-698
```cpp
    if (addr != 0) {
      std::string global_name = GetSymbolNameFromAddress(process_sp, addr);
      if (!global_name.empty()) {
        summary = summary + " at " + global_name;
      } else {
        summary = summary + " at " + Sprintf("0x%llx", addr);
      }
    } else {
      int fd = loc->GetAsDictionary()
                   ->GetValueForKey("file_descriptor")
                   ->GetSignedIntegerValue();
      if (fd != 0) {
        summary = summary + " on file descriptor " + Sprintf("%d", fd);
      }
    }
  }

```
- **EN**: Implements logic around `GetSymbolNameFromAddress`, `empty`, `Sprintf`, `GetAsDictionary`, and 2 more symbols; this block controls debugger-side formatting or synthetic presentation of values; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `GetSymbolNameFromAddress`, `empty`, `Sprintf`, `GetAsDictionary`, and 2 more symbols 实现具体逻辑；该代码块控制调试器侧的值格式化或 synthetic 展示，并抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 699-714
```cpp
  return summary;
}

addr_t InstrumentationRuntimeTSan::GetMainRacyAddress(
    StructuredData::ObjectSP report) {
  addr_t result = (addr_t)-1;

  report->GetObjectForDotSeparatedPath("mops")->GetAsArray()->ForEach(
      [&result](StructuredData::Object *o) -> bool {
        addr_t addr = o->GetObjectForDotSeparatedPath("address")
                          ->GetUnsignedIntegerValue();
        if (addr < result)
          result = addr;
        return true;
      });

```
- **EN**: Implements logic around `GetMainRacyAddress`, `GetObjectForDotSeparatedPath`, `GetUnsignedIntegerValue`; this block maps executable state back to modules, symbols, sections, or addresses; controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `GetMainRacyAddress`, `GetObjectForDotSeparatedPath`, `GetUnsignedIntegerValue` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并控制调试器侧的值格式化或 synthetic 展示。

### Lines 715-739
```cpp
  return (result == (addr_t)-1) ? 0 : result;
}

std::string InstrumentationRuntimeTSan::GetLocationDescription(
    StructuredData::ObjectSP report, addr_t &global_addr,
    std::string &global_name, std::string &filename, uint32_t &line) {
  std::string result;

  ProcessSP process_sp = GetProcessSP();

  if (report->GetAsDictionary()
          ->GetValueForKey("locs")
          ->GetAsArray()
          ->GetSize() > 0) {
    StructuredData::ObjectSP loc = report->GetAsDictionary()
                                       ->GetValueForKey("locs")
                                       ->GetAsArray()
                                       ->GetItemAtIndex(0);
    std::string type = std::string(
        loc->GetAsDictionary()->GetValueForKey("type")->GetStringValue());
    if (type == "global") {
      global_addr = loc->GetAsDictionary()
                        ->GetValueForKey("address")
                        ->GetUnsignedIntegerValue();

```
- **EN**: Implements logic around `GetLocationDescription`, `GetProcessSP`, `GetAsDictionary`, `GetValueForKey`, and 5 more symbols; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `GetLocationDescription`, `GetProcessSP`, `GetAsDictionary`, `GetValueForKey`, and 5 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 740-758
```cpp
      global_name = GetSymbolNameFromAddress(process_sp, global_addr);
      if (!global_name.empty()) {
        result = Sprintf("'%s' is a global variable (0x%llx)",
                         global_name.c_str(), global_addr);
      } else {
        result = Sprintf("0x%llx is a global variable", global_addr);
      }

      Declaration decl;
      GetSymbolDeclarationFromAddress(process_sp, global_addr, decl);
      if (decl.GetFile()) {
        filename = decl.GetFile().GetPath();
        line = decl.GetLine();
      }
    } else if (type == "heap") {
      addr_t addr = loc->GetAsDictionary()
                        ->GetValueForKey("start")
                        ->GetUnsignedIntegerValue();

```
- **EN**: Implements logic around `GetSymbolNameFromAddress`, `empty`, `Sprintf`, `c_str`, and 6 more symbols.
- **CN**: 围绕 `GetSymbolNameFromAddress`, `empty`, `Sprintf`, `c_str`, and 6 more symbols 实现具体逻辑。

### Lines 759-778
```cpp
      size_t size = loc->GetAsDictionary()
                        ->GetValueForKey("size")
                        ->GetUnsignedIntegerValue();

      std::string object_type = std::string(loc->GetAsDictionary()
                                                ->GetValueForKey("object_type")
                                                ->GetAsString()
                                                ->GetValue());
      if (!object_type.empty()) {
        result = Sprintf("Location is a %ld-byte %s object at 0x%llx", size,
                         object_type.c_str(), addr);
      } else {
        result =
            Sprintf("Location is a %ld-byte heap object at 0x%llx", size, addr);
      }
    } else if (type == "stack") {
      lldb::tid_t tid = loc->GetAsDictionary()
                            ->GetValueForKey("thread_id")
                            ->GetUnsignedIntegerValue();

```
- **EN**: Implements logic around `GetAsDictionary`, `GetValueForKey`, `GetUnsignedIntegerValue`, `string`, and 5 more symbols; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetAsDictionary`, `GetValueForKey`, `GetUnsignedIntegerValue`, `string`, and 5 more symbols 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 779-794
```cpp
      result = Sprintf("Location is stack of thread %d", tid);
    } else if (type == "tls") {
      lldb::tid_t tid = loc->GetAsDictionary()
                            ->GetValueForKey("thread_id")
                            ->GetUnsignedIntegerValue();

      result = Sprintf("Location is TLS of thread %d", tid);
    } else if (type == "fd") {
      int fd = loc->GetAsDictionary()
                   ->GetValueForKey("file_descriptor")
                   ->GetSignedIntegerValue();

      result = Sprintf("Location is file descriptor %d", fd);
    }
  }

```
- **EN**: Implements logic around `Sprintf`, `GetAsDictionary`, `GetValueForKey`, `GetUnsignedIntegerValue`, and 1 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; abstracts host OS resources such as files, terminals, sockets, or platforms; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `Sprintf`, `GetAsDictionary`, `GetValueForKey`, `GetUnsignedIntegerValue`, and 1 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并抽象文件、终端、套接字或平台等主机操作系统资源，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 795-809
```cpp
  return result;
}

bool InstrumentationRuntimeTSan::NotifyBreakpointHit(
    void *baton, StoppointCallbackContext *context, user_id_t break_id,
    user_id_t break_loc_id) {
  assert(baton && "null baton");
  if (!baton)
    return false;

  InstrumentationRuntimeTSan *const instance =
      static_cast<InstrumentationRuntimeTSan *>(baton);

  ProcessSP process_sp = instance->GetProcessSP();

```
- **EN**: Implements logic around `NotifyBreakpointHit`, `assert`, `GetProcessSP`.
- **CN**: 围绕 `NotifyBreakpointHit`, `assert`, `GetProcessSP` 实现具体逻辑。

### Lines 810-828
```cpp
  if (process_sp->GetModIDRef().IsLastResumeForUserExpression())
    return false;

  StructuredData::ObjectSP report =
      instance->RetrieveReportData(context->exe_ctx_ref);
  std::string stop_reason_description =
      "unknown thread sanitizer fault (unable to extract thread sanitizer "
      "report)";
  if (report) {
    std::string issue_description = instance->FormatDescription(report);
    report->GetAsDictionary()->AddStringItem("description", issue_description);
    stop_reason_description = issue_description + " detected";
    report->GetAsDictionary()->AddStringItem("stop_description",
                                             stop_reason_description);
    std::string summary = instance->GenerateSummary(report);
    report->GetAsDictionary()->AddStringItem("summary", summary);
    addr_t main_address = instance->GetMainRacyAddress(report);
    report->GetAsDictionary()->AddIntegerItem("memory_address", main_address);

```
- **EN**: Implements logic around `GetModIDRef`, `RetrieveReportData`, `fault`, `FormatDescription`, and 3 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `GetModIDRef`, `RetrieveReportData`, `fault`, `FormatDescription`, and 3 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并控制调试器侧的值格式化或 synthetic 展示。

### Lines 829-848
```cpp
    addr_t global_addr = 0;
    std::string global_name;
    std::string location_filename;
    uint32_t location_line = 0;
    std::string location_description = instance->GetLocationDescription(
        report, global_addr, global_name, location_filename, location_line);
    report->GetAsDictionary()->AddStringItem("location_description",
                                             location_description);
    if (global_addr != 0) {
      report->GetAsDictionary()->AddIntegerItem("global_address", global_addr);
    }
    if (!global_name.empty()) {
      report->GetAsDictionary()->AddStringItem("global_name", global_name);
    }
    if (location_filename != "") {
      report->GetAsDictionary()->AddStringItem("location_filename",
                                               location_filename);
      report->GetAsDictionary()->AddIntegerItem("location_line", location_line);
    }

```
- **EN**: Implements logic around `GetLocationDescription`, `GetAsDictionary`, `empty`.
- **CN**: 围绕 `GetLocationDescription`, `GetAsDictionary`, `empty` 实现具体逻辑。

### Lines 849-862
```cpp
    bool all_addresses_are_same = true;
    report->GetObjectForDotSeparatedPath("mops")->GetAsArray()->ForEach(
        [&all_addresses_are_same,
         main_address](StructuredData::Object *o) -> bool {
          addr_t addr = o->GetObjectForDotSeparatedPath("address")
                            ->GetUnsignedIntegerValue();
          if (main_address != addr)
            all_addresses_are_same = false;
          return true;
        });
    report->GetAsDictionary()->AddBooleanItem("all_addresses_are_same",
                                              all_addresses_are_same);
  }

```
- **EN**: Implements logic around `GetObjectForDotSeparatedPath`, `GetUnsignedIntegerValue`, `GetAsDictionary`; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `GetObjectForDotSeparatedPath`, `GetUnsignedIntegerValue`, `GetAsDictionary` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 863-877
```cpp
  // Make sure this is the right process
  if (process_sp && process_sp == context->exe_ctx_ref.GetProcessSP()) {
    ThreadSP thread_sp = context->exe_ctx_ref.GetThreadSP();
    if (thread_sp)
      thread_sp->SetStopInfo(
          InstrumentationRuntimeStopInfo::
              CreateStopReasonWithInstrumentationData(
                  *thread_sp, stop_reason_description, report));

    lldb::StreamSP s =
        process_sp->GetTarget().GetDebugger().GetAsyncOutputStream();
    s->Printf("ThreadSanitizer report breakpoint hit. Use 'thread "
              "info -s' to get extended information about the "
              "report.\n");

```
- **EN**: Implements logic around `GetProcessSP`, `GetThreadSP`, `SetStopInfo`, `CreateStopReasonWithInstrumentationData`, and 2 more symbols; this block tracks breakpoint state, stop conditions, or hit-processing policy; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetProcessSP`, `GetThreadSP`, `SetStopInfo`, `CreateStopReasonWithInstrumentationData`, and 2 more symbols 实现具体逻辑；该代码块跟踪断点状态、停机条件或命中处理策略，并协调调试器运行时对象、事件或异步控制流。

### Lines 878-896
```cpp
    return true; // Return true to stop the target
  }
    return false; // Let target run
}

const RegularExpression &
InstrumentationRuntimeTSan::GetPatternForRuntimeLibrary() {
  static RegularExpression regex(llvm::StringRef("libclang_rt.tsan_"));
  return regex;
}

bool InstrumentationRuntimeTSan::CheckIfRuntimeIsValid(
    const lldb::ModuleSP module_sp) {
  static ConstString g_tsan_get_current_report("__tsan_get_current_report");
  const Symbol *symbol = module_sp->FindFirstSymbolWithNameAndType(
      g_tsan_get_current_report, lldb::eSymbolTypeAny);
  return symbol != nullptr;
}

```
- **EN**: Implements logic around `GetPatternForRuntimeLibrary`, `regex`, `CheckIfRuntimeIsValid`, `g_tsan_get_current_report`, and 1 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `GetPatternForRuntimeLibrary`, `regex`, `CheckIfRuntimeIsValid`, `g_tsan_get_current_report`, and 1 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址。

### Lines 897-911
```cpp
void InstrumentationRuntimeTSan::Activate() {
  if (IsActive())
    return;

  ProcessSP process_sp = GetProcessSP();
  if (!process_sp)
    return;

  ConstString symbol_name("__tsan_on_report");
  const Symbol *symbol = GetRuntimeModuleSP()->FindFirstSymbolWithNameAndType(
      symbol_name, eSymbolTypeCode);

  if (symbol == nullptr)
    return;

```
- **EN**: Implements logic around `Activate`, `IsActive`, `GetProcessSP`, `symbol_name`, and 1 more symbols; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `Activate`, `IsActive`, `GetProcessSP`, `symbol_name`, and 1 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 912-932
```cpp
  if (!symbol->ValueIsAddress() || !symbol->GetAddressRef().IsValid())
    return;

  Target &target = process_sp->GetTarget();
  addr_t symbol_address = symbol->GetAddressRef().GetOpcodeLoadAddress(&target);

  if (symbol_address == LLDB_INVALID_ADDRESS)
    return;

  const bool internal = true;
  const bool hardware = false;
  const bool sync = false;
  Breakpoint *breakpoint =
      process_sp->GetTarget()
          .CreateBreakpoint(symbol_address, internal, hardware)
          .get();
  breakpoint->SetCallback(InstrumentationRuntimeTSan::NotifyBreakpointHit, this,
                          sync);
  breakpoint->SetBreakpointKind("thread-sanitizer-report");
  SetBreakpointID(breakpoint->GetID());

```
- **EN**: Implements logic around `ValueIsAddress`, `GetTarget`, `GetAddressRef`, `CreateBreakpoint`, and 4 more symbols; this block tracks breakpoint state, stop conditions, or hit-processing policy; coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `ValueIsAddress`, `GetTarget`, `GetAddressRef`, `CreateBreakpoint`, and 4 more symbols 实现具体逻辑；该代码块跟踪断点状态、停机条件或命中处理策略，并协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址。

### Lines 933-950
```cpp
  SetActive(true);
}

void InstrumentationRuntimeTSan::Deactivate() {
  if (GetBreakpointID() != LLDB_INVALID_BREAK_ID) {
    ProcessSP process_sp = GetProcessSP();
    if (process_sp) {
      process_sp->GetTarget().RemoveBreakpointByID(GetBreakpointID());
      SetBreakpointID(LLDB_INVALID_BREAK_ID);
    }
  }
  SetActive(false);
}
static std::string GenerateThreadName(const std::string &path,
                                      StructuredData::Object *o,
                                      StructuredData::ObjectSP main_info) {
  std::string result = "additional information";

```
- **EN**: Implements logic around `SetActive`, `Deactivate`, `GetBreakpointID`, `GetProcessSP`, and 3 more symbols.
- **CN**: 围绕 `SetActive`, `Deactivate`, `GetBreakpointID`, `GetProcessSP`, and 3 more symbols 实现具体逻辑。

### Lines 951-964
```cpp
  if (path == "mops") {
    size_t size =
        o->GetObjectForDotSeparatedPath("size")->GetUnsignedIntegerValue();
    lldb::tid_t thread_id =
        o->GetObjectForDotSeparatedPath("thread_id")->GetUnsignedIntegerValue();
    bool is_write =
        o->GetObjectForDotSeparatedPath("is_write")->GetBooleanValue();
    bool is_atomic =
        o->GetObjectForDotSeparatedPath("is_atomic")->GetBooleanValue();
    addr_t addr =
        o->GetObjectForDotSeparatedPath("address")->GetUnsignedIntegerValue();

    std::string addr_string = Sprintf(" at 0x%llx", addr);

```
- **EN**: Implements logic around `GetObjectForDotSeparatedPath`, `Sprintf`; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `GetObjectForDotSeparatedPath`, `Sprintf` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 965-983
```cpp
    if (main_info->GetObjectForDotSeparatedPath("all_addresses_are_same")
            ->GetBooleanValue()) {
      addr_string = "";
    }

    if (main_info->GetObjectForDotSeparatedPath("issue_type")
            ->GetStringValue() == "external-race") {
      result = Sprintf("%s access by thread %d",
                       is_write ? "mutating" : "read-only", thread_id);
    } else if (main_info->GetObjectForDotSeparatedPath("issue_type")
                   ->GetStringValue() == "swift-access-race") {
      result = Sprintf("modifying access by thread %d", thread_id);
    } else {
      result = Sprintf("%s%s of size %zu%s by thread %" PRIu64,
                       is_atomic ? "atomic " : "", is_write ? "write" : "read",
                       size, addr_string.c_str(), thread_id);
    }
  }

```
- **EN**: Implements logic around `GetObjectForDotSeparatedPath`, `GetBooleanValue`, `GetStringValue`, `Sprintf`, and 1 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetObjectForDotSeparatedPath`, `GetBooleanValue`, `GetStringValue`, `Sprintf`, and 1 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 984-1004
```cpp
  if (path == "threads") {
    lldb::tid_t thread_id =
        o->GetObjectForDotSeparatedPath("thread_id")->GetUnsignedIntegerValue();
    result = Sprintf("Thread %zu created", thread_id);
  }

  if (path == "locs") {
    std::string type = std::string(
        o->GetAsDictionary()->GetValueForKey("type")->GetStringValue());
    lldb::tid_t thread_id =
        o->GetObjectForDotSeparatedPath("thread_id")->GetUnsignedIntegerValue();
    int fd = o->GetObjectForDotSeparatedPath("file_descriptor")
                 ->GetSignedIntegerValue();
    if (type == "heap") {
      result = Sprintf("Heap block allocated by thread %" PRIu64, thread_id);
    } else if (type == "fd") {
      result = Sprintf("File descriptor %d created by thread %" PRIu64, fd,
                       thread_id);
    }
  }

```
- **EN**: Implements logic around `GetObjectForDotSeparatedPath`, `Sprintf`, `string`, `GetAsDictionary`, and 1 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `GetObjectForDotSeparatedPath`, `Sprintf`, `string`, `GetAsDictionary`, and 1 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 1005-1019
```cpp
  if (path == "mutexes") {
    int mutex_id =
        o->GetObjectForDotSeparatedPath("mutex_id")->GetSignedIntegerValue();

    result = Sprintf("Mutex M%d created", mutex_id);
  }

  if (path == "stacks") {
    lldb::tid_t thread_id =
        o->GetObjectForDotSeparatedPath("thread_id")->GetUnsignedIntegerValue();
    result = Sprintf("Thread %" PRIu64, thread_id);
  }

  result[0] = toupper(result[0]);

```
- **EN**: Implements logic around `GetObjectForDotSeparatedPath`, `Sprintf`, `toupper`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetObjectForDotSeparatedPath`, `Sprintf`, `toupper` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1020-1034
```cpp
  return result;
}

static void AddThreadsForPath(const std::string &path,
                              ThreadCollectionSP threads, ProcessSP process_sp,
                              StructuredData::ObjectSP info) {
  info->GetObjectForDotSeparatedPath(path)->GetAsArray()->ForEach(
      [process_sp, threads, path, info](StructuredData::Object *o) -> bool {
        std::vector<lldb::addr_t> pcs;
        o->GetObjectForDotSeparatedPath("trace")->GetAsArray()->ForEach(
            [&pcs](StructuredData::Object *pc) -> bool {
              pcs.push_back(pc->GetUnsignedIntegerValue());
              return true;
            });

```
- **EN**: Implements logic around `AddThreadsForPath`, `GetObjectForDotSeparatedPath`, `push_back`.
- **CN**: 围绕 `AddThreadsForPath`, `GetObjectForDotSeparatedPath`, `push_back` 实现具体逻辑。

### Lines 1035-1051
```cpp
        if (pcs.size() == 0)
          return true;

        StructuredData::ObjectSP thread_id_obj =
            o->GetObjectForDotSeparatedPath("thread_os_id");
        lldb::tid_t tid =
            thread_id_obj ? thread_id_obj->GetUnsignedIntegerValue() : 0;

        ThreadSP new_thread_sp =
            std::make_shared<HistoryThread>(*process_sp, tid, pcs);
        new_thread_sp->SetName(GenerateThreadName(path, o, info).c_str());

        // Save this in the Process' ExtendedThreadList so a strong pointer
        // retains the object
        process_sp->GetExtendedThreadList().AddThread(new_thread_sp);
        threads->AddThread(new_thread_sp);

```
- **EN**: Implements logic around `size`, `GetObjectForDotSeparatedPath`, `GetUnsignedIntegerValue`, `make_shared`, and 3 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `size`, `GetObjectForDotSeparatedPath`, `GetUnsignedIntegerValue`, `make_shared`, and 3 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1052-1065
```cpp
        return true;
      });
}

lldb::ThreadCollectionSP
InstrumentationRuntimeTSan::GetBacktracesFromExtendedStopInfo(
    StructuredData::ObjectSP info) {

  ThreadCollectionSP threads = std::make_shared<ThreadCollection>();

  if (info->GetObjectForDotSeparatedPath("instrumentation_class")
          ->GetStringValue() != "ThreadSanitizer")
    return threads;

```
- **EN**: Implements logic around `GetBacktracesFromExtendedStopInfo`, `make_shared`, `GetObjectForDotSeparatedPath`, `GetStringValue`.
- **CN**: 围绕 `GetBacktracesFromExtendedStopInfo`, `make_shared`, `GetObjectForDotSeparatedPath`, `GetStringValue` 实现具体逻辑。

### Lines 1066-1075
```cpp
  ProcessSP process_sp = GetProcessSP();

  AddThreadsForPath("stacks", threads, process_sp, info);
  AddThreadsForPath("mops", threads, process_sp, info);
  AddThreadsForPath("locs", threads, process_sp, info);
  AddThreadsForPath("mutexes", threads, process_sp, info);
  AddThreadsForPath("threads", threads, process_sp, info);

  return threads;
}
```
- **EN**: Implements logic around `GetProcessSP`, `AddThreadsForPath`.
- **CN**: 围绕 `GetProcessSP`, `AddThreadsForPath` 实现具体逻辑。

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

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `InstrumentationRuntimeTSan.h`, `Plugins/Process/Utility/HistoryThread.h`, `lldb/Breakpoint/StoppointCallbackContext.h`, `lldb/Core/Debugger.h`, `lldb/Core/Module.h`, `lldb/Core/PluginInterface.h`, `lldb/Core/PluginManager.h`, `lldb/Expression/UserExpression.h`, `lldb/Host/StreamFile.h`, `lldb/Interpreter/CommandReturnObject.h` ... (+14 more)
- **Standard-library headers / 标准库头文件**: `<memory>`
- **Subsystem categories / 子系统类别**: target, process, and thread control / 目标、进程与线程控制 (5), LLDB core debugger abstractions / LLDB 核心调试抽象 (4), symbol and debug-info abstractions / 符号与调试信息抽象 (4), shared LLDB utility classes / 共享 LLDB 工具类 (4), breakpoint-management infrastructure / 断点管理基础设施 (1), expression parsing and evaluation support / 表达式解析与求值支持 (1), host OS abstraction helpers / 主机操作系统抽象辅助组件 (1), command interpreter support / 命令解释器支持 (1)
