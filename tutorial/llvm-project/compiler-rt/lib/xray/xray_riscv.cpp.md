# xray_riscv.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `compiler-rt/lib/xray/xray_riscv.cpp`
- **Repository**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN)**: This file is a part of XRay, a dynamic runtime instrumentation system.
- **目的（中文）**: 该实现文件提供与 `XRay riscv` 相关的运行时逻辑。

## Line-by-Line Analysis / 逐行分析
### Line 1
````cpp
//===-- xray_riscv.cpp ----------------------------------------*- C++ -*-===//
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 2
````cpp
//
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 3
````cpp
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
````
- **EN**: Comment documenting `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
- **CN**: 注释说明了 `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。

### Line 4
````cpp
// See https://llvm.org/LICENSE.txt for license information.
````
- **EN**: Comment documenting `See https://llvm.org/LICENSE.txt for license information.`.
- **CN**: 注释说明了 `See https://llvm.org/LICENSE.txt for license information.`。

### Line 5
````cpp
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
````
- **EN**: Comment documenting `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
- **CN**: 注释说明了 `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。

### Line 6
````cpp
//
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 7
````cpp
//===----------------------------------------------------------------------===//
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 8
````cpp
//
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 9
````cpp
// This file is a part of XRay, a dynamic runtime instrumentation system.
````
- **EN**: Comment documenting `This file is a part of XRay, a dynamic runtime instrumentation system.`.
- **CN**: 注释说明了 `This file is a part of XRay, a dynamic runtime instrumentation system.`。

### Line 10
````cpp
//
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 11
````cpp
// Implementation of RISC-V specific routines (32- and 64-bit).
````
- **EN**: Comment documenting `Implementation of RISC-V specific routines (32- and 64-bit).`.
- **CN**: 注释说明了 `Implementation of RISC-V specific routines (32- and 64-bit).`。

### Line 12
````cpp
//
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 13
````cpp
//===----------------------------------------------------------------------===//
````
- **EN**: Banner comment delimiting a file header or major section.
- **CN**: 横幅注释，用于分隔文件头或主要章节。

### Line 14
````cpp
#include "sanitizer_common/sanitizer_common.h"
````
- **EN**: Includes the local dependency `sanitizer_common/sanitizer_common.h`.
- **CN**: 引入本地依赖 `sanitizer_common/sanitizer_common.h`。

### Line 15
````cpp
#include "xray_defs.h"
````
- **EN**: Includes the local dependency `xray_defs.h`.
- **CN**: 引入本地依赖 `xray_defs.h`。

### Line 16
````cpp
#include "xray_interface_internal.h"
````
- **EN**: Includes the local dependency `xray_interface_internal.h`.
- **CN**: 引入本地依赖 `xray_interface_internal.h`。

### Line 17
````cpp
#include <atomic>
````
- **EN**: Includes the system dependency `atomic`.
- **CN**: 引入系统依赖 `atomic`。

