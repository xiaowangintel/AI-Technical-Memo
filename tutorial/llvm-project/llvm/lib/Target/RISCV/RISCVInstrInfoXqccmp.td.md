# RISCVInstrInfoXqccmp.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoXqccmp.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Commentary and design intent / 注释与设计意图
```tablegen
//===---------------- RISCVInstrInfoXqccmp.td --------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes Qualcomm's Xqccmp extension.
//
// Xqccmp is broadly equivalent to (and incompatible with) Zcmp except the
// following changes:
//
// - The registers are pushed in the opposite order, so `ra` and `fp` are
//   closest to the incoming stack pointer (to be compatible with the
//   frame-pointer convention), and
//
// - There is a new `qc.cm.pushfp` instruction which is `qc.cm.push` but it sets
//   `fp` to the incoming stack pointer value, as expected by the frame-pointer
//   convention.
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 21-34: Commentary and design intent / 注释与设计意图
```tablegen
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// Operand and SDNode transformation definitions.
//===----------------------------------------------------------------------===//

def RegListS0AsmOperand : AsmOperandClass {
  let Name = "RegListS0";
  let ParserMethod = "parseRegListS0";
  let RenderMethod = "addRegListOperands";
  let DiagnosticType = "InvalidRegListS0";
  let DiagnosticString = "operand must be {ra, s0[-sN]} or {x1, x8[-x9][, x18[-xN]]}";
}
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 35-48: TableGen record reglist_s0 / TableGen 记录 reglist_s0
```tablegen

def reglist_s0 : RISCVOp<OtherVT> {
   let ParserMatchClass = RegListS0AsmOperand;
   let PrintMethod = "printRegList";
   let DecoderMethod = "decodeXqccmpRlistS0";
   let EncoderMethod = "getRlistS0OpValue";
   let MCOperandPredicate = [{
    int64_t Imm;
    if (!MCOp.evaluateAsConstantImm(Imm))
      return false;
    // 0~4 invalid for `qc.cm.pushfp`
    return isUInt<4>(Imm) && Imm >= RISCVZC::RA_S0;
  }];
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 49-58: Definitions and supporting logic / 定义与支撑逻辑
```tablegen
  string OperandType = "OPERAND_RLIST_S0";
}

//===----------------------------------------------------------------------===//
// Instruction Formats
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// Instruction Class Templates
//===----------------------------------------------------------------------===//
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 59-72: TableGen class RVInstXqccmpCPPPFP<bits<5> / TableGen 类 RVInstXqccmpCPPPFP<bits<5>
```tablegen

class RVInstXqccmpCPPPFP<bits<5> funct5, string opcodestr,
                         DAGOperand immtype = stackadj>
    : RVInst16<(outs), (ins reglist_s0:$rlist, immtype:$stackadj),
               opcodestr, "$rlist, $stackadj", [], InstFormatOther> {
  bits<4> rlist;
  bits<16> stackadj;

  let Inst{1-0} = 0b10;
  let Inst{3-2} = stackadj{5-4};
  let Inst{7-4} = rlist;
  let Inst{12-8} = funct5;
  let Inst{15-13} = 0b101;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 73-85: TableGen record QC_CM_MVA01S / TableGen 记录 QC_CM_MVA01S
```tablegen

//===----------------------------------------------------------------------===//
// Instructions
//===----------------------------------------------------------------------===//

