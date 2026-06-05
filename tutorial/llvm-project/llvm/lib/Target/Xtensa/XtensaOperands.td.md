# XtensaOperands.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/XtensaOperands.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines operand classes, immediates, and parser/printer constraints in TableGen DSL.
  - **CN**: 使用 TableGen DSL 定义操作数类别、立即数以及解析/打印约束。
- **Note / 说明**:
  - **EN**: This file uses LLVM TableGen DSL, so many definitions are declarative records rather than executable C++ statements.
  - **CN**: 该文件使用 LLVM TableGen DSL，因此许多内容是声明式记录，而不是可执行的 C++ 语句。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```tablegen
//===- XtensaOperands.td - Xtensa instruction operands -------*- tblgen-*--===//
//
//                     The LLVM Compiler Infrastructure
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 10-17
```tablegen

// Immediate operands with a shared generic render method.
class ImmAsmOperand<string name> : AsmOperandClass {
  let Name = name;
  let RenderMethod = "addImmOperands";
  let DiagnosticType = !strconcat("Invalid", name);
}

```
- **EN**: Declares TableGen records such as `ImmAsmOperand`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `ImmAsmOperand` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 18-30
```tablegen
class Immediate<ValueType vt, code pred, string asmop>
  : Operand<vt>, ImmLeaf<vt, pred> {
  let PrintMethod = "print"#asmop;
  let ParserMatchClass = !cast<AsmOperandClass>(asmop);
}

// imm8 predicate - Immediate in the range [-128,127]
def Imm8_AsmOperand : ImmAsmOperand<"Imm8">;
def imm8 : Immediate<i32, [{ return Imm >= -128 && Imm <= 127; }], "Imm8_AsmOperand"> {
  let EncoderMethod = "getImm8OpValue";
  let DecoderMethod = "decodeImm8Operand";
}

```
- **EN**: Declares TableGen records such as `Immediate`, `Imm8_AsmOperand`, `imm8`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `Immediate`, `Imm8_AsmOperand`, `imm8` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 31-39
```tablegen
// imm8_sh8 predicate - Immediate in the range [-32768,32512] with (bits[7-0] == 0)
// imm8 value left shifted by 8 bits
def Imm8_sh8_AsmOperand : ImmAsmOperand<"Imm8_sh8">;
def imm8_sh8 : Immediate<i32, [{ return Imm >= -32768 && Imm <= 32512 && ((Imm & 0xFF) == 0); }],
                        "Imm8_sh8_AsmOperand"> {
  let EncoderMethod = "getImm8_sh8OpValue";
  let DecoderMethod = "decodeImm8_sh8Operand";
}

```
- **EN**: Declares TableGen records such as `Imm8_sh8_AsmOperand`, `imm8_sh8`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `Imm8_sh8_AsmOperand`, `imm8_sh8` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 40-46
```tablegen
// imm8n_7 predicate - Immediate in the range [-8,7]
def Imm8n_7_AsmOperand: ImmAsmOperand<"Imm8n_7">;
def imm8n_7: Immediate<i32, [{ return Imm >= -8 && Imm <= 7; }], "Imm8n_7_AsmOperand"> {
  let EncoderMethod = "getImm8n_7OpValue";
  let DecoderMethod = "decodeImm8n_7Operand";
}

```
- **EN**: Declares TableGen records such as `Imm8n_7_AsmOperand`, `imm8n_7`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `Imm8n_7_AsmOperand`, `imm8n_7` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 47-53
```tablegen
// imm64n_4n predicate - Immediate in the range [-64,-4]
def Imm64n_4n_AsmOperand: ImmAsmOperand<"Imm64n_4n">;
def imm64n_4n: Immediate<i32, [{ return Imm >= -64 && Imm <= -4; }], "Imm64n_4n_AsmOperand"> {
  let EncoderMethod = "getImm64n_4nOpValue";
  let DecoderMethod = "decodeImm64n_4nOperand";
}

```
- **EN**: Declares TableGen records such as `Imm64n_4n_AsmOperand`, `imm64n_4n`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `Imm64n_4n_AsmOperand`, `imm64n_4n` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 54-60
```tablegen
// imm12 predicate - Immediate in the range [-2048,2047]
def Imm12_AsmOperand : ImmAsmOperand<"Imm12">;
def imm12 : Immediate<i32, [{ return Imm >= -2048 && Imm <= 2047; }], "Imm12_AsmOperand"> {
  let EncoderMethod = "getImm12OpValue";
  let DecoderMethod = "decodeImm12Operand";
}

