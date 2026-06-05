# ProcessKDP.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Process/MacOSX-Kernel/ProcessKDP.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `ProcessKDP`.
  - **CN**: 实现与 `ProcessKDP` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28
```cpp
//===-- ProcessKDP.cpp ----------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include <cerrno>
#include <cstdlib>

#include <memory>

#include "lldb/Core/Debugger.h"
#include "lldb/Core/Module.h"
#include "lldb/Core/ModuleSpec.h"
#include "lldb/Core/PluginManager.h"
#include "lldb/Host/ConnectionFileDescriptor.h"
#include "lldb/Host/Host.h"
#include "lldb/Host/ThreadLauncher.h"
#include "lldb/Host/common/TCPSocket.h"
#include "lldb/Interpreter/CommandInterpreter.h"
#include "lldb/Interpreter/CommandObject.h"
#include "lldb/Interpreter/CommandObjectMultiword.h"
#include "lldb/Interpreter/CommandReturnObject.h"
#include "lldb/Interpreter/OptionGroupString.h"
#include "lldb/Interpreter/OptionGroupUInt64.h"
#include "lldb/Interpreter/OptionValueProperties.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `cerrno`, `cstdlib`, `memory`, `lldb/Core/Debugger.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `cerrno`, `cstdlib`, `memory`, `lldb/Core/Debugger.h`。

### Lines 29-46
```cpp
#include "lldb/Symbol/ObjectFile.h"
#include "lldb/Target/RegisterContext.h"
#include "lldb/Target/Target.h"
#include "lldb/Target/Thread.h"
#include "lldb/Utility/LLDBLog.h"
#include "lldb/Utility/Log.h"
#include "lldb/Utility/State.h"
#include "lldb/Utility/StringExtractor.h"
#include "lldb/Utility/UUID.h"

#define USEC_PER_SEC 1000000

#include "Plugins/DynamicLoader/Darwin-Kernel/DynamicLoaderDarwinKernel.h"
#include "Plugins/DynamicLoader/Static/DynamicLoaderStatic.h"
#include "ProcessKDP.h"
#include "ProcessKDPLog.h"
#include "ThreadKDP.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `lldb/Symbol/ObjectFile.h`, `lldb/Target/RegisterContext.h`, `lldb/Target/Target.h`, `lldb/Target/Thread.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `lldb/Symbol/ObjectFile.h`, `lldb/Target/RegisterContext.h`, `lldb/Target/Target.h`, `lldb/Target/Thread.h`。

### Lines 47-61
```cpp
using namespace lldb;
using namespace lldb_private;

LLDB_PLUGIN_DEFINE_ADV(ProcessKDP, ProcessMacOSXKernel)

namespace {

#define LLDB_PROPERTIES_processkdp
#include "ProcessKDPProperties.inc"

enum {
#define LLDB_PROPERTIES_processkdp
#include "ProcessKDPPropertiesEnum.inc"
};

```
- **EN**: Pulls in the headers needed by this translation unit, including `ProcessKDPProperties.inc`, `ProcessKDPPropertiesEnum.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `ProcessKDPProperties.inc`, `ProcessKDPPropertiesEnum.inc`。

### Lines 62-81
```cpp
class PluginProperties : public Properties {
public:
  static llvm::StringRef GetSettingName() {
    return ProcessKDP::GetPluginNameStatic();
  }

  PluginProperties() : Properties() {
    m_collection_sp = std::make_shared<OptionValueProperties>(GetSettingName());
    m_collection_sp->Initialize(g_processkdp_properties_def);
  }

  ~PluginProperties() override = default;

  uint64_t GetPacketTimeout() {
    const uint32_t idx = ePropertyKDPPacketTimeout;
    return GetPropertyAtIndexAs<uint64_t>(
        idx, g_processkdp_properties[idx].default_uint_value);
  }
};

```
- **EN**: Introduces declarations for `PluginProperties`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `PluginProperties` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 82-95
```cpp
} // namespace

static PluginProperties &GetGlobalPluginProperties() {
  static PluginProperties g_settings;
  return g_settings;
}

static const lldb::tid_t g_kernel_tid = 1;

llvm::StringRef ProcessKDP::GetPluginDescriptionStatic() {
  return "KDP Remote protocol based debugging plug-in for darwin kernel "
         "debugging.";
}

```
- **EN**: Implements logic around `GetGlobalPluginProperties`, `GetPluginDescriptionStatic`.
- **CN**: 围绕 `GetGlobalPluginProperties`, `GetPluginDescriptionStatic` 实现具体逻辑。

### Lines 96-110
```cpp
void ProcessKDP::Terminate() {
  ProcessKDPLog::Terminate();
  PluginManager::UnregisterPlugin(ProcessKDP::CreateInstance);
}

lldb::ProcessSP ProcessKDP::CreateInstance(TargetSP target_sp,
                                           ListenerSP listener_sp,
                                           const FileSpec *crash_file_path,
                                           bool can_connect) {
  lldb::ProcessSP process_sp;
  if (crash_file_path == NULL)
    process_sp = std::make_shared<ProcessKDP>(target_sp, listener_sp);
  return process_sp;
}

```
- **EN**: Implements logic around `Terminate`, `UnregisterPlugin`, `CreateInstance`, `make_shared`.
- **CN**: 围绕 `Terminate`, `UnregisterPlugin`, `CreateInstance`, `make_shared` 实现具体逻辑。

### Lines 111-134
```cpp
bool ProcessKDP::CanDebug(TargetSP target_sp, bool plugin_specified_by_name) {
  if (plugin_specified_by_name)
    return true;

  // For now we are just making sure the file exists for a given module
  Module *exe_module = target_sp->GetExecutableModulePointer();
  if (exe_module) {
    const llvm::Triple &triple_ref = target_sp->GetArchitecture().GetTriple();
    switch (triple_ref.getOS()) {
    case llvm::Triple::Darwin: // Should use "macosx" for desktop and "ios" for
                               // iOS, but accept darwin just in case
    case llvm::Triple::MacOSX: // For desktop targets
    case llvm::Triple::IOS:    // For arm targets
    case llvm::Triple::TvOS:
    case llvm::Triple::WatchOS:
    case llvm::Triple::XROS:
      if (triple_ref.getVendor() == llvm::Triple::Apple) {
        ObjectFile *exe_objfile = exe_module->GetObjectFile();
        if (exe_objfile->GetType() == ObjectFile::eTypeExecutable &&
            exe_objfile->GetStrata() == ObjectFile::eStrataKernel)
          return true;
      }
      break;

```
- **EN**: Implements logic around `CanDebug`, `GetExecutableModulePointer`, `GetArchitecture`, `getOS`, and 4 more symbols; this block maps executable state back to modules, symbols, sections, or addresses; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `CanDebug`, `GetExecutableModulePointer`, `GetArchitecture`, `getOS`, and 4 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 135-158
```cpp
    default:
      break;
    }
  }
  return false;
}

