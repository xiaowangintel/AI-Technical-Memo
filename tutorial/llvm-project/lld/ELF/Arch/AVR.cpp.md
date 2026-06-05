# AVR.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `lld/ELF/Arch/AVR.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: AVR.cpp AVR is a Harvard-architecture 8-bit microcontroller designed for small baremetal programs. All AVR-family processors have 32 8-bit registers. The tiniest AVR has 32 byte RAM and 1 KiB program memory, and the largest. It also sits in code that implements ELF linking components in LLD. / 该文件实现 LLD 中的 ELF 链接组件。 源码头部说明其职责是：AVR.cpp AVR is a Harvard-architecture 8-bit microcontroller designed for small baremetal programs. All AVR-family processors have 32 8-bit registers. The tiniest AVR has 32 byte RAM and 1 KiB program memory, and the largest。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```cpp
//===- AVR.cpp ------------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// AVR is a Harvard-architecture 8-bit microcontroller designed for small
// baremetal programs. All AVR-family processors have 32 8-bit registers.
// The tiniest AVR has 32 byte RAM and 1 KiB program memory, and the largest
// one supports up to 2^24 data address space and 2^22 code address space.
//
// Since it is a baremetal programming, there's usually no loader to load
// ELF files on AVRs. You are expected to link your program against address
// 0 and pull out a .text section from the result using objcopy, so that you
// can write the linked code to on-chip flush memory. You can do that with
// the following commands:
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 19-27

```cpp
//
//   ld.lld -Ttext=0 -o foo foo.o
//   objcopy -O binary --only-section=.text foo output.bin
//
// Note that the current AVR support is very preliminary so you can't
// link any useful program yet, though.
//
//===----------------------------------------------------------------------===//
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 28-41

```cpp
#include "InputFiles.h"
#include "Symbols.h"
#include "Target.h"
#include "Thunks.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/Support/Endian.h"

using namespace llvm;
using namespace llvm::object;
using namespace llvm::support::endian;
using namespace llvm::ELF;
using namespace lld;
using namespace lld::elf;
```

- EN: Pulls in 6 header(s) from LLVM, system dependencies needed by this range. Works inside namespace scope `llvm`, `lld` to organize symbols. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `llvm`, `lld`.
- CN: 这里引入 6 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里位于命名空间 `llvm`, `lld` 中，用于组织符号作用域。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `llvm`, `lld`。

### Lines 42-56

```cpp
namespace {
class AVR final : public TargetInfo {
public:
  AVR(Ctx &ctx) : TargetInfo(ctx) { needsThunks = true; }
  uint32_t calcEFlags() const override;
  RelExpr getRelExpr(RelType type, const Symbol &s,
                     const uint8_t *loc) const override;
  bool needsThunk(RelExpr expr, RelType type, const InputFile *file,
                  uint64_t branchAddr, const Symbol &s,
                  int64_t a) const override;
  void relocate(uint8_t *loc, const Relocation &rel,
                uint64_t val) const override;
};
} // namespace
```

- EN: Introduces type definitions such as `AVR`. Declares or implements routines including `AVR`, `calcEFlags`. Notable symbols here include `AVR`, `calcEFlags`.
- CN: 这里引入类型定义，例如 `AVR`。这里声明或实现函数，例如 `AVR`, `calcEFlags`。这里较值得关注的符号包括 `AVR`, `calcEFlags`。

### Lines 57-74

```cpp
RelExpr AVR::getRelExpr(RelType type, const Symbol &s,
                        const uint8_t *loc) const {
  switch (type) {
  case R_AVR_6:
  case R_AVR_6_ADIW:
  case R_AVR_8:
  case R_AVR_8_LO8:
  case R_AVR_8_HI8:
  case R_AVR_8_HLO8:
  case R_AVR_16:
  case R_AVR_16_PM:
  case R_AVR_32:
  case R_AVR_LDI:
  case R_AVR_LO8_LDI:
  case R_AVR_LO8_LDI_NEG:
  case R_AVR_HI8_LDI:
  case R_AVR_HI8_LDI_NEG:
  case R_AVR_HH8_LDI_NEG:
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 75-92

```cpp
  case R_AVR_HH8_LDI:
  case R_AVR_MS8_LDI_NEG:
  case R_AVR_MS8_LDI:
  case R_AVR_LO8_LDI_GS:
  case R_AVR_LO8_LDI_PM:
  case R_AVR_LO8_LDI_PM_NEG:
  case R_AVR_HI8_LDI_GS:
  case R_AVR_HI8_LDI_PM:
  case R_AVR_HI8_LDI_PM_NEG:
  case R_AVR_HH8_LDI_PM:
  case R_AVR_HH8_LDI_PM_NEG:
  case R_AVR_LDS_STS_16:
  case R_AVR_PORT5:
  case R_AVR_PORT6:
  case R_AVR_CALL:
    return R_ABS;
  case R_AVR_7_PCREL:
  case R_AVR_13_PCREL:
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 93-100

```cpp
    return R_PC;
  default:
    Err(ctx) << getErrorLoc(ctx, loc) << "unknown relocation (" << type.v
             << ") against symbol " << &s;
    return R_NONE;
  }
}
```

- EN: Declares or implements routines including `Err`. Notable symbols here include `Err`.
- CN: 这里声明或实现函数，例如 `Err`。这里较值得关注的符号包括 `Err`。

### Lines 101-117

```cpp
static void writeLDI(uint8_t *loc, uint64_t val) {
  write16le(loc, (read16le(loc) & 0xf0f0) | (val & 0xf0) << 4 | (val & 0x0f));
}

