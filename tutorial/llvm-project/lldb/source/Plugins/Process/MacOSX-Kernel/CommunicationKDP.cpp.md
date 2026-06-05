# CommunicationKDP.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Process/MacOSX-Kernel/CommunicationKDP.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `CommunicationKDP`.
  - **CN**: 实现与 `CommunicationKDP` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24
```cpp
//===-- CommunicationKDP.cpp ----------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "CommunicationKDP.h"

#include <cerrno>
#include <climits>
#include <cstring>

#include "lldb/Core/DumpDataExtractor.h"
#include "lldb/Host/Host.h"
#include "lldb/Target/Process.h"
#include "lldb/Utility/DataBufferHeap.h"
#include "lldb/Utility/DataExtractor.h"
#include "lldb/Utility/FileSpec.h"
#include "lldb/Utility/Log.h"
#include "lldb/Utility/State.h"
#include "lldb/Utility/UUID.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `CommunicationKDP.h`, `cerrno`, `climits`, `cstring`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `CommunicationKDP.h`, `cerrno`, `climits`, `cstring`。

### Lines 25-45
```cpp
#include "ProcessKDPLog.h"

using namespace lldb;
using namespace lldb_private;

// CommunicationKDP constructor
CommunicationKDP::CommunicationKDP(const char *comm_name)
    : Communication(), m_addr_byte_size(4),
      m_byte_order(eByteOrderLittle), m_packet_timeout(5), m_sequence_mutex(),
      m_is_running(false), m_session_key(0u), m_request_sequence_id(0u),
      m_exception_sequence_id(0u), m_kdp_version_version(0u),
      m_kdp_version_feature(0u), m_kdp_hostinfo_cpu_mask(0u),
      m_kdp_hostinfo_cpu_type(0u), m_kdp_hostinfo_cpu_subtype(0u) {}

// Destructor
CommunicationKDP::~CommunicationKDP() {
  if (IsConnected()) {
    Disconnect();
  }
}

```
- **EN**: Pulls in the headers needed by this translation unit, including `ProcessKDPLog.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `ProcessKDPLog.h`。

### Lines 46-63
```cpp
bool CommunicationKDP::SendRequestPacket(
    const PacketStreamType &request_packet) {
  std::lock_guard<std::recursive_mutex> guard(m_sequence_mutex);
  return SendRequestPacketNoLock(request_packet);
}

void CommunicationKDP::MakeRequestPacketHeader(CommandType request_type,
                                               PacketStreamType &request_packet,
                                               uint16_t request_length) {
  request_packet.Clear();
  request_packet.PutHex8(request_type |
                         ePacketTypeRequest);      // Set the request type
  request_packet.PutHex8(m_request_sequence_id++); // Sequence number
  request_packet.PutHex16(
      request_length); // Length of the packet including this header
  request_packet.PutHex32(m_session_key); // Session key
}

```
- **EN**: Implements logic around `SendRequestPacket`, `guard`, `SendRequestPacketNoLock`, `MakeRequestPacketHeader`, and 4 more symbols.
- **CN**: 围绕 `SendRequestPacket`, `guard`, `SendRequestPacketNoLock`, `MakeRequestPacketHeader`, and 4 more symbols 实现具体逻辑。

### Lines 64-99
```cpp
bool CommunicationKDP::SendRequestAndGetReply(
    const CommandType command, const PacketStreamType &request_packet,
    DataExtractor &reply_packet) {
  if (IsRunning()) {
    Log *log = GetLog(KDPLog::Packets);
    if (log) {
      PacketStreamType log_strm;
      DumpPacket(log_strm, request_packet.GetData(), request_packet.GetSize());
      LLDB_LOGF(log, "error: kdp running, not sending packet: %.*s",
                (uint32_t)log_strm.GetSize(), log_strm.GetData());
    }
    return false;
  }

  std::lock_guard<std::recursive_mutex> guard(m_sequence_mutex);
  // NOTE: this only works for packets that are in native endian byte order
  assert(request_packet.GetSize() ==
         *((const uint16_t *)(request_packet.GetData() + 2)));
  lldb::offset_t offset = 1;
  const uint32_t num_retries = 3;
  for (uint32_t i = 0; i < num_retries; ++i) {
    if (SendRequestPacketNoLock(request_packet)) {
      const uint8_t request_sequence_id = (uint8_t)request_packet.GetData()[1];
      while (true) {
        if (WaitForPacketWithTimeoutMicroSecondsNoLock(
                reply_packet,
                std::chrono::microseconds(GetPacketTimeout()).count())) {
          offset = 0;
          const uint8_t reply_command = reply_packet.GetU8(&offset);
          const uint8_t reply_sequence_id = reply_packet.GetU8(&offset);
          if (request_sequence_id == reply_sequence_id) {
            // The sequent ID was correct, now verify we got the response we
            // were looking for
            if ((reply_command & eCommandTypeMask) == command) {
              // Success
              if (command == KDP_RESUMECPUS)
```
- **EN**: Implements logic around `SendRequestAndGetReply`, `IsRunning`, `GetLog`, `DumpPacket`, and 9 more symbols; this block propagates recoverable errors, status objects, or diagnostics; registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `SendRequestAndGetReply`, `IsRunning`, `GetLog`, `DumpPacket`, and 9 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并注册命令、插件或面向解释器的扩展点。

