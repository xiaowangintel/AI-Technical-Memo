# ELFObjcopy.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/ObjCopy/ELF/ELFObjcopy.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements ELF-specific object rewriting for llvm-objcopy.
  - **CN**: 实现 llvm-objcopy 的 ELF 专用目标文件改写逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28
```cpp
//===- ELFObjcopy.cpp -----------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/ObjCopy/ELF/ELFObjcopy.h"
#include "ELFObject.h"
#include "llvm/ADT/BitmaskEnum.h"
#include "llvm/ADT/DenseSet.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/Twine.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/MC/MCTargetOptions.h"
#include "llvm/ObjCopy/CommonConfig.h"
#include "llvm/ObjCopy/ELF/ELFConfig.h"
#include "llvm/Object/Binary.h"
#include "llvm/Object/ELFObjectFile.h"
#include "llvm/Object/ELFTypes.h"
#include "llvm/Object/Error.h"
#include "llvm/Option/Option.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/Compression.h"
#include "llvm/Support/Errc.h"
#include "llvm/Support/Error.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/ObjCopy/ELF/ELFObjcopy.h`, `ELFObject.h`, `llvm/ADT/BitmaskEnum.h`, `llvm/ADT/DenseSet.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/ObjCopy/ELF/ELFObjcopy.h`, `ELFObject.h`, `llvm/ADT/BitmaskEnum.h`, `llvm/ADT/DenseSet.h`。

### Lines 29-46
```cpp
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/Memory.h"
#include "llvm/Support/raw_ostream.h"
#include <algorithm>
#include <cassert>
#include <cstdlib>
#include <functional>
#include <memory>
#include <string>
#include <system_error>
#include <utility>

using namespace llvm;
using namespace llvm::ELF;
using namespace llvm::objcopy;
using namespace llvm::objcopy::elf;
using namespace llvm::object;

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/Support/ErrorHandling.h`, `llvm/Support/Memory.h`, `llvm/Support/raw_ostream.h`, `algorithm`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/Support/ErrorHandling.h`, `llvm/Support/Memory.h`, `llvm/Support/raw_ostream.h`, `algorithm`。

### Lines 47-65
```cpp
using SectionPred = std::function<bool(const SectionBase &Sec)>;

static bool isDebugSection(const SectionBase &Sec) {
  return StringRef(Sec.Name).starts_with(".debug") || Sec.Name == ".gdb_index";
}

static bool isDWOSection(const SectionBase &Sec) {
  return StringRef(Sec.Name).ends_with(".dwo");
}

static bool onlyKeepDWOPred(const Object &Obj, const SectionBase &Sec) {
  // We can't remove the section header string table.
  if (&Sec == Obj.SectionNames)
    return false;
  // Short of keeping the string table we want to keep everything that is a DWO
  // section and remove everything else.
  return !isDWOSection(Sec);
}

```
- **EN**: Implements logic around `function`, `isDebugSection`, `StringRef`, `isDWOSection`, and 1 more symbols.
- **CN**: 围绕 `function`, `isDebugSection`, `StringRef`, `isDWOSection`, and 1 more symbols 实现具体逻辑。

### Lines 66-90
```cpp
static Expected<uint64_t> getNewShfFlags(SectionFlag AllFlags,
                                         uint16_t EMachine) {
  uint64_t NewFlags = 0;
  if (AllFlags & SectionFlag::SecAlloc)
    NewFlags |= ELF::SHF_ALLOC;
  if (!(AllFlags & SectionFlag::SecReadonly))
    NewFlags |= ELF::SHF_WRITE;
  if (AllFlags & SectionFlag::SecCode)
    NewFlags |= ELF::SHF_EXECINSTR;
  if (AllFlags & SectionFlag::SecMerge)
    NewFlags |= ELF::SHF_MERGE;
  if (AllFlags & SectionFlag::SecStrings)
    NewFlags |= ELF::SHF_STRINGS;
  if (AllFlags & SectionFlag::SecExclude)
    NewFlags |= ELF::SHF_EXCLUDE;
  if (AllFlags & SectionFlag::SecLarge) {
    if (EMachine != EM_X86_64)
      return createStringError(errc::invalid_argument,
                               "section flag SHF_X86_64_LARGE can only be used "
                               "with x86_64 architecture");
    NewFlags |= ELF::SHF_X86_64_LARGE;
  }
  return NewFlags;
}

```
- **EN**: Implements logic around `getNewShfFlags`, `createStringError`; this block propagates recoverable errors through LLVM error utilities; applies object-format-specific rules.
- **CN**: 围绕 `getNewShfFlags`, `createStringError` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并应用目标文件格式专用规则。

### Lines 91-104
```cpp
static uint64_t getSectionFlagsPreserveMask(uint64_t OldFlags,
                                            uint64_t NewFlags,
                                            uint16_t EMachine) {
  // Preserve some flags which should not be dropped when setting flags.
  // Also, preserve anything OS/processor dependant.
  const uint64_t PreserveMask =
      (ELF::SHF_COMPRESSED | ELF::SHF_GROUP | ELF::SHF_LINK_ORDER |
       ELF::SHF_MASKOS | ELF::SHF_MASKPROC | ELF::SHF_TLS |
       ELF::SHF_INFO_LINK) &
      ~ELF::SHF_EXCLUDE &
      ~(EMachine == EM_X86_64 ? (uint64_t)ELF::SHF_X86_64_LARGE : 0UL);
  return (OldFlags & PreserveMask) | (NewFlags & ~PreserveMask);
}

```
- **EN**: Implements logic around `getSectionFlagsPreserveMask`, `~`; this block applies object-format-specific rules.
- **CN**: 围绕 `getSectionFlagsPreserveMask`, `~` 实现具体逻辑；该代码块应用目标文件格式专用规则。

### Lines 105-119
```cpp
static void setSectionType(SectionBase &Sec, uint64_t Type) {
  // If Sec's type is changed from SHT_NOBITS due to --set-section-flags,
  // Offset may not be aligned. Align it to max(Align, 1).
  if (Sec.Type == ELF::SHT_NOBITS && Type != ELF::SHT_NOBITS)
    Sec.Offset = alignTo(Sec.Offset, std::max(Sec.Align, uint64_t(1)));
  Sec.Type = Type;
}

static Error setSectionFlagsAndType(SectionBase &Sec, SectionFlag Flags,
                                    uint16_t EMachine) {
  Expected<uint64_t> NewFlags = getNewShfFlags(Flags, EMachine);
  if (!NewFlags)
    return NewFlags.takeError();
  Sec.Flags = getSectionFlagsPreserveMask(Sec.Flags, *NewFlags, EMachine);

```
- **EN**: Implements logic around `setSectionType`, `alignTo`, `setSectionFlagsAndType`, `getNewShfFlags`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities; applies object-format-specific rules.
- **CN**: 围绕 `setSectionType`, `alignTo`, `setSectionFlagsAndType`, `getNewShfFlags`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并应用目标文件格式专用规则。

### Lines 120-143
```cpp
  // In GNU objcopy, certain flags promote SHT_NOBITS to SHT_PROGBITS. This rule
  // may promote more non-ALLOC sections than GNU objcopy, but it is fine as
  // non-ALLOC SHT_NOBITS sections do not make much sense.
  if (Sec.Type == SHT_NOBITS &&
      (!(Sec.Flags & ELF::SHF_ALLOC) ||
       Flags & (SectionFlag::SecContents | SectionFlag::SecLoad)))
    setSectionType(Sec, ELF::SHT_PROGBITS);

  return Error::success();
}

