# MipsCondMov.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsCondMov.td`
- Repository: `llvm-project`
- Purpose (EN): Defines `MipsCondMov` in LLVM TableGen DSL for the Mips backend, covering target descriptions, instruction records, and code-generation metadata.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `MipsCondMov`，涵盖目标描述、指令记录以及代码生成元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===-- MipsCondMov.td - Describe Mips Conditional Moves --*- tablegen -*--===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This is the Conditional Moves implementation.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-22
```tablegen
// Conditional moves:
// These instructions are expanded in
// MipsISelLowering::EmitInstrWithCustomInserter if target does not have
// conditional move instructions.
// cond:int, data:int
class CMov_I_I_FT<string opstr, RegisterOperand CRC, RegisterOperand DRC> :
  InstSE<(outs DRC:$rd), (ins DRC:$rs, CRC:$rt, DRC:$F),
         !strconcat(opstr, "\t$rd, $rs, $rt"), [], FrmFR, opstr> {
  let Constraints = "$F = $rd";
}
```
- EN: Declares reusable TableGen class `CMov_I_I_FT` for `MipsCondMov`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsCondMov` 声明可复用的 TableGen 类 `CMov_I_I_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 24-30
```tablegen
// cond:int, data:float
class CMov_I_F_FT<string opstr, RegisterOperand CRC, RegisterOperand DRC> :
  InstSE<(outs DRC:$fd), (ins DRC:$fs, CRC:$rt, DRC:$F),
         !strconcat(opstr, "\t$fd, $fs, $rt"), [], FrmFR, opstr>,
  HARDFLOAT {
  let Constraints = "$F = $fd";
}
```
- EN: Declares reusable TableGen class `CMov_I_F_FT` for `MipsCondMov`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsCondMov` 声明可复用的 TableGen 类 `CMov_I_F_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 32-40
```tablegen
// cond:float, data:int
class CMov_F_I_FT<string opstr, RegisterOperand RC,
                  SDPatternOperator OpNode = null_frag> :
  InstSE<(outs RC:$rd), (ins RC:$rs, FCCRegsOpnd:$fcc, RC:$F),
         !strconcat(opstr, "\t$rd, $rs, $fcc"),
         [(set RC:$rd, (OpNode RC:$rs, FCCRegsOpnd:$fcc, RC:$F))],
         FrmFR, opstr>, HARDFLOAT {
  let Constraints = "$F = $rd";
}
```
- EN: Declares reusable TableGen class `CMov_F_I_FT` for `MipsCondMov`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsCondMov` 声明可复用的 TableGen 类 `CMov_F_I_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 42-50
```tablegen
// cond:float, data:float
class CMov_F_F_FT<string opstr, RegisterOperand RC,
                  SDPatternOperator OpNode = null_frag> :
  InstSE<(outs RC:$fd), (ins RC:$fs, FCCRegsOpnd:$fcc, RC:$F),
         !strconcat(opstr, "\t$fd, $fs, $fcc"),
         [(set RC:$fd, (OpNode RC:$fs, FCCRegsOpnd:$fcc, RC:$F))],
         FrmFR, opstr>, HARDFLOAT {
  let Constraints = "$F = $fd";
}
```
- EN: Declares reusable TableGen class `CMov_F_F_FT` for `MipsCondMov`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsCondMov` 声明可复用的 TableGen 类 `CMov_F_F_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 52-69
```tablegen
// select patterns
multiclass MovzPats0<RegisterClass CRC, RegisterClass DRC,
                     Instruction MOVZInst, Instruction SLTOp,
                     Instruction SLTuOp, Instruction SLTiOp,
                     Instruction SLTiuOp> {
  def : MipsPat<(select (i32 (setge CRC:$lhs, CRC:$rhs)), DRC:$T, DRC:$F),
                (MOVZInst DRC:$T, (SLTOp CRC:$lhs, CRC:$rhs), DRC:$F)>;
  def : MipsPat<(select (i32 (setuge CRC:$lhs, CRC:$rhs)), DRC:$T, DRC:$F),
                (MOVZInst DRC:$T, (SLTuOp CRC:$lhs, CRC:$rhs), DRC:$F)>;
  def : MipsPat<(select (i32 (setge CRC:$lhs, immSExt16:$rhs)), DRC:$T, DRC:$F),
                (MOVZInst DRC:$T, (SLTiOp CRC:$lhs, immSExt16:$rhs), DRC:$F)>;
  def : MipsPat<(select (i32 (setuge CRC:$lh, immSExt16:$rh)), DRC:$T, DRC:$F),
                (MOVZInst DRC:$T, (SLTiuOp CRC:$lh, immSExt16:$rh), DRC:$F)>;
  def : MipsPat<(select (i32 (setle CRC:$lhs, CRC:$rhs)), DRC:$T, DRC:$F),
                (MOVZInst DRC:$T, (SLTOp CRC:$rhs, CRC:$lhs), DRC:$F)>;
  def : MipsPat<(select (i32 (setule CRC:$lhs, CRC:$rhs)), DRC:$T, DRC:$F),
                (MOVZInst DRC:$T, (SLTuOp CRC:$rhs, CRC:$lhs), DRC:$F)>;
  def : MipsPat<(select (i32 (setgt CRC:$lhs, immSExt16Plus1:$rhs)),
```
- EN: Declares TableGen `multiclass MovzPats0`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MovzPats0`，它是一个可复用模板，可展开为多个相关记录。

### Lines 70-76
```tablegen
                        DRC:$T, DRC:$F),
                (MOVZInst DRC:$T, (SLTiOp CRC:$lhs, (Plus1 imm:$rhs)), DRC:$F)>;
  def : MipsPat<(select (i32 (setugt CRC:$lhs, immSExt16Plus1:$rhs)),
                        DRC:$T, DRC:$F),
                (MOVZInst DRC:$T, (SLTiuOp CRC:$lhs, (Plus1 imm:$rhs)),
                          DRC:$F)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 78-84
