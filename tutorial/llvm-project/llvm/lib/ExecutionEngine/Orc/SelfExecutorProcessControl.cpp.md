# SelfExecutorProcessControl.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/Orc/SelfExecutorProcessControl.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements ORC JIT infrastructure such as execution sessions, JITDylibs, materialization, symbol lookup, and execution helpers.
  - **CN**: 实现 ORC JIT 基础设施，例如 ExecutionSession、JITDylib、物化流程、符号查找以及执行辅助组件。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===------ SelfExecutorProcessControl.cpp -- EPC for in-process JITs -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-20
```cpp

#include "llvm/ExecutionEngine/Orc/SelfExecutorProcessControl.h"

#include "llvm/ExecutionEngine/JITLink/JITLinkMemoryManager.h"
#include "llvm/ExecutionEngine/Orc/Core.h"
#include "llvm/ExecutionEngine/Orc/DylibManager.h"
#include "llvm/ExecutionEngine/Orc/InProcessMemoryAccess.h"
#include "llvm/ExecutionEngine/Orc/TargetProcess/DefaultHostBootstrapValues.h"
#include "llvm/ExecutionEngine/Orc/TargetProcess/TargetExecutionUtils.h"
#include "llvm/Support/DynamicLibrary.h"
#include "llvm/Support/Process.h"
#include "llvm/TargetParser/Host.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/Orc/SelfExecutorProcessControl.h`, `llvm/ExecutionEngine/JITLink/JITLinkMemoryManager.h`, `llvm/ExecutionEngine/Orc/Core.h`, `llvm/ExecutionEngine/Orc/DylibManager.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/Orc/SelfExecutorProcessControl.h`, `llvm/ExecutionEngine/JITLink/JITLinkMemoryManager.h`, `llvm/ExecutionEngine/Orc/Core.h`, `llvm/ExecutionEngine/Orc/DylibManager.h`。

### Lines 21-32
```cpp
#define DEBUG_TYPE "orc"

namespace llvm::orc {

class SelfExecutorProcessControl::InProcessDylibManager : public DylibManager {
public:
  InProcessDylibManager(char GlobalManglingPrefix);
  Expected<tpctypes::DylibHandle> loadDylib(const char *DylibPath) override;
  void
  lookupSymbolsAsync(tpctypes::DylibHandle H, const SymbolLookupSet &Symbols,
                     DylibManager::SymbolLookupCompleteFn Complete) override;

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 33-41
```cpp
private:
  char GlobalManglingPrefix;
};

SelfExecutorProcessControl::SelfExecutorProcessControl(
    std::shared_ptr<SymbolStringPool> SSP, std::unique_ptr<TaskDispatcher> D,
    Triple TargetTriple, unsigned PageSize)
    : ExecutorProcessControl(std::move(SSP), std::move(D)) {

```
- **EN**: Implements logic around `SelfExecutorProcessControl`, `ExecutorProcessControl`.
- **CN**: 围绕 `SelfExecutorProcessControl`, `ExecutorProcessControl` 实现具体逻辑。

### Lines 42-48
```cpp
  this->TargetTriple = std::move(TargetTriple);
  this->PageSize = PageSize;
  this->JDI = {ExecutorAddr::fromPtr(jitDispatchViaWrapperFunctionManager),
               ExecutorAddr::fromPtr(this)};

  addDefaultBootstrapValuesForHostProcess(BootstrapMap, BootstrapSymbols);

```
- **EN**: Implements logic around `move`, `fromPtr`, `addDefaultBootstrapValuesForHostProcess`.
- **CN**: 围绕 `move`, `fromPtr`, `addDefaultBootstrapValuesForHostProcess` 实现具体逻辑。

### Lines 49-57
```cpp
#ifdef __APPLE__
  // FIXME: Don't add an UnwindInfoManager by default -- it's redundant when
  //        the ORC runtime is loaded. We'll need a way to document this and
  //        allow clients to choose.
  if (UnwindInfoManager::TryEnable())
    UnwindInfoManager::addBootstrapSymbols(this->BootstrapSymbols);
#endif // __APPLE__
}

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 58-64
```cpp
Expected<std::unique_ptr<SelfExecutorProcessControl>>
SelfExecutorProcessControl::Create(std::shared_ptr<SymbolStringPool> SSP,
                                   std::unique_ptr<TaskDispatcher> D) {

  if (!SSP)
    SSP = std::make_shared<SymbolStringPool>();

```
- **EN**: Implements logic around `Create`, `make_shared<SymbolStringPool>`.
- **CN**: 围绕 `Create`, `make_shared<SymbolStringPool>` 实现具体逻辑。

