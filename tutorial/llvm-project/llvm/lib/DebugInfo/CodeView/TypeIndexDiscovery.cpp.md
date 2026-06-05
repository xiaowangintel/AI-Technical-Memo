# TypeIndexDiscovery.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/DebugInfo/CodeView/TypeIndexDiscovery.cpp`
- Repository: `llvm-project`
- Purpose (EN): Array of: 0: Attrs 2: Padding 4: TypeIndex
- Purpose (CN): 该文件位于 LLVM 的 `DebugInfo/CodeView` 目录中，主要实现与 `TypeIndexDiscovery` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60

```cpp
//===- TypeIndexDiscovery.cpp -----------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/DebugInfo/CodeView/TypeIndexDiscovery.h"
#include "llvm/DebugInfo/CodeView/TypeRecord.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/Support/Endian.h"

using namespace llvm;
using namespace llvm::codeview;

static inline MethodKind getMethodKind(uint16_t Attrs) {
  Attrs &= uint16_t(MethodOptions::MethodKindMask);
  Attrs >>= 2;
  return MethodKind(Attrs);
}

static inline bool isIntroVirtual(uint16_t Attrs) {
  MethodKind MK = getMethodKind(Attrs);
  return MK == MethodKind::IntroducingVirtual ||
         MK == MethodKind::PureIntroducingVirtual;
}

static inline PointerMode getPointerMode(uint32_t Attrs) {
  return static_cast<PointerMode>((Attrs >> PointerRecord::PointerModeShift) &
                                  PointerRecord::PointerModeMask);
}

static inline bool isMemberPointer(uint32_t Attrs) {
  PointerMode Mode = getPointerMode(Attrs);
  return Mode == PointerMode::PointerToDataMember ||
         Mode == PointerMode::PointerToMemberFunction;
}

static inline uint32_t getEncodedIntegerLength(ArrayRef<uint8_t> Data) {
  uint16_t N = support::endian::read16le(Data.data());
  if (N < LF_NUMERIC)
    return 2;

  assert(N <= LF_UQUADWORD);

  constexpr uint32_t Sizes[] = {
      1,  // LF_CHAR
      2,  // LF_SHORT
      2,  // LF_USHORT
      4,  // LF_LONG
      4,  // LF_ULONG
      4,  // LF_REAL32
      8,  // LF_REAL64
      10, // LF_REAL80
      16, // LF_REAL128
      8,  // LF_QUADWORD
      8,  // LF_UQUADWORD
  };

```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 4 direct dependencies, including `llvm/DebugInfo/CodeView/TypeIndexDiscovery.h`, `llvm/DebugInfo/CodeView/TypeRecord.h`, `llvm/ADT/ArrayRef.h`, `llvm/Support/Endian.h`.
  CN: 引入了 4 个直接依赖，其中包括 `llvm/DebugInfo/CodeView/TypeIndexDiscovery.h`, `llvm/DebugInfo/CodeView/TypeRecord.h`, `llvm/ADT/ArrayRef.h`, `llvm/Support/Endian.h`。
- EN: This section centers on `getMethodKind`, `MethodKind`, `isIntroVirtual` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getMethodKind`, `MethodKind`, `isIntroVirtual` 等符号展开，负责查询现有状态并返回计算出的元数据。

### Lines 61-120

```cpp
  return 2 + Sizes[N - LF_NUMERIC];
}

static inline uint32_t getCStringLength(ArrayRef<uint8_t> Data) {
  const char *S = reinterpret_cast<const char *>(Data.data());
  return strlen(S) + 1;
}

static void handleMethodOverloadList(ArrayRef<uint8_t> Content,
                                     SmallVectorImpl<TiReference> &Refs) {
  uint32_t Offset = 0;

  while (!Content.empty()) {
    // Array of:
    //   0: Attrs
    //   2: Padding
    //   4: TypeIndex
    //   if (isIntroVirtual())
    //     8: VFTableOffset

    // At least 8 bytes are guaranteed.  4 extra bytes come iff function is an
    // intro virtual.
    uint32_t Len = 8;

    uint16_t Attrs = support::endian::read16le(Content.data());
    Refs.push_back({TiRefKind::TypeRef, Offset + 4, 1});

    if (LLVM_UNLIKELY(isIntroVirtual(Attrs)))
      Len += 4;
    Offset += Len;
    Content = Content.drop_front(Len);
  }
}