### Lines 100-129
```cpp
                m_is_running.SetValue(true, eBroadcastAlways);
              return true;
            } else {
              // Failed to get the correct response, bail
              reply_packet.Clear();
              return false;
            }
          } else if (reply_sequence_id > request_sequence_id) {
            // Sequence ID was greater than the sequence ID of the packet we
            // sent, something is really wrong...
            reply_packet.Clear();
            return false;
          } else {
            // The reply sequence ID was less than our current packet's
            // sequence ID so we should keep trying to get a response because
            // this was a response for a previous packet that we must have
            // retried.
          }
        } else {
          // Break and retry sending the packet as we didn't get a response due
          // to timeout
          break;
        }
      }
    }
  }
  reply_packet.Clear();
  return false;
}

```
- **EN**: Implements logic around `SetValue`, `Clear`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `SetValue`, `Clear` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 130-148
```cpp
bool CommunicationKDP::SendRequestPacketNoLock(
    const PacketStreamType &request_packet) {
  if (IsConnected()) {
    const char *packet_data = request_packet.GetData();
    const size_t packet_size = request_packet.GetSize();

    Log *log = GetLog(KDPLog::Packets);
    if (log) {
      PacketStreamType log_strm;
      DumpPacket(log_strm, packet_data, packet_size);
      LLDB_LOGF(log, "%.*s", (uint32_t)log_strm.GetSize(), log_strm.GetData());
    }
    ConnectionStatus status = eConnectionStatusSuccess;

    size_t bytes_written = Write(packet_data, packet_size, status, NULL);

    if (bytes_written == packet_size)
      return true;

```
- **EN**: Implements logic around `SendRequestPacketNoLock`, `IsConnected`, `GetData`, `GetSize`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `SendRequestPacketNoLock`, `IsConnected`, `GetData`, `GetSize`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 149-168
```cpp
    LLDB_LOGF(log,
              "error: failed to send packet entire packet %" PRIu64
              " of %" PRIu64 " bytes sent",
              (uint64_t)bytes_written, (uint64_t)packet_size);
  }
  return false;
}

bool CommunicationKDP::GetSequenceMutex(
    std::unique_lock<std::recursive_mutex> &lock) {
  return (lock = std::unique_lock<std::recursive_mutex>(m_sequence_mutex,
                                                        std::try_to_lock))
      .owns_lock();
}

bool CommunicationKDP::WaitForNotRunningPrivate(
    const std::chrono::microseconds &timeout) {
  return m_is_running.WaitForValueEqualTo(false, timeout);
}

```
- **EN**: Implements logic around `LLDB_LOGF`, `GetSequenceMutex`, `recursive_mutex>`, `owns_lock`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `LLDB_LOGF`, `GetSequenceMutex`, `recursive_mutex>`, `owns_lock`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 169-186
```cpp
size_t
CommunicationKDP::WaitForPacketWithTimeoutMicroSeconds(DataExtractor &packet,
                                                       uint32_t timeout_usec) {
  std::lock_guard<std::recursive_mutex> guard(m_sequence_mutex);
  return WaitForPacketWithTimeoutMicroSecondsNoLock(packet, timeout_usec);
}

size_t CommunicationKDP::WaitForPacketWithTimeoutMicroSecondsNoLock(
    DataExtractor &packet, uint32_t timeout_usec) {
  uint8_t buffer[8192];
  Status error;

  Log *log = GetLog(KDPLog::Packets);

  // Check for a packet from our cache first without trying any reading...
  if (CheckForPacket(NULL, 0, packet))
    return packet.GetByteSize();

```
- **EN**: Implements logic around `WaitForPacketWithTimeoutMicroSeconds`, `guard`, `WaitForPacketWithTimeoutMicroSecondsNoLock`, `GetLog`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `WaitForPacketWithTimeoutMicroSeconds`, `guard`, `WaitForPacketWithTimeoutMicroSecondsNoLock`, `GetLog`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 187-216
```cpp
  bool timed_out = false;
  while (IsConnected() && !timed_out) {
    lldb::ConnectionStatus status = eConnectionStatusNoConnection;
    size_t bytes_read = Read(buffer, sizeof(buffer),
                             timeout_usec == UINT32_MAX
                                 ? Timeout<std::micro>(std::nullopt)
                                 : std::chrono::microseconds(timeout_usec),
                             status, &error);

    LLDB_LOG_VERBOSE(log,
                     "Read (buffer, sizeof(buffer), timeout_usec = {:x+}, "
                     "status = {}, error = {}) => bytes_read = {}",
                     timeout_usec,
                     Communication::ConnectionStatusAsString(status), error,
                     bytes_read);

    if (bytes_read > 0) {
      if (CheckForPacket(buffer, bytes_read, packet))
        return packet.GetByteSize();
    } else {
      switch (status) {
      case eConnectionStatusInterrupted:
      case eConnectionStatusTimedOut:
        timed_out = true;
        break;
      case eConnectionStatusSuccess:
        // printf ("status = success but error = %s\n",
        // error.AsCString("<invalid>"));
        break;

```
- **EN**: Implements logic around `IsConnected`, `Read`, `micro>`, `microseconds`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `IsConnected`, `Read`, `micro>`, `microseconds`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 217-234
```cpp
      case eConnectionStatusEndOfFile:
      case eConnectionStatusNoConnection:
      case eConnectionStatusLostConnection:
      case eConnectionStatusError:
        Disconnect();
        break;
      }
    }
  }
  packet.Clear();
  return 0;
}

bool CommunicationKDP::CheckForPacket(const uint8_t *src, size_t src_len,
                                      DataExtractor &packet) {
  // Put the packet data into the buffer in a thread safe fashion
  std::lock_guard<std::recursive_mutex> guard(m_bytes_mutex);

```
- **EN**: Implements logic around `Disconnect`, `Clear`, `CheckForPacket`, `guard`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `Disconnect`, `Clear`, `CheckForPacket`, `guard` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 235-270
```cpp
  Log *log = GetLog(KDPLog::Packets);

  if (src && src_len > 0) {
    if (log && log->GetVerbose()) {
      PacketStreamType log_strm;
      DumpHexBytes(&log_strm, src, src_len, UINT32_MAX, LLDB_INVALID_ADDRESS);
      log_strm.PutChar('\0');
      LLDB_LOGF(log, "CommunicationKDP::%s adding %u bytes: %s", __FUNCTION__,
                (uint32_t)src_len, log_strm.GetData());
    }
    m_bytes.append((const char *)src, src_len);
  }

  // Make sure we at least have enough bytes for a packet header
  const size_t bytes_available = m_bytes.size();
  if (bytes_available >= 8) {
    packet.SetData(&m_bytes[0], bytes_available, m_byte_order);
    lldb::offset_t offset = 0;
    uint8_t reply_command = packet.GetU8(&offset);
    switch (reply_command) {
    case ePacketTypeRequest | KDP_EXCEPTION:
    case ePacketTypeRequest | KDP_TERMINATION:
      // We got an exception request, so be sure to send an ACK
      {
        PacketStreamType request_ack_packet(Stream::eBinary, m_byte_order);
        // Set the reply but and make the ACK packet
        request_ack_packet.PutHex8(reply_command | ePacketTypeReply);
        request_ack_packet.PutHex8(packet.GetU8(&offset));
        request_ack_packet.PutHex16(packet.GetU16(&offset));
        request_ack_packet.PutHex32(packet.GetU32(&offset));
        m_is_running.SetValue(false, eBroadcastAlways);
        // Ack to the exception or termination
        SendRequestPacketNoLock(request_ack_packet);
      }
      // Fall through to case below to get packet contents
      [[fallthrough]];
```
- **EN**: Implements logic around `GetLog`, `GetVerbose`, `DumpHexBytes`, `PutChar`, and 12 more symbols.
- **CN**: 围绕 `GetLog`, `GetVerbose`, `DumpHexBytes`, `PutChar`, and 12 more symbols 实现具体逻辑。

### Lines 271-306
```cpp
    case ePacketTypeReply | KDP_CONNECT:
    case ePacketTypeReply | KDP_DISCONNECT:
    case ePacketTypeReply | KDP_HOSTINFO:
    case ePacketTypeReply | KDP_VERSION:
    case ePacketTypeReply | KDP_MAXBYTES:
    case ePacketTypeReply | KDP_READMEM:
    case ePacketTypeReply | KDP_WRITEMEM:
    case ePacketTypeReply | KDP_READREGS:
    case ePacketTypeReply | KDP_WRITEREGS:
    case ePacketTypeReply | KDP_LOAD:
    case ePacketTypeReply | KDP_IMAGEPATH:
    case ePacketTypeReply | KDP_SUSPEND:
    case ePacketTypeReply | KDP_RESUMECPUS:
    case ePacketTypeReply | KDP_BREAKPOINT_SET:
    case ePacketTypeReply | KDP_BREAKPOINT_REMOVE:
    case ePacketTypeReply | KDP_REGIONS:
    case ePacketTypeReply | KDP_REATTACH:
    case ePacketTypeReply | KDP_HOSTREBOOT:
    case ePacketTypeReply | KDP_READMEM64:
    case ePacketTypeReply | KDP_WRITEMEM64:
    case ePacketTypeReply | KDP_BREAKPOINT_SET64:
    case ePacketTypeReply | KDP_BREAKPOINT_REMOVE64:
    case ePacketTypeReply | KDP_KERNELVERSION:
    case ePacketTypeReply | KDP_READPHYSMEM64:
    case ePacketTypeReply | KDP_WRITEPHYSMEM64:
    case ePacketTypeReply | KDP_READIOPORT:
    case ePacketTypeReply | KDP_WRITEIOPORT:
    case ePacketTypeReply | KDP_READMSR64:
    case ePacketTypeReply | KDP_WRITEMSR64:
    case ePacketTypeReply | KDP_DUMPINFO: {
      offset = 2;
      const uint16_t length = packet.GetU16(&offset);
      if (length <= bytes_available) {
        // We have an entire packet ready, we need to copy the data bytes into
        // a buffer that will be owned by the packet and erase the bytes from
        // our communication buffer "m_bytes"
```
- **EN**: Implements logic around `GetU16`.
- **CN**: 围绕 `GetU16` 实现具体逻辑。

### Lines 307-333
```cpp
        packet.SetData(DataBufferSP(new DataBufferHeap(&m_bytes[0], length)));
        m_bytes.erase(0, length);

        if (log) {
          PacketStreamType log_strm;
          DumpPacket(log_strm, packet);

          LLDB_LOGF(log, "%.*s", (uint32_t)log_strm.GetSize(),
                    log_strm.GetData());
        }
        return true;
      }
    } break;

    default:
      // Unrecognized reply command byte, erase this byte and try to get back
      // on track
      LLDB_LOGF(log, "CommunicationKDP::%s: tossing junk byte: 0x%2.2x",
                __FUNCTION__, (uint8_t)m_bytes[0]);
      m_bytes.erase(0, 1);
      break;
    }
  }
  packet.Clear();
  return false;
}

```
- **EN**: Implements logic around `SetData`, `erase`, `DumpPacket`, `LLDB_LOGF`, and 2 more symbols; this block registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `SetData`, `erase`, `DumpPacket`, `LLDB_LOGF`, and 2 more symbols 实现具体逻辑；该代码块注册命令、插件或面向解释器的扩展点。