static ElfType getOutputElfType(const Binary &Bin) {
  // Infer output ELF type from the input ELF object
  if (isa<ELFObjectFile<ELF32LE>>(Bin))
    return ELFT_ELF32LE;
  if (isa<ELFObjectFile<ELF64LE>>(Bin))
    return ELFT_ELF64LE;
  if (isa<ELFObjectFile<ELF32BE>>(Bin))
    return ELFT_ELF32BE;
  if (isa<ELFObjectFile<ELF64BE>>(Bin))
    return ELFT_ELF64BE;
  llvm_unreachable("Invalid ELFType");
}

```
- **EN**: Implements logic around `setSectionType`, `success`, `getOutputElfType`, `isa`, and 1 more symbols; this block applies object-format-specific rules.
- **CN**: 围绕 `setSectionType`, `success`, `getOutputElfType`, `isa`, and 1 more symbols 实现具体逻辑；该代码块应用目标文件格式专用规则。

### Lines 144-171
```cpp
static ElfType getOutputElfType(const MachineInfo &MI) {
  // Infer output ELF type from the binary arch specified
  if (MI.Is64Bit)
    return MI.IsLittleEndian ? ELFT_ELF64LE : ELFT_ELF64BE;
  else
    return MI.IsLittleEndian ? ELFT_ELF32LE : ELFT_ELF32BE;
}

static std::unique_ptr<Writer> createELFWriter(const CommonConfig &Config,
                                               Object &Obj, raw_ostream &Out,
                                               ElfType OutputElfType) {
  // Depending on the initial ELFT and OutputFormat we need a different Writer.
  switch (OutputElfType) {
  case ELFT_ELF32LE:
    return std::make_unique<ELFWriter<ELF32LE>>(Obj, Out, !Config.StripSections,
                                                Config.OnlyKeepDebug);
  case ELFT_ELF64LE:
    return std::make_unique<ELFWriter<ELF64LE>>(Obj, Out, !Config.StripSections,
                                                Config.OnlyKeepDebug);
  case ELFT_ELF32BE:
    return std::make_unique<ELFWriter<ELF32BE>>(Obj, Out, !Config.StripSections,
                                                Config.OnlyKeepDebug);
  case ELFT_ELF64BE:
    return std::make_unique<ELFWriter<ELF64BE>>(Obj, Out, !Config.StripSections,
                                                Config.OnlyKeepDebug);
  }
  llvm_unreachable("Invalid output format");
}
```
- **EN**: Implements logic around `getOutputElfType`, `createELFWriter`, `make_unique`, `llvm_unreachable`; this block uses `switch`-style dispatch; emits or serializes data to an external representation; applies object-format-specific rules.
- **CN**: 围绕 `getOutputElfType`, `createELFWriter`, `make_unique`, `llvm_unreachable` 实现具体逻辑；该代码块使用 `switch` 风格分派，并把数据输出或序列化为外部表示，并应用目标文件格式专用规则。

### Lines 172-187
```cpp

static std::unique_ptr<Writer> createWriter(const CommonConfig &Config,
                                            Object &Obj, raw_ostream &Out,
                                            ElfType OutputElfType) {
  switch (Config.OutputFormat) {
  case FileFormat::Binary:
    return std::make_unique<BinaryWriter>(Obj, Out, Config);
  case FileFormat::IHex:
    return std::make_unique<IHexWriter>(Obj, Out, Config.OutputFilename);
  case FileFormat::SREC:
    return std::make_unique<SRECWriter>(Obj, Out, Config.OutputFilename);
  default:
    return createELFWriter(Config, Obj, Out, OutputElfType);
  }
}

```
- **EN**: Implements logic around `createWriter`, `make_unique`, `createELFWriter`; this block uses `switch`-style dispatch; emits or serializes data to an external representation.
- **CN**: 围绕 `createWriter`, `make_unique`, `createELFWriter` 实现具体逻辑；该代码块使用 `switch` 风格分派，并把数据输出或序列化为外部表示。

### Lines 188-207
```cpp
static Error dumpSectionToFile(StringRef SecName, StringRef Filename,
                               StringRef InputFilename, Object &Obj) {
  for (auto &Sec : Obj.sections()) {
    if (Sec.Name == SecName) {
      if (Sec.Type == SHT_NOBITS)
        return createFileError(InputFilename, object_error::parse_failed,
                               "cannot dump section '%s': it has no contents",
                               SecName.str().c_str());
      Expected<std::unique_ptr<FileOutputBuffer>> BufferOrErr =
          FileOutputBuffer::create(Filename, Sec.OriginalData.size());
      if (!BufferOrErr)
        return createFileError(Filename, BufferOrErr.takeError());
      std::unique_ptr<FileOutputBuffer> Buf = std::move(*BufferOrErr);
      llvm::copy(Sec.OriginalData, Buf->getBufferStart());
      if (Error E = Buf->commit())
        return createFileError(Filename, std::move(E));
      return Error::success();
    }
  }

```
- **EN**: Implements logic around `dumpSectionToFile`, `sections`, `createFileError`, `str`, and 5 more symbols; this block propagates recoverable errors through LLVM error utilities; parses or classifies structured input.
- **CN**: 围绕 `dumpSectionToFile`, `sections`, `createFileError`, `str`, and 5 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并解析或分类结构化输入。

### Lines 208-233
```cpp
  return createFileError(InputFilename, object_error::parse_failed,
                         "section '%s' not found", SecName.str().c_str());
}

Error Object::compressOrDecompressSections(const CommonConfig &Config) {
  // Build a list of sections we are going to replace.
  // We can't call `addSection` while iterating over sections,
  // because it would mutate the sections array.
  SmallVector<std::pair<SectionBase *, std::function<SectionBase *()>>, 0>
      ToReplace;
  for (SectionBase &Sec : sections()) {
    std::optional<DebugCompressionType> CType;
    for (auto &[Matcher, T] : Config.compressSections)
      if (Matcher.matches(Sec.Name))
        CType = T;
    // Handle --compress-debug-sections and --decompress-debug-sections, which
    // apply to non-ALLOC debug sections.
    if (!(Sec.Flags & SHF_ALLOC) && StringRef(Sec.Name).starts_with(".debug")) {
      if (Config.CompressionType != DebugCompressionType::None)
        CType = Config.CompressionType;
      else if (Config.DecompressDebugSections)
        CType = DebugCompressionType::None;
    }
    if (!CType)
      continue;

```
- **EN**: Implements logic around `createFileError`, `str`, `compressOrDecompressSections`, `sections`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities; parses or classifies structured input.
- **CN**: 围绕 `createFileError`, `str`, `compressOrDecompressSections`, `sections`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并解析或分类结构化输入。

### Lines 234-251
```cpp
    if (Sec.ParentSegment)
      return createStringError(
          errc::invalid_argument,
          "section '" + Sec.Name +
              "' within a segment cannot be (de)compressed");

    if (auto *CS = dyn_cast<CompressedSection>(&Sec)) {
      if (*CType == DebugCompressionType::None)
        ToReplace.emplace_back(
            &Sec, [=] { return &addSection<DecompressedSection>(*CS); });
    } else if (*CType != DebugCompressionType::None) {
      ToReplace.emplace_back(&Sec, [=, S = &Sec] {
        return &addSection<CompressedSection>(
            CompressedSection(*S, *CType, Is64Bits));
      });
    }
  }

```
- **EN**: Implements logic around `createStringError`, `be`, `dyn_cast`, `emplace_back`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `createStringError`, `be`, `dyn_cast`, `emplace_back`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 252-267
```cpp
  DenseMap<SectionBase *, SectionBase *> FromTo;
  for (auto [S, Func] : ToReplace)
    FromTo[S] = Func();
  return replaceSections(FromTo);
}

