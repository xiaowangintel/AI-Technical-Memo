# EPCGenericRTDyldMemoryManager.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/Orc/EPCGenericRTDyldMemoryManager.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements EPC-bbasde MemMgr.
  - **CN**: 实现 ORC JIT 基础设施，例如 ExecutionSession、JITDylib、物化流程、符号查找以及执行辅助组件。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```cpp
//===----- EPCGenericRTDyldMemoryManager.cpp - EPC-bbasde MemMgr -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/ExecutionEngine/Orc/EPCGenericRTDyldMemoryManager.h"
#include "llvm/ExecutionEngine/Orc/Shared/OrcRTBridge.h"
#include "llvm/Support/Alignment.h"
#include "llvm/Support/FormatVariadic.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/Orc/EPCGenericRTDyldMemoryManager.h`, `llvm/ExecutionEngine/Orc/Shared/OrcRTBridge.h`, `llvm/Support/Alignment.h`, `llvm/Support/FormatVariadic.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/Orc/EPCGenericRTDyldMemoryManager.h`, `llvm/ExecutionEngine/Orc/Shared/OrcRTBridge.h`, `llvm/Support/Alignment.h`, `llvm/Support/FormatVariadic.h`。

### Lines 14-31
```cpp
#define DEBUG_TYPE "orc"

using namespace llvm::orc::shared;

namespace llvm {
namespace orc {

Expected<std::unique_ptr<EPCGenericRTDyldMemoryManager>>
EPCGenericRTDyldMemoryManager::CreateWithDefaultBootstrapSymbols(
    ExecutorProcessControl &EPC) {
  SymbolAddrs SAs;
  if (auto Err = EPC.getBootstrapSymbols(
          {{SAs.Instance, rt::SimpleExecutorMemoryManagerInstanceName},
           {SAs.Reserve, rt::SimpleExecutorMemoryManagerReserveWrapperName},
           {SAs.Initialize,
            rt::SimpleExecutorMemoryManagerInitializeWrapperName},
           {SAs.Release, rt::SimpleExecutorMemoryManagerReleaseWrapperName},
           {SAs.RegisterEHFrame, rt::RegisterEHFrameSectionAllocActionName},
```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 32-43
```cpp
           {SAs.DeregisterEHFrame,
            rt::DeregisterEHFrameSectionAllocActionName}}))
    return std::move(Err);
  return std::make_unique<EPCGenericRTDyldMemoryManager>(EPC, std::move(SAs));
}

EPCGenericRTDyldMemoryManager::EPCGenericRTDyldMemoryManager(
    ExecutorProcessControl &EPC, SymbolAddrs SAs)
    : EPC(EPC), SAs(std::move(SAs)) {
  LLVM_DEBUG(dbgs() << "Created remote allocator " << (void *)this << "\n");
}

```
- **EN**: Implements logic around `move`, `make_unique<EPCGenericRTDyldMemoryManager>`, `EPCGenericRTDyldMemoryManager`, `EPC`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `move`, `make_unique<EPCGenericRTDyldMemoryManager>`, `EPCGenericRTDyldMemoryManager`, `EPC` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 44-57
```cpp
EPCGenericRTDyldMemoryManager::~EPCGenericRTDyldMemoryManager() {
  LLVM_DEBUG(dbgs() << "Destroyed remote allocator " << (void *)this << "\n");
  if (!ErrMsg.empty())
    errs() << "Destroying with existing errors:\n" << ErrMsg << "\n";

  Error Err = Error::success();
  if (auto Err2 = EPC.callSPSWrapper<
                  rt::SPSSimpleExecutorMemoryManagerReleaseSignature>(
          SAs.Reserve, Err, SAs.Instance, FinalizedAllocs)) {
    // FIXME: Report errors through EPC once that functionality is available.
    logAllUnhandledErrors(std::move(Err2), errs(), "");
    return;
  }

```
- **EN**: Implements logic around `~EPCGenericRTDyldMemoryManager`, `errs`, `success`, `SPSSimpleExecutorMemoryManagerReleaseSignature>`, and 1 more symbols.
- **CN**: 围绕 `~EPCGenericRTDyldMemoryManager`, `errs`, `success`, `SPSSimpleExecutorMemoryManagerReleaseSignature>`, and 1 more symbols 实现具体逻辑。

