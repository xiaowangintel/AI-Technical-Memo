# RISCVInstrInfoZclsd.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoZclsd.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoZclsd.td -----------------------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the RISC-V instructions from the standard 'Zclsd',
// Compressed Load/Store pair instructions extension.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 13-23: TableGen record GPRPairNoX0RV32Operand / TableGen 记录 GPRPairNoX0RV32Operand
```tablegen

//===----------------------------------------------------------------------===//
// Instruction Class Templates
//===----------------------------------------------------------------------===//

def GPRPairNoX0RV32Operand : AsmOperandClass {
  let Name = "GPRPairNoX0RV32";
  let ParserMethod = "parseGPRPair<false>";
  let PredicateMethod = "isGPRPairNoX0";
  let RenderMethod = "addRegOperands";
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 24-34: TableGen record GPRPairNoX0RV32 / TableGen 记录 GPRPairNoX0RV32
```tablegen

def GPRPairNoX0RV32 : RegisterOperand<GPRPairNoX0> {
  let ParserMatchClass = GPRPairNoX0RV32Operand;
}

def GPRPairCRV32Operand : AsmOperandClass {
  let Name = "GPRPairCRV32";
  let ParserMethod = "parseGPRPair<false>";
  let PredicateMethod = "isGPRPairC";
  let RenderMethod = "addRegOperands";
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 35-42: TableGen record GPRPairCRV32 / TableGen 记录 GPRPairCRV32
```tablegen

def GPRPairCRV32 : RegisterOperand<GPRPairC> {
  let ParserMatchClass = GPRPairCRV32Operand;
}

//===----------------------------------------------------------------------===//
// Instructions
//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 43-53: TableGen record C_LDSP_RV32 / TableGen 记录 C_LDSP_RV32
```tablegen

let Predicates = [HasStdExtZclsd, IsRV32], DecoderNamespace = "ZcOverlap" in {
def C_LDSP_RV32 : CStackLoad<0b011, "c.ldsp", GPRPairNoX0RV32, uimm9_lsb000>,
                  Sched<[WriteLDD, ReadMemBase]> {
  let Inst{4-2} = imm{8-6};
}

def C_SDSP_RV32 : CStackStore<0b111, "c.sdsp", GPRPairRV32, uimm9_lsb000>,
                  Sched<[WriteSTD, ReadStoreData, ReadMemBase]> {
  let Inst{9-7} = imm{8-6};
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 54-61: TableGen record C_LD_RV32 / TableGen 记录 C_LD_RV32
```tablegen

def C_LD_RV32 : CLoad_ri<0b011, "c.ld", GPRPairCRV32, uimm8_lsb000>,
                Sched<[WriteLDD, ReadMemBase]> {
  bits<8> imm;
  let Inst{12-10} = imm{5-3};
  let Inst{6-5} = imm{7-6};
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 62-69: TableGen record C_SD_RV32 / TableGen 记录 C_SD_RV32
```tablegen
def C_SD_RV32 : CStore_rri<0b111, "c.sd", GPRPairCRV32, uimm8_lsb000>,
                Sched<[WriteSTD, ReadStoreData, ReadMemBase]> {
  bits<8> imm;
  let Inst{12-10} = imm{5-3};
  let Inst{6-5} = imm{7-6};
}
}// Predicates = [HasStdExtZclsd, IsRV32], DecoderNamespace = "ZcOverlap"
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 70-83: TableGen record definition / TableGen 记录定义
```tablegen
//===----------------------------------------------------------------------===//
// Assembler Pseudo Instructions
//===----------------------------------------------------------------------===//

let Predicates = [HasStdExtZclsd, IsRV32] in {
def : InstAlias<"c.ld $rd, (${rs1})",
                (C_LD_RV32 GPRPairCRV32:$rd, GPRCMem:$rs1, 0), 0>;
def : InstAlias<"c.sd $rs2, (${rs1})",
                (C_SD_RV32 GPRPairCRV32:$rs2, GPRCMem:$rs1, 0), 0>;
def : InstAlias<"c.ldsp $rd, (${rs1})",
                (C_LDSP_RV32 GPRPairNoX0RV32:$rd, SPMem:$rs1, 0), 0>;
def : InstAlias<"c.sdsp $rs2, (${rs1})",
                (C_SDSP_RV32 GPRPairRV32:$rs2, SPMem:$rs1, 0), 0>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 84-98: TableGen record definition / TableGen 记录定义
```tablegen

//===----------------------------------------------------------------------===//
// Compress Instruction tablegen backend.
//===----------------------------------------------------------------------===//

let Predicates = [HasStdExtZclsd, IsRV32] in {
def : CompressPat<(LD_RV32 GPRPairNoX0RV32:$rd, SPMem:$rs1, uimm9_lsb000:$imm),
                  (C_LDSP_RV32 GPRPairNoX0RV32:$rd, SPMem:$rs1, uimm9_lsb000:$imm)>;
def : CompressPat<(SD_RV32 GPRPairRV32:$rs2, SPMem:$rs1, uimm9_lsb000:$imm),
                  (C_SDSP_RV32 GPRPairRV32:$rs2, SPMem:$rs1, uimm9_lsb000:$imm)>;
def : CompressPat<(LD_RV32 GPRPairCRV32:$rd, GPRCMem:$rs1, uimm8_lsb000:$imm),
                  (C_LD_RV32 GPRPairCRV32:$rd, GPRCMem:$rs1, uimm8_lsb000:$imm)>;
def : CompressPat<(SD_RV32 GPRPairCRV32:$rs2, GPRCMem:$rs1, uimm8_lsb000:$imm),
                  (C_SD_RV32 GPRPairCRV32:$rs2, GPRCMem:$rs1, uimm8_lsb000:$imm)>;
} // Predicates = [HasStdExtZclsd, IsRV32]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
