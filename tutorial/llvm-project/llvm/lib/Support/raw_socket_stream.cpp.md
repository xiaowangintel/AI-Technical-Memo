# raw_socket_stream.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/raw_socket_stream.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains raw_ostream implementations for streams to communicate via UNIX sockets
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `raw_socket_stream` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===-- llvm/Support/raw_socket_stream.cpp - Socket streams --*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains raw_ostream implementations for streams to communicate
// via UNIX sockets
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/raw_socket_stream.h"
#include "llvm/Config/config.h"
#include "llvm/Support/Error.h"
#include "llvm/Support/FileSystem.h"

#include <atomic>
#include <fcntl.h>
#include <functional>

#ifndef _WIN32
#include <poll.h>
#include <sys/socket.h>
#include <sys/un.h>
#else
#include "llvm/Support/Windows/WindowsSupport.h"
// winsock2.h must be included before afunix.h. Briefly turn off clang-format to
// avoid error.
// clang-format off
#include <winsock2.h>
#include <afunix.h>
// clang-format on
#include <io.h>
#endif // _WIN32

#if defined(HAVE_UNISTD_H)
#include <unistd.h>
#endif
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 15 direct dependencies, including `llvm/Support/raw_socket_stream.h`, `llvm/Config/config.h`, `llvm/Support/Error.h`, `llvm/Support/FileSystem.h`.
  CN: 引入了 15 个直接依赖，其中包括 `llvm/Support/raw_socket_stream.h`, `llvm/Config/config.h`, `llvm/Support/Error.h`, `llvm/Support/FileSystem.h`。
- EN: This range propagates LLVM-style errors and invariants.
  CN: 这一段传播 LLVM 风格的错误处理并维护不变量。

### Lines 41-80

```cpp

using namespace llvm;

#ifdef _WIN32
WSABalancer::WSABalancer() {
  WSADATA WsaData;
  ::memset(&WsaData, 0, sizeof(WsaData));
  if (WSAStartup(MAKEWORD(2, 2), &WsaData) != 0) {
    llvm::report_fatal_error("WSAStartup failed");
  }
}

WSABalancer::~WSABalancer() { WSACleanup(); }
#endif // _WIN32

static std::error_code getLastSocketErrorCode() {
#ifdef _WIN32
  return std::error_code(::WSAGetLastError(), std::system_category());
#else
  return errnoAsErrorCode();
#endif
}

static Expected<sockaddr_un> setSocketAddr(StringRef SocketPath) {
  struct sockaddr_un Addr;
  memset(&Addr, 0, sizeof(Addr));
  Addr.sun_family = AF_UNIX;

  if (sizeof(sockaddr_un::sun_path) <= SocketPath.size())
    return make_error<StringError>(
        std::make_error_code(std::errc::filename_too_long),
        "Socket path exceeds sockaddr_un::sun_path size limit");

  strncpy(Addr.sun_path, SocketPath.str().c_str(), sizeof(Addr.sun_path) - 1);
  return Addr;
}

static Expected<int> getSocketFD(StringRef SocketPath) {
#ifdef _WIN32
  SOCKET Socket = socket(AF_UNIX, SOCK_STREAM, 0);
```
- EN: This section centers on `WSABalancer`, `report_fatal_error`, `getLastSocketErrorCode` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `WSABalancer`, `report_fatal_error`, `getLastSocketErrorCode` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 81-120

