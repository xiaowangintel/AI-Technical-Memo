# Mips64r6InstrInfo.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/Mips64r6InstrInfo.td`
- Repository: `llvm-project`
- Purpose (EN): This file describes Mips64r6 instructions.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `Mips64r6InstrInfo`，涵盖指令语义、调度提示以及机器级辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//=- Mips64r6InstrInfo.td - Mips64r6 Instruction Information -*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes Mips64r6 instructions.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-14
```tablegen
// Notes about removals/changes from MIPS32r6:
// Reencoded: dclo, dclz
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 16-20
```tablegen
//===----------------------------------------------------------------------===//
//
// Instruction Encodings
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 22-39
```tablegen
class DALIGN_ENC  : SPECIAL3_DALIGN_FM<OPCODE6_DALIGN>;
class DAUI_ENC    : DAUI_FM;
class DAHI_ENC    : REGIMM_FM<OPCODE5_DAHI>;
class DATI_ENC    : REGIMM_FM<OPCODE5_DATI>;
class DBITSWAP_ENC : SPECIAL3_2R_FM<OPCODE6_DBITSWAP>;
class DCLO_R6_ENC : SPECIAL_2R_FM<OPCODE6_DCLO>;
class DCLZ_R6_ENC : SPECIAL_2R_FM<OPCODE6_DCLZ>;
class DDIV_ENC    : SPECIAL_3R_FM<0b00010, 0b011110>;
class DDIVU_ENC   : SPECIAL_3R_FM<0b00010, 0b011111>;
class DLSA_R6_ENC : SPECIAL_LSA_FM<OPCODE6_DLSA>;
class DMOD_ENC    : SPECIAL_3R_FM<0b00011, 0b011110>;
class DMODU_ENC   : SPECIAL_3R_FM<0b00011, 0b011111>;
class DMUH_ENC    : SPECIAL_3R_FM<0b00011, 0b011100>;
class DMUHU_ENC   : SPECIAL_3R_FM<0b00011, 0b011101>;
class DMUL_R6_ENC : SPECIAL_3R_FM<0b00010, 0b011100>;
class DMULU_ENC   : SPECIAL_3R_FM<0b00010, 0b011101>;
class LDPC_ENC    : PCREL18_FM<OPCODE3_LDPC>;
class LWUPC_ENC   : PCREL19_FM<OPCODE2_LWUPC>;
```
- EN: Declares reusable TableGen class `DALIGN_ENC` for `Mips64r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips64r6InstrInfo` 声明可复用的 TableGen 类 `DALIGN_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 40-43
```tablegen
class LLD_R6_ENC : SPECIAL3_LL_SC_FM<OPCODE6_LLD>;
class SCD_R6_ENC : SPECIAL3_LL_SC_FM<OPCODE6_SCD>;
class CRC32D_ENC  : SPECIAL3_2R_SZ_CRC<3,0>;
class CRC32CD_ENC : SPECIAL3_2R_SZ_CRC<3,1>;
```
- EN: Declares reusable TableGen class `LLD_R6_ENC` for `Mips64r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips64r6InstrInfo` 声明可复用的 TableGen 类 `LLD_R6_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 45-49
```tablegen
//===----------------------------------------------------------------------===//
//
// Instruction Descriptions
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 51-56
```tablegen
class AHI_ATI_DESC_BASE<string instr_asm, RegisterOperand GPROpnd> {
  dag OutOperandList = (outs GPROpnd:$rs);
  dag InOperandList = (ins GPROpnd:$rt, uimm16_altrelaxed:$imm);
  string AsmString = !strconcat(instr_asm, "\t$rs, $rt, $imm");
  string Constraints = "$rs = $rt";
}
```
- EN: Declares reusable TableGen class `AHI_ATI_DESC_BASE` for `Mips64r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips64r6InstrInfo` 声明可复用的 TableGen 类 `AHI_ATI_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 58-75
```tablegen
class DALIGN_DESC  : ALIGN_DESC_BASE<"dalign", GPR64Opnd, uimm3>;
class DAHI_DESC    : AHI_ATI_DESC_BASE<"dahi", GPR64Opnd>;
class DATI_DESC    : AHI_ATI_DESC_BASE<"dati", GPR64Opnd>;
class DAUI_DESC    : AUI_DESC_BASE<"daui", GPR64Opnd>;
class DBITSWAP_DESC : BITSWAP_DESC_BASE<"dbitswap", GPR64Opnd>;
class DCLO_R6_DESC : CLO_R6_DESC_BASE<"dclo", GPR64Opnd>;
class DCLZ_R6_DESC : CLZ_R6_DESC_BASE<"dclz", GPR64Opnd>;
class DDIV_DESC    : DIVMOD_DESC_BASE<"ddiv", GPR64Opnd, sdiv>;
class DDIVU_DESC   : DIVMOD_DESC_BASE<"ddivu", GPR64Opnd, udiv>;
class DLSA_R6_DESC : LSA_R6_DESC_BASE<"dlsa", GPR64Opnd, uimm2_plus1>;
class DMOD_DESC    : DIVMOD_DESC_BASE<"dmod", GPR64Opnd, srem>;
class DMODU_DESC   : DIVMOD_DESC_BASE<"dmodu", GPR64Opnd, urem>;
class DMUH_DESC    : MUL_R6_DESC_BASE<"dmuh", GPR64Opnd, mulhs>;
class DMUHU_DESC   : MUL_R6_DESC_BASE<"dmuhu", GPR64Opnd, mulhu>;
class DMUL_R6_DESC : MUL_R6_DESC_BASE<"dmul", GPR64Opnd, mul>;
class DMULU_DESC   : MUL_R6_DESC_BASE<"dmulu", GPR64Opnd>;
class LDPC_DESC    : PCREL_DESC_BASE<"ldpc", GPR64Opnd, simm18_lsl3>;
class LWUPC_DESC   : PCREL_DESC_BASE<"lwupc", GPR32Opnd, simm19_lsl2>;
```
- EN: Declares reusable TableGen class `DALIGN_DESC` for `Mips64r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips64r6InstrInfo` 声明可复用的 TableGen 类 `DALIGN_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 76-79
```tablegen
class LLD_R6_DESC   : LL_R6_DESC_BASE<"lld", GPR64Opnd, mem_simm9_exp>;
class SCD_R6_DESC   : SC_R6_DESC_BASE<"scd", GPR64Opnd>;
class SELEQZ64_DESC : SELEQNE_Z_DESC_BASE<"seleqz", GPR64Opnd>;
class SELNEZ64_DESC : SELEQNE_Z_DESC_BASE<"selnez", GPR64Opnd>;
```
- EN: Declares reusable TableGen class `LLD_R6_DESC` for `Mips64r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips64r6InstrInfo` 声明可复用的 TableGen 类 `LLD_R6_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 81-92
```tablegen
class BGEC64_DESC : CMP_BC_DESC_BASE<"bgec", brtarget, GPR64Opnd>;
class BGEUC64_DESC : CMP_BC_DESC_BASE<"bgeuc", brtarget, GPR64Opnd>;
class BEQC64_DESC : CMP_BC_DESC_BASE<"beqc", brtarget, GPR64Opnd>;
class BNEC64_DESC : CMP_BC_DESC_BASE<"bnec", brtarget, GPR64Opnd>;
class BLTC64_DESC : CMP_BC_DESC_BASE<"bltc", brtarget, GPR64Opnd>;
class BLTUC64_DESC : CMP_BC_DESC_BASE<"bltuc", brtarget, GPR64Opnd>;
class BLTZC64_DESC : CMP_CBR_RT_Z_DESC_BASE<"bltzc", brtarget, GPR64Opnd>;
class BGEZC64_DESC : CMP_CBR_RT_Z_DESC_BASE<"bgezc", brtarget, GPR64Opnd>;
class BLEZC64_DESC : CMP_CBR_RT_Z_DESC_BASE<"blezc", brtarget, GPR64Opnd>;
class BGTZC64_DESC : CMP_CBR_RT_Z_DESC_BASE<"bgtzc", brtarget, GPR64Opnd>;
class BEQZC64_DESC : CMP_CBR_EQNE_Z_DESC_BASE<"beqzc", brtarget21, GPR64Opnd>;
class BNEZC64_DESC : CMP_CBR_EQNE_Z_DESC_BASE<"bnezc", brtarget21, GPR64Opnd>;
```
- EN: Declares reusable TableGen class `BGEC64_DESC` for `Mips64r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips64r6InstrInfo` 声明可复用的 TableGen 类 `BGEC64_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 94-98
```tablegen
class JIALC64_DESC : JMP_IDX_COMPACT_DESC_BASE<"jialc", calloffset16,
                                               GPR64Opnd> {
  bit isCall = 1;
  list<Register> Defs = [RA];
}
```
- EN: Declares reusable TableGen class `JIALC64_DESC` for `Mips64r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips64r6InstrInfo` 声明可复用的 TableGen 类 `JIALC64_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 100-105
```tablegen
class JIC64_DESC : JMP_IDX_COMPACT_DESC_BASE<"jic", jmpoffset16,
                                             GPR64Opnd> {
  bit isBarrier = 1;
  bit isTerminator = 1;
  list<Register> Defs = [AT];
}
```
- EN: Declares reusable TableGen class `JIC64_DESC` for `Mips64r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips64r6InstrInfo` 声明可复用的 TableGen 类 `JIC64_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 107-108
```tablegen
class LL64_R6_DESC : LL_R6_DESC_BASE<"ll", GPR32Opnd, mem_simm9_exp>;
class SC64_R6_DESC : SC_R6_DESC_BASE<"sc", GPR32Opnd>;
```
- EN: Declares reusable TableGen class `LL64_R6_DESC` for `Mips64r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips64r6InstrInfo` 声明可复用的 TableGen 类 `LL64_R6_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 110-117
```tablegen
class JR_HB64_R6_DESC : JR_HB_DESC_BASE<"jr.hb", GPR64Opnd> {
  bit isBranch = 1;
  bit isIndirectBranch = 1;
  bit hasDelaySlot = 1;
  bit isTerminator=1;
  bit isBarrier=1;
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `JR_HB64_R6_DESC` for `Mips64r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips64r6InstrInfo` 声明可复用的 TableGen 类 `JR_HB64_R6_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 119-120
```tablegen
class CRC32D_DESC  : CRC_DESC_BASE<"crc32d", GPR32Opnd>;
class CRC32CD_DESC : CRC_DESC_BASE<"crc32cd", GPR32Opnd>;
```
- EN: Declares reusable TableGen class `CRC32D_DESC` for `Mips64r6InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips64r6InstrInfo` 声明可复用的 TableGen 类 `CRC32D_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 122-126
```tablegen
//===----------------------------------------------------------------------===//
//
// Instruction Definitions
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 128-145
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  let DecoderMethod = "DecodeDAHIDATI" in {
    def DATI : DATI_ENC, DATI_DESC, ISA_MIPS64R6;
    def DAHI : DAHI_ENC, DAHI_DESC, ISA_MIPS64R6;
  }
  def DAUI : DAUI_ENC, DAUI_DESC, ISA_MIPS64R6;
  def DALIGN : DALIGN_ENC, DALIGN_DESC, ISA_MIPS64R6;
  def DBITSWAP : DBITSWAP_ENC, DBITSWAP_DESC, ISA_MIPS64R6;
  def DCLO_R6 : DCLO_R6_ENC, DCLO_R6_DESC, ISA_MIPS64R6;
  def DCLZ_R6 : DCLZ_R6_ENC, DCLZ_R6_DESC, ISA_MIPS64R6;
  def DDIV : DDIV_ENC, DDIV_DESC, ISA_MIPS64R6;
  def DDIVU : DDIVU_ENC, DDIVU_DESC, ISA_MIPS64R6;
  def DMOD : DMOD_ENC, DMOD_DESC, ISA_MIPS64R6;
  def DMODU : DMODU_ENC, DMODU_DESC, ISA_MIPS64R6;
  def DLSA_R6 : DLSA_R6_ENC, DLSA_R6_DESC, ISA_MIPS64R6;
  def DMUH: DMUH_ENC, DMUH_DESC, ISA_MIPS64R6;
  def DMUHU: DMUHU_ENC, DMUHU_DESC, ISA_MIPS64R6;
  def DMUL_R6: DMUL_R6_ENC, DMUL_R6_DESC, ISA_MIPS64R6;
