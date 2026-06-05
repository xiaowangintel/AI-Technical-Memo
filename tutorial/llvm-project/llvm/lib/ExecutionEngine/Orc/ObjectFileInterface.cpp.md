# ObjectFileInterface.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/Orc/ObjectFileInterface.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements MU interface utils for objects.
  - **CN**: 实现 ORC JIT 基础设施，例如 ExecutionSession、JITDylib、物化流程、符号查找以及执行辅助组件。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```cpp
//===------ ObjectFileInterface.cpp - MU interface utils for objects ------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/ExecutionEngine/Orc/ObjectFileInterface.h"
#include "llvm/ExecutionEngine/JITSymbol.h"
#include "llvm/ExecutionEngine/Orc/Shared/ObjectFormats.h"
#include "llvm/Object/COFF.h"
#include "llvm/Object/ELFObjectFile.h"
#include "llvm/Object/MachO.h"
#include "llvm/Object/ObjectFile.h"
#include "llvm/Object/XCOFFObjectFile.h"
#include <optional>

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/Orc/ObjectFileInterface.h`, `llvm/ExecutionEngine/JITSymbol.h`, `llvm/ExecutionEngine/Orc/Shared/ObjectFormats.h`, `llvm/Object/COFF.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/Orc/ObjectFileInterface.h`, `llvm/ExecutionEngine/JITSymbol.h`, `llvm/ExecutionEngine/Orc/Shared/ObjectFormats.h`, `llvm/Object/COFF.h`。

### Lines 19-28
```cpp
#define DEBUG_TYPE "orc"

namespace llvm {
namespace orc {

void addInitSymbol(MaterializationUnit::Interface &I, ExecutionSession &ES,
                   StringRef ObjFileName) {
  assert(!I.InitSymbol && "I already has an init symbol");
  size_t Counter = 0;

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 29-38
```cpp
  do {
    std::string InitSymString;
    raw_string_ostream(InitSymString)
        << "$." << ObjFileName << ".__inits." << Counter++;
    I.InitSymbol = ES.intern(InitSymString);
  } while (I.SymbolFlags.count(I.InitSymbol));

  I.SymbolFlags[I.InitSymbol] = JITSymbolFlags::MaterializationSideEffectsOnly;
}