static bool isAArch64MappingSymbol(const Symbol &Sym) {
  if (Sym.Binding != STB_LOCAL || Sym.Type != STT_NOTYPE ||
      Sym.getShndx() == SHN_UNDEF)
    return false;
  StringRef Name = Sym.Name;
  if (!Name.consume_front("$x") && !Name.consume_front("$d"))
    return false;
  return Name.empty() || Name.starts_with(".");
}

```
- **EN**: Implements logic around `Func`, `replaceSections`, `isAArch64MappingSymbol`, `getShndx`, and 2 more symbols; this block parses or classifies structured input.
- **CN**: 围绕 `Func`, `replaceSections`, `isAArch64MappingSymbol`, `getShndx`, and 2 more symbols 实现具体逻辑；该代码块解析或分类结构化输入。

### Lines 268-292
```cpp
static bool isArmMappingSymbol(const Symbol &Sym) {
  if (Sym.Binding != STB_LOCAL || Sym.Type != STT_NOTYPE ||
      Sym.getShndx() == SHN_UNDEF)
    return false;
  StringRef Name = Sym.Name;
  if (!Name.consume_front("$a") && !Name.consume_front("$d") &&
      !Name.consume_front("$t"))
    return false;
  return Name.empty() || Name.starts_with(".");
}

// Check if the symbol should be preserved because it is required by ABI.
static bool isRequiredByABISymbol(const Object &Obj, const Symbol &Sym) {
  switch (Obj.Machine) {
  case EM_AARCH64:
    // Mapping symbols should be preserved for a relocatable object file.
    return Obj.isRelocatable() && isAArch64MappingSymbol(Sym);
  case EM_ARM:
    // Mapping symbols should be preserved for a relocatable object file.
    return Obj.isRelocatable() && isArmMappingSymbol(Sym);
  default:
    return false;
  }
}

```
- **EN**: Implements logic around `isArmMappingSymbol`, `getShndx`, `consume_front`, `empty`, and 2 more symbols; this block uses `switch`-style dispatch; parses or classifies structured input.
- **CN**: 围绕 `isArmMappingSymbol`, `getShndx`, `consume_front`, `empty`, and 2 more symbols 实现具体逻辑；该代码块使用 `switch` 风格分派，并解析或分类结构化输入。

### Lines 293-309
```cpp
static bool isUnneededSymbol(const Symbol &Sym) {
  return !Sym.Referenced &&
         (Sym.Binding == STB_LOCAL || Sym.getShndx() == SHN_UNDEF) &&
         Sym.Type != STT_SECTION;
}

static Error updateAndRemoveSymbols(const CommonConfig &Config,
                                    const ELFConfig &ELFConfig, Object &Obj) {
  // TODO: update or remove symbols only if there is an option that affects
  // them.
  if (!Obj.SymbolTable)
    return Error::success();

  Obj.SymbolTable->updateSymbols([&](Symbol &Sym) {
    if (Config.SymbolsToSkip.matches(Sym.Name))
      return;

```
- **EN**: Implements logic around `isUnneededSymbol`, `getShndx`, `updateAndRemoveSymbols`, `success`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `isUnneededSymbol`, `getShndx`, `updateAndRemoveSymbols`, `success`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 310-336
```cpp
    // Common and undefined symbols don't make sense as local symbols, and can
    // even cause crashes if we localize those, so skip them.
    if (!Sym.isCommon() && Sym.getShndx() != SHN_UNDEF &&
        ((ELFConfig.LocalizeHidden &&
          (Sym.Visibility == STV_HIDDEN || Sym.Visibility == STV_INTERNAL)) ||
         Config.SymbolsToLocalize.matches(Sym.Name)))
      Sym.Binding = STB_LOCAL;

    for (auto &[Matcher, Visibility] : ELFConfig.SymbolsToSetVisibility)
      if (Matcher.matches(Sym.Name))
        Sym.Visibility = Visibility;

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
    if (!Config.SymbolsToKeepGlobal.empty() &&
        !Config.SymbolsToKeepGlobal.matches(Sym.Name) &&
        Sym.getShndx() != SHN_UNDEF)
      Sym.Binding = STB_LOCAL;

```
- **EN**: Implements logic around `isCommon`, `matches`, `empty`, `getShndx`.
- **CN**: 围绕 `isCommon`, `matches`, `empty`, `getShndx` 实现具体逻辑。

### Lines 337-352
```cpp
    if (Config.SymbolsToGlobalize.matches(Sym.Name) &&
        Sym.getShndx() != SHN_UNDEF)
      Sym.Binding = STB_GLOBAL;

    // SymbolsToWeaken applies to both STB_GLOBAL and STB_GNU_UNIQUE.
    if (Config.SymbolsToWeaken.matches(Sym.Name) && Sym.Binding != STB_LOCAL)
      Sym.Binding = STB_WEAK;

    if (Config.Weaken && Sym.Binding != STB_LOCAL &&
        Sym.getShndx() != SHN_UNDEF)
      Sym.Binding = STB_WEAK;

    const auto I = Config.SymbolsToRename.find(Sym.Name);
    if (I != Config.SymbolsToRename.end())
      Sym.Name = std::string(I->getValue());

```
- **EN**: Implements logic around `matches`, `getShndx`, `find`, `end`, and 1 more symbols.
- **CN**: 围绕 `matches`, `getShndx`, `find`, `end`, and 1 more symbols 实现具体逻辑。

### Lines 353-369
```cpp
    if (!Config.SymbolsPrefixRemove.empty() && Sym.Type != STT_SECTION)
      if (StringRef(Sym.Name).starts_with(Config.SymbolsPrefixRemove))
        Sym.Name = Sym.Name.substr(Config.SymbolsPrefixRemove.size());

    if (!Config.SymbolsPrefix.empty() && Sym.Type != STT_SECTION)
      Sym.Name = (Config.SymbolsPrefix + Sym.Name).str();
  });

  // The purpose of this loop is to mark symbols referenced by sections
  // (like GroupSection or RelocationSection). This way, we know which
  // symbols are still 'needed' and which are not.
  if (Config.StripUnneeded || !Config.UnneededSymbolsToRemove.empty() ||
      !Config.OnlySection.empty() || Config.DiscardMode != DiscardType::None) {
    for (SectionBase &Sec : Obj.sections())
      Sec.markSymbols();
  }

```
- **EN**: Implements logic around `empty`, `StringRef`, `substr`, `str`, and 2 more symbols.
- **CN**: 围绕 `empty`, `StringRef`, `substr`, `str`, and 2 more symbols 实现具体逻辑。

### Lines 370-383
```cpp
  auto RemoveSymbolsPred = [&](const Symbol &Sym) {
    if (Config.SymbolsToKeep.matches(Sym.Name) ||
        (ELFConfig.KeepFileSymbols && Sym.Type == STT_FILE))
      return false;

    if (Config.SymbolsToRemove.matches(Sym.Name))
      return true;

    if (Config.StripAll || Config.StripAllGNU)
      return true;

    if (isRequiredByABISymbol(Obj, Sym))
      return false;

```
- **EN**: Implements logic around `matches`, `isRequiredByABISymbol`.
- **CN**: 围绕 `matches`, `isRequiredByABISymbol` 实现具体逻辑。

### Lines 384-404
```cpp
    if (Config.StripDebug && Sym.Type == STT_FILE)
      return true;

    if ((Config.StripUnneeded ||
         Config.UnneededSymbolsToRemove.matches(Sym.Name)) &&
        (!Obj.isRelocatable() || isUnneededSymbol(Sym)))
      return true;

    if (!Sym.Referenced) {
      if ((Config.DiscardMode == DiscardType::All ||
           (Config.DiscardMode == DiscardType::Locals &&
            StringRef(Sym.Name).starts_with(".L"))) &&
          Sym.Binding == STB_LOCAL && Sym.getShndx() != SHN_UNDEF &&
          Sym.Type != STT_FILE && Sym.Type != STT_SECTION)
        return true;
      // We want to remove undefined symbols if all references have been
      // stripped.
      if (!Config.OnlySection.empty() && Sym.getShndx() == SHN_UNDEF)
        return true;
    }

```
- **EN**: Implements logic around `matches`, `isRelocatable`, `StringRef`, `getShndx`, and 1 more symbols.
- **CN**: 围绕 `matches`, `isRelocatable`, `StringRef`, `getShndx`, and 1 more symbols 实现具体逻辑。

### Lines 405-421
```cpp
    return false;
  };

  return Obj.removeSymbols(RemoveSymbolsPred);
}

