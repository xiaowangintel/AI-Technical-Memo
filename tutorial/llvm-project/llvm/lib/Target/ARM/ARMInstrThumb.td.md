# ARMInstrThumb.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMInstrThumb.td`
- Repository: `llvm-project`
- Purpose (EN): This file describes the Thumb instruction set.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMInstrThumb`，涵盖指令定义与目标操作码元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===-- ARMInstrThumb.td - Thumb support for ARM -----------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the Thumb instruction set.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-15
```tablegen
//===----------------------------------------------------------------------===//
// Thumb specific DAG Nodes.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 17-20
```tablegen
// CMSE non-secure function call.
def ARMtsecall : SDNode<"ARMISD::tSECALL", SDT_ARMcall,
                        [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
                         SDNPVariadic]>;
```
- EN: Defines TableGen record `ARMtsecall` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMtsecall`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 22-32
```tablegen
def imm_sr_XFORM: SDNodeXForm<imm, [{
  unsigned Imm = N->getZExtValue();
  return CurDAG->getTargetConstant((Imm == 32 ? 0 : Imm), SDLoc(N), MVT::i32);
}]>;
def ThumbSRImmAsmOperand: ImmAsmOperand<1,32> { let Name = "ImmThumbSR"; }
def imm_sr : Operand<i32>, ImmLeaf<i32, [{
  return Imm > 0 && Imm <= 32;
}], imm_sr_XFORM> {
  let PrintMethod = "printThumbSRImm";
  let ParserMatchClass = ThumbSRImmAsmOperand;
}
```
- EN: Defines TableGen record `imm_sr_XFORM:` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `imm_sr_XFORM:`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 34-36
```tablegen
def imm0_7_neg : PatLeaf<(i32 imm), [{
  return (uint32_t)-N->getZExtValue() < 8;
}], imm_neg_XFORM>;
```
- EN: Defines TableGen record `imm0_7_neg` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `imm0_7_neg`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 38-44
```tablegen
def ThumbModImmNeg1_7AsmOperand : AsmOperandClass { let Name = "ThumbModImmNeg1_7"; }
def mod_imm1_7_neg : Operand<i32>, PatLeaf<(imm), [{
    unsigned Value = -(unsigned)N->getZExtValue();
    return 0 < Value && Value < 8;
  }], imm_neg_XFORM> {
  let ParserMatchClass = ThumbModImmNeg1_7AsmOperand;
}
```
- EN: Defines TableGen record `ThumbModImmNeg1_7AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ThumbModImmNeg1_7AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 46-52
```tablegen
def ThumbModImmNeg8_255AsmOperand : AsmOperandClass { let Name = "ThumbModImmNeg8_255"; }
def mod_imm8_255_neg : Operand<i32>, PatLeaf<(imm), [{
    unsigned Value = -(unsigned)N->getZExtValue();
    return 7 < Value && Value < 256;
  }], imm_neg_XFORM> {
  let ParserMatchClass = ThumbModImmNeg8_255AsmOperand;
}
```
- EN: Defines TableGen record `ThumbModImmNeg8_255AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ThumbModImmNeg8_255AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 55-57
```tablegen
def imm0_255_comp : PatLeaf<(i32 imm), [{
  return ~((uint32_t)N->getZExtValue()) < 256;
}]>;
```
- EN: Defines TableGen record `imm0_255_comp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `imm0_255_comp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 59-62
```tablegen
def imm8_255_neg : PatLeaf<(i32 imm), [{
  unsigned Val = -N->getZExtValue();
  return Val >= 8 && Val < 256;
}], imm_neg_XFORM>;
```
- EN: Defines TableGen record `imm8_255_neg` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `imm8_255_neg`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 64-69
```tablegen
// Break imm's up into two pieces: an immediate + a left shift. This uses
// thumb_immshifted to match and thumb_immshifted_val and thumb_immshifted_shamt
// to get the val/shift pieces.
def thumb_immshifted : PatLeaf<(imm), [{
  return ARM_AM::isThumbImmShiftedVal((unsigned)N->getZExtValue());
}]>;
```
- EN: Defines TableGen record `thumb_immshifted` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `thumb_immshifted`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 71-74
```tablegen
def thumb_immshifted_val : SDNodeXForm<imm, [{
  unsigned V = ARM_AM::getThumbImmNonShiftedVal((unsigned)N->getZExtValue());
  return CurDAG->getTargetConstant(V, SDLoc(N), MVT::i32);
}]>;
```
- EN: Defines TableGen record `thumb_immshifted_val` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `thumb_immshifted_val`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 76-79
```tablegen
def thumb_immshifted_shamt : SDNodeXForm<imm, [{
  unsigned V = ARM_AM::getThumbImmValShift((unsigned)N->getZExtValue());
  return CurDAG->getTargetConstant(V, SDLoc(N), MVT::i32);
}]>;
```
- EN: Defines TableGen record `thumb_immshifted_shamt` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `thumb_immshifted_shamt`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 81-83
```tablegen
def imm256_510 : ImmLeaf<i32, [{
  return Imm >= 256 && Imm < 511;
}]>;
```
- EN: Defines TableGen record `imm256_510` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `imm256_510`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 85-87
```tablegen
def thumb_imm256_510_addend : SDNodeXForm<imm, [{
  return CurDAG->getTargetConstant(N->getZExtValue() - 255, SDLoc(N), MVT::i32);
}]>;
```
- EN: Defines TableGen record `thumb_imm256_510_addend` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `thumb_imm256_510_addend`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 89-95
```tablegen
// Scaled 4 immediate.
def t_imm0_1020s4_asmoperand: AsmOperandClass { let Name = "Imm0_1020s4"; }
def t_imm0_1020s4 : Operand<i32> {
  let PrintMethod = "printThumbS4ImmOperand";
  let ParserMatchClass = t_imm0_1020s4_asmoperand;
  let OperandType = "OPERAND_IMMEDIATE";
}
```
- EN: Defines TableGen record `t_imm0_1020s4_asmoperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t_imm0_1020s4_asmoperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 97-108
```tablegen
def t_imm0_508s4_asmoperand: AsmOperandClass { let Name = "Imm0_508s4"; }
def t_imm0_508s4 : Operand<i32> {
  let PrintMethod = "printThumbS4ImmOperand";
  let ParserMatchClass = t_imm0_508s4_asmoperand;
  let OperandType = "OPERAND_IMMEDIATE";
}
// Alias use only, so no printer is necessary.
def t_imm0_508s4_neg_asmoperand: AsmOperandClass { let Name = "Imm0_508s4Neg"; }
def t_imm0_508s4_neg : Operand<i32> {
  let ParserMatchClass = t_imm0_508s4_neg_asmoperand;
  let OperandType = "OPERAND_IMMEDIATE";
}
```
- EN: Defines TableGen record `t_imm0_508s4_asmoperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t_imm0_508s4_asmoperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 110-110
```tablegen
// Define Thumb specific addressing modes.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 112-116
```tablegen
// unsigned 8-bit, 2-scaled memory offset
class OperandUnsignedOffset_b8s2 : AsmOperandClass {
  let Name = "UnsignedOffset_b8s2";
  let PredicateMethod = "isUnsignedOffset<8, 2>";
}
```
- EN: Declares reusable TableGen class `OperandUnsignedOffset_b8s2` for `ARMInstrThumb`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb` 声明可复用的 TableGen 类 `OperandUnsignedOffset_b8s2`，通常用于抽象共享字段、谓词或编码结构。

### Lines 118-118
```tablegen
def UnsignedOffset_b8s2 : OperandUnsignedOffset_b8s2;
```
- EN: Defines TableGen record `UnsignedOffset_b8s2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UnsignedOffset_b8s2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 120-125
```tablegen
// thumb style PC relative operand. signed, 8 bits magnitude,
// two bits shift. can be represented as either [pc, #imm], #imm,
// or relocatable expression...
def ThumbMemPC : AsmOperandClass {
  let Name = "ThumbMemPC";
}
```
- EN: Defines TableGen record `ThumbMemPC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ThumbMemPC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 127-131
```tablegen
let OperandType = "OPERAND_PCREL" in {
def t_brtarget : Operand<OtherVT> {
  let EncoderMethod = "getThumbBRTargetOpValue";
  let DecoderMethod = "DecodeThumbBROperand";
}
```
- EN: Defines TableGen record `t_brtarget` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t_brtarget`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 133-138
```tablegen
// ADR instruction labels.
def t_adrlabel : Operand<i32> {
  let EncoderMethod = "getThumbAdrLabelOpValue";
  let PrintMethod = "printAdrLabelOperand<2>";
  let ParserMatchClass = UnsignedOffset_b8s2;
}
```
- EN: Defines TableGen record `t_adrlabel` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t_adrlabel`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 141-145
```tablegen
def thumb_br_target : Operand<OtherVT> {
  let ParserMatchClass = ThumbBranchTarget;
  let EncoderMethod = "getThumbBranchTargetOpValue";
  let OperandType = "OPERAND_PCREL";
}
```
- EN: Defines TableGen record `thumb_br_target` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `thumb_br_target`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 147-151
```tablegen
def thumb_bl_target : Operand<i32> {
  let ParserMatchClass = ThumbBranchTarget;
  let EncoderMethod = "getThumbBLTargetOpValue";
  let DecoderMethod = "DecodeThumbBLTargetOperand";
}
```
- EN: Defines TableGen record `thumb_bl_target` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `thumb_bl_target`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 153-158
```tablegen
// Target for BLX *from* thumb mode.
def thumb_blx_target : Operand<i32> {
  let ParserMatchClass = ARMBranchTarget;
  let EncoderMethod = "getThumbBLXTargetOpValue";
  let DecoderMethod = "DecodeThumbBLXOffset";
}
```
- EN: Defines TableGen record `thumb_blx_target` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `thumb_blx_target`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 160-164
```tablegen
def thumb_bcc_target : Operand<OtherVT> {
  let ParserMatchClass = ThumbBranchTarget;
  let EncoderMethod = "getThumbBCCTargetOpValue";
  let DecoderMethod = "DecodeThumbBCCTargetOperand";
}
```
- EN: Defines TableGen record `thumb_bcc_target` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `thumb_bcc_target`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 166-171
```tablegen
def thumb_cb_target : Operand<OtherVT> {
  let ParserMatchClass = ThumbBranchTarget;
  let EncoderMethod = "getThumbCBTargetOpValue";
  let DecoderMethod = "DecodeThumbCmpBROperand";
}
} // OperandType = "OPERAND_PCREL"
```
- EN: Defines TableGen record `thumb_cb_target` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `thumb_cb_target`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 173-180
```tablegen
// t_addrmode_pc := <label> => pc + imm8 * 4
//
def t_addrmode_pc : MemOperand {
  let EncoderMethod = "getAddrModePCOpValue";
  let DecoderMethod = "DecodeThumbAddrModePC";
  let PrintMethod = "printThumbLdrLabelOperand";
  let ParserMatchClass = ThumbMemPC;
}
```
- EN: Defines TableGen record `t_addrmode_pc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t_addrmode_pc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 182-192
```tablegen
// t_addrmode_rr := reg + reg
//
def t_addrmode_rr_asm_operand : AsmOperandClass { let Name = "MemThumbRR"; }
def t_addrmode_rr : MemOperand,
                    ComplexPattern<i32, 2, "SelectThumbAddrModeRR", []> {
  let EncoderMethod = "getThumbAddrModeRegRegOpValue";
  let PrintMethod = "printThumbAddrModeRROperand";
  let DecoderMethod = "DecodeThumbAddrModeRR";
  let ParserMatchClass = t_addrmode_rr_asm_operand;
  let MIOperandInfo = (ops tGPR:$base, tGPR:$offsreg);
}
```
- EN: Defines TableGen record `t_addrmode_rr_asm_operand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t_addrmode_rr_asm_operand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 194-205
```tablegen
// t_addrmode_rr_sext := reg + reg
//
// This is similar to t_addrmode_rr, but uses different heuristics for
// ldrsb/ldrsh.
def t_addrmode_rr_sext : MemOperand,
                    ComplexPattern<i32, 2, "SelectThumbAddrModeRRSext", []> {
  let EncoderMethod = "getThumbAddrModeRegRegOpValue";
  let PrintMethod = "printThumbAddrModeRROperand";
  let DecoderMethod = "DecodeThumbAddrModeRR";
  let ParserMatchClass = t_addrmode_rr_asm_operand;
  let MIOperandInfo = (ops tGPR:$base, tGPR:$offsreg);
}
```
- EN: Defines TableGen record `t_addrmode_rr_sext` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t_addrmode_rr_sext`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 207-224
```tablegen
// t_addrmode_rrs := reg + reg
//
// We use separate scaled versions because the Select* functions need
// to explicitly check for a matching constant and return false here so that
// the reg+imm forms will match instead. This is a horrible way to do that,
// as it forces tight coupling between the methods, but it's how selectiondag
// currently works.
def t_addrmode_rrs1 : MemOperand,
                      ComplexPattern<i32, 2, "SelectThumbAddrModeRI5S1", []> {
  let EncoderMethod = "getThumbAddrModeRegRegOpValue";
  let PrintMethod = "printThumbAddrModeRROperand";
  let DecoderMethod = "DecodeThumbAddrModeRR";
  let ParserMatchClass = t_addrmode_rr_asm_operand;
  let MIOperandInfo = (ops tGPR:$base, tGPR:$offsreg);
}
def t_addrmode_rrs2 : MemOperand,
                      ComplexPattern<i32, 2, "SelectThumbAddrModeRI5S2", []> {
  let EncoderMethod = "getThumbAddrModeRegRegOpValue";
```
- EN: Defines TableGen record `t_addrmode_rrs1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t_addrmode_rrs1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 225-237
```tablegen
  let DecoderMethod = "DecodeThumbAddrModeRR";
  let PrintMethod = "printThumbAddrModeRROperand";
  let ParserMatchClass = t_addrmode_rr_asm_operand;
  let MIOperandInfo = (ops tGPR:$base, tGPR:$offsreg);
}
def t_addrmode_rrs4 : MemOperand,
                      ComplexPattern<i32, 2, "SelectThumbAddrModeRI5S4", []> {
  let EncoderMethod = "getThumbAddrModeRegRegOpValue";
  let DecoderMethod = "DecodeThumbAddrModeRR";
  let PrintMethod = "printThumbAddrModeRROperand";
  let ParserMatchClass = t_addrmode_rr_asm_operand;
  let MIOperandInfo = (ops tGPR:$base, tGPR:$offsreg);
}
```
- EN: Defines TableGen record `t_addrmode_rrs4` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t_addrmode_rrs4`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 239-249
```tablegen
// t_addrmode_is4 := reg + imm5 * 4
//
def t_addrmode_is4_asm_operand : AsmOperandClass { let Name = "MemThumbRIs4"; }
def t_addrmode_is4 : MemOperand,
                     ComplexPattern<i32, 2, "SelectThumbAddrModeImm5S4", []> {
  let EncoderMethod = "getAddrModeISOpValue";
  let DecoderMethod = "DecodeThumbAddrModeIS";
  let PrintMethod = "printThumbAddrModeImm5S4Operand";
  let ParserMatchClass = t_addrmode_is4_asm_operand;
  let MIOperandInfo = (ops tGPR:$base, i32imm:$offsimm);
}
```
- EN: Defines TableGen record `t_addrmode_is4_asm_operand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t_addrmode_is4_asm_operand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 251-261
```tablegen
// t_addrmode_is2 := reg + imm5 * 2
//
def t_addrmode_is2_asm_operand : AsmOperandClass { let Name = "MemThumbRIs2"; }
def t_addrmode_is2 : MemOperand,
                     ComplexPattern<i32, 2, "SelectThumbAddrModeImm5S2", []> {
  let EncoderMethod = "getAddrModeISOpValue";
  let DecoderMethod = "DecodeThumbAddrModeIS";
  let PrintMethod = "printThumbAddrModeImm5S2Operand";
  let ParserMatchClass = t_addrmode_is2_asm_operand;
  let MIOperandInfo = (ops tGPR:$base, i32imm:$offsimm);
}
```
- EN: Defines TableGen record `t_addrmode_is2_asm_operand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t_addrmode_is2_asm_operand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 263-273
```tablegen
// t_addrmode_is1 := reg + imm5
//
def t_addrmode_is1_asm_operand : AsmOperandClass { let Name = "MemThumbRIs1"; }
def t_addrmode_is1 : MemOperand,
                     ComplexPattern<i32, 2, "SelectThumbAddrModeImm5S1", []> {
  let EncoderMethod = "getAddrModeISOpValue";
  let DecoderMethod = "DecodeThumbAddrModeIS";
  let PrintMethod = "printThumbAddrModeImm5S1Operand";
  let ParserMatchClass = t_addrmode_is1_asm_operand;
  let MIOperandInfo = (ops tGPR:$base, i32imm:$offsimm);
}
```
- EN: Defines TableGen record `t_addrmode_is1_asm_operand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t_addrmode_is1_asm_operand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 275-287
```tablegen
// t_addrmode_sp := sp + imm8 * 4
//
// FIXME: This really shouldn't have an explicit SP operand at all. It should
// be implicit, just like in the instruction encoding itself.
def t_addrmode_sp_asm_operand : AsmOperandClass { let Name = "MemThumbSPI"; }
def t_addrmode_sp : MemOperand,
                    ComplexPattern<i32, 2, "SelectThumbAddrModeSP", []> {
  let EncoderMethod = "getAddrModeThumbSPOpValue";
  let DecoderMethod = "DecodeThumbAddrModeSP";
  let PrintMethod = "printThumbAddrModeSPOperand";
  let ParserMatchClass = t_addrmode_sp_asm_operand;
  let MIOperandInfo = (ops GPR:$base, i32imm:$offsimm);
}
```
- EN: Defines TableGen record `t_addrmode_sp_asm_operand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t_addrmode_sp_asm_operand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 289-292
```tablegen
// Inspects parent to determine whether an or instruction can be implemented as
// an add (i.e. whether we know overflow won't occur in the add).
let WantsParent = true in
def AddLikeOrOp : ComplexPattern<i32, 1, "SelectAddLikeOr">;
```
- EN: Defines TableGen record `AddLikeOrOp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AddLikeOrOp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 294-295
```tablegen
// Pattern to exclude immediates from matching
def non_imm32 : PatLeaf<(i32 GPR), [{ return !isa<ConstantSDNode>(N); }]>;
```
- EN: Defines TableGen record `non_imm32` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `non_imm32`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 297-299
```tablegen
//===----------------------------------------------------------------------===//
//  Miscellaneous Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 301-308
```tablegen
// FIXME: Marking these as hasSideEffects is necessary to prevent machine DCE
// from removing one half of the matched pairs. That breaks PEI, which assumes
// these will always be in pairs, and asserts if it finds otherwise. Better way?
let Defs = [SP], Uses = [SP], hasSideEffects = 1 in {
def tADJCALLSTACKUP :
  PseudoInst<(outs), (ins i32imm:$amt1, i32imm:$amt2), NoItinerary,
             [(ARMcallseq_end imm:$amt1, imm:$amt2)]>,
            Requires<[IsThumb, IsThumb1Only]>;
```
- EN: Defines TableGen record `tADJCALLSTACKUP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tADJCALLSTACKUP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 310-314
```tablegen
def tADJCALLSTACKDOWN :
  PseudoInst<(outs), (ins i32imm:$amt, i32imm:$amt2), NoItinerary,
             [(ARMcallseq_start imm:$amt, imm:$amt2)]>,
            Requires<[IsThumb, IsThumb1Only]>;
}
```
- EN: Defines TableGen record `tADJCALLSTACKDOWN` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tADJCALLSTACKDOWN`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 316-320
```tablegen
class T1SystemEncoding<bits<8> opc>
  : T1Encoding<0b101111> {
  let Inst{9-8} = 0b11;
  let Inst{7-0} = opc;
}
```
- EN: Declares reusable TableGen class `T1SystemEncoding` for `ARMInstrThumb`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb` 声明可复用的 TableGen 类 `T1SystemEncoding`，通常用于抽象共享字段、谓词或编码结构。

### Lines 322-328
```tablegen
def tHINT : T1pI<(outs), (ins imm0_15:$imm), NoItinerary, "hint", "\t$imm",
                 [(int_arm_hint imm0_15:$imm)]>,
            T1SystemEncoding<0x00>,
            Requires<[IsThumb, HasV6M]> {
  bits<4> imm;
  let Inst{7-4} = imm;
}
```
- EN: Defines TableGen record `tHINT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tHINT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 330-333
```tablegen
// Note: When EmitPriority == 1, the alias will be used for printing
class tHintAlias<string Asm, dag Result, bit EmitPriority = 0> : tInstAlias<Asm, Result, EmitPriority> {
  let Predicates = [IsThumb, HasV6M];
}
```
- EN: Declares reusable TableGen class `tHintAlias` for `ARMInstrThumb`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb` 声明可复用的 TableGen 类 `tHintAlias`，通常用于抽象共享字段、谓词或编码结构。

### Lines 335-342
```tablegen
def : tHintAlias<"nop$p", (tHINT 0, pred:$p), 1>; // A8.6.110
def : tHintAlias<"yield$p", (tHINT 1, pred:$p), 1>; // A8.6.410
def : tHintAlias<"wfe$p", (tHINT 2, pred:$p), 1>; // A8.6.408
def : tHintAlias<"wfi$p", (tHINT 3, pred:$p), 1>; // A8.6.409
def : tHintAlias<"sev$p", (tHINT 4, pred:$p), 1>; // A8.6.157
def : tInstAlias<"sevl$p", (tHINT 5, pred:$p), 1> {
  let Predicates = [IsThumb2, HasV8];
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 344-355
```tablegen
// The imm operand $val can be used by a debugger to store more information
// about the breakpoint.
def tBKPT : T1I<(outs), (ins imm0_255:$val), NoItinerary, "bkpt\t$val",
                []>,
           T1Encoding<0b101111> {
  let Inst{9-8} = 0b10;
  // A8.6.22
  bits<8> val;
  let Inst{7-0} = val;
}
// default immediate for breakpoint mnemonic
def : InstAlias<"bkpt", (tBKPT 0), 0>, Requires<[IsThumb]>;
```
- EN: Defines TableGen record `tBKPT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tBKPT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 357-362
```tablegen
def tHLT : T1I<(outs), (ins imm0_63:$val), NoItinerary, "hlt\t$val",
                []>, T1Encoding<0b101110>, Requires<[IsThumb, HasV8]> {
  let Inst{9-6} = 0b1010;
  bits<6> val;
  let Inst{5-0} = val;
}
```
- EN: Defines TableGen record `tHLT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tHLT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 364-372
```tablegen
def tSETEND : T1I<(outs), (ins setend_op:$end), NoItinerary, "setend\t$end",
                  []>, T1Encoding<0b101101>, Requires<[IsThumb, IsNotMClass]>, Deprecated<HasV8Ops> {
  bits<1> end;
  // A8.6.156
  let Inst{9-5} = 0b10010;
  let Inst{4}   = 1;
  let Inst{3}   = end;
  let Inst{2-0} = 0b000;
}
```
- EN: Defines TableGen record `tSETEND` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tSETEND`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 374-380
```tablegen
// Change Processor State is a system instruction -- for disassembly only.
def tCPS : T1I<(outs), (ins imod_op:$imod, iflags_op:$iflags),
                NoItinerary, "cps$imod $iflags", []>,
           T1Misc<0b0110011> {
  // A8.6.38 & B6.1.1
  bit imod;
  bits<3> iflags;
```
- EN: Defines TableGen record `tCPS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tCPS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 382-386
```tablegen
  let Inst{4}   = imod;
  let Inst{3}   = 0;
  let Inst{2-0} = iflags;
  let DecoderMethod = "DecodeThumbCPS";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 388-397
```tablegen
// For both thumb1 and thumb2.
let isNotDuplicable = 1, isCodeGenOnly = 1 in
def tPICADD : TIt<(outs GPR:$dst), (ins GPR:$lhs, pclabel:$cp), IIC_iALUr, "",
                  [(set GPR:$dst, (ARMpic_add GPR:$lhs, imm:$cp))]>,
              T1Special<{0,0,?,?}>, Sched<[WriteALU]> {
  // A8.6.6
  bits<3> dst;
  let Inst{6-3} = 0b1111; // Rm = pc
  let Inst{2-0} = dst;
}
```
- EN: Defines TableGen record `tPICADD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tPICADD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 399-410
```tablegen
// ADD <Rd>, sp, #<imm8>
def tADDrSPi : T1pI<(outs tGPR:$dst), (ins GPRsp:$sp, t_imm0_1020s4:$imm),
                    IIC_iALUi, "add", "\t$dst, $sp, $imm", []>,
               T1Encoding<{1,0,1,0,1,?}>, Sched<[WriteALU]> {
  // A6.2 & A8.6.8
  bits<3> dst;
  bits<8> imm;
  let Inst{10-8} = dst;
  let Inst{7-0}  = imm;
  let DecoderMethod = "DecodeThumbAddSpecialReg";
  let hasSideEffects = 0;
}
```
- EN: Defines TableGen record `tADDrSPi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tADDrSPi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 412-418
```tablegen
// Thumb1 frame lowering is rather fragile, we hope to be able to use
// tADDrSPi, but we may need to insert a sequence that clobbers CPSR.
def tADDframe : PseudoInst<(outs tGPR:$dst), (ins i32imm:$base, i32imm:$offset),
                           NoItinerary, []>,
                Requires<[IsThumb, IsThumb1Only]> {
  let Defs = [CPSR];
}
```
- EN: Defines TableGen record `tADDframe` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tADDframe`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 420-429
```tablegen
// ADD sp, sp, #<imm7>
def tADDspi : T1pIt<(outs GPRsp:$Rdn), (ins GPRsp:$Rn, t_imm0_508s4:$imm),
                     IIC_iALUi, "add", "\t$Rdn, $imm", []>,
              T1Misc<{0,0,0,0,0,?,?}>, Sched<[WriteALU]> {
  // A6.2.5 & A8.6.8
  bits<7> imm;
  let Inst{6-0} = imm;
  let DecoderMethod = "DecodeThumbAddSPImm";
  let hasSideEffects = 0;
}
```
- EN: Defines TableGen record `tADDspi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tADDspi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 431-441
```tablegen
// SUB sp, sp, #<imm7>
// FIXME: The encoding and the ASM string don't match up.
def tSUBspi : T1pIt<(outs GPRsp:$Rdn), (ins GPRsp:$Rn, t_imm0_508s4:$imm),
                    IIC_iALUi, "sub", "\t$Rdn, $imm", []>,
              T1Misc<{0,0,0,0,1,?,?}>, Sched<[WriteALU]> {
  // A6.2.5 & A8.6.214
  bits<7> imm;
  let Inst{6-0} = imm;
  let DecoderMethod = "DecodeThumbAddSPImm";
  let hasSideEffects = 0;
}
```
- EN: Defines TableGen record `tSUBspi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tSUBspi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 443-446
```tablegen
def : tInstSubst<"add${p} sp, $imm",
                 (tSUBspi SP, t_imm0_508s4_neg:$imm, pred:$p)>;
def : tInstSubst<"add${p} sp, sp, $imm",
                 (tSUBspi SP, t_imm0_508s4_neg:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 448-452
```tablegen
// Can optionally specify SP as a three operand instruction.
def : tInstAlias<"add${p} sp, sp, $imm",
                 (tADDspi SP, t_imm0_508s4:$imm, pred:$p)>;
def : tInstAlias<"sub${p} sp, sp, $imm",
                 (tSUBspi SP, t_imm0_508s4:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 454-465
```tablegen
// ADD <Rm>, sp
def tADDrSP : T1pI<(outs GPR:$Rdn), (ins GPRsp:$sp, GPR:$Rn), IIC_iALUr,
                   "add", "\t$Rdn, $sp, $Rn", []>,
              T1Special<{0,0,?,?}>, Sched<[WriteALU]> {
  // A8.6.9 Encoding T1
  bits<4> Rdn;
  let Inst{7}   = Rdn{3};
  let Inst{6-3} = 0b1101;
  let Inst{2-0} = Rdn{2-0};
  let DecoderMethod = "DecodeThumbAddSPReg";
  let hasSideEffects = 0;
}
```
- EN: Defines TableGen record `tADDrSP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tADDrSP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 467-478
```tablegen
// ADD sp, <Rm>
def tADDspr : T1pIt<(outs GPRsp:$Rdn), (ins GPRsp:$Rn, GPR:$Rm), IIC_iALUr,
                  "add", "\t$Rdn, $Rm", []>,
              T1Special<{0,0,?,?}>, Sched<[WriteALU]> {
  // A8.6.9 Encoding T2
  bits<4> Rm;
  let Inst{7} = 1;
  let Inst{6-3} = Rm;
  let Inst{2-0} = 0b101;
  let DecoderMethod = "DecodeThumbAddSPReg";
  let hasSideEffects = 0;
}
```
- EN: Defines TableGen record `tADDspr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tADDspr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 480-482
```tablegen
//===----------------------------------------------------------------------===//
//  Control Flow Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 484-501
```tablegen
// Indirect branches
let isBranch = 1, isTerminator = 1, isBarrier = 1, isIndirectBranch = 1, hasSideEffects = 0 in {
  def tBX : TI<(outs), (ins GPR:$Rm, pred:$p), IIC_Br, "bx${p}\t$Rm", []>,
            T1Special<{1,1,0,?}>, Sched<[WriteBr]> {
    // A6.2.3 & A8.6.25
    bits<0> p;
    bits<4> Rm;
    let Inst{6-3} = Rm;
    let Inst{2-0} = 0b000;
    let Unpredictable{2-0} = 0b111;
  }
  def tBXNS : TI<(outs), (ins GPR:$Rm, pred:$p), IIC_Br, "bxns${p}\t$Rm", []>,
              Requires<[IsThumb, Has8MSecExt]>,
              T1Special<{1,1,0,?}>, Sched<[WriteBr]> {
    bits<0> p;
    bits<4> Rm;
    let Inst{6-3} = Rm;
    let Inst{2-0} = 0b100;
```
- EN: Defines TableGen record `tBX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tBX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 502-504
```tablegen
    let Unpredictable{1-0} = 0b11;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 506-508
```tablegen
let isReturn = 1, isTerminator = 1, isBarrier = 1 in {
  def tBX_RET : tPseudoExpand<(outs), (ins pred:$p), 2, IIC_Br,
                   [(ARMretglue)], (tBX LR, pred:$p)>, Sched<[WriteBr]>;
```
- EN: Defines TableGen record `tBX_RET` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tBX_RET`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 510-512
```tablegen
  // alternative return for CMSE entry functions
  def tBXNS_RET : tPseudoInst<(outs), (ins), 2, IIC_Br,
                  [(ARMseretglue)]>, Sched<[WriteBr]>;
```
- EN: Defines TableGen record `tBXNS_RET` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tBXNS_RET`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 514-518
```tablegen
  // Alternative return instruction used by vararg functions.
  def tBX_RET_vararg : tPseudoExpand<(outs), (ins tGPR:$Rm, pred:$p),
                   2, IIC_Br, [],
                   (tBX GPR:$Rm, pred:$p)>, Sched<[WriteBr]>;
}
```
- EN: Defines TableGen record `tBX_RET_vararg` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tBX_RET_vararg`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 520-537
```tablegen
// All calls clobber the non-callee saved registers. SP is marked as a use to
// prevent stack-pointer assignments that appear immediately before calls from
// potentially appearing dead.
let isCall = 1,
  Defs = [LR], Uses = [SP] in {
  // Also used for Thumb2
  def tBL  : TIx2<0b11110, 0b11, 1,
                  (outs), (ins pred:$p, thumb_bl_target:$func), IIC_Br,
                  "bl${p}\t$func",
                  [(ARMcall tglobaladdr:$func)]>,
             Requires<[IsThumb]>, Sched<[WriteBrL]> {
    bits<0> p;
    bits<24> func;
    let Inst{26} = func{23};
    let Inst{25-16} = func{20-11};
    let Inst{13} = func{22};
    let Inst{11} = func{21};
    let Inst{10-0} = func{10-0};
```
- EN: Defines TableGen record `tBL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tBL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 538-538
```tablegen
  }
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 540-553
```tablegen
  // ARMv5T and above, also used for Thumb2
  def tBLXi : TIx2<0b11110, 0b11, 0,
                 (outs), (ins pred:$p, thumb_blx_target:$func), IIC_Br,
                   "blx${p}\t$func", []>,
              Requires<[IsThumb, HasV5T, IsNotMClass]>, Sched<[WriteBrL]> {
    bits<0> p;
    bits<24> func;
    let Inst{26} = func{23};
    let Inst{25-16} = func{20-11};
    let Inst{13} = func{22};
    let Inst{11} = func{21};
    let Inst{10-1} = func{10-1};
    let Inst{0} = 0; // func{0} is assumed zero
  }
```
- EN: Defines TableGen record `tBLXi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tBLXi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 555-568
```tablegen
  // Also used for Thumb2
  def tBLXr : TI<(outs), (ins pred:$p, GPR:$func), IIC_Br,
                  "blx${p}\t$func", []>,
              Requires<[IsThumb, HasV5T]>,
              T1Special<{1,1,1,?}>, Sched<[WriteBrL]> { // A6.2.3 & A8.6.24;
    bits<0> p;
    bits<4> func;
    let Inst{6-3} = func;
    let Inst{2-0} = 0b000;
  }
  def tBLXr_noip :  ARMPseudoExpand<(outs), (ins pred:$p, GPRnoip:$func),
                   2, IIC_Br, [], (tBLXr pred:$p, GPR:$func)>,
                   Requires<[IsThumb, HasV5T]>,
                   Sched<[WriteBrL]>;
```
- EN: Defines TableGen record `tBLXr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tBLXr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 571-581
```tablegen
  // ARMv8-M Security Extensions
  def tBLXNSr : TI<(outs), (ins pred:$p, GPRnopc:$func), IIC_Br,
                   "blxns${p}\t$func", []>,
                Requires<[IsThumb, Has8MSecExt]>,
                T1Special<{1,1,1,?}>, Sched<[WriteBrL]> {
    bits<0> p;
    bits<4> func;
    let Inst{6-3} = func;
    let Inst{2-0} = 0b100;
    let Unpredictable{1-0} = 0b11;
  }
```
- EN: Defines TableGen record `tBLXNSr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tBLXNSr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 583-585
```tablegen
  def tBLXNS_CALL : PseudoInst<(outs), (ins GPRnopc:$func), IIC_Br,
                    [(ARMtsecall GPRnopc:$func)]>,
                    Requires<[IsThumb, Has8MSecExt]>, Sched<[WriteBr]>;
```
- EN: Defines TableGen record `tBLXNS_CALL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tBLXNS_CALL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 587-591
```tablegen
  // ARMv4T
  def tBX_CALL : tPseudoInst<(outs), (ins tGPR:$func),
                  4, IIC_Br,
                  [(ARMcall_nolink tGPR:$func)]>,
            Requires<[IsThumb, IsThumb1Only]>, Sched<[WriteBr]>;
```
- EN: Defines TableGen record `tBX_CALL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tBX_CALL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 593-599
```tablegen
  // Also used for Thumb2
  // push lr before the call
  def tBL_PUSHLR : tPseudoInst<(outs), (ins GPRlr:$ra, pred:$p, thumb_bl_target:$func),
                  4, IIC_Br,
                  []>,
             Requires<[IsThumb]>, Sched<[WriteBr]>;
}
```
- EN: Defines TableGen record `tBL_PUSHLR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tBL_PUSHLR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 601-604
```tablegen
def : ARMPat<(ARMcall GPR:$func), (tBLXr $func)>,
      Requires<[IsThumb, HasV5T, NoSLSBLRMitigation]>;
def : ARMPat<(ARMcall GPRnoip:$func), (tBLXr_noip $func)>,
      Requires<[IsThumb, HasV5T, SLSBLRMitigation]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 606-614
```tablegen
let isBranch = 1, isTerminator = 1, isBarrier = 1 in {
  let isPredicable = 1 in
  def tB   : T1pI<(outs), (ins t_brtarget:$target), IIC_Br,
                 "b", "\t$target", [(br bb:$target)]>,
             T1Encoding<{1,1,1,0,0,?}>, Sched<[WriteBr]> {
    bits<11> target;
    let Inst{10-0} = target;
    let AsmMatchConverter = "cvtThumbBranches";
 }
```
- EN: Defines TableGen record `tB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 616-623
```tablegen
  // Far jump
  // Just a pseudo for a tBL instruction. Needed to let regalloc know about
  // the clobber of LR.
  let Defs = [LR] in
  def tBfar : tPseudoExpand<(outs), (ins thumb_bl_target:$target, pred:$p),
                          4, IIC_Br, [],
                          (tBL pred:$p, thumb_bl_target:$target)>,
                          Sched<[WriteBrTbl]>;
```
- EN: Defines TableGen record `tBfar` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tBfar`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 625-634
```tablegen
  def tBR_JTr : tPseudoInst<(outs),
                      (ins tGPR:$target, i32imm:$jt),
                      0, IIC_Br,
                      [(ARMbrjt tGPR:$target, tjumptable:$jt)]>,
                      Sched<[WriteBrTbl]> {
    let Size = 2;
    let isNotDuplicable = 1;
    list<Predicate> Predicates = [IsThumb, IsThumb1Only];
  }
}
```
- EN: Defines TableGen record `tBR_JTr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tBR_JTr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 636-648
```tablegen
// FIXME: should be able to write a pattern for ARMBrcond, but can't use
// a two-value operand where a dag node expects two operands. :(
let isBranch = 1, isTerminator = 1, hasSideEffects = 0 in
  def tBcc : T1I<(outs), (ins thumb_bcc_target:$target, pred:$p), IIC_Br,
                 "b${p}\t$target",
                 [/*(ARMbrcond bb:$target, imm:$cc)*/]>,
             T1BranchCond<{1,1,0,1}>, Sched<[WriteBr]> {
  bits<4> p;
  bits<8> target;
  let Inst{11-8} = p;
  let Inst{7-0} = target;
  let AsmMatchConverter = "cvtThumbBranches";
}
```
- EN: Defines TableGen record `tBcc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tBcc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 651-668
```tablegen
// Tail calls
let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1 in {
  // IOS versions.
  let Uses = [SP] in {
    def tTAILJMPr : tPseudoExpand<(outs), (ins tcGPR:$dst),
                     4, IIC_Br, [],
                     (tBX GPR:$dst, (ops 14, zero_reg))>,
                     Requires<[IsThumb]>, Sched<[WriteBr]>;
  }
  // tTAILJMPd: MachO version uses a Thumb2 branch (no Thumb1 tail calls
  // on MachO), so it's in ARMInstrThumb2.td.
  // Non-MachO version:
  let Uses = [SP] in {
    def tTAILJMPdND : tPseudoExpand<(outs),
                   (ins t_brtarget:$dst, pred:$p),
                   4, IIC_Br, [],
                   (tB t_brtarget:$dst, pred:$p)>,
                 Requires<[IsThumb, IsNotMachO]>, Sched<[WriteBr]>;
```
- EN: Defines TableGen record `tTAILJMPr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tTAILJMPr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 669-670
```tablegen
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 673-683
```tablegen
// A8.6.218 Supervisor Call (Software Interrupt)
// A8.6.16 B: Encoding T1
// If Inst{11-8} == 0b1111 then SEE SVC
let isCall = 1, Uses = [SP] in
def tSVC : T1pI<(outs), (ins imm0_255:$imm), IIC_Br,
                "svc", "\t$imm", []>, Encoding16, Sched<[WriteBr]> {
  bits<8> imm;
  let Inst{15-12} = 0b1101;
  let Inst{11-8}  = 0b1111;
  let Inst{7-0}   = imm;
}
```
- EN: Defines TableGen record `tSVC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tSVC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 685-690
```tablegen
// The assembler uses 0xDEFE for a trap instruction.
let isTrap = 1 in
def tTRAP : TI<(outs), (ins), IIC_Br,
               "trap", [(trap)]>, Encoding16, Sched<[WriteBr]> {
  let Inst = 0xdefe;
}
```
- EN: Defines TableGen record `tTRAP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tTRAP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 692-694
```tablegen
//===----------------------------------------------------------------------===//
//  Load Store Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 696-709
```tablegen
// PC-relative loads need to be matched first as constant pool accesses need to
// always be PC-relative. We do this using AddedComplexity, as the pattern is
// simpler than the patterns of the other load instructions.
let canFoldAsLoad = 1, isReMaterializable = 1, AddedComplexity = 10 in
def tLDRpci : T1pIs<(outs tGPR:$Rt), (ins t_addrmode_pc:$addr), IIC_iLoad_i,
                  "ldr", "\t$Rt, $addr",
                  [(set tGPR:$Rt, (load (ARMWrapper tconstpool:$addr)))]>,
              T1Encoding<{0,1,0,0,1,?}>, Sched<[WriteLd]> {
  // A6.2 & A8.6.59
  bits<3> Rt;
  bits<8> addr;
  let Inst{10-8} = Rt;
  let Inst{7-0}  = addr;
}
```
- EN: Defines TableGen record `tLDRpci` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tLDRpci`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 711-722
```tablegen
// SP-relative loads should be matched before standard immediate-offset loads as
// it means we avoid having to move SP to another register.
let canFoldAsLoad = 1 in
def tLDRspi : T1pIs<(outs tGPR:$Rt), (ins t_addrmode_sp:$addr), IIC_iLoad_i,
                    "ldr", "\t$Rt, $addr",
                    [(set tGPR:$Rt, (load t_addrmode_sp:$addr))]>,
              T1LdStSP<{1,?,?}>, Sched<[WriteLd]> {
  bits<3> Rt;
  bits<8> addr;
  let Inst{10-8} = Rt;
  let Inst{7-0} = addr;
}
```
- EN: Defines TableGen record `tLDRspi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tLDRspi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 724-741
```tablegen
// Loads: reg/reg and reg/imm5
let canFoldAsLoad = 1, isReMaterializable = 1 in
multiclass thumb_ld_rr_ri_enc<bits<3> reg_opc, bits<4> imm_opc,
                              Operand AddrMode_r, Operand AddrMode_i,
                              AddrMode am, InstrItinClass itin_r,
                              InstrItinClass itin_i, string asm,
                              PatFrag opnode> {
  // Immediate-offset loads should be matched before register-offset loads as
  // when the offset is a constant it's simpler to first check if it fits in the
  // immediate offset field then fall back to register-offset if it doesn't.
  def i : // reg/imm5
    T1pILdStEncodeImm<imm_opc, 1 /* Load */,
                      (outs tGPR:$Rt), (ins AddrMode_i:$addr),
                      am, itin_i, asm, "\t$Rt, $addr",
                      [(set tGPR:$Rt, (opnode AddrMode_i:$addr))]>;
  // Register-offset loads are matched last.
  def r : // reg/reg
    T1pILdStEncode<reg_opc,
```
- EN: Declares TableGen `multiclass thumb_ld_rr_ri_enc`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass thumb_ld_rr_ri_enc`，它是一个可复用模板，可展开为多个相关记录。

### Lines 742-759
```tablegen
                   (outs tGPR:$Rt), (ins AddrMode_r:$addr),
                   am, itin_r, asm, "\t$Rt, $addr",
                   [(set tGPR:$Rt, (opnode AddrMode_r:$addr))]>;
}
// Stores: reg/reg and reg/imm5
multiclass thumb_st_rr_ri_enc<bits<3> reg_opc, bits<4> imm_opc,
                              Operand AddrMode_r, Operand AddrMode_i,
                              AddrMode am, InstrItinClass itin_r,
                              InstrItinClass itin_i, string asm,
                              PatFrag opnode> {
  def i : // reg/imm5
    T1pILdStEncodeImm<imm_opc, 0 /* Store */,
                      (outs), (ins tGPR:$Rt, AddrMode_i:$addr),
                      am, itin_i, asm, "\t$Rt, $addr",
                      [(opnode tGPR:$Rt, AddrMode_i:$addr)]>;
  def r : // reg/reg
    T1pILdStEncode<reg_opc,
                   (outs), (ins tGPR:$Rt, AddrMode_r:$addr),
```
- EN: Declares TableGen `multiclass thumb_st_rr_ri_enc`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass thumb_st_rr_ri_enc`，它是一个可复用模板，可展开为多个相关记录。

### Lines 760-762
```tablegen
                   am, itin_r, asm, "\t$Rt, $addr",
                   [(opnode tGPR:$Rt, AddrMode_r:$addr)]>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 764-768
```tablegen
// A8.6.57 & A8.6.60
defm tLDR  : thumb_ld_rr_ri_enc<0b100, 0b0110, t_addrmode_rr,
                                t_addrmode_is4, AddrModeT1_4,
                                IIC_iLoad_r, IIC_iLoad_i, "ldr",
                                load>, Sched<[WriteLd]>;
```
- EN: Defines TableGen record `tLDR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tLDR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 770-774
```tablegen
// A8.6.64 & A8.6.61
defm tLDRB : thumb_ld_rr_ri_enc<0b110, 0b0111, t_addrmode_rr,
                                t_addrmode_is1, AddrModeT1_1,
                                IIC_iLoad_bh_r, IIC_iLoad_bh_i, "ldrb",
                                zextloadi8>, Sched<[WriteLd]>;
```
- EN: Defines TableGen record `tLDRB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tLDRB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 776-780
```tablegen
// A8.6.76 & A8.6.73
defm tLDRH : thumb_ld_rr_ri_enc<0b101, 0b1000, t_addrmode_rr,
                                t_addrmode_is2, AddrModeT1_2,
                                IIC_iLoad_bh_r, IIC_iLoad_bh_i, "ldrh",
                                zextloadi16>, Sched<[WriteLd]>;
```
- EN: Defines TableGen record `tLDRH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tLDRH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 782-787
```tablegen
let AddedComplexity = 10 in
def tLDRSB :                    // A8.6.80
  T1pILdStEncode<0b011, (outs tGPR:$Rt), (ins t_addrmode_rr_sext:$addr),
                 AddrModeT1_1, IIC_iLoad_bh_r,
                 "ldrsb", "\t$Rt, $addr",
                 [(set tGPR:$Rt, (sextloadi8 t_addrmode_rr_sext:$addr))]>, Sched<[WriteLd]>;
```
- EN: Defines TableGen record `tLDRSB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tLDRSB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 789-794
```tablegen
let AddedComplexity = 10 in
def tLDRSH :                    // A8.6.84
  T1pILdStEncode<0b111, (outs tGPR:$Rt), (ins t_addrmode_rr_sext:$addr),
                 AddrModeT1_2, IIC_iLoad_bh_r,
                 "ldrsh", "\t$Rt, $addr",
                 [(set tGPR:$Rt, (sextloadi16 t_addrmode_rr_sext:$addr))]>, Sched<[WriteLd]>;
```
- EN: Defines TableGen record `tLDRSH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tLDRSH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 797-805
```tablegen
def tSTRspi : T1pIs<(outs), (ins tGPR:$Rt, t_addrmode_sp:$addr), IIC_iStore_i,
                    "str", "\t$Rt, $addr",
                    [(store tGPR:$Rt, t_addrmode_sp:$addr)]>,
              T1LdStSP<{0,?,?}>, Sched<[WriteST]> {
  bits<3> Rt;
  bits<8> addr;
  let Inst{10-8} = Rt;
  let Inst{7-0} = addr;
}
```
- EN: Defines TableGen record `tSTRspi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tSTRspi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 807-811
```tablegen
// A8.6.194 & A8.6.192
defm tSTR  : thumb_st_rr_ri_enc<0b000, 0b0110, t_addrmode_rr,
                                t_addrmode_is4, AddrModeT1_4,
                                IIC_iStore_r, IIC_iStore_i, "str",
                                store>, Sched<[WriteST]>;
```
- EN: Defines TableGen record `tSTR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tSTR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 813-817
```tablegen
// A8.6.197 & A8.6.195
defm tSTRB : thumb_st_rr_ri_enc<0b010, 0b0111, t_addrmode_rr,
                                t_addrmode_is1, AddrModeT1_1,
                                IIC_iStore_bh_r, IIC_iStore_bh_i, "strb",
                                truncstorei8>, Sched<[WriteST]>;
```
- EN: Defines TableGen record `tSTRB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tSTRB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 819-823
```tablegen
// A8.6.207 & A8.6.205
defm tSTRH : thumb_st_rr_ri_enc<0b001, 0b1000, t_addrmode_rr,
                               t_addrmode_is2, AddrModeT1_2,
                               IIC_iStore_bh_r, IIC_iStore_bh_i, "strh",
                               truncstorei16>, Sched<[WriteST]>;
```
- EN: Defines TableGen record `tSTRH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tSTRH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 826-828
```tablegen
//===----------------------------------------------------------------------===//
//  Load / store multiple Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 830-831
```tablegen
// These require base address to be written back or one of the loaded regs.
let hasSideEffects = 0 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 833-841
```tablegen
let mayLoad = 1, hasExtraDefRegAllocReq = 1, variadicOpsAreDefs = 1 in
def tLDMIA : T1I<(outs), (ins tGPR:$Rn, pred:$p, reglist:$regs, variable_ops),
        IIC_iLoad_m, "ldm${p}\t$Rn, $regs", []>, T1Encoding<{1,1,0,0,1,?}> {
  bits<0> p;
  bits<3> Rn;
  bits<8> regs;
  let Inst{10-8} = Rn;
  let Inst{7-0}  = regs;
}
```
- EN: Defines TableGen record `tLDMIA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tLDMIA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 843-858
```tablegen
// Writeback version is just a pseudo, as there's no encoding difference.
// Writeback happens iff the base register is not in the destination register
// list.
let mayLoad = 1, hasExtraDefRegAllocReq = 1 in
def tLDMIA_UPD :
    InstTemplate<AddrModeNone, 0, IndexModeNone, Pseudo, GenericDomain,
                 "$Rn = $wb", IIC_iLoad_mu>,
    PseudoInstExpansion<(tLDMIA tGPR:$Rn, pred:$p, reglist:$regs)> {
  let Size = 2;
  let OutOperandList = (outs tGPR:$wb);
  let InOperandList = (ins tGPR:$Rn, pred:$p, reglist:$regs, variable_ops);
  let Pattern = [];
  let isCodeGenOnly = 1;
  let isPseudo = 1;
  list<Predicate> Predicates = [IsThumb];
}
```
- EN: Defines TableGen record `tLDMIA_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tLDMIA_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 860-872
```tablegen
// There is no non-writeback version of STM for Thumb.
let mayStore = 1, hasExtraSrcRegAllocReq = 1 in
def tSTMIA_UPD : Thumb1I<(outs tGPR:$wb),
                         (ins tGPR:$Rn, pred:$p, reglist:$regs, variable_ops),
                         AddrModeNone, 2, IIC_iStore_mu,
                         "stm${p}\t$Rn!, $regs", "$Rn = $wb", []>,
                     T1Encoding<{1,1,0,0,0,?}> {
  bits<0> p;
  bits<3> Rn;
  bits<8> regs;
  let Inst{10-8} = Rn;
  let Inst{7-0}  = regs;
}
```
- EN: Defines TableGen record `tSTMIA_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tSTMIA_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 874-874
```tablegen
} // hasSideEffects
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 876-878
```tablegen
def : InstAlias<"ldm${p} $Rn!, $regs",
                (tLDMIA tGPR:$Rn, pred:$p, reglist:$regs), 0>,
        Requires<[IsThumb, IsThumb1Only]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 880-890
```tablegen
let mayLoad = 1, Uses = [SP], Defs = [SP], hasExtraDefRegAllocReq = 1,
    variadicOpsAreDefs = 1 in
def tPOP : T1I<(outs), (ins pred:$p, reglist:$regs, variable_ops),
               IIC_iPop,
               "pop${p}\t$regs", []>,
           T1Misc<{1,1,0,?,?,?,?}>, Sched<[WriteLd]> {
  bits<0> p;
  bits<16> regs;
  let Inst{8}   = regs{15};
  let Inst{7-0} = regs{7-0};
}
```
- EN: Defines TableGen record `tPOP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tPOP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 892-901
```tablegen
let mayStore = 1, Uses = [SP], Defs = [SP], hasExtraSrcRegAllocReq = 1 in
def tPUSH : T1I<(outs), (ins pred:$p, reglist:$regs, variable_ops),
                IIC_iStore_m,
                "push${p}\t$regs", []>,
            T1Misc<{0,1,0,?,?,?,?}>, Sched<[WriteST]> {
  bits<0> p;
  bits<16> regs;
  let Inst{8}   = regs{14};
  let Inst{7-0} = regs{7-0};
}
```
- EN: Defines TableGen record `tPUSH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tPUSH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 903-905
```tablegen
//===----------------------------------------------------------------------===//
//  Arithmetic Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 907-924
```tablegen
// Helper classes for encoding T1pI patterns:
class T1pIDPEncode<bits<4> opA, dag oops, dag iops, InstrItinClass itin,
                   string opc, string asm, list<dag> pattern>
    : T1pI<oops, iops, itin, opc, asm, pattern>,
      T1DataProcessing<opA> {
  bits<3> Rm;
  bits<3> Rn;
  let Inst{5-3} = Rm;
  let Inst{2-0} = Rn;
}
class T1pIMiscEncode<bits<7> opA, dag oops, dag iops, InstrItinClass itin,
                     string opc, string asm, list<dag> pattern>
    : T1pI<oops, iops, itin, opc, asm, pattern>,
      T1Misc<opA> {
  bits<3> Rm;
  bits<3> Rd;
  let Inst{5-3} = Rm;
  let Inst{2-0} = Rd;
```
- EN: Declares reusable TableGen class `T1pIDPEncode` for `ARMInstrThumb`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb` 声明可复用的 TableGen 类 `T1pIDPEncode`，通常用于抽象共享字段、谓词或编码结构。

### Lines 925-925
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 927-944
```tablegen
// Helper classes for encoding T1sI patterns:
class T1sIDPEncode<bits<4> opA, dag oops, dag iops, InstrItinClass itin,
                   string opc, string asm, list<dag> pattern>
    : T1sI<oops, iops, itin, opc, asm, pattern>,
      T1DataProcessing<opA> {
  bits<3> Rd;
  bits<3> Rn;
  let Inst{5-3} = Rn;
  let Inst{2-0} = Rd;
}
class T1sIGenEncode<bits<5> opA, dag oops, dag iops, InstrItinClass itin,
                    string opc, string asm, list<dag> pattern>
    : T1sI<oops, iops, itin, opc, asm, pattern>,
      T1General<opA> {
  bits<3> Rm;
  bits<3> Rn;
  bits<3> Rd;
  let Inst{8-6} = Rm;
```
- EN: Declares reusable TableGen class `T1sIDPEncode` for `ARMInstrThumb`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb` 声明可复用的 TableGen 类 `T1sIDPEncode`，通常用于抽象共享字段、谓词或编码结构。

### Lines 945-956
```tablegen
  let Inst{5-3} = Rn;
  let Inst{2-0} = Rd;
}
class T1sIGenEncodeImm<bits<5> opA, dag oops, dag iops, InstrItinClass itin,
                       string opc, string asm, list<dag> pattern>
    : T1sI<oops, iops, itin, opc, asm, pattern>,
      T1General<opA> {
  bits<3> Rd;
  bits<3> Rm;
  let Inst{5-3} = Rm;
  let Inst{2-0} = Rd;
}
```
- EN: Declares reusable TableGen class `T1sIGenEncodeImm` for `ARMInstrThumb`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb` 声明可复用的 TableGen 类 `T1sIGenEncodeImm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 958-975
```tablegen
// Helper classes for encoding T1sIt patterns:
class T1sItDPEncode<bits<4> opA, dag oops, dag iops, InstrItinClass itin,
                    string opc, string asm, list<dag> pattern>
    : T1sIt<oops, iops, itin, opc, asm, pattern>,
      T1DataProcessing<opA> {
  bits<3> Rdn;
  bits<3> Rm;
  let Inst{5-3} = Rm;
  let Inst{2-0} = Rdn;
}
class T1sItGenEncodeImm<bits<5> opA, dag oops, dag iops, InstrItinClass itin,
                        string opc, string asm, list<dag> pattern>
    : T1sIt<oops, iops, itin, opc, asm, pattern>,
      T1General<opA> {
  bits<3> Rdn;
  bits<8> imm8;
  let Inst{10-8} = Rdn;
  let Inst{7-0}  = imm8;
```
- EN: Declares reusable TableGen class `T1sItDPEncode` for `ARMInstrThumb`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrThumb` 声明可复用的 TableGen 类 `T1sItDPEncode`，通常用于抽象共享字段、谓词或编码结构。

### Lines 976-976
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 978-984
```tablegen
let isAdd = 1, hasSideEffects = 0 in {
  // Add with carry register
  let isCommutable = 1, Uses = [CPSR] in
  def tADC :                      // A8.6.2
    T1sItDPEncode<0b0101, (outs tGPR:$Rdn), (ins tGPR:$Rn, tGPR:$Rm), IIC_iALUr,
                  "adc", "\t$Rdn, $Rm",
                  []>, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tADC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tADC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 986-995
```tablegen
  // Add immediate
  def tADDi3 :                    // A8.6.4 T1
    T1sIGenEncodeImm<0b01110, (outs tGPR:$Rd), (ins tGPR:$Rm, imm0_7:$imm3),
                     IIC_iALUi,
                     "add", "\t$Rd, $Rm, $imm3",
                     [(set tGPR:$Rd, (add tGPR:$Rm, imm0_7:$imm3))]>,
                     Sched<[WriteALU]> {
    bits<3> imm3;
    let Inst{8-6} = imm3;
  }
```
- EN: Defines TableGen record `tADDi3` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tADDi3`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 997-1002
```tablegen
  def tADDi8 :                    // A8.6.4 T2
    T1sItGenEncodeImm<{1,1,0,?,?}, (outs tGPR:$Rdn),
                      (ins tGPR:$Rn, imm0_255_expr:$imm8), IIC_iALUi,
                      "add", "\t$Rdn, $imm8",
                      [(set tGPR:$Rdn, (add tGPR:$Rn, imm0_255_expr:$imm8))]>,
                      Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tADDi8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tADDi8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1004-1011
```tablegen
  // Add register
  let isCommutable = 1 in
  def tADDrr :                    // A8.6.6 T1
    T1sIGenEncode<0b01100, (outs tGPR:$Rd), (ins tGPR:$Rn, tGPR:$Rm),
                  IIC_iALUr,
                  "add", "\t$Rd, $Rn, $Rm",
                  [(set tGPR:$Rd, (add tGPR:$Rn, tGPR:$Rm))]>,
                  Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tADDrr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tADDrr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1013-1025
```tablegen
  /// Similar to the above except these set the 's' bit so the
  /// instruction modifies the CPSR register.
  ///
  /// These opcodes will be converted to the real non-S opcodes by
  /// AdjustInstrPostInstrSelection after giving then an optional CPSR operand.
  let hasPostISelHook = 1, Defs = [CPSR] in {
    let isCommutable = 1, Uses = [CPSR] in
    def tADCS : tPseudoInst<(outs tGPR:$Rdn), (ins tGPR:$Rn, tGPR:$Rm),
                            2, IIC_iALUr,
                            [(set tGPR:$Rdn, CPSR, (ARMadde tGPR:$Rn, tGPR:$Rm,
                                                            CPSR))]>,
                Requires<[IsThumb1Only]>,
                Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tADCS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tADCS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1027-1032
```tablegen
    def tADDSi3 : tPseudoInst<(outs tGPR:$Rd), (ins tGPR:$Rm, imm0_7:$imm3),
                              2, IIC_iALUi,
                              [(set tGPR:$Rd, CPSR, (ARMaddc tGPR:$Rm,
                                                             imm0_7:$imm3))]>,
                  Requires<[IsThumb1Only]>,
                  Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tADDSi3` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tADDSi3`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1034-1039
```tablegen
    def tADDSi8 : tPseudoInst<(outs tGPR:$Rdn), (ins tGPR:$Rn, imm0_255_expr:$imm8),
                              2, IIC_iALUi,
                              [(set tGPR:$Rdn, CPSR, (ARMaddc tGPR:$Rn,
                                                      imm0_255_expr:$imm8))]>,
                  Requires<[IsThumb1Only]>,
                  Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tADDSi8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tADDSi8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1041-1048
```tablegen
    let isCommutable = 1 in
    def tADDSrr : tPseudoInst<(outs tGPR:$Rd), (ins tGPR:$Rn, tGPR:$Rm),
                              2, IIC_iALUr,
                              [(set tGPR:$Rd, CPSR, (ARMaddc tGPR:$Rn,
                                                             tGPR:$Rm))]>,
                  Requires<[IsThumb1Only]>,
                  Sched<[WriteALU]>;
  }
```
- EN: Defines TableGen record `tADDSrr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tADDSrr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1050-1061
```tablegen
  let hasSideEffects = 0 in
  def tADDhirr : T1pIt<(outs GPR:$Rdn), (ins GPR:$Rn, GPR:$Rm), IIC_iALUr,
                       "add", "\t$Rdn, $Rm", []>,
                 T1Special<{0,0,?,?}>, Sched<[WriteALU]> {
    // A8.6.6 T2
    bits<4> Rdn;
    bits<4> Rm;
    let Inst{7}   = Rdn{3};
    let Inst{6-3} = Rm;
    let Inst{2-0} = Rdn{2-0};
  }
}
```
- EN: Defines TableGen record `tADDhirr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tADDhirr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1063-1065
```tablegen
// Thumb has more flexible short encodings for ADD than ORR, so use those where
// possible.
def : T1Pat<(or AddLikeOrOp:$Rn, imm0_7:$imm), (tADDi3 $Rn, imm0_7:$imm)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1067-1067
```tablegen
def : T1Pat<(or AddLikeOrOp:$Rn, imm8_255:$imm), (tADDi8 $Rn, imm8_255:$imm)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1069-1069
```tablegen
def : T1Pat<(or AddLikeOrOp:$Rn, tGPR:$Rm), (tADDrr $Rn, $Rm)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1072-1073
```tablegen
def : tInstAlias <"add${s}${p} $Rdn, $Rm",
                 (tADDrr tGPR:$Rdn,s_cc_out:$s, tGPR:$Rdn, tGPR:$Rm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1075-1078
```tablegen
def : tInstSubst<"sub${s}${p} $rd, $rn, $imm",
                 (tADDi3 tGPR:$rd, s_cc_out:$s, tGPR:$rn, mod_imm1_7_neg:$imm, pred:$p)>;
def : tInstSubst<"sub${s}${p} $rdn, $imm",
                 (tADDi8 tGPR:$rdn, s_cc_out:$s, mod_imm8_255_neg:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1081-1087
```tablegen
// AND register
let isCommutable = 1 in
def tAND :                      // A8.6.12
  T1sItDPEncode<0b0000, (outs tGPR:$Rdn), (ins tGPR:$Rn, tGPR:$Rm),
                IIC_iBITr,
                "and", "\t$Rdn, $Rm",
                [(set tGPR:$Rdn, (and tGPR:$Rn, tGPR:$Rm))]>, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tAND` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tAND`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1089-1098
```tablegen
// ASR immediate
def tASRri :                    // A8.6.14
  T1sIGenEncodeImm<{0,1,0,?,?}, (outs tGPR:$Rd), (ins tGPR:$Rm, imm_sr:$imm5),
                   IIC_iMOVsi,
                   "asr", "\t$Rd, $Rm, $imm5",
                   [(set tGPR:$Rd, (sra tGPR:$Rm, (i32 imm_sr:$imm5)))]>,
                   Sched<[WriteALU]> {
  bits<5> imm5;
  let Inst{10-6} = imm5;
}
```
- EN: Defines TableGen record `tASRri` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tASRri`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1100-1106
```tablegen
// ASR register
def tASRrr :                    // A8.6.15
  T1sItDPEncode<0b0100, (outs tGPR:$Rdn), (ins tGPR:$Rn, tGPR:$Rm),
                IIC_iMOVsr,
                "asr", "\t$Rdn, $Rm",
                [(set tGPR:$Rdn, (sra tGPR:$Rn, tGPR:$Rm))]>,
                Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tASRrr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tASRrr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1108-1114
```tablegen
// BIC register
def tBIC :                      // A8.6.20
  T1sItDPEncode<0b1110, (outs tGPR:$Rdn), (ins tGPR:$Rn, tGPR:$Rm),
                IIC_iBITr,
                "bic", "\t$Rdn, $Rm",
                [(set tGPR:$Rdn, (and tGPR:$Rn, (not tGPR:$Rm)))]>,
                Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tBIC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tBIC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1116-1124
```tablegen
// CMN register
let isCompare = 1, Defs = [CPSR] in {
def tCMN :                     // A8.6.33
  T1pIDPEncode<0b1011, (outs), (ins tGPR:$Rn, tGPR:$Rm),
               IIC_iCMPr,
               "cmn", "\t$Rn, $Rm",
               [(set CPSR, (ARMcmn tGPR:$Rn, tGPR:$Rm))]>,
               Sched<[WriteCMP]>;
} // isCompare = 1, Defs = [CPSR]
```
- EN: Defines TableGen record `tCMN` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tCMN`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1126-1137
```tablegen
// CMP immediate
let isCompare = 1, Defs = [CPSR], hasSideEffects = 0 in {
def tCMPi8 : T1pI<(outs), (ins tGPR:$Rn, imm0_255:$imm8), IIC_iCMPi,
                  "cmp", "\t$Rn, $imm8",
                  [(set CPSR, (ARMcmp tGPR:$Rn, imm0_255:$imm8))]>,
             T1General<{1,0,1,?,?}>, Sched<[WriteCMP]> {
  // A8.6.35
  bits<3> Rn;
  bits<8> imm8;
  let Inst{10-8} = Rn;
  let Inst{7-0}  = imm8;
}
```
- EN: Defines TableGen record `tCMPi8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tCMPi8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1139-1144
```tablegen
// CMP register
def tCMPr :                     // A8.6.36 T1
  T1pIDPEncode<0b1010, (outs), (ins tGPR:$Rn, tGPR:$Rm),
               IIC_iCMPr,
               "cmp", "\t$Rn, $Rm",
               [(set CPSR, (ARMcmp tGPR:$Rn, tGPR:$Rm))]>, Sched<[WriteCMP]>;
```
- EN: Defines TableGen record `tCMPr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tCMPr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1146-1156
```tablegen
def tCMPhir : T1pI<(outs), (ins GPR:$Rn, GPR:$Rm), IIC_iCMPr,
                   "cmp", "\t$Rn, $Rm", []>,
              T1Special<{0,1,?,?}>, Sched<[WriteCMP]> {
  // A8.6.36 T2
  bits<4> Rm;
  bits<4> Rn;
  let Inst{7}   = Rn{3};
  let Inst{6-3} = Rm;
  let Inst{2-0} = Rn{2-0};
}
} // isCompare = 1, Defs = [CPSR]
```
- EN: Defines TableGen record `tCMPhir` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tCMPhir`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1159-1165
```tablegen
// XOR register
let isCommutable = 1 in
def tEOR :                      // A8.6.45
  T1sItDPEncode<0b0001, (outs tGPR:$Rdn), (ins tGPR:$Rn, tGPR:$Rm),
                IIC_iBITr,
                "eor", "\t$Rdn, $Rm",
                [(set tGPR:$Rdn, (xor tGPR:$Rn, tGPR:$Rm))]>, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tEOR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tEOR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1167-1176
```tablegen
// LSL immediate
def tLSLri :                    // A8.6.88
  T1sIGenEncodeImm<{0,0,0,?,?}, (outs tGPR:$Rd), (ins tGPR:$Rm, imm0_31:$imm5),
                   IIC_iMOVsi,
                   "lsl", "\t$Rd, $Rm, $imm5",
                   [(set tGPR:$Rd, (shl tGPR:$Rm, (i32 imm:$imm5)))]>,
                   Sched<[WriteALU]> {
  bits<5> imm5;
  let Inst{10-6} = imm5;
}
```
- EN: Defines TableGen record `tLSLri` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tLSLri`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1178-1183
```tablegen
// LSL register
def tLSLrr :                    // A8.6.89
  T1sItDPEncode<0b0010, (outs tGPR:$Rdn), (ins tGPR:$Rn, tGPR:$Rm),
                IIC_iMOVsr,
                "lsl", "\t$Rdn, $Rm",
                [(set tGPR:$Rdn, (shl tGPR:$Rn, tGPR:$Rm))]>, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tLSLrr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tLSLrr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1185-1194
```tablegen
// LSR immediate
def tLSRri :                    // A8.6.90
  T1sIGenEncodeImm<{0,0,1,?,?}, (outs tGPR:$Rd), (ins tGPR:$Rm, imm_sr:$imm5),
                   IIC_iMOVsi,
                   "lsr", "\t$Rd, $Rm, $imm5",
                   [(set tGPR:$Rd, (srl tGPR:$Rm, (i32 imm_sr:$imm5)))]>,
                   Sched<[WriteALU]> {
  bits<5> imm5;
  let Inst{10-6} = imm5;
}
```
- EN: Defines TableGen record `tLSRri` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tLSRri`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1196-1201
```tablegen
// LSR register
def tLSRrr :                    // A8.6.91
  T1sItDPEncode<0b0011, (outs tGPR:$Rdn), (ins tGPR:$Rn, tGPR:$Rm),
                IIC_iMOVsr,
                "lsr", "\t$Rdn, $Rm",
                [(set tGPR:$Rdn, (srl tGPR:$Rn, tGPR:$Rm))]>, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tLSRrr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tLSRrr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1203-1219
```tablegen
// Move register
let isMoveImm = 1 in
def tMOVi8 : T1sI<(outs tGPR:$Rd), (ins imm0_255_expr:$imm8), IIC_iMOVi,
                  "mov", "\t$Rd, $imm8",
                  [(set tGPR:$Rd, imm0_255_expr:$imm8)]>,
             T1General<{1,0,0,?,?}>, Sched<[WriteALU]> {
  // A8.6.96
  bits<3> Rd;
  bits<8> imm8;
  let Inst{10-8} = Rd;
  let Inst{7-0}  = imm8;
}
// Because we have an explicit tMOVSr below, we need an alias to handle
// the immediate "movs" form here. Blech.
def : tInstAlias<"movs $Rdn, $imm8",
                 (tMOVi8 tGPR:$Rdn, (s_cc_out CPSR),
                         imm0_255_expr:$imm8, (pred 14, zero_reg))>;
```
- EN: Defines TableGen record `tMOVi8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tMOVi8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1221-1221
```tablegen
// A7-73: MOV(2) - mov setting flag.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1223-1240
```tablegen
let hasSideEffects = 0, isMoveReg = 1 in {
def tMOVr : Thumb1pI<(outs GPR:$Rd), (ins GPR:$Rm), AddrModeNone,
                      2, IIC_iMOVr,
                      "mov", "\t$Rd, $Rm", "", []>,
                  T1Special<{1,0,?,?}>, Sched<[WriteALU]> {
  // A8.6.97
  bits<4> Rd;
  bits<4> Rm;
  let Inst{7}   = Rd{3};
  let Inst{6-3} = Rm;
  let Inst{2-0} = Rd{2-0};
}
let Defs = [CPSR] in
def tMOVSr      : T1I<(outs tGPR:$Rd), (ins tGPR:$Rm), IIC_iMOVr,
                      "movs\t$Rd, $Rm", []>, Encoding16, Sched<[WriteALU]> {
  // A8.6.97
  bits<3> Rd;
  bits<3> Rm;
```
- EN: Defines TableGen record `tMOVr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tMOVr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1241-1245
```tablegen
  let Inst{15-6} = 0b0000000000;
  let Inst{5-3}  = Rm;
  let Inst{2-0}  = Rd;
}
} // hasSideEffects
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1247-1259
```tablegen
// Multiply register
let isCommutable = 1 in
def tMUL :                      // A8.6.105 T1
  Thumb1sI<(outs tGPR:$Rd), (ins tGPR:$Rn, tGPR:$Rm), AddrModeNone, 2,
           IIC_iMUL32, "mul", "\t$Rd, $Rn, $Rm", "$Rm = $Rd",
           [(set tGPR:$Rd, (mul tGPR:$Rn, tGPR:$Rm))]>,
      T1DataProcessing<0b1101>, Sched<[WriteMUL32, ReadMUL, ReadMUL]> {
  bits<3> Rd;
  bits<3> Rn;
  let Inst{5-3} = Rn;
  let Inst{2-0} = Rd;
  let AsmMatchConverter = "cvtThumbMultiply";
}
```
- EN: Defines TableGen record `tMUL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tMUL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1261-1262
```tablegen
def :tInstAlias<"mul${s}${p} $Rdm, $Rn", (tMUL tGPR:$Rdm, s_cc_out:$s, tGPR:$Rn,
                                               pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1264-1268
```tablegen
// Move inverse register
def tMVN :                      // A8.6.107
  T1sIDPEncode<0b1111, (outs tGPR:$Rd), (ins tGPR:$Rn), IIC_iMVNr,
               "mvn", "\t$Rd, $Rn",
               [(set tGPR:$Rd, (not tGPR:$Rn))]>, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tMVN` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tMVN`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1270-1276
```tablegen
// Bitwise or register
let isCommutable = 1 in
def tORR :                      // A8.6.114
  T1sItDPEncode<0b1100, (outs tGPR:$Rdn), (ins tGPR:$Rn, tGPR:$Rm),
                IIC_iBITr,
                "orr", "\t$Rdn, $Rm",
                [(set tGPR:$Rdn, (or tGPR:$Rn, tGPR:$Rm))]>, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tORR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tORR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1278-1284
```tablegen
// Swaps
def tREV :                      // A8.6.134
  T1pIMiscEncode<{1,0,1,0,0,0,?}, (outs tGPR:$Rd), (ins tGPR:$Rm),
                 IIC_iUNAr,
                 "rev", "\t$Rd, $Rm",
                 [(set tGPR:$Rd, (bswap tGPR:$Rm))]>,
                 Requires<[IsThumb, IsThumb1Only, HasV6]>, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tREV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tREV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1286-1291
```tablegen
def tREV16 :                    // A8.6.135
  T1pIMiscEncode<{1,0,1,0,0,1,?}, (outs tGPR:$Rd), (ins tGPR:$Rm),
                 IIC_iUNAr,
                 "rev16", "\t$Rd, $Rm",
             [(set tGPR:$Rd, (rotr (bswap tGPR:$Rm), (i32 16)))]>,
                Requires<[IsThumb, IsThumb1Only, HasV6]>, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tREV16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tREV16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1293-1298
```tablegen
def tREVSH :                    // A8.6.136
  T1pIMiscEncode<{1,0,1,0,1,1,?}, (outs tGPR:$Rd), (ins tGPR:$Rm),
                 IIC_iUNAr,
                 "revsh", "\t$Rd, $Rm",
                 [(set tGPR:$Rd, (sra (bswap tGPR:$Rm), (i32 16)))]>,
                 Requires<[IsThumb, IsThumb1Only, HasV6]>, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tREVSH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tREVSH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1300-1306
```tablegen
// Rotate right register
def tROR :                      // A8.6.139
  T1sItDPEncode<0b0111, (outs tGPR:$Rdn), (ins tGPR:$Rn, tGPR:$Rm),
                IIC_iMOVsr,
                "ror", "\t$Rdn, $Rm",
                [(set tGPR:$Rdn, (rotr tGPR:$Rn, tGPR:$Rm))]>,
                Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tROR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tROR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1308-1313
```tablegen
// Negate register
def tRSB :                      // A8.6.141
  T1sIDPEncode<0b1001, (outs tGPR:$Rd), (ins tGPR:$Rn),
               IIC_iALUi,
               "rsb", "\t$Rd, $Rn, #0",
               [(set tGPR:$Rd, (ineg tGPR:$Rn))]>, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tRSB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tRSB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1315-1322
```tablegen
// Subtract with carry register
let Uses = [CPSR], hasSideEffects = 0 in
def tSBC :                      // A8.6.151
  T1sItDPEncode<0b0110, (outs tGPR:$Rdn), (ins tGPR:$Rn, tGPR:$Rm),
                IIC_iALUr,
                "sbc", "\t$Rdn, $Rm",
                []>,
                Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tSBC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tSBC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1324-1333
```tablegen
// Subtract immediate
def tSUBi3 :                    // A8.6.210 T1
  T1sIGenEncodeImm<0b01111, (outs tGPR:$Rd), (ins tGPR:$Rm, imm0_7:$imm3),
                   IIC_iALUi,
                   "sub", "\t$Rd, $Rm, $imm3",
                   [(set tGPR:$Rd, (add tGPR:$Rm, imm0_7_neg:$imm3))]>,
                   Sched<[WriteALU]> {
  bits<3> imm3;
  let Inst{8-6} = imm3;
}
```
- EN: Defines TableGen record `tSUBi3` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tSUBi3`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1335-1340
```tablegen
def tSUBi8 :                    // A8.6.210 T2
  T1sItGenEncodeImm<{1,1,1,?,?}, (outs tGPR:$Rdn),
                    (ins tGPR:$Rn, imm0_255:$imm8), IIC_iALUi,
                    "sub", "\t$Rdn, $imm8",
                    [(set tGPR:$Rdn, (add tGPR:$Rn, imm8_255_neg:$imm8))]>,
                    Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tSUBi8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tSUBi8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1342-1343
```tablegen
def : tInstSubst<"add${s}${p} $rd, $rn, $imm",
                 (tSUBi3 tGPR:$rd, s_cc_out:$s, tGPR:$rn, mod_imm1_7_neg:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1346-1347
```tablegen
def : tInstSubst<"add${s}${p} $rdn, $imm",
                 (tSUBi8 tGPR:$rdn, s_cc_out:$s, mod_imm8_255_neg:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1350-1356
```tablegen
// Subtract register
def tSUBrr :                    // A8.6.212
  T1sIGenEncode<0b01101, (outs tGPR:$Rd), (ins tGPR:$Rn, tGPR:$Rm),
                IIC_iALUr,
                "sub", "\t$Rd, $Rn, $Rm",
                [(set tGPR:$Rd, (sub tGPR:$Rn, tGPR:$Rm))]>,
                Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tSUBrr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tSUBrr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1358-1359
```tablegen
def : tInstAlias <"sub${s}${p} $Rdn, $Rm",
                 (tSUBrr tGPR:$Rdn,s_cc_out:$s, tGPR:$Rdn, tGPR:$Rm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1361-1373
```tablegen
/// Similar to the above except these set the 's' bit so the
/// instruction modifies the CPSR register.
///
/// These opcodes will be converted to the real non-S opcodes by
/// AdjustInstrPostInstrSelection after giving then an optional CPSR operand.
let hasPostISelHook = 1, Defs = [CPSR] in {
  let Uses = [CPSR] in
  def tSBCS : tPseudoInst<(outs tGPR:$Rdn), (ins tGPR:$Rn, tGPR:$Rm),
                          2, IIC_iALUr,
                          [(set tGPR:$Rdn, CPSR, (ARMsube tGPR:$Rn, tGPR:$Rm,
                                                          CPSR))]>,
              Requires<[IsThumb1Only]>,
              Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tSBCS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tSBCS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1375-1380
```tablegen
  def tSUBSi3 : tPseudoInst<(outs tGPR:$Rd), (ins tGPR:$Rm, imm0_7:$imm3),
                            2, IIC_iALUi,
                            [(set tGPR:$Rd, CPSR, (ARMsubc tGPR:$Rm,
                                                           imm0_7:$imm3))]>,
                Requires<[IsThumb1Only]>,
                Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tSUBSi3` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tSUBSi3`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1382-1387
```tablegen
  def tSUBSi8 : tPseudoInst<(outs tGPR:$Rdn), (ins tGPR:$Rn, imm0_255:$imm8),
                            2, IIC_iALUi,
                            [(set tGPR:$Rdn, CPSR, (ARMsubc tGPR:$Rn,
                                                            imm8_255:$imm8))]>,
                Requires<[IsThumb1Only]>,
                Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tSUBSi8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tSUBSi8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1389-1394
```tablegen
  def tSUBSrr : tPseudoInst<(outs tGPR:$Rd), (ins tGPR:$Rn, tGPR:$Rm),
                            2, IIC_iALUr,
                            [(set tGPR:$Rd, CPSR, (ARMsubc tGPR:$Rn,
                                                           tGPR:$Rm))]>,
                Requires<[IsThumb1Only]>,
                Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tSUBSrr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tSUBSrr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1396-1400
```tablegen
  def tRSBS   : tPseudoInst<(outs tGPR:$Rd), (ins tGPR:$Rn),
                            2, IIC_iALUr,
                            [(set tGPR:$Rd, CPSR, (ARMsubc 0, tGPR:$Rn))]>,
                Requires<[IsThumb1Only]>,
                Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tRSBS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tRSBS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1402-1407
```tablegen
  def tLSLSri : tPseudoInst<(outs tGPR:$Rd), (ins tGPR:$Rn, imm0_31:$imm5),
                            2, IIC_iALUr,
                            [(set tGPR:$Rd, CPSR, (ARMlsls tGPR:$Rn, imm0_31:$imm5))]>,
                Requires<[IsThumb1Only]>,
                Sched<[WriteALU]>;
}
```
- EN: Defines TableGen record `tLSLSri` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tLSLSri`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1409-1416
```tablegen
// Sign-extend byte
def tSXTB :                     // A8.6.222
  T1pIMiscEncode<{0,0,1,0,0,1,?}, (outs tGPR:$Rd), (ins tGPR:$Rm),
                 IIC_iUNAr,
                 "sxtb", "\t$Rd, $Rm",
                 [(set tGPR:$Rd, (sext_inreg tGPR:$Rm, i8))]>,
                 Requires<[IsThumb, IsThumb1Only, HasV6]>,
                 Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tSXTB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tSXTB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1418-1425
```tablegen
// Sign-extend short
def tSXTH :                     // A8.6.224
  T1pIMiscEncode<{0,0,1,0,0,0,?}, (outs tGPR:$Rd), (ins tGPR:$Rm),
                 IIC_iUNAr,
                 "sxth", "\t$Rd, $Rm",
                 [(set tGPR:$Rd, (sext_inreg tGPR:$Rm, i16))]>,
                 Requires<[IsThumb, IsThumb1Only, HasV6]>,
                 Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tSXTH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tSXTH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1427-1433
```tablegen
// Test
let isCompare = 1, isCommutable = 1, Defs = [CPSR] in
def tTST :                      // A8.6.230
  T1pIDPEncode<0b1000, (outs), (ins tGPR:$Rn, tGPR:$Rm), IIC_iTSTr,
               "tst", "\t$Rn, $Rm",
               [(set CPSR, (ARMcmpZ (and_su tGPR:$Rn, tGPR:$Rm), 0))]>,
               Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tTST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tTST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1435-1442
```tablegen
// A8.8.247  UDF - Undefined (Encoding T1)
def tUDF : TI<(outs), (ins imm0_255:$imm8), IIC_Br, "udf\t$imm8",
              [(int_arm_undefined imm0_255:$imm8)]>, Encoding16 {
  bits<8> imm8;
  let Inst{15-12} = 0b1101;
  let Inst{11-8} = 0b1110;
  let Inst{7-0} = imm8;
}
```
- EN: Defines TableGen record `tUDF` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tUDF`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1444-1445
```tablegen
def : Pat<(debugtrap), (tBKPT 0)>, Requires<[IsThumb, HasV5T]>;
def : Pat<(debugtrap), (tUDF 254)>, Requires<[IsThumb, NoV5T]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1447-1452
```tablegen
def t__brkdiv0 : TI<(outs), (ins), IIC_Br, "__brkdiv0",
                    [(int_arm_undefined 249)]>, Encoding16,
    Requires<[IsThumb, IsWindows]> {
  let Inst = 0xdef9;
  let isTerminator = 1;
}
```
- EN: Defines TableGen record `t__brkdiv0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `t__brkdiv0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1454-1461
```tablegen
// Zero-extend byte
def tUXTB :                     // A8.6.262
  T1pIMiscEncode<{0,0,1,0,1,1,?}, (outs tGPR:$Rd), (ins tGPR:$Rm),
                 IIC_iUNAr,
                 "uxtb", "\t$Rd, $Rm",
                 [(set tGPR:$Rd, (and tGPR:$Rm, 0xFF))]>,
                 Requires<[IsThumb, IsThumb1Only, HasV6]>,
                 Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tUXTB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tUXTB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1463-1469
```tablegen
// Zero-extend short
def tUXTH :                     // A8.6.264
  T1pIMiscEncode<{0,0,1,0,1,0,?}, (outs tGPR:$Rd), (ins tGPR:$Rm),
                 IIC_iUNAr,
                 "uxth", "\t$Rd, $Rm",
                 [(set tGPR:$Rd, (and tGPR:$Rm, 0xFFFF))]>,
                 Requires<[IsThumb, IsThumb1Only, HasV6]>, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tUXTH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tUXTH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1471-1476
```tablegen
// Conditional move tMOVCCr - Used to implement the Thumb SELECT_CC operation.
// Expanded after instruction selection into a branch sequence.
let usesCustomInserter = 1 in  // Expanded after instruction selection.
  def tMOVCCr_pseudo :
  PseudoInst<(outs tGPR:$dst), (ins tGPR:$false, tGPR:$true, pred:$p),
             NoItinerary, []>;
```
- EN: Defines TableGen record `tMOVCCr_pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tMOVCCr_pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1478-1479
```tablegen
def : Pat<(ARMcmov tGPR:$false, tGPR:$true, imm:$cc, CPSR),
          (tMOVCCr_pseudo $false, $true, imm:$cc, CPSR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1481-1482
```tablegen
// tLEApcrel - Load a pc-relative address into a register without offending the
// assembler.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1484-1493
```tablegen
def tADR : T1I<(outs tGPR:$Rd), (ins t_adrlabel:$addr, pred:$p),
               IIC_iALUi, "adr{$p}\t$Rd, $addr", []>,
               T1Encoding<{1,0,1,0,0,?}>, Sched<[WriteALU]> {
  bits<3> Rd;
  bits<8> addr;
  let Inst{10-8} = Rd;
  let Inst{7-0} = addr;
  let DecoderMethod = "DecodeThumbAddSpecialReg";
  let hasSideEffects = 0;
}
```
- EN: Defines TableGen record `tADR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tADR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1495-1497
```tablegen
let hasSideEffects = 0, isReMaterializable = 1 in
def tLEApcrel   : tPseudoInst<(outs tGPR:$Rd), (ins i32imm:$label, pred:$p),
                              2, IIC_iALUi, []>, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tLEApcrel` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tLEApcrel`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1499-1502
```tablegen
let hasSideEffects = 1 in
def tLEApcrelJT : tPseudoInst<(outs tGPR:$Rd),
                              (ins i32imm:$label, pred:$p),
                              2, IIC_iALUi, []>, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `tLEApcrelJT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tLEApcrelJT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1504-1510
```tablegen
// Thumb-1 doesn't have the TBB or TBH instructions, but we can synthesize them
// and make use of the same compressed jump table format as Thumb-2.
let Size = 2, isBranch = 1, isTerminator = 1, isBarrier = 1,
    isIndirectBranch = 1, isNotDuplicable = 1 in {
def tTBB_JT : tPseudoInst<(outs),
        (ins tGPRwithpc:$base, tGPR:$index, i32imm:$jt, i32imm:$pclbl), 0,
         IIC_Br, []>, Sched<[WriteBr]>;
```
- EN: Defines TableGen record `tTBB_JT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tTBB_JT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1512-1515
```tablegen
def tTBH_JT : tPseudoInst<(outs),
        (ins tGPRwithpc:$base, tGPR:$index, i32imm:$jt, i32imm:$pclbl), 0,
         IIC_Br, []>,  Sched<[WriteBr]>;
}
```
- EN: Defines TableGen record `tTBH_JT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tTBH_JT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1517-1519
```tablegen
//===----------------------------------------------------------------------===//
// TLS Instructions
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 1521-1528
```tablegen
// __aeabi_read_tp preserves the registers r1-r3.
// This is a pseudo inst so that we can get the encoding right,
// complete with fixup for the aeabi_read_tp function.
let isCall = 1, Defs = [R0, R12, LR, CPSR], Uses = [SP] in
def tTPsoft : tPseudoInst<(outs), (ins), 4, IIC_Br,
                          [(set R0, ARMthread_pointer)]>,
                          Requires<[IsThumb, IsReadTPSoft]>,
                          Sched<[WriteBr]>;
```
- EN: Defines TableGen record `tTPsoft` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tTPsoft`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1530-1532
```tablegen
//===----------------------------------------------------------------------===//
// SJLJ Exception handling intrinsics
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 1534-1550
```tablegen
// eh_sjlj_setjmp() is an instruction sequence to store the return address and
// save #0 in R0 for the non-longjmp case.  Since by its nature we may be coming
// from some other function to get here, and we're using the stack frame for the
// containing function to save/restore registers, we can't keep anything live in
// regs across the eh_sjlj_setjmp(), else it will almost certainly have been
// tromped upon when we get here from a longjmp(). We force everything out of
// registers except for our own input by listing the relevant registers in
// Defs. By doing so, we also cause the prologue/epilogue code to actively
// preserve all of the callee-saved registers, which is exactly what we want.
// $val is a scratch register for our use.
// This gets lowered to an instruction sequence of 12 bytes
let Defs = [ R0,  R1,  R2,  R3,  R4,  R5,  R6,  R7, R12, CPSR ],
    hasSideEffects = 1, isBarrier = 1, isCodeGenOnly = 1, Size = 12,
    usesCustomInserter = 1 in
def tInt_eh_sjlj_setjmp : ThumbXI<(outs),(ins tGPR:$src, tGPR:$val),
                                  AddrModeNone, 0, NoItinerary, "","",
                          [(set R0, (ARMeh_sjlj_setjmp tGPR:$src, tGPR:$val))]>;
```
- EN: Defines TableGen record `tInt_eh_sjlj_setjmp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tInt_eh_sjlj_setjmp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1552-1560
```tablegen
// This gets lowered to an instruction sequence of 10 bytes
// FIXME: Non-IOS version(s)
let isBarrier = 1, hasSideEffects = 1, isTerminator = 1, isCodeGenOnly = 1,
    Size = 10, Defs = [ R7, LR, SP ] in
def tInt_eh_sjlj_longjmp : XI<(outs), (ins tGPR:$src, tGPR:$scratch),
                              AddrModeNone, 0, IndexModeNone,
                              Pseudo, NoItinerary, "", "",
                              [(ARMeh_sjlj_longjmp tGPR:$src, tGPR:$scratch)]>,
                             Requires<[IsThumb,IsNotWindows]>;
```
- EN: Defines TableGen record `tInt_eh_sjlj_longjmp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tInt_eh_sjlj_longjmp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1562-1569
```tablegen
// This gets lowered to an instruction sequence of 12 bytes
// (Windows is Thumb2-only)
let isBarrier = 1, hasSideEffects = 1, isTerminator = 1, isCodeGenOnly = 1,
    Size = 12, Defs = [ R11, LR, SP ] in
def tInt_WIN_eh_sjlj_longjmp
  : XI<(outs), (ins GPR:$src, GPR:$scratch), AddrModeNone, 0, IndexModeNone,
       Pseudo, NoItinerary, "", "", [(ARMeh_sjlj_longjmp GPR:$src, GPR:$scratch)]>,
    Requires<[IsThumb,IsWindows]>;
```
- EN: Defines TableGen record `tInt_WIN_eh_sjlj_longjmp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tInt_WIN_eh_sjlj_longjmp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1571-1573
```tablegen
//===----------------------------------------------------------------------===//
// Non-Instruction Patterns
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 1575-1577
```tablegen
// Comparisons
def : T1Pat<(ARMcmpZ tGPR:$Rn, imm0_255:$imm8),
            (tCMPi8  tGPR:$Rn, imm0_255:$imm8)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1579-1581
```tablegen
// Fold compare-equal of a negated register into CMN register form.
def : T1Pat<(ARMcmpZ tGPR:$Rn, (ineg tGPR:$Rm)),
            (tCMN tGPR:$Rn, tGPR:$Rm)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1583-1584
```tablegen
def : T1Pat<(ARMcmpZ tGPR:$Rn, tGPR:$Rm),
            (tCMPr   tGPR:$Rn, tGPR:$Rm)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1586-1598
```tablegen
// Bswap 16 with load/store
def : T1Pat<(srl (bswap (extloadi16 t_addrmode_is2:$addr)), (i32 16)),
            (tREV16 (tLDRHi t_addrmode_is2:$addr))>;
def : T1Pat<(srl (bswap (extloadi16 t_addrmode_rr:$addr)), (i32 16)),
            (tREV16 (tLDRHr t_addrmode_rr:$addr))>;
def : T1Pat<(srl (bswap top16Zero:$Rn), (i32 16)),
            (tREV16 tGPR:$Rn)>;
def : T1Pat<(truncstorei16 (srl (bswap tGPR:$Rn), (i32 16)),
                           t_addrmode_is2:$addr),
            (tSTRHi(tREV16 tGPR:$Rn), t_addrmode_is2:$addr)>;
def : T1Pat<(truncstorei16 (srl (bswap tGPR:$Rn), (i32 16)),
                           t_addrmode_rr:$addr),
            (tSTRHr (tREV16 tGPR:$Rn), t_addrmode_rr:$addr)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1600-1601
```tablegen
// ConstantPool
def : T1Pat<(ARMWrapper  tconstpool  :$dst), (tLEApcrel tconstpool  :$dst)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1603-1610
```tablegen
// GlobalAddress
def tLDRLIT_ga_pcrel : PseudoInst<(outs tGPR:$dst), (ins i32imm:$addr),
                                  IIC_iLoadiALU,
                                  [(set tGPR:$dst,
                                        (ARMWrapperPIC tglobaladdr:$addr))]>,
                       Requires<[IsThumb, DontUseMovtInPic]> {
  let Size = 8;  // 2 instructions + constant.
}
```
- EN: Defines TableGen record `tLDRLIT_ga_pcrel` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tLDRLIT_ga_pcrel`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1612-1618
```tablegen
def tLDRLIT_ga_abs : PseudoInst<(outs tGPR:$dst), (ins i32imm:$src),
                                IIC_iLoad_i,
                                [(set tGPR:$dst,
                                      (ARMWrapper tglobaladdr:$src))]>,
                     Requires<[IsThumb, DontUseMovt, DontGenExecuteOnly]> {
  let Size = 6;  // 1 instruction + constant.
}
```
- EN: Defines TableGen record `tLDRLIT_ga_abs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tLDRLIT_ga_abs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1620-1627
```tablegen
// 32-bit immediate using mov/add with the 4 :lower0_7: to :upper8_15:
// relocations.
// This is a single pseudo instruction to make it re-materializable.
// FIXME: Remove this when we can do generalized remat.
let Defs = [CPSR], isReMaterializable = 1, isMoveImm = 1, Size = 16, hasNoSchedulingInfo = 1 in
def tMOVi32imm : PseudoInst<(outs rGPR:$dst), (ins i32imm:$src), NoItinerary,
                            [(set rGPR:$dst, (i32 imm:$src))]>,
                            Requires<[IsThumb1Only, GenExecuteOnly, DontUseMovt]>;
```
- EN: Defines TableGen record `tMOVi32imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tMOVi32imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1629-1632
```tablegen
def : ARMPat<(ARMWrapper  tglobaladdr :$dst), (tMOVi32imm tglobaladdr :$dst)>,
            Requires<[GenT1ExecuteOnly]>;
def : ARMPat<(ARMWrapper texternalsym :$dst), (tMOVi32imm texternalsym :$dst)>,
            Requires<[GenT1ExecuteOnly]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1635-1641
```tablegen
// TLS globals
def : Pat<(ARMWrapperPIC tglobaltlsaddr:$addr),
          (tLDRLIT_ga_pcrel tglobaltlsaddr:$addr)>,
      Requires<[IsThumb, DontUseMovtInPic]>;
def : Pat<(ARMWrapper tglobaltlsaddr:$addr),
          (tLDRLIT_ga_abs tglobaltlsaddr:$addr)>,
      Requires<[IsThumb, DontUseMovt]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1644-1646
```tablegen
// JumpTable
def : T1Pat<(ARMWrapperJT tjumptable:$dst),
            (tLEApcrelJT tjumptable:$dst)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1648-1650
```tablegen
// Direct calls
def : T1Pat<(ARMcall texternalsym:$func), (tBL texternalsym:$func)>,
      Requires<[IsThumb]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1652-1656
```tablegen
// zextload i1 -> zextload i8
def : T1Pat<(zextloadi1 t_addrmode_is1:$addr),
            (tLDRBi t_addrmode_is1:$addr)>;
def : T1Pat<(zextloadi1 t_addrmode_rr:$addr),
            (tLDRBr t_addrmode_rr:$addr)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1658-1667
```tablegen
// extload from the stack -> word load from the stack, as it avoids having to
// materialize the base in a separate register. This only works when a word
// load puts the byte/halfword value in the same place in the register that the
// byte/halfword load would, i.e. when little-endian.
def : T1Pat<(extloadi1  t_addrmode_sp:$addr), (tLDRspi t_addrmode_sp:$addr)>,
      Requires<[IsThumb, IsThumb1Only, IsLE]>;
def : T1Pat<(extloadi8  t_addrmode_sp:$addr), (tLDRspi t_addrmode_sp:$addr)>,
      Requires<[IsThumb, IsThumb1Only, IsLE]>;
def : T1Pat<(extloadi16 t_addrmode_sp:$addr), (tLDRspi t_addrmode_sp:$addr)>,
      Requires<[IsThumb, IsThumb1Only, IsLE]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1669-1675
```tablegen
// extload -> zextload
def : T1Pat<(extloadi1  t_addrmode_is1:$addr), (tLDRBi t_addrmode_is1:$addr)>;
def : T1Pat<(extloadi1  t_addrmode_rr:$addr),  (tLDRBr t_addrmode_rr:$addr)>;
def : T1Pat<(extloadi8  t_addrmode_is1:$addr), (tLDRBi t_addrmode_is1:$addr)>;
def : T1Pat<(extloadi8  t_addrmode_rr:$addr),  (tLDRBr t_addrmode_rr:$addr)>;
def : T1Pat<(extloadi16 t_addrmode_is2:$addr), (tLDRHi t_addrmode_is2:$addr)>;
def : T1Pat<(extloadi16 t_addrmode_rr:$addr),  (tLDRHr t_addrmode_rr:$addr)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1677-1677
```tablegen
// post-inc loads and stores
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1679-1687
```tablegen
// post-inc LDR -> LDM r0!, {r1}. The way operands are layed out in LDMs is
// different to how ISel expects them for a post-inc load, so use a pseudo
// and expand it just after ISel.
let usesCustomInserter = 1, mayLoad = 1, hasSideEffects = 0,
    Constraints = "$Rn = $Rn_wb,@earlyclobber $Rn_wb" in
 def tLDR_postidx: tPseudoInst<(outs tGPR:$Rt, tGPR:$Rn_wb),
                               (ins tGPR:$Rn, pred:$p),
                               4, IIC_iStore_ru,
                               []>;
```
- EN: Defines TableGen record `tLDR_postidx:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tLDR_postidx:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1689-1693
```tablegen
// post-inc STR -> STM r0!, {r1}. The layout of this (because it doesn't def
// multiple registers) is the same in ISel as MachineInstr, so there's no need
// for a pseudo.
def : T1Pat<(post_store tGPR:$Rt, tGPR:$Rn, 4),
            (tSTMIA_UPD tGPR:$Rn, tGPR:$Rt)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1695-1705
```tablegen
// If it's impossible to use [r,r] address mode for sextload, select to
// ldsr{b|h} r, 0 instead, in a hope that the mov 0 will be more likely to be
// commoned out than a sxth.
let AddedComplexity = 10 in {
def : T1Pat<(sextloadi8 tGPR:$Rn),
            (tLDRSB tGPR:$Rn, (tMOVi8 0))>,
      Requires<[IsThumb, IsThumb1Only, HasV6]>;
def : T1Pat<(sextloadi16 tGPR:$Rn),
            (tLDRSH tGPR:$Rn, (tMOVi8 0))>,
      Requires<[IsThumb, IsThumb1Only, HasV6]>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1707-1714
```tablegen
def : T1Pat<(sextloadi8 t_addrmode_is1:$addr),
            (tASRri (tLSLri (tLDRBi t_addrmode_is1:$addr), 24), 24)>;
def : T1Pat<(sextloadi8 t_addrmode_rr:$addr),
            (tASRri (tLSLri (tLDRBr t_addrmode_rr:$addr), 24), 24)>;
def : T1Pat<(sextloadi16 t_addrmode_is2:$addr),
            (tASRri (tLSLri (tLDRHi t_addrmode_is2:$addr), 16), 16)>;
def : T1Pat<(sextloadi16 t_addrmode_rr:$addr),
            (tASRri (tLSLri (tLDRHr t_addrmode_rr:$addr), 16), 16)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1716-1733
```tablegen
def : T1Pat<(atomic_load_azext_8 t_addrmode_is1:$src),
             (tLDRBi t_addrmode_is1:$src)>;
def : T1Pat<(atomic_load_azext_8 t_addrmode_rr:$src),
             (tLDRBr t_addrmode_rr:$src)>;
def : T1Pat<(atomic_load_azext_16 t_addrmode_is2:$src),
             (tLDRHi t_addrmode_is2:$src)>;
def : T1Pat<(atomic_load_azext_16 t_addrmode_rr:$src),
             (tLDRHr t_addrmode_rr:$src)>;
def : T1Pat<(atomic_load_nonext_32 t_addrmode_is4:$src),
             (tLDRi t_addrmode_is4:$src)>;
def : T1Pat<(atomic_load_nonext_32 t_addrmode_rr:$src),
             (tLDRr t_addrmode_rr:$src)>;
def : T1Pat<(atomic_store_8 tGPR:$val, t_addrmode_is1:$ptr),
             (tSTRBi tGPR:$val, t_addrmode_is1:$ptr)>;
def : T1Pat<(atomic_store_8 tGPR:$val, t_addrmode_rr:$ptr),
             (tSTRBr tGPR:$val, t_addrmode_rr:$ptr)>;
def : T1Pat<(atomic_store_16 tGPR:$val, t_addrmode_is2:$ptr),
             (tSTRHi tGPR:$val, t_addrmode_is2:$ptr)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1734-1739
```tablegen
def : T1Pat<(atomic_store_16 tGPR:$val, t_addrmode_rr:$ptr),
             (tSTRHr tGPR:$val, t_addrmode_rr:$ptr)>;
def : T1Pat<(atomic_store_32 tGPR:$val, t_addrmode_is4:$ptr),
             (tSTRi tGPR:$val, t_addrmode_is4:$ptr)>;
def : T1Pat<(atomic_store_32 tGPR:$val, t_addrmode_rr:$ptr),
             (tSTRr tGPR:$val, t_addrmode_rr:$ptr)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1741-1741
```tablegen
// Large immediate handling.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1743-1746
```tablegen
// Two piece imms.
def : T1Pat<(i32 thumb_immshifted:$src),
            (tLSLri (tMOVi8 (thumb_immshifted_val imm:$src)),
                    (thumb_immshifted_shamt imm:$src))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1748-1749
```tablegen
def : T1Pat<(i32 imm0_255_comp:$src),
            (tMVN (tMOVi8 (imm_not_XFORM imm:$src)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1751-1753
```tablegen
def : T1Pat<(i32 imm256_510:$src),
            (tADDi8 (tMOVi8 255),
                    (thumb_imm256_510_addend imm:$src))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1755-1763
```tablegen
// Pseudo instruction that combines ldr from constpool and add pc. This should
// be expanded into two instructions late to allow if-conversion and
// scheduling.
let isReMaterializable = 1 in
def tLDRpci_pic : PseudoInst<(outs tGPR:$dst), (ins i32imm:$addr, pclabel:$cp),
                             NoItinerary,
               [(set tGPR:$dst, (ARMpic_add (load (ARMWrapper tconstpool:$addr)),
                                           imm:$cp))]>,
               Requires<[IsThumb, IsThumb1Only]>;
```
- EN: Defines TableGen record `tLDRpci_pic` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tLDRpci_pic`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1765-1771
```tablegen
// Pseudo-instruction for merged POP and return.
// FIXME: remove when we have a way to marking a MI with these properties.
let isReturn = 1, isTerminator = 1, isBarrier = 1, mayLoad = 1,
    hasExtraDefRegAllocReq = 1 in
def tPOP_RET : tPseudoExpand<(outs), (ins pred:$p, reglist:$regs, variable_ops),
                           2, IIC_iPop_Br, [],
                           (tPOP pred:$p, reglist:$regs)>, Sched<[WriteBrL]>;
```
- EN: Defines TableGen record `tPOP_RET` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tPOP_RET`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1773-1778
```tablegen
// Indirect branch using "mov pc, $Rm"
let isBranch = 1, isTerminator = 1, isBarrier = 1, isIndirectBranch = 1 in {
  def tBRIND : tPseudoExpand<(outs), (ins GPR:$Rm, pred:$p),
                  2, IIC_Br, [(brind GPR:$Rm)],
                  (tMOVr PC, GPR:$Rm, pred:$p)>, Sched<[WriteBr]>;
}
```
- EN: Defines TableGen record `tBRIND` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tBRIND`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1781-1784
```tablegen
// In Thumb1, "nop" is encoded as a "mov r8, r8". Technically, the bf00
// encoding is available on ARMv6K, but we don't differentiate that finely.
def : InstAlias<"nop", (tMOVr R8, R8, (pred 14, zero_reg)), 0>,
      Requires<[IsThumb, IsThumb1Only]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1787-1789
```tablegen
// "neg" is and alias for "rsb rd, rn, #0"
def : tInstAlias<"neg${s}${p} $Rd, $Rm",
                 (tRSB tGPR:$Rd, s_cc_out:$s, tGPR:$Rm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1792-1798
```tablegen
// Implied destination operand forms for shifts.
def : tInstAlias<"lsl${s}${p} $Rdm, $imm",
             (tLSLri tGPR:$Rdm, cc_out:$s, tGPR:$Rdm, imm0_31:$imm, pred:$p)>;
def : tInstAlias<"lsr${s}${p} $Rdm, $imm",
             (tLSRri tGPR:$Rdm, cc_out:$s, tGPR:$Rdm, imm_sr:$imm, pred:$p)>;
def : tInstAlias<"asr${s}${p} $Rdm, $imm",
             (tASRri tGPR:$Rdm, cc_out:$s, tGPR:$Rdm, imm_sr:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1800-1804
```tablegen
// Pseudo instruction ldr Rt, =immediate
let hasSideEffects = 0 in
def tLDRConstPool
  : tAsmPseudo<"ldr${p} $Rt, $immediate",
               (ins tGPR:$Rt, const_pool_asm_imm:$immediate, pred:$p)>;
```
- EN: Defines TableGen record `tLDRConstPool` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tLDRConstPool`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1806-1808
```tablegen
//===----------------------------------
// Atomic cmpxchg for -O0
//===----------------------------------
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 1810-1811
```tablegen
// See ARMInstrInfo.td. These two thumb specific pseudos are required to
// restrict the register class for the UXTB/UXTH ops used in the expansion.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1813-1817
```tablegen
let Constraints = "@earlyclobber $Rd,@earlyclobber $temp",
    mayLoad = 1, mayStore = 1 in {
def tCMP_SWAP_8 : PseudoInst<(outs GPR:$Rd, tGPR:$temp),
                             (ins GPR:$addr, tGPR:$desired, GPR:$new),
                             NoItinerary, []>, Sched<[]>;
```
- EN: Defines TableGen record `tCMP_SWAP_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tCMP_SWAP_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1819-1821
```tablegen
def tCMP_SWAP_16 : PseudoInst<(outs GPR:$Rd, tGPR:$temp),
                              (ins GPR:$addr, tGPR:$desired, GPR:$new),
                              NoItinerary, []>, Sched<[]>;
```
- EN: Defines TableGen record `tCMP_SWAP_16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tCMP_SWAP_16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1823-1826
```tablegen
def tCMP_SWAP_32 : PseudoInst<(outs GPR:$Rd, tGPR:$temp),
                              (ins GPR:$addr, GPR:$desired, GPR:$new),
                              NoItinerary, []>, Sched<[]>;
}
```
- EN: Defines TableGen record `tCMP_SWAP_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `tCMP_SWAP_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

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
