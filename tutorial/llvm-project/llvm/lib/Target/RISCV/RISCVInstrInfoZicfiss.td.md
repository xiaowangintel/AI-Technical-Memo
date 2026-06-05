# RISCVInstrInfoZicfiss.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoZicfiss.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Commentary and design intent / 注释与设计意图
```tablegen
//===------ RISCVInstrInfoZicfiss.td - RISC-V Zicfiss -*- tablegen -*------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 9-21: TableGen class templates / TableGen 类 templates
```tablegen
//===----------------------------------------------------------------------===//
// Instruction class templates
//===----------------------------------------------------------------------===//

class RVC_SSInst<bits<5> rs1val, RegisterClass reg_class, string opcodestr> :
  RVInst16<(outs), (ins reg_class:$rs1), opcodestr, "$rs1", [], InstFormatOther> {
  bits<0> rs1;
  let Inst{15-13} = 0b011;
  let Inst{12} = 0;
  let Inst{11-7} = rs1val;
  let Inst{6-2} = 0b00000;
  let Inst{1-0} = 0b01;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 22-37: Commentary and design intent / 注释与设计意图
```tablegen

//===----------------------------------------------------------------------===//
// Instructions
//===----------------------------------------------------------------------===//

// Zicfiss instructions that use Zimop encoding space are available when Zimop
// is enabled, without requiring Zicfiss explicitly. Per psABI decision
// (riscv-non-isa/riscv-elf-psabi-doc#474).
let Predicates = [HasStdExtZimop] in {
let Uses = [SSP], Defs = [SSP], hasSideEffects = 0, mayLoad = 1, mayStore = 0 in
def SSPOPCHK : RVInstI<0b100, OPC_SYSTEM, (outs), (ins GPRX1X5:$rs1), "sspopchk",
                       "$rs1"> {
  let rd = 0;
  let imm12 = 0b110011011100;
} // Uses = [SSP],  Defs = [SSP], hasSideEffects = 0, mayLoad = 1, mayStore = 0
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 38-50: TableGen record SSRDP / TableGen 记录 SSRDP
```tablegen
let Uses = [SSP], hasSideEffects = 0, mayLoad = 0, mayStore = 0 in {
def SSRDP : RVInstI<0b100, OPC_SYSTEM, (outs GPRNoX0:$rd), (ins), "ssrdp", "$rd"> {
  let imm12 = 0b110011011100;
  let rs1 = 0b00000;
}
} // Uses = [SSP], hasSideEffects = 0, mayLoad = 0, mayStore = 0

let Uses = [SSP], Defs = [SSP], hasSideEffects = 0, mayLoad = 0, mayStore = 1 in
def SSPUSH : RVInstR<0b1100111, 0b100, OPC_SYSTEM, (outs), (ins GPRX1X5:$rs2),
                     "sspush", "$rs2"> {
  let rd = 0b00000;
  let rs1 = 0b00000;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 51-60: TableGen record C_SSPUSH / TableGen 记录 C_SSPUSH
```tablegen
} // Predicates = [HasStdExtZimop]

// Compressed Zicfiss instructions use Zcmop encoding space and are available
// when Zcmop is enabled, without requiring Zicfiss explicitly. Per psABI
// decision (riscv-non-isa/riscv-elf-psabi-doc#474).
let Predicates = [HasStdExtZcmop],
    DecoderNamespace = "Zicfiss" in {
let Uses = [SSP], Defs = [SSP], hasSideEffects = 0, mayLoad = 0, mayStore = 1 in
def C_SSPUSH : RVC_SSInst<0b00001, GPRX1, "c.sspush">;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 61-70: Bulk record instantiation C_SSPOPCHK / 批量记录实例化 C_SSPOPCHK
```tablegen
let Uses = [SSP], Defs = [SSP], hasSideEffects = 0, mayLoad = 1, mayStore = 0 in
def C_SSPOPCHK : RVC_SSInst<0b00101, GPRX5, "c.sspopchk">;
} // Predicates = [HasStdExtZcmop]

let Predicates = [HasStdExtZicfiss] in
defm SSAMOSWAP_W  : AMO_rr_aq_rl<0b01001, 0b010, "ssamoswap.w">;

let Predicates = [HasStdExtZicfiss, IsRV64] in
defm SSAMOSWAP_D  : AMO_rr_aq_rl<0b01001, 0b011, "ssamoswap.d">;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 71-79: TableGen record PseudoMOP_SSPUSH / TableGen 记录 PseudoMOP_SSPUSH
```tablegen
let Predicates = [HasStdExtZimop] in {
let hasSideEffects = 1, mayLoad = 0, mayStore = 1 in
def PseudoMOP_SSPUSH : Pseudo<(outs), (ins GPRX1X5:$rs2), []>,
    PseudoInstExpansion<(MOP_RR_7 X0, X0, GPR:$rs2)>;
let hasSideEffects = 1, mayLoad = 1, mayStore = 0 in
def PseudoMOP_SSPOPCHK : Pseudo<(outs), (ins GPRX1X5:$rs1), []>,
    PseudoInstExpansion<(MOP_R_28 X0, GPR:$rs1)>;
} // Predicates = [HasStdExtZimop]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 80-88: TableGen record PseudoMOP_C_SSPUSH / TableGen 记录 PseudoMOP_C_SSPUSH
```tablegen
let Predicates = [HasStdExtZcmop] in {
let Uses = [X1], hasSideEffects = 1, mayLoad = 0, mayStore = 1 in
def PseudoMOP_C_SSPUSH : Pseudo<(outs), (ins), []>,
    PseudoInstExpansion<(C_SSPUSH X1)>;
} // Predicates = [HasStdExtZcmop]

//===----------------------------------------------------------------------===/
// Compress Instruction tablegen backend.
//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 89-100: TableGen record definition / TableGen 记录定义
```tablegen

let Predicates = [HasStdExtZcmop] in {
def : CompressPat<(SSPUSH X1), (C_SSPUSH X1)>;
def : CompressPat<(SSPOPCHK X5), (C_SSPOPCHK X5)>;
} // Predicates = [HasStdExtZcmop]

// c.mop.1 and c.mop.5 are aliases for c.sspush ra and c.sspopchk t0.
// Use EmitPriority=0 so disassembler prints c.sspush/c.sspopchk.
let Predicates = [HasStdExtZcmop], EmitPriority = 0 in {
def : InstAlias<"c.mop.1", (C_SSPUSH X1)>;
def : InstAlias<"c.mop.5", (C_SSPOPCHK X5)>;
} // Predicates = [HasStdExtZcmop], EmitPriority = 0
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