### Lines 58-75
```cpp
  if (Err)
    logAllUnhandledErrors(std::move(Err), errs(), "");
}

uint8_t *EPCGenericRTDyldMemoryManager::allocateCodeSection(
    uintptr_t Size, unsigned Alignment, unsigned SectionID,
    StringRef SectionName) {
  std::lock_guard<std::mutex> Lock(M);
  LLVM_DEBUG({
    dbgs() << "Allocator " << (void *)this << " allocating code section "
           << SectionName << ": size = " << formatv("{0:x}", Size)
           << " bytes, alignment = " << Alignment << "\n";
  });
  auto &Seg = Unmapped.back().CodeAllocs;
  Seg.emplace_back(Size, Alignment);
  return reinterpret_cast<uint8_t *>(
      alignAddr(Seg.back().Contents.get(), Align(Alignment)));
}
```
- **EN**: Implements logic around `logAllUnhandledErrors`, `allocateCodeSection`, `Lock`, `dbgs`, and 4 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `logAllUnhandledErrors`, `allocateCodeSection`, `Lock`, `dbgs`, and 4 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 76-87
```cpp

uint8_t *EPCGenericRTDyldMemoryManager::allocateDataSection(
    uintptr_t Size, unsigned Alignment, unsigned SectionID,
    StringRef SectionName, bool IsReadOnly) {
  std::lock_guard<std::mutex> Lock(M);
  LLVM_DEBUG({
    dbgs() << "Allocator " << (void *)this << " allocating "
           << (IsReadOnly ? "ro" : "rw") << "-data section " << SectionName
           << ": size = " << formatv("{0:x}", Size) << " bytes, alignment "
           << Alignment << ")\n";
  });

```
- **EN**: Implements logic around `allocateDataSection`, `Lock`, `dbgs`, `formatv`.
- **CN**: 围绕 `allocateDataSection`, `Lock`, `dbgs`, `formatv` 实现具体逻辑。

### Lines 88-99
```cpp
  auto &Seg =
      IsReadOnly ? Unmapped.back().RODataAllocs : Unmapped.back().RWDataAllocs;

  Seg.emplace_back(Size, Alignment);
  return reinterpret_cast<uint8_t *>(
      alignAddr(Seg.back().Contents.get(), Align(Alignment)));
}

void EPCGenericRTDyldMemoryManager::reserveAllocationSpace(
    uintptr_t CodeSize, Align CodeAlign, uintptr_t RODataSize,
    Align RODataAlign, uintptr_t RWDataSize, Align RWDataAlign) {

```
- **EN**: Implements logic around `back`, `emplace_back`, `alignAddr`, `reserveAllocationSpace`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `back`, `emplace_back`, `alignAddr`, `reserveAllocationSpace` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 100-117
```cpp
  {
    std::lock_guard<std::mutex> Lock(M);
    // If there's already an error then bail out.
    if (!ErrMsg.empty())
      return;

    if (CodeAlign > EPC.getPageSize()) {
      ErrMsg = "Invalid code alignment in reserveAllocationSpace";
      return;
    }
    if (RODataAlign > EPC.getPageSize()) {
      ErrMsg = "Invalid ro-data alignment in reserveAllocationSpace";
      return;
    }
    if (RWDataAlign > EPC.getPageSize()) {
      ErrMsg = "Invalid rw-data alignment in reserveAllocationSpace";
      return;
    }
```
- **EN**: Implements logic around `Lock`.
- **CN**: 围绕 `Lock` 实现具体逻辑。

### Lines 118-129
```cpp
  }

  uint64_t TotalSize = 0;
  TotalSize += alignTo(CodeSize, EPC.getPageSize());
  TotalSize += alignTo(RODataSize, EPC.getPageSize());
  TotalSize += alignTo(RWDataSize, EPC.getPageSize());

  LLVM_DEBUG({
    dbgs() << "Allocator " << (void *)this << " reserving "
           << formatv("{0:x}", TotalSize) << " bytes.\n";
  });

```
- **EN**: Implements logic around `alignTo`, `dbgs`, `formatv`.
- **CN**: 围绕 `alignTo`, `dbgs`, `formatv` 实现具体逻辑。

