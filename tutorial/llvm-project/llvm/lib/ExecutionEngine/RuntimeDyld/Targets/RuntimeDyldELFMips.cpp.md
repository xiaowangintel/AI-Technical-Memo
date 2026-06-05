# RuntimeDyldELFMips.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/RuntimeDyld/Targets/RuntimeDyldELFMips.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements RuntimeDyld, which loads object code into memory, applies relocations, and resolves symbols at runtime.
  - **CN**: 实现 RuntimeDyld，在运行时把目标代码装入内存、应用重定位并解析符号。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- RuntimeDyldELFMips.cpp ---- ELF/Mips specific code. -----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "RuntimeDyldELFMips.h"
#include "llvm/BinaryFormat/ELF.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `RuntimeDyldELFMips.h`, `llvm/BinaryFormat/ELF.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `RuntimeDyldELFMips.h`, `llvm/BinaryFormat/ELF.h`。

### Lines 12-28
```cpp
#define DEBUG_TYPE "dyld"

void RuntimeDyldELFMips::resolveRelocation(const RelocationEntry &RE,
                                           uint64_t Value) {
  const SectionEntry &Section = Sections[RE.SectionID];
  if (IsMipsO32ABI)
    resolveMIPSO32Relocation(Section, RE.Offset, Value, RE.RelType, RE.Addend);
  else if (IsMipsN32ABI) {
    resolveMIPSN32Relocation(Section, RE.Offset, Value, RE.RelType, RE.Addend,
                             RE.SymOffset, RE.SectionID);
  } else if (IsMipsN64ABI)
    resolveMIPSN64Relocation(Section, RE.Offset, Value, RE.RelType, RE.Addend,
                             RE.SymOffset, RE.SectionID);
  else
    llvm_unreachable("Mips ABI not handled");
}

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 29-40
```cpp
uint64_t RuntimeDyldELFMips::evaluateRelocation(const RelocationEntry &RE,
                                                uint64_t Value,
                                                uint64_t Addend) {
  if (IsMipsN32ABI) {
    const SectionEntry &Section = Sections[RE.SectionID];
    Value = evaluateMIPS64Relocation(Section, RE.Offset, Value, RE.RelType,
                                     Addend, RE.SymOffset, RE.SectionID);
    return Value;
  }
  llvm_unreachable("Not reachable");
}

```
- **EN**: Implements logic around `evaluateRelocation`, `evaluateMIPS64Relocation`, `llvm_unreachable`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `evaluateRelocation`, `evaluateMIPS64Relocation`, `llvm_unreachable` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 41-51
```cpp
void RuntimeDyldELFMips::applyRelocation(const RelocationEntry &RE,
                                         uint64_t Value) {
  if (IsMipsN32ABI) {
    const SectionEntry &Section = Sections[RE.SectionID];
    applyMIPSRelocation(Section.getAddressWithOffset(RE.Offset), Value,
                        RE.RelType);
    return;
  }
  llvm_unreachable("Not reachable");
}

