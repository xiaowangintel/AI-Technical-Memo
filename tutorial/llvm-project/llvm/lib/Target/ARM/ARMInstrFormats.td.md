# ARMInstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMInstrFormats.td`
- Repository: `llvm-project`
- Purpose (EN): Defines `ARMInstrFormats` in LLVM TableGen DSL for the ARM backend, covering instruction encoding formats and bitfield layout.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMInstrFormats`，涵盖指令编码格式与位域布局。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- ARMInstrFormats.td - ARM Instruction Formats -------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 9-12
```tablegen
//===----------------------------------------------------------------------===//
//
// ARM Instruction Format Definitions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 14-19
```tablegen
// Format specifies the encoding used by the instruction.  This is part of the
// ad-hoc solution used to emit machine instruction encodings by our machine
// code emitter.
class Format<bits<6> val> {
  bits<6> Value = val;
}
```
- EN: Declares reusable TableGen class `Format` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `Format`，通常用于抽象共享字段、谓词或编码结构。

### Lines 21-24
```tablegen
def Pseudo        : Format<0>;
def MulFrm        : Format<1>;
def BrFrm         : Format<2>;
def BrMiscFrm     : Format<3>;
```
- EN: Defines TableGen record `Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 26-27
```tablegen
def DPFrm         : Format<4>;
def DPSoRegRegFrm    : Format<5>;
```
- EN: Defines TableGen record `DPFrm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DPFrm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 29-33
```tablegen
def LdFrm         : Format<6>;
def StFrm         : Format<7>;
def LdMiscFrm     : Format<8>;
def StMiscFrm     : Format<9>;
def LdStMulFrm    : Format<10>;
```
- EN: Defines TableGen record `LdFrm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LdFrm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 35-35
```tablegen
def LdStExFrm     : Format<11>;
```
- EN: Defines TableGen record `LdStExFrm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LdStExFrm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 37-39
```tablegen
def ArithMiscFrm  : Format<12>;
def SatFrm        : Format<13>;
def ExtFrm        : Format<14>;
```
- EN: Defines TableGen record `ArithMiscFrm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ArithMiscFrm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 41-50
```tablegen
def VFPUnaryFrm   : Format<15>;
def VFPBinaryFrm  : Format<16>;
def VFPConv1Frm   : Format<17>;
def VFPConv2Frm   : Format<18>;
def VFPConv3Frm   : Format<19>;
def VFPConv4Frm   : Format<20>;
def VFPConv5Frm   : Format<21>;
def VFPLdStFrm    : Format<22>;
def VFPLdStMulFrm : Format<23>;
def VFPMiscFrm    : Format<24>;
```
- EN: Defines TableGen record `VFPUnaryFrm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VFPUnaryFrm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 52-53
```tablegen
def ThumbFrm      : Format<25>;
def MiscFrm       : Format<26>;
```
- EN: Defines TableGen record `ThumbFrm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ThumbFrm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 55-71
```tablegen
def NGetLnFrm     : Format<27>;
def NSetLnFrm     : Format<28>;
def NDupFrm       : Format<29>;
def NLdStFrm      : Format<30>;
def N1RegModImmFrm: Format<31>;
def N2RegFrm      : Format<32>;
def NVCVTFrm      : Format<33>;
def NVDupLnFrm    : Format<34>;
def N2RegVShLFrm  : Format<35>;
def N2RegVShRFrm  : Format<36>;
def N3RegFrm      : Format<37>;
def N3RegVShFrm   : Format<38>;
def NVExtFrm      : Format<39>;
def NVMulSLFrm    : Format<40>;
def NVTBLFrm      : Format<41>;
def DPSoRegImmFrm  : Format<42>;
def N3RegCplxFrm  : Format<43>;
```
- EN: Defines TableGen record `NGetLnFrm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `NGetLnFrm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 73-73
```tablegen
// Misc flags.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 75-78
```tablegen
// The instruction has an Rn register operand.
// UnaryDP - Indicates this is a unary data processing instruction, i.e.
// it doesn't have a Rn operand.
class UnaryDP    { bit isUnaryDataProc = 1; }
```
- EN: Declares reusable TableGen class `UnaryDP` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `UnaryDP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 80-82
```tablegen
// Xform16Bit - Indicates this Thumb2 instruction may be transformed into
// a 16-bit Thumb instruction if certain conditions are met.
class Xform16Bit { bit canXformTo16Bit = 1; }
```
- EN: Declares reusable TableGen class `Xform16Bit` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `Xform16Bit`，通常用于抽象共享字段、谓词或编码结构。

### Lines 84-86
```tablegen
//===----------------------------------------------------------------------===//
// ARM Instruction flags.  These need to match ARMBaseInstrInfo.h.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 88-105
```tablegen
// FIXME: Once the JIT is MC-ized, these can go away.
// Addressing mode.
class AddrMode<bits<5> val> {
  bits<5> Value = val;
}
def AddrModeNone    : AddrMode<0>;
def AddrMode1       : AddrMode<1>;
def AddrMode2       : AddrMode<2>;
def AddrMode3       : AddrMode<3>;
def AddrMode4       : AddrMode<4>;
def AddrMode5       : AddrMode<5>;
def AddrMode6       : AddrMode<6>;
def AddrModeT1_1    : AddrMode<7>;
def AddrModeT1_2    : AddrMode<8>;
def AddrModeT1_4    : AddrMode<9>;
def AddrModeT1_s    : AddrMode<10>;
def AddrModeT2_i12  : AddrMode<11>;
def AddrModeT2_i8   : AddrMode<12>;
```
- EN: Declares reusable TableGen class `AddrMode` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AddrMode`，通常用于抽象共享字段、谓词或编码结构。

### Lines 106-116
```tablegen
def AddrModeT2_i8pos : AddrMode<13>;
def AddrModeT2_i8neg : AddrMode<14>;
def AddrModeT2_so   : AddrMode<15>;
def AddrModeT2_pc   : AddrMode<16>;
def AddrModeT2_i8s4 : AddrMode<17>;
def AddrMode_i12    : AddrMode<18>;
def AddrMode5FP16   : AddrMode<19>;
def AddrModeT2_ldrex : AddrMode<20>;
def AddrModeT2_i7s4 : AddrMode<21>;
def AddrModeT2_i7s2 : AddrMode<22>;
def AddrModeT2_i7   : AddrMode<23>;
```
- EN: Defines TableGen record `AddrModeT2_i8pos` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AddrModeT2_i8pos`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 118-125
```tablegen
// Load / store index mode.
class IndexMode<bits<2> val> {
  bits<2> Value = val;
}
def IndexModeNone : IndexMode<0>;
def IndexModePre  : IndexMode<1>;
def IndexModePost : IndexMode<2>;
def IndexModeUpd  : IndexMode<3>;
```
- EN: Declares reusable TableGen class `IndexMode` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `IndexMode`，通常用于抽象共享字段、谓词或编码结构。

### Lines 127-136
```tablegen
// Instruction execution domain.
class Domain<bits<4> val> {
  bits<4> Value = val;
}
def GenericDomain : Domain<0>;
def VFPDomain     : Domain<1>; // Instructions in VFP domain only
def NeonDomain    : Domain<2>; // Instructions in Neon domain only
def VFPNeonDomain : Domain<3>; // Instructions in both VFP & Neon domains
def VFPNeonA8Domain : Domain<5>; // Instructions in VFP & Neon under A8
def MVEDomain : Domain<8>; // Instructions in MVE and ARMv8.1m
```
- EN: Declares reusable TableGen class `Domain` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `Domain`，通常用于抽象共享字段、谓词或编码结构。

