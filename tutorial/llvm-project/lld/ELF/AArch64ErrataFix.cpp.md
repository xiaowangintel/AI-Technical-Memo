# AArch64ErrataFix.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `lld/ELF/AArch64ErrataFix.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: AArch64ErrataFix.cpp This file implements Section Patching for the purpose of working around the AArch64 Cortex-53 errata 843419 that affects r0p0, r0p1, r0p2 and r0p4 versions of the core.. It also sits in code that implements ELF linking components in LLD. / 该文件实现 LLD 中的 ELF 链接组件。 源码头部说明其职责是：AArch64ErrataFix.cpp This file implements Section Patching for the purpose of working around the AArch64 Cortex-53 errata 843419 that affects r0p0, r0p1, r0p2 and r0p4 versions of the core.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17

```cpp
//===- AArch64ErrataFix.cpp -----------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
// This file implements Section Patching for the purpose of working around
// the AArch64 Cortex-53 errata 843419 that affects r0p0, r0p1, r0p2 and r0p4
// versions of the core.
//
// The general principle is that an erratum sequence of one or
// more instructions is detected in the instruction stream, one of the
// instructions in the sequence is replaced with a branch to a patch sequence
// of replacement instructions. At the end of the replacement sequence the
// patch branches back to the instruction stream.
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 18-27

```cpp
// This technique is only suitable for fixing an erratum when:
// - There is a set of necessary conditions required to trigger the erratum that
// can be detected at static link time.
// - There is a set of replacement instructions that can be used to remove at
// least one of the necessary conditions that trigger the erratum.
// - We can overwrite an instruction in the erratum sequence with a branch to
// the replacement sequence.
// - We can place the replacement sequence within range of the branch.
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 28-39

```cpp
#include "AArch64ErrataFix.h"
#include "InputFiles.h"
#include "LinkerScript.h"
#include "OutputSections.h"
#include "Relocations.h"
#include "Symbols.h"
#include "SyntheticSections.h"
#include "Target.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/Support/Endian.h"
#include <algorithm>
```

- EN: Pulls in 11 header(s) from LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 11 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 40-47

```cpp
using namespace llvm;
using namespace llvm::ELF;
using namespace llvm::object;
using namespace llvm::support;
using namespace llvm::support::endian;
using namespace lld;
using namespace lld::elf;
```

- EN: Works inside namespace scope `llvm`, `lld` to organize symbols. Notable symbols here include `llvm`, `lld`.
- CN: 这里位于命名空间 `llvm`, `lld` 中，用于组织符号作用域。这里较值得关注的符号包括 `llvm`, `lld`。

### Lines 48-56

```cpp
// Helper functions to identify instructions and conditions needed to trigger
// the Cortex-A53-843419 erratum.

// ADRP
// | 1 | immlo (2) | 1 | 0 0 0 0 | immhi (19) | Rd (5) |
static bool isADRP(uint32_t instr) {
  return (instr & 0x9f000000) == 0x90000000;
}
```

- EN: Declares or implements routines including `isADRP`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isADRP`.
- CN: 这里声明或实现函数，例如 `isADRP`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isADRP`。

### Lines 57-66

```cpp
// Load and store bit patterns from ARMv8-A.
// Instructions appear in order of appearance starting from table in
// C4.1.3 Loads and Stores.

// All loads and stores have 1 (at bit position 27), (0 at bit position 25).
// | op0 x op1 (2) | 1 op2 0 op3 (2) | x | op4 (5) | xxxx | op5 (2) | x (10) |
static bool isLoadStoreClass(uint32_t instr) {
  return (instr & 0x0a000000) == 0x08000000;
}
```

- EN: Declares or implements routines including `isLoadStoreClass`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isLoadStoreClass`.
- CN: 这里声明或实现函数，例如 `isLoadStoreClass`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isLoadStoreClass`。

### Lines 67-84

```cpp
// LDN/STN multiple no offset
// | 0 Q 00 | 1100 | 0 L 00 | 0000 | opcode (4) | size (2) | Rn (5) | Rt (5) |
// LDN/STN multiple post-indexed
// | 0 Q 00 | 1100 | 1 L 0 | Rm (5)| opcode (4) | size (2) | Rn (5) | Rt (5) |
// L == 0 for stores.

// Utility routine to decode opcode field of LDN/STN multiple structure
// instructions to find the ST1 instructions.
// opcode == 0010 ST1 4 registers.
// opcode == 0110 ST1 3 registers.
// opcode == 0111 ST1 1 register.
// opcode == 1010 ST1 2 registers.
static bool isST1MultipleOpcode(uint32_t instr) {
  return (instr & 0x0000f000) == 0x00002000 ||
         (instr & 0x0000f000) == 0x00006000 ||
         (instr & 0x0000f000) == 0x00007000 ||
         (instr & 0x0000f000) == 0x0000a000;
}
```

- EN: Declares or implements routines including `isST1MultipleOpcode`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isST1MultipleOpcode`.
- CN: 这里声明或实现函数，例如 `isST1MultipleOpcode`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isST1MultipleOpcode`。

### Lines 85-94

```cpp

static bool isST1Multiple(uint32_t instr) {
  return (instr & 0xbfff0000) == 0x0c000000 && isST1MultipleOpcode(instr);
}

// Writes to Rn (writeback).
static bool isST1MultiplePost(uint32_t instr) {
  return (instr & 0xbfe00000) == 0x0c800000 && isST1MultipleOpcode(instr);
}
```

