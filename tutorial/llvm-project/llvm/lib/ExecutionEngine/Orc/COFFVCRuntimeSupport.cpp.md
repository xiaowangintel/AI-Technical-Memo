# COFFVCRuntimeSupport.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/Orc/COFFVCRuntimeSupport.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements VC runtime support in ORC.
  - **CN**: 实现 ORC JIT 基础设施，例如 ExecutionSession、JITDylib、物化流程、符号查找以及执行辅助组件。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===------- COFFVCRuntimeSupport.cpp - VC runtime support in ORC ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-16
```cpp

#include "llvm/ExecutionEngine/Orc/COFFVCRuntimeSupport.h"

#include "llvm/ExecutionEngine/Orc/COFF.h"
#include "llvm/ExecutionEngine/Orc/ExecutionUtils.h"
#include "llvm/ExecutionEngine/Orc/LookupAndRecordAddrs.h"
#include "llvm/Support/VirtualFileSystem.h"
#include "llvm/WindowsDriver/MSVCPaths.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/Orc/COFFVCRuntimeSupport.h`, `llvm/ExecutionEngine/Orc/COFF.h`, `llvm/ExecutionEngine/Orc/ExecutionUtils.h`, `llvm/ExecutionEngine/Orc/LookupAndRecordAddrs.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/Orc/COFFVCRuntimeSupport.h`, `llvm/ExecutionEngine/Orc/COFF.h`, `llvm/ExecutionEngine/Orc/ExecutionUtils.h`, `llvm/ExecutionEngine/Orc/LookupAndRecordAddrs.h`。

### Lines 17-30
```cpp
#define DEBUG_TYPE "orc"

using namespace llvm;
using namespace llvm::orc;
using namespace llvm::orc::shared;

Expected<std::unique_ptr<COFFVCRuntimeBootstrapper>>
COFFVCRuntimeBootstrapper::Create(ExecutionSession &ES,
                                  ObjectLinkingLayer &ObjLinkingLayer,
                                  const char *RuntimePath) {
  return std::unique_ptr<COFFVCRuntimeBootstrapper>(
      new COFFVCRuntimeBootstrapper(ES, ObjLinkingLayer, RuntimePath));
}

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 31-38
```cpp
COFFVCRuntimeBootstrapper::COFFVCRuntimeBootstrapper(
    ExecutionSession &ES, ObjectLinkingLayer &ObjLinkingLayer,
    const char *RuntimePath)
    : ES(ES), ObjLinkingLayer(ObjLinkingLayer) {
  if (RuntimePath)
    this->RuntimePath = RuntimePath;
}

```
- **EN**: Implements logic around `COFFVCRuntimeBootstrapper`, `ES`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `COFFVCRuntimeBootstrapper`, `ES` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 39-50
```cpp
Expected<std::vector<std::string>>
COFFVCRuntimeBootstrapper::loadStaticVCRuntime(JITDylib &JD,
                                               bool DebugVersion) {
  StringRef VCLibs[] = {"libvcruntime.lib", "libcmt.lib", "libcpmt.lib"};
  StringRef UCRTLibs[] = {"libucrt.lib"};
  std::vector<std::string> ImportedLibraries;
  if (auto Err = loadVCRuntime(JD, ImportedLibraries, ArrayRef(VCLibs),
                               ArrayRef(UCRTLibs)))
    return std::move(Err);
  return ImportedLibraries;
}

```
- **EN**: Implements logic around `loadStaticVCRuntime`, `ArrayRef`, `move`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `loadStaticVCRuntime`, `ArrayRef`, `move` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 51-62
```cpp
Expected<std::vector<std::string>>
COFFVCRuntimeBootstrapper::loadDynamicVCRuntime(JITDylib &JD,
                                                bool DebugVersion) {
  StringRef VCLibs[] = {"vcruntime.lib", "msvcrt.lib", "msvcprt.lib"};
  StringRef UCRTLibs[] = {"ucrt.lib"};
  std::vector<std::string> ImportedLibraries;
  if (auto Err = loadVCRuntime(JD, ImportedLibraries, ArrayRef(VCLibs),
                               ArrayRef(UCRTLibs)))
    return std::move(Err);
  return ImportedLibraries;
}

