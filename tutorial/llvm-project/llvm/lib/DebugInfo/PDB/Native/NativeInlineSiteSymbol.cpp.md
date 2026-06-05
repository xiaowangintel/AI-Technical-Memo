# NativeInlineSiteSymbol.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/DebugInfo/PDB/Native/NativeInlineSiteSymbol.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements NativeInlineSiteSymbol-related logic for LLVM's DebugInfo/PDB/Native component.
- Purpose (CN): 该文件位于 LLVM 的 `DebugInfo/PDB/Native` 目录中，主要实现与 `NativeInlineSiteSymbol` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===- NativeInlineSiteSymbol.cpp - info about inline sites -----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/DebugInfo/PDB/Native/NativeInlineSiteSymbol.h"

#include "llvm/DebugInfo/CodeView/DebugInlineeLinesSubsection.h"
#include "llvm/DebugInfo/CodeView/LazyRandomTypeCollection.h"
#include "llvm/DebugInfo/CodeView/SymbolRecord.h"
#include "llvm/DebugInfo/CodeView/TypeDeserializer.h"
#include "llvm/DebugInfo/PDB/Native/ModuleDebugStream.h"
#include "llvm/DebugInfo/PDB/Native/NativeEnumLineNumbers.h"
#include "llvm/DebugInfo/PDB/Native/NativeLineNumber.h"
#include "llvm/DebugInfo/PDB/Native/NativeSession.h"
#include "llvm/DebugInfo/PDB/Native/PDBFile.h"
#include "llvm/DebugInfo/PDB/Native/SymbolCache.h"
#include "llvm/DebugInfo/PDB/Native/TpiStream.h"
#include "llvm/DebugInfo/PDB/PDBExtras.h"

using namespace llvm;
using namespace llvm::codeview;
using namespace llvm::pdb;

NativeInlineSiteSymbol::NativeInlineSiteSymbol(
    NativeSession &Session, SymIndexId Id, const codeview::InlineSiteSym &Sym,
    uint64_t ParentAddr)
    : NativeRawSymbol(Session, PDB_SymType::InlineSite, Id), Sym(Sym),
      ParentAddr(ParentAddr) {}

NativeInlineSiteSymbol::~NativeInlineSiteSymbol() = default;