- EN: Declares or implements routines including `isST1Multiple`, `isST1MultiplePost`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isST1Multiple`, `isST1MultiplePost`.
- CN: 这里声明或实现函数，例如 `isST1Multiple`, `isST1MultiplePost`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isST1Multiple`, `isST1MultiplePost`。

### Lines 95-112

```cpp
// LDN/STN single no offset
// | 0 Q 00 | 1101 | 0 L R 0 | 0000 | opc (3) S | size (2) | Rn (5) | Rt (5)|
// LDN/STN single post-indexed
// | 0 Q 00 | 1101 | 1 L R | Rm (5) | opc (3) S | size (2) | Rn (5) | Rt (5)|
// L == 0 for stores

// Utility routine to decode opcode field of LDN/STN single structure
// instructions to find the ST1 instructions.
// R == 0 for ST1 and ST3, R == 1 for ST2 and ST4.
// opcode == 000 ST1 8-bit.
// opcode == 010 ST1 16-bit.
// opcode == 100 ST1 32 or 64-bit (Size determines which).
static bool isST1SingleOpcode(uint32_t instr) {
  return (instr & 0x0040e000) == 0x00000000 ||
         (instr & 0x0040e000) == 0x00004000 ||
         (instr & 0x0040e000) == 0x00008000;
}
```

- EN: Declares or implements routines including `isST1SingleOpcode`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isST1SingleOpcode`.
- CN: 这里声明或实现函数，例如 `isST1SingleOpcode`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isST1SingleOpcode`。

### Lines 113-121

```cpp
static bool isST1Single(uint32_t instr) {
  return (instr & 0xbfff0000) == 0x0d000000 && isST1SingleOpcode(instr);
}

// Writes to Rn (writeback).
static bool isST1SinglePost(uint32_t instr) {
  return (instr & 0xbfe00000) == 0x0d800000 && isST1SingleOpcode(instr);
}
```

- EN: Declares or implements routines including `isST1Single`, `isST1SinglePost`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isST1Single`, `isST1SinglePost`.
- CN: 这里声明或实现函数，例如 `isST1Single`, `isST1SinglePost`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isST1Single`, `isST1SinglePost`。

### Lines 122-133

```cpp
static bool isST1(uint32_t instr) {
  return isST1Multiple(instr) || isST1MultiplePost(instr) ||
         isST1Single(instr) || isST1SinglePost(instr);
}

// Load/store exclusive
// | size (2) 00 | 1000 | o2 L o1 | Rs (5) | o0 | Rt2 (5) | Rn (5) | Rt (5) |
// L == 0 for Stores.
static bool isLoadStoreExclusive(uint32_t instr) {
  return (instr & 0x3f000000) == 0x08000000;
}
```

- EN: Declares or implements routines including `isST1`, `isST1Single`, `isLoadStoreExclusive`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isST1`, `isST1Single`, `isLoadStoreExclusive`.
- CN: 这里声明或实现函数，例如 `isST1`, `isST1Single`, `isLoadStoreExclusive`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isST1`, `isST1Single`, `isLoadStoreExclusive`。

### Lines 134-143

```cpp
static bool isLoadExclusive(uint32_t instr) {
  return (instr & 0x3f400000) == 0x08400000;
}

// Load register literal
// | opc (2) 01 | 1 V 00 | imm19 | Rt (5) |
static bool isLoadLiteral(uint32_t instr) {
  return (instr & 0x3b000000) == 0x18000000;
}
```

- EN: Declares or implements routines including `isLoadExclusive`, `isLoadLiteral`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isLoadExclusive`, `isLoadLiteral`.
- CN: 这里声明或实现函数，例如 `isLoadExclusive`, `isLoadLiteral`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isLoadExclusive`, `isLoadLiteral`。

### Lines 144-152

```cpp
// Load/store no-allocate pair
// (offset)
// | opc (2) 10 | 1 V 00 | 0 L | imm7 | Rt2 (5) | Rn (5) | Rt (5) |
// L == 0 for stores.
// Never writes to register
static bool isSTNP(uint32_t instr) {
  return (instr & 0x3bc00000) == 0x28000000;
}
```

- EN: Declares or implements routines including `isSTNP`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isSTNP`.
- CN: 这里声明或实现函数，例如 `isSTNP`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isSTNP`。

### Lines 153-161

```cpp
// Load/store register pair
// (post-indexed)
// | opc (2) 10 | 1 V 00 | 1 L | imm7 | Rt2 (5) | Rn (5) | Rt (5) |
// L == 0 for stores, V == 0 for Scalar, V == 1 for Simd/FP
// Writes to Rn.
static bool isSTPPost(uint32_t instr) {
  return (instr & 0x3bc00000) == 0x28800000;
}
```

- EN: Declares or implements routines including `isSTPPost`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isSTPPost`.
- CN: 这里声明或实现函数，例如 `isSTPPost`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isSTPPost`。

### Lines 162-174

```cpp
// (offset)
// | opc (2) 10 | 1 V 01 | 0 L | imm7 | Rt2 (5) | Rn (5) | Rt (5) |
static bool isSTPOffset(uint32_t instr) {
  return (instr & 0x3bc00000) == 0x29000000;
}

// (pre-index)
// | opc (2) 10 | 1 V 01 | 1 L | imm7 | Rt2 (5) | Rn (5) | Rt (5) |
// Writes to Rn.
static bool isSTPPre(uint32_t instr) {
  return (instr & 0x3bc00000) == 0x29800000;
}
```

- EN: Declares or implements routines including `isSTPOffset`, `isSTPPre`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isSTPOffset`, `isSTPPre`.
- CN: 这里声明或实现函数，例如 `isSTPOffset`, `isSTPPre`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isSTPOffset`, `isSTPPre`。

