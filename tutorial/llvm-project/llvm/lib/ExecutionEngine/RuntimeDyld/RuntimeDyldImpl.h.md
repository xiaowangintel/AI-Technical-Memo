# RuntimeDyldImpl.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/RuntimeDyld/RuntimeDyldImpl.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Run-time dynamic linker for MC-JIT.
  - **CN**: 实现 RuntimeDyld，在运行时把目标代码装入内存、应用重定位并解析符号。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- RuntimeDyldImpl.h - Run-time dynamic linker for MC-JIT --*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Interface for the implementations of runtime dynamic linker facilities.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 12-29
```cpp

#ifndef LLVM_LIB_EXECUTIONENGINE_RUNTIMEDYLD_RUNTIMEDYLDIMPL_H
#define LLVM_LIB_EXECUTIONENGINE_RUNTIMEDYLD_RUNTIMEDYLDIMPL_H

#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringMap.h"
#include "llvm/ExecutionEngine/Orc/SymbolStringPool.h"
#include "llvm/ExecutionEngine/RTDyldMemoryManager.h"
#include "llvm/ExecutionEngine/RuntimeDyld.h"
#include "llvm/ExecutionEngine/RuntimeDyldChecker.h"
#include "llvm/Object/ObjectFile.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/Format.h"
#include "llvm/Support/Mutex.h"
#include "llvm/Support/SwapByteOrder.h"
#include "llvm/TargetParser/Host.h"
#include "llvm/TargetParser/Triple.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ADT/SmallVector.h`, `llvm/ADT/StringMap.h`, `llvm/ExecutionEngine/Orc/SymbolStringPool.h`, `llvm/ExecutionEngine/RTDyldMemoryManager.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ADT/SmallVector.h`, `llvm/ADT/StringMap.h`, `llvm/ExecutionEngine/Orc/SymbolStringPool.h`, `llvm/ExecutionEngine/RTDyldMemoryManager.h`。

### Lines 30-39
```cpp
#include <deque>
#include <map>
#include <system_error>
#include <unordered_map>

using namespace llvm;
using namespace llvm::object;

namespace llvm {

```
- **EN**: Pulls in the headers needed for this implementation, including `deque`, `map`, `system_error`, `unordered_map`.
- **CN**: 引入该实现所需的头文件，其中包括 `deque`, `map`, `system_error`, `unordered_map`。

### Lines 40-49
```cpp
#define UNIMPLEMENTED_RELOC(RelType) \
  case RelType: \
    return make_error<RuntimeDyldError>("Unimplemented relocation: " #RelType)

/// SectionEntry - represents a section emitted into memory by the dynamic
/// linker.
class SectionEntry {
  /// Name - section name.
  std::string Name;

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 50-58
```cpp
  /// Address - address in the linker's memory where the section resides.
  uint8_t *Address;

  /// Size - section size. Doesn't include the stubs.
  size_t Size;

  /// LoadAddress - the address of the section in the target process's memory.
  /// Used for situations in which JIT-ed code is being executed in the address
  /// space of a separate process.  If the code executes in the same address
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 59-67
```cpp
  /// space where it was JIT-ed, this just equals Address.
  uint64_t LoadAddress;

  /// StubOffset - used for architectures with stub functions for far
  /// relocations (like ARM).
  uintptr_t StubOffset;

  /// The total amount of space allocated for this section.  This includes the
  /// section size and the maximum amount of space that the stubs can occupy.
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 68-84
```cpp
  size_t AllocationSize;

  /// ObjAddress - address of the section in the in-memory object file.  Used
  /// for calculating relocations in some object formats (like MachO).
  uintptr_t ObjAddress;

public:
  SectionEntry(StringRef name, uint8_t *address, size_t size,
               size_t allocationSize, uintptr_t objAddress)
      : Name(std::string(name)), Address(address), Size(size),
        LoadAddress(reinterpret_cast<uintptr_t>(address)), StubOffset(size),
        AllocationSize(allocationSize), ObjAddress(objAddress) {
    // AllocationSize is used only in asserts, prevent an "unused private field"
    // warning:
    (void)AllocationSize;
  }