### Lines 65-71
```cpp
  if (!D)
    D = std::make_unique<InPlaceTaskDispatcher>();

  auto PageSize = sys::Process::getPageSize();
  if (!PageSize)
    return PageSize.takeError();

```
- **EN**: Implements logic around `make_unique<InPlaceTaskDispatcher>`, `getPageSize`, `takeError`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `make_unique<InPlaceTaskDispatcher>`, `getPageSize`, `takeError` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 72-84
```cpp
  Triple TT(sys::getProcessTriple());

  return std::make_unique<SelfExecutorProcessControl>(
      std::move(SSP), std::move(D), std::move(TT), *PageSize);
}

Expected<int32_t>
SelfExecutorProcessControl::runAsMain(ExecutorAddr MainFnAddr,
                                      ArrayRef<std::string> Args) {
  using MainTy = int (*)(int, char *[]);
  return orc::runAsMain(MainFnAddr.toPtr<MainTy>(), Args);
}

```
- **EN**: Implements logic around `TT`, `make_unique<SelfExecutorProcessControl>`, `move`, `runAsMain`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `TT`, `make_unique<SelfExecutorProcessControl>`, `move`, `runAsMain`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 85-96
```cpp
Expected<int32_t>
SelfExecutorProcessControl::runAsVoidFunction(ExecutorAddr VoidFnAddr) {
  using VoidTy = int (*)();
  return orc::runAsVoidFunction(VoidFnAddr.toPtr<VoidTy>());
}

Expected<int32_t>
SelfExecutorProcessControl::runAsIntFunction(ExecutorAddr IntFnAddr, int Arg) {
  using IntTy = int (*)(int);
  return orc::runAsIntFunction(IntFnAddr.toPtr<IntTy>(), Arg);
}

```
- **EN**: Implements logic around `runAsVoidFunction`, `int`, `runAsIntFunction`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `runAsVoidFunction`, `int`, `runAsIntFunction` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 97-106
```cpp
void SelfExecutorProcessControl::callWrapperAsync(ExecutorAddr WrapperFnAddr,
                                                  IncomingWFRHandler SendResult,
                                                  ArrayRef<char> ArgBuffer) {
  using WrapperFnTy =
      shared::CWrapperFunctionBuffer (*)(const char *Data, size_t Size);
  auto *WrapperFn = WrapperFnAddr.toPtr<WrapperFnTy>();
  SendResult(shared::WrapperFunctionBuffer(
      WrapperFn(ArgBuffer.data(), ArgBuffer.size())));
}

```
- **EN**: Implements logic around `callWrapperAsync`, `CWrapperFunctionBuffer`, `toPtr<WrapperFnTy>`, `SendResult`, and 1 more symbols.
- **CN**: 围绕 `callWrapperAsync`, `CWrapperFunctionBuffer`, `toPtr<WrapperFnTy>`, `SendResult`, and 1 more symbols 实现具体逻辑。

### Lines 107-117
```cpp
Error SelfExecutorProcessControl::disconnect() {
  D->shutdown();
  return Error::success();
}

Expected<std::unique_ptr<jitlink::JITLinkMemoryManager>>
SelfExecutorProcessControl::createDefaultMemoryManager() {
  return std::make_unique<jitlink::InProcessMemoryManager>(
      sys::Process::getPageSizeEstimate());
}

```
- **EN**: Implements logic around `disconnect`, `shutdown`, `success`, `createDefaultMemoryManager`, and 2 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `disconnect`, `shutdown`, `success`, `createDefaultMemoryManager`, and 2 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 118-128
```cpp
Expected<std::unique_ptr<DylibManager>>
SelfExecutorProcessControl::createDefaultDylibMgr() {
  char Prefix = TargetTriple.isOSBinFormatMachO() ? '_' : '\0';
  return std::make_unique<InProcessDylibManager>(Prefix);
}

Expected<std::unique_ptr<MemoryAccess>>
SelfExecutorProcessControl::createDefaultMemoryAccess() {
  return std::make_unique<InProcessMemoryAccess>(TargetTriple.isArch64Bit());
}

```
- **EN**: Implements logic around `createDefaultDylibMgr`, `isOSBinFormatMachO`, `make_unique<InProcessDylibManager>`, `createDefaultMemoryAccess`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `createDefaultDylibMgr`, `isOSBinFormatMachO`, `make_unique<InProcessDylibManager>`, `createDefaultMemoryAccess`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 129-137
```cpp
shared::CWrapperFunctionBuffer
SelfExecutorProcessControl::jitDispatchViaWrapperFunctionManager(
    void *Ctx, const void *FnTag, const char *Data, size_t Size) {

  LLVM_DEBUG({
    dbgs() << "jit-dispatch call with tag " << FnTag << " and " << Size
           << " byte payload.\n";
  });

```
- **EN**: Implements logic around `jitDispatchViaWrapperFunctionManager`, `dbgs`.
- **CN**: 围绕 `jitDispatchViaWrapperFunctionManager`, `dbgs` 实现具体逻辑。

