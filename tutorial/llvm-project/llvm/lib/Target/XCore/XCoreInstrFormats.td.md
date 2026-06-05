# XCoreInstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/XCore/XCoreInstrFormats.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines reusable instruction encoding formats in TableGen DSL.
  - **CN**: 使用 TableGen DSL 定义可复用的指令编码格式。
- **Note / 说明**:
  - **EN**: This file uses LLVM TableGen DSL, so many definitions are declarative records rather than executable C++ statements.
  - **CN**: 该文件使用 LLVM TableGen DSL，因此许多内容是声明式记录，而不是可执行的 C++ 语句。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- XCoreInstrFormats.td - XCore Instruction Formats ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-15
```tablegen

//===----------------------------------------------------------------------===//
// Instruction format superclass
//===----------------------------------------------------------------------===//
class InstXCore<int sz, dag outs, dag ins, string asmstr, list<dag> pattern>
    : Instruction {
  field bits<32> Inst;

```
- **EN**: Declares TableGen records such as `InstXCore`; this block attaches SelectionDAG patterns; describes instruction encoding bits.
- **CN**: 声明了 `InstXCore` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、描述指令编码位。

### Lines 16-23
```tablegen
  let Namespace = "XCore";
  dag OutOperandList = outs;
  dag InOperandList = ins;
  let AsmString   = asmstr;
  let Pattern = pattern;
  let Size = sz;
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 24-30
```tablegen
// XCore pseudo instructions format
class PseudoInstXCore<dag outs, dag ins, string asmstr, list<dag> pattern>
   : InstXCore<0, outs, ins, asmstr, pattern> {
  let isPseudo = 1;
}

//===----------------------------------------------------------------------===//
```
- **EN**: Declares TableGen records such as `PseudoInstXCore`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `PseudoInstXCore` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 31-39
```tablegen
// Instruction formats
//===----------------------------------------------------------------------===//

class _F3R<bits<5> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : InstXCore<2, outs, ins, asmstr, pattern> {
  let Inst{15-11} = opc;
  let DecoderMethod = "Decode3RInstruction";
}

```
- **EN**: Declares TableGen records such as `_F3R`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_F3R` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 40-47
```tablegen
// 3R with first operand as an immediate. Used for TSETR where the first
// operand is treated as an immediate since it refers to a register number in
// another thread.
class _F3RImm<bits<5> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : _F3R<opc, outs, ins, asmstr, pattern> {
  let DecoderMethod = "Decode3RImmInstruction";
}

```
- **EN**: Declares TableGen records such as `_F3RImm`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_F3RImm` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 48-57
```tablegen
class _FL3R<bits<9> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : InstXCore<4, outs, ins, asmstr, pattern> {
  let Inst{31-27} = opc{8-4};
  let Inst{26-20} = 0b1111110;
  let Inst{19-16} = opc{3-0};

  let Inst{15-11} = 0b11111;
  let DecoderMethod = "DecodeL3RInstruction";
}

```
- **EN**: Declares TableGen records such as `_FL3R`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_FL3R` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 58-69
```tablegen
// L3R with first operand as both a source and a destination.
class _FL3RSrcDst<bits<9> opc, dag outs, dag ins, string asmstr,
                  list<dag> pattern> : _FL3R<opc, outs, ins, asmstr, pattern> {
  let DecoderMethod = "DecodeL3RSrcDstInstruction";
}

class _F2RUS<bits<5> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : InstXCore<2, outs, ins, asmstr, pattern> {
  let Inst{15-11} = opc;
  let DecoderMethod = "Decode2RUSInstruction";
}

```
- **EN**: Declares TableGen records such as `_FL3RSrcDst`, `_F2RUS`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_FL3RSrcDst`, `_F2RUS` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 70-76
```tablegen
// 2RUS with bitp operand
class _F2RUSBitp<bits<5> opc, dag outs, dag ins, string asmstr,
                 list<dag> pattern>
    : _F2RUS<opc, outs, ins, asmstr, pattern> {
  let DecoderMethod = "Decode2RUSBitpInstruction";
}

```
- **EN**: Declares TableGen records such as `_F2RUSBitp`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_F2RUSBitp` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 77-86
```tablegen
class _FL2RUS<bits<9> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : InstXCore<4, outs, ins, asmstr, pattern> {
  let Inst{31-27} = opc{8-4};
  let Inst{26-20} = 0b1111110;
  let Inst{19-16} = opc{3-0};

  let Inst{15-11} = 0b11111;
  let DecoderMethod = "DecodeL2RUSInstruction";
}

```
- **EN**: Declares TableGen records such as `_FL2RUS`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_FL2RUS` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 87-93
```tablegen
// L2RUS with bitp operand
class _FL2RUSBitp<bits<9> opc, dag outs, dag ins, string asmstr,
                  list<dag> pattern>
    : _FL2RUS<opc, outs, ins, asmstr, pattern> {
  let DecoderMethod = "DecodeL2RUSBitpInstruction";
}