### Lines 334-355
```cpp
bool CommunicationKDP::SendRequestConnect(uint16_t reply_port,
                                          uint16_t exc_port,
                                          const char *greeting) {
  PacketStreamType request_packet(Stream::eBinary, m_byte_order);
  if (greeting == NULL)
    greeting = "";

  const CommandType command = KDP_CONNECT;
  // Length is 82 uint16_t and the length of the greeting C string with the
  // terminating NULL
  const uint32_t command_length = 8 + 2 + 2 + ::strlen(greeting) + 1;
  MakeRequestPacketHeader(command, request_packet, command_length);
  // Always send connect ports as little endian
  request_packet.SetByteOrder(eByteOrderLittle);
  request_packet.PutHex16(htons(reply_port));
  request_packet.PutHex16(htons(exc_port));
  request_packet.SetByteOrder(m_byte_order);
  request_packet.PutCString(greeting);
  DataExtractor reply_packet;
  return SendRequestAndGetReply(command, request_packet, reply_packet);
}

```
- **EN**: Implements logic around `SendRequestConnect`, `request_packet`, `strlen`, `MakeRequestPacketHeader`, and 4 more symbols; this block registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `SendRequestConnect`, `request_packet`, `strlen`, `MakeRequestPacketHeader`, and 4 more symbols 实现具体逻辑；该代码块注册命令、插件或面向解释器的扩展点。

### Lines 356-385
```cpp
void CommunicationKDP::ClearKDPSettings() {
  m_request_sequence_id = 0;
  m_kdp_version_version = 0;
  m_kdp_version_feature = 0;
  m_kdp_hostinfo_cpu_mask = 0;
  m_kdp_hostinfo_cpu_type = 0;
  m_kdp_hostinfo_cpu_subtype = 0;
}

bool CommunicationKDP::SendRequestReattach(uint16_t reply_port) {
  PacketStreamType request_packet(Stream::eBinary, m_byte_order);
  const CommandType command = KDP_REATTACH;
  // Length is 8 bytes for the header plus 2 bytes for the reply UDP port
  const uint32_t command_length = 8 + 2;
  MakeRequestPacketHeader(command, request_packet, command_length);
  // Always send connect ports as little endian
  request_packet.SetByteOrder(eByteOrderLittle);
  request_packet.PutHex16(htons(reply_port));
  request_packet.SetByteOrder(m_byte_order);
  DataExtractor reply_packet;
  if (SendRequestAndGetReply(command, request_packet, reply_packet)) {
    // Reset the sequence ID to zero for reattach
    ClearKDPSettings();
    lldb::offset_t offset = 4;
    m_session_key = reply_packet.GetU32(&offset);
    return true;
  }
  return false;
}

```
- **EN**: Implements logic around `ClearKDPSettings`, `SendRequestReattach`, `request_packet`, `MakeRequestPacketHeader`, and 4 more symbols; this block registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `ClearKDPSettings`, `SendRequestReattach`, `request_packet`, `MakeRequestPacketHeader`, and 4 more symbols 实现具体逻辑；该代码块注册命令、插件或面向解释器的扩展点。

### Lines 386-412
```cpp
uint32_t CommunicationKDP::GetVersion() {
  if (!VersionIsValid())
    SendRequestVersion();
  return m_kdp_version_version;
}

uint32_t CommunicationKDP::GetFeatureFlags() {
  if (!VersionIsValid())
    SendRequestVersion();
  return m_kdp_version_feature;
}

bool CommunicationKDP::SendRequestVersion() {
  PacketStreamType request_packet(Stream::eBinary, m_byte_order);
  const CommandType command = KDP_VERSION;
  const uint32_t command_length = 8;
  MakeRequestPacketHeader(command, request_packet, command_length);
  DataExtractor reply_packet;
  if (SendRequestAndGetReply(command, request_packet, reply_packet)) {
    lldb::offset_t offset = 8;
    m_kdp_version_version = reply_packet.GetU32(&offset);
    m_kdp_version_feature = reply_packet.GetU32(&offset);
    return true;
  }
  return false;
}

```
- **EN**: Implements logic around `GetVersion`, `VersionIsValid`, `SendRequestVersion`, `GetFeatureFlags`, and 4 more symbols; this block registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `GetVersion`, `VersionIsValid`, `SendRequestVersion`, `GetFeatureFlags`, and 4 more symbols 实现具体逻辑；该代码块注册命令、插件或面向解释器的扩展点。

### Lines 413-430
```cpp
uint32_t CommunicationKDP::GetCPUMask() {
  if (!HostInfoIsValid())
    SendRequestHostInfo();
  return m_kdp_hostinfo_cpu_mask;
}

uint32_t CommunicationKDP::GetCPUType() {
  if (!HostInfoIsValid())
    SendRequestHostInfo();
  return m_kdp_hostinfo_cpu_type;
}

uint32_t CommunicationKDP::GetCPUSubtype() {
  if (!HostInfoIsValid())
    SendRequestHostInfo();
  return m_kdp_hostinfo_cpu_subtype;
}

```
- **EN**: Implements logic around `GetCPUMask`, `HostInfoIsValid`, `SendRequestHostInfo`, `GetCPUType`, and 1 more symbols.
- **CN**: 围绕 `GetCPUMask`, `HostInfoIsValid`, `SendRequestHostInfo`, `GetCPUType`, and 1 more symbols 实现具体逻辑。

### Lines 431-448
```cpp
lldb_private::UUID CommunicationKDP::GetUUID() {
  UUID uuid;
  if (GetKernelVersion() == NULL)
    return uuid;

  if (m_kernel_version.find("UUID=") == std::string::npos)
    return uuid;

  size_t p = m_kernel_version.find("UUID=") + strlen("UUID=");
  std::string uuid_str = m_kernel_version.substr(p, 36);
  if (uuid_str.size() < 32)
    return uuid;

  if (!uuid.SetFromStringRef(uuid_str)) {
    UUID invalid_uuid;
    return invalid_uuid;
  }

```
- **EN**: Implements logic around `GetUUID`, `GetKernelVersion`, `find`, `substr`, and 2 more symbols.
- **CN**: 围绕 `GetUUID`, `GetKernelVersion`, `find`, `substr`, and 2 more symbols 实现具体逻辑。

### Lines 449-467
```cpp
  return uuid;
}

bool CommunicationKDP::RemoteIsEFI() {
  if (GetKernelVersion() == NULL)
    return false;
  return strncmp(m_kernel_version.c_str(), "EFI", 3) == 0;
}

bool CommunicationKDP::RemoteIsDarwinKernel() {
  if (GetKernelVersion() == NULL)
    return false;
  return m_kernel_version.find("Darwin Kernel") != std::string::npos;
}

lldb::addr_t CommunicationKDP::GetLoadAddress() {
  if (GetKernelVersion() == NULL)
    return LLDB_INVALID_ADDRESS;

```
- **EN**: Implements logic around `RemoteIsEFI`, `GetKernelVersion`, `strncmp`, `RemoteIsDarwinKernel`, and 2 more symbols.
- **CN**: 围绕 `RemoteIsEFI`, `GetKernelVersion`, `strncmp`, `RemoteIsDarwinKernel`, and 2 more symbols 实现具体逻辑。

### Lines 468-494
```cpp
  if (m_kernel_version.find("stext=") == std::string::npos)
    return LLDB_INVALID_ADDRESS;
  size_t p = m_kernel_version.find("stext=") + strlen("stext=");
  if (m_kernel_version[p] != '0' || m_kernel_version[p + 1] != 'x')
    return LLDB_INVALID_ADDRESS;

  addr_t kernel_load_address;
  errno = 0;
  kernel_load_address = ::strtoul(m_kernel_version.c_str() + p, NULL, 16);
  if (errno != 0 || kernel_load_address == 0)
    return LLDB_INVALID_ADDRESS;

  return kernel_load_address;
}

bool CommunicationKDP::SendRequestHostInfo() {
  PacketStreamType request_packet(Stream::eBinary, m_byte_order);
  const CommandType command = KDP_HOSTINFO;
  const uint32_t command_length = 8;
  MakeRequestPacketHeader(command, request_packet, command_length);
  DataExtractor reply_packet;
  if (SendRequestAndGetReply(command, request_packet, reply_packet)) {
    lldb::offset_t offset = 8;
    m_kdp_hostinfo_cpu_mask = reply_packet.GetU32(&offset);
    m_kdp_hostinfo_cpu_type = reply_packet.GetU32(&offset);
    m_kdp_hostinfo_cpu_subtype = reply_packet.GetU32(&offset);

```
- **EN**: Implements logic around `find`, `strtoul`, `SendRequestHostInfo`, `request_packet`, and 3 more symbols; this block registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `find`, `strtoul`, `SendRequestHostInfo`, `request_packet`, and 3 more symbols 实现具体逻辑；该代码块注册命令、插件或面向解释器的扩展点。

