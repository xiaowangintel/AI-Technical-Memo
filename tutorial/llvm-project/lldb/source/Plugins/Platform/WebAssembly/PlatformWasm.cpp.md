# PlatformWasm.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Platform/WebAssembly/PlatformWasm.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `PlatformWasm`.
  - **CN**: 实现与 `PlatformWasm` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===----------------------------------------------------------------------===//
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

#include "Plugins/Platform/WebAssembly/PlatformWasm.h"
#include "Plugins/Platform/WebAssembly/PlatformWasmRemoteGDBServer.h"
#include "Plugins/Platform/WebAssembly/PlatformWebInspectorWasm.h"
#include "Plugins/Process/wasm/ProcessWasm.h"
#include "lldb/Core/PluginManager.h"
#include "lldb/Host/FileSystem.h"
#include "lldb/Host/ProcessLaunchInfo.h"
#include "lldb/Host/common/TCPSocket.h"
#include "lldb/Interpreter/OptionValueProperties.h"
#include "lldb/Target/Process.h"
#include "lldb/Target/Target.h"
#include "lldb/Utility/LLDBLog.h"
#include "lldb/Utility/Listener.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `Plugins/Platform/WebAssembly/PlatformWasm.h`, `Plugins/Platform/WebAssembly/PlatformWasmRemoteGDBServer.h`, `Plugins/Platform/WebAssembly/PlatformWebInspectorWasm.h`, `Plugins/Process/wasm/ProcessWasm.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `Plugins/Platform/WebAssembly/PlatformWasm.h`, `Plugins/Platform/WebAssembly/PlatformWasmRemoteGDBServer.h`, `Plugins/Platform/WebAssembly/PlatformWebInspectorWasm.h`, `Plugins/Process/wasm/ProcessWasm.h`。

### Lines 22-28
```cpp
#include "lldb/Utility/Log.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/Support/ErrorExtras.h"

using namespace lldb;
using namespace lldb_private;

```
- **EN**: Pulls in the headers needed by this translation unit, including `lldb/Utility/Log.h`, `llvm/ADT/StringExtras.h`, `llvm/Support/ErrorExtras.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `lldb/Utility/Log.h`, `llvm/ADT/StringExtras.h`, `llvm/Support/ErrorExtras.h`。

### Lines 29-35
```cpp
LLDB_PLUGIN_DEFINE(PlatformWasm)

namespace {
#define LLDB_PROPERTIES_platformwasm
#include "PlatformWasmProperties.inc"

enum {
```
- **EN**: Pulls in the headers needed by this translation unit, including `PlatformWasmProperties.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `PlatformWasmProperties.inc`。

### Lines 36-47
```cpp
#define LLDB_PROPERTIES_platformwasm
#include "PlatformWasmPropertiesEnum.inc"
};

class PluginProperties : public Properties {
public:
  PluginProperties() {
    m_collection_sp = std::make_shared<OptionValueProperties>(
        PlatformWasm::GetPluginNameStatic());
    m_collection_sp->Initialize(g_platformwasm_properties_def);
  }

```
- **EN**: Pulls in the headers needed by this translation unit, including `PlatformWasmPropertiesEnum.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `PlatformWasmPropertiesEnum.inc`。

### Lines 48-57
```cpp
  FileSpec GetRuntimePath() const {
    return GetPropertyAtIndexAs<FileSpec>(ePropertyRuntimePath, {});
  }

  Args GetRuntimeArgs() const {
    Args result;
    m_collection_sp->GetPropertyAtIndexAsArgs(ePropertyRuntimeArgs, result);
    return result;
  }

```
- **EN**: Implements logic around `GetRuntimePath`, `GetPropertyAtIndexAs`, `GetRuntimeArgs`, `GetPropertyAtIndexAsArgs`.
- **CN**: 围绕 `GetRuntimePath`, `GetPropertyAtIndexAs`, `GetRuntimeArgs`, `GetPropertyAtIndexAsArgs` 实现具体逻辑。

### Lines 58-64
```cpp
  llvm::StringRef GetPortArg() const {
    return GetPropertyAtIndexAs<llvm::StringRef>(ePropertyPortArg, {});
  }
};

} // namespace

```
- **EN**: Implements logic around `GetPortArg`, `StringRef>`.
- **CN**: 围绕 `GetPortArg`, `StringRef>` 实现具体逻辑。

### Lines 65-73
```cpp
static PluginProperties &GetGlobalProperties() {
  static PluginProperties g_settings;
  return g_settings;
}

