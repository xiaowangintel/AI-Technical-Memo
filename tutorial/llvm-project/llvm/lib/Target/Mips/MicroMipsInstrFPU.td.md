# MicroMipsInstrFPU.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MicroMipsInstrFPU.td`
- Repository: `llvm-project`
- Purpose (EN): This file describes the microMIPS FPU instruction set.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `MicroMipsInstrFPU`，涵盖指令定义与目标操作码元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//==- MicroMipsInstrFPU.td - microMIPS FPU Instruction Info -*- tablegen -*-==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the microMIPS FPU instruction set.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-23
```tablegen
multiclass ADDS_MMM<string opstr, bit IsComm,
                    SDPatternOperator OpNode = null_frag> {
  def _D32_MM : MMRel, ADDS_FT<opstr, AFGR64Opnd, IsComm, OpNode>,
                FGR_32 {
    string DecoderNamespace = "MicroMips";
  }
  // FIXME: This needs to be part of the instruction mapping tables.
  def _D64_MM : ADDS_FT<opstr, FGR64Opnd, IsComm, OpNode>, FGR_64 {
    string DecoderNamespace = "MicroMipsFP64";
  }
}
```
- EN: Declares TableGen `multiclass ADDS_MMM`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass ADDS_MMM`，它是一个可复用模板，可展开为多个相关记录。

### Lines 25-33
```tablegen
let mayRaiseFPException = 1, Uses = [FCR31] in {
  def FADD_S_MM : MMRel, ADDS_FT<"add.s", FGR32Opnd, 1, fadd>,
                  ADDS_FM_MM<0, 0x30>, ISA_MICROMIPS;
  def FDIV_S_MM : MMRel, ADDS_FT<"div.s", FGR32Opnd, 0, fdiv>,
                  ADDS_FM_MM<0, 0xf0>, ISA_MICROMIPS;
  def FMUL_S_MM : MMRel, ADDS_FT<"mul.s", FGR32Opnd, 1, fmul>,
                  ADDS_FM_MM<0, 0xb0>, ISA_MICROMIPS;
  def FSUB_S_MM : MMRel, ADDS_FT<"sub.s", FGR32Opnd, 0, fsub>,
                  ADDS_FM_MM<0, 0x70>, ISA_MICROMIPS;
```
- EN: Defines TableGen record `FADD_S_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FADD_S_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 35-43
```tablegen
  defm FADD : ADDS_MMM<"add.d", 1, fadd>,
              ADDS_FM_MM<1, 0x30>, ISA_MICROMIPS;
  defm FDIV : ADDS_MMM<"div.d", 0, fdiv>,
              ADDS_FM_MM<1, 0xf0>, ISA_MICROMIPS;
  defm FMUL : ADDS_MMM<"mul.d", 1, fmul>,
              ADDS_FM_MM<1, 0xb0>, ISA_MICROMIPS;
  defm FSUB : ADDS_MMM<"sub.d", 0, fsub>,
              ADDS_FM_MM<1, 0x70>, ISA_MICROMIPS;
}
```
- EN: Defines TableGen record `FADD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FADD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 45-49
```tablegen
let DecoderNamespace = "MicroMips" in {
  def LWXC1_MM : MMRel, LWXC1_FT<"lwxc1", FGR32Opnd, load>,
                 LWXC1_FM_MM<0x48>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def SWXC1_MM : MMRel, SWXC1_FT<"swxc1", FGR32Opnd, store>,
                 SWXC1_FM_MM<0x88>, ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `LWXC1_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LWXC1_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 51-63
```tablegen
  def LUXC1_MM : MMRel, LWXC1_FT<"luxc1", FGR64Opnd>,
                 LWXC1_FM_MM<0x148>, FGR_64, ISA_MICROMIPS32_NOT_MIPS32R6;
  def SUXC1_MM : MMRel, SWXC1_FT<"suxc1", FGR64Opnd>,
                 SWXC1_FM_MM<0x188>, FGR_64, ISA_MICROMIPS32_NOT_MIPS32R6;
}
let isCodeGenOnly = 1 in {
def FCMP_S32_MM : MMRel, CEQS_FT<"s", FGR32, MipsFPCmp>,
                  CEQS_FM_MM<0>, ISA_MICROMIPS32_NOT_MIPS32R6 {
  // FIXME: This is a required to work around the fact that these instructions
  //        only use $fcc0. Ideally, MipsFPCmp nodes could be removed and the
  //        fcc register set is used directly.
  bits<3> fcc = 0;
}
```
- EN: Defines TableGen record `LUXC1_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LUXC1_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 65-71
```tablegen
def FCMP_D32_MM : MMRel, CEQS_FT<"d", AFGR64, MipsFPCmp>,
                  CEQS_FM_MM<1>, ISA_MICROMIPS32_NOT_MIPS32R6 {
  // FIXME: This is a required to work around the fact that these instructions
  //        only use $fcc0. Ideally, MipsFPCmp nodes could be removed and the
  //        fcc register set is used directly.
  bits<3> fcc = 0;
}
```
- EN: Defines TableGen record `FCMP_D32_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FCMP_D32_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 73-73
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 75-79
```tablegen
let DecoderNamespace = "MicroMips" in {
  def BC1F_MM : MMRel, BC1F_FT<"bc1f", brtarget_mm, MIPS_BRANCH_F>,
                BC1F_FM_MM<0x1c>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def BC1T_MM : MMRel, BC1F_FT<"bc1t", brtarget_mm, MIPS_BRANCH_T>,
                BC1F_FM_MM<0x1d>, ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `BC1F_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BC1F_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 81-84
```tablegen
  let mayRaiseFPException = 1, Uses = [FCR31] in
  def CVT_W_S_MM   : MMRel, ABSS_FT<"cvt.w.s", FGR32Opnd, FGR32Opnd>,
                     ROUND_W_FM_MM<0, 0x24>, ISA_MICROMIPS;
}
```
- EN: Defines TableGen record `CVT_W_S_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CVT_W_S_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 86-89
```tablegen
let DecoderNamespace = "MicroMips", mayRaiseFPException = 1 in {
  def ROUND_W_S_MM : MMRel, StdMMR6Rel, ABSS_FT<"round.w.s", FGR32Opnd,
                                                FGR32Opnd>,
                     ROUND_W_FM_MM<0, 0xec>, ISA_MICROMIPS;
```
- EN: Defines TableGen record `ROUND_W_S_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ROUND_W_S_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 91-99
```tablegen
  def CEIL_W_MM  : MMRel, ABSS_FT<"ceil.w.d", FGR32Opnd, AFGR64Opnd>,
                   ROUND_W_FM_MM<1, 0x6c>, ISA_MICROMIPS, FGR_32;
  def FLOOR_W_MM : MMRel, ABSS_FT<"floor.w.d", FGR32Opnd, AFGR64Opnd>,
                   ROUND_W_FM_MM<1, 0x2c>, ISA_MICROMIPS, FGR_32;
  def ROUND_W_MM : MMRel, StdMMR6Rel, ABSS_FT<"round.w.d", FGR32Opnd,
                                              AFGR64Opnd>,
                   ROUND_W_FM_MM<1, 0xec>, ISA_MICROMIPS, FGR_32;
  def TRUNC_W_MM : MMRel, ABSS_FT<"trunc.w.d", FGR32Opnd, AFGR64Opnd>,
                   ROUND_W_FM_MM<1, 0xac>, ISA_MICROMIPS, FGR_32;
```
- EN: Defines TableGen record `CEIL_W_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CEIL_W_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 101-105
```tablegen
  let Uses = [FCR31] in {
    def CVT_L_S_MM   : MMRel, ABSS_FT<"cvt.l.s", FGR64Opnd, FGR32Opnd>,
                       ROUND_W_FM_MM<0, 0x4>, ISA_MICROMIPS, FGR_64;
    def CVT_L_D64_MM : MMRel, ABSS_FT<"cvt.l.d", FGR64Opnd, FGR64Opnd>,
                       ROUND_W_FM_MM<1, 0x4>, ISA_MICROMIPS, FGR_64;
```
- EN: Defines TableGen record `CVT_L_S_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CVT_L_S_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 107-114
```tablegen
    def CVT_W_D32_MM : MMRel, ABSS_FT<"cvt.w.d", FGR32Opnd, AFGR64Opnd>,
                       ROUND_W_FM_MM<1, 0x24>, ISA_MICROMIPS, FGR_32;
  }
}
let DecoderNamespace = "MicroMipsFP64", mayRaiseFPException = 1, Uses = [FCR31] in {
  def CVT_W_D64_MM : ABSS_FT<"cvt.w.d", FGR32Opnd, FGR64Opnd>,
                     ROUND_W_FM_MM<1, 0x24>, ISA_MICROMIPS, FGR_64;
}
```
- EN: Defines TableGen record `CVT_W_D32_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CVT_W_D32_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 116-126
```tablegen
multiclass ABSS_MMM<string opstr,
                    SDPatternOperator OpNode = null_frag> {
  def _D32_MM : MMRel, ABSS_FT<opstr, AFGR64Opnd, AFGR64Opnd, OpNode>,
                ISA_MICROMIPS, FGR_32 {
    string DecoderNamespace = "MicroMips";
  }
  def _D64_MM : StdMMR6Rel, ABSS_FT<opstr, FGR64Opnd, FGR64Opnd, OpNode>,
                ISA_MICROMIPS, FGR_64 {
    string DecoderNamespace = "MicroMipsFP64";
  }
}
```
- EN: Declares TableGen `multiclass ABSS_MMM`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass ABSS_MMM`，它是一个可复用模板，可展开为多个相关记录。

### Lines 128-130
```tablegen
let mayRaiseFPException = 1, Uses = [FCR31] in
defm FSQRT : ABSS_MMM<"sqrt.d", fsqrt>, ROUND_W_FM_MM<1, 0x28>;
defm FABS : ABSS_MMM<"abs.d", fabs>, ABS_FM_MM<1, 0xd>;
```
- EN: Defines TableGen record `FSQRT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FSQRT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 132-136
```tablegen
let DecoderNamespace = "MicroMips" in {
  def FABS_S_MM : MMRel, ABSS_FT<"abs.s", FGR32Opnd, FGR32Opnd, fabs>,
                  ABS_FM_MM<0, 0xd>, ISA_MICROMIPS;
}
def : Pat<(ABSS_FT_PatFrag<fabs> FGR32Opnd:$fs), (FABS_S_MM FGR32Opnd:$fs)>;
```
- EN: Defines TableGen record `FABS_S_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FABS_S_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 138-143
```tablegen
def FMOV_S_MM : MMRel, ABSS_FT<"mov.s", FGR32Opnd, FGR32Opnd>,
                ABS_FM_MM<0, 0x1>, ISA_MICROMIPS {
  let isMoveReg = 1;
}
def FNEG_S_MM : MMRel, ABSS_FT<"neg.s", FGR32Opnd, FGR32Opnd, fneg>,
                ABS_FM_MM<0, 0x2d>, ISA_MICROMIPS;
