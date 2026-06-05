# SimpleRemoteEPCUtils.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/Orc/Shared/SimpleRemoteEPCUtils.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Utils for Simple Remote EPC.
  - **CN**: 实现 ORC JIT 基础设施，例如 ExecutionSession、JITDylib、物化流程、符号查找以及执行辅助组件。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===------ SimpleRemoteEPCUtils.cpp - Utils for Simple Remote EPC --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-17
```cpp
//
// Message definitions and other utilities for SimpleRemoteEPC and
// SimpleRemoteEPCServer.
//
//===----------------------------------------------------------------------===//

#include "llvm/ExecutionEngine/Orc/Shared/SimpleRemoteEPCUtils.h"
#include "llvm/Config/llvm-config.h" // for LLVM_ENABLE_THREADS
#include "llvm/Support/Endian.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/Orc/Shared/SimpleRemoteEPCUtils.h`, `llvm/Config/llvm-config.h`, `llvm/Support/Endian.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/Orc/Shared/SimpleRemoteEPCUtils.h`, `llvm/Config/llvm-config.h`, `llvm/Support/Endian.h`。

### Lines 18-25
```cpp
#if !defined(_MSC_VER) && !defined(__MINGW32__)
#include <unistd.h>
#else
#include <io.h>
#endif

namespace {

```
- **EN**: Pulls in the headers needed for this implementation, including `unistd.h`, `io.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `unistd.h`, `io.h`。

### Lines 26-33
```cpp
struct FDMsgHeader {
  static constexpr unsigned MsgSizeOffset = 0;
  static constexpr unsigned OpCOffset = MsgSizeOffset + sizeof(uint64_t);
  static constexpr unsigned SeqNoOffset = OpCOffset + sizeof(uint64_t);
  static constexpr unsigned TagAddrOffset = SeqNoOffset + sizeof(uint64_t);
  static constexpr unsigned Size = TagAddrOffset + sizeof(uint64_t);
};

```
- **EN**: Introduces declarations for `FDMsgHeader`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `FDMsgHeader` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 34-43
```cpp
} // namespace

