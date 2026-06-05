# DynamicLibrary.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Support/DynamicLibrary.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements the operating system DynamicLibrary concept.
- Purpose (CN): 该文件位于 LLVM 的 `Support` 目录中，主要实现与 `DynamicLibrary` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===-- DynamicLibrary.cpp - Runtime link/load libraries --------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
//  This file implements the operating system DynamicLibrary concept.
//
//===----------------------------------------------------------------------===//

#include "llvm/Support/DynamicLibrary.h"
#include "llvm-c/Support.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/StringMap.h"
#include "llvm/Config/config.h"
#include "llvm/Support/Mutex.h"
#include <vector>

using namespace llvm;
using namespace llvm::sys;

// All methods for HandleSet should be used holding SymbolsMutex.
class DynamicLibrary::HandleSet {
  using HandleList = std::vector<void *>;
  HandleList Handles;
  void *Process = &Invalid;

public:
  static void *DLOpen(const char *Filename, std::string *Err);
  static void DLClose(void *Handle);
  static void *DLSym(void *Handle, const char *Symbol);

  HandleSet() = default;
  ~HandleSet();

  HandleList::iterator Find(void *Handle) { return find(Handles, Handle); }

  bool Contains(void *Handle) {
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 7 direct dependencies, including `llvm/Support/DynamicLibrary.h`, `llvm-c/Support.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/StringMap.h`.
  CN: 引入了 7 个直接依赖，其中包括 `llvm/Support/DynamicLibrary.h`, `llvm-c/Support.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/StringMap.h`。
- EN: This section centers on `DLClose`, `~HandleSet`, `Find` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `DLClose`, `~HandleSet`, `Find` 等符号展开，负责查询现有状态并返回计算出的元数据。

### Lines 41-80

```cpp
    return Handle == Process || Find(Handle) != Handles.end();
  }

  bool AddLibrary(void *Handle, bool IsProcess = false, bool CanClose = true,
                  bool AllowDuplicates = false) {
#ifdef _WIN32
    assert((Handle == this ? IsProcess : !IsProcess) && "Bad Handle.");
#endif
    assert((!AllowDuplicates || !CanClose) &&
           "CanClose must be false if AllowDuplicates is true.");

    if (LLVM_LIKELY(!IsProcess)) {
      if (!AllowDuplicates && Find(Handle) != Handles.end()) {
        if (CanClose)
          DLClose(Handle);
        return false;
      }
      Handles.push_back(Handle);
    } else {
#ifndef _WIN32
      if (Process != &Invalid) {
        if (CanClose)
          DLClose(Process);
        if (Process == Handle)
          return false;
      }
#endif
      Process = Handle;
    }
    return true;
  }

  void CloseLibrary(void *Handle) {
    DLClose(Handle);
    HandleList::iterator it = Find(Handle);
    if (it != Handles.end()) {
      Handles.erase(it);
    }
  }

```
- EN: This section centers on `AddLibrary`, `assert`, `CloseLibrary` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `AddLibrary`, `assert`, `CloseLibrary` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 81-120

```cpp
  void *LibLookup(const char *Symbol, DynamicLibrary::SearchOrdering Order) {
    if (Order & SO_LoadOrder) {
      for (void *Handle : Handles) {
        if (void *Ptr = DLSym(Handle, Symbol))
          return Ptr;
      }
    } else {
      for (void *Handle : llvm::reverse(Handles)) {
        if (void *Ptr = DLSym(Handle, Symbol))
          return Ptr;
      }
    }
    return nullptr;
  }

  void *Lookup(const char *Symbol, DynamicLibrary::SearchOrdering Order) {
    assert(!((Order & SO_LoadedFirst) && (Order & SO_LoadedLast)) &&
           "Invalid Ordering");

    if (Process == &Invalid || (Order & SO_LoadedFirst)) {
      if (void *Ptr = LibLookup(Symbol, Order))
        return Ptr;
    }
    if (Process != &Invalid) {
      // Use OS facilities to search the current binary and all loaded libs.
      if (void *Ptr = DLSym(Process, Symbol))
        return Ptr;

      // Search any libs that might have been skipped because of RTLD_LOCAL.
      if (Order & SO_LoadedLast) {
        if (void *Ptr = LibLookup(Symbol, Order))
          return Ptr;
      }
    }
    return nullptr;
  }
};

namespace {

```
- EN: This section centers on `assert` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `assert` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 121-160

```cpp
struct Globals {
  // Collection of symbol name/value pairs to be searched prior to any
  // libraries.
  llvm::StringMap<void *> ExplicitSymbols;
  // Collections of known library handles.
  DynamicLibrary::HandleSet OpenedHandles;
  DynamicLibrary::HandleSet OpenedTemporaryHandles;
  // Lock for ExplicitSymbols, OpenedHandles, and OpenedTemporaryHandles.
  llvm::sys::SmartMutex<true> SymbolsMutex;
};

Globals &getGlobals() {
  static Globals G;
  return G;
}

} // namespace

#ifdef _WIN32

#include "Windows/DynamicLibrary.inc"

#else

#include "Unix/DynamicLibrary.inc"

#endif

char DynamicLibrary::Invalid;
DynamicLibrary::SearchOrdering DynamicLibrary::SearchOrder =
    DynamicLibrary::SO_Linker;

namespace llvm {
void *SearchForAddressOfSpecialSymbol(const char *SymbolName) {
  return DoSearch(SymbolName); // DynamicLibrary.inc
}
} // namespace llvm

void DynamicLibrary::AddSymbol(StringRef SymbolName, void *SymbolValue) {
  auto &G = getGlobals();
```
- EN: Brings in 2 direct dependencies, including `Windows/DynamicLibrary.inc`, `Unix/DynamicLibrary.inc`.
  CN: 引入了 2 个直接依赖，其中包括 `Windows/DynamicLibrary.inc`, `Unix/DynamicLibrary.inc`。
- EN: This section centers on `DoSearch`, `AddSymbol` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `DoSearch`, `AddSymbol` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code updates helper containers and temporary state and returns the resulting value to its callers.
  CN: 在这一段中，代码更新辅助容器和临时状态，并将结果返回给调用方。

### Lines 161-200

```cpp
  SmartScopedLock<true> Lock(G.SymbolsMutex);
  G.ExplicitSymbols[SymbolName] = SymbolValue;
}

DynamicLibrary DynamicLibrary::getPermanentLibrary(const char *FileName,
                                                   std::string *Err) {
  auto &G = getGlobals();
  void *Handle = HandleSet::DLOpen(FileName, Err);
  if (Handle != &Invalid) {
    SmartScopedLock<true> Lock(G.SymbolsMutex);
    G.OpenedHandles.AddLibrary(Handle, /*IsProcess*/ FileName == nullptr);
  }

  return DynamicLibrary(Handle);
}

DynamicLibrary DynamicLibrary::addPermanentLibrary(void *Handle,
                                                   std::string *Err) {
  auto &G = getGlobals();
  SmartScopedLock<true> Lock(G.SymbolsMutex);
  // If we've already loaded this library, tell the caller.
  if (!G.OpenedHandles.AddLibrary(Handle, /*IsProcess*/ false,
                                  /*CanClose*/ false))
    *Err = "Library already loaded";

  return DynamicLibrary(Handle);
}

DynamicLibrary DynamicLibrary::getLibrary(const char *FileName,
                                          std::string *Err) {
  assert(FileName && "Use getPermanentLibrary() for opening process handle");
  void *Handle = HandleSet::DLOpen(FileName, Err);
  if (Handle != &Invalid) {
    auto &G = getGlobals();
    SmartScopedLock<true> Lock(G.SymbolsMutex);
    G.OpenedTemporaryHandles.AddLibrary(Handle, /*IsProcess*/ false,
                                        /*CanClose*/ false,
                                        /*AllowDuplicates*/ true);
  }
  return DynamicLibrary(Handle);
```
- EN: This section centers on `Lock`, `getPermanentLibrary`, `DynamicLibrary` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `Lock`, `getPermanentLibrary`, `DynamicLibrary` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 201-240

```cpp
}

void DynamicLibrary::closeLibrary(DynamicLibrary &Lib) {
  auto &G = getGlobals();
  SmartScopedLock<true> Lock(G.SymbolsMutex);
  if (Lib.isValid()) {
    G.OpenedTemporaryHandles.CloseLibrary(Lib.Data);
    Lib.Data = &Invalid;
  }
}

void *DynamicLibrary::getAddressOfSymbol(const char *SymbolName) {
  if (!isValid())
    return nullptr;
  return HandleSet::DLSym(Data, SymbolName);
}

void *DynamicLibrary::SearchForAddressOfSymbol(const char *SymbolName) {
  {
    auto &G = getGlobals();
    SmartScopedLock<true> Lock(G.SymbolsMutex);

    // First check symbols added via AddSymbol().
    StringMap<void *>::iterator i = G.ExplicitSymbols.find(SymbolName);

    if (i != G.ExplicitSymbols.end())
      return i->second;

    // Now search the libraries.
    if (void *Ptr = G.OpenedHandles.Lookup(SymbolName, SearchOrder))
      return Ptr;
    if (void *Ptr = G.OpenedTemporaryHandles.Lookup(SymbolName, SearchOrder))
      return Ptr;
  }

  return llvm::SearchForAddressOfSpecialSymbol(SymbolName);
}

//===----------------------------------------------------------------------===//
// C API.
```
- EN: This section centers on `closeLibrary`, `Lock`, `DLSym` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `closeLibrary`, `Lock`, `DLSym` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and updates helper containers and temporary state.
  CN: 在这一段中，代码检查条件并处理边界情况，并更新辅助容器和临时状态。

### Lines 241-253

```cpp
//===----------------------------------------------------------------------===//

LLVMBool LLVMLoadLibraryPermanently(const char *Filename) {
  return llvm::sys::DynamicLibrary::LoadLibraryPermanently(Filename);
}

void *LLVMSearchForAddressOfSymbol(const char *symbolName) {
  return llvm::sys::DynamicLibrary::SearchForAddressOfSymbol(symbolName);
}

void LLVMAddSymbol(const char *symbolName, void *symbolValue) {
  return llvm::sys::DynamicLibrary::AddSymbol(symbolName, symbolValue);
}
```
- EN: This section centers on `LLVMLoadLibraryPermanently`, `LoadLibraryPermanently`, `SearchForAddressOfSymbol` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `LLVMLoadLibraryPermanently`, `LoadLibraryPermanently`, `SearchForAddressOfSymbol` 等符号展开，负责实现局部控制流程与状态维护。
- EN: This range returns the resulting value to its callers.
  CN: 这一段将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: LLVM support utilities / LLVM 支撑工具
- Core symbols / 核心符号: `DynamicLibrary`, `Globals`, `DLClose`, `~HandleSet`, `Find`, `Contains` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/Support/DynamicLibrary.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/StringMap.h`, `llvm/Config/config.h`, `llvm/Support/Mutex.h`
- Standard library / 标准库: `vector`
- Other/system headers / 其他或系统头文件: `llvm-c/Support.h`, `Windows/DynamicLibrary.inc`, `Unix/DynamicLibrary.inc`
- Related symbols / 相关符号: `DynamicLibrary`, `Globals`, `DLClose`, `~HandleSet`, `Find`, `Contains`, `AddLibrary`
