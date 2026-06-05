# ARMInstrThumb2.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMInstrThumb2.td`
- Repository: `llvm-project`
- Purpose (EN): This file describes the Thumb2 instruction set.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMInstrThumb2`，涵盖指令定义与目标操作码元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===-- ARMInstrThumb2.td - Thumb2 support for ARM ---------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the Thumb2 instruction set.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-21
```tablegen
// IT block predicate field
def it_pred_asmoperand : AsmOperandClass {
  let Name = "ITCondCode";
  let ParserMethod = "parseITCondCode";
}
def it_pred : Operand<i32> {
  let PrintMethod = "printMandatoryPredicateOperand";
  let ParserMatchClass = it_pred_asmoperand;
}
```
- EN: Defines TableGen record `it_pred_asmoperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `it_pred_asmoperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 23-29
```tablegen
// IT block condition mask
def it_mask_asmoperand : AsmOperandClass { let Name = "ITMask"; }
def it_mask : Operand<i32> {
  let PrintMethod = "printThumbITMask";
  let ParserMatchClass = it_mask_asmoperand;
  let EncoderMethod = "getITMaskOpValue";
}
```
- EN: Defines TableGen record `it_mask_asmoperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `it_mask_asmoperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 31-41
```tablegen
// t2_shift_imm: An integer that encodes a shift amount and the type of shift
// (asr or lsl). The 6-bit immediate encodes as:
//    {5}     0 ==> lsl
//            1     asr
//    {4-0}   imm5 shift amount.
//            asr #32 not allowed
def t2_shift_imm : Operand<i32> {
  let PrintMethod = "printShiftImmOperand";
  let ParserMatchClass = ShifterImmAsmOperand;
  let DecoderMethod = "DecodeT2ShifterImmOperand";
}
```
- EN: Defines TableGen record `t2_shift_imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2_shift_imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 43-52
```tablegen
def mve_shift_imm : AsmOperandClass {
  let Name = "MVELongShift";
  let RenderMethod = "addImmOperands";
  let DiagnosticString = "operand must be an immediate in the range [1,32]";
}
def long_shift : Operand<i32>,
                 ImmLeaf<i32, [{ return Imm > 0 && Imm <= 32; }]> {
  let ParserMatchClass = mve_shift_imm;
  let DecoderMethod = "DecodeLongShiftOperand";
}
```
- EN: Defines TableGen record `mve_shift_imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mve_shift_imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 54-64
```tablegen
// Shifted operands. No register controlled shifts for Thumb2.
// Note: We do not support rrx shifted operands yet.
def t2_so_reg : Operand<i32>,    // reg imm
                ComplexPattern<i32, 2, "SelectShiftImmShifterOperand",
                               [shl,srl,sra,rotr]> {
  let EncoderMethod = "getT2SORegOpValue";
  let PrintMethod = "printT2SOOperand";
  let DecoderMethod = "DecodeSORegImmOperand";
  let ParserMatchClass = ShiftedImmAsmOperand;
  let MIOperandInfo = (ops rGPR, i32imm);
}
```
- EN: Defines TableGen record `t2_so_reg` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2_so_reg`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 66-70
```tablegen
// Same as above, but only matching on a single use node.
def t2_so_reg_oneuse : Operand<i32>,
                       ComplexPattern<i32, 2,
                                      "SelectShiftImmShifterOperandOneUse",
                                      [shl,srl,sra,rotr]>;
```
- EN: Defines TableGen record `t2_so_reg_oneuse` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2_so_reg_oneuse`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 72-76
```tablegen
// t2_so_imm_not_XFORM - Return the complement of a t2_so_imm value
def t2_so_imm_not_XFORM : SDNodeXForm<imm, [{
  return CurDAG->getTargetConstant(~((uint32_t)N->getZExtValue()), SDLoc(N),
                                   MVT::i32);
}]>;
```
- EN: Defines TableGen record `t2_so_imm_not_XFORM` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `t2_so_imm_not_XFORM`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 78-82
```tablegen
// t2_so_imm_neg_XFORM - Return the negation of a t2_so_imm value
def t2_so_imm_neg_XFORM : SDNodeXForm<imm, [{
  return CurDAG->getSignedTargetConstant(-((int)N->getZExtValue()), SDLoc(N),
                                         MVT::i32);
}]>;
```
- EN: Defines TableGen record `t2_so_imm_neg_XFORM` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `t2_so_imm_neg_XFORM`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 84-91
```tablegen
// so_imm_notSext_XFORM - Return a so_imm value packed into the format
// described for so_imm_notSext def below, with sign extension from 16
// bits.
def t2_so_imm_notSext16_XFORM : SDNodeXForm<imm, [{
  APInt apIntN = N->getAPIntValue();
  unsigned N16bitSignExt = apIntN.trunc(16).sext(32).getZExtValue();
  return CurDAG->getTargetConstant(~N16bitSignExt, SDLoc(N), MVT::i32);
}]>;
```
- EN: Defines TableGen record `below` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `below`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 93-98
```tablegen
// t2_so_imm - Match a 32-bit immediate operand, which is an
// 8-bit immediate rotated by an arbitrary number of bits, or an 8-bit
// immediate splatted into multiple bytes of the word.
def t2_so_imm_asmoperand : AsmOperandClass {
  let Name = "T2SOImm";
  let RenderMethod = "addImmOperands";
```
- EN: Defines TableGen record `t2_so_imm_asmoperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2_so_imm_asmoperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 100-107
```tablegen
}
def t2_so_imm : Operand<i32>, ImmLeaf<i32, [{
    return ARM_AM::getT2SOImmVal(Imm) != -1;
  }]> {
  let ParserMatchClass = t2_so_imm_asmoperand;
  let EncoderMethod = "getT2SOImmOpValue";
  let DecoderMethod = "DecodeT2SOImm";
}
```
- EN: Defines TableGen record `t2_so_imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2_so_imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 109-119
```tablegen
// t2_so_imm_not - Match an immediate that is a complement
// of a t2_so_imm.
// Note: this pattern doesn't require an encoder method and such, as it's
// only used on aliases (Pat<> and InstAlias<>). The actual encoding
// is handled by the destination instructions, which use t2_so_imm.
def t2_so_imm_not_asmoperand : AsmOperandClass { let Name = "T2SOImmNot"; }
def t2_so_imm_not : Operand<i32>, PatLeaf<(imm), [{
  return ARM_AM::getT2SOImmVal(~((uint32_t)N->getZExtValue())) != -1;
}], t2_so_imm_not_XFORM> {
  let ParserMatchClass = t2_so_imm_not_asmoperand;
}
```
- EN: Defines TableGen record `t2_so_imm_not_asmoperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2_so_imm_not_asmoperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 121-130
```tablegen
// t2_so_imm_notSext - match an immediate that is a complement of a t2_so_imm
// if the upper 16 bits are zero.
def t2_so_imm_notSext : Operand<i32>, PatLeaf<(imm), [{
    APInt apIntN = N->getAPIntValue();
    if (!apIntN.isIntN(16)) return false;
    unsigned N16bitSignExt = apIntN.trunc(16).sext(32).getZExtValue();
    return ARM_AM::getT2SOImmVal(~N16bitSignExt) != -1;
  }], t2_so_imm_notSext16_XFORM> {
  let ParserMatchClass = t2_so_imm_not_asmoperand;
}
```
- EN: Defines TableGen record `t2_so_imm_notSext` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2_so_imm_notSext`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 132-138
```tablegen
// t2_so_imm_neg - Match an immediate that is a negation of a t2_so_imm.
def t2_so_imm_neg_asmoperand : AsmOperandClass { let Name = "T2SOImmNeg"; }
def t2_so_imm_neg : Operand<i32>, ImmLeaf<i32, [{
  return Imm && ARM_AM::getT2SOImmVal(-(uint32_t)Imm) != -1;
}], t2_so_imm_neg_XFORM> {
  let ParserMatchClass = t2_so_imm_neg_asmoperand;
}
```
- EN: Defines TableGen record `t2_so_imm_neg_asmoperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2_so_imm_neg_asmoperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 140-146
```tablegen
/// imm0_4095 predicate - True if the 32-bit immediate is in the range [0,4095].
def imm0_4095_asmoperand: ImmAsmOperand<0,4095> { let Name = "Imm0_4095"; }
def imm0_4095 : Operand<i32>, ImmLeaf<i32, [{
  return Imm >= 0 && Imm < 4096;
}]> {
  let ParserMatchClass = imm0_4095_asmoperand;
}
```
- EN: Defines TableGen record `imm0_4095_asmoperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `imm0_4095_asmoperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 148-153
```tablegen
def imm0_4095_neg_asmoperand: AsmOperandClass { let Name = "Imm0_4095Neg"; }
def imm0_4095_neg : Operand<i32>, PatLeaf<(i32 imm), [{
 return (uint32_t)(-N->getZExtValue()) < 4096;
}], imm_neg_XFORM> {
  let ParserMatchClass = imm0_4095_neg_asmoperand;
}
```
- EN: Defines TableGen record `imm0_4095_neg_asmoperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `imm0_4095_neg_asmoperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 155-158
```tablegen
def imm1_255_neg : PatLeaf<(i32 imm), [{
  uint32_t Val = -N->getZExtValue();
  return (Val > 0 && Val < 255);
}], imm_neg_XFORM>;
```
- EN: Defines TableGen record `imm1_255_neg` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `imm1_255_neg`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 160-162
```tablegen
def imm0_255_not : PatLeaf<(i32 imm), [{
  return (uint32_t)(~N->getZExtValue()) < 255;
}], imm_not_XFORM>;
```
- EN: Defines TableGen record `imm0_255_not` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `imm0_255_not`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 164-167
```tablegen
def lo5AllOne : PatLeaf<(i32 imm), [{
  // Returns true if all low 5-bits are 1.
  return (((uint32_t)N->getZExtValue()) & 0x1FUL) == 0x1FUL;
}]>;
```
- EN: Defines TableGen record `lo5AllOne` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `lo5AllOne`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 169-169
```tablegen
// Define Thumb2 specific addressing modes.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 171-179
```tablegen
// t2_addr_offset_none := reg
def MemNoOffsetT2AsmOperand
  : AsmOperandClass { let Name = "MemNoOffsetT2"; }
def t2_addr_offset_none : MemOperand {
  let PrintMethod = "printAddrMode7Operand";
  let DecoderMethod = "DecodeGPRnopcRegisterClass";
  let ParserMatchClass = MemNoOffsetT2AsmOperand;
  let MIOperandInfo = (ops GPRnopc:$base);
}
```
- EN: Defines TableGen record `MemNoOffsetT2AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MemNoOffsetT2AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 181-189
```tablegen
// t2_nosp_addr_offset_none := reg
def MemNoOffsetT2NoSpAsmOperand
  : AsmOperandClass { let Name = "MemNoOffsetT2NoSp"; }
def t2_nosp_addr_offset_none : MemOperand {
  let PrintMethod = "printAddrMode7Operand";
  let DecoderMethod = "DecoderGPRRegisterClass";
  let ParserMatchClass = MemNoOffsetT2NoSpAsmOperand;
  let MIOperandInfo = (ops rGPR:$base);
}
```
- EN: Defines TableGen record `MemNoOffsetT2NoSpAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MemNoOffsetT2NoSpAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 191-200
```tablegen
// t2addrmode_imm12  := reg + imm12
def t2addrmode_imm12_asmoperand : AsmOperandClass {let Name="MemUImm12Offset";}
def t2addrmode_imm12 : MemOperand,
                       ComplexPattern<i32, 2, "SelectT2AddrModeImm12", []> {
  let PrintMethod = "printAddrModeImm12Operand<false>";
  let EncoderMethod = "getAddrModeImm12OpValue";
  let DecoderMethod = "DecodeT2AddrModeImm12";
  let ParserMatchClass = t2addrmode_imm12_asmoperand;
  let MIOperandInfo = (ops GPR:$base, i32imm:$offsimm);
}
```
- EN: Defines TableGen record `t2addrmode_imm12_asmoperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2addrmode_imm12_asmoperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 202-206
```tablegen
// t2ldrlabel  := imm12
def t2ldrlabel : MemOperand {
  let EncoderMethod = "getAddrModeImm12OpValue";
  let PrintMethod = "printThumbLdrLabelOperand";
}
```
- EN: Defines TableGen record `t2ldrlabel` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2ldrlabel`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 208-213
```tablegen
def t2ldr_pcrel_imm12_asmoperand : AsmOperandClass {let Name = "MemPCRelImm12";}
def t2ldr_pcrel_imm12 : Operand<i32> {
  let ParserMatchClass = t2ldr_pcrel_imm12_asmoperand;
  // used for assembler pseudo instruction and maps to t2ldrlabel, so
  // doesn't need encoder or print methods of its own.
}
```
- EN: Defines TableGen record `t2ldr_pcrel_imm12_asmoperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2ldr_pcrel_imm12_asmoperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 215-219
```tablegen
// ADR instruction labels.
def t2adrlabel : Operand<i32> {
  let EncoderMethod = "getT2AdrLabelOpValue";
  let PrintMethod = "printAdrLabelOperand<0>";
}
```
- EN: Defines TableGen record `t2adrlabel` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2adrlabel`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 221-232
```tablegen
// t2addrmode_posimm8  := reg + imm8
def MemPosImm8OffsetAsmOperand : AsmOperandClass {
  let Name="MemPosImm8Offset";
  let RenderMethod = "addMemImmOffsetOperands";
}
def t2addrmode_posimm8 : MemOperand {
  let PrintMethod = "printT2AddrModeImm8Operand<false>";
  let EncoderMethod = "getT2AddrModeImmOpValue<8,0>";
  let DecoderMethod = "DecodeT2AddrModeImm8";
  let ParserMatchClass = MemPosImm8OffsetAsmOperand;
  let MIOperandInfo = (ops GPR:$base, i32imm:$offsimm);
}
```
- EN: Defines TableGen record `MemPosImm8OffsetAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MemPosImm8OffsetAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 234-246
```tablegen
// t2addrmode_negimm8  := reg - imm8
def MemNegImm8OffsetAsmOperand : AsmOperandClass {
  let Name="MemNegImm8Offset";
  let RenderMethod = "addMemImmOffsetOperands";
}
def t2addrmode_negimm8 : MemOperand,
                      ComplexPattern<i32, 2, "SelectT2AddrModeImm8", []> {
  let PrintMethod = "printT2AddrModeImm8Operand<false>";
  let EncoderMethod = "getT2AddrModeImmOpValue<8,0>";
  let DecoderMethod = "DecodeT2AddrModeImm8";
  let ParserMatchClass = MemNegImm8OffsetAsmOperand;
  let MIOperandInfo = (ops GPR:$base, i32imm:$offsimm);
}
```
- EN: Defines TableGen record `MemNegImm8OffsetAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MemNegImm8OffsetAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 248-259
```tablegen
// t2addrmode_imm8  := reg +/- imm8
def MemImm8OffsetAsmOperand : AsmOperandClass {
  let Name = "MemImm8Offset";
  let RenderMethod = "addMemImmOffsetOperands";
}
class T2AddrMode_Imm8 : MemOperand,
                        ComplexPattern<i32, 2, "SelectT2AddrModeImm8", []> {
  let EncoderMethod = "getT2AddrModeImmOpValue<8,0>";
  let DecoderMethod = "DecodeT2AddrModeImm8";
  let ParserMatchClass = MemImm8OffsetAsmOperand;
  let MIOperandInfo = (ops GPR:$base, i32imm:$offsimm);
}
```
- EN: Declares reusable TableGen class `T2AddrMode_Imm8` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2AddrMode_Imm8`，通常用于抽象共享字段、谓词或编码结构。

### Lines 261-263
```tablegen
def t2addrmode_imm8 : T2AddrMode_Imm8 {
  let PrintMethod = "printT2AddrModeImm8Operand<false>";
}
```
- EN: Defines TableGen record `t2addrmode_imm8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2addrmode_imm8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 265-267
```tablegen
def t2addrmode_imm8_pre : T2AddrMode_Imm8 {
  let PrintMethod = "printT2AddrModeImm8Operand<true>";
}
```
- EN: Defines TableGen record `t2addrmode_imm8_pre` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2addrmode_imm8_pre`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 269-275
```tablegen
def t2am_imm8_offset : MemOperand,
                       ComplexPattern<i32, 1, "SelectT2AddrModeImm8Offset"> {
  let PrintMethod = "printT2AddrModeImm8OffsetOperand";
  let EncoderMethod = "getT2AddrModeImm8OffsetOpValue";
  let DecoderMethod = "DecodeT2Imm8";
  let WantsRoot = true;
}
```
- EN: Defines TableGen record `t2am_imm8_offset` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2am_imm8_offset`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 277-285
```tablegen
// t2addrmode_imm8s4  := reg +/- (imm8 << 2)
def MemImm8s4OffsetAsmOperand : AsmOperandClass {let Name = "MemImm8s4Offset";}
class T2AddrMode_Imm8s4 : MemOperand,
                          ComplexPattern<i32, 2, "SelectT2AddrModeImm8<2>", []> {
  let EncoderMethod = "getT2AddrModeImm8s4OpValue";
  let DecoderMethod = "DecodeT2AddrModeImm8s4";
  let ParserMatchClass = MemImm8s4OffsetAsmOperand;
  let MIOperandInfo = (ops GPR:$base, i32imm:$offsimm);
}
```
- EN: Declares reusable TableGen class `T2AddrMode_Imm8s4` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2AddrMode_Imm8s4`，通常用于抽象共享字段、谓词或编码结构。

### Lines 287-289
```tablegen
def t2addrmode_imm8s4 : T2AddrMode_Imm8s4 {
  let PrintMethod = "printT2AddrModeImm8s4Operand<false>";
}
```
- EN: Defines TableGen record `t2addrmode_imm8s4` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2addrmode_imm8s4`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 291-293
```tablegen
def t2addrmode_imm8s4_pre : T2AddrMode_Imm8s4 {
  let PrintMethod = "printT2AddrModeImm8s4Operand<true>";
}
```
- EN: Defines TableGen record `t2addrmode_imm8s4_pre` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2addrmode_imm8s4_pre`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 295-300
```tablegen
def t2am_imm8s4_offset_asmoperand : AsmOperandClass { let Name = "Imm8s4"; }
def t2am_imm8s4_offset : MemOperand {
  let PrintMethod = "printT2AddrModeImm8s4OffsetOperand";
  let EncoderMethod = "getT2ScaledImmOpValue<8,2>";
  let DecoderMethod = "DecodeT2Imm8S4";
}
```
- EN: Defines TableGen record `t2am_imm8s4_offset_asmoperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2am_imm8s4_offset_asmoperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 302-309
```tablegen
// t2addrmode_imm7s4  := reg +/- (imm7 << 2)
def MemImm7s4OffsetAsmOperand : AsmOperandClass {let Name = "MemImm7s4Offset";}
class T2AddrMode_Imm7s4 : MemOperand {
  let EncoderMethod = "getT2AddrModeImm7s4OpValue";
  let DecoderMethod = "DecodeT2AddrModeImm7<2,0>";
  let ParserMatchClass = MemImm7s4OffsetAsmOperand;
  let MIOperandInfo = (ops GPRnopc:$base, i32imm:$offsimm);
}
```
- EN: Declares reusable TableGen class `T2AddrMode_Imm7s4` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2AddrMode_Imm7s4`，通常用于抽象共享字段、谓词或编码结构。

### Lines 311-314
```tablegen
def t2addrmode_imm7s4 : T2AddrMode_Imm7s4 {
  // They are printed the same way as the imm8 version
  let PrintMethod = "printT2AddrModeImm8s4Operand<false>";
}
```
- EN: Defines TableGen record `t2addrmode_imm7s4` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2addrmode_imm7s4`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 316-319
```tablegen
def t2addrmode_imm7s4_pre : T2AddrMode_Imm7s4 {
  // They are printed the same way as the imm8 version
  let PrintMethod = "printT2AddrModeImm8s4Operand<true>";
}
```
- EN: Defines TableGen record `t2addrmode_imm7s4_pre` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2addrmode_imm7s4_pre`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 321-328
```tablegen
def t2am_imm7s4_offset_asmoperand : AsmOperandClass { let Name = "Imm7s4"; }
def t2am_imm7s4_offset : MemOperand {
  // They are printed the same way as the imm8 version
  let PrintMethod = "printT2AddrModeImm8s4OffsetOperand";
  let ParserMatchClass = t2am_imm7s4_offset_asmoperand;
  let EncoderMethod = "getT2ScaledImmOpValue<7,2>";
  let DecoderMethod = "DecodeT2Imm7S4";
}
```
- EN: Defines TableGen record `t2am_imm7s4_offset_asmoperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2am_imm7s4_offset_asmoperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 330-341
```tablegen
// t2addrmode_imm0_1020s4  := reg + (imm8 << 2)
def MemImm0_1020s4OffsetAsmOperand : AsmOperandClass {
  let Name = "MemImm0_1020s4Offset";
}
def t2addrmode_imm0_1020s4 : MemOperand,
                         ComplexPattern<i32, 2, "SelectT2AddrModeExclusive"> {
  let PrintMethod = "printT2AddrModeImm0_1020s4Operand";
  let EncoderMethod = "getT2AddrModeImm0_1020s4OpValue";
  let DecoderMethod = "DecodeT2AddrModeImm0_1020s4";
  let ParserMatchClass = MemImm0_1020s4OffsetAsmOperand;
  let MIOperandInfo = (ops GPRnopc:$base, i32imm:$offsimm);
}
```
- EN: Defines TableGen record `MemImm0_1020s4OffsetAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MemImm0_1020s4OffsetAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 343-352
```tablegen
// t2addrmode_so_reg  := reg + (reg << imm2)
def t2addrmode_so_reg_asmoperand : AsmOperandClass {let Name="T2MemRegOffset";}
def t2addrmode_so_reg : MemOperand,
                        ComplexPattern<i32, 3, "SelectT2AddrModeSoReg", []> {
  let PrintMethod = "printT2AddrModeSoRegOperand";
  let EncoderMethod = "getT2AddrModeSORegOpValue";
  let DecoderMethod = "DecodeT2AddrModeSOReg";
  let ParserMatchClass = t2addrmode_so_reg_asmoperand;
  let MIOperandInfo = (ops GPRnopc:$base, rGPR:$offsreg, i32imm:$offsimm);
}
```
- EN: Defines TableGen record `t2addrmode_so_reg_asmoperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2addrmode_so_reg_asmoperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 354-366
```tablegen
// Addresses for the TBB/TBH instructions.
def addrmode_tbb_asmoperand : AsmOperandClass { let Name = "MemTBB"; }
def addrmode_tbb : MemOperand {
  let PrintMethod = "printAddrModeTBB";
  let ParserMatchClass = addrmode_tbb_asmoperand;
  let MIOperandInfo = (ops GPR:$Rn, rGPR:$Rm);
}
def addrmode_tbh_asmoperand : AsmOperandClass { let Name = "MemTBH"; }
def addrmode_tbh : MemOperand {
  let PrintMethod = "printAddrModeTBH";
  let ParserMatchClass = addrmode_tbh_asmoperand;
  let MIOperandInfo = (ops GPR:$Rn, rGPR:$Rm);
}
```
- EN: Defines TableGen record `addrmode_tbb_asmoperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `addrmode_tbb_asmoperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 368-368
```tablegen
// Define ARMv8.1-M specific addressing modes.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 370-383
```tablegen
// Label operands for BF/BFL/WLS/DLS/LE
class BFLabelOp<string signed, string isNeg, string zeroPermitted, string size,
                string fixup>
  : Operand<OtherVT> {
  let EncoderMethod = !strconcat("getBFTargetOpValue<", isNeg, ", ",
                                 fixup, ">");
  let OperandType = "OPERAND_PCREL";
  let DecoderMethod = !strconcat("DecodeBFLabelOperand<", signed, ", ",
                                 isNeg, ", ", zeroPermitted, ", ", size, ">");
}
def bflabel_u4  : BFLabelOp<"false", "false", "false", "4",  "ARM::fixup_bf_branch">;
def bflabel_s12 : BFLabelOp<"true",  "false", "true",  "12", "ARM::fixup_bfc_target">;
def bflabel_s16 : BFLabelOp<"true",  "false", "true",  "16", "ARM::fixup_bf_target">;
def bflabel_s18 : BFLabelOp<"true",  "false", "true",  "18", "ARM::fixup_bfl_target">;
```
- EN: Declares reusable TableGen class `BFLabelOp` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `BFLabelOp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 385-402
```tablegen
def wlslabel_u11_asmoperand : AsmOperandClass {
  let Name = "WLSLabel";
  let RenderMethod = "addImmOperands";
  let PredicateMethod = "isUnsignedOffset<11, 1>";
  let DiagnosticString =
    "loop end is out of range or not a positive multiple of 2";
}
def wlslabel_u11 : BFLabelOp<"false", "false", "true",  "11", "ARM::fixup_wls"> {
  let ParserMatchClass = wlslabel_u11_asmoperand;
}
def lelabel_u11_asmoperand : AsmOperandClass {
  let Name = "LELabel";
  let RenderMethod = "addImmOperands";
  let PredicateMethod = "isLEOffset";
  let DiagnosticString =
    "loop start is out of range or not a negative multiple of 2";
}
def lelabel_u11 : BFLabelOp<"false", "true",  "true",  "11", "ARM::fixup_le"> {
```
- EN: Defines TableGen record `wlslabel_u11_asmoperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `wlslabel_u11_asmoperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 403-404
```tablegen
  let ParserMatchClass = lelabel_u11_asmoperand;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 406-410
```tablegen
def bfafter_target : Operand<OtherVT> {
    let EncoderMethod = "getBFAfterTargetOpValue";
    let OperandType = "OPERAND_PCREL";
    let DecoderMethod = "DecodeBFAfterTargetOperand";
}
```
- EN: Defines TableGen record `bfafter_target` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `bfafter_target`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 412-423
```tablegen
// pred operand excluding AL
def pred_noal_asmoperand : AsmOperandClass {
  let Name = "CondCodeNoAL";
  let RenderMethod = "addITCondCodeOperands";
  let PredicateMethod = "isITCondCodeNoAL";
  let ParserMethod = "parseITCondCode";
}
def pred_noal : Operand<i32> {
  let PrintMethod = "printMandatoryPredicateOperand";
  let ParserMatchClass = pred_noal_asmoperand;
  let DecoderMethod = "DecodePredNoALOperand";
}
```
- EN: Defines TableGen record `pred_noal_asmoperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `pred_noal_asmoperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 426-439
```tablegen
// CSEL aliases inverted predicate
def pred_noal_inv_asmoperand : AsmOperandClass {
  let Name = "CondCodeNoALInv";
  let RenderMethod = "addITCondCodeInvOperands";
  let PredicateMethod = "isITCondCodeNoAL";
  let ParserMethod = "parseITCondCode";
}
def pred_noal_inv : Operand<i32> {
  let PrintMethod = "printMandatoryInvertedPredicateOperand";
  let ParserMatchClass = pred_noal_inv_asmoperand;
}
//===----------------------------------------------------------------------===//
// Multiclass helpers...
//
```
- EN: Defines TableGen record `pred_noal_inv_asmoperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `pred_noal_inv_asmoperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 442-446
```tablegen
class T2OneRegImm<dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : T2I<oops, iops, itin, opc, asm, pattern> {
  bits<4> Rd;
  bits<12> imm;
```
- EN: Declares reusable TableGen class `T2OneRegImm` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2OneRegImm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 448-452
```tablegen
  let Inst{11-8}  = Rd;
  let Inst{26}    = imm{11};
  let Inst{14-12} = imm{10-8};
  let Inst{7-0}   = imm{7-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 455-460
```tablegen
class T2sOneRegImm<dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : T2sI<oops, iops, itin, opc, asm, pattern> {
  bits<4> Rd;
  bits<4> Rn;
  bits<12> imm;
```
- EN: Declares reusable TableGen class `T2sOneRegImm` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2sOneRegImm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 462-466
```tablegen
  let Inst{11-8}  = Rd;
  let Inst{26}    = imm{11};
  let Inst{14-12} = imm{10-8};
  let Inst{7-0}   = imm{7-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 468-472
```tablegen
class T2OneRegCmpImm<dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : T2I<oops, iops, itin, opc, asm, pattern> {
  bits<4> Rn;
  bits<12> imm;
```
- EN: Declares reusable TableGen class `T2OneRegCmpImm` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2OneRegCmpImm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 474-478
```tablegen
  let Inst{19-16}  = Rn;
  let Inst{26}    = imm{11};
  let Inst{14-12} = imm{10-8};
  let Inst{7-0}   = imm{7-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 481-485
```tablegen
class T2OneRegShiftedReg<dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : T2I<oops, iops, itin, opc, asm, pattern> {
  bits<4> Rd;
  bits<12> ShiftedRm;
```
- EN: Declares reusable TableGen class `T2OneRegShiftedReg` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2OneRegShiftedReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 487-492
```tablegen
  let Inst{11-8}  = Rd;
  let Inst{3-0}   = ShiftedRm{3-0};
  let Inst{5-4}   = ShiftedRm{6-5};
  let Inst{14-12} = ShiftedRm{11-9};
  let Inst{7-6}   = ShiftedRm{8-7};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 494-498
```tablegen
class T2sOneRegShiftedReg<dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : T2sI<oops, iops, itin, opc, asm, pattern> {
  bits<4> Rd;
  bits<12> ShiftedRm;
```
- EN: Declares reusable TableGen class `T2sOneRegShiftedReg` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2sOneRegShiftedReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 500-505
```tablegen
  let Inst{11-8}  = Rd;
  let Inst{3-0}   = ShiftedRm{3-0};
  let Inst{5-4}   = ShiftedRm{6-5};
  let Inst{14-12} = ShiftedRm{11-9};
  let Inst{7-6}   = ShiftedRm{8-7};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 507-511
```tablegen
class T2OneRegCmpShiftedReg<dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : T2I<oops, iops, itin, opc, asm, pattern> {
  bits<4> Rn;
  bits<12> ShiftedRm;
```
- EN: Declares reusable TableGen class `T2OneRegCmpShiftedReg` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2OneRegCmpShiftedReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 513-518
```tablegen
  let Inst{19-16} = Rn;
  let Inst{3-0}   = ShiftedRm{3-0};
  let Inst{5-4}   = ShiftedRm{6-5};
  let Inst{14-12} = ShiftedRm{11-9};
  let Inst{7-6}   = ShiftedRm{8-7};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 520-524
```tablegen
class T2TwoReg<dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : T2I<oops, iops, itin, opc, asm, pattern> {
  bits<4> Rd;
  bits<4> Rm;
```
- EN: Declares reusable TableGen class `T2TwoReg` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2TwoReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 526-528
```tablegen
  let Inst{11-8}  = Rd;
  let Inst{3-0}   = Rm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 530-534
```tablegen
class T2sTwoReg<dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : T2sI<oops, iops, itin, opc, asm, pattern> {
  bits<4> Rd;
  bits<4> Rm;
```
- EN: Declares reusable TableGen class `T2sTwoReg` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2sTwoReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 536-538
```tablegen
  let Inst{11-8}  = Rd;
  let Inst{3-0}   = Rm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 540-544
```tablegen
class T2TwoRegCmp<dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : T2I<oops, iops, itin, opc, asm, pattern> {
  bits<4> Rn;
  bits<4> Rm;
```
- EN: Declares reusable TableGen class `T2TwoRegCmp` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2TwoRegCmp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 546-548
```tablegen
  let Inst{19-16} = Rn;
  let Inst{3-0}   = Rm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 551-556
```tablegen
class T2TwoRegImm<dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : T2I<oops, iops, itin, opc, asm, pattern> {
  bits<4> Rd;
  bits<4> Rn;
  bits<12> imm;
```
- EN: Declares reusable TableGen class `T2TwoRegImm` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2TwoRegImm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 558-563
```tablegen
  let Inst{11-8}  = Rd;
  let Inst{19-16} = Rn;
  let Inst{26}    = imm{11};
  let Inst{14-12} = imm{10-8};
  let Inst{7-0}   = imm{7-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 565-570
```tablegen
class T2sTwoRegImm<dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : T2sI<oops, iops, itin, opc, asm, pattern> {
  bits<4> Rd;
  bits<4> Rn;
  bits<12> imm;
```
- EN: Declares reusable TableGen class `T2sTwoRegImm` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2sTwoRegImm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 572-577
```tablegen
  let Inst{11-8}  = Rd;
  let Inst{19-16} = Rn;
  let Inst{26}    = imm{11};
  let Inst{14-12} = imm{10-8};
  let Inst{7-0}   = imm{7-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 579-584
```tablegen
class T2TwoRegShiftImm<dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : T2I<oops, iops, itin, opc, asm, pattern> {
  bits<4> Rd;
  bits<4> Rm;
  bits<5> imm;
```
- EN: Declares reusable TableGen class `T2TwoRegShiftImm` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2TwoRegShiftImm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 586-590
```tablegen
  let Inst{11-8}  = Rd;
  let Inst{3-0}   = Rm;
  let Inst{14-12} = imm{4-2};
  let Inst{7-6}   = imm{1-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 592-597
```tablegen
class T2sTwoRegShiftImm<dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : T2sI<oops, iops, itin, opc, asm, pattern> {
  bits<4> Rd;
  bits<4> Rm;
  bits<5> imm;
```
- EN: Declares reusable TableGen class `T2sTwoRegShiftImm` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2sTwoRegShiftImm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 599-603
```tablegen
  let Inst{11-8}  = Rd;
  let Inst{3-0}   = Rm;
  let Inst{14-12} = imm{4-2};
  let Inst{7-6}   = imm{1-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 605-610
```tablegen
class T2ThreeReg<dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : T2I<oops, iops, itin, opc, asm, pattern> {
  bits<4> Rd;
  bits<4> Rn;
  bits<4> Rm;
```
- EN: Declares reusable TableGen class `T2ThreeReg` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2ThreeReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 612-615
```tablegen
  let Inst{11-8}  = Rd;
  let Inst{19-16} = Rn;
  let Inst{3-0}   = Rm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 617-622
```tablegen
class T2ThreeRegNoP<dag oops, dag iops, InstrItinClass itin,
           string asm, list<dag> pattern>
  : T2XI<oops, iops, itin, asm, pattern> {
  bits<4> Rd;
  bits<4> Rn;
  bits<4> Rm;
```
- EN: Declares reusable TableGen class `T2ThreeRegNoP` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2ThreeRegNoP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 624-627
```tablegen
  let Inst{11-8}  = Rd;
  let Inst{19-16} = Rn;
  let Inst{3-0}   = Rm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 629-634
```tablegen
class T2sThreeReg<dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : T2sI<oops, iops, itin, opc, asm, pattern> {
  bits<4> Rd;
  bits<4> Rn;
  bits<4> Rm;
```
- EN: Declares reusable TableGen class `T2sThreeReg` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2sThreeReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 636-639
```tablegen
  let Inst{11-8}  = Rd;
  let Inst{19-16} = Rn;
  let Inst{3-0}   = Rm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 641-646
```tablegen
class T2TwoRegShiftedReg<dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : T2I<oops, iops, itin, opc, asm, pattern> {
  bits<4> Rd;
  bits<4> Rn;
  bits<12> ShiftedRm;
```
- EN: Declares reusable TableGen class `T2TwoRegShiftedReg` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2TwoRegShiftedReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 648-654
```tablegen
  let Inst{11-8}  = Rd;
  let Inst{19-16} = Rn;
  let Inst{3-0}   = ShiftedRm{3-0};
  let Inst{5-4}   = ShiftedRm{6-5};
  let Inst{14-12} = ShiftedRm{11-9};
  let Inst{7-6}   = ShiftedRm{8-7};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 656-661
```tablegen
class T2sTwoRegShiftedReg<dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : T2sI<oops, iops, itin, opc, asm, pattern> {
  bits<4> Rd;
  bits<4> Rn;
  bits<12> ShiftedRm;
```
- EN: Declares reusable TableGen class `T2sTwoRegShiftedReg` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2sTwoRegShiftedReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 663-669
```tablegen
  let Inst{11-8}  = Rd;
  let Inst{19-16} = Rn;
  let Inst{3-0}   = ShiftedRm{3-0};
  let Inst{5-4}   = ShiftedRm{6-5};
  let Inst{14-12} = ShiftedRm{11-9};
  let Inst{7-6}   = ShiftedRm{8-7};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 671-677
```tablegen
class T2FourReg<dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : T2I<oops, iops, itin, opc, asm, pattern> {
  bits<4> Rd;
  bits<4> Rn;
  bits<4> Rm;
  bits<4> Ra;
```
- EN: Declares reusable TableGen class `T2FourReg` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2FourReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 679-683
```tablegen
  let Inst{19-16} = Rn;
  let Inst{15-12} = Ra;
  let Inst{11-8}  = Rd;
  let Inst{3-0}   = Rm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 685-693
```tablegen
class T2MulLong<bits<3> opc22_20, bits<4> opc7_4,
                string opc, list<dag> pattern>
  : T2I<(outs rGPR:$RdLo, rGPR:$RdHi), (ins rGPR:$Rn, rGPR:$Rm), IIC_iMUL64,
         opc, "\t$RdLo, $RdHi, $Rn, $Rm", pattern>,
    Sched<[WriteMUL64Lo, WriteMUL64Hi, ReadMUL, ReadMUL]> {
  bits<4> RdLo;
  bits<4> RdHi;
  bits<4> Rn;
  bits<4> Rm;
```
- EN: Declares reusable TableGen class `T2MulLong` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2MulLong`，通常用于抽象共享字段、谓词或编码结构。

### Lines 695-712
```tablegen
  let Inst{31-23} = 0b111110111;
  let Inst{22-20} = opc22_20;
  let Inst{19-16} = Rn;
  let Inst{15-12} = RdLo;
  let Inst{11-8}  = RdHi;
  let Inst{7-4}   = opc7_4;
  let Inst{3-0}   = Rm;
}
class T2MlaLong<bits<3> opc22_20, bits<4> opc7_4, string opc>
  : T2I<(outs rGPR:$RdLo, rGPR:$RdHi),
        (ins rGPR:$Rn, rGPR:$Rm, rGPR:$RLo, rGPR:$RHi), IIC_iMAC64,
        opc, "\t$RdLo, $RdHi, $Rn, $Rm", []>,
        RegConstraint<"$RLo = $RdLo, $RHi = $RdHi">,
    Sched<[WriteMAC64Lo, WriteMAC64Hi, ReadMUL, ReadMUL, ReadMAC, ReadMAC]> {
  bits<4> RdLo;
  bits<4> RdHi;
  bits<4> Rn;
  bits<4> Rm;
```
- EN: Declares reusable TableGen class `T2MlaLong` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2MlaLong`，通常用于抽象共享字段、谓词或编码结构。

### Lines 714-721
```tablegen
  let Inst{31-23} = 0b111110111;
  let Inst{22-20} = opc22_20;
  let Inst{19-16} = Rn;
  let Inst{15-12} = RdLo;
  let Inst{11-8}  = RdHi;
  let Inst{7-4}   = opc7_4;
  let Inst{3-0}   = Rm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 724-741
```tablegen
/// T2I_bin_irs - Defines a set of (op reg, {so_imm|r|so_reg}) patterns for a
/// binary operation that produces a value. These are predicable and can be
/// changed to modify CPSR.
multiclass T2I_bin_irs<bits<4> opcod, string opc,
                     InstrItinClass iii, InstrItinClass iir, InstrItinClass iis,
                     SDPatternOperator opnode, bit Commutable = 0,
                     string wide = ""> {
   // shifted imm
   def ri : T2sTwoRegImm<
                (outs rGPR:$Rd), (ins rGPR:$Rn, t2_so_imm:$imm), iii,
                 opc, "\t$Rd, $Rn, $imm",
                 [(set rGPR:$Rd, (opnode rGPR:$Rn, t2_so_imm:$imm))]>,
                 Sched<[WriteALU, ReadALU]> {
     let Inst{31-27} = 0b11110;
     let Inst{25} = 0;
     let Inst{24-21} = opcod;
     let Inst{15} = 0;
   }
```
- EN: Declares TableGen `multiclass T2I_bin_irs`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass T2I_bin_irs`，它是一个可复用模板，可展开为多个相关记录。

### Lines 742-759
```tablegen
   // register
   def rr : T2sThreeReg<(outs rGPR:$Rd), (ins rGPR:$Rn, rGPR:$Rm), iir,
                 opc, !strconcat(wide, "\t$Rd, $Rn, $Rm"),
                 [(set rGPR:$Rd, (opnode rGPR:$Rn, rGPR:$Rm))]>,
                 Sched<[WriteALU, ReadALU, ReadALU]> {
     let isCommutable = Commutable;
     let Inst{31-27} = 0b11101;
     let Inst{26-25} = 0b01;
     let Inst{24-21} = opcod;
     let Inst{15} = 0b0;
     // In most of these instructions, and most versions of the Arm
     // architecture, bit 15 of this encoding is listed as (0) rather
     // than 0, i.e. setting it to 1 is UNPREDICTABLE or a soft-fail
     // rather than a hard failure. In v8.1-M, this requirement is
     // upgraded to a hard one for ORR, so that the encodings with 1
     // in this bit can be reused for other instructions (such as
     // CSEL). Setting Unpredictable{15} = 1 here would reintroduce
     // that encoding clash in the auto- generated MC decoder, so I
```
- EN: Defines TableGen record `rr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 760-777
```tablegen
     // comment it out.
     let Unpredictable{15} = !if(!eq(opcod, 0b0010), 0b0, 0b1);
     let Inst{14-12} = 0b000; // imm3
     let Inst{7-6} = 0b00; // imm2
     let Inst{5-4} = 0b00; // type
   }
   // shifted register
   def rs : T2sTwoRegShiftedReg<
                 (outs rGPR:$Rd), (ins rGPR:$Rn, t2_so_reg:$ShiftedRm), iis,
                 opc, !strconcat(wide, "\t$Rd, $Rn, $ShiftedRm"),
                 [(set rGPR:$Rd, (opnode rGPR:$Rn, t2_so_reg:$ShiftedRm))]>,
                 Sched<[WriteALUsi, ReadALU]>  {
     let Inst{31-27} = 0b11101;
     let Inst{26-25} = 0b01;
     let Inst{24-21} = opcod;
     let Inst{15} = 0;
     let Unpredictable{15} = !if(!eq(opcod, 0b0010), 0b0, 0b1); // see above
   }
```
- EN: Defines TableGen record `rs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 778-792
```tablegen
  // Assembly aliases for optional destination operand when it's the same
  // as the source operand.
  def : t2InstAlias<!strconcat(opc, "${s}${p} $Rdn, $imm"),
     (!cast<Instruction>(NAME#"ri") rGPR:$Rdn, rGPR:$Rdn,
                                                    t2_so_imm:$imm, pred:$p,
                                                    cc_out:$s)>;
  def : t2InstAlias<!strconcat(opc, "${s}${p}", wide, " $Rdn, $Rm"),
     (!cast<Instruction>(NAME#"rr") rGPR:$Rdn, rGPR:$Rdn,
                                                    rGPR:$Rm, pred:$p,
                                                    cc_out:$s)>;
  def : t2InstAlias<!strconcat(opc, "${s}${p}", wide, " $Rdn, $shift"),
     (!cast<Instruction>(NAME#"rs") rGPR:$Rdn, rGPR:$Rdn,
                                                    t2_so_reg:$shift, pred:$p,
                                                    cc_out:$s)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 794-810
```tablegen
/// T2I_bin_w_irs - Same as T2I_bin_irs except these operations need
//  the ".w" suffix to indicate that they are wide.
multiclass T2I_bin_w_irs<bits<4> opcod, string opc,
                     InstrItinClass iii, InstrItinClass iir, InstrItinClass iis,
                     SDPatternOperator opnode, bit Commutable = 0> :
    T2I_bin_irs<opcod, opc, iii, iir, iis, opnode, Commutable, ".w"> {
  // Assembler aliases w/ the ".w" suffix.
  def : t2InstAlias<!strconcat(opc, "${s}${p}.w", " $Rd, $Rn, $imm"),
     (!cast<Instruction>(NAME#"ri") rGPR:$Rd, rGPR:$Rn, t2_so_imm:$imm, pred:$p,
                                    cc_out:$s)>;
  // Assembler aliases w/o the ".w" suffix.
  def : t2InstAlias<!strconcat(opc, "${s}${p}", " $Rd, $Rn, $Rm"),
     (!cast<Instruction>(NAME#"rr") rGPR:$Rd, rGPR:$Rn, rGPR:$Rm, pred:$p,
                                    cc_out:$s)>;
  def : t2InstAlias<!strconcat(opc, "${s}${p}", " $Rd, $Rn, $shift"),
     (!cast<Instruction>(NAME#"rs") rGPR:$Rd, rGPR:$Rn, t2_so_reg:$shift,
                                    pred:$p, cc_out:$s)>;
```
- EN: Declares TableGen `multiclass T2I_bin_w_irs`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass T2I_bin_w_irs`，它是一个可复用模板，可展开为多个相关记录。

### Lines 812-822
```tablegen
  // and with the optional destination operand, too.
  def : t2InstAlias<!strconcat(opc, "${s}${p}.w", " $Rdn, $imm"),
     (!cast<Instruction>(NAME#"ri") rGPR:$Rdn, rGPR:$Rdn, t2_so_imm:$imm,
                                    pred:$p, cc_out:$s)>;
  def : t2InstAlias<!strconcat(opc, "${s}${p}", " $Rdn, $Rm"),
     (!cast<Instruction>(NAME#"rr") rGPR:$Rdn, rGPR:$Rdn, rGPR:$Rm, pred:$p,
                                    cc_out:$s)>;
  def : t2InstAlias<!strconcat(opc, "${s}${p}", " $Rdn, $shift"),
     (!cast<Instruction>(NAME#"rs") rGPR:$Rdn, rGPR:$Rdn, t2_so_reg:$shift,
                                    pred:$p, cc_out:$s)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 824-841
```tablegen
/// T2I_rbin_is - Same as T2I_bin_irs except the order of operands are
/// reversed.  The 'rr' form is only defined for the disassembler; for codegen
/// it is equivalent to the T2I_bin_irs counterpart.
multiclass T2I_rbin_irs<bits<4> opcod, string opc, SDNode opnode> {
   // shifted imm
   def ri : T2sTwoRegImm<
                 (outs rGPR:$Rd), (ins rGPR:$Rn, t2_so_imm:$imm), IIC_iALUi,
                 opc, ".w\t$Rd, $Rn, $imm",
                 [(set rGPR:$Rd, (opnode t2_so_imm:$imm, rGPR:$Rn))]>,
                 Sched<[WriteALU, ReadALU]> {
     let Inst{31-27} = 0b11110;
     let Inst{25} = 0;
     let Inst{24-21} = opcod;
     let Inst{15} = 0;
   }
   // register
   def rr : T2sThreeReg<
                 (outs rGPR:$Rd), (ins rGPR:$Rn, rGPR:$Rm), IIC_iALUr,
```
- EN: Declares TableGen `multiclass T2I_rbin_irs`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass T2I_rbin_irs`，它是一个可复用模板，可展开为多个相关记录。

### Lines 842-859
```tablegen
                 opc, "\t$Rd, $Rn, $Rm",
                 [/* For disassembly only; pattern left blank */]>,
                 Sched<[WriteALU, ReadALU, ReadALU]> {
     let Inst{31-27} = 0b11101;
     let Inst{26-25} = 0b01;
     let Inst{24-21} = opcod;
     let Inst{14-12} = 0b000; // imm3
     let Inst{7-6} = 0b00; // imm2
     let Inst{5-4} = 0b00; // type
     let hasSideEffects = 0;
   }
   // shifted register
   def rs : T2sTwoRegShiftedReg<
                 (outs rGPR:$Rd), (ins rGPR:$Rn, t2_so_reg:$ShiftedRm),
                 IIC_iALUsir, opc, "\t$Rd, $Rn, $ShiftedRm",
                 [(set rGPR:$Rd, (opnode t2_so_reg:$ShiftedRm, rGPR:$Rn))]>,
                 Sched<[WriteALUsi, ReadALU]> {
     let Inst{31-27} = 0b11101;
```
- EN: Defines TableGen record `rs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 860-863
```tablegen
     let Inst{26-25} = 0b01;
     let Inst{24-21} = opcod;
   }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 865-882
```tablegen
/// T2I_bin_s_irs - Similar to T2I_bin_irs except it sets the 's' bit so the
/// instruction modifies the CPSR register.
///
/// These opcodes will be converted to the real non-S opcodes by
/// AdjustInstrPostInstrSelection after giving then an optional CPSR operand.
let hasPostISelHook = 1, Defs = [CPSR] in {
multiclass T2I_bin_s_irs<InstrItinClass iii, InstrItinClass iir,
                         InstrItinClass iis, SDNode opnode,
                         bit Commutable = 0> {
   // shifted imm
   def ri : t2PseudoInst<(outs rGPR:$Rd),
                         (ins GPRnopc:$Rn, t2_so_imm:$imm, pred:$p),
                         4, iii,
                         [(set rGPR:$Rd, CPSR, (opnode GPRnopc:$Rn,
                                                t2_so_imm:$imm))]>,
            Sched<[WriteALU, ReadALU]>;
   // register
   def rr : t2PseudoInst<(outs rGPR:$Rd), (ins GPRnopc:$Rn, rGPR:$Rm, pred:$p),
```
- EN: Declares TableGen `multiclass T2I_bin_s_irs`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass T2I_bin_s_irs`，它是一个可复用模板，可展开为多个相关记录。

### Lines 883-897
```tablegen
                         4, iir,
                         [(set rGPR:$Rd, CPSR, (opnode GPRnopc:$Rn,
                                                rGPR:$Rm))]>,
            Sched<[WriteALU, ReadALU, ReadALU]> {
     let isCommutable = Commutable;
   }
   // shifted register
   def rs : t2PseudoInst<(outs rGPR:$Rd),
                         (ins GPRnopc:$Rn, t2_so_reg:$ShiftedRm, pred:$p),
                         4, iis,
                         [(set rGPR:$Rd, CPSR, (opnode GPRnopc:$Rn,
                                                t2_so_reg:$ShiftedRm))]>,
            Sched<[WriteALUsi, ReadALUsr]>;
}
}
```
- EN: Defines TableGen record `rs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 899-916
```tablegen
/// T2I_rbin_s_is -  Same as T2I_bin_s_irs, except selection DAG
/// operands are reversed.
let hasPostISelHook = 1, Defs = [CPSR] in {
multiclass T2I_rbin_s_is<SDNode opnode> {
   // shifted imm
   def ri : t2PseudoInst<(outs rGPR:$Rd),
                         (ins rGPR:$Rn, t2_so_imm:$imm, pred:$p),
                         4, IIC_iALUi,
                         [(set rGPR:$Rd, CPSR, (opnode t2_so_imm:$imm,
                                                rGPR:$Rn))]>,
            Sched<[WriteALU, ReadALU]>;
   // shifted register
   def rs : t2PseudoInst<(outs rGPR:$Rd),
                         (ins rGPR:$Rn, t2_so_reg:$ShiftedRm, pred:$p),
                         4, IIC_iALUsi,
                         [(set rGPR:$Rd, CPSR, (opnode t2_so_reg:$ShiftedRm,
                                                rGPR:$Rn))]>,
            Sched<[WriteALUsi, ReadALU]>;
```
- EN: Declares TableGen `multiclass T2I_rbin_s_is`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass T2I_rbin_s_is`，它是一个可复用模板，可展开为多个相关记录。

### Lines 917-918
```tablegen
}
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 920-934
```tablegen
/// T2I_bin_ii12rs - Defines a set of (op reg, {so_imm|imm0_4095|r|so_reg})
/// patterns for a binary operation that produces a value.
multiclass T2I_bin_ii12rs<bits<3> op23_21, string opc, SDNode opnode,
                          bit Commutable = 0> {
   // shifted imm
   // The register-immediate version is re-materializable. This is useful
   // in particular for taking the address of a local.
   let isReMaterializable = 1 in {
    def spImm : T2sTwoRegImm<
              (outs GPRsp:$Rd), (ins GPRsp:$Rn, t2_so_imm:$imm), IIC_iALUi,
              opc, ".w\t$Rd, $Rn, $imm",
              []>,
              Sched<[WriteALU, ReadALU]> {
    let  Rn = 13;
    let  Rd = 13;
```
- EN: Declares TableGen `multiclass T2I_bin_ii12rs`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass T2I_bin_ii12rs`，它是一个可复用模板，可展开为多个相关记录。

### Lines 936-939
```tablegen
    let Inst{31-27} = 0b11110;
    let Inst{25-24} = 0b01;
    let Inst{23-21} = op23_21;
    let Inst{15}    = 0;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 941-943
```tablegen
    let DecoderMethod = "DecodeT2AddSubSPImm";
    let hasSideEffects = 0;
   }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 945-962
```tablegen
   def ri : T2sTwoRegImm<
               (outs rGPR:$Rd), (ins GPRnopc:$Rn, t2_so_imm:$imm), IIC_iALUi,
               opc, ".w\t$Rd, $Rn, $imm",
               [(set rGPR:$Rd, (opnode GPRnopc:$Rn, t2_so_imm:$imm))]>,
               Sched<[WriteALU, ReadALU]> {
     let Inst{31-27} = 0b11110;
     let Inst{25} = 0;
     let Inst{24} = 1;
     let Inst{23-21} = op23_21;
     let Inst{15} = 0;
   }
   }
   // 12-bit imm
   def ri12 : T2I<
                  (outs rGPR:$Rd), (ins GPR:$Rn, imm0_4095:$imm), IIC_iALUi,
                  !strconcat(opc, "w"), "\t$Rd, $Rn, $imm",
                  [(set rGPR:$Rd, (opnode GPR:$Rn, imm0_4095:$imm))]>,
                  Sched<[WriteALU, ReadALU]> {
```
- EN: Defines TableGen record `ri` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ri`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 963-980
```tablegen
     bits<4> Rd;
     bits<4> Rn;
     bits<12> imm;
     let Inst{31-27} = 0b11110;
     let Inst{26} = imm{11};
     let Inst{25-24} = 0b10;
     let Inst{23-21} = op23_21;
     let Inst{20} = 0; // The S bit.
     let Inst{19-16} = Rn;
     let Inst{15} = 0;
     let Inst{14-12} = imm{10-8};
     let Inst{11-8} = Rd;
     let Inst{7-0} = imm{7-0};
   }
     def spImm12 : T2I<
                    (outs GPRsp:$Rd), (ins GPRsp:$Rn, imm0_4095:$imm), IIC_iALUi,
                    !strconcat(opc, "w"), "\t$Rd, $Rn, $imm",
                    []>,
```
- EN: Defines TableGen record `spImm12` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `spImm12`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 981-998
```tablegen
                    Sched<[WriteALU, ReadALU]> {
       bits<4> Rd = 13;
       bits<4> Rn = 13;
       bits<12> imm;
       let Inst{31-27} = 0b11110;
       let Inst{26} = imm{11};
       let Inst{25-24} = 0b10;
       let Inst{23-21} = op23_21;
       let Inst{20} = 0; // The S bit.
       let Inst{19-16} = Rn;
       let Inst{15} = 0;
       let Inst{14-12} = imm{10-8};
       let Inst{11-8} = Rd;
       let Inst{7-0} = imm{7-0};
       let DecoderMethod = "DecodeT2AddSubSPImm";
       let hasSideEffects = 0;
     }
   // register
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 999-1016
```tablegen
   def rr : T2sThreeReg<(outs GPRnopc:$Rd), (ins GPRnopc:$Rn, rGPR:$Rm),
                 IIC_iALUr, opc, ".w\t$Rd, $Rn, $Rm",
                 [(set GPRnopc:$Rd, (opnode GPRnopc:$Rn, rGPR:$Rm))]>,
                 Sched<[WriteALU, ReadALU, ReadALU]> {
     let isCommutable = Commutable;
     let Inst{31-27} = 0b11101;
     let Inst{26-25} = 0b01;
     let Inst{24} = 1;
     let Inst{23-21} = op23_21;
     let Inst{14-12} = 0b000; // imm3
     let Inst{7-6} = 0b00; // imm2
     let Inst{5-4} = 0b00; // type
   }
   // shifted register
   def rs : T2sTwoRegShiftedReg<
                 (outs GPRnopc:$Rd), (ins GPRnopc:$Rn, t2_so_reg:$ShiftedRm),
                 IIC_iALUsi, opc, ".w\t$Rd, $Rn, $ShiftedRm",
              [(set GPRnopc:$Rd, (opnode GPRnopc:$Rn, t2_so_reg:$ShiftedRm))]>,
```
- EN: Defines TableGen record `rr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1017-1023
```tablegen
              Sched<[WriteALUsi, ReadALU]> {
     let Inst{31-27} = 0b11101;
     let Inst{26-25} = 0b01;
     let Inst{24} = 1;
     let Inst{23-21} = op23_21;
   }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1025-1042
```tablegen
/// T2I_adde_sube_irs - Defines a set of (op reg, {so_imm|r|so_reg}) patterns
/// for a binary operation that produces a value and use the carry
/// bit. It's not predicable.
multiclass T2I_adde_sube_irs<bits<4> opcod, string opc, SDNode opnode,
                             bit Commutable = 0, bit PostISelHook = 0> {
  let Defs = [CPSR], Uses = [CPSR], hasPostISelHook = PostISelHook in {
   // shifted imm
   def ri : T2sTwoRegImm<(outs rGPR:$Rd), (ins rGPR:$Rn, t2_so_imm:$imm),
                 IIC_iALUi, opc, "\t$Rd, $Rn, $imm",
               [(set rGPR:$Rd, CPSR, (opnode rGPR:$Rn, t2_so_imm:$imm, CPSR))]>,
                 Requires<[IsThumb2]>, Sched<[WriteALU, ReadALU]> {
     let Inst{31-27} = 0b11110;
     let Inst{25} = 0;
     let Inst{24-21} = opcod;
     let Inst{15} = 0;
   }
   // register
   def rr : T2sThreeReg<(outs rGPR:$Rd), (ins rGPR:$Rn, rGPR:$Rm), IIC_iALUr,
```
- EN: Declares TableGen `multiclass T2I_adde_sube_irs`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass T2I_adde_sube_irs`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1043-1060
```tablegen
                 opc, ".w\t$Rd, $Rn, $Rm",
                 [(set rGPR:$Rd, CPSR, (opnode rGPR:$Rn, rGPR:$Rm, CPSR))]>,
                 Requires<[IsThumb2]>, Sched<[WriteALU, ReadALU, ReadALU]> {
     let isCommutable = Commutable;
     let Inst{31-27} = 0b11101;
     let Inst{26-25} = 0b01;
     let Inst{24-21} = opcod;
     let Inst{14-12} = 0b000; // imm3
     let Inst{7-6} = 0b00; // imm2
     let Inst{5-4} = 0b00; // type
   }
   // shifted register
   def rs : T2sTwoRegShiftedReg<
                 (outs rGPR:$Rd), (ins rGPR:$Rn, t2_so_reg:$ShiftedRm),
                 IIC_iALUsi, opc, ".w\t$Rd, $Rn, $ShiftedRm",
         [(set rGPR:$Rd, CPSR, (opnode rGPR:$Rn, t2_so_reg:$ShiftedRm, CPSR))]>,
                 Requires<[IsThumb2]>, Sched<[WriteALUsi, ReadALU]> {
     let Inst{31-27} = 0b11101;
```
- EN: Defines TableGen record `rs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1061-1078
```tablegen
     let Inst{26-25} = 0b01;
     let Inst{24-21} = opcod;
   }
  }
  // Shortened forms
  def : t2InstAlias<!strconcat(opc, "${s}${p}", ".w $Rdn, $imm"),
     (!cast<Instruction>(NAME#"ri") rGPR:$Rdn, rGPR:$Rdn, t2_so_imm:$imm, pred:$p,
                                    cc_out:$s)>;
  def : t2InstAlias<!strconcat(opc, "${s}${p}", ".w $Rdn, $Rm"),
     (!cast<Instruction>(NAME#"rr") rGPR:$Rdn, rGPR:$Rdn, rGPR:$Rm, pred:$p,
                                    cc_out:$s)>;
  def : t2InstAlias<!strconcat(opc, "${s}${p}", ".w $Rdn, $ShiftedRm"),
     (!cast<Instruction>(NAME#"rs") rGPR:$Rdn, rGPR:$Rdn, t2_so_reg:$ShiftedRm, pred:$p,
                                    cc_out:$s)>;
  def : t2InstAlias<!strconcat(opc, "${s}${p}", "$Rdn, $imm"),
     (!cast<Instruction>(NAME#"ri") rGPR:$Rdn, rGPR:$Rdn, t2_so_imm:$imm, pred:$p,
                                    cc_out:$s)>;
  def : t2InstAlias<!strconcat(opc, "${s}${p}", "$Rdn, $Rm"),
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1079-1084
```tablegen
     (!cast<Instruction>(NAME#"rr") rGPR:$Rdn, rGPR:$Rdn, rGPR:$Rm, pred:$p,
                                    cc_out:$s)>;
  def : t2InstAlias<!strconcat(opc, "${s}${p}", "$Rdn, $ShiftedRm"),
     (!cast<Instruction>(NAME#"rs") rGPR:$Rdn, rGPR:$Rdn, t2_so_reg:$ShiftedRm, pred:$p,
                                    cc_out:$s)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1086-1103
```tablegen
/// T2I_sh_ir - Defines a set of (op reg, {so_imm|r}) patterns for a shift /
//  rotate operation that produces a value.
multiclass T2I_sh_ir<bits<2> opcod, string opc, Operand ty, SDNode opnode> {
   // 5-bit imm
   def ri : T2sTwoRegShiftImm<
                 (outs rGPR:$Rd), (ins rGPR:$Rm, ty:$imm), IIC_iMOVsi,
                 opc, ".w\t$Rd, $Rm, $imm",
                 [(set rGPR:$Rd, (opnode rGPR:$Rm, (i32 ty:$imm)))]>,
                 Sched<[WriteALU]> {
     let Inst{31-27} = 0b11101;
     let Inst{26-21} = 0b010010;
     let Inst{19-16} = 0b1111; // Rn
     let Inst{15}    = 0b0;
     let Inst{5-4} = opcod;
   }
   // register
   def rr : T2sThreeReg<
                 (outs rGPR:$Rd), (ins rGPR:$Rn, rGPR:$Rm), IIC_iMOVsr,
```
- EN: Declares TableGen `multiclass T2I_sh_ir`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass T2I_sh_ir`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1104-1112
```tablegen
                 opc, ".w\t$Rd, $Rn, $Rm",
                 [(set rGPR:$Rd, (opnode rGPR:$Rn, rGPR:$Rm))]>,
                 Sched<[WriteALU]> {
     let Inst{31-27} = 0b11111;
     let Inst{26-23} = 0b0100;
     let Inst{22-21} = opcod;
     let Inst{15-12} = 0b1111;
     let Inst{7-4} = 0b0000;
   }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1114-1120
```tablegen
  // Optional destination register
  def : t2InstAlias<!strconcat(opc, "${s}${p}", ".w $Rdn, $imm"),
     (!cast<Instruction>(NAME#"ri") rGPR:$Rdn, rGPR:$Rdn, ty:$imm, pred:$p,
                                    cc_out:$s)>;
  def : t2InstAlias<!strconcat(opc, "${s}${p}", ".w $Rdn, $Rm"),
     (!cast<Instruction>(NAME#"rr") rGPR:$Rdn, rGPR:$Rdn, rGPR:$Rm, pred:$p,
                                    cc_out:$s)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1122-1128
```tablegen
  // Assembler aliases w/o the ".w" suffix.
  def : t2InstAlias<!strconcat(opc, "${s}${p}", " $Rd, $Rn, $imm"),
     (!cast<Instruction>(NAME#"ri") rGPR:$Rd, rGPR:$Rn, ty:$imm, pred:$p,
                                    cc_out:$s)>;
  def : t2InstAlias<!strconcat(opc, "${s}${p}", " $Rd, $Rn, $Rm"),
     (!cast<Instruction>(NAME#"rr") rGPR:$Rd, rGPR:$Rn, rGPR:$Rm, pred:$p,
                                    cc_out:$s)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1130-1137
```tablegen
  // and with the optional destination operand, too.
  def : t2InstAlias<!strconcat(opc, "${s}${p}", " $Rdn, $imm"),
     (!cast<Instruction>(NAME#"ri") rGPR:$Rdn, rGPR:$Rdn, ty:$imm, pred:$p,
                                    cc_out:$s)>;
  def : t2InstAlias<!strconcat(opc, "${s}${p}", " $Rdn, $Rm"),
     (!cast<Instruction>(NAME#"rr") rGPR:$Rdn, rGPR:$Rdn, rGPR:$Rm, pred:$p,
                                    cc_out:$s)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1139-1156
```tablegen
/// T2I_cmp_irs - Defines a set of (op r, {so_imm|r|so_reg}) cmp / test
/// patterns. Similar to T2I_bin_irs except the instruction does not produce
/// a explicit result, only implicitly set CPSR.
multiclass T2I_cmp_irs<bits<4> opcod, string opc, RegisterClass LHSGPR,
                     InstrItinClass iii, InstrItinClass iir, InstrItinClass iis,
                     SDPatternOperator opnode> {
let isCompare = 1, Defs = [CPSR] in {
   // shifted imm
   def ri : T2OneRegCmpImm<
                (outs), (ins LHSGPR:$Rn, t2_so_imm:$imm), iii,
                opc, ".w\t$Rn, $imm",
                [(set CPSR, (opnode LHSGPR:$Rn, t2_so_imm:$imm))]>,
            Sched<[WriteCMP]> {
     let Inst{31-27} = 0b11110;
     let Inst{25} = 0;
     let Inst{24-21} = opcod;
     let Inst{20} = 1; // The S bit.
     let Inst{15} = 0;
```
- EN: Declares TableGen `multiclass T2I_cmp_irs`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass T2I_cmp_irs`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1157-1174
```tablegen
     let Inst{11-8} = 0b1111; // Rd
   }
   // register
   def rr : T2TwoRegCmp<
                (outs), (ins LHSGPR:$Rn, rGPR:$Rm), iir,
                opc, ".w\t$Rn, $Rm",
                [(set CPSR, (opnode LHSGPR:$Rn, rGPR:$Rm))]>,
            Sched<[WriteCMP]> {
     let Inst{31-27} = 0b11101;
     let Inst{26-25} = 0b01;
     let Inst{24-21} = opcod;
     let Inst{20} = 1; // The S bit.
     let Inst{14-12} = 0b000; // imm3
     let Inst{11-8} = 0b1111; // Rd
     let Inst{7-6} = 0b00; // imm2
     let Inst{5-4} = 0b00; // type
   }
   // shifted register
```
- EN: Defines TableGen record `rr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1175-1186
```tablegen
   def rs : T2OneRegCmpShiftedReg<
                (outs), (ins LHSGPR:$Rn, t2_so_reg:$ShiftedRm), iis,
                opc, ".w\t$Rn, $ShiftedRm",
                [(set CPSR, (opnode LHSGPR:$Rn, t2_so_reg:$ShiftedRm))]>,
                Sched<[WriteCMPsi]> {
     let Inst{31-27} = 0b11101;
     let Inst{26-25} = 0b01;
     let Inst{24-21} = opcod;
     let Inst{20} = 1; // The S bit.
     let Inst{11-8} = 0b1111; // Rd
   }
}
```
- EN: Defines TableGen record `rs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1188-1195
```tablegen
  // Assembler aliases w/o the ".w" suffix.
  // No alias here for 'rr' version as not all instantiations of this
  // multiclass want one (CMP in particular, does not).
  def : t2InstAlias<!strconcat(opc, "${p}", " $Rn, $imm"),
     (!cast<Instruction>(NAME#"ri") LHSGPR:$Rn, t2_so_imm:$imm, pred:$p)>;
  def : t2InstAlias<!strconcat(opc, "${p}", " $Rn, $shift"),
     (!cast<Instruction>(NAME#"rs") LHSGPR:$Rn, t2_so_reg:$shift, pred:$p)>;
}
```
- EN: Declares TableGen `multiclass want`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass want`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1197-1214
```tablegen
/// T2I_ld - Defines a set of (op r, {imm12|imm8|so_reg}) load patterns.
multiclass T2I_ld<bit signed, bits<2> opcod, string opc,
                  InstrItinClass iii, InstrItinClass iis, RegisterClass target,
                  PatFrag opnode> {
  def i12 : T2Ii12<(outs target:$Rt), (ins t2addrmode_imm12:$addr), iii,
                   opc, ".w\t$Rt, $addr",
                   [(set target:$Rt, (opnode t2addrmode_imm12:$addr))]>,
            Sched<[WriteLd]> {
    bits<4> Rt;
    bits<17> addr;
    let Inst{31-25} = 0b1111100;
    let Inst{24} = signed;
    let Inst{23} = 1;
    let Inst{22-21} = opcod;
    let Inst{20} = 1; // load
    let Inst{19-16} = addr{16-13}; // Rn
    let Inst{15-12} = Rt;
    let Inst{11-0}  = addr{11-0};  // imm
```
- EN: Declares TableGen `multiclass T2I_ld`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass T2I_ld`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1216-1233
```tablegen
    let DecoderMethod = "DecodeT2LoadImm12";
  }
  def i8  : T2Ii8n <(outs target:$Rt), (ins t2addrmode_negimm8:$addr), iii,
                    opc, "\t$Rt, $addr",
                    [(set target:$Rt, (opnode t2addrmode_negimm8:$addr))]>,
            Sched<[WriteLd]> {
    bits<4> Rt;
    bits<13> addr;
    let Inst{31-27} = 0b11111;
    let Inst{26-25} = 0b00;
    let Inst{24} = signed;
    let Inst{23} = 0;
    let Inst{22-21} = opcod;
    let Inst{20} = 1; // load
    let Inst{19-16} = addr{12-9}; // Rn
    let Inst{15-12} = Rt;
    let Inst{11} = 1;
    // Offset: index==TRUE, wback==FALSE
```
- EN: Defines TableGen record `i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1234-1237
```tablegen
    let Inst{10} = 1; // The P bit.
    let Inst{9}     = addr{8};    // U
    let Inst{8} = 0; // The W bit.
    let Inst{7-0}   = addr{7-0};  // imm
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1239-1251
```tablegen
    let DecoderMethod = "DecodeT2LoadImm8";
  }
  def s   : T2Iso <(outs target:$Rt), (ins t2addrmode_so_reg:$addr), iis,
                   opc, ".w\t$Rt, $addr",
                   [(set target:$Rt, (opnode t2addrmode_so_reg:$addr))]>,
            Sched<[WriteLd]> {
    let Inst{31-27} = 0b11111;
    let Inst{26-25} = 0b00;
    let Inst{24} = signed;
    let Inst{23} = 0;
    let Inst{22-21} = opcod;
    let Inst{20} = 1; // load
    let Inst{11-6} = 0b000000;
```
- EN: Defines TableGen record `s` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `s`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1253-1254
```tablegen
    bits<4> Rt;
    let Inst{15-12} = Rt;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1256-1259
```tablegen
    bits<10> addr;
    let Inst{19-16} = addr{9-6}; // Rn
    let Inst{3-0}   = addr{5-2}; // Rm
    let Inst{5-4}   = addr{1-0}; // imm
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1261-1262
```tablegen
    let DecoderMethod = "DecodeT2LoadShift";
  }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1264-1276
```tablegen
  // pci variant is very similar to i12, but supports negative offsets
  // from the PC.
  def pci : T2Ipc <(outs target:$Rt), (ins t2ldrlabel:$addr), iii,
                   opc, ".w\t$Rt, $addr",
                   [(set target:$Rt, (opnode (ARMWrapper tconstpool:$addr)))]>,
            Sched<[WriteLd]> {
    let isReMaterializable = 1;
    let Inst{31-27} = 0b11111;
    let Inst{26-25} = 0b00;
    let Inst{24} = signed;
    let Inst{22-21} = opcod;
    let Inst{20} = 1; // load
    let Inst{19-16} = 0b1111; // Rn
```
- EN: Defines TableGen record `pci` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `pci`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1278-1279
```tablegen
    bits<4> Rt;
    let Inst{15-12} = Rt{3-0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1281-1283
```tablegen
    bits<13> addr;
    let Inst{23} = addr{12}; // add = (U == '1')
    let Inst{11-0}  = addr{11-0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1285-1287
```tablegen
    let DecoderMethod = "DecodeT2LoadLabel";
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1289-1300
```tablegen
/// T2I_st - Defines a set of (op r, {imm12|imm8|so_reg}) store patterns.
multiclass T2I_st<bits<2> opcod, string opc,
                  InstrItinClass iii, InstrItinClass iis, RegisterClass target,
                  PatFrag opnode> {
  def i12 : T2Ii12<(outs), (ins target:$Rt, t2addrmode_imm12:$addr), iii,
                   opc, ".w\t$Rt, $addr",
                   [(opnode target:$Rt, t2addrmode_imm12:$addr)]>,
            Sched<[WriteST]> {
    let Inst{31-27} = 0b11111;
    let Inst{26-23} = 0b0001;
    let Inst{22-21} = opcod;
    let Inst{20} = 0; // !load
```
- EN: Declares TableGen `multiclass T2I_st`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass T2I_st`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1302-1303
```tablegen
    bits<4> Rt;
    let Inst{15-12} = Rt;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1305-1322
```tablegen
    bits<17> addr;
    let addr{12}    = 1;           // add = TRUE
    let Inst{19-16} = addr{16-13}; // Rn
    let Inst{23}    = addr{12};    // U
    let Inst{11-0}  = addr{11-0};  // imm
  }
  def i8  : T2Ii8n <(outs), (ins target:$Rt, t2addrmode_negimm8:$addr), iii,
                    opc, "\t$Rt, $addr",
                    [(opnode target:$Rt, t2addrmode_negimm8:$addr)]>,
            Sched<[WriteST]> {
    let Inst{31-27} = 0b11111;
    let Inst{26-23} = 0b0000;
    let Inst{22-21} = opcod;
    let Inst{20} = 0; // !load
    let Inst{11} = 1;
    // Offset: index==TRUE, wback==FALSE
    let Inst{10} = 1; // The P bit.
    let Inst{8} = 0; // The W bit.
```
- EN: Defines TableGen record `i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1324-1325
```tablegen
    bits<4> Rt;
    let Inst{15-12} = Rt;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1327-1340
```tablegen
    bits<13> addr;
    let Inst{19-16} = addr{12-9}; // Rn
    let Inst{9}     = addr{8};    // U
    let Inst{7-0}   = addr{7-0};  // imm
  }
  def s   : T2Iso <(outs), (ins target:$Rt, t2addrmode_so_reg:$addr), iis,
                   opc, ".w\t$Rt, $addr",
                   [(opnode target:$Rt, t2addrmode_so_reg:$addr)]>,
            Sched<[WriteST]> {
    let Inst{31-27} = 0b11111;
    let Inst{26-23} = 0b0000;
    let Inst{22-21} = opcod;
    let Inst{20} = 0; // !load
    let Inst{11-6} = 0b000000;
```
- EN: Defines TableGen record `s` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `s`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1342-1343
```tablegen
    bits<4> Rt;
    let Inst{15-12} = Rt;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1345-1350
```tablegen
    bits<10> addr;
    let Inst{19-16}   = addr{9-6}; // Rn
    let Inst{3-0} = addr{5-2}; // Rm
    let Inst{5-4}   = addr{1-0}; // imm
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1352-1366
```tablegen
/// T2I_ext_rrot - A unary operation with two forms: one whose operand is a
/// register and one whose operand is a register rotated by 8/16/24.
class T2I_ext_rrot_base<bits<3> opcod, dag iops, dag oops,
                        string opc, string oprs,
                        list<dag> pattern>
  : T2TwoReg<iops, oops, IIC_iEXTr, opc, oprs, pattern> {
  bits<2> rot;
  let Inst{31-27} = 0b11111;
  let Inst{26-23} = 0b0100;
  let Inst{22-20} = opcod;
  let Inst{19-16} = 0b1111; // Rn
  let Inst{15-12} = 0b1111;
  let Inst{7} = 1;
  let Inst{5-4} = rot; // rotate
}
```
- EN: Declares reusable TableGen class `T2I_ext_rrot_base` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2I_ext_rrot_base`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1368-1374
```tablegen
class T2I_ext_rrot<bits<3> opcod, string opc>
  : T2I_ext_rrot_base<opcod,
                      (outs rGPR:$Rd),
                      (ins rGPR:$Rm, rot_imm:$rot),
                      opc, ".w\t$Rd, $Rm$rot", []>,
                      Requires<[IsThumb2]>,
                      Sched<[WriteALU, ReadALU]>;
```
- EN: Declares reusable TableGen class `T2I_ext_rrot` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2I_ext_rrot`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1376-1383
```tablegen
// UXTB16, SXTB16 - Requires HasDSP, does not need the .w qualifier.
class T2I_ext_rrot_xtb16<bits<3> opcod, string opc>
  : T2I_ext_rrot_base<opcod,
                      (outs rGPR:$Rd),
                      (ins rGPR:$Rm, rot_imm:$rot),
                      opc, "\t$Rd, $Rm$rot", []>,
                      Requires<[HasDSP, IsThumb2]>,
                      Sched<[WriteALU, ReadALU]>;
```
- EN: Declares reusable TableGen class `T2I_ext_rrot_xtb16` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2I_ext_rrot_xtb16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1385-1400
```tablegen
/// T2I_exta_rrot - A binary operation with two forms: one whose operand is a
/// register and one whose operand is a register rotated by 8/16/24.
class T2I_exta_rrot<bits<3> opcod, string opc>
  : T2ThreeReg<(outs rGPR:$Rd),
               (ins rGPR:$Rn, rGPR:$Rm, rot_imm:$rot),
               IIC_iEXTAsr, opc, "\t$Rd, $Rn, $Rm$rot", []>,
               Requires<[HasDSP, IsThumb2]>,
               Sched<[WriteALU, ReadALU]> {
  bits<2> rot;
  let Inst{31-27} = 0b11111;
  let Inst{26-23} = 0b0100;
  let Inst{22-20} = opcod;
  let Inst{15-12} = 0b1111;
  let Inst{7} = 1;
  let Inst{5-4} = rot;
}
```
- EN: Declares reusable TableGen class `T2I_exta_rrot` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2I_exta_rrot`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1402-1404
```tablegen
//===----------------------------------------------------------------------===//
// Instructions
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 1406-1408
```tablegen
//===----------------------------------------------------------------------===//
//  Miscellaneous Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 1410-1414
```tablegen
class T2PCOneRegImm<dag oops, dag iops, InstrItinClass itin,
           string asm, list<dag> pattern>
  : T2XI<oops, iops, itin, asm, pattern> {
  bits<4> Rd;
  bits<12> label;
```
- EN: Declares reusable TableGen class `T2PCOneRegImm` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2PCOneRegImm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1416-1420
```tablegen
  let Inst{11-8}  = Rd;
  let Inst{26}    = label{11};
  let Inst{14-12} = label{10-8};
  let Inst{7-0}   = label{7-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1422-1434
```tablegen
// LEApcrel - Load a pc-relative address into a register without offending the
// assembler.
def t2ADR : T2PCOneRegImm<(outs rGPR:$Rd),
              (ins t2adrlabel:$addr, pred:$p),
              IIC_iALUi, "adr{$p}.w\t$Rd, $addr", []>,
              Sched<[WriteALU, ReadALU]> {
  let Inst{31-27} = 0b11110;
  let Inst{25-24} = 0b10;
  // Inst{23:21} = '11' (add = FALSE) or '00' (add = TRUE)
  let Inst{22} = 0;
  let Inst{20} = 0;
  let Inst{19-16} = 0b1111; // Rn
  let Inst{15} = 0;
```
- EN: Defines TableGen record `t2ADR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2ADR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1436-1443
```tablegen
  bits<4> Rd;
  bits<13> addr;
  let Inst{11-8} = Rd;
  let Inst{23}    = addr{12};
  let Inst{21}    = addr{12};
  let Inst{26}    = addr{11};
  let Inst{14-12} = addr{10-8};
  let Inst{7-0}   = addr{7-0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1445-1447
```tablegen
  let DecoderMethod = "DecodeT2Adr";
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1449-1456
```tablegen
let hasSideEffects = 0, isReMaterializable = 1 in
def t2LEApcrel   : t2PseudoInst<(outs rGPR:$Rd), (ins i32imm:$label, pred:$p),
                                4, IIC_iALUi, []>, Sched<[WriteALU, ReadALU]>;
let hasSideEffects = 1 in
def t2LEApcrelJT : t2PseudoInst<(outs rGPR:$Rd),
                                (ins i32imm:$label, pred:$p),
                                4, IIC_iALUi,
                                []>, Sched<[WriteALU, ReadALU]>;
```
- EN: Defines TableGen record `t2LEApcrel` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LEApcrel`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1459-1461
```tablegen
//===----------------------------------------------------------------------===//
//  Load / store Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 1463-1465
```tablegen
// Load
let canFoldAsLoad = 1, isReMaterializable = 1  in
defm t2LDR   : T2I_ld<0, 0b10, "ldr", IIC_iLoad_i, IIC_iLoad_si, GPR, load>;
```
- EN: Defines TableGen record `t2LDR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1467-1471
```tablegen
// Loads with zero extension
defm t2LDRH  : T2I_ld<0, 0b01, "ldrh", IIC_iLoad_bh_i, IIC_iLoad_bh_si,
                      GPRnopc, zextloadi16>;
defm t2LDRB  : T2I_ld<0, 0b00, "ldrb", IIC_iLoad_bh_i, IIC_iLoad_bh_si,
                      GPRnopc, zextloadi8>;
```
- EN: Defines TableGen record `t2LDRH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1473-1477
```tablegen
// Loads with sign extension
defm t2LDRSH : T2I_ld<1, 0b01, "ldrsh", IIC_iLoad_bh_i, IIC_iLoad_bh_si,
                      GPRnopc, sextloadi16>;
defm t2LDRSB : T2I_ld<1, 0b00, "ldrsb", IIC_iLoad_bh_i, IIC_iLoad_bh_si,
                      GPRnopc, sextloadi8>;
```
- EN: Defines TableGen record `t2LDRSH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRSH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1479-1486
```tablegen
let mayLoad = 1, hasSideEffects = 0, hasExtraDefRegAllocReq = 1 in {
// Load doubleword
def t2LDRDi8  : T2Ii8s4<1, 0, 1, (outs rGPR:$Rt, rGPR:$Rt2),
                        (ins t2addrmode_imm8s4:$addr),
                        IIC_iLoad_d_i, "ldrd", "\t$Rt, $Rt2, $addr", "",
                        [(set rGPR:$Rt, rGPR:$Rt2, (ARMldrd t2addrmode_imm8s4:$addr))]>,
                 Sched<[WriteLd]>;
} // mayLoad = 1, hasSideEffects = 0, hasExtraDefRegAllocReq = 1
```
- EN: Defines TableGen record `t2LDRDi8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRDi8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1488-1496
```tablegen
// zextload i1 -> zextload i8
def : T2Pat<(zextloadi1 t2addrmode_imm12:$addr),
            (t2LDRBi12  t2addrmode_imm12:$addr)>;
def : T2Pat<(zextloadi1 t2addrmode_negimm8:$addr),
            (t2LDRBi8   t2addrmode_negimm8:$addr)>;
def : T2Pat<(zextloadi1 t2addrmode_so_reg:$addr),
            (t2LDRBs    t2addrmode_so_reg:$addr)>;
def : T2Pat<(zextloadi1 (ARMWrapper tconstpool:$addr)),
            (t2LDRBpci  tconstpool:$addr)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1498-1508
```tablegen
// extload -> zextload
// FIXME: Reduce the number of patterns by legalizing extload to zextload
// earlier?
def : T2Pat<(extloadi1  t2addrmode_imm12:$addr),
            (t2LDRBi12  t2addrmode_imm12:$addr)>;
def : T2Pat<(extloadi1  t2addrmode_negimm8:$addr),
            (t2LDRBi8   t2addrmode_negimm8:$addr)>;
def : T2Pat<(extloadi1  t2addrmode_so_reg:$addr),
            (t2LDRBs    t2addrmode_so_reg:$addr)>;
def : T2Pat<(extloadi1  (ARMWrapper tconstpool:$addr)),
            (t2LDRBpci  tconstpool:$addr)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1510-1517
```tablegen
def : T2Pat<(extloadi8  t2addrmode_imm12:$addr),
            (t2LDRBi12  t2addrmode_imm12:$addr)>;
def : T2Pat<(extloadi8  t2addrmode_negimm8:$addr),
            (t2LDRBi8   t2addrmode_negimm8:$addr)>;
def : T2Pat<(extloadi8  t2addrmode_so_reg:$addr),
            (t2LDRBs    t2addrmode_so_reg:$addr)>;
def : T2Pat<(extloadi8  (ARMWrapper tconstpool:$addr)),
            (t2LDRBpci  tconstpool:$addr)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1519-1526
```tablegen
def : T2Pat<(extloadi16 t2addrmode_imm12:$addr),
            (t2LDRHi12  t2addrmode_imm12:$addr)>;
def : T2Pat<(extloadi16 t2addrmode_negimm8:$addr),
            (t2LDRHi8   t2addrmode_negimm8:$addr)>;
def : T2Pat<(extloadi16 t2addrmode_so_reg:$addr),
            (t2LDRHs    t2addrmode_so_reg:$addr)>;
def : T2Pat<(extloadi16 (ARMWrapper tconstpool:$addr)),
            (t2LDRHpci  tconstpool:$addr)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1528-1531
```tablegen
// FIXME: The destination register of the loads and stores can't be PC, but
//        can be SP. We need another regclass (similar to rGPR) to represent
//        that. Not a pressing issue since these are selected manually,
//        not via pattern.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1533-1533
```tablegen
// Indexed loads
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1535-1540
```tablegen
let mayLoad = 1, hasSideEffects = 0 in {
def t2LDR_PRE  : T2Ipreldst<0, 0b10, 1, 1, (outs GPR:$Rt, GPR:$Rn_wb),
                            (ins t2addrmode_imm8_pre:$addr),
                            AddrModeT2_i8, IndexModePre, IIC_iLoad_iu,
                            "ldr", "\t$Rt, $addr!", "$addr.base = $Rn_wb", []>,
                 Sched<[WriteLd]>;
```
- EN: Defines TableGen record `t2LDR_PRE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDR_PRE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1542-1546
```tablegen
def t2LDR_POST : T2Ipostldst<0, 0b10, 1, 0, (outs GPR:$Rt, GPR:$Rn_wb),
                          (ins addr_offset_none:$Rn, t2am_imm8_offset:$offset),
                          AddrModeT2_i8, IndexModePost, IIC_iLoad_iu,
                          "ldr", "\t$Rt, $Rn$offset", "$Rn = $Rn_wb", []>,
                  Sched<[WriteLd]>;
```
- EN: Defines TableGen record `t2LDR_POST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDR_POST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1548-1552
```tablegen
def t2LDRB_PRE : T2Ipreldst<0, 0b00, 1, 1, (outs GPR:$Rt, GPR:$Rn_wb),
                            (ins t2addrmode_imm8_pre:$addr),
                            AddrModeT2_i8, IndexModePre, IIC_iLoad_bh_iu,
                            "ldrb", "\t$Rt, $addr!", "$addr.base = $Rn_wb", []>,
                 Sched<[WriteLd]>;
```
- EN: Defines TableGen record `t2LDRB_PRE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRB_PRE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1554-1558
```tablegen
def t2LDRB_POST : T2Ipostldst<0, 0b00, 1, 0, (outs GPR:$Rt, GPR:$Rn_wb),
                          (ins addr_offset_none:$Rn, t2am_imm8_offset:$offset),
                          AddrModeT2_i8, IndexModePost, IIC_iLoad_bh_iu,
                          "ldrb", "\t$Rt, $Rn$offset", "$Rn = $Rn_wb", []>,
                  Sched<[WriteLd]>;
```
- EN: Defines TableGen record `t2LDRB_POST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRB_POST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1560-1564
```tablegen
def t2LDRH_PRE : T2Ipreldst<0, 0b01, 1, 1, (outs GPR:$Rt, GPR:$Rn_wb),
                            (ins t2addrmode_imm8_pre:$addr),
                            AddrModeT2_i8, IndexModePre, IIC_iLoad_bh_iu,
                            "ldrh", "\t$Rt, $addr!", "$addr.base = $Rn_wb", []>,
                Sched<[WriteLd]>;
```
- EN: Defines TableGen record `t2LDRH_PRE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRH_PRE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1566-1570
```tablegen
def t2LDRH_POST : T2Ipostldst<0, 0b01, 1, 0, (outs GPR:$Rt, GPR:$Rn_wb),
                          (ins addr_offset_none:$Rn, t2am_imm8_offset:$offset),
                          AddrModeT2_i8, IndexModePost, IIC_iLoad_bh_iu,
                          "ldrh", "\t$Rt, $Rn$offset", "$Rn = $Rn_wb", []>,
                  Sched<[WriteLd]>;
```
- EN: Defines TableGen record `t2LDRH_POST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRH_POST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1572-1576
```tablegen
def t2LDRSB_PRE : T2Ipreldst<1, 0b00, 1, 1, (outs GPR:$Rt, GPR:$Rn_wb),
                            (ins t2addrmode_imm8_pre:$addr),
                            AddrModeT2_i8, IndexModePre, IIC_iLoad_bh_iu,
                            "ldrsb", "\t$Rt, $addr!", "$addr.base = $Rn_wb",
                            []>, Sched<[WriteLd]>;
```
- EN: Defines TableGen record `t2LDRSB_PRE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRSB_PRE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1578-1582
```tablegen
def t2LDRSB_POST : T2Ipostldst<1, 0b00, 1, 0, (outs GPR:$Rt, GPR:$Rn_wb),
                          (ins addr_offset_none:$Rn, t2am_imm8_offset:$offset),
                          AddrModeT2_i8, IndexModePost, IIC_iLoad_bh_iu,
                          "ldrsb", "\t$Rt, $Rn$offset", "$Rn = $Rn_wb", []>,
                   Sched<[WriteLd]>;
```
- EN: Defines TableGen record `t2LDRSB_POST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRSB_POST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1584-1588
```tablegen
def t2LDRSH_PRE : T2Ipreldst<1, 0b01, 1, 1, (outs GPR:$Rt, GPR:$Rn_wb),
                            (ins t2addrmode_imm8_pre:$addr),
                            AddrModeT2_i8, IndexModePre, IIC_iLoad_bh_iu,
                            "ldrsh", "\t$Rt, $addr!", "$addr.base = $Rn_wb",
                            []>, Sched<[WriteLd]>;
```
- EN: Defines TableGen record `t2LDRSH_PRE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRSH_PRE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1590-1595
```tablegen
def t2LDRSH_POST : T2Ipostldst<1, 0b01, 1, 0, (outs GPR:$Rt, GPR:$Rn_wb),
                          (ins addr_offset_none:$Rn, t2am_imm8_offset:$offset),
                          AddrModeT2_i8, IndexModePost, IIC_iLoad_bh_iu,
                          "ldrsh", "\t$Rt, $Rn$offset", "$Rn = $Rn_wb", []>,
                  Sched<[WriteLd]>;
} // mayLoad = 1, hasSideEffects = 0
```
- EN: Defines TableGen record `t2LDRSH_POST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRSH_POST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1597-1603
```tablegen
// F5.1.72 LDR (immediate) T4
// .w suffixes; Constraints can't be used on t2InstAlias to describe
// "$Rn =  $Rn_wb" on POST or "$addr.base = $Rn_wb" on PRE.
def t2LDR_PRE_imm : t2AsmPseudo<"ldr${p}.w $Rt, $addr!",
                         (ins GPR:$Rt, t2addrmode_imm8_pre:$addr, pred:$p)>;
def t2LDR_POST_imm : t2AsmPseudo<"ldr${p}.w $Rt, $Rn, $imm",
                         (ins GPR:$Rt, addr_offset_none:$Rn, t2am_imm8_offset:$imm, pred:$p)>;
```
- EN: Defines TableGen record `t2LDR_PRE_imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDR_PRE_imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1605-1613
```tablegen
// A7.7.46 LDRB (immediate) T3
// .w suffixes; Constraints can't be used on t2InstAlias to describe
// "$Rn =  $Rn_wb" on POST or "$addr.base = $Rn_wb" on PRE.
def t2LDRB_OFFSET_imm : t2AsmPseudo<"ldrb${p}.w $Rt, $addr",
                         (ins GPR:$Rt, t2addrmode_negimm8:$addr, pred:$p)>;
def t2LDRB_PRE_imm : t2AsmPseudo<"ldrb${p}.w $Rt, $addr!",
                         (ins GPR:$Rt, t2addrmode_imm8_pre:$addr, pred:$p)>;
def t2LDRB_POST_imm : t2AsmPseudo<"ldrb${p}.w $Rt, $Rn, $imm",
                         (ins GPR:$Rt, addr_offset_none:$Rn, t2am_imm8_offset:$imm, pred:$p)>;
```
- EN: Defines TableGen record `t2LDRB_OFFSET_imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRB_OFFSET_imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1615-1623
```tablegen
// A7.7.55 LDRH (immediate) T3
// .w suffixes; Constraints can't be used on t2InstAlias to describe
// "$Rn =  $Rn_wb" on POST or "$addr.base = $Rn_wb" on PRE.
def t2LDRH_OFFSET_imm : t2AsmPseudo<"ldrh${p}.w $Rt, $addr",
                         (ins GPR:$Rt, t2addrmode_negimm8:$addr, pred:$p)>;
def t2LDRH_PRE_imm : t2AsmPseudo<"ldrh${p}.w $Rt, $addr!",
                         (ins GPR:$Rt, t2addrmode_imm8_pre:$addr, pred:$p)>;
def t2LDRH_POST_imm : t2AsmPseudo<"ldrh${p}.w $Rt, $Rn, $imm",
                         (ins GPR:$Rt, addr_offset_none:$Rn, t2am_imm8_offset:$imm, pred:$p)>;
```
- EN: Defines TableGen record `t2LDRH_OFFSET_imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRH_OFFSET_imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1625-1633
```tablegen
// A7.7.59 LDRSB (immediate) T2
// .w suffixes; Constraints can't be used on t2InstAlias to describe
// "$Rn =  $Rn_wb" on POST or "$addr.base = $Rn_wb" on PRE.
def t2LDRSB_OFFSET_imm : t2AsmPseudo<"ldrsb${p}.w $Rt, $addr",
                         (ins GPR:$Rt, t2addrmode_negimm8:$addr, pred:$p)>;
def t2LDRSB_PRE_imm : t2AsmPseudo<"ldrsb${p}.w $Rt, $addr!",
                         (ins GPR:$Rt, t2addrmode_imm8_pre:$addr, pred:$p)>;
def t2LDRSB_POST_imm : t2AsmPseudo<"ldrsb${p}.w $Rt, $Rn, $imm",
                         (ins GPR:$Rt, addr_offset_none:$Rn, t2am_imm8_offset:$imm, pred:$p)>;
```
- EN: Defines TableGen record `t2LDRSB_OFFSET_imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRSB_OFFSET_imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1635-1643
```tablegen
// A7.7.63 LDRSH (immediate) T2
// .w suffixes; Constraints can't be used on t2InstAlias to describe
// "$Rn =  $Rn_wb" on POST or "$addr.base = $Rn_wb" on PRE.
def t2LDRSH_OFFSET_imm : t2AsmPseudo<"ldrsh${p}.w $Rt, $addr",
                         (ins GPR:$Rt, t2addrmode_negimm8:$addr, pred:$p)>;
def t2LDRSH_PRE_imm : t2AsmPseudo<"ldrsh${p}.w $Rt, $addr!",
                         (ins GPR:$Rt, t2addrmode_imm8_pre:$addr, pred:$p)>;
def t2LDRSH_POST_imm : t2AsmPseudo<"ldrsh${p}.w $Rt, $Rn, $imm",
                         (ins GPR:$Rt, addr_offset_none:$Rn, t2am_imm8_offset:$imm, pred:$p)>;
```
- EN: Defines TableGen record `t2LDRSH_OFFSET_imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRSH_OFFSET_imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1645-1662
```tablegen
// LDRT, LDRBT, LDRHT, LDRSBT, LDRSHT all have offset mode (PUW=0b110).
// Ref: A8.6.57 LDR (immediate, Thumb) Encoding T4
class T2IldT<bit signed, bits<2> type, string opc, InstrItinClass ii>
  : T2Ii8p<(outs rGPR:$Rt), (ins t2addrmode_posimm8:$addr), ii, opc,
           "\t$Rt, $addr", []>, Sched<[WriteLd]> {
  bits<4> Rt;
  bits<13> addr;
  let Inst{31-27} = 0b11111;
  let Inst{26-25} = 0b00;
  let Inst{24} = signed;
  let Inst{23} = 0;
  let Inst{22-21} = type;
  let Inst{20} = 1; // load
  let Inst{19-16} = addr{12-9};
  let Inst{15-12} = Rt;
  let Inst{11} = 1;
  let Inst{10-8} = 0b110; // PUW.
  let Inst{7-0} = addr{7-0};
```
- EN: Declares reusable TableGen class `T2IldT` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2IldT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1664-1665
```tablegen
  let DecoderMethod = "DecodeT2LoadT";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1667-1671
```tablegen
def t2LDRT   : T2IldT<0, 0b10, "ldrt", IIC_iLoad_i>;
def t2LDRBT  : T2IldT<0, 0b00, "ldrbt", IIC_iLoad_bh_i>;
def t2LDRHT  : T2IldT<0, 0b01, "ldrht", IIC_iLoad_bh_i>;
def t2LDRSBT : T2IldT<1, 0b00, "ldrsbt", IIC_iLoad_bh_i>;
def t2LDRSHT : T2IldT<1, 0b01, "ldrsht", IIC_iLoad_bh_i>;
```
- EN: Defines TableGen record `t2LDRT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1673-1678
```tablegen
class T2Ildacq<bits<4> bits23_20, bits<2> bit54, dag oops, dag iops,
               string opc, string asm, list<dag> pattern>
  : Thumb2I<oops, iops, AddrModeNone, 4, NoItinerary,
            opc, asm, "", pattern>, Requires<[IsThumb, HasAcquireRelease]> {
  bits<4> Rt;
  bits<4> addr;
```
- EN: Declares reusable TableGen class `T2Ildacq` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2Ildacq`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1680-1685
```tablegen
  let Inst{31-27} = 0b11101;
  let Inst{26-24} = 0b000;
  let Inst{23-20} = bits23_20;
  let Inst{11-6} = 0b111110;
  let Inst{5-4} = bit54;
  let Inst{3-0} = 0b1111;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1687-1690
```tablegen
  // Encode instruction operands
  let Inst{19-16} = addr;
  let Inst{15-12} = Rt;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1692-1700
```tablegen
def t2LDA : T2Ildacq<0b1101, 0b10, (outs rGPR:$Rt),
                     (ins addr_offset_none:$addr), "lda", "\t$Rt, $addr", []>,
            Sched<[WriteLd]>;
def t2LDAB : T2Ildacq<0b1101, 0b00, (outs rGPR:$Rt),
                      (ins addr_offset_none:$addr), "ldab", "\t$Rt, $addr", []>,
            Sched<[WriteLd]>;
def t2LDAH : T2Ildacq<0b1101, 0b01, (outs rGPR:$Rt),
                      (ins addr_offset_none:$addr), "ldah", "\t$Rt, $addr", []>,
            Sched<[WriteLd]>;
```
- EN: Defines TableGen record `t2LDA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1702-1707
```tablegen
// Store
defm t2STR :T2I_st<0b10,"str", IIC_iStore_i, IIC_iStore_si, GPR, store>;
defm t2STRB:T2I_st<0b00,"strb", IIC_iStore_bh_i, IIC_iStore_bh_si,
                   rGPR, truncstorei8>;
defm t2STRH:T2I_st<0b01,"strh", IIC_iStore_bh_i, IIC_iStore_bh_si,
                   rGPR, truncstorei16>;
```
- EN: Defines TableGen record `t2STR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1709-1715
```tablegen
// Store doubleword
let mayStore = 1, hasSideEffects = 0, hasExtraSrcRegAllocReq = 1 in
def t2STRDi8 : T2Ii8s4<1, 0, 0, (outs),
                       (ins rGPR:$Rt, rGPR:$Rt2, t2addrmode_imm8s4:$addr),
               IIC_iStore_d_r, "strd", "\t$Rt, $Rt2, $addr", "",
               [(ARMstrd rGPR:$Rt, rGPR:$Rt2, t2addrmode_imm8s4:$addr)]>,
               Sched<[WriteST]>;
```
- EN: Defines TableGen record `t2STRDi8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STRDi8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1717-1717
```tablegen
// Indexed stores
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1719-1725
```tablegen
let mayStore = 1, hasSideEffects = 0 in {
def t2STR_PRE  : T2Ipreldst<0, 0b10, 0, 1, (outs GPRnopc:$Rn_wb),
                            (ins GPRnopc:$Rt, t2addrmode_imm8_pre:$addr),
                            AddrModeT2_i8, IndexModePre, IIC_iStore_iu,
                            "str", "\t$Rt, $addr!",
                            "$addr.base = $Rn_wb,@earlyclobber $Rn_wb", []>,
                 Sched<[WriteST]>;
```
- EN: Defines TableGen record `t2STR_PRE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STR_PRE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1727-1732
```tablegen
def t2STRH_PRE  : T2Ipreldst<0, 0b01, 0, 1, (outs GPRnopc:$Rn_wb),
                            (ins rGPR:$Rt, t2addrmode_imm8_pre:$addr),
                            AddrModeT2_i8, IndexModePre, IIC_iStore_iu,
                        "strh", "\t$Rt, $addr!",
                        "$addr.base = $Rn_wb,@earlyclobber $Rn_wb", []>,
                  Sched<[WriteST]>;
```
- EN: Defines TableGen record `t2STRH_PRE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STRH_PRE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1734-1740
```tablegen
def t2STRB_PRE  : T2Ipreldst<0, 0b00, 0, 1, (outs GPRnopc:$Rn_wb),
                            (ins rGPR:$Rt, t2addrmode_imm8_pre:$addr),
                            AddrModeT2_i8, IndexModePre, IIC_iStore_bh_iu,
                        "strb", "\t$Rt, $addr!",
                        "$addr.base = $Rn_wb,@earlyclobber $Rn_wb", []>,
            Sched<[WriteST]>;
} // mayStore = 1, hasSideEffects = 0
```
- EN: Defines TableGen record `t2STRB_PRE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STRB_PRE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1742-1751
```tablegen
def t2STR_POST : T2Ipostldst<0, 0b10, 0, 0, (outs GPRnopc:$Rn_wb),
                            (ins GPRnopc:$Rt, addr_offset_none:$Rn,
                                 t2am_imm8_offset:$offset),
                            AddrModeT2_i8, IndexModePost, IIC_iStore_iu,
                          "str", "\t$Rt, $Rn$offset",
                          "$Rn = $Rn_wb,@earlyclobber $Rn_wb",
             [(set GPRnopc:$Rn_wb,
                  (post_store GPRnopc:$Rt, addr_offset_none:$Rn,
                              t2am_imm8_offset:$offset))]>,
            Sched<[WriteST]>;
```
- EN: Defines TableGen record `t2STR_POST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STR_POST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1753-1762
```tablegen
def t2STRH_POST : T2Ipostldst<0, 0b01, 0, 0, (outs GPRnopc:$Rn_wb),
                            (ins rGPR:$Rt, addr_offset_none:$Rn,
                                 t2am_imm8_offset:$offset),
                            AddrModeT2_i8, IndexModePost, IIC_iStore_bh_iu,
                         "strh", "\t$Rt, $Rn$offset",
                         "$Rn = $Rn_wb,@earlyclobber $Rn_wb",
       [(set GPRnopc:$Rn_wb,
             (post_truncsti16 rGPR:$Rt, addr_offset_none:$Rn,
                              t2am_imm8_offset:$offset))]>,
            Sched<[WriteST]>;
```
- EN: Defines TableGen record `t2STRH_POST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STRH_POST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1764-1773
```tablegen
def t2STRB_POST : T2Ipostldst<0, 0b00, 0, 0, (outs GPRnopc:$Rn_wb),
                            (ins rGPR:$Rt, addr_offset_none:$Rn,
                                 t2am_imm8_offset:$offset),
                            AddrModeT2_i8, IndexModePost, IIC_iStore_bh_iu,
                         "strb", "\t$Rt, $Rn$offset",
                         "$Rn = $Rn_wb,@earlyclobber $Rn_wb",
        [(set GPRnopc:$Rn_wb,
              (post_truncsti8 rGPR:$Rt, addr_offset_none:$Rn,
                              t2am_imm8_offset:$offset))]>,
            Sched<[WriteST]>;
```
- EN: Defines TableGen record `t2STRB_POST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STRB_POST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1775-1792
```tablegen
// Pseudo-instructions for pattern matching the pre-indexed stores. We can't
// put the patterns on the instruction definitions directly as ISel wants
// the address base and offset to be separate operands, not a single
// complex operand like we represent the instructions themselves. The
// pseudos map between the two.
let usesCustomInserter = 1,
    Constraints = "$Rn = $Rn_wb,@earlyclobber $Rn_wb" in {
def t2STR_preidx: t2PseudoInst<(outs GPRnopc:$Rn_wb),
               (ins rGPR:$Rt, GPRnopc:$Rn, t2am_imm8_offset:$offset, pred:$p),
               4, IIC_iStore_ru,
      [(set GPRnopc:$Rn_wb,
            (pre_store rGPR:$Rt, GPRnopc:$Rn, t2am_imm8_offset:$offset))]>,
            Sched<[WriteST]>;
def t2STRB_preidx: t2PseudoInst<(outs GPRnopc:$Rn_wb),
               (ins rGPR:$Rt, GPRnopc:$Rn, t2am_imm8_offset:$offset, pred:$p),
               4, IIC_iStore_ru,
      [(set GPRnopc:$Rn_wb,
            (pre_truncsti8 rGPR:$Rt, GPRnopc:$Rn, t2am_imm8_offset:$offset))]>,
```
- EN: Defines TableGen record `t2STR_preidx:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STR_preidx:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1793-1800
```tablegen
            Sched<[WriteST]>;
def t2STRH_preidx: t2PseudoInst<(outs GPRnopc:$Rn_wb),
               (ins rGPR:$Rt, GPRnopc:$Rn, t2am_imm8_offset:$offset, pred:$p),
               4, IIC_iStore_ru,
      [(set GPRnopc:$Rn_wb,
            (pre_truncsti16 rGPR:$Rt, GPRnopc:$Rn, t2am_imm8_offset:$offset))]>,
            Sched<[WriteST]>;
}
```
- EN: Defines TableGen record `t2STRH_preidx:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STRH_preidx:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1802-1802
```tablegen
let mayStore = 1, hasSideEffects = 0 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1804-1811
```tablegen
// F5.1.229 STR (immediate) T4
// .w suffixes; Constraints can't be used on t2InstAlias to describe
// "$Rn =  $Rn_wb,@earlyclobber $Rn_wb" on POST or
// "$addr.base = $Rn_wb,@earlyclobber $Rn_wb" on PRE.
def t2STR_PRE_imm : t2AsmPseudo<"str${p}.w $Rt, $addr!",
  (ins GPR:$Rt, t2addrmode_imm8_pre:$addr, pred:$p)>;
def t2STR_POST_imm : t2AsmPseudo<"str${p}.w $Rt, $Rn, $imm",
  (ins GPR:$Rt, addr_offset_none:$Rn, t2am_imm8_offset:$imm, pred:$p)>;
```
- EN: Defines TableGen record `t2STR_PRE_imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STR_PRE_imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1813-1821
```tablegen
// A7.7.163 STRB (immediate) T3
// .w suffixes; Constraints can't be used on t2InstAlias to describe
// "$Rn =  $Rn_wb" on POST or "$addr.base = $Rn_wb" on PRE.
def t2STRB_OFFSET_imm : t2AsmPseudo<"strb${p}.w $Rt, $addr",
  (ins GPR:$Rt, t2addrmode_negimm8:$addr, pred:$p)>;
def t2STRB_PRE_imm : t2AsmPseudo<"strb${p}.w $Rt, $addr!",
  (ins GPR:$Rt, t2addrmode_imm8_pre:$addr, pred:$p)>;
def t2STRB_POST_imm : t2AsmPseudo<"strb${p}.w $Rt, $Rn, $imm",
  (ins GPR:$Rt, addr_offset_none:$Rn, t2am_imm8_offset:$imm, pred:$p)>;
```
- EN: Defines TableGen record `t2STRB_OFFSET_imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STRB_OFFSET_imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1823-1831
```tablegen
// A7.7.170 STRH (immediate) T3
// .w suffixes; Constraints can't be used on t2InstAlias to describe
// "$Rn =  $Rn_wb" on POST or "$addr.base = $Rn_wb" on PRE.
def t2STRH_OFFSET_imm : t2AsmPseudo<"strh${p}.w $Rt, $addr",
  (ins GPR:$Rt, t2addrmode_negimm8:$addr, pred:$p)>;
def t2STRH_PRE_imm : t2AsmPseudo<"strh${p}.w $Rt, $addr!",
  (ins GPR:$Rt, t2addrmode_imm8_pre:$addr, pred:$p)>;
def t2STRH_POST_imm : t2AsmPseudo<"strh${p}.w $Rt, $Rn, $imm",
  (ins GPR:$Rt, addr_offset_none:$Rn, t2am_imm8_offset:$imm, pred:$p)>;
```
- EN: Defines TableGen record `t2STRH_OFFSET_imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STRH_OFFSET_imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1833-1846
```tablegen
// STRT, STRBT, STRHT all have offset mode (PUW=0b110) and are for disassembly
// only.
// Ref: A8.6.193 STR (immediate, Thumb) Encoding T4
class T2IstT<bits<2> type, string opc, InstrItinClass ii>
  : T2Ii8p<(outs), (ins rGPR:$Rt, t2addrmode_posimm8:$addr), ii, opc,
           "\t$Rt, $addr", []>, Sched<[WriteST]> {
  let Inst{31-27} = 0b11111;
  let Inst{26-25} = 0b00;
  let Inst{24} = 0; // not signed
  let Inst{23} = 0;
  let Inst{22-21} = type;
  let Inst{20} = 0; // store
  let Inst{11} = 1;
  let Inst{10-8} = 0b110; // PUW
```
- EN: Declares reusable TableGen class `T2IstT` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2IstT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1848-1853
```tablegen
  bits<4> Rt;
  bits<13> addr;
  let Inst{15-12} = Rt;
  let Inst{19-16} = addr{12-9};
  let Inst{7-0}   = addr{7-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1855-1857
```tablegen
def t2STRT   : T2IstT<0b10, "strt", IIC_iStore_i>;
def t2STRBT  : T2IstT<0b00, "strbt", IIC_iStore_bh_i>;
def t2STRHT  : T2IstT<0b01, "strht", IIC_iStore_bh_i>;
```
- EN: Defines TableGen record `t2STRT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STRT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1859-1859
```tablegen
} // mayStore = 1, hasSideEffects = 0
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1861-1861
```tablegen
// ldrd / strd pre / post variants
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1863-1869
```tablegen
let mayLoad = 1, hasSideEffects = 0 in
def t2LDRD_PRE  : T2Ii8s4<1, 1, 1, (outs rGPR:$Rt, rGPR:$Rt2, GPR:$wb),
                 (ins t2addrmode_imm8s4_pre:$addr), IIC_iLoad_d_ru,
                 "ldrd", "\t$Rt, $Rt2, $addr!", "$addr.base = $wb", []>,
                 Sched<[WriteLd]> {
  let DecoderMethod = "DecodeT2LDRDPreInstruction";
}
```
- EN: Defines TableGen record `t2LDRD_PRE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRD_PRE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1871-1875
```tablegen
let mayLoad = 1, hasSideEffects = 0 in
def t2LDRD_POST : T2Ii8s4post<0, 1, 1, (outs rGPR:$Rt, rGPR:$Rt2, GPR:$wb),
                 (ins addr_offset_none:$addr, t2am_imm8s4_offset:$imm),
                 IIC_iLoad_d_ru, "ldrd", "\t$Rt, $Rt2, $addr$imm",
                 "$addr.base = $wb", []>, Sched<[WriteLd]>;
```
- EN: Defines TableGen record `t2LDRD_POST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRD_POST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1877-1883
```tablegen
let mayStore = 1, hasSideEffects = 0 in
def t2STRD_PRE  : T2Ii8s4<1, 1, 0, (outs GPR:$wb),
                 (ins rGPR:$Rt, rGPR:$Rt2, t2addrmode_imm8s4_pre:$addr),
                 IIC_iStore_d_ru, "strd", "\t$Rt, $Rt2, $addr!",
                 "$addr.base = $wb", []>, Sched<[WriteST]> {
  let DecoderMethod = "DecodeT2STRDPreInstruction";
}
```
- EN: Defines TableGen record `t2STRD_PRE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STRD_PRE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1885-1890
```tablegen
let mayStore = 1, hasSideEffects = 0 in
def t2STRD_POST : T2Ii8s4post<0, 1, 0, (outs GPR:$wb),
                 (ins rGPR:$Rt, rGPR:$Rt2, addr_offset_none:$addr,
                      t2am_imm8s4_offset:$imm),
                 IIC_iStore_d_ru, "strd", "\t$Rt, $Rt2, $addr$imm",
                 "$addr.base = $wb", []>, Sched<[WriteST]>;
```
- EN: Defines TableGen record `t2STRD_POST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STRD_POST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1892-1898
```tablegen
class T2Istrrel<bits<2> bit54, dag oops, dag iops,
                string opc, string asm, list<dag> pattern>
  : Thumb2I<oops, iops, AddrModeNone, 4, NoItinerary, opc,
            asm, "", pattern>, Requires<[IsThumb, HasAcquireRelease]>,
    Sched<[WriteST]> {
  bits<4> Rt;
  bits<4> addr;
```
- EN: Declares reusable TableGen class `T2Istrrel` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2Istrrel`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1900-1904
```tablegen
  let Inst{31-27} = 0b11101;
  let Inst{26-20} = 0b0001100;
  let Inst{11-6} = 0b111110;
  let Inst{5-4} = bit54;
  let Inst{3-0} = 0b1111;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1906-1909
```tablegen
  // Encode instruction operands
  let Inst{19-16} = addr;
  let Inst{15-12} = Rt;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1911-1916
```tablegen
def t2STL  : T2Istrrel<0b10, (outs), (ins rGPR:$Rt, addr_offset_none:$addr),
                       "stl", "\t$Rt, $addr", []>;
def t2STLB : T2Istrrel<0b00, (outs), (ins rGPR:$Rt, addr_offset_none:$addr),
                       "stlb", "\t$Rt, $addr", []>;
def t2STLH : T2Istrrel<0b01, (outs), (ins rGPR:$Rt, addr_offset_none:$addr),
                       "stlh", "\t$Rt, $addr", []>;
```
- EN: Defines TableGen record `t2STL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1918-1922
```tablegen
// T2Ipl (Preload Data/Instruction) signals the memory system of possible future
// data/instruction access.
// instr_write is inverted for Thumb mode: (prefetch 3) -> (preload 0),
// (prefetch 1) -> (preload 2),  (prefetch 2) -> (preload 1).
multiclass T2Ipl<bits<1> write, bits<1> instr, string opc> {
```
- EN: Declares TableGen `multiclass T2Ipl`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass T2Ipl`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1924-1934
```tablegen
  def i12 : T2Ii12<(outs), (ins t2addrmode_imm12:$addr), IIC_Preload, opc,
                "\t$addr",
              [(ARMPreload t2addrmode_imm12:$addr, (i32 write), (i32 instr))]>,
              Sched<[WritePreLd]> {
    let Inst{31-25} = 0b1111100;
    let Inst{24} = instr;
    let Inst{23} = 1;
    let Inst{22} = 0;
    let Inst{21} = write;
    let Inst{20} = 1;
    let Inst{15-12} = 0b1111;
```
- EN: Defines TableGen record `i12` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `i12`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1936-1938
```tablegen
    bits<17> addr;
    let Inst{19-16} = addr{16-13}; // Rn
    let Inst{11-0}  = addr{11-0};  // imm12
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1940-1941
```tablegen
    let DecoderMethod = "DecodeT2LoadImm12";
  }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1943-1954
```tablegen
  def i8 : T2Ii8n<(outs), (ins t2addrmode_negimm8:$addr), IIC_Preload, opc,
                 "\t$addr",
            [(ARMPreload t2addrmode_negimm8:$addr, (i32 write), (i32 instr))]>,
            Sched<[WritePreLd]> {
    let Inst{31-25} = 0b1111100;
    let Inst{24} = instr;
    let Inst{23} = 0; // U = 0
    let Inst{22} = 0;
    let Inst{21} = write;
    let Inst{20} = 1;
    let Inst{15-12} = 0b1111;
    let Inst{11-8} = 0b1100;
```
- EN: Defines TableGen record `i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1956-1958
```tablegen
    bits<13> addr;
    let Inst{19-16} = addr{12-9}; // Rn
    let Inst{7-0}   = addr{7-0};  // imm8
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1960-1961
```tablegen
    let DecoderMethod = "DecodeT2LoadImm8";
  }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1963-1974
```tablegen
  def s : T2Iso<(outs), (ins t2addrmode_so_reg:$addr), IIC_Preload, opc,
               "\t$addr",
             [(ARMPreload t2addrmode_so_reg:$addr, (i32 write), (i32 instr))]>,
             Sched<[WritePreLd]> {
    let Inst{31-25} = 0b1111100;
    let Inst{24} = instr;
    let Inst{23} = 0; // add = TRUE for T1
    let Inst{22} = 0;
    let Inst{21} = write;
    let Inst{20} = 1;
    let Inst{15-12} = 0b1111;
    let Inst{11-6} = 0b000000;
```
- EN: Defines TableGen record `s` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `s`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1976-1979
```tablegen
    bits<10> addr;
    let Inst{19-16} = addr{9-6}; // Rn
    let Inst{3-0}   = addr{5-2}; // Rm
    let Inst{5-4}   = addr{1-0}; // imm2
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1981-1983
```tablegen
    let DecoderMethod = "DecodeT2LoadShift";
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1985-1987
```tablegen
defm t2PLD    : T2Ipl<0, 0, "pld">,  Requires<[IsThumb2]>;
defm t2PLDW   : T2Ipl<1, 0, "pldw">, Requires<[IsThumb2,HasV7,HasMP]>;
defm t2PLI    : T2Ipl<0, 1, "pli">,  Requires<[IsThumb2,HasV7]>;
```
- EN: Defines TableGen record `t2PLD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2PLD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1989-1995
```tablegen
// PLD/PLDW/PLI aliases w/ the optional .w suffix
def : t2InstAlias<"pld${p}.w\t$addr",
                 (t2PLDi12  t2addrmode_imm12:$addr, pred:$p)>;
def : t2InstAlias<"pld${p}.w\t$addr",
                 (t2PLDi8   t2addrmode_negimm8:$addr, pred:$p)>;
def : t2InstAlias<"pld${p}.w\t$addr",
                 (t2PLDs    t2addrmode_so_reg:$addr, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1997-2005
```tablegen
def : InstAlias<"pldw${p}.w\t$addr",
                 (t2PLDWi12  t2addrmode_imm12:$addr, pred:$p), 0>,
      Requires<[IsThumb2,HasV7,HasMP]>;
def : InstAlias<"pldw${p}.w\t$addr",
                 (t2PLDWi8   t2addrmode_negimm8:$addr, pred:$p), 0>,
      Requires<[IsThumb2,HasV7,HasMP]>;
def : InstAlias<"pldw${p}.w\t$addr",
                 (t2PLDWs    t2addrmode_so_reg:$addr, pred:$p), 0>,
      Requires<[IsThumb2,HasV7,HasMP]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2007-2015
```tablegen
def : InstAlias<"pli${p}.w\t$addr",
                 (t2PLIi12  t2addrmode_imm12:$addr, pred:$p), 0>,
      Requires<[IsThumb2,HasV7]>;
def : InstAlias<"pli${p}.w\t$addr",
                 (t2PLIi8   t2addrmode_negimm8:$addr, pred:$p), 0>,
      Requires<[IsThumb2,HasV7]>;
def : InstAlias<"pli${p}.w\t$addr",
                 (t2PLIs    t2addrmode_so_reg:$addr, pred:$p), 0>,
      Requires<[IsThumb2,HasV7]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2017-2027
```tablegen
// pci variant is very similar to i12, but supports negative offsets
// from the PC. Only PLD and PLI have pci variants (not PLDW)
class T2Iplpci<bits<1> inst, string opc> : T2Ipc<(outs), (ins t2ldrlabel:$addr),
               IIC_Preload, opc, "\t$addr",
               [(ARMPreload (ARMWrapper tconstpool:$addr),
                (i32 0), (i32 inst))]>, Sched<[WritePreLd]> {
  let Inst{31-25} = 0b1111100;
  let Inst{24} = inst;
  let Inst{22-20} = 0b001;
  let Inst{19-16} = 0b1111;
  let Inst{15-12} = 0b1111;
```
- EN: Declares reusable TableGen class `T2Iplpci` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2Iplpci`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2029-2031
```tablegen
  bits<13> addr;
  let Inst{23}   = addr{12};   // add = (U == '1')
  let Inst{11-0} = addr{11-0}; // imm12
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2033-2034
```tablegen
  let DecoderMethod = "DecodeT2LoadLabel";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2036-2037
```tablegen
def t2PLDpci : T2Iplpci<0, "pld">,  Requires<[IsThumb2]>;
def t2PLIpci : T2Iplpci<1, "pli">,  Requires<[IsThumb2,HasV7]>;
```
- EN: Defines TableGen record `t2PLDpci` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2PLDpci`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2039-2043
```tablegen
def : t2InstAlias<"pld${p}.w $addr",
                  (t2PLDpci t2ldrlabel:$addr, pred:$p)>;
def : InstAlias<"pli${p}.w $addr",
                 (t2PLIpci  t2ldrlabel:$addr, pred:$p), 0>,
      Requires<[IsThumb2,HasV7]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2045-2055
```tablegen
// PLD/PLI with alternate literal form.
def : t2InstAlias<"pld${p} $addr",
                  (t2PLDpci t2ldr_pcrel_imm12:$addr, pred:$p)>;
def : InstAlias<"pli${p} $addr",
                 (t2PLIpci  t2ldr_pcrel_imm12:$addr, pred:$p), 0>,
      Requires<[IsThumb2,HasV7]>;
def : t2InstAlias<"pld${p}.w $addr",
                  (t2PLDpci t2ldr_pcrel_imm12:$addr, pred:$p)>;
def : InstAlias<"pli${p}.w $addr",
                 (t2PLIpci  t2ldr_pcrel_imm12:$addr, pred:$p), 0>,
      Requires<[IsThumb2,HasV7]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2057-2059
```tablegen
//===----------------------------------------------------------------------===//
//  Load / store multiple Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2061-2068
```tablegen
multiclass thumb2_ld_mult<string asm, InstrItinClass itin,
                            InstrItinClass itin_upd, bit L_bit> {
  def IA :
    T2XI<(outs), (ins GPR:$Rn, pred:$p, reglist:$regs, variable_ops),
         itin, !strconcat(asm, "${p}.w\t$Rn, $regs"), []> {
    bits<0>  p;
    bits<4>  Rn;
    bits<16> regs;
```
- EN: Declares TableGen `multiclass thumb2_ld_mult`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass thumb2_ld_mult`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2070-2084
```tablegen
    let Inst{31-27} = 0b11101;
    let Inst{26-25} = 0b00;
    let Inst{24-23} = 0b01;     // Increment After
    let Inst{22}    = 0;
    let Inst{21}    = 0;        // No writeback
    let Inst{20}    = L_bit;
    let Inst{19-16} = Rn;
    let Inst{15-0}  = regs;
  }
  def IA_UPD :
    T2XIt<(outs GPR:$wb), (ins GPR:$Rn, pred:$p, reglist:$regs, variable_ops),
          itin_upd, !strconcat(asm, "${p}.w\t$Rn!, $regs"), "$Rn = $wb", []> {
    bits<0>  p;
    bits<4>  Rn;
    bits<16> regs;
```
- EN: Defines TableGen record `IA_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IA_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2086-2100
```tablegen
    let Inst{31-27} = 0b11101;
    let Inst{26-25} = 0b00;
    let Inst{24-23} = 0b01;     // Increment After
    let Inst{22}    = 0;
    let Inst{21}    = 1;        // Writeback
    let Inst{20}    = L_bit;
    let Inst{19-16} = Rn;
    let Inst{15-0}  = regs;
  }
  def DB :
    T2XI<(outs), (ins GPR:$Rn, pred:$p, reglist:$regs, variable_ops),
         itin, !strconcat(asm, "db${p}\t$Rn, $regs"), []> {
    bits<0>  p;
    bits<4>  Rn;
    bits<16> regs;
```
- EN: Defines TableGen record `DB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2102-2116
```tablegen
    let Inst{31-27} = 0b11101;
    let Inst{26-25} = 0b00;
    let Inst{24-23} = 0b10;     // Decrement Before
    let Inst{22}    = 0;
    let Inst{21}    = 0;        // No writeback
    let Inst{20}    = L_bit;
    let Inst{19-16} = Rn;
    let Inst{15-0}  = regs;
  }
  def DB_UPD :
    T2XIt<(outs GPR:$wb), (ins GPR:$Rn, pred:$p, reglist:$regs, variable_ops),
          itin_upd, !strconcat(asm, "db${p}\t$Rn!, $regs"), "$Rn = $wb", []> {
    bits<0>  p;
    bits<4>  Rn;
    bits<16> regs;
```
- EN: Defines TableGen record `DB_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DB_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2118-2127
```tablegen
    let Inst{31-27} = 0b11101;
    let Inst{26-25} = 0b00;
    let Inst{24-23} = 0b10;     // Decrement Before
    let Inst{22}    = 0;
    let Inst{21}    = 1;        // Writeback
    let Inst{20}    = L_bit;
    let Inst{19-16} = Rn;
    let Inst{15-0}  = regs;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2129-2129
```tablegen
let hasSideEffects = 0 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2131-2132
```tablegen
let mayLoad = 1, hasExtraDefRegAllocReq = 1, variadicOpsAreDefs = 1 in
defm t2LDM : thumb2_ld_mult<"ldm", IIC_iLoad_m, IIC_iLoad_mu, 1>;
```
- EN: Defines TableGen record `t2LDM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2134-2141
```tablegen
multiclass thumb2_st_mult<string asm, InstrItinClass itin,
                            InstrItinClass itin_upd, bit L_bit> {
  def IA :
    T2XI<(outs), (ins GPR:$Rn, pred:$p, reglist:$regs, variable_ops),
         itin, !strconcat(asm, "${p}.w\t$Rn, $regs"), []> {
    bits<0>  p;
    bits<4>  Rn;
    bits<16> regs;
```
- EN: Declares TableGen `multiclass thumb2_st_mult`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass thumb2_st_mult`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2143-2160
```tablegen
    let Inst{31-27} = 0b11101;
    let Inst{26-25} = 0b00;
    let Inst{24-23} = 0b01;     // Increment After
    let Inst{22}    = 0;
    let Inst{21}    = 0;        // No writeback
    let Inst{20}    = L_bit;
    let Inst{19-16} = Rn;
    let Inst{15}    = 0;
    let Inst{14}    = regs{14};
    let Inst{13}    = 0;
    let Inst{12-0}  = regs{12-0};
  }
  def IA_UPD :
    T2XIt<(outs GPR:$wb), (ins GPR:$Rn, pred:$p, reglist:$regs, variable_ops),
          itin_upd, !strconcat(asm, "${p}.w\t$Rn!, $regs"), "$Rn = $wb", []> {
    bits<0>  p;
    bits<4>  Rn;
    bits<16> regs;
```
- EN: Defines TableGen record `IA_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IA_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2162-2179
```tablegen
    let Inst{31-27} = 0b11101;
    let Inst{26-25} = 0b00;
    let Inst{24-23} = 0b01;     // Increment After
    let Inst{22}    = 0;
    let Inst{21}    = 1;        // Writeback
    let Inst{20}    = L_bit;
    let Inst{19-16} = Rn;
    let Inst{15}    = 0;
    let Inst{14}    = regs{14};
    let Inst{13}    = 0;
    let Inst{12-0}  = regs{12-0};
  }
  def DB :
    T2XI<(outs), (ins GPR:$Rn, pred:$p, reglist:$regs, variable_ops),
         itin, !strconcat(asm, "db${p}\t$Rn, $regs"), []> {
    bits<0>  p;
    bits<4>  Rn;
    bits<16> regs;
```
- EN: Defines TableGen record `DB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2181-2198
```tablegen
    let Inst{31-27} = 0b11101;
    let Inst{26-25} = 0b00;
    let Inst{24-23} = 0b10;     // Decrement Before
    let Inst{22}    = 0;
    let Inst{21}    = 0;        // No writeback
    let Inst{20}    = L_bit;
    let Inst{19-16} = Rn;
    let Inst{15}    = 0;
    let Inst{14}    = regs{14};
    let Inst{13}    = 0;
    let Inst{12-0}  = regs{12-0};
  }
  def DB_UPD :
    T2XIt<(outs GPR:$wb), (ins GPR:$Rn, pred:$p, reglist:$regs, variable_ops),
          itin_upd, !strconcat(asm, "db${p}\t$Rn!, $regs"), "$Rn = $wb", []> {
    bits<0>  p;
    bits<4>  Rn;
    bits<16> regs;
```
- EN: Defines TableGen record `DB_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DB_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2200-2212
```tablegen
    let Inst{31-27} = 0b11101;
    let Inst{26-25} = 0b00;
    let Inst{24-23} = 0b10;     // Decrement Before
    let Inst{22}    = 0;
    let Inst{21}    = 1;        // Writeback
    let Inst{20}    = L_bit;
    let Inst{19-16} = Rn;
    let Inst{15}    = 0;
    let Inst{14}    = regs{14};
    let Inst{13}    = 0;
    let Inst{12-0}  = regs{12-0};
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2215-2216
```tablegen
let mayStore = 1, hasExtraSrcRegAllocReq = 1 in
defm t2STM : thumb2_st_mult<"stm", IIC_iStore_m, IIC_iStore_mu, 0>;
```
- EN: Defines TableGen record `t2STM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2218-2218
```tablegen
} // hasSideEffects
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2221-2223
```tablegen
//===----------------------------------------------------------------------===//
//  Move Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2225-2241
```tablegen
let hasSideEffects = 0 in
def t2MOVr : T2sTwoReg<(outs GPRnopc:$Rd), (ins GPRnopc:$Rm), IIC_iMOVr,
                   "mov", ".w\t$Rd, $Rm", []>, Sched<[WriteALU]> {
  let Inst{31-27} = 0b11101;
  let Inst{26-25} = 0b01;
  let Inst{24-21} = 0b0010;
  let Inst{19-16} = 0b1111; // Rn
  let Inst{15} = 0b0;
  let Inst{14-12} = 0b000;
  let Inst{7-4} = 0b0000;
}
def : t2InstAlias<"mov${p}.w $Rd, $Rm", (t2MOVr GPRnopc:$Rd, GPRnopc:$Rm,
                                                pred:$p, (cc_out zero_reg))>;
def : t2InstAlias<"movs${p}.w $Rd, $Rm", (t2MOVr GPRnopc:$Rd, GPRnopc:$Rm,
                                                 pred:$p, (cc_out CPSR))>;
def : t2InstAlias<"movs${p} $Rd, $Rm", (t2MOVr GPRnopc:$Rd, GPRnopc:$Rm,
                                               pred:$p, (cc_out CPSR))>;
```
- EN: Defines TableGen record `t2MOVr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MOVr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2243-2254
```tablegen
// AddedComplexity to ensure isel tries t2MOVi before t2MOVi16.
let isReMaterializable = 1, isAsCheapAsAMove = 1, isMoveImm = 1,
    AddedComplexity = 1 in
def t2MOVi : T2sOneRegImm<(outs rGPR:$Rd), (ins t2_so_imm:$imm), IIC_iMOVi,
                   "mov", ".w\t$Rd, $imm",
                   [(set rGPR:$Rd, t2_so_imm:$imm)]>, Sched<[WriteALU]> {
  let Inst{31-27} = 0b11110;
  let Inst{25} = 0;
  let Inst{24-21} = 0b0010;
  let Inst{19-16} = 0b1111; // Rn
  let Inst{15} = 0;
}
```
- EN: Defines TableGen record `t2MOVi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MOVi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2256-2261
```tablegen
// cc_out is handled as part of the explicit mnemonic in the parser for 'mov'.
// Use aliases to get that to play nice here.
def : t2InstAlias<"movs${p}.w $Rd, $imm", (t2MOVi rGPR:$Rd, t2_so_imm:$imm,
                                                pred:$p, (cc_out CPSR))>;
def : t2InstAlias<"movs${p} $Rd, $imm", (t2MOVi rGPR:$Rd, t2_so_imm:$imm,
                                                pred:$p, (cc_out CPSR))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2263-2266
```tablegen
def : t2InstAlias<"mov${p}.w $Rd, $imm", (t2MOVi rGPR:$Rd, t2_so_imm:$imm,
                                                 pred:$p, (cc_out zero_reg))>;
def : t2InstAlias<"mov${p} $Rd, $imm", (t2MOVi rGPR:$Rd, t2_so_imm:$imm,
                                               pred:$p, (cc_out zero_reg))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2268-2277
```tablegen
let isReMaterializable = 1, isAsCheapAsAMove = 1, isMoveImm = 1 in
def t2MOVi16 : T2I<(outs rGPR:$Rd), (ins imm0_65535_expr:$imm), IIC_iMOVi,
                   "movw", "\t$Rd, $imm",
                   [(set rGPR:$Rd, imm0_65535:$imm)]>, Sched<[WriteALU]>,
                   Requires<[IsThumb, HasV8MBaseline]> {
  let Inst{31-27} = 0b11110;
  let Inst{25} = 1;
  let Inst{24-21} = 0b0010;
  let Inst{20} = 0; // The S bit.
  let Inst{15} = 0;
```
- EN: Defines TableGen record `t2MOVi16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MOVi16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2279-2280
```tablegen
  bits<4> Rd;
  bits<16> imm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2282-2288
```tablegen
  let Inst{11-8}  = Rd;
  let Inst{19-16} = imm{15-12};
  let Inst{26}    = imm{11};
  let Inst{14-12} = imm{10-8};
  let Inst{7-0}   = imm{7-0};
  let DecoderMethod = "DecodeT2MOVTWInstruction";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2290-2292
```tablegen
def : InstAlias<"mov${p} $Rd, $imm",
                (t2MOVi16 rGPR:$Rd, imm256_65535_expr:$imm, pred:$p), 0>,
                Requires<[IsThumb, HasV8MBaseline]>, Sched<[WriteALU]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2294-2298
```tablegen
// This gets lowered to a single 4-byte instructions
let Size = 4 in
def t2MOVi16_ga_pcrel : PseudoInst<(outs rGPR:$Rd),
                                (ins i32imm:$addr, pclabel:$id), IIC_iMOVi, []>,
                        Sched<[WriteALU]>;
```
- EN: Defines TableGen record `t2MOVi16_ga_pcrel` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MOVi16_ga_pcrel`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2300-2312
```tablegen
let Constraints = "$src = $Rd" in {
def t2MOVTi16 : T2I<(outs rGPR:$Rd),
                    (ins rGPR:$src, imm0_65535_expr:$imm), IIC_iMOVi,
                    "movt", "\t$Rd, $imm",
                    [(set rGPR:$Rd,
                          (or (and rGPR:$src, 0xffff), lo16AllZero:$imm))]>,
                          Sched<[WriteALU]>,
                          Requires<[IsThumb, HasV8MBaseline]> {
  let Inst{31-27} = 0b11110;
  let Inst{25} = 1;
  let Inst{24-21} = 0b0110;
  let Inst{20} = 0; // The S bit.
  let Inst{15} = 0;
```
- EN: Defines TableGen record `t2MOVTi16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MOVTi16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2314-2315
```tablegen
  bits<4> Rd;
  bits<16> imm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2317-2323
```tablegen
  let Inst{11-8}  = Rd;
  let Inst{19-16} = imm{15-12};
  let Inst{26}    = imm{11};
  let Inst{14-12} = imm{10-8};
  let Inst{7-0}   = imm{7-0};
  let DecoderMethod = "DecodeT2MOVTWInstruction";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2325-2330
```tablegen
// This gets lowered to a single 4-byte instructions
let Size = 4 in
def t2MOVTi16_ga_pcrel : PseudoInst<(outs rGPR:$Rd),
                     (ins rGPR:$src, i32imm:$addr, pclabel:$id), IIC_iMOVi, []>,
                     Sched<[WriteALU]>, Requires<[IsThumb, HasV8MBaseline]>;
} // Constraints
```
- EN: Defines TableGen record `t2MOVTi16_ga_pcrel` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MOVTi16_ga_pcrel`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2332-2332
```tablegen
def : T2Pat<(or rGPR:$src, 0xffff0000), (t2MOVTi16 rGPR:$src, 0xffff)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2334-2336
```tablegen
//===----------------------------------------------------------------------===//
//  Extend Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2338-2338
```tablegen
// Sign extenders
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 2340-2342
```tablegen
def t2SXTB  : T2I_ext_rrot<0b100, "sxtb">;
def t2SXTH  : T2I_ext_rrot<0b000, "sxth">;
def t2SXTB16 : T2I_ext_rrot_xtb16<0b010, "sxtb16">;
```
- EN: Defines TableGen record `t2SXTB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SXTB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2344-2346
```tablegen
def t2SXTAB : T2I_exta_rrot<0b100, "sxtab">;
def t2SXTAH : T2I_exta_rrot<0b000, "sxtah">;
def t2SXTAB16 : T2I_exta_rrot<0b010, "sxtab16">;
```
- EN: Defines TableGen record `t2SXTAB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SXTAB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2348-2365
```tablegen
def : T2Pat<(sext_inreg (rotr rGPR:$Rn, rot_imm:$rot), i8),
            (t2SXTB rGPR:$Rn, rot_imm:$rot)>;
def : T2Pat<(sext_inreg (rotr rGPR:$Rn, rot_imm:$rot), i16),
            (t2SXTH rGPR:$Rn, rot_imm:$rot)>;
def : Thumb2DSPPat<(add rGPR:$Rn,
                            (sext_inreg (rotr rGPR:$Rm, rot_imm:$rot), i8)),
            (t2SXTAB rGPR:$Rn, rGPR:$Rm, rot_imm:$rot)>;
def : Thumb2DSPPat<(add rGPR:$Rn,
                            (sext_inreg (rotr rGPR:$Rm, rot_imm:$rot), i16)),
            (t2SXTAH rGPR:$Rn, rGPR:$Rm, rot_imm:$rot)>;
def : Thumb2DSPPat<(int_arm_sxtb16 rGPR:$Rn),
                   (t2SXTB16 rGPR:$Rn, 0)>;
def : Thumb2DSPPat<(int_arm_sxtab16 rGPR:$Rn, rGPR:$Rm),
                   (t2SXTAB16 rGPR:$Rn, rGPR:$Rm, 0)>;
def : Thumb2DSPPat<(int_arm_sxtb16 (rotr rGPR:$Rn, rot_imm:$rot)),
                   (t2SXTB16 rGPR:$Rn, rot_imm:$rot)>;
def : Thumb2DSPPat<(int_arm_sxtab16 rGPR:$Rn, (rotr rGPR:$Rm, rot_imm:$rot)),
                   (t2SXTAB16 rGPR:$Rn, rGPR:$Rm, rot_imm:$rot)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2368-2383
```tablegen
// A simple right-shift can also be used in most cases (the exception is the
// SXTH operations with a rotate of 24: there the non-contiguous bits are
// relevant).
def : Thumb2DSPPat<(add rGPR:$Rn, (sext_inreg
                                        (srl rGPR:$Rm, rot_imm:$rot), i8)),
                       (t2SXTAB rGPR:$Rn, rGPR:$Rm, rot_imm:$rot)>;
def : Thumb2DSPPat<(add rGPR:$Rn, (sext_inreg
                                        (srl rGPR:$Rm, imm8_or_16:$rot), i16)),
                       (t2SXTAH rGPR:$Rn, rGPR:$Rm, rot_imm:$rot)>;
def : Thumb2DSPPat<(add rGPR:$Rn, (sext_inreg
                                        (rotr rGPR:$Rm, (i32 24)), i16)),
                       (t2SXTAH rGPR:$Rn, rGPR:$Rm, (i32 3))>;
def : Thumb2DSPPat<(add rGPR:$Rn, (sext_inreg
                                        (or (srl rGPR:$Rm, (i32 24)),
                                              (shl rGPR:$Rm, (i32 8))), i16)),
                       (t2SXTAH rGPR:$Rn, rGPR:$Rm, (i32 3))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2385-2385
```tablegen
// Zero extenders
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 2387-2390
```tablegen
let AddedComplexity = 16 in {
def t2UXTB   : T2I_ext_rrot<0b101, "uxtb">;
def t2UXTH   : T2I_ext_rrot<0b001, "uxth">;
def t2UXTB16 : T2I_ext_rrot_xtb16<0b011, "uxtb16">;
```
- EN: Defines TableGen record `t2UXTB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2UXTB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2392-2397
```tablegen
def : Thumb2DSPPat<(and (rotr rGPR:$Rm, rot_imm:$rot), 0x000000FF),
                       (t2UXTB rGPR:$Rm, rot_imm:$rot)>;
def : Thumb2DSPPat<(and (rotr rGPR:$Rm, rot_imm:$rot), 0x0000FFFF),
                       (t2UXTH rGPR:$Rm, rot_imm:$rot)>;
def : Thumb2DSPPat<(and (rotr rGPR:$Rm, rot_imm:$rot), 0x00FF00FF),
                       (t2UXTB16 rGPR:$Rm, rot_imm:$rot)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2399-2402
```tablegen
def : Thumb2DSPPat<(int_arm_uxtb16 rGPR:$Rm),
                   (t2UXTB16 rGPR:$Rm, 0)>;
def : Thumb2DSPPat<(int_arm_uxtb16 (rotr rGPR:$Rn, rot_imm:$rot)),
                   (t2UXTB16 rGPR:$Rn, rot_imm:$rot)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2404-2413
```tablegen
// FIXME: This pattern incorrectly assumes the shl operator is a rotate.
//        The transformation should probably be done as a combiner action
//        instead so we can include a check for masking back in the upper
//        eight bits of the source into the lower eight bits of the result.
//def : T2Pat<(and (shl rGPR:$Src, (i32 8)), 0xFF00FF),
//            (t2UXTB16 rGPR:$Src, 3)>,
//          Requires<[HasDSP, IsThumb2]>;
def : T2Pat<(and (srl rGPR:$Src, (i32 8)), 0xFF00FF),
            (t2UXTB16 rGPR:$Src, 1)>,
        Requires<[HasDSP, IsThumb2]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2415-2417
```tablegen
def t2UXTAB : T2I_exta_rrot<0b101, "uxtab">;
def t2UXTAH : T2I_exta_rrot<0b001, "uxtah">;
def t2UXTAB16 : T2I_exta_rrot<0b011, "uxtab16">;
```
- EN: Defines TableGen record `t2UXTAB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2UXTAB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2419-2435
```tablegen
def : Thumb2DSPPat<(add rGPR:$Rn, (and (rotr rGPR:$Rm, rot_imm:$rot),
                                            0x00FF)),
                       (t2UXTAB rGPR:$Rn, rGPR:$Rm, rot_imm:$rot)>;
def : Thumb2DSPPat<(add rGPR:$Rn, (and (rotr rGPR:$Rm, rot_imm:$rot),
                                            0xFFFF)),
                       (t2UXTAH rGPR:$Rn, rGPR:$Rm, rot_imm:$rot)>;
def : Thumb2DSPPat<(add rGPR:$Rn, (and (srl rGPR:$Rm, rot_imm:$rot),
                                           0xFF)),
                       (t2UXTAB rGPR:$Rn, rGPR:$Rm, rot_imm:$rot)>;
def : Thumb2DSPPat<(add rGPR:$Rn, (and (srl rGPR:$Rm, imm8_or_16:$rot),
                                            0xFFFF)),
                       (t2UXTAH rGPR:$Rn, rGPR:$Rm, rot_imm:$rot)>;
def : Thumb2DSPPat<(int_arm_uxtab16 rGPR:$Rn, rGPR:$Rm),
                      (t2UXTAB16 rGPR:$Rn, rGPR:$Rm, 0)>;
def : Thumb2DSPPat<(int_arm_uxtab16 rGPR:$Rn, (rotr rGPR:$Rm, rot_imm:$rot)),
                   (t2UXTAB16 rGPR:$Rn, rGPR:$Rm, rot_imm:$rot)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2438-2440
```tablegen
//===----------------------------------------------------------------------===//
//  Arithmetic Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2442-2444
```tablegen
let isAdd = 1 in
defm t2ADD  : T2I_bin_ii12rs<0b000, "add", add, 1>;
defm t2SUB  : T2I_bin_ii12rs<0b101, "sub", sub>;
```
- EN: Defines TableGen record `t2ADD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2ADD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2446-2457
```tablegen
// ADD and SUB with 's' bit set. No 12-bit immediate (T4) variants.
//
// Currently, t2ADDS/t2SUBS are pseudo opcodes that exist only in the
// selection DAG. They are "lowered" to real t2ADD/t2SUB opcodes by
// AdjustInstrPostInstrSelection where we determine whether or not to
// set the "s" bit based on CPSR liveness.
//
// FIXME: Eliminate t2ADDS/t2SUBS pseudo opcodes after adding tablegen
// support for an optional CPSR definition that corresponds to the DAG
// node's second value. We can then eliminate the implicit def of CPSR.
defm t2ADDS : T2I_bin_s_irs <IIC_iALUi, IIC_iALUr, IIC_iALUsi, ARMaddc, 1>;
defm t2SUBS : T2I_bin_s_irs <IIC_iALUi, IIC_iALUr, IIC_iALUsi, ARMsubc>;
```
- EN: Defines TableGen record `of` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `of`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2459-2460
```tablegen
defm t2ADC  : T2I_adde_sube_irs<0b1010, "adc", ARMadde, 1, 1>;
defm t2SBC  : T2I_adde_sube_irs<0b1011, "sbc", ARMsube, 0, 1>;
```
- EN: Defines TableGen record `t2ADC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2ADC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2462-2469
```tablegen
def : t2InstSubst<"adc${s}${p} $rd, $rn, $imm",
                 (t2SBCri rGPR:$rd, rGPR:$rn, t2_so_imm_not:$imm, pred:$p, s_cc_out:$s)>;
def : t2InstSubst<"adc${s}${p} $rdn, $imm",
                 (t2SBCri rGPR:$rdn, rGPR:$rdn, t2_so_imm_not:$imm, pred:$p, s_cc_out:$s)>;
def : t2InstSubst<"sbc${s}${p} $rd, $rn, $imm",
                 (t2ADCri rGPR:$rd, rGPR:$rn, t2_so_imm_not:$imm, pred:$p, s_cc_out:$s)>;
def : t2InstSubst<"sbc${s}${p} $rdn, $imm",
                 (t2ADCri rGPR:$rdn, rGPR:$rdn, t2_so_imm_not:$imm, pred:$p, s_cc_out:$s)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2471-2480
```tablegen
def : t2InstSubst<"add${s}${p}.w $rd, $rn, $imm",
                 (t2SUBri rGPR:$rd, GPRnopc:$rn, t2_so_imm_neg:$imm, pred:$p, s_cc_out:$s)>;
def : t2InstSubst<"sub${s}${p}.w $rd, $rn, $imm",
                 (t2ADDri rGPR:$rd, GPRnopc:$rn, t2_so_imm_neg:$imm, pred:$p, s_cc_out:$s)>;
def : t2InstSubst<"subw${p} $Rd, $Rn, $imm",
                 (t2ADDri12 rGPR:$Rd, GPR:$Rn, imm0_4095_neg:$imm, pred:$p)>;
def : t2InstSubst<"sub${s}${p} $rd, $rn, $imm",
                 (t2ADDri rGPR:$rd, GPRnopc:$rn, t2_so_imm_neg:$imm, pred:$p, s_cc_out:$s)>;
def : t2InstSubst<"sub${p} $rd, $rn, $imm",
                 (t2ADDri12 rGPR:$rd, GPR:$rn, imm0_4095_neg:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2482-2492
```tablegen
// SP to SP alike
def : t2InstSubst<"add${s}${p}.w $rd, $rn, $imm",
                 (t2SUBspImm GPRsp:$rd, GPRsp:$rn, t2_so_imm_neg:$imm, pred:$p, s_cc_out:$s)>;
def : t2InstSubst<"sub${s}${p}.w $rd, $rn, $imm",
                 (t2ADDspImm GPRsp:$rd, GPRsp:$rn, t2_so_imm_neg:$imm, pred:$p, s_cc_out:$s)>;
def : t2InstSubst<"subw${p} $Rd, $Rn, $imm",
                 (t2ADDspImm12 GPRsp:$Rd, GPRsp:$Rn, imm0_4095_neg:$imm, pred:$p)>;
def : t2InstSubst<"sub${s}${p} $rd, $rn, $imm",
                 (t2ADDspImm GPRsp:$rd, GPRsp:$rn, t2_so_imm_neg:$imm, pred:$p, s_cc_out:$s)>;
def : t2InstSubst<"sub${p} $rd, $rn, $imm",
                 (t2ADDspImm12 GPRsp:$rd, GPRsp:$rn, imm0_4095_neg:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2495-2496
```tablegen
// RSB
defm t2RSB  : T2I_rbin_irs  <0b1110, "rsb", sub>;
```
- EN: Defines TableGen record `t2RSB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2RSB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2498-2500
```tablegen
// FIXME: Eliminate them if we can write def : Pat patterns which defines
// CPSR and the implicit def of CPSR is not needed.
defm t2RSBS : T2I_rbin_s_is <ARMsubc>;
```
- EN: Defines TableGen record `of` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `of`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2502-2517
```tablegen
// (sub X, imm) gets canonicalized to (add X, -imm).  Match this form.
// The assume-no-carry-in form uses the negation of the input since add/sub
// assume opposite meanings of the carry flag (i.e., carry == !borrow).
// See the definition of AddWithCarry() in the ARM ARM A2.2.1 for the gory
// details.
// The AddedComplexity preferences the first variant over the others since
// it can be shrunk to a 16-bit wide encoding, while the others cannot.
let AddedComplexity = 1 in
def : T2Pat<(add        rGPR:$src, imm1_255_neg:$imm),
            (t2SUBri    rGPR:$src, imm1_255_neg:$imm)>;
def : T2Pat<(add        rGPR:$src, t2_so_imm_neg:$imm),
            (t2SUBri    rGPR:$src, t2_so_imm_neg:$imm)>;
def : T2Pat<(add        rGPR:$src, imm0_4095_neg:$imm),
            (t2SUBri12  rGPR:$src, imm0_4095_neg:$imm)>;
def : T2Pat<(add        GPR:$src, imm0_65535_neg:$imm),
            (t2SUBrr    GPR:$src, (t2MOVi16 (imm_neg_XFORM imm:$imm)))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2519-2522
```tablegen
// Do the same for v8m targets since they support movw with a 16-bit value.
def : T1Pat<(add tGPR:$src, imm0_65535_neg:$imm),
             (tSUBrr tGPR:$src, (t2MOVi16 (imm_neg_XFORM imm:$imm)))>,
             Requires<[HasV8MBaseline]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2524-2540
```tablegen
let AddedComplexity = 1 in
def : T2Pat<(ARMaddc    rGPR:$src, imm1_255_neg:$imm),
            (t2SUBSri   rGPR:$src, imm1_255_neg:$imm)>;
def : T2Pat<(ARMaddc    rGPR:$src, t2_so_imm_neg:$imm),
            (t2SUBSri   rGPR:$src, t2_so_imm_neg:$imm)>;
def : T2Pat<(ARMaddc    rGPR:$src, imm0_65535_neg:$imm),
            (t2SUBSrr   rGPR:$src, (t2MOVi16 (imm_neg_XFORM imm:$imm)))>;
// The with-carry-in form matches bitwise not instead of the negation.
// Effectively, the inverse interpretation of the carry flag already accounts
// for part of the negation.
let AddedComplexity = 1 in
def : T2Pat<(ARMadde    rGPR:$src, imm0_255_not:$imm, CPSR),
            (t2SBCri    rGPR:$src, imm0_255_not:$imm)>;
def : T2Pat<(ARMadde    rGPR:$src, t2_so_imm_not:$imm, CPSR),
            (t2SBCri    rGPR:$src, t2_so_imm_not:$imm)>;
def : T2Pat<(ARMadde    rGPR:$src, imm0_65535_neg:$imm, CPSR),
            (t2SBCrr    rGPR:$src, (t2MOVi16 (imm_not_XFORM imm:$imm)))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2542-2553
```tablegen
def t2SEL : T2ThreeReg<(outs GPR:$Rd), (ins GPR:$Rn, GPR:$Rm),
                NoItinerary, "sel", "\t$Rd, $Rn, $Rm",
                [(set GPR:$Rd, (int_arm_sel GPR:$Rn, GPR:$Rm))]>,
          Requires<[IsThumb2, HasDSP]> {
  let Inst{31-27} = 0b11111;
  let Inst{26-24} = 0b010;
  let Inst{23} = 0b1;
  let Inst{22-20} = 0b010;
  let Inst{15-12} = 0b1111;
  let Inst{7} = 0b1;
  let Inst{6-4} = 0b000;
}
```
- EN: Defines TableGen record `t2SEL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SEL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2555-2565
```tablegen
// A6.3.13, A6.3.14, A6.3.15 Parallel addition and subtraction (signed/unsigned)
// And Miscellaneous operations
class T2I_pam<bits<3> op22_20, bits<4> op7_4, string opc,
              list<dag> pat, dag iops, string asm>
  : T2I<(outs rGPR:$Rd), iops, NoItinerary, opc, asm, pat>,
    Requires<[IsThumb2, HasDSP]> {
  let Inst{31-27} = 0b11111;
  let Inst{26-23} = 0b0101;
  let Inst{22-20} = op22_20;
  let Inst{15-12} = 0b1111;
  let Inst{7-4} = op7_4;
```
- EN: Declares reusable TableGen class `T2I_pam` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2I_pam`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2567-2569
```tablegen
  bits<4> Rd;
  bits<4> Rn;
  bits<4> Rm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2571-2574
```tablegen
  let Inst{11-8}  = Rd;
  let Inst{19-16} = Rn;
  let Inst{3-0}   = Rm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2576-2580
```tablegen
class T2I_pam_intrinsics<bits<3> op22_20, bits<4> op7_4, string opc,
                         Intrinsic intrinsic>
  : T2I_pam<op22_20, op7_4, opc,
    [(set rGPR:$Rd, (intrinsic rGPR:$Rn, rGPR:$Rm))],
    (ins rGPR:$Rn, rGPR:$Rm), "\t$Rd, $Rn, $Rm">;
```
- EN: Declares reusable TableGen class `T2I_pam_intrinsics` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2I_pam_intrinsics`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2582-2584
```tablegen
class T2I_pam_intrinsics_rev<bits<3> op22_20, bits<4> op7_4, string opc>
  : T2I_pam<op22_20, op7_4, opc, [],
    (ins rGPR:$Rm, rGPR:$Rn), "\t$Rd, $Rm, $Rn">;
```
- EN: Declares reusable TableGen class `T2I_pam_intrinsics_rev` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2I_pam_intrinsics_rev`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2586-2602
```tablegen
// Saturating add/subtract
def t2QADD16  : T2I_pam_intrinsics<0b001, 0b0001, "qadd16", int_arm_qadd16>;
def t2QADD8   : T2I_pam_intrinsics<0b000, 0b0001, "qadd8", int_arm_qadd8>;
def t2QASX    : T2I_pam_intrinsics<0b010, 0b0001, "qasx", int_arm_qasx>;
def t2UQSUB8  : T2I_pam_intrinsics<0b100, 0b0101, "uqsub8", int_arm_uqsub8>;
def t2QSAX    : T2I_pam_intrinsics<0b110, 0b0001, "qsax", int_arm_qsax>;
def t2QSUB16  : T2I_pam_intrinsics<0b101, 0b0001, "qsub16", int_arm_qsub16>;
def t2QSUB8   : T2I_pam_intrinsics<0b100, 0b0001, "qsub8", int_arm_qsub8>;
def t2UQADD16 : T2I_pam_intrinsics<0b001, 0b0101, "uqadd16", int_arm_uqadd16>;
def t2UQADD8  : T2I_pam_intrinsics<0b000, 0b0101, "uqadd8", int_arm_uqadd8>;
def t2UQASX   : T2I_pam_intrinsics<0b010, 0b0101, "uqasx", int_arm_uqasx>;
def t2UQSAX   : T2I_pam_intrinsics<0b110, 0b0101, "uqsax", int_arm_uqsax>;
def t2UQSUB16 : T2I_pam_intrinsics<0b101, 0b0101, "uqsub16", int_arm_uqsub16>;
def t2QADD    : T2I_pam_intrinsics_rev<0b000, 0b1000, "qadd">;
def t2QSUB    : T2I_pam_intrinsics_rev<0b000, 0b1010, "qsub">;
def t2QDADD   : T2I_pam_intrinsics_rev<0b000, 0b1001, "qdadd">;
def t2QDSUB   : T2I_pam_intrinsics_rev<0b000, 0b1011, "qdsub">;
```
- EN: Defines TableGen record `t2QADD16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2QADD16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2604-2611
```tablegen
def : Thumb2DSPPat<(int_arm_qadd rGPR:$Rm, rGPR:$Rn),
                   (t2QADD rGPR:$Rm, rGPR:$Rn)>;
def : Thumb2DSPPat<(int_arm_qsub rGPR:$Rm, rGPR:$Rn),
                   (t2QSUB rGPR:$Rm, rGPR:$Rn)>;
def : Thumb2DSPPat<(int_arm_qadd rGPR:$Rm, (int_arm_qadd rGPR:$Rn, rGPR:$Rn)),
                   (t2QDADD rGPR:$Rm, rGPR:$Rn)>;
def : Thumb2DSPPat<(int_arm_qsub rGPR:$Rm, (int_arm_qadd rGPR:$Rn, rGPR:$Rn)),
                   (t2QDSUB rGPR:$Rm, rGPR:$Rn)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2613-2620
```tablegen
def : Thumb2DSPPat<(saddsat rGPR:$Rm, rGPR:$Rn),
                   (t2QADD rGPR:$Rm, rGPR:$Rn)>;
def : Thumb2DSPPat<(ssubsat rGPR:$Rm, rGPR:$Rn),
                   (t2QSUB rGPR:$Rm, rGPR:$Rn)>;
def : Thumb2DSPPat<(saddsat rGPR:$Rm, (saddsat rGPR:$Rn, rGPR:$Rn)),
                   (t2QDADD rGPR:$Rm, rGPR:$Rn)>;
def : Thumb2DSPPat<(ssubsat rGPR:$Rm, (saddsat rGPR:$Rn, rGPR:$Rn)),
                   (t2QDSUB rGPR:$Rm, rGPR:$Rn)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2622-2629
```tablegen
def : Thumb2DSPPat<(ARMqadd8b rGPR:$Rm, rGPR:$Rn),
                   (t2QADD8 rGPR:$Rm, rGPR:$Rn)>;
def : Thumb2DSPPat<(ARMqsub8b rGPR:$Rm, rGPR:$Rn),
                   (t2QSUB8 rGPR:$Rm, rGPR:$Rn)>;
def : Thumb2DSPPat<(ARMqadd16b rGPR:$Rm, rGPR:$Rn),
                   (t2QADD16 rGPR:$Rm, rGPR:$Rn)>;
def : Thumb2DSPPat<(ARMqsub16b rGPR:$Rm, rGPR:$Rn),
                   (t2QSUB16 rGPR:$Rm, rGPR:$Rn)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2631-2638
```tablegen
def : Thumb2DSPPat<(ARMuqadd8b rGPR:$Rm, rGPR:$Rn),
                   (t2UQADD8 rGPR:$Rm, rGPR:$Rn)>;
def : Thumb2DSPPat<(ARMuqsub8b rGPR:$Rm, rGPR:$Rn),
                   (t2UQSUB8 rGPR:$Rm, rGPR:$Rn)>;
def : Thumb2DSPPat<(ARMuqadd16b rGPR:$Rm, rGPR:$Rn),
                   (t2UQADD16 rGPR:$Rm, rGPR:$Rn)>;
def : Thumb2DSPPat<(ARMuqsub16b rGPR:$Rm, rGPR:$Rn),
                   (t2UQSUB16 rGPR:$Rm, rGPR:$Rn)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2640-2640
```tablegen
// Signed/Unsigned add/subtract
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 2642-2653
```tablegen
def t2SASX    : T2I_pam_intrinsics<0b010, 0b0000, "sasx", int_arm_sasx>;
def t2SADD16  : T2I_pam_intrinsics<0b001, 0b0000, "sadd16", int_arm_sadd16>;
def t2SADD8   : T2I_pam_intrinsics<0b000, 0b0000, "sadd8", int_arm_sadd8>;
def t2SSAX    : T2I_pam_intrinsics<0b110, 0b0000, "ssax", int_arm_ssax>;
def t2SSUB16  : T2I_pam_intrinsics<0b101, 0b0000, "ssub16", int_arm_ssub16>;
def t2SSUB8   : T2I_pam_intrinsics<0b100, 0b0000, "ssub8", int_arm_ssub8>;
def t2UASX    : T2I_pam_intrinsics<0b010, 0b0100, "uasx", int_arm_uasx>;
def t2UADD16  : T2I_pam_intrinsics<0b001, 0b0100, "uadd16", int_arm_uadd16>;
def t2UADD8   : T2I_pam_intrinsics<0b000, 0b0100, "uadd8", int_arm_uadd8>;
def t2USAX    : T2I_pam_intrinsics<0b110, 0b0100, "usax", int_arm_usax>;
def t2USUB16  : T2I_pam_intrinsics<0b101, 0b0100, "usub16", int_arm_usub16>;
def t2USUB8   : T2I_pam_intrinsics<0b100, 0b0100, "usub8", int_arm_usub8>;
```
- EN: Defines TableGen record `t2SASX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SASX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2655-2655
```tablegen
// Signed/Unsigned halving add/subtract
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 2657-2668
```tablegen
def t2SHASX   : T2I_pam_intrinsics<0b010, 0b0010, "shasx", int_arm_shasx>;
def t2SHADD16 : T2I_pam_intrinsics<0b001, 0b0010, "shadd16", int_arm_shadd16>;
def t2SHADD8  : T2I_pam_intrinsics<0b000, 0b0010, "shadd8", int_arm_shadd8>;
def t2SHSAX   : T2I_pam_intrinsics<0b110, 0b0010, "shsax", int_arm_shsax>;
def t2SHSUB16 : T2I_pam_intrinsics<0b101, 0b0010, "shsub16", int_arm_shsub16>;
def t2SHSUB8  : T2I_pam_intrinsics<0b100, 0b0010, "shsub8", int_arm_shsub8>;
def t2UHASX   : T2I_pam_intrinsics<0b010, 0b0110, "uhasx", int_arm_uhasx>;
def t2UHADD16 : T2I_pam_intrinsics<0b001, 0b0110, "uhadd16", int_arm_uhadd16>;
def t2UHADD8  : T2I_pam_intrinsics<0b000, 0b0110, "uhadd8", int_arm_uhadd8>;
def t2UHSAX   : T2I_pam_intrinsics<0b110, 0b0110, "uhsax", int_arm_uhsax>;
def t2UHSUB16 : T2I_pam_intrinsics<0b101, 0b0110, "uhsub16", int_arm_uhsub16>;
def t2UHSUB8  : T2I_pam_intrinsics<0b100, 0b0110, "uhsub8", int_arm_uhsub8>;
```
- EN: Defines TableGen record `t2SHASX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SHASX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2670-2681
```tablegen
// Helper class for disassembly only
// A6.3.16 & A6.3.17
// T2Imac - Thumb2 multiply [accumulate, and absolute difference] instructions.
class T2ThreeReg_mac<bit long, bits<3> op22_20, bits<4> op7_4, dag oops,
  dag iops, InstrItinClass itin, string opc, string asm, list<dag> pattern>
  : T2ThreeReg<oops, iops, itin, opc, asm, pattern> {
  let Inst{31-27} = 0b11111;
  let Inst{26-24} = 0b011;
  let Inst{23}    = long;
  let Inst{22-20} = op22_20;
  let Inst{7-4}   = op7_4;
}
```
- EN: Declares reusable TableGen class `for` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `for`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2683-2691
```tablegen
class T2FourReg_mac<bit long, bits<3> op22_20, bits<4> op7_4, dag oops,
  dag iops, InstrItinClass itin, string opc, string asm, list<dag> pattern>
  : T2FourReg<oops, iops, itin, opc, asm, pattern> {
  let Inst{31-27} = 0b11111;
  let Inst{26-24} = 0b011;
  let Inst{23}    = long;
  let Inst{22-20} = op22_20;
  let Inst{7-4}   = op7_4;
}
```
- EN: Declares reusable TableGen class `T2FourReg_mac` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2FourReg_mac`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2693-2705
```tablegen
// Unsigned Sum of Absolute Differences [and Accumulate].
def t2USAD8   : T2ThreeReg_mac<0, 0b111, 0b0000, (outs rGPR:$Rd),
                                           (ins rGPR:$Rn, rGPR:$Rm),
                        NoItinerary, "usad8", "\t$Rd, $Rn, $Rm",
                        [(set rGPR:$Rd, (int_arm_usad8 rGPR:$Rn, rGPR:$Rm))]>,
          Requires<[IsThumb2, HasDSP]> {
  let Inst{15-12} = 0b1111;
}
def t2USADA8  : T2FourReg_mac<0, 0b111, 0b0000, (outs rGPR:$Rd),
                       (ins rGPR:$Rn, rGPR:$Rm, rGPR:$Ra), NoItinerary,
                        "usada8", "\t$Rd, $Rn, $Rm, $Ra",
          [(set rGPR:$Rd, (int_arm_usada8 rGPR:$Rn, rGPR:$Rm, rGPR:$Ra))]>,
          Requires<[IsThumb2, HasDSP]>;
```
- EN: Defines TableGen record `t2USAD8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2USAD8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2707-2713
```tablegen
// Signed/Unsigned saturate.
class T2SatI<dag iops, string opc, string asm>
  : T2I<(outs rGPR:$Rd), iops, NoItinerary, opc, asm, []> {
  bits<4> Rd;
  bits<4> Rn;
  bits<5> sat_imm;
  bits<6> sh;
```
- EN: Declares reusable TableGen class `T2SatI` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2SatI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2715-2725
```tablegen
  let Inst{31-24} = 0b11110011;
  let Inst{21} = sh{5};
  let Inst{20} = 0;
  let Inst{19-16} = Rn;
  let Inst{15} = 0;
  let Inst{14-12} = sh{4-2};
  let Inst{11-8}  = Rd;
  let Inst{7-6} = sh{1-0};
  let Inst{5} = 0;
  let Inst{4-0}   = sat_imm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2727-2732
```tablegen
def t2SSAT: T2SatI<(ins imm1_32:$sat_imm, rGPR:$Rn, t2_shift_imm:$sh),
                   "ssat", "\t$Rd, $sat_imm, $Rn$sh">,
                   Requires<[IsThumb2]>, Sched<[WriteALU]> {
  let Inst{23-22} = 0b00;
  let Inst{5}  = 0;
}
```
- EN: Defines TableGen record `t2SSAT:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SSAT:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2734-2740
```tablegen
def t2SSAT16: T2SatI<(ins imm1_16:$sat_imm, rGPR:$Rn),
                     "ssat16", "\t$Rd, $sat_imm, $Rn">,
                     Requires<[IsThumb2, HasDSP]>, Sched<[WriteALU]> {
  let Inst{23-22} = 0b00;
  let sh = 0b100000;
  let Inst{4} = 0;
}
```
- EN: Defines TableGen record `t2SSAT16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SSAT16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2742-2746
```tablegen
def t2USAT: T2SatI<(ins imm0_31:$sat_imm, rGPR:$Rn, t2_shift_imm:$sh),
                    "usat", "\t$Rd, $sat_imm, $Rn$sh">,
                    Requires<[IsThumb2]>, Sched<[WriteALU]> {
  let Inst{23-22} = 0b10;
}
```
- EN: Defines TableGen record `t2USAT:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2USAT:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2748-2754
```tablegen
def t2USAT16: T2SatI<(ins imm0_15:$sat_imm, rGPR:$Rn),
                     "usat16", "\t$Rd, $sat_imm, $Rn">,
                     Requires<[IsThumb2, HasDSP]>, Sched<[WriteALU]> {
  let Inst{23-22} = 0b10;
  let sh = 0b100000;
  let Inst{4} = 0;
}
```
- EN: Defines TableGen record `t2USAT16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2USAT16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2756-2773
```tablegen
def : T2Pat<(ARMssat GPRnopc:$Rn, imm0_31:$imm),
             (t2SSAT imm0_31:$imm, GPRnopc:$Rn, 0)>;
def : T2Pat<(ARMusat GPRnopc:$Rn, imm0_31:$imm),
             (t2USAT imm0_31:$imm, GPRnopc:$Rn, 0)>;
def : T2Pat<(int_arm_ssat GPR:$a, imm1_32:$pos),
            (t2SSAT imm1_32:$pos, GPR:$a, 0)>;
def : T2Pat<(int_arm_usat GPR:$a, imm0_31:$pos),
            (t2USAT imm0_31:$pos, GPR:$a, 0)>;
def : T2Pat<(int_arm_ssat16 GPR:$a, imm1_16:$pos),
            (t2SSAT16 imm1_16:$pos, GPR:$a)>;
def : T2Pat<(int_arm_usat16 GPR:$a, imm0_15:$pos),
            (t2USAT16 imm0_15:$pos, GPR:$a)>;
def : T2Pat<(int_arm_ssat (shl GPRnopc:$a, imm0_31:$shft), imm1_32:$pos),
            (t2SSAT imm1_32:$pos, GPRnopc:$a, imm0_31:$shft)>;
def : T2Pat<(int_arm_ssat (sra GPRnopc:$a, asr_imm:$shft), imm1_32:$pos),
            (t2SSAT imm1_32:$pos, GPRnopc:$a, asr_imm:$shft)>;
def : T2Pat<(int_arm_usat (shl GPRnopc:$a, imm0_31:$shft), imm0_31:$pos),
            (t2USAT imm0_31:$pos, GPRnopc:$a, imm0_31:$shft)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2774-2783
```tablegen
def : T2Pat<(int_arm_usat (sra GPRnopc:$a, asr_imm:$shft), imm0_31:$pos),
            (t2USAT imm0_31:$pos, GPRnopc:$a, asr_imm:$shft)>;
def : T2Pat<(ARMssat (shl GPRnopc:$a, imm0_31:$shft), imm0_31:$pos),
            (t2SSAT imm0_31:$pos, GPRnopc:$a, imm0_31:$shft)>;
def : T2Pat<(ARMssat (sra GPRnopc:$Rn, asr_imm:$shft), imm0_31:$pos),
            (t2SSAT imm0_31:$pos, GPRnopc:$Rn, asr_imm:$shft)>;
def : T2Pat<(ARMusat (shl GPRnopc:$a, imm0_31:$shft), imm0_31:$pos),
            (t2USAT imm0_31:$pos, GPRnopc:$a, imm0_31:$shft)>;
def : T2Pat<(ARMusat (sra GPRnopc:$Rn, asr_imm:$shft), imm0_31:$pos),
            (t2USAT imm0_31:$pos, GPRnopc:$Rn, asr_imm:$shft)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2786-2788
```tablegen
//===----------------------------------------------------------------------===//
//  Shift and rotate Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2790-2793
```tablegen
defm t2LSL  : T2I_sh_ir<0b00, "lsl", imm1_31, shl>;
defm t2LSR  : T2I_sh_ir<0b01, "lsr", imm_sr,  srl>;
defm t2ASR  : T2I_sh_ir<0b10, "asr", imm_sr,  sra>;
defm t2ROR  : T2I_sh_ir<0b11, "ror", imm1_31, rotr>;
```
- EN: Defines TableGen record `t2LSL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LSL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2795-2800
```tablegen
// LSL #0 is actually MOV, and has slightly different permitted registers to
// LSL with non-zero shift
def : t2InstAlias<"lsl${s}${p} $Rd, $Rm, #0",
                  (t2MOVr GPRnopc:$Rd, GPRnopc:$Rm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"lsl${s}${p}.w $Rd, $Rm, #0",
                  (t2MOVr GPRnopc:$Rd, GPRnopc:$Rm, pred:$p, cc_out:$s)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2802-2804
```tablegen
// (rotr x, (and y, 0x...1f)) ==> (ROR x, y)
def : T2Pat<(rotr rGPR:$lhs, (and rGPR:$rhs, lo5AllOne)),
            (t2RORrr rGPR:$lhs, rGPR:$rhs)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2806-2820
```tablegen
let Uses = [CPSR] in {
def t2RRX : T2sTwoReg<(outs rGPR:$Rd), (ins rGPR:$Rm), IIC_iMOVsi,
                      "rrx", "\t$Rd, $Rm",
                      [(set rGPR:$Rd, (ARMrrx rGPR:$Rm, CPSR))]>,
            Sched<[WriteALU]> {
  let Inst{31-27} = 0b11101;
  let Inst{26-25} = 0b01;
  let Inst{24-21} = 0b0010;
  let Inst{19-16} = 0b1111; // Rn
  let Inst{15} = 0b0;
  let Unpredictable{15} = 0b1;
  let Inst{14-12} = 0b000;
  let Inst{7-4} = 0b0011;
}
}
```
- EN: Defines TableGen record `t2RRX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2RRX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2822-2839
```tablegen
// These differ from t2LSRri / t2ASRri in that they are flag-setting
// and have a hardcoded shift amount = 1.
let isCodeGenOnly = 1, Defs = [CPSR] in {
def t2LSRs1 : T2TwoRegShiftImm<(outs rGPR:$Rd), (ins rGPR:$Rm), IIC_iMOVsi,
                               "lsrs", ".w\t$Rd, $Rm, #1",
                               [(set rGPR:$Rd, CPSR, (ARMlsrs1 rGPR:$Rm))]>,
              Sched<[WriteALU]> {
  let Inst{31-27} = 0b11101;
  let Inst{26-25} = 0b01;
  let Inst{24-21} = 0b0010;
  let Inst{20} = 1; // The S bit.
  let Inst{19-16} = 0b1111; // Rn
  let Inst{5-4} = 0b01; // Shift type.
  // Shift amount = Inst{14-12:7-6} = 1.
  let Inst{14-12} = 0b000;
  let Inst{7-6} = 0b01;
}
def t2ASRs1 : T2TwoRegShiftImm<(outs rGPR:$Rd), (ins rGPR:$Rm), IIC_iMOVsi,
```
- EN: Defines TableGen record `t2LSRs1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LSRs1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2840-2853
```tablegen
                               "asrs", ".w\t$Rd, $Rm, #1",
                               [(set rGPR:$Rd, CPSR, (ARMasrs1 rGPR:$Rm))]>,
              Sched<[WriteALU]> {
  let Inst{31-27} = 0b11101;
  let Inst{26-25} = 0b01;
  let Inst{24-21} = 0b0010;
  let Inst{20} = 1; // The S bit.
  let Inst{19-16} = 0b1111; // Rn
  let Inst{5-4} = 0b10; // Shift type.
  // Shift amount = Inst{14-12:7-6} = 1.
  let Inst{14-12} = 0b000;
  let Inst{7-6} = 0b01;
}
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2855-2857
```tablegen
//===----------------------------------------------------------------------===//
//  Bitwise Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2859-2864
```tablegen
defm t2AND  : T2I_bin_w_irs<0b0000, "and",
                            IIC_iBITi, IIC_iBITr, IIC_iBITsi, and, 1>;
defm t2ORR  : T2I_bin_w_irs<0b0010, "orr",
                            IIC_iBITi, IIC_iBITr, IIC_iBITsi, or, 1>;
defm t2EOR  : T2I_bin_w_irs<0b0100, "eor",
                            IIC_iBITi, IIC_iBITr, IIC_iBITsi, xor, 1>;
```
- EN: Defines TableGen record `t2AND` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2AND`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2866-2868
```tablegen
defm t2BIC  : T2I_bin_w_irs<0b0001, "bic",
                            IIC_iBITi, IIC_iBITr, IIC_iBITsi,
                            BinOpFrag<(and node:$LHS, (not node:$RHS))>>;
```
- EN: Defines TableGen record `t2BIC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2BIC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2870-2875
```tablegen
class T2BitFI<dag oops, dag iops, InstrItinClass itin,
              string opc, string asm, list<dag> pattern>
    : T2I<oops, iops, itin, opc, asm, pattern> {
  bits<4> Rd;
  bits<5> msb;
  bits<5> lsb;
```
- EN: Declares reusable TableGen class `T2BitFI` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2BitFI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2877-2881
```tablegen
  let Inst{11-8}  = Rd;
  let Inst{4-0}   = msb{4-0};
  let Inst{14-12} = lsb{4-2};
  let Inst{7-6}   = lsb{1-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2883-2886
```tablegen
class T2TwoRegBitFI<dag oops, dag iops, InstrItinClass itin,
              string opc, string asm, list<dag> pattern>
    : T2BitFI<oops, iops, itin, opc, asm, pattern> {
  bits<4> Rn;
```
- EN: Declares reusable TableGen class `T2TwoRegBitFI` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2TwoRegBitFI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2888-2889
```tablegen
  let Inst{19-16} = Rn;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2891-2901
```tablegen
let Constraints = "$src = $Rd" in
def t2BFC : T2BitFI<(outs rGPR:$Rd), (ins rGPR:$src, bf_inv_mask_imm:$imm),
                IIC_iUNAsi, "bfc", "\t$Rd, $imm",
                [(set rGPR:$Rd, (and rGPR:$src, bf_inv_mask_imm:$imm))]>, Sched<[WriteALU]> {
  let Inst{31-27} = 0b11110;
  let Inst{26} = 0; // should be 0.
  let Inst{25} = 1;
  let Inst{24-20} = 0b10110;
  let Inst{19-16} = 0b1111; // Rn
  let Inst{15} = 0;
  let Inst{5} = 0; // should be 0.
```
- EN: Defines TableGen record `t2BFC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2BFC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2903-2906
```tablegen
  bits<10> imm;
  let msb{4-0} = imm{9-5};
  let lsb{4-0} = imm{4-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2908-2914
```tablegen
def t2SBFX: T2TwoRegBitFI<
                (outs rGPR:$Rd), (ins rGPR:$Rn, imm0_31:$lsb, imm1_32:$msb),
                 IIC_iUNAsi, "sbfx", "\t$Rd, $Rn, $lsb, $msb", []>, Sched<[WriteALU]> {
  let Inst{31-27} = 0b11110;
  let Inst{25} = 1;
  let Inst{24-20} = 0b10100;
  let Inst{15} = 0;
```
- EN: Defines TableGen record `t2SBFX:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SBFX:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2916-2917
```tablegen
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2919-2925
```tablegen
def t2UBFX: T2TwoRegBitFI<
                (outs rGPR:$Rd), (ins rGPR:$Rn, imm0_31:$lsb, imm1_32:$msb),
                 IIC_iUNAsi, "ubfx", "\t$Rd, $Rn, $lsb, $msb", []>, Sched<[WriteALU]> {
  let Inst{31-27} = 0b11110;
  let Inst{25} = 1;
  let Inst{24-20} = 0b11100;
  let Inst{15} = 0;
```
- EN: Defines TableGen record `t2UBFX:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2UBFX:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2927-2928
```tablegen
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2930-2941
```tablegen
// A8.8.247  UDF - Undefined (Encoding T2)
def t2UDF : T2XI<(outs), (ins imm0_65535:$imm16), IIC_Br, "udf.w\t$imm16",
                 [(int_arm_undefined imm0_65535:$imm16)]> {
  bits<16> imm16;
  let Inst{31-29} = 0b111;
  let Inst{28-27} = 0b10;
  let Inst{26-20} = 0b1111111;
  let Inst{19-16} = imm16{15-12};
  let Inst{15} = 0b1;
  let Inst{14-12} = 0b010;
  let Inst{11-0} = imm16{11-0};
}
```
- EN: Defines TableGen record `t2UDF` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2UDF`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2943-2955
```tablegen
// A8.6.18  BFI - Bitfield insert (Encoding T1)
let Constraints = "$src = $Rd" in {
  def t2BFI : T2TwoRegBitFI<(outs rGPR:$Rd),
                  (ins rGPR:$src, rGPR:$Rn, bf_inv_mask_imm:$imm),
                  IIC_iBITi, "bfi", "\t$Rd, $Rn, $imm",
                  [(set rGPR:$Rd, (ARMbfi rGPR:$src, rGPR:$Rn,
                                   bf_inv_mask_imm:$imm))]>, Sched<[WriteALU]> {
    let Inst{31-27} = 0b11110;
    let Inst{26} = 0; // should be 0.
    let Inst{25} = 1;
    let Inst{24-20} = 0b10110;
    let Inst{15} = 0;
    let Inst{5} = 0; // should be 0.
```
- EN: Defines TableGen record `t2BFI` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2BFI`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2957-2961
```tablegen
    bits<10> imm;
    let msb{4-0} = imm{9-5};
    let lsb{4-0} = imm{4-0};
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2963-2971
```tablegen
defm t2ORN  : T2I_bin_irs<0b0011, "orn",
                          IIC_iBITi, IIC_iBITr, IIC_iBITsi,
                          BinOpFrag<(or node:$LHS, (not node:$RHS))>, 0, "">;
def : t2InstAlias<"orn${s}${p}.w $Rd, $Rn, $imm",
   (t2ORNri rGPR:$Rd, rGPR:$Rn, t2_so_imm:$imm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"orn${s}${p}.w $Rd, $Rn, $Rm",
   (t2ORNrr rGPR:$Rd, rGPR:$Rn, rGPR:$Rm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"orn${s}${p}.w $Rd, $Rn, $ShiftedRm",
   (t2ORNrs rGPR:$Rd, rGPR:$Rn, t2_so_reg:$ShiftedRm, pred:$p, cc_out:$s)>;
```
- EN: Defines TableGen record `t2ORN` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2ORN`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2973-2990
```tablegen
/// T2I_un_irs - Defines a set of (op reg, {so_imm|r|so_reg}) patterns for a
/// unary operation that produces a value. These are predicable and can be
/// changed to modify CPSR.
multiclass T2I_un_irs<bits<4> opcod, string opc,
                     InstrItinClass iii, InstrItinClass iir, InstrItinClass iis,
                      PatFrag opnode,
                      bit Cheap = 0, bit ReMat = 0, bit MoveImm = 0> {
   // shifted imm
   def i : T2sOneRegImm<(outs rGPR:$Rd), (ins t2_so_imm:$imm), iii,
                opc, "\t$Rd, $imm",
                [(set rGPR:$Rd, (opnode t2_so_imm:$imm))]>, Sched<[WriteALU]> {
     let isAsCheapAsAMove = Cheap;
     let isReMaterializable = ReMat;
     let isMoveImm = MoveImm;
     let Inst{31-27} = 0b11110;
     let Inst{25} = 0;
     let Inst{24-21} = opcod;
     let Inst{19-16} = 0b1111; // Rn
```
- EN: Declares TableGen `multiclass T2I_un_irs`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass T2I_un_irs`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2991-3008
```tablegen
     let Inst{15} = 0;
   }
   // register
   def r : T2sTwoReg<(outs rGPR:$Rd), (ins rGPR:$Rm), iir,
                opc, ".w\t$Rd, $Rm",
                [(set rGPR:$Rd, (opnode rGPR:$Rm))]>, Sched<[WriteALU]> {
     let Inst{31-27} = 0b11101;
     let Inst{26-25} = 0b01;
     let Inst{24-21} = opcod;
     let Inst{19-16} = 0b1111; // Rn
     let Inst{14-12} = 0b000; // imm3
     let Inst{7-6} = 0b00; // imm2
     let Inst{5-4} = 0b00; // type
   }
   // shifted register
   def s : T2sOneRegShiftedReg<(outs rGPR:$Rd), (ins t2_so_reg:$ShiftedRm), iis,
                opc, ".w\t$Rd, $ShiftedRm",
                [(set rGPR:$Rd, (opnode t2_so_reg:$ShiftedRm))]>,
```
- EN: Defines TableGen record `r` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `r`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3009-3015
```tablegen
                Sched<[WriteALU]> {
     let Inst{31-27} = 0b11101;
     let Inst{26-25} = 0b01;
     let Inst{24-21} = opcod;
     let Inst{19-16} = 0b1111; // Rn
   }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3017-3021
```tablegen
// Prefer over of t2EORri ra, rb, -1 because mvn has 16-bit version
let AddedComplexity = 1 in
defm t2MVN  : T2I_un_irs <0b0011, "mvn",
                          IIC_iMVNi, IIC_iMVNr, IIC_iMVNsi,
                          not, 1, 1, 1>;
```
- EN: Defines TableGen record `t2MVN` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MVN`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3023-3025
```tablegen
let AddedComplexity = 1 in
def : T2Pat<(and     rGPR:$src, t2_so_imm_not:$imm),
            (t2BICri rGPR:$src, t2_so_imm_not:$imm)>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3027-3030
```tablegen
// so_imm_notSext is needed instead of so_imm_not, as the value of imm
// will match the extended, not the original bitWidth for $src.
def : T2Pat<(and top16Zero:$src, t2_so_imm_notSext:$imm),
            (t2BICri rGPR:$src, t2_so_imm_notSext:$imm)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3032-3035
```tablegen
// FIXME: Disable this pattern on Darwin to workaround an assembler bug.
def : T2Pat<(or      rGPR:$src, t2_so_imm_not:$imm),
            (t2ORNri rGPR:$src, t2_so_imm_not:$imm)>,
            Requires<[IsThumb2]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3037-3038
```tablegen
def : T2Pat<(t2_so_imm_not:$src),
            (t2MVNi t2_so_imm_not:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3040-3044
```tablegen
// There are shorter Thumb encodings for ADD than ORR, so to increase
// Thumb2SizeReduction's chances later on we select a t2ADD for an or where
// possible.
def : T2Pat<(or AddLikeOrOp:$Rn, t2_so_imm:$imm),
            (t2ADDri rGPR:$Rn, t2_so_imm:$imm)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3046-3047
```tablegen
def : T2Pat<(or AddLikeOrOp:$Rn, imm0_4095:$Rm),
            (t2ADDri12 rGPR:$Rn, imm0_4095:$Rm)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3049-3050
```tablegen
def : T2Pat<(or AddLikeOrOp:$Rn, non_imm32:$Rm),
            (t2ADDrr $Rn, $Rm)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3052-3065
```tablegen
//===----------------------------------------------------------------------===//
//  Multiply Instructions.
//
let isCommutable = 1 in
def t2MUL: T2ThreeReg<(outs rGPR:$Rd), (ins rGPR:$Rn, rGPR:$Rm), IIC_iMUL32,
                "mul", "\t$Rd, $Rn, $Rm",
                [(set rGPR:$Rd, (mul rGPR:$Rn, rGPR:$Rm))]>,
           Sched<[WriteMUL32, ReadMUL, ReadMUL]> {
  let Inst{31-27} = 0b11111;
  let Inst{26-23} = 0b0110;
  let Inst{22-20} = 0b000;
  let Inst{15-12} = 0b1111; // Ra = 0b1111 (no accumulate)
  let Inst{7-4} = 0b0000; // Multiply
}
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 3067-3076
```tablegen
class T2FourRegMLA<bits<4> op7_4, string opc, list<dag> pattern>
  : T2FourReg<(outs rGPR:$Rd), (ins rGPR:$Rn, rGPR:$Rm, rGPR:$Ra), IIC_iMAC32,
               opc, "\t$Rd, $Rn, $Rm, $Ra", pattern>,
               Requires<[IsThumb2, UseMulOps]>,
    Sched<[WriteMAC32, ReadMUL, ReadMUL, ReadMAC]>  {
  let Inst{31-27} = 0b11111;
  let Inst{26-23} = 0b0110;
  let Inst{22-20} = 0b000;
  let Inst{7-4} = op7_4;
}
```
- EN: Declares reusable TableGen class `T2FourRegMLA` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2FourRegMLA`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3078-3083
```tablegen
def t2MLA : T2FourRegMLA<0b0000, "mla",
                         [(set rGPR:$Rd, (add (mul rGPR:$Rn, rGPR:$Rm),
                                               rGPR:$Ra))]>;
def t2MLS: T2FourRegMLA<0b0001, "mls",
                        [(set rGPR:$Rd, (sub rGPR:$Ra, (mul rGPR:$Rn,
                                                            rGPR:$Rm)))]>;
```
- EN: Defines TableGen record `t2MLA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MLA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3085-3094
```tablegen
// Extra precision multiplies with low / high results
let hasSideEffects = 0 in {
let isCommutable = 1 in {
def t2SMULL : T2MulLong<0b000, 0b0000, "smull",
                        [(set rGPR:$RdLo, rGPR:$RdHi,
                              (smullohi rGPR:$Rn, rGPR:$Rm))]>;
def t2UMULL : T2MulLong<0b010, 0b0000, "umull",
                        [(set rGPR:$RdLo, rGPR:$RdHi,
                              (umullohi rGPR:$Rn, rGPR:$Rm))]>;
} // isCommutable
```
- EN: Defines TableGen record `t2SMULL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SMULL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3096-3100
```tablegen
// Multiply + accumulate
def t2SMLAL : T2MlaLong<0b100, 0b0000, "smlal">;
def t2UMLAL : T2MlaLong<0b110, 0b0000, "umlal">;
def t2UMAAL : T2MlaLong<0b110, 0b0110, "umaal">, Requires<[IsThumb2, HasDSP]>;
} // hasSideEffects
```
- EN: Defines TableGen record `t2SMLAL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SMLAL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3102-3102
```tablegen
// Rounding variants of the below included for disassembly only
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 3104-3121
```tablegen
// Most significant word multiply
class T2SMMUL<bits<4> op7_4, string opc, list<dag> pattern>
  : T2ThreeReg<(outs rGPR:$Rd),
               (ins rGPR:$Rn, rGPR:$Rm), IIC_iMUL32,
               opc, "\t$Rd, $Rn, $Rm", pattern>,
               Requires<[IsThumb2, HasDSP]>,
    Sched<[WriteMUL32, ReadMUL, ReadMUL]> {
  let Inst{31-27} = 0b11111;
  let Inst{26-23} = 0b0110;
  let Inst{22-20} = 0b101;
  let Inst{15-12} = 0b1111; // Ra = 0b1111 (no accumulate)
  let Inst{7-4} = op7_4;
}
def t2SMMUL : T2SMMUL<0b0000, "smmul", [(set rGPR:$Rd, (mulhs rGPR:$Rn,
                                                              rGPR:$Rm))]>;
def t2SMMULR :
  T2SMMUL<0b0001, "smmulr",
          [(set rGPR:$Rd, (ARMsmmlar rGPR:$Rn, rGPR:$Rm, (i32 0)))]>;
```
- EN: Declares reusable TableGen class `T2SMMUL` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2SMMUL`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3123-3132
```tablegen
class T2FourRegSMMLA<bits<3> op22_20, bits<4> op7_4, string opc,
                     list<dag> pattern>
  : T2FourReg<(outs rGPR:$Rd), (ins rGPR:$Rn, rGPR:$Rm, rGPR:$Ra), IIC_iMAC32,
              opc, "\t$Rd, $Rn, $Rm, $Ra", pattern>,
              Requires<[IsThumb2, HasDSP, UseMulOps]>,
    Sched<[WriteMAC32, ReadMUL, ReadMUL, ReadMAC]> {
  let Inst{31-27} = 0b11111;
  let Inst{26-23} = 0b0110;
  let Inst{22-20} = op22_20;
  let Inst{7-4} = op7_4;
```
- EN: Declares reusable TableGen class `T2FourRegSMMLA` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2FourRegSMMLA`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3134-3135
```tablegen
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3137-3143
```tablegen
def t2SMMLA :   T2FourRegSMMLA<0b101, 0b0000, "smmla",
                [(set rGPR:$Rd, (add (mulhs rGPR:$Rm, rGPR:$Rn), rGPR:$Ra))]>;
def t2SMMLAR:   T2FourRegSMMLA<0b101, 0b0001, "smmlar",
                [(set rGPR:$Rd, (ARMsmmlar rGPR:$Rn, rGPR:$Rm, rGPR:$Ra))]>;
def t2SMMLS:    T2FourRegSMMLA<0b110, 0b0000, "smmls", []>;
def t2SMMLSR:   T2FourRegSMMLA<0b110, 0b0001, "smmlsr",
                [(set rGPR:$Rd, (ARMsmmlsr rGPR:$Rn, rGPR:$Rm, rGPR:$Ra))]>;
```
- EN: Defines TableGen record `t2SMMLA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SMMLA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3145-3157
```tablegen
class T2ThreeRegSMUL<bits<3> op22_20, bits<2> op5_4, string opc,
                     list<dag> pattern>
  : T2ThreeReg<(outs rGPR:$Rd), (ins rGPR:$Rn, rGPR:$Rm), IIC_iMUL16, opc,
               "\t$Rd, $Rn, $Rm", pattern>,
    Requires<[IsThumb2, HasDSP]>,
    Sched<[WriteMUL16, ReadMUL, ReadMUL]> {
    let Inst{31-27} = 0b11111;
    let Inst{26-23} = 0b0110;
    let Inst{22-20} = op22_20;
    let Inst{15-12} = 0b1111; // Ra = 0b1111 (no accumulate)
    let Inst{7-6} = 0b00;
    let Inst{5-4} = op5_4;
}
```
- EN: Declares reusable TableGen class `T2ThreeRegSMUL` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2ThreeRegSMUL`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3159-3170
```tablegen
def t2SMULBB : T2ThreeRegSMUL<0b001, 0b00, "smulbb",
             [(set rGPR:$Rd, (bb_mul rGPR:$Rn, rGPR:$Rm))]>;
def t2SMULBT : T2ThreeRegSMUL<0b001, 0b01, "smulbt",
             [(set rGPR:$Rd, (bt_mul rGPR:$Rn, rGPR:$Rm))]>;
def t2SMULTB : T2ThreeRegSMUL<0b001, 0b10, "smultb",
             [(set rGPR:$Rd, (tb_mul rGPR:$Rn, rGPR:$Rm))]>;
def t2SMULTT : T2ThreeRegSMUL<0b001, 0b11, "smultt",
             [(set rGPR:$Rd, (tt_mul rGPR:$Rn, rGPR:$Rm))]>;
def t2SMULWB : T2ThreeRegSMUL<0b011, 0b00, "smulwb",
             [(set rGPR:$Rd, (ARMsmulwb rGPR:$Rn, rGPR:$Rm))]>;
def t2SMULWT : T2ThreeRegSMUL<0b011, 0b01, "smulwt",
             [(set rGPR:$Rd, (ARMsmulwt rGPR:$Rn, rGPR:$Rm))]>;
```
- EN: Defines TableGen record `t2SMULBB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SMULBB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3172-3177
```tablegen
def : Thumb2DSPPat<(mul sext_16_node:$Rn, (sext_bottom_16 rGPR:$Rm)),
                   (t2SMULBB rGPR:$Rn, rGPR:$Rm)>;
def : Thumb2DSPPat<(mul sext_16_node:$Rn, (sext_top_16 rGPR:$Rm)),
                   (t2SMULBT rGPR:$Rn, rGPR:$Rm)>;
def : Thumb2DSPPat<(mul (sext_top_16 rGPR:$Rn), sext_16_node:$Rm),
                   (t2SMULTB rGPR:$Rn, rGPR:$Rm)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3179-3190
```tablegen
def : Thumb2DSPPat<(int_arm_smulbb rGPR:$Rn, rGPR:$Rm),
                   (t2SMULBB rGPR:$Rn, rGPR:$Rm)>;
def : Thumb2DSPPat<(int_arm_smulbt rGPR:$Rn, rGPR:$Rm),
                   (t2SMULBT rGPR:$Rn, rGPR:$Rm)>;
def : Thumb2DSPPat<(int_arm_smultb rGPR:$Rn, rGPR:$Rm),
                   (t2SMULTB rGPR:$Rn, rGPR:$Rm)>;
def : Thumb2DSPPat<(int_arm_smultt rGPR:$Rn, rGPR:$Rm),
                   (t2SMULTT rGPR:$Rn, rGPR:$Rm)>;
def : Thumb2DSPPat<(int_arm_smulwb rGPR:$Rn, rGPR:$Rm),
                   (t2SMULWB rGPR:$Rn, rGPR:$Rm)>;
def : Thumb2DSPPat<(int_arm_smulwt rGPR:$Rn, rGPR:$Rm),
                   (t2SMULWT rGPR:$Rn, rGPR:$Rm)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3192-3203
```tablegen
class T2FourRegSMLA<bits<3> op22_20, bits<2> op5_4, string opc,
                    list<dag> pattern>
  : T2FourReg<(outs rGPR:$Rd), (ins rGPR:$Rn, rGPR:$Rm, rGPR:$Ra), IIC_iMUL16,
               opc, "\t$Rd, $Rn, $Rm, $Ra", pattern>,
    Requires<[IsThumb2, HasDSP, UseMulOps]>,
    Sched<[WriteMAC16, ReadMUL, ReadMUL, ReadMAC]>  {
    let Inst{31-27} = 0b11111;
    let Inst{26-23} = 0b0110;
    let Inst{22-20} = op22_20;
    let Inst{7-6} = 0b00;
    let Inst{5-4} = op5_4;
}
```
- EN: Declares reusable TableGen class `T2FourRegSMLA` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2FourRegSMLA`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3205-3216
```tablegen
def t2SMLABB : T2FourRegSMLA<0b001, 0b00, "smlabb",
             [(set rGPR:$Rd, (add rGPR:$Ra, (bb_mul rGPR:$Rn, rGPR:$Rm)))]>;
def t2SMLABT : T2FourRegSMLA<0b001, 0b01, "smlabt",
             [(set rGPR:$Rd, (add rGPR:$Ra, (bt_mul rGPR:$Rn, rGPR:$Rm)))]>;
def t2SMLATB : T2FourRegSMLA<0b001, 0b10, "smlatb",
             [(set rGPR:$Rd, (add rGPR:$Ra, (tb_mul rGPR:$Rn, rGPR:$Rm)))]>;
def t2SMLATT : T2FourRegSMLA<0b001, 0b11, "smlatt",
             [(set rGPR:$Rd, (add rGPR:$Ra, (tt_mul rGPR:$Rn, rGPR:$Rm)))]>;
def t2SMLAWB : T2FourRegSMLA<0b011, 0b00, "smlawb",
             [(set rGPR:$Rd, (add rGPR:$Ra, (ARMsmulwb rGPR:$Rn, rGPR:$Rm)))]>;
def t2SMLAWT : T2FourRegSMLA<0b011, 0b01, "smlawt",
             [(set rGPR:$Rd, (add rGPR:$Ra, (ARMsmulwt rGPR:$Rn, rGPR:$Rm)))]>;
```
- EN: Defines TableGen record `t2SMLABB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SMLABB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3218-3228
```tablegen
def : Thumb2DSPMulPat<(add rGPR:$Ra, (mul sext_16_node:$Rn, sext_16_node:$Rm)),
                      (t2SMLABB rGPR:$Rn, rGPR:$Rm, rGPR:$Ra)>;
def : Thumb2DSPMulPat<(add rGPR:$Ra, (mul sext_16_node:$Rn, 
                                          (sext_bottom_16 rGPR:$Rm))),
                      (t2SMLABB rGPR:$Rn, rGPR:$Rm, rGPR:$Ra)>;
def : Thumb2DSPMulPat<(add rGPR:$Ra, (mul sext_16_node:$Rn,
                                          (sext_top_16 rGPR:$Rm))),
                      (t2SMLABT rGPR:$Rn, rGPR:$Rm, rGPR:$Ra)>;
def : Thumb2DSPMulPat<(add rGPR:$Ra, (mul (sext_top_16 rGPR:$Rn),
                                          sext_16_node:$Rm)),
                      (t2SMLATB rGPR:$Rn, rGPR:$Rm, rGPR:$Ra)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3230-3241
```tablegen
def : Thumb2DSPPat<(int_arm_smlabb GPR:$a, GPR:$b, GPR:$acc),
                   (t2SMLABB GPR:$a, GPR:$b, GPR:$acc)>;
def : Thumb2DSPPat<(int_arm_smlabt GPR:$a, GPR:$b, GPR:$acc),
                   (t2SMLABT GPR:$a, GPR:$b, GPR:$acc)>;
def : Thumb2DSPPat<(int_arm_smlatb GPR:$a, GPR:$b, GPR:$acc),
                   (t2SMLATB GPR:$a, GPR:$b, GPR:$acc)>;
def : Thumb2DSPPat<(int_arm_smlatt GPR:$a, GPR:$b, GPR:$acc),
                   (t2SMLATT GPR:$a, GPR:$b, GPR:$acc)>;
def : Thumb2DSPPat<(int_arm_smlawb GPR:$a, GPR:$b, GPR:$acc),
                   (t2SMLAWB GPR:$a, GPR:$b, GPR:$acc)>;
def : Thumb2DSPPat<(int_arm_smlawt GPR:$a, GPR:$b, GPR:$acc),
                   (t2SMLAWT GPR:$a, GPR:$b, GPR:$acc)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3243-3251
```tablegen
// Halfword multiple accumulate long: SMLAL<x><y>
def t2SMLALBB : T2MlaLong<0b100, 0b1000, "smlalbb">,
                          Requires<[IsThumb2, HasDSP]>;
def t2SMLALBT : T2MlaLong<0b100, 0b1001, "smlalbt">,
                          Requires<[IsThumb2, HasDSP]>;
def t2SMLALTB : T2MlaLong<0b100, 0b1010, "smlaltb">,
                          Requires<[IsThumb2, HasDSP]>;
def t2SMLALTT : T2MlaLong<0b100, 0b1011, "smlaltt">,
                          Requires<[IsThumb2, HasDSP]>;
```
- EN: Defines TableGen record `t2SMLALBB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SMLALBB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3253-3260
```tablegen
def : Thumb2DSPPat<(ARMsmlalbb GPR:$Rn, GPR:$Rm, GPR:$RLo, GPR:$RHi),
                   (t2SMLALBB $Rn, $Rm, $RLo, $RHi)>;
def : Thumb2DSPPat<(ARMsmlalbt GPR:$Rn, GPR:$Rm, GPR:$RLo, GPR:$RHi),
                   (t2SMLALBT $Rn, $Rm, $RLo, $RHi)>;
def : Thumb2DSPPat<(ARMsmlaltb GPR:$Rn, GPR:$Rm, GPR:$RLo, GPR:$RHi),
                   (t2SMLALTB $Rn, $Rm, $RLo, $RHi)>;
def : Thumb2DSPPat<(ARMsmlaltt GPR:$Rn, GPR:$Rm, GPR:$RLo, GPR:$RHi),
                   (t2SMLALTT $Rn, $Rm, $RLo, $RHi)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3262-3272
```tablegen
class T2DualHalfMul<bits<3> op22_20, bits<4> op7_4, string opc,
                    Intrinsic intrinsic>
  : T2ThreeReg_mac<0, op22_20, op7_4,
                   (outs rGPR:$Rd),
                   (ins rGPR:$Rn, rGPR:$Rm),
                   IIC_iMAC32, opc, "\t$Rd, $Rn, $Rm",
                   [(set rGPR:$Rd, (intrinsic rGPR:$Rn, rGPR:$Rm))]>,
                   Requires<[IsThumb2, HasDSP]>,
   Sched<[WriteMAC32, ReadMUL, ReadMUL, ReadMAC]> {
  let Inst{15-12} = 0b1111;
}
```
- EN: Declares reusable TableGen class `T2DualHalfMul` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2DualHalfMul`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3274-3278
```tablegen
// Dual halfword multiple: SMUAD, SMUSD, SMLAD, SMLSD, SMLALD, SMLSLD
def t2SMUAD: T2DualHalfMul<0b010, 0b0000, "smuad", int_arm_smuad>;
def t2SMUADX: T2DualHalfMul<0b010, 0b0001, "smuadx", int_arm_smuadx>;
def t2SMUSD: T2DualHalfMul<0b100, 0b0000, "smusd", int_arm_smusd>;
def t2SMUSDX: T2DualHalfMul<0b100, 0b0001, "smusdx", int_arm_smusdx>;
```
- EN: Defines TableGen record `t2SMUAD:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SMUAD:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3280-3287
```tablegen
class T2DualHalfMulAdd<bits<3> op22_20, bits<4> op7_4, string opc,
                       Intrinsic intrinsic>
  : T2FourReg_mac<0, op22_20, op7_4,
                  (outs rGPR:$Rd),
                  (ins rGPR:$Rn, rGPR:$Rm, rGPR:$Ra),
                  IIC_iMAC32, opc, "\t$Rd, $Rn, $Rm, $Ra",
                  [(set rGPR:$Rd, (intrinsic rGPR:$Rn, rGPR:$Rm, rGPR:$Ra))]>,
                  Requires<[IsThumb2, HasDSP]>;
```
- EN: Declares reusable TableGen class `T2DualHalfMulAdd` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2DualHalfMulAdd`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3289-3292
```tablegen
def t2SMLAD   : T2DualHalfMulAdd<0b010, 0b0000, "smlad", int_arm_smlad>;
def t2SMLADX  : T2DualHalfMulAdd<0b010, 0b0001, "smladx", int_arm_smladx>;
def t2SMLSD   : T2DualHalfMulAdd<0b100, 0b0000, "smlsd", int_arm_smlsd>;
def t2SMLSDX  : T2DualHalfMulAdd<0b100, 0b0001, "smlsdx", int_arm_smlsdx>;
```
- EN: Defines TableGen record `t2SMLAD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SMLAD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3294-3301
```tablegen
class T2DualHalfMulAddLong<bits<3> op22_20, bits<4> op7_4, string opc>
  : T2FourReg_mac<1, op22_20, op7_4,
                  (outs rGPR:$Ra, rGPR:$Rd),
                  (ins rGPR:$Rn, rGPR:$Rm, rGPR:$RLo, rGPR:$RHi),
                  IIC_iMAC64, opc, "\t$Ra, $Rd, $Rn, $Rm", []>,
                  RegConstraint<"$Ra = $RLo, $Rd = $RHi">,
                  Requires<[IsThumb2, HasDSP]>,
    Sched<[WriteMAC64Lo, WriteMAC64Hi, ReadMUL, ReadMUL, ReadMAC, ReadMAC]>;
```
- EN: Declares reusable TableGen class `T2DualHalfMulAddLong` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2DualHalfMulAddLong`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3303-3306
```tablegen
def t2SMLALD  : T2DualHalfMulAddLong<0b100, 0b1100, "smlald">;
def t2SMLALDX : T2DualHalfMulAddLong<0b100, 0b1101, "smlaldx">;
def t2SMLSLD  : T2DualHalfMulAddLong<0b101, 0b1100, "smlsld">;
def t2SMLSLDX : T2DualHalfMulAddLong<0b101, 0b1101, "smlsldx">;
```
- EN: Defines TableGen record `t2SMLALD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SMLALD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3308-3315
```tablegen
def : Thumb2DSPPat<(ARMSmlald rGPR:$Rn, rGPR:$Rm, rGPR:$RLo, rGPR:$RHi),
                   (t2SMLALD rGPR:$Rn, rGPR:$Rm, rGPR:$RLo, rGPR:$RHi)>;
def : Thumb2DSPPat<(ARMSmlaldx rGPR:$Rn, rGPR:$Rm, rGPR:$RLo, rGPR:$RHi),
                   (t2SMLALDX rGPR:$Rn, rGPR:$Rm, rGPR:$RLo, rGPR:$RHi)>;
def : Thumb2DSPPat<(ARMSmlsld rGPR:$Rn, rGPR:$Rm, rGPR:$RLo, rGPR:$RHi),
                   (t2SMLSLD rGPR:$Rn, rGPR:$Rm, rGPR:$RLo, rGPR:$RHi)>;
def : Thumb2DSPPat<(ARMSmlsldx rGPR:$Rn, rGPR:$Rm, rGPR:$RLo, rGPR:$RHi),
                   (t2SMLSLDX rGPR:$Rn, rGPR:$Rm, rGPR:$RLo, rGPR:$RHi)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3317-3332
```tablegen
//===----------------------------------------------------------------------===//
//  Division Instructions.
//  Signed and unsigned division on v7-M
//
let TwoOperandAliasConstraint = "$Rn = $Rd" in {
def t2SDIV : T2ThreeReg<(outs rGPR:$Rd), (ins rGPR:$Rn, rGPR:$Rm), IIC_iDIV,
                 "sdiv", "\t$Rd, $Rn, $Rm",
                 [(set rGPR:$Rd, (sdiv rGPR:$Rn, rGPR:$Rm))]>,
                 Requires<[HasDivideInThumb, IsThumb, HasV8MBaseline]>,
             Sched<[WriteDIV]> {
  let Inst{31-27} = 0b11111;
  let Inst{26-21} = 0b011100;
  let Inst{20} = 0b1;
  let Inst{15-12} = 0b1111;
  let Inst{7-4} = 0b1111;
}
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 3334-3345
```tablegen
def t2UDIV : T2ThreeReg<(outs rGPR:$Rd), (ins rGPR:$Rn, rGPR:$Rm), IIC_iDIV,
                 "udiv", "\t$Rd, $Rn, $Rm",
                 [(set rGPR:$Rd, (udiv rGPR:$Rn, rGPR:$Rm))]>,
                 Requires<[HasDivideInThumb, IsThumb, HasV8MBaseline]>,
             Sched<[WriteDIV]> {
  let Inst{31-27} = 0b11111;
  let Inst{26-21} = 0b011101;
  let Inst{20} = 0b1;
  let Inst{15-12} = 0b1111;
  let Inst{7-4} = 0b1111;
}
}
```
- EN: Defines TableGen record `t2UDIV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2UDIV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3347-3349
```tablegen
//===----------------------------------------------------------------------===//
//  Misc. Arithmetic Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 3351-3361
```tablegen
class T2I_misc<bits<2> op1, bits<2> op2, dag oops, dag iops,
      InstrItinClass itin, string opc, string asm, list<dag> pattern>
  : T2ThreeReg<oops, iops, itin, opc, asm, pattern> {
  let Inst{31-27} = 0b11111;
  let Inst{26-22} = 0b01010;
  let Inst{21-20} = op1;
  let Inst{15-12} = 0b1111;
  let Inst{7-6} = 0b10;
  let Inst{5-4} = op2;
  let Rn{3-0} = Rm;
}
```
- EN: Declares reusable TableGen class `T2I_misc` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2I_misc`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3363-3365
```tablegen
def t2CLZ : T2I_misc<0b11, 0b00, (outs rGPR:$Rd), (ins rGPR:$Rm), IIC_iUNAr,
                    "clz", "\t$Rd, $Rm", [(set rGPR:$Rd, (ctlz rGPR:$Rm))]>,
                    Sched<[WriteALU]>;
```
- EN: Defines TableGen record `t2CLZ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2CLZ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3367-3370
```tablegen
def t2RBIT : T2I_misc<0b01, 0b10, (outs rGPR:$Rd), (ins rGPR:$Rm), IIC_iUNAr,
                      "rbit", "\t$Rd, $Rm",
                      [(set rGPR:$Rd, (bitreverse rGPR:$Rm))]>,
                      Sched<[WriteALU]>;
```
- EN: Defines TableGen record `t2RBIT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2RBIT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3372-3374
```tablegen
def t2REV : T2I_misc<0b01, 0b00, (outs rGPR:$Rd), (ins rGPR:$Rm), IIC_iUNAr,
                 "rev", ".w\t$Rd, $Rm", [(set rGPR:$Rd, (bswap rGPR:$Rm))]>,
                 Sched<[WriteALU]>;
```
- EN: Defines TableGen record `t2REV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2REV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3376-3379
```tablegen
def t2REV16 : T2I_misc<0b01, 0b01, (outs rGPR:$Rd), (ins rGPR:$Rm), IIC_iUNAr,
                       "rev16", ".w\t$Rd, $Rm",
                [(set rGPR:$Rd, (rotr (bswap rGPR:$Rm), (i32 16)))]>,
                Sched<[WriteALU]>;
```
- EN: Defines TableGen record `t2REV16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2REV16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3381-3382
```tablegen
def : T2Pat<(srl (bswap top16Zero:$Rn), (i32 16)),
            (t2REV16 rGPR:$Rn)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3384-3387
```tablegen
def t2REVSH : T2I_misc<0b01, 0b11, (outs rGPR:$Rd), (ins rGPR:$Rm), IIC_iUNAr,
                       "revsh", ".w\t$Rd, $Rm",
                 [(set rGPR:$Rd, (sra (bswap rGPR:$Rm), (i32 16)))]>,
                 Sched<[WriteALU]>;
```
- EN: Defines TableGen record `t2REVSH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2REVSH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3389-3391
```tablegen
def : T2Pat<(or (sra (shl rGPR:$Rm, (i32 24)), (i32 16)),
                (and (srl rGPR:$Rm, (i32 8)), 0xFF)),
            (t2REVSH rGPR:$Rm)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3393-3405
```tablegen
def t2PKHBT : T2ThreeReg<
            (outs rGPR:$Rd), (ins rGPR:$Rn, rGPR:$Rm, pkh_lsl_amt:$sh),
                  IIC_iBITsi, "pkhbt", "\t$Rd, $Rn, $Rm$sh",
                  [(set rGPR:$Rd, (or (and rGPR:$Rn, 0xFFFF),
                                      (and (shl rGPR:$Rm, pkh_lsl_amt:$sh),
                                           0xFFFF0000)))]>,
                  Requires<[HasDSP, IsThumb2]>,
                  Sched<[WriteALUsi, ReadALU]> {
  let Inst{31-27} = 0b11101;
  let Inst{26-25} = 0b01;
  let Inst{24-20} = 0b01100;
  let Inst{5} = 0; // BT form
  let Inst{4} = 0;
```
- EN: Defines TableGen record `t2PKHBT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2PKHBT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3407-3410
```tablegen
  bits<5> sh;
  let Inst{14-12} = sh{4-2};
  let Inst{7-6}   = sh{1-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3412-3418
```tablegen
// Alternate cases for PKHBT where identities eliminate some nodes.
def : T2Pat<(or (and rGPR:$src1, 0xFFFF), (and rGPR:$src2, 0xFFFF0000)),
            (t2PKHBT rGPR:$src1, rGPR:$src2, 0)>,
            Requires<[HasDSP, IsThumb2]>;
def : T2Pat<(or (and rGPR:$src1, 0xFFFF), (shl rGPR:$src2, imm16_31:$sh)),
            (t2PKHBT rGPR:$src1, rGPR:$src2, imm16_31:$sh)>,
            Requires<[HasDSP, IsThumb2]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3420-3434
```tablegen
// Note: Shifts of 1-15 bits will be transformed to srl instead of sra and
// will match the pattern below.
def t2PKHTB : T2ThreeReg<
                  (outs rGPR:$Rd), (ins rGPR:$Rn, rGPR:$Rm, pkh_asr_amt:$sh),
                  IIC_iBITsi, "pkhtb", "\t$Rd, $Rn, $Rm$sh",
                  [(set rGPR:$Rd, (or (and rGPR:$Rn, 0xFFFF0000),
                                       (and (sra rGPR:$Rm, pkh_asr_amt:$sh),
                                            0xFFFF)))]>,
                  Requires<[HasDSP, IsThumb2]>,
                  Sched<[WriteALUsi, ReadALU]> {
  let Inst{31-27} = 0b11101;
  let Inst{26-25} = 0b01;
  let Inst{24-20} = 0b01100;
  let Inst{5} = 1; // TB form
  let Inst{4} = 0;
```
- EN: Defines TableGen record `t2PKHTB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2PKHTB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3436-3439
```tablegen
  bits<5> sh;
  let Inst{14-12} = sh{4-2};
  let Inst{7-6}   = sh{1-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3441-3454
```tablegen
// Alternate cases for PKHTB where identities eliminate some nodes.  Note that
// a shift amount of 0 is *not legal* here, it is PKHBT instead.
// We also can not replace a srl (17..31) by an arithmetic shift we would use in
// pkhtb src1, src2, asr (17..31).
def : T2Pat<(or (and rGPR:$src1, 0xFFFF0000), (srl rGPR:$src2, imm16:$sh)),
            (t2PKHTB rGPR:$src1, rGPR:$src2, imm16:$sh)>,
            Requires<[HasDSP, IsThumb2]>;
def : T2Pat<(or (and rGPR:$src1, 0xFFFF0000), (sra rGPR:$src2, imm16_31:$sh)),
            (t2PKHTB rGPR:$src1, rGPR:$src2, imm16_31:$sh)>,
            Requires<[HasDSP, IsThumb2]>;
def : T2Pat<(or (and rGPR:$src1, 0xFFFF0000),
                (and (srl rGPR:$src2, imm1_15:$sh), 0xFFFF)),
            (t2PKHTB rGPR:$src1, rGPR:$src2, imm1_15:$sh)>,
            Requires<[HasDSP, IsThumb2]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3456-3462
```tablegen
//===----------------------------------------------------------------------===//
// CRC32 Instructions
//
// Polynomials:
// + CRC32{B,H,W}       0x04C11DB7
// + CRC32C{B,H,W}      0x1EDC6F41
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 3464-3475
```tablegen
class T2I_crc32<bit C, bits<2> sz, string suffix, SDPatternOperator builtin>
  : T2ThreeRegNoP<(outs rGPR:$Rd), (ins rGPR:$Rn, rGPR:$Rm), NoItinerary,
               !strconcat("crc32", suffix, "\t$Rd, $Rn, $Rm"),
               [(set rGPR:$Rd, (builtin rGPR:$Rn, rGPR:$Rm))]>,
               Requires<[IsThumb2, HasCRC]> {
  let Inst{31-27} = 0b11111;
  let Inst{26-21} = 0b010110;
  let Inst{20}    = C;
  let Inst{15-12} = 0b1111;
  let Inst{7-6}   = 0b10;
  let Inst{5-4}   = sz;
}
```
- EN: Declares reusable TableGen class `T2I_crc32` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2I_crc32`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3477-3482
```tablegen
def t2CRC32B  : T2I_crc32<0, 0b00, "b", int_arm_crc32b>;
def t2CRC32CB : T2I_crc32<1, 0b00, "cb", int_arm_crc32cb>;
def t2CRC32H  : T2I_crc32<0, 0b01, "h", int_arm_crc32h>;
def t2CRC32CH : T2I_crc32<1, 0b01, "ch", int_arm_crc32ch>;
def t2CRC32W  : T2I_crc32<0, 0b10, "w", int_arm_crc32w>;
def t2CRC32CW : T2I_crc32<1, 0b10, "cw", int_arm_crc32cw>;
```
- EN: Defines TableGen record `t2CRC32B` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2CRC32B`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3484-3488
```tablegen
//===----------------------------------------------------------------------===//
//  Comparison Instructions...
//
defm t2CMP  : T2I_cmp_irs<0b1101, "cmp", GPRnopc,
                          IIC_iCMPi, IIC_iCMPr, IIC_iCMPsi, ARMcmp>;
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 3490-3495
```tablegen
def : T2Pat<(ARMcmpZ  GPRnopc:$lhs, t2_so_imm:$imm),
            (t2CMPri  GPRnopc:$lhs, t2_so_imm:$imm)>;
def : T2Pat<(ARMcmpZ  GPRnopc:$lhs, rGPR:$rhs),
            (t2CMPrr  GPRnopc:$lhs, rGPR:$rhs)>;
def : T2Pat<(ARMcmpZ  GPRnopc:$lhs, t2_so_reg_oneuse:$rhs),
            (t2CMPrs  GPRnopc:$lhs, t2_so_reg_oneuse:$rhs)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3497-3514
```tablegen
let isCompare = 1, Defs = [CPSR] in {
   // shifted imm
   def t2CMNri : T2OneRegCmpImm<
                (outs), (ins GPRnopc:$Rn, t2_so_imm:$imm), IIC_iCMPi,
                "cmn", ".w\t$Rn, $imm",
                [(set CPSR, (ARMcmn GPRnopc:$Rn, (ineg t2_so_imm:$imm)))]>,
                Sched<[WriteCMP, ReadALU]> {
     let Inst{31-27} = 0b11110;
     let Inst{25} = 0;
     let Inst{24-21} = 0b1000;
     let Inst{20} = 1; // The S bit.
     let Inst{15} = 0;
     let Inst{11-8} = 0b1111; // Rd
   }
   // register
   def t2CMNrr : T2TwoRegCmp<
                (outs), (ins GPRnopc:$Rn, rGPR:$Rm), IIC_iCMPr,
                "cmn", ".w\t$Rn, $Rm",
```
- EN: Defines TableGen record `t2CMNri` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2CMNri`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3515-3532
```tablegen
                [(set CPSR, (ARMcmn GPRnopc:$Rn, rGPR:$Rm))]>,
                Sched<[WriteCMP, ReadALU, ReadALU]> {
     let Inst{31-27} = 0b11101;
     let Inst{26-25} = 0b01;
     let Inst{24-21} = 0b1000;
     let Inst{20} = 1; // The S bit.
     let Inst{14-12} = 0b000; // imm3
     let Inst{11-8} = 0b1111; // Rd
     let Inst{7-6} = 0b00; // imm2
     let Inst{5-4} = 0b00; // type
   }
   // shifted register
   def t2CMNrs : T2OneRegCmpShiftedReg<
                (outs), (ins GPRnopc:$Rn, t2_so_reg:$ShiftedRm), IIC_iCMPsi,
                "cmn", ".w\t$Rn, $ShiftedRm",
                [(set CPSR, (ARMcmn GPRnopc:$Rn, t2_so_reg:$ShiftedRm))]>,
                Sched<[WriteCMPsi, ReadALU, ReadALU]> {
     let Inst{31-27} = 0b11101;
```
- EN: Defines TableGen record `t2CMNrs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2CMNrs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3533-3538
```tablegen
     let Inst{26-25} = 0b01;
     let Inst{24-21} = 0b1000;
     let Inst{20} = 1; // The S bit.
     let Inst{11-8} = 0b1111; // Rd
   }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3540-3546
```tablegen
// Assembler aliases w/o the ".w" suffix.
// No alias here for 'rr' version as not all instantiations of this multiclass
// want one (CMP in particular, does not).
def : t2InstAlias<"cmn${p} $Rn, $imm",
   (t2CMNri GPRnopc:$Rn, t2_so_imm:$imm, pred:$p)>;
def : t2InstAlias<"cmn${p} $Rn, $shift",
   (t2CMNrs GPRnopc:$Rn, t2_so_reg:$shift, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3548-3549
```tablegen
def : T2Pat<(ARMcmp  GPR:$src, t2_so_imm_neg:$imm),
            (t2CMNri GPR:$src, t2_so_imm_neg:$imm)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3551-3552
```tablegen
def : T2Pat<(ARMcmpZ GPRnopc:$src, t2_so_imm_neg:$imm),
            (t2CMNri GPRnopc:$src, t2_so_imm_neg:$imm)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3554-3558
```tablegen
// Fold compare-to-zero of a negated register into CMN register forms.
def : T2Pat<(ARMcmpZ GPRnopc:$Rn, (ineg rGPR:$Rm)),
            (t2CMNrr GPRnopc:$Rn, rGPR:$Rm)>;
def : T2Pat<(ARMcmpZ GPRnopc:$Rn, (ineg t2_so_reg:$ShiftedRm)),
            (t2CMNrs GPRnopc:$Rn, t2_so_reg:$ShiftedRm)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3560-3565
```tablegen
defm t2TST  : T2I_cmp_irs<0b0000, "tst", rGPR,
                          IIC_iTSTi, IIC_iTSTr, IIC_iTSTsi,
                         BinOpFrag<(ARMcmpZ (and_su node:$LHS, node:$RHS), 0)>>;
defm t2TEQ  : T2I_cmp_irs<0b0100, "teq", rGPR,
                          IIC_iTSTi, IIC_iTSTr, IIC_iTSTsi,
                         BinOpFrag<(ARMcmpZ (xor_su node:$LHS, node:$RHS), 0)>>;
```
- EN: Defines TableGen record `t2TST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2TST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3567-3568
```tablegen
// Conditional moves
let hasSideEffects = 0 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3570-3574
```tablegen
let isCommutable = 1, isSelect = 1 in
def t2MOVCCr : t2PseudoInst<(outs rGPR:$Rd),
                            (ins rGPR:$false, rGPR:$Rm, pred:$p),
                            4, IIC_iCMOVr, []>,
               RegConstraint<"$false = $Rd">, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `t2MOVCCr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MOVCCr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3576-3581
```tablegen
let isMoveImm = 1 in
def t2MOVCCi
    : t2PseudoInst<(outs rGPR:$Rd),
                   (ins rGPR:$false, t2_so_imm:$imm, pred:$p),
                   4, IIC_iCMOVi, []>,
      RegConstraint<"$false = $Rd">, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `t2MOVCCi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MOVCCi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3583-3589
```tablegen
let isCodeGenOnly = 1 in {
let isMoveImm = 1 in
def t2MOVCCi16
    : t2PseudoInst<(outs rGPR:$Rd),
                   (ins rGPR:$false, imm0_65535_expr:$imm, pred:$p),
                   4, IIC_iCMOVi, []>,
      RegConstraint<"$false = $Rd">, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `t2MOVCCi16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MOVCCi16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3591-3596
```tablegen
let isMoveImm = 1 in
def t2MVNCCi
    : t2PseudoInst<(outs rGPR:$Rd),
                   (ins rGPR:$false, t2_so_imm:$imm, pred:$p),
                   4, IIC_iCMOVi, []>,
      RegConstraint<"$false = $Rd">, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `t2MVNCCi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MVNCCi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3598-3602
```tablegen
class MOVCCShPseudo
    : t2PseudoInst<(outs rGPR:$Rd),
                   (ins rGPR:$false, rGPR:$Rm, i32imm:$imm, pred:$p),
                   4, IIC_iCMOVsi, []>,
      RegConstraint<"$false = $Rd">, Sched<[WriteALU]>;
```
- EN: Declares reusable TableGen class `MOVCCShPseudo` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `MOVCCShPseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3604-3607
```tablegen
def t2MOVCClsl : MOVCCShPseudo;
def t2MOVCClsr : MOVCCShPseudo;
def t2MOVCCasr : MOVCCShPseudo;
def t2MOVCCror : MOVCCShPseudo;
```
- EN: Defines TableGen record `t2MOVCClsl` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MOVCClsl`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3609-3615
```tablegen
let isMoveImm = 1 in
def t2MOVCCi32imm
    : t2PseudoInst<(outs rGPR:$dst),
                   (ins rGPR:$false, i32imm:$src, pred:$p),
                   8, IIC_iCMOVix2, []>,
      RegConstraint<"$false = $dst">;
} // isCodeGenOnly = 1
```
- EN: Defines TableGen record `t2MOVCCi32imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MOVCCi32imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3617-3617
```tablegen
} // hasSideEffects
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3619-3621
```tablegen
// The following patterns have to be defined out-of-line because the number
// of instruction operands does not match the number of SDNode operands
// (`pred` counts as one operand).
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 3623-3624
```tablegen
def : T2Pat<(ARMcmov i32:$false, i32:$Rm, imm:$cc, CPSR),
            (t2MOVCCr $false, $Rm, imm:$cc, CPSR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3626-3627
```tablegen
def : T2Pat<(ARMcmov i32:$false, t2_so_imm:$imm, imm:$cc, CPSR),
            (t2MOVCCi $false, t2_so_imm:$imm, imm:$cc, CPSR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3629-3630
```tablegen
def : T2Pat<(ARMcmov i32:$false, imm0_65535:$imm, imm:$cc, CPSR),
            (t2MOVCCi16 $false, imm0_65535:$imm, imm:$cc, CPSR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3632-3633
```tablegen
def : T2Pat<(ARMcmov i32:$false, t2_so_imm_not:$imm, imm:$cc, CPSR),
            (t2MVNCCi $false, t2_so_imm_not:$imm, imm:$cc, CPSR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3635-3636
```tablegen
def : T2Pat<(ARMcmov i32:$false, (shl i32:$Rm, imm0_31:$imm), imm:$cc, CPSR),
            (t2MOVCClsl $false, $Rm, imm0_31:$imm, imm:$cc, CPSR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3638-3639
```tablegen
def : T2Pat<(ARMcmov i32:$false, (srl i32:$Rm, imm_sr:$imm), imm:$cc, CPSR),
            (t2MOVCClsr $false, $Rm, imm_sr:$imm, imm:$cc, CPSR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3641-3642
```tablegen
def : T2Pat<(ARMcmov i32:$false, (sra i32:$Rm, imm_sr:$imm), imm:$cc, CPSR),
            (t2MOVCCasr $false, $Rm, imm_sr:$imm, imm:$cc, CPSR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3644-3645
```tablegen
def : T2Pat<(ARMcmov i32:$false, (rotr i32:$Rm, imm0_31:$imm), imm:$cc, CPSR),
            (t2MOVCCror $false, $Rm, imm0_31:$imm, imm:$cc, CPSR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3647-3648
```tablegen
def : T2Pat<(ARMcmov i32:$false, imm:$src, imm:$cc, CPSR),
            (t2MOVCCi32imm $false, imm:$src, imm:$cc, CPSR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3650-3652
```tablegen
//===----------------------------------------------------------------------===//
// Atomic operations intrinsics
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 3654-3662
```tablegen
// memory barriers protect the atomic sequences
let hasSideEffects = 1 in {
def t2DMB : T2I<(outs), (ins memb_opt:$opt), NoItinerary,
                "dmb", "\t$opt", [(int_arm_dmb (i32 imm0_15:$opt))]>,
                Requires<[IsThumb, HasDB]> {
  bits<4> opt;
  let Inst{31-4} = 0xf3bf8f5;
  let Inst{3-0} = opt;
}
```
- EN: Defines TableGen record `t2DMB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2DMB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3664-3670
```tablegen
def t2DSB : T2I<(outs), (ins memb_opt:$opt), NoItinerary,
                "dsb", "\t$opt", [(int_arm_dsb (i32 imm0_15:$opt))]>,
                Requires<[IsThumb, HasDB]> {
  bits<4> opt;
  let Inst{31-4} = 0xf3bf8f4;
  let Inst{3-0} = opt;
}
```
- EN: Defines TableGen record `t2DSB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2DSB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3672-3678
```tablegen
def t2ISB : T2I<(outs), (ins instsyncb_opt:$opt), NoItinerary,
                "isb", "\t$opt", [(int_arm_isb (i32 imm0_15:$opt))]>,
                Requires<[IsThumb, HasDB]> {
  bits<4> opt;
  let Inst{31-4} = 0xf3bf8f6;
  let Inst{3-0} = opt;
}
```
- EN: Defines TableGen record `t2ISB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2ISB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3680-3685
```tablegen
let hasNoSchedulingInfo = 1 in
def t2TSB : T2I<(outs), (ins tsb_opt:$opt), NoItinerary,
                "tsb", "\t$opt", []>, Requires<[IsThumb, HasV8_4a]> {
  let Inst{31-0} = 0xf3af8012;
  let DecoderMethod = "DecodeTSBInstruction";
}
```
- EN: Defines TableGen record `t2TSB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2TSB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3687-3693
```tablegen
// Armv8.5-A speculation barrier
def t2SB : Thumb2XI<(outs), (ins), AddrModeNone, 4, NoItinerary, "sb", "", []>,
           Requires<[IsThumb2, HasSB]>, Sched<[]> {
  let Inst{31-0} = 0xf3bf8f70;
  let Unpredictable = 0x000f2f0f;
}
} // hasSideEffects = 1
```
- EN: Defines TableGen record `t2SB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3695-3703
```tablegen
class T2I_ldrex<bits<4> opcod, dag oops, dag iops, AddrMode am, int sz,
                InstrItinClass itin, string opc, string asm, string cstr,
                list<dag> pattern, bits<4> rt2 = 0b1111>
  : Thumb2I<oops, iops, am, sz, itin, opc, asm, cstr, pattern> {
  let Inst{31-27} = 0b11101;
  let Inst{26-20} = 0b0001101;
  let Inst{11-8} = rt2;
  let Inst{7-4} = opcod;
  let Inst{3-0} = 0b1111;
```
- EN: Declares reusable TableGen class `T2I_ldrex` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2I_ldrex`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3705-3717
```tablegen
  bits<4> addr;
  bits<4> Rt;
  let Inst{19-16} = addr;
  let Inst{15-12} = Rt;
}
class T2I_strex<bits<4> opcod, dag oops, dag iops, AddrMode am, int sz,
                InstrItinClass itin, string opc, string asm, string cstr,
                list<dag> pattern, bits<4> rt2 = 0b1111>
  : Thumb2I<oops, iops, am, sz, itin, opc, asm, cstr, pattern> {
  let Inst{31-27} = 0b11101;
  let Inst{26-20} = 0b0001100;
  let Inst{11-8} = rt2;
  let Inst{7-4} = opcod;
```
- EN: Declares reusable TableGen class `T2I_strex` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2I_strex`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3719-3725
```tablegen
  bits<4> Rd;
  bits<4> addr;
  bits<4> Rt;
  let Inst{3-0}  = Rd;
  let Inst{19-16} = addr;
  let Inst{15-12} = Rt;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3727-3744
```tablegen
let mayLoad = 1 in {
def t2LDREXB : T2I_ldrex<0b0100, (outs rGPR:$Rt), (ins addr_offset_none:$addr),
                         AddrModeNone, 4, NoItinerary,
                         "ldrexb", "\t$Rt, $addr", "",
                         [(set rGPR:$Rt, (ldrex_1 addr_offset_none:$addr))]>,
               Requires<[IsThumb, HasV8MBaseline]>, Sched<[WriteLd]>;
def t2LDREXH : T2I_ldrex<0b0101, (outs rGPR:$Rt), (ins addr_offset_none:$addr),
                         AddrModeNone, 4, NoItinerary,
                         "ldrexh", "\t$Rt, $addr", "",
                         [(set rGPR:$Rt, (ldrex_2 addr_offset_none:$addr))]>,
               Requires<[IsThumb, HasV8MBaseline]>, Sched<[WriteLd]>;
def t2LDREX  : Thumb2I<(outs rGPR:$Rt), (ins t2addrmode_imm0_1020s4:$addr),
                       AddrModeT2_ldrex, 4, NoItinerary,
                       "ldrex", "\t$Rt, $addr", "",
                     [(set rGPR:$Rt, (ldrex_4 t2addrmode_imm0_1020s4:$addr))]>,
               Requires<[IsThumb, HasV8MBaseline]>, Sched<[WriteLd]> {
  bits<4> Rt;
  bits<12> addr;
```
- EN: Defines TableGen record `t2LDREXB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDREXB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3745-3762
```tablegen
  let Inst{31-27} = 0b11101;
  let Inst{26-20} = 0b0000101;
  let Inst{19-16} = addr{11-8};
  let Inst{15-12} = Rt;
  let Inst{11-8} = 0b1111;
  let Inst{7-0} = addr{7-0};
}
let hasExtraDefRegAllocReq = 1 in
def t2LDREXD : T2I_ldrex<0b0111, (outs rGPR:$Rt, rGPR:$Rt2),
                         (ins addr_offset_none:$addr),
                         AddrModeNone, 4, NoItinerary,
                         "ldrexd", "\t$Rt, $Rt2, $addr", "",
                         [], {?, ?, ?, ?}>,
               Requires<[IsThumb2, IsNotMClass]>, Sched<[WriteLd]> {
  bits<4> Rt2;
  let Inst{11-8} = Rt2;
}
def t2LDAEXB : T2I_ldrex<0b1100, (outs rGPR:$Rt), (ins addr_offset_none:$addr),
```
- EN: Defines TableGen record `t2LDREXD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDREXD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3763-3780
```tablegen
                         AddrModeNone, 4, NoItinerary,
                         "ldaexb", "\t$Rt, $addr", "",
                         [(set rGPR:$Rt, (ldaex_1 addr_offset_none:$addr))]>,
               Requires<[IsThumb, HasAcquireRelease, HasV7Clrex]>, Sched<[WriteLd]>;
def t2LDAEXH : T2I_ldrex<0b1101, (outs rGPR:$Rt), (ins addr_offset_none:$addr),
                         AddrModeNone, 4, NoItinerary,
                         "ldaexh", "\t$Rt, $addr", "",
                         [(set rGPR:$Rt, (ldaex_2 addr_offset_none:$addr))]>,
               Requires<[IsThumb, HasAcquireRelease, HasV7Clrex]>, Sched<[WriteLd]>;
def t2LDAEX  : Thumb2I<(outs rGPR:$Rt), (ins addr_offset_none:$addr),
                       AddrModeNone, 4, NoItinerary,
                       "ldaex", "\t$Rt, $addr", "",
                         [(set rGPR:$Rt, (ldaex_4 addr_offset_none:$addr))]>,
               Requires<[IsThumb, HasAcquireRelease, HasV7Clrex]>, Sched<[WriteLd]> {
  bits<4> Rt;
  bits<4> addr;
  let Inst{31-27} = 0b11101;
  let Inst{26-20} = 0b0001101;
```
- EN: Defines TableGen record `t2LDAEXH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDAEXH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3781-3794
```tablegen
  let Inst{19-16} = addr;
  let Inst{15-12} = Rt;
  let Inst{11-8} = 0b1111;
  let Inst{7-0} = 0b11101111;
}
let hasExtraDefRegAllocReq = 1 in
def t2LDAEXD : T2I_ldrex<0b1111, (outs rGPR:$Rt, rGPR:$Rt2),
                         (ins addr_offset_none:$addr),
                         AddrModeNone, 4, NoItinerary,
                         "ldaexd", "\t$Rt, $Rt2, $addr", "",
                         [], {?, ?, ?, ?}>, Requires<[IsThumb,
                         HasAcquireRelease, HasV7Clrex, IsNotMClass]>, Sched<[WriteLd]> {
  bits<4> Rt2;
  let Inst{11-8} = Rt2;
```
- EN: Defines TableGen record `t2LDAEXD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDAEXD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3796-3798
```tablegen
  let Inst{7} = 1;
}
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3800-3814
```tablegen
let mayStore = 1, Constraints = "@earlyclobber $Rd" in {
def t2STREXB : T2I_strex<0b0100, (outs rGPR:$Rd),
                         (ins rGPR:$Rt, addr_offset_none:$addr),
                         AddrModeNone, 4, NoItinerary,
                         "strexb", "\t$Rd, $Rt, $addr", "",
                         [(set rGPR:$Rd,
                               (strex_1 rGPR:$Rt, addr_offset_none:$addr))]>,
               Requires<[IsThumb, HasV8MBaseline]>, Sched<[WriteST]>;
def t2STREXH : T2I_strex<0b0101, (outs rGPR:$Rd),
                         (ins rGPR:$Rt, addr_offset_none:$addr),
                         AddrModeNone, 4, NoItinerary,
                         "strexh", "\t$Rd, $Rt, $addr", "",
                         [(set rGPR:$Rd,
                               (strex_2 rGPR:$Rt, addr_offset_none:$addr))]>,
               Requires<[IsThumb, HasV8MBaseline]>, Sched<[WriteST]>;
```
- EN: Defines TableGen record `t2STREXB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STREXB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3816-3833
```tablegen
def t2STREX  : Thumb2I<(outs rGPR:$Rd), (ins rGPR:$Rt,
                             t2addrmode_imm0_1020s4:$addr),
                  AddrModeT2_ldrex, 4, NoItinerary,
                  "strex", "\t$Rd, $Rt, $addr", "",
                  [(set rGPR:$Rd,
                        (strex_4 rGPR:$Rt, t2addrmode_imm0_1020s4:$addr))]>,
               Requires<[IsThumb, HasV8MBaseline]>, Sched<[WriteST]> {
  bits<4> Rd;
  bits<4> Rt;
  bits<12> addr;
  let Inst{31-27} = 0b11101;
  let Inst{26-20} = 0b0000100;
  let Inst{19-16} = addr{11-8};
  let Inst{15-12} = Rt;
  let Inst{11-8}  = Rd;
  let Inst{7-0} = addr{7-0};
}
let hasExtraSrcRegAllocReq = 1 in
```
- EN: Defines TableGen record `t2STREX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STREX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3834-3850
```tablegen
def t2STREXD : T2I_strex<0b0111, (outs rGPR:$Rd),
                         (ins rGPR:$Rt, rGPR:$Rt2, addr_offset_none:$addr),
                         AddrModeNone, 4, NoItinerary,
                         "strexd", "\t$Rd, $Rt, $Rt2, $addr", "", [],
                         {?, ?, ?, ?}>,
               Requires<[IsThumb2, IsNotMClass]>, Sched<[WriteST]> {
  bits<4> Rt2;
  let Inst{11-8} = Rt2;
}
def t2STLEXB : T2I_strex<0b1100, (outs rGPR:$Rd),
                         (ins rGPR:$Rt, addr_offset_none:$addr),
                         AddrModeNone, 4, NoItinerary,
                         "stlexb", "\t$Rd, $Rt, $addr", "",
                         [(set rGPR:$Rd,
                               (stlex_1 rGPR:$Rt, addr_offset_none:$addr))]>,
                         Requires<[IsThumb, HasAcquireRelease,
                                   HasV7Clrex]>, Sched<[WriteST]>;
```
- EN: Defines TableGen record `t2STREXD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STREXD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3852-3859
```tablegen
def t2STLEXH : T2I_strex<0b1101, (outs rGPR:$Rd),
                         (ins rGPR:$Rt, addr_offset_none:$addr),
                         AddrModeNone, 4, NoItinerary,
                         "stlexh", "\t$Rd, $Rt, $addr", "",
                         [(set rGPR:$Rd,
                               (stlex_2 rGPR:$Rt, addr_offset_none:$addr))]>,
                         Requires<[IsThumb, HasAcquireRelease,
                                   HasV7Clrex]>, Sched<[WriteST]>;
```
- EN: Defines TableGen record `t2STLEXH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STLEXH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3861-3878
```tablegen
def t2STLEX  : Thumb2I<(outs rGPR:$Rd), (ins rGPR:$Rt,
                             addr_offset_none:$addr),
                  AddrModeNone, 4, NoItinerary,
                  "stlex", "\t$Rd, $Rt, $addr", "",
                  [(set rGPR:$Rd,
                        (stlex_4 rGPR:$Rt, addr_offset_none:$addr))]>,
                  Requires<[IsThumb, HasAcquireRelease, HasV7Clrex]>,
                  Sched<[WriteST]> {
  bits<4> Rd;
  bits<4> Rt;
  bits<4> addr;
  let Inst{31-27} = 0b11101;
  let Inst{26-20} = 0b0001100;
  let Inst{19-16} = addr;
  let Inst{15-12} = Rt;
  let Inst{11-4}  = 0b11111110;
  let Inst{3-0}   = Rd;
}
```
- EN: Defines TableGen record `t2STLEX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STLEX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3879-3889
```tablegen
let hasExtraSrcRegAllocReq = 1 in
def t2STLEXD : T2I_strex<0b1111, (outs rGPR:$Rd),
                         (ins rGPR:$Rt, rGPR:$Rt2, addr_offset_none:$addr),
                         AddrModeNone, 4, NoItinerary,
                         "stlexd", "\t$Rd, $Rt, $Rt2, $addr", "", [],
                         {?, ?, ?, ?}>, Requires<[IsThumb, HasAcquireRelease,
                         HasV7Clrex, IsNotMClass]>, Sched<[WriteST]> {
  bits<4> Rt2;
  let Inst{11-8} = Rt2;
}
}
```
- EN: Defines TableGen record `t2STLEXD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2STLEXD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3891-3900
```tablegen
def t2CLREX : T2I<(outs), (ins), NoItinerary, "clrex", "", [(int_arm_clrex)]>,
            Requires<[IsThumb, HasV7Clrex]>  {
  let Inst{31-16} = 0xf3bf;
  let Inst{15-14} = 0b10;
  let Inst{13} = 0;
  let Inst{12} = 0;
  let Inst{11-8} = 0b1111;
  let Inst{7-4} = 0b0010;
  let Inst{3-0} = 0b1111;
}
```
- EN: Defines TableGen record `t2CLREX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2CLREX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3902-3913
```tablegen
def : T2Pat<(and (ldrex_1 addr_offset_none:$addr), 0xff),
            (t2LDREXB addr_offset_none:$addr)>,
            Requires<[IsThumb, HasV8MBaseline]>;
def : T2Pat<(and (ldrex_2 addr_offset_none:$addr), 0xffff),
            (t2LDREXH addr_offset_none:$addr)>,
            Requires<[IsThumb, HasV8MBaseline]>;
def : T2Pat<(strex_1 (and GPR:$Rt, 0xff), addr_offset_none:$addr),
            (t2STREXB GPR:$Rt, addr_offset_none:$addr)>,
            Requires<[IsThumb, HasV8MBaseline]>;
def : T2Pat<(strex_2 (and GPR:$Rt, 0xffff), addr_offset_none:$addr),
            (t2STREXH GPR:$Rt, addr_offset_none:$addr)>,
            Requires<[IsThumb, HasV8MBaseline]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3915-3926
```tablegen
def : T2Pat<(and (ldaex_1 addr_offset_none:$addr), 0xff),
            (t2LDAEXB addr_offset_none:$addr)>,
            Requires<[IsThumb, HasAcquireRelease, HasV7Clrex]>;
def : T2Pat<(and (ldaex_2 addr_offset_none:$addr), 0xffff),
            (t2LDAEXH addr_offset_none:$addr)>,
            Requires<[IsThumb, HasAcquireRelease, HasV7Clrex]>;
def : T2Pat<(stlex_1 (and GPR:$Rt, 0xff), addr_offset_none:$addr),
            (t2STLEXB GPR:$Rt, addr_offset_none:$addr)>,
            Requires<[IsThumb, HasAcquireRelease, HasV7Clrex]>;
def : T2Pat<(stlex_2 (and GPR:$Rt, 0xffff), addr_offset_none:$addr),
            (t2STLEXH GPR:$Rt, addr_offset_none:$addr)>,
            Requires<[IsThumb, HasAcquireRelease, HasV7Clrex]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3928-3945
```tablegen
//===----------------------------------------------------------------------===//
// SJLJ Exception handling intrinsics
//   eh_sjlj_setjmp() is an instruction sequence to store the return
//   address and save #0 in R0 for the non-longjmp case.
//   Since by its nature we may be coming from some other function to get
//   here, and we're using the stack frame for the containing function to
//   save/restore registers, we can't keep anything live in regs across
//   the eh_sjlj_setjmp(), else it will almost certainly have been tromped upon
//   when we get here from a longjmp(). We force everything out of registers
//   except for our own input by listing the relevant registers in Defs. By
//   doing so, we also cause the prologue/epilogue code to actively preserve
//   all of the callee-saved registers, which is exactly what we want.
//   $val is a scratch register for our use.
// This gets lowered to an instruction sequence of 12 bytes
let Defs =
  [ R0,  R1,  R2,  R3,  R4,  R5,  R6,  R7,  R8,  R9,  R10, R11, R12, LR, CPSR,
    Q0, Q1, Q2, Q3, Q8, Q9, Q10, Q11, Q12, Q13, Q14, Q15],
  hasSideEffects = 1, isBarrier = 1, isCodeGenOnly = 1, Size = 12,
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 3946-3951
```tablegen
  usesCustomInserter = 1 in {
  def t2Int_eh_sjlj_setjmp : Thumb2XI<(outs), (ins tGPR:$src, tGPR:$val),
                               AddrModeNone, 0, NoItinerary, "", "",
                          [(set R0, (ARMeh_sjlj_setjmp tGPR:$src, tGPR:$val))]>,
                             Requires<[IsThumb2, HasVFP2]>;
}
```
- EN: Defines TableGen record `t2Int_eh_sjlj_setjmp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2Int_eh_sjlj_setjmp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3953-3962
```tablegen
// This gets lowered to an instruction sequence of 12 bytes
let Defs =
  [ R0,  R1,  R2,  R3,  R4,  R5,  R6,  R7,  R8,  R9,  R10, R11, R12, LR, CPSR ],
  hasSideEffects = 1, isBarrier = 1, isCodeGenOnly = 1, Size = 12,
  usesCustomInserter = 1 in {
  def t2Int_eh_sjlj_setjmp_nofp : Thumb2XI<(outs), (ins tGPR:$src, tGPR:$val),
                               AddrModeNone, 0, NoItinerary, "", "",
                          [(set R0, (ARMeh_sjlj_setjmp tGPR:$src, tGPR:$val))]>,
                                  Requires<[IsThumb2, NoVFP]>;
}
```
- EN: Defines TableGen record `t2Int_eh_sjlj_setjmp_nofp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2Int_eh_sjlj_setjmp_nofp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3965-3967
```tablegen
//===----------------------------------------------------------------------===//
// Control-Flow Instructions
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 3969-3977
```tablegen
// FIXME: remove when we have a way to marking a MI with these properties.
// FIXME: Should pc be an implicit operand like PICADD, etc?
let isReturn = 1, isTerminator = 1, isBarrier = 1, mayLoad = 1,
    hasExtraDefRegAllocReq = 1, isCodeGenOnly = 1 in
def t2LDMIA_RET: t2PseudoExpand<(outs GPR:$wb), (ins GPR:$Rn, pred:$p,
                                                   reglist:$regs, variable_ops),
                              4, IIC_iLoad_mBr, [],
            (t2LDMIA_UPD GPR:$wb, GPR:$Rn, pred:$p, reglist:$regs)>,
                         RegConstraint<"$Rn = $wb">;
```
- EN: Defines TableGen record `t2LDMIA_RET:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDMIA_RET:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3979-3987
```tablegen
let isBranch = 1, isTerminator = 1, isBarrier = 1 in {
let isPredicable = 1 in
def t2B   : T2I<(outs), (ins thumb_br_target:$target), IIC_Br,
                 "b", ".w\t$target",
                 [(br bb:$target)]>, Sched<[WriteBr]>,
                 Requires<[IsThumb, HasV8MBaseline]> {
  let Inst{31-27} = 0b11110;
  let Inst{15-14} = 0b10;
  let Inst{12} = 1;
```
- EN: Defines TableGen record `t2B` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2B`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3989-3997
```tablegen
  bits<24> target;
  let Inst{26} = target{23};
  let Inst{13} = target{22};
  let Inst{11} = target{21};
  let Inst{25-16} = target{20-11};
  let Inst{10-0} = target{10-0};
  let DecoderMethod = "DecodeT2BInstruction";
  let AsmMatchConverter = "cvtThumbBranches";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3999-4000
```tablegen
let Size = 4, isNotDuplicable = 1, isBranch = 1, isTerminator = 1,
    isBarrier = 1, isIndirectBranch = 1 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4002-4007
```tablegen
// available in both v8-M.Baseline and Thumb2 targets
def t2BR_JT : t2basePseudoInst<(outs),
          (ins GPR:$target, GPR:$index, i32imm:$jt),
           0, IIC_Br,
          [(ARMbr2jt GPR:$target, GPR:$index, tjumptable:$jt)]>,
          Sched<[WriteBr]>;
```
- EN: Defines TableGen record `t2BR_JT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2BR_JT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4009-4012
```tablegen
// FIXME: Add a case that can be predicated.
def t2TBB_JT : t2PseudoInst<(outs),
        (ins GPR:$base, GPR:$index, i32imm:$jt, i32imm:$pclbl), 0, IIC_Br, []>,
        Sched<[WriteBr]>;
```
- EN: Defines TableGen record `t2TBB_JT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2TBB_JT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4014-4016
```tablegen
def t2TBH_JT : t2PseudoInst<(outs),
        (ins GPR:$base, GPR:$index, i32imm:$jt, i32imm:$pclbl), 0, IIC_Br, []>,
        Sched<[WriteBr]>;
```
- EN: Defines TableGen record `t2TBH_JT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2TBH_JT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4018-4026
```tablegen
def t2TBB : T2I<(outs), (ins (addrmode_tbb $Rn, $Rm):$addr), IIC_Br,
                    "tbb", "\t$addr", []>, Sched<[WriteBrTbl]> {
  bits<4> Rn;
  bits<4> Rm;
  let Inst{31-20} = 0b111010001101;
  let Inst{19-16} = Rn;
  let Inst{15-5} = 0b11110000000;
  let Inst{4} = 0; // B form
  let Inst{3-0} = Rm;
```
- EN: Defines TableGen record `t2TBB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2TBB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4028-4029
```tablegen
  let DecoderMethod = "DecodeThumbTableBranch";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4031-4039
```tablegen
def t2TBH : T2I<(outs), (ins (addrmode_tbh $Rn, $Rm):$addr), IIC_Br,
                   "tbh", "\t$addr", []>, Sched<[WriteBrTbl]> {
  bits<4> Rn;
  bits<4> Rm;
  let Inst{31-20} = 0b111010001101;
  let Inst{19-16} = Rn;
  let Inst{15-5} = 0b11110000000;
  let Inst{4} = 1; // H form
  let Inst{3-0} = Rm;
```
- EN: Defines TableGen record `t2TBH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2TBH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4041-4043
```tablegen
  let DecoderMethod = "DecodeThumbTableBranch";
}
} // isNotDuplicable, isIndirectBranch
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4045-4045
```tablegen
} // isBranch, isTerminator, isBarrier
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4047-4057
```tablegen
// FIXME: should be able to write a pattern for ARMBrcond, but can't use
// a two-value operand where a dag node expects ", "two operands. :(
let isBranch = 1, isTerminator = 1, hasSideEffects = 0 in
def t2Bcc : Thumb2XI<(outs), (ins brtarget:$target, pred:$p),
                     AddrModeNone, 4, IIC_Br,
                     "b${p}.w\t$target", "",
                     [/*(ARMbrcond bb:$target, imm:$cc)*/]>,
            Sched<[WriteBr]> {
  let Inst{31-27} = 0b11110;
  let Inst{15-14} = 0b10;
  let Inst{12} = 0;
```
- EN: Defines TableGen record `t2Bcc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2Bcc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4059-4060
```tablegen
  bits<4> p;
  let Inst{25-22} = p;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4062-4067
```tablegen
  bits<21> target;
  let Inst{26} = target{20};
  let Inst{11} = target{19};
  let Inst{13} = target{18};
  let Inst{21-16} = target{17-12};
  let Inst{10-0} = target{11-1};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4069-4071
```tablegen
  let DecoderMethod = "DecodeThumb2BCCInstruction";
  let AsmMatchConverter = "cvtThumbBranches";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4073-4084
```tablegen
// Tail calls. The MachO version of thumb tail calls uses a t2 branch, so
// it goes here.
// Windows SEH unwinding also needs a strict t2 branch for tail calls.
let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1 in {
  // IOS version.
  let Uses = [SP] in
  def tTAILJMPd: tPseudoExpand<(outs),
                   (ins thumb_br_target:$dst, pred:$p),
                   4, IIC_Br, [],
                   (t2B thumb_br_target:$dst, pred:$p)>,
                 Requires<[IsThumb2]>, Sched<[WriteBr]>;
}
```
- EN: Defines TableGen record `tTAILJMPd:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tTAILJMPd:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4086-4093
```tablegen
// IT block
let Defs = [ITSTATE] in
def t2IT : Thumb2XI<(outs), (ins it_pred:$cc, it_mask:$mask),
                    AddrModeNone, 2,  IIC_iALUx,
                    "it$mask\t$cc", "", []> {
  // 16-bit instruction.
  let Inst{31-16} = 0x0000;
  let Inst{15-8} = 0b10111111;
```
- EN: Defines TableGen record `t2IT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2IT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4095-4098
```tablegen
  bits<4> cc;
  bits<4> mask;
  let Inst{7-4} = cc;
  let Inst{3-0} = mask;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4100-4101
```tablegen
  let DecoderMethod = "DecodeIT";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4103-4114
```tablegen
// Branch and Exchange Jazelle -- for disassembly only
// Rm = Inst{19-16}
let isBranch = 1, isTerminator = 1, isBarrier = 1, isIndirectBranch = 1 in
def t2BXJ : T2I<(outs), (ins GPRnopc:$func), NoItinerary, "bxj", "\t$func", []>,
    Sched<[WriteBr]>, Requires<[IsThumb2, IsNotMClass]> {
  bits<4> func;
  let Inst{31-27} = 0b11110;
  let Inst{26} = 0;
  let Inst{25-20} = 0b111100;
  let Inst{19-16} = func;
  let Inst{15-0} = 0b1000111100000000;
}
```
- EN: Defines TableGen record `t2BXJ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2BXJ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4116-4116
```tablegen
def : t2InstAlias<"bl${p}.w $func", (tBL pred:$p, thumb_bl_target:$func), 0>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4118-4130
```tablegen
// Compare and branch on zero / non-zero
let isBranch = 1, isTerminator = 1, hasSideEffects = 0 in {
  def tCBZ  : T1I<(outs), (ins tGPR:$Rn, thumb_cb_target:$target), IIC_Br,
                  "cbz\t$Rn, $target", []>,
              T1Misc<{0,0,?,1,?,?,?}>,
              Requires<[IsThumb, HasV8MBaseline]>, Sched<[WriteBr]> {
    // A8.6.27
    bits<6> target;
    bits<3> Rn;
    let Inst{9}   = target{5};
    let Inst{7-3} = target{4-0};
    let Inst{2-0} = Rn;
  }
```
- EN: Defines TableGen record `tCBZ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tCBZ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4132-4143
```tablegen
  def tCBNZ : T1I<(outs), (ins tGPR:$Rn, thumb_cb_target:$target), IIC_Br,
                  "cbnz\t$Rn, $target", []>,
              T1Misc<{1,0,?,1,?,?,?}>,
              Requires<[IsThumb, HasV8MBaseline]>, Sched<[WriteBr]> {
    // A8.6.27
    bits<6> target;
    bits<3> Rn;
    let Inst{9}   = target{5};
    let Inst{7-3} = target{4-0};
    let Inst{2-0} = Rn;
  }
}
```
- EN: Defines TableGen record `tCBNZ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tCBNZ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4146-4156
```tablegen
// Change Processor State is a system instruction.
// FIXME: Since the asm parser has currently no clean way to handle optional
// operands, create 3 versions of the same instruction. Once there's a clean
// framework to represent optional operands, change this behavior.
class t2CPS<dag iops, string asm_op> : T2XI<(outs), iops, NoItinerary,
            !strconcat("cps", asm_op), []>,
          Requires<[IsThumb2, IsNotMClass]> {
  bits<2> imod;
  bits<3> iflags;
  bits<5> mode;
  bit M;
```
- EN: Declares reusable TableGen class `t2CPS` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `t2CPS`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4158-4164
```tablegen
  let Inst{31-11} = 0b111100111010111110000;
  let Inst{10-9}  = imod;
  let Inst{8}     = M;
  let Inst{7-5}   = iflags;
  let Inst{4-0}   = mode;
  let DecoderMethod = "DecodeT2CPSInstruction";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4166-4173
```tablegen
let M = 1 in
  def t2CPS3p : t2CPS<(ins imod_op:$imod, iflags_op:$iflags, i32imm:$mode),
                      "$imod\t$iflags, $mode">;
let mode = 0, M = 0 in
  def t2CPS2p : t2CPS<(ins imod_op:$imod, iflags_op:$iflags),
                      "$imod.w\t$iflags">;
let imod = 0, iflags = 0, M = 1 in
  def t2CPS1p : t2CPS<(ins imm0_31:$mode), "\t$mode">;
```
- EN: Defines TableGen record `t2CPS3p` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2CPS3p`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4175-4177
```tablegen
def : t2InstAlias<"cps$imod.w $iflags, $mode",
                   (t2CPS3p imod_op:$imod, iflags_op:$iflags, i32imm:$mode), 0>;
def : t2InstAlias<"cps.w $mode", (t2CPS1p imm0_31:$mode), 0>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4179-4185
```tablegen
// A6.3.4 Branches and miscellaneous control
// Table A6-14 Change Processor State, and hint instructions
def t2HINT : T2I<(outs), (ins imm0_239:$imm), NoItinerary, "hint", ".w\t$imm",
                  [(int_arm_hint imm0_239:$imm)]> {
  bits<8> imm;
  let Inst{31-3} = 0b11110011101011111000000000000;
  let Inst{7-0} = imm;
```
- EN: Defines TableGen record `t2HINT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2HINT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4187-4188
```tablegen
  let DecoderMethod = "DecodeT2HintSpaceInstruction";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4190-4206
```tablegen
def : t2InstAlias<"hint$p $imm", (t2HINT imm0_239:$imm, pred:$p), 0>;
def : t2InstAlias<"nop$p.w", (t2HINT 0, pred:$p), 1>;
def : t2InstAlias<"yield$p.w", (t2HINT 1, pred:$p), 1>;
def : t2InstAlias<"wfe$p.w", (t2HINT 2, pred:$p), 1>;
def : t2InstAlias<"wfi$p.w", (t2HINT 3, pred:$p), 1>;
def : t2InstAlias<"sev$p.w", (t2HINT 4, pred:$p), 1>;
def : t2InstAlias<"sevl$p.w", (t2HINT 5, pred:$p), 1> {
  let Predicates = [IsThumb2, HasV8];
}
def : t2InstAlias<"esb$p.w", (t2HINT 16, pred:$p), 1> {
  let Predicates = [IsThumb2, HasRAS];
}
def : t2InstAlias<"esb$p", (t2HINT 16, pred:$p), 0> {
  let Predicates = [IsThumb2, HasRAS];
}
def : t2InstAlias<"csdb$p.w", (t2HINT 20, pred:$p), 0>;
def : t2InstAlias<"csdb$p",   (t2HINT 20, pred:$p), 1>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4208-4211
```tablegen
def : t2InstAlias<"pacbti$p r12,lr,sp", (t2HINT 13, pred:$p), 1>;
def : t2InstAlias<"bti$p", (t2HINT 15, pred:$p), 1>;
def : t2InstAlias<"pac$p r12,lr,sp", (t2HINT 29, pred:$p), 1>;
def : t2InstAlias<"aut$p r12,lr,sp", (t2HINT 45, pred:$p), 1>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4213-4215
```tablegen
// Clear BHB instruction
def : InstAlias<"clrbhb$p", (t2HINT 22, pred:$p), 0>, Requires<[IsThumb2, HasV8]>;
def : InstAlias<"clrbhb$p", (t2HINT 22, pred:$p), 1>, Requires<[IsThumb2, HasV8, HasCLRBHB]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4217-4226
```tablegen
def t2DBG : T2I<(outs), (ins imm0_15:$opt), NoItinerary, "dbg", "\t$opt",
                [(int_arm_dbg imm0_15:$opt)]> {
  bits<4> opt;
  let Inst{31-20} = 0b111100111010;
  let Inst{19-16} = 0b1111;
  let Inst{15-8} = 0b10000000;
  let Inst{7-4} = 0b1111;
  let Inst{3-0} = opt;
}
def : t2InstAlias<"dbg${p}.w $opt", (t2DBG imm0_15:$opt, pred:$p), 0>;
```
- EN: Defines TableGen record `t2DBG` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2DBG`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4228-4235
```tablegen
// Secure Monitor Call is a system instruction.
// Option = Inst{19-16}
let isCall = 1, Uses = [SP] in
def t2SMC : T2I<(outs), (ins imm0_15:$opt), NoItinerary, "smc", "\t$opt",
                []>, Requires<[IsThumb2, HasTrustZone]> {
  let Inst{31-27} = 0b11110;
  let Inst{26-20} = 0b1111111;
  let Inst{15-12} = 0b1000;
```
- EN: Defines TableGen record `t2SMC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SMC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4237-4239
```tablegen
  bits<4> opt;
  let Inst{19-16} = opt;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4241-4249
```tablegen
class T2DCPS<bits<2> opt, string opc>
  : T2I<(outs), (ins), NoItinerary, opc, "", []>, Requires<[IsThumb2, HasV8]> {
  let Inst{31-27} = 0b11110;
  let Inst{26-20} = 0b1111000;
  let Inst{19-16} = 0b1111;
  let Inst{15-12} = 0b1000;
  let Inst{11-2} = 0b0000000000;
  let Inst{1-0} = opt;
}
```
- EN: Declares reusable TableGen class `T2DCPS` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2DCPS`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4251-4253
```tablegen
def t2DCPS1 : T2DCPS<0b01, "dcps1">;
def t2DCPS2 : T2DCPS<0b10, "dcps2">;
def t2DCPS3 : T2DCPS<0b11, "dcps3">;
```
- EN: Defines TableGen record `t2DCPS1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2DCPS1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4255-4267
```tablegen
class T2SRS<bits<2> Op, bit W, dag oops, dag iops, InstrItinClass itin,
            string opc, string asm, list<dag> pattern>
  : T2I<oops, iops, itin, opc, asm, pattern>,
    Requires<[IsThumb2,IsNotMClass]> {
  bits<5> mode;
  let Inst{31-25} = 0b1110100;
  let Inst{24-23} = Op;
  let Inst{22} = 0;
  let Inst{21} = W;
  let Inst{20-16} = 0b01101;
  let Inst{15-5} = 0b11000000000;
  let Inst{4-0} = mode{4-0};
}
```
- EN: Declares reusable TableGen class `T2SRS` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2SRS`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4269-4277
```tablegen
// Store Return State is a system instruction.
def t2SRSDB_UPD : T2SRS<0b00, 1, (outs), (ins imm0_31:$mode), NoItinerary,
                        "srsdb", "\tsp!, $mode", []>;
def t2SRSDB  : T2SRS<0b00, 0, (outs), (ins imm0_31:$mode), NoItinerary,
                     "srsdb","\tsp, $mode", []>;
def t2SRSIA_UPD : T2SRS<0b11, 1, (outs), (ins imm0_31:$mode), NoItinerary,
                        "srsia","\tsp!, $mode", []>;
def t2SRSIA  : T2SRS<0b11, 0, (outs), (ins imm0_31:$mode), NoItinerary,
                     "srsia","\tsp, $mode", []>;
```
- EN: Defines TableGen record `t2SRSDB_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SRSDB_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4280-4281
```tablegen
def : t2InstAlias<"srsdb${p} $mode", (t2SRSDB imm0_31:$mode, pred:$p)>;
def : t2InstAlias<"srsdb${p} $mode!", (t2SRSDB_UPD imm0_31:$mode, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4283-4284
```tablegen
def : t2InstAlias<"srsia${p} $mode", (t2SRSIA imm0_31:$mode, pred:$p)>;
def : t2InstAlias<"srsia${p} $mode!", (t2SRSIA_UPD imm0_31:$mode, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4286-4292
```tablegen
// Return From Exception is a system instruction.
let isReturn = 1, isBarrier = 1, isTerminator = 1, Defs = [PC] in
class T2RFE<bits<12> op31_20, dag oops, dag iops, InstrItinClass itin,
          string opc, string asm, list<dag> pattern>
  : T2I<oops, iops, itin, opc, asm, pattern>,
    Requires<[IsThumb2,IsNotMClass]> {
  let Inst{31-20} = op31_20{11-0};
```
- EN: Declares reusable TableGen class `T2RFE` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2RFE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4294-4297
```tablegen
  bits<4> Rn;
  let Inst{19-16} = Rn;
  let Inst{15-0} = 0xc000;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4299-4310
```tablegen
def t2RFEDBW : T2RFE<0b111010000011,
                   (outs), (ins GPR:$Rn), NoItinerary, "rfedb", "\t$Rn!",
                   [/* For disassembly only; pattern left blank */]>;
def t2RFEDB  : T2RFE<0b111010000001,
                   (outs), (ins GPR:$Rn), NoItinerary, "rfedb", "\t$Rn",
                   [/* For disassembly only; pattern left blank */]>;
def t2RFEIAW : T2RFE<0b111010011011,
                   (outs), (ins GPR:$Rn), NoItinerary, "rfeia", "\t$Rn!",
                   [/* For disassembly only; pattern left blank */]>;
def t2RFEIA  : T2RFE<0b111010011001,
                   (outs), (ins GPR:$Rn), NoItinerary, "rfeia", "\t$Rn",
                   [/* For disassembly only; pattern left blank */]>;
```
- EN: Defines TableGen record `t2RFEDBW` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2RFEDBW`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4312-4319
```tablegen
// B9.3.19 SUBS PC, LR, #imm (Thumb2) system instruction.
// Exception return instruction is "subs pc, lr, #imm".
let isReturn = 1, isBarrier = 1, isTerminator = 1, Defs = [PC] in
def t2SUBS_PC_LR : T2I <(outs), (ins imm0_255:$imm), NoItinerary,
                        "subs", "\tpc, lr, $imm",
                        [(ARMintretglue imm0_255:$imm)]>,
                   Requires<[IsThumb2,IsNotMClass]> {
  let Inst{31-8} = 0b111100111101111010001111;
```
- EN: Defines TableGen record `t2SUBS_PC_LR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SUBS_PC_LR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4321-4323
```tablegen
  bits<8> imm;
  let Inst{7-0} = imm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4325-4329
```tablegen
// B9.3.19 SUBS PC, LR (Thumb)
// In the Thumb instruction set, MOVS{<c>}{<q>} PC, LR is a pseudo-instruction
// for SUBS{<c>}{<q>} PC, LR, #0.
def : t2InstAlias<"movs${p}\tpc, lr", (t2SUBS_PC_LR 0, pred:$p)>;
def : t2InstAlias<"movs${p}.w\tpc, lr", (t2SUBS_PC_LR 0, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4331-4335
```tablegen
// ERET - Return from exception in Hypervisor mode.
// B9.3.3, B9.3.20: ERET is an alias for "SUBS PC, LR, #0" in an implementation that
// includes virtualization extensions.
def t2ERET : InstAlias<"eret${p}", (t2SUBS_PC_LR 0, pred:$p), 1>,
             Requires<[IsThumb2, HasVirtualization]>;
```
- EN: Defines TableGen record `t2ERET` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2ERET`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4337-4347
```tablegen
// Hypervisor Call is a system instruction.
let isCall = 1 in {
def t2HVC : T2XI <(outs), (ins imm0_65535:$imm16), IIC_Br, "hvc.w\t$imm16", []>,
      Requires<[IsThumb2, HasVirtualization]>, Sched<[WriteBr]> {
    bits<16> imm16;
    let Inst{31-20} = 0b111101111110;
    let Inst{19-16} = imm16{15-12};
    let Inst{15-12} = 0b1000;
    let Inst{11-0} = imm16{11-0};
}
}
```
- EN: Defines TableGen record `t2HVC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2HVC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4349-4352
```tablegen
// Alias for HVC without the ".w" optional width specifier
def : t2InstAlias<"hvc\t$imm16", (t2HVC imm0_65535:$imm16)> {
  let Predicates = [IsThumb2, HasVirtualization];
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4354-4356
```tablegen
//===----------------------------------------------------------------------===//
// Non-Instruction Patterns
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 4358-4364
```tablegen
// 32-bit immediate using movw + movt.
// This is a single pseudo instruction to make it re-materializable.
// FIXME: Remove this when we can do generalized remat.
let isReMaterializable = 1, isMoveImm = 1, Size = 8 in
def t2MOVi32imm : PseudoInst<(outs rGPR:$dst), (ins i32imm:$src), IIC_iMOVix2,
                            [(set rGPR:$dst, (i32 imm:$src))]>,
                            Requires<[IsThumb, UseMovt]>;
```
- EN: Defines TableGen record `t2MOVi32imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MOVi32imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4366-4374
```tablegen
// Pseudo instruction that combines movw + movt + add pc (if pic).
// It also makes it possible to rematerialize the instructions.
// FIXME: Remove this when we can do generalized remat and when machine licm
// can properly the instructions.
let isReMaterializable = 1 in {
def t2MOV_ga_pcrel : PseudoInst<(outs rGPR:$dst), (ins i32imm:$addr),
                                IIC_iMOVix2addpc,
                          [(set rGPR:$dst, (ARMWrapperPIC tglobaladdr:$addr))]>,
                          Requires<[IsThumb, HasV8MBaseline, UseMovtInPic]>;
```
- EN: Defines TableGen record `t2MOV_ga_pcrel` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MOV_ga_pcrel`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4376-4376
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4378-4383
```tablegen
def : T2Pat<(ARMWrapperPIC tglobaltlsaddr :$dst),
            (t2MOV_ga_pcrel tglobaltlsaddr:$dst)>,
      Requires<[IsThumb2, UseMovtInPic]>;
def : T2Pat<(ARMWrapper tglobaltlsaddr:$dst),
            (t2MOVi32imm tglobaltlsaddr:$dst)>,
      Requires<[IsThumb2, UseMovt]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4385-4390
```tablegen
// ConstantPool, GlobalAddress, and JumpTable
def : T2Pat<(ARMWrapper tconstpool :$dst), (t2LEApcrel tconstpool :$dst)>;
def : T2Pat<(ARMWrapper texternalsym :$dst), (t2MOVi32imm texternalsym :$dst)>,
    Requires<[IsThumb, HasV8MBaseline, UseMovt]>;
def : T2Pat<(ARMWrapper tglobaladdr :$dst), (t2MOVi32imm tglobaladdr :$dst)>,
    Requires<[IsThumb, HasV8MBaseline, UseMovt]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4392-4392
```tablegen
def : T2Pat<(ARMWrapperJT tjumptable:$dst), (t2LEApcrelJT tjumptable:$dst)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4394-4400
```tablegen
let hasNoSchedulingInfo = 1 in {
def t2LDRLIT_ga_pcrel : PseudoInst<(outs rGPR:$dst), (ins i32imm:$addr),
                                  IIC_iLoadiALU,
                                  [(set rGPR:$dst,
                                        (ARMWrapperPIC tglobaladdr:$addr))]>,
                       Requires<[IsThumb, HasV8MBaseline, DontUseMovtInPic]>;
}
```
- EN: Defines TableGen record `t2LDRLIT_ga_pcrel` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRLIT_ga_pcrel`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4402-4405
```tablegen
// TLS globals
def : Pat<(ARMWrapperPIC tglobaltlsaddr:$addr),
          (t2LDRLIT_ga_pcrel tglobaltlsaddr:$addr)>,
      Requires<[IsThumb, HasV8MBaseline, DontUseMovtInPic]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4407-4415
```tablegen
// Pseudo instruction that combines ldr from constpool and add pc. This should
// be expanded into two instructions late to allow if-conversion and
// scheduling.
let canFoldAsLoad = 1, isReMaterializable = 1 in
def t2LDRpci_pic : PseudoInst<(outs rGPR:$dst), (ins i32imm:$addr, pclabel:$cp),
                   IIC_iLoadiALU,
              [(set rGPR:$dst, (ARMpic_add (load (ARMWrapper tconstpool:$addr)),
                                           imm:$cp))]>,
               Requires<[IsThumb2]>;
```
- EN: Defines TableGen record `t2LDRpci_pic` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRpci_pic`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4417-4425
```tablegen
//===----------------------------------------------------------------------===//
// Coprocessor load/store -- for disassembly only
//
class T2CI<bits<4> op31_28, dag oops, dag iops, string opc, string asm,
           list<dag> pattern, AddrMode am = AddrModeNone>
  : T2I<oops, iops, NoItinerary, opc, asm, pattern, am> {
  let Inst{31-28} = op31_28;
  let Inst{27-25} = 0b110;
}
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 4427-4444
```tablegen
multiclass t2LdStCop<bits<4> op31_28, bit load, bit Dbit, string asm, list<dag> pattern> {
  def _OFFSET : T2CI<op31_28,
                     (outs), (ins p_imm:$cop, c_imm:$CRd, addrmode5:$addr),
                     asm, "\t$cop, $CRd, $addr", pattern, AddrMode5> {
    bits<13> addr;
    bits<4> cop;
    bits<4> CRd;
    let Inst{24} = 1; // P = 1
    let Inst{23} = addr{8};
    let Inst{22} = Dbit;
    let Inst{21} = 0; // W = 0
    let Inst{20} = load;
    let Inst{19-16} = addr{12-9};
    let Inst{15-12} = CRd;
    let Inst{11-8} = cop;
    let Inst{7-0} = addr{7-0};
    let DecoderMethod = "DecodeCopMemInstruction";
  }
```
- EN: Declares TableGen `multiclass t2LdStCop`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass t2LdStCop`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4445-4462
```tablegen
  def _PRE : T2CI<op31_28,
                  (outs), (ins p_imm:$cop, c_imm:$CRd, addrmode5_pre:$addr),
                  asm, "\t$cop, $CRd, $addr!", []> {
    bits<13> addr;
    bits<4> cop;
    bits<4> CRd;
    let Inst{24} = 1; // P = 1
    let Inst{23} = addr{8};
    let Inst{22} = Dbit;
    let Inst{21} = 1; // W = 1
    let Inst{20} = load;
    let Inst{19-16} = addr{12-9};
    let Inst{15-12} = CRd;
    let Inst{11-8} = cop;
    let Inst{7-0} = addr{7-0};
    let DecoderMethod = "DecodeCopMemInstruction";
  }
  def _POST: T2CI<op31_28,
```
- EN: Defines TableGen record `_PRE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_PRE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4463-4480
```tablegen
                  (outs), (ins p_imm:$cop, c_imm:$CRd, addr_offset_none:$addr,
                               postidx_imm8s4:$offset),
                 asm, "\t$cop, $CRd, $addr, $offset", []> {
    bits<9> offset;
    bits<4> addr;
    bits<4> cop;
    bits<4> CRd;
    let Inst{24} = 0; // P = 0
    let Inst{23} = offset{8};
    let Inst{22} = Dbit;
    let Inst{21} = 1; // W = 1
    let Inst{20} = load;
    let Inst{19-16} = addr;
    let Inst{15-12} = CRd;
    let Inst{11-8} = cop;
    let Inst{7-0} = offset{7-0};
    let DecoderMethod = "DecodeCopMemInstruction";
  }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4481-4498
```tablegen
  def _OPTION : T2CI<op31_28, (outs),
                     (ins p_imm:$cop, c_imm:$CRd, addr_offset_none:$addr,
                          coproc_option_imm:$option),
      asm, "\t$cop, $CRd, $addr, $option", []> {
    bits<8> option;
    bits<4> addr;
    bits<4> cop;
    bits<4> CRd;
    let Inst{24} = 0; // P = 0
    let Inst{23} = 1; // U = 1
    let Inst{22} = Dbit;
    let Inst{21} = 0; // W = 0
    let Inst{20} = load;
    let Inst{19-16} = addr;
    let Inst{15-12} = CRd;
    let Inst{11-8} = cop;
    let Inst{7-0} = option;
    let DecoderMethod = "DecodeCopMemInstruction";
```
- EN: Defines TableGen record `_OPTION` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_OPTION`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4499-4500
```tablegen
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4502-4515
```tablegen
let DecoderNamespace = "Thumb2CoProc" in {
let mayLoad = 1 in {
defm t2LDC   : t2LdStCop<0b1110, 1, 0, "ldc", [(int_arm_ldc timm:$cop, timm:$CRd, addrmode5:$addr)]>;
defm t2LDCL  : t2LdStCop<0b1110, 1, 1, "ldcl", [(int_arm_ldcl timm:$cop, timm:$CRd, addrmode5:$addr)]>;
defm t2LDC2  : t2LdStCop<0b1111, 1, 0, "ldc2", [(int_arm_ldc2 timm:$cop, timm:$CRd, addrmode5:$addr)]>, Requires<[PreV8,IsThumb2]>;
defm t2LDC2L : t2LdStCop<0b1111, 1, 1, "ldc2l", [(int_arm_ldc2l timm:$cop, timm:$CRd, addrmode5:$addr)]>, Requires<[PreV8,IsThumb2]>;
}
let mayStore = 1 in {
defm t2STC   : t2LdStCop<0b1110, 0, 0, "stc", [(int_arm_stc timm:$cop, timm:$CRd, addrmode5:$addr)]>;
defm t2STCL  : t2LdStCop<0b1110, 0, 1, "stcl", [(int_arm_stcl timm:$cop, timm:$CRd, addrmode5:$addr)]>;
defm t2STC2  : t2LdStCop<0b1111, 0, 0, "stc2", [(int_arm_stc2 timm:$cop, timm:$CRd, addrmode5:$addr)]>, Requires<[PreV8,IsThumb2]>;
defm t2STC2L : t2LdStCop<0b1111, 0, 1, "stc2l", [(int_arm_stc2l timm:$cop, timm:$CRd, addrmode5:$addr)]>, Requires<[PreV8,IsThumb2]>;
}
}
```
- EN: Defines TableGen record `t2LDC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4518-4521
```tablegen
//===----------------------------------------------------------------------===//
// Move between special register and ARM core register -- for disassembly only
//
// Move to ARM core register from Special Register
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 4523-4532
```tablegen
// A/R class MRS.
//
// A/R class can only move from CPSR or SPSR.
def t2MRS_AR : T2I<(outs GPR:$Rd), (ins), NoItinerary, "mrs", "\t$Rd, apsr",
                  []>, Requires<[IsThumb2,IsNotMClass]> {
  bits<4> Rd;
  let Inst{31-12} = 0b11110011111011111000;
  let Inst{11-8} = Rd;
  let Inst{7-0} = 0b00000000;
}
```
- EN: Declares reusable TableGen class `MRS` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `MRS`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4534-4534
```tablegen
def : t2InstAlias<"mrs${p} $Rd, cpsr", (t2MRS_AR GPR:$Rd, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4536-4542
```tablegen
def t2MRSsys_AR: T2I<(outs GPR:$Rd), (ins), NoItinerary, "mrs", "\t$Rd, spsr",
                   []>, Requires<[IsThumb2,IsNotMClass]> {
  bits<4> Rd;
  let Inst{31-12} = 0b11110011111111111000;
  let Inst{11-8} = Rd;
  let Inst{7-0} = 0b00000000;
}
```
- EN: Defines TableGen record `t2MRSsys_AR:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MRSsys_AR:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4544-4548
```tablegen
def t2MRSbanked : T2I<(outs rGPR:$Rd), (ins banked_reg:$banked),
                      NoItinerary, "mrs", "\t$Rd, $banked", []>,
                  Requires<[IsThumb, HasVirtualization]> {
  bits<6> banked;
  bits<4> Rd;
```
- EN: Defines TableGen record `t2MRSbanked` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MRSbanked`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4550-4558
```tablegen
  let Inst{31-21} = 0b11110011111;
  let Inst{20} = banked{5}; // R bit
  let Inst{19-16} = banked{3-0};
  let Inst{15-12} = 0b1000;
  let Inst{11-8} = Rd;
  let Inst{7-5} = 0b001;
  let Inst{4} = banked{4};
  let Inst{3-0} = 0b0000;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4561-4572
```tablegen
// M class MRS.
//
// This MRS has a mask field in bits 7-0 and can take more values than
// the A/R class (a full msr_mask).
def t2MRS_M : T2I<(outs rGPR:$Rd), (ins msr_mask:$SYSm), NoItinerary,
                  "mrs", "\t$Rd, $SYSm", []>,
              Requires<[IsThumb,IsMClass]> {
  bits<4> Rd;
  bits<8> SYSm;
  let Inst{31-12} = 0b11110011111011111000;
  let Inst{11-8} = Rd;
  let Inst{7-0} = SYSm;
```
- EN: Declares reusable TableGen class `MRS` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `MRS`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4574-4576
```tablegen
  let Unpredictable{20-16} = 0b11111;
  let Unpredictable{13} = 0b1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4579-4596
```tablegen
// Move from ARM core register to Special Register
//
// A/R class MSR.
//
// No need to have both system and application versions, the encodings are the
// same and the assembly parser has no way to distinguish between them. The mask
// operand contains the special register (R Bit) in bit 4 and bits 3-0 contains
// the mask with the fields to be accessed in the special register.
let Defs = [CPSR] in
def t2MSR_AR : T2I<(outs), (ins msr_mask:$mask, rGPR:$Rn),
                   NoItinerary, "msr", "\t$mask, $Rn", []>,
               Requires<[IsThumb2,IsNotMClass]> {
  bits<5> mask;
  bits<4> Rn;
  let Inst{31-21} = 0b11110011100;
  let Inst{20}    = mask{4}; // R Bit
  let Inst{19-16} = Rn;
  let Inst{15-12} = 0b1000;
```
- EN: Declares reusable TableGen class `MSR` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `MSR`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4597-4599
```tablegen
  let Inst{11-8}  = mask{3-0};
  let Inst{7-0}   = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4601-4607
```tablegen
// However, the MSR (banked register) system instruction (ARMv7VE) *does* have a
// separate encoding (distinguished by bit 5.
def t2MSRbanked : T2I<(outs), (ins banked_reg:$banked, rGPR:$Rn),
                      NoItinerary, "msr", "\t$banked, $Rn", []>,
                  Requires<[IsThumb, HasVirtualization]> {
  bits<6> banked;
  bits<4> Rn;
```
- EN: Defines TableGen record `t2MSRbanked` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MSRbanked`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4609-4617
```tablegen
  let Inst{31-21} = 0b11110011100;
  let Inst{20} = banked{5}; // R bit
  let Inst{19-16} = Rn;
  let Inst{15-12} = 0b1000;
  let Inst{11-8} = banked{3-0};
  let Inst{7-5} = 0b001;
  let Inst{4} = banked{4};
  let Inst{3-0} = 0b0000;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4620-4635
```tablegen
// M class MSR.
//
// Move from ARM core register to Special Register
let Defs = [CPSR] in
def t2MSR_M : T2I<(outs), (ins msr_mask:$SYSm, rGPR:$Rn),
                  NoItinerary, "msr", "\t$SYSm, $Rn", []>,
              Requires<[IsThumb,IsMClass]> {
  bits<12> SYSm;
  bits<4> Rn;
  let Inst{31-21} = 0b11110011100;
  let Inst{20}    = 0b0;
  let Inst{19-16} = Rn;
  let Inst{15-12} = 0b1000;
  let Inst{11-10} = SYSm{11-10};
  let Inst{9-8}   = 0b00;
  let Inst{7-0}   = SYSm{7-0};
```
- EN: Declares reusable TableGen class `MSR` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `MSR`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4637-4640
```tablegen
  let Unpredictable{20} = 0b1;
  let Unpredictable{13} = 0b1;
  let Unpredictable{9-8} = 0b11;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4643-4645
```tablegen
//===----------------------------------------------------------------------===//
// Move between coprocessor and ARM core register
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 4647-4653
```tablegen
class t2MovRCopro<bits<4> Op, string opc, bit direction, dag oops, dag iops,
                  list<dag> pattern>
  : T2Cop<Op, oops, iops, opc, "\t$cop, $opc1, $Rt, $CRn, $CRm, $opc2",
          pattern> {
  let Inst{27-24} = 0b1110;
  let Inst{20} = direction;
  let Inst{4} = 1;
```
- EN: Declares reusable TableGen class `t2MovRCopro` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `t2MovRCopro`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4655-4660
```tablegen
  bits<4> Rt;
  bits<4> cop;
  bits<3> opc1;
  bits<3> opc2;
  bits<4> CRm;
  bits<4> CRn;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4662-4667
```tablegen
  let Inst{15-12} = Rt;
  let Inst{11-8}  = cop;
  let Inst{23-21} = opc1;
  let Inst{7-5}   = opc2;
  let Inst{3-0}   = CRm;
  let Inst{19-16} = CRn;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4669-4670
```tablegen
  let DecoderNamespace = "Thumb2CoProc";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4672-4677
```tablegen
class t2MovRRCopro<bits<4> Op, string opc, bit direction, dag oops, dag iops,
                   list<dag> pattern = []>
  : T2Cop<Op, oops, iops, opc, "\t$cop, $opc1, $Rt, $Rt2, $CRm", pattern> {
  let Inst{27-24} = 0b1100;
  let Inst{23-21} = 0b010;
  let Inst{20} = direction;
```
- EN: Declares reusable TableGen class `t2MovRRCopro` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `t2MovRRCopro`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4679-4683
```tablegen
  bits<4> Rt;
  bits<4> Rt2;
  bits<4> cop;
  bits<4> opc1;
  bits<4> CRm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4685-4689
```tablegen
  let Inst{15-12} = Rt;
  let Inst{19-16} = Rt2;
  let Inst{11-8}  = cop;
  let Inst{7-4}   = opc1;
  let Inst{3-0}   = CRm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4691-4692
```tablegen
  let DecoderNamespace = "Thumb2CoProc";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4694-4711
```tablegen
/* from ARM core register to coprocessor */
def t2MCR : t2MovRCopro<0b1110, "mcr", 0,
           (outs),
           (ins p_imm:$cop, imm0_7:$opc1, GPR:$Rt, c_imm:$CRn,
                c_imm:$CRm, imm0_7:$opc2),
           [(int_arm_mcr timm:$cop, timm:$opc1, GPR:$Rt, timm:$CRn,
                         timm:$CRm, timm:$opc2)]>,
           ComplexDeprecationPredicate<"MCR">;
def : t2InstAlias<"mcr${p} $cop, $opc1, $Rt, $CRn, $CRm",
                  (t2MCR p_imm:$cop, imm0_7:$opc1, GPR:$Rt, c_imm:$CRn,
                         c_imm:$CRm, 0, pred:$p)>;
def t2MCR2 : t2MovRCopro<0b1111, "mcr2", 0,
             (outs), (ins p_imm:$cop, imm0_7:$opc1, GPR:$Rt, c_imm:$CRn,
                          c_imm:$CRm, imm0_7:$opc2),
             [(int_arm_mcr2 timm:$cop, timm:$opc1, GPR:$Rt, timm:$CRn,
                            timm:$CRm, timm:$opc2)]> {
  let Predicates = [IsThumb2, PreV8];
}
```
- EN: Defines TableGen record `t2MCR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MCR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4712-4714
```tablegen
def : t2InstAlias<"mcr2${p} $cop, $opc1, $Rt, $CRn, $CRm",
                  (t2MCR2 p_imm:$cop, imm0_7:$opc1, GPR:$Rt, c_imm:$CRn,
                          c_imm:$CRm, 0, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4716-4722
```tablegen
/* from coprocessor to ARM core register */
def t2MRC : t2MovRCopro<0b1110, "mrc", 1,
             (outs GPRwithAPSR:$Rt), (ins p_imm:$cop, imm0_7:$opc1, c_imm:$CRn,
                                  c_imm:$CRm, imm0_7:$opc2), []>;
def : t2InstAlias<"mrc${p} $cop, $opc1, $Rt, $CRn, $CRm",
                  (t2MRC GPRwithAPSR:$Rt, p_imm:$cop, imm0_7:$opc1, c_imm:$CRn,
                         c_imm:$CRm, 0, pred:$p)>;
```
- EN: Defines TableGen record `t2MRC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MRC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4724-4731
```tablegen
def t2MRC2 : t2MovRCopro<0b1111, "mrc2", 1,
             (outs GPRwithAPSR:$Rt), (ins p_imm:$cop, imm0_7:$opc1, c_imm:$CRn,
                                  c_imm:$CRm, imm0_7:$opc2), []> {
  let Predicates = [IsThumb2, PreV8];
}
def : t2InstAlias<"mrc2${p} $cop, $opc1, $Rt, $CRn, $CRm",
                  (t2MRC2 GPRwithAPSR:$Rt, p_imm:$cop, imm0_7:$opc1, c_imm:$CRn,
                          c_imm:$CRm, 0, pred:$p)>;
```
- EN: Defines TableGen record `t2MRC2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MRC2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4733-4734
```tablegen
def : T2v6Pat<(int_arm_mrc  timm:$cop, timm:$opc1, timm:$CRn, timm:$CRm, timm:$opc2),
              (t2MRC p_imm:$cop, imm0_7:$opc1, c_imm:$CRn, c_imm:$CRm, imm0_7:$opc2)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4736-4737
```tablegen
def : T2v6Pat<(int_arm_mrc2 timm:$cop, timm:$opc1, timm:$CRn, timm:$CRm, timm:$opc2),
              (t2MRC2 p_imm:$cop, imm0_7:$opc1, c_imm:$CRn, c_imm:$CRm, imm0_7:$opc2)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4740-4752
```tablegen
/* from ARM core register to coprocessor */
def t2MCRR : t2MovRRCopro<0b1110, "mcrr", 0, (outs),
                         (ins p_imm:$cop, imm0_15:$opc1, GPR:$Rt, GPR:$Rt2,
                         c_imm:$CRm),
                        [(int_arm_mcrr timm:$cop, timm:$opc1, GPR:$Rt, GPR:$Rt2,
                                       timm:$CRm)]>;
def t2MCRR2 : t2MovRRCopro<0b1111, "mcrr2", 0, (outs),
                          (ins p_imm:$cop, imm0_15:$opc1, GPR:$Rt, GPR:$Rt2,
                           c_imm:$CRm),
                          [(int_arm_mcrr2 timm:$cop, timm:$opc1, GPR:$Rt,
                                          GPR:$Rt2, timm:$CRm)]> {
  let Predicates = [IsThumb2, PreV8];
}
```
- EN: Defines TableGen record `t2MCRR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MCRR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4754-4756
```tablegen
/* from coprocessor to ARM core register */
def t2MRRC : t2MovRRCopro<0b1110, "mrrc", 1, (outs GPR:$Rt, GPR:$Rt2),
                          (ins p_imm:$cop, imm0_15:$opc1, c_imm:$CRm)>;
```
- EN: Defines TableGen record `t2MRRC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MRRC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4758-4761
```tablegen
def t2MRRC2 : t2MovRRCopro<0b1111, "mrrc2", 1, (outs GPR:$Rt, GPR:$Rt2),
                           (ins p_imm:$cop, imm0_15:$opc1, c_imm:$CRm)> {
  let Predicates = [IsThumb2, PreV8];
}
```
- EN: Defines TableGen record `t2MRRC2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MRRC2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4763-4765
```tablegen
//===----------------------------------------------------------------------===//
// Other Coprocessor Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 4767-4772
```tablegen
def t2CDP : T2Cop<0b1110, (outs), (ins p_imm:$cop, imm0_15:$opc1,
                 c_imm:$CRd, c_imm:$CRn, c_imm:$CRm, imm0_7:$opc2),
                 "cdp", "\t$cop, $opc1, $CRd, $CRn, $CRm, $opc2",
                 [(int_arm_cdp timm:$cop, timm:$opc1, timm:$CRd, timm:$CRn,
                               timm:$CRm, timm:$opc2)]> {
  let Inst{27-24} = 0b1110;
```
- EN: Defines TableGen record `t2CDP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2CDP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4774-4779
```tablegen
  bits<4> opc1;
  bits<4> CRn;
  bits<4> CRd;
  bits<4> cop;
  bits<3> opc2;
  bits<4> CRm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4781-4787
```tablegen
  let Inst{3-0}   = CRm;
  let Inst{4}     = 0;
  let Inst{7-5}   = opc2;
  let Inst{11-8}  = cop;
  let Inst{15-12} = CRd;
  let Inst{19-16} = CRn;
  let Inst{23-20} = opc1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4789-4791
```tablegen
  let Predicates = [IsThumb2, PreV8];
  let DecoderNamespace = "Thumb2CoProc";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4793-4798
```tablegen
def t2CDP2 : T2Cop<0b1111, (outs), (ins p_imm:$cop, imm0_15:$opc1,
                   c_imm:$CRd, c_imm:$CRn, c_imm:$CRm, imm0_7:$opc2),
                   "cdp2", "\t$cop, $opc1, $CRd, $CRn, $CRm, $opc2",
                   [(int_arm_cdp2 timm:$cop, timm:$opc1, timm:$CRd, timm:$CRn,
                                  timm:$CRm, timm:$opc2)]> {
  let Inst{27-24} = 0b1110;
```
- EN: Defines TableGen record `t2CDP2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2CDP2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4800-4805
```tablegen
  bits<4> opc1;
  bits<4> CRn;
  bits<4> CRd;
  bits<4> cop;
  bits<3> opc2;
  bits<4> CRm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4807-4813
```tablegen
  let Inst{3-0}   = CRm;
  let Inst{4}     = 0;
  let Inst{7-5}   = opc2;
  let Inst{11-8}  = cop;
  let Inst{15-12} = CRd;
  let Inst{19-16} = CRn;
  let Inst{23-20} = opc1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4815-4817
```tablegen
  let Predicates = [IsThumb2, PreV8];
  let DecoderNamespace = "Thumb2CoProc";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4820-4826
```tablegen
// Reading thread pointer from coprocessor register
def : T2Pat<(ARMthread_pointer), (t2MRC 15, 0, 13, 0, 2)>,
      Requires<[IsThumb2, IsReadTPTPIDRURW]>;
def : T2Pat<(ARMthread_pointer), (t2MRC 15, 0, 13, 0, 3)>,
      Requires<[IsThumb2, IsReadTPTPIDRURO]>;
def : T2Pat<(ARMthread_pointer), (t2MRC 15, 0, 13, 0, 4)>,
      Requires<[IsThumb2, IsReadTPTPIDRPRW]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4828-4831
```tablegen
//===----------------------------------------------------------------------===//
// ARMv8.1 Privilege Access Never extension
//
// SETPAN #imm1
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 4833-4835
```tablegen
def t2SETPAN : T1I<(outs), (ins imm0_1:$imm), NoItinerary, "setpan\t$imm", []>,
               T1Misc<0b0110000>, Requires<[IsThumb2, HasV8, HasV8_1a]> {
  bits<1> imm;
```
- EN: Defines TableGen record `t2SETPAN` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SETPAN`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4837-4839
```tablegen
  let Inst{4} = 0b1;
  let Inst{3} = imm;
  let Inst{2-0} = 0b000;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4841-4843
```tablegen
  let Unpredictable{4} = 0b1;
  let Unpredictable{2-0} = 0b111;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4845-4847
```tablegen
//===----------------------------------------------------------------------===//
// ARMv8-M Security Extensions instructions
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 4849-4853
```tablegen
let hasSideEffects = 1 in
def t2SG : T2I<(outs), (ins), NoItinerary, "sg", "", []>,
           Requires<[Has8MSecExt]> {
  let Inst = 0xe97fe97f;
}
```
- EN: Defines TableGen record `t2SG` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SG`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4855-4859
```tablegen
class T2TT<bits<2> at, string asm, list<dag> pattern>
  : T2I<(outs rGPR:$Rt), (ins GPRnopc:$Rn), NoItinerary, asm, "\t$Rt, $Rn",
        pattern> {
  bits<4> Rn;
  bits<4> Rt;
```
- EN: Declares reusable TableGen class `T2TT` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `T2TT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4861-4866
```tablegen
  let Inst{31-20} = 0b111010000100;
  let Inst{19-16} = Rn;
  let Inst{15-12} = 0b1111;
  let Inst{11-8} = Rt;
  let Inst{7-6} = at;
  let Inst{5-0} = 0b000000;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4868-4869
```tablegen
  let Unpredictable{5-0} = 0b111111;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4871-4882
```tablegen
def t2TT   : T2TT<0b00, "tt",
                 [(set rGPR:$Rt, (int_arm_cmse_tt   GPRnopc:$Rn))]>,
             Requires<[IsThumb, Has8MSecExt]>;
def t2TTT  : T2TT<0b01, "ttt",
                  [(set rGPR:$Rt, (int_arm_cmse_ttt  GPRnopc:$Rn))]>,
             Requires<[IsThumb, Has8MSecExt]>;
def t2TTA  : T2TT<0b10, "tta",
                  [(set rGPR:$Rt, (int_arm_cmse_tta  GPRnopc:$Rn))]>,
             Requires<[IsThumb, Has8MSecExt]>;
def t2TTAT : T2TT<0b11, "ttat",
                  [(set rGPR:$Rt, (int_arm_cmse_ttat GPRnopc:$Rn))]>,
             Requires<[IsThumb, Has8MSecExt]>;
```
- EN: Defines TableGen record `t2TT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2TT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4884-4886
```tablegen
//===----------------------------------------------------------------------===//
// Non-Instruction Patterns
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 4888-4902
```tablegen
// SXT/UXT with no rotate
let AddedComplexity = 16 in {
def : T2Pat<(and rGPR:$Rm, 0x000000FF), (t2UXTB rGPR:$Rm, 0)>,
           Requires<[IsThumb2]>;
def : T2Pat<(and rGPR:$Rm, 0x0000FFFF), (t2UXTH rGPR:$Rm, 0)>,
           Requires<[IsThumb2]>;
def : T2Pat<(and rGPR:$Rm, 0x00FF00FF), (t2UXTB16 rGPR:$Rm, 0)>,
           Requires<[HasDSP, IsThumb2]>;
def : T2Pat<(add rGPR:$Rn, (and rGPR:$Rm, 0x00FF)),
            (t2UXTAB rGPR:$Rn, rGPR:$Rm, 0)>,
           Requires<[HasDSP, IsThumb2]>;
def : T2Pat<(add rGPR:$Rn, (and rGPR:$Rm, 0xFFFF)),
            (t2UXTAH rGPR:$Rn, rGPR:$Rm, 0)>,
           Requires<[HasDSP, IsThumb2]>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4904-4913
```tablegen
def : T2Pat<(sext_inreg rGPR:$Src, i8),  (t2SXTB rGPR:$Src, 0)>,
           Requires<[IsThumb2]>;
def : T2Pat<(sext_inreg rGPR:$Src, i16), (t2SXTH rGPR:$Src, 0)>,
           Requires<[IsThumb2]>;
def : T2Pat<(add rGPR:$Rn, (sext_inreg rGPR:$Rm, i8)),
            (t2SXTAB rGPR:$Rn, rGPR:$Rm, 0)>,
           Requires<[HasDSP, IsThumb2]>;
def : T2Pat<(add rGPR:$Rn, (sext_inreg rGPR:$Rm, i16)),
            (t2SXTAH rGPR:$Rn, rGPR:$Rm, 0)>,
           Requires<[HasDSP, IsThumb2]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4915-4932
```tablegen
// Atomic load/store patterns
def : T2Pat<(atomic_load_azext_8   t2addrmode_imm12:$addr),
            (t2LDRBi12  t2addrmode_imm12:$addr)>;
def : T2Pat<(atomic_load_azext_8   t2addrmode_negimm8:$addr),
            (t2LDRBi8   t2addrmode_negimm8:$addr)>;
def : T2Pat<(atomic_load_azext_8   t2addrmode_so_reg:$addr),
            (t2LDRBs    t2addrmode_so_reg:$addr)>;
def : T2Pat<(atomic_load_azext_16  t2addrmode_imm12:$addr),
            (t2LDRHi12  t2addrmode_imm12:$addr)>;
def : T2Pat<(atomic_load_azext_16  t2addrmode_negimm8:$addr),
            (t2LDRHi8   t2addrmode_negimm8:$addr)>;
def : T2Pat<(atomic_load_azext_16  t2addrmode_so_reg:$addr),
            (t2LDRHs    t2addrmode_so_reg:$addr)>;
def : T2Pat<(atomic_load_nonext_32 t2addrmode_imm12:$addr),
            (t2LDRi12   t2addrmode_imm12:$addr)>;
def : T2Pat<(atomic_load_nonext_32 t2addrmode_negimm8:$addr),
            (t2LDRi8    t2addrmode_negimm8:$addr)>;
def : T2Pat<(atomic_load_nonext_32 t2addrmode_so_reg:$addr),
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4933-4950
```tablegen
            (t2LDRs     t2addrmode_so_reg:$addr)>;
def : T2Pat<(atomic_store_8  GPR:$val, t2addrmode_imm12:$addr),
            (t2STRBi12  GPR:$val, t2addrmode_imm12:$addr)>;
def : T2Pat<(atomic_store_8  GPR:$val, t2addrmode_negimm8:$addr),
            (t2STRBi8   GPR:$val, t2addrmode_negimm8:$addr)>;
def : T2Pat<(atomic_store_8  GPR:$val, t2addrmode_so_reg:$addr),
            (t2STRBs    GPR:$val, t2addrmode_so_reg:$addr)>;
def : T2Pat<(atomic_store_16 GPR:$val, t2addrmode_imm12:$addr),
            (t2STRHi12  GPR:$val, t2addrmode_imm12:$addr)>;
def : T2Pat<(atomic_store_16 GPR:$val, t2addrmode_negimm8:$addr),
            (t2STRHi8   GPR:$val, t2addrmode_negimm8:$addr)>;
def : T2Pat<(atomic_store_16 GPR:$val, t2addrmode_so_reg:$addr),
            (t2STRHs    GPR:$val, t2addrmode_so_reg:$addr)>;
def : T2Pat<(atomic_store_32 GPR:$val,t2addrmode_imm12:$addr),
            (t2STRi12   GPR:$val, t2addrmode_imm12:$addr)>;
def : T2Pat<(atomic_store_32 GPR:$val, t2addrmode_negimm8:$addr),
            (t2STRi8    GPR:$val, t2addrmode_negimm8:$addr)>;
def : T2Pat<(atomic_store_32 GPR:$val, t2addrmode_so_reg:$addr),
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4951-4951
```tablegen
            (t2STRs     GPR:$val, t2addrmode_so_reg:$addr)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4953-4960
```tablegen
let AddedComplexity = 8, Predicates = [IsThumb, HasAcquireRelease, HasV7Clrex] in {
  def : Pat<(atomic_load_azext_acquire_8 addr_offset_none:$addr),  (t2LDAB addr_offset_none:$addr)>;
  def : Pat<(atomic_load_azext_acquire_16 addr_offset_none:$addr), (t2LDAH addr_offset_none:$addr)>;
  def : Pat<(atomic_load_nonext_acquire_32 addr_offset_none:$addr), (t2LDA  addr_offset_none:$addr)>;
  def : Pat<(atomic_store_release_8 addr_offset_none:$addr, GPR:$val),  (t2STLB GPR:$val, addr_offset_none:$addr)>;
  def : Pat<(atomic_store_release_16 addr_offset_none:$addr, GPR:$val), (t2STLH GPR:$val, addr_offset_none:$addr)>;
  def : Pat<(atomic_store_release_32 addr_offset_none:$addr, GPR:$val), (t2STL  GPR:$val, addr_offset_none:$addr)>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4963-4965
```tablegen
//===----------------------------------------------------------------------===//
// Assembler aliases
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 4967-4977
```tablegen
// Aliases for ADC without the ".w" optional width specifier.
def : t2InstAlias<"adc${s}${p} $Rd, $Rn, $Rm",
                  (t2ADCrr rGPR:$Rd, rGPR:$Rn, rGPR:$Rm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"adc${s}${p} $Rd, $Rn, $ShiftedRm",
                  (t2ADCrs rGPR:$Rd, rGPR:$Rn, t2_so_reg:$ShiftedRm,
                           pred:$p, cc_out:$s)>;
def : t2InstAlias<"adc${s}${p} $Rdn, $Rm",
                  (t2ADCrr rGPR:$Rdn, rGPR:$Rdn, rGPR:$Rm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"adc${s}${p} $Rdn, $ShiftedRm",
                  (t2ADCrs rGPR:$Rdn, rGPR:$Rdn, t2_so_reg:$ShiftedRm,
                           pred:$p, cc_out:$s)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4979-4984
```tablegen
// Aliases for SBC without the ".w" optional width specifier.
def : t2InstAlias<"sbc${s}${p} $Rd, $Rn, $Rm",
                  (t2SBCrr rGPR:$Rd, rGPR:$Rn, rGPR:$Rm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"sbc${s}${p} $Rd, $Rn, $ShiftedRm",
                  (t2SBCrs rGPR:$Rd, rGPR:$Rn, t2_so_reg:$ShiftedRm,
                           pred:$p, cc_out:$s)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4986-5003
```tablegen
// Aliases for ADD without the ".w" optional width specifier.
def : t2InstAlias<"add${s}${p} $Rd, $Rn, $imm",
        (t2ADDri rGPR:$Rd, GPRnopc:$Rn, t2_so_imm:$imm, pred:$p,
         cc_out:$s)>;
def : t2InstAlias<"add${p} $Rd, $Rn, $imm",
           (t2ADDri12 rGPR:$Rd, GPR:$Rn, imm0_4095:$imm, pred:$p)>;
def : t2InstAlias<"add${s}${p} $Rd, $Rn, $Rm",
              (t2ADDrr GPRnopc:$Rd, GPRnopc:$Rn, rGPR:$Rm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"add${s}${p} $Rd, $Rn, $ShiftedRm",
                  (t2ADDrs GPRnopc:$Rd, GPRnopc:$Rn, t2_so_reg:$ShiftedRm,
                           pred:$p, cc_out:$s)>;
// ... and with the destination and source register combined.
def : t2InstAlias<"add${s}${p} $Rdn, $imm",
      (t2ADDri rGPR:$Rdn, rGPR:$Rdn, t2_so_imm:$imm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"add${p} $Rdn, $imm",
           (t2ADDri12 rGPR:$Rdn, rGPR:$Rdn, imm0_4095:$imm, pred:$p)>;
def : t2InstAlias<"addw${p} $Rdn, $imm",
           (t2ADDri12 rGPR:$Rdn, rGPR:$Rdn, imm0_4095:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5004-5008
```tablegen
def : t2InstAlias<"add${s}${p} $Rdn, $Rm",
            (t2ADDrr GPRnopc:$Rdn, GPRnopc:$Rdn, rGPR:$Rm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"add${s}${p} $Rdn, $ShiftedRm",
                  (t2ADDrs GPRnopc:$Rdn, GPRnopc:$Rdn, t2_so_reg:$ShiftedRm,
                           pred:$p, cc_out:$s)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5010-5020
```tablegen
// add w/ negative immediates is just a sub.
def : t2InstSubst<"add${s}${p} $Rd, $Rn, $imm",
        (t2SUBri rGPR:$Rd, GPRnopc:$Rn, t2_so_imm_neg:$imm, pred:$p,
                 cc_out:$s)>;
def : t2InstSubst<"add${p} $Rd, $Rn, $imm",
           (t2SUBri12 rGPR:$Rd, GPR:$Rn, imm0_4095_neg:$imm, pred:$p)>;
def : t2InstSubst<"add${s}${p} $Rdn, $imm",
      (t2SUBri rGPR:$Rdn, rGPR:$Rdn, t2_so_imm_neg:$imm, pred:$p,
               cc_out:$s)>;
def : t2InstSubst<"add${p} $Rdn, $imm",
           (t2SUBri12 rGPR:$Rdn, rGPR:$Rdn, imm0_4095_neg:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5022-5031
```tablegen
def : t2InstSubst<"add${s}${p}.w $Rd, $Rn, $imm",
        (t2SUBri rGPR:$Rd, GPRnopc:$Rn, t2_so_imm_neg:$imm, pred:$p,
                 cc_out:$s)>;
def : t2InstSubst<"addw${p} $Rd, $Rn, $imm",
           (t2SUBri12 rGPR:$Rd, rGPR:$Rn, imm0_4095_neg:$imm, pred:$p)>;
def : t2InstSubst<"add${s}${p}.w $Rdn, $imm",
      (t2SUBri rGPR:$Rdn, rGPR:$Rdn, t2_so_imm_neg:$imm, pred:$p,
               cc_out:$s)>;
def : t2InstSubst<"addw${p} $Rdn, $imm",
           (t2SUBri12 rGPR:$Rdn, rGPR:$Rdn, imm0_4095_neg:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5034-5051
```tablegen
// Aliases for SUB without the ".w" optional width specifier.
def : t2InstAlias<"sub${s}${p} $Rd, $Rn, $imm",
        (t2SUBri rGPR:$Rd, GPRnopc:$Rn, t2_so_imm:$imm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"sub${p} $Rd, $Rn, $imm",
           (t2SUBri12 rGPR:$Rd, GPR:$Rn, imm0_4095:$imm, pred:$p)>;
def : t2InstAlias<"sub${s}${p} $Rd, $Rn, $Rm",
              (t2SUBrr GPRnopc:$Rd, GPRnopc:$Rn, rGPR:$Rm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"sub${s}${p} $Rd, $Rn, $ShiftedRm",
                  (t2SUBrs GPRnopc:$Rd, GPRnopc:$Rn, t2_so_reg:$ShiftedRm,
                           pred:$p, cc_out:$s)>;
// ... and with the destination and source register combined.
def : t2InstAlias<"sub${s}${p} $Rdn, $imm",
      (t2SUBri rGPR:$Rdn, rGPR:$Rdn, t2_so_imm:$imm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"sub${p} $Rdn, $imm",
           (t2SUBri12 rGPR:$Rdn, rGPR:$Rdn, imm0_4095:$imm, pred:$p)>;
def : t2InstAlias<"subw${p} $Rdn, $imm",
           (t2SUBri12 rGPR:$Rdn, rGPR:$Rdn, imm0_4095:$imm, pred:$p)>;
def : t2InstAlias<"sub${s}${p}.w $Rdn, $Rm",
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5052-5057
```tablegen
            (t2SUBrr GPRnopc:$Rdn, GPRnopc:$Rdn, rGPR:$Rm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"sub${s}${p} $Rdn, $Rm",
            (t2SUBrr GPRnopc:$Rdn, GPRnopc:$Rdn, rGPR:$Rm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"sub${s}${p} $Rdn, $ShiftedRm",
                  (t2SUBrs GPRnopc:$Rdn, GPRnopc:$Rdn, t2_so_reg:$ShiftedRm,
                           pred:$p, cc_out:$s)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5059-5068
```tablegen
// SP to SP alike aliases
// Aliases for ADD without the ".w" optional width specifier.
def : t2InstAlias<"add${s}${p} $Rd, $Rn, $imm",
        (t2ADDspImm GPRsp:$Rd, GPRsp:$Rn, t2_so_imm:$imm, pred:$p,
         cc_out:$s)>;
def : t2InstAlias<"add${p} $Rd, $Rn, $imm",
           (t2ADDspImm12 GPRsp:$Rd, GPRsp:$Rn, imm0_4095:$imm, pred:$p)>;
// ... and with the destination and source register combined.
def : t2InstAlias<"add${s}${p} $Rdn, $imm",
      (t2ADDspImm GPRsp:$Rdn, GPRsp:$Rdn, t2_so_imm:$imm, pred:$p, cc_out:$s)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5070-5071
```tablegen
def : t2InstAlias<"add${s}${p}.w $Rdn, $imm",
      (t2ADDspImm GPRsp:$Rdn, GPRsp:$Rdn, t2_so_imm:$imm, pred:$p, cc_out:$s)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5073-5074
```tablegen
def : t2InstAlias<"add${p} $Rdn, $imm",
           (t2ADDspImm12 GPRsp:$Rdn, GPRsp:$Rdn, imm0_4095:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5076-5077
```tablegen
def : t2InstAlias<"addw${p} $Rdn, $imm",
           (t2ADDspImm12 GPRsp:$Rdn, GPRsp:$Rdn, imm0_4095:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5079-5089
```tablegen
// add w/ negative immediates is just a sub.
def : t2InstSubst<"add${s}${p} $Rd, $Rn, $imm",
        (t2SUBspImm GPRsp:$Rd, GPRsp:$Rn, t2_so_imm_neg:$imm, pred:$p,
                 cc_out:$s)>;
def : t2InstSubst<"add${p} $Rd, $Rn, $imm",
           (t2SUBspImm12 GPRsp:$Rd, GPRsp:$Rn, imm0_4095_neg:$imm, pred:$p)>;
def : t2InstSubst<"add${s}${p} $Rdn, $imm",
      (t2SUBspImm GPRsp:$Rdn, GPRsp:$Rdn, t2_so_imm_neg:$imm, pred:$p,
               cc_out:$s)>;
def : t2InstSubst<"add${p} $Rdn, $imm",
           (t2SUBspImm12 GPRsp:$Rdn, GPRsp:$Rdn, imm0_4095_neg:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5091-5100
```tablegen
def : t2InstSubst<"add${s}${p}.w $Rd, $Rn, $imm",
        (t2SUBspImm GPRsp:$Rd, GPRsp:$Rn, t2_so_imm_neg:$imm, pred:$p,
                 cc_out:$s)>;
def : t2InstSubst<"addw${p} $Rd, $Rn, $imm",
           (t2SUBspImm12 GPRsp:$Rd, GPRsp:$Rn, imm0_4095_neg:$imm, pred:$p)>;
def : t2InstSubst<"add${s}${p}.w $Rdn, $imm",
      (t2SUBspImm GPRsp:$Rdn, GPRsp:$Rdn, t2_so_imm_neg:$imm, pred:$p,
               cc_out:$s)>;
def : t2InstSubst<"addw${p} $Rdn, $imm",
           (t2SUBspImm12 GPRsp:$Rdn, GPRsp:$Rdn, imm0_4095_neg:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5103-5116
```tablegen
// Aliases for SUB without the ".w" optional width specifier.
def : t2InstAlias<"sub${s}${p} $Rd, $Rn, $imm",
        (t2SUBspImm GPRsp:$Rd, GPRsp:$Rn, t2_so_imm:$imm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"sub${p} $Rd, $Rn, $imm",
           (t2SUBspImm12 GPRsp:$Rd, GPRsp:$Rn, imm0_4095:$imm, pred:$p)>;
// ... and with the destination and source register combined.
def : t2InstAlias<"sub${s}${p} $Rdn, $imm",
      (t2SUBspImm GPRsp:$Rdn, GPRsp:$Rdn, t2_so_imm:$imm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"sub${s}${p}.w $Rdn, $imm",
      (t2SUBspImm GPRsp:$Rdn, GPRsp:$Rdn, t2_so_imm:$imm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"sub${p} $Rdn, $imm",
           (t2SUBspImm12 GPRsp:$Rdn, GPRsp:$Rdn, imm0_4095:$imm, pred:$p)>;
def : t2InstAlias<"subw${p} $Rdn, $imm",
           (t2SUBspImm12 GPRsp:$Rdn, GPRsp:$Rdn, imm0_4095:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5118-5124
```tablegen
// Alias for compares without the ".w" optional width specifier.
def : t2InstAlias<"cmn${p} $Rn, $Rm",
                  (t2CMNrr GPRnopc:$Rn, rGPR:$Rm, pred:$p)>;
def : t2InstAlias<"teq${p} $Rn, $Rm",
                  (t2TEQrr rGPR:$Rn, rGPR:$Rm, pred:$p)>;
def : t2InstAlias<"tst${p} $Rn, $Rm",
                  (t2TSTrr rGPR:$Rn, rGPR:$Rm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5126-5135
```tablegen
// Memory barriers
def : InstAlias<"dmb${p}.w\t$opt", (t2DMB memb_opt:$opt, pred:$p), 0>, Requires<[HasDB]>;
def : InstAlias<"dmb${p}", (t2DMB 0xf, pred:$p), 0>, Requires<[HasDB]>;
def : InstAlias<"dmb${p}.w", (t2DMB 0xf, pred:$p), 0>, Requires<[HasDB]>;
def : InstAlias<"dsb${p}.w\t$opt", (t2DSB memb_opt:$opt, pred:$p), 0>, Requires<[HasDB]>;
def : InstAlias<"dsb${p}", (t2DSB 0xf, pred:$p), 0>, Requires<[HasDB]>;
def : InstAlias<"dsb${p}.w", (t2DSB 0xf, pred:$p), 0>, Requires<[HasDB]>;
def : InstAlias<"isb${p}.w\t$opt", (t2ISB instsyncb_opt:$opt, pred:$p), 0>, Requires<[HasDB]>;
def : InstAlias<"isb${p}", (t2ISB 0xf, pred:$p), 0>, Requires<[HasDB]>;
def : InstAlias<"isb${p}.w", (t2ISB 0xf, pred:$p), 0>, Requires<[HasDB]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5137-5142
```tablegen
// Non-predicable aliases of a predicable DSB: the predicate is (14, zero_reg) where
// 14 = AL (always execute) and zero_reg = "instruction doesn't read the CPSR".
def : InstAlias<"ssbb", (t2DSB 0x0, (pred 14, zero_reg)), 1>,
      Requires<[HasDB, IsThumb2]>;
def : InstAlias<"pssbb", (t2DSB 0x4, (pred 14, zero_reg)), 1>,
      Requires<[HasDB, IsThumb2]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5144-5145
```tablegen
// Armv8-R 'Data Full Barrier'
def : InstAlias<"dfb${p}", (t2DSB 0xc, pred:$p), 1>, Requires<[HasDFB]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5147-5158
```tablegen
// SpeculationBarrierEndBB must only be used after an unconditional control
// flow, i.e. after a terminator for which isBarrier is True.
let hasSideEffects = 1, isCodeGenOnly = 1, isTerminator = 1, isBarrier = 1 in {
  // This gets lowered to a pair of 4-byte instructions
  let Size = 8 in
  def t2SpeculationBarrierISBDSBEndBB
      : PseudoInst<(outs), (ins), NoItinerary, []>, Sched<[]>;
  // This gets lowered to a single 4-byte instructions
  let Size = 4 in
  def t2SpeculationBarrierSBEndBB
      : PseudoInst<(outs), (ins), NoItinerary, []>, Sched<[]>;
}
```
- EN: Defines TableGen record `t2SpeculationBarrierISBDSBEndBB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2SpeculationBarrierISBDSBEndBB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5160-5171
```tablegen
// Alias for LDR, LDRB, LDRH, LDRSB, and LDRSH without the ".w" optional
// width specifier.
def : t2InstAlias<"ldr${p} $Rt, $addr",
                  (t2LDRi12 GPR:$Rt, t2addrmode_imm12:$addr, pred:$p)>;
def : t2InstAlias<"ldrb${p} $Rt, $addr",
                  (t2LDRBi12 rGPR:$Rt, t2addrmode_imm12:$addr, pred:$p)>;
def : t2InstAlias<"ldrh${p} $Rt, $addr",
                  (t2LDRHi12 rGPR:$Rt, t2addrmode_imm12:$addr, pred:$p)>;
def : t2InstAlias<"ldrsb${p} $Rt, $addr",
                  (t2LDRSBi12 rGPR:$Rt, t2addrmode_imm12:$addr, pred:$p)>;
def : t2InstAlias<"ldrsh${p} $Rt, $addr",
                  (t2LDRSHi12 rGPR:$Rt, t2addrmode_imm12:$addr, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5173-5182
```tablegen
def : t2InstAlias<"ldr${p} $Rt, $addr",
                  (t2LDRs GPR:$Rt, t2addrmode_so_reg:$addr, pred:$p)>;
def : t2InstAlias<"ldrb${p} $Rt, $addr",
                  (t2LDRBs rGPR:$Rt, t2addrmode_so_reg:$addr, pred:$p)>;
def : t2InstAlias<"ldrh${p} $Rt, $addr",
                  (t2LDRHs rGPR:$Rt, t2addrmode_so_reg:$addr, pred:$p)>;
def : t2InstAlias<"ldrsb${p} $Rt, $addr",
                  (t2LDRSBs rGPR:$Rt, t2addrmode_so_reg:$addr, pred:$p)>;
def : t2InstAlias<"ldrsh${p} $Rt, $addr",
                  (t2LDRSHs rGPR:$Rt, t2addrmode_so_reg:$addr, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5184-5193
```tablegen
def : t2InstAlias<"ldr${p} $Rt, $addr",
                  (t2LDRpci GPR:$Rt, t2ldrlabel:$addr, pred:$p)>;
def : t2InstAlias<"ldrb${p} $Rt, $addr",
                  (t2LDRBpci rGPR:$Rt, t2ldrlabel:$addr, pred:$p)>;
def : t2InstAlias<"ldrh${p} $Rt, $addr",
                  (t2LDRHpci rGPR:$Rt, t2ldrlabel:$addr, pred:$p)>;
def : t2InstAlias<"ldrsb${p} $Rt, $addr",
                  (t2LDRSBpci rGPR:$Rt, t2ldrlabel:$addr, pred:$p)>;
def : t2InstAlias<"ldrsh${p} $Rt, $addr",
                  (t2LDRSHpci rGPR:$Rt, t2ldrlabel:$addr, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5195-5201
```tablegen
// Alias for MVN with(out) the ".w" optional width specifier.
def : t2InstAlias<"mvn${s}${p}.w $Rd, $imm",
           (t2MVNi rGPR:$Rd, t2_so_imm:$imm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"mvn${s}${p} $Rd, $Rm",
           (t2MVNr rGPR:$Rd, rGPR:$Rm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"mvn${s}${p} $Rd, $ShiftedRm",
           (t2MVNs rGPR:$Rd, t2_so_reg:$ShiftedRm, pred:$p, cc_out:$s)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5203-5210
```tablegen
// PKHBT/PKHTB with default shift amount. PKHTB is equivalent to PKHBT with the
// input operands swapped when the shift amount is zero (i.e., unspecified).
def : InstAlias<"pkhbt${p} $Rd, $Rn, $Rm",
                (t2PKHBT rGPR:$Rd, rGPR:$Rn, rGPR:$Rm, 0, pred:$p), 0>,
            Requires<[HasDSP, IsThumb2]>;
def : InstAlias<"pkhtb${p} $Rd, $Rn, $Rm",
                (t2PKHBT rGPR:$Rd, rGPR:$Rm, rGPR:$Rn, 0, pred:$p), 0>,
            Requires<[HasDSP, IsThumb2]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5212-5216
```tablegen
// PUSH/POP aliases for STM/LDM
def : t2InstAlias<"push${p}.w $regs", (t2STMDB_UPD SP, pred:$p, reglist:$regs)>;
def : t2InstAlias<"push${p} $regs", (t2STMDB_UPD SP, pred:$p, reglist:$regs)>;
def : t2InstAlias<"pop${p}.w $regs", (t2LDMIA_UPD SP, pred:$p, reglist:$regs)>;
def : t2InstAlias<"pop${p} $regs", (t2LDMIA_UPD SP, pred:$p, reglist:$regs)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5218-5222
```tablegen
// STMIA/STMIA_UPD aliases w/o the optional .w suffix
def : t2InstAlias<"stm${p} $Rn, $regs",
                  (t2STMIA GPR:$Rn, pred:$p, reglist:$regs)>;
def : t2InstAlias<"stm${p} $Rn!, $regs",
                  (t2STMIA_UPD GPR:$Rn, pred:$p, reglist:$regs)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5224-5228
```tablegen
// LDMIA/LDMIA_UPD aliases w/o the optional .w suffix
def : t2InstAlias<"ldm${p} $Rn, $regs",
                  (t2LDMIA GPR:$Rn, pred:$p, reglist:$regs)>;
def : t2InstAlias<"ldm${p} $Rn!, $regs",
                  (t2LDMIA_UPD GPR:$Rn, pred:$p, reglist:$regs)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5230-5234
```tablegen
// STMDB/STMDB_UPD aliases w/ the optional .w suffix
def : t2InstAlias<"stmdb${p}.w $Rn, $regs",
                  (t2STMDB GPR:$Rn, pred:$p, reglist:$regs)>;
def : t2InstAlias<"stmdb${p}.w $Rn!, $regs",
                  (t2STMDB_UPD GPR:$Rn, pred:$p, reglist:$regs)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5236-5240
```tablegen
// LDMDB/LDMDB_UPD aliases w/ the optional .w suffix
def : t2InstAlias<"ldmdb${p}.w $Rn, $regs",
                  (t2LDMDB GPR:$Rn, pred:$p, reglist:$regs)>;
def : t2InstAlias<"ldmdb${p}.w $Rn!, $regs",
                  (t2LDMDB_UPD GPR:$Rn, pred:$p, reglist:$regs)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5242-5245
```tablegen
// Alias for REV/REV16/REVSH without the ".w" optional width specifier.
def : t2InstAlias<"rev${p} $Rd, $Rm", (t2REV rGPR:$Rd, rGPR:$Rm, pred:$p)>;
def : t2InstAlias<"rev16${p} $Rd, $Rm", (t2REV16 rGPR:$Rd, rGPR:$Rm, pred:$p)>;
def : t2InstAlias<"revsh${p} $Rd, $Rm", (t2REVSH rGPR:$Rd, rGPR:$Rm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5248-5265
```tablegen
// Alias for RSB with and without the ".w" optional width specifier, with and
// without explicit destination register.
def : t2InstAlias<"rsb${s}${p} $Rd, $Rn, $imm",
           (t2RSBri rGPR:$Rd, rGPR:$Rn, t2_so_imm:$imm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"rsb${s}${p} $Rdn, $imm",
           (t2RSBri rGPR:$Rdn, rGPR:$Rdn, t2_so_imm:$imm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"rsb${s}${p} $Rdn, $Rm",
           (t2RSBrr rGPR:$Rdn, rGPR:$Rdn, rGPR:$Rm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"rsb${s}${p} $Rdn, $ShiftedRm",
           (t2RSBrs rGPR:$Rdn, rGPR:$Rdn, t2_so_reg:$ShiftedRm, pred:$p,
                    cc_out:$s)>;
def : t2InstAlias<"rsb${s}${p}.w $Rdn, $Rm",
           (t2RSBrr rGPR:$Rdn, rGPR:$Rdn, rGPR:$Rm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"rsb${s}${p}.w $Rd, $Rn, $Rm",
           (t2RSBrr rGPR:$Rd, rGPR:$Rn, rGPR:$Rm, pred:$p, cc_out:$s)>;
def : t2InstAlias<"rsb${s}${p}.w $Rd, $Rn, $ShiftedRm",
           (t2RSBrs rGPR:$Rd, rGPR:$Rn, t2_so_reg:$ShiftedRm, pred:$p,
                    cc_out:$s)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5267-5271
```tablegen
// SSAT/USAT optional shift operand.
def : t2InstAlias<"ssat${p} $Rd, $sat_imm, $Rn",
                  (t2SSAT rGPR:$Rd, imm1_32:$sat_imm, rGPR:$Rn, 0, pred:$p)>;
def : t2InstAlias<"usat${p} $Rd, $sat_imm, $Rn",
                  (t2USAT rGPR:$Rd, imm0_31:$sat_imm, rGPR:$Rn, 0, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5273-5275
```tablegen
// STM w/o the .w suffix.
def : t2InstAlias<"stm${p} $Rn, $regs",
                  (t2STMIA GPR:$Rn, pred:$p, reglist:$regs)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5277-5284
```tablegen
// Alias for STR, STRB, and STRH without the ".w" optional
// width specifier.
def : t2InstAlias<"str${p} $Rt, $addr",
                  (t2STRi12 GPR:$Rt, t2addrmode_imm12:$addr, pred:$p)>;
def : t2InstAlias<"strb${p} $Rt, $addr",
                  (t2STRBi12 rGPR:$Rt, t2addrmode_imm12:$addr, pred:$p)>;
def : t2InstAlias<"strh${p} $Rt, $addr",
                  (t2STRHi12 rGPR:$Rt, t2addrmode_imm12:$addr, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5286-5291
```tablegen
def : t2InstAlias<"str${p} $Rt, $addr",
                  (t2STRs GPR:$Rt, t2addrmode_so_reg:$addr, pred:$p)>;
def : t2InstAlias<"strb${p} $Rt, $addr",
                  (t2STRBs rGPR:$Rt, t2addrmode_so_reg:$addr, pred:$p)>;
def : t2InstAlias<"strh${p} $Rt, $addr",
                  (t2STRHs rGPR:$Rt, t2addrmode_so_reg:$addr, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5293-5305
```tablegen
// Extend instruction optional rotate operand.
def : InstAlias<"sxtab${p} $Rd, $Rn, $Rm",
              (t2SXTAB rGPR:$Rd, rGPR:$Rn, rGPR:$Rm, 0, pred:$p), 0>,
              Requires<[HasDSP, IsThumb2]>;
def : InstAlias<"sxtah${p} $Rd, $Rn, $Rm",
              (t2SXTAH rGPR:$Rd, rGPR:$Rn, rGPR:$Rm, 0, pred:$p), 0>,
              Requires<[HasDSP, IsThumb2]>;
def : InstAlias<"sxtab16${p} $Rd, $Rn, $Rm",
              (t2SXTAB16 rGPR:$Rd, rGPR:$Rn, rGPR:$Rm, 0, pred:$p), 0>,
              Requires<[HasDSP, IsThumb2]>;
def : InstAlias<"sxtb16${p} $Rd, $Rm",
              (t2SXTB16 rGPR:$Rd, rGPR:$Rm, 0, pred:$p), 0>,
              Requires<[HasDSP, IsThumb2]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5307-5314
```tablegen
def : t2InstAlias<"sxtb${p} $Rd, $Rm",
                (t2SXTB rGPR:$Rd, rGPR:$Rm, 0, pred:$p)>;
def : t2InstAlias<"sxth${p} $Rd, $Rm",
                (t2SXTH rGPR:$Rd, rGPR:$Rm, 0, pred:$p)>;
def : t2InstAlias<"sxtb${p}.w $Rd, $Rm",
                (t2SXTB rGPR:$Rd, rGPR:$Rm, 0, pred:$p)>;
def : t2InstAlias<"sxth${p}.w $Rd, $Rm",
                (t2SXTH rGPR:$Rd, rGPR:$Rm, 0, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5316-5327
```tablegen
def : InstAlias<"uxtab${p} $Rd, $Rn, $Rm",
              (t2UXTAB rGPR:$Rd, rGPR:$Rn, rGPR:$Rm, 0, pred:$p), 0>,
              Requires<[HasDSP, IsThumb2]>;
def : InstAlias<"uxtah${p} $Rd, $Rn, $Rm",
              (t2UXTAH rGPR:$Rd, rGPR:$Rn, rGPR:$Rm, 0, pred:$p), 0>,
              Requires<[HasDSP, IsThumb2]>;
def : InstAlias<"uxtab16${p} $Rd, $Rn, $Rm",
              (t2UXTAB16 rGPR:$Rd, rGPR:$Rn, rGPR:$Rm, 0, pred:$p), 0>,
              Requires<[HasDSP, IsThumb2]>;
def : InstAlias<"uxtb16${p} $Rd, $Rm",
              (t2UXTB16 rGPR:$Rd, rGPR:$Rm, 0, pred:$p), 0>,
              Requires<[HasDSP, IsThumb2]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5329-5336
```tablegen
def : t2InstAlias<"uxtb${p} $Rd, $Rm",
                (t2UXTB rGPR:$Rd, rGPR:$Rm, 0, pred:$p)>;
def : t2InstAlias<"uxth${p} $Rd, $Rm",
                (t2UXTH rGPR:$Rd, rGPR:$Rm, 0, pred:$p)>;
def : t2InstAlias<"uxtb${p}.w $Rd, $Rm",
                (t2UXTB rGPR:$Rd, rGPR:$Rm, 0, pred:$p)>;
def : t2InstAlias<"uxth${p}.w $Rd, $Rm",
                (t2UXTH rGPR:$Rd, rGPR:$Rm, 0, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5338-5345
```tablegen
// Extend instruction w/o the ".w" optional width specifier.
def : t2InstAlias<"uxtb${p} $Rd, $Rm$rot",
                  (t2UXTB rGPR:$Rd, rGPR:$Rm, rot_imm:$rot, pred:$p)>;
def : InstAlias<"uxtb16${p} $Rd, $Rm$rot",
                (t2UXTB16 rGPR:$Rd, rGPR:$Rm, rot_imm:$rot, pred:$p), 0>,
                Requires<[HasDSP, IsThumb2]>;
def : t2InstAlias<"uxth${p} $Rd, $Rm$rot",
                  (t2UXTH rGPR:$Rd, rGPR:$Rm, rot_imm:$rot, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5347-5353
```tablegen
def : t2InstAlias<"sxtb${p} $Rd, $Rm$rot",
                  (t2SXTB rGPR:$Rd, rGPR:$Rm, rot_imm:$rot, pred:$p)>;
def : InstAlias<"sxtb16${p} $Rd, $Rm$rot",
                (t2SXTB16 rGPR:$Rd, rGPR:$Rm, rot_imm:$rot, pred:$p), 0>,
                Requires<[HasDSP, IsThumb2]>;
def : t2InstAlias<"sxth${p} $Rd, $Rm$rot",
                  (t2SXTH rGPR:$Rd, rGPR:$Rm, rot_imm:$rot, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5356-5373
```tablegen
// "mov Rd, t2_so_imm_not" can be handled via "mvn" in assembly, just like
// for isel.
def : t2InstSubst<"mov${p} $Rd, $imm",
                  (t2MVNi rGPR:$Rd, t2_so_imm_not:$imm, pred:$p,
                          (cc_out zero_reg))>;
def : t2InstSubst<"mvn${s}${p} $Rd, $imm",
                  (t2MOVi rGPR:$Rd, t2_so_imm_not:$imm, pred:$p, s_cc_out:$s)>;
// Same for AND <--> BIC
def : t2InstSubst<"bic${s}${p} $Rd, $Rn, $imm",
                  (t2ANDri rGPR:$Rd, rGPR:$Rn, t2_so_imm_not:$imm,
                           pred:$p, cc_out:$s)>;
def : t2InstSubst<"bic${s}${p} $Rdn, $imm",
                  (t2ANDri rGPR:$Rdn, rGPR:$Rdn, t2_so_imm_not:$imm,
                           pred:$p, cc_out:$s)>;
def : t2InstSubst<"bic${s}${p}.w $Rd, $Rn, $imm",
                  (t2ANDri rGPR:$Rd, rGPR:$Rn, t2_so_imm_not:$imm,
                           pred:$p, cc_out:$s)>;
def : t2InstSubst<"bic${s}${p}.w $Rdn, $imm",
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5374-5391
```tablegen
                  (t2ANDri rGPR:$Rdn, rGPR:$Rdn, t2_so_imm_not:$imm,
                           pred:$p, cc_out:$s)>;
def : t2InstSubst<"and${s}${p} $Rd, $Rn, $imm",
                  (t2BICri rGPR:$Rd, rGPR:$Rn, t2_so_imm_not:$imm,
                           pred:$p, cc_out:$s)>;
def : t2InstSubst<"and${s}${p} $Rdn, $imm",
                  (t2BICri rGPR:$Rdn, rGPR:$Rdn, t2_so_imm_not:$imm,
                           pred:$p, cc_out:$s)>;
def : t2InstSubst<"and${s}${p}.w $Rd, $Rn, $imm",
                  (t2BICri rGPR:$Rd, rGPR:$Rn, t2_so_imm_not:$imm,
                           pred:$p, cc_out:$s)>;
def : t2InstSubst<"and${s}${p}.w $Rdn, $imm",
                  (t2BICri rGPR:$Rdn, rGPR:$Rdn, t2_so_imm_not:$imm,
                           pred:$p, cc_out:$s)>;
// And ORR <--> ORN
def : t2InstSubst<"orn${s}${p} $Rd, $Rn, $imm",
                  (t2ORRri rGPR:$Rd, rGPR:$Rn, t2_so_imm_not:$imm,
                           pred:$p, cc_out:$s)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5392-5409
```tablegen
def : t2InstSubst<"orn${s}${p} $Rdn, $imm",
                  (t2ORRri rGPR:$Rdn, rGPR:$Rdn, t2_so_imm_not:$imm,
                           pred:$p, cc_out:$s)>;
def : t2InstSubst<"orr${s}${p} $Rd, $Rn, $imm",
                  (t2ORNri rGPR:$Rd, rGPR:$Rn, t2_so_imm_not:$imm,
                           pred:$p, cc_out:$s)>;
def : t2InstSubst<"orr${s}${p} $Rdn, $imm",
                  (t2ORNri rGPR:$Rdn, rGPR:$Rdn, t2_so_imm_not:$imm,
                           pred:$p, cc_out:$s)>;
// Likewise, "add Rd, t2_so_imm_neg" -> sub
def : t2InstSubst<"add${s}${p} $Rd, $Rn, $imm",
                  (t2SUBri rGPR:$Rd, GPRnopc:$Rn, t2_so_imm_neg:$imm,
                           pred:$p, cc_out:$s)>;
def : t2InstSubst<"add${s}${p} $Rd, $Rn, $imm",
                  (t2SUBspImm GPRsp:$Rd, GPRsp:$Rn, t2_so_imm_neg:$imm,
                           pred:$p, cc_out:$s)>;
def : t2InstSubst<"add${s}${p} $Rd, $imm",
                  (t2SUBri rGPR:$Rd, rGPR:$Rd, t2_so_imm_neg:$imm,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5410-5418
```tablegen
                           pred:$p, cc_out:$s)>;
def : t2InstSubst<"add${s}${p} $Rd, $imm",
                  (t2SUBspImm GPRsp:$Rd, GPRsp:$Rd, t2_so_imm_neg:$imm,
                           pred:$p, cc_out:$s)>;
// Same for CMP <--> CMN via t2_so_imm_neg
def : t2InstSubst<"cmp${p} $Rd, $imm",
                  (t2CMNri rGPR:$Rd, t2_so_imm_neg:$imm, pred:$p)>;
def : t2InstSubst<"cmn${p} $Rd, $imm",
                  (t2CMPri rGPR:$Rd, t2_so_imm_neg:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5421-5423
```tablegen
// Wide 'mul' encoding can be specified with only two operands.
def : t2InstAlias<"mul${p} $Rn, $Rm",
                  (t2MUL rGPR:$Rn, rGPR:$Rm, rGPR:$Rn, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5425-5427
```tablegen
// "neg" is and alias for "rsb rd, rn, #0"
def : t2InstAlias<"neg${s}${p} $Rd, $Rm",
                  (t2RSBri rGPR:$Rd, rGPR:$Rm, 0, pred:$p, cc_out:$s)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5429-5437
```tablegen
// MOV so_reg assembler pseudos. InstAlias isn't expressive enough for
// these, unfortunately.
// FIXME: LSL #0 in the shift should allow SP to be used as either the
// source or destination (but not both).
let hasSideEffects = 0 in {
def t2MOVsi: t2AsmPseudo<"mov${p} $Rd, $shift",
                         (ins rGPR:$Rd, t2_so_reg:$shift, pred:$p)>;
def t2MOVSsi: t2AsmPseudo<"movs${p} $Rd, $shift",
                          (ins rGPR:$Rd, t2_so_reg:$shift, pred:$p)>;
```
- EN: Defines TableGen record `t2MOVsi:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MOVsi:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5439-5443
```tablegen
def t2MOVsr: t2AsmPseudo<"mov${p} $Rd, $shift",
                         (ins rGPR:$Rd, so_reg_reg:$shift, pred:$p)>;
def t2MOVSsr: t2AsmPseudo<"movs${p} $Rd, $shift",
                          (ins rGPR:$Rd, so_reg_reg:$shift, pred:$p)>;
}
```
- EN: Defines TableGen record `t2MOVsr:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2MOVsr:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5445-5453
```tablegen
// Aliases for the above with the .w qualifier
def : t2InstAlias<"mov${p}.w $Rd, $shift",
                  (t2MOVsi rGPR:$Rd, t2_so_reg:$shift, pred:$p)>;
def : t2InstAlias<"movs${p}.w $Rd, $shift",
                  (t2MOVSsi rGPR:$Rd, t2_so_reg:$shift, pred:$p)>;
def : t2InstAlias<"mov${p}.w $Rd, $shift",
                  (t2MOVsr rGPR:$Rd, so_reg_reg:$shift, pred:$p)>;
def : t2InstAlias<"movs${p}.w $Rd, $shift",
                  (t2MOVSsr rGPR:$Rd, so_reg_reg:$shift, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5455-5457
```tablegen
// ADR w/o the .w suffix
def : t2InstAlias<"adr${p} $Rd, $addr",
                  (t2ADR rGPR:$Rd, t2adrlabel:$addr, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5459-5476
```tablegen
// LDR(literal) w/ alternate [pc, #imm] syntax.
def t2LDRpcrel   : t2AsmPseudo<"ldr${p} $Rt, $addr",
                         (ins GPR:$Rt, t2ldr_pcrel_imm12:$addr, pred:$p)>;
def t2LDRBpcrel  : t2AsmPseudo<"ldrb${p} $Rt, $addr",
                         (ins GPRnopc:$Rt, t2ldr_pcrel_imm12:$addr, pred:$p)>;
def t2LDRHpcrel  : t2AsmPseudo<"ldrh${p} $Rt, $addr",
                         (ins GPRnopc:$Rt, t2ldr_pcrel_imm12:$addr, pred:$p)>;
def t2LDRSBpcrel  : t2AsmPseudo<"ldrsb${p} $Rt, $addr",
                         (ins GPRnopc:$Rt, t2ldr_pcrel_imm12:$addr, pred:$p)>;
def t2LDRSHpcrel  : t2AsmPseudo<"ldrsh${p} $Rt, $addr",
                         (ins GPRnopc:$Rt, t2ldr_pcrel_imm12:$addr, pred:$p)>;
    // Version w/ the .w suffix.
def : t2InstAlias<"ldr${p}.w $Rt, $addr",
                  (t2LDRpcrel GPR:$Rt, t2ldr_pcrel_imm12:$addr, pred:$p), 0>;
def : t2InstAlias<"ldrb${p}.w $Rt, $addr",
                  (t2LDRBpcrel GPRnopc:$Rt, t2ldr_pcrel_imm12:$addr, pred:$p)>;
def : t2InstAlias<"ldrh${p}.w $Rt, $addr",
                  (t2LDRHpcrel GPRnopc:$Rt, t2ldr_pcrel_imm12:$addr, pred:$p)>;
```
- EN: Defines TableGen record `t2LDRpcrel` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRpcrel`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5477-5480
```tablegen
def : t2InstAlias<"ldrsb${p}.w $Rt, $addr",
                  (t2LDRSBpcrel GPRnopc:$Rt, t2ldr_pcrel_imm12:$addr, pred:$p)>;
def : t2InstAlias<"ldrsh${p}.w $Rt, $addr",
                  (t2LDRSHpcrel GPRnopc:$Rt, t2ldr_pcrel_imm12:$addr, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5482-5483
```tablegen
def : t2InstAlias<"add${p} $Rd, pc, $imm",
                  (t2ADR rGPR:$Rd, imm0_4095:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5485-5492
```tablegen
// Pseudo instruction ldr Rt, =immediate
def t2LDRConstPool
  : t2AsmPseudo<"ldr${p} $Rt, $immediate",
                (ins GPR:$Rt, const_pool_asm_imm:$immediate, pred:$p)>;
// Version w/ the .w suffix.
def : t2InstAlias<"ldr${p}.w $Rt, $immediate",
                  (t2LDRConstPool GPRnopc:$Rt,
                  const_pool_asm_imm:$immediate, pred:$p)>;
```
- EN: Defines TableGen record `t2LDRConstPool` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LDRConstPool`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5494-5496
```tablegen
//===----------------------------------------------------------------------===//
// ARMv8.1m instructions
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 5498-5502
```tablegen
class V8_1MI<dag oops, dag iops, AddrMode am, InstrItinClass itin, string asm,
             string ops, string cstr, list<dag> pattern>
  : Thumb2XI<oops, iops, am, 4, itin, !strconcat(asm, "\t", ops), cstr,
             pattern>,
    Requires<[HasV8_1MMainline]>;
```
- EN: Declares reusable TableGen class `V8_1MI` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `V8_1MI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5504-5508
```tablegen
def t2CLRM : V8_1MI<(outs),
                    (ins pred:$p, reglist_with_apsr:$regs, variable_ops),
                    AddrModeNone, NoItinerary, "clrm${p}", "$regs", "", []> {
  bits<0> p;
  bits<16> regs;
```
- EN: Defines TableGen record `t2CLRM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2CLRM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5510-5514
```tablegen
  let Inst{31-16} = 0b1110100010011111;
  let Inst{15-14} = regs{15-14};
  let Inst{13} = 0b0;
  let Inst{12-0} = regs{12-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5516-5517
```tablegen
class t2BF<dag iops, string asm, string ops>
  : V8_1MI<(outs ), iops, AddrModeNone, NoItinerary, asm, ops, "", []> {
```
- EN: Declares reusable TableGen class `t2BF` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `t2BF`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5519-5522
```tablegen
  let Inst{31-27} = 0b11110;
  let Inst{15-14} = 0b11;
  let Inst{12} = 0b0;
  let Inst{0} = 0b1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5524-5525
```tablegen
  let Predicates = [IsThumb2, HasV8_1MMainline, HasLOB];
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5527-5532
```tablegen
def t2BF_LabelPseudo
  : t2PseudoInst<(outs ), (ins pclabel:$cp), 0, NoItinerary, []> {
  let isTerminator = 1;
  let Predicates = [IsThumb2, HasV8_1MMainline, HasLOB];
  let hasNoSchedulingInfo = 1;
}
```
- EN: Defines TableGen record `t2BF_LabelPseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2BF_LabelPseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5534-5538
```tablegen
def t2BFi : t2BF<(ins bflabel_u4:$b_label, bflabel_s16:$label, pred:$p),
                 !strconcat("bf", "${p}"), "$b_label, $label"> {
  bits<0> p;
  bits<4> b_label;
  bits<16> label;
```
- EN: Defines TableGen record `t2BFi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2BFi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5540-5546
```tablegen
  let Inst{26-23} = b_label{3-0};
  let Inst{22-21} = 0b10;
  let Inst{20-16} = label{15-11};
  let Inst{13} = 0b1;
  let Inst{11} = label{0};
  let Inst{10-1} = label{10-1};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5548-5554
```tablegen
def t2BFic : t2BF<(ins bflabel_u4:$b_label, bflabel_s12:$label,
                   bfafter_target:$ba_label, pred_noal:$bcond), "bfcsel",
                  "$b_label, $label, $ba_label, $bcond"> {
  bits<4> bcond;
  bits<12> label;
  bits<1> ba_label;
  bits<4> b_label;
```
- EN: Defines TableGen record `t2BFic` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2BFic`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5556-5564
```tablegen
  let Inst{26-23} = b_label{3-0};
  let Inst{22} = 0b0;
  let Inst{21-18} = bcond{3-0};
  let Inst{17} = ba_label{0};
  let Inst{16} = label{11};
  let Inst{13} = 0b1;
  let Inst{11} = label{0};
  let Inst{10-1} = label{10-1};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5566-5570
```tablegen
def t2BFr : t2BF<(ins bflabel_u4:$b_label, rGPR:$Rn, pred:$p),
                 !strconcat("bfx", "${p}"), "$b_label, $Rn"> {
  bits<0> p;
  bits<4> b_label;
  bits<4> Rn;
```
- EN: Defines TableGen record `t2BFr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2BFr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5572-5576
```tablegen
  let Inst{26-23} = b_label{3-0};
  let Inst{22-20} = 0b110;
  let Inst{19-16} = Rn{3-0};
  let Inst{13-1} = 0b1000000000000;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5578-5582
```tablegen
def t2BFLi : t2BF<(ins bflabel_u4:$b_label, bflabel_s18:$label, pred:$p),
                  !strconcat("bfl", "${p}"), "$b_label, $label"> {
  bits<0> p;
  bits<4> b_label;
  bits<18> label;
```
- EN: Defines TableGen record `t2BFLi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2BFLi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5584-5589
```tablegen
  let Inst{26-23} = b_label{3-0};
  let Inst{22-16} = label{17-11};
  let Inst{13} = 0b0;
  let Inst{11} = label{0};
  let Inst{10-1} = label{10-1};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5591-5595
```tablegen
def t2BFLr : t2BF<(ins bflabel_u4:$b_label, rGPR:$Rn, pred:$p),
                  !strconcat("bflx", "${p}"), "$b_label, $Rn"> {
  bits<0> p;
  bits<4> b_label;
  bits<4> Rn;
```
- EN: Defines TableGen record `t2BFLr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2BFLr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5597-5601
```tablegen
  let Inst{26-23} = b_label{3-0};
  let Inst{22-20} = 0b111;
  let Inst{19-16} = Rn{3-0};
  let Inst{13-1} = 0b1000000000000;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5603-5610
```tablegen
class t2LOL<dag oops, dag iops, string asm, string ops>
  : V8_1MI<oops, iops, AddrModeNone, NoItinerary, asm, ops, "", [] > {
  let Inst{31-23} = 0b111100000;
  let Inst{15-14} = 0b11;
  let Inst{0} = 0b1;
  let DecoderMethod = "DecodeLOLoop";
  let Predicates = [IsThumb2, HasV8_1MMainline, HasLOB];
}
```
- EN: Declares reusable TableGen class `t2LOL` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `t2LOL`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5612-5616
```tablegen
// Setup for the iteration count of a WLS. See t2WhileLoopSetup.
def arm_wlssetup
    : SDNode<"ARMISD::WLSSETUP",
             SDTypeProfile<1, 1, [SDTCisInt<0>, SDTCisSameAs<1, 0>]>,
             [SDNPSideEffect]>;
```
- EN: Defines TableGen record `arm_wlssetup` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `arm_wlssetup`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 5618-5621
```tablegen
// Low-overhead loops, While Loop Start branch. See t2WhileLoopStart
def arm_wls : SDNode<"ARMISD::WLS",
                     SDTypeProfile<0, 2, [SDTCisInt<0>, SDTCisVT<1, OtherVT>]>,
                     [SDNPHasChain]>;
```
- EN: Defines TableGen record `arm_wls` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `arm_wls`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 5623-5624
```tablegen
// Really a part of LE, performs the sub
def arm_loop_dec : SDNode<"ARMISD::LOOP_DEC", SDTIntBinOp, [SDNPHasChain]>;
```
- EN: Defines TableGen record `arm_loop_dec` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `arm_loop_dec`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 5626-5629
```tablegen
// Low-overhead loops, Loop End
def arm_le : SDNode<"ARMISD::LE",
                    SDTypeProfile<0, 2, [SDTCisInt<0>, SDTCisVT<1, OtherVT>]>,
                    [SDNPHasChain]>;
```
- EN: Defines TableGen record `arm_le` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `arm_le`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 5631-5645
```tablegen
let isNotDuplicable = 1 in {
def t2WLS : t2LOL<(outs GPRlr:$LR),
                  (ins rGPR:$Rn, wlslabel_u11:$label),
                  "wls", "$LR, $Rn, $label"> {
  bits<4> Rn;
  bits<11> label;
  let Inst{22-20} = 0b100;
  let Inst{19-16} = Rn{3-0};
  let Inst{13-12} = 0b00;
  let Inst{11} = label{0};
  let Inst{10-1} = label{10-1};
  let usesCustomInserter = 1;
  let isBranch = 1;
  let isTerminator = 1;
}
```
- EN: Defines TableGen record `t2WLS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2WLS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5647-5654
```tablegen
def t2DLS : t2LOL<(outs GPRlr:$LR), (ins rGPR:$Rn),
                  "dls", "$LR, $Rn"> {
  bits<4> Rn;
  let Inst{22-20} = 0b100;
  let Inst{19-16} = Rn{3-0};
  let Inst{13-1} = 0b1000000000000;
  let usesCustomInserter = 1;
}
```
- EN: Defines TableGen record `t2DLS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2DLS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5656-5667
```tablegen
def t2LEUpdate : t2LOL<(outs GPRlr:$LRout),
                       (ins GPRlr:$LRin, lelabel_u11:$label),
                       "le", "$LRin, $label"> {
  bits<11> label;
  let Inst{22-16} = 0b0001111;
  let Inst{13-12} = 0b00;
  let Inst{11} = label{0};
  let Inst{10-1} = label{10-1};
  let usesCustomInserter = 1;
  let isBranch = 1;
  let isTerminator = 1;
}
```
- EN: Defines TableGen record `t2LEUpdate` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LEUpdate`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5669-5677
```tablegen
def t2LE : t2LOL<(outs ), (ins lelabel_u11:$label), "le", "$label"> {
  bits<11> label;
  let Inst{22-16} = 0b0101111;
  let Inst{13-12} = 0b00;
  let Inst{11} = label{0};
  let Inst{10-1} = label{10-1};
  let isBranch = 1;
  let isTerminator = 1;
}
```
- EN: Defines TableGen record `t2LE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5679-5679
```tablegen
let Predicates = [IsThumb2, HasV8_1MMainline, HasLOB] in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5681-5685
```tablegen
// t2DoLoopStart a pseudo for DLS hardware loops. Lowered into a DLS in
// ARMLowOverheadLoops if possible, or reverted to a Mov if not.
def t2DoLoopStart :
  t2PseudoInst<(outs GPRlr:$X), (ins rGPR:$tc), 4, IIC_Br,
  [(set GPRlr:$X, (int_start_loop_iterations rGPR:$tc))]>;
```
- EN: Defines TableGen record `t2DoLoopStart` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2DoLoopStart`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5687-5693
```tablegen
// A pseudo for a DLSTP, created in the MVETPAndVPTOptimizationPass from a
// t2DoLoopStart if the loops is tail predicated. Holds both the element
// count and trip count of the loop, picking the correct one during
// ARMLowOverheadLoops when it is converted to a DLSTP or DLS as required.
let isTerminator = 1, hasSideEffects = 1 in
def t2DoLoopStartTP :
  t2PseudoInst<(outs GPRlr:$X), (ins rGPR:$tc, rGPR:$elts), 4, IIC_Br, []>;
```
- EN: Defines TableGen record `t2DoLoopStartTP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2DoLoopStartTP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5695-5703
```tablegen
// Setup for a t2WhileLoopStart. A pair of t2WhileLoopSetup and t2WhileLoopStart
// will be created post-ISel from a llvm.test.start.loop.iterations. This
// t2WhileLoopSetup to setup LR and t2WhileLoopStart to perform the branch. Not
// valid after reg alloc, as it should be lowered during MVETPAndVPTOptimisations
// into a t2WhileLoopStartLR (or expanded).
let hasSideEffects = 1 in
def t2WhileLoopSetup :
    t2PseudoInst<(outs GPRlr:$lr), (ins rGPR:$tc), 4, IIC_Br,
                 [(set i32:$lr, (arm_wlssetup i32:$tc))]>;
```
- EN: Defines TableGen record `t2WhileLoopSetup` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2WhileLoopSetup`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5705-5712
```tablegen
// A pseudo to represent the decrement in a low overhead loop. A t2LoopDec and
// t2LoopEnd together represent a LE instruction. Ideally these are converted
// to a t2LoopEndDec which is lowered as a single instruction.
let hasSideEffects = 0 in
def t2LoopDec :
    t2PseudoInst<(outs GPRlr:$Rm), (ins GPRlr:$Rn, imm0_7:$size), 4, IIC_Br,
                 [(set i32:$Rm, (arm_loop_dec i32:$Rn, timm:$size))]>,
    Sched<[WriteBr]>;
```
- EN: Defines TableGen record `t2LoopDec` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LoopDec`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5714-5721
```tablegen
let isBranch = 1, isTerminator = 1, hasSideEffects = 1, Defs = [CPSR] in {
// The branch in a t2WhileLoopSetup/t2WhileLoopStart pair, eventually turned
// into a t2WhileLoopStartLR that does both the LR setup and branch.
def t2WhileLoopStart :
    t2PseudoInst<(outs),
                 (ins GPRlr:$tc, brtarget:$target),
                 4, IIC_Br, [(arm_wls i32:$tc, bb:$target)]>,
    Sched<[WriteBr]>;
```
- EN: Defines TableGen record `t2WhileLoopStart` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2WhileLoopStart`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5723-5731
```tablegen
// WhileLoopStartLR that sets up LR and branches on zero, equivalent to WLS. It
// is lowered in the ARMLowOverheadLoops pass providing the branches are within
// range. WhileLoopStartLR and LoopEnd to occupy 8 bytes because they may get
// converted into t2CMP and t2Bcc.
def t2WhileLoopStartLR :
    t2PseudoInst<(outs GPRlr:$lr),
                 (ins rGPR:$tc, brtarget:$target),
                 8, IIC_Br, []>,
                 Sched<[WriteBr]>;
```
- EN: Defines TableGen record `t2WhileLoopStartLR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2WhileLoopStartLR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5733-5739
```tablegen
// Similar to a t2DoLoopStartTP, a t2WhileLoopStartTP is a pseudo for a WLSTP
// holding both the element count and the tripcount of the loop.
def t2WhileLoopStartTP :
    t2PseudoInst<(outs GPRlr:$lr),
                 (ins rGPR:$tc, rGPR:$elts, brtarget:$target),
                 8, IIC_Br, []>,
                 Sched<[WriteBr]>;
```
- EN: Defines TableGen record `t2WhileLoopStartTP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2WhileLoopStartTP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5741-5745
```tablegen
// t2LoopEnd - the branch half of a t2LoopDec/t2LoopEnd pair.
def t2LoopEnd :
    t2PseudoInst<(outs), (ins GPRlr:$tc, brtarget:$target),
                 8, IIC_Br, [(arm_le i32:$tc, bb:$target)]>,
    Sched<[WriteBr]>;
```
- EN: Defines TableGen record `t2LoopEnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LoopEnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5747-5753
```tablegen
// The combination of a t2LoopDec and t2LoopEnd, performing both the LR
// decrement and branch as a single instruction. Is lowered to a LE or
// LETP in ARMLowOverheadLoops as appropriate, or converted to t2CMP/t2Bcc
// if the branches are out of range.
def t2LoopEndDec :
  t2PseudoInst<(outs GPRlr:$Rm), (ins GPRlr:$tc, brtarget:$target),
  8, IIC_Br, []>, Sched<[WriteBr]>;
```
- EN: Defines TableGen record `t2LoopEndDec` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2LoopEndDec`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5755-5755
```tablegen
} // end isBranch, isTerminator, hasSideEffects
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5757-5757
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5759-5759
```tablegen
} // end isNotDuplicable
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5761-5767
```tablegen
class CS<string iname, bits<4> opcode, list<dag> pattern=[]>
  : V8_1MI<(outs rGPR:$Rd), (ins GPRwithZRnosp:$Rn, GPRwithZRnosp:$Rm, pred_noal:$fcond),
           AddrModeNone, NoItinerary, iname, "$Rd, $Rn, $Rm, $fcond", "", pattern> {
  bits<4> Rd;
  bits<4> Rm;
  bits<4> Rn;
  bits<4> fcond;
```
- EN: Declares reusable TableGen class `CS` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `CS`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5769-5774
```tablegen
  let Inst{31-20} = 0b111010100101;
  let Inst{19-16} = Rn{3-0};
  let Inst{15-12} = opcode;
  let Inst{11-8} = Rd{3-0};
  let Inst{7-4} = fcond{3-0};
  let Inst{3-0} = Rm{3-0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5776-5778
```tablegen
  let Uses = [CPSR];
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5780-5783
```tablegen
def t2CSEL  : CS<"csel",  0b1000>;
def t2CSINC : CS<"csinc", 0b1001>;
def t2CSINV : CS<"csinv", 0b1010>;
def t2CSNEG : CS<"csneg", 0b1011>;
```
- EN: Defines TableGen record `t2CSEL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2CSEL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5785-5788
```tablegen
let HasOneUse = 1 in
def ARMcsinc_su
    : PatFrag<(ops node:$lhs, node:$rhs, node:$cc, node:$flags),
              (ARMcsinc node:$lhs, node:$rhs, node:$cc, node:$flags)>;
```
- EN: Defines TableGen record `ARMcsinc_su` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMcsinc_su`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5790-5800
```tablegen
let Predicates = [HasV8_1MMainline] in {
  multiclass CSPats<SDNode Node, Instruction Insn> {
    def : T2Pat<(Node GPRwithZR:$tval, GPRwithZR:$fval, imm:$cc, CPSR),
                (Insn GPRwithZR:$tval, GPRwithZR:$fval, imm:$cc)>;
    def : T2Pat<(Node (i32 0), GPRwithZR:$fval, imm:$cc, CPSR),
                (Insn ZR, GPRwithZR:$fval, imm:$cc)>;
    def : T2Pat<(Node GPRwithZR:$tval, (i32 0), imm:$cc, CPSR),
                (Insn GPRwithZR:$tval, ZR, imm:$cc)>;
    def : T2Pat<(Node (i32 0), (i32 0), imm:$cc, CPSR),
                (Insn ZR, ZR, imm:$cc)>;
  }
```
- EN: Declares TableGen `multiclass CSPats`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass CSPats`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5802-5804
```tablegen
  defm : CSPats<ARMcsinc, t2CSINC>;
  defm : CSPats<ARMcsinv, t2CSINV>;
  defm : CSPats<ARMcsneg, t2CSNEG>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5806-5813
```tablegen
  def : T2Pat<(ARMcmov (i32 1), (i32 0), imm:$cc, CPSR),
              (t2CSINC ZR, ZR, imm:$cc)>;
  def : T2Pat<(ARMcmov (i32 -1), (i32 0), imm:$cc, CPSR),
              (t2CSINV ZR, ZR, imm:$cc)>;
  def : T2Pat<(ARMcmov (i32 0), (i32 1), imm:$cc, CPSR),
              (t2CSINC ZR, ZR, (inv_cond_XFORM imm:$cc))>;
  def : T2Pat<(ARMcmov (i32 0), (i32 -1), imm:$cc, CPSR),
              (t2CSINV ZR, ZR, (inv_cond_XFORM imm:$cc))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5815-5824
```tablegen
  multiclass ModifiedV8_1CSEL<Instruction Insn, dag modvalue> {
    def : T2Pat<(ARMcmov modvalue, GPRwithZR:$tval, imm:$cc, CPSR),
                (Insn GPRwithZR:$tval, GPRwithZR:$fval, imm:$cc)>;
    def : T2Pat<(ARMcmov GPRwithZR:$tval, modvalue, imm:$cc, CPSR),
                (Insn GPRwithZR:$tval, GPRwithZR:$fval,
                         (i32 (inv_cond_XFORM imm:$cc)))>;
  }
  defm : ModifiedV8_1CSEL<t2CSINC, (add rGPR:$fval, 1)>;
  defm : ModifiedV8_1CSEL<t2CSINV, (xor rGPR:$fval, -1)>;
  defm : ModifiedV8_1CSEL<t2CSNEG, (sub 0, rGPR:$fval)>;
```
- EN: Declares TableGen `multiclass ModifiedV8_1CSEL`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass ModifiedV8_1CSEL`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5826-5831
```tablegen
  def : T2Pat<(ARMcmov (topbitsallzero32:$Rn), (i32 1), imm:$cc, CPSR),
              (t2CSINC $Rn, ZR, (inv_cond_XFORM imm:$cc))>;
  def : T2Pat<(and (topbitsallzero32:$Rn),
                   (ARMcsinc_su (i32 0), (i32 0), imm:$cc, CPSR)),
              (t2CSEL ZR, $Rn, imm:$cc)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5833-5836
```tablegen
// CS aliases.
let Predicates = [HasV8_1MMainline] in {
  def : InstAlias<"csetm\t$Rd, $fcond",
                 (t2CSINV rGPR:$Rd, ZR, ZR, pred_noal_inv:$fcond)>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5838-5839
```tablegen
  def : InstAlias<"cset\t$Rd, $fcond",
                 (t2CSINC rGPR:$Rd, ZR, ZR, pred_noal_inv:$fcond)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5841-5842
```tablegen
  def : InstAlias<"cinc\t$Rd, $Rn, $fcond",
                 (t2CSINC rGPR:$Rd, GPRwithZRnosp:$Rn, GPRwithZRnosp:$Rn, pred_noal_inv:$fcond)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5844-5845
```tablegen
  def : InstAlias<"cinv\t$Rd, $Rn, $fcond",
                 (t2CSINV rGPR:$Rd, GPRwithZRnosp:$Rn, GPRwithZRnosp:$Rn, pred_noal_inv:$fcond)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5847-5849
```tablegen
  def : InstAlias<"cneg\t$Rd, $Rn, $fcond",
                 (t2CSNEG rGPR:$Rd, GPRwithZRnosp:$Rn, GPRwithZRnosp:$Rn, pred_noal_inv:$fcond)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5852-5867
```tablegen
// PACBTI
let Predicates = [IsThumb2, HasV8_1MMainline, HasPACBTI] in {
def t2PACG : V8_1MI<(outs rGPR:$Rd),
                    (ins pred:$p, GPRnopc:$Rn, GPRnopc:$Rm),
                    AddrModeNone, NoItinerary, "pacg${p}", "$Rd, $Rn, $Rm", "", []> {
  bits<0> p;
  bits<4> Rd;
  bits<4> Rn;
  bits<4> Rm;
  let Inst{31-20} = 0b111110110110;
  let Inst{19-16} = Rn;
  let Inst{15-12} = 0b1111;
  let Inst{11-8}  = Rd;
  let Inst{7-4}   = 0b0000;
  let Inst{3-0}   = Rm;
}
```
- EN: Defines TableGen record `t2PACG` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2PACG`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5869-5884
```tablegen
let hasSideEffects = 1 in {
class PACBTIAut<dag iops, string asm, bit b>
  : V8_1MI<(outs), iops,
           AddrModeNone, NoItinerary, asm, "$Ra, $Rn, $Rm", "", []> {
  bits<0> p;
  bits<4> Ra;
  bits<4> Rn;
  bits<4> Rm;
  let Inst{31-20} = 0b111110110101;
  let Inst{19-16} = Rn;
  let Inst{15-12} = Ra;
  let Inst{11-5}  = 0b1111000;
  let Inst{4}     = b;
  let Inst{3-0}   = Rm;
}
}
```
- EN: Declares reusable TableGen class `PACBTIAut` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `PACBTIAut`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5886-5887
```tablegen
def t2AUTG  : PACBTIAut<(ins pred:$p, GPRnosp:$Ra, GPRnopc:$Rn, GPRnopc:$Rm),
                        "autg${p}", 0>;
```
- EN: Defines TableGen record `t2AUTG` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2AUTG`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5889-5893
```tablegen
let isBranch = 1, isTerminator = 1, isIndirectBranch = 1 in {
  def t2BXAUT : PACBTIAut<(ins pred:$p, GPRnosp:$Ra, rGPR:$Rn, GPRnopc:$Rm),
                          "bxaut${p}", 1>;
}
}
```
- EN: Defines TableGen record `t2BXAUT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2BXAUT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5895-5898
```tablegen
let isReturn = 1, isTerminator = 1, isBarrier = 1 in {
  def t2BXAUT_RET : tPseudoExpand<(outs), (ins pred:$p), 4, IIC_Br,
                   [(ARMretglue)], (t2BXAUT pred:$p, R12, LR, SP)>, Sched<[WriteBr]>;
}
```
- EN: Defines TableGen record `t2BXAUT_RET` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2BXAUT_RET`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5900-5904
```tablegen
class PACBTIHintSpaceInst<string asm, string ops, bits<8> imm>
  : Thumb2XI<(outs), (ins), AddrModeNone, 4, NoItinerary, !strconcat(asm, "\t", ops), "", []>,
    Requires<[HasV7, IsMClass]> {
  let Inst{31-8} = 0b111100111010111110000000;
  let Inst{7-0}  = imm;
```
- EN: Declares reusable TableGen class `PACBTIHintSpaceInst` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `PACBTIHintSpaceInst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5906-5907
```tablegen
  let Unpredictable{19-16} = 0b1111;
  let Unpredictable{13-11} = 0b101;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5909-5910
```tablegen
  let DecoderMethod = "DecodeT2HintSpaceInstruction";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5912-5913
```tablegen
class PACBTIHintSpaceNoOpsInst<string asm, bits<8> imm>
  : PACBTIHintSpaceInst<asm, "", imm>;
```
- EN: Declares reusable TableGen class `PACBTIHintSpaceNoOpsInst` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `PACBTIHintSpaceNoOpsInst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5915-5919
```tablegen
class PACBTIHintSpaceDefInst<string asm, bits<8> imm>
  : PACBTIHintSpaceInst<asm, "r12, lr, sp", imm> {
  let Defs = [R12];
  let Uses = [LR, SP];
}
```
- EN: Declares reusable TableGen class `PACBTIHintSpaceDefInst` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `PACBTIHintSpaceDefInst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5921-5924
```tablegen
class PACBTIHintSpaceUseInst<string asm, bits<8> imm>
  : PACBTIHintSpaceInst<asm, "r12, lr, sp", imm> {
  let Uses = [R12, LR, SP];
}
```
- EN: Declares reusable TableGen class `PACBTIHintSpaceUseInst` for `ARMInstrThumb2`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb2` 声明可复用的 TableGen 类 `PACBTIHintSpaceUseInst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5926-5931
```tablegen
def t2PAC    : PACBTIHintSpaceDefInst<"pac", 0b00011101>;
def t2PACBTI : PACBTIHintSpaceDefInst<"pacbti", 0b00001101>;
def t2BTI    : PACBTIHintSpaceNoOpsInst<"bti", 0b00001111>;
def t2AUT    : PACBTIHintSpaceUseInst<"aut", 0b00101101> {
  let hasSideEffects = 1;
}
```
- EN: Defines TableGen record `t2PAC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2PAC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5933-5935
```tablegen
// Thumb function call followed by BTI instruction.
def ARMt2CallBTI : SDNode<"ARMISD::t2CALL_BTI", SDT_ARMcall,
                   [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue, SDNPVariadic]>;
```
- EN: Defines TableGen record `ARMt2CallBTI` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMt2CallBTI`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 5937-5940
```tablegen
let Defs = [LR], Uses = [SP] in
def t2CALL_BTI : PseudoInst<(outs), (ins pred:$p, thumb_bl_target:$func),
                 IIC_Br, [(ARMt2CallBTI tglobaladdr:$func)]>,
                 Requires<[IsThumb2]>, Sched<[WriteBrL]>;
```
- EN: Defines TableGen record `t2CALL_BTI` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t2CALL_BTI`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

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