llvm::StringRef PlatformWasm::GetPluginDescriptionStatic() {
  return "Platform for debugging Wasm";
}

```
- **EN**: Implements logic around `GetGlobalProperties`, `GetPluginDescriptionStatic`; this block abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `GetGlobalProperties`, `GetPluginDescriptionStatic` 实现具体逻辑；该代码块抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 74-80
```cpp
void PlatformWasm::Initialize() {
  PlatformWebInspectorWasm::Initialize();
  PluginManager::RegisterPlugin(
      GetPluginNameStatic(), GetPluginDescriptionStatic(),
      PlatformWasm::CreateInstance, PlatformWasm::DebuggerInitialize);
}

```
- **EN**: Implements logic around `Initialize`, `RegisterPlugin`, `GetPluginNameStatic`.
- **CN**: 围绕 `Initialize`, `RegisterPlugin`, `GetPluginNameStatic` 实现具体逻辑。

### Lines 81-94
```cpp
void PlatformWasm::Terminate() {
  PluginManager::UnregisterPlugin(PlatformWasm::CreateInstance);
  PlatformWebInspectorWasm::Terminate();
}

void PlatformWasm::DebuggerInitialize(Debugger &debugger) {
  if (!PluginManager::GetSettingForPlatformPlugin(debugger,
                                                  GetPluginNameStatic())) {
    PluginManager::CreateSettingForPlatformPlugin(
        debugger, GetGlobalProperties().GetValueProperties(),
        "Properties for the wasm platform plugin.",
        /*is_global_property=*/true);
  }
}
```
- **EN**: Implements logic around `Terminate`, `UnregisterPlugin`, `DebuggerInitialize`, `GetSettingForPlatformPlugin`, and 3 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; abstracts host OS resources such as files, terminals, sockets, or platforms; registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `Terminate`, `UnregisterPlugin`, `DebuggerInitialize`, `GetSettingForPlatformPlugin`, and 3 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并抽象文件、终端、套接字或平台等主机操作系统资源，并注册命令、插件或面向解释器的扩展点。

### Lines 95-101
```cpp

PlatformSP PlatformWasm::CreateInstance(bool force, const ArchSpec *arch) {
  Log *log = GetLog(LLDBLog::Platform);
  LLDB_LOG(log, "force = {0}, arch = ({1}, {2})", force,
           arch ? arch->GetArchitectureName() : "<null>",
           arch ? arch->GetTriple().getTriple() : "<null>");

```
- **EN**: Implements logic around `CreateInstance`, `GetLog`, `LLDB_LOG`, `GetArchitectureName`, and 1 more symbols; this block abstracts host OS resources such as files, terminals, sockets, or platforms; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `CreateInstance`, `GetLog`, `LLDB_LOG`, `GetArchitectureName`, and 1 more symbols 实现具体逻辑；该代码块抽象文件、终端、套接字或平台等主机操作系统资源，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 102-114
```cpp
  bool create = force;
  if (!create && arch && arch->IsValid()) {
    const llvm::Triple &triple = arch->GetTriple();
    switch (triple.getArch()) {
    case llvm::Triple::wasm32:
    case llvm::Triple::wasm64:
      create = true;
      break;
    default:
      break;
    }
  }

```
- **EN**: Implements logic around `IsValid`, `GetTriple`, `getArch`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `IsValid`, `GetTriple`, `getArch` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 115-126
```cpp
  LLDB_LOG(log, "create = {0}", create);
  return create ? PlatformSP(new PlatformWasm()) : PlatformSP();
}

llvm::Expected<uint16_t> PlatformWasm::FindFreeTCPPort() {
  TCPSocket sock(/*should_close=*/true);
  Status status = sock.Listen("localhost:0", /*backlog=*/5);
  if (status.Fail())
    return status.takeError();
  return sock.GetLocalPortNumber();
}

```
- **EN**: Implements logic around `LLDB_LOG`, `PlatformSP`, `FindFreeTCPPort`, `sock`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `LLDB_LOG`, `PlatformSP`, `FindFreeTCPPort`, `sock`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 127-138
```cpp
std::vector<ArchSpec>
PlatformWasm::GetSupportedArchitectures(const ArchSpec &process_host_arch) {
  return {ArchSpec("wasm32-unknown-unknown-wasm"),
          ArchSpec("wasm64-unknown-unknown-wasm")};
}

