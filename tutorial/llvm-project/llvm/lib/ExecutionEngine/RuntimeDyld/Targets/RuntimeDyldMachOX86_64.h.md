# RuntimeDyldMachOX86_64.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/RuntimeDyld/Targets/RuntimeDyldMachOX86_64.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements RuntimeDyld, which loads object code into memory, applies relocations, and resolves symbols at runtime.
  - **CN**: 实现 RuntimeDyld，在运行时把目标代码装入内存、应用重定位并解析符号。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- RuntimeDyldMachOX86_64.h ---- MachO/X86_64 specific code. -*- C++ -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-15
```cpp

#ifndef LLVM_LIB_EXECUTIONENGINE_RUNTIMEDYLD_TARGETS_RUNTIMEDYLDMACHOX86_64_H
#define LLVM_LIB_EXECUTIONENGINE_RUNTIMEDYLD_TARGETS_RUNTIMEDYLDMACHOX86_64_H

#include "../RuntimeDyldMachO.h"

#define DEBUG_TYPE "dyld"

```
- **EN**: Pulls in the headers needed for this implementation, including `../RuntimeDyldMachO.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `../RuntimeDyldMachO.h`。

### Lines 16-23
```cpp
namespace llvm {

class RuntimeDyldMachOX86_64
    : public RuntimeDyldMachOCRTPBase<RuntimeDyldMachOX86_64> {
public:

  typedef uint64_t TargetPtrT;

```
- **EN**: Introduces declarations for `llvm`, `RuntimeDyldMachOX86_64`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `RuntimeDyldMachOX86_64` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 24-31
```cpp
  RuntimeDyldMachOX86_64(RuntimeDyld::MemoryManager &MM,
                         JITSymbolResolver &Resolver)
      : RuntimeDyldMachOCRTPBase(MM, Resolver) {}

  unsigned getMaxStubSize() const override { return 8; }

  Align getStubAlignment() override { return Align(8); }

```
- **EN**: Implements logic around `RuntimeDyldMachOX86_64`, `RuntimeDyldMachOCRTPBase`, `getMaxStubSize`, `getStubAlignment`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `RuntimeDyldMachOX86_64`, `RuntimeDyldMachOCRTPBase`, `getMaxStubSize`, `getStubAlignment` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 32-42
```cpp
  Expected<relocation_iterator>
  processRelocationRef(unsigned SectionID, relocation_iterator RelI,
                       const ObjectFile &BaseObjT,
                       ObjSectionToIDMap &ObjSectionToID,
                       StubMap &Stubs) override {
    const MachOObjectFile &Obj =
      static_cast<const MachOObjectFile &>(BaseObjT);
    MachO::any_relocation_info RelInfo =
        Obj.getRelocation(RelI->getRawDataRefImpl());
    uint32_t RelType = Obj.getAnyRelocationType(RelInfo);

```
- **EN**: Implements logic around `processRelocationRef`, `getRelocation`, `getAnyRelocationType`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `processRelocationRef`, `getRelocation`, `getAnyRelocationType` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 43-56
```cpp
    if (RelType == MachO::X86_64_RELOC_SUBTRACTOR)
      return processSubtractRelocation(SectionID, RelI, Obj, ObjSectionToID);

    assert(!Obj.isRelocationScattered(RelInfo) &&
           "Scattered relocations not supported on X86_64");

    RelocationEntry RE(getRelocationEntry(SectionID, Obj, RelI));
    RE.Addend = memcpyAddend(RE);
    RelocationValueRef Value;
    if (auto ValueOrErr = getRelocationValueRef(Obj, RelI, RE, ObjSectionToID))
      Value = *ValueOrErr;
    else
      return ValueOrErr.takeError();

```
- **EN**: Implements logic around `processSubtractRelocation`, `assert`, `RE`, `memcpyAddend`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `processSubtractRelocation`, `assert`, `RE`, `memcpyAddend`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 57-70
```cpp
    bool IsExtern = Obj.getPlainRelocationExternal(RelInfo);
    if (!IsExtern && RE.IsPCRel)
      makeValueAddendPCRel(Value, RelI, 1 << RE.Size);

    switch (RelType) {
    UNIMPLEMENTED_RELOC(MachO::X86_64_RELOC_TLV);
    default:
      if (RelType > MachO::X86_64_RELOC_TLV)
        return make_error<RuntimeDyldError>(("MachO X86_64 relocation type " +
                                             Twine(RelType) +
                                             " is out of range").str());
      break;
    }

```
- **EN**: Implements logic around `getPlainRelocationExternal`, `makeValueAddendPCRel`, `UNIMPLEMENTED_RELOC`, `make_error<RuntimeDyldError>`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getPlainRelocationExternal`, `makeValueAddendPCRel`, `UNIMPLEMENTED_RELOC`, `make_error<RuntimeDyldError>`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 71-81
```cpp
    if (RE.RelType == MachO::X86_64_RELOC_GOT ||
        RE.RelType == MachO::X86_64_RELOC_GOT_LOAD)
      processGOTRelocation(RE, Value, Stubs);
    else {
      RE.Addend = Value.Offset;
      if (Value.SymbolName)
        addRelocationForSymbol(RE, Value.SymbolName);
      else
        addRelocationForSection(RE, Value.SectionID);
    }

```
- **EN**: Implements logic around `processGOTRelocation`, `addRelocationForSymbol`, `addRelocationForSection`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `processGOTRelocation`, `addRelocationForSymbol`, `addRelocationForSection` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 82-89
```cpp
    return ++RelI;
  }

  void resolveRelocation(const RelocationEntry &RE, uint64_t Value) override {
    LLVM_DEBUG(dumpRelocationToResolve(RE, Value));
    const SectionEntry &Section = Sections[RE.SectionID];
    uint8_t *LocalAddress = Section.getAddressWithOffset(RE.Offset);

```
- **EN**: Implements logic around `resolveRelocation`, `getAddressWithOffset`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `resolveRelocation`, `getAddressWithOffset` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 90-98
```cpp
    // If the relocation is PC-relative, the value to be encoded is the
    // pointer difference.
    if (RE.IsPCRel) {
      // FIXME: It seems this value needs to be adjusted by 4 for an effective
      // PC address. Is that expected? Only for branches, perhaps?
      uint64_t FinalAddress = Section.getLoadAddressWithOffset(RE.Offset);
      Value -= FinalAddress + 4;
    }

```
- **EN**: Implements logic around `getLoadAddressWithOffset`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `getLoadAddressWithOffset` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 99-112
```cpp
    switch (RE.RelType) {
    default:
      llvm_unreachable("Invalid relocation type!");
    case MachO::X86_64_RELOC_SIGNED_1:
    case MachO::X86_64_RELOC_SIGNED_2:
    case MachO::X86_64_RELOC_SIGNED_4:
    case MachO::X86_64_RELOC_SIGNED:
    case MachO::X86_64_RELOC_UNSIGNED:
    case MachO::X86_64_RELOC_BRANCH:
      writeBytesUnaligned(Value + RE.Addend, LocalAddress, 1 << RE.Size);
      break;
    case MachO::X86_64_RELOC_SUBTRACTOR: {
      uint64_t SectionABase = Sections[RE.Sections.SectionA].getLoadAddress();
      uint64_t SectionBBase = Sections[RE.Sections.SectionB].getLoadAddress();
```
- **EN**: Implements logic around `llvm_unreachable`, `writeBytesUnaligned`, `getLoadAddress`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `llvm_unreachable`, `writeBytesUnaligned`, `getLoadAddress` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 113-121
```cpp
      assert((Value == SectionABase || Value == SectionBBase) &&
             "Unexpected SUBTRACTOR relocation value.");
      Value = SectionABase - SectionBBase + RE.Addend;
      writeBytesUnaligned(Value, LocalAddress, 1 << RE.Size);
      break;
    }
    }
  }

```
- **EN**: Implements logic around `assert`, `writeBytesUnaligned`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `assert`, `writeBytesUnaligned` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为。

### Lines 122-135
```cpp
  Error finalizeSection(const ObjectFile &Obj, unsigned SectionID,
                        const SectionRef &Section) {
    return Error::success();
  }

private:
  void processGOTRelocation(const RelocationEntry &RE,
                            RelocationValueRef &Value, StubMap &Stubs) {
    SectionEntry &Section = Sections[RE.SectionID];
    assert(RE.IsPCRel);
    assert(RE.Size == 2);
    Value.Offset -= RE.Addend;
    RuntimeDyldMachO::StubMap::const_iterator i = Stubs.find(Value);
    uint8_t *Addr;
```
- **EN**: Implements logic around `finalizeSection`, `success`, `processGOTRelocation`, `assert`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `finalizeSection`, `success`, `processGOTRelocation`, `assert`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 136-149
```cpp
    if (i != Stubs.end()) {
      Addr = Section.getAddressWithOffset(i->second);
    } else {
      Stubs[Value] = Section.getStubOffset();
      uint8_t *GOTEntry = Section.getAddressWithOffset(Section.getStubOffset());
      RelocationEntry GOTRE(RE.SectionID, Section.getStubOffset(),
                            MachO::X86_64_RELOC_UNSIGNED, Value.Offset, false,
                            3);
      if (Value.SymbolName)
        addRelocationForSymbol(GOTRE, Value.SymbolName);
      else
        addRelocationForSection(GOTRE, Value.SectionID);
      Section.advanceStubOffset(8);
      Addr = GOTEntry;
```
- **EN**: Implements logic around `getAddressWithOffset`, `getStubOffset`, `GOTRE`, `addRelocationForSymbol`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `getAddressWithOffset`, `getStubOffset`, `GOTRE`, `addRelocationForSymbol`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 150-163
```cpp
    }
    RelocationEntry TargetRE(RE.SectionID, RE.Offset,
                             MachO::X86_64_RELOC_UNSIGNED, RE.Addend, true, 2);
    resolveRelocation(TargetRE, (uint64_t)Addr);
  }

  Expected<relocation_iterator>
  processSubtractRelocation(unsigned SectionID, relocation_iterator RelI,
                            const MachOObjectFile &BaseObj,
                            ObjSectionToIDMap &ObjSectionToID) {
    const MachOObjectFile &Obj = BaseObj;
    MachO::any_relocation_info RE =
        Obj.getRelocation(RelI->getRawDataRefImpl());

```
- **EN**: Implements logic around `TargetRE`, `resolveRelocation`, `processSubtractRelocation`, `getRelocation`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `TargetRE`, `resolveRelocation`, `processSubtractRelocation`, `getRelocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 164-170
```cpp
    unsigned Size = Obj.getAnyRelocationLength(RE);
    uint64_t Offset = RelI->getOffset();
    uint8_t *LocalAddress = Sections[SectionID].getAddressWithOffset(Offset);
    unsigned NumBytes = 1 << Size;
    int64_t Addend =
      SignExtend64(readBytesUnaligned(LocalAddress, NumBytes), NumBytes * 8);

```
- **EN**: Implements logic around `getAnyRelocationLength`, `getOffset`, `getAddressWithOffset`, `SignExtend64`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `getAnyRelocationLength`, `getOffset`, `getAddressWithOffset`, `SignExtend64` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 171-178
```cpp
    unsigned SectionBID = ~0U;
    uint64_t SectionBOffset = 0;

    MachO::any_relocation_info RelInfo =
      Obj.getRelocation(RelI->getRawDataRefImpl());

    bool AIsExternal = BaseObj.getPlainRelocationExternal(RelInfo);

```
- **EN**: Implements logic around `getRelocation`, `getPlainRelocationExternal`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `getRelocation`, `getPlainRelocationExternal` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 179-192
```cpp
    if (AIsExternal) {
      Expected<StringRef> SubtrahendNameOrErr = RelI->getSymbol()->getName();
      if (!SubtrahendNameOrErr)
        return SubtrahendNameOrErr.takeError();
      auto SubtrahendI = GlobalSymbolTable.find(*SubtrahendNameOrErr);
      SectionBID = SubtrahendI->second.getSectionID();
      SectionBOffset = SubtrahendI->second.getOffset();
    } else {
      SectionRef SecB = Obj.getAnyRelocationSection(RelInfo);
      bool IsCode = SecB.isText();
      Expected<unsigned> SectionBIDOrErr =
        findOrEmitSection(Obj, SecB, IsCode, ObjSectionToID);
      if (!SectionBIDOrErr)
        return SectionBIDOrErr.takeError();
```
- **EN**: Implements logic around `getSymbol`, `takeError`, `find`, `getSectionID`, and 4 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getSymbol`, `takeError`, `find`, `getSectionID`, and 4 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 193-201
```cpp
      SectionBID = *SectionBIDOrErr;
      Addend += SecB.getAddress();
    }

    ++RelI;

    unsigned SectionAID = ~0U;
    uint64_t SectionAOffset = 0;

```
- **EN**: Implements logic around `getAddress`.
- **CN**: 围绕 `getAddress` 实现具体逻辑。

### Lines 202-215
```cpp
    RelInfo = Obj.getRelocation(RelI->getRawDataRefImpl());

    bool BIsExternal = BaseObj.getPlainRelocationExternal(RelInfo);
    if (BIsExternal) {
      Expected<StringRef> MinuendNameOrErr = RelI->getSymbol()->getName();
      if (!MinuendNameOrErr)
        return MinuendNameOrErr.takeError();
      auto MinuendI = GlobalSymbolTable.find(*MinuendNameOrErr);
      SectionAID = MinuendI->second.getSectionID();
      SectionAOffset = MinuendI->second.getOffset();
    } else {
      SectionRef SecA = Obj.getAnyRelocationSection(RelInfo);
      bool IsCode = SecA.isText();
      Expected<unsigned> SectionAIDOrErr =
```
- **EN**: Implements logic around `getRelocation`, `getPlainRelocationExternal`, `getSymbol`, `takeError`, and 5 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getRelocation`, `getPlainRelocationExternal`, `getSymbol`, `takeError`, and 5 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 216-222
```cpp
        findOrEmitSection(Obj, SecA, IsCode, ObjSectionToID);
      if (!SectionAIDOrErr)
        return SectionAIDOrErr.takeError();
      SectionAID = *SectionAIDOrErr;
      Addend -= SecA.getAddress();
    }

```
- **EN**: Implements logic around `findOrEmitSection`, `takeError`, `getAddress`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `findOrEmitSection`, `takeError`, `getAddress` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 223-231
```cpp
    RelocationEntry R(SectionID, Offset, MachO::X86_64_RELOC_SUBTRACTOR, (uint64_t)Addend,
                      SectionAID, SectionAOffset, SectionBID, SectionBOffset,
                      false, Size);

    addRelocationForSection(R, SectionAID);

    return ++RelI;
  }

```
- **EN**: Implements logic around `R`, `addRelocationForSection`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `R`, `addRelocationForSection` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 232-237
```cpp
};
}

#undef DEBUG_TYPE

#endif
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

## Key Concepts / 关键概念

- **Runtime relocation / 运行时重定位**:
  - **EN**: Loads object code into memory and resolves relocations against runtime symbol tables
  - **CN**: 把目标代码装入内存并针对运行时符号表解析重定位
- **Relocations and fixups / 重定位与 fixup**:
  - **EN**: Bridges symbolic references to concrete addresses or relocation records
  - **CN**: 把符号引用连接到具体地址或重定位记录
- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `../RuntimeDyldMachO.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine
