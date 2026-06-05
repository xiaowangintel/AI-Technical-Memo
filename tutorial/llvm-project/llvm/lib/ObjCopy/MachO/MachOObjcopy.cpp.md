# MachOObjcopy.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/ObjCopy/MachO/MachOObjcopy.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements Mach-O-specific object rewriting for llvm-objcopy.
  - **CN**: 实现 llvm-objcopy 的 Mach-O 专用目标文件改写逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
//===- MachOObjcopy.cpp -----------------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/ObjCopy/MachO/MachOObjcopy.h"
#include "Archive.h"
#include "MachOReader.h"
#include "MachOWriter.h"
#include "llvm/ADT/DenseSet.h"
#include "llvm/ObjCopy/CommonConfig.h"
#include "llvm/ObjCopy/MachO/MachOConfig.h"
#include "llvm/ObjCopy/MultiFormatConfig.h"
#include "llvm/ObjCopy/ObjCopy.h"
#include "llvm/Object/ArchiveWriter.h"
#include "llvm/Object/MachOUniversal.h"
#include "llvm/Object/MachOUniversalWriter.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/ObjCopy/MachO/MachOObjcopy.h`, `Archive.h`, `MachOReader.h`, `MachOWriter.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/ObjCopy/MachO/MachOObjcopy.h`, `Archive.h`, `MachOReader.h`, `MachOWriter.h`。

### Lines 21-31
```cpp
#include "llvm/Support/Errc.h"
#include "llvm/Support/Error.h"
#include "llvm/Support/FileOutputBuffer.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/SmallVectorMemoryBuffer.h"

using namespace llvm;
using namespace llvm::objcopy;
using namespace llvm::objcopy::macho;
using namespace llvm::object;

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/Support/Errc.h`, `llvm/Support/Error.h`, `llvm/Support/FileOutputBuffer.h`, `llvm/Support/Path.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/Support/Errc.h`, `llvm/Support/Error.h`, `llvm/Support/FileOutputBuffer.h`, `llvm/Support/Path.h`。

### Lines 32-45
```cpp
using SectionPred = std::function<bool(const std::unique_ptr<Section> &Sec)>;
using LoadCommandPred = std::function<bool(const LoadCommand &LC)>;

#ifndef NDEBUG
static bool isLoadCommandWithPayloadString(const LoadCommand &LC) {
  // TODO: Add support for LC_REEXPORT_DYLIB, LC_LOAD_UPWARD_DYLIB and
  // LC_LAZY_LOAD_DYLIB
  return LC.MachOLoadCommand.load_command_data.cmd == MachO::LC_RPATH ||
         LC.MachOLoadCommand.load_command_data.cmd == MachO::LC_ID_DYLIB ||
         LC.MachOLoadCommand.load_command_data.cmd == MachO::LC_LOAD_DYLIB ||
         LC.MachOLoadCommand.load_command_data.cmd == MachO::LC_LOAD_WEAK_DYLIB;
}
#endif

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 46-59
```cpp
static StringRef getPayloadString(const LoadCommand &LC) {
  assert(isLoadCommandWithPayloadString(LC) &&
         "unsupported load command encountered");

  return StringRef(reinterpret_cast<const char *>(LC.Payload.data()),
                   LC.Payload.size())
      .rtrim('\0');
}

static Error removeSections(const CommonConfig &Config, Object &Obj) {
  SectionPred RemovePred = [](const std::unique_ptr<Section> &) {
    return false;
  };

```
- **EN**: Implements logic around `getPayloadString`, `assert`, `StringRef`, `size`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `getPayloadString`, `assert`, `StringRef`, `size`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 60-71
```cpp
  if (!Config.ToRemove.empty()) {
    RemovePred = [&Config, RemovePred](const std::unique_ptr<Section> &Sec) {
      return Config.ToRemove.matches(Sec->CanonicalName);
    };
  }

  if (Config.StripAll || Config.StripDebug) {
    // Remove all debug sections.
    RemovePred = [RemovePred](const std::unique_ptr<Section> &Sec) {
      if (Sec->Segname == "__DWARF")
        return true;

```
- **EN**: Implements logic around `empty`, `matches`.
- **CN**: 围绕 `empty`, `matches` 实现具体逻辑。

### Lines 72-82
```cpp
      return RemovePred(Sec);
    };
  }

  if (!Config.OnlySection.empty()) {
    // Overwrite RemovePred because --only-section takes priority.
    RemovePred = [&Config](const std::unique_ptr<Section> &Sec) {
      return !Config.OnlySection.matches(Sec->CanonicalName);
    };
  }

```
- **EN**: Implements logic around `RemovePred`, `empty`, `matches`.
- **CN**: 围绕 `RemovePred`, `empty`, `matches` 实现具体逻辑。

### Lines 83-92
```cpp
  return Obj.removeSections(RemovePred);
}