### Lines 175-185

```cpp
static bool isSTP(uint32_t instr) {
  return isSTPPost(instr) || isSTPOffset(instr) || isSTPPre(instr);
}

// Load/store register (unscaled immediate)
// | size (2) 11 | 1 V 00 | opc (2) 0 | imm9 | 00 | Rn (5) | Rt (5) |
// V == 0 for Scalar, V == 1 for Simd/FP.
static bool isLoadStoreUnscaled(uint32_t instr) {
  return (instr & 0x3b000c00) == 0x38000000;
}
```

- EN: Declares or implements routines including `isSTP`, `isLoadStoreUnscaled`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isSTP`, `isLoadStoreUnscaled`.
- CN: 这里声明或实现函数，例如 `isSTP`, `isLoadStoreUnscaled`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isSTP`, `isLoadStoreUnscaled`。

### Lines 186-197

```cpp
// Load/store register (immediate post-indexed)
// | size (2) 11 | 1 V 00 | opc (2) 0 | imm9 | 01 | Rn (5) | Rt (5) |
static bool isLoadStoreImmediatePost(uint32_t instr) {
  return (instr & 0x3b200c00) == 0x38000400;
}

// Load/store register (unprivileged)
// | size (2) 11 | 1 V 00 | opc (2) 0 | imm9 | 10 | Rn (5) | Rt (5) |
static bool isLoadStoreUnpriv(uint32_t instr) {
  return (instr & 0x3b200c00) == 0x38000800;
}
```

- EN: Declares or implements routines including `isLoadStoreImmediatePost`, `isLoadStoreUnpriv`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isLoadStoreImmediatePost`, `isLoadStoreUnpriv`.
- CN: 这里声明或实现函数，例如 `isLoadStoreImmediatePost`, `isLoadStoreUnpriv`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isLoadStoreImmediatePost`, `isLoadStoreUnpriv`。

### Lines 198-209

```cpp
// Load/store register (immediate pre-indexed)
// | size (2) 11 | 1 V 00 | opc (2) 0 | imm9 | 11 | Rn (5) | Rt (5) |
static bool isLoadStoreImmediatePre(uint32_t instr) {
  return (instr & 0x3b200c00) == 0x38000c00;
}

// Load/store register (register offset)
// | size (2) 11 | 1 V 00 | opc (2) 1 | Rm (5) | option (3) S | 10 | Rn | Rt |
static bool isLoadStoreRegisterOff(uint32_t instr) {
  return (instr & 0x3b200c00) == 0x38200800;
}
```

- EN: Declares or implements routines including `isLoadStoreImmediatePre`, `isLoadStoreRegisterOff`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isLoadStoreImmediatePre`, `isLoadStoreRegisterOff`.
- CN: 这里声明或实现函数，例如 `isLoadStoreImmediatePre`, `isLoadStoreRegisterOff`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isLoadStoreImmediatePre`, `isLoadStoreRegisterOff`。

### Lines 210-218

```cpp
// Load/store register (unsigned immediate)
// | size (2) 11 | 1 V 01 | opc (2) | imm12 | Rn (5) | Rt (5) |
static bool isLoadStoreRegisterUnsigned(uint32_t instr) {
  return (instr & 0x3b000000) == 0x39000000;
}

// Rt is always in bit position 0 - 4.
static uint32_t getRt(uint32_t instr) { return (instr & 0x1f); }
```

- EN: Declares or implements routines including `isLoadStoreRegisterUnsigned`, `getRt`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isLoadStoreRegisterUnsigned`, `getRt`.
- CN: 这里声明或实现函数，例如 `isLoadStoreRegisterUnsigned`, `getRt`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isLoadStoreRegisterUnsigned`, `getRt`。

### Lines 219-235

```cpp
// Rn is always in bit position 5 - 9.
static uint32_t getRn(uint32_t instr) { return (instr >> 5) & 0x1f; }

// C4.1.2 Branches, Exception Generating and System instructions
// | op0 (3) 1 | 01 op1 (4) | x (22) |
// op0 == 010 101 op1 == 0xxx Conditional Branch.
// op0 == 110 101 op1 == 1xxx Unconditional Branch Register.
// op0 == x00 101 op1 == xxxx Unconditional Branch immediate.
// op0 == x01 101 op1 == 0xxx Compare and branch immediate.
// op0 == x01 101 op1 == 1xxx Test and branch immediate.
static bool isBranch(uint32_t instr) {
  return ((instr & 0xfe000000) == 0xd6000000) || // Cond branch.
         ((instr & 0xfe000000) == 0x54000000) || // Uncond branch reg.
         ((instr & 0x7c000000) == 0x14000000) || // Uncond branch imm.
         ((instr & 0x7c000000) == 0x34000000);   // Compare and test branch.
}
```

- EN: Declares or implements routines including `getRn`, `isBranch`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getRn`, `isBranch`.
- CN: 这里声明或实现函数，例如 `getRn`, `isBranch`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getRn`, `isBranch`。

### Lines 236-253

```cpp
static bool isV8SingleRegisterNonStructureLoadStore(uint32_t instr) {
  return isLoadStoreUnscaled(instr) || isLoadStoreImmediatePost(instr) ||
         isLoadStoreUnpriv(instr) || isLoadStoreImmediatePre(instr) ||
         isLoadStoreRegisterOff(instr) || isLoadStoreRegisterUnsigned(instr);
}

// Note that this function refers to v8.0 only and does not include the
// additional load and store instructions added for in later revisions of
// the architecture such as the Atomic memory operations introduced
// in v8.1.
static bool isV8NonStructureLoad(uint32_t instr) {
  if (isLoadExclusive(instr))
    return true;
  if (isLoadLiteral(instr))
    return true;
  else if (isV8SingleRegisterNonStructureLoadStore(instr)) {
    // For Load and Store single register, Loads are derived from a
    // combination of the Size, V and Opc fields.
```

