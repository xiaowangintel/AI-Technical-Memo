# MinGW.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `lld/COFF/MinGW.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: MinGW.cpp. It also sits in code that implements COFF/PE linking components in LLD. / 该文件实现 LLD 中的 COFF/PE 链接组件。 源码头部说明其职责是：MinGW.cpp。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
//===- MinGW.cpp ----------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 9-20

```cpp
#include "MinGW.h"
#include "COFFLinkerContext.h"
#include "Driver.h"
#include "InputFiles.h"
#include "SymbolTable.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/DenseSet.h"
#include "llvm/Support/Parallel.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/TimeProfiler.h"
#include "llvm/Support/raw_ostream.h"
```

- EN: Pulls in 11 header(s) from LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 11 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 21-38

```cpp
using namespace llvm;
using namespace llvm::COFF;
using namespace lld;
using namespace lld::coff;

AutoExporter::AutoExporter(
    SymbolTable &symtab, const llvm::DenseSet<StringRef> &manualExcludeSymbols)
    : manualExcludeSymbols(manualExcludeSymbols), symtab(symtab) {
  excludeLibs = {
      "libgcc",
      "libgcc_s",
      "libstdc++",
      "libmingw32",
      "libmingwex",
      "libg2c",
      "libsupc++",
      "libobjc",
      "libgcj",
```

- EN: Works inside namespace scope `llvm`, `lld` to organize symbols. Declares or implements routines including `manualExcludeSymbols`. Notable symbols here include `manualExcludeSymbols`, `llvm`, `lld`.
- CN: 这里位于命名空间 `llvm`, `lld` 中，用于组织符号作用域。这里声明或实现函数，例如 `manualExcludeSymbols`。这里较值得关注的符号包括 `manualExcludeSymbols`, `llvm`, `lld`。

### Lines 39-56

```cpp
      "libclang_rt.builtins",
      "libclang_rt.builtins-aarch64",
      "libclang_rt.builtins-arm",
      "libclang_rt.builtins-i386",
      "libclang_rt.builtins-x86_64",
      "libclang_rt.profile",
      "libclang_rt.profile-aarch64",
      "libclang_rt.profile-arm",
      "libclang_rt.profile-i386",
      "libclang_rt.profile-x86_64",
      "libcygwin",
      "libmsys-2.0",
      "libc++",
      "libc++abi",
      "libflang_rt.runtime",
      "libunwind",
      "libmsvcrt",
      "libmsvcrt-os",
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 57-68

```cpp
      "libucrtbase",
      "libucrt",
      "libucrtapp",
      "libpthread",
      "libwinpthread",
  };

  excludeObjects = {
      "crt0.o",    "crt1.o",  "crt1u.o", "crt2.o",  "crt2u.o",    "dllcrt1.o",
      "dllcrt2.o", "gcrt0.o", "gcrt1.o", "gcrt2.o", "crtbegin.o", "crtend.o",
  };
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 69-85

```cpp
  excludeSymbolPrefixes = {
      // Import symbols
      "__imp_",
      "__IMPORT_DESCRIPTOR_",
      // Extra import symbols from GNU import libraries
      "__nm_",
      // C++ symbols
      "__rtti_",
      "__builtin_",
      // Artificial symbols such as .refptr
      ".",
      // profile generate symbols
      "__profc_",
      "__profd_",
      "__profvp_",
  };
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 86-103

```cpp
  excludeSymbolSuffixes = {
      "_iname",
      "_NULL_THUNK_DATA",
  };

  if (symtab.machine == I386) {
    excludeSymbols = {
        "__NULL_IMPORT_DESCRIPTOR",
        "__pei386_runtime_relocator",
        "_do_pseudo_reloc",
        "_impure_ptr",
        "__impure_ptr",
        "__fmode",
        "_environ",
        "___dso_handle",
        "__load_config_used",
        // These are the MinGW names that differ from the standard
        // ones (lacking an extra underscore).
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 104-121

```cpp
        "_DllMain@12",
        "_DllEntryPoint@12",
        "_DllMainCRTStartup@12",
    };
    excludeSymbolPrefixes.insert("__head_");
  } else {
    excludeSymbols = {
        "__NULL_IMPORT_DESCRIPTOR",
        "_pei386_runtime_relocator",
        "do_pseudo_reloc",
        "impure_ptr",
        "_impure_ptr",
        "_fmode",
        "environ",
        "__dso_handle",
        "_load_config_used",
        // These are the MinGW names that differ from the standard
        // ones (lacking an extra underscore).
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 122-133

```cpp
        "DllMain",
        "DllEntryPoint",
        "DllMainCRTStartup",
    };
    excludeSymbolPrefixes.insert("_head_");
  }
  if (symtab.isEC()) {
    excludeSymbols.insert("__chpe_metadata");
    excludeSymbolPrefixes.insert("__os_arm64x_");
  }
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 134-144