```tablegen
multiclass MovzPats1<RegisterClass CRC, RegisterClass DRC,
                     Instruction MOVZInst, Instruction XOROp> {
  def : MipsPat<(select (i32 (seteq CRC:$lhs, CRC:$rhs)), DRC:$T, DRC:$F),
                (MOVZInst DRC:$T, (XOROp CRC:$lhs, CRC:$rhs), DRC:$F)>;
  def : MipsPat<(select (i32 (seteq CRC:$lhs, 0)), DRC:$T, DRC:$F),
                (MOVZInst DRC:$T, CRC:$lhs, DRC:$F)>;
}
```
- EN: Declares TableGen `multiclass MovzPats1`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MovzPats1`，它是一个可复用模板，可展开为多个相关记录。

### Lines 86-91
```tablegen
multiclass MovzPats2<RegisterClass CRC, RegisterClass DRC,
                     Instruction MOVZInst, Instruction XORiOp> {
  def : MipsPat<
            (select (i32 (seteq CRC:$lhs, immZExt16:$uimm16)), DRC:$T, DRC:$F),
            (MOVZInst DRC:$T, (XORiOp CRC:$lhs, immZExt16:$uimm16), DRC:$F)>;
}
```
- EN: Declares TableGen `multiclass MovzPats2`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MovzPats2`，它是一个可复用模板，可展开为多个相关记录。