```
- **EN**: Implements logic around `loadDynamicVCRuntime`, `ArrayRef`, `move`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `loadDynamicVCRuntime`, `ArrayRef`, `move` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 63-76
```cpp
Error COFFVCRuntimeBootstrapper::loadVCRuntime(
    JITDylib &JD, std::vector<std::string> &ImportedLibraries,
    ArrayRef<StringRef> VCLibs, ArrayRef<StringRef> UCRTLibs) {
  MSVCToolchainPath Path;
  if (!RuntimePath.empty()) {
    Path.UCRTSdkLib = RuntimePath;
    Path.VCToolchainLib = RuntimePath;
  } else {
    auto ToolchainPath = getMSVCToolchainPath();
    if (!ToolchainPath)
      return ToolchainPath.takeError();
    Path = *ToolchainPath;
  }
  LLVM_DEBUG({
```
- **EN**: Implements logic around `loadVCRuntime`, `getMSVCToolchainPath`, `takeError`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `loadVCRuntime`, `getMSVCToolchainPath`, `takeError` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 77-84
```cpp
    dbgs() << "Using VC toolchain pathes\n";
    dbgs() << "  VC toolchain path: " << Path.VCToolchainLib << "\n";
    dbgs() << "  UCRT path: " << Path.UCRTSdkLib << "\n";
  });

  auto LoadLibrary = [&](SmallString<256> LibPath, StringRef LibName) -> Error {
    sys::path::append(LibPath, LibName);

```
- **EN**: Implements logic around `dbgs`, `append`.
- **CN**: 围绕 `dbgs`, `append` 实现具体逻辑。

### Lines 85-91
```cpp
    std::set<std::string> NewImportedLibraries;
    auto G = StaticLibraryDefinitionGenerator::Load(
        ObjLinkingLayer, LibPath.c_str(),
        COFFImportFileScanner(NewImportedLibraries));
    if (!G)
      return G.takeError();

```
- **EN**: Implements logic around `Load`, `c_str`, `COFFImportFileScanner`, `takeError`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `Load`, `c_str`, `COFFImportFileScanner`, `takeError` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 92-101
```cpp
    llvm::append_range(ImportedLibraries, NewImportedLibraries);

    JD.addGenerator(std::move(*G));

    return Error::success();
  };
  for (auto &Lib : UCRTLibs)
    if (auto Err = LoadLibrary(Path.UCRTSdkLib, Lib))
      return Err;

```
- **EN**: Implements logic around `append_range`, `addGenerator`, `success`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `append_range`, `addGenerator`, `success` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 102-110
```cpp
  for (auto &Lib : VCLibs)
    if (auto Err = LoadLibrary(Path.VCToolchainLib, Lib))
      return Err;
  ImportedLibraries.push_back("ntdll.dll");
  ImportedLibraries.push_back("Kernel32.dll");

  return Error::success();
}

