# DWARFDebugMacro.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/DebugInfo/DWARF/DWARFDebugMacro.cpp`
- Repository: `llvm-project`
- Purpose (EN): FIXME: Add support for dumping opcode_operands_table
- Purpose (CN): 该文件位于 LLVM 的 `DebugInfo/DWARF` 目录中，主要实现与 `DWARFDebugMacro` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===- DWARFDebugMacro.cpp ------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/DebugInfo/DWARF/DWARFDebugMacro.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/BinaryFormat/Dwarf.h"
#include "llvm/DebugInfo/DWARF/DWARFDataExtractor.h"
#include "llvm/DebugInfo/DWARF/DWARFDie.h"
#include "llvm/DebugInfo/DWARF/DWARFFormValue.h"
#include "llvm/Support/Errc.h"
#include "llvm/Support/FormatAdapters.h"
#include "llvm/Support/FormatVariadic.h"
#include "llvm/Support/WithColor.h"
#include "llvm/Support/raw_ostream.h"
#include <cstdint>

using namespace llvm;
using namespace dwarf;

DwarfFormat DWARFDebugMacro::MacroHeader::getDwarfFormat() const {
  return Flags & MACRO_OFFSET_SIZE ? DWARF64 : DWARF32;
}

uint8_t DWARFDebugMacro::MacroHeader::getOffsetByteSize() const {
  return getDwarfOffsetByteSize(getDwarfFormat());
}

void DWARFDebugMacro::MacroHeader::dumpMacroHeader(raw_ostream &OS) const {
  // FIXME: Add support for dumping opcode_operands_table
  OS << formatv("macro header: version = {0:x4}", Version)
     << formatv(", flags = {0:x2}", Flags)
     << ", format = " << FormatString(getDwarfFormat());
  if (Flags & MACRO_DEBUG_LINE_OFFSET)
    OS << formatv(", debug_line_offset = 0x{0:x-}",
                  fmt_align(DebugLineOffset, AlignStyle::Right,
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 12 direct dependencies, including `llvm/DebugInfo/DWARF/DWARFDebugMacro.h`, `llvm/ADT/DenseMap.h`, `llvm/BinaryFormat/Dwarf.h`, `llvm/DebugInfo/DWARF/DWARFDataExtractor.h`.
  CN: 引入了 12 个直接依赖，其中包括 `llvm/DebugInfo/DWARF/DWARFDebugMacro.h`, `llvm/ADT/DenseMap.h`, `llvm/BinaryFormat/Dwarf.h`, `llvm/DebugInfo/DWARF/DWARFDataExtractor.h`。
- EN: This section centers on `getDwarfFormat`, `getOffsetByteSize`, `getDwarfOffsetByteSize` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getDwarfFormat`, `getOffsetByteSize`, `getDwarfOffsetByteSize` 等符号展开，负责查询现有状态并返回计算出的元数据。

### Lines 41-80

```cpp
                            2 * getOffsetByteSize(), '0'));
  OS << "\n";
}

