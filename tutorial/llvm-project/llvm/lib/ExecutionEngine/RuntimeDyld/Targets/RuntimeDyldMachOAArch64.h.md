# RuntimeDyldMachOAArch64.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/RuntimeDyld/Targets/RuntimeDyldMachOAArch64.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements RuntimeDyld, which loads object code into memory, applies relocations, and resolves symbols at runtime.
  - **CN**: 实现 RuntimeDyld，在运行时把目标代码装入内存、应用重定位并解析符号。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- RuntimeDyldMachOAArch64.h -- MachO/AArch64 specific code. -*- C++ -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_EXECUTIONENGINE_RUNTIMEDYLD_TARGETS_RUNTIMEDYLDMACHOAARCH64_H
#define LLVM_LIB_EXECUTIONENGINE_RUNTIMEDYLD_TARGETS_RUNTIMEDYLDMACHOAARCH64_H

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 12-22
```cpp
#include "../RuntimeDyldMachO.h"
#include "llvm/Support/Endian.h"

#define DEBUG_TYPE "dyld"

namespace llvm {

class RuntimeDyldMachOAArch64
    : public RuntimeDyldMachOCRTPBase<RuntimeDyldMachOAArch64> {
public:

```
- **EN**: Pulls in the headers needed for this implementation, including `../RuntimeDyldMachO.h`, `llvm/Support/Endian.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `../RuntimeDyldMachO.h`, `llvm/Support/Endian.h`。

### Lines 23-32
```cpp
  typedef uint64_t TargetPtrT;

  RuntimeDyldMachOAArch64(RuntimeDyld::MemoryManager &MM,
                          JITSymbolResolver &Resolver)
      : RuntimeDyldMachOCRTPBase(MM, Resolver) {}

  unsigned getMaxStubSize() const override { return 8; }

  Align getStubAlignment() override { return Align(8); }