static Error replaceAndRemoveSections(const CommonConfig &Config,
                                      const ELFConfig &ELFConfig, Object &Obj) {
  SectionPred RemovePred = [](const SectionBase &) { return false; };

  // Removes:
  if (!Config.ToRemove.empty()) {
    RemovePred = [&Config](const SectionBase &Sec) {
      return Config.ToRemove.matches(Sec.Name);
    };
  }

```
- **EN**: Implements logic around `removeSymbols`, `replaceAndRemoveSections`, `empty`, `matches`; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `removeSymbols`, `replaceAndRemoveSections`, `empty`, `matches` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 422-449
```cpp
  if (Config.StripDWO)
    RemovePred = [RemovePred](const SectionBase &Sec) {
      return isDWOSection(Sec) || RemovePred(Sec);
    };

  if (Config.ExtractDWO)
    RemovePred = [RemovePred, &Obj](const SectionBase &Sec) {
      return onlyKeepDWOPred(Obj, Sec) || RemovePred(Sec);
    };

  if (Config.StripAllGNU)
    RemovePred = [RemovePred, &Obj](const SectionBase &Sec) {
      if (RemovePred(Sec))
        return true;
      if ((Sec.Flags & SHF_ALLOC) != 0)
        return false;
      if (&Sec == Obj.SectionNames)
        return false;
      switch (Sec.Type) {
      case SHT_SYMTAB:
      case SHT_REL:
      case SHT_RELA:
      case SHT_STRTAB:
        return true;
      }
      return isDebugSection(Sec);
    };

```
- **EN**: Implements logic around `isDWOSection`, `onlyKeepDWOPred`, `RemovePred`, `isDebugSection`; this block uses `switch`-style dispatch.
- **CN**: 围绕 `isDWOSection`, `onlyKeepDWOPred`, `RemovePred`, `isDebugSection` 实现具体逻辑；该代码块使用 `switch` 风格分派。

### Lines 450-470
```cpp
  if (Config.StripSections) {
    RemovePred = [RemovePred](const SectionBase &Sec) {
      return RemovePred(Sec) || Sec.ParentSegment == nullptr;
    };
  }

  if (Config.StripDebug || Config.StripUnneeded) {
    RemovePred = [RemovePred](const SectionBase &Sec) {
      return RemovePred(Sec) || isDebugSection(Sec);
    };
  }

  if (Config.StripNonAlloc)
    RemovePred = [RemovePred, &Obj](const SectionBase &Sec) {
      if (RemovePred(Sec))
        return true;
      if (&Sec == Obj.SectionNames)
        return false;
      return (Sec.Flags & SHF_ALLOC) == 0 && Sec.ParentSegment == nullptr;
    };

```
- **EN**: Implements logic around `RemovePred`.
- **CN**: 围绕 `RemovePred` 实现具体逻辑。

### Lines 471-491
```cpp
  if (Config.StripAll)
    RemovePred = [RemovePred, &Obj](const SectionBase &Sec) {
      if (RemovePred(Sec))
        return true;
      if (&Sec == Obj.SectionNames)
        return false;
      if (StringRef(Sec.Name).starts_with(".gnu.warning"))
        return false;
      if (StringRef(Sec.Name).starts_with(".gnu_debuglink"))
        return false;
      // We keep the .ARM.attribute section to maintain compatibility
      // with Debian derived distributions. This is a bug in their
      // patchset as documented here:
      // https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=943798
      if (Sec.Type == SHT_ARM_ATTRIBUTES)
        return false;
      if (Sec.ParentSegment != nullptr)
        return false;
      return (Sec.Flags & SHF_ALLOC) == 0;
    };

```
- **EN**: Implements logic around `RemovePred`, `StringRef`.
- **CN**: 围绕 `RemovePred`, `StringRef` 实现具体逻辑。

### Lines 492-508
```cpp
  if (Config.ExtractPartition || Config.ExtractMainPartition) {
    RemovePred = [RemovePred](const SectionBase &Sec) {
      if (RemovePred(Sec))
        return true;
      if (Sec.Type == SHT_LLVM_PART_EHDR || Sec.Type == SHT_LLVM_PART_PHDR)
        return true;
      return (Sec.Flags & SHF_ALLOC) != 0 && !Sec.ParentSegment;
    };
  }

  // Explicit copies:
  if (!Config.OnlySection.empty()) {
    RemovePred = [&Config, RemovePred, &Obj](const SectionBase &Sec) {
      // Explicitly keep these sections regardless of previous removes.
      if (Config.OnlySection.matches(Sec.Name))
        return false;

```
- **EN**: Implements logic around `RemovePred`, `empty`, `matches`.
- **CN**: 围绕 `RemovePred`, `empty`, `matches` 实现具体逻辑。

### Lines 509-524
```cpp
      // Allow all implicit removes.
      if (RemovePred(Sec))
        return true;

      // Keep special sections.
      if (Obj.SectionNames == &Sec)
        return false;
      if (Obj.SymbolTable == &Sec ||
          (Obj.SymbolTable && Obj.SymbolTable->getStrTab() == &Sec))
        return false;

      // Remove everything else.
      return true;
    };
  }

```
- **EN**: Implements logic around `RemovePred`, `getStrTab`.
- **CN**: 围绕 `RemovePred`, `getStrTab` 实现具体逻辑。

### Lines 525-548
```cpp
  if (!Config.KeepSection.empty()) {
    RemovePred = [&Config, RemovePred](const SectionBase &Sec) {
      // Explicitly keep these sections regardless of previous removes.
      if (Config.KeepSection.matches(Sec.Name))
        return false;
      // Otherwise defer to RemovePred.
      return RemovePred(Sec);
    };
  }

  // This has to be the last predicate assignment.
  // If the option --keep-symbol has been specified
  // and at least one of those symbols is present
  // (equivalently, the updated symbol table is not empty)
  // the symbol table and the string table should not be removed.
  if ((!Config.SymbolsToKeep.empty() || ELFConfig.KeepFileSymbols) &&
      Obj.SymbolTable && !Obj.SymbolTable->empty()) {
    RemovePred = [&Obj, RemovePred](const SectionBase &Sec) {
      if (&Sec == Obj.SymbolTable || &Sec == Obj.SymbolTable->getStrTab())
        return false;
      return RemovePred(Sec);
    };
  }

```
- **EN**: Implements logic around `empty`, `matches`, `RemovePred`, `getStrTab`.
- **CN**: 围绕 `empty`, `matches`, `RemovePred`, `getStrTab` 实现具体逻辑。

### Lines 549-563
```cpp
  if (Error E = Obj.removeSections(ELFConfig.AllowBrokenLinks, RemovePred))
    return E;

  if (Error E = Obj.compressOrDecompressSections(Config))
    return E;

  return Error::success();
}