- EN: Declares or implements routines including `isV8SingleRegisterNonStructureLoadStore`, `isLoadStoreUnpriv`, `isLoadStoreRegisterOff`, `isV8NonStructureLoad`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isV8SingleRegisterNonStructureLoadStore`, `isLoadStoreUnpriv`, `isLoadStoreRegisterOff`, `isV8NonStructureLoad`.
- CN: 这里声明或实现函数，例如 `isV8SingleRegisterNonStructureLoadStore`, `isLoadStoreUnpriv`, `isLoadStoreRegisterOff`, `isV8NonStructureLoad`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isV8SingleRegisterNonStructureLoadStore`, `isLoadStoreUnpriv`, `isLoadStoreRegisterOff`, `isV8NonStructureLoad`。

### Lines 254-267

```cpp
    uint32_t size = (instr >> 30) & 0xff;
    uint32_t v = (instr >> 26) & 0x1;
    uint32_t opc = (instr >> 22) & 0x3;
    // For the load and store instructions that we are decoding.
    // Opc == 0 are all stores.
    // Opc == 1 with a couple of exceptions are loads. The exceptions are:
    // Size == 00 (0), V == 1, Opc == 10 (2) which is a store and
    // Size == 11 (3), V == 0, Opc == 10 (2) which is a prefetch.
    return opc != 0 && !(size == 0 && v == 1 && opc == 2) &&
           !(size == 3 && v == 0 && opc == 2);
  }
  return false;
}
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 268-277

```cpp
// The following decode instructions are only complete up to the instructions
// needed for errata 843419.

// Instruction with writeback updates the index register after the load/store.
static bool hasWriteback(uint32_t instr) {
  return isLoadStoreImmediatePre(instr) || isLoadStoreImmediatePost(instr) ||
         isSTPPre(instr) || isSTPPost(instr) || isST1SinglePost(instr) ||
         isST1MultiplePost(instr);
}
```

- EN: Declares or implements routines including `hasWriteback`, `isSTPPre`, `isST1MultiplePost`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `hasWriteback`, `isSTPPre`, `isST1MultiplePost`.
- CN: 这里声明或实现函数，例如 `hasWriteback`, `isSTPPre`, `isST1MultiplePost`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `hasWriteback`, `isSTPPre`, `isST1MultiplePost`。

### Lines 278-285

```cpp
// For the load and store class of instructions, a load can write to the
// destination register, a load and a store can write to the base register when
// the instruction has writeback.
static bool doesLoadStoreWriteToReg(uint32_t instr, uint32_t reg) {
  return (isV8NonStructureLoad(instr) && getRt(instr) == reg) ||
         (hasWriteback(instr) && getRn(instr) == reg);
}
```

- EN: Introduces type definitions such as `of`. Declares or implements routines including `doesLoadStoreWriteToReg`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `of`, `doesLoadStoreWriteToReg`.
- CN: 这里引入类型定义，例如 `of`。这里声明或实现函数，例如 `doesLoadStoreWriteToReg`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `of`, `doesLoadStoreWriteToReg`。

### Lines 286-303

```cpp
// Scanner for Cortex-A53 errata 843419
// Full details are available in the Cortex A53 MPCore revision 0 Software
// Developers Errata Notice (ARM-EPM-048406).
//
// The instruction sequence that triggers the erratum is common in compiled
// AArch64 code, however it is sensitive to the offset of the sequence within
// a 4k page. This means that by scanning and fixing the patch after we have
// assigned addresses we only need to disassemble and fix instances of the
// sequence in the range of affected offsets.
//
// In summary the erratum conditions are a series of 4 instructions:
// 1.) An ADRP instruction that writes to register Rn with low 12 bits of
//     address of instruction either 0xff8 or 0xffc.
// 2.) A load or store instruction that can be:
// - A single register load or store, of either integer or vector registers.
// - An STP or STNP, of either integer or vector registers.
// - An Advanced SIMD ST1 store instruction.
// - Must not write to Rn, but may optionally read from it.
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 304-311

