# MapFile.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `lld/COFF/MapFile.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: MapFile.cpp This file implements the /map option in the same format as link.exe (based on observations) Header (program name, timestamp info, preferred load address). It also sits in code that implements COFF/PE linking components in LLD. / 该文件实现 LLD 中的 COFF/PE 链接组件。 源码头部说明其职责是：MapFile.cpp This file implements the /map option in the same format as link.exe (based on observations) Header (program name, timestamp info, preferred load address)。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```cpp
//===- MapFile.cpp --------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the /map option in the same format as link.exe
// (based on observations)
//
// Header (program name, timestamp info, preferred load address)
//
// Section list (Start = Section index:Base address):
// Start         Length     Name                   Class
// 0001:00001000 00000015H .text                   CODE
//
// Symbols list:
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 19-29

```cpp
// Address        Publics by Value    Rva + Base          Lib:Object
// 0001:00001000  main                 0000000140001000    main.obj
// 0001:00001300  ?__scrt_common_main@@YAHXZ  0000000140001300 libcmt:exe_main.obj
//
// entry point at        0001:00000360
//
// Static symbols
//
// 0000:00000000  __guard_fids__       0000000140000000     libcmt : exe_main.obj
//===----------------------------------------------------------------------===//
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 30-40

```cpp
#include "MapFile.h"
#include "COFFLinkerContext.h"
#include "SymbolTable.h"
#include "Symbols.h"
#include "Writer.h"
#include "lld/Common/Timer.h"
#include "llvm/Support/Parallel.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/TimeProfiler.h"
#include "llvm/Support/raw_ostream.h"
```

- EN: Pulls in 10 header(s) from local project, LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 10 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 41-50

```cpp
using namespace llvm;
using namespace llvm::object;
using namespace lld;
using namespace lld::coff;

// Print out the first two columns of a line.
static void writeHeader(raw_ostream &os, uint32_t sec, uint64_t addr) {
  os << format(" %04x:%08llx", sec, addr);
}
```

- EN: Works inside namespace scope `llvm`, `lld` to organize symbols. Declares or implements routines including `writeHeader`, `format`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `writeHeader`, `format`, `llvm`, `lld`.
- CN: 这里位于命名空间 `llvm`, `lld` 中，用于组织符号作用域。这里声明或实现函数，例如 `writeHeader`, `format`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `writeHeader`, `format`, `llvm`, `lld`。

### Lines 51-65

```cpp
// Write the time stamp with the format used by link.exe
// It seems identical to strftime with "%c" on msvc build, but we need a
// locale-agnostic version.
static void writeFormattedTimestamp(raw_ostream &os, time_t tds) {
  constexpr const char *const days[7] = {"Sun", "Mon", "Tue", "Wed",
                                         "Thu", "Fri", "Sat"};
  constexpr const char *const months[12] = {"Jan", "Feb", "Mar", "Apr",
                                            "May", "Jun", "Jul", "Aug",
                                            "Sep", "Oct", "Nov", "Dec"};
  tm *time = localtime(&tds);
  os << format("%s %s %2d %02d:%02d:%02d %d", days[time->tm_wday],
               months[time->tm_mon], time->tm_mday, time->tm_hour, time->tm_min,
               time->tm_sec, time->tm_year + 1900);
}
```

- EN: Declares or implements routines including `writeFormattedTimestamp`, `localtime`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `writeFormattedTimestamp`, `localtime`.
- CN: 这里声明或实现函数，例如 `writeFormattedTimestamp`, `localtime`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `writeFormattedTimestamp`, `localtime`。

### Lines 66-74

```cpp
static void sortUniqueSymbols(std::vector<Defined *> &syms,
                              uint64_t imageBase) {
  // Build helper vector
  using SortEntry = std::pair<Defined *, size_t>;
  std::vector<SortEntry> v;
  v.resize(syms.size());
  for (size_t i = 0, e = syms.size(); i < e; ++i)
    v[i] = SortEntry(syms[i], i);
```

- EN: Declares or implements routines including `SortEntry`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `SortEntry`.
- CN: 这里声明或实现函数，例如 `SortEntry`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `SortEntry`。

### Lines 75-90

```cpp
  // Remove duplicate symbol pointers
  parallelSort(v, std::less<SortEntry>());
  auto end = llvm::unique(v, [](const SortEntry &a, const SortEntry &b) {
    return a.first == b.first;
  });
  v.erase(end, v.end());

  // Sort by RVA then original order
  parallelSort(v, [imageBase](const SortEntry &a, const SortEntry &b) {
    // Add config.imageBase to avoid comparing "negative" RVAs.
    // This can happen with symbols of Absolute kind
    uint64_t rvaa = imageBase + a.first->getRVA();
    uint64_t rvab = imageBase + b.first->getRVA();
    return rvaa < rvab || (rvaa == rvab && a.second < b.second);
  });
```

- EN: Declares or implements routines including `parallelSort`, `unique`, `getRVA`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `parallelSort`, `unique`, `getRVA`.
- CN: 这里声明或实现函数，例如 `parallelSort`, `unique`, `getRVA`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `parallelSort`, `unique`, `getRVA`。

### Lines 91-100

```cpp
  syms.resize(v.size());
  for (size_t i = 0, e = v.size(); i < e; ++i)
    syms[i] = v[i].first;
}

