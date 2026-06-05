# ExecutorSharedMemoryMapperService.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/Orc/TargetProcess/ExecutorSharedMemoryMapperService.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements ORC JIT infrastructure such as execution sessions, JITDylibs, materialization, symbol lookup, and execution helpers.
  - **CN**: 实现 ORC JIT 基础设施，例如 ExecutionSession、JITDylib、物化流程、符号查找以及执行辅助组件。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
//===---------- ExecutorSharedMemoryMapperService.cpp -----------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/ExecutionEngine/Orc/TargetProcess/ExecutorSharedMemoryMapperService.h"
#include "llvm/Config/llvm-config.h" // for LLVM_ON_UNIX
#include "llvm/ExecutionEngine/Orc/Shared/OrcRTBridge.h"
#include "llvm/Support/Process.h"
#include "llvm/Support/WindowsError.h"
#include <sstream>

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/Orc/TargetProcess/ExecutorSharedMemoryMapperService.h`, `llvm/Config/llvm-config.h`, `llvm/ExecutionEngine/Orc/Shared/OrcRTBridge.h`, `llvm/Support/Process.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/Orc/TargetProcess/ExecutorSharedMemoryMapperService.h`, `llvm/Config/llvm-config.h`, `llvm/ExecutionEngine/Orc/Shared/OrcRTBridge.h`, `llvm/Support/Process.h`。

### Lines 16-26
```cpp
#if defined(LLVM_ON_UNIX)
#include <errno.h>
#include <fcntl.h>
#include <sys/mman.h>
#if defined(__MVS__)
#include "llvm/Support/BLAKE3.h"
#include <sys/shm.h>
#endif
#include <unistd.h>
#endif

```
- **EN**: Pulls in the headers needed for this implementation, including `errno.h`, `fcntl.h`, `sys/mman.h`, `llvm/Support/BLAKE3.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `errno.h`, `fcntl.h`, `sys/mman.h`, `llvm/Support/BLAKE3.h`。

### Lines 27-44
```cpp
namespace llvm {
namespace orc {
namespace rt_bootstrap {

#if defined(_WIN32)
static DWORD getWindowsProtectionFlags(MemProt MP) {
  if (MP == MemProt::Read)
    return PAGE_READONLY;
  if (MP == MemProt::Write ||
      MP == (MemProt::Write | MemProt::Read)) {
    // Note: PAGE_WRITE is not supported by VirtualProtect
    return PAGE_READWRITE;
  }
  if (MP == (MemProt::Read | MemProt::Exec))
    return PAGE_EXECUTE_READ;
  if (MP == (MemProt::Read | MemProt::Write | MemProt::Exec))
    return PAGE_EXECUTE_READWRITE;
  if (MP == MemProt::Exec)
```
- **EN**: Introduces declarations for `llvm`, `orc`, `rt_bootstrap`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `orc`, `rt_bootstrap` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 45-54
```cpp
    return PAGE_EXECUTE;

  return PAGE_NOACCESS;
}
#endif

Expected<std::pair<ExecutorAddr, std::string>>
ExecutorSharedMemoryMapperService::reserve(uint64_t Size) {
#if (defined(LLVM_ON_UNIX) && !defined(__ANDROID__)) || defined(_WIN32)

```
- **EN**: Implements logic around `reserve`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `reserve` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 55-64
```cpp
#if defined(LLVM_ON_UNIX)

  std::string SharedMemoryName;
  {
    std::stringstream SharedMemoryNameStream;
    SharedMemoryNameStream << "/jitlink_" << sys::Process::getProcessId() << '_'
                           << (++SharedMemoryCount);
    SharedMemoryName = SharedMemoryNameStream.str();
  }

```
- **EN**: Implements logic around `getProcessId`, `str`.
- **CN**: 围绕 `getProcessId`, `str` 实现具体逻辑。

