# ContinuationRecordBuilder.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/DebugInfo/CodeView/ContinuationRecordBuilder.cpp`
- Repository: `llvm-project`
- Purpose (EN): Seed the first record with an appropriate record prefix.
- Purpose (CN): 该文件位于 LLVM 的 `DebugInfo/CodeView` 目录中，主要实现与 `ContinuationRecordBuilder` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
#include "llvm/DebugInfo/CodeView/ContinuationRecordBuilder.h"

using namespace llvm;
using namespace llvm::codeview;

namespace {
struct ContinuationRecord {
  ulittle16_t Kind{uint16_t(TypeLeafKind::LF_INDEX)};
  ulittle16_t Size{0};
  ulittle32_t IndexRef{0xB0C0B0C0};
};

struct SegmentInjection {
  SegmentInjection(TypeLeafKind Kind) { Prefix.RecordKind = Kind; }

  ContinuationRecord Cont;
  RecordPrefix Prefix;
};
} // namespace

static void addPadding(BinaryStreamWriter &Writer) {
  uint32_t Align = Writer.getOffset() % 4;
  if (Align == 0)
    return;

  int PaddingBytes = 4 - Align;
  while (PaddingBytes > 0) {
    uint8_t Pad = static_cast<uint8_t>(LF_PAD0 + PaddingBytes);
    cantFail(Writer.writeInteger(Pad));
    --PaddingBytes;
  }
}

static SegmentInjection InjectFieldList(TypeLeafKind::LF_FIELDLIST);
static SegmentInjection InjectMethodOverloadList(TypeLeafKind::LF_METHODLIST);

static constexpr uint32_t ContinuationLength = sizeof(ContinuationRecord);
static constexpr uint32_t MaxSegmentLength =
    MaxRecordLength - ContinuationLength;

```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 1 direct dependencies, including `llvm/DebugInfo/CodeView/ContinuationRecordBuilder.h`.
  CN: 引入了 1 个直接依赖，其中包括 `llvm/DebugInfo/CodeView/ContinuationRecordBuilder.h`。
- EN: This section centers on `SegmentInjection`, `addPadding`, `cantFail` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `SegmentInjection`, `addPadding`, `cantFail` 等符号展开，负责为下游使用方写出、格式化或序列化结果。

### Lines 41-80

```cpp
static inline TypeLeafKind getTypeLeafKind(ContinuationRecordKind CK) {
  return (CK == ContinuationRecordKind::FieldList) ? LF_FIELDLIST
                                                   : LF_METHODLIST;
}

ContinuationRecordBuilder::ContinuationRecordBuilder()
    : SegmentWriter(Buffer), Mapping(SegmentWriter) {}

ContinuationRecordBuilder::~ContinuationRecordBuilder() = default;

void ContinuationRecordBuilder::begin(ContinuationRecordKind RecordKind) {
  assert(!Kind);
  Kind = RecordKind;
  Buffer.clear();
  SegmentWriter.setOffset(0);
  SegmentOffsets.clear();
  SegmentOffsets.push_back(0);
  assert(SegmentWriter.getOffset() == 0);
  assert(SegmentWriter.getLength() == 0);

  const SegmentInjection *FLI =
      (RecordKind == ContinuationRecordKind::FieldList)
          ? &InjectFieldList
          : &InjectMethodOverloadList;
  const uint8_t *FLIB = reinterpret_cast<const uint8_t *>(FLI);
  InjectedSegmentBytes =
      ArrayRef<uint8_t>(FLIB, FLIB + sizeof(SegmentInjection));

  // Seed the first record with an appropriate record prefix.
  RecordPrefix Prefix(getTypeLeafKind(RecordKind));
  CVType Type(&Prefix, sizeof(Prefix));
  cantFail(Mapping.visitTypeBegin(Type));

  cantFail(SegmentWriter.writeObject(Prefix));
}

