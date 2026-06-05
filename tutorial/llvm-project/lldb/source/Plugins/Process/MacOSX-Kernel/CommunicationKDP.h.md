# CommunicationKDP.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Process/MacOSX-Kernel/CommunicationKDP.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Declares the LLDB interfaces, data structures, and helper APIs associated with `CommunicationKDP`.
  - **CN**: 声明与 `CommunicationKDP` 相关的 LLDB 接口、数据结构以及辅助 API。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- CommunicationKDP.h --------------------------------------*- C++ -*-===//
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

#ifndef LLDB_SOURCE_PLUGINS_PROCESS_MACOSX_KERNEL_COMMUNICATIONKDP_H
#define LLDB_SOURCE_PLUGINS_PROCESS_MACOSX_KERNEL_COMMUNICATIONKDP_H

#include <list>
#include <mutex>
#include <string>

```
- **EN**: Pulls in the headers needed by this translation unit, including `list`, `mutex`, `string`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `list`, `mutex`, `string`。

### Lines 16-22
```cpp
#include "lldb/Core/Communication.h"
#include "lldb/Utility/Listener.h"
#include "lldb/Utility/Predicate.h"
#include "lldb/Utility/StreamBuffer.h"
#include "lldb/lldb-private.h"

class CommunicationKDP : public lldb_private::Communication {
```
- **EN**: Pulls in the headers needed by this translation unit, including `lldb/Core/Communication.h`, `lldb/Utility/Listener.h`, `lldb/Utility/Predicate.h`, `lldb/Utility/StreamBuffer.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `lldb/Core/Communication.h`, `lldb/Utility/Listener.h`, `lldb/Utility/Predicate.h`, `lldb/Utility/StreamBuffer.h`。

### Lines 23-36
```cpp
public:
  const static uint32_t kMaxPacketSize = 1200;
  const static uint32_t kMaxDataSize = 1024;
  typedef lldb_private::StreamBuffer<4096> PacketStreamType;
  enum CommandType {
    KDP_CONNECT = 0u,
    KDP_DISCONNECT,
    KDP_HOSTINFO,
    KDP_VERSION,
    KDP_MAXBYTES,
    KDP_READMEM,
    KDP_WRITEMEM,
    KDP_READREGS,
    KDP_WRITEREGS,
```
- **EN**: Introduces declarations for `CommandType`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `CommandType` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 37-50
```cpp
    KDP_LOAD,
    KDP_IMAGEPATH,
    KDP_SUSPEND,
    KDP_RESUMECPUS,
    KDP_EXCEPTION,
    KDP_TERMINATION,
    KDP_BREAKPOINT_SET,
    KDP_BREAKPOINT_REMOVE,
    KDP_REGIONS,
    KDP_REATTACH,
    KDP_HOSTREBOOT,
    KDP_READMEM64,
    KDP_WRITEMEM64,
    KDP_BREAKPOINT_SET64,
```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 51-61
```cpp
    KDP_BREAKPOINT_REMOVE64,
    KDP_KERNELVERSION,
    KDP_READPHYSMEM64,
    KDP_WRITEPHYSMEM64,
    KDP_READIOPORT,
    KDP_WRITEIOPORT,
    KDP_READMSR64,
    KDP_WRITEMSR64,
    KDP_DUMPINFO
  };

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 62-70
```cpp
  enum { KDP_FEATURE_BP = (1u << 0) };

  enum KDPError {
    KDP_PROTERR_SUCCESS = 0,
    KDP_PROTERR_ALREADY_CONNECTED,
    KDP_PROTERR_BAD_NBYTES,
    KDP_PROTERR_BADFLAVOR
  };

```
- **EN**: Introduces declarations for `KDPError`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `KDPError` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 71-79
```cpp
  enum PacketType {
    ePacketTypeRequest = 0x00u,
    ePacketTypeReply = 0x80u,
    ePacketTypeMask = 0x80u,
    eCommandTypeMask = 0x7fu
  };
  // Constructors and Destructors
  CommunicationKDP(const char *comm_name);

```
- **EN**: Introduces declarations for `PacketType`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `PacketType` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 80-88
```cpp
  ~CommunicationKDP() override;