// ProcessKDP constructor
ProcessKDP::ProcessKDP(TargetSP target_sp, ListenerSP listener_sp)
    : Process(target_sp, listener_sp),
      m_comm("lldb.process.kdp-remote.communication"),
      m_async_broadcaster(NULL, "lldb.process.kdp-remote.async-broadcaster"),
      m_kernel_load_addr(LLDB_INVALID_ADDRESS), m_command_sp(),
      m_kernel_thread_wp() {
  m_async_broadcaster.SetEventName(eBroadcastBitAsyncThreadShouldExit,
                                   "async thread should exit");
  m_async_broadcaster.SetEventName(eBroadcastBitAsyncContinue,
                                   "async thread continue");
  const uint64_t timeout_seconds =
      GetGlobalPluginProperties().GetPacketTimeout();
  if (timeout_seconds > 0)
    m_comm.SetPacketTimeout(std::chrono::seconds(timeout_seconds));
}

```
- **EN**: Implements logic around `ProcessKDP`, `Process`, `m_comm`, `m_async_broadcaster`, and 5 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `ProcessKDP`, `Process`, `m_comm`, `m_async_broadcaster`, and 5 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 159-175
```cpp
// Destructor
ProcessKDP::~ProcessKDP() {
  Clear();
  // We need to call finalize on the process before destroying ourselves to
  // make sure all of the broadcaster cleanup goes as planned. If we destruct
  // this class, then Process::~Process() might have problems trying to fully
  // destroy the broadcaster.
  Finalize(true /* destructing */);
}

Status ProcessKDP::DoWillLaunch(Module *module) {
  Status error;
  return Status::FromErrorString(
      "launching not supported in kdp-remote plug-in");
  return error;
}

```
- **EN**: Implements logic around `~ProcessKDP`, `Clear`, `Finalize`, `DoWillLaunch`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `~ProcessKDP`, `Clear`, `Finalize`, `DoWillLaunch`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址。

### Lines 176-190
```cpp
Status ProcessKDP::DoWillAttachToProcessWithID(lldb::pid_t pid) {
  Status error;
  return Status::FromErrorString(
      "attaching to a by process ID not supported in kdp-remote plug-in");
  return error;
}

Status ProcessKDP::DoWillAttachToProcessWithName(const char *process_name,
                                                 bool wait_for_launch) {
  Status error;
  return Status::FromErrorString(
      "attaching to a by process name not supported in kdp-remote plug-in");
  return error;
}

```
- **EN**: Implements logic around `DoWillAttachToProcessWithID`, `FromErrorString`, `DoWillAttachToProcessWithName`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `DoWillAttachToProcessWithID`, `FromErrorString`, `DoWillAttachToProcessWithName` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 191-204
```cpp
bool ProcessKDP::GetHostArchitecture(ArchSpec &arch) {
  uint32_t cpu = m_comm.GetCPUType();
  if (cpu) {
    uint32_t sub = m_comm.GetCPUSubtype();
    arch.SetArchitecture(eArchTypeMachO, cpu, sub);
    // Leave architecture vendor as unspecified unknown
    arch.GetTriple().setVendor(llvm::Triple::UnknownVendor);
    arch.GetTriple().setVendorName(llvm::StringRef());
    return true;
  }
  arch.Clear();
  return false;
}

```
- **EN**: Implements logic around `GetHostArchitecture`, `GetCPUType`, `GetCPUSubtype`, `SetArchitecture`, and 2 more symbols; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetHostArchitecture`, `GetCPUType`, `GetCPUSubtype`, `SetArchitecture`, and 2 more symbols 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 205-229
```cpp
Status ProcessKDP::DoConnectRemote(llvm::StringRef remote_url) {
  Status error;

  // Don't let any JIT happen when doing KDP as we can't allocate memory and we
  // don't want to be mucking with threads that might already be handling
  // exceptions
  SetCanJIT(false);

  if (remote_url.empty())
    return Status::FromErrorString("empty connection URL");

  std::unique_ptr<ConnectionFileDescriptor> conn_up(
      new ConnectionFileDescriptor());
  if (conn_up) {
    // Only try once for now.
    // TODO: check if we should be retrying?
    const uint32_t max_retry_count = 1;
    for (uint32_t retry_count = 0; retry_count < max_retry_count;
         ++retry_count) {
      if (conn_up->Connect(remote_url, &error) == eConnectionStatusSuccess)
        break;
      usleep(100000);
    }
  }

```
- **EN**: Implements logic around `DoConnectRemote`, `SetCanJIT`, `empty`, `FromErrorString`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics; supports expression parsing, wrapping, or debug-time code generation; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `DoConnectRemote`, `SetCanJIT`, `empty`, `FromErrorString`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并支持表达式解析、包装或调试期代码生成，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 230-252
```cpp
  if (conn_up->IsConnected()) {
    const TCPSocket &socket =
        static_cast<const TCPSocket &>(*conn_up->GetReadObject());
    const uint16_t reply_port = socket.GetLocalPortNumber();

    if (reply_port != 0) {
      m_comm.SetConnection(std::move(conn_up));

      if (m_comm.SendRequestReattach(reply_port)) {
        if (m_comm.SendRequestConnect(reply_port, reply_port,
                                      "Greetings from LLDB...")) {
          m_comm.GetVersion();

          Target &target = GetTarget();
          ArchSpec kernel_arch;
          // The host architecture
          GetHostArchitecture(kernel_arch);
          ArchSpec target_arch = target.GetArchitecture();
          // Merge in any unspecified stuff into the target architecture in
          // case the target arch isn't set at all or incompletely.
          target_arch.MergeFrom(kernel_arch);
          target.SetArchitecture(target_arch);

```
- **EN**: Implements logic around `IsConnected`, `GetReadObject`, `GetLocalPortNumber`, `SetConnection`, and 8 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; abstracts host OS resources such as files, terminals, sockets, or platforms; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `IsConnected`, `GetReadObject`, `GetLocalPortNumber`, `SetConnection`, and 8 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并抽象文件、终端、套接字或平台等主机操作系统资源，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 253-273
```cpp
          /* Get the kernel's UUID and load address via KDP_KERNELVERSION
           * packet.  */
          /* An EFI kdp session has neither UUID nor load address. */

          UUID kernel_uuid = m_comm.GetUUID();
          addr_t kernel_load_addr = m_comm.GetLoadAddress();

          if (m_comm.RemoteIsEFI()) {
            // Select an invalid plugin name for the dynamic loader so one
            // doesn't get used since EFI does its own manual loading via
            // python scripting
            m_dyld_plugin_name = "none";

            if (kernel_uuid.IsValid()) {
              // If EFI passed in a UUID= try to lookup UUID The slide will not
              // be provided. But the UUID lookup will be used to launch EFI
              // debug scripts from the dSYM, that can load all of the symbols.
              ModuleSpec module_spec;
              module_spec.GetUUID() = kernel_uuid;
              module_spec.GetArchitecture() = target.GetArchitecture();

```
- **EN**: Implements logic around `GetUUID`, `GetLoadAddress`, `RemoteIsEFI`, `IsValid`, and 1 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses; registers commands, plugins, or interpreter-facing extension points; uses Python scripting hooks to extend or automate LLDB.
- **CN**: 围绕 `GetUUID`, `GetLoadAddress`, `RemoteIsEFI`, `IsValid`, and 1 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址，并注册命令、插件或面向解释器的扩展点，并使用 Python 脚本钩子扩展或自动化 LLDB。