// Add symbol to the Object symbol table with the specified properties.
static void addSymbol(Object &Obj, const NewSymbolInfo &SymInfo,
                      uint8_t DefaultVisibility) {
  SectionBase *Sec = Obj.findSection(SymInfo.SectionName);
  uint64_t Value = Sec ? Sec->Addr + SymInfo.Value : SymInfo.Value;

```
- **EN**: Implements logic around `removeSections`, `compressOrDecompressSections`, `success`, `addSymbol`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `removeSections`, `compressOrDecompressSections`, `success`, `addSymbol`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 564-591
```cpp
  uint8_t Bind = ELF::STB_GLOBAL;
  uint8_t Type = ELF::STT_NOTYPE;
  uint8_t Visibility = DefaultVisibility;

  for (SymbolFlag FlagValue : SymInfo.Flags)
    switch (FlagValue) {
    case SymbolFlag::Global:
      Bind = ELF::STB_GLOBAL;
      break;
    case SymbolFlag::Local:
      Bind = ELF::STB_LOCAL;
      break;
    case SymbolFlag::Weak:
      Bind = ELF::STB_WEAK;
      break;
    case SymbolFlag::Default:
      Visibility = ELF::STV_DEFAULT;
      break;
    case SymbolFlag::Hidden:
      Visibility = ELF::STV_HIDDEN;
      break;
    case SymbolFlag::Protected:
      Visibility = ELF::STV_PROTECTED;
      break;
    case SymbolFlag::File:
      Type = ELF::STT_FILE;
      break;
    case SymbolFlag::Section:
```
- **EN**: Contains supporting implementation details for the surrounding LLVM library component.
- **CN**: 包含周边 LLVM 库组件所需的辅助实现细节。

### Lines 592-606
```cpp
      Type = ELF::STT_SECTION;
      break;
    case SymbolFlag::Object:
      Type = ELF::STT_OBJECT;
      break;
    case SymbolFlag::Function:
      Type = ELF::STT_FUNC;
      break;
    case SymbolFlag::IndirectFunction:
      Type = ELF::STT_GNU_IFUNC;
      break;
    default: /* Other flag values are ignored for ELF. */
      break;
    };

```
- **EN**: Contains supporting implementation details for the surrounding LLVM library component.
- **CN**: 包含周边 LLVM 库组件所需的辅助实现细节。

### Lines 607-627
```cpp
  Obj.SymbolTable->addSymbol(
      SymInfo.SymbolName, Bind, Type, Sec, Value, Visibility,
      Sec ? (uint16_t)SYMBOL_SIMPLE_INDEX : (uint16_t)SHN_ABS, 0);
}

namespace {
struct RemoveNoteDetail {
  struct DeletedRange {
    uint64_t OldFrom;
    uint64_t OldTo;
  };

  template <class ELFT>
  static std::vector<DeletedRange>
  findNotesToRemove(ArrayRef<uint8_t> Data, size_t Align,
                    ArrayRef<RemoveNoteInfo> NotesToRemove);
  static std::vector<uint8_t> updateData(ArrayRef<uint8_t> OldData,
                                         ArrayRef<DeletedRange> ToRemove);
};
} // namespace

```
- **EN**: Introduces declarations for `RemoveNoteDetail`, `DeletedRange`, `ELFT`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `RemoveNoteDetail`, `DeletedRange`, `ELFT` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 628-653
```cpp
template <class ELFT>
std::vector<RemoveNoteDetail::DeletedRange>
RemoveNoteDetail::findNotesToRemove(ArrayRef<uint8_t> Data, size_t Align,
                                    ArrayRef<RemoveNoteInfo> NotesToRemove) {
  using Elf_Nhdr = typename ELFT::Nhdr;
  using Elf_Note = typename ELFT::Note;
  std::vector<DeletedRange> ToRemove;
  uint64_t CurPos = 0;
  while (CurPos + sizeof(Elf_Nhdr) <= Data.size()) {
    auto Nhdr = reinterpret_cast<const Elf_Nhdr *>(Data.data() + CurPos);
    size_t FullSize = Nhdr->getSize(Align);
    if (CurPos + FullSize > Data.size())
      break;
    Elf_Note Note(*Nhdr);
    bool ShouldRemove =
        llvm::any_of(NotesToRemove, [&Note](const RemoveNoteInfo &NoteInfo) {
          return NoteInfo.TypeId == Note.getType() &&
                 (NoteInfo.Name.empty() || NoteInfo.Name == Note.getName());
        });
    if (ShouldRemove)
      ToRemove.push_back({CurPos, CurPos + FullSize});
    CurPos += FullSize;
  }
  return ToRemove;
}

```
- **EN**: Introduces declarations for `ELFT`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ELFT` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 654-673
```cpp
std::vector<uint8_t>
RemoveNoteDetail::updateData(ArrayRef<uint8_t> OldData,
                             ArrayRef<DeletedRange> ToRemove) {
  std::vector<uint8_t> NewData;
  NewData.reserve(OldData.size());
  uint64_t CurPos = 0;
  for (const DeletedRange &RemRange : ToRemove) {
    if (CurPos < RemRange.OldFrom) {
      auto Slice = OldData.slice(CurPos, RemRange.OldFrom - CurPos);
      llvm::append_range(NewData, Slice);
    }
    CurPos = RemRange.OldTo;
  }
  if (CurPos < OldData.size()) {
    auto Slice = OldData.slice(CurPos);
    llvm::append_range(NewData, Slice);
  }
  return NewData;
}

```
- **EN**: Implements logic around `updateData`, `reserve`, `slice`, `append_range`, and 1 more symbols.
- **CN**: 围绕 `updateData`, `reserve`, `slice`, `append_range`, and 1 more symbols 实现具体逻辑。

### Lines 674-701
```cpp
static Error removeNotes(Object &Obj, endianness Endianness,
                         ArrayRef<RemoveNoteInfo> NotesToRemove,
                         function_ref<Error(Error)> ErrorCallback) {
  // TODO: Support note segments.
  if (ErrorCallback) {
    for (Segment &Seg : Obj.segments()) {
      if (Seg.Type == PT_NOTE) {
        if (Error E = ErrorCallback(createStringError(
                errc::not_supported, "note segments are not supported")))
          return E;
        break;
      }
    }
  }
  for (auto &Sec : Obj.sections()) {
    if (Sec.Type != SHT_NOTE || !Sec.hasContents())
      continue;
    // TODO: Support note sections in segments.
    if (Sec.ParentSegment) {
      if (ErrorCallback)
        if (Error E = ErrorCallback(createStringError(
                errc::not_supported,
                "cannot remove note(s) from " + Sec.Name +
                    ": sections in segments are not supported")))
          return E;
      continue;
    }
    ArrayRef<uint8_t> OldData = Sec.getContents();
```
- **EN**: Implements logic around `removeNotes`, `function_ref`, `segments`, `ErrorCallback`, and 4 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `removeNotes`, `function_ref`, `segments`, `ErrorCallback`, and 4 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 702-718
```cpp
    size_t Align = std::max<size_t>(4, Sec.Align);
    // Note: notes for both 32-bit and 64-bit ELF files use 4-byte words in the
    // header, so the parsers are the same.
    auto ToRemove = (Endianness == endianness::little)
                        ? RemoveNoteDetail::findNotesToRemove<ELF64LE>(
                              OldData, Align, NotesToRemove)
                        : RemoveNoteDetail::findNotesToRemove<ELF64BE>(
                              OldData, Align, NotesToRemove);
    if (!ToRemove.empty()) {
      if (Error E = Obj.updateSectionData(
              Sec, RemoveNoteDetail::updateData(OldData, ToRemove)))
        return E;
    }
  }
  return Error::success();
}

```
- **EN**: Implements logic around `max`, `findNotesToRemove`, `empty`, `updateSectionData`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities; parses or classifies structured input; applies object-format-specific rules.
- **CN**: 围绕 `max`, `findNotesToRemove`, `empty`, `updateSectionData`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并解析或分类结构化输入，并应用目标文件格式专用规则。

