# RISCVInstrInfoSFB.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoSFB.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoSFB.td - Pseudos for SFB -------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the pseudos for SFB (Short Forward Branch).
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 12-21: TableGen record bcc_opcode / TableGen 记录 bcc_opcode
```tablegen

// Represents the Opcode for a branch instruction, used by SFB pseudos
def bcc_opcode : RISCVOp<i32> {
  let OperandType = "OPERAND_BCC_OPCODE";
}

def sfb_rhs : RegisterOperand<GPR> {
  let OperandNamespace = "RISCVOp";
  let OperandType = "OPERAND_SFB_RHS";
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 22-31: TableGen record CCtoRISCVBCC / TableGen 记录 CCtoRISCVBCC
```tablegen

// cond -> bcc_opcode for reg, reg branches
def CCtoRISCVBCC : SDNodeXForm<cond, [{
  ISD::CondCode CC = N->get();
  RISCVCC::CondCode RvCC = getRISCVCCForIntCC(CC);
  RvCC = RISCVCC::getInverseBranchCondition(RvCC);
  unsigned BccOpcode = RISCVCC::getBrCond(RvCC);
  return CurDAG->getTargetConstant(BccOpcode, SDLoc(N), MVT::i32);
}]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 32-47: TableGen record CCtoQC_E_BccI / TableGen 记录 CCtoQC_E_BccI
```tablegen
// cond -> bcc_opcode (QC_E_BccI)
def CCtoQC_E_BccI : SDNodeXForm<cond, [{
  ISD::CondCode CC = N->get();
  RISCVCC::CondCode RvCC = getRISCVCCForIntCC(CC);
  RvCC = RISCVCC::getInverseBranchCondition(RvCC);
  unsigned SelectOpc;
  switch (RvCC) {
  case RISCVCC::COND_LTU:
  case RISCVCC::COND_GEU:
    SelectOpc = RISCV::Select_GPRNoX0_Using_CC_UImm16NonZero_QC;
    break;
  default:
    // EQ/NE/LT/GE treated as signed condition codes
    SelectOpc = RISCV::Select_GPRNoX0_Using_CC_SImm16NonZero_QC;
    break;
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 48-67: TableGen record CCtoQC_BccI / TableGen 记录 CCtoQC_BccI
```tablegen
  unsigned BccOpcode = RISCVCC::getBrCond(RvCC, SelectOpc);
  return CurDAG->getTargetConstant(BccOpcode, SDLoc(N), MVT::i32);
}]>;