### Lines 274-298
```cpp
              // Lookup UUID locally, before attempting dsymForUUID like action
              FileSpecList search_paths =
                  Target::GetDefaultDebugFileSearchPaths();

              StatisticsMap symbol_locator_map;
              module_spec.GetSymbolFileSpec() =
                  PluginManager::LocateExecutableSymbolFile(
                      module_spec, search_paths, symbol_locator_map);
              if (module_spec.GetSymbolFileSpec()) {
                ModuleSpec executable_module_spec =
                    PluginManager::LocateExecutableObjectFile(
                        module_spec, symbol_locator_map);
                if (FileSystem::Instance().Exists(
                        executable_module_spec.GetFileSpec())) {
                  module_spec.GetFileSpec() =
                      executable_module_spec.GetFileSpec();
                }
              }
              if (!module_spec.GetSymbolFileSpec() ||
                  !module_spec.GetSymbolFileSpec()) {
                Status symbl_error;
                PluginManager::DownloadObjectAndSymbolFile(module_spec,
                                                           symbl_error, true);
              }

```
- **EN**: Implements logic around `GetDefaultDebugFileSearchPaths`, `GetSymbolFileSpec`, `LocateExecutableSymbolFile`, `LocateExecutableObjectFile`, and 3 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetDefaultDebugFileSearchPaths`, `GetSymbolFileSpec`, `LocateExecutableSymbolFile`, `LocateExecutableObjectFile`, and 3 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 299-321
```cpp
              if (FileSystem::Instance().Exists(module_spec.GetFileSpec())) {
                ModuleSP module_sp(new Module(module_spec));
                module_sp->GetSymbolLocatorStatistics().merge(
                    symbol_locator_map);
                if (module_sp.get() && module_sp->GetObjectFile()) {
                  // Get the current target executable
                  ModuleSP exe_module_sp(target.GetExecutableModule());

                  // Make sure you don't already have the right module loaded
                  // and they will be uniqued
                  if (exe_module_sp.get() != module_sp.get())
                    target.SetExecutableModule(module_sp, eLoadDependentsNo);
                }
              }
            }
          } else if (m_comm.RemoteIsDarwinKernel()) {
            m_dyld_plugin_name =
                DynamicLoaderDarwinKernel::GetPluginNameStatic();
            if (kernel_load_addr != LLDB_INVALID_ADDRESS) {
              m_kernel_load_addr = kernel_load_addr;
            }
          }

```
- **EN**: Implements logic around `Instance`, `module_sp`, `GetSymbolLocatorStatistics`, `get`, and 4 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `Instance`, `module_sp`, `GetSymbolLocatorStatistics`, `get`, and 4 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址。

### Lines 322-347
```cpp
          // Set the thread ID
          UpdateThreadListIfNeeded();
          SetID(1);
          GetThreadList();
          SetPrivateState(eStateStopped);
          const char *cstr;
          if ((cstr = m_comm.GetKernelVersion()) != NULL)
            target.GetDebugger().GetAsyncOutputStream()->Printf("Version: %s\n",
                                                                cstr);
        } else {
          return Status::FromErrorString("KDP_REATTACH failed");
        }
      } else {
        return Status::FromErrorString("KDP_REATTACH failed");
      }
    } else {
      return Status::FromErrorString("invalid reply port from UDP connection");
    }
  } else {
    if (error.Success())
      error = Status::FromErrorStringWithFormat("failed to connect to '%s'",
                                                remote_url.str().c_str());
  }
  if (error.Fail())
    m_comm.Disconnect();

```
- **EN**: Implements logic around `UpdateThreadListIfNeeded`, `SetID`, `GetThreadList`, `SetPrivateState`, and 8 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `UpdateThreadListIfNeeded`, `SetID`, `GetThreadList`, `SetPrivateState`, and 8 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 348-368
```cpp
  return error;
}

// Process Control
Status ProcessKDP::DoLaunch(Module *exe_module,
                            ProcessLaunchInfo &launch_info) {
  Status error;
  return Status::FromErrorString(
      "launching not supported in kdp-remote plug-in");
  return error;
}

Status
ProcessKDP::DoAttachToProcessWithID(lldb::pid_t attach_pid,
                                    const ProcessAttachInfo &attach_info) {
  Status error;
  return Status::FromErrorString(
      "attach to process by ID is not supported in kdp remote debugging");
  return error;
}

```
- **EN**: Implements logic around `DoLaunch`, `FromErrorString`, `DoAttachToProcessWithID`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `DoLaunch`, `FromErrorString`, `DoAttachToProcessWithID` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址。

### Lines 369-387
```cpp
Status
ProcessKDP::DoAttachToProcessWithName(const char *process_name,
                                      const ProcessAttachInfo &attach_info) {
  Status error;
  return Status::FromErrorString(
      "attach to process by name is not supported in kdp remote debugging");
  return error;
}