void DWARFDebugMacro::dump(raw_ostream &OS) const {
  unsigned IndLevel = 0;
  for (const auto &Macros : MacroLists) {
    OS << formatv("{0:x8}:\n", Macros.Offset);
    if (Macros.IsDebugMacro)
      Macros.Header.dumpMacroHeader(OS);
    for (const Entry &E : Macros.Macros) {
      // There should not be DW_MACINFO_end_file when IndLevel is Zero. However,
      // this check handles the case of corrupted ".debug_macinfo" section.
      if (IndLevel > 0)
        IndLevel -= (E.Type == DW_MACINFO_end_file);
      // Print indentation.
      for (unsigned I = 0; I < IndLevel; I++)
        OS << "  ";
      IndLevel += (E.Type == DW_MACINFO_start_file);
      // Based on which version we are handling choose appropriate macro forms.
      if (Macros.IsDebugMacro)
        WithColor(OS, HighlightColor::Macro).get()
            << (Macros.Header.Version < 5 ? GnuMacroString(E.Type)
                                          : MacroString(E.Type));
      else
        WithColor(OS, HighlightColor::Macro).get() << MacinfoString(E.Type);
      switch (E.Type) {
      default:
        // Got a corrupted ".debug_macinfo/.debug_macro" section (invalid
        // macinfo type).
        break;
        // debug_macro and debug_macinfo share some common encodings.
        // DW_MACRO_define     == DW_MACINFO_define
        // DW_MACRO_undef      == DW_MACINFO_undef
        // DW_MACRO_start_file == DW_MACINFO_start_file
        // DW_MACRO_end_file   == DW_MACINFO_end_file
        // For readability/uniformity we are using DW_MACRO_*.
        //
        // The GNU .debug_macro extension's entries have the same encoding
        // as DWARF 5's DW_MACRO_* entries, so we only use the latter here.
```
- EN: This section centers on `dump`, `WithColor` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `dump`, `WithColor` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and iterates over collections, ranges, or records.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并遍历集合、区间或记录。

### Lines 81-120

```cpp
      case DW_MACRO_define:
      case DW_MACRO_undef:
      case DW_MACRO_define_strp:
      case DW_MACRO_undef_strp:
      case DW_MACRO_define_strx:
      case DW_MACRO_undef_strx:
        OS << " - lineno: " << E.Line;
        OS << " macro: " << E.MacroStr;
        break;
      case DW_MACRO_start_file:
        OS << " - lineno: " << E.Line;
        OS << " filenum: " << E.File;
        break;
      case DW_MACRO_import:
        OS << formatv(" - import offset: 0x{0:x-}",
                      fmt_align(E.ImportOffset, AlignStyle::Right,
                                2 * Macros.Header.getOffsetByteSize(), '0'));
        break;
      case DW_MACRO_end_file:
        break;
      case DW_MACINFO_vendor_ext:
        OS << " - constant: " << E.ExtConstant;
        OS << " string: " << E.ExtStr;
        break;
      }
      OS << "\n";
    }
  }
}

Error DWARFDebugMacro::parseImpl(
    std::optional<DWARFUnitVector::compile_unit_range> Units,
    std::optional<DataExtractor> StringExtractor, DWARFDataExtractor Data,
    bool IsMacro) {
  uint64_t Offset = 0;
  MacroList *M = nullptr;
  using MacroToUnitsMap = DenseMap<uint64_t, DWARFUnit *>;
  MacroToUnitsMap MacroToUnits;
  if (IsMacro && Data.isValidOffset(Offset)) {
    // Keep a mapping from Macro contribution to CUs, this will
```
- EN: This section centers on `fmt_align`, `parseImpl` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `fmt_align`, `parseImpl` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-160

```cpp
    // be needed while retrieving macro from DW_MACRO_define_strx form.
    for (const auto &U : *Units)
      if (auto CUDIE = U->getUnitDIE())
        // Skip units which does not contibutes to macro section.
        if (auto MacroOffset = toSectionOffset(CUDIE.find(DW_AT_macros)))
          MacroToUnits.try_emplace(*MacroOffset, U.get());
  }
  while (Data.isValidOffset(Offset)) {
    if (!M) {
      MacroLists.emplace_back();
      M = &MacroLists.back();
      M->Offset = Offset;
      M->IsDebugMacro = IsMacro;
      if (IsMacro) {
        auto Err = M->Header.parseMacroHeader(Data, &Offset);
        if (Err)
          return Err;
      }
    }
    // A macro list entry consists of:
    M->Macros.emplace_back();
    Entry &E = M->Macros.back();
    // 1. Macinfo type
    E.Type = Data.getULEB128(&Offset);

    if (E.Type == 0) {
      // Reached end of a ".debug_macinfo/debug_macro" section contribution.
      M = nullptr;
      continue;
    }

    switch (E.Type) {
    default:
      // Got a corrupted ".debug_macinfo" section (invalid macinfo type).
      // Push the corrupted entry to the list and halt parsing.
      E.Type = DW_MACINFO_invalid;
      return Error::success();
    // debug_macro and debug_macinfo share some common encodings.
    // DW_MACRO_define     == DW_MACINFO_define
    // DW_MACRO_undef      == DW_MACINFO_undef
```
- EN: This section centers on `success` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `success` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and iterates over collections, ranges, or records.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并遍历集合、区间或记录。

### Lines 161-200

```cpp
    // DW_MACRO_start_file == DW_MACINFO_start_file
    // DW_MACRO_end_file   == DW_MACINFO_end_file
    // For readibility/uniformity we are using DW_MACRO_*.
    case DW_MACRO_define:
    case DW_MACRO_undef:
      // 2. Source line
      E.Line = Data.getULEB128(&Offset);
      // 3. Macro string
      E.MacroStr = Data.getCStr(&Offset);
      break;
    case DW_MACRO_define_strp:
    case DW_MACRO_undef_strp: {
      if (!IsMacro) {
        // DW_MACRO_define_strp is a new form introduced in DWARFv5, it is
        // not supported in debug_macinfo[.dwo] sections. Assume it as an
        // invalid entry, push it and halt parsing.
        E.Type = DW_MACINFO_invalid;
        return Error::success();
      }
      uint64_t StrOffset = 0;
      // 2. Source line
      E.Line = Data.getULEB128(&Offset);
      // 3. Macro string
      StrOffset =
          Data.getRelocatedValue(M->Header.getOffsetByteSize(), &Offset);
      assert(StringExtractor && "String Extractor not found");
      E.MacroStr = StringExtractor->getCStr(&StrOffset);
      break;
    }
    case DW_MACRO_define_strx:
    case DW_MACRO_undef_strx: {
      if (!IsMacro) {
        // DW_MACRO_define_strx is a new form introduced in DWARFv5, it is
        // not supported in debug_macinfo[.dwo] sections. Assume it as an
        // invalid entry, push it and halt parsing.
        E.Type = DW_MACINFO_invalid;
        return Error::success();
      }
      E.Line = Data.getULEB128(&Offset);
      auto MacroContributionOffset = MacroToUnits.find(M->Offset);
```
- EN: This section centers on `success`, `assert` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `success`, `assert` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 201-240

```cpp
      if (MacroContributionOffset == MacroToUnits.end())
        return createStringError(errc::invalid_argument,
                                 "Macro contribution of the unit not found");
      Expected<uint64_t> StrOffset =
          MacroContributionOffset->second->getStringOffsetSectionItem(
              Data.getULEB128(&Offset));
      if (!StrOffset)
        return StrOffset.takeError();
      E.MacroStr =
          MacroContributionOffset->second->getStringExtractor().getCStr(
              &*StrOffset);
      break;
    }
    case DW_MACRO_start_file:
      // 2. Source line
      E.Line = Data.getULEB128(&Offset);
      // 3. Source file id
      E.File = Data.getULEB128(&Offset);
      break;
    case DW_MACRO_end_file:
      break;
    case DW_MACRO_import:
      E.ImportOffset =
          Data.getRelocatedValue(M->Header.getOffsetByteSize(), &Offset);
      break;
    case DW_MACINFO_vendor_ext:
      // 2. Vendor extension constant
      E.ExtConstant = Data.getULEB128(&Offset);
      // 3. Vendor extension string
      E.ExtStr = Data.getCStr(&Offset);
      break;
    }
  }
  return Error::success();
}

