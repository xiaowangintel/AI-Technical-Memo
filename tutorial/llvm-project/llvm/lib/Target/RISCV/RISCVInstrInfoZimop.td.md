# RISCVInstrInfoZimop.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoZimop.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoZimop.td -----------------------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the RISC-V instructions from the standard
// May-Be-Operations Extension (Zimop).
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 13-23: TableGen class RVInstIMopr<bits<7> / TableGen 类 RVInstIMopr<bits<7>
```tablegen

class RVInstIMopr<bits<7> imm7, bits<5> imm5, bits<3> funct3, RISCVOpcode opcode,
                   dag outs, dag ins, string opcodestr, string argstr>
    : RVInstIBase<funct3, opcode, outs, ins, opcodestr, argstr> {
  let Inst{31} = imm7{6};
  let Inst{30} = imm5{4};
  let Inst{29-28} = imm7{5-4};
  let Inst{27-26} = imm5{3-2};
  let Inst{25-22} = imm7{3-0};
  let Inst{21-20} = imm5{1-0};
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 24-33: TableGen class RVInstRMoprr<bits<4> / TableGen 类 RVInstRMoprr<bits<4>
```tablegen

class RVInstRMoprr<bits<4> imm4, bits<3> imm3, bits<3> funct3, RISCVOpcode opcode,
                   dag outs, dag ins, string opcodestr, string argstr>
    : RVInstRBase<funct3, opcode, outs, ins, opcodestr, argstr> {
  let Inst{31} = imm4{3};
  let Inst{30} = imm3{2};
  let Inst{29-28} = imm4{2-1};
  let Inst{27-26} = imm3{1-0};
  let Inst{25} = imm4{0};
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 34-43: TableGen record riscv_mop_r / TableGen 记录 riscv_mop_r
```tablegen

// May-Be-Operations
def riscv_mop_r  : RVSDNode<"MOP_R",
                            SDTypeProfile<1, 2, [SDTCisInt<0>, SDTCisSameAs<0, 1>,
                                                 SDTCisSameAs<0, 2>]>>;
def riscv_mop_rr : RVSDNode<"MOP_RR",
                            SDTypeProfile<1, 3, [SDTCisInt<0>, SDTCisSameAs<0, 1>,
                                                 SDTCisSameAs<0, 2>,
                                                 SDTCisSameAs<0, 3>]>>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 44-56: TableGen class RVMopr<bits<7> / TableGen 类 RVMopr<bits<7>
```tablegen
let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
class RVMopr<bits<7> imm7, bits<5> imm5, bits<3> funct3,
             RISCVOpcode opcode, string opcodestr>
    : RVInstIMopr<imm7, imm5, funct3, opcode, (outs GPR:$rd), (ins GPR:$rs1),
                   opcodestr, "$rd, $rs1">;

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
class RVMoprr<bits<4> imm4, bits<3> imm3, bits<3> funct3,
             RISCVOpcode opcode, string opcodestr>
    : RVInstRMoprr<imm4, imm3, funct3, opcode, (outs GPR:$rd),
                   (ins GPR:$rs1, GPR:$rs2),
                   opcodestr, "$rd, $rs1, $rs2">;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 57-67: TableGen record i / TableGen 记录 i
```tablegen
foreach i = 0...31 in {
  let Predicates = [HasStdExtZimop] in
  def MOP_R_#i : RVMopr<0b1000111, i, 0b100, OPC_SYSTEM, "mop.r."#i>,
                 Sched<[]>;
}

foreach i = 0...7 in {
  let Predicates = [HasStdExtZimop] in
  def MOP_RR_#i : RVMoprr<0b1001, i, 0b100, OPC_SYSTEM, "mop.rr."#i>,
                Sched<[]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 68-75: TableGen record i / TableGen 记录 i
```tablegen

let Predicates = [HasStdExtZimop] in {
// Zimop instructions
foreach i = 0...31 in {
  def : Pat<(XLenVT (riscv_mop_r GPR:$rs1, (XLenVT i))),
            (!cast<Instruction>("MOP_R_"#i) GPR:$rs1)>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 76-81: TableGen record i / TableGen 记录 i
```tablegen
foreach i = 0...7 in {
  def : Pat<(XLenVT (riscv_mop_rr GPR:$rs1, GPR:$rs2, (XLenVT i))),
            (!cast<Instruction>("MOP_RR_"#i) GPR:$rs1, GPR:$rs2)>;
}

} // Predicates = [HasStdExtZimop]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