```
- **EN**: Declares TableGen records such as `Imm12_AsmOperand`, `imm12`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `Imm12_AsmOperand`, `imm12` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 61-67
```tablegen
// imm12m predicate - Immediate for MOV operation
def Imm12m_AsmOperand : ImmAsmOperand<"Imm12m">;
def imm12m : Immediate<i32, [{ return Imm >= -2048 && Imm <= 2047; }], "Imm12m_AsmOperand"> {
  let EncoderMethod = "getImm12OpValue";
  let DecoderMethod = "decodeImm12Operand";
}

```
- **EN**: Declares TableGen records such as `Imm12m_AsmOperand`, `imm12m`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `Imm12m_AsmOperand`, `imm12m` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 68-74
```tablegen
// uimm4 predicate - Immediate in the range [0,15]
def Uimm4_AsmOperand : ImmAsmOperand<"Uimm4">;
def uimm4 : Immediate<i32, [{ return Imm >= 0 && Imm <= 15; }], "Uimm4_AsmOperand"> {
  let EncoderMethod = "getUimm4OpValue";
  let DecoderMethod = "decodeUimm4Operand";
}

```
- **EN**: Declares TableGen records such as `Uimm4_AsmOperand`, `uimm4`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `Uimm4_AsmOperand`, `uimm4` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 75-81
```tablegen
// uimm5 predicate - Immediate in the range [0,31]
def Uimm5_AsmOperand : ImmAsmOperand<"Uimm5">;
def uimm5 : Immediate<i32, [{ return Imm >= 0 && Imm <= 31; }], "Uimm5_AsmOperand"> {
  let EncoderMethod = "getUimm5OpValue";
  let DecoderMethod = "decodeUimm5Operand";
}

```
- **EN**: Declares TableGen records such as `Uimm5_AsmOperand`, `uimm5`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `Uimm5_AsmOperand`, `uimm5` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 82-88
```tablegen
// imm1_16 predicate - Immediate in the range [1,16]
def Imm1_16_AsmOperand : ImmAsmOperand<"Imm1_16">;
def imm1_16 : Immediate<i32, [{ return Imm >= 1 && Imm <= 16; }], "Imm1_16_AsmOperand"> {
  let EncoderMethod = "getImm1_16OpValue";
  let DecoderMethod = "decodeImm1_16Operand";
}

```
- **EN**: Declares TableGen records such as `Imm1_16_AsmOperand`, `imm1_16`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `Imm1_16_AsmOperand`, `imm1_16` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 89-95
```tablegen
// imm1n_15 predicate - Immediate in the range [-1,15], except 0
def Imm1n_15_AsmOperand: ImmAsmOperand<"Imm1n_15">;
def imm1n_15: Immediate<i32, [{ return Imm >= -1 && Imm <= 15 && Imm != 0; }], "Imm1n_15_AsmOperand"> {
  let EncoderMethod = "getImm1n_15OpValue";
  let DecoderMethod = "decodeImm1n_15Operand";
}

```
- **EN**: Declares TableGen records such as `Imm1n_15_AsmOperand`, `imm1n_15`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `Imm1n_15_AsmOperand`, `imm1n_15` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 96-102
```tablegen
// imm32n_95 predicate - Immediate in the range [-32,95]
def Imm32n_95_AsmOperand: ImmAsmOperand<"Imm32n_95">;
def imm32n_95: Immediate<i32, [{ return Imm >= -32 && Imm <= 95; }], "Imm32n_95_AsmOperand"> {
  let EncoderMethod = "getImm32n_95OpValue";
  let DecoderMethod = "decodeImm32n_95Operand";
}

```
- **EN**: Declares TableGen records such as `Imm32n_95_AsmOperand`, `imm32n_95`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `Imm32n_95_AsmOperand`, `imm32n_95` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 103-109
```tablegen
// shimm1_31 predicate - Immediate in the range [1,31]
def Shimm1_31_AsmOperand : ImmAsmOperand<"Shimm1_31">;
def shimm1_31 : Immediate<i32, [{ return Imm >= 1 && Imm <= 31; }], "Shimm1_31_AsmOperand"> {
  let EncoderMethod = "getShimm1_31OpValue";
  let DecoderMethod = "decodeShimm1_31Operand";
}

```
- **EN**: Declares TableGen records such as `Shimm1_31_AsmOperand`, `shimm1_31`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `Shimm1_31_AsmOperand`, `shimm1_31` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 110-121
```tablegen
// Memory offset 0..255 for 8-bit memory accesses
def Offset8m8_AsmOperand : ImmAsmOperand<"Offset8m8">;
def offset8m8 : Immediate<i32,
    [{ return Imm >= 0 && Imm <= 255; }],
    "Offset8m8_AsmOperand">;