static void markSymbols(const CommonConfig &, Object &Obj) {
  // Symbols referenced from the indirect symbol table must not be removed.
  for (IndirectSymbolEntry &ISE : Obj.IndirectSymTable.Symbols)
    if (ISE.Symbol)
      (*ISE.Symbol)->Referenced = true;
}

```
- **EN**: Implements logic around `removeSections`, `markSymbols`.
- **CN**: 围绕 `removeSections`, `markSymbols` 实现具体逻辑。

### Lines 93-102
```cpp
static void updateAndRemoveSymbols(const CommonConfig &Config,
                                   const MachOConfig &MachOConfig,
                                   Object &Obj) {
  Obj.SymTable.updateSymbols([&](SymbolEntry &Sym) {
    if (Config.SymbolsToSkip.matches(Sym.Name))
      return;

    if (!Sym.isUndefinedSymbol() && Config.SymbolsToLocalize.matches(Sym.Name))
      Sym.n_type &= ~MachO::N_EXT;

```
- **EN**: Implements logic around `updateAndRemoveSymbols`, `updateSymbols`, `matches`, `isUndefinedSymbol`; this block applies object-format-specific rules.
- **CN**: 围绕 `updateAndRemoveSymbols`, `updateSymbols`, `matches`, `isUndefinedSymbol` 实现具体逻辑；该代码块应用目标文件格式专用规则。

### Lines 103-116
```cpp
    // Note: these two globalize flags have very similar names but different
    // meanings:
    //
    // --globalize-symbol: promote a symbol to global
    // --keep-global-symbol: all symbols except for these should be made local
    //
    // If --globalize-symbol is specified for a given symbol, it will be
    // global in the output file even if it is not included via
    // --keep-global-symbol. Because of that, make sure to check
    // --globalize-symbol second.
    if (!Sym.isUndefinedSymbol() && !Config.SymbolsToKeepGlobal.empty() &&
        !Config.SymbolsToKeepGlobal.matches(Sym.Name))
      Sym.n_type &= ~MachO::N_EXT;

```
- **EN**: Implements logic around `isUndefinedSymbol`, `matches`; this block applies object-format-specific rules.
- **CN**: 围绕 `isUndefinedSymbol`, `matches` 实现具体逻辑；该代码块应用目标文件格式专用规则。

### Lines 117-128
```cpp
    if (!Sym.isUndefinedSymbol() && Config.SymbolsToGlobalize.matches(Sym.Name))
      Sym.n_type |= MachO::N_EXT;

    if (Sym.isExternalSymbol() && !Sym.isUndefinedSymbol() &&
        (Config.Weaken || Config.SymbolsToWeaken.matches(Sym.Name)))
      Sym.n_desc |= MachO::N_WEAK_DEF;

    auto I = Config.SymbolsToRename.find(Sym.Name);
    if (I != Config.SymbolsToRename.end())
      Sym.Name = std::string(I->getValue());
  });

```
- **EN**: Implements logic around `isUndefinedSymbol`, `isExternalSymbol`, `matches`, `find`, and 2 more symbols; this block applies object-format-specific rules.
- **CN**: 围绕 `isUndefinedSymbol`, `isExternalSymbol`, `matches`, `find`, and 2 more symbols 实现具体逻辑；该代码块应用目标文件格式专用规则。

### Lines 129-148
```cpp
  auto RemovePred = [&Config, &MachOConfig,
                     &Obj](const std::unique_ptr<SymbolEntry> &N) {
    if (N->Referenced)
      return false;
    if (MachOConfig.KeepUndefined && N->isUndefinedSymbol())
      return false;
    if (N->n_desc & MachO::REFERENCED_DYNAMICALLY)
      return false;
    if (Config.StripAll)
      return true;
    if (Config.DiscardMode == DiscardType::All && !(N->n_type & MachO::N_EXT))
      return true;
    // This behavior is consistent with cctools' strip.
    if (Config.StripDebug && (N->n_type & MachO::N_STAB))
      return true;
    // This behavior is consistent with cctools' strip.
    if (MachOConfig.StripSwiftSymbols &&
        (Obj.Header.Flags & MachO::MH_DYLDLINK) && Obj.SwiftVersion &&
        *Obj.SwiftVersion && N->isSwiftSymbol())
      return true;
```
- **EN**: Implements logic around `isUndefinedSymbol`, `isSwiftSymbol`; this block applies object-format-specific rules.
- **CN**: 围绕 `isUndefinedSymbol`, `isSwiftSymbol` 实现具体逻辑；该代码块应用目标文件格式专用规则。

### Lines 149-159
```cpp
    return false;
  };

  Obj.SymTable.removeSymbols(RemovePred);
}

