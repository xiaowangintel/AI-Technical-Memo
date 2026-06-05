# RuntimeDyldCOFFAArch64.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/RuntimeDyld/Targets/RuntimeDyldCOFFAArch64.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements RuntimeDyld, which loads object code into memory, applies relocations, and resolves symbols at runtime.
  - **CN**: 实现 RuntimeDyld，在运行时把目标代码装入内存、应用重定位并解析符号。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- RuntimeDyldCOFFAArch64.h --- COFF/AArch64 specific code ---*- C++*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// COFF AArch64 support for MC-JIT runtime dynamic linker.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 12-21
```cpp

#ifndef LLVM_LIB_EXECUTIONENGINE_RUNTIMEDYLD_TARGETS_RUNTIMEDYLDCOFFAARCH64_H
#define LLVM_LIB_EXECUTIONENGINE_RUNTIMEDYLD_TARGETS_RUNTIMEDYLDCOFFAARCH64_H

#include "../RuntimeDyldCOFF.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/BinaryFormat/COFF.h"
#include "llvm/Object/COFF.h"
#include "llvm/Support/Endian.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `../RuntimeDyldCOFF.h`, `llvm/ADT/SmallString.h`, `llvm/BinaryFormat/COFF.h`, `llvm/Object/COFF.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `../RuntimeDyldCOFF.h`, `llvm/ADT/SmallString.h`, `llvm/BinaryFormat/COFF.h`, `llvm/Object/COFF.h`。

### Lines 22-31
```cpp
#define DEBUG_TYPE "dyld"

namespace llvm {

// This relocation type is used for handling long branch instruction
// through the Stub.
enum InternalRelocationType : unsigned {
  INTERNAL_REL_ARM64_LONG_BRANCH26 = 0x111,
};

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 32-42
```cpp
static void add16(uint8_t *p, int16_t v) {
  using namespace llvm::support::endian;
  write16le(p, read16le(p) + v);
}

static void or32le(void *P, int32_t V) {
  using namespace llvm::support::endian;

  write32le(P, read32le(P) | V);
}

```
- **EN**: Introduces declarations for `llvm::support::endian`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm::support::endian` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 43-53
```cpp
static void write32AArch64Imm(uint8_t *T, uint64_t imm, uint32_t rangeLimit) {
  using namespace llvm::support::endian;

  uint32_t orig = read32le(T);
  orig &= ~(0xFFF << 10);
  write32le(T, orig | ((imm & (0xFFF >> rangeLimit)) << 10));
}

static void write32AArch64Ldr(uint8_t *T, uint64_t imm) {
  using namespace llvm::support::endian;

```
- **EN**: Introduces declarations for `llvm::support::endian`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm::support::endian` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 54-64
```cpp
  uint32_t orig = read32le(T);
  uint32_t size = orig >> 30;
  // 0x04000000 indicates SIMD/FP registers
  // 0x00800000 indicates 128 bit
  if ((orig & 0x04800000) == 0x04800000)
    size += 4;
  if ((imm & ((1 << size) - 1)) != 0)
    assert(0 && "misaligned ldr/str offset");
  write32AArch64Imm(T, imm >> size, size);
}

```
- **EN**: Implements logic around `read32le`, `assert`, `write32AArch64Imm`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `read32le`, `assert`, `write32AArch64Imm` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 65-74
```cpp
static void write32AArch64Addr(void *T, uint64_t s, uint64_t p, int shift) {
  using namespace llvm::support::endian;

  uint64_t Imm = (s >> shift) - (p >> shift);
  uint32_t ImmLo = (Imm & 0x3) << 29;
  uint32_t ImmHi = (Imm & 0x1FFFFC) << 3;
  uint64_t Mask = (0x3 << 29) | (0x1FFFFC << 3);
  write32le(T, (read32le(T) & ~Mask) | ImmLo | ImmHi);
}

