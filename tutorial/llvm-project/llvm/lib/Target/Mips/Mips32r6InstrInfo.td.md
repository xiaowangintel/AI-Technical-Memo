# Mips32r6InstrInfo.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/Mips32r6InstrInfo.td`
- Repository: `llvm-project`
- Purpose (EN): This file describes Mips32r6 instructions.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `Mips32r6InstrInfo`，涵盖指令语义、调度提示以及机器级辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//=- Mips32r6InstrInfo.td - Mips32r6 Instruction Information -*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes Mips32r6 instructions.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-13
```tablegen
include "Mips32r6InstrFormats.td"
```
- EN: Pulls in other TableGen fragments so this file can reuse common records, predicates, and target-wide definitions.
- CN: 这里引入其他 TableGen 片段，以便复用公共记录、谓词以及整个目标范围内的定义。

### Lines 15-19
```tablegen
//===----------------------------------------------------------------------===//
//
// Mips profiles and nodes
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 21-23
```tablegen
def SDT_MipsFSelect : SDTypeProfile<1, 3, [SDTCisFP<1>,
                                           SDTCisSameAs<0,2>,
                                           SDTCisSameAs<2,3>]>;
```
- EN: Defines TableGen record `SDT_MipsFSelect` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_MipsFSelect`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 25-26
```tablegen
// Floating point select
def MipsFSelect : SDNode<"MipsISD::FSELECT", SDT_MipsFSelect>;
```
- EN: Defines TableGen record `MipsFSelect` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsFSelect`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 28-32
```tablegen
//===----------------------------------------------------------------------===//
//
// Mips Operands
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 34-36
```tablegen
// Notes about removals/changes from MIPS32r6:
// Reencoded: jr -> jalr
// Reencoded: jr.hb -> jalr.hb
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 38-44
```tablegen
def brtarget21 : Operand<OtherVT> {
  let EncoderMethod = "getBranchTarget21OpValue";
  let OperandType = "OPERAND_PCREL";
  let DecoderMethod = "DecodeBranchTarget21";
  let ParserMatchClass = MipsJumpTargetAsmOperand;
  let PrintMethod = "printBranchOperand";
}
```
- EN: Defines TableGen record `brtarget21` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `brtarget21`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 46-52
```tablegen
def brtarget26 : Operand<OtherVT> {
  let EncoderMethod = "getBranchTarget26OpValue";
  let OperandType = "OPERAND_PCREL";
  let DecoderMethod = "DecodeBranchTarget26";
  let ParserMatchClass = MipsJumpTargetAsmOperand;
  let PrintMethod = "printBranchOperand";
}
```
- EN: Defines TableGen record `brtarget26` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `brtarget26`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 54-57
```tablegen
def jmpoffset16 : Operand<OtherVT> {
  let EncoderMethod = "getJumpOffset16OpValue";
  let ParserMatchClass = MipsJumpTargetAsmOperand;
}
```
- EN: Defines TableGen record `jmpoffset16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `jmpoffset16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 59-62
```tablegen
def calloffset16 : Operand<iPTR> {
  let EncoderMethod = "getJumpOffset16OpValue";
  let ParserMatchClass = MipsJumpTargetAsmOperand;
}
```
- EN: Defines TableGen record `calloffset16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `calloffset16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 64-68
```tablegen
//===----------------------------------------------------------------------===//
//
// Instruction Encodings
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 70-74
```tablegen
class ADDIUPC_ENC : PCREL19_FM<OPCODE2_ADDIUPC>;
class ALIGN_ENC  : SPECIAL3_ALIGN_FM<OPCODE6_ALIGN>;
class ALUIPC_ENC : PCREL16_FM<OPCODE5_ALUIPC>;
class AUI_ENC    : AUI_FM;
class AUIPC_ENC  : PCREL16_FM<OPCODE5_AUIPC>;
```
- EN: Declares reusable TableGen class `ADDIUPC_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `ADDIUPC_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 76-87
```tablegen
class BAL_ENC   : BAL_FM;
class NAL_ENC   : NAL_FM;
class BALC_ENC  : BRANCH_OFF26_FM<0b111010>;
class BC_ENC    : BRANCH_OFF26_FM<0b110010>;
class BEQC_ENC  : CMP_BRANCH_2R_OFF16_FM<OPGROUP_ADDI>,
                  DecodeDisambiguates<"AddiGroupBranch">;
class BEQZALC_ENC : CMP_BRANCH_1R_RT_OFF16_FM<OPGROUP_ADDI>,
                    DecodeDisambiguatedBy<"DaddiGroupBranch">;
class BNEC_ENC  : CMP_BRANCH_2R_OFF16_FM<OPGROUP_DADDI>,
                  DecodeDisambiguates<"DaddiGroupBranch">;
class BNEZALC_ENC : CMP_BRANCH_1R_RT_OFF16_FM<OPGROUP_DADDI>,
                    DecodeDisambiguatedBy<"DaddiGroupBranch">;
```
- EN: Declares reusable TableGen class `BAL_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BAL_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 89-98
```tablegen
class BLTZC_ENC : CMP_BRANCH_1R_BOTH_OFF16_FM<OPGROUP_BGTZL>,
                  DecodeDisambiguates<"BgtzlGroupBranch">;
class BGEC_ENC  : CMP_BRANCH_2R_OFF16_FM<OPGROUP_BLEZL>,
                  DecodeDisambiguatedBy<"BlezlGroupBranch">;
class BGEUC_ENC : CMP_BRANCH_2R_OFF16_FM<OPGROUP_BLEZ>,
                  DecodeDisambiguatedBy<"BlezGroupBranch">;
class BGEZC_ENC : CMP_BRANCH_1R_BOTH_OFF16_FM<OPGROUP_BLEZL>,
                  DecodeDisambiguates<"BlezlGroupBranch">;
class BGTZALC_ENC : CMP_BRANCH_1R_RT_OFF16_FM<OPGROUP_BGTZ>,
                    DecodeDisambiguatedBy<"BgtzGroupBranch">;
```
- EN: Declares reusable TableGen class `BLTZC_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BLTZC_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 100-103
```tablegen
class BLTC_ENC : CMP_BRANCH_2R_OFF16_FM<OPGROUP_BGTZL>,
                 DecodeDisambiguatedBy<"BgtzlGroupBranch">;
class BLTUC_ENC : CMP_BRANCH_2R_OFF16_FM<OPGROUP_BGTZ>,
                  DecodeDisambiguatedBy<"BgtzGroupBranch">;
```
- EN: Declares reusable TableGen class `BLTC_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BLTC_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 105-110
```tablegen
class BLEZC_ENC : CMP_BRANCH_1R_RT_OFF16_FM<OPGROUP_BLEZL>,
                  DecodeDisambiguatedBy<"BlezlGroupBranch">;
class BLTZALC_ENC : CMP_BRANCH_1R_BOTH_OFF16_FM<OPGROUP_BGTZ>,
                    DecodeDisambiguates<"BgtzGroupBranch">;
class BGTZC_ENC : CMP_BRANCH_1R_RT_OFF16_FM<OPGROUP_BGTZL>,
                  DecodeDisambiguatedBy<"BgtzlGroupBranch">;
