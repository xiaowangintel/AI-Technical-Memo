# AppleObjCRuntime.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/LanguageRuntime/ObjC/AppleObjCRuntime/AppleObjCRuntime.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `AppleObjCRuntime`.
  - **CN**: 实现与 `AppleObjCRuntime` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
//===-- AppleObjCRuntime.cpp ----------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "AppleObjCRuntime.h"
#include "AppleObjCRuntimeV1.h"
#include "AppleObjCRuntimeV2.h"
#include "AppleObjCTrampolineHandler.h"
#include "Plugins/Language/ObjC/NSString.h"
#include "Plugins/LanguageRuntime/CPlusPlus/CPPLanguageRuntime.h"
#include "Plugins/Process/Utility/HistoryThread.h"
#include "lldb/Breakpoint/BreakpointLocation.h"
#include "lldb/Core/Module.h"
#include "lldb/Core/ModuleList.h"
#include "lldb/Core/PluginManager.h"
#include "lldb/Core/Section.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `AppleObjCRuntime.h`, `AppleObjCRuntimeV1.h`, `AppleObjCRuntimeV2.h`, `AppleObjCTrampolineHandler.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `AppleObjCRuntime.h`, `AppleObjCRuntimeV1.h`, `AppleObjCRuntimeV2.h`, `AppleObjCTrampolineHandler.h`。

### Lines 21-40
```cpp
#include "lldb/DataFormatters/FormattersHelpers.h"
#include "lldb/Expression/DiagnosticManager.h"
#include "lldb/Expression/FunctionCaller.h"
#include "lldb/Symbol/ObjectFile.h"
#include "lldb/Target/ExecutionContext.h"
#include "lldb/Target/Process.h"
#include "lldb/Target/RegisterContext.h"
#include "lldb/Target/StopInfo.h"
#include "lldb/Target/Target.h"
#include "lldb/Target/Thread.h"
#include "lldb/Utility/ConstString.h"
#include "lldb/Utility/ErrorMessages.h"
#include "lldb/Utility/LLDBLog.h"
#include "lldb/Utility/Log.h"
#include "lldb/Utility/Scalar.h"
#include "lldb/Utility/Status.h"
#include "lldb/Utility/StreamString.h"
#include "lldb/ValueObject/ValueObject.h"
#include "lldb/ValueObject/ValueObjectConstResult.h"
#include "clang/AST/Type.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `lldb/DataFormatters/FormattersHelpers.h`, `lldb/Expression/DiagnosticManager.h`, `lldb/Expression/FunctionCaller.h`, `lldb/Symbol/ObjectFile.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `lldb/DataFormatters/FormattersHelpers.h`, `lldb/Expression/DiagnosticManager.h`, `lldb/Expression/FunctionCaller.h`, `lldb/Symbol/ObjectFile.h`。

### Lines 41-50
```cpp

#include "Plugins/TypeSystem/Clang/TypeSystemClang.h"

#include <vector>

using namespace lldb;
using namespace lldb_private;

LLDB_PLUGIN_DEFINE(AppleObjCRuntime)

```
- **EN**: Pulls in the headers needed by this translation unit, including `Plugins/TypeSystem/Clang/TypeSystemClang.h`, `vector`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `Plugins/TypeSystem/Clang/TypeSystemClang.h`, `vector`。

### Lines 51-60
```cpp
char AppleObjCRuntime::ID = 0;

AppleObjCRuntime::~AppleObjCRuntime() = default;

AppleObjCRuntime::AppleObjCRuntime(Process *process)
    : ObjCLanguageRuntime(process), m_read_objc_library(false),
      m_objc_trampoline_handler_up(), m_Foundation_major() {
  ReadObjCLibraryIfNeeded(process->GetTarget().GetImages());
}

```
- **EN**: Implements logic around `~AppleObjCRuntime`, `AppleObjCRuntime`, `ObjCLanguageRuntime`, `m_objc_trampoline_handler_up`, and 1 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `~AppleObjCRuntime`, `AppleObjCRuntime`, `ObjCLanguageRuntime`, `m_objc_trampoline_handler_up`, and 1 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 61-70
```cpp
void AppleObjCRuntime::Initialize() {
  AppleObjCRuntimeV2::Initialize();
  AppleObjCRuntimeV1::Initialize();
}

void AppleObjCRuntime::Terminate() {
  AppleObjCRuntimeV2::Terminate();
  AppleObjCRuntimeV1::Terminate();
}

```
- **EN**: Implements logic around `Initialize`, `Terminate`.
- **CN**: 围绕 `Initialize`, `Terminate` 实现具体逻辑。

### Lines 71-83
```cpp
llvm::Error AppleObjCRuntime::GetObjectDescription(Stream &str,
                                                   ValueObject &valobj) {
  CompilerType compiler_type(valobj.GetCompilerType());
  bool is_signed;
  // ObjC objects can only be pointers (or numbers that actually represents
  // pointers but haven't been typecast, because reasons..)
  if (!compiler_type.IsIntegerType(is_signed) && !compiler_type.IsPointerType())
    return llvm::createStringError("not a pointer type");

  // Make the argument list: we pass one arg, the address of our pointer, to
  // the print function.
  Value val;

```
- **EN**: Implements logic around `GetObjectDescription`, `compiler_type`, `IsIntegerType`, `createStringError`; this block propagates recoverable errors, status objects, or diagnostics; maps executable state back to modules, symbols, sections, or addresses; controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `GetObjectDescription`, `compiler_type`, `IsIntegerType`, `createStringError` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并将可执行状态映射回模块、符号、节区或地址，并控制调试器侧的值格式化或 synthetic 展示。

