# VTuneSupportPlugin.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/Orc/Debugging/VTuneSupportPlugin.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements ORC JIT infrastructure such as execution sessions, JITDylibs, materialization, symbol lookup, and execution helpers.
  - **CN**: 实现 ORC JIT 基础设施，例如 ExecutionSession、JITDylib、物化流程、符号查找以及执行辅助组件。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===--- VTuneSupportPlugin.cpp -- Support for VTune profiler --*- C++ -*--===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-15
```cpp
//
// Handles support for registering code with VIntel Tune's Amplfiier JIT API.
//
//===----------------------------------------------------------------------===//
#include "llvm/ExecutionEngine/Orc/Debugging/VTuneSupportPlugin.h"
#include "llvm/DebugInfo/DWARF/DWARFContext.h"
#include "llvm/ExecutionEngine/Orc/Debugging/DebugInfoSupport.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/Orc/Debugging/VTuneSupportPlugin.h`, `llvm/DebugInfo/DWARF/DWARFContext.h`, `llvm/ExecutionEngine/Orc/Debugging/DebugInfoSupport.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/Orc/Debugging/VTuneSupportPlugin.h`, `llvm/DebugInfo/DWARF/DWARFContext.h`, `llvm/ExecutionEngine/Orc/Debugging/DebugInfoSupport.h`。

### Lines 16-25
```cpp
using namespace llvm;
using namespace llvm::orc;
using namespace llvm::jitlink;

static constexpr StringRef RegisterVTuneImplName = "llvm_orc_registerVTuneImpl";
static constexpr StringRef UnregisterVTuneImplName =
    "llvm_orc_unregisterVTuneImpl";
static constexpr StringRef RegisterTestVTuneImplName =
    "llvm_orc_test_registerVTuneImpl";

```
- **EN**: Introduces declarations for `llvm`, `llvm::orc`, `llvm::jitlink`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `llvm::orc`, `llvm::jitlink` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 26-39
```cpp
static VTuneMethodBatch getMethodBatch(LinkGraph &G, bool EmitDebugInfo) {
  VTuneMethodBatch Batch;
  std::unique_ptr<DWARFContext> DC;
  StringMap<std::unique_ptr<MemoryBuffer>> DCBacking;
  if (EmitDebugInfo) {
    auto EDC = createDWARFContext(G);
    if (!EDC) {
      EmitDebugInfo = false;
    } else {
      DC = std::move(EDC->first);
      DCBacking = std::move(EDC->second);
    }
  }

```
- **EN**: Implements logic around `getMethodBatch`, `createDWARFContext`, `move`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `getMethodBatch`, `createDWARFContext`, `move` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 40-52
```cpp
  auto GetStringIdx = [Deduplicator = StringMap<uint32_t>(),
                       &Batch](StringRef S) mutable {
    auto [I, Inserted] = Deduplicator.try_emplace(S);
    if (Inserted) {
      Batch.Strings.push_back(S.str());
      I->second = Batch.Strings.size();
    }
    return I->second;
  };
  for (auto Sym : G.defined_symbols()) {
    if (!Sym->isCallable())
      continue;

```
- **EN**: Implements logic around `StringMap<uint32_t>`, `try_emplace`, `push_back`, `size`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `StringMap<uint32_t>`, `try_emplace`, `push_back`, `size` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 53-62
```cpp
    Batch.Methods.push_back(VTuneMethodInfo());
    auto &Method = Batch.Methods.back();
    Method.MethodID = 0;
    Method.ParentMI = 0;
    Method.LoadAddr = Sym->getAddress();
    Method.LoadSize = Sym->getSize();
    Method.NameSI = GetStringIdx(*Sym->getName());
    Method.ClassFileSI = 0;
    Method.SourceFileSI = 0;

```
- **EN**: Implements logic around `push_back`, `back`, `getAddress`, `getSize`, and 1 more symbols.
- **CN**: 围绕 `push_back`, `back`, `getAddress`, `getSize`, and 1 more symbols 实现具体逻辑。