Error DWARFDebugMacro::MacroHeader::parseMacroHeader(DWARFDataExtractor Data,
                                                     uint64_t *Offset) {
  Version = Data.getU16(Offset);
  uint8_t FlagData = Data.getU8(Offset);
```
- EN: This section centers on `success`, `parseMacroHeader` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `success`, `parseMacroHeader` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 241-250

```cpp

  // FIXME: Add support for parsing opcode_operands_table
  if (FlagData & MACRO_OPCODE_OPERANDS_TABLE)
    return createStringError(errc::not_supported,
                             "opcode_operands_table is not supported");
  Flags = FlagData;
  if (Flags & MACRO_DEBUG_LINE_OFFSET)
    DebugLineOffset = Data.getUnsigned(Offset, getOffsetByteSize());
  return Error::success();
}
```
- EN: This section centers on `success` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `success` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

## Key Concepts / 关键概念
- Domain / 领域: DWARF debug information / DWARF 调试信息
- Core symbols / 核心符号: `getDwarfFormat`, `getOffsetByteSize`, `getDwarfOffsetByteSize`, `dumpMacroHeader` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Formatting, emission, and outward serialization. / 重点关注格式化、输出与序列化。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/DebugInfo/DWARF/DWARFDebugMacro.h`, `llvm/ADT/DenseMap.h`, `llvm/BinaryFormat/Dwarf.h`, `llvm/DebugInfo/DWARF/DWARFDataExtractor.h`, `llvm/DebugInfo/DWARF/DWARFDie.h`, `llvm/DebugInfo/DWARF/DWARFFormValue.h`, `llvm/Support/Errc.h`, `llvm/Support/FormatAdapters.h`, `llvm/Support/FormatVariadic.h`, `llvm/Support/WithColor.h`, `llvm/Support/raw_ostream.h`
- Standard library / 标准库: `cstdint`
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `getDwarfFormat`, `getOffsetByteSize`, `getDwarfOffsetByteSize`, `dumpMacroHeader`, `fmt_align`