```
- EN: Defines TableGen record `FMOV_S_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FMOV_S_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 145-150
```tablegen
let DecoderNamespace = "MicroMips", mayRaiseFPException = 1, Uses = [FCR31] in {
  def CVT_D32_S_MM : MMRel, ABSS_FT<"cvt.d.s", AFGR64Opnd, FGR32Opnd>,
                     ABS_FM_MM<0, 0x4d>, ISA_MICROMIPS, FGR_32;
  def CVT_D32_W_MM : MMRel, ABSS_FT<"cvt.d.w", AFGR64Opnd, FGR32Opnd>,
                     ABS_FM_MM<1, 0x4d>, ISA_MICROMIPS, FGR_32;
}
```
- EN: Defines TableGen record `CVT_D32_S_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CVT_D32_S_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 152-159
```tablegen
let DecoderNamespace = "MicroMipsFP64", mayRaiseFPException = 1, Uses = [FCR31] in {
  def CVT_D64_S_MM : ABSS_FT<"cvt.d.s", FGR64Opnd, FGR32Opnd>,
                     ABS_FM_MM<0, 0x4d>, ISA_MICROMIPS, FGR_64;
  def CVT_D64_W_MM : ABSS_FT<"cvt.d.w", FGR64Opnd, FGR32Opnd>,
                     ABS_FM_MM<1, 0x4d>, ISA_MICROMIPS, FGR_64;
  def CVT_S_D64_MM : ABSS_FT<"cvt.s.d", FGR32Opnd, FGR64Opnd>,
                     ABS_FM_MM<0, 0x6d>, ISA_MICROMIPS, FGR_64;
}
```
- EN: Defines TableGen record `CVT_D64_S_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CVT_D64_S_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 161-166
```tablegen
let DecoderNamespace = "MicroMips", mayRaiseFPException = 1, Uses = [FCR31] in {
  def CVT_S_D32_MM : MMRel, ABSS_FT<"cvt.s.d", FGR32Opnd, AFGR64Opnd>,
                     ABS_FM_MM<0, 0x6d>, ISA_MICROMIPS, FGR_32;
  def CVT_S_W_MM : MMRel, ABSS_FT<"cvt.s.w", FGR32Opnd, FGR32Opnd>,
                   ABS_FM_MM<1, 0x6d>, ISA_MICROMIPS;
}
```
- EN: Defines TableGen record `CVT_S_D32_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CVT_S_D32_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 169-170
```tablegen
defm FNEG : ABSS_MMM<"neg.d", fneg>, ABS_FM_MM<1, 0x2d>;
defm FMOV : ABSS_MMM<"mov.d">, ABS_FM_MM<1, 0x1>;
```
- EN: Defines TableGen record `FNEG` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FNEG`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 172-184
```tablegen
let DecoderNamespace = "MicroMips" in {
  def MOVZ_I_S_MM : MMRel, CMov_I_F_FT<"movz.s", GPR32Opnd, FGR32Opnd>,
                    CMov_I_F_FM_MM<0x78, 0>,
                    ISA_MICROMIPS32_NOT_MIPS32R6;
  def MOVN_I_S_MM : MMRel, CMov_I_F_FT<"movn.s", GPR32Opnd, FGR32Opnd>,
                    CMov_I_F_FM_MM<0x38, 0>,
                    ISA_MICROMIPS32_NOT_MIPS32R6;
  def MOVZ_I_D32_MM : MMRel, CMov_I_F_FT<"movz.d", GPR32Opnd, AFGR64Opnd>,
                      CMov_I_F_FM_MM<0x78, 1>,
                      ISA_MICROMIPS32_NOT_MIPS32R6, FGR_32;
  def MOVN_I_D32_MM : MMRel, CMov_I_F_FT<"movn.d", GPR32Opnd, AFGR64Opnd>,
                      CMov_I_F_FM_MM<0x38, 1>,
                      ISA_MICROMIPS32_NOT_MIPS32R6, FGR_32;
```
- EN: Defines TableGen record `MOVZ_I_S_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVZ_I_S_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 186-197
```tablegen
  def MOVT_S_MM : MMRel, CMov_F_F_FT<"movt.s", FGR32Opnd,
                                     MipsCMovFP_T>, CMov_F_F_FM_MM<0x60, 0>,
                  ISA_MICROMIPS32_NOT_MIPS32R6;
  def MOVF_S_MM : MMRel, CMov_F_F_FT<"movf.s", FGR32Opnd,
                                     MipsCMovFP_F>, CMov_F_F_FM_MM<0x20, 0>,
                  ISA_MICROMIPS32_NOT_MIPS32R6;
  def MOVT_D32_MM : MMRel, CMov_F_F_FT<"movt.d", AFGR64Opnd,
                                       MipsCMovFP_T>, CMov_F_F_FM_MM<0x60, 1>,
                    ISA_MICROMIPS32_NOT_MIPS32R6, FGR_32;
  def MOVF_D32_MM : MMRel, CMov_F_F_FT<"movf.d", AFGR64Opnd,
                                       MipsCMovFP_F>, CMov_F_F_FM_MM<0x20, 1>,
                    ISA_MICROMIPS32_NOT_MIPS32R6, FGR_32;
