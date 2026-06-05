# RISCVInstrInfoXwch.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoXwch.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoXwch.td ------------------------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the vendor extension(s) defined by WCH.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 12-22: TableGen class QKStackInst<bits<2> / TableGen 类 QKStackInst<bits<2>
```tablegen

class QKStackInst<bits<2> funct2, dag outs, dag ins,
                  string opcodestr, string argstr>
    : RVInst16<outs, ins, opcodestr, argstr, [], InstFormatOther> {
  bits<3> rd_rs2;

  let Inst{15-11} = 0b10000;
  let Inst{6-5} = funct2;
  let Inst{4-2} = rd_rs2;
  let Inst{1-0} = 0b00;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 23-33: TableGen record uimm5_lsb0 / TableGen 记录 uimm5_lsb0
```tablegen

//===----------------------------------------------------------------------===//
// Operand definitions.
//===----------------------------------------------------------------------===//

// A 5-bit unsigned immediate where the least significant bit is zero.
def uimm5_lsb0 : RISCVUImmLsbZeroOp<5, 1>;

// A 6-bit unsigned immediate where the least significant bit is zero.
def uimm6_lsb0 : RISCVUImmLsbZeroOp<6, 1>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 34-48: TableGen record QK_C_LBU / TableGen 记录 QK_C_LBU
```tablegen
//===----------------------------------------------------------------------===//
// Instructions
//===----------------------------------------------------------------------===//
let Predicates = [HasVendorXwchc], DecoderNamespace = "Xwchc" in {

let hasSideEffects = 0, mayLoad = 1, mayStore = 0 in
def QK_C_LBU : RVInst16CL<0b001, OPC_C0, (outs GPRC:$rd),
                          (ins GPRCMem:$rs1, uimm5:$imm),
                          "qk.c.lbu", "$rd, ${imm}(${rs1})">,
               Sched<[WriteLDB, ReadMemBase]> {
  bits<5> imm;
  let Inst{12} = imm{0};
  let Inst{11-10} = imm{4-3};
  let Inst{6-5} = imm{2-1};
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 49-59: TableGen record QK_C_SB / TableGen 记录 QK_C_SB
```tablegen
let hasSideEffects = 0, mayLoad = 0, mayStore = 1 in
def QK_C_SB : RVInst16CS<0b101, OPC_C0, (outs),
                         (ins GPRC:$rs2, GPRCMem:$rs1,
                              uimm5:$imm),
                         "qk.c.sb", "$rs2, ${imm}(${rs1})">,
              Sched<[WriteSTB, ReadStoreData, ReadMemBase]> {
  bits<5> imm;
  let Inst{12} = imm{0};
  let Inst{11-10} = imm{4-3};
  let Inst{6-5} = imm{2-1};
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 60-69: TableGen record QK_C_LHU / TableGen 记录 QK_C_LHU
```tablegen

let hasSideEffects = 0, mayLoad = 1, mayStore = 0 in
def QK_C_LHU : RVInst16CL<0b001, OPC_C2, (outs GPRC:$rd),
                          (ins GPRCMem:$rs1, uimm6_lsb0:$imm),
                          "qk.c.lhu", "$rd, ${imm}(${rs1})">,
               Sched<[WriteLDH, ReadMemBase]> {
  bits<6> imm;
  let Inst{12-10} = imm{5-3};
  let Inst{6-5} = imm{2-1};
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 70-79: TableGen record QK_C_SH / TableGen 记录 QK_C_SH
```tablegen
let hasSideEffects = 0, mayLoad = 0, mayStore = 1 in
def QK_C_SH : RVInst16CS<0b101, OPC_C2, (outs),
                         (ins GPRC:$rs2, GPRCMem:$rs1, uimm6_lsb0:$imm),
                         "qk.c.sh", "$rs2, ${imm}(${rs1})">,
              Sched<[WriteSTH, ReadStoreData, ReadMemBase]> {
  bits<6> imm;
  let Inst{12-10} = imm{5-3};
  let Inst{6-5} = imm{2-1};
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 80-98: TableGen record QK_C_LBUSP / TableGen 记录 QK_C_LBUSP
```tablegen
let hasSideEffects = 0, mayLoad = 1, mayStore = 0 in
def QK_C_LBUSP : QKStackInst<0b00, (outs GPRC:$rd_rs2),
                             (ins SPMem:$rs1, uimm4:$imm),
                             "qk.c.lbusp", "$rd_rs2, ${imm}(${rs1})">,
                 Sched<[WriteLDB, ReadMemBase]> {
  bits<0> rs1;
  bits<4> imm;
  let Inst{10-7} = imm;
}
let hasSideEffects = 0, mayLoad = 0, mayStore = 1 in
def QK_C_SBSP : QKStackInst<0b10, (outs),
                            (ins GPRC:$rd_rs2, SPMem:$rs1,
                                 uimm4:$imm),
                            "qk.c.sbsp", "$rd_rs2, ${imm}(${rs1})">,
                Sched<[WriteSTB, ReadStoreData, ReadMemBase]> {
  bits<0> rs1;
  bits<4> imm;
  let Inst{10-7} = imm;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 99-109: TableGen record QK_C_LHUSP / TableGen 记录 QK_C_LHUSP
```tablegen

let hasSideEffects = 0, mayLoad = 1, mayStore = 0 in
def QK_C_LHUSP : QKStackInst<0b01, (outs GPRC:$rd_rs2),
                             (ins SPMem:$rs1, uimm5_lsb0:$imm),
                             "qk.c.lhusp", "$rd_rs2, ${imm}(${rs1})">,
                 Sched<[WriteLDH, ReadMemBase]> {
  bits<0> rs1;
  bits<5> imm;
  let Inst{10-8} = imm{3-1};
  let Inst{7} = imm{4};
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 110-119: TableGen record QK_C_SHSP / TableGen 记录 QK_C_SHSP
```tablegen
let hasSideEffects = 0, mayLoad = 0, mayStore = 1 in
def QK_C_SHSP : QKStackInst<0b11, (outs),
                            (ins GPRC:$rd_rs2, SPMem:$rs1, uimm5_lsb0:$imm),
                            "qk.c.shsp", "$rd_rs2, ${imm}(${rs1})">,
                Sched<[WriteSTH, ReadStoreData, ReadMemBase]> {
  bits<0> rs1;
  bits<5> imm;
  let Inst{10-8} = imm{3-1};
  let Inst{7} = imm{4};
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 120-137: TableGen record definition / TableGen 记录定义
```tablegen

} // Predicates = [HasVendorXwchc], DecoderNamespace = "Xwchc"

//===----------------------------------------------------------------------===//
// Assembler Pseudo Instructions
//===----------------------------------------------------------------------===//

let EmitPriority = 0 in {
let Predicates = [HasVendorXwchc] in {
def : InstAlias<"qk.c.lbu $rd, (${rs1})", (QK_C_LBU GPRC:$rd, GPRCMem:$rs1, 0)>;
def : InstAlias<"qk.c.sb $rs2, (${rs1})", (QK_C_SB GPRC:$rs2, GPRCMem:$rs1, 0)>;
def : InstAlias<"qk.c.lhu $rd, (${rs1})", (QK_C_LHU GPRC:$rd, GPRCMem:$rs1, 0)>;
def : InstAlias<"qk.c.sh $rs2, (${rs1})", (QK_C_SH GPRC:$rs2, GPRCMem:$rs1, 0)>;
def : InstAlias<"qk.c.lbusp $rd, (${rs1})", (QK_C_LBUSP GPRC:$rd, SPMem:$rs1, 0)>;
def : InstAlias<"qk.c.sbsp $rs2, (${rs1})", (QK_C_SBSP GPRC:$rs2, SPMem:$rs1, 0)>;
def : InstAlias<"qk.c.lhusp $rd, (${rs1})", (QK_C_LHUSP GPRC:$rd, SPMem:$rs1, 0)>;
def : InstAlias<"qk.c.shsp $rs2, (${rs1})", (QK_C_SHSP GPRC:$rs2, SPMem:$rs1, 0)>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 138-157: TableGen record definition / TableGen 记录定义
```tablegen
}

//===----------------------------------------------------------------------===/
// Compress Instruction tablegen backend.
//===----------------------------------------------------------------------===//

let Predicates = [HasVendorXwchc] in {
def : CompressPat<(LBU GPRC:$rd, GPRCMem:$rs1, uimm5:$imm),
                  (QK_C_LBU GPRC:$rd, GPRCMem:$rs1, uimm5:$imm)>;
def : CompressPat<(SB GPRC:$rs2, GPRCMem:$rs1, uimm5:$imm),
                  (QK_C_SB GPRC:$rs2, GPRCMem:$rs1, uimm5:$imm)>;
def : CompressPat<(LHU GPRC:$rd, GPRCMem:$rs1, uimm6_lsb0:$imm),
                  (QK_C_LHU GPRC:$rd, GPRCMem:$rs1, uimm6_lsb0:$imm)>;
def : CompressPat<(SH GPRC:$rs2, GPRCMem:$rs1, uimm6_lsb0:$imm),
                  (QK_C_SH GPRC:$rs2, GPRCMem:$rs1, uimm6_lsb0:$imm)>;
def : CompressPat<(LBU GPRC:$rd, SPMem:$rs1,   uimm4:$imm),
                  (QK_C_LBUSP GPRC:$rd, SPMem:$rs1, uimm4:$imm)>;
def : CompressPat<(SB GPRC:$rs2, SPMem:$rs1,   uimm4:$imm),
                  (QK_C_SBSP GPRC:$rs2, SPMem:$rs1, uimm4:$imm)>;
def : CompressPat<(LHU GPRC:$rd, SPMem:$rs1,   uimm5_lsb0:$imm),
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 158-161: TableGen record definition / TableGen 记录定义
```tablegen
                  (QK_C_LHUSP GPRC:$rd, SPMem:$rs1, uimm5_lsb0:$imm)>;
def : CompressPat<(SH GPRC:$rs2, SPMem:$rs1,   uimm5_lsb0:$imm),
                  (QK_C_SHSP GPRC:$rs2, SPMem:$rs1, uimm5_lsb0:$imm)>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