### Lines 93-101
```tablegen
multiclass MovnPats<RegisterClass CRC, RegisterClass DRC, Instruction MOVNInst,
                    Instruction XOROp> {
  def : MipsPat<(select (i32 (setne CRC:$lhs, CRC:$rhs)), DRC:$T, DRC:$F),
                (MOVNInst DRC:$T, (XOROp CRC:$lhs, CRC:$rhs), DRC:$F)>;
  def : MipsPat<(select CRC:$cond, DRC:$T, DRC:$F),
                (MOVNInst DRC:$T, CRC:$cond, DRC:$F)>;
  def : MipsPat<(select (i32 (setne CRC:$lhs, 0)),DRC:$T, DRC:$F),
                (MOVNInst DRC:$T, CRC:$lhs, DRC:$F)>;
}
```
- EN: Declares TableGen `multiclass MovnPats`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MovnPats`，它是一个可复用模板，可展开为多个相关记录。

### Lines 103-106
```tablegen
// Instantiation of instructions.
let AdditionalPredicates = [NotInMicroMips] in {
  def MOVZ_I_I : MMRel, CMov_I_I_FT<"movz", GPR32Opnd, GPR32Opnd>,
                 ADD_FM<0, 0xa>, INSN_MIPS4_32_NOT_32R6_64R6;
```
- EN: Defines TableGen record `MOVZ_I_I` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVZ_I_I`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 108-115
```tablegen
  let isCodeGenOnly = 1 in {
    def MOVZ_I_I64   : CMov_I_I_FT<"movz", GPR32Opnd, GPR64Opnd>,
                       ADD_FM<0, 0xa>, INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
    def MOVZ_I64_I   : CMov_I_I_FT<"movz", GPR64Opnd, GPR32Opnd>,
                       ADD_FM<0, 0xa>, INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
    def MOVZ_I64_I64 : CMov_I_I_FT<"movz", GPR64Opnd, GPR64Opnd>,
                       ADD_FM<0, 0xa>, INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
  }
```
- EN: Defines TableGen record `MOVZ_I_I64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVZ_I_I64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 117-118
```tablegen
  def MOVN_I_I     : MMRel, CMov_I_I_FT<"movn", GPR32Opnd, GPR32Opnd>,
                     ADD_FM<0, 0xb>, INSN_MIPS4_32_NOT_32R6_64R6;
```
- EN: Defines TableGen record `MOVN_I_I` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVN_I_I`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 120-129
```tablegen
  let isCodeGenOnly = 1 in {
    def MOVN_I_I64   : CMov_I_I_FT<"movn", GPR32Opnd, GPR64Opnd>,
                       ADD_FM<0, 0xb>, INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
    def MOVN_I64_I   : CMov_I_I_FT<"movn", GPR64Opnd, GPR32Opnd>,
                       ADD_FM<0, 0xb>, INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
    def MOVN_I64_I64 : CMov_I_I_FT<"movn", GPR64Opnd, GPR64Opnd>,
                       ADD_FM<0, 0xb>, INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
  }
  def MOVZ_I_S : MMRel, CMov_I_F_FT<"movz.s", GPR32Opnd, FGR32Opnd>,
                 CMov_I_F_FM<18, 16>, INSN_MIPS4_32_NOT_32R6_64R6;
```
- EN: Defines TableGen record `MOVN_I_I64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVN_I_I64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 131-133
```tablegen
  let isCodeGenOnly = 1 in
  def MOVZ_I64_S : CMov_I_F_FT<"movz.s", GPR64Opnd, FGR32Opnd>,
                   CMov_I_F_FM<18, 16>, INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
```
- EN: Defines TableGen record `MOVZ_I64_S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVZ_I64_S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 135-136
```tablegen
  def MOVN_I_S : MMRel, CMov_I_F_FT<"movn.s", GPR32Opnd, FGR32Opnd>,
                 CMov_I_F_FM<19, 16>, INSN_MIPS4_32_NOT_32R6_64R6;
```
- EN: Defines TableGen record `MOVN_I_S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVN_I_S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 138-140
```tablegen
  let isCodeGenOnly = 1 in
  def MOVN_I64_S : CMov_I_F_FT<"movn.s", GPR64Opnd, FGR32Opnd>,
                   CMov_I_F_FM<19, 16>, INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
```
- EN: Defines TableGen record `MOVN_I64_S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVN_I64_S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 142-147
```tablegen
  def MOVZ_I_D32 : MMRel, CMov_I_F_FT<"movz.d", GPR32Opnd, AFGR64Opnd>,
                   CMov_I_F_FM<18, 17>,
                   INSN_MIPS4_32_NOT_32R6_64R6, FGR_32;
  def MOVN_I_D32 : MMRel, CMov_I_F_FT<"movn.d", GPR32Opnd, AFGR64Opnd>,
                   CMov_I_F_FM<19, 17>,
                   INSN_MIPS4_32_NOT_32R6_64R6, FGR_32;
```
- EN: Defines TableGen record `MOVZ_I_D32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVZ_I_D32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 149-162
```tablegen
  let DecoderNamespace = "MipsFP64" in {
    def MOVZ_I_D64 : CMov_I_F_FT<"movz.d", GPR32Opnd, FGR64Opnd>,
                     CMov_I_F_FM<18, 17>, INSN_MIPS4_32_NOT_32R6_64R6, FGR_64;
    def MOVN_I_D64 : CMov_I_F_FT<"movn.d", GPR32Opnd, FGR64Opnd>,
                     CMov_I_F_FM<19, 17>, INSN_MIPS4_32_NOT_32R6_64R6, FGR_64;
    let isCodeGenOnly = 1 in {
      def MOVZ_I64_D64 : CMov_I_F_FT<"movz.d", GPR64Opnd, FGR64Opnd>,
                         CMov_I_F_FM<18, 17>,
                         INSN_MIPS4_32_NOT_32R6_64R6, GPR_64, FGR_64;
      def MOVN_I64_D64 : CMov_I_F_FT<"movn.d", GPR64Opnd, FGR64Opnd>,
                         CMov_I_F_FM<19, 17>,
                         INSN_MIPS4_32_NOT_32R6_64R6, GPR_64, FGR_64;
    }
  }
```
- EN: Defines TableGen record `MOVZ_I_D64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVZ_I_D64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 164-165
```tablegen
  def MOVT_I : MMRel, CMov_F_I_FT<"movt", GPR32Opnd, MipsCMovFP_T>,
               CMov_F_I_FM<1>, INSN_MIPS4_32_NOT_32R6_64R6;
```
- EN: Defines TableGen record `MOVT_I` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVT_I`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 167-169
```tablegen
  let isCodeGenOnly = 1 in
  def MOVT_I64 : CMov_F_I_FT<"movt", GPR64Opnd, MipsCMovFP_T>,
                 CMov_F_I_FM<1>, INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
```
- EN: Defines TableGen record `MOVT_I64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVT_I64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 171-172
```tablegen
  def MOVF_I : MMRel, CMov_F_I_FT<"movf", GPR32Opnd, MipsCMovFP_F>,
               CMov_F_I_FM<0>, INSN_MIPS4_32_NOT_32R6_64R6;
```
- EN: Defines TableGen record `MOVF_I` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVF_I`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 174-180
```tablegen
  let isCodeGenOnly = 1 in
  def MOVF_I64 : CMov_F_I_FT<"movf", GPR64Opnd, MipsCMovFP_F>,
                 CMov_F_I_FM<0>, INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
  def MOVT_S : MMRel, CMov_F_F_FT<"movt.s", FGR32Opnd, MipsCMovFP_T>,
               CMov_F_F_FM<16, 1>, INSN_MIPS4_32_NOT_32R6_64R6;
  def MOVF_S : MMRel, CMov_F_F_FT<"movf.s", FGR32Opnd, MipsCMovFP_F>,
               CMov_F_F_FM<16, 0>, INSN_MIPS4_32_NOT_32R6_64R6;
```
- EN: Defines TableGen record `MOVF_I64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVF_I64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 182-187
```tablegen
  def MOVT_D32 : MMRel, CMov_F_F_FT<"movt.d", AFGR64Opnd, MipsCMovFP_T>,
                 CMov_F_F_FM<17, 1>,
                 INSN_MIPS4_32_NOT_32R6_64R6, FGR_32;
  def MOVF_D32 : MMRel, CMov_F_F_FT<"movf.d", AFGR64Opnd, MipsCMovFP_F>,
                 CMov_F_F_FM<17, 0>,
                 INSN_MIPS4_32_NOT_32R6_64R6, FGR_32;
```
- EN: Defines TableGen record `MOVT_D32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVT_D32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 189-194
```tablegen
  let DecoderNamespace = "MipsFP64" in {
    def MOVT_D64 : CMov_F_F_FT<"movt.d", FGR64Opnd, MipsCMovFP_T>,
                   CMov_F_F_FM<17, 1>, INSN_MIPS4_32_NOT_32R6_64R6, FGR_64;
    def MOVF_D64 : CMov_F_F_FT<"movf.d", FGR64Opnd, MipsCMovFP_F>,
                   CMov_F_F_FM<17, 0>, INSN_MIPS4_32_NOT_32R6_64R6, FGR_64;
  }
```
- EN: Defines TableGen record `MOVT_D64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVT_D64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 196-200
```tablegen
  // Instantiation of conditional move patterns.
  defm : MovzPats0<GPR32, GPR32, MOVZ_I_I, SLT, SLTu, SLTi, SLTiu>,
         INSN_MIPS4_32_NOT_32R6_64R6;
  defm : MovzPats1<GPR32, GPR32, MOVZ_I_I, XOR>, INSN_MIPS4_32_NOT_32R6_64R6;
  defm : MovzPats2<GPR32, GPR32, MOVZ_I_I, XORi>, INSN_MIPS4_32_NOT_32R6_64R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 202-219
```tablegen
  defm : MovzPats0<GPR32, GPR64, MOVZ_I_I64, SLT, SLTu, SLTi, SLTiu>,
         INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
  defm : MovzPats0<GPR64, GPR32, MOVZ_I_I, SLT64, SLTu64, SLTi64, SLTiu64>,
         INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
  defm : MovzPats0<GPR64, GPR64, MOVZ_I_I64, SLT64, SLTu64, SLTi64, SLTiu64>,
         INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
  defm : MovzPats1<GPR32, GPR64, MOVZ_I_I64, XOR>,
         INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
  defm : MovzPats1<GPR64, GPR32, MOVZ_I64_I, XOR64>,
         INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
  defm : MovzPats1<GPR64, GPR64, MOVZ_I64_I64, XOR64>,
         INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
  defm : MovzPats2<GPR32, GPR64, MOVZ_I_I64, XORi>,
         INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
  defm : MovzPats2<GPR64, GPR32, MOVZ_I64_I, XORi64>,
         INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
  defm : MovzPats2<GPR64, GPR64, MOVZ_I64_I64, XORi64>,
         INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 221-221
```tablegen
  defm : MovnPats<GPR32, GPR32, MOVN_I_I, XOR>, INSN_MIPS4_32_NOT_32R6_64R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 223-228
```tablegen
  defm : MovnPats<GPR32, GPR64, MOVN_I_I64, XOR>, INSN_MIPS4_32_NOT_32R6_64R6,
         GPR_64;
  defm : MovnPats<GPR64, GPR32, MOVN_I64_I, XOR64>, INSN_MIPS4_32_NOT_32R6_64R6,
         GPR_64;
  defm : MovnPats<GPR64, GPR64, MOVN_I64_I64, XOR64>,
         INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 230-233
```tablegen
  defm : MovzPats0<GPR32, FGR32, MOVZ_I_S, SLT, SLTu, SLTi, SLTiu>,
         INSN_MIPS4_32_NOT_32R6_64R6;
  defm : MovzPats1<GPR32, FGR32, MOVZ_I_S, XOR>, INSN_MIPS4_32_NOT_32R6_64R6;
  defm : MovnPats<GPR32, FGR32, MOVN_I_S, XOR>, INSN_MIPS4_32_NOT_32R6_64R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 235-240
```tablegen
  defm : MovzPats0<GPR64, FGR32, MOVZ_I_S, SLT64, SLTu64, SLTi64, SLTiu64>,
         INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
  defm : MovzPats1<GPR64, FGR32, MOVZ_I64_S, XOR64>,
         INSN_MIPS4_32_NOT_32R6_64R6, GPR_64;
  defm : MovnPats<GPR64, FGR32, MOVN_I64_S, XOR64>, INSN_MIPS4_32_NOT_32R6_64R6,
         GPR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 242-247
```tablegen
  defm : MovzPats0<GPR32, AFGR64, MOVZ_I_D32, SLT, SLTu, SLTi, SLTiu>,
         INSN_MIPS4_32_NOT_32R6_64R6, FGR_32;
  defm : MovzPats1<GPR32, AFGR64, MOVZ_I_D32, XOR>, INSN_MIPS4_32_NOT_32R6_64R6,
         FGR_32;
  defm : MovnPats<GPR32, AFGR64, MOVN_I_D32, XOR>, INSN_MIPS4_32_NOT_32R6_64R6,
         FGR_32;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 249-266
```tablegen
  defm : MovzPats0<GPR32, FGR64, MOVZ_I_D64, SLT, SLTu, SLTi, SLTiu>,
         INSN_MIPS4_32_NOT_32R6_64R6, FGR_64;
  defm : MovzPats0<GPR64, FGR64, MOVZ_I_D64, SLT64, SLTu64, SLTi64, SLTiu64>,
         INSN_MIPS4_32_NOT_32R6_64R6, FGR_64;
  defm : MovzPats1<GPR32, FGR64, MOVZ_I_D64, XOR>, INSN_MIPS4_32_NOT_32R6_64R6,
         FGR_64;
  defm : MovzPats1<GPR64, FGR64, MOVZ_I64_D64, XOR64>,
         INSN_MIPS4_32_NOT_32R6_64R6, FGR_64;
  defm : MovnPats<GPR32, FGR64, MOVN_I_D64, XOR>, INSN_MIPS4_32_NOT_32R6_64R6,
         FGR_64;
  defm : MovnPats<GPR64, FGR64, MOVN_I64_D64, XOR64>,
         INSN_MIPS4_32_NOT_32R6_64R6, FGR_64;
}
// For targets that don't have conditional-move instructions
// we have to match SELECT nodes with pseudo instructions.
let usesCustomInserter = 1, hasNoSchedulingInfo = 1 in {
  class Select_Pseudo<RegisterOperand RC> :
    PseudoSE<(outs RC:$dst), (ins GPR32Opnd:$cond, RC:$T, RC:$F),
```
- EN: Declares reusable TableGen class `Select_Pseudo` for `MipsCondMov`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsCondMov` 声明可复用的 TableGen 类 `Select_Pseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 267-268
```tablegen
            [(set RC:$dst, (select GPR32Opnd:$cond, RC:$T, RC:$F))]>,
    ISA_MIPS1_NOT_4_32;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 270-273
```tablegen
  class SelectFP_Pseudo_T<RegisterOperand RC> :
    PseudoSE<(outs RC:$dst), (ins FCCRegsOpnd:$cond, RC:$T, RC:$F),
             [(set RC:$dst, (MipsCMovFP_T RC:$T, FCCRegsOpnd:$cond, RC:$F))]>,
    ISA_MIPS1_NOT_4_32;
```
- EN: Declares reusable TableGen class `SelectFP_Pseudo_T` for `MipsCondMov`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsCondMov` 声明可复用的 TableGen 类 `SelectFP_Pseudo_T`，通常用于抽象共享字段、谓词或编码结构。

### Lines 275-279
```tablegen
  class SelectFP_Pseudo_F<RegisterOperand RC> :
    PseudoSE<(outs RC:$dst), (ins FCCRegsOpnd:$cond, RC:$T, RC:$F),
             [(set RC:$dst, (MipsCMovFP_F RC:$T, FCCRegsOpnd:$cond, RC:$F))]>,
    ISA_MIPS1_NOT_4_32;
}
```
- EN: Declares reusable TableGen class `SelectFP_Pseudo_F` for `MipsCondMov`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsCondMov` 声明可复用的 TableGen 类 `SelectFP_Pseudo_F`，通常用于抽象共享字段、谓词或编码结构。

