# RISCVMacroFusion.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVMacroFusion.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Commentary and design intent / 注释与设计意图
```tablegen
//==----- RISCVMacroFusion.td - Macro Fusion Definitions -----*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

// ===---------------------------------------------------------------------===//
// The following definitions describe the macro fusion predicators.
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 12-21: TableGen record TuneLUIADDIFusion / TableGen 记录 TuneLUIADDIFusion
```tablegen
// Fuse LUI followed by ADDI or ADDIW:
//   rd = imm[31:0] which decomposes to
//   lui rd, imm[31:12]
//   addi(w) rd, rd, imm[11:0]
def TuneLUIADDIFusion
  : SimpleFusion<"lui-addi", "HasLUIADDIFusion",
                 "Enable LUI+ADDI macro fusion",
                 CheckOpcode<[LUI]>,
                 CheckOpcode<[ADDI, ADDIW]>>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 22-41: TableGen record TuneAUIPCADDIFusion / TableGen 记录 TuneAUIPCADDIFusion
```tablegen
// Fuse AUIPC followed by ADDI:
//   auipc rd, imm20
//   addi rd, rd, imm12
def TuneAUIPCADDIFusion
  : SimpleFusion<"auipc-addi", "HasAUIPCADDIFusion",
                 "Enable AUIPC+ADDI macrofusion",
                 CheckOpcode<[AUIPC]>,
                 CheckOpcode<[ADDI]>>;

// Fuse zero extension of halfword:
//   slli rd, rs1, 48
//   srli rd, rd, 48
def TuneZExtHFusion
  : SimpleFusion<"zexth", "HasZExtHFusion",
                 "Enable SLLI+SRLI to be fused to zero extension of halfword",
                 CheckAll<[
                   CheckOpcode<[SLLI]>,
                   CheckIsImmOperand<2>,
                   CheckImmOperand<2, 48>
                 ]>,
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 42-61: TableGen record TuneZExtWFusion / TableGen 记录 TuneZExtWFusion
```tablegen
                 CheckAll<[
                   CheckOpcode<[SRLI]>,
                   CheckIsImmOperand<2>,
                   CheckImmOperand<2, 48>
                 ]>>;