### Lines 84-100
```cpp
  if (!valobj.ResolveValue(val.GetScalar()))
    return llvm::createStringError("pointer value could not be resolved");

  // Value Objects may not have a process in their ExecutionContextRef.  But we
  // need to have one in the ref we pass down to eventually call description.
  // Get it from the target if it isn't present.
  ExecutionContext exe_ctx;
  if (valobj.GetProcessSP()) {
    exe_ctx = ExecutionContext(valobj.GetExecutionContextRef());
  } else {
    exe_ctx.SetContext(valobj.GetTargetSP(), true);
    if (!exe_ctx.HasProcessScope())
      return llvm::createStringError("no process");
  }
  return GetObjectDescription(str, val, exe_ctx.GetBestExecutionContextScope());
}

```
- **EN**: Implements logic around `ResolveValue`, `createStringError`, `GetProcessSP`, `ExecutionContext`, and 3 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `ResolveValue`, `createStringError`, `GetProcessSP`, `ExecutionContext`, and 3 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并控制调试器侧的值格式化或 synthetic 展示。

### Lines 101-112
```cpp
llvm::Error
AppleObjCRuntime::GetObjectDescription(Stream &strm, Value &value,
                                       ExecutionContextScope *exe_scope) {
  if (!m_read_objc_library)
    return llvm::createStringError("Objective-C runtime not loaded");

  ExecutionContext exe_ctx;
  exe_scope->CalculateExecutionContext(exe_ctx);
  Process *process = exe_ctx.GetProcessPtr();
  if (!process)
    return llvm::createStringError("no process");

```
- **EN**: Implements logic around `GetObjectDescription`, `createStringError`, `CalculateExecutionContext`, `GetProcessPtr`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetObjectDescription`, `createStringError`, `CalculateExecutionContext`, `GetProcessPtr` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 113-132
```cpp
  // We need other parts of the exe_ctx, but the processes have to match.
  assert(m_process == process);

  // Get the function address for the print function.
  const Address *function_address = GetPrintForDebuggerAddr();
  if (!function_address)
    return llvm::createStringError("no print function");

  Target *target = exe_ctx.GetTargetPtr();
  CompilerType compiler_type = value.GetCompilerType();
  if (compiler_type) {
    if (!TypeSystemClang::IsObjCObjectPointerType(compiler_type))
      return llvm::createStringError(
          "Value doesn't point to an ObjC object.\n");
  } else {
    // If it is not a pointer, see if we can make it into a pointer.
    TypeSystemClangSP scratch_ts_sp =
        ScratchTypeSystemClang::GetForTarget(*target);
    if (!scratch_ts_sp)
      return llvm::createStringError("no scratch type system");
```
- **EN**: Implements logic around `assert`, `GetPrintForDebuggerAddr`, `createStringError`, `GetTargetPtr`, and 3 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `assert`, `GetPrintForDebuggerAddr`, `createStringError`, `GetTargetPtr`, and 3 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址。

### Lines 133-144
```cpp

    CompilerType opaque_type = scratch_ts_sp->GetBasicType(eBasicTypeObjCID);
    if (!opaque_type)
      opaque_type =
          scratch_ts_sp->GetBasicType(eBasicTypeVoid).GetPointerType();
    // value.SetContext(Value::eContextTypeClangType, opaque_type_ptr);
    value.SetCompilerType(opaque_type);
  }

  ValueList arg_value_list;
  arg_value_list.PushValue(value);

```
- **EN**: Implements logic around `GetBasicType`, `SetCompilerType`, `PushValue`.
- **CN**: 围绕 `GetBasicType`, `SetCompilerType`, `PushValue` 实现具体逻辑。

### Lines 145-155
```cpp
  // This is the return value:
  TypeSystemClangSP scratch_ts_sp =
      ScratchTypeSystemClang::GetForTarget(*target);
  if (!scratch_ts_sp)
    return llvm::createStringError("no scratch type system");

  CompilerType return_compiler_type = scratch_ts_sp->GetCStringType(true);
  Value ret;
  //    ret.SetContext(Value::eContextTypeClangType, return_compiler_type);
  ret.SetCompilerType(return_compiler_type);