```
- **EN**: Implements logic around `SectionEntry`, `Name`, `LoadAddress`, `AllocationSize`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `SectionEntry`, `Name`, `LoadAddress`, `AllocationSize` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 85-94
```cpp
  StringRef getName() const { return Name; }

  uint8_t *getAddress() const { return Address; }

  /// Return the address of this section with an offset.
  uint8_t *getAddressWithOffset(unsigned OffsetBytes) const {
    assert(OffsetBytes <= AllocationSize && "Offset out of bounds!");
    return Address + OffsetBytes;
  }

```
- **EN**: Implements logic around `getName`, `getAddress`, `getAddressWithOffset`, `assert`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getName`, `getAddress`, `getAddressWithOffset`, `assert` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 95-105
```cpp
  size_t getSize() const { return Size; }

  uint64_t getLoadAddress() const { return LoadAddress; }
  void setLoadAddress(uint64_t LA) { LoadAddress = LA; }

  /// Return the load address of this section with an offset.
  uint64_t getLoadAddressWithOffset(unsigned OffsetBytes) const {
    assert(OffsetBytes <= AllocationSize && "Offset out of bounds!");
    return LoadAddress + OffsetBytes;
  }

```
- **EN**: Implements logic around `getSize`, `getLoadAddress`, `setLoadAddress`, `getLoadAddressWithOffset`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getSize`, `getLoadAddress`, `setLoadAddress`, `getLoadAddressWithOffset`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 106-115
```cpp
  uintptr_t getStubOffset() const { return StubOffset; }

  void advanceStubOffset(unsigned StubSize) {
    StubOffset += StubSize;
    assert(StubOffset <= AllocationSize && "Not enough space allocated!");
  }

  uintptr_t getObjAddress() const { return ObjAddress; }
};

```
- **EN**: Implements logic around `getStubOffset`, `advanceStubOffset`, `assert`, `getObjAddress`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getStubOffset`, `advanceStubOffset`, `assert`, `getObjAddress` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 116-124
```cpp
/// RelocationEntry - used to represent relocations internally in the dynamic
/// linker.
class RelocationEntry {
public:
  /// Offset - offset into the section.
  uint64_t Offset;

  /// Addend - the relocation addend encoded in the instruction itself.  Also
  /// used to make a relocation section relative instead of symbol relative.
```
- **EN**: Introduces declarations for `RelocationEntry`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `RelocationEntry` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 125-137
```cpp
  int64_t Addend;

  /// SectionID - the section this relocation points to.
  unsigned SectionID;

  /// RelType - relocation type.
  uint32_t RelType;

  struct SectionPair {
    uint32_t SectionA;
    uint32_t SectionB;
  };

```
- **EN**: Introduces declarations for `SectionPair`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `SectionPair` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 138-147
```cpp
  /// SymOffset - Section offset of the relocation entry's symbol (used for GOT
  /// lookup).
  union {
    uint64_t SymOffset;
    SectionPair Sections;
  };

  /// The size of this relocation (MachO specific).
  unsigned Size;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 148-157
```cpp
  /// True if this is a PCRel relocation (MachO specific).
  bool IsPCRel : 1;

  // ARM (MachO and COFF) specific.
  bool IsTargetThumbFunc : 1;

  RelocationEntry(unsigned id, uint64_t offset, uint32_t type, int64_t addend)
      : Offset(offset), Addend(addend), SectionID(id), RelType(type),
        SymOffset(0), Size(0), IsPCRel(false), IsTargetThumbFunc(false) {}

```
- **EN**: Implements logic around `RelocationEntry`, `Offset`, `SymOffset`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `RelocationEntry`, `Offset`, `SymOffset` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 158-168
```cpp
  RelocationEntry(unsigned id, uint64_t offset, uint32_t type, int64_t addend,
                  uint64_t symoffset)
      : Offset(offset), Addend(addend), SectionID(id), RelType(type),
        SymOffset(symoffset), Size(0), IsPCRel(false),
        IsTargetThumbFunc(false) {}

  RelocationEntry(unsigned id, uint64_t offset, uint32_t type, int64_t addend,
                  bool IsPCRel, unsigned Size)
      : Offset(offset), Addend(addend), SectionID(id), RelType(type),
        SymOffset(0), Size(Size), IsPCRel(IsPCRel), IsTargetThumbFunc(false) {}

