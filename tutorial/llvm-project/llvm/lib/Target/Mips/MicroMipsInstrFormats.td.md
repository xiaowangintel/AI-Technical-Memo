# MicroMipsInstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MicroMipsInstrFormats.td`
- Repository: `llvm-project`
- Purpose (EN): This files describes the formats of the microMIPS instruction set.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `MicroMipsInstrFormats`，涵盖指令编码格式与位域布局。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===-- MicroMipsInstrFormats.td - microMIPS Inst Formats -*- tablegen -*--===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This files describes the formats of the microMIPS instruction set.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-15
```tablegen
//===----------------------------------------------------------------------===//
// MicroMIPS Base Classes
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 17-25
```tablegen
//
// Base class for MicroMips instructions.
// This class does not depend on the instruction size.
//
class MicroMipsInstBase<dag outs, dag ins, string asmstr, list<dag> pattern,
                        Format f> : Instruction,
                        PredicateControl {
  let Namespace = "Mips";
  let DecoderNamespace = "MicroMips";
```
- EN: Declares reusable TableGen class `for` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `for`，通常用于抽象共享字段、谓词或编码结构。

### Lines 27-28
```tablegen
  let OutOperandList = outs;
  let InOperandList  = ins;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 30-32
```tablegen
  let AsmString   = asmstr;
  let Pattern     = pattern;
  let EncodingPredicates = [InMicroMips];
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 34-35
```tablegen
  Format Form = f;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 37-47
```tablegen
//
// Base class for MicroMIPS 16-bit instructions.
//
class MicroMipsInst16<dag outs, dag ins, string asmstr, list<dag> pattern,
               Format f> :
  MicroMipsInstBase<outs, ins, asmstr, pattern, f>
{
  let Size = 2;
  field bits<16> Inst;
  bits<6> Opcode = 0x0;
}
```
- EN: Declares reusable TableGen class `for` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `for`，通常用于抽象共享字段、谓词或编码结构。

### Lines 49-51
```tablegen
//===----------------------------------------------------------------------===//
// MicroMIPS 16-bit Instruction Formats
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 53-56
```tablegen
class ARITH_FM_MM16<bit funct> {
  bits<3> rd;
  bits<3> rt;
  bits<3> rs;
```
- EN: Declares reusable TableGen class `ARITH_FM_MM16` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `ARITH_FM_MM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 58-58
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 60-65
```tablegen
  let Inst{15-10} = 0x01;
  let Inst{9-7}   = rd;
  let Inst{6-4}   = rt;
  let Inst{3-1}   = rs;
  let Inst{0}     = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 67-70
```tablegen
class ANDI_FM_MM16<bits<6> funct> {
  bits<3> rd;
  bits<3> rs;
  bits<4> imm;
```
- EN: Declares reusable TableGen class `ANDI_FM_MM16` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `ANDI_FM_MM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 72-72
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 74-78
```tablegen
  let Inst{15-10} = funct;
  let Inst{9-7}   = rd;
  let Inst{6-4}   = rs;
  let Inst{3-0}   = imm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 80-82
```tablegen
class LOGIC_FM_MM16<bits<4> funct> {
  bits<3> rt;
  bits<3> rs;
```
- EN: Declares reusable TableGen class `LOGIC_FM_MM16` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `LOGIC_FM_MM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 84-84
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 86-90
```tablegen
  let Inst{15-10} = 0x11;
  let Inst{9-6}   = funct;
  let Inst{5-3}   = rt;
  let Inst{2-0}   = rs;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 92-95
```tablegen
class SHIFT_FM_MM16<bits<1> funct> {
  bits<3> rd;
  bits<3> rt;
  bits<3> shamt;
```
- EN: Declares reusable TableGen class `SHIFT_FM_MM16` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `SHIFT_FM_MM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 97-97
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 99-104
```tablegen
  let Inst{15-10} = 0x09;
  let Inst{9-7}   = rd;
  let Inst{6-4}   = rt;
  let Inst{3-1}   = shamt;
  let Inst{0}     = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 106-109
```tablegen
class ADDIUR2_FM_MM16 {
  bits<3> rd;
  bits<3> rs;
  bits<3> imm;
```
- EN: Declares reusable TableGen class `ADDIUR2_FM_MM16` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `ADDIUR2_FM_MM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 111-111
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 113-118
```tablegen
  let Inst{15-10} = 0x1b;
  let Inst{9-7}   = rd;
  let Inst{6-4}   = rs;
  let Inst{3-1}   = imm;
  let Inst{0}     = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 120-122
```tablegen
class LOAD_STORE_FM_MM16<bits<6> op> {
  bits<3> rt;
  bits<7> addr;
```
- EN: Declares reusable TableGen class `LOAD_STORE_FM_MM16` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `LOAD_STORE_FM_MM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 124-124
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 126-130
```tablegen
  let Inst{15-10} = op;
  let Inst{9-7}   = rt;
  let Inst{6-4}   = addr{6-4};
  let Inst{3-0}   = addr{3-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 132-134
```tablegen
class LOAD_STORE_SP_FM_MM16<bits<6> op> {
  bits<5> rt;
  bits<5> offset;
```
- EN: Declares reusable TableGen class `LOAD_STORE_SP_FM_MM16` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `LOAD_STORE_SP_FM_MM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 136-136
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 138-141
```tablegen
  let Inst{15-10} = op;
  let Inst{9-5}   = rt;
  let Inst{4-0}   = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 143-145
```tablegen
class LOAD_GP_FM_MM16<bits<6> op> {
  bits<3> rt;
  bits<7> offset;
```
- EN: Declares reusable TableGen class `LOAD_GP_FM_MM16` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `LOAD_GP_FM_MM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 147-147
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 149-152
```tablegen
  let Inst{15-10} = op;
  let Inst{9-7} = rt;
  let Inst{6-0} = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 154-156
```tablegen
class ADDIUS5_FM_MM16 {
  bits<5> rd;
  bits<4> imm;
```
- EN: Declares reusable TableGen class `ADDIUS5_FM_MM16` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `ADDIUS5_FM_MM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 158-158
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 160-164
```tablegen
  let Inst{15-10} = 0x13;
  let Inst{9-5}   = rd;
  let Inst{4-1}   = imm;
  let Inst{0}     = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 166-167
```tablegen
class ADDIUSP_FM_MM16 {
  bits<9> imm;
```
- EN: Declares reusable TableGen class `ADDIUSP_FM_MM16` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `ADDIUSP_FM_MM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 169-169
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 171-174
```tablegen
  let Inst{15-10} = 0x13;
  let Inst{9-1}   = imm;
  let Inst{0}     = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 176-178
```tablegen
class MOVE_FM_MM16<bits<6> funct> {
  bits<5> rs;
  bits<5> rd;
```
- EN: Declares reusable TableGen class `MOVE_FM_MM16` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `MOVE_FM_MM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 180-180
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 182-185
```tablegen
  let Inst{15-10} = funct;
  let Inst{9-5}   = rd;
  let Inst{4-0}   = rs;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 187-189
```tablegen
class LI_FM_MM16 {
  bits<3> rd;
  bits<7> imm;
```
- EN: Declares reusable TableGen class `LI_FM_MM16` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `LI_FM_MM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 191-191
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 193-196
```tablegen
  let Inst{15-10} = 0x3b;
  let Inst{9-7}   = rd;
  let Inst{6-0}   = imm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 198-199
```tablegen
class JALR_FM_MM16<bits<5> op> {
  bits<5> rs;
```
- EN: Declares reusable TableGen class `JALR_FM_MM16` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `JALR_FM_MM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 201-201
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 203-206
```tablegen
  let Inst{15-10} = 0x11;
  let Inst{9-5}   = op;
  let Inst{4-0}   = rs;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 208-209
```tablegen
class MFHILO_FM_MM16<bits<5> funct> {
  bits<5> rd;
```
- EN: Declares reusable TableGen class `MFHILO_FM_MM16` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `MFHILO_FM_MM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 211-211
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 213-216
```tablegen
  let Inst{15-10} = 0x11;
  let Inst{9-5}   = funct;
  let Inst{4-0}   = rd;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 218-220
```tablegen
class JRADDIUSP_FM_MM16<bits<5> op> {
  bits<5> rs;
  bits<5> imm;
```
- EN: Declares reusable TableGen class `JRADDIUSP_FM_MM16` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `JRADDIUSP_FM_MM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 222-222
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 224-227
```tablegen
  let Inst{15-10} = 0x11;
  let Inst{9-5}   = op;
  let Inst{4-0}   = imm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 229-231
```tablegen
class ADDIUR1SP_FM_MM16 {
  bits<3> rd;
  bits<6> imm;
```
- EN: Declares reusable TableGen class `ADDIUR1SP_FM_MM16` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `ADDIUR1SP_FM_MM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 233-233
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 235-239
```tablegen
  let Inst{15-10} = 0x1b;
  let Inst{9-7}   = rd;
  let Inst{6-1}   = imm;
  let Inst{0}     = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 241-243
```tablegen
class BRKSDBBP16_FM_MM<bits<6> op> {
  bits<4> code_;
  bits<16> Inst;
```
- EN: Declares reusable TableGen class `BRKSDBBP16_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `BRKSDBBP16_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 245-248
```tablegen
  let Inst{15-10} = 0x11;
  let Inst{9-4}   = op;
  let Inst{3-0}   = code_;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 250-252
```tablegen
class BEQNEZ_FM_MM16<bits<6> op> {
  bits<3> rs;
  bits<7> offset;
```
- EN: Declares reusable TableGen class `BEQNEZ_FM_MM16` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `BEQNEZ_FM_MM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 254-254
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 256-259
```tablegen
  let Inst{15-10} = op;
  let Inst{9-7}   = rs;
  let Inst{6-0}   = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 261-262
```tablegen
class B16_FM {
  bits<10> offset;
```
- EN: Declares reusable TableGen class `B16_FM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `B16_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 264-264
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 266-268
```tablegen
  let Inst{15-10} = 0x33;
  let Inst{9-0}   = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 270-272
```tablegen
class MOVEP_FM_MM16 {
  bits<3> rt;
  bits<3> rs;
```
- EN: Declares reusable TableGen class `MOVEP_FM_MM16` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `MOVEP_FM_MM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 274-274
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 276-283
```tablegen
  let Inst{15-10} = 0x21;
  // bits 7-9 are populated by MipsMCCodeEmitter::encodeInstruction, with a
  // special encoding of both rd1 and rd2.
  let Inst{9-7}   = ?;
  let Inst{6-4}   = rt;
  let Inst{3-1}   = rs;
  let Inst{0}     = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 285-287
```tablegen
//===----------------------------------------------------------------------===//
// MicroMIPS 32-bit Instruction Formats
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 289-291
```tablegen
class MMArch {
  string Arch = "micromips";
}
```
- EN: Declares reusable TableGen class `MMArch` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `MMArch`，通常用于抽象共享字段、谓词或编码结构。

### Lines 293-296
```tablegen
class ADD_FM_MM<bits<6> op, bits<10> funct> : MMArch {
  bits<5> rt;
  bits<5> rs;
  bits<5> rd;
```
- EN: Declares reusable TableGen class `ADD_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `ADD_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 298-298
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 300-306
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-11} = rd;
  let Inst{10}    = 0;
  let Inst{9-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 308-311
```tablegen
class ADDI_FM_MM<bits<6> op> : MMArch {
  bits<5>  rs;
  bits<5>  rt;
  bits<16> imm16;
```
- EN: Declares reusable TableGen class `ADDI_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `ADDI_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 313-313
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 315-319
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-0}  = imm16;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 321-324
```tablegen
class SLTI_FM_MM<bits<6> op> : MMArch {
  bits<5> rt;
  bits<5> rs;
  bits<16> imm16;
```
- EN: Declares reusable TableGen class `SLTI_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `SLTI_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 326-326
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 328-332
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-0}  = imm16;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 334-336
```tablegen
class LUI_FM_MM : MMArch {
  bits<5> rt;
  bits<16> imm16;
```
- EN: Declares reusable TableGen class `LUI_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `LUI_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 338-338
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 340-344
```tablegen
  let Inst{31-26} = 0x10;
  let Inst{25-21} = 0xd;
  let Inst{20-16} = rt;
  let Inst{15-0}  = imm16;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 346-348
```tablegen
class MULT_FM_MM<bits<10> funct> : MMArch {
  bits<5>  rs;
  bits<5>  rt;
```
- EN: Declares reusable TableGen class `MULT_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `MULT_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 350-350
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 352-357
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

### Lines 359-362
```tablegen
class SRA_FM_MM<bits<10> funct, bit rotate> : MMArch {
  bits<5> rd;
  bits<5> rt;
  bits<5> shamt;
```
- EN: Declares reusable TableGen class `SRA_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `SRA_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 364-364
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 366-372
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

### Lines 374-377
```tablegen
class SRLV_FM_MM<bits<10> funct, bit rotate> : MMArch {
  bits<5> rd;
  bits<5> rt;
  bits<5> rs;
```
- EN: Declares reusable TableGen class `SRLV_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `SRLV_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 379-379
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 381-387
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-11} = rd;
  let Inst{10}    = rotate;
  let Inst{9-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 389-393
```tablegen
class LW_FM_MM<bits<6> op> : MMArch {
  bits<5> rt;
  bits<21> addr;
  bits<5> base = addr{20-16};
  bits<16> offset = addr{15-0};
```
- EN: Declares reusable TableGen class `LW_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `LW_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 395-395
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 397-401
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = rt;
  let Inst{20-16} = base;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 403-407
```tablegen
class POOL32C_LHUE_FM_MM<bits<6> op, bits<4> fmt, bits<3> funct> : MMArch {
  bits<5> rt;
  bits<21> addr;
  bits<5> base = addr{20-16};
  bits<9> offset = addr{8-0};
```
- EN: Declares reusable TableGen class `POOL32C_LHUE_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `POOL32C_LHUE_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 409-409
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 411-417
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = rt;
  let Inst{20-16} = base;
  let Inst{15-12} = fmt;
  let Inst{11-9} = funct;
  let Inst{8-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 419-421
```tablegen
class LWL_FM_MM<bits<4> funct> : MMArch {
  bits<5> rt;
  bits<21> addr;
```
- EN: Declares reusable TableGen class `LWL_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `LWL_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 423-423
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 425-430
```tablegen
  let Inst{31-26} = 0x18;
  let Inst{25-21} = rt;
  let Inst{20-16} = addr{20-16};
  let Inst{15-12} = funct;
  let Inst{11-0}  = addr{11-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 432-436
```tablegen
class POOL32C_STEVA_LDEVA_FM_MM<bits<4> type, bits<3> funct> : MMArch {
  bits<5> rt;
  bits<21> addr;
  bits<5> base = addr{20-16};
  bits<9> offset = addr{8-0};
```
- EN: Declares reusable TableGen class `POOL32C_STEVA_LDEVA_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `POOL32C_STEVA_LDEVA_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 438-438
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 440-446
```tablegen
  let Inst{31-26} = 0x18;
  let Inst{25-21} = rt;
  let Inst{20-16} = base;
  let Inst{15-12} = type;
  let Inst{11-9} = funct;
  let Inst{8-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 448-451
```tablegen
class CMov_F_I_FM_MM<bits<7> func> : MMArch {
  bits<5> rd;
  bits<5> rs;
  bits<3> fcc;
```
- EN: Declares reusable TableGen class `CMov_F_I_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `CMov_F_I_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 453-453
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 455-461
```tablegen
  let Inst{31-26} = 0x15;
  let Inst{25-21} = rd;
  let Inst{20-16} = rs;
  let Inst{15-13} = fcc;
  let Inst{12-6}  = func;
  let Inst{5-0}   = 0x3b;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 463-464
```tablegen
class MTLO_FM_MM<bits<10> funct> : MMArch {
  bits<5> rs;
```
- EN: Declares reusable TableGen class `MTLO_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `MTLO_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 466-466
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 468-473
```tablegen
  let Inst{31-26} = 0x00;
  let Inst{25-21} = 0x00;
  let Inst{20-16} = rs;
  let Inst{15-6}  = funct;
  let Inst{5-0}   = 0x3c;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 475-476
```tablegen
class MFLO_FM_MM<bits<10> funct> : MMArch {
  bits<5> rd;
```
- EN: Declares reusable TableGen class `MFLO_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `MFLO_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 478-478
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 480-485
```tablegen
  let Inst{31-26} = 0x00;
  let Inst{25-21} = 0x00;
  let Inst{20-16} = rd;
  let Inst{15-6}  = funct;
  let Inst{5-0}   = 0x3c;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 487-489
```tablegen
class CLO_FM_MM<bits<10> funct> : MMArch {
  bits<5> rd;
  bits<5> rs;
```
- EN: Declares reusable TableGen class `CLO_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `CLO_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 491-491
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 493-498
```tablegen
  let Inst{31-26} = 0x00;
  let Inst{25-21} = rd;
  let Inst{20-16} = rs;
  let Inst{15-6}  = funct;
  let Inst{5-0}   = 0x3c;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 500-502
```tablegen
class SEB_FM_MM<bits<10> funct> : MMArch {
  bits<5> rd;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `SEB_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `SEB_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 504-504
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 506-511
```tablegen
  let Inst{31-26} = 0x00;
  let Inst{25-21} = rd;
  let Inst{20-16} = rt;
  let Inst{15-6}  = funct;
  let Inst{5-0}   = 0x3c;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 513-517
```tablegen
class EXT_FM_MM<bits<6> funct> : MMArch {
  bits<5> rt;
  bits<5> rs;
  bits<5> pos;
  bits<5> size;
```
- EN: Declares reusable TableGen class `EXT_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `EXT_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 519-519
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 521-527
```tablegen
  let Inst{31-26} = 0x00;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-11} = size;
  let Inst{10-6}  = pos;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 529-530
```tablegen
class J_FM_MM<bits<6> op> : MMArch {
  bits<26> target;
```
- EN: Declares reusable TableGen class `J_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `J_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 532-532
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 534-536
```tablegen
  let Inst{31-26} = op;
  let Inst{25-0}  = target;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 538-539
```tablegen
class JR_FM_MM<bits<8> funct> : MMArch {
  bits<5> rs;
```
- EN: Declares reusable TableGen class `JR_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `JR_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 541-541
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 543-548
```tablegen
  let Inst{31-21} = 0x00;
  let Inst{20-16} = rs;
  let Inst{15-14} = 0x0;
  let Inst{13-6}  = funct;
  let Inst{5-0}   = 0x3c;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 550-552
```tablegen
class JALR_FM_MM<bits<10> funct> {
  bits<5> rs;
  bits<5> rd;
```
- EN: Declares reusable TableGen class `JALR_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `JALR_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 554-554
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 556-561
```tablegen
  let Inst{31-26} = 0x00;
  let Inst{25-21} = rd;
  let Inst{20-16} = rs;
  let Inst{15-6}  = funct;
  let Inst{5-0}   = 0x3c;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 563-566
```tablegen
class BEQ_FM_MM<bits<6> op> : MMArch {
  bits<5>  rs;
  bits<5>  rt;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `BEQ_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `BEQ_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 568-568
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 570-574
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 576-578
```tablegen
class BGEZ_FM_MM<bits<5> funct> : MMArch {
  bits<5>  rs;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `BGEZ_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `BGEZ_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 580-580
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 582-586
```tablegen
  let Inst{31-26} = 0x10;
  let Inst{25-21} = funct;
  let Inst{20-16} = rs;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 588-590
```tablegen
class BGEZAL_FM_MM<bits<5> funct> : MMArch {
  bits<5>  rs;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `BGEZAL_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `BGEZAL_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 592-592
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 594-598
```tablegen
  let Inst{31-26} = 0x10;
  let Inst{25-21} = funct;
  let Inst{20-16} = rs;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 600-601
```tablegen
class SYNC_FM_MM : MMArch {
  bits<5> stype;
```
- EN: Declares reusable TableGen class `SYNC_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `SYNC_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 603-603
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 605-610
```tablegen
  let Inst{31-26} = 0x00;
  let Inst{25-21} = 0x0;
  let Inst{20-16} = stype;
  let Inst{15-6}  = 0x1ad;
  let Inst{5-0}   = 0x3c;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 612-616
```tablegen
class SYNCI_FM_MM : MMArch {
  bits<21> addr;
  bits<5> rs = addr{20-16};
  bits<16> offset = addr{15-0};
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `SYNCI_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `SYNCI_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 618-622
```tablegen
  let Inst{31-26} = 0b010000;
  let Inst{25-21} = 0b10000;
  let Inst{20-16} = rs;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 624-632
```tablegen
class BRK_FM_MM : MMArch {
  bits<10> code_1;
  bits<10> code_2;
  bits<32> Inst;
  let Inst{31-26} = 0x0;
  let Inst{25-16} = code_1;
  let Inst{15-6}  = code_2;
  let Inst{5-0}   = 0x07;
}
```
- EN: Declares reusable TableGen class `BRK_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `BRK_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 634-641
```tablegen
class SYS_FM_MM : MMArch {
  bits<10> code_;
  bits<32> Inst;
  let Inst{31-26} = 0x0;
  let Inst{25-16} = code_;
  let Inst{15-6}  = 0x22d;
  let Inst{5-0}   = 0x3c;
}
```
- EN: Declares reusable TableGen class `SYS_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `SYS_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 643-645
```tablegen
class WAIT_FM_MM : MMArch {
  bits<10> code_;
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `WAIT_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `WAIT_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 647-651
```tablegen
  let Inst{31-26} = 0x00;
  let Inst{25-16} = code_;
  let Inst{15-6}  = 0x24d;
  let Inst{5-0}   = 0x3c;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 653-654
```tablegen
class ER_FM_MM<bits<10> funct> : MMArch {
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `ER_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `ER_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 656-660
```tablegen
  let Inst{31-26} = 0x00;
  let Inst{25-16} = 0x00;
  let Inst{15-6}  = funct;
  let Inst{5-0}   = 0x3c;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 662-664
```tablegen
class EI_FM_MM<bits<10> funct> : MMArch {
  bits<32> Inst;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `EI_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `EI_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 666-671
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

### Lines 673-676
```tablegen
class TEQ_FM_MM<bits<6> funct> : MMArch {
  bits<5> rs;
  bits<5> rt;
  bits<4> code_;
```
- EN: Declares reusable TableGen class `TEQ_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `TEQ_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 678-678
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 680-686
```tablegen
  let Inst{31-26} = 0x00;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-12} = code_;
  let Inst{11-6}  = funct;
  let Inst{5-0}   = 0x3c;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 688-690
```tablegen
class TEQI_FM_MM<bits<5> funct> : MMArch {
  bits<5> rs;
  bits<16> imm16;
```
- EN: Declares reusable TableGen class `TEQI_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `TEQI_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 692-692
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 694-698
```tablegen
  let Inst{31-26} = 0x10;
  let Inst{25-21} = funct;
  let Inst{20-16} = rs;
  let Inst{15-0}  = imm16;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 700-702
```tablegen
class LL_FM_MM<bits<4> funct> : MMArch {
  bits<5> rt;
  bits<21> addr;
```
- EN: Declares reusable TableGen class `LL_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `LL_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 704-704
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 706-711
```tablegen
  let Inst{31-26} = 0x18;
  let Inst{25-21} = rt;
  let Inst{20-16} = addr{20-16};
  let Inst{15-12} = funct;
  let Inst{11-0}  = addr{11-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 713-717
```tablegen
class LLE_FM_MM<bits<4> funct> : MMArch {
  bits<5> rt;
  bits<21> addr;
  bits<5> base = addr{20-16};
  bits<9> offset = addr{8-0};
```
- EN: Declares reusable TableGen class `LLE_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `LLE_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 719-719
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 721-727
```tablegen
  let Inst{31-26} = 0x18;
  let Inst{25-21} = rt;
  let Inst{20-16} = base;
  let Inst{15-12} = funct;
  let Inst{11-9} = 0x6;
  let Inst{8-0} = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 729-732
```tablegen
class ADDS_FM_MM<bits<2> fmt, bits<8> funct> : MMArch {
  bits<5> ft;
  bits<5> fs;
  bits<5> fd;
```
- EN: Declares reusable TableGen class `ADDS_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `ADDS_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 734-734
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 736-742
```tablegen
  let Inst{31-26} = 0x15;
  let Inst{25-21} = ft;
  let Inst{20-16} = fs;
  let Inst{15-11} = fd;
  let Inst{10}    = 0;
  let Inst{9-8}   = fmt;
  let Inst{7-0}   = funct;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 744-744
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 746-749
```tablegen
class LWXC1_FM_MM<bits<9> funct> : MMArch {
  bits<5> fd;
  bits<5> base;
  bits<5> index;
```
- EN: Declares reusable TableGen class `LWXC1_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `LWXC1_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 751-751
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 753-759
```tablegen
  let Inst{31-26} = 0x15;
  let Inst{25-21} = index;
  let Inst{20-16} = base;
  let Inst{15-11} = fd;
  let Inst{10-9}  = 0x0;
  let Inst{8-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 761-764
```tablegen
class SWXC1_FM_MM<bits<9> funct> : MMArch {
  bits<5> fs;
  bits<5> base;
  bits<5> index;
```
- EN: Declares reusable TableGen class `SWXC1_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `SWXC1_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 766-766
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 768-774
```tablegen
  let Inst{31-26} = 0x15;
  let Inst{25-21} = index;
  let Inst{20-16} = base;
  let Inst{15-11} = fs;
  let Inst{10-9}  = 0x0;
  let Inst{8-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 776-780
```tablegen
class CEQS_FM_MM<bits<2> fmt> : MMArch {
  bits<5> fs;
  bits<5> ft;
  bits<3> fcc;
  bits<4> cond;
```
- EN: Declares reusable TableGen class `CEQS_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `CEQS_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 782-782
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 784-792
```tablegen
  let Inst{31-26} = 0x15;
  let Inst{25-21} = ft;
  let Inst{20-16} = fs;
  let Inst{15-13} = fcc;
  let Inst{12}    = 0;
  let Inst{11-10} = fmt;
  let Inst{9-6}   = cond;
  let Inst{5-0}   = 0x3c;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 794-796
```tablegen
class C_COND_FM_MM<bits <2> fmt, bits<4> c> : CEQS_FM_MM<fmt> {
  let cond = c;
}
```
- EN: Declares reusable TableGen class `C_COND_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `C_COND_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 798-800
```tablegen
class BC1F_FM_MM<bits<5> tf> : MMArch {
  bits<3> fcc;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `BC1F_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `BC1F_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 802-802
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 804-809
```tablegen
  let Inst{31-26} = 0x10;
  let Inst{25-21} = tf;
  let Inst{20-18} = fcc; // cc
  let Inst{17-16} = 0x0;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 811-813
```tablegen
class ROUND_W_FM_MM<bits<1> fmt, bits<8> funct> : MMArch {
  bits<5> fd;
  bits<5> fs;
```
- EN: Declares reusable TableGen class `ROUND_W_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `ROUND_W_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 815-815
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 817-824
```tablegen
  let Inst{31-26} = 0x15;
  let Inst{25-21} = fd;
  let Inst{20-16} = fs;
  let Inst{15}    = 0;
  let Inst{14}    = fmt;
  let Inst{13-6}  = funct;
  let Inst{5-0}   = 0x3b;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 826-828
```tablegen
class ABS_FM_MM<bits<2> fmt, bits<7> funct> : MMArch {
  bits<5> fd;
  bits<5> fs;
```
- EN: Declares reusable TableGen class `ABS_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `ABS_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 830-830
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 832-839
```tablegen
  let Inst{31-26} = 0x15;
  let Inst{25-21} = fd;
  let Inst{20-16} = fs;
  let Inst{15}    = 0;
  let Inst{14-13} = fmt;
  let Inst{12-6}  = funct;
  let Inst{5-0}   = 0x3b;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 841-845
```tablegen
class CMov_F_F_FM_MM<bits<9> func, bits<2> fmt> : MMArch {
  bits<5> fd;
  bits<5> fs;
  bits<3> fcc;
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `CMov_F_F_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `CMov_F_F_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 847-854
```tablegen
  let Inst{31-26} = 0x15;
  let Inst{25-21} = fd;
  let Inst{20-16} = fs;
  let Inst{15-13} = fcc; //cc
  let Inst{12-11} = 0x0;
  let Inst{10-9}  = fmt;
  let Inst{8-0}   = func;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 856-859
```tablegen
class CMov_I_F_FM_MM<bits<8> funct, bits<2> fmt> : MMArch {
  bits<5> fd;
  bits<5> fs;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `CMov_I_F_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `CMov_I_F_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 861-861
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 863-869
```tablegen
  let Inst{31-26} = 0x15;
  let Inst{25-21} = rt;
  let Inst{20-16} = fs;
  let Inst{15-11} = fd;
  let Inst{9-8}   = fmt;
  let Inst{7-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 871-873
```tablegen
class MFC1_FM_MM<bits<8> funct> : MMArch {
  bits<5> rt;
  bits<5> fs;
```
- EN: Declares reusable TableGen class `MFC1_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `MFC1_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 875-875
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 877-883
```tablegen
  let Inst{31-26} = 0x15;
  let Inst{25-21} = rt;
  let Inst{20-16} = fs;
  let Inst{15-14} = 0x0;
  let Inst{13-6}  = funct;
  let Inst{5-0}   = 0x3b;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 885-889
```tablegen
class MADDS_FM_MM<bits<6> funct>: MMArch {
  bits<5> ft;
  bits<5> fs;
  bits<5> fd;
  bits<5> fr;
```
- EN: Declares reusable TableGen class `MADDS_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `MADDS_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 891-891
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 893-899
```tablegen
  let Inst{31-26} = 0x15;
  let Inst{25-21} = ft;
  let Inst{20-16} = fs;
  let Inst{15-11} = fd;
  let Inst{10-6}  = fr;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 901-903
```tablegen
class COMPACT_BRANCH_FM_MM<bits<5> funct> {
  bits<5>  rs;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `COMPACT_BRANCH_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `COMPACT_BRANCH_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 905-905
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 907-911
```tablegen
  let Inst{31-26} = 0x10;
  let Inst{25-21} = funct;
  let Inst{20-16} = rs;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 913-914
```tablegen
class COP0_TLB_FM_MM<bits<10> op> : MMArch {
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `COP0_TLB_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `COP0_TLB_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 916-920
```tablegen
  let Inst{31-26} = 0x0;
  let Inst{25-16} = 0x0;
  let Inst{15-6}  = op;
  let Inst{5-0}   = 0x3c;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 922-923
```tablegen
class SDBBP_FM_MM : MMArch {
  bits<10> code_;
```
- EN: Declares reusable TableGen class `SDBBP_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `SDBBP_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 925-925
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 927-931
```tablegen
  let Inst{31-26} = 0x0;
  let Inst{25-16} = code_;
  let Inst{15-6}  = 0x36d;
  let Inst{5-0}   = 0x3c;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 933-934
```tablegen
class SIGRIE_FM_MM : MMArch {
  bits<16> code_;
```
- EN: Declares reusable TableGen class `SIGRIE_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `SIGRIE_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 936-936
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 938-942
```tablegen
  let Inst{31-26} = 0x0;
  let Inst{25-22} = 0x0;
  let Inst{21-6} = code_;
  let Inst{5-0} = 0b111111;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 944-946
```tablegen
class RDHWR_FM_MM : MMArch {
  bits<5> rt;
  bits<5> rd;
```
- EN: Declares reusable TableGen class `RDHWR_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `RDHWR_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 948-948
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 950-955
```tablegen
  let Inst{31-26} = 0x0;
  let Inst{25-21} = rt;
  let Inst{20-16} = rd;
  let Inst{15-6}  = 0x1ac;
  let Inst{5-0}   = 0x3c;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 957-960
```tablegen
class LWXS_FM_MM<bits<10> funct> {
  bits<5> rd;
  bits<5> base;
  bits<5> index;
```
- EN: Declares reusable TableGen class `LWXS_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `LWXS_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 962-962
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 964-970
```tablegen
  let Inst{31-26} = 0x0;
  let Inst{25-21} = index;
  let Inst{20-16} = base;
  let Inst{15-11} = rd;
  let Inst{10}    = 0;
  let Inst{9-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 972-974
```tablegen
class LWM_FM_MM<bits<4> funct> : MMArch {
  bits<5> rt;
  bits<21> addr;
```
- EN: Declares reusable TableGen class `LWM_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `LWM_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 976-976
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 978-983
```tablegen
  let Inst{31-26} = 0x8;
  let Inst{25-21} = rt;
  let Inst{20-16} = addr{20-16};
  let Inst{15-12} = funct;
  let Inst{11-0}  = addr{11-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 985-987
```tablegen
class LWM_FM_MM16<bits<4> funct> : MMArch {
  bits<2> rt;
  bits<4> addr;
```
- EN: Declares reusable TableGen class `LWM_FM_MM16` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `LWM_FM_MM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 989-989
```tablegen
  bits<16> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 991-995
```tablegen
  let Inst{15-10} = 0x11;
  let Inst{9-6}   = funct;
  let Inst{5-4}   = rt;
  let Inst{3-0}   = addr;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 997-1001
```tablegen
class CACHE_PREF_FM_MM<bits<6> op, bits<4> funct> : MMArch {
  bits<21> addr;
  bits<5> hint;
  bits<5> base = addr{20-16};
  bits<12> offset = addr{11-0};
```
- EN: Declares reusable TableGen class `CACHE_PREF_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `CACHE_PREF_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1003-1003
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1005-1010
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = hint;
  let Inst{20-16} = base;
  let Inst{15-12} = funct;
  let Inst{11-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1012-1016
```tablegen
class CACHE_PREFE_FM_MM<bits<6> op, bits<3> funct> : MMArch {
  bits<21> addr;
  bits<5> hint;
  bits<5> base = addr{20-16};
  bits<9> offset = addr{8-0};
```
- EN: Declares reusable TableGen class `CACHE_PREFE_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `CACHE_PREFE_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1018-1018
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1020-1026
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = hint;
  let Inst{20-16} = base;
  let Inst{15-12} = 0xA;
  let Inst{11-9} = funct;
  let Inst{8-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1028-1031
```tablegen
class POOL32F_PREFX_FM_MM<bits<6> op, bits<9> funct> : MMArch {
  bits<5> index;
  bits<5> base;
  bits<5> hint;
```
- EN: Declares reusable TableGen class `POOL32F_PREFX_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `POOL32F_PREFX_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1033-1033
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1035-1041
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = index;
  let Inst{20-16} = base;
  let Inst{15-11} = hint;
  let Inst{10-9}  = 0x0;
  let Inst{8-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1043-1044
```tablegen
class BARRIER_FM_MM<bits<5> op> : MMArch {
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `BARRIER_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `BARRIER_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1046-1052
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

### Lines 1054-1056
```tablegen
class ADDIUPC_FM_MM {
  bits<3> rs;
  bits<23> imm;
```
- EN: Declares reusable TableGen class `ADDIUPC_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `ADDIUPC_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1058-1058
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1060-1063
```tablegen
  let Inst{31-26} = 0x1e;
  let Inst{25-23} = rs;
  let Inst{22-0} = imm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1065-1067
```tablegen
class POOL32A_CFTC2_FM_MM<bits<10> funct> : MMArch {
  bits<5> rt;
  bits<5> impl;
```
- EN: Declares reusable TableGen class `POOL32A_CFTC2_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `POOL32A_CFTC2_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1069-1069
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1071-1076
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

### Lines 1078-1079
```tablegen
class POOL32A_TLBINV_FM_MM<bits<10> funct> : MMArch {
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `POOL32A_TLBINV_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `POOL32A_TLBINV_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1081-1085
```tablegen
  let Inst{31-26} = 0x0;
  let Inst{25-16} = 0x0;
  let Inst{15-6}  = funct;
  let Inst{5-0}   = 0b111100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1087-1090
```tablegen
class POOL32A_MFTC0_FM_MM<bits<5> funct, bits<6> opcode> : MMArch {
  bits<5> rt;
  bits<5> rs;
  bits<3> sel;
```
- EN: Declares reusable TableGen class `POOL32A_MFTC0_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `POOL32A_MFTC0_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1092-1092
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1094-1101
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

### Lines 1103-1104
```tablegen
class POOL32A_HYPCALL_FM_MM : MMArch {
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `POOL32A_HYPCALL_FM_MM` for `MicroMipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrFormats` 声明可复用的 TableGen 类 `POOL32A_HYPCALL_FM_MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1106-1106
```tablegen
  bits<10> code_;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1108-1112
```tablegen
  let Inst{31-26} = 0x0;
  let Inst{25-16} = code_;
  let Inst{15-6}  = 0b1100001101;
  let Inst{5-0}   = 0b111100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

## Key Concepts / 关键概念

- EN: Primary role: instruction encoding formats and bitfield layout.
  - CN: 核心职责：指令编码格式与位域布局。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: SelectionDAG patterns or node profiles connect IR-level intent to target instructions.
  - CN: SelectionDAG 模式或节点轮廓把 IR 层意图连接到目标指令。
- EN: Bitfield assignments describe exact instruction encodings and per-instruction flags.
  - CN: 位字段赋值用于描述精确的指令编码以及每条指令的标志位。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
