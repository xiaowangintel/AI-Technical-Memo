# RecordName.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/DebugInfo/CodeView/RecordName.cpp`
- Repository: `llvm-project`
- Purpose (EN): / The type collection.
- Purpose (CN): 该文件位于 LLVM 的 `DebugInfo/CodeView` 目录中，主要实现与 `RecordName` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===- RecordName.cpp ----------------------------------------- *- C++ --*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/DebugInfo/CodeView/RecordName.h"

#include "llvm/ADT/SmallString.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/DebugInfo/CodeView/CVTypeVisitor.h"
#include "llvm/DebugInfo/CodeView/CodeView.h"
#include "llvm/DebugInfo/CodeView/SymbolRecord.h"
#include "llvm/DebugInfo/CodeView/SymbolRecordMapping.h"
#include "llvm/DebugInfo/CodeView/TypeCollection.h"
#include "llvm/DebugInfo/CodeView/TypeIndex.h"
#include "llvm/DebugInfo/CodeView/TypeRecord.h"
#include "llvm/DebugInfo/CodeView/TypeVisitorCallbacks.h"
#include "llvm/Support/FormatVariadic.h"

using namespace llvm;
using namespace llvm::codeview;

namespace {
class TypeNameComputer : public TypeVisitorCallbacks {
  /// The type collection.  Used to calculate names of nested types.
  TypeCollection &Types;
  TypeIndex CurrentTypeIndex = TypeIndex::None();

  /// Name of the current type. Only valid before visitTypeEnd.
  SmallString<256> Name;

public:
  explicit TypeNameComputer(TypeCollection &Types) : Types(Types) {}

  StringRef name() const { return Name; }