```
- EN: Defines TableGen record `DATI` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DATI`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 146-161
```tablegen
  def DMULU: DMULU_ENC, DMULU_DESC, ISA_MIPS64R6;
  def LLD_R6 : LLD_R6_ENC, LLD_R6_DESC, ISA_MIPS64R6;
}
def LDPC: LDPC_ENC, LDPC_DESC, ISA_MIPS64R6;
def LWUPC : LWUPC_ENC, LWUPC_DESC, ISA_MIPS64R6;
def SCD_R6 : SCD_R6_ENC, SCD_R6_DESC, ISA_MIPS32R6;
let DecoderNamespace = "Mips32r6_64r6_GP64" in {
  def SELEQZ64 : SELEQZ_ENC, SELEQZ64_DESC, ISA_MIPS32R6, GPR_64;
  def SELNEZ64 : SELNEZ_ENC, SELNEZ64_DESC, ISA_MIPS32R6, GPR_64;
  def JR_HB64_R6 : JR_HB_R6_ENC, JR_HB64_R6_DESC, ISA_MIPS32R6;
}
let AdditionalPredicates = [NotInMicroMips],
    DecoderNamespace = "Mips32r6_64r6_PTR64" in {
  def LL64_R6 : LL_R6_ENC, LL64_R6_DESC, PTR_64, ISA_MIPS64R6;
  def SC64_R6 : SC_R6_ENC, SC64_R6_DESC, PTR_64, ISA_MIPS64R6;
}
```
- EN: Defines TableGen record `DMULU:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DMULU:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 163-166
```tablegen
let DecoderNamespace = "Mips32r6_64r6_GP64" in {
// Jump and Branch Instructions
def JIALC64 : JIALC_ENC, JIALC64_DESC, ISA_MIPS64R6, GPR_64;
def JIC64 : JIC_ENC, JIC64_DESC, ISA_MIPS64R6, GPR_64;
```
- EN: Defines TableGen record `JIALC64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JIALC64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 168-185
```tablegen
def BEQC64 : BEQC_ENC, BEQC64_DESC, ISA_MIPS64R6, GPR_64;
def BEQZC64 : BEQZC_ENC, BEQZC64_DESC, ISA_MIPS64R6, GPR_64;
def BGEC64 : BGEC_ENC, BGEC64_DESC, ISA_MIPS64R6, GPR_64;
def BGEUC64 : BGEUC_ENC, BGEUC64_DESC, ISA_MIPS64R6, GPR_64;
def BGTZC64 : BGTZC_ENC, BGTZC64_DESC, ISA_MIPS64R6, GPR_64;
def BLEZC64 : BLEZC_ENC, BLEZC64_DESC, ISA_MIPS64R6, GPR_64;
def BLTC64 : BLTC_ENC, BLTC64_DESC, ISA_MIPS64R6, GPR_64;
def BLTUC64 : BLTUC_ENC, BLTUC64_DESC, ISA_MIPS64R6, GPR_64;
def BNEC64 : BNEC_ENC, BNEC64_DESC, ISA_MIPS64R6, GPR_64;
def BNEZC64 : BNEZC_ENC, BNEZC64_DESC, ISA_MIPS64R6, GPR_64;
}
let DecoderNamespace = "Mips32r6_64r6_BranchZero" in {
def BLTZC64 : BLTZC_ENC, BLTZC64_DESC, ISA_MIPS64R6, GPR_64;
def BGEZC64 : BGEZC_ENC, BGEZC64_DESC, ISA_MIPS64R6, GPR_64;
}
let AdditionalPredicates = [NotInMicroMips] in {
  def CRC32D : R6MMR6Rel, CRC32D_ENC, CRC32D_DESC, ISA_MIPS64R6, ASE_CRC;
  def CRC32CD : R6MMR6Rel, CRC32CD_ENC, CRC32CD_DESC, ISA_MIPS64R6, ASE_CRC;
```
- EN: Defines TableGen record `BEQC64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BEQC64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 186-186
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 188-192
```tablegen
//===----------------------------------------------------------------------===//
//
// Instruction Aliases
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 194-194
```tablegen
def : MipsInstAlias<"jr $rs", (JALR64 ZERO_64, GPR64Opnd:$rs), 1>, ISA_MIPS64R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 196-196
```tablegen
def : MipsInstAlias<"jrc $rs", (JIC64 GPR64Opnd:$rs, 0), 1>, ISA_MIPS64R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 198-203
```tablegen
def : MipsInstAlias<"jalrc $rs", (JIALC64 GPR64Opnd:$rs, 0), 1>, ISA_MIPS64R6;
//===----------------------------------------------------------------------===//
//
// Patterns and Pseudo Instructions
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 205-222
```tablegen
// i64 selects
def : MipsPat<(select i64:$cond, i64:$t, i64:$f),
              (OR64 (SELNEZ64 i64:$t, i64:$cond),
                    (SELEQZ64 i64:$f, i64:$cond))>,
              ISA_MIPS64R6;
def : MipsPat<(select (i32 (seteq i64:$cond, immz)), i64:$t, i64:$f),
              (OR64 (SELEQZ64 i64:$t, i64:$cond),
                    (SELNEZ64 i64:$f, i64:$cond))>,
              ISA_MIPS64R6;
def : MipsPat<(select (i32 (setne i64:$cond, immz)), i64:$t, i64:$f),
              (OR64 (SELNEZ64 i64:$t, i64:$cond),
                    (SELEQZ64 i64:$f, i64:$cond))>,
              ISA_MIPS64R6;
def : MipsPat<(select (i32 (seteq i64:$cond, immZExt16_64:$imm)), i64:$t, i64:$f),
              (OR64 (SELEQZ64 i64:$t, (XORi64 i64:$cond, immZExt16_64:$imm)),
                    (SELNEZ64 i64:$f, (XORi64 i64:$cond, immZExt16_64:$imm)))>,
              ISA_MIPS64R6;
def : MipsPat<(select (i32 (setne i64:$cond, immZExt16_64:$imm)), i64:$t, i64:$f),
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 223-240
```tablegen
              (OR64 (SELNEZ64 i64:$t, (XORi64 i64:$cond, immZExt16_64:$imm)),
                    (SELEQZ64 i64:$f, (XORi64 i64:$cond, immZExt16_64:$imm)))>,
              ISA_MIPS64R6;
def : MipsPat<
  (select (i32 (setgt i64:$cond, immSExt16Plus1:$imm)), i64:$t, i64:$f),
  (OR64 (SELEQZ64 i64:$t,
                  (SUBREG_TO_REG (SLTi64 i64:$cond, (Plus1 imm:$imm)),
                                 sub_32)),
        (SELNEZ64 i64:$f,
                  (SUBREG_TO_REG (SLTi64 i64:$cond, (Plus1 imm:$imm)),
                                 sub_32)))>,
  ISA_MIPS64R6;
def : MipsPat<
  (select (i32 (setugt i64:$cond, immSExt16Plus1:$imm)), i64:$t, i64:$f),
  (OR64 (SELEQZ64 i64:$t,
                  (SUBREG_TO_REG (SLTiu64 i64:$cond, (Plus1 imm:$imm)),
                                 sub_32)),
        (SELNEZ64 i64:$f,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 241-243
```tablegen
                  (SUBREG_TO_REG (SLTiu64 i64:$cond, (Plus1 imm:$imm)),
                                 sub_32)))>,
  ISA_MIPS64R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 245-252