```cpp
// 3.) An optional instruction that is not a branch and does not write to Rn.
// 4.) A load or store from the  Load/store register (unsigned immediate) class
//     that uses Rn as the base address register.
//
// Note that we do not attempt to scan for Sequence 2 as described in the
// Software Developers Errata Notice as this has been assessed to be extremely
// unlikely to occur in compiled code. This matches gold and ld.bfd behavior.
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 312-319

```cpp
// Return true if the Instruction sequence Adrp, Instr2, and Instr4 match
// the erratum sequence. The Adrp, Instr2 and Instr4 correspond to 1.), 2.),
// and 4.) in the Scanner for Cortex-A53 errata comment above.
static bool is843419ErratumSequence(uint32_t instr1, uint32_t instr2,
                                    uint32_t instr4) {
  if (!isADRP(instr1))
    return false;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 320-328

```cpp
  uint32_t rn = getRt(instr1);
  return isLoadStoreClass(instr2) &&
         (isLoadStoreExclusive(instr2) || isLoadLiteral(instr2) ||
          isV8SingleRegisterNonStructureLoadStore(instr2) || isSTP(instr2) ||
          isSTNP(instr2) || isST1(instr2)) &&
         !doesLoadStoreWriteToReg(instr2, rn) &&
         isLoadStoreRegisterUnsigned(instr4) && getRn(instr4) == rn;
}
```

- EN: Declares or implements routines including `getRt`, `isV8SingleRegisterNonStructureLoadStore`, `isSTNP`, `isLoadStoreRegisterUnsigned`. Notable symbols here include `getRt`, `isV8SingleRegisterNonStructureLoadStore`, `isSTNP`, `isLoadStoreRegisterUnsigned`.
- CN: 这里声明或实现函数，例如 `getRt`, `isV8SingleRegisterNonStructureLoadStore`, `isSTNP`, `isLoadStoreRegisterUnsigned`。这里较值得关注的符号包括 `getRt`, `isV8SingleRegisterNonStructureLoadStore`, `isSTNP`, `isLoadStoreRegisterUnsigned`。

### Lines 329-338

```cpp
// Scan the instruction sequence starting at Offset Off from the base of
// InputSection isec. We update Off in this function rather than in the caller
// as we can skip ahead much further into the section when we know how many
// instructions we've scanned.
// Return the offset of the load or store instruction in isec that we want to
// patch or 0 if no patch required.
static uint64_t scanCortexA53Errata843419(InputSection *isec, uint64_t &off,
                                          uint64_t limit) {
  uint64_t isecAddr = isec->getVA(0);
```

- EN: Declares or implements routines including `getVA`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getVA`.
- CN: 这里声明或实现函数，例如 `getVA`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getVA`。

### Lines 339-350

```cpp
  // Advance Off so that (isecAddr + Off) modulo 0x1000 is at least 0xff8.
  uint64_t initialPageOff = (isecAddr + off) & 0xfff;
  if (initialPageOff < 0xff8)
    off += 0xff8 - initialPageOff;

  bool optionalAllowed = limit - off > 12;
  if (off >= limit || limit - off < 12) {
    // Need at least 3 4-byte sized instructions to trigger erratum.
    off = limit;
    return 0;
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 351-368

```cpp
  uint64_t patchOff = 0;
  const uint8_t *buf = isec->content().begin();
  const ulittle32_t *instBuf = reinterpret_cast<const ulittle32_t *>(buf + off);
  uint32_t instr1 = *instBuf++;
  uint32_t instr2 = *instBuf++;
  uint32_t instr3 = *instBuf++;
  if (is843419ErratumSequence(instr1, instr2, instr3)) {
    patchOff = off + 8;
  } else if (optionalAllowed && !isBranch(instr3)) {
    uint32_t instr4 = *instBuf++;
    if (is843419ErratumSequence(instr1, instr2, instr4))
      patchOff = off + 12;
  }
  if (((isecAddr + off) & 0xfff) == 0xff8)
    off += 4;
  else
    off += 0xffc;
  return patchOff;
```

- EN: Declares or implements routines including `content`, `if`. Notable symbols here include `content`, `if`.
- CN: 这里声明或实现函数，例如 `content`, `if`。这里较值得关注的符号包括 `content`, `if`。

### Lines 369-377

```cpp
}

class elf::Patch843419Section final : public SyntheticSection {
public:
  Patch843419Section(Ctx &, InputSection *p, uint64_t off,
                     Defined *patcheeCodeSym);

  void writeTo(uint8_t *buf) override;
```

- EN: Introduces type definitions such as `elf`. Declares or implements routines including `writeTo`. Notable symbols here include `elf`, `writeTo`.
- CN: 这里引入类型定义，例如 `elf`。这里声明或实现函数，例如 `writeTo`。这里较值得关注的符号包括 `elf`, `writeTo`。

### Lines 378-385

```cpp
  size_t getSize() const override { return 8; }

  uint64_t getLDSTAddr() const;

  static bool classof(const SectionBase *d) {
    return d->kind() == InputSectionBase::Synthetic && d->name == ".text.patch";
  }
```

- EN: Declares or implements routines including `getSize`, `getLDSTAddr`, `classof`. Notable symbols here include `getSize`, `getLDSTAddr`, `classof`.
- CN: 这里声明或实现函数，例如 `getSize`, `getLDSTAddr`, `classof`。这里较值得关注的符号包括 `getSize`, `getLDSTAddr`, `classof`。

### Lines 386-393

```cpp
  // The Section we are patching.
  const InputSection *patchee;
  // The offset of the instruction in the patchee section we are patching.
  uint64_t patcheeOffset;
  // A label for the start of the Patch that we can use as a relocation target.
  Symbol *patchSym;
};
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 394-408

```cpp
Patch843419Section::Patch843419Section(Ctx &ctx, InputSection *p, uint64_t off,
                                       Defined *patcheeCodeSym)
    : SyntheticSection(ctx, ".text.patch", SHT_PROGBITS,
                       SHF_ALLOC | SHF_EXECINSTR, 4),
      patchee(p), patcheeOffset(off) {
  this->parent = p->getParent();
  patchSym = addSyntheticLocal(
      ctx, ctx.saver.save("__CortexA53843419_" + utohexstr(getLDSTAddr())),
      STT_FUNC, 0, getSize(), *this);
  addSyntheticLocal(ctx, ctx.saver.save("$x"), STT_NOTYPE, 0, 0, *this);
  int64_t retToPatcheeSymOffset =
      getLDSTAddr() - p->getVA(patcheeCodeSym->value) + 4;
  addReloc({R_PC, R_AARCH64_JUMP26, 4, retToPatcheeSymOffset, patcheeCodeSym});
}
```