### Lines 495-526
```cpp
    ArchSpec kernel_arch;
    kernel_arch.SetArchitecture(eArchTypeMachO, m_kdp_hostinfo_cpu_type,
                                m_kdp_hostinfo_cpu_subtype);

    m_addr_byte_size = kernel_arch.GetAddressByteSize();
    m_byte_order = kernel_arch.GetByteOrder();
    return true;
  }
  return false;
}

const char *CommunicationKDP::GetKernelVersion() {
  if (m_kernel_version.empty())
    SendRequestKernelVersion();
  return m_kernel_version.c_str();
}

bool CommunicationKDP::SendRequestKernelVersion() {
  PacketStreamType request_packet(Stream::eBinary, m_byte_order);
  const CommandType command = KDP_KERNELVERSION;
  const uint32_t command_length = 8;
  MakeRequestPacketHeader(command, request_packet, command_length);
  DataExtractor reply_packet;
  if (SendRequestAndGetReply(command, request_packet, reply_packet)) {
    const char *kernel_version_cstr = reply_packet.PeekCStr(8);
    if (kernel_version_cstr && kernel_version_cstr[0])
      m_kernel_version.assign(kernel_version_cstr);
    return true;
  }
  return false;
}

```
- **EN**: Implements logic around `SetArchitecture`, `GetAddressByteSize`, `GetByteOrder`, `GetKernelVersion`, and 8 more symbols; this block registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `SetArchitecture`, `GetAddressByteSize`, `GetByteOrder`, `GetKernelVersion`, and 8 more symbols 实现具体逻辑；该代码块注册命令、插件或面向解释器的扩展点。

### Lines 527-557
```cpp
bool CommunicationKDP::SendRequestDisconnect() {
  PacketStreamType request_packet(Stream::eBinary, m_byte_order);
  const CommandType command = KDP_DISCONNECT;
  const uint32_t command_length = 8;
  MakeRequestPacketHeader(command, request_packet, command_length);
  DataExtractor reply_packet;
  if (SendRequestAndGetReply(command, request_packet, reply_packet)) {
    // Are we supposed to get a reply for disconnect?
  }
  ClearKDPSettings();
  return true;
}

uint32_t CommunicationKDP::SendRequestReadMemory(lldb::addr_t addr, void *dst,
                                                 uint32_t dst_len,
                                                 Status &error) {
  PacketStreamType request_packet(Stream::eBinary, m_byte_order);
  bool use_64 = (GetVersion() >= 11);
  uint32_t command_addr_byte_size = use_64 ? 8 : 4;
  const CommandType command = use_64 ? KDP_READMEM64 : KDP_READMEM;
  // Size is header + address size + uint32_t length
  const uint32_t command_length = 8 + command_addr_byte_size + 4;
  MakeRequestPacketHeader(command, request_packet, command_length);
  request_packet.PutMaxHex64(addr, command_addr_byte_size);
  request_packet.PutHex32(dst_len);
  DataExtractor reply_packet;
  if (SendRequestAndGetReply(command, request_packet, reply_packet)) {
    lldb::offset_t offset = 8;
    uint32_t kdp_error = reply_packet.GetU32(&offset);
    uint32_t src_len = reply_packet.GetByteSize() - 12;

```
- **EN**: Implements logic around `SendRequestDisconnect`, `request_packet`, `MakeRequestPacketHeader`, `SendRequestAndGetReply`, and 7 more symbols; this block propagates recoverable errors, status objects, or diagnostics; maps executable state back to modules, symbols, sections, or addresses; registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `SendRequestDisconnect`, `request_packet`, `MakeRequestPacketHeader`, `SendRequestAndGetReply`, and 7 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并将可执行状态映射回模块、符号、节区或地址，并注册命令、插件或面向解释器的扩展点。

### Lines 558-576
```cpp
    if (src_len > 0) {
      const void *src = reply_packet.GetData(&offset, src_len);
      if (src) {
        ::memcpy(dst, src, src_len);
        error.Clear();
        return src_len;
      }
    }
    if (kdp_error)
      error = Status::FromErrorStringWithFormat(
          "kdp read memory failed (error %u)", kdp_error);
    else
      error = Status::FromErrorString("kdp read memory failed");
  } else {
    error = Status::FromErrorString("failed to send packet");
  }
  return 0;
}

```
- **EN**: Implements logic around `GetData`, `memcpy`, `Clear`, `FromErrorStringWithFormat`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetData`, `memcpy`, `Clear`, `FromErrorStringWithFormat`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 577-608
```cpp
uint32_t CommunicationKDP::SendRequestWriteMemory(lldb::addr_t addr,
                                                  const void *src,
                                                  uint32_t src_len,
                                                  Status &error) {
  PacketStreamType request_packet(Stream::eBinary, m_byte_order);
  bool use_64 = (GetVersion() >= 11);
  uint32_t command_addr_byte_size = use_64 ? 8 : 4;
  const CommandType command = use_64 ? KDP_WRITEMEM64 : KDP_WRITEMEM;
  // Size is header + address size + uint32_t length
  const uint32_t command_length = 8 + command_addr_byte_size + 4 + src_len;
  MakeRequestPacketHeader(command, request_packet, command_length);
  request_packet.PutMaxHex64(addr, command_addr_byte_size);
  request_packet.PutHex32(src_len);
  request_packet.PutRawBytes(src, src_len);

  DataExtractor reply_packet;
  if (SendRequestAndGetReply(command, request_packet, reply_packet)) {
    lldb::offset_t offset = 8;
    uint32_t kdp_error = reply_packet.GetU32(&offset);
    if (kdp_error)
      error = Status::FromErrorStringWithFormat(
          "kdp write memory failed (error %u)", kdp_error);
    else {
      error.Clear();
      return src_len;
    }
  } else {
    error = Status::FromErrorString("failed to send packet");
  }
  return 0;
}

```
- **EN**: Implements logic around `SendRequestWriteMemory`, `request_packet`, `GetVersion`, `MakeRequestPacketHeader`, and 9 more symbols; this block propagates recoverable errors, status objects, or diagnostics; maps executable state back to modules, symbols, sections, or addresses; registers commands, plugins, or interpreter-facing extension points; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `SendRequestWriteMemory`, `request_packet`, `GetVersion`, `MakeRequestPacketHeader`, and 9 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并将可执行状态映射回模块、符号、节区或地址，并注册命令、插件或面向解释器的扩展点，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 609-636
```cpp
bool CommunicationKDP::SendRawRequest(
    uint8_t command_byte,
    const void *src,  // Raw packet payload bytes
    uint32_t src_len, // Raw packet payload length
    DataExtractor &reply_packet, Status &error) {
  PacketStreamType request_packet(Stream::eBinary, m_byte_order);
  // Size is header + address size + uint32_t length
  const uint32_t command_length = 8 + src_len;
  const CommandType command = (CommandType)command_byte;
  MakeRequestPacketHeader(command, request_packet, command_length);
  request_packet.PutRawBytes(src, src_len);

  if (SendRequestAndGetReply(command, request_packet, reply_packet)) {
    lldb::offset_t offset = 8;
    uint32_t kdp_error = reply_packet.GetU32(&offset);
    if (kdp_error && (command_byte != KDP_DUMPINFO))
      error = Status::FromErrorStringWithFormat(
          "request packet 0x%8.8x failed (error %u)", command_byte, kdp_error);
    else {
      error.Clear();
      return true;
    }
  } else {
    error = Status::FromErrorString("failed to send packet");
  }
  return false;
}