template <typename LCType>
static void updateLoadCommandPayloadString(LoadCommand &LC, StringRef S) {
  assert(isLoadCommandWithPayloadString(LC) &&
         "unsupported load command encountered");

```
- **EN**: Implements logic around `removeSymbols`, `updateLoadCommandPayloadString`, `assert`.
- **CN**: 围绕 `removeSymbols`, `updateLoadCommandPayloadString`, `assert` 实现具体逻辑。

### Lines 160-178
```cpp
  uint32_t NewCmdsize = alignTo(sizeof(LCType) + S.size() + 1, 8);

  LC.MachOLoadCommand.load_command_data.cmdsize = NewCmdsize;
  LC.Payload.assign(NewCmdsize - sizeof(LCType), 0);
  llvm::copy(S, LC.Payload.begin());
}

static LoadCommand buildRPathLoadCommand(StringRef Path) {
  LoadCommand LC;
  MachO::rpath_command RPathLC;
  RPathLC.cmd = MachO::LC_RPATH;
  RPathLC.path = sizeof(MachO::rpath_command);
  RPathLC.cmdsize = alignTo(sizeof(MachO::rpath_command) + Path.size() + 1, 8);
  LC.MachOLoadCommand.rpath_command_data = RPathLC;
  LC.Payload.assign(RPathLC.cmdsize - sizeof(MachO::rpath_command), 0);
  llvm::copy(Path, LC.Payload.begin());
  return LC;
}