template <typename RecordType>
void ContinuationRecordBuilder::writeMemberType(RecordType &Record) {
  assert(Kind);

```
- EN: This section centers on `getTypeLeafKind`, `ContinuationRecordBuilder`, `begin` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getTypeLeafKind`, `ContinuationRecordBuilder`, `begin` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code propagates LLVM-style errors and invariants and returns the resulting value to its callers.
  CN: 在这一段中，代码传播 LLVM 风格的错误处理并维护不变量，并将结果返回给调用方。

### Lines 81-120

```cpp
  uint32_t OriginalOffset = SegmentWriter.getOffset();
  CVMemberRecord CVMR;
  CVMR.Kind = static_cast<TypeLeafKind>(Record.getKind());

  // Member Records aren't length-prefixed, they only have a 2-byte TypeLeafKind
  // at the beginning.
  cantFail(SegmentWriter.writeEnum(CVMR.Kind));

  // Let the Mapping handle the rest.
  cantFail(Mapping.visitMemberBegin(CVMR));
  cantFail(Mapping.visitKnownMember(CVMR, Record));
  cantFail(Mapping.visitMemberEnd(CVMR));

  // Make sure it's padded to 4 bytes.
  addPadding(SegmentWriter);
  assert(getCurrentSegmentLength() % 4 == 0);

  // The maximum length of a single segment is 64KB minus the size to insert a
  // continuation.  So if we are over that, inject a continuation between the
  // previous member and the member that was just written, then end the previous
  // segment after the continuation and begin a new one with the just-written
  // member.
  if (getCurrentSegmentLength() > MaxSegmentLength) {
    // We need to inject some bytes before the member we just wrote but after
    // the previous member.  Save off the length of the member we just wrote so
    // that we can do validate it.
    uint32_t MemberLength = SegmentWriter.getOffset() - OriginalOffset;
    (void) MemberLength;
    insertSegmentEnd(OriginalOffset);
    // Since this member now becomes a new top-level record, it should have
    // gotten a RecordPrefix injected, and that RecordPrefix + the member we
    // just wrote should now constitute the entirety of the current "new"
    // segment.
    assert(getCurrentSegmentLength() == MemberLength + sizeof(RecordPrefix));
  }

  assert(getCurrentSegmentLength() % 4 == 0);
  assert(getCurrentSegmentLength() <= MaxSegmentLength);
}

```
- EN: This section centers on `cantFail`, `addPadding`, `assert` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `cantFail`, `addPadding`, `assert` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 121-160

```cpp
uint32_t ContinuationRecordBuilder::getCurrentSegmentLength() const {
  return SegmentWriter.getOffset() - SegmentOffsets.back();
}

void ContinuationRecordBuilder::insertSegmentEnd(uint32_t Offset) {
  uint32_t SegmentBegin = SegmentOffsets.back();
  (void)SegmentBegin;
  assert(Offset > SegmentBegin);
  assert(Offset - SegmentBegin <= MaxSegmentLength);

  // We need to make space for the continuation record.  For now we can't fill
  // out the length or the TypeIndex of the back-reference, but we need the
  // space to at least be there.
  Buffer.insert(Offset, InjectedSegmentBytes);

  uint32_t NewSegmentBegin = Offset + ContinuationLength;
  uint32_t SegmentLength = NewSegmentBegin - SegmentOffsets.back();
  (void) SegmentLength;

  assert(SegmentLength % 4 == 0);
  assert(SegmentLength <= MaxRecordLength);
  SegmentOffsets.push_back(NewSegmentBegin);

  // Seek to the end so that we can keep writing against the new segment.
  SegmentWriter.setOffset(SegmentWriter.getLength());
  assert(SegmentWriter.bytesRemaining() == 0);
}

CVType ContinuationRecordBuilder::createSegmentRecord(
    uint32_t OffBegin, uint32_t OffEnd, std::optional<TypeIndex> RefersTo) {
  assert(OffEnd - OffBegin <= USHRT_MAX);

  MutableArrayRef<uint8_t> Data = Buffer.data();
  Data = Data.slice(OffBegin, OffEnd - OffBegin);

  // Write the length to the RecordPrefix, making sure it does not include
  // sizeof(RecordPrefix.Length)
  RecordPrefix *Prefix = reinterpret_cast<RecordPrefix *>(Data.data());
  Prefix->RecordLen = Data.size() - sizeof(RecordPrefix::RecordLen);

```
- EN: This section centers on `getCurrentSegmentLength`, `insertSegmentEnd`, `assert` and queries existing state and returns computed metadata.
  CN: 这一段主要围绕 `getCurrentSegmentLength`, `insertSegmentEnd`, `assert` 等符号展开，负责查询现有状态并返回计算出的元数据。
- EN: In this range, the code propagates LLVM-style errors and invariants and returns the resulting value to its callers.
  CN: 在这一段中，代码传播 LLVM 风格的错误处理并维护不变量，并将结果返回给调用方。

### Lines 161-200

```cpp
  if (RefersTo) {
    auto Continuation = Data.take_back(ContinuationLength);
    ContinuationRecord *CR =
        reinterpret_cast<ContinuationRecord *>(Continuation.data());
    assert(CR->Kind == TypeLeafKind::LF_INDEX);
    assert(CR->IndexRef == 0xB0C0B0C0);
    CR->IndexRef = RefersTo->getIndex();
  }

  return CVType(Data);
}

std::vector<CVType> ContinuationRecordBuilder::end(TypeIndex Index) {
  RecordPrefix Prefix(getTypeLeafKind(*Kind));
  CVType Type(&Prefix, sizeof(Prefix));
  cantFail(Mapping.visitTypeEnd(Type));

  // We're now done, and we have a series of segments each beginning at an
  // offset specified in the SegmentOffsets array.  We now need to iterate
  // over each segment and post-process them in the following two ways:
  // 1) Each top-level record has a RecordPrefix whose type is either
  //    LF_FIELDLIST or LF_METHODLIST, but the Length field is still 0.
  //    Those should all be set to the correct length now.
  // 2) Each continuation record has an IndexRef field which we set to the
  //    magic value 0xB0C0B0C0.  Now that the caller has told us the TypeIndex
  //    they want this sequence to start from, we can go through and update
  //    each one.
  //
  // Logically, the sequence of records we've built up looks like this:
  //
  // SegmentOffsets[0]:   <Length>                    (Initially: uninitialized)
  // SegmentOffsets[0]+2: LF_FIELDLIST
  // SegmentOffsets[0]+4: Member[0]
  // SegmentOffsets[0]+?: ...
  // SegmentOffsets[0]+?: Member[4]
  // SegmentOffsets[1]-8: LF_INDEX
  // SegmentOffsets[1]-6: 0
  // SegmentOffsets[1]-4: <Type Index of Next Record> (Initially: 0xB0C0B0C0)
  //
  // SegmentOffsets[1]:   <Length>                    (Initially: uninitialized)
```
- EN: This section centers on `assert`, `CVType`, `end` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `assert`, `CVType`, `end` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 201-240

```cpp
  // SegmentOffsets[1]+2: LF_FIELDLIST
  // SegmentOffsets[1]+4: Member[0]
  // SegmentOffsets[1]+?: ...
  // SegmentOffsets[1]+?: Member[s]
  // SegmentOffsets[2]-8: LF_INDEX
  // SegmentOffsets[2]-6: 0
  // SegmentOffsets[2]-4: <Type Index of Next Record> (Initially: 0xB0C0B0C0)
  //
  // ...
  //
  // SegmentOffsets[N]:   <Length>                    (Initially: uninitialized)
  // SegmentOffsets[N]+2: LF_FIELDLIST
  // SegmentOffsets[N]+4: Member[0]
  // SegmentOffsets[N]+?: ...
  // SegmentOffsets[N]+?: Member[t]
  //
  // And this is the way we have laid them out in the serialization buffer.  But
  // we cannot actually commit them to the underlying stream this way, due to
  // the topological sorting requirement of a type stream (specifically,
  // TypeIndex references can only point backwards, not forwards).  So the
  // sequence that we return to the caller contains the records in reverse
  // order, which is the proper order for committing the serialized records.

  std::vector<CVType> Types;
  Types.reserve(SegmentOffsets.size());

  ArrayRef SO = SegmentOffsets;

  uint32_t End = SegmentWriter.getOffset();

  std::optional<TypeIndex> RefersTo;
  for (uint32_t Offset : reverse(SO)) {
    Types.push_back(createSegmentRecord(Offset, End, RefersTo));

    End = Offset;
    RefersTo = Index++;
  }

  Kind.reset();
  return Types;
```
- EN: In this range, the code iterates over collections, ranges, or records and updates helper containers and temporary state.
  CN: 在这一段中，代码遍历集合、区间或记录，并更新辅助容器和临时状态。

### Lines 241-252

```cpp
}

// Explicitly instantiate the member function for each known type so that we can
// implement this in the cpp file.
#define TYPE_RECORD(EnumName, EnumVal, Name)
#define TYPE_RECORD_ALIAS(EnumName, EnumVal, Name, AliasName)
#define MEMBER_RECORD(EnumName, EnumVal, Name)                                 \
  template LLVM_ABI void                                                       \
  llvm::codeview::ContinuationRecordBuilder::writeMemberType(                  \
      Name##Record &Record);
#define MEMBER_RECORD_ALIAS(EnumName, EnumVal, Name, AliasName)
#include "llvm/DebugInfo/CodeView/CodeViewTypes.def"
```
- EN: Brings in 1 direct dependencies, including `llvm/DebugInfo/CodeView/CodeViewTypes.def`.
  CN: 引入了 1 个直接依赖，其中包括 `llvm/DebugInfo/CodeView/CodeViewTypes.def`。
- EN: This section centers on `writeMemberType` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `writeMemberType` 等符号展开，负责为下游使用方写出、格式化或序列化结果。

## Key Concepts / 关键概念
- Domain / 领域: CodeView debug information / CodeView 调试信息
- Core symbols / 核心符号: `ContinuationRecord`, `SegmentInjection`, `addPadding`, `cantFail`, `InjectFieldList` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Formatting, emission, and outward serialization. / 重点关注格式化、输出与序列化。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/DebugInfo/CodeView/ContinuationRecordBuilder.h`, `llvm/DebugInfo/CodeView/CodeViewTypes.def`
- Standard library / 标准库: None / 无
- Other/system headers / 其他或系统头文件: None / 无
- Related symbols / 相关符号: `ContinuationRecord`, `SegmentInjection`, `addPadding`, `cantFail`, `InjectFieldList`, `InjectMethodOverloadList`
