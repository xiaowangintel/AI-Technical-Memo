# DLL.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `lld/COFF/DLL.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: DLL.cpp This file defines various types of chunks for the DLL import or export descriptor tables. They are inherently Windows-specific. You need to read Microsoft PE/COFF spec to understand details. It also sits in code that implements COFF/PE linking components in LLD. / 该文件实现 LLD 中的 COFF/PE 链接组件。 源码头部说明其职责是：DLL.cpp This file defines various types of chunks for the DLL import or export descriptor tables. They are inherently Windows-specific. You need to read Microsoft PE/COFF spec to understand details。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```cpp
//===- DLL.cpp ------------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines various types of chunks for the DLL import or export
// descriptor tables. They are inherently Windows-specific.
// You need to read Microsoft PE/COFF spec to understand details
// about the data structures.
//
// If you are not particularly interested in linking against Windows
// DLL, you can skip this file, and you should still be able to
// understand the rest of the linker.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 19-28

```cpp

#include "DLL.h"
#include "COFFLinkerContext.h"
#include "Chunks.h"
#include "SymbolTable.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/Object/COFF.h"
#include "llvm/Support/Endian.h"
#include "llvm/Support/Path.h"
```

- EN: Pulls in 8 header(s) from LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 8 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 29-36

```cpp
using namespace llvm;
using namespace llvm::object;
using namespace llvm::support::endian;
using namespace llvm::COFF;

namespace lld::coff {
namespace {
```

- EN: Works inside namespace scope `llvm`, `lld` to organize symbols. Notable symbols here include `llvm`, `lld`.
- CN: 这里位于命名空间 `llvm`, `lld` 中，用于组织符号作用域。这里较值得关注的符号包括 `llvm`, `lld`。

### Lines 37-49

```cpp
// Import table

// A chunk for the import descriptor table.
class HintNameChunk : public NonSectionChunk {
public:
  HintNameChunk(StringRef n, uint16_t h) : name(n), hint(h) {}

  size_t getSize() const override {
    // Starts with 2 byte Hint field, followed by a null-terminated string,
    // ends with 0 or 1 byte padding.
    return alignTo(name.size() + 3, 2);
  }
```

- EN: Introduces type definitions such as `HintNameChunk`. Declares or implements routines including `HintNameChunk`, `getSize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `HintNameChunk`, `getSize`.
- CN: 这里引入类型定义，例如 `HintNameChunk`。这里声明或实现函数，例如 `HintNameChunk`, `getSize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `HintNameChunk`, `getSize`。

### Lines 50-60

```cpp
  void writeTo(uint8_t *buf) const override {
    memset(buf, 0, getSize());
    write16le(buf, hint);
    memcpy(buf + 2, name.data(), name.size());
  }

private:
  StringRef name;
  uint16_t hint;
};
```

- EN: Declares or implements routines including `writeTo`, `memset`, `write16le`, `memcpy`. Notable symbols here include `writeTo`, `memset`, `write16le`, `memcpy`.
- CN: 这里声明或实现函数，例如 `writeTo`, `memset`, `write16le`, `memcpy`。这里较值得关注的符号包括 `writeTo`, `memset`, `write16le`, `memcpy`。

### Lines 61-69

```cpp
// A chunk for the import descriptor table.
class LookupChunk : public NonSectionChunk {
public:
  explicit LookupChunk(COFFLinkerContext &ctx, Chunk *c)
      : hintName(c), ctx(ctx) {
    setAlignment(ctx.config.wordsize);
  }
  size_t getSize() const override { return ctx.config.wordsize; }
```

- EN: Introduces type definitions such as `LookupChunk`. Declares or implements routines including `LookupChunk`, `hintName`, `setAlignment`, `getSize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LookupChunk`, `hintName`, `setAlignment`, `getSize`.
- CN: 这里引入类型定义，例如 `LookupChunk`。这里声明或实现函数，例如 `LookupChunk`, `hintName`, `setAlignment`, `getSize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LookupChunk`, `hintName`, `setAlignment`, `getSize`。

### Lines 70-78

```cpp
  void writeTo(uint8_t *buf) const override {
    if (ctx.config.is64())
      write64le(buf, hintName->getRVA());
    else
      write32le(buf, hintName->getRVA());
  }

  Chunk *hintName;
```

- EN: Declares or implements routines including `writeTo`, `write64le`, `write32le`. Notable symbols here include `writeTo`, `write64le`, `write32le`.
- CN: 这里声明或实现函数，例如 `writeTo`, `write64le`, `write32le`。这里较值得关注的符号包括 `writeTo`, `write64le`, `write32le`。

### Lines 79-93

```cpp
private:
  COFFLinkerContext &ctx;
};

// A chunk for the import descriptor table.
// This chunk represent import-by-ordinal symbols.
// See Microsoft PE/COFF spec 7.1. Import Header for details.
class OrdinalOnlyChunk : public NonSectionChunk {
public:
  explicit OrdinalOnlyChunk(COFFLinkerContext &c, uint16_t v)
      : ordinal(v), ctx(c) {
    setAlignment(ctx.config.wordsize);
  }
  size_t getSize() const override { return ctx.config.wordsize; }
```

- EN: Introduces type definitions such as `OrdinalOnlyChunk`. Declares or implements routines including `OrdinalOnlyChunk`, `ordinal`, `setAlignment`, `getSize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `OrdinalOnlyChunk`, `ordinal`, `setAlignment`, `getSize`.
- CN: 这里引入类型定义，例如 `OrdinalOnlyChunk`。这里声明或实现函数，例如 `OrdinalOnlyChunk`, `ordinal`, `setAlignment`, `getSize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `OrdinalOnlyChunk`, `ordinal`, `setAlignment`, `getSize`。

### Lines 94-103

```cpp
  void writeTo(uint8_t *buf) const override {
    // An import-by-ordinal slot has MSB 1 to indicate that
    // this is import-by-ordinal (and not import-by-name).
    if (ctx.config.is64()) {
      write64le(buf, (1ULL << 63) | ordinal);
    } else {
      write32le(buf, (1ULL << 31) | ordinal);
    }
  }
```

- EN: Declares or implements routines including `writeTo`, `write64le`, `write32le`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `writeTo`, `write64le`, `write32le`.
- CN: 这里声明或实现函数，例如 `writeTo`, `write64le`, `write32le`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `writeTo`, `write64le`, `write32le`。

### Lines 104-115

```cpp
  uint16_t ordinal;

private:
  COFFLinkerContext &ctx;
};

// A chunk for the import descriptor table.
class ImportDirectoryChunk : public NonSectionChunk {
public:
  explicit ImportDirectoryChunk(Chunk *n) : dllName(n) { setAlignment(4); }
  size_t getSize() const override { return sizeof(ImportDirectoryTableEntry); }
```

- EN: Introduces type definitions such as `ImportDirectoryChunk`. Declares or implements routines including `ImportDirectoryChunk`, `getSize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `ImportDirectoryChunk`, `getSize`.
- CN: 这里引入类型定义，例如 `ImportDirectoryChunk`。这里声明或实现函数，例如 `ImportDirectoryChunk`, `getSize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `ImportDirectoryChunk`, `getSize`。

### Lines 116-124

```cpp
  void writeTo(uint8_t *buf) const override {
    memset(buf, 0, getSize());

    auto *e = (coff_import_directory_table_entry *)(buf);
    e->ImportLookupTableRVA = lookupTab->getRVA();
    e->NameRVA = dllName->getRVA();
    e->ImportAddressTableRVA = addressTab->getRVA();
  }
```

- EN: Declares or implements routines including `writeTo`, `memset`, `getRVA`. Notable symbols here include `writeTo`, `memset`, `getRVA`.
- CN: 这里声明或实现函数，例如 `writeTo`, `memset`, `getRVA`。这里较值得关注的符号包括 `writeTo`, `memset`, `getRVA`。

### Lines 125-142

```cpp
  Chunk *dllName;
  Chunk *lookupTab;
  Chunk *addressTab;
};

// A chunk representing null terminator in the import table.
// Contents of this chunk is always null bytes.
class NullChunk : public NonSectionChunk {
public:
  explicit NullChunk(size_t n, uint32_t align) : size(n) {
    setAlignment(align);
  }
  explicit NullChunk(COFFLinkerContext &ctx)
      : NullChunk(ctx.config.wordsize, ctx.config.wordsize) {}
  explicit NullChunk(COFFLinkerContext &ctx, size_t n)
      : NullChunk(n, ctx.config.wordsize) {}
  size_t getSize() const override { return size; }
```

- EN: Introduces type definitions such as `NullChunk`. Declares or implements routines including `NullChunk`, `setAlignment`, `getSize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `NullChunk`, `setAlignment`, `getSize`.
- CN: 这里引入类型定义，例如 `NullChunk`。这里声明或实现函数，例如 `NullChunk`, `setAlignment`, `getSize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `NullChunk`, `setAlignment`, `getSize`。

### Lines 143-150

```cpp
  void writeTo(uint8_t *buf) const override {
    memset(buf, 0, size);
  }

private:
  size_t size;
};
```

- EN: Declares or implements routines including `writeTo`, `memset`. Notable symbols here include `writeTo`, `memset`.
- CN: 这里声明或实现函数，例如 `writeTo`, `memset`。这里较值得关注的符号包括 `writeTo`, `memset`。

### Lines 151-158

```cpp
// A chunk for ARM64EC auxiliary IAT.
class AuxImportChunk : public NonSectionChunk {
public:
  explicit AuxImportChunk(ImportFile *file) : file(file) {
    setAlignment(sizeof(uint64_t));
  }
  size_t getSize() const override { return sizeof(uint64_t); }
```

- EN: Introduces type definitions such as `AuxImportChunk`. Declares or implements routines including `AuxImportChunk`, `setAlignment`, `getSize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `AuxImportChunk`, `setAlignment`, `getSize`.
- CN: 这里引入类型定义，例如 `AuxImportChunk`。这里声明或实现函数，例如 `AuxImportChunk`, `setAlignment`, `getSize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `AuxImportChunk`, `setAlignment`, `getSize`。

### Lines 159-166

```cpp
  void writeTo(uint8_t *buf) const override {
    uint64_t impchkVA = 0;
    if (file->impchkThunk)
      impchkVA =
          file->impchkThunk->getRVA() + file->symtab.ctx.config.imageBase;
    write64le(buf, impchkVA);
  }
```

- EN: Declares or implements routines including `writeTo`, `getRVA`, `write64le`. Notable symbols here include `writeTo`, `getRVA`, `write64le`.
- CN: 这里声明或实现函数，例如 `writeTo`, `getRVA`, `write64le`。这里较值得关注的符号包括 `writeTo`, `getRVA`, `write64le`。

### Lines 167-175

```cpp
  void getBaserels(std::vector<Baserel> *res) override {
    if (file->impchkThunk)
      res->emplace_back(rva, file->symtab.machine);
  }

private:
  ImportFile *file;
};
```

- EN: Declares or implements routines including `getBaserels`, `emplace_back`. Notable symbols here include `getBaserels`, `emplace_back`.
- CN: 这里声明或实现函数，例如 `getBaserels`, `emplace_back`。这里较值得关注的符号包括 `getBaserels`, `emplace_back`。

### Lines 176-188

```cpp
static std::vector<std::vector<DefinedImportData *>>
binImports(COFFLinkerContext &ctx,
           const std::vector<DefinedImportData *> &imports) {
  // Group DLL-imported symbols by DLL name because that's how
  // symbols are laid out in the import descriptor table.
  auto less = [&ctx](const std::string &a, const std::string &b) {
    return ctx.config.dllOrder[a] < ctx.config.dllOrder[b];
  };
  std::map<std::string, std::vector<DefinedImportData *>, decltype(less)> m(
      less);
  for (DefinedImportData *sym : imports)
    m[sym->getDLLName().lower()].push_back(sym);
```

- EN: Declares or implements routines including `decltype`, `getDLLName`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `decltype`, `getDLLName`.
- CN: 这里声明或实现函数，例如 `decltype`, `getDLLName`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `decltype`, `getDLLName`。

### Lines 189-206

```cpp
  std::vector<std::vector<DefinedImportData *>> v;
  for (auto &kv : m) {
    // Sort symbols by name for each group.
    std::vector<DefinedImportData *> &syms = kv.second;
    llvm::sort(syms, [](DefinedImportData *a, DefinedImportData *b) {
      auto getBaseName = [](DefinedImportData *sym) {
        StringRef name = sym->getName();
        name.consume_front("__imp_");
        // Skip aux_ part of ARM64EC function symbol name.
        if (sym->file->impchkThunk)
          name.consume_front("aux_");
        return name;
      };
      return getBaseName(a) < getBaseName(b);
    });
    v.push_back(std::move(syms));
  }
  return v;
```

- EN: Declares or implements routines including `sort`, `getName`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `sort`, `getName`.
- CN: 这里声明或实现函数，例如 `sort`, `getName`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `sort`, `getName`。

### Lines 207-215

```cpp
}

// See Microsoft PE/COFF spec 4.3 for details.

// A chunk for the delay import descriptor table etnry.
class DelayDirectoryChunk : public NonSectionChunk {
public:
  explicit DelayDirectoryChunk(Chunk *n) : dllName(n) { setAlignment(4); }
```

- EN: Introduces type definitions such as `DelayDirectoryChunk`. Declares or implements routines including `DelayDirectoryChunk`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `DelayDirectoryChunk`.
- CN: 这里引入类型定义，例如 `DelayDirectoryChunk`。这里声明或实现函数，例如 `DelayDirectoryChunk`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `DelayDirectoryChunk`。

### Lines 216-230

```cpp
  size_t getSize() const override {
    return sizeof(delay_import_directory_table_entry);
  }

  void writeTo(uint8_t *buf) const override {
    memset(buf, 0, getSize());

    auto *e = (delay_import_directory_table_entry *)(buf);
    e->Attributes = 1;
    e->Name = dllName->getRVA();
    e->ModuleHandle = moduleHandle->getRVA();
    e->DelayImportAddressTable = addressTab->getRVA();
    e->DelayImportNameTable = nameTab->getRVA();
  }
```

- EN: Declares or implements routines including `getSize`, `writeTo`, `memset`, `getRVA`. Notable symbols here include `getSize`, `writeTo`, `memset`, `getRVA`.
- CN: 这里声明或实现函数，例如 `getSize`, `writeTo`, `memset`, `getRVA`。这里较值得关注的符号包括 `getSize`, `writeTo`, `memset`, `getRVA`。

### Lines 231-245