```cpp
void AutoExporter::addWholeArchive(StringRef path) {
  StringRef libName = sys::path::filename(path);
  // Drop the file extension, to match the processing below.
  libName = libName.substr(0, libName.rfind('.'));
  excludeLibs.erase(libName);
}

void AutoExporter::addExcludedSymbol(StringRef symbol) {
  excludeSymbols.insert(symbol);
}
```

- EN: Declares or implements routines including `addWholeArchive`, `filename`, `addExcludedSymbol`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `addWholeArchive`, `filename`, `addExcludedSymbol`.
- CN: 这里声明或实现函数，例如 `addWholeArchive`, `filename`, `addExcludedSymbol`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `addWholeArchive`, `filename`, `addExcludedSymbol`。

### Lines 145-156

```cpp
bool AutoExporter::shouldExport(Defined *sym) const {
  if (!sym || !sym->getChunk())
    return false;

  // Only allow the symbol kinds that make sense to export; in particular,
  // disallow import symbols.
  if (!isa<DefinedRegular>(sym) && !isa<DefinedCommon>(sym))
    return false;
  if (excludeSymbols.contains(sym->getName()) ||
      manualExcludeSymbols.contains(sym->getName()))
    return false;
```

- EN: Declares or implements routines including `shouldExport`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `shouldExport`.
- CN: 这里声明或实现函数，例如 `shouldExport`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `shouldExport`。

### Lines 157-167

```cpp
  for (StringRef prefix : excludeSymbolPrefixes.keys())
    if (sym->getName().starts_with(prefix))
      return false;
  for (StringRef suffix : excludeSymbolSuffixes.keys())
    if (sym->getName().ends_with(suffix))
      return false;

  // If a corresponding __imp_ symbol exists and is defined, don't export it.
  if (symtab.find(("__imp_" + sym->getName()).str()))
    return false;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 168-179

```cpp
  // Check that file is non-null before dereferencing it, symbols not
  // originating in regular object files probably shouldn't be exported.
  if (!sym->getFile())
    return false;

  StringRef libName = sys::path::filename(sym->getFile()->parentName);

  // Drop the file extension.
  libName = libName.substr(0, libName.rfind('.'));
  if (!libName.empty())
    return !excludeLibs.contains(libName);
```

- EN: Declares or implements routines including `filename`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `filename`.
- CN: 这里声明或实现函数，例如 `filename`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `filename`。

### Lines 180-191

```cpp
  StringRef fileName = sys::path::filename(sym->getFile()->getName());
  return !excludeObjects.contains(fileName);
}