```
- **EN**: Implements logic around `alignTo`, `assign`, `copy`, `buildRPathLoadCommand`; this block applies object-format-specific rules.
- **CN**: 围绕 `alignTo`, `assign`, `copy`, `buildRPathLoadCommand` 实现具体逻辑；该代码块应用目标文件格式专用规则。

### Lines 179-191
```cpp
static Error processLoadCommands(const MachOConfig &MachOConfig, Object &Obj) {
  // Remove RPaths.
  DenseSet<StringRef> RPathsToRemove(MachOConfig.RPathsToRemove.begin(),
                                     MachOConfig.RPathsToRemove.end());

  LoadCommandPred RemovePred = [&RPathsToRemove,
                                &MachOConfig](const LoadCommand &LC) {
    if (LC.MachOLoadCommand.load_command_data.cmd == MachO::LC_RPATH) {
      // When removing all RPaths we don't need to care
      // about what it contains
      if (MachOConfig.RemoveAllRpaths)
        return true;

```
- **EN**: Implements logic around `processLoadCommands`, `RPathsToRemove`, `end`; this block propagates recoverable errors through LLVM error utilities; applies object-format-specific rules.
- **CN**: 围绕 `processLoadCommands`, `RPathsToRemove`, `end` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并应用目标文件格式专用规则。

### Lines 192-203
```cpp
      StringRef RPath = getPayloadString(LC);
      if (RPathsToRemove.count(RPath)) {
        RPathsToRemove.erase(RPath);
        return true;
      }
    }
    return false;
  };

  if (Error E = Obj.removeLoadCommands(RemovePred))
    return E;

```
- **EN**: Implements logic around `getPayloadString`, `count`, `erase`, `removeLoadCommands`; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `getPayloadString`, `count`, `erase`, `removeLoadCommands` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 204-214
```cpp
  // Emit an error if the Mach-O binary does not contain an rpath path name
  // specified in -delete_rpath.
  for (StringRef RPath : MachOConfig.RPathsToRemove) {
    if (RPathsToRemove.count(RPath))
      return createStringError(errc::invalid_argument,
                               "no LC_RPATH load command with path: %s",
                               RPath.str().c_str());
  }

  DenseSet<StringRef> RPaths;

```
- **EN**: Implements logic around `count`, `createStringError`, `str`; this block propagates recoverable errors through LLVM error utilities; emits or serializes data to an external representation.
- **CN**: 围绕 `count`, `createStringError`, `str` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并把数据输出或序列化为外部表示。

### Lines 215-233
```cpp
  // Get all existing RPaths.
  for (LoadCommand &LC : Obj.LoadCommands) {
    if (LC.MachOLoadCommand.load_command_data.cmd == MachO::LC_RPATH)
      RPaths.insert(getPayloadString(LC));
  }

  // Throw errors for invalid RPaths.
  for (const auto &OldNew : MachOConfig.RPathsToUpdate) {
    StringRef Old = OldNew.getFirst();
    StringRef New = OldNew.getSecond();
    if (!RPaths.contains(Old))
      return createStringError(errc::invalid_argument,
                               "no LC_RPATH load command with path: " + Old);
    if (RPaths.contains(New))
      return createStringError(errc::invalid_argument,
                               "rpath '" + New +
                                   "' would create a duplicate load command");
  }

```
- **EN**: Implements logic around `insert`, `getFirst`, `getSecond`, `contains`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities; applies object-format-specific rules.
- **CN**: 围绕 `insert`, `getFirst`, `getSecond`, `contains`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并应用目标文件格式专用规则。

### Lines 234-250
```cpp
  // Update load commands.
  for (LoadCommand &LC : Obj.LoadCommands) {
    switch (LC.MachOLoadCommand.load_command_data.cmd) {
    case MachO::LC_ID_DYLIB:
      if (MachOConfig.SharedLibId)
        updateLoadCommandPayloadString<MachO::dylib_command>(
            LC, *MachOConfig.SharedLibId);
      break;

    case MachO::LC_RPATH: {
      StringRef RPath = getPayloadString(LC);
      StringRef NewRPath = MachOConfig.RPathsToUpdate.lookup(RPath);
      if (!NewRPath.empty())
        updateLoadCommandPayloadString<MachO::rpath_command>(LC, NewRPath);
      break;
    }

```
- **EN**: Implements logic around `dylib_command>`, `getPayloadString`, `lookup`, `empty`, and 1 more symbols; this block uses `switch`-style dispatch; applies object-format-specific rules.
- **CN**: 围绕 `dylib_command>`, `getPayloadString`, `lookup`, `empty`, and 1 more symbols 实现具体逻辑；该代码块使用 `switch` 风格分派，并应用目标文件格式专用规则。

### Lines 251-264
```cpp
    // TODO: Add LC_REEXPORT_DYLIB, LC_LAZY_LOAD_DYLIB, and LC_LOAD_UPWARD_DYLIB
    // here once llvm-objcopy supports them.
    case MachO::LC_LOAD_DYLIB:
    case MachO::LC_LOAD_WEAK_DYLIB:
      StringRef InstallName = getPayloadString(LC);
      StringRef NewInstallName =
          MachOConfig.InstallNamesToUpdate.lookup(InstallName);
      if (!NewInstallName.empty())
        updateLoadCommandPayloadString<MachO::dylib_command>(LC,
                                                             NewInstallName);
      break;
    }
  }

```
- **EN**: Implements logic around `getPayloadString`, `lookup`, `empty`, `dylib_command>`; this block applies object-format-specific rules.
- **CN**: 围绕 `getPayloadString`, `lookup`, `empty`, `dylib_command>` 实现具体逻辑；该代码块应用目标文件格式专用规则。

### Lines 265-274
```cpp
  // Add new RPaths.
  for (StringRef RPath : MachOConfig.RPathToAdd) {
    if (RPaths.contains(RPath))
      return createStringError(errc::invalid_argument,
                               "rpath '" + RPath +
                                   "' would create a duplicate load command");
    RPaths.insert(RPath);
    Obj.LoadCommands.push_back(buildRPathLoadCommand(RPath));
  }

```
- **EN**: Implements logic around `contains`, `createStringError`, `insert`, `push_back`; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `contains`, `createStringError`, `insert`, `push_back` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 275-285
```cpp
  for (StringRef RPath : MachOConfig.RPathToPrepend) {
    if (RPaths.contains(RPath))
      return createStringError(errc::invalid_argument,
                               "rpath '" + RPath +
                                   "' would create a duplicate load command");

    RPaths.insert(RPath);
    Obj.LoadCommands.insert(Obj.LoadCommands.begin(),
                            buildRPathLoadCommand(RPath));
  }

```
- **EN**: Implements logic around `contains`, `createStringError`, `insert`, `buildRPathLoadCommand`; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `contains`, `createStringError`, `insert`, `buildRPathLoadCommand` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 286-304
```cpp
  // Unlike appending rpaths, the indexes of subsequent load commands must
  // be recalculated after prepending one.
  if (!MachOConfig.RPathToPrepend.empty())
    Obj.updateLoadCommandIndexes();

  // Remove any empty segments if required.
  if (!MachOConfig.EmptySegmentsToRemove.empty()) {
    auto RemovePred = [&MachOConfig](const LoadCommand &LC) {
      if (LC.MachOLoadCommand.load_command_data.cmd == MachO::LC_SEGMENT_64 ||
          LC.MachOLoadCommand.load_command_data.cmd == MachO::LC_SEGMENT) {
        return LC.Sections.empty() &&
               MachOConfig.EmptySegmentsToRemove.contains(*LC.getSegmentName());
      }
      return false;
    };
    if (Error E = Obj.removeLoadCommands(RemovePred))
      return E;
  }

```
- **EN**: Implements logic around `empty`, `updateLoadCommandIndexes`, `contains`, `removeLoadCommands`; this block propagates recoverable errors through LLVM error utilities; applies object-format-specific rules.
- **CN**: 围绕 `empty`, `updateLoadCommandIndexes`, `contains`, `removeLoadCommands` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并应用目标文件格式专用规则。

### Lines 305-319
```cpp
  return Error::success();
}