```cpp
  if (Socket == INVALID_SOCKET) {
#else
  int Socket = socket(AF_UNIX, SOCK_STREAM, 0);
  if (Socket == -1) {
#endif // _WIN32
    return llvm::make_error<StringError>(getLastSocketErrorCode(),
                                         "Create socket failed");
  }

#ifdef __CYGWIN__
  // On Cygwin, UNIX sockets involve a handshake between connect and accept
  // to enable SO_PEERCRED/getpeereid handling.  This necessitates accept being
  // called before connect can return, but at least the tests in
  // llvm/unittests/Support/raw_socket_stream_test do both on the same thread
  // (first connect and then accept), resulting in a deadlock.  This call turns
  // off the handshake (and SO_PEERCRED/getpeereid support).
  setsockopt(Socket, SOL_SOCKET, SO_PEERCRED, NULL, 0);
#endif
  Expected<struct sockaddr_un> Addr = setSocketAddr(SocketPath);
  if (!Addr)
    return Addr.takeError();

  if (::connect(Socket, (struct sockaddr *)&*Addr, sizeof(*Addr)) == -1) {
    ::close(Socket);
    return llvm::make_error<StringError>(getLastSocketErrorCode(),
                                         "Connect socket failed");
  }

#ifdef _WIN32
  return _open_osfhandle(Socket, 0);
#else
  return Socket;
#endif // _WIN32
}

ListeningSocket::ListeningSocket(int SocketFD, StringRef SocketPath,
                                 int PipeFD[2])
    : FD(SocketFD), SocketPath(SocketPath), PipeFD{PipeFD[0], PipeFD[1]} {}

ListeningSocket::ListeningSocket(ListeningSocket &&LS)
```
- EN: This section centers on `setsockopt`, `_open_osfhandle` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `setsockopt`, `_open_osfhandle` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-160

```cpp
    : FD(LS.FD.load()), SocketPath(LS.SocketPath),
      PipeFD{LS.PipeFD[0], LS.PipeFD[1]} {

  LS.FD = -1;
  LS.SocketPath.clear();
  LS.PipeFD[0] = -1;
  LS.PipeFD[1] = -1;
}

Expected<ListeningSocket> ListeningSocket::createUnix(StringRef SocketPath,
                                                      int MaxBacklog) {

  // Handle instances where the target socket address already exists and
  // differentiate between a preexisting file with and without a bound socket
  //
  // ::bind will return std::errc:address_in_use if a file at the socket address
  // already exists (e.g., the file was not properly unlinked due to a crash)
  // even if another socket has not yet binded to that address
  if (llvm::sys::fs::exists(SocketPath)) {
    Expected<int> MaybeFD = getSocketFD(SocketPath);
    if (!MaybeFD) {

      // Regardless of the error, notify the caller that a file already exists
      // at the desired socket address and that there is no bound socket at that
      // address. The file must be removed before ::bind can use the address
      consumeError(MaybeFD.takeError());
      return llvm::make_error<StringError>(
          std::make_error_code(std::errc::file_exists),
          "Socket address unavailable");
    }
    ::close(std::move(*MaybeFD));

    // Notify caller that the provided socket address already has a bound socket
    return llvm::make_error<StringError>(
        std::make_error_code(std::errc::address_in_use),
        "Socket address unavailable");
  }

#ifdef _WIN32
  WSABalancer _;
```
- EN: This section centers on `createUnix`, `consumeError`, `make_error_code` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `createUnix`, `consumeError`, `make_error_code` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 161-200

```cpp
  SOCKET Socket = socket(AF_UNIX, SOCK_STREAM, 0);
  if (Socket == INVALID_SOCKET)
#else
  int Socket = socket(AF_UNIX, SOCK_STREAM, 0);
  if (Socket == -1)
#endif
    return llvm::make_error<StringError>(getLastSocketErrorCode(),
                                         "socket create failed");

#ifdef __CYGWIN__
  // On Cygwin, UNIX sockets involve a handshake between connect and accept
  // to enable SO_PEERCRED/getpeereid handling.  This necessitates accept being
  // called before connect can return, but at least the tests in
  // llvm/unittests/Support/raw_socket_stream_test do both on the same thread
  // (first connect and then accept), resulting in a deadlock.  This call turns
  // off the handshake (and SO_PEERCRED/getpeereid support).
  setsockopt(Socket, SOL_SOCKET, SO_PEERCRED, NULL, 0);
#endif
  Expected<struct sockaddr_un> Addr = setSocketAddr(SocketPath);
  if (!Addr)
    return Addr.takeError();

  if (::bind(Socket, (struct sockaddr *)&*Addr, sizeof(*Addr)) == -1) {
    // Grab error code from call to ::bind before calling ::close
    std::error_code EC = getLastSocketErrorCode();
    ::close(Socket);
    return llvm::make_error<StringError>(EC, "Bind error");
  }

  // Mark socket as passive so incoming connections can be accepted
  if (::listen(Socket, MaxBacklog) == -1)
    return llvm::make_error<StringError>(getLastSocketErrorCode(),
                                         "Listen error");

  int PipeFD[2];
#ifdef _WIN32
  // Reserve 1 byte for the pipe and use default textmode
  if (::_pipe(PipeFD, 1, 0) == -1)
#else
  if (::pipe(PipeFD) == -1)
```
- EN: This section centers on `setsockopt` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `setsockopt` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 201-240