### Lines 63-76
```cpp
    if (!EmitDebugInfo)
      continue;

    auto &Section = Sym->getSection();
    auto Addr = Sym->getAddress();
    auto SAddr =
        object::SectionedAddress{Addr.getValue(), Section.getOrdinal()};
    DILineInfoTable LinesInfo = DC->getLineInfoForAddressRange(
        SAddr, Sym->getSize(),
        DILineInfoSpecifier::FileLineInfoKind::AbsoluteFilePath);
    Method.SourceFileSI = Batch.Strings.size();
    Batch.Strings.push_back(
        DC->getLineInfoForAddress(SAddr).value_or(DILineInfo()).FileName);
    for (auto &LInfo : LinesInfo) {
```
- **EN**: Implements logic around `getSection`, `getAddress`, `getValue`, `getLineInfoForAddressRange`, and 4 more symbols.
- **CN**: 围绕 `getSection`, `getAddress`, `getValue`, `getLineInfoForAddressRange`, and 4 more symbols 实现具体逻辑。

### Lines 77-84
```cpp
      Method.LineTable.push_back(
          std::pair<unsigned, unsigned>{/*unsigned*/ Sym->getOffset(),
                                        /*DILineInfo*/ LInfo.second.Line});
    }
  }
  return Batch;
}

```
- **EN**: Implements logic around `push_back`, `getOffset`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `push_back`, `getOffset` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 85-98
```cpp
void VTuneSupportPlugin::modifyPassConfig(MaterializationResponsibility &MR,
                                          LinkGraph &G,
                                          PassConfiguration &Config) {
  Config.PostFixupPasses.push_back([this, MR = &MR](LinkGraph &G) {
    // the object file is generated but not linked yet
    auto Batch = getMethodBatch(G, EmitDebugInfo);
    if (Batch.Methods.empty()) {
      return Error::success();
    }
    {
      std::lock_guard<std::mutex> Lock(PluginMutex);
      uint64_t Allocated = Batch.Methods.size();
      uint64_t Start = NextMethodID;
      NextMethodID += Allocated;
```
- **EN**: Implements logic around `modifyPassConfig`, `push_back`, `getMethodBatch`, `success`, and 2 more symbols; this block coordinates ORC symbol lookup or materialization state; manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `modifyPassConfig`, `push_back`, `getMethodBatch`, `success`, and 2 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 99-112
```cpp
      for (size_t i = Start; i < NextMethodID; ++i) {
        Batch.Methods[i - Start].MethodID = i;
      }
      this->PendingMethodIDs[MR] = {Start, Allocated};
    }
    G.allocActions().push_back(
        {cantFail(shared::WrapperFunctionCall::Create<
                  shared::SPSArgList<shared::SPSVTuneMethodBatch>>(
             RegisterVTuneImplAddr, Batch)),
         {}});
    return Error::success();
  });
}

```
- **EN**: Implements logic around `allocActions`, `cantFail`, `SPSVTuneMethodBatch>>`, `success`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `allocActions`, `cantFail`, `SPSVTuneMethodBatch>>`, `success` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 113-119
```cpp
Error VTuneSupportPlugin::notifyEmitted(MaterializationResponsibility &MR) {
  if (auto Err = MR.withResourceKeyDo([this, MR = &MR](ResourceKey K) {
        std::lock_guard<std::mutex> Lock(PluginMutex);
        auto I = PendingMethodIDs.find(MR);
        if (I == PendingMethodIDs.end())
          return;

```
- **EN**: Implements logic around `notifyEmitted`, `Lock`, `find`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `notifyEmitted`, `Lock`, `find` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 120-127
```cpp
        LoadedMethodIDs[K].push_back(I->second);
        PendingMethodIDs.erase(I);
      })) {
    return Err;
  }
  return Error::success();
}

```
- **EN**: Implements logic around `push_back`, `erase`, `success`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `push_back`, `erase`, `success` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 128-141
```cpp
Error VTuneSupportPlugin::notifyFailed(MaterializationResponsibility &MR) {
  std::lock_guard<std::mutex> Lock(PluginMutex);
  PendingMethodIDs.erase(&MR);
  return Error::success();
}

Error VTuneSupportPlugin::notifyRemovingResources(JITDylib &JD, ResourceKey K) {
  // Unregistration not required if not provided
  if (!UnregisterVTuneImplAddr) {
    return Error::success();
  }
  VTuneUnloadedMethodIDs UnloadedIDs;
  {
    std::lock_guard<std::mutex> Lock(PluginMutex);
```
- **EN**: Implements logic around `notifyFailed`, `Lock`, `erase`, `success`, and 1 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `notifyFailed`, `Lock`, `erase`, `success`, and 1 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 142-152
```cpp
    auto I = LoadedMethodIDs.find(K);
    if (I == LoadedMethodIDs.end())
      return Error::success();

    UnloadedIDs = std::move(I->second);
    LoadedMethodIDs.erase(I);
  }
  if (auto Err = EPC.callSPSWrapper<void(shared::SPSVTuneUnloadedMethodIDs)>(
          UnregisterVTuneImplAddr, UnloadedIDs))
    return Err;

```
- **EN**: Implements logic around `find`, `success`, `move`, `erase`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `find`, `success`, `move`, `erase` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 153-163
```cpp
  return Error::success();
}

void VTuneSupportPlugin::notifyTransferringResources(JITDylib &JD,
                                                     ResourceKey DstKey,
                                                     ResourceKey SrcKey) {
  std::lock_guard<std::mutex> Lock(PluginMutex);
  auto I = LoadedMethodIDs.find(SrcKey);
  if (I == LoadedMethodIDs.end())
    return;

```
- **EN**: Implements logic around `success`, `notifyTransferringResources`, `Lock`, `find`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `notifyTransferringResources`, `Lock`, `find` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 164-177
```cpp
  auto &Dest = LoadedMethodIDs[DstKey];
  llvm::append_range(Dest, I->second);
  LoadedMethodIDs.erase(SrcKey);
}

Expected<std::unique_ptr<VTuneSupportPlugin>>
VTuneSupportPlugin::Create(ExecutorProcessControl &EPC, JITDylib &JD,
                           bool EmitDebugInfo, bool TestMode) {
  auto &ES = EPC.getExecutionSession();
  auto RegisterImplName =
      ES.intern(TestMode ? RegisterTestVTuneImplName : RegisterVTuneImplName);
  auto UnregisterImplName = ES.intern(UnregisterVTuneImplName);
  SymbolLookupSet SLS{RegisterImplName, UnregisterImplName};
  auto Res = ES.lookup(makeJITDylibSearchOrder({&JD}), std::move(SLS));
```
- **EN**: Implements logic around `append_range`, `erase`, `Create`, `getExecutionSession`, and 2 more symbols; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `append_range`, `erase`, `Create`, `getExecutionSession`, and 2 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 178-186
```cpp
  if (!Res)
    return Res.takeError();
  ExecutorAddr RegisterImplAddr(
      Res->find(RegisterImplName)->second.getAddress());
  ExecutorAddr UnregisterImplAddr(
      Res->find(UnregisterImplName)->second.getAddress());
  return std::make_unique<VTuneSupportPlugin>(
      EPC, RegisterImplAddr, UnregisterImplAddr, EmitDebugInfo);
}
```
- **EN**: Implements logic around `takeError`, `RegisterImplAddr`, `find`, `UnregisterImplAddr`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `takeError`, `RegisterImplAddr`, `find`, `UnregisterImplAddr`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

## Key Concepts / 关键概念

- **ORC JIT / ORC JIT**:
  - **EN**: Coordinates JITDylibs, symbol materialization, execution sessions, and asynchronous compilation flows
  - **CN**: 协调 JITDylib、符号物化、ExecutionSession 与异步编译流程
- **Symbol materialization / 符号物化**:
  - **EN**: Defers code or data generation until symbols are requested
  - **CN**: 将代码或数据的生成延迟到符号真正被请求时
- **Relocations and fixups / 重定位与 fixup**:
  - **EN**: Bridges symbolic references to concrete addresses or relocation records
  - **CN**: 把符号引用连接到具体地址或重定位记录
- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/Orc/Debugging/VTuneSupportPlugin.h`, `llvm/DebugInfo/DWARF/DWARFContext.h`, `llvm/ExecutionEngine/Orc/Debugging/DebugInfoSupport.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine
