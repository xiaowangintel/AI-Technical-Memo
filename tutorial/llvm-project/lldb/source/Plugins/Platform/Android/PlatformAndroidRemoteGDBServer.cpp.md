# PlatformAndroidRemoteGDBServer.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Platform/Android/PlatformAndroidRemoteGDBServer.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `PlatformAndroidRemoteGDBServer`.
  - **CN**: 实现与 `PlatformAndroidRemoteGDBServer` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- PlatformAndroidRemoteGDBServer.cpp --------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/LLDB file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/LLDB 标准文件头、许可证声明以及文件的高层说明。

### Lines 8-15
```cpp

#include "lldb/Host/ConnectionFileDescriptor.h"
#include "lldb/Host/common/TCPSocket.h"
#include "lldb/Utility/LLDBLog.h"
#include "lldb/Utility/Log.h"
#include "lldb/Utility/Status.h"
#include "lldb/Utility/UriParser.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `lldb/Host/ConnectionFileDescriptor.h`, `lldb/Host/common/TCPSocket.h`, `lldb/Utility/LLDBLog.h`, `lldb/Utility/Log.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `lldb/Host/ConnectionFileDescriptor.h`, `lldb/Host/common/TCPSocket.h`, `lldb/Utility/LLDBLog.h`, `lldb/Utility/Log.h`。

### Lines 16-25
```cpp
#include "PlatformAndroidRemoteGDBServer.h"

#include <optional>
#include <sstream>

using namespace lldb;
using namespace lldb_private;
using namespace platform_android;
using namespace llvm;

```
- **EN**: Pulls in the headers needed by this translation unit, including `PlatformAndroidRemoteGDBServer.h`, `optional`, `sstream`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `PlatformAndroidRemoteGDBServer.h`, `optional`, `sstream`。

### Lines 26-35
```cpp
static const lldb::pid_t g_remote_platform_pid =
    0; // Alias for the process id of lldb-platform

static Status ForwardPortWithAdb(
    const uint16_t local_port, const uint16_t remote_port,
    llvm::StringRef remote_socket_name,
    const std::optional<AdbClient::UnixSocketNamespace> &socket_namespace,
    std::string &device_id) {
  Log *log = GetLog(LLDBLog::Platform);

```
- **EN**: Implements logic around `ForwardPortWithAdb`, `GetLog`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; abstracts host OS resources such as files, terminals, sockets, or platforms; registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `ForwardPortWithAdb`, `GetLog` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并抽象文件、终端、套接字或平台等主机操作系统资源，并注册命令、插件或面向解释器的扩展点。