```cpp
  Chunk *dllName;
  Chunk *moduleHandle;
  Chunk *addressTab;
  Chunk *nameTab;
};

// Initial contents for delay-loaded functions.
// This code calls __delayLoadHelper2 function to resolve a symbol
// which then overwrites its jump table slot with the result
// for subsequent function calls.
static const uint8_t thunkX64[] = {
    0x48, 0x8D, 0x05, 0, 0, 0, 0,       // lea     rax, [__imp_<FUNCNAME>]
    0xE9, 0, 0, 0, 0,                   // jmp     __tailMerge_<lib>
};
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 246-263

```cpp
static const uint8_t tailMergeX64[] = {
    0x48, 0x89, 0x4C, 0x24, 0x08,          // mov    qword ptr [rsp+8], rcx
    0x48, 0x89, 0x54, 0x24, 0x10,          // mov    qword ptr [rsp+10h], rdx
    0x4C, 0x89, 0x44, 0x24, 0x18,          // mov    qword ptr [rsp+18h], r8
    0x4C, 0x89, 0x4C, 0x24, 0x20,          // mov    qword ptr [rsp+20h], r9
    0x48, 0x83, 0xEC, 0x68,                // sub    rsp, 68h
    0x66, 0x0F, 0x7F, 0x44, 0x24, 0x20,    // movdqa xmmword ptr [rsp+20h], xmm0
    0x66, 0x0F, 0x7F, 0x4C, 0x24, 0x30,    // movdqa xmmword ptr [rsp+30h], xmm1
    0x66, 0x0F, 0x7F, 0x54, 0x24, 0x40,    // movdqa xmmword ptr [rsp+40h], xmm2
    0x66, 0x0F, 0x7F, 0x5C, 0x24, 0x50,    // movdqa xmmword ptr [rsp+50h], xmm3
    0x48, 0x8B, 0xD0,                      // mov    rdx, rax
    0x48, 0x8D, 0x0D, 0, 0, 0, 0,          // lea    rcx, [___DELAY_IMPORT_...]
    0xE8, 0, 0, 0, 0,                      // call   __delayLoadHelper2
    0x66, 0x0F, 0x6F, 0x44, 0x24, 0x20,    // movdqa xmm0, xmmword ptr [rsp+20h]
    0x66, 0x0F, 0x6F, 0x4C, 0x24, 0x30,    // movdqa xmm1, xmmword ptr [rsp+30h]
    0x66, 0x0F, 0x6F, 0x54, 0x24, 0x40,    // movdqa xmm2, xmmword ptr [rsp+40h]
    0x66, 0x0F, 0x6F, 0x5C, 0x24, 0x50,    // movdqa xmm3, xmmword ptr [rsp+50h]
    0x48, 0x8B, 0x4C, 0x24, 0x70,          // mov    rcx, qword ptr [rsp+70h]
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 264-279

```cpp
    0x48, 0x8B, 0x54, 0x24, 0x78,          // mov    rdx, qword ptr [rsp+78h]
    0x4C, 0x8B, 0x84, 0x24, 0x80, 0, 0, 0, // mov    r8, qword ptr [rsp+80h]
    0x4C, 0x8B, 0x8C, 0x24, 0x88, 0, 0, 0, // mov    r9, qword ptr [rsp+88h]
    0x48, 0x83, 0xC4, 0x68,                // add    rsp, 68h
    0xFF, 0xE0,                            // jmp    rax
};

static const uint8_t tailMergeUnwindInfoX64[] = {
    0x01,       // Version=1, Flags=UNW_FLAG_NHANDLER
    0x18,       // Size of prolog
    0x01,       // Count of unwind codes
    0x00,       // No frame register
    0x18, 0xC2, // Offset 0x18: UWOP_ALLOC_SMALL(0x68)
    0x00, 0x00  // Padding to align on 32-bits
};
```

- EN: Declares or implements routines including `UWOP_ALLOC_SMALL`. Notable symbols here include `UWOP_ALLOC_SMALL`.
- CN: 这里声明或实现函数，例如 `UWOP_ALLOC_SMALL`。这里较值得关注的符号包括 `UWOP_ALLOC_SMALL`。

### Lines 280-295

```cpp
static const uint8_t thunkX86[] = {
    0xB8, 0, 0, 0, 0,  // mov   eax, offset ___imp__<FUNCNAME>
    0xE9, 0, 0, 0, 0,  // jmp   __tailMerge_<lib>
};

static const uint8_t tailMergeX86[] = {
    0x51,              // push  ecx
    0x52,              // push  edx
    0x50,              // push  eax
    0x68, 0, 0, 0, 0,  // push  offset ___DELAY_IMPORT_DESCRIPTOR_<DLLNAME>_dll
    0xE8, 0, 0, 0, 0,  // call  ___delayLoadHelper2@8
    0x5A,              // pop   edx
    0x59,              // pop   ecx
    0xFF, 0xE0,        // jmp   eax
};
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 296-313

```cpp
static const uint8_t thunkARM[] = {
    0x40, 0xf2, 0x00, 0x0c, // mov.w   ip, #0 __imp_<FUNCNAME>
    0xc0, 0xf2, 0x00, 0x0c, // mov.t   ip, #0 __imp_<FUNCNAME>
    0x00, 0xf0, 0x00, 0xb8, // b.w     __tailMerge_<lib>
};

static const uint8_t tailMergeARM[] = {
    0x2d, 0xe9, 0x0f, 0x48, // push.w  {r0, r1, r2, r3, r11, lr}
    0x0d, 0xf2, 0x10, 0x0b, // addw    r11, sp, #16
    0x2d, 0xed, 0x10, 0x0b, // vpush   {d0, d1, d2, d3, d4, d5, d6, d7}
    0x61, 0x46,             // mov     r1, ip
    0x40, 0xf2, 0x00, 0x00, // mov.w   r0, #0 DELAY_IMPORT_DESCRIPTOR
    0xc0, 0xf2, 0x00, 0x00, // mov.t   r0, #0 DELAY_IMPORT_DESCRIPTOR
    0x00, 0xf0, 0x00, 0xd0, // bl      #0 __delayLoadHelper2
    0x84, 0x46,             // mov     ip, r0
    0xbd, 0xec, 0x10, 0x0b, // vpop    {d0, d1, d2, d3, d4, d5, d6, d7}
    0xbd, 0xe8, 0x0f, 0x48, // pop.w   {r0, r1, r2, r3, r11, lr}
    0x60, 0x47,             // bx      ip
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 314-321

```cpp
};

static const uint8_t thunkARM64[] = {
    0x11, 0x00, 0x00, 0x90, // adrp    x17, #0      __imp_<FUNCNAME>
    0x31, 0x02, 0x00, 0x91, // add     x17, x17, #0 :lo12:__imp_<FUNCNAME>
    0x00, 0x00, 0x00, 0x14, // b       __tailMerge_<lib>
};
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 322-339

```cpp
static const uint8_t tailMergeARM64[] = {
    0xfd, 0x7b, 0xb2, 0xa9, // stp     x29, x30, [sp, #-224]!
    0xfd, 0x03, 0x00, 0x91, // mov     x29, sp
    0xe0, 0x07, 0x01, 0xa9, // stp     x0, x1, [sp, #16]
    0xe2, 0x0f, 0x02, 0xa9, // stp     x2, x3, [sp, #32]
    0xe4, 0x17, 0x03, 0xa9, // stp     x4, x5, [sp, #48]
    0xe6, 0x1f, 0x04, 0xa9, // stp     x6, x7, [sp, #64]
    0xe8, 0x2b, 0x00, 0xf9, // str     x8,     [sp, #80]
    0xe0, 0x07, 0x03, 0xad, // stp     q0, q1, [sp, #96]
    0xe2, 0x0f, 0x04, 0xad, // stp     q2, q3, [sp, #128]
    0xe4, 0x17, 0x05, 0xad, // stp     q4, q5, [sp, #160]
    0xe6, 0x1f, 0x06, 0xad, // stp     q6, q7, [sp, #192]
    0xe1, 0x03, 0x11, 0xaa, // mov     x1, x17
    0x00, 0x00, 0x00, 0x90, // adrp    x0, #0     DELAY_IMPORT_DESCRIPTOR
    0x00, 0x00, 0x00, 0x91, // add     x0, x0, #0 :lo12:DELAY_IMPORT_DESCRIPTOR
    0x02, 0x00, 0x00, 0x90, // adrp    x2, #0     __delayLoadHelper2
    0x42, 0x00, 0x00, 0x91, // add     x2, x2, #0 :lo12:__delayLoadHelper2
    0x40, 0x00, 0x3f, 0xd6, // blr     x2
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 340-353

```cpp
    0xf0, 0x03, 0x00, 0xaa, // mov     x16, x0
    0xe6, 0x1f, 0x46, 0xad, // ldp     q6, q7, [sp, #192]
    0xe4, 0x17, 0x45, 0xad, // ldp     q4, q5, [sp, #160]
    0xe2, 0x0f, 0x44, 0xad, // ldp     q2, q3, [sp, #128]
    0xe0, 0x07, 0x43, 0xad, // ldp     q0, q1, [sp, #96]
    0xe8, 0x2b, 0x40, 0xf9, // ldr     x8,     [sp, #80]
    0xe6, 0x1f, 0x44, 0xa9, // ldp     x6, x7, [sp, #64]
    0xe4, 0x17, 0x43, 0xa9, // ldp     x4, x5, [sp, #48]
    0xe2, 0x0f, 0x42, 0xa9, // ldp     x2, x3, [sp, #32]
    0xe0, 0x07, 0x41, 0xa9, // ldp     x0, x1, [sp, #16]
    0xfd, 0x7b, 0xce, 0xa8, // ldp     x29, x30, [sp], #224
    0x00, 0x02, 0x1f, 0xd6, // br      x16
};
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 354-361

```cpp
// A chunk for the delay import thunk.
class ThunkChunkX64 : public NonSectionCodeChunk {
public:
  ThunkChunkX64(Defined *i, Chunk *tm) : imp(i), tailMerge(tm) {}

  size_t getSize() const override { return sizeof(thunkX64); }
  MachineTypes getMachine() const override { return AMD64; }
```

- EN: Introduces type definitions such as `ThunkChunkX64`. Declares or implements routines including `ThunkChunkX64`, `getSize`, `getMachine`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `ThunkChunkX64`, `getSize`, `getMachine`.
- CN: 这里引入类型定义，例如 `ThunkChunkX64`。这里声明或实现函数，例如 `ThunkChunkX64`, `getSize`, `getMachine`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `ThunkChunkX64`, `getSize`, `getMachine`。

### Lines 362-371

```cpp
  void writeTo(uint8_t *buf) const override {
    memcpy(buf, thunkX64, sizeof(thunkX64));
    write32le(buf + 3, imp->getRVA() - rva - 7);
    write32le(buf + 8, tailMerge->getRVA() - rva - 12);
  }

  Defined *imp = nullptr;
  Chunk *tailMerge = nullptr;
};
```

- EN: Declares or implements routines including `writeTo`, `memcpy`, `write32le`. Notable symbols here include `writeTo`, `memcpy`, `write32le`.
- CN: 这里声明或实现函数，例如 `writeTo`, `memcpy`, `write32le`。这里较值得关注的符号包括 `writeTo`, `memcpy`, `write32le`。

### Lines 372-384

```cpp
class TailMergeChunkX64 : public NonSectionCodeChunk {
public:
  TailMergeChunkX64(Chunk *d, Defined *h) : desc(d), helper(h) {}

  size_t getSize() const override { return sizeof(tailMergeX64); }
  MachineTypes getMachine() const override { return AMD64; }

  void writeTo(uint8_t *buf) const override {
    memcpy(buf, tailMergeX64, sizeof(tailMergeX64));
    write32le(buf + 54, desc->getRVA() - rva - 58);
    write32le(buf + 59, helper->getRVA() - rva - 63);
  }
```

- EN: Introduces type definitions such as `TailMergeChunkX64`. Declares or implements routines including `TailMergeChunkX64`, `getSize`, `getMachine`, `writeTo`, `memcpy`, and 1 more. Notable symbols here include `TailMergeChunkX64`, `getSize`, `getMachine`, `writeTo`, `memcpy`, `write32le`.
- CN: 这里引入类型定义，例如 `TailMergeChunkX64`。这里声明或实现函数，例如 `TailMergeChunkX64`, `getSize`, `getMachine`, `writeTo`, `memcpy`, and 1 more。这里较值得关注的符号包括 `TailMergeChunkX64`, `getSize`, `getMachine`, `writeTo`, `memcpy`, `write32le`。

### Lines 385-396

```cpp
  Chunk *desc = nullptr;
  Defined *helper = nullptr;
};

class TailMergePDataChunkX64 : public NonSectionChunk {
public:
  TailMergePDataChunkX64(Chunk *tm, Chunk *unwind) : tm(tm), unwind(unwind) {
    // See
    // https://learn.microsoft.com/en-us/cpp/build/exception-handling-x64#struct-runtime_function
    setAlignment(4);
  }
```

- EN: Introduces type definitions such as `TailMergePDataChunkX64`. Declares or implements routines including `TailMergePDataChunkX64`, `setAlignment`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `TailMergePDataChunkX64`, `setAlignment`.
- CN: 这里引入类型定义，例如 `TailMergePDataChunkX64`。这里声明或实现函数，例如 `TailMergePDataChunkX64`, `setAlignment`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `TailMergePDataChunkX64`, `setAlignment`。

### Lines 397-405

```cpp
  size_t getSize() const override { return 3 * sizeof(uint32_t); }
  MachineTypes getMachine() const override { return AMD64; }

  void writeTo(uint8_t *buf) const override {
    write32le(buf + 0, tm->getRVA()); // TailMergeChunk start RVA
    write32le(buf + 4, tm->getRVA() + tm->getSize()); // TailMergeChunk stop RVA
    write32le(buf + 8, unwind->getRVA());             // UnwindInfo RVA
  }
```

- EN: Declares or implements routines including `getSize`, `getMachine`, `writeTo`, `write32le`. Notable symbols here include `getSize`, `getMachine`, `writeTo`, `write32le`.
- CN: 这里声明或实现函数，例如 `getSize`, `getMachine`, `writeTo`, `write32le`。这里较值得关注的符号包括 `getSize`, `getMachine`, `writeTo`, `write32le`。

### Lines 406-417

```cpp
  Chunk *tm = nullptr;
  Chunk *unwind = nullptr;
};

class TailMergeUnwindInfoX64 : public NonSectionChunk {
public:
  TailMergeUnwindInfoX64() {
    // See
    // https://learn.microsoft.com/en-us/cpp/build/exception-handling-x64#struct-unwind_info
    setAlignment(4);
  }
```

- EN: Introduces type definitions such as `TailMergeUnwindInfoX64`. Declares or implements routines including `TailMergeUnwindInfoX64`, `setAlignment`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `TailMergeUnwindInfoX64`, `setAlignment`.
- CN: 这里引入类型定义，例如 `TailMergeUnwindInfoX64`。这里声明或实现函数，例如 `TailMergeUnwindInfoX64`, `setAlignment`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `TailMergeUnwindInfoX64`, `setAlignment`。

### Lines 418-425

```cpp
  size_t getSize() const override { return sizeof(tailMergeUnwindInfoX64); }
  MachineTypes getMachine() const override { return AMD64; }

  void writeTo(uint8_t *buf) const override {
    memcpy(buf, tailMergeUnwindInfoX64, sizeof(tailMergeUnwindInfoX64));
  }
};
```

- EN: Declares or implements routines including `getSize`, `getMachine`, `writeTo`, `memcpy`. Notable symbols here include `getSize`, `getMachine`, `writeTo`, `memcpy`.
- CN: 这里声明或实现函数，例如 `getSize`, `getMachine`, `writeTo`, `memcpy`。这里较值得关注的符号包括 `getSize`, `getMachine`, `writeTo`, `memcpy`。

### Lines 426-433

```cpp
class ThunkChunkX86 : public NonSectionCodeChunk {
public:
  ThunkChunkX86(COFFLinkerContext &ctx, Defined *i, Chunk *tm)
      : imp(i), tailMerge(tm), ctx(ctx) {}

  size_t getSize() const override { return sizeof(thunkX86); }
  MachineTypes getMachine() const override { return I386; }
```

- EN: Introduces type definitions such as `ThunkChunkX86`. Declares or implements routines including `ThunkChunkX86`, `imp`, `getSize`, `getMachine`. Notable symbols here include `ThunkChunkX86`, `imp`, `getSize`, `getMachine`.
- CN: 这里引入类型定义，例如 `ThunkChunkX86`。这里声明或实现函数，例如 `ThunkChunkX86`, `imp`, `getSize`, `getMachine`。这里较值得关注的符号包括 `ThunkChunkX86`, `imp`, `getSize`, `getMachine`。

### Lines 434-443

```cpp
  void writeTo(uint8_t *buf) const override {
    memcpy(buf, thunkX86, sizeof(thunkX86));
    write32le(buf + 1, imp->getRVA() + ctx.config.imageBase);
    write32le(buf + 6, tailMerge->getRVA() - rva - 10);
  }

  void getBaserels(std::vector<Baserel> *res) override {
    res->emplace_back(rva + 1, ctx.config.machine);
  }
```

- EN: Declares or implements routines including `writeTo`, `memcpy`, `write32le`, `getBaserels`, `emplace_back`. Notable symbols here include `writeTo`, `memcpy`, `write32le`, `getBaserels`, `emplace_back`.
- CN: 这里声明或实现函数，例如 `writeTo`, `memcpy`, `write32le`, `getBaserels`, `emplace_back`。这里较值得关注的符号包括 `writeTo`, `memcpy`, `write32le`, `getBaserels`, `emplace_back`。

### Lines 444-455

```cpp
  Defined *imp = nullptr;
  Chunk *tailMerge = nullptr;

private:
  const COFFLinkerContext &ctx;
};

class TailMergeChunkX86 : public NonSectionCodeChunk {
public:
  TailMergeChunkX86(COFFLinkerContext &ctx, Chunk *d, Defined *h)
      : desc(d), helper(h), ctx(ctx) {}
```

- EN: Introduces type definitions such as `TailMergeChunkX86`. Declares or implements routines including `TailMergeChunkX86`, `desc`. Notable symbols here include `TailMergeChunkX86`, `desc`.
- CN: 这里引入类型定义，例如 `TailMergeChunkX86`。这里声明或实现函数，例如 `TailMergeChunkX86`, `desc`。这里较值得关注的符号包括 `TailMergeChunkX86`, `desc`。

### Lines 456-464

```cpp
  size_t getSize() const override { return sizeof(tailMergeX86); }
  MachineTypes getMachine() const override { return I386; }

  void writeTo(uint8_t *buf) const override {
    memcpy(buf, tailMergeX86, sizeof(tailMergeX86));
    write32le(buf + 4, desc->getRVA() + ctx.config.imageBase);
    write32le(buf + 9, helper->getRVA() - rva - 13);
  }
```

- EN: Declares or implements routines including `getSize`, `getMachine`, `writeTo`, `memcpy`, `write32le`. Notable symbols here include `getSize`, `getMachine`, `writeTo`, `memcpy`, `write32le`.
- CN: 这里声明或实现函数，例如 `getSize`, `getMachine`, `writeTo`, `memcpy`, `write32le`。这里较值得关注的符号包括 `getSize`, `getMachine`, `writeTo`, `memcpy`, `write32le`。

### Lines 465-475

```cpp
  void getBaserels(std::vector<Baserel> *res) override {
    res->emplace_back(rva + 4, ctx.config.machine);
  }

  Chunk *desc = nullptr;
  Defined *helper = nullptr;

private:
  const COFFLinkerContext &ctx;
};
```

- EN: Declares or implements routines including `getBaserels`, `emplace_back`. Notable symbols here include `getBaserels`, `emplace_back`.
- CN: 这里声明或实现函数，例如 `getBaserels`, `emplace_back`。这里较值得关注的符号包括 `getBaserels`, `emplace_back`。

### Lines 476-485

```cpp
class ThunkChunkARM : public NonSectionCodeChunk {
public:
  ThunkChunkARM(COFFLinkerContext &ctx, Defined *i, Chunk *tm)
      : imp(i), tailMerge(tm), ctx(ctx) {
    setAlignment(2);
  }

  size_t getSize() const override { return sizeof(thunkARM); }
  MachineTypes getMachine() const override { return ARMNT; }
```

- EN: Introduces type definitions such as `ThunkChunkARM`. Declares or implements routines including `ThunkChunkARM`, `imp`, `setAlignment`, `getSize`, `getMachine`. Notable symbols here include `ThunkChunkARM`, `imp`, `setAlignment`, `getSize`, `getMachine`.
- CN: 这里引入类型定义，例如 `ThunkChunkARM`。这里声明或实现函数，例如 `ThunkChunkARM`, `imp`, `setAlignment`, `getSize`, `getMachine`。这里较值得关注的符号包括 `ThunkChunkARM`, `imp`, `setAlignment`, `getSize`, `getMachine`。

### Lines 486-495

```cpp
  void writeTo(uint8_t *buf) const override {
    memcpy(buf, thunkARM, sizeof(thunkARM));
    applyMOV32T(buf + 0, imp->getRVA() + ctx.config.imageBase);
    applyBranch24T(buf + 8, tailMerge->getRVA() - rva - 12);
  }

  void getBaserels(std::vector<Baserel> *res) override {
    res->emplace_back(rva + 0, IMAGE_REL_BASED_ARM_MOV32T);
  }
```

- EN: Declares or implements routines including `writeTo`, `memcpy`, `applyMOV32T`, `applyBranch24T`, `getBaserels`, and 1 more. Notable symbols here include `writeTo`, `memcpy`, `applyMOV32T`, `applyBranch24T`, `getBaserels`, `emplace_back`.
- CN: 这里声明或实现函数，例如 `writeTo`, `memcpy`, `applyMOV32T`, `applyBranch24T`, `getBaserels`, and 1 more。这里较值得关注的符号包括 `writeTo`, `memcpy`, `applyMOV32T`, `applyBranch24T`, `getBaserels`, `emplace_back`。

### Lines 496-509

```cpp
  Defined *imp = nullptr;
  Chunk *tailMerge = nullptr;

private:
  const COFFLinkerContext &ctx;
};

class TailMergeChunkARM : public NonSectionCodeChunk {
public:
  TailMergeChunkARM(COFFLinkerContext &ctx, Chunk *d, Defined *h)
      : desc(d), helper(h), ctx(ctx) {
    setAlignment(2);
  }
```

- EN: Introduces type definitions such as `TailMergeChunkARM`. Declares or implements routines including `TailMergeChunkARM`, `desc`, `setAlignment`. Notable symbols here include `TailMergeChunkARM`, `desc`, `setAlignment`.
- CN: 这里引入类型定义，例如 `TailMergeChunkARM`。这里声明或实现函数，例如 `TailMergeChunkARM`, `desc`, `setAlignment`。这里较值得关注的符号包括 `TailMergeChunkARM`, `desc`, `setAlignment`。

### Lines 510-518

```cpp
  size_t getSize() const override { return sizeof(tailMergeARM); }
  MachineTypes getMachine() const override { return ARMNT; }

  void writeTo(uint8_t *buf) const override {
    memcpy(buf, tailMergeARM, sizeof(tailMergeARM));
    applyMOV32T(buf + 14, desc->getRVA() + ctx.config.imageBase);
    applyBranch24T(buf + 22, helper->getRVA() - rva - 26);
  }
```

- EN: Declares or implements routines including `getSize`, `getMachine`, `writeTo`, `memcpy`, `applyMOV32T`, and 1 more. Notable symbols here include `getSize`, `getMachine`, `writeTo`, `memcpy`, `applyMOV32T`, `applyBranch24T`.
- CN: 这里声明或实现函数，例如 `getSize`, `getMachine`, `writeTo`, `memcpy`, `applyMOV32T`, and 1 more。这里较值得关注的符号包括 `getSize`, `getMachine`, `writeTo`, `memcpy`, `applyMOV32T`, `applyBranch24T`。

### Lines 519-529

```cpp
  void getBaserels(std::vector<Baserel> *res) override {
    res->emplace_back(rva + 14, IMAGE_REL_BASED_ARM_MOV32T);
  }

  Chunk *desc = nullptr;
  Defined *helper = nullptr;

private:
  const COFFLinkerContext &ctx;
};
```

- EN: Declares or implements routines including `getBaserels`, `emplace_back`. Notable symbols here include `getBaserels`, `emplace_back`.
- CN: 这里声明或实现函数，例如 `getBaserels`, `emplace_back`。这里较值得关注的符号包括 `getBaserels`, `emplace_back`。

### Lines 530-538

```cpp
class ThunkChunkARM64 : public NonSectionCodeChunk {
public:
  ThunkChunkARM64(Defined *i, Chunk *tm) : imp(i), tailMerge(tm) {
    setAlignment(4);
  }

  size_t getSize() const override { return sizeof(thunkARM64); }
  MachineTypes getMachine() const override { return ARM64; }
```

- EN: Introduces type definitions such as `ThunkChunkARM64`. Declares or implements routines including `ThunkChunkARM64`, `setAlignment`, `getSize`, `getMachine`. Notable symbols here include `ThunkChunkARM64`, `setAlignment`, `getSize`, `getMachine`.
- CN: 这里引入类型定义，例如 `ThunkChunkARM64`。这里声明或实现函数，例如 `ThunkChunkARM64`, `setAlignment`, `getSize`, `getMachine`。这里较值得关注的符号包括 `ThunkChunkARM64`, `setAlignment`, `getSize`, `getMachine`。

### Lines 539-549

```cpp
  void writeTo(uint8_t *buf) const override {
    memcpy(buf, thunkARM64, sizeof(thunkARM64));
    applyArm64Addr(buf + 0, imp->getRVA(), rva + 0, 12);
    applyArm64Imm(buf + 4, imp->getRVA() & 0xfff, 0);
    applyArm64Branch26(buf + 8, tailMerge->getRVA() - rva - 8);
  }

  Defined *imp = nullptr;
  Chunk *tailMerge = nullptr;
};
```

- EN: Declares or implements routines including `writeTo`, `memcpy`, `applyArm64Addr`, `applyArm64Imm`, `applyArm64Branch26`. Notable symbols here include `writeTo`, `memcpy`, `applyArm64Addr`, `applyArm64Imm`, `applyArm64Branch26`.
- CN: 这里声明或实现函数，例如 `writeTo`, `memcpy`, `applyArm64Addr`, `applyArm64Imm`, `applyArm64Branch26`。这里较值得关注的符号包括 `writeTo`, `memcpy`, `applyArm64Addr`, `applyArm64Imm`, `applyArm64Branch26`。

### Lines 550-558

```cpp
class TailMergeChunkARM64 : public NonSectionCodeChunk {
public:
  TailMergeChunkARM64(Chunk *d, Defined *h) : desc(d), helper(h) {
    setAlignment(4);
  }

  size_t getSize() const override { return sizeof(tailMergeARM64); }
  MachineTypes getMachine() const override { return ARM64; }
```

- EN: Introduces type definitions such as `TailMergeChunkARM64`. Declares or implements routines including `TailMergeChunkARM64`, `setAlignment`, `getSize`, `getMachine`. Notable symbols here include `TailMergeChunkARM64`, `setAlignment`, `getSize`, `getMachine`.
- CN: 这里引入类型定义，例如 `TailMergeChunkARM64`。这里声明或实现函数，例如 `TailMergeChunkARM64`, `setAlignment`, `getSize`, `getMachine`。这里较值得关注的符号包括 `TailMergeChunkARM64`, `setAlignment`, `getSize`, `getMachine`。

### Lines 559-568

```cpp
  void writeTo(uint8_t *buf) const override {
    memcpy(buf, tailMergeARM64, sizeof(tailMergeARM64));
    applyArm64Addr(buf + 48, desc->getRVA(), rva + 48, 12);
    applyArm64Imm(buf + 52, desc->getRVA() & 0xfff, 0);
    if (helper) {
      applyArm64Addr(buf + 56, helper->getRVA(), rva + 56, 12);
      applyArm64Imm(buf + 60, helper->getRVA() & 0xfff, 0);
    }
  }
```

- EN: Declares or implements routines including `writeTo`, `memcpy`, `applyArm64Addr`, `applyArm64Imm`. Notable symbols here include `writeTo`, `memcpy`, `applyArm64Addr`, `applyArm64Imm`.
- CN: 这里声明或实现函数，例如 `writeTo`, `memcpy`, `applyArm64Addr`, `applyArm64Imm`。这里较值得关注的符号包括 `writeTo`, `memcpy`, `applyArm64Addr`, `applyArm64Imm`。

### Lines 569-581

```cpp
  Chunk *desc = nullptr;
  Defined *helper = nullptr;
};

// A chunk for the import descriptor table.
class DelayAddressChunk : public NonSectionChunk {
public:
  explicit DelayAddressChunk(COFFLinkerContext &ctx, Chunk *c)
      : thunk(c), ctx(ctx) {
    setAlignment(ctx.config.wordsize);
  }
  size_t getSize() const override { return ctx.config.wordsize; }
```

- EN: Introduces type definitions such as `DelayAddressChunk`. Declares or implements routines including `DelayAddressChunk`, `thunk`, `setAlignment`, `getSize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `DelayAddressChunk`, `thunk`, `setAlignment`, `getSize`.
- CN: 这里引入类型定义，例如 `DelayAddressChunk`。这里声明或实现函数，例如 `DelayAddressChunk`, `thunk`, `setAlignment`, `getSize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `DelayAddressChunk`, `thunk`, `setAlignment`, `getSize`。

### Lines 582-593

```cpp
  void writeTo(uint8_t *buf) const override {
    if (ctx.config.is64()) {
      write64le(buf, thunk->getRVA() + ctx.config.imageBase);
    } else {
      uint32_t bit = 0;
      // Pointer to thumb code must have the LSB set, so adjust it.
      if (ctx.config.machine == ARMNT)
        bit = 1;
      write32le(buf, (thunk->getRVA() + ctx.config.imageBase) | bit);
    }
  }
```

- EN: Declares or implements routines including `writeTo`, `write64le`, `write32le`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `writeTo`, `write64le`, `write32le`.
- CN: 这里声明或实现函数，例如 `writeTo`, `write64le`, `write32le`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `writeTo`, `write64le`, `write32le`。

### Lines 594-603

```cpp
  void getBaserels(std::vector<Baserel> *res) override {
    res->emplace_back(rva, ctx.config.machine);
  }

  Chunk *thunk;

private:
  const COFFLinkerContext &ctx;
};
```

- EN: Declares or implements routines including `getBaserels`, `emplace_back`. Notable symbols here include `getBaserels`, `emplace_back`.
- CN: 这里声明或实现函数，例如 `getBaserels`, `emplace_back`。这里较值得关注的符号包括 `getBaserels`, `emplace_back`。

### Lines 604-615

```cpp
// Export table
// Read Microsoft PE/COFF spec 5.3 for details.

// A chunk for the export descriptor table.
class ExportDirectoryChunk : public NonSectionChunk {
public:
  ExportDirectoryChunk(int baseOrdinal, int maxOrdinal, int nameTabSize,
                       Chunk *d, Chunk *a, Chunk *n, Chunk *o)
      : baseOrdinal(baseOrdinal), maxOrdinal(maxOrdinal),
        nameTabSize(nameTabSize), dllName(d), addressTab(a), nameTab(n),
        ordinalTab(o) {}
```

- EN: Introduces type definitions such as `ExportDirectoryChunk`. Declares or implements routines including `baseOrdinal`, `nameTabSize`, `ordinalTab`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `ExportDirectoryChunk`, `baseOrdinal`, `nameTabSize`, `ordinalTab`.
- CN: 这里引入类型定义，例如 `ExportDirectoryChunk`。这里声明或实现函数，例如 `baseOrdinal`, `nameTabSize`, `ordinalTab`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `ExportDirectoryChunk`, `baseOrdinal`, `nameTabSize`, `ordinalTab`。

### Lines 616-632

```cpp
  size_t getSize() const override {
    return sizeof(export_directory_table_entry);
  }

  void writeTo(uint8_t *buf) const override {
    memset(buf, 0, getSize());

    auto *e = (export_directory_table_entry *)(buf);
    e->NameRVA = dllName->getRVA();
    e->OrdinalBase = baseOrdinal;
    e->AddressTableEntries = (maxOrdinal - baseOrdinal) + 1;
    e->NumberOfNamePointers = nameTabSize;
    e->ExportAddressTableRVA = addressTab->getRVA();
    e->NamePointerRVA = nameTab->getRVA();
    e->OrdinalTableRVA = ordinalTab->getRVA();
  }
```

- EN: Declares or implements routines including `getSize`, `writeTo`, `memset`, `getRVA`. Notable symbols here include `getSize`, `writeTo`, `memset`, `getRVA`.
- CN: 这里声明或实现函数，例如 `getSize`, `writeTo`, `memset`, `getRVA`。这里较值得关注的符号包括 `getSize`, `writeTo`, `memset`, `getRVA`。

### Lines 633-641

```cpp
  uint16_t baseOrdinal;
  uint16_t maxOrdinal;
  uint16_t nameTabSize;
  Chunk *dllName;
  Chunk *addressTab;
  Chunk *nameTab;
  Chunk *ordinalTab;
};
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 642-649

```cpp
class AddressTableChunk : public NonSectionChunk {
public:
  explicit AddressTableChunk(SymbolTable &symtab, size_t baseOrdinal,
                             size_t maxOrdinal)
      : baseOrdinal(baseOrdinal), size((maxOrdinal - baseOrdinal) + 1),
        symtab(symtab) {}
  size_t getSize() const override { return size * 4; }
```

- EN: Introduces type definitions such as `AddressTableChunk`. Declares or implements routines including `baseOrdinal`, `symtab`, `getSize`. Notable symbols here include `AddressTableChunk`, `baseOrdinal`, `symtab`, `getSize`.
- CN: 这里引入类型定义，例如 `AddressTableChunk`。这里声明或实现函数，例如 `baseOrdinal`, `symtab`, `getSize`。这里较值得关注的符号包括 `AddressTableChunk`, `baseOrdinal`, `symtab`, `getSize`。

### Lines 650-667

```cpp
  void writeTo(uint8_t *buf) const override {
    memset(buf, 0, getSize());

    for (const Export &e : symtab.exports) {
      assert(e.ordinal >= baseOrdinal && "Export symbol has invalid ordinal");
      // Subtract the OrdinalBase to get the index.
      uint8_t *p = buf + (e.ordinal - baseOrdinal) * 4;
      uint32_t bit = 0;
      // Pointer to thumb code must have the LSB set, so adjust it.
      if (symtab.machine == ARMNT && !e.data)
        bit = 1;
      if (e.forwardChunk) {
        write32le(p, e.forwardChunk->getRVA() | bit);
      } else {
        assert(cast<Defined>(e.sym)->getRVA() != 0 &&
               "Exported symbol unmapped");
        write32le(p, cast<Defined>(e.sym)->getRVA() | bit);
      }
```

- EN: Declares or implements routines including `writeTo`, `memset`, `assert`, `write32le`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `writeTo`, `memset`, `assert`, `write32le`.
- CN: 这里声明或实现函数，例如 `writeTo`, `memset`, `assert`, `write32le`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `writeTo`, `memset`, `assert`, `write32le`。

### Lines 668-676

```cpp
    }
  }

private:
  size_t baseOrdinal;
  size_t size;
  const SymbolTable &symtab;
};
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 677-688

```cpp
class NamePointersChunk : public NonSectionChunk {
public:
  explicit NamePointersChunk(std::vector<Chunk *> &v) : chunks(v) {}
  size_t getSize() const override { return chunks.size() * 4; }

  void writeTo(uint8_t *buf) const override {
    for (Chunk *c : chunks) {
      write32le(buf, c->getRVA());
      buf += 4;
    }
  }
```

- EN: Introduces type definitions such as `NamePointersChunk`. Declares or implements routines including `NamePointersChunk`, `getSize`, `writeTo`, `write32le`. Notable symbols here include `NamePointersChunk`, `getSize`, `writeTo`, `write32le`.
- CN: 这里引入类型定义，例如 `NamePointersChunk`。这里声明或实现函数，例如 `NamePointersChunk`, `getSize`, `writeTo`, `write32le`。这里较值得关注的符号包括 `NamePointersChunk`, `getSize`, `writeTo`, `write32le`。

### Lines 689-699

```cpp
private:
  std::vector<Chunk *> chunks;
};

class ExportOrdinalChunk : public NonSectionChunk {
public:
  explicit ExportOrdinalChunk(const SymbolTable &symtab, size_t baseOrdinal,
                              size_t tableSize)
      : baseOrdinal(baseOrdinal), size(tableSize), symtab(symtab) {}
  size_t getSize() const override { return size * 2; }
```

- EN: Introduces type definitions such as `ExportOrdinalChunk`. Declares or implements routines including `baseOrdinal`, `getSize`. Notable symbols here include `ExportOrdinalChunk`, `baseOrdinal`, `getSize`.
- CN: 这里引入类型定义，例如 `ExportOrdinalChunk`。这里声明或实现函数，例如 `baseOrdinal`, `getSize`。这里较值得关注的符号包括 `ExportOrdinalChunk`, `baseOrdinal`, `getSize`。

### Lines 700-710

```cpp
  void writeTo(uint8_t *buf) const override {
    for (const Export &e : symtab.exports) {
      if (e.noname)
        continue;
      assert(e.ordinal >= baseOrdinal && "Export symbol has invalid ordinal");
      // This table stores unbiased indices, so subtract OrdinalBase.
      write16le(buf, e.ordinal - baseOrdinal);
      buf += 2;
    }
  }
```

- EN: Declares or implements routines including `writeTo`, `assert`, `write16le`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `writeTo`, `assert`, `write16le`.
- CN: 这里声明或实现函数，例如 `writeTo`, `assert`, `write16le`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `writeTo`, `assert`, `write16le`。

### Lines 711-718

```cpp
private:
  size_t baseOrdinal;
  size_t size;
  const SymbolTable &symtab;
};

} // anonymous namespace
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 719-736

```cpp
void IdataContents::create(COFFLinkerContext &ctx) {
  std::vector<std::vector<DefinedImportData *>> v = binImports(ctx, imports);

  // In hybrid images, EC and native code are usually very similar,
  // resulting in a highly similar set of imported symbols. Consequently,
  // their import tables can be shared, with ARM64X relocations handling any
  // differences. Identify matching import files used by EC and native code, and
  // merge them into a single hybrid import entry.
  if (ctx.hybridSymtab) {
    for (std::vector<DefinedImportData *> &syms : v) {
      std::vector<DefinedImportData *> hybridSyms;
      ImportFile *prev = nullptr;
      for (DefinedImportData *sym : syms) {
        ImportFile *file = sym->file;
        // At this stage, symbols are sorted by base name, ensuring that
        // compatible import files, if present, are adjacent. Check if the
        // current symbol's file imports the same symbol as the previously added
        // one (if any and if it was not already merged). Additionally, verify
```

- EN: Declares or implements routines including `create`, `binImports`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `create`, `binImports`.
- CN: 这里声明或实现函数，例如 `create`, `binImports`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `create`, `binImports`。

### Lines 737-749

```cpp
        // that one of them is native while the other is EC. In rare cases,
        // separate matching import entries may exist within the same namespace,
        // which cannot be merged.
        if (!prev || file->isEC() == prev->isEC() ||
            !file->isSameImport(prev)) {
          // We can't merge the import file, just add it to hybridSyms
          // and set prev to its file so that we can try to match the next
          // symbol.
          hybridSyms.push_back(sym);
          prev = file;
          continue;
        }
```

- EN: Declares or implements routines including `isSameImport`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isSameImport`.
- CN: 这里声明或实现函数，例如 `isSameImport`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isSameImport`。

### Lines 750-759

```cpp
        // A matching symbol may appear in syms in any order. The native variant
        // exposes a subset of EC symbols and chunks, so always use the EC
        // variant as the hybrid import file. If the native file was already
        // added, replace it with the EC symbol in hybridSyms. Otherwise, the EC
        // variant is already pushed, so we can simply merge it.
        if (file->isEC()) {
          hybridSyms.pop_back();
          hybridSyms.push_back(sym);
        }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 760-777

```cpp
        // Merge import files by storing their hybrid form in the corresponding
        // file class.
        prev->hybridFile = file;
        file->hybridFile = prev;
        prev = nullptr; // A hybrid import file cannot be merged again.
      }

      // Sort symbols by type: native-only files first, followed by merged
      // hybrid files, and then EC-only files.
      llvm::stable_sort(hybridSyms,
                        [](DefinedImportData *a, DefinedImportData *b) {
                          if (a->file->hybridFile)
                            return !b->file->hybridFile && b->file->isEC();
                          return !a->file->isEC() && b->file->isEC();
                        });
      syms = std::move(hybridSyms);
    }
  }