// Fuse zero extension of word:
//   slli rd, rs1, 32
//   srli rd, rd, 32
def TuneZExtWFusion
  : SimpleFusion<"zextw", "HasZExtWFusion",
                 "Enable SLLI+SRLI to be fused to zero extension of word",
                 CheckAll<[
                   CheckOpcode<[SLLI]>,
                   CheckIsImmOperand<2>,
                   CheckImmOperand<2, 32>
                 ]>,
                 CheckAll<[
                   CheckOpcode<[SRLI]>,
                   CheckIsImmOperand<2>,
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 62-81: TableGen record TuneShiftedZExtWFusion / TableGen 记录 TuneShiftedZExtWFusion
```tablegen
                   CheckImmOperand<2, 32>
                 ]>>;

// Fuse shifted zero extension of word:
//   slli rd, rs1, 32
//   srli rd, rd, x
//   where 0 <= x < 32
def TuneShiftedZExtWFusion
  : SimpleFusion<"shifted-zextw", "HasShiftedZExtWFusion",
                 "Enable SLLI+SRLI to be fused when computing (shifted) word zero extension",
                 CheckAll<[
                   CheckOpcode<[SLLI]>,
                   CheckIsImmOperand<2>,
                   CheckImmOperand<2, 32>
                 ]>,
                 CheckAll<[
                   CheckOpcode<[SRLI]>,
                   CheckIsImmOperand<2>,
                   CheckImmOperandRange<2, 0, 31>
                 ]>>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 82-94: TableGen record TuneLDADDFusion / TableGen 记录 TuneLDADDFusion
```tablegen

// Fuse load with add:
//   add rd, rs1, rs2
//   ld rd, 0(rd)
def TuneLDADDFusion
  : SimpleFusion<"ld-add", "HasLDADDFusion", "Enable LD+ADD macrofusion",
                 CheckOpcode<[ADD]>,
                 CheckAll<[
                   CheckOpcode<[LD]>,
                   CheckIsImmOperand<2>,
                   CheckImmOperand<2, 0>
                 ]>>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 95-104: Definitions and supporting logic / 定义与支撑逻辑
```tablegen
defvar Load = [LB, LH, LW, LD, LBU, LHU, LWU];
defvar LoadStore = [LB, LBU, LH, LHU, LW, LWU, LD,
                    FLH, FLW, FLD,
                    SB, SH, SW, SD,
                    FSH, FSW, FSD];
defvar LogicOp = [AND, OR, XOR];
defvar LogicImmOp = [ANDI, ORI, XORI];
defvar ShiftLeft = [SLLI, SLLIW];
defvar ShiftRight = [SRLI, SRLIW, SRAI, SRAIW];
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 105-121: TableGen record TuneADDLoadFusion / TableGen 记录 TuneADDLoadFusion
```tablegen
// Fuse add(.uw) followed by a load (lb, lh, lw, ld, lbu, lhu, lwu):
//   add(.uw) rd, rs1, rs2
//   load rd, imm12(rd)
def TuneADDLoadFusion
  : SimpleFusion<"add-load", "HasADDLoadFusion", "Enable ADD(.UW) + load macrofusion",
                 CheckOpcode<[ADD, ADD_UW]>,
                 CheckOpcode<Load>>;

// Fuse AUIPC followed by by a load (lb, lh, lw, ld, lbu, lhu, lwu)
//   auipc rd, imm20
//   load rd, imm12(rd)
def TuneAUIPCLoadFusion
  : SimpleFusion<"auipc-load", "HasAUIPCLoadFusion",
                 "Enable AUIPC + load macrofusion",
                 CheckOpcode<[AUIPC]>,
                 CheckOpcode<Load>>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 122-140: TableGen record TuneLUILoadFusion / TableGen 记录 TuneLUILoadFusion
```tablegen
// Fuse LUI followed by a load (lb, lh, lw, ld, lbu, lhu, lwu)
//   lui rd, imm[31:12]
//   load rd, imm12(rd)
def TuneLUILoadFusion
  : SimpleFusion<"lui-load", "HasLUILoadFusion",
                 "Enable LUI + load macrofusion",
                 CheckOpcode<[LUI]>,
                 CheckOpcode<Load>>;

// Bitfield extract fusion: similar to TuneShiftedZExtWFusion
// but without the immediate restriction
//   slli rd, rs1, imm12
//   srli rd, rd, imm12
def TuneBFExtFusion
  : SimpleFusion<"bfext", "HasBFExtFusion",
                 "Enable SLLI+SRLI (bitfield extract) macrofusion",
                 CheckOpcode<[SLLI]>,
                 CheckOpcode<[SRLI]>>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 141-158: TableGen record TuneADDILoadFusion / TableGen 记录 TuneADDILoadFusion
```tablegen
// Fuse ADDI followed by a load (lb, lh, lw, ld, lbu, lhu, lwu)
//   addi rd, rs1, imm12
//   load rd, imm12(rd)
def TuneADDILoadFusion
  : SimpleFusion<"addi-load", "HasADDILoadFusion",
                 "Enable ADDI + load macrofusion",
                 CheckOpcode<[ADDI]>,
                 CheckOpcode<Load>>;

// Fuse shXadd(.uw) followed by a load (lb, lh, lw, ld, lbu, lhu, lwu)
//   shXadd(.uw) rd, rs1, rs2
//   load rd, imm12(rd)
def TuneSHXADDLoadFusion
  : SimpleFusion<"shxadd-load", "HasSHXADDLoadFusion",
                 "Enable SH(1|2|3)ADD(.UW) + load macrofusion",
                 CheckOpcode<[SH1ADD, SH2ADD, SH3ADD, SH1ADD_UW, SH2ADD_UW, SH3ADD_UW]>,
                 CheckOpcode<Load>>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 159-168: TableGen record TuneFusionLogicRegReg / TableGen 记录 TuneFusionLogicRegReg
```tablegen
// Fuse logic operation followed by another logic operation:
//   and/or/xor rd, rs1, rs2
//   and/or/xor rd, rd, rs3
let IsCommutable = 1 in
def TuneFusionLogicRegReg
  : SimpleFusion<"logic-reg-reg", "HasFusionLogicRegReg",
                 "Enable AND/OR/XOR+AND/OR/XOR macrofusion",
                 CheckOpcode<LogicOp>,
                 CheckOpcode<LogicOp>>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 169-187: TableGen record TuneFusionLogicRegImm / TableGen 记录 TuneFusionLogicRegImm
```tablegen
// Fuse logic operation followed by logic immediate operation:
//   and/or/xor rd, rs1, rs2
//   andi/ori/xori rd, rd, imm
def TuneFusionLogicRegImm
  : SimpleFusion<"logic-reg-imm", "HasFusionLogicRegImm",
                 "Enable AND/OR/XOR+ANDI/ORI/XORI macrofusion",
                 CheckOpcode<LogicOp>,
                 CheckOpcode<LogicImmOp>>;

// Fuse logic immediate operation followed by logic operation:
//   andi/ori/xori rd, rs1, imm
//   and/or/xor rd, rd, rs2
let IsCommutable = 1 in
def TuneFusionLogicImmReg
  : SimpleFusion<"logic-imm-reg", "HasFusionLogicImmReg",
                 "Enable ANDI/ORI/XORI+AND/OR/XOR macrofusion",
                 CheckOpcode<LogicImmOp>,
                 CheckOpcode<LogicOp>>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 188-207: TableGen record TuneFusionMulAdd / TableGen 记录 TuneFusionMulAdd
```tablegen
// Fuse multiply followed by add:
//   mul(w) rd, rs1, rs2
//   add(w) rd, rd, rs3
def TuneFusionMulAdd
  : SimpleFusion<"mul-add", "HasFusionMulAdd",
                 "Enable MUL+ADD macrofusion",
                 CheckOpcode<[MUL, MULW]>,
                 CheckOpcode<[ADD, ADDW]>,
                 epilog = [
                   // Check that both instructions have the same width
                   FusionPredicateWithCode<[{
                     if ((FirstMI->getDesc().TSFlags & RISCVII::IsSignExtendingOpWMask) !=
                         (SecondMI.getDesc().TSFlags & RISCVII::IsSignExtendingOpWMask))
                       return false;
                   }]>,
                   SecondFusionPredicateWithMCInstPredicate<
                     CheckNot<CheckSameRegOperand<0, 2>>
                   >
                 ]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 208-227: TableGen record TuneFusionAddMem / TableGen 记录 TuneFusionAddMem
```tablegen
// Fuse add with load or store with zero offset:
//   add rd, rs1, rs2
//   load/store rt, 0(rd)
def TuneFusionAddMem
  : Fusion<"add-mem", "HasFusionAddMem",
           "Enable ADD+LOAD/STORE macrofusion",
           [
             SecondFusionPredicateWithMCInstPredicate<
               CheckAll<[
                 CheckOpcode<LoadStore>,
                 CheckIsImmOperand<2>,
                 CheckImmOperand<2, 0>
               ]>
             >,
             WildcardTrue,
             FirstFusionPredicateWithMCInstPredicate<
               CheckAll<[
                 CheckOpcode<[ADD]>,
                 CheckNot<CheckSameRegOperand<0, 1>>,
                 CheckNot<CheckSameRegOperand<0, 2>>
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 228-247: TableGen record TuneFusionShiftBitExtract / TableGen 记录 TuneFusionShiftBitExtract
```tablegen
               ]>
             >,
             TieReg<0, 1>
           ]>;

// Fuse shift left followed by shift right for bit extraction:
//   slli(w) rd, rs1, imm1
//   srli(w)/srai(w) rd, rd, imm2
//   where imm1 <= imm2
def TuneFusionShiftBitExtract
  : SimpleFusion<"shift-bit-extract", "HasFusionShiftBitExtract",
                 "Enable SLLI+SRLI/SRAI macrofusion",
                 CheckOpcode<ShiftLeft>,
                 CheckOpcode<ShiftRight>,
                 epilog = [
                   // Check that both instructions have the same width
                   FusionPredicateWithCode<[{
                     if ((FirstMI->getDesc().TSFlags & RISCVII::IsSignExtendingOpWMask) !=
                         (SecondMI.getDesc().TSFlags & RISCVII::IsSignExtendingOpWMask))
                       return false;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 248-257: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```tablegen
                   }]>,
                   // Check that FirstMI's immediate <= SecondMI's immediate
                   FusionPredicateWithCode<[{
                     if (!(FirstMI->getOperand(2).isImm() &&
                           SecondMI.getOperand(2).isImm() &&
                           FirstMI->getOperand(2).getImm() <=
                           SecondMI.getOperand(2).getImm()))
                       return false;
                   }]>,
                 ]>;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

## Key Concepts / 关键概念
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