```
- **EN**: Declares TableGen records such as `_FL2RUSBitp`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_FL2RUSBitp` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 94-103
```tablegen
class _FRU6<bits<6> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : InstXCore<2, outs, ins, asmstr, pattern> {
  bits<4> a;
  bits<6> b;

  let Inst{15-10} = opc;
  let Inst{9-6} = a;
  let Inst{5-0} = b;
}

```
- **EN**: Declares TableGen records such as `_FRU6`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_FRU6` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 104-115
```tablegen
class _FLRU6<bits<6> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : InstXCore<4, outs, ins, asmstr, pattern> {
  bits<4> a;
  bits<16> b;

  let Inst{31-26} = opc;
  let Inst{25-22} = a;
  let Inst{21-16} = b{5-0};
  let Inst{15-10} = 0b111100;
  let Inst{9-0} = b{15-6};
}

```
- **EN**: Declares TableGen records such as `_FLRU6`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_FLRU6` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 116-123
```tablegen
class _FU6<bits<10> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : InstXCore<2, outs, ins, asmstr, pattern> {
  bits<6> a;

  let Inst{15-6} = opc;
  let Inst{5-0} = a;
}

```
- **EN**: Declares TableGen records such as `_FU6`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_FU6` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 124-133
```tablegen
class _FLU6<bits<10> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : InstXCore<4, outs, ins, asmstr, pattern> {
  bits<16> a;

  let Inst{31-22} = opc;
  let Inst{21-16} = a{5-0};
  let Inst{15-10} = 0b111100;
  let Inst{9-0} = a{15-6};
}

```
- **EN**: Declares TableGen records such as `_FLU6`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_FLU6` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 134-141
```tablegen
class _FU10<bits<6> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : InstXCore<2, outs, ins, asmstr, pattern> {
  bits<10> a;

  let Inst{15-10} = opc;
  let Inst{9-0} = a;
}

```
- **EN**: Declares TableGen records such as `_FU10`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_FU10` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 142-151
```tablegen
class _FLU10<bits<6> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : InstXCore<4, outs, ins, asmstr, pattern> {
  bits<20> a;

  let Inst{31-26} = opc;
  let Inst{25-16} = a{9-0};
  let Inst{15-10} = 0b111100;
  let Inst{9-0} = a{19-10};
}

```
- **EN**: Declares TableGen records such as `_FLU10`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_FLU10` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 152-158
```tablegen
class _F2R<bits<6> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : InstXCore<2, outs, ins, asmstr, pattern> {
  let Inst{15-11} = opc{5-1};
  let Inst{4} = opc{0};
  let DecoderMethod = "Decode2RInstruction";
}

```
- **EN**: Declares TableGen records such as `_F2R`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_F2R` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 159-166
```tablegen
// 2R with first operand as an immediate. Used for TSETMR where the first
// operand is treated as an immediate since it refers to a register number in
// another thread.
class _F2RImm<bits<6> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : _F2R<opc, outs, ins, asmstr, pattern> {
  let DecoderMethod = "Decode2RImmInstruction";
}

```
- **EN**: Declares TableGen records such as `_F2RImm`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_F2RImm` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 167-178
```tablegen
// 2R with first operand as both a source and a destination.
class _F2RSrcDst<bits<6> opc, dag outs, dag ins, string asmstr,
                 list<dag> pattern> : _F2R<opc, outs, ins, asmstr, pattern> {
  let DecoderMethod = "Decode2RSrcDstInstruction";
}

// Same as 2R with last two operands swapped
class _FR2R<bits<6> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : _F2R<opc, outs, ins, asmstr, pattern> {
  let DecoderMethod = "DecodeR2RInstruction";
}

```
- **EN**: Declares TableGen records such as `_F2RSrcDst`, `_FR2R`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_F2RSrcDst`, `_FR2R` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 179-185
```tablegen
class _FRUS<bits<6> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : InstXCore<2, outs, ins, asmstr, pattern> {
  let Inst{15-11} = opc{5-1};
  let Inst{4} = opc{0};
  let DecoderMethod = "DecodeRUSInstruction";
}

