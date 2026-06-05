# Mips16InstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/Mips16InstrFormats.td`
- Repository: `llvm-project`
- Purpose (EN): Describe MIPS instructions format CPU INSTRUCTION FORMATS funct or f      Function field immediate       4-,5-,8- or 11-bit immediate, branch displacement, or or imm          address displacement op              5-bit major operation code rx              3-bit source or destination register ry              3-bit source or destination register rz              3-bit source or destination register sa              3- or 5-bit shift amount.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `Mips16InstrFormats`，涵盖指令编码格式与位域布局。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===- Mips16InstrFormats.td - Mips Instruction Formats ----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 9-26
```tablegen
//===----------------------------------------------------------------------===//
//  Describe MIPS instructions format
//
//  CPU INSTRUCTION FORMATS
//
//  funct or f      Function field
//
//  immediate       4-,5-,8- or 11-bit immediate, branch displacement, or
//  or imm          address displacement
//
//  op              5-bit major operation code
//
//  rx              3-bit source or destination register
//
//  ry              3-bit source or destination register
//
//  rz              3-bit source or destination register
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 27-29
```tablegen
//  sa              3- or 5-bit shift amount
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 32-37
```tablegen
// Base class for Mips 16 Format
// This class does not depend on the instruction size
//
class MipsInst16_Base<dag outs, dag ins, string asmstr,
                      list<dag> pattern>: Instruction
{
```
- EN: Declares reusable TableGen class `for` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `for`，通常用于抽象共享字段、谓词或编码结构。

### Lines 39-39
```tablegen
  let Namespace = "Mips";
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 41-42
```tablegen
  let OutOperandList = outs;
  let InOperandList  = ins;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 44-45
```tablegen
  let AsmString   = asmstr;
  let Pattern     = pattern;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 47-48
```tablegen
  let Predicates = [InMips16Mode];
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 50-58
```tablegen
//
// Generic Mips 16 Format
//
class MipsInst16<dag outs, dag ins, string asmstr,
                 list<dag> pattern>:
  MipsInst16_Base<outs, ins, asmstr, pattern>
{
  field bits<16> Inst;
  bits<5> Opcode = 0;
```
- EN: Declares reusable TableGen class `MipsInst16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `MipsInst16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 60-61
```tablegen
  // Top 5 bits are the 'opcode' field
  let Inst{15-11} = Opcode;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 63-64
```tablegen
  let Size=2;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 66-73
```tablegen
//
// For 32 bit extended instruction forms.
//
class MipsInst16_32<dag outs, dag ins, string asmstr,
                    list<dag> pattern>:
  MipsInst16_Base<outs, ins, asmstr, pattern>
{
  field bits<32> Inst;
```
- EN: Declares reusable TableGen class `MipsInst16_32` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `MipsInst16_32`，通常用于抽象共享字段、谓词或编码结构。

### Lines 75-76
```tablegen
  let Size=4;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 78-83
```tablegen
class MipsInst16_EXTEND<dag outs, dag ins, string asmstr,
                        list<dag> pattern>:
  MipsInst16_32<outs, ins, asmstr, pattern>
{
  let Inst{31-27} = 0b11110;
}
```
- EN: Declares reusable TableGen class `MipsInst16_EXTEND` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `MipsInst16_EXTEND`，通常用于抽象共享字段、谓词或编码结构。

### Lines 87-92
```tablegen
// Mips Pseudo Instructions Format
class MipsPseudo16<dag outs, dag ins, string asmstr, list<dag> pattern>:
  MipsInst16<outs, ins, asmstr, pattern> {
  let isCodeGenOnly = 1;
  let isPseudo = 1;
}
```
- EN: Declares reusable TableGen class `MipsPseudo16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `MipsPseudo16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 95-97
```tablegen
//===----------------------------------------------------------------------===//
// Format I instruction class in Mips : <|opcode|imm11|>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 99-103
```tablegen
class FI16<bits<5> op, dag outs, dag ins, string asmstr,
           list<dag> pattern>:
  MipsInst16<outs, ins, asmstr, pattern>
{
  bits<11> imm11;
```
- EN: Declares reusable TableGen class `FI16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FI16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 105-105
```tablegen
  let Opcode = op;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 107-108
```tablegen
  let Inst{10-0}  = imm11;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 110-112
```tablegen
//===----------------------------------------------------------------------===//
// Format RI instruction class in Mips : <|opcode|rx|imm8|>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 114-119
```tablegen
class FRI16<bits<5> op, dag outs, dag ins, string asmstr,
            list<dag> pattern>:
  MipsInst16<outs, ins, asmstr, pattern>
{
  bits<3>  rx;
  bits<8>   imm8;
```
- EN: Declares reusable TableGen class `FRI16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FRI16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 121-121
```tablegen
  let Opcode = op;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 123-125
```tablegen
  let Inst{10-8} = rx;
  let Inst{7-0} = imm8;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 127-129
```tablegen
//===----------------------------------------------------------------------===//
// Format RR instruction class in Mips : <|opcode|rx|ry|funct|>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 131-137
```tablegen
class FRR16<bits<5> _funct, dag outs, dag ins, string asmstr,
            list<dag> pattern>:
  MipsInst16<outs, ins, asmstr, pattern>
{
  bits<3>  rx;
  bits<3>  ry;
  bits<5>  funct;
```
- EN: Declares reusable TableGen class `FRR16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FRR16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 139-140
```tablegen
  let Opcode = 0b11101;
  let funct  = _funct;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 142-145
```tablegen
  let Inst{10-8} = rx;
  let Inst{7-5} = ry;
  let Inst{4-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 147-152
```tablegen
class FRRBreak16<dag outs, dag ins, string asmstr,
                 list<dag> pattern>:
  MipsInst16<outs, ins, asmstr, pattern>
{
  bits<6>  Code;
  bits<5>  funct;
```
- EN: Declares reusable TableGen class `FRRBreak16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FRRBreak16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 154-155
```tablegen
  let Opcode = 0b11101;
  let funct  = 0b00101;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 157-159
```tablegen
  let Inst{10-5} = Code;
  let Inst{4-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 161-170
```tablegen
//
// For conversion functions.
//
class FRR_SF16<bits<5> _funct, bits<3> _subfunct, dag outs, dag ins,
               string asmstr, list<dag> pattern>:
  MipsInst16<outs, ins, asmstr, pattern>
{
  bits<3>  rx;
  bits<3>  subfunct;
  bits<5>  funct;
```
- EN: Declares reusable TableGen class `FRR_SF16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FRR_SF16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 172-174
```tablegen
  let Opcode = 0b11101; // RR
  let funct  = _funct;
  let subfunct = _subfunct;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 176-179
```tablegen
  let Inst{10-8} = rx;
  let Inst{7-5} = subfunct;
  let Inst{4-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 181-189
```tablegen
//
// just used for breakpoint (hardware and software) instructions.
//
class FC16<bits<5> _funct, dag outs, dag ins, string asmstr,
           list<dag> pattern>:
  MipsInst16<outs, ins, asmstr, pattern>
{
  bits<6>  _code;  // code is a keyword in tablegen
  bits<5>  funct;
```
- EN: Declares reusable TableGen class `FC16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FC16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 191-192
```tablegen
  let Opcode = 0b11101; // RR
  let funct  = _funct;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 194-196
```tablegen
  let Inst{10-5} = _code;
  let Inst{4-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 198-209
```tablegen
//
// J(AL)R(C) subformat
//
class FRR16_JALRC<bits<1> _nd, bits<1> _l, bits<1> r_a,
                  dag outs, dag ins, string asmstr,
                  list<dag> pattern>:
  MipsInst16<outs, ins, asmstr, pattern>
{
  bits<3>  rx;
  bits<1>  nd;
  bits<1>  l;
  bits<1>  ra;
```
- EN: Declares reusable TableGen class `FRR16_JALRC` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FRR16_JALRC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 211-213
```tablegen
  let nd = _nd;
  let l = _l;
  let ra = r_a;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 215-215
```tablegen
  let Opcode = 0b11101;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 217-222
```tablegen
  let Inst{10-8} = rx;
  let Inst{7} = nd;
  let Inst{6} = l;
  let Inst{5} = ra;
  let Inst{4-0} = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 224-226
```tablegen
//===----------------------------------------------------------------------===//
// Format RRI instruction class in Mips : <|opcode|rx|ry|imm5|>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 228-234
```tablegen
class FRRI16<bits<5> op, dag outs, dag ins, string asmstr,
             list<dag> pattern>:
  MipsInst16<outs, ins, asmstr, pattern>
{
  bits<3>  rx;
  bits<3>  ry;
  bits<5>  imm5;
```
- EN: Declares reusable TableGen class `FRRI16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FRRI16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 236-236
```tablegen
  let Opcode = op;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 239-242
```tablegen
  let Inst{10-8} = rx;
  let Inst{7-5} = ry;
  let Inst{4-0}   = imm5;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 244-246
```tablegen
//===----------------------------------------------------------------------===//
// Format RRR instruction class in Mips : <|opcode|rx|ry|rz|f|>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 248-255
```tablegen
class FRRR16<bits<2> _f, dag outs, dag ins, string asmstr,
             list<dag> pattern>:
  MipsInst16<outs, ins, asmstr, pattern>
{
  bits<3>  rx;
  bits<3>  ry;
  bits<3>  rz;
  bits<2>  f;
```
- EN: Declares reusable TableGen class `FRRR16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FRRR16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 257-258
```tablegen
  let Opcode = 0b11100;
  let f  = _f;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 260-264
```tablegen
  let Inst{10-8} = rx;
  let Inst{7-5} = ry;
  let Inst{4-2} = rz;
  let Inst{1-0}   = f;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 266-268
```tablegen
//===----------------------------------------------------------------------===//
// Format RRI-A instruction class in Mips : <|opcode|rx|ry|f|imm4|>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 270-277
```tablegen
class FRRI_A16<bits<1> _f, dag outs, dag ins, string asmstr,
               list<dag> pattern>:
  MipsInst16<outs, ins, asmstr, pattern>
{
  bits<3>  rx;
  bits<3>  ry;
  bits<1>  f;
  bits<4>  imm4;
```
- EN: Declares reusable TableGen class `FRRI_A16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FRRI_A16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 279-280
```tablegen
  let Opcode = 0b01000;
  let  f = _f;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 282-286
```tablegen
  let Inst{10-8} = rx;
  let Inst{7-5} = ry;
  let Inst{4} = f;
  let Inst{3-0}   = imm4;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 288-290
```tablegen
//===----------------------------------------------------------------------===//
// Format Shift instruction class in Mips : <|opcode|rx|ry|sa|f|>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 292-299
```tablegen
class FSHIFT16<bits<2> _f, dag outs, dag ins, string asmstr,
               list<dag> pattern>:
  MipsInst16<outs, ins, asmstr, pattern>
{
  bits<3>  rx;
  bits<3>  ry;
  bits<3>  sa;
  bits<2>  f;
```
- EN: Declares reusable TableGen class `FSHIFT16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FSHIFT16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 301-302
```tablegen
  let Opcode = 0b00110;
  let f  = _f;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 304-308
```tablegen
  let Inst{10-8} = rx;
  let Inst{7-5} = ry;
  let Inst{4-2} = sa;
  let Inst{1-0}   = f;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 310-312
```tablegen
//===----------------------------------------------------------------------===//
// Format i8 instruction class in Mips : <|opcode|funct|imm8>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 314-319
```tablegen
class FI816<bits<3> _func, dag outs, dag ins, string asmstr,
            list<dag> pattern>:
  MipsInst16<outs, ins, asmstr, pattern>
{
  bits<3>  func;
  bits<8>   imm8;
```
- EN: Declares reusable TableGen class `FI816` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FI816`，通常用于抽象共享字段、谓词或编码结构。

### Lines 321-322
```tablegen
  let Opcode = 0b01100;
  let func  = _func;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 324-326
```tablegen
  let Inst{10-8} = func;
  let Inst{7-0} = imm8;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 328-330
```tablegen
//===----------------------------------------------------------------------===//
// Format i8_MOVR32 instruction class in Mips : <|opcode|func|ry|r32>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 332-338
```tablegen
class FI8_MOVR3216<dag outs, dag ins, string asmstr,
                   list<dag> pattern>:
  MipsInst16<outs, ins, asmstr, pattern>
{
  // FIXME: this seems wrong? 'ry' should be 3 bits, and 'r32' 5?
  bits<4> ry;
  bits<4> r32;
```
- EN: Declares reusable TableGen class `FI8_MOVR3216` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FI8_MOVR3216`，通常用于抽象共享字段、谓词或编码结构。

### Lines 340-340
```tablegen
  let Opcode = 0b01100;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 342-344
```tablegen
  let Inst{10-8} = 0b111;
  let Inst{7-4} = ry;
  let Inst{3-0} = r32;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 346-346
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 350-352
```tablegen
//===----------------------------------------------------------------------===//
// Format i8_MOV32R instruction class in Mips : <|opcode|func|r32|rz>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 354-357
```tablegen
class FI8_MOV32R16<dag outs, dag ins, string asmstr,
                   list<dag> pattern>:
  MipsInst16<outs, ins, asmstr, pattern>
{
```
- EN: Declares reusable TableGen class `FI8_MOV32R16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FI8_MOV32R16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 359-361
```tablegen
  bits<3>  func;
  bits<5> r32;
  bits<3> rz;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 364-364
```tablegen
  let Opcode = 0b01100;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 366-369
```tablegen
  let Inst{10-8} = 0b101;
  let Inst{7-5} = r32{2-0};
  let Inst{4-3} = r32{4-3};
  let Inst{2-0} = rz;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 371-371
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 373-376
```tablegen
//===----------------------------------------------------------------------===//
// Format i8_SVRS instruction class in Mips :
//    <|opcode|svrs|s|ra|s0|s1|framesize>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 378-386
```tablegen
class FI8_SVRS16<bits<1> _s, dag outs, dag ins, string asmstr,
                 list<dag> pattern>:
  MipsInst16<outs, ins, asmstr, pattern>
{
  bits<1> s;
  bits<1> ra = 0;
  bits<1> s0 = 0;
  bits<1> s1 = 0;
  bits<4> framesize = 0;
```
- EN: Declares reusable TableGen class `FI8_SVRS16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FI8_SVRS16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 388-389
```tablegen
  let s =_s;
  let Opcode = 0b01100;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 391-396
```tablegen
  let Inst{10-8} = 0b100;
  let Inst{7} = s;
  let Inst{6} = ra;
  let Inst{5} = s0;
  let Inst{4} = s1;
  let Inst{3-0} = framesize;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 398-398
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 400-403
```tablegen
//===----------------------------------------------------------------------===//
// Format JAL instruction class in Mips16 :
//    <|opcode|svrs|s|ra|s0|s1|framesize>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 405-410
```tablegen
class FJAL16<bits<1> _X, dag outs, dag ins, string asmstr,
             list<dag> pattern>:
  MipsInst16_32<outs, ins, asmstr, pattern>
{
  bits<1> X;
  bits<26> imm26;
```
- EN: Declares reusable TableGen class `FJAL16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FJAL16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 413-413
```tablegen
  let X = _X;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 415-419
```tablegen
  let Inst{31-27} = 0b00011;
  let Inst{26} = X;
  let Inst{25-21} = imm26{20-16};
  let Inst{20-16} = imm26{25-21};
  let Inst{15-0}  = imm26{15-0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 421-421
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 423-426
```tablegen
//===----------------------------------------------------------------------===//
// Format EXT-I instruction class in Mips16 :
//     <|EXTEND|imm10:5|imm15:11|op|0|0|0|0|0|0|imm4:0>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 428-433
```tablegen
class FEXT_I16<bits<5> _eop, dag outs, dag ins, string asmstr,
               list<dag> pattern>:
  MipsInst16_EXTEND<outs, ins, asmstr, pattern>
{
  bits<16> imm16;
  bits<5> eop;
```
- EN: Declares reusable TableGen class `FEXT_I16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FEXT_I16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 435-435
```tablegen
  let eop = _eop;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 437-441
```tablegen
  let Inst{26-21} = imm16{10-5};
  let Inst{20-16} = imm16{15-11};
  let Inst{15-11} = eop;
  let Inst{10-5} = 0;
  let Inst{4-0} = imm16{4-0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 443-443
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 445-448
```tablegen
//===----------------------------------------------------------------------===//
// Format ASMACRO instruction class in Mips16 :
//    <EXTEND|select|p4|p3|RRR|p2|p1|p0>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 450-460
```tablegen
class FASMACRO16<dag outs, dag ins, string asmstr,
                 list<dag> pattern>:
  MipsInst16_EXTEND<outs, ins, asmstr, pattern>
{
  bits<3> select;
  bits<3> p4;
  bits<5> p3;
  bits<5> RRR = 0b11100;
  bits<3> p2;
  bits<3> p1;
  bits<5> p0;
```
- EN: Declares reusable TableGen class `FASMACRO16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FASMACRO16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 463-469
```tablegen
  let Inst{26-24} = select;
  let Inst{23-21} = p4;
  let Inst{20-16} = p3;
  let Inst{15-11} = RRR;
  let Inst{10-8} = p2;
  let Inst{7-5} = p1;
  let Inst{4-0} = p0;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 471-471
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 474-477
```tablegen
//===----------------------------------------------------------------------===//
// Format EXT-RI instruction class in Mips16 :
//    <|EXTEND|imm10:5|imm15:11|op|rx|0|0|0|imm4:0>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 479-485
```tablegen
class FEXT_RI16<bits<5> _op, dag outs, dag ins, string asmstr,
                list<dag> pattern>:
  MipsInst16_EXTEND<outs, ins, asmstr, pattern>
{
  bits<16> imm16;
  bits<5> op;
  bits<3> rx;
```
- EN: Declares reusable TableGen class `FEXT_RI16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FEXT_RI16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 487-487
```tablegen
  let op = _op;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 489-494
```tablegen
  let Inst{26-21} = imm16{10-5};
  let Inst{20-16} = imm16{15-11};
  let Inst{15-11} = op;
  let Inst{10-8} = rx;
  let Inst{7-5} = 0;
  let Inst{4-0} = imm16{4-0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 496-496
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 498-501
```tablegen
//===----------------------------------------------------------------------===//
// Format EXT-RRI instruction class in Mips16 :
//     <|EXTEND|imm10:5|imm15:11|op|rx|ry|imm4:0>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 503-510
```tablegen
class FEXT_RRI16<bits<5> _op, dag outs, dag ins, string asmstr,
                 list<dag> pattern>:
  MipsInst16_EXTEND<outs, ins, asmstr, pattern>
{
  bits<5> op;
  bits<16> imm16;
  bits<3> rx;
  bits<3> ry;
```
- EN: Declares reusable TableGen class `FEXT_RRI16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FEXT_RRI16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 512-512
```tablegen
  let op=_op;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 514-519
```tablegen
  let Inst{26-21} = imm16{10-5};
  let Inst{20-16} = imm16{15-11};
  let Inst{15-11} = op;
  let Inst{10-8} = rx;
  let Inst{7-5} = ry;
  let Inst{4-0} = imm16{4-0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 521-521
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 523-526
```tablegen
//===----------------------------------------------------------------------===//
// Format EXT-RRI-A instruction class in Mips16 :
//    <|EXTEND|imm10:4|imm14:11|RRI-A|rx|ry|f|imm3:0>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 528-535
```tablegen
class FEXT_RRI_A16<bits<1> _f, dag outs, dag ins, string asmstr,
                   list<dag> pattern>:
  MipsInst16_EXTEND<outs, ins, asmstr, pattern>
{
  bits<15> imm15;
  bits<3> rx;
  bits<3> ry;
  bits<1> f;
```
- EN: Declares reusable TableGen class `FEXT_RRI_A16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FEXT_RRI_A16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 537-537
```tablegen
  let f = _f;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 539-545
```tablegen
  let Inst{26-20} = imm15{10-4};
  let Inst{19-16} = imm15{14-11};
  let Inst{15-11} = 0b01000;
  let Inst{10-8} = rx;
  let Inst{7-5} = ry;
  let Inst{4} = f;
  let Inst{3-0} = imm15{3-0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 547-547
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 549-552
```tablegen
//===----------------------------------------------------------------------===//
// Format EXT-SHIFT instruction class in Mips16 :
//    <|EXTEND|sa 4:0|s5|0|SHIFT|rx|ry|0|f>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 554-561
```tablegen
class FEXT_SHIFT16<bits<2> _f, dag outs, dag ins, string asmstr,
                   list<dag> pattern>:
  MipsInst16_EXTEND<outs, ins, asmstr, pattern>
{
  bits<6> sa6;
  bits<3> rx;
  bits<3> ry;
  bits<2> f;
```
- EN: Declares reusable TableGen class `FEXT_SHIFT16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FEXT_SHIFT16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 563-563
```tablegen
  let f = _f;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 565-572
```tablegen
  let Inst{26-22} = sa6{4-0};
  let Inst{21} = sa6{5};
  let Inst{20-16} = 0;
  let Inst{15-11} = 0b00110;
  let Inst{10-8} = rx;
  let Inst{7-5} = ry;
  let Inst{4-2} = 0;
  let Inst{1-0} = f;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 574-574
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 576-579
```tablegen
//===----------------------------------------------------------------------===//
// Format EXT-I8 instruction class in Mips16 :
//    <|EXTEND|imm10:5|imm15:11|I8|funct|0|imm4:0>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 581-587
```tablegen
class FEXT_I816<bits<3> _funct, dag outs, dag ins, string asmstr,
                list<dag> pattern>:
  MipsInst16_EXTEND<outs, ins, asmstr, pattern>
{
  bits<16> imm16;
  bits<5> I8;
  bits<3> funct;
```
- EN: Declares reusable TableGen class `FEXT_I816` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FEXT_I816`，通常用于抽象共享字段、谓词或编码结构。

### Lines 589-590
```tablegen
  let funct = _funct;
  let I8 = 0b00110;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 592-597
```tablegen
  let Inst{26-21} = imm16{10-5};
  let Inst{20-16} = imm16{15-11};
  let Inst{15-11} = I8;
  let Inst{10-8} = funct;
  let Inst{7-5} = 0;
  let Inst{4-0} = imm16{4-0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 599-599
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 601-604
```tablegen
//===----------------------------------------------------------------------===//
// Format EXT-I8_SVRS instruction class in Mips16 :
//    <|EXTEND|xsregs|framesize7:4|aregs|I8|SVRS|s|ra|s0|s1|framesize3:0>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 606-618
```tablegen
class FEXT_I8_SVRS16<bits<1> s_, dag outs, dag ins, string asmstr,
                     list<dag> pattern>:
  MipsInst16_EXTEND<outs, ins, asmstr, pattern>
{
  bits<3> xsregs =0;
  bits<8> framesize =0;
  bits<3> aregs =0;
  bits<5> I8 = 0b01100;
  bits<3> SVRS = 0b100;
  bits<1> s;
  bits<1> ra = 0;
  bits<1> s0 = 0;
  bits<1> s1 = 0;
```
- EN: Declares reusable TableGen class `FEXT_I8_SVRS16` for `Mips16InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrFormats` 声明可复用的 TableGen 类 `FEXT_I8_SVRS16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 620-620
```tablegen
  let s= s_;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 622-632
```tablegen
  let Inst{26-24} = xsregs;
  let Inst{23-20} = framesize{7-4};
  let Inst{19} = 0;
  let Inst{18-16} = aregs;
  let Inst{15-11} = I8;
  let Inst{10-8} = SVRS;
  let Inst{7} = s;
  let Inst{6} = ra;
  let Inst{5} = s0;
  let Inst{4} = s1;
  let Inst{3-0} = framesize{3-0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 635-635
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

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