```
- **EN**: Introduces declarations for `llvm::support::endian`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm::support::endian` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 75-84
```cpp
class RuntimeDyldCOFFAArch64 : public RuntimeDyldCOFF {

private:
  // When a module is loaded we save the SectionID of the unwind
  // sections in a table until we receive a request to register all
  // unregisteredEH frame sections with the memory manager.
  SmallVector<SID, 2> UnregisteredEHFrameSections;
  SmallVector<SID, 2> RegisteredEHFrameSections;
  uint64_t ImageBase;

```
- **EN**: Introduces declarations for `RuntimeDyldCOFFAArch64`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `RuntimeDyldCOFFAArch64` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 85-100
```cpp
  // Fake an __ImageBase pointer by returning the section with the lowest adress
  uint64_t getImageBase() {
    if (!ImageBase) {
      ImageBase = std::numeric_limits<uint64_t>::max();
      for (const SectionEntry &Section : Sections)
        // The Sections list may contain sections that weren't loaded for
        // whatever reason: they may be debug sections, and ProcessAllSections
        // is false, or they may be sections that contain 0 bytes. If the
        // section isn't loaded, the load address will be 0, and it should not
        // be included in the ImageBase calculation.
        if (Section.getLoadAddress() != 0)
          ImageBase = std::min(ImageBase, Section.getLoadAddress());
    }
    return ImageBase;
  }

```
- **EN**: Implements logic around `getImageBase`, `max`, `min`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getImageBase`, `max`, `min` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 101-110
```cpp
public:
  RuntimeDyldCOFFAArch64(RuntimeDyld::MemoryManager &MM,
                         JITSymbolResolver &Resolver)
      : RuntimeDyldCOFF(MM, Resolver, 8, COFF::IMAGE_REL_ARM64_ADDR64),
        ImageBase(0) {}

  Align getStubAlignment() override { return Align(8); }

  unsigned getMaxStubSize() const override { return 20; }