```
- **EN**: Implements logic around `RelocationEntry`, `Offset`, `SymOffset`, `IsTargetThumbFunc`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `RelocationEntry`, `Offset`, `SymOffset`, `IsTargetThumbFunc` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 169-178
```cpp
  RelocationEntry(unsigned id, uint64_t offset, uint32_t type, int64_t addend,
                  unsigned SectionA, uint64_t SectionAOffset, unsigned SectionB,
                  uint64_t SectionBOffset, bool IsPCRel, unsigned Size)
      : Offset(offset), Addend(SectionAOffset - SectionBOffset + addend),
        SectionID(id), RelType(type), Size(Size), IsPCRel(IsPCRel),
        IsTargetThumbFunc(false) {
    Sections.SectionA = SectionA;
    Sections.SectionB = SectionB;
  }

```
- **EN**: Implements logic around `RelocationEntry`, `Offset`, `SectionID`, `IsTargetThumbFunc`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `RelocationEntry`, `Offset`, `SectionID`, `IsTargetThumbFunc` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 179-190
```cpp
  RelocationEntry(unsigned id, uint64_t offset, uint32_t type, int64_t addend,
                  unsigned SectionA, uint64_t SectionAOffset, unsigned SectionB,
                  uint64_t SectionBOffset, bool IsPCRel, unsigned Size,
                  bool IsTargetThumbFunc)
      : Offset(offset), Addend(SectionAOffset - SectionBOffset + addend),
        SectionID(id), RelType(type), Size(Size), IsPCRel(IsPCRel),
        IsTargetThumbFunc(IsTargetThumbFunc) {
    Sections.SectionA = SectionA;
    Sections.SectionB = SectionB;
  }
};

```
- **EN**: Implements logic around `RelocationEntry`, `Offset`, `SectionID`, `IsTargetThumbFunc`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `RelocationEntry`, `Offset`, `SectionID`, `IsTargetThumbFunc` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 191-208
```cpp
class RelocationValueRef {
public:
  unsigned SectionID = 0;
  uint64_t Offset = 0;
  int64_t Addend = 0;
  const char *SymbolName = nullptr;
  bool IsStubThumb = false;

  inline bool operator==(const RelocationValueRef &Other) const {
    return SectionID == Other.SectionID && Offset == Other.Offset &&
           Addend == Other.Addend && SymbolName == Other.SymbolName &&
           IsStubThumb == Other.IsStubThumb;
  }
  inline bool operator<(const RelocationValueRef &Other) const {
    return std::tie(SectionID, Offset, Addend, IsStubThumb, SymbolName) <
           std::tie(Other.SectionID, Other.Offset, Other.Addend,
                    Other.IsStubThumb, Other.SymbolName);
  }
```
- **EN**: Introduces declarations for `RelocationValueRef`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `RelocationValueRef` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 209-218
```cpp
};

/// Symbol info for RuntimeDyld.
class SymbolTableEntry {
public:
  SymbolTableEntry() = default;

  SymbolTableEntry(unsigned SectionID, uint64_t Offset, JITSymbolFlags Flags)
      : Offset(Offset), SectionID(SectionID), Flags(Flags) {}

```
- **EN**: Introduces declarations for `SymbolTableEntry`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `SymbolTableEntry` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 219-230
```cpp
  unsigned getSectionID() const { return SectionID; }
  uint64_t getOffset() const { return Offset; }
  void setOffset(uint64_t NewOffset) { Offset = NewOffset; }

  JITSymbolFlags getFlags() const { return Flags; }

private:
  uint64_t Offset = 0;
  unsigned SectionID = 0;
  JITSymbolFlags Flags = JITSymbolFlags::None;
};

```
- **EN**: Implements logic around `getSectionID`, `getOffset`, `setOffset`, `getFlags`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getSectionID`, `getOffset`, `setOffset`, `getFlags` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 231-240
```cpp
typedef StringMap<SymbolTableEntry> RTDyldSymbolTable;

class RuntimeDyldImpl {
  friend class RuntimeDyld::LoadedObjectInfo;
protected:
  static const unsigned AbsoluteSymbolSection = ~0U;

  // The MemoryManager to load objects into.
  RuntimeDyld::MemoryManager &MemMgr;

```
- **EN**: Introduces declarations for `RuntimeDyldImpl`, `RuntimeDyld::LoadedObjectInfo`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `RuntimeDyldImpl`, `RuntimeDyld::LoadedObjectInfo` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 241-250
```cpp
  // The symbol resolver to use for external symbols.
  JITSymbolResolver &Resolver;

  // A list of all sections emitted by the dynamic linker.  These sections are
  // referenced in the code by means of their index in this list - SectionID.
  // Because references may be kept while the list grows, use a container that
  // guarantees reference stability.
  typedef std::deque<SectionEntry> SectionList;
  SectionList Sections;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 251-260
```cpp
  typedef unsigned SID; // Type for SectionIDs
