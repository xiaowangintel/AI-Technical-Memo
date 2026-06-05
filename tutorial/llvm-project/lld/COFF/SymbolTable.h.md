# SymbolTable.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `lld/COFF/SymbolTable.h`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: *- C++ -*. It also sits in code that implements COFF/PE linking components in LLD. / 该文件实现 LLD 中的 COFF/PE 链接组件。 源码头部说明其职责是：*- C++ -*。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
//===- SymbolTable.h --------------------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 9-19

```cpp
#ifndef LLD_COFF_SYMBOL_TABLE_H
#define LLD_COFF_SYMBOL_TABLE_H

#include "InputFiles.h"
#include "LTO.h"
#include "llvm/ADT/CachedHashString.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/DenseMapInfo.h"
#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/Support/raw_ostream.h"
```

- EN: Pulls in 7 header(s) from LLVM, system dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Defines macros such as `LLD_COFF_SYMBOL_TABLE_H` for constants or compile-time switches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 7 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里定义宏 `LLD_COFF_SYMBOL_TABLE_H`，用于常量或编译期开关。这里的注释记录了设计假设、不变量或使用说明。

### Lines 20-31

```cpp
namespace llvm {
struct LTOCodeGenerator;
}

namespace lld::coff {

class Chunk;
class CommonChunk;
class COFFLinkerContext;
class Defined;
class DefinedAbsolute;
class DefinedRegular;
```

- EN: Works inside namespace scope `llvm`, `lld` to organize symbols. Introduces type definitions such as `LTOCodeGenerator`, `Chunk`, `CommonChunk`, `COFFLinkerContext`, `Defined`, and 2 more. Notable symbols here include `LTOCodeGenerator`, `Chunk`, `CommonChunk`, `COFFLinkerContext`, `Defined`, `DefinedAbsolute`.
- CN: 这里位于命名空间 `llvm`, `lld` 中，用于组织符号作用域。这里引入类型定义，例如 `LTOCodeGenerator`, `Chunk`, `CommonChunk`, `COFFLinkerContext`, `Defined`, and 2 more。这里较值得关注的符号包括 `LTOCodeGenerator`, `Chunk`, `CommonChunk`, `COFFLinkerContext`, `Defined`, `DefinedAbsolute`。

### Lines 32-44

```cpp
class ImportThunkChunk;
class LazyArchive;
class SameAddressThunkARM64EC;
class SectionChunk;
class Symbol;

// This data structure is instantiated for each -wrap option.
struct WrappedSymbol {
  Symbol *sym;
  Symbol *real;
  Symbol *wrap;
};
```

- EN: Introduces type definitions such as `ImportThunkChunk`, `LazyArchive`, `SameAddressThunkARM64EC`, `SectionChunk`, `Symbol`, and 1 more. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `ImportThunkChunk`, `LazyArchive`, `SameAddressThunkARM64EC`, `SectionChunk`, `Symbol`, `WrappedSymbol`.
- CN: 这里引入类型定义，例如 `ImportThunkChunk`, `LazyArchive`, `SameAddressThunkARM64EC`, `SectionChunk`, `Symbol`, and 1 more。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `ImportThunkChunk`, `LazyArchive`, `SameAddressThunkARM64EC`, `SectionChunk`, `Symbol`, `WrappedSymbol`。

### Lines 45-59

```cpp
struct UndefinedDiag;

// SymbolTable is a bucket of all known symbols, including defined,
// undefined, or lazy symbols (the last one is symbols in archive
// files whose archive members are not yet loaded).
//
// We put all symbols of all files to a SymbolTable, and the
// SymbolTable selects the "best" symbols if there are name
// conflicts. For example, obviously, a defined symbol is better than
// an undefined symbol. Or, if there's a conflict between a lazy and a
// undefined, it'll read an archive member to read a real definition
// to replace the lazy symbol. The logic is implemented in the
// add*() functions, which are called by input files as they are parsed.
// There is one add* function per symbol type.
class SymbolTable {
```

- EN: Introduces type definitions such as `UndefinedDiag`, `SymbolTable`. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `UndefinedDiag`, `SymbolTable`.
- CN: 这里引入类型定义，例如 `UndefinedDiag`, `SymbolTable`。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `UndefinedDiag`, `SymbolTable`。

### Lines 60-67

```cpp
public:
  SymbolTable(COFFLinkerContext &c,
              llvm::COFF::MachineTypes machine = IMAGE_FILE_MACHINE_UNKNOWN)
      : ctx(c), machine(machine) {}

  // Emit errors for symbols that cannot be resolved.
  void reportUnresolvable();
```

- EN: Declares or implements routines including `ctx`, `reportUnresolvable`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `ctx`, `reportUnresolvable`.
- CN: 这里声明或实现函数，例如 `ctx`, `reportUnresolvable`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `ctx`, `reportUnresolvable`。