void ProcessKDP::DidAttach(ArchSpec &process_arch) {
  Process::DidAttach(process_arch);

  Log *log = GetLog(KDPLog::Process);
  LLDB_LOGF(log, "ProcessKDP::DidAttach()");
  if (GetID() != LLDB_INVALID_PROCESS_ID) {
    GetHostArchitecture(process_arch);
  }
}

```
- **EN**: Implements logic around `DoAttachToProcessWithName`, `FromErrorString`, `DidAttach`, `GetLog`, and 3 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `DoAttachToProcessWithName`, `FromErrorString`, `DidAttach`, `GetLog`, and 3 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 388-401
```cpp
addr_t ProcessKDP::GetImageInfoAddress() { return m_kernel_load_addr; }

lldb_private::DynamicLoader *ProcessKDP::GetDynamicLoader() {
  if (m_dyld_up.get() == NULL)
    m_dyld_up.reset(DynamicLoader::FindPlugin(this, m_dyld_plugin_name));
  return m_dyld_up.get();
}

Status ProcessKDP::WillResume() { return Status(); }

Status ProcessKDP::DoResume(RunDirection direction) {
  Status error;
  Log *log = GetLog(KDPLog::Process);

```
- **EN**: Implements logic around `GetImageInfoAddress`, `GetDynamicLoader`, `get`, `reset`, and 3 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetImageInfoAddress`, `GetDynamicLoader`, `get`, `reset`, and 3 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 402-418
```cpp
  if (direction == RunDirection::eRunReverse)
    return Status::FromErrorStringWithFormatv(
        "{0} does not support reverse execution of processes", GetPluginName());

  // Only start the async thread if we try to do any process control
  if (!m_async_thread.IsJoinable())
    StartAsyncThread();

  bool resume = false;

  // With KDP there is only one thread we can tell what to do
  ThreadSP kernel_thread_sp(m_thread_list.FindThreadByProtocolID(g_kernel_tid));

  if (kernel_thread_sp) {
    const StateType thread_resume_state =
        kernel_thread_sp->GetTemporaryResumeState();

```
- **EN**: Implements logic around `FromErrorStringWithFormatv`, `GetPluginName`, `IsJoinable`, `StartAsyncThread`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `FromErrorStringWithFormatv`, `GetPluginName`, `IsJoinable`, `StartAsyncThread`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 419-444
```cpp
    LLDB_LOGF(log, "ProcessKDP::DoResume() thread_resume_state = %s",
              StateAsCString(thread_resume_state));
    switch (thread_resume_state) {
    case eStateSuspended:
      // Nothing to do here when a thread will stay suspended we just leave the
      // CPU mask bit set to zero for the thread
      LLDB_LOGF(log, "ProcessKDP::DoResume() = suspended???");
      break;

    case eStateStepping: {
      lldb::RegisterContextSP reg_ctx_sp(
          kernel_thread_sp->GetRegisterContext());

      if (reg_ctx_sp) {
        LLDB_LOGF(
            log,
            "ProcessKDP::DoResume () reg_ctx_sp->HardwareSingleStep (true);");
        reg_ctx_sp->HardwareSingleStep(true);
        resume = true;
      } else {
        error = Status::FromErrorStringWithFormat(
            "KDP thread 0x%llx has no register context",
            kernel_thread_sp->GetID());
      }
    } break;

```
- **EN**: Implements logic around `LLDB_LOGF`, `StateAsCString`, `reg_ctx_sp`, `GetRegisterContext`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `LLDB_LOGF`, `StateAsCString`, `reg_ctx_sp`, `GetRegisterContext`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 445-460
```cpp
    case eStateRunning: {
      lldb::RegisterContextSP reg_ctx_sp(
          kernel_thread_sp->GetRegisterContext());

      if (reg_ctx_sp) {
        LLDB_LOGF(log, "ProcessKDP::DoResume () reg_ctx_sp->HardwareSingleStep "
                       "(false);");
        reg_ctx_sp->HardwareSingleStep(false);
        resume = true;
      } else {
        error = Status::FromErrorStringWithFormat(
            "KDP thread 0x%llx has no register context",
            kernel_thread_sp->GetID());
      }
    } break;

```
- **EN**: Implements logic around `reg_ctx_sp`, `GetRegisterContext`, `LLDB_LOGF`, `HardwareSingleStep`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `reg_ctx_sp`, `GetRegisterContext`, `LLDB_LOGF`, `HardwareSingleStep`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 461-478
```cpp
    default:
      // The only valid thread resume states are listed above
      llvm_unreachable("invalid thread resume state");
    }
  }

  if (resume) {
    LLDB_LOGF(log, "ProcessKDP::DoResume () sending resume");

    if (m_comm.SendRequestResume()) {
      m_async_broadcaster.BroadcastEvent(eBroadcastBitAsyncContinue);
      SetPrivateState(eStateRunning);
    } else
      return Status::FromErrorString("KDP resume failed");
  } else {
    return Status::FromErrorString("kernel thread is suspended");
  }

```
- **EN**: Implements logic around `llvm_unreachable`, `LLDB_LOGF`, `SendRequestResume`, `BroadcastEvent`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `llvm_unreachable`, `LLDB_LOGF`, `SendRequestResume`, `BroadcastEvent`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 479-493
```cpp
  return error;
}

lldb::ThreadSP ProcessKDP::GetKernelThread() {
  // KDP only tells us about one thread/core. Any other threads will usually
  // be the ones that are read from memory by the OS plug-ins.

  ThreadSP thread_sp(m_kernel_thread_wp.lock());
  if (!thread_sp) {
    thread_sp = std::make_shared<ThreadKDP>(*this, g_kernel_tid);
    m_kernel_thread_wp = thread_sp;
  }
  return thread_sp;
}

```
- **EN**: Implements logic around `GetKernelThread`, `thread_sp`, `make_shared`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetKernelThread`, `thread_sp`, `make_shared` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 494-507
```cpp
bool ProcessKDP::DoUpdateThreadList(ThreadList &old_thread_list,
                                    ThreadList &new_thread_list) {
  // locker will keep a mutex locked until it goes out of scope
  Log *log = GetLog(KDPLog::Thread);
  LLDB_LOG_VERBOSE(log, "pid = {0}", GetID());

  // Even though there is a CPU mask, it doesn't mean we can see each CPU
  // individually, there is really only one. Lets call this thread 1.
  ThreadSP thread_sp(
      old_thread_list.FindThreadByProtocolID(g_kernel_tid, false));
  if (!thread_sp)
    thread_sp = GetKernelThread();
  new_thread_list.AddThread(thread_sp);

```
- **EN**: Implements logic around `DoUpdateThreadList`, `GetLog`, `LLDB_LOG_VERBOSE`, `thread_sp`, and 3 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `DoUpdateThreadList`, `GetLog`, `LLDB_LOG_VERBOSE`, `thread_sp`, and 3 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 508-532
```cpp
  return new_thread_list.GetSize(false) > 0;
}

void ProcessKDP::RefreshStateAfterStop() {
  // Let all threads recover from stopping and do any clean up based on the
  // previous thread state (if any).
  m_thread_list.RefreshStateAfterStop();
}

Status ProcessKDP::DoHalt(bool &caused_stop) {
  Status error;

  if (m_comm.IsRunning()) {
    if (m_destroy_in_process) {
      // If we are attempting to destroy, we need to not return an error to Halt
      // or DoDestroy won't get called. We are also currently running, so send
      // a process stopped event
      SetPrivateState(eStateStopped);
    } else {
      return Status::FromErrorString("KDP cannot interrupt a running kernel");
    }
  }
  return error;
}

```
- **EN**: Implements logic around `GetSize`, `RefreshStateAfterStop`, `DoHalt`, `IsRunning`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetSize`, `RefreshStateAfterStop`, `DoHalt`, `IsRunning`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 533-559
```cpp
Status ProcessKDP::DoDetach(bool keep_stopped) {
  Status error;
  Log *log = GetLog(KDPLog::Process);
  LLDB_LOGF(log, "ProcessKDP::DoDetach(keep_stopped = %i)", keep_stopped);

  if (m_comm.IsRunning()) {
    // We are running and we can't interrupt a running kernel, so we need to
    // just close the connection to the kernel and hope for the best
  } else {
    // If we are going to keep the target stopped, then don't send the
    // disconnect message.
    if (!keep_stopped && m_comm.IsConnected()) {
      const bool success = m_comm.SendRequestDisconnect();
      if (log) {
        if (success)
          log->PutCString(
              "ProcessKDP::DoDetach() detach packet sent successfully");
        else
          log->PutCString(
              "ProcessKDP::DoDetach() connection channel shutdown failed");
      }
      m_comm.Disconnect();
    }
  }
  StopAsyncThread();
  m_comm.Clear();

```
- **EN**: Implements logic around `DoDetach`, `GetLog`, `LLDB_LOGF`, `IsRunning`, and 6 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `DoDetach`, `GetLog`, `LLDB_LOGF`, `IsRunning`, and 6 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 560-574
```cpp
  SetPrivateState(eStateDetached);
  ResumePrivateStateThread();

  // KillDebugserverProcess ();
  return error;
}

Status ProcessKDP::DoDestroy() {
  // For KDP there really is no difference between destroy and detach
  bool keep_stopped = false;
  return DoDetach(keep_stopped);
}

// Process Queries

```
- **EN**: Implements logic around `SetPrivateState`, `ResumePrivateStateThread`, `DoDestroy`, `DoDetach`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `SetPrivateState`, `ResumePrivateStateThread`, `DoDestroy`, `DoDetach` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 575-601
```cpp
bool ProcessKDP::IsAlive() {
  return m_comm.IsConnected() && Process::IsAlive();
}

// Process Memory
size_t ProcessKDP::DoReadMemory(addr_t addr, void *buf, size_t size,
                                Status &error) {
  uint8_t *data_buffer = (uint8_t *)buf;
  if (m_comm.IsConnected()) {
    const size_t max_read_size = 512;
    size_t total_bytes_read = 0;

    // Read the requested amount of memory in 512 byte chunks
    while (total_bytes_read < size) {
      size_t bytes_to_read_this_request = size - total_bytes_read;
      if (bytes_to_read_this_request > max_read_size) {
        bytes_to_read_this_request = max_read_size;
      }
      size_t bytes_read = m_comm.SendRequestReadMemory(
          addr + total_bytes_read, data_buffer + total_bytes_read,
          bytes_to_read_this_request, error);
      total_bytes_read += bytes_read;
      if (error.Fail() || bytes_read == 0) {
        return total_bytes_read;
      }
    }

```
- **EN**: Implements logic around `IsAlive`, `IsConnected`, `DoReadMemory`, `SendRequestReadMemory`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `IsAlive`, `IsConnected`, `DoReadMemory`, `SendRequestReadMemory`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 602-615
```cpp
    return total_bytes_read;
  }
  error = Status::FromErrorString("not connected");
  return 0;
}

size_t ProcessKDP::DoWriteMemory(addr_t addr, const void *buf, size_t size,
                                 Status &error) {
  if (m_comm.IsConnected())
    return m_comm.SendRequestWriteMemory(addr, buf, size, error);
  error = Status::FromErrorString("not connected");
  return 0;
}

```
- **EN**: Implements logic around `FromErrorString`, `DoWriteMemory`, `IsConnected`, `SendRequestWriteMemory`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `FromErrorString`, `DoWriteMemory`, `IsConnected`, `SendRequestWriteMemory` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 616-629
```cpp
lldb::addr_t ProcessKDP::DoAllocateMemory(size_t size, uint32_t permissions,
                                          Status &error) {
  error = Status::FromErrorString(
      "memory allocation not supported in kdp remote debugging");
  return LLDB_INVALID_ADDRESS;
}

Status ProcessKDP::DoDeallocateMemory(lldb::addr_t addr) {
  Status error;
  return Status::FromErrorString(
      "memory deallocation not supported in kdp remote debugging");
  return error;
}

```
- **EN**: Implements logic around `DoAllocateMemory`, `FromErrorString`, `DoDeallocateMemory`; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `DoAllocateMemory`, `FromErrorString`, `DoDeallocateMemory` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 630-648
```cpp
Status ProcessKDP::EnableBreakpointSite(BreakpointSite *bp_site) {
  if (bp_site->HardwareRequired())
    return Status::FromErrorString("Hardware breakpoints are not supported.");

  if (m_comm.LocalBreakpointsAreSupported()) {
    Status error;
    if (!IsBreakpointSitePhysicallyEnabled(*bp_site)) {
      if (m_comm.SendRequestBreakpoint(true, bp_site->GetLoadAddress())) {
        SetBreakpointSiteEnabled(*bp_site);
        bp_site->SetType(BreakpointSite::eExternal);
      } else {
        return Status::FromErrorString("KDP set breakpoint failed");
      }
    }
    return error;
  }
  return EnableSoftwareBreakpoint(bp_site);
}

```
- **EN**: Implements logic around `EnableBreakpointSite`, `HardwareRequired`, `FromErrorString`, `LocalBreakpointsAreSupported`, and 5 more symbols; this block propagates recoverable errors, status objects, or diagnostics; tracks breakpoint state, stop conditions, or hit-processing policy.
- **CN**: 围绕 `EnableBreakpointSite`, `HardwareRequired`, `FromErrorString`, `LocalBreakpointsAreSupported`, and 5 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并跟踪断点状态、停机条件或命中处理策略。

### Lines 649-672
```cpp
Status ProcessKDP::DisableBreakpointSite(BreakpointSite *bp_site) {
  if (m_comm.LocalBreakpointsAreSupported()) {
    Status error;
    if (IsBreakpointSitePhysicallyEnabled(*bp_site)) {
      BreakpointSite::Type bp_type = bp_site->GetType();
      if (bp_type == BreakpointSite::eExternal) {
        if (m_destroy_in_process && m_comm.IsRunning()) {
          // We are trying to destroy our connection and we are running
          SetBreakpointSiteEnabled(*bp_site, false);
        } else {
          if (m_comm.SendRequestBreakpoint(false, bp_site->GetLoadAddress()))
            SetBreakpointSiteEnabled(*bp_site, false);
          else
            return Status::FromErrorString("KDP remove breakpoint failed");
        }
      } else {
        error = DisableSoftwareBreakpoint(bp_site);
      }
    }
    return error;
  }
  return DisableSoftwareBreakpoint(bp_site);
}

```
- **EN**: Implements logic around `DisableBreakpointSite`, `LocalBreakpointsAreSupported`, `IsBreakpointSitePhysicallyEnabled`, `GetType`, and 5 more symbols; this block propagates recoverable errors, status objects, or diagnostics; tracks breakpoint state, stop conditions, or hit-processing policy.
- **CN**: 围绕 `DisableBreakpointSite`, `LocalBreakpointsAreSupported`, `IsBreakpointSitePhysicallyEnabled`, `GetType`, and 5 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并跟踪断点状态、停机条件或命中处理策略。

### Lines 673-686
```cpp
void ProcessKDP::Clear() { m_thread_list.Clear(); }

Status ProcessKDP::DoSignal(int signo) {
  Status error;
  return Status::FromErrorString(
      "sending signals is not supported in kdp remote debugging");
  return error;
}

void ProcessKDP::Initialize() {
  PluginManager::RegisterPlugin(GetPluginNameStatic(),
                                GetPluginDescriptionStatic(), CreateInstance,
                                DebuggerInitialize);

```
- **EN**: Implements logic around `Clear`, `DoSignal`, `FromErrorString`, `Initialize`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `Clear`, `DoSignal`, `FromErrorString`, `Initialize`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 687-702
```cpp
  ProcessKDPLog::Initialize();
}

void ProcessKDP::DebuggerInitialize(lldb_private::Debugger &debugger) {
  if (!PluginManager::GetSettingForProcessPlugin(
          debugger, PluginProperties::GetSettingName())) {
    const bool is_global_setting = true;
    PluginManager::CreateSettingForProcessPlugin(
        debugger, GetGlobalPluginProperties().GetValueProperties(),
        "Properties for the kdp-remote process plug-in.", is_global_setting);
  }
}

bool ProcessKDP::StartAsyncThread() {
  Log *log = GetLog(KDPLog::Process);

```
- **EN**: Implements logic around `Initialize`, `DebuggerInitialize`, `GetSettingForProcessPlugin`, `GetSettingName`, and 4 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `Initialize`, `DebuggerInitialize`, `GetSettingForProcessPlugin`, `GetSettingName`, and 4 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 703-718
```cpp
  LLDB_LOGF(log, "ProcessKDP::StartAsyncThread ()");

  if (m_async_thread.IsJoinable())
    return true;

  llvm::Expected<HostThread> async_thread = ThreadLauncher::LaunchThread(
      "<lldb.process.kdp-remote.async>", [this] { return AsyncThread(); });
  if (!async_thread) {
    LLDB_LOG_ERROR(GetLog(LLDBLog::Host), async_thread.takeError(),
                   "failed to launch host thread: {0}");
    return false;
  }
  m_async_thread = *async_thread;
  return m_async_thread.IsJoinable();
}

```
- **EN**: Implements logic around `LLDB_LOGF`, `IsJoinable`, `LaunchThread`, `AsyncThread`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `LLDB_LOGF`, `IsJoinable`, `LaunchThread`, `AsyncThread`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 719-733
```cpp
void ProcessKDP::StopAsyncThread() {
  Log *log = GetLog(KDPLog::Process);

  LLDB_LOGF(log, "ProcessKDP::StopAsyncThread ()");

  m_async_broadcaster.BroadcastEvent(eBroadcastBitAsyncThreadShouldExit);

  // Stop the stdio thread
  if (m_async_thread.IsJoinable())
    m_async_thread.Join(nullptr);
}

void *ProcessKDP::AsyncThread() {
  const lldb::pid_t pid = GetID();

```
- **EN**: Implements logic around `StopAsyncThread`, `GetLog`, `LLDB_LOGF`, `BroadcastEvent`, and 4 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `StopAsyncThread`, `GetLog`, `LLDB_LOGF`, `BroadcastEvent`, and 4 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 734-758
```cpp
  Log *log = GetLog(KDPLog::Process);
  LLDB_LOGF(log,
            "ProcessKDP::AsyncThread(pid = %" PRIu64 ") thread starting...",
            pid);

  ListenerSP listener_sp(Listener::MakeListener("ProcessKDP::AsyncThread"));
  EventSP event_sp;
  const uint32_t desired_event_mask =
      eBroadcastBitAsyncContinue | eBroadcastBitAsyncThreadShouldExit;

  if (listener_sp->StartListeningForEvents(
          &m_async_broadcaster, desired_event_mask) == desired_event_mask) {
    bool done = false;
    while (!done) {
      LLDB_LOGF(log,
                "ProcessKDP::AsyncThread (pid = %" PRIu64
                ") listener.WaitForEvent (NULL, event_sp)...",
                pid);
      if (listener_sp->GetEvent(event_sp, std::nullopt)) {
        uint32_t event_type = event_sp->GetType();
        LLDB_LOGF(log,
                  "ProcessKDP::AsyncThread (pid = %" PRIu64
                  ") Got an event of type: %d...",
                  pid, event_type);

```
- **EN**: Implements logic around `GetLog`, `LLDB_LOGF`, `AsyncThread`, `listener_sp`, and 4 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetLog`, `LLDB_LOGF`, `AsyncThread`, `listener_sp`, and 4 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 759-779
```cpp
        // When we are running, poll for 1 second to try and get an exception
        // to indicate the process has stopped. If we don't get one, check to
        // make sure no one asked us to exit
        bool is_running = false;
        DataExtractor exc_reply_packet;
        do {
          switch (event_type) {
          case eBroadcastBitAsyncContinue: {
            is_running = true;
            if (m_comm.WaitForPacketWithTimeoutMicroSeconds(
                    exc_reply_packet, 1 * USEC_PER_SEC)) {
              ThreadSP thread_sp(GetKernelThread());
              if (thread_sp) {
                lldb::RegisterContextSP reg_ctx_sp(
                    thread_sp->GetRegisterContext());
                if (reg_ctx_sp)
                  reg_ctx_sp->InvalidateAllRegisters();
                static_cast<ThreadKDP *>(thread_sp.get())
                    ->SetStopInfoFrom_KDP_EXCEPTION(exc_reply_packet);
              }

```
- **EN**: Implements logic around `WaitForPacketWithTimeoutMicroSeconds`, `thread_sp`, `reg_ctx_sp`, `GetRegisterContext`, and 3 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `WaitForPacketWithTimeoutMicroSeconds`, `thread_sp`, `reg_ctx_sp`, `GetRegisterContext`, and 3 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 780-794
```cpp
              // TODO: parse the stop reply packet
              is_running = false;
              SetPrivateState(eStateStopped);
            } else {
              // Check to see if we are supposed to exit. There is no way to
              // interrupt a running kernel, so all we can do is wait for an
              // exception or detach...
              if (listener_sp->GetEvent(event_sp,
                                        std::chrono::microseconds(0))) {
                // We got an event, go through the loop again
                event_type = event_sp->GetType();
              }
            }
          } break;

```
- **EN**: Implements logic around `SetPrivateState`, `GetEvent`, `microseconds`, `GetType`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `SetPrivateState`, `GetEvent`, `microseconds`, `GetType` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 795-822
```cpp
          case eBroadcastBitAsyncThreadShouldExit:
            LLDB_LOGF(log,
                      "ProcessKDP::AsyncThread (pid = %" PRIu64
                      ") got eBroadcastBitAsyncThreadShouldExit...",
                      pid);
            done = true;
            is_running = false;
            break;

          default:
            LLDB_LOGF(log,
                      "ProcessKDP::AsyncThread (pid = %" PRIu64
                      ") got unknown event 0x%8.8x",
                      pid, event_type);
            done = true;
            is_running = false;
            break;
          }
        } while (is_running);
      } else {
        LLDB_LOGF(log,
                  "ProcessKDP::AsyncThread (pid = %" PRIu64
                  ") listener.WaitForEvent (NULL, event_sp) => false",
                  pid);
        done = true;
      }
    }
  }