// Memory offset 0..510 for 16-bit memory accesses
def Offset8m16_AsmOperand : ImmAsmOperand<"Offset8m16">;
def offset8m16 : Immediate<i32,
    [{ return Imm >= 0 && Imm <= 510 && (Imm & 0x1 == 0); }],
    "Offset8m16_AsmOperand">;

```
- **EN**: Declares TableGen records such as `Offset8m8_AsmOperand`, `offset8m8`, `Offset8m16_AsmOperand`, `offset8m16`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `Offset8m8_AsmOperand`, `offset8m8`, `Offset8m16_AsmOperand`, `offset8m16` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 122-133
```tablegen
// Memory offset 0..1020 for 32-bit memory accesses
def Offset8m32_AsmOperand : ImmAsmOperand<"Offset8m32">;
def offset8m32 : Immediate<i32,
    [{ return Imm >= 0 && Imm <= 1020 && (Imm & 0x3 == 0); }],
    "Offset8m32_AsmOperand">;

// Memory offset 0..60 for 32-bit memory accesses
def Offset4m32_AsmOperand : ImmAsmOperand<"Offset4m32">;
def offset4m32 : Immediate<i32,
    [{ return Imm >= 0 && Imm <= 60 && (Imm & 0x3 == 0); }],
    "Offset4m32_AsmOperand">;

```
- **EN**: Declares TableGen records such as `Offset8m32_AsmOperand`, `offset8m32`, `Offset4m32_AsmOperand`, `offset4m32`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `Offset8m32_AsmOperand`, `offset8m32`, `Offset4m32_AsmOperand`, `offset4m32` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 134-140
```tablegen
// entry_imm12 predicate - Immediate in the range [0,32760], ENTRY parameter
def Entry_Imm12_AsmOperand: ImmAsmOperand<"entry_imm12">;
def entry_imm12: Immediate<i32, [{ return Imm >= 0 && Imm <= 32760 && (Imm % 8 == 0); }], "Entry_Imm12_AsmOperand"> {
  let EncoderMethod = "getEntry_Imm12OpValue";
  let DecoderMethod = "decodeEntry_Imm12OpValue";
}

```
- **EN**: Declares TableGen records such as `Entry_Imm12_AsmOperand`, `entry_imm12`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `Entry_Imm12_AsmOperand`, `entry_imm12` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 141-154
```tablegen
// b4const predicate - Branch Immediate 4-bit signed operand
def B4const_AsmOperand: ImmAsmOperand<"B4const">;
def b4const: Immediate<i32,
  [{ switch (Imm) {
        case -1: case 1: case 2: case 3:  case 4:
        case 5:  case 6: case 7: case 8: case 10: case 12:
        case 16: case 32: case 64: case 128: case 256: return 1;
        default: return 0;
     }
  }],
  "B4const_AsmOperand"> {
  let EncoderMethod = "getB4constOpValue";
  let DecoderMethod = "decodeB4constOperand";
}
```
- **EN**: Declares TableGen records such as `B4const_AsmOperand`, `b4const`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `B4const_AsmOperand`, `b4const` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 155-168
```tablegen

// b4constu predicate - Branch Immediate 4-bit unsigned operand
def B4constu_AsmOperand: ImmAsmOperand<"B4constu">;
def b4constu: Immediate<i32,
  [{ switch (Imm) {
        case 32768: case 65536: case 2: case 3:  case 4:
        case 5:  case 6: case 7: case 8: case 10: case 12:
        case 16: case 32: case 64: case 128: case 256: return 1;
        default: return 0;
     }
  }],
  "B4constu_AsmOperand"> {
  let EncoderMethod = "getB4constuOpValue";
  let DecoderMethod = "decodeB4constuOperand";
```
- **EN**: Declares TableGen records such as `B4constu_AsmOperand`, `b4constu`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `B4constu_AsmOperand`, `b4constu` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 169-177
```tablegen
}

// imm7_22 predicate - Immediate in the range [7,22] for sign extend and clamps
def Imm7_22_AsmOperand: ImmAsmOperand<"imm7_22">;
def imm7_22: Immediate<i32, [{ return Imm >= 7 && Imm <= 22; }], "Imm7_22_AsmOperand"> {
  let EncoderMethod = "getImm7_22OpValue";
  let DecoderMethod = "decodeImm7_22Operand";
}

```
- **EN**: Declares TableGen records such as `Imm7_22_AsmOperand`, `imm7_22`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `Imm7_22_AsmOperand`, `imm7_22` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 178-188
```tablegen
//===----------------------------------------------------------------------===//
// Memory address operands
//===----------------------------------------------------------------------===//