#define RTDYLD_INVALID_SECTION_ID ((RuntimeDyldImpl::SID)(-1))

  // Keep a map of sections from object file to the SectionID which
  // references it.
  typedef std::map<SectionRef, unsigned> ObjSectionToIDMap;

  // A global symbol table for symbols from all loaded modules.
  RTDyldSymbolTable GlobalSymbolTable;

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 261-274
```cpp
  // Keep a map of common symbols to their info pairs
  typedef std::vector<SymbolRef> CommonSymbolList;

  // For each symbol, keep a list of relocations based on it. Anytime
  // its address is reassigned (the JIT re-compiled the function, e.g.),
  // the relocations get re-resolved.
  // The symbol (or section) the relocation is sourced from is the Key
  // in the relocation list where it's stored.
  typedef SmallVector<RelocationEntry, 64> RelocationList;
  // Relocations to sections already loaded. Indexed by SectionID which is the
  // source of the address. The target where the address will be written is
  // SectionID/Offset in the relocation itself.
  std::unordered_map<unsigned, RelocationList> Relocations;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 275-288
```cpp
  // Relocations to external symbols that are not yet resolved.  Symbols are
  // external when they aren't found in the global symbol table of all loaded
  // modules.  This map is indexed by symbol name.
  StringMap<RelocationList> ExternalSymbolRelocations;


  typedef std::map<RelocationValueRef, uintptr_t> StubMap;

  Triple::ArchType Arch;
  bool IsTargetLittleEndian;
  bool IsMipsO32ABI;
  bool IsMipsN32ABI;
  bool IsMipsN64ABI;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 289-304
```cpp
  // True if all sections should be passed to the memory manager, false if only
  // sections containing relocations should be. Defaults to 'false'.
  bool ProcessAllSections;

  // This mutex prevents simultaneously loading objects from two different
  // threads.  This keeps us from having to protect individual data structures
  // and guarantees that section allocation requests to the memory manager
  // won't be interleaved between modules.  It is also used in mapSectionAddress
  // and resolveRelocations to protect write access to internal data structures.
  //
  // loadObject may be called on the same thread during the handling of
  // processRelocations, and that's OK.  The handling of the relocation lists
  // is written in such a way as to work correctly if new elements are added to
  // the end of the list while the list is being processed.
  sys::Mutex lock;

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 305-314
```cpp
  using NotifyStubEmittedFunction =
    RuntimeDyld::NotifyStubEmittedFunction;
  NotifyStubEmittedFunction NotifyStubEmitted;

  virtual unsigned getMaxStubSize() const = 0;
  virtual Align getStubAlignment() = 0;

  bool HasError;
  std::string ErrorStr;

```
- **EN**: Implements logic around `getMaxStubSize`, `getStubAlignment`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `getMaxStubSize`, `getStubAlignment` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 315-328
```cpp
  void writeInt16BE(uint8_t *Addr, uint16_t Value) {
    llvm::support::endian::write<uint16_t>(Addr, Value,
                                           IsTargetLittleEndian
                                               ? llvm::endianness::little
                                               : llvm::endianness::big);
  }

  void writeInt32BE(uint8_t *Addr, uint32_t Value) {
    llvm::support::endian::write<uint32_t>(Addr, Value,
                                           IsTargetLittleEndian
                                               ? llvm::endianness::little
                                               : llvm::endianness::big);
  }