```
- **EN**: Implements logic around `SendRawRequest`, `request_packet`, `MakeRequestPacketHeader`, `PutRawBytes`, and 6 more symbols; this block propagates recoverable errors, status objects, or diagnostics; maps executable state back to modules, symbols, sections, or addresses; registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `SendRawRequest`, `request_packet`, `MakeRequestPacketHeader`, `PutRawBytes`, and 6 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并将可执行状态映射回模块、符号、节区或地址，并注册命令、插件或面向解释器的扩展点。

### Lines 637-672
```cpp
const char *CommunicationKDP::GetCommandAsCString(uint8_t command) {
  switch (command) {
  case KDP_CONNECT:
    return "KDP_CONNECT";
  case KDP_DISCONNECT:
    return "KDP_DISCONNECT";
  case KDP_HOSTINFO:
    return "KDP_HOSTINFO";
  case KDP_VERSION:
    return "KDP_VERSION";
  case KDP_MAXBYTES:
    return "KDP_MAXBYTES";
  case KDP_READMEM:
    return "KDP_READMEM";
  case KDP_WRITEMEM:
    return "KDP_WRITEMEM";
  case KDP_READREGS:
    return "KDP_READREGS";
  case KDP_WRITEREGS:
    return "KDP_WRITEREGS";
  case KDP_LOAD:
    return "KDP_LOAD";
  case KDP_IMAGEPATH:
    return "KDP_IMAGEPATH";
  case KDP_SUSPEND:
    return "KDP_SUSPEND";
  case KDP_RESUMECPUS:
    return "KDP_RESUMECPUS";
  case KDP_EXCEPTION:
    return "KDP_EXCEPTION";
  case KDP_TERMINATION:
    return "KDP_TERMINATION";
  case KDP_BREAKPOINT_SET:
    return "KDP_BREAKPOINT_SET";
  case KDP_BREAKPOINT_REMOVE:
    return "KDP_BREAKPOINT_REMOVE";
```
- **EN**: Implements logic around `GetCommandAsCString`; this block registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `GetCommandAsCString` 实现具体逻辑；该代码块注册命令、插件或面向解释器的扩展点。

### Lines 673-706
```cpp
  case KDP_REGIONS:
    return "KDP_REGIONS";
  case KDP_REATTACH:
    return "KDP_REATTACH";
  case KDP_HOSTREBOOT:
    return "KDP_HOSTREBOOT";
  case KDP_READMEM64:
    return "KDP_READMEM64";
  case KDP_WRITEMEM64:
    return "KDP_WRITEMEM64";
  case KDP_BREAKPOINT_SET64:
    return "KDP_BREAKPOINT64_SET";
  case KDP_BREAKPOINT_REMOVE64:
    return "KDP_BREAKPOINT64_REMOVE";
  case KDP_KERNELVERSION:
    return "KDP_KERNELVERSION";
  case KDP_READPHYSMEM64:
    return "KDP_READPHYSMEM64";
  case KDP_WRITEPHYSMEM64:
    return "KDP_WRITEPHYSMEM64";
  case KDP_READIOPORT:
    return "KDP_READIOPORT";
  case KDP_WRITEIOPORT:
    return "KDP_WRITEIOPORT";
  case KDP_READMSR64:
    return "KDP_READMSR64";
  case KDP_WRITEMSR64:
    return "KDP_WRITEMSR64";
  case KDP_DUMPINFO:
    return "KDP_DUMPINFO";
  }
  return NULL;
}

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 707-730
```cpp
void CommunicationKDP::DumpPacket(Stream &s, const void *data,
                                  uint32_t data_len) {
  DataExtractor extractor(data, data_len, m_byte_order, m_addr_byte_size);
  DumpPacket(s, extractor);
}

void CommunicationKDP::DumpPacket(Stream &s, const DataExtractor &packet) {
  const char *error_desc = NULL;
  if (packet.GetByteSize() < 8) {
    error_desc = "error: invalid packet (too short): ";
  } else {
    lldb::offset_t offset = 0;
    const uint8_t first_packet_byte = packet.GetU8(&offset);
    const uint8_t sequence_id = packet.GetU8(&offset);
    const uint16_t length = packet.GetU16(&offset);
    const uint32_t key = packet.GetU32(&offset);
    const CommandType command = ExtractCommand(first_packet_byte);
    const char *command_name = GetCommandAsCString(command);
    if (command_name) {
      const bool is_reply = ExtractIsReply(first_packet_byte);
      s.Printf("(running=%i) %s %24s: 0x%2.2x 0x%2.2x 0x%4.4x 0x%8.8x ",
               IsRunning(), is_reply ? "<--" : "-->", command_name,
               first_packet_byte, sequence_id, length, key);

```
- **EN**: Implements logic around `DumpPacket`, `extractor`, `GetByteSize`, `packet`, and 8 more symbols; this block propagates recoverable errors, status objects, or diagnostics; registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `DumpPacket`, `extractor`, `GetByteSize`, `packet`, and 8 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并注册命令、插件或面向解释器的扩展点。

### Lines 731-749
```cpp
      if (is_reply) {
        // Dump request reply packets
        switch (command) {
        // Commands that return a single 32 bit error
        case KDP_CONNECT:
        case KDP_WRITEMEM:
        case KDP_WRITEMEM64:
        case KDP_BREAKPOINT_SET:
        case KDP_BREAKPOINT_REMOVE:
        case KDP_BREAKPOINT_SET64:
        case KDP_BREAKPOINT_REMOVE64:
        case KDP_WRITEREGS:
        case KDP_LOAD:
        case KDP_WRITEIOPORT:
        case KDP_WRITEMSR64: {
          const uint32_t error = packet.GetU32(&offset);
          s.Printf(" (error=0x%8.8x)", error);
        } break;

```
- **EN**: Implements logic around `GetU32`, `Printf`; this block propagates recoverable errors, status objects, or diagnostics; registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `GetU32`, `Printf` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并注册命令、插件或面向解释器的扩展点。

### Lines 750-768
```cpp
        case KDP_DISCONNECT:
        case KDP_REATTACH:
        case KDP_HOSTREBOOT:
        case KDP_SUSPEND:
        case KDP_RESUMECPUS:
        case KDP_EXCEPTION:
        case KDP_TERMINATION:
          // No return value for the reply, just the header to ack
          s.PutCString(" ()");
          break;

        case KDP_HOSTINFO: {
          const uint32_t cpu_mask = packet.GetU32(&offset);
          const uint32_t cpu_type = packet.GetU32(&offset);
          const uint32_t cpu_subtype = packet.GetU32(&offset);
          s.Printf(" (cpu_mask=0x%8.8x, cpu_type=0x%8.8x, cpu_subtype=0x%8.8x)",
                   cpu_mask, cpu_type, cpu_subtype);
        } break;

```
- **EN**: Implements logic around `PutCString`, `GetU32`, `Printf`.
- **CN**: 围绕 `PutCString`, `GetU32`, `Printf` 实现具体逻辑。

### Lines 769-788
```cpp
        case KDP_VERSION: {
          const uint32_t version = packet.GetU32(&offset);
          const uint32_t feature = packet.GetU32(&offset);
          s.Printf(" (version=0x%8.8x, feature=0x%8.8x)", version, feature);
        } break;

        case KDP_REGIONS: {
          const uint32_t region_count = packet.GetU32(&offset);
          s.Printf(" (count = %u", region_count);
          for (uint32_t i = 0; i < region_count; ++i) {
            const addr_t region_addr = packet.GetAddress(&offset);
            const uint32_t region_size = packet.GetU32(&offset);
            const uint32_t region_prot = packet.GetU32(&offset);
            s.Printf("\n\tregion[%" PRIu64 "] = { range = [0x%16.16" PRIx64
                     " - 0x%16.16" PRIx64 "), size = 0x%8.8x, prot = %s }",
                     region_addr, region_addr, region_addr + region_size,
                     region_size, GetPermissionsAsCString(region_prot));
          }
        } break;

```
- **EN**: Implements logic around `GetU32`, `Printf`, `GetAddress`, `GetPermissionsAsCString`.
- **CN**: 围绕 `GetU32`, `Printf`, `GetAddress`, `GetPermissionsAsCString` 实现具体逻辑。

### Lines 789-808
```cpp
        case KDP_READMEM:
        case KDP_READMEM64:
        case KDP_READPHYSMEM64: {
          const uint32_t error = packet.GetU32(&offset);
          const uint32_t count = packet.GetByteSize() - offset;
          s.Printf(" (error = 0x%8.8x:\n", error);
          if (count > 0)
            DumpDataExtractor(packet, 
                              &s,                      // Stream to dump to
                              offset,                  // Offset within "packet"
                              eFormatBytesWithASCII,   // Format to use
                              1,                       // Size of each item 
                                                       // in bytes
                              count,                   // Number of items
                              16,                      // Number per line
                              m_last_read_memory_addr, // Don't show addresses
                                                       // before each line
                              0, 0);                   // No bitfields
        } break;

```
- **EN**: Implements logic around `GetU32`, `GetByteSize`, `Printf`, `DumpDataExtractor`; this block propagates recoverable errors, status objects, or diagnostics; controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `GetU32`, `GetByteSize`, `Printf`, `DumpDataExtractor` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并控制调试器侧的值格式化或 synthetic 展示。

