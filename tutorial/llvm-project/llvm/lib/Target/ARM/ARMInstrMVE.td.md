# ARMInstrMVE.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMInstrMVE.td`
- Repository: `llvm-project`
- Purpose (EN): This file describes the ARM MVE instruction set.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMInstrMVE`，涵盖指令定义与目标操作码元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===-- ARMInstrMVE.td - MVE support for ARM ---------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the ARM MVE instruction set.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-19
```tablegen
// VPT condition mask
def vpt_mask : Operand<i32> {
  let PrintMethod = "printVPTMask";
  let ParserMatchClass = it_mask_asmoperand;
  let EncoderMethod = "getVPTMaskOpValue";
  let DecoderMethod = "DecodeVPTMaskOperand";
}
```
- EN: Defines TableGen record `vpt_mask` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `vpt_mask`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 21-29
```tablegen
// VPT/VCMP restricted predicate for sign invariant types
def pred_restricted_i_asmoperand : AsmOperandClass {
  let Name = "CondCodeRestrictedI";
  let RenderMethod = "addITCondCodeOperands";
  let PredicateMethod = "isITCondCodeRestrictedI";
  let ParserMethod = "parseITCondCode";
  let DiagnosticString = "condition code for sign-independent integer "#
                         "comparison must be EQ or NE";
}
```
- EN: Defines TableGen record `pred_restricted_i_asmoperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `pred_restricted_i_asmoperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 31-39
```tablegen
// VPT/VCMP restricted predicate for signed types
def pred_restricted_s_asmoperand : AsmOperandClass {
  let Name = "CondCodeRestrictedS";
  let RenderMethod = "addITCondCodeOperands";
  let PredicateMethod = "isITCondCodeRestrictedS";
  let ParserMethod = "parseITCondCode";
  let DiagnosticString = "condition code for signed integer "#
                         "comparison must be EQ, NE, LT, GT, LE or GE";
}
```
- EN: Defines TableGen record `pred_restricted_s_asmoperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `pred_restricted_s_asmoperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 41-49
```tablegen
// VPT/VCMP restricted predicate for unsigned types
def pred_restricted_u_asmoperand : AsmOperandClass {
  let Name = "CondCodeRestrictedU";
  let RenderMethod = "addITCondCodeOperands";
  let PredicateMethod = "isITCondCodeRestrictedU";
  let ParserMethod = "parseITCondCode";
  let DiagnosticString = "condition code for unsigned integer "#
                         "comparison must be EQ, NE, HS or HI";
}
```
- EN: Defines TableGen record `pred_restricted_u_asmoperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `pred_restricted_u_asmoperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 51-59
```tablegen
// VPT/VCMP restricted predicate for floating point
def pred_restricted_fp_asmoperand : AsmOperandClass {
  let Name = "CondCodeRestrictedFP";
  let RenderMethod = "addITCondCodeOperands";
  let PredicateMethod = "isITCondCodeRestrictedFP";
  let ParserMethod = "parseITCondCode";
  let DiagnosticString = "condition code for floating-point "#
                         "comparison must be EQ, NE, LT, GT, LE or GE";
}
```
- EN: Defines TableGen record `pred_restricted_fp_asmoperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `pred_restricted_fp_asmoperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 61-61
```tablegen
class VCMPPredicateOperand : Operand<i32>;
```
- EN: Declares reusable TableGen class `VCMPPredicateOperand` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `VCMPPredicateOperand`，通常用于抽象共享字段、谓词或编码结构。

### Lines 63-68
```tablegen
def pred_basic_i : VCMPPredicateOperand {
  let PrintMethod = "printMandatoryRestrictedPredicateOperand";
  let ParserMatchClass = pred_restricted_i_asmoperand;
  let DecoderMethod = "DecodeRestrictedIPredicateOperand";
  let EncoderMethod = "getRestrictedCondCodeOpValue";
}
```
- EN: Defines TableGen record `pred_basic_i` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `pred_basic_i`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 70-75
```tablegen
def pred_basic_u : VCMPPredicateOperand {
  let PrintMethod = "printMandatoryRestrictedPredicateOperand";
  let ParserMatchClass = pred_restricted_u_asmoperand;
  let DecoderMethod = "DecodeRestrictedUPredicateOperand";
  let EncoderMethod = "getRestrictedCondCodeOpValue";
}
```
- EN: Defines TableGen record `pred_basic_u` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `pred_basic_u`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 77-82
```tablegen
def pred_basic_s : VCMPPredicateOperand {
  let PrintMethod = "printMandatoryRestrictedPredicateOperand";
  let ParserMatchClass = pred_restricted_s_asmoperand;
  let DecoderMethod = "DecodeRestrictedSPredicateOperand";
  let EncoderMethod = "getRestrictedCondCodeOpValue";
}
```
- EN: Defines TableGen record `pred_basic_s` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `pred_basic_s`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 84-89
```tablegen
def pred_basic_fp : VCMPPredicateOperand {
  let PrintMethod = "printMandatoryRestrictedPredicateOperand";
  let ParserMatchClass = pred_restricted_fp_asmoperand;
  let DecoderMethod = "DecodeRestrictedFPPredicateOperand";
  let EncoderMethod = "getRestrictedCondCodeOpValue";
}
```
- EN: Defines TableGen record `pred_basic_fp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `pred_basic_fp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 91-98
```tablegen
// Register list operands for interleaving load/stores
def VecList2QAsmOperand : AsmOperandClass {
  let Name = "VecListTwoMQ";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addMVEVecListOperands";
  let DiagnosticString = "operand must be a list of two consecutive "#
                         "q-registers in range [q0,q7]";
}
```
- EN: Defines TableGen record `VecList2QAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecList2QAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 100-103
```tablegen
def VecList2Q : RegisterOperand<MQQPR, "printMVEVectorListTwoQ"> {
  let ParserMatchClass = VecList2QAsmOperand;
  let PrintMethod = "printMVEVectorList<2>";
}
```
- EN: Defines TableGen record `VecList2Q` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecList2Q`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 105-111
```tablegen
def VecList4QAsmOperand : AsmOperandClass {
  let Name = "VecListFourMQ";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addMVEVecListOperands";
  let DiagnosticString = "operand must be a list of four consecutive "#
                         "q-registers in range [q0,q7]";
}
```
- EN: Defines TableGen record `VecList4QAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecList4QAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 113-116
```tablegen
def VecList4Q : RegisterOperand<MQQQQPR, "printMVEVectorListFourQ"> {
  let ParserMatchClass = VecList4QAsmOperand;
  let PrintMethod = "printMVEVectorList<4>";
}
```
- EN: Defines TableGen record `VecList4Q` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecList4Q`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 118-123
```tablegen
// taddrmode_imm7  := reg[r0-r7] +/- (imm7 << shift)
class TMemImm7ShiftOffsetAsmOperand<int shift> : AsmOperandClass {
  let Name = "TMemImm7Shift"#shift#"Offset";
  let PredicateMethod = "isMemImm7ShiftedOffset<"#shift#",ARM::tGPRRegClassID>";
  let RenderMethod = "addMemImmOffsetOperands";
}
```
- EN: Declares reusable TableGen class `TMemImm7ShiftOffsetAsmOperand` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `TMemImm7ShiftOffsetAsmOperand`，通常用于抽象共享字段、谓词或编码结构。

### Lines 125-134
```tablegen
class taddrmode_imm7<int shift> : MemOperand,
    ComplexPattern<i32, 2, "SelectTAddrModeImm7<"#shift#">", []>  {
  let ParserMatchClass = TMemImm7ShiftOffsetAsmOperand<shift>;
  // They are printed the same way as the T2 imm8 version
  let PrintMethod = "printT2AddrModeImm8Operand<false>";
  // This can also be the same as the T2 version.
  let EncoderMethod = "getT2AddrModeImmOpValue<7,"#shift#">";
  let DecoderMethod = "DecodeTAddrModeImm7<"#shift#">";
  let MIOperandInfo = (ops tGPR:$base, i32imm:$offsimm);
}
```
- EN: Declares reusable TableGen class `taddrmode_imm7` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `taddrmode_imm7`，通常用于抽象共享字段、谓词或编码结构。

### Lines 136-142
```tablegen
// t2addrmode_imm7  := reg +/- (imm7)
class MemImm7ShiftOffsetAsmOperand<int shift> : AsmOperandClass {
  let Name = "MemImm7Shift"#shift#"Offset";
  let PredicateMethod = "isMemImm7ShiftedOffset<" # shift #
                        ",ARM::GPRnopcRegClassID>";
  let RenderMethod = "addMemImmOffsetOperands";
}
```
- EN: Declares reusable TableGen class `MemImm7ShiftOffsetAsmOperand` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MemImm7ShiftOffsetAsmOperand`，通常用于抽象共享字段、谓词或编码结构。

### Lines 144-154
```tablegen
def MemImm7Shift0OffsetAsmOperand : MemImm7ShiftOffsetAsmOperand<0>;
def MemImm7Shift1OffsetAsmOperand : MemImm7ShiftOffsetAsmOperand<1>;
def MemImm7Shift2OffsetAsmOperand : MemImm7ShiftOffsetAsmOperand<2>;
class T2AddrMode_Imm7<int shift> : MemOperand,
      ComplexPattern<i32, 2, "SelectT2AddrModeImm7<"#shift#">", []> {
  let EncoderMethod = "getT2AddrModeImmOpValue<7,"#shift#">";
  let DecoderMethod = "DecodeT2AddrModeImm7<"#shift#", 0>";
  let ParserMatchClass =
    !cast<AsmOperandClass>("MemImm7Shift"#shift#"OffsetAsmOperand");
  let MIOperandInfo = (ops GPRnopc:$base, i32imm:$offsimm);
}
```
- EN: Declares reusable TableGen class `T2AddrMode_Imm7` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `T2AddrMode_Imm7`，通常用于抽象共享字段、谓词或编码结构。

### Lines 156-159
```tablegen
class t2addrmode_imm7<int shift> : T2AddrMode_Imm7<shift> {
  // They are printed the same way as the imm8 version
  let PrintMethod = "printT2AddrModeImm8Operand<false>";
}
```
- EN: Declares reusable TableGen class `t2addrmode_imm7` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `t2addrmode_imm7`，通常用于抽象共享字段、谓词或编码结构。

### Lines 161-166
```tablegen
class MemImm7ShiftOffsetWBAsmOperand<int shift> : AsmOperandClass {
  let Name = "MemImm7Shift"#shift#"OffsetWB";
  let PredicateMethod = "isMemImm7ShiftedOffset<" # shift #
                        ",ARM::rGPRRegClassID>";
  let RenderMethod = "addMemImmOffsetOperands";
}
```
- EN: Declares reusable TableGen class `MemImm7ShiftOffsetWBAsmOperand` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MemImm7ShiftOffsetWBAsmOperand`，通常用于抽象共享字段、谓词或编码结构。

### Lines 168-170
```tablegen
def MemImm7Shift0OffsetWBAsmOperand : MemImm7ShiftOffsetWBAsmOperand<0>;
def MemImm7Shift1OffsetWBAsmOperand : MemImm7ShiftOffsetWBAsmOperand<1>;
def MemImm7Shift2OffsetWBAsmOperand : MemImm7ShiftOffsetWBAsmOperand<2>;
```
- EN: Defines TableGen record `MemImm7Shift0OffsetWBAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MemImm7Shift0OffsetWBAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 172-179
```tablegen
class t2addrmode_imm7_pre<int shift> : T2AddrMode_Imm7<shift> {
  // They are printed the same way as the imm8 version
  let PrintMethod = "printT2AddrModeImm8Operand<true>";
  let ParserMatchClass =
    !cast<AsmOperandClass>("MemImm7Shift"#shift#"OffsetWBAsmOperand");
  let DecoderMethod = "DecodeT2AddrModeImm7<"#shift#", 1>";
  let MIOperandInfo = (ops rGPR:$base, i32imm:$offsim);
}
```
- EN: Declares reusable TableGen class `t2addrmode_imm7_pre` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `t2addrmode_imm7_pre`，通常用于抽象共享字段、谓词或编码结构。

### Lines 181-185
```tablegen
class t2am_imm7shiftOffsetAsmOperand<int shift>
  : AsmOperandClass { let Name = "Imm7Shift"#shift; }
def t2am_imm7shift0OffsetAsmOperand : t2am_imm7shiftOffsetAsmOperand<0>;
def t2am_imm7shift1OffsetAsmOperand : t2am_imm7shiftOffsetAsmOperand<1>;
def t2am_imm7shift2OffsetAsmOperand : t2am_imm7shiftOffsetAsmOperand<2>;
```
- EN: Declares reusable TableGen class `t2am_imm7shiftOffsetAsmOperand` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `t2am_imm7shiftOffsetAsmOperand`，通常用于抽象共享字段、谓词或编码结构。

### Lines 187-197
```tablegen
class t2am_imm7_offset<int shift>
    : MemOperand,
      ComplexPattern<i32, 1, "SelectT2AddrModeImm7Offset<"#shift#">"> {
  // They are printed the same way as the imm8 version
  let PrintMethod = "printT2AddrModeImm8OffsetOperand";
  let ParserMatchClass =
    !cast<AsmOperandClass>("t2am_imm7shift"#shift#"OffsetAsmOperand");
  let EncoderMethod = "getT2ScaledImmOpValue<7,"#shift#">";
  let DecoderMethod = "DecodeT2Imm7<"#shift#">";
  let WantsRoot = true;
}
```
- EN: Declares reusable TableGen class `t2am_imm7_offset` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `t2am_imm7_offset`，通常用于抽象共享字段、谓词或编码结构。

### Lines 199-204
```tablegen
// Operands for gather/scatter loads of the form [Rbase, Qoffsets]
class MemRegRQOffsetAsmOperand<int shift> : AsmOperandClass {
  let Name = "MemRegRQS"#shift#"Offset";
  let PredicateMethod = "isMemRegRQOffset<"#shift#">";
  let RenderMethod = "addMemRegRQOffsetOperands";
}
```
- EN: Declares reusable TableGen class `MemRegRQOffsetAsmOperand` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MemRegRQOffsetAsmOperand`，通常用于抽象共享字段、谓词或编码结构。

### Lines 206-209
```tablegen
def MemRegRQS0OffsetAsmOperand : MemRegRQOffsetAsmOperand<0>;
def MemRegRQS1OffsetAsmOperand : MemRegRQOffsetAsmOperand<1>;
def MemRegRQS2OffsetAsmOperand : MemRegRQOffsetAsmOperand<2>;
def MemRegRQS3OffsetAsmOperand : MemRegRQOffsetAsmOperand<3>;
```
- EN: Defines TableGen record `MemRegRQS0OffsetAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MemRegRQS0OffsetAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 211-219
```tablegen
// mve_addr_rq_shift  := reg + vreg{ << UXTW #shift}
class mve_addr_rq_shift<int shift> : MemOperand {
  let EncoderMethod = "getMveAddrModeRQOpValue";
  let PrintMethod = "printMveAddrModeRQOperand<"#shift#">";
  let ParserMatchClass =
    !cast<AsmOperandClass>("MemRegRQS"#shift#"OffsetAsmOperand");
  let DecoderMethod = "DecodeMveAddrModeRQ";
  let MIOperandInfo = (ops GPRnopc:$base, MQPR:$offsreg);
}
```
- EN: Declares reusable TableGen class `mve_addr_rq_shift` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `mve_addr_rq_shift`，通常用于抽象共享字段、谓词或编码结构。

### Lines 221-225
```tablegen
class MemRegQOffsetAsmOperand<int shift> : AsmOperandClass {
  let Name = "MemRegQS"#shift#"Offset";
  let PredicateMethod = "isMemRegQOffset<"#shift#">";
  let RenderMethod = "addMemImmOffsetOperands";
}
```
- EN: Declares reusable TableGen class `MemRegQOffsetAsmOperand` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MemRegQOffsetAsmOperand`，通常用于抽象共享字段、谓词或编码结构。

### Lines 227-228
```tablegen
def MemRegQS2OffsetAsmOperand : MemRegQOffsetAsmOperand<2>;
def MemRegQS3OffsetAsmOperand : MemRegQOffsetAsmOperand<3>;
```
- EN: Defines TableGen record `MemRegQS2OffsetAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MemRegQS2OffsetAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 230-239
```tablegen
// mve_addr_q_shift  := vreg {+ #imm7s2/4}
class mve_addr_q_shift<int shift> : MemOperand {
  let EncoderMethod = "getMveAddrModeQOpValue<"#shift#">";
  // Can be printed same way as other reg + imm operands
  let PrintMethod = "printT2AddrModeImm8Operand<false>";
  let ParserMatchClass =
    !cast<AsmOperandClass>("MemRegQS"#shift#"OffsetAsmOperand");
  let DecoderMethod = "DecodeMveAddrModeQ<"#shift#">";
  let MIOperandInfo = (ops MQPR:$base, i32imm:$imm);
}
```
- EN: Declares reusable TableGen class `mve_addr_q_shift` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `mve_addr_q_shift`，通常用于抽象共享字段、谓词或编码结构。

### Lines 241-248
```tablegen
// A family of classes wrapping up information about the vector types
// used by MVE.
class MVEVectorVTInfo<ValueType vec, ValueType dblvec,
                      ValueType pred, ValueType dblpred,
                      bits<2> size, string suffixletter, bit unsigned> {
  // The LLVM ValueType representing the vector, so we can use it in
  // ISel patterns.
  ValueType Vec = vec;
```
- EN: Declares reusable TableGen class `MVEVectorVTInfo` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVEVectorVTInfo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 250-253
```tablegen
  // The LLVM ValueType representing a vector with elements double the size
  // of those in Vec, so we can use it in ISel patterns. It is up to the
  // invoker of this class to ensure that this is a correct choice.
  ValueType DblVec = dblvec;
```
- EN: Declares reusable TableGen class `to` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `to`，通常用于抽象共享字段、谓词或编码结构。

### Lines 255-258
```tablegen
  // An LLVM ValueType representing a corresponding vector of
  // predicate bits, for use in ISel patterns that handle an IR
  // intrinsic describing the predicated form of the instruction.
  ValueType Pred = pred;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 260-261
```tablegen
  // Same as Pred but for DblVec rather than Vec.
  ValueType DblPred = dblpred;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 263-266
```tablegen
  // The most common representation of the vector element size in MVE
  // instruction encodings: a 2-bit value V representing an (8<<V)-bit
  // vector element.
  bits<2> Size = size;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 268-270
```tablegen
  // For vectors explicitly mentioning a signedness of integers: 0 for
  // signed and 1 for unsigned. For anything else, undefined.
  bit Unsigned = unsigned;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 272-273
```tablegen
  // The number of bits in a vector element, in integer form.
  int LaneBits = !shl(8, Size);
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 275-279
```tablegen
  // The suffix used in assembly language on an instruction operating
  // on this lane if it only cares about number of bits.
  string BitsSuffix = !if(!eq(suffixletter, "p"),
                          !if(!eq(unsigned, 0b0), "8", "16"),
                          !cast<string>(LaneBits));
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 281-282
```tablegen
  // The suffix used on an instruction that mentions the whole type.
  string Suffix = suffixletter # BitsSuffix;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 284-286
```tablegen
  // The letter part of the suffix only.
  string SuffixLetter = suffixletter;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 288-292
```tablegen
// Integer vector types that don't treat signed and unsigned differently.
def MVE_v16i8 : MVEVectorVTInfo<v16i8, v8i16, v16i1, v8i1, 0b00, "i", ?>;
def MVE_v8i16 : MVEVectorVTInfo<v8i16, v4i32, v8i1,  v4i1, 0b01, "i", ?>;
def MVE_v4i32 : MVEVectorVTInfo<v4i32, v2i64, v4i1,  v2i1, 0b10, "i", ?>;
def MVE_v2i64 : MVEVectorVTInfo<v2i64, ?,     v2i1,  ?,    0b11, "i", ?>;
```
- EN: Defines TableGen record `MVE_v16i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_v16i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 294-304
```tablegen
// Explicitly signed and unsigned integer vectors. They map to the
// same set of LLVM ValueTypes as above, but are represented
// differently in assembly and instruction encodings.
def MVE_v16s8 : MVEVectorVTInfo<v16i8, v8i16, v16i1, v8i1, 0b00, "s", 0b0>;
def MVE_v8s16 : MVEVectorVTInfo<v8i16, v4i32, v8i1,  v4i1, 0b01, "s", 0b0>;
def MVE_v4s32 : MVEVectorVTInfo<v4i32, v2i64, v4i1,  v2i1, 0b10, "s", 0b0>;
def MVE_v2s64 : MVEVectorVTInfo<v2i64, ?,     v2i1,  ?,    0b11, "s", 0b0>;
def MVE_v16u8 : MVEVectorVTInfo<v16i8, v8i16, v16i1, v8i1, 0b00, "u", 0b1>;
def MVE_v8u16 : MVEVectorVTInfo<v8i16, v4i32, v8i1,  v4i1, 0b01, "u", 0b1>;
def MVE_v4u32 : MVEVectorVTInfo<v4i32, v2i64, v4i1,  v2i1, 0b10, "u", 0b1>;
def MVE_v2u64 : MVEVectorVTInfo<v2i64, ?,     v2i1,  ?,    0b11, "u", 0b1>;
```
- EN: Defines TableGen record `MVE_v16s8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_v16s8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 306-309
```tablegen
// FP vector types.
def MVE_v8f16 : MVEVectorVTInfo<v8f16, v4f32, v8i1,  v4i1, 0b01, "f", ?>;
def MVE_v4f32 : MVEVectorVTInfo<v4f32, v2f64, v4i1,  v2i1, 0b10, "f", ?>;
def MVE_v2f64 : MVEVectorVTInfo<v2f64, ?,     v2i1,  ?,    0b11, "f", ?>;
```
- EN: Defines TableGen record `MVE_v8f16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_v8f16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 311-313
```tablegen
// Polynomial vector types.
def MVE_v16p8 : MVEVectorVTInfo<v16i8, v8i16, v16i1, v8i1, 0b11, "p", 0b0>;
def MVE_v8p16 : MVEVectorVTInfo<v8i16, v4i32, v8i1,  v4i1, 0b11, "p", 0b1>;
```
- EN: Defines TableGen record `MVE_v16p8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_v16p8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 315-320
```tablegen
multiclass MVE_TwoOpPattern<MVEVectorVTInfo VTI, SDPatternOperator Op, Intrinsic PredInt,
                            dag PredOperands, Instruction Inst,
                            SDPatternOperator IdentityVec = null_frag> {
  // Unpredicated
  def : Pat<(VTI.Vec (Op (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn))),
            (VTI.Vec (Inst (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn)))>;
```
- EN: Declares TableGen `multiclass MVE_TwoOpPattern`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_TwoOpPattern`，它是一个可复用模板，可展开为多个相关记录。

### Lines 322-330
```tablegen
  // Predicated with select
  if !ne(VTI.Size, 0b11) then {
    def : Pat<(VTI.Vec (vselect (VTI.Pred VCCR:$mask),
                                (VTI.Vec (Op (VTI.Vec MQPR:$Qm),
                                             (VTI.Vec MQPR:$Qn))),
                                (VTI.Vec MQPR:$inactive))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn),
                              ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg,
                              (VTI.Vec MQPR:$inactive)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 332-340
```tablegen
    // Optionally with the select folded through the op
    def : Pat<(VTI.Vec (Op (VTI.Vec MQPR:$Qm),
                           (VTI.Vec (vselect (VTI.Pred VCCR:$mask),
                                             (VTI.Vec MQPR:$Qn),
                                             (VTI.Vec IdentityVec))))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn),
                              ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg,
                              (VTI.Vec MQPR:$Qm)))>;
  }
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 342-349
```tablegen
  // Predicated with intrinsic
  def : Pat<(VTI.Vec !con((PredInt (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn)),
                          PredOperands,
                          (? (VTI.Pred VCCR:$mask), (VTI.Vec MQPR:$inactive)))),
            (VTI.Vec (Inst (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn),
                            ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg,
                            (VTI.Vec MQPR:$inactive)))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 351-356
```tablegen
multiclass MVE_TwoOpPatternDup<MVEVectorVTInfo VTI, SDPatternOperator Op, Intrinsic PredInt,
                               dag PredOperands, Instruction Inst,
                               SDPatternOperator IdentityVec = null_frag> {
  // Unpredicated
  def : Pat<(VTI.Vec (Op (VTI.Vec MQPR:$Qm), (VTI.Vec (ARMvdup rGPR:$Rn)))),
            (VTI.Vec (Inst (VTI.Vec MQPR:$Qm), rGPR:$Rn))>;
```
- EN: Declares TableGen `multiclass MVE_TwoOpPatternDup`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_TwoOpPatternDup`，它是一个可复用模板，可展开为多个相关记录。

### Lines 358-366
```tablegen
  // Predicated with select
  if !ne(VTI.Size, 0b11) then {
    def : Pat<(VTI.Vec (vselect (VTI.Pred VCCR:$mask),
                                (VTI.Vec (Op (VTI.Vec MQPR:$Qm),
                                             (VTI.Vec (ARMvdup rGPR:$Rn)))),
                                (VTI.Vec MQPR:$inactive))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qm), rGPR:$Rn,
                              ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg,
                              (VTI.Vec MQPR:$inactive)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 368-376
```tablegen
    // Optionally with the select folded through the op
    def : Pat<(VTI.Vec (Op (VTI.Vec MQPR:$Qm),
                           (VTI.Vec (vselect (VTI.Pred VCCR:$mask),
                                             (ARMvdup rGPR:$Rn),
                                             (VTI.Vec IdentityVec))))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qm), rGPR:$Rn,
                              ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg,
                              (VTI.Vec MQPR:$Qm)))>;
  }
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 378-385
```tablegen
  // Predicated with intrinsic
  def : Pat<(VTI.Vec !con((PredInt (VTI.Vec MQPR:$Qm), (VTI.Vec (ARMvdup rGPR:$Rn))),
                          PredOperands,
                          (? (VTI.Pred VCCR:$mask), (VTI.Vec MQPR:$inactive)))),
            (VTI.Vec (Inst (VTI.Vec MQPR:$Qm), rGPR:$Rn,
                            ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg,
                            (VTI.Vec MQPR:$inactive)))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 387-401
```tablegen
def vadd : PatFrags<(ops node:$lhs, node:$rhs),
                    [(fadd node:$lhs, node:$rhs),
                     (int_arm_mve_vadd node:$lhs, node:$rhs)]>;
def vsub : PatFrags<(ops node:$lhs, node:$rhs),
                    [(fsub node:$lhs, node:$rhs),
                     (int_arm_mve_vsub node:$lhs, node:$rhs)]>;
def vmul : PatFrags<(ops node:$lhs, node:$rhs),
                    [(fmul node:$lhs, node:$rhs),
                     (int_arm_mve_vmul node:$lhs, node:$rhs)]>;
def vminnm : PatFrags<(ops node:$lhs, node:$rhs),
                    [(fminnum node:$lhs, node:$rhs),
                     (int_arm_mve_vminnm node:$lhs, node:$rhs)]>;
def vmaxnm : PatFrags<(ops node:$lhs, node:$rhs),
                    [(fmaxnum node:$lhs, node:$rhs),
                     (int_arm_mve_vmaxnm node:$lhs, node:$rhs)]>;
```
- EN: Defines TableGen record `vadd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `vadd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 403-420
```tablegen
def fcmpeq : PatFrags<(ops node:$a, node:$b),
                      [(ARMvcmp node:$a, node:$b, ARMCCeq),
                       (int_arm_mve_cmp_eq node:$a, node:$b)]>;
def fcmpne : PatFrags<(ops node:$a, node:$b),
                      [(ARMvcmp node:$a, node:$b, ARMCCne),
                       (int_arm_mve_cmp_ne node:$a, node:$b)]>;
def fcmpge : PatFrags<(ops node:$a, node:$b),
                      [(ARMvcmp node:$a, node:$b, ARMCCge),
                       (int_arm_mve_cmp_ge node:$a, node:$b)]>;
def fcmplt : PatFrags<(ops node:$a, node:$b),
                      [(ARMvcmp node:$a, node:$b, ARMCClt),
                       (int_arm_mve_cmp_lt node:$a, node:$b)]>;
def fcmpgt : PatFrags<(ops node:$a, node:$b),
                      [(ARMvcmp node:$a, node:$b, ARMCCgt),
                       (int_arm_mve_cmp_gt node:$a, node:$b)]>;
def fcmple : PatFrags<(ops node:$a, node:$b),
                      [(ARMvcmp node:$a, node:$b, ARMCCle),
                       (int_arm_mve_cmp_le node:$a, node:$b)]>;
```
- EN: Defines TableGen record `fcmpeq` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `fcmpeq`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 422-422
```tablegen
// --------- Start of base classes for the instructions themselves
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 424-432
```tablegen
class MVE_MI<dag oops, dag iops, InstrItinClass itin, string asm,
             string ops, string cstr, bits<2> vecsize, list<dag> pattern>
  : Thumb2XI<oops, iops, AddrModeNone, 4, itin, !strconcat(asm, "\t", ops), cstr,
             pattern>,
    Requires<[HasMVEInt]> {
  let D = MVEDomain;
  let DecoderNamespace = "MVE";
  let VecSize = vecsize;
}
```
- EN: Declares reusable TableGen class `MVE_MI` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_MI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 434-450
```tablegen
// MVE_p is used for most predicated instructions, to add the cluster
// of input operands that provides the VPT suffix (none, T or E) and
// the input predicate register.
class MVE_p<dag oops, dag iops, InstrItinClass itin, string iname,
            string suffix, string ops, vpred_ops vpred, string cstr,
            bits<2> vecsize, list<dag> pattern=[]>
  : MVE_MI<oops, !con(iops, (ins vpred:$vp)), itin,
           // If the instruction has a suffix, like vadd.f32, then the
           // VPT predication suffix goes before the dot, so the full
           // name has to be "vadd${vp}.f32".
           !strconcat(iname, "${vp}",
                      !if(!eq(suffix, ""), "", !strconcat(".", suffix))),
           ops, !strconcat(cstr, vpred.vpred_constraint), vecsize, pattern> {
  bits<0> vp;
  let Inst{31-29} = 0b111;
  let Inst{27-26} = 0b11;
}
```
- EN: Declares reusable TableGen class `MVE_p` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_p`，通常用于抽象共享字段、谓词或编码结构。

### Lines 452-457
```tablegen
class MVE_f<dag oops, dag iops, InstrItinClass itin, string iname,
            string suffix, string ops, vpred_ops vpred, string cstr,
            bits<2> vecsize, list<dag> pattern=[]>
  : MVE_p<oops, iops, itin, iname, suffix, ops, vpred, cstr, vecsize, pattern> {
  let Predicates = [HasMVEFloat];
}
```
- EN: Declares reusable TableGen class `MVE_f` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_f`，通常用于抽象共享字段、谓词或编码结构。

### Lines 459-466
```tablegen
class MVE_MI_with_pred<dag oops, dag iops, InstrItinClass itin, string asm,
                       string ops, string cstr, list<dag> pattern>
  : Thumb2I<oops, iops, AddrModeNone, 4, itin, asm, !strconcat("\t", ops), cstr,
             pattern>,
    Requires<[HasV8_1MMainline, HasMVEInt]> {
  let D = MVEDomain;
  let DecoderNamespace = "MVE";
}
```
- EN: Declares reusable TableGen class `MVE_MI_with_pred` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_MI_with_pred`，通常用于抽象共享字段、谓词或编码结构。

### Lines 468-477
```tablegen
class MVE_VMOV_lane_base<dag oops, dag iops, InstrItinClass itin, string asm,
                         string suffix, string ops, string cstr,
                         list<dag> pattern>
  : Thumb2I<oops, iops, AddrModeNone, 4, itin, asm,
            !if(!eq(suffix, ""), "", "." # suffix) # "\t" # ops,
            cstr, pattern>,
    Requires<[HasV8_1MMainline, HasMVEInt]> {
  let D = MVEDomain;
  let DecoderNamespace = "MVE";
}
```
- EN: Declares reusable TableGen class `MVE_VMOV_lane_base` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VMOV_lane_base`，通常用于抽象共享字段、谓词或编码结构。

### Lines 479-485
```tablegen
class MVE_ScalarShift<string iname, dag oops, dag iops, string asm, string cstr,
            list<dag> pattern=[]>
  : MVE_MI_with_pred<oops, iops, NoItinerary, iname, asm, cstr, pattern> {
  let Inst{31-20} = 0b111010100101;
  let Inst{8} = 0b1;
  let validForTailPredication=1;
}
```
- EN: Declares reusable TableGen class `MVE_ScalarShift` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_ScalarShift`，通常用于抽象共享字段、谓词或编码结构。

### Lines 487-490
```tablegen
class MVE_ScalarShiftSingleReg<string iname, dag iops, string asm, string cstr,
                    list<dag> pattern=[]>
  : MVE_ScalarShift<iname, (outs rGPR:$RdaDest), iops, asm, cstr, pattern> {
  bits<4> RdaDest;
```
- EN: Declares reusable TableGen class `MVE_ScalarShiftSingleReg` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_ScalarShiftSingleReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 492-493
```tablegen
  let Inst{19-16} = RdaDest{3-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 495-501
```tablegen
class MVE_ScalarShiftSRegImm<string iname, bits<2> op5_4>
  : MVE_ScalarShiftSingleReg<iname, (ins rGPR:$RdaSrc, long_shift:$imm),
                     "$RdaSrc, $imm", "$RdaDest = $RdaSrc",
                     [(set rGPR:$RdaDest,
                          (i32 (!cast<Intrinsic>("int_arm_mve_" # iname)
                                    (i32 rGPR:$RdaSrc), (i32 imm:$imm))))]> {
  bits<5> imm;
```
- EN: Declares reusable TableGen class `MVE_ScalarShiftSRegImm` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_ScalarShiftSRegImm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 503-509
```tablegen
  let Inst{15} = 0b0;
  let Inst{14-12} = imm{4-2};
  let Inst{11-8} = 0b1111;
  let Inst{7-6} = imm{1-0};
  let Inst{5-4} = op5_4{1-0};
  let Inst{3-0} = 0b1111;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 511-514
```tablegen
def MVE_SQSHL : MVE_ScalarShiftSRegImm<"sqshl", 0b11>;
def MVE_SRSHR : MVE_ScalarShiftSRegImm<"srshr", 0b10>;
def MVE_UQSHL : MVE_ScalarShiftSRegImm<"uqshl", 0b00>;
def MVE_URSHR : MVE_ScalarShiftSRegImm<"urshr", 0b01>;
```
- EN: Defines TableGen record `MVE_SQSHL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_SQSHL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 516-522
```tablegen
class MVE_ScalarShiftSRegReg<string iname, bits<2> op5_4>
  : MVE_ScalarShiftSingleReg<iname, (ins rGPR:$RdaSrc, rGPR:$Rm),
                     "$RdaSrc, $Rm", "@earlyclobber $RdaDest,$RdaDest = $RdaSrc",
                     [(set rGPR:$RdaDest,
                         (i32 (!cast<Intrinsic>("int_arm_mve_" # iname)
                                   (i32 rGPR:$RdaSrc), (i32 rGPR:$Rm))))]> {
  bits<4> Rm;
```
- EN: Declares reusable TableGen class `MVE_ScalarShiftSRegReg` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_ScalarShiftSRegReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 524-528
```tablegen
  let Inst{15-12} = Rm{3-0};
  let Inst{11-8} = 0b1111;
  let Inst{7-6} = 0b00;
  let Inst{5-4} = op5_4{1-0};
  let Inst{3-0} = 0b1101;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 530-531
```tablegen
  let Unpredictable{8-6} = 0b111;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 533-534
```tablegen
def MVE_SQRSHR : MVE_ScalarShiftSRegReg<"sqrshr", 0b10>;
def MVE_UQRSHL : MVE_ScalarShiftSRegReg<"uqrshl", 0b00>;
```
- EN: Defines TableGen record `MVE_SQRSHR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_SQRSHR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 536-541
```tablegen
class MVE_ScalarShiftDoubleReg<string iname, dag iops, string asm,
                               string cstr, list<dag> pattern=[]>
  : MVE_ScalarShift<iname, (outs tGPREven:$RdaLo, tGPROdd:$RdaHi),
                    iops, asm, cstr, pattern> {
  bits<4> RdaLo;
  bits<4> RdaHi;
```
- EN: Declares reusable TableGen class `MVE_ScalarShiftDoubleReg` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_ScalarShiftDoubleReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 543-544
```tablegen
  let Inst{19-17} = RdaLo{3-1};
  let Inst{11-9} = RdaHi{3-1};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 546-547
```tablegen
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 549-555
```tablegen
class MVE_ScalarShiftDRegImm<string iname, bits<2> op5_4, bit op16,
                             list<dag> pattern=[]>
  : MVE_ScalarShiftDoubleReg<
      iname, (ins tGPREven:$RdaLo_src, tGPROdd:$RdaHi_src, long_shift:$imm),
      "$RdaLo, $RdaHi, $imm", "$RdaLo = $RdaLo_src,$RdaHi = $RdaHi_src",
      pattern> {
  bits<5> imm;
```
- EN: Declares reusable TableGen class `MVE_ScalarShiftDRegImm` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_ScalarShiftDRegImm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 557-563
```tablegen
  let Inst{16} = op16;
  let Inst{15} = 0b0;
  let Inst{14-12} = imm{4-2};
  let Inst{7-6} = imm{1-0};
  let Inst{5-4} = op5_4{1-0};
  let Inst{3-0} = 0b1111;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 565-571
```tablegen
class MVE_ScalarShiftDRegRegBase<string iname, dag iops, string asm,
                                 bit op5, bit op16, list<dag> pattern=[]>
  : MVE_ScalarShiftDoubleReg<
     iname, iops, asm, "@earlyclobber $RdaHi,@earlyclobber $RdaLo,"
                       "$RdaLo = $RdaLo_src,$RdaHi = $RdaHi_src",
     pattern> {
  bits<4> Rm;
```
- EN: Declares reusable TableGen class `MVE_ScalarShiftDRegRegBase` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_ScalarShiftDRegRegBase`，通常用于抽象共享字段、谓词或编码结构。

### Lines 573-578
```tablegen
  let Inst{16} = op16;
  let Inst{15-12} = Rm{3-0};
  let Inst{6} = 0b0;
  let Inst{5} = op5;
  let Inst{4} = 0b0;
  let Inst{3-0} = 0b1101;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 580-586
```tablegen
  // Custom decoder method because of the following overlapping encodings:
  // ASRL and SQRSHR
  // LSLL and UQRSHL
  // SQRSHRL and SQRSHR
  // UQRSHLL and UQRSHL
  let DecoderMethod = "DecodeMVEOverlappingLongShift";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 588-591
```tablegen
class MVE_ScalarShiftDRegReg<string iname, bit op5, list<dag> pattern=[]>
  : MVE_ScalarShiftDRegRegBase<
     iname, (ins tGPREven:$RdaLo_src, tGPROdd:$RdaHi_src, rGPR:$Rm),
     "$RdaLo, $RdaHi, $Rm", op5, 0b0, pattern> {
```
- EN: Declares reusable TableGen class `MVE_ScalarShiftDRegReg` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_ScalarShiftDRegReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 593-594
```tablegen
  let Inst{7} = 0b0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 596-600
```tablegen
class MVE_ScalarShiftDRegRegWithSat<string iname, bit op5, list<dag> pattern=[]>
  : MVE_ScalarShiftDRegRegBase<
     iname, (ins tGPREven:$RdaLo_src, tGPROdd:$RdaHi_src, rGPR:$Rm, saturateop:$sat),
     "$RdaLo, $RdaHi, $sat, $Rm", op5, 0b1, pattern> {
  bit sat;
```
- EN: Declares reusable TableGen class `MVE_ScalarShiftDRegRegWithSat` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_ScalarShiftDRegRegWithSat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 602-603
```tablegen
  let Inst{7} = sat;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 605-619
```tablegen
def MVE_ASRLr   : MVE_ScalarShiftDRegReg<"asrl",    0b1,  [(set tGPREven:$RdaLo, tGPROdd:$RdaHi,
                                        (ARMasrl tGPREven:$RdaLo_src,
                                        tGPROdd:$RdaHi_src, rGPR:$Rm))]>;
def MVE_ASRLi   : MVE_ScalarShiftDRegImm<"asrl",    0b10, ?, [(set tGPREven:$RdaLo, tGPROdd:$RdaHi,
                                        (ARMasrl tGPREven:$RdaLo_src,
                                        tGPROdd:$RdaHi_src, (i32 long_shift:$imm)))]>;
def MVE_LSLLr   : MVE_ScalarShiftDRegReg<"lsll",    0b0,  [(set tGPREven:$RdaLo, tGPROdd:$RdaHi,
                                        (ARMlsll tGPREven:$RdaLo_src,
                                        tGPROdd:$RdaHi_src, rGPR:$Rm))]>;
def MVE_LSLLi   : MVE_ScalarShiftDRegImm<"lsll",    0b00, ?, [(set tGPREven:$RdaLo, tGPROdd:$RdaHi,
                                        (ARMlsll tGPREven:$RdaLo_src,
                                        tGPROdd:$RdaHi_src, (i32 long_shift:$imm)))]>;
def MVE_LSRL    : MVE_ScalarShiftDRegImm<"lsrl",    0b01, ?, [(set tGPREven:$RdaLo, tGPROdd:$RdaHi,
                                        (ARMlsrl tGPREven:$RdaLo_src,
                                        tGPROdd:$RdaHi_src, (i32 long_shift:$imm)))]>;
```
- EN: Defines TableGen record `MVE_ASRLr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_ASRLr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 621-623
```tablegen
def MVE_SQRSHRL : MVE_ScalarShiftDRegRegWithSat<"sqrshrl", 0b1>;
def MVE_SQSHLL  : MVE_ScalarShiftDRegImm<"sqshll",  0b11, 0b1>;
def MVE_SRSHRL  : MVE_ScalarShiftDRegImm<"srshrl",  0b10, 0b1>;
```
- EN: Defines TableGen record `MVE_SQRSHRL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_SQRSHRL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 625-627
```tablegen
def MVE_UQRSHLL : MVE_ScalarShiftDRegRegWithSat<"uqrshll", 0b0>;
def MVE_UQSHLL  : MVE_ScalarShiftDRegImm<"uqshll",  0b00, 0b1>;
def MVE_URSHRL  : MVE_ScalarShiftDRegImm<"urshrl",  0b01, 0b1>;
```
- EN: Defines TableGen record `MVE_UQRSHLL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_UQRSHLL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 629-629
```tablegen
// start of mve_rDest instructions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 631-637
```tablegen
class MVE_rDest<dag oops, dag iops, InstrItinClass itin,
                string iname, string suffix,
                string ops, string cstr, bits<2> vecsize, list<dag> pattern=[]>
// Always use vpred_n and not vpred_r: with the output register being
// a GPR and not a vector register, there can't be any question of
// what to put in its inactive lanes.
  : MVE_p<oops, iops, itin, iname, suffix, ops, vpred_n, cstr, vecsize, pattern> {
```
- EN: Declares reusable TableGen class `MVE_rDest` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_rDest`，通常用于抽象共享字段、谓词或编码结构。

### Lines 639-642
```tablegen
  let Inst{25-23} = 0b101;
  let Inst{11-9} = 0b111;
  let Inst{4} = 0b0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 644-650
```tablegen
class MVE_VABAV<string suffix, bit U, bits<2> size>
  : MVE_rDest<(outs rGPR:$Rda), (ins rGPR:$Rda_src, MQPR:$Qn, MQPR:$Qm),
              NoItinerary, "vabav", suffix, "$Rda, $Qn, $Qm", "$Rda = $Rda_src",
              size, []> {
  bits<4> Qm;
  bits<4> Qn;
  bits<4> Rda;
```
- EN: Declares reusable TableGen class `MVE_VABAV` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VABAV`，通常用于抽象共享字段、谓词或编码结构。

### Lines 652-665
```tablegen
  let Inst{28} = U;
  let Inst{22} = 0b0;
  let Inst{21-20} = size{1-0};
  let Inst{19-17} = Qn{2-0};
  let Inst{16} = 0b0;
  let Inst{15-12} = Rda{3-0};
  let Inst{8} = 0b1;
  let Inst{7} = Qn{3};
  let Inst{6} = 0b0;
  let Inst{5} = Qm{3};
  let Inst{3-1} = Qm{2-0};
  let Inst{0} = 0b1;
  let horizontalReduction = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 667-669
```tablegen
multiclass MVE_VABAV_m<MVEVectorVTInfo VTI> {
  def "" : MVE_VABAV<VTI.Suffix, VTI.Unsigned, VTI.Size>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 671-677
```tablegen
  let Predicates = [HasMVEInt] in {
    def : Pat<(i32 (int_arm_mve_vabav
                         (i32 VTI.Unsigned),
                         (i32 rGPR:$Rda_src),
                         (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm))),
              (i32 (Inst (i32 rGPR:$Rda_src),
                         (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm)))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 679-688
```tablegen
    def : Pat<(i32 (int_arm_mve_vabav_predicated
                         (i32 VTI.Unsigned),
                         (i32 rGPR:$Rda_src),
                         (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm),
                         (VTI.Pred VCCR:$mask))),
              (i32 (Inst (i32 rGPR:$Rda_src),
                         (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm),
                         ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg))>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 690-695
```tablegen
defm MVE_VABAVs8  : MVE_VABAV_m<MVE_v16s8>;
defm MVE_VABAVs16 : MVE_VABAV_m<MVE_v8s16>;
defm MVE_VABAVs32 : MVE_VABAV_m<MVE_v4s32>;
defm MVE_VABAVu8  : MVE_VABAV_m<MVE_v16u8>;
defm MVE_VABAVu16 : MVE_VABAV_m<MVE_v8u16>;
defm MVE_VABAVu32 : MVE_VABAV_m<MVE_v4u32>;
```
- EN: Defines TableGen record `MVE_VABAVs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VABAVs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 697-702
```tablegen
class MVE_VADDV<string iname, string suffix, dag iops, string cstr,
              bit A, bit U, bits<2> size, list<dag> pattern=[]>
  : MVE_rDest<(outs tGPREven:$Rda), iops, NoItinerary,
              iname, suffix, "$Rda, $Qm", cstr, size, pattern> {
  bits<3> Qm;
  bits<4> Rda;
```
- EN: Declares reusable TableGen class `MVE_VADDV` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VADDV`，通常用于抽象共享字段、谓词或编码结构。

### Lines 704-716
```tablegen
  let Inst{28} = U;
  let Inst{22-20} = 0b111;
  let Inst{19-18} = size{1-0};
  let Inst{17-16} = 0b01;
  let Inst{15-13} = Rda{3-1};
  let Inst{12} = 0b0;
  let Inst{8-6} = 0b100;
  let Inst{5} = A;
  let Inst{3-1} = Qm{2-0};
  let Inst{0} = 0b0;
  let horizontalReduction = 1;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 718-720
```tablegen
def SDTVecReduceP : SDTypeProfile<1, 2, [    // VADDLVp
  SDTCisInt<0>, SDTCisVec<1>, SDTCisVec<2>
]>;
```
- EN: Defines TableGen record `SDTVecReduceP` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTVecReduceP`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 722-725
```tablegen
// sign- or zero-extend the elements of a vector to i32,
// add them all together, and return an i32 of their sum
def ARMVADDVs       : SDNode<"ARMISD::VADDVs", SDTVecReduce>;
def ARMVADDVu       : SDNode<"ARMISD::VADDVu", SDTVecReduce>;
```
- EN: Defines TableGen record `ARMVADDVs` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMVADDVs`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 727-729
```tablegen
// Same as VADDV[su] but with a v4i1 predicate mask
def ARMVADDVps      : SDNode<"ARMISD::VADDVps", SDTVecReduceP>;
def ARMVADDVpu      : SDNode<"ARMISD::VADDVpu", SDTVecReduceP>;
```
- EN: Defines TableGen record `ARMVADDVps` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMVADDVps`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 731-737
```tablegen
multiclass MVE_VADDV_A<MVEVectorVTInfo VTI> {
  def acc    : MVE_VADDV<"vaddva", VTI.Suffix,
                         (ins tGPREven:$Rda_src, MQPR:$Qm), "$Rda = $Rda_src",
                         0b1, VTI.Unsigned, VTI.Size>;
  def no_acc : MVE_VADDV<"vaddv", VTI.Suffix,
                         (ins MQPR:$Qm), "",
                         0b0, VTI.Unsigned, VTI.Size>;
```
- EN: Declares TableGen `multiclass MVE_VADDV_A`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VADDV_A`，它是一个可复用模板，可展开为多个相关记录。

### Lines 739-740
```tablegen
  defvar InstA = !cast<Instruction>(NAME # "acc");
  defvar InstN = !cast<Instruction>(NAME # "no_acc");
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 742-759
```tablegen
  let Predicates = [HasMVEInt] in {
    if VTI.Unsigned then {
      def : Pat<(i32 (vecreduce_add (VTI.Vec MQPR:$vec))),
                (i32 (InstN $vec))>;
      def : Pat<(i32 (vecreduce_add (VTI.Vec (vselect (VTI.Pred VCCR:$pred),
                                                      (VTI.Vec MQPR:$vec),
                                                      (VTI.Vec ARMimmAllZerosV))))),
                (i32 (InstN $vec, ARMVCCThen, $pred, zero_reg))>;
      def : Pat<(i32 (ARMVADDVu (VTI.Vec MQPR:$vec))),
                (i32 (InstN $vec))>;
      def : Pat<(i32 (ARMVADDVpu (VTI.Vec MQPR:$vec), (VTI.Pred VCCR:$pred))),
                (i32 (InstN $vec, ARMVCCThen, $pred, zero_reg))>;
      def : Pat<(i32 (add (i32 (vecreduce_add (VTI.Vec MQPR:$vec))),
                          (i32 tGPREven:$acc))),
                (i32 (InstA $acc, $vec))>;
      def : Pat<(i32 (add (i32 (vecreduce_add (VTI.Vec (vselect (VTI.Pred VCCR:$pred),
                                                                (VTI.Vec MQPR:$vec),
                                                                (VTI.Vec ARMimmAllZerosV))))),
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 760-777
```tablegen
                          (i32 tGPREven:$acc))),
                (i32 (InstA $acc, $vec, ARMVCCThen, $pred, zero_reg))>;
      def : Pat<(i32 (add (i32 (ARMVADDVu (VTI.Vec MQPR:$vec))),
                          (i32 tGPREven:$acc))),
                (i32 (InstA $acc, $vec))>;
      def : Pat<(i32 (add (i32 (ARMVADDVpu (VTI.Vec MQPR:$vec), (VTI.Pred VCCR:$pred))),
                          (i32 tGPREven:$acc))),
                (i32 (InstA $acc, $vec, ARMVCCThen, $pred, zero_reg))>;
    } else {
      def : Pat<(i32 (ARMVADDVs (VTI.Vec MQPR:$vec))),
                (i32 (InstN $vec))>;
      def : Pat<(i32 (add (i32 (ARMVADDVs (VTI.Vec MQPR:$vec))),
                          (i32 tGPREven:$acc))),
                (i32 (InstA $acc, $vec))>;
      def : Pat<(i32 (ARMVADDVps (VTI.Vec MQPR:$vec), (VTI.Pred VCCR:$pred))),
                (i32 (InstN $vec, ARMVCCThen, $pred, zero_reg))>;
      def : Pat<(i32 (add (i32 (ARMVADDVps (VTI.Vec MQPR:$vec), (VTI.Pred VCCR:$pred))),
                          (i32 tGPREven:$acc))),
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 778-779
```tablegen
                (i32 (InstA $acc, $vec, ARMVCCThen, $pred, zero_reg))>;
    }
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 781-791
```tablegen
    def : Pat<(i32 (int_arm_mve_addv_predicated (VTI.Vec MQPR:$vec),
                                                (i32 VTI.Unsigned),
                                                (VTI.Pred VCCR:$pred))),
              (i32 (InstN $vec, ARMVCCThen, $pred, zero_reg))>;
    def : Pat<(i32 (add (int_arm_mve_addv_predicated (VTI.Vec MQPR:$vec),
                                                     (i32 VTI.Unsigned),
                                                     (VTI.Pred VCCR:$pred)),
                        (i32 tGPREven:$acc))),
              (i32 (InstA $acc, $vec, ARMVCCThen, $pred, zero_reg))>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 793-798
```tablegen
defm MVE_VADDVs8  : MVE_VADDV_A<MVE_v16s8>;
defm MVE_VADDVs16 : MVE_VADDV_A<MVE_v8s16>;
defm MVE_VADDVs32 : MVE_VADDV_A<MVE_v4s32>;
defm MVE_VADDVu8  : MVE_VADDV_A<MVE_v16u8>;
defm MVE_VADDVu16 : MVE_VADDV_A<MVE_v8u16>;
defm MVE_VADDVu32 : MVE_VADDV_A<MVE_v4u32>;
```
- EN: Defines TableGen record `MVE_VADDVs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VADDVs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 800-806
```tablegen
class MVE_VADDLV<string iname, string suffix, dag iops, string cstr,
               bit A, bit U, list<dag> pattern=[]>
  : MVE_rDest<(outs tGPREven:$RdaLo, tGPROdd:$RdaHi), iops, NoItinerary, iname,
              suffix, "$RdaLo, $RdaHi, $Qm", cstr, 0b10, pattern> {
  bits<3> Qm;
  bits<4> RdaLo;
  bits<4> RdaHi;
```
- EN: Declares reusable TableGen class `MVE_VADDLV` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VADDLV`，通常用于抽象共享字段、谓词或编码结构。

### Lines 808-819
```tablegen
  let Inst{28} = U;
  let Inst{22-20} = RdaHi{3-1};
  let Inst{19-18} = 0b10;
  let Inst{17-16} = 0b01;
  let Inst{15-13} = RdaLo{3-1};
  let Inst{12} = 0b0;
  let Inst{8-6} = 0b100;
  let Inst{5} = A;
  let Inst{3-1} = Qm{2-0};
  let Inst{0} = 0b0;
  let horizontalReduction = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 821-834
```tablegen
def SDTVecReduceL : SDTypeProfile<2, 1, [    // VADDLV
  SDTCisInt<0>, SDTCisInt<1>, SDTCisVec<2>
]>;
def SDTVecReduceLA : SDTypeProfile<2, 3, [    // VADDLVA
  SDTCisInt<0>, SDTCisInt<1>, SDTCisInt<2>, SDTCisInt<3>,
  SDTCisVec<4>
]>;
def SDTVecReduceLP : SDTypeProfile<2, 2, [    // VADDLVp
  SDTCisInt<0>, SDTCisInt<1>, SDTCisVec<2>, SDTCisVec<2>
]>;
def SDTVecReduceLPA : SDTypeProfile<2, 4, [    // VADDLVAp
  SDTCisInt<0>, SDTCisInt<1>, SDTCisInt<2>, SDTCisInt<3>,
  SDTCisVec<4>, SDTCisVec<5>
]>;
```
- EN: Defines TableGen record `SDTVecReduceL` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTVecReduceL`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 836-843
```tablegen
multiclass MVE_VADDLV_A<MVEVectorVTInfo VTI> {
  def acc    : MVE_VADDLV<"vaddlva", VTI.Suffix,
                        (ins tGPREven:$RdaLo_src, tGPROdd:$RdaHi_src, MQPR:$Qm),
                        "$RdaLo = $RdaLo_src,$RdaHi = $RdaHi_src",
                        0b1, VTI.Unsigned>;
  def no_acc : MVE_VADDLV<"vaddlv", VTI.Suffix,
                        (ins MQPR:$Qm), "",
                        0b0, VTI.Unsigned>;
```
- EN: Declares TableGen `multiclass MVE_VADDLV_A`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VADDLV_A`，它是一个可复用模板，可展开为多个相关记录。

### Lines 845-846
```tablegen
  defvar InstA = !cast<Instruction>(NAME # "acc");
  defvar InstN = !cast<Instruction>(NAME # "no_acc");
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 848-848
```tablegen
  defvar letter = VTI.SuffixLetter;
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 850-852
```tablegen
  // sign- or zero-extend elements to i64 and sum, returning
  // the low and high 32-bit halves of the sum
  defvar ARMVADDLV = SDNode<"ARMISD::VADDLV" # letter, SDTVecReduceL>;
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 854-856
```tablegen
  // Same as VADDLV[su] but also add an input accumulator
  // provided as low and high halves
  defvar ARMVADDLVA = SDNode<"ARMISD::VADDLVA" # letter, SDTVecReduceLA>;
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 858-859
```tablegen
  // Same as VADDLV[su] but with a v4i1 predicate mask
  defvar ARMVADDLVp = SDNode<"ARMISD::VADDLVp" # letter, SDTVecReduceLP>;
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 861-862
```tablegen
  // Same as VADDLVp[su] but with a v4i1 predicate mask
  defvar ARMVADDLVAp = SDNode<"ARMISD::VADDLVAp" # letter, SDTVecReduceLPA>;
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 864-876
```tablegen
  let Predicates = [HasMVEInt] in {
    def : Pat<(ARMVADDLV (v4i32 MQPR:$vec)),
              (InstN (v4i32 MQPR:$vec))>;
    def : Pat<(ARMVADDLVA tGPREven:$acclo, tGPROdd:$acchi, (v4i32 MQPR:$vec)),
              (InstA tGPREven:$acclo, tGPROdd:$acchi, (v4i32 MQPR:$vec))>;
    def : Pat<(ARMVADDLVp (v4i32 MQPR:$vec), (VTI.Pred VCCR:$pred)),
              (InstN (v4i32 MQPR:$vec), ARMVCCThen, (VTI.Pred VCCR:$pred), zero_reg)>;
    def : Pat<(ARMVADDLVAp tGPREven:$acclo, tGPROdd:$acchi, (v4i32 MQPR:$vec),
                           (VTI.Pred VCCR:$pred)),
              (InstA tGPREven:$acclo, tGPROdd:$acchi, (v4i32 MQPR:$vec),
                     ARMVCCThen, (VTI.Pred VCCR:$pred), zero_reg)>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 878-879
```tablegen
defm MVE_VADDLVs32 : MVE_VADDLV_A<MVE_v4s32>;
defm MVE_VADDLVu32 : MVE_VADDLV_A<MVE_v4u32>;
```
- EN: Defines TableGen record `MVE_VADDLVs32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VADDLVs32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 881-887
```tablegen
class MVE_VMINMAXNMV<string iname, string suffix, bit sz,
                     bit bit_17, bit bit_7, list<dag> pattern=[]>
  : MVE_rDest<(outs rGPR:$RdaDest), (ins rGPR:$RdaSrc, MQPR:$Qm),
              NoItinerary, iname, suffix, "$RdaSrc, $Qm",
              "$RdaDest = $RdaSrc", !if(sz, 0b01, 0b10), pattern> {
  bits<3> Qm;
  bits<4> RdaDest;
```
- EN: Declares reusable TableGen class `MVE_VMINMAXNMV` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VMINMAXNMV`，通常用于抽象共享字段、谓词或编码结构。

### Lines 889-900
```tablegen
  let Inst{28} = sz;
  let Inst{22-20} = 0b110;
  let Inst{19-18} = 0b11;
  let Inst{17} = bit_17;
  let Inst{16} = 0b0;
  let Inst{15-12} = RdaDest{3-0};
  let Inst{8} = 0b1;
  let Inst{7} = bit_7;
  let Inst{6-5} = 0b00;
  let Inst{3-1} = Qm{2-0};
  let Inst{0} = 0b0;
  let horizontalReduction = 1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 902-904
```tablegen
  let Predicates = [HasMVEFloat];
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 906-912
```tablegen
multiclass MVE_VMINMAXNMV_p<string iname, bit notAbs, bit isMin,
                            MVEVectorVTInfo VTI, string intrBaseName,
                            ValueType Scalar, RegisterClass ScalarReg> {
  def "": MVE_VMINMAXNMV<iname, VTI.Suffix, VTI.Size{0}, notAbs, isMin>;
  defvar Inst        = !cast<Instruction>(NAME);
  defvar unpred_intr = !cast<Intrinsic>(intrBaseName);
  defvar pred_intr   = !cast<Intrinsic>(intrBaseName#"_predicated");
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 914-928
```tablegen
  let Predicates = [HasMVEFloat] in {
    def : Pat<(Scalar (unpred_intr (Scalar ScalarReg:$prev),
                                   (VTI.Vec MQPR:$vec))),
           (COPY_TO_REGCLASS (Inst (COPY_TO_REGCLASS ScalarReg:$prev, rGPR),
                                   (VTI.Vec MQPR:$vec)),
                              ScalarReg)>;
    def : Pat<(Scalar (pred_intr   (Scalar ScalarReg:$prev),
                                   (VTI.Vec MQPR:$vec),
                                   (VTI.Pred VCCR:$pred))),
           (COPY_TO_REGCLASS (Inst (COPY_TO_REGCLASS ScalarReg:$prev, rGPR),
                                   (VTI.Vec MQPR:$vec),
                                   ARMVCCThen, (VTI.Pred VCCR:$pred), zero_reg),
                              ScalarReg)>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 930-936
```tablegen
multiclass MVE_VMINMAXNMV_fty<string iname, bit notAbs, bit isMin,
                              string intrBase> {
  defm f32 : MVE_VMINMAXNMV_p<iname, notAbs, isMin, MVE_v4f32, intrBase,
                              f32, SPR>;
  defm f16 : MVE_VMINMAXNMV_p<iname, notAbs, isMin, MVE_v8f16, intrBase,
                              f16, HPR>;
}
```
- EN: Declares TableGen `multiclass MVE_VMINMAXNMV_fty`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMINMAXNMV_fty`，它是一个可复用模板，可展开为多个相关记录。

### Lines 938-941
```tablegen
defm MVE_VMINNMV : MVE_VMINMAXNMV_fty<"vminnmv",  1, 1, "int_arm_mve_minnmv">;
defm MVE_VMAXNMV : MVE_VMINMAXNMV_fty<"vmaxnmv",  1, 0, "int_arm_mve_maxnmv">;
defm MVE_VMINNMAV: MVE_VMINMAXNMV_fty<"vminnmav", 0, 1, "int_arm_mve_minnmav">;
defm MVE_VMAXNMAV: MVE_VMINMAXNMV_fty<"vmaxnmav", 0, 0, "int_arm_mve_maxnmav">;
```
- EN: Defines TableGen record `MVE_VMINNMV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMINNMV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 943-948
```tablegen
class MVE_VMINMAXV<string iname, string suffix, bit U, bits<2> size,
                 bit bit_17, bit bit_7, list<dag> pattern=[]>
  : MVE_rDest<(outs rGPR:$RdaDest), (ins rGPR:$RdaSrc, MQPR:$Qm), NoItinerary,
              iname, suffix, "$RdaSrc, $Qm", "$RdaDest = $RdaSrc", size, pattern> {
  bits<3> Qm;
  bits<4> RdaDest;
```
- EN: Declares reusable TableGen class `MVE_VMINMAXV` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VMINMAXV`，通常用于抽象共享字段、谓词或编码结构。

### Lines 950-962
```tablegen
  let Inst{28} = U;
  let Inst{22-20} = 0b110;
  let Inst{19-18} = size{1-0};
  let Inst{17} = bit_17;
  let Inst{16} = 0b0;
  let Inst{15-12} = RdaDest{3-0};
  let Inst{8} = 0b1;
  let Inst{7} = bit_7;
  let Inst{6-5} = 0b00;
  let Inst{3-1} = Qm{2-0};
  let Inst{0} = 0b0;
  let horizontalReduction = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 964-973
```tablegen
multiclass MVE_VMINMAXV_p<string iname, bit notAbs, bit isMin,
                          MVEVectorVTInfo VTI, string intrBaseName> {
  def "": MVE_VMINMAXV<iname, VTI.Suffix, VTI.Unsigned, VTI.Size,
                       notAbs, isMin>;
  defvar Inst        = !cast<Instruction>(NAME);
  defvar unpred_intr = !cast<Intrinsic>(intrBaseName);
  defvar pred_intr   = !cast<Intrinsic>(intrBaseName#"_predicated");
  defvar base_args   = (? (i32 rGPR:$prev), (VTI.Vec MQPR:$vec));
  defvar args        = !if(notAbs, !con(base_args, (? (i32 VTI.Unsigned))),
                           base_args);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 975-982
```tablegen
  let Predicates = [HasMVEInt] in {
    def : Pat<(i32 !con(args, (unpred_intr))),
              (i32 (Inst (i32 rGPR:$prev), (VTI.Vec MQPR:$vec)))>;
    def : Pat<(i32 !con(args, (pred_intr (VTI.Pred VCCR:$pred)))),
              (i32 (Inst (i32 rGPR:$prev), (VTI.Vec MQPR:$vec),
                         ARMVCCThen, (VTI.Pred VCCR:$pred), zero_reg))>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 984-991
```tablegen
multiclass MVE_VMINMAXV_ty<string iname, bit isMin, string intrBaseName> {
  defm s8 : MVE_VMINMAXV_p<iname, 1, isMin, MVE_v16s8, intrBaseName>;
  defm s16: MVE_VMINMAXV_p<iname, 1, isMin, MVE_v8s16, intrBaseName>;
  defm s32: MVE_VMINMAXV_p<iname, 1, isMin, MVE_v4s32, intrBaseName>;
  defm u8 : MVE_VMINMAXV_p<iname, 1, isMin, MVE_v16u8, intrBaseName>;
  defm u16: MVE_VMINMAXV_p<iname, 1, isMin, MVE_v8u16, intrBaseName>;
  defm u32: MVE_VMINMAXV_p<iname, 1, isMin, MVE_v4u32, intrBaseName>;
}
```
- EN: Declares TableGen `multiclass MVE_VMINMAXV_ty`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMINMAXV_ty`，它是一个可复用模板，可展开为多个相关记录。

### Lines 993-995
```tablegen
def SDTVecReduceR : SDTypeProfile<1, 2, [   // Reduction of an integer and vector into an integer
  SDTCisInt<0>, SDTCisInt<1>, SDTCisVec<2>
]>;
```
- EN: Defines TableGen record `SDTVecReduceR` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTVecReduceR`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 997-998
```tablegen
// Find minimum unsigned value of a vector and register
def ARMVMINVu       : SDNode<"ARMISD::VMINVu", SDTVecReduceR>;
```
- EN: Defines TableGen record `ARMVMINVu` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMVMINVu`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 1000-1001
```tablegen
// Find minimum signed value of a vector and register
def ARMVMINVs       : SDNode<"ARMISD::VMINVs", SDTVecReduceR>;
```
- EN: Defines TableGen record `ARMVMINVs` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMVMINVs`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 1003-1004
```tablegen
// Find maximum unsigned value of a vector and register
def ARMVMAXVu       : SDNode<"ARMISD::VMAXVu", SDTVecReduceR>;
```
- EN: Defines TableGen record `ARMVMAXVu` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMVMAXVu`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 1006-1007
```tablegen
// Find maximum signed value of a vector and register
def ARMVMAXVs       : SDNode<"ARMISD::VMAXVs", SDTVecReduceR>;
```
- EN: Defines TableGen record `ARMVMAXVs` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMVMAXVs`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 1009-1010
```tablegen
defm MVE_VMINV : MVE_VMINMAXV_ty<"vminv", 1, "int_arm_mve_minv">;
defm MVE_VMAXV : MVE_VMINMAXV_ty<"vmaxv", 0, "int_arm_mve_maxv">;
```
- EN: Defines TableGen record `MVE_VMINV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMINV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1012-1024
```tablegen
let Predicates = [HasMVEInt] in {
  def : Pat<(i32 (vecreduce_smax (v16i8 MQPR:$src))),
            (i32 (MVE_VMAXVs8 (t2MVNi (i32 127)), $src))>;
  def : Pat<(i32 (vecreduce_smax (v8i16 MQPR:$src))),
            (i32 (MVE_VMAXVs16 (t2MOVi32imm (i32 -32768)), $src))>;
  def : Pat<(i32 (vecreduce_smax (v4i32 MQPR:$src))),
            (i32 (MVE_VMAXVs32 (t2MOVi (i32 -2147483648)), $src))>;
  def : Pat<(i32 (vecreduce_umax (v16i8 MQPR:$src))),
            (i32 (MVE_VMAXVu8 (t2MOVi (i32 0)), $src))>;
  def : Pat<(i32 (vecreduce_umax (v8i16 MQPR:$src))),
            (i32 (MVE_VMAXVu16 (t2MOVi (i32 0)), $src))>;
  def : Pat<(i32 (vecreduce_umax (v4i32 MQPR:$src))),
            (i32 (MVE_VMAXVu32 (t2MOVi (i32 0)), $src))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1026-1037
```tablegen
  def : Pat<(i32 (vecreduce_smin (v16i8 MQPR:$src))),
            (i32 (MVE_VMINVs8 (t2MOVi (i32 127)), $src))>;
  def : Pat<(i32 (vecreduce_smin (v8i16 MQPR:$src))),
            (i32 (MVE_VMINVs16 (t2MOVi16 (i32 32767)), $src))>;
  def : Pat<(i32 (vecreduce_smin (v4i32 MQPR:$src))),
            (i32 (MVE_VMINVs32 (t2MVNi (i32 -2147483648)), $src))>;
  def : Pat<(i32 (vecreduce_umin (v16i8 MQPR:$src))),
            (i32 (MVE_VMINVu8 (t2MOVi (i32 255)), $src))>;
  def : Pat<(i32 (vecreduce_umin (v8i16 MQPR:$src))),
            (i32 (MVE_VMINVu16 (t2MOVi16 (i32 65535)), $src))>;
  def : Pat<(i32 (vecreduce_umin (v4i32 MQPR:$src))),
            (i32 (MVE_VMINVu32 (t2MOVi (i32 4294967295)), $src))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1039-1050
```tablegen
  def : Pat<(i32 (ARMVMINVu (i32 rGPR:$x), (v16i8 MQPR:$src))),
            (i32 (MVE_VMINVu8 $x, $src))>;
  def : Pat<(i32 (ARMVMINVu (i32 rGPR:$x), (v8i16 MQPR:$src))),
            (i32 (MVE_VMINVu16 $x, $src))>;
  def : Pat<(i32 (ARMVMINVu (i32 rGPR:$x), (v4i32 MQPR:$src))),
            (i32 (MVE_VMINVu32 $x, $src))>;
  def : Pat<(i32 (ARMVMINVs (i32 rGPR:$x), (v16i8 MQPR:$src))),
            (i32 (MVE_VMINVs8 $x, $src))>;
  def : Pat<(i32 (ARMVMINVs (i32 rGPR:$x), (v8i16 MQPR:$src))),
            (i32 (MVE_VMINVs16 $x, $src))>;
  def : Pat<(i32 (ARMVMINVs (i32 rGPR:$x), (v4i32 MQPR:$src))),
            (i32 (MVE_VMINVs32 $x, $src))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1052-1063
```tablegen
  def : Pat<(i32 (ARMVMAXVu (i32 rGPR:$x), (v16i8 MQPR:$src))),
            (i32 (MVE_VMAXVu8 $x, $src))>;
  def : Pat<(i32 (ARMVMAXVu (i32 rGPR:$x), (v8i16 MQPR:$src))),
            (i32 (MVE_VMAXVu16 $x, $src))>;
  def : Pat<(i32 (ARMVMAXVu (i32 rGPR:$x), (v4i32 MQPR:$src))),
            (i32 (MVE_VMAXVu32 $x, $src))>;
  def : Pat<(i32 (ARMVMAXVs (i32 rGPR:$x), (v16i8 MQPR:$src))),
            (i32 (MVE_VMAXVs8 $x, $src))>;
  def : Pat<(i32 (ARMVMAXVs (i32 rGPR:$x), (v8i16 MQPR:$src))),
            (i32 (MVE_VMAXVs16 $x, $src))>;
  def : Pat<(i32 (ARMVMAXVs (i32 rGPR:$x), (v4i32 MQPR:$src))),
            (i32 (MVE_VMAXVs32 $x, $src))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1065-1065
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1067-1071
```tablegen
multiclass MVE_VMINMAXAV_ty<string iname, bit isMin, string intrBaseName> {
  defm s8 : MVE_VMINMAXV_p<iname, 0, isMin, MVE_v16s8, intrBaseName>;
  defm s16: MVE_VMINMAXV_p<iname, 0, isMin, MVE_v8s16, intrBaseName>;
  defm s32: MVE_VMINMAXV_p<iname, 0, isMin, MVE_v4s32, intrBaseName>;
}
```
- EN: Declares TableGen `multiclass MVE_VMINMAXAV_ty`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMINMAXAV_ty`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1073-1074
```tablegen
defm MVE_VMINAV : MVE_VMINMAXAV_ty<"vminav", 1, "int_arm_mve_minav">;
defm MVE_VMAXAV : MVE_VMINMAXAV_ty<"vmaxav", 0, "int_arm_mve_maxav">;
```
- EN: Defines TableGen record `MVE_VMINAV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMINAV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1076-1083
```tablegen
class MVE_VMLAMLSDAV<string iname, string suffix, dag iops, string cstr,
                   bit sz, bit bit_28, bit A, bit X, bit bit_8, bit bit_0,
                   bits<2> vecsize>
  : MVE_rDest<(outs tGPREven:$RdaDest), iops, NoItinerary, iname, suffix,
              "$RdaDest, $Qn, $Qm", cstr, vecsize, []> {
  bits<4> RdaDest;
  bits<3> Qm;
  bits<3> Qn;
```
- EN: Declares reusable TableGen class `MVE_VMLAMLSDAV` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VMLAMLSDAV`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1085-1102
```tablegen
  let Inst{28} = bit_28;
  let Inst{22-20} = 0b111;
  let Inst{19-17} = Qn{2-0};
  let Inst{16} = sz;
  let Inst{15-13} = RdaDest{3-1};
  let Inst{12} = X;
  let Inst{8} = bit_8;
  let Inst{7-6} = 0b00;
  let Inst{5} = A;
  let Inst{3-1} = Qm{2-0};
  let Inst{0} = bit_0;
  let horizontalReduction = 1;
  // Allow tail predication for non-exchanging versions. As this is also a
  // horizontalReduction, ARMLowOverheadLoops will also have to check that
  // the vector operands contain zeros in their false lanes for the instruction
  // to be properly valid.
  let validForTailPredication = !eq(X, 0);
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1104-1121
```tablegen
multiclass MVE_VMLAMLSDAV_A<string iname, string x, MVEVectorVTInfo VTI,
                            bit sz, bit bit_28, bit X, bit bit_8, bit bit_0> {
  def ""#x#VTI.Suffix : MVE_VMLAMLSDAV<iname # x, VTI.Suffix,
                                   (ins MQPR:$Qn, MQPR:$Qm), "",
                                   sz, bit_28, 0b0, X, bit_8, bit_0, VTI.Size>;
  def "a"#x#VTI.Suffix : MVE_VMLAMLSDAV<iname # "a" # x, VTI.Suffix,
                                    (ins tGPREven:$RdaSrc, MQPR:$Qn, MQPR:$Qm),
                                    "$RdaDest = $RdaSrc",
                                    sz, bit_28, 0b1, X, bit_8, bit_0, VTI.Size>;
  let Predicates = [HasMVEInt] in {
    def : Pat<(i32 (int_arm_mve_vmldava
                            (i32 VTI.Unsigned),
                            (i32 bit_0) /* subtract */,
                            (i32 X) /* exchange */,
                            (i32 0) /* accumulator */,
                            (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm))),
              (i32 (!cast<Instruction>(NAME # x # VTI.Suffix)
                            (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm)))>;
```
- EN: Declares TableGen `multiclass MVE_VMLAMLSDAV_A`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMLAMLSDAV_A`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1123-1132
```tablegen
    def : Pat<(i32 (int_arm_mve_vmldava_predicated
                            (i32 VTI.Unsigned),
                            (i32 bit_0) /* subtract */,
                            (i32 X) /* exchange */,
                            (i32 0) /* accumulator */,
                            (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm),
                            (VTI.Pred VCCR:$mask))),
              (i32 (!cast<Instruction>(NAME # x # VTI.Suffix)
                            (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm),
                             ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1134-1142
```tablegen
    def : Pat<(i32 (int_arm_mve_vmldava
                            (i32 VTI.Unsigned),
                            (i32 bit_0) /* subtract */,
                            (i32 X) /* exchange */,
                            (i32 tGPREven:$RdaSrc),
                            (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm))),
              (i32 (!cast<Instruction>(NAME # "a" # x # VTI.Suffix)
                            (i32 tGPREven:$RdaSrc),
                            (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1144-1156
```tablegen
    def : Pat<(i32 (int_arm_mve_vmldava_predicated
                            (i32 VTI.Unsigned),
                            (i32 bit_0) /* subtract */,
                            (i32 X) /* exchange */,
                            (i32 tGPREven:$RdaSrc),
                            (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm),
                            (VTI.Pred VCCR:$mask))),
              (i32 (!cast<Instruction>(NAME # "a" # x # VTI.Suffix)
                            (i32 tGPREven:$RdaSrc),
                            (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm),
                             ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg))>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1158-1164
```tablegen
multiclass MVE_VMLAMLSDAV_AX<string iname, MVEVectorVTInfo VTI, bit sz,
                             bit bit_28, bit bit_8, bit bit_0> {
  defm "" : MVE_VMLAMLSDAV_A<iname, "", VTI, sz, bit_28,
                             0b0, bit_8, bit_0>;
  defm "" : MVE_VMLAMLSDAV_A<iname, "x", VTI, sz, bit_28,
                             0b1, bit_8, bit_0>;
}
```
- EN: Declares TableGen `multiclass MVE_VMLAMLSDAV_AX`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMLAMLSDAV_AX`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1166-1172
```tablegen
multiclass MVE_VMLADAV_multi<MVEVectorVTInfo SVTI, MVEVectorVTInfo UVTI,
                             bit sz, bit bit_8> {
  defm "" : MVE_VMLAMLSDAV_AX<"vmladav", SVTI,
                              sz, 0b0, bit_8, 0b0>;
  defm "" : MVE_VMLAMLSDAV_A<"vmladav", "", UVTI,
                             sz, 0b1, 0b0, bit_8, 0b0>;
}
```
- EN: Declares TableGen `multiclass MVE_VMLADAV_multi`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMLADAV_multi`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1174-1177
```tablegen
multiclass MVE_VMLSDAV_multi<MVEVectorVTInfo VTI, bit sz, bit bit_28> {
  defm "" : MVE_VMLAMLSDAV_AX<"vmlsdav", VTI,
                              sz, bit_28, 0b0, 0b1>;
}
```
- EN: Declares TableGen `multiclass MVE_VMLSDAV_multi`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMLSDAV_multi`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1179-1181
```tablegen
defm MVE_VMLADAV : MVE_VMLADAV_multi<MVE_v16s8, MVE_v16u8, 0b0, 0b1>;
defm MVE_VMLADAV : MVE_VMLADAV_multi<MVE_v8s16, MVE_v8u16, 0b0, 0b0>;
defm MVE_VMLADAV : MVE_VMLADAV_multi<MVE_v4s32, MVE_v4u32, 0b1, 0b0>;
```
- EN: Defines TableGen record `MVE_VMLADAV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMLADAV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1183-1185
```tablegen
defm MVE_VMLSDAV : MVE_VMLSDAV_multi<MVE_v16s8, 0b0, 0b1>;
defm MVE_VMLSDAV : MVE_VMLSDAV_multi<MVE_v8s16, 0b0, 0b0>;
defm MVE_VMLSDAV : MVE_VMLSDAV_multi<MVE_v4s32, 0b1, 0b0>;
```
- EN: Defines TableGen record `MVE_VMLSDAV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMLSDAV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1187-1204
```tablegen
def SDTVecReduce2 : SDTypeProfile<1, 2, [    // VMLAV
  SDTCisInt<0>, SDTCisVec<1>, SDTCisVec<2>
]>;
def SDTVecReduce2L : SDTypeProfile<2, 2, [    // VMLALV
  SDTCisInt<0>, SDTCisInt<1>, SDTCisVec<2>, SDTCisVec<3>
]>;
def SDTVecReduce2LA : SDTypeProfile<2, 4, [    // VMLALVA
  SDTCisInt<0>, SDTCisInt<1>, SDTCisInt<2>, SDTCisInt<3>,
  SDTCisVec<4>, SDTCisVec<5>
]>;
def SDTVecReduce2P : SDTypeProfile<1, 3, [    // VMLAV
  SDTCisInt<0>, SDTCisVec<1>, SDTCisVec<2>, SDTCisVec<3>
]>;
def SDTVecReduce2LP : SDTypeProfile<2, 3, [    // VMLALV
  SDTCisInt<0>, SDTCisInt<1>, SDTCisVec<2>, SDTCisVec<3>, SDTCisVec<4>
]>;
def SDTVecReduce2LAP : SDTypeProfile<2, 5, [    // VMLALVA
  SDTCisInt<0>, SDTCisInt<1>, SDTCisInt<2>, SDTCisInt<3>,
```
- EN: Defines TableGen record `SDTVecReduce2` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTVecReduce2`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 1205-1206
```tablegen
  SDTCisVec<4>, SDTCisVec<5>, SDTCisVec<6>
]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1208-1211
```tablegen
// sign- or zero-extend the elements of two vectors to i32, multiply
// them and add the results together, returning an i32 of the sum
def ARMVMLAVs       : SDNode<"ARMISD::VMLAVs", SDTVecReduce2>;
def ARMVMLAVu       : SDNode<"ARMISD::VMLAVu", SDTVecReduce2>;
```
- EN: Defines TableGen record `ARMVMLAVs` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMVMLAVs`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 1213-1216
```tablegen
// Same as VMLAV but with i64, returning the low and
// high 32-bit halves of the sum
def ARMVMLALVs      : SDNode<"ARMISD::VMLALVs", SDTVecReduce2L>;
def ARMVMLALVu      : SDNode<"ARMISD::VMLALVu", SDTVecReduce2L>;
```
- EN: Defines TableGen record `ARMVMLALVs` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMVMLALVs`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 1218-1221
```tablegen
// Same as VMLALV but also add an input accumulator
// provided as low and high halves
def ARMVMLALVAs     : SDNode<"ARMISD::VMLALVAs", SDTVecReduce2LA>;
def ARMVMLALVAu     : SDNode<"ARMISD::VMLALVAu", SDTVecReduce2LA>;
```
- EN: Defines TableGen record `ARMVMLALVAs` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMVMLALVAs`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 1223-1225
```tablegen
// Same as VMLAV[su] with a v4i1 predicate mask
def ARMVMLAVps      : SDNode<"ARMISD::VMLAVps", SDTVecReduce2P>;
def ARMVMLAVpu      : SDNode<"ARMISD::VMLAVpu", SDTVecReduce2P>;
```
- EN: Defines TableGen record `ARMVMLAVps` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMVMLAVps`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 1227-1229
```tablegen
// Same as VMLALV[su] with a v4i1 predicate mask
def ARMVMLALVps     : SDNode<"ARMISD::VMLALVps", SDTVecReduce2LP>;
def ARMVMLALVpu     : SDNode<"ARMISD::VMLALVpu", SDTVecReduce2LP>;
```
- EN: Defines TableGen record `ARMVMLALVps` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMVMLALVps`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 1231-1233
```tablegen
// Same as VMLALVA[su] with a v4i1 predicate mask
def ARMVMLALVAps    : SDNode<"ARMISD::VMLALVAps", SDTVecReduce2LAP>;
def ARMVMLALVApu    : SDNode<"ARMISD::VMLALVApu", SDTVecReduce2LAP>;
```
- EN: Defines TableGen record `ARMVMLALVAps` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMVMLALVAps`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 1235-1249
```tablegen
let Predicates = [HasMVEInt] in {
  def : Pat<(i32 (vecreduce_add (mul (v4i32 MQPR:$src1), (v4i32 MQPR:$src2)))),
            (i32 (MVE_VMLADAVu32 $src1, $src2))>;
  def : Pat<(i32 (vecreduce_add (mul (v8i16 MQPR:$src1), (v8i16 MQPR:$src2)))),
            (i32 (MVE_VMLADAVu16 $src1, $src2))>;
  def : Pat<(i32 (ARMVMLAVs (v8i16 MQPR:$val1), (v8i16 MQPR:$val2))),
            (i32 (MVE_VMLADAVs16 (v8i16 MQPR:$val1), (v8i16 MQPR:$val2)))>;
  def : Pat<(i32 (ARMVMLAVu (v8i16 MQPR:$val1), (v8i16 MQPR:$val2))),
            (i32 (MVE_VMLADAVu16 (v8i16 MQPR:$val1), (v8i16 MQPR:$val2)))>;
  def : Pat<(i32 (vecreduce_add (mul (v16i8 MQPR:$src1), (v16i8 MQPR:$src2)))),
            (i32 (MVE_VMLADAVu8 $src1, $src2))>;
  def : Pat<(i32 (ARMVMLAVs (v16i8 MQPR:$val1), (v16i8 MQPR:$val2))),
            (i32 (MVE_VMLADAVs8 (v16i8 MQPR:$val1), (v16i8 MQPR:$val2)))>;
  def : Pat<(i32 (ARMVMLAVu (v16i8 MQPR:$val1), (v16i8 MQPR:$val2))),
            (i32 (MVE_VMLADAVu8 (v16i8 MQPR:$val1), (v16i8 MQPR:$val2)))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1251-1267
```tablegen
  def : Pat<(i32 (add (i32 (vecreduce_add (mul (v4i32 MQPR:$src1), (v4i32 MQPR:$src2)))),
                      (i32 tGPREven:$src3))),
            (i32 (MVE_VMLADAVau32 $src3, $src1, $src2))>;
  def : Pat<(i32 (add (i32 (vecreduce_add (mul (v8i16 MQPR:$src1), (v8i16 MQPR:$src2)))),
                      (i32 tGPREven:$src3))),
            (i32 (MVE_VMLADAVau16 $src3, $src1, $src2))>;
  def : Pat<(i32 (add (ARMVMLAVs (v8i16 MQPR:$val1), (v8i16 MQPR:$val2)), tGPREven:$Rd)),
            (i32 (MVE_VMLADAVas16 tGPREven:$Rd, (v8i16 MQPR:$val1), (v8i16 MQPR:$val2)))>;
  def : Pat<(i32 (add (ARMVMLAVu (v8i16 MQPR:$val1), (v8i16 MQPR:$val2)), tGPREven:$Rd)),
            (i32 (MVE_VMLADAVau16 tGPREven:$Rd, (v8i16 MQPR:$val1), (v8i16 MQPR:$val2)))>;
  def : Pat<(i32 (add (i32 (vecreduce_add (mul (v16i8 MQPR:$src1), (v16i8 MQPR:$src2)))),
                      (i32 tGPREven:$src3))),
            (i32 (MVE_VMLADAVau8 $src3, $src1, $src2))>;
  def : Pat<(i32 (add (ARMVMLAVs (v16i8 MQPR:$val1), (v16i8 MQPR:$val2)), tGPREven:$Rd)),
            (i32 (MVE_VMLADAVas8 tGPREven:$Rd, (v16i8 MQPR:$val1), (v16i8 MQPR:$val2)))>;
  def : Pat<(i32 (add (ARMVMLAVu (v16i8 MQPR:$val1), (v16i8 MQPR:$val2)), tGPREven:$Rd)),
            (i32 (MVE_VMLADAVau8 tGPREven:$Rd, (v16i8 MQPR:$val1), (v16i8 MQPR:$val2)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1269-1286
```tablegen
  // Predicated
  def : Pat<(i32 (vecreduce_add (vselect (v4i1 VCCR:$pred),
                                         (mul (v4i32 MQPR:$src1), (v4i32 MQPR:$src2)),
                                         (v4i32 ARMimmAllZerosV)))),
            (i32 (MVE_VMLADAVu32 $src1, $src2, ARMVCCThen, $pred, zero_reg))>;
  def : Pat<(i32 (vecreduce_add (vselect (v8i1 VCCR:$pred),
                                         (mul (v8i16 MQPR:$src1), (v8i16 MQPR:$src2)),
                                         (v8i16 ARMimmAllZerosV)))),
            (i32 (MVE_VMLADAVu16 $src1, $src2, ARMVCCThen, $pred, zero_reg))>;
  def : Pat<(i32 (ARMVMLAVps (v8i16 MQPR:$val1), (v8i16 MQPR:$val2), (v8i1 VCCR:$pred))),
            (i32 (MVE_VMLADAVs16 (v8i16 MQPR:$val1), (v8i16 MQPR:$val2), ARMVCCThen, $pred, zero_reg))>;
  def : Pat<(i32 (ARMVMLAVpu (v8i16 MQPR:$val1), (v8i16 MQPR:$val2), (v8i1 VCCR:$pred))),
            (i32 (MVE_VMLADAVu16 (v8i16 MQPR:$val1), (v8i16 MQPR:$val2), ARMVCCThen, $pred, zero_reg))>;
  def : Pat<(i32 (vecreduce_add (vselect (v16i1 VCCR:$pred),
                                         (mul (v16i8 MQPR:$src1), (v16i8 MQPR:$src2)),
                                         (v16i8 ARMimmAllZerosV)))),
            (i32 (MVE_VMLADAVu8 $src1, $src2, ARMVCCThen, $pred, zero_reg))>;
  def : Pat<(i32 (ARMVMLAVps (v16i8 MQPR:$val1), (v16i8 MQPR:$val2), (v16i1 VCCR:$pred))),
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1287-1289
```tablegen
            (i32 (MVE_VMLADAVs8 (v16i8 MQPR:$val1), (v16i8 MQPR:$val2), ARMVCCThen, $pred, zero_reg))>;
  def : Pat<(i32 (ARMVMLAVpu (v16i8 MQPR:$val1), (v16i8 MQPR:$val2), (v16i1 VCCR:$pred))),
            (i32 (MVE_VMLADAVu8 (v16i8 MQPR:$val1), (v16i8 MQPR:$val2), ARMVCCThen, $pred, zero_reg))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1291-1308
```tablegen
  def : Pat<(i32 (add (i32 (vecreduce_add (vselect (v4i1 VCCR:$pred),
                                                   (mul (v4i32 MQPR:$src1), (v4i32 MQPR:$src2)),
                                                   (v4i32 ARMimmAllZerosV)))),
                      (i32 tGPREven:$src3))),
            (i32 (MVE_VMLADAVau32 $src3, $src1, $src2, ARMVCCThen, $pred, zero_reg))>;
  def : Pat<(i32 (add (i32 (vecreduce_add (vselect (v8i1 VCCR:$pred),
                                                   (mul (v8i16 MQPR:$src1), (v8i16 MQPR:$src2)),
                                                   (v8i16 ARMimmAllZerosV)))),
                      (i32 tGPREven:$src3))),
            (i32 (MVE_VMLADAVau16 $src3, $src1, $src2, ARMVCCThen, $pred, zero_reg))>;
  def : Pat<(i32 (add (ARMVMLAVps (v8i16 MQPR:$val1), (v8i16 MQPR:$val2), (v8i1 VCCR:$pred)), tGPREven:$Rd)),
            (i32 (MVE_VMLADAVas16 tGPREven:$Rd, (v8i16 MQPR:$val1), (v8i16 MQPR:$val2), ARMVCCThen, $pred, zero_reg))>;
  def : Pat<(i32 (add (ARMVMLAVpu (v8i16 MQPR:$val1), (v8i16 MQPR:$val2), (v8i1 VCCR:$pred)), tGPREven:$Rd)),
            (i32 (MVE_VMLADAVau16 tGPREven:$Rd, (v8i16 MQPR:$val1), (v8i16 MQPR:$val2), ARMVCCThen, $pred, zero_reg))>;
  def : Pat<(i32 (add (i32 (vecreduce_add (vselect (v16i1 VCCR:$pred),
                                                   (mul (v16i8 MQPR:$src1), (v16i8 MQPR:$src2)),
                                                   (v16i8 ARMimmAllZerosV)))),
                      (i32 tGPREven:$src3))),
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1309-1314
```tablegen
            (i32 (MVE_VMLADAVau8 $src3, $src1, $src2, ARMVCCThen, $pred, zero_reg))>;
  def : Pat<(i32 (add (ARMVMLAVps (v16i8 MQPR:$val1), (v16i8 MQPR:$val2), (v16i1 VCCR:$pred)), tGPREven:$Rd)),
            (i32 (MVE_VMLADAVas8 tGPREven:$Rd, (v16i8 MQPR:$val1), (v16i8 MQPR:$val2), ARMVCCThen, $pred, zero_reg))>;
  def : Pat<(i32 (add (ARMVMLAVpu (v16i8 MQPR:$val1), (v16i8 MQPR:$val2), (v16i1 VCCR:$pred)), tGPREven:$Rd)),
            (i32 (MVE_VMLADAVau8 tGPREven:$Rd, (v16i8 MQPR:$val1), (v16i8 MQPR:$val2), ARMVCCThen, $pred, zero_reg))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1316-1323
```tablegen
// vmlav aliases vmladav
foreach acc = ["", "a"] in {
  foreach suffix = ["s8", "s16", "s32", "u8", "u16", "u32"] in {
    def : MVEInstAlias<"vmlav"#acc#"${vp}."#suffix#"\t$RdaDest, $Qn, $Qm",
                       (!cast<Instruction>("MVE_VMLADAV"#acc#suffix)
                        tGPREven:$RdaDest, MQPR:$Qn, MQPR:$Qm, vpred_n:$vp)>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1325-1334
```tablegen
// Base class for VMLALDAV and VMLSLDAV, VRMLALDAVH, VRMLSLDAVH
class MVE_VMLALDAVBase<string iname, string suffix, dag iops, string cstr,
                       bit sz, bit bit_28, bit A, bit X, bit bit_8, bit bit_0,
                       bits<2> vecsize, list<dag> pattern=[]>
  : MVE_rDest<(outs tGPREven:$RdaLoDest, tGPROdd:$RdaHiDest), iops, NoItinerary,
              iname, suffix, "$RdaLoDest, $RdaHiDest, $Qn, $Qm", cstr, vecsize, pattern> {
  bits<4> RdaLoDest;
  bits<4> RdaHiDest;
  bits<3> Qm;
  bits<3> Qn;
```
- EN: Declares reusable TableGen class `for` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `for`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1336-1352
```tablegen
  let Inst{28} = bit_28;
  let Inst{22-20} = RdaHiDest{3-1};
  let Inst{19-17} = Qn{2-0};
  let Inst{16} = sz;
  let Inst{15-13} = RdaLoDest{3-1};
  let Inst{12} = X;
  let Inst{8} = bit_8;
  let Inst{7-6} = 0b00;
  let Inst{5} = A;
  let Inst{3-1} = Qm{2-0};
  let Inst{0} = bit_0;
  let horizontalReduction = 1;
  // Allow tail predication for non-exchanging versions. As this is also a
  // horizontalReduction, ARMLowOverheadLoops will also have to check that
  // the vector operands contain zeros in their false lanes for the instruction
  // to be properly valid.
  let validForTailPredication = !eq(X, 0);
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1354-1355
```tablegen
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1357-1368
```tablegen
multiclass MVE_VMLALDAVBase_A<string iname, string x, string suffix,
                              bit sz, bit bit_28, bit X, bit bit_8, bit bit_0,
                              bits<2> vecsize, list<dag> pattern=[]> {
  def ""#x#suffix : MVE_VMLALDAVBase<
     iname # x, suffix, (ins MQPR:$Qn, MQPR:$Qm), "",
     sz, bit_28, 0b0, X, bit_8, bit_0, vecsize, pattern>;
  def "a"#x#suffix : MVE_VMLALDAVBase<
     iname # "a" # x, suffix,
     (ins tGPREven:$RdaLoSrc, tGPROdd:$RdaHiSrc, MQPR:$Qn, MQPR:$Qm),
     "$RdaLoDest = $RdaLoSrc,$RdaHiDest = $RdaHiSrc",
     sz, bit_28, 0b1, X, bit_8, bit_0, vecsize, pattern>;
}
```
- EN: Declares TableGen `multiclass MVE_VMLALDAVBase_A`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMLALDAVBase_A`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1371-1377
```tablegen
multiclass MVE_VMLALDAVBase_AX<string iname, string suffix, bit sz, bit bit_28,
                               bit bit_8, bit bit_0, bits<2> vecsize, list<dag> pattern=[]> {
  defm "" : MVE_VMLALDAVBase_A<iname, "", suffix, sz,
                               bit_28, 0b0, bit_8, bit_0, vecsize, pattern>;
  defm "" : MVE_VMLALDAVBase_A<iname, "x", suffix, sz,
                               bit_28, 0b1, bit_8, bit_0, vecsize, pattern>;
}
```
- EN: Declares TableGen `multiclass MVE_VMLALDAVBase_AX`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMLALDAVBase_AX`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1379-1384
```tablegen
multiclass MVE_VRMLALDAVH_multi<MVEVectorVTInfo VTI, list<dag> pattern=[]> {
  defm "" : MVE_VMLALDAVBase_AX<"vrmlaldavh", "s"#VTI.BitsSuffix,
                                0b0, 0b0, 0b1, 0b0, VTI.Size, pattern>;
  defm "" : MVE_VMLALDAVBase_A<"vrmlaldavh", "", "u"#VTI.BitsSuffix,
                               0b0, 0b1, 0b0, 0b1, 0b0, VTI.Size, pattern>;
}
```
- EN: Declares TableGen `multiclass MVE_VRMLALDAVH_multi`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VRMLALDAVH_multi`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1386-1386
```tablegen
defm MVE_VRMLALDAVH : MVE_VRMLALDAVH_multi<MVE_v4i32>;
```
- EN: Defines TableGen record `MVE_VRMLALDAVH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VRMLALDAVH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1388-1404
```tablegen
// vrmlalvh aliases for vrmlaldavh
def : MVEInstAlias<"vrmlalvh${vp}.s32\t$RdaLo, $RdaHi, $Qn, $Qm",
                  (MVE_VRMLALDAVHs32
                   tGPREven:$RdaLo, tGPROdd:$RdaHi,
                   MQPR:$Qn, MQPR:$Qm, vpred_n:$vp)>;
def : MVEInstAlias<"vrmlalvha${vp}.s32\t$RdaLo, $RdaHi, $Qn, $Qm",
                  (MVE_VRMLALDAVHas32
                   tGPREven:$RdaLo, tGPROdd:$RdaHi,
                   MQPR:$Qn, MQPR:$Qm, vpred_n:$vp)>;
def : MVEInstAlias<"vrmlalvh${vp}.u32\t$RdaLo, $RdaHi, $Qn, $Qm",
                  (MVE_VRMLALDAVHu32
                   tGPREven:$RdaLo, tGPROdd:$RdaHi,
                   MQPR:$Qn, MQPR:$Qm, vpred_n:$vp)>;
def : MVEInstAlias<"vrmlalvha${vp}.u32\t$RdaLo, $RdaHi, $Qn, $Qm",
                  (MVE_VRMLALDAVHau32
                   tGPREven:$RdaLo, tGPROdd:$RdaHi,
                   MQPR:$Qn, MQPR:$Qm, vpred_n:$vp)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1406-1411
```tablegen
multiclass MVE_VMLALDAV_multi<MVEVectorVTInfo VTI, list<dag> pattern=[]> {
  defm "" : MVE_VMLALDAVBase_AX<"vmlaldav", "s"#VTI.BitsSuffix,
                                VTI.Size{1}, 0b0, 0b0, 0b0, VTI.Size, pattern>;
  defm "" : MVE_VMLALDAVBase_A<"vmlaldav", "", "u"#VTI.BitsSuffix,
                               VTI.Size{1}, 0b1, 0b0, 0b0, 0b0, VTI.Size, pattern>;
}
```
- EN: Declares TableGen `multiclass MVE_VMLALDAV_multi`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMLALDAV_multi`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1413-1414
```tablegen
defm MVE_VMLALDAV : MVE_VMLALDAV_multi<MVE_v8i16>;
defm MVE_VMLALDAV : MVE_VMLALDAV_multi<MVE_v4i32>;
```
- EN: Defines TableGen record `MVE_VMLALDAV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMLALDAV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1416-1424
```tablegen
let Predicates = [HasMVEInt] in {
  def : Pat<(ARMVMLALVs (v4i32 MQPR:$val1), (v4i32 MQPR:$val2)),
            (MVE_VMLALDAVs32 (v4i32 MQPR:$val1), (v4i32 MQPR:$val2))>;
  def : Pat<(ARMVMLALVu (v4i32 MQPR:$val1), (v4i32 MQPR:$val2)),
            (MVE_VMLALDAVu32 (v4i32 MQPR:$val1), (v4i32 MQPR:$val2))>;
  def : Pat<(ARMVMLALVs (v8i16 MQPR:$val1), (v8i16 MQPR:$val2)),
            (MVE_VMLALDAVs16 (v8i16 MQPR:$val1), (v8i16 MQPR:$val2))>;
  def : Pat<(ARMVMLALVu (v8i16 MQPR:$val1), (v8i16 MQPR:$val2)),
            (MVE_VMLALDAVu16 (v8i16 MQPR:$val1), (v8i16 MQPR:$val2))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1426-1433
```tablegen
  def : Pat<(ARMVMLALVAs tGPREven:$Rda, tGPROdd:$Rdb, (v4i32 MQPR:$val1), (v4i32 MQPR:$val2)),
            (MVE_VMLALDAVas32 tGPREven:$Rda, tGPROdd:$Rdb, (v4i32 MQPR:$val1), (v4i32 MQPR:$val2))>;
  def : Pat<(ARMVMLALVAu tGPREven:$Rda, tGPROdd:$Rdb, (v4i32 MQPR:$val1), (v4i32 MQPR:$val2)),
            (MVE_VMLALDAVau32 tGPREven:$Rda, tGPROdd:$Rdb, (v4i32 MQPR:$val1), (v4i32 MQPR:$val2))>;
  def : Pat<(ARMVMLALVAs tGPREven:$Rda, tGPROdd:$Rdb, (v8i16 MQPR:$val1), (v8i16 MQPR:$val2)),
            (MVE_VMLALDAVas16 tGPREven:$Rda, tGPROdd:$Rdb, (v8i16 MQPR:$val1), (v8i16 MQPR:$val2))>;
  def : Pat<(ARMVMLALVAu tGPREven:$Rda, tGPROdd:$Rdb, (v8i16 MQPR:$val1), (v8i16 MQPR:$val2)),
            (MVE_VMLALDAVau16 tGPREven:$Rda, tGPROdd:$Rdb, (v8i16 MQPR:$val1), (v8i16 MQPR:$val2))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1435-1443
```tablegen
  // Predicated
  def : Pat<(ARMVMLALVps (v4i32 MQPR:$val1), (v4i32 MQPR:$val2), (v4i1 VCCR:$pred)),
            (MVE_VMLALDAVs32 (v4i32 MQPR:$val1), (v4i32 MQPR:$val2), ARMVCCThen, $pred, zero_reg)>;
  def : Pat<(ARMVMLALVpu (v4i32 MQPR:$val1), (v4i32 MQPR:$val2), (v4i1 VCCR:$pred)),
            (MVE_VMLALDAVu32 (v4i32 MQPR:$val1), (v4i32 MQPR:$val2), ARMVCCThen, $pred, zero_reg)>;
  def : Pat<(ARMVMLALVps (v8i16 MQPR:$val1), (v8i16 MQPR:$val2), (v8i1 VCCR:$pred)),
            (MVE_VMLALDAVs16 (v8i16 MQPR:$val1), (v8i16 MQPR:$val2), ARMVCCThen, $pred, zero_reg)>;
  def : Pat<(ARMVMLALVpu (v8i16 MQPR:$val1), (v8i16 MQPR:$val2), (v8i1 VCCR:$pred)),
            (MVE_VMLALDAVu16 (v8i16 MQPR:$val1), (v8i16 MQPR:$val2), ARMVCCThen, $pred, zero_reg)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1445-1453
```tablegen
  def : Pat<(ARMVMLALVAps tGPREven:$Rda, tGPROdd:$Rdb, (v4i32 MQPR:$val1), (v4i32 MQPR:$val2), (v4i1 VCCR:$pred)),
            (MVE_VMLALDAVas32 tGPREven:$Rda, tGPROdd:$Rdb, (v4i32 MQPR:$val1), (v4i32 MQPR:$val2), ARMVCCThen, $pred, zero_reg)>;
  def : Pat<(ARMVMLALVApu tGPREven:$Rda, tGPROdd:$Rdb, (v4i32 MQPR:$val1), (v4i32 MQPR:$val2), (v4i1 VCCR:$pred)),
            (MVE_VMLALDAVau32 tGPREven:$Rda, tGPROdd:$Rdb, (v4i32 MQPR:$val1), (v4i32 MQPR:$val2), ARMVCCThen, $pred, zero_reg)>;
  def : Pat<(ARMVMLALVAps tGPREven:$Rda, tGPROdd:$Rdb, (v8i16 MQPR:$val1), (v8i16 MQPR:$val2), (v8i1 VCCR:$pred)),
            (MVE_VMLALDAVas16 tGPREven:$Rda, tGPROdd:$Rdb, (v8i16 MQPR:$val1), (v8i16 MQPR:$val2), ARMVCCThen, $pred, zero_reg)>;
  def : Pat<(ARMVMLALVApu tGPREven:$Rda, tGPROdd:$Rdb, (v8i16 MQPR:$val1), (v8i16 MQPR:$val2), (v8i1 VCCR:$pred)),
            (MVE_VMLALDAVau16 tGPREven:$Rda, tGPROdd:$Rdb, (v8i16 MQPR:$val1), (v8i16 MQPR:$val2), ARMVCCThen, $pred, zero_reg)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1455-1464
```tablegen
// vmlalv aliases vmlaldav
foreach acc = ["", "a"] in {
  foreach suffix = ["s16", "s32", "u16", "u32"] in {
    def : MVEInstAlias<"vmlalv" # acc # "${vp}." # suffix #
                          "\t$RdaLoDest, $RdaHiDest, $Qn, $Qm",
                       (!cast<Instruction>("MVE_VMLALDAV"#acc#suffix)
                       tGPREven:$RdaLoDest, tGPROdd:$RdaHiDest,
                       MQPR:$Qn, MQPR:$Qm, vpred_n:$vp)>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1466-1469
```tablegen
multiclass MVE_VMLSLDAV_multi<string iname, string suffix, bit sz,
                              bit bit_28, bits<2> vecsize, list<dag> pattern=[]> {
  defm "" : MVE_VMLALDAVBase_AX<iname, suffix, sz, bit_28, 0b0, 0b1, vecsize, pattern>;
}
```
- EN: Declares TableGen `multiclass MVE_VMLSLDAV_multi`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMLSLDAV_multi`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1471-1473
```tablegen
defm MVE_VMLSLDAV   : MVE_VMLSLDAV_multi<"vmlsldav", "s16", 0b0, 0b0, 0b01>;
defm MVE_VMLSLDAV   : MVE_VMLSLDAV_multi<"vmlsldav", "s32", 0b1, 0b0, 0b10>;
defm MVE_VRMLSLDAVH : MVE_VMLSLDAV_multi<"vrmlsldavh", "s32", 0b0, 0b1, 0b10>;
```
- EN: Defines TableGen record `MVE_VMLSLDAV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMLSLDAV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1475-1475
```tablegen
// end of mve_rDest instructions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1477-1477
```tablegen
// start of mve_comp instructions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1479-1485
```tablegen
class MVE_comp<InstrItinClass itin, string iname, string suffix,
               string cstr, bits<2> vecsize, list<dag> pattern=[]>
  : MVE_p<(outs MQPR:$Qd), (ins MQPR:$Qn, MQPR:$Qm), itin, iname, suffix,
           "$Qd, $Qn, $Qm", vpred_r, cstr, vecsize, pattern> {
  bits<4> Qd;
  bits<4> Qn;
  bits<4> Qm;
```
- EN: Declares reusable TableGen class `MVE_comp` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_comp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1487-1497
```tablegen
  let Inst{22} = Qd{3};
  let Inst{19-17} = Qn{2-0};
  let Inst{16} = 0b0;
  let Inst{15-13} = Qd{2-0};
  let Inst{12} = 0b0;
  let Inst{10-9} = 0b11;
  let Inst{7} = Qn{3};
  let Inst{5} = Qm{3};
  let Inst{3-1} = Qm{2-0};
  let Inst{0} = 0b0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1499-1501
```tablegen
class MVE_VMINMAXNM<string iname, string suffix, bits<2> sz, bit bit_21,
                    list<dag> pattern=[]>
  : MVE_comp<NoItinerary, iname, suffix, "", sz, pattern> {
```
- EN: Declares reusable TableGen class `MVE_VMINMAXNM` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VMINMAXNM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1503-1511
```tablegen
  let Inst{28} = 0b1;
  let Inst{25-24} = 0b11;
  let Inst{23} = 0b0;
  let Inst{21} = bit_21;
  let Inst{20} = sz{0};
  let Inst{11} = 0b1;
  let Inst{8} = 0b1;
  let Inst{6} = 0b1;
  let Inst{4} = 0b1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1513-1515
```tablegen
  let Predicates = [HasMVEFloat];
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1517-1518
```tablegen
multiclass MVE_VMINMAXNM_m<string iname, bit bit_4, MVEVectorVTInfo VTI, SDPatternOperator Op, Intrinsic PredInt> {
  def "" : MVE_VMINMAXNM<iname, VTI.Suffix, VTI.Size, bit_4>;
```
- EN: Declares TableGen `multiclass MVE_VMINMAXNM_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMINMAXNM_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1520-1523
```tablegen
  let Predicates = [HasMVEFloat] in {
    defm : MVE_TwoOpPattern<VTI, Op, PredInt, (? (i32 0)), !cast<Instruction>(NAME)>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1525-1528
```tablegen
defm MVE_VMAXNMf32 : MVE_VMINMAXNM_m<"vmaxnm", 0b0, MVE_v4f32, vmaxnm, int_arm_mve_max_predicated>;
defm MVE_VMAXNMf16 : MVE_VMINMAXNM_m<"vmaxnm", 0b0, MVE_v8f16, vmaxnm, int_arm_mve_max_predicated>;
defm MVE_VMINNMf32 : MVE_VMINMAXNM_m<"vminnm", 0b1, MVE_v4f32, vminnm, int_arm_mve_min_predicated>;
defm MVE_VMINNMf16 : MVE_VMINMAXNM_m<"vminnm", 0b1, MVE_v8f16, vminnm, int_arm_mve_min_predicated>;
```
- EN: Defines TableGen record `MVE_VMAXNMf32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMAXNMf32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1531-1533
```tablegen
class MVE_VMINMAX<string iname, string suffix, bit U, bits<2> size,
              bit bit_4, list<dag> pattern=[]>
  : MVE_comp<NoItinerary, iname, suffix, "", size, pattern> {
```
- EN: Declares reusable TableGen class `MVE_VMINMAX` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VMINMAX`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1535-1544
```tablegen
  let Inst{28} = U;
  let Inst{25-24} = 0b11;
  let Inst{23} = 0b0;
  let Inst{21-20} = size{1-0};
  let Inst{11} = 0b0;
  let Inst{8} = 0b0;
  let Inst{6} = 0b1;
  let Inst{4} = bit_4;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1546-1548
```tablegen
multiclass MVE_VMINMAX_m<string iname, bit bit_4, MVEVectorVTInfo VTI,
                      SDNode Op, Intrinsic PredInt> {
  def "" : MVE_VMINMAX<iname, VTI.Suffix, VTI.Unsigned, VTI.Size, bit_4>;
```
- EN: Declares TableGen `multiclass MVE_VMINMAX_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMINMAX_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1550-1553
```tablegen
  let Predicates = [HasMVEInt] in {
    defm : MVE_TwoOpPattern<VTI, Op, PredInt, (? (i32 VTI.Unsigned)), !cast<Instruction>(NAME)>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1555-1558
```tablegen
multiclass MVE_VMAX<MVEVectorVTInfo VTI>
  : MVE_VMINMAX_m<"vmax", 0b0, VTI, !if(VTI.Unsigned, umax, smax), int_arm_mve_max_predicated>;
multiclass MVE_VMIN<MVEVectorVTInfo VTI>
  : MVE_VMINMAX_m<"vmin", 0b1, VTI, !if(VTI.Unsigned, umin, smin), int_arm_mve_min_predicated>;
```
- EN: Declares TableGen `multiclass MVE_VMAX`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMAX`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1560-1565
```tablegen
defm MVE_VMINs8   : MVE_VMIN<MVE_v16s8>;
defm MVE_VMINs16  : MVE_VMIN<MVE_v8s16>;
defm MVE_VMINs32  : MVE_VMIN<MVE_v4s32>;
defm MVE_VMINu8   : MVE_VMIN<MVE_v16u8>;
defm MVE_VMINu16  : MVE_VMIN<MVE_v8u16>;
defm MVE_VMINu32  : MVE_VMIN<MVE_v4u32>;
```
- EN: Defines TableGen record `MVE_VMINs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMINs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1567-1572
```tablegen
defm MVE_VMAXs8   : MVE_VMAX<MVE_v16s8>;
defm MVE_VMAXs16  : MVE_VMAX<MVE_v8s16>;
defm MVE_VMAXs32  : MVE_VMAX<MVE_v4s32>;
defm MVE_VMAXu8   : MVE_VMAX<MVE_v16u8>;
defm MVE_VMAXu16  : MVE_VMAX<MVE_v8u16>;
defm MVE_VMAXu32  : MVE_VMAX<MVE_v4u32>;
```
- EN: Defines TableGen record `MVE_VMAXs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMAXs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1574-1574
```tablegen
// end of mve_comp instructions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1576-1576
```tablegen
// start of mve_bit instructions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1578-1582
```tablegen
class MVE_bit_arith<dag oops, dag iops, string iname, string suffix,
                    string ops, string cstr, bits<2> vecsize, list<dag> pattern=[]>
  : MVE_p<oops, iops, NoItinerary, iname, suffix, ops, vpred_r, cstr, vecsize, pattern> {
  bits<4> Qd;
  bits<4> Qm;
```
- EN: Declares reusable TableGen class `MVE_bit_arith` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_bit_arith`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1584-1588
```tablegen
  let Inst{22} = Qd{3};
  let Inst{15-13} = Qd{2-0};
  let Inst{5} = Qm{3};
  let Inst{3-1} = Qm{2-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1590-1592
```tablegen
def MVE_VBIC : MVE_bit_arith<(outs MQPR:$Qd), (ins MQPR:$Qn, MQPR:$Qm),
                             "vbic", "", "$Qd, $Qn, $Qm", "", 0b00> {
  bits<4> Qn;
```
- EN: Defines TableGen record `MVE_VBIC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VBIC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1594-1605
```tablegen
  let Inst{28} = 0b0;
  let Inst{25-23} = 0b110;
  let Inst{21-20} = 0b01;
  let Inst{19-17} = Qn{2-0};
  let Inst{16} = 0b0;
  let Inst{12-8} = 0b00001;
  let Inst{7} = Qn{3};
  let Inst{6} = 0b1;
  let Inst{4} = 0b1;
  let Inst{0} = 0b0;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1607-1610
```tablegen
class MVE_VREV<string iname, string suffix, bits<2> size, bits<2> bit_8_7,
               bits<2> vecsize, string cstr="">
  : MVE_bit_arith<(outs MQPR:$Qd), (ins MQPR:$Qm), iname,
                  suffix, "$Qd, $Qm", cstr, vecsize> {
```
- EN: Declares reusable TableGen class `MVE_VREV` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VREV`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1612-1622
```tablegen
  let Inst{28} = 0b1;
  let Inst{25-23} = 0b111;
  let Inst{21-20} = 0b11;
  let Inst{19-18} = size;
  let Inst{17-16} = 0b00;
  let Inst{12-9} = 0b0000;
  let Inst{8-7} = bit_8_7;
  let Inst{6} = 0b1;
  let Inst{4} = 0b0;
  let Inst{0} = 0b0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1624-1626
```tablegen
def MVE_VREV64_8  : MVE_VREV<"vrev64", "8", 0b00, 0b00, 0b11, "@earlyclobber $Qd">;
def MVE_VREV64_16 : MVE_VREV<"vrev64", "16", 0b01, 0b00, 0b11, "@earlyclobber $Qd">;
def MVE_VREV64_32 : MVE_VREV<"vrev64", "32", 0b10, 0b00, 0b11, "@earlyclobber $Qd">;
```
- EN: Defines TableGen record `MVE_VREV64_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VREV64_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1628-1629
```tablegen
def MVE_VREV32_8  : MVE_VREV<"vrev32", "8", 0b00, 0b01, 0b10>;
def MVE_VREV32_16 : MVE_VREV<"vrev32", "16", 0b01, 0b01, 0b10>;
```
- EN: Defines TableGen record `MVE_VREV32_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VREV32_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1631-1631
```tablegen
def MVE_VREV16_8  : MVE_VREV<"vrev16", "8", 0b00, 0b10, 0b01>;
```
- EN: Defines TableGen record `MVE_VREV16_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VREV16_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1633-1638
```tablegen
let Predicates = [HasMVEInt] in {
  def : Pat<(v8i16 (bswap (v8i16 MQPR:$src))),
            (v8i16 (MVE_VREV16_8 (v8i16 MQPR:$src)))>;
  def : Pat<(v4i32 (bswap (v4i32 MQPR:$src))),
            (v4i32 (MVE_VREV32_8 (v4i32 MQPR:$src)))>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1640-1642
```tablegen
multiclass MVE_VREV_basic_patterns<int revbits, list<MVEVectorVTInfo> VTIs,
                                   Instruction Inst> {
  defvar unpred_op = !cast<SDNode>("ARMvrev" # revbits);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 1644-1652
```tablegen
  foreach VTI = VTIs in {
    def : Pat<(VTI.Vec (unpred_op (VTI.Vec MQPR:$src))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$src)))>;
    def : Pat<(VTI.Vec (int_arm_mve_vrev_predicated (VTI.Vec MQPR:$src),
                  revbits, (VTI.Pred VCCR:$pred), (VTI.Vec MQPR:$inactive))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$src), ARMVCCThen,
                  (VTI.Pred VCCR:$pred), zero_reg, (VTI.Vec MQPR:$inactive)))>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1654-1657
```tablegen
let Predicates = [HasMVEInt] in {
  defm: MVE_VREV_basic_patterns<64, [MVE_v4i32, MVE_v4f32], MVE_VREV64_32>;
  defm: MVE_VREV_basic_patterns<64, [MVE_v8i16, MVE_v8f16], MVE_VREV64_16>;
  defm: MVE_VREV_basic_patterns<64, [MVE_v16i8           ], MVE_VREV64_8>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1659-1660
```tablegen
  defm: MVE_VREV_basic_patterns<32, [MVE_v8i16, MVE_v8f16], MVE_VREV32_16>;
  defm: MVE_VREV_basic_patterns<32, [MVE_v16i8           ], MVE_VREV32_8>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1662-1663
```tablegen
  defm: MVE_VREV_basic_patterns<16, [MVE_v16i8           ], MVE_VREV16_8>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1665-1674
```tablegen
def MVE_VMVN : MVE_bit_arith<(outs MQPR:$Qd), (ins MQPR:$Qm),
                             "vmvn", "", "$Qd, $Qm", "", 0b00> {
  let Inst{28} = 0b1;
  let Inst{25-23} = 0b111;
  let Inst{21-16} = 0b110000;
  let Inst{12-6} = 0b0010111;
  let Inst{4} = 0b0;
  let Inst{0} = 0b0;
  let validForTailPredication = 1;
}
```
- EN: Defines TableGen record `MVE_VMVN` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMVN`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1676-1685
```tablegen
let Predicates = [HasMVEInt] in {
  foreach VTI = [ MVE_v16i8, MVE_v8i16, MVE_v4i32, MVE_v2i64 ] in {
    def : Pat<(VTI.Vec (vnotq    (VTI.Vec MQPR:$val1))),
              (VTI.Vec (MVE_VMVN (VTI.Vec MQPR:$val1)))>;
    def : Pat<(VTI.Vec (int_arm_mve_mvn_predicated (VTI.Vec MQPR:$val1),
                       (VTI.Pred VCCR:$pred), (VTI.Vec MQPR:$inactive))),
              (VTI.Vec (MVE_VMVN (VTI.Vec MQPR:$val1), ARMVCCThen,
                       (VTI.Pred VCCR:$pred), zero_reg, (VTI.Vec MQPR:$inactive)))>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1687-1690
```tablegen
class MVE_bit_ops<string iname, bits<2> bit_21_20, bit bit_28>
  : MVE_bit_arith<(outs MQPR:$Qd), (ins MQPR:$Qn, MQPR:$Qm),
                  iname, "", "$Qd, $Qn, $Qm", "", 0b00> {
  bits<4> Qn;
```
- EN: Declares reusable TableGen class `MVE_bit_ops` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_bit_ops`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1692-1703
```tablegen
  let Inst{28} = bit_28;
  let Inst{25-23} = 0b110;
  let Inst{21-20} = bit_21_20;
  let Inst{19-17} = Qn{2-0};
  let Inst{16} = 0b0;
  let Inst{12-8} = 0b00001;
  let Inst{7} = Qn{3};
  let Inst{6} = 0b1;
  let Inst{4} = 0b1;
  let Inst{0} = 0b0;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1705-1708
```tablegen
def MVE_VEOR : MVE_bit_ops<"veor", 0b00, 0b1>;
def MVE_VORN : MVE_bit_ops<"vorn", 0b11, 0b0>;
def MVE_VORR : MVE_bit_ops<"vorr", 0b10, 0b0>;
def MVE_VAND : MVE_bit_ops<"vand", 0b00, 0b0>;
```
- EN: Defines TableGen record `MVE_VEOR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VEOR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1710-1710
```tablegen
// add ignored suffixes as aliases
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1712-1723
```tablegen
foreach s=["s8", "s16", "s32", "u8", "u16", "u32", "i8", "i16", "i32", "f16", "f32"] in {
  def : MVEInstAlias<"vbic${vp}." # s # "\t$QdSrc, $QnSrc, $QmSrc",
        (MVE_VBIC MQPR:$QdSrc, MQPR:$QnSrc, MQPR:$QmSrc, vpred_r:$vp)>;
  def : MVEInstAlias<"veor${vp}." # s # "\t$QdSrc, $QnSrc, $QmSrc",
        (MVE_VEOR MQPR:$QdSrc, MQPR:$QnSrc, MQPR:$QmSrc, vpred_r:$vp)>;
  def : MVEInstAlias<"vorn${vp}." # s # "\t$QdSrc, $QnSrc, $QmSrc",
        (MVE_VORN MQPR:$QdSrc, MQPR:$QnSrc, MQPR:$QmSrc, vpred_r:$vp)>;
  def : MVEInstAlias<"vorr${vp}." # s # "\t$QdSrc, $QnSrc, $QmSrc",
        (MVE_VORR MQPR:$QdSrc, MQPR:$QnSrc, MQPR:$QmSrc, vpred_r:$vp)>;
  def : MVEInstAlias<"vand${vp}." # s # "\t$QdSrc, $QnSrc, $QmSrc",
        (MVE_VAND MQPR:$QdSrc, MQPR:$QnSrc, MQPR:$QmSrc, vpred_r:$vp)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1725-1729
```tablegen
let Predicates = [HasMVEInt] in {
  defm : MVE_TwoOpPattern<MVE_v16i8, and, int_arm_mve_and_predicated, (? ), MVE_VAND, ARMimmAllOnesV>;
  defm : MVE_TwoOpPattern<MVE_v8i16, and, int_arm_mve_and_predicated, (? ), MVE_VAND, ARMimmAllOnesV>;
  defm : MVE_TwoOpPattern<MVE_v4i32, and, int_arm_mve_and_predicated, (? ), MVE_VAND, ARMimmAllOnesV>;
  defm : MVE_TwoOpPattern<MVE_v2i64, and, int_arm_mve_and_predicated, (? ), MVE_VAND, ARMimmAllOnesV>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1731-1734
```tablegen
  defm : MVE_TwoOpPattern<MVE_v16i8, or, int_arm_mve_orr_predicated, (? ), MVE_VORR, ARMimmAllZerosV>;
  defm : MVE_TwoOpPattern<MVE_v8i16, or, int_arm_mve_orr_predicated, (? ), MVE_VORR, ARMimmAllZerosV>;
  defm : MVE_TwoOpPattern<MVE_v4i32, or, int_arm_mve_orr_predicated, (? ), MVE_VORR, ARMimmAllZerosV>;
  defm : MVE_TwoOpPattern<MVE_v2i64, or, int_arm_mve_orr_predicated, (? ), MVE_VORR, ARMimmAllZerosV>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1736-1739
```tablegen
  defm : MVE_TwoOpPattern<MVE_v16i8, xor, int_arm_mve_eor_predicated, (? ), MVE_VEOR, ARMimmAllZerosV>;
  defm : MVE_TwoOpPattern<MVE_v8i16, xor, int_arm_mve_eor_predicated, (? ), MVE_VEOR, ARMimmAllZerosV>;
  defm : MVE_TwoOpPattern<MVE_v4i32, xor, int_arm_mve_eor_predicated, (? ), MVE_VEOR, ARMimmAllZerosV>;
  defm : MVE_TwoOpPattern<MVE_v2i64, xor, int_arm_mve_eor_predicated, (? ), MVE_VEOR, ARMimmAllZerosV>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1741-1748
```tablegen
  defm : MVE_TwoOpPattern<MVE_v16i8, BinOpFrag<(and node:$LHS, (vnotq node:$RHS))>,
                          int_arm_mve_bic_predicated, (? ), MVE_VBIC>;
  defm : MVE_TwoOpPattern<MVE_v8i16, BinOpFrag<(and node:$LHS, (vnotq node:$RHS))>,
                          int_arm_mve_bic_predicated, (? ), MVE_VBIC>;
  defm : MVE_TwoOpPattern<MVE_v4i32, BinOpFrag<(and node:$LHS, (vnotq node:$RHS))>,
                          int_arm_mve_bic_predicated, (? ), MVE_VBIC>;
  defm : MVE_TwoOpPattern<MVE_v2i64, BinOpFrag<(and node:$LHS, (vnotq node:$RHS))>,
                          int_arm_mve_bic_predicated, (? ), MVE_VBIC>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1750-1758
```tablegen
  defm : MVE_TwoOpPattern<MVE_v16i8, BinOpFrag<(or node:$LHS, (vnotq node:$RHS))>,
                          int_arm_mve_orn_predicated, (? ), MVE_VORN>;
  defm : MVE_TwoOpPattern<MVE_v8i16, BinOpFrag<(or node:$LHS, (vnotq node:$RHS))>,
                          int_arm_mve_orn_predicated, (? ), MVE_VORN>;
  defm : MVE_TwoOpPattern<MVE_v4i32, BinOpFrag<(or node:$LHS, (vnotq node:$RHS))>,
                          int_arm_mve_orn_predicated, (? ), MVE_VORN>;
  defm : MVE_TwoOpPattern<MVE_v2i64, BinOpFrag<(or node:$LHS, (vnotq node:$RHS))>,
                          int_arm_mve_orn_predicated, (? ), MVE_VORN>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1760-1764
```tablegen
class MVE_bit_cmode<string iname, string suffix, bit halfword, dag inOps, bits<2> vecsize>
  : MVE_p<(outs MQPR:$Qd), inOps, NoItinerary,
          iname, suffix, "$Qd, $imm", vpred_n, "$Qd = $Qd_src", vecsize> {
  bits<12> imm;
  bits<4> Qd;
```
- EN: Declares reusable TableGen class `MVE_bit_cmode` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_bit_cmode`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1766-1780
```tablegen
  let Inst{28} = imm{7};
  let Inst{27-23} = 0b11111;
  let Inst{22} = Qd{3};
  let Inst{21-19} = 0b000;
  let Inst{18-16} = imm{6-4};
  let Inst{15-13} = Qd{2-0};
  let Inst{12} = 0b0;
  let Inst{11} = halfword;
  let Inst{10} = !if(halfword, 0, imm{10});
  let Inst{9} = imm{9};
  let Inst{8} = 0b1;
  let Inst{7-6} = 0b01;
  let Inst{4} = 0b1;
  let Inst{3-0} = imm{3-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1782-1788
```tablegen
multiclass MVE_bit_cmode_p<string iname, bit opcode,
                           MVEVectorVTInfo VTI, Operand imm_type, SDNode op> {
  def "" : MVE_bit_cmode<iname, VTI.Suffix, VTI.Size{0},
                         (ins MQPR:$Qd_src, imm_type:$imm), VTI.Size> {
    let Inst{5} = opcode;
    let validForTailPredication = 1;
  }
```
- EN: Declares TableGen `multiclass MVE_bit_cmode_p`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_bit_cmode_p`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1790-1791
```tablegen
  defvar Inst = !cast<Instruction>(NAME);
  defvar UnpredPat = (VTI.Vec (op (VTI.Vec MQPR:$src), timm:$simm));
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 1793-1801
```tablegen
  let Predicates = [HasMVEInt] in {
    def : Pat<UnpredPat,
              (VTI.Vec (Inst (VTI.Vec MQPR:$src), imm_type:$simm))>;
    def : Pat<(VTI.Vec (vselect (VTI.Pred VCCR:$pred),
                          UnpredPat, (VTI.Vec MQPR:$src))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$src), imm_type:$simm,
                             ARMVCCThen, (VTI.Pred VCCR:$pred), zero_reg))>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1803-1808
```tablegen
multiclass MVE_VORRimm<MVEVectorVTInfo VTI, Operand imm_type> {
  defm "": MVE_bit_cmode_p<"vorr", 0, VTI, imm_type, ARMvorrImm>;
}
multiclass MVE_VBICimm<MVEVectorVTInfo VTI, Operand imm_type> {
  defm "": MVE_bit_cmode_p<"vbic", 1, VTI, imm_type, ARMvbicImm>;
}
```
- EN: Declares TableGen `multiclass MVE_VORRimm`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VORRimm`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1810-1813
```tablegen
defm MVE_VORRimmi16 : MVE_VORRimm<MVE_v8i16, nImmSplatI16>;
defm MVE_VORRimmi32 : MVE_VORRimm<MVE_v4i32, nImmSplatI32>;
defm MVE_VBICimmi16 : MVE_VBICimm<MVE_v8i16, nImmSplatI16>;
defm MVE_VBICimmi32 : MVE_VBICimm<MVE_v4i32, nImmSplatI32>;
```
- EN: Defines TableGen record `MVE_VORRimmi16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VORRimmi16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1815-1818
```tablegen
def MVE_VORNimmi16 : MVEInstAlias<"vorn${vp}.i16\t$Qd, $imm",
    (MVE_VORRimmi16 MQPR:$Qd, nImmSplatNotI16:$imm, vpred_n:$vp), 0>;
def MVE_VORNimmi32 : MVEInstAlias<"vorn${vp}.i32\t$Qd, $imm",
    (MVE_VORRimmi32 MQPR:$Qd, nImmSplatNotI32:$imm, vpred_n:$vp), 0>;
```
- EN: Defines TableGen record `MVE_VORNimmi16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VORNimmi16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1820-1823
```tablegen
def MVE_VANDimmi16 : MVEInstAlias<"vand${vp}.i16\t$Qd, $imm",
    (MVE_VBICimmi16 MQPR:$Qd, nImmSplatNotI16:$imm, vpred_n:$vp), 0>;
def MVE_VANDimmi32 : MVEInstAlias<"vand${vp}.i32\t$Qd, $imm",
    (MVE_VBICimmi32 MQPR:$Qd, nImmSplatNotI32:$imm, vpred_n:$vp), 0>;
```
- EN: Defines TableGen record `MVE_VANDimmi16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VANDimmi16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1825-1826
```tablegen
def MVE_VMOV : MVEInstAlias<"vmov${vp}\t$Qd, $Qm",
    (MVE_VORR MQPR:$Qd, MQPR:$Qm, MQPR:$Qm, vpred_r:$vp)>;
```
- EN: Defines TableGen record `MVE_VMOV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMOV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1828-1845
```tablegen
class MVE_VMOV_lane_direction {
  bit bit_20;
  dag oops;
  dag iops;
  string ops;
  string cstr;
}
def MVE_VMOV_from_lane : MVE_VMOV_lane_direction {
  let bit_20 = 0b1;
  let oops = (outs rGPR:$Rt);
  let iops = (ins MQPR:$Qd);
  let ops = "$Rt, $Qd$Idx";
  let cstr = "";
}
def MVE_VMOV_to_lane : MVE_VMOV_lane_direction {
  let bit_20 = 0b0;
  let oops = (outs MQPR:$Qd);
  let iops = (ins MQPR:$Qd_src, rGPR:$Rt);
```
- EN: Declares reusable TableGen class `MVE_VMOV_lane_direction` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VMOV_lane_direction`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1846-1848
```tablegen
  let ops = "$Qd$Idx, $Rt";
  let cstr = "$Qd = $Qd_src";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1850-1855
```tablegen
class MVE_VMOV_lane<string suffix, bit U, dag indexop,
                    MVE_VMOV_lane_direction dir>
  : MVE_VMOV_lane_base<dir.oops, !con(dir.iops, indexop), NoItinerary,
                       "vmov", suffix, dir.ops, dir.cstr, []> {
  bits<4> Qd;
  bits<4> Rt;
```
- EN: Declares reusable TableGen class `MVE_VMOV_lane` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VMOV_lane`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1857-1864
```tablegen
  let Inst{31-24} = 0b11101110;
  let Inst{23} = U;
  let Inst{20} = dir.bit_20;
  let Inst{19-17} = Qd{2-0};
  let Inst{15-12} = Rt{3-0};
  let Inst{11-8} = 0b1011;
  let Inst{7} = Qd{3};
  let Inst{4-0} = 0b10000;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1866-1867
```tablegen
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1869-1875
```tablegen
class MVE_VMOV_lane_32<MVE_VMOV_lane_direction dir>
    : MVE_VMOV_lane<"32", 0b0, (ins MVEVectorIndex<4>:$Idx), dir> {
  bits<2> Idx;
  let Inst{22} = 0b0;
  let Inst{6-5} = 0b00;
  let Inst{16} = Idx{1};
  let Inst{21} = Idx{0};
```
- EN: Declares reusable TableGen class `MVE_VMOV_lane_32` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VMOV_lane_32`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1877-1879
```tablegen
  let VecSize = 0b10;
  let Predicates = [HasFPRegsV8_1M];
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1881-1888
```tablegen
class MVE_VMOV_lane_16<string suffix, bit U, MVE_VMOV_lane_direction dir>
  : MVE_VMOV_lane<suffix, U, (ins MVEVectorIndex<8>:$Idx), dir> {
  bits<3> Idx;
  let Inst{22} = 0b0;
  let Inst{5} = 0b1;
  let Inst{16} = Idx{2};
  let Inst{21} = Idx{1};
  let Inst{6} = Idx{0};
```
- EN: Declares reusable TableGen class `MVE_VMOV_lane_16` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VMOV_lane_16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1890-1891
```tablegen
  let VecSize = 0b01;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1893-1900
```tablegen
class MVE_VMOV_lane_8<string suffix, bit U, MVE_VMOV_lane_direction dir>
  : MVE_VMOV_lane<suffix, U, (ins MVEVectorIndex<16>:$Idx), dir> {
  bits<4> Idx;
  let Inst{22} = 0b1;
  let Inst{16} = Idx{3};
  let Inst{21} = Idx{2};
  let Inst{6} = Idx{1};
  let Inst{5} = Idx{0};
```
- EN: Declares reusable TableGen class `MVE_VMOV_lane_8` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VMOV_lane_8`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1902-1903
```tablegen
  let VecSize = 0b00;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1905-1913
```tablegen
def MVE_VMOV_from_lane_32  : MVE_VMOV_lane_32<            MVE_VMOV_from_lane>;
def MVE_VMOV_from_lane_s16 : MVE_VMOV_lane_16<"s16", 0b0, MVE_VMOV_from_lane>;
def MVE_VMOV_from_lane_u16 : MVE_VMOV_lane_16<"u16", 0b1, MVE_VMOV_from_lane>;
def MVE_VMOV_from_lane_s8  : MVE_VMOV_lane_8 < "s8", 0b0, MVE_VMOV_from_lane>;
def MVE_VMOV_from_lane_u8  : MVE_VMOV_lane_8 < "u8", 0b1, MVE_VMOV_from_lane>;
let isInsertSubreg = 1 in
def MVE_VMOV_to_lane_32    : MVE_VMOV_lane_32<            MVE_VMOV_to_lane>;
def MVE_VMOV_to_lane_16    : MVE_VMOV_lane_16< "16", 0b0, MVE_VMOV_to_lane>;
def MVE_VMOV_to_lane_8     : MVE_VMOV_lane_8 <  "8", 0b0, MVE_VMOV_to_lane>;
```
- EN: Defines TableGen record `MVE_VMOV_from_lane_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMOV_from_lane_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1915-1920
```tablegen
// This is the same as insertelt but allows the inserted value to be an i32 as
// will be used when it is the only legal type.
def ARMVecInsert : SDTypeProfile<1, 3, [
  SDTCisVT<2, i32>, SDTCisSameAs<0, 1>, SDTCisPtrTy<3>
]>;
def ARMinsertelt  : SDNode<"ISD::INSERT_VECTOR_ELT", ARMVecInsert>;
```
- EN: Defines TableGen record `ARMVecInsert` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMVecInsert`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 1922-1926
```tablegen
let Predicates = [HasMVEInt] in {
  def : Pat<(extractelt (v2f64 MQPR:$src), imm:$lane),
            (f64 (EXTRACT_SUBREG MQPR:$src, (DSubReg_f64_reg imm:$lane)))>;
  def : Pat<(insertelt (v2f64 MQPR:$src1), DPR:$src2, imm:$lane),
            (INSERT_SUBREG (v2f64 (COPY_TO_REGCLASS MQPR:$src1, MQPR)), DPR:$src2, (DSubReg_f64_reg imm:$lane))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1928-1940
```tablegen
  def : Pat<(extractelt (v4i32 MQPR:$src), imm:$lane),
            (COPY_TO_REGCLASS
              (i32 (EXTRACT_SUBREG MQPR:$src, (SSubReg_f32_reg imm:$lane))), rGPR)>;
  def : Pat<(insertelt (v4i32 MQPR:$src1), rGPR:$src2, imm:$lane),
            (MVE_VMOV_to_lane_32 MQPR:$src1, rGPR:$src2, imm:$lane)>;
  // This tries to copy from one lane to another, without going via GPR regs
  def : Pat<(insertelt (v4i32 MQPR:$src1), (extractelt (v4i32 MQPR:$src2), imm:$extlane), imm:$inslane),
            (v4i32 (COPY_TO_REGCLASS
                     (INSERT_SUBREG (v4f32 (COPY_TO_REGCLASS (v4i32 MQPR:$src1), MQPR)),
                                    (f32 (EXTRACT_SUBREG (v4f32 (COPY_TO_REGCLASS (v4i32 MQPR:$src2), MQPR)),
                                                         (SSubReg_f32_reg imm:$extlane))),
                                    (SSubReg_f32_reg imm:$inslane)),
                      MQPR))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1942-1945
```tablegen
  def : Pat<(vector_insert (v16i8 MQPR:$src1), rGPR:$src2, imm:$lane),
            (MVE_VMOV_to_lane_8  MQPR:$src1, rGPR:$src2, imm:$lane)>;
  def : Pat<(vector_insert (v8i16 MQPR:$src1), rGPR:$src2, imm:$lane),
            (MVE_VMOV_to_lane_16 MQPR:$src1, rGPR:$src2, imm:$lane)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1947-1964
```tablegen
  def : Pat<(ARMvgetlanes (v16i8 MQPR:$src), imm:$lane),
            (MVE_VMOV_from_lane_s8 MQPR:$src, imm:$lane)>;
  def : Pat<(ARMvgetlanes (v8i16 MQPR:$src), imm:$lane),
            (MVE_VMOV_from_lane_s16 MQPR:$src, imm:$lane)>;
  def : Pat<(ARMvgetlanes (v8f16 MQPR:$src), imm:$lane),
            (MVE_VMOV_from_lane_s16 MQPR:$src, imm:$lane)>;
  def : Pat<(ARMvgetlaneu (v16i8 MQPR:$src), imm:$lane),
            (MVE_VMOV_from_lane_u8 MQPR:$src, imm:$lane)>;
  def : Pat<(ARMvgetlaneu (v8i16 MQPR:$src), imm:$lane),
            (MVE_VMOV_from_lane_u16 MQPR:$src, imm:$lane)>;
  def : Pat<(ARMvgetlaneu (v8f16 MQPR:$src), imm:$lane),
            (MVE_VMOV_from_lane_u16 MQPR:$src, imm:$lane)>;
  // For i16's inserts being extracted from low lanes, then may use VINS.
  let Predicates = [HasFullFP16] in {
  def : Pat<(ARMinsertelt (v8i16 MQPR:$src1),
                          (ARMvgetlaneu (v8i16 MQPR:$src2), imm_even:$extlane),
                          imm_odd:$inslane),
            (COPY_TO_REGCLASS (INSERT_SUBREG (v4f32 (COPY_TO_REGCLASS MQPR:$src1, MQPR)),
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1965-1968
```tablegen
                                (VINSH (EXTRACT_SUBREG MQPR:$src1, (SSubReg_f16_reg imm_odd:$inslane)),
                                       (EXTRACT_SUBREG MQPR:$src2, (SSubReg_f16_reg imm_even:$extlane))),
                                (SSubReg_f16_reg imm_odd:$inslane)), MQPR)>;
  }
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1970-1975
```tablegen
  def : Pat<(v16i8 (scalar_to_vector GPR:$src)),
            (MVE_VMOV_to_lane_8  (v16i8 (IMPLICIT_DEF)), rGPR:$src, (i32 0))>;
  def : Pat<(v8i16 (scalar_to_vector GPR:$src)),
            (MVE_VMOV_to_lane_16 (v8i16 (IMPLICIT_DEF)), rGPR:$src, (i32 0))>;
  def : Pat<(v4i32 (scalar_to_vector GPR:$src)),
            (MVE_VMOV_to_lane_32 (v4i32 (IMPLICIT_DEF)), rGPR:$src, (i32 0))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1977-1981
```tablegen
  // Floating point patterns, still enabled under HasMVEInt
  def : Pat<(extractelt (v4f32 MQPR:$src), imm:$lane),
            (COPY_TO_REGCLASS (f32 (EXTRACT_SUBREG MQPR:$src, (SSubReg_f32_reg imm:$lane))), SPR)>;
  def : Pat<(insertelt (v4f32 MQPR:$src1), (f32 SPR:$src2), imm:$lane),
            (INSERT_SUBREG (v4f32 (COPY_TO_REGCLASS MQPR:$src1, MQPR)), SPR:$src2, (SSubReg_f32_reg imm:$lane))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1983-1999
```tablegen
  def : Pat<(insertelt (v8f16 MQPR:$src1), (f16 HPR:$src2), imm_even:$lane),
            (MVE_VMOV_to_lane_16 MQPR:$src1, (COPY_TO_REGCLASS (f16 HPR:$src2), rGPR), imm:$lane)>;
  let Predicates = [HasFullFP16] in {
  def : Pat<(insertelt (v8f16 MQPR:$src1), (f16 HPR:$src2), imm_odd:$lane),
            (COPY_TO_REGCLASS (INSERT_SUBREG (v4f32 (COPY_TO_REGCLASS MQPR:$src1, MQPR)),
                                (VINSH (EXTRACT_SUBREG MQPR:$src1, (SSubReg_f16_reg imm_odd:$lane)),
                                       (COPY_TO_REGCLASS HPR:$src2, SPR)),
                                (SSubReg_f16_reg imm_odd:$lane)), MQPR)>;
  }
  def : Pat<(extractelt (v8f16 MQPR:$src), imm_even:$lane),
            (EXTRACT_SUBREG MQPR:$src, (SSubReg_f16_reg imm_even:$lane))>;
  let Predicates = [HasFullFP16] in {
  def : Pat<(extractelt (v8f16 MQPR:$src), imm_odd:$lane),
            (COPY_TO_REGCLASS
              (VMOVH (EXTRACT_SUBREG MQPR:$src, (SSubReg_f16_reg imm_odd:$lane))),
              HPR)>;
  }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2001-2011
```tablegen
  def : Pat<(v2f64 (scalar_to_vector (f64 DPR:$src))),
            (INSERT_SUBREG (v2f64 (IMPLICIT_DEF)), DPR:$src, dsub_0)>;
  def : Pat<(v4f32 (scalar_to_vector SPR:$src)),
            (INSERT_SUBREG (v4f32 (IMPLICIT_DEF)), SPR:$src, ssub_0)>;
  def : Pat<(v4f32 (scalar_to_vector GPR:$src)),
            (MVE_VMOV_to_lane_32 (v4f32 (IMPLICIT_DEF)), rGPR:$src, (i32 0))>;
  def : Pat<(v8f16 (scalar_to_vector (f16 HPR:$src))),
            (INSERT_SUBREG (v8f16 (IMPLICIT_DEF)), (f16 HPR:$src), ssub_0)>;
  def : Pat<(v8f16 (scalar_to_vector GPR:$src)),
            (MVE_VMOV_to_lane_16 (v8f16 (IMPLICIT_DEF)), rGPR:$src, (i32 0))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2013-2013
```tablegen
// end of mve_bit instructions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 2015-2015
```tablegen
// start of MVE Integer instructions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 2017-2022
```tablegen
class MVE_int<string iname, string suffix, bits<2> size, list<dag> pattern=[]>
  : MVE_p<(outs MQPR:$Qd), (ins MQPR:$Qn, MQPR:$Qm), NoItinerary,
          iname, suffix, "$Qd, $Qn, $Qm", vpred_r, "", size, pattern> {
  bits<4> Qd;
  bits<4> Qn;
  bits<4> Qm;
```
- EN: Declares reusable TableGen class `MVE_int` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_int`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2024-2032
```tablegen
  let Inst{22} = Qd{3};
  let Inst{21-20} = size;
  let Inst{19-17} = Qn{2-0};
  let Inst{15-13} = Qd{2-0};
  let Inst{7} = Qn{3};
  let Inst{6} = 0b1;
  let Inst{5} = Qm{3};
  let Inst{3-1} = Qm{2-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2034-2036
```tablegen
class MVE_VMULt1<string iname, string suffix, bits<2> size,
                   list<dag> pattern=[]>
  : MVE_int<iname, suffix, size, pattern> {
```
- EN: Declares reusable TableGen class `MVE_VMULt1` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VMULt1`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2038-2045
```tablegen
  let Inst{28} = 0b0;
  let Inst{25-23} = 0b110;
  let Inst{16} = 0b0;
  let Inst{12-8} = 0b01001;
  let Inst{4} = 0b1;
  let Inst{0} = 0b0;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2047-2048
```tablegen
multiclass MVE_VMUL_m<MVEVectorVTInfo VTI> {
  def "" : MVE_VMULt1<"vmul", VTI.Suffix, VTI.Size>;
```
- EN: Declares TableGen `multiclass MVE_VMUL_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMUL_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2050-2054
```tablegen
  let Predicates = [HasMVEInt] in {
    defm : MVE_TwoOpPattern<VTI, mul, int_arm_mve_mul_predicated, (? ),
                            !cast<Instruction>(NAME), ARMimmOneV>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2056-2058
```tablegen
defm MVE_VMULi8  : MVE_VMUL_m<MVE_v16i8>;
defm MVE_VMULi16 : MVE_VMUL_m<MVE_v8i16>;
defm MVE_VMULi32 : MVE_VMUL_m<MVE_v4i32>;
```
- EN: Defines TableGen record `MVE_VMULi8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMULi8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2060-2062
```tablegen
class MVE_VQxDMULH_Base<string iname, string suffix, bits<2> size, bit rounding,
                  list<dag> pattern=[]>
  : MVE_int<iname, suffix, size, pattern> {
```
- EN: Declares reusable TableGen class `MVE_VQxDMULH_Base` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VQxDMULH_Base`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2064-2071
```tablegen
  let Inst{28} = rounding;
  let Inst{25-23} = 0b110;
  let Inst{16} = 0b0;
  let Inst{12-8} = 0b01011;
  let Inst{4} = 0b0;
  let Inst{0} = 0b0;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2073-2074
```tablegen
// MVE vqdmulh instruction
def MVEvqdmulh : SDNode<"ARMISD::VQDMULH", SDTIntBinOp>;
```
- EN: Defines TableGen record `MVEvqdmulh` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MVEvqdmulh`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 2076-2080
```tablegen
multiclass MVE_VQxDMULH_m<string iname, MVEVectorVTInfo VTI,
                      SDPatternOperator Op, Intrinsic unpred_int, Intrinsic pred_int,
                      bit rounding> {
  def "" : MVE_VQxDMULH_Base<iname, VTI.Suffix, VTI.Size, rounding>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 2082-2083
```tablegen
  let Predicates = [HasMVEInt] in {
    defm : MVE_TwoOpPattern<VTI, Op, pred_int, (? ), Inst>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2085-2089
```tablegen
    // Extra unpredicated multiply intrinsic patterns
    def : Pat<(VTI.Vec (unpred_int (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn)))>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2091-2098
```tablegen
multiclass MVE_VQxDMULH<string iname, MVEVectorVTInfo VTI, bit rounding>
  : MVE_VQxDMULH_m<iname, VTI, !if(rounding, null_frag,
                                             MVEvqdmulh),
                               !if(rounding, int_arm_mve_vqrdmulh,
                                             int_arm_mve_vqdmulh),
                               !if(rounding, int_arm_mve_qrdmulh_predicated,
                                             int_arm_mve_qdmulh_predicated),
                   rounding>;
```
- EN: Declares TableGen `multiclass MVE_VQxDMULH`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VQxDMULH`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2100-2102
```tablegen
defm MVE_VQDMULHi8  : MVE_VQxDMULH<"vqdmulh", MVE_v16s8, 0b0>;
defm MVE_VQDMULHi16 : MVE_VQxDMULH<"vqdmulh", MVE_v8s16, 0b0>;
defm MVE_VQDMULHi32 : MVE_VQxDMULH<"vqdmulh", MVE_v4s32, 0b0>;
```
- EN: Defines TableGen record `MVE_VQDMULHi8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQDMULHi8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2104-2106
```tablegen
defm MVE_VQRDMULHi8  : MVE_VQxDMULH<"vqrdmulh", MVE_v16s8, 0b1>;
defm MVE_VQRDMULHi16 : MVE_VQxDMULH<"vqrdmulh", MVE_v8s16, 0b1>;
defm MVE_VQRDMULHi32 : MVE_VQxDMULH<"vqrdmulh", MVE_v4s32, 0b1>;
```
- EN: Defines TableGen record `MVE_VQRDMULHi8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQRDMULHi8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2108-2110
```tablegen
class MVE_VADDSUB<string iname, string suffix, bits<2> size, bit subtract,
                    list<dag> pattern=[]>
  : MVE_int<iname, suffix, size, pattern> {
```
- EN: Declares reusable TableGen class `MVE_VADDSUB` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VADDSUB`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2112-2119
```tablegen
  let Inst{28} = subtract;
  let Inst{25-23} = 0b110;
  let Inst{16} = 0b0;
  let Inst{12-8} = 0b01000;
  let Inst{4} = 0b0;
  let Inst{0} = 0b0;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2121-2124
```tablegen
multiclass MVE_VADDSUB_m<string iname, MVEVectorVTInfo VTI, bit subtract,
                         SDNode Op, Intrinsic PredInt> {
  def "" : MVE_VADDSUB<iname, VTI.Suffix, VTI.Size, subtract>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 2126-2129
```tablegen
  let Predicates = [HasMVEInt] in {
    defm : MVE_TwoOpPattern<VTI, Op, PredInt, (? ), !cast<Instruction>(NAME), ARMimmAllZerosV>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2131-2134
```tablegen
multiclass MVE_VADD<MVEVectorVTInfo VTI>
  : MVE_VADDSUB_m<"vadd", VTI, 0b0, add, int_arm_mve_add_predicated>;
multiclass MVE_VSUB<MVEVectorVTInfo VTI>
  : MVE_VADDSUB_m<"vsub", VTI, 0b1, sub, int_arm_mve_sub_predicated>;
```
- EN: Declares TableGen `multiclass MVE_VADD`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VADD`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2136-2138
```tablegen
defm MVE_VADDi8  : MVE_VADD<MVE_v16i8>;
defm MVE_VADDi16 : MVE_VADD<MVE_v8i16>;
defm MVE_VADDi32 : MVE_VADD<MVE_v4i32>;
```
- EN: Defines TableGen record `MVE_VADDi8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VADDi8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2140-2142
```tablegen
defm MVE_VSUBi8  : MVE_VSUB<MVE_v16i8>;
defm MVE_VSUBi16 : MVE_VSUB<MVE_v8i16>;
defm MVE_VSUBi32 : MVE_VSUB<MVE_v4i32>;
```
- EN: Defines TableGen record `MVE_VSUBi8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSUBi8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2144-2146
```tablegen
class MVE_VQADDSUB<string iname, string suffix, bit U, bit subtract,
                   bits<2> size>
  : MVE_int<iname, suffix, size, []> {
```
- EN: Declares reusable TableGen class `MVE_VQADDSUB` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VQADDSUB`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2148-2157
```tablegen
  let Inst{28} = U;
  let Inst{25-23} = 0b110;
  let Inst{16} = 0b0;
  let Inst{12-10} = 0b000;
  let Inst{9} = subtract;
  let Inst{8} = 0b0;
  let Inst{4} = 0b1;
  let Inst{0} = 0b0;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2159-2162
```tablegen
class MVE_VQADD_<string suffix, bit U, bits<2> size>
  : MVE_VQADDSUB<"vqadd", suffix, U, 0b0, size>;
class MVE_VQSUB_<string suffix, bit U, bits<2> size>
  : MVE_VQADDSUB<"vqsub", suffix, U, 0b1, size>;
```
- EN: Declares reusable TableGen class `MVE_VQADD_` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VQADD_`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2164-2167
```tablegen
multiclass MVE_VQADD_m<MVEVectorVTInfo VTI,
                      SDNode Op, Intrinsic PredInt> {
  def "" : MVE_VQADD_<VTI.Suffix, VTI.Unsigned, VTI.Size>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 2169-2173
```tablegen
  let Predicates = [HasMVEInt] in {
    defm : MVE_TwoOpPattern<VTI, Op, PredInt, (? (i32 VTI.Unsigned)),
                            !cast<Instruction>(NAME)>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2175-2176
```tablegen
multiclass MVE_VQADD<MVEVectorVTInfo VTI, SDNode unpred_op>
  : MVE_VQADD_m<VTI, unpred_op, int_arm_mve_qadd_predicated>;
```
- EN: Declares TableGen `multiclass MVE_VQADD`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VQADD`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2178-2183
```tablegen
defm MVE_VQADDs8  : MVE_VQADD<MVE_v16s8, saddsat>;
defm MVE_VQADDs16 : MVE_VQADD<MVE_v8s16, saddsat>;
defm MVE_VQADDs32 : MVE_VQADD<MVE_v4s32, saddsat>;
defm MVE_VQADDu8  : MVE_VQADD<MVE_v16u8, uaddsat>;
defm MVE_VQADDu16 : MVE_VQADD<MVE_v8u16, uaddsat>;
defm MVE_VQADDu32 : MVE_VQADD<MVE_v4u32, uaddsat>;
```
- EN: Defines TableGen record `MVE_VQADDs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQADDs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2185-2188
```tablegen
multiclass MVE_VQSUB_m<MVEVectorVTInfo VTI,
                      SDNode Op, Intrinsic PredInt> {
  def "" : MVE_VQSUB_<VTI.Suffix, VTI.Unsigned, VTI.Size>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 2190-2194
```tablegen
  let Predicates = [HasMVEInt] in {
    defm : MVE_TwoOpPattern<VTI, Op, PredInt, (? (i32 VTI.Unsigned)),
                            !cast<Instruction>(NAME)>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2196-2197
```tablegen
multiclass MVE_VQSUB<MVEVectorVTInfo VTI, SDNode unpred_op>
  : MVE_VQSUB_m<VTI, unpred_op, int_arm_mve_qsub_predicated>;
```
- EN: Declares TableGen `multiclass MVE_VQSUB`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VQSUB`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2199-2204
```tablegen
defm MVE_VQSUBs8  : MVE_VQSUB<MVE_v16s8, ssubsat>;
defm MVE_VQSUBs16 : MVE_VQSUB<MVE_v8s16, ssubsat>;
defm MVE_VQSUBs32 : MVE_VQSUB<MVE_v4s32, ssubsat>;
defm MVE_VQSUBu8  : MVE_VQSUB<MVE_v16u8, usubsat>;
defm MVE_VQSUBu16 : MVE_VQSUB<MVE_v8u16, usubsat>;
defm MVE_VQSUBu32 : MVE_VQSUB<MVE_v4u32, usubsat>;
```
- EN: Defines TableGen record `MVE_VQSUBs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQSUBs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2206-2208
```tablegen
class MVE_VABD_int<string suffix, bit U, bits<2> size,
                     list<dag> pattern=[]>
  : MVE_int<"vabd", suffix, size, pattern> {
```
- EN: Declares reusable TableGen class `MVE_VABD_int` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VABD_int`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2210-2217
```tablegen
  let Inst{28} = U;
  let Inst{25-23} = 0b110;
  let Inst{16} = 0b0;
  let Inst{12-8} = 0b00111;
  let Inst{4} = 0b0;
  let Inst{0} = 0b0;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2219-2222
```tablegen
multiclass MVE_VABD_m<MVEVectorVTInfo VTI, SDNode Op,
                      Intrinsic unpred_int, Intrinsic PredInt> {
  def "" : MVE_VABD_int<VTI.Suffix, VTI.Unsigned, VTI.Size>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 2224-2226
```tablegen
  let Predicates = [HasMVEInt] in {
    defm : MVE_TwoOpPattern<VTI, Op, PredInt, (? (i32 VTI.Unsigned)),
                            !cast<Instruction>(NAME)>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2228-2233
```tablegen
    // Unpredicated absolute difference
    def : Pat<(VTI.Vec (unpred_int (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn),
                            (i32 VTI.Unsigned))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn)))>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2235-2236
```tablegen
multiclass MVE_VABD<MVEVectorVTInfo VTI, SDNode Op>
  : MVE_VABD_m<VTI, Op, int_arm_mve_vabd, int_arm_mve_abd_predicated>;
```
- EN: Declares TableGen `multiclass MVE_VABD`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VABD`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2238-2243
```tablegen
defm MVE_VABDs8  : MVE_VABD<MVE_v16s8, abds>;
defm MVE_VABDs16 : MVE_VABD<MVE_v8s16, abds>;
defm MVE_VABDs32 : MVE_VABD<MVE_v4s32, abds>;
defm MVE_VABDu8  : MVE_VABD<MVE_v16u8, abdu>;
defm MVE_VABDu16 : MVE_VABD<MVE_v8u16, abdu>;
defm MVE_VABDu32 : MVE_VABD<MVE_v4u32, abdu>;
```
- EN: Defines TableGen record `MVE_VABDs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VABDs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2245-2246
```tablegen
class MVE_VRHADD_Base<string suffix, bit U, bits<2> size, list<dag> pattern=[]>
  : MVE_int<"vrhadd", suffix, size, pattern> {
```
- EN: Declares reusable TableGen class `MVE_VRHADD_Base` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VRHADD_Base`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2248-2255
```tablegen
  let Inst{28} = U;
  let Inst{25-23} = 0b110;
  let Inst{16} = 0b0;
  let Inst{12-8} = 0b00001;
  let Inst{4} = 0b0;
  let Inst{0} = 0b0;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2257-2260
```tablegen
def addnuw : PatFrag<(ops node:$lhs, node:$rhs),
                     (add node:$lhs, node:$rhs), [{
  return N->getFlags().hasNoUnsignedWrap();
}]>;
```
- EN: Defines TableGen record `addnuw` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `addnuw`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2262-2265
```tablegen
def addnsw : PatFrag<(ops node:$lhs, node:$rhs),
                     (add node:$lhs, node:$rhs), [{
  return N->getFlags().hasNoSignedWrap();
}]>;
```
- EN: Defines TableGen record `addnsw` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `addnsw`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2267-2270
```tablegen
def subnuw : PatFrag<(ops node:$lhs, node:$rhs),
                     (sub node:$lhs, node:$rhs), [{
  return N->getFlags().hasNoUnsignedWrap();
}]>;
```
- EN: Defines TableGen record `subnuw` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `subnuw`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2272-2275
```tablegen
def subnsw : PatFrag<(ops node:$lhs, node:$rhs),
                     (sub node:$lhs, node:$rhs), [{
  return N->getFlags().hasNoSignedWrap();
}]>;
```
- EN: Defines TableGen record `subnsw` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `subnsw`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2277-2281
```tablegen
multiclass MVE_VRHADD_m<MVEVectorVTInfo VTI, SDNode Op,
                      SDPatternOperator unpred_op, Intrinsic PredInt> {
  def "" : MVE_VRHADD_Base<VTI.Suffix, VTI.Unsigned, VTI.Size>;
  defvar Inst = !cast<Instruction>(NAME);
  defm : MVE_TwoOpPattern<VTI, Op, PredInt, (? (i32 VTI.Unsigned)), !cast<Instruction>(NAME)>;
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 2283-2289
```tablegen
  let Predicates = [HasMVEInt] in {
    // Unpredicated rounding add-with-divide-by-two intrinsic
    def : Pat<(VTI.Vec (unpred_op (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn),
                            (i32 VTI.Unsigned))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn)))>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2291-2292
```tablegen
multiclass MVE_VRHADD<MVEVectorVTInfo VTI, SDNode rhadd>
  : MVE_VRHADD_m<VTI, rhadd, int_arm_mve_vrhadd, int_arm_mve_rhadd_predicated>;
```
- EN: Declares TableGen `multiclass MVE_VRHADD`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VRHADD`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2294-2299
```tablegen
defm MVE_VRHADDs8  : MVE_VRHADD<MVE_v16s8, avgceils>;
defm MVE_VRHADDs16 : MVE_VRHADD<MVE_v8s16, avgceils>;
defm MVE_VRHADDs32 : MVE_VRHADD<MVE_v4s32, avgceils>;
defm MVE_VRHADDu8  : MVE_VRHADD<MVE_v16u8, avgceilu>;
defm MVE_VRHADDu16 : MVE_VRHADD<MVE_v8u16, avgceilu>;
defm MVE_VRHADDu32 : MVE_VRHADD<MVE_v4u32, avgceilu>;
```
- EN: Defines TableGen record `MVE_VRHADDs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VRHADDs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2301-2303
```tablegen
class MVE_VHADDSUB<string iname, string suffix, bit U, bit subtract,
                   bits<2> size, list<dag> pattern=[]>
  : MVE_int<iname, suffix, size, pattern> {
```
- EN: Declares reusable TableGen class `MVE_VHADDSUB` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VHADDSUB`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2305-2314
```tablegen
  let Inst{28} = U;
  let Inst{25-23} = 0b110;
  let Inst{16} = 0b0;
  let Inst{12-10} = 0b000;
  let Inst{9} = subtract;
  let Inst{8} = 0b0;
  let Inst{4} = 0b0;
  let Inst{0} = 0b0;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2316-2321
```tablegen
class MVE_VHADD_<string suffix, bit U, bits<2> size,
              list<dag> pattern=[]>
  : MVE_VHADDSUB<"vhadd", suffix, U, 0b0, size, pattern>;
class MVE_VHSUB_<string suffix, bit U, bits<2> size,
              list<dag> pattern=[]>
  : MVE_VHADDSUB<"vhsub", suffix, U, 0b1, size, pattern>;
```
- EN: Declares reusable TableGen class `MVE_VHADD_` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VHADD_`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2323-2327
```tablegen
multiclass MVE_VHADD_m<MVEVectorVTInfo VTI, SDNode Op,
                      SDPatternOperator unpred_op, Intrinsic PredInt> {
  def "" : MVE_VHADD_<VTI.Suffix, VTI.Unsigned, VTI.Size>;
  defvar Inst = !cast<Instruction>(NAME);
  defm : MVE_TwoOpPattern<VTI, Op, PredInt, (? (i32 VTI.Unsigned)), !cast<Instruction>(NAME)>;
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 2329-2334
```tablegen
  let Predicates = [HasMVEInt] in {
    // Unpredicated add-and-divide-by-two
    def : Pat<(VTI.Vec (unpred_op (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn), (i32 VTI.Unsigned))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn)))>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2336-2337
```tablegen
multiclass MVE_VHADD<MVEVectorVTInfo VTI, SDNode Op>
  : MVE_VHADD_m<VTI, Op, int_arm_mve_vhadd, int_arm_mve_hadd_predicated>;
```
- EN: Declares TableGen `multiclass MVE_VHADD`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VHADD`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2339-2344
```tablegen
defm MVE_VHADDs8  : MVE_VHADD<MVE_v16s8, avgfloors>;
defm MVE_VHADDs16 : MVE_VHADD<MVE_v8s16, avgfloors>;
defm MVE_VHADDs32 : MVE_VHADD<MVE_v4s32, avgfloors>;
defm MVE_VHADDu8  : MVE_VHADD<MVE_v16u8, avgflooru>;
defm MVE_VHADDu16 : MVE_VHADD<MVE_v8u16, avgflooru>;
defm MVE_VHADDu32 : MVE_VHADD<MVE_v4u32, avgflooru>;
```
- EN: Defines TableGen record `MVE_VHADDs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VHADDs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2346-2350
```tablegen
multiclass MVE_VHSUB_m<MVEVectorVTInfo VTI,
                      SDPatternOperator unpred_op, Intrinsic pred_int, PatFrag sub_op,
                      SDNode shift_op> {
  def "" : MVE_VHSUB_<VTI.Suffix, VTI.Unsigned, VTI.Size>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 2352-2356
```tablegen
  let Predicates = [HasMVEInt] in {
    // Unpredicated subtract-and-divide-by-two
    def : Pat<(VTI.Vec (unpred_op (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn),
                            (i32 VTI.Unsigned))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn)))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2358-2359
```tablegen
    def : Pat<(VTI.Vec (shift_op (sub_op (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn)), (i32 1))),
              (Inst MQPR:$Qm, MQPR:$Qn)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2362-2370
```tablegen
    // Predicated subtract-and-divide-by-two
    def : Pat<(VTI.Vec (pred_int (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn),
                            (i32 VTI.Unsigned), (VTI.Pred VCCR:$mask),
                            (VTI.Vec MQPR:$inactive))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn),
                             ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg,
                             (VTI.Vec MQPR:$inactive)))>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2372-2374
```tablegen
multiclass MVE_VHSUB<MVEVectorVTInfo VTI, PatFrag sub_op, SDNode shift_op>
  : MVE_VHSUB_m<VTI, int_arm_mve_vhsub, int_arm_mve_hsub_predicated, sub_op,
                shift_op>;
```
- EN: Declares TableGen `multiclass MVE_VHSUB`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VHSUB`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2376-2381
```tablegen
defm MVE_VHSUBs8  : MVE_VHSUB<MVE_v16s8, subnsw, ARMvshrsImm>;
defm MVE_VHSUBs16 : MVE_VHSUB<MVE_v8s16, subnsw, ARMvshrsImm>;
defm MVE_VHSUBs32 : MVE_VHSUB<MVE_v4s32, subnsw, ARMvshrsImm>;
defm MVE_VHSUBu8  : MVE_VHSUB<MVE_v16u8, subnuw, ARMvshruImm>;
defm MVE_VHSUBu16 : MVE_VHSUB<MVE_v8u16, subnuw, ARMvshruImm>;
defm MVE_VHSUBu32 : MVE_VHSUB<MVE_v4u32, subnuw, ARMvshruImm>;
```
- EN: Defines TableGen record `MVE_VHSUBs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VHSUBs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2383-2387
```tablegen
class MVE_VDUP<string suffix, bit B, bit E, bits<2> vecsize, list<dag> pattern=[]>
  : MVE_p<(outs MQPR:$Qd), (ins rGPR:$Rt), NoItinerary,
          "vdup", suffix, "$Qd, $Rt", vpred_r, "", vecsize, pattern> {
  bits<4> Qd;
  bits<4> Rt;
```
- EN: Declares reusable TableGen class `MVE_VDUP` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VDUP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2389-2402
```tablegen
  let Inst{28} = 0b0;
  let Inst{25-23} = 0b101;
  let Inst{22} = B;
  let Inst{21-20} = 0b10;
  let Inst{19-17} = Qd{2-0};
  let Inst{16} = 0b0;
  let Inst{15-12} = Rt;
  let Inst{11-8} = 0b1011;
  let Inst{7} = Qd{3};
  let Inst{6} = 0b0;
  let Inst{5} = E;
  let Inst{4-0} = 0b10000;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2404-2406
```tablegen
def MVE_VDUP32 : MVE_VDUP<"32", 0b0, 0b0, 0b10>;
def MVE_VDUP16 : MVE_VDUP<"16", 0b0, 0b1, 0b01>;
def MVE_VDUP8  : MVE_VDUP<"8",  0b1, 0b0, 0b00>;
```
- EN: Defines TableGen record `MVE_VDUP32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VDUP32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2408-2414
```tablegen
let Predicates = [HasMVEInt] in {
  def : Pat<(v16i8 (ARMvdup (i32 rGPR:$elem))),
            (MVE_VDUP8  rGPR:$elem)>;
  def : Pat<(v8i16 (ARMvdup (i32 rGPR:$elem))),
            (MVE_VDUP16 rGPR:$elem)>;
  def : Pat<(v4i32 (ARMvdup (i32 rGPR:$elem))),
            (MVE_VDUP32 rGPR:$elem)>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2416-2419
```tablegen
  def : Pat<(v8f16 (ARMvdup (i32 rGPR:$elem))),
            (MVE_VDUP16 rGPR:$elem)>;
  def : Pat<(v4f32 (ARMvdup (i32 rGPR:$elem))),
            (MVE_VDUP32 rGPR:$elem)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2421-2438
```tablegen
  // Match a vselect with an ARMvdup as a predicated MVE_VDUP
  def : Pat<(v16i8 (vselect (v16i1 VCCR:$pred),
                            (v16i8 (ARMvdup (i32 rGPR:$elem))),
                            (v16i8 MQPR:$inactive))),
            (MVE_VDUP8  rGPR:$elem, ARMVCCThen, (v16i1 VCCR:$pred), zero_reg,
                        (v16i8 MQPR:$inactive))>;
  def : Pat<(v8i16 (vselect (v8i1 VCCR:$pred),
                            (v8i16 (ARMvdup (i32 rGPR:$elem))),
                            (v8i16 MQPR:$inactive))),
            (MVE_VDUP16 rGPR:$elem, ARMVCCThen, (v8i1 VCCR:$pred), zero_reg,
                            (v8i16 MQPR:$inactive))>;
  def : Pat<(v4i32 (vselect (v4i1 VCCR:$pred),
                            (v4i32 (ARMvdup (i32 rGPR:$elem))),
                            (v4i32 MQPR:$inactive))),
            (MVE_VDUP32 rGPR:$elem, ARMVCCThen, (v4i1 VCCR:$pred), zero_reg,
                            (v4i32 MQPR:$inactive))>;
  def : Pat<(v4f32 (vselect (v4i1 VCCR:$pred),
                            (v4f32 (ARMvdup (i32 rGPR:$elem))),
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2439-2447
```tablegen
                            (v4f32 MQPR:$inactive))),
            (MVE_VDUP32 rGPR:$elem, ARMVCCThen, (v4i1 VCCR:$pred), zero_reg,
                            (v4f32 MQPR:$inactive))>;
  def : Pat<(v8f16 (vselect (v8i1 VCCR:$pred),
                            (v8f16 (ARMvdup (i32 rGPR:$elem))),
                            (v8f16 MQPR:$inactive))),
            (MVE_VDUP16 rGPR:$elem, ARMVCCThen, (v8i1 VCCR:$pred), zero_reg,
                            (v8f16 MQPR:$inactive))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2450-2455
```tablegen
class MVEIntSingleSrc<string iname, string suffix, bits<2> size,
                         list<dag> pattern=[]>
  : MVE_p<(outs MQPR:$Qd), (ins MQPR:$Qm), NoItinerary,
          iname, suffix, "$Qd, $Qm", vpred_r, "", size, pattern> {
  bits<4> Qd;
  bits<4> Qm;
```
- EN: Declares reusable TableGen class `MVEIntSingleSrc` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVEIntSingleSrc`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2457-2462
```tablegen
  let Inst{22} = Qd{3};
  let Inst{19-18} = size{1-0};
  let Inst{15-13} = Qd{2-0};
  let Inst{5} = Qm{3};
  let Inst{3-1} = Qm{2-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2464-2466
```tablegen
class MVE_VCLSCLZ<string iname, string suffix, bits<2> size,
                   bit count_zeroes, list<dag> pattern=[]>
  : MVEIntSingleSrc<iname, suffix, size, pattern> {
```
- EN: Declares reusable TableGen class `MVE_VCLSCLZ` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VCLSCLZ`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2468-2478
```tablegen
  let Inst{28} = 0b1;
  let Inst{25-23} = 0b111;
  let Inst{21-20} = 0b11;
  let Inst{17-16} = 0b00;
  let Inst{12-8} = 0b00100;
  let Inst{7} = count_zeroes;
  let Inst{6} = 0b1;
  let Inst{4} = 0b0;
  let Inst{0} = 0b0;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2480-2482
```tablegen
multiclass MVE_VCLSCLZ_p<string opname, bit opcode, MVEVectorVTInfo VTI,
                         SDPatternOperator unpred_op> {
  def "": MVE_VCLSCLZ<"v"#opname, VTI.Suffix, VTI.Size, opcode>;
```
- EN: Declares TableGen `multiclass MVE_VCLSCLZ_p`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VCLSCLZ_p`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2484-2485
```tablegen
  defvar Inst     = !cast<Instruction>(NAME);
  defvar pred_int = !cast<Intrinsic>("int_arm_mve_"#opname#"_predicated");
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 2487-2495
```tablegen
  let Predicates = [HasMVEInt] in {
    def : Pat<(VTI.Vec (unpred_op (VTI.Vec MQPR:$val))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$val)))>;
    def : Pat<(VTI.Vec (pred_int (VTI.Vec MQPR:$val), (VTI.Pred VCCR:$pred),
                                 (VTI.Vec MQPR:$inactive))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$val), ARMVCCThen,
                             (VTI.Pred VCCR:$pred), zero_reg, (VTI.Vec MQPR:$inactive)))>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2497-2499
```tablegen
defm MVE_VCLSs8  : MVE_VCLSCLZ_p<"cls", 0, MVE_v16s8, ctls>;
defm MVE_VCLSs16 : MVE_VCLSCLZ_p<"cls", 0, MVE_v8s16, ctls>;
defm MVE_VCLSs32 : MVE_VCLSCLZ_p<"cls", 0, MVE_v4s32, ctls>;
```
- EN: Defines TableGen record `MVE_VCLSs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VCLSs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2501-2503
```tablegen
defm MVE_VCLZs8  : MVE_VCLSCLZ_p<"clz", 1, MVE_v16i8, ctlz>;
defm MVE_VCLZs16 : MVE_VCLSCLZ_p<"clz", 1, MVE_v8i16, ctlz>;
defm MVE_VCLZs32 : MVE_VCLSCLZ_p<"clz", 1, MVE_v4i32, ctlz>;
```
- EN: Defines TableGen record `MVE_VCLZs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VCLZs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2505-2507
```tablegen
class MVE_VABSNEG_int<string iname, string suffix, bits<2> size, bit negate,
                      bit saturate, list<dag> pattern=[]>
  : MVEIntSingleSrc<iname, suffix, size, pattern> {
```
- EN: Declares reusable TableGen class `MVE_VABSNEG_int` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VABSNEG_int`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2509-2522
```tablegen
  let Inst{28} = 0b1;
  let Inst{25-23} = 0b111;
  let Inst{21-20} = 0b11;
  let Inst{17} = 0b0;
  let Inst{16} = !eq(saturate, 0);
  let Inst{12-11} = 0b00;
  let Inst{10} = saturate;
  let Inst{9-8} = 0b11;
  let Inst{7} = negate;
  let Inst{6} = 0b1;
  let Inst{4} = 0b0;
  let Inst{0} = 0b0;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2524-2528
```tablegen
multiclass MVE_VABSNEG_int_m<string iname, bit negate, bit saturate,
                             SDPatternOperator unpred_op, Intrinsic pred_int,
                             MVEVectorVTInfo VTI> {
  def "" : MVE_VABSNEG_int<iname, VTI.Suffix, VTI.Size, negate, saturate>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 2530-2535
```tablegen
  let Predicates = [HasMVEInt] in {
    // VQABS and VQNEG have more difficult isel patterns defined elsewhere
    if !not(saturate) then {
      def : Pat<(VTI.Vec (unpred_op (VTI.Vec MQPR:$v))),
                (VTI.Vec (Inst $v))>;
    }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2537-2541
```tablegen
    def : Pat<(VTI.Vec (pred_int  (VTI.Vec MQPR:$v), (VTI.Pred VCCR:$mask),
                                  (VTI.Vec MQPR:$inactive))),
              (VTI.Vec (Inst $v, ARMVCCThen, $mask, zero_reg, $inactive))>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2543-2552
```tablegen
foreach VTI = [ MVE_v16s8, MVE_v8s16, MVE_v4s32 ] in {
  defm "MVE_VABS" # VTI.Suffix : MVE_VABSNEG_int_m<
     "vabs",  0, 0, abs,   int_arm_mve_abs_predicated,  VTI>;
  defm "MVE_VQABS" # VTI.Suffix : MVE_VABSNEG_int_m<
     "vqabs", 0, 1, ?,     int_arm_mve_qabs_predicated, VTI>;
  defm "MVE_VNEG" # VTI.Suffix : MVE_VABSNEG_int_m<
     "vneg",  1, 0, vnegq, int_arm_mve_neg_predicated,  VTI>;
  defm "MVE_VQNEG" # VTI.Suffix : MVE_VABSNEG_int_m<
     "vqneg", 1, 1, ?,     int_arm_mve_qneg_predicated, VTI>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2554-2571
```tablegen
// int_min/int_max: vector containing INT_MIN/INT_MAX VTI.Size times
// zero_vec: v4i32-initialized zero vector, potentially wrapped in a bitconvert
multiclass vqabsneg_pattern<MVEVectorVTInfo VTI, dag int_min, dag int_max,
                         dag zero_vec,  MVE_VABSNEG_int vqabs_instruction,
                         MVE_VABSNEG_int vqneg_instruction> {
  let Predicates = [HasMVEInt] in {
    // The below tree can be replaced by a vqabs instruction, as it represents
    // the following vectorized expression (r being the value in $reg):
    // r > 0 ? r : (r == INT_MIN ? INT_MAX : -r)
    def : Pat<(VTI.Vec (vselect
                      (VTI.Pred (ARMvcmpz (VTI.Vec MQPR:$reg), ARMCCgt)),
                      (VTI.Vec MQPR:$reg),
                      (VTI.Vec (vselect
                                (VTI.Pred (ARMvcmp (VTI.Vec MQPR:$reg), int_min, ARMCCeq)),
                                int_max,
                                (sub (VTI.Vec zero_vec), (VTI.Vec MQPR:$reg)))))),
            (VTI.Vec (vqabs_instruction (VTI.Vec MQPR:$reg)))>;
    // Similarly, this tree represents vqneg, i.e. the following vectorized expression:
```
- EN: Declares TableGen `multiclass vqabsneg_pattern`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass vqabsneg_pattern`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2572-2579
```tablegen
    // r == INT_MIN ? INT_MAX : -r
    def : Pat<(VTI.Vec (vselect
                        (VTI.Pred (ARMvcmp (VTI.Vec MQPR:$reg), int_min, ARMCCeq)),
                        int_max,
                        (sub (VTI.Vec zero_vec), (VTI.Vec MQPR:$reg)))),
               (VTI.Vec (vqneg_instruction (VTI.Vec MQPR:$reg)))>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2581-2595
```tablegen
defm MVE_VQABSNEG_Ps8  : vqabsneg_pattern<MVE_v16i8,
                                    (v16i8 (ARMvmovImm (i32 3712))),
                                    (v16i8 (ARMvmovImm (i32 3711))),
                                    (bitconvert (v4i32 (ARMvmovImm (i32 0)))),
                                    MVE_VQABSs8, MVE_VQNEGs8>;
defm MVE_VQABSNEG_Ps16 : vqabsneg_pattern<MVE_v8i16,
                                    (v8i16 (ARMvmovImm (i32 2688))),
                                    (v8i16 (ARMvmvnImm (i32 2688))),
                                    (bitconvert (v4i32 (ARMvmovImm (i32 0)))),
                                    MVE_VQABSs16, MVE_VQNEGs16>;
defm MVE_VQABSNEG_Ps32 : vqabsneg_pattern<MVE_v4i32,
                                    (v4i32 (ARMvmovImm (i32 1664))),
                                    (v4i32 (ARMvmvnImm (i32 1664))),
                                    (ARMvmovImm (i32 0)),
                                    MVE_VQABSs32, MVE_VQNEGs32>;
```
- EN: Defines TableGen record `MVE_VQABSNEG_Ps8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQABSNEG_Ps8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2597-2602
```tablegen
class MVE_mod_imm<string iname, string suffix, bits<4> cmode, bit op,
                  dag iops, bits<2> vecsize, list<dag> pattern=[]>
  : MVE_p<(outs MQPR:$Qd), iops, NoItinerary, iname, suffix, "$Qd, $imm",
          vpred_r, "", vecsize, pattern> {
  bits<13> imm;
  bits<4> Qd;
```
- EN: Declares reusable TableGen class `MVE_mod_imm` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_mod_imm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2604-2615
```tablegen
  let Inst{28} = imm{7};
  let Inst{25-23} = 0b111;
  let Inst{22} = Qd{3};
  let Inst{21-19} = 0b000;
  let Inst{18-16} = imm{6-4};
  let Inst{15-13} = Qd{2-0};
  let Inst{12} = 0b0;
  let Inst{11-8} = cmode{3-0};
  let Inst{7-6} = 0b01;
  let Inst{5} = op;
  let Inst{4} = 0b1;
  let Inst{3-0} = imm{3-0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2617-2619
```tablegen
  let DecoderMethod = "DecodeMVEModImmInstruction";
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2621-2632
```tablegen
let isReMaterializable = 1 in {
let isAsCheapAsAMove = 1 in {
def MVE_VMOVimmi8  : MVE_mod_imm<"vmov", "i8",  {1,1,1,0}, 0b0, (ins nImmSplatI8:$imm), 0b00>;
def MVE_VMOVimmi16 : MVE_mod_imm<"vmov", "i16", {1,0,?,0}, 0b0, (ins nImmSplatI16:$imm), 0b01> {
  let Inst{9} = imm{9};
}
def MVE_VMOVimmi32 : MVE_mod_imm<"vmov", "i32", {?,?,?,?}, 0b0, (ins nImmVMOVI32:$imm), 0b10> {
  let Inst{11-8} = imm{11-8};
}
def MVE_VMOVimmi64 : MVE_mod_imm<"vmov", "i64", {1,1,1,0}, 0b1, (ins nImmSplatI64:$imm), 0b11>;
def MVE_VMOVimmf32 : MVE_mod_imm<"vmov", "f32", {1,1,1,1}, 0b0, (ins nImmVMOVF32:$imm), 0b10>;
} // let isAsCheapAsAMove = 1
```
- EN: Defines TableGen record `MVE_VMOVimmi8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMOVimmi8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2634-2640
```tablegen
def MVE_VMVNimmi16 : MVE_mod_imm<"vmvn", "i16", {1,0,?,0}, 0b1, (ins nImmSplatI16:$imm), 0b01> {
  let Inst{9} = imm{9};
}
def MVE_VMVNimmi32 : MVE_mod_imm<"vmvn", "i32", {?,?,?,?}, 0b1, (ins nImmVMOVI32:$imm), 0b10> {
  let Inst{11-8} = imm{11-8};
}
} // let isReMaterializable = 1
```
- EN: Defines TableGen record `MVE_VMVNimmi16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMVNimmi16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2642-2650
```tablegen
let Predicates = [HasMVEInt] in {
  def : Pat<(v16i8 (ARMvmovImm timm:$simm)),
            (v16i8 (MVE_VMOVimmi8  nImmSplatI8:$simm))>;
  def : Pat<(v8i16 (ARMvmovImm timm:$simm)),
            (v8i16 (MVE_VMOVimmi16 nImmSplatI16:$simm))>;
  def : Pat<(v4i32 (ARMvmovImm timm:$simm)),
            (v4i32 (MVE_VMOVimmi32 nImmVMOVI32:$simm))>;
  def : Pat<(v2i64 (ARMvmovImm timm:$simm)),
            (v2i64 (MVE_VMOVimmi64 nImmSplatI64:$simm))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2652-2655
```tablegen
  def : Pat<(v8i16 (ARMvmvnImm timm:$simm)),
            (v8i16 (MVE_VMVNimmi16 nImmSplatI16:$simm))>;
  def : Pat<(v4i32 (ARMvmvnImm timm:$simm)),
            (v4i32 (MVE_VMVNimmi32 nImmVMOVI32:$simm))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2657-2658
```tablegen
  def : Pat<(v4f32 (ARMvmovFPImm timm:$simm)),
            (v4f32 (MVE_VMOVimmf32 nImmVMOVF32:$simm))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2660-2668
```tablegen
  def : Pat<(v8i16 (vselect (v8i1 VCCR:$pred), (ARMvmvnImm timm:$simm),
                            MQPR:$inactive)),
            (v8i16 (MVE_VMVNimmi16 nImmSplatI16:$simm,
                            ARMVCCThen, VCCR:$pred, zero_reg, MQPR:$inactive))>;
  def : Pat<(v4i32 (vselect (v4i1 VCCR:$pred), (ARMvmvnImm timm:$simm),
                            MQPR:$inactive)),
            (v4i32 (MVE_VMVNimmi32 nImmSplatI32:$simm,
                            ARMVCCThen, VCCR:$pred, zero_reg, MQPR:$inactive))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2670-2676
```tablegen
class MVE_VMINMAXA<string iname, string suffix, bits<2> size,
                   bit bit_12, list<dag> pattern=[]>
  : MVE_p<(outs MQPR:$Qd), (ins MQPR:$Qd_src, MQPR:$Qm),
          NoItinerary, iname, suffix, "$Qd, $Qm", vpred_n, "$Qd = $Qd_src",
          size, pattern> {
  bits<4> Qd;
  bits<4> Qm;
```
- EN: Declares reusable TableGen class `MVE_VMINMAXA` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VMINMAXA`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2678-2692
```tablegen
  let Inst{28} = 0b0;
  let Inst{25-23} = 0b100;
  let Inst{22} = Qd{3};
  let Inst{21-20} = 0b11;
  let Inst{19-18} = size;
  let Inst{17-16} = 0b11;
  let Inst{15-13} = Qd{2-0};
  let Inst{12} = bit_12;
  let Inst{11-6} = 0b111010;
  let Inst{5} = Qm{3};
  let Inst{4} = 0b0;
  let Inst{3-1} = Qm{2-0};
  let Inst{0} = 0b1;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2694-2697
```tablegen
multiclass MVE_VMINMAXA_m<string iname, MVEVectorVTInfo VTI,
                      SDNode unpred_op, Intrinsic pred_int, bit bit_12> {
  def "" : MVE_VMINMAXA<iname, VTI.Suffix, VTI.Size, bit_12>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 2699-2702
```tablegen
  let Predicates = [HasMVEInt] in {
    // Unpredicated v(min|max)a
    def : Pat<(VTI.Vec (unpred_op (VTI.Vec MQPR:$Qd), (abs (VTI.Vec MQPR:$Qm)))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qd), (VTI.Vec MQPR:$Qm)))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2704-2710
```tablegen
    // Predicated v(min|max)a
    def : Pat<(VTI.Vec (pred_int (VTI.Vec MQPR:$Qd), (VTI.Vec MQPR:$Qm),
                            (VTI.Pred VCCR:$mask))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qd), (VTI.Vec MQPR:$Qm),
                            ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg))>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2712-2713
```tablegen
multiclass MVE_VMINA<MVEVectorVTInfo VTI>
  : MVE_VMINMAXA_m<"vmina", VTI, umin, int_arm_mve_vmina_predicated, 0b1>;
```
- EN: Declares TableGen `multiclass MVE_VMINA`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMINA`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2715-2717
```tablegen
defm MVE_VMINAs8  : MVE_VMINA<MVE_v16s8>;
defm MVE_VMINAs16 : MVE_VMINA<MVE_v8s16>;
defm MVE_VMINAs32 : MVE_VMINA<MVE_v4s32>;
```
- EN: Defines TableGen record `MVE_VMINAs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMINAs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2719-2720
```tablegen
multiclass MVE_VMAXA<MVEVectorVTInfo VTI>
  : MVE_VMINMAXA_m<"vmaxa", VTI, umax, int_arm_mve_vmaxa_predicated, 0b0>;
```
- EN: Declares TableGen `multiclass MVE_VMAXA`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMAXA`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2722-2724
```tablegen
defm MVE_VMAXAs8  : MVE_VMAXA<MVE_v16s8>;
defm MVE_VMAXAs16 : MVE_VMAXA<MVE_v8s16>;
defm MVE_VMAXAs32 : MVE_VMAXA<MVE_v4s32>;
```
- EN: Defines TableGen record `MVE_VMAXAs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMAXAs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2726-2726
```tablegen
// end of MVE Integer instructions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 2728-2728
```tablegen
// start of mve_imm_shift instructions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 2730-2736
```tablegen
def MVE_VSHLC : MVE_p<(outs rGPR:$RdmDest, MQPR:$Qd),
                      (ins MQPR:$QdSrc, rGPR:$RdmSrc, long_shift:$imm),
                      NoItinerary, "vshlc", "", "$QdSrc, $RdmSrc, $imm",
                      vpred_n, "$RdmDest = $RdmSrc,$Qd = $QdSrc", 0b10> {
  bits<5> imm;
  bits<4> Qd;
  bits<4> RdmDest;
```
- EN: Defines TableGen record `MVE_VSHLC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSHLC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2738-2746
```tablegen
  let Inst{28} = 0b0;
  let Inst{25-23} = 0b101;
  let Inst{22} = Qd{3};
  let Inst{21} = 0b1;
  let Inst{20-16} = imm{4-0};
  let Inst{15-13} = Qd{2-0};
  let Inst{12-4} = 0b011111100;
  let Inst{3-0} = RdmDest{3-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2748-2753
```tablegen
class MVE_shift_imm<dag oops, dag iops, string iname, string suffix,
                    string ops, vpred_ops vpred, string cstr,
                    bits<2> vecsize, list<dag> pattern=[]>
  : MVE_p<oops, iops, NoItinerary, iname, suffix, ops, vpred, cstr, vecsize, pattern> {
  bits<4> Qd;
  bits<4> Qm;
```
- EN: Declares reusable TableGen class `MVE_shift_imm` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_shift_imm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2755-2759
```tablegen
  let Inst{22} = Qd{3};
  let Inst{15-13} = Qd{2-0};
  let Inst{5} = Qm{3};
  let Inst{3-1} = Qm{2-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2761-2776
```tablegen
class MVE_VMOVL<string iname, string suffix, bits<2> sz, bit U, bit top,
              list<dag> pattern=[]>
  : MVE_shift_imm<(outs MQPR:$Qd), (ins MQPR:$Qm),
                  iname, suffix, "$Qd, $Qm", vpred_r, "",
                  sz, pattern> {
  let Inst{28} = U;
  let Inst{25-23} = 0b101;
  let Inst{21} = 0b1;
  let Inst{20-19} = sz{1-0};
  let Inst{18-16} = 0b000;
  let Inst{12} = top;
  let Inst{11-6} = 0b111101;
  let Inst{4} = 0b0;
  let Inst{0} = 0b0;
  let doubleWidthResult = 1;
}
```
- EN: Declares reusable TableGen class `MVE_VMOVL` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VMOVL`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2778-2782
```tablegen
multiclass MVE_VMOVL_m<bit top, string chr, MVEVectorVTInfo OutVTI,
                       MVEVectorVTInfo InVTI> {
  def "": MVE_VMOVL<"vmovl" # chr, InVTI.Suffix, OutVTI.Size,
                    InVTI.Unsigned, top>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 2784-2791
```tablegen
  def : Pat<(OutVTI.Vec (int_arm_mve_vmovl_predicated (InVTI.Vec MQPR:$src),
                            (i32 InVTI.Unsigned), (i32 top),
                            (OutVTI.Pred VCCR:$pred),
                            (OutVTI.Vec MQPR:$inactive))),
            (OutVTI.Vec (Inst (InVTI.Vec MQPR:$src), ARMVCCThen,
                            (OutVTI.Pred VCCR:$pred), zero_reg,
                            (OutVTI.Vec MQPR:$inactive)))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2793-2800
```tablegen
defm MVE_VMOVLs8bh  : MVE_VMOVL_m<0, "b", MVE_v8s16, MVE_v16s8>;
defm MVE_VMOVLs8th  : MVE_VMOVL_m<1, "t", MVE_v8s16, MVE_v16s8>;
defm MVE_VMOVLu8bh  : MVE_VMOVL_m<0, "b", MVE_v8u16, MVE_v16u8>;
defm MVE_VMOVLu8th  : MVE_VMOVL_m<1, "t", MVE_v8u16, MVE_v16u8>;
defm MVE_VMOVLs16bh : MVE_VMOVL_m<0, "b", MVE_v4s32, MVE_v8s16>;
defm MVE_VMOVLs16th : MVE_VMOVL_m<1, "t", MVE_v4s32, MVE_v8s16>;
defm MVE_VMOVLu16bh : MVE_VMOVL_m<0, "b", MVE_v4s32, MVE_v8u16>;
defm MVE_VMOVLu16th : MVE_VMOVL_m<1, "t", MVE_v4s32, MVE_v8u16>;
```
- EN: Defines TableGen record `MVE_VMOVLs8bh` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMOVLs8bh`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2802-2808
```tablegen
let Predicates = [HasMVEInt] in {
  def : Pat<(sext_inreg (v4i32 MQPR:$src), v4i16),
            (MVE_VMOVLs16bh MQPR:$src)>;
  def : Pat<(sext_inreg (v8i16 MQPR:$src), v8i8),
            (MVE_VMOVLs8bh MQPR:$src)>;
  def : Pat<(sext_inreg (v4i32 MQPR:$src), v4i8),
            (MVE_VMOVLs16bh (MVE_VMOVLs8bh MQPR:$src))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2810-2813
```tablegen
  def : Pat<(sext_inreg (v8i16 (ARMVectorRegCast (ARMvrev16 (v16i8 MQPR:$src)))), v8i8),
            (MVE_VMOVLs8th MQPR:$src)>;
  def : Pat<(sext_inreg (v4i32 (ARMVectorRegCast (ARMvrev32 (v8i16 MQPR:$src)))), v4i16),
            (MVE_VMOVLs16th MQPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2815-2827
```tablegen
  // zext_inreg 8 -> 16
  def : Pat<(ARMvbicImm (v8i16 MQPR:$src), (i32 0xAFF)),
            (MVE_VMOVLu8bh MQPR:$src)>;
  // zext_inreg 16 -> 32
  def : Pat<(and (v4i32 MQPR:$src), (v4i32 (ARMvmovImm (i32 0xCFF)))),
            (MVE_VMOVLu16bh MQPR:$src)>;
  // Same zext_inreg with vrevs, picking the top half
  def : Pat<(ARMvbicImm (v8i16 (ARMVectorRegCast (ARMvrev16 (v16i8 MQPR:$src)))), (i32 0xAFF)),
            (MVE_VMOVLu8th MQPR:$src)>;
  def : Pat<(and (v4i32 (ARMVectorRegCast (ARMvrev32 (v8i16 MQPR:$src)))),
                 (v4i32 (ARMvmovImm (i32 0xCFF)))),
            (MVE_VMOVLu16th MQPR:$src)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2830-2840
```tablegen
class MVE_VSHLL_imm<string iname, string suffix, bit U, bit th,
                    Operand immtype, bits<2> vecsize, list<dag> pattern=[]>
  : MVE_shift_imm<(outs MQPR:$Qd), (ins MQPR:$Qm, immtype:$imm),
                  iname, suffix, "$Qd, $Qm, $imm", vpred_r, "", vecsize, pattern> {
  let Inst{28} = U;
  let Inst{25-23} = 0b101;
  let Inst{21} = 0b1;
  let Inst{12} = th;
  let Inst{11-6} = 0b111101;
  let Inst{4} = 0b0;
  let Inst{0} = 0b0;
```
- EN: Declares reusable TableGen class `MVE_VSHLL_imm` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VSHLL_imm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2842-2843
```tablegen
  // For the MVE_VSHLL_patterns multiclass to refer to
  Operand immediateType = immtype;
```
- EN: Declares TableGen `multiclass to`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass to`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2845-2846
```tablegen
  let doubleWidthResult = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2848-2850
```tablegen
// The immediate VSHLL instructions accept shift counts from 1 up to
// the lane width (8 or 16), but the full-width shifts have an
// entirely separate encoding, given below with 'lw' in the name.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 2852-2858
```tablegen
class MVE_VSHLL_imm8<string iname, string suffix,
                     bit U, bit th, list<dag> pattern=[]>
  : MVE_VSHLL_imm<iname, suffix, U, th, mve_shift_imm1_7, 0b01, pattern> {
  bits<3> imm;
  let Inst{20-19} = 0b01;
  let Inst{18-16} = imm;
}
```
- EN: Declares reusable TableGen class `MVE_VSHLL_imm8` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VSHLL_imm8`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2860-2866
```tablegen
class MVE_VSHLL_imm16<string iname, string suffix,
                      bit U, bit th, list<dag> pattern=[]>
  : MVE_VSHLL_imm<iname, suffix, U, th, mve_shift_imm1_15, 0b10, pattern> {
  bits<4> imm;
  let Inst{20} = 0b1;
  let Inst{19-16} = imm;
}
```
- EN: Declares reusable TableGen class `MVE_VSHLL_imm16` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VSHLL_imm16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2868-2875
```tablegen
def MVE_VSHLL_imms8bh  : MVE_VSHLL_imm8 <"vshllb", "s8", 0b0, 0b0>;
def MVE_VSHLL_imms8th  : MVE_VSHLL_imm8 <"vshllt", "s8", 0b0, 0b1>;
def MVE_VSHLL_immu8bh  : MVE_VSHLL_imm8 <"vshllb", "u8", 0b1, 0b0>;
def MVE_VSHLL_immu8th  : MVE_VSHLL_imm8 <"vshllt", "u8", 0b1, 0b1>;
def MVE_VSHLL_imms16bh : MVE_VSHLL_imm16<"vshllb", "s16", 0b0, 0b0>;
def MVE_VSHLL_imms16th : MVE_VSHLL_imm16<"vshllt", "s16", 0b0, 0b1>;
def MVE_VSHLL_immu16bh : MVE_VSHLL_imm16<"vshllb", "u16", 0b1, 0b0>;
def MVE_VSHLL_immu16th : MVE_VSHLL_imm16<"vshllt", "u16", 0b1, 0b1>;
```
- EN: Defines TableGen record `MVE_VSHLL_imms8bh` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSHLL_imms8bh`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2877-2890
```tablegen
class MVE_VSHLL_by_lane_width<string iname, string suffix, bits<2> size,
                              bit U, string ops, list<dag> pattern=[]>
  : MVE_shift_imm<(outs MQPR:$Qd), (ins MQPR:$Qm),
                  iname, suffix, ops, vpred_r, "", !if(size, 0b10, 0b01), pattern> {
  let Inst{28} = U;
  let Inst{25-23} = 0b100;
  let Inst{21-20} = 0b11;
  let Inst{19-18} = size{1-0};
  let Inst{17-16} = 0b01;
  let Inst{11-6} = 0b111000;
  let Inst{4} = 0b0;
  let Inst{0} = 0b1;
  let doubleWidthResult = 1;
}
```
- EN: Declares reusable TableGen class `MVE_VSHLL_by_lane_width` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VSHLL_by_lane_width`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2892-2900
```tablegen
multiclass MVE_VSHLL_lw<string iname, string suffix, bits<2> sz, bit U,
                              string ops, list<dag> pattern=[]> {
  def bh : MVE_VSHLL_by_lane_width<iname#"b", suffix, sz, U, ops, pattern> {
    let Inst{12} = 0b0;
  }
  def th : MVE_VSHLL_by_lane_width<iname#"t", suffix, sz, U, ops, pattern> {
    let Inst{12} = 0b1;
  }
}
```
- EN: Declares TableGen `multiclass MVE_VSHLL_lw`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VSHLL_lw`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2902-2905
```tablegen
defm MVE_VSHLL_lws8  : MVE_VSHLL_lw<"vshll", "s8",  0b00, 0b0, "$Qd, $Qm, #8">;
defm MVE_VSHLL_lws16 : MVE_VSHLL_lw<"vshll", "s16", 0b01, 0b0, "$Qd, $Qm, #16">;
defm MVE_VSHLL_lwu8  : MVE_VSHLL_lw<"vshll", "u8",  0b00, 0b1, "$Qd, $Qm, #8">;
defm MVE_VSHLL_lwu16 : MVE_VSHLL_lw<"vshll", "u16", 0b01, 0b1, "$Qd, $Qm, #16">;
```
- EN: Defines TableGen record `MVE_VSHLL_lws8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSHLL_lws8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2907-2913
```tablegen
multiclass MVE_VSHLL_patterns<MVEVectorVTInfo VTI, int top> {
  defvar suffix     = !strconcat(VTI.Suffix, !if(top, "th", "bh"));
  defvar inst_imm   = !cast<MVE_VSHLL_imm>("MVE_VSHLL_imm" # suffix);
  defvar inst_lw    = !cast<MVE_VSHLL_by_lane_width>("MVE_VSHLL_lw" # suffix);
  defvar unpred_int = int_arm_mve_vshll_imm;
  defvar pred_int   = int_arm_mve_vshll_imm_predicated;
  defvar imm        = inst_imm.immediateType;
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 2915-2920
```tablegen
  def : Pat<(VTI.DblVec (unpred_int (VTI.Vec MQPR:$src), imm:$imm,
                                    (i32 VTI.Unsigned), (i32 top))),
            (VTI.DblVec (inst_imm   (VTI.Vec MQPR:$src), imm:$imm))>;
  def : Pat<(VTI.DblVec (unpred_int (VTI.Vec MQPR:$src), (i32 VTI.LaneBits),
                                    (i32 VTI.Unsigned), (i32 top))),
            (VTI.DblVec (inst_lw    (VTI.Vec MQPR:$src)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2922-2936
```tablegen
  def : Pat<(VTI.DblVec (pred_int   (VTI.Vec MQPR:$src), imm:$imm,
                                    (i32 VTI.Unsigned), (i32 top),
                                    (VTI.DblPred VCCR:$mask),
                                    (VTI.DblVec MQPR:$inactive))),
            (VTI.DblVec (inst_imm   (VTI.Vec MQPR:$src), imm:$imm,
                                    ARMVCCThen, (VTI.DblPred VCCR:$mask), zero_reg,
                                    (VTI.DblVec MQPR:$inactive)))>;
  def : Pat<(VTI.DblVec (pred_int   (VTI.Vec MQPR:$src), (i32 VTI.LaneBits),
                                    (i32 VTI.Unsigned), (i32 top),
                                    (VTI.DblPred VCCR:$mask),
                                    (VTI.DblVec MQPR:$inactive))),
            (VTI.DblVec (inst_lw    (VTI.Vec MQPR:$src), ARMVCCThen,
                                    (VTI.DblPred VCCR:$mask), zero_reg,
                                    (VTI.DblVec MQPR:$inactive)))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2938-2940
```tablegen
foreach VTI = [MVE_v16s8, MVE_v8s16, MVE_v16u8, MVE_v8u16] in
  foreach top = [0, 1] in
    defm : MVE_VSHLL_patterns<VTI, top>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2942-2946
```tablegen
class MVE_shift_imm_partial<Operand imm, string iname, string suffix, bits<2> vecsize>
  : MVE_shift_imm<(outs MQPR:$Qd), (ins MQPR:$QdSrc, MQPR:$Qm, imm:$imm),
                  iname, suffix, "$Qd, $Qm, $imm", vpred_n, "$Qd = $QdSrc", vecsize> {
  Operand immediateType = imm;
}
```
- EN: Declares reusable TableGen class `MVE_shift_imm_partial` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_shift_imm_partial`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2948-2951
```tablegen
class MVE_VxSHRN<string iname, string suffix, bit bit_12, bit bit_28,
                 Operand imm, bits<2> vecsize>
  : MVE_shift_imm_partial<imm, iname, suffix, vecsize> {
  bits<5> imm;
```
- EN: Declares reusable TableGen class `MVE_VxSHRN` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VxSHRN`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2953-2963
```tablegen
  let Inst{28} = bit_28;
  let Inst{25-23} = 0b101;
  let Inst{21} = 0b0;
  let Inst{20-16} = imm{4-0};
  let Inst{12} = bit_12;
  let Inst{11-6} = 0b111111;
  let Inst{4} = 0b0;
  let Inst{0} = 0b1;
  let validForTailPredication = 1;
  let retainsPreviousHalfElement = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2965-2976
```tablegen
def MVE_VRSHRNi16bh : MVE_VxSHRN<"vrshrnb", "i16", 0b0, 0b1, shr_imm8, 0b01> {
  let Inst{20-19} = 0b01;
}
def MVE_VRSHRNi16th : MVE_VxSHRN<"vrshrnt", "i16", 0b1, 0b1, shr_imm8, 0b01> {
  let Inst{20-19} = 0b01;
}
def MVE_VRSHRNi32bh : MVE_VxSHRN<"vrshrnb", "i32", 0b0, 0b1, shr_imm16, 0b10> {
  let Inst{20} = 0b1;
}
def MVE_VRSHRNi32th : MVE_VxSHRN<"vrshrnt", "i32", 0b1, 0b1, shr_imm16, 0b10> {
  let Inst{20} = 0b1;
}
```
- EN: Defines TableGen record `MVE_VRSHRNi16bh` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VRSHRNi16bh`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2978-2989
```tablegen
def MVE_VSHRNi16bh : MVE_VxSHRN<"vshrnb", "i16", 0b0, 0b0, shr_imm8, 0b01> {
  let Inst{20-19} = 0b01;
}
def MVE_VSHRNi16th : MVE_VxSHRN<"vshrnt", "i16", 0b1, 0b0, shr_imm8, 0b01> {
  let Inst{20-19} = 0b01;
}
def MVE_VSHRNi32bh : MVE_VxSHRN<"vshrnb", "i32", 0b0, 0b0, shr_imm16, 0b10> {
  let Inst{20} = 0b1;
}
def MVE_VSHRNi32th : MVE_VxSHRN<"vshrnt", "i32", 0b1, 0b0, shr_imm16, 0b10> {
  let Inst{20} = 0b1;
}
```
- EN: Defines TableGen record `MVE_VSHRNi16bh` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSHRNi16bh`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2991-2994
```tablegen
class MVE_VxQRSHRUN<string iname, string suffix, bit bit_28, bit bit_12,
                    Operand imm, bits<2> vecsize>
  : MVE_shift_imm_partial<imm, iname, suffix, vecsize> {
  bits<5> imm;
```
- EN: Declares reusable TableGen class `MVE_VxQRSHRUN` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VxQRSHRUN`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2996-3006
```tablegen
  let Inst{28} = bit_28;
  let Inst{25-23} = 0b101;
  let Inst{21} = 0b0;
  let Inst{20-16} = imm{4-0};
  let Inst{12} = bit_12;
  let Inst{11-6} = 0b111111;
  let Inst{4} = 0b0;
  let Inst{0} = 0b0;
  let validForTailPredication = 1;
  let retainsPreviousHalfElement = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3008-3023
```tablegen
def MVE_VQRSHRUNs16bh : MVE_VxQRSHRUN<
    "vqrshrunb", "s16", 0b1, 0b0, shr_imm8, 0b01> {
  let Inst{20-19} = 0b01;
}
def MVE_VQRSHRUNs16th : MVE_VxQRSHRUN<
    "vqrshrunt", "s16", 0b1, 0b1, shr_imm8, 0b01> {
  let Inst{20-19} = 0b01;
}
def MVE_VQRSHRUNs32bh : MVE_VxQRSHRUN<
    "vqrshrunb", "s32", 0b1, 0b0, shr_imm16, 0b10> {
  let Inst{20} = 0b1;
}
def MVE_VQRSHRUNs32th : MVE_VxQRSHRUN<
    "vqrshrunt", "s32", 0b1, 0b1, shr_imm16, 0b10> {
  let Inst{20} = 0b1;
}
```
- EN: Defines TableGen record `MVE_VQRSHRUNs16bh` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQRSHRUNs16bh`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3025-3040
```tablegen
def MVE_VQSHRUNs16bh : MVE_VxQRSHRUN<
    "vqshrunb", "s16", 0b0, 0b0, shr_imm8, 0b01> {
  let Inst{20-19} = 0b01;
}
def MVE_VQSHRUNs16th : MVE_VxQRSHRUN<
    "vqshrunt", "s16", 0b0, 0b1, shr_imm8, 0b01> {
  let Inst{20-19} = 0b01;
}
def MVE_VQSHRUNs32bh : MVE_VxQRSHRUN<
    "vqshrunb", "s32", 0b0, 0b0, shr_imm16, 0b10> {
  let Inst{20} = 0b1;
}
def MVE_VQSHRUNs32th : MVE_VxQRSHRUN<
    "vqshrunt", "s32", 0b0, 0b1, shr_imm16, 0b10> {
  let Inst{20} = 0b1;
}
```
- EN: Defines TableGen record `MVE_VQSHRUNs16bh` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQSHRUNs16bh`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3042-3045
```tablegen
class MVE_VxQRSHRN<string iname, string suffix, bit bit_0, bit bit_12,
                   Operand imm, bits<2> vecsize>
  : MVE_shift_imm_partial<imm, iname, suffix, vecsize> {
  bits<5> imm;
```
- EN: Declares reusable TableGen class `MVE_VxQRSHRN` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VxQRSHRN`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3047-3056
```tablegen
  let Inst{25-23} = 0b101;
  let Inst{21} = 0b0;
  let Inst{20-16} = imm{4-0};
  let Inst{12} = bit_12;
  let Inst{11-6} = 0b111101;
  let Inst{4} = 0b0;
  let Inst{0} = bit_0;
  let validForTailPredication = 1;
  let retainsPreviousHalfElement = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3058-3075
```tablegen
multiclass MVE_VxQRSHRN_types<string iname, bit bit_0, bit bit_12> {
  def s16 : MVE_VxQRSHRN<iname, "s16", bit_0, bit_12, shr_imm8, 0b01> {
    let Inst{28} = 0b0;
    let Inst{20-19} = 0b01;
  }
  def u16 : MVE_VxQRSHRN<iname, "u16", bit_0, bit_12, shr_imm8, 0b01> {
    let Inst{28} = 0b1;
    let Inst{20-19} = 0b01;
  }
  def s32 : MVE_VxQRSHRN<iname, "s32", bit_0, bit_12, shr_imm16, 0b10> {
    let Inst{28} = 0b0;
    let Inst{20} = 0b1;
  }
  def u32 : MVE_VxQRSHRN<iname, "u32", bit_0, bit_12, shr_imm16, 0b10> {
    let Inst{28} = 0b1;
    let Inst{20} = 0b1;
  }
}
```
- EN: Declares TableGen `multiclass MVE_VxQRSHRN_types`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VxQRSHRN_types`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3077-3080
```tablegen
defm MVE_VQRSHRNbh : MVE_VxQRSHRN_types<"vqrshrnb", 0b1, 0b0>;
defm MVE_VQRSHRNth : MVE_VxQRSHRN_types<"vqrshrnt", 0b1, 0b1>;
defm MVE_VQSHRNbh  : MVE_VxQRSHRN_types<"vqshrnb", 0b0, 0b0>;
defm MVE_VQSHRNth  : MVE_VxQRSHRN_types<"vqshrnt", 0b0, 0b1>;
```
- EN: Defines TableGen record `MVE_VQRSHRNbh` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQRSHRNbh`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3082-3089
```tablegen
multiclass MVE_VSHRN_patterns<MVE_shift_imm_partial inst,
                              MVEVectorVTInfo OutVTI, MVEVectorVTInfo InVTI,
                              bit q, bit r, bit top> {
  defvar inparams = (? (OutVTI.Vec MQPR:$QdSrc), (InVTI.Vec MQPR:$Qm),
                       (inst.immediateType:$imm), (i32 q), (i32 r),
                       (i32 OutVTI.Unsigned), (i32 InVTI.Unsigned), (i32 top));
  defvar outparams = (inst (OutVTI.Vec MQPR:$QdSrc), (InVTI.Vec MQPR:$Qm),
                           (imm:$imm));
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 3091-3096
```tablegen
  def : Pat<(OutVTI.Vec !setdagop(inparams, int_arm_mve_vshrn)),
            (OutVTI.Vec outparams)>;
  def : Pat<(OutVTI.Vec !con(inparams, (int_arm_mve_vshrn_predicated
                                           (InVTI.Pred VCCR:$pred)))),
            (OutVTI.Vec !con(outparams, (? ARMVCCThen, VCCR:$pred, zero_reg)))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3098-3115
```tablegen
defm : MVE_VSHRN_patterns<MVE_VSHRNi16bh,    MVE_v16s8, MVE_v8s16, 0,0,0>;
defm : MVE_VSHRN_patterns<MVE_VSHRNi16th,    MVE_v16s8, MVE_v8s16, 0,0,1>;
defm : MVE_VSHRN_patterns<MVE_VSHRNi32bh,    MVE_v8s16, MVE_v4s32, 0,0,0>;
defm : MVE_VSHRN_patterns<MVE_VSHRNi32th,    MVE_v8s16, MVE_v4s32, 0,0,1>;
defm : MVE_VSHRN_patterns<MVE_VSHRNi16bh,    MVE_v16u8, MVE_v8u16, 0,0,0>;
defm : MVE_VSHRN_patterns<MVE_VSHRNi16th,    MVE_v16u8, MVE_v8u16, 0,0,1>;
defm : MVE_VSHRN_patterns<MVE_VSHRNi32bh,    MVE_v8u16, MVE_v4u32, 0,0,0>;
defm : MVE_VSHRN_patterns<MVE_VSHRNi32th,    MVE_v8u16, MVE_v4u32, 0,0,1>;
defm : MVE_VSHRN_patterns<MVE_VRSHRNi16bh,   MVE_v16s8, MVE_v8s16, 0,1,0>;
defm : MVE_VSHRN_patterns<MVE_VRSHRNi16th,   MVE_v16s8, MVE_v8s16, 0,1,1>;
defm : MVE_VSHRN_patterns<MVE_VRSHRNi32bh,   MVE_v8s16, MVE_v4s32, 0,1,0>;
defm : MVE_VSHRN_patterns<MVE_VRSHRNi32th,   MVE_v8s16, MVE_v4s32, 0,1,1>;
defm : MVE_VSHRN_patterns<MVE_VRSHRNi16bh,   MVE_v16u8, MVE_v8u16, 0,1,0>;
defm : MVE_VSHRN_patterns<MVE_VRSHRNi16th,   MVE_v16u8, MVE_v8u16, 0,1,1>;
defm : MVE_VSHRN_patterns<MVE_VRSHRNi32bh,   MVE_v8u16, MVE_v4u32, 0,1,0>;
defm : MVE_VSHRN_patterns<MVE_VRSHRNi32th,   MVE_v8u16, MVE_v4u32, 0,1,1>;
defm : MVE_VSHRN_patterns<MVE_VQSHRNbhs16,   MVE_v16s8, MVE_v8s16, 1,0,0>;
defm : MVE_VSHRN_patterns<MVE_VQSHRNths16,   MVE_v16s8, MVE_v8s16, 1,0,1>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3116-3133
```tablegen
defm : MVE_VSHRN_patterns<MVE_VQSHRNbhs32,   MVE_v8s16, MVE_v4s32, 1,0,0>;
defm : MVE_VSHRN_patterns<MVE_VQSHRNths32,   MVE_v8s16, MVE_v4s32, 1,0,1>;
defm : MVE_VSHRN_patterns<MVE_VQSHRNbhu16,   MVE_v16u8, MVE_v8u16, 1,0,0>;
defm : MVE_VSHRN_patterns<MVE_VQSHRNthu16,   MVE_v16u8, MVE_v8u16, 1,0,1>;
defm : MVE_VSHRN_patterns<MVE_VQSHRNbhu32,   MVE_v8u16, MVE_v4u32, 1,0,0>;
defm : MVE_VSHRN_patterns<MVE_VQSHRNthu32,   MVE_v8u16, MVE_v4u32, 1,0,1>;
defm : MVE_VSHRN_patterns<MVE_VQRSHRNbhs16,  MVE_v16s8, MVE_v8s16, 1,1,0>;
defm : MVE_VSHRN_patterns<MVE_VQRSHRNths16,  MVE_v16s8, MVE_v8s16, 1,1,1>;
defm : MVE_VSHRN_patterns<MVE_VQRSHRNbhs32,  MVE_v8s16, MVE_v4s32, 1,1,0>;
defm : MVE_VSHRN_patterns<MVE_VQRSHRNths32,  MVE_v8s16, MVE_v4s32, 1,1,1>;
defm : MVE_VSHRN_patterns<MVE_VQRSHRNbhu16,  MVE_v16u8, MVE_v8u16, 1,1,0>;
defm : MVE_VSHRN_patterns<MVE_VQRSHRNthu16,  MVE_v16u8, MVE_v8u16, 1,1,1>;
defm : MVE_VSHRN_patterns<MVE_VQRSHRNbhu32,  MVE_v8u16, MVE_v4u32, 1,1,0>;
defm : MVE_VSHRN_patterns<MVE_VQRSHRNthu32,  MVE_v8u16, MVE_v4u32, 1,1,1>;
defm : MVE_VSHRN_patterns<MVE_VQSHRUNs16bh,  MVE_v16u8, MVE_v8s16, 1,0,0>;
defm : MVE_VSHRN_patterns<MVE_VQSHRUNs16th,  MVE_v16u8, MVE_v8s16, 1,0,1>;
defm : MVE_VSHRN_patterns<MVE_VQSHRUNs32bh,  MVE_v8u16, MVE_v4s32, 1,0,0>;
defm : MVE_VSHRN_patterns<MVE_VQSHRUNs32th,  MVE_v8u16, MVE_v4s32, 1,0,1>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3134-3137
```tablegen
defm : MVE_VSHRN_patterns<MVE_VQRSHRUNs16bh, MVE_v16u8, MVE_v8s16, 1,1,0>;
defm : MVE_VSHRN_patterns<MVE_VQRSHRUNs16th, MVE_v16u8, MVE_v8s16, 1,1,1>;
defm : MVE_VSHRN_patterns<MVE_VQRSHRUNs32bh, MVE_v8u16, MVE_v4s32, 1,1,0>;
defm : MVE_VSHRN_patterns<MVE_VQRSHRUNs32th, MVE_v8u16, MVE_v4s32, 1,1,1>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3139-3139
```tablegen
// end of mve_imm_shift instructions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 3141-3141
```tablegen
// start of mve_shift instructions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 3143-3150
```tablegen
class MVE_shift_by_vec<string iname, string suffix, bit U,
                       bits<2> size, bit bit_4, bit bit_8>
  : MVE_p<(outs MQPR:$Qd), (ins MQPR:$Qm, MQPR:$Qn), NoItinerary,
           iname, suffix, "$Qd, $Qm, $Qn", vpred_r, "", size, []> {
  // Shift instructions which take a vector of shift counts
  bits<4> Qd;
  bits<4> Qm;
  bits<4> Qn;
```
- EN: Declares reusable TableGen class `MVE_shift_by_vec` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_shift_by_vec`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3152-3169
```tablegen
  let Inst{28} = U;
  let Inst{25-24} = 0b11;
  let Inst{23} = 0b0;
  let Inst{22} = Qd{3};
  let Inst{21-20} = size;
  let Inst{19-17} = Qn{2-0};
  let Inst{16} = 0b0;
  let Inst{15-13} = Qd{2-0};
  let Inst{12-9} = 0b0010;
  let Inst{8} = bit_8;
  let Inst{7} = Qn{3};
  let Inst{6} = 0b1;
  let Inst{5} = Qm{3};
  let Inst{4} = bit_4;
  let Inst{3-1} = Qm{2-0};
  let Inst{0} = 0b0;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3171-3173
```tablegen
multiclass MVE_shift_by_vec_p<string iname, MVEVectorVTInfo VTI, bit q, bit r> {
  def "" : MVE_shift_by_vec<iname, VTI.Suffix, VTI.Unsigned, VTI.Size, q, r>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 3175-3178
```tablegen
  def : Pat<(VTI.Vec (int_arm_mve_vshl_vector
                         (VTI.Vec MQPR:$in), (VTI.Vec MQPR:$sh),
                         (i32 q), (i32 r), (i32 VTI.Unsigned))),
            (VTI.Vec (Inst (VTI.Vec MQPR:$in), (VTI.Vec MQPR:$sh)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3180-3187
```tablegen
  def : Pat<(VTI.Vec (int_arm_mve_vshl_vector_predicated
                         (VTI.Vec MQPR:$in), (VTI.Vec MQPR:$sh),
                         (i32 q), (i32 r), (i32 VTI.Unsigned),
                         (VTI.Pred VCCR:$mask), (VTI.Vec MQPR:$inactive))),
            (VTI.Vec (Inst (VTI.Vec MQPR:$in), (VTI.Vec MQPR:$sh),
                           ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg,
                           (VTI.Vec MQPR:$inactive)))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3189-3196
```tablegen
multiclass mve_shift_by_vec_multi<string iname, bit bit_4, bit bit_8> {
  defm s8  : MVE_shift_by_vec_p<iname, MVE_v16s8, bit_4, bit_8>;
  defm s16 : MVE_shift_by_vec_p<iname, MVE_v8s16, bit_4, bit_8>;
  defm s32 : MVE_shift_by_vec_p<iname, MVE_v4s32, bit_4, bit_8>;
  defm u8  : MVE_shift_by_vec_p<iname, MVE_v16u8, bit_4, bit_8>;
  defm u16 : MVE_shift_by_vec_p<iname, MVE_v8u16, bit_4, bit_8>;
  defm u32 : MVE_shift_by_vec_p<iname, MVE_v4u32, bit_4, bit_8>;
}
```
- EN: Declares TableGen `multiclass mve_shift_by_vec_multi`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass mve_shift_by_vec_multi`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3198-3201
```tablegen
defm MVE_VSHL_by_vec   : mve_shift_by_vec_multi<"vshl",   0b0, 0b0>;
defm MVE_VQSHL_by_vec  : mve_shift_by_vec_multi<"vqshl",  0b1, 0b0>;
defm MVE_VQRSHL_by_vec : mve_shift_by_vec_multi<"vqrshl", 0b1, 0b1>;
defm MVE_VRSHL_by_vec  : mve_shift_by_vec_multi<"vrshl",  0b0, 0b1>;
```
- EN: Defines TableGen record `MVE_VSHL_by_vec` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSHL_by_vec`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3203-3216
```tablegen
let Predicates = [HasMVEInt] in {
  defm : MVE_TwoOpPattern<MVE_v16i8, ARMvshlu, int_arm_mve_vshl_vector_predicated,
                          (? (i32 0), (i32 0), (i32 1)), MVE_VSHL_by_vecu8, null_frag>;
  defm : MVE_TwoOpPattern<MVE_v8i16, ARMvshlu, int_arm_mve_vshl_vector_predicated,
                          (? (i32 0), (i32 0), (i32 1)), MVE_VSHL_by_vecu16, null_frag>;
  defm : MVE_TwoOpPattern<MVE_v4i32, ARMvshlu, int_arm_mve_vshl_vector_predicated,
                          (? (i32 0), (i32 0), (i32 1)), MVE_VSHL_by_vecu32, null_frag>;
  defm : MVE_TwoOpPattern<MVE_v16i8, ARMvshls, int_arm_mve_vshl_vector_predicated,
                          (? (i32 0), (i32 0), (i32 0)), MVE_VSHL_by_vecs8, null_frag>;
  defm : MVE_TwoOpPattern<MVE_v8i16, ARMvshls, int_arm_mve_vshl_vector_predicated,
                          (? (i32 0), (i32 0), (i32 0)), MVE_VSHL_by_vecs16, null_frag>;
  defm : MVE_TwoOpPattern<MVE_v4i32, ARMvshls, int_arm_mve_vshl_vector_predicated,
                          (? (i32 0), (i32 0), (i32 0)), MVE_VSHL_by_vecs32, null_frag>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3218-3223
```tablegen
class MVE_shift_with_imm<string iname, string suffix, dag oops, dag iops,
                         string ops, vpred_ops vpred, string cstr,
                         bits<2> vecsize, list<dag> pattern=[]>
  : MVE_p<oops, iops, NoItinerary, iname, suffix, ops, vpred, cstr, vecsize, pattern> {
  bits<4> Qd;
  bits<4> Qm;
```
- EN: Declares reusable TableGen class `MVE_shift_with_imm` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_shift_with_imm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3225-3234
```tablegen
  let Inst{23} = 0b1;
  let Inst{22} = Qd{3};
  let Inst{15-13} = Qd{2-0};
  let Inst{12-11} = 0b00;
  let Inst{7-6} = 0b01;
  let Inst{5} = Qm{3};
  let Inst{4} = 0b1;
  let Inst{3-1} = Qm{2-0};
  let Inst{0} = 0b0;
  let validForTailPredication = 1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3236-3242
```tablegen
  // For the MVE_shift_imm_patterns multiclass to refer to
  MVEVectorVTInfo VTI;
  Operand immediateType;
  Intrinsic unpred_int;
  Intrinsic pred_int;
  dag unsignedFlag = (?);
}
```
- EN: Declares TableGen `multiclass to`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass to`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3244-3254
```tablegen
class MVE_VSxI_imm<string iname, string suffix, bit bit_8, Operand immType, bits<2> vecsize>
  : MVE_shift_with_imm<iname, suffix, (outs MQPR:$Qd),
                       (ins MQPR:$Qd_src, MQPR:$Qm, immType:$imm),
                       "$Qd, $Qm, $imm", vpred_n, "$Qd = $Qd_src", vecsize> {
  bits<6> imm;
  let Inst{28} = 0b1;
  let Inst{25-24} = 0b11;
  let Inst{21-16} = imm;
  let Inst{10-9} = 0b10;
  let Inst{8} = bit_8;
  let validForTailPredication = 1;
```
- EN: Declares reusable TableGen class `MVE_VSxI_imm` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VSxI_imm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3256-3257
```tablegen
  Operand immediateType = immType;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3259-3261
```tablegen
def MVE_VSRIimm8 : MVE_VSxI_imm<"vsri", "8", 0b0, shr_imm8, 0b00> {
  let Inst{21-19} = 0b001;
}
```
- EN: Defines TableGen record `MVE_VSRIimm8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSRIimm8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3263-3265
```tablegen
def MVE_VSRIimm16 : MVE_VSxI_imm<"vsri", "16", 0b0, shr_imm16, 0b01> {
  let Inst{21-20} = 0b01;
}
```
- EN: Defines TableGen record `MVE_VSRIimm16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSRIimm16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3267-3269
```tablegen
def MVE_VSRIimm32 : MVE_VSxI_imm<"vsri", "32", 0b0, shr_imm32, 0b10> {
  let Inst{21} = 0b1;
}
```
- EN: Defines TableGen record `MVE_VSRIimm32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSRIimm32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3271-3273
```tablegen
def MVE_VSLIimm8 : MVE_VSxI_imm<"vsli", "8", 0b1, imm0_7, 0b00> {
  let Inst{21-19} = 0b001;
}
```
- EN: Defines TableGen record `MVE_VSLIimm8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSLIimm8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3275-3277
```tablegen
def MVE_VSLIimm16 : MVE_VSxI_imm<"vsli", "16", 0b1, imm0_15, 0b01> {
  let Inst{21-20} = 0b01;
}
```
- EN: Defines TableGen record `MVE_VSLIimm16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSLIimm16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3279-3281
```tablegen
def MVE_VSLIimm32 : MVE_VSxI_imm<"vsli", "32", 0b1,imm0_31, 0b10> {
  let Inst{21} = 0b1;
}
```
- EN: Defines TableGen record `MVE_VSLIimm32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSLIimm32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3283-3288
```tablegen
multiclass MVE_VSxI_patterns<MVE_VSxI_imm inst, SDPatternOperator Node,
                             SDPatternOperator PredIntr, MVEVectorVTInfo VTI> {
  defvar inparams = (? (VTI.Vec MQPR:$QdSrc), (VTI.Vec MQPR:$Qm),
                       (inst.immediateType:$imm));
  defvar outparams = (inst (VTI.Vec MQPR:$QdSrc), (VTI.Vec MQPR:$Qm),
                           (inst.immediateType:$imm));
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 3290-3294
```tablegen
  def : Pat<(VTI.Vec !setdagop(inparams, Node)),
            (VTI.Vec outparams)>;
  def : Pat<(VTI.Vec !con(inparams, (PredIntr (VTI.Pred VCCR:$pred)))),
            (VTI.Vec !con(outparams, (? ARMVCCThen, VCCR:$pred, zero_reg)))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3296-3303
```tablegen
let Predicates = [HasMVEInt] in {
defm : MVE_VSxI_patterns<MVE_VSLIimm8,  NEONvsliImm, int_arm_mve_vsli_predicated, MVE_v16i8>;
defm : MVE_VSxI_patterns<MVE_VSLIimm16, NEONvsliImm, int_arm_mve_vsli_predicated, MVE_v8i16>;
defm : MVE_VSxI_patterns<MVE_VSLIimm32, NEONvsliImm, int_arm_mve_vsli_predicated, MVE_v4i32>;
defm : MVE_VSxI_patterns<MVE_VSRIimm8,  NEONvsriImm, int_arm_mve_vsri_predicated, MVE_v16i8>;
defm : MVE_VSxI_patterns<MVE_VSRIimm16, NEONvsriImm, int_arm_mve_vsri_predicated, MVE_v8i16>;
defm : MVE_VSxI_patterns<MVE_VSRIimm32, NEONvsriImm, int_arm_mve_vsri_predicated, MVE_v4i32>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3305-3309
```tablegen
class MVE_VQSHL_imm<MVEVectorVTInfo VTI_, Operand immType>
  : MVE_shift_with_imm<"vqshl", VTI_.Suffix, (outs MQPR:$Qd),
                       (ins MQPR:$Qm, immType:$imm), "$Qd, $Qm, $imm",
                       vpred_r, "", VTI_.Size> {
  bits<6> imm;
```
- EN: Declares reusable TableGen class `MVE_VQSHL_imm` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VQSHL_imm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3311-3314
```tablegen
  let Inst{28} = VTI_.Unsigned;
  let Inst{25-24} = 0b11;
  let Inst{21-16} = imm;
  let Inst{10-8} = 0b111;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3316-3319
```tablegen
  let VTI = VTI_;
  let immediateType = immType;
  let unsignedFlag = (? (i32 VTI.Unsigned));
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3321-3328
```tablegen
let unpred_int = int_arm_mve_vqshl_imm,
    pred_int = int_arm_mve_vqshl_imm_predicated in {
  def MVE_VQSHLimms8 : MVE_VQSHL_imm<MVE_v16s8, imm0_7> {
    let Inst{21-19} = 0b001;
  }
  def MVE_VQSHLimmu8 : MVE_VQSHL_imm<MVE_v16u8, imm0_7> {
    let Inst{21-19} = 0b001;
  }
```
- EN: Defines TableGen record `MVE_VQSHLimms8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQSHLimms8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3330-3335
```tablegen
  def MVE_VQSHLimms16 : MVE_VQSHL_imm<MVE_v8s16, imm0_15> {
    let Inst{21-20} = 0b01;
  }
  def MVE_VQSHLimmu16 : MVE_VQSHL_imm<MVE_v8u16, imm0_15> {
    let Inst{21-20} = 0b01;
  }
```
- EN: Defines TableGen record `MVE_VQSHLimms16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQSHLimms16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3337-3343
```tablegen
  def MVE_VQSHLimms32 : MVE_VQSHL_imm<MVE_v4s32, imm0_31> {
    let Inst{21} = 0b1;
  }
  def MVE_VQSHLimmu32 : MVE_VQSHL_imm<MVE_v4u32, imm0_31> {
    let Inst{21} = 0b1;
  }
}
```
- EN: Defines TableGen record `MVE_VQSHLimms32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQSHLimms32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3345-3349
```tablegen
class MVE_VQSHLU_imm<MVEVectorVTInfo VTI_, Operand immType>
  : MVE_shift_with_imm<"vqshlu", VTI_.Suffix, (outs MQPR:$Qd),
                       (ins MQPR:$Qm, immType:$imm), "$Qd, $Qm, $imm",
                       vpred_r, "", VTI_.Size> {
  bits<6> imm;
```
- EN: Declares reusable TableGen class `MVE_VQSHLU_imm` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VQSHLU_imm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3351-3354
```tablegen
  let Inst{28} = 0b1;
  let Inst{25-24} = 0b11;
  let Inst{21-16} = imm;
  let Inst{10-8} = 0b110;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3356-3358
```tablegen
  let VTI = VTI_;
  let immediateType = immType;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3360-3364
```tablegen
let unpred_int = int_arm_mve_vqshlu_imm,
    pred_int = int_arm_mve_vqshlu_imm_predicated in {
  def MVE_VQSHLU_imms8 : MVE_VQSHLU_imm<MVE_v16s8, imm0_7> {
    let Inst{21-19} = 0b001;
  }
```
- EN: Defines TableGen record `MVE_VQSHLU_imms8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQSHLU_imms8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3366-3368
```tablegen
  def MVE_VQSHLU_imms16 : MVE_VQSHLU_imm<MVE_v8s16, imm0_15> {
    let Inst{21-20} = 0b01;
  }
```
- EN: Defines TableGen record `MVE_VQSHLU_imms16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQSHLU_imms16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3370-3373
```tablegen
  def MVE_VQSHLU_imms32 : MVE_VQSHLU_imm<MVE_v4s32, imm0_31> {
    let Inst{21} = 0b1;
  }
}
```
- EN: Defines TableGen record `MVE_VQSHLU_imms32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQSHLU_imms32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3375-3379
```tablegen
class MVE_VRSHR_imm<MVEVectorVTInfo VTI_, Operand immType>
  : MVE_shift_with_imm<"vrshr", VTI_.Suffix, (outs MQPR:$Qd),
                       (ins MQPR:$Qm, immType:$imm), "$Qd, $Qm, $imm",
                       vpred_r, "", VTI_.Size> {
  bits<6> imm;
```
- EN: Declares reusable TableGen class `MVE_VRSHR_imm` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VRSHR_imm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3381-3384
```tablegen
  let Inst{28} = VTI_.Unsigned;
  let Inst{25-24} = 0b11;
  let Inst{21-16} = imm;
  let Inst{10-8} = 0b010;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3386-3389
```tablegen
  let VTI = VTI_;
  let immediateType = immType;
  let unsignedFlag = (? (i32 VTI.Unsigned));
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3391-3395
```tablegen
let unpred_int = int_arm_mve_vrshr_imm,
    pred_int = int_arm_mve_vrshr_imm_predicated in {
  def MVE_VRSHR_imms8 : MVE_VRSHR_imm<MVE_v16s8, shr_imm8> {
    let Inst{21-19} = 0b001;
  }
```
- EN: Defines TableGen record `MVE_VRSHR_imms8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VRSHR_imms8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3397-3399
```tablegen
  def MVE_VRSHR_immu8 : MVE_VRSHR_imm<MVE_v16u8, shr_imm8> {
    let Inst{21-19} = 0b001;
  }
```
- EN: Defines TableGen record `MVE_VRSHR_immu8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VRSHR_immu8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3401-3403
```tablegen
  def MVE_VRSHR_imms16 : MVE_VRSHR_imm<MVE_v8s16, shr_imm16> {
    let Inst{21-20} = 0b01;
  }
```
- EN: Defines TableGen record `MVE_VRSHR_imms16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VRSHR_imms16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3405-3407
```tablegen
  def MVE_VRSHR_immu16 : MVE_VRSHR_imm<MVE_v8u16, shr_imm16> {
    let Inst{21-20} = 0b01;
  }
```
- EN: Defines TableGen record `MVE_VRSHR_immu16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VRSHR_immu16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3409-3411
```tablegen
  def MVE_VRSHR_imms32 : MVE_VRSHR_imm<MVE_v4s32, shr_imm32> {
    let Inst{21} = 0b1;
  }
```
- EN: Defines TableGen record `MVE_VRSHR_imms32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VRSHR_imms32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3413-3416
```tablegen
  def MVE_VRSHR_immu32 : MVE_VRSHR_imm<MVE_v4u32, shr_imm32> {
    let Inst{21} = 0b1;
  }
}
```
- EN: Defines TableGen record `MVE_VRSHR_immu32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VRSHR_immu32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3418-3423
```tablegen
multiclass MVE_shift_imm_patterns<MVE_shift_with_imm inst> {
  def : Pat<(inst.VTI.Vec !con((inst.unpred_int (inst.VTI.Vec MQPR:$src),
                                                inst.immediateType:$imm),
                               inst.unsignedFlag)),
            (inst.VTI.Vec (inst (inst.VTI.Vec MQPR:$src),
                                inst.immediateType:$imm))>;
```
- EN: Declares TableGen `multiclass MVE_shift_imm_patterns`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_shift_imm_patterns`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3425-3434
```tablegen
  def : Pat<(inst.VTI.Vec !con((inst.pred_int (inst.VTI.Vec MQPR:$src),
                                              inst.immediateType:$imm),
                               inst.unsignedFlag,
                               (? (inst.VTI.Pred VCCR:$mask),
                                  (inst.VTI.Vec MQPR:$inactive)))),
            (inst.VTI.Vec (inst (inst.VTI.Vec MQPR:$src),
                                inst.immediateType:$imm,
                                ARMVCCThen, (inst.VTI.Pred VCCR:$mask), zero_reg,
                                (inst.VTI.Vec MQPR:$inactive)))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3436-3450
```tablegen
defm : MVE_shift_imm_patterns<MVE_VQSHLimms8>;
defm : MVE_shift_imm_patterns<MVE_VQSHLimmu8>;
defm : MVE_shift_imm_patterns<MVE_VQSHLimms16>;
defm : MVE_shift_imm_patterns<MVE_VQSHLimmu16>;
defm : MVE_shift_imm_patterns<MVE_VQSHLimms32>;
defm : MVE_shift_imm_patterns<MVE_VQSHLimmu32>;
defm : MVE_shift_imm_patterns<MVE_VQSHLU_imms8>;
defm : MVE_shift_imm_patterns<MVE_VQSHLU_imms16>;
defm : MVE_shift_imm_patterns<MVE_VQSHLU_imms32>;
defm : MVE_shift_imm_patterns<MVE_VRSHR_imms8>;
defm : MVE_shift_imm_patterns<MVE_VRSHR_immu8>;
defm : MVE_shift_imm_patterns<MVE_VRSHR_imms16>;
defm : MVE_shift_imm_patterns<MVE_VRSHR_immu16>;
defm : MVE_shift_imm_patterns<MVE_VRSHR_imms32>;
defm : MVE_shift_imm_patterns<MVE_VRSHR_immu32>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3452-3456
```tablegen
class MVE_VSHR_imm<string suffix, dag imm, bits<2> vecsize>
  : MVE_shift_with_imm<"vshr", suffix, (outs MQPR:$Qd),
                       !con((ins MQPR:$Qm), imm), "$Qd, $Qm, $imm",
                       vpred_r, "", vecsize> {
  bits<6> imm;
```
- EN: Declares reusable TableGen class `MVE_VSHR_imm` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VSHR_imm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3458-3461
```tablegen
  let Inst{25-24} = 0b11;
  let Inst{21-16} = imm;
  let Inst{10-8} = 0b000;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3463-3466
```tablegen
def MVE_VSHR_imms8 : MVE_VSHR_imm<"s8", (ins shr_imm8:$imm), 0b00> {
  let Inst{28} = 0b0;
  let Inst{21-19} = 0b001;
}
```
- EN: Defines TableGen record `MVE_VSHR_imms8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSHR_imms8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3468-3471
```tablegen
def MVE_VSHR_immu8 : MVE_VSHR_imm<"u8", (ins shr_imm8:$imm), 0b00> {
  let Inst{28} = 0b1;
  let Inst{21-19} = 0b001;
}
```
- EN: Defines TableGen record `MVE_VSHR_immu8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSHR_immu8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3473-3476
```tablegen
def MVE_VSHR_imms16 : MVE_VSHR_imm<"s16", (ins shr_imm16:$imm), 0b01> {
  let Inst{28} = 0b0;
  let Inst{21-20} = 0b01;
}
```
- EN: Defines TableGen record `MVE_VSHR_imms16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSHR_imms16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3478-3481
```tablegen
def MVE_VSHR_immu16 : MVE_VSHR_imm<"u16", (ins shr_imm16:$imm), 0b01> {
  let Inst{28} = 0b1;
  let Inst{21-20} = 0b01;
}
```
- EN: Defines TableGen record `MVE_VSHR_immu16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSHR_immu16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3483-3486
```tablegen
def MVE_VSHR_imms32 : MVE_VSHR_imm<"s32", (ins shr_imm32:$imm), 0b10> {
  let Inst{28} = 0b0;
  let Inst{21} = 0b1;
}
```
- EN: Defines TableGen record `MVE_VSHR_imms32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSHR_imms32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3488-3491
```tablegen
def MVE_VSHR_immu32 : MVE_VSHR_imm<"u32", (ins shr_imm32:$imm), 0b10> {
  let Inst{28} = 0b1;
  let Inst{21} = 0b1;
}
```
- EN: Defines TableGen record `MVE_VSHR_immu32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSHR_immu32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3493-3497
```tablegen
class MVE_VSHL_imm<string suffix, dag imm, bits<2> vecsize>
  : MVE_shift_with_imm<"vshl", suffix, (outs MQPR:$Qd),
                       !con((ins MQPR:$Qm), imm), "$Qd, $Qm, $imm",
                       vpred_r, "", vecsize> {
  bits<6> imm;
```
- EN: Declares reusable TableGen class `MVE_VSHL_imm` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VSHL_imm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3499-3503
```tablegen
  let Inst{28} = 0b0;
  let Inst{25-24} = 0b11;
  let Inst{21-16} = imm;
  let Inst{10-8} = 0b101;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3505-3507
```tablegen
def MVE_VSHL_immi8 : MVE_VSHL_imm<"i8", (ins imm0_7:$imm), 0b00> {
  let Inst{21-19} = 0b001;
}
```
- EN: Defines TableGen record `MVE_VSHL_immi8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSHL_immi8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3509-3511
```tablegen
def MVE_VSHL_immi16 : MVE_VSHL_imm<"i16", (ins imm0_15:$imm), 0b01> {
  let Inst{21-20} = 0b01;
}
```
- EN: Defines TableGen record `MVE_VSHL_immi16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSHL_immi16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3513-3515
```tablegen
def MVE_VSHL_immi32 : MVE_VSHL_imm<"i32", (ins imm0_31:$imm), 0b10> {
  let Inst{21} = 0b1;
}
```
- EN: Defines TableGen record `MVE_VSHL_immi32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSHL_immi32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3517-3519
```tablegen
multiclass MVE_immediate_shift_patterns_inner<
    MVEVectorVTInfo VTI, Operand imm_operand_type, SDNode unpred_op,
    Intrinsic pred_int, Instruction inst, list<int> unsignedFlag = []> {
```
- EN: Declares TableGen `multiclass MVE_immediate_shift_patterns_inner`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_immediate_shift_patterns_inner`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3521-3522
```tablegen
  def : Pat<(VTI.Vec (unpred_op (VTI.Vec MQPR:$src), imm_operand_type:$imm)),
            (VTI.Vec (inst (VTI.Vec MQPR:$src), imm_operand_type:$imm))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3524-3531
```tablegen
  def : Pat<(VTI.Vec !con((pred_int (VTI.Vec MQPR:$src), imm_operand_type:$imm),
                          !dag(pred_int, unsignedFlag, ?),
                          (pred_int (VTI.Pred VCCR:$mask),
                                   (VTI.Vec MQPR:$inactive)))),
            (VTI.Vec (inst (VTI.Vec MQPR:$src), imm_operand_type:$imm,
                           ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg,
                           (VTI.Vec MQPR:$inactive)))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3533-3544
```tablegen
multiclass MVE_immediate_shift_patterns<MVEVectorVTInfo VTI,
                                        Operand imm_operand_type> {
  defm : MVE_immediate_shift_patterns_inner<VTI, imm_operand_type,
      ARMvshlImm, int_arm_mve_shl_imm_predicated,
      !cast<Instruction>("MVE_VSHL_immi" # VTI.BitsSuffix)>;
  defm : MVE_immediate_shift_patterns_inner<VTI, imm_operand_type,
      ARMvshruImm, int_arm_mve_shr_imm_predicated,
      !cast<Instruction>("MVE_VSHR_immu" # VTI.BitsSuffix), [1]>;
  defm : MVE_immediate_shift_patterns_inner<VTI, imm_operand_type,
      ARMvshrsImm, int_arm_mve_shr_imm_predicated,
      !cast<Instruction>("MVE_VSHR_imms" # VTI.BitsSuffix), [0]>;
}
```
- EN: Declares TableGen `multiclass MVE_immediate_shift_patterns`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_immediate_shift_patterns`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3546-3550
```tablegen
let Predicates = [HasMVEInt] in {
  defm : MVE_immediate_shift_patterns<MVE_v16i8, imm0_7>;
  defm : MVE_immediate_shift_patterns<MVE_v8i16, imm0_15>;
  defm : MVE_immediate_shift_patterns<MVE_v4i32, imm0_31>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3552-3552
```tablegen
// end of mve_shift instructions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 3554-3554
```tablegen
// start of MVE Floating Point instructions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 3556-3559
```tablegen
class MVE_float<string iname, string suffix, dag oops, dag iops, string ops,
                vpred_ops vpred, string cstr, bits<2> vecsize, list<dag> pattern=[]>
  : MVE_f<oops, iops, NoItinerary, iname, suffix, ops, vpred, cstr, vecsize, pattern> {
  bits<4> Qm;
```
- EN: Declares reusable TableGen class `MVE_float` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_float`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3561-3566
```tablegen
  let Inst{12} = 0b0;
  let Inst{6} = 0b1;
  let Inst{5} = Qm{3};
  let Inst{3-1} = Qm{2-0};
  let Inst{0} = 0b0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3568-3572
```tablegen
class MVE_VRINT<string rmode, bits<3> op, string suffix, bits<2> size,
                list<dag> pattern=[]>
  : MVE_float<!strconcat("vrint", rmode), suffix, (outs MQPR:$Qd),
              (ins MQPR:$Qm), "$Qd, $Qm", vpred_r, "", size, pattern> {
  bits<4> Qd;
```
- EN: Declares reusable TableGen class `MVE_VRINT` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VRINT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3574-3584
```tablegen
  let Inst{28} = 0b1;
  let Inst{25-23} = 0b111;
  let Inst{22} = Qd{3};
  let Inst{21-20} = 0b11;
  let Inst{19-18} = size;
  let Inst{17-16} = 0b10;
  let Inst{15-13} = Qd{2-0};
  let Inst{11-10} = 0b01;
  let Inst{9-7} = op{2-0};
  let Inst{4} = 0b0;
  let validForTailPredication = 1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3586-3586
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3588-3593
```tablegen
multiclass MVE_VRINT_m<MVEVectorVTInfo VTI, string suffix, bits<3> opcode,
                       SDPatternOperator unpred_op> {
  def "": MVE_VRINT<suffix, opcode, VTI.Suffix, VTI.Size>;
  defvar Inst = !cast<Instruction>(NAME);
  defvar unpred_int = !cast<Intrinsic>("int_arm_mve_vrint"#suffix);
  defvar pred_int = !cast<Intrinsic>("int_arm_mve_vrint"#suffix#"_predicated");
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 3595-3605
```tablegen
  let Predicates = [HasMVEFloat] in {
    def : Pat<(VTI.Vec (unpred_op (VTI.Vec MQPR:$val))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$val)))>;
    def : Pat<(VTI.Vec (unpred_int (VTI.Vec MQPR:$val))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$val)))>;
    def : Pat<(VTI.Vec (pred_int (VTI.Vec MQPR:$val), (VTI.Pred VCCR:$pred),
                                 (VTI.Vec MQPR:$inactive))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$val), ARMVCCThen,
                             (VTI.Pred VCCR:$pred), zero_reg, (VTI.Vec MQPR:$inactive)))>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3607-3614
```tablegen
multiclass MVE_VRINT_ops<MVEVectorVTInfo VTI> {
  defm N : MVE_VRINT_m<VTI, "n", 0b000, any_froundeven>;
  defm X : MVE_VRINT_m<VTI, "x", 0b001, any_frint>;
  defm A : MVE_VRINT_m<VTI, "a", 0b010, any_fround>;
  defm Z : MVE_VRINT_m<VTI, "z", 0b011, any_ftrunc>;
  defm M : MVE_VRINT_m<VTI, "m", 0b101, any_ffloor>;
  defm P : MVE_VRINT_m<VTI, "p", 0b111, any_fceil>;
}
```
- EN: Declares TableGen `multiclass MVE_VRINT_ops`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VRINT_ops`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3616-3617
```tablegen
defm MVE_VRINTf16 : MVE_VRINT_ops<MVE_v8f16>;
defm MVE_VRINTf32 : MVE_VRINT_ops<MVE_v4f32>;
```
- EN: Defines TableGen record `MVE_VRINTf16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VRINTf16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3619-3625
```tablegen
class MVEFloatArithNeon<string iname, string suffix, bit size,
                           dag oops, dag iops, string ops,
                           vpred_ops vpred, string cstr, bits<2> vecsize, list<dag> pattern=[]>
  : MVE_float<iname, suffix, oops, iops, ops, vpred, cstr, vecsize, pattern> {
  let Inst{20} = size;
  let Inst{16} = 0b0;
}
```
- EN: Declares reusable TableGen class `MVEFloatArithNeon` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVEFloatArithNeon`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3627-3632
```tablegen
class MVE_VMUL_fp<string iname, string suffix, bits<2> size, list<dag> pattern=[]>
  : MVEFloatArithNeon<iname, suffix, size{0}, (outs MQPR:$Qd),
                      (ins MQPR:$Qn, MQPR:$Qm), "$Qd, $Qn, $Qm", vpred_r, "",
                      size, pattern> {
  bits<4> Qd;
  bits<4> Qn;
```
- EN: Declares reusable TableGen class `MVE_VMUL_fp` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VMUL_fp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3634-3644
```tablegen
  let Inst{28} = 0b1;
  let Inst{25-23} = 0b110;
  let Inst{22} = Qd{3};
  let Inst{21} = 0b0;
  let Inst{19-17} = Qn{2-0};
  let Inst{15-13} = Qd{2-0};
  let Inst{12-8} = 0b01101;
  let Inst{7} = Qn{3};
  let Inst{4} = 0b1;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3646-3649
```tablegen
multiclass MVE_VMULT_fp_m<string iname, MVEVectorVTInfo VTI, SDPatternOperator Op,
                          Intrinsic PredInt, SDPatternOperator IdentityVec> {
  def "" : MVE_VMUL_fp<iname, VTI.Suffix, VTI.Size>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 3651-3654
```tablegen
  let Predicates = [HasMVEFloat] in {
    defm : MVE_TwoOpPattern<VTI, Op, PredInt, (? ), !cast<Instruction>(NAME), IdentityVec>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3656-3657
```tablegen
multiclass MVE_VMUL_fp_m<MVEVectorVTInfo VTI, SDPatternOperator IdentityVec>
  : MVE_VMULT_fp_m<"vmul", VTI, vmul, int_arm_mve_mul_predicated, IdentityVec>;
```
- EN: Declares TableGen `multiclass MVE_VMUL_fp_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMUL_fp_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3659-3660
```tablegen
def ARMimmOneF: PatLeaf<(bitconvert (v4f32 (ARMvmovFPImm (i32 112))))>; // 1.0 float
def ARMimmOneH: PatLeaf<(bitconvert (v8i16 (ARMvmovImm (i32 2620))))>; // 1.0 half
```
- EN: Defines TableGen record `ARMimmOneF:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMimmOneF:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3662-3663
```tablegen
defm MVE_VMULf32 : MVE_VMUL_fp_m<MVE_v4f32, ARMimmOneF>;
defm MVE_VMULf16 : MVE_VMUL_fp_m<MVE_v8f16, ARMimmOneH>;
```
- EN: Defines TableGen record `MVE_VMULf32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMULf32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3665-3671
```tablegen
class MVE_VCMLA<string suffix, bits<2> size, string cstr>
  : MVEFloatArithNeon<"vcmla", suffix, size{1}, (outs MQPR:$Qd),
                         (ins MQPR:$Qd_src, MQPR:$Qn, MQPR:$Qm, complexrotateop:$rot),
                         "$Qd, $Qn, $Qm, $rot", vpred_n, "$Qd = $Qd_src"#cstr, size, []> {
  bits<4> Qd;
  bits<4> Qn;
  bits<2> rot;
```
- EN: Declares reusable TableGen class `MVE_VCMLA` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VCMLA`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3673-3683
```tablegen
  let Inst{28} = 0b1;
  let Inst{25} = 0b0;
  let Inst{24-23} = rot;
  let Inst{22} = Qd{3};
  let Inst{21} = 0b1;
  let Inst{19-17} = Qn{2-0};
  let Inst{15-13} = Qd{2-0};
  let Inst{12-8} = 0b01000;
  let Inst{7} = Qn{3};
  let Inst{4} = 0b0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3685-3687
```tablegen
multiclass MVE_VCMLA_m<MVEVectorVTInfo VTI, string cstr=""> {
  def "" : MVE_VCMLA<VTI.Suffix, VTI.Size, cstr>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 3689-3695
```tablegen
  let Predicates = [HasMVEFloat] in {
    def : Pat<(VTI.Vec (int_arm_mve_vcmlaq
                            imm:$rot, (VTI.Vec MQPR:$Qd_src),
                            (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qd_src),
                             (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm),
                             imm:$rot))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3697-3702
```tablegen
    def: Pat<(VTI.Vec (fadd_contract MQPR:$Qd_src,
                            (int_arm_mve_vcmulq imm:$rot,
                              (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm)))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qd_src),
                             (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm),
                             imm:$rot))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3704-3712
```tablegen
    def : Pat<(VTI.Vec (int_arm_mve_vcmlaq_predicated
                            imm:$rot, (VTI.Vec MQPR:$Qd_src),
                            (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm),
                            (VTI.Pred VCCR:$mask))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qd_src), (VTI.Vec MQPR:$Qn),
                             (VTI.Vec MQPR:$Qm), imm:$rot,
                             ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg))>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3714-3715
```tablegen
defm MVE_VCMLAf16 : MVE_VCMLA_m<MVE_v8f16>;
defm MVE_VCMLAf32 : MVE_VCMLA_m<MVE_v4f32, ",@earlyclobber $Qd">;
```
- EN: Defines TableGen record `MVE_VCMLAf16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VCMLAf16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3717-3725
```tablegen
class MVE_VADDSUBFMA_fp<string iname, string suffix, bits<2> size, bit bit_4,
                        bit bit_8, bit bit_21, dag iops=(ins),
                        vpred_ops vpred=vpred_r, string cstr="",
                        list<dag> pattern=[]>
  : MVEFloatArithNeon<iname, suffix, size{0}, (outs MQPR:$Qd),
                      !con(iops, (ins MQPR:$Qn, MQPR:$Qm)), "$Qd, $Qn, $Qm",
                      vpred, cstr, size, pattern> {
  bits<4> Qd;
  bits<4> Qn;
```
- EN: Declares reusable TableGen class `MVE_VADDSUBFMA_fp` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VADDSUBFMA_fp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3727-3738
```tablegen
  let Inst{28} = 0b0;
  let Inst{25-23} = 0b110;
  let Inst{22} = Qd{3};
  let Inst{21} = bit_21;
  let Inst{19-17} = Qn{2-0};
  let Inst{15-13} = Qd{2-0};
  let Inst{11-9} = 0b110;
  let Inst{8} = bit_8;
  let Inst{7} = Qn{3};
  let Inst{4} = bit_4;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3740-3748
```tablegen
multiclass MVE_VFMA_fp_multi<string iname, bit fms, MVEVectorVTInfo VTI> {
  def "" : MVE_VADDSUBFMA_fp<iname, VTI.Suffix, VTI.Size, 0b1, 0b0, fms,
                             (ins MQPR:$Qd_src), vpred_n, "$Qd = $Qd_src">;
  defvar Inst = !cast<Instruction>(NAME);
  defvar pred_int = int_arm_mve_fma_predicated;
  defvar m1   = (VTI.Vec MQPR:$m1);
  defvar m2   = (VTI.Vec MQPR:$m2);
  defvar add  = (VTI.Vec MQPR:$add);
  defvar pred = (VTI.Pred VCCR:$pred);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 3750-3767
```tablegen
  let Predicates = [HasMVEFloat] in {
    if fms then {
      def : Pat<(VTI.Vec (fma (fneg m1), m2, add)),
                (Inst $add, $m1, $m2)>;
      def : Pat<(VTI.Vec (int_arm_mve_fma (fneg m1), m2, add)),
                (Inst $add, $m1, $m2)>;
      def : Pat<(VTI.Vec (int_arm_mve_fma m1, (fneg m2), add)),
                (Inst $add, $m1, $m2)>;
      def : Pat<(VTI.Vec (vselect (VTI.Pred VCCR:$pred),
                                  (VTI.Vec (fma (fneg m1), m2, add)),
                                  add)),
                (Inst $add, $m1, $m2, ARMVCCThen, $pred, zero_reg)>;
      def : Pat<(VTI.Vec (pred_int (fneg m1), m2, add, pred)),
                (Inst $add, $m1, $m2, ARMVCCThen, $pred, zero_reg)>;
      def : Pat<(VTI.Vec (pred_int m1, (fneg m2), add, pred)),
                (Inst $add, $m1, $m2, ARMVCCThen, $pred, zero_reg)>;
    } else {
      def : Pat<(VTI.Vec (fma m1, m2, add)),
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3768-3779
```tablegen
                (Inst $add, $m1, $m2)>;
      def : Pat<(VTI.Vec (int_arm_mve_fma m1, m2, add)),
                (Inst $add, $m1, $m2)>;
      def : Pat<(VTI.Vec (vselect (VTI.Pred VCCR:$pred),
                                  (VTI.Vec (fma m1, m2, add)),
                                  add)),
                (Inst $add, $m1, $m2, ARMVCCThen, $pred, zero_reg)>;
      def : Pat<(VTI.Vec (pred_int m1, m2, add, pred)),
                (Inst $add, $m1, $m2, ARMVCCThen, $pred, zero_reg)>;
    }
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3781-3784
```tablegen
defm MVE_VFMAf32 : MVE_VFMA_fp_multi<"vfma", 0, MVE_v4f32>;
defm MVE_VFMAf16 : MVE_VFMA_fp_multi<"vfma", 0, MVE_v8f16>;
defm MVE_VFMSf32 : MVE_VFMA_fp_multi<"vfms", 1, MVE_v4f32>;
defm MVE_VFMSf16 : MVE_VFMA_fp_multi<"vfms", 1, MVE_v8f16>;
```
- EN: Defines TableGen record `MVE_VFMAf32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VFMAf32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3786-3791
```tablegen
multiclass MVE_VADDSUB_fp_m<string iname, bit bit_21, MVEVectorVTInfo VTI,
                            SDPatternOperator Op, Intrinsic PredInt, SDPatternOperator IdentityVec> {
  def "" : MVE_VADDSUBFMA_fp<iname, VTI.Suffix, VTI.Size, 0, 1, bit_21> {
    let validForTailPredication = 1;
  }
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 3793-3796
```tablegen
  let Predicates = [HasMVEFloat] in {
    defm : MVE_TwoOpPattern<VTI, Op, PredInt, (? ), !cast<Instruction>(NAME), IdentityVec>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3798-3801
```tablegen
multiclass MVE_VADD_fp_m<MVEVectorVTInfo VTI, SDPatternOperator IdentityVec>
  : MVE_VADDSUB_fp_m<"vadd", 0, VTI, vadd, int_arm_mve_add_predicated, IdentityVec>;
multiclass MVE_VSUB_fp_m<MVEVectorVTInfo VTI, SDPatternOperator IdentityVec>
  : MVE_VADDSUB_fp_m<"vsub", 1, VTI, vsub, int_arm_mve_sub_predicated, IdentityVec>;
```
- EN: Declares TableGen `multiclass MVE_VADD_fp_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VADD_fp_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3803-3804
```tablegen
def ARMimmMinusZeroF: PatLeaf<(bitconvert (v4i32 (ARMvmovImm (i32 1664))))>; // -0.0 float
def ARMimmMinusZeroH: PatLeaf<(bitconvert (v8i16 (ARMvmovImm (i32 2688))))>; // -0.0 half
```
- EN: Defines TableGen record `ARMimmMinusZeroF:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMimmMinusZeroF:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3806-3807
```tablegen
defm MVE_VADDf32 : MVE_VADD_fp_m<MVE_v4f32, ARMimmMinusZeroF>;
defm MVE_VADDf16 : MVE_VADD_fp_m<MVE_v8f16, ARMimmMinusZeroH>;
```
- EN: Defines TableGen record `MVE_VADDf32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VADDf32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3809-3810
```tablegen
defm MVE_VSUBf32 : MVE_VSUB_fp_m<MVE_v4f32, ARMimmAllZerosV>;
defm MVE_VSUBf16 : MVE_VSUB_fp_m<MVE_v8f16, ARMimmAllZerosV>;
```
- EN: Defines TableGen record `MVE_VSUBf32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSUBf32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3812-3818
```tablegen
class MVE_VCADD<string suffix, bits<2> size, string cstr="">
  : MVEFloatArithNeon<"vcadd", suffix, size{1}, (outs MQPR:$Qd),
                         (ins MQPR:$Qn, MQPR:$Qm, complexrotateopodd:$rot),
                         "$Qd, $Qn, $Qm, $rot", vpred_r, cstr, size, []> {
  bits<4> Qd;
  bits<4> Qn;
  bit rot;
```
- EN: Declares reusable TableGen class `MVE_VCADD` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VCADD`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3820-3831
```tablegen
  let Inst{28} = 0b1;
  let Inst{25} = 0b0;
  let Inst{24} = rot;
  let Inst{23} = 0b1;
  let Inst{22} = Qd{3};
  let Inst{21} = 0b0;
  let Inst{19-17} = Qn{2-0};
  let Inst{15-13} = Qd{2-0};
  let Inst{12-8} = 0b01000;
  let Inst{7} = Qn{3};
  let Inst{4} = 0b0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3833-3835
```tablegen
multiclass MVE_VCADD_m<MVEVectorVTInfo VTI, string cstr=""> {
  def "" : MVE_VCADD<VTI.Suffix, VTI.Size, cstr>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 3837-3841
```tablegen
  let Predicates = [HasMVEFloat] in {
    def : Pat<(VTI.Vec (int_arm_mve_vcaddq (i32 1),
                            imm:$rot, (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm),
                             imm:$rot))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3843-3849
```tablegen
    def : Pat<(VTI.Vec (int_arm_mve_vcaddq_predicated (i32 1),
                            imm:$rot, (VTI.Vec MQPR:$inactive),
                            (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm),
                            (VTI.Pred VCCR:$mask))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm),
                             imm:$rot, ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg,
                             (VTI.Vec MQPR:$inactive)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3851-3852
```tablegen
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3854-3855
```tablegen
defm MVE_VCADDf16 : MVE_VCADD_m<MVE_v8f16>;
defm MVE_VCADDf32 : MVE_VCADD_m<MVE_v4f32, "@earlyclobber $Qd">;
```
- EN: Defines TableGen record `MVE_VCADDf16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VCADDf16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3857-3861
```tablegen
class MVE_VABD_fp<string suffix, bits<2> size>
  : MVE_float<"vabd", suffix, (outs MQPR:$Qd), (ins MQPR:$Qn, MQPR:$Qm),
              "$Qd, $Qn, $Qm", vpred_r, "", size> {
  bits<4> Qd;
  bits<4> Qn;
```
- EN: Declares reusable TableGen class `MVE_VABD_fp` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VABD_fp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3863-3875
```tablegen
  let Inst{28} = 0b1;
  let Inst{25-23} = 0b110;
  let Inst{22} = Qd{3};
  let Inst{21} = 0b1;
  let Inst{20} = size{0};
  let Inst{19-17} = Qn{2-0};
  let Inst{16} = 0b0;
  let Inst{15-13} = Qd{2-0};
  let Inst{11-8} = 0b1101;
  let Inst{7} = Qn{3};
  let Inst{4} = 0b0;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3877-3880
```tablegen
multiclass MVE_VABDT_fp_m<MVEVectorVTInfo VTI,
                            Intrinsic unpred_int, Intrinsic pred_int> {
  def "" : MVE_VABD_fp<VTI.Suffix, VTI.Size>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 3882-3893
```tablegen
  let Predicates = [HasMVEFloat] in {
    def : Pat<(VTI.Vec (unpred_int (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn),
                            (i32 0))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn)))>;
    def : Pat<(VTI.Vec (pred_int (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn),
                            (i32 0), (VTI.Pred VCCR:$mask),
                            (VTI.Vec MQPR:$inactive))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn),
                             ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg,
                             (VTI.Vec MQPR:$inactive)))>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3895-3896
```tablegen
multiclass MVE_VABD_fp_m<MVEVectorVTInfo VTI>
  : MVE_VABDT_fp_m<VTI, int_arm_mve_vabd, int_arm_mve_abd_predicated>;
```
- EN: Declares TableGen `multiclass MVE_VABD_fp_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VABD_fp_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3898-3899
```tablegen
defm MVE_VABDf32 : MVE_VABD_fp_m<MVE_v4f32>;
defm MVE_VABDf16 : MVE_VABD_fp_m<MVE_v8f16>;
```
- EN: Defines TableGen record `MVE_VABDf32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VABDf32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3901-3906
```tablegen
let Predicates = [HasMVEFloat] in {
  def : Pat<(v8f16 (fabs (fsub (v8f16 MQPR:$Qm), (v8f16 MQPR:$Qn)))),
            (MVE_VABDf16 MQPR:$Qm, MQPR:$Qn)>;
  def : Pat<(v4f32 (fabs (fsub (v4f32 MQPR:$Qm), (v4f32 MQPR:$Qn)))),
            (MVE_VABDf32 MQPR:$Qm, MQPR:$Qn)>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3908-3914
```tablegen
class MVE_VCVT_fix<string suffix, bit fsi, bit U, bit op,
                   Operand imm_operand_type>
  : MVE_float<"vcvt", suffix,
              (outs MQPR:$Qd), (ins MQPR:$Qm, imm_operand_type:$imm6),
              "$Qd, $Qm, $imm6", vpred_r, "", !if(fsi, 0b10, 0b01), []> {
  bits<4> Qd;
  bits<6> imm6;
```
- EN: Declares reusable TableGen class `MVE_VCVT_fix` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VCVT_fix`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3916-3926
```tablegen
  let Inst{28} = U;
  let Inst{25-23} = 0b111;
  let Inst{22} = Qd{3};
  let Inst{21} = 0b1;
  let Inst{19-16} = imm6{3-0};
  let Inst{15-13} = Qd{2-0};
  let Inst{11-10} = 0b11;
  let Inst{9} = fsi;
  let Inst{8} = op;
  let Inst{7} = 0b0;
  let Inst{4} = 0b1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3928-3930
```tablegen
  let DecoderMethod = "DecodeMVEVCVTt1fp";
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3932-3943
```tablegen
class MVE_VCVT_imm_asmop<int Bits> : AsmOperandClass {
  let PredicateMethod = "isImmediate<1," # Bits # ">";
  let DiagnosticString =
      "MVE fixed-point immediate operand must be between 1 and " # Bits;
  let Name = "MVEVcvtImm" # Bits;
  let RenderMethod = "addImmOperands";
}
class MVE_VCVT_imm<int Bits>: Operand<i32> {
  let ParserMatchClass = MVE_VCVT_imm_asmop<Bits>;
  let EncoderMethod = "getNEONVcvtImm32OpValue";
  let DecoderMethod = "DecodeVCVTImmOperand";
}
```
- EN: Declares reusable TableGen class `MVE_VCVT_imm_asmop` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VCVT_imm_asmop`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3945-3952
```tablegen
class MVE_VCVT_fix_f32<string suffix, bit U, bit op>
    : MVE_VCVT_fix<suffix, 0b1, U, op, MVE_VCVT_imm<32>> {
  let Inst{20} = imm6{4};
}
class MVE_VCVT_fix_f16<string suffix, bit U, bit op>
    : MVE_VCVT_fix<suffix, 0b0, U, op, MVE_VCVT_imm<16>> {
  let Inst{20} = 0b1;
}
```
- EN: Declares reusable TableGen class `MVE_VCVT_fix_f32` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VCVT_fix_f32`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3954-3969
```tablegen
multiclass MVE_VCVT_fix_patterns<Instruction Inst, bit U, MVEVectorVTInfo DestVTI,
                                 MVEVectorVTInfo SrcVTI> {
  let Predicates = [HasMVEFloat] in {
    def : Pat<(DestVTI.Vec (int_arm_mve_vcvt_fix
                              (i32 U), (SrcVTI.Vec MQPR:$Qm), imm:$scale)),
              (DestVTI.Vec (Inst (SrcVTI.Vec MQPR:$Qm), imm:$scale))>;
    def : Pat<(DestVTI.Vec (int_arm_mve_vcvt_fix_predicated (i32 U),
                              (DestVTI.Vec MQPR:$inactive),
                              (SrcVTI.Vec MQPR:$Qm),
                              imm:$scale,
                              (DestVTI.Pred VCCR:$mask))),
              (DestVTI.Vec (Inst (SrcVTI.Vec MQPR:$Qm), imm:$scale,
                             ARMVCCThen, (DestVTI.Pred VCCR:$mask), zero_reg,
                             (DestVTI.Vec MQPR:$inactive)))>;
  }
}
```
- EN: Declares TableGen `multiclass MVE_VCVT_fix_patterns`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VCVT_fix_patterns`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3971-3975
```tablegen
multiclass MVE_VCVT_fix_f32_m<bit U, bit op,
                              MVEVectorVTInfo DestVTI, MVEVectorVTInfo SrcVTI> {
  def "" : MVE_VCVT_fix_f32<DestVTI.Suffix#"."#SrcVTI.Suffix, U, op>;
  defm : MVE_VCVT_fix_patterns<!cast<Instruction>(NAME), U, DestVTI, SrcVTI>;
}
```
- EN: Declares TableGen `multiclass MVE_VCVT_fix_f32_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VCVT_fix_f32_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3977-3981
```tablegen
multiclass MVE_VCVT_fix_f16_m<bit U, bit op,
                              MVEVectorVTInfo DestVTI, MVEVectorVTInfo SrcVTI> {
  def "" : MVE_VCVT_fix_f16<DestVTI.Suffix#"."#SrcVTI.Suffix, U, op>;
  defm : MVE_VCVT_fix_patterns<!cast<Instruction>(NAME), U, DestVTI, SrcVTI>;
}
```
- EN: Declares TableGen `multiclass MVE_VCVT_fix_f16_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VCVT_fix_f16_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3983-3990
```tablegen
defm MVE_VCVTf16s16_fix : MVE_VCVT_fix_f16_m<0b0, 0b0, MVE_v8f16, MVE_v8s16>;
defm MVE_VCVTs16f16_fix : MVE_VCVT_fix_f16_m<0b0, 0b1, MVE_v8s16, MVE_v8f16>;
defm MVE_VCVTf16u16_fix : MVE_VCVT_fix_f16_m<0b1, 0b0, MVE_v8f16, MVE_v8u16>;
defm MVE_VCVTu16f16_fix : MVE_VCVT_fix_f16_m<0b1, 0b1, MVE_v8u16, MVE_v8f16>;
defm MVE_VCVTf32s32_fix : MVE_VCVT_fix_f32_m<0b0, 0b0, MVE_v4f32, MVE_v4s32>;
defm MVE_VCVTs32f32_fix : MVE_VCVT_fix_f32_m<0b0, 0b1, MVE_v4s32, MVE_v4f32>;
defm MVE_VCVTf32u32_fix : MVE_VCVT_fix_f32_m<0b1, 0b0, MVE_v4f32, MVE_v4u32>;
defm MVE_VCVTu32f32_fix : MVE_VCVT_fix_f32_m<0b1, 0b1, MVE_v4u32, MVE_v4f32>;
```
- EN: Defines TableGen record `MVE_VCVTf16s16_fix` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VCVTf16s16_fix`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3992-3996
```tablegen
class MVE_VCVT_fp_int_anpm<string suffix, bits<2> size, bit op, string anpm,
                bits<2> rm, list<dag> pattern=[]>
  : MVE_float<!strconcat("vcvt", anpm), suffix, (outs MQPR:$Qd),
              (ins MQPR:$Qm), "$Qd, $Qm", vpred_r, "", size, pattern> {
  bits<4> Qd;
```
- EN: Declares reusable TableGen class `MVE_VCVT_fp_int_anpm` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VCVT_fp_int_anpm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3998-4010
```tablegen
  let Inst{28} = 0b1;
  let Inst{25-23} = 0b111;
  let Inst{22} = Qd{3};
  let Inst{21-20} = 0b11;
  let Inst{19-18} = size;
  let Inst{17-16} = 0b11;
  let Inst{15-13} = Qd{2-0};
  let Inst{12-10} = 0b000;
  let Inst{9-8} = rm;
  let Inst{7} = op;
  let Inst{4} = 0b0;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4012-4015
```tablegen
multiclass MVE_VCVT_fp_int_anpm_inner<MVEVectorVTInfo Int, MVEVectorVTInfo Flt,
                                      string anpm, bits<2> rm> {
  def "": MVE_VCVT_fp_int_anpm<Int.Suffix # "." # Flt.Suffix, Int.Size,
                               Int.Unsigned, anpm, rm>;
```
- EN: Declares TableGen `multiclass MVE_VCVT_fp_int_anpm_inner`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VCVT_fp_int_anpm_inner`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4017-4020
```tablegen
  defvar Inst         = !cast<Instruction>(NAME);
  defvar IntrBaseName = "int_arm_mve_vcvt" # anpm;
  defvar UnpredIntr   = !cast<Intrinsic>(IntrBaseName);
  defvar PredIntr     = !cast<Intrinsic>(IntrBaseName # "_predicated");
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 4022-4024
```tablegen
  let Predicates = [HasMVEFloat] in {
    def : Pat<(Int.Vec (UnpredIntr (i32 Int.Unsigned), (Flt.Vec MQPR:$in))),
              (Int.Vec (Inst (Flt.Vec MQPR:$in)))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4026-4031
```tablegen
    def : Pat<(Int.Vec (PredIntr (i32 Int.Unsigned), (Int.Vec MQPR:$inactive),
                                 (Flt.Vec MQPR:$in), (Flt.Pred VCCR:$pred))),
              (Int.Vec (Inst (Flt.Vec MQPR:$in), ARMVCCThen,
                             (Flt.Pred VCCR:$pred), zero_reg, (Int.Vec MQPR:$inactive)))>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4033-4039
```tablegen
multiclass MVE_VCVT_fp_int_anpm_outer<MVEVectorVTInfo Int,
                                      MVEVectorVTInfo Flt> {
  defm a : MVE_VCVT_fp_int_anpm_inner<Int, Flt, "a", 0b00>;
  defm n : MVE_VCVT_fp_int_anpm_inner<Int, Flt, "n", 0b01>;
  defm p : MVE_VCVT_fp_int_anpm_inner<Int, Flt, "p", 0b10>;
  defm m : MVE_VCVT_fp_int_anpm_inner<Int, Flt, "m", 0b11>;
}
```
- EN: Declares TableGen `multiclass MVE_VCVT_fp_int_anpm_outer`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VCVT_fp_int_anpm_outer`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4041-4047
```tablegen
// This defines instructions such as MVE_VCVTu16f16a, with an explicit
// rounding-mode suffix on the mnemonic. The class below will define
// the bare MVE_VCVTu16f16 (with implied rounding toward zero).
defm MVE_VCVTs16f16 : MVE_VCVT_fp_int_anpm_outer<MVE_v8s16, MVE_v8f16>;
defm MVE_VCVTu16f16 : MVE_VCVT_fp_int_anpm_outer<MVE_v8u16, MVE_v8f16>;
defm MVE_VCVTs32f32 : MVE_VCVT_fp_int_anpm_outer<MVE_v4s32, MVE_v4f32>;
defm MVE_VCVTu32f32 : MVE_VCVT_fp_int_anpm_outer<MVE_v4u32, MVE_v4f32>;
```
- EN: Declares reusable TableGen class `below` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `below`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4049-4053
```tablegen
class MVE_VCVT_fp_int<string suffix, bits<2> size, bit toint, bit unsigned,
                      list<dag> pattern=[]>
  : MVE_float<"vcvt", suffix, (outs MQPR:$Qd),
              (ins MQPR:$Qm), "$Qd, $Qm", vpred_r, "", size, pattern> {
  bits<4> Qd;
```
- EN: Declares reusable TableGen class `MVE_VCVT_fp_int` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VCVT_fp_int`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4055-4067
```tablegen
  let Inst{28} = 0b1;
  let Inst{25-23} = 0b111;
  let Inst{22} = Qd{3};
  let Inst{21-20} = 0b11;
  let Inst{19-18} = size;
  let Inst{17-16} = 0b11;
  let Inst{15-13} = Qd{2-0};
  let Inst{12-9} = 0b0011;
  let Inst{8} = toint;
  let Inst{7} = unsigned;
  let Inst{4} = 0b0;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4069-4072
```tablegen
multiclass MVE_VCVT_fp_int_m<MVEVectorVTInfo Dest, MVEVectorVTInfo Src,
                             SDNode unpred_op, SDPatternOperator unpred_intrinsic> {
  defvar Unsigned = !or(!eq(Dest.SuffixLetter,"u"), !eq(Src.SuffixLetter,"u"));
  defvar ToInt = !eq(Src.SuffixLetter,"f");
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 4074-4076
```tablegen
  def "" : MVE_VCVT_fp_int<Dest.Suffix # "." # Src.Suffix, Dest.Size,
                           ToInt, Unsigned>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 4078-4095
```tablegen
  let Predicates = [HasMVEFloat] in {
    def : Pat<(Dest.Vec (unpred_op (Src.Vec MQPR:$src))),
              (Dest.Vec (Inst (Src.Vec MQPR:$src)))>;
    def : Pat<(Dest.Vec (unpred_intrinsic (Src.Vec MQPR:$src), (i32 Unsigned))),
              (Dest.Vec (Inst (Src.Vec MQPR:$src)))>;
    def : Pat<(Dest.Vec (int_arm_mve_vcvt_fp_int_predicated
                             (Src.Vec MQPR:$src), (i32 Unsigned),
                             (Src.Pred VCCR:$mask), (Dest.Vec MQPR:$inactive))),
              (Dest.Vec (Inst (Src.Vec MQPR:$src), ARMVCCThen,
                              (Src.Pred VCCR:$mask), zero_reg,
                              (Dest.Vec MQPR:$inactive)))>;
  }
}
// The unsuffixed VCVT for float->int implicitly rounds toward zero,
// which I reflect here in the llvm instruction names
defm MVE_VCVTs16f16z : MVE_VCVT_fp_int_m<MVE_v8s16, MVE_v8f16, fp_to_sint, int_arm_mve_vcvt_int_fp>;
defm MVE_VCVTu16f16z : MVE_VCVT_fp_int_m<MVE_v8u16, MVE_v8f16, fp_to_uint, int_arm_mve_vcvt_int_fp>;
defm MVE_VCVTs32f32z : MVE_VCVT_fp_int_m<MVE_v4s32, MVE_v4f32, fp_to_sint, int_arm_mve_vcvt_int_fp>;
```
- EN: Defines TableGen record `MVE_VCVTs16f16z` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VCVTs16f16z`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4096-4101
```tablegen
defm MVE_VCVTu32f32z : MVE_VCVT_fp_int_m<MVE_v4u32, MVE_v4f32, fp_to_uint, int_arm_mve_vcvt_int_fp>;
// Whereas VCVT for int->float rounds to nearest
defm MVE_VCVTf16s16n : MVE_VCVT_fp_int_m<MVE_v8f16, MVE_v8s16, sint_to_fp, int_arm_mve_vcvt_fp_int>;
defm MVE_VCVTf16u16n : MVE_VCVT_fp_int_m<MVE_v8f16, MVE_v8u16, uint_to_fp, int_arm_mve_vcvt_fp_int>;
defm MVE_VCVTf32s32n : MVE_VCVT_fp_int_m<MVE_v4f32, MVE_v4s32, sint_to_fp, int_arm_mve_vcvt_fp_int>;
defm MVE_VCVTf32u32n : MVE_VCVT_fp_int_m<MVE_v4f32, MVE_v4u32, uint_to_fp, int_arm_mve_vcvt_fp_int>;
```
- EN: Defines TableGen record `MVE_VCVTu32f32z` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VCVTu32f32z`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4103-4112
```tablegen
let Predicates = [HasMVEFloat] in {
  def : Pat<(v4i32 (fp_to_sint_sat v4f32:$src, i32)),
            (MVE_VCVTs32f32z v4f32:$src)>;
  def : Pat<(v4i32 (fp_to_uint_sat v4f32:$src, i32)),
            (MVE_VCVTu32f32z v4f32:$src)>;
  def : Pat<(v8i16 (fp_to_sint_sat v8f16:$src, i16)),
            (MVE_VCVTs16f16z v8f16:$src)>;
  def : Pat<(v8i16 (fp_to_uint_sat v8f16:$src, i16)),
            (MVE_VCVTu16f16z v8f16:$src)>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4114-4118
```tablegen
class MVE_VABSNEG_fp<string iname, string suffix, bits<2> size, bit negate,
                   list<dag> pattern=[]>
  : MVE_float<iname, suffix, (outs MQPR:$Qd),
              (ins MQPR:$Qm), "$Qd, $Qm", vpred_r, "", size, pattern> {
  bits<4> Qd;
```
- EN: Declares reusable TableGen class `MVE_VABSNEG_fp` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VABSNEG_fp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4120-4131
```tablegen
  let Inst{28} = 0b1;
  let Inst{25-23} = 0b111;
  let Inst{22} = Qd{3};
  let Inst{21-20} = 0b11;
  let Inst{19-18} = size;
  let Inst{17-16} = 0b01;
  let Inst{15-13} = Qd{2-0};
  let Inst{11-8} = 0b0111;
  let Inst{7} = negate;
  let Inst{4} = 0b0;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4133-4136
```tablegen
multiclass MVE_VABSNEG_fp_m<string iname, SDNode unpred_op, Intrinsic pred_int,
                            MVEVectorVTInfo VTI, bit opcode> {
  def "" : MVE_VABSNEG_fp<iname, VTI.Suffix, VTI.Size, opcode>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 4138-4145
```tablegen
  let Predicates = [HasMVEInt] in {
    def : Pat<(VTI.Vec (unpred_op (VTI.Vec MQPR:$v))),
              (VTI.Vec (Inst $v))>;
    def : Pat<(VTI.Vec (pred_int  (VTI.Vec MQPR:$v), (VTI.Pred VCCR:$mask),
                                  (VTI.Vec MQPR:$inactive))),
              (VTI.Vec (Inst $v, ARMVCCThen, $mask, zero_reg, $inactive))>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4147-4154
```tablegen
defm MVE_VABSf16 : MVE_VABSNEG_fp_m<"vabs", fabs, int_arm_mve_abs_predicated,
                                    MVE_v8f16, 0>;
defm MVE_VABSf32 : MVE_VABSNEG_fp_m<"vabs", fabs, int_arm_mve_abs_predicated,
                                    MVE_v4f32, 0>;
defm MVE_VNEGf16 : MVE_VABSNEG_fp_m<"vneg", fneg, int_arm_mve_neg_predicated,
                                    MVE_v8f16, 1>;
defm MVE_VNEGf32 : MVE_VABSNEG_fp_m<"vneg", fneg, int_arm_mve_neg_predicated,
                                    MVE_v4f32, 1>;
```
- EN: Defines TableGen record `MVE_VABSf16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VABSf16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4156-4162
```tablegen
class MVE_VMAXMINNMA<string iname, string suffix, bits<2> size, bit bit_12,
                     list<dag> pattern=[]>
  : MVE_f<(outs MQPR:$Qd), (ins MQPR:$Qd_src, MQPR:$Qm),
          NoItinerary, iname, suffix, "$Qd, $Qm", vpred_n, "$Qd = $Qd_src",
          size, pattern> {
  bits<4> Qd;
  bits<4> Qm;
```
- EN: Declares reusable TableGen class `MVE_VMAXMINNMA` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VMAXMINNMA`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4164-4174
```tablegen
  let Inst{28} = size{0};
  let Inst{25-23} = 0b100;
  let Inst{22} = Qd{3};
  let Inst{21-16} = 0b111111;
  let Inst{15-13} = Qd{2-0};
  let Inst{12} = bit_12;
  let Inst{11-6} = 0b111010;
  let Inst{5} = Qm{3};
  let Inst{4} = 0b0;
  let Inst{3-1} = Qm{2-0};
  let Inst{0} = 0b1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4176-4178
```tablegen
  let isCommutable = 1;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4180-4184
```tablegen
multiclass MVE_VMAXMINNMA_m<string iname, MVEVectorVTInfo VTI,
                      SDPatternOperator unpred_op, Intrinsic pred_int,
                      bit bit_12> {
  def "" : MVE_VMAXMINNMA<iname, VTI.Suffix, VTI.Size, bit_12>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 4186-4190
```tablegen
  let Predicates = [HasMVEInt] in {
    // Unpredicated v(max|min)nma
    def : Pat<(VTI.Vec (unpred_op (fabs (VTI.Vec MQPR:$Qd)),
                                  (fabs (VTI.Vec MQPR:$Qm)))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qd), (VTI.Vec MQPR:$Qm)))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4192-4198
```tablegen
    // Predicated v(max|min)nma
    def : Pat<(VTI.Vec (pred_int (VTI.Vec MQPR:$Qd), (VTI.Vec MQPR:$Qm),
                            (VTI.Pred VCCR:$mask))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qd), (VTI.Vec MQPR:$Qm),
                            ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg))>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4200-4201
```tablegen
multiclass MVE_VMAXNMA<MVEVectorVTInfo VTI, bit bit_12>
  : MVE_VMAXMINNMA_m<"vmaxnma", VTI, vmaxnm, int_arm_mve_vmaxnma_predicated, bit_12>;
```
- EN: Declares TableGen `multiclass MVE_VMAXNMA`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMAXNMA`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4203-4204
```tablegen
defm MVE_VMAXNMAf32 : MVE_VMAXNMA<MVE_v4f32, 0b0>;
defm MVE_VMAXNMAf16 : MVE_VMAXNMA<MVE_v8f16, 0b0>;
```
- EN: Defines TableGen record `MVE_VMAXNMAf32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMAXNMAf32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4206-4207
```tablegen
multiclass MVE_VMINNMA<MVEVectorVTInfo VTI, bit bit_12>
  : MVE_VMAXMINNMA_m<"vminnma", VTI, vminnm, int_arm_mve_vminnma_predicated, bit_12>;
```
- EN: Declares TableGen `multiclass MVE_VMINNMA`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMINNMA`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4209-4210
```tablegen
defm MVE_VMINNMAf32 : MVE_VMINNMA<MVE_v4f32, 0b1>;
defm MVE_VMINNMAf16 : MVE_VMINNMA<MVE_v8f16, 0b1>;
```
- EN: Defines TableGen record `MVE_VMINNMAf32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMINNMAf32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4212-4212
```tablegen
// end of MVE Floating Point instructions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 4214-4214
```tablegen
// start of MVE compares
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 4216-4223
```tablegen
class MVE_VCMPqq<string suffix, bit bit_28, bits<2> bits_21_20,
                 VCMPPredicateOperand predtype, bits<2> vecsize, list<dag> pattern=[]>
  : MVE_p<(outs VCCR:$P0), (ins MQPR:$Qn, MQPR:$Qm, predtype:$fc),
           NoItinerary, "vcmp", suffix, "$fc, $Qn, $Qm", vpred_n, "", vecsize, pattern> {
  // Base class for comparing two vector registers
  bits<3> fc;
  bits<4> Qn;
  bits<4> Qm;
```
- EN: Declares reusable TableGen class `MVE_VCMPqq` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VCMPqq`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4225-4237
```tablegen
  let Inst{28} = bit_28;
  let Inst{25-22} = 0b1000;
  let Inst{21-20} = bits_21_20;
  let Inst{19-17} = Qn{2-0};
  let Inst{16-13} = 0b1000;
  let Inst{12} = fc{2};
  let Inst{11-8} = 0b1111;
  let Inst{7} = fc{0};
  let Inst{6} = 0b0;
  let Inst{5} = Qm{3};
  let Inst{4} = 0b0;
  let Inst{3-1} = Qm{2-0};
  let Inst{0} = fc{1};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4239-4239
```tablegen
  let Constraints = "";
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4241-4251
```tablegen
  // We need a custom decoder method for these instructions because of
  // the output VCCR operand, which isn't encoded in the instruction
  // bits anywhere (there is only one choice for it) but has to be
  // included in the MC operands so that codegen will be able to track
  // its data flow between instructions, spill/reload it when
  // necessary, etc. There seems to be no way to get the Tablegen
  // decoder to emit an operand that isn't affected by any instruction
  // bit.
  let DecoderMethod = "DecodeMVEVCMP<false," # predtype.DecoderMethod # ">";
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4253-4256
```tablegen
class MVE_VCMPqqf<string suffix, bit size>
    : MVE_VCMPqq<suffix, size, 0b11, pred_basic_fp, !if(size, 0b01, 0b10)> {
  let Predicates = [HasMVEFloat];
}
```
- EN: Declares reusable TableGen class `MVE_VCMPqqf` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VCMPqqf`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4258-4262
```tablegen
class MVE_VCMPqqi<string suffix, bits<2> size>
    : MVE_VCMPqq<suffix, 0b1, size, pred_basic_i, size> {
  let Inst{12} = 0b0;
  let Inst{0} = 0b0;
}
```
- EN: Declares reusable TableGen class `MVE_VCMPqqi` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VCMPqqi`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4264-4268
```tablegen
class MVE_VCMPqqu<string suffix, bits<2> size>
    : MVE_VCMPqq<suffix, 0b1, size, pred_basic_u, size> {
  let Inst{12} = 0b0;
  let Inst{0} = 0b1;
}
```
- EN: Declares reusable TableGen class `MVE_VCMPqqu` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VCMPqqu`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4270-4273
```tablegen
class MVE_VCMPqqs<string suffix, bits<2> size>
    : MVE_VCMPqq<suffix, 0b1, size, pred_basic_s, size> {
  let Inst{12} = 0b1;
}
```
- EN: Declares reusable TableGen class `MVE_VCMPqqs` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VCMPqqs`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4275-4276
```tablegen
def MVE_VCMPf32 : MVE_VCMPqqf<"f32", 0b0>;
def MVE_VCMPf16 : MVE_VCMPqqf<"f16", 0b1>;
```
- EN: Defines TableGen record `MVE_VCMPf32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VCMPf32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4278-4280
```tablegen
def MVE_VCMPi8  : MVE_VCMPqqi<"i8",  0b00>;
def MVE_VCMPi16 : MVE_VCMPqqi<"i16", 0b01>;
def MVE_VCMPi32 : MVE_VCMPqqi<"i32", 0b10>;
```
- EN: Defines TableGen record `MVE_VCMPi8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VCMPi8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4282-4284
```tablegen
def MVE_VCMPu8  : MVE_VCMPqqu<"u8",  0b00>;
def MVE_VCMPu16 : MVE_VCMPqqu<"u16", 0b01>;
def MVE_VCMPu32 : MVE_VCMPqqu<"u32", 0b10>;
```
- EN: Defines TableGen record `MVE_VCMPu8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VCMPu8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4286-4288
```tablegen
def MVE_VCMPs8  : MVE_VCMPqqs<"s8",  0b00>;
def MVE_VCMPs16 : MVE_VCMPqqs<"s16", 0b01>;
def MVE_VCMPs32 : MVE_VCMPqqs<"s32", 0b10>;
```
- EN: Defines TableGen record `MVE_VCMPs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VCMPs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4290-4297
```tablegen
class MVE_VCMPqr<string suffix, bit bit_28, bits<2> bits_21_20,
                 VCMPPredicateOperand predtype, bits<2> vecsize, list<dag> pattern=[]>
  : MVE_p<(outs VCCR:$P0), (ins MQPR:$Qn, GPRwithZR:$Rm, predtype:$fc),
           NoItinerary, "vcmp", suffix, "$fc, $Qn, $Rm", vpred_n, "", vecsize, pattern> {
  // Base class for comparing a vector register with a scalar
  bits<3> fc;
  bits<4> Qn;
  bits<4> Rm;
```
- EN: Declares reusable TableGen class `MVE_VCMPqr` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VCMPqr`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4299-4310
```tablegen
  let Inst{28} = bit_28;
  let Inst{25-22} = 0b1000;
  let Inst{21-20} = bits_21_20;
  let Inst{19-17} = Qn{2-0};
  let Inst{16-13} = 0b1000;
  let Inst{12} = fc{2};
  let Inst{11-8} = 0b1111;
  let Inst{7} = fc{0};
  let Inst{6} = 0b1;
  let Inst{5} = fc{1};
  let Inst{4} = 0b0;
  let Inst{3-0} = Rm{3-0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4312-4316
```tablegen
  let Constraints = "";
  // Custom decoder method, for the same reason as MVE_VCMPqq
  let DecoderMethod = "DecodeMVEVCMP<true," # predtype.DecoderMethod # ">";
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4318-4321
```tablegen
class MVE_VCMPqrf<string suffix, bit size>
    : MVE_VCMPqr<suffix, size, 0b11, pred_basic_fp, !if(size, 0b01, 0b10)> {
  let Predicates = [HasMVEFloat];
}
```
- EN: Declares reusable TableGen class `MVE_VCMPqrf` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VCMPqrf`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4323-4327
```tablegen
class MVE_VCMPqri<string suffix, bits<2> size>
    : MVE_VCMPqr<suffix, 0b1, size, pred_basic_i, size> {
  let Inst{12} = 0b0;
  let Inst{5} = 0b0;
}
```
- EN: Declares reusable TableGen class `MVE_VCMPqri` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VCMPqri`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4329-4333
```tablegen
class MVE_VCMPqru<string suffix, bits<2> size>
    : MVE_VCMPqr<suffix, 0b1, size, pred_basic_u, size> {
  let Inst{12} = 0b0;
  let Inst{5} = 0b1;
}
```
- EN: Declares reusable TableGen class `MVE_VCMPqru` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VCMPqru`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4335-4338
```tablegen
class MVE_VCMPqrs<string suffix, bits<2> size>
    : MVE_VCMPqr<suffix, 0b1, size, pred_basic_s, size> {
  let Inst{12} = 0b1;
}
```
- EN: Declares reusable TableGen class `MVE_VCMPqrs` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VCMPqrs`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4340-4341
```tablegen
def MVE_VCMPf32r : MVE_VCMPqrf<"f32", 0b0>;
def MVE_VCMPf16r : MVE_VCMPqrf<"f16", 0b1>;
```
- EN: Defines TableGen record `MVE_VCMPf32r` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VCMPf32r`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4343-4345
```tablegen
def MVE_VCMPi8r  : MVE_VCMPqri<"i8",  0b00>;
def MVE_VCMPi16r : MVE_VCMPqri<"i16", 0b01>;
def MVE_VCMPi32r : MVE_VCMPqri<"i32", 0b10>;
```
- EN: Defines TableGen record `MVE_VCMPi8r` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VCMPi8r`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4347-4349
```tablegen
def MVE_VCMPu8r  : MVE_VCMPqru<"u8",  0b00>;
def MVE_VCMPu16r : MVE_VCMPqru<"u16", 0b01>;
def MVE_VCMPu32r : MVE_VCMPqru<"u32", 0b10>;
```
- EN: Defines TableGen record `MVE_VCMPu8r` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VCMPu8r`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4351-4353
```tablegen
def MVE_VCMPs8r  : MVE_VCMPqrs<"s8",  0b00>;
def MVE_VCMPs16r : MVE_VCMPqrs<"s16", 0b01>;
def MVE_VCMPs32r : MVE_VCMPqrs<"s32", 0b10>;
```
- EN: Defines TableGen record `MVE_VCMPs8r` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VCMPs8r`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4355-4361
```tablegen
multiclass unpred_vcmp_z<string suffix, PatLeaf fc> {
  def i8  : Pat<(v16i1 (ARMvcmpz (v16i8 MQPR:$v1), fc)),
                (v16i1 (!cast<Instruction>("MVE_VCMP"#suffix#"8r") (v16i8 MQPR:$v1), ZR, fc))>;
  def i16 : Pat<(v8i1 (ARMvcmpz (v8i16 MQPR:$v1), fc)),
                (v8i1 (!cast<Instruction>("MVE_VCMP"#suffix#"16r") (v8i16 MQPR:$v1), ZR, fc))>;
  def i32 : Pat<(v4i1 (ARMvcmpz (v4i32 MQPR:$v1), fc)),
                (v4i1 (!cast<Instruction>("MVE_VCMP"#suffix#"32r") (v4i32 MQPR:$v1), ZR, fc))>;
```
- EN: Declares TableGen `multiclass unpred_vcmp_z`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass unpred_vcmp_z`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4363-4369
```tablegen
  def : Pat<(v16i1 (and (v16i1 VCCR:$p1), (v16i1 (ARMvcmpz (v16i8 MQPR:$v1), fc)))),
            (v16i1 (!cast<Instruction>("MVE_VCMP"#suffix#"8r") (v16i8 MQPR:$v1), ZR, fc, ARMVCCThen, VCCR:$p1, zero_reg))>;
  def : Pat<(v8i1 (and (v8i1 VCCR:$p1), (v8i1 (ARMvcmpz (v8i16 MQPR:$v1), fc)))),
            (v8i1 (!cast<Instruction>("MVE_VCMP"#suffix#"16r") (v8i16 MQPR:$v1), ZR, fc, ARMVCCThen, VCCR:$p1, zero_reg))>;
  def : Pat<(v4i1 (and (v4i1 VCCR:$p1), (v4i1 (ARMvcmpz (v4i32 MQPR:$v1), fc)))),
            (v4i1 (!cast<Instruction>("MVE_VCMP"#suffix#"32r") (v4i32 MQPR:$v1), ZR, fc, ARMVCCThen, VCCR:$p1, zero_reg))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4371-4377
```tablegen
multiclass unpred_vcmp_r<string suffix, PatLeaf fc> {
  def i8  : Pat<(v16i1 (ARMvcmp (v16i8 MQPR:$v1), (v16i8 MQPR:$v2), fc)),
                (v16i1 (!cast<Instruction>("MVE_VCMP"#suffix#"8") (v16i8 MQPR:$v1), (v16i8 MQPR:$v2), fc))>;
  def i16 : Pat<(v8i1 (ARMvcmp (v8i16 MQPR:$v1), (v8i16 MQPR:$v2), fc)),
                (v8i1 (!cast<Instruction>("MVE_VCMP"#suffix#"16") (v8i16 MQPR:$v1), (v8i16 MQPR:$v2), fc))>;
  def i32 : Pat<(v4i1 (ARMvcmp (v4i32 MQPR:$v1), (v4i32 MQPR:$v2), fc)),
                (v4i1 (!cast<Instruction>("MVE_VCMP"#suffix#"32") (v4i32 MQPR:$v1), (v4i32 MQPR:$v2), fc))>;
```
- EN: Declares TableGen `multiclass unpred_vcmp_r`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass unpred_vcmp_r`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4379-4384
```tablegen
  def i8r  : Pat<(v16i1 (ARMvcmp (v16i8 MQPR:$v1), (v16i8 (ARMvdup rGPR:$v2)), fc)),
                 (v16i1 (!cast<Instruction>("MVE_VCMP"#suffix#"8r") (v16i8 MQPR:$v1), (i32 rGPR:$v2), fc))>;
  def i16r : Pat<(v8i1 (ARMvcmp (v8i16 MQPR:$v1), (v8i16 (ARMvdup rGPR:$v2)), fc)),
                 (v8i1 (!cast<Instruction>("MVE_VCMP"#suffix#"16r") (v8i16 MQPR:$v1), (i32 rGPR:$v2), fc))>;
  def i32r : Pat<(v4i1 (ARMvcmp (v4i32 MQPR:$v1), (v4i32 (ARMvdup rGPR:$v2)), fc)),
                 (v4i1 (!cast<Instruction>("MVE_VCMP"#suffix#"32r") (v4i32 MQPR:$v1), (i32 rGPR:$v2), fc))>;
```
- EN: Defines TableGen record `i8r` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `i8r`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4386-4391
```tablegen
  def : Pat<(v16i1 (and (v16i1 VCCR:$p1), (v16i1 (ARMvcmp (v16i8 MQPR:$v1), (v16i8 MQPR:$v2), fc)))),
            (v16i1 (!cast<Instruction>("MVE_VCMP"#suffix#"8") (v16i8 MQPR:$v1), (v16i8 MQPR:$v2), fc, ARMVCCThen, VCCR:$p1, zero_reg))>;
  def : Pat<(v8i1 (and (v8i1 VCCR:$p1), (v8i1 (ARMvcmp (v8i16 MQPR:$v1), (v8i16 MQPR:$v2), fc)))),
            (v8i1 (!cast<Instruction>("MVE_VCMP"#suffix#"16") (v8i16 MQPR:$v1), (v8i16 MQPR:$v2), fc, ARMVCCThen, VCCR:$p1, zero_reg))>;
  def : Pat<(v4i1 (and (v4i1 VCCR:$p1), (v4i1 (ARMvcmp (v4i32 MQPR:$v1), (v4i32 MQPR:$v2), fc)))),
            (v4i1 (!cast<Instruction>("MVE_VCMP"#suffix#"32") (v4i32 MQPR:$v1), (v4i32 MQPR:$v2), fc, ARMVCCThen, VCCR:$p1, zero_reg))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4393-4399
```tablegen
  def : Pat<(v16i1 (and (v16i1 VCCR:$p1), (v16i1 (ARMvcmp (v16i8 MQPR:$v1), (v16i8 (ARMvdup rGPR:$v2)), fc)))),
            (v16i1 (!cast<Instruction>("MVE_VCMP"#suffix#"8r") (v16i8 MQPR:$v1), (i32 rGPR:$v2), fc, ARMVCCThen, VCCR:$p1, zero_reg))>;
  def : Pat<(v8i1 (and (v8i1 VCCR:$p1), (v8i1 (ARMvcmp (v8i16 MQPR:$v1), (v8i16 (ARMvdup rGPR:$v2)), fc)))),
            (v8i1 (!cast<Instruction>("MVE_VCMP"#suffix#"16r") (v8i16 MQPR:$v1), (i32 rGPR:$v2), fc, ARMVCCThen, VCCR:$p1, zero_reg))>;
  def : Pat<(v4i1 (and (v4i1 VCCR:$p1), (v4i1 (ARMvcmp (v4i32 MQPR:$v1), (v4i32 (ARMvdup rGPR:$v2)), fc)))),
            (v4i1 (!cast<Instruction>("MVE_VCMP"#suffix#"32r") (v4i32 MQPR:$v1), (i32 rGPR:$v2), fc, ARMVCCThen, VCCR:$p1, zero_reg))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4401-4405
```tablegen
multiclass unpred_vcmpf_z<PatLeaf fc> {
  def f16 : Pat<(v8i1 (ARMvcmpz (v8f16 MQPR:$v1), fc)),
                (v8i1 (MVE_VCMPf16r (v8f16 MQPR:$v1), ZR, fc))>;
  def f32 : Pat<(v4i1 (ARMvcmpz (v4f32 MQPR:$v1), fc)),
                (v4i1 (MVE_VCMPf32r (v4f32 MQPR:$v1), ZR, fc))>;
```
- EN: Declares TableGen `multiclass unpred_vcmpf_z`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass unpred_vcmpf_z`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4407-4411
```tablegen
  def : Pat<(v8i1 (and (v8i1 VCCR:$p1), (v8i1 (ARMvcmpz (v8f16 MQPR:$v1), fc)))),
            (v8i1 (MVE_VCMPf16r (v8f16 MQPR:$v1), ZR, fc, ARMVCCThen, VCCR:$p1, zero_reg))>;
  def : Pat<(v4i1 (and (v4i1 VCCR:$p1), (v4i1 (ARMvcmpz (v4f32 MQPR:$v1), fc)))),
            (v4i1 (MVE_VCMPf32r (v4f32 MQPR:$v1), ZR, fc, ARMVCCThen, VCCR:$p1, zero_reg))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4413-4417
```tablegen
multiclass unpred_vcmpf_r<SDPatternOperator cmp, PatLeaf fc> {
  def : Pat<(v8i1 (cmp (v8f16 MQPR:$v1), (v8f16 MQPR:$v2))),
            (v8i1 (MVE_VCMPf16 (v8f16 MQPR:$v1), (v8f16 MQPR:$v2), fc))>;
  def : Pat<(v4i1 (cmp (v4f32 MQPR:$v1), (v4f32 MQPR:$v2))),
            (v4i1 (MVE_VCMPf32 (v4f32 MQPR:$v1), (v4f32 MQPR:$v2), fc))>;
```
- EN: Declares TableGen `multiclass unpred_vcmpf_r`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass unpred_vcmpf_r`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4419-4422
```tablegen
  def : Pat<(v8i1 (cmp (v8f16 MQPR:$v1), (v8f16 (ARMvdup rGPR:$v2)))),
            (v8i1 (MVE_VCMPf16r (v8f16 MQPR:$v1), (i32 rGPR:$v2), fc))>;
  def : Pat<(v4i1 (cmp (v4f32 MQPR:$v1), (v4f32 (ARMvdup rGPR:$v2)))),
            (v4i1 (MVE_VCMPf32r (v4f32 MQPR:$v1), (i32 rGPR:$v2), fc))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4424-4427
```tablegen
  def : Pat<(v8i1 (and (v8i1 VCCR:$p1), (v8i1 (cmp (v8f16 MQPR:$v1), (v8f16 MQPR:$v2))))),
            (v8i1 (MVE_VCMPf16 (v8f16 MQPR:$v1), (v8f16 MQPR:$v2), fc, ARMVCCThen, VCCR:$p1, zero_reg))>;
  def : Pat<(v4i1 (and (v4i1 VCCR:$p1), (v4i1 (cmp (v4f32 MQPR:$v1), (v4f32 MQPR:$v2))))),
            (v4i1 (MVE_VCMPf32 (v4f32 MQPR:$v1), (v4f32 MQPR:$v2), fc, ARMVCCThen, VCCR:$p1, zero_reg))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4429-4433
```tablegen
  def : Pat<(v8i1 (and (v8i1 VCCR:$p1), (v8i1 (cmp (v8f16 MQPR:$v1), (v8f16 (ARMvdup rGPR:$v2)))))),
            (v8i1 (MVE_VCMPf16r (v8f16 MQPR:$v1), (i32 rGPR:$v2), fc, ARMVCCThen, VCCR:$p1, zero_reg))>;
  def : Pat<(v4i1 (and (v4i1 VCCR:$p1), (v4i1 (cmp (v4f32 MQPR:$v1), (v4f32 (ARMvdup rGPR:$v2)))))),
            (v4i1 (MVE_VCMPf32r (v4f32 MQPR:$v1), (i32 rGPR:$v2), fc, ARMVCCThen, VCCR:$p1, zero_reg))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4435-4443
```tablegen
let Predicates = [HasMVEInt] in {
  defm MVE_VCEQZ  : unpred_vcmp_z<"i", ARMCCeq>;
  defm MVE_VCNEZ  : unpred_vcmp_z<"i", ARMCCne>;
  defm MVE_VCGEZ  : unpred_vcmp_z<"s", ARMCCge>;
  defm MVE_VCLTZ  : unpred_vcmp_z<"s", ARMCClt>;
  defm MVE_VCGTZ  : unpred_vcmp_z<"s", ARMCCgt>;
  defm MVE_VCLEZ  : unpred_vcmp_z<"s", ARMCCle>;
  defm MVE_VCGTUZ : unpred_vcmp_z<"u", ARMCChi>;
  defm MVE_VCGEUZ : unpred_vcmp_z<"u", ARMCChs>;
```
- EN: Defines TableGen record `MVE_VCEQZ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VCEQZ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4445-4453
```tablegen
  defm MVE_VCEQ   : unpred_vcmp_r<"i", ARMCCeq>;
  defm MVE_VCNE   : unpred_vcmp_r<"i", ARMCCne>;
  defm MVE_VCGE   : unpred_vcmp_r<"s", ARMCCge>;
  defm MVE_VCLT   : unpred_vcmp_r<"s", ARMCClt>;
  defm MVE_VCGT   : unpred_vcmp_r<"s", ARMCCgt>;
  defm MVE_VCLE   : unpred_vcmp_r<"s", ARMCCle>;
  defm MVE_VCGTU  : unpred_vcmp_r<"u", ARMCChi>;
  defm MVE_VCGEU  : unpred_vcmp_r<"u", ARMCChs>;
}
```
- EN: Defines TableGen record `MVE_VCEQ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VCEQ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4455-4461
```tablegen
let Predicates = [HasMVEFloat] in {
  defm MVE_VFCEQZ  : unpred_vcmpf_z<ARMCCeq>;
  defm MVE_VFCNEZ  : unpred_vcmpf_z<ARMCCne>;
  defm MVE_VFCGEZ  : unpred_vcmpf_z<ARMCCge>;
  defm MVE_VFCLTZ  : unpred_vcmpf_z<ARMCClt>;
  defm MVE_VFCGTZ  : unpred_vcmpf_z<ARMCCgt>;
  defm MVE_VFCLEZ  : unpred_vcmpf_z<ARMCCle>;
```
- EN: Defines TableGen record `MVE_VFCEQZ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VFCEQZ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4463-4469
```tablegen
  defm MVE_VFCEQ   : unpred_vcmpf_r<fcmpeq, ARMCCeq>;
  defm MVE_VFCNE   : unpred_vcmpf_r<fcmpne, ARMCCne>;
  defm MVE_VFCGE   : unpred_vcmpf_r<fcmpge, ARMCCge>;
  defm MVE_VFCLT   : unpred_vcmpf_r<fcmplt, ARMCClt>;
  defm MVE_VFCGT   : unpred_vcmpf_r<fcmpgt, ARMCCgt>;
  defm MVE_VFCLE   : unpred_vcmpf_r<fcmple, ARMCCle>;
}
```
- EN: Defines TableGen record `MVE_VFCEQ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VFCEQ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4472-4489
```tablegen
// Extra "worst case" and/or/xor patterns, going into and out of GRP
multiclass two_predops<SDPatternOperator opnode, Instruction insn> {
  def v16i1 : Pat<(v16i1 (opnode (v16i1 VCCR:$p1), (v16i1 VCCR:$p2))),
                  (v16i1 (COPY_TO_REGCLASS
                           (insn (i32 (COPY_TO_REGCLASS (v16i1 VCCR:$p1), rGPR)),
                                 (i32 (COPY_TO_REGCLASS (v16i1 VCCR:$p2), rGPR))),
                           VCCR))>;
  def v8i1  : Pat<(v8i1 (opnode (v8i1 VCCR:$p1), (v8i1 VCCR:$p2))),
                  (v8i1 (COPY_TO_REGCLASS
                          (insn (i32 (COPY_TO_REGCLASS (v8i1 VCCR:$p1), rGPR)),
                                (i32 (COPY_TO_REGCLASS (v8i1 VCCR:$p2), rGPR))),
                          VCCR))>;
  def v4i1  : Pat<(v4i1 (opnode (v4i1 VCCR:$p1), (v4i1 VCCR:$p2))),
                  (v4i1 (COPY_TO_REGCLASS
                          (insn (i32 (COPY_TO_REGCLASS (v4i1 VCCR:$p1), rGPR)),
                                (i32 (COPY_TO_REGCLASS (v4i1 VCCR:$p2), rGPR))),
                          VCCR))>;
  def v2i1  : Pat<(v2i1 (opnode (v2i1 VCCR:$p1), (v2i1 VCCR:$p2))),
```
- EN: Declares TableGen `multiclass two_predops`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass two_predops`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4490-4494
```tablegen
                  (v2i1 (COPY_TO_REGCLASS
                          (insn (i32 (COPY_TO_REGCLASS (v2i1 VCCR:$p1), rGPR)),
                                (i32 (COPY_TO_REGCLASS (v2i1 VCCR:$p2), rGPR))),
                          VCCR))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4496-4500
```tablegen
let Predicates = [HasMVEInt] in {
  defm POR    : two_predops<or,  t2ORRrr>;
  defm PAND   : two_predops<and, t2ANDrr>;
  defm PEOR   : two_predops<xor, t2EORrr>;
}
```
- EN: Defines TableGen record `POR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `POR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4502-4507
```tablegen
// Predicate cast for MVE i1 types
// Occasionally we need to cast between a i32 and a boolean vector, for
// example when moving between rGPR and VPR.P0 as part of predicate vector
// shuffles. We also sometimes need to cast between different predicate
// vector types (v4i1<>v8i1, etc.) also as part of lowering vector shuffles.
def predicate_cast : SDNode<"ARMISD::PREDICATE_CAST", SDTUnaryOp>;
```
- EN: Defines TableGen record `predicate_cast` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `predicate_cast`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 4509-4511
```tablegen
def load_align4 : PatFrag<(ops node:$ptr), (load node:$ptr), [{
  return cast<LoadSDNode>(N)->getAlign() >= 4;
}]>;
```
- EN: Defines TableGen record `load_align4` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `load_align4`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 4513-4518
```tablegen
let Predicates = [HasMVEInt] in {
  foreach VT = [ v2i1, v4i1, v8i1, v16i1 ] in {
    def : Pat<(i32 (predicate_cast (VT VCCR:$src))),
              (i32 (COPY_TO_REGCLASS (VT VCCR:$src), VCCR))>;
    def : Pat<(VT  (predicate_cast (i32 VCCR:$src))),
              (VT  (COPY_TO_REGCLASS (i32 VCCR:$src), VCCR))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4520-4523
```tablegen
    foreach VT2 = [ v2i1, v4i1, v8i1, v16i1 ] in
      def : Pat<(VT  (predicate_cast (VT2 VCCR:$src))),
                (VT  (COPY_TO_REGCLASS (VT2 VCCR:$src), VCCR))>;
  }
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4525-4530
```tablegen
  // If we happen to be casting from a load we can convert that straight
  // into a predicate load, so long as the load is of the correct type.
  foreach VT = [ v2i1, v4i1, v8i1, v16i1 ] in {
    def : Pat<(VT (predicate_cast (i32 (load_align4 taddrmode_imm7<2>:$addr)))),
              (VT (VLDR_P0_off taddrmode_imm7<2>:$addr))>;
  }
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4532-4536
```tablegen
  // Here we match the specific SDNode type 'ARMVectorRegCastImpl'
  // rather than the more general 'ARMVectorRegCast' which would also
  // match some bitconverts. If we use the latter in cases where the
  // input and output types are the same, the bitconvert gets elided
  // and we end up generating a nonsense match of nothing.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 4538-4542
```tablegen
  foreach VT = [ v16i8, v8i16, v8f16, v4i32, v4f32, v2i64, v2f64 ] in
    foreach VT2 = [ v16i8, v8i16, v8f16, v4i32, v4f32, v2i64, v2f64 ] in
      def : Pat<(VT (ARMVectorRegCastImpl (VT2 MQPR:$src))),
                (VT MQPR:$src)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4544-4544
```tablegen
// end of MVE compares
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 4546-4546
```tablegen
// start of MVE_qDest_qSrc
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 4548-4554
```tablegen
class MVE_qDest_qSrc<string iname, string suffix, dag oops, dag iops,
                     string ops, vpred_ops vpred, string cstr,
                     bits<2> vecsize, list<dag> pattern=[]>
  : MVE_p<oops, iops, NoItinerary, iname, suffix,
          ops, vpred, cstr, vecsize, pattern> {
  bits<4> Qd;
  bits<4> Qm;
```
- EN: Declares reusable TableGen class `MVE_qDest_qSrc` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_qDest_qSrc`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4556-4564
```tablegen
  let Inst{25-23} = 0b100;
  let Inst{22} = Qd{3};
  let Inst{15-13} = Qd{2-0};
  let Inst{11-9} = 0b111;
  let Inst{6} = 0b0;
  let Inst{5} = Qm{3};
  let Inst{4} = 0b0;
  let Inst{3-1} = Qm{2-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4566-4572
```tablegen
class MVE_VQxDMLxDH<string iname, bit exch, bit round, bit subtract,
                    string suffix, bits<2> size, string cstr="",
                    list<dag> pattern=[]>
  : MVE_qDest_qSrc<iname, suffix, (outs MQPR:$Qd),
                   (ins MQPR:$Qd_src, MQPR:$Qn, MQPR:$Qm), "$Qd, $Qn, $Qm",
                   vpred_n, "$Qd = $Qd_src"#cstr, size, pattern> {
  bits<4> Qn;
```
- EN: Declares reusable TableGen class `MVE_VQxDMLxDH` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VQxDMLxDH`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4574-4582
```tablegen
  let Inst{28} = subtract;
  let Inst{21-20} = size;
  let Inst{19-17} = Qn{2-0};
  let Inst{16} = 0b0;
  let Inst{12} = exch;
  let Inst{8} = 0b0;
  let Inst{7} = Qn{3};
  let Inst{0} = round;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4584-4591
```tablegen
multiclass MVE_VQxDMLxDH_p<string iname, bit exch, bit round, bit subtract,
                           MVEVectorVTInfo VTI> {
  def "": MVE_VQxDMLxDH<iname, exch, round, subtract, VTI.Suffix, VTI.Size,
                        !if(!eq(VTI.LaneBits, 32), ",@earlyclobber $Qd", "")>;
  defvar Inst = !cast<Instruction>(NAME);
  defvar ConstParams = (? (i32 exch), (i32 round), (i32 subtract));
  defvar unpred_intr = int_arm_mve_vqdmlad;
  defvar pred_intr = int_arm_mve_vqdmlad_predicated;
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 4593-4603
```tablegen
  def : Pat<(VTI.Vec !con((unpred_intr (VTI.Vec MQPR:$a), (VTI.Vec MQPR:$b),
                                       (VTI.Vec MQPR:$c)), ConstParams)),
            (VTI.Vec (Inst (VTI.Vec MQPR:$a), (VTI.Vec MQPR:$b),
                           (VTI.Vec MQPR:$c)))>;
  def : Pat<(VTI.Vec !con((pred_intr (VTI.Vec MQPR:$a), (VTI.Vec MQPR:$b),
                                     (VTI.Vec MQPR:$c)), ConstParams,
                          (? (VTI.Pred VCCR:$pred)))),
            (VTI.Vec (Inst (VTI.Vec MQPR:$a), (VTI.Vec MQPR:$b),
                           (VTI.Vec MQPR:$c),
                           ARMVCCThen, (VTI.Pred VCCR:$pred), zero_reg))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4605-4610
```tablegen
multiclass MVE_VQxDMLxDH_multi<string iname, bit exch,
                               bit round, bit subtract> {
  defm s8  : MVE_VQxDMLxDH_p<iname, exch, round, subtract, MVE_v16s8>;
  defm s16 : MVE_VQxDMLxDH_p<iname, exch, round, subtract, MVE_v8s16>;
  defm s32 : MVE_VQxDMLxDH_p<iname, exch, round, subtract, MVE_v4s32>;
}
```
- EN: Declares TableGen `multiclass MVE_VQxDMLxDH_multi`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VQxDMLxDH_multi`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4612-4619
```tablegen
defm MVE_VQDMLADH   : MVE_VQxDMLxDH_multi<"vqdmladh",   0b0, 0b0, 0b0>;
defm MVE_VQDMLADHX  : MVE_VQxDMLxDH_multi<"vqdmladhx",  0b1, 0b0, 0b0>;
defm MVE_VQRDMLADH  : MVE_VQxDMLxDH_multi<"vqrdmladh",  0b0, 0b1, 0b0>;
defm MVE_VQRDMLADHX : MVE_VQxDMLxDH_multi<"vqrdmladhx", 0b1, 0b1, 0b0>;
defm MVE_VQDMLSDH   : MVE_VQxDMLxDH_multi<"vqdmlsdh",   0b0, 0b0, 0b1>;
defm MVE_VQDMLSDHX  : MVE_VQxDMLxDH_multi<"vqdmlsdhx",  0b1, 0b0, 0b1>;
defm MVE_VQRDMLSDH  : MVE_VQxDMLxDH_multi<"vqrdmlsdh",  0b0, 0b1, 0b1>;
defm MVE_VQRDMLSDHX : MVE_VQxDMLxDH_multi<"vqrdmlsdhx", 0b1, 0b1, 0b1>;
```
- EN: Defines TableGen record `MVE_VQDMLADH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQDMLADH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4621-4627
```tablegen
class MVE_VCMUL<string iname, string suffix, bits<2> size, string cstr="">
  : MVE_qDest_qSrc<iname, suffix, (outs MQPR:$Qd),
                   (ins MQPR:$Qn, MQPR:$Qm, complexrotateop:$rot),
                   "$Qd, $Qn, $Qm, $rot", vpred_r, cstr, size,
                   []> {
  bits<4> Qn;
  bits<2> rot;
```
- EN: Declares reusable TableGen class `MVE_VCMUL` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VCMUL`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4629-4636
```tablegen
  let Inst{28} = size{1};
  let Inst{21-20} = 0b11;
  let Inst{19-17} = Qn{2-0};
  let Inst{16} = 0b0;
  let Inst{12} = rot{1};
  let Inst{8} = 0b0;
  let Inst{7} = Qn{3};
  let Inst{0} = rot{0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4638-4639
```tablegen
  let Predicates = [HasMVEFloat];
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4641-4644
```tablegen
multiclass MVE_VCMUL_m<string iname, MVEVectorVTInfo VTI,
                       string cstr=""> {
  def "" : MVE_VCMUL<iname, VTI.Suffix, VTI.Size, cstr>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 4646-4650
```tablegen
  let Predicates = [HasMVEFloat] in {
    def : Pat<(VTI.Vec (int_arm_mve_vcmulq
                            imm:$rot, (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm),
                             imm:$rot))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4652-4658
```tablegen
    def : Pat<(VTI.Vec (int_arm_mve_vcmulq_predicated
                            imm:$rot, (VTI.Vec MQPR:$inactive),
                            (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm),
                            (VTI.Pred VCCR:$mask))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm),
                             imm:$rot, ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg,
                             (VTI.Vec MQPR:$inactive)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4660-4661
```tablegen
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4663-4664
```tablegen
defm MVE_VCMULf16 : MVE_VCMUL_m<"vcmul", MVE_v8f16>;
defm MVE_VCMULf32 : MVE_VCMUL_m<"vcmul", MVE_v4f32, "@earlyclobber $Qd">;
```
- EN: Defines TableGen record `MVE_VCMULf16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VCMULf16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4666-4673
```tablegen
class MVE_VMULL<string iname, string suffix, bit bit_28, bits<2> bits_21_20,
                bit T, string cstr, bits<2> vecsize, list<dag> pattern=[]>
  : MVE_qDest_qSrc<iname, suffix, (outs MQPR:$Qd),
                   (ins MQPR:$Qn, MQPR:$Qm), "$Qd, $Qn, $Qm",
                   vpred_r, cstr, vecsize, pattern> {
  bits<4> Qd;
  bits<4> Qn;
  bits<4> Qm;
```
- EN: Declares reusable TableGen class `MVE_VMULL` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VMULL`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4675-4685
```tablegen
  let Inst{28} = bit_28;
  let Inst{21-20} = bits_21_20;
  let Inst{19-17} = Qn{2-0};
  let Inst{16} = 0b1;
  let Inst{12} = T;
  let Inst{8} = 0b0;
  let Inst{7} = Qn{3};
  let Inst{0} = 0b0;
  let validForTailPredication = 1;
  let doubleWidthResult = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4687-4692
```tablegen
multiclass MVE_VMULL_m<MVEVectorVTInfo VTI,
                       SDPatternOperator unpred_op, Intrinsic pred_int,
                       bit Top, bits<2> vecsize, string cstr=""> {
  def "" : MVE_VMULL<"vmull" # !if(Top, "t", "b"), VTI.Suffix, VTI.Unsigned,
                     VTI.Size, Top, cstr, vecsize>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 4694-4695
```tablegen
  let Predicates = [HasMVEInt] in {
    defvar uflag = !if(!eq(VTI.SuffixLetter, "p"), (?), (? (i32 VTI.Unsigned)));
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 4697-4701
```tablegen
    // Unpredicated multiply
    def : Pat<(VTI.DblVec !con((unpred_op (VTI.Vec MQPR:$Qm),
                                          (VTI.Vec MQPR:$Qn)),
                               uflag, (? (i32 Top)))),
              (VTI.DblVec (Inst (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4703-4712
```tablegen
    // Predicated multiply
    def : Pat<(VTI.DblVec !con((pred_int (VTI.Vec MQPR:$Qm),
                                         (VTI.Vec MQPR:$Qn)),
                               uflag, (? (i32 Top), (VTI.DblPred VCCR:$mask),
                                         (VTI.DblVec MQPR:$inactive)))),
              (VTI.DblVec (Inst (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn),
                                ARMVCCThen, (VTI.DblPred VCCR:$mask), zero_reg,
                                (VTI.DblVec MQPR:$inactive)))>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4714-4715
```tablegen
// For polynomial multiplies, the size bits take the unused value 0b11, and
// the unsigned bit switches to encoding the size.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 4717-4730
```tablegen
defm MVE_VMULLBs8  : MVE_VMULL_m<MVE_v16s8, int_arm_mve_vmull,
                                 int_arm_mve_mull_int_predicated, 0b0, 0b01>;
defm MVE_VMULLTs8  : MVE_VMULL_m<MVE_v16s8, int_arm_mve_vmull,
                                 int_arm_mve_mull_int_predicated, 0b1, 0b01>;
defm MVE_VMULLBs16 : MVE_VMULL_m<MVE_v8s16, int_arm_mve_vmull,
                                 int_arm_mve_mull_int_predicated, 0b0, 0b10>;
defm MVE_VMULLTs16 : MVE_VMULL_m<MVE_v8s16, int_arm_mve_vmull,
                                 int_arm_mve_mull_int_predicated, 0b1, 0b10>;
defm MVE_VMULLBs32 : MVE_VMULL_m<MVE_v4s32, int_arm_mve_vmull,
                                 int_arm_mve_mull_int_predicated, 0b0, 0b11,
                                 "@earlyclobber $Qd">;
defm MVE_VMULLTs32 : MVE_VMULL_m<MVE_v4s32, int_arm_mve_vmull,
                                 int_arm_mve_mull_int_predicated, 0b1, 0b11,
                                 "@earlyclobber $Qd">;
```
- EN: Defines TableGen record `MVE_VMULLBs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMULLBs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4732-4745
```tablegen
defm MVE_VMULLBu8  : MVE_VMULL_m<MVE_v16u8, int_arm_mve_vmull,
                                 int_arm_mve_mull_int_predicated, 0b0, 0b01>;
defm MVE_VMULLTu8  : MVE_VMULL_m<MVE_v16u8, int_arm_mve_vmull,
                                 int_arm_mve_mull_int_predicated, 0b1, 0b01>;
defm MVE_VMULLBu16 : MVE_VMULL_m<MVE_v8u16, int_arm_mve_vmull,
                                 int_arm_mve_mull_int_predicated, 0b0, 0b10>;
defm MVE_VMULLTu16 : MVE_VMULL_m<MVE_v8u16, int_arm_mve_vmull,
                                 int_arm_mve_mull_int_predicated, 0b1, 0b10>;
defm MVE_VMULLBu32 : MVE_VMULL_m<MVE_v4u32, int_arm_mve_vmull,
                                 int_arm_mve_mull_int_predicated, 0b0, 0b11,
                                 "@earlyclobber $Qd">;
defm MVE_VMULLTu32 : MVE_VMULL_m<MVE_v4u32, int_arm_mve_vmull,
                                 int_arm_mve_mull_int_predicated, 0b1, 0b11,
                                 "@earlyclobber $Qd">;
```
- EN: Defines TableGen record `MVE_VMULLBu8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMULLBu8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4747-4754
```tablegen
defm MVE_VMULLBp8  : MVE_VMULL_m<MVE_v16p8, int_arm_mve_vmull_poly,
                                 int_arm_mve_mull_poly_predicated, 0b0, 0b01>;
defm MVE_VMULLTp8  : MVE_VMULL_m<MVE_v16p8, int_arm_mve_vmull_poly,
                                 int_arm_mve_mull_poly_predicated, 0b1, 0b01>;
defm MVE_VMULLBp16 : MVE_VMULL_m<MVE_v8p16, int_arm_mve_vmull_poly,
                                 int_arm_mve_mull_poly_predicated, 0b0, 0b10>;
defm MVE_VMULLTp16 : MVE_VMULL_m<MVE_v8p16, int_arm_mve_vmull_poly,
                                 int_arm_mve_mull_poly_predicated, 0b1, 0b10>;
```
- EN: Defines TableGen record `MVE_VMULLBp8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMULLBp8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4756-4761
```tablegen
let Predicates = [HasMVEInt] in {
  def : Pat<(v2i64 (ARMvmulls (v4i32 MQPR:$src1), (v4i32 MQPR:$src2))),
            (MVE_VMULLBs32 MQPR:$src1, MQPR:$src2)>;
  def : Pat<(v2i64 (ARMvmulls (v4i32 (ARMvrev64 (v4i32 MQPR:$src1))),
                              (v4i32 (ARMvrev64 (v4i32 MQPR:$src2))))),
            (MVE_VMULLTs32 MQPR:$src1, MQPR:$src2)>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4763-4768
```tablegen
  def : Pat<(mul (sext_inreg (v4i32 MQPR:$src1), v4i16),
                 (sext_inreg (v4i32 MQPR:$src2), v4i16)),
            (MVE_VMULLBs16 MQPR:$src1, MQPR:$src2)>;
  def : Pat<(mul (sext_inreg (v4i32 (ARMVectorRegCast (ARMvrev32 (v8i16 MQPR:$src1)))), v4i16),
                 (sext_inreg (v4i32 (ARMVectorRegCast (ARMvrev32 (v8i16 MQPR:$src2)))), v4i16)),
            (MVE_VMULLTs16 MQPR:$src1, MQPR:$src2)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4770-4775
```tablegen
  def : Pat<(mul (sext_inreg (v8i16 MQPR:$src1), v8i8),
                 (sext_inreg (v8i16 MQPR:$src2), v8i8)),
            (MVE_VMULLBs8 MQPR:$src1, MQPR:$src2)>;
  def : Pat<(mul (sext_inreg (v8i16 (ARMVectorRegCast (ARMvrev16 (v16i8 MQPR:$src1)))), v8i8),
                 (sext_inreg (v8i16 (ARMVectorRegCast (ARMvrev16 (v16i8 MQPR:$src2)))), v8i8)),
            (MVE_VMULLTs8 MQPR:$src1, MQPR:$src2)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4777-4781
```tablegen
  def : Pat<(v2i64 (ARMvmullu (v4i32 MQPR:$src1), (v4i32 MQPR:$src2))),
            (MVE_VMULLBu32 MQPR:$src1, MQPR:$src2)>;
  def : Pat<(v2i64 (ARMvmullu (v4i32 (ARMvrev64 (v4i32 MQPR:$src1))),
                              (v4i32 (ARMvrev64 (v4i32 MQPR:$src2))))),
            (MVE_VMULLTu32 MQPR:$src1, MQPR:$src2)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4783-4790
```tablegen
  def : Pat<(mul (and (v4i32 MQPR:$src1), (v4i32 (ARMvmovImm (i32 0xCFF)))),
                 (and (v4i32 MQPR:$src2), (v4i32 (ARMvmovImm (i32 0xCFF))))),
            (MVE_VMULLBu16 MQPR:$src1, MQPR:$src2)>;
  def : Pat<(mul (and (v4i32 (ARMVectorRegCast (ARMvrev32 (v8i16 MQPR:$src1)))),
                      (v4i32 (ARMvmovImm (i32 0xCFF)))),
                 (and (v4i32 (ARMVectorRegCast (ARMvrev32 (v8i16 MQPR:$src2)))),
                      (v4i32 (ARMvmovImm (i32 0xCFF))))),
            (MVE_VMULLTu16 MQPR:$src1, MQPR:$src2)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4792-4798
```tablegen
  def : Pat<(mul (ARMvbicImm (v8i16 MQPR:$src1), (i32 0xAFF)),
                 (ARMvbicImm (v8i16 MQPR:$src2), (i32 0xAFF))),
            (MVE_VMULLBu8 MQPR:$src1, MQPR:$src2)>;
  def : Pat<(mul (ARMvbicImm (v8i16 (ARMVectorRegCast (ARMvrev16 (v16i8 MQPR:$src1)))), (i32 0xAFF)),
                 (ARMvbicImm (v8i16 (ARMVectorRegCast (ARMvrev16 (v16i8 MQPR:$src2)))), (i32 0xAFF))),
            (MVE_VMULLTu8 MQPR:$src1, MQPR:$src2)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4800-4805
```tablegen
class MVE_VxMULH<string iname, string suffix, bit U, bits<2> size, bit round,
                 list<dag> pattern=[]>
  : MVE_qDest_qSrc<iname, suffix, (outs MQPR:$Qd),
                   (ins MQPR:$Qn, MQPR:$Qm), "$Qd, $Qn, $Qm",
                   vpred_r, "", size, pattern> {
  bits<4> Qn;
```
- EN: Declares reusable TableGen class `MVE_VxMULH` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VxMULH`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4807-4816
```tablegen
  let Inst{28} = U;
  let Inst{21-20} = size;
  let Inst{19-17} = Qn{2-0};
  let Inst{16} = 0b1;
  let Inst{12} = round;
  let Inst{8} = 0b0;
  let Inst{7} = Qn{3};
  let Inst{0} = 0b1;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4818-4821
```tablegen
multiclass MVE_VxMULH_m<string iname, MVEVectorVTInfo VTI, SDPatternOperator unpred_op,
                        Intrinsic PredInt, bit round> {
  def "" : MVE_VxMULH<iname, VTI.Suffix, VTI.Unsigned, VTI.Size, round>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 4823-4836
```tablegen
  let Predicates = [HasMVEInt] in {
    if !eq(round, 0b0) then {
      defvar mulh = !if(VTI.Unsigned, mulhu, mulhs);
      defm : MVE_TwoOpPattern<VTI, mulh, PredInt, (? (i32 VTI.Unsigned)),
                              !cast<Instruction>(NAME)>;
    } else {
      // Predicated multiply returning high bits
      def : Pat<(VTI.Vec (PredInt (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn),
                              (i32 VTI.Unsigned), (VTI.Pred VCCR:$mask),
                              (VTI.Vec MQPR:$inactive))),
                (VTI.Vec (Inst (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn),
                              ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg,
                              (VTI.Vec MQPR:$inactive)))>;
    }
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 4838-4843
```tablegen
    // Unpredicated intrinsic
    def : Pat<(VTI.Vec (unpred_op (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn),
                            (i32 VTI.Unsigned))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn)))>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4845-4849
```tablegen
multiclass MVE_VMULT<string iname, MVEVectorVTInfo VTI, bit round>
  : MVE_VxMULH_m<iname, VTI, !if(round, int_arm_mve_vrmulh, int_arm_mve_vmulh),
                 !if(round, int_arm_mve_rmulh_predicated,
                            int_arm_mve_mulh_predicated),
                 round>;
```
- EN: Declares TableGen `multiclass MVE_VMULT`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMULT`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4851-4856
```tablegen
defm MVE_VMULHs8   : MVE_VMULT<"vmulh",  MVE_v16s8, 0b0>;
defm MVE_VMULHs16  : MVE_VMULT<"vmulh",  MVE_v8s16, 0b0>;
defm MVE_VMULHs32  : MVE_VMULT<"vmulh",  MVE_v4s32, 0b0>;
defm MVE_VMULHu8   : MVE_VMULT<"vmulh",  MVE_v16u8, 0b0>;
defm MVE_VMULHu16  : MVE_VMULT<"vmulh",  MVE_v8u16, 0b0>;
defm MVE_VMULHu32  : MVE_VMULT<"vmulh",  MVE_v4u32, 0b0>;
```
- EN: Defines TableGen record `MVE_VMULHs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMULHs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4858-4863
```tablegen
defm MVE_VRMULHs8  : MVE_VMULT<"vrmulh", MVE_v16s8, 0b1>;
defm MVE_VRMULHs16 : MVE_VMULT<"vrmulh", MVE_v8s16, 0b1>;
defm MVE_VRMULHs32 : MVE_VMULT<"vrmulh", MVE_v4s32, 0b1>;
defm MVE_VRMULHu8  : MVE_VMULT<"vrmulh", MVE_v16u8, 0b1>;
defm MVE_VRMULHu16 : MVE_VMULT<"vrmulh", MVE_v8u16, 0b1>;
defm MVE_VRMULHu32 : MVE_VMULT<"vrmulh", MVE_v4u32, 0b1>;
```
- EN: Defines TableGen record `MVE_VRMULHs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VRMULHs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4865-4869
```tablegen
class MVE_VxMOVxN<string iname, string suffix, bit bit_28, bit bit_17,
                  bits<2> size, bit T, list<dag> pattern=[]>
  : MVE_qDest_qSrc<iname, suffix, (outs MQPR:$Qd),
                   (ins MQPR:$Qd_src, MQPR:$Qm), "$Qd, $Qm",
                   vpred_n, "$Qd = $Qd_src", !if(size, 0b10, 0b01), pattern> {
```
- EN: Declares reusable TableGen class `MVE_VxMOVxN` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VxMOVxN`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4871-4882
```tablegen
  let Inst{28} = bit_28;
  let Inst{21-20} = 0b11;
  let Inst{19-18} = size;
  let Inst{17} = bit_17;
  let Inst{16} = 0b1;
  let Inst{12} = T;
  let Inst{8} = 0b0;
  let Inst{7} = !not(bit_17);
  let Inst{0} = 0b1;
  let validForTailPredication = 1;
  let retainsPreviousHalfElement = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4884-4888
```tablegen
multiclass MVE_VxMOVxN_halves<string iname, string suffix,
                              bit bit_28, bit bit_17, bits<2> size> {
  def bh : MVE_VxMOVxN<iname # "b", suffix, bit_28, bit_17, size, 0b0>;
  def th : MVE_VxMOVxN<iname # "t", suffix, bit_28, bit_17, size, 0b1>;
}
```
- EN: Declares TableGen `multiclass MVE_VxMOVxN_halves`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VxMOVxN_halves`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4890-4897
```tablegen
defm MVE_VMOVNi16   : MVE_VxMOVxN_halves<"vmovn",   "i16", 0b1, 0b0, 0b00>;
defm MVE_VMOVNi32   : MVE_VxMOVxN_halves<"vmovn",   "i32", 0b1, 0b0, 0b01>;
defm MVE_VQMOVNs16  : MVE_VxMOVxN_halves<"vqmovn",  "s16", 0b0, 0b1, 0b00>;
defm MVE_VQMOVNs32  : MVE_VxMOVxN_halves<"vqmovn",  "s32", 0b0, 0b1, 0b01>;
defm MVE_VQMOVNu16  : MVE_VxMOVxN_halves<"vqmovn",  "u16", 0b1, 0b1, 0b00>;
defm MVE_VQMOVNu32  : MVE_VxMOVxN_halves<"vqmovn",  "u32", 0b1, 0b1, 0b01>;
defm MVE_VQMOVUNs16 : MVE_VxMOVxN_halves<"vqmovun", "s16", 0b0, 0b0, 0b00>;
defm MVE_VQMOVUNs32 : MVE_VxMOVxN_halves<"vqmovun", "s32", 0b0, 0b0, 0b01>;
```
- EN: Defines TableGen record `MVE_VMOVNi16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMOVNi16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4899-4900
```tablegen
// MVE vmovn
def MVEvmovn       : SDNode<"ARMISD::VMOVN", SDTARMVEXT>;
```
- EN: Defines TableGen record `MVEvmovn` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MVEvmovn`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 4902-4908
```tablegen
multiclass MVE_VMOVN_p<Instruction Inst, bit top,
                       MVEVectorVTInfo VTI, MVEVectorVTInfo InVTI> {
  // Match the most obvious MVEvmovn(a,b,t), which overwrites the odd or even
  // lanes of a (depending on t) with the even lanes of b.
  def : Pat<(VTI.Vec (MVEvmovn (VTI.Vec MQPR:$Qd_src),
                               (VTI.Vec MQPR:$Qm), (i32 top))),
            (VTI.Vec (Inst (VTI.Vec MQPR:$Qd_src), (VTI.Vec MQPR:$Qm)))>;
```
- EN: Declares TableGen `multiclass MVE_VMOVN_p`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMOVN_p`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4910-4920
```tablegen
  if !not(top) then {
    // If we see MVEvmovn(a,ARMvrev(b),1), that wants to overwrite the odd
    // lanes of a with the odd lanes of b. In other words, the lanes we're
    // _keeping_ from a are the even ones. So we can flip it round and say that
    // this is the same as overwriting the even lanes of b with the even lanes
    // of a, i.e. it's a VMOVNB with the operands reversed.
    defvar vrev = !cast<SDNode>("ARMvrev" # InVTI.LaneBits);
    def : Pat<(VTI.Vec (MVEvmovn (VTI.Vec MQPR:$Qm),
                                 (VTI.Vec (vrev MQPR:$Qd_src)), (i32 1))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qd_src), (VTI.Vec MQPR:$Qm)))>;
  }
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 4922-4931
```tablegen
  // Match the IR intrinsic for a predicated VMOVN. This regards the Qm input
  // as having wider lanes that we're narrowing, instead of already-narrow
  // lanes that we're taking every other one of.
  def : Pat<(VTI.Vec (int_arm_mve_vmovn_predicated (VTI.Vec MQPR:$Qd_src),
                                  (InVTI.Vec MQPR:$Qm), (i32 top),
                                  (InVTI.Pred VCCR:$pred))),
            (VTI.Vec (Inst (VTI.Vec MQPR:$Qd_src),
                              (InVTI.Vec MQPR:$Qm),
                              ARMVCCThen, (InVTI.Pred VCCR:$pred), zero_reg))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4933-4936
```tablegen
defm : MVE_VMOVN_p<MVE_VMOVNi32bh, 0, MVE_v8i16, MVE_v4i32>;
defm : MVE_VMOVN_p<MVE_VMOVNi32th, 1, MVE_v8i16, MVE_v4i32>;
defm : MVE_VMOVN_p<MVE_VMOVNi16bh, 0, MVE_v16i8, MVE_v8i16>;
defm : MVE_VMOVN_p<MVE_VMOVNi16th, 1, MVE_v16i8, MVE_v8i16>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4938-4944
```tablegen
multiclass MVE_VQMOVN_p<Instruction Inst, bit outU, bit inU, bit top,
                        MVEVectorVTInfo VTI, MVEVectorVTInfo InVTI> {
  def : Pat<(VTI.Vec (int_arm_mve_vqmovn (VTI.Vec MQPR:$Qd_src),
                                  (InVTI.Vec MQPR:$Qm),
                                  (i32 outU), (i32 inU), (i32 top))),
            (VTI.Vec (Inst (VTI.Vec MQPR:$Qd_src),
                              (InVTI.Vec MQPR:$Qm)))>;
```
- EN: Declares TableGen `multiclass MVE_VQMOVN_p`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VQMOVN_p`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4946-4953
```tablegen
  def : Pat<(VTI.Vec (int_arm_mve_vqmovn_predicated (VTI.Vec MQPR:$Qd_src),
                                  (InVTI.Vec MQPR:$Qm),
                                  (i32 outU), (i32 inU), (i32 top),
                                  (InVTI.Pred VCCR:$pred))),
            (VTI.Vec (Inst (VTI.Vec MQPR:$Qd_src),
                              (InVTI.Vec MQPR:$Qm),
                              ARMVCCThen, (InVTI.Pred VCCR:$pred), zero_reg))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4955-4966
```tablegen
defm : MVE_VQMOVN_p<MVE_VQMOVNs32bh,  0, 0, 0, MVE_v8i16, MVE_v4i32>;
defm : MVE_VQMOVN_p<MVE_VQMOVNs32th,  0, 0, 1, MVE_v8i16, MVE_v4i32>;
defm : MVE_VQMOVN_p<MVE_VQMOVNs16bh,  0, 0, 0, MVE_v16i8, MVE_v8i16>;
defm : MVE_VQMOVN_p<MVE_VQMOVNs16th,  0, 0, 1, MVE_v16i8, MVE_v8i16>;
defm : MVE_VQMOVN_p<MVE_VQMOVNu32bh,  1, 1, 0, MVE_v8i16, MVE_v4i32>;
defm : MVE_VQMOVN_p<MVE_VQMOVNu32th,  1, 1, 1, MVE_v8i16, MVE_v4i32>;
defm : MVE_VQMOVN_p<MVE_VQMOVNu16bh,  1, 1, 0, MVE_v16i8, MVE_v8i16>;
defm : MVE_VQMOVN_p<MVE_VQMOVNu16th,  1, 1, 1, MVE_v16i8, MVE_v8i16>;
defm : MVE_VQMOVN_p<MVE_VQMOVUNs32bh, 1, 0, 0, MVE_v8i16, MVE_v4i32>;
defm : MVE_VQMOVN_p<MVE_VQMOVUNs32th, 1, 0, 1, MVE_v8i16, MVE_v4i32>;
defm : MVE_VQMOVN_p<MVE_VQMOVUNs16bh, 1, 0, 0, MVE_v16i8, MVE_v8i16>;
defm : MVE_VQMOVN_p<MVE_VQMOVUNs16th, 1, 0, 1, MVE_v16i8, MVE_v8i16>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4968-4969
```tablegen
def SDTARMVMOVNQ : SDTypeProfile<1, 3, [SDTCisVec<0>, SDTCisSameAs<0, 1>,
                                        SDTCisVec<2>, SDTCisVT<3, i32>]>;
```
- EN: Defines TableGen record `SDTARMVMOVNQ` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTARMVMOVNQ`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 4971-4972
```tablegen
// Vector (V) Saturating (Q) Move and Narrow (N), signed (s)
def MVEvqmovns   : SDNode<"ARMISD::VQMOVNs", SDTARMVMOVNQ>;
```
- EN: Defines TableGen record `MVEvqmovns` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MVEvqmovns`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 4974-4975
```tablegen
// Vector (V) Saturating (Q) Move and Narrow (N), unsigned (u)
def MVEvqmovnu   : SDNode<"ARMISD::VQMOVNu", SDTARMVMOVNQ>;
```
- EN: Defines TableGen record `MVEvqmovnu` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MVEvqmovnu`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 4977-4985
```tablegen
let Predicates = [HasMVEInt] in {
  def : Pat<(v8i16 (MVEvqmovns (v8i16 MQPR:$Qd_src), (v4i32 MQPR:$Qm), (i32 0))),
            (v8i16 (MVE_VQMOVNs32bh (v8i16 MQPR:$Qd_src), (v4i32 MQPR:$Qm)))>;
  def : Pat<(v8i16 (MVEvqmovns (v8i16 MQPR:$Qd_src), (v4i32 MQPR:$Qm), (i32 1))),
            (v8i16 (MVE_VQMOVNs32th (v8i16 MQPR:$Qd_src), (v4i32 MQPR:$Qm)))>;
  def : Pat<(v16i8 (MVEvqmovns (v16i8 MQPR:$Qd_src), (v8i16 MQPR:$Qm), (i32 0))),
            (v16i8 (MVE_VQMOVNs16bh (v16i8 MQPR:$Qd_src), (v8i16 MQPR:$Qm)))>;
  def : Pat<(v16i8 (MVEvqmovns (v16i8 MQPR:$Qd_src), (v8i16 MQPR:$Qm), (i32 1))),
            (v16i8 (MVE_VQMOVNs16th (v16i8 MQPR:$Qd_src), (v8i16 MQPR:$Qm)))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4987-4994
```tablegen
  def : Pat<(v8i16 (MVEvqmovnu (v8i16 MQPR:$Qd_src), (v4i32 MQPR:$Qm), (i32 0))),
            (v8i16 (MVE_VQMOVNu32bh (v8i16 MQPR:$Qd_src), (v4i32 MQPR:$Qm)))>;
  def : Pat<(v8i16 (MVEvqmovnu (v8i16 MQPR:$Qd_src), (v4i32 MQPR:$Qm), (i32 1))),
            (v8i16 (MVE_VQMOVNu32th (v8i16 MQPR:$Qd_src), (v4i32 MQPR:$Qm)))>;
  def : Pat<(v16i8 (MVEvqmovnu (v16i8 MQPR:$Qd_src), (v8i16 MQPR:$Qm), (i32 0))),
            (v16i8 (MVE_VQMOVNu16bh (v16i8 MQPR:$Qd_src), (v8i16 MQPR:$Qm)))>;
  def : Pat<(v16i8 (MVEvqmovnu (v16i8 MQPR:$Qd_src), (v8i16 MQPR:$Qm), (i32 1))),
            (v16i8 (MVE_VQMOVNu16th (v16i8 MQPR:$Qd_src), (v8i16 MQPR:$Qm)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4996-5003
```tablegen
  def : Pat<(v8i16 (MVEvqmovns (v8i16 MQPR:$Qd_src), (v4i32 (ARMvshrsImm (v4i32 MQPR:$Qm), imm0_31:$imm)), (i32 0))),
            (v8i16 (MVE_VQSHRNbhs32 (v8i16 MQPR:$Qd_src), (v4i32 MQPR:$Qm), imm0_31:$imm))>;
  def : Pat<(v16i8 (MVEvqmovns (v16i8 MQPR:$Qd_src), (v8i16 (ARMvshrsImm (v8i16 MQPR:$Qm), imm0_15:$imm)), (i32 0))),
            (v16i8 (MVE_VQSHRNbhs16 (v16i8 MQPR:$Qd_src), (v8i16 MQPR:$Qm), imm0_15:$imm))>;
  def : Pat<(v8i16 (MVEvqmovns (v8i16 MQPR:$Qd_src), (v4i32 (ARMvshrsImm (v4i32 MQPR:$Qm), imm0_31:$imm)), (i32 1))),
            (v8i16 (MVE_VQSHRNths32 (v8i16 MQPR:$Qd_src), (v4i32 MQPR:$Qm), imm0_31:$imm))>;
  def : Pat<(v16i8 (MVEvqmovns (v16i8 MQPR:$Qd_src), (v8i16 (ARMvshrsImm (v8i16 MQPR:$Qm), imm0_15:$imm)), (i32 1))),
            (v16i8 (MVE_VQSHRNths16 (v16i8 MQPR:$Qd_src), (v8i16 MQPR:$Qm), imm0_15:$imm))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5005-5013
```tablegen
  def : Pat<(v8i16 (MVEvqmovnu (v8i16 MQPR:$Qd_src), (v4i32 (ARMvshruImm (v4i32 MQPR:$Qm), imm0_31:$imm)), (i32 0))),
            (v8i16 (MVE_VQSHRNbhu32 (v8i16 MQPR:$Qd_src), (v4i32 MQPR:$Qm), imm0_31:$imm))>;
  def : Pat<(v16i8 (MVEvqmovnu (v16i8 MQPR:$Qd_src), (v8i16 (ARMvshruImm (v8i16 MQPR:$Qm), imm0_15:$imm)), (i32 0))),
            (v16i8 (MVE_VQSHRNbhu16 (v16i8 MQPR:$Qd_src), (v8i16 MQPR:$Qm), imm0_15:$imm))>;
  def : Pat<(v8i16 (MVEvqmovnu (v8i16 MQPR:$Qd_src), (v4i32 (ARMvshruImm (v4i32 MQPR:$Qm), imm0_31:$imm)), (i32 1))),
            (v8i16 (MVE_VQSHRNthu32 (v8i16 MQPR:$Qd_src), (v4i32 MQPR:$Qm), imm0_31:$imm))>;
  def : Pat<(v16i8 (MVEvqmovnu (v16i8 MQPR:$Qd_src), (v8i16 (ARMvshruImm (v8i16 MQPR:$Qm), imm0_15:$imm)), (i32 1))),
            (v16i8 (MVE_VQSHRNthu16 (v16i8 MQPR:$Qd_src), (v8i16 MQPR:$Qm), imm0_15:$imm))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5015-5024
```tablegen
class MVE_VCVT_ff<string iname, string suffix, bit op, bit T,
                  dag iops_extra, vpred_ops vpred, string cstr>
  : MVE_qDest_qSrc<iname, suffix, (outs MQPR:$Qd),
                   !con(iops_extra, (ins MQPR:$Qm)), "$Qd, $Qm",
                   vpred, cstr, 0b10, []> {
  let Inst{28} = op;
  let Inst{21-16} = 0b111111;
  let Inst{12} = T;
  let Inst{8-7} = 0b00;
  let Inst{0} = 0b1;
```
- EN: Declares reusable TableGen class `MVE_VCVT_ff` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VCVT_ff`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5026-5028
```tablegen
  let Predicates = [HasMVEFloat];
  let retainsPreviousHalfElement = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5030-5031
```tablegen
def SDTARMVCVTL    : SDTypeProfile<1, 2, [SDTCisVec<0>, SDTCisVec<1>,
                                         SDTCisVT<2, i32>]>;
```
- EN: Defines TableGen record `SDTARMVCVTL` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTARMVCVTL`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 5033-5034
```tablegen
// MVE vcvt f32 -> f16, truncating into either the bottom or top lanes
def MVEvcvtn       : SDNode<"ARMISD::VCVTN", SDTARMVMOVNQ>;
```
- EN: Defines TableGen record `MVEvcvtn` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MVEvcvtn`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 5036-5037
```tablegen
// MVE vcvt f16 -> f32, extending from either the bottom or top lanes
def MVEvcvtl       : SDNode<"ARMISD::VCVTL", SDTARMVCVTL>;
```
- EN: Defines TableGen record `MVEvcvtl` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MVEvcvtl`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 5039-5042
```tablegen
multiclass MVE_VCVT_f2h_m<string iname, int half> {
  def "": MVE_VCVT_ff<iname, "f16.f32", 0b0, half,
                      (ins MQPR:$Qd_src), vpred_n, "$Qd = $Qd_src">;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 5044-5052
```tablegen
  let Predicates = [HasMVEFloat] in {
    def : Pat<(v8f16 (int_arm_mve_vcvt_narrow
                         (v8f16 MQPR:$Qd_src), (v4f32 MQPR:$Qm), (i32 half))),
              (v8f16 (Inst (v8f16 MQPR:$Qd_src), (v4f32 MQPR:$Qm)))>;
    def : Pat<(v8f16 (int_arm_mve_vcvt_narrow_predicated
                         (v8f16 MQPR:$Qd_src), (v4f32 MQPR:$Qm), (i32 half),
                         (v4i1 VCCR:$mask))),
              (v8f16 (Inst (v8f16 MQPR:$Qd_src), (v4f32 MQPR:$Qm),
                           ARMVCCThen, (v4i1 VCCR:$mask), zero_reg))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5054-5057
```tablegen
    def : Pat<(v8f16 (MVEvcvtn (v8f16 MQPR:$Qd_src), (v4f32 MQPR:$Qm), (i32 half))),
              (v8f16 (Inst (v8f16 MQPR:$Qd_src), (v4f32 MQPR:$Qm)))>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5059-5061
```tablegen
multiclass MVE_VCVT_h2f_m<string iname, int half> {
  def "": MVE_VCVT_ff<iname, "f32.f16", 0b1, half, (ins), vpred_r, "">;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 5063-5070
```tablegen
  let Predicates = [HasMVEFloat] in {
    def : Pat<(v4f32 (int_arm_mve_vcvt_widen (v8f16 MQPR:$Qm), (i32 half))),
              (v4f32 (Inst (v8f16 MQPR:$Qm)))>;
    def : Pat<(v4f32 (int_arm_mve_vcvt_widen_predicated
                         (v4f32 MQPR:$inactive), (v8f16 MQPR:$Qm), (i32 half),
                         (v4i1 VCCR:$mask))),
              (v4f32 (Inst (v8f16 MQPR:$Qm), ARMVCCThen,
                           (v4i1 VCCR:$mask), zero_reg, (v4f32 MQPR:$inactive)))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5072-5075
```tablegen
    def : Pat<(v4f32 (MVEvcvtl (v8f16 MQPR:$Qm), (i32 half))),
              (v4f32 (Inst (v8f16 MQPR:$Qm)))>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5077-5080
```tablegen
defm MVE_VCVTf16f32bh : MVE_VCVT_f2h_m<"vcvtb", 0b0>;
defm MVE_VCVTf16f32th : MVE_VCVT_f2h_m<"vcvtt", 0b1>;
defm MVE_VCVTf32f16bh : MVE_VCVT_h2f_m<"vcvtb", 0b0>;
defm MVE_VCVTf32f16th : MVE_VCVT_h2f_m<"vcvtt", 0b1>;
```
- EN: Defines TableGen record `MVE_VCVTf16f32bh` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VCVTf16f32bh`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5082-5088
```tablegen
class MVE_VxCADD<string iname, string suffix, bits<2> size, bit halve,
                 string cstr="">
  : MVE_qDest_qSrc<iname, suffix, (outs MQPR:$Qd),
                   (ins MQPR:$Qn, MQPR:$Qm, complexrotateopodd:$rot),
                   "$Qd, $Qn, $Qm, $rot", vpred_r, cstr, size, []> {
  bits<4> Qn;
  bit rot;
```
- EN: Declares reusable TableGen class `MVE_VxCADD` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VxCADD`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5090-5098
```tablegen
  let Inst{28} = halve;
  let Inst{21-20} = size;
  let Inst{19-17} = Qn{2-0};
  let Inst{16} = 0b0;
  let Inst{12} = rot;
  let Inst{8} = 0b1;
  let Inst{7} = Qn{3};
  let Inst{0} = 0b0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5100-5103
```tablegen
multiclass MVE_VxCADD_m<string iname, MVEVectorVTInfo VTI,
                        bit halve, string cstr=""> {
  def "" : MVE_VxCADD<iname, VTI.Suffix, VTI.Size, halve, cstr>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 5105-5109
```tablegen
  let Predicates = [HasMVEInt] in {
    def : Pat<(VTI.Vec (int_arm_mve_vcaddq halve,
                            imm:$rot, (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm),
                             imm:$rot))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5111-5117
```tablegen
    def : Pat<(VTI.Vec (int_arm_mve_vcaddq_predicated halve,
                            imm:$rot, (VTI.Vec MQPR:$inactive),
                            (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm),
                            (VTI.Pred VCCR:$mask))),
              (VTI.Vec (Inst (VTI.Vec MQPR:$Qn), (VTI.Vec MQPR:$Qm),
                             imm:$rot, ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg,
                             (VTI.Vec MQPR:$inactive)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5119-5120
```tablegen
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5122-5124
```tablegen
defm MVE_VCADDi8   : MVE_VxCADD_m<"vcadd", MVE_v16i8, 0b1>;
defm MVE_VCADDi16  : MVE_VxCADD_m<"vcadd", MVE_v8i16, 0b1>;
defm MVE_VCADDi32  : MVE_VxCADD_m<"vcadd", MVE_v4i32, 0b1, "@earlyclobber $Qd">;
```
- EN: Defines TableGen record `MVE_VCADDi8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VCADDi8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5126-5128
```tablegen
defm MVE_VHCADDs8  : MVE_VxCADD_m<"vhcadd", MVE_v16s8, 0b0>;
defm MVE_VHCADDs16 : MVE_VxCADD_m<"vhcadd", MVE_v8s16, 0b0>;
defm MVE_VHCADDs32 : MVE_VxCADD_m<"vhcadd", MVE_v4s32, 0b0, "@earlyclobber $Qd">;
```
- EN: Defines TableGen record `MVE_VHCADDs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VHCADDs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5130-5135
```tablegen
class MVE_VADCSBC<string iname, bit I, bit subtract,
                  dag carryin, list<dag> pattern=[]>
  : MVE_qDest_qSrc<iname, "i32", (outs MQPR:$Qd, cl_FPSCR_NZCV:$carryout),
                   !con((ins MQPR:$Qn, MQPR:$Qm), carryin),
                   "$Qd, $Qn, $Qm", vpred_r, "", 0b10, pattern> {
  bits<4> Qn;
```
- EN: Declares reusable TableGen class `MVE_VADCSBC` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VADCSBC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5137-5144
```tablegen
  let Inst{28} = subtract;
  let Inst{21-20} = 0b11;
  let Inst{19-17} = Qn{2-0};
  let Inst{16} = 0b0;
  let Inst{12} = I;
  let Inst{8} = 0b1;
  let Inst{7} = Qn{3};
  let Inst{0} = 0b0;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5146-5149
```tablegen
  // Custom decoder method in order to add the FPSCR operand(s), which
  // Tablegen won't do right
  let DecoderMethod = "DecodeMVEVADCInstruction";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5151-5152
```tablegen
def MVE_VADC  : MVE_VADCSBC<"vadc",  0b0, 0b0, (ins cl_FPSCR_NZCV:$carryin)>;
def MVE_VADCI : MVE_VADCSBC<"vadci", 0b1, 0b0, (ins)>;
```
- EN: Defines TableGen record `MVE_VADC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VADC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5154-5155
```tablegen
def MVE_VSBC  : MVE_VADCSBC<"vsbc",  0b0, 0b1, (ins cl_FPSCR_NZCV:$carryin)>;
def MVE_VSBCI : MVE_VADCSBC<"vsbci", 0b1, 0b1, (ins)>;
```
- EN: Defines TableGen record `MVE_VSBC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSBC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5157-5162
```tablegen
class MVE_VQDMULL<string iname, string suffix, bit size, bit T,
                  string cstr="", list<dag> pattern=[]>
  : MVE_qDest_qSrc<iname, suffix, (outs MQPR:$Qd),
                   (ins MQPR:$Qn, MQPR:$Qm), "$Qd, $Qn, $Qm",
                   vpred_r, cstr, !if(size, 0b10, 0b01), pattern> {
  bits<4> Qn;
```
- EN: Declares reusable TableGen class `MVE_VQDMULL` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VQDMULL`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5164-5174
```tablegen
  let Inst{28} = size;
  let Inst{21-20} = 0b11;
  let Inst{19-17} = Qn{2-0};
  let Inst{16} = 0b0;
  let Inst{12} = T;
  let Inst{8} = 0b1;
  let Inst{7} = Qn{3};
  let Inst{0} = 0b1;
  let validForTailPredication = 1;
  let doubleWidthResult = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5176-5179
```tablegen
multiclass MVE_VQDMULL_m<string iname, MVEVectorVTInfo VTI, bit size, bit T,
                         string cstr> {
  def "" : MVE_VQDMULL<iname, VTI.Suffix, size, T, cstr>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 5181-5195
```tablegen
  let Predicates = [HasMVEInt] in {
    // Unpredicated saturating multiply
    def : Pat<(VTI.DblVec (int_arm_mve_vqdmull (VTI.Vec MQPR:$Qm),
                                               (VTI.Vec MQPR:$Qn), (i32 T))),
              (VTI.DblVec (Inst (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn)))>;
    // Predicated saturating multiply
    def : Pat<(VTI.DblVec (int_arm_mve_vqdmull_predicated
                                    (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn),
                                    (i32 T), (VTI.DblPred VCCR:$mask),
                                    (VTI.DblVec MQPR:$inactive))),
              (VTI.DblVec (Inst (VTI.Vec MQPR:$Qm), (VTI.Vec MQPR:$Qn),
                                ARMVCCThen, (VTI.DblPred VCCR:$mask), zero_reg,
                                (VTI.DblVec MQPR:$inactive)))>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5197-5200
```tablegen
multiclass MVE_VQDMULL_halves<MVEVectorVTInfo VTI, bit size, string cstr=""> {
  defm bh : MVE_VQDMULL_m<"vqdmullb", VTI, size, 0b0, cstr>;
  defm th : MVE_VQDMULL_m<"vqdmullt", VTI, size, 0b1, cstr>;
}
```
- EN: Declares TableGen `multiclass MVE_VQDMULL_halves`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VQDMULL_halves`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5202-5203
```tablegen
defm MVE_VQDMULLs16 : MVE_VQDMULL_halves<MVE_v8s16, 0b0>;
defm MVE_VQDMULLs32 : MVE_VQDMULL_halves<MVE_v4s32, 0b1, "@earlyclobber $Qd">;
```
- EN: Defines TableGen record `MVE_VQDMULLs16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQDMULLs16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5205-5205
```tablegen
// end of mve_qDest_qSrc
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 5207-5207
```tablegen
// start of mve_qDest_rSrc
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 5209-5214
```tablegen
class MVE_qr_base<dag oops, dag iops, string iname, string suffix, string ops,
                  vpred_ops vpred, string cstr, bits<2> vecsize, list<dag> pattern=[]>
   : MVE_p<oops, iops, NoItinerary, iname, suffix, ops, vpred, cstr, vecsize, pattern> {
  bits<4> Qd;
  bits<4> Qn;
  bits<4> Rm;
```
- EN: Declares reusable TableGen class `MVE_qr_base` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_qr_base`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5216-5225
```tablegen
  let Inst{25-23} = 0b100;
  let Inst{22} = Qd{3};
  let Inst{19-17} = Qn{2-0};
  let Inst{15-13} = Qd{2-0};
  let Inst{11-9} = 0b111;
  let Inst{7} = Qn{3};
  let Inst{6} = 0b1;
  let Inst{4} = 0b0;
  let Inst{3-0} = Rm{3-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5227-5230
```tablegen
class MVE_qDest_rSrc<string iname, string suffix, string cstr="", bits<2> vecsize, list<dag> pattern=[]>
  : MVE_qr_base<(outs MQPR:$Qd), (ins MQPR:$Qn, rGPR:$Rm),
                 iname, suffix, "$Qd, $Qn, $Rm", vpred_r, cstr,
                 vecsize, pattern>;
```
- EN: Declares reusable TableGen class `MVE_qDest_rSrc` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_qDest_rSrc`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5232-5235
```tablegen
class MVE_qDestSrc_rSrc<string iname, string suffix, bits<2> vecsize, list<dag> pattern=[]>
  : MVE_qr_base<(outs MQPR:$Qd), (ins MQPR:$Qd_src, MQPR:$Qn, rGPR:$Rm),
                 iname, suffix, "$Qd, $Qn, $Rm", vpred_n, "$Qd = $Qd_src",
                 vecsize, pattern>;
```
- EN: Declares reusable TableGen class `MVE_qDestSrc_rSrc` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_qDestSrc_rSrc`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5237-5241
```tablegen
class MVE_qDest_single_rSrc<string iname, string suffix, bits<2> vecsize, list<dag> pattern=[]>
  : MVE_p<(outs MQPR:$Qd), (ins MQPR:$Qd_src, rGPR:$Rm), NoItinerary, iname,
          suffix, "$Qd, $Rm", vpred_n, "$Qd = $Qd_src", vecsize, pattern> {
  bits<4> Qd;
  bits<4> Rm;
```
- EN: Declares reusable TableGen class `MVE_qDest_single_rSrc` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_qDest_single_rSrc`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5243-5246
```tablegen
  let Inst{22} = Qd{3};
  let Inst{15-13} = Qd{2-0};
  let Inst{3-0} = Rm{3-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5248-5255
```tablegen
// Patterns for vector-scalar instructions with integer operands
multiclass MVE_vec_scalar_int_pat_m<Instruction inst, MVEVectorVTInfo VTI,
                                    SDPatternOperator unpred_op,
                                    SDPatternOperator pred_op,
                                    bit unpred_has_sign = 0,
                                    bit pred_has_sign = 0> {
  defvar UnpredSign = !if(unpred_has_sign, (? (i32 VTI.Unsigned)), (?));
  defvar PredSign = !if(pred_has_sign, (? (i32 VTI.Unsigned)), (?));
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 5257-5273
```tablegen
  let Predicates = [HasMVEInt] in {
    // Unpredicated version
    def : Pat<(VTI.Vec !con((unpred_op (VTI.Vec MQPR:$Qm),
                                       (VTI.Vec (ARMvdup rGPR:$val))),
                            UnpredSign)),
              (VTI.Vec (inst (VTI.Vec MQPR:$Qm), (i32 rGPR:$val)))>;
    // Predicated version
    def : Pat<(VTI.Vec !con((pred_op (VTI.Vec MQPR:$Qm),
                                     (VTI.Vec (ARMvdup rGPR:$val))),
                            PredSign,
                            (pred_op (VTI.Pred VCCR:$mask),
                                     (VTI.Vec MQPR:$inactive)))),
              (VTI.Vec (inst (VTI.Vec MQPR:$Qm), (i32 rGPR:$val),
                             ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg,
                             (VTI.Vec MQPR:$inactive)))>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5275-5277
```tablegen
class MVE_VADDSUB_qr<string iname, string suffix, bits<2> size,
                     bit bit_5, bit bit_12, bit bit_16, bit bit_28>
  : MVE_qDest_rSrc<iname, suffix, "", size> {
```
- EN: Declares reusable TableGen class `MVE_VADDSUB_qr` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VADDSUB_qr`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5279-5286
```tablegen
  let Inst{28} = bit_28;
  let Inst{21-20} = size;
  let Inst{16} = bit_16;
  let Inst{12} = bit_12;
  let Inst{8} = 0b1;
  let Inst{5} = bit_5;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5288-5295
```tablegen
// Vector-scalar add/sub
multiclass MVE_VADDSUB_qr_m<string iname, MVEVectorVTInfo VTI, bit subtract,
                            SDNode Op, Intrinsic PredInt> {
  def "" : MVE_VADDSUB_qr<iname, VTI.Suffix, VTI.Size, 0b0, subtract, 0b1, 0b0>;
  let Predicates = [HasMVEInt] in {
    defm : MVE_TwoOpPatternDup<VTI, Op, PredInt, (? ), !cast<Instruction>(NAME), ARMimmAllZerosV>;
  }
}
```
- EN: Declares TableGen `multiclass MVE_VADDSUB_qr_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VADDSUB_qr_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5297-5298
```tablegen
multiclass MVE_VADD_qr_m<MVEVectorVTInfo VTI>
  : MVE_VADDSUB_qr_m<"vadd", VTI, 0b0, add, int_arm_mve_add_predicated>;
```
- EN: Declares TableGen `multiclass MVE_VADD_qr_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VADD_qr_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5300-5301
```tablegen
multiclass MVE_VSUB_qr_m<MVEVectorVTInfo VTI>
  : MVE_VADDSUB_qr_m<"vsub", VTI, 0b1, sub, int_arm_mve_sub_predicated>;
```
- EN: Declares TableGen `multiclass MVE_VSUB_qr_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VSUB_qr_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5303-5305
```tablegen
defm MVE_VADD_qr_i8  : MVE_VADD_qr_m<MVE_v16i8>;
defm MVE_VADD_qr_i16 : MVE_VADD_qr_m<MVE_v8i16>;
defm MVE_VADD_qr_i32 : MVE_VADD_qr_m<MVE_v4i32>;
```
- EN: Defines TableGen record `MVE_VADD_qr_i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VADD_qr_i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5307-5309
```tablegen
defm MVE_VSUB_qr_i8  : MVE_VSUB_qr_m<MVE_v16i8>;
defm MVE_VSUB_qr_i16 : MVE_VSUB_qr_m<MVE_v8i16>;
defm MVE_VSUB_qr_i32 : MVE_VSUB_qr_m<MVE_v4i32>;
```
- EN: Defines TableGen record `MVE_VSUB_qr_i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSUB_qr_i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5311-5315
```tablegen
// Vector-scalar saturating add/sub
multiclass MVE_VQADDSUB_qr_m<string iname, MVEVectorVTInfo VTI, bit subtract,
                             SDNode Op, Intrinsic PredInt> {
  def "" : MVE_VADDSUB_qr<iname, VTI.Suffix, VTI.Size, 0b1, subtract,
                          0b0, VTI.Unsigned>;
```
- EN: Declares TableGen `multiclass MVE_VQADDSUB_qr_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VQADDSUB_qr_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5317-5321
```tablegen
  let Predicates = [HasMVEInt] in {
    defm : MVE_TwoOpPatternDup<VTI, Op, PredInt, (? (i32 VTI.Unsigned)),
                               !cast<Instruction>(NAME)>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5323-5324
```tablegen
multiclass MVE_VQADD_qr_m<MVEVectorVTInfo VTI, SDNode Op>
  : MVE_VQADDSUB_qr_m<"vqadd", VTI, 0b0, Op, int_arm_mve_qadd_predicated>;
```
- EN: Declares TableGen `multiclass MVE_VQADD_qr_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VQADD_qr_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5326-5327
```tablegen
multiclass MVE_VQSUB_qr_m<MVEVectorVTInfo VTI, SDNode Op>
  : MVE_VQADDSUB_qr_m<"vqsub", VTI, 0b1, Op, int_arm_mve_qsub_predicated>;
```
- EN: Declares TableGen `multiclass MVE_VQSUB_qr_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VQSUB_qr_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5329-5334
```tablegen
defm MVE_VQADD_qr_s8  : MVE_VQADD_qr_m<MVE_v16s8, saddsat>;
defm MVE_VQADD_qr_s16 : MVE_VQADD_qr_m<MVE_v8s16, saddsat>;
defm MVE_VQADD_qr_s32 : MVE_VQADD_qr_m<MVE_v4s32, saddsat>;
defm MVE_VQADD_qr_u8  : MVE_VQADD_qr_m<MVE_v16u8, uaddsat>;
defm MVE_VQADD_qr_u16 : MVE_VQADD_qr_m<MVE_v8u16, uaddsat>;
defm MVE_VQADD_qr_u32 : MVE_VQADD_qr_m<MVE_v4u32, uaddsat>;
```
- EN: Defines TableGen record `MVE_VQADD_qr_s8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQADD_qr_s8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5336-5341
```tablegen
defm MVE_VQSUB_qr_s8  : MVE_VQSUB_qr_m<MVE_v16s8, ssubsat>;
defm MVE_VQSUB_qr_s16 : MVE_VQSUB_qr_m<MVE_v8s16, ssubsat>;
defm MVE_VQSUB_qr_s32 : MVE_VQSUB_qr_m<MVE_v4s32, ssubsat>;
defm MVE_VQSUB_qr_u8  : MVE_VQSUB_qr_m<MVE_v16u8, usubsat>;
defm MVE_VQSUB_qr_u16 : MVE_VQSUB_qr_m<MVE_v8u16, usubsat>;
defm MVE_VQSUB_qr_u32 : MVE_VQSUB_qr_m<MVE_v4u32, usubsat>;
```
- EN: Defines TableGen record `MVE_VQSUB_qr_s8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQSUB_qr_s8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5343-5345
```tablegen
class MVE_VQDMULL_qr<string iname, string suffix, bit size,
                     bit T, string cstr="", list<dag> pattern=[]>
  : MVE_qDest_rSrc<iname, suffix, cstr, !if(size, 0b10, 0b01), pattern> {
```
- EN: Declares reusable TableGen class `MVE_VQDMULL_qr` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VQDMULL_qr`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5347-5355
```tablegen
  let Inst{28} = size;
  let Inst{21-20} = 0b11;
  let Inst{16} = 0b0;
  let Inst{12} = T;
  let Inst{8} = 0b1;
  let Inst{5} = 0b1;
  let validForTailPredication = 1;
  let doubleWidthResult = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5357-5360
```tablegen
multiclass MVE_VQDMULL_qr_m<string iname, MVEVectorVTInfo VTI, bit size,
                            bit T, string cstr> {
  def "" : MVE_VQDMULL_qr<iname, VTI.Suffix, size, T, cstr>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 5362-5379
```tablegen
  let Predicates = [HasMVEInt] in {
    // Unpredicated saturating multiply
    def : Pat<(VTI.DblVec (int_arm_mve_vqdmull (VTI.Vec MQPR:$Qm),
                                               (VTI.Vec (ARMvdup rGPR:$val)),
                                               (i32 T))),
              (VTI.DblVec (Inst (VTI.Vec MQPR:$Qm), (i32 rGPR:$val)))>;
    // Predicated saturating multiply
    def : Pat<(VTI.DblVec (int_arm_mve_vqdmull_predicated
                                    (VTI.Vec MQPR:$Qm),
                                    (VTI.Vec (ARMvdup rGPR:$val)),
                                    (i32 T),
                                    (VTI.DblPred VCCR:$mask),
                                    (VTI.DblVec MQPR:$inactive))),
              (VTI.DblVec (Inst (VTI.Vec MQPR:$Qm), (i32 rGPR:$val),
                             ARMVCCThen, (VTI.DblPred VCCR:$mask), zero_reg,
                             (VTI.DblVec MQPR:$inactive)))>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5381-5384
```tablegen
multiclass MVE_VQDMULL_qr_halves<MVEVectorVTInfo VTI, bit size, string cstr=""> {
  defm bh : MVE_VQDMULL_qr_m<"vqdmullb", VTI, size, 0b0, cstr>;
  defm th : MVE_VQDMULL_qr_m<"vqdmullt", VTI, size, 0b1, cstr>;
}
```
- EN: Declares TableGen `multiclass MVE_VQDMULL_qr_halves`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VQDMULL_qr_halves`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5386-5387
```tablegen
defm MVE_VQDMULL_qr_s16 : MVE_VQDMULL_qr_halves<MVE_v8s16, 0b0>;
defm MVE_VQDMULL_qr_s32 : MVE_VQDMULL_qr_halves<MVE_v4s32, 0b1, "@earlyclobber $Qd">;
```
- EN: Defines TableGen record `MVE_VQDMULL_qr_s16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQDMULL_qr_s16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5389-5392
```tablegen
class MVE_VxADDSUB_qr<string iname, string suffix,
                      bit bit_28, bits<2> size, bit subtract,
                      bits<2> vecsize, list<dag> pattern=[]>
  : MVE_qDest_rSrc<iname, suffix, "", vecsize, pattern> {
```
- EN: Declares reusable TableGen class `MVE_VxADDSUB_qr` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VxADDSUB_qr`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5394-5401
```tablegen
  let Inst{28} = bit_28;
  let Inst{21-20} = size;
  let Inst{16} = 0b0;
  let Inst{12} = subtract;
  let Inst{8} = 0b1;
  let Inst{5} = 0b0;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5403-5409
```tablegen
multiclass MVE_VHADDSUB_qr_m<string iname, MVEVectorVTInfo VTI, bit subtract, SDPatternOperator Op,
                             Intrinsic unpred_int, Intrinsic pred_int, PatFrag add_op, SDNode shift_op> {
  def "" : MVE_VxADDSUB_qr<iname, VTI.Suffix, VTI.Unsigned, VTI.Size, subtract, VTI.Size>;
  defm : MVE_TwoOpPatternDup<VTI, Op, pred_int, (? (i32 VTI.Unsigned)), !cast<Instruction>(NAME)>;
  defm : MVE_vec_scalar_int_pat_m<!cast<Instruction>(NAME),
                                  VTI, unpred_int, pred_int, 1, 1>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 5411-5415
```tablegen
  let Predicates = [HasMVEInt] in {
    def : Pat<(VTI.Vec (shift_op (add_op (VTI.Vec MQPR:$Qm), (VTI.Vec (ARMvdup rGPR:$Rn))), (i32 1))),
              (Inst MQPR:$Qm, rGPR:$Rn)>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5417-5419
```tablegen
multiclass MVE_VHADD_qr_m<MVEVectorVTInfo VTI, PatFrag add_op, SDNode shift_op, SDNode Op> :
  MVE_VHADDSUB_qr_m<"vhadd", VTI, 0b0, Op, int_arm_mve_vhadd,
                    int_arm_mve_hadd_predicated, add_op, shift_op>;
```
- EN: Declares TableGen `multiclass MVE_VHADD_qr_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VHADD_qr_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5421-5423
```tablegen
multiclass MVE_VHSUB_qr_m<MVEVectorVTInfo VTI, PatFrag add_op, SDNode shift_op> :
  MVE_VHADDSUB_qr_m<"vhsub", VTI, 0b1, null_frag, int_arm_mve_vhsub,
                    int_arm_mve_hsub_predicated, add_op, shift_op>;
```
- EN: Declares TableGen `multiclass MVE_VHSUB_qr_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VHSUB_qr_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5425-5430
```tablegen
defm MVE_VHADD_qr_s8  : MVE_VHADD_qr_m<MVE_v16s8, addnsw, ARMvshrsImm, avgfloors>;
defm MVE_VHADD_qr_s16 : MVE_VHADD_qr_m<MVE_v8s16, addnsw, ARMvshrsImm, avgfloors>;
defm MVE_VHADD_qr_s32 : MVE_VHADD_qr_m<MVE_v4s32, addnsw, ARMvshrsImm, avgfloors>;
defm MVE_VHADD_qr_u8  : MVE_VHADD_qr_m<MVE_v16u8, addnuw, ARMvshruImm, avgflooru>;
defm MVE_VHADD_qr_u16 : MVE_VHADD_qr_m<MVE_v8u16, addnuw, ARMvshruImm, avgflooru>;
defm MVE_VHADD_qr_u32 : MVE_VHADD_qr_m<MVE_v4u32, addnuw, ARMvshruImm, avgflooru>;
```
- EN: Defines TableGen record `MVE_VHADD_qr_s8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VHADD_qr_s8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5432-5437
```tablegen
defm MVE_VHSUB_qr_s8  : MVE_VHSUB_qr_m<MVE_v16s8, subnsw, ARMvshrsImm>;
defm MVE_VHSUB_qr_s16 : MVE_VHSUB_qr_m<MVE_v8s16, subnsw, ARMvshrsImm>;
defm MVE_VHSUB_qr_s32 : MVE_VHSUB_qr_m<MVE_v4s32, subnsw, ARMvshrsImm>;
defm MVE_VHSUB_qr_u8  : MVE_VHSUB_qr_m<MVE_v16u8, subnuw, ARMvshruImm>;
defm MVE_VHSUB_qr_u16 : MVE_VHSUB_qr_m<MVE_v8u16, subnuw, ARMvshruImm>;
defm MVE_VHSUB_qr_u32 : MVE_VHSUB_qr_m<MVE_v4u32, subnuw, ARMvshruImm>;
```
- EN: Defines TableGen record `MVE_VHSUB_qr_s8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VHSUB_qr_s8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5439-5445
```tablegen
multiclass MVE_VADDSUB_qr_f<string iname, MVEVectorVTInfo VTI, bit subtract,
                            SDPatternOperator Op, Intrinsic PredInt,
                            SDPatternOperator IdentityVec> {
  def "" : MVE_VxADDSUB_qr<iname, VTI.Suffix, VTI.Size{0}, 0b11, subtract, VTI.Size>;
  defm : MVE_TwoOpPatternDup<VTI, Op, PredInt, (? ),
                              !cast<Instruction>(NAME), IdentityVec>;
}
```
- EN: Declares TableGen `multiclass MVE_VADDSUB_qr_f`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VADDSUB_qr_f`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5447-5451
```tablegen
let Predicates = [HasMVEFloat] in {
  defm MVE_VADD_qr_f32 : MVE_VADDSUB_qr_f<"vadd", MVE_v4f32, 0b0, vadd,
                                          int_arm_mve_add_predicated, ARMimmMinusZeroF>;
  defm MVE_VADD_qr_f16 : MVE_VADDSUB_qr_f<"vadd", MVE_v8f16, 0b0, vadd,
                                          int_arm_mve_add_predicated, ARMimmMinusZeroH>;
```
- EN: Defines TableGen record `MVE_VADD_qr_f32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VADD_qr_f32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5453-5457
```tablegen
  defm MVE_VSUB_qr_f32 : MVE_VADDSUB_qr_f<"vsub", MVE_v4f32, 0b1, vsub,
                                          int_arm_mve_sub_predicated, ARMimmAllZerosV>;
  defm MVE_VSUB_qr_f16 : MVE_VADDSUB_qr_f<"vsub", MVE_v8f16, 0b1, vsub,
                                          int_arm_mve_sub_predicated, ARMimmAllZerosV>;
}
```
- EN: Defines TableGen record `MVE_VSUB_qr_f32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSUB_qr_f32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5459-5461
```tablegen
class MVE_VxSHL_qr<string iname, string suffix, bit U, bits<2> size,
                   bit bit_7, bit bit_17, list<dag> pattern=[]>
  : MVE_qDest_single_rSrc<iname, suffix, size, pattern> {
```
- EN: Declares reusable TableGen class `MVE_VxSHL_qr` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VxSHL_qr`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5463-5473
```tablegen
  let Inst{28} = U;
  let Inst{25-23} = 0b100;
  let Inst{21-20} = 0b11;
  let Inst{19-18} = size;
  let Inst{17} = bit_17;
  let Inst{16} = 0b1;
  let Inst{12-8} = 0b11110;
  let Inst{7} = bit_7;
  let Inst{6-4} = 0b110;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5475-5477
```tablegen
multiclass MVE_VxSHL_qr_p<string iname, MVEVectorVTInfo VTI, bit q, bit r> {
  def "" : MVE_VxSHL_qr<iname, VTI.Suffix, VTI.Unsigned, VTI.Size, q, r>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 5479-5482
```tablegen
  def : Pat<(VTI.Vec (int_arm_mve_vshl_scalar
                         (VTI.Vec MQPR:$in), (i32 rGPR:$sh),
                         (i32 q), (i32 r), (i32 VTI.Unsigned))),
            (VTI.Vec (Inst (VTI.Vec MQPR:$in), (i32 rGPR:$sh)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5484-5490
```tablegen
  def : Pat<(VTI.Vec (int_arm_mve_vshl_scalar_predicated
                         (VTI.Vec MQPR:$in), (i32 rGPR:$sh),
                         (i32 q), (i32 r), (i32 VTI.Unsigned),
                         (VTI.Pred VCCR:$mask))),
            (VTI.Vec (Inst (VTI.Vec MQPR:$in), (i32 rGPR:$sh),
                           ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5492-5499
```tablegen
multiclass MVE_VxSHL_qr_types<string iname, bit bit_7, bit bit_17> {
  defm s8  : MVE_VxSHL_qr_p<iname, MVE_v16s8, bit_7, bit_17>;
  defm s16 : MVE_VxSHL_qr_p<iname, MVE_v8s16, bit_7, bit_17>;
  defm s32 : MVE_VxSHL_qr_p<iname, MVE_v4s32, bit_7, bit_17>;
  defm u8  : MVE_VxSHL_qr_p<iname, MVE_v16u8, bit_7, bit_17>;
  defm u16 : MVE_VxSHL_qr_p<iname, MVE_v8u16, bit_7, bit_17>;
  defm u32 : MVE_VxSHL_qr_p<iname, MVE_v4u32, bit_7, bit_17>;
}
```
- EN: Declares TableGen `multiclass MVE_VxSHL_qr_types`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VxSHL_qr_types`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5501-5504
```tablegen
defm MVE_VSHL_qr   : MVE_VxSHL_qr_types<"vshl",   0b0, 0b0>;
defm MVE_VRSHL_qr  : MVE_VxSHL_qr_types<"vrshl",  0b0, 0b1>;
defm MVE_VQSHL_qr  : MVE_VxSHL_qr_types<"vqshl",  0b1, 0b0>;
defm MVE_VQRSHL_qr : MVE_VxSHL_qr_types<"vqrshl", 0b1, 0b1>;
```
- EN: Defines TableGen record `MVE_VSHL_qr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSHL_qr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5506-5512
```tablegen
let Predicates = [HasMVEInt] in {
  def : Pat<(v4i32 (ARMvshlu (v4i32 MQPR:$Qm), (v4i32 (ARMvdup rGPR:$Rm)))),
            (v4i32 (MVE_VSHL_qru32 (v4i32 MQPR:$Qm), rGPR:$Rm))>;
  def : Pat<(v8i16 (ARMvshlu (v8i16 MQPR:$Qm), (v8i16 (ARMvdup rGPR:$Rm)))),
            (v8i16 (MVE_VSHL_qru16 (v8i16 MQPR:$Qm), rGPR:$Rm))>;
  def : Pat<(v16i8 (ARMvshlu (v16i8 MQPR:$Qm), (v16i8 (ARMvdup rGPR:$Rm)))),
            (v16i8 (MVE_VSHL_qru8 (v16i8 MQPR:$Qm), rGPR:$Rm))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5514-5520
```tablegen
  def : Pat<(v4i32 (ARMvshls (v4i32 MQPR:$Qm), (v4i32 (ARMvdup rGPR:$Rm)))),
            (v4i32 (MVE_VSHL_qrs32 (v4i32 MQPR:$Qm), rGPR:$Rm))>;
  def : Pat<(v8i16 (ARMvshls (v8i16 MQPR:$Qm), (v8i16 (ARMvdup rGPR:$Rm)))),
            (v8i16 (MVE_VSHL_qrs16 (v8i16 MQPR:$Qm), rGPR:$Rm))>;
  def : Pat<(v16i8 (ARMvshls (v16i8 MQPR:$Qm), (v16i8 (ARMvdup rGPR:$Rm)))),
            (v16i8 (MVE_VSHL_qrs8 (v16i8 MQPR:$Qm), rGPR:$Rm))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5522-5523
```tablegen
class MVE_VBRSR<string iname, string suffix, bits<2> size, list<dag> pattern=[]>
  : MVE_qDest_rSrc<iname, suffix, "", size, pattern> {
```
- EN: Declares reusable TableGen class `MVE_VBRSR` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VBRSR`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5525-5532
```tablegen
  let Inst{28} = 0b1;
  let Inst{21-20} = size;
  let Inst{16} = 0b1;
  let Inst{12} = 0b1;
  let Inst{8} = 0b0;
  let Inst{5} = 0b1;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5534-5536
```tablegen
def MVE_VBRSR8  : MVE_VBRSR<"vbrsr", "8", 0b00>;
def MVE_VBRSR16 : MVE_VBRSR<"vbrsr", "16", 0b01>;
def MVE_VBRSR32 : MVE_VBRSR<"vbrsr", "32", 0b10>;
```
- EN: Defines TableGen record `MVE_VBRSR8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VBRSR8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5538-5550
```tablegen
multiclass MVE_VBRSR_pat_m<MVEVectorVTInfo VTI, Instruction Inst> {
  // Unpredicated
  def : Pat<(VTI.Vec (int_arm_mve_vbrsr (VTI.Vec MQPR:$Qn), (i32 rGPR:$Rm))),
            (VTI.Vec (Inst (VTI.Vec MQPR:$Qn), (i32 rGPR:$Rm)))>;
  // Predicated
  def : Pat<(VTI.Vec (int_arm_mve_vbrsr_predicated
                          (VTI.Vec MQPR:$inactive),
                          (VTI.Vec MQPR:$Qn), (i32 rGPR:$Rm),
                          (VTI.Pred VCCR:$mask))),
            (VTI.Vec (Inst (VTI.Vec MQPR:$Qn), (i32 rGPR:$Rm),
                          ARMVCCThen, (VTI.Pred VCCR:$mask), zero_reg,
                          (VTI.Vec MQPR:$inactive)))>;
}
```
- EN: Declares TableGen `multiclass MVE_VBRSR_pat_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VBRSR_pat_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5552-5554
```tablegen
let Predicates = [HasMVEInt] in {
  def : Pat<(v16i8 ( bitreverse (v16i8 MQPR:$val1))),
            (v16i8 ( MVE_VBRSR8 (v16i8 MQPR:$val1), (t2MOVi (i32 8)) ))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5556-5557
```tablegen
  def : Pat<(v4i32 ( bitreverse (v4i32 MQPR:$val1))),
            (v4i32 ( MVE_VBRSR32 (v4i32 MQPR:$val1), (t2MOVi (i32 32)) ))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5559-5560
```tablegen
  def : Pat<(v8i16 ( bitreverse (v8i16 MQPR:$val1))),
            (v8i16 ( MVE_VBRSR16 (v8i16 MQPR:$val1), (t2MOVi (i32 16)) ))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5562-5565
```tablegen
  defm : MVE_VBRSR_pat_m<MVE_v16i8, MVE_VBRSR8>;
  defm : MVE_VBRSR_pat_m<MVE_v8i16, MVE_VBRSR16>;
  defm : MVE_VBRSR_pat_m<MVE_v4i32, MVE_VBRSR32>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5567-5570
```tablegen
let Predicates = [HasMVEFloat] in {
  defm : MVE_VBRSR_pat_m<MVE_v8f16, MVE_VBRSR16>;
  defm : MVE_VBRSR_pat_m<MVE_v4f32, MVE_VBRSR32>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5572-5573
```tablegen
class MVE_VMUL_qr_int<string iname, string suffix, bits<2> size>
  : MVE_qDest_rSrc<iname, suffix, "", size> {
```
- EN: Declares reusable TableGen class `MVE_VMUL_qr_int` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VMUL_qr_int`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5575-5582
```tablegen
  let Inst{28} = 0b0;
  let Inst{21-20} = size;
  let Inst{16} = 0b1;
  let Inst{12} = 0b1;
  let Inst{8} = 0b0;
  let Inst{5} = 0b1;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5584-5590
```tablegen
multiclass MVE_VMUL_qr_int_m<MVEVectorVTInfo VTI> {
  def "" : MVE_VMUL_qr_int<"vmul", VTI.Suffix, VTI.Size>;
  let Predicates = [HasMVEInt] in {
    defm : MVE_TwoOpPatternDup<VTI, mul, int_arm_mve_mul_predicated, (? ),
                               !cast<Instruction>(NAME), ARMimmOneV>;
  }
}
```
- EN: Declares TableGen `multiclass MVE_VMUL_qr_int_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VMUL_qr_int_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5592-5594
```tablegen
defm MVE_VMUL_qr_i8  : MVE_VMUL_qr_int_m<MVE_v16i8>;
defm MVE_VMUL_qr_i16 : MVE_VMUL_qr_int_m<MVE_v8i16>;
defm MVE_VMUL_qr_i32 : MVE_VMUL_qr_int_m<MVE_v4i32>;
```
- EN: Defines TableGen record `MVE_VMUL_qr_i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMUL_qr_i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5596-5598
```tablegen
class MVE_VxxMUL_qr<string iname, string suffix,
                    bit bit_28, bits<2> size, bits<2> vecsize, list<dag> pattern=[]>
  : MVE_qDest_rSrc<iname, suffix, "", vecsize, pattern> {
```
- EN: Declares reusable TableGen class `MVE_VxxMUL_qr` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VxxMUL_qr`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5600-5607
```tablegen
  let Inst{28} = bit_28;
  let Inst{21-20} = size;
  let Inst{16} = 0b1;
  let Inst{12} = 0b0;
  let Inst{8} = 0b0;
  let Inst{5} = 0b1;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5609-5611
```tablegen
multiclass MVE_VxxMUL_qr_m<string iname, MVEVectorVTInfo VTI, bit bit_28,
                           SDPatternOperator Op, Intrinsic int_unpred, Intrinsic int_pred> {
  def "" : MVE_VxxMUL_qr<iname, VTI.Suffix, bit_28, VTI.Size, VTI.Size>;
```
- EN: Declares TableGen `multiclass MVE_VxxMUL_qr_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VxxMUL_qr_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5613-5617
```tablegen
  let Predicates = [HasMVEInt] in {
    defm : MVE_TwoOpPatternDup<VTI, Op, int_pred, (? ), !cast<Instruction>(NAME)>;
  }
  defm : MVE_vec_scalar_int_pat_m<!cast<Instruction>(NAME), VTI, int_unpred, int_pred>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5619-5621
```tablegen
multiclass MVE_VQDMULH_qr_m<MVEVectorVTInfo VTI> :
  MVE_VxxMUL_qr_m<"vqdmulh", VTI, 0b0, MVEvqdmulh,
                  int_arm_mve_vqdmulh, int_arm_mve_qdmulh_predicated>;
```
- EN: Declares TableGen `multiclass MVE_VQDMULH_qr_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VQDMULH_qr_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5623-5625
```tablegen
multiclass MVE_VQRDMULH_qr_m<MVEVectorVTInfo VTI> :
  MVE_VxxMUL_qr_m<"vqrdmulh", VTI, 0b1, null_frag,
                  int_arm_mve_vqrdmulh, int_arm_mve_qrdmulh_predicated>;
```
- EN: Declares TableGen `multiclass MVE_VQRDMULH_qr_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VQRDMULH_qr_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5627-5629
```tablegen
defm MVE_VQDMULH_qr_s8    : MVE_VQDMULH_qr_m<MVE_v16s8>;
defm MVE_VQDMULH_qr_s16   : MVE_VQDMULH_qr_m<MVE_v8s16>;
defm MVE_VQDMULH_qr_s32   : MVE_VQDMULH_qr_m<MVE_v4s32>;
```
- EN: Defines TableGen record `MVE_VQDMULH_qr_s8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQDMULH_qr_s8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5631-5633
```tablegen
defm MVE_VQRDMULH_qr_s8   : MVE_VQRDMULH_qr_m<MVE_v16s8>;
defm MVE_VQRDMULH_qr_s16  : MVE_VQRDMULH_qr_m<MVE_v8s16>;
defm MVE_VQRDMULH_qr_s32  : MVE_VQRDMULH_qr_m<MVE_v4s32>;
```
- EN: Defines TableGen record `MVE_VQRDMULH_qr_s8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQRDMULH_qr_s8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5635-5640
```tablegen
multiclass MVE_VxxMUL_qr_f_m<MVEVectorVTInfo VTI, SDPatternOperator IdentityVec> {
  let validForTailPredication = 1 in
  def "" : MVE_VxxMUL_qr<"vmul", VTI.Suffix, VTI.Size{0}, 0b11, VTI.Size>;
  defm : MVE_TwoOpPatternDup<VTI, vmul, int_arm_mve_mul_predicated, (? ),
                             !cast<Instruction>(NAME), IdentityVec>;
}
```
- EN: Declares TableGen `multiclass MVE_VxxMUL_qr_f_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VxxMUL_qr_f_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5642-5645
```tablegen
let Predicates = [HasMVEFloat] in {
  defm MVE_VMUL_qr_f16   : MVE_VxxMUL_qr_f_m<MVE_v8f16, ARMimmOneH>;
  defm MVE_VMUL_qr_f32   : MVE_VxxMUL_qr_f_m<MVE_v4f32, ARMimmOneF>;
}
```
- EN: Defines TableGen record `MVE_VMUL_qr_f16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMUL_qr_f16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5647-5650
```tablegen
class MVE_VFMAMLA_qr<string iname, string suffix,
                     bit bit_28, bits<2> bits_21_20, bit S,
                     bits<2> vecsize, list<dag> pattern=[]>
  : MVE_qDestSrc_rSrc<iname, suffix, vecsize, pattern> {
```
- EN: Declares reusable TableGen class `MVE_VFMAMLA_qr` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VFMAMLA_qr`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5652-5660
```tablegen
  let Inst{28} = bit_28;
  let Inst{21-20} = bits_21_20;
  let Inst{16} = 0b1;
  let Inst{12} = S;
  let Inst{8} = 0b0;
  let Inst{5} = 0b0;
  let validForTailPredication = 1;
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5662-5672
```tablegen
multiclass MVE_VMLA_qr_multi<string iname, MVEVectorVTInfo VTI,
                             bit scalar_addend> {
  def "": MVE_VFMAMLA_qr<iname, VTI.Suffix, 0b0, VTI.Size,
                         scalar_addend, VTI.Size>;
  defvar Inst = !cast<Instruction>(NAME);
  defvar pred_int = !cast<Intrinsic>("int_arm_mve_" # iname # "_n_predicated");
  defvar v1   = (VTI.Vec MQPR:$v1);
  defvar v2   = (VTI.Vec MQPR:$v2);
  defvar vs   = (VTI.Vec (ARMvdup rGPR:$s));
  defvar s    = (i32 rGPR:$s);
  defvar pred = (VTI.Pred VCCR:$pred);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 5674-5681
```tablegen
  let Predicates = [HasMVEInt] in {
    if scalar_addend then {
      def : Pat<(VTI.Vec (add (mul v1, v2), vs)),
                (VTI.Vec (Inst v1, v2, s))>;
    } else {
      def : Pat<(VTI.Vec (add (mul v2, vs), v1)),
                (VTI.Vec (Inst v1, v2, s))>;
    }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5683-5686
```tablegen
    def : Pat<(VTI.Vec (pred_int v1, v2, s, pred)),
              (VTI.Vec (Inst v1, v2, s, ARMVCCThen, pred, zero_reg))>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5688-5690
```tablegen
defm MVE_VMLA_qr_i8   : MVE_VMLA_qr_multi<"vmla", MVE_v16i8, 0b0>;
defm MVE_VMLA_qr_i16  : MVE_VMLA_qr_multi<"vmla", MVE_v8i16, 0b0>;
defm MVE_VMLA_qr_i32  : MVE_VMLA_qr_multi<"vmla", MVE_v4i32, 0b0>;
```
- EN: Defines TableGen record `MVE_VMLA_qr_i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMLA_qr_i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5692-5694
```tablegen
defm MVE_VMLAS_qr_i8  : MVE_VMLA_qr_multi<"vmlas", MVE_v16i8, 0b1>;
defm MVE_VMLAS_qr_i16 : MVE_VMLA_qr_multi<"vmlas", MVE_v8i16, 0b1>;
defm MVE_VMLAS_qr_i32 : MVE_VMLA_qr_multi<"vmlas", MVE_v4i32, 0b1>;
```
- EN: Defines TableGen record `MVE_VMLAS_qr_i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMLAS_qr_i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5696-5705
```tablegen
multiclass MVE_VFMA_qr_multi<string iname, MVEVectorVTInfo VTI,
                             bit scalar_addend> {
  def "": MVE_VFMAMLA_qr<iname, VTI.Suffix, VTI.Size{0}, 0b11, scalar_addend, VTI.Size>;
  defvar Inst = !cast<Instruction>(NAME);
  defvar pred_int = int_arm_mve_fma_predicated;
  defvar v1   = (VTI.Vec MQPR:$v1);
  defvar v2   = (VTI.Vec MQPR:$v2);
  defvar vs   = (VTI.Vec (ARMvdup (i32 rGPR:$s)));
  defvar is   = (i32 rGPR:$s);
  defvar pred = (VTI.Pred VCCR:$pred);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 5707-5724
```tablegen
  let Predicates = [HasMVEFloat] in {
    if scalar_addend then {
      def : Pat<(VTI.Vec (fma v1, v2, vs)),
                (VTI.Vec (Inst v1, v2, is))>;
      def : Pat<(VTI.Vec (int_arm_mve_fma v1, v2, vs)),
                (VTI.Vec (Inst v1, v2, is))>;
      def : Pat<(VTI.Vec (vselect (VTI.Pred VCCR:$pred),
                                  (VTI.Vec (fma v1, v2, vs)),
                                  v1)),
                (VTI.Vec (Inst v1, v2, is, ARMVCCThen, $pred, zero_reg))>;
    } else {
      def : Pat<(VTI.Vec (fma v1, vs, v2)),
                (VTI.Vec (Inst v2, v1, is))>;
      def : Pat<(VTI.Vec (fma vs, v1, v2)),
                (VTI.Vec (Inst v2, v1, is))>;
      def : Pat<(VTI.Vec (int_arm_mve_fma v1, vs, v2)),
                (VTI.Vec (Inst v2, v1, is))>;
      def : Pat<(VTI.Vec (int_arm_mve_fma vs, v1, v2)),
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5725-5740
```tablegen
                (VTI.Vec (Inst v2, v1, is))>;
      def : Pat<(VTI.Vec (vselect (VTI.Pred VCCR:$pred),
                                  (VTI.Vec (fma vs, v2, v1)),
                                  v1)),
                (VTI.Vec (Inst v1, v2, is, ARMVCCThen, $pred, zero_reg))>;
      def : Pat<(VTI.Vec (vselect (VTI.Pred VCCR:$pred),
                                  (VTI.Vec (fma v2, vs, v1)),
                                  v1)),
                (VTI.Vec (Inst v1, v2, is, ARMVCCThen, $pred, zero_reg))>;
      def : Pat<(VTI.Vec (pred_int v1, vs, v2, pred)),
                (VTI.Vec (Inst v2, v1, is, ARMVCCThen, pred, zero_reg))>;
      def : Pat<(VTI.Vec (pred_int vs, v1, v2, pred)),
                (VTI.Vec (Inst v2, v1, is, ARMVCCThen, pred, zero_reg))>;
    }
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5742-5747
```tablegen
let Predicates = [HasMVEFloat] in {
  defm MVE_VFMA_qr_f16  : MVE_VFMA_qr_multi<"vfma",  MVE_v8f16, 0>;
  defm MVE_VFMA_qr_f32  : MVE_VFMA_qr_multi<"vfma",  MVE_v4f32, 0>;
  defm MVE_VFMA_qr_Sf16 : MVE_VFMA_qr_multi<"vfmas", MVE_v8f16, 1>;
  defm MVE_VFMA_qr_Sf32 : MVE_VFMA_qr_multi<"vfmas", MVE_v4f32, 1>;
}
```
- EN: Defines TableGen record `MVE_VFMA_qr_f16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VFMA_qr_f16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5749-5751
```tablegen
class MVE_VQDMLAH_qr<string iname, string suffix, bit U, bits<2> size,
                     bit bit_5, bit bit_12, list<dag> pattern=[]>
  : MVE_qDestSrc_rSrc<iname, suffix, size, pattern> {
```
- EN: Declares reusable TableGen class `MVE_VQDMLAH_qr` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VQDMLAH_qr`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5753-5759
```tablegen
  let Inst{28} = U;
  let Inst{21-20} = size;
  let Inst{16} = 0b0;
  let Inst{12} = bit_12;
  let Inst{8} = 0b0;
  let Inst{5} = bit_5;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5761-5766
```tablegen
multiclass MVE_VQDMLAH_qr_multi<string iname, MVEVectorVTInfo VTI,
                                bit bit_5, bit bit_12> {
  def "": MVE_VQDMLAH_qr<iname, VTI.Suffix, 0b0, VTI.Size, bit_5, bit_12>;
  defvar Inst = !cast<Instruction>(NAME);
  defvar unpred_int = !cast<Intrinsic>("int_arm_mve_" # iname);
  defvar pred_int = !cast<Intrinsic>("int_arm_mve_" # iname # "_predicated");
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 5768-5779
```tablegen
  let Predicates = [HasMVEInt] in {
    def : Pat<(VTI.Vec (unpred_int (VTI.Vec MQPR:$v1), (VTI.Vec MQPR:$v2),
                                   (i32 rGPR:$s))),
              (VTI.Vec (Inst       (VTI.Vec MQPR:$v1), (VTI.Vec MQPR:$v2),
                                   (i32 rGPR:$s)))>;
    def : Pat<(VTI.Vec (pred_int   (VTI.Vec MQPR:$v1), (VTI.Vec MQPR:$v2),
                                   (i32 rGPR:$s), (VTI.Pred VCCR:$pred))),
              (VTI.Vec (Inst       (VTI.Vec MQPR:$v1), (VTI.Vec MQPR:$v2),
                                   (i32 rGPR:$s), ARMVCCThen,
                                   (VTI.Pred VCCR:$pred), zero_reg))>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5781-5785
```tablegen
multiclass MVE_VQDMLAH_qr_types<string iname, bit bit_5, bit bit_12> {
  defm s8  : MVE_VQDMLAH_qr_multi<iname, MVE_v16s8, bit_5, bit_12>;
  defm s16 : MVE_VQDMLAH_qr_multi<iname, MVE_v8s16, bit_5, bit_12>;
  defm s32 : MVE_VQDMLAH_qr_multi<iname, MVE_v4s32, bit_5, bit_12>;
}
```
- EN: Declares TableGen `multiclass MVE_VQDMLAH_qr_types`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VQDMLAH_qr_types`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5787-5790
```tablegen
defm MVE_VQDMLAH_qr   : MVE_VQDMLAH_qr_types<"vqdmlah",   0b1, 0b0>;
defm MVE_VQRDMLAH_qr  : MVE_VQDMLAH_qr_types<"vqrdmlah",  0b0, 0b0>;
defm MVE_VQDMLASH_qr  : MVE_VQDMLAH_qr_types<"vqdmlash",  0b1, 0b1>;
defm MVE_VQRDMLASH_qr : MVE_VQDMLAH_qr_types<"vqrdmlash", 0b0, 0b1>;
```
- EN: Defines TableGen record `MVE_VQDMLAH_qr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VQDMLAH_qr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5792-5801
```tablegen
class MVE_VxDUP<string iname, string suffix, bits<2> size, bit bit_12,
                ValueType VT, SDPatternOperator vxdup>
  : MVE_p<(outs MQPR:$Qd, tGPREven:$Rn),
          (ins tGPREven:$Rn_src, MVE_VIDUP_imm:$imm), NoItinerary,
          iname, suffix, "$Qd, $Rn, $imm", vpred_r, "$Rn = $Rn_src", size,
          [(set (VT MQPR:$Qd), (i32 tGPREven:$Rn),
              (vxdup (i32 tGPREven:$Rn_src), (i32 imm:$imm)))]> {
  bits<4> Qd;
  bits<4> Rn;
  bits<2> imm;
```
- EN: Declares reusable TableGen class `MVE_VxDUP` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VxDUP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5803-5817
```tablegen
  let Inst{28} = 0b0;
  let Inst{25-23} = 0b100;
  let Inst{22} = Qd{3};
  let Inst{21-20} = size;
  let Inst{19-17} = Rn{3-1};
  let Inst{16} = 0b1;
  let Inst{15-13} = Qd{2-0};
  let Inst{12} = bit_12;
  let Inst{11-8} = 0b1111;
  let Inst{7} = imm{1};
  let Inst{6-1} = 0b110111;
  let Inst{0} = imm{0};
  let validForTailPredication = 1;
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5819-5821
```tablegen
def MVE_VIDUPu8  : MVE_VxDUP<"vidup", "u8",  0b00, 0b0, v16i8, ARMvidup>;
def MVE_VIDUPu16 : MVE_VxDUP<"vidup", "u16", 0b01, 0b0, v8i16, ARMvidup>;
def MVE_VIDUPu32 : MVE_VxDUP<"vidup", "u32", 0b10, 0b0, v4i32, ARMvidup>;
```
- EN: Defines TableGen record `MVE_VIDUPu8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VIDUPu8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5823-5825
```tablegen
def MVE_VDDUPu8  : MVE_VxDUP<"vddup", "u8",  0b00, 0b1, v16i8, null_frag>;
def MVE_VDDUPu16 : MVE_VxDUP<"vddup", "u16", 0b01, 0b1, v8i16, null_frag>;
def MVE_VDDUPu32 : MVE_VxDUP<"vddup", "u32", 0b10, 0b1, v4i32, null_frag>;
```
- EN: Defines TableGen record `MVE_VDDUPu8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VDDUPu8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5827-5836
```tablegen
class MVE_VxWDUP<string iname, string suffix, bits<2> size, bit bit_12,
                 list<dag> pattern=[]>
  : MVE_p<(outs MQPR:$Qd, tGPREven:$Rn),
          (ins tGPREven:$Rn_src, tGPROdd:$Rm, MVE_VIDUP_imm:$imm), NoItinerary,
          iname, suffix, "$Qd, $Rn, $Rm, $imm", vpred_r, "$Rn = $Rn_src", size,
          pattern> {
  bits<4> Qd;
  bits<4> Rm;
  bits<4> Rn;
  bits<2> imm;
```
- EN: Declares reusable TableGen class `MVE_VxWDUP` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VxWDUP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5838-5853
```tablegen
  let Inst{28} = 0b0;
  let Inst{25-23} = 0b100;
  let Inst{22} = Qd{3};
  let Inst{21-20} = size;
  let Inst{19-17} = Rn{3-1};
  let Inst{16} = 0b1;
  let Inst{15-13} = Qd{2-0};
  let Inst{12} = bit_12;
  let Inst{11-8} = 0b1111;
  let Inst{7} = imm{1};
  let Inst{6-4} = 0b110;
  let Inst{3-1} = Rm{3-1};
  let Inst{0} = imm{0};
  let validForTailPredication = 1;
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5855-5857
```tablegen
def MVE_VIWDUPu8  : MVE_VxWDUP<"viwdup", "u8",  0b00, 0b0>;
def MVE_VIWDUPu16 : MVE_VxWDUP<"viwdup", "u16", 0b01, 0b0>;
def MVE_VIWDUPu32 : MVE_VxWDUP<"viwdup", "u32", 0b10, 0b0>;
```
- EN: Defines TableGen record `MVE_VIWDUPu8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VIWDUPu8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5859-5861
```tablegen
def MVE_VDWDUPu8  : MVE_VxWDUP<"vdwdup", "u8",  0b00, 0b1>;
def MVE_VDWDUPu16 : MVE_VxWDUP<"vdwdup", "u16", 0b01, 0b1>;
def MVE_VDWDUPu32 : MVE_VxWDUP<"vdwdup", "u32", 0b10, 0b1>;
```
- EN: Defines TableGen record `MVE_VDWDUPu8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VDWDUPu8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5863-5867
```tablegen
let isReMaterializable = 1 in
class MVE_VCTPInst<string suffix, bits<2> size, list<dag> pattern=[]>
  : MVE_p<(outs VCCR:$P0), (ins rGPR:$Rn), NoItinerary, "vctp", suffix,
          "$Rn", vpred_n, "", size, pattern> {
  bits<4> Rn;
```
- EN: Declares reusable TableGen class `MVE_VCTPInst` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VCTPInst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5869-5875
```tablegen
  let Inst{28-27} = 0b10;
  let Inst{26-22} = 0b00000;
  let Inst{21-20} = size;
  let Inst{19-16} = Rn{3-0};
  let Inst{15-11} = 0b11101;
  let Inst{10-0}  = 0b00000000001;
  let Unpredictable{10-0} = 0b11111111111;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5877-5880
```tablegen
  let Constraints = "";
  let DecoderMethod = "DecodeMveVCTP";
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5882-5884
```tablegen
multiclass MVE_VCTP<MVEVectorVTInfo VTI, Intrinsic intr> {
  def "": MVE_VCTPInst<VTI.BitsSuffix, VTI.Size>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 5886-5892
```tablegen
  let Predicates = [HasMVEInt] in {
    def : Pat<(intr rGPR:$Rn),
              (VTI.Pred (Inst rGPR:$Rn))>;
    def : Pat<(and (intr rGPR:$Rn), (VTI.Pred VCCR:$mask)),
              (VTI.Pred (Inst rGPR:$Rn, ARMVCCThen, VCCR:$mask, zero_reg))>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5894-5897
```tablegen
defm MVE_VCTP8  : MVE_VCTP<MVE_v16i8, int_arm_mve_vctp8>;
defm MVE_VCTP16 : MVE_VCTP<MVE_v8i16, int_arm_mve_vctp16>;
defm MVE_VCTP32 : MVE_VCTP<MVE_v4i32, int_arm_mve_vctp32>;
defm MVE_VCTP64 : MVE_VCTP<MVE_v2i64, int_arm_mve_vctp64>;
```
- EN: Defines TableGen record `MVE_VCTP8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VCTP8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5899-5899
```tablegen
// end of mve_qDest_rSrc
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 5901-5901
```tablegen
// start of coproc mov
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 5903-5911
```tablegen
class MVE_VMOV_64bit<dag oops, dag iops, bit to_qreg, string ops, string cstr>
  : MVE_VMOV_lane_base<oops, !con(iops, (ins MVEPairVectorIndex2:$idx,
                                             MVEPairVectorIndex0:$idx2)),
                       NoItinerary, "vmov", "", ops, cstr, []> {
  bits<5> Rt;
  bits<5> Rt2;
  bits<4> Qd;
  bit idx;
  bit idx2;
```
- EN: Declares reusable TableGen class `MVE_VMOV_64bit` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VMOV_64bit`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5913-5921
```tablegen
  let Inst{31-23} = 0b111011000;
  let Inst{22} = Qd{3};
  let Inst{21} = 0b0;
  let Inst{20} = to_qreg;
  let Inst{19-16} = Rt2{3-0};
  let Inst{15-13} = Qd{2-0};
  let Inst{12-5} = 0b01111000;
  let Inst{4} = idx2;
  let Inst{3-0} = Rt{3-0};
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5923-5925
```tablegen
  let VecSize = 0b10;
  let hasSideEffects = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5927-5944
```tablegen
// The assembly syntax for these instructions mentions the vector
// register name twice, e.g.
//
//    vmov q2[2], q2[0], r0, r1
//    vmov r0, r1, q2[2], q2[0]
//
// which needs a bit of juggling with MC operand handling.
//
// For the move _into_ a vector register, the MC operand list also has
// to mention the register name twice: once as the output, and once as
// an extra input to represent where the unchanged half of the output
// register comes from (when this instruction is used in code
// generation). So we arrange that the first mention of the vector reg
// in the instruction is considered by the AsmMatcher to be the output
// ($Qd), and the second one is the input ($QdSrc). Binding them
// together with the existing 'tie' constraint is enough to enforce at
// register allocation time that they have to be the same register.
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 5945-5952
```tablegen
// For the move _from_ a vector register, there's no way to get round
// the fact that both instances of that register name have to be
// inputs. They have to be the same register again, but this time, we
// can't use a tie constraint, because that has to be between an
// output and an input operand. So this time, we have to arrange that
// the q-reg appears just once in the MC operand list, in spite of
// being mentioned twice in the asm syntax - which needs a custom
// AsmMatchConverter.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 5954-5959
```tablegen
def MVE_VMOV_q_rr : MVE_VMOV_64bit<(outs MQPR:$Qd),
                                   (ins MQPR:$QdSrc, rGPR:$Rt, rGPR:$Rt2),
                                   0b1, "$Qd$idx, $QdSrc$idx2, $Rt, $Rt2",
                                   "$Qd = $QdSrc"> {
  let DecoderMethod = "DecodeMVEVMOVDRegtoQ";
}
```
- EN: Defines TableGen record `MVE_VMOV_q_rr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMOV_q_rr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5961-5965
```tablegen
def MVE_VMOV_rr_q : MVE_VMOV_64bit<(outs rGPR:$Rt, rGPR:$Rt2), (ins MQPR:$Qd),
                                   0b0, "$Rt, $Rt2, $Qd$idx, $Qd$idx2", ""> {
  let DecoderMethod = "DecodeMVEVMOVQtoDReg";
  let AsmMatchConverter = "cvtMVEVMOVQtoDReg";
}
```
- EN: Defines TableGen record `MVE_VMOV_rr_q` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VMOV_rr_q`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5967-5984
```tablegen
let Predicates = [HasMVEInt] in {
  // Double lane moves. There are a number of patterns here. We know that the
  // insertelt's will be in descending order by index, and need to match the 5
  // patterns that might contain 2-0 or 3-1 pairs. These are:
  // 3 2 1 0    -> vmovqrr 31; vmovqrr 20
  // 3 2 1      -> vmovqrr 31; vmov 2
  // 3 1        -> vmovqrr 31
  // 2 1 0      -> vmovqrr 20; vmov 1
  // 2 0        -> vmovqrr 20
  // The other potential patterns will be handled by single lane inserts.
  def : Pat<(insertelt (insertelt (insertelt (insertelt (v4i32 MQPR:$src1),
                                                        rGPR:$srcA, (i32 0)),
                                             rGPR:$srcB, (i32 1)),
                                  rGPR:$srcC, (i32 2)),
                       rGPR:$srcD, (i32 3)),
            (MVE_VMOV_q_rr (MVE_VMOV_q_rr MQPR:$src1, rGPR:$srcA, rGPR:$srcC, (i32 2), (i32 0)),
                           rGPR:$srcB, rGPR:$srcD, (i32 3), (i32 1))>;
  def : Pat<(insertelt (insertelt (insertelt (v4i32 MQPR:$src1),
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5985-6000
```tablegen
                                             rGPR:$srcB, (i32 1)),
                                  rGPR:$srcC, (i32 2)),
                       rGPR:$srcD, (i32 3)),
            (MVE_VMOV_q_rr (MVE_VMOV_to_lane_32 MQPR:$src1, rGPR:$srcC, (i32 2)),
                           rGPR:$srcB, rGPR:$srcD, (i32 3), (i32 1))>;
  def : Pat<(insertelt (insertelt (v4i32 MQPR:$src1), rGPR:$srcA, (i32 1)), rGPR:$srcB, (i32 3)),
            (MVE_VMOV_q_rr MQPR:$src1, rGPR:$srcA, rGPR:$srcB, (i32 3), (i32 1))>;
  def : Pat<(insertelt (insertelt (insertelt (v4i32 MQPR:$src1),
                                             rGPR:$srcB, (i32 0)),
                                  rGPR:$srcC, (i32 1)),
                       rGPR:$srcD, (i32 2)),
            (MVE_VMOV_q_rr (MVE_VMOV_to_lane_32 MQPR:$src1, rGPR:$srcC, (i32 1)),
                           rGPR:$srcB, rGPR:$srcD, (i32 2), (i32 0))>;
  def : Pat<(insertelt (insertelt (v4i32 MQPR:$src1), rGPR:$srcA, (i32 0)), rGPR:$srcB, (i32 2)),
            (MVE_VMOV_q_rr MQPR:$src1, rGPR:$srcA, rGPR:$srcB, (i32 2), (i32 0))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6002-6002
```tablegen
// end of coproc mov
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6004-6004
```tablegen
// start of MVE interleaving load/store
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6006-6014
```tablegen
// Base class for the family of interleaving/deinterleaving
// load/stores with names like VLD20.8 and VST43.32.
class MVE_vldst24_base<bit writeback, bit fourregs, bits<2> stage, bits<2> size,
                       bit load, dag Oops, dag loadIops, dag wbIops,
                       string iname, string ops,
                       string cstr, list<dag> pattern=[]>
  : MVE_MI<Oops, !con(loadIops, wbIops), NoItinerary, iname, ops, cstr, size, pattern> {
  bits<4> VQd;
  bits<4> Rn;
```
- EN: Declares reusable TableGen class `for` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `for`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6016-6025
```tablegen
  let Inst{31-22} = 0b1111110010;
  let Inst{21} = writeback;
  let Inst{20} = load;
  let Inst{19-16} = Rn;
  let Inst{15-13} = VQd{2-0};
  let Inst{12-9} = 0b1111;
  let Inst{8-7} = size;
  let Inst{6-5} = stage;
  let Inst{4-1} = 0b0000;
  let Inst{0} = fourregs;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6027-6031
```tablegen
  let mayLoad = load;
  let mayStore = !eq(load,0);
  let hasSideEffects = 0;
  let validForTailPredication = load;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6033-6043
```tablegen
// A parameter class used to encapsulate all the ways the writeback
// variants of VLD20 and friends differ from the non-writeback ones.
class MVE_vldst24_writeback<bit b, dag Oo, dag Io,
                            string sy="", string c="", string n=""> {
  bit writeback = b;
  dag Oops = Oo;
  dag Iops = Io;
  string syntax = sy;
  string cstr = c;
  string id_suffix = n;
}
```
- EN: Declares reusable TableGen class `used` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `used`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6045-6052
```tablegen
// Another parameter class that encapsulates the differences between VLD2x
// and VLD4x.
class MVE_vldst24_nvecs<int n, list<int> s, bit b, RegisterOperand vl> {
  int nvecs = n;
  list<int> stages = s;
  bit bit0 = b;
  RegisterOperand VecList = vl;
}
```
- EN: Declares reusable TableGen class `that` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `that`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6054-6058
```tablegen
// A third parameter class that distinguishes VLDnn.8 from .16 from .32.
class MVE_vldst24_lanesize<int i, bits<2> b> {
  int lanesize = i;
  bits<2> sizebits = b;
}
```
- EN: Declares reusable TableGen class `that` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `that`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6060-6063
```tablegen
// A base class for each direction of transfer: one for load, one for
// store. I can't make these a fourth independent parametric tuple
// class, because they have to take the nvecs tuple class as a
// parameter, in order to find the right VecList operand type.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6065-6072
```tablegen
class MVE_vld24_base<MVE_vldst24_nvecs n, bits<2> pat, bits<2> size,
                     MVE_vldst24_writeback wb, string iname,
                     list<dag> pattern=[]>
  : MVE_vldst24_base<wb.writeback, n.bit0, pat, size, 1,
                     !con((outs n.VecList:$VQd), wb.Oops),
                     (ins n.VecList:$VQdSrc), wb.Iops,
                     iname, "$VQd, $Rn" # wb.syntax,
                     wb.cstr # ",$VQdSrc = $VQd", pattern>;
```
- EN: Declares reusable TableGen class `MVE_vld24_base` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_vld24_base`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6074-6080
```tablegen
class MVE_vst24_base<MVE_vldst24_nvecs n, bits<2> pat, bits<2> size,
                     MVE_vldst24_writeback wb, string iname,
                     list<dag> pattern=[]>
  : MVE_vldst24_base<wb.writeback, n.bit0, pat, size, 0,
                     wb.Oops, (ins n.VecList:$VQd), wb.Iops,
                     iname, "$VQd, $Rn" # wb.syntax,
                     wb.cstr, pattern>;
```
- EN: Declares reusable TableGen class `MVE_vst24_base` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_vst24_base`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6082-6095
```tablegen
// Actually define all the interleaving loads and stores, by a series
// of nested foreaches over number of vectors (VLD2/VLD4); stage
// within one of those series (VLDx0/VLDx1/VLDx2/VLDx3); size of
// vector lane; writeback or no writeback.
foreach n = [MVE_vldst24_nvecs<2, [0,1],     0, VecList2Q>,
             MVE_vldst24_nvecs<4, [0,1,2,3], 1, VecList4Q>] in
foreach stage = n.stages in
foreach s = [MVE_vldst24_lanesize< 8, 0b00>,
             MVE_vldst24_lanesize<16, 0b01>,
             MVE_vldst24_lanesize<32, 0b10>] in
foreach wb = [MVE_vldst24_writeback<
                1, (outs rGPR:$wb), (ins t2_nosp_addr_offset_none:$Rn),
                "!", "$Rn.base = $wb", "_wb">,
              MVE_vldst24_writeback<0, (outs), (ins t2_addr_offset_none:$Rn)>] in {
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6097-6100
```tablegen
  // For each case within all of those foreaches, define the actual
  // instructions. The def names are made by gluing together pieces
  // from all the parameter classes, and will end up being things like
  // MVE_VLD20_8 and MVE_VST43_16_wb.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6102-6104
```tablegen
  def "MVE_VLD" # n.nvecs # stage # "_" # s.lanesize # wb.id_suffix
    : MVE_vld24_base<n, stage, s.sizebits, wb,
                     "vld" # n.nvecs # stage # "." # s.lanesize>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6106-6109
```tablegen
  def "MVE_VST" # n.nvecs # stage # "_" # s.lanesize # wb.id_suffix
    : MVE_vst24_base<n, stage, s.sizebits, wb,
                     "vst" # n.nvecs # stage # "." # s.lanesize>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6111-6117
```tablegen
def SDTARMVST2    : SDTypeProfile<1, 5, [SDTCisPtrTy<0>, SDTCisPtrTy<1>, SDTCisVT<2, i32>, SDTCisVec<3>,
                                         SDTCisSameAs<3, 4>, SDTCisVT<5, i32>]>;
def SDTARMVST4    : SDTypeProfile<1, 7, [SDTCisPtrTy<0>, SDTCisPtrTy<1>, SDTCisVT<2, i32>, SDTCisVec<3>,
                                         SDTCisSameAs<3, 4>, SDTCisSameAs<3, 5>,
                                         SDTCisSameAs<3, 6>, SDTCisVT<7, i32>]>;
def MVEVST2UPD       : SDNode<"ARMISD::VST2_UPD", SDTARMVST2, [SDNPHasChain, SDNPMemOperand]>;
def MVEVST4UPD       : SDNode<"ARMISD::VST4_UPD", SDTARMVST4, [SDNPHasChain, SDNPMemOperand]>;
```
- EN: Defines TableGen record `SDTARMVST2` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTARMVST2`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 6119-6131
```tablegen
multiclass MVE_vst24_patterns<int lanesize, ValueType VT> {
  foreach stage = [0,1] in
    def : Pat<(int_arm_mve_vst2q i32:$addr,
                (VT MQPR:$v0), (VT MQPR:$v1), (i32 stage)),
              (!cast<Instruction>("MVE_VST2"#stage#"_"#lanesize)
                (REG_SEQUENCE MQQPR, VT:$v0, qsub_0, VT:$v1, qsub_1),
                t2_addr_offset_none:$addr)>;
  foreach stage = [0,1] in
    def : Pat<(i32 (MVEVST2UPD i32:$addr, (i32 32),
                (VT MQPR:$v0), (VT MQPR:$v1), (i32 stage))),
              (i32 (!cast<Instruction>("MVE_VST2"#stage#"_"#lanesize#_wb)
                (REG_SEQUENCE MQQPR, VT:$v0, qsub_0, VT:$v1, qsub_1),
                t2_addr_offset_none:$addr))>;
```
- EN: Declares TableGen `multiclass MVE_vst24_patterns`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_vst24_patterns`，它是一个可复用模板，可展开为多个相关记录。

### Lines 6133-6150
```tablegen
  foreach stage = [0,1,2,3] in
    def : Pat<(int_arm_mve_vst4q i32:$addr,
                (VT MQPR:$v0), (VT MQPR:$v1),
                (VT MQPR:$v2), (VT MQPR:$v3), (i32 stage)),
              (!cast<Instruction>("MVE_VST4"#stage#"_"#lanesize)
                (REG_SEQUENCE MQQQQPR, VT:$v0, qsub_0, VT:$v1, qsub_1,
                                       VT:$v2, qsub_2, VT:$v3, qsub_3),
                t2_addr_offset_none:$addr)>;
  foreach stage = [0,1,2,3] in
    def : Pat<(i32 (MVEVST4UPD i32:$addr, (i32 64),
                (VT MQPR:$v0), (VT MQPR:$v1),
                (VT MQPR:$v2), (VT MQPR:$v3), (i32 stage))),
              (i32 (!cast<Instruction>("MVE_VST4"#stage#"_"#lanesize#_wb)
                (REG_SEQUENCE MQQQQPR, VT:$v0, qsub_0, VT:$v1, qsub_1,
                                       VT:$v2, qsub_2, VT:$v3, qsub_3),
                t2_addr_offset_none:$addr))>;
}
defm : MVE_vst24_patterns<8, v16i8>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6151-6154
```tablegen
defm : MVE_vst24_patterns<16, v8i16>;
defm : MVE_vst24_patterns<32, v4i32>;
defm : MVE_vst24_patterns<16, v8f16>;
defm : MVE_vst24_patterns<32, v4f32>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6156-6156
```tablegen
// end of MVE interleaving load/store
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6158-6158
```tablegen
// start of MVE predicable load/store
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6160-6168
```tablegen
// A parameter class for the direction of transfer.
class MVE_ldst_direction<bit b, dag Oo, dag Io, string c=""> {
  bit load = b;
  dag Oops = Oo;
  dag Iops = Io;
  string cstr = c;
}
def MVE_ld: MVE_ldst_direction<1, (outs MQPR:$Qd), (ins), ",@earlyclobber $Qd">;
def MVE_st: MVE_ldst_direction<0, (outs), (ins MQPR:$Qd)>;
```
- EN: Declares reusable TableGen class `for` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `for`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6170-6174
```tablegen
// A parameter class for the size of memory access in a load.
class MVE_memsz<bits<2> e, int s, AddrMode m, string mn, list<string> types> {
  bits<2> encoding = e;         // opcode bit(s) for encoding
  int shift = s;                // shift applied to immediate load offset
  AddrMode AM = m;
```
- EN: Declares reusable TableGen class `for` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `for`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6176-6184
```tablegen
  // For instruction aliases: define the complete list of type
  // suffixes at this size, and the canonical ones for loads and
  // stores.
  string MnemonicLetter = mn;
  int TypeBits = !shl(8, s);
  string CanonLoadSuffix = ".u" # TypeBits;
  string CanonStoreSuffix = "." # TypeBits;
  list<string> suffixes = !foreach(letter, types, "." # letter # TypeBits);
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6186-6193
```tablegen
// Instances of MVE_memsz.
//
// (memD doesn't need an AddrMode, because those are only for
// contiguous loads, and memD is only used by gather/scatters.)
def MVE_memB: MVE_memsz<0b00, 0, AddrModeT2_i7,   "b", ["", "u", "s"]>;
def MVE_memH: MVE_memsz<0b01, 1, AddrModeT2_i7s2, "h", ["", "u", "s", "f"]>;
def MVE_memW: MVE_memsz<0b10, 2, AddrModeT2_i7s4, "w", ["", "u", "s", "f"]>;
def MVE_memD: MVE_memsz<0b11, 3, ?,               "d", ["", "u", "s", "f"]>;
```
- EN: Defines TableGen record `MVE_memB:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_memB:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6195-6212
```tablegen
// This is the base class for all the MVE loads and stores other than
// the interleaving ones. All the non-interleaving loads/stores share
// the characteristic that they operate on just one vector register,
// so they are VPT-predicable.
//
// The predication operand is vpred_n, for both loads and stores. For
// store instructions, the reason is obvious: if there is no output
// register, there can't be a need for an input parameter giving the
// output register's previous value. Load instructions also don't need
// that input parameter, because unlike MVE data processing
// instructions, predicated loads are defined to set the inactive
// lanes of the output register to zero, instead of preserving their
// input values.
class MVE_VLDRSTR_base<MVE_ldst_direction dir, bit U, bit P, bit W, bit opc,
                       dag oops, dag iops, string asm, string suffix,
                       string ops, string cstr, bits<2> vecsize, list<dag> pattern=[]>
 : MVE_p<oops, iops, NoItinerary, asm, suffix, ops, vpred_n, cstr, vecsize, pattern> {
  bits<3> Qd;
```
- EN: Declares reusable TableGen class `for` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `for`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6214-6222
```tablegen
  let Inst{28} = U;
  let Inst{25} = 0b0;
  let Inst{24} = P;
  let Inst{22} = 0b0;
  let Inst{21} = W;
  let Inst{20} = dir.load;
  let Inst{15-13} = Qd{2-0};
  let Inst{12} = opc;
  let Inst{11-9} = 0b111;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6224-6228
```tablegen
  let mayLoad = dir.load;
  let mayStore = !eq(dir.load,0);
  let hasSideEffects = 0;
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6230-6241
```tablegen
// Contiguous load and store instructions. These come in two main
// categories: same-size loads/stores in which 128 bits of vector
// register is transferred to or from 128 bits of memory in the most
// obvious way, and widening loads / narrowing stores, in which the
// size of memory accessed is less than the size of a vector register,
// so the load instructions sign- or zero-extend each memory value
// into a wider vector lane, and the store instructions truncate
// correspondingly.
//
// The instruction mnemonics for these two classes look reasonably
// similar, but the actual encodings are different enough to need two
// separate base classes.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6243-6252
```tablegen
// Contiguous, same size
class MVE_VLDRSTR_cs<MVE_ldst_direction dir, MVE_memsz memsz, bit P, bit W,
                     dag oops, dag iops, string asm, string suffix,
                     IndexMode im, string ops, string cstr>
  : MVE_VLDRSTR_base<dir, 0, P, W, 1, oops, iops, asm, suffix, ops, cstr, memsz.encoding> {
  bits<12> addr;
  let Inst{23} = addr{7};
  let Inst{19-16} = addr{11-8};
  let Inst{8-7} = memsz.encoding;
  let Inst{6-0} = addr{6-0};
```
- EN: Declares reusable TableGen class `MVE_VLDRSTR_cs` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VLDRSTR_cs`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6254-6255
```tablegen
  let IM = im;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6257-6268
```tablegen
// Contiguous, widening/narrowing
class MVE_VLDRSTR_cw<MVE_ldst_direction dir, MVE_memsz memsz, bit U,
                     bit P, bit W, bits<2> size, dag oops, dag iops,
                     string asm, string suffix, IndexMode im,
                     string ops, string cstr>
  : MVE_VLDRSTR_base<dir, U, P, W, 0, oops, iops, asm, suffix, ops, cstr, size> {
  bits<11> addr;
  let Inst{23} = addr{7};
  let Inst{19} = memsz.encoding{0}; // enough to tell 16- from 32-bit
  let Inst{18-16} = addr{10-8};
  let Inst{8-7} = size;
  let Inst{6-0} = addr{6-0};
```
- EN: Declares reusable TableGen class `MVE_VLDRSTR_cw` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VLDRSTR_cw`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6270-6271
```tablegen
  let IM = im;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6273-6274
```tablegen
// Multiclass wrapper on each of the _cw and _cs base classes, to
// generate three writeback modes (none, preindex, postindex).
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6276-6282
```tablegen
multiclass MVE_VLDRSTR_cw_m<MVE_ldst_direction dir, MVE_memsz memsz,
                            string asm, string suffix, bit U, bits<2> size> {
  let AM = memsz.AM in {
    def "" : MVE_VLDRSTR_cw<
        dir, memsz, U, 1, 0, size,
        dir.Oops, !con(dir.Iops, (ins taddrmode_imm7<memsz.shift>:$addr)),
        asm, suffix, IndexModeNone, "$Qd, $addr", "">;
```
- EN: Declares TableGen `multiclass MVE_VLDRSTR_cw_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VLDRSTR_cw_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 6284-6290
```tablegen
    def _pre : MVE_VLDRSTR_cw<
        dir, memsz, U, 1, 1, size,
        !con((outs tGPR:$wb), dir.Oops),
        !con(dir.Iops, (ins taddrmode_imm7<memsz.shift>:$addr)),
        asm, suffix, IndexModePre, "$Qd, $addr!", "$addr.base = $wb"> {
      let DecoderMethod = "DecodeMVE_MEM_1_pre<"#memsz.shift#">";
    }
```
- EN: Defines TableGen record `_pre` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_pre`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6292-6302
```tablegen
    def _post : MVE_VLDRSTR_cw<
        dir, memsz, U, 0, 1, size,
        !con((outs tGPR:$wb), dir.Oops),
        !con(dir.Iops, (ins t_addr_offset_none:$Rn,
                            t2am_imm7_offset<memsz.shift>:$addr)),
        asm, suffix, IndexModePost, "$Qd, $Rn$addr", "$Rn.base = $wb"> {
      bits<4> Rn;
      let Inst{18-16} = Rn{2-0};
    }
  }
}
```
- EN: Defines TableGen record `_post` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_post`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6304-6310
```tablegen
multiclass MVE_VLDRSTR_cs_m<MVE_ldst_direction dir, MVE_memsz memsz,
                            string asm, string suffix> {
  let AM = memsz.AM in {
    def "" : MVE_VLDRSTR_cs<
        dir, memsz, 1, 0,
        dir.Oops, !con(dir.Iops, (ins t2addrmode_imm7<memsz.shift>:$addr)),
        asm, suffix, IndexModeNone, "$Qd, $addr", "">;
```
- EN: Declares TableGen `multiclass MVE_VLDRSTR_cs_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VLDRSTR_cs_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 6312-6318
```tablegen
    def _pre : MVE_VLDRSTR_cs<
        dir, memsz, 1, 1,
        !con((outs rGPR:$wb), dir.Oops),
        !con(dir.Iops, (ins t2addrmode_imm7_pre<memsz.shift>:$addr)),
        asm, suffix, IndexModePre, "$Qd, $addr!", "$addr.base = $wb"> {
      let DecoderMethod = "DecodeMVE_MEM_2_pre<"#memsz.shift#">";
    }
```
- EN: Defines TableGen record `_pre` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_pre`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6320-6330
```tablegen
    def _post : MVE_VLDRSTR_cs<
        dir, memsz, 0, 1,
        !con((outs rGPR:$wb), dir.Oops),
        !con(dir.Iops, (ins t2_nosp_addr_offset_none:$Rn,
                            t2am_imm7_offset<memsz.shift>:$addr)),
        asm, suffix, IndexModePost, "$Qd, $Rn$addr", "$Rn.base = $wb"> {
      bits<4> Rn;
      let Inst{19-16} = Rn{3-0};
    }
  }
}
```
- EN: Defines TableGen record `_post` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_post`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6332-6335
```tablegen
// Now actually declare all the contiguous load/stores, via those
// multiclasses. The instruction ids coming out of this are the bare
// names shown in the defm, with _pre or _post appended for writeback,
// e.g. MVE_VLDRBS16, MVE_VSTRB16_pre, MVE_VSTRHU16_post.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6337-6342
```tablegen
defm MVE_VLDRBS16: MVE_VLDRSTR_cw_m<MVE_ld, MVE_memB, "vldrb", "s16", 0, 0b01>;
defm MVE_VLDRBS32: MVE_VLDRSTR_cw_m<MVE_ld, MVE_memB, "vldrb", "s32", 0, 0b10>;
defm MVE_VLDRBU16: MVE_VLDRSTR_cw_m<MVE_ld, MVE_memB, "vldrb", "u16", 1, 0b01>;
defm MVE_VLDRBU32: MVE_VLDRSTR_cw_m<MVE_ld, MVE_memB, "vldrb", "u32", 1, 0b10>;
defm MVE_VLDRHS32: MVE_VLDRSTR_cw_m<MVE_ld, MVE_memH, "vldrh", "s32", 0, 0b10>;
defm MVE_VLDRHU32: MVE_VLDRSTR_cw_m<MVE_ld, MVE_memH, "vldrh", "u32", 1, 0b10>;
```
- EN: Defines TableGen record `MVE_VLDRBS16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VLDRBS16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6344-6346
```tablegen
defm MVE_VLDRBU8:  MVE_VLDRSTR_cs_m<MVE_ld, MVE_memB, "vldrb", "u8">;
defm MVE_VLDRHU16: MVE_VLDRSTR_cs_m<MVE_ld, MVE_memH, "vldrh", "u16">;
defm MVE_VLDRWU32: MVE_VLDRSTR_cs_m<MVE_ld, MVE_memW, "vldrw", "u32">;
```
- EN: Defines TableGen record `MVE_VLDRBU8:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VLDRBU8:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6348-6350
```tablegen
defm MVE_VSTRB16:  MVE_VLDRSTR_cw_m<MVE_st, MVE_memB, "vstrb", "16",  0, 0b01>;
defm MVE_VSTRB32:  MVE_VLDRSTR_cw_m<MVE_st, MVE_memB, "vstrb", "32",  0, 0b10>;
defm MVE_VSTRH32:  MVE_VLDRSTR_cw_m<MVE_st, MVE_memH, "vstrh", "32",  0, 0b10>;
```
- EN: Defines TableGen record `MVE_VSTRB16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSTRB16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6352-6354
```tablegen
defm MVE_VSTRBU8 : MVE_VLDRSTR_cs_m<MVE_st, MVE_memB, "vstrb", "8">;
defm MVE_VSTRHU16: MVE_VLDRSTR_cs_m<MVE_st, MVE_memH, "vstrh", "16">;
defm MVE_VSTRWU32: MVE_VLDRSTR_cs_m<MVE_st, MVE_memW, "vstrw", "32">;
```
- EN: Defines TableGen record `MVE_VSTRBU8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSTRBU8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6356-6372
```tablegen
// Gather loads / scatter stores whose address operand is of the form
// [Rn,Qm], i.e. a single GPR as the common base address, plus a
// vector of offset from it. ('Load/store this sequence of elements of
// the same array.')
//
// Like the contiguous family, these loads and stores can widen the
// loaded values / truncate the stored ones, or they can just
// load/store the same size of memory and vector lane. But unlike the
// contiguous family, there's no particular difference in encoding
// between those two cases.
//
// This family also comes with the option to scale the offset values
// in Qm by the size of the loaded memory (i.e. to treat them as array
// indices), or not to scale them (to treat them as plain byte offsets
// in memory, so that perhaps the loaded values are unaligned). The
// scaled instructions' address operand in assembly looks like
// [Rn,Qm,UXTW #2] or similar.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6374-6389
```tablegen
// Base class.
class MVE_VLDRSTR_rq<MVE_ldst_direction dir, MVE_memsz memsz, bit U,
                     bits<2> size, bit os, string asm, string suffix, int shift>
  : MVE_VLDRSTR_base<dir, U, 0b0, 0b0, 0, dir.Oops,
                     !con(dir.Iops, (ins mve_addr_rq_shift<shift>:$addr)),
                     asm, suffix, "$Qd, $addr", dir.cstr, size> {
  bits<7> addr;
  let Inst{23} = 0b1;
  let Inst{19-16} = addr{6-3};
  let Inst{8-7} = size;
  let Inst{6} = memsz.encoding{1};
  let Inst{5} = 0;
  let Inst{4} = memsz.encoding{0};
  let Inst{3-1} = addr{2-0};
  let Inst{0} = os;
}
```
- EN: Declares reusable TableGen class `MVE_VLDRSTR_rq` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VLDRSTR_rq`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6391-6400
```tablegen
// Multiclass that defines the scaled and unscaled versions of an
// instruction, when the memory size is wider than a byte. The scaled
// version gets the default name like MVE_VLDRBU16_rq; the unscaled /
// potentially unaligned version gets a "_u" suffix, e.g.
// MVE_VLDRBU16_rq_u.
multiclass MVE_VLDRSTR_rq_w<MVE_ldst_direction dir, MVE_memsz memsz,
                            string asm, string suffix, bit U, bits<2> size> {
  def _u : MVE_VLDRSTR_rq<dir, memsz, U, size, 0, asm, suffix, 0>;
  def "" : MVE_VLDRSTR_rq<dir, memsz, U, size, 1, asm, suffix, memsz.shift>;
}
```
- EN: Declares TableGen `multiclass MVE_VLDRSTR_rq_w`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VLDRSTR_rq_w`，它是一个可复用模板，可展开为多个相关记录。

### Lines 6402-6408
```tablegen
// Subclass of MVE_VLDRSTR_rq with the same API as that multiclass,
// for use when the memory size is one byte, so there's no 'scaled'
// version of the instruction at all. (This is encoded as if it were
// unscaled, but named in the default way with no _u suffix.)
class MVE_VLDRSTR_rq_b<MVE_ldst_direction dir, MVE_memsz memsz,
                       string asm, string suffix, bit U, bits<2> size>
  : MVE_VLDRSTR_rq<dir, memsz, U, size, 0, asm, suffix, 0>;
```
- EN: Declares reusable TableGen class `MVE_VLDRSTR_rq_b` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VLDRSTR_rq_b`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6410-6415
```tablegen
// Multiclasses wrapping that to add ISel patterns for intrinsics.
multiclass MVE_VLDR_rq_w<MVE_memsz memsz, list<MVEVectorVTInfo> VTIs> {
  defm "": MVE_VLDRSTR_rq_w<MVE_ld, memsz, "vldr" # memsz.MnemonicLetter,
                            VTIs[0].Suffix, VTIs[0].Unsigned, VTIs[0].Size>;
  defvar Inst = !cast<Instruction>(NAME);
  defvar InstU = !cast<Instruction>(NAME # "_u");
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 6417-6433
```tablegen
  foreach VTI = VTIs in
  foreach UnsignedFlag = !if(!eq(VTI.Size, memsz.encoding),
                             [0,1], [VTI.Unsigned]) in {
    def : Pat<(VTI.Vec (int_arm_mve_vldr_gather_offset GPR:$base, (VTIs[0].Vec MQPR:$offsets), memsz.TypeBits, 0, UnsignedFlag)),
              (VTI.Vec (InstU GPR:$base, MQPR:$offsets))>;
    def : Pat<(VTI.Vec (int_arm_mve_vldr_gather_offset GPR:$base, (VTIs[0].Vec MQPR:$offsets), memsz.TypeBits, memsz.shift, UnsignedFlag)),
              (VTI.Vec (Inst GPR:$base, MQPR:$offsets))>;
    def : Pat<(VTI.Vec (int_arm_mve_vldr_gather_offset_predicated GPR:$base, (VTIs[0].Vec MQPR:$offsets), memsz.TypeBits, 0, UnsignedFlag, (VTI.Pred VCCR:$pred))),
              (VTI.Vec (InstU GPR:$base, MQPR:$offsets, ARMVCCThen, VCCR:$pred, zero_reg))>;
    def : Pat<(VTI.Vec (int_arm_mve_vldr_gather_offset_predicated GPR:$base, (VTIs[0].Vec MQPR:$offsets), memsz.TypeBits, memsz.shift, UnsignedFlag, (VTI.Pred VCCR:$pred))),
              (VTI.Vec (Inst GPR:$base, MQPR:$offsets, ARMVCCThen, VCCR:$pred, zero_reg))>;
  }
}
multiclass MVE_VLDR_rq_b<list<MVEVectorVTInfo> VTIs> {
  def "": MVE_VLDRSTR_rq_b<MVE_ld, MVE_memB, "vldrb",
                           VTIs[0].Suffix, VTIs[0].Unsigned, VTIs[0].Size>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 6435-6446
```tablegen
  foreach VTI = VTIs in {
    def : Pat<(VTI.Vec (int_arm_mve_vldr_gather_offset GPR:$base, (VTIs[0].Vec MQPR:$offsets), 8, 0, VTI.Unsigned)),
              (VTI.Vec (Inst GPR:$base, MQPR:$offsets))>;
    def : Pat<(VTI.Vec (int_arm_mve_vldr_gather_offset_predicated GPR:$base, (VTIs[0].Vec MQPR:$offsets), 8, 0, VTI.Unsigned, (VTI.Pred VCCR:$pred))),
              (VTI.Vec (Inst GPR:$base, MQPR:$offsets, ARMVCCThen, VCCR:$pred, zero_reg))>;
  }
}
multiclass MVE_VSTR_rq_w<MVE_memsz memsz, list<MVEVectorVTInfo> VTIs> {
  defm "": MVE_VLDRSTR_rq_w<MVE_st, memsz, "vstr" # memsz.MnemonicLetter,
                            VTIs[0].BitsSuffix, 0, VTIs[0].Size>;
  defvar Inst = !cast<Instruction>(NAME);
  defvar InstU = !cast<Instruction>(NAME # "_u");
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 6448-6462
```tablegen
  foreach VTI = VTIs in {
    def : Pat<(int_arm_mve_vstr_scatter_offset GPR:$base, (VTIs[0].Vec MQPR:$offsets), (VTI.Vec MQPR:$data), memsz.TypeBits, 0),
              (InstU MQPR:$data, GPR:$base, MQPR:$offsets)>;
    def : Pat<(int_arm_mve_vstr_scatter_offset GPR:$base, (VTIs[0].Vec MQPR:$offsets), (VTI.Vec MQPR:$data), memsz.TypeBits, memsz.shift),
              (Inst MQPR:$data, GPR:$base, MQPR:$offsets)>;
    def : Pat<(int_arm_mve_vstr_scatter_offset_predicated GPR:$base, (VTIs[0].Vec MQPR:$offsets), (VTI.Vec MQPR:$data), memsz.TypeBits, 0, (VTI.Pred VCCR:$pred)),
              (InstU MQPR:$data, GPR:$base, MQPR:$offsets, ARMVCCThen, VCCR:$pred, zero_reg)>;
    def : Pat<(int_arm_mve_vstr_scatter_offset_predicated GPR:$base, (VTIs[0].Vec MQPR:$offsets), (VTI.Vec MQPR:$data), memsz.TypeBits, memsz.shift, (VTI.Pred VCCR:$pred)),
              (Inst MQPR:$data, GPR:$base, MQPR:$offsets, ARMVCCThen, VCCR:$pred, zero_reg)>;
  }
}
multiclass MVE_VSTR_rq_b<list<MVEVectorVTInfo> VTIs> {
  def "": MVE_VLDRSTR_rq_b<MVE_st, MVE_memB, "vstrb",
                           VTIs[0].BitsSuffix, 0, VTIs[0].Size>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 6464-6470
```tablegen
  foreach VTI = VTIs in {
    def : Pat<(int_arm_mve_vstr_scatter_offset GPR:$base, (VTIs[0].Vec MQPR:$offsets), (VTI.Vec MQPR:$data), 8, 0),
              (Inst MQPR:$data, GPR:$base, MQPR:$offsets)>;
    def : Pat<(int_arm_mve_vstr_scatter_offset_predicated GPR:$base, (VTIs[0].Vec MQPR:$offsets), (VTI.Vec MQPR:$data), 8, 0, (VTI.Pred VCCR:$pred)),
              (Inst MQPR:$data, GPR:$base, MQPR:$offsets, ARMVCCThen, VCCR:$pred, zero_reg)>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6472-6472
```tablegen
// Actually define all the loads and stores in this family.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6474-6478
```tablegen
defm MVE_VLDRBU8_rq : MVE_VLDR_rq_b<[MVE_v16u8,MVE_v16s8]>;
defm MVE_VLDRBU16_rq: MVE_VLDR_rq_b<[MVE_v8u16]>;
defm MVE_VLDRBS16_rq: MVE_VLDR_rq_b<[MVE_v8s16]>;
defm MVE_VLDRBU32_rq: MVE_VLDR_rq_b<[MVE_v4u32]>;
defm MVE_VLDRBS32_rq: MVE_VLDR_rq_b<[MVE_v4s32]>;
```
- EN: Defines TableGen record `MVE_VLDRBU8_rq` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VLDRBU8_rq`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6480-6484
```tablegen
defm MVE_VLDRHU16_rq: MVE_VLDR_rq_w<MVE_memH, [MVE_v8u16,MVE_v8s16,MVE_v8f16]>;
defm MVE_VLDRHU32_rq: MVE_VLDR_rq_w<MVE_memH, [MVE_v4u32]>;
defm MVE_VLDRHS32_rq: MVE_VLDR_rq_w<MVE_memH, [MVE_v4s32]>;
defm MVE_VLDRWU32_rq: MVE_VLDR_rq_w<MVE_memW, [MVE_v4u32,MVE_v4s32,MVE_v4f32]>;
defm MVE_VLDRDU64_rq: MVE_VLDR_rq_w<MVE_memD, [MVE_v2u64,MVE_v2s64]>;
```
- EN: Defines TableGen record `MVE_VLDRHU16_rq:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VLDRHU16_rq:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6486-6488
```tablegen
defm MVE_VSTRB8_rq  : MVE_VSTR_rq_b<[MVE_v16i8]>;
defm MVE_VSTRB16_rq : MVE_VSTR_rq_b<[MVE_v8i16]>;
defm MVE_VSTRB32_rq : MVE_VSTR_rq_b<[MVE_v4i32]>;
```
- EN: Defines TableGen record `MVE_VSTRB8_rq` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSTRB8_rq`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6490-6493
```tablegen
defm MVE_VSTRH16_rq : MVE_VSTR_rq_w<MVE_memH, [MVE_v8i16,MVE_v8f16]>;
defm MVE_VSTRH32_rq : MVE_VSTR_rq_w<MVE_memH, [MVE_v4i32]>;
defm MVE_VSTRW32_rq : MVE_VSTR_rq_w<MVE_memW, [MVE_v4i32,MVE_v4f32]>;
defm MVE_VSTRD64_rq : MVE_VSTR_rq_w<MVE_memD, [MVE_v2i64]>;
```
- EN: Defines TableGen record `MVE_VSTRH16_rq` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VSTRH16_rq`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6495-6505
```tablegen
// Gather loads / scatter stores whose address operand is of the form
// [Qm,#imm], i.e. a vector containing a full base address for each
// loaded item, plus an immediate offset applied consistently to all
// of them. ('Load/store the same field from this vector of pointers
// to a structure type.')
//
// This family requires the vector lane size to be at least 32 bits
// (so there's room for an address in each lane at all). It has no
// widening/narrowing variants. But it does support preindex
// writeback, in which the address vector is updated to hold the
// addresses actually loaded from.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6507-6520
```tablegen
// Base class.
class MVE_VLDRSTR_qi<MVE_ldst_direction dir, MVE_memsz memsz, bit W, dag wbops,
                     string asm, string wbAsm, string suffix, string cstr = "">
  : MVE_VLDRSTR_base<dir, 1, 1, W, 1, !con(wbops, dir.Oops),
                     !con(dir.Iops, (ins mve_addr_q_shift<memsz.shift>:$addr)),
                     asm, suffix, "$Qd, $addr" # wbAsm, cstr # dir.cstr, memsz.encoding> {
  bits<11> addr;
  let Inst{23} = addr{7};
  let Inst{19-17} = addr{10-8};
  let Inst{16} = 0;
  let Inst{8} = memsz.encoding{0}; // enough to distinguish 32- from 64-bit
  let Inst{7} = 0;
  let Inst{6-0} = addr{6-0};
}
```
- EN: Declares reusable TableGen class `MVE_VLDRSTR_qi` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VLDRSTR_qi`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6522-6530
```tablegen
// Multiclass that generates the non-writeback and writeback variants.
multiclass MVE_VLDRSTR_qi_m<MVE_ldst_direction dir, MVE_memsz memsz,
                            string asm, string suffix> {
  def ""   : MVE_VLDRSTR_qi<dir, memsz, 0, (outs),          asm, "",  suffix>;
  def _pre : MVE_VLDRSTR_qi<dir, memsz, 1, (outs MQPR:$wb), asm, "!", suffix,
                            "$addr.base = $wb"> {
    let DecoderMethod="DecodeMVE_MEM_3_pre<"#memsz.shift#">";
  }
}
```
- EN: Declares TableGen `multiclass MVE_VLDRSTR_qi_m`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_VLDRSTR_qi_m`，它是一个可复用模板，可展开为多个相关记录。

### Lines 6532-6540
```tablegen
// Multiclasses wrapping that one, adding selection patterns for the
// non-writeback loads and all the stores. (The writeback loads must
// deliver multiple output values, so they have to be selected by C++
// code.)
multiclass MVE_VLDR_qi<MVE_memsz memsz, MVEVectorVTInfo AVTI,
                       list<MVEVectorVTInfo> DVTIs> {
  defm "" : MVE_VLDRSTR_qi_m<MVE_ld, memsz, "vldr" # memsz.MnemonicLetter,
                             "u" # memsz.TypeBits>;
  defvar Inst = !cast<Instruction>(NAME);
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 6542-6557
```tablegen
  foreach DVTI = DVTIs in {
    def : Pat<(DVTI.Vec (int_arm_mve_vldr_gather_base
                 (AVTI.Vec MQPR:$addr), (i32 imm:$offset))),
              (DVTI.Vec (Inst (AVTI.Vec MQPR:$addr), (i32 imm:$offset)))>;
    def : Pat<(DVTI.Vec (int_arm_mve_vldr_gather_base_predicated
                 (AVTI.Vec MQPR:$addr), (i32 imm:$offset), (AVTI.Pred VCCR:$pred))),
              (DVTI.Vec (Inst (AVTI.Vec MQPR:$addr), (i32 imm:$offset),
                        ARMVCCThen, VCCR:$pred, zero_reg))>;
  }
}
multiclass MVE_VSTR_qi<MVE_memsz memsz, MVEVectorVTInfo AVTI,
                       list<MVEVectorVTInfo> DVTIs> {
  defm "" : MVE_VLDRSTR_qi_m<MVE_st, memsz, "vstr" # memsz.MnemonicLetter,
                             !cast<string>(memsz.TypeBits)>;
  defvar Inst = !cast<Instruction>(NAME);
  defvar InstPre = !cast<Instruction>(NAME # "_pre");
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 6559-6576
```tablegen
  foreach DVTI = DVTIs in {
    def : Pat<(int_arm_mve_vstr_scatter_base
                (AVTI.Vec MQPR:$addr), (i32 imm:$offset), (DVTI.Vec MQPR:$data)),
              (Inst (DVTI.Vec MQPR:$data), (AVTI.Vec MQPR:$addr),
                    (i32 imm:$offset))>;
    def : Pat<(int_arm_mve_vstr_scatter_base_predicated
                (AVTI.Vec MQPR:$addr), (i32 imm:$offset), (DVTI.Vec MQPR:$data), (AVTI.Pred VCCR:$pred)),
              (Inst (DVTI.Vec MQPR:$data), (AVTI.Vec MQPR:$addr),
                    (i32 imm:$offset), ARMVCCThen, VCCR:$pred, zero_reg)>;
    def : Pat<(AVTI.Vec (int_arm_mve_vstr_scatter_base_wb
                (AVTI.Vec MQPR:$addr), (i32 imm:$offset), (DVTI.Vec MQPR:$data))),
              (AVTI.Vec (InstPre (DVTI.Vec MQPR:$data), (AVTI.Vec MQPR:$addr),
                                 (i32 imm:$offset)))>;
    def : Pat<(AVTI.Vec (int_arm_mve_vstr_scatter_base_wb_predicated
                (AVTI.Vec MQPR:$addr), (i32 imm:$offset), (DVTI.Vec MQPR:$data), (AVTI.Pred VCCR:$pred))),
              (AVTI.Vec (InstPre (DVTI.Vec MQPR:$data), (AVTI.Vec MQPR:$addr),
                                 (i32 imm:$offset), ARMVCCThen, VCCR:$pred, zero_reg))>;
  }
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6577-6577
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6579-6583
```tablegen
// Actual instruction definitions.
defm MVE_VLDRWU32_qi: MVE_VLDR_qi<MVE_memW, MVE_v4i32, [MVE_v4i32,MVE_v4f32]>;
defm MVE_VLDRDU64_qi: MVE_VLDR_qi<MVE_memD, MVE_v2i64, [MVE_v2i64,MVE_v2f64]>;
defm MVE_VSTRW32_qi:  MVE_VSTR_qi<MVE_memW, MVE_v4i32, [MVE_v4i32,MVE_v4f32]>;
defm MVE_VSTRD64_qi:  MVE_VSTR_qi<MVE_memD, MVE_v2i64, [MVE_v2i64,MVE_v2f64]>;
```
- EN: Defines TableGen record `MVE_VLDRWU32_qi:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VLDRWU32_qi:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6585-6596
```tablegen
// Define aliases for all the instructions where memory size and
// vector lane size are the same. These are mnemonic aliases, so they
// apply consistently across all of the above families - contiguous
// loads, and both the rq and qi types of gather/scatter.
//
// Rationale: As long as you're loading (for example) 16-bit memory
// values into 16-bit vector lanes, you can think of them as signed or
// unsigned integers, fp16 or just raw 16-bit blobs and it makes no
// difference. So we permit all of vldrh.16, vldrh.u16, vldrh.s16,
// vldrh.f16 and treat them all as equivalent to the canonical
// spelling (which happens to be .u16 for loads, and just .16 for
// stores).
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6598-6603
```tablegen
foreach vpt_cond = ["", "t", "e"] in
foreach memsz = [MVE_memB, MVE_memH, MVE_memW, MVE_memD] in
foreach suffix = memsz.suffixes in {
  // Define an alias with every suffix in the list, except for the one
  // used by the real Instruction record (i.e. the one that all the
  // rest are aliases *for*).
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6605-6609
```tablegen
  if !ne(suffix, memsz.CanonLoadSuffix) then {
    def : MnemonicAlias<
      "vldr" # memsz.MnemonicLetter # vpt_cond # suffix,
      "vldr" # memsz.MnemonicLetter # vpt_cond # memsz.CanonLoadSuffix>;
  }
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6611-6616
```tablegen
  if !ne(suffix, memsz.CanonStoreSuffix) then {
    def : MnemonicAlias<
      "vstr" # memsz.MnemonicLetter # vpt_cond # suffix,
      "vstr" # memsz.MnemonicLetter # vpt_cond # memsz.CanonStoreSuffix>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6618-6618
```tablegen
// end of MVE predicable load/store
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6620-6624
```tablegen
class MVE_VPT<string suffix, bits<2> size, dag iops, string asm, list<dag> pattern=[]>
  : MVE_MI<(outs ), iops, NoItinerary, !strconcat("vpt", "${Mk}", ".", suffix), asm, "", size, pattern> {
  bits<3> fc;
  bits<4> Mk;
  bits<3> Qn;
```
- EN: Declares reusable TableGen class `MVE_VPT` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VPT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6626-6635
```tablegen
  let Inst{31-23} = 0b111111100;
  let Inst{22} = Mk{3};
  let Inst{21-20} = size;
  let Inst{19-17} = Qn{2-0};
  let Inst{16} = 0b1;
  let Inst{15-13} = Mk{2-0};
  let Inst{12} = fc{2};
  let Inst{11-8} = 0b1111;
  let Inst{7} = fc{0};
  let Inst{4} = 0b0;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6637-6639
```tablegen
  let Defs = [VPR];
  let validForTailPredication=1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6641-6644
```tablegen
class MVE_VPTt1<string suffix, bits<2> size, dag iops>
  : MVE_VPT<suffix, size, iops, "$fc, $Qn, $Qm"> {
  bits<4> Qm;
  bits<4> Mk;
```
- EN: Declares reusable TableGen class `MVE_VPTt1` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VPTt1`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6646-6650
```tablegen
  let Inst{6} = 0b0;
  let Inst{5} = Qm{3};
  let Inst{3-1} = Qm{2-0};
  let Inst{0} = fc{1};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6652-6657
```tablegen
class MVE_VPTt1i<string suffix, bits<2> size>
 : MVE_VPTt1<suffix, size,
           (ins vpt_mask:$Mk, MQPR:$Qn, MQPR:$Qm, pred_basic_i:$fc)> {
  let Inst{12} = 0b0;
  let Inst{0} = 0b0;
}
```
- EN: Declares reusable TableGen class `MVE_VPTt1i` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VPTt1i`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6659-6661
```tablegen
def MVE_VPTv4i32 : MVE_VPTt1i<"i32", 0b10>;
def MVE_VPTv8i16 : MVE_VPTt1i<"i16", 0b01>;
def MVE_VPTv16i8 : MVE_VPTt1i<"i8", 0b00>;
```
- EN: Defines TableGen record `MVE_VPTv4i32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VPTv4i32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6663-6668
```tablegen
class MVE_VPTt1u<string suffix, bits<2> size>
 : MVE_VPTt1<suffix, size,
           (ins vpt_mask:$Mk, MQPR:$Qn, MQPR:$Qm, pred_basic_u:$fc)> {
  let Inst{12} = 0b0;
  let Inst{0} = 0b1;
}
```
- EN: Declares reusable TableGen class `MVE_VPTt1u` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VPTt1u`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6670-6672
```tablegen
def MVE_VPTv4u32 : MVE_VPTt1u<"u32", 0b10>;
def MVE_VPTv8u16 : MVE_VPTt1u<"u16", 0b01>;
def MVE_VPTv16u8 : MVE_VPTt1u<"u8", 0b00>;
```
- EN: Defines TableGen record `MVE_VPTv4u32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VPTv4u32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6674-6678
```tablegen
class MVE_VPTt1s<string suffix, bits<2> size>
 : MVE_VPTt1<suffix, size,
           (ins vpt_mask:$Mk, MQPR:$Qn, MQPR:$Qm, pred_basic_s:$fc)> {
  let Inst{12} = 0b1;
}
```
- EN: Declares reusable TableGen class `MVE_VPTt1s` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VPTt1s`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6680-6682
```tablegen
def MVE_VPTv4s32 : MVE_VPTt1s<"s32", 0b10>;
def MVE_VPTv8s16 : MVE_VPTt1s<"s16", 0b01>;
def MVE_VPTv16s8 : MVE_VPTt1s<"s8", 0b00>;
```
- EN: Defines TableGen record `MVE_VPTv4s32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VPTv4s32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6684-6689
```tablegen
class MVE_VPTt2<string suffix, bits<2> size, dag iops>
  : MVE_VPT<suffix, size, iops,
          "$fc, $Qn, $Rm"> {
  bits<4> Rm;
  bits<3> fc;
  bits<4> Mk;
```
- EN: Declares reusable TableGen class `MVE_VPTt2` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VPTt2`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6691-6694
```tablegen
  let Inst{6} = 0b1;
  let Inst{5} = fc{1};
  let Inst{3-0} = Rm{3-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6696-6701
```tablegen
class MVE_VPTt2i<string suffix, bits<2> size>
  : MVE_VPTt2<suffix, size,
            (ins vpt_mask:$Mk, MQPR:$Qn, GPRwithZR:$Rm, pred_basic_i:$fc)> {
  let Inst{12} = 0b0;
  let Inst{5} = 0b0;
}
```
- EN: Declares reusable TableGen class `MVE_VPTt2i` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VPTt2i`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6703-6705
```tablegen
def MVE_VPTv4i32r : MVE_VPTt2i<"i32", 0b10>;
def MVE_VPTv8i16r : MVE_VPTt2i<"i16", 0b01>;
def MVE_VPTv16i8r : MVE_VPTt2i<"i8", 0b00>;
```
- EN: Defines TableGen record `MVE_VPTv4i32r` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VPTv4i32r`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6707-6712
```tablegen
class MVE_VPTt2u<string suffix, bits<2> size>
  : MVE_VPTt2<suffix, size,
            (ins vpt_mask:$Mk, MQPR:$Qn, GPRwithZR:$Rm, pred_basic_u:$fc)> {
  let Inst{12} = 0b0;
  let Inst{5} = 0b1;
}
```
- EN: Declares reusable TableGen class `MVE_VPTt2u` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VPTt2u`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6714-6716
```tablegen
def MVE_VPTv4u32r : MVE_VPTt2u<"u32", 0b10>;
def MVE_VPTv8u16r : MVE_VPTt2u<"u16", 0b01>;
def MVE_VPTv16u8r : MVE_VPTt2u<"u8", 0b00>;
```
- EN: Defines TableGen record `MVE_VPTv4u32r` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VPTv4u32r`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6718-6722
```tablegen
class MVE_VPTt2s<string suffix, bits<2> size>
  : MVE_VPTt2<suffix, size,
            (ins vpt_mask:$Mk, MQPR:$Qn, GPRwithZR:$Rm, pred_basic_s:$fc)> {
  let Inst{12} = 0b1;
}
```
- EN: Declares reusable TableGen class `MVE_VPTt2s` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VPTt2s`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6724-6726
```tablegen
def MVE_VPTv4s32r : MVE_VPTt2s<"s32", 0b10>;
def MVE_VPTv8s16r : MVE_VPTt2s<"s16", 0b01>;
def MVE_VPTv16s8r : MVE_VPTt2s<"s8", 0b00>;
```
- EN: Defines TableGen record `MVE_VPTv4s32r` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VPTv4s32r`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6729-6734
```tablegen
class MVE_VPTf<string suffix, bit size, dag iops, string asm, list<dag> pattern=[]>
  : MVE_MI<(outs ), iops, NoItinerary, !strconcat("vpt", "${Mk}", ".", suffix), asm,
            "", !if(size, 0b01, 0b10), pattern> {
  bits<3> fc;
  bits<4> Mk;
  bits<3> Qn;
```
- EN: Declares reusable TableGen class `MVE_VPTf` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VPTf`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6736-6747
```tablegen
  let Inst{31-29} = 0b111;
  let Inst{28} = size;
  let Inst{27-23} = 0b11100;
  let Inst{22} = Mk{3};
  let Inst{21-20} = 0b11;
  let Inst{19-17} = Qn{2-0};
  let Inst{16} = 0b1;
  let Inst{15-13} = Mk{2-0};
  let Inst{12} = fc{2};
  let Inst{11-8} = 0b1111;
  let Inst{7} = fc{0};
  let Inst{4} = 0b0;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6749-6752
```tablegen
  let Defs = [VPR];
  let Predicates = [HasMVEFloat];
  let validForTailPredication=1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6754-6758
```tablegen
class MVE_VPTft1<string suffix, bit size>
  : MVE_VPTf<suffix, size, (ins vpt_mask:$Mk, MQPR:$Qn, MQPR:$Qm, pred_basic_fp:$fc),
          "$fc, $Qn, $Qm"> {
  bits<3> fc;
  bits<4> Qm;
```
- EN: Declares reusable TableGen class `MVE_VPTft1` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VPTft1`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6760-6764
```tablegen
  let Inst{6} = 0b0;
  let Inst{5} = Qm{3};
  let Inst{3-1} = Qm{2-0};
  let Inst{0} = fc{1};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6766-6767
```tablegen
def MVE_VPTv4f32         : MVE_VPTft1<"f32", 0b0>;
def MVE_VPTv8f16         : MVE_VPTft1<"f16", 0b1>;
```
- EN: Defines TableGen record `MVE_VPTv4f32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VPTv4f32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6769-6773
```tablegen
class MVE_VPTft2<string suffix, bit size>
  : MVE_VPTf<suffix, size, (ins vpt_mask:$Mk, MQPR:$Qn, GPRwithZR:$Rm, pred_basic_fp:$fc),
          "$fc, $Qn, $Rm"> {
  bits<3> fc;
  bits<4> Rm;
```
- EN: Declares reusable TableGen class `MVE_VPTft2` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_VPTft2`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6775-6778
```tablegen
  let Inst{6} = 0b1;
  let Inst{5} = fc{1};
  let Inst{3-0} = Rm{3-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6780-6781
```tablegen
def MVE_VPTv4f32r        : MVE_VPTft2<"f32", 0b0>;
def MVE_VPTv8f16r        : MVE_VPTft2<"f16", 0b1>;
```
- EN: Defines TableGen record `MVE_VPTv4f32r` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VPTv4f32r`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6783-6785
```tablegen
def MVE_VPST : MVE_MI<(outs ), (ins vpt_mask:$Mk), NoItinerary,
       !strconcat("vpst", "${Mk}"), "", "", 0b00, []> {
  bits<4> Mk;
```
- EN: Defines TableGen record `MVE_VPST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VPST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6787-6794
```tablegen
  let Inst{31-23} = 0b111111100;
  let Inst{22} = Mk{3};
  let Inst{21-16} = 0b110001;
  let Inst{15-13} = Mk{2-0};
  let Inst{12-0} = 0b0111101001101;
  let Unpredictable{12} = 0b1;
  let Unpredictable{7} = 0b1;
  let Unpredictable{5} = 0b1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6796-6798
```tablegen
  let Uses = [VPR];
  let validForTailPredication = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6800-6804
```tablegen
def MVE_VPSEL : MVE_p<(outs MQPR:$Qd), (ins MQPR:$Qn, MQPR:$Qm), NoItinerary,
                      "vpsel", "", "$Qd, $Qn, $Qm", vpred_n, "", 0b00, []> {
  bits<4> Qn;
  bits<4> Qd;
  bits<4> Qm;
```
- EN: Defines TableGen record `MVE_VPSEL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VPSEL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6806-6821
```tablegen
  let Inst{28} = 0b1;
  let Inst{25-23} = 0b100;
  let Inst{22} = Qd{3};
  let Inst{21-20} = 0b11;
  let Inst{19-17} = Qn{2-0};
  let Inst{16} = 0b1;
  let Inst{15-13} = Qd{2-0};
  let Inst{12-9} = 0b0111;
  let Inst{8} = 0b1;
  let Inst{7} = Qn{3};
  let Inst{6} = 0b0;
  let Inst{5} = Qm{3};
  let Inst{4} = 0b0;
  let Inst{3-1} = Qm{2-0};
  let Inst{0} = 0b1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6823-6826
```tablegen
foreach suffix = ["s8", "s16", "s32", "u8", "u16", "u32",
                  "i8", "i16", "i32",       "f16", "f32"] in
def : MVEInstAlias<"vpsel${vp}." # suffix # "\t$Qd, $Qn, $Qm",
                   (MVE_VPSEL MQPR:$Qd, MQPR:$Qn, MQPR:$Qm, vpred_n:$vp)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6828-6836
```tablegen
let Predicates = [HasMVEInt] in {
  def : Pat<(v16i8 (vselect (v16i1 VCCR:$pred), (v16i8 MQPR:$v1), (v16i8 MQPR:$v2))),
            (v16i8 (MVE_VPSEL MQPR:$v1, MQPR:$v2, ARMVCCNone, VCCR:$pred, zero_reg))>;
  def : Pat<(v8i16 (vselect (v8i1 VCCR:$pred), (v8i16 MQPR:$v1), (v8i16 MQPR:$v2))),
            (v8i16 (MVE_VPSEL MQPR:$v1, MQPR:$v2, ARMVCCNone, VCCR:$pred, zero_reg))>;
  def : Pat<(v4i32 (vselect (v4i1 VCCR:$pred), (v4i32 MQPR:$v1), (v4i32 MQPR:$v2))),
            (v4i32 (MVE_VPSEL MQPR:$v1, MQPR:$v2, ARMVCCNone, VCCR:$pred, zero_reg))>;
  def : Pat<(v2i64 (vselect (v2i1 VCCR:$pred), (v2i64 MQPR:$v1), (v2i64 MQPR:$v2))),
            (v2i64 (MVE_VPSEL MQPR:$v1, MQPR:$v2, ARMVCCNone, VCCR:$pred, zero_reg))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6838-6843
```tablegen
  def : Pat<(v8f16 (vselect (v8i1 VCCR:$pred), (v8f16 MQPR:$v1), (v8f16 MQPR:$v2))),
            (v8f16 (MVE_VPSEL MQPR:$v1, MQPR:$v2, ARMVCCNone, VCCR:$pred, zero_reg))>;
  def : Pat<(v4f32 (vselect (v4i1 VCCR:$pred), (v4f32 MQPR:$v1), (v4f32 MQPR:$v2))),
            (v4f32 (MVE_VPSEL MQPR:$v1, MQPR:$v2, ARMVCCNone, VCCR:$pred, zero_reg))>;
  def : Pat<(v2f64 (vselect (v2i1 VCCR:$pred), (v2f64 MQPR:$v1), (v2f64 MQPR:$v2))),
            (v2f64 (MVE_VPSEL MQPR:$v1, MQPR:$v2, ARMVCCNone, VCCR:$pred, zero_reg))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6845-6853
```tablegen
  def : Pat<(v16i8 (vselect (v16i8 MQPR:$pred), (v16i8 MQPR:$v1), (v16i8 MQPR:$v2))),
            (v16i8 (MVE_VPSEL MQPR:$v1, MQPR:$v2, ARMVCCNone,
                              (MVE_VCMPi8 (v16i8 MQPR:$pred), (MVE_VMOVimmi8 0), ARMCCne), zero_reg))>;
  def : Pat<(v8i16 (vselect (v8i16 MQPR:$pred), (v8i16 MQPR:$v1), (v8i16 MQPR:$v2))),
            (v8i16 (MVE_VPSEL MQPR:$v1, MQPR:$v2, ARMVCCNone,
                              (MVE_VCMPi16 (v8i16 MQPR:$pred), (MVE_VMOVimmi16 0), ARMCCne), zero_reg))>;
  def : Pat<(v4i32 (vselect (v4i32 MQPR:$pred), (v4i32 MQPR:$v1), (v4i32 MQPR:$v2))),
            (v4i32 (MVE_VPSEL MQPR:$v1, MQPR:$v2, ARMVCCNone,
                              (MVE_VCMPi32 (v4i32 MQPR:$pred), (MVE_VMOVimmi32 0), ARMCCne), zero_reg))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6855-6860
```tablegen
  def : Pat<(v8f16 (vselect (v8i16 MQPR:$pred), (v8f16 MQPR:$v1), (v8f16 MQPR:$v2))),
            (v8f16 (MVE_VPSEL MQPR:$v1, MQPR:$v2, ARMVCCNone,
                              (MVE_VCMPi16 (v8i16 MQPR:$pred), (MVE_VMOVimmi16 0), ARMCCne), zero_reg))>;
  def : Pat<(v4f32 (vselect (v4i32 MQPR:$pred), (v4f32 MQPR:$v1), (v4f32 MQPR:$v2))),
            (v4f32 (MVE_VPSEL MQPR:$v1, MQPR:$v2, ARMVCCNone,
                              (MVE_VCMPi32 (v4i32 MQPR:$pred), (MVE_VMOVimmi32 0), ARMCCne), zero_reg))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6862-6870
```tablegen
  // Pred <-> Int
  def : Pat<(v16i8 (zext  (v16i1 VCCR:$pred))),
            (v16i8 (MVE_VPSEL (MVE_VMOVimmi8 1), (MVE_VMOVimmi8 0), ARMVCCNone, VCCR:$pred, zero_reg))>;
  def : Pat<(v8i16 (zext  (v8i1  VCCR:$pred))),
            (v8i16 (MVE_VPSEL (MVE_VMOVimmi16 1), (MVE_VMOVimmi16 0), ARMVCCNone, VCCR:$pred, zero_reg))>;
  def : Pat<(v4i32 (zext  (v4i1  VCCR:$pred))),
            (v4i32 (MVE_VPSEL (MVE_VMOVimmi32 1), (MVE_VMOVimmi32 0), ARMVCCNone, VCCR:$pred, zero_reg))>;
  def : Pat<(v2i64 (zext  (v2i1  VCCR:$pred))),
            (v2i64 (MVE_VPSEL (MVE_VMOVimmi64 1), (MVE_VMOVimmi32 0), ARMVCCNone, VCCR:$pred, zero_reg))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6872-6879
```tablegen
  def : Pat<(v16i8 (sext  (v16i1 VCCR:$pred))),
            (v16i8 (MVE_VPSEL (MVE_VMOVimmi8 255), (MVE_VMOVimmi8 0), ARMVCCNone, VCCR:$pred, zero_reg))>;
  def : Pat<(v8i16 (sext  (v8i1  VCCR:$pred))),
            (v8i16 (MVE_VPSEL (MVE_VMOVimmi8 255), (MVE_VMOVimmi16 0), ARMVCCNone, VCCR:$pred, zero_reg))>;
  def : Pat<(v4i32 (sext  (v4i1  VCCR:$pred))),
            (v4i32 (MVE_VPSEL (MVE_VMOVimmi8 255), (MVE_VMOVimmi32 0), ARMVCCNone, VCCR:$pred, zero_reg))>;
  def : Pat<(v2i64 (sext  (v2i1  VCCR:$pred))),
            (v2i64 (MVE_VPSEL (MVE_VMOVimmi8 255), (MVE_VMOVimmi32 0), ARMVCCNone, VCCR:$pred, zero_reg))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6881-6889
```tablegen
  def : Pat<(v16i8 (anyext  (v16i1 VCCR:$pred))),
            (v16i8 (MVE_VPSEL (MVE_VMOVimmi8 1), (MVE_VMOVimmi8 0), ARMVCCNone, VCCR:$pred, zero_reg))>;
  def : Pat<(v8i16 (anyext  (v8i1  VCCR:$pred))),
            (v8i16 (MVE_VPSEL (MVE_VMOVimmi16 1), (MVE_VMOVimmi16 0), ARMVCCNone, VCCR:$pred, zero_reg))>;
  def : Pat<(v4i32 (anyext  (v4i1  VCCR:$pred))),
            (v4i32 (MVE_VPSEL (MVE_VMOVimmi32 1), (MVE_VMOVimmi32 0), ARMVCCNone, VCCR:$pred, zero_reg))>;
  def : Pat<(v2i64 (anyext  (v2i1  VCCR:$pred))),
            (v2i64 (MVE_VPSEL (MVE_VMOVimmi64 1), (MVE_VMOVimmi32 0), ARMVCCNone, VCCR:$pred, zero_reg))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6891-6904
```tablegen
let Predicates = [HasMVEFloat] in {
  // Pred <-> Float
  // 112 is 1.0 in float
  def : Pat<(v4f32 (uint_to_fp (v4i1 VCCR:$pred))),
            (v4f32 (MVE_VPSEL (v4f32 (MVE_VMOVimmf32 112)), (v4f32 (MVE_VMOVimmi32 0)), ARMVCCNone, VCCR:$pred, zero_reg))>;
  // 2620 in 1.0 in half
  def : Pat<(v8f16 (uint_to_fp (v8i1 VCCR:$pred))),
            (v8f16 (MVE_VPSEL (v8f16 (MVE_VMOVimmi16 2620)), (v8f16 (MVE_VMOVimmi16 0)), ARMVCCNone, VCCR:$pred, zero_reg))>;
  // 240 is -1.0 in float
  def : Pat<(v4f32 (sint_to_fp (v4i1 VCCR:$pred))),
            (v4f32 (MVE_VPSEL (v4f32 (MVE_VMOVimmf32 240)), (v4f32 (MVE_VMOVimmi32 0)), ARMVCCNone, VCCR:$pred, zero_reg))>;
  // 2748 is -1.0 in half
  def : Pat<(v8f16 (sint_to_fp (v8i1 VCCR:$pred))),
            (v8f16 (MVE_VPSEL (v8f16 (MVE_VMOVimmi16 2748)), (v8f16 (MVE_VMOVimmi16 0)), ARMVCCNone, VCCR:$pred, zero_reg))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6906-6914
```tablegen
  def : Pat<(v4i1 (fp_to_uint (v4f32 MQPR:$v1))),
            (v4i1 (MVE_VCMPf32r (v4f32 MQPR:$v1), ZR, ARMCCne))>;
  def : Pat<(v8i1 (fp_to_uint (v8f16 MQPR:$v1))),
            (v8i1 (MVE_VCMPf16r (v8f16 MQPR:$v1), ZR, ARMCCne))>;
  def : Pat<(v4i1 (fp_to_sint (v4f32 MQPR:$v1))),
            (v4i1 (MVE_VCMPf32r (v4f32 MQPR:$v1), ZR, ARMCCne))>;
  def : Pat<(v8i1 (fp_to_sint (v8f16 MQPR:$v1))),
            (v8i1 (MVE_VCMPf16r (v8f16 MQPR:$v1), ZR, ARMCCne))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6916-6922
```tablegen
def MVE_VPNOT : MVE_p<(outs VCCR:$P0), (ins VCCR:$P0_in), NoItinerary,
                      "vpnot", "", "", vpred_n, "", 0b00, []> {
  let Inst{31-0} = 0b11111110001100010000111101001101;
  let Unpredictable{19-17} = 0b111;
  let Unpredictable{12} = 0b1;
  let Unpredictable{7} = 0b1;
  let Unpredictable{5} = 0b1;
```
- EN: Defines TableGen record `MVE_VPNOT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VPNOT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6924-6926
```tablegen
  let Constraints = "";
  let DecoderMethod = "DecodeMVEVPNOT";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6928-6937
```tablegen
let Predicates = [HasMVEInt] in {
  def : Pat<(v2i1 (xor (v2i1 VCCR:$pred), (v2i1 (predicate_cast (i32 65535))))),
            (v2i1 (MVE_VPNOT (v2i1 VCCR:$pred)))>;
  def : Pat<(v4i1 (xor (v4i1 VCCR:$pred), (v4i1 (predicate_cast (i32 65535))))),
            (v4i1 (MVE_VPNOT (v4i1 VCCR:$pred)))>;
  def : Pat<(v8i1 (xor (v8i1 VCCR:$pred), (v8i1 (predicate_cast (i32 65535))))),
            (v8i1 (MVE_VPNOT (v8i1 VCCR:$pred)))>;
  def : Pat<(v16i1 (xor (v16i1 VCCR:$pred), (v16i1 (predicate_cast (i32 65535))))),
            (v16i1 (MVE_VPNOT (v16i1 VCCR:$pred)))>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6940-6948
```tablegen
class MVE_loltp_start<dag iops, string asm, string ops, bits<2> size>
  : t2LOL<(outs GPRlr:$LR), iops, asm, ops> {
  bits<4> Rn;
  let Predicates = [HasMVEInt];
  let Inst{22} = 0b0;
  let Inst{21-20} = size;
  let Inst{19-16} = Rn{3-0};
  let Inst{12} = 0b0;
}
```
- EN: Declares reusable TableGen class `MVE_loltp_start` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_loltp_start`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6950-6955
```tablegen
class MVE_DLSTP<string asm, bits<2> size>
  : MVE_loltp_start<(ins rGPR:$Rn), asm, "$LR, $Rn", size> {
  let Inst{13} = 0b1;
  let Inst{11-1} = 0b00000000000;
  let Unpredictable{10-1} = 0b1111111111;
}
```
- EN: Declares reusable TableGen class `MVE_DLSTP` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_DLSTP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6957-6966
```tablegen
class MVE_WLSTP<string asm, bits<2> size>
  : MVE_loltp_start<(ins rGPR:$Rn, wlslabel_u11:$label),
                    asm, "$LR, $Rn, $label", size> {
  bits<11> label;
  let Inst{13} = 0b0;
  let Inst{11} = label{0};
  let Inst{10-1} = label{10-1};
  let isBranch = 1;
  let isTerminator = 1;
}
```
- EN: Declares reusable TableGen class `MVE_WLSTP` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_WLSTP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6968-6969
```tablegen
def SDT_MVEMEMCPYLOOPNODE
    : SDTypeProfile<0, 3, [SDTCisPtrTy<0>, SDTCisPtrTy<1>, SDTCisVT<2, i32>]>;
```
- EN: Defines TableGen record `SDT_MVEMEMCPYLOOPNODE` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_MVEMEMCPYLOOPNODE`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 6971-6974
```tablegen
// Pseudo-instruction representing a memory copy using a tail predicated
// loop
def MVE_MEMCPYLOOPNODE : SDNode<"ARMISD::MEMCPYLOOP", SDT_MVEMEMCPYLOOPNODE,
                                [SDNPHasChain, SDNPMayStore, SDNPMayLoad]>;
```
- EN: Defines TableGen record `MVE_MEMCPYLOOPNODE` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MVE_MEMCPYLOOPNODE`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 6976-6981
```tablegen
let usesCustomInserter = 1, hasNoSchedulingInfo = 1, Defs = [CPSR] in {
  def MVE_MEMCPYLOOPINST : PseudoInst<(outs),
        (ins rGPR:$dst, rGPR:$src, rGPR:$sz),
        NoItinerary,
        [(MVE_MEMCPYLOOPNODE rGPR:$dst, rGPR:$src, rGPR:$sz)]>;
}
```
- EN: Defines TableGen record `MVE_MEMCPYLOOPINST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_MEMCPYLOOPINST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6983-6984
```tablegen
def SDT_MVEMEMSETLOOPNODE
    : SDTypeProfile<0, 3, [SDTCisPtrTy<0>, SDTCisVT<1, v16i8>, SDTCisVT<2, i32>]>;
```
- EN: Defines TableGen record `SDT_MVEMEMSETLOOPNODE` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_MVEMEMSETLOOPNODE`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 6986-6989
```tablegen
// Pseudo-instruction representing a memset using a tail predicated
// loop
def MVE_MEMSETLOOPNODE : SDNode<"ARMISD::MEMSETLOOP", SDT_MVEMEMSETLOOPNODE,
                                [SDNPHasChain, SDNPMayStore, SDNPMayLoad]>;
```
- EN: Defines TableGen record `MVE_MEMSETLOOPNODE` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MVE_MEMSETLOOPNODE`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 6991-6996
```tablegen
let usesCustomInserter = 1, hasNoSchedulingInfo = 1, Defs = [CPSR] in {
  def MVE_MEMSETLOOPINST : PseudoInst<(outs),
        (ins rGPR:$dst, MQPR:$src, rGPR:$sz),
        NoItinerary,
        [(MVE_MEMSETLOOPNODE rGPR:$dst, MQPR:$src, rGPR:$sz)]>;
}
```
- EN: Defines TableGen record `MVE_MEMSETLOOPINST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_MEMSETLOOPINST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6998-7001
```tablegen
def MVE_DLSTP_8  : MVE_DLSTP<"dlstp.8",  0b00>;
def MVE_DLSTP_16 : MVE_DLSTP<"dlstp.16", 0b01>;
def MVE_DLSTP_32 : MVE_DLSTP<"dlstp.32", 0b10>;
def MVE_DLSTP_64 : MVE_DLSTP<"dlstp.64", 0b11>;
```
- EN: Defines TableGen record `MVE_DLSTP_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_DLSTP_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7003-7006
```tablegen
def MVE_WLSTP_8  : MVE_WLSTP<"wlstp.8",  0b00>;
def MVE_WLSTP_16 : MVE_WLSTP<"wlstp.16", 0b01>;
def MVE_WLSTP_32 : MVE_WLSTP<"wlstp.32", 0b10>;
def MVE_WLSTP_64 : MVE_WLSTP<"wlstp.64", 0b11>;
```
- EN: Defines TableGen record `MVE_WLSTP_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_WLSTP_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7008-7014
```tablegen
class MVE_loltp_end<dag oops, dag iops, string asm, string ops>
  : t2LOL<oops, iops, asm, ops> {
  let Predicates = [HasMVEInt];
  let Inst{22-21} = 0b00;
  let Inst{19-16} = 0b1111;
  let Inst{12} = 0b0;
}
```
- EN: Declares reusable TableGen class `MVE_loltp_end` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_loltp_end`，通常用于抽象共享字段、谓词或编码结构。

### Lines 7016-7026
```tablegen
def MVE_LETP : MVE_loltp_end<(outs GPRlr:$LRout),
                             (ins GPRlr:$LRin, lelabel_u11:$label),
                             "letp", "$LRin, $label"> {
  bits<11> label;
  let Inst{20} = 0b1;
  let Inst{13} = 0b0;
  let Inst{11} = label{0};
  let Inst{10-1} = label{10-1};
  let isBranch = 1;
  let isTerminator = 1;
}
```
- EN: Defines TableGen record `MVE_LETP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_LETP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7028-7034
```tablegen
def MVE_LCTP : MVE_loltp_end<(outs), (ins pred:$p), "lctp${p}", ""> {
  let Inst{20} = 0b0;
  let Inst{13} = 0b1;
  let Inst{11-1} = 0b00000000000;
  let Unpredictable{21-20} = 0b11;
  let Unpredictable{11-1} = 0b11111111111;
}
```
- EN: Defines TableGen record `MVE_LCTP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_LCTP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7037-7039
```tablegen
// Pseudo instructions for lowering MQQPR and MQQQQPR stack spills and reloads.
// They are equivalent to VLDMDIA/VSTMDIA with a single reg, as opposed to multiple
// dreg subregs.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 7041-7054
```tablegen
let Predicates = [HasMVEInt], AM = AddrMode4 in {
let mayStore = 1, hasSideEffects = 0 in {
  def MQQPRStore : t2PseudoInst<(outs), (ins MQQPR:$val, GPRnopc:$ptr),
                                4, NoItinerary, []>;
  def MQQQQPRStore : t2PseudoInst<(outs), (ins MQQQQPR:$val, GPRnopc:$ptr),
                                  4, NoItinerary, []>;
}
let mayLoad = 1, hasSideEffects = 0 in {
  def MQQPRLoad : t2PseudoInst<(outs MQQPR:$val), (ins GPRnopc:$ptr),
                               4, NoItinerary, []>;
  def MQQQQPRLoad : t2PseudoInst<(outs MQQQQPR:$val), (ins GPRnopc:$ptr),
                                 4, NoItinerary, []>;
}
}
```
- EN: Defines TableGen record `MQQPRStore` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MQQPRStore`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7056-7065
```tablegen
// Pseudo for lowering MVE Q register COPYs. These will usually get converted
// to a "MVE_VORR dst, src, src", but may behave differently in tail predicated
// loops to ensure the whole register is copied, not a subset from a
// tail-predicated MVE_VORR. In the event we cannot prove a MVE_VORR is valid,
// it will become a pair of VMOVD instructions for each half of the Q register.
let Predicates = [HasMVEInt], hasSideEffects = 0, isMoveReg = 1,
    D = MVEDomain in {
  def MQPRCopy : t2PseudoInst<(outs MQPR:$dst), (ins MQPR:$src),
                              8, NoItinerary, []>;
}
```
- EN: Defines TableGen record `MQPRCopy` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MQPRCopy`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7068-7070
```tablegen
//===----------------------------------------------------------------------===//
// Patterns
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 7072-7072
```tablegen
// PatFrags for loads and stores. Often trying to keep semi-consistent names.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 7074-7089
```tablegen
def aligned32_pre_store : PatFrag<(ops node:$val, node:$ptr, node:$offset),
                                  (pre_store node:$val, node:$ptr, node:$offset), [{
  return cast<StoreSDNode>(N)->getAlign() >= 4;
}]>;
def aligned32_post_store : PatFrag<(ops node:$val, node:$ptr, node:$offset),
                                   (post_store node:$val, node:$ptr, node:$offset), [{
  return cast<StoreSDNode>(N)->getAlign() >= 4;
}]>;
def aligned16_pre_store : PatFrag<(ops node:$val, node:$ptr, node:$offset),
                                  (pre_store node:$val, node:$ptr, node:$offset), [{
  return cast<StoreSDNode>(N)->getAlign() >= 2;
}]>;
def aligned16_post_store : PatFrag<(ops node:$val, node:$ptr, node:$offset),
                                   (post_store node:$val, node:$ptr, node:$offset), [{
  return cast<StoreSDNode>(N)->getAlign() >= 2;
}]>;
```
- EN: Defines TableGen record `aligned32_pre_store` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `aligned32_pre_store`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 7092-7109
```tablegen
def aligned_maskedloadvi8 : PatFrag<(ops node:$ptr, node:$pred, node:$passthru),
                                    (masked_ld node:$ptr, undef, node:$pred, node:$passthru), [{
  auto *Ld = cast<MaskedLoadSDNode>(N);
  return Ld->getMemoryVT().getScalarType() == MVT::i8;
}]>;
def aligned_sextmaskedloadvi8 : PatFrag<(ops node:$ptr, node:$pred, node:$passthru),
                                        (aligned_maskedloadvi8 node:$ptr, node:$pred, node:$passthru), [{
  return cast<MaskedLoadSDNode>(N)->getExtensionType() == ISD::SEXTLOAD;
}]>;
def aligned_zextmaskedloadvi8 : PatFrag<(ops node:$ptr, node:$pred, node:$passthru),
                                        (aligned_maskedloadvi8 node:$ptr, node:$pred, node:$passthru), [{
  return cast<MaskedLoadSDNode>(N)->getExtensionType() == ISD::ZEXTLOAD;
}]>;
def aligned_extmaskedloadvi8 : PatFrag<(ops node:$ptr, node:$pred, node:$passthru),
                                       (aligned_maskedloadvi8 node:$ptr, node:$pred, node:$passthru), [{
  auto *Ld = cast<MaskedLoadSDNode>(N);
  EVT ScalarVT = Ld->getMemoryVT().getScalarType();
  return ScalarVT.isInteger() && Ld->getExtensionType() == ISD::EXTLOAD;
```
- EN: Defines TableGen record `aligned_maskedloadvi8` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `aligned_maskedloadvi8`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 7110-7127
```tablegen
}]>;
def aligned_maskedloadvi16: PatFrag<(ops node:$ptr, node:$pred, node:$passthru),
                                    (masked_ld node:$ptr, undef, node:$pred, node:$passthru), [{
  auto *Ld = cast<MaskedLoadSDNode>(N);
  EVT ScalarVT = Ld->getMemoryVT().getScalarType();
  return (ScalarVT == MVT::i16 || ScalarVT == MVT::f16) && Ld->getAlign() >= 2;
}]>;
def aligned_sextmaskedloadvi16 : PatFrag<(ops node:$ptr, node:$pred, node:$passthru),
                                         (aligned_maskedloadvi16 node:$ptr, node:$pred, node:$passthru), [{
  return cast<MaskedLoadSDNode>(N)->getExtensionType() == ISD::SEXTLOAD;
}]>;
def aligned_zextmaskedloadvi16 : PatFrag<(ops node:$ptr, node:$pred, node:$passthru),
                                         (aligned_maskedloadvi16 node:$ptr, node:$pred, node:$passthru), [{
  return cast<MaskedLoadSDNode>(N)->getExtensionType() == ISD::ZEXTLOAD;
}]>;
def aligned_extmaskedloadvi16 : PatFrag<(ops node:$ptr, node:$pred, node:$passthru),
                                        (aligned_maskedloadvi16 node:$ptr, node:$pred, node:$passthru), [{
  auto *Ld = cast<MaskedLoadSDNode>(N);
```
- EN: Defines TableGen record `aligned_maskedloadvi16:` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `aligned_maskedloadvi16:`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 7128-7136
```tablegen
  EVT ScalarVT = Ld->getMemoryVT().getScalarType();
  return ScalarVT.isInteger() && Ld->getExtensionType() == ISD::EXTLOAD;
}]>;
def aligned_maskedloadvi32: PatFrag<(ops node:$ptr, node:$pred, node:$passthru),
                                    (masked_ld node:$ptr, undef, node:$pred, node:$passthru), [{
  auto *Ld = cast<MaskedLoadSDNode>(N);
  EVT ScalarVT = Ld->getMemoryVT().getScalarType();
  return (ScalarVT == MVT::i32 || ScalarVT == MVT::f32) && Ld->getAlign() >= 4;
}]>;
```
- EN: Defines TableGen record `aligned_maskedloadvi32:` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `aligned_maskedloadvi32:`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 7138-7153
```tablegen
def aligned_maskedstvi8 : PatFrag<(ops node:$val, node:$ptr, node:$pred),
                                  (masked_st node:$val, node:$ptr, undef, node:$pred), [{
  return cast<MaskedStoreSDNode>(N)->getMemoryVT().getScalarType() == MVT::i8;
}]>;
def aligned_maskedstvi16 : PatFrag<(ops node:$val, node:$ptr, node:$pred),
                                   (masked_st node:$val, node:$ptr, undef, node:$pred), [{
  auto *St = cast<MaskedStoreSDNode>(N);
  EVT ScalarVT = St->getMemoryVT().getScalarType();
  return (ScalarVT == MVT::i16 || ScalarVT == MVT::f16) && St->getAlign() >= 2;
}]>;
def aligned_maskedstvi32 : PatFrag<(ops node:$val, node:$ptr, node:$pred),
                                   (masked_st node:$val, node:$ptr, undef, node:$pred), [{
  auto *St = cast<MaskedStoreSDNode>(N);
  EVT ScalarVT = St->getMemoryVT().getScalarType();
  return (ScalarVT == MVT::i32 || ScalarVT == MVT::f32) && St->getAlign() >= 4;
}]>;
```
- EN: Defines TableGen record `aligned_maskedstvi8` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `aligned_maskedstvi8`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 7155-7172
```tablegen
def pre_maskedstore : PatFrag<(ops node:$val, node:$base, node:$offset, node:$mask),
                              (masked_st node:$val, node:$base, node:$offset, node:$mask), [{
  ISD::MemIndexedMode AM = cast<MaskedStoreSDNode>(N)->getAddressingMode();
  return AM == ISD::PRE_INC || AM == ISD::PRE_DEC;
}]>;
def post_maskedstore : PatFrag<(ops node:$val, node:$base, node:$offset, node:$mask),
                               (masked_st node:$val, node:$base, node:$offset, node:$mask), [{
  ISD::MemIndexedMode AM = cast<MaskedStoreSDNode>(N)->getAddressingMode();
  return AM == ISD::POST_INC || AM == ISD::POST_DEC;
}]>;
def aligned_pre_maskedstorevi8 : PatFrag<(ops node:$val, node:$ptr, node:$offset, node:$mask),
                                         (pre_maskedstore node:$val, node:$ptr, node:$offset, node:$mask), [{
  return cast<MaskedStoreSDNode>(N)->getMemoryVT().getScalarType() == MVT::i8;
}]>;
def aligned_post_maskedstorevi8 : PatFrag<(ops node:$val, node:$ptr, node:$offset, node:$mask),
                                          (post_maskedstore node:$val, node:$ptr, node:$offset, node:$mask), [{
  return cast<MaskedStoreSDNode>(N)->getMemoryVT().getScalarType() == MVT::i8;
}]>;
```
- EN: Defines TableGen record `pre_maskedstore` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `pre_maskedstore`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 7173-7190
```tablegen
def aligned_pre_maskedstorevi16 : PatFrag<(ops node:$val, node:$ptr, node:$offset, node:$mask),
                                          (pre_maskedstore node:$val, node:$ptr, node:$offset, node:$mask), [{
  auto *St = cast<MaskedStoreSDNode>(N);
  EVT ScalarVT = St->getMemoryVT().getScalarType();
  return (ScalarVT == MVT::i16 || ScalarVT == MVT::f16) && St->getAlign() >= 2;
}]>;
def aligned_post_maskedstorevi16 : PatFrag<(ops node:$val, node:$ptr, node:$offset, node:$mask),
                                           (post_maskedstore node:$val, node:$ptr, node:$offset, node:$mask), [{
  auto *St = cast<MaskedStoreSDNode>(N);
  EVT ScalarVT = St->getMemoryVT().getScalarType();
  return (ScalarVT == MVT::i16 || ScalarVT == MVT::f16) && St->getAlign() >= 2;
}]>;
def aligned_pre_maskedstorevi32 : PatFrag<(ops node:$val, node:$ptr, node:$offset, node:$mask),
                                          (pre_maskedstore node:$val, node:$ptr, node:$offset, node:$mask), [{
  auto *St = cast<MaskedStoreSDNode>(N);
  EVT ScalarVT = St->getMemoryVT().getScalarType();
  return (ScalarVT == MVT::i32 || ScalarVT == MVT::f32) && St->getAlign() >= 4;
}]>;
```
- EN: Defines TableGen record `aligned_pre_maskedstorevi16` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `aligned_pre_maskedstorevi16`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 7191-7196
```tablegen
def aligned_post_maskedstorevi32 : PatFrag<(ops node:$val, node:$ptr, node:$offset, node:$mask),
                                           (post_maskedstore node:$val, node:$ptr, node:$offset, node:$mask), [{
  auto *St = cast<MaskedStoreSDNode>(N);
  EVT ScalarVT = St->getMemoryVT().getScalarType();
  return (ScalarVT == MVT::i32 || ScalarVT == MVT::f32) && St->getAlign() >= 4;
}]>;
```
- EN: Defines TableGen record `aligned_post_maskedstorevi32` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `aligned_post_maskedstorevi32`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 7199-7199
```tablegen
// PatFrags for "Aligned" extending / truncating
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 7201-7203
```tablegen
def aligned_extloadvi8  : PatFrag<(ops node:$ptr), (extloadvi8 node:$ptr)>;
def aligned_sextloadvi8 : PatFrag<(ops node:$ptr), (sextloadvi8 node:$ptr)>;
def aligned_zextloadvi8 : PatFrag<(ops node:$ptr), (zextloadvi8 node:$ptr)>;
```
- EN: Defines TableGen record `aligned_extloadvi8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `aligned_extloadvi8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7205-7210
```tablegen
def aligned_truncstvi8 : PatFrag<(ops node:$val, node:$ptr),
                                 (truncstorevi8 node:$val, node:$ptr)>;
def aligned_post_truncstvi8 : PatFrag<(ops node:$val, node:$base, node:$offset),
                                      (post_truncstvi8 node:$val, node:$base, node:$offset)>;
def aligned_pre_truncstvi8 : PatFrag<(ops node:$val, node:$base, node:$offset),
                                     (pre_truncstvi8 node:$val, node:$base, node:$offset)>;
```
- EN: Defines TableGen record `aligned_truncstvi8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `aligned_truncstvi8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7212-7215
```tablegen
let MinAlignment = 2 in {
  def aligned_extloadvi16  : PatFrag<(ops node:$ptr), (extloadvi16 node:$ptr)>;
  def aligned_sextloadvi16 : PatFrag<(ops node:$ptr), (sextloadvi16 node:$ptr)>;
  def aligned_zextloadvi16 : PatFrag<(ops node:$ptr), (zextloadvi16 node:$ptr)>;
```
- EN: Defines TableGen record `aligned_extloadvi16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `aligned_extloadvi16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7217-7223
```tablegen
  def aligned_truncstvi16 : PatFrag<(ops node:$val, node:$ptr),
                                    (truncstorevi16 node:$val, node:$ptr)>;
  def aligned_post_truncstvi16 : PatFrag<(ops node:$val, node:$base, node:$offset),
                                         (post_truncstvi16 node:$val, node:$base, node:$offset)>;
  def aligned_pre_truncstvi16 : PatFrag<(ops node:$val, node:$base, node:$offset),
                                        (pre_truncstvi16 node:$val, node:$base, node:$offset)>;
}
```
- EN: Defines TableGen record `aligned_truncstvi16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `aligned_truncstvi16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7225-7242
```tablegen
def truncmaskedst : PatFrag<(ops node:$val, node:$base, node:$pred),
                            (masked_st node:$val, node:$base, undef, node:$pred), [{
  return cast<MaskedStoreSDNode>(N)->isTruncatingStore();
}]>;
def aligned_truncmaskedstvi8 : PatFrag<(ops node:$val, node:$base, node:$pred),
                                       (truncmaskedst node:$val, node:$base, node:$pred), [{
  return cast<MaskedStoreSDNode>(N)->getMemoryVT().getScalarType() == MVT::i8;
}]>;
def aligned_truncmaskedstvi16 : PatFrag<(ops node:$val, node:$base, node:$pred),
                                        (truncmaskedst node:$val, node:$base, node:$pred), [{
  auto *St = cast<MaskedStoreSDNode>(N);
  EVT ScalarVT = St->getMemoryVT().getScalarType();
  return (ScalarVT == MVT::i16 || ScalarVT == MVT::f16) && St->getAlign() >= 2;
}]>;
def pre_truncmaskedst : PatFrag<(ops node:$val, node:$base, node:$offset, node:$pred),
                                (masked_st node:$val, node:$base, node:$offset, node:$pred), [{
  ISD::MemIndexedMode AM = cast<MaskedStoreSDNode>(N)->getAddressingMode();
  return cast<MaskedStoreSDNode>(N)->isTruncatingStore() && (AM == ISD::PRE_INC || AM == ISD::PRE_DEC);
```
- EN: Defines TableGen record `truncmaskedst` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `truncmaskedst`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 7243-7260
```tablegen
}]>;
def aligned_pre_truncmaskedstvi8 : PatFrag<(ops node:$val, node:$base, node:$offset, node:$pred),
                                           (pre_truncmaskedst node:$val, node:$base, node:$offset, node:$pred), [{
  return cast<MaskedStoreSDNode>(N)->getMemoryVT().getScalarType() == MVT::i8;
}]>;
def aligned_pre_truncmaskedstvi16 : PatFrag<(ops node:$val, node:$base, node:$offset, node:$pred),
                                            (pre_truncmaskedst node:$val, node:$base, node:$offset, node:$pred), [{
  auto *St = cast<MaskedStoreSDNode>(N);
  EVT ScalarVT = St->getMemoryVT().getScalarType();
  return (ScalarVT == MVT::i16 || ScalarVT == MVT::f16) && St->getAlign() >= 2;
}]>;
def post_truncmaskedst : PatFrag<(ops node:$val, node:$base, node:$offset, node:$postd),
                                 (masked_st node:$val, node:$base, node:$offset, node:$postd), [{
  ISD::MemIndexedMode AM = cast<MaskedStoreSDNode>(N)->getAddressingMode();
  return cast<MaskedStoreSDNode>(N)->isTruncatingStore() && (AM == ISD::POST_INC || AM == ISD::POST_DEC);
}]>;
def aligned_post_truncmaskedstvi8 : PatFrag<(ops node:$val, node:$base, node:$offset, node:$postd),
                                            (post_truncmaskedst node:$val, node:$base, node:$offset, node:$postd), [{
```
- EN: Defines TableGen record `aligned_pre_truncmaskedstvi8` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `aligned_pre_truncmaskedstvi8`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 7261-7268
```tablegen
  return cast<MaskedStoreSDNode>(N)->getMemoryVT().getScalarType() == MVT::i8;
}]>;
def aligned_post_truncmaskedstvi16 : PatFrag<(ops node:$val, node:$base, node:$offset, node:$postd),
                                             (post_truncmaskedst node:$val, node:$base, node:$offset, node:$postd), [{
  auto *St = cast<MaskedStoreSDNode>(N);
  EVT ScalarVT = St->getMemoryVT().getScalarType();
  return (ScalarVT == MVT::i16 || ScalarVT == MVT::f16) && St->getAlign() >= 2;
}]>;
```
- EN: Defines TableGen record `aligned_post_truncmaskedstvi16` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `aligned_post_truncmaskedstvi16`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 7270-7270
```tablegen
// Load/store patterns
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 7272-7275
```tablegen
class MVE_vector_store_typed<ValueType Ty, Instruction RegImmInst,
                             PatFrag StoreKind, int shift>
  : Pat<(StoreKind (Ty MQPR:$val), t2addrmode_imm7<shift>:$addr),
        (RegImmInst (Ty MQPR:$val), t2addrmode_imm7<shift>:$addr)>;
```
- EN: Declares reusable TableGen class `MVE_vector_store_typed` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_vector_store_typed`，通常用于抽象共享字段、谓词或编码结构。

### Lines 7277-7280
```tablegen
class MVE_vector_maskedstore_typed<ValueType Ty, Instruction RegImmInst,
                                   PatFrag StoreKind, int shift>
  : Pat<(StoreKind (Ty MQPR:$val), t2addrmode_imm7<shift>:$addr, VCCR:$pred),
        (RegImmInst (Ty MQPR:$val), t2addrmode_imm7<shift>:$addr, ARMVCCThen, VCCR:$pred, zero_reg)>;
```
- EN: Declares reusable TableGen class `MVE_vector_maskedstore_typed` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_vector_maskedstore_typed`，通常用于抽象共享字段、谓词或编码结构。

### Lines 7282-7291
```tablegen
multiclass MVE_vector_store<Instruction RegImmInst, PatFrag StoreKind,
                            int shift> {
  def : MVE_vector_store_typed<v16i8, RegImmInst, StoreKind, shift>;
  def : MVE_vector_store_typed<v8i16, RegImmInst, StoreKind, shift>;
  def : MVE_vector_store_typed<v8f16, RegImmInst, StoreKind, shift>;
  def : MVE_vector_store_typed<v4i32, RegImmInst, StoreKind, shift>;
  def : MVE_vector_store_typed<v4f32, RegImmInst, StoreKind, shift>;
  def : MVE_vector_store_typed<v2i64, RegImmInst, StoreKind, shift>;
  def : MVE_vector_store_typed<v2f64, RegImmInst, StoreKind, shift>;
}
```
- EN: Declares TableGen `multiclass MVE_vector_store`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_vector_store`，它是一个可复用模板，可展开为多个相关记录。

### Lines 7293-7296
```tablegen
class MVE_vector_load_typed<ValueType Ty, Instruction RegImmInst,
                            PatFrag LoadKind, int shift>
  : Pat<(Ty (LoadKind t2addrmode_imm7<shift>:$addr)),
        (Ty (RegImmInst t2addrmode_imm7<shift>:$addr))>;
```
- EN: Declares reusable TableGen class `MVE_vector_load_typed` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_vector_load_typed`，通常用于抽象共享字段、谓词或编码结构。

### Lines 7298-7301
```tablegen
class MVE_vector_maskedload_typed<ValueType Ty, Instruction RegImmInst,
                                  PatFrag LoadKind, int shift>
  : Pat<(Ty (LoadKind t2addrmode_imm7<shift>:$addr, VCCR:$pred, (Ty (ARMvmovImm (i32 0))))),
        (Ty (RegImmInst t2addrmode_imm7<shift>:$addr, ARMVCCThen, VCCR:$pred, zero_reg))>;
```
- EN: Declares reusable TableGen class `MVE_vector_maskedload_typed` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_vector_maskedload_typed`，通常用于抽象共享字段、谓词或编码结构。

### Lines 7303-7312
```tablegen
multiclass MVE_vector_load<Instruction RegImmInst, PatFrag LoadKind,
                           int shift> {
  def : MVE_vector_load_typed<v16i8, RegImmInst, LoadKind, shift>;
  def : MVE_vector_load_typed<v8i16, RegImmInst, LoadKind, shift>;
  def : MVE_vector_load_typed<v8f16, RegImmInst, LoadKind, shift>;
  def : MVE_vector_load_typed<v4i32, RegImmInst, LoadKind, shift>;
  def : MVE_vector_load_typed<v4f32, RegImmInst, LoadKind, shift>;
  def : MVE_vector_load_typed<v2i64, RegImmInst, LoadKind, shift>;
  def : MVE_vector_load_typed<v2f64, RegImmInst, LoadKind, shift>;
}
```
- EN: Declares TableGen `multiclass MVE_vector_load`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_vector_load`，它是一个可复用模板，可展开为多个相关记录。

### Lines 7314-7317
```tablegen
class MVE_vector_offset_store_typed<ValueType Ty, Instruction Opcode,
                                    PatFrag StoreKind, int shift>
  : Pat<(StoreKind (Ty MQPR:$Rt), tGPR:$Rn, t2am_imm7_offset<shift>:$addr),
        (Opcode MQPR:$Rt, tGPR:$Rn, t2am_imm7_offset<shift>:$addr)>;
```
- EN: Declares reusable TableGen class `MVE_vector_offset_store_typed` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_vector_offset_store_typed`，通常用于抽象共享字段、谓词或编码结构。

### Lines 7319-7322
```tablegen
class MVE_vector_offset_maskedstore_typed<ValueType Ty, Instruction Opcode,
                                          PatFrag StoreKind, int shift>
  : Pat<(StoreKind (Ty MQPR:$Rt), tGPR:$Rn, t2am_imm7_offset<shift>:$addr, VCCR:$pred),
        (Opcode MQPR:$Rt, tGPR:$Rn, t2am_imm7_offset<shift>:$addr, ARMVCCThen, VCCR:$pred, zero_reg)>;
```
- EN: Declares reusable TableGen class `MVE_vector_offset_maskedstore_typed` for `ARMInstrMVE`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrMVE` 声明可复用的 TableGen 类 `MVE_vector_offset_maskedstore_typed`，通常用于抽象共享字段、谓词或编码结构。

### Lines 7324-7333
```tablegen
multiclass MVE_vector_offset_store<Instruction RegImmInst, PatFrag StoreKind,
                                   int shift> {
  def : MVE_vector_offset_store_typed<v16i8, RegImmInst, StoreKind, shift>;
  def : MVE_vector_offset_store_typed<v8i16, RegImmInst, StoreKind, shift>;
  def : MVE_vector_offset_store_typed<v8f16, RegImmInst, StoreKind, shift>;
  def : MVE_vector_offset_store_typed<v4i32, RegImmInst, StoreKind, shift>;
  def : MVE_vector_offset_store_typed<v4f32, RegImmInst, StoreKind, shift>;
  def : MVE_vector_offset_store_typed<v2i64, RegImmInst, StoreKind, shift>;
  def : MVE_vector_offset_store_typed<v2f64, RegImmInst, StoreKind, shift>;
}
```
- EN: Declares TableGen `multiclass MVE_vector_offset_store`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVE_vector_offset_store`，它是一个可复用模板，可展开为多个相关记录。

### Lines 7336-7340
```tablegen
let Predicates = [HasMVEInt, IsLE] in {
  // Stores
  defm : MVE_vector_store<MVE_VSTRBU8, byte_alignedstore, 0>;
  defm : MVE_vector_store<MVE_VSTRHU16, hword_alignedstore, 1>;
  defm : MVE_vector_store<MVE_VSTRWU32, alignedstore32, 2>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 7342-7345
```tablegen
  // Loads
  defm : MVE_vector_load<MVE_VLDRBU8, byte_alignedload, 0>;
  defm : MVE_vector_load<MVE_VLDRHU16, hword_alignedload, 1>;
  defm : MVE_vector_load<MVE_VLDRWU32, alignedload32, 2>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7347-7354
```tablegen
  // Pre/post inc stores
  defm : MVE_vector_offset_store<MVE_VSTRBU8_pre, pre_store, 0>;
  defm : MVE_vector_offset_store<MVE_VSTRBU8_post, post_store, 0>;
  defm : MVE_vector_offset_store<MVE_VSTRHU16_pre, aligned16_pre_store, 1>;
  defm : MVE_vector_offset_store<MVE_VSTRHU16_post, aligned16_post_store, 1>;
  defm : MVE_vector_offset_store<MVE_VSTRWU32_pre, aligned32_pre_store, 2>;
  defm : MVE_vector_offset_store<MVE_VSTRWU32_post, aligned32_post_store, 2>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7356-7362
```tablegen
let Predicates = [HasMVEInt, IsBE] in {
  // Aligned Stores
  def : MVE_vector_store_typed<v16i8, MVE_VSTRBU8, store, 0>;
  def : MVE_vector_store_typed<v8i16, MVE_VSTRHU16, alignedstore16, 1>;
  def : MVE_vector_store_typed<v8f16, MVE_VSTRHU16, alignedstore16, 1>;
  def : MVE_vector_store_typed<v4i32, MVE_VSTRWU32, alignedstore32, 2>;
  def : MVE_vector_store_typed<v4f32, MVE_VSTRWU32, alignedstore32, 2>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 7364-7369
```tablegen
  // Aligned Loads
  def : MVE_vector_load_typed<v16i8, MVE_VLDRBU8, load, 0>;
  def : MVE_vector_load_typed<v8i16, MVE_VLDRHU16, alignedload16, 1>;
  def : MVE_vector_load_typed<v8f16, MVE_VLDRHU16, alignedload16, 1>;
  def : MVE_vector_load_typed<v4i32, MVE_VLDRWU32, alignedload32, 2>;
  def : MVE_vector_load_typed<v4f32, MVE_VLDRWU32, alignedload32, 2>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7371-7388
```tablegen
  // Other unaligned loads/stores need to go though a VREV
  def : Pat<(v2f64 (load t2addrmode_imm7<0>:$addr)),
            (v2f64 (MVE_VREV64_8 (MVE_VLDRBU8 t2addrmode_imm7<0>:$addr)))>;
  def : Pat<(v2i64 (load t2addrmode_imm7<0>:$addr)),
            (v2i64 (MVE_VREV64_8 (MVE_VLDRBU8 t2addrmode_imm7<0>:$addr)))>;
  def : Pat<(v4i32 (load t2addrmode_imm7<0>:$addr)),
            (v4i32 (MVE_VREV32_8 (MVE_VLDRBU8 t2addrmode_imm7<0>:$addr)))>;
  def : Pat<(v4f32 (load t2addrmode_imm7<0>:$addr)),
            (v4f32 (MVE_VREV32_8 (MVE_VLDRBU8 t2addrmode_imm7<0>:$addr)))>;
  def : Pat<(v8i16 (load t2addrmode_imm7<0>:$addr)),
            (v8i16 (MVE_VREV16_8 (MVE_VLDRBU8 t2addrmode_imm7<0>:$addr)))>;
  def : Pat<(v8f16 (load t2addrmode_imm7<0>:$addr)),
            (v8f16 (MVE_VREV16_8 (MVE_VLDRBU8 t2addrmode_imm7<0>:$addr)))>;
  def : Pat<(store (v2f64 MQPR:$val), t2addrmode_imm7<0>:$addr),
            (MVE_VSTRBU8 (MVE_VREV64_8 MQPR:$val), t2addrmode_imm7<0>:$addr)>;
  def : Pat<(store (v2i64 MQPR:$val), t2addrmode_imm7<0>:$addr),
            (MVE_VSTRBU8 (MVE_VREV64_8 MQPR:$val), t2addrmode_imm7<0>:$addr)>;
  def : Pat<(store (v4i32 MQPR:$val), t2addrmode_imm7<0>:$addr),
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7389-7395
```tablegen
            (MVE_VSTRBU8 (MVE_VREV32_8 MQPR:$val), t2addrmode_imm7<0>:$addr)>;
  def : Pat<(store (v4f32 MQPR:$val), t2addrmode_imm7<0>:$addr),
            (MVE_VSTRBU8 (MVE_VREV32_8 MQPR:$val), t2addrmode_imm7<0>:$addr)>;
  def : Pat<(store (v8i16 MQPR:$val), t2addrmode_imm7<0>:$addr),
            (MVE_VSTRBU8 (MVE_VREV16_8 MQPR:$val), t2addrmode_imm7<0>:$addr)>;
  def : Pat<(store (v8f16 MQPR:$val), t2addrmode_imm7<0>:$addr),
            (MVE_VSTRBU8 (MVE_VREV16_8 MQPR:$val), t2addrmode_imm7<0>:$addr)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7397-7408
```tablegen
  // Pre/Post inc stores
  def : MVE_vector_offset_store_typed<v16i8, MVE_VSTRBU8_pre, pre_store, 0>;
  def : MVE_vector_offset_store_typed<v16i8, MVE_VSTRBU8_post, post_store, 0>;
  def : MVE_vector_offset_store_typed<v8i16, MVE_VSTRHU16_pre, aligned16_pre_store, 1>;
  def : MVE_vector_offset_store_typed<v8i16, MVE_VSTRHU16_post, aligned16_post_store, 1>;
  def : MVE_vector_offset_store_typed<v8f16, MVE_VSTRHU16_pre, aligned16_pre_store, 1>;
  def : MVE_vector_offset_store_typed<v8f16, MVE_VSTRHU16_post, aligned16_post_store, 1>;
  def : MVE_vector_offset_store_typed<v4i32, MVE_VSTRWU32_pre, aligned32_pre_store, 2>;
  def : MVE_vector_offset_store_typed<v4i32, MVE_VSTRWU32_post, aligned32_post_store, 2>;
  def : MVE_vector_offset_store_typed<v4f32, MVE_VSTRWU32_pre, aligned32_pre_store, 2>;
  def : MVE_vector_offset_store_typed<v4f32, MVE_VSTRWU32_post, aligned32_post_store, 2>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7410-7416
```tablegen
let Predicates = [HasMVEInt] in {
  // Aligned masked store, shared between LE and BE
  def : MVE_vector_maskedstore_typed<v16i8, MVE_VSTRBU8, aligned_maskedstvi8, 0>;
  def : MVE_vector_maskedstore_typed<v8i16, MVE_VSTRHU16, aligned_maskedstvi16, 1>;
  def : MVE_vector_maskedstore_typed<v8f16, MVE_VSTRHU16, aligned_maskedstvi16, 1>;
  def : MVE_vector_maskedstore_typed<v4i32, MVE_VSTRWU32, aligned_maskedstvi32, 2>;
  def : MVE_vector_maskedstore_typed<v4f32, MVE_VSTRWU32, aligned_maskedstvi32, 2>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 7418-7428
```tablegen
  // Pre/Post inc masked stores
  def : MVE_vector_offset_maskedstore_typed<v16i8, MVE_VSTRBU8_pre, aligned_pre_maskedstorevi8, 0>;
  def : MVE_vector_offset_maskedstore_typed<v16i8, MVE_VSTRBU8_post, aligned_post_maskedstorevi8, 0>;
  def : MVE_vector_offset_maskedstore_typed<v8i16, MVE_VSTRHU16_pre, aligned_pre_maskedstorevi16, 1>;
  def : MVE_vector_offset_maskedstore_typed<v8i16, MVE_VSTRHU16_post, aligned_post_maskedstorevi16, 1>;
  def : MVE_vector_offset_maskedstore_typed<v8f16, MVE_VSTRHU16_pre, aligned_pre_maskedstorevi16, 1>;
  def : MVE_vector_offset_maskedstore_typed<v8f16, MVE_VSTRHU16_post, aligned_post_maskedstorevi16, 1>;
  def : MVE_vector_offset_maskedstore_typed<v4i32, MVE_VSTRWU32_pre, aligned_pre_maskedstorevi32, 2>;
  def : MVE_vector_offset_maskedstore_typed<v4i32, MVE_VSTRWU32_post, aligned_post_maskedstorevi32, 2>;
  def : MVE_vector_offset_maskedstore_typed<v4f32, MVE_VSTRWU32_pre, aligned_pre_maskedstorevi32, 2>;
  def : MVE_vector_offset_maskedstore_typed<v4f32, MVE_VSTRWU32_post, aligned_post_maskedstorevi32, 2>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7430-7436
```tablegen
  // Aligned masked loads
  def : MVE_vector_maskedload_typed<v16i8, MVE_VLDRBU8, aligned_maskedloadvi8, 0>;
  def : MVE_vector_maskedload_typed<v8i16, MVE_VLDRHU16, aligned_maskedloadvi16, 1>;
  def : MVE_vector_maskedload_typed<v8f16, MVE_VLDRHU16, aligned_maskedloadvi16, 1>;
  def : MVE_vector_maskedload_typed<v4i32, MVE_VLDRWU32, aligned_maskedloadvi32, 2>;
  def : MVE_vector_maskedload_typed<v4f32, MVE_VLDRWU32, aligned_maskedloadvi32, 2>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7438-7438
```tablegen
// Widening/Narrowing Loads/Stores
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 7440-7448
```tablegen
multiclass MVEExtLoadStore<Instruction LoadSInst, Instruction LoadUInst, string StoreInst,
                         string Amble, ValueType VT, int Shift> {
  // Trunc stores
  def : Pat<(!cast<PatFrag>("aligned_truncst"#Amble) (VT MQPR:$val), taddrmode_imm7<Shift>:$addr),
            (!cast<Instruction>(StoreInst) MQPR:$val, taddrmode_imm7<Shift>:$addr)>;
  def : Pat<(!cast<PatFrag>("aligned_post_truncst"#Amble) (VT MQPR:$Rt), tGPR:$Rn, t2am_imm7_offset<Shift>:$addr),
            (!cast<Instruction>(StoreInst#"_post") MQPR:$Rt, tGPR:$Rn, t2am_imm7_offset<Shift>:$addr)>;
  def : Pat<(!cast<PatFrag>("aligned_pre_truncst"#Amble) (VT MQPR:$Rt), tGPR:$Rn, t2am_imm7_offset<Shift>:$addr),
            (!cast<Instruction>(StoreInst#"_pre") MQPR:$Rt, tGPR:$Rn, t2am_imm7_offset<Shift>:$addr)>;
```
- EN: Declares TableGen `multiclass MVEExtLoadStore`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MVEExtLoadStore`，它是一个可复用模板，可展开为多个相关记录。

### Lines 7450-7456
```tablegen
  // Masked trunc stores
  def : Pat<(!cast<PatFrag>("aligned_truncmaskedst"#Amble) (VT MQPR:$val), taddrmode_imm7<Shift>:$addr, VCCR:$pred),
            (!cast<Instruction>(StoreInst) MQPR:$val, taddrmode_imm7<Shift>:$addr, ARMVCCThen, VCCR:$pred, zero_reg)>;
  def : Pat<(!cast<PatFrag>("aligned_post_truncmaskedst"#Amble) (VT MQPR:$Rt), tGPR:$Rn, t2am_imm7_offset<Shift>:$addr, VCCR:$pred),
            (!cast<Instruction>(StoreInst#"_post") MQPR:$Rt, tGPR:$Rn, t2am_imm7_offset<Shift>:$addr, ARMVCCThen, VCCR:$pred, zero_reg)>;
  def : Pat<(!cast<PatFrag>("aligned_pre_truncmaskedst"#Amble) (VT MQPR:$Rt), tGPR:$Rn, t2am_imm7_offset<Shift>:$addr, VCCR:$pred),
            (!cast<Instruction>(StoreInst#"_pre") MQPR:$Rt, tGPR:$Rn, t2am_imm7_offset<Shift>:$addr, ARMVCCThen, VCCR:$pred, zero_reg)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7458-7464
```tablegen
  // Ext loads
  def : Pat<(VT (!cast<PatFrag>("aligned_extload"#Amble) taddrmode_imm7<Shift>:$addr)),
            (VT (LoadUInst taddrmode_imm7<Shift>:$addr))>;
  def : Pat<(VT (!cast<PatFrag>("aligned_sextload"#Amble) taddrmode_imm7<Shift>:$addr)),
            (VT (LoadSInst taddrmode_imm7<Shift>:$addr))>;
  def : Pat<(VT (!cast<PatFrag>("aligned_zextload"#Amble) taddrmode_imm7<Shift>:$addr)),
            (VT (LoadUInst taddrmode_imm7<Shift>:$addr))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7466-7473
```tablegen
  // Masked ext loads
  def : Pat<(VT (!cast<PatFrag>("aligned_extmaskedload"#Amble) taddrmode_imm7<Shift>:$addr, VCCR:$pred, (VT (ARMvmovImm (i32 0))))),
            (VT (LoadUInst taddrmode_imm7<Shift>:$addr, ARMVCCThen, VCCR:$pred, zero_reg))>;
  def : Pat<(VT (!cast<PatFrag>("aligned_sextmaskedload"#Amble) taddrmode_imm7<Shift>:$addr, VCCR:$pred, (VT (ARMvmovImm (i32 0))))),
            (VT (LoadSInst taddrmode_imm7<Shift>:$addr, ARMVCCThen, VCCR:$pred, zero_reg))>;
  def : Pat<(VT (!cast<PatFrag>("aligned_zextmaskedload"#Amble) taddrmode_imm7<Shift>:$addr, VCCR:$pred, (VT (ARMvmovImm (i32 0))))),
            (VT (LoadUInst taddrmode_imm7<Shift>:$addr, ARMVCCThen, VCCR:$pred, zero_reg))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7475-7479
```tablegen
let Predicates = [HasMVEInt] in {
  defm : MVEExtLoadStore<MVE_VLDRBS16, MVE_VLDRBU16, "MVE_VSTRB16", "vi8", v8i16, 0>;
  defm : MVEExtLoadStore<MVE_VLDRBS32, MVE_VLDRBU32, "MVE_VSTRB32", "vi8", v4i32, 0>;
  defm : MVEExtLoadStore<MVE_VLDRHS32, MVE_VLDRHU32, "MVE_VSTRH32", "vi16", v4i32, 1>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 7482-7482
```tablegen
// Bit convert patterns
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 7484-7486
```tablegen
let Predicates = [HasMVEInt] in {
  def : Pat<(v2f64 (bitconvert (v2i64 MQPR:$src))), (v2f64 MQPR:$src)>;
  def : Pat<(v2i64 (bitconvert (v2f64 MQPR:$src))), (v2i64 MQPR:$src)>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 7488-7489
```tablegen
  def : Pat<(v4i32 (bitconvert (v4f32 MQPR:$src))), (v4i32 MQPR:$src)>;
  def : Pat<(v4f32 (bitconvert (v4i32 MQPR:$src))), (v4f32 MQPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7491-7493
```tablegen
  def : Pat<(v8i16 (bitconvert (v8f16 MQPR:$src))), (v8i16  MQPR:$src)>;
  def : Pat<(v8f16 (bitconvert (v8i16 MQPR:$src))), (v8f16  MQPR:$src)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7495-7500
```tablegen
let Predicates = [IsLE,HasMVEInt] in {
  def : Pat<(v2f64 (bitconvert (v4f32 MQPR:$src))), (v2f64 MQPR:$src)>;
  def : Pat<(v2f64 (bitconvert (v4i32 MQPR:$src))), (v2f64 MQPR:$src)>;
  def : Pat<(v2f64 (bitconvert (v8f16 MQPR:$src))), (v2f64 MQPR:$src)>;
  def : Pat<(v2f64 (bitconvert (v8i16 MQPR:$src))), (v2f64 MQPR:$src)>;
  def : Pat<(v2f64 (bitconvert (v16i8 MQPR:$src))), (v2f64 MQPR:$src)>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 7502-7506
```tablegen
  def : Pat<(v2i64 (bitconvert (v4f32 MQPR:$src))), (v2i64 MQPR:$src)>;
  def : Pat<(v2i64 (bitconvert (v4i32 MQPR:$src))), (v2i64 MQPR:$src)>;
  def : Pat<(v2i64 (bitconvert (v8f16 MQPR:$src))), (v2i64 MQPR:$src)>;
  def : Pat<(v2i64 (bitconvert (v8i16 MQPR:$src))), (v2i64 MQPR:$src)>;
  def : Pat<(v2i64 (bitconvert (v16i8 MQPR:$src))), (v2i64 MQPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7508-7512
```tablegen
  def : Pat<(v4f32 (bitconvert (v2f64 MQPR:$src))), (v4f32 MQPR:$src)>;
  def : Pat<(v4f32 (bitconvert (v2i64 MQPR:$src))), (v4f32 MQPR:$src)>;
  def : Pat<(v4f32 (bitconvert (v8f16 MQPR:$src))), (v4f32 MQPR:$src)>;
  def : Pat<(v4f32 (bitconvert (v8i16 MQPR:$src))), (v4f32 MQPR:$src)>;
  def : Pat<(v4f32 (bitconvert (v16i8 MQPR:$src))), (v4f32 MQPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7514-7518
```tablegen
  def : Pat<(v4i32 (bitconvert (v2f64 MQPR:$src))), (v4i32 MQPR:$src)>;
  def : Pat<(v4i32 (bitconvert (v2i64 MQPR:$src))), (v4i32 MQPR:$src)>;
  def : Pat<(v4i32 (bitconvert (v8f16 MQPR:$src))), (v4i32 MQPR:$src)>;
  def : Pat<(v4i32 (bitconvert (v8i16 MQPR:$src))), (v4i32 MQPR:$src)>;
  def : Pat<(v4i32 (bitconvert (v16i8 MQPR:$src))), (v4i32 MQPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7520-7524
```tablegen
  def : Pat<(v8f16 (bitconvert (v2f64 MQPR:$src))), (v8f16 MQPR:$src)>;
  def : Pat<(v8f16 (bitconvert (v2i64 MQPR:$src))), (v8f16 MQPR:$src)>;
  def : Pat<(v8f16 (bitconvert (v4f32 MQPR:$src))), (v8f16 MQPR:$src)>;
  def : Pat<(v8f16 (bitconvert (v4i32 MQPR:$src))), (v8f16 MQPR:$src)>;
  def : Pat<(v8f16 (bitconvert (v16i8 MQPR:$src))), (v8f16 MQPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7526-7530
```tablegen
  def : Pat<(v8i16 (bitconvert (v2f64 MQPR:$src))), (v8i16 MQPR:$src)>;
  def : Pat<(v8i16 (bitconvert (v2i64 MQPR:$src))), (v8i16 MQPR:$src)>;
  def : Pat<(v8i16 (bitconvert (v4f32 MQPR:$src))), (v8i16 MQPR:$src)>;
  def : Pat<(v8i16 (bitconvert (v4i32 MQPR:$src))), (v8i16 MQPR:$src)>;
  def : Pat<(v8i16 (bitconvert (v16i8 MQPR:$src))), (v8i16 MQPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7532-7538
```tablegen
  def : Pat<(v16i8 (bitconvert (v2f64 MQPR:$src))), (v16i8 MQPR:$src)>;
  def : Pat<(v16i8 (bitconvert (v2i64 MQPR:$src))), (v16i8 MQPR:$src)>;
  def : Pat<(v16i8 (bitconvert (v4f32 MQPR:$src))), (v16i8 MQPR:$src)>;
  def : Pat<(v16i8 (bitconvert (v4i32 MQPR:$src))), (v16i8 MQPR:$src)>;
  def : Pat<(v16i8 (bitconvert (v8f16 MQPR:$src))), (v16i8 MQPR:$src)>;
  def : Pat<(v16i8 (bitconvert (v8i16 MQPR:$src))), (v16i8 MQPR:$src)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7540-7545
```tablegen
let Predicates = [IsBE,HasMVEInt] in {
  def : Pat<(v2f64 (bitconvert (v4f32 MQPR:$src))), (v2f64 (MVE_VREV64_32 MQPR:$src))>;
  def : Pat<(v2f64 (bitconvert (v4i32 MQPR:$src))), (v2f64 (MVE_VREV64_32 MQPR:$src))>;
  def : Pat<(v2f64 (bitconvert (v8f16 MQPR:$src))), (v2f64 (MVE_VREV64_16 MQPR:$src))>;
  def : Pat<(v2f64 (bitconvert (v8i16 MQPR:$src))), (v2f64 (MVE_VREV64_16 MQPR:$src))>;
  def : Pat<(v2f64 (bitconvert (v16i8 MQPR:$src))), (v2f64 (MVE_VREV64_8 MQPR:$src))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 7547-7551
```tablegen
  def : Pat<(v2i64 (bitconvert (v4f32 MQPR:$src))), (v2i64 (MVE_VREV64_32 MQPR:$src))>;
  def : Pat<(v2i64 (bitconvert (v4i32 MQPR:$src))), (v2i64 (MVE_VREV64_32 MQPR:$src))>;
  def : Pat<(v2i64 (bitconvert (v8f16 MQPR:$src))), (v2i64 (MVE_VREV64_16 MQPR:$src))>;
  def : Pat<(v2i64 (bitconvert (v8i16 MQPR:$src))), (v2i64 (MVE_VREV64_16 MQPR:$src))>;
  def : Pat<(v2i64 (bitconvert (v16i8 MQPR:$src))), (v2i64 (MVE_VREV64_8 MQPR:$src))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7553-7557
```tablegen
  def : Pat<(v4f32 (bitconvert (v2f64 MQPR:$src))), (v4f32 (MVE_VREV64_32 MQPR:$src))>;
  def : Pat<(v4f32 (bitconvert (v2i64 MQPR:$src))), (v4f32 (MVE_VREV64_32 MQPR:$src))>;
  def : Pat<(v4f32 (bitconvert (v8f16 MQPR:$src))), (v4f32 (MVE_VREV32_16 MQPR:$src))>;
  def : Pat<(v4f32 (bitconvert (v8i16 MQPR:$src))), (v4f32 (MVE_VREV32_16 MQPR:$src))>;
  def : Pat<(v4f32 (bitconvert (v16i8 MQPR:$src))), (v4f32 (MVE_VREV32_8 MQPR:$src))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7559-7563
```tablegen
  def : Pat<(v4i32 (bitconvert (v2f64 MQPR:$src))), (v4i32 (MVE_VREV64_32 MQPR:$src))>;
  def : Pat<(v4i32 (bitconvert (v2i64 MQPR:$src))), (v4i32 (MVE_VREV64_32 MQPR:$src))>;
  def : Pat<(v4i32 (bitconvert (v8f16 MQPR:$src))), (v4i32 (MVE_VREV32_16 MQPR:$src))>;
  def : Pat<(v4i32 (bitconvert (v8i16 MQPR:$src))), (v4i32 (MVE_VREV32_16 MQPR:$src))>;
  def : Pat<(v4i32 (bitconvert (v16i8 MQPR:$src))), (v4i32 (MVE_VREV32_8 MQPR:$src))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7565-7569
```tablegen
  def : Pat<(v8f16 (bitconvert (v2f64 MQPR:$src))), (v8f16 (MVE_VREV64_16 MQPR:$src))>;
  def : Pat<(v8f16 (bitconvert (v2i64 MQPR:$src))), (v8f16 (MVE_VREV64_16 MQPR:$src))>;
  def : Pat<(v8f16 (bitconvert (v4f32 MQPR:$src))), (v8f16 (MVE_VREV32_16 MQPR:$src))>;
  def : Pat<(v8f16 (bitconvert (v4i32 MQPR:$src))), (v8f16 (MVE_VREV32_16 MQPR:$src))>;
  def : Pat<(v8f16 (bitconvert (v16i8 MQPR:$src))), (v8f16 (MVE_VREV16_8 MQPR:$src))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7571-7575
```tablegen
  def : Pat<(v8i16 (bitconvert (v2f64 MQPR:$src))), (v8i16 (MVE_VREV64_16 MQPR:$src))>;
  def : Pat<(v8i16 (bitconvert (v2i64 MQPR:$src))), (v8i16 (MVE_VREV64_16 MQPR:$src))>;
  def : Pat<(v8i16 (bitconvert (v4f32 MQPR:$src))), (v8i16 (MVE_VREV32_16 MQPR:$src))>;
  def : Pat<(v8i16 (bitconvert (v4i32 MQPR:$src))), (v8i16 (MVE_VREV32_16 MQPR:$src))>;
  def : Pat<(v8i16 (bitconvert (v16i8 MQPR:$src))), (v8i16 (MVE_VREV16_8 MQPR:$src))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7577-7583
```tablegen
  def : Pat<(v16i8 (bitconvert (v2f64 MQPR:$src))), (v16i8 (MVE_VREV64_8 MQPR:$src))>;
  def : Pat<(v16i8 (bitconvert (v2i64 MQPR:$src))), (v16i8 (MVE_VREV64_8 MQPR:$src))>;
  def : Pat<(v16i8 (bitconvert (v4f32 MQPR:$src))), (v16i8 (MVE_VREV32_8 MQPR:$src))>;
  def : Pat<(v16i8 (bitconvert (v4i32 MQPR:$src))), (v16i8 (MVE_VREV32_8 MQPR:$src))>;
  def : Pat<(v16i8 (bitconvert (v8f16 MQPR:$src))), (v16i8 (MVE_VREV16_8 MQPR:$src))>;
  def : Pat<(v16i8 (bitconvert (v8i16 MQPR:$src))), (v16i8 (MVE_VREV16_8 MQPR:$src))>;
}
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