### Lines 138-140
```tablegen
//===----------------------------------------------------------------------===//
// ARM special operands.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 142-145
```tablegen
// ARM imod and iflag operands, used only by the CPS instruction.
def imod_op : Operand<i32> {
  let PrintMethod = "printCPSIMod";
}
```
- EN: Defines TableGen record `imod_op` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `imod_op`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 147-154
```tablegen
def ProcIFlagsOperand : AsmOperandClass {
  let Name = "ProcIFlags";
  let ParserMethod = "parseProcIFlagsOperand";
}
def iflags_op : Operand<i32> {
  let PrintMethod = "printCPSIFlag";
  let ParserMatchClass = ProcIFlagsOperand;
}
```
- EN: Defines TableGen record `ProcIFlagsOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ProcIFlagsOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 156-168
```tablegen
// ARM Predicate operand. Default to 14 = always (AL). Second part is CC
// register whose default is 0 (no register).
def CondCodeOperand : AsmOperandClass {
  let Name = "CondCode";
  let DefaultMethod = "defaultCondCodeOp";
  let IsOptional = true;
}
def pred : PredicateOperand<OtherVT, (ops i32imm, CCR),
                                     (ops (i32 14), (i32 zero_reg))> {
  let PrintMethod = "printPredicateOperand";
  let ParserMatchClass = CondCodeOperand;
  let DecoderMethod = "DecodePredicateOperand";
}
```
- EN: Defines TableGen record `CondCodeOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CondCodeOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 170-181
```tablegen
// Conditional code result for instructions whose 's' bit is set, e.g. subs.
def CCOutOperand : AsmOperandClass {
  let Name = "CCOut";
  let DefaultMethod = "defaultCCOutOp";
  let IsOptional = true;
}
def cc_out : OptionalDefOperand<OtherVT, (ops CCR), (ops (i32 zero_reg))> {
  let EncoderMethod = "getCCOutOpValue";
  let PrintMethod = "printSBitModifierOperand";
  let ParserMatchClass = CCOutOperand;
  let DecoderMethod = "DecodeCCOutOperand";
}
```
- EN: Defines TableGen record `CCOutOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CCOutOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 183-189
```tablegen
// Same as cc_out except it defaults to setting CPSR.
def s_cc_out : OptionalDefOperand<OtherVT, (ops CCR), (ops (i32 CPSR))> {
  let EncoderMethod = "getCCOutOpValue";
  let PrintMethod = "printSBitModifierOperand";
  let ParserMatchClass = CCOutOperand;
  let DecoderMethod = "DecodeCCOutOperand";
}
```
- EN: Defines TableGen record `s_cc_out` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `s_cc_out`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 191-196
```tablegen
// Transform to generate the inverse of a condition code during ISel
def inv_cond_XFORM : SDNodeXForm<imm, [{
  ARMCC::CondCodes CC = static_cast<ARMCC::CondCodes>(N->getZExtValue());
  return CurDAG->getTargetConstant(ARMCC::getOppositeCondition(CC), SDLoc(N),
                                   MVT::i32);
}]>;
```
- EN: Defines TableGen record `inv_cond_XFORM` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `inv_cond_XFORM`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 198-198
```tablegen
// VPT predicate
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 200-211
```tablegen
def VPTPredNOperand : AsmOperandClass {
  let Name = "VPTPredN";
  let PredicateMethod = "isVPTPred";
  let DefaultMethod = "defaultVPTPredOp";
  let IsOptional = true;
}
def VPTPredROperand : AsmOperandClass {
  let Name = "VPTPredR";
  let PredicateMethod = "isVPTPred";
  let DefaultMethod = "defaultVPTPredOp";
  let IsOptional = true;
}
```
- EN: Defines TableGen record `VPTPredNOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VPTPredNOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 213-230
```tablegen
// Operand classes for the cluster of MC operands describing a
// VPT-predicated MVE instruction.
//
// There are two of these classes. Both of them have the same first
// two options:
//
// $cond (an integer) indicates the instruction's predication status:
//   * ARMVCC::None means it's unpredicated
//   * ARMVCC::Then means it's in a VPT block and appears with the T suffix
//   * ARMVCC::Else means it's in a VPT block and appears with the E suffix.
// During code generation, unpredicated and predicated instructions
// are indicated by setting this parameter to 'None' or to 'Then'; the
// third value 'Else' is only used for assembly and disassembly.
//
// $cond_reg (type VCCR) gives the input predicate register. This is
// always either zero_reg or VPR, but needs to be modelled as an
// explicit operand so that it can be register-allocated and spilled
// when these operands are used in code generation).
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 231-248
```tablegen
//
// For 'vpred_r', there's an extra operand $inactive, which specifies
// the vector register which will supply any lanes of the output
// register that the predication mask prevents from being written by
// this instruction. It's always tied to the actual output register
// (i.e. must be allocated into the same physical reg), but again,
// code generation will need to model it as a separate input value.
//
// 'vpred_n' doesn't have that extra operand: it only has $cond and
// $cond_reg. This variant is used for any instruction that can't, or
// doesn't want to, tie $inactive to the output register. Sometimes
// that's because another input parameter is already tied to it (e.g.
// instructions that both read and write their Qd register even when
// unpredicated, either because they only partially overwrite it like
// a narrowing integer conversion, or simply because the instruction
// encoding doesn't have enough register fields to make the output
// independent of all inputs). It can also be because the instruction
// is defined to set disabled output lanes to zero rather than leaving
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 249-252
```tablegen
// them unchanged (vector loads), or because it doesn't output a
// vector register at all (stores, compares). In any of these
// situations it's unnecessary to have an extra operand tied to the
// output, and inconvenient to leave it there unused.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 254-259
```tablegen
// Base class for both kinds of vpred.
class vpred_ops<dag extra_op, dag extra_mi> : OperandWithDefaultOps<OtherVT,
            !con((ops (i32 0), (i32 zero_reg), (i32 zero_reg)), extra_op)> {
  let PrintMethod = "printVPTPredicateOperand";
  let OperandNamespace = "ARM";
  let MIOperandInfo = !con((ops i32imm:$cond, VCCR:$cond_reg, GPRlr:$tp_reg), extra_mi);
```
- EN: Declares reusable TableGen class `for` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `for`，通常用于抽象共享字段、谓词或编码结构。

### Lines 261-266
```tablegen
  // For convenience, we provide a string value that can be appended
  // to the constraints string. It's empty for vpred_n, and for
  // vpred_r it ties the $inactive operand to the output q-register
  // (which by convention will be called $Qd).
  string vpred_constraint;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 268-273
```tablegen
def vpred_r : vpred_ops<(ops (v4i32 undef_tied_input)), (ops MQPR:$inactive)> {
  let ParserMatchClass = VPTPredROperand;
  let OperandType = "OPERAND_VPRED_R";
  let DecoderMethod = "DecodeVpredROperand";
  let vpred_constraint = ",$Qd = $vp.inactive";
}
```
- EN: Defines TableGen record `vpred_r` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `vpred_r`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 275-280
```tablegen
def vpred_n : vpred_ops<(ops), (ops)> {
  let ParserMatchClass = VPTPredNOperand;
  let OperandType = "OPERAND_VPRED_N";
  let DecoderMethod = "DecodeVpredNOperand";
  let vpred_constraint = "";
}
```
- EN: Defines TableGen record `vpred_n` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `vpred_n`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 282-291
```tablegen
// ARM special operands for disassembly only.
//
def SetEndAsmOperand : ImmAsmOperand<0,1> {
  let Name = "SetEndImm";
  let ParserMethod = "parseSetEndImm";
}
def setend_op : Operand<i32> {
  let PrintMethod = "printSetendOperand";
  let ParserMatchClass = SetEndAsmOperand;
}
```
- EN: Defines TableGen record `SetEndAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SetEndAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 293-301
```tablegen
def MSRMaskOperand : AsmOperandClass {
  let Name = "MSRMask";
  let ParserMethod = "parseMSRMaskOperand";
}
def msr_mask : Operand<i32> {
  let PrintMethod = "printMSRMaskOperand";
  let DecoderMethod = "DecodeMSRMask";
  let ParserMatchClass = MSRMaskOperand;
}
```
- EN: Defines TableGen record `MSRMaskOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MSRMaskOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 303-311
```tablegen
def BankedRegOperand : AsmOperandClass {
  let Name = "BankedReg";
  let ParserMethod = "parseBankedRegOperand";
}
def banked_reg : Operand<i32> {
  let PrintMethod = "printBankedRegOperand";
  let DecoderMethod = "DecodeBankedReg";
  let ParserMatchClass = BankedRegOperand;
}
```
- EN: Defines TableGen record `BankedRegOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BankedRegOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 313-330
```tablegen
// Shift Right Immediate - A shift right immediate is encoded differently from
// other shift immediates. The imm6 field is encoded like so:
//
//    Offset    Encoding
//     8        imm6<5:3> = '001', 8 - <imm> is encoded in imm6<2:0>
//     16       imm6<5:4> = '01', 16 - <imm> is encoded in imm6<3:0>
//     32       imm6<5> = '1', 32 - <imm> is encoded in imm6<4:0>
//     64       64 - <imm> is encoded in imm6<5:0>
def shr_imm8_asm_operand : ImmAsmOperand<1,8> { let Name = "ShrImm8"; }
def shr_imm8  : Operand<i32>, ImmLeaf<i32, [{ return Imm > 0 && Imm <= 8; }]> {
  let EncoderMethod = "getShiftRight8Imm";
  let DecoderMethod = "DecodeShiftRight8Imm";
  let ParserMatchClass = shr_imm8_asm_operand;
}
def shr_imm16_asm_operand : ImmAsmOperand<1,16> { let Name = "ShrImm16"; }
def shr_imm16 : Operand<i32>, ImmLeaf<i32, [{ return Imm > 0 && Imm <= 16; }]> {
  let EncoderMethod = "getShiftRight16Imm";
  let DecoderMethod = "DecodeShiftRight16Imm";
```
- EN: Defines TableGen record `shr_imm8_asm_operand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `shr_imm8_asm_operand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 331-344
```tablegen
  let ParserMatchClass = shr_imm16_asm_operand;
}
def shr_imm32_asm_operand : ImmAsmOperand<1,32> { let Name = "ShrImm32"; }
def shr_imm32 : Operand<i32>, ImmLeaf<i32, [{ return Imm > 0 && Imm <= 32; }]> {
  let EncoderMethod = "getShiftRight32Imm";
  let DecoderMethod = "DecodeShiftRight32Imm";
  let ParserMatchClass = shr_imm32_asm_operand;
}
def shr_imm64_asm_operand : ImmAsmOperand<1,64> { let Name = "ShrImm64"; }
def shr_imm64 : Operand<i32>, ImmLeaf<i32, [{ return Imm > 0 && Imm <= 64; }]> {
  let EncoderMethod = "getShiftRight64Imm";
  let DecoderMethod = "DecodeShiftRight64Imm";
  let ParserMatchClass = shr_imm64_asm_operand;
}
```
- EN: Defines TableGen record `shr_imm32_asm_operand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `shr_imm32_asm_operand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 347-352
```tablegen
// ARM Assembler operand for ldr Rd, =expression which generates an offset
// to a constant pool entry or a MOV depending on the value of expression
def const_pool_asm_operand : AsmOperandClass { let Name = "ConstPoolAsmImm"; }
def const_pool_asm_imm : Operand<i32> {
  let ParserMatchClass = const_pool_asm_operand;
}
```
- EN: Defines TableGen record `const_pool_asm_operand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `const_pool_asm_operand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 355-372
```tablegen
//===----------------------------------------------------------------------===//
// ARM Assembler alias templates.
//
// Note: When EmitPriority == 1, the alias will be used for printing
class ARMInstAlias<string Asm, dag Result, bit EmitPriority = 0>
      : InstAlias<Asm, Result, EmitPriority>, Requires<[IsARM]>;
class ARMInstSubst<string Asm, dag Result, bit EmitPriority = 0>
      : InstAlias<Asm, Result, EmitPriority>,
        Requires<[IsARM,UseNegativeImmediates]>;
class  tInstAlias<string Asm, dag Result, bit EmitPriority = 0>
      : InstAlias<Asm, Result, EmitPriority>, Requires<[IsThumb]>;
class  tInstSubst<string Asm, dag Result, bit EmitPriority = 0>
      : InstAlias<Asm, Result, EmitPriority>,
        Requires<[IsThumb,UseNegativeImmediates]>;
class t2InstAlias<string Asm, dag Result, bit EmitPriority = 0>
      : InstAlias<Asm, Result, EmitPriority>, Requires<[IsThumb2]>;
class t2InstSubst<string Asm, dag Result, bit EmitPriority = 0>
      : InstAlias<Asm, Result, EmitPriority>,
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 373-383
```tablegen
        Requires<[IsThumb2,UseNegativeImmediates]>;
class VFP2InstAlias<string Asm, dag Result, bit EmitPriority = 0>
      : InstAlias<Asm, Result, EmitPriority>, Requires<[HasVFP2]>;
class VFP2DPInstAlias<string Asm, dag Result, bit EmitPriority = 0>
      : InstAlias<Asm, Result, EmitPriority>, Requires<[HasVFP2,HasDPVFP]>;
class VFP3InstAlias<string Asm, dag Result, bit EmitPriority = 0>
      : InstAlias<Asm, Result, EmitPriority>, Requires<[HasVFP3]>;
class NEONInstAlias<string Asm, dag Result, bit EmitPriority = 0>
      : InstAlias<Asm, Result, EmitPriority>, Requires<[HasNEON]>;
class MVEInstAlias<string Asm, dag Result, bit EmitPriority = 1>
      : InstAlias<Asm, Result, EmitPriority>, Requires<[HasMVEInt, IsThumb]>;
```
- EN: Declares reusable TableGen class `VFP2InstAlias` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `VFP2InstAlias`，通常用于抽象共享字段、谓词或编码结构。

### Lines 386-389
```tablegen
class VFP2MnemonicAlias<string src, string dst> : MnemonicAlias<src, dst>,
          Requires<[HasVFP2]>;
class NEONMnemonicAlias<string src, string dst> : MnemonicAlias<src, dst>,
          Requires<[HasNEON]>;
```
- EN: Declares reusable TableGen class `VFP2MnemonicAlias` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `VFP2MnemonicAlias`，通常用于抽象共享字段、谓词或编码结构。

### Lines 391-393
```tablegen
//===----------------------------------------------------------------------===//
// ARM Instruction templates.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 396-399
```tablegen
class InstTemplate<AddrMode am, int sz, IndexMode im,
                   Format f, Domain d, string cstr, InstrItinClass itin>
  : Instruction {
  let Namespace = "ARM";
```
- EN: Declares reusable TableGen class `InstTemplate` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `InstTemplate`，通常用于抽象共享字段、谓词或编码结构。

### Lines 401-414
```tablegen
  AddrMode AM = am;
  int Size = sz;
  IndexMode IM = im;
  bits<2> IndexModeBits = IM.Value;
  Format F = f;
  bits<6> Form = F.Value;
  Domain D = d;
  bit isUnaryDataProc = 0;
  bit canXformTo16Bit = 0;
  // The instruction is a 16-bit flag setting Thumb instruction. Used
  // by the parser and if-converter to determine whether to require the 'S'
  // suffix on the mnemonic (when not in an IT block) or preclude it (when
  // in an IT block).
  bit thumbArithFlagSetting = 0;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 416-420
```tablegen
  bits<2> VecSize = 0;
  bit validForTailPredication = 0;
  bit retainsPreviousHalfElement = 0;
  bit horizontalReduction = 0;
  bit doubleWidthResult = 0;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 422-423
```tablegen
  // If this is a pseudo instruction, mark it isCodeGenOnly.
  let isCodeGenOnly = !eq(!cast<string>(f), "Pseudo");
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 425-437
```tablegen
  // The layout of TSFlags should be kept in sync with ARMBaseInfo.h.
  let TSFlags{4-0}   = AM.Value;
  let TSFlags{6-5}   = IndexModeBits;
  let TSFlags{12-7} = Form;
  let TSFlags{13}    = isUnaryDataProc;
  let TSFlags{14}    = canXformTo16Bit;
  let TSFlags{18-15} = D.Value;
  let TSFlags{19}    = thumbArithFlagSetting;
  let TSFlags{20}    = validForTailPredication;
  let TSFlags{21}    = retainsPreviousHalfElement;
  let TSFlags{22}    = horizontalReduction;
  let TSFlags{23}    = doubleWidthResult;
  let TSFlags{25-24} = VecSize;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 439-441
```tablegen
  let Constraints = cstr;
  let Itinerary = itin;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 443-453
```tablegen
class Encoding {
  field bits<32> Inst;
  // Mask of bits that cause an encoding to be UNPREDICTABLE.
  // If a bit is set, then if the corresponding bit in the
  // target encoding differs from its value in the "Inst" field,
  // the instruction is UNPREDICTABLE (SoftFail in abstract parlance).
  field bits<32> Unpredictable = 0;
  // SoftFail is the generic name for this field, but we alias it so
  // as to make it more obvious what it means in ARM-land.
  field bits<32> SoftFail = Unpredictable;
}
```
- EN: Declares reusable TableGen class `Encoding` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `Encoding`，通常用于抽象共享字段、谓词或编码结构。

### Lines 455-459
```tablegen
class InstARM<AddrMode am, int sz, IndexMode im,
              Format f, Domain d, string cstr, InstrItinClass itin>
  : InstTemplate<am, sz, im, f, d, cstr, itin>, Encoding {
  let DecoderNamespace = "ARM";
}
```
- EN: Declares reusable TableGen class `InstARM` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `InstARM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 461-467
```tablegen
// This Encoding-less class is used by Thumb1 to specify the encoding bits later
// on by adding flavors to specific instructions.
class InstThumb<AddrMode am, int sz, IndexMode im,
                Format f, Domain d, string cstr, InstrItinClass itin>
  : InstTemplate<am, sz, im, f, d, cstr, itin> {
  let DecoderNamespace = "Thumb";
}
```
- EN: Declares reusable TableGen class `is` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `is`，通常用于抽象共享字段、谓词或编码结构。

### Lines 469-482
```tablegen
// Pseudo-instructions for alternate assembly syntax (never used by codegen).
// These are aliases that require C++ handling to convert to the target
// instruction, while InstAliases can be handled directly by tblgen.
class AsmPseudoInst<string asm, dag iops, dag oops = (outs)>
  : InstTemplate<AddrModeNone, 4, IndexModeNone, Pseudo, GenericDomain,
                 "", NoItinerary> {
  let OutOperandList = oops;
  let InOperandList = iops;
  let Pattern = [];
  let isCodeGenOnly = 0; // So we get asm matcher for it.
  let AsmString = asm;
  let isPseudo = 1;
  let hasNoSchedulingInfo = 1;
}
```
- EN: Declares reusable TableGen class `AsmPseudoInst` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AsmPseudoInst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 484-495
```tablegen
class ARMAsmPseudo<string asm, dag iops, dag oops = (outs)>
  : AsmPseudoInst<asm, iops, oops>, Requires<[IsARM]>;
class tAsmPseudo<string asm, dag iops, dag oops = (outs)>
  : AsmPseudoInst<asm, iops, oops>, Requires<[IsThumb]>;
class t2AsmPseudo<string asm, dag iops, dag oops = (outs)>
  : AsmPseudoInst<asm, iops, oops>, Requires<[IsThumb2]>;
class VFP2AsmPseudo<string asm, dag iops, dag oops = (outs)>
  : AsmPseudoInst<asm, iops, oops>, Requires<[HasVFP2]>;
class NEONAsmPseudo<string asm, dag iops, dag oops = (outs)>
  : AsmPseudoInst<asm, iops, oops>, Requires<[HasNEON]>;
class MVEAsmPseudo<string asm, dag iops, dag oops = (outs)>
  : AsmPseudoInst<asm, iops, oops>, Requires<[HasMVEInt]>;
```
- EN: Declares reusable TableGen class `ARMAsmPseudo` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `ARMAsmPseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 497-506
```tablegen
// Pseudo instructions for the code generator.
class PseudoInst<dag oops, dag iops, InstrItinClass itin, list<dag> pattern>
  : InstTemplate<AddrModeNone, 0, IndexModeNone, Pseudo,
                 GenericDomain, "", itin> {
  let OutOperandList = oops;
  let InOperandList = iops;
  let Pattern = pattern;
  let isCodeGenOnly = 1;
  let isPseudo = 1;
}
```
- EN: Declares reusable TableGen class `PseudoInst` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `PseudoInst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 508-514
```tablegen
// PseudoInst that's ARM-mode only.
class ARMPseudoInst<dag oops, dag iops, int sz, InstrItinClass itin,
                    list<dag> pattern>
  : PseudoInst<oops, iops, itin, pattern> {
  let Size = sz;
  list<Predicate> Predicates = [IsARM];
}
```
- EN: Declares reusable TableGen class `ARMPseudoInst` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `ARMPseudoInst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 516-522
```tablegen
// PseudoInst that's Thumb-mode only.
class tPseudoInst<dag oops, dag iops, int sz, InstrItinClass itin,
                    list<dag> pattern>
  : PseudoInst<oops, iops, itin, pattern> {
  let Size = sz;
  list<Predicate> Predicates = [IsThumb];
}
```
- EN: Declares reusable TableGen class `tPseudoInst` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `tPseudoInst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 524-530
```tablegen
// PseudoInst that's in ARMv8-M baseline (Somewhere between Thumb and Thumb2)
class t2basePseudoInst<dag oops, dag iops, int sz, InstrItinClass itin,
                    list<dag> pattern>
  : PseudoInst<oops, iops, itin, pattern> {
  let Size = sz;
  list<Predicate> Predicates = [IsThumb,HasV8MBaseline];
}
```
- EN: Declares reusable TableGen class `t2basePseudoInst` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `t2basePseudoInst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 532-538
```tablegen
// PseudoInst that's Thumb2-mode only.
class t2PseudoInst<dag oops, dag iops, int sz, InstrItinClass itin,
                    list<dag> pattern>
  : PseudoInst<oops, iops, itin, pattern> {
  let Size = sz;
  list<Predicate> Predicates = [IsThumb2];
}
```
- EN: Declares reusable TableGen class `t2PseudoInst` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `t2PseudoInst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 540-544
```tablegen
class ARMPseudoExpand<dag oops, dag iops, int sz,
                      InstrItinClass itin, list<dag> pattern,
                      dag Result>
  : ARMPseudoInst<oops, iops, sz, itin, pattern>,
    PseudoInstExpansion<Result>;
```
- EN: Declares reusable TableGen class `ARMPseudoExpand` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `ARMPseudoExpand`，通常用于抽象共享字段、谓词或编码结构。

### Lines 546-550
```tablegen
class tPseudoExpand<dag oops, dag iops, int sz,
                    InstrItinClass itin, list<dag> pattern,
                    dag Result>
  : tPseudoInst<oops, iops, sz, itin, pattern>,
    PseudoInstExpansion<Result>;
```
- EN: Declares reusable TableGen class `tPseudoExpand` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `tPseudoExpand`，通常用于抽象共享字段、谓词或编码结构。

### Lines 552-556
```tablegen
class t2PseudoExpand<dag oops, dag iops, int sz,
                    InstrItinClass itin, list<dag> pattern,
                    dag Result>
  : t2PseudoInst<oops, iops, sz, itin, pattern>,
    PseudoInstExpansion<Result>;
```
- EN: Declares reusable TableGen class `t2PseudoExpand` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `t2PseudoExpand`，通常用于抽象共享字段、谓词或编码结构。

### Lines 558-571
```tablegen
// Almost all ARM instructions are predicable.
class I<dag oops, dag iops, AddrMode am, int sz,
        IndexMode im, Format f, InstrItinClass itin,
        string opc, string asm, string cstr,
        list<dag> pattern>
  : InstARM<am, sz, im, f, GenericDomain, cstr, itin> {
  bits<4> p;
  let Inst{31-28} = p;
  let OutOperandList = oops;
  let InOperandList = !con(iops, (ins pred:$p));
  let AsmString = !strconcat(opc, "${p}", asm);
  let Pattern = pattern;
  list<Predicate> Predicates = [IsARM];
}
```
- EN: Declares reusable TableGen class `I` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `I`，通常用于抽象共享字段、谓词或编码结构。

### Lines 573-585
```tablegen
// A few are not predicable
class InoP<dag oops, dag iops, AddrMode am, int sz,
           IndexMode im, Format f, InstrItinClass itin,
           string opc, string asm, string cstr,
           list<dag> pattern>
  : InstARM<am, sz, im, f, GenericDomain, cstr, itin> {
  let OutOperandList = oops;
  let InOperandList = iops;
  let AsmString = !strconcat(opc, asm);
  let Pattern = pattern;
  let isPredicable = 0;
  list<Predicate> Predicates = [IsARM];
}
```
- EN: Declares reusable TableGen class `InoP` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `InoP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 587-598
```tablegen
// Same as I except it can optionally modify CPSR. Note it's modeled as an input
// operand since by default it's a zero register. It will become an implicit def
// once it's "flipped".
class sI<dag oops, dag iops, AddrMode am, int sz,
         IndexMode im, Format f, InstrItinClass itin,
         string opc, string asm, string cstr,
         list<dag> pattern>
  : InstARM<am, sz, im, f, GenericDomain, cstr, itin> {
  bits<4> p; // Predicate operand
  bits<1> s; // condition-code set flag ('1' if the insn should set the flags)
  let Inst{31-28} = p;
  let Inst{20} = s;
```
- EN: Declares reusable TableGen class `sI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `sI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 600-605
```tablegen
  let OutOperandList = oops;
  let InOperandList = !con(iops, (ins pred:$p, cc_out:$s));
  let AsmString = !strconcat(opc, "${s}${p}", asm);
  let Pattern = pattern;
  list<Predicate> Predicates = [IsARM];
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 607-617
```tablegen
// Special cases
class XI<dag oops, dag iops, AddrMode am, int sz,
         IndexMode im, Format f, InstrItinClass itin,
         string asm, string cstr, list<dag> pattern>
  : InstARM<am, sz, im, f, GenericDomain, cstr, itin> {
  let OutOperandList = oops;
  let InOperandList = iops;
  let AsmString = asm;
  let Pattern = pattern;
  list<Predicate> Predicates = [IsARM];
}
```
- EN: Declares reusable TableGen class `XI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `XI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 619-636
```tablegen
class AI<dag oops, dag iops, Format f, InstrItinClass itin,
         string opc, string asm, list<dag> pattern>
  : I<oops, iops, AddrModeNone, 4, IndexModeNone, f, itin,
      opc, asm, "", pattern>;
class AsI<dag oops, dag iops, Format f, InstrItinClass itin,
          string opc, string asm, list<dag> pattern>
  : sI<oops, iops, AddrModeNone, 4, IndexModeNone, f, itin,
       opc, asm, "", pattern>;
class AXI<dag oops, dag iops, Format f, InstrItinClass itin,
          string asm, list<dag> pattern>
  : XI<oops, iops, AddrModeNone, 4, IndexModeNone, f, itin,
       asm, "", pattern>;
class AXIM<dag oops, dag iops, AddrMode am, Format f, InstrItinClass itin,
          string asm, list<dag> pattern>
  : XI<oops, iops, am, 4, IndexModeNone, f, itin,
       asm, "", pattern>;
class AInoP<dag oops, dag iops, Format f, InstrItinClass itin,
            string opc, string asm, list<dag> pattern>
```
- EN: Declares reusable TableGen class `AI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 637-638
```tablegen
  : InoP<oops, iops, AddrModeNone, 4, IndexModeNone, f, itin,
         opc, asm, "", pattern>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 640-652
```tablegen
// Ctrl flow instructions
class ABI<bits<4> opcod, dag oops, dag iops, InstrItinClass itin,
          string opc, string asm, list<dag> pattern>
  : I<oops, iops, AddrModeNone, 4, IndexModeNone, BrFrm, itin,
      opc, asm, "", pattern> {
  let Inst{27-24} = opcod;
}
class ABXI<bits<4> opcod, dag oops, dag iops, InstrItinClass itin,
           string asm, list<dag> pattern>
  : XI<oops, iops, AddrModeNone, 4, IndexModeNone, BrFrm, itin,
       asm, "", pattern> {
  let Inst{27-24} = opcod;
}
```
- EN: Declares reusable TableGen class `ABI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `ABI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 654-658
```tablegen
// BR_JT instructions
class JTI<dag oops, dag iops, InstrItinClass itin,
          string asm, list<dag> pattern>
  : XI<oops, iops, AddrModeNone, 0, IndexModeNone, BrMiscFrm, itin,
       asm, "", pattern>;
```
- EN: Declares reusable TableGen class `JTI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `JTI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 660-677
```tablegen
class AIldr_ex_or_acq<bits<2> opcod, bits<2> opcod2, dag oops, dag iops, InstrItinClass itin,
              string opc, string asm, list<dag> pattern>
  : I<oops, iops, AddrModeNone, 4, IndexModeNone, LdStExFrm, itin,
      opc, asm, "", pattern> {
  bits<4> Rt;
  bits<4> addr;
  let Inst{27-23} = 0b00011;
  let Inst{22-21} = opcod;
  let Inst{20}    = 1;
  let Inst{19-16} = addr;
  let Inst{15-12} = Rt;
  let Inst{11-10} = 0b11;
  let Inst{9-8}   = opcod2;
  let Inst{7-0}   = 0b10011111;
}
class AIstr_ex_or_rel<bits<2> opcod, bits<2> opcod2, dag oops, dag iops, InstrItinClass itin,
              string opc, string asm, list<dag> pattern>
  : I<oops, iops, AddrModeNone, 4, IndexModeNone, LdStExFrm, itin,
```
- EN: Declares reusable TableGen class `AIldr_ex_or_acq` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AIldr_ex_or_acq`，通常用于抽象共享字段、谓词或编码结构。

### Lines 678-688
```tablegen
      opc, asm, "", pattern> {
  bits<4> Rt;
  bits<4> addr;
  let Inst{27-23} = 0b00011;
  let Inst{22-21} = opcod;
  let Inst{20}    = 0;
  let Inst{19-16} = addr;
  let Inst{11-10} = 0b11;
  let Inst{9-8}   = opcod2;
  let Inst{7-4}   = 0b1001;
  let Inst{3-0}   = Rt;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 690-695
```tablegen
  let mayStore = 1;
}
// Atomic load/store instructions
class AIldrex<bits<2> opcod, dag oops, dag iops, InstrItinClass itin,
              string opc, string asm, list<dag> pattern>
  : AIldr_ex_or_acq<opcod, 0b11, oops, iops, itin, opc, asm, pattern>;
```
- EN: Declares reusable TableGen class `AIldrex` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AIldrex`，通常用于抽象共享字段、谓词或编码结构。

### Lines 697-702
```tablegen
class AIstrex<bits<2> opcod, dag oops, dag iops, InstrItinClass itin,
              string opc, string asm, list<dag> pattern>
  : AIstr_ex_or_rel<opcod, 0b11, oops, iops, itin, opc, asm, pattern> {
  bits<4> Rd;
  let Inst{15-12} = Rd;
}
```
- EN: Declares reusable TableGen class `AIstrex` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AIstrex`，通常用于抽象共享字段、谓词或编码结构。

### Lines 704-704
```tablegen
// Exclusive load/store instructions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 706-709
```tablegen
class AIldaex<bits<2> opcod, dag oops, dag iops, InstrItinClass itin,
              string opc, string asm, list<dag> pattern>
  : AIldr_ex_or_acq<opcod, 0b10, oops, iops, itin, opc, asm, pattern>,
    Requires<[IsARM, HasAcquireRelease, HasV7Clrex]>;
```
- EN: Declares reusable TableGen class `AIldaex` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AIldaex`，通常用于抽象共享字段、谓词或编码结构。

### Lines 711-717
```tablegen
class AIstlex<bits<2> opcod, dag oops, dag iops, InstrItinClass itin,
              string opc, string asm, list<dag> pattern>
  : AIstr_ex_or_rel<opcod, 0b10, oops, iops, itin, opc, asm, pattern>,
    Requires<[IsARM, HasAcquireRelease, HasV7Clrex]> {
  bits<4> Rd;
  let Inst{15-12} = Rd;
}
```
- EN: Declares reusable TableGen class `AIstlex` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AIstlex`，通常用于抽象共享字段、谓词或编码结构。

### Lines 719-730
```tablegen
class AIswp<bit b, dag oops, dag iops, string opc, list<dag> pattern>
  : AI<oops, iops, MiscFrm, NoItinerary, opc, "\t$Rt, $Rt2, $addr", pattern> {
  bits<4> Rt;
  bits<4> Rt2;
  bits<4> addr;
  let Inst{27-23} = 0b00010;
  let Inst{22} = b;
  let Inst{21-20} = 0b00;
  let Inst{19-16} = addr;
  let Inst{15-12} = Rt;
  let Inst{11-4} = 0b00001001;
  let Inst{3-0} = Rt2;
```
- EN: Declares reusable TableGen class `AIswp` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AIswp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 732-739
```tablegen
  let Unpredictable{11-8} = 0b1111;
  let DecoderMethod = "DecodeSwap";
}
// Acquire/Release load/store instructions
class AIldracq<bits<2> opcod, dag oops, dag iops, InstrItinClass itin,
              string opc, string asm, list<dag> pattern>
  : AIldr_ex_or_acq<opcod, 0b00, oops, iops, itin, opc, asm, pattern>,
    Requires<[IsARM, HasAcquireRelease]>;
```
- EN: Declares reusable TableGen class `AIldracq` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AIldracq`，通常用于抽象共享字段、谓词或编码结构。

### Lines 741-746
```tablegen
class AIstrrel<bits<2> opcod, dag oops, dag iops, InstrItinClass itin,
              string opc, string asm, list<dag> pattern>
  : AIstr_ex_or_rel<opcod, 0b00, oops, iops, itin, opc, asm, pattern>,
    Requires<[IsARM, HasAcquireRelease]> {
  let Inst{15-12}   = 0b1111;
}
```
- EN: Declares reusable TableGen class `AIstrrel` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AIstrrel`，通常用于抽象共享字段、谓词或编码结构。

### Lines 748-765
```tablegen
// addrmode1 instructions
class AI1<bits<4> opcod, dag oops, dag iops, Format f, InstrItinClass itin,
          string opc, string asm, list<dag> pattern>
  : I<oops, iops, AddrMode1, 4, IndexModeNone, f, itin,
      opc, asm, "", pattern> {
  let Inst{24-21} = opcod;
  let Inst{27-26} = 0b00;
}
class AsI1<bits<4> opcod, dag oops, dag iops, Format f, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : sI<oops, iops, AddrMode1, 4, IndexModeNone, f, itin,
       opc, asm, "", pattern> {
  let Inst{24-21} = opcod;
  let Inst{27-26} = 0b00;
}
class AXI1<bits<4> opcod, dag oops, dag iops, Format f, InstrItinClass itin,
           string asm, list<dag> pattern>
  : XI<oops, iops, AddrMode1, 4, IndexModeNone, f, itin,
```
- EN: Declares reusable TableGen class `AI1` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AI1`，通常用于抽象共享字段、谓词或编码结构。

### Lines 766-769
```tablegen
       asm, "", pattern> {
  let Inst{24-21} = opcod;
  let Inst{27-26} = 0b00;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 771-771
```tablegen
// loads
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 773-790
```tablegen
// LDR/LDRB/STR/STRB/...
class AI2ldst<bits<3> op, bit isLd, bit isByte, dag oops, dag iops, AddrMode am,
             Format f, InstrItinClass itin, string opc, string asm,
             list<dag> pattern>
  : I<oops, iops, am, 4, IndexModeNone, f, itin, opc, asm,
      "", pattern> {
  let Inst{27-25} = op;
  let Inst{24} = 1;  // 24 == P
  // 23 == U
  let Inst{22} = isByte;
  let Inst{21} = 0;  // 21 == W
  let Inst{20} = isLd;
}
// Indexed load/stores
class AI2ldstidx<bit isLd, bit isByte, bit isPre, dag oops, dag iops,
                IndexMode im, Format f, InstrItinClass itin, string opc,
                string asm, string cstr, list<dag> pattern>
  : I<oops, iops, AddrMode2, 4, im, f, itin,
```
- EN: Declares reusable TableGen class `AI2ldst` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AI2ldst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 791-808
```tablegen
      opc, asm, cstr, pattern> {
  bits<4> Rt;
  let Inst{27-26} = 0b01;
  let Inst{24}    = isPre; // P bit
  let Inst{22}    = isByte; // B bit
  let Inst{21}    = isPre; // W bit
  let Inst{20}    = isLd; // L bit
  let Inst{15-12} = Rt;
}
class AI2stridx_reg<bit isByte, bit isPre, dag oops, dag iops,
                IndexMode im, Format f, InstrItinClass itin, string opc,
                string asm, string cstr, list<dag> pattern>
  : AI2ldstidx<0, isByte, isPre, oops, iops, im, f, itin, opc, asm, cstr,
               pattern> {
  // AM2 store w/ two operands: (GPR, am2offset)
  // {12}     isAdd
  // {11-0}   imm12/Rm
  bits<14> offset;
```
- EN: Declares reusable TableGen class `AI2stridx_reg` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AI2stridx_reg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 809-816
```tablegen
  bits<4> Rn;
  let Inst{25} = 1;
  let Inst{23} = offset{12};
  let Inst{19-16} = Rn;
  let Inst{11-5} = offset{11-5};
  let Inst{4} = 0;
  let Inst{3-0} = offset{3-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 818-832
```tablegen
class AI2stridx_imm<bit isByte, bit isPre, dag oops, dag iops,
                IndexMode im, Format f, InstrItinClass itin, string opc,
                string asm, string cstr, list<dag> pattern>
  : AI2ldstidx<0, isByte, isPre, oops, iops, im, f, itin, opc, asm, cstr,
               pattern> {
  // AM2 store w/ two operands: (GPR, am2offset)
  // {12}     isAdd
  // {11-0}   imm12/Rm
  bits<14> offset;
  bits<4> Rn;
  let Inst{25} = 0;
  let Inst{23} = offset{12};
  let Inst{19-16} = Rn;
  let Inst{11-0} = offset{11-0};
}
```
- EN: Declares reusable TableGen class `AI2stridx_imm` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AI2stridx_imm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 835-852
```tablegen
// FIXME: Merge with the above class when addrmode2 gets used for STR, STRB
// but for now use this class for STRT and STRBT.
class AI2stridxT<bit isByte, bit isPre, dag oops, dag iops,
                IndexMode im, Format f, InstrItinClass itin, string opc,
                string asm, string cstr, list<dag> pattern>
  : AI2ldstidx<0, isByte, isPre, oops, iops, im, f, itin, opc, asm, cstr,
               pattern> {
  // AM2 store w/ two operands: (GPR, am2offset)
  // {17-14}  Rn
  // {13}     1 == Rm, 0 == imm12
  // {12}     isAdd
  // {11-0}   imm12/Rm
  bits<18> addr;
  let Inst{25} = addr{13};
  let Inst{23} = addr{12};
  let Inst{19-16} = addr{17-14};
  let Inst{11-0} = addr{11-0};
}
```
- EN: Declares reusable TableGen class `when` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `when`，通常用于抽象共享字段、谓词或编码结构。

### Lines 854-871
```tablegen
// addrmode3 instructions
class AI3ld<bits<4> op, bit op20, dag oops, dag iops, Format f,
            InstrItinClass itin, string opc, string asm, list<dag> pattern>
  : I<oops, iops, AddrMode3, 4, IndexModeNone, f, itin,
      opc, asm, "", pattern> {
  bits<14> addr;
  bits<4> Rt;
  let Inst{27-25} = 0b000;
  let Inst{24}    = 1;            // P bit
  let Inst{23}    = addr{8};      // U bit
  let Inst{22}    = addr{13};     // 1 == imm8, 0 == Rm
  let Inst{21}    = 0;            // W bit
  let Inst{20}    = op20;         // L bit
  let Inst{19-16} = addr{12-9};   // Rn
  let Inst{15-12} = Rt;           // Rt
  let Inst{11-8}  = addr{7-4};    // imm7_4/zero
  let Inst{7-4}   = op;
  let Inst{3-0}   = addr{3-0};    // imm3_0/Rm
```
- EN: Declares reusable TableGen class `AI3ld` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AI3ld`，通常用于抽象共享字段、谓词或编码结构。

### Lines 873-874
```tablegen
  let DecoderMethod = "DecodeAddrMode3Instruction";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 876-888
```tablegen
class AI3ldstidx<bits<4> op, bit op20, bit isPre, dag oops, dag iops,
                IndexMode im, Format f, InstrItinClass itin, string opc,
                string asm, string cstr, list<dag> pattern>
  : I<oops, iops, AddrMode3, 4, im, f, itin,
      opc, asm, cstr, pattern> {
  bits<4> Rt;
  let Inst{27-25} = 0b000;
  let Inst{24}    = isPre;        // P bit
  let Inst{21}    = isPre;        // W bit
  let Inst{20}    = op20;         // L bit
  let Inst{15-12} = Rt;           // Rt
  let Inst{7-4}   = op;
}
```
- EN: Declares reusable TableGen class `AI3ldstidx` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AI3ldstidx`，通常用于抽象共享字段、谓词或编码结构。

### Lines 890-907
```tablegen
// FIXME: Merge with the above class when addrmode2 gets used for LDR, LDRB
// but for now use this class for LDRSBT, LDRHT, LDSHT.
class AI3ldstidxT<bits<4> op, bit isLoad, dag oops, dag iops,
                  IndexMode im, Format f, InstrItinClass itin, string opc,
                  string asm, string cstr, list<dag> pattern>
  : I<oops, iops, AddrMode3, 4, im, f, itin, opc, asm, cstr, pattern> {
  // {13}     1 == imm8, 0 == Rm
  // {12-9}   Rn
  // {8}      isAdd
  // {7-4}    imm7_4/zero
  // {3-0}    imm3_0/Rm
  bits<4> addr;
  bits<4> Rt;
  let Inst{27-25} = 0b000;
  let Inst{24}    = 0;            // P bit
  let Inst{21}    = 1;
  let Inst{20}    = isLoad;       // L bit
  let Inst{19-16} = addr;         // Rn
```
- EN: Declares reusable TableGen class `when` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `when`，通常用于抽象共享字段、谓词或编码结构。

### Lines 908-910
```tablegen
  let Inst{15-12} = Rt;           // Rt
  let Inst{7-4}   = op;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 912-929
```tablegen
// stores
class AI3str<bits<4> op, dag oops, dag iops, Format f, InstrItinClass itin,
             string opc, string asm, list<dag> pattern>
  : I<oops, iops, AddrMode3, 4, IndexModeNone, f, itin,
      opc, asm, "", pattern> {
  bits<14> addr;
  bits<4> Rt;
  let Inst{27-25} = 0b000;
  let Inst{24}    = 1;            // P bit
  let Inst{23}    = addr{8};      // U bit
  let Inst{22}    = addr{13};     // 1 == imm8, 0 == Rm
  let Inst{21}    = 0;            // W bit
  let Inst{20}    = 0;            // L bit
  let Inst{19-16} = addr{12-9};   // Rn
  let Inst{15-12} = Rt;           // Rt
  let Inst{11-8}  = addr{7-4};    // imm7_4/zero
  let Inst{7-4}   = op;
  let Inst{3-0}   = addr{3-0};    // imm3_0/Rm
```
- EN: Declares reusable TableGen class `AI3str` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AI3str`，通常用于抽象共享字段、谓词或编码结构。

### Lines 930-931
```tablegen
  let DecoderMethod = "DecodeAddrMode3Instruction";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 933-945
```tablegen
// addrmode4 instructions
class AXI4<dag oops, dag iops, IndexMode im, Format f, InstrItinClass itin,
           string asm, string cstr, list<dag> pattern>
  : XI<oops, iops, AddrMode4, 4, im, f, itin, asm, cstr, pattern> {
  bits<4>  p;
  bits<16> regs;
  bits<4>  Rn;
  let Inst{31-28} = p;
  let Inst{27-25} = 0b100;
  let Inst{22}    = 0; // S bit
  let Inst{19-16} = Rn;
  let Inst{15-0}  = regs;
}
```
- EN: Declares reusable TableGen class `AXI4` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AXI4`，通常用于抽象共享字段、谓词或编码结构。

### Lines 947-962
```tablegen
// Unsigned multiply, multiply-accumulate instructions.
class AMul1I<bits<7> opcod, dag oops, dag iops, InstrItinClass itin,
             string opc, string asm, list<dag> pattern>
  : I<oops, iops, AddrModeNone, 4, IndexModeNone, MulFrm, itin,
      opc, asm, "", pattern> {
  let Inst{7-4}   = 0b1001;
  let Inst{20}    = 0; // S bit
  let Inst{27-21} = opcod;
}
class AsMul1I<bits<7> opcod, dag oops, dag iops, InstrItinClass itin,
              string opc, string asm, list<dag> pattern>
  : sI<oops, iops, AddrModeNone, 4, IndexModeNone, MulFrm, itin,
       opc, asm, "", pattern> {
  let Inst{7-4}   = 0b1001;
  let Inst{27-21} = opcod;
}
```
- EN: Declares reusable TableGen class `AMul1I` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AMul1I`，通常用于抽象共享字段、谓词或编码结构。

### Lines 964-981
```tablegen
// Most significant word multiply
class AMul2I<bits<7> opcod, bits<4> opc7_4, dag oops, dag iops,
             InstrItinClass itin, string opc, string asm, list<dag> pattern>
  : I<oops, iops, AddrModeNone, 4, IndexModeNone, MulFrm, itin,
      opc, asm, "", pattern> {
  bits<4> Rd;
  bits<4> Rn;
  bits<4> Rm;
  let Inst{7-4}   = opc7_4;
  let Inst{20}    = 1;
  let Inst{27-21} = opcod;
  let Inst{19-16} = Rd;
  let Inst{11-8}  = Rm;
  let Inst{3-0}   = Rn;
}
// MSW multiple w/ Ra operand
class AMul2Ia<bits<7> opcod, bits<4> opc7_4, dag oops, dag iops,
              InstrItinClass itin, string opc, string asm, list<dag> pattern>
```
- EN: Declares reusable TableGen class `AMul2I` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AMul2I`，通常用于抽象共享字段、谓词或编码结构。

### Lines 982-986
```tablegen
  : AMul2I<opcod, opc7_4, oops, iops, itin, opc, asm, pattern> {
  bits<4> Ra;
  let Inst{15-12} = Ra;
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 988-1005
```tablegen
// SMUL<x><y> / SMULW<y> / SMLA<x><y> / SMLAW<x><y>
class AMulxyIbase<bits<7> opcod, bits<2> bit6_5, dag oops, dag iops,
              InstrItinClass itin, string opc, string asm, list<dag> pattern>
  : I<oops, iops, AddrModeNone, 4, IndexModeNone, MulFrm, itin,
      opc, asm, "", pattern> {
  bits<4> Rn;
  bits<4> Rm;
  let Inst{4}     = 0;
  let Inst{7}     = 1;
  let Inst{20}    = 0;
  let Inst{27-21} = opcod;
  let Inst{6-5}   = bit6_5;
  let Inst{11-8}  = Rm;
  let Inst{3-0}   = Rn;
}
class AMulxyI<bits<7> opcod, bits<2> bit6_5, dag oops, dag iops,
              InstrItinClass itin, string opc, string asm, list<dag> pattern>
  : AMulxyIbase<opcod, bit6_5, oops, iops, itin, opc, asm, pattern> {
```
- EN: Declares reusable TableGen class `AMulxyIbase` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AMulxyIbase`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1006-1008
```tablegen
  bits<4> Rd;
  let Inst{19-16} = Rd;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1010-1025
```tablegen
// AMulxyI with Ra operand
class AMulxyIa<bits<7> opcod, bits<2> bit6_5, dag oops, dag iops,
              InstrItinClass itin, string opc, string asm, list<dag> pattern>
  : AMulxyI<opcod, bit6_5, oops, iops, itin, opc, asm, pattern> {
  bits<4> Ra;
  let Inst{15-12} = Ra;
}
// SMLAL*
class AMulxyI64<bits<7> opcod, bits<2> bit6_5, dag oops, dag iops,
              InstrItinClass itin, string opc, string asm, list<dag> pattern>
  : AMulxyIbase<opcod, bit6_5, oops, iops, itin, opc, asm, pattern> {
  bits<4> RdLo;
  bits<4> RdHi;
  let Inst{19-16} = RdHi;
  let Inst{15-12} = RdLo;
}
```
- EN: Declares reusable TableGen class `AMulxyIa` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AMulxyIa`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1027-1039
```tablegen
// Extend instructions.
class AExtI<bits<8> opcod, dag oops, dag iops, InstrItinClass itin,
            string opc, string asm, list<dag> pattern>
  : I<oops, iops, AddrModeNone, 4, IndexModeNone, ExtFrm, itin,
      opc, asm, "", pattern> {
  // All AExtI instructions have Rd and Rm register operands.
  bits<4> Rd;
  bits<4> Rm;
  let Inst{15-12} = Rd;
  let Inst{3-0}   = Rm;
  let Inst{7-4}   = 0b0111;
  let Inst{9-8}   = 0b00;
  let Inst{27-20} = opcod;
```
- EN: Declares reusable TableGen class `AExtI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AExtI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1041-1042
```tablegen
  let Unpredictable{9-8} = 0b11;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1044-1057
```tablegen
// Misc Arithmetic instructions.
class AMiscA1I<bits<8> opcod, bits<4> opc7_4, dag oops, dag iops,
               InstrItinClass itin, string opc, string asm, list<dag> pattern>
  : I<oops, iops, AddrModeNone, 4, IndexModeNone, ArithMiscFrm, itin,
      opc, asm, "", pattern> {
  bits<4> Rd;
  bits<4> Rm;
  let Inst{27-20} = opcod;
  let Inst{19-16} = 0b1111;
  let Inst{15-12} = Rd;
  let Inst{11-8}  = 0b1111;
  let Inst{7-4}   = opc7_4;
  let Inst{3-0}   = Rm;
}
```
- EN: Declares reusable TableGen class `AMiscA1I` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AMiscA1I`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1059-1074
```tablegen
// Division instructions.
class ADivA1I<bits<3> opcod, dag oops, dag iops,
              InstrItinClass itin, string opc, string asm, list<dag> pattern>
  : I<oops, iops, AddrModeNone, 4, IndexModeNone, ArithMiscFrm, itin,
      opc, asm, "", pattern> {
  bits<4> Rd;
  bits<4> Rn;
  bits<4> Rm;
  let Inst{27-23} = 0b01110;
  let Inst{22-20} = opcod;
  let Inst{19-16} = Rd;
  let Inst{15-12} = 0b1111;
  let Inst{11-8}  = Rm;
  let Inst{7-4}   = 0b0001;
  let Inst{3-0}   = Rn;
}
```
- EN: Declares reusable TableGen class `ADivA1I` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `ADivA1I`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1076-1092
```tablegen
// PKH instructions
def PKHLSLAsmOperand : ImmAsmOperand<0,31> {
  let Name = "PKHLSLImm";
  let ParserMethod = "parsePKHLSLImm";
}
def pkh_lsl_amt: Operand<i32>, ImmLeaf<i32, [{ return Imm >= 0 && Imm < 32; }]>{
  let PrintMethod = "printPKHLSLShiftImm";
  let ParserMatchClass = PKHLSLAsmOperand;
}
def PKHASRAsmOperand : AsmOperandClass {
  let Name = "PKHASRImm";
  let ParserMethod = "parsePKHASRImm";
}
def pkh_asr_amt: Operand<i32>, ImmLeaf<i32, [{ return Imm > 0 && Imm <= 32; }]>{
  let PrintMethod = "printPKHASRShiftImm";
  let ParserMatchClass = PKHASRAsmOperand;
}
```
- EN: Defines TableGen record `PKHLSLAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PKHLSLAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1094-1109
```tablegen
class APKHI<bits<8> opcod, bit tb, dag oops, dag iops, InstrItinClass itin,
            string opc, string asm, list<dag> pattern>
  : I<oops, iops, AddrModeNone, 4, IndexModeNone, ArithMiscFrm, itin,
      opc, asm, "", pattern> {
  bits<4> Rd;
  bits<4> Rn;
  bits<4> Rm;
  bits<5> sh;
  let Inst{27-20} = opcod;
  let Inst{19-16} = Rn;
  let Inst{15-12} = Rd;
  let Inst{11-7}  = sh;
  let Inst{6}     = tb;
  let Inst{5-4}   = 0b01;
  let Inst{3-0}   = Rm;
}
```
- EN: Declares reusable TableGen class `APKHI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `APKHI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1111-1111
```tablegen
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 1113-1130
```tablegen
// ARMPat - Same as Pat<>, but requires that the compiler be in ARM mode.
class ARMPat<dag pattern, dag result> : Pat<pattern, result> {
  list<Predicate> Predicates = [IsARM];
}
class ARMV5TPat<dag pattern, dag result> : Pat<pattern, result> {
  list<Predicate> Predicates = [IsARM, HasV5T];
}
class ARMV5TEPat<dag pattern, dag result> : Pat<pattern, result> {
  list<Predicate> Predicates = [IsARM, HasV5TE];
}
// ARMV5MOPat - Same as ARMV5TEPat with UseMulOps.
class ARMV5MOPat<dag pattern, dag result> : Pat<pattern, result> {
  list<Predicate> Predicates = [IsARM, HasV5TE, UseMulOps];
}
class ARMV6Pat<dag pattern, dag result> : Pat<pattern, result> {
  list<Predicate> Predicates = [IsARM, HasV6];
}
class ARMV6T2Pat<dag pattern, dag result> : Pat<pattern, result> {
```
- EN: Declares reusable TableGen class `ARMPat` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `ARMPat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1131-1148
```tablegen
  list<Predicate> Predicates = [IsARM, HasV6T2];
}
class VFPPat<dag pattern, dag result> : Pat<pattern, result> {
  list<Predicate> Predicates = [HasVFP2];
}
class VFPNoNEONPat<dag pattern, dag result> : Pat<pattern, result> {
  list<Predicate> Predicates = [HasVFP2, DontUseNEONForFP];
}
class Thumb2DSPPat<dag pattern, dag result> : Pat<pattern, result> {
  list<Predicate> Predicates = [IsThumb2, HasDSP];
}
class Thumb2DSPMulPat<dag pattern, dag result> : Pat<pattern, result> {
  list<Predicate> Predicates = [IsThumb2, UseMulOps, HasDSP];
}
class FPRegs16Pat<dag pattern, dag result> : Pat<pattern, result> {
  list<Predicate> Predicates = [HasFPRegs16];
}
class FP16Pat<dag pattern, dag result> : Pat<pattern, result> {
```
- EN: Declares reusable TableGen class `VFPPat` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `VFPPat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1149-1156
```tablegen
  list<Predicate> Predicates = [HasFP16];
}
class FullFP16Pat<dag pattern, dag result> : Pat<pattern, result> {
  list<Predicate> Predicates = [HasFullFP16];
}
//===----------------------------------------------------------------------===//
// Thumb Instruction Format Definitions.
//
```
- EN: Declares reusable TableGen class `FullFP16Pat` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `FullFP16Pat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1158-1166
```tablegen
class ThumbI<dag oops, dag iops, AddrMode am, int sz,
             InstrItinClass itin, string asm, string cstr, list<dag> pattern>
  : InstThumb<am, sz, IndexModeNone, ThumbFrm, GenericDomain, cstr, itin> {
  let OutOperandList = oops;
  let InOperandList = iops;
  let AsmString = asm;
  let Pattern = pattern;
  list<Predicate> Predicates = [IsThumb];
}
```
- EN: Declares reusable TableGen class `ThumbI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `ThumbI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1168-1170
```tablegen
// TI - Thumb instruction.
class TI<dag oops, dag iops, InstrItinClass itin, string asm, list<dag> pattern>
  : ThumbI<oops, iops, AddrModeNone, 2, itin, asm, "", pattern>;
```
- EN: Declares reusable TableGen class `TI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `TI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1172-1176
```tablegen
// Two-address instructions
class TIt<dag oops, dag iops, InstrItinClass itin, string asm,
          list<dag> pattern>
  : ThumbI<oops, iops, AddrModeNone, 2, itin, asm, "$lhs = $dst",
           pattern>;
```
- EN: Declares reusable TableGen class `TIt` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `TIt`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1178-1187
```tablegen
// tBL, tBX 32-bit instructions
class TIx2<bits<5> opcod1, bits<2> opcod2, bit opcod3,
           dag oops, dag iops, InstrItinClass itin, string asm,
           list<dag> pattern>
    : ThumbI<oops, iops, AddrModeNone, 4, itin, asm, "", pattern>,
      Encoding {
  let Inst{31-27} = opcod1;
  let Inst{15-14} = opcod2;
  let Inst{12}    = opcod3;
}
```
- EN: Declares reusable TableGen class `TIx2` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `TIx2`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1189-1192
```tablegen
// BR_JT instructions
class TJTI<dag oops, dag iops, InstrItinClass itin, string asm,
           list<dag> pattern>
  : ThumbI<oops, iops, AddrModeNone, 0, itin, asm, "", pattern>;
```
- EN: Declares reusable TableGen class `TJTI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `TJTI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1194-1203
```tablegen
// Thumb1 only
class Thumb1I<dag oops, dag iops, AddrMode am, int sz,
              InstrItinClass itin, string asm, string cstr, list<dag> pattern>
  : InstThumb<am, sz, IndexModeNone, ThumbFrm, GenericDomain, cstr, itin> {
  let OutOperandList = oops;
  let InOperandList = iops;
  let AsmString = asm;
  let Pattern = pattern;
  list<Predicate> Predicates = [IsThumb, IsThumb1Only];
}
```
- EN: Declares reusable TableGen class `Thumb1I` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `Thumb1I`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1205-1210
```tablegen
class T1I<dag oops, dag iops, InstrItinClass itin,
          string asm, list<dag> pattern>
  : Thumb1I<oops, iops, AddrModeNone, 2, itin, asm, "", pattern>;
class T1Ix2<dag oops, dag iops, InstrItinClass itin,
            string asm, list<dag> pattern>
  : Thumb1I<oops, iops, AddrModeNone, 4, itin, asm, "", pattern>;
```
- EN: Declares reusable TableGen class `T1I` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T1I`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1212-1216
```tablegen
// Two-address instructions
class T1It<dag oops, dag iops, InstrItinClass itin,
           string asm, string cstr, list<dag> pattern>
  : Thumb1I<oops, iops, AddrModeNone, 2, itin,
            asm, cstr, pattern>;
```
- EN: Declares reusable TableGen class `T1It` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T1It`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1218-1232
```tablegen
// Thumb1 instruction that can either be predicated or set CPSR.
class Thumb1sI<dag oops, dag iops, AddrMode am, int sz,
               InstrItinClass itin,
               string opc, string asm, string cstr, list<dag> pattern>
  : InstThumb<am, sz, IndexModeNone, ThumbFrm, GenericDomain, cstr, itin> {
  bits<0> s;
  bits<0> p;
  let OutOperandList = !con(oops, (outs s_cc_out:$s));
  let InOperandList = !con(iops, (ins pred:$p));
  let AsmString = !strconcat(opc, "${s}${p}", asm);
  let Pattern = pattern;
  let thumbArithFlagSetting = 1;
  list<Predicate> Predicates = [IsThumb, IsThumb1Only];
  let DecoderNamespace = "ThumbSBit";
}
```
- EN: Declares reusable TableGen class `Thumb1sI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `Thumb1sI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1234-1236
```tablegen
class T1sI<dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : Thumb1sI<oops, iops, AddrModeNone, 2, itin, opc, asm, "", pattern>;
```
- EN: Declares reusable TableGen class `T1sI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T1sI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1238-1242
```tablegen
// Two-address instructions
class T1sIt<dag oops, dag iops, InstrItinClass itin,
            string opc, string asm, list<dag> pattern>
  : Thumb1sI<oops, iops, AddrModeNone, 2, itin, opc, asm,
             "$Rn = $Rdn", pattern>;
```
- EN: Declares reusable TableGen class `T1sIt` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T1sIt`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1244-1255
```tablegen
// Thumb1 instruction that can be predicated.
class Thumb1pI<dag oops, dag iops, AddrMode am, int sz,
               InstrItinClass itin,
               string opc, string asm, string cstr, list<dag> pattern>
  : InstThumb<am, sz, IndexModeNone, ThumbFrm, GenericDomain, cstr, itin> {
  bits<0> p;
  let OutOperandList = oops;
  let InOperandList = !con(iops, (ins pred:$p));
  let AsmString = !strconcat(opc, "${p}", asm);
  let Pattern = pattern;
  list<Predicate> Predicates = [IsThumb, IsThumb1Only];
}
```
- EN: Declares reusable TableGen class `Thumb1pI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `Thumb1pI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1257-1259
```tablegen
class T1pI<dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : Thumb1pI<oops, iops, AddrModeNone, 2, itin, opc, asm, "", pattern>;
```
- EN: Declares reusable TableGen class `T1pI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T1pI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1261-1265
```tablegen
// Two-address instructions
class T1pIt<dag oops, dag iops, InstrItinClass itin,
            string opc, string asm, list<dag> pattern>
  : Thumb1pI<oops, iops, AddrModeNone, 2, itin, opc, asm,
             "$Rn = $Rdn", pattern>;
```
- EN: Declares reusable TableGen class `T1pIt` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T1pIt`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1267-1269
```tablegen
class T1pIs<dag oops, dag iops,
            InstrItinClass itin, string opc, string asm, list<dag> pattern>
  : Thumb1pI<oops, iops, AddrModeT1_s, 2, itin, opc, asm, "", pattern>;
```
- EN: Declares reusable TableGen class `T1pIs` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T1pIs`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1271-1273
```tablegen
class Encoding16 : Encoding {
  let Inst{31-16} = 0x0000;
}
```
- EN: Declares reusable TableGen class `Encoding16` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `Encoding16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1275-1278
```tablegen
// A6.2 16-bit Thumb instruction encoding
class T1Encoding<bits<6> opcode> : Encoding16 {
  let Inst{15-10} = opcode;
}
```
- EN: Declares reusable TableGen class `T1Encoding` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T1Encoding`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1280-1284
```tablegen
// A6.2.1 Shift (immediate), add, subtract, move, and compare encoding.
class T1General<bits<5> opcode> : Encoding16 {
  let Inst{15-14} = 0b00;
  let Inst{13-9} = opcode;
}
```
- EN: Declares reusable TableGen class `T1General` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T1General`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1286-1290
```tablegen
// A6.2.2 Data-processing encoding.
class T1DataProcessing<bits<4> opcode> : Encoding16 {
  let Inst{15-10} = 0b010000;
  let Inst{9-6} = opcode;
}
```
- EN: Declares reusable TableGen class `T1DataProcessing` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T1DataProcessing`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1292-1296
```tablegen
// A6.2.3 Special data instructions and branch and exchange encoding.
class T1Special<bits<4> opcode> : Encoding16 {
  let Inst{15-10} = 0b010001;
  let Inst{9-6}   = opcode;
}
```
- EN: Declares reusable TableGen class `T1Special` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T1Special`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1298-1303
```tablegen
// A6.2.4 Load/store single data item encoding.
class T1LoadStore<bits<4> opA, bits<3> opB> : Encoding16 {
  let Inst{15-12} = opA;
  let Inst{11-9}  = opB;
}
class T1LdStSP<bits<3> opB>   : T1LoadStore<0b1001, opB>; // SP relative
```
- EN: Declares reusable TableGen class `T1LoadStore` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T1LoadStore`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1305-1307
```tablegen
class T1BranchCond<bits<4> opcode> : Encoding16 {
  let Inst{15-12} = opcode;
}
```
- EN: Declares reusable TableGen class `T1BranchCond` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T1BranchCond`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1309-1326
```tablegen
// Helper classes to encode Thumb1 loads and stores. For immediates, the
// following bits are used for "opA" (see A6.2.4):
//
//   0b0110 => Immediate, 4 bytes
//   0b1000 => Immediate, 2 bytes
//   0b0111 => Immediate, 1 byte
class T1pILdStEncode<bits<3> opcode, dag oops, dag iops, AddrMode am,
                     InstrItinClass itin, string opc, string asm,
                     list<dag> pattern>
  : Thumb1pI<oops, iops, am, 2, itin, opc, asm, "", pattern>,
    T1LoadStore<0b0101, opcode> {
  bits<3> Rt;
  bits<8> addr;
  let Inst{8-6} = addr{5-3};    // Rm
  let Inst{5-3} = addr{2-0};    // Rn
  let Inst{2-0} = Rt;
}
class T1pILdStEncodeImm<bits<4> opA, bit opB, dag oops, dag iops, AddrMode am,
```
- EN: Declares reusable TableGen class `T1pILdStEncode` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T1pILdStEncode`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1327-1336
```tablegen
                        InstrItinClass itin, string opc, string asm,
                        list<dag> pattern>
  : Thumb1pI<oops, iops, am, 2, itin, opc, asm, "", pattern>,
    T1LoadStore<opA, {opB,?,?}> {
  bits<3> Rt;
  bits<8> addr;
  let Inst{10-6} = addr{7-3};   // imm5
  let Inst{5-3}  = addr{2-0};   // Rn
  let Inst{2-0}  = Rt;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1338-1342
```tablegen
// A6.2.5 Miscellaneous 16-bit instructions encoding.
class T1Misc<bits<7> opcode> : Encoding16 {
  let Inst{15-12} = 0b1011;
  let Inst{11-5} = opcode;
}
```
- EN: Declares reusable TableGen class `T1Misc` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T1Misc`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1344-1356
```tablegen
// Thumb2I - Thumb2 instruction. Almost all Thumb2 instructions are predicable.
class Thumb2I<dag oops, dag iops, AddrMode am, int sz,
              InstrItinClass itin,
              string opc, string asm, string cstr, list<dag> pattern>
  : InstARM<am, sz, IndexModeNone, ThumbFrm, GenericDomain, cstr, itin> {
  bits<0> p;
  let OutOperandList = oops;
  let InOperandList = !con(iops, (ins pred:$p));
  let AsmString = !strconcat(opc, "${p}", asm);
  let Pattern = pattern;
  list<Predicate> Predicates = [IsThumb2];
  let DecoderNamespace = "Thumb2";
}
```
- EN: Declares reusable TableGen class `Thumb2I` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `Thumb2I`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1358-1370
```tablegen
// Same as Thumb2I except it can optionally modify CPSR. Note it's modeled as an
// input operand since by default it's a zero register. It will become an
// implicit def once it's "flipped".
//
// FIXME: This uses unified syntax so {s} comes before {p}. We should make it
// more consistent.
class Thumb2sI<dag oops, dag iops, AddrMode am, int sz,
               InstrItinClass itin,
               string opc, string asm, string cstr, list<dag> pattern>
  : InstARM<am, sz, IndexModeNone, ThumbFrm, GenericDomain, cstr, itin> {
  bits<0> p;
  bits<1> s; // condition-code set flag ('1' if the insn should set the flags)
  let Inst{20} = s;
```
- EN: Declares reusable TableGen class `Thumb2sI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `Thumb2sI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1372-1378
```tablegen
  let OutOperandList = oops;
  let InOperandList = !con(iops, (ins pred:$p, cc_out:$s));
  let AsmString = !strconcat(opc, "${s}${p}", asm);
  let Pattern = pattern;
  list<Predicate> Predicates = [IsThumb2];
  let DecoderNamespace = "Thumb2";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1380-1391
```tablegen
// Special cases
class Thumb2XI<dag oops, dag iops, AddrMode am, int sz,
               InstrItinClass itin,
               string asm, string cstr, list<dag> pattern>
  : InstARM<am, sz, IndexModeNone, ThumbFrm, GenericDomain, cstr, itin> {
  let OutOperandList = oops;
  let InOperandList = iops;
  let AsmString = asm;
  let Pattern = pattern;
  list<Predicate> Predicates = [IsThumb2];
  let DecoderNamespace = "Thumb2";
}
```
- EN: Declares reusable TableGen class `Thumb2XI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `Thumb2XI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1393-1403
```tablegen
class ThumbXI<dag oops, dag iops, AddrMode am, int sz,
              InstrItinClass itin,
              string asm, string cstr, list<dag> pattern>
  : InstARM<am, sz, IndexModeNone, ThumbFrm, GenericDomain, cstr, itin> {
  let OutOperandList = oops;
  let InOperandList = iops;
  let AsmString = asm;
  let Pattern = pattern;
  list<Predicate> Predicates = [IsThumb, IsThumb1Only];
  let DecoderNamespace = "Thumb";
}
```
- EN: Declares reusable TableGen class `ThumbXI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `ThumbXI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1405-1422
```tablegen
class T2I<dag oops, dag iops, InstrItinClass itin,
          string opc, string asm, list<dag> pattern, AddrMode am = AddrModeNone>
  : Thumb2I<oops, iops, am, 4, itin, opc, asm, "", pattern>;
class T2Ii12<dag oops, dag iops, InstrItinClass itin,
             string opc, string asm, list<dag> pattern>
  : Thumb2I<oops, iops, AddrModeT2_i12, 4, itin, opc, asm, "",pattern>;
class T2Ii8p<dag oops, dag iops, InstrItinClass itin,
             string opc, string asm, list<dag> pattern>
  : Thumb2I<oops, iops, AddrModeT2_i8pos, 4, itin, opc, asm, "", pattern>;
class T2Ii8n<dag oops, dag iops, InstrItinClass itin,
             string opc, string asm, list<dag> pattern>
  : Thumb2I<oops, iops, AddrModeT2_i8neg, 4, itin, opc, asm, "", pattern>;
class T2Iso<dag oops, dag iops, InstrItinClass itin,
            string opc, string asm, list<dag> pattern>
  : Thumb2I<oops, iops, AddrModeT2_so, 4, itin, opc, asm, "", pattern>;
class T2Ipc<dag oops, dag iops, InstrItinClass itin,
            string opc, string asm, list<dag> pattern>
  : Thumb2I<oops, iops, AddrModeT2_pc, 4, itin, opc, asm, "", pattern>;
```
- EN: Declares reusable TableGen class `T2I` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T2I`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1423-1440
```tablegen
class T2Ii8s4<bit P, bit W, bit isLoad, dag oops, dag iops, InstrItinClass itin,
              string opc, string asm, string cstr, list<dag> pattern>
  : Thumb2I<oops, iops, AddrModeT2_i8s4, 4, itin, opc, asm, cstr,
            pattern> {
  bits<4> Rt;
  bits<4> Rt2;
  bits<13> addr;
  let Inst{31-25} = 0b1110100;
  let Inst{24}    = P;
  let Inst{23}    = addr{8};
  let Inst{22}    = 1;
  let Inst{21}    = W;
  let Inst{20}    = isLoad;
  let Inst{19-16} = addr{12-9};
  let Inst{15-12} = Rt{3-0};
  let Inst{11-8}  = Rt2{3-0};
  let Inst{7-0}   = addr{7-0};
}
```
- EN: Declares reusable TableGen class `T2Ii8s4` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T2Ii8s4`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1441-1458
```tablegen
class T2Ii8s4post<bit P, bit W, bit isLoad, dag oops, dag iops,
                  InstrItinClass itin, string opc, string asm, string cstr,
                  list<dag> pattern>
  : Thumb2I<oops, iops, AddrModeT2_i8s4, 4, itin, opc, asm, cstr,
            pattern> {
  bits<4> Rt;
  bits<4> Rt2;
  bits<4> addr;
  bits<9> imm;
  let Inst{31-25} = 0b1110100;
  let Inst{24}    = P;
  let Inst{23}    = imm{8};
  let Inst{22}    = 1;
  let Inst{21}    = W;
  let Inst{20}    = isLoad;
  let Inst{19-16} = addr;
  let Inst{15-12} = Rt{3-0};
  let Inst{11-8}  = Rt2{3-0};
```
- EN: Declares reusable TableGen class `T2Ii8s4post` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T2Ii8s4post`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1459-1460
```tablegen
  let Inst{7-0}   = imm{7-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1462-1464
```tablegen
class T2sI<dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : Thumb2sI<oops, iops, AddrModeNone, 4, itin, opc, asm, "", pattern>;
```
- EN: Declares reusable TableGen class `T2sI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T2sI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1466-1471
```tablegen
class T2XI<dag oops, dag iops, InstrItinClass itin,
           string asm, list<dag> pattern>
  : Thumb2XI<oops, iops, AddrModeNone, 4, itin, asm, "", pattern>;
class T2JTI<dag oops, dag iops, InstrItinClass itin,
            string asm, list<dag> pattern>
  : Thumb2XI<oops, iops, AddrModeNone, 0, itin, asm, "", pattern>;
```
- EN: Declares reusable TableGen class `T2XI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T2XI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1473-1478
```tablegen
// Move to/from coprocessor instructions
class T2Cop<bits<4> opc, dag oops, dag iops, string opcstr, string asm,
            list<dag> pattern>
  : T2I <oops, iops, NoItinerary, opcstr, asm, pattern>, Requires<[IsThumb2]> {
  let Inst{31-28} = opc;
}
```
- EN: Declares reusable TableGen class `T2Cop` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T2Cop`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1480-1483
```tablegen
// Two-address instructions
class T2XIt<dag oops, dag iops, InstrItinClass itin,
            string asm, string cstr, list<dag> pattern>
  : Thumb2XI<oops, iops, AddrModeNone, 4, itin, asm, cstr, pattern>;
```
- EN: Declares reusable TableGen class `T2XIt` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T2XIt`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1485-1496
```tablegen
// T2Ipreldst - Thumb2 pre-indexed load / store instructions.
class T2Ipreldst<bit signed, bits<2> opcod, bit load, bit pre,
                 dag oops, dag iops,
                 AddrMode am, IndexMode im, InstrItinClass itin,
                 string opc, string asm, string cstr, list<dag> pattern>
  : InstARM<am, 4, im, ThumbFrm, GenericDomain, cstr, itin> {
  let OutOperandList = oops;
  let InOperandList = !con(iops, (ins pred:$p));
  let AsmString = !strconcat(opc, "${p}", asm);
  let Pattern = pattern;
  list<Predicate> Predicates = [IsThumb2];
  let DecoderNamespace = "Thumb2";
```
- EN: Declares reusable TableGen class `T2Ipreldst` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T2Ipreldst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1498-1513
```tablegen
  bits<4> Rt;
  bits<13> addr;
  let Inst{31-27} = 0b11111;
  let Inst{26-25} = 0b00;
  let Inst{24}    = signed;
  let Inst{23}    = 0;
  let Inst{22-21} = opcod;
  let Inst{20}    = load;
  let Inst{19-16} = addr{12-9};
  let Inst{15-12} = Rt{3-0};
  let Inst{11}    = 1;
  // (P, W) = (1, 1) Pre-indexed or (0, 1) Post-indexed
  let Inst{10}    = pre; // The P bit.
  let Inst{9}     = addr{8}; // Sign bit
  let Inst{8}     = 1; // The W bit.
  let Inst{7-0}   = addr{7-0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1515-1516
```tablegen
  let DecoderMethod = "DecodeT2LdStPre";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1518-1529
```tablegen
// T2Ipostldst - Thumb2 post-indexed load / store instructions.
class T2Ipostldst<bit signed, bits<2> opcod, bit load, bit pre,
                 dag oops, dag iops,
                 AddrMode am, IndexMode im, InstrItinClass itin,
                 string opc, string asm, string cstr, list<dag> pattern>
  : InstARM<am, 4, im, ThumbFrm, GenericDomain, cstr, itin> {
  let OutOperandList = oops;
  let InOperandList = !con(iops, (ins pred:$p));
  let AsmString = !strconcat(opc, "${p}", asm);
  let Pattern = pattern;
  list<Predicate> Predicates = [IsThumb2];
  let DecoderNamespace = "Thumb2";
```
- EN: Declares reusable TableGen class `T2Ipostldst` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T2Ipostldst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1531-1547
```tablegen
  bits<4> Rt;
  bits<4> Rn;
  bits<9> offset;
  let Inst{31-27} = 0b11111;
  let Inst{26-25} = 0b00;
  let Inst{24}    = signed;
  let Inst{23}    = 0;
  let Inst{22-21} = opcod;
  let Inst{20}    = load;
  let Inst{19-16} = Rn;
  let Inst{15-12} = Rt{3-0};
  let Inst{11}    = 1;
  // (P, W) = (1, 1) Pre-indexed or (0, 1) Post-indexed
  let Inst{10}    = pre; // The P bit.
  let Inst{9}     = offset{8}; // Sign bit
  let Inst{8}     = 1; // The W bit.
  let Inst{7-0}   = offset{7-0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1549-1550
```tablegen
  let DecoderMethod = "DecodeT2LdStPre";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1552-1555
```tablegen
// T1Pat - Same as Pat<>, but requires that the compiler be in Thumb1 mode.
class T1Pat<dag pattern, dag result> : Pat<pattern, result> {
  list<Predicate> Predicates = [IsThumb, IsThumb1Only];
}
```
- EN: Declares reusable TableGen class `T1Pat` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T1Pat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1557-1560
```tablegen
// T2v6Pat - Same as Pat<>, but requires V6T2 Thumb2 mode.
class T2v6Pat<dag pattern, dag result> : Pat<pattern, result> {
  list<Predicate> Predicates = [IsThumb2, HasV6T2];
}
```
- EN: Declares reusable TableGen class `T2v6Pat` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T2v6Pat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1562-1565
```tablegen
// T2Pat - Same as Pat<>, but requires that the compiler be in Thumb2 mode.
class T2Pat<dag pattern, dag result> : Pat<pattern, result> {
  list<Predicate> Predicates = [IsThumb2];
}
```
- EN: Declares reusable TableGen class `T2Pat` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `T2Pat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1567-1567
```tablegen
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 1569-1571
```tablegen
//===----------------------------------------------------------------------===//
// ARM VFP Instruction templates.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 1573-1587
```tablegen
// Almost all VFP instructions are predicable.
class VFPI<dag oops, dag iops, AddrMode am, int sz,
           IndexMode im, Format f, InstrItinClass itin,
           string opc, string asm, string cstr, list<dag> pattern>
  : InstARM<am, sz, im, f, VFPDomain, cstr, itin> {
  bits<4> p;
  let Inst{31-28} = p;
  let OutOperandList = oops;
  let InOperandList = !con(iops, (ins pred:$p));
  let AsmString = !strconcat(opc, "${p}", asm);
  let Pattern = pattern;
  let PostEncoderMethod = "VFPThumb2PostEncoder";
  let DecoderNamespace = "VFP";
  list<Predicate> Predicates = [HasVFP2];
}
```
- EN: Declares reusable TableGen class `VFPI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `VFPI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1589-1603
```tablegen
// Special cases
class VFPXI<dag oops, dag iops, AddrMode am, int sz,
            IndexMode im, Format f, InstrItinClass itin,
            string asm, string cstr, list<dag> pattern>
  : InstARM<am, sz, im, f, VFPDomain, cstr, itin> {
  bits<4> p;
  let Inst{31-28} = p;
  let OutOperandList = oops;
  let InOperandList = iops;
  let AsmString = asm;
  let Pattern = pattern;
  let PostEncoderMethod = "VFPThumb2PostEncoder";
  let DecoderNamespace = "VFP";
  list<Predicate> Predicates = [HasVFP2];
}
```
- EN: Declares reusable TableGen class `VFPXI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `VFPXI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1605-1610
```tablegen
class VFPAI<dag oops, dag iops, Format f, InstrItinClass itin,
            string opc, string asm, string cstr, list<dag> pattern>
  : VFPI<oops, iops, AddrModeNone, 4, IndexModeNone, f, itin,
         opc, asm, cstr, pattern> {
  let PostEncoderMethod = "VFPThumb2PostEncoder";
}
```
- EN: Declares reusable TableGen class `VFPAI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `VFPAI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1612-1620
```tablegen
// ARM VFP addrmode5 loads and stores
class ADI5<bits<4> opcod1, bits<2> opcod2, dag oops, dag iops,
           InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : VFPI<oops, iops, AddrMode5, 4, IndexModeNone,
         VFPLdStFrm, itin, opc, asm, "", pattern> {
  // Instruction operands.
  bits<5>  Dd;
  bits<13> addr;
```
- EN: Declares reusable TableGen class `ADI5` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `ADI5`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1622-1627
```tablegen
  // Encode instruction operands.
  let Inst{23}    = addr{8};      // U (add = (U == '1'))
  let Inst{22}    = Dd{4};
  let Inst{19-16} = addr{12-9};   // Rn
  let Inst{15-12} = Dd{3-0};
  let Inst{7-0}   = addr{7-0};    // imm8
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1629-1632
```tablegen
  let Inst{27-24} = opcod1;
  let Inst{21-20} = opcod2;
  let Inst{11-9}  = 0b101;
  let Inst{8}     = 1;          // Double precision
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1634-1636
```tablegen
  // Loads & stores operate on both NEON and VFP pipelines.
  let D = VFPNeonDomain;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1638-1645
```tablegen
class ASI5<bits<4> opcod1, bits<2> opcod2, dag oops, dag iops,
           InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : VFPI<oops, iops, AddrMode5, 4, IndexModeNone,
         VFPLdStFrm, itin, opc, asm, "", pattern> {
  // Instruction operands.
  bits<5>  Sd;
  bits<13> addr;
```
- EN: Declares reusable TableGen class `ASI5` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `ASI5`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1647-1652
```tablegen
  // Encode instruction operands.
  let Inst{23}    = addr{8};      // U (add = (U == '1'))
  let Inst{22}    = Sd{0};
  let Inst{19-16} = addr{12-9};   // Rn
  let Inst{15-12} = Sd{4-1};
  let Inst{7-0}   = addr{7-0};    // imm8
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1654-1657
```tablegen
  let Inst{27-24} = opcod1;
  let Inst{21-20} = opcod2;
  let Inst{11-9}  = 0b101;
  let Inst{8}     = 0;          // Single precision
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1659-1661
```tablegen
  // Loads & stores operate on both NEON and VFP pipelines.
  let D = VFPNeonDomain;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1663-1668
```tablegen
class AHI5<bits<4> opcod1, bits<2> opcod2, dag oops, dag iops,
           InstrItinClass itin,
           string opc, string asm, list<dag> pattern>
  : VFPI<oops, iops, AddrMode5FP16, 4, IndexModeNone,
         VFPLdStFrm, itin, opc, asm, "", pattern> {
  list<Predicate> Predicates = [HasFullFP16];
```
- EN: Declares reusable TableGen class `AHI5` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AHI5`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1670-1672
```tablegen
  // Instruction operands.
  bits<5>  Sd;
  bits<13> addr;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1674-1679
```tablegen
  // Encode instruction operands.
  let Inst{23}    = addr{8};      // U (add = (U == '1'))
  let Inst{22}    = Sd{0};
  let Inst{19-16} = addr{12-9};   // Rn
  let Inst{15-12} = Sd{4-1};
  let Inst{7-0}   = addr{7-0};    // imm8
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1681-1683
```tablegen
  let Inst{27-24} = opcod1;
  let Inst{21-20} = opcod2;
  let Inst{11-8}  = 0b1001;     // Half precision
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1685-1686
```tablegen
  // Loads & stores operate on both NEON and VFP pipelines.
  let D = VFPNeonDomain;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1688-1689
```tablegen
  let isUnpredicable = 1; // FP16 instructions cannot in general be conditional
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1691-1700
```tablegen
// VFP Load / store multiple pseudo instructions.
class PseudoVFPLdStM<dag oops, dag iops, InstrItinClass itin, string cstr,
                     list<dag> pattern>
  : InstARM<AddrMode4, 4, IndexModeNone, Pseudo, VFPNeonDomain,
            cstr, itin> {
  let OutOperandList = oops;
  let InOperandList = !con(iops, (ins pred:$p));
  let Pattern = pattern;
  list<Predicate> Predicates = [HasVFP2];
}
```
- EN: Declares reusable TableGen class `PseudoVFPLdStM` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `PseudoVFPLdStM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1702-1702
```tablegen
// Load / store multiple
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1704-1711
```tablegen
// Unknown precision
class AXXI4<dag oops, dag iops, IndexMode im,
            string asm, string cstr, list<dag> pattern>
  : VFPXI<oops, iops, AddrMode4, 4, im,
          VFPLdStFrm, NoItinerary, asm, cstr, pattern> {
  // Instruction operands.
  bits<4>  Rn;
  bits<13> regs;
```
- EN: Declares reusable TableGen class `AXXI4` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AXXI4`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1713-1717
```tablegen
  // Encode instruction operands.
  let Inst{19-16} = Rn;
  let Inst{22}    = 0;
  let Inst{15-12} = regs{11-8};
  let Inst{7-1}   = regs{7-1};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1719-1722
```tablegen
  let Inst{27-25} = 0b110;
  let Inst{11-8}  = 0b1011;
  let Inst{0}     = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1724-1731
```tablegen
// Double precision
class AXDI4<dag oops, dag iops, IndexMode im, InstrItinClass itin,
            string asm, string cstr, list<dag> pattern>
  : VFPXI<oops, iops, AddrMode4, 4, im,
          VFPLdStMulFrm, itin, asm, cstr, pattern> {
  // Instruction operands.
  bits<4>  Rn;
  bits<13> regs;
```
- EN: Declares reusable TableGen class `AXDI4` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AXDI4`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1733-1737
```tablegen
  // Encode instruction operands.
  let Inst{19-16} = Rn;
  let Inst{22}    = regs{12};
  let Inst{15-12} = regs{11-8};
  let Inst{7-1}   = regs{7-1};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1739-1743
```tablegen
  let Inst{27-25} = 0b110;
  let Inst{11-9}  = 0b101;
  let Inst{8}     = 1;          // Double precision
  let Inst{0}     = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1745-1752
```tablegen
// Single Precision
class AXSI4<dag oops, dag iops, IndexMode im, InstrItinClass itin,
            string asm, string cstr, list<dag> pattern>
  : VFPXI<oops, iops, AddrMode4, 4, im,
          VFPLdStMulFrm, itin, asm, cstr, pattern> {
  // Instruction operands.
  bits<4> Rn;
  bits<13> regs;
```
- EN: Declares reusable TableGen class `AXSI4` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AXSI4`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1754-1758
```tablegen
  // Encode instruction operands.
  let Inst{19-16} = Rn;
  let Inst{22}    = regs{8};
  let Inst{15-12} = regs{12-9};
  let Inst{7-0}   = regs{7-0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1760-1763
```tablegen
  let Inst{27-25} = 0b110;
  let Inst{11-9}  = 0b101;
  let Inst{8}     = 0;          // Single precision
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1765-1782
```tablegen
// Single Precision with fixed registers.
// For when the registers-to-be-stored/loaded are fixed, e.g. VLLDM and VLSTM
class AXSI4FR<string asm, bit et, bit load>
    : InstARM<AddrMode4, 4, IndexModeNone, VFPLdStMulFrm, VFPDomain, "", NoItinerary> {
  // Instruction operands.
  bits<4> Rn;
  bits<13> regs;    // Does not affect encoding, for assembly/disassembly only.
  list<Predicate> Predicates = [HasVFP2];
  let OutOperandList = (outs);
  let InOperandList = (ins GPRnopc:$Rn, pred:$p, dpr_reglist:$regs);
  let AsmString = asm;
  let Pattern = [];
  let DecoderNamespace = "VFP";
  // Encode instruction operands.
  let Inst{19-16} = Rn;
  let Inst{31-28} = 0b1110;
  let Inst{27-25} = 0b110;
  let Inst{24}    = 0b0;
```
- EN: Declares reusable TableGen class `AXSI4FR` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AXSI4FR`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1783-1794
```tablegen
  let Inst{23}    = 0b0;
  let Inst{22}    = 0b0;
  let Inst{21}    = 0b1;
  let Inst{20}    = load;       // Distinguishes vlldm from vlstm
  let Inst{15-12} = 0b0000;
  let Inst{11-9}  = 0b101;
  let Inst{8}     = 0;          // Single precision
  let Inst{7}     = et;         // encoding type, 0 for T1 and 1 for T2.
  let Inst{6-0}   = 0b0000000;
  let mayLoad     = load;
  let mayStore    = !eq(load, 0);
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1796-1803
```tablegen
// Double precision, unary
class ADuI<bits<5> opcod1, bits<2> opcod2, bits<4> opcod3, bits<2> opcod4,
           bit opcod5, dag oops, dag iops, InstrItinClass itin, string opc,
           string asm, string cstr, list<dag> pattern>
  : VFPAI<oops, iops, VFPUnaryFrm, itin, opc, asm, cstr, pattern> {
  // Instruction operands.
  bits<5> Dd;
  bits<5> Dm;
```
- EN: Declares reusable TableGen class `ADuI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `ADuI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1805-1809
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Dm{3-0};
  let Inst{5}     = Dm{4};
  let Inst{15-12} = Dd{3-0};
  let Inst{22}    = Dd{4};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1811-1817
```tablegen
  let Inst{27-23} = opcod1;
  let Inst{21-20} = opcod2;
  let Inst{19-16} = opcod3;
  let Inst{11-9}  = 0b101;
  let Inst{8}     = 1;          // Double precision
  let Inst{7-6}   = opcod4;
  let Inst{4}     = opcod5;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1819-1820
```tablegen
  let Predicates = [HasVFP2, HasDPVFP];
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1822-1829
```tablegen
// Double precision, unary, not-predicated
class ADuInp<bits<5> opcod1, bits<2> opcod2, bits<4> opcod3, bits<2> opcod4,
           bit opcod5, dag oops, dag iops, InstrItinClass itin,
           string asm, list<dag> pattern>
  : VFPXI<oops, iops, AddrModeNone, 4, IndexModeNone, VFPUnaryFrm, itin, asm, "", pattern> {
  // Instruction operands.
  bits<5> Dd;
  bits<5> Dm;
```
- EN: Declares reusable TableGen class `ADuInp` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `ADuInp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1831-1831
```tablegen
  let Inst{31-28} = 0b1111;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1833-1837
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Dm{3-0};
  let Inst{5}     = Dm{4};
  let Inst{15-12} = Dd{3-0};
  let Inst{22}    = Dd{4};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1839-1846
```tablegen
  let Inst{27-23} = opcod1;
  let Inst{21-20} = opcod2;
  let Inst{19-16} = opcod3;
  let Inst{11-9}  = 0b101;
  let Inst{8}     = 1;          // Double precision
  let Inst{7-6}   = opcod4;
  let Inst{4}     = opcod5;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1848-1856
```tablegen
// Double precision, binary
class ADbI<bits<5> opcod1, bits<2> opcod2, bit op6, bit op4, dag oops,
           dag iops, InstrItinClass itin, string opc, string asm,
           list<dag> pattern>
  : VFPAI<oops, iops, VFPBinaryFrm, itin, opc, asm, "", pattern> {
  // Instruction operands.
  bits<5> Dd;
  bits<5> Dn;
  bits<5> Dm;
```
- EN: Declares reusable TableGen class `ADbI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `ADbI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1858-1864
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Dm{3-0};
  let Inst{5}     = Dm{4};
  let Inst{19-16} = Dn{3-0};
  let Inst{7}     = Dn{4};
  let Inst{15-12} = Dd{3-0};
  let Inst{22}    = Dd{4};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1866-1871
```tablegen
  let Inst{27-23} = opcod1;
  let Inst{21-20} = opcod2;
  let Inst{11-9}  = 0b101;
  let Inst{8}     = 1;          // Double precision
  let Inst{6}     = op6;
  let Inst{4}     = op4;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1873-1874
```tablegen
  let Predicates = [HasVFP2, HasDPVFP];
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1876-1885
```tablegen
// FP, binary, not predicated
class ADbInp<bits<5> opcod1, bits<2> opcod2, bit opcod3, dag oops, dag iops,
           InstrItinClass itin, string asm, list<dag> pattern>
  : VFPXI<oops, iops, AddrModeNone, 4, IndexModeNone, VFPBinaryFrm, itin,
          asm, "", pattern>
{
  // Instruction operands.
  bits<5> Dd;
  bits<5> Dn;
  bits<5> Dm;
```
- EN: Declares reusable TableGen class `ADbInp` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `ADbInp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1887-1887
```tablegen
  let Inst{31-28} = 0b1111;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1889-1895
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Dm{3-0};
  let Inst{5}     = Dm{4};
  let Inst{19-16} = Dn{3-0};
  let Inst{7}     = Dn{4};
  let Inst{15-12} = Dd{3-0};
  let Inst{22}    = Dd{4};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1897-1902
```tablegen
  let Inst{27-23} = opcod1;
  let Inst{21-20} = opcod2;
  let Inst{11-9}  = 0b101;
  let Inst{8}     = 1; // double precision
  let Inst{6}     = opcod3;
  let Inst{4}     = 0;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1904-1905
```tablegen
  let Predicates = [HasVFP2, HasDPVFP];
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1907-1914
```tablegen
// Single precision, unary, predicated
class ASuI<bits<5> opcod1, bits<2> opcod2, bits<4> opcod3, bits<2> opcod4,
           bit opcod5, dag oops, dag iops, InstrItinClass itin, string opc,
           string asm, string cstr, list<dag> pattern>
  : VFPAI<oops, iops, VFPUnaryFrm, itin, opc, asm, cstr, pattern> {
  // Instruction operands.
  bits<5> Sd;
  bits<5> Sm;
```
- EN: Declares reusable TableGen class `ASuI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `ASuI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1916-1920
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Sm{4-1};
  let Inst{5}     = Sm{0};
  let Inst{15-12} = Sd{4-1};
  let Inst{22}    = Sd{0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1922-1929
```tablegen
  let Inst{27-23} = opcod1;
  let Inst{21-20} = opcod2;
  let Inst{19-16} = opcod3;
  let Inst{11-9}  = 0b101;
  let Inst{8}     = 0;          // Single precision
  let Inst{7-6}   = opcod4;
  let Inst{4}     = opcod5;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1931-1939
```tablegen
// Single precision, unary, non-predicated
class ASuInp<bits<5> opcod1, bits<2> opcod2, bits<4> opcod3, bits<2> opcod4,
             bit opcod5, dag oops, dag iops, InstrItinClass itin,
             string asm, list<dag> pattern>
  : VFPXI<oops, iops, AddrModeNone, 4, IndexModeNone,
          VFPUnaryFrm, itin, asm, "", pattern> {
  // Instruction operands.
  bits<5> Sd;
  bits<5> Sm;
```
- EN: Declares reusable TableGen class `ASuInp` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `ASuInp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1941-1941
```tablegen
  let Inst{31-28} = 0b1111;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1943-1947
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Sm{4-1};
  let Inst{5}     = Sm{0};
  let Inst{15-12} = Sd{4-1};
  let Inst{22}    = Sd{0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1949-1956
```tablegen
  let Inst{27-23} = opcod1;
  let Inst{21-20} = opcod2;
  let Inst{19-16} = opcod3;
  let Inst{11-9}  = 0b101;
  let Inst{8}     = 0;          // Single precision
  let Inst{7-6}   = opcod4;
  let Inst{4}     = opcod5;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1958-1966
```tablegen
// Single precision unary, if no NEON. Same as ASuI except not available if
// NEON is enabled.
class ASuIn<bits<5> opcod1, bits<2> opcod2, bits<4> opcod3, bits<2> opcod4,
            bit opcod5, dag oops, dag iops, InstrItinClass itin, string opc,
            string asm, list<dag> pattern>
  : ASuI<opcod1, opcod2, opcod3, opcod4, opcod5, oops, iops, itin, opc, asm,
         "", pattern> {
  list<Predicate> Predicates = [HasVFP2,DontUseNEONForFP];
}
```
- EN: Declares reusable TableGen class `ASuIn` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `ASuIn`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1968-1975
```tablegen
// Single precision, binary
class ASbI<bits<5> opcod1, bits<2> opcod2, bit op6, bit op4, dag oops, dag iops,
           InstrItinClass itin, string opc, string asm, list<dag> pattern>
  : VFPAI<oops, iops, VFPBinaryFrm, itin, opc, asm, "", pattern> {
  // Instruction operands.
  bits<5> Sd;
  bits<5> Sn;
  bits<5> Sm;
```
- EN: Declares reusable TableGen class `ASbI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `ASbI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1977-1983
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Sm{4-1};
  let Inst{5}     = Sm{0};
  let Inst{19-16} = Sn{4-1};
  let Inst{7}     = Sn{0};
  let Inst{15-12} = Sd{4-1};
  let Inst{22}    = Sd{0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1985-1991
```tablegen
  let Inst{27-23} = opcod1;
  let Inst{21-20} = opcod2;
  let Inst{11-9}  = 0b101;
  let Inst{8}     = 0;          // Single precision
  let Inst{6}     = op6;
  let Inst{4}     = op4;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1993-2002
```tablegen
// Single precision, binary, not predicated
class ASbInp<bits<5> opcod1, bits<2> opcod2, bit opcod3, dag oops, dag iops,
           InstrItinClass itin, string asm, list<dag> pattern>
  : VFPXI<oops, iops, AddrModeNone, 4, IndexModeNone,
          VFPBinaryFrm, itin, asm, "", pattern>
{
  // Instruction operands.
  bits<5> Sd;
  bits<5> Sn;
  bits<5> Sm;
```
- EN: Declares reusable TableGen class `ASbInp` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `ASbInp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2004-2004
```tablegen
  let Inst{31-28} = 0b1111;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2006-2012
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Sm{4-1};
  let Inst{5}     = Sm{0};
  let Inst{19-16} = Sn{4-1};
  let Inst{7}     = Sn{0};
  let Inst{15-12} = Sd{4-1};
  let Inst{22}    = Sd{0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2014-2020
```tablegen
  let Inst{27-23} = opcod1;
  let Inst{21-20} = opcod2;
  let Inst{11-9}  = 0b101;
  let Inst{8}     = 0; // Single precision
  let Inst{6}     = opcod3;
  let Inst{4}     = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2022-2028
```tablegen
// Single precision binary, if no NEON. Same as ASbI except not available if
// NEON is enabled.
class ASbIn<bits<5> opcod1, bits<2> opcod2, bit op6, bit op4, dag oops,
            dag iops, InstrItinClass itin, string opc, string asm,
            list<dag> pattern>
  : ASbI<opcod1, opcod2, op6, op4, oops, iops, itin, opc, asm, pattern> {
  list<Predicate> Predicates = [HasVFP2,DontUseNEONForFP];
```
- EN: Declares reusable TableGen class `ASbIn` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `ASbIn`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2030-2033
```tablegen
  // Instruction operands.
  bits<5> Sd;
  bits<5> Sn;
  bits<5> Sm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2035-2042
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Sm{4-1};
  let Inst{5}     = Sm{0};
  let Inst{19-16} = Sn{4-1};
  let Inst{7}     = Sn{0};
  let Inst{15-12} = Sd{4-1};
  let Inst{22}    = Sd{0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2044-2049
```tablegen
// Half precision, unary, predicated
class AHuI<bits<5> opcod1, bits<2> opcod2, bits<4> opcod3, bits<2> opcod4,
           bit opcod5, dag oops, dag iops, InstrItinClass itin, string opc,
           string asm, list<dag> pattern>
  : VFPAI<oops, iops, VFPUnaryFrm, itin, opc, asm, "", pattern> {
  list<Predicate> Predicates = [HasFullFP16];
```
- EN: Declares reusable TableGen class `AHuI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AHuI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2051-2053
```tablegen
  // Instruction operands.
  bits<5> Sd;
  bits<5> Sm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2055-2059
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Sm{4-1};
  let Inst{5}     = Sm{0};
  let Inst{15-12} = Sd{4-1};
  let Inst{22}    = Sd{0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2061-2066
```tablegen
  let Inst{27-23} = opcod1;
  let Inst{21-20} = opcod2;
  let Inst{19-16} = opcod3;
  let Inst{11-8}  = 0b1001;   // Half precision
  let Inst{7-6}   = opcod4;
  let Inst{4}     = opcod5;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2068-2069
```tablegen
  let isUnpredicable = 1; // FP16 instructions cannot in general be conditional
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2071-2077
```tablegen
// Half precision, unary, non-predicated
class AHuInp<bits<5> opcod1, bits<2> opcod2, bits<4> opcod3, bits<2> opcod4,
             bit opcod5, dag oops, dag iops, InstrItinClass itin,
             string asm, list<dag> pattern>
  : VFPXI<oops, iops, AddrModeNone, 4, IndexModeNone,
          VFPUnaryFrm, itin, asm, "", pattern> {
  list<Predicate> Predicates = [HasFullFP16];
```
- EN: Declares reusable TableGen class `AHuInp` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AHuInp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2079-2081
```tablegen
  // Instruction operands.
  bits<5> Sd;
  bits<5> Sm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2083-2083
```tablegen
  let Inst{31-28} = 0b1111;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2085-2089
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Sm{4-1};
  let Inst{5}     = Sm{0};
  let Inst{15-12} = Sd{4-1};
  let Inst{22}    = Sd{0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2091-2096
```tablegen
  let Inst{27-23} = opcod1;
  let Inst{21-20} = opcod2;
  let Inst{19-16} = opcod3;
  let Inst{11-8}  = 0b1001;   // Half precision
  let Inst{7-6}   = opcod4;
  let Inst{4}     = opcod5;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2098-2099
```tablegen
  let isUnpredicable = 1; // FP16 instructions cannot in general be conditional
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2101-2105
```tablegen
// Half precision, binary
class AHbI<bits<5> opcod1, bits<2> opcod2, bit op6, bit op4, dag oops, dag iops,
           InstrItinClass itin, string opc, string asm, list<dag> pattern>
  : VFPAI<oops, iops, VFPBinaryFrm, itin, opc, asm, "", pattern> {
  list<Predicate> Predicates = [HasFullFP16];
```
- EN: Declares reusable TableGen class `AHbI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AHbI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2107-2110
```tablegen
  // Instruction operands.
  bits<5> Sd;
  bits<5> Sn;
  bits<5> Sm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2112-2118
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Sm{4-1};
  let Inst{5}     = Sm{0};
  let Inst{19-16} = Sn{4-1};
  let Inst{7}     = Sn{0};
  let Inst{15-12} = Sd{4-1};
  let Inst{22}    = Sd{0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2120-2124
```tablegen
  let Inst{27-23} = opcod1;
  let Inst{21-20} = opcod2;
  let Inst{11-8}  = 0b1001;   // Half precision
  let Inst{6}     = op6;
  let Inst{4}     = op4;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2126-2127
```tablegen
  let isUnpredicable = 1; // FP16 instructions cannot in general be conditional
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2129-2134
```tablegen
// Half precision, binary, not predicated
class AHbInp<bits<5> opcod1, bits<2> opcod2, bit opcod3, dag oops, dag iops,
           InstrItinClass itin, string asm, list<dag> pattern>
  : VFPXI<oops, iops, AddrModeNone, 4, IndexModeNone,
          VFPBinaryFrm, itin, asm, "", pattern> {
  list<Predicate> Predicates = [HasFullFP16];
```
- EN: Declares reusable TableGen class `AHbInp` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AHbInp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2136-2139
```tablegen
  // Instruction operands.
  bits<5> Sd;
  bits<5> Sn;
  bits<5> Sm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2141-2141
```tablegen
  let Inst{31-28} = 0b1111;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2143-2149
```tablegen
  // Encode instruction operands.
  let Inst{3-0}   = Sm{4-1};
  let Inst{5}     = Sm{0};
  let Inst{19-16} = Sn{4-1};
  let Inst{7}     = Sn{0};
  let Inst{15-12} = Sd{4-1};
  let Inst{22}    = Sd{0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2151-2155
```tablegen
  let Inst{27-23} = opcod1;
  let Inst{21-20} = opcod2;
  let Inst{11-8}  = 0b1001;   // Half precision
  let Inst{6}     = opcod3;
  let Inst{4}     = 0;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2157-2158
```tablegen
  let isUnpredicable = 1; // FP16 instructions cannot in general be conditional
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2160-2171
```tablegen
// VFP conversion instructions
class AVConv1I<bits<5> opcod1, bits<2> opcod2, bits<4> opcod3, bits<4> opcod4,
               dag oops, dag iops, InstrItinClass itin, string opc, string asm,
               list<dag> pattern>
  : VFPAI<oops, iops, VFPConv1Frm, itin, opc, asm, "", pattern> {
  let Inst{27-23} = opcod1;
  let Inst{21-20} = opcod2;
  let Inst{19-16} = opcod3;
  let Inst{11-8}  = opcod4;
  let Inst{6}     = 1;
  let Inst{4}     = 0;
}
```
- EN: Declares reusable TableGen class `AVConv1I` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AVConv1I`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2173-2183
```tablegen
// VFP conversion between floating-point and fixed-point
class AVConv1XI<bits<5> op1, bits<2> op2, bits<4> op3, bits<4> op4, bit op5,
                dag oops, dag iops, InstrItinClass itin, string opc, string asm,
                list<dag> pattern>
  : AVConv1I<op1, op2, op3, op4, oops, iops, itin, opc, asm, pattern> {
  bits<5> fbits;
  // size (fixed-point number): sx == 0 ? 16 : 32
  let Inst{7} = op5; // sx
  let Inst{5} = fbits{0};
  let Inst{3-0} = fbits{4-1};
}
```
- EN: Declares reusable TableGen class `AVConv1XI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AVConv1XI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2185-2192
```tablegen
// VFP conversion instructions, if no NEON
class AVConv1In<bits<5> opcod1, bits<2> opcod2, bits<4> opcod3, bits<4> opcod4,
                dag oops, dag iops, InstrItinClass itin,
                string opc, string asm, list<dag> pattern>
  : AVConv1I<opcod1, opcod2, opcod3, opcod4, oops, iops, itin, opc, asm,
             pattern> {
  list<Predicate> Predicates = [HasVFP2,DontUseNEONForFP];
}
```
- EN: Declares reusable TableGen class `AVConv1In` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AVConv1In`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2194-2201
```tablegen
class AVConvXI<bits<8> opcod1, bits<4> opcod2, dag oops, dag iops, Format f,
               InstrItinClass itin,
               string opc, string asm, list<dag> pattern>
  : VFPAI<oops, iops, f, itin, opc, asm, "", pattern> {
  let Inst{27-20} = opcod1;
  let Inst{11-8}  = opcod2;
  let Inst{4}     = 1;
}
```
- EN: Declares reusable TableGen class `AVConvXI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AVConvXI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2203-2205
```tablegen
class AVConv2I<bits<8> opcod1, bits<4> opcod2, dag oops, dag iops,
               InstrItinClass itin, string opc, string asm, list<dag> pattern>
  : AVConvXI<opcod1, opcod2, oops, iops, VFPConv2Frm, itin, opc, asm, pattern>;
```
- EN: Declares reusable TableGen class `AVConv2I` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AVConv2I`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2207-2209
```tablegen
class AVConv3I<bits<8> opcod1, bits<4> opcod2, dag oops, dag iops,
               InstrItinClass itin, string opc, string asm, list<dag> pattern>
  : AVConvXI<opcod1, opcod2, oops, iops, VFPConv3Frm, itin, opc, asm, pattern>;
```
- EN: Declares reusable TableGen class `AVConv3I` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AVConv3I`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2211-2213
```tablegen
class AVConv4I<bits<8> opcod1, bits<4> opcod2, dag oops, dag iops,
               InstrItinClass itin, string opc, string asm, list<dag> pattern>
  : AVConvXI<opcod1, opcod2, oops, iops, VFPConv4Frm, itin, opc, asm, pattern>;
```
- EN: Declares reusable TableGen class `AVConv4I` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AVConv4I`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2215-2217
```tablegen
class AVConv5I<bits<8> opcod1, bits<4> opcod2, dag oops, dag iops,
               InstrItinClass itin, string opc, string asm, list<dag> pattern>
  : AVConvXI<opcod1, opcod2, oops, iops, VFPConv5Frm, itin, opc, asm, pattern>;
```
- EN: Declares reusable TableGen class `AVConv5I` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `AVConv5I`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2219-2219
```tablegen
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2221-2223
```tablegen
//===----------------------------------------------------------------------===//
// ARM NEON Instruction templates.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2225-2236
```tablegen
class NeonI<dag oops, dag iops, AddrMode am, IndexMode im, Format f,
            InstrItinClass itin, string opc, string dt, string asm, string cstr,
            list<dag> pattern>
  : InstARM<am, 4, im, f, NeonDomain, cstr, itin> {
  bits<0> p;
  let OutOperandList = oops;
  let InOperandList = !con(iops, (ins pred:$p));
  let AsmString = !strconcat(opc, "${p}", ".", dt, "\t", asm);
  let Pattern = pattern;
  list<Predicate> Predicates = [HasNEON];
  let DecoderNamespace = "NEON";
}
```
- EN: Declares reusable TableGen class `NeonI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `NeonI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2238-2250
```tablegen
// Same as NeonI except it does not have a "data type" specifier.
class NeonXI<dag oops, dag iops, AddrMode am, IndexMode im, Format f,
             InstrItinClass itin, string opc, string asm, string cstr,
             list<dag> pattern>
  : InstARM<am, 4, im, f, NeonDomain, cstr, itin> {
  bits<0> p;
  let OutOperandList = oops;
  let InOperandList = !con(iops, (ins pred:$p));
  let AsmString = !strconcat(opc, "${p}", "\t", asm);
  let Pattern = pattern;
  list<Predicate> Predicates = [HasNEON];
  let DecoderNamespace = "NEON";
}
```
- EN: Declares reusable TableGen class `NeonXI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `NeonXI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2252-2262
```tablegen
// Same as NeonI except it is not predicated
class NeonInp<dag oops, dag iops, AddrMode am, IndexMode im, Format f,
            InstrItinClass itin, string opc, string dt, string asm, string cstr,
            list<dag> pattern>
  : InstARM<am, 4, im, f, NeonDomain, cstr, itin> {
  let OutOperandList = oops;
  let InOperandList = iops;
  let AsmString = !strconcat(opc, ".", dt, "\t", asm);
  let Pattern = pattern;
  list<Predicate> Predicates = [HasNEON];
  let DecoderNamespace = "NEON";
```
- EN: Declares reusable TableGen class `NeonInp` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `NeonInp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2264-2265
```tablegen
  let Inst{31-28} = 0b1111;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2267-2276
```tablegen
class NLdSt<bit op23, bits<2> op21_20, bits<4> op11_8, bits<4> op7_4,
            dag oops, dag iops, InstrItinClass itin,
            string opc, string dt, string asm, string cstr, list<dag> pattern>
  : NeonI<oops, iops, AddrMode6, IndexModeNone, NLdStFrm, itin, opc, dt, asm,
          cstr, pattern> {
  let Inst{31-24} = 0b11110100;
  let Inst{23}    = op23;
  let Inst{21-20} = op21_20;
  let Inst{11-8}  = op11_8;
  let Inst{7-4}   = op7_4;
```
- EN: Declares reusable TableGen class `NLdSt` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `NLdSt`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2278-2279
```tablegen
  let PostEncoderMethod = "NEONThumb2LoadStorePostEncoder";
  let DecoderNamespace = "NEONLoadStore";
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2281-2283
```tablegen
  bits<5> Vd;
  bits<6> Rn;
  bits<4> Rm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2285-2289
```tablegen
  let Inst{22}    = Vd{4};
  let Inst{15-12} = Vd{3-0};
  let Inst{19-16} = Rn{3-0};
  let Inst{3-0}   = Rm{3-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2291-2297
```tablegen
class NLdStLn<bit op23, bits<2> op21_20, bits<4> op11_8, bits<4> op7_4,
            dag oops, dag iops, InstrItinClass itin,
            string opc, string dt, string asm, string cstr, list<dag> pattern>
  : NLdSt<op23, op21_20, op11_8, op7_4, oops, iops, itin, opc,
          dt, asm, cstr, pattern> {
  bits<3> lane;
}
```
- EN: Declares reusable TableGen class `NLdStLn` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `NLdStLn`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2299-2305
```tablegen
class PseudoNLdSt<dag oops, dag iops, InstrItinClass itin, string cstr>
  : InstARM<AddrMode6, 4, IndexModeNone, Pseudo, NeonDomain, cstr,
            itin> {
  let OutOperandList = oops;
  let InOperandList = !con(iops, (ins pred:$p));
  list<Predicate> Predicates = [HasNEON];
}
```
- EN: Declares reusable TableGen class `PseudoNLdSt` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `PseudoNLdSt`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2307-2315
```tablegen
class PseudoNeonI<dag oops, dag iops, InstrItinClass itin, string cstr,
                  list<dag> pattern>
  : InstARM<AddrModeNone, 4, IndexModeNone, Pseudo, NeonDomain, cstr,
            itin> {
  let OutOperandList = oops;
  let InOperandList = !con(iops, (ins pred:$p));
  let Pattern = pattern;
  list<Predicate> Predicates = [HasNEON];
}
```
- EN: Declares reusable TableGen class `PseudoNeonI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `PseudoNeonI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2317-2324
```tablegen
class NDataI<dag oops, dag iops, Format f, InstrItinClass itin,
             string opc, string dt, string asm, string cstr, list<dag> pattern>
  : NeonI<oops, iops, AddrModeNone, IndexModeNone, f, itin, opc, dt, asm, cstr,
          pattern> {
  let Inst{31-25} = 0b1111001;
  let PostEncoderMethod = "NEONThumb2DataIPostEncoder";
  let DecoderNamespace = "NEONData";
}
```
- EN: Declares reusable TableGen class `NDataI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `NDataI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2326-2333
```tablegen
class NDataXI<dag oops, dag iops, Format f, InstrItinClass itin,
              string opc, string asm, string cstr, list<dag> pattern>
  : NeonXI<oops, iops, AddrModeNone, IndexModeNone, f, itin, opc, asm,
           cstr, pattern> {
  let Inst{31-25} = 0b1111001;
  let PostEncoderMethod = "NEONThumb2DataIPostEncoder";
  let DecoderNamespace = "NEONData";
}
```
- EN: Declares reusable TableGen class `NDataXI` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `NDataXI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2335-2348
```tablegen
// NEON "one register and a modified immediate" format.
class N1ModImm<bit op23, bits<3> op21_19, bits<4> op11_8, bit op7, bit op6,
               bit op5, bit op4,
               dag oops, dag iops, InstrItinClass itin,
               string opc, string dt, string asm, string cstr,
               list<dag> pattern>
  : NDataI<oops, iops, N1RegModImmFrm, itin, opc, dt, asm, cstr, pattern> {
  let Inst{23}    = op23;
  let Inst{21-19} = op21_19;
  let Inst{11-8}  = op11_8;
  let Inst{7}     = op7;
  let Inst{6}     = op6;
  let Inst{5}     = op5;
  let Inst{4}     = op4;
```
- EN: Declares reusable TableGen class `N1ModImm` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `N1ModImm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2350-2352
```tablegen
  // Instruction operands.
  bits<5> Vd;
  bits<13> SIMM;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2354-2360
```tablegen
  let Inst{15-12} = Vd{3-0};
  let Inst{22}    = Vd{4};
  let Inst{24}    = SIMM{7};
  let Inst{18-16} = SIMM{6-4};
  let Inst{3-0}   = SIMM{3-0};
  let DecoderMethod = "DecodeVMOVModImmInstruction";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2362-2374
```tablegen
// NEON 2 vector register format.
class N2V<bits<2> op24_23, bits<2> op21_20, bits<2> op19_18, bits<2> op17_16,
          bits<5> op11_7, bit op6, bit op4,
          dag oops, dag iops, InstrItinClass itin,
          string opc, string dt, string asm, string cstr, list<dag> pattern>
  : NDataI<oops, iops, N2RegFrm, itin, opc, dt, asm, cstr, pattern> {
  let Inst{24-23} = op24_23;
  let Inst{21-20} = op21_20;
  let Inst{19-18} = op19_18;
  let Inst{17-16} = op17_16;
  let Inst{11-7}  = op11_7;
  let Inst{6}     = op6;
  let Inst{4}     = op4;
```
- EN: Declares reusable TableGen class `N2V` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `N2V`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2376-2378
```tablegen
  // Instruction operands.
  bits<5> Vd;
  bits<5> Vm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2380-2384
```tablegen
  let Inst{15-12} = Vd{3-0};
  let Inst{22}    = Vd{4};
  let Inst{3-0}   = Vm{3-0};
  let Inst{5}     = Vm{4};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2386-2393
```tablegen
// Same as N2V but not predicated.
class N2Vnp<bits<2> op19_18, bits<2> op17_16, bits<3> op10_8, bit op7, bit op6,
            dag oops, dag iops, InstrItinClass itin, string OpcodeStr,
            string Dt, list<dag> pattern>
   : NeonInp<oops, iops, AddrModeNone, IndexModeNone, N2RegFrm, itin,
             OpcodeStr, Dt, "$Vd, $Vm", "", pattern> {
  bits<5> Vd;
  bits<5> Vm;
```
- EN: Declares reusable TableGen class `N2Vnp` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `N2Vnp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2395-2399
```tablegen
  // Encode instruction operands
  let Inst{22}    = Vd{4};
  let Inst{15-12} = Vd{3-0};
  let Inst{5}     = Vm{4};
  let Inst{3-0}   = Vm{3-0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2401-2410
```tablegen
  // Encode constant bits
  let Inst{27-23} = 0b00111;
  let Inst{21-20} = 0b11;
  let Inst{19-18} = op19_18;
  let Inst{17-16} = op17_16;
  let Inst{11} = 0;
  let Inst{10-8} = op10_8;
  let Inst{7} = op7;
  let Inst{6} = op6;
  let Inst{4} = 0;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2412-2413
```tablegen
  let DecoderNamespace = "NEON";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2415-2427
```tablegen
// Same as N2V except it doesn't have a datatype suffix.
class N2VX<bits<2> op24_23, bits<2> op21_20, bits<2> op19_18, bits<2> op17_16,
           bits<5> op11_7, bit op6, bit op4,
           dag oops, dag iops, InstrItinClass itin,
           string opc, string asm, string cstr, list<dag> pattern>
  : NDataXI<oops, iops, N2RegFrm, itin, opc, asm, cstr, pattern> {
  let Inst{24-23} = op24_23;
  let Inst{21-20} = op21_20;
  let Inst{19-18} = op19_18;
  let Inst{17-16} = op17_16;
  let Inst{11-7}  = op11_7;
  let Inst{6}     = op6;
  let Inst{4}     = op4;
```
- EN: Declares reusable TableGen class `N2VX` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `N2VX`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2429-2431
```tablegen
  // Instruction operands.
  bits<5> Vd;
  bits<5> Vm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2433-2437
```tablegen
  let Inst{15-12} = Vd{3-0};
  let Inst{22}    = Vd{4};
  let Inst{3-0}   = Vm{3-0};
  let Inst{5}     = Vm{4};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2439-2449
```tablegen
// NEON 2 vector register with immediate.
class N2VImm<bit op24, bit op23, bits<4> op11_8, bit op7, bit op6, bit op4,
             dag oops, dag iops, Format f, InstrItinClass itin,
             string opc, string dt, string asm, string cstr, list<dag> pattern>
  : NDataI<oops, iops, f, itin, opc, dt, asm, cstr, pattern> {
  let Inst{24}   = op24;
  let Inst{23}   = op23;
  let Inst{11-8} = op11_8;
  let Inst{7}    = op7;
  let Inst{6}    = op6;
  let Inst{4}    = op4;
```
- EN: Declares reusable TableGen class `N2VImm` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `N2VImm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2451-2454
```tablegen
  // Instruction operands.
  bits<5> Vd;
  bits<5> Vm;
  bits<6> SIMM;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2456-2461
```tablegen
  let Inst{15-12} = Vd{3-0};
  let Inst{22}    = Vd{4};
  let Inst{3-0}   = Vm{3-0};
  let Inst{5}     = Vm{4};
  let Inst{21-16} = SIMM{5-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2463-2463
```tablegen
// NEON 3 vector register format.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 2465-2476
```tablegen
class N3VCommon<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op6,
                bit op4, dag oops, dag iops, Format f, InstrItinClass itin,
                string opc, string dt, string asm, string cstr,
                list<dag> pattern>
  : NDataI<oops, iops, f, itin, opc, dt, asm, cstr, pattern> {
  let Inst{24}    = op24;
  let Inst{23}    = op23;
  let Inst{21-20} = op21_20;
  let Inst{11-8}  = op11_8;
  let Inst{6}     = op6;
  let Inst{4}     = op4;
}
```
- EN: Declares reusable TableGen class `N3VCommon` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `N3VCommon`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2478-2486
```tablegen
class N3V<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op6, bit op4,
          dag oops, dag iops, Format f, InstrItinClass itin,
          string opc, string dt, string asm, string cstr, list<dag> pattern>
  : N3VCommon<op24, op23, op21_20, op11_8, op6, op4,
              oops, iops, f, itin, opc, dt, asm, cstr, pattern> {
  // Instruction operands.
  bits<5> Vd;
  bits<5> Vn;
  bits<5> Vm;
```
- EN: Declares reusable TableGen class `N3V` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `N3V`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2488-2494
```tablegen
  let Inst{15-12} = Vd{3-0};
  let Inst{22}    = Vd{4};
  let Inst{19-16} = Vn{3-0};
  let Inst{7}     = Vn{4};
  let Inst{3-0}   = Vm{3-0};
  let Inst{5}     = Vm{4};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2496-2503
```tablegen
class N3Vnp<bits<5> op27_23, bits<2> op21_20, bits<4> op11_8, bit op6,
                bit op4, dag oops, dag iops,Format f, InstrItinClass itin,
                string OpcodeStr, string Dt, list<dag> pattern>
  : NeonInp<oops, iops, AddrModeNone, IndexModeNone, f, itin, OpcodeStr,
            Dt, "$Vd, $Vn, $Vm", "", pattern> {
  bits<5> Vd;
  bits<5> Vn;
  bits<5> Vm;
```
- EN: Declares reusable TableGen class `N3Vnp` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `N3Vnp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2505-2511
```tablegen
  // Encode instruction operands
  let Inst{22} = Vd{4};
  let Inst{15-12} = Vd{3-0};
  let Inst{19-16} = Vn{3-0};
  let Inst{7} = Vn{4};
  let Inst{5} = Vm{4};
  let Inst{3-0} = Vm{3-0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2513-2519
```tablegen
  // Encode constant bits
  let Inst{27-23} = op27_23;
  let Inst{21-20} = op21_20;
  let Inst{11-8}  = op11_8;
  let Inst{6}     = op6;
  let Inst{4}     = op4;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2521-2526
```tablegen
class N3VLane32<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op6,
                bit op4, dag oops, dag iops, Format f, InstrItinClass itin,
                string opc, string dt, string asm, string cstr,
                list<dag> pattern>
  : N3VCommon<op24, op23, op21_20, op11_8, op6, op4,
              oops, iops, f, itin, opc, dt, asm, cstr, pattern> {
```
- EN: Declares reusable TableGen class `N3VLane32` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `N3VLane32`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2528-2532
```tablegen
  // Instruction operands.
  bits<5> Vd;
  bits<5> Vn;
  bits<5> Vm;
  bit lane;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2534-2540
```tablegen
  let Inst{15-12} = Vd{3-0};
  let Inst{22}    = Vd{4};
  let Inst{19-16} = Vn{3-0};
  let Inst{7}     = Vn{4};
  let Inst{3-0}   = Vm{3-0};
  let Inst{5}     = lane;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2542-2547
```tablegen
class N3VLane16<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op6,
                bit op4, dag oops, dag iops, Format f, InstrItinClass itin,
                string opc, string dt, string asm, string cstr,
                list<dag> pattern>
  : N3VCommon<op24, op23, op21_20, op11_8, op6, op4,
              oops, iops, f, itin, opc, dt, asm, cstr, pattern> {
```
- EN: Declares reusable TableGen class `N3VLane16` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `N3VLane16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2549-2553
```tablegen
  // Instruction operands.
  bits<5> Vd;
  bits<5> Vn;
  bits<5> Vm;
  bits<2> lane;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2555-2562
```tablegen
  let Inst{15-12} = Vd{3-0};
  let Inst{22}    = Vd{4};
  let Inst{19-16} = Vn{3-0};
  let Inst{7}     = Vn{4};
  let Inst{2-0}   = Vm{2-0};
  let Inst{5}     = lane{1};
  let Inst{3}     = lane{0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2564-2575
```tablegen
// Same as N3V except it doesn't have a data type suffix.
class N3VX<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op6,
           bit op4,
           dag oops, dag iops, Format f, InstrItinClass itin,
           string opc, string asm, string cstr, list<dag> pattern>
  : NDataXI<oops, iops, f, itin, opc, asm, cstr, pattern> {
  let Inst{24}    = op24;
  let Inst{23}    = op23;
  let Inst{21-20} = op21_20;
  let Inst{11-8}  = op11_8;
  let Inst{6}     = op6;
  let Inst{4}     = op4;
```
- EN: Declares reusable TableGen class `N3VX` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `N3VX`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2577-2580
```tablegen
  // Instruction operands.
  bits<5> Vd;
  bits<5> Vn;
  bits<5> Vm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2582-2588
```tablegen
  let Inst{15-12} = Vd{3-0};
  let Inst{22}    = Vd{4};
  let Inst{19-16} = Vn{3-0};
  let Inst{7}     = Vn{4};
  let Inst{3-0}   = Vm{3-0};
  let Inst{5}     = Vm{4};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2590-2601
```tablegen
// NEON VMOVs between scalar and core registers.
class NVLaneOp<bits<8> opcod1, bits<4> opcod2, bits<2> opcod3,
               dag oops, dag iops, Format f, InstrItinClass itin,
               string opc, string dt, string asm, list<dag> pattern>
  : InstARM<AddrModeNone, 4, IndexModeNone, f, NeonDomain,
            "", itin> {
  let Inst{27-20} = opcod1;
  let Inst{11-8}  = opcod2;
  let Inst{6-5}   = opcod3;
  let Inst{4}     = 1;
  // A8.6.303, A8.6.328, A8.6.329
  let Inst{3-0}   = 0b0000;
```
- EN: Declares reusable TableGen class `NVLaneOp` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `NVLaneOp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2603-2607
```tablegen
  let OutOperandList = oops;
  let InOperandList = !con(iops, (ins pred:$p));
  let AsmString = !strconcat(opc, "${p}", ".", dt, "\t", asm);
  let Pattern = pattern;
  list<Predicate> Predicates = [HasNEON];
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2609-2610
```tablegen
  let PostEncoderMethod = "NEONThumb2DupPostEncoder";
  let DecoderNamespace = "NEONDup";
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2612-2615
```tablegen
  bits<5> V;
  bits<4> R;
  bits<4> p;
  bits<4> lane;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2617-2634
```tablegen
  let Inst{31-28} = p{3-0};
  let Inst{7}     = V{4};
  let Inst{19-16} = V{3-0};
  let Inst{15-12} = R{3-0};
}
class NVGetLane<bits<8> opcod1, bits<4> opcod2, bits<2> opcod3,
                dag oops, dag iops, InstrItinClass itin,
                string opc, string dt, string asm, list<dag> pattern>
  : NVLaneOp<opcod1, opcod2, opcod3, oops, iops, NGetLnFrm, itin,
             opc, dt, asm, pattern>;
class NVSetLane<bits<8> opcod1, bits<4> opcod2, bits<2> opcod3,
                dag oops, dag iops, InstrItinClass itin,
                string opc, string dt, string asm, list<dag> pattern>
  : NVLaneOp<opcod1, opcod2, opcod3, oops, iops, NSetLnFrm, itin,
             opc, dt, asm, pattern>;
class NVDup<bits<8> opcod1, bits<4> opcod2, bits<2> opcod3,
            dag oops, dag iops, InstrItinClass itin,
            string opc, string dt, string asm, list<dag> pattern>
```
- EN: Declares reusable TableGen class `NVGetLane` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `NVGetLane`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2635-2636
```tablegen
  : NVLaneOp<opcod1, opcod2, opcod3, oops, iops, NDupFrm, itin,
             opc, dt, asm, pattern>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2638-2648
```tablegen
// Vector Duplicate Lane (from scalar to all elements)
class NVDupLane<bits<4> op19_16, bit op6, dag oops, dag iops,
                InstrItinClass itin, string opc, string dt, string asm,
                list<dag> pattern>
  : NDataI<oops, iops, NVDupLnFrm, itin, opc, dt, asm, "", pattern> {
  let Inst{24-23} = 0b11;
  let Inst{21-20} = 0b11;
  let Inst{19-16} = op19_16;
  let Inst{11-7}  = 0b11000;
  let Inst{6}     = op6;
  let Inst{4}     = 0;
```
- EN: Declares reusable TableGen class `NVDupLane` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `NVDupLane`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2650-2651
```tablegen
  bits<5> Vd;
  bits<5> Vm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2653-2657
```tablegen
  let Inst{22}     = Vd{4};
  let Inst{15-12} = Vd{3-0};
  let Inst{5}     = Vm{4};
  let Inst{3-0} = Vm{3-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2659-2663
```tablegen
// NEONFPPat - Same as Pat<>, but requires that the compiler be using NEON
// for single-precision FP.
class NEONFPPat<dag pattern, dag result> : Pat<pattern, result> {
  list<Predicate> Predicates = [HasNEON,UseNEONForFP];
}
```
- EN: Declares reusable TableGen class `NEONFPPat` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `NEONFPPat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2665-2668
```tablegen
// VFP/NEON Instruction aliases for type suffices.
// Note: When EmitPriority == 1, the alias will be used for printing
class VFPDataTypeInstAlias<string opc, string dt, string asm, dag Result, bit EmitPriority = 0> :
  InstAlias<!strconcat(opc, dt, "\t", asm), Result, EmitPriority>, Requires<[HasFPRegs]>;
```
- EN: Declares reusable TableGen class `VFPDataTypeInstAlias` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `VFPDataTypeInstAlias`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2670-2676
```tablegen
// Note: When EmitPriority == 1, the alias will be used for printing
multiclass VFPDTAnyInstAlias<string opc, string asm, dag Result, bit EmitPriority = 0> {
  def : VFPDataTypeInstAlias<opc, ".8", asm, Result, EmitPriority>;
  def : VFPDataTypeInstAlias<opc, ".16", asm, Result, EmitPriority>;
  def : VFPDataTypeInstAlias<opc, ".32", asm, Result, EmitPriority>;
  def : VFPDataTypeInstAlias<opc, ".64", asm, Result, EmitPriority>;
}
```
- EN: Declares TableGen `multiclass VFPDTAnyInstAlias`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass VFPDTAnyInstAlias`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2678-2686
```tablegen
// Note: When EmitPriority == 1, the alias will be used for printing
multiclass NEONDTAnyInstAlias<string opc, string asm, dag Result, bit EmitPriority = 0> {
  let Predicates = [HasNEON] in {
  def : VFPDataTypeInstAlias<opc, ".8", asm, Result, EmitPriority>;
  def : VFPDataTypeInstAlias<opc, ".16", asm, Result, EmitPriority>;
  def : VFPDataTypeInstAlias<opc, ".32", asm, Result, EmitPriority>;
  def : VFPDataTypeInstAlias<opc, ".64", asm, Result, EmitPriority>;
}
}
```
- EN: Declares TableGen `multiclass NEONDTAnyInstAlias`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass NEONDTAnyInstAlias`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2688-2694
```tablegen
// The same alias classes using AsmPseudo instead, for the more complex
// stuff in NEON that InstAlias can't quite handle.
// Note that we can't use anonymous defm references here like we can
// above, as we care about the ultimate instruction enum names generated, unlike
// for instalias defs.
class NEONDataTypeAsmPseudoInst<string opc, string dt, string asm, dag iops> :
  AsmPseudoInst<!strconcat(opc, dt, "\t", asm), iops>, Requires<[HasNEON]>;
```
- EN: Declares reusable TableGen class `NEONDataTypeAsmPseudoInst` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `NEONDataTypeAsmPseudoInst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2696-2705
```tablegen
// Extension of NEON 3-vector data processing instructions in coprocessor 8
// encoding space, introduced in ARMv8.3-A.
class N3VCP8<bits<2> op24_23, bits<2> op21_20, bit op6, bit op4,
             dag oops, dag iops, InstrItinClass itin,
             string opc, string dt, string asm, string cstr, list<dag> pattern>
  : NeonInp<oops, iops, AddrModeNone, IndexModeNone, N3RegCplxFrm, itin, opc,
            dt, asm, cstr, pattern> {
  bits<5> Vd;
  bits<5> Vn;
  bits<5> Vm;
```
- EN: Declares reusable TableGen class `N3VCP8` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `N3VCP8`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2707-2709
```tablegen
  let DecoderNamespace = "VFPV8";
  // These have the same encodings in ARM and Thumb2
  let PostEncoderMethod = "";
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2711-2723
```tablegen
  let Inst{31-25} = 0b1111110;
  let Inst{24-23} = op24_23;
  let Inst{22}    = Vd{4};
  let Inst{21-20} = op21_20;
  let Inst{19-16} = Vn{3-0};
  let Inst{15-12} = Vd{3-0};
  let Inst{11-8}  = 0b1000;
  let Inst{7}     = Vn{4};
  let Inst{6}     = op6;
  let Inst{5}     = Vm{4};
  let Inst{4}     = op4;
  let Inst{3-0}   = Vm{3-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2725-2734
```tablegen
// Extension of NEON 2-vector-and-scalar data processing instructions in
// coprocessor 8 encoding space, introduced in ARMv8.3-A.
class N3VLaneCP8<bit op23, bits<2> op21_20, bit op6, bit op4,
             dag oops, dag iops, InstrItinClass itin,
             string opc, string dt, string asm, string cstr, list<dag> pattern>
  : NeonInp<oops, iops, AddrModeNone, IndexModeNone, N3RegCplxFrm, itin, opc,
            dt, asm, cstr, pattern> {
  bits<5> Vd;
  bits<5> Vn;
  bits<5> Vm;
```
- EN: Declares reusable TableGen class `N3VLaneCP8` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `N3VLaneCP8`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2736-2738
```tablegen
  let DecoderNamespace = "VFPV8";
  // These have the same encodings in ARM and Thumb2
  let PostEncoderMethod = "";
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2740-2752
```tablegen
  let Inst{31-24} = 0b11111110;
  let Inst{23}    = op23;
  let Inst{22}    = Vd{4};
  let Inst{21-20} = op21_20;
  let Inst{19-16} = Vn{3-0};
  let Inst{15-12} = Vd{3-0};
  let Inst{11-8}  = 0b1000;
  let Inst{7}     = Vn{4};
  let Inst{6}     = op6;
  // Bit 5 set by sub-classes
  let Inst{4}     = op4;
  let Inst{3-0}   = Vm{3-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2754-2765
```tablegen
// In Armv8.2-A, some NEON instructions are added that encode Vn and Vm
// differently:
//    if Q == ‘1’ then UInt(N:Vn) else UInt(Vn:N);
//    if Q == ‘1’ then UInt(M:Vm) else UInt(Vm:M);
// Class N3VCP8 above describes the Q=1 case, and this class the Q=0 case.
class N3VCP8Q0<bits<2> op24_23, bits<2> op21_20, bit op6, bit op4,
             dag oops, dag iops, InstrItinClass itin,
             string opc, string dt, string asm, string cstr, list<dag> pattern>
  : NeonInp<oops, iops, AddrModeNone, IndexModeNone, N3RegCplxFrm, itin, opc, dt, asm, cstr, pattern> {
  bits<5> Vd;
  bits<5> Vn;
  bits<5> Vm;
```
- EN: Declares reusable TableGen class `the` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `the`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2767-2769
```tablegen
  let DecoderNamespace = "VFPV8";
  // These have the same encodings in ARM and Thumb2
  let PostEncoderMethod = "";
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2771-2783
```tablegen
  let Inst{31-25} = 0b1111110;
  let Inst{24-23} = op24_23;
  let Inst{22}    = Vd{4};
  let Inst{21-20} = op21_20;
  let Inst{19-16} = Vn{4-1};
  let Inst{15-12} = Vd{3-0};
  let Inst{11-8}  = 0b1000;
  let Inst{7}     = Vn{0};
  let Inst{6}     = op6;
  let Inst{5}     = Vm{0};
  let Inst{4}     = op4;
  let Inst{3-0}   = Vm{4-1};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2785-2799
```tablegen
// Operand types for complex instructions
class ComplexRotationOperand<int Angle, int Remainder, string Type, string Diag>
  : AsmOperandClass {
  let PredicateMethod = "isComplexRotation<" # Angle # ", " # Remainder # ">";
  let DiagnosticString = "complex rotation must be " # Diag;
  let Name = "ComplexRotation" # Type;
}
def complexrotateop : Operand<i32> {
  let ParserMatchClass = ComplexRotationOperand<90, 0, "Even", "0, 90, 180 or 270">;
  let PrintMethod = "printComplexRotationOp<90, 0>";
}
def complexrotateopodd : Operand<i32> {
  let ParserMatchClass = ComplexRotationOperand<180, 90, "Odd", "90 or 270">;
  let PrintMethod = "printComplexRotationOp<180, 90>";
}
```
- EN: Declares reusable TableGen class `ComplexRotationOperand` for `ARMInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrFormats` 声明可复用的 TableGen 类 `ComplexRotationOperand`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2801-2809
```tablegen
def MveSaturateOperand : AsmOperandClass {
  let PredicateMethod = "isMveSaturateOp";
  let DiagnosticString = "saturate operand must be 48 or 64";
  let Name = "MveSaturate";
}
def saturateop : Operand<i32> {
  let ParserMatchClass = MveSaturateOperand;
  let PrintMethod = "printMveSaturateOp";
}
```
- EN: Defines TableGen record `MveSaturateOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MveSaturateOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2811-2819
```tablegen
// Data type suffix token aliases. Implements Table A7-3 in the ARM ARM.
def : TokenAlias<".s8", ".i8">;
def : TokenAlias<".u8", ".i8">;
def : TokenAlias<".s16", ".i16">;
def : TokenAlias<".u16", ".i16">;
def : TokenAlias<".s32", ".i32">;
def : TokenAlias<".u32", ".i32">;
def : TokenAlias<".s64", ".i64">;
def : TokenAlias<".u64", ".i64">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2821-2824
```tablegen
def : TokenAlias<".i8", ".8">;
def : TokenAlias<".i16", ".16">;
def : TokenAlias<".i32", ".32">;
def : TokenAlias<".i64", ".64">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2826-2827
```tablegen
def : TokenAlias<".p8", ".8">;
def : TokenAlias<".p16", ".16">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2829-2832
```tablegen
def : TokenAlias<".f32", ".32">;
def : TokenAlias<".f64", ".64">;
def : TokenAlias<".f", ".f32">;
def : TokenAlias<".d", ".f64">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

## Key Concepts / 关键概念

- EN: Primary role: instruction encoding formats and bitfield layout.
  - CN: 核心职责：指令编码格式与位域布局。
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