### Lines 809-828
```cpp
        case KDP_READREGS: {
          const uint32_t error = packet.GetU32(&offset);
          const uint32_t count = packet.GetByteSize() - offset;
          s.Printf(" (error = 0x%8.8x regs:\n", error);
          if (count > 0)
            DumpDataExtractor(packet, 
                              &s,                       // Stream to dump to
                              offset,                   // Offset within "packet"
                              eFormatHex,               // Format to use
                              m_addr_byte_size,         // Size of each item 
                                                        // in bytes
                              count / m_addr_byte_size, // Number of items
                              16 / m_addr_byte_size,    // Number per line
                              LLDB_INVALID_ADDRESS, 
                                                        // Don't 
                                                        // show addresses before
                                                        // each line
                              0, 0);                    // No bitfields
        } break;

```
- **EN**: Implements logic around `GetU32`, `GetByteSize`, `Printf`, `DumpDataExtractor`; this block propagates recoverable errors, status objects, or diagnostics; controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `GetU32`, `GetByteSize`, `Printf`, `DumpDataExtractor` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并控制调试器侧的值格式化或 synthetic 展示。

### Lines 829-864
```cpp
        case KDP_KERNELVERSION: {
          const char *kernel_version = packet.PeekCStr(8);
          s.Printf(" (version = \"%s\")", kernel_version);
        } break;

        case KDP_MAXBYTES: {
          const uint32_t max_bytes = packet.GetU32(&offset);
          s.Printf(" (max_bytes = 0x%8.8x (%u))", max_bytes, max_bytes);
        } break;
        case KDP_IMAGEPATH: {
          const char *path = packet.GetCStr(&offset);
          s.Printf(" (path = \"%s\")", path);
        } break;

        case KDP_READIOPORT:
        case KDP_READMSR64: {
          const uint32_t error = packet.GetU32(&offset);
          const uint32_t count = packet.GetByteSize() - offset;
          s.Printf(" (error = 0x%8.8x io:\n", error);
          if (count > 0)
            DumpDataExtractor(packet, 
                              &s,                   // Stream to dump to
                              offset,               // Offset within "packet"
                              eFormatHex,           // Format to use
                              1,                    // Size of each item in bytes
                              count,                // Number of items
                              16,                   // Number per line
                              LLDB_INVALID_ADDRESS, // Don't show addresses 
                                                    // before each line
                              0, 0);                // No bitfields
        } break;
        case KDP_DUMPINFO: {
          const uint32_t count = packet.GetByteSize() - offset;
          s.Printf(" (count = %u, bytes = \n", count);
          if (count > 0)
            DumpDataExtractor(packet, 
```
- **EN**: Implements logic around `PeekCStr`, `Printf`, `GetU32`, `GetCStr`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `PeekCStr`, `Printf`, `GetU32`, `GetCStr`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并控制调试器侧的值格式化或 synthetic 展示。

### Lines 865-891
```cpp
                              &s,                   // Stream to dump to
                              offset,               // Offset within "packet"
                              eFormatHex,           // Format to use
                              1,                    // Size of each item in 
                                                    // bytes
                              count,                // Number of items
                              16,                   // Number per line
                              LLDB_INVALID_ADDRESS, // Don't show addresses 
                                                    // before each line
                              0, 0);                // No bitfields

        } break;

        default:
          s.Printf(" (add support for dumping this packet reply!!!");
          break;
        }
      } else {
        // Dump request packets
        switch (command) {
        case KDP_CONNECT: {
          const uint16_t reply_port = ntohs(packet.GetU16(&offset));
          const uint16_t exc_port = ntohs(packet.GetU16(&offset));
          s.Printf(" (reply_port = %u, exc_port = %u, greeting = \"%s\")",
                   reply_port, exc_port, packet.GetCStr(&offset));
        } break;

```
- **EN**: Implements logic around `Printf`, `ntohs`, `GetCStr`; this block controls debugger-side formatting or synthetic presentation of values; registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `Printf`, `ntohs`, `GetCStr` 实现具体逻辑；该代码块控制调试器侧的值格式化或 synthetic 展示，并注册命令、插件或面向解释器的扩展点。

### Lines 892-909
```cpp
        case KDP_DISCONNECT:
        case KDP_HOSTREBOOT:
        case KDP_HOSTINFO:
        case KDP_VERSION:
        case KDP_REGIONS:
        case KDP_KERNELVERSION:
        case KDP_MAXBYTES:
        case KDP_IMAGEPATH:
        case KDP_SUSPEND:
          // No args, just the header in the request...
          s.PutCString(" ()");
          break;

        case KDP_RESUMECPUS: {
          const uint32_t cpu_mask = packet.GetU32(&offset);
          s.Printf(" (cpu_mask = 0x%8.8x)", cpu_mask);
        } break;

```
- **EN**: Implements logic around `PutCString`, `GetU32`, `Printf`.
- **CN**: 围绕 `PutCString`, `GetU32`, `Printf` 实现具体逻辑。

### Lines 910-931
```cpp
        case KDP_READMEM: {
          const uint32_t addr = packet.GetU32(&offset);
          const uint32_t size = packet.GetU32(&offset);
          s.Printf(" (addr = 0x%8.8x, size = %u)", addr, size);
          m_last_read_memory_addr = addr;
        } break;

        case KDP_WRITEMEM: {
          const uint32_t addr = packet.GetU32(&offset);
          const uint32_t size = packet.GetU32(&offset);
          s.Printf(" (addr = 0x%8.8x, size = %u, bytes = \n", addr, size);
          if (size > 0)
            DumpHexBytes(&s, packet.GetData(&offset, size), size, 32, addr);
        } break;

        case KDP_READMEM64: {
          const uint64_t addr = packet.GetU64(&offset);
          const uint32_t size = packet.GetU32(&offset);
          s.Printf(" (addr = 0x%16.16" PRIx64 ", size = %u)", addr, size);
          m_last_read_memory_addr = addr;
        } break;

```
- **EN**: Implements logic around `GetU32`, `Printf`, `DumpHexBytes`, `GetU64`.
- **CN**: 围绕 `GetU32`, `Printf`, `DumpHexBytes`, `GetU64` 实现具体逻辑。

### Lines 932-949
```cpp
        case KDP_READPHYSMEM64: {
          const uint64_t addr = packet.GetU64(&offset);
          const uint32_t size = packet.GetU32(&offset);
          const uint32_t lcpu = packet.GetU16(&offset);
          s.Printf(" (addr = 0x%16.16llx, size = %u, lcpu = %u)", addr, size,
                   lcpu);
          m_last_read_memory_addr = addr;
        } break;

        case KDP_WRITEMEM64: {
          const uint64_t addr = packet.GetU64(&offset);
          const uint32_t size = packet.GetU32(&offset);
          s.Printf(" (addr = 0x%16.16" PRIx64 ", size = %u, bytes = \n", addr,
                   size);
          if (size > 0)
            DumpHexBytes(&s, packet.GetData(&offset, size), size, 32, addr);
        } break;

```
- **EN**: Implements logic around `GetU64`, `GetU32`, `GetU16`, `Printf`, and 1 more symbols.
- **CN**: 围绕 `GetU64`, `GetU32`, `GetU16`, `Printf`, and 1 more symbols 实现具体逻辑。