  /// Paired begin/end actions for all types. Receives all record data,
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 12 direct dependencies, including `llvm/DebugInfo/CodeView/RecordName.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/StringExtras.h`, `llvm/DebugInfo/CodeView/CVTypeVisitor.h`.
  CN: 引入了 12 个直接依赖，其中包括 `llvm/DebugInfo/CodeView/RecordName.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/StringExtras.h`, `llvm/DebugInfo/CodeView/CVTypeVisitor.h`。
- EN: This section centers on `TypeNameComputer`, `name` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `TypeNameComputer`, `name` 等符号展开，负责实现局部控制流程与状态维护。

### Lines 41-80

```cpp
  /// including the fixed-length record prefix.
  Error visitTypeBegin(CVType &Record) override;
  Error visitTypeBegin(CVType &Record, TypeIndex Index) override;
  Error visitTypeEnd(CVType &Record) override;

#define TYPE_RECORD(EnumName, EnumVal, Name)                                   \
  Error visitKnownRecord(CVType &CVR, Name##Record &Record) override;
#define TYPE_RECORD_ALIAS(EnumName, EnumVal, Name, AliasName)
#define MEMBER_RECORD(EnumName, EnumVal, Name)
#include "llvm/DebugInfo/CodeView/CodeViewTypes.def"
};
} // namespace

Error TypeNameComputer::visitTypeBegin(CVType &Record) {
  llvm_unreachable("Must call visitTypeBegin with a TypeIndex!");
  return Error::success();
}

Error TypeNameComputer::visitTypeBegin(CVType &Record, TypeIndex Index) {
  // Reset Name to the empty string. If the visitor sets it, we know it.
  Name = "";
  CurrentTypeIndex = Index;
  return Error::success();
}

Error TypeNameComputer::visitTypeEnd(CVType &CVR) { return Error::success(); }

Error TypeNameComputer::visitKnownRecord(CVType &CVR,
                                         FieldListRecord &FieldList) {
  Name = "<field list>";
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVRecord<TypeLeafKind> &CVR,
                                         StringIdRecord &String) {
  Name = String.getString();
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVType &CVR, ArgListRecord &Args) {
```
- EN: Brings in 1 direct dependencies, including `llvm/DebugInfo/CodeView/CodeViewTypes.def`.
  CN: 引入了 1 个直接依赖，其中包括 `llvm/DebugInfo/CodeView/CodeViewTypes.def`。
- EN: This section centers on `visitTypeBegin`, `llvm_unreachable`, `success` and walks over records and applies per-item processing.
  CN: 这一段主要围绕 `visitTypeBegin`, `llvm_unreachable`, `success` 等符号展开，负责遍历记录并执行逐项处理。
- EN: In this range, the code propagates LLVM-style errors and invariants and returns the resulting value to its callers.
  CN: 在这一段中，代码传播 LLVM 风格的错误处理并维护不变量，并将结果返回给调用方。

### Lines 81-120

```cpp
  auto Indices = Args.getIndices();
  uint32_t Size = Indices.size();
  Name = "(";
  for (uint32_t I = 0; I < Size; ++I) {
    if (Indices[I] < CurrentTypeIndex)
      Name.append(Types.getTypeName(Indices[I]));
    else
      Name.append("<unknown 0x" + utohexstr(Indices[I].getIndex()) + ">");
    if (I + 1 != Size)
      Name.append(", ");
  }
  Name.push_back(')');
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVType &CVR,
                                         StringListRecord &Strings) {
  auto Indices = Strings.getIndices();
  uint32_t Size = Indices.size();
  Name = "\"";
  for (uint32_t I = 0; I < Size; ++I) {
    Name.append(Types.getTypeName(Indices[I]));
    if (I + 1 != Size)
      Name.append("\" \"");
  }
  Name.push_back('\"');
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVType &CVR, ClassRecord &Class) {
  Name = Class.getName();
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVType &CVR, UnionRecord &Union) {
  Name = Union.getName();
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVType &CVR, EnumRecord &Enum) {
```
- EN: This section centers on `success`, `visitKnownRecord` and walks over records and applies per-item processing.
  CN: 这一段主要围绕 `success`, `visitKnownRecord` 等符号展开，负责遍历记录并执行逐项处理。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 121-160

```cpp
  Name = Enum.getName();
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVType &CVR, ArrayRecord &AT) {
  Name = AT.getName();
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVType &CVR, VFTableRecord &VFT) {
  Name = VFT.getName();
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVType &CVR, MemberFuncIdRecord &Id) {
  Name = Id.getName();
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVType &CVR, ProcedureRecord &Proc) {
  StringRef Ret = Types.getTypeName(Proc.getReturnType());
  StringRef Params = Types.getTypeName(Proc.getArgumentList());
  Name = formatv("{0} {1}", Ret, Params).sstr<256>();
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVType &CVR,
                                         MemberFunctionRecord &MF) {
  StringRef Ret = Types.getTypeName(MF.getReturnType());
  StringRef Class = Types.getTypeName(MF.getClassType());
  StringRef Params = Types.getTypeName(MF.getArgumentList());
  Name = formatv("{0} {1}::{2}", Ret, Class, Params).sstr<256>();
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVType &CVR, FuncIdRecord &Func) {
  Name = Func.getName();
  return Error::success();
}

```
- EN: This section centers on `success`, `visitKnownRecord` and walks over records and applies per-item processing.
  CN: 这一段主要围绕 `success`, `visitKnownRecord` 等符号展开，负责遍历记录并执行逐项处理。
- EN: In this range, the code propagates LLVM-style errors and invariants and returns the resulting value to its callers.
  CN: 在这一段中，代码传播 LLVM 风格的错误处理并维护不变量，并将结果返回给调用方。

### Lines 161-200

```cpp
Error TypeNameComputer::visitKnownRecord(CVType &CVR, TypeServer2Record &TS) {
  Name = TS.getName();
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVType &CVR, PointerRecord &Ptr) {

  if (Ptr.isPointerToMember()) {
    const MemberPointerInfo &MI = Ptr.getMemberInfo();

    StringRef Pointee = Types.getTypeName(Ptr.getReferentType());
    StringRef Class = Types.getTypeName(MI.getContainingType());
    Name = formatv("{0} {1}::*", Pointee, Class);
  } else {
    Name.append(Types.getTypeName(Ptr.getReferentType()));

    if (Ptr.getMode() == PointerMode::LValueReference)
      Name.append("&");
    else if (Ptr.getMode() == PointerMode::RValueReference)
      Name.append("&&");
    else if (Ptr.getMode() == PointerMode::Pointer)
      Name.append("*");

    // Qualifiers in pointer records apply to the pointer, not the pointee, so
    // they go on the right.
    if (Ptr.isConst())
      Name.append(" const");
    if (Ptr.isVolatile())
      Name.append(" volatile");
    if (Ptr.isUnaligned())
      Name.append(" __unaligned");
    if (Ptr.isRestrict())
      Name.append(" __restrict");
  }
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVType &CVR, ModifierRecord &Mod) {
  uint16_t Mods = static_cast<uint16_t>(Mod.getModifiers());

```
- EN: This section centers on `visitKnownRecord`, `success` and walks over records and applies per-item processing.
  CN: 这一段主要围绕 `visitKnownRecord`, `success` 等符号展开，负责遍历记录并执行逐项处理。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 201-240

```cpp
  if (Mods & uint16_t(ModifierOptions::Const))
    Name.append("const ");
  if (Mods & uint16_t(ModifierOptions::Volatile))
    Name.append("volatile ");
  if (Mods & uint16_t(ModifierOptions::Unaligned))
    Name.append("__unaligned ");
  Name.append(Types.getTypeName(Mod.getModifiedType()));
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVType &CVR,
                                         VFTableShapeRecord &Shape) {
  Name = formatv("<vftable {0} methods>", Shape.getEntryCount());
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(
    CVType &CVR, UdtModSourceLineRecord &ModSourceLine) {
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVType &CVR,
                                         UdtSourceLineRecord &SourceLine) {
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVType &CVR, BitFieldRecord &BF) {
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVType &CVR,
                                         MethodOverloadListRecord &Overloads) {
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVType &CVR, BuildInfoRecord &BI) {
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVType &CVR, LabelRecord &R) {
```
- EN: This section centers on `success`, `visitKnownRecord` and walks over records and applies per-item processing.
  CN: 这一段主要围绕 `success`, `visitKnownRecord` 等符号展开，负责遍历记录并执行逐项处理。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 241-280

```cpp
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVType &CVR,
                                         PrecompRecord &Precomp) {
  return Error::success();
}

Error TypeNameComputer::visitKnownRecord(CVType &CVR,
                                         EndPrecompRecord &EndPrecomp) {
  return Error::success();
}

std::string llvm::codeview::computeTypeName(TypeCollection &Types,
                                            TypeIndex Index) {
  TypeNameComputer Computer(Types);
  CVType Record = Types.getType(Index);
  if (auto EC = visitTypeRecord(Record, Index, Computer)) {
    consumeError(std::move(EC));
    return "<unknown UDT>";
  }
  return std::string(Computer.name());
}

static int getSymbolNameOffset(CVSymbol Sym) {
  switch (Sym.kind()) {
  // See ProcSym
  case SymbolKind::S_GPROC32:
  case SymbolKind::S_LPROC32:
  case SymbolKind::S_GPROC32_ID:
  case SymbolKind::S_LPROC32_ID:
  case SymbolKind::S_LPROC32_DPC:
  case SymbolKind::S_LPROC32_DPC_ID:
    return 35;
  // See Thunk32Sym
  case SymbolKind::S_THUNK32:
    return 21;
  // See SectionSym
  case SymbolKind::S_SECTION:
    return 16;
```
- EN: This section centers on `success`, `visitKnownRecord`, `computeTypeName` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `success`, `visitKnownRecord`, `computeTypeName` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and checks conditions and handles edge cases.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并检查条件并处理边界情况。

### Lines 281-320

```cpp
  // See CoffGroupSym, RegRelativeIndirSym
  case SymbolKind::S_COFFGROUP:
  case SymbolKind::S_REGREL32_INDIR:
    return 14;
  // See PublicSym32, FileStaticSym, RegRelativeSym, DataSym, ThreadLocalDataSym
  case SymbolKind::S_PUB32:
  case SymbolKind::S_FILESTATIC:
  case SymbolKind::S_REGREL32:
  case SymbolKind::S_GDATA32:
  case SymbolKind::S_LDATA32:
  case SymbolKind::S_LMANDATA:
  case SymbolKind::S_GMANDATA:
  case SymbolKind::S_LTHREAD32:
  case SymbolKind::S_GTHREAD32:
  case SymbolKind::S_PROCREF:
  case SymbolKind::S_LPROCREF:
    return 10;
  // See RegisterSym and LocalSym
  case SymbolKind::S_REGISTER:
  case SymbolKind::S_LOCAL:
    return 6;
  // See BlockSym
  case SymbolKind::S_BLOCK32:
    return 18;
  // See LabelSym
  case SymbolKind::S_LABEL32:
    return 7;
  // See ObjNameSym, ExportSym, and UDTSym
  case SymbolKind::S_OBJNAME:
  case SymbolKind::S_EXPORT:
  case SymbolKind::S_UDT:
    return 4;
  // See BPRelativeSym
  case SymbolKind::S_BPREL32:
    return 8;
  // See UsingNamespaceSym
  case SymbolKind::S_UNAMESPACE:
    return 0;
  default:
    return -1;
```
- EN: This range returns the resulting value to its callers.
  CN: 这一段将结果返回给调用方。

### Lines 321-344

```cpp
  }
}

StringRef llvm::codeview::getSymbolName(CVSymbol Sym) {
  if (Sym.kind() == SymbolKind::S_CONSTANT) {
    // S_CONSTANT is preceded by an APSInt, which has a variable length.  So we
    // have to do a full deserialization.
    BinaryStreamReader Reader(Sym.content(), llvm::endianness::little);
    // The container doesn't matter for single records.
    SymbolRecordMapping Mapping(Reader, CodeViewContainer::ObjectFile);
    ConstantSym Const(SymbolKind::S_CONSTANT);
    cantFail(Mapping.visitSymbolBegin(Sym));
    cantFail(Mapping.visitKnownRecord(Sym, Const));
    cantFail(Mapping.visitSymbolEnd(Sym));
    return Const.Name;
  }

  int Offset = getSymbolNameOffset(Sym);
  if (Offset == -1)
    return StringRef();

  StringRef StringData = toStringRef(Sym.content()).drop_front(Offset);
  return StringData.split('\0').first;
}
```
- EN: This section centers on `getSymbolName`, `Reader`, `Mapping` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `getSymbolName`, `Reader`, `Mapping` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

## Key Concepts / 关键概念
- Domain / 领域: CodeView debug information / CodeView 调试信息
- Core symbols / 核心符号: `TypeNameComputer`, `name`, `visitTypeBegin`, `llvm_unreachable` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/DebugInfo/CodeView/RecordName.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/StringExtras.h`, `llvm/DebugInfo/CodeView/CVTypeVisitor.h`, `llvm/DebugInfo/CodeView/CodeView.h`, `llvm/DebugInfo/CodeView/SymbolRecord.h`, `llvm/DebugInfo/CodeView/SymbolRecordMapping.h`, `llvm/DebugInfo/CodeView/TypeCollection.h`, `llvm/DebugInfo/CodeView/TypeIndex.h`, `llvm/DebugInfo/CodeView/TypeRecord.h`, `llvm/DebugInfo/CodeView/TypeVisitorCallbacks.h`, `llvm/Support/FormatVariadic.h`, `llvm/DebugInfo/CodeView/CodeViewTypes.def`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `TypeNameComputer`, `name`, `visitTypeBegin`, `llvm_unreachable`, `success`