```
- **EN**: Implements logic around `applyRelocation`, `applyMIPSRelocation`, `llvm_unreachable`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `applyRelocation`, `applyMIPSRelocation`, `llvm_unreachable` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 52-63
```cpp
int64_t
RuntimeDyldELFMips::evaluateMIPS32Relocation(const SectionEntry &Section,
                                             uint64_t Offset, uint64_t Value,
                                             uint32_t Type) {

  LLVM_DEBUG(dbgs() << "evaluateMIPS32Relocation, LocalAddress: 0x"
                    << format("%llx", Section.getAddressWithOffset(Offset))
                    << " FinalAddress: 0x"
                    << format("%llx", Section.getLoadAddressWithOffset(Offset))
                    << " Value: 0x" << format("%llx", Value) << " Type: 0x"
                    << format("%x", Type) << "\n");

```
- **EN**: Implements logic around `evaluateMIPS32Relocation`, `format`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `evaluateMIPS32Relocation`, `format` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 64-81
```cpp
  switch (Type) {
  default:
    llvm_unreachable("Unknown relocation type!");
    return Value;
  case ELF::R_MIPS_32:
    return Value;
  case ELF::R_MIPS_26:
    return Value >> 2;
  case ELF::R_MIPS_HI16:
    // Get the higher 16-bits. Also add 1 if bit 15 is 1.
    return (Value + 0x8000) >> 16;
  case ELF::R_MIPS_LO16:
    return Value;
  case ELF::R_MIPS_PC32: {
    uint32_t FinalAddress = Section.getLoadAddressWithOffset(Offset);
    return Value - FinalAddress;
  }
  case ELF::R_MIPS_PC16: {
```
- **EN**: Implements logic around `llvm_unreachable`, `getLoadAddressWithOffset`; this block handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `llvm_unreachable`, `getLoadAddressWithOffset` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 82-99
```cpp
    uint32_t FinalAddress = Section.getLoadAddressWithOffset(Offset);
    return (Value - FinalAddress) >> 2;
  }
  case ELF::R_MIPS_PC19_S2: {
    uint32_t FinalAddress = Section.getLoadAddressWithOffset(Offset);
    return (Value - (FinalAddress & ~0x3)) >> 2;
  }
  case ELF::R_MIPS_PC21_S2: {
    uint32_t FinalAddress = Section.getLoadAddressWithOffset(Offset);
    return (Value - FinalAddress) >> 2;
  }
  case ELF::R_MIPS_PC26_S2: {
    uint32_t FinalAddress = Section.getLoadAddressWithOffset(Offset);
    return (Value - FinalAddress) >> 2;
  }
  case ELF::R_MIPS_PCHI16: {
    uint32_t FinalAddress = Section.getLoadAddressWithOffset(Offset);
    return (Value - FinalAddress + 0x8000) >> 16;
```
- **EN**: Implements logic around `getLoadAddressWithOffset`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getLoadAddressWithOffset` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 100-111
```cpp
  }
  case ELF::R_MIPS_PCLO16: {
    uint32_t FinalAddress = Section.getLoadAddressWithOffset(Offset);
    return Value - FinalAddress;
  }
  }
}

int64_t RuntimeDyldELFMips::evaluateMIPS64Relocation(
    const SectionEntry &Section, uint64_t Offset, uint64_t Value, uint32_t Type,
    int64_t Addend, uint64_t SymOffset, SID SectionID) {

```
- **EN**: Implements logic around `getLoadAddressWithOffset`, `evaluateMIPS64Relocation`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getLoadAddressWithOffset`, `evaluateMIPS64Relocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 112-122
```cpp
  LLVM_DEBUG(dbgs() << "evaluateMIPS64Relocation, LocalAddress: 0x"
                    << format("%llx", Section.getAddressWithOffset(Offset))
                    << " FinalAddress: 0x"
                    << format("%llx", Section.getLoadAddressWithOffset(Offset))
                    << " Value: 0x" << format("%llx", Value) << " Type: 0x"
                    << format("%x", Type) << " Addend: 0x"
                    << format("%llx", Addend)
                    << " Offset: " << format("%llx" PRIx64, Offset)
                    << " SID: " << format("%d", SectionID)
                    << " SymOffset: " << format("%x", SymOffset) << "\n");

```
- **EN**: Implements logic around `format`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `format` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 123-140
```cpp
  switch (Type) {
  default:
    llvm_unreachable("Not implemented relocation type!");
    break;
  case ELF::R_MIPS_JALR:
  case ELF::R_MIPS_NONE:
    break;
  case ELF::R_MIPS_32:
  case ELF::R_MIPS_64:
    return Value + Addend;
  case ELF::R_MIPS_26:
    return ((Value + Addend) >> 2) & 0x3ffffff;
  case ELF::R_MIPS_GPREL16: {
    uint64_t GOTAddr = getSectionLoadAddress(SectionToGOTMap[SectionID]);
    return Value + Addend - (GOTAddr + 0x7ff0);
  }
  case ELF::R_MIPS_SUB:
    return Value - Addend;
```
- **EN**: Implements logic around `llvm_unreachable`, `getSectionLoadAddress`; this block handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `llvm_unreachable`, `getSectionLoadAddress` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 141-156
```cpp
  case ELF::R_MIPS_HI16:
    // Get the higher 16-bits. Also add 1 if bit 15 is 1.
    return ((Value + Addend + 0x8000) >> 16) & 0xffff;
  case ELF::R_MIPS_LO16:
    return (Value + Addend) & 0xffff;
  case ELF::R_MIPS_HIGHER:
    return ((Value + Addend + 0x80008000) >> 32) & 0xffff;
  case ELF::R_MIPS_HIGHEST:
    return ((Value + Addend + 0x800080008000) >> 48) & 0xffff;
  case ELF::R_MIPS_CALL16:
  case ELF::R_MIPS_GOT_DISP:
  case ELF::R_MIPS_GOT_PAGE: {
    uint8_t *LocalGOTAddr =
        getSectionAddress(SectionToGOTMap[SectionID]) + SymOffset;
    uint64_t GOTEntry = readBytesUnaligned(LocalGOTAddr, getGOTEntrySize());

```
- **EN**: Implements logic around `getSectionAddress`, `readBytesUnaligned`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getSectionAddress`, `readBytesUnaligned` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 157-166
```cpp
    Value += Addend;
    if (Type == ELF::R_MIPS_GOT_PAGE)
      Value = (Value + 0x8000) & ~0xffff;

    if (GOTEntry)
      assert(GOTEntry == Value &&
                   "GOT entry has two different addresses.");
    else
      writeBytesUnaligned(Value, LocalGOTAddr, getGOTEntrySize());

```
- **EN**: Implements logic around `assert`, `writeBytesUnaligned`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `assert`, `writeBytesUnaligned` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 167-184
```cpp
    return (SymOffset - 0x7ff0) & 0xffff;
  }
  case ELF::R_MIPS_GOT_OFST: {
    int64_t page = (Value + Addend + 0x8000) & ~0xffff;
    return (Value + Addend - page) & 0xffff;
  }
  case ELF::R_MIPS_GPREL32: {
    uint64_t GOTAddr = getSectionLoadAddress(SectionToGOTMap[SectionID]);
    return Value + Addend - (GOTAddr + 0x7ff0);
  }
  case ELF::R_MIPS_PC16: {
    uint64_t FinalAddress = Section.getLoadAddressWithOffset(Offset);
    return ((Value + Addend - FinalAddress) >> 2) & 0xffff;
  }
  case ELF::R_MIPS_PC32: {
    uint64_t FinalAddress = Section.getLoadAddressWithOffset(Offset);
    return Value + Addend - FinalAddress;
  }
```
- **EN**: Implements logic around `getSectionLoadAddress`, `getLoadAddressWithOffset`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getSectionLoadAddress`, `getLoadAddressWithOffset` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 185-202
```cpp
  case ELF::R_MIPS_PC18_S3: {
    uint64_t FinalAddress = Section.getLoadAddressWithOffset(Offset);
    return ((Value + Addend - (FinalAddress & ~0x7)) >> 3) & 0x3ffff;
  }
  case ELF::R_MIPS_PC19_S2: {
    uint64_t FinalAddress = Section.getLoadAddressWithOffset(Offset);
    return ((Value + Addend - (FinalAddress & ~0x3)) >> 2) & 0x7ffff;
  }
  case ELF::R_MIPS_PC21_S2: {
    uint64_t FinalAddress = Section.getLoadAddressWithOffset(Offset);
    return ((Value + Addend - FinalAddress) >> 2) & 0x1fffff;
  }
  case ELF::R_MIPS_PC26_S2: {
    uint64_t FinalAddress = Section.getLoadAddressWithOffset(Offset);
    return ((Value + Addend - FinalAddress) >> 2) & 0x3ffffff;
  }
  case ELF::R_MIPS_PCHI16: {
    uint64_t FinalAddress = Section.getLoadAddressWithOffset(Offset);
```
- **EN**: Implements logic around `getLoadAddressWithOffset`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getLoadAddressWithOffset` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 203-212
```cpp
    return ((Value + Addend - FinalAddress + 0x8000) >> 16) & 0xffff;
  }
  case ELF::R_MIPS_PCLO16: {
    uint64_t FinalAddress = Section.getLoadAddressWithOffset(Offset);
    return (Value + Addend - FinalAddress) & 0xffff;
  }
  }
  return 0;
}