```
- **EN**: Implements logic around `push_back`, `success`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `push_back`, `success` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 111-124
```cpp
Error COFFVCRuntimeBootstrapper::initializeStaticVCRuntime(JITDylib &JD) {
  ExecutorAddr jit_scrt_initialize, jit_scrt_dllmain_before_initialize_c,
      jit_scrt_initialize_type_info,
      jit_scrt_initialize_default_local_stdio_options;
  if (auto Err = lookupAndRecordAddrs(
          ES, LookupKind::Static, makeJITDylibSearchOrder(&JD),
          {{ES.intern("__scrt_initialize_crt"), &jit_scrt_initialize},
           {ES.intern("__scrt_dllmain_before_initialize_c"),
            &jit_scrt_dllmain_before_initialize_c},
           {ES.intern("?__scrt_initialize_type_info@@YAXXZ"),
            &jit_scrt_initialize_type_info},
           {ES.intern("__scrt_initialize_default_local_stdio_options"),
            &jit_scrt_initialize_default_local_stdio_options}}))
    return Err;
```
- **EN**: Implements logic around `initializeStaticVCRuntime`, `makeJITDylibSearchOrder`, `intern`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `initializeStaticVCRuntime`, `makeJITDylibSearchOrder`, `intern` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 125-132
```cpp

  auto RunVoidInitFunc = [&](ExecutorAddr Addr) -> Error {
    if (auto Res = ES.getExecutorProcessControl().runAsVoidFunction(Addr))
      return Error::success();
    else
      return Res.takeError();
  };

```
- **EN**: Implements logic around `success`, `takeError`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `takeError` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 133-140
```cpp
  auto R =
      ES.getExecutorProcessControl().runAsIntFunction(jit_scrt_initialize, 0);
  if (!R)
    return R.takeError();

  if (auto Err = RunVoidInitFunc(jit_scrt_dllmain_before_initialize_c))
    return Err;

```
- **EN**: Implements logic around `getExecutorProcessControl`, `takeError`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getExecutorProcessControl`, `takeError` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 141-147
```cpp
  if (auto Err = RunVoidInitFunc(jit_scrt_initialize_type_info))
    return Err;

  if (auto Err =
          RunVoidInitFunc(jit_scrt_initialize_default_local_stdio_options))
    return Err;

```
- **EN**: Implements logic around `RunVoidInitFunc`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `RunVoidInitFunc` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 148-156
```cpp
  SymbolAliasMap Alias;
  Alias[ES.intern("__run_after_c_init")] = {
      ES.intern("__scrt_dllmain_after_initialize_c"), JITSymbolFlags::Exported};
  if (auto Err = JD.define(symbolAliases(Alias)))
    return Err;

  return Error::success();
}

```
- **EN**: Implements logic around `intern`, `success`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `intern`, `success` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 157-169
```cpp
Expected<COFFVCRuntimeBootstrapper::MSVCToolchainPath>
COFFVCRuntimeBootstrapper::getMSVCToolchainPath() {
  std::string VCToolChainPath;
  ToolsetLayout VSLayout;
  IntrusiveRefCntPtr<vfs::FileSystem> VFS = vfs::getRealFileSystem();
  if (!findVCToolChainViaCommandLine(*VFS, std::nullopt, std::nullopt,
                                     std::nullopt, VCToolChainPath, VSLayout) &&
      !findVCToolChainViaEnvironment(*VFS, VCToolChainPath, VSLayout) &&
      !findVCToolChainViaSetupConfig(*VFS, {}, VCToolChainPath, VSLayout) &&
      !findVCToolChainViaRegistry(VCToolChainPath, VSLayout))
    return make_error<StringError>("Couldn't find msvc toolchain.",
                                   inconvertibleErrorCode());

```
- **EN**: Implements logic around `getMSVCToolchainPath`, `getRealFileSystem`, `findVCToolChainViaEnvironment`, `findVCToolChainViaSetupConfig`, and 3 more symbols; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `getMSVCToolchainPath`, `getRealFileSystem`, `findVCToolChainViaEnvironment`, `findVCToolChainViaSetupConfig`, and 3 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 170-176
```cpp
  std::string UniversalCRTSdkPath;
  std::string UCRTVersion;
  if (!getUniversalCRTSdkDir(*VFS, std::nullopt, std::nullopt, std::nullopt,
                             UniversalCRTSdkPath, UCRTVersion))
    return make_error<StringError>("Couldn't find universal sdk.",
                                   inconvertibleErrorCode());

```
- **EN**: Implements logic around `make_error<StringError>`, `inconvertibleErrorCode`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<StringError>`, `inconvertibleErrorCode` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 177-186
```cpp
  MSVCToolchainPath ToolchainPath;
  SmallString<256> VCToolchainLib(VCToolChainPath);
  sys::path::append(VCToolchainLib, "lib", "x64");
  ToolchainPath.VCToolchainLib = std::move(VCToolchainLib);

  SmallString<256> UCRTSdkLib(UniversalCRTSdkPath);
  sys::path::append(UCRTSdkLib, "Lib", UCRTVersion, "ucrt", "x64");
  ToolchainPath.UCRTSdkLib = std::move(UCRTSdkLib);
  return ToolchainPath;
}
```
- **EN**: Implements logic around `VCToolchainLib`, `append`, `move`, `UCRTSdkLib`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `VCToolchainLib`, `append`, `move`, `UCRTSdkLib` 实现具体逻辑；这一段返回子系统相关的计算结果。

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

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/Orc/COFFVCRuntimeSupport.h`, `llvm/ExecutionEngine/Orc/COFF.h`, `llvm/ExecutionEngine/Orc/ExecutionUtils.h`, `llvm/ExecutionEngine/Orc/LookupAndRecordAddrs.h`, `llvm/Support/VirtualFileSystem.h`, `llvm/WindowsDriver/MSVCPaths.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Support