```
- **EN**: Implements logic around `writeInt16BE`, `write<uint16_t>`, `writeInt32BE`, `write<uint32_t>`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `writeInt16BE`, `write<uint16_t>`, `writeInt32BE`, `write<uint32_t>` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 329-341
```cpp
  void writeInt64BE(uint8_t *Addr, uint64_t Value) {
    llvm::support::endian::write<uint64_t>(Addr, Value,
                                           IsTargetLittleEndian
                                               ? llvm::endianness::little
                                               : llvm::endianness::big);
  }

  virtual void setMipsABI(const ObjectFile &Obj) {
    IsMipsO32ABI = false;
    IsMipsN32ABI = false;
    IsMipsN64ABI = false;
  }

```
- **EN**: Implements logic around `writeInt64BE`, `write<uint64_t>`, `setMipsABI`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `writeInt64BE`, `write<uint64_t>`, `setMipsABI` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 342-351
```cpp
  /// Endian-aware read Read the least significant Size bytes from Src.
  uint64_t readBytesUnaligned(uint8_t *Src, unsigned Size) const;

  /// Endian-aware write. Write the least significant Size bytes from Value to
  /// Dst.
  void writeBytesUnaligned(uint64_t Value, uint8_t *Dst, unsigned Size) const;

  /// Generate JITSymbolFlags from a libObject symbol.
  virtual Expected<JITSymbolFlags> getJITSymbolFlags(const SymbolRef &Sym);

```
- **EN**: Implements logic around `readBytesUnaligned`, `writeBytesUnaligned`, `getJITSymbolFlags`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `readBytesUnaligned`, `writeBytesUnaligned`, `getJITSymbolFlags` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 352-360
```cpp
  /// Modify the given target address based on the given symbol flags.
  /// This can be used by subclasses to tweak addresses based on symbol flags,
  /// For example: the MachO/ARM target uses it to set the low bit if the target
  /// is a thumb symbol.
  virtual uint64_t modifyAddressBasedOnFlags(uint64_t Addr,
                                             JITSymbolFlags Flags) const {
    return Addr;
  }

```
- **EN**: Implements logic around `modifyAddressBasedOnFlags`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `modifyAddressBasedOnFlags` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 361-369
```cpp
  /// Given the common symbols discovered in the object file, emit a
  /// new section for them and update the symbol mappings in the object and
  /// symbol table.
  Error emitCommonSymbols(const ObjectFile &Obj,
                          CommonSymbolList &CommonSymbols, uint64_t CommonSize,
                          uint32_t CommonAlign);

  /// Emits section data from the object file to the MemoryManager.
  /// \param IsCode if it's true then allocateCodeSection() will be
```
- **EN**: Implements logic around `emitCommonSymbols`.
- **CN**: 围绕 `emitCommonSymbols` 实现具体逻辑。

### Lines 370-378
```cpp
  ///        used for emits, else allocateDataSection() will be used.
  /// \return SectionID.
  Expected<unsigned> emitSection(const ObjectFile &Obj,
                                 const SectionRef &Section,
                                 bool IsCode);

  /// Find Section in LocalSections. If the secton is not found - emit
  ///        it and store in LocalSections.
  /// \param IsCode if it's true then allocateCodeSection() will be
```
- **EN**: Implements logic around `emitSection`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `emitSection` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 379-387
```cpp
  ///        used for emmits, else allocateDataSection() will be used.
  /// \return SectionID.
  Expected<unsigned> findOrEmitSection(const ObjectFile &Obj,
                                       const SectionRef &Section, bool IsCode,
                                       ObjSectionToIDMap &LocalSections);

  // Add a relocation entry that uses the given section.
  void addRelocationForSection(const RelocationEntry &RE, unsigned SectionID);

```
- **EN**: Implements logic around `findOrEmitSection`, `addRelocationForSection`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `findOrEmitSection`, `addRelocationForSection` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 388-396
```cpp
  // Add a relocation entry that uses the given symbol.  This symbol may
  // be found in the global symbol table, or it may be external.
  void addRelocationForSymbol(const RelocationEntry &RE, StringRef SymbolName);

  /// Emits long jump instruction to Addr.
  /// \return Pointer to the memory area for emitting target address.
  uint8_t *createStubFunction(uint8_t *Addr, unsigned AbiVariant = 0);

  /// Resolves relocations from Relocs list with address from Value.