- EN: Declares or implements routines including `patchee`, `getParent`, `getSize`, `addSyntheticLocal`, `getLDSTAddr`, and 1 more. Notable symbols here include `patchee`, `getParent`, `getSize`, `addSyntheticLocal`, `getLDSTAddr`, `addReloc`.
- CN: 这里声明或实现函数，例如 `patchee`, `getParent`, `getSize`, `addSyntheticLocal`, `getLDSTAddr`, and 1 more。这里较值得关注的符号包括 `patchee`, `getParent`, `getSize`, `addSyntheticLocal`, `getLDSTAddr`, `addReloc`。

### Lines 409-417

```cpp
uint64_t Patch843419Section::getLDSTAddr() const {
  return patchee->getVA(patcheeOffset);
}

void Patch843419Section::writeTo(uint8_t *buf) {
  // Copy the instruction that we will be replacing with a branch in the
  // patchee Section.
  write32le(buf, read32le(patchee->content().begin() + patcheeOffset));
```

- EN: Declares or implements routines including `getLDSTAddr`, `writeTo`, `write32le`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getLDSTAddr`, `writeTo`, `write32le`.
- CN: 这里声明或实现函数，例如 `getLDSTAddr`, `writeTo`, `write32le`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getLDSTAddr`, `writeTo`, `write32le`。

### Lines 418-427

```cpp
  // Apply relocations
  ctx.target->relocateAlloc(*this, buf);
}

void AArch64Err843419Patcher::init() {
  // The AArch64 ABI permits data in executable sections. We must avoid scanning
  // this data as if it were instructions to avoid false matches. We use the
  // mapping symbols in the InputObjects to identify this data, caching the
  // results in sectionMap so we don't have to recalculate it each pass.
```

- EN: Declares or implements routines including `relocateAlloc`, `init`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `relocateAlloc`, `init`.
- CN: 这里声明或实现函数，例如 `relocateAlloc`, `init`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `relocateAlloc`, `init`。

### Lines 428-439

```cpp
  // The ABI Section 4.5.4 Mapping symbols; defines local symbols that describe
  // half open intervals [Symbol Value, Next Symbol Value) of code and data
  // within sections. If there is no next symbol then the half open interval is
  // [Symbol Value, End of section). The type, code or data, is determined by
  // the mapping symbol name, $x for code, $d for data.
  auto isCodeMapSymbol = [](const Symbol *b) {
    return b->getName() == "$x" || b->getName().starts_with("$x.");
  };
  auto isDataMapSymbol = [](const Symbol *b) {
    return b->getName() == "$d" || b->getName().starts_with("$d.");
  };
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 440-457

```cpp
  // Collect mapping symbols for every executable InputSection.
  for (ELFFileBase *file : ctx.objectFiles) {
    for (Symbol *b : file->getLocalSymbols()) {
      auto *def = dyn_cast<Defined>(b);
      if (!def)
        continue;
      if (!def->isSection() && !isCodeMapSymbol(def) && !isDataMapSymbol(def))
        continue;
      if (auto *sec = dyn_cast_or_null<InputSection>(def->section)) {
        if (def->isSection())
          sectionMap[sec].first = def;
        else if (sec->flags & SHF_EXECINSTR)
          sectionMap[sec].second.push_back(def);
      }
    }
  }
  // For each InputSection make sure the mapping symbols are in sorted in
  // ascending order and free from consecutive runs of mapping symbols with
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 458-475

```cpp
  // the same type. For example we must remove the redundant $d.1 from $x.0
  // $d.0 $d.1 $x.1.
  for (auto &kv : sectionMap) {
    auto &mapSyms = kv.second.second;
    llvm::stable_sort(mapSyms, [](const Defined *a, const Defined *b) {
      return a->value < b->value;
    });
    mapSyms.erase(llvm::unique(mapSyms,
                               [=](const Defined *a, const Defined *b) {
                                 return isCodeMapSymbol(a) ==
                                        isCodeMapSymbol(b);
                               }),
                  mapSyms.end());
    // Always start with a Code Mapping Symbol.
    if (!mapSyms.empty() && !isCodeMapSymbol(mapSyms.front()))
      mapSyms.erase(mapSyms.begin());
  }
  initialized = true;
```

- EN: Declares or implements routines including `stable_sort`, `isCodeMapSymbol`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `stable_sort`, `isCodeMapSymbol`.
- CN: 这里声明或实现函数，例如 `stable_sort`, `isCodeMapSymbol`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `stable_sort`, `isCodeMapSymbol`。

### Lines 476-490

```cpp
}

// Insert the PatchSections we have created back into the
// InputSectionDescription. As inserting patches alters the addresses of
// InputSections that follow them, we try and place the patches after all the
// executable sections, although we may need to insert them earlier if the
// InputSectionDescription is larger than the maximum branch range.
void AArch64Err843419Patcher::insertPatches(
    InputSectionDescription &isd, std::vector<Patch843419Section *> &patches) {
  uint64_t isecLimit;
  uint64_t prevIsecLimit = isd.sections.front()->outSecOff;
  uint64_t patchUpperBound =
      prevIsecLimit + ctx.target->getThunkSectionSpacing();
  uint64_t outSecAddr = isd.sections.front()->getParent()->addr;
```

- EN: Declares or implements routines including `getThunkSectionSpacing`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getThunkSectionSpacing`.
- CN: 这里声明或实现函数，例如 `getThunkSectionSpacing`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getThunkSectionSpacing`。

### Lines 491-508