```cpp
#endif // _WIN32
    return llvm::make_error<StringError>(getLastSocketErrorCode(),
                                         "pipe failed");

#ifdef _WIN32
  return ListeningSocket{_open_osfhandle(Socket, 0), SocketPath, PipeFD};
#else
  return ListeningSocket{Socket, SocketPath, PipeFD};
#endif // _WIN32
}

// If a file descriptor being monitored by ::poll is closed by another thread,
// the result is unspecified. In the case ::poll does not unblock and return,
// when ActiveFD is closed, you can provide another file descriptor via CancelFD
// that when written to will cause poll to return. Typically CancelFD is the
// read end of a unidirectional pipe.
//
// Timeout should be -1 to block indefinitly
//
// getActiveFD is a callback to handle ActiveFD's of std::atomic<int> and int
static std::error_code
manageTimeout(const std::chrono::milliseconds &Timeout,
              const std::function<int()> &getActiveFD,
              const std::optional<int> &CancelFD = std::nullopt) {
  struct pollfd FD[2];
  FD[0].events = POLLIN;
#ifdef _WIN32
  SOCKET WinServerSock = _get_osfhandle(getActiveFD());
  FD[0].fd = WinServerSock;
#else
  FD[0].fd = getActiveFD();
#endif
  uint8_t FDCount = 1;
  if (CancelFD.has_value()) {
    FD[1].events = POLLIN;
    FD[1].fd = CancelFD.value();
    FDCount++;
  }

  // Keep track of how much time has passed in case ::poll or WSAPoll are
```
- EN: This section centers on `manageTimeout` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `manageTimeout` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 241-280

```cpp
  // interupted by a signal and need to be recalled
  auto Start = std::chrono::steady_clock::now();
  auto RemainingTimeout = Timeout;
  int PollStatus = 0;
  do {
    // If Timeout is -1 then poll should block and RemainingTimeout does not
    // need to be recalculated
    if (PollStatus != 0 && Timeout != std::chrono::milliseconds(-1)) {
      auto TotalElapsedTime =
          std::chrono::duration_cast<std::chrono::milliseconds>(
              std::chrono::steady_clock::now() - Start);

      if (TotalElapsedTime >= Timeout)
        return std::make_error_code(std::errc::operation_would_block);

      RemainingTimeout = Timeout - TotalElapsedTime;
    }
#ifdef _WIN32
    PollStatus = WSAPoll(FD, FDCount, RemainingTimeout.count());
  } while (PollStatus == SOCKET_ERROR &&
           getLastSocketErrorCode() == std::errc::interrupted);
#else
    PollStatus = ::poll(FD, FDCount, RemainingTimeout.count());
  } while (PollStatus == -1 &&
           getLastSocketErrorCode() == std::errc::interrupted);
#endif

  // If ActiveFD equals -1 or CancelFD has data to be read then the operation
  // has been canceled by another thread
  if (getActiveFD() == -1 || (CancelFD.has_value() && FD[1].revents & POLLIN))
    return std::make_error_code(std::errc::operation_canceled);
#ifdef _WIN32
  if (PollStatus == SOCKET_ERROR)
#else
  if (PollStatus == -1)
#endif
    return getLastSocketErrorCode();
  if (PollStatus == 0)
    return std::make_error_code(std::errc::timed_out);
  if (FD[0].revents & POLLNVAL)
```
- EN: This section centers on `now`, `getLastSocketErrorCode` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `now`, `getLastSocketErrorCode` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 281-320