```
- **EN**: Implements logic around `getLoadAddressWithOffset`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getLoadAddressWithOffset` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 213-230
```cpp
void RuntimeDyldELFMips::applyMIPSRelocation(uint8_t *TargetPtr, int64_t Value,
                                             uint32_t Type) {
  uint32_t Insn = readBytesUnaligned(TargetPtr, 4);

  switch (Type) {
  default:
    llvm_unreachable("Unknown relocation type!");
    break;
  case ELF::R_MIPS_GPREL16:
  case ELF::R_MIPS_HI16:
  case ELF::R_MIPS_LO16:
  case ELF::R_MIPS_HIGHER:
  case ELF::R_MIPS_HIGHEST:
  case ELF::R_MIPS_PC16:
  case ELF::R_MIPS_PCHI16:
  case ELF::R_MIPS_PCLO16:
  case ELF::R_MIPS_CALL16:
  case ELF::R_MIPS_GOT_DISP:
```
- **EN**: Implements logic around `applyMIPSRelocation`, `readBytesUnaligned`, `llvm_unreachable`; this block handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `applyMIPSRelocation`, `readBytesUnaligned`, `llvm_unreachable` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 231-248
```cpp
  case ELF::R_MIPS_GOT_PAGE:
  case ELF::R_MIPS_GOT_OFST:
    Insn = (Insn & 0xffff0000) | (Value & 0x0000ffff);
    writeBytesUnaligned(Insn, TargetPtr, 4);
    break;
  case ELF::R_MIPS_PC18_S3:
    Insn = (Insn & 0xfffc0000) | (Value & 0x0003ffff);
    writeBytesUnaligned(Insn, TargetPtr, 4);
    break;
  case ELF::R_MIPS_PC19_S2:
    Insn = (Insn & 0xfff80000) | (Value & 0x0007ffff);
    writeBytesUnaligned(Insn, TargetPtr, 4);
    break;
  case ELF::R_MIPS_PC21_S2:
    Insn = (Insn & 0xffe00000) | (Value & 0x001fffff);
    writeBytesUnaligned(Insn, TargetPtr, 4);
    break;
  case ELF::R_MIPS_26:
```
- **EN**: Implements logic around `writeBytesUnaligned`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `writeBytesUnaligned` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 249-264
```cpp
  case ELF::R_MIPS_PC26_S2:
    Insn = (Insn & 0xfc000000) | (Value & 0x03ffffff);
    writeBytesUnaligned(Insn, TargetPtr, 4);
    break;
  case ELF::R_MIPS_32:
  case ELF::R_MIPS_GPREL32:
  case ELF::R_MIPS_PC32:
    writeBytesUnaligned(Value & 0xffffffff, TargetPtr, 4);
    break;
  case ELF::R_MIPS_64:
  case ELF::R_MIPS_SUB:
    writeBytesUnaligned(Value, TargetPtr, 8);
    break;
  }
}

```
- **EN**: Implements logic around `writeBytesUnaligned`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `writeBytesUnaligned` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 265-273
```cpp
void RuntimeDyldELFMips::resolveMIPSN32Relocation(
    const SectionEntry &Section, uint64_t Offset, uint64_t Value, uint32_t Type,
    int64_t Addend, uint64_t SymOffset, SID SectionID) {
  int64_t CalculatedValue = evaluateMIPS64Relocation(
      Section, Offset, Value, Type, Addend, SymOffset, SectionID);
  applyMIPSRelocation(Section.getAddressWithOffset(Offset), CalculatedValue,
                      Type);
}

```
- **EN**: Implements logic around `resolveMIPSN32Relocation`, `evaluateMIPS64Relocation`, `applyMIPSRelocation`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `resolveMIPSN32Relocation`, `evaluateMIPS64Relocation`, `applyMIPSRelocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 274-291
```cpp
void RuntimeDyldELFMips::resolveMIPSN64Relocation(
    const SectionEntry &Section, uint64_t Offset, uint64_t Value, uint32_t Type,
    int64_t Addend, uint64_t SymOffset, SID SectionID) {
  uint32_t r_type = Type & 0xff;
  uint32_t r_type2 = (Type >> 8) & 0xff;
  uint32_t r_type3 = (Type >> 16) & 0xff;

  // RelType is used to keep information for which relocation type we are
  // applying relocation.
  uint32_t RelType = r_type;
  int64_t CalculatedValue = evaluateMIPS64Relocation(Section, Offset, Value,
                                                     RelType, Addend,
                                                     SymOffset, SectionID);
  if (r_type2 != ELF::R_MIPS_NONE) {
    RelType = r_type2;
    CalculatedValue = evaluateMIPS64Relocation(Section, Offset, 0, RelType,
                                               CalculatedValue, SymOffset,
                                               SectionID);
```
- **EN**: Implements logic around `resolveMIPSN64Relocation`, `evaluateMIPS64Relocation`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `resolveMIPSN64Relocation`, `evaluateMIPS64Relocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 292-302
```cpp
  }
  if (r_type3 != ELF::R_MIPS_NONE) {
    RelType = r_type3;
    CalculatedValue = evaluateMIPS64Relocation(Section, Offset, 0, RelType,
                                               CalculatedValue, SymOffset,
                                               SectionID);
  }
  applyMIPSRelocation(Section.getAddressWithOffset(Offset), CalculatedValue,
                      RelType);
}

```
- **EN**: Implements logic around `evaluateMIPS64Relocation`, `applyMIPSRelocation`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `evaluateMIPS64Relocation`, `applyMIPSRelocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 303-316
```cpp
void RuntimeDyldELFMips::resolveMIPSO32Relocation(const SectionEntry &Section,
                                                  uint64_t Offset,
                                                  uint32_t Value, uint32_t Type,
                                                  int32_t Addend) {
  uint8_t *TargetPtr = Section.getAddressWithOffset(Offset);
  Value += Addend;

  LLVM_DEBUG(dbgs() << "resolveMIPSO32Relocation, LocalAddress: "
                    << Section.getAddressWithOffset(Offset) << " FinalAddress: "
                    << format("%p", Section.getLoadAddressWithOffset(Offset))
                    << " Value: " << format("%x", Value) << " Type: "
                    << format("%x", Type) << " Addend: " << format("%x", Addend)
                    << " SymOffset: " << format("%x", Offset) << "\n");

```
- **EN**: Implements logic around `resolveMIPSO32Relocation`, `getAddressWithOffset`, `format`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `resolveMIPSO32Relocation`, `getAddressWithOffset`, `format` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 317-320
```cpp
  Value = evaluateMIPS32Relocation(Section, Offset, Value, Type);

  applyMIPSRelocation(TargetPtr, Value, Type);
}
```
- **EN**: Implements logic around `evaluateMIPS32Relocation`, `applyMIPSRelocation`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `evaluateMIPS32Relocation`, `applyMIPSRelocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

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

- **Direct includes / 直接包含**: `RuntimeDyldELFMips.h`, `llvm/BinaryFormat/ELF.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, BinaryFormat