```tablegen
def : MipsPat<(select (i32 (setne i64:$cond, immz)), i64:$t, immz),
              (SELNEZ64 i64:$t, i64:$cond)>, ISA_MIPS64R6;
def : MipsPat<(select (i32 (seteq i64:$cond, immz)), i64:$t, immz),
              (SELEQZ64 i64:$t, i64:$cond)>, ISA_MIPS64R6;
def : MipsPat<(select (i32 (setne i64:$cond, immz)), immz, i64:$f),
              (SELEQZ64 i64:$f, i64:$cond)>, ISA_MIPS64R6;
def : MipsPat<(select (i32 (seteq i64:$cond, immz)), immz, i64:$f),
              (SELNEZ64 i64:$f, i64:$cond)>, ISA_MIPS64R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 254-271
```tablegen
// i64 selects from an i32 comparison
// One complicating factor here is that bits 32-63 of an i32 are undefined.
// FIXME: Ideally, setcc would always produce an i64 on MIPS64 targets.
//        This would allow us to remove the sign-extensions here.
def : MipsPat<(select i32:$cond, i64:$t, i64:$f),
              (OR64 (SELNEZ64 i64:$t, (SLL64_32 i32:$cond)),
                    (SELEQZ64 i64:$f, (SLL64_32 i32:$cond)))>,
              ISA_MIPS64R6;