```
- **EN**: Implements logic around `addRelocationForSymbol`, `createStubFunction`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `addRelocationForSymbol`, `createStubFunction` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 397-405
```cpp
  void resolveRelocationList(const RelocationList &Relocs, uint64_t Value);

  /// A object file specific relocation resolver
  /// \param RE The relocation to be resolved
  /// \param Value Target symbol address to apply the relocation action
  virtual void resolveRelocation(const RelocationEntry &RE, uint64_t Value) = 0;

  /// Parses one or more object file relocations (some object files use
  ///        relocation pairs) and stores it to Relocations or SymbolRelocations
```
- **EN**: Implements logic around `resolveRelocationList`, `resolveRelocation`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `resolveRelocationList`, `resolveRelocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 406-415
```cpp
  ///        (this depends on the object file type).
  /// \return Iterator to the next relocation that needs to be parsed.
  virtual Expected<relocation_iterator>
  processRelocationRef(unsigned SectionID, relocation_iterator RelI,
                       const ObjectFile &Obj, ObjSectionToIDMap &ObjSectionToID,
                       StubMap &Stubs) = 0;

  void applyExternalSymbolRelocations(
      const StringMap<JITEvaluatedSymbol> ExternalSymbolMap);

```
- **EN**: Implements logic around `processRelocationRef`, `applyExternalSymbolRelocations`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `processRelocationRef`, `applyExternalSymbolRelocations` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 416-425
```cpp
  /// Resolve relocations to external symbols.
  Error resolveExternalSymbols();

  // Compute an upper bound of the memory that is required to load all
  // sections
  Error computeTotalAllocSize(const ObjectFile &Obj, uint64_t &CodeSize,
                              Align &CodeAlign, uint64_t &RODataSize,
                              Align &RODataAlign, uint64_t &RWDataSize,
                              Align &RWDataAlign);

```
- **EN**: Implements logic around `resolveExternalSymbols`, `computeTotalAllocSize`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `resolveExternalSymbols`, `computeTotalAllocSize` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 426-435
```cpp
  // Compute GOT size
  unsigned computeGOTSize(const ObjectFile &Obj);

  // Compute the stub buffer size required for a section
  unsigned computeSectionStubBufSize(const ObjectFile &Obj,
                                     const SectionRef &Section);

  // Implementation of the generic part of the loadObject algorithm.
  Expected<ObjSectionToIDMap> loadObjectImpl(const object::ObjectFile &Obj);

```
- **EN**: Implements logic around `computeGOTSize`, `computeSectionStubBufSize`, `loadObjectImpl`.
- **CN**: 围绕 `computeGOTSize`, `computeSectionStubBufSize`, `loadObjectImpl` 实现具体逻辑。

### Lines 436-447
```cpp
  // Return size of Global Offset Table (GOT) entry
  virtual size_t getGOTEntrySize() { return 0; }

  // Hook for the subclasses to do further processing when a symbol is added to
  // the global symbol table. This function may modify the symbol table entry.
  virtual void processNewSymbol(const SymbolRef &ObjSymbol, SymbolTableEntry& Entry) {}

  // Return true if the relocation R may require allocating a GOT entry.
  virtual bool relocationNeedsGot(const RelocationRef &R) const {
    return false;
  }

```
- **EN**: Implements logic around `getGOTEntrySize`, `processNewSymbol`, `relocationNeedsGot`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getGOTEntrySize`, `processNewSymbol`, `relocationNeedsGot` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 448-457
```cpp
  // Return true if the relocation R may require allocating a stub.
  virtual bool relocationNeedsStub(const RelocationRef &R) const {
    return true;    // Conservative answer
  }

  // Return true if the relocation R may require allocating a DLL import stub.
  virtual bool relocationNeedsDLLImportStub(const RelocationRef &R) const {
    return false;
  }

```
- **EN**: Implements logic around `relocationNeedsStub`, `relocationNeedsDLLImportStub`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `relocationNeedsStub`, `relocationNeedsDLLImportStub` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 458-469
```cpp
  // Add the size of a DLL import stub to the buffer size
  virtual unsigned sizeAfterAddingDLLImportStub(unsigned Size) const {
    return Size;
  }

public:
  RuntimeDyldImpl(RuntimeDyld::MemoryManager &MemMgr,
                  JITSymbolResolver &Resolver)
    : MemMgr(MemMgr), Resolver(Resolver),
      ProcessAllSections(false), HasError(false) {
  }