static Error dumpSectionToFile(StringRef SecName, StringRef Filename,
                               StringRef InputFilename, Object &Obj) {
  for (LoadCommand &LC : Obj.LoadCommands)
    for (const std::unique_ptr<Section> &Sec : LC.Sections) {
      if (Sec->CanonicalName == SecName) {
        Expected<std::unique_ptr<FileOutputBuffer>> BufferOrErr =
            FileOutputBuffer::create(Filename, Sec->Content.size());
        if (!BufferOrErr)
          return createFileError(Filename, BufferOrErr.takeError());
        std::unique_ptr<FileOutputBuffer> Buf = std::move(*BufferOrErr);
        llvm::copy(Sec->Content, Buf->getBufferStart());

```
- **EN**: Implements logic around `success`, `dumpSectionToFile`, `create`, `createFileError`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `success`, `dumpSectionToFile`, `create`, `createFileError`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 320-329
```cpp
        if (Error E = Buf->commit())
          return createFileError(Filename, std::move(E));
        return Error::success();
      }
    }

  return createFileError(InputFilename, object_error::parse_failed,
                         "section '%s' not found", SecName.str().c_str());
}

```
- **EN**: Implements logic around `commit`, `createFileError`, `success`, `str`; this block propagates recoverable errors through LLVM error utilities; parses or classifies structured input.
- **CN**: 围绕 `commit`, `createFileError`, `success`, `str` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并解析或分类结构化输入。

### Lines 330-349
```cpp
static Error addSection(const NewSectionInfo &NewSection, Object &Obj) {
  std::pair<StringRef, StringRef> Pair = NewSection.SectionName.split(',');
  StringRef TargetSegName = Pair.first;
  Section Sec(TargetSegName, Pair.second);
  Sec.Content =
      Obj.NewSectionsContents.save(NewSection.SectionData->getBuffer());
  Sec.Size = Sec.Content.size();

  // Add the a section into an existing segment.
  for (LoadCommand &LC : Obj.LoadCommands) {
    std::optional<StringRef> SegName = LC.getSegmentName();
    if (SegName && SegName == TargetSegName) {
      uint64_t Addr = *LC.getSegmentVMAddr();
      for (const std::unique_ptr<Section> &S : LC.Sections)
        Addr = std::max(Addr, S->Addr + S->Size);
      LC.Sections.push_back(std::make_unique<Section>(Sec));
      LC.Sections.back()->Addr = Addr;
      return Error::success();
    }
  }
```
- **EN**: Implements logic around `addSection`, `split`, `Sec`, `save`, and 7 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `addSection`, `split`, `Sec`, `save`, and 7 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 350-359
```cpp

  // There's no segment named TargetSegName. Create a new load command and
  // Insert a new section into it.
  LoadCommand &NewSegment =
      Obj.addSegment(TargetSegName, alignTo(Sec.Size, 16384));
  NewSegment.Sections.push_back(std::make_unique<Section>(Sec));
  NewSegment.Sections.back()->Addr = *NewSegment.getSegmentVMAddr();
  return Error::success();
}

```
- **EN**: Implements logic around `addSegment`, `push_back`, `back`, `success`.
- **CN**: 围绕 `addSegment`, `push_back`, `back`, `success` 实现具体逻辑。

### Lines 360-376
```cpp
static Expected<Section &> findSection(StringRef SecName, Object &O) {
  StringRef SegName;
  std::tie(SegName, SecName) = SecName.split(",");
  // For compactness, intermediate object files (MH_OBJECT) contain
  // only one segment in which all sections are placed.
  // The static linker places each section in the named segment when building
  // the final product (any file that is not of type MH_OBJECT).
  //
  // Source:
  // https://math-atlas.sourceforge.net/devel/assembly/MachORuntime.pdf
  // page 57
  if (O.Header.FileType == MachO::HeaderFileType::MH_OBJECT) {
    for (const auto& LC : O.LoadCommands)
      for (const auto& Sec : LC.Sections)
        if (Sec->Segname == SegName && Sec->Sectname == SecName)
          return *Sec;

```
- **EN**: Implements logic around `findSection`, `tie`; this block applies object-format-specific rules.
- **CN**: 围绕 `findSection`, `tie` 实现具体逻辑；该代码块应用目标文件格式专用规则。

### Lines 377-396
```cpp
    StringRef ErrMsg = "could not find section with name '%s' in '%s' segment";
    return createStringError(errc::invalid_argument, ErrMsg.str().c_str(),
                             SecName.str().c_str(), SegName.str().c_str());
  }
  auto FoundSeg =
      llvm::find_if(O.LoadCommands, [SegName](const LoadCommand &LC) {
        return LC.getSegmentName() == SegName;
      });
  if (FoundSeg == O.LoadCommands.end())
    return createStringError(errc::invalid_argument,
                             "could not find segment with name '%s'",
                             SegName.str().c_str());
  auto FoundSec = llvm::find_if(FoundSeg->Sections,
                                [SecName](const std::unique_ptr<Section> &Sec) {
                                  return Sec->Sectname == SecName;
                                });
  if (FoundSec == FoundSeg->Sections.end())
    return createStringError(errc::invalid_argument,
                             "could not find section with name '%s'",
                             SecName.str().c_str());
```
- **EN**: Implements logic around `createStringError`, `str`, `find_if`, `getSegmentName`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `createStringError`, `str`, `find_if`, `getSegmentName`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 397-408
```cpp

  assert(FoundSec->get()->CanonicalName == (SegName + "," + SecName).str());
  return **FoundSec;
}