void lld::coff::writeDefFile(COFFLinkerContext &ctx, StringRef name,
                             const std::vector<Export> &exports) {
  llvm::TimeTraceScope timeScope("Write .def file");
  std::error_code ec;
  raw_fd_ostream os(name, ec, sys::fs::OF_None);
  if (ec)
    Fatal(ctx) << "cannot open " << name << ": " << ec.message();
```

- EN: Declares or implements routines including `filename`, `timeScope`, `os`, `Fatal`. Notable symbols here include `filename`, `timeScope`, `os`, `Fatal`.
- CN: 这里声明或实现函数，例如 `filename`, `timeScope`, `os`, `Fatal`。这里较值得关注的符号包括 `filename`, `timeScope`, `os`, `Fatal`。

### Lines 192-204

```cpp
  os << "EXPORTS\n";
  for (const Export &e : exports) {
    os << "    " << e.exportName << " "
       << "@" << e.ordinal;
    if (auto *def = dyn_cast_or_null<Defined>(e.sym)) {
      if (def && def->getChunk() &&
          !(def->getChunk()->getOutputCharacteristics() & IMAGE_SCN_MEM_EXECUTE))
        os << " DATA";
    }
    os << "\n";
  }
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 205-221

```cpp
static StringRef mangle(Twine sym, MachineTypes machine) {
  assert(machine != IMAGE_FILE_MACHINE_UNKNOWN);
  if (machine == I386)
    return saver().save("_" + sym);
  return saver().save(sym);
}

// Handles -wrap option.
//
// This function instantiates wrapper symbols. At this point, they seem
// like they are not being used at all, so we explicitly set some flags so
// that LTO won't eliminate them.
void lld::coff::addWrappedSymbols(SymbolTable &symtab,
                                  opt::InputArgList &args) {
  std::vector<WrappedSymbol> v;
  DenseSet<StringRef> seen;
```

- EN: Declares or implements routines including `mangle`, `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `mangle`, `assert`.
- CN: 这里声明或实现函数，例如 `mangle`, `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `mangle`, `assert`。

### Lines 222-230

```cpp
  for (auto *arg : args.filtered(OPT_wrap)) {
    StringRef name = arg->getValue();
    if (!seen.insert(name).second)
      continue;

    Symbol *sym = symtab.findUnderscore(name);
    if (!sym)
      continue;
```

- EN: Declares or implements routines including `getValue`. Notable symbols here include `getValue`.
- CN: 这里声明或实现函数，例如 `getValue`。这里较值得关注的符号包括 `getValue`。

### Lines 231-247

```cpp
    Symbol *real =
        symtab.addUndefined(mangle("__real_" + name, symtab.machine));
    Symbol *wrap =
        symtab.addUndefined(mangle("__wrap_" + name, symtab.machine));
    v.push_back({sym, real, wrap});

    // These symbols may seem undefined initially, but don't bail out
    // at symtab.reportUnresolvable() due to them, but let wrapSymbols
    // below sort things out before checking finally with
    // symtab.resolveRemainingUndefines().
    sym->deferUndefined = true;
    real->deferUndefined = true;
    // We want to tell LTO not to inline symbols to be overwritten
    // because LTO doesn't know the final symbol contents after renaming.
    real->canInline = false;
    sym->canInline = false;
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 248-255

```cpp
    // Tell LTO not to eliminate these symbols.
    sym->isUsedInRegularObj = true;
    if (!isa<Undefined>(wrap))
      wrap->isUsedInRegularObj = true;
  }
  symtab.wrapped = std::move(v);
}
```

- EN: Declares or implements routines including `move`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `move`.
- CN: 这里声明或实现函数，例如 `move`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `move`。

### Lines 256-273

```cpp
// Do renaming for -wrap by updating pointers to symbols.
//
// When this function is executed, only InputFiles and symbol table
// contain pointers to symbol objects. We visit them to replace pointers,
// so that wrapped symbols are swapped as instructed by the command line.
void lld::coff::wrapSymbols(SymbolTable &symtab) {
  DenseMap<Symbol *, Symbol *> map;
  for (const WrappedSymbol &w : symtab.wrapped) {
    map[w.sym] = w.wrap;
    map[w.real] = w.sym;
    if (Defined *d = dyn_cast<Defined>(w.wrap)) {
      Symbol *imp = symtab.find(("__imp_" + w.sym->getName()).str());
      // Create a new defined local import for the wrap symbol. If
      // no imp prefixed symbol existed, there's no need for it.
      // (We can't easily distinguish whether any object file actually
      // referenced it or not, though.)
      if (imp) {
        if (Symbol *wrapimp =
```

- EN: Declares or implements routines including `wrapSymbols`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `wrapSymbols`.
- CN: 这里声明或实现函数，例如 `wrapSymbols`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `wrapSymbols`。

### Lines 274-285

```cpp
                symtab.find(("__imp_" + w.wrap->getName()).str())) {
          map[imp] = wrapimp;
        } else {
          DefinedLocalImport *localwrapimp = make<DefinedLocalImport>(
              symtab.ctx, saver().save("__imp_" + w.wrap->getName()), d);
          symtab.localImportChunks.push_back(localwrapimp->getChunk());
          map[imp] = localwrapimp;
        }
      }
    }
  }
```

- EN: Declares or implements routines including `saver`. Notable symbols here include `saver`.
- CN: 这里声明或实现函数，例如 `saver`。这里较值得关注的符号包括 `saver`。

### Lines 286-293

```cpp
  // Update pointers in input files.
  parallelForEach(symtab.ctx.objFileInstances, [&](ObjFile *file) {
    MutableArrayRef<Symbol *> syms = file->getMutableSymbols();
    for (auto &sym : syms)
      if (Symbol *s = map.lookup(sym))
        sym = s;
  });
}
```

- EN: Declares or implements routines including `parallelForEach`, `getMutableSymbols`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `parallelForEach`, `getMutableSymbols`.
- CN: 这里声明或实现函数，例如 `parallelForEach`, `getMutableSymbols`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `parallelForEach`, `getMutableSymbols`。

## Key Concepts / 关键概念

- `manualExcludeSymbols`: function or method entry point / 函数或方法入口
- `addWholeArchive`: function or method entry point / 函数或方法入口
- `filename`: function or method entry point / 函数或方法入口
- `addExcludedSymbol`: function or method entry point / 函数或方法入口
- `shouldExport`: function or method entry point / 函数或方法入口
- `llvm`: namespace scope / 命名空间作用域
- `lld`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- LLVM headers / LLVM 头文件: `llvm/ADT/DenseMap.h`, `llvm/ADT/DenseSet.h`, `llvm/Support/Parallel.h`, `llvm/Support/Path.h`, `llvm/Support/TimeProfiler.h`, `llvm/Support/raw_ostream.h`
- System headers / 系统头文件: `MinGW.h`, `COFFLinkerContext.h`, `Driver.h`, `InputFiles.h`, `SymbolTable.h`
- Directory context / 目录上下文: `lld/COFF` neighbors usually cooperate with this file to provide the surrounding subsystem / `lld/COFF` 下的相邻文件通常与本文件协作组成对应子系统
