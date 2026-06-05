# MicroMips32r6InstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MicroMips32r6InstrFormats.td`
- Repository: `llvm-project`
- Purpose (EN): This file describes microMIPS32r6 instruction formats.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `MicroMips32r6InstrFormats`，涵盖指令编码格式与位域布局。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//=- MicroMips32r6InstrFormats.td - Mips32r6 Instruction Formats -*- tablegen -*-==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes microMIPS32r6 instruction formats.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-17
```tablegen
class MMR6Arch<string opstr> {
  string Arch = "micromipsr6";
  string BaseOpcode = opstr;
  string DecoderNamespace = "MicroMipsR6";
}
```
- EN: Declares reusable TableGen class `MMR6Arch` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `MMR6Arch`，通常用于抽象共享字段、谓词或编码结构。

### Lines 19-25
```tablegen
//===----------------------------------------------------------------------===//
//
// Disambiguators
//
//===----------------------------------------------------------------------===//
//
// Some encodings are ambiguous except by comparing field values.
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 27-29
```tablegen
class MMDecodeDisambiguatedBy<string Name> : DecodeDisambiguates<Name> {
  string DecoderNamespace = "MicroMipsR6_Ambiguous";
}
```
- EN: Declares reusable TableGen class `MMDecodeDisambiguatedBy` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `MMDecodeDisambiguatedBy`，通常用于抽象共享字段、谓词或编码结构。

### Lines 31-35
```tablegen
//===----------------------------------------------------------------------===//
//
// Encoding Formats
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 37-38
```tablegen
class BC16_FM_MM16R6 {
  bits<10> offset;
```
- EN: Declares reusable TableGen class `BC16_FM_MM16R6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `BC16_FM_MM16R6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 40-40
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 42-44
```tablegen
  let Inst{15-10} = 0x33;
  let Inst{9-0}   = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 46-48
```tablegen
class BEQZC_BNEZC_FM_MM16R6<bits<6> op> {
  bits<3> rs;
  bits<7> offset;
```
- EN: Declares reusable TableGen class `BEQZC_BNEZC_FM_MM16R6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `BEQZC_BNEZC_FM_MM16R6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 50-50
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 52-55
```tablegen
  let Inst{15-10} = op;
  let Inst{9-7}   = rs;
  let Inst{6-0}   = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 57-58
```tablegen
class POOL16C_JALRC_FM_MM16R6<bits<5> op> {
  bits<5> rs;
```
- EN: Declares reusable TableGen class `POOL16C_JALRC_FM_MM16R6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL16C_JALRC_FM_MM16R6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 60-60
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 62-65
```tablegen
  let Inst{15-10} = 0x11;
  let Inst{9-5}   = rs;
  let Inst{4-0}   = op;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 67-70
```tablegen
class POP35_BOVC_FM_MMR6<string instr_asm> : MipsR6Inst, MMR6Arch<instr_asm> {
  bits<5> rt;
  bits<5> rs;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `POP35_BOVC_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POP35_BOVC_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 72-72
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 74-78
```tablegen
  let Inst{31-26} = 0b011101;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-0} = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 80-83
```tablegen
class POP37_BNVC_FM_MMR6<string instr_asm> : MipsR6Inst, MMR6Arch<instr_asm> {
  bits<5> rt;
  bits<5> rs;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `POP37_BNVC_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POP37_BNVC_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 85-85
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 87-91
```tablegen
  let Inst{31-26} = 0b011111;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-0} = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 93-94
```tablegen
class POOL16C_JRCADDIUSP_FM_MM16R6<bits<5> op> {
  bits<5> imm;
```
- EN: Declares reusable TableGen class `POOL16C_JRCADDIUSP_FM_MM16R6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL16C_JRCADDIUSP_FM_MM16R6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 96-96
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 98-101
```tablegen
  let Inst{15-10} = 0x11;
  let Inst{9-5}   = imm;
  let Inst{4-0}   = op;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 103-105
```tablegen
class POOL16C_LWM_SWM_FM_MM16R6<bits<4> funct> {
  bits<2> rt;
  bits<4> addr;
```
- EN: Declares reusable TableGen class `POOL16C_LWM_SWM_FM_MM16R6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL16C_LWM_SWM_FM_MM16R6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 107-107
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 109-113
```tablegen
  let Inst{15-10} = 0x11;
  let Inst{9-8}   = rt;
  let Inst{7-4}   = addr;
  let Inst{3-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 115-117
```tablegen
class POOL32A_BITSWAP_FM_MMR6<bits<6> funct> : MipsR6Inst {
  bits<5> rd;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `POOL32A_BITSWAP_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32A_BITSWAP_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 119-119
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 121-127
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = rt;
  let Inst{20-16} = rd;
  let Inst{15-12} = 0b0000;
  let Inst{11-6} = funct;
  let Inst{5-0} = 0b111100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 129-131
```tablegen
class CACHE_PREF_FM_MMR6<bits<6> opgroup, bits<4> funct> : MipsR6Inst {
  bits<21> addr;
  bits<5> hint;
```
- EN: Declares reusable TableGen class `CACHE_PREF_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `CACHE_PREF_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 133-133
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 135-140
```tablegen
  let Inst{31-26} = opgroup;
  let Inst{25-21} = hint;
  let Inst{20-16} = addr{20-16};
  let Inst{15-12} = funct;
  let Inst{11-0}  = addr{11-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 142-145
```tablegen
class ARITH_FM_MMR6<string instr_asm, bits<10> funct> : MMR6Arch<instr_asm> {
  bits<5> rd;
  bits<5> rt;
  bits<5> rs;
```
- EN: Declares reusable TableGen class `ARITH_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `ARITH_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 147-147
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 149-155
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-11} = rd;
  let Inst{10}    = 0;
  let Inst{9-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 157-160
```tablegen
class ADDI_FM_MMR6<string instr_asm, bits<6> op> : MMR6Arch<instr_asm> {
  bits<5>  rt;
  bits<5>  rs;
  bits<16> imm16;
```
- EN: Declares reusable TableGen class `ADDI_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `ADDI_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 162-162
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 164-168
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-0}  = imm16;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 170-174
```tablegen
class LB32_FM_MMR6 : MipsR6Inst {
  bits<21> addr;
  bits<5> rt;
  bits<5> base = addr{20-16};
  bits<16> offset = addr{15-0};
```
- EN: Declares reusable TableGen class `LB32_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `LB32_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 176-176
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 178-182
```tablegen
  let Inst{31-26} = 0b000111;
  let Inst{25-21} = rt;
  let Inst{20-16} = base;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 184-188
```tablegen
class LBU32_FM_MMR6 : MipsR6Inst {
  bits<21> addr;
  bits<5> rt;
  bits<5> base = addr{20-16};
  bits<16> offset = addr{15-0};
```
- EN: Declares reusable TableGen class `LBU32_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `LBU32_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 190-190
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 192-196
```tablegen
  let Inst{31-26} = 0b000101;
  let Inst{25-21} = rt;
  let Inst{20-16} = base;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 198-200
```tablegen
class PCREL19_FM_MMR6<bits<2> funct> : MipsR6Inst {
  bits<5> rt;
  bits<19> imm;
```
- EN: Declares reusable TableGen class `PCREL19_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `PCREL19_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 202-202
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 204-208
```tablegen
  let Inst{31-26} = 0b011110;
  let Inst{25-21} = rt;
  let Inst{20-19} = funct;
  let Inst{18-0}  = imm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 210-212
```tablegen
class PCREL16_FM_MMR6<bits<5> funct> : MipsR6Inst {
  bits<5> rt;
  bits<16> imm;
```
- EN: Declares reusable TableGen class `PCREL16_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `PCREL16_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 214-214
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 216-220
```tablegen
  let Inst{31-26} = 0b011110;
  let Inst{25-21} = rt;
  let Inst{20-16} = funct;
  let Inst{15-0}  = imm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 222-225
```tablegen
class POOL32A_FM_MMR6<bits<10> funct> : MipsR6Inst {
  bits<5> rd;
  bits<5> rs;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `POOL32A_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32A_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 227-227
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 229-235
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-11} = rd;
  let Inst{10}    = 0;
  let Inst{9-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 237-239
```tablegen
class POOL32A_PAUSE_FM_MMR6<string instr_asm, bits<5> op>
    : MMR6Arch<instr_asm> {
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `POOL32A_PAUSE_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32A_PAUSE_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 241-247
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-21} = 0;
  let Inst{20-16} = 0;
  let Inst{15-11} = op;
  let Inst{10-6} = 0;
  let Inst{5-0} = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 249-252
```tablegen
class POOL32A_RDPGPR_FM_MMR6<bits<10> funct> {
  bits<5> rt;
  bits<5> rd;
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `POOL32A_RDPGPR_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32A_RDPGPR_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 254-259
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-21} = rt;
  let Inst{20-16} = rd;
  let Inst{15-6} = funct;
  let Inst{5-0} = 0b111100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 261-265
```tablegen
class POOL32A_RDHWR_FM_MMR6 {
  bits<5> rt;
  bits<5> rs;
  bits<3> sel;
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `POOL32A_RDHWR_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32A_RDHWR_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 267-274
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-14} = 0;
  let Inst{13-11} = sel;
  let Inst{10} = 0;
  let Inst{9-0} = 0b0111000000;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 276-277
```tablegen
class POOL32A_SYNC_FM_MMR6 {
  bits<5> stype;
```
- EN: Declares reusable TableGen class `POOL32A_SYNC_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32A_SYNC_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 279-279
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 281-286
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-21} = 0;
  let Inst{20-16} = stype;
  let Inst{15-6}  = 0b0110101101;
  let Inst{5-0}   = 0b111100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 288-291
```tablegen
class POOL32I_SYNCI_FM_MMR6 {
  bits<21> addr;
  bits<5> base = addr{20-16};
  bits<16> immediate = addr{15-0};
```
- EN: Declares reusable TableGen class `POOL32I_SYNCI_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32I_SYNCI_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 293-293
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 295-299
```tablegen
  let Inst{31-26} = 0b010000;
  let Inst{25-21} = 0b01100;
  let Inst{20-16} = base;
  let Inst{15-0}  = immediate;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 301-303
```tablegen
class POOL32A_2R_FM_MMR6<bits<10> funct> : MipsR6Inst {
  bits<5> rs;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `POOL32A_2R_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32A_2R_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 305-305
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 307-312
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-6}  = funct;
  let Inst{5-0}   = 0b111100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 314-316
```tablegen
class SPECIAL_2R_FM_MMR6<bits<6> funct> : MipsR6Inst {
  bits<5> rs;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `SPECIAL_2R_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `SPECIAL_2R_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 318-318
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 320-326
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = rs;
  let Inst{20-16} = 0b00000;
  let Inst{15-11} = rt;
  let Inst{10-6}  = 0b00001;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 328-332
```tablegen
class POOL32A_ALIGN_FM_MMR6<bits<6> funct> : MipsR6Inst {
  bits<5> rd;
  bits<5> rs;
  bits<5> rt;
  bits<2> bp;
```
- EN: Declares reusable TableGen class `POOL32A_ALIGN_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32A_ALIGN_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 334-334
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 336-343
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-9}  = bp;
  let Inst{8-6}   = 0b000;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 345-348
```tablegen
class AUI_FM_MMR6 : MipsR6Inst {
  bits<5> rs;
  bits<5> rt;
  bits<16> imm;
```
- EN: Declares reusable TableGen class `AUI_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `AUI_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 350-350
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 352-356
```tablegen
  let Inst{31-26} = 0b000100;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-0} = imm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 358-362
```tablegen
class POOL32A_LSA_FM<bits<6> funct> : MipsR6Inst {
  bits<5> rd;
  bits<5> rs;
  bits<5> rt;
  bits<2> imm2;
```
- EN: Declares reusable TableGen class `POOL32A_LSA_FM` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32A_LSA_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 364-364
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 366-373
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-11} = rd;
  let Inst{10-9}  = imm2;
  let Inst{8-6}   = 0b000;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 375-379
```tablegen
class SB32_SH32_STORE_FM_MMR6<bits<6> op> {
  bits<5> rt;
  bits<21> addr;
  bits<5> base = addr{20-16};
  bits<16> offset = addr{15-0};
```
- EN: Declares reusable TableGen class `SB32_SH32_STORE_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `SB32_SH32_STORE_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 381-381
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 383-387
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = rt;
  let Inst{20-16} = base;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 389-393
```tablegen
class LOAD_WORD_FM_MMR6 {
  bits<5> rt;
  bits<21> addr;
  bits<5> base = addr{20-16};
  bits<16> offset = addr{15-0};
```
- EN: Declares reusable TableGen class `LOAD_WORD_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `LOAD_WORD_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 395-395
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 397-401
```tablegen
  let Inst{31-26} = 0b111111;
  let Inst{25-21} = rt;
  let Inst{20-16} = base;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 403-405
```tablegen
class LOAD_UPPER_IMM_FM_MMR6 {
  bits<5> rt;
  bits<16> imm16;
```
- EN: Declares reusable TableGen class `LOAD_UPPER_IMM_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `LOAD_UPPER_IMM_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 407-407
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 409-413
```tablegen
  let Inst{31-26} = 0b000100;
  let Inst{25-21} = rt;
  let Inst{20-16} = 0;
  let Inst{15-0}  = imm16;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 415-418
```tablegen
class CMP_BRANCH_1R_RT_OFF16_FM_MMR6<string instr_asm, bits<6> funct>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<5> rt;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `CMP_BRANCH_1R_RT_OFF16_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `CMP_BRANCH_1R_RT_OFF16_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 420-420
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 422-426
```tablegen
  let Inst{31-26} = funct;
  let Inst{25-21} = rt;
  let Inst{20-16} = 0b00000;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 428-431
```tablegen
class CMP_BRANCH_1R_BOTH_OFF16_FM_MMR6<string instr_asm, bits<6> funct>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<5> rt;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `CMP_BRANCH_1R_BOTH_OFF16_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `CMP_BRANCH_1R_BOTH_OFF16_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 433-433
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 435-439
```tablegen
  let Inst{31-26} = funct;
  let Inst{25-21} = rt;
  let Inst{20-16} = rt;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 441-444
```tablegen
class POOL32A_JALRC_FM_MMR6<string instr_asm, bits<10> funct>
    : MipsR6Inst, MMR6Arch<instr_asm> {
  bits<5> rt;
  bits<5> rs;
```
- EN: Declares reusable TableGen class `POOL32A_JALRC_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32A_JALRC_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 446-446
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 448-453
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-6} = funct;
  let Inst{5-0} = 0b111100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 455-460
```tablegen
class POOL32A_EXT_INS_FM_MMR6<string instr_asm, bits<6> funct>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<5> rt;
  bits<5> rs;
  bits<5> size;
  bits<5> pos;
```
- EN: Declares reusable TableGen class `POOL32A_EXT_INS_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32A_EXT_INS_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 462-462
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 464-470
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-11} = size;
  let Inst{10-6}  = pos;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 472-474
```tablegen
class POOL32A_ERET_FM_MMR6<string instr_asm, bits<10> funct>
    : MMR6Arch<instr_asm> {
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `POOL32A_ERET_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32A_ERET_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 476-480
```tablegen
  let Inst{31-26} = 0x00;
  let Inst{25-16} = 0x00;
  let Inst{15-6}  = funct;
  let Inst{5-0}   = 0x3c;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 482-483
```tablegen
class ERETNC_FM_MMR6<string instr_asm> : MMR6Arch<instr_asm> {
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `ERETNC_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `ERETNC_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 485-490
```tablegen
  let Inst{31-26} = 0x00;
  let Inst{25-17} = 0x00;
  let Inst{16-16} = 0x01;
  let Inst{15-6}  = 0x3cd;
  let Inst{5-0}   = 0x3c;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 492-500
```tablegen
class BREAK_MMR6_ENC<string instr_asm> : MMR6Arch<instr_asm> {
  bits<10> code_1;
  bits<10> code_2;
  bits<32> Inst;
  let Inst{31-26} = 0x0;
  let Inst{25-16} = code_1;
  let Inst{15-6}  = code_2;
  let Inst{5-0}   = 0x07;
}
```
- EN: Declares reusable TableGen class `BREAK_MMR6_ENC` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `BREAK_MMR6_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 502-503
```tablegen
class BARRIER_MMR6_ENC<string instr_asm, bits<5> op> : MMR6Arch<instr_asm> {
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `BARRIER_MMR6_ENC` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `BARRIER_MMR6_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 505-511
```tablegen
  let Inst{31-26} = 0x0;
  let Inst{25-21} = 0x0;
  let Inst{20-16} = 0x0;
  let Inst{15-11} = op;
  let Inst{10-6}  = 0x0;
  let Inst{5-0}   = 0x0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 513-517
```tablegen
class POOL32A_EIDI_MMR6_ENC<string instr_asm, bits<10> funct>
    : MMR6Arch<instr_asm> {
  bits<32> Inst;
  bits<5> rt; // Actually rs but we're sharing code with the standard encodings
              // which call it rt
```
- EN: Declares reusable TableGen class `POOL32A_EIDI_MMR6_ENC` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32A_EIDI_MMR6_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 519-524
```tablegen
  let Inst{31-26} = 0x00;
  let Inst{25-21} = 0x00;
  let Inst{20-16} = rt;
  let Inst{15-6}  = funct;
  let Inst{5-0}   = 0x3c;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 526-530
```tablegen
class SHIFT_MMR6_ENC<string instr_asm, bits<10> funct, bit rotate>
    : MMR6Arch<instr_asm> {
  bits<5> rd;
  bits<5> rt;
  bits<5> shamt;
```
- EN: Declares reusable TableGen class `SHIFT_MMR6_ENC` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `SHIFT_MMR6_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 532-532
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 534-540
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-21} = rd;
  let Inst{20-16} = rt;
  let Inst{15-11} = shamt;
  let Inst{10}    = rotate;
  let Inst{9-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 542-544
```tablegen
class SW32_FM_MMR6<string instr_asm, bits<6> op> : MMR6Arch<instr_asm> {
  bits<5> rt;
  bits<21> addr;
```
- EN: Declares reusable TableGen class `SW32_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `SW32_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 546-546
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 548-552
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = rt;
  let Inst{20-16} = addr{20-16};
  let Inst{15-0}  = addr{15-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 554-558
```tablegen
class POOL32F_ARITH_FM_MMR6<string instr_asm, bits<2> fmt, bits<8> funct>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<5> ft;
  bits<5> fs;
  bits<5> fd;
```
- EN: Declares reusable TableGen class `POOL32F_ARITH_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32F_ARITH_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 560-560
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 562-569
```tablegen
  let Inst{31-26} = 0b010101;
  let Inst{25-21} = ft;
  let Inst{20-16} = fs;
  let Inst{15-11} = fd;
  let Inst{10}    = 0;
  let Inst{9-8}   = fmt;
  let Inst{7-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 571-575
```tablegen
class POOL32F_ARITHF_FM_MMR6<string instr_asm, bits<2> fmt, bits<9> funct>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<5> ft;
  bits<5> fs;
  bits<5> fd;
```
- EN: Declares reusable TableGen class `POOL32F_ARITHF_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32F_ARITHF_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 577-577
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 579-585
```tablegen
  let Inst{31-26} = 0b010101;
  let Inst{25-21} = ft;
  let Inst{20-16} = fs;
  let Inst{15-11} = fd;
  let Inst{10-9}  = fmt;
  let Inst{8-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 587-590
```tablegen
class POOL32F_MOV_NEG_FM_MMR6<string instr_asm, bits<2> fmt, bits<7> funct>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<5> ft;
  bits<5> fs;
```
- EN: Declares reusable TableGen class `POOL32F_MOV_NEG_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32F_MOV_NEG_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 592-592
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 594-601
```tablegen
  let Inst{31-26} = 0b010101;
  let Inst{25-21} = ft;
  let Inst{20-16} = fs;
  let Inst{15}    = 0;
  let Inst{14-13} = fmt;
  let Inst{12-6}  = funct;
  let Inst{5-0}   = 0b111011;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 603-607
```tablegen
class POOL32F_MINMAX_FM<string instr_asm, bits<2> fmt, bits<9> funct>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<5> ft;
  bits<5> fs;
  bits<5> fd;
```
- EN: Declares reusable TableGen class `POOL32F_MINMAX_FM` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32F_MINMAX_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 609-609
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 611-617
```tablegen
  let Inst{31-26} = 0b010101;
  let Inst{25-21} = ft;
  let Inst{20-16} = fs;
  let Inst{15-11} = fd;
  let Inst{10-9} = fmt;
  let Inst{8-0} = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 619-623
```tablegen
class POOL32F_CMP_FM<string instr_asm, bits<6> format, FIELD_CMP_COND Cond>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<5> ft;
  bits<5> fs;
  bits<5> fd;
```
- EN: Declares reusable TableGen class `POOL32F_CMP_FM` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32F_CMP_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 625-625
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 627-633
```tablegen
  let Inst{31-26} = 0b010101;
  let Inst{25-21} = ft;
  let Inst{20-16} = fs;
  let Inst{15-11} = fd;
  let Inst{10-6} = Cond.Value;
  let Inst{5-0} = format;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 635-638
```tablegen
class POOL32F_CVT_LW_FM<string instr_asm, bit fmt, bits<8> funct>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<5> ft;
  bits<5> fs;
```
- EN: Declares reusable TableGen class `POOL32F_CVT_LW_FM` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32F_CVT_LW_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 640-648
```tablegen
  bits<32> Inst;
  let Inst{31-26} = 0b010101;
  let Inst{25-21} = ft;
  let Inst{20-16} = fs;
  let Inst{15} = 0;
  let Inst{14} = fmt;
  let Inst{13-6} = funct;
  let Inst{5-0} = 0b111011;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 650-653
```tablegen
class POOL32F_CVT_DS_FM<string instr_asm, bits<2> fmt, bits<7> funct>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<5> ft;
  bits<5> fs;
```
- EN: Declares reusable TableGen class `POOL32F_CVT_DS_FM` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32F_CVT_DS_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 655-663
```tablegen
  bits<32> Inst;
  let Inst{31-26} = 0b010101;
  let Inst{25-21} = ft;
  let Inst{20-16} = fs;
  let Inst{15} = 0;
  let Inst{14-13} = fmt;
  let Inst{12-6} = funct;
  let Inst{5-0} = 0b111011;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 665-668
```tablegen
class POOL32F_ABS_FM_MMR6<string instr_asm, bits<2> fmt, bits<7> funct>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<5> ft;
  bits<5> fs;
```
- EN: Declares reusable TableGen class `POOL32F_ABS_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32F_ABS_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 670-670
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 672-679
```tablegen
  let Inst{31-26} = 0b010101;
  let Inst{25-21} = ft;
  let Inst{20-16} = fs;
  let Inst{15}    = 0;
  let Inst{14-13} = fmt;
  let Inst{12-6}  = funct;
  let Inst{5-0}   = 0b111011;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 681-684
```tablegen
class POOL32F_MATH_FM_MMR6<string instr_asm, bits<1> fmt, bits<8> funct>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<5> ft;
  bits<5> fs;
```
- EN: Declares reusable TableGen class `POOL32F_MATH_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32F_MATH_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 686-686
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 688-695
```tablegen
  let Inst{31-26} = 0b010101;
  let Inst{25-21} = ft;
  let Inst{20-16} = fs;
  let Inst{15}    = 0;
  let Inst{14}    = fmt;
  let Inst{13-6}  = funct;
  let Inst{5-0}   = 0b111011;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 697-700
```tablegen
class POOL16A_ADDU16_FM_MMR6 {
  bits<3> rs;
  bits<3> rt;
  bits<3> rd;
```
- EN: Declares reusable TableGen class `POOL16A_ADDU16_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL16A_ADDU16_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 702-702
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 704-709
```tablegen
  let Inst{15-10} = 0b000001;
  let Inst{9-7}   = rs;
  let Inst{6-4}   = rt;
  let Inst{3-1}   = rd;
  let Inst{0}     = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 711-713
```tablegen
class POOL16C_AND16_FM_MMR6 {
  bits<3> rt;
  bits<3> rs;
```
- EN: Declares reusable TableGen class `POOL16C_AND16_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL16C_AND16_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 715-715
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 717-721
```tablegen
  let Inst{15-10} = 0b010001;
  let Inst{9-7}   = rt;
  let Inst{6-4}   = rs;
  let Inst{3-0}   = 0b0001;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 723-725
```tablegen
class POOL16C_NOT16_FM_MMR6 {
  bits<3> rt;
  bits<3> rs;
```
- EN: Declares reusable TableGen class `POOL16C_NOT16_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL16C_NOT16_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 727-727
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 729-733
```tablegen
  let Inst{15-10} = 0x11;
  let Inst{9-7}   = rt;
  let Inst{6-4}   = rs;
  let Inst{3-0}   = 0b0000;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 735-737
```tablegen
class POOL16C_MOVEP16_FM_MMR6 {
  bits<3> rt;
  bits<3> rs;
```
- EN: Declares reusable TableGen class `POOL16C_MOVEP16_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL16C_MOVEP16_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 739-739
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 741-749
```tablegen
  let Inst{15-10} = 0b010001;
  // bits 7-9 are populated by MipsMCCodeEmitter::encodeInstruction, with a
  // special encoding of both rd1 and rd2.
  let Inst{9-7}   = ?;
  let Inst{6-4}   = rt;
  let Inst{3}     = rs{2};
  let Inst{2}     = 0b1;
  let Inst{1-0}   = rs{1-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 751-753
```tablegen
class POOL16C_OR16_XOR16_FM_MMR6<bits<4> op> {
  bits<3> rt;
  bits<3> rs;
```
- EN: Declares reusable TableGen class `POOL16C_OR16_XOR16_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL16C_OR16_XOR16_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 755-755
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 757-761
```tablegen
  let Inst{15-10} = 0b010001;
  let Inst{9-7}   = rt;
  let Inst{6-4}   = rs;
  let Inst{3-0}   = op;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 763-765
```tablegen
class POOL16C_BREAKPOINT_FM_MMR6<bits<6> op> {
  bits<4> code_;
  bits<16> Inst;
```
- EN: Declares reusable TableGen class `POOL16C_BREAKPOINT_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL16C_BREAKPOINT_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 767-770
```tablegen
  let Inst{15-10} = 0b010001;
  let Inst{9-6}   = code_;
  let Inst{5-0}   = op;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 772-775
```tablegen
class POOL16A_SUBU16_FM_MMR6 {
  bits<3> rs;
  bits<3> rt;
  bits<3> rd;
```
- EN: Declares reusable TableGen class `POOL16A_SUBU16_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL16A_SUBU16_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 777-777
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 779-784
```tablegen
  let Inst{15-10} = 0b000001;
  let Inst{9-7}   = rs;
  let Inst{6-4}   = rt;
  let Inst{3-1}   = rd;
  let Inst{0}     = 0b1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 786-789
```tablegen
class POOL32A_WRPGPR_WSBH_FM_MMR6<string instr_asm, bits<10> funct>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<5> rt;
  bits<5> rs;
```
- EN: Declares reusable TableGen class `POOL32A_WRPGPR_WSBH_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32A_WRPGPR_WSBH_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 791-791
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 793-798
```tablegen
  let Inst{31-26} = 0x00;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-6}  = funct;
  let Inst{5-0}   = 0x3c;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 800-803
```tablegen
class POOL32F_RECIP_ROUND_FM_MMR6<string instr_asm, bits<1> fmt, bits<8> funct>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<5> ft;
  bits<5> fs;
```
- EN: Declares reusable TableGen class `POOL32F_RECIP_ROUND_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32F_RECIP_ROUND_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 805-805
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 807-814
```tablegen
  let Inst{31-26} = 0b010101;
  let Inst{25-21} = ft;
  let Inst{20-16} = fs;
  let Inst{15}    = 0;
  let Inst{14}    = fmt;
  let Inst{13-6}  = funct;
  let Inst{5-0}   = 0b111011;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 816-819
```tablegen
class POOL32F_RINT_FM_MMR6<string instr_asm, bits<2> fmt> : MMR6Arch<instr_asm>,
                                                            MipsR6Inst {
  bits<5> fs;
  bits<5> fd;
```
- EN: Declares reusable TableGen class `POOL32F_RINT_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32F_RINT_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 821-821
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 823-829
```tablegen
  let Inst{31-26} = 0b010101;
  let Inst{25-21} = fs;
  let Inst{20-16} = fd;
  let Inst{15-11} = 0;
  let Inst{10-9}  = fmt;
  let Inst{8-0}   = 0b000100000;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 831-835
```tablegen
class POOL32F_SEL_FM_MMR6<string instr_asm, bits<2> fmt, bits<9> funct>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<5> ft;
  bits<5> fs;
  bits<5> fd;
```
- EN: Declares reusable TableGen class `POOL32F_SEL_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32F_SEL_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 837-837
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 839-845
```tablegen
  let Inst{31-26} = 0b010101;
  let Inst{25-21} = ft;
  let Inst{20-16} = fs;
  let Inst{15-11} = fd;
  let Inst{10-9}  = fmt;
  let Inst{8-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 847-850
```tablegen
class POOL32F_CLASS_FM_MMR6<string instr_asm, bits<2> fmt, bits<9> funct>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<5> fs;
  bits<5> fd;
```
- EN: Declares reusable TableGen class `POOL32F_CLASS_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32F_CLASS_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 852-852
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 854-860
```tablegen
  let Inst{31-26} = 0b010101;
  let Inst{25-21} = fs;
  let Inst{20-16} = fd;
  let Inst{15-11} = 0b00000;
  let Inst{10-9}  = fmt;
  let Inst{8-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 862-864
```tablegen
class POOL32A_TLBINV_FM_MMR6<string instr_asm, bits<10> funct>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `POOL32A_TLBINV_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32A_TLBINV_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 866-870
```tablegen
  let Inst{31-26} = 0x0;
  let Inst{25-16} = 0x0;
  let Inst{15-6}  = funct;
  let Inst{5-0}   = 0b111100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 872-876
```tablegen
class POOL32A_MFTC0_FM_MMR6<string instr_asm, bits<5> funct, bits<6> opcode>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<5> rt;
  bits<5> rs;
  bits<3> sel;
```
- EN: Declares reusable TableGen class `POOL32A_MFTC0_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32A_MFTC0_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 878-878
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 880-887
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-14} = 0;
  let Inst{13-11} = sel;
  let Inst{10-6}  = funct;
  let Inst{5-0}   = opcode;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 889-892
```tablegen
class POOL32A_GINV_FM_MMR6<string instr_asm, bits<2> ginv>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<5> rs;
  bits<2> type;
```
- EN: Declares reusable TableGen class `POOL32A_GINV_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32A_GINV_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 894-894
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 896-904
```tablegen
  let Inst{31-26} = 0x0;
  let Inst{25-21} = 0x0;
  let Inst{20-16} = rs;
  let Inst{15-13} = 0b011;
  let Inst{12-11} = ginv;
  let Inst{10-9}  = type;
  let Inst{8-6}   = 0b101;
  let Inst{5-0}   = 0b111100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 906-909
```tablegen
class POOL32F_MFTC1_FM_MMR6<string instr_asm, bits<8> funct>
    : MMR6Arch<instr_asm> {
  bits<5> rt;
  bits<5> fs;
```
- EN: Declares reusable TableGen class `POOL32F_MFTC1_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32F_MFTC1_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 911-911
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 913-919
```tablegen
  let Inst{31-26} = 0b010101;
  let Inst{25-21} = rt;
  let Inst{20-16} = fs;
  let Inst{15-14} = 0;
  let Inst{13-6}  = funct;
  let Inst{5-0}   = 0b111011;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 921-924
```tablegen
class POOL32A_MFTC2_FM_MMR6<string instr_asm, bits<10> funct>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<5> rt;
  bits<5> impl;
```
- EN: Declares reusable TableGen class `POOL32A_MFTC2_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32A_MFTC2_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 926-926
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 928-933
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = rt;
  let Inst{20-16} = impl;
  let Inst{15-6}  = funct;
  let Inst{5-0}   = 0b111100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 935-939
```tablegen
class CMP_BRANCH_2R_OFF16_FM_MMR6<string opstr, bits<6> funct>
    : MipsR6Inst, MMR6Arch<opstr> {
  bits<5> rt;
  bits<5> rs;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `CMP_BRANCH_2R_OFF16_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `CMP_BRANCH_2R_OFF16_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 941-941
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 943-947
```tablegen
  let Inst{31-26} = funct;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 949-951
```tablegen
class POOL32A_DVPEVP_FM_MMR6<string instr_asm, bits<10> funct>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<5> rs;
```
- EN: Declares reusable TableGen class `POOL32A_DVPEVP_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32A_DVPEVP_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 953-953
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 955-960
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = 0b00000;
  let Inst{20-16} = rs;
  let Inst{15-6}  = funct;
  let Inst{5-0}   = 0b111100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 962-964
```tablegen
class CMP_BRANCH_OFF21_FM_MMR6<bits<6> funct> : MipsR6Inst {
  bits<5> rs;
  bits<21> offset;
```
- EN: Declares reusable TableGen class `CMP_BRANCH_OFF21_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `CMP_BRANCH_OFF21_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 966-966
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 968-971
```tablegen
  let Inst{31-26} = funct;
  let Inst{25-21} = rs;
  let Inst{20-0} = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 973-976
```tablegen
class POOL32I_BRANCH_COP_1_2_FM_MMR6<string instr_asm, bits<5> funct>
    : MMR6Arch<instr_asm> {
  bits<5> rt;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `POOL32I_BRANCH_COP_1_2_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32I_BRANCH_COP_1_2_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 978-978
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 980-984
```tablegen
  let Inst{31-26} = 0b010000;
  let Inst{25-21} = funct;
  let Inst{20-16} = rt;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 986-991
```tablegen
class LDWC1_SDWC1_FM_MMR6<string instr_asm, bits<6> funct>
    : MMR6Arch<instr_asm> {
  bits<5> ft;
  bits<21> addr;
  bits<5> base = addr{20-16};
  bits<16> offset = addr{15-0};
```
- EN: Declares reusable TableGen class `LDWC1_SDWC1_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `LDWC1_SDWC1_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 993-993
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 995-999
```tablegen
  let Inst{31-26} = funct;
  let Inst{25-21} = ft;
  let Inst{20-16} = base;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1001-1006
```tablegen
class POOL32B_LDWC2_SDWC2_FM_MMR6<string instr_asm, bits<4> funct>
    : MMR6Arch<instr_asm>, MipsR6Inst {
  bits<5> rt;
  bits<21> addr;
  bits<5> base = addr{20-16};
  bits<11> offset = addr{10-0};
```
- EN: Declares reusable TableGen class `POOL32B_LDWC2_SDWC2_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32B_LDWC2_SDWC2_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1008-1008
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1010-1016
```tablegen
  let Inst{31-26} = 0b001000;
  let Inst{25-21} = rt;
  let Inst{20-16} = base;
  let Inst{15-12} = funct;
  let Inst{11}    = 0;
  let Inst{10-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1018-1024
```tablegen
class POOL32C_LL_E_SC_E_FM_MMR6<string instr_asm, bits<4> majorFunc,
                                bits<3> minorFunc> : MMR6Arch<instr_asm>,
                                                     MipsR6Inst {
  bits<5> rt;
  bits<21> addr;
  bits<5> base = addr{20-16};
  bits<9> offset = addr{8-0};
```
- EN: Declares reusable TableGen class `POOL32C_LL_E_SC_E_FM_MMR6` for `MicroMips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMips32r6InstrFormats` 声明可复用的 TableGen 类 `POOL32C_LL_E_SC_E_FM_MMR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1026-1026
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1028-1034
```tablegen
  let Inst{31-26} = 0b011000;
  let Inst{25-21} = rt;
  let Inst{20-16} = base;
  let Inst{15-12} = majorFunc;
  let Inst{11-9}  = minorFunc;
  let Inst{8-0}   = offset;
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