### Lines 68-75

```cpp
  // Try to resolve any undefined symbols and update the symbol table
  // accordingly, then print an error message for any remaining undefined
  // symbols and warn about imported local symbols.
  void resolveRemainingUndefines(std::vector<Undefined *> &aliases);

  // Try to resolve undefined symbols with alternate names.
  void resolveAlternateNames();
```

- EN: Declares or implements routines including `resolveRemainingUndefines`, `resolveAlternateNames`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `resolveRemainingUndefines`, `resolveAlternateNames`.
- CN: 这里声明或实现函数，例如 `resolveRemainingUndefines`, `resolveAlternateNames`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `resolveRemainingUndefines`, `resolveAlternateNames`。

### Lines 76-84

```cpp
  // Load lazy objects that are needed for MinGW automatic import and for
  // doing stdcall fixups.
  void loadMinGWSymbols();
  bool handleMinGWAutomaticImport(Symbol *sym, StringRef name);

  // Returns a symbol for a given name. Returns a nullptr if not found.
  Symbol *find(StringRef name) const;
  Symbol *findUnderscore(StringRef name) const;
```

- EN: Declares or implements routines including `loadMinGWSymbols`, `handleMinGWAutomaticImport`, `find`, `findUnderscore`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `loadMinGWSymbols`, `handleMinGWAutomaticImport`, `find`, `findUnderscore`.
- CN: 这里声明或实现函数，例如 `loadMinGWSymbols`, `handleMinGWAutomaticImport`, `find`, `findUnderscore`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `loadMinGWSymbols`, `handleMinGWAutomaticImport`, `find`, `findUnderscore`。

### Lines 85-93

```cpp
  void addUndefinedGlob(StringRef arg);

  // Occasionally we have to resolve an undefined symbol to its
  // mangled symbol. This function tries to find a mangled name
  // for U from the symbol table, and if found, set the symbol as
  // a weak alias for U.
  Symbol *findMangle(StringRef name);
  StringRef mangleMaybe(Symbol *s);
```

- EN: Declares or implements routines including `addUndefinedGlob`, `findMangle`, `mangleMaybe`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `addUndefinedGlob`, `findMangle`, `mangleMaybe`.
- CN: 这里声明或实现函数，例如 `addUndefinedGlob`, `findMangle`, `mangleMaybe`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `addUndefinedGlob`, `findMangle`, `mangleMaybe`。

### Lines 94-106

```cpp
  // Symbol names are mangled by prepending "_" on x86.
  StringRef mangle(StringRef sym);

  // Windows specific -- "main" is not the only main function in Windows.
  // You can choose one from these four -- {w,}{WinMain,main}.
  // There are four different entry point functions for them,
  // {w,}{WinMain,main}CRTStartup, respectively. The linker needs to
  // choose the right one depending on which "main" function is defined.
  // This function looks up the symbol table and resolve corresponding
  // entry point name.
  StringRef findDefaultEntry();
  WindowsSubsystem inferSubsystem();
```

- EN: Declares or implements routines including `mangle`, `findDefaultEntry`, `inferSubsystem`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `mangle`, `findDefaultEntry`, `inferSubsystem`.
- CN: 这里声明或实现函数，例如 `mangle`, `findDefaultEntry`, `inferSubsystem`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `mangle`, `findDefaultEntry`, `inferSubsystem`。

### Lines 107-114

```cpp
  // Build a set of COFF objects representing the combined contents of
  // BitcodeFiles and add them to the symbol table. Called after all files are
  // added and before the writer writes results to a file.
  void compileBitcodeFiles();

  // Creates an Undefined symbol and marks it as live.
  Symbol *addGCRoot(StringRef sym, bool aliasEC = false);
```

- EN: Declares or implements routines including `compileBitcodeFiles`, `addGCRoot`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `compileBitcodeFiles`, `addGCRoot`.
- CN: 这里声明或实现函数，例如 `compileBitcodeFiles`, `addGCRoot`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `compileBitcodeFiles`, `addGCRoot`。

### Lines 115-132

```cpp
  // Creates an Undefined symbol for a given name.
  Symbol *addUndefined(StringRef name);

  Symbol *addSynthetic(StringRef n, Chunk *c);
  Symbol *addAbsolute(StringRef n, uint64_t va);

  Symbol *addUndefined(StringRef name, InputFile *f, bool overrideLazy);
  void addLazyArchive(ArchiveFile *f, const Archive::Symbol &sym);
  void addLazyObject(InputFile *f, StringRef n);
  void addLazyDLLSymbol(DLLFile *f, DLLFile::Symbol *sym, StringRef n);
  Symbol *addAbsolute(StringRef n, COFFSymbolRef s);
  Symbol *addRegular(InputFile *f, StringRef n,
                     const llvm::object::coff_symbol_generic *s = nullptr,
                     SectionChunk *c = nullptr, uint32_t sectionOffset = 0,
                     bool isWeak = false);
  std::pair<DefinedRegular *, bool>
  addComdat(InputFile *f, StringRef n,
            const llvm::object::coff_symbol_generic *s = nullptr);
```