### Lines 36-43
```cpp
  auto resolved_device_id_or_error = AdbClient::ResolveDeviceID(device_id);
  if (!resolved_device_id_or_error)
    return Status::FromError(resolved_device_id_or_error.takeError());
  device_id = *resolved_device_id_or_error;

  AdbClient adb(device_id);
  LLDB_LOGF(log, "Connected to Android device \"%s\"", device_id.c_str());

```
- **EN**: Implements logic around `ResolveDeviceID`, `FromError`, `adb`, `LLDB_LOGF`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `ResolveDeviceID`, `FromError`, `adb`, `LLDB_LOGF` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 44-52
```cpp
  if (remote_port != 0) {
    LLDB_LOGF(log, "Forwarding remote TCP port %d to local TCP port %d",
              remote_port, local_port);
    return adb.SetPortForwarding(local_port, remote_port);
  }

  LLDB_LOGF(log, "Forwarding remote socket \"%s\" to local TCP port %d",
            remote_socket_name.str().c_str(), local_port);

```
- **EN**: Implements logic around `LLDB_LOGF`, `SetPortForwarding`, `str`; this block abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `LLDB_LOGF`, `SetPortForwarding`, `str` 实现具体逻辑；该代码块抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 53-59
```cpp
  if (!socket_namespace)
    return Status::FromErrorString("Invalid socket namespace");

  return adb.SetPortForwarding(local_port, remote_socket_name,
                               *socket_namespace);
}

```
- **EN**: Implements logic around `FromErrorString`, `SetPortForwarding`; this block propagates recoverable errors, status objects, or diagnostics; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `FromErrorString`, `SetPortForwarding` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 60-71
```cpp
static Status DeleteForwardPortWithAdb(uint16_t local_port,
                                       const std::string &device_id) {
  AdbClient adb(device_id);
  return adb.DeletePortForwarding(local_port);
}

static Status FindUnusedPort(uint16_t &port) {
  Status error;
  std::unique_ptr<TCPSocket> tcp_socket(new TCPSocket(true));
  if (error.Fail())
    return error;

```
- **EN**: Implements logic around `DeleteForwardPortWithAdb`, `adb`, `DeletePortForwarding`, `FindUnusedPort`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `DeleteForwardPortWithAdb`, `adb`, `DeletePortForwarding`, `FindUnusedPort`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 72-78
```cpp
  error = tcp_socket->Listen("127.0.0.1:0", 1);
  if (error.Success())
    port = tcp_socket->GetLocalPortNumber();

  return error;
}

```
- **EN**: Implements logic around `Listen`, `Success`, `GetLocalPortNumber`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `Listen`, `Success`, `GetLocalPortNumber` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 79-92
```cpp
PlatformAndroidRemoteGDBServer::~PlatformAndroidRemoteGDBServer() {
  for (const auto &it : m_port_forwards)
    DeleteForwardPortWithAdb(it.second, m_device_id);
}

bool PlatformAndroidRemoteGDBServer::LaunchGDBServer(lldb::pid_t &pid,
                                                     std::string &connect_url) {
  assert(IsConnected());
  uint16_t remote_port = 0;
  std::string socket_name;
  if (!m_gdb_client_up->LaunchGDBServer("127.0.0.1", pid, remote_port,
                                        socket_name))
    return false;

```
- **EN**: Implements logic around `~PlatformAndroidRemoteGDBServer`, `DeleteForwardPortWithAdb`, `LaunchGDBServer`, `assert`.
- **CN**: 围绕 `~PlatformAndroidRemoteGDBServer`, `DeleteForwardPortWithAdb`, `LaunchGDBServer`, `assert` 实现具体逻辑。

### Lines 93-99
```cpp
  Log *log = GetLog(LLDBLog::Platform);

  uint16_t local_port = 0;
  const char *gdbstub_port = std::getenv("ANDROID_PLATFORM_LOCAL_GDB_PORT");
  if (gdbstub_port)
    local_port = std::stoi(gdbstub_port);

```
- **EN**: Implements logic around `GetLog`, `getenv`, `stoi`; this block abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `GetLog`, `getenv`, `stoi` 实现具体逻辑；该代码块抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 100-107
```cpp
  auto error = MakeConnectURL(pid, local_port, remote_port, socket_name.c_str(),
                              connect_url);
  if (error.Success() && log)
    LLDB_LOGF(log, "gdbserver connect URL: %s", connect_url.c_str());

  return error.Success();
}

```
- **EN**: Implements logic around `MakeConnectURL`, `Success`, `LLDB_LOGF`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `MakeConnectURL`, `Success`, `LLDB_LOGF` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 108-116
```cpp
bool PlatformAndroidRemoteGDBServer::KillSpawnedProcess(lldb::pid_t pid) {
  assert(IsConnected());
  DeleteForwardPort(pid);
  return m_gdb_client_up->KillSpawnedProcess(pid);
}

Status PlatformAndroidRemoteGDBServer::ConnectRemote(Args &args) {
  m_device_id.clear();

```
- **EN**: Implements logic around `KillSpawnedProcess`, `assert`, `DeleteForwardPort`, `ConnectRemote`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `KillSpawnedProcess`, `assert`, `DeleteForwardPort`, `ConnectRemote`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 117-129
```cpp
  if (args.GetArgumentCount() != 1)
    return Status::FromErrorString(
        "\"platform connect\" takes a single argument: <connect-url>");

  const char *url = args.GetArgumentAtIndex(0);
  if (!url)
    return Status::FromErrorString("URL is null.");
  std::optional<URI> parsed_url = URI::Parse(url);
  if (!parsed_url)
    return Status::FromErrorStringWithFormat("Invalid URL: %s", url);
  if (parsed_url->hostname != "localhost")
    m_device_id = parsed_url->hostname.str();

```
- **EN**: Implements logic around `GetArgumentCount`, `FromErrorString`, `GetArgumentAtIndex`, `Parse`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `GetArgumentCount`, `FromErrorString`, `GetArgumentAtIndex`, `Parse`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 130-140
```cpp
  m_socket_namespace.reset();
  if (parsed_url->scheme == "unix-connect")
    m_socket_namespace = AdbClient::UnixSocketNamespaceFileSystem;
  else if (parsed_url->scheme == "unix-abstract-connect")
    m_socket_namespace = AdbClient::UnixSocketNamespaceAbstract;

  uint16_t local_port = 0;
  const char *platform_local_port = std::getenv("ANDROID_PLATFORM_LOCAL_PORT");
  if (platform_local_port)
    local_port = std::stoi(platform_local_port);

```
- **EN**: Implements logic around `reset`, `getenv`, `stoi`.
- **CN**: 围绕 `reset`, `getenv`, `stoi` 实现具体逻辑。