// Returns the lists of all symbols that we want to print out.
static void getSymbols(const COFFLinkerContext &ctx,
                       std::vector<Defined *> &syms,
                       std::vector<Defined *> &staticSyms) {
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 101-118

```cpp
  for (ObjFile *file : ctx.objFileInstances)
    for (Symbol *b : file->getSymbols()) {
      if (!b || !b->isLive())
        continue;
      if (auto *sym = dyn_cast<DefinedCOFF>(b)) {
        COFFSymbolRef symRef = sym->getCOFFSymbol();
        if (!symRef.isSectionDefinition() &&
            symRef.getStorageClass() != COFF::IMAGE_SYM_CLASS_LABEL) {
          if (symRef.getStorageClass() == COFF::IMAGE_SYM_CLASS_STATIC)
            staticSyms.push_back(sym);
          else
            syms.push_back(sym);
        }
      } else if (auto *sym = dyn_cast<Defined>(b)) {
        syms.push_back(sym);
      }
    }
```

- EN: Declares or implements routines including `getCOFFSymbol`, `if`. Notable symbols here include `getCOFFSymbol`, `if`.
- CN: 这里声明或实现函数，例如 `getCOFFSymbol`, `if`。这里较值得关注的符号包括 `getCOFFSymbol`, `if`。

### Lines 119-136

```cpp
  for (ImportFile *file : ctx.importFileInstances) {
    if (!file->live)
      continue;

    if (file->impSym)
      syms.push_back(file->impSym);
    if (file->thunkSym && file->thunkSym->isLive())
      syms.push_back(file->thunkSym);
    if (file->auxThunkSym && file->auxThunkSym->isLive())
      syms.push_back(file->auxThunkSym);
    if (file->impchkThunk)
      syms.push_back(file->impchkThunk->sym);
    if (file->impECSym)
      syms.push_back(file->impECSym);
    if (file->auxImpCopySym)
      syms.push_back(file->auxImpCopySym);
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 137-148

```cpp
  sortUniqueSymbols(syms, ctx.config.imageBase);
  sortUniqueSymbols(staticSyms, ctx.config.imageBase);
}