### Lines 719-736
```cpp
static Error
handleUserSection(const NewSectionInfo &NewSection,
                  function_ref<Error(StringRef, ArrayRef<uint8_t>)> F) {
  ArrayRef<uint8_t> Data(reinterpret_cast<const uint8_t *>(
                             NewSection.SectionData->getBufferStart()),
                         NewSection.SectionData->getBufferSize());
  return F(NewSection.SectionName, Data);
}

static Error verifyNoteSection(StringRef Name, endianness Endianness,
                               ArrayRef<uint8_t> Data) {
  // An ELF note has the following structure:
  // Name Size: 4 bytes (integer)
  // Desc Size: 4 bytes (integer)
  // Type     : 4 bytes
  // Name     : variable size, padded to a 4 byte boundary
  // Desc     : variable size, padded to a 4 byte boundary

```
- **EN**: Implements logic around `handleUserSection`, `function_ref`, `Data`, `getBufferStart`, and 3 more symbols; this block propagates recoverable errors through LLVM error utilities; applies object-format-specific rules.
- **CN**: 围绕 `handleUserSection`, `function_ref`, `Data`, `getBufferStart`, and 3 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并应用目标文件格式专用规则。

### Lines 737-754
```cpp
  if (Data.empty())
    return Error::success();

  if (Data.size() < 12) {
    std::string msg;
    raw_string_ostream(msg)
        << Name << " data must be either empty or at least 12 bytes long";
    return createStringError(errc::invalid_argument, msg);
  }
  if (Data.size() % 4 != 0) {
    std::string msg;
    raw_string_ostream(msg)
        << Name << " data size must be a  multiple of 4 bytes";
    return createStringError(errc::invalid_argument, msg);
  }
  ArrayRef<uint8_t> NameSize = Data.slice(0, 4);
  ArrayRef<uint8_t> DescSize = Data.slice(4, 4);

```
- **EN**: Implements logic around `empty`, `success`, `size`, `raw_string_ostream`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `empty`, `success`, `size`, `raw_string_ostream`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 755-772
```cpp
  uint32_t NameSizeValue = support::endian::read32(NameSize.data(), Endianness);
  uint32_t DescSizeValue = support::endian::read32(DescSize.data(), Endianness);

  uint64_t ExpectedDataSize =
      /*NameSize=*/4 + /*DescSize=*/4 + /*Type=*/4 +
      /*Name=*/alignTo(NameSizeValue, 4) +
      /*Desc=*/alignTo(DescSizeValue, 4);
  uint64_t ActualDataSize = Data.size();
  if (ActualDataSize != ExpectedDataSize) {
    std::string msg;
    raw_string_ostream(msg)
        << Name
        << " data size is incompatible with the content of "
           "the name and description size fields:"
        << " expecting " << ExpectedDataSize << ", found " << ActualDataSize;
    return createStringError(errc::invalid_argument, msg);
  }

```
- **EN**: Implements logic around `read32`, `alignTo`, `size`, `raw_string_ostream`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `read32`, `alignTo`, `size`, `raw_string_ostream`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 773-789
```cpp
  return Error::success();
}

// This function handles the high level operations of GNU objcopy including
// handling command line options. It's important to outline certain properties
// we expect to hold of the command line operations. Any operation that "keeps"
// should keep regardless of a remove. Additionally any removal should respect
// any previous removals. Lastly whether or not something is removed shouldn't
// depend a) on the order the options occur in or b) on some opaque priority
// system. The only priority is that keeps/copies overrule removes.
static Error handleArgs(const CommonConfig &Config, const ELFConfig &ELFConfig,
                        ElfType OutputElfType, Object &Obj) {
  if (Config.OutputArch) {
    Obj.Machine = Config.OutputArch->EMachine;
    Obj.OSABI = Config.OutputArch->OSABI;
  }

```
- **EN**: Implements logic around `success`, `handleArgs`; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `success`, `handleArgs` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 790-805
```cpp
  if (!Config.SplitDWO.empty() && Config.ExtractDWO) {
    return Obj.removeSections(
        ELFConfig.AllowBrokenLinks,
        [&Obj](const SectionBase &Sec) { return onlyKeepDWOPred(Obj, Sec); });
  }

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
- **EN**: Implements logic around `empty`, `removeSections`, `onlyKeepDWOPred`, `tie`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `empty`, `removeSections`, `onlyKeepDWOPred`, `tie`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 806-823
```cpp
  // It is important to remove the sections first. For example, we want to
  // remove the relocation sections before removing the symbols. That allows
  // us to avoid reporting the inappropriate errors about removing symbols
  // named in relocations.
  if (Error E = replaceAndRemoveSections(Config, ELFConfig, Obj))
    return createFileError(Config.InputFilename, std::move(E));

  if (Error E = updateAndRemoveSymbols(Config, ELFConfig, Obj))
    return createFileError(Config.InputFilename, std::move(E));

  if (!Config.SetSectionAlignment.empty()) {
    for (SectionBase &Sec : Obj.sections()) {
      auto I = Config.SetSectionAlignment.find(Sec.Name);
      if (I != Config.SetSectionAlignment.end())
        Sec.Align = I->second;
    }
  }

```
- **EN**: Implements logic around `replaceAndRemoveSections`, `createFileError`, `updateAndRemoveSymbols`, `empty`, and 3 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `replaceAndRemoveSections`, `createFileError`, `updateAndRemoveSymbols`, `empty`, and 3 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 824-850
```cpp
  if (Config.ChangeSectionLMAValAll != 0) {
    for (Segment &Seg : Obj.segments()) {
      if (Seg.MemSize > 0) {
        if (Config.ChangeSectionLMAValAll > 0 &&
            Seg.PAddr > std::numeric_limits<uint64_t>::max() -
                            Config.ChangeSectionLMAValAll) {
          return createFileError(
              Config.InputFilename, errc::invalid_argument,
              "address 0x" + Twine::utohexstr(Seg.PAddr) +
                  " cannot be increased by 0x" +
                  Twine::utohexstr(Config.ChangeSectionLMAValAll) +
                  ". The result would overflow");
        } else if (Config.ChangeSectionLMAValAll < 0 &&
                   Seg.PAddr < std::numeric_limits<uint64_t>::min() -
                                   Config.ChangeSectionLMAValAll) {
          return createFileError(
              Config.InputFilename, errc::invalid_argument,
              "address 0x" + Twine::utohexstr(Seg.PAddr) +
                  " cannot be decreased by 0x" +
                  Twine::utohexstr(std::abs(Config.ChangeSectionLMAValAll)) +
                  ". The result would underflow");
        }
        Seg.PAddr += Config.ChangeSectionLMAValAll;
      }
    }
  }