```
- **EN**: Implements logic around `LLDB_LOGF`, `AsyncThread`, `WaitForEvent`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `LLDB_LOGF`, `AsyncThread`, `WaitForEvent` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 823-836
```cpp

  LLDB_LOGF(log, "ProcessKDP::AsyncThread(pid = %" PRIu64 ") thread exiting...",
            pid);

  m_async_thread.Reset();
  return NULL;
}

class CommandObjectProcessKDPPacketSend : public CommandObjectParsed {
private:
  OptionGroupOptions m_option_group;
  OptionGroupUInt64 m_command_byte;
  OptionGroupString m_packet_data;

```
- **EN**: Introduces declarations for `CommandObjectProcessKDPPacketSend`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `CommandObjectProcessKDPPacketSend` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 837-861
```cpp
  Options *GetOptions() override { return &m_option_group; }

public:
  CommandObjectProcessKDPPacketSend(CommandInterpreter &interpreter)
      : CommandObjectParsed(interpreter, "process plugin packet send",
                            "Send a custom packet through the KDP protocol by "
                            "specifying the command byte and the packet "
                            "payload data. A packet will be sent with a "
                            "correct header and payload, and the raw result "
                            "bytes will be displayed as a string value. ",
                            NULL),
        m_option_group(),
        m_command_byte(LLDB_OPT_SET_1, true, "command", 'c', 0, eArgTypeNone,
                       "Specify the command byte to use when sending the KDP "
                       "request packet.",
                       0),
        m_packet_data(LLDB_OPT_SET_1, false, "payload", 'p', 0, eArgTypeNone,
                      "Specify packet payload bytes as a hex ASCII string with "
                      "no spaces or hex prefixes.",
                      NULL) {
    m_option_group.Append(&m_command_byte, LLDB_OPT_SET_ALL, LLDB_OPT_SET_1);
    m_option_group.Append(&m_packet_data, LLDB_OPT_SET_ALL, LLDB_OPT_SET_1);
    m_option_group.Finalize();
  }

```
- **EN**: Implements logic around `GetOptions`, `CommandObjectProcessKDPPacketSend`, `CommandObjectParsed`, `m_option_group`, and 4 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `GetOptions`, `CommandObjectProcessKDPPacketSend`, `CommandObjectParsed`, `m_option_group`, and 4 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并注册命令、插件或面向解释器的扩展点。

### Lines 862-876
```cpp
  ~CommandObjectProcessKDPPacketSend() override = default;