// Construct a map from symbols to their stringified representations.
static DenseMap<Defined *, std::string>
getSymbolStrings(const COFFLinkerContext &ctx, ArrayRef<Defined *> syms) {
  std::vector<std::string> str(syms.size());
  parallelFor((size_t)0, syms.size(), [&](size_t i) {
    raw_string_ostream os(str[i]);
    Defined *sym = syms[i];
```

- EN: Declares or implements routines including `sortUniqueSymbols`, `getSymbolStrings`, `str`, `parallelFor`, `os`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `sortUniqueSymbols`, `getSymbolStrings`, `str`, `parallelFor`, `os`.
- CN: 这里声明或实现函数，例如 `sortUniqueSymbols`, `getSymbolStrings`, `str`, `parallelFor`, `os`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `sortUniqueSymbols`, `getSymbolStrings`, `str`, `parallelFor`, `os`。

### Lines 149-164

```cpp
    uint16_t sectionIdx = 0;
    uint64_t address = 0;
    SmallString<128> fileDescr;

    if (auto *absSym = dyn_cast<DefinedAbsolute>(sym)) {
      address = absSym->getVA();
      fileDescr = "<absolute>";
    } else if (isa<DefinedSynthetic>(sym)) {
      fileDescr = "<linker-defined>";
    } else if (isa<DefinedCommon>(sym)) {
      fileDescr = "<common>";
    } else if (Chunk *chunk = sym->getChunk()) {
      address = sym->getRVA();
      if (OutputSection *sec = ctx.getOutputSection(chunk))
        address -= sec->header.VirtualAddress;
```

- EN: Declares or implements routines including `getVA`, `if`, `getRVA`. Notable symbols here include `getVA`, `if`, `getRVA`.
- CN: 这里声明或实现函数，例如 `getVA`, `if`, `getRVA`。这里较值得关注的符号包括 `getVA`, `if`, `getRVA`。

### Lines 165-174

```cpp
      sectionIdx = chunk->getOutputSectionIdx();

      InputFile *file;
      if (auto *impSym = dyn_cast<DefinedImportData>(sym))
        file = impSym->file;
      else if (auto *thunkSym = dyn_cast<DefinedImportThunk>(sym))
        file = thunkSym->wrappedSym->file;
      else
        file = sym->getFile();
```

- EN: Declares or implements routines including `getOutputSectionIdx`, `getFile`. Notable symbols here include `getOutputSectionIdx`, `getFile`.
- CN: 这里声明或实现函数，例如 `getOutputSectionIdx`, `getFile`。这里较值得关注的符号包括 `getOutputSectionIdx`, `getFile`。

### Lines 175-192

```cpp
      if (file) {
        if (!file->parentName.empty()) {
          fileDescr = sys::path::filename(file->parentName);
          sys::path::replace_extension(fileDescr, "");
          fileDescr += ":";
        }
        fileDescr += sys::path::filename(file->getName());
      }
    }
    writeHeader(os, sectionIdx, address);
    os << "       ";
    os << left_justify(sym->getName(), 26);
    os << " ";
    os << format_hex_no_prefix((ctx.config.imageBase + sym->getRVA()), 16);
    if (!fileDescr.empty()) {
      os << "     "; // FIXME : Handle "f" and "i" flags sometimes generated
                     // by link.exe in those spaces
      os << fileDescr;
```

- EN: Declares or implements routines including `filename`, `replace_extension`, `writeHeader`, `left_justify`, `format_hex_no_prefix`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `filename`, `replace_extension`, `writeHeader`, `left_justify`, `format_hex_no_prefix`.
- CN: 这里声明或实现函数，例如 `filename`, `replace_extension`, `writeHeader`, `left_justify`, `format_hex_no_prefix`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `filename`, `replace_extension`, `writeHeader`, `left_justify`, `format_hex_no_prefix`。

### Lines 193-201

```cpp
    }
  });

  DenseMap<Defined *, std::string> ret;
  for (size_t i = 0, e = syms.size(); i < e; ++i)
    ret[syms[i]] = std::move(str[i]);
  return ret;
}
```

- EN: Declares or implements routines including `move`. Notable symbols here include `move`.
- CN: 这里声明或实现函数，例如 `move`。这里较值得关注的符号包括 `move`。

### Lines 202-211

```cpp
void lld::coff::writeMapFile(COFFLinkerContext &ctx) {
  if (ctx.config.mapFile.empty())
    return;

  llvm::TimeTraceScope timeScope("Map file");
  std::error_code ec;
  raw_fd_ostream os(ctx.config.mapFile, ec, sys::fs::OF_None);
  if (ec)
    Fatal(ctx) << "cannot open " << ctx.config.mapFile << ": " << ec.message();
```

- EN: Declares or implements routines including `writeMapFile`, `timeScope`, `os`, `Fatal`. Notable symbols here include `writeMapFile`, `timeScope`, `os`, `Fatal`.
- CN: 这里声明或实现函数，例如 `writeMapFile`, `timeScope`, `os`, `Fatal`。这里较值得关注的符号包括 `writeMapFile`, `timeScope`, `os`, `Fatal`。

### Lines 212-220

```cpp
  ScopedTimer t1(ctx.totalMapTimer);

  // Collect symbol info that we want to print out.
  ScopedTimer t2(ctx.symbolGatherTimer);
  std::vector<Defined *> syms;
  std::vector<Defined *> staticSyms;
  getSymbols(ctx, syms, staticSyms);
  t2.stop();
```

- EN: Declares or implements routines including `t1`, `t2`, `getSymbols`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `t1`, `t2`, `getSymbols`.
- CN: 这里声明或实现函数，例如 `t1`, `t2`, `getSymbols`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `t1`, `t2`, `getSymbols`。

### Lines 221-230

```cpp
  ScopedTimer t3(ctx.symbolStringsTimer);
  DenseMap<Defined *, std::string> symStr = getSymbolStrings(ctx, syms);
  DenseMap<Defined *, std::string> staticSymStr =
      getSymbolStrings(ctx, staticSyms);
  t3.stop();

  ScopedTimer t4(ctx.writeTimer);
  SmallString<128> AppName = sys::path::filename(ctx.config.outputFile);
  sys::path::replace_extension(AppName, "");
```

- EN: Declares or implements routines including `t3`, `getSymbolStrings`, `t4`, `filename`, `replace_extension`. Notable symbols here include `t3`, `getSymbolStrings`, `t4`, `filename`, `replace_extension`.
- CN: 这里声明或实现函数，例如 `t3`, `getSymbolStrings`, `t4`, `filename`, `replace_extension`。这里较值得关注的符号包括 `t3`, `getSymbolStrings`, `t4`, `filename`, `replace_extension`。

### Lines 231-243

```cpp
  // Print out the file header
  os << " " << AppName << "\n";
  os << "\n";

  os << " Timestamp is " << format_hex_no_prefix(ctx.config.timestamp, 8)
     << " (";
  if (ctx.config.repro) {
    os << "Repro mode";
  } else {
    writeFormattedTimestamp(os, ctx.config.timestamp);
  }
  os << ")\n";
```

- EN: Declares or implements routines including `format_hex_no_prefix`, `writeFormattedTimestamp`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `format_hex_no_prefix`, `writeFormattedTimestamp`.
- CN: 这里声明或实现函数，例如 `format_hex_no_prefix`, `writeFormattedTimestamp`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `format_hex_no_prefix`, `writeFormattedTimestamp`。

### Lines 244-251

```cpp
  os << "\n";
  os << " Preferred load address is "
     << format_hex_no_prefix(ctx.config.imageBase, 16) << "\n";
  os << "\n";

  // Print out section table.
  os << " Start         Length     Name                   Class\n";
```

- EN: Declares or implements routines including `format_hex_no_prefix`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `format_hex_no_prefix`.
- CN: 这里声明或实现函数，例如 `format_hex_no_prefix`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `format_hex_no_prefix`。

### Lines 252-259

```cpp
  for (OutputSection *sec : ctx.outputSections) {
    // Merge display of chunks with same sectionName
    std::vector<std::pair<SectionChunk *, SectionChunk *>> ChunkRanges;
    for (Chunk *c : sec->chunks) {
      auto *sc = dyn_cast<SectionChunk>(c);
      if (!sc)
        continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 260-267

```cpp
      if (ChunkRanges.empty() ||
          c->getSectionName() != ChunkRanges.back().first->getSectionName()) {
        ChunkRanges.emplace_back(sc, sc);
      } else {
        ChunkRanges.back().second = sc;
      }
    }
```

- EN: Declares or implements routines including `getSectionName`. Notable symbols here include `getSectionName`.
- CN: 这里声明或实现函数，例如 `getSectionName`。这里较值得关注的符号包括 `getSectionName`。

### Lines 268-277

```cpp
    const bool isCodeSection =
        (sec->header.Characteristics & COFF::IMAGE_SCN_CNT_CODE) &&
        (sec->header.Characteristics & COFF::IMAGE_SCN_MEM_READ) &&
        (sec->header.Characteristics & COFF::IMAGE_SCN_MEM_EXECUTE);
    StringRef SectionClass = (isCodeSection ? "CODE" : "DATA");

    for (auto &cr : ChunkRanges) {
      size_t size =
          cr.second->getRVA() + cr.second->getSize() - cr.first->getRVA();
```

- EN: Declares or implements routines including `getRVA`. Notable symbols here include `getRVA`.
- CN: 这里声明或实现函数，例如 `getRVA`。这里较值得关注的符号包括 `getRVA`。

### Lines 278-286

```cpp
      auto address = cr.first->getRVA() - sec->header.VirtualAddress;
      writeHeader(os, sec->sectionIndex, address);
      os << " " << format_hex_no_prefix(size, 8) << "H";
      os << " " << left_justify(cr.first->getSectionName(), 23);
      os << " " << SectionClass;
      os << '\n';
    }
  }
```

- EN: Declares or implements routines including `getRVA`, `writeHeader`, `format_hex_no_prefix`, `left_justify`. Notable symbols here include `getRVA`, `writeHeader`, `format_hex_no_prefix`, `left_justify`.
- CN: 这里声明或实现函数，例如 `getRVA`, `writeHeader`, `format_hex_no_prefix`, `left_justify`。这里较值得关注的符号包括 `getRVA`, `writeHeader`, `format_hex_no_prefix`, `left_justify`。

### Lines 287-294

```cpp
  // Print out the symbols table (without static symbols)
  os << "\n";
  os << "  Address         Publics by Value              Rva+Base"
        "               Lib:Object\n";
  os << "\n";
  for (Defined *sym : syms)
    os << symStr[sym] << '\n';
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 295-312

```cpp
  // Print out the entry point.
  os << "\n";

  uint16_t entrySecIndex = 0;
  uint64_t entryAddress = 0;

  if (!ctx.config.noEntry) {
    Defined *entry = dyn_cast_or_null<Defined>(ctx.symtab.entry);
    if (entry) {
      Chunk *chunk = entry->getChunk();
      entrySecIndex = chunk->getOutputSectionIdx();
      entryAddress =
          entry->getRVA() - ctx.getOutputSection(chunk)->header.VirtualAddress;
    }
  }
  os << " entry point at         ";
  os << format("%04x:%08llx", entrySecIndex, entryAddress);
  os << "\n";
```

- EN: Declares or implements routines including `getChunk`, `getOutputSectionIdx`, `getRVA`, `format`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getChunk`, `getOutputSectionIdx`, `getRVA`, `format`.
- CN: 这里声明或实现函数，例如 `getChunk`, `getOutputSectionIdx`, `getRVA`, `format`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getChunk`, `getOutputSectionIdx`, `getRVA`, `format`。

### Lines 313-320

```cpp

  // Print out the static symbols
  os << "\n";
  os << " Static symbols\n";
  os << "\n";
  for (Defined *sym : staticSyms)
    os << staticSymStr[sym] << '\n';
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 321-333

```cpp
  // Print out the exported functions
  if (ctx.config.mapInfo) {
    os << "\n";
    os << " Exports\n";
    os << "\n";
    os << "  ordinal    name\n\n";
    for (Export &e : ctx.symtab.exports) {
      os << format("  %7d", e.ordinal) << "    " << e.name << "\n";
      if (!e.extName.empty() && e.extName != e.name)
        os << "               exported name: " << e.extName << "\n";
    }
  }
```

- EN: Declares or implements routines including `format`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `format`.
- CN: 这里声明或实现函数，例如 `format`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `format`。

### Lines 334-336

```cpp
  t4.stop();
  t1.stop();
}
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

## Key Concepts / 关键概念

- `writeHeader`: function or method entry point / 函数或方法入口
- `format`: function or method entry point / 函数或方法入口
- `writeFormattedTimestamp`: function or method entry point / 函数或方法入口
- `localtime`: function or method entry point / 函数或方法入口
- `SortEntry`: function or method entry point / 函数或方法入口
- `llvm`: namespace scope / 命名空间作用域
- `lld`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `lld/Common/Timer.h`
- LLVM headers / LLVM 头文件: `llvm/Support/Parallel.h`, `llvm/Support/Path.h`, `llvm/Support/TimeProfiler.h`, `llvm/Support/raw_ostream.h`
- System headers / 系统头文件: `MapFile.h`, `COFFLinkerContext.h`, `SymbolTable.h`, `Symbols.h`, `Writer.h`
- Directory context / 目录上下文: `lld/COFF` neighbors usually cooperate with this file to provide the surrounding subsystem / `lld/COFF` 下的相邻文件通常与本文件协作组成对应子系统