```
- **EN**: Implements logic around `RuntimeDyldMachOAArch64`, `RuntimeDyldMachOCRTPBase`, `getMaxStubSize`, `getStubAlignment`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `RuntimeDyldMachOAArch64`, `RuntimeDyldMachOCRTPBase`, `getMaxStubSize`, `getStubAlignment` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 33-50
```cpp
  /// Extract the addend encoded in the instruction / memory location.
  Expected<int64_t> decodeAddend(const RelocationEntry &RE) const {
    const SectionEntry &Section = Sections[RE.SectionID];
    uint8_t *LocalAddress = Section.getAddressWithOffset(RE.Offset);
    unsigned NumBytes = 1 << RE.Size;
    int64_t Addend = 0;
    // Verify that the relocation has the correct size and alignment.
    switch (RE.RelType) {
    default: {
      std::string ErrMsg;
      {
        raw_string_ostream ErrStream(ErrMsg);
        ErrStream << "Unsupported relocation type: "
                  << getRelocName(RE.RelType);
      }
      return make_error<StringError>(std::move(ErrMsg),
                                     inconvertibleErrorCode());
    }
```
- **EN**: Implements logic around `decodeAddend`, `getAddressWithOffset`, `ErrStream`, `getRelocName`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `decodeAddend`, `getAddressWithOffset`, `ErrStream`, `getRelocName`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 51-68
```cpp
    case MachO::ARM64_RELOC_POINTER_TO_GOT:
    case MachO::ARM64_RELOC_UNSIGNED: {
      if (NumBytes != 4 && NumBytes != 8) {
        std::string ErrMsg;
        {
          raw_string_ostream ErrStream(ErrMsg);
          ErrStream << "Invalid relocation size for relocation "
                    << getRelocName(RE.RelType);
        }
        return make_error<StringError>(std::move(ErrMsg),
                                       inconvertibleErrorCode());
      }
      break;
    }
    case MachO::ARM64_RELOC_BRANCH26:
    case MachO::ARM64_RELOC_PAGE21:
    case MachO::ARM64_RELOC_PAGEOFF12:
    case MachO::ARM64_RELOC_GOT_LOAD_PAGE21:
```
- **EN**: Implements logic around `ErrStream`, `getRelocName`, `make_error<StringError>`, `inconvertibleErrorCode`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `ErrStream`, `getRelocName`, `make_error<StringError>`, `inconvertibleErrorCode` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 69-86
```cpp
    case MachO::ARM64_RELOC_GOT_LOAD_PAGEOFF12:
      assert(NumBytes == 4 && "Invalid relocation size.");
      assert((((uintptr_t)LocalAddress & 0x3) == 0) &&
             "Instruction address is not aligned to 4 bytes.");
      break;
    }

    switch (RE.RelType) {
    default:
      llvm_unreachable("Unsupported relocation type!");
    case MachO::ARM64_RELOC_POINTER_TO_GOT:
    case MachO::ARM64_RELOC_UNSIGNED:
      // This could be an unaligned memory location.
      if (NumBytes == 4)
        Addend = *reinterpret_cast<support::ulittle32_t *>(LocalAddress);
      else
        Addend = *reinterpret_cast<support::ulittle64_t *>(LocalAddress);
      break;
```
- **EN**: Implements logic around `assert`, `llvm_unreachable`; this block handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `assert`, `llvm_unreachable` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 87-104
```cpp
    case MachO::ARM64_RELOC_BRANCH26: {
      // Verify that the relocation points to a B/BL instruction.
      auto *p = reinterpret_cast<support::aligned_ulittle32_t *>(LocalAddress);
      assert(((*p & 0xFC000000) == 0x14000000 ||
              (*p & 0xFC000000) == 0x94000000) &&
             "Expected branch instruction.");

      // Get the 26 bit addend encoded in the branch instruction and sign-extend
      // to 64 bit. The lower 2 bits are always zeros and are therefore implicit
      // (<< 2).
      Addend = (*p & 0x03FFFFFF) << 2;
      Addend = SignExtend64(Addend, 28);
      break;
    }
    case MachO::ARM64_RELOC_GOT_LOAD_PAGE21:
    case MachO::ARM64_RELOC_PAGE21: {
      // Verify that the relocation points to the expected adrp instruction.
      auto *p = reinterpret_cast<support::aligned_ulittle32_t *>(LocalAddress);
```
- **EN**: Implements logic around `assert`, `SignExtend64`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `assert`, `SignExtend64` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 105-122
```cpp
      assert((*p & 0x9F000000) == 0x90000000 && "Expected adrp instruction.");

      // Get the 21 bit addend encoded in the adrp instruction and sign-extend
      // to 64 bit. The lower 12 bits (4096 byte page) are always zeros and are
      // therefore implicit (<< 12).
      Addend = ((*p & 0x60000000) >> 29) | ((*p & 0x01FFFFE0) >> 3) << 12;
      Addend = SignExtend64(Addend, 33);
      break;
    }
    case MachO::ARM64_RELOC_GOT_LOAD_PAGEOFF12: {
      // Verify that the relocation points to one of the expected load / store
      // instructions.
      auto *p = reinterpret_cast<support::aligned_ulittle32_t *>(LocalAddress);
      (void)p;
      assert((*p & 0x3B000000) == 0x39000000 &&
             "Only expected load / store instructions.");
      [[fallthrough]];
    }
```
- **EN**: Implements logic around `assert`, `SignExtend64`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `assert`, `SignExtend64` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 123-133
```cpp
    case MachO::ARM64_RELOC_PAGEOFF12: {
      // Verify that the relocation points to one of the expected load / store
      // or add / sub instructions.
      auto *p = reinterpret_cast<support::aligned_ulittle32_t *>(LocalAddress);
      assert((((*p & 0x3B000000) == 0x39000000) ||
              ((*p & 0x11C00000) == 0x11000000)   ) &&
             "Expected load / store  or add/sub instruction.");

      // Get the 12 bit addend encoded in the instruction.
      Addend = (*p & 0x003FFC00) >> 10;

```
- **EN**: Implements logic around `assert`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `assert` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 134-151
```cpp
      // Check which instruction we are decoding to obtain the implicit shift
      // factor of the instruction.
      int ImplicitShift = 0;
      if ((*p & 0x3B000000) == 0x39000000) { // << load / store
        // For load / store instructions the size is encoded in bits 31:30.
        ImplicitShift = ((*p >> 30) & 0x3);
        if (ImplicitShift == 0) {
          // Check if this a vector op to get the correct shift value.
          if ((*p & 0x04800000) == 0x04800000)
            ImplicitShift = 4;
        }
      }
      // Compensate for implicit shift.
      Addend <<= ImplicitShift;
      break;
    }
    }
    return Addend;
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 152-169
```cpp
  }

  /// Extract the addend encoded in the instruction.
  void encodeAddend(uint8_t *LocalAddress, unsigned NumBytes,
                    MachO::RelocationInfoType RelType, int64_t Addend) const {
    // Verify that the relocation has the correct alignment.
    switch (RelType) {
    default:
      llvm_unreachable("Unsupported relocation type!");
    case MachO::ARM64_RELOC_POINTER_TO_GOT:
    case MachO::ARM64_RELOC_UNSIGNED:
      assert((NumBytes == 4 || NumBytes == 8) && "Invalid relocation size.");
      break;
    case MachO::ARM64_RELOC_BRANCH26:
    case MachO::ARM64_RELOC_PAGE21:
    case MachO::ARM64_RELOC_PAGEOFF12:
    case MachO::ARM64_RELOC_GOT_LOAD_PAGE21:
    case MachO::ARM64_RELOC_GOT_LOAD_PAGEOFF12:
```
- **EN**: Implements logic around `encodeAddend`, `llvm_unreachable`, `assert`; this block handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `encodeAddend`, `llvm_unreachable`, `assert` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 170-187
```cpp
      assert(NumBytes == 4 && "Invalid relocation size.");
      assert((((uintptr_t)LocalAddress & 0x3) == 0) &&
             "Instruction address is not aligned to 4 bytes.");
      break;
    }

    switch (RelType) {
    default:
      llvm_unreachable("Unsupported relocation type!");
    case MachO::ARM64_RELOC_POINTER_TO_GOT:
    case MachO::ARM64_RELOC_UNSIGNED:
      // This could be an unaligned memory location.
      if (NumBytes == 4)
        *reinterpret_cast<support::ulittle32_t *>(LocalAddress) = Addend;
      else
        *reinterpret_cast<support::ulittle64_t *>(LocalAddress) = Addend;
      break;
    case MachO::ARM64_RELOC_BRANCH26: {
```
- **EN**: Implements logic around `assert`, `llvm_unreachable`; this block handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `assert`, `llvm_unreachable` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 188-197
```cpp
      auto *p = reinterpret_cast<support::aligned_ulittle32_t *>(LocalAddress);
      // Verify that the relocation points to the expected branch instruction.
      assert(((*p & 0xFC000000) == 0x14000000 ||
              (*p & 0xFC000000) == 0x94000000) &&
             "Expected branch instruction.");

      // Verify addend value.
      assert((Addend & 0x3) == 0 && "Branch target is not aligned");
      assert(isInt<28>(Addend) && "Branch target is out of range.");

```
- **EN**: Implements logic around `assert`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `assert` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 198-207
```cpp
      // Encode the addend as 26 bit immediate in the branch instruction.
      *p = (*p & 0xFC000000) | ((uint32_t)(Addend >> 2) & 0x03FFFFFF);
      break;
    }
    case MachO::ARM64_RELOC_GOT_LOAD_PAGE21:
    case MachO::ARM64_RELOC_PAGE21: {
      // Verify that the relocation points to the expected adrp instruction.
      auto *p = reinterpret_cast<support::aligned_ulittle32_t *>(LocalAddress);
      assert((*p & 0x9F000000) == 0x90000000 && "Expected adrp instruction.");

```
- **EN**: Implements logic around `assert`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `assert` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 208-225
```cpp
      // Check that the addend fits into 21 bits (+ 12 lower bits).
      assert((Addend & 0xFFF) == 0 && "ADRP target is not page aligned.");
      assert(isInt<33>(Addend) && "Invalid page reloc value.");

      // Encode the addend into the instruction.
      uint32_t ImmLoValue = ((uint64_t)Addend << 17) & 0x60000000;
      uint32_t ImmHiValue = ((uint64_t)Addend >> 9) & 0x00FFFFE0;
      *p = (*p & 0x9F00001F) | ImmHiValue | ImmLoValue;
      break;
    }
    case MachO::ARM64_RELOC_GOT_LOAD_PAGEOFF12: {
      // Verify that the relocation points to one of the expected load / store
      // instructions.
      auto *p = reinterpret_cast<support::aligned_ulittle32_t *>(LocalAddress);
      assert((*p & 0x3B000000) == 0x39000000 &&
             "Only expected load / store instructions.");
      (void)p;
      [[fallthrough]];
```
- **EN**: Implements logic around `assert`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `assert` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 226-234
```cpp
    }
    case MachO::ARM64_RELOC_PAGEOFF12: {
      // Verify that the relocation points to one of the expected load / store
      // or add / sub instructions.
      auto *p = reinterpret_cast<support::aligned_ulittle32_t *>(LocalAddress);
      assert((((*p & 0x3B000000) == 0x39000000) ||
              ((*p & 0x11C00000) == 0x11000000)   ) &&
             "Expected load / store  or add/sub instruction.");

```
- **EN**: Implements logic around `assert`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `assert` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 235-252
```cpp
      // Check which instruction we are decoding to obtain the implicit shift
      // factor of the instruction and verify alignment.
      int ImplicitShift = 0;
      if ((*p & 0x3B000000) == 0x39000000) { // << load / store
        // For load / store instructions the size is encoded in bits 31:30.
        ImplicitShift = ((*p >> 30) & 0x3);
        switch (ImplicitShift) {
        case 0:
          // Check if this a vector op to get the correct shift value.
          if ((*p & 0x04800000) == 0x04800000) {
            ImplicitShift = 4;
            assert(((Addend & 0xF) == 0) &&
                   "128-bit LDR/STR not 16-byte aligned.");
          }
          break;
        case 1:
          assert(((Addend & 0x1) == 0) && "16-bit LDR/STR not 2-byte aligned.");
          break;
```
- **EN**: Implements logic around `assert`; this block uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `assert` 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 253-264
```cpp
        case 2:
          assert(((Addend & 0x3) == 0) && "32-bit LDR/STR not 4-byte aligned.");
          break;
        case 3:
          assert(((Addend & 0x7) == 0) && "64-bit LDR/STR not 8-byte aligned.");
          break;
        }
      }
      // Compensate for implicit shift.
      Addend >>= ImplicitShift;
      assert(isUInt<12>(Addend) && "Addend cannot be encoded.");

```
- **EN**: Implements logic around `assert`.
- **CN**: 围绕 `assert` 实现具体逻辑。

### Lines 265-281
```cpp
      // Encode the addend into the instruction.
      *p = (*p & 0xFFC003FF) | ((uint32_t)(Addend << 10) & 0x003FFC00);
      break;
    }
    }
  }

  Expected<relocation_iterator>
  processRelocationRef(unsigned SectionID, relocation_iterator RelI,
                       const ObjectFile &BaseObjT,
                       ObjSectionToIDMap &ObjSectionToID,
                       StubMap &Stubs) override {
    const MachOObjectFile &Obj =
      static_cast<const MachOObjectFile &>(BaseObjT);
    MachO::any_relocation_info RelInfo =
        Obj.getRelocation(RelI->getRawDataRefImpl());

```
- **EN**: Implements logic around `processRelocationRef`, `getRelocation`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `processRelocationRef`, `getRelocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 282-299
```cpp
    if (Obj.isRelocationScattered(RelInfo))
      return make_error<RuntimeDyldError>("Scattered relocations not supported "
                                          "for MachO AArch64");

    // ARM64 has an ARM64_RELOC_ADDEND relocation type that carries an explicit
    // addend for the following relocation. If found: (1) store the associated
    // addend, (2) consume the next relocation, and (3) use the stored addend to
    // override the addend.
    int64_t ExplicitAddend = 0;
    if (Obj.getAnyRelocationType(RelInfo) == MachO::ARM64_RELOC_ADDEND) {
      assert(!Obj.getPlainRelocationExternal(RelInfo));
      assert(!Obj.getAnyRelocationPCRel(RelInfo));
      assert(Obj.getAnyRelocationLength(RelInfo) == 2);
      int64_t RawAddend = Obj.getPlainRelocationSymbolNum(RelInfo);
      // Sign-extend the 24-bit to 64-bit.
      ExplicitAddend = SignExtend64(RawAddend, 24);
      ++RelI;
      RelInfo = Obj.getRelocation(RelI->getRawDataRefImpl());
```
- **EN**: Implements logic around `make_error<RuntimeDyldError>`, `assert`, `getPlainRelocationSymbolNum`, `SignExtend64`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<RuntimeDyldError>`, `assert`, `getPlainRelocationSymbolNum`, `SignExtend64`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 300-315
```cpp
    }

    if (Obj.getAnyRelocationType(RelInfo) == MachO::ARM64_RELOC_SUBTRACTOR)
      return processSubtractRelocation(SectionID, RelI, Obj, ObjSectionToID);

    RelocationEntry RE(getRelocationEntry(SectionID, Obj, RelI));

    if (RE.RelType == MachO::ARM64_RELOC_POINTER_TO_GOT) {
      bool Valid =
          (RE.Size == 2 && RE.IsPCRel) || (RE.Size == 3 && !RE.IsPCRel);
      if (!Valid)
        return make_error<StringError>("ARM64_RELOC_POINTER_TO_GOT supports "
                                       "32-bit pc-rel or 64-bit absolute only",
                                       inconvertibleErrorCode());
    }

```
- **EN**: Implements logic around `processSubtractRelocation`, `RE`, `make_error<StringError>`, `inconvertibleErrorCode`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `processSubtractRelocation`, `RE`, `make_error<StringError>`, `inconvertibleErrorCode` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 316-325
```cpp
    if (auto Addend = decodeAddend(RE))
      RE.Addend = *Addend;
    else
      return Addend.takeError();

    assert((ExplicitAddend == 0 || RE.Addend == 0) && "Relocation has "\
      "ARM64_RELOC_ADDEND and embedded addend in the instruction.");
    if (ExplicitAddend)
      RE.Addend = ExplicitAddend;

```
- **EN**: Implements logic around `takeError`, `assert`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `takeError`, `assert` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 326-338
```cpp
    RelocationValueRef Value;
    if (auto ValueOrErr = getRelocationValueRef(Obj, RelI, RE, ObjSectionToID))
      Value = *ValueOrErr;
    else
      return ValueOrErr.takeError();

    bool IsExtern = Obj.getPlainRelocationExternal(RelInfo);
    if (RE.RelType == MachO::ARM64_RELOC_POINTER_TO_GOT) {
      // We'll take care of the offset in processGOTRelocation.
      Value.Offset = 0;
    } else if (!IsExtern && RE.IsPCRel)
      makeValueAddendPCRel(Value, RelI, 1 << RE.Size);

```
- **EN**: Implements logic around `takeError`, `getPlainRelocationExternal`, `makeValueAddendPCRel`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `takeError`, `getPlainRelocationExternal`, `makeValueAddendPCRel` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 339-351
```cpp
    RE.Addend = Value.Offset;

    if (RE.RelType == MachO::ARM64_RELOC_GOT_LOAD_PAGE21 ||
        RE.RelType == MachO::ARM64_RELOC_GOT_LOAD_PAGEOFF12 ||
        RE.RelType == MachO::ARM64_RELOC_POINTER_TO_GOT)
      processGOTRelocation(RE, Value, Stubs);
    else {
      if (Value.SymbolName)
        addRelocationForSymbol(RE, Value.SymbolName);
      else
        addRelocationForSection(RE, Value.SectionID);
    }

```
- **EN**: Implements logic around `processGOTRelocation`, `addRelocationForSymbol`, `addRelocationForSection`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `processGOTRelocation`, `addRelocationForSymbol`, `addRelocationForSection` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 352-362
```cpp
    return ++RelI;
  }

  void resolveRelocation(const RelocationEntry &RE, uint64_t Value) override {
    LLVM_DEBUG(dumpRelocationToResolve(RE, Value));

    const SectionEntry &Section = Sections[RE.SectionID];
    uint8_t *LocalAddress = Section.getAddressWithOffset(RE.Offset);
    MachO::RelocationInfoType RelType =
      static_cast<MachO::RelocationInfoType>(RE.RelType);

```
- **EN**: Implements logic around `resolveRelocation`, `getAddressWithOffset`, `RelocationInfoType>`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `resolveRelocation`, `getAddressWithOffset`, `RelocationInfoType>` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 363-372
```cpp
    switch (RelType) {
    default:
      llvm_unreachable("Invalid relocation type!");
    case MachO::ARM64_RELOC_UNSIGNED: {
      assert(!RE.IsPCRel && "PCRel and ARM64_RELOC_UNSIGNED not supported");
      // Mask in the target value a byte at a time (we don't have an alignment
      // guarantee for the target address, so this is safest).
      if (RE.Size < 2)
        llvm_unreachable("Invalid size for ARM64_RELOC_UNSIGNED");

```
- **EN**: Implements logic around `llvm_unreachable`, `assert`; this block handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `llvm_unreachable`, `assert` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 373-388
```cpp
      encodeAddend(LocalAddress, 1 << RE.Size, RelType, Value + RE.Addend);
      break;
    }

    case MachO::ARM64_RELOC_POINTER_TO_GOT: {
      assert(((RE.Size == 2 && RE.IsPCRel) || (RE.Size == 3 && !RE.IsPCRel)) &&
             "ARM64_RELOC_POINTER_TO_GOT only supports 32-bit pc-rel or 64-bit "
             "absolute");
      // Addend is the GOT entry address and RE.Offset the target of the
      // relocation.
      uint64_t Result =
          RE.IsPCRel ? (RE.Addend - RE.Offset) : (Value + RE.Addend);
      encodeAddend(LocalAddress, 1 << RE.Size, RelType, Result);
      break;
    }

```
- **EN**: Implements logic around `encodeAddend`, `assert`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `encodeAddend`, `assert` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 389-406
```cpp
    case MachO::ARM64_RELOC_BRANCH26: {
      assert(RE.IsPCRel && "not PCRel and ARM64_RELOC_BRANCH26 not supported");
      // Check if branch is in range.
      uint64_t FinalAddress = Section.getLoadAddressWithOffset(RE.Offset);
      int64_t PCRelVal = Value - FinalAddress + RE.Addend;
      encodeAddend(LocalAddress, /*Size=*/4, RelType, PCRelVal);
      break;
    }
    case MachO::ARM64_RELOC_GOT_LOAD_PAGE21:
    case MachO::ARM64_RELOC_PAGE21: {
      assert(RE.IsPCRel && "not PCRel and ARM64_RELOC_PAGE21 not supported");
      // Adjust for PC-relative relocation and offset.
      uint64_t FinalAddress = Section.getLoadAddressWithOffset(RE.Offset);
      int64_t PCRelVal =
        ((Value + RE.Addend) & (-4096)) - (FinalAddress & (-4096));
      encodeAddend(LocalAddress, /*Size=*/4, RelType, PCRelVal);
      break;
    }
```
- **EN**: Implements logic around `assert`, `getLoadAddressWithOffset`, `encodeAddend`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `assert`, `getLoadAddressWithOffset`, `encodeAddend` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 407-424
```cpp
    case MachO::ARM64_RELOC_GOT_LOAD_PAGEOFF12:
    case MachO::ARM64_RELOC_PAGEOFF12: {
      assert(!RE.IsPCRel && "PCRel and ARM64_RELOC_PAGEOFF21 not supported");
      // Add the offset from the symbol.
      Value += RE.Addend;
      // Mask out the page address and only use the lower 12 bits.
      Value &= 0xFFF;
      encodeAddend(LocalAddress, /*Size=*/4, RelType, Value);
      break;
    }
    case MachO::ARM64_RELOC_SUBTRACTOR: {
      uint64_t SectionABase = Sections[RE.Sections.SectionA].getLoadAddress();
      uint64_t SectionBBase = Sections[RE.Sections.SectionB].getLoadAddress();
      assert((Value == SectionABase || Value == SectionBBase) &&
             "Unexpected SUBTRACTOR relocation value.");
      Value = SectionABase - SectionBBase + RE.Addend;
      writeBytesUnaligned(Value, LocalAddress, 1 << RE.Size);
      break;
```
- **EN**: Implements logic around `assert`, `encodeAddend`, `getLoadAddress`, `writeBytesUnaligned`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `assert`, `encodeAddend`, `getLoadAddress`, `writeBytesUnaligned` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为。

### Lines 425-435
```cpp
    }

    case MachO::ARM64_RELOC_TLVP_LOAD_PAGE21:
    case MachO::ARM64_RELOC_TLVP_LOAD_PAGEOFF12:
      llvm_unreachable("Relocation type not yet implemented!");
    case MachO::ARM64_RELOC_ADDEND:
      llvm_unreachable("ARM64_RELOC_ADDEND should have been handeled by "
                       "processRelocationRef!");
    }
  }

```
- **EN**: Implements logic around `llvm_unreachable`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `llvm_unreachable` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 436-453
```cpp
  Error finalizeSection(const ObjectFile &Obj, unsigned SectionID,
                       const SectionRef &Section) {
    return Error::success();
  }

private:
  void processGOTRelocation(const RelocationEntry &RE,
                            RelocationValueRef &Value, StubMap &Stubs) {
    assert((RE.RelType == MachO::ARM64_RELOC_POINTER_TO_GOT &&
            (RE.Size == 2 || RE.Size == 3)) ||
           RE.Size == 2);
    SectionEntry &Section = Sections[RE.SectionID];
    auto [It, Inserted] = Stubs.try_emplace(Value);
    int64_t Offset;
    if (!Inserted)
      Offset = static_cast<int64_t>(It->second);
    else {
      // FIXME: There must be a better way to do this then to check and fix the
```
- **EN**: Implements logic around `finalizeSection`, `success`, `processGOTRelocation`, `assert`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `finalizeSection`, `success`, `processGOTRelocation`, `assert`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 454-466
```cpp
      // alignment every time!!!
      uintptr_t BaseAddress = uintptr_t(Section.getAddress());
      uintptr_t StubAlignment = getStubAlignment().value();
      uintptr_t StubAddress =
          (BaseAddress + Section.getStubOffset() + StubAlignment - 1) &
          -StubAlignment;
      unsigned StubOffset = StubAddress - BaseAddress;
      It->second = StubOffset;
      assert(isAligned(getStubAlignment(), StubAddress) &&
             "GOT entry not aligned");
      RelocationEntry GOTRE(RE.SectionID, StubOffset,
                            MachO::ARM64_RELOC_UNSIGNED, Value.Offset,
                            /*IsPCRel=*/false, /*Size=*/3);
```
- **EN**: Implements logic around `uintptr_t`, `getStubAlignment`, `getStubOffset`, `assert`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `uintptr_t`, `getStubAlignment`, `getStubOffset`, `assert`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 467-478
```cpp
      if (Value.SymbolName)
        addRelocationForSymbol(GOTRE, Value.SymbolName);
      else
        addRelocationForSection(GOTRE, Value.SectionID);
      Section.advanceStubOffset(getMaxStubSize());
      Offset = static_cast<int64_t>(StubOffset);
    }
    RelocationEntry TargetRE(RE.SectionID, RE.Offset, RE.RelType, Offset,
                             RE.IsPCRel, RE.Size);
    addRelocationForSection(TargetRE, RE.SectionID);
  }

```
- **EN**: Implements logic around `addRelocationForSymbol`, `addRelocationForSection`, `advanceStubOffset`, `static_cast<int64_t>`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `addRelocationForSymbol`, `addRelocationForSection`, `advanceStubOffset`, `static_cast<int64_t>`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 479-487
```cpp
  Expected<relocation_iterator>
  processSubtractRelocation(unsigned SectionID, relocation_iterator RelI,
                            const ObjectFile &BaseObjT,
                            ObjSectionToIDMap &ObjSectionToID) {
    const MachOObjectFile &Obj =
        static_cast<const MachOObjectFile&>(BaseObjT);
    MachO::any_relocation_info RE =
        Obj.getRelocation(RelI->getRawDataRefImpl());

```
- **EN**: Implements logic around `processSubtractRelocation`, `getRelocation`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `processSubtractRelocation`, `getRelocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 488-501
```cpp
    unsigned Size = Obj.getAnyRelocationLength(RE);
    uint64_t Offset = RelI->getOffset();
    uint8_t *LocalAddress = Sections[SectionID].getAddressWithOffset(Offset);
    unsigned NumBytes = 1 << Size;

    Expected<StringRef> SubtrahendNameOrErr = RelI->getSymbol()->getName();
    if (!SubtrahendNameOrErr)
      return SubtrahendNameOrErr.takeError();
    auto SubtrahendI = GlobalSymbolTable.find(*SubtrahendNameOrErr);
    unsigned SectionBID = SubtrahendI->second.getSectionID();
    uint64_t SectionBOffset = SubtrahendI->second.getOffset();
    int64_t Addend =
      SignExtend64(readBytesUnaligned(LocalAddress, NumBytes), NumBytes * 8);

```
- **EN**: Implements logic around `getAnyRelocationLength`, `getOffset`, `getAddressWithOffset`, `getSymbol`, and 4 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getAnyRelocationLength`, `getOffset`, `getAddressWithOffset`, `getSymbol`, and 4 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 502-513
```cpp
    ++RelI;
    Expected<StringRef> MinuendNameOrErr = RelI->getSymbol()->getName();
    if (!MinuendNameOrErr)
      return MinuendNameOrErr.takeError();
    auto MinuendI = GlobalSymbolTable.find(*MinuendNameOrErr);
    unsigned SectionAID = MinuendI->second.getSectionID();
    uint64_t SectionAOffset = MinuendI->second.getOffset();

    RelocationEntry R(SectionID, Offset, MachO::ARM64_RELOC_SUBTRACTOR, (uint64_t)Addend,
                      SectionAID, SectionAOffset, SectionBID, SectionBOffset,
                      false, Size);

```
- **EN**: Implements logic around `getSymbol`, `takeError`, `find`, `getSectionID`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getSymbol`, `takeError`, `find`, `getSectionID`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 514-531
```cpp
    addRelocationForSection(R, SectionAID);

    return ++RelI;
  }

  static const char *getRelocName(uint32_t RelocType) {
    switch (RelocType) {
      case MachO::ARM64_RELOC_UNSIGNED: return "ARM64_RELOC_UNSIGNED";
      case MachO::ARM64_RELOC_SUBTRACTOR: return "ARM64_RELOC_SUBTRACTOR";
      case MachO::ARM64_RELOC_BRANCH26: return "ARM64_RELOC_BRANCH26";
      case MachO::ARM64_RELOC_PAGE21: return "ARM64_RELOC_PAGE21";
      case MachO::ARM64_RELOC_PAGEOFF12: return "ARM64_RELOC_PAGEOFF12";
      case MachO::ARM64_RELOC_GOT_LOAD_PAGE21: return "ARM64_RELOC_GOT_LOAD_PAGE21";
      case MachO::ARM64_RELOC_GOT_LOAD_PAGEOFF12: return "ARM64_RELOC_GOT_LOAD_PAGEOFF12";
      case MachO::ARM64_RELOC_POINTER_TO_GOT: return "ARM64_RELOC_POINTER_TO_GOT";
      case MachO::ARM64_RELOC_TLVP_LOAD_PAGE21: return "ARM64_RELOC_TLVP_LOAD_PAGE21";
      case MachO::ARM64_RELOC_TLVP_LOAD_PAGEOFF12: return "ARM64_RELOC_TLVP_LOAD_PAGEOFF12";
      case MachO::ARM64_RELOC_ADDEND: return "ARM64_RELOC_ADDEND";
```
- **EN**: Implements logic around `addRelocationForSection`, `getRelocName`; this block handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `addRelocationForSection`, `getRelocName` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 532-540
```cpp
    }
    return "Unrecognized arm64 addend";
  }

};
}

#undef DEBUG_TYPE

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 541-541
```cpp
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

- **Direct includes / 直接包含**: `../RuntimeDyldMachO.h`, `llvm/Support/Endian.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Support