### Lines 65-75
```cpp
#if defined(__MVS__)
  ArrayRef<uint8_t> Data(
      reinterpret_cast<const uint8_t *>(SharedMemoryName.c_str()),
      SharedMemoryName.size());
  auto HashedName = BLAKE3::hash<sizeof(key_t)>(Data);
  key_t Key = *reinterpret_cast<key_t *>(HashedName.data());
  int SharedMemoryId =
      shmget(Key, Size, IPC_CREAT | IPC_EXCL | __IPC_SHAREAS | 0700);
  if (SharedMemoryId < 0)
    return errorCodeToError(errnoAsErrorCode());

```
- **EN**: Implements logic around `Data`, `c_str`, `size`, `hash<sizeof`, and 3 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `Data`, `c_str`, `size`, `hash<sizeof`, and 3 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 76-84
```cpp
  void *Addr = shmat(SharedMemoryId, nullptr, 0);
  if (Addr == reinterpret_cast<void *>(-1))
    return errorCodeToError(errnoAsErrorCode());
#else
  int SharedMemoryFile =
      shm_open(SharedMemoryName.c_str(), O_RDWR | O_CREAT | O_EXCL, 0700);
  if (SharedMemoryFile < 0)
    return errorCodeToError(errnoAsErrorCode());

```
- **EN**: Implements logic around `shmat`, `errorCodeToError`, `shm_open`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `shmat`, `errorCodeToError`, `shm_open` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 85-95
```cpp
  // by default size is 0
  if (ftruncate(SharedMemoryFile, Size) < 0)
    return errorCodeToError(errnoAsErrorCode());

  void *Addr = mmap(nullptr, Size, PROT_NONE, MAP_SHARED, SharedMemoryFile, 0);
  if (Addr == MAP_FAILED)
    return errorCodeToError(errnoAsErrorCode());

  close(SharedMemoryFile);
#endif

```
- **EN**: Implements logic around `errorCodeToError`, `mmap`, `close`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `errorCodeToError`, `mmap`, `close` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 96-105
```cpp
#elif defined(_WIN32)

  std::string SharedMemoryName;
  {
    std::stringstream SharedMemoryNameStream;
    SharedMemoryNameStream << "jitlink_" << sys::Process::getProcessId() << '_'
                           << (++SharedMemoryCount);
    SharedMemoryName = SharedMemoryNameStream.str();
  }

```
- **EN**: Implements logic around `getProcessId`, `str`.
- **CN**: 围绕 `getProcessId`, `str` 实现具体逻辑。

### Lines 106-120
```cpp
  std::wstring WideSharedMemoryName(SharedMemoryName.begin(),
                                    SharedMemoryName.end());
  HANDLE SharedMemoryFile = CreateFileMappingW(
      INVALID_HANDLE_VALUE, NULL, PAGE_EXECUTE_READWRITE, Size >> 32,
      Size & 0xffffffff, WideSharedMemoryName.c_str());
  if (!SharedMemoryFile)
    return errorCodeToError(mapWindowsError(GetLastError()));

  void *Addr = MapViewOfFile(SharedMemoryFile,
                             FILE_MAP_ALL_ACCESS | FILE_MAP_EXECUTE, 0, 0, 0);
  if (!Addr) {
    CloseHandle(SharedMemoryFile);
    return errorCodeToError(mapWindowsError(GetLastError()));
  }

```
- **EN**: Implements logic around `WideSharedMemoryName`, `end`, `CreateFileMappingW`, `c_str`, and 3 more symbols; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `WideSharedMemoryName`, `end`, `CreateFileMappingW`, `c_str`, and 3 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 121-130
```cpp
#endif

  {
    std::lock_guard<std::mutex> Lock(Mutex);
    Reservations[Addr].Size = Size;
#if defined(_WIN32)
    Reservations[Addr].SharedMemoryFile = SharedMemoryFile;
#endif
  }

```
- **EN**: Implements logic around `Lock`.
- **CN**: 围绕 `Lock` 实现具体逻辑。

### Lines 131-139
```cpp
  return std::make_pair(ExecutorAddr::fromPtr(Addr),
                        std::move(SharedMemoryName));
#else
  return make_error<StringError>(
      "SharedMemoryMapper is not supported on this platform yet",
      inconvertibleErrorCode());
#endif
}

```
- **EN**: Implements logic around `make_pair`, `move`, `make_error<StringError>`, `inconvertibleErrorCode`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `make_pair`, `move`, `make_error<StringError>`, `inconvertibleErrorCode` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 140-150
```cpp
Expected<ExecutorAddr> ExecutorSharedMemoryMapperService::initialize(
    ExecutorAddr Reservation, tpctypes::SharedMemoryFinalizeRequest &FR) {
#if (defined(LLVM_ON_UNIX) && !defined(__ANDROID__)) || defined(_WIN32)

  ExecutorAddr MinAddr(~0ULL);

  // Contents are already in place
  for (auto &Segment : FR.Segments) {
    if (Segment.Addr < MinAddr)
      MinAddr = Segment.Addr;

```
- **EN**: Implements logic around `initialize`, `MinAddr`.
- **CN**: 围绕 `initialize`, `MinAddr` 实现具体逻辑。