static Error updateSection(const NewSectionInfo &NewSection, Object &O) {
  Expected<Section &> SecToUpdateOrErr = findSection(NewSection.SectionName, O);

  if (!SecToUpdateOrErr)
    return SecToUpdateOrErr.takeError();
  Section &Sec = *SecToUpdateOrErr;

```
- **EN**: Implements logic around `assert`, `updateSection`, `findSection`, `takeError`; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `assert`, `updateSection`, `findSection`, `takeError` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 409-427
```cpp
  if (NewSection.SectionData->getBufferSize() > Sec.Size)
    return createStringError(
        errc::invalid_argument,
        "new section cannot be larger than previous section");
  Sec.Content = O.NewSectionsContents.save(NewSection.SectionData->getBuffer());
  Sec.Size = Sec.Content.size();
  return Error::success();
}

// isValidMachOCannonicalName returns success if Name is a MachO cannonical name
// ("<segment>,<section>") and lengths of both segment and section names are
// valid.
static Error isValidMachOCannonicalName(StringRef Name) {
  if (Name.count(',') != 1)
    return createStringError(errc::invalid_argument,
                             "invalid section name '%s' (should be formatted "
                             "as '<segment name>,<section name>')",
                             Name.str().c_str());

```
- **EN**: Implements logic around `getBufferSize`, `createStringError`, `save`, `size`, and 4 more symbols; this block propagates recoverable errors through LLVM error utilities; applies object-format-specific rules.
- **CN**: 围绕 `getBufferSize`, `createStringError`, `save`, `size`, and 4 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并应用目标文件格式专用规则。

### Lines 428-439
```cpp
  std::pair<StringRef, StringRef> Pair = Name.split(',');
  if (Pair.first.size() > 16)
    return createStringError(errc::invalid_argument,
                             "too long segment name: '%s'",
                             Pair.first.str().c_str());
  if (Pair.second.size() > 16)
    return createStringError(errc::invalid_argument,
                             "too long section name: '%s'",
                             Pair.second.str().c_str());
  return Error::success();
}

```
- **EN**: Implements logic around `split`, `size`, `createStringError`, `str`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `split`, `size`, `createStringError`, `str`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 440-451
```cpp
static Error handleArgs(const CommonConfig &Config,
                        const MachOConfig &MachOConfig, Object &Obj) {
  // Dump sections before add/remove for compatibility with GNU objcopy.
  for (StringRef Flag : Config.DumpSection) {
    StringRef SectionName;
    StringRef FileName;
    std::tie(SectionName, FileName) = Flag.split('=');
    if (Error E =
            dumpSectionToFile(SectionName, FileName, Config.InputFilename, Obj))
      return E;
  }

```
- **EN**: Implements logic around `handleArgs`, `tie`, `dumpSectionToFile`; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `handleArgs`, `tie`, `dumpSectionToFile` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 452-465
```cpp
  if (Error E = removeSections(Config, Obj))
    return createFileError(Config.InputFilename, std::move(E));

  // Mark symbols to determine which symbols are still needed.
  if (Config.StripAll)
    markSymbols(Config, Obj);

  updateAndRemoveSymbols(Config, MachOConfig, Obj);

  if (Config.StripAll)
    for (LoadCommand &LC : Obj.LoadCommands)
      for (std::unique_ptr<Section> &Sec : LC.Sections)
        Sec->Relocations.clear();

```
- **EN**: Implements logic around `removeSections`, `createFileError`, `markSymbols`, `updateAndRemoveSymbols`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `removeSections`, `createFileError`, `markSymbols`, `updateAndRemoveSymbols`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 466-479
```cpp
  for (const NewSectionInfo &NewSection : Config.AddSection) {
    if (Error E = isValidMachOCannonicalName(NewSection.SectionName))
      return createFileError(Config.InputFilename, std::move(E));
    if (Error E = addSection(NewSection, Obj))
      return createFileError(Config.InputFilename, std::move(E));
  }

  for (const NewSectionInfo &NewSection : Config.UpdateSection) {
    if (Error E = isValidMachOCannonicalName(NewSection.SectionName))
      return createFileError(Config.InputFilename, std::move(E));
    if (Error E = updateSection(NewSection, Obj))
      return createFileError(Config.InputFilename, std::move(E));
  }

```
- **EN**: Implements logic around `isValidMachOCannonicalName`, `createFileError`, `addSection`, `updateSection`; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `isValidMachOCannonicalName`, `createFileError`, `addSection`, `updateSection` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 480-494
```cpp
  if (Error E = processLoadCommands(MachOConfig, Obj))
    return createFileError(Config.InputFilename, std::move(E));

  return Error::success();
}

