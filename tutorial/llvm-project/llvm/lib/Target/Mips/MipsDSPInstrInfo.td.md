# MipsDSPInstrInfo.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsDSPInstrInfo.td`
- Repository: `llvm-project`
- Purpose (EN): This file describes Mips DSP ASE instructions.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `MipsDSPInstrInfo`，涵盖指令语义、调度提示以及机器级辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===- MipsDSPInstrInfo.td - DSP ASE instructions -*- tablegen ------------*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes Mips DSP ASE instructions.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-28
```tablegen
// ImmLeaf
def immZExt1 : ImmLeaf<i32, [{return isUInt<1>(Imm);}]>;
def timmZExt1 : ImmLeaf<i32, [{return isUInt<1>(Imm);}], NOOP_SDNodeXForm, timm>;
def immZExt2 : ImmLeaf<i32, [{return isUInt<2>(Imm);}]>;
def timmZExt2 : ImmLeaf<i32, [{return isUInt<2>(Imm);}], NOOP_SDNodeXForm, timm>;
def immZExt3 : ImmLeaf<i32, [{return isUInt<3>(Imm);}]>;
def timmZExt3 : ImmLeaf<i32, [{return isUInt<3>(Imm);}], NOOP_SDNodeXForm, timm>;
def immZExt4 : ImmLeaf<i32, [{return isUInt<4>(Imm);}]>;
def timmZExt4 : ImmLeaf<i32, [{return isUInt<4>(Imm);}], NOOP_SDNodeXForm, timm>;
def immZExt8 : ImmLeaf<i32, [{return isUInt<8>(Imm);}]>;
def timmZExt8 : ImmLeaf<i32, [{return isUInt<8>(Imm);}], NOOP_SDNodeXForm, timm>;
def immZExt10 : ImmLeaf<i32, [{return isUInt<10>(Imm);}]>;
def timmZExt10 : ImmLeaf<i32, [{return isUInt<10>(Imm);}], NOOP_SDNodeXForm, timm>;
def immSExt6 : ImmLeaf<i32, [{return isInt<6>(Imm);}]>;
def timmSExt6 : ImmLeaf<i32, [{return isInt<6>(Imm);}], NOOP_SDNodeXForm, timm>;
def immSExt10 : ImmLeaf<i32, [{return isInt<10>(Imm);}]>;
```
- EN: Defines TableGen record `immZExt1` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `immZExt1`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 30-42
```tablegen
// Mips-specific dsp nodes
def SDT_MipsExtr : SDTypeProfile<1, 2, [SDTCisVT<0, i32>, SDTCisSameAs<0, 1>,
                                        SDTCisVT<2, untyped>]>;
def SDT_MipsShilo : SDTypeProfile<1, 2, [SDTCisVT<0, untyped>,
                                         SDTCisSameAs<0, 2>, SDTCisVT<1, i32>]>;
def SDT_MipsDPA_H : SDTypeProfile<1, 3, [SDTCisVT<0, untyped>, SDTCisSameAs<0, 3>,
                                         SDTCisVT<1, v4i8>, SDTCisSameAs<1, 2>]>;
def SDT_MipsDPA_W : SDTypeProfile<1, 3, [SDTCisVT<0, untyped>, SDTCisSameAs<0, 3>,
                                         SDTCisVT<1, v2i16>, SDTCisSameAs<1, 2>]>;
def SDT_MipsDPA_L : SDTypeProfile<1, 3, [SDTCisVT<0, untyped>, SDTCisSameAs<0, 3>,
                                         SDTCisVT<1, i32>, SDTCisSameAs<1, 2>]>;
def SDT_MipsSHIFT_DSP : SDTypeProfile<1, 2, [SDTCisVec<0>, SDTCisSameAs<0, 1>,
                                             SDTCisVT<2, i32>]>;
```
- EN: Defines TableGen record `SDT_MipsExtr` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_MipsExtr`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 44-45
```tablegen
class MipsDSPBase<string Opc, SDTypeProfile Prof> :
  SDNode<!strconcat("MipsISD::", Opc), Prof>;
```
- EN: Declares reusable TableGen class `MipsDSPBase` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MipsDSPBase`，通常用于抽象共享字段、谓词或编码结构。