bool AVR::needsThunk(RelExpr expr, RelType type, const InputFile *file,
                     uint64_t branchAddr, const Symbol &s, int64_t a) const {
  switch (type) {
  case R_AVR_LO8_LDI_GS:
  case R_AVR_HI8_LDI_GS:
    // A thunk is needed if the symbol's virtual address is out of range
    // [0, 0x1ffff].
    return s.getVA(ctx) >= 0x20000;
  default:
    return false;
  }
}
```

- EN: Declares or implements routines including `writeLDI`, `write16le`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `writeLDI`, `write16le`.
- CN: 这里声明或实现函数，例如 `writeLDI`, `write16le`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `writeLDI`, `write16le`。

### Lines 118-135

```cpp
void AVR::relocate(uint8_t *loc, const Relocation &rel, uint64_t val) const {
  switch (rel.type) {
  case R_AVR_8:
    checkUInt(ctx, loc, val, 8, rel);
    *loc = val;
    break;
  case R_AVR_8_LO8:
    checkUInt(ctx, loc, val, 32, rel);
    *loc = val & 0xff;
    break;
  case R_AVR_8_HI8:
    checkUInt(ctx, loc, val, 32, rel);
    *loc = (val >> 8) & 0xff;
    break;
  case R_AVR_8_HLO8:
    checkUInt(ctx, loc, val, 32, rel);
    *loc = (val >> 16) & 0xff;
    break;
```

- EN: Declares or implements routines including `relocate`, `checkUInt`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `relocate`, `checkUInt`.
- CN: 这里声明或实现函数，例如 `relocate`, `checkUInt`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `relocate`, `checkUInt`。

### Lines 136-151

```cpp
  case R_AVR_16:
    // Note: this relocation is often used between code and data space, which
    // are 0x800000 apart in the output ELF file. The bitmask cuts off the high
    // bit.
    write16le(loc, val & 0xffff);
    break;
  case R_AVR_16_PM:
    checkAlignment(ctx, loc, val, 2, rel);
    checkUInt(ctx, loc, val >> 1, 16, rel);
    write16le(loc, val >> 1);
    break;
  case R_AVR_32:
    checkUInt(ctx, loc, val, 32, rel);
    write32le(loc, val);
    break;
```

- EN: Declares or implements routines including `write16le`, `checkAlignment`, `checkUInt`, `write32le`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `write16le`, `checkAlignment`, `checkUInt`, `write32le`.
- CN: 这里声明或实现函数，例如 `write16le`, `checkAlignment`, `checkUInt`, `write32le`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `write16le`, `checkAlignment`, `checkUInt`, `write32le`。

### Lines 152-169

```cpp
  case R_AVR_LDI:
    checkUInt(ctx, loc, val, 8, rel);
    writeLDI(loc, val & 0xff);
    break;

  case R_AVR_LO8_LDI_NEG:
    writeLDI(loc, -val & 0xff);
    break;
  case R_AVR_LO8_LDI:
    writeLDI(loc, val & 0xff);
    break;
  case R_AVR_HI8_LDI_NEG:
    writeLDI(loc, (-val >> 8) & 0xff);
    break;
  case R_AVR_HI8_LDI:
    writeLDI(loc, (val >> 8) & 0xff);
    break;
  case R_AVR_HH8_LDI_NEG:
```

- EN: Declares or implements routines including `checkUInt`, `writeLDI`. Notable symbols here include `checkUInt`, `writeLDI`.
- CN: 这里声明或实现函数，例如 `checkUInt`, `writeLDI`。这里较值得关注的符号包括 `checkUInt`, `writeLDI`。

### Lines 170-181

```cpp
    writeLDI(loc, (-val >> 16) & 0xff);
    break;
  case R_AVR_HH8_LDI:
    writeLDI(loc, (val >> 16) & 0xff);
    break;
  case R_AVR_MS8_LDI_NEG:
    writeLDI(loc, (-val >> 24) & 0xff);
    break;
  case R_AVR_MS8_LDI:
    writeLDI(loc, (val >> 24) & 0xff);
    break;
```

- EN: Declares or implements routines including `writeLDI`. Notable symbols here include `writeLDI`.
- CN: 这里声明或实现函数，例如 `writeLDI`。这里较值得关注的符号包括 `writeLDI`。

### Lines 182-199

```cpp
  case R_AVR_LO8_LDI_GS:
    checkUInt(ctx, loc, val, 17, rel);
    [[fallthrough]];
  case R_AVR_LO8_LDI_PM:
    checkAlignment(ctx, loc, val, 2, rel);
    writeLDI(loc, (val >> 1) & 0xff);
    break;
  case R_AVR_HI8_LDI_GS:
    checkUInt(ctx, loc, val, 17, rel);
    [[fallthrough]];
  case R_AVR_HI8_LDI_PM:
    checkAlignment(ctx, loc, val, 2, rel);
    writeLDI(loc, (val >> 9) & 0xff);
    break;
  case R_AVR_HH8_LDI_PM:
    checkAlignment(ctx, loc, val, 2, rel);
    writeLDI(loc, (val >> 17) & 0xff);
    break;
```

- EN: Declares or implements routines including `checkUInt`, `checkAlignment`, `writeLDI`. Notable symbols here include `checkUInt`, `checkAlignment`, `writeLDI`.
- CN: 这里声明或实现函数，例如 `checkUInt`, `checkAlignment`, `writeLDI`。这里较值得关注的符号包括 `checkUInt`, `checkAlignment`, `writeLDI`。

### Lines 200-213

```cpp

  case R_AVR_LO8_LDI_PM_NEG:
    checkAlignment(ctx, loc, val, 2, rel);
    writeLDI(loc, (-val >> 1) & 0xff);
    break;
  case R_AVR_HI8_LDI_PM_NEG:
    checkAlignment(ctx, loc, val, 2, rel);
    writeLDI(loc, (-val >> 9) & 0xff);
    break;
  case R_AVR_HH8_LDI_PM_NEG:
    checkAlignment(ctx, loc, val, 2, rel);
    writeLDI(loc, (-val >> 17) & 0xff);
    break;
```

- EN: Declares or implements routines including `checkAlignment`, `writeLDI`. Notable symbols here include `checkAlignment`, `writeLDI`.
- CN: 这里声明或实现函数，例如 `checkAlignment`, `writeLDI`。这里较值得关注的符号包括 `checkAlignment`, `writeLDI`。

### Lines 214-221

```cpp
  case R_AVR_LDS_STS_16: {
    checkUInt(ctx, loc, val, 7, rel);
    const uint16_t hi = val >> 4;
    const uint16_t lo = val & 0xf;
    write16le(loc, (read16le(loc) & 0xf8f0) | ((hi << 8) | lo));
    break;
  }
```

- EN: Declares or implements routines including `checkUInt`, `write16le`. Notable symbols here include `checkUInt`, `write16le`.
- CN: 这里声明或实现函数，例如 `checkUInt`, `write16le`。这里较值得关注的符号包括 `checkUInt`, `write16le`。

### Lines 222-230

```cpp
  case R_AVR_PORT5:
    checkUInt(ctx, loc, val, 5, rel);
    write16le(loc, (read16le(loc) & 0xff07) | (val << 3));
    break;
  case R_AVR_PORT6:
    checkUInt(ctx, loc, val, 6, rel);
    write16le(loc, (read16le(loc) & 0xf9f0) | (val & 0x30) << 5 | (val & 0x0f));
    break;
```

- EN: Declares or implements routines including `checkUInt`, `write16le`. Notable symbols here include `checkUInt`, `write16le`.
- CN: 这里声明或实现函数，例如 `checkUInt`, `write16le`。这里较值得关注的符号包括 `checkUInt`, `write16le`。

### Lines 231-245

```cpp
  // Since every jump destination is word aligned we gain an extra bit
  case R_AVR_7_PCREL: {
    checkInt(ctx, loc, val - 2, 8, rel);
    checkAlignment(ctx, loc, val, 2, rel);
    const uint16_t target = (val - 2) >> 1;
    write16le(loc, (read16le(loc) & 0xfc07) | ((target & 0x7f) << 3));
    break;
  }
  case R_AVR_13_PCREL: {
    checkAlignment(ctx, loc, val, 2, rel);
    const uint16_t target = (val - 2) >> 1;
    write16le(loc, (read16le(loc) & 0xf000) | (target & 0xfff));
    break;
  }
```

- EN: Declares or implements routines including `checkInt`, `checkAlignment`, `write16le`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `checkInt`, `checkAlignment`, `write16le`.
- CN: 这里声明或实现函数，例如 `checkInt`, `checkAlignment`, `write16le`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `checkInt`, `checkAlignment`, `write16le`。

### Lines 246-255

```cpp
  case R_AVR_6:
    checkInt(ctx, loc, val, 6, rel);
    write16le(loc, (read16le(loc) & 0xd3f8) | (val & 0x20) << 8 |
                       (val & 0x18) << 7 | (val & 0x07));
    break;
  case R_AVR_6_ADIW:
    checkInt(ctx, loc, val, 6, rel);
    write16le(loc, (read16le(loc) & 0xff30) | (val & 0x30) << 2 | (val & 0x0F));
    break;
```

- EN: Declares or implements routines including `checkInt`, `write16le`. Notable symbols here include `checkInt`, `write16le`.
- CN: 这里声明或实现函数，例如 `checkInt`, `write16le`。这里较值得关注的符号包括 `checkInt`, `write16le`。

### Lines 256-268

```cpp
  case R_AVR_CALL: {
    checkAlignment(ctx, loc, val, 2, rel);
    uint16_t hi = val >> 17;
    uint16_t lo = val >> 1;
    write16le(loc, read16le(loc) | ((hi >> 1) << 4) | (hi & 1));
    write16le(loc + 2, lo);
    break;
  }
  default:
    llvm_unreachable("unknown relocation");
  }
}
```

- EN: Declares or implements routines including `checkAlignment`, `write16le`, `llvm_unreachable`. Notable symbols here include `checkAlignment`, `write16le`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `checkAlignment`, `write16le`, `llvm_unreachable`。这里较值得关注的符号包括 `checkAlignment`, `write16le`, `llvm_unreachable`。

### Lines 269-277

```cpp
void elf::setAVRTargetInfo(Ctx &ctx) { ctx.target.reset(new AVR(ctx)); }