```
- **EN**: Implements logic around `raw_string_ostream`, `intern`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `raw_string_ostream`, `intern` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 39-49
```cpp
static Expected<MaterializationUnit::Interface>
getMachOObjectFileSymbolInfo(ExecutionSession &ES,
                             const object::MachOObjectFile &Obj) {
  MaterializationUnit::Interface I;

  for (auto &Sym : Obj.symbols()) {
    Expected<uint32_t> SymFlagsOrErr = Sym.getFlags();
    if (!SymFlagsOrErr)
      // TODO: Test this error.
      return SymFlagsOrErr.takeError();

```
- **EN**: Implements logic around `getMachOObjectFileSymbolInfo`, `getFlags`, `takeError`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `getMachOObjectFileSymbolInfo`, `getFlags`, `takeError` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 50-64
```cpp
    // Skip symbols not defined in this object file.
    if (*SymFlagsOrErr & object::BasicSymbolRef::SF_Undefined)
      continue;

    // Skip symbols that are not global.
    if (!(*SymFlagsOrErr & object::BasicSymbolRef::SF_Global))
      continue;

    // Skip symbols that have type SF_File.
    if (auto SymType = Sym.getType()) {
      if (*SymType == object::SymbolRef::ST_File)
        continue;
    } else
      return SymType.takeError();

```
- **EN**: Implements logic around `takeError`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `takeError` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 65-75
```cpp
    auto Name = Sym.getName();
    if (!Name)
      return Name.takeError();
    auto SymFlags = JITSymbolFlags::fromObjectSymbol(Sym);
    if (!SymFlags)
      return SymFlags.takeError();

    // Strip the 'exported' flag from MachO linker-private symbols.
    if (Name->starts_with("l"))
      *SymFlags &= ~JITSymbolFlags::Exported;

```
- **EN**: Implements logic around `getName`, `takeError`, `fromObjectSymbol`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getName`, `takeError`, `fromObjectSymbol` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 76-92
```cpp
    I.SymbolFlags[ES.intern(*Name)] = std::move(*SymFlags);
  }

  for (auto &Sec : Obj.sections()) {
    auto SecType = Obj.getSectionType(Sec);
    if ((SecType & MachO::SECTION_TYPE) == MachO::S_MOD_INIT_FUNC_POINTERS) {
      addInitSymbol(I, ES, Obj.getFileName());
      break;
    }
    auto SegName = Obj.getSectionFinalSegmentName(Sec.getRawDataRefImpl());
    auto SecName = cantFail(Obj.getSectionName(Sec.getRawDataRefImpl()));
    if (isMachOInitializerSection(SegName, SecName)) {
      addInitSymbol(I, ES, Obj.getFileName());
      break;
    }
  }

```
- **EN**: Implements logic around `intern`, `getSectionType`, `addInitSymbol`, `getSectionFinalSegmentName`, and 1 more symbols.
- **CN**: 围绕 `intern`, `getSectionType`, `addInitSymbol`, `getSectionFinalSegmentName`, and 1 more symbols 实现具体逻辑。

### Lines 93-106
```cpp
  return I;
}

static Expected<MaterializationUnit::Interface>
getELFObjectFileSymbolInfo(ExecutionSession &ES,
                           const object::ELFObjectFileBase &Obj) {
  MaterializationUnit::Interface I;

  for (auto &Sym : Obj.symbols()) {
    Expected<uint32_t> SymFlagsOrErr = Sym.getFlags();
    if (!SymFlagsOrErr)
      // TODO: Test this error.
      return SymFlagsOrErr.takeError();

```
- **EN**: Implements logic around `getELFObjectFileSymbolInfo`, `getFlags`, `takeError`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `getELFObjectFileSymbolInfo`, `getFlags`, `takeError` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 107-121
```cpp
    // Skip symbols not defined in this object file.
    if (*SymFlagsOrErr & object::BasicSymbolRef::SF_Undefined)
      continue;

    // Skip symbols that are not global.
    if (!(*SymFlagsOrErr & object::BasicSymbolRef::SF_Global))
      continue;

    // Skip symbols that have type SF_File.
    if (auto SymType = Sym.getType()) {
      if (*SymType == object::SymbolRef::ST_File)
        continue;
    } else
      return SymType.takeError();

```
- **EN**: Implements logic around `takeError`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `takeError` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 122-133
```cpp
    auto Name = Sym.getName();
    if (!Name)
      return Name.takeError();

    auto SymFlags = JITSymbolFlags::fromObjectSymbol(Sym);
    if (!SymFlags)
      return SymFlags.takeError();

    // ELF STB_GNU_UNIQUE should map to Weak for ORC.
    if (Sym.getBinding() == ELF::STB_GNU_UNIQUE)
      *SymFlags |= JITSymbolFlags::Weak;

```
- **EN**: Implements logic around `getName`, `takeError`, `fromObjectSymbol`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getName`, `takeError`, `fromObjectSymbol` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 134-146
```cpp
    I.SymbolFlags[ES.intern(std::move(*Name))] = std::move(*SymFlags);
  }

  SymbolStringPtr InitSymbol;
  for (auto &Sec : Obj.sections()) {
    if (auto SecName = Sec.getName()) {
      if (isELFInitializerSection(*SecName)) {
        addInitSymbol(I, ES, Obj.getFileName());
        break;
      }
    }
  }

```
- **EN**: Implements logic around `intern`, `addInitSymbol`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `intern`, `addInitSymbol` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 147-161
```cpp
  return I;
}