```
- **EN**: Implements logic around `sizeAfterAddingDLLImportStub`, `RuntimeDyldImpl`, `MemMgr`, `ProcessAllSections`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `sizeAfterAddingDLLImportStub`, `RuntimeDyldImpl`, `MemMgr`, `ProcessAllSections` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 470-478
```cpp
  virtual ~RuntimeDyldImpl();

  void setProcessAllSections(bool ProcessAllSections) {
    this->ProcessAllSections = ProcessAllSections;
  }

  virtual std::unique_ptr<RuntimeDyld::LoadedObjectInfo>
  loadObject(const object::ObjectFile &Obj) = 0;

```
- **EN**: Implements logic around `~RuntimeDyldImpl`, `setProcessAllSections`, `loadObject`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `~RuntimeDyldImpl`, `setProcessAllSections`, `loadObject` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 479-492
```cpp
  uint64_t getSectionLoadAddress(unsigned SectionID) const {
    if (SectionID == AbsoluteSymbolSection)
      return 0;
    else
      return Sections[SectionID].getLoadAddress();
  }

  uint8_t *getSectionAddress(unsigned SectionID) const {
    if (SectionID == AbsoluteSymbolSection)
      return nullptr;
    else
      return Sections[SectionID].getAddress();
  }

```
- **EN**: Implements logic around `getSectionLoadAddress`, `getLoadAddress`, `getSectionAddress`, `getAddress`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getSectionLoadAddress`, `getLoadAddress`, `getSectionAddress`, `getAddress` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 493-501
```cpp
  StringRef getSectionContent(unsigned SectionID) const {
    if (SectionID == AbsoluteSymbolSection)
      return {};
    else
      return StringRef(
          reinterpret_cast<char *>(Sections[SectionID].getAddress()),
          Sections[SectionID].getStubOffset() + getMaxStubSize());
  }

```
- **EN**: Implements logic around `getSectionContent`, `StringRef`, `getAddress`, `getStubOffset`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getSectionContent`, `StringRef`, `getAddress`, `getStubOffset` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 502-514
```cpp
  uint8_t* getSymbolLocalAddress(StringRef Name) const {
    // FIXME: Just look up as a function for now. Overly simple of course.
    // Work in progress.
    RTDyldSymbolTable::const_iterator pos = GlobalSymbolTable.find(Name);
    if (pos == GlobalSymbolTable.end())
      return nullptr;
    const auto &SymInfo = pos->second;
    // Absolute symbols do not have a local address.
    if (SymInfo.getSectionID() == AbsoluteSymbolSection)
      return nullptr;
    return getSectionAddress(SymInfo.getSectionID()) + SymInfo.getOffset();
  }

```
- **EN**: Implements logic around `getSymbolLocalAddress`, `find`, `getSectionAddress`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getSymbolLocalAddress`, `find`, `getSectionAddress` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 515-532
```cpp
  unsigned getSymbolSectionID(StringRef Name) const {
    auto GSTItr = GlobalSymbolTable.find(Name);
    if (GSTItr == GlobalSymbolTable.end())
      return ~0U;
    return GSTItr->second.getSectionID();
  }

  JITEvaluatedSymbol getSymbol(StringRef Name) const {
    // FIXME: Just look up as a function for now. Overly simple of course.
    // Work in progress.
    RTDyldSymbolTable::const_iterator pos = GlobalSymbolTable.find(Name);
    if (pos == GlobalSymbolTable.end())
      return nullptr;
    const auto &SymEntry = pos->second;
    uint64_t SectionAddr = 0;
    if (SymEntry.getSectionID() != AbsoluteSymbolSection)
      SectionAddr = getSectionLoadAddress(SymEntry.getSectionID());
    uint64_t TargetAddr = SectionAddr + SymEntry.getOffset();
```
- **EN**: Implements logic around `getSymbolSectionID`, `find`, `getSectionID`, `getSymbol`, and 2 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getSymbolSectionID`, `find`, `getSectionID`, `getSymbol`, and 2 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 533-541
```cpp

    // FIXME: Have getSymbol should return the actual address and the client
    //        modify it based on the flags. This will require clients to be
    //        aware of the target architecture, which we should build
    //        infrastructure for.
    TargetAddr = modifyAddressBasedOnFlags(TargetAddr, SymEntry.getFlags());
    return JITEvaluatedSymbol(TargetAddr, SymEntry.getFlags());
  }