```

- EN: Declares or implements routines including `move`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `move`.
- CN: 这里声明或实现函数，例如 `move`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `move`。

### Lines 778-792

```cpp

  // Create .idata contents for each DLL.
  for (std::vector<DefinedImportData *> &syms : v) {
    // Create lookup and address tables. If they have external names,
    // we need to create hintName chunks to store the names.
    // If they don't (if they are import-by-ordinals), we store only
    // ordinal values to the table.
    size_t base = lookups.size();
    Chunk *lookupsTerminator = nullptr, *addressesTerminator = nullptr;
    uint32_t nativeOnly = 0;
    for (DefinedImportData *s : syms) {
      uint16_t ord = s->getOrdinal();
      HintNameChunk *hintChunk = nullptr;
      Chunk *lookupsChunk, *addressesChunk;
```

- EN: Declares or implements routines including `getOrdinal`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getOrdinal`.
- CN: 这里声明或实现函数，例如 `getOrdinal`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getOrdinal`。

### Lines 793-802

```cpp
      if (s->getExternalName().empty()) {
        lookupsChunk = make<OrdinalOnlyChunk>(ctx, ord);
        addressesChunk = make<OrdinalOnlyChunk>(ctx, ord);
      } else {
        hintChunk = make<HintNameChunk>(s->getExternalName(), ord);
        lookupsChunk = make<LookupChunk>(ctx, hintChunk);
        addressesChunk = make<LookupChunk>(ctx, hintChunk);
        hints.push_back(hintChunk);
      }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 803-819

```cpp
      // Detect the first EC-only import in the hybrid IAT. Emit null chunk
      // as a terminator for the native view, and add an ARM64X relocation to
      // replace it with the correct import for the EC view.
      //
      // Additionally, for MSVC compatibility, store the lookup and address
      // chunks and append them at the end of EC-only imports, where a null
      // terminator chunk would typically be placed. Since they appear after
      // the native terminator, they will be ignored in the native view.
      // In the EC view, they should act as terminators, so emit ZEROFILL
      // relocations overriding them.
      if (ctx.config.machine == ARM64X && !lookupsTerminator &&
          s->file->isEC() && !s->file->hybridFile) {
        lookupsTerminator = lookupsChunk;
        addressesTerminator = addressesChunk;
        lookupsChunk = make<NullChunk>(ctx);
        addressesChunk = make<NullChunk>(ctx);
```

- EN: Declares or implements routines including `isEC`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isEC`.
- CN: 这里声明或实现函数，例如 `isEC`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isEC`。

### Lines 820-832

```cpp
        Arm64XRelocVal relocVal = hintChunk;
        if (!hintChunk)
          relocVal = (1ULL << 63) | ord;
        ctx.dynamicRelocs->add(IMAGE_DVRT_ARM64X_FIXUP_TYPE_VALUE,
                               sizeof(uint64_t), lookupsChunk, relocVal);
        ctx.dynamicRelocs->add(IMAGE_DVRT_ARM64X_FIXUP_TYPE_VALUE,
                               sizeof(uint64_t), addressesChunk, relocVal);
        ctx.dynamicRelocs->add(IMAGE_DVRT_ARM64X_FIXUP_TYPE_ZEROFILL,
                               sizeof(uint64_t), lookupsTerminator);
        ctx.dynamicRelocs->add(IMAGE_DVRT_ARM64X_FIXUP_TYPE_ZEROFILL,
                               sizeof(uint64_t), addressesTerminator);
      }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 833-840

```cpp
      lookups.push_back(lookupsChunk);
      addresses.push_back(addressesChunk);

      if (s->file->isEC()) {
        auto chunk = make<AuxImportChunk>(s->file);
        auxIat.push_back(chunk);
        s->file->impECSym->setLocation(chunk);
```

- EN: Declares or implements routines including `setLocation`. Notable symbols here include `setLocation`.
- CN: 这里声明或实现函数，例如 `setLocation`。这里较值得关注的符号包括 `setLocation`。

### Lines 841-858

```cpp
        chunk = make<AuxImportChunk>(s->file);
        auxIatCopy.push_back(chunk);
        s->file->auxImpCopySym->setLocation(chunk);
      } else if (ctx.hybridSymtab) {
        // Fill the auxiliary IAT with null chunks for native-only imports.
        auxIat.push_back(make<NullChunk>(ctx));
        auxIatCopy.push_back(make<NullChunk>(ctx));
        ++nativeOnly;
      }
    }
    // Terminate with null values.
    lookups.push_back(lookupsTerminator ? lookupsTerminator
                                        : make<NullChunk>(ctx));
    addresses.push_back(addressesTerminator ? addressesTerminator
                                            : make<NullChunk>(ctx));
    if (ctx.symtab.isEC()) {
      auxIat.push_back(make<NullChunk>(ctx));
      auxIatCopy.push_back(make<NullChunk>(ctx));
```

- EN: Declares or implements routines including `setLocation`, `if`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `setLocation`, `if`.
- CN: 这里声明或实现函数，例如 `setLocation`, `if`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `setLocation`, `if`。

### Lines 859-866

```cpp
    }

    for (int i = 0, e = syms.size(); i < e; ++i) {
      syms[i]->setLocation(addresses[base + i]);
      if (syms[i]->file->hybridFile)
        syms[i]->file->hybridFile->impSym->setLocation(addresses[base + i]);
    }
```

- EN: Declares or implements routines including `setLocation`. Notable symbols here include `setLocation`.
- CN: 这里声明或实现函数，例如 `setLocation`。这里较值得关注的符号包括 `setLocation`。

### Lines 867-884

```cpp
    // Create the import table header.
    dllNames.push_back(make<StringChunk>(syms[0]->getDLLName()));
    auto *dir = make<ImportDirectoryChunk>(dllNames.back());

    if (ctx.hybridSymtab && nativeOnly) {
      if (ctx.config.machine != ARM64X)
        // On pure ARM64EC targets, skip native-only imports in the import
        // directory.
        base += nativeOnly;
      else if (nativeOnly) {
        // If native-only imports exist, they will appear as a prefix to all
        // imports. Emit ARM64X relocations to skip them in the EC view.
        ctx.dynamicRelocs->add(
            IMAGE_DVRT_ARM64X_FIXUP_TYPE_DELTA, 0,
            Arm64XRelocVal(
                dir, offsetof(ImportDirectoryTableEntry, ImportLookupTableRVA)),
            nativeOnly * sizeof(uint64_t));
        ctx.dynamicRelocs->add(
```

- EN: Declares or implements routines including `offsetof`, `sizeof`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `offsetof`, `sizeof`.
- CN: 这里声明或实现函数，例如 `offsetof`, `sizeof`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `offsetof`, `sizeof`。

### Lines 885-899

```cpp
            IMAGE_DVRT_ARM64X_FIXUP_TYPE_DELTA, 0,
            Arm64XRelocVal(dir, offsetof(ImportDirectoryTableEntry,
                                         ImportAddressTableRVA)),
            nativeOnly * sizeof(uint64_t));
      }
    }

    dir->lookupTab = lookups[base];
    dir->addressTab = addresses[base];
    dirs.push_back(dir);
  }
  // Add null terminator.
  dirs.push_back(make<NullChunk>(sizeof(ImportDirectoryTableEntry), 4));
}
```

- EN: Declares or implements routines including `sizeof`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `sizeof`.
- CN: 这里声明或实现函数，例如 `sizeof`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `sizeof`。

### Lines 900-908

```cpp
std::vector<Chunk *> DelayLoadContents::getChunks() {
  std::vector<Chunk *> v;
  v.insert(v.end(), dirs.begin(), dirs.end());
  v.insert(v.end(), names.begin(), names.end());
  v.insert(v.end(), hintNames.begin(), hintNames.end());
  v.insert(v.end(), dllNames.begin(), dllNames.end());
  return v;
}
```

- EN: Declares or implements routines including `getChunks`. Notable symbols here include `getChunks`.
- CN: 这里声明或实现函数，例如 `getChunks`。这里较值得关注的符号包括 `getChunks`。

### Lines 909-919

```cpp
std::vector<Chunk *> DelayLoadContents::getDataChunks() {
  std::vector<Chunk *> v;
  v.insert(v.end(), moduleHandles.begin(), moduleHandles.end());
  v.insert(v.end(), addresses.begin(), addresses.end());
  return v;
}

