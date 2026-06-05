# XtensaInstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/XtensaInstrFormats.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines reusable instruction encoding formats in TableGen DSL.
  - **CN**: 使用 TableGen DSL 定义可复用的指令编码格式。
- **Note / 说明**:
  - **EN**: This file uses LLVM TableGen DSL, so many definitions are declarative records rather than executable C++ statements.
  - **CN**: 该文件使用 LLVM TableGen DSL，因此许多内容是声明式记录，而不是可执行的 C++ 语句。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```tablegen
//===- XtensaInstrFormats.td - Xtensa Instruction Formats --*- tablegen -*-===//
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

### Lines 10-16
```tablegen

// Base class for Xtensa 16 & 24 bit Formats
class XtensaInst<int size, dag outs, dag ins, string asmstr, list<dag> pattern,
                 InstrItinClass itin = NoItinerary>
  : Instruction {
  let Namespace = "Xtensa";

```
- **EN**: Declares TableGen records such as `XtensaInst`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `XtensaInst` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 17-25
```tablegen
  let Size = size;

  let OutOperandList = outs;
  let InOperandList  = ins;

  let AsmString   = asmstr;
  let Pattern     = pattern;
  let Itinerary   = itin;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 26-34
```tablegen
}

// Base class for Xtensa 24 bit Format
class XtensaInst24<dag outs, dag ins, string asmstr, list<dag> pattern,
                   InstrItinClass itin = NoItinerary>
  : XtensaInst<3, outs, ins, asmstr, pattern, itin> {
  field bits<24> Inst;
}

```
- **EN**: Declares TableGen records such as `XtensaInst24`; this block attaches SelectionDAG patterns; describes instruction encoding bits.
- **CN**: 声明了 `XtensaInst24` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、描述指令编码位。

### Lines 35-42
```tablegen
// Base class for Xtensa 16 bit Format
class XtensaInst16<dag outs, dag ins, string asmstr, list<dag> pattern,
                   InstrItinClass itin = NoItinerary>
  : XtensaInst<2, outs, ins, asmstr, pattern, itin> {
  field bits<16> Inst;
  let Predicates = [HasDensity];
}

```
- **EN**: Declares TableGen records such as `XtensaInst16`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `XtensaInst16` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 43-49
```tablegen
class RRR_Inst<bits<4> op0, bits<4> op1, bits<4> op2, dag outs, dag ins,
               string asmstr, list<dag> pattern, InstrItinClass itin = NoItinerary>
  : XtensaInst24<outs, ins, asmstr, pattern, itin> {
  bits<4> r;
  bits<4> s;
  bits<4> t;

```
- **EN**: Declares TableGen records such as `RRR_Inst`; this block attaches SelectionDAG patterns; describes instruction encoding bits.
- **CN**: 声明了 `RRR_Inst` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、描述指令编码位。

### Lines 50-57
```tablegen
  let Inst{23-20} = op2;
  let Inst{19-16} = op1;
  let Inst{15-12} = r;
  let Inst{11-8} = s;
  let Inst{7-4} = t;
  let Inst{3-0} = op0;
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 58-65
```tablegen
class RRI4_Inst<bits<4> op0, bits<4> op1, dag outs, dag ins,
                string asmstr, list<dag> pattern, InstrItinClass itin = NoItinerary>
  : XtensaInst24<outs, ins, asmstr, pattern, itin> {
  bits<4> r;
  bits<4> s;
  bits<4> t;
  bits<4> imm4;

```
- **EN**: Declares TableGen records such as `RRI4_Inst`; this block attaches SelectionDAG patterns; describes instruction encoding bits.
- **CN**: 声明了 `RRI4_Inst` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、描述指令编码位。

### Lines 66-73
```tablegen
  let Inst{23-20} = imm4;
  let Inst{19-16} = op1;
  let Inst{15-12} = r;
  let Inst{11-8} = s;
  let Inst{7-4} = t;
  let Inst{3-0} = op0;
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 74-81
```tablegen
class RRI8_Inst<bits<4> op0, dag outs, dag ins,
                string asmstr, list<dag> pattern, InstrItinClass itin = NoItinerary>
  : XtensaInst24<outs, ins, asmstr, pattern, itin> {
  bits<4> r;
  bits<4> s;
  bits<4> t;
  bits<8> imm8;

```
- **EN**: Declares TableGen records such as `RRI8_Inst`; this block attaches SelectionDAG patterns; describes instruction encoding bits.
- **CN**: 声明了 `RRI8_Inst` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、描述指令编码位。

### Lines 82-88
```tablegen
  let Inst{23-16} = imm8;
  let Inst{15-12} = r;
  let Inst{11-8} = s;
  let Inst{7-4} = t;
  let Inst{3-0} = op0;
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 89-99
```tablegen
class RI16_Inst<bits<4> op0, dag outs, dag ins,
                string asmstr, list<dag> pattern, InstrItinClass itin = NoItinerary>
  : XtensaInst24<outs, ins, asmstr, pattern, itin> {
  bits<4> t;
  bits<16> imm16;

  let Inst{23-8} = imm16;
  let Inst{7-4} = t;
  let Inst{3-0} = op0;
}

```
- **EN**: Declares TableGen records such as `RI16_Inst`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `RI16_Inst` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 100-112
```tablegen
class RSR_Inst<bits<4> op0, bits<4> op1, bits<4> op2, dag outs, dag ins,
               string asmstr, list<dag> pattern, InstrItinClass itin = NoItinerary>
  : XtensaInst24<outs, ins, asmstr, pattern, itin> {
  bits<8> sr;
  bits<4> t;

  let Inst{23-20} = op2;
  let Inst{19-16} = op1;
  let Inst{15-8} = sr;
  let Inst{7-4} = t;
  let Inst{3-0} = op0;
}

```
- **EN**: Declares TableGen records such as `RSR_Inst`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `RSR_Inst` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 113-123
```tablegen
class CALL_Inst<bits<4> op0, dag outs, dag ins,
                string asmstr, list<dag> pattern, InstrItinClass itin = NoItinerary>
  : XtensaInst24<outs, ins, asmstr, pattern, itin> {
  bits<18> offset;
  bits<2> n;

  let Inst{23-6} = offset;
  let Inst{5-4} = n;
  let Inst{3-0} = op0;
}

```
- **EN**: Declares TableGen records such as `CALL_Inst`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `CALL_Inst` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 124-131
```tablegen
class CALLX_Inst<bits<4> op0, bits<4> op1, bits<4> op2, dag outs, dag ins,
                 string asmstr, list<dag> pattern, InstrItinClass itin = NoItinerary>
  : XtensaInst24<outs, ins, asmstr, pattern, itin> {
  bits<4> r;
  bits<4> s;
  bits<2> m;
  bits<2> n;

```
- **EN**: Declares TableGen records such as `CALLX_Inst`; this block attaches SelectionDAG patterns; describes instruction encoding bits.
- **CN**: 声明了 `CALLX_Inst` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、描述指令编码位。

### Lines 132-140
```tablegen
  let Inst{23-20} = op2;
  let Inst{19-16} = op1;
  let Inst{15-12} = r;
  let Inst{11-8} = s;
  let Inst{7-6} = m;
  let Inst{5-4} = n;
  let Inst{3-0} = op0;
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 141-149
```tablegen
class BRI8_Inst<bits<4> op0, dag outs, dag ins,
                string asmstr, list<dag> pattern, InstrItinClass itin = NoItinerary>
  : XtensaInst24<outs, ins, asmstr, pattern, itin> {
  bits<8> imm8;
  bits<4> r;
  bits<4> s;
  bits<2> m;
  bits<2> n;

```
- **EN**: Declares TableGen records such as `BRI8_Inst`; this block attaches SelectionDAG patterns; describes instruction encoding bits.
- **CN**: 声明了 `BRI8_Inst` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、描述指令编码位。

### Lines 150-157
```tablegen
  let Inst{23-16} = imm8;
  let Inst{15-12} = r;
  let Inst{11-8} = s;
  let Inst{7-6} = m;
  let Inst{5-4} = n;
  let Inst{3-0} = op0;
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 158-170
```tablegen
class BRI12_Inst<bits<4> op0, bits<2> n, bits<2> m, dag outs, dag ins,
                 string asmstr, list<dag> pattern, InstrItinClass itin = NoItinerary>
  : XtensaInst24<outs, ins, asmstr, pattern, itin> {
  bits<12> imm12;
  bits<4> s;

  let Inst{23-12} = imm12;
  let Inst{11-8} = s;
  let Inst{7-6} = m;
  let Inst{5-4} = n;
  let Inst{3-0} = op0;
}

```
- **EN**: Declares TableGen records such as `BRI12_Inst`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `BRI12_Inst` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 171-177
```tablegen
class RRRN_Inst<bits<4> op0, dag outs, dag ins,
                string asmstr, list<dag> pattern, InstrItinClass itin = NoItinerary>
  : XtensaInst16<outs, ins, asmstr, pattern, itin> {
  bits<4> r;
  bits<4> s;
  bits<4> t;

```
- **EN**: Declares TableGen records such as `RRRN_Inst`; this block attaches SelectionDAG patterns; describes instruction encoding bits.
- **CN**: 声明了 `RRRN_Inst` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、描述指令编码位。

### Lines 178-189
```tablegen
  let Inst{15-12} = r;
  let Inst{11-8} = s;
  let Inst{7-4} = t;
  let Inst{3-0} = op0;
}

class RI7_Inst<bits<4> op0, bits<1> i, dag outs, dag ins,
               string asmstr, list<dag> pattern, InstrItinClass itin = NoItinerary>
  : XtensaInst16<outs, ins, asmstr, pattern, itin> {
  bits<7> imm7;
  bits<4> s;

```
- **EN**: Declares TableGen records such as `RI7_Inst`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `RI7_Inst` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 190-196
```tablegen
  let Inst{15-12} = imm7{3-0};
  let Inst{11-8} = s;
  let Inst{7} = i;
  let Inst{6-4} = imm7{6-4};
  let Inst{3-0} = op0;
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 197-210
```tablegen
class RI6_Inst<bits<4> op0, bits<1> i,  bits<1> z, dag outs, dag ins,
               string asmstr, list<dag> pattern, InstrItinClass itin = NoItinerary>
  : XtensaInst16<outs, ins, asmstr, pattern, itin> {
  bits<6> imm6;
  bits<4> s;

  let Inst{15-12} = imm6{3-0};
  let Inst{11-8} = s;
  let Inst{7} = i;
  let Inst{6} = z;
  let Inst{5-4} = imm6{5-4};
  let Inst{3-0} = op0;
}

```
- **EN**: Declares TableGen records such as `RI6_Inst`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `RI6_Inst` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 211-216
```tablegen
// Pseudo instructions
class Pseudo<dag outs, dag ins, string asmstr, list<dag> pattern>
  : XtensaInst<2, outs, ins, asmstr, pattern> {
  let isPseudo = 1;
  let isCodeGenOnly = 1;
}
```
- **EN**: Declares TableGen records such as `Pseudo`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `Pseudo` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

## Key Concepts / 关键概念

- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **LLVM subsystems / LLVM 子系统**: TableGen generators