static uint32_t getEFlags(InputFile *file) {
  return cast<ObjFile<ELF32LE>>(file)->getObj().getHeader().e_flags;
}

uint32_t AVR::calcEFlags() const {
  assert(!ctx.objectFiles.empty());
```

- EN: Declares or implements routines including `setAVRTargetInfo`, `getEFlags`, `calcEFlags`, `assert`. Notable symbols here include `setAVRTargetInfo`, `getEFlags`, `calcEFlags`, `assert`.
- CN: 这里声明或实现函数，例如 `setAVRTargetInfo`, `getEFlags`, `calcEFlags`, `assert`。这里较值得关注的符号包括 `setAVRTargetInfo`, `getEFlags`, `calcEFlags`, `assert`。

### Lines 278-289

```cpp
  uint32_t flags = getEFlags(ctx.objectFiles[0]);
  bool hasLinkRelaxFlag = flags & EF_AVR_LINKRELAX_PREPARED;

  for (InputFile *f : ArrayRef(ctx.objectFiles).slice(1)) {
    uint32_t objFlags = getEFlags(f);
    if ((objFlags & EF_AVR_ARCH_MASK) != (flags & EF_AVR_ARCH_MASK))
      ErrAlways(ctx)
          << f << ": cannot link object files with incompatible target ISA";
    if (!(objFlags & EF_AVR_LINKRELAX_PREPARED))
      hasLinkRelaxFlag = false;
  }
```

- EN: Declares or implements routines including `getEFlags`, `ErrAlways`. Notable symbols here include `getEFlags`, `ErrAlways`.
- CN: 这里声明或实现函数，例如 `getEFlags`, `ErrAlways`。这里较值得关注的符号包括 `getEFlags`, `ErrAlways`。

### Lines 290-294

```cpp
  if (!hasLinkRelaxFlag)
    flags &= ~EF_AVR_LINKRELAX_PREPARED;

  return flags;
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

## Key Concepts / 关键概念

- `AVR`: class or struct interface / 类或结构体接口
- `AVR`: function or method entry point / 函数或方法入口
- `calcEFlags`: function or method entry point / 函数或方法入口
- `Err`: function or method entry point / 函数或方法入口
- `writeLDI`: function or method entry point / 函数或方法入口
- `write16le`: function or method entry point / 函数或方法入口
- `llvm`: namespace scope / 命名空间作用域
- `lld`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- LLVM headers / LLVM 头文件: `llvm/BinaryFormat/ELF.h`, `llvm/Support/Endian.h`
- System headers / 系统头文件: `InputFiles.h`, `Symbols.h`, `Target.h`, `Thunks.h`
- Directory context / 目录上下文: `lld/ELF/Arch` neighbors usually cooperate with this file to provide the surrounding subsystem / `lld/ELF/Arch` 下的相邻文件通常与本文件协作组成对应子系统