### Lines 950-985
```cpp
        case KDP_WRITEPHYSMEM64: {
          const uint64_t addr = packet.GetU64(&offset);
          const uint32_t size = packet.GetU32(&offset);
          const uint32_t lcpu = packet.GetU16(&offset);
          s.Printf(" (addr = 0x%16.16llx, size = %u, lcpu = %u, bytes = \n",
                   addr, size, lcpu);
          if (size > 0)
            DumpHexBytes(&s, packet.GetData(&offset, size), size, 32, addr);
        } break;

        case KDP_READREGS: {
          const uint32_t cpu = packet.GetU32(&offset);
          const uint32_t flavor = packet.GetU32(&offset);
          s.Printf(" (cpu = %u, flavor = %u)", cpu, flavor);
        } break;

        case KDP_WRITEREGS: {
          const uint32_t cpu = packet.GetU32(&offset);
          const uint32_t flavor = packet.GetU32(&offset);
          const uint32_t nbytes = packet.GetByteSize() - offset;
          s.Printf(" (cpu = %u, flavor = %u, regs = \n", cpu, flavor);
          if (nbytes > 0)
            DumpDataExtractor(packet, 
                              &s,                        // Stream to dump to
                              offset,                    // Offset within 
                                                         // "packet"
                              eFormatHex,                // Format to use
                              m_addr_byte_size,          // Size of each item in 
                                                         // bytes
                              nbytes / m_addr_byte_size, // Number of items
                              16 / m_addr_byte_size,     // Number per line
                              LLDB_INVALID_ADDRESS,      // Don't show addresses
                                                         // before each line
                              0, 0);                // No bitfields
        } break;

```
- **EN**: Implements logic around `GetU64`, `GetU32`, `GetU16`, `Printf`, and 3 more symbols; this block controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `GetU64`, `GetU32`, `GetU16`, `Printf`, and 3 more symbols 实现具体逻辑；该代码块控制调试器侧的值格式化或 synthetic 展示。

### Lines 986-1005
```cpp
        case KDP_BREAKPOINT_SET:
        case KDP_BREAKPOINT_REMOVE: {
          const uint32_t addr = packet.GetU32(&offset);
          s.Printf(" (addr = 0x%8.8x)", addr);
        } break;

        case KDP_BREAKPOINT_SET64:
        case KDP_BREAKPOINT_REMOVE64: {
          const uint64_t addr = packet.GetU64(&offset);
          s.Printf(" (addr = 0x%16.16" PRIx64 ")", addr);
        } break;

        case KDP_LOAD: {
          const char *path = packet.GetCStr(&offset);
          s.Printf(" (path = \"%s\")", path);
        } break;

        case KDP_EXCEPTION: {
          const uint32_t count = packet.GetU32(&offset);

```
- **EN**: Implements logic around `GetU32`, `Printf`, `GetU64`, `GetCStr`.
- **CN**: 围绕 `GetU32`, `Printf`, `GetU64`, `GetCStr` 实现具体逻辑。

### Lines 1006-1041
```cpp
          for (uint32_t i = 0; i < count; ++i) {
            const uint32_t cpu = packet.GetU32(&offset);
            const uint32_t exc = packet.GetU32(&offset);
            const uint32_t code = packet.GetU32(&offset);
            const uint32_t subcode = packet.GetU32(&offset);
            const char *exc_cstr = NULL;
            switch (exc) {
            case 1:
              exc_cstr = "EXC_BAD_ACCESS";
              break;
            case 2:
              exc_cstr = "EXC_BAD_INSTRUCTION";
              break;
            case 3:
              exc_cstr = "EXC_ARITHMETIC";
              break;
            case 4:
              exc_cstr = "EXC_EMULATION";
              break;
            case 5:
              exc_cstr = "EXC_SOFTWARE";
              break;
            case 6:
              exc_cstr = "EXC_BREAKPOINT";
              break;
            case 7:
              exc_cstr = "EXC_SYSCALL";
              break;
            case 8:
              exc_cstr = "EXC_MACH_SYSCALL";
              break;
            case 9:
              exc_cstr = "EXC_RPC_ALERT";
              break;
            case 10:
              exc_cstr = "EXC_CRASH";
```
- **EN**: Implements logic around `GetU32`.
- **CN**: 围绕 `GetU32` 实现具体逻辑。

### Lines 1042-1059
```cpp
              break;
            default:
              break;
            }

            s.Printf("{ cpu = 0x%8.8x, exc = %s (%u), code = %u (0x%8.8x), "
                     "subcode = %u (0x%8.8x)} ",
                     cpu, exc_cstr, exc, code, code, subcode, subcode);
          }
        } break;

        case KDP_TERMINATION: {
          const uint32_t term_code = packet.GetU32(&offset);
          const uint32_t exit_code = packet.GetU32(&offset);
          s.Printf(" (term_code = 0x%8.8x (%u), exit_code = 0x%8.8x (%u))",
                   term_code, term_code, exit_code, exit_code);
        } break;

```
- **EN**: Implements logic around `Printf`, `u`, `GetU32`.
- **CN**: 围绕 `Printf`, `u`, `GetU32` 实现具体逻辑。

### Lines 1060-1090
```cpp
        case KDP_REATTACH: {
          const uint16_t reply_port = ntohs(packet.GetU16(&offset));
          s.Printf(" (reply_port = %u)", reply_port);
        } break;

        case KDP_READMSR64: {
          const uint32_t address = packet.GetU32(&offset);
          const uint16_t lcpu = packet.GetU16(&offset);
          s.Printf(" (address=0x%8.8x, lcpu=0x%4.4x)", address, lcpu);
        } break;

        case KDP_WRITEMSR64: {
          const uint32_t address = packet.GetU32(&offset);
          const uint16_t lcpu = packet.GetU16(&offset);
          const uint32_t nbytes = packet.GetByteSize() - offset;
          s.Printf(" (address=0x%8.8x, lcpu=0x%4.4x, nbytes=0x%8.8x)", lcpu,
                   address, nbytes);
          if (nbytes > 0)
            DumpDataExtractor(packet, 
                              &s,                   // Stream to dump to
                              offset,               // Offset within "packet"
                              eFormatHex,           // Format to use
                              1,                    // Size of each item in 
                                                    // bytes
                              nbytes,               // Number of items
                              16,                   // Number per line
                              LLDB_INVALID_ADDRESS, // Don't show addresses 
                                                    // before each line
                              0, 0);                // No bitfields
        } break;

```
- **EN**: Implements logic around `ntohs`, `Printf`, `GetU32`, `GetU16`, and 2 more symbols; this block maps executable state back to modules, symbols, sections, or addresses; controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `ntohs`, `Printf`, `GetU32`, `GetU16`, and 2 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并控制调试器侧的值格式化或 synthetic 展示。

### Lines 1091-1118
```cpp
        case KDP_READIOPORT: {
          const uint16_t lcpu = packet.GetU16(&offset);
          const uint16_t address = packet.GetU16(&offset);
          const uint16_t nbytes = packet.GetU16(&offset);
          s.Printf(" (lcpu=0x%4.4x, address=0x%4.4x, nbytes=%u)", lcpu, address,
                   nbytes);
        } break;

        case KDP_WRITEIOPORT: {
          const uint16_t lcpu = packet.GetU16(&offset);
          const uint16_t address = packet.GetU16(&offset);
          const uint16_t nbytes = packet.GetU16(&offset);
          s.Printf(" (lcpu = %u, addr = 0x%4.4x, nbytes = %u, bytes = \n", lcpu,
                   address, nbytes);
          if (nbytes > 0)
            DumpDataExtractor(packet, 
                              &s,                   // Stream to dump to
                              offset,               // Offset within "packet"
                              eFormatHex,           // Format to use
                              1,                    // Size of each item in 
                                                    // bytes
                              nbytes,               // Number of items
                              16,                   // Number per line
                              LLDB_INVALID_ADDRESS, // Don't show addresses 
                                                    // before each line
                              0, 0);                // No bitfields
        } break;

```
- **EN**: Implements logic around `GetU16`, `Printf`, `DumpDataExtractor`; this block maps executable state back to modules, symbols, sections, or addresses; controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `GetU16`, `Printf`, `DumpDataExtractor` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并控制调试器侧的值格式化或 synthetic 展示。

### Lines 1119-1140
```cpp
        case KDP_DUMPINFO: {
          const uint32_t count = packet.GetByteSize() - offset;
          s.Printf(" (count = %u, bytes = \n", count);
          if (count > 0)
            DumpDataExtractor(packet, 
                &s,                   // Stream to dump to
                offset,               // Offset within "packet"
                eFormatHex,           // Format to use
                1,                    // Size of each item in bytes
                count,                // Number of items
                16,                   // Number per line
                LLDB_INVALID_ADDRESS, // Don't show addresses before each line
                0, 0);                // No bitfields

        } break;
        }
      }
    } else {
      error_desc = "error: invalid packet command: ";
    }
  }

```
- **EN**: Implements logic around `GetByteSize`, `Printf`, `DumpDataExtractor`; this block propagates recoverable errors, status objects, or diagnostics; controls debugger-side formatting or synthetic presentation of values; registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `GetByteSize`, `Printf`, `DumpDataExtractor` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并控制调试器侧的值格式化或 synthetic 展示，并注册命令、插件或面向解释器的扩展点。