  void DoExecute(Args &command, CommandReturnObject &result) override {
    if (!m_command_byte.GetOptionValue().OptionWasSet()) {
      result.AppendError(
          "the --command option must be set to a valid command byte");
    } else {
      const uint64_t command_byte =
          m_command_byte.GetOptionValue().GetValueAs<uint64_t>().value_or(0);
      if (command_byte > 0 && command_byte <= UINT8_MAX) {
        ProcessKDP *process =
            (ProcessKDP *)m_interpreter.GetExecutionContext().GetProcessPtr();
        if (process) {
          const StateType state = process->GetState();

```
- **EN**: Implements logic around `~CommandObjectProcessKDPPacketSend`, `DoExecute`, `GetOptionValue`, `AppendError`, and 2 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; registers commands, plugins, or interpreter-facing extension points; defines user-visible settings, options, or policy flags.
- **CN**: 围绕 `~CommandObjectProcessKDPPacketSend`, `DoExecute`, `GetOptionValue`, `AppendError`, and 2 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并注册命令、插件或面向解释器的扩展点，并定义用户可见的设置、选项或策略标志。

### Lines 877-904
```cpp
          if (StateIsStoppedState(state, true)) {
            std::vector<uint8_t> payload_bytes;
            const char *ascii_hex_bytes_cstr =
                m_packet_data.GetOptionValue().GetCurrentValue();
            if (ascii_hex_bytes_cstr && ascii_hex_bytes_cstr[0]) {
              StringExtractor extractor(ascii_hex_bytes_cstr);
              const size_t ascii_hex_bytes_cstr_len =
                  extractor.GetStringRef().size();
              if (ascii_hex_bytes_cstr_len & 1) {
                result.AppendErrorWithFormat("payload data must contain an "
                                             "even number of ASCII hex "
                                             "characters: '%s'",
                                             ascii_hex_bytes_cstr);
                return;
              }
              payload_bytes.resize(ascii_hex_bytes_cstr_len / 2);
              if (extractor.GetHexBytes(payload_bytes, '\xdd') !=
                  payload_bytes.size()) {
                result.AppendErrorWithFormat("payload data must only contain "
                                             "ASCII hex characters (no "
                                             "spaces or hex prefixes): '%s'",
                                             ascii_hex_bytes_cstr);
                return;
              }
            }
            Status error;
            DataExtractor reply;
            process->GetCommunication().SendRawRequest(
```
- **EN**: Implements logic around `StateIsStoppedState`, `GetOptionValue`, `extractor`, `GetStringRef`, and 6 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `StateIsStoppedState`, `GetOptionValue`, `extractor`, `GetStringRef`, and 6 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 905-932
```cpp
                command_byte,
                payload_bytes.empty() ? NULL : payload_bytes.data(),
                payload_bytes.size(), reply, error);

            if (error.Success()) {
              // Copy the binary bytes into a hex ASCII string for the result
              StreamString packet;
              packet.PutBytesAsRawHex8(
                  reply.GetDataStart(), reply.GetByteSize(),
                  endian::InlHostByteOrder(), endian::InlHostByteOrder());
              result.AppendMessage(packet.GetString());
              result.SetStatus(eReturnStatusSuccessFinishResult);
              return;
            } else {
              const char *error_cstr = error.AsCString();
              if (error_cstr && error_cstr[0])
                result.AppendError(error_cstr);
              else
                result.AppendErrorWithFormat("unknown error 0x%8.8x",
                                             error.GetError());
              return;
            }
          } else {
            result.AppendErrorWithFormat("process must be stopped in order "
                                         "to send KDP packets, state is %s",
                                         StateAsCString(state));
          }
        } else {
```
- **EN**: Implements logic around `empty`, `size`, `Success`, `PutBytesAsRawHex8`, and 9 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `empty`, `size`, `Success`, `PutBytesAsRawHex8`, and 9 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 933-946
```cpp
          result.AppendError("invalid process");
        }
      } else {
        result.AppendErrorWithFormat("invalid command byte 0x%" PRIx64
                                     ", valid values are 1 - 255",
                                     command_byte);
      }
    }
  }
};