```
- EN: Defines TableGen record `MOVT_S_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVT_S_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 199-215
```tablegen
  def MFC1_MM : MMRel, MFC1_FT<"mfc1", GPR32Opnd, FGR32Opnd,
                               bitconvert>, MFC1_FM_MM<0x80>,
                ISA_MICROMIPS;
  def MTC1_MM : MMRel, MTC1_FT<"mtc1", FGR32Opnd, GPR32Opnd,
                               bitconvert>, MFC1_FM_MM<0xa0>,
                ISA_MICROMIPS;
  let mayRaiseFPException = 1, Uses = [FCR31] in {
    def MADD_S_MM : MMRel, MADDS_FT<"madd.s", FGR32Opnd>,
                    MADDS_FM_MM<0x1>, ISA_MICROMIPS32_NOT_MIPS32R6, MADD4;
    def MSUB_S_MM : MMRel, MADDS_FT<"msub.s", FGR32Opnd>,
                    MADDS_FM_MM<0x21>, ISA_MICROMIPS32_NOT_MIPS32R6, MADD4;
    def MADD_D32_MM  : MMRel, MADDS_FT<"madd.d", AFGR64Opnd>,
                       MADDS_FM_MM<0x9>, ISA_MICROMIPS32_NOT_MIPS32R6, FGR_32,
                       MADD4;
    def MSUB_D32_MM  : MMRel, MADDS_FT<"msub.d", AFGR64Opnd>,
                       MADDS_FM_MM<0x29>, ISA_MICROMIPS32_NOT_MIPS32R6, FGR_32,
                       MADD4;
```
- EN: Defines TableGen record `MFC1_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MFC1_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 217-227
```tablegen
    let AdditionalPredicates = [HasMadd4] in {
      def NMADD_S_MM : MMRel, NMADDS_FT<"nmadd.s", FGR32Opnd>,
                       MADDS_FM_MM<0x2>, ISA_MICROMIPS32_NOT_MIPS32R6;
      def NMSUB_S_MM : MMRel, NMADDS_FT<"nmsub.s", FGR32Opnd>,
                       MADDS_FM_MM<0x22>, ISA_MICROMIPS32_NOT_MIPS32R6;
      def NMADD_D32_MM : MMRel, NMADDS_FT<"nmadd.d", AFGR64Opnd>,
                         MADDS_FM_MM<0xa>, ISA_MICROMIPS32_NOT_MIPS32R6, FGR_32;
      def NMSUB_D32_MM : MMRel, NMADDS_FT<"nmsub.d", AFGR64Opnd>,
                         MADDS_FM_MM<0x2a>, ISA_MICROMIPS32_NOT_MIPS32R6, FGR_32;
    }
  }