### Lines 47-48
```tablegen
class MipsDSPSideEffectBase<string Opc, SDTypeProfile Prof> :
  SDNode<!strconcat("MipsISD::", Opc), Prof, [SDNPHasChain, SDNPSideEffect]>;
```
- EN: Declares reusable TableGen class `MipsDSPSideEffectBase` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MipsDSPSideEffectBase`，通常用于抽象共享字段、谓词或编码结构。

### Lines 50-56
```tablegen
// EXTR.W intrinsic nodes.
def MipsEXTP : MipsDSPSideEffectBase<"EXTP", SDT_MipsExtr>;
def MipsEXTPDP : MipsDSPSideEffectBase<"EXTPDP", SDT_MipsExtr>;
def MipsEXTR_S_H : MipsDSPSideEffectBase<"EXTR_S_H", SDT_MipsExtr>;
def MipsEXTR_W : MipsDSPSideEffectBase<"EXTR_W", SDT_MipsExtr>;
def MipsEXTR_R_W : MipsDSPSideEffectBase<"EXTR_R_W", SDT_MipsExtr>;
def MipsEXTR_RS_W : MipsDSPSideEffectBase<"EXTR_RS_W", SDT_MipsExtr>;
```
- EN: Defines TableGen record `MipsEXTP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MipsEXTP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 58-59
```tablegen
def MipsSHILO : MipsDSPBase<"SHILO", SDT_MipsShilo>;
def MipsMTHLIP : MipsDSPSideEffectBase<"MTHLIP", SDT_MipsShilo>;
```
- EN: Defines TableGen record `MipsSHILO` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MipsSHILO`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 61-66
```tablegen
// DPA.W intrinsic nodes.
def MipsMULSAQ_S_W_PH : MipsDSPSideEffectBase<"MULSAQ_S_W_PH", SDT_MipsDPA_W>;
def MipsMAQ_S_W_PHL : MipsDSPSideEffectBase<"MAQ_S_W_PHL", SDT_MipsDPA_W>;
def MipsMAQ_S_W_PHR : MipsDSPSideEffectBase<"MAQ_S_W_PHR", SDT_MipsDPA_W>;
def MipsMAQ_SA_W_PHL : MipsDSPSideEffectBase<"MAQ_SA_W_PHL", SDT_MipsDPA_W>;
def MipsMAQ_SA_W_PHR : MipsDSPSideEffectBase<"MAQ_SA_W_PHR", SDT_MipsDPA_W>;
```
- EN: Defines TableGen record `MipsMULSAQ_S_W_PH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MipsMULSAQ_S_W_PH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 68-75
```tablegen
def MipsDPAU_H_QBL : MipsDSPBase<"DPAU_H_QBL", SDT_MipsDPA_H>;
def MipsDPAU_H_QBR : MipsDSPBase<"DPAU_H_QBR", SDT_MipsDPA_H>;
def MipsDPSU_H_QBL : MipsDSPBase<"DPSU_H_QBL", SDT_MipsDPA_H>;
def MipsDPSU_H_QBR : MipsDSPBase<"DPSU_H_QBR", SDT_MipsDPA_H>;
def MipsDPAQ_S_W_PH : MipsDSPSideEffectBase<"DPAQ_S_W_PH", SDT_MipsDPA_W>;
def MipsDPSQ_S_W_PH : MipsDSPSideEffectBase<"DPSQ_S_W_PH", SDT_MipsDPA_W>;
def MipsDPAQ_SA_L_W : MipsDSPSideEffectBase<"DPAQ_SA_L_W", SDT_MipsDPA_L>;
def MipsDPSQ_SA_L_W : MipsDSPSideEffectBase<"DPSQ_SA_L_W", SDT_MipsDPA_L>;
```
- EN: Defines TableGen record `MipsDPAU_H_QBL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MipsDPAU_H_QBL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 77-85
```tablegen
def MipsDPA_W_PH : MipsDSPBase<"DPA_W_PH", SDT_MipsDPA_W>;
def MipsDPS_W_PH : MipsDSPBase<"DPS_W_PH", SDT_MipsDPA_W>;
def MipsDPAQX_S_W_PH : MipsDSPSideEffectBase<"DPAQX_S_W_PH", SDT_MipsDPA_W>;
def MipsDPAQX_SA_W_PH : MipsDSPSideEffectBase<"DPAQX_SA_W_PH", SDT_MipsDPA_W>;
def MipsDPAX_W_PH : MipsDSPBase<"DPAX_W_PH", SDT_MipsDPA_W>;
def MipsDPSX_W_PH : MipsDSPBase<"DPSX_W_PH", SDT_MipsDPA_W>;
def MipsDPSQX_S_W_PH : MipsDSPSideEffectBase<"DPSQX_S_W_PH", SDT_MipsDPA_W>;
def MipsDPSQX_SA_W_PH : MipsDSPSideEffectBase<"DPSQX_SA_W_PH", SDT_MipsDPA_W>;
def MipsMULSA_W_PH : MipsDSPBase<"MULSA_W_PH", SDT_MipsDPA_W>;
```
- EN: Defines TableGen record `MipsDPA_W_PH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MipsDPA_W_PH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 87-92
```tablegen
def MipsMULT : MipsDSPBase<"MULT", SDT_MipsDPA_L>;
def MipsMULTU : MipsDSPBase<"MULTU", SDT_MipsDPA_L>;
def MipsMADD_DSP : MipsDSPBase<"MADD_DSP", SDT_MipsDPA_L>;
def MipsMADDU_DSP : MipsDSPBase<"MADDU_DSP", SDT_MipsDPA_L>;
def MipsMSUB_DSP : MipsDSPBase<"MSUB_DSP", SDT_MipsDPA_L>;
def MipsMSUBU_DSP : MipsDSPBase<"MSUBU_DSP", SDT_MipsDPA_L>;
```
- EN: Defines TableGen record `MipsMULT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MipsMULT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 94-97
```tablegen
// DSP shift nodes.
def MipsSHLL_DSP : MipsDSPBase<"SHLL_DSP", SDT_MipsSHIFT_DSP>;
def MipsSHRA_DSP : MipsDSPBase<"SHRA_DSP", SDT_MipsSHIFT_DSP>;
def MipsSHRL_DSP : MipsDSPBase<"SHRL_DSP", SDT_MipsSHIFT_DSP>;
```
- EN: Defines TableGen record `MipsSHLL_DSP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MipsSHLL_DSP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 99-101
```tablegen
// DSP setcc and select_cc nodes.
def MipsSETCC_DSP : MipsDSPBase<"SETCC_DSP", SDTSetCC>;
def MipsSELECT_CC_DSP : MipsDSPBase<"SELECT_CC_DSP", SDTSelectCC>;
```
- EN: Defines TableGen record `MipsSETCC_DSP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MipsSETCC_DSP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 103-106
```tablegen
// Flags.
class Uses<list<Register> Regs> {
  list<Register> Uses = Regs;
}
```
- EN: Declares reusable TableGen class `Uses` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `Uses`，通常用于抽象共享字段、谓词或编码结构。

### Lines 108-110
```tablegen
class Defs<list<Register> Regs> {
  list<Register> Defs = Regs;
}
```
- EN: Declares reusable TableGen class `Defs` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `Defs`，通常用于抽象共享字段、谓词或编码结构。

### Lines 112-129
```tablegen
// Instruction encoding.
class ADDU_QB_ENC : ADDU_QB_FMT<0b00000>;
class ADDU_S_QB_ENC : ADDU_QB_FMT<0b00100>;
class SUBU_QB_ENC : ADDU_QB_FMT<0b00001>;
class SUBU_S_QB_ENC : ADDU_QB_FMT<0b00101>;
class ADDQ_PH_ENC : ADDU_QB_FMT<0b01010>;
class ADDQ_S_PH_ENC : ADDU_QB_FMT<0b01110>;
class SUBQ_PH_ENC : ADDU_QB_FMT<0b01011>;
class SUBQ_S_PH_ENC : ADDU_QB_FMT<0b01111>;
class ADDQ_S_W_ENC : ADDU_QB_FMT<0b10110>;
class SUBQ_S_W_ENC : ADDU_QB_FMT<0b10111>;
class ADDSC_ENC : ADDU_QB_FMT<0b10000>;
class ADDWC_ENC : ADDU_QB_FMT<0b10001>;
class MODSUB_ENC : ADDU_QB_FMT<0b10010>;
class RADDU_W_QB_ENC : RADDU_W_QB_FMT<0b10100>;
class ABSQ_S_PH_ENC : ABSQ_S_PH_R2_FMT<0b01001>;
class ABSQ_S_W_ENC : ABSQ_S_PH_R2_FMT<0b10001>;
class PRECRQ_QB_PH_ENC : CMP_EQ_QB_R3_FMT<0b01100>;
```
- EN: Declares reusable TableGen class `ADDU_QB_ENC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ADDU_QB_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 130-147
```tablegen
class PRECRQ_PH_W_ENC : CMP_EQ_QB_R3_FMT<0b10100>;
class PRECRQ_RS_PH_W_ENC : CMP_EQ_QB_R3_FMT<0b10101>;
class PRECRQU_S_QB_PH_ENC : CMP_EQ_QB_R3_FMT<0b01111>;
class PRECEQ_W_PHL_ENC : ABSQ_S_PH_R2_FMT<0b01100>;
class PRECEQ_W_PHR_ENC : ABSQ_S_PH_R2_FMT<0b01101>;
class PRECEQU_PH_QBL_ENC : ABSQ_S_PH_R2_FMT<0b00100>;
class PRECEQU_PH_QBR_ENC : ABSQ_S_PH_R2_FMT<0b00101>;
class PRECEQU_PH_QBLA_ENC : ABSQ_S_PH_R2_FMT<0b00110>;
class PRECEQU_PH_QBRA_ENC : ABSQ_S_PH_R2_FMT<0b00111>;
class PRECEU_PH_QBL_ENC : ABSQ_S_PH_R2_FMT<0b11100>;
class PRECEU_PH_QBR_ENC : ABSQ_S_PH_R2_FMT<0b11101>;
class PRECEU_PH_QBLA_ENC : ABSQ_S_PH_R2_FMT<0b11110>;
class PRECEU_PH_QBRA_ENC : ABSQ_S_PH_R2_FMT<0b11111>;
class SHLL_QB_ENC : SHLL_QB_FMT<0b00000>;
class SHLLV_QB_ENC : SHLL_QB_FMT<0b00010>;
class SHRL_QB_ENC : SHLL_QB_FMT<0b00001>;
class SHRLV_QB_ENC : SHLL_QB_FMT<0b00011>;
class SHLL_PH_ENC : SHLL_QB_FMT<0b01000>;
```
- EN: Declares reusable TableGen class `PRECRQ_PH_W_ENC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PRECRQ_PH_W_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 148-165
```tablegen
class SHLLV_PH_ENC : SHLL_QB_FMT<0b01010>;
class SHLL_S_PH_ENC : SHLL_QB_FMT<0b01100>;
class SHLLV_S_PH_ENC : SHLL_QB_FMT<0b01110>;
class SHRA_PH_ENC : SHLL_QB_FMT<0b01001>;
class SHRAV_PH_ENC : SHLL_QB_FMT<0b01011>;
class SHRA_R_PH_ENC : SHLL_QB_FMT<0b01101>;
class SHRAV_R_PH_ENC : SHLL_QB_FMT<0b01111>;
class SHLL_S_W_ENC : SHLL_QB_FMT<0b10100>;
class SHLLV_S_W_ENC : SHLL_QB_FMT<0b10110>;
class SHRA_R_W_ENC : SHLL_QB_FMT<0b10101>;
class SHRAV_R_W_ENC : SHLL_QB_FMT<0b10111>;
class MULEU_S_PH_QBL_ENC : ADDU_QB_FMT<0b00110>;
class MULEU_S_PH_QBR_ENC : ADDU_QB_FMT<0b00111>;
class MULEQ_S_W_PHL_ENC : ADDU_QB_FMT<0b11100>;
class MULEQ_S_W_PHR_ENC : ADDU_QB_FMT<0b11101>;
class MULQ_RS_PH_ENC : ADDU_QB_FMT<0b11111>;
class MULSAQ_S_W_PH_ENC : DPA_W_PH_FMT<0b00110>;
class MAQ_S_W_PHL_ENC : DPA_W_PH_FMT<0b10100>;
```
- EN: Declares reusable TableGen class `SHLLV_PH_ENC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHLLV_PH_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 166-183
```tablegen
class MAQ_S_W_PHR_ENC : DPA_W_PH_FMT<0b10110>;
class MAQ_SA_W_PHL_ENC : DPA_W_PH_FMT<0b10000>;
class MAQ_SA_W_PHR_ENC : DPA_W_PH_FMT<0b10010>;
class MFHI_ENC : MFHI_FMT<0b010000>;
class MFLO_ENC : MFHI_FMT<0b010010>;
class MTHI_ENC : MTHI_FMT<0b010001>;
class MTLO_ENC : MTHI_FMT<0b010011>;
class DPAU_H_QBL_ENC : DPA_W_PH_FMT<0b00011>;
class DPAU_H_QBR_ENC : DPA_W_PH_FMT<0b00111>;
class DPSU_H_QBL_ENC : DPA_W_PH_FMT<0b01011>;
class DPSU_H_QBR_ENC : DPA_W_PH_FMT<0b01111>;
class DPAQ_S_W_PH_ENC : DPA_W_PH_FMT<0b00100>;
class DPSQ_S_W_PH_ENC : DPA_W_PH_FMT<0b00101>;
class DPAQ_SA_L_W_ENC : DPA_W_PH_FMT<0b01100>;
class DPSQ_SA_L_W_ENC : DPA_W_PH_FMT<0b01101>;
class MULT_DSP_ENC : MULT_FMT<0b000000, 0b011000>;
class MULTU_DSP_ENC : MULT_FMT<0b000000, 0b011001>;
class MADD_DSP_ENC : MULT_FMT<0b011100, 0b000000>;
```
- EN: Declares reusable TableGen class `MAQ_S_W_PHR_ENC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MAQ_S_W_PHR_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 184-201
```tablegen
class MADDU_DSP_ENC : MULT_FMT<0b011100, 0b000001>;
class MSUB_DSP_ENC : MULT_FMT<0b011100, 0b000100>;
class MSUBU_DSP_ENC : MULT_FMT<0b011100, 0b000101>;
class CMPU_EQ_QB_ENC : CMP_EQ_QB_R2_FMT<0b00000>;
class CMPU_LT_QB_ENC : CMP_EQ_QB_R2_FMT<0b00001>;
class CMPU_LE_QB_ENC : CMP_EQ_QB_R2_FMT<0b00010>;
class CMPGU_EQ_QB_ENC : CMP_EQ_QB_R3_FMT<0b00100>;
class CMPGU_LT_QB_ENC : CMP_EQ_QB_R3_FMT<0b00101>;
class CMPGU_LE_QB_ENC : CMP_EQ_QB_R3_FMT<0b00110>;
class CMP_EQ_PH_ENC : CMP_EQ_QB_R2_FMT<0b01000>;
class CMP_LT_PH_ENC : CMP_EQ_QB_R2_FMT<0b01001>;
class CMP_LE_PH_ENC : CMP_EQ_QB_R2_FMT<0b01010>;
class BITREV_ENC : ABSQ_S_PH_R2_FMT<0b11011>;
class PACKRL_PH_ENC : CMP_EQ_QB_R3_FMT<0b01110>;
class REPL_QB_ENC : REPL_FMT<0b00010>;
class REPL_PH_ENC : REPL_FMT<0b01010>;
class REPLV_QB_ENC : ABSQ_S_PH_R2_FMT<0b00011>;
class REPLV_PH_ENC : ABSQ_S_PH_R2_FMT<0b01011>;
```
- EN: Declares reusable TableGen class `MADDU_DSP_ENC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MADDU_DSP_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 202-208
```tablegen
class PICK_QB_ENC : CMP_EQ_QB_R3_FMT<0b00011>;
class PICK_PH_ENC : CMP_EQ_QB_R3_FMT<0b01011>;
class LWX_ENC : LX_FMT<0b00000>;
class LHX_ENC : LX_FMT<0b00100>;
class LBUX_ENC : LX_FMT<0b00110>;
class BPOSGE32_ENC : BPOSGE32_FMT<0b11100>;
class INSV_ENC : INSV_FMT<0b001100>;
```
- EN: Declares reusable TableGen class `PICK_QB_ENC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PICK_QB_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 210-224
```tablegen
class EXTP_ENC : EXTR_W_TY1_FMT<0b00010>;
class EXTPV_ENC : EXTR_W_TY1_FMT<0b00011>;
class EXTPDP_ENC : EXTR_W_TY1_FMT<0b01010>;
class EXTPDPV_ENC : EXTR_W_TY1_FMT<0b01011>;
class EXTR_W_ENC : EXTR_W_TY1_FMT<0b00000>;
class EXTRV_W_ENC : EXTR_W_TY1_FMT<0b00001>;
class EXTR_R_W_ENC : EXTR_W_TY1_FMT<0b00100>;
class EXTRV_R_W_ENC : EXTR_W_TY1_FMT<0b00101>;
class EXTR_RS_W_ENC : EXTR_W_TY1_FMT<0b00110>;
class EXTRV_RS_W_ENC : EXTR_W_TY1_FMT<0b00111>;
class EXTR_S_H_ENC : EXTR_W_TY1_FMT<0b01110>;
class EXTRV_S_H_ENC : EXTR_W_TY1_FMT<0b01111>;
class SHILO_ENC : SHILO_R1_FMT<0b11010>;
class SHILOV_ENC : SHILO_R2_FMT<0b11011>;
class MTHLIP_ENC : SHILO_R2_FMT<0b11111>;
```
- EN: Declares reusable TableGen class `EXTP_ENC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `EXTP_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 226-243
```tablegen
class RDDSP_ENC : RDDSP_FMT<0b10010>;
class WRDSP_ENC : WRDSP_FMT<0b10011>;
class ADDU_PH_ENC : ADDU_QB_FMT<0b01000>;
class ADDU_S_PH_ENC : ADDU_QB_FMT<0b01100>;
class SUBU_PH_ENC : ADDU_QB_FMT<0b01001>;
class SUBU_S_PH_ENC : ADDU_QB_FMT<0b01101>;
class CMPGDU_EQ_QB_ENC : CMP_EQ_QB_R3_FMT<0b11000>;
class CMPGDU_LT_QB_ENC : CMP_EQ_QB_R3_FMT<0b11001>;
class CMPGDU_LE_QB_ENC : CMP_EQ_QB_R3_FMT<0b11010>;
class ABSQ_S_QB_ENC : ABSQ_S_PH_R2_FMT<0b00001>;
class ADDUH_QB_ENC : ADDUH_QB_FMT<0b00000>;
class ADDUH_R_QB_ENC : ADDUH_QB_FMT<0b00010>;
class SUBUH_QB_ENC : ADDUH_QB_FMT<0b00001>;
class SUBUH_R_QB_ENC : ADDUH_QB_FMT<0b00011>;
class ADDQH_PH_ENC : ADDUH_QB_FMT<0b01000>;
class ADDQH_R_PH_ENC : ADDUH_QB_FMT<0b01010>;
class SUBQH_PH_ENC : ADDUH_QB_FMT<0b01001>;
class SUBQH_R_PH_ENC : ADDUH_QB_FMT<0b01011>;
```
- EN: Declares reusable TableGen class `RDDSP_ENC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `RDDSP_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 244-261
```tablegen
class ADDQH_W_ENC : ADDUH_QB_FMT<0b10000>;
class ADDQH_R_W_ENC : ADDUH_QB_FMT<0b10010>;
class SUBQH_W_ENC : ADDUH_QB_FMT<0b10001>;
class SUBQH_R_W_ENC : ADDUH_QB_FMT<0b10011>;
class MUL_PH_ENC : ADDUH_QB_FMT<0b01100>;
class MUL_S_PH_ENC : ADDUH_QB_FMT<0b01110>;
class MULQ_S_W_ENC : ADDUH_QB_FMT<0b10110>;
class MULQ_RS_W_ENC : ADDUH_QB_FMT<0b10111>;
class MULQ_S_PH_ENC : ADDU_QB_FMT<0b11110>;
class DPA_W_PH_ENC : DPA_W_PH_FMT<0b00000>;
class DPS_W_PH_ENC : DPA_W_PH_FMT<0b00001>;
class DPAQX_S_W_PH_ENC : DPA_W_PH_FMT<0b11000>;
class DPAQX_SA_W_PH_ENC : DPA_W_PH_FMT<0b11010>;
class DPAX_W_PH_ENC : DPA_W_PH_FMT<0b01000>;
class DPSX_W_PH_ENC : DPA_W_PH_FMT<0b01001>;
class DPSQX_S_W_PH_ENC : DPA_W_PH_FMT<0b11001>;
class DPSQX_SA_W_PH_ENC : DPA_W_PH_FMT<0b11011>;
class MULSA_W_PH_ENC : DPA_W_PH_FMT<0b00010>;
```
- EN: Declares reusable TableGen class `ADDQH_W_ENC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ADDQH_W_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 262-273
```tablegen
class PRECR_QB_PH_ENC : CMP_EQ_QB_R3_FMT<0b01101>;
class PRECR_SRA_PH_W_ENC : PRECR_SRA_PH_W_FMT<0b11110>;
class PRECR_SRA_R_PH_W_ENC : PRECR_SRA_PH_W_FMT<0b11111>;
class SHRA_QB_ENC : SHLL_QB_FMT<0b00100>;
class SHRAV_QB_ENC : SHLL_QB_FMT<0b00110>;
class SHRA_R_QB_ENC : SHLL_QB_FMT<0b00101>;
class SHRAV_R_QB_ENC : SHLL_QB_FMT<0b00111>;
class SHRL_PH_ENC : SHLL_QB_FMT<0b11001>;
class SHRLV_PH_ENC : SHLL_QB_FMT<0b11011>;
class APPEND_ENC : APPEND_FMT<0b00000>;
class BALIGN_ENC : APPEND_FMT<0b10000>;
class PREPEND_ENC : APPEND_FMT<0b00001>;
```
- EN: Declares reusable TableGen class `PRECR_QB_PH_ENC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PRECR_QB_PH_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 275-284
```tablegen
// Instruction desc.
class ADDU_QB_DESC_BASE<string instr_asm, SDPatternOperator OpNode,
                        RegisterOperand ROD,
                        RegisterOperand ROS,  RegisterOperand ROT = ROS> {
  dag OutOperandList = (outs ROD:$rd);
  dag InOperandList = (ins ROS:$rs, ROT:$rt);
  string AsmString = !strconcat(instr_asm, "\t$rd, $rs, $rt");
  list<dag> Pattern = [(set ROD:$rd, (OpNode ROS:$rs, ROT:$rt))];
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `ADDU_QB_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ADDU_QB_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 286-294
```tablegen
class RADDU_W_QB_DESC_BASE<string instr_asm, SDPatternOperator OpNode,
                           RegisterOperand ROD,
                           RegisterOperand ROS = ROD> {
  dag OutOperandList = (outs ROD:$rd);
  dag InOperandList = (ins ROS:$rs);
  string AsmString = !strconcat(instr_asm, "\t$rd, $rs");
  list<dag> Pattern = [(set ROD:$rd, (OpNode ROS:$rs))];
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `RADDU_W_QB_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `RADDU_W_QB_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 296-304
```tablegen
class CMP_EQ_QB_R2_DESC_BASE<string instr_asm, SDPatternOperator OpNode,
                             RegisterOperand ROS,
                             RegisterOperand ROT = ROS> {
  dag OutOperandList = (outs);
  dag InOperandList = (ins ROS:$rs, ROT:$rt);
  string AsmString = !strconcat(instr_asm, "\t$rs, $rt");
  list<dag> Pattern = [(OpNode ROS:$rs, ROT:$rt)];
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `CMP_EQ_QB_R2_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `CMP_EQ_QB_R2_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 306-314
```tablegen
class CMP_EQ_QB_R3_DESC_BASE<string instr_asm, SDPatternOperator OpNode,
                             RegisterOperand ROD,
                             RegisterOperand ROS,  RegisterOperand ROT = ROS> {
  dag OutOperandList = (outs ROD:$rd);
  dag InOperandList = (ins ROS:$rs, ROT:$rt);
  string AsmString = !strconcat(instr_asm, "\t$rd, $rs, $rt");
  list<dag> Pattern = [(set ROD:$rd, (OpNode ROS:$rs, ROT:$rt))];
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `CMP_EQ_QB_R3_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `CMP_EQ_QB_R3_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 316-325
```tablegen
class PRECR_SRA_PH_W_DESC_BASE<string instr_asm, SDPatternOperator OpNode,
                               RegisterOperand ROT,
                               RegisterOperand ROS = ROT> {
  dag OutOperandList = (outs ROT:$rt);
  dag InOperandList = (ins ROS:$rs, uimm5:$sa, ROS:$src);
  string AsmString = !strconcat(instr_asm, "\t$rt, $rs, $sa");
  list<dag> Pattern = [(set ROT:$rt, (OpNode ROS:$src, ROS:$rs, timmZExt5:$sa))];
  string Constraints = "$src = $rt";
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `PRECR_SRA_PH_W_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PRECR_SRA_PH_W_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 327-335
```tablegen
class ABSQ_S_PH_R2_DESC_BASE<string instr_asm, SDPatternOperator OpNode,
                             RegisterOperand ROD,
                             RegisterOperand ROT = ROD> {
  dag OutOperandList = (outs ROD:$rd);
  dag InOperandList = (ins ROT:$rt);
  string AsmString = !strconcat(instr_asm, "\t$rd, $rt");
  list<dag> Pattern = [(set ROD:$rd, (OpNode ROT:$rt))];
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `ABSQ_S_PH_R2_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ABSQ_S_PH_R2_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 337-345
```tablegen
class REPL_DESC_BASE<string instr_asm, SDPatternOperator OpNode,
                     Operand ImmOp, ImmLeaf immPat,
                     RegisterOperand RO> {
  dag OutOperandList = (outs RO:$rd);
  dag InOperandList = (ins ImmOp:$imm);
  string AsmString = !strconcat(instr_asm, "\t$rd, $imm");
  list<dag> Pattern = [(set RO:$rd, (OpNode immPat:$imm))];
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `REPL_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `REPL_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 347-354
```tablegen
class SHLL_QB_R3_DESC_BASE<string instr_asm, SDPatternOperator OpNode,
                           RegisterOperand RO> {
  dag OutOperandList = (outs RO:$rd);
  dag InOperandList =  (ins RO:$rt, GPR32Opnd:$rs_sa);
  string AsmString = !strconcat(instr_asm, "\t$rd, $rt, $rs_sa");
  list<dag> Pattern = [(set RO:$rd, (OpNode RO:$rt, GPR32Opnd:$rs_sa))];
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `SHLL_QB_R3_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHLL_QB_R3_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 356-365
```tablegen
class SHLL_QB_R2_DESC_BASE<string instr_asm, SDPatternOperator OpNode,
                           SDPatternOperator ImmPat,
                           RegisterOperand RO, Operand ImmOpnd> {
  dag OutOperandList = (outs RO:$rd);
  dag InOperandList = (ins RO:$rt, ImmOpnd:$rs_sa);
  string AsmString = !strconcat(instr_asm, "\t$rd, $rt, $rs_sa");
  list<dag> Pattern = [(set RO:$rd, (OpNode RO:$rt, ImmPat:$rs_sa))];
  bit hasSideEffects = 1;
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `SHLL_QB_R2_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHLL_QB_R2_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 367-374
```tablegen
class LX_DESC_BASE<string instr_asm, SDPatternOperator OpNode> {
  dag OutOperandList = (outs GPR32Opnd:$rd);
  dag InOperandList = (ins PtrRC:$base, PtrRC:$index);
  string AsmString = !strconcat(instr_asm, "\t$rd, ${index}(${base})");
  list<dag> Pattern = [(set GPR32Opnd:$rd, (OpNode iPTR:$base, iPTR:$index))];
  bit mayLoad = 1;
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `LX_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `LX_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 376-384
```tablegen
class ADDUH_QB_DESC_BASE<string instr_asm, SDPatternOperator OpNode,
                         RegisterOperand ROD,
                         RegisterOperand ROS = ROD, RegisterOperand ROT = ROD> {
  dag OutOperandList = (outs ROD:$rd);
  dag InOperandList = (ins ROS:$rs, ROT:$rt);
  string AsmString = !strconcat(instr_asm, "\t$rd, $rs, $rt");
  list<dag> Pattern = [(set ROD:$rd, (OpNode ROS:$rs, ROT:$rt))];
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `ADDUH_QB_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ADDUH_QB_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 386-395
```tablegen
class APPEND_DESC_BASE<string instr_asm, SDPatternOperator OpNode,
                       Operand ImmOp, SDPatternOperator Imm> {
  dag OutOperandList = (outs GPR32Opnd:$rt);
  dag InOperandList = (ins GPR32Opnd:$rs, ImmOp:$sa, GPR32Opnd:$src);
  string AsmString = !strconcat(instr_asm, "\t$rt, $rs, $sa");
  list<dag> Pattern =  [(set GPR32Opnd:$rt,
                        (OpNode GPR32Opnd:$src, GPR32Opnd:$rs, Imm:$sa))];
  string Constraints = "$src = $rt";
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `APPEND_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `APPEND_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 397-402
```tablegen
class EXTR_W_TY1_R2_DESC_BASE<string instr_asm> {
  dag OutOperandList = (outs GPR32Opnd:$rt);
  dag InOperandList = (ins ACC64DSPOpnd:$ac, GPR32Opnd:$shift_rs);
  string AsmString = !strconcat(instr_asm, "\t$rt, $ac, $shift_rs");
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `EXTR_W_TY1_R2_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `EXTR_W_TY1_R2_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 404-409
```tablegen
class EXTR_W_TY1_R1_DESC_BASE<string instr_asm> {
  dag OutOperandList = (outs GPR32Opnd:$rt);
  dag InOperandList = (ins ACC64DSPOpnd:$ac, uimm5:$shift_rs);
  string AsmString = !strconcat(instr_asm, "\t$rt, $ac, $shift_rs");
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `EXTR_W_TY1_R1_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `EXTR_W_TY1_R1_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 411-419
```tablegen
class SHILO_R1_DESC_BASE<string instr_asm, SDPatternOperator OpNode> {
  dag OutOperandList = (outs ACC64DSPOpnd:$ac);
  dag InOperandList = (ins simm6:$shift, ACC64DSPOpnd:$acin);
  string AsmString = !strconcat(instr_asm, "\t$ac, $shift");
  list<dag> Pattern = [(set ACC64DSPOpnd:$ac,
                        (OpNode immSExt6:$shift, ACC64DSPOpnd:$acin))];
  string Constraints = "$acin = $ac";
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `SHILO_R1_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHILO_R1_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 421-429
```tablegen
class SHILO_R2_DESC_BASE<string instr_asm, SDPatternOperator OpNode> {
  dag OutOperandList = (outs ACC64DSPOpnd:$ac);
  dag InOperandList = (ins GPR32Opnd:$rs, ACC64DSPOpnd:$acin);
  string AsmString = !strconcat(instr_asm, "\t$ac, $rs");
  list<dag> Pattern = [(set ACC64DSPOpnd:$ac,
                        (OpNode GPR32Opnd:$rs, ACC64DSPOpnd:$acin))];
  string Constraints = "$acin = $ac";
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `SHILO_R2_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHILO_R2_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 431-439
```tablegen
class MTHLIP_DESC_BASE<string instr_asm, SDPatternOperator OpNode> {
  dag OutOperandList = (outs ACC64DSPOpnd:$ac);
  dag InOperandList = (ins GPR32Opnd:$rs, ACC64DSPOpnd:$acin);
  string AsmString = !strconcat(instr_asm, "\t$rs, $ac");
  list<dag> Pattern = [(set ACC64DSPOpnd:$ac,
                        (OpNode GPR32Opnd:$rs, ACC64DSPOpnd:$acin))];
  string Constraints = "$acin = $ac";
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `MTHLIP_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MTHLIP_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 441-447
```tablegen
class RDDSP_DESC_BASE<string instr_asm, SDPatternOperator OpNode> {
  dag OutOperandList = (outs GPR32Opnd:$rd);
  dag InOperandList = (ins uimm10:$mask);
  string AsmString = !strconcat(instr_asm, "\t$rd, $mask");
  list<dag> Pattern = [(set GPR32Opnd:$rd, (OpNode timmZExt10:$mask))];
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `RDDSP_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `RDDSP_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 449-455
```tablegen
class WRDSP_DESC_BASE<string instr_asm, SDPatternOperator OpNode> {
  dag OutOperandList = (outs);
  dag InOperandList = (ins GPR32Opnd:$rs, uimm10:$mask);
  string AsmString = !strconcat(instr_asm, "\t$rs, $mask");
  list<dag> Pattern = [(OpNode GPR32Opnd:$rs, timmZExt10:$mask)];
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `WRDSP_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `WRDSP_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 457-465
```tablegen
class DPA_DESC_BASE<string instr_asm, SDPatternOperator OpNode, ValueType VT> {
  dag OutOperandList = (outs ACC64DSPOpnd:$ac);
  dag InOperandList = (ins DSPROpnd:$rs, DSPROpnd:$rt, ACC64DSPOpnd:$acin);
  string AsmString = !strconcat(instr_asm, "\t$ac, $rs, $rt");
  list<dag> Pattern = [(set ACC64DSPOpnd:$ac,
                        (OpNode VT:$rs, VT:$rt, ACC64DSPOpnd:$acin))];
  string Constraints = "$acin = $ac";
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `DPA_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DPA_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 467-474
```tablegen
class MULT_DESC_BASE<string instr_asm, SDPatternOperator OpNode> {
  dag OutOperandList = (outs ACC64DSPOpnd:$ac);
  dag InOperandList = (ins GPR32Opnd:$rs, GPR32Opnd:$rt);
  string AsmString = !strconcat(instr_asm, "\t$ac, $rs, $rt");
  list<dag> Pattern = [(set ACC64DSPOpnd:$ac, (OpNode GPR32Opnd:$rs, GPR32Opnd:$rt))];
  bit isCommutable = 1;
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `MULT_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MULT_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 476-484
```tablegen
class MADD_DESC_BASE<string instr_asm, SDPatternOperator OpNode> {
  dag OutOperandList = (outs ACC64DSPOpnd:$ac);
  dag InOperandList = (ins GPR32Opnd:$rs, GPR32Opnd:$rt, ACC64DSPOpnd:$acin);
  string AsmString = !strconcat(instr_asm, "\t$ac, $rs, $rt");
  list<dag> Pattern = [(set ACC64DSPOpnd:$ac,
                        (OpNode GPR32Opnd:$rs, GPR32Opnd:$rt, ACC64DSPOpnd:$acin))];
  string Constraints = "$acin = $ac";
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `MADD_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MADD_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 486-493
```tablegen
class MFHI_DESC_BASE<string instr_asm, RegisterOperand RO, SDNode OpNode> {
  dag OutOperandList = (outs GPR32Opnd:$rd);
  dag InOperandList = (ins RO:$ac);
  string AsmString = !strconcat(instr_asm, "\t$rd, $ac");
  list<dag> Pattern = [(set GPR32Opnd:$rd, (OpNode RO:$ac))];
  string BaseOpcode = instr_asm;
  bit isMoveReg = 1;
}
```
- EN: Declares reusable TableGen class `MFHI_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MFHI_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 495-501
```tablegen
class MTHI_DESC_BASE<string instr_asm, RegisterOperand RO> {
  dag OutOperandList = (outs RO:$ac);
  dag InOperandList = (ins GPR32Opnd:$rs);
  string AsmString = !strconcat(instr_asm, "\t$rs, $ac");
  string BaseOpcode = instr_asm;
  bit isMoveReg = 1;
}
```
- EN: Declares reusable TableGen class `MTHI_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MTHI_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 503-507
```tablegen
class BPOSGE32_PSEUDO_DESC_BASE<SDPatternOperator OpNode> :
  MipsPseudo<(outs GPR32Opnd:$dst), (ins), [(set GPR32Opnd:$dst, (OpNode))]> {
  bit hasNoSchedulingInfo = 1;
  bit usesCustomInserter = 1;
}
```
- EN: Declares reusable TableGen class `BPOSGE32_PSEUDO_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `BPOSGE32_PSEUDO_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 509-517
```tablegen
class BPOSGE32_DESC_BASE<string instr_asm, DAGOperand opnd> {
  dag OutOperandList = (outs);
  dag InOperandList = (ins opnd:$offset);
  string AsmString = !strconcat(instr_asm, "\t$offset");
  bit isBranch = 1;
  bit isTerminator = 1;
  bit hasDelaySlot = 1;
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `BPOSGE32_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `BPOSGE32_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 519-526
```tablegen
class INSV_DESC_BASE<string instr_asm, SDPatternOperator OpNode> {
  dag OutOperandList = (outs GPR32Opnd:$rt);
  dag InOperandList = (ins GPR32Opnd:$src, GPR32Opnd:$rs);
  string AsmString = !strconcat(instr_asm, "\t$rt, $rs");
  list<dag> Pattern = [(set GPR32Opnd:$rt, (OpNode GPR32Opnd:$src, GPR32Opnd:$rs))];
  string Constraints = "$src = $rt";
  string BaseOpcode = instr_asm;
}
```
- EN: Declares reusable TableGen class `INSV_DESC_BASE` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `INSV_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 528-530
```tablegen
//===----------------------------------------------------------------------===//
// MIPS DSP Rev 1
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 532-535
```tablegen
// Addition/subtraction
class ADDU_QB_DESC : ADDU_QB_DESC_BASE<"addu.qb", null_frag,
                                       DSPROpnd, DSPROpnd>, IsCommutable,
                     Defs<[DSPOutFlag20]>;
```
- EN: Declares reusable TableGen class `ADDU_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ADDU_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 537-539
```tablegen
class ADDU_S_QB_DESC : ADDU_QB_DESC_BASE<"addu_s.qb", int_mips_addu_s_qb,
                                         DSPROpnd, DSPROpnd>,
                       IsCommutable, Defs<[DSPOutFlag20]>;
```
- EN: Declares reusable TableGen class `ADDU_S_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ADDU_S_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 541-543
```tablegen
class SUBU_QB_DESC : ADDU_QB_DESC_BASE<"subu.qb", null_frag,
                                       DSPROpnd, DSPROpnd>,
                     Defs<[DSPOutFlag20]>;
```
- EN: Declares reusable TableGen class `SUBU_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SUBU_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 545-547
```tablegen
class SUBU_S_QB_DESC : ADDU_QB_DESC_BASE<"subu_s.qb", int_mips_subu_s_qb,
                                         DSPROpnd, DSPROpnd>,
                       Defs<[DSPOutFlag20]>;
```
- EN: Declares reusable TableGen class `SUBU_S_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SUBU_S_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 549-551
```tablegen
class ADDQ_PH_DESC : ADDU_QB_DESC_BASE<"addq.ph", null_frag,
                                       DSPROpnd, DSPROpnd>, IsCommutable,
                     Defs<[DSPOutFlag20]>;
```
- EN: Declares reusable TableGen class `ADDQ_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ADDQ_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 553-555
```tablegen
class ADDQ_S_PH_DESC : ADDU_QB_DESC_BASE<"addq_s.ph", int_mips_addq_s_ph,
                                         DSPROpnd, DSPROpnd>,
                       IsCommutable, Defs<[DSPOutFlag20]>;
```
- EN: Declares reusable TableGen class `ADDQ_S_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ADDQ_S_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 557-559
```tablegen
class SUBQ_PH_DESC : ADDU_QB_DESC_BASE<"subq.ph", null_frag,
                                       DSPROpnd, DSPROpnd>,
                     Defs<[DSPOutFlag20]>;
```
- EN: Declares reusable TableGen class `SUBQ_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SUBQ_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 561-563
```tablegen
class SUBQ_S_PH_DESC : ADDU_QB_DESC_BASE<"subq_s.ph", int_mips_subq_s_ph,
                                         DSPROpnd, DSPROpnd>,
                       Defs<[DSPOutFlag20]>;
```
- EN: Declares reusable TableGen class `SUBQ_S_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SUBQ_S_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 565-567
```tablegen
class ADDQ_S_W_DESC : ADDU_QB_DESC_BASE<"addq_s.w", int_mips_addq_s_w,
                                        GPR32Opnd, GPR32Opnd>,
                      IsCommutable, Defs<[DSPOutFlag20]>;
```
- EN: Declares reusable TableGen class `ADDQ_S_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ADDQ_S_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 569-571
```tablegen
class SUBQ_S_W_DESC : ADDU_QB_DESC_BASE<"subq_s.w", int_mips_subq_s_w,
                                        GPR32Opnd, GPR32Opnd>,
                      Defs<[DSPOutFlag20]>;
```
- EN: Declares reusable TableGen class `SUBQ_S_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SUBQ_S_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 573-575
```tablegen
class ADDSC_DESC : ADDU_QB_DESC_BASE<"addsc", null_frag,
                                     GPR32Opnd, GPR32Opnd>, IsCommutable,
                   Defs<[DSPCarry]>;
```
- EN: Declares reusable TableGen class `ADDSC_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ADDSC_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 577-579
```tablegen
class ADDWC_DESC : ADDU_QB_DESC_BASE<"addwc", null_frag,
                                     GPR32Opnd, GPR32Opnd>,
                   IsCommutable, Uses<[DSPCarry]>, Defs<[DSPOutFlag20]>;
```
- EN: Declares reusable TableGen class `ADDWC_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ADDWC_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 581-582
```tablegen
class MODSUB_DESC : ADDU_QB_DESC_BASE<"modsub", int_mips_modsub,
                                      GPR32Opnd, GPR32Opnd>;
```
- EN: Declares reusable TableGen class `MODSUB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MODSUB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 584-585
```tablegen
class RADDU_W_QB_DESC : RADDU_W_QB_DESC_BASE<"raddu.w.qb", int_mips_raddu_w_qb,
                                             GPR32Opnd, DSPROpnd>;
```
- EN: Declares reusable TableGen class `RADDU_W_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `RADDU_W_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 587-590
```tablegen
// Absolute value
class ABSQ_S_PH_DESC : ABSQ_S_PH_R2_DESC_BASE<"absq_s.ph", int_mips_absq_s_ph,
                                              DSPROpnd>,
                       Defs<[DSPOutFlag20]>;
```
- EN: Declares reusable TableGen class `ABSQ_S_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ABSQ_S_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 592-594
```tablegen
class ABSQ_S_W_DESC : ABSQ_S_PH_R2_DESC_BASE<"absq_s.w", int_mips_absq_s_w,
                                             GPR32Opnd>,
                      Defs<[DSPOutFlag20]>;
```
- EN: Declares reusable TableGen class `ABSQ_S_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ABSQ_S_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 596-599
```tablegen
// Precision reduce/expand
class PRECRQ_QB_PH_DESC : CMP_EQ_QB_R3_DESC_BASE<"precrq.qb.ph",
                                                 int_mips_precrq_qb_ph,
                                                 DSPROpnd, DSPROpnd>;
```
- EN: Declares reusable TableGen class `PRECRQ_QB_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PRECRQ_QB_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 601-603
```tablegen
class PRECRQ_PH_W_DESC : CMP_EQ_QB_R3_DESC_BASE<"precrq.ph.w",
                                                int_mips_precrq_ph_w,
                                                DSPROpnd, GPR32Opnd>;
```
- EN: Declares reusable TableGen class `PRECRQ_PH_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PRECRQ_PH_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 605-609
```tablegen
class PRECRQ_RS_PH_W_DESC : CMP_EQ_QB_R3_DESC_BASE<"precrq_rs.ph.w",
                                                   int_mips_precrq_rs_ph_w,
                                                   DSPROpnd,
                                                   GPR32Opnd>,
                            Defs<[DSPOutFlag22]>;
```
- EN: Declares reusable TableGen class `PRECRQ_RS_PH_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PRECRQ_RS_PH_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 611-615
```tablegen
class PRECRQU_S_QB_PH_DESC : CMP_EQ_QB_R3_DESC_BASE<"precrqu_s.qb.ph",
                                                    int_mips_precrqu_s_qb_ph,
                                                    DSPROpnd,
                                                    DSPROpnd>,
                             Defs<[DSPOutFlag22]>;
```
- EN: Declares reusable TableGen class `PRECRQU_S_QB_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PRECRQU_S_QB_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 617-619
```tablegen
class PRECEQ_W_PHL_DESC : ABSQ_S_PH_R2_DESC_BASE<"preceq.w.phl",
                                                 int_mips_preceq_w_phl,
                                                 GPR32Opnd, DSPROpnd>;
```
- EN: Declares reusable TableGen class `PRECEQ_W_PHL_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PRECEQ_W_PHL_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 621-623
```tablegen
class PRECEQ_W_PHR_DESC : ABSQ_S_PH_R2_DESC_BASE<"preceq.w.phr",
                                                 int_mips_preceq_w_phr,
                                                 GPR32Opnd, DSPROpnd>;
```
- EN: Declares reusable TableGen class `PRECEQ_W_PHR_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PRECEQ_W_PHR_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 625-627
```tablegen
class PRECEQU_PH_QBL_DESC : ABSQ_S_PH_R2_DESC_BASE<"precequ.ph.qbl",
                                                   int_mips_precequ_ph_qbl,
                                                   DSPROpnd>;
```
- EN: Declares reusable TableGen class `PRECEQU_PH_QBL_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PRECEQU_PH_QBL_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 629-631
```tablegen
class PRECEQU_PH_QBR_DESC : ABSQ_S_PH_R2_DESC_BASE<"precequ.ph.qbr",
                                                   int_mips_precequ_ph_qbr,
                                                   DSPROpnd>;
```
- EN: Declares reusable TableGen class `PRECEQU_PH_QBR_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PRECEQU_PH_QBR_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 633-635
```tablegen
class PRECEQU_PH_QBLA_DESC : ABSQ_S_PH_R2_DESC_BASE<"precequ.ph.qbla",
                                                    int_mips_precequ_ph_qbla,
                                                    DSPROpnd>;
```
- EN: Declares reusable TableGen class `PRECEQU_PH_QBLA_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PRECEQU_PH_QBLA_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 637-639
```tablegen
class PRECEQU_PH_QBRA_DESC : ABSQ_S_PH_R2_DESC_BASE<"precequ.ph.qbra",
                                                    int_mips_precequ_ph_qbra,
                                                    DSPROpnd>;
```
- EN: Declares reusable TableGen class `PRECEQU_PH_QBRA_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PRECEQU_PH_QBRA_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 641-643
```tablegen
class PRECEU_PH_QBL_DESC : ABSQ_S_PH_R2_DESC_BASE<"preceu.ph.qbl",
                                                  int_mips_preceu_ph_qbl,
                                                  DSPROpnd>;
```
- EN: Declares reusable TableGen class `PRECEU_PH_QBL_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PRECEU_PH_QBL_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 645-647
```tablegen
class PRECEU_PH_QBR_DESC : ABSQ_S_PH_R2_DESC_BASE<"preceu.ph.qbr",
                                                  int_mips_preceu_ph_qbr,
                                                  DSPROpnd>;
```
- EN: Declares reusable TableGen class `PRECEU_PH_QBR_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PRECEU_PH_QBR_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 649-651
```tablegen
class PRECEU_PH_QBLA_DESC : ABSQ_S_PH_R2_DESC_BASE<"preceu.ph.qbla",
                                                   int_mips_preceu_ph_qbla,
                                                   DSPROpnd>;
```
- EN: Declares reusable TableGen class `PRECEU_PH_QBLA_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PRECEU_PH_QBLA_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 653-655
```tablegen
class PRECEU_PH_QBRA_DESC : ABSQ_S_PH_R2_DESC_BASE<"preceu.ph.qbra",
                                                   int_mips_preceu_ph_qbra,
                                                   DSPROpnd>;
```
- EN: Declares reusable TableGen class `PRECEU_PH_QBRA_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PRECEU_PH_QBRA_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 657-660
```tablegen
// Shift
class SHLL_QB_DESC : SHLL_QB_R2_DESC_BASE<"shll.qb", null_frag, immZExt3,
                                          DSPROpnd, uimm3>,
                     Defs<[DSPOutFlag22]>;
```
- EN: Declares reusable TableGen class `SHLL_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHLL_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 662-664
```tablegen
class SHLLV_QB_DESC : SHLL_QB_R3_DESC_BASE<"shllv.qb", int_mips_shll_qb,
                                           DSPROpnd>,
                      Defs<[DSPOutFlag22]>;
```
- EN: Declares reusable TableGen class `SHLLV_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHLLV_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 666-667
```tablegen
class SHRL_QB_DESC : SHLL_QB_R2_DESC_BASE<"shrl.qb", null_frag, immZExt3,
                                          DSPROpnd, uimm3>;
```
- EN: Declares reusable TableGen class `SHRL_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHRL_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 669-670
```tablegen
class SHRLV_QB_DESC : SHLL_QB_R3_DESC_BASE<"shrlv.qb", int_mips_shrl_qb,
                                           DSPROpnd>;
```
- EN: Declares reusable TableGen class `SHRLV_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHRLV_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 672-674
```tablegen
class SHLL_PH_DESC : SHLL_QB_R2_DESC_BASE<"shll.ph", null_frag, immZExt4,
                                          DSPROpnd, uimm4>,
                     Defs<[DSPOutFlag22]>;
```
- EN: Declares reusable TableGen class `SHLL_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHLL_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 676-678
```tablegen
class SHLLV_PH_DESC : SHLL_QB_R3_DESC_BASE<"shllv.ph", int_mips_shll_ph,
                                           DSPROpnd>,
                      Defs<[DSPOutFlag22]>;
```
- EN: Declares reusable TableGen class `SHLLV_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHLLV_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 680-683
```tablegen
class SHLL_S_PH_DESC : SHLL_QB_R2_DESC_BASE<"shll_s.ph", int_mips_shll_s_ph,
                                            immZExt4, DSPROpnd,
                                            uimm4>,
                       Defs<[DSPOutFlag22]>;
```
- EN: Declares reusable TableGen class `SHLL_S_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHLL_S_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 685-687
```tablegen
class SHLLV_S_PH_DESC : SHLL_QB_R3_DESC_BASE<"shllv_s.ph", int_mips_shll_s_ph,
                                             DSPROpnd>,
                        Defs<[DSPOutFlag22]>;
```
- EN: Declares reusable TableGen class `SHLLV_S_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHLLV_S_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 689-690
```tablegen
class SHRA_PH_DESC : SHLL_QB_R2_DESC_BASE<"shra.ph", null_frag, immZExt4,
                                          DSPROpnd, uimm4>;
```
- EN: Declares reusable TableGen class `SHRA_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHRA_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 692-693
```tablegen
class SHRAV_PH_DESC : SHLL_QB_R3_DESC_BASE<"shrav.ph", int_mips_shra_ph,
                                           DSPROpnd>;
```
- EN: Declares reusable TableGen class `SHRAV_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHRAV_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 695-697
```tablegen
class SHRA_R_PH_DESC : SHLL_QB_R2_DESC_BASE<"shra_r.ph", int_mips_shra_r_ph,
                                            immZExt4, DSPROpnd,
                                            uimm4>;
```
- EN: Declares reusable TableGen class `SHRA_R_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHRA_R_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 699-700
```tablegen
class SHRAV_R_PH_DESC : SHLL_QB_R3_DESC_BASE<"shrav_r.ph", int_mips_shra_r_ph,
                                             DSPROpnd>;
```
- EN: Declares reusable TableGen class `SHRAV_R_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHRAV_R_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 702-705
```tablegen
class SHLL_S_W_DESC : SHLL_QB_R2_DESC_BASE<"shll_s.w", int_mips_shll_s_w,
                                           immZExt5, GPR32Opnd,
                                           uimm5>,
                      Defs<[DSPOutFlag22]>;
```
- EN: Declares reusable TableGen class `SHLL_S_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHLL_S_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 707-709
```tablegen
class SHLLV_S_W_DESC : SHLL_QB_R3_DESC_BASE<"shllv_s.w", int_mips_shll_s_w,
                                            GPR32Opnd>,
                       Defs<[DSPOutFlag22]>;
```
- EN: Declares reusable TableGen class `SHLLV_S_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHLLV_S_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 711-713
```tablegen
class SHRA_R_W_DESC : SHLL_QB_R2_DESC_BASE<"shra_r.w", int_mips_shra_r_w,
                                           immZExt5, GPR32Opnd,
                                           uimm5>;
```
- EN: Declares reusable TableGen class `SHRA_R_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHRA_R_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 715-716
```tablegen
class SHRAV_R_W_DESC : SHLL_QB_R3_DESC_BASE<"shrav_r.w", int_mips_shra_r_w,
                                            GPR32Opnd>;
```
- EN: Declares reusable TableGen class `SHRAV_R_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHRAV_R_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 718-722
```tablegen
// Multiplication
class MULEU_S_PH_QBL_DESC : ADDU_QB_DESC_BASE<"muleu_s.ph.qbl",
                                              int_mips_muleu_s_ph_qbl,
                                              DSPROpnd, DSPROpnd>,
                            Defs<[DSPOutFlag21]>;
```
- EN: Declares reusable TableGen class `MULEU_S_PH_QBL_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MULEU_S_PH_QBL_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 724-727
```tablegen
class MULEU_S_PH_QBR_DESC : ADDU_QB_DESC_BASE<"muleu_s.ph.qbr",
                                              int_mips_muleu_s_ph_qbr,
                                              DSPROpnd, DSPROpnd>,
                            Defs<[DSPOutFlag21]>;
```
- EN: Declares reusable TableGen class `MULEU_S_PH_QBR_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MULEU_S_PH_QBR_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 729-732
```tablegen
class MULEQ_S_W_PHL_DESC : ADDU_QB_DESC_BASE<"muleq_s.w.phl",
                                             int_mips_muleq_s_w_phl,
                                             GPR32Opnd, DSPROpnd>,
                           IsCommutable, Defs<[DSPOutFlag21]>;
```
- EN: Declares reusable TableGen class `MULEQ_S_W_PHL_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MULEQ_S_W_PHL_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 734-737
```tablegen
class MULEQ_S_W_PHR_DESC : ADDU_QB_DESC_BASE<"muleq_s.w.phr",
                                             int_mips_muleq_s_w_phr,
                                             GPR32Opnd, DSPROpnd>,
                           IsCommutable, Defs<[DSPOutFlag21]>;
```
- EN: Declares reusable TableGen class `MULEQ_S_W_PHR_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MULEQ_S_W_PHR_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 739-741
```tablegen
class MULQ_RS_PH_DESC : ADDU_QB_DESC_BASE<"mulq_rs.ph", int_mips_mulq_rs_ph,
                                          DSPROpnd, DSPROpnd>,
                        IsCommutable, Defs<[DSPOutFlag21]>;
```
- EN: Declares reusable TableGen class `MULQ_RS_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MULQ_RS_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 743-745
```tablegen
class MULSAQ_S_W_PH_DESC : DPA_DESC_BASE<"mulsaq_s.w.ph",
                                         MipsMULSAQ_S_W_PH, v2i16>,
                           Defs<[DSPOutFlag16_19]>;
```
- EN: Declares reusable TableGen class `MULSAQ_S_W_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MULSAQ_S_W_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 747-748
```tablegen
class MAQ_S_W_PHL_DESC : DPA_DESC_BASE<"maq_s.w.phl", MipsMAQ_S_W_PHL, v2i16>,
                         Defs<[DSPOutFlag16_19]>;
```
- EN: Declares reusable TableGen class `MAQ_S_W_PHL_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MAQ_S_W_PHL_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 750-751
```tablegen
class MAQ_S_W_PHR_DESC : DPA_DESC_BASE<"maq_s.w.phr", MipsMAQ_S_W_PHR, v2i16>,
                         Defs<[DSPOutFlag16_19]>;
```
- EN: Declares reusable TableGen class `MAQ_S_W_PHR_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MAQ_S_W_PHR_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 753-754
```tablegen
class MAQ_SA_W_PHL_DESC : DPA_DESC_BASE<"maq_sa.w.phl", MipsMAQ_SA_W_PHL, v2i16>,
                          Defs<[DSPOutFlag16_19]>;
```
- EN: Declares reusable TableGen class `MAQ_SA_W_PHL_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MAQ_SA_W_PHL_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 756-757
```tablegen
class MAQ_SA_W_PHR_DESC : DPA_DESC_BASE<"maq_sa.w.phr", MipsMAQ_SA_W_PHR, v2i16>,
                          Defs<[DSPOutFlag16_19]>;
```
- EN: Declares reusable TableGen class `MAQ_SA_W_PHR_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MAQ_SA_W_PHR_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 759-763
```tablegen
// Move from/to hi/lo.
class MFHI_DESC : MFHI_DESC_BASE<"mfhi", ACC64DSPOpnd, MipsMFHI>;
class MFLO_DESC : MFHI_DESC_BASE<"mflo", ACC64DSPOpnd, MipsMFLO>;
class MTHI_DESC : MTHI_DESC_BASE<"mthi", HI32DSPOpnd>;
class MTLO_DESC : MTHI_DESC_BASE<"mtlo", LO32DSPOpnd>;
```
- EN: Declares reusable TableGen class `MFHI_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MFHI_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 765-766
```tablegen
// Dot product with accumulate/subtract
class DPAU_H_QBL_DESC : DPA_DESC_BASE<"dpau.h.qbl", MipsDPAU_H_QBL, v4i8>;
```
- EN: Declares reusable TableGen class `DPAU_H_QBL_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DPAU_H_QBL_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 768-768
```tablegen
class DPAU_H_QBR_DESC : DPA_DESC_BASE<"dpau.h.qbr", MipsDPAU_H_QBR, v4i8>;
```
- EN: Declares reusable TableGen class `DPAU_H_QBR_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DPAU_H_QBR_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 770-770
```tablegen
class DPSU_H_QBL_DESC : DPA_DESC_BASE<"dpsu.h.qbl", MipsDPSU_H_QBL, v4i8>;
```
- EN: Declares reusable TableGen class `DPSU_H_QBL_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DPSU_H_QBL_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 772-772
```tablegen
class DPSU_H_QBR_DESC : DPA_DESC_BASE<"dpsu.h.qbr", MipsDPSU_H_QBR, v4i8>;
```
- EN: Declares reusable TableGen class `DPSU_H_QBR_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DPSU_H_QBR_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 774-775
```tablegen
class DPAQ_S_W_PH_DESC : DPA_DESC_BASE<"dpaq_s.w.ph", MipsDPAQ_S_W_PH, v2i16>,
                         Defs<[DSPOutFlag16_19]>;
```
- EN: Declares reusable TableGen class `DPAQ_S_W_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DPAQ_S_W_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 777-778
```tablegen
class DPSQ_S_W_PH_DESC : DPA_DESC_BASE<"dpsq_s.w.ph", MipsDPSQ_S_W_PH, v2i16>,
                         Defs<[DSPOutFlag16_19]>;
```
- EN: Declares reusable TableGen class `DPSQ_S_W_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DPSQ_S_W_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 780-781
```tablegen
class DPAQ_SA_L_W_DESC : DPA_DESC_BASE<"dpaq_sa.l.w", MipsDPAQ_SA_L_W, i32>,
                         Defs<[DSPOutFlag16_19]>;
```
- EN: Declares reusable TableGen class `DPAQ_SA_L_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DPAQ_SA_L_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 783-784
```tablegen
class DPSQ_SA_L_W_DESC : DPA_DESC_BASE<"dpsq_sa.l.w", MipsDPSQ_SA_L_W, i32>,
                         Defs<[DSPOutFlag16_19]>;
```
- EN: Declares reusable TableGen class `DPSQ_SA_L_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DPSQ_SA_L_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 786-791
```tablegen
class MULT_DSP_DESC  : MULT_DESC_BASE<"mult", MipsMult>;
class MULTU_DSP_DESC : MULT_DESC_BASE<"multu", MipsMultu>;
class MADD_DSP_DESC  : MADD_DESC_BASE<"madd", MipsMAdd>;
class MADDU_DSP_DESC : MADD_DESC_BASE<"maddu", MipsMAddu>;
class MSUB_DSP_DESC  : MADD_DESC_BASE<"msub", MipsMSub>;
class MSUBU_DSP_DESC : MADD_DESC_BASE<"msubu", MipsMSubu>;
```
- EN: Declares reusable TableGen class `MULT_DSP_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MULT_DSP_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 793-797
```tablegen
// Comparison
class CMPU_EQ_QB_DESC : CMP_EQ_QB_R2_DESC_BASE<"cmpu.eq.qb",
                                               int_mips_cmpu_eq_qb,
                                               DSPROpnd>,
                        IsCommutable, Defs<[DSPCCond]>;
```
- EN: Declares reusable TableGen class `CMPU_EQ_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `CMPU_EQ_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 799-801
```tablegen
class CMPU_LT_QB_DESC : CMP_EQ_QB_R2_DESC_BASE<"cmpu.lt.qb",
                                               int_mips_cmpu_lt_qb,
                                               DSPROpnd>, Defs<[DSPCCond]>;
```
- EN: Declares reusable TableGen class `CMPU_LT_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `CMPU_LT_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 803-805
```tablegen
class CMPU_LE_QB_DESC : CMP_EQ_QB_R2_DESC_BASE<"cmpu.le.qb",
                                               int_mips_cmpu_le_qb,
                                               DSPROpnd>, Defs<[DSPCCond]>;
```
- EN: Declares reusable TableGen class `CMPU_LE_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `CMPU_LE_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 807-810
```tablegen
class CMPGU_EQ_QB_DESC : CMP_EQ_QB_R3_DESC_BASE<"cmpgu.eq.qb",
                                                int_mips_cmpgu_eq_qb,
                                                GPR32Opnd, DSPROpnd>,
                         IsCommutable;
```
- EN: Declares reusable TableGen class `CMPGU_EQ_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `CMPGU_EQ_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 812-814
```tablegen
class CMPGU_LT_QB_DESC : CMP_EQ_QB_R3_DESC_BASE<"cmpgu.lt.qb",
                                                int_mips_cmpgu_lt_qb,
                                                GPR32Opnd, DSPROpnd>;
```
- EN: Declares reusable TableGen class `CMPGU_LT_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `CMPGU_LT_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 816-818
```tablegen
class CMPGU_LE_QB_DESC : CMP_EQ_QB_R3_DESC_BASE<"cmpgu.le.qb",
                                                int_mips_cmpgu_le_qb,
                                                GPR32Opnd, DSPROpnd>;
```
- EN: Declares reusable TableGen class `CMPGU_LE_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `CMPGU_LE_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 820-822
```tablegen
class CMP_EQ_PH_DESC : CMP_EQ_QB_R2_DESC_BASE<"cmp.eq.ph", int_mips_cmp_eq_ph,
                                              DSPROpnd>,
                       IsCommutable, Defs<[DSPCCond]>;
```
- EN: Declares reusable TableGen class `CMP_EQ_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `CMP_EQ_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 824-826
```tablegen
class CMP_LT_PH_DESC : CMP_EQ_QB_R2_DESC_BASE<"cmp.lt.ph", int_mips_cmp_lt_ph,
                                              DSPROpnd>,
                       Defs<[DSPCCond]>;
```
- EN: Declares reusable TableGen class `CMP_LT_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `CMP_LT_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 828-830
```tablegen
class CMP_LE_PH_DESC : CMP_EQ_QB_R2_DESC_BASE<"cmp.le.ph", int_mips_cmp_le_ph,
                                              DSPROpnd>,
                       Defs<[DSPCCond]>;
```
- EN: Declares reusable TableGen class `CMP_LE_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `CMP_LE_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 832-834
```tablegen
// Misc
class BITREV_DESC : ABSQ_S_PH_R2_DESC_BASE<"bitrev", int_mips_bitrev,
                                           GPR32Opnd>;
```
- EN: Declares reusable TableGen class `BITREV_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `BITREV_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 836-837
```tablegen
class PACKRL_PH_DESC : CMP_EQ_QB_R3_DESC_BASE<"packrl.ph", int_mips_packrl_ph,
                                              DSPROpnd, DSPROpnd>;
```
- EN: Declares reusable TableGen class `PACKRL_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PACKRL_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 839-840
```tablegen
class REPL_QB_DESC : REPL_DESC_BASE<"repl.qb", int_mips_repl_qb, uimm8,
                                    immZExt8, DSPROpnd>;
```
- EN: Declares reusable TableGen class `REPL_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `REPL_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 842-843
```tablegen
class REPL_PH_DESC : REPL_DESC_BASE<"repl.ph", int_mips_repl_ph, simm10,
                                    immSExt10, DSPROpnd>;
```
- EN: Declares reusable TableGen class `REPL_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `REPL_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 845-846
```tablegen
class REPLV_QB_DESC : ABSQ_S_PH_R2_DESC_BASE<"replv.qb", int_mips_repl_qb,
                                             DSPROpnd, GPR32Opnd>;
```
- EN: Declares reusable TableGen class `REPLV_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `REPLV_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 848-849
```tablegen
class REPLV_PH_DESC : ABSQ_S_PH_R2_DESC_BASE<"replv.ph", int_mips_repl_ph,
                                             DSPROpnd, GPR32Opnd>;
```
- EN: Declares reusable TableGen class `REPLV_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `REPLV_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 851-853
```tablegen
class PICK_QB_DESC : CMP_EQ_QB_R3_DESC_BASE<"pick.qb", int_mips_pick_qb,
                                            DSPROpnd, DSPROpnd>,
                     Uses<[DSPCCond]>;
```
- EN: Declares reusable TableGen class `PICK_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PICK_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 855-857
```tablegen
class PICK_PH_DESC : CMP_EQ_QB_R3_DESC_BASE<"pick.ph", int_mips_pick_ph,
                                            DSPROpnd, DSPROpnd>,
                     Uses<[DSPCCond]>;
```
- EN: Declares reusable TableGen class `PICK_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PICK_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 859-859
```tablegen
class LWX_DESC : LX_DESC_BASE<"lwx", int_mips_lwx>;
```
- EN: Declares reusable TableGen class `LWX_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `LWX_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 861-861
```tablegen
class LHX_DESC : LX_DESC_BASE<"lhx", int_mips_lhx>;
```
- EN: Declares reusable TableGen class `LHX_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `LHX_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 863-863
```tablegen
class LBUX_DESC : LX_DESC_BASE<"lbux", int_mips_lbux>;
```
- EN: Declares reusable TableGen class `LBUX_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `LBUX_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 865-865
```tablegen
class BPOSGE32_DESC : BPOSGE32_DESC_BASE<"bposge32", brtarget>;
```
- EN: Declares reusable TableGen class `BPOSGE32_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `BPOSGE32_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 867-869
```tablegen
// Extr
class EXTP_DESC : EXTR_W_TY1_R1_DESC_BASE<"extp">,
                  Uses<[DSPPos]>, Defs<[DSPEFI]>;
```
- EN: Declares reusable TableGen class `EXTP_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `EXTP_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 871-872
```tablegen
class EXTPV_DESC : EXTR_W_TY1_R2_DESC_BASE<"extpv">,
                   Uses<[DSPPos]>, Defs<[DSPEFI]>;
```
- EN: Declares reusable TableGen class `EXTPV_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `EXTPV_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 874-875
```tablegen
class EXTPDP_DESC : EXTR_W_TY1_R1_DESC_BASE<"extpdp">,
                    Uses<[DSPPos]>, Defs<[DSPPos, DSPEFI]>;
```
- EN: Declares reusable TableGen class `EXTPDP_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `EXTPDP_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 877-878
```tablegen
class EXTPDPV_DESC : EXTR_W_TY1_R2_DESC_BASE<"extpdpv">,
                     Uses<[DSPPos]>, Defs<[DSPPos, DSPEFI]>;
```
- EN: Declares reusable TableGen class `EXTPDPV_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `EXTPDPV_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 880-881
```tablegen
class EXTR_W_DESC : EXTR_W_TY1_R1_DESC_BASE<"extr.w">,
                    Defs<[DSPOutFlag23]>;
```
- EN: Declares reusable TableGen class `EXTR_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `EXTR_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 883-884
```tablegen
class EXTRV_W_DESC : EXTR_W_TY1_R2_DESC_BASE<"extrv.w">,
                     Defs<[DSPOutFlag23]>;
```
- EN: Declares reusable TableGen class `EXTRV_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `EXTRV_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 886-887
```tablegen
class EXTR_R_W_DESC : EXTR_W_TY1_R1_DESC_BASE<"extr_r.w">,
                      Defs<[DSPOutFlag23]>;
```
- EN: Declares reusable TableGen class `EXTR_R_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `EXTR_R_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 889-890
```tablegen
class EXTRV_R_W_DESC : EXTR_W_TY1_R2_DESC_BASE<"extrv_r.w">,
                       Defs<[DSPOutFlag23]>;
```
- EN: Declares reusable TableGen class `EXTRV_R_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `EXTRV_R_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 892-893
```tablegen
class EXTR_RS_W_DESC : EXTR_W_TY1_R1_DESC_BASE<"extr_rs.w">,
                       Defs<[DSPOutFlag23]>;
```
- EN: Declares reusable TableGen class `EXTR_RS_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `EXTR_RS_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 895-896
```tablegen
class EXTRV_RS_W_DESC : EXTR_W_TY1_R2_DESC_BASE<"extrv_rs.w">,
                        Defs<[DSPOutFlag23]>;
```
- EN: Declares reusable TableGen class `EXTRV_RS_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `EXTRV_RS_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 898-899
```tablegen
class EXTR_S_H_DESC : EXTR_W_TY1_R1_DESC_BASE<"extr_s.h">,
                      Defs<[DSPOutFlag23]>;
```
- EN: Declares reusable TableGen class `EXTR_S_H_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `EXTR_S_H_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 901-902
```tablegen
class EXTRV_S_H_DESC : EXTR_W_TY1_R2_DESC_BASE<"extrv_s.h">,
                       Defs<[DSPOutFlag23]>;
```
- EN: Declares reusable TableGen class `EXTRV_S_H_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `EXTRV_S_H_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 904-904
```tablegen
class SHILO_DESC : SHILO_R1_DESC_BASE<"shilo", MipsSHILO>;
```
- EN: Declares reusable TableGen class `SHILO_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHILO_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 906-906
```tablegen
class SHILOV_DESC : SHILO_R2_DESC_BASE<"shilov", MipsSHILO>;
```
- EN: Declares reusable TableGen class `SHILOV_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHILOV_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 908-908
```tablegen
class MTHLIP_DESC : MTHLIP_DESC_BASE<"mthlip", MipsMTHLIP>, Defs<[DSPPos]>;
```
- EN: Declares reusable TableGen class `MTHLIP_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MTHLIP_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 910-910
```tablegen
class RDDSP_DESC : RDDSP_DESC_BASE<"rddsp", int_mips_rddsp>;
```
- EN: Declares reusable TableGen class `RDDSP_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `RDDSP_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 912-912
```tablegen
class WRDSP_DESC : WRDSP_DESC_BASE<"wrdsp", int_mips_wrdsp>;
```
- EN: Declares reusable TableGen class `WRDSP_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `WRDSP_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 914-915
```tablegen
class INSV_DESC : INSV_DESC_BASE<"insv", int_mips_insv>,
                  Uses<[DSPPos, DSPSCount]>;
```
- EN: Declares reusable TableGen class `INSV_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `INSV_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 917-922
```tablegen
//===----------------------------------------------------------------------===//
// MIPS DSP Rev 2
// Addition/subtraction
class ADDU_PH_DESC : ADDU_QB_DESC_BASE<"addu.ph", int_mips_addu_ph,
                                       DSPROpnd, DSPROpnd>, IsCommutable,
                     Defs<[DSPOutFlag20]>;
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 924-926
```tablegen
class ADDU_S_PH_DESC : ADDU_QB_DESC_BASE<"addu_s.ph", int_mips_addu_s_ph,
                                         DSPROpnd, DSPROpnd>,
                       IsCommutable, Defs<[DSPOutFlag20]>;
```
- EN: Declares reusable TableGen class `ADDU_S_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ADDU_S_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 928-930
```tablegen
class SUBU_PH_DESC : ADDU_QB_DESC_BASE<"subu.ph", int_mips_subu_ph,
                                       DSPROpnd, DSPROpnd>,
                     Defs<[DSPOutFlag20]>;
```
- EN: Declares reusable TableGen class `SUBU_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SUBU_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 932-934
```tablegen
class SUBU_S_PH_DESC : ADDU_QB_DESC_BASE<"subu_s.ph", int_mips_subu_s_ph,
                                         DSPROpnd, DSPROpnd>,
                       Defs<[DSPOutFlag20]>;
```
- EN: Declares reusable TableGen class `SUBU_S_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SUBU_S_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 936-937
```tablegen
class ADDUH_QB_DESC : ADDUH_QB_DESC_BASE<"adduh.qb", int_mips_adduh_qb,
                                         DSPROpnd>, IsCommutable;
```
- EN: Declares reusable TableGen class `ADDUH_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ADDUH_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 939-940
```tablegen
class ADDUH_R_QB_DESC : ADDUH_QB_DESC_BASE<"adduh_r.qb", int_mips_adduh_r_qb,
                                           DSPROpnd>, IsCommutable;
```
- EN: Declares reusable TableGen class `ADDUH_R_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ADDUH_R_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 942-943
```tablegen
class SUBUH_QB_DESC : ADDUH_QB_DESC_BASE<"subuh.qb", int_mips_subuh_qb,
                                         DSPROpnd>;
```
- EN: Declares reusable TableGen class `SUBUH_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SUBUH_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 945-946
```tablegen
class SUBUH_R_QB_DESC : ADDUH_QB_DESC_BASE<"subuh_r.qb", int_mips_subuh_r_qb,
                                           DSPROpnd>;
```
- EN: Declares reusable TableGen class `SUBUH_R_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SUBUH_R_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 948-949
```tablegen
class ADDQH_PH_DESC : ADDUH_QB_DESC_BASE<"addqh.ph", int_mips_addqh_ph,
                                         DSPROpnd>, IsCommutable;
```
- EN: Declares reusable TableGen class `ADDQH_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ADDQH_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 951-952
```tablegen
class ADDQH_R_PH_DESC : ADDUH_QB_DESC_BASE<"addqh_r.ph", int_mips_addqh_r_ph,
                                           DSPROpnd>, IsCommutable;
```
- EN: Declares reusable TableGen class `ADDQH_R_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ADDQH_R_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 954-955
```tablegen
class SUBQH_PH_DESC : ADDUH_QB_DESC_BASE<"subqh.ph", int_mips_subqh_ph,
                                         DSPROpnd>;
```
- EN: Declares reusable TableGen class `SUBQH_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SUBQH_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 957-958
```tablegen
class SUBQH_R_PH_DESC : ADDUH_QB_DESC_BASE<"subqh_r.ph", int_mips_subqh_r_ph,
                                           DSPROpnd>;
```
- EN: Declares reusable TableGen class `SUBQH_R_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SUBQH_R_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 960-961
```tablegen
class ADDQH_W_DESC : ADDUH_QB_DESC_BASE<"addqh.w", int_mips_addqh_w,
                                        GPR32Opnd>, IsCommutable;
```
- EN: Declares reusable TableGen class `ADDQH_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ADDQH_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 963-964
```tablegen
class ADDQH_R_W_DESC : ADDUH_QB_DESC_BASE<"addqh_r.w", int_mips_addqh_r_w,
                                          GPR32Opnd>, IsCommutable;
```
- EN: Declares reusable TableGen class `ADDQH_R_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ADDQH_R_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 966-967
```tablegen
class SUBQH_W_DESC : ADDUH_QB_DESC_BASE<"subqh.w", int_mips_subqh_w,
                                        GPR32Opnd>;
```
- EN: Declares reusable TableGen class `SUBQH_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SUBQH_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 969-970
```tablegen
class SUBQH_R_W_DESC : ADDUH_QB_DESC_BASE<"subqh_r.w", int_mips_subqh_r_w,
                                          GPR32Opnd>;
```
- EN: Declares reusable TableGen class `SUBQH_R_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SUBQH_R_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 972-976
```tablegen
// Comparison
class CMPGDU_EQ_QB_DESC : CMP_EQ_QB_R3_DESC_BASE<"cmpgdu.eq.qb",
                                                 int_mips_cmpgdu_eq_qb,
                                                 GPR32Opnd, DSPROpnd>,
                          IsCommutable, Defs<[DSPCCond]>;
```
- EN: Declares reusable TableGen class `CMPGDU_EQ_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `CMPGDU_EQ_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 978-981
```tablegen
class CMPGDU_LT_QB_DESC : CMP_EQ_QB_R3_DESC_BASE<"cmpgdu.lt.qb",
                                                 int_mips_cmpgdu_lt_qb,
                                                 GPR32Opnd, DSPROpnd>,
                          Defs<[DSPCCond]>;
```
- EN: Declares reusable TableGen class `CMPGDU_LT_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `CMPGDU_LT_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 983-986
```tablegen
class CMPGDU_LE_QB_DESC : CMP_EQ_QB_R3_DESC_BASE<"cmpgdu.le.qb",
                                                 int_mips_cmpgdu_le_qb,
                                                 GPR32Opnd, DSPROpnd>,
                          Defs<[DSPCCond]>;
```
- EN: Declares reusable TableGen class `CMPGDU_LE_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `CMPGDU_LE_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 988-991
```tablegen
// Absolute
class ABSQ_S_QB_DESC : ABSQ_S_PH_R2_DESC_BASE<"absq_s.qb", int_mips_absq_s_qb,
                                              DSPROpnd>,
                       Defs<[DSPOutFlag20]>;
```
- EN: Declares reusable TableGen class `ABSQ_S_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `ABSQ_S_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 993-996
```tablegen
// Multiplication
class MUL_PH_DESC : ADDUH_QB_DESC_BASE<"mul.ph", null_frag,
                                       DSPROpnd>, IsCommutable,
                    Defs<[DSPOutFlag21]>;
```
- EN: Declares reusable TableGen class `MUL_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MUL_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 998-1000
```tablegen
class MUL_S_PH_DESC : ADDUH_QB_DESC_BASE<"mul_s.ph", int_mips_mul_s_ph,
                                         DSPROpnd>, IsCommutable,
                      Defs<[DSPOutFlag21]>;
```
- EN: Declares reusable TableGen class `MUL_S_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MUL_S_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1002-1004
```tablegen
class MULQ_S_W_DESC : ADDUH_QB_DESC_BASE<"mulq_s.w", int_mips_mulq_s_w,
                                         GPR32Opnd>, IsCommutable,
                      Defs<[DSPOutFlag21]>;
```
- EN: Declares reusable TableGen class `MULQ_S_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MULQ_S_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1006-1008
```tablegen
class MULQ_RS_W_DESC : ADDUH_QB_DESC_BASE<"mulq_rs.w", int_mips_mulq_rs_w,
                                          GPR32Opnd>, IsCommutable,
                       Defs<[DSPOutFlag21]>;
```
- EN: Declares reusable TableGen class `MULQ_RS_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MULQ_RS_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1010-1012
```tablegen
class MULQ_S_PH_DESC : ADDU_QB_DESC_BASE<"mulq_s.ph", int_mips_mulq_s_ph,
                                         DSPROpnd, DSPROpnd>,
                       IsCommutable, Defs<[DSPOutFlag21]>;
```
- EN: Declares reusable TableGen class `MULQ_S_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MULQ_S_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1014-1015
```tablegen
// Dot product with accumulate/subtract
class DPA_W_PH_DESC : DPA_DESC_BASE<"dpa.w.ph", MipsDPA_W_PH, v2i16>;
```
- EN: Declares reusable TableGen class `DPA_W_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DPA_W_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1017-1017
```tablegen
class DPS_W_PH_DESC : DPA_DESC_BASE<"dps.w.ph", MipsDPS_W_PH, v2i16>;
```
- EN: Declares reusable TableGen class `DPS_W_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DPS_W_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1019-1020
```tablegen
class DPAQX_S_W_PH_DESC : DPA_DESC_BASE<"dpaqx_s.w.ph", MipsDPAQX_S_W_PH, v2i16>,
                          Defs<[DSPOutFlag16_19]>;
```
- EN: Declares reusable TableGen class `DPAQX_S_W_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DPAQX_S_W_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1022-1024
```tablegen
class DPAQX_SA_W_PH_DESC : DPA_DESC_BASE<"dpaqx_sa.w.ph",
                                         MipsDPAQX_SA_W_PH, v2i16>,
                           Defs<[DSPOutFlag16_19]>;
```
- EN: Declares reusable TableGen class `DPAQX_SA_W_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DPAQX_SA_W_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1026-1026
```tablegen
class DPAX_W_PH_DESC : DPA_DESC_BASE<"dpax.w.ph", MipsDPAX_W_PH, v2i16>;
```
- EN: Declares reusable TableGen class `DPAX_W_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DPAX_W_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1028-1028
```tablegen
class DPSX_W_PH_DESC : DPA_DESC_BASE<"dpsx.w.ph", MipsDPSX_W_PH, v2i16>;
```
- EN: Declares reusable TableGen class `DPSX_W_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DPSX_W_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1030-1031
```tablegen
class DPSQX_S_W_PH_DESC : DPA_DESC_BASE<"dpsqx_s.w.ph", MipsDPSQX_S_W_PH, v2i16>,
                          Defs<[DSPOutFlag16_19]>;
```
- EN: Declares reusable TableGen class `DPSQX_S_W_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DPSQX_S_W_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1033-1035
```tablegen
class DPSQX_SA_W_PH_DESC : DPA_DESC_BASE<"dpsqx_sa.w.ph",
                                         MipsDPSQX_SA_W_PH, v2i16>,
                           Defs<[DSPOutFlag16_19]>;
```
- EN: Declares reusable TableGen class `DPSQX_SA_W_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DPSQX_SA_W_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1037-1037
```tablegen
class MULSA_W_PH_DESC : DPA_DESC_BASE<"mulsa.w.ph", MipsMULSA_W_PH, v2i16>;
```
- EN: Declares reusable TableGen class `MULSA_W_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `MULSA_W_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1039-1042
```tablegen
// Precision reduce/expand
class PRECR_QB_PH_DESC : CMP_EQ_QB_R3_DESC_BASE<"precr.qb.ph",
                                                int_mips_precr_qb_ph,
                                                DSPROpnd, DSPROpnd>;
```
- EN: Declares reusable TableGen class `PRECR_QB_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PRECR_QB_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1044-1047
```tablegen
class PRECR_SRA_PH_W_DESC : PRECR_SRA_PH_W_DESC_BASE<"precr_sra.ph.w",
                                                     int_mips_precr_sra_ph_w,
                                                     DSPROpnd,
                                                     GPR32Opnd>;
```
- EN: Declares reusable TableGen class `PRECR_SRA_PH_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PRECR_SRA_PH_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1049-1052
```tablegen
class PRECR_SRA_R_PH_W_DESC : PRECR_SRA_PH_W_DESC_BASE<"precr_sra_r.ph.w",
                                                      int_mips_precr_sra_r_ph_w,
                                                       DSPROpnd,
                                                       GPR32Opnd>;
```
- EN: Declares reusable TableGen class `PRECR_SRA_R_PH_W_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PRECR_SRA_R_PH_W_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1054-1056
```tablegen
// Shift
class SHRA_QB_DESC : SHLL_QB_R2_DESC_BASE<"shra.qb", null_frag, immZExt3,
                                          DSPROpnd, uimm3>;
```
- EN: Declares reusable TableGen class `SHRA_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHRA_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1058-1059
```tablegen
class SHRAV_QB_DESC : SHLL_QB_R3_DESC_BASE<"shrav.qb", int_mips_shra_qb,
                                           DSPROpnd>;
```
- EN: Declares reusable TableGen class `SHRAV_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHRAV_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1061-1063
```tablegen
class SHRA_R_QB_DESC : SHLL_QB_R2_DESC_BASE<"shra_r.qb", int_mips_shra_r_qb,
                                            immZExt3, DSPROpnd,
                                            uimm3>;
```
- EN: Declares reusable TableGen class `SHRA_R_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHRA_R_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1065-1066
```tablegen
class SHRAV_R_QB_DESC : SHLL_QB_R3_DESC_BASE<"shrav_r.qb", int_mips_shra_r_qb,
                                             DSPROpnd>;
```
- EN: Declares reusable TableGen class `SHRAV_R_QB_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHRAV_R_QB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1068-1069
```tablegen
class SHRL_PH_DESC : SHLL_QB_R2_DESC_BASE<"shrl.ph", null_frag, immZExt4,
                                          DSPROpnd, uimm4>;
```
- EN: Declares reusable TableGen class `SHRL_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHRL_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1071-1072
```tablegen
class SHRLV_PH_DESC : SHLL_QB_R3_DESC_BASE<"shrlv.ph", int_mips_shrl_ph,
                                           DSPROpnd>;
```
- EN: Declares reusable TableGen class `SHRLV_PH_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `SHRLV_PH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1074-1075
```tablegen
// Misc
class APPEND_DESC : APPEND_DESC_BASE<"append", int_mips_append, uimm5, timmZExt5>;
```
- EN: Declares reusable TableGen class `APPEND_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `APPEND_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1077-1077
```tablegen
class BALIGN_DESC : APPEND_DESC_BASE<"balign", int_mips_balign, uimm2, timmZExt2>;
```
- EN: Declares reusable TableGen class `BALIGN_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `BALIGN_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1079-1080
```tablegen
class PREPEND_DESC : APPEND_DESC_BASE<"prepend", int_mips_prepend, uimm5,
                                      timmZExt5>;
```
- EN: Declares reusable TableGen class `PREPEND_DESC` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PREPEND_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1082-1084
```tablegen
// Pseudos.
def BPOSGE32_PSEUDO : BPOSGE32_PSEUDO_DESC_BASE<int_mips_bposge32>,
                      Uses<[DSPPos]>;
```
- EN: Defines TableGen record `BPOSGE32_PSEUDO` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BPOSGE32_PSEUDO`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1086-1103
```tablegen
// Instruction defs.
// MIPS DSP Rev 1
def ADDU_QB : DspMMRel, ADDU_QB_ENC, ADDU_QB_DESC;
def ADDU_S_QB : DspMMRel, ADDU_S_QB_ENC, ADDU_S_QB_DESC;
def SUBU_QB : DspMMRel, SUBU_QB_ENC, SUBU_QB_DESC;
def SUBU_S_QB : DspMMRel, SUBU_S_QB_ENC, SUBU_S_QB_DESC;
def ADDQ_PH : DspMMRel, ADDQ_PH_ENC, ADDQ_PH_DESC;
def ADDQ_S_PH : DspMMRel, ADDQ_S_PH_ENC, ADDQ_S_PH_DESC;
def SUBQ_PH : DspMMRel, SUBQ_PH_ENC, SUBQ_PH_DESC;
def SUBQ_S_PH : DspMMRel, SUBQ_S_PH_ENC, SUBQ_S_PH_DESC;
def ADDQ_S_W : DspMMRel, ADDQ_S_W_ENC, ADDQ_S_W_DESC;
def SUBQ_S_W : DspMMRel, SUBQ_S_W_ENC, SUBQ_S_W_DESC;
def ADDSC : DspMMRel, ADDSC_ENC, ADDSC_DESC;
def ADDWC : DspMMRel, ADDWC_ENC, ADDWC_DESC;
def MODSUB : DspMMRel, MODSUB_ENC, MODSUB_DESC;
def RADDU_W_QB : DspMMRel, RADDU_W_QB_ENC, RADDU_W_QB_DESC;
def ABSQ_S_PH : DspMMRel, ABSQ_S_PH_ENC, ABSQ_S_PH_DESC;
def ABSQ_S_W : DspMMRel, ABSQ_S_W_ENC, ABSQ_S_W_DESC;
```
- EN: Defines TableGen record `ADDU_QB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ADDU_QB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1104-1121
```tablegen
def PRECRQ_QB_PH : DspMMRel, PRECRQ_QB_PH_ENC, PRECRQ_QB_PH_DESC;
def PRECRQ_PH_W : DspMMRel, PRECRQ_PH_W_ENC, PRECRQ_PH_W_DESC;
def PRECRQ_RS_PH_W : DspMMRel, PRECRQ_RS_PH_W_ENC, PRECRQ_RS_PH_W_DESC;
def PRECRQU_S_QB_PH : DspMMRel, PRECRQU_S_QB_PH_ENC, PRECRQU_S_QB_PH_DESC;
def PRECEQ_W_PHL : DspMMRel, PRECEQ_W_PHL_ENC, PRECEQ_W_PHL_DESC;
def PRECEQ_W_PHR : DspMMRel, PRECEQ_W_PHR_ENC, PRECEQ_W_PHR_DESC;
def PRECEQU_PH_QBL : DspMMRel, PRECEQU_PH_QBL_ENC, PRECEQU_PH_QBL_DESC;
def PRECEQU_PH_QBR : DspMMRel, PRECEQU_PH_QBR_ENC, PRECEQU_PH_QBR_DESC;
def PRECEQU_PH_QBLA : DspMMRel, PRECEQU_PH_QBLA_ENC, PRECEQU_PH_QBLA_DESC;
def PRECEQU_PH_QBRA : DspMMRel, PRECEQU_PH_QBRA_ENC, PRECEQU_PH_QBRA_DESC;
def PRECEU_PH_QBL : DspMMRel, PRECEU_PH_QBL_ENC, PRECEU_PH_QBL_DESC;
def PRECEU_PH_QBR : DspMMRel, PRECEU_PH_QBR_ENC, PRECEU_PH_QBR_DESC;
def PRECEU_PH_QBLA : DspMMRel, PRECEU_PH_QBLA_ENC, PRECEU_PH_QBLA_DESC;
def PRECEU_PH_QBRA : DspMMRel, PRECEU_PH_QBRA_ENC, PRECEU_PH_QBRA_DESC;
def SHLL_QB : DspMMRel, SHLL_QB_ENC, SHLL_QB_DESC;
def SHLLV_QB : DspMMRel, SHLLV_QB_ENC, SHLLV_QB_DESC;
def SHRL_QB : DspMMRel, SHRL_QB_ENC, SHRL_QB_DESC;
def SHRLV_QB : DspMMRel, SHRLV_QB_ENC, SHRLV_QB_DESC;
```
- EN: Defines TableGen record `PRECRQ_QB_PH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PRECRQ_QB_PH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1122-1139
```tablegen
def SHLL_PH : DspMMRel, SHLL_PH_ENC, SHLL_PH_DESC;
def SHLLV_PH : DspMMRel, SHLLV_PH_ENC, SHLLV_PH_DESC;
def SHLL_S_PH : DspMMRel, SHLL_S_PH_ENC, SHLL_S_PH_DESC;
def SHLLV_S_PH : DspMMRel, SHLLV_S_PH_ENC, SHLLV_S_PH_DESC;
def SHRA_PH : DspMMRel, SHRA_PH_ENC, SHRA_PH_DESC;
def SHRAV_PH : DspMMRel, SHRAV_PH_ENC, SHRAV_PH_DESC;
def SHRA_R_PH : DspMMRel, SHRA_R_PH_ENC, SHRA_R_PH_DESC;
def SHRAV_R_PH : DspMMRel, SHRAV_R_PH_ENC, SHRAV_R_PH_DESC;
def SHLL_S_W : DspMMRel, SHLL_S_W_ENC, SHLL_S_W_DESC;
def SHLLV_S_W : DspMMRel, SHLLV_S_W_ENC, SHLLV_S_W_DESC;
def SHRA_R_W : DspMMRel, SHRA_R_W_ENC, SHRA_R_W_DESC;
def SHRAV_R_W : DspMMRel, SHRAV_R_W_ENC, SHRAV_R_W_DESC;
def MULEU_S_PH_QBL : DspMMRel, MULEU_S_PH_QBL_ENC, MULEU_S_PH_QBL_DESC;
def MULEU_S_PH_QBR : DspMMRel, MULEU_S_PH_QBR_ENC, MULEU_S_PH_QBR_DESC;
def MULEQ_S_W_PHL : DspMMRel, MULEQ_S_W_PHL_ENC, MULEQ_S_W_PHL_DESC;
def MULEQ_S_W_PHR : DspMMRel, MULEQ_S_W_PHR_ENC, MULEQ_S_W_PHR_DESC;
def MULQ_RS_PH : DspMMRel, MULQ_RS_PH_ENC, MULQ_RS_PH_DESC;
def MULSAQ_S_W_PH : DspMMRel, MULSAQ_S_W_PH_ENC, MULSAQ_S_W_PH_DESC;
```
- EN: Defines TableGen record `SHLL_PH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SHLL_PH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1140-1157
```tablegen
def MAQ_S_W_PHL : DspMMRel, MAQ_S_W_PHL_ENC, MAQ_S_W_PHL_DESC;
def MAQ_S_W_PHR : DspMMRel, MAQ_S_W_PHR_ENC, MAQ_S_W_PHR_DESC;
def MAQ_SA_W_PHL : DspMMRel, MAQ_SA_W_PHL_ENC, MAQ_SA_W_PHL_DESC;
def MAQ_SA_W_PHR : DspMMRel, MAQ_SA_W_PHR_ENC, MAQ_SA_W_PHR_DESC;
def MFHI_DSP : DspMMRel, MFHI_ENC, MFHI_DESC;
def MFLO_DSP : DspMMRel, MFLO_ENC, MFLO_DESC;
def MTHI_DSP : DspMMRel, MTHI_ENC, MTHI_DESC;
def MTLO_DSP : DspMMRel, MTLO_ENC, MTLO_DESC;
def DPAU_H_QBL : DspMMRel, DPAU_H_QBL_ENC, DPAU_H_QBL_DESC;
def DPAU_H_QBR : DspMMRel, DPAU_H_QBR_ENC, DPAU_H_QBR_DESC;
def DPSU_H_QBL : DspMMRel, DPSU_H_QBL_ENC, DPSU_H_QBL_DESC;
def DPSU_H_QBR : DspMMRel, DPSU_H_QBR_ENC, DPSU_H_QBR_DESC;
def DPAQ_S_W_PH : DspMMRel, DPAQ_S_W_PH_ENC, DPAQ_S_W_PH_DESC;
def DPSQ_S_W_PH : DspMMRel, DPSQ_S_W_PH_ENC, DPSQ_S_W_PH_DESC;
def DPAQ_SA_L_W : DspMMRel, DPAQ_SA_L_W_ENC, DPAQ_SA_L_W_DESC;
def DPSQ_SA_L_W : DspMMRel, DPSQ_SA_L_W_ENC, DPSQ_SA_L_W_DESC;
def MULT_DSP : DspMMRel, MULT_DSP_ENC, MULT_DSP_DESC;
def MULTU_DSP : DspMMRel, MULTU_DSP_ENC, MULTU_DSP_DESC;
```
- EN: Defines TableGen record `MAQ_S_W_PHL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MAQ_S_W_PHL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1158-1175
```tablegen
def MADD_DSP : DspMMRel, MADD_DSP_ENC, MADD_DSP_DESC;
def MADDU_DSP : DspMMRel, MADDU_DSP_ENC, MADDU_DSP_DESC;
def MSUB_DSP : DspMMRel, MSUB_DSP_ENC, MSUB_DSP_DESC;
def MSUBU_DSP : DspMMRel, MSUBU_DSP_ENC, MSUBU_DSP_DESC;
def CMPU_EQ_QB : DspMMRel, CMPU_EQ_QB_ENC, CMPU_EQ_QB_DESC;
def CMPU_LT_QB : DspMMRel, CMPU_LT_QB_ENC, CMPU_LT_QB_DESC;
def CMPU_LE_QB : DspMMRel, CMPU_LE_QB_ENC, CMPU_LE_QB_DESC;
def CMPGU_EQ_QB : DspMMRel, CMPGU_EQ_QB_ENC, CMPGU_EQ_QB_DESC;
def CMPGU_LT_QB : DspMMRel, CMPGU_LT_QB_ENC, CMPGU_LT_QB_DESC;
def CMPGU_LE_QB : DspMMRel, CMPGU_LE_QB_ENC, CMPGU_LE_QB_DESC;
def CMP_EQ_PH : DspMMRel, CMP_EQ_PH_ENC, CMP_EQ_PH_DESC;
def CMP_LT_PH : DspMMRel, CMP_LT_PH_ENC, CMP_LT_PH_DESC;
def CMP_LE_PH : DspMMRel, CMP_LE_PH_ENC, CMP_LE_PH_DESC;
def BITREV : DspMMRel, BITREV_ENC, BITREV_DESC;
def PACKRL_PH : DspMMRel, PACKRL_PH_ENC, PACKRL_PH_DESC;
def REPL_QB : DspMMRel, REPL_QB_ENC, REPL_QB_DESC;
def REPL_PH : DspMMRel, REPL_PH_ENC, REPL_PH_DESC;
def REPLV_QB : DspMMRel, REPLV_QB_ENC, REPLV_QB_DESC;
```
- EN: Defines TableGen record `MADD_DSP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MADD_DSP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1176-1193
```tablegen
def REPLV_PH : DspMMRel, REPLV_PH_ENC, REPLV_PH_DESC;
def PICK_QB : DspMMRel, PICK_QB_ENC, PICK_QB_DESC;
def PICK_PH : DspMMRel, PICK_PH_ENC, PICK_PH_DESC;
def LWX : DspMMRel, LWX_ENC, LWX_DESC;
def LHX : DspMMRel, LHX_ENC, LHX_DESC;
def LBUX : DspMMRel, LBUX_ENC, LBUX_DESC;
let AdditionalPredicates = [NotInMicroMips] in {
  def BPOSGE32 : DspMMRel, BPOSGE32_ENC, BPOSGE32_DESC;
}
def INSV : DspMMRel, INSV_ENC, INSV_DESC;
def EXTP : DspMMRel, EXTP_ENC, EXTP_DESC;
def EXTPV : DspMMRel, EXTPV_ENC, EXTPV_DESC;
def EXTPDP : DspMMRel, EXTPDP_ENC, EXTPDP_DESC;
def EXTPDPV : DspMMRel, EXTPDPV_ENC, EXTPDPV_DESC;
def EXTR_W : DspMMRel, EXTR_W_ENC, EXTR_W_DESC;
def EXTRV_W : DspMMRel, EXTRV_W_ENC, EXTRV_W_DESC;
def EXTR_R_W : DspMMRel, EXTR_R_W_ENC, EXTR_R_W_DESC;
def EXTRV_R_W : DspMMRel, EXTRV_R_W_ENC, EXTRV_R_W_DESC;
```
- EN: Defines TableGen record `REPLV_PH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `REPLV_PH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1194-1204
```tablegen
def EXTR_RS_W : DspMMRel, EXTR_RS_W_ENC, EXTR_RS_W_DESC;
def EXTRV_RS_W : DspMMRel, EXTRV_RS_W_ENC, EXTRV_RS_W_DESC;
def EXTR_S_H : DspMMRel, EXTR_S_H_ENC, EXTR_S_H_DESC;
def EXTRV_S_H : DspMMRel, EXTRV_S_H_ENC, EXTRV_S_H_DESC;
def SHILO : DspMMRel, SHILO_ENC, SHILO_DESC;
def SHILOV : DspMMRel, SHILOV_ENC, SHILOV_DESC;
def MTHLIP : DspMMRel, MTHLIP_ENC, MTHLIP_DESC;
def RDDSP : DspMMRel, RDDSP_ENC, RDDSP_DESC;
let AdditionalPredicates = [NotInMicroMips] in {
  def WRDSP : WRDSP_ENC, WRDSP_DESC;
}
```
- EN: Defines TableGen record `EXTR_RS_W` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `EXTR_RS_W`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1206-1223
```tablegen
// MIPS DSP Rev 2
def ADDU_PH : DspMMRel, ADDU_PH_ENC, ADDU_PH_DESC, ISA_DSPR2;
def ADDU_S_PH : DspMMRel, ADDU_S_PH_ENC, ADDU_S_PH_DESC, ISA_DSPR2;
def SUBU_PH : DspMMRel, SUBU_PH_ENC, SUBU_PH_DESC, ISA_DSPR2;
def SUBU_S_PH : DspMMRel, SUBU_S_PH_ENC, SUBU_S_PH_DESC, ISA_DSPR2;
def CMPGDU_EQ_QB : DspMMRel, CMPGDU_EQ_QB_ENC, CMPGDU_EQ_QB_DESC, ISA_DSPR2;
def CMPGDU_LT_QB : DspMMRel, CMPGDU_LT_QB_ENC, CMPGDU_LT_QB_DESC, ISA_DSPR2;
def CMPGDU_LE_QB : DspMMRel, CMPGDU_LE_QB_ENC, CMPGDU_LE_QB_DESC, ISA_DSPR2;
def ABSQ_S_QB : DspMMRel, ABSQ_S_QB_ENC, ABSQ_S_QB_DESC, ISA_DSPR2;
def ADDUH_QB : DspMMRel, ADDUH_QB_ENC, ADDUH_QB_DESC, ISA_DSPR2;
def ADDUH_R_QB : DspMMRel, ADDUH_R_QB_ENC, ADDUH_R_QB_DESC, ISA_DSPR2;
def SUBUH_QB : DspMMRel, SUBUH_QB_ENC, SUBUH_QB_DESC, ISA_DSPR2;
def SUBUH_R_QB : DspMMRel, SUBUH_R_QB_ENC, SUBUH_R_QB_DESC, ISA_DSPR2;
def ADDQH_PH : DspMMRel, ADDQH_PH_ENC, ADDQH_PH_DESC, ISA_DSPR2;
def ADDQH_R_PH : DspMMRel, ADDQH_R_PH_ENC, ADDQH_R_PH_DESC, ISA_DSPR2;
def SUBQH_PH : DspMMRel, SUBQH_PH_ENC, SUBQH_PH_DESC, ISA_DSPR2;
def SUBQH_R_PH : DspMMRel, SUBQH_R_PH_ENC, SUBQH_R_PH_DESC, ISA_DSPR2;
def ADDQH_W : DspMMRel, ADDQH_W_ENC, ADDQH_W_DESC, ISA_DSPR2;
```
- EN: Defines TableGen record `ADDU_PH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ADDU_PH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1224-1241
```tablegen
def ADDQH_R_W : DspMMRel, ADDQH_R_W_ENC, ADDQH_R_W_DESC, ISA_DSPR2;
def SUBQH_W : DspMMRel, SUBQH_W_ENC, SUBQH_W_DESC, ISA_DSPR2;
def SUBQH_R_W : DspMMRel, SUBQH_R_W_ENC, SUBQH_R_W_DESC, ISA_DSPR2;
def MUL_PH : DspMMRel, MUL_PH_ENC, MUL_PH_DESC, ISA_DSPR2;
def MUL_S_PH : DspMMRel, MUL_S_PH_ENC, MUL_S_PH_DESC, ISA_DSPR2;
def MULQ_S_W : DspMMRel, MULQ_S_W_ENC, MULQ_S_W_DESC, ISA_DSPR2;
def MULQ_RS_W : DspMMRel, MULQ_RS_W_ENC, MULQ_RS_W_DESC, ISA_DSPR2;
def MULQ_S_PH : DspMMRel, MULQ_S_PH_ENC, MULQ_S_PH_DESC, ISA_DSPR2;
def DPA_W_PH : DspMMRel, DPA_W_PH_ENC, DPA_W_PH_DESC, ISA_DSPR2;
def DPS_W_PH : DspMMRel, DPS_W_PH_ENC, DPS_W_PH_DESC, ISA_DSPR2;
def DPAQX_S_W_PH : DspMMRel, DPAQX_S_W_PH_ENC, DPAQX_S_W_PH_DESC, ISA_DSPR2;
def DPAQX_SA_W_PH : DspMMRel, DPAQX_SA_W_PH_ENC, DPAQX_SA_W_PH_DESC, ISA_DSPR2;
def DPAX_W_PH : DspMMRel, DPAX_W_PH_ENC, DPAX_W_PH_DESC, ISA_DSPR2;
def DPSX_W_PH : DspMMRel, DPSX_W_PH_ENC, DPSX_W_PH_DESC, ISA_DSPR2;
def DPSQX_S_W_PH : DspMMRel, DPSQX_S_W_PH_ENC, DPSQX_S_W_PH_DESC, ISA_DSPR2;
def DPSQX_SA_W_PH : DspMMRel, DPSQX_SA_W_PH_ENC, DPSQX_SA_W_PH_DESC, ISA_DSPR2;
def MULSA_W_PH : DspMMRel, MULSA_W_PH_ENC, MULSA_W_PH_DESC, ISA_DSPR2;
def PRECR_QB_PH : DspMMRel, PRECR_QB_PH_ENC, PRECR_QB_PH_DESC, ISA_DSPR2;
```
- EN: Defines TableGen record `ADDQH_R_W` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ADDQH_R_W`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1242-1252
```tablegen
def PRECR_SRA_PH_W : DspMMRel, PRECR_SRA_PH_W_ENC, PRECR_SRA_PH_W_DESC, ISA_DSPR2;
def PRECR_SRA_R_PH_W : DspMMRel, PRECR_SRA_R_PH_W_ENC, PRECR_SRA_R_PH_W_DESC, ISA_DSPR2;
def SHRA_QB : DspMMRel, SHRA_QB_ENC, SHRA_QB_DESC, ISA_DSPR2;
def SHRAV_QB : DspMMRel, SHRAV_QB_ENC, SHRAV_QB_DESC, ISA_DSPR2;
def SHRA_R_QB : DspMMRel, SHRA_R_QB_ENC, SHRA_R_QB_DESC, ISA_DSPR2;
def SHRAV_R_QB : DspMMRel, SHRAV_R_QB_ENC, SHRAV_R_QB_DESC, ISA_DSPR2;
def SHRL_PH : DspMMRel, SHRL_PH_ENC, SHRL_PH_DESC, ISA_DSPR2;
def SHRLV_PH : DspMMRel, SHRLV_PH_ENC, SHRLV_PH_DESC, ISA_DSPR2;
def APPEND : DspMMRel, APPEND_ENC, APPEND_DESC, ISA_DSPR2;
def BALIGN : DspMMRel, BALIGN_ENC, BALIGN_DESC, ISA_DSPR2;
def PREPEND : DspMMRel, PREPEND_ENC, PREPEND_DESC, ISA_DSPR2;
```
- EN: Defines TableGen record `PRECR_SRA_PH_W` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PRECR_SRA_PH_W`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1254-1258
```tablegen
// Pseudos.
let isPseudo = 1, isCodeGenOnly = 1, hasNoSchedulingInfo = 1 in {
  // Pseudo instructions for loading and storing accumulator registers.
  def LOAD_ACC64DSP  : Load<"", ACC64DSPOpnd>;
  def STORE_ACC64DSP : Store<"", ACC64DSPOpnd>;
```
- EN: Defines TableGen record `LOAD_ACC64DSP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LOAD_ACC64DSP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1260-1263
```tablegen
  // Pseudos for loading and storing ccond field of DSP control register.
  def LOAD_CCOND_DSP  : Load<"load_ccond_dsp", DSPCC>;
  def STORE_CCOND_DSP : Store<"store_ccond_dsp", DSPCC>;
}
```
- EN: Defines TableGen record `LOAD_CCOND_DSP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LOAD_CCOND_DSP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1265-1269
```tablegen
let DecoderNamespace = "MipsDSP", Arch = "dsp",
    ASEPredicate = [HasDSP] in {
  def LWDSP : Load<"lw", DSPROpnd>, DspMMRel, LW_FM<0x23>;
  def SWDSP : Store<"sw", DSPROpnd>, DspMMRel, LW_FM<0x2b>;
}
```
- EN: Defines TableGen record `LWDSP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LWDSP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1271-1275
```tablegen
// Pseudo CMP and PICK instructions.
class PseudoCMP<Instruction RealInst> :
  PseudoDSP<(outs DSPCC:$cmp), (ins DSPROpnd:$rs, DSPROpnd:$rt), []>,
  PseudoInstExpansion<(RealInst DSPROpnd:$rs, DSPROpnd:$rt)>,
  NeverHasSideEffects;
```
- EN: Declares reusable TableGen class `PseudoCMP` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PseudoCMP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1277-1280
```tablegen
class PseudoPICK<Instruction RealInst> :
  PseudoDSP<(outs DSPROpnd:$rd), (ins DSPCC:$cmp, DSPROpnd:$rs, DSPROpnd:$rt), []>,
  PseudoInstExpansion<(RealInst DSPROpnd:$rd, DSPROpnd:$rs, DSPROpnd:$rt)>,
  NeverHasSideEffects;
```
- EN: Declares reusable TableGen class `PseudoPICK` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `PseudoPICK`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1282-1287
```tablegen
def PseudoCMP_EQ_PH : PseudoCMP<CMP_EQ_PH>;
def PseudoCMP_LT_PH : PseudoCMP<CMP_LT_PH>;
def PseudoCMP_LE_PH : PseudoCMP<CMP_LE_PH>;
def PseudoCMPU_EQ_QB : PseudoCMP<CMPU_EQ_QB>;
def PseudoCMPU_LT_QB : PseudoCMP<CMPU_LT_QB>;
def PseudoCMPU_LE_QB : PseudoCMP<CMPU_LE_QB>;
```
- EN: Defines TableGen record `PseudoCMP_EQ_PH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PseudoCMP_EQ_PH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1289-1290
```tablegen
def PseudoPICK_PH : PseudoPICK<PICK_PH>;
def PseudoPICK_QB : PseudoPICK<PICK_QB>;
```
- EN: Defines TableGen record `PseudoPICK_PH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PseudoPICK_PH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1292-1294
```tablegen
let AdditionalPredicates = [HasDSP] in {
  def PseudoMTLOHI_DSP : PseudoMTLOHI<ACC64DSP, GPR32>;
}
```
- EN: Defines TableGen record `PseudoMTLOHI_DSP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PseudoMTLOHI_DSP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1296-1298
```tablegen
// Patterns.
class DSPPat<dag pattern, dag result, Predicate pred = HasDSP> :
  Pat<pattern, result>, Requires<[pred]>;
```
- EN: Declares reusable TableGen class `DSPPat` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DSPPat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1300-1303
```tablegen
class BitconvertPat<ValueType DstVT, ValueType SrcVT, RegisterClass DstRC,
                    RegisterClass SrcRC> :
   DSPPat<(DstVT (bitconvert (SrcVT SrcRC:$src))),
          (COPY_TO_REGCLASS SrcRC:$src, DstRC)>;
```
- EN: Declares reusable TableGen class `BitconvertPat` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `BitconvertPat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1305-1312
```tablegen
def : BitconvertPat<i32, v2i16, GPR32, DSPR>;
def : BitconvertPat<i32, v4i8, GPR32, DSPR>;
def : BitconvertPat<v2i16, i32, DSPR, GPR32>;
def : BitconvertPat<v4i8, i32, DSPR, GPR32>;
def : BitconvertPat<f32, v2i16, FGR32, DSPR>;
def : BitconvertPat<f32, v4i8, FGR32, DSPR>;
def : BitconvertPat<v2i16, f32, DSPR, FGR32>;
def : BitconvertPat<v4i8, f32, DSPR, FGR32>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1314-1321
```tablegen
def : DSPPat<(v2i16 (load addr:$a)),
             (v2i16 (COPY_TO_REGCLASS (LW addr:$a), DSPR))>;
def : DSPPat<(v4i8 (load addr:$a)),
             (v4i8 (COPY_TO_REGCLASS (LW addr:$a), DSPR))>;
def : DSPPat<(store (v2i16 DSPR:$val), addr:$a),
             (SW (COPY_TO_REGCLASS DSPR:$val, GPR32), addr:$a)>;
def : DSPPat<(store (v4i8 DSPR:$val), addr:$a),
             (SW (COPY_TO_REGCLASS DSPR:$val, GPR32), addr:$a)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1323-1326
```tablegen
// Binary operations.
class DSPBinPat<Instruction Inst, ValueType ValTy, SDPatternOperator Node,
                Predicate Pred = HasDSP> :
  DSPPat<(Node ValTy:$a, ValTy:$b), (Inst ValTy:$a, ValTy:$b), Pred>;
```
- EN: Declares reusable TableGen class `DSPBinPat` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DSPBinPat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1328-1341
```tablegen
def : DSPBinPat<ADDQ_PH, v2i16, int_mips_addq_ph>;
def : DSPBinPat<ADDQ_PH, v2i16, add>;
def : DSPBinPat<SUBQ_PH, v2i16, int_mips_subq_ph>;
def : DSPBinPat<SUBQ_PH, v2i16, sub>;
def : DSPBinPat<MUL_PH, v2i16, int_mips_mul_ph, HasDSPR2>;
def : DSPBinPat<MUL_PH, v2i16, mul, HasDSPR2>;
def : DSPBinPat<ADDU_QB, v4i8, int_mips_addu_qb>;
def : DSPBinPat<ADDU_QB, v4i8, add>;
def : DSPBinPat<SUBU_QB, v4i8, int_mips_subu_qb>;
def : DSPBinPat<SUBU_QB, v4i8, sub>;
def : DSPBinPat<ADDSC, i32, int_mips_addsc>;
def : DSPBinPat<ADDSC, i32, addc>;
def : DSPBinPat<ADDWC, i32, int_mips_addwc>;
def : DSPBinPat<ADDWC, i32, adde>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1343-1346
```tablegen
// Shift immediate patterns.
class DSPShiftPat<Instruction Inst, ValueType ValTy, SDPatternOperator Node,
                  SDPatternOperator Imm, Predicate Pred = HasDSP> :
  DSPPat<(Node ValTy:$a, Imm:$shamt), (Inst ValTy:$a, Imm:$shamt), Pred>;
```
- EN: Declares reusable TableGen class `DSPShiftPat` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DSPShiftPat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1348-1359
```tablegen
def : DSPShiftPat<SHLL_PH, v2i16, MipsSHLL_DSP, imm>;
def : DSPShiftPat<SHRA_PH, v2i16, MipsSHRA_DSP, imm>;
def : DSPShiftPat<SHRL_PH, v2i16, MipsSHRL_DSP, imm, HasDSPR2>;
def : DSPShiftPat<SHLL_PH, v2i16, int_mips_shll_ph, immZExt4>;
def : DSPShiftPat<SHRA_PH, v2i16, int_mips_shra_ph, immZExt4>;
def : DSPShiftPat<SHRL_PH, v2i16, int_mips_shrl_ph, immZExt4, HasDSPR2>;
def : DSPShiftPat<SHLL_QB, v4i8, MipsSHLL_DSP, imm>;
def : DSPShiftPat<SHRA_QB, v4i8, MipsSHRA_DSP, imm, HasDSPR2>;
def : DSPShiftPat<SHRL_QB, v4i8, MipsSHRL_DSP, imm>;
def : DSPShiftPat<SHLL_QB, v4i8, int_mips_shll_qb, immZExt3>;
def : DSPShiftPat<SHRA_QB, v4i8, int_mips_shra_qb, immZExt3, HasDSPR2>;
def : DSPShiftPat<SHRL_QB, v4i8, int_mips_shrl_qb, immZExt3>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1361-1367
```tablegen
// SETCC/SELECT_CC patterns.
class DSPSetCCPat<Instruction Cmp, Instruction Pick, ValueType ValTy,
                  CondCode CC> :
  DSPPat<(ValTy (MipsSETCC_DSP ValTy:$a, ValTy:$b, CC)),
         (ValTy (Pick (ValTy (Cmp ValTy:$a, ValTy:$b)),
                      (ValTy (COPY_TO_REGCLASS (ADDiu ZERO, -1), DSPR)),
                      (ValTy ZERO)))>;
```
- EN: Declares reusable TableGen class `DSPSetCCPat` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DSPSetCCPat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1369-1374
```tablegen
class DSPSetCCPatInv<Instruction Cmp, Instruction Pick, ValueType ValTy,
                     CondCode CC> :
  DSPPat<(ValTy (MipsSETCC_DSP ValTy:$a, ValTy:$b, CC)),
         (ValTy (Pick (ValTy (Cmp ValTy:$a, ValTy:$b)),
                      (ValTy ZERO),
                      (ValTy (COPY_TO_REGCLASS (ADDiu ZERO, -1), DSPR))))>;
```
- EN: Declares reusable TableGen class `DSPSetCCPatInv` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DSPSetCCPatInv`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1376-1379
```tablegen
class DSPSelectCCPat<Instruction Cmp, Instruction Pick, ValueType ValTy,
                     CondCode CC> :
  DSPPat<(ValTy (MipsSELECT_CC_DSP ValTy:$a, ValTy:$b, ValTy:$c, ValTy:$d, CC)),
         (ValTy (Pick (ValTy (Cmp ValTy:$a, ValTy:$b)), $c, $d))>;
```
- EN: Declares reusable TableGen class `DSPSelectCCPat` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DSPSelectCCPat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1381-1384
```tablegen
class DSPSelectCCPatInv<Instruction Cmp, Instruction Pick, ValueType ValTy,
                        CondCode CC> :
  DSPPat<(ValTy (MipsSELECT_CC_DSP ValTy:$a, ValTy:$b, ValTy:$c, ValTy:$d, CC)),
         (ValTy (Pick (ValTy (Cmp ValTy:$a, ValTy:$b)), $d, $c))>;
```
- EN: Declares reusable TableGen class `DSPSelectCCPatInv` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `DSPSelectCCPatInv`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1386-1397
```tablegen
def : DSPSetCCPat<PseudoCMP_EQ_PH, PseudoPICK_PH, v2i16, SETEQ>;
def : DSPSetCCPat<PseudoCMP_LT_PH, PseudoPICK_PH, v2i16, SETLT>;
def : DSPSetCCPat<PseudoCMP_LE_PH, PseudoPICK_PH, v2i16, SETLE>;
def : DSPSetCCPatInv<PseudoCMP_EQ_PH, PseudoPICK_PH, v2i16, SETNE>;
def : DSPSetCCPatInv<PseudoCMP_LT_PH, PseudoPICK_PH, v2i16, SETGE>;
def : DSPSetCCPatInv<PseudoCMP_LE_PH, PseudoPICK_PH, v2i16, SETGT>;
def : DSPSetCCPat<PseudoCMPU_EQ_QB, PseudoPICK_QB, v4i8, SETEQ>;
def : DSPSetCCPat<PseudoCMPU_LT_QB, PseudoPICK_QB, v4i8, SETULT>;
def : DSPSetCCPat<PseudoCMPU_LE_QB, PseudoPICK_QB, v4i8, SETULE>;
def : DSPSetCCPatInv<PseudoCMPU_EQ_QB, PseudoPICK_QB, v4i8, SETNE>;
def : DSPSetCCPatInv<PseudoCMPU_LT_QB, PseudoPICK_QB, v4i8, SETUGE>;
def : DSPSetCCPatInv<PseudoCMPU_LE_QB, PseudoPICK_QB, v4i8, SETUGT>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1399-1410
```tablegen
def : DSPSelectCCPat<PseudoCMP_EQ_PH, PseudoPICK_PH, v2i16, SETEQ>;
def : DSPSelectCCPat<PseudoCMP_LT_PH, PseudoPICK_PH, v2i16, SETLT>;
def : DSPSelectCCPat<PseudoCMP_LE_PH, PseudoPICK_PH, v2i16, SETLE>;
def : DSPSelectCCPatInv<PseudoCMP_EQ_PH, PseudoPICK_PH, v2i16, SETNE>;
def : DSPSelectCCPatInv<PseudoCMP_LT_PH, PseudoPICK_PH, v2i16, SETGE>;
def : DSPSelectCCPatInv<PseudoCMP_LE_PH, PseudoPICK_PH, v2i16, SETGT>;
def : DSPSelectCCPat<PseudoCMPU_EQ_QB, PseudoPICK_QB, v4i8, SETEQ>;
def : DSPSelectCCPat<PseudoCMPU_LT_QB, PseudoPICK_QB, v4i8, SETULT>;
def : DSPSelectCCPat<PseudoCMPU_LE_QB, PseudoPICK_QB, v4i8, SETULE>;
def : DSPSelectCCPatInv<PseudoCMPU_EQ_QB, PseudoPICK_QB, v4i8, SETNE>;
def : DSPSelectCCPatInv<PseudoCMPU_LT_QB, PseudoPICK_QB, v4i8, SETUGE>;
def : DSPSelectCCPatInv<PseudoCMPU_LE_QB, PseudoPICK_QB, v4i8, SETUGT>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1412-1415
```tablegen
// Extr patterns.
class EXTR_W_TY1_R2_Pat<SDPatternOperator OpNode, Instruction Instr> :
  DSPPat<(i32 (OpNode GPR32:$rs, ACC64DSP:$ac)),
         (Instr ACC64DSP:$ac, GPR32:$rs)>;
```
- EN: Declares reusable TableGen class `EXTR_W_TY1_R2_Pat` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `EXTR_W_TY1_R2_Pat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1417-1419
```tablegen
class EXTR_W_TY1_R1_Pat<SDPatternOperator OpNode, Instruction Instr> :
  DSPPat<(i32 (OpNode immZExt5:$shift, ACC64DSP:$ac)),
         (Instr ACC64DSP:$ac, immZExt5:$shift)>;
```
- EN: Declares reusable TableGen class `EXTR_W_TY1_R1_Pat` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `EXTR_W_TY1_R1_Pat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1421-1432
```tablegen
def : EXTR_W_TY1_R1_Pat<MipsEXTP, EXTP>;
def : EXTR_W_TY1_R2_Pat<MipsEXTP, EXTPV>;
def : EXTR_W_TY1_R1_Pat<MipsEXTPDP, EXTPDP>;
def : EXTR_W_TY1_R2_Pat<MipsEXTPDP, EXTPDPV>;
def : EXTR_W_TY1_R1_Pat<MipsEXTR_W, EXTR_W>;
def : EXTR_W_TY1_R2_Pat<MipsEXTR_W, EXTRV_W>;
def : EXTR_W_TY1_R1_Pat<MipsEXTR_R_W, EXTR_R_W>;
def : EXTR_W_TY1_R2_Pat<MipsEXTR_R_W, EXTRV_R_W>;
def : EXTR_W_TY1_R1_Pat<MipsEXTR_RS_W, EXTR_RS_W>;
def : EXTR_W_TY1_R2_Pat<MipsEXTR_RS_W, EXTRV_RS_W>;
def : EXTR_W_TY1_R1_Pat<MipsEXTR_S_H, EXTR_S_H>;
def : EXTR_W_TY1_R2_Pat<MipsEXTR_S_H, EXTRV_S_H>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1434-1437
```tablegen
// Indexed load patterns.
class IndexedLoadPat<SDPatternOperator LoadNode, Instruction Instr> :
  DSPPat<(i32 (LoadNode (add i32:$base, i32:$index))),
         (Instr i32:$base, i32:$index)>;
```
- EN: Declares reusable TableGen class `IndexedLoadPat` for `MipsDSPInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrInfo` 声明可复用的 TableGen 类 `IndexedLoadPat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1439-1443
```tablegen
let AddedComplexity = 20 in {
  def : IndexedLoadPat<zextloadi8, LBUX>;
  def : IndexedLoadPat<sextloadi16, LHX>;
  def : IndexedLoadPat<load, LWX>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1445-1448
```tablegen
// Instruction alias.
let AdditionalPredicates = [NotInMicroMips] in {
  def : DSPInstAlias<"wrdsp $rt", (WRDSP GPR32Opnd:$rt, 0x1F), 1>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

## Key Concepts / 关键概念

- EN: Primary role: instruction semantics, scheduling hints, and machine-level helpers.
  - CN: 核心职责：指令语义、调度提示以及机器级辅助逻辑。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: SelectionDAG patterns or node profiles connect IR-level intent to target instructions.
  - CN: SelectionDAG 模式或节点轮廓把 IR 层意图连接到目标指令。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