class CommandObjectProcessKDPPacket : public CommandObjectMultiword {
private:
public:
```
- **EN**: Introduces declarations for `CommandObjectProcessKDPPacket`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `CommandObjectProcessKDPPacket` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 947-960
```cpp
  CommandObjectProcessKDPPacket(CommandInterpreter &interpreter)
      : CommandObjectMultiword(interpreter, "process plugin packet",
                               "Commands that deal with KDP remote packets.",
                               NULL) {
    LoadSubCommand(
        "send",
        CommandObjectSP(new CommandObjectProcessKDPPacketSend(interpreter)));
  }

  ~CommandObjectProcessKDPPacket() override = default;
};

class CommandObjectMultiwordProcessKDP : public CommandObjectMultiword {
public:
```
- **EN**: Introduces declarations for `CommandObjectMultiwordProcessKDP`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `CommandObjectMultiwordProcessKDP` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 961-978
```cpp
  CommandObjectMultiwordProcessKDP(CommandInterpreter &interpreter)
      : CommandObjectMultiword(
            interpreter, "process plugin",
            "Commands for operating on a ProcessKDP process.",
            "process plugin <subcommand> [<subcommand-options>]") {
    LoadSubCommand("packet", CommandObjectSP(new CommandObjectProcessKDPPacket(
                                 interpreter)));
  }