// cond -> bcc_opcode (QC_BeqI)
def CCtoQC_BccI : SDNodeXForm<cond, [{
  ISD::CondCode CC = N->get();
  RISCVCC::CondCode RvCC = getRISCVCCForIntCC(CC);
  RvCC = RISCVCC::getInverseBranchCondition(RvCC);
  unsigned SelectOpc;
  switch (RvCC) {
  case RISCVCC::COND_LTU:
  case RISCVCC::COND_GEU:
    SelectOpc = RISCV::Select_GPRNoX0_Using_CC_UImm5NonZero_QC;
    break;
  default:
    // EQ/NE/LT/GE treated as signed condition codes
    SelectOpc = RISCV::Select_GPRNoX0_Using_CC_SImm5NonZero_QC;
    break;
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 68-84: TableGen record PseudoCCMOVGPR / TableGen 记录 PseudoCCMOVGPR
```tablegen
  unsigned BccOpcode = RISCVCC::getBrCond(RvCC, SelectOpc);
  return CurDAG->getTargetConstant(BccOpcode, SDLoc(N), MVT::i32);
}]>;

// For each of the short forward branch pseudos, corresponding code for
// getting correct size of the pseduo is needed in getInstSizeInBytes.
let Predicates = [HasShortForwardBranchIALU], isSelect = 1,
    Constraints = "$dst = $falsev", isCommutable = 1 in {
// This instruction moves $truev to $dst when the condition is true. It will
// be expanded to control flow in RISCVExpandPseudoInsts.
def PseudoCCMOVGPR : Pseudo<(outs GPR:$dst),
                            (ins GPR:$falsev, GPR:$truev, bcc_opcode:$bcc,
                             GPR:$lhs, sfb_rhs:$rhs),
                            []>,
                     Sched<[WriteSFB, ReadSFBALU, ReadSFBALU,
                            ReadSFBJmp, ReadSFBJmp]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 85-102: TableGen record PseudoCCMOVGPRNoX0 / TableGen 记录 PseudoCCMOVGPRNoX0
```tablegen

let Predicates = [HasShortForwardBranchIALU] in
def : Pat<(riscv_selectcc (XLenVT GPR:$lhs), GPR:$rhs, cond:$cc, (XLenVT GPR:$truev), GPR:$falsev),
           (PseudoCCMOVGPR GPR:$falsev, GPR:$truev, (CCtoRISCVBCC $cc), GPR:$lhs, GPR:$rhs)>;

// This should always expand to a branch+c.mv so the size is 6 or 4 if the
// branch is compressible.
let Predicates = [HasConditionalMoveFusion, NoShortForwardBranch],
    Constraints = "$dst = $falsev", isCommutable = 1 in {
// This instruction moves $truev to $dst when the condition is true. It will
// be expanded to control flow in RISCVExpandPseudoInsts.
// We use GPRNoX0 because c.mv cannot encode X0.
def PseudoCCMOVGPRNoX0 : Pseudo<(outs GPRNoX0:$dst),
                                (ins GPRNoX0:$falsev, GPRNoX0:$truev, bcc_opcode:$bcc,
                                 GPR:$lhs, sfb_rhs:$rhs),
                                []>,
                         Sched<[]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 103-118: TableGen class SFBALU_rr / TableGen 类 SFBALU_rr
```tablegen

let Predicates = [HasConditionalMoveFusion, NoShortForwardBranch] in
def :Pat<(riscv_selectcc (XLenVT GPR:$lhs), GPR:$rhs, cond:$cc, (XLenVT GPRNoX0:$truev), GPRNoX0:$falsev),
         (PseudoCCMOVGPRNoX0 GPRNoX0:$falsev, GPRNoX0:$truev, (CCtoRISCVBCC $cc), GPR:$lhs, GPR:$rhs )>;

class SFBALU_rr
    : Pseudo<(outs GPR:$dst),
             (ins GPR:$falsev, GPR:$rs1, GPR:$rs2, bcc_opcode:$bcc,
                  GPR:$lhs, sfb_rhs:$rhs), []>,
      Sched<[WriteSFB, ReadSFBALU, ReadSFBALU,
             ReadSFBALU, ReadSFBJmp, ReadSFBJmp]> {
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 0;
  let Constraints = "$dst = $falsev";
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 119-129: TableGen class SFBALU_ri / TableGen 类 SFBALU_ri
```tablegen

class SFBALU_ri
    : Pseudo<(outs GPR:$dst),
             (ins GPR:$falsev, GPR:$rs1, simm12_lo:$imm, bcc_opcode:$bcc,
                  GPR:$lhs, sfb_rhs:$rhs), []>,
      Sched<[WriteSFB, ReadSFBALU, ReadSFBALU, ReadSFBJmp, ReadSFBJmp]> {
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 0;
  let Constraints = "$dst = $falsev";
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 130-139: TableGen class SFBLUI / TableGen 类 SFBLUI
```tablegen

class SFBLUI
    : Pseudo<(outs GPR:$dst),
             (ins GPR:$falsev, uimm20_lui:$imm, bcc_opcode:$bcc,
                  GPR:$lhs, sfb_rhs:$rhs), []> {
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 0;
  let Constraints = "$dst = $falsev";
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 140-149: TableGen class SFBLoad / TableGen 类 SFBLoad
```tablegen

class SFBLoad
    : Pseudo<(outs GPR:$dst),
             (ins GPR:$falsev, GPR:$rs1, simm12_lo:$imm, bcc_opcode:$bcc,
                  GPR:$lhs, sfb_rhs:$rhs), []> {
  let hasSideEffects = 0;
  let mayLoad = 1;
  let mayStore = 0;
  let Constraints = "$dst = $falsev";
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 150-160: TableGen class SFBShift_ri / TableGen 类 SFBShift_ri
```tablegen

class SFBShift_ri
    : Pseudo<(outs GPR:$dst),
             (ins GPR:$falsev, GPR:$rs1, uimmlog2xlen:$imm, bcc_opcode:$bcc,
                  GPR:$lhs, sfb_rhs:$rhs), []>,
      Sched<[WriteSFB, ReadSFBALU, ReadSFBALU, ReadSFBJmp, ReadSFBJmp]> {
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 0;
  let Constraints = "$dst = $falsev";
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 161-171: TableGen class SFBShiftW_ri / TableGen 类 SFBShiftW_ri
```tablegen

class SFBShiftW_ri
    : Pseudo<(outs GPR:$dst),
             (ins GPR:$falsev, GPR:$rs1, uimm5:$imm, bcc_opcode:$bcc,
                  GPR:$lhs, sfb_rhs:$rhs), []>,
      Sched<[WriteSFB, ReadSFBALU, ReadSFBALU, ReadSFBJmp, ReadSFBJmp]> {
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 0;
  let Constraints = "$dst = $falsev";
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 172-186: TableGen record PseudoCCADD / TableGen 记录 PseudoCCADD
```tablegen

// Conditional binops, that updates update $dst to (op rs1, rs2) when condition
// is true. Returns $falsev otherwise. Selected by optimizeSelect.
// TODO: Can we use DefaultOperands on the regular binop to accomplish this more
// like how ARM does predication?
let Predicates = [HasShortForwardBranchIALU] in {
def PseudoCCADD : SFBALU_rr;
def PseudoCCSUB : SFBALU_rr;
def PseudoCCSLL : SFBALU_rr;
def PseudoCCSRL : SFBALU_rr;
def PseudoCCSRA : SFBALU_rr;
def PseudoCCAND : SFBALU_rr;
def PseudoCCOR  : SFBALU_rr;
def PseudoCCXOR : SFBALU_rr;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 187-197: TableGen record PseudoCCADDI / TableGen 记录 PseudoCCADDI
```tablegen
def PseudoCCADDI : SFBALU_ri;
def PseudoCCANDI : SFBALU_ri;
def PseudoCCORI  : SFBALU_ri;
def PseudoCCXORI : SFBALU_ri;

def PseudoCCLUI : SFBLUI;

def PseudoCCSLLI : SFBShift_ri;
def PseudoCCSRLI : SFBShift_ri;
def PseudoCCSRAI : SFBShift_ri;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 198-210: TableGen record PseudoCCADDW / TableGen 记录 PseudoCCADDW
```tablegen
// RV64I instructions
def PseudoCCADDW : SFBALU_rr;
def PseudoCCSUBW : SFBALU_rr;
def PseudoCCSLLW : SFBALU_rr;
def PseudoCCSRLW : SFBALU_rr;
def PseudoCCSRAW : SFBALU_rr;

def PseudoCCADDIW : SFBALU_ri;

def PseudoCCSLLIW : SFBShiftW_ri;
def PseudoCCSRLIW : SFBShiftW_ri;
def PseudoCCSRAIW : SFBShiftW_ri;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 211-225: TableGen record PseudoCCANDN / TableGen 记录 PseudoCCANDN
```tablegen
// Zbb/Zbkb instructions
def PseudoCCANDN : SFBALU_rr;
def PseudoCCORN  : SFBALU_rr;
def PseudoCCXNOR : SFBALU_rr;
}

// imm -> bcc_opcode for reg, reg  branches
def RISCVCCtoRISCVBCC : SDNodeXForm<imm, [{
  auto CCImm = cast<ConstantSDNode>(N)->getZExtValue();
  auto RvCC = static_cast<RISCVCC::CondCode>(CCImm);
  RvCC = RISCVCC::getInverseBranchCondition(RvCC);
  unsigned BccOpcode = RISCVCC::getBrCond(RvCC);
  return CurDAG->getTargetConstant(BccOpcode, SDLoc(N), MVT::i32);
}]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 226-240: TableGen record PseudoCCMAX / TableGen 记录 PseudoCCMAX
```tablegen
let Predicates = [HasShortForwardBranchIALU] in
def : Pat<(XLenVT (abs GPR:$rs1)),
          (PseudoCCSUB (XLenVT GPR:$rs1), (XLenVT X0), (XLenVT GPR:$rs1),
          (RISCVCCtoRISCVBCC (i32 /* COND_LT */ 2)),(XLenVT GPR:$rs1), (XLenVT X0))>;
let Predicates = [HasShortForwardBranchIALU, IsRV64] in
def : Pat<(sext_inreg (abs 33signbits_node:$rs1), i32),
          (PseudoCCSUBW (i64 GPR:$rs1), (i64 X0), (i64 GPR:$rs1), (RISCVCCtoRISCVBCC (i32 /* COND_LT */ 2)),
           (i64 GPR:$rs1), (i64 X0))>;

let Predicates = [HasShortForwardBranchIMinMax] in {
def PseudoCCMAX : SFBALU_rr;
def PseudoCCMIN : SFBALU_rr;
def PseudoCCMAXU : SFBALU_rr;
def PseudoCCMINU : SFBALU_rr;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 241-253: TableGen record PseudoCCMUL / TableGen 记录 PseudoCCMUL
```tablegen

let Predicates = [HasShortForwardBranchIMul] in
def PseudoCCMUL : SFBALU_rr;

let Predicates = [HasShortForwardBranchILoad] in {
def PseudoCCLB : SFBLoad;
def PseudoCCLH : SFBLoad;
def PseudoCCLW : SFBLoad;
def PseudoCCLHU : SFBLoad;
def PseudoCCLBU : SFBLoad;
def PseudoCCLWU : SFBLoad;
def PseudoCCLD : SFBLoad;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