```cpp
    return std::make_error_code(std::errc::bad_file_descriptor);
  return std::error_code();
}

Expected<std::unique_ptr<raw_socket_stream>>
ListeningSocket::accept(const std::chrono::milliseconds &Timeout) {
  auto getActiveFD = [this]() -> int { return FD; };
  std::error_code TimeoutErr = manageTimeout(Timeout, getActiveFD, PipeFD[0]);
  if (TimeoutErr)
    return llvm::make_error<StringError>(TimeoutErr, "Timeout error");

  int AcceptFD;
#ifdef _WIN32
  SOCKET WinAcceptSock = ::accept(_get_osfhandle(FD), NULL, NULL);
  AcceptFD = _open_osfhandle(WinAcceptSock, 0);
#else
  AcceptFD = ::accept(FD, NULL, NULL);
#endif

  if (AcceptFD == -1)
    return llvm::make_error<StringError>(getLastSocketErrorCode(),
                                         "Socket accept failed");
  return std::make_unique<raw_socket_stream>(AcceptFD);
}

void ListeningSocket::shutdown() {
  int ObservedFD = FD.load();

  if (ObservedFD == -1)
    return;

  // If FD equals ObservedFD set FD to -1; If FD doesn't equal ObservedFD then
  // another thread is responsible for shutdown so return
  if (!FD.compare_exchange_strong(ObservedFD, -1))
    return;

  ::close(ObservedFD);
  ::unlink(SocketPath.c_str());

  // Ensure ::poll returns if shutdown is called by a separate thread
```
- EN: This section centers on `make_error_code`, `error_code`, `accept` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `make_error_code`, `error_code`, `accept` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 321-360

```cpp
  char Byte = 'A';
  ssize_t written = ::write(PipeFD[1], &Byte, 1);

  // Ignore any write() error
  (void)written;
}

ListeningSocket::~ListeningSocket() {
  shutdown();

  // Close the pipe's FDs in the destructor instead of within
  // ListeningSocket::shutdown to avoid unnecessary synchronization issues that
  // would occur as PipeFD's values would have to be changed to -1
  //
  // The move constructor sets PipeFD to -1
  if (PipeFD[0] != -1)
    ::close(PipeFD[0]);
  if (PipeFD[1] != -1)
    ::close(PipeFD[1]);
}

//===----------------------------------------------------------------------===//
//  raw_socket_stream
//===----------------------------------------------------------------------===//

raw_socket_stream::raw_socket_stream(int SocketFD)
    : raw_fd_stream(SocketFD, true) {}

raw_socket_stream::~raw_socket_stream() = default;

Expected<std::unique_ptr<raw_socket_stream>>
raw_socket_stream::createConnectedUnix(StringRef SocketPath) {
#ifdef _WIN32
  WSABalancer _;
#endif // _WIN32
  Expected<int> FD = getSocketFD(SocketPath);
  if (!FD)
    return FD.takeError();
  return std::make_unique<raw_socket_stream>(*FD);
}
```
- EN: This section centers on `shutdown`, `raw_socket_stream`, `createConnectedUnix` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `shutdown`, `raw_socket_stream`, `createConnectedUnix` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 361-372

```cpp

ssize_t raw_socket_stream::read(char *Ptr, size_t Size,
                                const std::chrono::milliseconds &Timeout) {
  auto getActiveFD = [this]() -> int { return this->get_fd(); };
  std::error_code Err = manageTimeout(Timeout, getActiveFD);
  // Mimic raw_fd_stream::read error handling behavior
  if (Err) {
    raw_fd_stream::error_detected(Err);
    return -1;
  }
  return raw_fd_stream::read(Ptr, Size);
}
```
- EN: This section centers on `read`, `error_detected` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `read`, `error_detected` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `sockaddr_un`, `sockaddr`, `pollfd`, `WSABalancer`, `report_fatal_error`, `getLastSocketErrorCode`, `error_code` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Parsing, decoding, and structural validation. / 重点关注解析、解码与结构校验。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/raw_socket_stream.h`, `llvm/Config/config.h`, `llvm/Support/Error.h`, `llvm/Support/FileSystem.h`, `llvm/Support/Windows/WindowsSupport.h`
- Standard library / 标准库: `atomic`, `fcntl.h`, `functional`, `unistd.h`
- Other/system headers / 其他或系统头文件: `poll.h`, `sys/socket.h`, `sys/un.h`, `winsock2.h`, `afunix.h`, `io.h`
- Related symbols / 相关符号: `sockaddr_un`, `sockaddr`, `pollfd`, `WSABalancer`, `report_fatal_error`, `getLastSocketErrorCode`, `error_code`, `errnoAsErrorCode`