```
- **EN**: Implements logic around `RuntimeDyldCOFFAArch64`, `RuntimeDyldCOFF`, `ImageBase`, `getStubAlignment`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `RuntimeDyldCOFFAArch64`, `RuntimeDyldCOFF`, `ImageBase`, `getStubAlignment`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 111-123
```cpp
  std::tuple<uint64_t, uint64_t, uint64_t>
  generateRelocationStub(unsigned SectionID, StringRef TargetName,
                         uint64_t Offset, uint64_t RelType, uint64_t Addend,
                         StubMap &Stubs) {
    uintptr_t StubOffset;
    SectionEntry &Section = Sections[SectionID];

    RelocationValueRef OriginalRelValueRef;
    OriginalRelValueRef.SectionID = SectionID;
    OriginalRelValueRef.Offset = Offset;
    OriginalRelValueRef.Addend = Addend;
    OriginalRelValueRef.SymbolName = TargetName.data();

```
- **EN**: Implements logic around `generateRelocationStub`, `data`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `generateRelocationStub`, `data` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 124-138
```cpp
    auto [Stub, Inserted] = Stubs.try_emplace(OriginalRelValueRef);
    if (Inserted) {
      LLVM_DEBUG(dbgs() << " Create a new stub function for "
                        << TargetName.data() << "\n");

      StubOffset = Section.getStubOffset();
      Stub->second = StubOffset;
      createStubFunction(Section.getAddressWithOffset(StubOffset));
      Section.advanceStubOffset(getMaxStubSize());
    } else {
      LLVM_DEBUG(dbgs() << " Stub function found for " << TargetName.data()
                        << "\n");
      StubOffset = Stub->second;
    }

```
- **EN**: Implements logic around `try_emplace`, `data`, `getStubOffset`, `createStubFunction`, and 1 more symbols.
- **CN**: 围绕 `try_emplace`, `data`, `getStubOffset`, `createStubFunction`, and 1 more symbols 实现具体逻辑。

### Lines 139-149
```cpp
    // Resolve original relocation to stub function.
    const RelocationEntry RE(SectionID, Offset, RelType, Addend);
    resolveRelocation(RE, Section.getLoadAddressWithOffset(StubOffset));

    // adjust relocation info so resolution writes to the stub function
    // Here an internal relocation type is used for resolving long branch via
    // stub instruction.
    Addend = 0;
    Offset = StubOffset;
    RelType = INTERNAL_REL_ARM64_LONG_BRANCH26;

```
- **EN**: Implements logic around `RE`, `resolveRelocation`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `RE`, `resolveRelocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为。

### Lines 150-159
```cpp
    return std::make_tuple(Offset, RelType, Addend);
  }

  Expected<object::relocation_iterator>
  processRelocationRef(unsigned SectionID, object::relocation_iterator RelI,
                       const object::ObjectFile &Obj,
                       ObjSectionToIDMap &ObjSectionToID,
                       StubMap &Stubs) override {
    using namespace llvm::support::endian;

```
- **EN**: Introduces declarations for `llvm::support::endian`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm::support::endian` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 160-168
```cpp
    auto Symbol = RelI->getSymbol();
    if (Symbol == Obj.symbol_end())
      report_fatal_error("Unknown symbol in relocation");

    Expected<StringRef> TargetNameOrErr = Symbol->getName();
    if (!TargetNameOrErr)
      return TargetNameOrErr.takeError();
    StringRef TargetName = *TargetNameOrErr;

```
- **EN**: Implements logic around `getSymbol`, `report_fatal_error`, `getName`, `takeError`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getSymbol`, `report_fatal_error`, `getName`, `takeError` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 169-179
```cpp
    auto SectionOrErr = Symbol->getSection();
    if (!SectionOrErr)
      return SectionOrErr.takeError();
    auto Section = *SectionOrErr;

    uint64_t RelType = RelI->getType();
    uint64_t Offset = RelI->getOffset();

    // If there is no section, this must be an external reference.
    bool IsExtern = Section == Obj.section_end();

```
- **EN**: Implements logic around `getSection`, `takeError`, `getType`, `getOffset`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getSection`, `takeError`, `getType`, `getOffset`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 180-188
```cpp
    // Determine the Addend used to adjust the relocation value.
    uint64_t Addend = 0;
    SectionEntry &AddendSection = Sections[SectionID];
    uintptr_t ObjTarget = AddendSection.getObjAddress() + Offset;
    uint8_t *Displacement = (uint8_t *)ObjTarget;

    unsigned TargetSectionID = -1;
    uint64_t TargetOffset = -1;

```
- **EN**: Implements logic around `getObjAddress`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `getObjAddress` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 189-200
```cpp
    if (TargetName.starts_with(getImportSymbolPrefix())) {
      TargetSectionID = SectionID;
      TargetOffset = getDLLImportOffset(SectionID, Stubs, TargetName);
      TargetName = StringRef();
      IsExtern = false;
    } else if (!IsExtern) {
      if (auto TargetSectionIDOrErr = findOrEmitSection(
              Obj, *Section, Section->isText(), ObjSectionToID))
        TargetSectionID = *TargetSectionIDOrErr;
      else
        return TargetSectionIDOrErr.takeError();

```
- **EN**: Implements logic around `getDLLImportOffset`, `StringRef`, `isText`, `takeError`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getDLLImportOffset`, `StringRef`, `isText`, `takeError` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 201-214
```cpp
      TargetOffset = getSymbolOffset(*Symbol);
    }

    switch (RelType) {
    case COFF::IMAGE_REL_ARM64_ADDR32:
    case COFF::IMAGE_REL_ARM64_ADDR32NB:
    case COFF::IMAGE_REL_ARM64_REL32:
    case COFF::IMAGE_REL_ARM64_SECREL:
      Addend = read32le(Displacement);
      break;
    case COFF::IMAGE_REL_ARM64_BRANCH26: {
      uint32_t orig = read32le(Displacement);
      Addend = (orig & 0x03FFFFFF) << 2;

```
- **EN**: Implements logic around `getSymbolOffset`, `read32le`; this block uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `getSymbolOffset`, `read32le` 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 215-232
```cpp
      if (IsExtern)
        std::tie(Offset, RelType, Addend) = generateRelocationStub(
            SectionID, TargetName, Offset, RelType, Addend, Stubs);
      break;
    }
    case COFF::IMAGE_REL_ARM64_BRANCH19: {
      uint32_t orig = read32le(Displacement);
      Addend = (orig & 0x00FFFFE0) >> 3;
      break;
    }
    case COFF::IMAGE_REL_ARM64_BRANCH14: {
      uint32_t orig = read32le(Displacement);
      Addend = (orig & 0x000FFFE0) >> 3;
      break;
    }
    case COFF::IMAGE_REL_ARM64_REL21:
    case COFF::IMAGE_REL_ARM64_PAGEBASE_REL21: {
      uint32_t orig = read32le(Displacement);
```
- **EN**: Implements logic around `tie`, `read32le`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `tie`, `read32le` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 233-249
```cpp
      Addend = ((orig >> 29) & 0x3) | ((orig >> 3) & 0x1FFFFC);
      break;
    }
    case COFF::IMAGE_REL_ARM64_PAGEOFFSET_12L:
    case COFF::IMAGE_REL_ARM64_PAGEOFFSET_12A: {
      uint32_t orig = read32le(Displacement);
      Addend = ((orig >> 10) & 0xFFF);
      break;
    }
    case COFF::IMAGE_REL_ARM64_ADDR64: {
      Addend = read64le(Displacement);
      break;
    }
    default:
      break;
    }

```
- **EN**: Implements logic around `read32le`, `read64le`.
- **CN**: 围绕 `read32le`, `read64le` 实现具体逻辑。

### Lines 250-258
```cpp
#if !defined(NDEBUG)
    SmallString<32> RelTypeName;
    RelI->getTypeName(RelTypeName);

    LLVM_DEBUG(dbgs() << "\t\tIn Section " << SectionID << " Offset " << Offset
                      << " RelType: " << RelTypeName << " TargetName: "
                      << TargetName << " Addend " << Addend << "\n");
#endif

```
- **EN**: Implements logic around `getTypeName`.
- **CN**: 围绕 `getTypeName` 实现具体逻辑。

### Lines 259-268
```cpp
    if (IsExtern) {
      RelocationEntry RE(SectionID, Offset, RelType, Addend);
      addRelocationForSymbol(RE, TargetName);
    } else {
      RelocationEntry RE(SectionID, Offset, RelType, TargetOffset + Addend);
      addRelocationForSection(RE, TargetSectionID);
    }
    return ++RelI;
  }

```
- **EN**: Implements logic around `RE`, `addRelocationForSymbol`, `addRelocationForSection`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `RE`, `addRelocationForSymbol`, `addRelocationForSection` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 269-286
```cpp
  void resolveRelocation(const RelocationEntry &RE, uint64_t Value) override {
    using namespace llvm::support::endian;

    const auto Section = Sections[RE.SectionID];
    uint8_t *Target = Section.getAddressWithOffset(RE.Offset);
    uint64_t FinalAddress = Section.getLoadAddressWithOffset(RE.Offset);

    switch (RE.RelType) {
    default:
      llvm_unreachable("unsupported relocation type");
    case COFF::IMAGE_REL_ARM64_ABSOLUTE: {
      // This relocation is ignored.
      break;
    }
    case COFF::IMAGE_REL_ARM64_PAGEBASE_REL21: {
      // The page base of the target, for ADRP instruction.
      Value += RE.Addend;
      write32AArch64Addr(Target, Value, FinalAddress, 12);
```
- **EN**: Introduces declarations for `llvm::support::endian`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm::support::endian` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 287-304
```cpp
      break;
    }
    case COFF::IMAGE_REL_ARM64_REL21: {
      // The 12-bit relative displacement to the target, for instruction ADR
      Value += RE.Addend;
      write32AArch64Addr(Target, Value, FinalAddress, 0);
      break;
    }
    case COFF::IMAGE_REL_ARM64_PAGEOFFSET_12A: {
      // The 12-bit page offset of the target,
      // for instructions ADD/ADDS (immediate) with zero shift.
      Value += RE.Addend;
      write32AArch64Imm(Target, Value & 0xFFF, 0);
      break;
    }
    case COFF::IMAGE_REL_ARM64_PAGEOFFSET_12L: {
      // The 12-bit page offset of the target,
      // for instruction LDR (indexed, unsigned immediate).
```
- **EN**: Implements logic around `write32AArch64Addr`, `write32AArch64Imm`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `write32AArch64Addr`, `write32AArch64Imm` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 305-322
```cpp
      Value += RE.Addend;
      write32AArch64Ldr(Target, Value & 0xFFF);
      break;
    }
    case COFF::IMAGE_REL_ARM64_ADDR32: {
      // The 32-bit VA of the target.
      uint32_t VA = Value + RE.Addend;
      write32le(Target, VA);
      break;
    }
    case COFF::IMAGE_REL_ARM64_ADDR32NB: {
      // The target's 32-bit RVA.
      uint64_t RVA = Value + RE.Addend - getImageBase();
      write32le(Target, RVA);
      break;
    }
    case INTERNAL_REL_ARM64_LONG_BRANCH26: {
      // Encode the immadiate value for generated Stub instruction (MOVZ)
```
- **EN**: Implements logic around `write32AArch64Ldr`, `write32le`, `getImageBase`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `write32AArch64Ldr`, `write32le`, `getImageBase` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 323-340
```cpp
      or32le(Target + 12, ((Value + RE.Addend) & 0xFFFF) << 5);
      or32le(Target + 8, ((Value + RE.Addend) & 0xFFFF0000) >> 11);
      or32le(Target + 4, ((Value + RE.Addend) & 0xFFFF00000000) >> 27);
      or32le(Target + 0, ((Value + RE.Addend) & 0xFFFF000000000000) >> 43);
      break;
    }
    case COFF::IMAGE_REL_ARM64_BRANCH26: {
      // The 26-bit relative displacement to the target, for B and BL
      // instructions.
      uint64_t PCRelVal = Value + RE.Addend - FinalAddress;
      assert(isInt<28>(PCRelVal) && "Branch target is out of range.");
      write32le(Target, (read32le(Target) & ~(0x03FFFFFF)) |
                            (PCRelVal & 0x0FFFFFFC) >> 2);
      break;
    }
    case COFF::IMAGE_REL_ARM64_BRANCH19: {
      // The 19-bit offset to the relocation target,
      // for conditional B instruction.
```
- **EN**: Implements logic around `or32le`, `assert`, `write32le`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `or32le`, `assert`, `write32le` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为。

### Lines 341-358
```cpp
      uint64_t PCRelVal = Value + RE.Addend - FinalAddress;
      assert(isInt<21>(PCRelVal) && "Branch target is out of range.");
      write32le(Target, (read32le(Target) & ~(0x00FFFFE0)) |
                            (PCRelVal & 0x001FFFFC) << 3);
      break;
    }
    case COFF::IMAGE_REL_ARM64_BRANCH14: {
      // The 14-bit offset to the relocation target,
      // for instructions TBZ and TBNZ.
      uint64_t PCRelVal = Value + RE.Addend - FinalAddress;
      assert(isInt<16>(PCRelVal) && "Branch target is out of range.");
      write32le(Target, (read32le(Target) & ~(0x000FFFE0)) |
                            (PCRelVal & 0x0000FFFC) << 3);
      break;
    }
    case COFF::IMAGE_REL_ARM64_ADDR64: {
      // The 64-bit VA of the relocation target.
      write64le(Target, Value + RE.Addend);
```
- **EN**: Implements logic around `assert`, `write32le`, `write64le`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `assert`, `write32le`, `write64le` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为。

### Lines 359-376
```cpp
      break;
    }
    case COFF::IMAGE_REL_ARM64_SECTION: {
      // 16-bit section index of the section that contains the target.
      assert(static_cast<uint32_t>(RE.SectionID) <= UINT16_MAX &&
             "relocation overflow");
      add16(Target, RE.SectionID);
      break;
    }
    case COFF::IMAGE_REL_ARM64_SECREL: {
      // 32-bit offset of the target from the beginning of its section.
      assert(static_cast<int64_t>(RE.Addend) <= INT32_MAX &&
             "Relocation overflow");
      assert(static_cast<int64_t>(RE.Addend) >= INT32_MIN &&
             "Relocation underflow");
      write32le(Target, RE.Addend);
      break;
    }
```
- **EN**: Implements logic around `assert`, `add16`, `write32le`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `assert`, `add16`, `write32le` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为。

### Lines 377-385
```cpp
    case COFF::IMAGE_REL_ARM64_REL32: {
      // The 32-bit relative address from the byte following the relocation.
      uint64_t Result = Value - FinalAddress - 4;
      write32le(Target, Result + RE.Addend);
      break;
    }
    }
  }

```
- **EN**: Implements logic around `write32le`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `write32le` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为。

### Lines 386-393
```cpp
  void registerEHFrames() override {}
};

} // End namespace llvm

#undef DEBUG_TYPE

#endif // LLVM_LIB_EXECUTIONENGINE_RUNTIMEDYLD_TARGETS_RUNTIMEDYLDCOFFAARCH64_H
```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

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

- **Direct includes / 直接包含**: `../RuntimeDyldCOFF.h`, `llvm/ADT/SmallString.h`, `llvm/BinaryFormat/COFF.h`, `llvm/Object/COFF.h`, `llvm/Support/Endian.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Object, Support, BinaryFormat
