# ScriptInterpreterLua.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/ScriptInterpreter/Lua/ScriptInterpreterLua.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `ScriptInterpreterLua`.
  - **CN**: 实现与 `ScriptInterpreterLua` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
//===-- ScriptInterpreterLua.cpp ------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "ScriptInterpreterLua.h"
#include "LuaState.h"
#include "lldb/Breakpoint/StoppointCallbackContext.h"
#include "lldb/Core/Debugger.h"
#include "lldb/Core/PluginManager.h"
#include "lldb/Host/StreamFile.h"
#include "lldb/Interpreter/CommandReturnObject.h"
#include "lldb/Target/ExecutionContext.h"
#include "lldb/Utility/Stream.h"
#include "lldb/Utility/StringList.h"
#include "lldb/Utility/Timer.h"
#include "lldb/lldb-forward.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `ScriptInterpreterLua.h`, `LuaState.h`, `lldb/Breakpoint/StoppointCallbackContext.h`, `lldb/Core/Debugger.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `ScriptInterpreterLua.h`, `LuaState.h`, `lldb/Breakpoint/StoppointCallbackContext.h`, `lldb/Core/Debugger.h`。

### Lines 21-30
```cpp
#include "llvm/ADT/StringRef.h"
#include "llvm/Support/FormatAdapters.h"
#include <memory>
#include <vector>

using namespace lldb;
using namespace lldb_private;

LLDB_PLUGIN_DEFINE(ScriptInterpreterLua)

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/ADT/StringRef.h`, `llvm/Support/FormatAdapters.h`, `memory`, `vector`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/ADT/StringRef.h`, `llvm/Support/FormatAdapters.h`, `memory`, `vector`。

### Lines 31-50
```cpp
enum ActiveIOHandler {
  eIOHandlerNone,
  eIOHandlerBreakpoint,
  eIOHandlerWatchpoint
};

