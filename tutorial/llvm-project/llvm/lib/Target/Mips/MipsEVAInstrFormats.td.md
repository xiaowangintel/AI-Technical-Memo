# MipsEVAInstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsEVAInstrFormats.td`
- Repository: `llvm-project`
- Purpose (EN): This file describes Mips32r6 instruction formats.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `MipsEVAInstrFormats`，涵盖指令编码格式与位域布局。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===- MipsEVAInstrFormats.td - Mips Instruction Formats ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes Mips32r6 instruction formats.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-17
```tablegen
class MipsEVAInst : MipsInst<(outs), (ins), "", [], FrmOther>,
                    StdArch {
  let DecoderNamespace = "Mips";
  let EncodingPredicates = [HasStdEnc];
}
```
- EN: Declares reusable TableGen class `MipsEVAInst` for `MipsEVAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrFormats` 声明可复用的 TableGen 类 `MipsEVAInst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 19-23
```tablegen
//===----------------------------------------------------------------------===//
//
// Field Values
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 25-30
```tablegen
// Memory Load/Store EVA
def OPCODE6_LBE        : OPCODE6<0b101100>;
def OPCODE6_LBuE       : OPCODE6<0b101000>;
def OPCODE6_LHE        : OPCODE6<0b101101>;
def OPCODE6_LHuE       : OPCODE6<0b101001>;
def OPCODE6_LWE        : OPCODE6<0b101111>;
```
- EN: Defines TableGen record `OPCODE6_LBE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `OPCODE6_LBE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 32-34
```tablegen
def OPCODE6_SBE        : OPCODE6<0b011100>;
def OPCODE6_SHE        : OPCODE6<0b011101>;
def OPCODE6_SWE        : OPCODE6<0b011111>;
```
- EN: Defines TableGen record `OPCODE6_SBE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `OPCODE6_SBE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 36-40
```tablegen
// load/store left/right EVA
def OPCODE6_LWLE       : OPCODE6<0b011001>;
def OPCODE6_LWRE       : OPCODE6<0b011010>;
def OPCODE6_SWLE       : OPCODE6<0b100001>;
def OPCODE6_SWRE       : OPCODE6<0b100010>;
```
- EN: Defines TableGen record `OPCODE6_LWLE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `OPCODE6_LWLE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 42-44
```tablegen
// Load-linked EVA, Store-conditional EVA
def OPCODE6_LLE        : OPCODE6<0b101110>;
def OPCODE6_SCE        : OPCODE6<0b011110>;
```
- EN: Defines TableGen record `OPCODE6_LLE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `OPCODE6_LLE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 46-47
```tablegen
def OPCODE6_TLBINV     : OPCODE6<0b000011>;
def OPCODE6_TLBINVF    : OPCODE6<0b000100>;
```
- EN: Defines TableGen record `OPCODE6_TLBINV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `OPCODE6_TLBINV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 49-50
```tablegen
def OPCODE6_CACHEE     : OPCODE6<0b011011>;
def OPCODE6_PREFE      : OPCODE6<0b100011>;
```
- EN: Defines TableGen record `OPCODE6_CACHEE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `OPCODE6_CACHEE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 52-52
```tablegen
def OPGROUP_COP0_TLB   : OPGROUP<0b010000>;
```
- EN: Defines TableGen record `OPGROUP_COP0_TLB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `OPGROUP_COP0_TLB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 54-58
```tablegen
//===----------------------------------------------------------------------===//
//
// Encoding Formats
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 60-64
```tablegen
class SPECIAL3_EVA_LOAD_STORE_FM<OPCODE6 Operation> : MipsEVAInst {
  bits<21> addr;
  bits<5> rt;
  bits<5> base = addr{20-16};
  bits<9> offset = addr{8-0};
```
- EN: Declares reusable TableGen class `SPECIAL3_EVA_LOAD_STORE_FM` for `MipsEVAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrFormats` 声明可复用的 TableGen 类 `SPECIAL3_EVA_LOAD_STORE_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 66-66
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 68-74
```tablegen
  let Inst{31-26} = OPGROUP_SPECIAL3.Value;
  let Inst{25-21} = base;
  let Inst{20-16} = rt;
  let Inst{15-7}  = offset;
  let Inst{6}     = 0;
  let Inst{5-0}   = Operation.Value;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 76-77
```tablegen
class TLB_FM<OPCODE6 Operation> : MipsEVAInst {
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `TLB_FM` for `MipsEVAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsEVAInstrFormats` 声明可复用的 TableGen 类 `TLB_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 79-83
```tablegen
  let Inst{31-26} = OPGROUP_COP0_TLB.Value;
  let Inst{25} = 1;       // CO
  let Inst{24-6} = 0;
  let Inst{5-0} = Operation.Value;
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