### Lines 281-285
```tablegen
def PseudoSELECT_I : Select_Pseudo<GPR32Opnd>;
def PseudoSELECT_I64 : Select_Pseudo<GPR64Opnd>;
def PseudoSELECT_S : Select_Pseudo<FGR32Opnd>;
def PseudoSELECT_D32 : Select_Pseudo<AFGR64Opnd>, FGR_32;
def PseudoSELECT_D64 : Select_Pseudo<FGR64Opnd>, FGR_64;
```
- EN: Defines TableGen record `PseudoSELECT_I` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PseudoSELECT_I`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 287-291
```tablegen
def PseudoSELECTFP_T_I : SelectFP_Pseudo_T<GPR32Opnd>;
def PseudoSELECTFP_T_I64 : SelectFP_Pseudo_T<GPR64Opnd>;
def PseudoSELECTFP_T_S : SelectFP_Pseudo_T<FGR32Opnd>;
def PseudoSELECTFP_T_D32 : SelectFP_Pseudo_T<AFGR64Opnd>, FGR_32;
def PseudoSELECTFP_T_D64 : SelectFP_Pseudo_T<FGR64Opnd>, FGR_64;
```
- EN: Defines TableGen record `PseudoSELECTFP_T_I` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PseudoSELECTFP_T_I`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 293-297
```tablegen
def PseudoSELECTFP_F_I : SelectFP_Pseudo_F<GPR32Opnd>;
def PseudoSELECTFP_F_I64 : SelectFP_Pseudo_F<GPR64Opnd>;
def PseudoSELECTFP_F_S : SelectFP_Pseudo_F<FGR32Opnd>;
def PseudoSELECTFP_F_D32 : SelectFP_Pseudo_F<AFGR64Opnd>, FGR_32;
def PseudoSELECTFP_F_D64 : SelectFP_Pseudo_F<FGR64Opnd>, FGR_64;
```
- EN: Defines TableGen record `PseudoSELECTFP_F_I` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PseudoSELECTFP_F_I`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 299-304
```tablegen
let usesCustomInserter = 1, hasNoSchedulingInfo = 1 in {
class D_SELECT_CLASS<RegisterOperand RC> :
  PseudoSE<(outs RC:$dst1, RC:$dst2),
           (ins GPR32Opnd:$cond, RC:$a1, RC:$a2, RC:$b1, RC:$b2), []>,
  ISA_MIPS1_NOT_4_32;
}
```
- EN: Declares reusable TableGen class `D_SELECT_CLASS` for `MipsCondMov`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsCondMov` 声明可复用的 TableGen 类 `D_SELECT_CLASS`，通常用于抽象共享字段、谓词或编码结构。

### Lines 306-307
```tablegen
def PseudoD_SELECT_I   : D_SELECT_CLASS<GPR32Opnd>;
def PseudoD_SELECT_I64 : D_SELECT_CLASS<GPR64Opnd>;
```
- EN: Defines TableGen record `PseudoD_SELECT_I` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PseudoD_SELECT_I`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

## Key Concepts / 关键概念

- EN: Primary role: target descriptions, instruction records, and code-generation metadata.
  - CN: 核心职责：目标描述、指令记录以及代码生成元数据。
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