class IOHandlerLuaInterpreter : public IOHandlerDelegate,
                                public IOHandlerEditline {
public:
  IOHandlerLuaInterpreter(Debugger &debugger,
                          ScriptInterpreterLua &script_interpreter,
                          ActiveIOHandler active_io_handler = eIOHandlerNone)
      : IOHandlerEditline(debugger, IOHandler::Type::LuaInterpreter, "lua",
                          llvm::StringRef(">>> "), llvm::StringRef("..> "),
                          true, debugger.GetUseColor(), 0, *this),
        m_script_interpreter(script_interpreter),
        m_active_io_handler(active_io_handler) {
    llvm::cantFail(m_script_interpreter.GetLuaState().ChangeIO(
        debugger.GetOutputFileSP()->GetStream(),
        debugger.GetErrorFileSP()->GetStream()));
```
- **EN**: Introduces declarations for `ActiveIOHandler`, `IOHandlerLuaInterpreter`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ActiveIOHandler`, `IOHandlerLuaInterpreter` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 51-70
```cpp
    llvm::cantFail(m_script_interpreter.EnterSession(debugger.GetID()));
  }

  ~IOHandlerLuaInterpreter() override {
    llvm::cantFail(m_script_interpreter.LeaveSession());
  }

  void IOHandlerActivated(IOHandler &io_handler, bool interactive) override {
    const char *instructions = nullptr;
    switch (m_active_io_handler) {
    case eIOHandlerNone:
      break;
    case eIOHandlerWatchpoint:
      instructions = "Enter your Lua command(s). Type 'quit' to end.\n"
                     "The commands are compiled as the body of the following "
                     "Lua function\n"
                     "function (frame, wp) end\n";
      SetPrompt(llvm::StringRef("..> "));
      break;
    case eIOHandlerBreakpoint:
```
- **EN**: Implements logic around `cantFail`, `~IOHandlerLuaInterpreter`, `IOHandlerActivated`, `command`, and 2 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; registers commands, plugins, or interpreter-facing extension points; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `cantFail`, `~IOHandlerLuaInterpreter`, `IOHandlerActivated`, `command`, and 2 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并注册命令、插件或面向解释器的扩展点，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 71-88
```cpp
      instructions = "Enter your Lua command(s). Type 'quit' to end.\n"
                     "The commands are compiled as the body of the following "
                     "Lua function\n"
                     "function (frame, bp_loc, ...) end\n";
      SetPrompt(llvm::StringRef("..> "));
      break;
    }
    if (instructions == nullptr)
      return;
    if (interactive) {
      if (lldb::LockableStreamFileSP output_sp =
              io_handler.GetOutputStreamFileSP()) {
        LockedStreamFile locked_stream = output_sp->Lock();
        locked_stream << instructions;
      }
    }
  }

```
- **EN**: Implements logic around `command`, `function`, `SetPrompt`, `GetOutputStreamFileSP`, and 1 more symbols; this block registers commands, plugins, or interpreter-facing extension points; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `command`, `function`, `SetPrompt`, `GetOutputStreamFileSP`, and 1 more symbols 实现具体逻辑；该代码块注册命令、插件或面向解释器的扩展点，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 89-108
```cpp
  bool IOHandlerIsInputComplete(IOHandler &io_handler,
                                StringList &lines) override {
    size_t last = lines.GetSize() - 1;
    if (IsQuitCommand(lines.GetStringAtIndex(last))) {
      if (m_active_io_handler == eIOHandlerBreakpoint ||
          m_active_io_handler == eIOHandlerWatchpoint)
        lines.DeleteStringAtIndex(last);
      return true;
    }
    StreamString str;
    lines.Join("\n", str);
    if (llvm::Error E =
            m_script_interpreter.GetLuaState().CheckSyntax(str.GetString())) {
      std::string error_str = toString(std::move(E));
      // Lua always errors out to incomplete code with '<eof>'
      return error_str.find("<eof>") == std::string::npos;
    }
    // The breakpoint and watchpoint handler only exits with a explicit 'quit'
    return m_active_io_handler != eIOHandlerBreakpoint &&
           m_active_io_handler != eIOHandlerWatchpoint;
```
- **EN**: Implements logic around `IOHandlerIsInputComplete`, `GetSize`, `IsQuitCommand`, `DeleteStringAtIndex`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics; tracks breakpoint state, stop conditions, or hit-processing policy.
- **CN**: 围绕 `IOHandlerIsInputComplete`, `GetSize`, `IsQuitCommand`, `DeleteStringAtIndex`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并跟踪断点状态、停机条件或命中处理策略。

### Lines 109-128
```cpp
  }

  void IOHandlerInputComplete(IOHandler &io_handler,
                              std::string &data) override {
    switch (m_active_io_handler) {
    case eIOHandlerBreakpoint: {
      auto *bp_options_vec =
          static_cast<std::vector<std::reference_wrapper<BreakpointOptions>> *>(
              io_handler.GetUserData());
      for (BreakpointOptions &bp_options : *bp_options_vec) {
        Status error = m_script_interpreter.SetBreakpointCommandCallback(
            bp_options, data.c_str(), /*is_callback=*/false);
        if (error.Fail()) {
          LockedStreamFile locked_stream =
              io_handler.GetErrorStreamFileSP()->Lock();
          locked_stream << error.AsCString() << '\n';
        }
      }
      io_handler.SetIsDone(true);
    } break;
```
- **EN**: Implements logic around `IOHandlerInputComplete`, `GetUserData`, `SetBreakpointCommandCallback`, `c_str`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `IOHandlerInputComplete`, `GetUserData`, `SetBreakpointCommandCallback`, `c_str`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 129-148
```cpp
    case eIOHandlerWatchpoint: {
      auto *wp_options =
          static_cast<WatchpointOptions *>(io_handler.GetUserData());
      m_script_interpreter.SetWatchpointCommandCallback(wp_options,
                                                        data.c_str(),
                                                        /*is_callback=*/false);
      io_handler.SetIsDone(true);
    } break;
    case eIOHandlerNone:
      if (IsQuitCommand(data)) {
        io_handler.SetIsDone(true);
        return;
      }
      if (llvm::Error error = m_script_interpreter.GetLuaState().Run(data)) {
        LockedStreamFile locked_stream =
            io_handler.GetErrorStreamFileSP()->Lock();
        locked_stream << toString(std::move(error));
      }
      break;
    }
```
- **EN**: Implements logic around `GetUserData`, `SetWatchpointCommandCallback`, `c_str`, `SetIsDone`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `GetUserData`, `SetWatchpointCommandCallback`, `c_str`, `SetIsDone`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 149-161
```cpp
  }

private:
  ScriptInterpreterLua &m_script_interpreter;
  ActiveIOHandler m_active_io_handler;

  bool IsQuitCommand(llvm::StringRef cmd) { return cmd.rtrim() == "quit"; }
};

ScriptInterpreterLua::ScriptInterpreterLua(Debugger &debugger)
    : ScriptInterpreter(debugger, eScriptLanguageLua),
      m_lua_state(std::make_unique<LuaState>()) {}

```
- **EN**: Implements logic around `IsQuitCommand`, `ScriptInterpreterLua`, `ScriptInterpreter`, `m_lua_state`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `IsQuitCommand`, `ScriptInterpreterLua`, `ScriptInterpreter`, `m_lua_state` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 162-178
```cpp
ScriptInterpreterLua::~ScriptInterpreterLua() = default;

StructuredData::DictionarySP ScriptInterpreterLua::GetInterpreterInfo() {
  auto info = std::make_shared<StructuredData::Dictionary>();
  info->AddStringItem("language", "lua");
  return info;
}

bool ScriptInterpreterLua::ExecuteOneLine(llvm::StringRef command,
                                          CommandReturnObject *result,
                                          const ExecuteScriptOptions &options) {
  if (command.empty()) {
    if (result)
      result->AppendError("empty command passed to lua\n");
    return false;
  }

```
- **EN**: Implements logic around `~ScriptInterpreterLua`, `GetInterpreterInfo`, `Dictionary>`, `AddStringItem`, and 3 more symbols; this block registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `~ScriptInterpreterLua`, `GetInterpreterInfo`, `Dictionary>`, `AddStringItem`, and 3 more symbols 实现具体逻辑；该代码块注册命令、插件或面向解释器的扩展点。

### Lines 179-191
```cpp
  llvm::Expected<std::unique_ptr<ScriptInterpreterIORedirect>>
      io_redirect_or_error = ScriptInterpreterIORedirect::Create(
          options.GetEnableIO(), m_debugger, result);
  if (!io_redirect_or_error) {
    if (result)
      result->AppendErrorWithFormatv(
          "failed to redirect I/O: {0}\n",
          llvm::fmt_consume(io_redirect_or_error.takeError()));
    else
      llvm::consumeError(io_redirect_or_error.takeError());
    return false;
  }

```
- **EN**: Implements logic around `Create`, `GetEnableIO`, `AppendErrorWithFormatv`, `fmt_consume`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `Create`, `GetEnableIO`, `AppendErrorWithFormatv`, `fmt_consume`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 192-201
```cpp
  ScriptInterpreterIORedirect &io_redirect = **io_redirect_or_error;

  if (llvm::Error e =
          m_lua_state->ChangeIO(io_redirect.GetOutputFile()->GetStream(),
                                io_redirect.GetErrorFile()->GetStream())) {
    result->AppendErrorWithFormatv("lua failed to redirect I/O: {0}\n",
                                   llvm::toString(std::move(e)));
    return false;
  }

```
- **EN**: Implements logic around `ChangeIO`, `GetErrorFile`, `AppendErrorWithFormatv`, `toString`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `ChangeIO`, `GetErrorFile`, `AppendErrorWithFormatv`, `toString` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 202-212
```cpp
  if (llvm::Error e = m_lua_state->Run(command)) {
    result->AppendErrorWithFormatv(
        "lua failed attempting to evaluate '{0}': {1}\n", command,
        llvm::toString(std::move(e)));
    return false;
  }

  io_redirect.Flush();
  return true;
}

```
- **EN**: Implements logic around `Run`, `AppendErrorWithFormatv`, `toString`, `Flush`; this block propagates recoverable errors, status objects, or diagnostics; registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `Run`, `AppendErrorWithFormatv`, `toString`, `Flush` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并注册命令、插件或面向解释器的扩展点。

### Lines 213-223
```cpp
void ScriptInterpreterLua::ExecuteInterpreterLoop() {
  LLDB_SCOPED_TIMER();

  // At the moment, the only time the debugger does not have an input file
  // handle is when this is called directly from lua, in which case it is
  // both dangerous and unnecessary (not to mention confusing) to try to embed
  // a running interpreter loop inside the already running lua interpreter
  // loop, so we won't do it.
  if (!m_debugger.GetInputFile().IsValid())
    return;

```
- **EN**: Implements logic around `ExecuteInterpreterLoop`, `LLDB_SCOPED_TIMER`, `GetInputFile`; this block coordinates debugger runtime objects, events, or asynchronous control flow; abstracts host OS resources such as files, terminals, sockets, or platforms; registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `ExecuteInterpreterLoop`, `LLDB_SCOPED_TIMER`, `GetInputFile` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并抽象文件、终端、套接字或平台等主机操作系统资源，并注册命令、插件或面向解释器的扩展点。

### Lines 224-241
```cpp
  IOHandlerSP io_handler_sp(new IOHandlerLuaInterpreter(m_debugger, *this));
  m_debugger.RunIOHandlerAsync(io_handler_sp);
}

bool ScriptInterpreterLua::LoadScriptingModule(
    const char *filename, const LoadScriptOptions &options,
    lldb_private::Status &error, StructuredData::ObjectSP *module_sp,
    FileSpec extra_search_dir, lldb::TargetSP loaded_into_target_sp) {

  if (llvm::Error e = m_lua_state->LoadModule(filename)) {
    error = Status::FromErrorStringWithFormatv(
        "lua failed to import '{0}': {1}\n", filename,
        llvm::toString(std::move(e)));
    return false;
  }
  return true;
}

```
- **EN**: Implements logic around `io_handler_sp`, `RunIOHandlerAsync`, `LoadScriptingModule`, `LoadModule`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `io_handler_sp`, `RunIOHandlerAsync`, `LoadScriptingModule`, `LoadModule`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 242-251
```cpp
void ScriptInterpreterLua::Initialize() {
  PluginManager::RegisterPlugin(GetPluginNameStatic(),
                                GetPluginDescriptionStatic(),
                                lldb::eScriptLanguageLua, CreateInstance);
}

void ScriptInterpreterLua::Terminate() {
  PluginManager::UnregisterPlugin(CreateInstance);
}

```
- **EN**: Implements logic around `Initialize`, `RegisterPlugin`, `GetPluginDescriptionStatic`, `Terminate`, and 1 more symbols.
- **CN**: 围绕 `Initialize`, `RegisterPlugin`, `GetPluginDescriptionStatic`, `Terminate`, and 1 more symbols 实现具体逻辑。

### Lines 252-264
```cpp
llvm::Error ScriptInterpreterLua::EnterSession(user_id_t debugger_id) {
  if (m_session_is_active)
    return llvm::Error::success();

  const char *fmt_str =
      "lldb.debugger = lldb.SBDebugger.FindDebuggerWithID({0}); "
      "lldb.target = lldb.debugger:GetSelectedTarget(); "
      "lldb.process = lldb.target:GetProcess(); "
      "lldb.thread = lldb.process:GetSelectedThread(); "
      "lldb.frame = lldb.thread:GetSelectedFrame()";
  return m_lua_state->Run(llvm::formatv(fmt_str, debugger_id).str());
}

```
- **EN**: Implements logic around `EnterSession`, `success`, `FindDebuggerWithID`, `debugger:GetSelectedTarget`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `EnterSession`, `success`, `FindDebuggerWithID`, `debugger:GetSelectedTarget`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 265-278
```cpp
llvm::Error ScriptInterpreterLua::LeaveSession() {
  if (!m_session_is_active)
    return llvm::Error::success();

  m_session_is_active = false;

  llvm::StringRef str = "lldb.debugger = nil; "
                        "lldb.target = nil; "
                        "lldb.process = nil; "
                        "lldb.thread = nil; "
                        "lldb.frame = nil";
  return m_lua_state->Run(str);
}

```
- **EN**: Implements logic around `LeaveSession`, `success`, `Run`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `LeaveSession`, `success`, `Run` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 279-288
```cpp
bool ScriptInterpreterLua::BreakpointCallbackFunction(
    void *baton, StoppointCallbackContext *context, user_id_t break_id,
    user_id_t break_loc_id) {
  assert(context);

  ExecutionContext exe_ctx(context->exe_ctx_ref);
  Target *target = exe_ctx.GetTargetPtr();
  if (target == nullptr)
    return true;

```
- **EN**: Implements logic around `BreakpointCallbackFunction`, `assert`, `exe_ctx`, `GetTargetPtr`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `BreakpointCallbackFunction`, `assert`, `exe_ctx`, `GetTargetPtr` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 289-305
```cpp
  StackFrameSP stop_frame_sp(exe_ctx.GetFrameSP());
  BreakpointSP breakpoint_sp = target->GetBreakpointByID(break_id);
  BreakpointLocationSP bp_loc_sp(breakpoint_sp->FindLocationByID(break_loc_id));

  Debugger &debugger = target->GetDebugger();
  ScriptInterpreterLua *lua_interpreter = static_cast<ScriptInterpreterLua *>(
      debugger.GetScriptInterpreter(true, eScriptLanguageLua));
  LuaState &lua_state = lua_interpreter->GetLuaState();

  CommandDataLua *bp_option_data = static_cast<CommandDataLua *>(baton);
  llvm::Expected<bool> BoolOrErr = lua_state.CallBreakpointCallback(
      baton, stop_frame_sp, bp_loc_sp, bp_option_data->m_extra_args_sp);
  if (llvm::Error E = BoolOrErr.takeError()) {
    *debugger.GetAsyncErrorStream() << toString(std::move(E));
    return true;
  }

```
- **EN**: Implements logic around `stop_frame_sp`, `GetBreakpointByID`, `bp_loc_sp`, `GetDebugger`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `stop_frame_sp`, `GetBreakpointByID`, `bp_loc_sp`, `GetDebugger`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 306-317
```cpp
  return *BoolOrErr;
}

bool ScriptInterpreterLua::WatchpointCallbackFunction(
    void *baton, StoppointCallbackContext *context, user_id_t watch_id) {
  assert(context);

  ExecutionContext exe_ctx(context->exe_ctx_ref);
  Target *target = exe_ctx.GetTargetPtr();
  if (target == nullptr)
    return true;

```
- **EN**: Implements logic around `WatchpointCallbackFunction`, `assert`, `exe_ctx`, `GetTargetPtr`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `WatchpointCallbackFunction`, `assert`, `exe_ctx`, `GetTargetPtr` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 318-332
```cpp
  StackFrameSP stop_frame_sp(exe_ctx.GetFrameSP());
  WatchpointSP wp_sp = target->GetWatchpointList().FindByID(watch_id);

  Debugger &debugger = target->GetDebugger();
  ScriptInterpreterLua *lua_interpreter = static_cast<ScriptInterpreterLua *>(
      debugger.GetScriptInterpreter(true, eScriptLanguageLua));
  LuaState &lua_state = lua_interpreter->GetLuaState();

  llvm::Expected<bool> BoolOrErr =
      lua_state.CallWatchpointCallback(baton, stop_frame_sp, wp_sp);
  if (llvm::Error E = BoolOrErr.takeError()) {
    *debugger.GetAsyncErrorStream() << toString(std::move(E));
    return true;
  }

```
- **EN**: Implements logic around `stop_frame_sp`, `GetWatchpointList`, `GetDebugger`, `GetScriptInterpreter`, and 3 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `stop_frame_sp`, `GetWatchpointList`, `GetDebugger`, `GetScriptInterpreter`, and 3 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 333-344
```cpp
  return *BoolOrErr;
}

void ScriptInterpreterLua::CollectDataForBreakpointCommandCallback(
    std::vector<std::reference_wrapper<BreakpointOptions>> &bp_options_vec,
    CommandReturnObject &result) {
  IOHandlerSP io_handler_sp(
      new IOHandlerLuaInterpreter(m_debugger, *this, eIOHandlerBreakpoint));
  io_handler_sp->SetUserData(&bp_options_vec);
  m_debugger.RunIOHandlerAsync(io_handler_sp);
}

```
- **EN**: Implements logic around `CollectDataForBreakpointCommandCallback`, `io_handler_sp`, `IOHandlerLuaInterpreter`, `SetUserData`, and 1 more symbols.
- **CN**: 围绕 `CollectDataForBreakpointCommandCallback`, `io_handler_sp`, `IOHandlerLuaInterpreter`, `SetUserData`, and 1 more symbols 实现具体逻辑。

### Lines 345-361
```cpp
void ScriptInterpreterLua::CollectDataForWatchpointCommandCallback(
    WatchpointOptions *wp_options, CommandReturnObject &result) {
  IOHandlerSP io_handler_sp(
      new IOHandlerLuaInterpreter(m_debugger, *this, eIOHandlerWatchpoint));
  io_handler_sp->SetUserData(wp_options);
  m_debugger.RunIOHandlerAsync(io_handler_sp);
}

Status ScriptInterpreterLua::SetBreakpointCommandCallbackFunction(
    BreakpointOptions &bp_options, const char *function_name,
    StructuredData::ObjectSP extra_args_sp) {
  const char *fmt_str = "return {0}(frame, bp_loc, ...)";
  std::string oneliner = llvm::formatv(fmt_str, function_name).str();
  return RegisterBreakpointCallback(bp_options, oneliner.c_str(),
                                    extra_args_sp);
}

```
- **EN**: Implements logic around `CollectDataForWatchpointCommandCallback`, `io_handler_sp`, `IOHandlerLuaInterpreter`, `SetUserData`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `CollectDataForWatchpointCommandCallback`, `io_handler_sp`, `IOHandlerLuaInterpreter`, `SetUserData`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 362-381
```cpp
Status ScriptInterpreterLua::SetBreakpointCommandCallback(
    BreakpointOptions &bp_options, const char *command_body_text,
    bool is_callback) {
  return RegisterBreakpointCallback(bp_options, command_body_text, {});
}

Status ScriptInterpreterLua::RegisterBreakpointCallback(
    BreakpointOptions &bp_options, const char *command_body_text,
    StructuredData::ObjectSP extra_args_sp) {
  auto data_up = std::make_unique<CommandDataLua>(extra_args_sp);
  llvm::Error err =
      m_lua_state->RegisterBreakpointCallback(data_up.get(), command_body_text);
  if (err)
    return Status::FromError(std::move(err));
  auto baton_sp =
      std::make_shared<BreakpointOptions::CommandBaton>(std::move(data_up));
  bp_options.SetCallback(ScriptInterpreterLua::BreakpointCallbackFunction,
                         baton_sp);
  return {};
}
```
- **EN**: Implements logic around `SetBreakpointCommandCallback`, `RegisterBreakpointCallback`, `make_unique`, `FromError`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `SetBreakpointCommandCallback`, `RegisterBreakpointCallback`, `make_unique`, `FromError`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 382-401
```cpp

void ScriptInterpreterLua::SetWatchpointCommandCallback(
    WatchpointOptions *wp_options, const char *command_body_text,
    bool is_callback) {
  RegisterWatchpointCallback(wp_options, command_body_text, {});
}

Status ScriptInterpreterLua::RegisterWatchpointCallback(
    WatchpointOptions *wp_options, const char *command_body_text,
    StructuredData::ObjectSP extra_args_sp) {
  auto data_up = std::make_unique<WatchpointOptions::CommandData>();
  llvm::Error err =
      m_lua_state->RegisterWatchpointCallback(data_up.get(), command_body_text);
  if (err)
    return Status::FromError(std::move(err));
  auto baton_sp =
      std::make_shared<WatchpointOptions::CommandBaton>(std::move(data_up));
  wp_options->SetCallback(ScriptInterpreterLua::WatchpointCallbackFunction,
                          baton_sp);
  return {};
```
- **EN**: Implements logic around `SetWatchpointCommandCallback`, `RegisterWatchpointCallback`, `CommandData>`, `FromError`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `SetWatchpointCommandCallback`, `RegisterWatchpointCallback`, `CommandData>`, `FromError`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 402-412
```cpp
}

lldb::ScriptInterpreterSP
ScriptInterpreterLua::CreateInstance(Debugger &debugger) {
  return std::make_shared<ScriptInterpreterLua>(debugger);
}

llvm::StringRef ScriptInterpreterLua::GetPluginDescriptionStatic() {
  return "Lua script interpreter";
}

```
- **EN**: Implements logic around `CreateInstance`, `make_shared`, `GetPluginDescriptionStatic`; this block coordinates debugger runtime objects, events, or asynchronous control flow; registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `CreateInstance`, `make_shared`, `GetPluginDescriptionStatic` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并注册命令、插件或面向解释器的扩展点。

### Lines 413-413
```cpp
LuaState &ScriptInterpreterLua::GetLuaState() { return *m_lua_state; }
```
- **EN**: Implements logic around `GetLuaState`.
- **CN**: 围绕 `GetLuaState` 实现具体逻辑。

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
- **OS integration / 操作系统集成**:
  - **EN**: Bridges LLDB abstractions to host files, sockets, terminals, and native process facilities.
  - **CN**: 将 LLDB 抽象连接到主机文件、套接字、终端与本地进程设施。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `ScriptInterpreterLua.h`, `LuaState.h`, `lldb/Breakpoint/StoppointCallbackContext.h`, `lldb/Core/Debugger.h`, `lldb/Core/PluginManager.h`, `lldb/Host/StreamFile.h`, `lldb/Interpreter/CommandReturnObject.h`, `lldb/Target/ExecutionContext.h`, `lldb/Utility/Stream.h`, `lldb/Utility/StringList.h` ... (+4 more)
- **Standard-library headers / 标准库头文件**: `<memory>`, `<vector>`
- **Subsystem categories / 子系统类别**: shared LLDB utility classes / 共享 LLDB 工具类 (3), LLDB core debugger abstractions / LLDB 核心调试抽象 (2), breakpoint-management infrastructure / 断点管理基础设施 (1), host OS abstraction helpers / 主机操作系统抽象辅助组件 (1), command interpreter support / 命令解释器支持 (1), target, process, and thread control / 目标、进程与线程控制 (1), shared LLDB public/private type declarations / 共享 LLDB 公共/私有类型声明 (1), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1)