### Line 18
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 19
````cpp
namespace __xray {
````
- **EN**: Opens namespace `__xray`.
- **CN**: 打开命名空间 `__xray`。

### Line 20
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 21
````cpp
// The machine codes for some instructions used in runtime patching.
````
- **EN**: Comment documenting `The machine codes for some instructions used in runtime patching.`.
- **CN**: 注释说明了 `The machine codes for some instructions used in runtime patching.`。

### Line 22
````cpp
enum PatchOpcodes : uint32_t {
````
- **EN**: Declares the enum `PatchOpcodes`.
- **CN**: 声明 enum `PatchOpcodes`。

### Line 23
````cpp
  PO_ADDI = 0x00000013, // addi rd, rs1, imm
````
- **EN**: Carries part of the local implementation logic: `PO_ADDI = 0x00000013, // addi rd, rs1, imm`.
- **CN**: 承载局部实现逻辑：`PO_ADDI = 0x00000013, // addi rd, rs1, imm`。

### Line 24
````cpp
  PO_ADD = 0x00000033,  // add rd, rs1, rs2
````
- **EN**: Carries part of the local implementation logic: `PO_ADD = 0x00000033,  // add rd, rs1, rs2`.
- **CN**: 承载局部实现逻辑：`PO_ADD = 0x00000033,  // add rd, rs1, rs2`。

### Line 25
````cpp
  PO_SW = 0x00002023,   // sw rs2, imm(rs1)
````
- **EN**: Carries part of the local implementation logic: `PO_SW = 0x00002023,   // sw rs2, imm(rs1)`.
- **CN**: 承载局部实现逻辑：`PO_SW = 0x00002023,   // sw rs2, imm(rs1)`。

### Line 26
````cpp
  PO_SD = 0x00003023,   // sd rs2, imm(rs1)
````
- **EN**: Carries part of the local implementation logic: `PO_SD = 0x00003023,   // sd rs2, imm(rs1)`.
- **CN**: 承载局部实现逻辑：`PO_SD = 0x00003023,   // sd rs2, imm(rs1)`。

### Line 27
````cpp
  PO_LUI = 0x00000037,  // lui rd, imm
````
- **EN**: Carries part of the local implementation logic: `PO_LUI = 0x00000037,  // lui rd, imm`.
- **CN**: 承载局部实现逻辑：`PO_LUI = 0x00000037,  // lui rd, imm`。

### Line 28
````cpp
  PO_OR = 0x00006033,   // or rd, rs1, rs2
````
- **EN**: Carries part of the local implementation logic: `PO_OR = 0x00006033,   // or rd, rs1, rs2`.
- **CN**: 承载局部实现逻辑：`PO_OR = 0x00006033,   // or rd, rs1, rs2`。

### Line 29
````cpp
  PO_SLLI = 0x00001013, // slli rd, rs1, shamt
````
- **EN**: Carries part of the local implementation logic: `PO_SLLI = 0x00001013, // slli rd, rs1, shamt`.
- **CN**: 承载局部实现逻辑：`PO_SLLI = 0x00001013, // slli rd, rs1, shamt`。

### Line 30
````cpp
  PO_JALR = 0x00000067, // jalr rd, rs1
````
- **EN**: Carries part of the local implementation logic: `PO_JALR = 0x00000067, // jalr rd, rs1`.
- **CN**: 承载局部实现逻辑：`PO_JALR = 0x00000067, // jalr rd, rs1`。

### Line 31
````cpp
  PO_LW = 0x00002003,   // lw rd, imm(rs1)
````
- **EN**: Carries part of the local implementation logic: `PO_LW = 0x00002003,   // lw rd, imm(rs1)`.
- **CN**: 承载局部实现逻辑：`PO_LW = 0x00002003,   // lw rd, imm(rs1)`。

### Line 32
````cpp
  PO_LD = 0x00003003,   // ld rd, imm(rs1)
````
- **EN**: Carries part of the local implementation logic: `PO_LD = 0x00003003,   // ld rd, imm(rs1)`.
- **CN**: 承载局部实现逻辑：`PO_LD = 0x00003003,   // ld rd, imm(rs1)`。

### Line 33
````cpp
  PO_J = 0x0000006f,    // jal imm
````
- **EN**: Carries part of the local implementation logic: `PO_J = 0x0000006f,    // jal imm`.
- **CN**: 承载局部实现逻辑：`PO_J = 0x0000006f,    // jal imm`。

### Line 34
````cpp
  PO_NOP = PO_ADDI,     // addi x0, x0, 0
````
- **EN**: Carries part of the local implementation logic: `PO_NOP = PO_ADDI,     // addi x0, x0, 0`.
- **CN**: 承载局部实现逻辑：`PO_NOP = PO_ADDI,     // addi x0, x0, 0`。

### Line 35
````cpp
};
````
- **EN**: Closes a type definition or aggregate block.
- **CN**: 结束一个类型定义或聚合块。

### Line 36
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 37
````cpp
enum RegNum : uint32_t {
````
- **EN**: Declares the enum `RegNum`.
- **CN**: 声明 enum `RegNum`。

### Line 38
````cpp
  RN_X0 = 0,
````
- **EN**: Carries part of the local implementation logic: `RN_X0 = 0,`.
- **CN**: 承载局部实现逻辑：`RN_X0 = 0,`。

### Line 39
````cpp
  RN_RA = 1,
````
- **EN**: Carries part of the local implementation logic: `RN_RA = 1,`.
- **CN**: 承载局部实现逻辑：`RN_RA = 1,`。

### Line 40
````cpp
  RN_SP = 2,
````
- **EN**: Carries part of the local implementation logic: `RN_SP = 2,`.
- **CN**: 承载局部实现逻辑：`RN_SP = 2,`。

### Line 41
````cpp
  RN_T1 = 6,
````
- **EN**: Carries part of the local implementation logic: `RN_T1 = 6,`.
- **CN**: 承载局部实现逻辑：`RN_T1 = 6,`。

### Line 42
````cpp
  RN_A0 = 10,
````
- **EN**: Carries part of the local implementation logic: `RN_A0 = 10,`.
- **CN**: 承载局部实现逻辑：`RN_A0 = 10,`。

### Line 43
````cpp
};
````
- **EN**: Closes a type definition or aggregate block.
- **CN**: 结束一个类型定义或聚合块。

### Line 44
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 45
````cpp
static inline uint32_t encodeRTypeInstruction(uint32_t Opcode, uint32_t Rs1,
````
- **EN**: Carries part of the local implementation logic: `static inline uint32_t encodeRTypeInstruction(uint32_t Opcode, uint32_t Rs1,`.
- **CN**: 承载局部实现逻辑：`static inline uint32_t encodeRTypeInstruction(uint32_t Opcode, uint32_t Rs1,`。

### Line 46
````cpp
                                              uint32_t Rs2, uint32_t Rd) {
````
- **EN**: Carries part of the local implementation logic: `uint32_t Rs2, uint32_t Rd) {`.
- **CN**: 承载局部实现逻辑：`uint32_t Rs2, uint32_t Rd) {`。

### Line 47
````cpp
  return Rs2 << 20 | Rs1 << 15 | Rd << 7 | Opcode;
````
- **EN**: Returns from the current function with `Rs2 << 20 | Rs1 << 15 | Rd << 7 | Opcode;`.
- **CN**: 使用 `Rs2 << 20 | Rs1 << 15 | Rd << 7 | Opcode;` 从当前函数返回。

### Line 48
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 49
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 50
````cpp
static inline uint32_t encodeITypeInstruction(uint32_t Opcode, uint32_t Rs1,
````
- **EN**: Carries part of the local implementation logic: `static inline uint32_t encodeITypeInstruction(uint32_t Opcode, uint32_t Rs1,`.
- **CN**: 承载局部实现逻辑：`static inline uint32_t encodeITypeInstruction(uint32_t Opcode, uint32_t Rs1,`。

### Line 51
````cpp
                                              uint32_t Rd, uint32_t Imm) {
````
- **EN**: Carries part of the local implementation logic: `uint32_t Rd, uint32_t Imm) {`.
- **CN**: 承载局部实现逻辑：`uint32_t Rd, uint32_t Imm) {`。

### Line 52
````cpp
  return Imm << 20 | Rs1 << 15 | Rd << 7 | Opcode;
````
- **EN**: Returns from the current function with `Imm << 20 | Rs1 << 15 | Rd << 7 | Opcode;`.
- **CN**: 使用 `Imm << 20 | Rs1 << 15 | Rd << 7 | Opcode;` 从当前函数返回。

### Line 53
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 54
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 55
````cpp
static inline uint32_t encodeSTypeInstruction(uint32_t Opcode, uint32_t Rs1,
````
- **EN**: Carries part of the local implementation logic: `static inline uint32_t encodeSTypeInstruction(uint32_t Opcode, uint32_t Rs1,`.
- **CN**: 承载局部实现逻辑：`static inline uint32_t encodeSTypeInstruction(uint32_t Opcode, uint32_t Rs1,`。

### Line 56
````cpp
                                              uint32_t Rs2, uint32_t Imm) {
````
- **EN**: Carries part of the local implementation logic: `uint32_t Rs2, uint32_t Imm) {`.
- **CN**: 承载局部实现逻辑：`uint32_t Rs2, uint32_t Imm) {`。

### Line 57
````cpp
  uint32_t ImmMSB = (Imm & 0xfe0) << 20;
````
- **EN**: Declares an interface element or prototype: `uint32_t ImmMSB = (Imm & 0xfe0) << 20;`.
- **CN**: 声明一个接口元素或原型：`uint32_t ImmMSB = (Imm & 0xfe0) << 20;`。

### Line 58
````cpp
  uint32_t ImmLSB = (Imm & 0x01f) << 7;
````
- **EN**: Declares an interface element or prototype: `uint32_t ImmLSB = (Imm & 0x01f) << 7;`.
- **CN**: 声明一个接口元素或原型：`uint32_t ImmLSB = (Imm & 0x01f) << 7;`。

### Line 59
````cpp
  return ImmMSB | Rs2 << 20 | Rs1 << 15 | ImmLSB | Opcode;
````
- **EN**: Returns from the current function with `ImmMSB | Rs2 << 20 | Rs1 << 15 | ImmLSB | Opcode;`.
- **CN**: 使用 `ImmMSB | Rs2 << 20 | Rs1 << 15 | ImmLSB | Opcode;` 从当前函数返回。

### Line 60
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 61
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 62
````cpp
static inline uint32_t encodeUTypeInstruction(uint32_t Opcode, uint32_t Rd,
````
- **EN**: Carries part of the local implementation logic: `static inline uint32_t encodeUTypeInstruction(uint32_t Opcode, uint32_t Rd,`.
- **CN**: 承载局部实现逻辑：`static inline uint32_t encodeUTypeInstruction(uint32_t Opcode, uint32_t Rd,`。

### Line 63
````cpp
                                              uint32_t Imm) {
````
- **EN**: Carries part of the local implementation logic: `uint32_t Imm) {`.
- **CN**: 承载局部实现逻辑：`uint32_t Imm) {`。

### Line 64
````cpp
  return Imm << 12 | Rd << 7 | Opcode;
````
- **EN**: Returns from the current function with `Imm << 12 | Rd << 7 | Opcode;`.
- **CN**: 使用 `Imm << 12 | Rd << 7 | Opcode;` 从当前函数返回。

### Line 65
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 66
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 67
````cpp
static inline uint32_t encodeJTypeInstruction(uint32_t Opcode, uint32_t Rd,
````
- **EN**: Carries part of the local implementation logic: `static inline uint32_t encodeJTypeInstruction(uint32_t Opcode, uint32_t Rd,`.
- **CN**: 承载局部实现逻辑：`static inline uint32_t encodeJTypeInstruction(uint32_t Opcode, uint32_t Rd,`。

### Line 68
````cpp
                                              uint32_t Imm) {
````
- **EN**: Carries part of the local implementation logic: `uint32_t Imm) {`.
- **CN**: 承载局部实现逻辑：`uint32_t Imm) {`。

### Line 69
````cpp
  uint32_t ImmMSB = (Imm & 0x100000) << 11;
````
- **EN**: Declares an interface element or prototype: `uint32_t ImmMSB = (Imm & 0x100000) << 11;`.
- **CN**: 声明一个接口元素或原型：`uint32_t ImmMSB = (Imm & 0x100000) << 11;`。

### Line 70
````cpp
  uint32_t ImmLSB = (Imm & 0x7fe) << 20;
````
- **EN**: Declares an interface element or prototype: `uint32_t ImmLSB = (Imm & 0x7fe) << 20;`.
- **CN**: 声明一个接口元素或原型：`uint32_t ImmLSB = (Imm & 0x7fe) << 20;`。

### Line 71
````cpp
  uint32_t Imm11 = (Imm & 0x800) << 9;
````
- **EN**: Declares an interface element or prototype: `uint32_t Imm11 = (Imm & 0x800) << 9;`.
- **CN**: 声明一个接口元素或原型：`uint32_t Imm11 = (Imm & 0x800) << 9;`。

### Line 72
````cpp
  uint32_t Imm1912 = (Imm & 0xff000);
````
- **EN**: Declares an interface element or prototype: `uint32_t Imm1912 = (Imm & 0xff000);`.
- **CN**: 声明一个接口元素或原型：`uint32_t Imm1912 = (Imm & 0xff000);`。

### Line 73
````cpp
  return ImmMSB | ImmLSB | Imm11 | Imm1912 | Rd << 7 | Opcode;
````
- **EN**: Returns from the current function with `ImmMSB | ImmLSB | Imm11 | Imm1912 | Rd << 7 | Opcode;`.
- **CN**: 使用 `ImmMSB | ImmLSB | Imm11 | Imm1912 | Rd << 7 | Opcode;` 从当前函数返回。

### Line 74
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 75
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 76
````cpp
static uint32_t hi20(uint32_t val) { return (val + 0x800) >> 12; }
````
- **EN**: Carries part of the local implementation logic: `static uint32_t hi20(uint32_t val) { return (val + 0x800) >> 12; }`.
- **CN**: 承载局部实现逻辑：`static uint32_t hi20(uint32_t val) { return (val + 0x800) >> 12; }`。

### Line 77
````cpp
static uint32_t lo12(uint32_t val) { return val & 0xfff; }
````
- **EN**: Carries part of the local implementation logic: `static uint32_t lo12(uint32_t val) { return val & 0xfff; }`.
- **CN**: 承载局部实现逻辑：`static uint32_t lo12(uint32_t val) { return val & 0xfff; }`。

### Line 78
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 79
````cpp
static inline bool patchSled(const bool Enable, const uint32_t FuncId,
````
- **EN**: Carries part of the local implementation logic: `static inline bool patchSled(const bool Enable, const uint32_t FuncId,`.
- **CN**: 承载局部实现逻辑：`static inline bool patchSled(const bool Enable, const uint32_t FuncId,`。

### Line 80
````cpp
                             const XRaySledEntry &Sled,
````
- **EN**: Carries part of the local implementation logic: `const XRaySledEntry &Sled,`.
- **CN**: 承载局部实现逻辑：`const XRaySledEntry &Sled,`。

### Line 81
````cpp
                             void (*TracingHook)()) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `void (*TracingHook)()) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`void (*TracingHook)()) XRAY_NEVER_INSTRUMENT {`。

### Line 82
````cpp
  // When |Enable| == true,
````
- **EN**: Comment documenting `When |Enable| == true,`.
- **CN**: 注释说明了 `When |Enable| == true,`。

### Line 83
````cpp
  // We replace the following compile-time stub (sled):
````
- **EN**: Comment documenting `We replace the following compile-time stub (sled):`.
- **CN**: 注释说明了 `We replace the following compile-time stub (sled):`。

### Line 84
````cpp
  //
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 85
````cpp
  // xray_sled_n:
````
- **EN**: Comment documenting `xray_sled_n:`.
- **CN**: 注释说明了 `xray_sled_n:`。

### Line 86
````cpp
  //	J .tmpN
````
- **EN**: Comment documenting `J .tmpN`.
- **CN**: 注释说明了 `J .tmpN`。

### Line 87
````cpp
  //	21 or 33 C.NOPs (42 or 66 bytes)
````
- **EN**: Comment documenting `21 or 33 C.NOPs (42 or 66 bytes)`.
- **CN**: 注释说明了 `21 or 33 C.NOPs (42 or 66 bytes)`。

### Line 88
````cpp
  //	.tmpN
````
- **EN**: Comment documenting `.tmpN`.
- **CN**: 注释说明了 `.tmpN`。

### Line 89
````cpp
  //
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 90
````cpp
  // With one of the following runtime patches:
````
- **EN**: Comment documenting `With one of the following runtime patches:`.
- **CN**: 注释说明了 `With one of the following runtime patches:`。

### Line 91
````cpp
  //
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 92
````cpp
  // xray_sled_n (32-bit):
````
- **EN**: Comment documenting `xray_sled_n (32-bit):`.
- **CN**: 注释说明了 `xray_sled_n (32-bit):`。

### Line 93
````cpp
  //    addi sp, sp, -16                                ;create stack frame
````
- **EN**: Comment documenting `addi sp, sp, -16                                ;create stack frame`.
- **CN**: 注释说明了 `addi sp, sp, -16                                ;create stack frame`。

### Line 94
````cpp
  //    sw ra, 12(sp)                                   ;save return address
````
- **EN**: Comment documenting `sw ra, 12(sp)                                   ;save return address`.
- **CN**: 注释说明了 `sw ra, 12(sp)                                   ;save return address`。

### Line 95
````cpp
  //    sw a0, 8(sp)                                    ;save register a0
````
- **EN**: Comment documenting `sw a0, 8(sp)                                    ;save register a0`.
- **CN**: 注释说明了 `sw a0, 8(sp)                                    ;save register a0`。

### Line 96
````cpp
  //    lui ra, %hi(__xray_FunctionEntry/Exit)
````
- **EN**: Comment documenting `lui ra, %hi(__xray_FunctionEntry/Exit)`.
- **CN**: 注释说明了 `lui ra, %hi(__xray_FunctionEntry/Exit)`。

### Line 97
````cpp
  //    addi ra, ra, %lo(__xray_FunctionEntry/Exit)
````
- **EN**: Comment documenting `addi ra, ra, %lo(__xray_FunctionEntry/Exit)`.
- **CN**: 注释说明了 `addi ra, ra, %lo(__xray_FunctionEntry/Exit)`。

### Line 98
````cpp
  //    lui a0, %hi(function_id)
````
- **EN**: Comment documenting `lui a0, %hi(function_id)`.
- **CN**: 注释说明了 `lui a0, %hi(function_id)`。

### Line 99
````cpp
  //    addi a0, a0, %lo(function_id)                   ;pass function id
````
- **EN**: Comment documenting `addi a0, a0, %lo(function_id)                   ;pass function id`.
- **CN**: 注释说明了 `addi a0, a0, %lo(function_id)                   ;pass function id`。

### Line 100
````cpp
  //    jalr ra                                         ;call Tracing hook
````
- **EN**: Comment documenting `jalr ra                                         ;call Tracing hook`.
- **CN**: 注释说明了 `jalr ra                                         ;call Tracing hook`。

### Line 101
````cpp
  //    lw a0, 8(sp)                                    ;restore register a0
````
- **EN**: Comment documenting `lw a0, 8(sp)                                    ;restore register a0`.
- **CN**: 注释说明了 `lw a0, 8(sp)                                    ;restore register a0`。

### Line 102
````cpp
  //    lw ra, 12(sp)                                   ;restore return address
````
- **EN**: Comment documenting `lw ra, 12(sp)                                   ;restore return address`.
- **CN**: 注释说明了 `lw ra, 12(sp)                                   ;restore return address`。

### Line 103
````cpp
  //    addi sp, sp, 16                                 ;delete stack frame
````
- **EN**: Comment documenting `addi sp, sp, 16                                 ;delete stack frame`.
- **CN**: 注释说明了 `addi sp, sp, 16                                 ;delete stack frame`。

### Line 104
````cpp
  //
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 105
````cpp
  // xray_sled_n (64-bit):
````
- **EN**: Comment documenting `xray_sled_n (64-bit):`.
- **CN**: 注释说明了 `xray_sled_n (64-bit):`。

### Line 106
````cpp
  //    addi sp, sp, -32                                ;create stack frame
````
- **EN**: Comment documenting `addi sp, sp, -32                                ;create stack frame`.
- **CN**: 注释说明了 `addi sp, sp, -32                                ;create stack frame`。

### Line 107
````cpp
  //    sd ra, 24(sp)                                   ;save return address
````
- **EN**: Comment documenting `sd ra, 24(sp)                                   ;save return address`.
- **CN**: 注释说明了 `sd ra, 24(sp)                                   ;save return address`。

### Line 108
````cpp
  //    sd a0, 16(sp)                                   ;save register a0
````
- **EN**: Comment documenting `sd a0, 16(sp)                                   ;save register a0`.
- **CN**: 注释说明了 `sd a0, 16(sp)                                   ;save register a0`。

### Line 109
````cpp
  //    sd t1, 8(sp)                                    ;save register t1
````
- **EN**: Comment documenting `sd t1, 8(sp)                                    ;save register t1`.
- **CN**: 注释说明了 `sd t1, 8(sp)                                    ;save register t1`。

### Line 110
````cpp
  //    lui t1, %highest(__xray_FunctionEntry/Exit)
````
- **EN**: Comment documenting `lui t1, %highest(__xray_FunctionEntry/Exit)`.
- **CN**: 注释说明了 `lui t1, %highest(__xray_FunctionEntry/Exit)`。

### Line 111
````cpp
  //    addi t1, t1, %higher(__xray_FunctionEntry/Exit)
````
- **EN**: Comment documenting `addi t1, t1, %higher(__xray_FunctionEntry/Exit)`.
- **CN**: 注释说明了 `addi t1, t1, %higher(__xray_FunctionEntry/Exit)`。

### Line 112
````cpp
  //    slli t1, t1, 32
````
- **EN**: Comment documenting `slli t1, t1, 32`.
- **CN**: 注释说明了 `slli t1, t1, 32`。

### Line 113
````cpp
  //    lui ra, ra, %hi(__xray_FunctionEntry/Exit)
````
- **EN**: Comment documenting `lui ra, ra, %hi(__xray_FunctionEntry/Exit)`.
- **CN**: 注释说明了 `lui ra, ra, %hi(__xray_FunctionEntry/Exit)`。

### Line 114
````cpp
  //    addi ra, ra, %lo(__xray_FunctionEntry/Exit)
````
- **EN**: Comment documenting `addi ra, ra, %lo(__xray_FunctionEntry/Exit)`.
- **CN**: 注释说明了 `addi ra, ra, %lo(__xray_FunctionEntry/Exit)`。

### Line 115
````cpp
  //    add ra, t1, ra
````
- **EN**: Comment documenting `add ra, t1, ra`.
- **CN**: 注释说明了 `add ra, t1, ra`。

### Line 116
````cpp
  //    lui a0, %hi(function_id)
````
- **EN**: Comment documenting `lui a0, %hi(function_id)`.
- **CN**: 注释说明了 `lui a0, %hi(function_id)`。

### Line 117
````cpp
  //    addi a0, a0, %lo(function_id)                   ;pass function id
````
- **EN**: Comment documenting `addi a0, a0, %lo(function_id)                   ;pass function id`.
- **CN**: 注释说明了 `addi a0, a0, %lo(function_id)                   ;pass function id`。

### Line 118
````cpp
  //    jalr ra                                         ;call Tracing hook
````
- **EN**: Comment documenting `jalr ra                                         ;call Tracing hook`.
- **CN**: 注释说明了 `jalr ra                                         ;call Tracing hook`。

### Line 119
````cpp
  //    ld t1, 8(sp)                                    ;restore register t1
````
- **EN**: Comment documenting `ld t1, 8(sp)                                    ;restore register t1`.
- **CN**: 注释说明了 `ld t1, 8(sp)                                    ;restore register t1`。

### Line 120
````cpp
  //    ld a0, 16(sp)                                   ;restore register a0
````
- **EN**: Comment documenting `ld a0, 16(sp)                                   ;restore register a0`.
- **CN**: 注释说明了 `ld a0, 16(sp)                                   ;restore register a0`。

### Line 121
````cpp
  //    ld ra, 24(sp)                                   ;restore return address
````
- **EN**: Comment documenting `ld ra, 24(sp)                                   ;restore return address`.
- **CN**: 注释说明了 `ld ra, 24(sp)                                   ;restore return address`。

### Line 122
````cpp
  //    addi sp, sp, 32                                 ;delete stack frame
````
- **EN**: Comment documenting `addi sp, sp, 32                                 ;delete stack frame`.
- **CN**: 注释说明了 `addi sp, sp, 32                                 ;delete stack frame`。

### Line 123
````cpp
  //
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 124
````cpp
  // Replacement of the first 4-byte instruction should be the last and atomic
````
- **EN**: Comment documenting `Replacement of the first 4-byte instruction should be the last and atomic`.
- **CN**: 注释说明了 `Replacement of the first 4-byte instruction should be the last and atomic`。

### Line 125
````cpp
  // operation, so that the user code which reaches the sled concurrently
````
- **EN**: Comment documenting `operation, so that the user code which reaches the sled concurrently`.
- **CN**: 注释说明了 `operation, so that the user code which reaches the sled concurrently`。

### Line 126
````cpp
  // either jumps over the whole sled, or executes the whole sled when the
````
- **EN**: Comment documenting `either jumps over the whole sled, or executes the whole sled when the`.
- **CN**: 注释说明了 `either jumps over the whole sled, or executes the whole sled when the`。

### Line 127
````cpp
  // latter is ready.
````
- **EN**: Comment documenting `latter is ready.`.
- **CN**: 注释说明了 `latter is ready.`。

### Line 128
````cpp
  //
````
- **EN**: Comment formatting line in a banner or block comment.
- **CN**: 注释格式行，用于组织文件头或注释块。

### Line 129
````cpp
  // When |Enable|==false, we set back the first instruction in the sled to be
````
- **EN**: Comment documenting `When |Enable|==false, we set back the first instruction in the sled to be`.
- **CN**: 注释说明了 `When |Enable|==false, we set back the first instruction in the sled to be`。

### Line 130
````cpp
  //   J 44 bytes (rv32)
````
- **EN**: Comment documenting `J 44 bytes (rv32)`.
- **CN**: 注释说明了 `J 44 bytes (rv32)`。

### Line 131
````cpp
  //   J 68 bytes (rv64)
````
- **EN**: Comment documenting `J 68 bytes (rv64)`.
- **CN**: 注释说明了 `J 68 bytes (rv64)`。

### Line 132
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 133
````cpp
  uint32_t *Address = reinterpret_cast<uint32_t *>(Sled.address());
````
- **EN**: Declares an interface element or prototype: `uint32_t *Address = reinterpret_cast<uint32_t *>(Sled.address());`.
- **CN**: 声明一个接口元素或原型：`uint32_t *Address = reinterpret_cast<uint32_t *>(Sled.address());`。

### Line 134
````cpp
  if (Enable) {
````
- **EN**: Evaluates the conditional branch `if (Enable) {`.
- **CN**: 计算条件分支 `if (Enable) {`。

### Line 135
````cpp
#if __riscv_xlen == 64
````
- **EN**: Starts a preprocessor condition: `#if __riscv_xlen == 64`.
- **CN**: 开始一个预处理条件：`#if __riscv_xlen == 64`。

### Line 136
````cpp
    // If the ISA is RV64, the Tracing Hook needs to be typecast to a 64 bit
````
- **EN**: Comment documenting `If the ISA is RV64, the Tracing Hook needs to be typecast to a 64 bit`.
- **CN**: 注释说明了 `If the ISA is RV64, the Tracing Hook needs to be typecast to a 64 bit`。

### Line 137
````cpp
    // value.
````
- **EN**: Comment documenting `value.`.
- **CN**: 注释说明了 `value.`。

### Line 138
````cpp
    uint32_t LoTracingHookAddr = lo12(reinterpret_cast<uint64_t>(TracingHook));
````
- **EN**: Declares an interface element or prototype: `uint32_t LoTracingHookAddr = lo12(reinterpret_cast<uint64_t>(TracingHook));`.
- **CN**: 声明一个接口元素或原型：`uint32_t LoTracingHookAddr = lo12(reinterpret_cast<uint64_t>(TracingHook));`。

### Line 139
````cpp
    uint32_t HiTracingHookAddr = hi20(reinterpret_cast<uint64_t>(TracingHook));
````
- **EN**: Declares an interface element or prototype: `uint32_t HiTracingHookAddr = hi20(reinterpret_cast<uint64_t>(TracingHook));`.
- **CN**: 声明一个接口元素或原型：`uint32_t HiTracingHookAddr = hi20(reinterpret_cast<uint64_t>(TracingHook));`。

### Line 140
````cpp
    uint32_t HigherTracingHookAddr =
````
- **EN**: Carries part of the local implementation logic: `uint32_t HigherTracingHookAddr =`.
- **CN**: 承载局部实现逻辑：`uint32_t HigherTracingHookAddr =`。

### Line 141
````cpp
        lo12((reinterpret_cast<uint64_t>(TracingHook) + 0x80000000) >> 32);
````
- **EN**: Invokes a function-like statement: `lo12((reinterpret_cast<uint64_t>(TracingHook) + 0x80000000) >> 32);`.
- **CN**: 调用一个类似函数的语句：`lo12((reinterpret_cast<uint64_t>(TracingHook) + 0x80000000) >> 32);`。

### Line 142
````cpp
    uint32_t HighestTracingHookAddr =
````
- **EN**: Carries part of the local implementation logic: `uint32_t HighestTracingHookAddr =`.
- **CN**: 承载局部实现逻辑：`uint32_t HighestTracingHookAddr =`。

### Line 143
````cpp
        hi20((reinterpret_cast<uint64_t>(TracingHook) + 0x80000000) >> 32);
````
- **EN**: Invokes a function-like statement: `hi20((reinterpret_cast<uint64_t>(TracingHook) + 0x80000000) >> 32);`.
- **CN**: 调用一个类似函数的语句：`hi20((reinterpret_cast<uint64_t>(TracingHook) + 0x80000000) >> 32);`。

### Line 144
````cpp
#elif __riscv_xlen == 32
````
- **EN**: Checks an alternate preprocessor branch: `#elif __riscv_xlen == 32`.
- **CN**: 检查预处理器的备用分支：`#elif __riscv_xlen == 32`。

### Line 145
````cpp
    // We typecast the Tracing Hook to a 32 bit value for RV32
````
- **EN**: Comment documenting `We typecast the Tracing Hook to a 32 bit value for RV32`.
- **CN**: 注释说明了 `We typecast the Tracing Hook to a 32 bit value for RV32`。

### Line 146
````cpp
    uint32_t LoTracingHookAddr = lo12(reinterpret_cast<uint32_t>(TracingHook));
````
- **EN**: Declares an interface element or prototype: `uint32_t LoTracingHookAddr = lo12(reinterpret_cast<uint32_t>(TracingHook));`.
- **CN**: 声明一个接口元素或原型：`uint32_t LoTracingHookAddr = lo12(reinterpret_cast<uint32_t>(TracingHook));`。

### Line 147
````cpp
    uint32_t HiTracingHookAddr = hi20((reinterpret_cast<uint32_t>(TracingHook));
````
- **EN**: Declares an interface element or prototype: `uint32_t HiTracingHookAddr = hi20((reinterpret_cast<uint32_t>(TracingHook));`.
- **CN**: 声明一个接口元素或原型：`uint32_t HiTracingHookAddr = hi20((reinterpret_cast<uint32_t>(TracingHook));`。

### Line 148
````cpp
#endif
````
- **EN**: Ends the current preprocessor condition.
- **CN**: 结束当前的预处理条件。

### Line 149
````cpp
    uint32_t LoFunctionID = lo12(FuncId);
````
- **EN**: Declares an interface element or prototype: `uint32_t LoFunctionID = lo12(FuncId);`.
- **CN**: 声明一个接口元素或原型：`uint32_t LoFunctionID = lo12(FuncId);`。

### Line 150
````cpp
    uint32_t HiFunctionID = hi20(FuncId);
````
- **EN**: Declares an interface element or prototype: `uint32_t HiFunctionID = hi20(FuncId);`.
- **CN**: 声明一个接口元素或原型：`uint32_t HiFunctionID = hi20(FuncId);`。

### Line 151
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 152
````cpp
    // The sled that is patched in for RISCV64 defined below. We need the entire
````
- **EN**: Comment documenting `The sled that is patched in for RISCV64 defined below. We need the entire`.
- **CN**: 注释说明了 `The sled that is patched in for RISCV64 defined below. We need the entire`。

### Line 153
````cpp
    // sleds corresponding to both ISAs to be protected by defines because the
````
- **EN**: Comment documenting `sleds corresponding to both ISAs to be protected by defines because the`.
- **CN**: 注释说明了 `sleds corresponding to both ISAs to be protected by defines because the`。

### Line 154
````cpp
    // first few instructions are all different, because we store doubles in
````
- **EN**: Comment documenting `first few instructions are all different, because we store doubles in`.
- **CN**: 注释说明了 `first few instructions are all different, because we store doubles in`。

### Line 155
````cpp
    // case of RV64 and store words for RV32. Subsequently, we have LUI - and in
````
- **EN**: Comment documenting `case of RV64 and store words for RV32. Subsequently, we have LUI - and in`.
- **CN**: 注释说明了 `case of RV64 and store words for RV32. Subsequently, we have LUI - and in`。

### Line 156
````cpp
    // case of RV64, we need extra instructions from this point on, so we see
````
- **EN**: Comment documenting `case of RV64, we need extra instructions from this point on, so we see`.
- **CN**: 注释说明了 `case of RV64, we need extra instructions from this point on, so we see`。

### Line 157
````cpp
    // differences in addresses to which instructions are stored.
````
- **EN**: Comment documenting `differences in addresses to which instructions are stored.`.
- **CN**: 注释说明了 `differences in addresses to which instructions are stored.`。

### Line 158
````cpp
    size_t Idx = 1U;
````
- **EN**: Assigns or initializes state with `size_t Idx = 1U;`.
- **CN**: 使用 `size_t Idx = 1U;` 进行赋值或初始化。

### Line 159
````cpp
    const uint32_t XLenBytes = __riscv_xlen / 8;
````
- **EN**: Assigns or initializes state with `const uint32_t XLenBytes = __riscv_xlen / 8;`.
- **CN**: 使用 `const uint32_t XLenBytes = __riscv_xlen / 8;` 进行赋值或初始化。

### Line 160
````cpp
#if __riscv_xlen == 64
````
- **EN**: Starts a preprocessor condition: `#if __riscv_xlen == 64`.
- **CN**: 开始一个预处理条件：`#if __riscv_xlen == 64`。

### Line 161
````cpp
    const uint32_t LoadOp = PatchOpcodes::PO_LD;
````
- **EN**: Assigns or initializes state with `const uint32_t LoadOp = PatchOpcodes::PO_LD;`.
- **CN**: 使用 `const uint32_t LoadOp = PatchOpcodes::PO_LD;` 进行赋值或初始化。

### Line 162
````cpp
    const uint32_t StoreOp = PatchOpcodes::PO_SD;
````
- **EN**: Assigns or initializes state with `const uint32_t StoreOp = PatchOpcodes::PO_SD;`.
- **CN**: 使用 `const uint32_t StoreOp = PatchOpcodes::PO_SD;` 进行赋值或初始化。

### Line 163
````cpp
#elif __riscv_xlen == 32
````
- **EN**: Checks an alternate preprocessor branch: `#elif __riscv_xlen == 32`.
- **CN**: 检查预处理器的备用分支：`#elif __riscv_xlen == 32`。

### Line 164
````cpp
    const uint32_t LoadOp = PatchOpcodes::PO_LW;
````
- **EN**: Assigns or initializes state with `const uint32_t LoadOp = PatchOpcodes::PO_LW;`.
- **CN**: 使用 `const uint32_t LoadOp = PatchOpcodes::PO_LW;` 进行赋值或初始化。

### Line 165
````cpp
    const uint32_t StoreOp = PatchOpcodes::PO_SW;
````
- **EN**: Assigns or initializes state with `const uint32_t StoreOp = PatchOpcodes::PO_SW;`.
- **CN**: 使用 `const uint32_t StoreOp = PatchOpcodes::PO_SW;` 进行赋值或初始化。

### Line 166
````cpp
#endif
````
- **EN**: Ends the current preprocessor condition.
- **CN**: 结束当前的预处理条件。

### Line 167
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 168
````cpp
    Address[Idx++] = encodeSTypeInstruction(StoreOp, RegNum::RN_SP,
````
- **EN**: Carries part of the local implementation logic: `Address[Idx++] = encodeSTypeInstruction(StoreOp, RegNum::RN_SP,`.
- **CN**: 承载局部实现逻辑：`Address[Idx++] = encodeSTypeInstruction(StoreOp, RegNum::RN_SP,`。

### Line 169
````cpp
                                            RegNum::RN_RA, 3 * XLenBytes);
````
- **EN**: Executes or declares `RegNum::RN_RA, 3 * XLenBytes);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `RegNum::RN_RA, 3 * XLenBytes);`。

### Line 170
````cpp
    Address[Idx++] = encodeSTypeInstruction(StoreOp, RegNum::RN_SP,
````
- **EN**: Carries part of the local implementation logic: `Address[Idx++] = encodeSTypeInstruction(StoreOp, RegNum::RN_SP,`.
- **CN**: 承载局部实现逻辑：`Address[Idx++] = encodeSTypeInstruction(StoreOp, RegNum::RN_SP,`。

### Line 171
````cpp
                                            RegNum::RN_A0, 2 * XLenBytes);
````
- **EN**: Executes or declares `RegNum::RN_A0, 2 * XLenBytes);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `RegNum::RN_A0, 2 * XLenBytes);`。

### Line 172
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 173
````cpp
#if __riscv_xlen == 64
````
- **EN**: Starts a preprocessor condition: `#if __riscv_xlen == 64`.
- **CN**: 开始一个预处理条件：`#if __riscv_xlen == 64`。

### Line 174
````cpp
    Address[Idx++] = encodeSTypeInstruction(StoreOp, RegNum::RN_SP,
````
- **EN**: Carries part of the local implementation logic: `Address[Idx++] = encodeSTypeInstruction(StoreOp, RegNum::RN_SP,`.
- **CN**: 承载局部实现逻辑：`Address[Idx++] = encodeSTypeInstruction(StoreOp, RegNum::RN_SP,`。

### Line 175
````cpp
                                            RegNum::RN_T1, XLenBytes);
````
- **EN**: Executes or declares `RegNum::RN_T1, XLenBytes);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `RegNum::RN_T1, XLenBytes);`。

### Line 176
````cpp
    Address[Idx++] = encodeUTypeInstruction(PatchOpcodes::PO_LUI, RegNum::RN_T1,
````
- **EN**: Carries part of the local implementation logic: `Address[Idx++] = encodeUTypeInstruction(PatchOpcodes::PO_LUI, RegNum::RN_T1,`.
- **CN**: 承载局部实现逻辑：`Address[Idx++] = encodeUTypeInstruction(PatchOpcodes::PO_LUI, RegNum::RN_T1,`。

### Line 177
````cpp
                                            HighestTracingHookAddr);
````
- **EN**: Executes or declares `HighestTracingHookAddr);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `HighestTracingHookAddr);`。

### Line 178
````cpp
    Address[Idx++] =
````
- **EN**: Carries part of the local implementation logic: `Address[Idx++] =`.
- **CN**: 承载局部实现逻辑：`Address[Idx++] =`。

### Line 179
````cpp
        encodeITypeInstruction(PatchOpcodes::PO_ADDI, RegNum::RN_T1,
````
- **EN**: Carries part of the local implementation logic: `encodeITypeInstruction(PatchOpcodes::PO_ADDI, RegNum::RN_T1,`.
- **CN**: 承载局部实现逻辑：`encodeITypeInstruction(PatchOpcodes::PO_ADDI, RegNum::RN_T1,`。

### Line 180
````cpp
                               RegNum::RN_T1, HigherTracingHookAddr);
````
- **EN**: Executes or declares `RegNum::RN_T1, HigherTracingHookAddr);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `RegNum::RN_T1, HigherTracingHookAddr);`。

### Line 181
````cpp
    Address[Idx++] = encodeITypeInstruction(PatchOpcodes::PO_SLLI,
````
- **EN**: Carries part of the local implementation logic: `Address[Idx++] = encodeITypeInstruction(PatchOpcodes::PO_SLLI,`.
- **CN**: 承载局部实现逻辑：`Address[Idx++] = encodeITypeInstruction(PatchOpcodes::PO_SLLI,`。

### Line 182
````cpp
                                            RegNum::RN_T1, RegNum::RN_T1, 32);
````
- **EN**: Executes or declares `RegNum::RN_T1, RegNum::RN_T1, 32);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `RegNum::RN_T1, RegNum::RN_T1, 32);`。

### Line 183
````cpp
#endif
````
- **EN**: Ends the current preprocessor condition.
- **CN**: 结束当前的预处理条件。

### Line 184
````cpp
    Address[Idx++] = encodeUTypeInstruction(PatchOpcodes::PO_LUI, RegNum::RN_RA,
````
- **EN**: Carries part of the local implementation logic: `Address[Idx++] = encodeUTypeInstruction(PatchOpcodes::PO_LUI, RegNum::RN_RA,`.
- **CN**: 承载局部实现逻辑：`Address[Idx++] = encodeUTypeInstruction(PatchOpcodes::PO_LUI, RegNum::RN_RA,`。

### Line 185
````cpp
                                            HiTracingHookAddr);
````
- **EN**: Executes or declares `HiTracingHookAddr);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `HiTracingHookAddr);`。

### Line 186
````cpp
    Address[Idx++] = encodeITypeInstruction(
````
- **EN**: Carries part of the local implementation logic: `Address[Idx++] = encodeITypeInstruction(`.
- **CN**: 承载局部实现逻辑：`Address[Idx++] = encodeITypeInstruction(`。

### Line 187
````cpp
        PatchOpcodes::PO_ADDI, RegNum::RN_RA, RegNum::RN_RA, LoTracingHookAddr);
````
- **EN**: Executes or declares `PatchOpcodes::PO_ADDI, RegNum::RN_RA, RegNum::RN_RA, LoTracingHookAddr);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `PatchOpcodes::PO_ADDI, RegNum::RN_RA, RegNum::RN_RA, LoTracingHookAddr);`。

### Line 188
````cpp
#if __riscv_xlen == 64
````
- **EN**: Starts a preprocessor condition: `#if __riscv_xlen == 64`.
- **CN**: 开始一个预处理条件：`#if __riscv_xlen == 64`。

### Line 189
````cpp
    Address[Idx++] = encodeRTypeInstruction(PatchOpcodes::PO_ADD, RegNum::RN_RA,
````
- **EN**: Carries part of the local implementation logic: `Address[Idx++] = encodeRTypeInstruction(PatchOpcodes::PO_ADD, RegNum::RN_RA,`.
- **CN**: 承载局部实现逻辑：`Address[Idx++] = encodeRTypeInstruction(PatchOpcodes::PO_ADD, RegNum::RN_RA,`。

### Line 190
````cpp
                                            RegNum::RN_T1, RegNum::RN_RA);
````
- **EN**: Executes or declares `RegNum::RN_T1, RegNum::RN_RA);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `RegNum::RN_T1, RegNum::RN_RA);`。

### Line 191
````cpp
#endif
````
- **EN**: Ends the current preprocessor condition.
- **CN**: 结束当前的预处理条件。

### Line 192
````cpp
    Address[Idx++] = encodeUTypeInstruction(PatchOpcodes::PO_LUI, RegNum::RN_A0,
````
- **EN**: Carries part of the local implementation logic: `Address[Idx++] = encodeUTypeInstruction(PatchOpcodes::PO_LUI, RegNum::RN_A0,`.
- **CN**: 承载局部实现逻辑：`Address[Idx++] = encodeUTypeInstruction(PatchOpcodes::PO_LUI, RegNum::RN_A0,`。

### Line 193
````cpp
                                            HiFunctionID);
````
- **EN**: Executes or declares `HiFunctionID);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `HiFunctionID);`。

### Line 194
````cpp
    Address[Idx++] = encodeITypeInstruction(
````
- **EN**: Carries part of the local implementation logic: `Address[Idx++] = encodeITypeInstruction(`.
- **CN**: 承载局部实现逻辑：`Address[Idx++] = encodeITypeInstruction(`。

### Line 195
````cpp
        PatchOpcodes::PO_ADDI, RegNum::RN_A0, RegNum::RN_A0, LoFunctionID);
````
- **EN**: Executes or declares `PatchOpcodes::PO_ADDI, RegNum::RN_A0, RegNum::RN_A0, LoFunctionID);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `PatchOpcodes::PO_ADDI, RegNum::RN_A0, RegNum::RN_A0, LoFunctionID);`。

### Line 196
````cpp
    Address[Idx++] = encodeITypeInstruction(PatchOpcodes::PO_JALR,
````
- **EN**: Carries part of the local implementation logic: `Address[Idx++] = encodeITypeInstruction(PatchOpcodes::PO_JALR,`.
- **CN**: 承载局部实现逻辑：`Address[Idx++] = encodeITypeInstruction(PatchOpcodes::PO_JALR,`。

### Line 197
````cpp
                                            RegNum::RN_RA, RegNum::RN_RA, 0);
````
- **EN**: Executes or declares `RegNum::RN_RA, RegNum::RN_RA, 0);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `RegNum::RN_RA, RegNum::RN_RA, 0);`。

### Line 198
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 199
````cpp
#if __riscv_xlen == 64
````
- **EN**: Starts a preprocessor condition: `#if __riscv_xlen == 64`.
- **CN**: 开始一个预处理条件：`#if __riscv_xlen == 64`。

### Line 200
````cpp
    Address[Idx++] =
````
- **EN**: Carries part of the local implementation logic: `Address[Idx++] =`.
- **CN**: 承载局部实现逻辑：`Address[Idx++] =`。

### Line 201
````cpp
        encodeITypeInstruction(LoadOp, RegNum::RN_SP, RegNum::RN_T1, XLenBytes);
````
- **EN**: Declares an interface element or prototype: `encodeITypeInstruction(LoadOp, RegNum::RN_SP, RegNum::RN_T1, XLenBytes);`.
- **CN**: 声明一个接口元素或原型：`encodeITypeInstruction(LoadOp, RegNum::RN_SP, RegNum::RN_T1, XLenBytes);`。

### Line 202
````cpp
#endif
````
- **EN**: Ends the current preprocessor condition.
- **CN**: 结束当前的预处理条件。

### Line 203
````cpp
    Address[Idx++] = encodeITypeInstruction(LoadOp, RegNum::RN_SP,
````
- **EN**: Carries part of the local implementation logic: `Address[Idx++] = encodeITypeInstruction(LoadOp, RegNum::RN_SP,`.
- **CN**: 承载局部实现逻辑：`Address[Idx++] = encodeITypeInstruction(LoadOp, RegNum::RN_SP,`。

### Line 204
````cpp
                                            RegNum::RN_A0, 2 * XLenBytes);
````
- **EN**: Executes or declares `RegNum::RN_A0, 2 * XLenBytes);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `RegNum::RN_A0, 2 * XLenBytes);`。

### Line 205
````cpp
    Address[Idx++] = encodeITypeInstruction(LoadOp, RegNum::RN_SP,
````
- **EN**: Carries part of the local implementation logic: `Address[Idx++] = encodeITypeInstruction(LoadOp, RegNum::RN_SP,`.
- **CN**: 承载局部实现逻辑：`Address[Idx++] = encodeITypeInstruction(LoadOp, RegNum::RN_SP,`。

### Line 206
````cpp
                                            RegNum::RN_RA, 3 * XLenBytes);
````
- **EN**: Executes or declares `RegNum::RN_RA, 3 * XLenBytes);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `RegNum::RN_RA, 3 * XLenBytes);`。

### Line 207
````cpp
    Address[Idx++] = encodeITypeInstruction(
````
- **EN**: Carries part of the local implementation logic: `Address[Idx++] = encodeITypeInstruction(`.
- **CN**: 承载局部实现逻辑：`Address[Idx++] = encodeITypeInstruction(`。

### Line 208
````cpp
        PatchOpcodes::PO_ADDI, RegNum::RN_SP, RegNum::RN_SP, 4 * XLenBytes);
````
- **EN**: Executes or declares `PatchOpcodes::PO_ADDI, RegNum::RN_SP, RegNum::RN_SP, 4 * XLenBytes);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `PatchOpcodes::PO_ADDI, RegNum::RN_SP, RegNum::RN_SP, 4 * XLenBytes);`。

### Line 209
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 210
````cpp
    uint32_t CreateStackSpace = encodeITypeInstruction(
````
- **EN**: Carries part of the local implementation logic: `uint32_t CreateStackSpace = encodeITypeInstruction(`.
- **CN**: 承载局部实现逻辑：`uint32_t CreateStackSpace = encodeITypeInstruction(`。

### Line 211
````cpp
        PatchOpcodes::PO_ADDI, RegNum::RN_SP, RegNum::RN_SP, -4 * XLenBytes);
````
- **EN**: Executes or declares `PatchOpcodes::PO_ADDI, RegNum::RN_SP, RegNum::RN_SP, -4 * XLenBytes);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `PatchOpcodes::PO_ADDI, RegNum::RN_SP, RegNum::RN_SP, -4 * XLenBytes);`。

### Line 212
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 213
````cpp
    std::atomic_store_explicit(
````
- **EN**: Carries part of the local implementation logic: `std::atomic_store_explicit(`.
- **CN**: 承载局部实现逻辑：`std::atomic_store_explicit(`。

### Line 214
````cpp
        reinterpret_cast<std::atomic<uint32_t> *>(Address), CreateStackSpace,
````
- **EN**: Carries part of the local implementation logic: `reinterpret_cast<std::atomic<uint32_t> *>(Address), CreateStackSpace,`.
- **CN**: 承载局部实现逻辑：`reinterpret_cast<std::atomic<uint32_t> *>(Address), CreateStackSpace,`。

### Line 215
````cpp
        std::memory_order_release);
````
- **EN**: Executes or declares `std::memory_order_release);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `std::memory_order_release);`。

### Line 216
````cpp
  } else {
````
- **EN**: Carries part of the local implementation logic: `} else {`.
- **CN**: 承载局部实现逻辑：`} else {`。

### Line 217
````cpp
    uint32_t CreateBranch = encodeJTypeInstruction(
````
- **EN**: Carries part of the local implementation logic: `uint32_t CreateBranch = encodeJTypeInstruction(`.
- **CN**: 承载局部实现逻辑：`uint32_t CreateBranch = encodeJTypeInstruction(`。

### Line 218
````cpp
    // Jump distance is different in both ISAs due to difference in size of
````
- **EN**: Comment documenting `Jump distance is different in both ISAs due to difference in size of`.
- **CN**: 注释说明了 `Jump distance is different in both ISAs due to difference in size of`。

### Line 219
````cpp
    // sleds
````
- **EN**: Comment documenting `sleds`.
- **CN**: 注释说明了 `sleds`。

### Line 220
````cpp
#if __riscv_xlen == 64
````
- **EN**: Starts a preprocessor condition: `#if __riscv_xlen == 64`.
- **CN**: 开始一个预处理条件：`#if __riscv_xlen == 64`。

### Line 221
````cpp
        PatchOpcodes::PO_J, RegNum::RN_X0,
````
- **EN**: Carries part of the local implementation logic: `PatchOpcodes::PO_J, RegNum::RN_X0,`.
- **CN**: 承载局部实现逻辑：`PatchOpcodes::PO_J, RegNum::RN_X0,`。

### Line 222
````cpp
        68); // jump encodes an offset of 68
````
- **EN**: Carries part of the local implementation logic: `68); // jump encodes an offset of 68`.
- **CN**: 承载局部实现逻辑：`68); // jump encodes an offset of 68`。

### Line 223
````cpp
#elif __riscv_xlen == 32
````
- **EN**: Checks an alternate preprocessor branch: `#elif __riscv_xlen == 32`.
- **CN**: 检查预处理器的备用分支：`#elif __riscv_xlen == 32`。

### Line 224
````cpp
        PatchOpcodes::PO_J, RegNum::RN_X0,
````
- **EN**: Carries part of the local implementation logic: `PatchOpcodes::PO_J, RegNum::RN_X0,`.
- **CN**: 承载局部实现逻辑：`PatchOpcodes::PO_J, RegNum::RN_X0,`。

### Line 225
````cpp
        44); // jump encodes an offset of 44
````
- **EN**: Carries part of the local implementation logic: `44); // jump encodes an offset of 44`.
- **CN**: 承载局部实现逻辑：`44); // jump encodes an offset of 44`。

### Line 226
````cpp
#endif
````
- **EN**: Ends the current preprocessor condition.
- **CN**: 结束当前的预处理条件。

### Line 227
````cpp
    std::atomic_store_explicit(
````
- **EN**: Carries part of the local implementation logic: `std::atomic_store_explicit(`.
- **CN**: 承载局部实现逻辑：`std::atomic_store_explicit(`。

### Line 228
````cpp
        reinterpret_cast<std::atomic<uint32_t> *>(Address), CreateBranch,
````
- **EN**: Carries part of the local implementation logic: `reinterpret_cast<std::atomic<uint32_t> *>(Address), CreateBranch,`.
- **CN**: 承载局部实现逻辑：`reinterpret_cast<std::atomic<uint32_t> *>(Address), CreateBranch,`。

### Line 229
````cpp
        std::memory_order_release);
````
- **EN**: Executes or declares `std::memory_order_release);` within the current scope.
- **CN**: 在当前作用域中执行或声明 `std::memory_order_release);`。

### Line 230
````cpp
  }
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 231
````cpp
  return true;
````
- **EN**: Returns from the current function with `true;`.
- **CN**: 使用 `true;` 从当前函数返回。

### Line 232
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 233
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 234
````cpp
bool patchFunctionEntry(const bool Enable, const uint32_t FuncId,
````
- **EN**: Carries part of the local implementation logic: `bool patchFunctionEntry(const bool Enable, const uint32_t FuncId,`.
- **CN**: 承载局部实现逻辑：`bool patchFunctionEntry(const bool Enable, const uint32_t FuncId,`。

### Line 235
````cpp
                        const XRaySledEntry &Sled,
````
- **EN**: Carries part of the local implementation logic: `const XRaySledEntry &Sled,`.
- **CN**: 承载局部实现逻辑：`const XRaySledEntry &Sled,`。

### Line 236
````cpp
                        const XRayTrampolines &Trampolines,
````
- **EN**: Carries part of the local implementation logic: `const XRayTrampolines &Trampolines,`.
- **CN**: 承载局部实现逻辑：`const XRayTrampolines &Trampolines,`。

### Line 237
````cpp
                        bool LogArgs) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `bool LogArgs) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`bool LogArgs) XRAY_NEVER_INSTRUMENT {`。

### Line 238
````cpp
  // We don't support logging argument at this moment, so we always
````
- **EN**: Comment documenting `We don't support logging argument at this moment, so we always`.
- **CN**: 注释说明了 `We don't support logging argument at this moment, so we always`。

### Line 239
````cpp
  // use EntryTrampoline.
````
- **EN**: Comment documenting `use EntryTrampoline.`.
- **CN**: 注释说明了 `use EntryTrampoline.`。

### Line 240
````cpp
  return patchSled(Enable, FuncId, Sled, Trampolines.EntryTrampoline);
````
- **EN**: Returns from the current function with `patchSled(Enable, FuncId, Sled, Trampolines.EntryTrampoline);`.
- **CN**: 使用 `patchSled(Enable, FuncId, Sled, Trampolines.EntryTrampoline);` 从当前函数返回。

### Line 241
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 242
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 243
````cpp
bool patchFunctionExit(
````
- **EN**: Carries part of the local implementation logic: `bool patchFunctionExit(`.
- **CN**: 承载局部实现逻辑：`bool patchFunctionExit(`。

### Line 244
````cpp
    const bool Enable, const uint32_t FuncId, const XRaySledEntry &Sled,
````
- **EN**: Carries part of the local implementation logic: `const bool Enable, const uint32_t FuncId, const XRaySledEntry &Sled,`.
- **CN**: 承载局部实现逻辑：`const bool Enable, const uint32_t FuncId, const XRaySledEntry &Sled,`。

### Line 245
````cpp
    const XRayTrampolines &Trampolines) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `const XRayTrampolines &Trampolines) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`const XRayTrampolines &Trampolines) XRAY_NEVER_INSTRUMENT {`。

### Line 246
````cpp
  return patchSled(Enable, FuncId, Sled, Trampolines.ExitTrampoline);
````
- **EN**: Returns from the current function with `patchSled(Enable, FuncId, Sled, Trampolines.ExitTrampoline);`.
- **CN**: 使用 `patchSled(Enable, FuncId, Sled, Trampolines.ExitTrampoline);` 从当前函数返回。

### Line 247
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 248
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 249
````cpp
bool patchFunctionTailExit(
````
- **EN**: Carries part of the local implementation logic: `bool patchFunctionTailExit(`.
- **CN**: 承载局部实现逻辑：`bool patchFunctionTailExit(`。

### Line 250
````cpp
    const bool Enable, const uint32_t FuncId, const XRaySledEntry &Sled,
````
- **EN**: Carries part of the local implementation logic: `const bool Enable, const uint32_t FuncId, const XRaySledEntry &Sled,`.
- **CN**: 承载局部实现逻辑：`const bool Enable, const uint32_t FuncId, const XRaySledEntry &Sled,`。

### Line 251
````cpp
    const XRayTrampolines &Trampolines) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `const XRayTrampolines &Trampolines) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`const XRayTrampolines &Trampolines) XRAY_NEVER_INSTRUMENT {`。

### Line 252
````cpp
  return patchSled(Enable, FuncId, Sled, Trampolines.TailExitTrampoline);
````
- **EN**: Returns from the current function with `patchSled(Enable, FuncId, Sled, Trampolines.TailExitTrampoline);`.
- **CN**: 使用 `patchSled(Enable, FuncId, Sled, Trampolines.TailExitTrampoline);` 从当前函数返回。

### Line 253
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 254
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 255
````cpp
bool patchCustomEvent(const bool Enable, const uint32_t FuncId,
````
- **EN**: Carries part of the local implementation logic: `bool patchCustomEvent(const bool Enable, const uint32_t FuncId,`.
- **CN**: 承载局部实现逻辑：`bool patchCustomEvent(const bool Enable, const uint32_t FuncId,`。

### Line 256
````cpp
                      const XRaySledEntry &Sled) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `const XRaySledEntry &Sled) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`const XRaySledEntry &Sled) XRAY_NEVER_INSTRUMENT {`。

### Line 257
````cpp
  return false;
````
- **EN**: Returns from the current function with `false;`.
- **CN**: 使用 `false;` 从当前函数返回。

### Line 258
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 259
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 260
````cpp
bool patchTypedEvent(const bool Enable, const uint32_t FuncId,
````
- **EN**: Carries part of the local implementation logic: `bool patchTypedEvent(const bool Enable, const uint32_t FuncId,`.
- **CN**: 承载局部实现逻辑：`bool patchTypedEvent(const bool Enable, const uint32_t FuncId,`。

### Line 261
````cpp
                     const XRaySledEntry &Sled) XRAY_NEVER_INSTRUMENT {
````
- **EN**: Carries part of the local implementation logic: `const XRaySledEntry &Sled) XRAY_NEVER_INSTRUMENT {`.
- **CN**: 承载局部实现逻辑：`const XRaySledEntry &Sled) XRAY_NEVER_INSTRUMENT {`。

### Line 262
````cpp
  return false;
````
- **EN**: Returns from the current function with `false;`.
- **CN**: 使用 `false;` 从当前函数返回。

### Line 263
````cpp
}
````
- **EN**: Closes the current scope block.
- **CN**: 结束当前作用域块。

### Line 264
````cpp
} // namespace __xray
````
- **EN**: Closes namespace `__xray`.
- **CN**: 关闭命名空间 `__xray`。

### Line 265
````cpp
<blank>
````
- **EN**: Blank line separating logical sections.
- **CN**: 空行，用于分隔逻辑段落。

### Line 266
````cpp
extern "C" void __xray_ArgLoggerEntry() XRAY_NEVER_INSTRUMENT {}
````
- **EN**: Declares C linkage for the following interface: `extern "C" void __xray_ArgLoggerEntry() XRAY_NEVER_INSTRUMENT {}`.
- **CN**: 为后续接口声明 C 语言链接：`extern "C" void __xray_ArgLoggerEntry() XRAY_NEVER_INSTRUMENT {}`。

## Key Concepts / 关键概念
- Preprocessor control / 预处理控制
- Namespaces / 命名空间
- Types and interfaces / 类型与接口
- Function logic / 函数逻辑
- Threading and synchronization / 线程与同步

## Dependencies / 依赖关系
- **Local headers / 本地头文件**: `sanitizer_common/sanitizer_common.h`, `xray_defs.h`, `xray_interface_internal.h`
- **System headers / 系统头文件**: `atomic`
- **Compile-time conditions / 编译期条件**:
  - `#if __riscv_xlen == 64`
  - `#if __riscv_xlen == 64`
  - `#if __riscv_xlen == 64`
  - `#if __riscv_xlen == 64`
  - `#if __riscv_xlen == 64`
  - `#if __riscv_xlen == 64`