  bool SendRequestPacket(const PacketStreamType &request_packet);

  // Wait for a packet within 'nsec' seconds
  size_t
  WaitForPacketWithTimeoutMicroSeconds(lldb_private::DataExtractor &response,
                                       uint32_t usec);

```
- **EN**: Declares APIs around `~CommunicationKDP`, `SendRequestPacket`, `WaitForPacketWithTimeoutMicroSeconds`.
- **CN**: 声明与 `~CommunicationKDP`, `SendRequestPacket`, `WaitForPacketWithTimeoutMicroSeconds` 相关的 API。

### Lines 89-102
```cpp
  bool GetSequenceMutex(std::unique_lock<std::recursive_mutex> &lock);

  bool CheckForPacket(const uint8_t *src, size_t src_len,
                      lldb_private::DataExtractor &packet);
  bool IsRunning() const { return m_is_running.GetValue(); }

  // Set the global packet timeout.
  //
  // For clients, this is the timeout that gets used when sending
  // packets and waiting for responses. For servers, this might not
  // get used, and if it doesn't this should be moved to the
  // CommunicationKDPClient.
  std::chrono::seconds SetPacketTimeout(std::chrono::seconds packet_timeout) {
    const auto old_packet_timeout = m_packet_timeout;
```
- **EN**: Implements logic around `GetSequenceMutex`, `CheckForPacket`, `IsRunning`, `SetPacketTimeout`.
- **CN**: 围绕 `GetSequenceMutex`, `CheckForPacket`, `IsRunning`, `SetPacketTimeout` 实现具体逻辑。

### Lines 103-112
```cpp
    m_packet_timeout = packet_timeout;
    return old_packet_timeout;
  }

  std::chrono::seconds GetPacketTimeout() const { return m_packet_timeout; }

  // Public Request Packets
  bool SendRequestConnect(uint16_t reply_port, uint16_t exc_port,
                          const char *greeting);

```
- **EN**: Implements logic around `GetPacketTimeout`, `SendRequestConnect`.
- **CN**: 围绕 `GetPacketTimeout`, `SendRequestConnect` 实现具体逻辑。

### Lines 113-120
```cpp
  bool SendRequestReattach(uint16_t reply_port);

  bool SendRequestDisconnect();

  uint32_t SendRequestReadMemory(lldb::addr_t addr, void *dst,
                                 uint32_t dst_size,
                                 lldb_private::Status &error);

```
- **EN**: Declares APIs around `SendRequestReattach`, `SendRequestDisconnect`, `SendRequestReadMemory`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 声明与 `SendRequestReattach`, `SendRequestDisconnect`, `SendRequestReadMemory` 相关的 API；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 121-128
```cpp
  uint32_t SendRequestWriteMemory(lldb::addr_t addr, const void *src,
                                  uint32_t src_len,
                                  lldb_private::Status &error);

  bool SendRawRequest(uint8_t command_byte, const void *src, uint32_t src_len,
                      lldb_private::DataExtractor &reply,
                      lldb_private::Status &error);

```
- **EN**: Declares APIs around `SendRequestWriteMemory`, `SendRawRequest`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 声明与 `SendRequestWriteMemory`, `SendRawRequest` 相关的 API；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 129-136
```cpp
  uint32_t SendRequestReadRegisters(uint32_t cpu, uint32_t flavor, void *dst,
                                    uint32_t dst_size,
                                    lldb_private::Status &error);

  uint32_t SendRequestWriteRegisters(uint32_t cpu, uint32_t flavor,
                                     const void *src, uint32_t src_size,
                                     lldb_private::Status &error);

```
- **EN**: Declares APIs around `SendRequestReadRegisters`, `SendRequestWriteRegisters`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 声明与 `SendRequestReadRegisters`, `SendRequestWriteRegisters` 相关的 API；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 137-144
```cpp
  const char *GetKernelVersion();

  // Disable KDP_IMAGEPATH for now, it seems to hang the KDP connection...
  // const char *
  // GetImagePath ();

