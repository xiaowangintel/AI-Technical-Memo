# MipsMTInstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsMTInstrFormats.td`
- Repository: `llvm-project`
- Purpose (EN): Describe the MIPS MT instructions format opcode - operation code.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `MipsMTInstrFormats`，涵盖指令编码格式与位域布局。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- MipsMTInstrFormats.td - Mips Instruction Formats ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 9-15
```tablegen
//===----------------------------------------------------------------------===//
//  Describe the MIPS MT instructions format
//
//  opcode - operation code.
//  rt     - destination register
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 17-20
```tablegen
class MipsMTInst : MipsInst<(outs), (ins), "", [], FrmOther> {
  let DecoderNamespace = "Mips";
  let EncodingPredicates = [HasStdEnc];
}
```
- EN: Declares reusable TableGen class `MipsMTInst` for `MipsMTInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrFormats` 声明可复用的 TableGen 类 `MipsMTInst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 22-24
```tablegen
class OPCODE1<bits<1> Val> {
  bits<1> Value = Val;
}
```
- EN: Declares reusable TableGen class `OPCODE1` for `MipsMTInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrFormats` 声明可复用的 TableGen 类 `OPCODE1`，通常用于抽象共享字段、谓词或编码结构。

### Lines 26-27
```tablegen
def OPCODE_SC_D : OPCODE1<0b0>;
def OPCODE_SC_E : OPCODE1<0b1>;
```
- EN: Defines TableGen record `OPCODE_SC_D` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `OPCODE_SC_D`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 29-31
```tablegen
class FIELD5<bits<5> Val> {
  bits<5> Value = Val;
}
```
- EN: Declares reusable TableGen class `FIELD5` for `MipsMTInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrFormats` 声明可复用的 TableGen 类 `FIELD5`，通常用于抽象共享字段、谓词或编码结构。

### Lines 33-37
```tablegen
def FIELD5_1_DMT_EMT  : FIELD5<0b00001>;
def FIELD5_2_DMT_EMT  : FIELD5<0b01111>;
def FIELD5_1_2_DVPE_EVPE : FIELD5<0b00000>;
def FIELD5_MFTR : FIELD5<0b01000>;
def FIELD5_MTTR : FIELD5<0b01100>;
```
- EN: Defines TableGen record `FIELD5_1_DMT_EMT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FIELD5_1_DMT_EMT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 39-40
```tablegen
class COP0_MFMC0_MT<FIELD5 Op1, FIELD5 Op2, OPCODE1 sc> : MipsMTInst {
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `COP0_MFMC0_MT` for `MipsMTInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrFormats` 声明可复用的 TableGen 类 `COP0_MFMC0_MT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 42-51
```tablegen
  bits<5> rt;
  let Inst{31-26} = 0b010000; // COP0
  let Inst{25-21} = 0b01011;  // MFMC0
  let Inst{20-16} = rt;
  let Inst{15-11} = Op1.Value;
  let Inst{10-6}  = Op2.Value;
  let Inst{5}     = sc.Value;
  let Inst{4-3}   = 0b00;
  let Inst{2-0}   = 0b001;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 53-54
```tablegen
class COP0_MFTTR_MT<FIELD5 Op> : MipsMTInst {
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `COP0_MFTTR_MT` for `MipsMTInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrFormats` 声明可复用的 TableGen 类 `COP0_MFTTR_MT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 56-70
```tablegen
  bits<5> rt;
  bits<5> rd;
  bits<1> u;
  bits<1> h;
  bits<3> sel;
  let Inst{31-26} = 0b010000; // COP0
  let Inst{25-21} = Op.Value; // MFMC0
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-6}  = 0b00000;  // rx - currently unsupported.
  let Inst{5}     = u;
  let Inst{4}     = h;
  let Inst{3}     = 0b0;
  let Inst{2-0}   = sel;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 72-73
```tablegen
class SPECIAL3_MT_FORK : MipsMTInst {
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `SPECIAL3_MT_FORK` for `MipsMTInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrFormats` 声明可复用的 TableGen 类 `SPECIAL3_MT_FORK`，通常用于抽象共享字段、谓词或编码结构。

### Lines 75-84
```tablegen
  bits<5> rs;
  bits<5> rt;
  bits<5> rd;
  let Inst{31-26} = 0b011111; // SPECIAL3
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-6}  = 0b00000;
  let Inst{5-0}   = 0b001000; // FORK
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 86-87
```tablegen
class SPECIAL3_MT_YIELD : MipsMTInst {
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `SPECIAL3_MT_YIELD` for `MipsMTInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrFormats` 声明可复用的 TableGen 类 `SPECIAL3_MT_YIELD`，通常用于抽象共享字段、谓词或编码结构。

### Lines 89-97
```tablegen
  bits<5> rs;
  bits<5> rd;
  let Inst{31-26} = 0b011111; // SPECIAL3
  let Inst{25-21} = rs;
  let Inst{20-16} = 0b00000;
  let Inst{15-11} = rd;
  let Inst{10-6}  = 0b00000;
  let Inst{5-0}   = 0b001001; // FORK
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

## Key Concepts / 关键概念

- EN: Primary role: instruction encoding formats and bitfield layout.
  - CN: 核心职责：指令编码格式与位域布局。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: Bitfield assignments describe exact instruction encodings and per-instruction flags.
  - CN: 位字段赋值用于描述精确的指令编码以及每条指令的标志位。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
