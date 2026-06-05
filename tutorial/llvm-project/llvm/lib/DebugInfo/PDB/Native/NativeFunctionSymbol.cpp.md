# NativeFunctionSymbol.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/DebugInfo/PDB/Native/NativeFunctionSymbol.cpp`
- Repository: `llvm-project`
- Purpose (EN): Returns true if inline site contains the offset.
- Purpose (CN): 该文件位于 LLVM 的 `DebugInfo/PDB/Native` 目录中，主要实现与 `NativeFunctionSymbol` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===- NativeFunctionSymbol.cpp - info about function symbols----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/DebugInfo/PDB/Native/NativeFunctionSymbol.h"

#include "llvm/DebugInfo/CodeView/CVRecord.h"
#include "llvm/DebugInfo/CodeView/CodeView.h"
#include "llvm/DebugInfo/CodeView/SymbolDeserializer.h"
#include "llvm/DebugInfo/CodeView/SymbolRecord.h"
#include "llvm/DebugInfo/PDB/Native/ModuleDebugStream.h"
#include "llvm/DebugInfo/PDB/Native/NativeEnumSymbols.h"
#include "llvm/DebugInfo/PDB/Native/NativeSession.h"
#include "llvm/DebugInfo/PDB/Native/SymbolCache.h"
#include "llvm/DebugInfo/PDB/PDBExtras.h"

using namespace llvm;
using namespace llvm::codeview;
using namespace llvm::pdb;

NativeFunctionSymbol::NativeFunctionSymbol(NativeSession &Session,
                                           SymIndexId Id,
                                           const codeview::ProcSym &Sym,
                                           uint32_t Offset)
    : NativeRawSymbol(Session, PDB_SymType::Function, Id), Sym(Sym),
      RecordOffset(Offset) {}

NativeFunctionSymbol::~NativeFunctionSymbol() = default;