static Expected<MaterializationUnit::Interface>
getCOFFObjectFileSymbolInfo(ExecutionSession &ES,
                            const object::COFFObjectFile &Obj) {
  MaterializationUnit::Interface I;
  std::vector<std::optional<object::coff_aux_section_definition>> ComdatDefs(
      Obj.getNumberOfSections() + 1);
  for (auto &Sym : Obj.symbols()) {
    Expected<uint32_t> SymFlagsOrErr = Sym.getFlags();
    if (!SymFlagsOrErr)
      // TODO: Test this error.
      return SymFlagsOrErr.takeError();

```
- **EN**: Implements logic around `getCOFFObjectFileSymbolInfo`, `ComdatDefs`, `getNumberOfSections`, `getFlags`, and 1 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `getCOFFObjectFileSymbolInfo`, `ComdatDefs`, `getNumberOfSections`, `getFlags`, and 1 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 162-179
```cpp
    // Handle comdat symbols
    auto COFFSym = Obj.getCOFFSymbol(Sym);
    bool IsWeak = false;
    if (auto *Def = COFFSym.getSectionDefinition()) {
      auto Sec = Obj.getSection(COFFSym.getSectionNumber());
      if (!Sec)
        return Sec.takeError();
      if (((*Sec)->Characteristics & COFF::IMAGE_SCN_LNK_COMDAT) &&
          Def->Selection != COFF::IMAGE_COMDAT_SELECT_ASSOCIATIVE) {
        ComdatDefs[COFFSym.getSectionNumber()] = *Def;
        continue;
      }
    }
    if (!COFF::isReservedSectionNumber(COFFSym.getSectionNumber()) &&
        ComdatDefs[COFFSym.getSectionNumber()]) {
      auto Def = ComdatDefs[COFFSym.getSectionNumber()];
      if (Def->Selection != COFF::IMAGE_COMDAT_SELECT_NODUPLICATES) {
        IsWeak = true;
```
- **EN**: Implements logic around `getCOFFSymbol`, `getSection`, `takeError`, `getSectionNumber`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getCOFFSymbol`, `getSection`, `takeError`, `getSectionNumber` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 180-191
```cpp
      }
      ComdatDefs[COFFSym.getSectionNumber()] = std::nullopt;
    } else {
      // Skip symbols not defined in this object file.
      if (*SymFlagsOrErr & object::BasicSymbolRef::SF_Undefined)
        continue;
    }

    // Skip symbols that are not global.
    if (!(*SymFlagsOrErr & object::BasicSymbolRef::SF_Global))
      continue;

```
- **EN**: Implements logic around `getSectionNumber`.
- **CN**: 围绕 `getSectionNumber` 实现具体逻辑。

### Lines 192-202
```cpp
    // Skip symbols that have type SF_File.
    if (auto SymType = Sym.getType()) {
      if (*SymType == object::SymbolRef::ST_File)
        continue;
    } else
      return SymType.takeError();

    auto Name = Sym.getName();
    if (!Name)
      return Name.takeError();

```
- **EN**: Implements logic around `takeError`, `getName`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `takeError`, `getName` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 203-211
```cpp
    auto SymFlags = JITSymbolFlags::fromObjectSymbol(Sym);
    if (!SymFlags)
      return SymFlags.takeError();
    *SymFlags |= JITSymbolFlags::Exported;

    // Weak external is always a function
    if (COFFSym.isWeakExternal())
      *SymFlags |= JITSymbolFlags::Callable;

```
- **EN**: Implements logic around `fromObjectSymbol`, `takeError`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `fromObjectSymbol`, `takeError` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 212-228
```cpp
    if (IsWeak)
      *SymFlags |= JITSymbolFlags::Weak;

    I.SymbolFlags[ES.intern(*Name)] = std::move(*SymFlags);
  }

  SymbolStringPtr InitSymbol;
  for (auto &Sec : Obj.sections()) {
    if (auto SecName = Sec.getName()) {
      if (isCOFFInitializerSection(*SecName)) {
        addInitSymbol(I, ES, Obj.getFileName());
        break;
      }
    } else
      return SecName.takeError();
  }

```
- **EN**: Implements logic around `intern`, `addInitSymbol`, `takeError`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `intern`, `addInitSymbol`, `takeError` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 229-237
```cpp
  return I;
}

Expected<MaterializationUnit::Interface>
getXCOFFObjectFileSymbolInfo(ExecutionSession &ES,
                             const object::ObjectFile &Obj) {

  MaterializationUnit::Interface I;

```
- **EN**: Implements logic around `getXCOFFObjectFileSymbolInfo`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `getXCOFFObjectFileSymbolInfo` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 238-249
```cpp
  for (auto &Sym : Obj.symbols()) {
    Expected<uint32_t> SymFlagsOrErr = Sym.getFlags();
    if (!SymFlagsOrErr)
      return SymFlagsOrErr.takeError();
    uint32_t Flags = *SymFlagsOrErr;

    // Skip undefined, non global and ST_File
    if (Flags & object::SymbolRef::SF_Undefined)
      continue;
    if (!(Flags & object::SymbolRef::SF_Global))
      continue;

```
- **EN**: Implements logic around `getFlags`, `takeError`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getFlags`, `takeError` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 250-263
```cpp
    auto SymbolType = Sym.getType();
    if (!SymbolType)
      return SymbolType.takeError();

    if (*SymbolType == object::SymbolRef::ST_File)
      continue;

    auto Name = Sym.getName();
    if (!Name)
      return Name.takeError();
    auto SymFlags = JITSymbolFlags::fromObjectSymbol(Sym);
    if (!SymFlags)
      return SymFlags.takeError();

```
- **EN**: Implements logic around `getType`, `takeError`, `getName`, `fromObjectSymbol`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getType`, `takeError`, `getName`, `fromObjectSymbol` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 264-277
```cpp
    // TODO: Revisit symbol visibility
    // On AIX, symbols with C_EXT and C_WEAKEXT symbols have no specified
    // visibility are considered to have Default scope for LinkGraph. When the
    // object is not a DSO, symbol visibility is not specified. In the absence
    // of an Export List, its reasonable to minimic roughly the behaviour of
    // -bexpall or CreateExportList.
    *SymFlags |= JITSymbolFlags::Exported;

    I.SymbolFlags[ES.intern(std::move(*Name))] = std::move(*SymFlags);
  }
  // TODO: Find all initialization symbols for c++ static initializers
  return I;
}

```
- **EN**: Implements logic around `intern`; this block manipulates JITLink graph structures or link-time passes; executes or prepares runtime behavior for LLVM IR/JIT code; returns subsystem-specific computed results.
- **CN**: 围绕 `intern` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，执行或准备 LLVM IR/JIT 代码的运行时行为，返回子系统相关的计算结果。

### Lines 278-288
```cpp
Expected<MaterializationUnit::Interface>
getGenericObjectFileSymbolInfo(ExecutionSession &ES,
                               const object::ObjectFile &Obj) {
  MaterializationUnit::Interface I;

  for (auto &Sym : Obj.symbols()) {
    Expected<uint32_t> SymFlagsOrErr = Sym.getFlags();
    if (!SymFlagsOrErr)
      // TODO: Test this error.
      return SymFlagsOrErr.takeError();

```
- **EN**: Implements logic around `getGenericObjectFileSymbolInfo`, `getFlags`, `takeError`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `getGenericObjectFileSymbolInfo`, `getFlags`, `takeError` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 289-303
```cpp
    // Skip symbols not defined in this object file.
    if (*SymFlagsOrErr & object::BasicSymbolRef::SF_Undefined)
      continue;

    // Skip symbols that are not global.
    if (!(*SymFlagsOrErr & object::BasicSymbolRef::SF_Global))
      continue;

    // Skip symbols that have type SF_File.
    if (auto SymType = Sym.getType()) {
      if (*SymType == object::SymbolRef::ST_File)
        continue;
    } else
      return SymType.takeError();

```
- **EN**: Implements logic around `takeError`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `takeError` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 304-314
```cpp
    auto Name = Sym.getName();
    if (!Name)
      return Name.takeError();

    auto SymFlags = JITSymbolFlags::fromObjectSymbol(Sym);
    if (!SymFlags)
      return SymFlags.takeError();

    I.SymbolFlags[ES.intern(*Name)] = std::move(*SymFlags);
  }

```
- **EN**: Implements logic around `getName`, `takeError`, `fromObjectSymbol`, `intern`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getName`, `takeError`, `fromObjectSymbol`, `intern` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 315-324
```cpp
  return I;
}

Expected<MaterializationUnit::Interface>
getObjectFileInterface(ExecutionSession &ES, MemoryBufferRef ObjBuffer) {
  auto Obj = object::ObjectFile::createObjectFile(ObjBuffer);

  if (!Obj)
    return Obj.takeError();

```
- **EN**: Implements logic around `getObjectFileInterface`, `createObjectFile`, `takeError`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `getObjectFileInterface`, `createObjectFile`, `takeError` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 325-333
```cpp
  if (auto *MachOObj = dyn_cast<object::MachOObjectFile>(Obj->get()))
    return getMachOObjectFileSymbolInfo(ES, *MachOObj);
  else if (auto *ELFObj = dyn_cast<object::ELFObjectFileBase>(Obj->get()))
    return getELFObjectFileSymbolInfo(ES, *ELFObj);
  else if (auto *COFFObj = dyn_cast<object::COFFObjectFile>(Obj->get()))
    return getCOFFObjectFileSymbolInfo(ES, *COFFObj);
  else if (auto *XCOFFObj = dyn_cast<object::XCOFFObjectFile>(Obj->get()))
    return getXCOFFObjectFileSymbolInfo(ES, *XCOFFObj);

```
- **EN**: Implements logic around `getMachOObjectFileSymbolInfo`, `getELFObjectFileSymbolInfo`, `getCOFFObjectFileSymbolInfo`, `getXCOFFObjectFileSymbolInfo`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getMachOObjectFileSymbolInfo`, `getELFObjectFileSymbolInfo`, `getCOFFObjectFileSymbolInfo`, `getXCOFFObjectFileSymbolInfo` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 334-338
```cpp
  return getGenericObjectFileSymbolInfo(ES, **Obj);
}

} // End namespace orc.
} // End namespace llvm.
```
- **EN**: Introduces declarations for `orc`, `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `orc`, `llvm` 等声明，定义本文件后续使用的数据结构或接口。

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

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/Orc/ObjectFileInterface.h`, `llvm/ExecutionEngine/JITSymbol.h`, `llvm/ExecutionEngine/Orc/Shared/ObjectFormats.h`, `llvm/Object/COFF.h`, `llvm/Object/ELFObjectFile.h`, `llvm/Object/MachO.h`, `llvm/Object/ObjectFile.h`, `llvm/Object/XCOFFObjectFile.h`, `optional`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Object