### Lines 141-148
```cpp
  std::string connect_url;
  auto error = MakeConnectURL(g_remote_platform_pid, local_port,
                              parsed_url->port.value_or(0), parsed_url->path,
                              connect_url);

  if (error.Fail())
    return error;

```
- **EN**: Implements logic around `MakeConnectURL`, `value_or`, `Fail`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `MakeConnectURL`, `value_or`, `Fail` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 149-157
```cpp
  args.ReplaceArgumentAtIndex(0, connect_url);

  Log *log = GetLog(LLDBLog::Platform);
  LLDB_LOGF(log, "Rewritten platform connect URL: %s", connect_url.c_str());

  error = PlatformRemoteGDBServer::ConnectRemote(args);
  if (error.Fail())
    DeleteForwardPort(g_remote_platform_pid);

```
- **EN**: Implements logic around `ReplaceArgumentAtIndex`, `GetLog`, `LLDB_LOGF`, `ConnectRemote`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `ReplaceArgumentAtIndex`, `GetLog`, `LLDB_LOGF`, `ConnectRemote`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 158-165
```cpp
  return error;
}

Status PlatformAndroidRemoteGDBServer::DisconnectRemote() {
  DeleteForwardPort(g_remote_platform_pid);
  return PlatformRemoteGDBServer::DisconnectRemote();
}

```
- **EN**: Implements logic around `DisconnectRemote`, `DeleteForwardPort`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `DisconnectRemote`, `DeleteForwardPort` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 166-172
```cpp
void PlatformAndroidRemoteGDBServer::DeleteForwardPort(lldb::pid_t pid) {
  Log *log = GetLog(LLDBLog::Platform);

  auto it = m_port_forwards.find(pid);
  if (it == m_port_forwards.end())
    return;

```
- **EN**: Implements logic around `DeleteForwardPort`, `GetLog`, `find`, `end`; this block abstracts host OS resources such as files, terminals, sockets, or platforms.
- **CN**: 围绕 `DeleteForwardPort`, `GetLog`, `find`, `end` 实现具体逻辑；该代码块抽象文件、终端、套接字或平台等主机操作系统资源。