Error objcopy::macho::executeObjcopyOnBinary(const CommonConfig &Config,
                                             const MachOConfig &MachOConfig,
                                             object::MachOObjectFile &In,
                                             raw_ostream &Out) {
  MachOReader Reader(In);
  Expected<std::unique_ptr<Object>> O = Reader.create();
  if (!O)
    return createFileError(Config.InputFilename, O.takeError());

```
- **EN**: Implements logic around `processLoadCommands`, `createFileError`, `success`, `executeObjcopyOnBinary`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities; applies object-format-specific rules.
- **CN**: 围绕 `processLoadCommands`, `createFileError`, `success`, `executeObjcopyOnBinary`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并应用目标文件格式专用规则。

### Lines 495-514
```cpp
  if (O->get()->Header.FileType == MachO::HeaderFileType::MH_PRELOAD)
    return createStringError(std::errc::not_supported,
                             "%s: MH_PRELOAD files are not supported",
                             Config.InputFilename.str().c_str());

  if (Error E = handleArgs(Config, MachOConfig, **O))
    return E;

  // Page size used for alignment of segment sizes in Mach-O executables and
  // dynamic libraries.
  uint64_t PageSize;
  switch (In.getArch()) {
  case Triple::ArchType::arm:
  case Triple::ArchType::aarch64:
  case Triple::ArchType::aarch64_32:
    PageSize = 16384;
    break;
  default:
    PageSize = 4096;
  }
```
- **EN**: Implements logic around `get`, `createStringError`, `str`, `handleArgs`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities; uses `switch`-style dispatch; applies object-format-specific rules.
- **CN**: 围绕 `get`, `createStringError`, `str`, `handleArgs`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并使用 `switch` 风格分派，并应用目标文件格式专用规则。

### Lines 515-534
```cpp

  MachOWriter Writer(**O, In.is64Bit(), In.isLittleEndian(),
                     sys::path::filename(Config.OutputFilename), PageSize, Out);
  if (auto E = Writer.finalize())
    return E;
  return Writer.write();
}

