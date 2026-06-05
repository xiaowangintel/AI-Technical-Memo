# Relocation.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Core/Relocation.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: Object file relocations. It also sits in code that implements BOLT core binary-analysis logic. / 该文件实现 BOLT 核心二进制分析逻辑。 源码头部说明其职责是：Object file relocations。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Core/Relocation.cpp - Object file relocations -----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the Relocation class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-20

```cpp
#include "bolt/Core/Relocation.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/Object/ELF.h"
#include "llvm/Object/ObjectFile.h"
```

- EN: Pulls in 7 header(s) from local project, LLVM dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 7 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 21-30

```cpp
using namespace llvm;
using namespace bolt;

namespace ELFReserved {
enum {
  R_RISCV_TPREL_I = 49,
  R_RISCV_TPREL_S = 50,
};
} // namespace ELFReserved
```

- EN: Works inside namespace scope `llvm`, `bolt`, `ELFReserved` to organize symbols. Notable symbols here include `llvm`, `bolt`, `ELFReserved`.
- CN: 这里位于命名空间 `llvm`, `bolt`, `ELFReserved` 中，用于组织符号作用域。这里较值得关注的符号包括 `llvm`, `bolt`, `ELFReserved`。

### Lines 31-48

```cpp
Triple::ArchType Relocation::Arch;

static bool isSupportedX86(uint32_t Type) {
  switch (Type) {
  default:
    return false;
  case ELF::R_X86_64_8:
  case ELF::R_X86_64_16:
  case ELF::R_X86_64_32:
  case ELF::R_X86_64_32S:
  case ELF::R_X86_64_64:
  case ELF::R_X86_64_PC8:
  case ELF::R_X86_64_PC32:
  case ELF::R_X86_64_PC64:
  case ELF::R_X86_64_PLT32:
  case ELF::R_X86_64_GOTPC64:
  case ELF::R_X86_64_GOTPCREL:
  case ELF::R_X86_64_GOTTPOFF:
```

- EN: Declares or implements routines including `isSupportedX86`. Notable symbols here include `isSupportedX86`.
- CN: 这里声明或实现函数，例如 `isSupportedX86`。这里较值得关注的符号包括 `isSupportedX86`。

### Lines 49-66

```cpp
  case ELF::R_X86_64_TPOFF32:
  case ELF::R_X86_64_GOTPCRELX:
  case ELF::R_X86_64_REX_GOTPCRELX:
    return true;
  }
}

static bool isSupportedAArch64(uint32_t Type) {
  switch (Type) {
  default:
    return false;
  case ELF::R_AARCH64_CALL26:
  case ELF::R_AARCH64_JUMP26:
  case ELF::R_AARCH64_TSTBR14:
  case ELF::R_AARCH64_CONDBR19:
  case ELF::R_AARCH64_ADR_PREL_LO21:
  case ELF::R_AARCH64_ADR_PREL_PG_HI21:
  case ELF::R_AARCH64_ADR_PREL_PG_HI21_NC:
```

- EN: Declares or implements routines including `isSupportedAArch64`. Notable symbols here include `isSupportedAArch64`.
- CN: 这里声明或实现函数，例如 `isSupportedAArch64`。这里较值得关注的符号包括 `isSupportedAArch64`。

### Lines 67-84

```cpp
  case ELF::R_AARCH64_LDST64_ABS_LO12_NC:
  case ELF::R_AARCH64_ADD_ABS_LO12_NC:
  case ELF::R_AARCH64_LDST128_ABS_LO12_NC:
  case ELF::R_AARCH64_LDST32_ABS_LO12_NC:
  case ELF::R_AARCH64_LDST16_ABS_LO12_NC:
  case ELF::R_AARCH64_LDST8_ABS_LO12_NC:
  case ELF::R_AARCH64_ADR_GOT_PAGE:
  case ELF::R_AARCH64_TLSDESC_ADR_PREL21:
  case ELF::R_AARCH64_TLSDESC_ADR_PAGE21:
  case ELF::R_AARCH64_TLSIE_LD64_GOTTPREL_LO12_NC:
  case ELF::R_AARCH64_TLSLE_ADD_TPREL_HI12:
  case ELF::R_AARCH64_TLSLE_ADD_TPREL_LO12_NC:
  case ELF::R_AARCH64_TLSLE_MOVW_TPREL_G0:
  case ELF::R_AARCH64_TLSLE_MOVW_TPREL_G0_NC:
  case ELF::R_AARCH64_LD64_GOT_LO12_NC:
  case ELF::R_AARCH64_TLSDESC_LD64_LO12:
  case ELF::R_AARCH64_TLSDESC_ADD_LO12:
  case ELF::R_AARCH64_TLSIE_ADR_GOTTPREL_PAGE21:
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 85-102

```cpp
  case ELF::R_AARCH64_PREL16:
  case ELF::R_AARCH64_PREL32:
  case ELF::R_AARCH64_PREL64:
  case ELF::R_AARCH64_ABS16:
  case ELF::R_AARCH64_ABS32:
  case ELF::R_AARCH64_ABS64:
  case ELF::R_AARCH64_MOVW_UABS_G0:
  case ELF::R_AARCH64_MOVW_UABS_G0_NC:
  case ELF::R_AARCH64_MOVW_UABS_G1:
  case ELF::R_AARCH64_MOVW_UABS_G1_NC:
  case ELF::R_AARCH64_MOVW_UABS_G2:
  case ELF::R_AARCH64_MOVW_UABS_G2_NC:
  case ELF::R_AARCH64_MOVW_UABS_G3:
  case ELF::R_AARCH64_PLT32:
    return true;
  }
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 103-120

```cpp
static bool isSupportedRISCV(uint32_t Type) {
  switch (Type) {
  default:
    return false;
  case ELF::R_RISCV_JAL:
  case ELF::R_RISCV_CALL:
  case ELF::R_RISCV_CALL_PLT:
  case ELF::R_RISCV_BRANCH:
  case ELF::R_RISCV_RELAX:
  case ELF::R_RISCV_GOT_HI20:
  case ELF::R_RISCV_PCREL_HI20:
  case ELF::R_RISCV_PCREL_LO12_I:
  case ELF::R_RISCV_PCREL_LO12_S:
  case ELF::R_RISCV_RVC_JUMP:
  case ELF::R_RISCV_RVC_BRANCH:
  case ELF::R_RISCV_ADD32:
  case ELF::R_RISCV_SUB32:
  case ELF::R_RISCV_HI20:
```

- EN: Declares or implements routines including `isSupportedRISCV`. Notable symbols here include `isSupportedRISCV`.
- CN: 这里声明或实现函数，例如 `isSupportedRISCV`。这里较值得关注的符号包括 `isSupportedRISCV`。

### Lines 121-135