### Lines 173-183
```cpp
  const auto port = it->second;
  const auto error = DeleteForwardPortWithAdb(port, m_device_id);
  if (error.Fail()) {
    LLDB_LOGF(log,
              "Failed to delete port forwarding (pid=%" PRIu64
              ", port=%d, device=%s): %s",
              pid, port, m_device_id.c_str(), error.AsCString());
  }
  m_port_forwards.erase(it);
}

```
- **EN**: Implements logic around `DeleteForwardPortWithAdb`, `Fail`, `LLDB_LOGF`, `forwarding`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `DeleteForwardPortWithAdb`, `Fail`, `LLDB_LOGF`, `forwarding`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 184-191
```cpp
Status PlatformAndroidRemoteGDBServer::MakeConnectURL(
    const lldb::pid_t pid, const uint16_t local_port,
    const uint16_t remote_port, llvm::StringRef remote_socket_name,
    std::string &connect_url) {
  static const int kAttempsNum = 5;

  Status error;

```
- **EN**: Implements logic around `MakeConnectURL`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `MakeConnectURL` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 192-203
```cpp
  auto forward = [&](const uint16_t local, const uint16_t remote) {
    Status error = ForwardPortWithAdb(local, remote, remote_socket_name,
                                      m_socket_namespace, m_device_id);
    if (error.Success()) {
      m_port_forwards[pid] = local;
      std::ostringstream url_str;
      url_str << "connect://127.0.0.1:" << local;
      connect_url = url_str.str();
    }
    return error;
  };

```
- **EN**: Implements logic around `ForwardPortWithAdb`, `Success`, `str`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `ForwardPortWithAdb`, `Success`, `str` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 204-215
```cpp
  if (local_port != 0)
    return forward(local_port, remote_port);

  // There is a race possibility that somebody will occupy a port while we're
  // in between FindUnusedPort and ForwardPortWithAdb - adding the loop to
  // mitigate such problem.
  for (auto i = 0; i < kAttempsNum; ++i) {
    uint16_t local_port = 0;
    error = FindUnusedPort(local_port);
    if (error.Fail())
      return error;

```
- **EN**: Implements logic around `forward`, `FindUnusedPort`, `Fail`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `forward`, `FindUnusedPort`, `Fail` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 216-222
```cpp
    if (forward(local_port, remote_port).Success())
      break;
  }

  return error;
}

```
- **EN**: Implements logic around `forward`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `forward` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 223-232
```cpp
lldb::ProcessSP PlatformAndroidRemoteGDBServer::ConnectProcess(
    llvm::StringRef connect_url, llvm::StringRef plugin_name,
    lldb_private::Debugger &debugger, lldb_private::Target *target,
    lldb_private::Status &error) {
  // We don't have the pid of the remote gdbserver when it isn't started by us
  // but we still want to store the list of port forwards we set up in our port
  // forward map. Generate a fake pid for these cases what won't collide with
  // any other valid pid on android.
  static lldb::pid_t s_remote_gdbserver_fake_pid = 0xffffffffffffffffULL;

```
- **EN**: Implements logic around `ConnectProcess`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `ConnectProcess` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 233-245
```cpp
  std::optional<URI> parsed_url = URI::Parse(connect_url);
  if (!parsed_url) {
    error = Status::FromErrorStringWithFormatv("Invalid URL: {0}", connect_url);
    return nullptr;
  }

  std::string new_connect_url;
  error = MakeConnectURL(s_remote_gdbserver_fake_pid--, 0,
                         parsed_url->port.value_or(0), parsed_url->path,
                         new_connect_url);
  if (error.Fail())
    return nullptr;

```
- **EN**: Implements logic around `Parse`, `FromErrorStringWithFormatv`, `MakeConnectURL`, `value_or`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `Parse`, `FromErrorStringWithFormatv`, `MakeConnectURL`, `value_or`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 246-248
```cpp
  return PlatformRemoteGDBServer::ConnectProcess(new_connect_url, plugin_name,
                                                 debugger, target, error);
}
```
- **EN**: Implements logic around `ConnectProcess`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `ConnectProcess` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

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

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `lldb/Host/ConnectionFileDescriptor.h`, `lldb/Host/common/TCPSocket.h`, `lldb/Utility/LLDBLog.h`, `lldb/Utility/Log.h`, `lldb/Utility/Status.h`, `lldb/Utility/UriParser.h`, `PlatformAndroidRemoteGDBServer.h`
- **Standard-library headers / 标准库头文件**: `<optional>`, `<sstream>`
- **Subsystem categories / 子系统类别**: shared LLDB utility classes / 共享 LLDB 工具类 (4), host OS abstraction helpers / 主机操作系统抽象辅助组件 (2)