lldb::ProcessSP PlatformWasm::Attach(ProcessAttachInfo &attach_info,
                                     Debugger &debugger, Target *target,
                                     Status &status) {
  if (m_remote_platform_sp)
    return m_remote_platform_sp->Attach(attach_info, debugger, target, status);

```
- **EN**: Implements logic around `GetSupportedArchitectures`, `ArchSpec`, `Attach`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetSupportedArchitectures`, `ArchSpec`, `Attach` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 139-150
```cpp
  status = Status::FromErrorString(
      "attaching is only supported when connected to a remote Wasm platform");
  return nullptr;
}

lldb::ProcessSP PlatformWasm::DebugProcess(ProcessLaunchInfo &launch_info,
                                           Debugger &debugger, Target &target,
                                           Status &error) {
  if (m_remote_platform_sp)
    return m_remote_platform_sp->DebugProcess(launch_info, debugger, target,
                                              error);

```
- **EN**: Implements logic around `FromErrorString`, `DebugProcess`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `FromErrorString`, `DebugProcess` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 151-157
```cpp
  Log *log = GetLog(LLDBLog::Platform);

  const PluginProperties &properties = GetGlobalProperties();

  FileSpec runtime = properties.GetRuntimePath();
  FileSystem::Instance().ResolveExecutableLocation(runtime);

```
- **EN**: Implements logic around `GetLog`, `GetGlobalProperties`, `GetRuntimePath`, `Instance`; this block abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `GetLog`, `GetGlobalProperties`, `GetRuntimePath`, `Instance` 实现具体逻辑；该代码块抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 158-170
```cpp
  if (!FileSystem::Instance().Exists(runtime)) {
    error = Status::FromErrorStringWithFormatv(
        "WebAssembly runtime does not exist: {0}", runtime.GetPath());
    return nullptr;
  }

  llvm::Expected<uint16_t> expected_port = FindFreeTCPPort();
  if (!expected_port) {
    error = Status::FromError(expected_port.takeError());
    return nullptr;
  }
  uint16_t port = *expected_port;

```
- **EN**: Implements logic around `Instance`, `FromErrorStringWithFormatv`, `GetPath`, `FindFreeTCPPort`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `Instance`, `FromErrorStringWithFormatv`, `GetPath`, `FindFreeTCPPort`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 171-182
```cpp
  Args args({runtime.GetPath(),
             llvm::formatv("{0}{1}", properties.GetPortArg(), port).str()});
  args.AppendArguments(properties.GetRuntimeArgs());
  args.AppendArguments(launch_info.GetArguments());

  launch_info.SetArguments(args, true);
  launch_info.SetLaunchInSeparateProcessGroup(true);
  // We're launching the Wasm runtime (a native host binary), not the target
  // being debugged. Clear flags that don't apply to the runtime process.
  launch_info.GetFlags().Clear(eLaunchFlagDebug | eLaunchFlagDisableASLR);
  launch_info.GetEnvironment() = Host::GetEnvironment();

```
- **EN**: Implements logic around `args`, `formatv`, `AppendArguments`, `SetArguments`, and 3 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `args`, `formatv`, `AppendArguments`, `SetArguments`, and 3 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 183-192
```cpp
  auto exit_code = std::make_shared<std::optional<int>>();
  launch_info.SetMonitorProcessCallback(
      [=](lldb::pid_t pid, int signal, int status) {
        LLDB_LOG(
            log,
            "WebAssembly runtime exited: pid = {0}, signal = {1}, status = {2}",
            pid, signal, status);
        exit_code->emplace(status);
      });

```
- **EN**: Implements logic around `optional`, `SetMonitorProcessCallback`, `LLDB_LOG`, `emplace`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `optional`, `SetMonitorProcessCallback`, `LLDB_LOG`, `emplace` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 193-202
```cpp
  // This is automatically done for host platform in
  // Target::FinalizeFileActions, but we're not a host platform.
  llvm::Error Err = launch_info.SetUpPtyRedirection();
  LLDB_LOG_ERROR(log, std::move(Err), "SetUpPtyRedirection failed: {0}");

  LLDB_LOG(log, "{0}", GetArgRange(launch_info.GetArguments()));
  error = Host::LaunchProcess(launch_info);
  if (error.Fail())
    return nullptr;

```
- **EN**: Implements logic around `SetUpPtyRedirection`, `LLDB_LOG_ERROR`, `LLDB_LOG`, `LaunchProcess`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `SetUpPtyRedirection`, `LLDB_LOG_ERROR`, `LLDB_LOG`, `LaunchProcess`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 203-210
```cpp
  ProcessSP process_sp = target.CreateProcess(
      launch_info.GetListener(), wasm::ProcessWasm::GetPluginNameStatic(),
      nullptr, true);
  if (!process_sp) {
    error = Status::FromErrorString("failed to create WebAssembly process");
    return nullptr;
  }

```
- **EN**: Implements logic around `CreateProcess`, `GetListener`, `FromErrorString`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `CreateProcess`, `GetListener`, `FromErrorString` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 211-223
```cpp
  process_sp->HijackProcessEvents(launch_info.GetHijackListener());

  error = process_sp->ConnectRemote(
      llvm::formatv("connect://localhost:{0}", port).str());
  if (error.Fail()) {
    // If we know the runtime has exited, that's a better error message than
    // failing to connect.
    if (*exit_code)
      error = Status::FromError(llvm::joinErrors(
          llvm::createStringErrorV(
              "WebAssembly runtime exited with exit code {0}", **exit_code),
          error.takeError()));

```
- **EN**: Implements logic around `HijackProcessEvents`, `ConnectRemote`, `formatv`, `Fail`, and 3 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `HijackProcessEvents`, `ConnectRemote`, `formatv`, `Fail`, and 3 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 224-234
```cpp
    return nullptr;
  }
#ifndef _WIN32
  if (launch_info.GetPTY().GetPrimaryFileDescriptor() !=
      PseudoTerminal::invalid_fd)
    process_sp->SetSTDIOFileDescriptor(
        launch_info.GetPTY().ReleasePrimaryFileDescriptor());
#endif
  return process_sp;
}

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

### Lines 235-242
```cpp
Status PlatformWasm::ConnectRemote(Args &args) {
  if (IsHost())
    return Status::FromErrorString(
        "can't connect to the host platform, always connected");

  if (!m_remote_platform_sp)
    m_remote_platform_sp = PlatformSP(new PlatformWasmRemoteGDBServer());

```
- **EN**: Implements logic around `ConnectRemote`, `IsHost`, `FromErrorString`, `PlatformSP`; this block propagates recoverable errors, status objects, or diagnostics; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `ConnectRemote`, `IsHost`, `FromErrorString`, `PlatformSP` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 243-244
```cpp
  return m_remote_platform_sp->ConnectRemote(args);
}
```
- **EN**: Implements logic around `ConnectRemote`.
- **CN**: 围绕 `ConnectRemote` 实现具体逻辑。

## Key Concepts / 关键概念

- **LLDB architecture / LLDB 架构**:
  - **EN**: Explains how LLDB organizes debugger APIs, runtime objects, and platform-specific helpers.
  - **CN**: 说明 LLDB 如何组织调试器 API、运行时对象以及平台相关辅助组件。
- **Runtime control / 运行时控制**:
  - **EN**: Models debugger ownership over targets, processes, threads, and event delivery.
  - **CN**: 建模调试器对目标、进程、线程以及事件投递的控制。
- **OS integration / 操作系统集成**:
  - **EN**: Bridges LLDB abstractions to host files, sockets, terminals, and native process facilities.
  - **CN**: 将 LLDB 抽象连接到主机文件、套接字、终端与本地进程设施。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `Plugins/Platform/WebAssembly/PlatformWasm.h`, `Plugins/Platform/WebAssembly/PlatformWasmRemoteGDBServer.h`, `Plugins/Platform/WebAssembly/PlatformWebInspectorWasm.h`, `Plugins/Process/wasm/ProcessWasm.h`, `lldb/Core/PluginManager.h`, `lldb/Host/FileSystem.h`, `lldb/Host/ProcessLaunchInfo.h`, `lldb/Host/common/TCPSocket.h`, `lldb/Interpreter/OptionValueProperties.h`, `lldb/Target/Process.h` ... (+8 more)
- **Subsystem categories / 子系统类别**: host OS abstraction helpers / 主机操作系统抽象辅助组件 (3), shared LLDB utility classes / 共享 LLDB 工具类 (3), target, process, and thread control / 目标、进程与线程控制 (2), LLDB core debugger abstractions / LLDB 核心调试抽象 (1), command interpreter support / 命令解释器支持 (1), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1), LLVM support-library helpers / LLVM Support 库辅助组件 (1)