- EN: Declares or implements routines including `addUndefined`, `addSynthetic`, `addAbsolute`, `addLazyArchive`, `addLazyObject`, and 1 more. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `addUndefined`, `addSynthetic`, `addAbsolute`, `addLazyArchive`, `addLazyObject`, `addLazyDLLSymbol`.
- CN: 这里声明或实现函数，例如 `addUndefined`, `addSynthetic`, `addAbsolute`, `addLazyArchive`, `addLazyObject`, and 1 more。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `addUndefined`, `addSynthetic`, `addAbsolute`, `addLazyArchive`, `addLazyObject`, `addLazyDLLSymbol`。

### Lines 133-145

```cpp
  Symbol *addCommon(InputFile *f, StringRef n, uint64_t size,
                    const llvm::object::coff_symbol_generic *s = nullptr,
                    CommonChunk *c = nullptr);
  DefinedImportData *addImportData(StringRef n, ImportFile *f,
                                   Chunk *&location);
  Defined *addImportThunk(StringRef name, DefinedImportData *s,
                          ImportThunkChunk *chunk);
  void addLibcall(StringRef name);
  void addEntryThunk(Symbol *from, Symbol *to);
  void addExitThunk(Symbol *from, Symbol *to);
  void initializeECThunks();
  void initializeSameAddressThunks();
```

- EN: Declares or implements routines including `addLibcall`, `addEntryThunk`, `addExitThunk`, `initializeECThunks`, `initializeSameAddressThunks`. Notable symbols here include `addLibcall`, `addEntryThunk`, `addExitThunk`, `initializeECThunks`, `initializeSameAddressThunks`.
- CN: 这里声明或实现函数，例如 `addLibcall`, `addEntryThunk`, `addExitThunk`, `initializeECThunks`, `initializeSameAddressThunks`。这里较值得关注的符号包括 `addLibcall`, `addEntryThunk`, `addExitThunk`, `initializeECThunks`, `initializeSameAddressThunks`。

### Lines 146-154

```cpp
  void reportDuplicate(Symbol *existing, InputFile *newFile,
                       SectionChunk *newSc = nullptr,
                       uint32_t newSectionOffset = 0);

  COFFLinkerContext &ctx;
  llvm::COFF::MachineTypes machine;

  bool isEC() const { return machine == ARM64EC; }
```

- EN: Declares or implements routines including `isEC`. Notable symbols here include `isEC`.
- CN: 这里声明或实现函数，例如 `isEC`。这里较值得关注的符号包括 `isEC`。

### Lines 155-163

```cpp
  // An entry point symbol.
  Symbol *entry = nullptr;

  // A list of chunks which to be added to .rdata.
  std::vector<Chunk *> localImportChunks;

  // A list of EC EXP+ symbols.
  std::vector<Symbol *> expSymbols;
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 164-173

```cpp
  std::vector<SameAddressThunkARM64EC *> sameAddressThunks;

  // A list of DLL exports.
  std::vector<Export> exports;
  llvm::DenseSet<StringRef> directivesExports;
  bool hadExplicitExports;

  Chunk *edataStart = nullptr;
  Chunk *edataEnd = nullptr;
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 174-182

```cpp
  Symbol *delayLoadHelper = nullptr;
  Chunk *tailMergeUnwindInfoChunk = nullptr;

  // A list of wrapped symbols.
  std::vector<WrappedSymbol> wrapped;

  // Used for /alternatename.
  std::map<StringRef, StringRef> alternateNames;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 183-191

```cpp
  // Used for /aligncomm.
  std::map<std::string, int> alignComm;

  void fixupExports();
  void assignExportOrdinals();
  void parseModuleDefs(StringRef path);
  void parseAlternateName(StringRef);
  void parseAligncomm(StringRef);
```

- EN: Declares or implements routines including `fixupExports`, `assignExportOrdinals`, `parseModuleDefs`, `parseAlternateName`, `parseAligncomm`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `fixupExports`, `assignExportOrdinals`, `parseModuleDefs`, `parseAlternateName`, `parseAligncomm`.
- CN: 这里声明或实现函数，例如 `fixupExports`, `assignExportOrdinals`, `parseModuleDefs`, `parseAlternateName`, `parseAligncomm`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `fixupExports`, `assignExportOrdinals`, `parseModuleDefs`, `parseAlternateName`, `parseAligncomm`。

### Lines 192-199

```cpp
  // Iterates symbols in non-determinstic hash table order.
  template <typename T> void forEachSymbol(T callback) {
    for (auto &pair : symMap)
      callback(pair.second);
  }

  std::vector<BitcodeFile *> bitcodeFileInstances;