  uint32_t GetVersion();

```
- **EN**: Declares APIs around `GetKernelVersion`, `GetVersion`.
- **CN**: 声明与 `GetKernelVersion`, `GetVersion` 相关的 API。

### Lines 145-152
```cpp
  uint32_t GetFeatureFlags();

  bool LocalBreakpointsAreSupported() {
    return (GetFeatureFlags() & KDP_FEATURE_BP) != 0;
  }

  uint32_t GetCPUMask();

```
- **EN**: Implements logic around `GetFeatureFlags`, `LocalBreakpointsAreSupported`, `GetCPUMask`.
- **CN**: 围绕 `GetFeatureFlags`, `LocalBreakpointsAreSupported`, `GetCPUMask` 实现具体逻辑。

### Lines 153-160
```cpp
  uint32_t GetCPUType();

  uint32_t GetCPUSubtype();

  lldb_private::UUID GetUUID();

  bool RemoteIsEFI();

```
- **EN**: Declares APIs around `GetCPUType`, `GetCPUSubtype`, `GetUUID`, `RemoteIsEFI`.
- **CN**: 声明与 `GetCPUType`, `GetCPUSubtype`, `GetUUID`, `RemoteIsEFI` 相关的 API。

### Lines 161-168
```cpp
  bool RemoteIsDarwinKernel();

  lldb::addr_t GetLoadAddress();

  bool SendRequestResume();

  bool SendRequestSuspend();

```
- **EN**: Declares APIs around `RemoteIsDarwinKernel`, `GetLoadAddress`, `SendRequestResume`, `SendRequestSuspend`.
- **CN**: 声明与 `RemoteIsDarwinKernel`, `GetLoadAddress`, `SendRequestResume`, `SendRequestSuspend` 相关的 API。

### Lines 169-176
```cpp
  bool SendRequestBreakpoint(bool set, lldb::addr_t addr);

protected:
  bool SendRequestPacketNoLock(const PacketStreamType &request_packet);

  size_t WaitForPacketWithTimeoutMicroSecondsNoLock(
      lldb_private::DataExtractor &response, uint32_t timeout_usec);

```
- **EN**: Declares APIs around `SendRequestBreakpoint`, `SendRequestPacketNoLock`, `WaitForPacketWithTimeoutMicroSecondsNoLock`.
- **CN**: 声明与 `SendRequestBreakpoint`, `SendRequestPacketNoLock`, `WaitForPacketWithTimeoutMicroSecondsNoLock` 相关的 API。

### Lines 177-186
```cpp
  bool WaitForNotRunningPrivate(const std::chrono::microseconds &timeout);

  void MakeRequestPacketHeader(CommandType request_type,
                               PacketStreamType &request_packet,
                               uint16_t request_length);

  // Protected Request Packets (use public accessors which will cache
  // results.
  bool SendRequestVersion();

```
- **EN**: Declares APIs around `WaitForNotRunningPrivate`, `MakeRequestPacketHeader`, `SendRequestVersion`.
- **CN**: 声明与 `WaitForNotRunningPrivate`, `MakeRequestPacketHeader`, `SendRequestVersion` 相关的 API。

### Lines 187-194
```cpp
  bool SendRequestHostInfo();

  bool SendRequestKernelVersion();

  // Disable KDP_IMAGEPATH for now, it seems to hang the KDP connection...
  // bool
  // SendRequestImagePath ();

```
- **EN**: Declares APIs around `SendRequestHostInfo`, `SendRequestKernelVersion`.
- **CN**: 声明与 `SendRequestHostInfo`, `SendRequestKernelVersion` 相关的 API。

### Lines 195-201
```cpp
  void DumpPacket(lldb_private::Stream &s, const void *data, uint32_t data_len);

  void DumpPacket(lldb_private::Stream &s,
                  const lldb_private::DataExtractor &extractor);