```cpp
  // Set the outSecOff of patches to the place where we want to insert them.
  // We use a similar strategy to Thunk placement. Place patches roughly
  // every multiple of maximum branch range.
  auto patchIt = patches.begin();
  auto patchEnd = patches.end();
  for (const InputSection *isec : isd.sections) {
    isecLimit = isec->outSecOff + isec->getSize();
    if (isecLimit > patchUpperBound) {
      while (patchIt != patchEnd) {
        if ((*patchIt)->getLDSTAddr() - outSecAddr >= prevIsecLimit)
          break;
        (*patchIt)->outSecOff = prevIsecLimit;
        ++patchIt;
      }
      patchUpperBound = prevIsecLimit + ctx.target->getThunkSectionSpacing();
    }
    prevIsecLimit = isecLimit;
  }
```

- EN: Declares or implements routines including `getSize`, `getThunkSectionSpacing`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getSize`, `getThunkSectionSpacing`.
- CN: 这里声明或实现函数，例如 `getSize`, `getThunkSectionSpacing`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getSize`, `getThunkSectionSpacing`。

### Lines 509-526

```cpp
  for (; patchIt != patchEnd; ++patchIt) {
    (*patchIt)->outSecOff = isecLimit;
  }

  // Merge all patch sections. We use the outSecOff assigned above to
  // determine the insertion point. This is ok as we only merge into an
  // InputSectionDescription once per pass, and at the end of the pass
  // assignAddresses() will recalculate all the outSecOff values.
  SmallVector<InputSection *, 0> tmp;
  tmp.reserve(isd.sections.size() + patches.size());
  auto mergeCmp = [](const InputSection *a, const InputSection *b) {
    if (a->outSecOff != b->outSecOff)
      return a->outSecOff < b->outSecOff;
    return isa<Patch843419Section>(a) && !isa<Patch843419Section>(b);
  };
  std::merge(isd.sections.begin(), isd.sections.end(), patches.begin(),
             patches.end(), std::back_inserter(tmp), mergeCmp);
  isd.sections = std::move(tmp);
```

- EN: Declares or implements routines including `merge`, `move`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `merge`, `move`.
- CN: 这里声明或实现函数，例如 `merge`, `move`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `merge`, `move`。

### Lines 527-544

```cpp
}

// Given an erratum sequence that starts at address adrpAddr, with an
// instruction that we need to patch at patcheeOffset from the start of
// InputSection isec, create a Patch843419 Section and add it to the
// Patches that we need to insert.
static void implementPatch(Ctx &ctx, uint64_t adrpAddr, uint64_t patcheeOffset,
                           InputSection *isec,
                           std::vector<Patch843419Section *> &patches,
                           Defined *patcheeCodeSym) {
  // There may be a relocation at the same offset that we are patching. There
  // are four cases that we need to consider.
  // Case 1: R_AARCH64_JUMP26 branch relocation. We have already patched this
  // instance of the erratum on a previous patch and altered the relocation. We
  // have nothing more to do.
  // Case 2: A TLS IE to LE optimization. In this case the ADRP that we read
  // will be transformed into a MOVZ later so we actually don't match the
  // sequence and have nothing more to do.
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 545-561

```cpp
  // Case 3: A load/store register (unsigned immediate) class relocation. There
  // are two of these R_AARCH_LD64_ABS_LO12_NC and R_AARCH_LD64_GOT_LO12_NC and
  // they are both absolute. We need to add the same relocation to the patch,
  // and replace the relocation with a R_AARCH_JUMP26 branch relocation.
  // Case 4: No relocation. We must create a new R_AARCH64_JUMP26 branch
  // relocation at the offset.
  auto relIt = llvm::find_if(isec->relocs(), [=](const Relocation &r) {
    return r.offset == patcheeOffset;
  });
  // Detect and skip Case 1 and Case 2 above.
  if (relIt != isec->relocs().end() &&
      (relIt->type == R_AARCH64_JUMP26 ||
       ((relIt->type == R_AARCH64_TLSIE_ADR_GOTTPREL_PAGE21 ||
         relIt->type == R_AARCH64_TLSIE_LD64_GOTTPREL_LO12_NC) &&
        relIt->expr == R_TPREL)))
    return;
```

- EN: Introduces type definitions such as `relocation`. Declares or implements routines including `find_if`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `relocation`, `find_if`.
- CN: 这里引入类型定义，例如 `relocation`。这里声明或实现函数，例如 `find_if`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `relocation`, `find_if`。

### Lines 562-571

```cpp
  Log(ctx) << "detected cortex-a53-843419 erratum sequence starting at " <<
      utohexstr(adrpAddr) << " in unpatched output.";

  auto *ps = make<Patch843419Section>(ctx, isec, patcheeOffset, patcheeCodeSym);
  patches.push_back(ps);

  auto makeRelToPatch = [](uint64_t offset, Symbol *patchSym) {
    return Relocation{R_PC, R_AARCH64_JUMP26, offset, 0, patchSym};
  };
```

- EN: Declares or implements routines including `Log`, `utohexstr`. Notable symbols here include `Log`, `utohexstr`.
- CN: 这里声明或实现函数，例如 `Log`, `utohexstr`。这里较值得关注的符号包括 `Log`, `utohexstr`。

### Lines 572-589

```cpp
  if (relIt != isec->relocs().end()) {
    ps->addReloc({relIt->expr, relIt->type, 0, relIt->addend, relIt->sym});
    *relIt = makeRelToPatch(patcheeOffset, ps->patchSym);
  } else
    isec->addReloc(makeRelToPatch(patcheeOffset, ps->patchSym));
}