def : MipsPat<(select (i32 (seteq i32:$cond, immz)), i64:$t, i64:$f),
              (OR64 (SELEQZ64 i64:$t, (SLL64_32 i32:$cond)),
                    (SELNEZ64 i64:$f, (SLL64_32 i32:$cond)))>,
              ISA_MIPS64R6;
def : MipsPat<(select (i32 (setne i32:$cond, immz)), i64:$t, i64:$f),
              (OR64 (SELNEZ64 i64:$t, (SLL64_32 i32:$cond)),
                    (SELEQZ64 i64:$f, (SLL64_32 i32:$cond)))>,
              ISA_MIPS64R6;
def : MipsPat<(select (i32 (seteq i32:$cond, immZExt16:$imm)), i64:$t, i64:$f),
              (OR64 (SELEQZ64 i64:$t, (SLL64_32 (XORi i32:$cond,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 272-281
```tablegen
                                                      immZExt16:$imm))),
                    (SELNEZ64 i64:$f, (SLL64_32 (XORi i32:$cond,
                                                      immZExt16:$imm))))>,
              ISA_MIPS64R6;
def : MipsPat<(select (i32 (setne i32:$cond, immZExt16:$imm)), i64:$t, i64:$f),
              (OR64 (SELNEZ64 i64:$t, (SLL64_32 (XORi i32:$cond,
                                                      immZExt16:$imm))),
                    (SELEQZ64 i64:$f, (SLL64_32 (XORi i32:$cond,
                                                      immZExt16:$imm))))>,
              ISA_MIPS64R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 283-300
```tablegen
def : MipsPat<(select i32:$cond, i64:$t, immz),
              (SELNEZ64 i64:$t, (SLL64_32 i32:$cond))>,
              ISA_MIPS64R6;
def : MipsPat<(select (i32 (setne i32:$cond, immz)), i64:$t, immz),
              (SELNEZ64 i64:$t, (SLL64_32 i32:$cond))>,
              ISA_MIPS64R6;
def : MipsPat<(select (i32 (seteq i32:$cond, immz)), i64:$t, immz),
              (SELEQZ64 i64:$t, (SLL64_32 i32:$cond))>,
              ISA_MIPS64R6;
def : MipsPat<(select i32:$cond, immz, i64:$f),
              (SELEQZ64 i64:$f, (SLL64_32 i32:$cond))>,
              ISA_MIPS64R6;
def : MipsPat<(select (i32 (setne i32:$cond, immz)), immz, i64:$f),
              (SELEQZ64 i64:$f, (SLL64_32 i32:$cond))>,
              ISA_MIPS64R6;
def : MipsPat<(select (i32 (seteq i32:$cond, immz)), immz, i64:$f),
              (SELNEZ64 i64:$f, (SLL64_32 i32:$cond))>,
              ISA_MIPS64R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 302-318
```tablegen
// Patterns used for matching away redundant sign extensions.
// MIPS32 arithmetic instructions sign extend their result implicitly.
def : MipsPat<(i64 (sext (i32 (mul GPR32:$src, GPR32:$src2)))),
              (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
              (MUL_R6 GPR32:$src, GPR32:$src2), sub_32)>, ISA_MIPS64R6;
def : MipsPat<(i64 (sext (i32 (sdiv GPR32:$src, GPR32:$src2)))),
              (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
              (DIV GPR32:$src, GPR32:$src2), sub_32)>, ISA_MIPS64R6;
def : MipsPat<(i64 (sext (i32 (udiv GPR32:$src, GPR32:$src2)))),
              (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
              (DIVU GPR32:$src, GPR32:$src2), sub_32)>, ISA_MIPS64R6;
def : MipsPat<(i64 (sext (i32 (srem GPR32:$src, GPR32:$src2)))),
              (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
              (MOD GPR32:$src, GPR32:$src2), sub_32)>, ISA_MIPS64R6;
def : MipsPat<(i64 (sext (i32 (urem GPR32:$src, GPR32:$src2)))),
              (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
              (MODU GPR32:$src, GPR32:$src2), sub_32)>, ISA_MIPS64R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 320-320
```tablegen
// Pseudo instructions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 322-328
```tablegen
let AdditionalPredicates = [NotInMips16Mode, NotInMicroMips,
                            NoIndirectJumpGuards] in {
  def TAILCALL64R6REG : TailCallRegR6<JALR64, ZERO_64, GPR64Opnd>, ISA_MIPS64R6;
  def PseudoIndirectBranch64R6 : PseudoIndirectBranchBaseR6<JALR64, ZERO_64,
                                                            GPR64Opnd>,
                                 ISA_MIPS64R6;
}
```
- EN: Defines TableGen record `TAILCALL64R6REG` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TAILCALL64R6REG`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 330-337
```tablegen
let AdditionalPredicates = [NotInMips16Mode, NotInMicroMips,
                            UseIndirectJumpsHazard] in {
  def TAILCALLHB64R6REG : TailCallReg<JR_HB64_R6, GPR64Opnd>,
                          ISA_MIPS64R6;
  def PseudoIndrectHazardBranch64R6 : PseudoIndirectBranchBase<JR_HB64_R6,
                                                                 GPR64Opnd>,
                                      ISA_MIPS64R6;
}
```
- EN: Defines TableGen record `TAILCALLHB64R6REG` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TAILCALLHB64R6REG`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 339-356
```tablegen
// Combining branch and set instructions into one compact branch instruction
let AdditionalPredicates = [NotInMicroMips, UseCompactBranches] in {
  def : MipsPat<(brcond (i32 (setlt i64:$rs, 0)), bb:$offset),
                (BLTZC64 GPR64Opnd:$rs, bb:$offset)>, ISA_MIPS64R6;
  def : MipsPat<(brcond (i32 (setlt i64:$rs, 1)), bb:$offset),
                (BLEZC64 GPR64Opnd:$rs, bb:$offset)>, ISA_MIPS64R6;
  def : MipsPat<(brcond (i32 (setge i64:$rs, 0)), bb:$offset),
                (BGEZC64 GPR64Opnd:$rs, bb:$offset)>, ISA_MIPS64R6;
  def : MipsPat<(brcond (i32 (setge i64:$rs, 1)), bb:$offset),
                (BGTZC64 GPR64Opnd:$rs, bb:$offset)>, ISA_MIPS64R6;
  def : MipsPat<(brcond (i32 (setgt i64:$rs, 0)), bb:$offset),
                (BGTZC64 GPR64Opnd:$rs, bb:$offset)>, ISA_MIPS64R6;
  def : MipsPat<(brcond (i32 (setgt i64:$rs, -1)), bb:$offset),
                (BGEZC64 GPR64Opnd:$rs, bb:$offset)>, ISA_MIPS64R6;
  def : MipsPat<(brcond (i32 (setle i64:$rs, 0)), bb:$offset),
                (BLEZC64 GPR64Opnd:$rs, bb:$offset)>, ISA_MIPS64R6;
  def : MipsPat<(brcond (i32 (setle i64:$rs, -1)), bb:$offset),
                (BLTZC64 GPR64Opnd:$rs, bb:$offset)>, ISA_MIPS64R6;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 358-365
```tablegen
  def : MipsPat<(brcond (i32 (setlt GPR64:$rs, GPR64:$rt)), bb:$offset),
                (BLTC64 GPR64:$rs, GPR64:$rt, bb:$offset)>, ISA_MIPS64R6;
  def : MipsPat<(brcond (i32 (setge GPR64:$rs, GPR64:$rt)), bb:$offset),
                (BGEC64 GPR64:$rs, GPR64:$rt, bb:$offset)>, ISA_MIPS64R6;
  def : MipsPat<(brcond (i32 (setgt GPR64:$rs, GPR64:$rt)), bb:$offset),
                (BLTC64 GPR64:$rt, GPR64:$rs, bb:$offset)>, ISA_MIPS64R6;
  def : MipsPat<(brcond (i32 (setle GPR64:$rs, GPR64:$rt)), bb:$offset),
                (BGEC64 GPR64:$rt, GPR64:$rs, bb:$offset)>, ISA_MIPS64R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 367-375
```tablegen
  def : MipsPat<(brcond (i32 (setult GPR64:$rs, GPR64:$rt)), bb:$offset),
                (BLTUC64 GPR64:$rs, GPR64:$rt, bb:$offset)>, ISA_MIPS64R6;
  def : MipsPat<(brcond (i32 (setuge GPR64:$rs, GPR64:$rt)), bb:$offset),
                (BGEUC64 GPR64:$rs, GPR64:$rt, bb:$offset)>, ISA_MIPS64R6;
  def : MipsPat<(brcond (i32 (setugt GPR64:$rs, GPR64:$rt)), bb:$offset),
                (BLTUC64 GPR64:$rt, GPR64:$rs, bb:$offset)>, ISA_MIPS64R6;
  def : MipsPat<(brcond (i32 (setule GPR64:$rs, GPR64:$rt)), bb:$offset),
                (BGEUC64 GPR64:$rt, GPR64:$rs, bb:$offset)>, ISA_MIPS64R6;
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
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