  bool VersionIsValid() const { return m_kdp_version_version != 0; }

```
- **EN**: Implements logic around `DumpPacket`, `VersionIsValid`.
- **CN**: 围绕 `DumpPacket`, `VersionIsValid` 实现具体逻辑。

### Lines 202-208
```cpp
  bool HostInfoIsValid() const { return m_kdp_hostinfo_cpu_type != 0; }

  bool ExtractIsReply(uint8_t first_packet_byte) const {
    // TODO: handle big endian...
    return (first_packet_byte & ePacketTypeMask) != 0;
  }

```
- **EN**: Implements logic around `HostInfoIsValid`, `ExtractIsReply`.
- **CN**: 围绕 `HostInfoIsValid`, `ExtractIsReply` 实现具体逻辑。

### Lines 209-215
```cpp
  CommandType ExtractCommand(uint8_t first_packet_byte) const {
    // TODO: handle big endian...
    return (CommandType)(first_packet_byte & eCommandTypeMask);
  }

  static const char *GetCommandAsCString(uint8_t command);

```
- **EN**: Implements logic around `ExtractCommand`, `GetCommandAsCString`; this block registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `ExtractCommand`, `GetCommandAsCString` 实现具体逻辑；该代码块注册命令、插件或面向解释器的扩展点。

### Lines 216-229
```cpp
  void ClearKDPSettings();

  bool SendRequestAndGetReply(const CommandType command,
                              const PacketStreamType &request_packet,
                              lldb_private::DataExtractor &reply_packet);
  // Classes that inherit from CommunicationKDP can see and modify these
  uint32_t m_addr_byte_size;
  lldb::ByteOrder m_byte_order;
  std::string m_bytes;
  std::recursive_mutex m_bytes_mutex;
  std::chrono::seconds m_packet_timeout;
  std::recursive_mutex m_sequence_mutex; // Restrict access to sending/receiving
                                         // packets to a single thread at a time
  lldb_private::Predicate<bool> m_is_running;
```
- **EN**: Declares APIs around `ClearKDPSettings`, `SendRequestAndGetReply`; this block coordinates debugger runtime objects, events, or asynchronous control flow; registers commands, plugins, or interpreter-facing extension points.
- **CN**: 声明与 `ClearKDPSettings`, `SendRequestAndGetReply` 相关的 API；该代码块协调调试器运行时对象、事件或异步控制流，并注册命令、插件或面向解释器的扩展点。

### Lines 230-242
```cpp
  uint32_t m_session_key;
  uint8_t m_request_sequence_id;
  uint8_t m_exception_sequence_id;
  uint32_t m_kdp_version_version;
  uint32_t m_kdp_version_feature;
  uint32_t m_kdp_hostinfo_cpu_mask;
  uint32_t m_kdp_hostinfo_cpu_type;
  uint32_t m_kdp_hostinfo_cpu_subtype;
  std::string m_kernel_version;
  // std::string m_image_path; // Disable KDP_IMAGEPATH for now, it seems to
  // hang the KDP connection...
  lldb::addr_t m_last_read_memory_addr; // Last memory read address for logging
private:
```
- **EN**: Transitions between access-control regions inside a class or struct definition.
- **CN**: 在类或结构体定义内部切换访问控制区域。

### Lines 243-248
```cpp
  // For CommunicationKDP only
  CommunicationKDP(const CommunicationKDP &) = delete;
  const CommunicationKDP &operator=(const CommunicationKDP &) = delete;
};

#endif // LLDB_SOURCE_PLUGINS_PROCESS_MACOSX_KERNEL_COMMUNICATIONKDP_H
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

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

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `lldb/Core/Communication.h`, `lldb/Utility/Listener.h`, `lldb/Utility/Predicate.h`, `lldb/Utility/StreamBuffer.h`, `lldb/lldb-private.h`
- **Standard-library headers / 标准库头文件**: `<list>`, `<mutex>`, `<string>`
- **Subsystem categories / 子系统类别**: shared LLDB utility classes / 共享 LLDB 工具类 (3), LLDB core debugger abstractions / LLDB 核心调试抽象 (1), shared LLDB public/private type declarations / 共享 LLDB 公共/私有类型声明 (1)