### Lines 130-143
```cpp
  Expected<ExecutorAddr> TargetAllocAddr((ExecutorAddr()));
  if (auto Err = EPC.callSPSWrapper<
                 rt::SPSSimpleExecutorMemoryManagerReserveSignature>(
          SAs.Reserve, TargetAllocAddr, SAs.Instance, TotalSize)) {
    std::lock_guard<std::mutex> Lock(M);
    ErrMsg = toString(std::move(Err));
    return;
  }
  if (!TargetAllocAddr) {
    std::lock_guard<std::mutex> Lock(M);
    ErrMsg = toString(TargetAllocAddr.takeError());
    return;
  }

```
- **EN**: Implements logic around `TargetAllocAddr`, `SPSSimpleExecutorMemoryManagerReserveSignature>`, `Lock`, `toString`.
- **CN**: 围绕 `TargetAllocAddr`, `SPSSimpleExecutorMemoryManagerReserveSignature>`, `Lock`, `toString` 实现具体逻辑。

### Lines 144-155
```cpp
  std::lock_guard<std::mutex> Lock(M);
  Unmapped.push_back(SectionAllocGroup());
  Unmapped.back().RemoteCode = {
      *TargetAllocAddr, ExecutorAddrDiff(alignTo(CodeSize, EPC.getPageSize()))};
  Unmapped.back().RemoteROData = {
      Unmapped.back().RemoteCode.End,
      ExecutorAddrDiff(alignTo(RODataSize, EPC.getPageSize()))};
  Unmapped.back().RemoteRWData = {
      Unmapped.back().RemoteROData.End,
      ExecutorAddrDiff(alignTo(RWDataSize, EPC.getPageSize()))};
}

```
- **EN**: Implements logic around `Lock`, `push_back`, `back`, `ExecutorAddrDiff`.
- **CN**: 围绕 `Lock`, `push_back`, `back`, `ExecutorAddrDiff` 实现具体逻辑。

### Lines 156-171
```cpp
bool EPCGenericRTDyldMemoryManager::needsToReserveAllocationSpace() {
  return true;
}

void EPCGenericRTDyldMemoryManager::registerEHFrames(uint8_t *Addr,
                                                     uint64_t LoadAddr,
                                                     size_t Size) {
  LLVM_DEBUG({
    dbgs() << "Allocator " << (void *)this << " added unfinalized eh-frame "
           << formatv("[ {0:x} {1:x} ]", LoadAddr, LoadAddr + Size) << "\n";
  });
  std::lock_guard<std::mutex> Lock(M);
  // Bail out early if there's already an error.
  if (!ErrMsg.empty())
    return;

```
- **EN**: Implements logic around `needsToReserveAllocationSpace`, `registerEHFrames`, `dbgs`, `formatv`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `needsToReserveAllocationSpace`, `registerEHFrames`, `dbgs`, `formatv`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 172-183
```cpp
  ExecutorAddr LA(LoadAddr);
  for (auto &SecAllocGroup : llvm::reverse(Unfinalized)) {
    if (SecAllocGroup.RemoteCode.contains(LA) ||
        SecAllocGroup.RemoteROData.contains(LA) ||
        SecAllocGroup.RemoteRWData.contains(LA)) {
      SecAllocGroup.UnfinalizedEHFrames.push_back({LA, Size});
      return;
    }
  }
  ErrMsg = "eh-frame does not lie inside unfinalized alloc";
}

```
- **EN**: Implements logic around `LA`, `contains`, `push_back`.
- **CN**: 围绕 `LA`, `contains`, `push_back` 实现具体逻辑。

