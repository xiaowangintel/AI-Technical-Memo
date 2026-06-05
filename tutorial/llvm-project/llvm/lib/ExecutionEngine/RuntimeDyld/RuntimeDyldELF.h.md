# RuntimeDyldELF.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/RuntimeDyld/RuntimeDyldELF.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Run-time dynamic linker for MC-JIT.
  - **CN**: 实现 RuntimeDyld，在运行时把目标代码装入内存、应用重定位并解析符号。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- RuntimeDyldELF.h - Run-time dynamic linker for MC-JIT ---*- C++ -*-===//
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
//
// ELF support for MC-JIT runtime dynamic linker.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_EXECUTIONENGINE_RUNTIMEDYLD_RUNTIMEDYLDELF_H
#define LLVM_LIB_EXECUTIONENGINE_RUNTIMEDYLD_RUNTIMEDYLDELF_H

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 16-23
```cpp
#include "RuntimeDyldImpl.h"
#include "llvm/ADT/DenseMap.h"

namespace llvm {
namespace object {
class ELFObjectFileBase;
}

```
- **EN**: Pulls in the headers needed for this implementation, including `RuntimeDyldImpl.h`, `llvm/ADT/DenseMap.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `RuntimeDyldImpl.h`, `llvm/ADT/DenseMap.h`。

### Lines 24-33
```cpp
class RuntimeDyldELF : public RuntimeDyldImpl {

  void resolveRelocation(const SectionEntry &Section, uint64_t Offset,
                         uint64_t Value, uint32_t Type, int64_t Addend,
                         uint64_t SymOffset = 0, SID SectionID = 0);

  void resolveX86_64Relocation(const SectionEntry &Section, uint64_t Offset,
                               uint64_t Value, uint32_t Type, int64_t Addend,
                               uint64_t SymOffset);

```
- **EN**: Introduces declarations for `RuntimeDyldELF`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `RuntimeDyldELF` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 34-42
```cpp
  void resolveX86Relocation(const SectionEntry &Section, uint64_t Offset,
                            uint32_t Value, uint32_t Type, int32_t Addend);

  void resolveAArch64Relocation(const SectionEntry &Section, uint64_t Offset,
                                uint64_t Value, uint32_t Type, int64_t Addend);

  bool resolveAArch64ShortBranch(unsigned SectionID, relocation_iterator RelI,
                                 const RelocationValueRef &Value);

```
- **EN**: Implements logic around `resolveX86Relocation`, `resolveAArch64Relocation`, `resolveAArch64ShortBranch`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `resolveX86Relocation`, `resolveAArch64Relocation`, `resolveAArch64ShortBranch` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 43-52
```cpp
  void resolveAArch64Branch(unsigned SectionID, const RelocationValueRef &Value,
                            relocation_iterator RelI, StubMap &Stubs);

  void resolveARMRelocation(const SectionEntry &Section, uint64_t Offset,
                            uint32_t Value, uint32_t Type, int32_t Addend);

  void resolveLoongArch64Relocation(const SectionEntry &Section,
                                    uint64_t Offset, uint64_t Value,
                                    uint32_t Type, int64_t Addend);

```
- **EN**: Implements logic around `resolveAArch64Branch`, `resolveARMRelocation`, `resolveLoongArch64Relocation`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `resolveAArch64Branch`, `resolveARMRelocation`, `resolveLoongArch64Relocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 53-60
```cpp
  bool resolveLoongArch64ShortBranch(unsigned SectionID,
                                     relocation_iterator RelI,
                                     const RelocationValueRef &Value);

  void resolveLoongArch64Branch(unsigned SectionID,
                                const RelocationValueRef &Value,
                                relocation_iterator RelI, StubMap &Stubs);

```
- **EN**: Implements logic around `resolveLoongArch64ShortBranch`, `resolveLoongArch64Branch`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `resolveLoongArch64ShortBranch`, `resolveLoongArch64Branch` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 61-69
```cpp
  void resolvePPC32Relocation(const SectionEntry &Section, uint64_t Offset,
                              uint64_t Value, uint32_t Type, int64_t Addend);

  void resolvePPC64Relocation(const SectionEntry &Section, uint64_t Offset,
                              uint64_t Value, uint32_t Type, int64_t Addend);

  void resolveSystemZRelocation(const SectionEntry &Section, uint64_t Offset,
                                uint64_t Value, uint32_t Type, int64_t Addend);

```
- **EN**: Implements logic around `resolvePPC32Relocation`, `resolvePPC64Relocation`, `resolveSystemZRelocation`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `resolvePPC32Relocation`, `resolvePPC64Relocation`, `resolveSystemZRelocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 70-76
```cpp
  void resolveBPFRelocation(const SectionEntry &Section, uint64_t Offset,
                            uint64_t Value, uint32_t Type, int64_t Addend);