```
- **EN**: Implements logic around `segments`, `max`, `createFileError`, `utohexstr`, and 1 more symbols.
- **CN**: 围绕 `segments`, `max`, `createFileError`, `utohexstr`, and 1 more symbols 实现具体逻辑。

### Lines 851-878
```cpp
  if (!Config.ChangeSectionAddress.empty()) {
    if (Obj.Type != ELF::ET_REL)
      return createFileError(
          Config.InputFilename, object_error::invalid_file_type,
          "cannot change section address in a non-relocatable file");
    StringMap<AddressUpdate> SectionsToUpdateAddress;
    for (const SectionPatternAddressUpdate &PatternUpdate :
         reverse(Config.ChangeSectionAddress)) {
      for (SectionBase &Sec : Obj.sections()) {
        if (PatternUpdate.SectionPattern.matches(Sec.Name) &&
            SectionsToUpdateAddress.try_emplace(Sec.Name, PatternUpdate.Update)
                .second) {
          if (PatternUpdate.Update.Kind == AdjustKind::Subtract &&
              Sec.Addr < PatternUpdate.Update.Value) {
            return createFileError(
                Config.InputFilename, errc::invalid_argument,
                "address 0x" + Twine::utohexstr(Sec.Addr) +
                    " cannot be decreased by 0x" +
                    Twine::utohexstr(PatternUpdate.Update.Value) +
                    ". The result would underflow");
          }
          if (PatternUpdate.Update.Kind == AdjustKind::Add &&
              Sec.Addr > std::numeric_limits<uint64_t>::max() -
                             PatternUpdate.Update.Value) {
            return createFileError(
                Config.InputFilename, errc::invalid_argument,
                "address 0x" + Twine::utohexstr(Sec.Addr) +
                    " cannot be increased by 0x" +
```
- **EN**: Implements logic around `empty`, `createFileError`, `reverse`, `sections`, and 4 more symbols; this block applies object-format-specific rules.
- **CN**: 围绕 `empty`, `createFileError`, `reverse`, `sections`, and 4 more symbols 实现具体逻辑；该代码块应用目标文件格式专用规则。

### Lines 879-898
```cpp
                    Twine::utohexstr(PatternUpdate.Update.Value) +
                    ". The result would overflow");
          }

          switch (PatternUpdate.Update.Kind) {
          case (AdjustKind::Set):
            Sec.Addr = PatternUpdate.Update.Value;
            break;
          case (AdjustKind::Subtract):
            Sec.Addr -= PatternUpdate.Update.Value;
            break;
          case (AdjustKind::Add):
            Sec.Addr += PatternUpdate.Update.Value;
            break;
          }
        }
      }
    }
  }

```
- **EN**: Implements logic around `utohexstr`; this block uses `switch`-style dispatch.
- **CN**: 围绕 `utohexstr` 实现具体逻辑；该代码块使用 `switch` 风格分派。

### Lines 899-913
```cpp
  if (Config.OnlyKeepDebug)
    for (auto &Sec : Obj.sections())
      if (Sec.Flags & SHF_ALLOC && Sec.Type != SHT_NOTE)
        Sec.Type = SHT_NOBITS;

  endianness E = OutputElfType == ELFT_ELF32LE || OutputElfType == ELFT_ELF64LE
                     ? endianness::little
                     : endianness::big;

  if (!ELFConfig.NotesToRemove.empty()) {
    if (Error Err =
            removeNotes(Obj, E, ELFConfig.NotesToRemove, Config.ErrorCallback))
      return createFileError(Config.InputFilename, std::move(Err));
  }

```
- **EN**: Implements logic around `sections`, `empty`, `removeNotes`, `createFileError`; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `sections`, `empty`, `removeNotes`, `createFileError` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 914-928
```cpp
  for (const NewSectionInfo &AddedSection : Config.AddSection) {
    auto AddSection = [&](StringRef Name, ArrayRef<uint8_t> Data) -> Error {
      OwnedDataSection &NewSection =
          Obj.addSection<OwnedDataSection>(Name, Data);
      if (Name.starts_with(".note") && Name != ".note.GNU-stack") {
        NewSection.Type = SHT_NOTE;
        if (ELFConfig.VerifyNoteSections)
          return verifyNoteSection(Name, E, Data);
      }
      return Error::success();
    };
    if (Error E = handleUserSection(AddedSection, AddSection))
      return createFileError(Config.InputFilename, std::move(E));
  }

```
- **EN**: Implements logic around `addSection`, `starts_with`, `verifyNoteSection`, `success`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `addSection`, `starts_with`, `verifyNoteSection`, `success`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 929-946
```cpp
  for (const NewSectionInfo &NewSection : Config.UpdateSection) {
    auto UpdateSection = [&](StringRef Name, ArrayRef<uint8_t> Data) {
      return Obj.updateSection(Name, Data);
    };
    if (Error E = handleUserSection(NewSection, UpdateSection))
      return createFileError(Config.InputFilename, std::move(E));
  }

  if (!Config.AddGnuDebugLink.empty())
    Obj.addSection<GnuDebugLinkSection>(Config.AddGnuDebugLink,
                                        Config.GnuDebugLinkCRC32);

  // If the symbol table was previously removed, we need to create a new one
  // before adding new symbols.
  if (!Obj.SymbolTable && !Config.SymbolsToAdd.empty())
    if (Error E = Obj.addNewSymbolTable())
      return createFileError(Config.InputFilename, std::move(E));

```
- **EN**: Implements logic around `updateSection`, `handleUserSection`, `createFileError`, `empty`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `updateSection`, `handleUserSection`, `createFileError`, `empty`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 947-964
```cpp
  for (const NewSymbolInfo &SI : Config.SymbolsToAdd)
    addSymbol(Obj, SI, ELFConfig.NewSymbolVisibility);

  // --set-section-{flags,type} work with sections added by --add-section.
  if (!Config.SetSectionFlags.empty() || !Config.SetSectionType.empty()) {
    for (auto &Sec : Obj.sections()) {
      const auto Iter = Config.SetSectionFlags.find(Sec.Name);
      if (Iter != Config.SetSectionFlags.end()) {
        const SectionFlagsUpdate &SFU = Iter->second;
        if (Error E = setSectionFlagsAndType(Sec, SFU.NewFlags, Obj.Machine))
          return createFileError(Config.InputFilename, std::move(E));
      }
      auto It2 = Config.SetSectionType.find(Sec.Name);
      if (It2 != Config.SetSectionType.end())
        setSectionType(Sec, It2->second);
    }
  }

```
- **EN**: Implements logic around `addSymbol`, `empty`, `sections`, `find`, and 4 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `addSymbol`, `empty`, `sections`, `find`, and 4 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 965-988
```cpp
  if (!Config.SectionsToRename.empty()) {
    std::vector<RelocationSectionBase *> RelocSections;
    DenseSet<SectionBase *> RenamedSections;
    for (SectionBase &Sec : Obj.sections()) {
      auto *RelocSec = dyn_cast<RelocationSectionBase>(&Sec);
      const auto Iter = Config.SectionsToRename.find(Sec.Name);
      if (Iter != Config.SectionsToRename.end()) {
        const SectionRename &SR = Iter->second;
        Sec.Name = std::string(SR.NewName);
        if (SR.NewFlags) {
          if (Error E = setSectionFlagsAndType(Sec, *SR.NewFlags, Obj.Machine))
            return createFileError(Config.InputFilename, std::move(E));
        }
        RenamedSections.insert(&Sec);
      } else if (RelocSec && !(Sec.Flags & SHF_ALLOC))
        // Postpone processing relocation sections which are not specified in
        // their explicit '--rename-section' commands until after their target
        // sections are renamed.
        // Dynamic relocation sections (i.e. ones with SHF_ALLOC) should be
        // renamed only explicitly. Otherwise, renaming, for example, '.got.plt'
        // would affect '.rela.plt', which is not desirable.
        RelocSections.push_back(RelocSec);
    }

```
- **EN**: Implements logic around `empty`, `sections`, `dyn_cast`, `find`, and 6 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `empty`, `sections`, `dyn_cast`, `find`, and 6 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 989-1016
```cpp
    // Rename relocation sections according to their target sections.
    for (RelocationSectionBase *RelocSec : RelocSections) {
      auto Iter = RenamedSections.find(RelocSec->getSection());
      if (Iter != RenamedSections.end())
        RelocSec->Name = (RelocSec->getNamePrefix() + (*Iter)->Name).str();
    }
  }

  // Add a prefix to allocated sections and their relocation sections. This
  // should be done after renaming the section by Config.SectionToRename to
  // imitate the GNU objcopy behavior.
  if (!Config.AllocSectionsPrefix.empty()) {
    DenseSet<SectionBase *> PrefixedSections;
    for (SectionBase &Sec : Obj.sections()) {
      if (Sec.Flags & SHF_ALLOC) {
        Sec.Name = (Config.AllocSectionsPrefix + Sec.Name).str();
        PrefixedSections.insert(&Sec);
      } else if (auto *RelocSec = dyn_cast<RelocationSectionBase>(&Sec)) {
        // Rename relocation sections associated to the allocated sections.
        // For example, if we rename .text to .prefix.text, we also rename
        // .rel.text to .rel.prefix.text.
        //
        // Dynamic relocation sections (SHT_REL[A] with SHF_ALLOC) are handled
        // above, e.g., .rela.plt is renamed to .prefix.rela.plt, not
        // .rela.prefix.plt since GNU objcopy does so.
        const SectionBase *TargetSec = RelocSec->getSection();
        if (TargetSec && (TargetSec->Flags & SHF_ALLOC)) {
          // If the relocation section comes *after* the target section, we
```
- **EN**: Implements logic around `find`, `end`, `getNamePrefix`, `empty`, and 5 more symbols.
- **CN**: 围绕 `find`, `end`, `getNamePrefix`, `empty`, and 5 more symbols 实现具体逻辑。

### Lines 1017-1030
```cpp
          // don't add Config.AllocSectionsPrefix because we've already added
          // the prefix to TargetSec->Name. Otherwise, if the relocation
          // section comes *before* the target section, we add the prefix.
          if (PrefixedSections.count(TargetSec))
            Sec.Name = (RelocSec->getNamePrefix() + TargetSec->Name).str();
          else
            Sec.Name = (RelocSec->getNamePrefix() + Config.AllocSectionsPrefix +
                        TargetSec->Name)
                           .str();
        }
      }
    }
  }