### Lines 151-163
```cpp
#if defined(LLVM_ON_UNIX)

#if defined(__MVS__)
      // TODO Is it possible to change the protection level?
#else
    int NativeProt = 0;
    if ((Segment.RAG.Prot & MemProt::Read) == MemProt::Read)
      NativeProt |= PROT_READ;
    if ((Segment.RAG.Prot & MemProt::Write) == MemProt::Write)
      NativeProt |= PROT_WRITE;
    if ((Segment.RAG.Prot & MemProt::Exec) == MemProt::Exec)
      NativeProt |= PROT_EXEC;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 164-175
```cpp
    if (mprotect(Segment.Addr.toPtr<void *>(), Segment.Size, NativeProt))
      return errorCodeToError(errnoAsErrorCode());
#endif

#elif defined(_WIN32)

    DWORD NativeProt = getWindowsProtectionFlags(Segment.RAG.Prot);

    if (!VirtualProtect(Segment.Addr.toPtr<void *>(), Segment.Size, NativeProt,
                        &NativeProt))
      return errorCodeToError(mapWindowsError(GetLastError()));

```
- **EN**: Implements logic around `errorCodeToError`, `getWindowsProtectionFlags`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `errorCodeToError`, `getWindowsProtectionFlags` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 176-188
```cpp
#endif

    if ((Segment.RAG.Prot & MemProt::Exec) == MemProt::Exec)
      sys::Memory::InvalidateInstructionCache(Segment.Addr.toPtr<void *>(),
                                              Segment.Size);
  }

  // Run finalization actions and get deinitlization action list.
  auto DeinitializeActions = shared::runFinalizeActions(FR.Actions);
  if (!DeinitializeActions) {
    return DeinitializeActions.takeError();
  }

```
- **EN**: Implements logic around `InvalidateInstructionCache`, `runFinalizeActions`, `takeError`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `InvalidateInstructionCache`, `runFinalizeActions`, `takeError` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 189-197
```cpp
  {
    std::lock_guard<std::mutex> Lock(Mutex);
    Allocations[MinAddr].DeinitializationActions =
        std::move(*DeinitializeActions);
    Reservations[Reservation.toPtr<void *>()].Allocations.push_back(MinAddr);
  }

  return MinAddr;

```
- **EN**: Implements logic around `Lock`, `move`, `push_back`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `Lock`, `move`, `push_back` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 198-208
```cpp
#else
  return make_error<StringError>(
      "SharedMemoryMapper is not supported on this platform yet",
      inconvertibleErrorCode());
#endif
}