### Lines 184-201
```cpp
void EPCGenericRTDyldMemoryManager::deregisterEHFrames() {
  // This is a no-op for us: We've registered a deallocation action for it.
}

void EPCGenericRTDyldMemoryManager::notifyObjectLoaded(
    RuntimeDyld &Dyld, const object::ObjectFile &Obj) {
  std::lock_guard<std::mutex> Lock(M);
  LLVM_DEBUG(dbgs() << "Allocator " << (void *)this << " applied mappings:\n");
  for (auto &ObjAllocs : Unmapped) {
    mapAllocsToRemoteAddrs(Dyld, ObjAllocs.CodeAllocs,
                           ObjAllocs.RemoteCode.Start);
    mapAllocsToRemoteAddrs(Dyld, ObjAllocs.RODataAllocs,
                           ObjAllocs.RemoteROData.Start);
    mapAllocsToRemoteAddrs(Dyld, ObjAllocs.RWDataAllocs,
                           ObjAllocs.RemoteRWData.Start);
    Unfinalized.push_back(std::move(ObjAllocs));
  }
  Unmapped.clear();
```
- **EN**: Implements logic around `deregisterEHFrames`, `notifyObjectLoaded`, `Lock`, `mapAllocsToRemoteAddrs`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `deregisterEHFrames`, `notifyObjectLoaded`, `Lock`, `mapAllocsToRemoteAddrs`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 202-217
```cpp
}

bool EPCGenericRTDyldMemoryManager::finalizeMemory(std::string *ErrMsg) {
  LLVM_DEBUG(dbgs() << "Allocator " << (void *)this << " finalizing:\n");

  // If there's an error then bail out here.
  std::vector<SectionAllocGroup> SecAllocGroups;
  {
    std::lock_guard<std::mutex> Lock(M);
    if (ErrMsg && !this->ErrMsg.empty()) {
      *ErrMsg = std::move(this->ErrMsg);
      return true;
    }
    std::swap(SecAllocGroups, Unfinalized);
  }

```
- **EN**: Implements logic around `finalizeMemory`, `Lock`, `move`, `swap`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `finalizeMemory`, `Lock`, `move`, `swap` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 218-227
```cpp
  // Loop over unfinalized objects to make finalization requests.
  for (auto &SecAllocGroup : SecAllocGroups) {

    MemProt SegMemProts[3] = {MemProt::Read | MemProt::Exec, MemProt::Read,
                              MemProt::Read | MemProt::Write};

    ExecutorAddrRange *RemoteAddrs[3] = {&SecAllocGroup.RemoteCode,
                                         &SecAllocGroup.RemoteROData,
                                         &SecAllocGroup.RemoteRWData};

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 228-245
```cpp
    std::vector<SectionAlloc> *SegSections[3] = {&SecAllocGroup.CodeAllocs,
                                                 &SecAllocGroup.RODataAllocs,
                                                 &SecAllocGroup.RWDataAllocs};

    tpctypes::FinalizeRequest FR;
    std::unique_ptr<char[]> AggregateContents[3];

    for (unsigned I = 0; I != 3; ++I) {
      FR.Segments.push_back({});
      auto &Seg = FR.Segments.back();
      Seg.RAG = SegMemProts[I];
      Seg.Addr = RemoteAddrs[I]->Start;
      for (auto &SecAlloc : *SegSections[I]) {
        Seg.Size = alignTo(Seg.Size, SecAlloc.Align);
        Seg.Size += SecAlloc.Size;
      }
      AggregateContents[I] = std::make_unique<char[]>(Seg.Size);
      size_t SecOffset = 0;
```
- **EN**: Implements logic around `push_back`, `back`, `alignTo`.
- **CN**: 围绕 `push_back`, `back`, `alignTo` 实现具体逻辑。

### Lines 246-258
```cpp
      for (auto &SecAlloc : *SegSections[I]) {
        SecOffset = alignTo(SecOffset, SecAlloc.Align);
        memcpy(&AggregateContents[I][SecOffset],
               reinterpret_cast<const char *>(
                   alignAddr(SecAlloc.Contents.get(), Align(SecAlloc.Align))),
               SecAlloc.Size);
        SecOffset += SecAlloc.Size;
        // FIXME: Can we reset SecAlloc.Content here, now that it's copied into
        // the aggregated content?
      }
      Seg.Content = {AggregateContents[I].get(), SecOffset};
    }