// Scan all the instructions in InputSectionDescription, for each instance of
// the erratum sequence create a Patch843419Section. We return the list of
// Patch843419Sections that need to be applied to the InputSectionDescription.
std::vector<Patch843419Section *>
AArch64Err843419Patcher::patchInputSectionDescription(
    InputSectionDescription &isd) {
  std::vector<Patch843419Section *> patches;
  for (InputSection *isec : isd.sections) {
    //  LLD doesn't use the erratum sequence in SyntheticSections.
    if (isa<SyntheticSection>(isec))
      continue;
```

- EN: Declares or implements routines including `addReloc`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `addReloc`.
- CN: 这里声明或实现函数，例如 `addReloc`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `addReloc`。

### Lines 590-600

```cpp
    // Use sectionMap to make sure we only scan code and not inline data.
    // We have already sorted MapSyms in ascending order and removed consecutive
    // mapping symbols of the same type. Our range of executable instructions to
    // scan is therefore [codeSym->value, dataSym->value) or [codeSym->value,
    // section size).
    auto [it, inserted] = sectionMap.try_emplace(
        isec, std::make_pair(nullptr, SmallVector<Defined *, 0>{}));
    auto &[sectionSym, mapSyms] = it->second;
    if (inserted || sectionSym == nullptr)
      sectionSym = addSyntheticLocal(ctx, "", STT_SECTION, 0, 0, *isec);
```

- EN: Declares or implements routines including `make_pair`, `addSyntheticLocal`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `make_pair`, `addSyntheticLocal`.
- CN: 这里声明或实现函数，例如 `make_pair`, `addSyntheticLocal`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `make_pair`, `addSyntheticLocal`。

### Lines 601-618

```cpp
    auto codeSym = mapSyms.begin();
    while (codeSym != mapSyms.end()) {
      auto dataSym = std::next(codeSym);
      uint64_t off = (*codeSym)->value;
      uint64_t limit = (dataSym == mapSyms.end()) ? isec->content().size()
                                                  : (*dataSym)->value;

      while (off < limit) {
        uint64_t startAddr = isec->getVA(off);
        if (uint64_t patcheeOffset =
                scanCortexA53Errata843419(isec, off, limit))
          implementPatch(ctx, startAddr, patcheeOffset, isec, patches,
                         sectionSym);
      }
      if (dataSym == mapSyms.end())
        break;
      codeSym = std::next(dataSym);
    }
```

- EN: Declares or implements routines including `next`, `getVA`, `scanCortexA53Errata843419`. Notable symbols here include `next`, `getVA`, `scanCortexA53Errata843419`.
- CN: 这里声明或实现函数，例如 `next`, `getVA`, `scanCortexA53Errata843419`。这里较值得关注的符号包括 `next`, `getVA`, `scanCortexA53Errata843419`。

### Lines 619-636

```cpp
  }
  return patches;
}

// For each InputSectionDescription make one pass over the executable sections
// looking for the erratum sequence; creating a synthetic Patch843419Section
// for each instance found. We insert these synthetic patch sections after the
// executable code in each InputSectionDescription.
//
// PreConditions:
// The Output and Input Sections have had their final addresses assigned.
//
// PostConditions:
// Returns true if at least one patch was added. The addresses of the
// Output and Input Sections may have been changed.
// Returns false if no patches were required and no changes were made.
bool AArch64Err843419Patcher::createFixes() {
  if (!initialized)
```

- EN: Declares or implements routines including `createFixes`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `createFixes`.
- CN: 这里声明或实现函数，例如 `createFixes`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `createFixes`。

### Lines 637-654

```cpp
    init();

  bool addressesChanged = false;
  for (OutputSection *os : ctx.outputSections) {
    if (!(os->flags & SHF_ALLOC) || !(os->flags & SHF_EXECINSTR))
      continue;
    for (SectionCommand *cmd : os->commands)
      if (auto *isd = dyn_cast<InputSectionDescription>(cmd)) {
        std::vector<Patch843419Section *> patches =
            patchInputSectionDescription(*isd);
        if (!patches.empty()) {
          insertPatches(*isd, patches);
          addressesChanged = true;
        }
      }
  }
  return addressesChanged;
}
```

- EN: Declares or implements routines including `init`, `patchInputSectionDescription`, `insertPatches`. Notable symbols here include `init`, `patchInputSectionDescription`, `insertPatches`.
- CN: 这里声明或实现函数，例如 `init`, `patchInputSectionDescription`, `insertPatches`。这里较值得关注的符号包括 `init`, `patchInputSectionDescription`, `insertPatches`。

## Key Concepts / 关键概念

- `of`: class or struct interface / 类或结构体接口
- `elf`: class or struct interface / 类或结构体接口
- `relocation`: class or struct interface / 类或结构体接口
- `isADRP`: function or method entry point / 函数或方法入口
- `isLoadStoreClass`: function or method entry point / 函数或方法入口
- `isST1MultipleOpcode`: function or method entry point / 函数或方法入口
- `isST1Multiple`: function or method entry point / 函数或方法入口
- `isST1MultiplePost`: function or method entry point / 函数或方法入口

## Dependencies / 依赖关系

- LLVM headers / LLVM 头文件: `llvm/ADT/StringExtras.h`, `llvm/Support/Endian.h`
- System headers / 系统头文件: `AArch64ErrataFix.h`, `InputFiles.h`, `LinkerScript.h`, `OutputSections.h`, `Relocations.h`, `Symbols.h`, `SyntheticSections.h`, `Target.h`, `algorithm`
- Directory context / 目录上下文: `lld/ELF` neighbors usually cooperate with this file to provide the surrounding subsystem / `lld/ELF` 下的相邻文件通常与本文件协作组成对应子系统