void NativeFunctionSymbol::dump(raw_ostream &OS, int Indent,
                                PdbSymbolIdField ShowIdFields,
                                PdbSymbolIdField RecurseIdFields) const {
  NativeRawSymbol::dump(OS, Indent, ShowIdFields, RecurseIdFields);
  dumpSymbolField(OS, "name", getName(), Indent);
  dumpSymbolField(OS, "length", getLength(), Indent);
  dumpSymbolField(OS, "offset", getAddressOffset(), Indent);
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 10 direct dependencies, including `llvm/DebugInfo/PDB/Native/NativeFunctionSymbol.h`, `llvm/DebugInfo/CodeView/CVRecord.h`, `llvm/DebugInfo/CodeView/CodeView.h`, `llvm/DebugInfo/CodeView/SymbolDeserializer.h`.
  CN: 引入了 10 个直接依赖，其中包括 `llvm/DebugInfo/PDB/Native/NativeFunctionSymbol.h`, `llvm/DebugInfo/CodeView/CVRecord.h`, `llvm/DebugInfo/CodeView/CodeView.h`, `llvm/DebugInfo/CodeView/SymbolDeserializer.h`。
- EN: This section centers on `NativeFunctionSymbol`, `dump`, `dumpSymbolField` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `NativeFunctionSymbol`, `dump`, `dumpSymbolField` 等符号展开，负责为下游使用方写出、格式化或序列化结果。

### Lines 41-80

```cpp
  dumpSymbolField(OS, "section", getAddressSection(), Indent);
}

uint32_t NativeFunctionSymbol::getAddressOffset() const {
  return Sym.CodeOffset;
}

uint32_t NativeFunctionSymbol::getAddressSection() const { return Sym.Segment; }
std::string NativeFunctionSymbol::getName() const {
  return std::string(Sym.Name);
}

uint64_t NativeFunctionSymbol::getLength() const { return Sym.CodeSize; }

uint32_t NativeFunctionSymbol::getRelativeVirtualAddress() const {
  return Session.getRVAFromSectOffset(Sym.Segment, Sym.CodeOffset);
}

uint64_t NativeFunctionSymbol::getVirtualAddress() const {
  return Session.getVAFromSectOffset(Sym.Segment, Sym.CodeOffset);
}

static bool inlineSiteContainsAddress(InlineSiteSym &IS,
                                      uint32_t OffsetInFunc) {
  // Returns true if inline site contains the offset.
  bool Found = false;
  uint32_t CodeOffset = 0;
  for (auto &Annot : IS.annotations()) {
    switch (Annot.OpCode) {
    case BinaryAnnotationsOpCode::CodeOffset:
    case BinaryAnnotationsOpCode::ChangeCodeOffset:
    case BinaryAnnotationsOpCode::ChangeCodeOffsetAndLineOffset:
      CodeOffset += Annot.U1;
      if (OffsetInFunc >= CodeOffset)
        Found = true;
      break;
    case BinaryAnnotationsOpCode::ChangeCodeLength:
      CodeOffset += Annot.U1;
      if (Found && OffsetInFunc < CodeOffset)
        return true;
```
- EN: This section centers on `dumpSymbolField`, `getAddressOffset`, `getAddressSection` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `dumpSymbolField`, `getAddressOffset`, `getAddressSection` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and iterates over collections, ranges, or records.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并遍历集合、区间或记录。

### Lines 81-120

```cpp
      Found = false;
      break;
    case BinaryAnnotationsOpCode::ChangeCodeLengthAndCodeOffset:
      CodeOffset += Annot.U2;
      if (OffsetInFunc >= CodeOffset && OffsetInFunc < CodeOffset + Annot.U1)
        return true;
      Found = false;
      break;
    default:
      break;
    }
  }
  return false;
}

std::unique_ptr<IPDBEnumSymbols>
NativeFunctionSymbol::findInlineFramesByVA(uint64_t VA) const {
  uint16_t Modi;
  if (!Session.moduleIndexForVA(VA, Modi))
    return nullptr;

  Expected<ModuleDebugStreamRef> ModS = Session.getModuleDebugStream(Modi);
  if (!ModS) {
    consumeError(ModS.takeError());
    return nullptr;
  }
  CVSymbolArray Syms = ModS->getSymbolArray();

  // Search for inline sites. There should be one matching top level inline
  // site. Then search in its nested inline sites.
  std::vector<SymIndexId> Frames;
  uint32_t CodeOffset = VA - getVirtualAddress();
  auto Start = Syms.at(RecordOffset);
  auto End = Syms.at(Sym.End);
  while (Start != End) {
    bool Found = false;
    // Find matching inline site within Start and End.
    for (; Start != End; ++Start) {
      if (Start->kind() != S_INLINESITE)
        continue;
```
- EN: This section centers on `findInlineFramesByVA`, `consumeError` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `findInlineFramesByVA`, `consumeError` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 121-147

```cpp

      InlineSiteSym IS =
          cantFail(SymbolDeserializer::deserializeAs<InlineSiteSym>(*Start));
      if (inlineSiteContainsAddress(IS, CodeOffset)) {
        // Insert frames in reverse order.
        SymIndexId Id = Session.getSymbolCache().getOrCreateInlineSymbol(
            IS, getVirtualAddress(), Modi, Start.offset());
        Frames.insert(Frames.begin(), Id);

        // Update offsets to search within this inline site.
        ++Start;
        End = Syms.at(IS.End);
        Found = true;
        break;
      }

      Start = Syms.at(IS.End);
      if (Start == End)
        break;
    }

    if (!Found)
      break;
  }

  return std::make_unique<NativeEnumSymbols>(Session, std::move(Frames));
}
```
- EN: This section centers on `cantFail`, `getVirtualAddress` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `cantFail`, `getVirtualAddress` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

## Key Concepts / 关键概念
- Domain / 领域: PDB debug information / PDB 调试信息
- Core symbols / 核心符号: `NativeFunctionSymbol`, `dump`, `dumpSymbolField`, `getAddressOffset` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Formatting, emission, and outward serialization. / 重点关注格式化、输出与序列化。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/DebugInfo/PDB/Native/NativeFunctionSymbol.h`, `llvm/DebugInfo/CodeView/CVRecord.h`, `llvm/DebugInfo/CodeView/CodeView.h`, `llvm/DebugInfo/CodeView/SymbolDeserializer.h`, `llvm/DebugInfo/CodeView/SymbolRecord.h`, `llvm/DebugInfo/PDB/Native/ModuleDebugStream.h`, `llvm/DebugInfo/PDB/Native/NativeEnumSymbols.h`, `llvm/DebugInfo/PDB/Native/NativeSession.h`, `llvm/DebugInfo/PDB/Native/SymbolCache.h`, `llvm/DebugInfo/PDB/PDBExtras.h`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `NativeFunctionSymbol`, `dump`, `dumpSymbolField`, `getAddressOffset`, `getAddressSection`
