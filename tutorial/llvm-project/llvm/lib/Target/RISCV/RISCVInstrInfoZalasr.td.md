# RISCVInstrInfoZalasr.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoZalasr.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoZalasr.td  ---------------------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the RISC-V instructions from the Zalasr (Load-Acquire
// and Store-Release) extension
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 13-24: TableGen class templates / TableGen 类 templates
```tablegen

//===----------------------------------------------------------------------===//
// Instruction class templates
//===----------------------------------------------------------------------===//

let hasSideEffects = 0, mayLoad = 1, mayStore = 0 in
class LAQ_r<bit aq, bit rl, bits<3> funct3, string opcodestr>
    : RVInstRAtomic<0b00110, aq, rl, funct3, OPC_AMO,
                    (outs GPR:$rd), (ins GPRMemZeroOffset:$rs1),
                    opcodestr, "$rd, $rs1"> {
  let rs2 = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 25-32: TableGen class SRL_r<bit / TableGen 类 SRL_r<bit
```tablegen

let hasSideEffects = 0, mayLoad = 0, mayStore = 1 in
class SRL_r<bit aq, bit rl, bits<3> funct3, string opcodestr>
    : RVInstRAtomic<0b00111, aq, rl, funct3, OPC_AMO,
                    (outs), (ins GPR:$rs2, GPRMemZeroOffset:$rs1),
                    opcodestr, "$rs2, $rs1"> {
  let rd = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 33-42: Reusable TableGen multiclass LAQ_r_aq_rl / 可复用的 TableGen 多类 LAQ_r_aq_rl
```tablegen

multiclass LAQ_r_aq_rl<bits<3> funct3, string opcodestr> {
  def _AQ   : LAQ_r<1, 0, funct3, opcodestr # ".aq">;
  def _AQRL : LAQ_r<1, 1, funct3, opcodestr # ".aqrl">;
}

multiclass SRL_r_aq_rl<bits<3> funct3, string opcodestr> {
  def _RL   : SRL_r<0, 1, funct3, opcodestr # ".rl">;
  def _AQRL : SRL_r<1, 1, funct3, opcodestr # ".aqrl">;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 43-56: Bulk record instantiation LB / 批量记录实例化 LB
```tablegen

//===----------------------------------------------------------------------===//
// Instructions
//===----------------------------------------------------------------------===//

let Predicates = [HasStdExtZalasr], IsSignExtendingOpW = 1 in {
defm LB : LAQ_r_aq_rl<0b000, "lb">;
defm LH : LAQ_r_aq_rl<0b001, "lh">;
defm LW : LAQ_r_aq_rl<0b010, "lw">;
defm SB : SRL_r_aq_rl<0b000, "sb">;
defm SH : SRL_r_aq_rl<0b001, "sh">;
defm SW : SRL_r_aq_rl<0b010, "sw">;
} // Predicates = [HasStdExtZalasr]
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 57-64: Bulk record instantiation LD / 批量记录实例化 LD
```tablegen
let Predicates = [HasStdExtZalasr, IsRV64] in {
defm LD : LAQ_r_aq_rl<0b011, "ld">;
defm SD : SRL_r_aq_rl<0b011, "sd">;
} // Predicates = [HasStdExtZalasr, IsRV64]

//===----------------------------------------------------------------------===//
// Pseudo-instructions and codegen patterns
//===----------------------------------------------------------------------===//
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 65-76: TableGen class PatLAQ<SDPatternOperator / TableGen 类 PatLAQ<SDPatternOperator
```tablegen

class PatLAQ<SDPatternOperator OpNode, RVInst Inst, ValueType vt = XLenVT>
    : Pat<(vt (OpNode (XLenVT GPRMemZeroOffset:$rs1))),
          (Inst GPRMemZeroOffset:$rs1)>;

// n.b. this switches order of arguments
//  to deal with the fact that SRL has addr, data
//  while atomic_store has data, addr
class PatSRL<SDPatternOperator OpNode, RVInst Inst, ValueType vt = XLenVT>
    : Pat<(OpNode (vt GPR:$rs2), (XLenVT GPRMemZeroOffset:$rs1)),
          (Inst GPR:$rs2, GPRMemZeroOffset:$rs1)>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 77-86: TableGen record definition / TableGen 记录定义
```tablegen

let Predicates = [HasStdExtZalasr] in {
  // the sequentially consistent loads use
  //  .aq instead of .aqrl to match the psABI/A.7
  def : PatLAQ<acquiring_load<atomic_load_asext_8>, LB_AQ>;
  def : PatLAQ<seq_cst_load<atomic_load_asext_8>, LB_AQ>;

  def : PatLAQ<acquiring_load<atomic_load_asext_16>, LH_AQ>;
  def : PatLAQ<seq_cst_load<atomic_load_asext_16>, LH_AQ>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 87-94: TableGen record definition / TableGen 记录定义
```tablegen
  // the sequentially consistent stores use
  //  .rl instead of .aqrl to match the psABI/A.7
  def : PatSRL<releasing_store<atomic_store_8>, SB_RL>;
  def : PatSRL<seq_cst_store<atomic_store_8>, SB_RL>;

  def : PatSRL<releasing_store<atomic_store_16>, SH_RL>;
  def : PatSRL<seq_cst_store<atomic_store_16>, SH_RL>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 95-103: TableGen record definition / TableGen 记录定义
```tablegen
  def : PatSRL<releasing_store<atomic_store_32>, SW_RL>;
  def : PatSRL<seq_cst_store<atomic_store_32>, SW_RL>;
}

let Predicates = [HasStdExtZalasr, IsRV32] in {
  def : PatLAQ<acquiring_load<atomic_load_nonext_32>, LW_AQ, i32>;
  def : PatLAQ<seq_cst_load<atomic_load_nonext_32>, LW_AQ, i32>;
} // Predicates = [HasStdExtZalasr, IsRV32]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 104-113: TableGen record definition / TableGen 记录定义
```tablegen
let Predicates = [HasStdExtZalasr, IsRV64] in {
  def : PatLAQ<acquiring_load<atomic_load_asext_32>, LW_AQ, i64>;
  def : PatLAQ<seq_cst_load<atomic_load_asext_32>, LW_AQ, i64>;

  def : PatLAQ<acquiring_load<atomic_load_nonext_64>, LD_AQ, i64>;
  def : PatLAQ<seq_cst_load<atomic_load_nonext_64>, LD_AQ, i64>;

  def : PatSRL<releasing_store<atomic_store_64>, SD_RL, i64>;
  def : PatSRL<seq_cst_store<atomic_store_64>, SD_RL, i64>;
} // Predicates = [HasStdExtZalasr, IsRV64]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