```cpp
  case ELF::R_RISCV_LO12_I:
  case ELF::R_RISCV_LO12_S:
  case ELF::R_RISCV_64:
  case ELF::R_RISCV_TLS_GOT_HI20:
  case ELF::R_RISCV_TLS_GD_HI20:
  case ELF::R_RISCV_TPREL_HI20:
  case ELF::R_RISCV_TPREL_ADD:
  case ELF::R_RISCV_TPREL_LO12_I:
  case ELF::R_RISCV_TPREL_LO12_S:
  case ELFReserved::R_RISCV_TPREL_I:
  case ELFReserved::R_RISCV_TPREL_S:
    return true;
  }
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 136-153

```cpp
static size_t getSizeForTypeX86(uint32_t Type) {
  switch (Type) {
  default:
    errs() << object::getELFRelocationTypeName(ELF::EM_X86_64, Type) << '\n';
    llvm_unreachable("unsupported relocation type");
  case ELF::R_X86_64_8:
  case ELF::R_X86_64_PC8:
    return 1;
  case ELF::R_X86_64_16:
    return 2;
  case ELF::R_X86_64_PLT32:
  case ELF::R_X86_64_PC32:
  case ELF::R_X86_64_32S:
  case ELF::R_X86_64_32:
  case ELF::R_X86_64_GOTPCREL:
  case ELF::R_X86_64_GOTTPOFF:
  case ELF::R_X86_64_TPOFF32:
  case ELF::R_X86_64_GOTPCRELX:
```

- EN: Declares or implements routines including `getSizeForTypeX86`, `errs`, `llvm_unreachable`. Notable symbols here include `getSizeForTypeX86`, `errs`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `getSizeForTypeX86`, `errs`, `llvm_unreachable`。这里较值得关注的符号包括 `getSizeForTypeX86`, `errs`, `llvm_unreachable`。

### Lines 154-162

```cpp
  case ELF::R_X86_64_REX_GOTPCRELX:
    return 4;
  case ELF::R_X86_64_PC64:
  case ELF::R_X86_64_64:
  case ELF::R_X86_64_GOTPC64:
    return 8;
  }
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 163-180

```cpp
static size_t getSizeForTypeAArch64(uint32_t Type) {
  switch (Type) {
  default:
    errs() << object::getELFRelocationTypeName(ELF::EM_AARCH64, Type) << '\n';
    llvm_unreachable("unsupported relocation type");
  case ELF::R_AARCH64_ABS16:
  case ELF::R_AARCH64_PREL16:
    return 2;
  case ELF::R_AARCH64_CALL26:
  case ELF::R_AARCH64_JUMP26:
  case ELF::R_AARCH64_TSTBR14:
  case ELF::R_AARCH64_CONDBR19:
  case ELF::R_AARCH64_ADR_PREL_LO21:
  case ELF::R_AARCH64_ADR_PREL_PG_HI21:
  case ELF::R_AARCH64_ADR_PREL_PG_HI21_NC:
  case ELF::R_AARCH64_LDST64_ABS_LO12_NC:
  case ELF::R_AARCH64_ADD_ABS_LO12_NC:
  case ELF::R_AARCH64_LDST128_ABS_LO12_NC:
```

- EN: Declares or implements routines including `getSizeForTypeAArch64`, `errs`, `llvm_unreachable`. Notable symbols here include `getSizeForTypeAArch64`, `errs`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `getSizeForTypeAArch64`, `errs`, `llvm_unreachable`。这里较值得关注的符号包括 `getSizeForTypeAArch64`, `errs`, `llvm_unreachable`。

### Lines 181-198

```cpp
  case ELF::R_AARCH64_LDST32_ABS_LO12_NC:
  case ELF::R_AARCH64_LDST16_ABS_LO12_NC:
  case ELF::R_AARCH64_LDST8_ABS_LO12_NC:
  case ELF::R_AARCH64_ADR_GOT_PAGE:
  case ELF::R_AARCH64_TLSDESC_ADR_PREL21:
  case ELF::R_AARCH64_TLSDESC_ADR_PAGE21:
  case ELF::R_AARCH64_TLSIE_LD64_GOTTPREL_LO12_NC:
  case ELF::R_AARCH64_TLSLE_ADD_TPREL_HI12:
  case ELF::R_AARCH64_TLSLE_ADD_TPREL_LO12_NC:
  case ELF::R_AARCH64_TLSLE_MOVW_TPREL_G0:
  case ELF::R_AARCH64_TLSLE_MOVW_TPREL_G0_NC:
  case ELF::R_AARCH64_LD64_GOT_LO12_NC:
  case ELF::R_AARCH64_TLSDESC_LD64_LO12:
  case ELF::R_AARCH64_TLSDESC_ADD_LO12:
  case ELF::R_AARCH64_TLSIE_ADR_GOTTPREL_PAGE21:
  case ELF::R_AARCH64_PREL32:
  case ELF::R_AARCH64_MOVW_UABS_G0:
  case ELF::R_AARCH64_MOVW_UABS_G0_NC:
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 199-212

```cpp
  case ELF::R_AARCH64_MOVW_UABS_G1:
  case ELF::R_AARCH64_MOVW_UABS_G1_NC:
  case ELF::R_AARCH64_MOVW_UABS_G2:
  case ELF::R_AARCH64_MOVW_UABS_G2_NC:
  case ELF::R_AARCH64_MOVW_UABS_G3:
  case ELF::R_AARCH64_ABS32:
  case ELF::R_AARCH64_PLT32:
    return 4;
  case ELF::R_AARCH64_ABS64:
  case ELF::R_AARCH64_PREL64:
    return 8;
  }
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 213-230

```cpp
static size_t getSizeForTypeRISCV(uint32_t Type) {
  switch (Type) {
  default:
    errs() << object::getELFRelocationTypeName(ELF::EM_RISCV, Type) << '\n';
    llvm_unreachable("unsupported relocation type");
  case ELF::R_RISCV_RVC_JUMP:
  case ELF::R_RISCV_RVC_BRANCH:
    return 2;
  case ELF::R_RISCV_JAL:
  case ELF::R_RISCV_BRANCH:
  case ELF::R_RISCV_PCREL_HI20:
  case ELF::R_RISCV_PCREL_LO12_I:
  case ELF::R_RISCV_PCREL_LO12_S:
  case ELF::R_RISCV_32_PCREL:
  case ELF::R_RISCV_CALL:
  case ELF::R_RISCV_CALL_PLT:
  case ELF::R_RISCV_ADD32:
  case ELF::R_RISCV_SUB32:
```

- EN: Declares or implements routines including `getSizeForTypeRISCV`, `errs`, `llvm_unreachable`. Notable symbols here include `getSizeForTypeRISCV`, `errs`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `getSizeForTypeRISCV`, `errs`, `llvm_unreachable`。这里较值得关注的符号包括 `getSizeForTypeRISCV`, `errs`, `llvm_unreachable`。

### Lines 231-243

```cpp
  case ELF::R_RISCV_HI20:
  case ELF::R_RISCV_LO12_I:
  case ELF::R_RISCV_LO12_S:
    return 4;
  case ELF::R_RISCV_64:
  case ELF::R_RISCV_GOT_HI20:
  case ELF::R_RISCV_TLS_GOT_HI20:
  case ELF::R_RISCV_TLS_GD_HI20:
    // See extractValueRISCV for why this is necessary.
    return 8;
  }
}
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 244-258

```cpp
static bool skipRelocationTypeX86(uint32_t Type) {
  return Type == ELF::R_X86_64_NONE;
}