### Lines 138-149
```cpp
  std::promise<shared::WrapperFunctionBuffer> ResultP;
  auto ResultF = ResultP.get_future();
  static_cast<SelfExecutorProcessControl *>(Ctx)
      ->getExecutionSession()
      .runJITDispatchHandler(
          [ResultP = std::move(ResultP)](
              shared::WrapperFunctionBuffer Result) mutable {
            ResultP.set_value(std::move(Result));
          },
          ExecutorAddr::fromPtr(FnTag),
          shared::WrapperFunctionBuffer::copyFrom(Data, Size));

```
- **EN**: Implements logic around `get_future`, `getExecutionSession`, `runJITDispatchHandler`, `move`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `get_future`, `getExecutionSession`, `runJITDispatchHandler`, `move`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 150-156
```cpp
  return ResultF.get().release();
}

SelfExecutorProcessControl::InProcessDylibManager::InProcessDylibManager(
    char GlobalManglingPrefix)
    : GlobalManglingPrefix(GlobalManglingPrefix) {}

```
- **EN**: Implements logic around `get`, `InProcessDylibManager`, `GlobalManglingPrefix`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `get`, `InProcessDylibManager`, `GlobalManglingPrefix` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 157-166
```cpp
Expected<tpctypes::DylibHandle>
SelfExecutorProcessControl::InProcessDylibManager::loadDylib(
    const char *DylibPath) {
  std::string ErrMsg;
  auto Dylib = sys::DynamicLibrary::getPermanentLibrary(DylibPath, &ErrMsg);
  if (!Dylib.isValid())
    return make_error<StringError>(std::move(ErrMsg), inconvertibleErrorCode());
  return ExecutorAddr::fromPtr(Dylib.getOSSpecificHandle());
}

```
- **EN**: Implements logic around `loadDylib`, `getPermanentLibrary`, `make_error<StringError>`, `fromPtr`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `loadDylib`, `getPermanentLibrary`, `make_error<StringError>`, `fromPtr` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 167-180
```cpp
void SelfExecutorProcessControl::InProcessDylibManager::lookupSymbolsAsync(
    tpctypes::DylibHandle H, const SymbolLookupSet &Symbols,
    DylibManager::SymbolLookupCompleteFn Complete) {
  tpctypes::LookupResult R;

  sys::DynamicLibrary Dylib(H.toPtr<void *>());
  for (auto &KV : Symbols) {
    auto &Sym = KV.first;
    std::string Tmp((*Sym).data() + !!GlobalManglingPrefix,
                    (*Sym).size() - !!GlobalManglingPrefix);
    void *Addr = Dylib.getAddressOfSymbol(Tmp.c_str());
    if (!Addr && KV.second == SymbolLookupFlags::RequiredSymbol)
      R.emplace_back();
    else
```
- **EN**: Implements logic around `lookupSymbolsAsync`, `Dylib`, `Tmp`, `size`, and 2 more symbols.
- **CN**: 围绕 `lookupSymbolsAsync`, `Dylib`, `Tmp`, `size`, and 2 more symbols 实现具体逻辑。

### Lines 181-187
```cpp
      R.emplace_back(ExecutorSymbolDef(ExecutorAddr::fromPtr(Addr),
                                       JITSymbolFlags::Exported));
  }
  Complete(std::move(R));
}

} // namespace llvm::orc
```
- **EN**: Introduces declarations for `llvm::orc`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm::orc` 等声明，定义本文件后续使用的数据结构或接口。

## Key Concepts / 关键概念

- **ORC JIT / ORC JIT**:
  - **EN**: Coordinates JITDylibs, symbol materialization, execution sessions, and asynchronous compilation flows
  - **CN**: 协调 JITDylib、符号物化、ExecutionSession 与异步编译流程
- **Symbol materialization / 符号物化**:
  - **EN**: Defers code or data generation until symbols are requested
  - **CN**: 将代码或数据的生成延迟到符号真正被请求时
- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/Orc/SelfExecutorProcessControl.h`, `llvm/ExecutionEngine/JITLink/JITLinkMemoryManager.h`, `llvm/ExecutionEngine/Orc/Core.h`, `llvm/ExecutionEngine/Orc/DylibManager.h`, `llvm/ExecutionEngine/Orc/InProcessMemoryAccess.h`, `llvm/ExecutionEngine/Orc/TargetProcess/DefaultHostBootstrapValues.h`, `llvm/ExecutionEngine/Orc/TargetProcess/TargetExecutionUtils.h`, `llvm/Support/DynamicLibrary.h`, `llvm/Support/Process.h`, `llvm/TargetParser/Host.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Support, Target/TargetParser