```
- **EN**: Declares TableGen records such as `_FRUS`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_FRUS` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 186-192
```tablegen
// RUS with bitp operand
class _FRUSBitp<bits<6> opc, dag outs, dag ins, string asmstr,
                list<dag> pattern>
    : _FRUS<opc, outs, ins, asmstr, pattern> {
  let DecoderMethod = "DecodeRUSBitpInstruction";
}

```
- **EN**: Declares TableGen records such as `_FRUSBitp`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_FRUSBitp` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 193-200
```tablegen
// RUS with first operand as both a source and a destination and a bitp second
// operand
class _FRUSSrcDstBitp<bits<6> opc, dag outs, dag ins, string asmstr,
                      list<dag> pattern>
    : _FRUS<opc, outs, ins, asmstr, pattern> {
  let DecoderMethod = "DecodeRUSSrcDstBitpInstruction";
}

```
- **EN**: Declares TableGen records such as `_FRUSSrcDstBitp`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_FRUSSrcDstBitp` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 201-211
```tablegen
class _FL2R<bits<10> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : InstXCore<4, outs, ins, asmstr, pattern> {
  let Inst{31-27} = opc{9-5};
  let Inst{26-20} = 0b1111110;
  let Inst{19-16} = opc{4-1};

  let Inst{15-11} = 0b11111;
  let Inst{4} = opc{0};
  let DecoderMethod = "DecodeL2RInstruction";
}

```
- **EN**: Declares TableGen records such as `_FL2R`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_FL2R` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 212-221
```tablegen
// Same as L2R with last two operands swapped
class _FLR2R<bits<10> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : _FL2R<opc, outs, ins, asmstr, pattern> {
  let DecoderMethod = "DecodeLR2RInstruction";
}

class _F1R<bits<6> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : InstXCore<2, outs, ins, asmstr, pattern> {
  bits<4> a;

```
- **EN**: Declares TableGen records such as `_FLR2R`, `_F1R`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_FLR2R`, `_F1R` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 222-234
```tablegen
  let Inst{15-11} = opc{5-1};
  let Inst{10-5} = 0b111111;
  let Inst{4} = opc{0};
  let Inst{3-0} = a;
}

class _F0R<bits<10> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : InstXCore<2, outs, ins, asmstr, pattern> {
  let Inst{15-11} = opc{9-5};
  let Inst{10-5} = 0b111111;
  let Inst{4-0} = opc{4-0};
}

```
- **EN**: Declares TableGen records such as `_F0R`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_F0R` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 235-245
```tablegen
class _FL4R<bits<6> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : InstXCore<4, outs, ins, asmstr, pattern> {
  bits<4> d;

  let Inst{31-27} = opc{5-1};
  let Inst{26-21} = 0b111111;
  let Inst{20} = opc{0};
  let Inst{19-16} = d;
  let Inst{15-11} = 0b11111;
}

```
- **EN**: Declares TableGen records such as `_FL4R`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_FL4R` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 246-252
```tablegen
// L4R with 4th operand as both a source and a destination.
class _FL4RSrcDst<bits<6> opc, dag outs, dag ins, string asmstr,
                  list<dag> pattern>
    : _FL4R<opc, outs, ins, asmstr, pattern> {
  let DecoderMethod = "DecodeL4RSrcDstInstruction";
}

```
- **EN**: Declares TableGen records such as `_FL4RSrcDst`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_FL4RSrcDst` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 253-259
```tablegen
// L4R with 1st and 4th operand as both a source and a destination.
class _FL4RSrcDstSrcDst<bits<6> opc, dag outs, dag ins, string asmstr,
                        list<dag> pattern>
    : _FL4R<opc, outs, ins, asmstr, pattern> {
  let DecoderMethod = "DecodeL4RSrcDstSrcDstInstruction";
}

```
- **EN**: Declares TableGen records such as `_FL4RSrcDstSrcDst`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_FL4RSrcDstSrcDst` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 260-268
```tablegen
class _FL5R<bits<6> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : InstXCore<4, outs, ins, asmstr, pattern> {
  let Inst{31-27} = opc{5-1};
  let Inst{20} = opc{0};
  let Inst{15-11} = 0b11111;

  let DecoderMethod = "DecodeL5RInstruction";
}

```
- **EN**: Declares TableGen records such as `_FL5R`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_FL5R` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 269-275
```tablegen
class _FL6R<bits<5> opc, dag outs, dag ins, string asmstr, list<dag> pattern>
    : InstXCore<4, outs, ins, asmstr, pattern> {
  let Inst{31-27} = opc;
  let Inst{15-11} = 0b11111;

  let DecoderMethod = "DecodeL6RInstruction";
}
```
- **EN**: Declares TableGen records such as `_FL6R`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `_FL6R` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

## Key Concepts / 关键概念

- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **LLVM subsystems / LLVM 子系统**: TableGen generators