```
- **EN**: Implements logic around `alignTo`, `memcpy`, `alignAddr`, `get`.
- **CN**: 围绕 `alignTo`, `memcpy`, `alignAddr`, `get` 实现具体逻辑。

### Lines 259-267
```cpp
    for (auto &Frame : SecAllocGroup.UnfinalizedEHFrames)
      FR.Actions.push_back(
          {cantFail(
               WrapperFunctionCall::Create<SPSArgList<SPSExecutorAddrRange>>(
                   SAs.RegisterEHFrame, Frame)),
           cantFail(
               WrapperFunctionCall::Create<SPSArgList<SPSExecutorAddrRange>>(
                   SAs.DeregisterEHFrame, Frame))});

```
- **EN**: Implements logic around `push_back`, `cantFail`, `Create<SPSArgList<SPSExecutorAddrRange>>`.
- **CN**: 围绕 `push_back`, `cantFail`, `Create<SPSArgList<SPSExecutorAddrRange>>` 实现具体逻辑。

### Lines 268-285
```cpp
    // We'll also need to make an extra allocation for the eh-frame wrapper call
    // arguments.
    Expected<ExecutorAddr> InitializeKey((ExecutorAddr()));
    if (auto Err = EPC.callSPSWrapper<
                   rt::SPSSimpleExecutorMemoryManagerInitializeSignature>(
            SAs.Initialize, InitializeKey, SAs.Instance, std::move(FR))) {
      std::lock_guard<std::mutex> Lock(M);
      this->ErrMsg = toString(std::move(Err));
      dbgs() << "Serialization error: " << this->ErrMsg << "\n";
      if (ErrMsg)
        *ErrMsg = this->ErrMsg;
      return true;
    }
    if (!InitializeKey) {
      std::lock_guard<std::mutex> Lock(M);
      this->ErrMsg = toString(InitializeKey.takeError());
      dbgs() << "Finalization error: " << this->ErrMsg << "\n";
      if (ErrMsg)
```
- **EN**: Implements logic around `InitializeKey`, `SPSSimpleExecutorMemoryManagerInitializeSignature>`, `move`, `Lock`, and 2 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `InitializeKey`, `SPSSimpleExecutorMemoryManagerInitializeSignature>`, `move`, `Lock`, and 2 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 286-303
```cpp
        *ErrMsg = this->ErrMsg;
      return true;
    }
  }

  return false;
}

void EPCGenericRTDyldMemoryManager::mapAllocsToRemoteAddrs(
    RuntimeDyld &Dyld, std::vector<SectionAlloc> &Allocs,
    ExecutorAddr NextAddr) {
  for (auto &Alloc : Allocs) {
    NextAddr.setValue(alignTo(NextAddr.getValue(), Alloc.Align));
    LLVM_DEBUG({
      dbgs() << "     " << static_cast<void *>(Alloc.Contents.get()) << " -> "
             << format("0x%016" PRIx64, NextAddr.getValue()) << "\n";
    });
    Dyld.mapSectionAddress(reinterpret_cast<const void *>(alignAddr(
```
- **EN**: Implements logic around `mapAllocsToRemoteAddrs`, `setValue`, `dbgs`, `format`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `mapAllocsToRemoteAddrs`, `setValue`, `dbgs`, `format`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 304-313
```cpp
                               Alloc.Contents.get(), Align(Alloc.Align))),
                           NextAddr.getValue());
    Alloc.RemoteAddr = NextAddr;
    // Only advance NextAddr if it was non-null to begin with,
    // otherwise leave it as null.
    if (NextAddr)
      NextAddr += ExecutorAddrDiff(Alloc.Size);
  }
}

```
- **EN**: Implements logic around `get`, `getValue`, `ExecutorAddrDiff`.
- **CN**: 围绕 `get`, `getValue`, `ExecutorAddrDiff` 实现具体逻辑。

### Lines 314-315
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
- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/Orc/EPCGenericRTDyldMemoryManager.h`, `llvm/ExecutionEngine/Orc/Shared/OrcRTBridge.h`, `llvm/Support/Alignment.h`, `llvm/Support/FormatVariadic.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Support