  void resolveRISCVRelocation(const SectionEntry &Section, uint64_t Offset,
                              uint64_t Value, uint32_t Type, int64_t Addend,
                              SID SectionID);

```
- **EN**: Implements logic around `resolveBPFRelocation`, `resolveRISCVRelocation`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `resolveBPFRelocation`, `resolveRISCVRelocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 77-90
```cpp
  unsigned getMaxStubSize() const override {
    if (Arch == Triple::aarch64 || Arch == Triple::aarch64_be)
      return 20; // movz; movk; movk; movk; br
    if (Arch == Triple::arm || Arch == Triple::thumb)
      return 8; // 32-bit instruction and 32-bit address
    else if (IsMipsO32ABI || IsMipsN32ABI)
      return 16;
    else if (IsMipsN64ABI)
      return 32;
    if (Arch == Triple::loongarch64)
      return 20; // lu12i.w; ori; lu32i.d; lu52i.d; jr
    else if (Arch == Triple::ppc64 || Arch == Triple::ppc64le)
      return 44;
    else if (Arch == Triple::x86_64)
```
- **EN**: Implements logic around `getMaxStubSize`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getMaxStubSize` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 91-97
```cpp
      return 6; // 2-byte jmp instruction + 32-bit relative address
    else if (Arch == Triple::systemz)
      return 16;
    else
      return 0;
  }

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 98-104
```cpp
  Align getStubAlignment() override {
    if (Arch == Triple::systemz)
      return Align(8);
    else
      return Align(1);
  }

```
- **EN**: Implements logic around `getStubAlignment`, `Align`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getStubAlignment`, `Align` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 105-113
```cpp
  void setMipsABI(const ObjectFile &Obj) override;

  Error findPPC64TOCSection(const object::ELFObjectFileBase &Obj,
                            ObjSectionToIDMap &LocalSections,
                            RelocationValueRef &Rel);
  Error findOPDEntrySection(const object::ELFObjectFileBase &Obj,
                            ObjSectionToIDMap &LocalSections,
                            RelocationValueRef &Rel);

```
- **EN**: Implements logic around `setMipsABI`, `findPPC64TOCSection`, `findOPDEntrySection`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `setMipsABI`, `findPPC64TOCSection`, `findOPDEntrySection` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 114-122
```cpp
protected:
  size_t getGOTEntrySize() override;

private:
  SectionEntry &getSection(unsigned SectionID) { return Sections[SectionID]; }

  // Allocate no GOT entries for use in the given section.
  uint64_t allocateGOTEntries(unsigned no);