uint64_t DelayLoadContents::getDirSize() {
  return dirs.size() * sizeof(delay_import_directory_table_entry);
}
```

- EN: Declares or implements routines including `getDataChunks`, `getDirSize`. Notable symbols here include `getDataChunks`, `getDirSize`.
- CN: 这里声明或实现函数，例如 `getDataChunks`, `getDirSize`。这里较值得关注的符号包括 `getDataChunks`, `getDirSize`。

### Lines 920-928

```cpp
void DelayLoadContents::create() {
  std::vector<std::vector<DefinedImportData *>> v = binImports(ctx, imports);

  // Create .didat contents for each DLL.
  for (std::vector<DefinedImportData *> &syms : v) {
    // Create the delay import table header.
    dllNames.push_back(make<StringChunk>(syms[0]->getDLLName()));
    auto *dir = make<DelayDirectoryChunk>(dllNames.back());
```

- EN: Declares or implements routines including `create`, `binImports`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `create`, `binImports`.
- CN: 这里声明或实现函数，例如 `create`, `binImports`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `create`, `binImports`。

### Lines 929-946

```cpp
    size_t base = addresses.size();
    ctx.forEachSymtab([&](SymbolTable &symtab) {
      if (symtab.isEC()) {
        if (ctx.config.machine == ARM64X) {
          // For hybrid images, emit null-terminated native import entries
          // followed by null-terminated EC entries. If a view is missing
          // imports for a given module, only terminators are emitted. Emit
          // ARM64X relocations to skip native entries in the EC view.
          ctx.dynamicRelocs->add(
              IMAGE_DVRT_ARM64X_FIXUP_TYPE_DELTA, 0,
              Arm64XRelocVal(dir, offsetof(delay_import_directory_table_entry,
                                           DelayImportAddressTable)),
              (addresses.size() - base) * sizeof(uint64_t));
          ctx.dynamicRelocs->add(
              IMAGE_DVRT_ARM64X_FIXUP_TYPE_DELTA, 0,
              Arm64XRelocVal(dir, offsetof(delay_import_directory_table_entry,
                                           DelayImportNameTable)),
              (addresses.size() - base) * sizeof(uint64_t));
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 947-958

```cpp
        } else {
          base = addresses.size();
        }
      }

      Chunk *tm = nullptr;

      for (DefinedImportData *s : syms) {
        // Process only the symbols belonging to the current symtab.
        if (symtab.isEC() != s->file->isEC())
          continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 959-976

```cpp
        if (!tm) {
          tm = newTailMergeChunk(symtab, dir);
          Chunk *pdataChunk = newTailMergePDataChunk(symtab, tm);
          if (pdataChunk)
            pdata.push_back(pdataChunk);
        }

        Chunk *t = newThunkChunk(s, tm);
        auto *a = make<DelayAddressChunk>(ctx, t);
        addresses.push_back(a);
        s->setLocation(a);
        thunks.push_back(t);
        StringRef extName = s->getExternalName();
        if (extName.empty()) {
          names.push_back(make<OrdinalOnlyChunk>(ctx, s->getOrdinal()));
        } else {
          auto *c = make<HintNameChunk>(extName, 0);
          names.push_back(make<LookupChunk>(ctx, c));
```

- EN: Declares or implements routines including `newTailMergeChunk`, `newTailMergePDataChunk`, `newThunkChunk`, `setLocation`, `getExternalName`. Notable symbols here include `newTailMergeChunk`, `newTailMergePDataChunk`, `newThunkChunk`, `setLocation`, `getExternalName`.
- CN: 这里声明或实现函数，例如 `newTailMergeChunk`, `newTailMergePDataChunk`, `newThunkChunk`, `setLocation`, `getExternalName`。这里较值得关注的符号包括 `newTailMergeChunk`, `newTailMergePDataChunk`, `newThunkChunk`, `setLocation`, `getExternalName`。

### Lines 977-985

```cpp
          hintNames.push_back(c);
          // Add a synthetic symbol for this load thunk, using the
          // "__imp___load" prefix, in case this thunk needs to be added to the
          // list of valid call targets for Control Flow Guard.
          StringRef symName = saver().save("__imp___load_" + extName);
          s->loadThunkSym =
              cast<DefinedSynthetic>(symtab.addSynthetic(symName, t));
        }
```

- EN: Declares or implements routines including `saver`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `saver`.
- CN: 这里声明或实现函数，例如 `saver`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `saver`。

### Lines 986-1000

```cpp
        if (symtab.isEC()) {
          auto chunk = make<AuxImportChunk>(s->file);
          auxIat.push_back(chunk);
          s->file->impECSym->setLocation(chunk);

          chunk = make<AuxImportChunk>(s->file);
          auxIatCopy.push_back(chunk);
          s->file->auxImpCopySym->setLocation(chunk);
        } else if (ctx.config.machine == ARM64X) {
          // Fill the auxiliary IAT with null chunks for native imports.
          auxIat.push_back(make<NullChunk>(ctx));
          auxIatCopy.push_back(make<NullChunk>(ctx));
        }
      }
```

- EN: Declares or implements routines including `setLocation`, `if`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `setLocation`, `if`.
- CN: 这里声明或实现函数，例如 `setLocation`, `if`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `setLocation`, `if`。

### Lines 1001-1011

```cpp
      if (tm) {
        thunks.push_back(tm);
        StringRef tmName =
            saver().save("__tailMerge_" + syms[0]->getDLLName().lower());
        symtab.addSynthetic(tmName, tm);
      }

      // Skip terminators on pure ARM64EC target if there are no native imports.
      if (!tm && !symtab.isEC() && ctx.config.machine != ARM64X)
        return;
```

- EN: Declares or implements routines including `saver`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `saver`.
- CN: 这里声明或实现函数，例如 `saver`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `saver`。

### Lines 1012-1020

```cpp
      // Terminate with null values.
      addresses.push_back(make<NullChunk>(ctx, 8));
      names.push_back(make<NullChunk>(ctx, 8));
      if (ctx.symtab.isEC()) {
        auxIat.push_back(make<NullChunk>(ctx, 8));
        auxIatCopy.push_back(make<NullChunk>(ctx, 8));
      }
    });
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1021-1030

```cpp
    auto *mh = make<NullChunk>(8, 8);
    moduleHandles.push_back(mh);

    // Fill the delay import table header fields.
    dir->moduleHandle = mh;
    dir->addressTab = addresses[base];
    dir->nameTab = names[base];
    dirs.push_back(dir);
  }
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 1031-1039

```cpp
  ctx.forEachSymtab([&](SymbolTable &symtab) {
    if (symtab.tailMergeUnwindInfoChunk)
      unwindinfo.push_back(symtab.tailMergeUnwindInfoChunk);
  });
  // Add null terminator.
  dirs.push_back(
      make<NullChunk>(sizeof(delay_import_directory_table_entry), 4));
}
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1040-1056

```cpp
Chunk *DelayLoadContents::newTailMergeChunk(SymbolTable &symtab, Chunk *dir) {
  auto helper = cast_or_null<Defined>(symtab.delayLoadHelper);
  switch (symtab.machine) {
  case AMD64:
  case ARM64EC:
    return make<TailMergeChunkX64>(dir, helper);
  case I386:
    return make<TailMergeChunkX86>(ctx, dir, helper);
  case ARMNT:
    return make<TailMergeChunkARM>(ctx, dir, helper);
  case ARM64:
    return make<TailMergeChunkARM64>(dir, helper);
  default:
    llvm_unreachable("unsupported machine type");
  }
}
```

- EN: Declares or implements routines including `newTailMergeChunk`, `llvm_unreachable`. Notable symbols here include `newTailMergeChunk`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `newTailMergeChunk`, `llvm_unreachable`。这里较值得关注的符号包括 `newTailMergeChunk`, `llvm_unreachable`。

### Lines 1057-1070

```cpp
Chunk *DelayLoadContents::newTailMergePDataChunk(SymbolTable &symtab,
                                                 Chunk *tm) {
  switch (symtab.machine) {
  case AMD64:
  case ARM64EC:
    if (!symtab.tailMergeUnwindInfoChunk)
      symtab.tailMergeUnwindInfoChunk = make<TailMergeUnwindInfoX64>();
    return make<TailMergePDataChunkX64>(tm, symtab.tailMergeUnwindInfoChunk);
    // FIXME: Add support for other architectures.
  default:
    return nullptr; // Just don't generate unwind info.
  }
}
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1071-1087

```cpp
Chunk *DelayLoadContents::newThunkChunk(DefinedImportData *s,
                                        Chunk *tailMerge) {
  switch (s->file->getMachineType()) {
  case AMD64:
  case ARM64EC:
    return make<ThunkChunkX64>(s, tailMerge);
  case I386:
    return make<ThunkChunkX86>(ctx, s, tailMerge);
  case ARMNT:
    return make<ThunkChunkARM>(ctx, s, tailMerge);
  case ARM64:
    return make<ThunkChunkARM64>(s, tailMerge);
  default:
    llvm_unreachable("unsupported machine type");
  }
}
```

- EN: Declares or implements routines including `llvm_unreachable`. Notable symbols here include `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`。这里较值得关注的符号包括 `llvm_unreachable`。

### Lines 1088-1097

```cpp
void createEdataChunks(SymbolTable &symtab, std::vector<Chunk *> &chunks) {
  unsigned baseOrdinal = 1 << 16, maxOrdinal = 0;
  for (Export &e : symtab.exports) {
    baseOrdinal = std::min(baseOrdinal, (unsigned)e.ordinal);
    maxOrdinal = std::max(maxOrdinal, (unsigned)e.ordinal);
  }
  // Ordinals must start at 1 as suggested in:
  // https://learn.microsoft.com/en-us/cpp/build/reference/export-exports-a-function?view=msvc-170
  assert(baseOrdinal >= 1);
```

- EN: Declares or implements routines including `createEdataChunks`, `min`, `max`, `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `createEdataChunks`, `min`, `max`, `assert`.
- CN: 这里声明或实现函数，例如 `createEdataChunks`, `min`, `max`, `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `createEdataChunks`, `min`, `max`, `assert`。

### Lines 1098-1105

```cpp
  auto *dllName =
      make<StringChunk>(sys::path::filename(symtab.ctx.config.outputFile));
  auto *addressTab = make<AddressTableChunk>(symtab, baseOrdinal, maxOrdinal);
  std::vector<Chunk *> names;
  for (Export &e : symtab.exports)
    if (!e.noname)
      names.push_back(make<StringChunk>(e.exportName));
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1106-1113

```cpp
  std::vector<Chunk *> forwards;
  for (Export &e : symtab.exports) {
    if (e.forwardTo.empty())
      continue;
    e.forwardChunk = make<StringChunk>(e.forwardTo);
    forwards.push_back(e.forwardChunk);
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1114-1128

```cpp
  auto *nameTab = make<NamePointersChunk>(names);
  auto *ordinalTab =
      make<ExportOrdinalChunk>(symtab, baseOrdinal, names.size());
  auto *dir =
      make<ExportDirectoryChunk>(baseOrdinal, maxOrdinal, names.size(), dllName,
                                 addressTab, nameTab, ordinalTab);
  chunks.push_back(dir);
  chunks.push_back(dllName);
  chunks.push_back(addressTab);
  chunks.push_back(nameTab);
  chunks.push_back(ordinalTab);
  chunks.insert(chunks.end(), names.begin(), names.end());
  chunks.insert(chunks.end(), forwards.begin(), forwards.end());
}
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 1129-1129

```cpp
} // namespace lld::coff
```

- EN: Works inside namespace scope `lld` to organize symbols. Notable symbols here include `lld`.
- CN: 这里位于命名空间 `lld` 中，用于组织符号作用域。这里较值得关注的符号包括 `lld`。

## Key Concepts / 关键概念

- `HintNameChunk`: class or struct interface / 类或结构体接口
- `LookupChunk`: class or struct interface / 类或结构体接口
- `OrdinalOnlyChunk`: class or struct interface / 类或结构体接口
- `ImportDirectoryChunk`: class or struct interface / 类或结构体接口
- `HintNameChunk`: function or method entry point / 函数或方法入口
- `getSize`: function or method entry point / 函数或方法入口
- `writeTo`: function or method entry point / 函数或方法入口
- `memset`: function or method entry point / 函数或方法入口

## Dependencies / 依赖关系

- LLVM headers / LLVM 头文件: `llvm/ADT/STLExtras.h`, `llvm/Object/COFF.h`, `llvm/Support/Endian.h`, `llvm/Support/Path.h`
- System headers / 系统头文件: `DLL.h`, `COFFLinkerContext.h`, `Chunks.h`, `SymbolTable.h`
- Directory context / 目录上下文: `lld/COFF` neighbors usually cooperate with this file to provide the surrounding subsystem / `lld/COFF` 下的相邻文件通常与本文件协作组成对应子系统
