# RISCVInstrInfoM.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoM.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoM.td - RISC-V 'M' instructions -------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the RISC-V instructions from the standard 'M', Integer
// Multiplication and Division instruction set extension.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 13-27: TableGen record riscv_mulhsu / TableGen 记录 riscv_mulhsu
```tablegen

//===----------------------------------------------------------------------===//
// RISC-V specific DAG Nodes.
//===----------------------------------------------------------------------===//

// Multiply high for signedxunsigned.
def riscv_mulhsu : RVSDNode<"MULHSU", SDTIntBinOp>;

// 32-bit operations from RV64M that can't be simply matched with a pattern
// at instruction selection time. These have undefined behavior for division
// by 0 or overflow (divw) like their target independent counterparts.
def riscv_divw  : RVSDNode<"DIVW",  SDT_RISCVIntBinOpW>;
def riscv_divuw : RVSDNode<"DIVUW", SDT_RISCVIntBinOpW>;
def riscv_remuw : RVSDNode<"REMUW", SDT_RISCVIntBinOpW>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 28-42: TableGen record MUL / TableGen 记录 MUL
```tablegen
//===----------------------------------------------------------------------===//
// Instructions
//===----------------------------------------------------------------------===//

let Predicates = [HasStdExtZmmul] in {
def MUL     : ALU_rr<0b0000001, 0b000, "mul", Commutable=1>,
              Sched<[WriteIMul, ReadIMul, ReadIMul]>;
def MULH    : ALU_rr<0b0000001, 0b001, "mulh", Commutable=1>,
              Sched<[WriteIMul, ReadIMul, ReadIMul]>;
def MULHSU  : ALU_rr<0b0000001, 0b010, "mulhsu">,
              Sched<[WriteIMul, ReadIMul, ReadIMul]>;
def MULHU   : ALU_rr<0b0000001, 0b011, "mulhu", Commutable=1>,
              Sched<[WriteIMul, ReadIMul, ReadIMul]>;
} // Predicates = [HasStdExtZmmul]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 43-53: TableGen record DIV / TableGen 记录 DIV
```tablegen
let Predicates = [HasStdExtM] in {
def DIV     : ALU_rr<0b0000001, 0b100, "div">,
              Sched<[WriteIDiv, ReadIDiv, ReadIDiv]>;
def DIVU    : ALU_rr<0b0000001, 0b101, "divu">,
              Sched<[WriteIDiv, ReadIDiv, ReadIDiv]>;
def REM     : ALU_rr<0b0000001, 0b110, "rem">,
              Sched<[WriteIRem, ReadIRem, ReadIRem]>;
def REMU    : ALU_rr<0b0000001, 0b111, "remu">,
              Sched<[WriteIRem, ReadIRem, ReadIRem]>;
} // Predicates = [HasStdExtM]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 54-69: TableGen record MULW / TableGen 记录 MULW
```tablegen
let Predicates = [HasStdExtZmmul, IsRV64], IsSignExtendingOpW = 1 in {
def MULW    : ALUW_rr<0b0000001, 0b000, "mulw", Commutable=1>,
              Sched<[WriteIMul32, ReadIMul32, ReadIMul32]>;
} // Predicates = [HasStdExtZmmul, IsRV64]

let Predicates = [HasStdExtM, IsRV64], IsSignExtendingOpW = 1 in {
def DIVW    : ALUW_rr<0b0000001, 0b100, "divw">,
              Sched<[WriteIDiv32, ReadIDiv32, ReadIDiv32]>;
def DIVUW   : ALUW_rr<0b0000001, 0b101, "divuw">,
              Sched<[WriteIDiv32, ReadIDiv32, ReadIDiv32]>;
def REMW    : ALUW_rr<0b0000001, 0b110, "remw">,
              Sched<[WriteIRem32, ReadIRem32, ReadIRem32]>;
def REMUW   : ALUW_rr<0b0000001, 0b111, "remuw">,
              Sched<[WriteIRem32, ReadIRem32, ReadIRem32]>;
} // Predicates = [HasStdExtM, IsRV64]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 70-80: TableGen record definition / TableGen 记录定义
```tablegen
//===----------------------------------------------------------------------===//
// Pseudo-instructions and codegen patterns
//===----------------------------------------------------------------------===//

let Predicates = [HasStdExtZmmul] in {
def : PatGprGpr<mul, MUL>;
def : PatGprGpr<mulhs, MULH>;
def : PatGprGpr<mulhu, MULHU>;
def : PatGprGpr<riscv_mulhsu, MULHSU>;
} // Predicates = [HasStdExtZmmul]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 81-91: TableGen record definition / TableGen 记录定义
```tablegen
let Predicates = [HasStdExtM] in {
def : PatGprGpr<sdiv, DIV>;
def : PatGprGpr<udiv, DIVU>;
def : PatGprGpr<srem, REM>;
def : PatGprGpr<urem, REMU>;
} // Predicates = [HasStdExtM]

// Select W instructions if only the lower 32-bits of the result are used.
let Predicates = [HasStdExtZmmul, IsRV64] in
def : PatGprGpr<binop_allwusers<mul>, MULW>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 92-106: TableGen record definition / TableGen 记录定义
```tablegen
let Predicates = [HasStdExtM, IsRV64] in {
def : PatGprGpr<riscv_divw, DIVW>;
def : PatGprGpr<riscv_divuw, DIVUW>;
def : PatGprGpr<riscv_remuw, REMUW>;

// Handle the specific cases where using DIVU/REMU would be correct and result
// in fewer instructions than emitting DIVUW/REMUW then zero-extending the
// result.
def : Pat<(and (riscv_divuw (assertzexti32 GPR:$rs1),
                            (assertzexti32 GPR:$rs2)), 0xffffffff),
          (DIVU GPR:$rs1, GPR:$rs2)>;
def : Pat<(and (riscv_remuw (assertzexti32 GPR:$rs1),
                            (assertzexti32 GPR:$rs2)), 0xffffffff),
          (REMU GPR:$rs1, GPR:$rs2)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 107-121: TableGen record definition / TableGen 记录定义
```tablegen
// Although the sexti32 operands may not have originated from an i32 srem,
// this pattern is safe as it is impossible for two sign extended inputs to
// produce a result where res[63:32]=0 and res[31]=1.
def : Pat<(srem (sexti32 (i64 GPR:$rs1)), (sexti32 (i64 GPR:$rs2))),
          (REMW GPR:$rs1, GPR:$rs2)>;
} // Predicates = [HasStdExtM, IsRV64]

let Predicates = [HasStdExtZmmul, IsRV64, NoStdExtZba] in {
// Special case for calculating the full 64-bit product of a 32x32 unsigned
// multiply where the inputs aren't known to be zero extended. We can shift the
// inputs left by 32 and use a MULHU. This saves two SRLIs needed to finish
// zeroing the upper 32 bits.
def : Pat<(i64 (mul (and GPR:$rs1, 0xffffffff), (and GPR:$rs2, 0xffffffff))),
          (MULHU (i64 (SLLI GPR:$rs1, 32)), (i64 (SLLI GPR:$rs2, 32)))>;
} // Predicates = [HasStdExtZmmul, IsRV64, NoStdExtZba]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