```
- **EN**: Implements logic around `count`, `getNamePrefix`, `str`.
- **CN**: 围绕 `count`, `getNamePrefix`, `str` 实现具体逻辑。

### Lines 1031-1044
```cpp
  if (ELFConfig.EntryExpr)
    Obj.Entry = ELFConfig.EntryExpr(Obj.Entry);
  return Error::success();
}

static Error writeOutput(const CommonConfig &Config, Object &Obj,
                         raw_ostream &Out, ElfType OutputElfType) {
  std::unique_ptr<Writer> Writer =
      createWriter(Config, Obj, Out, OutputElfType);
  if (Error E = Writer->finalize())
    return E;
  return Writer->write();
}

```
- **EN**: Implements logic around `EntryExpr`, `success`, `writeOutput`, `createWriter`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities; emits or serializes data to an external representation.
- **CN**: 围绕 `EntryExpr`, `success`, `writeOutput`, `createWriter`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并把数据输出或序列化为外部表示。

### Lines 1045-1059
```cpp
Error objcopy::elf::executeObjcopyOnIHex(const CommonConfig &Config,
                                         const ELFConfig &ELFConfig,
                                         MemoryBuffer &In, raw_ostream &Out) {
  IHexReader Reader(&In);
  Expected<std::unique_ptr<Object>> Obj = Reader.create(true);
  if (!Obj)
    return Obj.takeError();

  const ElfType OutputElfType =
      getOutputElfType(Config.OutputArch.value_or(MachineInfo()));
  if (Error E = handleArgs(Config, ELFConfig, OutputElfType, **Obj))
    return E;
  return writeOutput(Config, **Obj, Out, OutputElfType);
}

```
- **EN**: Implements logic around `executeObjcopyOnIHex`, `Reader`, `create`, `takeError`, and 3 more symbols; this block propagates recoverable errors through LLVM error utilities; emits or serializes data to an external representation; applies object-format-specific rules.
- **CN**: 围绕 `executeObjcopyOnIHex`, `Reader`, `create`, `takeError`, and 3 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并把数据输出或序列化为外部表示，并应用目标文件格式专用规则。

### Lines 1060-1077
```cpp
Error objcopy::elf::executeObjcopyOnRawBinary(const CommonConfig &Config,
                                              const ELFConfig &ELFConfig,
                                              MemoryBuffer &In,
                                              raw_ostream &Out) {
  BinaryReader Reader(&In, ELFConfig.NewSymbolVisibility);
  Expected<std::unique_ptr<Object>> Obj = Reader.create(true);
  if (!Obj)
    return Obj.takeError();

  // Prefer OutputArch (-O<format>) if set, otherwise fallback to BinaryArch
  // (-B<arch>).
  const ElfType OutputElfType =
      getOutputElfType(Config.OutputArch.value_or(MachineInfo()));
  if (Error E = handleArgs(Config, ELFConfig, OutputElfType, **Obj))
    return E;
  return writeOutput(Config, **Obj, Out, OutputElfType);
}

```
- **EN**: Implements logic around `executeObjcopyOnRawBinary`, `Reader`, `create`, `takeError`, and 3 more symbols; this block propagates recoverable errors through LLVM error utilities; emits or serializes data to an external representation; applies object-format-specific rules.
- **CN**: 围绕 `executeObjcopyOnRawBinary`, `Reader`, `create`, `takeError`, and 3 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并把数据输出或序列化为外部表示，并应用目标文件格式专用规则。

### Lines 1078-1091
```cpp
Error objcopy::elf::executeObjcopyOnBinary(const CommonConfig &Config,
                                           const ELFConfig &ELFConfig,
                                           object::ELFObjectFileBase &In,
                                           raw_ostream &Out) {
  ELFReader Reader(&In, Config.ExtractPartition);
  Expected<std::unique_ptr<Object>> Obj =
      Reader.create(!Config.SymbolsToAdd.empty());
  if (!Obj)
    return Obj.takeError();
  // Prefer OutputArch (-O<format>) if set, otherwise infer it from the input.
  const ElfType OutputElfType = Config.OutputArch
                                    ? getOutputElfType(*Config.OutputArch)
                                    : getOutputElfType(In);

```
- **EN**: Implements logic around `executeObjcopyOnBinary`, `Reader`, `create`, `takeError`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities; applies object-format-specific rules.
- **CN**: 围绕 `executeObjcopyOnBinary`, `Reader`, `create`, `takeError`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并应用目标文件格式专用规则。

### Lines 1092-1099
```cpp
  if (Error E = handleArgs(Config, ELFConfig, OutputElfType, **Obj))
    return E;

  if (Error E = writeOutput(Config, **Obj, Out, OutputElfType))
    return createFileError(Config.InputFilename, std::move(E));

  return Error::success();
}
```
- **EN**: Implements logic around `handleArgs`, `writeOutput`, `createFileError`, `success`; this block propagates recoverable errors through LLVM error utilities; emits or serializes data to an external representation.
- **CN**: 围绕 `handleArgs`, `writeOutput`, `createFileError`, `success` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并把数据输出或序列化为外部表示。

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

- **Direct LLVM/local includes / 直接的 LLVM/本地包含**: `llvm/ObjCopy/ELF/ELFObjcopy.h`, `ELFObject.h`, `llvm/ADT/BitmaskEnum.h`, `llvm/ADT/DenseSet.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h`, `llvm/BinaryFormat/ELF.h`, `llvm/MC/MCTargetOptions.h`, `llvm/ObjCopy/CommonConfig.h` ... (+13 more)
- **Standard-library headers / 标准库头文件**: `<algorithm>`, `<cassert>`, `<cstdlib>`, `<functional>`, `<memory>`, `<string>`, `<system_error>`, `<utility>`
- **Subsystem categories / 子系统类别**: support-library helpers / Support 库辅助功能 (7), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (5), object-file reading abstractions / 目标文件读取抽象 (4), objcopy configuration and rewriting interfaces / objcopy 配置与改写接口 (3), binary-format constants and record definitions / 二进制格式常量与记录定义 (1), machine-code layer support / 机器码层支持 (1), command-line option parsing support / 命令行选项解析支持 (1)