```
- **EN**: Implements logic around `getGOTEntrySize`, `getSection`, `allocateGOTEntries`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getGOTEntrySize`, `getSection`, `allocateGOTEntries` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 123-131
```cpp
  // Find GOT entry corresponding to relocation or create new one.
  uint64_t findOrAllocGOTEntry(const RelocationValueRef &Value,
                               unsigned GOTRelType);

  // Resolve the relative address of GOTOffset in Section ID and place
  // it at the given Offset
  void resolveGOTOffsetRelocation(unsigned SectionID, uint64_t Offset,
                                  uint64_t GOTOffset, uint32_t Type);

```
- **EN**: Implements logic around `findOrAllocGOTEntry`, `resolveGOTOffsetRelocation`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `findOrAllocGOTEntry`, `resolveGOTOffsetRelocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 132-139
```cpp
  // For a GOT entry referenced from SectionID, compute a relocation entry
  // that will place the final resolved value in the GOT slot
  RelocationEntry computeGOTOffsetRE(uint64_t GOTOffset, uint64_t SymbolOffset,
                                     unsigned Type);

  // Compute the address in memory where we can find the placeholder
  void *computePlaceholderAddress(unsigned SectionID, uint64_t Offset) const;

```
- **EN**: Implements logic around `computeGOTOffsetRE`, `computePlaceholderAddress`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `computeGOTOffsetRE`, `computePlaceholderAddress` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 140-147
```cpp
  // Split out common case for creating the RelocationEntry for when the
  // relocation requires no particular advanced processing.
  void processSimpleRelocation(unsigned SectionID, uint64_t Offset, unsigned RelType, RelocationValueRef Value);

  // Return matching *LO16 relocation (Mips specific)
  uint32_t getMatchingLoRelocation(uint32_t RelType,
                                   bool IsLocal = false) const;

```
- **EN**: Implements logic around `processSimpleRelocation`, `getMatchingLoRelocation`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `processSimpleRelocation`, `getMatchingLoRelocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 148-155
```cpp
  // The tentative ID for the GOT section
  unsigned GOTSectionID;

  // Records the current number of allocated slots in the GOT
  // (This would be equivalent to GOTEntries.size() were it not for relocations
  // that consume more than one slot)
  unsigned CurrentGOTIndex;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 156-164
```cpp
protected:
  // A map from section to a GOT section that has entries for section's GOT
  // relocations. (Mips64 specific)
  DenseMap<SID, SID> SectionToGOTMap;

private:
  // A map to avoid duplicate got entries (Mips64 specific)
  StringMap<uint64_t> GOTSymbolOffsets;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 165-171
```cpp
  // *HI16 relocations will be added for resolving when we find matching
  // *LO16 part. (Mips specific)
  //
  // *HI20 relocations will be added for resolving when we find matching
  // *LO12 part. (RISC-V specific)
  SmallVector<std::pair<RelocationValueRef, RelocationEntry>, 8> PendingRelocs;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 172-179
```cpp
  // When a module is loaded we save the SectionID of the EH frame section
  // in a table until we receive a request to register all unregistered
  // EH frame sections with the memory manager.
  SmallVector<SID, 2> UnregisteredEHFrameSections;

  // Map between GOT relocation value and corresponding GOT offset
  std::map<RelocationValueRef, uint64_t> GOTOffsetMap;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 180-187
```cpp
  /// The ID of the current IFunc stub section
  unsigned IFuncStubSectionID = 0;
  /// The current offset into the IFunc stub section
  uint64_t IFuncStubOffset = 0;

  /// A IFunc stub and its original symbol
  struct IFuncStub {
    /// The offset of this stub in the IFunc stub section
```
- **EN**: Introduces declarations for `IFuncStub`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `IFuncStub` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 188-195
```cpp
    uint64_t StubOffset;
    /// The symbol table entry of the original symbol
    SymbolTableEntry OriginalSymbol;
  };

  /// The IFunc stubs
  SmallVector<IFuncStub, 2> IFuncStubs;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 196-202
```cpp
  /// Create the code for the IFunc resolver at the given address. This code
  /// works together with the stubs created in createIFuncStub() to call the
  /// resolver function and then jump to the real function address.
  /// It must not be larger than 64B.
  void createIFuncResolver(uint8_t *Addr) const;
  /// Create the code for an IFunc stub for the IFunc that is defined in
  /// section IFuncSectionID at offset IFuncOffset. The IFunc resolver created