```
- **EN**: Implements logic around `GetForTarget`, `createStringError`, `GetCStringType`, `SetCompilerType`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetForTarget`, `createStringError`, `GetCStringType`, `SetCompilerType` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 156-166
```cpp
  if (!exe_ctx.GetFramePtr()) {
    Thread *thread = exe_ctx.GetThreadPtr();
    if (thread == nullptr) {
      exe_ctx.SetThreadSP(process->GetThreadList().GetSelectedThread());
      thread = exe_ctx.GetThreadPtr();
    }
    if (thread) {
      exe_ctx.SetFrameSP(thread->GetSelectedFrame(DoNoSelectMostRelevantFrame));
    }
  }

```
- **EN**: Implements logic around `GetFramePtr`, `GetThreadPtr`, `SetThreadSP`, `SetFrameSP`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetFramePtr`, `GetThreadPtr`, `SetThreadSP`, `SetFrameSP` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 167-186
```cpp
  // Now we're ready to call the function:

  DiagnosticManager diagnostics;
  lldb::addr_t wrapper_struct_addr = LLDB_INVALID_ADDRESS;

  if (!m_print_object_caller_up) {
    Status error;
    m_print_object_caller_up.reset(
        exe_scope->CalculateTarget()->GetFunctionCallerForLanguage(
            eLanguageTypeObjC, return_compiler_type, *function_address,
            arg_value_list, "objc-object-description", error));
    if (error.Fail()) {
      m_print_object_caller_up.reset();
      return llvm::createStringError(
          llvm::Twine(
              "could not get function runner to call print for debugger "
              "function: ") +
          error.AsCString());
    }
    m_print_object_caller_up->InsertFunction(exe_ctx, wrapper_struct_addr,
```
- **EN**: Implements logic around `reset`, `CalculateTarget`, `Fail`, `createStringError`, and 3 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `reset`, `CalculateTarget`, `Fail`, `createStringError`, and 3 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 187-200
```cpp
                                             diagnostics);
  } else {
    m_print_object_caller_up->WriteFunctionArguments(
        exe_ctx, wrapper_struct_addr, arg_value_list, diagnostics);
  }

  EvaluateExpressionOptions options;
  options.SetUnwindOnError(true);
  options.SetTryAllThreads(true);
  options.SetStopOthers(true);
  options.SetIgnoreBreakpoints(true);
  options.SetTimeout(process->GetUtilityExpressionTimeout());
  options.SetIsForUtilityExpr(true);

```
- **EN**: Implements logic around `WriteFunctionArguments`, `SetUnwindOnError`, `SetTryAllThreads`, `SetStopOthers`, and 3 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `WriteFunctionArguments`, `SetUnwindOnError`, `SetTryAllThreads`, `SetStopOthers`, and 3 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 201-220
```cpp
  ExpressionResults results = m_print_object_caller_up->ExecuteFunction(
      exe_ctx, &wrapper_struct_addr, options, diagnostics, ret);
  if (results != eExpressionCompleted)
    return llvm::createStringError(
        "could not evaluate print object function: " + toString(results));

  addr_t result_ptr = ret.GetScalar().ULongLong(LLDB_INVALID_ADDRESS);

  char buf[512];
  size_t cstr_len = 0;
  size_t full_buffer_len = sizeof(buf) - 1;
  size_t curr_len = full_buffer_len;
  while (curr_len == full_buffer_len) {
    Status error;
    curr_len = process->ReadCStringFromMemory(result_ptr + cstr_len, buf,
                                              sizeof(buf), error);
    strm.Write(buf, curr_len);
    cstr_len += curr_len;
  }
  if (cstr_len > 0)
```
- **EN**: Implements logic around `ExecuteFunction`, `createStringError`, `toString`, `GetScalar`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `ExecuteFunction`, `createStringError`, `toString`, `GetScalar`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 221-240
```cpp
    return llvm::Error::success();
  return llvm::createStringError("empty object description");
}

lldb::ModuleSP AppleObjCRuntime::GetObjCModule() {
  ModuleSP module_sp(m_objc_module_wp.lock());
  if (module_sp)
    return module_sp;

  Process *process = GetProcess();
  if (process) {
    const ModuleList &modules = process->GetTarget().GetImages();
    for (uint32_t idx = 0; idx < modules.GetSize(); idx++) {
      module_sp = modules.GetModuleAtIndex(idx);
      if (AppleObjCRuntime::AppleIsModuleObjCLibrary(module_sp)) {
        m_objc_module_wp = module_sp;
        return module_sp;
      }
    }
  }
```
- **EN**: Implements logic around `success`, `createStringError`, `GetObjCModule`, `module_sp`, and 5 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `success`, `createStringError`, `GetObjCModule`, `module_sp`, and 5 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 241-250
```cpp
  return ModuleSP();
}

Address *AppleObjCRuntime::GetPrintForDebuggerAddr() {
  if (!m_PrintForDebugger_addr) {
    const ModuleList &modules = m_process->GetTarget().GetImages();

    SymbolContextList contexts;
    SymbolContext context;

```
- **EN**: Implements logic around `ModuleSP`, `GetPrintForDebuggerAddr`, `GetTarget`; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `ModuleSP`, `GetPrintForDebuggerAddr`, `GetTarget` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 251-261
```cpp
    modules.FindSymbolsWithNameAndType(ConstString("_NSPrintForDebugger"),
                                        eSymbolTypeCode, contexts);
    if (contexts.IsEmpty()) {
      modules.FindSymbolsWithNameAndType(ConstString("_CFPrintForDebugger"),
                                         eSymbolTypeCode, contexts);
      if (contexts.IsEmpty())
        return nullptr;
    }

    contexts.GetContextAtIndex(0, context);

```
- **EN**: Implements logic around `FindSymbolsWithNameAndType`, `IsEmpty`, `GetContextAtIndex`.
- **CN**: 围绕 `FindSymbolsWithNameAndType`, `IsEmpty`, `GetContextAtIndex` 实现具体逻辑。

### Lines 262-275
```cpp
    m_PrintForDebugger_addr =
        std::make_unique<Address>(context.symbol->GetAddress());
  }

  return m_PrintForDebugger_addr.get();
}