  ~CommandObjectMultiwordProcessKDP() override = default;
};

CommandObject *ProcessKDP::GetPluginCommandObject() {
  if (!m_command_sp)
    m_command_sp = std::make_shared<CommandObjectMultiwordProcessKDP>(
        GetTarget().GetDebugger().GetCommandInterpreter());
  return m_command_sp.get();
}
```
- **EN**: Implements logic around `CommandObjectMultiwordProcessKDP`, `CommandObjectMultiword`, `LoadSubCommand`, `~CommandObjectMultiwordProcessKDP`, and 4 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `CommandObjectMultiwordProcessKDP`, `CommandObjectMultiword`, `LoadSubCommand`, `~CommandObjectMultiwordProcessKDP`, and 4 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并注册命令、插件或面向解释器的扩展点。

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
- **OS integration / 操作系统集成**:
  - **EN**: Bridges LLDB abstractions to host files, sockets, terminals, and native process facilities.
  - **CN**: 将 LLDB 抽象连接到主机文件、套接字、终端与本地进程设施。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `lldb/Core/Debugger.h`, `lldb/Core/Module.h`, `lldb/Core/ModuleSpec.h`, `lldb/Core/PluginManager.h`, `lldb/Host/ConnectionFileDescriptor.h`, `lldb/Host/Host.h`, `lldb/Host/ThreadLauncher.h`, `lldb/Host/common/TCPSocket.h`, `lldb/Interpreter/CommandInterpreter.h`, `lldb/Interpreter/CommandObject.h` ... (+21 more)
- **Standard-library headers / 标准库头文件**: `<cerrno>`, `<cstdlib>`, `<memory>`
- **Subsystem categories / 子系统类别**: command interpreter support / 命令解释器支持 (7), shared LLDB utility classes / 共享 LLDB 工具类 (5), LLDB core debugger abstractions / LLDB 核心调试抽象 (4), host OS abstraction helpers / 主机操作系统抽象辅助组件 (4), target, process, and thread control / 目标、进程与线程控制 (3), symbol and debug-info abstractions / 符号与调试信息抽象 (1)