class mem<Operand offset> : Operand<i32> {
  let MIOperandInfo = (ops AR, offset);
  let EncoderMethod = "getMemRegEncoding";
  let OperandType = "OPERAND_MEMORY";
  let PrintMethod = "printMemOperand";
}

```
- **EN**: Declares TableGen records such as `mem`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `mem` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 189-196
```tablegen
def mem8   : mem<offset8m8> {
  let DecoderMethod = "decodeMem8Operand";
}

def mem16  : mem<offset8m16> {
  let DecoderMethod = "decodeMem16Operand";
}

```
- **EN**: Declares TableGen records such as `mem8`, `mem16`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `mem8`, `mem16` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 197-204
```tablegen
def mem32  : mem<offset8m32> {
  let DecoderMethod = "decodeMem32Operand";
}

def mem32n : mem<offset4m32> {
  let DecoderMethod = "decodeMem32nOperand";
}

```
- **EN**: Declares TableGen records such as `mem32`, `mem32n`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `mem32`, `mem32n` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 205-212
```tablegen
//Add patterns for future use in stack addressing mode
def addr_ish1 : ComplexPattern<iPTR, 2, "selectMemRegAddrISH1", [frameindex]>;
def addr_ish2 : ComplexPattern<iPTR, 2, "selectMemRegAddrISH2", [frameindex]>;
def addr_ish4 : ComplexPattern<iPTR, 2, "selectMemRegAddrISH4", [frameindex]>;

//===----------------------------------------------------------------------===//
// Symbolic address operands
//===----------------------------------------------------------------------===//
```
- **EN**: Declares TableGen records such as `addr_ish1`, `addr_ish2`, `addr_ish4`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `addr_ish1`, `addr_ish2`, `addr_ish4` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 213-219
```tablegen
def XtensaPCRelTargetAsmOperand : AsmOperandClass {
  let Name = "PCRelTarget";
  let ParserMethod = "parsePCRelTarget";
  let PredicateMethod = "isImm";
  let RenderMethod = "addImmOperands";
}

```
- **EN**: Declares TableGen records such as `XtensaPCRelTargetAsmOperand`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `XtensaPCRelTargetAsmOperand` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 220-227
```tablegen
def  pcrel32call : Operand<iPTR> {
  let PrintMethod = "printCallOperand";
  let EncoderMethod = "getCallEncoding";
  let DecoderMethod = "decodeCallOperand";
  let ParserMatchClass = XtensaPCRelTargetAsmOperand;
  let OperandType = "OPERAND_PCREL";
}

```
- **EN**: Declares TableGen records such as `pcrel32call`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `pcrel32call` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 228-235
```tablegen
def brtarget : Operand<OtherVT> {
  let PrintMethod = "printBranchTarget";
  let EncoderMethod = "getBranchTargetEncoding";
  let DecoderMethod = "decodeBranchOperand";
  let ParserMatchClass = XtensaPCRelTargetAsmOperand;
  let OperandType = "OPERAND_PCREL";
}

```
- **EN**: Declares TableGen records such as `brtarget`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `brtarget` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 236-243
```tablegen
def jumptarget : Operand<OtherVT> {
  let PrintMethod = "printJumpTarget";
  let EncoderMethod = "getJumpTargetEncoding";
  let DecoderMethod = "decodeJumpOperand";
  let ParserMatchClass = XtensaPCRelTargetAsmOperand;
  let OperandType = "OPERAND_PCREL";
}

```
- **EN**: Declares TableGen records such as `jumptarget`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `jumptarget` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 244-251
```tablegen
def ltarget : Operand<OtherVT> {
  let PrintMethod = "printLoopTarget";
  let EncoderMethod = "getLoopTargetEncoding";
  let DecoderMethod = "decodeLoopOperand";
  let ParserMatchClass = XtensaPCRelTargetAsmOperand;
  let OperandType = "OPERAND_PCREL";
}

```
- **EN**: Declares TableGen records such as `ltarget`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `ltarget` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 252-258
```tablegen
def L32Rtarget : Operand<i32> {
  let PrintMethod = "printL32RTarget";
  let EncoderMethod = "getL32RTargetEncoding";
  let DecoderMethod = "decodeL32ROperand";
  let ParserMatchClass = XtensaPCRelTargetAsmOperand;
  let OperandType = "OPERAND_PCREL";
}
```
- **EN**: Declares TableGen records such as `L32Rtarget`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `L32Rtarget` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

## Key Concepts / 关键概念

- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **LLVM subsystems / LLVM 子系统**: TableGen generators