bool AppleObjCRuntime::CouldHaveDynamicValue(ValueObject &in_value) {
  return in_value.GetCompilerType().IsPossibleDynamicType(
      nullptr,
      false, // do not check C++
      true); // check ObjC
}

```
- **EN**: Implements logic around `make_unique`, `get`, `CouldHaveDynamicValue`, `GetCompilerType`; this block maps executable state back to modules, symbols, sections, or addresses; controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `make_unique`, `get`, `CouldHaveDynamicValue`, `GetCompilerType` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并控制调试器侧的值格式化或 synthetic 展示。

### Lines 276-288
```cpp
bool AppleObjCRuntime::GetDynamicTypeAndAddress(
    ValueObject &in_value, lldb::DynamicValueType use_dynamic,
    TypeAndOrName &class_type_or_name, Address &address,
    Value::ValueType &value_type, llvm::ArrayRef<uint8_t> &local_buffer) {
  return false;
}

TypeAndOrName
AppleObjCRuntime::FixUpDynamicType(const TypeAndOrName &type_and_or_name,
                                   ValueObject &static_value) {
  CompilerType static_type(static_value.GetCompilerType());
  Flags static_type_flags(static_type.GetTypeInfo());

```
- **EN**: Implements logic around `GetDynamicTypeAndAddress`, `FixUpDynamicType`, `static_type`, `static_type_flags`; this block maps executable state back to modules, symbols, sections, or addresses; controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `GetDynamicTypeAndAddress`, `FixUpDynamicType`, `static_type`, `static_type_flags` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并控制调试器侧的值格式化或 synthetic 展示。

### Lines 289-308
```cpp
  TypeAndOrName ret(type_and_or_name);
  if (type_and_or_name.HasType()) {
    // The type will always be the type of the dynamic object.  If our parent's
    // type was a pointer, then our type should be a pointer to the type of the
    // dynamic object.  If a reference, then the original type should be
    // okay...
    CompilerType orig_type = type_and_or_name.GetCompilerType();
    CompilerType corrected_type = orig_type;
    if (static_type_flags.AllSet(eTypeIsPointer))
      corrected_type = orig_type.GetPointerType();
    ret.SetCompilerType(corrected_type);
  } else {
    // If we are here we need to adjust our dynamic type name to include the
    // correct & or * symbol
    std::string corrected_name(type_and_or_name.GetName().GetCString());
    if (static_type_flags.AllSet(eTypeIsPointer))
      corrected_name.append(" *");
    // the parent type should be a correctly pointer'ed or referenc'ed type
    ret.SetCompilerType(static_type);
    ret.SetName(corrected_name.c_str());
```
- **EN**: Implements logic around `ret`, `HasType`, `GetCompilerType`, `AllSet`, and 5 more symbols; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `ret`, `HasType`, `GetCompilerType`, `AllSet`, and 5 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 309-325
```cpp
  }
  return ret;
}

bool AppleObjCRuntime::AppleIsModuleObjCLibrary(const ModuleSP &module_sp) {
  if (module_sp) {
    const FileSpec &module_file_spec = module_sp->GetFileSpec();
    static ConstString ObjCName("libobjc.A.dylib");

    if (module_file_spec) {
      if (module_file_spec.GetFilename() == ObjCName)
        return true;
    }
  }
  return false;
}

```
- **EN**: Implements logic around `AppleIsModuleObjCLibrary`, `GetFileSpec`, `ObjCName`, `GetFilename`.
- **CN**: 围绕 `AppleIsModuleObjCLibrary`, `GetFileSpec`, `ObjCName`, `GetFilename` 实现具体逻辑。

### Lines 326-345
```cpp
// we use the version of Foundation to make assumptions about the ObjC runtime
// on a target
uint32_t AppleObjCRuntime::GetFoundationVersion() {
  if (!m_Foundation_major) {
    const ModuleList &modules = m_process->GetTarget().GetImages();
    for (uint32_t idx = 0; idx < modules.GetSize(); idx++) {
      lldb::ModuleSP module_sp = modules.GetModuleAtIndex(idx);
      if (!module_sp)
        continue;
      if (strcmp(module_sp->GetFileSpec().GetFilename().AsCString(""),
                 "Foundation") == 0) {
        m_Foundation_major = module_sp->GetVersion().getMajor();
        return *m_Foundation_major;
      }
    }
    return LLDB_INVALID_MODULE_VERSION;
  } else
    return *m_Foundation_major;
}