Error objcopy::macho::executeObjcopyOnMachOUniversalBinary(
    const MultiFormatConfig &Config, const MachOUniversalBinary &In,
    raw_ostream &Out) {
  SmallVector<OwningBinary<Binary>, 2> Binaries;
  SmallVector<Slice, 2> Slices;
  for (const auto &O : In.objects()) {
    Expected<std::unique_ptr<Archive>> ArOrErr = O.getAsArchive();
    if (ArOrErr) {
      Expected<std::vector<NewArchiveMember>> NewArchiveMembersOrErr =
          createNewArchiveMembers(Config, **ArOrErr);
      if (!NewArchiveMembersOrErr)
        return NewArchiveMembersOrErr.takeError();
```
- **EN**: Implements logic around `Writer`, `filename`, `finalize`, `write`, and 5 more symbols; this block propagates recoverable errors through LLVM error utilities; emits or serializes data to an external representation; applies object-format-specific rules.
- **CN**: 围绕 `Writer`, `filename`, `finalize`, `write`, and 5 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并把数据输出或序列化为外部表示，并应用目标文件格式专用规则。

### Lines 535-554
```cpp
      auto Kind = (*ArOrErr)->kind();
      if (Kind == object::Archive::K_BSD)
        Kind = object::Archive::K_DARWIN;
      Expected<std::unique_ptr<MemoryBuffer>> OutputBufferOrErr =
          writeArchiveToBuffer(
              *NewArchiveMembersOrErr,
              (*ArOrErr)->hasSymbolTable() ? SymtabWritingMode::NormalSymtab
                                           : SymtabWritingMode::NoSymtab,
              Kind, Config.getCommonConfig().DeterministicArchives,
              (*ArOrErr)->isThin());
      if (!OutputBufferOrErr)
        return OutputBufferOrErr.takeError();
      Expected<std::unique_ptr<Binary>> BinaryOrErr =
          object::createBinary(**OutputBufferOrErr);
      if (!BinaryOrErr)
        return BinaryOrErr.takeError();
      Binaries.emplace_back(std::move(*BinaryOrErr),
                            std::move(*OutputBufferOrErr));
      Slices.emplace_back(*cast<Archive>(Binaries.back().getBinary()),
                          O.getCPUType(), O.getCPUSubType(),
```
- **EN**: Implements logic around `kind`, `writeArchiveToBuffer`, `hasSymbolTable`, `getCommonConfig`, and 6 more symbols; this block propagates recoverable errors through LLVM error utilities; emits or serializes data to an external representation.
- **CN**: 围绕 `kind`, `writeArchiveToBuffer`, `hasSymbolTable`, `getCommonConfig`, and 6 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并把数据输出或序列化为外部表示。

### Lines 555-574
```cpp
                          O.getArchFlagName(), O.getAlign());
      continue;
    }
    // The methods getAsArchive, getAsObjectFile, getAsIRObject of the class
    // ObjectForArch return an Error in case of the type mismatch. We need to
    // check each in turn to see what kind of slice this is, so ignore errors
    // produced along the way.
    consumeError(ArOrErr.takeError());

    Expected<std::unique_ptr<MachOObjectFile>> ObjOrErr = O.getAsObjectFile();
    if (!ObjOrErr) {
      consumeError(ObjOrErr.takeError());
      return createStringError(
          std::errc::invalid_argument,
          "slice for '%s' of the universal Mach-O binary "
          "'%s' is not a Mach-O object or an archive",
          O.getArchFlagName().c_str(),
          Config.getCommonConfig().InputFilename.str().c_str());
    }
    std::string ArchFlagName = O.getArchFlagName();
```
- **EN**: Implements logic around `getArchFlagName`, `consumeError`, `getAsObjectFile`, `createStringError`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities; parses or classifies structured input.
- **CN**: 围绕 `getArchFlagName`, `consumeError`, `getAsObjectFile`, `createStringError`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并解析或分类结构化输入。

### Lines 575-586
```cpp

    SmallVector<char, 0> Buffer;
    raw_svector_ostream MemStream(Buffer);

    Expected<const MachOConfig &> MachO = Config.getMachOConfig();
    if (!MachO)
      return MachO.takeError();

    if (Error E = executeObjcopyOnBinary(Config.getCommonConfig(), *MachO,
                                         **ObjOrErr, MemStream))
      return E;

```
- **EN**: Implements logic around `MemStream`, `getMachOConfig`, `takeError`, `executeObjcopyOnBinary`; this block propagates recoverable errors through LLVM error utilities; applies object-format-specific rules.
- **CN**: 围绕 `MemStream`, `getMachOConfig`, `takeError`, `executeObjcopyOnBinary` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并应用目标文件格式专用规则。

### Lines 587-596
```cpp
    auto MB = std::make_unique<SmallVectorMemoryBuffer>(
        std::move(Buffer), ArchFlagName, /*RequiresNullTerminator=*/false);
    Expected<std::unique_ptr<Binary>> BinaryOrErr = object::createBinary(*MB);
    if (!BinaryOrErr)
      return BinaryOrErr.takeError();
    Binaries.emplace_back(std::move(*BinaryOrErr), std::move(MB));
    Slices.emplace_back(*cast<MachOObjectFile>(Binaries.back().getBinary()),
                        O.getAlign());
  }

```
- **EN**: Implements logic around `make_unique`, `move`, `createBinary`, `takeError`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `make_unique`, `move`, `createBinary`, `takeError`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 597-601
```cpp
  if (Error Err = writeUniversalBinaryToStream(Slices, Out))
    return Err;

  return Error::success();
}
```
- **EN**: Implements logic around `writeUniversalBinaryToStream`, `success`; this block propagates recoverable errors through LLVM error utilities; emits or serializes data to an external representation.
- **CN**: 围绕 `writeUniversalBinaryToStream`, `success` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并把数据输出或序列化为外部表示。

## Key Concepts / 关键概念

- **Object rewriting / 目标文件改写**:
  - **EN**: Edits sections, symbols, and metadata in existing object containers.
  - **CN**: 修改现有目标容器中的节、符号和元数据。
- **Structured error handling / 结构化错误处理**:
  - **EN**: Uses `Expected`, `Error`, or related helpers to make failures explicit.
  - **CN**: 使用 `Expected`、`Error` 或相关辅助工具显式表示失败。
- **LLVM container usage / LLVM 容器使用**:
  - **EN**: Relies on LLVM ADT containers for performance-conscious in-memory data management.
  - **CN**: 依赖 LLVM ADT 容器来进行注重性能的内存数据管理。
- **Streaming output / 流式输出**:
  - **EN**: Writes diagnostics, serialized data, or textual representations through LLVM stream APIs.
  - **CN**: 通过 LLVM 流式 API 输出诊断、序列化数据或文本表示。

## Dependencies / 依赖关系

- **Direct LLVM/local includes / 直接的 LLVM/本地包含**: `llvm/ObjCopy/MachO/MachOObjcopy.h`, `Archive.h`, `MachOReader.h`, `MachOWriter.h`, `llvm/ADT/DenseSet.h`, `llvm/ObjCopy/CommonConfig.h`, `llvm/ObjCopy/MachO/MachOConfig.h`, `llvm/ObjCopy/MultiFormatConfig.h`, `llvm/ObjCopy/ObjCopy.h`, `llvm/Object/ArchiveWriter.h` ... (+7 more)
- **Subsystem categories / 子系统类别**: objcopy configuration and rewriting interfaces / objcopy 配置与改写接口 (5), support-library helpers / Support 库辅助功能 (5), object-file reading abstractions / 目标文件读取抽象 (3), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1)