```
- EN: Defines TableGen record `NMADD_S_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `NMADD_S_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 229-241
```tablegen
  let mayRaiseFPException = 1 in {
    def FLOOR_W_S_MM : MMRel, ABSS_FT<"floor.w.s", FGR32Opnd, FGR32Opnd>,
                       ROUND_W_FM_MM<0, 0x2c>,
                       ISA_MICROMIPS;
    def TRUNC_W_S_MM : MMRel, StdMMR6Rel, ABSS_FT<"trunc.w.s", FGR32Opnd,
                                                  FGR32Opnd>,
                       ROUND_W_FM_MM<0, 0xac>, ISA_MICROMIPS;
    def CEIL_W_S_MM  : MMRel, ABSS_FT<"ceil.w.s", FGR32Opnd, FGR32Opnd>,
                       ROUND_W_FM_MM<0, 0x6c>, ISA_MICROMIPS;
    let Uses = [FCR31] in
    def FSQRT_S_MM : MMRel, ABSS_FT<"sqrt.s", FGR32Opnd, FGR32Opnd,
                                    fsqrt>, ROUND_W_FM_MM<0, 0x28>, ISA_MICROMIPS;
  }
```
- EN: Defines TableGen record `FLOOR_W_S_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FLOOR_W_S_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 244-249
```tablegen
  def MTHC1_D32_MM : MMRel,
                     MTC1_64_FT<"mthc1", AFGR64Opnd, GPR32Opnd>,
                     MFC1_FM_MM<0xe0>, ISA_MICROMIPS, FGR_32;
  def MFHC1_D32_MM : MMRel, MFC1_FT<"mfhc1", GPR32Opnd, AFGR64Opnd>,
                     MFC1_FM_MM<0xc0>, ISA_MICROMIPS, FGR_32;
}
```
- EN: Defines TableGen record `MTHC1_D32_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MTHC1_D32_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 251-258
```tablegen
let DecoderNamespace = "MicroMipsFP64" in {
  def MTHC1_D64_MM : MTC1_64_FT<"mthc1", FGR64Opnd, GPR32Opnd>,
                     MFC1_FM_MM<0xe0>, ISA_MICROMIPS, FGR_64;
  def MFHC1_D64_MM : MFC1_FT<"mfhc1", GPR32Opnd, FGR64Opnd>,
                     MFC1_FM_MM<0xc0>, ISA_MICROMIPS, FGR_64;
  def MTC1_D64_MM : MTC1_FT<"mtc1", FGR64Opnd, GPR32Opnd>,
                    MFC1_FM_MM<0xa0>, ISA_MICROMIPS, FGR_64;
}
```
- EN: Defines TableGen record `MTHC1_D64_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MTHC1_D64_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 260-264
```tablegen
let DecoderNamespace = "MicroMips" in {
  def CFC1_MM : MMRel, MFC1_FT<"cfc1", GPR32Opnd, CCROpnd>,
                MFC1_FM_MM<0x40>, ISA_MICROMIPS;
  def CTC1_MM : MMRel, MTC1_FT<"ctc1", CCROpnd, GPR32Opnd>,
                MFC1_FM_MM<0x60>, ISA_MICROMIPS;
```
- EN: Defines TableGen record `CFC1_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CFC1_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 266-283
```tablegen
  let mayRaiseFPException = 1, Uses = [FCR31] in {
    def RECIP_S_MM : MMRel, ABSS_FT<"recip.s", FGR32Opnd, FGR32Opnd>,
                       ROUND_W_FM_MM<0b0, 0b01001000>, ISA_MICROMIPS;
    def RECIP_D32_MM : MMRel, ABSS_FT<"recip.d", AFGR64Opnd, AFGR64Opnd>,
                       ROUND_W_FM_MM<0b1, 0b01001000>, ISA_MICROMIPS, FGR_32 {
      let BaseOpcode = "RECIP_D32";
    }
    let DecoderNamespace = "MicroMipsFP64" in
      def RECIP_D64_MM : MMRel, ABSS_FT<"recip.d", FGR64Opnd, FGR64Opnd>,
                       ROUND_W_FM_MM<0b1, 0b01001000>, ISA_MICROMIPS, FGR_64;
    def RSQRT_S_MM : MMRel, ABSS_FT<"rsqrt.s", FGR32Opnd, FGR32Opnd>,
                     ROUND_W_FM_MM<0b0, 0b00001000>, ISA_MICROMIPS;
    def RSQRT_D32_MM : MMRel, ABSS_FT<"rsqrt.d", AFGR64Opnd, AFGR64Opnd>,
                     ROUND_W_FM_MM<0b1, 0b00001000>, ISA_MICROMIPS, FGR_32 {
      let BaseOpcode = "RSQRT_D32";
    }
    let DecoderNamespace = "MicroMipsFP64" in
      def RSQRT_D64_MM : MMRel, ABSS_FT<"rsqrt.d", FGR64Opnd, FGR64Opnd>,
```
- EN: Defines TableGen record `RECIP_S_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RECIP_S_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 284-286
```tablegen
                         ROUND_W_FM_MM<0b1, 0b00001000>, ISA_MICROMIPS, FGR_64;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 288-301
```tablegen
let DecoderNamespace = "MicroMips",  DecoderMethod = "DecodeFMemMMR2" in {
  def LDC1_MM_D32 : MMRel, LW_FT<"ldc1", AFGR64Opnd, mem_mm_16, load>,
                    LW_FM_MM<0x2f>, ISA_MICROMIPS, FGR_32 {
    let BaseOpcode = "LDC132";
  }
  def SDC1_MM_D32 : MMRel, SW_FT<"sdc1", AFGR64Opnd, mem_mm_16, store>,
                    LW_FM_MM<0x2e>, ISA_MICROMIPS, FGR_32 {
    let BaseOpcode = "SDC164";
  }
  def LWC1_MM : MMRel, LW_FT<"lwc1", FGR32Opnd, mem_mm_16, load>,
                LW_FM_MM<0x27>, ISA_MICROMIPS;
  def SWC1_MM : MMRel, SW_FT<"swc1", FGR32Opnd, mem_mm_16, store>,
                LW_FM_MM<0x26>, ISA_MICROMIPS;
}
```
- EN: Defines TableGen record `LDC1_MM_D32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDC1_MM_D32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 303-312
```tablegen
let DecoderNamespace = "Mips64", DecoderMethod = "DecodeFMemMMR2" in {
  def LDC1_MM_D64 : MMRel, LW_FT<"ldc1", FGR64Opnd, mem_mm_16, load>,
                    LW_FM_MM<0x2f>, ISA_MICROMIPS, FGR_64 {
    let BaseOpcode = "LDC164";
  }
  def SDC1_MM_D64 : MMRel, SW_FT<"sdc1", FGR64Opnd, mem_mm_16, store>,
                    LW_FM_MM<0x2e>, ISA_MICROMIPS, FGR_64 {
    let BaseOpcode = "SDC164";
  }
}
```
- EN: Defines TableGen record `LDC1_MM_D64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDC1_MM_D64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 315-332
```tablegen
multiclass C_COND_MM<string TypeStr, RegisterOperand RC, bits<2> fmt> {
  def C_F_#NAME#_MM : MMRel, C_COND_FT<"f", TypeStr, RC>,
                      C_COND_FM_MM<fmt, 0> {
    let BaseOpcode = "c.f."#NAME;
    let isCommutable = 1;
  }
  def C_UN_#NAME#_MM : MMRel, C_COND_FT<"un", TypeStr, RC>,
                       C_COND_FM_MM<fmt, 1> {
    let BaseOpcode = "c.un."#NAME;
    let isCommutable = 1;
  }
  def C_EQ_#NAME#_MM : MMRel, C_COND_FT<"eq", TypeStr, RC>,
                       C_COND_FM_MM<fmt, 2> {
    let BaseOpcode = "c.eq."#NAME;
    let isCommutable = 1;
  }
  def C_UEQ_#NAME#_MM : MMRel, C_COND_FT<"ueq", TypeStr, RC>,
                        C_COND_FM_MM<fmt, 3> {
```
- EN: Declares TableGen `multiclass C_COND_MM`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass C_COND_MM`，它是一个可复用模板，可展开为多个相关记录。

### Lines 333-350
```tablegen
    let BaseOpcode = "c.ueq."#NAME;
    let isCommutable = 1;
  }
  def C_OLT_#NAME#_MM : MMRel, C_COND_FT<"olt", TypeStr, RC>,
                        C_COND_FM_MM<fmt, 4> {
    let BaseOpcode = "c.olt."#NAME;
  }
  def C_ULT_#NAME#_MM : MMRel, C_COND_FT<"ult", TypeStr, RC>,
                        C_COND_FM_MM<fmt, 5> {
    let BaseOpcode = "c.ult."#NAME;
  }
  def C_OLE_#NAME#_MM : MMRel, C_COND_FT<"ole", TypeStr, RC>,
                        C_COND_FM_MM<fmt, 6> {
    let BaseOpcode = "c.ole."#NAME;
  }
  def C_ULE_#NAME#_MM : MMRel, C_COND_FT<"ule", TypeStr, RC>,
                        C_COND_FM_MM<fmt, 7> {
    let BaseOpcode = "c.ule."#NAME;
```
- EN: Defines TableGen record `C_OLT_` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `C_OLT_`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 351-368
```tablegen
  }
  let mayRaiseFPException = 1 in {
    def C_SF_#NAME#_MM : MMRel, C_COND_FT<"sf", TypeStr, RC>,
                         C_COND_FM_MM<fmt, 8> {
      let BaseOpcode = "c.sf."#NAME;
      let isCommutable = 1;
    }
    def C_NGLE_#NAME#_MM : MMRel, C_COND_FT<"ngle", TypeStr, RC>,
                           C_COND_FM_MM<fmt, 9> {
      let BaseOpcode = "c.ngle."#NAME;
    }
    def C_SEQ_#NAME#_MM : MMRel, C_COND_FT<"seq", TypeStr, RC>,
                          C_COND_FM_MM<fmt, 10> {
      let BaseOpcode = "c.seq."#NAME;
      let isCommutable = 1;
    }
    def C_NGL_#NAME#_MM : MMRel, C_COND_FT<"ngl", TypeStr, RC>,
                          C_COND_FM_MM<fmt, 11> {
```
- EN: Defines TableGen record `C_SF_` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `C_SF_`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 369-386
```tablegen
      let BaseOpcode = "c.ngl."#NAME;
    }
    def C_LT_#NAME#_MM : MMRel, C_COND_FT<"lt", TypeStr, RC>,
                         C_COND_FM_MM<fmt, 12> {
      let BaseOpcode = "c.lt."#NAME;
    }
    def C_NGE_#NAME#_MM : MMRel, C_COND_FT<"nge", TypeStr, RC>,
                          C_COND_FM_MM<fmt, 13> {
      let BaseOpcode = "c.nge."#NAME;
    }
    def C_LE_#NAME#_MM : MMRel, C_COND_FT<"le", TypeStr, RC>,
                         C_COND_FM_MM<fmt, 14> {
      let BaseOpcode = "c.le."#NAME;
    }
    def C_NGT_#NAME#_MM : MMRel, C_COND_FT<"ngt", TypeStr, RC>,
                          C_COND_FM_MM<fmt, 15> {
      let BaseOpcode = "c.ngt."#NAME;
    }
```
- EN: Defines TableGen record `C_LT_` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `C_LT_`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 387-394
```tablegen
  }
}
let DecoderNamespace = "MicroMips" in {
  defm S   : C_COND_MM<"s", FGR32Opnd, 0b00>,
             ISA_MICROMIPS32_NOT_MIPS32R6;
  defm D32 : C_COND_MM<"d", AFGR64Opnd, 0b01>,
             ISA_MICROMIPS32_NOT_MIPS32R6, FGR_32;
}
```
- EN: Defines TableGen record `S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 396-398
```tablegen
let DecoderNamespace = "Mips64" in
  defm D64 : C_COND_MM<"d", FGR64Opnd, 0b01>,
             ISA_MICROMIPS32_NOT_MIPS32R6, FGR_64;
```
- EN: Defines TableGen record `D64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `D64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 400-411
```tablegen
defm S_MM   : C_COND_ALIASES<"s", FGR32Opnd>, HARDFLOAT,
              ISA_MICROMIPS32_NOT_MIPS32R6;
defm S_MM   : C_COND_NOTR5900_ALIASES<"s", FGR32Opnd>, HARDFLOAT,
              ISA_MICROMIPS32_NOT_MIPS32R6;
defm D32_MM : C_COND_ALIASES<"d", AFGR64Opnd>, HARDFLOAT,
              ISA_MICROMIPS32_NOT_MIPS32R6, FGR_32;
defm D32_MM : C_COND_NOTR5900_ALIASES<"d", AFGR64Opnd>, HARDFLOAT,
              ISA_MICROMIPS32_NOT_MIPS32R6, FGR_32;
defm D64_MM : C_COND_ALIASES<"d", FGR64Opnd>, HARDFLOAT,
              ISA_MICROMIPS32_NOT_MIPS32R6, FGR_64;
defm D64_MM : C_COND_NOTR5900_ALIASES<"d", FGR64Opnd>, HARDFLOAT,
              ISA_MICROMIPS32_NOT_MIPS32R6, FGR_64;
```
- EN: Defines TableGen record `S_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `S_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 413-414
```tablegen
defm : BC1_ALIASES<BC1T_MM, "bc1t", BC1F_MM, "bc1f">,
       ISA_MICROMIPS32_NOT_MIPS32R6, HARDFLOAT;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 417-423
```tablegen
// To generate NMADD and NMSUB instructions when fneg node is present
let AdditionalPredicates = [HasMadd4, InMicroMips, NotMips32r6] in {
  defm : NMADD_NMSUB<NMADD_S_MM, NMSUB_S_MM, FGR32Opnd>,
         ISA_MICROMIPS32_NOT_MIPS32R6;
  defm : NMADD_NMSUB<NMADD_D32_MM, NMSUB_D32_MM, AFGR64Opnd>,
         ISA_MICROMIPS32_NOT_MIPS32R6, FGR_32;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 425-427
```tablegen
//===----------------------------------------------------------------------===//
// Floating Point Patterns
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 429-437
```tablegen
// Patterns for loads/stores with a reg+imm operand.
let AddedComplexity = 40 in {
  def : LoadRegImmPat<LDC1_MM_D32, f64, load>, ISA_MICROMIPS, FGR_32;
  def : StoreRegImmPat<SDC1_MM_D32, f64>, ISA_MICROMIPS, FGR_32;
  def : LoadRegImmPat<LDC1_MM_D64, f64, load>, ISA_MICROMIPS, FGR_64;
  def : StoreRegImmPat<SDC1_MM_D64, f64>, ISA_MICROMIPS, FGR_64;
  def : LoadRegImmPat<LWC1_MM, f32, load>, ISA_MICROMIPS;
  def : StoreRegImmPat<SWC1_MM, f32>, ISA_MICROMIPS;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 439-440
```tablegen
def : MipsPat<(MipsMTC1_D64 GPR32Opnd:$src),
              (MTC1_D64_MM GPR32Opnd:$src)>, ISA_MICROMIPS, FGR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 442-444
```tablegen
def : MipsPat<(f32 fpimm0), (MTC1_MM ZERO)>, ISA_MICROMIPS32_NOT_MIPS32R6;
def : MipsPat<(f32 fpimm0neg), (FNEG_S_MM (MTC1_MM ZERO))>,
      ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 446-461
```tablegen
def : MipsPat<(f32 (fpround FGR64Opnd:$src)),
              (CVT_S_D64_MM FGR64Opnd:$src)>, ISA_MICROMIPS, FGR_64;
def : MipsPat<(f64 (fpextend FGR32Opnd:$src)),
              (CVT_D64_S_MM FGR32Opnd:$src)>, ISA_MICROMIPS, FGR_64;
def : MipsPat<(f32 (fpround AFGR64Opnd:$src)),
              (CVT_S_D32_MM AFGR64Opnd:$src)>, ISA_MICROMIPS, FGR_32;
def : MipsPat<(f64 (fpextend FGR32Opnd:$src)),
              (CVT_D32_S_MM FGR32Opnd:$src)>, ISA_MICROMIPS, FGR_32;
def : MipsPat<(MipsTruncIntFP AFGR64Opnd:$src),
              (TRUNC_W_MM AFGR64Opnd:$src)>, ISA_MICROMIPS32_NOT_MIPS32R6,
              FGR_32;
def : MipsPat<(MipsTruncIntFP FGR64Opnd:$src),
              (CVT_W_D64_MM FGR64Opnd:$src)>, ISA_MICROMIPS32_NOT_MIPS32R6,
              FGR_64;
def : MipsPat<(MipsTruncIntFP FGR32Opnd:$src),
              (TRUNC_W_S_MM FGR32Opnd:$src)>, ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 463-467
```tablegen
// Selects
defm : MovzPats0<GPR32, FGR32, MOVZ_I_S_MM, SLT_MM, SLTu_MM, SLTi_MM, SLTiu_MM>,
       ISA_MICROMIPS32_NOT_MIPS32R6;
defm : MovzPats1<GPR32, FGR32, MOVZ_I_S_MM, XOR_MM>,
       ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 469-470
```tablegen
defm : MovnPats<GPR32, FGR32, MOVN_I_S_MM, XOR_MM>,
       ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 472-478
```tablegen
defm : MovzPats0<GPR32, AFGR64, MOVZ_I_D32_MM, SLT_MM, SLTu_MM, SLTi_MM,
                 SLTiu_MM>,
       ISA_MICROMIPS32_NOT_MIPS32R6, FGR_32;
defm : MovzPats1<GPR32, AFGR64, MOVZ_I_D32_MM, XOR_MM>,
       ISA_MICROMIPS32_NOT_MIPS32R6, FGR_32;
defm : MovnPats<GPR32, AFGR64, MOVN_I_D32_MM, XOR_MM>,
       ISA_MICROMIPS32_NOT_MIPS32R6, FGR_32;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

## Key Concepts / 关键概念

- EN: Primary role: instruction definitions and target opcode metadata.
  - CN: 核心职责：指令定义与目标操作码元数据。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: SelectionDAG patterns or node profiles connect IR-level intent to target instructions.
  - CN: SelectionDAG 模式或节点轮廓把 IR 层意图连接到目标指令。
- EN: Bitfield assignments describe exact instruction encodings and per-instruction flags.
  - CN: 位字段赋值用于描述精确的指令编码以及每条指令的标志位。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