static uint32_t handleBaseClass(ArrayRef<uint8_t> Data, uint32_t Offset,
                                SmallVectorImpl<TiReference> &Refs) {
  // 0: Kind
  // 2: Padding
  // 4: TypeIndex
  // 8: Encoded Integer
  Refs.push_back({TiRefKind::TypeRef, Offset + 4, 1});
  return 8 + getEncodedIntegerLength(Data.drop_front(8));
}

static uint32_t handleEnumerator(ArrayRef<uint8_t> Data, uint32_t Offset,
                                 SmallVectorImpl<TiReference> &Refs) {
  // 0: Kind
  // 2: Padding
  // 4: Encoded Integer
  // <next>: Name
  uint32_t Size = 4 + getEncodedIntegerLength(Data.drop_front(4));
  return Size + getCStringLength(Data.drop_front(Size));
}

static uint32_t handleDataMember(ArrayRef<uint8_t> Data, uint32_t Offset,
                                 SmallVectorImpl<TiReference> &Refs) {
  // 0: Kind
  // 2: Padding
  // 4: TypeIndex
  // 8: Encoded Integer
```
- EN: This section centers on `getCStringLength`, `handleMethodOverloadList`, `handleBaseClass` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getCStringLength`, `handleMethodOverloadList`, `handleBaseClass` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 121-180

```cpp
  // <next>: Name
  Refs.push_back({TiRefKind::TypeRef, Offset + 4, 1});
  uint32_t Size = 8 + getEncodedIntegerLength(Data.drop_front(8));
  return Size + getCStringLength(Data.drop_front(Size));
}

static uint32_t handleOverloadedMethod(ArrayRef<uint8_t> Data, uint32_t Offset,
                                       SmallVectorImpl<TiReference> &Refs) {
  // 0: Kind
  // 2: Padding
  // 4: TypeIndex
  // 8: Name
  Refs.push_back({TiRefKind::TypeRef, Offset + 4, 1});
  return 8 + getCStringLength(Data.drop_front(8));
}

static uint32_t handleOneMethod(ArrayRef<uint8_t> Data, uint32_t Offset,
                                SmallVectorImpl<TiReference> &Refs) {
  // 0: Kind
  // 2: Attributes
  // 4: Type
  // if (isIntroVirtual)
  //   8: VFTableOffset
  // <next>: Name
  uint32_t Size = 8;
  Refs.push_back({TiRefKind::TypeRef, Offset + 4, 1});

  uint16_t Attrs = support::endian::read16le(Data.drop_front(2).data());
  if (LLVM_UNLIKELY(isIntroVirtual(Attrs)))
    Size += 4;

  return Size + getCStringLength(Data.drop_front(Size));
}

static uint32_t handleNestedType(ArrayRef<uint8_t> Data, uint32_t Offset,
                                 SmallVectorImpl<TiReference> &Refs) {
  // 0: Kind
  // 2: Padding
  // 4: TypeIndex
  // 8: Name
  Refs.push_back({TiRefKind::TypeRef, Offset + 4, 1});
  return 8 + getCStringLength(Data.drop_front(8));
}

static uint32_t handleStaticDataMember(ArrayRef<uint8_t> Data, uint32_t Offset,
                                       SmallVectorImpl<TiReference> &Refs) {
  // 0: Kind
  // 2: Padding
  // 4: TypeIndex
  // 8: Name
  Refs.push_back({TiRefKind::TypeRef, Offset + 4, 1});
  return 8 + getCStringLength(Data.drop_front(8));
}

static uint32_t handleVirtualBaseClass(ArrayRef<uint8_t> Data, uint32_t Offset,
                                       bool IsIndirect,
                                       SmallVectorImpl<TiReference> &Refs) {
  // 0: Kind
  // 2: Attrs
  // 4: TypeIndex
```
- EN: This section centers on `handleOverloadedMethod`, `handleOneMethod`, `handleNestedType` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `handleOverloadedMethod`, `handleOneMethod`, `handleNestedType` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and updates helper containers and temporary state.
  CN: 在这一段中，代码检查条件并处理边界情况，并更新辅助容器和临时状态。

### Lines 181-240

```cpp
  // 8: TypeIndex
  // 12: Encoded Integer
  // <next>: Encoded Integer
  uint32_t Size = 12;
  Refs.push_back({TiRefKind::TypeRef, Offset + 4, 2});
  Size += getEncodedIntegerLength(Data.drop_front(Size));
  Size += getEncodedIntegerLength(Data.drop_front(Size));
  return Size;
}

static uint32_t handleVFPtr(ArrayRef<uint8_t> Data, uint32_t Offset,
                            SmallVectorImpl<TiReference> &Refs) {
  // 0: Kind
  // 2: Padding
  // 4: TypeIndex
  Refs.push_back({TiRefKind::TypeRef, Offset + 4, 1});
  return 8;
}

static uint32_t handleListContinuation(ArrayRef<uint8_t> Data, uint32_t Offset,
                                       SmallVectorImpl<TiReference> &Refs) {
  // 0: Kind
  // 2: Padding
  // 4: TypeIndex
  Refs.push_back({TiRefKind::TypeRef, Offset + 4, 1});
  return 8;
}

static void handleFieldList(ArrayRef<uint8_t> Content,
                            SmallVectorImpl<TiReference> &Refs) {
  uint32_t Offset = 0;
  uint32_t ThisLen = 0;
  while (!Content.empty()) {
    TypeLeafKind Kind =
        static_cast<TypeLeafKind>(support::endian::read16le(Content.data()));
    switch (Kind) {
    case LF_BCLASS:
      ThisLen = handleBaseClass(Content, Offset, Refs);
      break;
    case LF_ENUMERATE:
      ThisLen = handleEnumerator(Content, Offset, Refs);
      break;
    case LF_MEMBER:
      ThisLen = handleDataMember(Content, Offset, Refs);
      break;
    case LF_METHOD:
      ThisLen = handleOverloadedMethod(Content, Offset, Refs);
      break;
    case LF_ONEMETHOD:
      ThisLen = handleOneMethod(Content, Offset, Refs);
      break;
    case LF_NESTTYPE:
      ThisLen = handleNestedType(Content, Offset, Refs);
      break;
    case LF_STMEMBER:
      ThisLen = handleStaticDataMember(Content, Offset, Refs);
      break;
    case LF_VBCLASS:
    case LF_IVBCLASS:
      ThisLen =
```
- EN: This section centers on `handleVFPtr`, `handleListContinuation`, `handleFieldList` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `handleVFPtr`, `handleListContinuation`, `handleFieldList` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and iterates over collections, ranges, or records.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并遍历集合、区间或记录。

### Lines 241-300

```cpp
          handleVirtualBaseClass(Content, Offset, Kind == LF_VBCLASS, Refs);
      break;
    case LF_VFUNCTAB:
      ThisLen = handleVFPtr(Content, Offset, Refs);
      break;
    case LF_INDEX:
      ThisLen = handleListContinuation(Content, Offset, Refs);
      break;
    default:
      return;
    }
    Content = Content.drop_front(ThisLen);
    Offset += ThisLen;
    if (!Content.empty()) {
      uint8_t Pad = Content.front();
      if (Pad >= LF_PAD0) {
        uint32_t Skip = Pad & 0x0F;
        Content = Content.drop_front(Skip);
        Offset += Skip;
      }
    }
  }
}

static void handlePointer(ArrayRef<uint8_t> Content,
                          SmallVectorImpl<TiReference> &Refs) {
  Refs.push_back({TiRefKind::TypeRef, 0, 1});

  uint32_t Attrs = support::endian::read32le(Content.drop_front(4).data());
  if (isMemberPointer(Attrs))
    Refs.push_back({TiRefKind::TypeRef, 8, 1});
}

static void discoverTypeIndices(ArrayRef<uint8_t> Content, TypeLeafKind Kind,
                                SmallVectorImpl<TiReference> &Refs) {
  uint32_t Count;
  // FIXME: In the future it would be nice if we could avoid hardcoding these
  // values.  One idea is to define some structures representing these types
  // that would allow the use of offsetof().
  switch (Kind) {
  case TypeLeafKind::LF_FUNC_ID:
    Refs.push_back({TiRefKind::IndexRef, 0, 1});
    Refs.push_back({TiRefKind::TypeRef, 4, 1});
    break;
  case TypeLeafKind::LF_MFUNC_ID:
    Refs.push_back({TiRefKind::TypeRef, 0, 2});
    break;
  case TypeLeafKind::LF_STRING_ID:
    Refs.push_back({TiRefKind::IndexRef, 0, 1});
    break;
  case TypeLeafKind::LF_SUBSTR_LIST:
    Count = support::endian::read32le(Content.data());
    if (Count > 0)
      Refs.push_back({TiRefKind::IndexRef, 4, Count});
    break;
  case TypeLeafKind::LF_BUILDINFO:
    Count = support::endian::read16le(Content.data());
    if (Count > 0)
      Refs.push_back({TiRefKind::IndexRef, 2, Count});
    break;
```
- EN: This section centers on `handleVirtualBaseClass`, `handlePointer`, `discoverTypeIndices` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `handleVirtualBaseClass`, `handlePointer`, `discoverTypeIndices` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and checks conditions and handles edge cases.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并检查条件并处理边界情况。

### Lines 301-360

```cpp
  case TypeLeafKind::LF_UDT_SRC_LINE:
    Refs.push_back({TiRefKind::TypeRef, 0, 1});
    Refs.push_back({TiRefKind::IndexRef, 4, 1});
    break;
  case TypeLeafKind::LF_UDT_MOD_SRC_LINE:
    Refs.push_back({TiRefKind::TypeRef, 0, 1});
    break;
  case TypeLeafKind::LF_MODIFIER:
    Refs.push_back({TiRefKind::TypeRef, 0, 1});
    break;
  case TypeLeafKind::LF_PROCEDURE:
    Refs.push_back({TiRefKind::TypeRef, 0, 1});
    Refs.push_back({TiRefKind::TypeRef, 8, 1});
    break;
  case TypeLeafKind::LF_MFUNCTION:
    Refs.push_back({TiRefKind::TypeRef, 0, 3});
    Refs.push_back({TiRefKind::TypeRef, 16, 1});
    break;
  case TypeLeafKind::LF_ARGLIST:
    Count = support::endian::read32le(Content.data());
    if (Count > 0)
      Refs.push_back({TiRefKind::TypeRef, 4, Count});
    break;
  case TypeLeafKind::LF_ARRAY:
    Refs.push_back({TiRefKind::TypeRef, 0, 2});
    break;
  case TypeLeafKind::LF_CLASS:
  case TypeLeafKind::LF_STRUCTURE:
  case TypeLeafKind::LF_INTERFACE:
    Refs.push_back({TiRefKind::TypeRef, 4, 3});
    break;
  case TypeLeafKind::LF_UNION:
    Refs.push_back({TiRefKind::TypeRef, 4, 1});
    break;
  case TypeLeafKind::LF_ENUM:
    Refs.push_back({TiRefKind::TypeRef, 4, 2});
    break;
  case TypeLeafKind::LF_BITFIELD:
    Refs.push_back({TiRefKind::TypeRef, 0, 1});
    break;
  case TypeLeafKind::LF_VFTABLE:
    Refs.push_back({TiRefKind::TypeRef, 0, 2});
    break;
  case TypeLeafKind::LF_VTSHAPE:
    break;
  case TypeLeafKind::LF_METHODLIST:
    handleMethodOverloadList(Content, Refs);
    break;
  case TypeLeafKind::LF_FIELDLIST:
    handleFieldList(Content, Refs);
    break;
  case TypeLeafKind::LF_POINTER:
    handlePointer(Content, Refs);
    break;
  default:
    break;
  }
}

static bool discoverTypeIndices(ArrayRef<uint8_t> Content, SymbolKind Kind,
```
- EN: This section centers on `handleMethodOverloadList`, `handleFieldList`, `handlePointer` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `handleMethodOverloadList`, `handleFieldList`, `handlePointer` 等符号展开，负责实现局部控制流程与状态维护。
- EN: This range checks conditions and handles edge cases.
  CN: 这一段检查条件并处理边界情况。

### Lines 361-420

```cpp
                                SmallVectorImpl<TiReference> &Refs) {
  uint32_t Count;
  // FIXME: In the future it would be nice if we could avoid hardcoding these
  // values.  One idea is to define some structures representing these types
  // that would allow the use of offsetof().
  switch (Kind) {
  case SymbolKind::S_GPROC32_ID:
  case SymbolKind::S_LPROC32_ID:
  case SymbolKind::S_LPROC32_DPC:
  case SymbolKind::S_LPROC32_DPC_ID:
    Refs.push_back({TiRefKind::IndexRef, 24, 1}); // LF_FUNC_ID
    break;
  case SymbolKind::S_GPROC32:
  case SymbolKind::S_LPROC32:
    Refs.push_back({TiRefKind::TypeRef, 24, 1}); // Type
    break;
  case SymbolKind::S_UDT:
    Refs.push_back({TiRefKind::TypeRef, 0, 1}); // UDT
    break;
  case SymbolKind::S_GDATA32:
  case SymbolKind::S_LDATA32:
    Refs.push_back({TiRefKind::TypeRef, 0, 1}); // Type
    break;
  case SymbolKind::S_BUILDINFO:
    Refs.push_back({TiRefKind::IndexRef, 0, 1}); // Compile flags
    break;
  case SymbolKind::S_LTHREAD32:
  case SymbolKind::S_GTHREAD32:
    Refs.push_back({TiRefKind::TypeRef, 0, 1}); // Type
    break;
  case SymbolKind::S_FILESTATIC:
    Refs.push_back({TiRefKind::TypeRef, 0, 1}); // Type
    break;
  case SymbolKind::S_LOCAL:
    Refs.push_back({TiRefKind::TypeRef, 0, 1}); // Type
    break;
  case SymbolKind::S_REGISTER:
    Refs.push_back({TiRefKind::TypeRef, 0, 1}); // Type
    break;
  case SymbolKind::S_CONSTANT:
    Refs.push_back({TiRefKind::TypeRef, 0, 1}); // Type
    break;
  case SymbolKind::S_BPREL32:
  case SymbolKind::S_REGREL32:
  case SymbolKind::S_REGREL32_INDIR:
    Refs.push_back({TiRefKind::TypeRef, 4, 1}); // Type
    break;
  case SymbolKind::S_CALLSITEINFO:
    Refs.push_back({TiRefKind::TypeRef, 8, 1}); // Call signature
    break;
  case SymbolKind::S_CALLERS:
  case SymbolKind::S_CALLEES:
  case SymbolKind::S_INLINEES:
    // The record is a count followed by an array of type indices.
    Count = *reinterpret_cast<const ulittle32_t *>(Content.data());
    Refs.push_back({TiRefKind::IndexRef, 4, Count}); // Callees
    break;
  case SymbolKind::S_INLINESITE:
    Refs.push_back({TiRefKind::IndexRef, 8, 1}); // ID of inlinee
    break;
```
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and updates helper containers and temporary state.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并更新辅助容器和临时状态。

### Lines 421-480

```cpp
  case SymbolKind::S_HEAPALLOCSITE:
    Refs.push_back({TiRefKind::TypeRef, 8, 1}); // UDT allocated
    break;

  // Defranges don't have types, just registers and code offsets.
  case SymbolKind::S_DEFRANGE_REGISTER:
  case SymbolKind::S_DEFRANGE_REGISTER_REL:
  case SymbolKind::S_DEFRANGE_REGISTER_REL_INDIR:
  case SymbolKind::S_DEFRANGE_FRAMEPOINTER_REL:
  case SymbolKind::S_DEFRANGE_FRAMEPOINTER_REL_FULL_SCOPE:
  case SymbolKind::S_DEFRANGE_SUBFIELD_REGISTER:
  case SymbolKind::S_DEFRANGE_SUBFIELD:
    break;

  // No type references.
  case SymbolKind::S_LABEL32:
  case SymbolKind::S_OBJNAME:
  case SymbolKind::S_COMPILE:
  case SymbolKind::S_COMPILE2:
  case SymbolKind::S_COMPILE3:
  case SymbolKind::S_ENVBLOCK:
  case SymbolKind::S_BLOCK32:
  case SymbolKind::S_FRAMEPROC:
  case SymbolKind::S_THUNK32:
  case SymbolKind::S_FRAMECOOKIE:
  case SymbolKind::S_UNAMESPACE:
  case SymbolKind::S_ARMSWITCHTABLE:
    break;
  // Scope ending symbols.
  case SymbolKind::S_END:
  case SymbolKind::S_INLINESITE_END:
  case SymbolKind::S_PROC_ID_END:
    break;
  default:
    return false; // Unknown symbol.
  }
  return true;
}

void llvm::codeview::discoverTypeIndices(const CVType &Type,
                                         SmallVectorImpl<TiReference> &Refs) {
  ::discoverTypeIndices(Type.content(), Type.kind(), Refs);
}

static void resolveTypeIndexReferences(ArrayRef<uint8_t> RecordData,
                                       ArrayRef<TiReference> Refs,
                                       SmallVectorImpl<TypeIndex> &Indices) {
  Indices.clear();

  if (Refs.empty())
    return;

  RecordData = RecordData.drop_front(sizeof(RecordPrefix));

  BinaryStreamReader Reader(RecordData, llvm::endianness::little);
  for (const auto &Ref : Refs) {
    Reader.setOffset(Ref.Offset);
    FixedStreamArray<TypeIndex> Run;
    cantFail(Reader.readArray(Run, Ref.Count));
    Indices.append(Run.begin(), Run.end());
```
- EN: This section centers on `discoverTypeIndices`, `resolveTypeIndexReferences`, `Reader` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `discoverTypeIndices`, `resolveTypeIndexReferences`, `Reader` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and iterates over collections, ranges, or records.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并遍历集合、区间或记录。

### Lines 481-526

```cpp
  }
}

void llvm::codeview::discoverTypeIndices(const CVType &Type,
                                         SmallVectorImpl<TypeIndex> &Indices) {
  return discoverTypeIndices(Type.RecordData, Indices);
}

void llvm::codeview::discoverTypeIndices(ArrayRef<uint8_t> RecordData,
                                         SmallVectorImpl<TypeIndex> &Indices) {
  SmallVector<TiReference, 4> Refs;
  discoverTypeIndices(RecordData, Refs);
  resolveTypeIndexReferences(RecordData, Refs, Indices);
}

void llvm::codeview::discoverTypeIndices(ArrayRef<uint8_t> RecordData,
                                         SmallVectorImpl<TiReference> &Refs) {
  const RecordPrefix *P =
      reinterpret_cast<const RecordPrefix *>(RecordData.data());
  TypeLeafKind K = static_cast<TypeLeafKind>(uint16_t(P->RecordKind));
  ::discoverTypeIndices(RecordData.drop_front(sizeof(RecordPrefix)), K, Refs);
}

bool llvm::codeview::discoverTypeIndicesInSymbol(
    const CVSymbol &Sym, SmallVectorImpl<TiReference> &Refs) {
  SymbolKind K = Sym.kind();
  return ::discoverTypeIndices(Sym.content(), K, Refs);
}

bool llvm::codeview::discoverTypeIndicesInSymbol(
    ArrayRef<uint8_t> RecordData, SmallVectorImpl<TiReference> &Refs) {
  const RecordPrefix *P =
      reinterpret_cast<const RecordPrefix *>(RecordData.data());
  SymbolKind K = static_cast<SymbolKind>(uint16_t(P->RecordKind));
  return ::discoverTypeIndices(RecordData.drop_front(sizeof(RecordPrefix)), K,
                               Refs);
}

bool llvm::codeview::discoverTypeIndicesInSymbol(
    ArrayRef<uint8_t> RecordData, SmallVectorImpl<TypeIndex> &Indices) {
  SmallVector<TiReference, 2> Refs;
  if (!discoverTypeIndicesInSymbol(RecordData, Refs))
    return false;
  resolveTypeIndexReferences(RecordData, Refs, Indices);
  return true;
}
```
- EN: This section centers on `discoverTypeIndices`, `resolveTypeIndexReferences`, `discoverTypeIndicesInSymbol` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `discoverTypeIndices`, `resolveTypeIndexReferences`, `discoverTypeIndicesInSymbol` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code checks conditions and handles edge cases and updates helper containers and temporary state.
  CN: 在这一段中，代码检查条件并处理边界情况，并更新辅助容器和临时状态。

## Key Concepts / 关键概念
- Domain / 领域: CodeView debug information / CodeView 调试信息
- Core symbols / 核心符号: `getMethodKind`, `MethodKind`, `isIntroVirtual`, `getPointerMode` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/DebugInfo/CodeView/TypeIndexDiscovery.h`, `llvm/DebugInfo/CodeView/TypeRecord.h`, `llvm/ADT/ArrayRef.h`, `llvm/Support/Endian.h`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `getMethodKind`, `MethodKind`, `isIntroVirtual`, `getPointerMode`, `isMemberPointer`