```
- EN: Declares reusable TableGen class `BLEZC_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BLEZC_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 112-115
```tablegen
class BEQZC_ENC : CMP_BRANCH_OFF21_FM<0b110110>;
class BGEZALC_ENC : CMP_BRANCH_1R_BOTH_OFF16_FM<OPGROUP_BLEZ>,
                    DecodeDisambiguates<"BlezGroupBranch">;
class BNEZC_ENC : CMP_BRANCH_OFF21_FM<0b111110>;
```
- EN: Declares reusable TableGen class `BEQZC_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BEQZC_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 117-120
```tablegen
class BC1EQZ_ENC : COP1_BCCZ_FM<OPCODE5_BC1EQZ>;
class BC1NEZ_ENC : COP1_BCCZ_FM<OPCODE5_BC1NEZ>;
class BC2EQZ_ENC : COP2_BCCZ_FM<OPCODE5_BC2EQZ>;
class BC2NEZ_ENC : COP2_BCCZ_FM<OPCODE5_BC2NEZ>;
```
- EN: Declares reusable TableGen class `BC1EQZ_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BC1EQZ_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 122-123
```tablegen
class DVP_ENC : COP0_EVP_DVP_FM<0b1>;
class EVP_ENC : COP0_EVP_DVP_FM<0b0>;
```
- EN: Declares reusable TableGen class `DVP_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `DVP_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 125-142
```tablegen
class JIALC_ENC : JMP_IDX_COMPACT_FM<0b111110>;
class JIC_ENC   : JMP_IDX_COMPACT_FM<0b110110>;
class JR_HB_R6_ENC : JR_HB_R6_FM<OPCODE6_JALR>;
class BITSWAP_ENC : SPECIAL3_2R_FM<OPCODE6_BITSWAP>;
class BLEZALC_ENC : CMP_BRANCH_1R_RT_OFF16_FM<OPGROUP_BLEZ>,
                    DecodeDisambiguatedBy<"BlezGroupBranch">;
class BNVC_ENC   : CMP_BRANCH_2R_OFF16_FM<OPGROUP_DADDI>,
                   DecodeDisambiguatedBy<"DaddiGroupBranch">;
class BOVC_ENC   : CMP_BRANCH_2R_OFF16_FM<OPGROUP_ADDI>,
                   DecodeDisambiguatedBy<"AddiGroupBranch">;
class DIV_ENC    : SPECIAL_3R_FM<0b00010, 0b011010>;
class DIVU_ENC   : SPECIAL_3R_FM<0b00010, 0b011011>;
class MOD_ENC    : SPECIAL_3R_FM<0b00011, 0b011010>;
class MODU_ENC   : SPECIAL_3R_FM<0b00011, 0b011011>;
class MUH_ENC    : SPECIAL_3R_FM<0b00011, 0b011000>;
class MUHU_ENC   : SPECIAL_3R_FM<0b00011, 0b011001>;
class MUL_R6_ENC : SPECIAL_3R_FM<0b00010, 0b011000>;
class MULU_ENC   : SPECIAL_3R_FM<0b00010, 0b011001>;
```
- EN: Declares reusable TableGen class `JIALC_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `JIALC_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 144-147
```tablegen
class MADDF_S_ENC  : COP1_3R_FM<0b011000, FIELD_FMT_S>;
class MADDF_D_ENC  : COP1_3R_FM<0b011000, FIELD_FMT_D>;
class MSUBF_S_ENC  : COP1_3R_FM<0b011001, FIELD_FMT_S>;
class MSUBF_D_ENC  : COP1_3R_FM<0b011001, FIELD_FMT_D>;
```
- EN: Declares reusable TableGen class `MADDF_S_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `MADDF_S_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 149-150
```tablegen
class SEL_D_ENC  : COP1_3R_FM<0b010000, FIELD_FMT_D>;
class SEL_S_ENC  : COP1_3R_FM<0b010000, FIELD_FMT_S>;
```
- EN: Declares reusable TableGen class `SEL_D_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `SEL_D_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 152-153
```tablegen
class SELEQZ_ENC : SPECIAL_3R_FM<0b00000, 0b110101>;
class SELNEZ_ENC : SPECIAL_3R_FM<0b00000, 0b110111>;
```
- EN: Declares reusable TableGen class `SELEQZ_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `SELEQZ_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 155-155
```tablegen
class LWPC_ENC   : PCREL19_FM<OPCODE2_LWPC>;
```
- EN: Declares reusable TableGen class `LWPC_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `LWPC_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 157-160
```tablegen
class MAX_S_ENC : COP1_3R_FM<0b011110, FIELD_FMT_S>;
class MAX_D_ENC : COP1_3R_FM<0b011110, FIELD_FMT_D>;
class MIN_S_ENC : COP1_3R_FM<0b011100, FIELD_FMT_S>;
class MIN_D_ENC : COP1_3R_FM<0b011100, FIELD_FMT_D>;
```
- EN: Declares reusable TableGen class `MAX_S_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `MAX_S_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 162-165
```tablegen
class MAXA_S_ENC : COP1_3R_FM<0b011111, FIELD_FMT_S>;
class MAXA_D_ENC : COP1_3R_FM<0b011111, FIELD_FMT_D>;
class MINA_S_ENC : COP1_3R_FM<0b011101, FIELD_FMT_S>;
class MINA_D_ENC : COP1_3R_FM<0b011101, FIELD_FMT_D>;
```
- EN: Declares reusable TableGen class `MAXA_S_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `MAXA_S_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 167-170
```tablegen
class SELEQZ_S_ENC : COP1_3R_FM<0b010100, FIELD_FMT_S>;
class SELEQZ_D_ENC : COP1_3R_FM<0b010100, FIELD_FMT_D>;
class SELNEZ_S_ENC : COP1_3R_FM<0b010111, FIELD_FMT_S>;
class SELNEZ_D_ENC : COP1_3R_FM<0b010111, FIELD_FMT_D>;
```
- EN: Declares reusable TableGen class `SELEQZ_S_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `SELEQZ_S_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 172-175
```tablegen
class RINT_S_ENC : COP1_2R_FM<0b011010, FIELD_FMT_S>;
class RINT_D_ENC : COP1_2R_FM<0b011010, FIELD_FMT_D>;
class CLASS_S_ENC : COP1_2R_FM<0b011011, FIELD_FMT_S>;
class CLASS_D_ENC : COP1_2R_FM<0b011011, FIELD_FMT_D>;
```
- EN: Declares reusable TableGen class `RINT_S_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `RINT_S_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 177-178
```tablegen
class CACHE_ENC : SPECIAL3_MEM_FM<OPCODE6_CACHE>;
class PREF_ENC : SPECIAL3_MEM_FM<OPCODE6_PREF>;
```
- EN: Declares reusable TableGen class `CACHE_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `CACHE_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 180-183
```tablegen
class LDC2_R6_ENC : COP2LDST_FM<OPCODE5_LDC2>;
class LWC2_R6_ENC : COP2LDST_FM<OPCODE5_LWC2>;
class SDC2_R6_ENC : COP2LDST_FM<OPCODE5_SDC2>;
class SWC2_R6_ENC : COP2LDST_FM<OPCODE5_SWC2>;
```
- EN: Declares reusable TableGen class `LDC2_R6_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `LDC2_R6_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 185-185
```tablegen
class LSA_R6_ENC : SPECIAL_LSA_FM<OPCODE6_LSA>;
```
- EN: Declares reusable TableGen class `LSA_R6_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `LSA_R6_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 187-188
```tablegen
class LL_R6_ENC : SPECIAL3_LL_SC_FM<OPCODE6_LL>;
class SC_R6_ENC : SPECIAL3_LL_SC_FM<OPCODE6_SC>;
```
- EN: Declares reusable TableGen class `LL_R6_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `LL_R6_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 190-191
```tablegen
class CLO_R6_ENC : SPECIAL_2R_FM<OPCODE6_CLO>;
class CLZ_R6_ENC : SPECIAL_2R_FM<OPCODE6_CLZ>;
```
- EN: Declares reusable TableGen class `CLO_R6_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `CLO_R6_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 193-193
```tablegen
class SDBBP_R6_ENC : SPECIAL_SDBBP_FM;
```
- EN: Declares reusable TableGen class `SDBBP_R6_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `SDBBP_R6_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 195-200
```tablegen
class CRC32B_ENC  : SPECIAL3_2R_SZ_CRC<0,0>;
class CRC32H_ENC  : SPECIAL3_2R_SZ_CRC<1,0>;
class CRC32W_ENC  : SPECIAL3_2R_SZ_CRC<2,0>;
class CRC32CB_ENC : SPECIAL3_2R_SZ_CRC<0,1>;
class CRC32CH_ENC : SPECIAL3_2R_SZ_CRC<1,1>;
class CRC32CW_ENC : SPECIAL3_2R_SZ_CRC<2,1>;
```
- EN: Declares reusable TableGen class `CRC32B_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `CRC32B_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 202-203
```tablegen
class GINVI_ENC : SPECIAL3_GINV<0>;
class GINVT_ENC : SPECIAL3_GINV<2>;
```
- EN: Declares reusable TableGen class `GINVI_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `GINVI_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 205-205
```tablegen
class SIGRIE_ENC : SIGRIE_FM;
```
- EN: Declares reusable TableGen class `SIGRIE_ENC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `SIGRIE_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 207-211
```tablegen
//===----------------------------------------------------------------------===//
//
// Instruction Multiclasses
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 213-222
```tablegen
class CMP_CONDN_DESC_BASE<string CondStr, string Typestr,
                          RegisterOperand FGROpnd,
                          RegisterOperand FGRCCOpnd,
                          SDPatternOperator Op = null_frag> {
  dag OutOperandList = (outs FGRCCOpnd:$fd);
  dag InOperandList = (ins FGROpnd:$fs, FGROpnd:$ft);
  string AsmString = !strconcat("cmp.", CondStr, ".", Typestr, "\t$fd, $fs, $ft");
  list<dag> Pattern = [(set FGRCCOpnd:$fd, (Op FGROpnd:$fs, FGROpnd:$ft))];
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `CMP_CONDN_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `CMP_CONDN_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 224-241
```tablegen
multiclass CMP_CC_M <FIELD_CMP_FORMAT Format, string Typestr,
                     RegisterOperand FGROpnd,
                     RegisterOperand FGRCCOpnd>{
  let AdditionalPredicates = [NotInMicroMips] in {
    def CMP_F_#NAME : R6MMR6Rel, COP1_CMP_CONDN_FM<Format, FIELD_CMP_COND_AF>,
                      CMP_CONDN_DESC_BASE<"af", Typestr, FGROpnd, FGRCCOpnd>,
                      MipsR6Arch<!strconcat("cmp.af.", Typestr)>,
                      ISA_MIPS32R6, HARDFLOAT;
    def CMP_UN_#NAME : R6MMR6Rel, COP1_CMP_CONDN_FM<Format, FIELD_CMP_COND_UN>,
                       CMP_CONDN_DESC_BASE<"un", Typestr, FGROpnd, FGRCCOpnd, setuo>,
                       MipsR6Arch<!strconcat("cmp.un.", Typestr)>,
                       ISA_MIPS32R6, HARDFLOAT;
    def CMP_EQ_#NAME : R6MMR6Rel, COP1_CMP_CONDN_FM<Format, FIELD_CMP_COND_EQ>,
                       CMP_CONDN_DESC_BASE<"eq", Typestr, FGROpnd, FGRCCOpnd,
                                           setoeq>,
                       MipsR6Arch<!strconcat("cmp.eq.", Typestr)>,
                       ISA_MIPS32R6, HARDFLOAT;
    def CMP_UEQ_#NAME : R6MMR6Rel, COP1_CMP_CONDN_FM<Format,
```
- EN: Declares TableGen `multiclass CMP_CC_M`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass CMP_CC_M`，它是一个可复用模板，可展开为多个相关记录。

### Lines 242-259
```tablegen
                                                     FIELD_CMP_COND_UEQ>,
                        CMP_CONDN_DESC_BASE<"ueq", Typestr, FGROpnd, FGRCCOpnd,
                                            setueq>,
                        MipsR6Arch<!strconcat("cmp.ueq.", Typestr)>,
                        ISA_MIPS32R6, HARDFLOAT;
    def CMP_LT_#NAME : R6MMR6Rel, COP1_CMP_CONDN_FM<Format, FIELD_CMP_COND_LT>,
                       CMP_CONDN_DESC_BASE<"lt", Typestr, FGROpnd, FGRCCOpnd,
                                           setolt>,
                       MipsR6Arch<!strconcat("cmp.lt.", Typestr)>,
                       ISA_MIPS32R6, HARDFLOAT;
    def CMP_ULT_#NAME : R6MMR6Rel, COP1_CMP_CONDN_FM<Format,
                                                     FIELD_CMP_COND_ULT>,
                        CMP_CONDN_DESC_BASE<"ult", Typestr, FGROpnd, FGRCCOpnd,
                                            setult>,
                        MipsR6Arch<!strconcat("cmp.ult.", Typestr)>,
                        ISA_MIPS32R6, HARDFLOAT;
    def CMP_LE_#NAME : R6MMR6Rel, COP1_CMP_CONDN_FM<Format, FIELD_CMP_COND_LE>,
                       CMP_CONDN_DESC_BASE<"le", Typestr, FGROpnd, FGRCCOpnd,
```
- EN: Defines TableGen record `CMP_LT_` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CMP_LT_`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 260-277
```tablegen
                                           setole>,
                       MipsR6Arch<!strconcat("cmp.le.", Typestr)>,
                       ISA_MIPS32R6, HARDFLOAT;
    def CMP_ULE_#NAME : R6MMR6Rel, COP1_CMP_CONDN_FM<Format,
                                                     FIELD_CMP_COND_ULE>,
                        CMP_CONDN_DESC_BASE<"ule", Typestr, FGROpnd, FGRCCOpnd,
                                            setule>,
                        MipsR6Arch<!strconcat("cmp.ule.", Typestr)>,
                        ISA_MIPS32R6, HARDFLOAT;
    let mayRaiseFPException = 1 in { 
      def CMP_SAF_#NAME : R6MMR6Rel, COP1_CMP_CONDN_FM<Format,
                                                       FIELD_CMP_COND_SAF>,
                          CMP_CONDN_DESC_BASE<"saf", Typestr, FGROpnd, FGRCCOpnd>,
                          MipsR6Arch<!strconcat("cmp.saf.", Typestr)>,
                          ISA_MIPS32R6, HARDFLOAT;
      def CMP_SUN_#NAME : R6MMR6Rel, COP1_CMP_CONDN_FM<Format,
                                                       FIELD_CMP_COND_SUN>,
                          CMP_CONDN_DESC_BASE<"sun", Typestr, FGROpnd, FGRCCOpnd>,
```
- EN: Defines TableGen record `CMP_ULE_` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CMP_ULE_`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 278-295
```tablegen
                          MipsR6Arch<!strconcat("cmp.sun.", Typestr)>,
                          ISA_MIPS32R6, HARDFLOAT;
      def CMP_SEQ_#NAME : R6MMR6Rel, COP1_CMP_CONDN_FM<Format,
                                                       FIELD_CMP_COND_SEQ>,
                          CMP_CONDN_DESC_BASE<"seq", Typestr, FGROpnd, FGRCCOpnd>,
                          MipsR6Arch<!strconcat("cmp.seq.", Typestr)>,
                          ISA_MIPS32R6, HARDFLOAT;
      def CMP_SUEQ_#NAME : R6MMR6Rel, COP1_CMP_CONDN_FM<Format,
                                                        FIELD_CMP_COND_SUEQ>,
                           CMP_CONDN_DESC_BASE<"sueq", Typestr, FGROpnd, FGRCCOpnd>,
                           MipsR6Arch<!strconcat("cmp.sueq.", Typestr)>,
                           ISA_MIPS32R6, HARDFLOAT;
      def CMP_SLT_#NAME : R6MMR6Rel, COP1_CMP_CONDN_FM<Format,
                                                       FIELD_CMP_COND_SLT>,
                          CMP_CONDN_DESC_BASE<"slt", Typestr, FGROpnd, FGRCCOpnd>,
                          MipsR6Arch<!strconcat("cmp.slt.", Typestr)>,
                          ISA_MIPS32R6, HARDFLOAT;
      def CMP_SULT_#NAME : R6MMR6Rel, COP1_CMP_CONDN_FM<Format,
```
- EN: Defines TableGen record `CMP_SEQ_` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CMP_SEQ_`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 296-312
```tablegen
                                                        FIELD_CMP_COND_SULT>,
                           CMP_CONDN_DESC_BASE<"sult", Typestr, FGROpnd, FGRCCOpnd>,
                           MipsR6Arch<!strconcat("cmp.sult.", Typestr)>,
                           ISA_MIPS32R6, HARDFLOAT;
      def CMP_SLE_#NAME : R6MMR6Rel, COP1_CMP_CONDN_FM<Format,
                                                       FIELD_CMP_COND_SLE>,
                          CMP_CONDN_DESC_BASE<"sle", Typestr, FGROpnd, FGRCCOpnd>,
                          MipsR6Arch<!strconcat("cmp.sle.", Typestr)>,
                          ISA_MIPS32R6, HARDFLOAT;
      def CMP_SULE_#NAME : R6MMR6Rel, COP1_CMP_CONDN_FM<Format,
                                                        FIELD_CMP_COND_SULE>,
                           CMP_CONDN_DESC_BASE<"sule", Typestr, FGROpnd, FGRCCOpnd>,
                           MipsR6Arch<!strconcat("cmp.sule.", Typestr)>,
                           ISA_MIPS32R6, HARDFLOAT;
    }
  }
}
```
- EN: Defines TableGen record `CMP_SLE_` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CMP_SLE_`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 314-318
```tablegen
//===----------------------------------------------------------------------===//
//
// Instruction Descriptions
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 320-327
```tablegen
class PCREL_DESC_BASE<string instr_asm, RegisterOperand GPROpnd,
                      Operand ImmOpnd>
      : MipsR6Arch<instr_asm> {
  dag OutOperandList = (outs GPROpnd:$rs);
  dag InOperandList = (ins ImmOpnd:$imm);
  string AsmString = !strconcat(instr_asm, "\t$rs, $imm");
  list<dag> Pattern = [];
}
```
- EN: Declares reusable TableGen class `PCREL_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `PCREL_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 329-330
```tablegen
class ADDIUPC_DESC : PCREL_DESC_BASE<"addiupc", GPR32Opnd, simm19_lsl2>;
class LWPC_DESC: PCREL_DESC_BASE<"lwpc", GPR32Opnd, simm19_lsl2>;
```
- EN: Declares reusable TableGen class `ADDIUPC_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `ADDIUPC_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 332-339
```tablegen
class ALIGN_DESC_BASE<string instr_asm, RegisterOperand GPROpnd,
                      Operand ImmOpnd>
      : MipsR6Arch<instr_asm> {
  dag OutOperandList = (outs GPROpnd:$rd);
  dag InOperandList = (ins GPROpnd:$rs, GPROpnd:$rt, ImmOpnd:$bp);
  string AsmString = !strconcat(instr_asm, "\t$rd, $rs, $rt, $bp");
  list<dag> Pattern = [];
}
```
- EN: Declares reusable TableGen class `ALIGN_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `ALIGN_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 341-341
```tablegen
class ALIGN_DESC : ALIGN_DESC_BASE<"align", GPR32Opnd, uimm2>;
```
- EN: Declares reusable TableGen class `ALIGN_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `ALIGN_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 343-349
```tablegen
class ALUIPC_DESC_BASE<string instr_asm, RegisterOperand GPROpnd>
      : MipsR6Arch<instr_asm> {
  dag OutOperandList = (outs GPROpnd:$rs);
  dag InOperandList = (ins simm16:$imm);
  string AsmString = !strconcat(instr_asm, "\t$rs, $imm");
  list<dag> Pattern = [];
}
```
- EN: Declares reusable TableGen class `ALUIPC_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `ALUIPC_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 351-352
```tablegen
class ALUIPC_DESC : ALUIPC_DESC_BASE<"aluipc", GPR32Opnd>;
class AUIPC_DESC : ALUIPC_DESC_BASE<"auipc", GPR32Opnd>;
```
- EN: Declares reusable TableGen class `ALUIPC_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `ALUIPC_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 354-360
```tablegen
class AUI_DESC_BASE<string instr_asm, RegisterOperand GPROpnd>
      : MipsR6Arch<instr_asm> {
  dag OutOperandList = (outs GPROpnd:$rt);
  dag InOperandList = (ins GPROpnd:$rs, uimm16:$imm);
  string AsmString = !strconcat(instr_asm, "\t$rt, $rs, $imm");
  list<dag> Pattern = [];
}
```
- EN: Declares reusable TableGen class `AUI_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `AUI_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 362-362
```tablegen
class AUI_DESC : AUI_DESC_BASE<"aui", GPR32Opnd>;
```
- EN: Declares reusable TableGen class `AUI_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `AUI_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 364-369
```tablegen
class BRANCH_DESC_BASE {
  bit isBranch = 1;
  bit isTerminator = 1;
  bit hasDelaySlot = 0;
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `BRANCH_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BRANCH_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 371-378
```tablegen
class BC_DESC_BASE<string instr_asm, DAGOperand opnd> : BRANCH_DESC_BASE,
    MipsR6Arch<instr_asm> {
  dag InOperandList = (ins opnd:$offset);
  dag OutOperandList = (outs);
  string AsmString = !strconcat(instr_asm, "\t$offset");
  bit isBarrier = 1;
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `BC_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BC_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 380-384
```tablegen
class NAL_DESC_BASE<string instr_asm> : BRANCH_DESC_BASE,
    MipsR6Arch<instr_asm> {
  string AsmString = instr_asm;
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `NAL_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `NAL_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 386-395
```tablegen
class CMP_BC_DESC_BASE<string instr_asm, DAGOperand opnd,
                       RegisterOperand GPROpnd> : BRANCH_DESC_BASE,
                                                  MipsR6Arch<instr_asm> {
  dag InOperandList = (ins GPROpnd:$rs, GPROpnd:$rt, opnd:$offset);
  dag OutOperandList = (outs);
  string AsmString = !strconcat(instr_asm, "\t$rs, $rt, $offset");
  list<Register> Defs = [AT];
  bit hasForbiddenSlot = 1;
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `CMP_BC_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `CMP_BC_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 397-406
```tablegen
class CMP_CBR_EQNE_Z_DESC_BASE<string instr_asm, DAGOperand opnd,
                               RegisterOperand GPROpnd>
    : BRANCH_DESC_BASE, MipsR6Arch<instr_asm> {
  dag InOperandList = (ins GPROpnd:$rs, opnd:$offset);
  dag OutOperandList = (outs);
  string AsmString = !strconcat(instr_asm, "\t$rs, $offset");
  list<Register> Defs = [AT];
  bit hasForbiddenSlot = 1;
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `CMP_CBR_EQNE_Z_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `CMP_CBR_EQNE_Z_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 408-417
```tablegen
class CMP_CBR_RT_Z_DESC_BASE<string instr_asm, DAGOperand opnd,
                             RegisterOperand GPROpnd>
    : BRANCH_DESC_BASE, MipsR6Arch<instr_asm> {
  dag InOperandList = (ins GPROpnd:$rt, opnd:$offset);
  dag OutOperandList = (outs);
  string AsmString = !strconcat(instr_asm, "\t$rt, $offset");
  list<Register> Defs = [AT];
  bit hasForbiddenSlot = 1;
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `CMP_CBR_RT_Z_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `CMP_CBR_RT_Z_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 419-424
```tablegen
class BAL_DESC : BC_DESC_BASE<"bal", brtarget> {
  bit isCall = 1;
  bit hasDelaySlot = 1;
  list<Register> Defs = [RA];
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `BAL_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BAL_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 426-430
```tablegen
class NAL_DESC : NAL_DESC_BASE<"nal"> {
  bit hasDelaySlot = 1;
  list<Register> Defs = [RA];
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `NAL_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `NAL_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 432-436
```tablegen
class BALC_DESC : BC_DESC_BASE<"balc", brtarget26> {
  bit isCall = 1;
  list<Register> Defs = [RA];
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `BALC_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BALC_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 438-442
```tablegen
class BC_DESC : BC_DESC_BASE<"bc", brtarget26>;
class BGEC_DESC : CMP_BC_DESC_BASE<"bgec", brtarget, GPR32Opnd>;
class BGEUC_DESC : CMP_BC_DESC_BASE<"bgeuc", brtarget, GPR32Opnd>;
class BEQC_DESC : CMP_BC_DESC_BASE<"beqc", brtarget, GPR32Opnd>;
class BNEC_DESC : CMP_BC_DESC_BASE<"bnec", brtarget, GPR32Opnd>;
```
- EN: Declares reusable TableGen class `BC_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BC_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 444-445
```tablegen
class BLTC_DESC : CMP_BC_DESC_BASE<"bltc", brtarget, GPR32Opnd>;
class BLTUC_DESC : CMP_BC_DESC_BASE<"bltuc", brtarget, GPR32Opnd>;
```
- EN: Declares reusable TableGen class `BLTC_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BLTC_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 447-448
```tablegen
class BLTZC_DESC : CMP_CBR_RT_Z_DESC_BASE<"bltzc", brtarget, GPR32Opnd>;
class BGEZC_DESC : CMP_CBR_RT_Z_DESC_BASE<"bgezc", brtarget, GPR32Opnd>;
```
- EN: Declares reusable TableGen class `BLTZC_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BLTZC_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 450-451
```tablegen
class BLEZC_DESC : CMP_CBR_RT_Z_DESC_BASE<"blezc", brtarget, GPR32Opnd>;
class BGTZC_DESC : CMP_CBR_RT_Z_DESC_BASE<"bgtzc", brtarget, GPR32Opnd>;
```
- EN: Declares reusable TableGen class `BLEZC_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BLEZC_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 453-454
```tablegen
class BEQZC_DESC : CMP_CBR_EQNE_Z_DESC_BASE<"beqzc", brtarget21, GPR32Opnd>;
class BNEZC_DESC : CMP_CBR_EQNE_Z_DESC_BASE<"bnezc", brtarget21, GPR32Opnd>;
```
- EN: Declares reusable TableGen class `BEQZC_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BEQZC_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 456-461
```tablegen
class COP1_BCCZ_DESC_BASE<string instr_asm> : BRANCH_DESC_BASE {
  dag InOperandList = (ins FGR64Opnd:$ft, brtarget:$offset);
  dag OutOperandList = (outs);
  string AsmString = instr_asm;
  bit hasDelaySlot = 1;
}
```
- EN: Declares reusable TableGen class `COP1_BCCZ_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `COP1_BCCZ_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 463-464
```tablegen
class BC1EQZ_DESC : COP1_BCCZ_DESC_BASE<"bc1eqz $ft, $offset">;
class BC1NEZ_DESC : COP1_BCCZ_DESC_BASE<"bc1nez $ft, $offset">;
```
- EN: Declares reusable TableGen class `BC1EQZ_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BC1EQZ_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 466-472
```tablegen
class COP2_BCCZ_DESC_BASE<string instr_asm> : BRANCH_DESC_BASE {
  dag InOperandList = (ins COP2Opnd:$ct, brtarget:$offset);
  dag OutOperandList = (outs);
  string AsmString = instr_asm;
  bit hasDelaySlot = 1;
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `COP2_BCCZ_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `COP2_BCCZ_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 474-475
```tablegen
class BC2EQZ_DESC : COP2_BCCZ_DESC_BASE<"bc2eqz $ct, $offset">;
class BC2NEZ_DESC : COP2_BCCZ_DESC_BASE<"bc2nez $ct, $offset">;
```
- EN: Declares reusable TableGen class `BC2EQZ_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BC2EQZ_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 477-478
```tablegen
class BOVC_DESC   : CMP_BC_DESC_BASE<"bovc", brtarget, GPR32Opnd>;
class BNVC_DESC   : CMP_BC_DESC_BASE<"bnvc", brtarget, GPR32Opnd>;
```
- EN: Declares reusable TableGen class `BOVC_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BOVC_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 480-490
```tablegen
class JMP_IDX_COMPACT_DESC_BASE<string opstr, DAGOperand opnd,
                                RegisterOperand GPROpnd>
    : MipsR6Arch<opstr> {
  dag InOperandList = (ins GPROpnd:$rt, opnd:$offset);
  string AsmString = !strconcat(opstr, "\t$rt, $offset");
  list<dag> Pattern = [];
  bit hasDelaySlot = 0;
  bit isCTI = 1;
  bit isBranch = 1;
  bit isIndirectBranch = 1;
}
```
- EN: Declares reusable TableGen class `JMP_IDX_COMPACT_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `JMP_IDX_COMPACT_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 492-496
```tablegen
class JIALC_DESC : JMP_IDX_COMPACT_DESC_BASE<"jialc", calloffset16,
                                             GPR32Opnd> {
  bit isCall = 1;
  list<Register> Defs = [RA];
}
```
- EN: Declares reusable TableGen class `JIALC_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `JIALC_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 498-503
```tablegen
class JIC_DESC : JMP_IDX_COMPACT_DESC_BASE<"jic", jmpoffset16,
                                           GPR32Opnd> {
  bit isBarrier = 1;
  bit isTerminator = 1;
  list<Register> Defs = [AT];
}
```
- EN: Declares reusable TableGen class `JIC_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `JIC_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 505-512
```tablegen
class JR_HB_R6_DESC : JR_HB_DESC_BASE<"jr.hb", GPR32Opnd> {
  bit isBranch = 1;
  bit isIndirectBranch = 1;
  bit hasDelaySlot = 1;
  bit isTerminator=1;
  bit isBarrier=1;
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `JR_HB_R6_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `JR_HB_R6_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 514-520
```tablegen
class BITSWAP_DESC_BASE<string instr_asm, RegisterOperand GPROpnd>
    : MipsR6Arch<instr_asm> {
  dag OutOperandList = (outs GPROpnd:$rd);
  dag InOperandList = (ins GPROpnd:$rt);
  string AsmString = !strconcat(instr_asm, "\t$rd, $rt");
  list<dag> Pattern = [];
}
```
- EN: Declares reusable TableGen class `BITSWAP_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BITSWAP_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 522-522
```tablegen
class BITSWAP_DESC : BITSWAP_DESC_BASE<"bitswap", GPR32Opnd>;
```
- EN: Declares reusable TableGen class `BITSWAP_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BITSWAP_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 524-534
```tablegen
class DIVMOD_DESC_BASE<string instr_asm, RegisterOperand GPROpnd,
                       SDPatternOperator Op=null_frag>
    : MipsR6Arch<instr_asm> {
  dag OutOperandList = (outs GPROpnd:$rd);
  dag InOperandList = (ins GPROpnd:$rs, GPROpnd:$rt);
  string AsmString = !strconcat(instr_asm, "\t$rd, $rs, $rt");
  list<dag> Pattern = [(set GPROpnd:$rd, (Op GPROpnd:$rs, GPROpnd:$rt))];
  // This instruction doesn't trap division by zero itself. We must insert
  // teq instructions as well.
  bit usesCustomInserter = 1;
}
```
- EN: Declares reusable TableGen class `DIVMOD_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `DIVMOD_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 536-543
```tablegen
class DVPEVP_DESC_BASE<string instr_asm>
    : MipsR6Arch<instr_asm> {
  dag OutOperandList = (outs GPR32Opnd:$rt);
  dag InOperandList = (ins);
  string AsmString = !strconcat(instr_asm, "\t$rt");
  list<dag> Pattern = [];
  bit hasUnModeledSideEffects = 1;
}
```
- EN: Declares reusable TableGen class `DVPEVP_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `DVPEVP_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 545-546
```tablegen
class DVP_DESC : DVPEVP_DESC_BASE<"dvp">;
class EVP_DESC : DVPEVP_DESC_BASE<"evp">;
```
- EN: Declares reusable TableGen class `DVP_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `DVP_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 548-551
```tablegen
class DIV_DESC  : DIVMOD_DESC_BASE<"div", GPR32Opnd, sdiv>;
class DIVU_DESC : DIVMOD_DESC_BASE<"divu", GPR32Opnd, udiv>;
class MOD_DESC  : DIVMOD_DESC_BASE<"mod", GPR32Opnd, srem>;
class MODU_DESC : DIVMOD_DESC_BASE<"modu", GPR32Opnd, urem>;
```
- EN: Declares reusable TableGen class `DIV_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `DIV_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 553-555
```tablegen
class BEQZALC_DESC : CMP_CBR_RT_Z_DESC_BASE<"beqzalc", brtarget, GPR32Opnd> {
  list<Register> Defs = [RA];
}
```
- EN: Declares reusable TableGen class `BEQZALC_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BEQZALC_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 557-559
```tablegen
class BGEZALC_DESC : CMP_CBR_RT_Z_DESC_BASE<"bgezalc", brtarget, GPR32Opnd> {
  list<Register> Defs = [RA];
}
```
- EN: Declares reusable TableGen class `BGEZALC_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BGEZALC_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 561-563
```tablegen
class BGTZALC_DESC : CMP_CBR_RT_Z_DESC_BASE<"bgtzalc", brtarget, GPR32Opnd> {
  list<Register> Defs = [RA];
}
```
- EN: Declares reusable TableGen class `BGTZALC_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BGTZALC_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 565-567
```tablegen
class BLEZALC_DESC : CMP_CBR_RT_Z_DESC_BASE<"blezalc", brtarget, GPR32Opnd> {
  list<Register> Defs = [RA];
}
```
- EN: Declares reusable TableGen class `BLEZALC_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BLEZALC_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 569-571
```tablegen
class BLTZALC_DESC : CMP_CBR_RT_Z_DESC_BASE<"bltzalc", brtarget, GPR32Opnd> {
  list<Register> Defs = [RA];
}
```
- EN: Declares reusable TableGen class `BLTZALC_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BLTZALC_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 573-575
```tablegen
class BNEZALC_DESC : CMP_CBR_RT_Z_DESC_BASE<"bnezalc", brtarget, GPR32Opnd> {
  list<Register> Defs = [RA];
}
```
- EN: Declares reusable TableGen class `BNEZALC_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `BNEZALC_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 577-583
```tablegen
class MUL_R6_DESC_BASE<string instr_asm, RegisterOperand GPROpnd,
                       SDPatternOperator Op=null_frag> : MipsR6Arch<instr_asm> {
  dag OutOperandList = (outs GPROpnd:$rd);
  dag InOperandList = (ins GPROpnd:$rs, GPROpnd:$rt);
  string AsmString = !strconcat(instr_asm, "\t$rd, $rs, $rt");
  list<dag> Pattern = [(set GPROpnd:$rd, (Op GPROpnd:$rs, GPROpnd:$rt))];
}
```
- EN: Declares reusable TableGen class `MUL_R6_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `MUL_R6_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 585-588
```tablegen
class MUH_DESC    : MUL_R6_DESC_BASE<"muh", GPR32Opnd, mulhs>;
class MUHU_DESC   : MUL_R6_DESC_BASE<"muhu", GPR32Opnd, mulhu>;
class MUL_R6_DESC : MUL_R6_DESC_BASE<"mul", GPR32Opnd, mul>;
class MULU_DESC   : MUL_R6_DESC_BASE<"mulu", GPR32Opnd>;
```
- EN: Declares reusable TableGen class `MUH_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `MUH_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 590-600
```tablegen
class COP1_SEL_DESC_BASE<string instr_asm,
                         RegisterOperand FGROpnd,
                         RegisterOperand FGRCCOpnd> {
  dag OutOperandList = (outs FGROpnd:$fd);
  dag InOperandList = (ins FGRCCOpnd:$fd_in, FGROpnd:$fs, FGROpnd:$ft);
  string AsmString = !strconcat(instr_asm, "\t$fd, $fs, $ft");
  list<dag> Pattern = [(set FGROpnd:$fd, (select FGRCCOpnd:$fd_in,
                                                 FGROpnd:$ft,
                                                 FGROpnd:$fs))];
  string Constraints = "$fd_in = $fd";
}
```
- EN: Declares reusable TableGen class `COP1_SEL_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `COP1_SEL_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 602-605
```tablegen
class SEL_D_DESC : COP1_SEL_DESC_BASE<"sel.d", FGR64Opnd, FGR64CCOpnd>,
                   MipsR6Arch<"sel.d">;
class SEL_S_DESC : COP1_SEL_DESC_BASE<"sel.s", FGR32Opnd, FGR32CCOpnd>,
                   MipsR6Arch<"sel.s">;
```
- EN: Declares reusable TableGen class `SEL_D_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `SEL_D_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 607-613
```tablegen
class SELEQNE_Z_DESC_BASE<string instr_asm, RegisterOperand GPROpnd>
    : MipsR6Arch<instr_asm> {
  dag OutOperandList = (outs GPROpnd:$rd);
  dag InOperandList = (ins GPROpnd:$rs, GPROpnd:$rt);
  string AsmString = !strconcat(instr_asm, "\t$rd, $rs, $rt");
  list<dag> Pattern = [];
}
```
- EN: Declares reusable TableGen class `SELEQNE_Z_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `SELEQNE_Z_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 615-616
```tablegen
class SELEQZ_DESC : SELEQNE_Z_DESC_BASE<"seleqz", GPR32Opnd>;
class SELNEZ_DESC : SELEQNE_Z_DESC_BASE<"selnez", GPR32Opnd>;
```
- EN: Declares reusable TableGen class `SELEQZ_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `SELEQZ_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 618-624
```tablegen
class COP1_4R_DESC_BASE<string instr_asm, RegisterOperand FGROpnd> {
  dag OutOperandList = (outs FGROpnd:$fd);
  dag InOperandList = (ins FGROpnd:$fd_in, FGROpnd:$fs, FGROpnd:$ft);
  string AsmString = !strconcat(instr_asm, "\t$fd, $fs, $ft");
  list<dag> Pattern = [];
  string Constraints = "$fd_in = $fd";
}
```
- EN: Declares reusable TableGen class `COP1_4R_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `COP1_4R_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 626-629
```tablegen
class MADDF_S_DESC  : COP1_4R_DESC_BASE<"maddf.s", FGR32Opnd>;
class MADDF_D_DESC  : COP1_4R_DESC_BASE<"maddf.d", FGR64Opnd>;
class MSUBF_S_DESC  : COP1_4R_DESC_BASE<"msubf.s", FGR32Opnd>;
class MSUBF_D_DESC  : COP1_4R_DESC_BASE<"msubf.d", FGR64Opnd>;
```
- EN: Declares reusable TableGen class `MADDF_S_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `MADDF_S_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 631-636
```tablegen
class MAX_MIN_DESC_BASE<string instr_asm, RegisterOperand FGROpnd> {
  dag OutOperandList = (outs FGROpnd:$fd);
  dag InOperandList = (ins FGROpnd:$fs, FGROpnd:$ft);
  string AsmString = !strconcat(instr_asm, "\t$fd, $fs, $ft");
  list<dag> Pattern = [];
}
```
- EN: Declares reusable TableGen class `MAX_MIN_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `MAX_MIN_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 638-641
```tablegen
class MAX_S_DESC : MAX_MIN_DESC_BASE<"max.s", FGR32Opnd>;
class MAX_D_DESC : MAX_MIN_DESC_BASE<"max.d", FGR64Opnd>;
class MIN_S_DESC : MAX_MIN_DESC_BASE<"min.s", FGR32Opnd>;
class MIN_D_DESC : MAX_MIN_DESC_BASE<"min.d", FGR64Opnd>;
```
- EN: Declares reusable TableGen class `MAX_S_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `MAX_S_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 643-646
```tablegen
class MAXA_S_DESC : MAX_MIN_DESC_BASE<"maxa.s", FGR32Opnd>;
class MAXA_D_DESC : MAX_MIN_DESC_BASE<"maxa.d", FGR64Opnd>;
class MINA_S_DESC : MAX_MIN_DESC_BASE<"mina.s", FGR32Opnd>;
class MINA_D_DESC : MAX_MIN_DESC_BASE<"mina.d", FGR64Opnd>;
```
- EN: Declares reusable TableGen class `MAXA_S_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `MAXA_S_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 648-653
```tablegen
class SELEQNEZ_DESC_BASE<string instr_asm, RegisterOperand FGROpnd> {
  dag OutOperandList = (outs FGROpnd:$fd);
  dag InOperandList = (ins FGROpnd:$fs, FGROpnd:$ft);
  string AsmString = !strconcat(instr_asm, "\t$fd, $fs, $ft");
  list<dag> Pattern = [];
}
```
- EN: Declares reusable TableGen class `SELEQNEZ_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `SELEQNEZ_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 655-662
```tablegen
class SELEQZ_S_DESC : SELEQNEZ_DESC_BASE<"seleqz.s", FGR32Opnd>,
                      MipsR6Arch<"seleqz.s">;
class SELEQZ_D_DESC : SELEQNEZ_DESC_BASE<"seleqz.d", FGR64Opnd>,
                      MipsR6Arch<"seleqz.d">;
class SELNEZ_S_DESC : SELEQNEZ_DESC_BASE<"selnez.s", FGR32Opnd>,
                      MipsR6Arch<"selnez.s">;
class SELNEZ_D_DESC : SELEQNEZ_DESC_BASE<"selnez.d", FGR64Opnd>,
                      MipsR6Arch<"selnez.d">;
```
- EN: Declares reusable TableGen class `SELEQZ_S_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `SELEQZ_S_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 664-669
```tablegen
class CLASS_RINT_DESC_BASE<string instr_asm, RegisterOperand FGROpnd> {
  dag OutOperandList = (outs FGROpnd:$fd);
  dag InOperandList = (ins FGROpnd:$fs);
  string AsmString = !strconcat(instr_asm, "\t$fd, $fs");
  list<dag> Pattern = [];
}
```
- EN: Declares reusable TableGen class `CLASS_RINT_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `CLASS_RINT_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 671-674
```tablegen
class RINT_S_DESC : CLASS_RINT_DESC_BASE<"rint.s", FGR32Opnd>;
class RINT_D_DESC : CLASS_RINT_DESC_BASE<"rint.d", FGR64Opnd>;
class CLASS_S_DESC : CLASS_RINT_DESC_BASE<"class.s", FGR32Opnd>;
class CLASS_D_DESC : CLASS_RINT_DESC_BASE<"class.d", FGR64Opnd>;
```
- EN: Declares reusable TableGen class `RINT_S_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `RINT_S_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 676-683
```tablegen
class CACHE_HINT_DESC<string instr_asm, Operand MemOpnd>
                     : MipsR6Arch<instr_asm> {
  dag OutOperandList = (outs);
  dag InOperandList = (ins MemOpnd:$addr, uimm5:$hint);
  string AsmString = !strconcat(instr_asm, "\t$hint, $addr");
  list<dag> Pattern = [];
  string DecoderMethod = "DecodeCacheeOp_CacheOpR6";
}
```
- EN: Declares reusable TableGen class `CACHE_HINT_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `CACHE_HINT_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 685-686
```tablegen
class CACHE_DESC : CACHE_HINT_DESC<"cache", mem_simm9>;
class PREF_DESC : CACHE_HINT_DESC<"pref", mem_simm9>;
```
- EN: Declares reusable TableGen class `CACHE_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `CACHE_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 688-695
```tablegen
class COP2LD_DESC_BASE<string instr_asm, RegisterOperand COPOpnd> {
  dag OutOperandList = (outs COPOpnd:$rt);
  dag InOperandList = (ins mem_simm11:$addr);
  string AsmString = !strconcat(instr_asm, "\t$rt, $addr");
  list<dag> Pattern = [];
  bit mayLoad = 1;
  string DecoderMethod = "DecodeFMemCop2R6";
}
```
- EN: Declares reusable TableGen class `COP2LD_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `COP2LD_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 697-698
```tablegen
class LDC2_R6_DESC : COP2LD_DESC_BASE<"ldc2", COP2Opnd>;
class LWC2_R6_DESC : COP2LD_DESC_BASE<"lwc2", COP2Opnd>;
```
- EN: Declares reusable TableGen class `LDC2_R6_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `LDC2_R6_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 700-707
```tablegen
class COP2ST_DESC_BASE<string instr_asm, RegisterOperand COPOpnd> {
  dag OutOperandList = (outs);
  dag InOperandList = (ins COPOpnd:$rt, mem_simm11:$addr);
  string AsmString = !strconcat(instr_asm, "\t$rt, $addr");
  list<dag> Pattern = [];
  bit mayStore = 1;
  string DecoderMethod = "DecodeFMemCop2R6";
}
```
- EN: Declares reusable TableGen class `COP2ST_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `COP2ST_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 709-710
```tablegen
class SDC2_R6_DESC : COP2ST_DESC_BASE<"sdc2", COP2Opnd>;
class SWC2_R6_DESC : COP2ST_DESC_BASE<"swc2", COP2Opnd>;
```
- EN: Declares reusable TableGen class `SDC2_R6_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `SDC2_R6_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 712-719
```tablegen
class LSA_R6_DESC_BASE<string instr_asm, RegisterOperand GPROpnd,
                       Operand ImmOpnd>
      : MipsR6Arch<instr_asm> {
  dag OutOperandList = (outs GPROpnd:$rd);
  dag InOperandList = (ins GPROpnd:$rs, GPROpnd:$rt, ImmOpnd:$imm2);
  string AsmString = !strconcat(instr_asm, "\t$rd, $rs, $rt, $imm2");
  list<dag> Pattern = [];
}
```
- EN: Declares reusable TableGen class `LSA_R6_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `LSA_R6_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 721-721
```tablegen
class LSA_R6_DESC : LSA_R6_DESC_BASE<"lsa", GPR32Opnd, uimm2_plus1>;
```
- EN: Declares reusable TableGen class `LSA_R6_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `LSA_R6_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 723-731
```tablegen
class LL_R6_DESC_BASE<string instr_asm, RegisterOperand GPROpnd,
                      Operand MemOpnd>
      : MipsR6Arch<instr_asm> {
  dag OutOperandList = (outs GPROpnd:$rt);
  dag InOperandList = (ins MemOpnd:$addr);
  string AsmString = !strconcat(instr_asm, "\t$rt, $addr");
  list<dag> Pattern = [];
  bit mayLoad = 1;
}
```
- EN: Declares reusable TableGen class `LL_R6_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `LL_R6_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 733-733
```tablegen
class LL_R6_DESC : LL_R6_DESC_BASE<"ll", GPR32Opnd, mem_simm9_exp>;
```
- EN: Declares reusable TableGen class `LL_R6_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `LL_R6_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 735-742
```tablegen
class SC_R6_DESC_BASE<string instr_asm, RegisterOperand GPROpnd> {
  dag OutOperandList = (outs GPROpnd:$dst);
  dag InOperandList = (ins GPROpnd:$rt, mem_simm9_exp:$addr);
  string AsmString = !strconcat(instr_asm, "\t$rt, $addr");
  list<dag> Pattern = [];
  bit mayStore = 1;
  string Constraints = "$rt = $dst";
}
```
- EN: Declares reusable TableGen class `SC_R6_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `SC_R6_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 744-744
```tablegen
class SC_R6_DESC : SC_R6_DESC_BASE<"sc", GPR32Opnd>;
```
- EN: Declares reusable TableGen class `SC_R6_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `SC_R6_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 746-751
```tablegen
class CLO_CLZ_R6_DESC_BASE<string instr_asm, RegisterOperand GPROpnd>
    : MipsR6Arch<instr_asm> {
  dag OutOperandList = (outs GPROpnd:$rd);
  dag InOperandList = (ins GPROpnd:$rs);
  string AsmString = !strconcat(instr_asm, "\t$rd, $rs");
}
```
- EN: Declares reusable TableGen class `CLO_CLZ_R6_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `CLO_CLZ_R6_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 753-756
```tablegen
class CLO_R6_DESC_BASE<string instr_asm, RegisterOperand GPROpnd> :
    CLO_CLZ_R6_DESC_BASE<instr_asm, GPROpnd> {
  list<dag> Pattern = [(set GPROpnd:$rd, (ctlz (not GPROpnd:$rs)))];
}
```
- EN: Declares reusable TableGen class `CLO_R6_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `CLO_R6_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 758-761
```tablegen
class CLZ_R6_DESC_BASE<string instr_asm, RegisterOperand GPROpnd> :
    CLO_CLZ_R6_DESC_BASE<instr_asm, GPROpnd> {
  list<dag> Pattern = [(set GPROpnd:$rd, (ctlz GPROpnd:$rs))];
}
```
- EN: Declares reusable TableGen class `CLZ_R6_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `CLZ_R6_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 763-764
```tablegen
class CLO_R6_DESC : CLO_R6_DESC_BASE<"clo", GPR32Opnd>;
class CLZ_R6_DESC : CLZ_R6_DESC_BASE<"clz", GPR32Opnd>;
```
- EN: Declares reusable TableGen class `CLO_R6_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `CLO_R6_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 766-772
```tablegen
class SDBBP_R6_DESC {
  dag OutOperandList = (outs);
  dag InOperandList = (ins uimm20:$code_);
  string AsmString = "sdbbp\t$code_";
  list<dag> Pattern = [];
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `SDBBP_R6_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `SDBBP_R6_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 774-780
```tablegen
class CRC_DESC_BASE<string instr_asm, RegisterOperand GPROpnd>
    : MipsR6Arch<instr_asm> {
  dag OutOperandList = (outs GPROpnd:$rd);
  dag InOperandList = (ins GPROpnd:$rs, GPROpnd:$rt);
  string AsmString = !strconcat(instr_asm, "\t$rd, $rs, $rt");
  list<dag> Pattern = [];
}
```
- EN: Declares reusable TableGen class `CRC_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `CRC_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 782-787
```tablegen
class CRC32B_DESC : CRC_DESC_BASE<"crc32b", GPR32Opnd>;
class CRC32H_DESC : CRC_DESC_BASE<"crc32h", GPR32Opnd>;
class CRC32W_DESC : CRC_DESC_BASE<"crc32w", GPR32Opnd>;
class CRC32CB_DESC : CRC_DESC_BASE<"crc32cb", GPR32Opnd>;
class CRC32CH_DESC : CRC_DESC_BASE<"crc32ch", GPR32Opnd>;
class CRC32CW_DESC : CRC_DESC_BASE<"crc32cw", GPR32Opnd>;
```
- EN: Declares reusable TableGen class `CRC32B_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `CRC32B_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 789-796
```tablegen
class GINV_DESC_BASE<string instr_asm, RegisterOperand GPROpnd>
    : MipsR6Arch<instr_asm> {
  dag OutOperandList = (outs);
  dag InOperandList = (ins GPROpnd:$rs, uimm2:$type_);
  string AsmString = !strconcat(instr_asm, "\t$rs, $type_");
  list<dag> Pattern = [];
  bit hasSideEffects = 1;
}
```
- EN: Declares reusable TableGen class `GINV_DESC_BASE` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `GINV_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 798-803
```tablegen
class GINVI_DESC : GINV_DESC_BASE<"ginvi", GPR32Opnd> {
  bits<2> type_ = 0b00;
  dag InOperandList = (ins GPR32Opnd:$rs);
  string AsmString = "ginvi\t$rs";
}
class GINVT_DESC : GINV_DESC_BASE<"ginvt", GPR32Opnd>;
```
- EN: Declares reusable TableGen class `GINVI_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `GINVI_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 805-810
```tablegen
class SIGRIE_DESC {
  dag OutOperandList = (outs);
  dag InOperandList = (ins uimm16:$code_);
  string AsmString = "sigrie\t$code_";
  list<dag> Pattern = [];
}
```
- EN: Declares reusable TableGen class `SIGRIE_DESC` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `SIGRIE_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 812-816
```tablegen
//===----------------------------------------------------------------------===//
//
// Instruction Definitions
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 818-825
```tablegen
def ADDIUPC : R6MMR6Rel, ADDIUPC_ENC, ADDIUPC_DESC, ISA_MIPS32R6;
def ALIGN : R6MMR6Rel, ALIGN_ENC, ALIGN_DESC, ISA_MIPS32R6;
def ALUIPC : R6MMR6Rel, ALUIPC_ENC, ALUIPC_DESC, ISA_MIPS32R6;
def AUI : R6MMR6Rel, AUI_ENC, AUI_DESC, ISA_MIPS32R6;
def AUIPC : R6MMR6Rel, AUIPC_ENC, AUIPC_DESC, ISA_MIPS32R6;
def BAL : BAL_ENC, BAL_DESC, ISA_MIPS32R6;
def BALC : R6MMR6Rel, BALC_ENC, BALC_DESC, ISA_MIPS32R6;
def NAL : NAL_ENC, NAL_DESC, ISA_MIPS32R6;
```
- EN: Defines TableGen record `ADDIUPC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ADDIUPC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 827-844
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  def BC1EQZ : BC1EQZ_ENC, BC1EQZ_DESC, ISA_MIPS32R6, HARDFLOAT;
  def BC1NEZ : BC1NEZ_ENC, BC1NEZ_DESC, ISA_MIPS32R6, HARDFLOAT;
  def BC2EQZ : BC2EQZ_ENC, BC2EQZ_DESC, ISA_MIPS32R6;
  def BC2NEZ : BC2NEZ_ENC, BC2NEZ_DESC, ISA_MIPS32R6;
  def BC : R6MMR6Rel, BC_ENC, BC_DESC, ISA_MIPS32R6;
  def BEQC : R6MMR6Rel, BEQC_ENC, BEQC_DESC, ISA_MIPS32R6;
  def BEQZALC : R6MMR6Rel, BEQZALC_ENC, BEQZALC_DESC, ISA_MIPS32R6;
  def BEQZC : R6MMR6Rel, BEQZC_ENC, BEQZC_DESC, ISA_MIPS32R6;
  def BGEC : R6MMR6Rel, BGEC_ENC, BGEC_DESC, ISA_MIPS32R6;
  def BGEUC : R6MMR6Rel, BGEUC_ENC, BGEUC_DESC, ISA_MIPS32R6;
  def BGEZALC : R6MMR6Rel, BGEZALC_ENC, BGEZALC_DESC, ISA_MIPS32R6;
  def BGEZC : R6MMR6Rel, BGEZC_ENC, BGEZC_DESC, ISA_MIPS32R6;
  def BGTZALC : R6MMR6Rel, BGTZALC_ENC, BGTZALC_DESC, ISA_MIPS32R6;
  def BGTZC : R6MMR6Rel, BGTZC_ENC, BGTZC_DESC, ISA_MIPS32R6;
}
def BITSWAP : R6MMR6Rel, BITSWAP_ENC, BITSWAP_DESC, ISA_MIPS32R6;
let AdditionalPredicates = [NotInMicroMips] in {
```
- EN: Defines TableGen record `BC1EQZ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BC1EQZ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 845-862
```tablegen
  def BLEZALC : R6MMR6Rel, BLEZALC_ENC, BLEZALC_DESC, ISA_MIPS32R6;
  def BLEZC : R6MMR6Rel, BLEZC_ENC, BLEZC_DESC, ISA_MIPS32R6;
  def BLTC : R6MMR6Rel, BLTC_ENC, BLTC_DESC, ISA_MIPS32R6;
  def BLTUC : R6MMR6Rel, BLTUC_ENC, BLTUC_DESC, ISA_MIPS32R6;
  def BLTZALC : R6MMR6Rel, BLTZALC_ENC, BLTZALC_DESC, ISA_MIPS32R6;
  def BLTZC : R6MMR6Rel, BLTZC_ENC, BLTZC_DESC, ISA_MIPS32R6;
  def BNEC : R6MMR6Rel, BNEC_ENC, BNEC_DESC, ISA_MIPS32R6;
  def BNEZALC : R6MMR6Rel, BNEZALC_ENC, BNEZALC_DESC, ISA_MIPS32R6;
  def BNEZC : R6MMR6Rel, BNEZC_ENC, BNEZC_DESC, ISA_MIPS32R6;
  def BNVC : R6MMR6Rel, BNVC_ENC, BNVC_DESC, ISA_MIPS32R6;
  def BOVC : R6MMR6Rel, BOVC_ENC, BOVC_DESC, ISA_MIPS32R6;
  def CACHE_R6 : R6MMR6Rel, CACHE_ENC, CACHE_DESC, ISA_MIPS32R6;
  def CLASS_D : CLASS_D_ENC, CLASS_D_DESC, ISA_MIPS32R6, HARDFLOAT;
  def CLASS_S : CLASS_S_ENC, CLASS_S_DESC, ISA_MIPS32R6, HARDFLOAT;
}
def CLO_R6 : R6MMR6Rel, CLO_R6_ENC, CLO_R6_DESC, ISA_MIPS32R6;
def CLZ_R6 : R6MMR6Rel, CLZ_R6_ENC, CLZ_R6_DESC, ISA_MIPS32R6;
defm S : CMP_CC_M<FIELD_CMP_FORMAT_S, "s", FGR32Opnd, FGR32CCOpnd>;
```
- EN: Defines TableGen record `BLEZALC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BLEZALC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 863-867
```tablegen
defm D : CMP_CC_M<FIELD_CMP_FORMAT_D, "d", FGR64Opnd, FGR64CCOpnd>;
let AdditionalPredicates = [NotInMicroMips] in {
  def DIV : R6MMR6Rel, DIV_ENC, DIV_DESC, ISA_MIPS32R6;
  def DIVU : R6MMR6Rel, DIVU_ENC, DIVU_DESC, ISA_MIPS32R6;
}
```
- EN: Defines TableGen record `D` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `D`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 869-870
```tablegen
def DVP : R6MMR6Rel, DVP_ENC, DVP_DESC, ISA_MIPS32R6;
def EVP : R6MMR6Rel, EVP_ENC, EVP_DESC, ISA_MIPS32R6;
```
- EN: Defines TableGen record `DVP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DVP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 872-889
```tablegen
def JIALC : R6MMR6Rel, JIALC_ENC, JIALC_DESC, ISA_MIPS32R6;
def JIC : R6MMR6Rel, JIC_ENC, JIC_DESC, ISA_MIPS32R6;
def JR_HB_R6 : JR_HB_R6_ENC, JR_HB_R6_DESC, ISA_MIPS32R6;
let AdditionalPredicates = [NotInMicroMips] in {
  def LDC2_R6 : LDC2_R6_ENC, LDC2_R6_DESC, ISA_MIPS32R6;
  def LL_R6 : LL_R6_ENC, LL_R6_DESC, PTR_32, ISA_MIPS32R6;
}
def LSA_R6 : R6MMR6Rel, LSA_R6_ENC, LSA_R6_DESC, ISA_MIPS32R6;
let AdditionalPredicates = [NotInMicroMips] in {
  def LWC2_R6 : LWC2_R6_ENC, LWC2_R6_DESC, ISA_MIPS32R6;
}
def LWPC : R6MMR6Rel, LWPC_ENC, LWPC_DESC, ISA_MIPS32R6;
let AdditionalPredicates = [NotInMicroMips] in {
  let mayRaiseFPException = 1 in {
    let Uses = [FCR31] in {
      def MADDF_S : MADDF_S_ENC, MADDF_S_DESC, ISA_MIPS32R6, HARDFLOAT;
      def MADDF_D : MADDF_D_ENC, MADDF_D_DESC, ISA_MIPS32R6, HARDFLOAT;
      def MSUBF_S : MSUBF_S_ENC, MSUBF_S_DESC, ISA_MIPS32R6, HARDFLOAT;
```
- EN: Defines TableGen record `JIALC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JIALC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 890-891
```tablegen
      def MSUBF_D : MSUBF_D_ENC, MSUBF_D_DESC, ISA_MIPS32R6, HARDFLOAT;
    }
```
- EN: Defines TableGen record `MSUBF_D` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MSUBF_D`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 893-901
```tablegen
    def MAXA_D : MAXA_D_ENC, MAXA_D_DESC, ISA_MIPS32R6, HARDFLOAT;
    def MAXA_S : MAXA_S_ENC, MAXA_S_DESC, ISA_MIPS32R6, HARDFLOAT;
    def MAX_D : MAX_D_ENC, MAX_D_DESC, ISA_MIPS32R6, HARDFLOAT;
    def MAX_S : MAX_S_ENC, MAX_S_DESC, ISA_MIPS32R6, HARDFLOAT;
    def MINA_D : MINA_D_ENC, MINA_D_DESC, ISA_MIPS32R6, HARDFLOAT;
    def MINA_S : MINA_S_ENC, MINA_S_DESC, ISA_MIPS32R6, HARDFLOAT;
    def MIN_D : MIN_D_ENC, MIN_D_DESC, ISA_MIPS32R6, HARDFLOAT;
    def MIN_S : MIN_S_ENC, MIN_S_DESC, ISA_MIPS32R6, HARDFLOAT;
  }
```
- EN: Defines TableGen record `MAXA_D` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MAXA_D`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 903-914
```tablegen
  def MOD : R6MMR6Rel, MOD_ENC, MOD_DESC, ISA_MIPS32R6;
  def MODU : R6MMR6Rel, MODU_ENC, MODU_DESC, ISA_MIPS32R6;
  def MUH    : R6MMR6Rel, MUH_ENC, MUH_DESC, ISA_MIPS32R6;
  def MUHU   : R6MMR6Rel, MUHU_ENC, MUHU_DESC, ISA_MIPS32R6;
  def MUL_R6 : R6MMR6Rel, MUL_R6_ENC, MUL_R6_DESC, ISA_MIPS32R6;
  def MULU   : R6MMR6Rel, MULU_ENC, MULU_DESC, ISA_MIPS32R6;
}
let AdditionalPredicates = [NotInMicroMips] in {
  let mayRaiseFPException = 1, Uses = [FCR31] in {
    def RINT_D : RINT_D_ENC, RINT_D_DESC, ISA_MIPS32R6, HARDFLOAT;
    def RINT_S : RINT_S_ENC, RINT_S_DESC, ISA_MIPS32R6, HARDFLOAT;
  }
```
- EN: Defines TableGen record `MOD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 916-933
```tablegen
  def PREF_R6 : R6MMR6Rel, PREF_ENC, PREF_DESC, ISA_MIPS32R6;
  def SC_R6 : SC_R6_ENC, SC_R6_DESC, PTR_32, ISA_MIPS32R6;
  def SDBBP_R6 : SDBBP_R6_ENC, SDBBP_R6_DESC, ISA_MIPS32R6;
  def SELEQZ : R6MMR6Rel, SELEQZ_ENC, SELEQZ_DESC, ISA_MIPS32R6, GPR_32;
  def SELNEZ : R6MMR6Rel, SELNEZ_ENC, SELNEZ_DESC, ISA_MIPS32R6, GPR_32;
  def SELEQZ_D : R6MMR6Rel, SELEQZ_D_ENC, SELEQZ_D_DESC, ISA_MIPS32R6,
                 HARDFLOAT;
  def SELEQZ_S : R6MMR6Rel, SELEQZ_S_ENC, SELEQZ_S_DESC, ISA_MIPS32R6,
                 HARDFLOAT;
  def SELNEZ_D : R6MMR6Rel, SELNEZ_D_ENC, SELNEZ_D_DESC, ISA_MIPS32R6,
                 HARDFLOAT;
  def SELNEZ_S : R6MMR6Rel, SELNEZ_S_ENC, SELNEZ_S_DESC, ISA_MIPS32R6,
                 HARDFLOAT;
  def SEL_D : R6MMR6Rel, SEL_D_ENC, SEL_D_DESC, ISA_MIPS32R6, HARDFLOAT;
  def SEL_S : R6MMR6Rel, SEL_S_ENC, SEL_S_DESC, ISA_MIPS32R6, HARDFLOAT;
  def SDC2_R6 : SDC2_R6_ENC, SDC2_R6_DESC, ISA_MIPS32R6;
  def SWC2_R6 : SWC2_R6_ENC, SWC2_R6_DESC, ISA_MIPS32R6;
  def SIGRIE : SIGRIE_ENC, SIGRIE_DESC, ISA_MIPS32R6;
```
- EN: Defines TableGen record `PREF_R6` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PREF_R6`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 934-934
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 936-943
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  def CRC32B : R6MMR6Rel, CRC32B_ENC, CRC32B_DESC, ISA_MIPS32R6, ASE_CRC;
  def CRC32H : R6MMR6Rel, CRC32H_ENC, CRC32H_DESC, ISA_MIPS32R6, ASE_CRC;
  def CRC32W : R6MMR6Rel, CRC32W_ENC, CRC32W_DESC, ISA_MIPS32R6, ASE_CRC;
  def CRC32CB : R6MMR6Rel, CRC32CB_ENC, CRC32CB_DESC, ISA_MIPS32R6, ASE_CRC;
  def CRC32CH : R6MMR6Rel, CRC32CH_ENC, CRC32CH_DESC, ISA_MIPS32R6, ASE_CRC;
  def CRC32CW : R6MMR6Rel, CRC32CW_ENC, CRC32CW_DESC, ISA_MIPS32R6, ASE_CRC;
}
```
- EN: Defines TableGen record `CRC32B` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CRC32B`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 945-948
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  def GINVI : R6MMR6Rel, GINVI_ENC, GINVI_DESC, ISA_MIPS32R6, ASE_GINV;
  def GINVT : R6MMR6Rel, GINVT_ENC, GINVT_DESC, ISA_MIPS32R6, ASE_GINV;
}
```
- EN: Defines TableGen record `GINVI` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GINVI`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 950-954
```tablegen
//===----------------------------------------------------------------------===//
//
// Instruction Aliases
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 956-957
```tablegen
def : MipsInstAlias<"dvp", (DVP ZERO), 0>, ISA_MIPS32R6;
def : MipsInstAlias<"evp", (EVP ZERO), 0>, ISA_MIPS32R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 959-964
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
def : MipsInstAlias<"sdbbp", (SDBBP_R6 0)>, ISA_MIPS32R6;
def : MipsInstAlias<"sigrie", (SIGRIE 0)>, ISA_MIPS32R6;
def : MipsInstAlias<"jr $rs", (JALR ZERO, GPR32Opnd:$rs), 1>,
      ISA_MIPS32R6, GPR_32;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 966-966
```tablegen
def : MipsInstAlias<"jrc $rs", (JIC GPR32Opnd:$rs, 0), 1>, ISA_MIPS32R6, GPR_32;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 968-971
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
def : MipsInstAlias<"jalrc $rs", (JIALC GPR32Opnd:$rs, 0), 1>,
      ISA_MIPS32R6, GPR_32;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 973-976
```tablegen
def : MipsInstAlias<"div $rs, $rt", (DIV GPR32Opnd:$rs, GPR32Opnd:$rs,
                                         GPR32Opnd:$rt)>, ISA_MIPS32R6;
def : MipsInstAlias<"divu $rs, $rt", (DIVU GPR32Opnd:$rs, GPR32Opnd:$rs,
                                           GPR32Opnd:$rt)>, ISA_MIPS32R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 978-979
```tablegen
def : MipsInstAlias<"lapc $rd, $imm",
                    (ADDIUPC GPR32Opnd:$rd, simm19_lsl2:$imm)>, ISA_MIPS32R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 981-985
```tablegen
//===----------------------------------------------------------------------===//
//
// Patterns and Pseudo Instructions
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 987-999
```tablegen
// comparisons supported via another comparison
multiclass Cmp_Pats<ValueType VT> {
def : MipsPat<(seteq VT:$lhs, VT:$rhs),
      (!cast<Instruction>("CMP_EQ_"#NAME) VT:$lhs, VT:$rhs)>;
def : MipsPat<(setgt VT:$lhs, VT:$rhs),
      (!cast<Instruction>("CMP_LE_"#NAME) VT:$rhs, VT:$lhs)>;
def : MipsPat<(setge VT:$lhs, VT:$rhs),
      (!cast<Instruction>("CMP_LT_"#NAME) VT:$rhs, VT:$lhs)>;
def : MipsPat<(setlt VT:$lhs, VT:$rhs),
      (!cast<Instruction>("CMP_LT_"#NAME) VT:$lhs, VT:$rhs)>;
def : MipsPat<(setle VT:$lhs, VT:$rhs),
      (!cast<Instruction>("CMP_LE_"#NAME) VT:$lhs, VT:$rhs)>;
}
```
- EN: Declares TableGen `multiclass Cmp_Pats`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass Cmp_Pats`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1001-1004
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  defm S : Cmp_Pats<f32>, ISA_MIPS32R6;
  defm D : Cmp_Pats<f64>, ISA_MIPS32R6;
}
```
- EN: Defines TableGen record `S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1006-1015
```tablegen
// i32 selects
multiclass SelectInt_Pats<ValueType RC, Instruction OROp, Instruction XORiOp,
                          Instruction SLTiOp, Instruction SLTiuOp,
                          Instruction SELEQZOp, Instruction SELNEZOp,
                          SDPatternOperator imm_type, ValueType Opg> {
// reg, immz
def : MipsPat<(select (Opg (seteq RC:$cond, immz)), RC:$t, RC:$f),
              (OROp (SELEQZOp RC:$t, RC:$cond), (SELNEZOp RC:$f, RC:$cond))>;
def : MipsPat<(select (Opg (setne RC:$cond, immz)), RC:$t, RC:$f),
              (OROp (SELNEZOp RC:$t, RC:$cond), (SELEQZOp RC:$f, RC:$cond))>;
```
- EN: Declares TableGen `multiclass SelectInt_Pats`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass SelectInt_Pats`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1017-1023
```tablegen
// reg, immZExt16[_64]
def : MipsPat<(select (Opg (seteq RC:$cond, imm_type:$imm)), RC:$t, RC:$f),
              (OROp (SELEQZOp RC:$t, (XORiOp RC:$cond, imm_type:$imm)),
                    (SELNEZOp RC:$f, (XORiOp RC:$cond, imm_type:$imm)))>;
def : MipsPat<(select (Opg (setne RC:$cond, imm_type:$imm)), RC:$t, RC:$f),
              (OROp (SELNEZOp RC:$t, (XORiOp RC:$cond, imm_type:$imm)),
                    (SELEQZOp RC:$f, (XORiOp RC:$cond, imm_type:$imm)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1025-1031
```tablegen
// reg, immSExt16Plus1
def : MipsPat<(select (Opg (setgt RC:$cond, immSExt16Plus1:$imm)), RC:$t, RC:$f),
              (OROp (SELEQZOp RC:$t, (SLTiOp RC:$cond, (Plus1 imm:$imm))),
                    (SELNEZOp RC:$f, (SLTiOp RC:$cond, (Plus1 imm:$imm))))>;
def : MipsPat<(select (Opg (setugt RC:$cond, immSExt16Plus1:$imm)), RC:$t, RC:$f),
              (OROp (SELEQZOp RC:$t, (SLTiuOp RC:$cond, (Plus1 imm:$imm))),
                    (SELNEZOp RC:$f, (SLTiuOp RC:$cond, (Plus1 imm:$imm))))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1033-1041
```tablegen
def : MipsPat<(select (Opg (seteq RC:$cond, immz)), RC:$t, immz),
              (SELEQZOp RC:$t, RC:$cond)>;
def : MipsPat<(select (Opg (setne RC:$cond, immz)), RC:$t, immz),
              (SELNEZOp RC:$t, RC:$cond)>;
def : MipsPat<(select (Opg (seteq RC:$cond, immz)), immz, RC:$f),
              (SELNEZOp RC:$f, RC:$cond)>;
def : MipsPat<(select (Opg (setne RC:$cond, immz)), immz, RC:$f),
              (SELEQZOp RC:$f, RC:$cond)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1043-1045
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
defm : SelectInt_Pats<i32, OR, XORi, SLTi, SLTiu, SELEQZ, SELNEZ,
                      immZExt16, i32>, ISA_MIPS32R6;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1047-1057
```tablegen
def : MipsPat<(select i32:$cond, i32:$t, i32:$f),
              (OR (SELNEZ i32:$t, i32:$cond),
                  (SELEQZ i32:$f, i32:$cond))>,
              ISA_MIPS32R6;
def : MipsPat<(select i32:$cond, i32:$t, immz),
              (SELNEZ i32:$t, i32:$cond)>,
              ISA_MIPS32R6;
def : MipsPat<(select i32:$cond, immz, i32:$f),
              (SELEQZ i32:$f, i32:$cond)>,
              ISA_MIPS32R6;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1059-1076
```tablegen
// llvm.fmin/fmax operations.
let AdditionalPredicates = [NotInMicroMips] in {
  def : MipsPat<(fmaxnum_ieee f32:$lhs, f32:$rhs),
                (MAX_S   f32:$lhs, f32:$rhs)>,
                ISA_MIPS32R6;
  def : MipsPat<(fmaxnum f32:$lhs, f32:$rhs),
                (MAX_S   f32:$lhs, f32:$rhs)>,
                ISA_MIPS32R6;
  def : MipsPat<(fmaxnum_ieee f64:$lhs, f64:$rhs),
                (MAX_D   f64:$lhs, f64:$rhs)>,
                ISA_MIPS32R6;
  def : MipsPat<(fmaxnum f64:$lhs, f64:$rhs),
                (MAX_D   f64:$lhs, f64:$rhs)>,
                ISA_MIPS32R6;
  def : MipsPat<(fminnum_ieee f32:$lhs, f32:$rhs),
                (MIN_S   f32:$lhs, f32:$rhs)>,
                ISA_MIPS32R6;
  def : MipsPat<(fminnum f32:$lhs, f32:$rhs),
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1077-1091
```tablegen
                (MIN_S   f32:$lhs, f32:$rhs)>,
                ISA_MIPS32R6;
  def : MipsPat<(fminnum_ieee f64:$lhs, f64:$rhs),
                (MIN_D   f64:$lhs, f64:$rhs)>,
                ISA_MIPS32R6;
  def : MipsPat<(fminnum f64:$lhs, f64:$rhs),
                (MIN_D   f64:$lhs, f64:$rhs)>,
                ISA_MIPS32R6;
  def : MipsPat<(f32 (fcanonicalize f32:$src)),
                (MIN_S   f32:$src, f32:$src)>,
                ISA_MIPS32R6;
  def : MipsPat<(f64 (fcanonicalize f64:$src)),
                (MIN_D   f64:$src, f64:$src)>,
                ISA_MIPS32R6;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1093-1110
```tablegen
// llvm.is_fpclass operations.
def to_fclass_mask: SDNodeXForm<imm, [{
  unsigned Check = N->getZExtValue();
  unsigned Mask = 0;
  if (Check & fcSNan)
    Mask |= Mips::FClassMaskSignalingNaN;
  if (Check & fcQNan)
    Mask |= Mips::FClassMaskQuietNaN;
  if (Check & fcPosInf)
    Mask |= Mips::FClassMaskPositiveInfinity;
  if (Check & fcNegInf)
    Mask |= Mips::FClassMaskNegativeInfinity;
  if (Check & fcPosNormal)
    Mask |= Mips::FClassMaskPositiveNormal;
  if (Check & fcNegNormal)
    Mask |= Mips::FClassMaskNegativeNormal;
  if (Check & fcPosSubnormal)
    Mask |= Mips::FClassMaskPositiveSubnormal;
```
- EN: Defines TableGen record `to_fclass_mask:` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `to_fclass_mask:`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 1111-1128
```tablegen
  if (Check & fcNegSubnormal)
    Mask |= Mips::FClassMaskNegativeSubnormal;
  if (Check & fcPosZero)
    Mask |= Mips::FClassMaskPositiveZero;
  if (Check & fcNegZero)
    Mask |= Mips::FClassMaskNegativeZero;
  return CurDAG->getTargetConstant(Mask, SDLoc(N), MVT::i32);
}]>;
let AdditionalPredicates = [NotInMicroMips] in {
  def : MipsPat<(is_fpclass f32:$lhs, i32:$imm),
                (SLTu ZERO, (ANDi (MFC1 (CLASS_S f32:$lhs)),
                          (to_fclass_mask imm:$imm)))>,
                ISA_MIPS32R6;
  def : MipsPat<(is_fpclass f64:$lhs, i32:$imm),
                (SLTu ZERO, (ANDi (MFC1_D64 (CLASS_D f64:$lhs)),
                          (to_fclass_mask imm:$imm)))>,
                ISA_MIPS32R6;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1130-1136
```tablegen
// Pseudo instructions
let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1, hasDelaySlot = 1,
    hasExtraSrcRegAllocReq = 1, isCTI = 1, Defs = [AT], hasPostISelHook = 1 in {
  class TailCallRegR6<Instruction JumpInst, Register RT, RegisterOperand RO> :
    PseudoSE<(outs), (ins RO:$rs), [(MipsTailCall RO:$rs)]>,
    PseudoInstExpansion<(JumpInst RT:$rt, RO:$rs)>;
}
```
- EN: Declares reusable TableGen class `TailCallRegR6` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `TailCallRegR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1138-1148
```tablegen
class PseudoIndirectBranchBaseR6<Instruction JumpInst, Register RT,
                                 RegisterOperand RO> :
    MipsPseudo<(outs), (ins RO:$rs), [(brind RO:$rs)]>,
    PseudoInstExpansion<(JumpInst RT:$rt, RO:$rs)> {
  let isTerminator=1;
  let isBarrier=1;
  let hasDelaySlot = 1;
  let isBranch = 1;
  let isIndirectBranch = 1;
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `PseudoIndirectBranchBaseR6` for `Mips32r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrInfo` 声明可复用的 TableGen 类 `PseudoIndirectBranchBaseR6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1151-1157
```tablegen
let AdditionalPredicates = [NotInMips16Mode, NotInMicroMips,
                            NoIndirectJumpGuards] in {
  def TAILCALLR6REG : TailCallRegR6<JALR, ZERO, GPR32Opnd>, ISA_MIPS32R6;
  def PseudoIndirectBranchR6 : PseudoIndirectBranchBaseR6<JALR, ZERO,
                                                          GPR32Opnd>,
                               ISA_MIPS32R6;
}
```
- EN: Defines TableGen record `TAILCALLR6REG` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TAILCALLR6REG`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1159-1165
```tablegen
let AdditionalPredicates = [NotInMips16Mode, NotInMicroMips,
                            UseIndirectJumpsHazard] in {
  def TAILCALLHBR6REG : TailCallReg<JR_HB_R6, GPR32Opnd>, ISA_MIPS32R6;
  def PseudoIndrectHazardBranchR6 : PseudoIndirectBranchBase<JR_HB_R6,
                                                             GPR32Opnd>,
                                    ISA_MIPS32R6;
}
```
- EN: Defines TableGen record `TAILCALLHBR6REG` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TAILCALLHBR6REG`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1167-1184
```tablegen
// Combining branch and set instructions into one compact branch instruction
let AdditionalPredicates = [NotInMicroMips, UseCompactBranches] in {
  def : MipsPat<(brcond (i32 (setlt i32:$rs, 0)), bb:$offset),
                (BLTZC GPR32Opnd:$rs, bb:$offset)>, ISA_MIPS32R6;
  def : MipsPat<(brcond (i32 (setlt i32:$rs, 1)), bb:$offset),
                (BLEZC GPR32Opnd:$rs, bb:$offset)>, ISA_MIPS32R6;
  def : MipsPat<(brcond (i32 (setge i32:$rs, 0)), bb:$offset),
                (BGEZC GPR32Opnd:$rs, bb:$offset)>, ISA_MIPS32R6;
  def : MipsPat<(brcond (i32 (setge i32:$rs, 1)), bb:$offset),
                (BGTZC GPR32Opnd:$rs, bb:$offset)>, ISA_MIPS32R6;
  def : MipsPat<(brcond (i32 (setgt i32:$rs, 0)), bb:$offset),
                (BGTZC GPR32Opnd:$rs, bb:$offset)>, ISA_MIPS32R6;
  def : MipsPat<(brcond (i32 (setgt i32:$rs, -1)), bb:$offset),
                (BGEZC GPR32Opnd:$rs, bb:$offset)>, ISA_MIPS32R6;
  def : MipsPat<(brcond (i32 (setle i32:$rs, 0)), bb:$offset),
                (BLEZC GPR32Opnd:$rs, bb:$offset)>, ISA_MIPS32R6;
  def : MipsPat<(brcond (i32 (setle i32:$rs, -1)), bb:$offset),
                (BLTZC GPR32Opnd:$rs, bb:$offset)>, ISA_MIPS32R6;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1186-1193
```tablegen
  def : MipsPat<(brcond (i32 (setlt GPR32:$rs, GPR32:$rt)), bb:$offset),
                (BLTC GPR32:$rs, GPR32:$rt, bb:$offset)>, ISA_MIPS32R6;
  def : MipsPat<(brcond (i32 (setge GPR32:$rs, GPR32:$rt)), bb:$offset),
                (BGEC GPR32:$rs, GPR32:$rt, bb:$offset)>, ISA_MIPS32R6;
  def : MipsPat<(brcond (i32 (setgt GPR32:$rs, GPR32:$rt)), bb:$offset),
                (BLTC GPR32:$rt, GPR32:$rs, bb:$offset)>, ISA_MIPS32R6;
  def : MipsPat<(brcond (i32 (setle GPR32:$rs, GPR32:$rt)), bb:$offset),
                (BGEC GPR32:$rt, GPR32:$rs, bb:$offset)>, ISA_MIPS32R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1195-1203
```tablegen
  def : MipsPat<(brcond (i32 (setult GPR32:$rs, GPR32:$rt)), bb:$offset),
                (BLTUC GPR32:$rs, GPR32:$rt, bb:$offset)>, ISA_MIPS32R6;
  def : MipsPat<(brcond (i32 (setuge GPR32:$rs, GPR32:$rt)), bb:$offset),
                (BGEUC GPR32:$rs, GPR32:$rt, bb:$offset)>, ISA_MIPS32R6;
  def : MipsPat<(brcond (i32 (setugt GPR32:$rs, GPR32:$rt)), bb:$offset),
                (BLTUC GPR32:$rt, GPR32:$rs, bb:$offset)>, ISA_MIPS32R6;
  def : MipsPat<(brcond (i32 (setule GPR32:$rs, GPR32:$rt)), bb:$offset),
                (BGEUC GPR32:$rt, GPR32:$rs, bb:$offset)>, ISA_MIPS32R6;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

## Key Concepts / 关键概念

- EN: Primary role: instruction semantics, scheduling hints, and machine-level helpers.
  - CN: 核心职责：指令语义、调度提示以及机器级辅助逻辑。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: SelectionDAG patterns or node profiles connect IR-level intent to target instructions.
  - CN: SelectionDAG 模式或节点轮廓把 IR 层意图连接到目标指令。
- EN: Bitfield assignments describe exact instruction encodings and per-instruction flags.
  - CN: 位字段赋值用于描述精确的指令编码以及每条指令的标志位。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `Mips32r6InstrFormats.td`.
  - CN: TableGen 包含项：`Mips32r6InstrFormats.td`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