```

- EN: Declares or implements routines including `forEachSymbol`, `callback`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `forEachSymbol`, `callback`.
- CN: 这里声明或实现函数，例如 `forEachSymbol`, `callback`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `forEachSymbol`, `callback`。

### Lines 200-214

```cpp
  DefinedRegular *loadConfigSym = nullptr;
  uint32_t loadConfigSize = 0;
  void initializeLoadConfig();

  std::string printSymbol(Symbol *sym) const;

private:
  /// Given a name without "__imp_" prefix, returns a defined symbol
  /// with the "__imp_" prefix, if it exists.
  Defined *impSymbol(StringRef name);
  /// Inserts symbol if not already present.
  std::pair<Symbol *, bool> insert(StringRef name);
  /// Same as insert(Name), but also sets isUsedInRegularObj.
  std::pair<Symbol *, bool> insert(StringRef name, InputFile *f);
```

- EN: Declares or implements routines including `initializeLoadConfig`, `printSymbol`, `impSymbol`, `insert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `initializeLoadConfig`, `printSymbol`, `impSymbol`, `insert`.
- CN: 这里声明或实现函数，例如 `initializeLoadConfig`, `printSymbol`, `impSymbol`, `insert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `initializeLoadConfig`, `printSymbol`, `impSymbol`, `insert`。

### Lines 215-222

```cpp
  bool findUnderscoreMangle(StringRef sym);
  std::vector<Symbol *> getSymsWithPrefix(StringRef prefix);

  llvm::DenseMap<llvm::CachedHashStringRef, Symbol *> symMap;
  std::unique_ptr<BitcodeCompiler> lto;
  std::vector<std::pair<Symbol *, Symbol *>> entryThunks;
  llvm::DenseMap<Symbol *, Symbol *> exitThunks;
```

- EN: Declares or implements routines including `findUnderscoreMangle`, `getSymsWithPrefix`. Notable symbols here include `findUnderscoreMangle`, `getSymsWithPrefix`.
- CN: 这里声明或实现函数，例如 `findUnderscoreMangle`, `getSymsWithPrefix`。这里较值得关注的符号包括 `findUnderscoreMangle`, `getSymsWithPrefix`。

### Lines 223-231

```cpp
  void
  reportProblemSymbols(const llvm::SmallPtrSetImpl<Symbol *> &undefs,
                       const llvm::DenseMap<Symbol *, Symbol *> *localImports,
                       bool needBitcodeFiles);
  void reportUndefinedSymbol(const UndefinedDiag &undefDiag);
};

std::vector<std::string> getSymbolLocations(ObjFile *file, uint32_t symIndex);
```

- EN: Declares or implements routines including `reportUndefinedSymbol`, `getSymbolLocations`. Notable symbols here include `reportUndefinedSymbol`, `getSymbolLocations`.
- CN: 这里声明或实现函数，例如 `reportUndefinedSymbol`, `getSymbolLocations`。这里较值得关注的符号包括 `reportUndefinedSymbol`, `getSymbolLocations`。

### Lines 232-236

```cpp
StringRef ltrim1(StringRef s, const char *chars);

} // namespace lld::coff

#endif
```

- EN: Uses conditional compilation to choose platform- or feature-specific code paths. Works inside namespace scope `lld` to organize symbols. Declares or implements routines including `ltrim1`. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里使用条件编译，根据平台或特性切换不同代码路径。这里位于命名空间 `lld` 中，用于组织符号作用域。这里声明或实现函数，例如 `ltrim1`。这里的注释记录了设计假设、不变量或使用说明。

## Key Concepts / 关键概念

- `LTOCodeGenerator`: class or struct interface / 类或结构体接口
- `Chunk`: class or struct interface / 类或结构体接口
- `CommonChunk`: class or struct interface / 类或结构体接口
- `COFFLinkerContext`: class or struct interface / 类或结构体接口
- `ctx`: function or method entry point / 函数或方法入口
- `reportUnresolvable`: function or method entry point / 函数或方法入口
- `resolveRemainingUndefines`: function or method entry point / 函数或方法入口
- `resolveAlternateNames`: function or method entry point / 函数或方法入口

## Dependencies / 依赖关系

- LLVM headers / LLVM 头文件: `llvm/ADT/CachedHashString.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/DenseMapInfo.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/Support/raw_ostream.h`
- System headers / 系统头文件: `InputFiles.h`, `LTO.h`
- Directory context / 目录上下文: `lld/COFF` neighbors usually cooperate with this file to provide the surrounding subsystem / `lld/COFF` 下的相邻文件通常与本文件协作组成对应子系统