Error ExecutorSharedMemoryMapperService::deinitialize(
    const std::vector<ExecutorAddr> &Bases) {
  Error AllErr = Error::success();

```
- **EN**: Implements logic around `make_error<StringError>`, `inconvertibleErrorCode`, `deinitialize`, `success`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<StringError>`, `inconvertibleErrorCode`, `deinitialize`, `success` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 209-217
```cpp
  {
    std::lock_guard<std::mutex> Lock(Mutex);

    for (auto Base : llvm::reverse(Bases)) {
      if (Error Err = shared::runDeallocActions(
              Allocations[Base].DeinitializationActions)) {
        AllErr = joinErrors(std::move(AllErr), std::move(Err));
      }

```
- **EN**: Implements logic around `Lock`, `joinErrors`.
- **CN**: 围绕 `Lock`, `joinErrors` 实现具体逻辑。

### Lines 218-226
```cpp
      // Remove the allocation from the allocation list of its reservation
      for (auto &Reservation : Reservations) {
        auto AllocationIt = llvm::find(Reservation.second.Allocations, Base);
        if (AllocationIt != Reservation.second.Allocations.end()) {
          Reservation.second.Allocations.erase(AllocationIt);
          break;
        }
      }

```
- **EN**: Implements logic around `find`, `erase`.
- **CN**: 围绕 `find`, `erase` 实现具体逻辑。

### Lines 227-238
```cpp
      Allocations.erase(Base);
    }
  }

  return AllErr;
}

Error ExecutorSharedMemoryMapperService::release(
    const std::vector<ExecutorAddr> &Bases) {
#if (defined(LLVM_ON_UNIX) && !defined(__ANDROID__)) || defined(_WIN32)
  Error Err = Error::success();

```
- **EN**: Implements logic around `erase`, `release`, `success`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `erase`, `release`, `success` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 239-251
```cpp
  for (auto Base : Bases) {
    std::vector<ExecutorAddr> AllocAddrs;
    size_t Size;

#if defined(_WIN32)
    HANDLE SharedMemoryFile;
#endif

    {
      std::lock_guard<std::mutex> Lock(Mutex);
      auto &R = Reservations[Base.toPtr<void *>()];
      Size = R.Size;

```
- **EN**: Implements logic around `Lock`.
- **CN**: 围绕 `Lock` 实现具体逻辑。

### Lines 252-262
```cpp
#if defined(_WIN32)
      SharedMemoryFile = R.SharedMemoryFile;
#endif

      AllocAddrs.swap(R.Allocations);
    }

    // deinitialize sub allocations
    if (Error E = deinitialize(AllocAddrs))
      Err = joinErrors(std::move(Err), std::move(E));

```
- **EN**: Implements logic around `swap`, `joinErrors`.
- **CN**: 围绕 `swap`, `joinErrors` 实现具体逻辑。

### Lines 263-274
```cpp
#if defined(LLVM_ON_UNIX)

#if defined(__MVS__)
    (void)Size;

    if (shmdt(Base.toPtr<void *>()) < 0)
      Err = joinErrors(std::move(Err), errorCodeToError(errnoAsErrorCode()));
#else
    if (munmap(Base.toPtr<void *>(), Size) != 0)
      Err = joinErrors(std::move(Err), errorCodeToError(errnoAsErrorCode()));
#endif

```
- **EN**: Implements logic around `joinErrors`.
- **CN**: 围绕 `joinErrors` 实现具体逻辑。

### Lines 275-283
```cpp
#elif defined(_WIN32)
    (void)Size;

    if (!UnmapViewOfFile(Base.toPtr<void *>()))
      Err = joinErrors(std::move(Err),
                       errorCodeToError(mapWindowsError(GetLastError())));

    CloseHandle(SharedMemoryFile);

```
- **EN**: Implements logic around `joinErrors`, `errorCodeToError`, `CloseHandle`.
- **CN**: 围绕 `joinErrors`, `errorCodeToError`, `CloseHandle` 实现具体逻辑。

### Lines 284-297
```cpp
#endif

    std::lock_guard<std::mutex> Lock(Mutex);
    Reservations.erase(Base.toPtr<void *>());
  }

  return Err;
#else
  return make_error<StringError>(
      "SharedMemoryMapper is not supported on this platform yet",
      inconvertibleErrorCode());
#endif
}

```
- **EN**: Implements logic around `Lock`, `erase`, `make_error<StringError>`, `inconvertibleErrorCode`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `Lock`, `erase`, `make_error<StringError>`, `inconvertibleErrorCode` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 298-306
```cpp
Error ExecutorSharedMemoryMapperService::shutdown() {
  if (Reservations.empty())
    return Error::success();

  std::vector<ExecutorAddr> ReservationAddrs;
  ReservationAddrs.reserve(Reservations.size());
  for (const auto &R : Reservations)
    ReservationAddrs.push_back(ExecutorAddr::fromPtr(R.getFirst()));

```
- **EN**: Implements logic around `shutdown`, `success`, `reserve`, `push_back`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `shutdown`, `success`, `reserve`, `push_back` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 307-323
```cpp
  return release(std::move(ReservationAddrs));
}

void ExecutorSharedMemoryMapperService::addBootstrapSymbols(
    StringMap<ExecutorAddr> &M) {
  M[rt::ExecutorSharedMemoryMapperServiceInstanceName] =
      ExecutorAddr::fromPtr(this);
  M[rt::ExecutorSharedMemoryMapperServiceReserveWrapperName] =
      ExecutorAddr::fromPtr(&reserveWrapper);
  M[rt::ExecutorSharedMemoryMapperServiceInitializeWrapperName] =
      ExecutorAddr::fromPtr(&initializeWrapper);
  M[rt::ExecutorSharedMemoryMapperServiceDeinitializeWrapperName] =
      ExecutorAddr::fromPtr(&deinitializeWrapper);
  M[rt::ExecutorSharedMemoryMapperServiceReleaseWrapperName] =
      ExecutorAddr::fromPtr(&releaseWrapper);
}

```
- **EN**: Implements logic around `release`, `addBootstrapSymbols`, `fromPtr`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `release`, `addBootstrapSymbols`, `fromPtr` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 324-334
```cpp
llvm::orc::shared::CWrapperFunctionBuffer
ExecutorSharedMemoryMapperService::reserveWrapper(const char *ArgData,
                                                  size_t ArgSize) {
  return shared::WrapperFunction<
             rt::SPSExecutorSharedMemoryMapperServiceReserveSignature>::
      handle(ArgData, ArgSize,
             shared::makeMethodWrapperHandler(
                 &ExecutorSharedMemoryMapperService::reserve))
          .release();
}

```
- **EN**: Implements logic around `reserveWrapper`, `handle`, `makeMethodWrapperHandler`, `release`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `reserveWrapper`, `handle`, `makeMethodWrapperHandler`, `release` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 335-345
```cpp
llvm::orc::shared::CWrapperFunctionBuffer
ExecutorSharedMemoryMapperService::initializeWrapper(const char *ArgData,
                                                     size_t ArgSize) {
  return shared::WrapperFunction<
             rt::SPSExecutorSharedMemoryMapperServiceInitializeSignature>::
      handle(ArgData, ArgSize,
             shared::makeMethodWrapperHandler(
                 &ExecutorSharedMemoryMapperService::initialize))
          .release();
}

```
- **EN**: Implements logic around `initializeWrapper`, `handle`, `makeMethodWrapperHandler`, `release`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `initializeWrapper`, `handle`, `makeMethodWrapperHandler`, `release` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 346-356
```cpp
llvm::orc::shared::CWrapperFunctionBuffer
ExecutorSharedMemoryMapperService::deinitializeWrapper(const char *ArgData,
                                                       size_t ArgSize) {
  return shared::WrapperFunction<
             rt::SPSExecutorSharedMemoryMapperServiceDeinitializeSignature>::
      handle(ArgData, ArgSize,
             shared::makeMethodWrapperHandler(
                 &ExecutorSharedMemoryMapperService::deinitialize))
          .release();
}

```
- **EN**: Implements logic around `deinitializeWrapper`, `handle`, `makeMethodWrapperHandler`, `release`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `deinitializeWrapper`, `handle`, `makeMethodWrapperHandler`, `release` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 357-367
```cpp
llvm::orc::shared::CWrapperFunctionBuffer
ExecutorSharedMemoryMapperService::releaseWrapper(const char *ArgData,
                                                  size_t ArgSize) {
  return shared::WrapperFunction<
             rt::SPSExecutorSharedMemoryMapperServiceReleaseSignature>::
      handle(ArgData, ArgSize,
             shared::makeMethodWrapperHandler(
                 &ExecutorSharedMemoryMapperService::release))
          .release();
}

```
- **EN**: Implements logic around `releaseWrapper`, `handle`, `makeMethodWrapperHandler`, `release`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `releaseWrapper`, `handle`, `makeMethodWrapperHandler`, `release` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 368-370
```cpp
} // namespace rt_bootstrap
} // end namespace orc
} // end namespace llvm
```
- **EN**: Introduces declarations for `rt_bootstrap`, `orc`, `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `rt_bootstrap`, `orc`, `llvm` 等声明，定义本文件后续使用的数据结构或接口。

## Key Concepts / 关键概念

- **ORC JIT / ORC JIT**:
  - **EN**: Coordinates JITDylibs, symbol materialization, execution sessions, and asynchronous compilation flows
  - **CN**: 协调 JITDylib、符号物化、ExecutionSession 与异步编译流程

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/Orc/TargetProcess/ExecutorSharedMemoryMapperService.h`, `llvm/Config/llvm-config.h`, `llvm/ExecutionEngine/Orc/Shared/OrcRTBridge.h`, `llvm/Support/Process.h`, `llvm/Support/WindowsError.h`, `sstream`, `errno.h`, `fcntl.h`, `sys/mman.h`, `llvm/Support/BLAKE3.h`, `sys/shm.h`, `unistd.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Support