let DecoderNamespace = "Xqccmp", Predicates = [HasVendorXqccmp] in {

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in {
let Defs = [X10, X11] in
def QC_CM_MVA01S : RVInst16CA<0b101011, 0b11, OPC_C2, (outs),
                              (ins SR07:$rs1, SR07:$rs2), "qc.cm.mva01s", "$rs1, $rs2">,
                   Sched<[WriteIALU, WriteIALU, ReadIALU, ReadIALU]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 86-98: TableGen record QC_CM_MVSA01 / TableGen 记录 QC_CM_MVSA01
```tablegen
let Uses = [X10, X11] in
def QC_CM_MVSA01 : RVInst16CA<0b101011, 0b01, OPC_C2, (outs SR07:$rs1, SR07:$rs2),
                              (ins), "qc.cm.mvsa01", "$rs1, $rs2">,
                   Sched<[WriteIALU, WriteIALU, ReadIALU, ReadIALU]>;
} // hasSideEffects = 0, mayLoad = 0, mayStore = 0

let hasSideEffects = 0, mayLoad = 0, mayStore = 1, Uses = [X2], Defs = [X2] in
def QC_CM_PUSH : RVInstZcCPPP<0b11000, "qc.cm.push", negstackadj>,
                 Sched<[WriteIALU, ReadIALU, ReadStoreData, ReadStoreData,
                        ReadStoreData, ReadStoreData, ReadStoreData, ReadStoreData,
                        ReadStoreData, ReadStoreData, ReadStoreData, ReadStoreData,
                        ReadStoreData, ReadStoreData, ReadStoreData]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 99-112: TableGen record QC_CM_PUSHFP / TableGen 记录 QC_CM_PUSHFP
```tablegen
let hasSideEffects = 0, mayLoad = 0, mayStore = 1, Uses = [X2], Defs = [X2, X8] in
def QC_CM_PUSHFP : RVInstXqccmpCPPPFP<0b11001, "qc.cm.pushfp", negstackadj>,
                   Sched<[WriteIALU, WriteIALU, ReadIALU, ReadStoreData, ReadStoreData,
                          ReadStoreData, ReadStoreData, ReadStoreData, ReadStoreData,
                          ReadStoreData, ReadStoreData, ReadStoreData, ReadStoreData,
                          ReadStoreData, ReadStoreData, ReadStoreData]>;

let hasSideEffects = 0, mayLoad = 1, mayStore = 0, isReturn = 1,
    Uses = [X2], Defs = [X2] in
def QC_CM_POPRET : RVInstZcCPPP<0b11110, "qc.cm.popret">,
                   Sched<[WriteIALU, WriteLDW, WriteLDW, WriteLDW, WriteLDW,
                          WriteLDW, WriteLDW, WriteLDW, WriteLDW, WriteLDW,
                          WriteLDW, WriteLDW, WriteLDW, WriteLDW, ReadIALU]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 113-127: TableGen record QC_CM_POPRETZ / TableGen 记录 QC_CM_POPRETZ
```tablegen
let hasSideEffects = 0, mayLoad = 1, mayStore = 0, isReturn = 1,
    Uses = [X2], Defs = [X2, X10] in
def QC_CM_POPRETZ : RVInstZcCPPP<0b11100, "qc.cm.popretz">,
                    Sched<[WriteIALU, WriteIALU, WriteLDW, WriteLDW, WriteLDW,
                           WriteLDW, WriteLDW, WriteLDW, WriteLDW, WriteLDW,
                           WriteLDW, WriteLDW, WriteLDW, WriteLDW, WriteLDW,
                           ReadIALU]>;

let hasSideEffects = 0, mayLoad = 1, mayStore = 0,
    Uses = [X2], Defs = [X2] in
def QC_CM_POP : RVInstZcCPPP<0b11010, "qc.cm.pop">,
                Sched<[WriteIALU, WriteLDW, WriteLDW, WriteLDW, WriteLDW,
                       WriteLDW, WriteLDW, WriteLDW, WriteLDW, WriteLDW, WriteLDW,
                       WriteLDW, WriteLDW, WriteLDW, ReadIALU]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 128-133: Definitions and supporting logic / 定义与支撑逻辑
```tablegen
} // DecoderNamespace = "Xqccmp", Predicates = [HasVendorXqccmp]

//===----------------------------------------------------------------------===//
// Aliases
//===----------------------------------------------------------------------===//
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