### Lines 1141-1174
```cpp
  if (error_desc) {
    s.PutCString(error_desc);

    DumpDataExtractor(packet,
                      &s,                   // Stream to dump to
                      0,                    // Offset into "packet"
                      eFormatBytes,         // Dump as hex bytes
                      1,                    // Size of each item is 1 for 
                                            // single bytes
                      packet.GetByteSize(), // Number of bytes
                      UINT32_MAX,           // Num bytes per line
                      LLDB_INVALID_ADDRESS, // Base address
                      0, 0);                // Bitfield info set to not do  
                                            // anything bitfield related
  }
}

uint32_t CommunicationKDP::SendRequestReadRegisters(uint32_t cpu,
                                                    uint32_t flavor, void *dst,
                                                    uint32_t dst_len,
                                                    Status &error) {
  PacketStreamType request_packet(Stream::eBinary, m_byte_order);
  const CommandType command = KDP_READREGS;
  // Size is header + 4 byte cpu and 4 byte flavor
  const uint32_t command_length = 8 + 4 + 4;
  MakeRequestPacketHeader(command, request_packet, command_length);
  request_packet.PutHex32(cpu);
  request_packet.PutHex32(flavor);
  DataExtractor reply_packet;
  if (SendRequestAndGetReply(command, request_packet, reply_packet)) {
    lldb::offset_t offset = 8;
    uint32_t kdp_error = reply_packet.GetU32(&offset);
    uint32_t src_len = reply_packet.GetByteSize() - 12;

```
- **EN**: Implements logic around `PutCString`, `DumpDataExtractor`, `GetByteSize`, `SendRequestReadRegisters`, and 5 more symbols; this block propagates recoverable errors, status objects, or diagnostics; maps executable state back to modules, symbols, sections, or addresses; registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `PutCString`, `DumpDataExtractor`, `GetByteSize`, `SendRequestReadRegisters`, and 5 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并将可执行状态映射回模块、符号、节区或地址，并注册命令、插件或面向解释器的扩展点。

### Lines 1175-1198
```cpp
    if (src_len > 0) {
      const uint32_t bytes_to_copy = std::min<uint32_t>(src_len, dst_len);
      const void *src = reply_packet.GetData(&offset, bytes_to_copy);
      if (src) {
        ::memcpy(dst, src, bytes_to_copy);
        error.Clear();
        // Return the number of bytes we could have returned regardless if we
        // copied them or not, just so we know when things don't match up
        return src_len;
      }
    }
    if (kdp_error)
      error = Status::FromErrorStringWithFormat(
          "failed to read kdp registers for cpu %u flavor %u (error %u)", cpu,
          flavor, kdp_error);
    else
      error = Status::FromErrorStringWithFormat(
          "failed to read kdp registers for cpu %u flavor %u", cpu, flavor);
  } else {
    error = Status::FromErrorString("failed to send packet");
  }
  return 0;
}

```
- **EN**: Implements logic around `min`, `GetData`, `memcpy`, `Clear`, and 3 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `min`, `GetData`, `memcpy`, `Clear`, and 3 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 1199-1226
```cpp
uint32_t CommunicationKDP::SendRequestWriteRegisters(uint32_t cpu,
                                                     uint32_t flavor,
                                                     const void *src,
                                                     uint32_t src_len,
                                                     Status &error) {
  PacketStreamType request_packet(Stream::eBinary, m_byte_order);
  const CommandType command = KDP_WRITEREGS;
  // Size is header + 4 byte cpu and 4 byte flavor
  const uint32_t command_length = 8 + 4 + 4 + src_len;
  MakeRequestPacketHeader(command, request_packet, command_length);
  request_packet.PutHex32(cpu);
  request_packet.PutHex32(flavor);
  request_packet.Write(src, src_len);
  DataExtractor reply_packet;
  if (SendRequestAndGetReply(command, request_packet, reply_packet)) {
    lldb::offset_t offset = 8;
    uint32_t kdp_error = reply_packet.GetU32(&offset);
    if (kdp_error == 0)
      return src_len;
    error = Status::FromErrorStringWithFormat(
        "failed to read kdp registers for cpu %u flavor %u (error %u)", cpu,
        flavor, kdp_error);
  } else {
    error = Status::FromErrorString("failed to send packet");
  }
  return 0;
}

```
- **EN**: Implements logic around `SendRequestWriteRegisters`, `request_packet`, `MakeRequestPacketHeader`, `PutHex32`, and 6 more symbols; this block propagates recoverable errors, status objects, or diagnostics; registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `SendRequestWriteRegisters`, `request_packet`, `MakeRequestPacketHeader`, `PutHex32`, and 6 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并注册命令、插件或面向解释器的扩展点。

### Lines 1227-1245
```cpp
bool CommunicationKDP::SendRequestResume() {
  PacketStreamType request_packet(Stream::eBinary, m_byte_order);
  const CommandType command = KDP_RESUMECPUS;
  const uint32_t command_length = 12;
  MakeRequestPacketHeader(command, request_packet, command_length);
  request_packet.PutHex32(GetCPUMask());

  DataExtractor reply_packet;
  return SendRequestAndGetReply(command, request_packet, reply_packet);
}

bool CommunicationKDP::SendRequestBreakpoint(bool set, addr_t addr) {
  PacketStreamType request_packet(Stream::eBinary, m_byte_order);
  bool use_64 = (GetVersion() >= 11);
  uint32_t command_addr_byte_size = use_64 ? 8 : 4;
  const CommandType command =
      set ? (use_64 ? KDP_BREAKPOINT_SET64 : KDP_BREAKPOINT_SET)
          : (use_64 ? KDP_BREAKPOINT_REMOVE64 : KDP_BREAKPOINT_REMOVE);

```
- **EN**: Implements logic around `SendRequestResume`, `request_packet`, `MakeRequestPacketHeader`, `PutHex32`, and 3 more symbols; this block registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `SendRequestResume`, `request_packet`, `MakeRequestPacketHeader`, `PutHex32`, and 3 more symbols 实现具体逻辑；该代码块注册命令、插件或面向解释器的扩展点。

### Lines 1246-1267
```cpp
  const uint32_t command_length = 8 + command_addr_byte_size;
  MakeRequestPacketHeader(command, request_packet, command_length);
  request_packet.PutMaxHex64(addr, command_addr_byte_size);

  DataExtractor reply_packet;
  if (SendRequestAndGetReply(command, request_packet, reply_packet)) {
    lldb::offset_t offset = 8;
    uint32_t kdp_error = reply_packet.GetU32(&offset);
    if (kdp_error == 0)
      return true;
  }
  return false;
}

bool CommunicationKDP::SendRequestSuspend() {
  PacketStreamType request_packet(Stream::eBinary, m_byte_order);
  const CommandType command = KDP_SUSPEND;
  const uint32_t command_length = 8;
  MakeRequestPacketHeader(command, request_packet, command_length);
  DataExtractor reply_packet;
  return SendRequestAndGetReply(command, request_packet, reply_packet);
}
```
- **EN**: Implements logic around `MakeRequestPacketHeader`, `PutMaxHex64`, `SendRequestAndGetReply`, `GetU32`, and 2 more symbols; this block registers commands, plugins, or interpreter-facing extension points.
- **CN**: 围绕 `MakeRequestPacketHeader`, `PutMaxHex64`, `SendRequestAndGetReply`, `GetU32`, and 2 more symbols 实现具体逻辑；该代码块注册命令、插件或面向解释器的扩展点。

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

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `CommunicationKDP.h`, `lldb/Core/DumpDataExtractor.h`, `lldb/Host/Host.h`, `lldb/Target/Process.h`, `lldb/Utility/DataBufferHeap.h`, `lldb/Utility/DataExtractor.h`, `lldb/Utility/FileSpec.h`, `lldb/Utility/Log.h`, `lldb/Utility/State.h`, `lldb/Utility/UUID.h` ... (+1 more)
- **Standard-library headers / 标准库头文件**: `<cerrno>`, `<climits>`, `<cstring>`
- **Subsystem categories / 子系统类别**: shared LLDB utility classes / 共享 LLDB 工具类 (6), LLDB core debugger abstractions / LLDB 核心调试抽象 (1), host OS abstraction helpers / 主机操作系统抽象辅助组件 (1), target, process, and thread control / 目标、进程与线程控制 (1)