```
- **EN**: Implements logic around `createIFuncResolver`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `createIFuncResolver` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 203-209
```cpp
  /// by createIFuncResolver() is defined in the section IFuncStubSectionID at
  /// offset IFuncResolverOffset. The code should be written into the section
  /// with the id IFuncStubSectionID at the offset IFuncStubOffset.
  void createIFuncStub(unsigned IFuncStubSectionID,
                       uint64_t IFuncResolverOffset, uint64_t IFuncStubOffset,
                       unsigned IFuncSectionID, uint64_t IFuncOffset);
  /// Return the maximum size of a stub created by createIFuncStub()
```
- **EN**: Implements logic around `createIFuncStub`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `createIFuncStub` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 210-216
```cpp
  unsigned getMaxIFuncStubSize() const;

  void processNewSymbol(const SymbolRef &ObjSymbol,
                        SymbolTableEntry &Entry) override;
  bool relocationNeedsGot(const RelocationRef &R) const override;
  bool relocationNeedsStub(const RelocationRef &R) const override;

```
- **EN**: Implements logic around `getMaxIFuncStubSize`, `processNewSymbol`, `relocationNeedsGot`, `relocationNeedsStub`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `getMaxIFuncStubSize`, `processNewSymbol`, `relocationNeedsGot`, `relocationNeedsStub` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 217-228
```cpp
  // Process a GOTTPOFF TLS relocation for x86-64
  // NOLINTNEXTLINE(readability-identifier-naming)
  void processX86_64GOTTPOFFRelocation(unsigned SectionID, uint64_t Offset,
                                       RelocationValueRef Value,
                                       int64_t Addend);
  // Process a TLSLD/TLSGD relocation for x86-64
  // NOLINTNEXTLINE(readability-identifier-naming)
  void processX86_64TLSRelocation(unsigned SectionID, uint64_t Offset,
                                  uint64_t RelType, RelocationValueRef Value,
                                  int64_t Addend,
                                  const RelocationRef &GetAddrRelocation);

```
- **EN**: Implements logic around `processX86_64GOTTPOFFRelocation`, `processX86_64TLSRelocation`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `processX86_64GOTTPOFFRelocation`, `processX86_64TLSRelocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 229-237
```cpp
public:
  RuntimeDyldELF(RuntimeDyld::MemoryManager &MemMgr,
                 JITSymbolResolver &Resolver);
  ~RuntimeDyldELF() override;

  static std::unique_ptr<RuntimeDyldELF>
  create(Triple::ArchType Arch, RuntimeDyld::MemoryManager &MemMgr,
         JITSymbolResolver &Resolver);

```
- **EN**: Implements logic around `RuntimeDyldELF`, `~RuntimeDyldELF`, `create`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `RuntimeDyldELF`, `~RuntimeDyldELF`, `create` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 238-251
```cpp
  std::unique_ptr<RuntimeDyld::LoadedObjectInfo>
  loadObject(const object::ObjectFile &O) override;

  void resolveRelocation(const RelocationEntry &RE, uint64_t Value) override;
  Expected<relocation_iterator>
  processRelocationRef(unsigned SectionID, relocation_iterator RelI,
                       const ObjectFile &Obj,
                       ObjSectionToIDMap &ObjSectionToID,
                       StubMap &Stubs) override;
  bool isCompatibleFile(const object::ObjectFile &Obj) const override;
  void registerEHFrames() override;
  Error finalizeLoad(const ObjectFile &Obj,
                     ObjSectionToIDMap &SectionMap) override;
};
```
- **EN**: Implements logic around `loadObject`, `resolveRelocation`, `processRelocationRef`, `isCompatibleFile`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `loadObject`, `resolveRelocation`, `processRelocationRef`, `isCompatibleFile`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 252-255
```cpp

} // end namespace llvm

#endif // LLVM_LIB_EXECUTIONENGINE_RUNTIMEDYLD_RUNTIMEDYLDELF_H
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

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `RuntimeDyldImpl.h`, `llvm/ADT/DenseMap.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine
