# RISCVInstrFormatsXAIF.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrFormatsXAIF.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for instruction format classes, bit layouts, and operand encodings for RISC-V. / 使用 TableGen 定义RISC-V 的指令格式类、位布局与操作数编码。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrFormatXAIF.td ----------------------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Instruction formats for the AI Foundry ET extensions, formerly known as the
// ET-SoC-1 Minion extensions by Esperanto Technologies.
//
// Full documentation for these extensions is publicly available at the
// following URL:
//
//   https://github.com/aifoundry-org/et-man/blob/main/ET%20Programmer's%20Reference%20Manual.pdf
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 18-28: Definitions and supporting logic / 定义与支撑逻辑
```tablegen

// Opcodes used by the ET extensions (referencing RISCVOpcode records defined in
// RISCVInstrFormats.td)
defvar OPC_ET_MEM_PS   = OPC_CUSTOM_0;
defvar OPC_ET_IMM_PS   = OPC_0011111;
defvar OPC_ET_IMM10_PI = OPC_0111111;
defvar OPC_ET_OP3_PS   = OPC_CUSTOM_2;
defvar OPC_ET_IMM_PI   = OPC_1011111;
defvar OPC_ET_CVT_PS   = OPC_OP_VE;
defvar OPC_ET_OP_PS    = OPC_CUSTOM_3;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 29-43: TableGen class RVInstET1<bits<7> / TableGen 类 RVInstET1<bits<7>
```tablegen
// Example: AIF.FSWIZZ.PS
class RVInstET1<bits<7> funct7, RISCVOpcode opcode, dag outs, dag ins,
                string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatOther> {
  bits<5> rd;
  bits<5> rs1;
  bits<8> imm;

  let Inst{31-25} = funct7;
  let Inst{24-20} = imm{7-3};
  let Inst{19-15} = rs1;
  let Inst{14-12} = imm{2-0};
  let Inst{11-7} = rd;
  let Inst{6-0} = opcode.Value;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 44-60: TableGen class RVInstET2<bits<3> / TableGen 类 RVInstET2<bits<3>
```tablegen

// Example: AIF.FADDI.PS
class RVInstET2<bits<3> funct3, RISCVOpcode opcode, dag outs, dag ins,
                string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatOther> {
  bits<10> imm;
  bits<5> rs1;
  bits<5> rd;

  let Inst{31-27} = imm{9-5};
  let Inst{26-25} = 0b10;
  let Inst{24-20} = imm{4-0};
  let Inst{19-15} = rs1;
  let Inst{14-12} = funct3;
  let Inst{11-7} = rd;
  let Inst{6-0} = opcode.Value;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 61-70: TableGen class RVInstET3<bits<7> / TableGen 类 RVInstET3<bits<7>
```tablegen

// Example: AIF.FADD.PS
class RVInstET3<bits<7> funct7, RISCVOpcode opcode, dag outs, dag ins,
                string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatOther> {
  bits<5> rd;
  bits<5> rs1;
  bits<5> rs2;
  bits<3> rm;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 71-88: TableGen class RVInstET4<bits<2> / TableGen 类 RVInstET4<bits<2>
```tablegen
  let Inst{31-25} = funct7;
  let Inst{24-20} = rs2;
  let Inst{19-15} = rs1;
  let Inst{14-12} = rm;
  let Inst{11-7} = rd;
  let Inst{6-0} = opcode.Value;
}

// Example: AIF.FMADD.PS
class RVInstET4<bits<2> funct2, RISCVOpcode opcode, dag outs, dag ins,
                string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatOther> {
  bits<5> rd;
  bits<5> rs1;
  bits<5> rs2;
  bits<5> rs3;
  bits<3> rm;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 89-106: TableGen class RVInstET5<bits<7> / TableGen 类 RVInstET5<bits<7>
```tablegen
  let Inst{31-27} = rs3;
  let Inst{26-25} = funct2;
  let Inst{24-20} = rs2;
  let Inst{19-15} = rs1;
  let Inst{14-12} = rm;
  let Inst{11-7} = rd;
  let Inst{6-0} = opcode.Value;
}

// Example: AIF.FMVZ.X.PS
class RVInstET5<bits<7> funct7, bits<3> funct3,
                RISCVOpcode opcode, dag outs, dag ins,
                string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatOther> {
  bits<5> rd;
  bits<5> rs1;
  bits<3> idx;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 107-123: TableGen class RVInstET6<bits<3> / TableGen 类 RVInstET6<bits<3>
```tablegen
  let Inst{31-25} = funct7;
  let Inst{24-23} = 0;
  let Inst{22-20} = idx;
  let Inst{19-15} = rs1;
  let Inst{14-12} = funct3;
  let Inst{11-7} = rd;
  let Inst{6-0} = opcode.Value;
}

// Example: AIF.FSLLI.PS
class RVInstET6<bits<3> funct3, RISCVOpcode opcode, dag outs, dag ins,
                string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatOther> {
  bits<5> imm;
  bits<5> rs1;
  bits<5> rd;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 124-139: TableGen class RVInstET7<RISCVOpcode / TableGen 类 RVInstET7<RISCVOpcode
```tablegen
  let Inst{31-25} = 0b0100111;
  let Inst{24-20} = imm;
  let Inst{19-15} = rs1;
  let Inst{14-12} = funct3;
  let Inst{11-7} = rd;
  let Inst{6-0} = opcode.Value;
}

// Example: AIF.FROUND.P
class RVInstET7<RISCVOpcode opcode, dag outs, dag ins,
                string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatOther> {
  bits<5> rs1;
  bits<3> rm;
  bits<5> rd;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 140-154: TableGen class RVInstET8<RISCVOpcode / TableGen 类 RVInstET8<RISCVOpcode
```tablegen
  let Inst{31-20} = 0b010110000001;
  let Inst{19-15} = rs1;
  let Inst{14-12} = rm;
  let Inst{11-7} = rd;
  let Inst{6-0} = opcode.Value;
}

// Example: AIF.MOV.M.X
class RVInstET8<RISCVOpcode opcode, dag outs, dag ins,
                string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatOther> {
  bits<8> imm;
  bits<5> rs1;
  bits<3> rd;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 155-171: TableGen class RVInstET9<bits<3> / TableGen 类 RVInstET9<bits<3>
```tablegen
  let Inst{31-25} = 0b0101011;
  let Inst{24-20} = imm{7-3};
  let Inst{19-15} = rs1;
  let Inst{14-12} = imm{2-0};
  let Inst{11-10} = 0b00;
  let Inst{9-7} = rd;
  let Inst{6-0} = opcode.Value;
}

// Example: AIF.MASKAND
class RVInstET9<bits<3> funct3, RISCVOpcode opcode, dag outs, dag ins,
                string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatOther> {
  bits<3> rs2;
  bits<3> rs1;
  bits<3> rd;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 172-181: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
  let Inst{31-25} = 0b0110011;
  let Inst{24-23} = 0b00;
  let Inst{22-20} = rs2;
  let Inst{19-18} = 0b00;
  let Inst{17-15} = rs1;
  let Inst{14-12} = funct3;
  let Inst{11-10} = 0b00;
  let Inst{9-7} = rd;
  let Inst{6-0} = opcode.Value;
}
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 182-197: TableGen class RVInstET10<bits<2> / TableGen 类 RVInstET10<bits<2>
```tablegen

// Example: AIF.MASKPOPC
class RVInstET10<bits<2> funct2, RISCVOpcode opcode, dag outs, dag ins,
                 string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatOther> {
  bits<3> rs1;
  bits<5> rd;

  let Inst{31-27} = 0b01010;
  let Inst{26-25} = funct2;
  let Inst{24-18} = 0b0000000;
  let Inst{17-15} = rs1;
  let Inst{14-12} = 0b000;
  let Inst{11-7} = rd;
  let Inst{6-0} = opcode.Value;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 198-207: TableGen class RVInstET11<RISCVOpcode / TableGen 类 RVInstET11<RISCVOpcode
```tablegen

// Example: AIF.MASKPOPC.RAST
class RVInstET11<RISCVOpcode opcode, dag outs, dag ins,
                 string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatOther> {
  bits<4> imm;
  bits<3> rs2;
  bits<3> rs1;
  bits<5> rd;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 208-217: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
  let Inst{31-25} = 0b0101111;
  let Inst{24-23} = imm{3-2};
  let Inst{22-20} = rs2;
  let Inst{19-18} = imm{1-0};
  let Inst{17-15} = rs1;
  let Inst{14-12} = 0b000;
  let Inst{11-7} = rd;
  let Inst{6-0} = opcode.Value;
}
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 218-233: TableGen class RVInstET12<bits<7> / TableGen 类 RVInstET12<bits<7>
```tablegen
// Example: AIF.FSCW.PS
class RVInstET12<bits<7> funct7, bits<3> funct3,
                 RISCVOpcode opcode, dag outs, dag ins,
                 string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatOther> {
  bits<5> rs1;
  bits<5> rs2;
  bits<5> rs3;

  let Inst{31-25} = funct7;
  let Inst{24-20} = rs2;
  let Inst{19-15} = rs1;
  let Inst{14-12} = funct3;
  let Inst{11-7} = rs3;
  let Inst{6-0} = opcode.Value;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 234-249: TableGen class RVInstET13<bits<7> / TableGen 类 RVInstET13<bits<7>
```tablegen

// Example: AIF.SBG
class RVInstET13<bits<7> funct7, bits<3> funct3,
                 RISCVOpcode opcode, dag outs, dag ins,
                 string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatOther> {
  bits<5> rs1;
  bits<5> rs2;

  let Inst{31-25} = funct7;
  let Inst{24-20} = rs2;
  let Inst{19-15} = rs1;
  let Inst{14-12} = funct3;
  let Inst{11-7} = 0b00000;
  let Inst{6-0} = opcode.Value;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 250-264: TableGen class RVInstET14<bits<7> / TableGen 类 RVInstET14<bits<7>
```tablegen

// Example: AIF.FSWL.PS
class RVInstET14<bits<7> funct7, bits<3> funct3, RISCVOpcode opcode, dag outs, dag ins,
                 string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatOther> {
  bits<5> fs3;
  bits<5> rs1;

  let Inst{31-25} = funct7;
  let Inst{24-20} = 0b00000;
  let Inst{19-15} = rs1;
  let Inst{14-12} = funct3;
  let Inst{11-7} = fs3;
  let Inst{6-0} = opcode.Value;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 265-265: Whitespace and block separation / 空白与代码分段
```tablegen

```
**EN:** This block mainly separates nearby logic and preserves the source layout for readability.

**CN:** 该区段主要用于分隔相邻逻辑，并保持源码布局的可读性。

## Key Concepts / 关键概念
- **Instruction encodings** / **指令编码**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