```
- **EN**: Implements logic around `modifyAddressBasedOnFlags`, `JITEvaluatedSymbol`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `modifyAddressBasedOnFlags`, `JITEvaluatedSymbol` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 542-551
```cpp
  std::map<StringRef, JITEvaluatedSymbol> getSymbolTable() const {
    std::map<StringRef, JITEvaluatedSymbol> Result;

    for (const auto &KV : GlobalSymbolTable) {
      auto SectionID = KV.second.getSectionID();
      uint64_t SectionAddr = getSectionLoadAddress(SectionID);
      Result[KV.first()] =
        JITEvaluatedSymbol(SectionAddr + KV.second.getOffset(), KV.second.getFlags());
    }

```
- **EN**: Implements logic around `getSymbolTable`, `getSectionID`, `getSectionLoadAddress`, `first`, and 1 more symbols.
- **CN**: 围绕 `getSymbolTable`, `getSectionID`, `getSectionLoadAddress`, `first`, and 1 more symbols 实现具体逻辑。

### Lines 552-567
```cpp
    return Result;
  }

  void resolveRelocations();

  void resolveLocalRelocations();

  static void finalizeAsync(
      std::unique_ptr<RuntimeDyldImpl> This,
      unique_function<void(object::OwningBinary<object::ObjectFile>,
                           std::unique_ptr<RuntimeDyld::LoadedObjectInfo>,
                           Error)>
          OnEmitted,
      object::OwningBinary<object::ObjectFile> O,
      std::unique_ptr<RuntimeDyld::LoadedObjectInfo> Info);

```
- **EN**: Implements logic around `resolveRelocations`, `resolveLocalRelocations`, `finalizeAsync`, `unique_function<void`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `resolveRelocations`, `resolveLocalRelocations`, `finalizeAsync`, `unique_function<void` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 568-577
```cpp
  void reassignSectionAddress(unsigned SectionID, uint64_t Addr);

  void mapSectionAddress(const void *LocalAddress, uint64_t TargetAddress);

  // Is the linker in an error state?
  bool hasError() { return HasError; }

  // Mark the error condition as handled and continue.
  void clearError() { HasError = false; }

```
- **EN**: Implements logic around `reassignSectionAddress`, `mapSectionAddress`, `hasError`, `clearError`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `reassignSectionAddress`, `mapSectionAddress`, `hasError`, `clearError` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 578-586
```cpp
  // Get the error message.
  StringRef getErrorString() { return ErrorStr; }

  virtual bool isCompatibleFile(const ObjectFile &Obj) const = 0;

  void setNotifyStubEmitted(NotifyStubEmittedFunction NotifyStubEmitted) {
    this->NotifyStubEmitted = std::move(NotifyStubEmitted);
  }

```
- **EN**: Implements logic around `getErrorString`, `isCompatibleFile`, `setNotifyStubEmitted`, `move`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getErrorString`, `isCompatibleFile`, `setNotifyStubEmitted`, `move` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 587-596
```cpp
  virtual void registerEHFrames();

  void deregisterEHFrames();

  virtual Error finalizeLoad(const ObjectFile &ObjImg,
                             ObjSectionToIDMap &SectionMap) {
    return Error::success();
  }
};

```
- **EN**: Implements logic around `registerEHFrames`, `deregisterEHFrames`, `finalizeLoad`, `success`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `registerEHFrames`, `deregisterEHFrames`, `finalizeLoad`, `success` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 597-599
```cpp
} // end namespace llvm

#endif
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

- **Direct includes / 直接包含**: `llvm/ADT/SmallVector.h`, `llvm/ADT/StringMap.h`, `llvm/ExecutionEngine/Orc/SymbolStringPool.h`, `llvm/ExecutionEngine/RTDyldMemoryManager.h`, `llvm/ExecutionEngine/RuntimeDyld.h`, `llvm/ExecutionEngine/RuntimeDyldChecker.h`, `llvm/Object/ObjectFile.h`, `llvm/Support/Debug.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/Format.h`, `llvm/Support/Mutex.h`, `llvm/Support/SwapByteOrder.h` ... (+6 more)
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Object, Support, Target/TargetParser
