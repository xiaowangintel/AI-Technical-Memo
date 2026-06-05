# JITSymbol.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/RuntimeDyld/JITSymbol.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements JITSymbol class implementation.
  - **CN**: 实现 RuntimeDyld，在运行时把目标代码装入内存、应用重定位并解析符号。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===----------- JITSymbol.cpp - JITSymbol class implementation -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-19
```cpp
//
// JITSymbol class implementation plus helper functions.
//
//===----------------------------------------------------------------------===//

#include "llvm/ExecutionEngine/JITSymbol.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/GlobalAlias.h"
#include "llvm/IR/GlobalValue.h"
#include "llvm/IR/ModuleSummaryIndex.h"
#include "llvm/Object/ObjectFile.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/JITSymbol.h`, `llvm/IR/Function.h`, `llvm/IR/GlobalAlias.h`, `llvm/IR/GlobalValue.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/JITSymbol.h`, `llvm/IR/Function.h`, `llvm/IR/GlobalAlias.h`, `llvm/IR/GlobalValue.h`。

### Lines 20-32
```cpp
using namespace llvm;

JITSymbolFlags llvm::JITSymbolFlags::fromGlobalValue(const GlobalValue &GV) {
  assert(GV.hasName() && "Can't get flags for anonymous symbol");

  JITSymbolFlags Flags = JITSymbolFlags::None;
  if (GV.hasWeakLinkage() || GV.hasLinkOnceLinkage())
    Flags |= JITSymbolFlags::Weak;
  if (GV.hasCommonLinkage())
    Flags |= JITSymbolFlags::Common;
  if (!GV.hasLocalLinkage() && !GV.hasHiddenVisibility())
    Flags |= JITSymbolFlags::Exported;

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 33-46
```cpp
  if (isa<Function>(GV))
    Flags |= JITSymbolFlags::Callable;
  else if (isa<GlobalAlias>(GV) &&
           isa<Function>(cast<GlobalAlias>(GV).getAliasee()))
    Flags |= JITSymbolFlags::Callable;

  // Check for a linker-private-global-prefix on the symbol name, in which
  // case it must be marked as non-exported.
  if (auto *M = GV.getParent()) {
    const auto &DL = M->getDataLayout();
    StringRef LPGP = DL.getLinkerPrivateGlobalPrefix();
    if (!LPGP.empty() && GV.getName().front() == '\01' &&
        GV.getName().substr(1).starts_with(LPGP))
      Flags &= ~JITSymbolFlags::Exported;
```
- **EN**: Implements logic around `isa<Function>`, `getDataLayout`, `getLinkerPrivateGlobalPrefix`, `getName`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `isa<Function>`, `getDataLayout`, `getLinkerPrivateGlobalPrefix`, `getName` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 47-60
```cpp
  }

  return Flags;
}

JITSymbolFlags llvm::JITSymbolFlags::fromSummary(GlobalValueSummary *S) {
  JITSymbolFlags Flags = JITSymbolFlags::None;
  auto L = S->linkage();
  if (GlobalValue::isWeakLinkage(L) || GlobalValue::isLinkOnceLinkage(L))
    Flags |= JITSymbolFlags::Weak;
  if (GlobalValue::isCommonLinkage(L))
    Flags |= JITSymbolFlags::Common;
  if (GlobalValue::isExternalLinkage(L) || GlobalValue::isExternalWeakLinkage(L))
    Flags |= JITSymbolFlags::Exported;
```
- **EN**: Implements logic around `fromSummary`, `linkage`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `fromSummary`, `linkage` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 61-67
```cpp

  if (isa<FunctionSummary>(S))
    Flags |= JITSymbolFlags::Callable;

  return Flags;
}

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 68-74
```cpp
Expected<JITSymbolFlags>
llvm::JITSymbolFlags::fromObjectSymbol(const object::SymbolRef &Symbol) {
  Expected<uint32_t> SymbolFlagsOrErr = Symbol.getFlags();
  if (!SymbolFlagsOrErr)
    // TODO: Test this error.
    return SymbolFlagsOrErr.takeError();

```
- **EN**: Implements logic around `fromObjectSymbol`, `getFlags`, `takeError`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `fromObjectSymbol`, `getFlags`, `takeError` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 75-82
```cpp
  JITSymbolFlags Flags = JITSymbolFlags::None;
  if (*SymbolFlagsOrErr & object::BasicSymbolRef::SF_Weak)
    Flags |= JITSymbolFlags::Weak;
  if (*SymbolFlagsOrErr & object::BasicSymbolRef::SF_Common)
    Flags |= JITSymbolFlags::Common;
  if (*SymbolFlagsOrErr & object::BasicSymbolRef::SF_Exported)
    Flags |= JITSymbolFlags::Exported;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 83-89
```cpp
  auto SymbolType = Symbol.getType();
  if (!SymbolType)
    return SymbolType.takeError();

  if (*SymbolType == object::SymbolRef::ST_Function)
    Flags |= JITSymbolFlags::Callable;

```
- **EN**: Implements logic around `getType`, `takeError`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getType`, `takeError` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 90-103
```cpp
  return Flags;
}

ARMJITSymbolFlags
llvm::ARMJITSymbolFlags::fromObjectSymbol(const object::SymbolRef &Symbol) {
  Expected<uint32_t> SymbolFlagsOrErr = Symbol.getFlags();
  if (!SymbolFlagsOrErr)
    // TODO: Actually report errors helpfully.
    report_fatal_error(SymbolFlagsOrErr.takeError());
  ARMJITSymbolFlags Flags;
  if (*SymbolFlagsOrErr & object::BasicSymbolRef::SF_Thumb)
    Flags |= ARMJITSymbolFlags::Thumb;
  return Flags;
}
```
- **EN**: Implements logic around `fromObjectSymbol`, `getFlags`, `report_fatal_error`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `fromObjectSymbol`, `getFlags`, `report_fatal_error` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 104-117
```cpp

/// Performs lookup by, for each symbol, first calling
///        findSymbolInLogicalDylib and if that fails calling
///        findSymbol.
void LegacyJITSymbolResolver::lookup(const LookupSet &Symbols,
                                     OnResolvedFunction OnResolved) {
  JITSymbolResolver::LookupResult Result;
  for (auto &Symbol : Symbols) {
    std::string SymName = Symbol.str();
    if (auto Sym = findSymbolInLogicalDylib(SymName)) {
      if (auto AddrOrErr = Sym.getAddress())
        Result[Symbol] = JITEvaluatedSymbol(*AddrOrErr, Sym.getFlags());
      else {
        OnResolved(AddrOrErr.takeError());
```
- **EN**: Implements logic around `lookup`, `str`, `JITEvaluatedSymbol`, `OnResolved`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `lookup`, `str`, `JITEvaluatedSymbol`, `OnResolved` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 118-131
```cpp
        return;
      }
    } else if (auto Err = Sym.takeError()) {
      OnResolved(std::move(Err));
      return;
    } else {
      // findSymbolInLogicalDylib failed. Lets try findSymbol.
      if (auto Sym = findSymbol(SymName)) {
        if (auto AddrOrErr = Sym.getAddress())
          Result[Symbol] = JITEvaluatedSymbol(*AddrOrErr, Sym.getFlags());
        else {
          OnResolved(AddrOrErr.takeError());
          return;
        }
```
- **EN**: Implements logic around `OnResolved`, `JITEvaluatedSymbol`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `OnResolved`, `JITEvaluatedSymbol` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 132-142
```cpp
      } else if (auto Err = Sym.takeError()) {
        OnResolved(std::move(Err));
        return;
      } else {
        OnResolved(make_error<StringError>("Symbol not found: " + Symbol,
                                           inconvertibleErrorCode()));
        return;
      }
    }
  }

```
- **EN**: Implements logic around `OnResolved`, `inconvertibleErrorCode`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `OnResolved`, `inconvertibleErrorCode` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 143-151
```cpp
  OnResolved(std::move(Result));
}

/// Performs flags lookup by calling findSymbolInLogicalDylib and
///        returning the flags value for that symbol.
Expected<JITSymbolResolver::LookupSet>
LegacyJITSymbolResolver::getResponsibilitySet(const LookupSet &Symbols) {
  JITSymbolResolver::LookupSet Result;

```
- **EN**: Implements logic around `OnResolved`, `getResponsibilitySet`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `OnResolved`, `getResponsibilitySet` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 152-165
```cpp
  for (auto &Symbol : Symbols) {
    std::string SymName = Symbol.str();
    if (auto Sym = findSymbolInLogicalDylib(SymName)) {
      // If there's an existing def but it is not strong, then the caller is
      // responsible for it.
      if (!Sym.getFlags().isStrong())
        Result.insert(Symbol);
    } else if (auto Err = Sym.takeError())
      return std::move(Err);
    else {
      // If there is no existing definition then the caller is responsible for
      // it.
      Result.insert(Symbol);
    }
```
- **EN**: Implements logic around `str`, `insert`, `move`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `str`, `insert`, `move` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 166-169
```cpp
  }

  return std::move(Result);
}
```
- **EN**: Implements logic around `move`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `move` 实现具体逻辑；这一段返回子系统相关的计算结果。

## Key Concepts / 关键概念

- **Runtime relocation / 运行时重定位**:
  - **EN**: Loads object code into memory and resolves relocations against runtime symbol tables
  - **CN**: 把目标代码装入内存并针对运行时符号表解析重定位

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/JITSymbol.h`, `llvm/IR/Function.h`, `llvm/IR/GlobalAlias.h`, `llvm/IR/GlobalValue.h`, `llvm/IR/ModuleSummaryIndex.h`, `llvm/Object/ObjectFile.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Object, IR