namespace llvm {
namespace orc {
namespace SimpleRemoteEPCDefaultBootstrapSymbolNames {

const char *ExecutorSessionObjectName =
    "__llvm_orc_SimpleRemoteEPC_dispatch_ctx";
const char *DispatchFnName = "__llvm_orc_SimpleRemoteEPC_dispatch_fn";

```
- **EN**: Introduces declarations for `llvm`, `orc`, `SimpleRemoteEPCDefaultBootstrapSymbolNames`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `orc`, `SimpleRemoteEPCDefaultBootstrapSymbolNames` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 44-57
```cpp
} // end namespace SimpleRemoteEPCDefaultBootstrapSymbolNames

SimpleRemoteEPCTransportClient::~SimpleRemoteEPCTransportClient() = default;
SimpleRemoteEPCTransport::~SimpleRemoteEPCTransport() = default;

Expected<std::unique_ptr<FDSimpleRemoteEPCTransport>>
FDSimpleRemoteEPCTransport::Create(SimpleRemoteEPCTransportClient &C, int InFD,
                                   int OutFD) {
#if LLVM_ENABLE_THREADS
  if (InFD == -1)
    return make_error<StringError>("Invalid input file descriptor " +
                                       Twine(InFD),
                                   inconvertibleErrorCode());
  if (OutFD == -1)
```
- **EN**: Introduces declarations for `SimpleRemoteEPCDefaultBootstrapSymbolNames`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `SimpleRemoteEPCDefaultBootstrapSymbolNames` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 58-71
```cpp
    return make_error<StringError>("Invalid output file descriptor " +
                                       Twine(OutFD),
                                   inconvertibleErrorCode());
  std::unique_ptr<FDSimpleRemoteEPCTransport> FDT(
      new FDSimpleRemoteEPCTransport(C, InFD, OutFD));
  return std::move(FDT);
#else
  return make_error<StringError>("FD-based SimpleRemoteEPC transport requires "
                                 "thread support, but llvm was built with "
                                 "LLVM_ENABLE_THREADS=Off",
                                 inconvertibleErrorCode());
#endif
}

```
- **EN**: Implements logic around `make_error<StringError>`, `Twine`, `inconvertibleErrorCode`, `FDT`, and 2 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<StringError>`, `Twine`, `inconvertibleErrorCode`, `FDT`, and 2 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 72-85
```cpp
FDSimpleRemoteEPCTransport::~FDSimpleRemoteEPCTransport() {
#if LLVM_ENABLE_THREADS
  ListenerThread.join();
#endif
}

Error FDSimpleRemoteEPCTransport::start() {
#if LLVM_ENABLE_THREADS
  ListenerThread = std::thread([this]() { listenLoop(); });
  return Error::success();
#endif
  llvm_unreachable("Should not be called with LLVM_ENABLE_THREADS=Off");
}

```
- **EN**: Implements logic around `~FDSimpleRemoteEPCTransport`, `join`, `start`, `thread`, and 2 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `~FDSimpleRemoteEPCTransport`, `join`, `start`, `thread`, and 2 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 86-99
```cpp
Error FDSimpleRemoteEPCTransport::sendMessage(SimpleRemoteEPCOpcode OpC,
                                              uint64_t SeqNo,
                                              ExecutorAddr TagAddr,
                                              ArrayRef<char> ArgBytes) {
  char HeaderBuffer[FDMsgHeader::Size];

  *((support::ulittle64_t *)(HeaderBuffer + FDMsgHeader::MsgSizeOffset)) =
      FDMsgHeader::Size + ArgBytes.size();
  *((support::ulittle64_t *)(HeaderBuffer + FDMsgHeader::OpCOffset)) =
      static_cast<uint64_t>(OpC);
  *((support::ulittle64_t *)(HeaderBuffer + FDMsgHeader::SeqNoOffset)) = SeqNo;
  *((support::ulittle64_t *)(HeaderBuffer + FDMsgHeader::TagAddrOffset)) =
      TagAddr.getValue();

```
- **EN**: Implements logic around `sendMessage`, `size`, `static_cast<uint64_t>`, `getValue`.
- **CN**: 围绕 `sendMessage`, `size`, `static_cast<uint64_t>`, `getValue` 实现具体逻辑。

### Lines 100-110
```cpp
  std::lock_guard<std::mutex> Lock(M);
  if (Disconnected)
    return make_error<StringError>("FD-transport disconnected",
                                   inconvertibleErrorCode());
  if (int ErrNo = writeBytes(HeaderBuffer, FDMsgHeader::Size))
    return errorCodeToError(std::error_code(ErrNo, std::generic_category()));
  if (int ErrNo = writeBytes(ArgBytes.data(), ArgBytes.size()))
    return errorCodeToError(std::error_code(ErrNo, std::generic_category()));
  return Error::success();
}

```
- **EN**: Implements logic around `Lock`, `make_error<StringError>`, `inconvertibleErrorCode`, `errorCodeToError`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `Lock`, `make_error<StringError>`, `inconvertibleErrorCode`, `errorCodeToError`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 111-117
```cpp
void FDSimpleRemoteEPCTransport::disconnect() {
  if (Disconnected)
    return; // Return if already disconnected.

  Disconnected = true;
  bool CloseOutFD = InFD != OutFD;

```
- **EN**: Implements logic around `disconnect`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `disconnect` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 118-131
```cpp
  // Close InFD.
  while (close(InFD) == -1) {
    if (errno == EBADF)
      break;
  }

  // Close OutFD.
  if (CloseOutFD) {
    while (close(OutFD) == -1) {
      if (errno == EBADF)
        break;
    }
  }
}
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 132-145
```cpp

static Error makeUnexpectedEOFError() {
  return make_error<StringError>("Unexpected end-of-file",
                                 inconvertibleErrorCode());
}

Error FDSimpleRemoteEPCTransport::readBytes(char *Dst, size_t Size,
                                            bool *IsEOF) {
  assert((Size == 0 || Dst) && "Attempt to read into null.");
  ssize_t Completed = 0;
  while (Completed < static_cast<ssize_t>(Size)) {
    ssize_t Read = ::read(InFD, Dst + Completed, Size - Completed);
    if (Read <= 0) {
      auto ErrNo = errno;
```
- **EN**: Implements logic around `makeUnexpectedEOFError`, `make_error<StringError>`, `inconvertibleErrorCode`, `readBytes`, and 2 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `makeUnexpectedEOFError`, `make_error<StringError>`, `inconvertibleErrorCode`, `readBytes`, and 2 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 146-159
```cpp
      if (Read == 0) {
        if (Completed == 0 && IsEOF) {
          *IsEOF = true;
          return Error::success();
        } else
          return makeUnexpectedEOFError();
      } else if (ErrNo == EAGAIN || ErrNo == EINTR)
        continue;
      else {
        std::lock_guard<std::mutex> Lock(M);
        if (Disconnected && IsEOF) { // disconnect called,  pretend this is EOF.
          *IsEOF = true;
          return Error::success();
        }
```
- **EN**: Implements logic around `success`, `makeUnexpectedEOFError`, `Lock`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `makeUnexpectedEOFError`, `Lock` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 160-168
```cpp
        return errorCodeToError(
            std::error_code(ErrNo, std::generic_category()));
      }
    }
    Completed += Read;
  }
  return Error::success();
}

```
- **EN**: Implements logic around `errorCodeToError`, `error_code`, `success`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `errorCodeToError`, `error_code`, `success` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 169-182
```cpp
int FDSimpleRemoteEPCTransport::writeBytes(const char *Src, size_t Size) {
  assert((Size == 0 || Src) && "Attempt to append from null.");
  ssize_t Completed = 0;
  while (Completed < static_cast<ssize_t>(Size)) {
    ssize_t Written = ::write(OutFD, Src + Completed, Size - Completed);
    if (Written < 0) {
      auto ErrNo = errno;
      if (ErrNo == EAGAIN || ErrNo == EINTR)
        continue;
      else
        return ErrNo;
    }
    Completed += Written;
  }
```
- **EN**: Implements logic around `writeBytes`, `assert`, `write`; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `writeBytes`, `assert`, `write` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 183-189
```cpp
  return 0;
}

void FDSimpleRemoteEPCTransport::listenLoop() {
  Error Err = Error::success();
  do {

```
- **EN**: Implements logic around `listenLoop`, `success`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `listenLoop`, `success` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 190-201
```cpp
    char HeaderBuffer[FDMsgHeader::Size];
    // Read the header buffer.
    {
      bool IsEOF = false;
      if (auto Err2 = readBytes(HeaderBuffer, FDMsgHeader::Size, &IsEOF)) {
        Err = joinErrors(std::move(Err), std::move(Err2));
        break;
      }
      if (IsEOF)
        break;
    }

```
- **EN**: Implements logic around `joinErrors`.
- **CN**: 围绕 `joinErrors` 实现具体逻辑。

### Lines 202-215
```cpp
    // Decode header buffer.
    uint64_t MsgSize;
    SimpleRemoteEPCOpcode OpC;
    uint64_t SeqNo;
    ExecutorAddr TagAddr;

    MsgSize =
        *((support::ulittle64_t *)(HeaderBuffer + FDMsgHeader::MsgSizeOffset));
    OpC = static_cast<SimpleRemoteEPCOpcode>(static_cast<uint64_t>(
        *((support::ulittle64_t *)(HeaderBuffer + FDMsgHeader::OpCOffset))));
    SeqNo =
        *((support::ulittle64_t *)(HeaderBuffer + FDMsgHeader::SeqNoOffset));
    TagAddr.setValue(
        *((support::ulittle64_t *)(HeaderBuffer + FDMsgHeader::TagAddrOffset)));
```
- **EN**: Implements logic around `static_cast<SimpleRemoteEPCOpcode>`, `setValue`.
- **CN**: 围绕 `static_cast<SimpleRemoteEPCOpcode>`, `setValue` 实现具体逻辑。

### Lines 216-223
```cpp

    if (MsgSize < FDMsgHeader::Size) {
      Err = joinErrors(std::move(Err),
                       make_error<StringError>("Message size too small",
                                               inconvertibleErrorCode()));
      break;
    }

```
- **EN**: Implements logic around `joinErrors`, `make_error<StringError>`, `inconvertibleErrorCode`.
- **CN**: 围绕 `joinErrors`, `make_error<StringError>`, `inconvertibleErrorCode` 实现具体逻辑。

### Lines 224-231
```cpp
    // Read the argument bytes.
    auto ArgBytes =
        shared::WrapperFunctionBuffer::allocate(MsgSize - FDMsgHeader::Size);
    if (auto Err2 = readBytes(ArgBytes.data(), ArgBytes.size())) {
      Err = joinErrors(std::move(Err), std::move(Err2));
      break;
    }

```
- **EN**: Implements logic around `allocate`, `joinErrors`.
- **CN**: 围绕 `allocate`, `joinErrors` 实现具体逻辑。

### Lines 232-241
```cpp
    if (auto Action =
            C.handleMessage(OpC, SeqNo, TagAddr, std::move(ArgBytes))) {
      if (*Action == SimpleRemoteEPCTransportClient::EndSession)
        break;
    } else {
      Err = joinErrors(std::move(Err), Action.takeError());
      break;
    }
  } while (true);

```
- **EN**: Implements logic around `handleMessage`, `joinErrors`.
- **CN**: 围绕 `handleMessage`, `joinErrors` 实现具体逻辑。

### Lines 242-249
```cpp
  // Attempt to close FDs, set Disconnected to true so that subsequent
  // sendMessage calls fail.
  disconnect();

  // Call up to the client to handle the disconnection.
  C.handleDisconnect(std::move(Err));
}

```
- **EN**: Implements logic around `disconnect`, `handleDisconnect`.
- **CN**: 围绕 `disconnect`, `handleDisconnect` 实现具体逻辑。

### Lines 250-251
```cpp
} // end namespace orc
} // end namespace llvm
```
- **EN**: Introduces declarations for `orc`, `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `orc`, `llvm` 等声明，定义本文件后续使用的数据结构或接口。

## Key Concepts / 关键概念

- **ORC JIT / ORC JIT**:
  - **EN**: Coordinates JITDylibs, symbol materialization, execution sessions, and asynchronous compilation flows
  - **CN**: 协调 JITDylib、符号物化、ExecutionSession 与异步编译流程

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/Orc/Shared/SimpleRemoteEPCUtils.h`, `llvm/Config/llvm-config.h`, `llvm/Support/Endian.h`, `unistd.h`, `io.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Support