void NativeInlineSiteSymbol::dump(raw_ostream &OS, int Indent,
                                  PdbSymbolIdField ShowIdFields,
                                  PdbSymbolIdField RecurseIdFields) const {
  NativeRawSymbol::dump(OS, Indent, ShowIdFields, RecurseIdFields);
  dumpSymbolField(OS, "name", getName(), Indent);
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 13 direct dependencies, including `llvm/DebugInfo/PDB/Native/NativeInlineSiteSymbol.h`, `llvm/DebugInfo/CodeView/DebugInlineeLinesSubsection.h`, `llvm/DebugInfo/CodeView/LazyRandomTypeCollection.h`, `llvm/DebugInfo/CodeView/SymbolRecord.h`.
  CN: 引入了 13 个直接依赖，其中包括 `llvm/DebugInfo/PDB/Native/NativeInlineSiteSymbol.h`, `llvm/DebugInfo/CodeView/DebugInlineeLinesSubsection.h`, `llvm/DebugInfo/CodeView/LazyRandomTypeCollection.h`, `llvm/DebugInfo/CodeView/SymbolRecord.h`。
- EN: This section centers on `NativeInlineSiteSymbol`, `dump`, `dumpSymbolField` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `NativeInlineSiteSymbol`, `dump`, `dumpSymbolField` 等符号展开，负责为下游使用方写出、格式化或序列化结果。

### Lines 41-80

```cpp
}

static std::optional<InlineeSourceLine>
findInlineeByTypeIndex(TypeIndex Id, ModuleDebugStreamRef &ModS) {
  for (const auto &SS : ModS.getSubsectionsArray()) {
    if (SS.kind() != DebugSubsectionKind::InlineeLines)
      continue;

    DebugInlineeLinesSubsectionRef InlineeLines;
    BinaryStreamReader Reader(SS.getRecordData());
    if (auto EC = InlineeLines.initialize(Reader)) {
      consumeError(std::move(EC));
      continue;
    }

    for (const InlineeSourceLine &Line : InlineeLines)
      if (Line.Header->Inlinee == Id)
        return Line;
  }
  return std::nullopt;
}

std::string NativeInlineSiteSymbol::getName() const {
  auto Tpi = Session.getPDBFile().getPDBTpiStream();
  if (!Tpi) {
    consumeError(Tpi.takeError());
    return "";
  }
  auto Ipi = Session.getPDBFile().getPDBIpiStream();
  if (!Ipi) {
    consumeError(Ipi.takeError());
    return "";
  }

  LazyRandomTypeCollection &Types = Tpi->typeCollection();
  LazyRandomTypeCollection &Ids = Ipi->typeCollection();
  CVType InlineeType = Ids.getType(Sym.Inlinee);
  std::string QualifiedName;
  if (InlineeType.kind() == LF_MFUNC_ID) {
    MemberFuncIdRecord MFRecord;
```
- EN: This section centers on `findInlineeByTypeIndex`, `Reader`, `consumeError` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `findInlineeByTypeIndex`, `Reader`, `consumeError` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 81-120

```cpp
    cantFail(TypeDeserializer::deserializeAs<MemberFuncIdRecord>(InlineeType,
                                                                 MFRecord));
    TypeIndex ClassTy = MFRecord.getClassType();
    QualifiedName.append(std::string(Types.getTypeName(ClassTy)));
    QualifiedName.append("::");
  } else if (InlineeType.kind() == LF_FUNC_ID) {
    FuncIdRecord FRecord;
    cantFail(
        TypeDeserializer::deserializeAs<FuncIdRecord>(InlineeType, FRecord));
    TypeIndex ParentScope = FRecord.getParentScope();
    if (!ParentScope.isNoneType()) {
      QualifiedName.append(std::string(Ids.getTypeName(ParentScope)));
      QualifiedName.append("::");
    }
  }

  QualifiedName.append(std::string(Ids.getTypeName(Sym.Inlinee)));
  return QualifiedName;
}

void NativeInlineSiteSymbol::getLineOffset(uint32_t OffsetInFunc,
                                           uint32_t &LineOffset,
                                           uint32_t &FileOffset) const {
  LineOffset = 0;
  FileOffset = 0;
  uint32_t CodeOffset = 0;
  std::optional<uint32_t> CodeOffsetBase;
  std::optional<uint32_t> CodeOffsetEnd;
  std::optional<int32_t> CurLineOffset;
  std::optional<int32_t> NextLineOffset;
  std::optional<uint32_t> NextFileOffset;
  auto UpdateCodeOffset = [&](uint32_t Delta) {
    if (!CodeOffsetBase)
      CodeOffsetBase = CodeOffset;
    else if (!CodeOffsetEnd)
      CodeOffsetEnd = *CodeOffsetBase + Delta;
  };
  auto UpdateLineOffset = [&](int32_t Delta) {
    LineOffset += Delta;
    if (!CodeOffsetBase || !CurLineOffset)
```
- EN: This section centers on `cantFail`, `getLineOffset` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `cantFail`, `getLineOffset` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-160

```cpp
      CurLineOffset = LineOffset;
    else
      NextLineOffset = LineOffset;
  };
  auto UpdateFileOffset = [&](uint32_t Offset) {
    if (!CodeOffsetBase)
      FileOffset = Offset;
    else
      NextFileOffset = Offset;
  };
  auto ValidateAndReset = [&]() {
    // Current range is finished. Check if OffsetInFunc is in the range.
    if (CodeOffsetBase && CodeOffsetEnd && CurLineOffset) {
      if (CodeOffsetBase <= OffsetInFunc && OffsetInFunc < CodeOffsetEnd) {
        LineOffset = *CurLineOffset;
        return true;
      }
      // Set base, end, file offset and line offset for next range.
      if (NextFileOffset)
        FileOffset = *NextFileOffset;
      if (NextLineOffset) {
        CurLineOffset = NextLineOffset;
        NextLineOffset = std::nullopt;
      }
      CodeOffsetBase = CodeOffsetEnd;
      CodeOffsetEnd = NextFileOffset = std::nullopt;
    }
    return false;
  };
  for (const auto &Annot : Sym.annotations()) {
    switch (Annot.OpCode) {
    case BinaryAnnotationsOpCode::CodeOffset:
    case BinaryAnnotationsOpCode::ChangeCodeOffset:
    case BinaryAnnotationsOpCode::ChangeCodeOffsetBase:
      CodeOffset += Annot.U1;
      UpdateCodeOffset(Annot.U1);
      break;
    case BinaryAnnotationsOpCode::ChangeCodeLength:
      UpdateCodeOffset(Annot.U1);
      break;
```
- EN: This section centers on `UpdateCodeOffset` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `UpdateCodeOffset` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and iterates over collections, ranges, or records.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并遍历集合、区间或记录。

### Lines 161-200

```cpp
    case BinaryAnnotationsOpCode::ChangeCodeLengthAndCodeOffset:
      CodeOffset += Annot.U2;
      UpdateCodeOffset(Annot.U2);
      UpdateCodeOffset(Annot.U1);
      break;
    case BinaryAnnotationsOpCode::ChangeLineOffset:
      UpdateLineOffset(Annot.S1);
      break;
    case BinaryAnnotationsOpCode::ChangeCodeOffsetAndLineOffset:
      CodeOffset += Annot.U1;
      UpdateCodeOffset(Annot.U1);
      UpdateLineOffset(Annot.S1);
      break;
    case BinaryAnnotationsOpCode::ChangeFile:
      UpdateFileOffset(Annot.U1);
      break;
    default:
      break;
    }

    if (ValidateAndReset())
      return;
  }
}

std::unique_ptr<IPDBEnumLineNumbers>
NativeInlineSiteSymbol::findInlineeLinesByVA(uint64_t VA,
                                             uint32_t Length) const {
  uint16_t Modi;
  if (!Session.moduleIndexForVA(VA, Modi))
    return nullptr;

  Expected<ModuleDebugStreamRef> ModS = Session.getModuleDebugStream(Modi);
  if (!ModS) {
    consumeError(ModS.takeError());
    return nullptr;
  }

  Expected<DebugChecksumsSubsectionRef> Checksums =
      ModS->findChecksumsSubsection();
```
- EN: This section centers on `UpdateCodeOffset`, `UpdateLineOffset`, `UpdateFileOffset` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `UpdateCodeOffset`, `UpdateLineOffset`, `UpdateFileOffset` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 201-235

```cpp
  if (!Checksums) {
    consumeError(Checksums.takeError());
    return nullptr;
  }

  // Get the line number offset and source file offset.
  uint32_t SrcLineOffset;
  uint32_t SrcFileOffset;
  getLineOffset(VA - ParentAddr, SrcLineOffset, SrcFileOffset);

  // Get line info from inlinee line table.
  std::optional<InlineeSourceLine> Inlinee =
      findInlineeByTypeIndex(Sym.Inlinee, ModS.get());

  if (!Inlinee)
    return nullptr;

  uint32_t SrcLine = Inlinee->Header->SourceLineNum + SrcLineOffset;
  uint32_t SrcCol = 0; // Inline sites don't seem to have column info.
  uint32_t FileChecksumOffset =
      (SrcFileOffset == 0) ? Inlinee->Header->FileID : SrcFileOffset;

  auto ChecksumIter = Checksums->getArray().at(FileChecksumOffset);
  uint32_t SrcFileId =
      Session.getSymbolCache().getOrCreateSourceFile(*ChecksumIter);

  uint32_t LineSect, LineOff;
  Session.addressForVA(VA, LineSect, LineOff);
  NativeLineNumber LineNum(Session, SrcLine, SrcCol, LineSect, LineOff, Length,
                           SrcFileId, Modi);
  auto SrcFile = Session.getSymbolCache().getSourceFileById(SrcFileId);
  std::vector<NativeLineNumber> Lines{LineNum};

  return std::make_unique<NativeEnumLineNumbers>(std::move(Lines));
}
```
- EN: This section centers on `consumeError`, `getLineOffset`, `findInlineeByTypeIndex` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `consumeError`, `getLineOffset`, `findInlineeByTypeIndex` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

## Key Concepts / 关键概念
- Domain / 领域: PDB debug information / PDB 调试信息
- Core symbols / 核心符号: `NativeInlineSiteSymbol`, `dump`, `dumpSymbolField`, `findInlineeByTypeIndex` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Formatting, emission, and outward serialization. / 重点关注格式化、输出与序列化。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/DebugInfo/PDB/Native/NativeInlineSiteSymbol.h`, `llvm/DebugInfo/CodeView/DebugInlineeLinesSubsection.h`, `llvm/DebugInfo/CodeView/LazyRandomTypeCollection.h`, `llvm/DebugInfo/CodeView/SymbolRecord.h`, `llvm/DebugInfo/CodeView/TypeDeserializer.h`, `llvm/DebugInfo/PDB/Native/ModuleDebugStream.h`, `llvm/DebugInfo/PDB/Native/NativeEnumLineNumbers.h`, `llvm/DebugInfo/PDB/Native/NativeLineNumber.h`, `llvm/DebugInfo/PDB/Native/NativeSession.h`, `llvm/DebugInfo/PDB/Native/PDBFile.h`, `llvm/DebugInfo/PDB/Native/SymbolCache.h`, `llvm/DebugInfo/PDB/Native/TpiStream.h`, `llvm/DebugInfo/PDB/PDBExtras.h`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `NativeInlineSiteSymbol`, `dump`, `dumpSymbolField`, `findInlineeByTypeIndex`, `Reader`