static bool skipRelocationTypeAArch64(uint32_t Type) {
  switch (Type) {
  default:
    return false;
  case ELF::R_AARCH64_NONE:
  case ELF::R_AARCH64_LD_PREL_LO19:
  case ELF::R_AARCH64_TLSDESC_CALL:
    return true;
  }
}
```

- EN: Declares or implements routines including `skipRelocationTypeX86`, `skipRelocationTypeAArch64`. Notable symbols here include `skipRelocationTypeX86`, `skipRelocationTypeAArch64`.
- CN: 这里声明或实现函数，例如 `skipRelocationTypeX86`, `skipRelocationTypeAArch64`。这里较值得关注的符号包括 `skipRelocationTypeX86`, `skipRelocationTypeAArch64`。

### Lines 259-268

```cpp
static bool skipRelocationTypeRISCV(uint32_t Type) {
  switch (Type) {
  default:
    return false;
  case ELF::R_RISCV_NONE:
  case ELF::R_RISCV_RELAX:
    return true;
  }
}
```

- EN: Declares or implements routines including `skipRelocationTypeRISCV`. Notable symbols here include `skipRelocationTypeRISCV`.
- CN: 这里声明或实现函数，例如 `skipRelocationTypeRISCV`。这里较值得关注的符号包括 `skipRelocationTypeRISCV`。

### Lines 269-282

```cpp
static uint64_t encodeValueX86(uint32_t Type, uint64_t Value, uint64_t PC) {
  switch (Type) {
  default:
    llvm_unreachable("unsupported relocation");
  case ELF::R_X86_64_64:
  case ELF::R_X86_64_32:
    break;
  case ELF::R_X86_64_PC32:
    Value -= PC;
    break;
  }
  return Value;
}
```

- EN: Declares or implements routines including `encodeValueX86`, `llvm_unreachable`. Notable symbols here include `encodeValueX86`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `encodeValueX86`, `llvm_unreachable`。这里较值得关注的符号包括 `encodeValueX86`, `llvm_unreachable`。

### Lines 283-292

```cpp
static bool canEncodeValueAArch64(uint32_t Type, uint64_t Value, uint64_t PC) {
  switch (Type) {
  default:
    llvm_unreachable("unsupported relocation");
  case ELF::R_AARCH64_CALL26:
  case ELF::R_AARCH64_JUMP26:
    return isInt<28>(Value - PC);
  }
}
```

- EN: Declares or implements routines including `canEncodeValueAArch64`, `llvm_unreachable`. Notable symbols here include `canEncodeValueAArch64`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `canEncodeValueAArch64`, `llvm_unreachable`。这里较值得关注的符号包括 `canEncodeValueAArch64`, `llvm_unreachable`。

### Lines 293-310

```cpp
static uint64_t encodeValueAArch64(uint32_t Type, uint64_t Value, uint64_t PC) {
  switch (Type) {
  default:
    llvm_unreachable("unsupported relocation");
  case ELF::R_AARCH64_ABS16:
  case ELF::R_AARCH64_ABS32:
  case ELF::R_AARCH64_ABS64:
    break;
  case ELF::R_AARCH64_PREL16:
  case ELF::R_AARCH64_PREL32:
  case ELF::R_AARCH64_PREL64:
    Value -= PC;
    break;
  case ELF::R_AARCH64_CALL26:
    Value -= PC;
    assert(isInt<28>(Value) && "only PC +/- 128MB is allowed for direct call");
    // Immediate goes in bits 25:0 of BL.
    // OP 1001_01 goes in bits 31:26 of BL.
```

- EN: Declares or implements routines including `encodeValueAArch64`, `llvm_unreachable`, `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `encodeValueAArch64`, `llvm_unreachable`, `assert`.
- CN: 这里声明或实现函数，例如 `encodeValueAArch64`, `llvm_unreachable`, `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `encodeValueAArch64`, `llvm_unreachable`, `assert`。

### Lines 311-324

```cpp
    Value = ((Value >> 2) & 0x3ffffff) | 0x94000000ULL;
    break;
  case ELF::R_AARCH64_JUMP26:
    Value -= PC;
    assert(isInt<28>(Value) &&
           "only PC +/- 128MB is allowed for direct branch");
    // Immediate goes in bits 25:0 of B.
    // OP 0001_01 goes in bits 31:26 of B.
    Value = ((Value >> 2) & 0x3ffffff) | 0x14000000ULL;
    break;
  }
  return Value;
}
```

- EN: Declares or implements routines including `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`。

### Lines 325-334

```cpp
static uint64_t canEncodeValueRISCV(uint32_t Type, uint64_t Value,
                                    uint64_t PC) {
  switch (Type) {
  default:
    llvm_unreachable("unsupported relocation");
  case ELF::R_RISCV_64:
    return true;
  }
}
```

- EN: Declares or implements routines including `llvm_unreachable`. Notable symbols here include `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`。这里较值得关注的符号包括 `llvm_unreachable`。

### Lines 335-344

```cpp
static uint64_t encodeValueRISCV(uint32_t Type, uint64_t Value, uint64_t PC) {
  switch (Type) {
  default:
    llvm_unreachable("unsupported relocation");
  case ELF::R_RISCV_64:
    break;
  }
  return Value;
}
```

- EN: Declares or implements routines including `encodeValueRISCV`, `llvm_unreachable`. Notable symbols here include `encodeValueRISCV`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `encodeValueRISCV`, `llvm_unreachable`。这里较值得关注的符号包括 `encodeValueRISCV`, `llvm_unreachable`。

### Lines 345-352

```cpp
static uint64_t extractValueX86(uint32_t Type, uint64_t Contents, uint64_t PC) {
  if (Type == ELF::R_X86_64_32S)
    return SignExtend64<32>(Contents);
  if (Relocation::isPCRelative(Type))
    return SignExtend64(Contents, 8 * Relocation::getSizeForType(Type));
  return Contents;
}
```

- EN: Declares or implements routines including `extractValueX86`. Notable symbols here include `extractValueX86`.
- CN: 这里声明或实现函数，例如 `extractValueX86`。这里较值得关注的符号包括 `extractValueX86`。

### Lines 353-370

```cpp
static uint64_t extractValueAArch64(uint32_t Type, uint64_t Contents,
                                    uint64_t PC) {
  switch (Type) {
  default:
    errs() << object::getELFRelocationTypeName(ELF::EM_AARCH64, Type) << '\n';
    llvm_unreachable("unsupported relocation type");
  case ELF::R_AARCH64_ABS16:
  case ELF::R_AARCH64_ABS32:
  case ELF::R_AARCH64_ABS64:
    return Contents;
  case ELF::R_AARCH64_PREL16:
    return static_cast<int64_t>(PC) + SignExtend64<16>(Contents & 0xffff);
  case ELF::R_AARCH64_PREL32:
  case ELF::R_AARCH64_PLT32:
    return static_cast<int64_t>(PC) + SignExtend64<32>(Contents & 0xffffffff);
  case ELF::R_AARCH64_PREL64:
    return static_cast<int64_t>(PC) + Contents;
  case ELF::R_AARCH64_JUMP26:
```

- EN: Declares or implements routines including `errs`, `llvm_unreachable`. Notable symbols here include `errs`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `errs`, `llvm_unreachable`。这里较值得关注的符号包括 `errs`, `llvm_unreachable`。

### Lines 371-388

```cpp
  case ELF::R_AARCH64_CALL26:
    // Immediate goes in bits 25:0 of B and BL.
    Contents &= ~0xfffffffffc000000ULL;
    return static_cast<int64_t>(PC) + SignExtend64<28>(Contents << 2);
  case ELF::R_AARCH64_TSTBR14:
    // Immediate:15:2 goes in bits 18:5 of TBZ, TBNZ
    Contents &= ~0xfffffffffff8001fULL;
    return static_cast<int64_t>(PC) + SignExtend64<16>(Contents >> 3);
  case ELF::R_AARCH64_CONDBR19:
    // Immediate:20:2 goes in bits 23:5 of Bcc, CBZ, CBNZ
    Contents &= ~0xffffffffff00001fULL;
    return static_cast<int64_t>(PC) + SignExtend64<21>(Contents >> 3);
  case ELF::R_AARCH64_ADR_GOT_PAGE:
  case ELF::R_AARCH64_TLSDESC_ADR_PREL21:
  case ELF::R_AARCH64_TLSDESC_ADR_PAGE21:
  case ELF::R_AARCH64_TLSIE_ADR_GOTTPREL_PAGE21:
  case ELF::R_AARCH64_ADR_PREL_LO21:
  case ELF::R_AARCH64_ADR_PREL_PG_HI21:
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 389-399

```cpp
  case ELF::R_AARCH64_ADR_PREL_PG_HI21_NC: {
    // Bits 32:12 of Symbol address goes in bits 30:29 + 23:5 of ADRP
    // and ADR instructions
    bool IsAdr = !!(((Contents >> 31) & 0x1) == 0);
    Contents &= ~0xffffffff9f00001fUll;
    uint64_t LowBits = (Contents >> 29) & 0x3;
    uint64_t HighBits = (Contents >> 5) & 0x7ffff;
    Contents = LowBits | (HighBits << 2);
    if (IsAdr)
      return static_cast<int64_t>(PC) + SignExtend64<21>(Contents);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 400-417

```cpp
    // ADRP instruction
    Contents = static_cast<int64_t>(PC) + SignExtend64<33>(Contents << 12);
    Contents &= ~0xfffUll;
    return Contents;
  }
  case ELF::R_AARCH64_TLSIE_LD64_GOTTPREL_LO12_NC:
  case ELF::R_AARCH64_TLSDESC_LD64_LO12:
  case ELF::R_AARCH64_LD64_GOT_LO12_NC:
  case ELF::R_AARCH64_LDST64_ABS_LO12_NC: {
    // Immediate goes in bits 21:10 of LD/ST instruction, taken
    // from bits 11:3 of Symbol address
    Contents &= ~0xffffffffffc003ffU;
    return Contents >> (10 - 3);
  }
  case ELF::R_AARCH64_TLSLE_ADD_TPREL_HI12:
  case ELF::R_AARCH64_TLSLE_ADD_TPREL_LO12_NC:
  case ELF::R_AARCH64_TLSDESC_ADD_LO12:
  case ELF::R_AARCH64_ADD_ABS_LO12_NC: {
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 418-435

```cpp
    // Immediate goes in bits 21:10 of ADD instruction
    Contents &= ~0xffffffffffc003ffU;
    return Contents >> (10 - 0);
  }
  case ELF::R_AARCH64_LDST128_ABS_LO12_NC: {
    // Immediate goes in bits 21:10 of ADD instruction, taken
    // from bits 11:4 of Symbol address
    Contents &= ~0xffffffffffc003ffU;
    return Contents >> (10 - 4);
  }
  case ELF::R_AARCH64_LDST32_ABS_LO12_NC: {
    // Immediate goes in bits 21:10 of ADD instruction, taken
    // from bits 11:2 of Symbol address
    Contents &= ~0xffffffffffc003ffU;
    return Contents >> (10 - 2);
  }
  case ELF::R_AARCH64_LDST16_ABS_LO12_NC: {
    // Immediate goes in bits 21:10 of ADD instruction, taken
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 436-453

```cpp
    // from bits 11:1 of Symbol address
    Contents &= ~0xffffffffffc003ffU;
    return Contents >> (10 - 1);
  }
  case ELF::R_AARCH64_LDST8_ABS_LO12_NC: {
    // Immediate goes in bits 21:10 of ADD instruction, taken
    // from bits 11:0 of Symbol address
    Contents &= ~0xffffffffffc003ffU;
    return Contents >> (10 - 0);
  }
  case ELF::R_AARCH64_MOVW_UABS_G3:
  case ELF::R_AARCH64_MOVW_UABS_G2_NC:
  case ELF::R_AARCH64_MOVW_UABS_G2:
  case ELF::R_AARCH64_MOVW_UABS_G1_NC:
  case ELF::R_AARCH64_MOVW_UABS_G1:
  case ELF::R_AARCH64_MOVW_UABS_G0_NC:
  case ELF::R_AARCH64_MOVW_UABS_G0:
    // The shift goes in bits 22:21 of MOV* instructions
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 454-464

```cpp
    uint8_t Shift = (Contents >> 21) & 0x3;
    // Immediate goes in bits 20:5
    Contents = (Contents >> 5) & 0xffff;
    return Contents << (16 * Shift);
  }
}

static uint64_t extractUImmRISCV(uint32_t Contents) {
  return SignExtend64<32>(Contents & 0xfffff000);
}
```

- EN: Declares or implements routines including `extractUImmRISCV`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `extractUImmRISCV`.
- CN: 这里声明或实现函数，例如 `extractUImmRISCV`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `extractUImmRISCV`。

### Lines 465-472

```cpp
static uint64_t extractIImmRISCV(uint32_t Contents) {
  return SignExtend64<12>(Contents >> 20);
}

static uint64_t extractSImmRISCV(uint32_t Contents) {
  return SignExtend64<12>(((Contents >> 7) & 0x1f) | ((Contents >> 25) << 5));
}
```

- EN: Declares or implements routines including `extractIImmRISCV`, `extractSImmRISCV`. Notable symbols here include `extractIImmRISCV`, `extractSImmRISCV`.
- CN: 这里声明或实现函数，例如 `extractIImmRISCV`, `extractSImmRISCV`。这里较值得关注的符号包括 `extractIImmRISCV`, `extractSImmRISCV`。

### Lines 473-484

```cpp
static uint64_t extractJImmRISCV(uint32_t Contents) {
  return SignExtend64<21>(
      (((Contents >> 21) & 0x3ff) << 1) | (((Contents >> 20) & 0x1) << 11) |
      (((Contents >> 12) & 0xff) << 12) | (((Contents >> 31) & 0x1) << 20));
}

static uint64_t extractBImmRISCV(uint32_t Contents) {
  return SignExtend64<13>(
      (((Contents >> 8) & 0xf) << 1) | (((Contents >> 25) & 0x3f) << 5) |
      (((Contents >> 7) & 0x1) << 11) | (((Contents >> 31) & 0x1) << 12));
}
```

- EN: Declares or implements routines including `extractJImmRISCV`, `extractBImmRISCV`. Notable symbols here include `extractJImmRISCV`, `extractBImmRISCV`.
- CN: 这里声明或实现函数，例如 `extractJImmRISCV`, `extractBImmRISCV`。这里较值得关注的符号包括 `extractJImmRISCV`, `extractBImmRISCV`。

### Lines 485-502

```cpp
static uint64_t extractValueRISCV(uint32_t Type, uint64_t Contents,
                                  uint64_t PC) {
  switch (Type) {
  default:
    errs() << object::getELFRelocationTypeName(ELF::EM_RISCV, Type) << '\n';
    llvm_unreachable("unsupported relocation type");
  case ELF::R_RISCV_JAL:
    return extractJImmRISCV(Contents);
  case ELF::R_RISCV_CALL:
  case ELF::R_RISCV_CALL_PLT:
    return extractUImmRISCV(Contents);
  case ELF::R_RISCV_BRANCH:
    return extractBImmRISCV(Contents);
  case ELF::R_RISCV_GOT_HI20:
  case ELF::R_RISCV_TLS_GOT_HI20:
  case ELF::R_RISCV_TLS_GD_HI20:
    // We need to know the exact address of the GOT entry so we extract the
    // value from both the AUIPC and L[D|W]. We cannot rely on the symbol in the
```

- EN: Declares or implements routines including `errs`, `llvm_unreachable`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `errs`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `errs`, `llvm_unreachable`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `errs`, `llvm_unreachable`。

### Lines 503-520

```cpp
    // relocation for this since it simply refers to the object that is stored
    // in the GOT entry, not to the entry itself.
    return extractUImmRISCV(Contents & 0xffffffff) +
           extractIImmRISCV(Contents >> 32);
  case ELF::R_RISCV_PCREL_HI20:
  case ELF::R_RISCV_HI20:
    return extractUImmRISCV(Contents);
  case ELF::R_RISCV_PCREL_LO12_I:
  case ELF::R_RISCV_LO12_I:
    return extractIImmRISCV(Contents);
  case ELF::R_RISCV_PCREL_LO12_S:
  case ELF::R_RISCV_LO12_S:
    return extractSImmRISCV(Contents);
  case ELF::R_RISCV_RVC_JUMP:
    return SignExtend64<11>(Contents >> 2);
  case ELF::R_RISCV_RVC_BRANCH:
    return SignExtend64<8>(((Contents >> 2) & 0x1f) | ((Contents >> 5) & 0xe0));
  case ELF::R_RISCV_ADD32:
```

- EN: Declares or implements routines including `extractIImmRISCV`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `extractIImmRISCV`.
- CN: 这里声明或实现函数，例如 `extractIImmRISCV`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `extractIImmRISCV`。

### Lines 521-538

```cpp
  case ELF::R_RISCV_SUB32:
  case ELF::R_RISCV_64:
    return Contents;
  }
}

static bool isGOTX86(uint32_t Type) {
  switch (Type) {
  default:
    return false;
  case ELF::R_X86_64_GOT32:
  case ELF::R_X86_64_GOTPCREL:
  case ELF::R_X86_64_GOTTPOFF:
  case ELF::R_X86_64_GOTOFF64:
  case ELF::R_X86_64_GOTPC32:
  case ELF::R_X86_64_GOT64:
  case ELF::R_X86_64_GOTPCREL64:
  case ELF::R_X86_64_GOTPC64:
```

- EN: Declares or implements routines including `isGOTX86`. Notable symbols here include `isGOTX86`.
- CN: 这里声明或实现函数，例如 `isGOTX86`。这里较值得关注的符号包括 `isGOTX86`。

### Lines 539-546

```cpp
  case ELF::R_X86_64_GOTPLT64:
  case ELF::R_X86_64_GOTPC32_TLSDESC:
  case ELF::R_X86_64_GOTPCRELX:
  case ELF::R_X86_64_REX_GOTPCRELX:
    return true;
  }
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 547-562

```cpp
static bool isGOTAArch64(uint32_t Type) {
  switch (Type) {
  default:
    return false;
  case ELF::R_AARCH64_ADR_GOT_PAGE:
  case ELF::R_AARCH64_LD64_GOT_LO12_NC:
  case ELF::R_AARCH64_TLSIE_LD64_GOTTPREL_LO12_NC:
  case ELF::R_AARCH64_TLSIE_ADR_GOTTPREL_PAGE21:
  case ELF::R_AARCH64_TLSDESC_ADR_PREL21:
  case ELF::R_AARCH64_TLSDESC_ADR_PAGE21:
  case ELF::R_AARCH64_TLSDESC_LD64_LO12:
  case ELF::R_AARCH64_TLSDESC_ADD_LO12:
    return true;
  }
}
```

- EN: Declares or implements routines including `isGOTAArch64`. Notable symbols here include `isGOTAArch64`.
- CN: 这里声明或实现函数，例如 `isGOTAArch64`。这里较值得关注的符号包括 `isGOTAArch64`。

### Lines 563-572

```cpp
static bool isGOTRISCV(uint32_t Type) {
  switch (Type) {
  default:
    return false;
  case ELF::R_RISCV_GOT_HI20:
  case ELF::R_RISCV_TLS_GOT_HI20:
    return true;
  }
}
```

- EN: Declares or implements routines including `isGOTRISCV`. Notable symbols here include `isGOTRISCV`.
- CN: 这里声明或实现函数，例如 `isGOTRISCV`。这里较值得关注的符号包括 `isGOTRISCV`。

### Lines 573-583

```cpp
static bool isTLSX86(uint32_t Type) {
  switch (Type) {
  default:
    return false;
  case ELF::R_X86_64_TPOFF32:
  case ELF::R_X86_64_TPOFF64:
  case ELF::R_X86_64_GOTTPOFF:
    return true;
  }
}
```

- EN: Declares or implements routines including `isTLSX86`. Notable symbols here include `isTLSX86`.
- CN: 这里声明或实现函数，例如 `isTLSX86`。这里较值得关注的符号包括 `isTLSX86`。

### Lines 584-601

```cpp
static bool isTLSAArch64(uint32_t Type) {
  switch (Type) {
  default:
    return false;
  case ELF::R_AARCH64_TLSDESC_ADR_PREL21:
  case ELF::R_AARCH64_TLSDESC_ADR_PAGE21:
  case ELF::R_AARCH64_TLSIE_LD64_GOTTPREL_LO12_NC:
  case ELF::R_AARCH64_TLSLE_ADD_TPREL_HI12:
  case ELF::R_AARCH64_TLSLE_ADD_TPREL_LO12_NC:
  case ELF::R_AARCH64_TLSLE_MOVW_TPREL_G0:
  case ELF::R_AARCH64_TLSLE_MOVW_TPREL_G0_NC:
  case ELF::R_AARCH64_TLSDESC_LD64_LO12:
  case ELF::R_AARCH64_TLSDESC_ADD_LO12:
  case ELF::R_AARCH64_TLSIE_ADR_GOTTPREL_PAGE21:
    return true;
  }
}
```

- EN: Declares or implements routines including `isTLSAArch64`. Notable symbols here include `isTLSAArch64`.
- CN: 这里声明或实现函数，例如 `isTLSAArch64`。这里较值得关注的符号包括 `isTLSAArch64`。

### Lines 602-616

```cpp
static bool isTLSRISCV(uint32_t Type) {
  switch (Type) {
  default:
    return false;
  case ELF::R_RISCV_TLS_GOT_HI20:
  case ELF::R_RISCV_TPREL_HI20:
  case ELF::R_RISCV_TPREL_ADD:
  case ELF::R_RISCV_TPREL_LO12_I:
  case ELF::R_RISCV_TPREL_LO12_S:
  case ELFReserved::R_RISCV_TPREL_I:
  case ELFReserved::R_RISCV_TPREL_S:
    return true;
  }
}
```

- EN: Declares or implements routines including `isTLSRISCV`. Notable symbols here include `isTLSRISCV`.
- CN: 这里声明或实现函数，例如 `isTLSRISCV`。这里较值得关注的符号包括 `isTLSRISCV`。

### Lines 617-634

```cpp
static bool isPCRelativeX86(uint32_t Type) {
  switch (Type) {
  default:
    llvm_unreachable("Unknown relocation type");
  case ELF::R_X86_64_64:
  case ELF::R_X86_64_32:
  case ELF::R_X86_64_32S:
  case ELF::R_X86_64_16:
  case ELF::R_X86_64_8:
  case ELF::R_X86_64_TPOFF32:
    return false;
  case ELF::R_X86_64_PC8:
  case ELF::R_X86_64_PC32:
  case ELF::R_X86_64_PC64:
  case ELF::R_X86_64_GOTPCREL:
  case ELF::R_X86_64_PLT32:
  case ELF::R_X86_64_GOTOFF64:
  case ELF::R_X86_64_GOTPC32:
```

- EN: Declares or implements routines including `isPCRelativeX86`, `llvm_unreachable`. Notable symbols here include `isPCRelativeX86`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `isPCRelativeX86`, `llvm_unreachable`。这里较值得关注的符号包括 `isPCRelativeX86`, `llvm_unreachable`。

### Lines 635-642

```cpp
  case ELF::R_X86_64_GOTPC64:
  case ELF::R_X86_64_GOTTPOFF:
  case ELF::R_X86_64_GOTPCRELX:
  case ELF::R_X86_64_REX_GOTPCRELX:
    return true;
  }
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 643-660

```cpp
static bool isPCRelativeAArch64(uint32_t Type) {
  switch (Type) {
  default:
    llvm_unreachable("Unknown relocation type");
  case ELF::R_AARCH64_ABS16:
  case ELF::R_AARCH64_ABS32:
  case ELF::R_AARCH64_ABS64:
  case ELF::R_AARCH64_LDST64_ABS_LO12_NC:
  case ELF::R_AARCH64_ADD_ABS_LO12_NC:
  case ELF::R_AARCH64_LDST128_ABS_LO12_NC:
  case ELF::R_AARCH64_LDST32_ABS_LO12_NC:
  case ELF::R_AARCH64_LDST16_ABS_LO12_NC:
  case ELF::R_AARCH64_LDST8_ABS_LO12_NC:
  case ELF::R_AARCH64_TLSIE_LD64_GOTTPREL_LO12_NC:
  case ELF::R_AARCH64_TLSLE_ADD_TPREL_HI12:
  case ELF::R_AARCH64_TLSLE_ADD_TPREL_LO12_NC:
  case ELF::R_AARCH64_TLSLE_MOVW_TPREL_G0:
  case ELF::R_AARCH64_TLSLE_MOVW_TPREL_G0_NC:
```

- EN: Declares or implements routines including `isPCRelativeAArch64`, `llvm_unreachable`. Notable symbols here include `isPCRelativeAArch64`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `isPCRelativeAArch64`, `llvm_unreachable`。这里较值得关注的符号包括 `isPCRelativeAArch64`, `llvm_unreachable`。

### Lines 661-678

```cpp
  case ELF::R_AARCH64_LD64_GOT_LO12_NC:
  case ELF::R_AARCH64_TLSDESC_LD64_LO12:
  case ELF::R_AARCH64_TLSDESC_ADD_LO12:
  case ELF::R_AARCH64_MOVW_UABS_G0:
  case ELF::R_AARCH64_MOVW_UABS_G0_NC:
  case ELF::R_AARCH64_MOVW_UABS_G1:
  case ELF::R_AARCH64_MOVW_UABS_G1_NC:
  case ELF::R_AARCH64_MOVW_UABS_G2:
  case ELF::R_AARCH64_MOVW_UABS_G2_NC:
  case ELF::R_AARCH64_MOVW_UABS_G3:
    return false;
  case ELF::R_AARCH64_CALL26:
  case ELF::R_AARCH64_JUMP26:
  case ELF::R_AARCH64_TSTBR14:
  case ELF::R_AARCH64_CONDBR19:
  case ELF::R_AARCH64_ADR_PREL_LO21:
  case ELF::R_AARCH64_ADR_PREL_PG_HI21:
  case ELF::R_AARCH64_ADR_PREL_PG_HI21_NC:
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 679-690

```cpp
  case ELF::R_AARCH64_ADR_GOT_PAGE:
  case ELF::R_AARCH64_TLSIE_ADR_GOTTPREL_PAGE21:
  case ELF::R_AARCH64_TLSDESC_ADR_PREL21:
  case ELF::R_AARCH64_TLSDESC_ADR_PAGE21:
  case ELF::R_AARCH64_PREL16:
  case ELF::R_AARCH64_PREL32:
  case ELF::R_AARCH64_PREL64:
  case ELF::R_AARCH64_PLT32:
    return true;
  }
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 691-708

```cpp
static bool isPCRelativeRISCV(uint32_t Type) {
  switch (Type) {
  default:
    llvm_unreachable("Unknown relocation type");
  case ELF::R_RISCV_ADD32:
  case ELF::R_RISCV_SUB32:
  case ELF::R_RISCV_HI20:
  case ELF::R_RISCV_LO12_I:
  case ELF::R_RISCV_LO12_S:
  case ELF::R_RISCV_64:
    return false;
  case ELF::R_RISCV_JAL:
  case ELF::R_RISCV_CALL:
  case ELF::R_RISCV_CALL_PLT:
  case ELF::R_RISCV_BRANCH:
  case ELF::R_RISCV_GOT_HI20:
  case ELF::R_RISCV_PCREL_HI20:
  case ELF::R_RISCV_PCREL_LO12_I:
```

- EN: Declares or implements routines including `isPCRelativeRISCV`, `llvm_unreachable`. Notable symbols here include `isPCRelativeRISCV`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `isPCRelativeRISCV`, `llvm_unreachable`。这里较值得关注的符号包括 `isPCRelativeRISCV`, `llvm_unreachable`。

### Lines 709-718

```cpp
  case ELF::R_RISCV_PCREL_LO12_S:
  case ELF::R_RISCV_RVC_JUMP:
  case ELF::R_RISCV_RVC_BRANCH:
  case ELF::R_RISCV_32_PCREL:
  case ELF::R_RISCV_TLS_GOT_HI20:
  case ELF::R_RISCV_TLS_GD_HI20:
    return true;
  }
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 719-731

```cpp
bool Relocation::isSupported(uint32_t Type) {
  switch (Arch) {
  default:
    return false;
  case Triple::aarch64:
    return isSupportedAArch64(Type);
  case Triple::riscv64:
    return isSupportedRISCV(Type);
  case Triple::x86_64:
    return isSupportedX86(Type);
  }
}
```

- EN: Declares or implements routines including `isSupported`. Notable symbols here include `isSupported`.
- CN: 这里声明或实现函数，例如 `isSupported`。这里较值得关注的符号包括 `isSupported`。

### Lines 732-744

```cpp
size_t Relocation::getSizeForType(uint32_t Type) {
  switch (Arch) {
  default:
    llvm_unreachable("Unsupported architecture");
  case Triple::aarch64:
    return getSizeForTypeAArch64(Type);
  case Triple::riscv64:
    return getSizeForTypeRISCV(Type);
  case Triple::x86_64:
    return getSizeForTypeX86(Type);
  }
}
```

- EN: Declares or implements routines including `getSizeForType`, `llvm_unreachable`. Notable symbols here include `getSizeForType`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `getSizeForType`, `llvm_unreachable`。这里较值得关注的符号包括 `getSizeForType`, `llvm_unreachable`。

### Lines 745-757

```cpp
bool Relocation::skipRelocationType(uint32_t Type) {
  switch (Arch) {
  default:
    llvm_unreachable("Unsupported architecture");
  case Triple::aarch64:
    return skipRelocationTypeAArch64(Type);
  case Triple::riscv64:
    return skipRelocationTypeRISCV(Type);
  case Triple::x86_64:
    return skipRelocationTypeX86(Type);
  }
}
```

- EN: Declares or implements routines including `skipRelocationType`, `llvm_unreachable`. Notable symbols here include `skipRelocationType`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `skipRelocationType`, `llvm_unreachable`。这里较值得关注的符号包括 `skipRelocationType`, `llvm_unreachable`。

### Lines 758-770

```cpp
uint64_t Relocation::encodeValue(uint32_t Type, uint64_t Value, uint64_t PC) {
  switch (Arch) {
  default:
    llvm_unreachable("Unsupported architecture");
  case Triple::aarch64:
    return encodeValueAArch64(Type, Value, PC);
  case Triple::riscv64:
    return encodeValueRISCV(Type, Value, PC);
  case Triple::x86_64:
    return encodeValueX86(Type, Value, PC);
  }
}
```

- EN: Declares or implements routines including `encodeValue`, `llvm_unreachable`. Notable symbols here include `encodeValue`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `encodeValue`, `llvm_unreachable`。这里较值得关注的符号包括 `encodeValue`, `llvm_unreachable`。

### Lines 771-783

```cpp
bool Relocation::canEncodeValue(uint32_t Type, uint64_t Value, uint64_t PC) {
  switch (Arch) {
  default:
    llvm_unreachable("Unsupported architecture");
  case Triple::aarch64:
    return canEncodeValueAArch64(Type, Value, PC);
  case Triple::riscv64:
    return canEncodeValueRISCV(Type, Value, PC);
  case Triple::x86_64:
    return true;
  }
}
```

- EN: Declares or implements routines including `canEncodeValue`, `llvm_unreachable`. Notable symbols here include `canEncodeValue`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `canEncodeValue`, `llvm_unreachable`。这里较值得关注的符号包括 `canEncodeValue`, `llvm_unreachable`。

### Lines 784-797

```cpp
uint64_t Relocation::extractValue(uint32_t Type, uint64_t Contents,
                                  uint64_t PC) {
  switch (Arch) {
  default:
    llvm_unreachable("Unsupported architecture");
  case Triple::aarch64:
    return extractValueAArch64(Type, Contents, PC);
  case Triple::riscv64:
    return extractValueRISCV(Type, Contents, PC);
  case Triple::x86_64:
    return extractValueX86(Type, Contents, PC);
  }
}
```

- EN: Declares or implements routines including `llvm_unreachable`. Notable symbols here include `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`。这里较值得关注的符号包括 `llvm_unreachable`。

### Lines 798-810

```cpp
bool Relocation::isGOT(uint32_t Type) {
  switch (Arch) {
  default:
    llvm_unreachable("Unsupported architecture");
  case Triple::aarch64:
    return isGOTAArch64(Type);
  case Triple::riscv64:
    return isGOTRISCV(Type);
  case Triple::x86_64:
    return isGOTX86(Type);
  }
}
```

- EN: Declares or implements routines including `isGOT`, `llvm_unreachable`. Notable symbols here include `isGOT`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `isGOT`, `llvm_unreachable`。这里较值得关注的符号包括 `isGOT`, `llvm_unreachable`。

### Lines 811-822

```cpp
bool Relocation::isX86GOTPCRELX(uint32_t Type) {
  if (Arch != Triple::x86_64)
    return false;
  return Type == ELF::R_X86_64_GOTPCRELX || Type == ELF::R_X86_64_REX_GOTPCRELX;
}

bool Relocation::isX86GOTPC64(uint32_t Type) {
  if (Arch != Triple::x86_64)
    return false;
  return Type == ELF::R_X86_64_GOTPC64;
}
```

- EN: Declares or implements routines including `isX86GOTPCRELX`, `isX86GOTPC64`. Notable symbols here include `isX86GOTPCRELX`, `isX86GOTPC64`.
- CN: 这里声明或实现函数，例如 `isX86GOTPCRELX`, `isX86GOTPC64`。这里较值得关注的符号包括 `isX86GOTPCRELX`, `isX86GOTPC64`。

### Lines 823-837

```cpp
bool Relocation::isNone(uint32_t Type) { return Type == getNone(); }

bool Relocation::isRelative(uint32_t Type) {
  switch (Arch) {
  default:
    llvm_unreachable("Unsupported architecture");
  case Triple::aarch64:
    return Type == ELF::R_AARCH64_RELATIVE;
  case Triple::riscv64:
    return Type == ELF::R_RISCV_RELATIVE;
  case Triple::x86_64:
    return Type == ELF::R_X86_64_RELATIVE;
  }
}
```

- EN: Declares or implements routines including `isNone`, `isRelative`, `llvm_unreachable`. Notable symbols here include `isNone`, `isRelative`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `isNone`, `isRelative`, `llvm_unreachable`。这里较值得关注的符号包括 `isNone`, `isRelative`, `llvm_unreachable`。

### Lines 838-850

```cpp
bool Relocation::isIRelative(uint32_t Type) {
  switch (Arch) {
  default:
    llvm_unreachable("Unsupported architecture");
  case Triple::aarch64:
    return Type == ELF::R_AARCH64_IRELATIVE;
  case Triple::riscv64:
    llvm_unreachable("not implemented");
  case Triple::x86_64:
    return Type == ELF::R_X86_64_IRELATIVE;
  }
}
```

- EN: Declares or implements routines including `isIRelative`, `llvm_unreachable`. Notable symbols here include `isIRelative`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `isIRelative`, `llvm_unreachable`。这里较值得关注的符号包括 `isIRelative`, `llvm_unreachable`。

### Lines 851-863

```cpp
bool Relocation::isTLS(uint32_t Type) {
  switch (Arch) {
  default:
    llvm_unreachable("Unsupported architecture");
  case Triple::aarch64:
    return isTLSAArch64(Type);
  case Triple::riscv64:
    return isTLSRISCV(Type);
  case Triple::x86_64:
    return isTLSX86(Type);
  }
}
```

- EN: Declares or implements routines including `isTLS`, `llvm_unreachable`. Notable symbols here include `isTLS`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `isTLS`, `llvm_unreachable`。这里较值得关注的符号包括 `isTLS`, `llvm_unreachable`。

### Lines 864-876

```cpp
bool Relocation::isInstructionReference(uint32_t Type) {
  if (Arch != Triple::riscv64)
    return false;

  switch (Type) {
  default:
    return false;
  case ELF::R_RISCV_PCREL_LO12_I:
  case ELF::R_RISCV_PCREL_LO12_S:
    return true;
  }
}
```

- EN: Declares or implements routines including `isInstructionReference`. Notable symbols here include `isInstructionReference`.
- CN: 这里声明或实现函数，例如 `isInstructionReference`。这里较值得关注的符号包括 `isInstructionReference`。

### Lines 877-889

```cpp
uint32_t Relocation::getNone() {
  switch (Arch) {
  default:
    llvm_unreachable("Unsupported architecture");
  case Triple::aarch64:
    return ELF::R_AARCH64_NONE;
  case Triple::riscv64:
    return ELF::R_RISCV_NONE;
  case Triple::x86_64:
    return ELF::R_X86_64_NONE;
  }
}
```

- EN: Declares or implements routines including `getNone`, `llvm_unreachable`. Notable symbols here include `getNone`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `getNone`, `llvm_unreachable`。这里较值得关注的符号包括 `getNone`, `llvm_unreachable`。

### Lines 890-902

```cpp
uint32_t Relocation::getPC32() {
  switch (Arch) {
  default:
    llvm_unreachable("Unsupported architecture");
  case Triple::aarch64:
    return ELF::R_AARCH64_PREL32;
  case Triple::riscv64:
    return ELF::R_RISCV_32_PCREL;
  case Triple::x86_64:
    return ELF::R_X86_64_PC32;
  }
}
```

- EN: Declares or implements routines including `getPC32`, `llvm_unreachable`. Notable symbols here include `getPC32`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `getPC32`, `llvm_unreachable`。这里较值得关注的符号包括 `getPC32`, `llvm_unreachable`。

### Lines 903-915

```cpp
uint32_t Relocation::getPC64() {
  switch (Arch) {
  default:
    llvm_unreachable("Unsupported architecture");
  case Triple::aarch64:
    return ELF::R_AARCH64_PREL64;
  case Triple::riscv64:
    llvm_unreachable("not implemented");
  case Triple::x86_64:
    return ELF::R_X86_64_PC64;
  }
}
```

- EN: Declares or implements routines including `getPC64`, `llvm_unreachable`. Notable symbols here include `getPC64`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `getPC64`, `llvm_unreachable`。这里较值得关注的符号包括 `getPC64`, `llvm_unreachable`。

### Lines 916-933

```cpp
uint32_t Relocation::getType(const object::RelocationRef &Rel) {
  uint64_t RelType = Rel.getType();
  assert(isUInt<32>(RelType) && "BOLT relocation types are 32 bits");
  return static_cast<uint32_t>(RelType);
}

bool Relocation::isPCRelative(uint32_t Type) {
  switch (Arch) {
  default:
    llvm_unreachable("Unsupported architecture");
  case Triple::aarch64:
    return isPCRelativeAArch64(Type);
  case Triple::riscv64:
    return isPCRelativeRISCV(Type);
  case Triple::x86_64:
    return isPCRelativeX86(Type);
  }
}
```

- EN: Declares or implements routines including `getType`, `assert`, `isPCRelative`, `llvm_unreachable`. Notable symbols here include `getType`, `assert`, `isPCRelative`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `getType`, `assert`, `isPCRelative`, `llvm_unreachable`。这里较值得关注的符号包括 `getType`, `assert`, `isPCRelative`, `llvm_unreachable`。

### Lines 934-947

```cpp

uint32_t Relocation::getAbs64() {
  switch (Arch) {
  default:
    llvm_unreachable("Unsupported architecture");
  case Triple::aarch64:
    return ELF::R_AARCH64_ABS64;
  case Triple::riscv64:
    return ELF::R_RISCV_64;
  case Triple::x86_64:
    return ELF::R_X86_64_64;
  }
}
```

- EN: Declares or implements routines including `getAbs64`, `llvm_unreachable`. Notable symbols here include `getAbs64`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `getAbs64`, `llvm_unreachable`。这里较值得关注的符号包括 `getAbs64`, `llvm_unreachable`。

### Lines 948-960

```cpp
uint32_t Relocation::getRelative() {
  switch (Arch) {
  default:
    llvm_unreachable("Unsupported architecture");
  case Triple::aarch64:
    return ELF::R_AARCH64_RELATIVE;
  case Triple::riscv64:
    llvm_unreachable("not implemented");
  case Triple::x86_64:
    return ELF::R_X86_64_RELATIVE;
  }
}
```

- EN: Declares or implements routines including `getRelative`, `llvm_unreachable`. Notable symbols here include `getRelative`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `getRelative`, `llvm_unreachable`。这里较值得关注的符号包括 `getRelative`, `llvm_unreachable`。

### Lines 961-971

```cpp
size_t Relocation::emit(MCStreamer *Streamer) const {
  const size_t Size = getSizeForType(Type);
  const auto *Value = createExpr(Streamer);
  Streamer->emitValue(Value, Size);
  return Size;
}

const MCExpr *Relocation::createExpr(MCStreamer *Streamer) const {
  MCContext &Ctx = Streamer->getContext();
  const MCExpr *Value = nullptr;
```

- EN: Declares or implements routines including `emit`, `getSizeForType`, `createExpr`, `emitValue`, `getContext`. Notable symbols here include `emit`, `getSizeForType`, `createExpr`, `emitValue`, `getContext`.
- CN: 这里声明或实现函数，例如 `emit`, `getSizeForType`, `createExpr`, `emitValue`, `getContext`。这里较值得关注的符号包括 `emit`, `getSizeForType`, `createExpr`, `emitValue`, `getContext`。

### Lines 972-980

```cpp
  if (Symbol && Addend) {
    Value = MCBinaryExpr::createAdd(MCSymbolRefExpr::create(Symbol, Ctx),
                                    MCConstantExpr::create(Addend, Ctx), Ctx);
  } else if (Symbol) {
    Value = MCSymbolRefExpr::create(Symbol, Ctx);
  } else {
    Value = MCConstantExpr::create(Addend, Ctx);
  }
```

- EN: Declares or implements routines including `createAdd`, `create`, `if`. Notable symbols here include `createAdd`, `create`, `if`.
- CN: 这里声明或实现函数，例如 `createAdd`, `create`, `if`。这里较值得关注的符号包括 `createAdd`, `create`, `if`。

### Lines 981-990

```cpp
  if (isPCRelative(Type)) {
    MCSymbol *TempLabel = Ctx.createNamedTempSymbol();
    Streamer->emitLabel(TempLabel);
    Value = MCBinaryExpr::createSub(
        Value, MCSymbolRefExpr::create(TempLabel, Ctx), Ctx);
  }

  return Value;
}
```

- EN: Declares or implements routines including `emitLabel`, `create`. Notable symbols here include `emitLabel`, `create`.
- CN: 这里声明或实现函数，例如 `emitLabel`, `create`。这里较值得关注的符号包括 `emitLabel`, `create`。

### Lines 991-999

```cpp
const MCExpr *Relocation::createExpr(MCStreamer *Streamer,
                                     const MCExpr *RetainedValue) const {
  const auto *Value = createExpr(Streamer);

  if (RetainedValue) {
    Value = MCBinaryExpr::create(getComposeOpcodeFor(Type), RetainedValue,
                                 Value, Streamer->getContext());
  }
```

- EN: Declares or implements routines including `createExpr`, `create`, `getContext`. Notable symbols here include `createExpr`, `create`, `getContext`.
- CN: 这里声明或实现函数，例如 `createExpr`, `create`, `getContext`。这里较值得关注的符号包括 `createExpr`, `create`, `getContext`。

### Lines 1000-1015

```cpp
  return Value;
}

MCBinaryExpr::Opcode Relocation::getComposeOpcodeFor(uint32_t Type) {
  assert(Arch == Triple::riscv64 && "only implemented for RISC-V");

  switch (Type) {
  default:
    llvm_unreachable("not implemented");
  case ELF::R_RISCV_ADD32:
    return MCBinaryExpr::Add;
  case ELF::R_RISCV_SUB32:
    return MCBinaryExpr::Sub;
  }
}
```

- EN: Declares or implements routines including `getComposeOpcodeFor`, `assert`, `llvm_unreachable`. Notable symbols here include `getComposeOpcodeFor`, `assert`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `getComposeOpcodeFor`, `assert`, `llvm_unreachable`。这里较值得关注的符号包括 `getComposeOpcodeFor`, `assert`, `llvm_unreachable`。

### Lines 1016-1033

```cpp
void Relocation::print(raw_ostream &OS) const {
  switch (Arch) {
  default:
    OS << "RType:" << Twine::utohexstr(Type);
    break;
  case Triple::aarch64:
    OS << object::getELFRelocationTypeName(ELF::EM_AARCH64, Type);
    break;
  case Triple::riscv64:
    OS << object::getELFRelocationTypeName(ELF::EM_RISCV, Type);
    break;
  case Triple::x86_64:
    OS << object::getELFRelocationTypeName(ELF::EM_X86_64, Type);
    break;
  }
  OS << ", 0x" << Twine::utohexstr(Offset);
  if (Symbol) {
    OS << ", " << Symbol->getName();
```

- EN: Declares or implements routines including `print`, `utohexstr`, `getELFRelocationTypeName`, `getName`. Notable symbols here include `print`, `utohexstr`, `getELFRelocationTypeName`, `getName`.
- CN: 这里声明或实现函数，例如 `print`, `utohexstr`, `getELFRelocationTypeName`, `getName`。这里较值得关注的符号包括 `print`, `utohexstr`, `getELFRelocationTypeName`, `getName`。

### Lines 1034-1040

```cpp
  }
  if (int64_t(Addend) < 0)
    OS << ", -0x" << Twine::utohexstr(-int64_t(Addend));
  else
    OS << ", 0x" << Twine::utohexstr(Addend);
  OS << ", 0x" << Twine::utohexstr(Value);
}
```

- EN: Declares or implements routines including `utohexstr`. Notable symbols here include `utohexstr`.
- CN: 这里声明或实现函数，例如 `utohexstr`。这里较值得关注的符号包括 `utohexstr`。

## Key Concepts / 关键概念

- `isSupportedX86`: function or method entry point / 函数或方法入口
- `isSupportedAArch64`: function or method entry point / 函数或方法入口
- `isSupportedRISCV`: function or method entry point / 函数或方法入口
- `getSizeForTypeX86`: function or method entry point / 函数或方法入口
- `errs`: function or method entry point / 函数或方法入口
- `llvm`: namespace scope / 命名空间作用域
- `bolt`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Core/Relocation.h`
- LLVM headers / LLVM 头文件: `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCStreamer.h`, `llvm/MC/MCSymbol.h`, `llvm/Object/ELF.h`, `llvm/Object/ObjectFile.h`
- Directory context / 目录上下文: `bolt/lib/Core` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Core` 下的相邻文件通常与本文件协作组成对应子系统