```
- **EN**: Implements logic around `GetFoundationVersion`, `GetTarget`, `GetSize`, `GetModuleAtIndex`, and 2 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetFoundationVersion`, `GetTarget`, `GetSize`, `GetModuleAtIndex`, and 2 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 346-365
```cpp
void AppleObjCRuntime::GetValuesForGlobalCFBooleans(lldb::addr_t &cf_true,
                                                    lldb::addr_t &cf_false) {
  cf_true = cf_false = LLDB_INVALID_ADDRESS;
}

bool AppleObjCRuntime::IsModuleObjCLibrary(const ModuleSP &module_sp) {
  return AppleIsModuleObjCLibrary(module_sp);
}

bool AppleObjCRuntime::ReadObjCLibrary(const ModuleSP &module_sp) {
  // Maybe check here and if we have a handler already, and the UUID of this
  // module is the same as the one in the current module, then we don't have to
  // reread it?
  m_objc_trampoline_handler_up = std::make_unique<AppleObjCTrampolineHandler>(
      m_process->shared_from_this(), module_sp);
  if (m_objc_trampoline_handler_up != nullptr) {
    m_read_objc_library = true;
    return true;
  } else
    return false;
```
- **EN**: Implements logic around `GetValuesForGlobalCFBooleans`, `IsModuleObjCLibrary`, `AppleIsModuleObjCLibrary`, `ReadObjCLibrary`, and 2 more symbols; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `GetValuesForGlobalCFBooleans`, `IsModuleObjCLibrary`, `AppleIsModuleObjCLibrary`, `ReadObjCLibrary`, and 2 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 366-376
```cpp
}

ThreadPlanSP AppleObjCRuntime::GetStepThroughTrampolinePlan(Thread &thread,
                                                            bool stop_others) {
  ThreadPlanSP thread_plan_sp;
  if (m_objc_trampoline_handler_up)
    thread_plan_sp = m_objc_trampoline_handler_up->GetStepThroughDispatchPlan(
        thread, stop_others);
  return thread_plan_sp;
}

```
- **EN**: Implements logic around `GetStepThroughTrampolinePlan`, `GetStepThroughDispatchPlan`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetStepThroughTrampolinePlan`, `GetStepThroughDispatchPlan` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 377-387
```cpp
// Static Functions
ObjCLanguageRuntime::ObjCRuntimeVersions
AppleObjCRuntime::GetObjCVersion(Process *process, ModuleSP &objc_module_sp) {
  if (!process)
    return ObjCRuntimeVersions::eObjC_VersionUnknown;

  Target &target = process->GetTarget();
  if (target.GetArchitecture().GetTriple().getVendor() !=
      llvm::Triple::VendorType::Apple)
    return ObjCRuntimeVersions::eObjC_VersionUnknown;

```
- **EN**: Implements logic around `GetObjCVersion`, `GetTarget`, `GetArchitecture`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetObjCVersion`, `GetTarget`, `GetArchitecture` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 388-401
```cpp
  for (ModuleSP module_sp : target.GetImages().Modules()) {
    // One tricky bit here is that we might get called as part of the initial
    // module loading, but before all the pre-run libraries get winnowed from
    // the module list.  So there might actually be an old and incorrect ObjC
    // library sitting around in the list, and we don't want to look at that.
    // That's why we call IsLoadedInTarget.

    if (AppleIsModuleObjCLibrary(module_sp) &&
        module_sp->IsLoadedInTarget(&target)) {
      objc_module_sp = module_sp;
      ObjectFile *ofile = module_sp->GetObjectFile();
      if (!ofile)
        return ObjCRuntimeVersions::eObjC_VersionUnknown;

```
- **EN**: Implements logic around `GetImages`, `AppleIsModuleObjCLibrary`, `IsLoadedInTarget`, `GetObjectFile`; this block coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `GetImages`, `AppleIsModuleObjCLibrary`, `IsLoadedInTarget`, `GetObjectFile` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址。

### Lines 402-413
```cpp
      SectionList *sections = module_sp->GetSectionList();
      if (!sections)
        return ObjCRuntimeVersions::eObjC_VersionUnknown;
      SectionSP v1_telltale_section_sp =
          sections->FindSectionByName(ConstString("__OBJC"));
      if (v1_telltale_section_sp) {
        return ObjCRuntimeVersions::eAppleObjC_V1;
      }
      return ObjCRuntimeVersions::eAppleObjC_V2;
    }
  }

```
- **EN**: Implements logic around `GetSectionList`, `FindSectionByName`.
- **CN**: 围绕 `GetSectionList`, `FindSectionByName` 实现具体逻辑。

### Lines 414-431
```cpp
  return ObjCRuntimeVersions::eObjC_VersionUnknown;
}

void AppleObjCRuntime::SetExceptionBreakpoints() {
  const bool catch_bp = false;
  const bool throw_bp = true;
  const bool is_internal = true;

  if (!m_objc_exception_bp_sp) {
    m_objc_exception_bp_sp = LanguageRuntime::CreateExceptionBreakpoint(
        m_process->GetTarget(), GetLanguageType(), catch_bp, throw_bp,
        is_internal);
    if (m_objc_exception_bp_sp)
      m_objc_exception_bp_sp->SetBreakpointKind("ObjC exception");
  } else
    m_objc_exception_bp_sp->SetEnabled(true);
}

```
- **EN**: Implements logic around `SetExceptionBreakpoints`, `CreateExceptionBreakpoint`, `GetTarget`, `SetBreakpointKind`, and 1 more symbols.
- **CN**: 围绕 `SetExceptionBreakpoints`, `CreateExceptionBreakpoint`, `GetTarget`, `SetBreakpointKind`, and 1 more symbols 实现具体逻辑。

### Lines 432-444
```cpp
void AppleObjCRuntime::ClearExceptionBreakpoints() {
  if (!m_process)
    return;

  if (m_objc_exception_bp_sp.get()) {
    m_objc_exception_bp_sp->SetEnabled(false);
  }
}

bool AppleObjCRuntime::ExceptionBreakpointsAreSet() {
  return m_objc_exception_bp_sp && m_objc_exception_bp_sp->IsEnabled();
}

```
- **EN**: Implements logic around `ClearExceptionBreakpoints`, `get`, `SetEnabled`, `ExceptionBreakpointsAreSet`, and 1 more symbols.
- **CN**: 围绕 `ClearExceptionBreakpoints`, `get`, `SetEnabled`, `ExceptionBreakpointsAreSet`, and 1 more symbols 实现具体逻辑。

### Lines 445-457
```cpp
bool AppleObjCRuntime::ExceptionBreakpointsExplainStop(
    lldb::StopInfoSP stop_reason) {
  if (!m_process)
    return false;

  if (!stop_reason || stop_reason->GetStopReason() != eStopReasonBreakpoint)
    return false;

  uint64_t break_site_id = stop_reason->GetValue();
  return m_process->GetBreakpointSiteList().StopPointSiteContainsBreakpoint(
      break_site_id, m_objc_exception_bp_sp->GetID());
}

```
- **EN**: Implements logic around `ExceptionBreakpointsExplainStop`, `GetStopReason`, `GetValue`, `GetBreakpointSiteList`, and 1 more symbols.
- **CN**: 围绕 `ExceptionBreakpointsExplainStop`, `GetStopReason`, `GetValue`, `GetBreakpointSiteList`, and 1 more symbols 实现具体逻辑。

### Lines 458-467
```cpp
bool AppleObjCRuntime::CalculateHasNewLiteralsAndIndexing() {
  if (!m_process)
    return false;

  static ConstString s_method_signature(
      "-[NSDictionary objectForKeyedSubscript:]");
  // NSDictionary is toll-free bridged with CFDictionary, so the
  // implementation lives in CoreFoundation, not Foundation.
  static ModuleSpec corefoundation_module_spec(FileSpec("CoreFoundation"));

```
- **EN**: Implements logic around `CalculateHasNewLiteralsAndIndexing`, `s_method_signature`, `corefoundation_module_spec`.
- **CN**: 围绕 `CalculateHasNewLiteralsAndIndexing`, `s_method_signature`, `corefoundation_module_spec` 实现具体逻辑。

### Lines 468-478
```cpp
  Target &target = m_process->GetTarget();
  if (ModuleSP corefoundation_module_sp =
          target.GetImages().FindFirstModule(corefoundation_module_spec)) {
    if (corefoundation_module_sp->FindFirstSymbolWithNameAndType(
            s_method_signature, eSymbolTypeCode))
      return true;
  }

  return false;
}

```
- **EN**: Implements logic around `GetTarget`, `GetImages`, `FindFirstSymbolWithNameAndType`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetTarget`, `GetImages`, `FindFirstSymbolWithNameAndType` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 479-488
```cpp
lldb::SearchFilterSP AppleObjCRuntime::CreateExceptionSearchFilter() {
  Target &target = m_process->GetTarget();

  FileSpecList filter_modules;
  if (target.GetArchitecture().GetTriple().getVendor() == llvm::Triple::Apple) {
    filter_modules.Append(std::get<0>(GetExceptionThrowLocation()));
  }
  return target.GetSearchFilterForModuleList(&filter_modules);
}

```
- **EN**: Implements logic around `CreateExceptionSearchFilter`, `GetTarget`, `GetArchitecture`, `Append`, and 1 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `CreateExceptionSearchFilter`, `GetTarget`, `GetArchitecture`, `Append`, and 1 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 489-498
```cpp
ValueObjectSP AppleObjCRuntime::GetExceptionObjectForThread(
    ThreadSP thread_sp) {
  auto *cpp_runtime = m_process->GetLanguageRuntime(eLanguageTypeC_plus_plus);
  if (!cpp_runtime) return ValueObjectSP();
  auto cpp_exception = cpp_runtime->GetExceptionObjectForThread(thread_sp);
  if (!cpp_exception) return ValueObjectSP();

  auto descriptor = GetClassDescriptor(*cpp_exception);
  if (!descriptor || !descriptor->IsValid()) return ValueObjectSP();

```
- **EN**: Implements logic around `GetExceptionObjectForThread`, `GetLanguageRuntime`, `ValueObjectSP`, `GetClassDescriptor`, and 1 more symbols.
- **CN**: 围绕 `GetExceptionObjectForThread`, `GetLanguageRuntime`, `ValueObjectSP`, `GetClassDescriptor`, and 1 more symbols 实现具体逻辑。

### Lines 499-508
```cpp
  while (descriptor) {
    ConstString class_name(descriptor->GetClassName());
    if (class_name == "NSException")
      return cpp_exception;
    descriptor = descriptor->GetSuperclass();
  }

  return ValueObjectSP();
}

```
- **EN**: Implements logic around `class_name`, `GetSuperclass`, `ValueObjectSP`.
- **CN**: 围绕 `class_name`, `GetSuperclass`, `ValueObjectSP` 实现具体逻辑。

### Lines 509-519
```cpp
/// Utility method for error handling in GetBacktraceThreadFromException.
/// \param msg The message to add to the log.
/// \return An invalid ThreadSP to be returned from
///         GetBacktraceThreadFromException.
[[nodiscard]]
static ThreadSP FailExceptionParsing(llvm::StringRef msg) {
  Log *log = GetLog(LLDBLog::Language);
  LLDB_LOG(log, "Failed getting backtrace from exception: {0}", msg);
  return ThreadSP();
}

```
- **EN**: Implements logic around `FailExceptionParsing`, `GetLog`, `LLDB_LOG`, `ThreadSP`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `FailExceptionParsing`, `GetLog`, `LLDB_LOG`, `ThreadSP` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 520-530
```cpp
ThreadSP AppleObjCRuntime::GetBacktraceThreadFromException(
    lldb::ValueObjectSP exception_sp) {
  ValueObjectSP reserved_dict =
      exception_sp->GetChildMemberWithName("reserved");
  if (!reserved_dict)
    return FailExceptionParsing("Failed to get 'reserved' member.");

  reserved_dict = reserved_dict->GetSyntheticValue();
  if (!reserved_dict)
    return FailExceptionParsing("Failed to get synthetic value.");

```
- **EN**: Implements logic around `GetBacktraceThreadFromException`, `GetChildMemberWithName`, `FailExceptionParsing`, `GetSyntheticValue`; this block propagates recoverable errors, status objects, or diagnostics; controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `GetBacktraceThreadFromException`, `GetChildMemberWithName`, `FailExceptionParsing`, `GetSyntheticValue` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并控制调试器侧的值格式化或 synthetic 展示。

### Lines 531-547
```cpp
  TypeSystemClangSP scratch_ts_sp =
      ScratchTypeSystemClang::GetForTarget(*exception_sp->GetTargetSP());
  if (!scratch_ts_sp)
    return FailExceptionParsing("Failed to get scratch AST.");
  CompilerType objc_id = scratch_ts_sp->GetBasicType(lldb::eBasicTypeObjCID);
  ValueObjectSP return_addresses;

  auto objc_object_from_address = [&exception_sp, &objc_id](uint64_t addr,
                                                            const char *name) {
    Value value(addr);
    value.SetCompilerType(objc_id);
    auto object = ValueObjectConstResult::Create(
        exception_sp->GetTargetSP().get(), value, ConstString(name));
    object = object->GetDynamicValue(eDynamicDontRunTarget);
    return object;
  };

```
- **EN**: Implements logic around `GetForTarget`, `FailExceptionParsing`, `GetBasicType`, `value`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `GetForTarget`, `FailExceptionParsing`, `GetBasicType`, `value`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 548-557
```cpp
  for (size_t idx = 0; idx < reserved_dict->GetNumChildrenIgnoringErrors();
       idx++) {
    ValueObjectSP dict_entry = reserved_dict->GetChildAtIndex(idx);

    DataExtractor data;
    data.SetAddressByteSize(dict_entry->GetProcessSP()->GetAddressByteSize());
    Status error;
    dict_entry->GetData(data, error);
    if (error.Fail()) return ThreadSP();

```
- **EN**: Implements logic around `GetNumChildrenIgnoringErrors`, `GetChildAtIndex`, `SetAddressByteSize`, `GetData`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `GetNumChildrenIgnoringErrors`, `GetChildAtIndex`, `SetAddressByteSize`, `GetData`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 558-574
```cpp
    lldb::offset_t data_offset = 0;
    auto dict_entry_key = data.GetAddress(&data_offset);
    auto dict_entry_value = data.GetAddress(&data_offset);

    auto key_nsstring = objc_object_from_address(dict_entry_key, "key");
    StreamString key_summary;
    if (lldb_private::formatters::NSStringSummaryProvider(
            *key_nsstring, key_summary, TypeSummaryOptions()) &&
        !key_summary.Empty()) {
      if (key_summary.GetString() == "\"callStackReturnAddresses\"") {
        return_addresses = objc_object_from_address(dict_entry_value,
                                                    "callStackReturnAddresses");
        break;
      }
    }
  }

```
- **EN**: Implements logic around `GetAddress`, `objc_object_from_address`, `NSStringSummaryProvider`, `Empty`, and 1 more symbols.
- **CN**: 围绕 `GetAddress`, `objc_object_from_address`, `NSStringSummaryProvider`, `Empty`, and 1 more symbols 实现具体逻辑。

### Lines 575-589
```cpp
  if (!return_addresses)
    return FailExceptionParsing("Failed to get return addresses.");
  auto frames_value = return_addresses->GetChildMemberWithName("_frames");
  if (!frames_value)
    return FailExceptionParsing("Failed to get frames_value.");
  addr_t frames_addr = frames_value->GetValueAsUnsigned(0);
  auto count_value = return_addresses->GetChildMemberWithName("_cnt");
  if (!count_value)
    return FailExceptionParsing("Failed to get count_value.");
  size_t count = count_value->GetValueAsUnsigned(0);
  auto ignore_value = return_addresses->GetChildMemberWithName("_ignore");
  if (!ignore_value)
    return FailExceptionParsing("Failed to get ignore_value.");
  size_t ignore = ignore_value->GetValueAsUnsigned(0);

```
- **EN**: Implements logic around `FailExceptionParsing`, `GetChildMemberWithName`, `GetValueAsUnsigned`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `FailExceptionParsing`, `GetChildMemberWithName`, `GetValueAsUnsigned` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 590-601
```cpp
  size_t ptr_size = m_process->GetAddressByteSize();
  std::vector<lldb::addr_t> pcs;
  for (size_t idx = 0; idx < count; idx++) {
    Status error;
    addr_t pc = m_process->ReadPointerFromMemory(
        frames_addr + (ignore + idx) * ptr_size, error);
    pcs.push_back(pc);
  }

  if (pcs.empty())
    return FailExceptionParsing("Failed to get PC list.");

```
- **EN**: Implements logic around `GetAddressByteSize`, `ReadPointerFromMemory`, `push_back`, `empty`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `GetAddressByteSize`, `ReadPointerFromMemory`, `push_back`, `empty`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 602-612
```cpp
  ThreadSP new_thread_sp(new HistoryThread(*m_process, 0, pcs));
  m_process->GetExtendedThreadList().AddThread(new_thread_sp);
  return new_thread_sp;
}

std::tuple<FileSpec, ConstString>
AppleObjCRuntime::GetExceptionThrowLocation() {
  return std::make_tuple(
      FileSpec("libobjc.A.dylib"), ConstString("objc_exception_throw"));
}

```
- **EN**: Implements logic around `new_thread_sp`, `GetExtendedThreadList`, `GetExceptionThrowLocation`, `make_tuple`, and 1 more symbols.
- **CN**: 围绕 `new_thread_sp`, `GetExtendedThreadList`, `GetExceptionThrowLocation`, `make_tuple`, and 1 more symbols 实现具体逻辑。

### Lines 613-627
```cpp
void AppleObjCRuntime::ReadObjCLibraryIfNeeded(const ModuleList &module_list) {
  if (!HasReadObjCLibrary()) {
    std::lock_guard<std::recursive_mutex> guard(module_list.GetMutex());

    size_t num_modules = module_list.GetSize();
    for (size_t i = 0; i < num_modules; i++) {
      auto mod = module_list.GetModuleAtIndex(i);
      if (IsModuleObjCLibrary(mod)) {
        ReadObjCLibrary(mod);
        break;
      }
    }
  }
}

```
- **EN**: Implements logic around `ReadObjCLibraryIfNeeded`, `HasReadObjCLibrary`, `guard`, `GetSize`, and 3 more symbols.
- **CN**: 围绕 `ReadObjCLibraryIfNeeded`, `HasReadObjCLibrary`, `guard`, `GetSize`, and 3 more symbols 实现具体逻辑。

### Lines 628-630
```cpp
void AppleObjCRuntime::ModulesDidLoad(const ModuleList &module_list) {
  ReadObjCLibraryIfNeeded(module_list);
}
```
- **EN**: Implements logic around `ModulesDidLoad`, `ReadObjCLibraryIfNeeded`.
- **CN**: 围绕 `ModulesDidLoad`, `ReadObjCLibraryIfNeeded` 实现具体逻辑。

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

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `AppleObjCRuntime.h`, `AppleObjCRuntimeV1.h`, `AppleObjCRuntimeV2.h`, `AppleObjCTrampolineHandler.h`, `Plugins/Language/ObjC/NSString.h`, `Plugins/LanguageRuntime/CPlusPlus/CPPLanguageRuntime.h`, `Plugins/Process/Utility/HistoryThread.h`, `lldb/Breakpoint/BreakpointLocation.h`, `lldb/Core/Module.h`, `lldb/Core/ModuleList.h` ... (+23 more)
- **Standard-library headers / 标准库头文件**: `<vector>`
- **Subsystem categories / 子系统类别**: shared LLDB utility classes / 共享 LLDB 工具类 (7), target, process, and thread control / 目标、进程与线程控制 (6), LLDB core debugger abstractions / LLDB 核心调试抽象 (4), expression parsing and evaluation support / 表达式解析与求值支持 (2), value inspection and presentation helpers / 值检查与展示辅助逻辑 (2), breakpoint-management infrastructure / 断点管理基础设施 (1), LLDB data formatter components / LLDB 数据格式化组件 (1), symbol and debug-info abstractions / 符号与调试信息抽象 (1)
