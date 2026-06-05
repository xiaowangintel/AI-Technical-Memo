# MicroMipsInstrInfo.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MicroMipsInstrInfo.td`
- Repository: `llvm-project`
- Purpose (EN): This files describes the definitions of the microMIPSr3 instructions.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `MicroMipsInstrInfo`，涵盖指令语义、调度提示以及机器级辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===--- MicroMipsInstrFormats.td - microMIPS Inst Defs -*- tablegen -*----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This files describes the definitions of the microMIPSr3 instructions.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-16
```tablegen
def addrimm11 : ComplexPattern<iPTR, 2, "selectIntAddr11MM", [frameindex]>;
def addrimm12 : ComplexPattern<iPTR, 2, "selectIntAddr12MM", [frameindex]>;
def addrimm16 : ComplexPattern<iPTR, 2, "selectIntAddr16MM", [frameindex]>;
def addrimm4lsl2 : ComplexPattern<iPTR, 2, "selectIntAddrLSL2MM", [frameindex]>;
```
- EN: Defines TableGen record `addrimm11` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `addrimm11`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 18-21
```tablegen
def simm9_addiusp : Operand<i32> {
  let EncoderMethod = "getSImm9AddiuspValue";
  let DecoderMethod = "DecodeSimm9SP";
}
```
- EN: Defines TableGen record `simm9_addiusp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `simm9_addiusp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 23-26
```tablegen
def uimm3_shift : Operand<i32> {
  let EncoderMethod = "getUImm3Mod8Encoding";
  let DecoderMethod = "DecodePOOL16BEncodedField";
}
```
- EN: Defines TableGen record `uimm3_shift` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm3_shift`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 28-31
```tablegen
def simm3_lsa2 : Operand<i32> {
  let EncoderMethod = "getSImm3Lsa2Value";
  let DecoderMethod = "DecodeAddiur2Simm7";
}
```
- EN: Defines TableGen record `simm3_lsa2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `simm3_lsa2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 33-36
```tablegen
def uimm4_andi : Operand<i32> {
  let EncoderMethod = "getUImm4AndValue";
  let DecoderMethod = "DecodeANDI16Imm";
}
```
- EN: Defines TableGen record `uimm4_andi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm4_andi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 38-40
```tablegen
def immSExtAddiur2 : ImmLeaf<i32, [{return Imm == 1 || Imm == -1 ||
                                           ((Imm % 4 == 0) &&
                                            Imm < 28 && Imm > 0);}]>;
```
- EN: Defines TableGen record `immSExtAddiur2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immSExtAddiur2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 42-42
```tablegen
def immSExtAddius5 : ImmLeaf<i32, [{return Imm >= -8 && Imm <= 7;}]>;
```
- EN: Defines TableGen record `immSExtAddius5` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immSExtAddius5`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 44-47
```tablegen
def immZExtAndi16 : ImmLeaf<i32,
  [{return (Imm == 128 || (Imm >= 1 && Imm <= 4) || Imm == 7 || Imm == 8 ||
            Imm == 15 || Imm == 16 || Imm == 31 || Imm == 32 || Imm == 63 ||
            Imm == 64 || Imm == 255 || Imm == 32768 || Imm == 65535 );}]>;
```
- EN: Defines TableGen record `immZExtAndi16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immZExtAndi16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 49-49
```tablegen
def immZExt2Shift : ImmLeaf<i32, [{return Imm >= 1 && Imm <= 8;}]>;
```
- EN: Defines TableGen record `immZExt2Shift` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immZExt2Shift`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 51-51
```tablegen
def immLi16 : ImmLeaf<i32, [{return Imm >= -1 && Imm <= 126;}]>;
```
- EN: Defines TableGen record `immLi16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immLi16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 53-58
```tablegen
def MicroMipsMemGPRMM16AsmOperand : AsmOperandClass {
  let Name = "MicroMipsMem";
  let RenderMethod = "addMicroMipsMemOperands";
  let ParserMethod = "parseMemOperand";
  let PredicateMethod = "isMemWithGRPMM16Base";
}
```
- EN: Defines TableGen record `MicroMipsMemGPRMM16AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MicroMipsMemGPRMM16AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 60-65
```tablegen
class mem_mm_4_generic : Operand<i32> {
  let PrintMethod = "printMemOperand";
  let MIOperandInfo = (ops ptr_gpr16mm_rc, simm4);
  let OperandType = "OPERAND_MEMORY";
  let ParserMatchClass = MicroMipsMemGPRMM16AsmOperand;
}
```
- EN: Declares reusable TableGen class `mem_mm_4_generic` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `mem_mm_4_generic`，通常用于抽象共享字段、谓词或编码结构。

### Lines 67-69
```tablegen
def mem_mm_4 : mem_mm_4_generic {
  let EncoderMethod = "getMemEncodingMMImm4";
}
```
- EN: Defines TableGen record `mem_mm_4` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mem_mm_4`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 71-73
```tablegen
def mem_mm_4_lsl1 : mem_mm_4_generic {
  let EncoderMethod = "getMemEncodingMMImm4Lsl1";
}
```
- EN: Defines TableGen record `mem_mm_4_lsl1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mem_mm_4_lsl1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 75-77
```tablegen
def mem_mm_4_lsl2 : mem_mm_4_generic {
  let EncoderMethod = "getMemEncodingMMImm4Lsl2";
}
```
- EN: Defines TableGen record `mem_mm_4_lsl2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mem_mm_4_lsl2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 79-84
```tablegen
def MicroMipsMemSPAsmOperand : AsmOperandClass {
  let Name = "MicroMipsMemSP";
  let RenderMethod = "addMemOperands";
  let ParserMethod = "parseMemOperand";
  let PredicateMethod = "isMemWithUimmWordAlignedOffsetSP<7>";
}
```
- EN: Defines TableGen record `MicroMipsMemSPAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MicroMipsMemSPAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 86-91
```tablegen
def MicroMipsMemGPAsmOperand : AsmOperandClass {
  let Name = "MicroMipsMemGP";
  let RenderMethod = "addMemOperands";
  let ParserMethod = "parseMemOperand";
  let PredicateMethod = "isMemWithSimmWordAlignedOffsetGP<9>";
}
```
- EN: Defines TableGen record `MicroMipsMemGPAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MicroMipsMemGPAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 93-99
```tablegen
def mem_mm_sp_imm5_lsl2 : Operand<i32> {
  let PrintMethod = "printMemOperand";
  let MIOperandInfo = (ops ptr_sp_rc:$base, simm5:$offset);
  let OperandType = "OPERAND_MEMORY";
  let ParserMatchClass = MicroMipsMemSPAsmOperand;
  let EncoderMethod = "getMemEncodingMMSPImm5Lsl2";
}
```
- EN: Defines TableGen record `mem_mm_sp_imm5_lsl2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mem_mm_sp_imm5_lsl2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 101-107
```tablegen
def mem_mm_gp_simm7_lsl2 : Operand<i32> {
  let PrintMethod = "printMemOperand";
  let MIOperandInfo = (ops ptr_gp_rc:$base, simm7_lsl2:$offset);
  let OperandType = "OPERAND_MEMORY";
  let ParserMatchClass = MicroMipsMemGPAsmOperand;
  let EncoderMethod = "getMemEncodingMMGPImm7Lsl2";
}
```
- EN: Defines TableGen record `mem_mm_gp_simm7_lsl2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mem_mm_gp_simm7_lsl2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 109-115
```tablegen
def mem_mm_9 : Operand<i32> {
  let PrintMethod = "printMemOperand";
  let MIOperandInfo = (ops mips_ptr_rc, simm9);
  let EncoderMethod = "getMemEncodingMMImm9";
  let ParserMatchClass = MipsMemSimmAsmOperand<9>;
  let OperandType = "OPERAND_MEMORY";
}
```
- EN: Defines TableGen record `mem_mm_9` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mem_mm_9`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 117-123
```tablegen
def mem_mm_11 : Operand<i32> {
  let PrintMethod = "printMemOperand";
  let MIOperandInfo = (ops GPR32, simm11);
  let EncoderMethod = "getMemEncodingMMImm11";
  let ParserMatchClass = MipsMemSimmAsmOperand<11>;
  let OperandType = "OPERAND_MEMORY";
}
```
- EN: Defines TableGen record `mem_mm_11` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mem_mm_11`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 125-131
```tablegen
def mem_mm_12 : Operand<i32> {
  let PrintMethod = "printMemOperand";
  let MIOperandInfo = (ops mips_ptr_rc, simm12);
  let EncoderMethod = "getMemEncodingMMImm12";
  let ParserMatchClass = MipsMemAsmOperand;
  let OperandType = "OPERAND_MEMORY";
}
```
- EN: Defines TableGen record `mem_mm_12` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mem_mm_12`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 133-140
```tablegen
def mem_mm_16 : Operand<i32> {
  let PrintMethod = "printMemOperand";
  let MIOperandInfo = (ops mips_ptr_rc, simm16);
  let EncoderMethod = "getMemEncodingMMImm16";
  let DecoderMethod = "DecodeMemMMImm16";
  let ParserMatchClass = MipsMemSimmAsmOperand<16>;
  let OperandType = "OPERAND_MEMORY";
}
```
- EN: Defines TableGen record `mem_mm_16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mem_mm_16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 142-148
```tablegen
def MipsMemUimm4AsmOperand : AsmOperandClass {
  let Name = "MemOffsetUimm4";
  let SuperClasses = [MipsMemAsmOperand];
  let RenderMethod = "addMemOperands";
  let ParserMethod = "parseMemOperand";
  let PredicateMethod = "isMemWithUimmOffsetSP<6>";
}
```
- EN: Defines TableGen record `MipsMemUimm4AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MipsMemUimm4AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 150-156
```tablegen
def mem_mm_4sp : Operand<i32> {
  let PrintMethod = "printMemOperand";
  let MIOperandInfo = (ops ptr_sp_rc, uimm8);
  let EncoderMethod = "getMemEncodingMMImm4sp";
  let ParserMatchClass = MipsMemUimm4AsmOperand;
  let OperandType = "OPERAND_MEMORY";
}
```
- EN: Defines TableGen record `mem_mm_4sp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mem_mm_4sp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 158-161
```tablegen
def jmptarget_mm : Operand<OtherVT> {
  let EncoderMethod = "getJumpTargetOpValueMM";
  let PrintMethod = "printJumpOperand";
}
```
- EN: Defines TableGen record `jmptarget_mm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `jmptarget_mm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 163-166
```tablegen
def calltarget_mm : Operand<iPTR> {
  let EncoderMethod = "getJumpTargetOpValueMM";
  let PrintMethod = "printJumpOperand";
}
```
- EN: Defines TableGen record `calltarget_mm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `calltarget_mm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 168-174
```tablegen
def brtarget7_mm : Operand<OtherVT> {
  let EncoderMethod = "getBranchTarget7OpValueMM";
  let OperandType   = "OPERAND_PCREL";
  let DecoderMethod = "DecodeBranchTarget7MM";
  let ParserMatchClass = MipsJumpTargetAsmOperand;
  let PrintMethod = "printBranchOperand";
}
```
- EN: Defines TableGen record `brtarget7_mm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `brtarget7_mm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 176-182
```tablegen
def brtarget10_mm : Operand<OtherVT> {
  let EncoderMethod = "getBranchTargetOpValueMMPC10";
  let OperandType   = "OPERAND_PCREL";
  let DecoderMethod = "DecodeBranchTarget10MM";
  let ParserMatchClass = MipsJumpTargetAsmOperand;
  let PrintMethod = "printBranchOperand";
}
```
- EN: Defines TableGen record `brtarget10_mm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `brtarget10_mm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 184-190
```tablegen
def brtarget_mm : Operand<OtherVT> {
  let EncoderMethod = "getBranchTargetOpValueMM";
  let OperandType   = "OPERAND_PCREL";
  let DecoderMethod = "DecodeBranchTargetMM";
  let ParserMatchClass = MipsJumpTargetAsmOperand;
  let PrintMethod = "printBranchOperand";
}
```
- EN: Defines TableGen record `brtarget_mm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `brtarget_mm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 192-195
```tablegen
def simm23_lsl2 : Operand<i32> {
  let EncoderMethod = "getSimm23Lsl2Encoding";
  let DecoderMethod = "DecodeSimm23Lsl2";
}
```
- EN: Defines TableGen record `simm23_lsl2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `simm23_lsl2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 197-204
```tablegen
class CompactBranchMM<string opstr, DAGOperand opnd, RegisterOperand RO> :
  InstSE<(outs), (ins RO:$rs, opnd:$offset),
         !strconcat(opstr, "\t$rs, $offset"), [], FrmI> {
  let isBranch = 1;
  let isTerminator = 1;
  let hasDelaySlot = 0;
  let Defs = [AT];
}
```
- EN: Declares reusable TableGen class `CompactBranchMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `CompactBranchMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 206-218
```tablegen
let canFoldAsLoad = 1 in
class LoadLeftRightMM<string opstr, SDNode OpNode, RegisterOperand RO,
                      Operand MemOpnd> :
  InstSE<(outs RO:$rt), (ins MemOpnd:$addr, RO:$src),
         !strconcat(opstr, "\t$rt, $addr"),
         [(set RO:$rt, (OpNode addrimm12:$addr, RO:$src))],
         FrmI> {
  let DecoderMethod = "DecodeMemMMImm12";
  string Constraints = "$src = $rt";
  let BaseOpcode = opstr;
  bit mayLoad = 1;
  bit mayStore = 0;
}
```
- EN: Declares reusable TableGen class `LoadLeftRightMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `LoadLeftRightMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 220-229
```tablegen
class StoreLeftRightMM<string opstr, SDNode OpNode, RegisterOperand RO,
                       Operand MemOpnd>:
  InstSE<(outs), (ins RO:$rt, MemOpnd:$addr),
         !strconcat(opstr, "\t$rt, $addr"),
         [(OpNode RO:$rt, addrimm12:$addr)], FrmI> {
  let DecoderMethod = "DecodeMemMMImm12";
  let BaseOpcode = opstr;
  bit mayLoad = 0;
  bit mayStore = 1;
}
```
- EN: Declares reusable TableGen class `StoreLeftRightMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `StoreLeftRightMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 231-239
```tablegen
class MovePMM16<string opstr, RegisterOperand RO1, RegisterOperand RO2,
                RegisterOperand RO3> :
MicroMipsInst16<(outs RO1:$rd1, RO2:$rd2), (ins RO3:$rs, RO3:$rt),
                 !strconcat(opstr, "\t$rd1, $rd2, $rs, $rt"), [],
                 FrmR> {
  let isReMaterializable = 1;
  let isMoveReg = 1;
  let DecoderMethod = "DecodeMovePOperands";
}
```
- EN: Declares reusable TableGen class `MovePMM16` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `MovePMM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 241-247
```tablegen
class StorePairMM<string opstr>
    :  InstSE<(outs), (ins GPR32Opnd:$rt, GPR32Opnd:$rt2, mem_simm12:$addr),
         !strconcat(opstr, "\t$rt, $addr"), [], FrmI, opstr> {
  let DecoderMethod = "DecodeMemMMImm12";
  let mayStore = 1;
  let AsmMatchConverter = "ConvertXWPOperands";
}
```
- EN: Declares reusable TableGen class `StorePairMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `StorePairMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 249-255
```tablegen
class LoadPairMM<string opstr>
    : InstSE<(outs GPR32Opnd:$rt, GPR32Opnd:$rt2), (ins mem_simm12:$addr),
          !strconcat(opstr, "\t$rt, $addr"), [], FrmI, opstr> {
  let DecoderMethod = "DecodeMemMMImm12";
  let mayLoad = 1;
  let AsmMatchConverter = "ConvertXWPOperands";
}
```
- EN: Declares reusable TableGen class `LoadPairMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `LoadPairMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 257-262
```tablegen
class LLBaseMM<string opstr, RegisterOperand RO> :
  InstSE<(outs RO:$rt), (ins mem_mm_12:$addr),
         !strconcat(opstr, "\t$rt, $addr"), [], FrmI> {
  let DecoderMethod = "DecodeMemMMImm12";
  let mayLoad = 1;
}
```
- EN: Declares reusable TableGen class `LLBaseMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `LLBaseMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 264-270
```tablegen
class LLEBaseMM<string opstr, RegisterOperand RO> :
  InstSE<(outs RO:$rt), (ins mem_simm9:$addr),
         !strconcat(opstr, "\t$rt, $addr"), [], FrmI> {
  let DecoderMethod = "DecodeMemMMImm9";
  string BaseOpcode = opstr;
  let mayLoad = 1;
}
```
- EN: Declares reusable TableGen class `LLEBaseMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `LLEBaseMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 272-278
```tablegen
class SCBaseMM<string opstr, RegisterOperand RO> :
  InstSE<(outs RO:$dst), (ins RO:$rt, mem_mm_12:$addr),
         !strconcat(opstr, "\t$rt, $addr"), [], FrmI> {
  let DecoderMethod = "DecodeMemMMImm12";
  let mayStore = 1;
  let Constraints = "$rt = $dst";
}
```
- EN: Declares reusable TableGen class `SCBaseMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `SCBaseMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 280-287
```tablegen
class SCEBaseMM<string opstr, RegisterOperand RO> :
  InstSE<(outs RO:$dst), (ins RO:$rt, mem_simm9:$addr),
         !strconcat(opstr, "\t$rt, $addr"), [], FrmI> {
  let DecoderMethod = "DecodeMemMMImm9";
  string BaseOpcode = opstr;
  let mayStore = 1;
  let Constraints = "$rt = $dst";
}
```
- EN: Declares reusable TableGen class `SCEBaseMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `SCEBaseMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 289-297
```tablegen
class LoadMM<string opstr, DAGOperand RO, SDPatternOperator OpNode = null_frag,
             DAGOperand MO = mem_mm_12> :
  InstSE<(outs RO:$rt), (ins MO:$addr),
         !strconcat(opstr, "\t$rt, $addr"),
         [(set RO:$rt, (OpNode addrimm12:$addr))], FrmI, opstr> {
  let DecoderMethod = "DecodeMemMMImm12";
  let canFoldAsLoad = 1;
  let mayLoad = 1;
}
```
- EN: Declares reusable TableGen class `LoadMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `LoadMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 299-305
```tablegen
class ArithRMM16<string opstr, RegisterOperand RO, bit isComm = 0,
                 SDPatternOperator OpNode = null_frag> :
  MicroMipsInst16<(outs RO:$rd), (ins RO:$rs, RO:$rt),
                  !strconcat(opstr, "\t$rd, $rs, $rt"),
                  [(set RO:$rd, (OpNode RO:$rs, RO:$rt))], FrmR> {
  let isCommutable = isComm;
}
```
- EN: Declares reusable TableGen class `ArithRMM16` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `ArithRMM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 307-309
```tablegen
class AndImmMM16<string opstr, RegisterOperand RO> :
  MicroMipsInst16<(outs RO:$rd), (ins RO:$rs, uimm4_andi:$imm),
                  !strconcat(opstr, "\t$rd, $rs, $imm"), [], FrmI>;
```
- EN: Declares reusable TableGen class `AndImmMM16` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `AndImmMM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 311-318
```tablegen
class LogicRMM16<string opstr, RegisterOperand RO,
                 SDPatternOperator OpNode = null_frag> :
  MicroMipsInst16<(outs RO:$dst), (ins RO:$rs, RO:$rt),
         !strconcat(opstr, "\t$rt, $rs"),
         [(set RO:$dst, (OpNode RO:$rs, RO:$rt))], FrmR> {
  let isCommutable = 1;
  let Constraints = "$rt = $dst";
}
```
- EN: Declares reusable TableGen class `LogicRMM16` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `LogicRMM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 320-323
```tablegen
class NotMM16<string opstr, RegisterOperand RO> :
  MicroMipsInst16<(outs RO:$rt), (ins RO:$rs),
         !strconcat(opstr, "\t$rt, $rs"),
         [(set RO:$rt, (not RO:$rs))], FrmR>;
```
- EN: Declares reusable TableGen class `NotMM16` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `NotMM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 325-327
```tablegen
class ShiftIMM16<string opstr, Operand ImmOpnd, RegisterOperand RO> :
  MicroMipsInst16<(outs RO:$rd), (ins RO:$rt, ImmOpnd:$shamt),
                  !strconcat(opstr, "\t$rd, $rt, $shamt"), [], FrmR>;
```
- EN: Declares reusable TableGen class `ShiftIMM16` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `ShiftIMM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 329-335
```tablegen
class LoadMM16<string opstr, DAGOperand RO, Operand MemOpnd> :
  MicroMipsInst16<(outs RO:$rt), (ins MemOpnd:$addr),
                  !strconcat(opstr, "\t$rt, $addr"), [], FrmI> {
  let DecoderMethod = "DecodeMemMMImm4";
  let canFoldAsLoad = 1;
  let mayLoad = 1;
}
```
- EN: Declares reusable TableGen class `LoadMM16` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `LoadMM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 337-342
```tablegen
class StoreMM16<string opstr, DAGOperand RTOpnd, Operand MemOpnd> :
  MicroMipsInst16<(outs), (ins RTOpnd:$rt, MemOpnd:$addr),
                  !strconcat(opstr, "\t$rt, $addr"), [], FrmI> {
  let DecoderMethod = "DecodeMemMMImm4";
  let mayStore = 1;
}
```
- EN: Declares reusable TableGen class `StoreMM16` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `StoreMM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 344-350
```tablegen
class LoadSPMM16<string opstr, DAGOperand RO, Operand MemOpnd> :
  MicroMipsInst16<(outs RO:$rt), (ins MemOpnd:$offset),
                  !strconcat(opstr, "\t$rt, $offset"), [], FrmI> {
  let DecoderMethod = "DecodeMemMMSPImm5Lsl2";
  let canFoldAsLoad = 1;
  let mayLoad = 1;
}
```
- EN: Declares reusable TableGen class `LoadSPMM16` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `LoadSPMM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 352-357
```tablegen
class StoreSPMM16<string opstr, DAGOperand RO, Operand MemOpnd> :
  MicroMipsInst16<(outs), (ins RO:$rt, MemOpnd:$offset),
                  !strconcat(opstr, "\t$rt, $offset"), [], FrmI> {
  let DecoderMethod = "DecodeMemMMSPImm5Lsl2";
  let mayStore = 1;
}
```
- EN: Declares reusable TableGen class `StoreSPMM16` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `StoreSPMM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 359-365
```tablegen
class LoadGPMM16<string opstr, DAGOperand RO, Operand MemOpnd> :
  MicroMipsInst16<(outs RO:$rt), (ins MemOpnd:$offset),
                  !strconcat(opstr, "\t$rt, $offset"), [], FrmI> {
  let DecoderMethod = "DecodeMemMMGPImm7Lsl2";
  let canFoldAsLoad = 1;
  let mayLoad = 1;
}
```
- EN: Declares reusable TableGen class `LoadGPMM16` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `LoadGPMM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 367-372
```tablegen
class AddImmUR2<string opstr, RegisterOperand RO> :
  MicroMipsInst16<(outs RO:$rd), (ins RO:$rs, simm3_lsa2:$imm),
                  !strconcat(opstr, "\t$rd, $rs, $imm"),
                  [], FrmR> {
  let isCommutable = 1;
}
```
- EN: Declares reusable TableGen class `AddImmUR2` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `AddImmUR2`，通常用于抽象共享字段、谓词或编码结构。

### Lines 374-378
```tablegen
class AddImmUS5<string opstr, RegisterOperand RO> :
  MicroMipsInst16<(outs RO:$dst), (ins RO:$rd, simm4:$imm),
                  !strconcat(opstr, "\t$rd, $imm"), [], FrmR> {
  let Constraints = "$rd = $dst";
}
```
- EN: Declares reusable TableGen class `AddImmUS5` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `AddImmUS5`，通常用于抽象共享字段、谓词或编码结构。

### Lines 380-382
```tablegen
class AddImmUR1SP<string opstr, RegisterOperand RO> :
  MicroMipsInst16<(outs RO:$rd), (ins uimm6_lsl2:$imm),
                  !strconcat(opstr, "\t$rd, $imm"), [], FrmR>;
```
- EN: Declares reusable TableGen class `AddImmUR1SP` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `AddImmUR1SP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 384-386
```tablegen
class AddImmUSP<string opstr> :
  MicroMipsInst16<(outs), (ins simm9_addiusp:$imm),
                  !strconcat(opstr, "\t$imm"), [], FrmI>;
```
- EN: Declares reusable TableGen class `AddImmUSP` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `AddImmUSP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 388-394
```tablegen
class MoveFromHILOMM<string opstr, RegisterOperand RO, Register UseReg> :
      MicroMipsInst16<(outs RO:$rd), (ins), !strconcat(opstr, "\t$rd"),
  [], FrmR> {
  let Uses = [UseReg];
  let hasSideEffects = 0;
  let isMoveReg = 1;
}
```
- EN: Declares reusable TableGen class `MoveFromHILOMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `MoveFromHILOMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 396-401
```tablegen
class MoveMM16<string opstr, RegisterOperand RO>
    :  MicroMipsInst16<(outs RO:$rd), (ins RO:$rs),
                       !strconcat(opstr, "\t$rd, $rs"), [], FrmR> {
  let isReMaterializable = 1;
  let isMoveReg = 1;
}
```
- EN: Declares reusable TableGen class `MoveMM16` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `MoveMM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 403-407
```tablegen
class LoadImmMM16<string opstr, Operand Od, RegisterOperand RO> :
  MicroMipsInst16<(outs RO:$rd), (ins Od:$imm),
                  !strconcat(opstr, "\t$rd, $imm"), [], FrmI> {
  let isReMaterializable = 1;
}
```
- EN: Declares reusable TableGen class `LoadImmMM16` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `LoadImmMM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 409-417
```tablegen
// 16-bit Jump and Link (Call)
class JumpLinkRegMM16<string opstr, RegisterOperand RO> :
  MicroMipsInst16<(outs), (ins RO:$rs), !strconcat(opstr, "\t$rs"),
           [(MipsJmpLink RO:$rs)], FrmR> {
  let isCall = 1;
  let hasDelaySlot = 1;
  let Defs = [RA];
  let hasPostISelHook = 1;
}
```
- EN: Declares reusable TableGen class `JumpLinkRegMM16` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `JumpLinkRegMM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 419-426
```tablegen
// 16-bit Jump Reg
class JumpRegMM16<string opstr, RegisterOperand RO> :
  MicroMipsInst16<(outs), (ins RO:$rs), !strconcat(opstr, "\t$rs"),
           [], FrmR> {
  let hasDelaySlot = 1;
  let isBranch = 1;
  let isIndirectBranch = 1;
}
```
- EN: Declares reusable TableGen class `JumpRegMM16` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `JumpRegMM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 428-436
```tablegen
// Base class for JRADDIUSP instruction.
class JumpRAddiuStackMM16 :
  MicroMipsInst16<(outs), (ins uimm5_lsl2:$imm), "jraddiusp\t$imm",
                  [], FrmR> {
  let isTerminator = 1;
  let isBarrier = 1;
  let isBranch = 1;
  let isIndirectBranch = 1;
}
```
- EN: Declares reusable TableGen class `for` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `for`，通常用于抽象共享字段、谓词或编码结构。

### Lines 438-445
```tablegen
// 16-bit Jump and Link (Call) - Short Delay Slot
class JumpLinkRegSMM16<string opstr, RegisterOperand RO> :
  MicroMipsInst16<(outs), (ins RO:$rs), !strconcat(opstr, "\t$rs"),
           [], FrmR> {
  let isCall = 1;
  let hasDelaySlot = 1;
  let Defs = [RA];
}
```
- EN: Declares reusable TableGen class `JumpLinkRegSMM16` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `JumpLinkRegSMM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 447-455
```tablegen
// 16-bit Jump Register Compact - No delay slot
class JumpRegCMM16<string opstr, RegisterOperand RO> :
  MicroMipsInst16<(outs), (ins RO:$rs), !strconcat(opstr, "\t$rs"),
                  [], FrmR> {
  let isTerminator = 1;
  let isBarrier = 1;
  let isBranch = 1;
  let isIndirectBranch = 1;
}
```
- EN: Declares reusable TableGen class `JumpRegCMM16` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `JumpRegCMM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 457-461
```tablegen
// Break16 and Sdbbp16
class BrkSdbbp16MM<string opstr> :
  MicroMipsInst16<(outs), (ins uimm4:$code_),
                  !strconcat(opstr, "\t$code_"),
                  [], FrmOther>;
```
- EN: Declares reusable TableGen class `BrkSdbbp16MM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `BrkSdbbp16MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 463-470
```tablegen
class CBranchZeroMM<string opstr, DAGOperand opnd, RegisterOperand RO> :
  MicroMipsInst16<(outs), (ins RO:$rs, opnd:$offset),
                  !strconcat(opstr, "\t$rs, $offset"), [], FrmI> {
  let isBranch = 1;
  let isTerminator = 1;
  let hasDelaySlot = 1;
  let Defs = [AT];
}
```
- EN: Declares reusable TableGen class `CBranchZeroMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `CBranchZeroMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 472-478
```tablegen
// MicroMIPS Jump and Link (Call) - Short Delay Slot
let isCall = 1, hasDelaySlot = 1, Defs = [RA] in {
  class JumpLinkMM<string opstr, DAGOperand opnd> :
    InstSE<(outs), (ins opnd:$target), !strconcat(opstr, "\t$target"),
           [], FrmJ, opstr> {
    let DecoderMethod = "DecodeJumpTargetMM";
  }
```
- EN: Declares reusable TableGen class `JumpLinkMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `JumpLinkMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 480-482
```tablegen
  class JumpLinkRegMM<string opstr, RegisterOperand RO>:
    InstSE<(outs RO:$rd), (ins RO:$rs), !strconcat(opstr, "\t$rd, $rs"),
            [], FrmR>;
```
- EN: Declares reusable TableGen class `JumpLinkRegMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `JumpLinkRegMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 484-488
```tablegen
  class BranchCompareToZeroLinkMM<string opstr, DAGOperand opnd,
                                  RegisterOperand RO> :
    InstSE<(outs), (ins RO:$rs, opnd:$offset),
           !strconcat(opstr, "\t$rs, $offset"), [], FrmI, opstr>;
}
```
- EN: Declares reusable TableGen class `BranchCompareToZeroLinkMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `BranchCompareToZeroLinkMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 490-492
```tablegen
class LoadWordIndexedScaledMM<string opstr, RegisterOperand RO> :
  InstSE<(outs RO:$rd), (ins PtrRC:$base, PtrRC:$index),
         !strconcat(opstr, "\t$rd, ${index}(${base})"), [], FrmFI>;
```
- EN: Declares reusable TableGen class `LoadWordIndexedScaledMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `LoadWordIndexedScaledMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 494-497
```tablegen
class PrefetchIndexed<string opstr> :
  InstSE<(outs), (ins PtrRC:$base, PtrRC:$index, uimm5:$hint),
         !strconcat(opstr, "\t$hint, ${index}(${base})"),
         [], FrmOther>;
```
- EN: Declares reusable TableGen class `PrefetchIndexed` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `PrefetchIndexed`，通常用于抽象共享字段、谓词或编码结构。

### Lines 499-501
```tablegen
class AddImmUPC<string opstr, RegisterOperand RO> :
  InstSE<(outs RO:$rs), (ins simm23_lsl2:$imm),
         !strconcat(opstr, "\t$rs, $imm"), [], FrmR>;
```
- EN: Declares reusable TableGen class `AddImmUPC` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `AddImmUPC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 503-507
```tablegen
/// A list of registers used by load/store multiple instructions.
def RegListAsmOperand : AsmOperandClass {
  let Name = "RegList";
  let ParserMethod = "parseRegisterList";
}
```
- EN: Defines TableGen record `RegListAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RegListAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 509-514
```tablegen
def reglist : Operand<i32> {
  let EncoderMethod = "getRegisterListOpValue";
  let ParserMatchClass = RegListAsmOperand;
  let PrintMethod = "printRegisterList";
  let DecoderMethod = "DecodeRegListOperand";
}
```
- EN: Defines TableGen record `reglist` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `reglist`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 516-521
```tablegen
def RegList16AsmOperand : AsmOperandClass {
  let Name = "RegList16";
  let ParserMethod = "parseRegisterList";
  let PredicateMethod = "isRegList16";
  let RenderMethod = "addRegListOperands";
}
```
- EN: Defines TableGen record `RegList16AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RegList16AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 523-528
```tablegen
def reglist16 : Operand<i32> {
  let EncoderMethod = "getRegisterListOpValue16";
  let DecoderMethod = "DecodeRegListOperand16";
  let PrintMethod = "printRegisterList";
  let ParserMatchClass = RegList16AsmOperand;
}
```
- EN: Defines TableGen record `reglist16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `reglist16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 530-535
```tablegen
class StoreMultMM<string opstr> :
  InstSE<(outs), (ins reglist:$rt, mem_mm_12:$addr),
         !strconcat(opstr, "\t$rt, $addr"), [], FrmI, opstr> {
  let DecoderMethod = "DecodeMemMMImm12";
  let mayStore = 1;
}
```
- EN: Declares reusable TableGen class `StoreMultMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `StoreMultMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 537-542
```tablegen
class LoadMultMM<string opstr> :
  InstSE<(outs reglist:$rt), (ins mem_mm_12:$addr),
          !strconcat(opstr, "\t$rt, $addr"), [], FrmI, opstr> {
  let DecoderMethod = "DecodeMemMMImm12";
  let mayLoad = 1;
}
```
- EN: Declares reusable TableGen class `LoadMultMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `LoadMultMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 544-549
```tablegen
class StoreMultMM16<string opstr> :
  MicroMipsInst16<(outs), (ins reglist16:$rt, mem_mm_4sp:$addr),
                  !strconcat(opstr, "\t$rt, $addr"), [], FrmI> {
  let DecoderMethod = "DecodeMemMMReglistImm4Lsl2";
  let mayStore = 1;
}
```
- EN: Declares reusable TableGen class `StoreMultMM16` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `StoreMultMM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 551-556
```tablegen
class LoadMultMM16<string opstr> :
  MicroMipsInst16<(outs reglist16:$rt), (ins mem_mm_4sp:$addr),
                  !strconcat(opstr, "\t$rt, $addr"), [], FrmI> {
  let DecoderMethod = "DecodeMemMMReglistImm4Lsl2";
  let mayLoad = 1;
}
```
- EN: Declares reusable TableGen class `LoadMultMM16` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `LoadMultMM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 558-568
```tablegen
class UncondBranchMM16<string opstr> :
  MicroMipsInst16<(outs), (ins brtarget10_mm:$offset),
                  !strconcat(opstr, "\t$offset"),
                  [], FrmI> {
  let isBranch = 1;
  let isTerminator = 1;
  let isBarrier = 1;
  let hasDelaySlot = 1;
  let Predicates = [RelocPIC, InMicroMips];
  let Defs = [AT];
}
```
- EN: Declares reusable TableGen class `UncondBranchMM16` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `UncondBranchMM16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 570-574
```tablegen
class HypcallMM<string opstr> :
  InstSE<(outs), (ins uimm10:$code_),
          !strconcat(opstr, "\t$code_"), [], FrmOther> {
  let BaseOpcode = opstr;
}
```
- EN: Declares reusable TableGen class `HypcallMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `HypcallMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 576-579
```tablegen
class TLBINVMM<string opstr> :
  InstSE<(outs), (ins), opstr, [], FrmOther> {
  let BaseOpcode = opstr;
}
```
- EN: Declares reusable TableGen class `TLBINVMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `TLBINVMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 581-586
```tablegen
class MfCop0MM<string opstr, RegisterOperand DstRC,
               RegisterOperand SrcRC> :
  InstSE<(outs DstRC:$rt), (ins SrcRC:$rs, uimm3:$sel),
          !strconcat(opstr, "\t$rt, $rs, $sel"), [], FrmR> {
  let BaseOpcode = opstr;
}
```
- EN: Declares reusable TableGen class `MfCop0MM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `MfCop0MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 588-593
```tablegen
class MtCop0MM<string opstr, RegisterOperand DstRC,
               RegisterOperand SrcRC> :
  InstSE<(outs DstRC:$rs), (ins SrcRC:$rt, uimm3:$sel),
          !strconcat(opstr, "\t$rt, $rs, $sel"), [], FrmR> {
  let BaseOpcode = opstr;
}
```
- EN: Declares reusable TableGen class `MtCop0MM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `MtCop0MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 595-600
```tablegen
let FastISelShouldIgnore = 1 in {
  def ADDU16_MM : ArithRMM16<"addu16", GPRMM16Opnd, 1, add>,
      ARITH_FM_MM16<0>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def AND16_MM : LogicRMM16<"and16", GPRMM16Opnd, and>,
      LOGIC_FM_MM16<0x2>, ISA_MICROMIPS32_NOT_MIPS32R6;
}
```
- EN: Defines TableGen record `ADDU16_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ADDU16_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 602-612
```tablegen
def ANDI16_MM : AndImmMM16<"andi16", GPRMM16Opnd>, ANDI_FM_MM16<0x0b>,
                ISA_MICROMIPS32_NOT_MIPS32R6;
def NOT16_MM : NotMM16<"not16", GPRMM16Opnd>, LOGIC_FM_MM16<0x0>,
               ISA_MICROMIPS32_NOT_MIPS32R6;
let FastISelShouldIgnore = 1 in
  def OR16_MM : LogicRMM16<"or16", GPRMM16Opnd, or>, LOGIC_FM_MM16<0x3>,
                ISA_MICROMIPS32_NOT_MIPS32R6;
def SLL16_MM : ShiftIMM16<"sll16", uimm3_shift, GPRMM16Opnd>,
    SHIFT_FM_MM16<0>, ISA_MICROMIPS32_NOT_MIPS32R6;
def SRL16_MM : ShiftIMM16<"srl16", uimm3_shift, GPRMM16Opnd>,
    SHIFT_FM_MM16<1>, ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `ANDI16_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ANDI16_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 614-631
```tablegen
let FastISelShouldIgnore = 1 in {
  def SUBU16_MM : ArithRMM16<"subu16", GPRMM16Opnd, 0, sub>,
                  ARITH_FM_MM16<1>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def XOR16_MM : LogicRMM16<"xor16", GPRMM16Opnd, xor>,
                 LOGIC_FM_MM16<0x1>, ISA_MICROMIPS32_NOT_MIPS32R6;
}
def LBU16_MM : LoadMM16<"lbu16", GPRMM16Opnd, mem_mm_4>,
               LOAD_STORE_FM_MM16<0x02>, ISA_MICROMIPS;
def LHU16_MM : LoadMM16<"lhu16", GPRMM16Opnd, mem_mm_4_lsl1>,
               LOAD_STORE_FM_MM16<0x0a>, ISA_MICROMIPS;
def LW16_MM : LoadMM16<"lw16", GPRMM16Opnd, mem_mm_4_lsl2>,
                      LOAD_STORE_FM_MM16<0x1a>, ISA_MICROMIPS;
def SB16_MM : StoreMM16<"sb16", GPRMM16OpndZero, mem_mm_4>,
              LOAD_STORE_FM_MM16<0x22>,
              ISA_MICROMIPS32_NOT_MIPS32R6;
def SH16_MM : StoreMM16<"sh16", GPRMM16OpndZero, mem_mm_4_lsl1>,
              LOAD_STORE_FM_MM16<0x2a>,
              ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `SUBU16_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SUBU16_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 632-649
```tablegen
def SW16_MM : StoreMM16<"sw16", GPRMM16OpndZero, mem_mm_4_lsl2>,
              LOAD_STORE_FM_MM16<0x3a>,
              ISA_MICROMIPS32_NOT_MIPS32R6;
def LWGP_MM : LoadGPMM16<"lw", GPRMM16Opnd, mem_mm_gp_simm7_lsl2>,
                         LOAD_GP_FM_MM16<0x19>, ISA_MICROMIPS;
def LWSP_MM : LoadSPMM16<"lw", GPR32Opnd, mem_mm_sp_imm5_lsl2>,
              LOAD_STORE_SP_FM_MM16<0x12>, ISA_MICROMIPS;
def SWSP_MM : StoreSPMM16<"swsp", GPR32Opnd, mem_mm_sp_imm5_lsl2>,
              LOAD_STORE_SP_FM_MM16<0x32>, ISA_MICROMIPS32_NOT_MIPS32R6;
def ADDIUR1SP_MM : AddImmUR1SP<"addiur1sp", GPRMM16Opnd>, ADDIUR1SP_FM_MM16,
                   ISA_MICROMIPS;
def ADDIUR2_MM : AddImmUR2<"addiur2", GPRMM16Opnd>, ADDIUR2_FM_MM16,
                 ISA_MICROMIPS;
def ADDIUS5_MM : AddImmUS5<"addius5", GPR32Opnd>, ADDIUS5_FM_MM16,
                 ISA_MICROMIPS;
def ADDIUSP_MM : AddImmUSP<"addiusp">, ADDIUSP_FM_MM16, ISA_MICROMIPS;
def MFHI16_MM : MoveFromHILOMM<"mfhi16", GPR32Opnd, AC0>,
                MFHILO_FM_MM16<0x10>, ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `SW16_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SW16_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 650-667
```tablegen
def MFLO16_MM : MoveFromHILOMM<"mflo16", GPR32Opnd, AC0>,
                MFHILO_FM_MM16<0x12>, ISA_MICROMIPS32_NOT_MIPS32R6;
def MOVE16_MM : MoveMM16<"move", GPR32Opnd>, MOVE_FM_MM16<0x03>,
                ISA_MICROMIPS32_NOT_MIPS32R6;
def MOVEP_MM : MovePMM16<"movep", GPRMM16OpndMovePPairFirst,
                         GPRMM16OpndMovePPairSecond, GPRMM16OpndMoveP>,
               MOVEP_FM_MM16, ISA_MICROMIPS32_NOT_MIPS32R6;
def LI16_MM : LoadImmMM16<"li16", li16_imm, GPRMM16Opnd>, LI_FM_MM16,
              IsAsCheapAsAMove, ISA_MICROMIPS32_NOT_MIPS32R6;
def JALR16_MM : JumpLinkRegMM16<"jalr", GPR32Opnd>, JALR_FM_MM16<0x0e>,
                ISA_MICROMIPS32_NOT_MIPS32R6;
def JALRS16_MM : JumpLinkRegSMM16<"jalrs16", GPR32Opnd>, JALR_FM_MM16<0x0f>,
                 ISA_MICROMIPS32_NOT_MIPS32R6;
def JRC16_MM : JumpRegCMM16<"jrc", GPR32Opnd>, JALR_FM_MM16<0x0d>,
               ISA_MICROMIPS32_NOT_MIPS32R6;
def JRADDIUSP : JumpRAddiuStackMM16, JRADDIUSP_FM_MM16<0x18>,
                ISA_MICROMIPS32_NOT_MIPS32R6;
def JR16_MM : JumpRegMM16<"jr16", GPR32Opnd>, JALR_FM_MM16<0x0c>,
```
- EN: Defines TableGen record `MFLO16_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MFLO16_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 668-677
```tablegen
              ISA_MICROMIPS32_NOT_MIPS32R6;
def BEQZ16_MM : CBranchZeroMM<"beqz16", brtarget7_mm, GPRMM16Opnd>,
                BEQNEZ_FM_MM16<0x23>, ISA_MICROMIPS32_NOT_MIPS32R6;
def BNEZ16_MM : CBranchZeroMM<"bnez16", brtarget7_mm, GPRMM16Opnd>,
                BEQNEZ_FM_MM16<0x2b>, ISA_MICROMIPS32_NOT_MIPS32R6;
def B16_MM : UncondBranchMM16<"b16">, B16_FM, ISA_MICROMIPS32_NOT_MIPS32R6;
def BREAK16_MM : BrkSdbbp16MM<"break16">, BRKSDBBP16_FM_MM<0x28>,
                 ISA_MICROMIPS32_NOT_MIPS32R6;
def SDBBP16_MM : BrkSdbbp16MM<"sdbbp16">, BRKSDBBP16_FM_MM<0x2C>,
                 ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `BEQZ16_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BEQZ16_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 679-681
```tablegen
class WaitMM<string opstr> :
  InstSE<(outs), (ins uimm10:$code_), !strconcat(opstr, "\t$code_"), [],
         FrmOther, opstr>;
```
- EN: Declares reusable TableGen class `WaitMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `WaitMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 683-694
```tablegen
let DecoderNamespace = "MicroMips" in {
  /// Load and Store Instructions - multiple
  def SWM16_MM : StoreMultMM16<"swm16">, LWM_FM_MM16<0x5>,
                 ISA_MICROMIPS32_NOT_MIPS32R6;
  def LWM16_MM : LoadMultMM16<"lwm16">, LWM_FM_MM16<0x4>,
                 ISA_MICROMIPS32_NOT_MIPS32R6;
  def CFC2_MM : InstSE<(outs GPR32Opnd:$rt), (ins COP2Opnd:$impl),
                       "cfc2\t$rt, $impl", [], FrmFR, "cfc2">,
                POOL32A_CFTC2_FM_MM<0b1100110100>, ISA_MICROMIPS;
  def CTC2_MM : InstSE<(outs COP2Opnd:$impl), (ins GPR32Opnd:$rt),
                       "ctc2\t$rt, $impl", [], FrmFR, "ctc2">,
                POOL32A_CFTC2_FM_MM<0b1101110100>, ISA_MICROMIPS;
```
- EN: Defines TableGen record `SWM16_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SWM16_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 696-700
```tablegen
  /// Compact Branch Instructions
  def BEQZC_MM : CompactBranchMM<"beqzc", brtarget_mm, GPR32Opnd>,
                 COMPACT_BRANCH_FM_MM<0x7>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def BNEZC_MM : CompactBranchMM<"bnezc", brtarget_mm, GPR32Opnd>,
                 COMPACT_BRANCH_FM_MM<0x5>, ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `BEQZC_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BEQZC_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 702-719
```tablegen
  /// Arithmetic Instructions (ALU Immediate)
  def ADDiu_MM : MMRel, ArithLogicI<"addiu", simm16, GPR32Opnd>,
                 ADDI_FM_MM<0xc>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def ADDi_MM  : MMRel, ArithLogicI<"addi", simm16, GPR32Opnd>,
                 ADDI_FM_MM<0x4>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def SLTi_MM  : MMRel, SetCC_I<"slti", setlt, simm16, immSExt16, GPR32Opnd>,
                 SLTI_FM_MM<0x24>, ISA_MICROMIPS;
  def SLTiu_MM : MMRel, SetCC_I<"sltiu", setult, simm16, immSExt16, GPR32Opnd>,
                 SLTI_FM_MM<0x2c>, ISA_MICROMIPS;
  def ANDi_MM  : MMRel, ArithLogicI<"andi", uimm16, GPR32Opnd>,
                 ADDI_FM_MM<0x34>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def ORi_MM   : MMRel, ArithLogicI<"ori", uimm16, GPR32Opnd, immZExt16,
                                    or>, ADDI_FM_MM<0x14>,
                 ISA_MICROMIPS32_NOT_MIPS32R6;
  def XORi_MM  : MMRel, ArithLogicI<"xori", uimm16, GPR32Opnd,
                                    immZExt16, xor>, ADDI_FM_MM<0x1c>,
                 ISA_MICROMIPS32_NOT_MIPS32R6;
  def LUi_MM   : MMRel, LoadUpper<"lui", GPR32Opnd, uimm16_relaxed>, LUI_FM_MM,
```
- EN: Defines TableGen record `ADDiu_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ADDiu_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 720-720
```tablegen
                 ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 722-723
```tablegen
  def LEA_ADDiu_MM : MMRel, EffectiveAddress<"addiu", GPR32Opnd>,
                     LW_FM_MM<0xc>, ISA_MICROMIPS;
```
- EN: Defines TableGen record `LEA_ADDiu_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LEA_ADDiu_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 725-742
```tablegen
  /// Arithmetic Instructions (3-Operand, R-Type)
  def ADDu_MM  : MMRel, ArithLogicR<"addu", GPR32Opnd, 1, add>,
                 ADD_FM_MM<0, 0x150>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def SUBu_MM  : MMRel, ArithLogicR<"subu", GPR32Opnd, 0, sub>,
                 ADD_FM_MM<0, 0x1d0>, ISA_MICROMIPS32_NOT_MIPS32R6;
  let Defs = [HI0, LO0] in
    def MUL_MM   : MMRel, ArithLogicR<"mul", GPR32Opnd, 1, mul>,
                   ADD_FM_MM<0, 0x210>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def ADD_MM   : MMRel, ArithLogicR<"add", GPR32Opnd, 1>,
                 ADD_FM_MM<0, 0x110>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def SUB_MM   : MMRel, ArithLogicR<"sub", GPR32Opnd, 0>,
                 ADD_FM_MM<0, 0x190>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def SLT_MM   : MMRel, SetCC_R<"slt", setlt, GPR32Opnd>, ADD_FM_MM<0, 0x350>,
                 ISA_MICROMIPS;
  def SLTu_MM  : MMRel, SetCC_R<"sltu", setult, GPR32Opnd>,
                 ADD_FM_MM<0, 0x390>, ISA_MICROMIPS;
  def AND_MM   : MMRel, ArithLogicR<"and", GPR32Opnd, 1, and>,
                 ADD_FM_MM<0, 0x250>, ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `ADDu_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ADDu_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 743-756
```tablegen
  def OR_MM    : MMRel, ArithLogicR<"or", GPR32Opnd, 1, or>,
                 ADD_FM_MM<0, 0x290>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def XOR_MM   : MMRel, ArithLogicR<"xor", GPR32Opnd, 1, xor>,
                 ADD_FM_MM<0, 0x310>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def NOR_MM   : MMRel, LogicNOR<"nor", GPR32Opnd>, ADD_FM_MM<0, 0x2d0>,
                 ISA_MICROMIPS32_NOT_MIPS32R6;
  def MULT_MM  : MMRel, Mult<"mult", GPR32Opnd, [HI0, LO0]>,
                 MULT_FM_MM<0x22c>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def MULTu_MM : MMRel, Mult<"multu", GPR32Opnd, [HI0, LO0]>,
                 MULT_FM_MM<0x26c>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def SDIV_MM  : MMRel, Div<"div", GPR32Opnd, [HI0, LO0]>,
                 MULT_FM_MM<0x2ac>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def UDIV_MM  : MMRel, Div<"divu", GPR32Opnd, [HI0, LO0]>,
                 MULT_FM_MM<0x2ec>, ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `OR_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `OR_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 758-760
```tablegen
  /// Arithmetic Instructions with PC and Immediate
  def ADDIUPC_MM : AddImmUPC<"addiupc", GPRMM16Opnd>, ADDIUPC_FM_MM,
                   ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `ADDIUPC_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ADDIUPC_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 762-779
```tablegen
  /// Shift Instructions
  def SLL_MM   : MMRel, shift_rotate_imm<"sll", uimm5, GPR32Opnd>,
                 SRA_FM_MM<0, 0>, ISA_MICROMIPS;
  def SRL_MM   : MMRel, shift_rotate_imm<"srl", uimm5, GPR32Opnd>,
                 SRA_FM_MM<0x40, 0>, ISA_MICROMIPS;
  def SRA_MM   : MMRel, shift_rotate_imm<"sra", uimm5, GPR32Opnd>,
                 SRA_FM_MM<0x80, 0>, ISA_MICROMIPS;
  def SLLV_MM  : MMRel, shift_rotate_reg<"sllv", GPR32Opnd>,
                 SRLV_FM_MM<0x10, 0>, ISA_MICROMIPS;
  def SRLV_MM  : MMRel, shift_rotate_reg<"srlv", GPR32Opnd>,
                 SRLV_FM_MM<0x50, 0>, ISA_MICROMIPS;
  def SRAV_MM  : MMRel, shift_rotate_reg<"srav", GPR32Opnd>,
                 SRLV_FM_MM<0x90, 0>, ISA_MICROMIPS;
  def ROTR_MM  : MMRel, shift_rotate_imm<"rotr", uimm5, GPR32Opnd>,
                 SRA_FM_MM<0xc0, 0>, ISA_MICROMIPS {
    list<dag> Pattern = [(set GPR32Opnd:$rd,
                          (rotr GPR32Opnd:$rt, immZExt5:$shamt))];
  }
```
- EN: Defines TableGen record `SLL_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SLL_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 780-784
```tablegen
  def ROTRV_MM : MMRel, shift_rotate_reg<"rotrv", GPR32Opnd>,
                 SRLV_FM_MM<0xd0, 0>, ISA_MICROMIPS {
    list<dag> Pattern = [(set GPR32Opnd:$rd,
                          (rotr GPR32Opnd:$rt, GPR32Opnd:$rs))];
  }
```
- EN: Defines TableGen record `ROTRV_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ROTRV_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 786-803
```tablegen
  /// Load and Store Instructions - aligned
  let DecoderMethod = "DecodeMemMMImm16" in {
    def LB_MM  : LoadMemory<"lb", GPR32Opnd, mem_mm_16, sextloadi8>,
                 MMRel, LW_FM_MM<0x7>, ISA_MICROMIPS;
    def LBu_MM : LoadMemory<"lbu", GPR32Opnd, mem_mm_16, zextloadi8>,
                 MMRel, LW_FM_MM<0x5>, ISA_MICROMIPS;
    def LH_MM  : LoadMemory<"lh", GPR32Opnd, mem_simmptr, sextloadi16,
                            addrDefault>, MMRel, LW_FM_MM<0xf>, ISA_MICROMIPS;
    def LHu_MM : LoadMemory<"lhu", GPR32Opnd, mem_simmptr, zextloadi16>,
                 MMRel, LW_FM_MM<0xd>, ISA_MICROMIPS;
    def LW_MM  : Load<"lw", GPR32Opnd>, MMRel, LW_FM_MM<0x3f>,
                 ISA_MICROMIPS;
    def SB_MM  : Store<"sb", GPR32Opnd, truncstorei8>, MMRel,
                 LW_FM_MM<0x6>, ISA_MICROMIPS;
    def SH_MM  : Store<"sh", GPR32Opnd, truncstorei16>, MMRel,
                 LW_FM_MM<0xe>, ISA_MICROMIPS;
    def SW_MM  : Store<"sw", GPR32Opnd>, MMRel,
                 LW_FM_MM<0x3e>, ISA_MICROMIPS;
```
- EN: Defines TableGen record `LB_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LB_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 804-804
```tablegen
  }
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 806-823
```tablegen
  let DecoderMethod = "DecodeMemMMImm9" in {
    def LBE_MM  : MMRel, Load<"lbe", GPR32Opnd>,
                  POOL32C_LHUE_FM_MM<0x18, 0x6, 0x4>, ISA_MICROMIPS, ASE_EVA;
    def LBuE_MM : MMRel, Load<"lbue", GPR32Opnd>,
                  POOL32C_LHUE_FM_MM<0x18, 0x6, 0x0>, ISA_MICROMIPS, ASE_EVA;
    def LHE_MM  : MMRel, LoadMemory<"lhe", GPR32Opnd, mem_simm9>,
                  POOL32C_LHUE_FM_MM<0x18, 0x6, 0x5>, ISA_MICROMIPS, ASE_EVA;
    def LHuE_MM : MMRel, LoadMemory<"lhue", GPR32Opnd, mem_simm9>,
                  POOL32C_LHUE_FM_MM<0x18, 0x6, 0x1>, ISA_MICROMIPS, ASE_EVA;
    def LWE_MM  : MMRel, LoadMemory<"lwe", GPR32Opnd, mem_simm9>,
                  POOL32C_LHUE_FM_MM<0x18, 0x6, 0x7>, ISA_MICROMIPS, ASE_EVA;
    def SBE_MM  : MMRel, StoreMemory<"sbe", GPR32Opnd, mem_simm9>,
                  POOL32C_LHUE_FM_MM<0x18, 0xa, 0x4>, ISA_MICROMIPS, ASE_EVA;
    def SHE_MM  : MMRel, StoreMemory<"she", GPR32Opnd, mem_simm9>,
                  POOL32C_LHUE_FM_MM<0x18, 0xa, 0x5>, ISA_MICROMIPS, ASE_EVA;
    def SWE_MM  : MMRel, StoreMemory<"swe", GPR32Opnd, mem_simm9>,
                  POOL32C_LHUE_FM_MM<0x18, 0xa, 0x7>, ISA_MICROMIPS, ASE_EVA;
    def LWLE_MM : MMRel, LoadLeftRightMM<"lwle", MipsLWL, GPR32Opnd, mem_mm_9>,
```
- EN: Defines TableGen record `LBE_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LBE_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 824-835
```tablegen
                  POOL32C_STEVA_LDEVA_FM_MM<0x6, 0x2>,
                  ISA_MICROMIPS32_NOT_MIPS32R6, ASE_EVA;
    def LWRE_MM : MMRel, LoadLeftRightMM<"lwre", MipsLWR, GPR32Opnd, mem_mm_9>,
                  POOL32C_STEVA_LDEVA_FM_MM<0x6, 0x3>,
                  ISA_MICROMIPS32_NOT_MIPS32R6, ASE_EVA;
    def SWLE_MM : MMRel, StoreLeftRightMM<"swle", MipsSWL, GPR32Opnd, mem_mm_9>,
                  POOL32C_STEVA_LDEVA_FM_MM<0xa, 0x0>,
                  ISA_MICROMIPS32_NOT_MIPS32R6, ASE_EVA;
    def SWRE_MM : MMRel, StoreLeftRightMM<"swre", MipsSWR, GPR32Opnd, mem_mm_9>,
                  POOL32C_STEVA_LDEVA_FM_MM<0xa, 0x1>,
                  ISA_MICROMIPS32_NOT_MIPS32R6, ASE_EVA;
  }
```
- EN: Defines TableGen record `LWRE_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LWRE_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 837-838
```tablegen
  def LWXS_MM : LoadWordIndexedScaledMM<"lwxs", GPR32Opnd>, LWXS_FM_MM<0x118>,
                ISA_MICROMIPS;
```
- EN: Defines TableGen record `LWXS_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LWXS_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 840-848
```tablegen
  /// Load and Store Instructions - unaligned
  def LWL_MM : MMRel, LoadLeftRightMM<"lwl", MipsLWL, GPR32Opnd, mem_mm_12>,
               LWL_FM_MM<0x0>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def LWR_MM : MMRel, LoadLeftRightMM<"lwr", MipsLWR, GPR32Opnd, mem_mm_12>,
               LWL_FM_MM<0x1>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def SWL_MM : MMRel, StoreLeftRightMM<"swl", MipsSWL, GPR32Opnd, mem_mm_12>,
               LWL_FM_MM<0x8>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def SWR_MM : MMRel, StoreLeftRightMM<"swr", MipsSWR, GPR32Opnd, mem_mm_12>,
               LWL_FM_MM<0x9>, ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `LWL_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LWL_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 850-852
```tablegen
  /// Load and Store Instructions - multiple
  def SWM32_MM  : StoreMultMM<"swm32">, LWM_FM_MM<0xd>, ISA_MICROMIPS;
  def LWM32_MM  : LoadMultMM<"lwm32">, LWM_FM_MM<0x5>, ISA_MICROMIPS;
```
- EN: Defines TableGen record `SWM32_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SWM32_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 854-856
```tablegen
  /// Load and Store Pair Instructions
  def SWP_MM  : StorePairMM<"swp">, LWM_FM_MM<0x9>, ISA_MICROMIPS;
  def LWP_MM  : LoadPairMM<"lwp">, LWM_FM_MM<0x1>, ISA_MICROMIPS;
```
- EN: Defines TableGen record `SWP_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SWP_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 858-861
```tablegen
  /// Load and Store multiple pseudo Instructions
  class LoadWordMultMM<string instr_asm > :
    MipsAsmPseudoInst<(outs reglist:$rt), (ins mem_mm_12:$addr),
                      !strconcat(instr_asm, "\t$rt, $addr")> ;
```
- EN: Declares reusable TableGen class `LoadWordMultMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `LoadWordMultMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 863-865
```tablegen
  class StoreWordMultMM<string instr_asm > :
    MipsAsmPseudoInst<(outs), (ins reglist:$rt, mem_mm_12:$addr),
                      !strconcat(instr_asm, "\t$rt, $addr")> ;
```
- EN: Declares reusable TableGen class `StoreWordMultMM` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `StoreWordMultMM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 868-869
```tablegen
  def SWM_MM  : StoreWordMultMM<"swm">, ISA_MICROMIPS;
  def LWM_MM  : LoadWordMultMM<"lwm">, ISA_MICROMIPS;
```
- EN: Defines TableGen record `SWM_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SWM_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 871-888
```tablegen
  /// Move Conditional
  def MOVZ_I_MM : MMRel, CMov_I_I_FT<"movz", GPR32Opnd, GPR32Opnd>,
                  ADD_FM_MM<0, 0x58>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def MOVN_I_MM : MMRel, CMov_I_I_FT<"movn", GPR32Opnd, GPR32Opnd>,
                  ADD_FM_MM<0, 0x18>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def MOVT_I_MM : MMRel, CMov_F_I_FT<"movt", GPR32Opnd, MipsCMovFP_T>,
                  CMov_F_I_FM_MM<0x25>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def MOVF_I_MM : MMRel, CMov_F_I_FT<"movf", GPR32Opnd, MipsCMovFP_F>,
                  CMov_F_I_FM_MM<0x5>, ISA_MICROMIPS32_NOT_MIPS32R6;
  /// Move to/from HI/LO
  def MTHI_MM : MMRel, MoveToLOHI<"mthi", GPR32Opnd, [HI0]>,
                MTLO_FM_MM<0x0b5>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def MTLO_MM : MMRel, MoveToLOHI<"mtlo", GPR32Opnd, [LO0]>,
                MTLO_FM_MM<0x0f5>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def MFHI_MM : MMRel, MoveFromLOHI<"mfhi", GPR32Opnd, AC0>,
                MFLO_FM_MM<0x035>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def MFLO_MM : MMRel, MoveFromLOHI<"mflo", GPR32Opnd, AC0>,
                MFLO_FM_MM<0x075>, ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `MOVZ_I_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVZ_I_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 890-898
```tablegen
  /// Multiply Add/Sub Instructions
  def MADD_MM  : MMRel, MArithR<"madd", 1>, MULT_FM_MM<0x32c>,
                 ISA_MICROMIPS32_NOT_MIPS32R6;
  def MADDU_MM : MMRel, MArithR<"maddu", 1>, MULT_FM_MM<0x36c>,
                 ISA_MICROMIPS32_NOT_MIPS32R6;
  def MSUB_MM  : MMRel, MArithR<"msub">, MULT_FM_MM<0x3ac>,
                 ISA_MICROMIPS32_NOT_MIPS32R6;
  def MSUBU_MM : MMRel, MArithR<"msubu">, MULT_FM_MM<0x3ec>,
                 ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `MADD_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MADD_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 900-904
```tablegen
  /// Count Leading
  def CLZ_MM : MMRel, CountLeading0<"clz", GPR32Opnd>, CLO_FM_MM<0x16c>,
               ISA_MICROMIPS;
  def CLO_MM : MMRel, CountLeading1<"clo", GPR32Opnd>, CLO_FM_MM<0x12c>,
               ISA_MICROMIPS;
```
- EN: Defines TableGen record `CLZ_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CLZ_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 906-910
```tablegen
  /// Sign Ext In Register Instructions.
  def SEB_MM : MMRel, SignExtInReg<"seb", i8, GPR32Opnd>,
               SEB_FM_MM<0x0ac>, ISA_MICROMIPS;
  def SEH_MM : MMRel, SignExtInReg<"seh", i16, GPR32Opnd>,
               SEB_FM_MM<0x0ec>, ISA_MICROMIPS;
```
- EN: Defines TableGen record `SEB_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SEB_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 912-921
```tablegen
  /// Word Swap Bytes Within Halfwords
  def WSBH_MM : MMRel, SubwordSwap<"wsbh", GPR32Opnd>,
                SEB_FM_MM<0x1ec>, ISA_MICROMIPS;
  // TODO: Add '0 < pos+size <= 32' constraint check to ext instruction
  def EXT_MM : MMRel, ExtBase<"ext", GPR32Opnd, uimm5, uimm5_plus1, immZExt5,
                              immZExt5Plus1, MipsExt>, EXT_FM_MM<0x2c>,
               ISA_MICROMIPS32_NOT_MIPS32R6;
  def INS_MM : MMRel, InsBase<"ins", GPR32Opnd, uimm5, uimm5_inssize_plus1,
                              immZExt5, immZExt5Plus1>,
               EXT_FM_MM<0x0c>, ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `WSBH_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WSBH_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 923-930
```tablegen
  /// Jump Instructions
  let DecoderMethod = "DecodeJumpTargetMM" in {
    def J_MM          : MMRel, JumpFJ<jmptarget_mm, "j", br, bb, "j">,
                        J_FM_MM<0x35>, AdditionalRequires<[RelocNotPIC]>,
                        IsBranch, ISA_MICROMIPS32_NOT_MIPS32R6;
    def JAL_MM      : MMRel, JumpLink<"jal", calltarget_mm>, J_FM_MM<0x3d>,
                      ISA_MICROMIPS32_NOT_MIPS32R6;
  }
```
- EN: Defines TableGen record `J_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `J_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 932-934
```tablegen
  let DecoderMethod = "DecodeJumpTargetXMM" in
    def JALX_MM     : MMRel, JumpLink<"jalx", calltarget>, J_FM_MM<0x3c>,
                      ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `JALX_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JALX_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 936-939
```tablegen
  def JR_MM : MMRel, IndirectBranch<"jr", GPR32Opnd>, JR_FM_MM<0x3c>,
              ISA_MICROMIPS32_NOT_MIPS32R6;
  def JALR_MM : JumpLinkReg<"jalr", GPR32Opnd>, JALR_FM_MM<0x03c>,
                ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `JR_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JR_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 941-945
```tablegen
  /// Jump Instructions - Short Delay Slot
  def JALS_MM   : JumpLinkMM<"jals", calltarget_mm>, J_FM_MM<0x1d>,
                  ISA_MICROMIPS32_NOT_MIPS32R6;
  def JALRS_MM  : JumpLinkRegMM<"jalrs", GPR32Opnd>, JALR_FM_MM<0x13c>,
                  ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `JALS_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JALS_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 947-964
```tablegen
  /// Branch Instructions
  def BEQ_MM  : MMRel, CBranch<"beq", brtarget_mm, seteq, GPR32Opnd>,
                BEQ_FM_MM<0x25>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def BNE_MM  : MMRel, CBranch<"bne", brtarget_mm, setne, GPR32Opnd>,
                BEQ_FM_MM<0x2d>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def BGEZ_MM : MMRel, CBranchZero<"bgez", brtarget_mm, setge, GPR32Opnd>,
                BGEZ_FM_MM<0x2>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def BGTZ_MM : MMRel, CBranchZero<"bgtz", brtarget_mm, setgt, GPR32Opnd>,
                BGEZ_FM_MM<0x6>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def BLEZ_MM : MMRel, CBranchZero<"blez", brtarget_mm, setle, GPR32Opnd>,
                BGEZ_FM_MM<0x4>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def BLTZ_MM : MMRel, CBranchZero<"bltz", brtarget_mm, setlt, GPR32Opnd>,
                BGEZ_FM_MM<0x0>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def BGEZAL_MM : MMRel, BGEZAL_FT<"bgezal", brtarget_mm, GPR32Opnd>,
                  BGEZAL_FM_MM<0x03>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def BLTZAL_MM : MMRel, BGEZAL_FT<"bltzal", brtarget_mm, GPR32Opnd>,
                  BGEZAL_FM_MM<0x01>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def BAL_BR_MM : BAL_BR_Pseudo<BGEZAL_MM, brtarget_mm>,
```
- EN: Defines TableGen record `BEQ_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BEQ_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 965-965
```tablegen
                  ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 967-975
```tablegen
  /// Branch Instructions - Short Delay Slot
  def BGEZALS_MM : BranchCompareToZeroLinkMM<"bgezals", brtarget_mm,
                                             GPR32Opnd>, BGEZAL_FM_MM<0x13>,
                   ISA_MICROMIPS32_NOT_MIPS32R6;
  def BLTZALS_MM : BranchCompareToZeroLinkMM<"bltzals", brtarget_mm,
                                             GPR32Opnd>, BGEZAL_FM_MM<0x11>,
                   ISA_MICROMIPS32_NOT_MIPS32R6;
  def B_MM    : UncondBranch<BEQ_MM, brtarget_mm>, IsBranch,
                ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `BGEZALS_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BGEZALS_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 977-994
```tablegen
  /// Control Instructions
  def SYNC_MM    : MMRel, SYNC_FT<"sync">, SYNC_FM_MM, ISA_MICROMIPS;
  let DecoderMethod = "DecodeSyncI_MM" in
    def SYNCI_MM   : MMRel, SYNCI_FT<"synci", mem_mm_16>, SYNCI_FM_MM,
                     ISA_MICROMIPS32_NOT_MIPS32R6;
  def BREAK_MM   : MMRel, BRK_FT<"break">, BRK_FM_MM, ISA_MICROMIPS;
  def SYSCALL_MM : MMRel, SYS_FT<"syscall", uimm10>, SYS_FM_MM,
                   ISA_MICROMIPS;
  def WAIT_MM    : MMRel, WaitMM<"wait">, WAIT_FM_MM, ISA_MICROMIPS;
  def ERET_MM    : MMRel, ER_FT<"eret">, ER_FM_MM<0x3cd>,
                   ISA_MICROMIPS;
  def DERET_MM   : MMRel, ER_FT<"deret">, ER_FM_MM<0x38d>,
                   ISA_MICROMIPS;
  def EI_MM      : MMRel, DEI_FT<"ei", GPR32Opnd>, EI_FM_MM<0x15d>,
                   ISA_MICROMIPS;
  def DI_MM      : MMRel, DEI_FT<"di", GPR32Opnd>, EI_FM_MM<0x11d>,
                   ISA_MICROMIPS;
  def TRAP_MM    : TrapBase<BREAK_MM>, ISA_MICROMIPS;
```
- EN: Defines TableGen record `SYNC_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SYNC_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 996-1008
```tablegen
  /// Trap Instructions
  def TEQ_MM  : MMRel, TEQ_FT<"teq", GPR32Opnd, uimm4>, TEQ_FM_MM<0x0>,
                ISA_MICROMIPS;
  def TGE_MM  : MMRel, TEQ_FT<"tge", GPR32Opnd, uimm4>, TEQ_FM_MM<0x08>,
                ISA_MICROMIPS;
  def TGEU_MM : MMRel, TEQ_FT<"tgeu", GPR32Opnd, uimm4>,
                TEQ_FM_MM<0x10>, ISA_MICROMIPS;
  def TLT_MM  : MMRel, TEQ_FT<"tlt", GPR32Opnd, uimm4>, TEQ_FM_MM<0x20>,
                ISA_MICROMIPS;
  def TLTU_MM : MMRel, TEQ_FT<"tltu", GPR32Opnd, uimm4>,
                TEQ_FM_MM<0x28>, ISA_MICROMIPS;
  def TNE_MM  : MMRel, TEQ_FT<"tne", GPR32Opnd, uimm4>, TEQ_FM_MM<0x30>,
                ISA_MICROMIPS;
```
- EN: Defines TableGen record `TEQ_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TEQ_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1010-1021
```tablegen
  def TEQI_MM  : MMRel, TEQI_FT<"teqi", GPR32Opnd>, TEQI_FM_MM<0x0e>,
                 ISA_MICROMIPS32_NOT_MIPS32R6;
  def TGEI_MM  : MMRel, TEQI_FT<"tgei", GPR32Opnd>, TEQI_FM_MM<0x09>,
                 ISA_MICROMIPS32_NOT_MIPS32R6;
  def TGEIU_MM : MMRel, TEQI_FT<"tgeiu", GPR32Opnd>,
                 TEQI_FM_MM<0x0b>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def TLTI_MM  : MMRel, TEQI_FT<"tlti", GPR32Opnd>, TEQI_FM_MM<0x08>,
                 ISA_MICROMIPS32_NOT_MIPS32R6;
  def TLTIU_MM : MMRel, TEQI_FT<"tltiu", GPR32Opnd>,
                 TEQI_FM_MM<0x0a>, ISA_MICROMIPS32_NOT_MIPS32R6;
  def TNEI_MM  : MMRel, TEQI_FT<"tnei", GPR32Opnd>, TEQI_FM_MM<0x0c>,
                 ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `TEQI_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TEQI_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1023-1027
```tablegen
  /// Load-linked, Store-conditional
  def LL_MM : LLBaseMM<"ll", GPR32Opnd>, LL_FM_MM<0x3>,
              ISA_MICROMIPS32_NOT_MIPS32R6;
  def SC_MM : SCBaseMM<"sc", GPR32Opnd>, LL_FM_MM<0xb>,
              ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `LL_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LL_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1029-1032
```tablegen
  def LLE_MM : MMRel, LLEBaseMM<"lle", GPR32Opnd>, LLE_FM_MM<0x6>,
               ISA_MICROMIPS, ASE_EVA;
  def SCE_MM : MMRel, SCEBaseMM<"sce", GPR32Opnd>, LLE_FM_MM<0xA>,
               ISA_MICROMIPS, ASE_EVA;
```
- EN: Defines TableGen record `LLE_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LLE_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1034-1039
```tablegen
  let DecoderMethod = "DecodeCacheOpMM" in {
    def CACHE_MM : MMRel, CacheOp<"cache", mem_mm_12>,
                   CACHE_PREF_FM_MM<0x08, 0x6>, ISA_MICROMIPS32_NOT_MIPS32R6;
    def PREF_MM  : MMRel, CacheOp<"pref", mem_mm_12>,
                   CACHE_PREF_FM_MM<0x18, 0x2>, ISA_MICROMIPS32_NOT_MIPS32R6;
  }
```
- EN: Defines TableGen record `CACHE_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CACHE_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1041-1052
```tablegen
  let DecoderMethod = "DecodePrefeOpMM" in {
    def PREFE_MM  : MMRel, CacheOp<"prefe", mem_mm_9>,
                    CACHE_PREFE_FM_MM<0x18, 0x2>, ISA_MICROMIPS, ASE_EVA;
    def CACHEE_MM : MMRel, CacheOp<"cachee", mem_mm_9>,
                    CACHE_PREFE_FM_MM<0x18, 0x3>, ISA_MICROMIPS, ASE_EVA;
  }
  def SSNOP_MM : MMRel, Barrier<"ssnop">, BARRIER_FM_MM<0x1>,
                 ISA_MICROMIPS;
  def EHB_MM   : MMRel, Barrier<"ehb">, BARRIER_FM_MM<0x3>,
                 ISA_MICROMIPS;
  def PAUSE_MM : MMRel, Barrier<"pause">, BARRIER_FM_MM<0x5>,
                 ISA_MICROMIPS;
```
- EN: Defines TableGen record `PREFE_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PREFE_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1054-1061
```tablegen
  def TLBP_MM : MMRel, TLB<"tlbp">, COP0_TLB_FM_MM<0x0d>,
                ISA_MICROMIPS;
  def TLBR_MM : MMRel, TLB<"tlbr">, COP0_TLB_FM_MM<0x4d>,
                ISA_MICROMIPS;
  def TLBWI_MM : MMRel, TLB<"tlbwi">, COP0_TLB_FM_MM<0x8d>,
                 ISA_MICROMIPS;
  def TLBWR_MM : MMRel, TLB<"tlbwr">, COP0_TLB_FM_MM<0xcd>,
                 ISA_MICROMIPS;
```
- EN: Defines TableGen record `TLBP_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TLBP_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1063-1064
```tablegen
  def SDBBP_MM : MMRel, SYS_FT<"sdbbp", uimm10>, SDBBP_FM_MM,
                 ISA_MICROMIPS;
```
- EN: Defines TableGen record `SDBBP_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SDBBP_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1066-1068
```tablegen
  def PREFX_MM : PrefetchIndexed<"prefx">, POOL32F_PREFX_FM_MM<0x15, 0x1A0>,
                 ISA_MICROMIPS32_NOT_MIPS32R6;
}
```
- EN: Defines TableGen record `PREFX_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PREFX_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1070-1087
```tablegen
let AdditionalPredicates = [NotDSP] in {
  def PseudoMULT_MM : MultDivPseudo<MULT, ACC64, GPR32Opnd, MipsMult>,
                      ISA_MICROMIPS32_NOT_MIPS32R6;
  def PseudoMULTu_MM : MultDivPseudo<MULTu, ACC64, GPR32Opnd, MipsMultu>,
                       ISA_MICROMIPS32_NOT_MIPS32R6;
  def PseudoMFHI_MM : PseudoMFLOHI<GPR32, ACC64, MipsMFHI>,
                      ISA_MICROMIPS32_NOT_MIPS32R6;
  def PseudoMFLO_MM : PseudoMFLOHI<GPR32, ACC64, MipsMFLO>,
                      ISA_MICROMIPS32_NOT_MIPS32R6;
  def PseudoMTLOHI_MM : PseudoMTLOHI<ACC64, GPR32>,
                        ISA_MICROMIPS32_NOT_MIPS32R6;
  def PseudoMADD_MM : MAddSubPseudo<MADD, MipsMAdd>,
                      ISA_MICROMIPS32_NOT_MIPS32R6;
  def PseudoMADDU_MM : MAddSubPseudo<MADDU, MipsMAddu>,
                       ISA_MICROMIPS32_NOT_MIPS32R6;
  def PseudoMSUB_MM : MAddSubPseudo<MSUB, MipsMSub>,
                      ISA_MICROMIPS32_NOT_MIPS32R6;
  def PseudoMSUBU_MM : MAddSubPseudo<MSUBU, MipsMSubu>,
```
- EN: Defines TableGen record `PseudoMULT_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PseudoMULT_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1088-1089
```tablegen
                       ISA_MICROMIPS32_NOT_MIPS32R6;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1091-1092
```tablegen
def TAILCALL_MM : TailCall<J_MM, jmptarget_mm>,
                  ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `TAILCALL_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TAILCALL_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1094-1095
```tablegen
def TAILCALLREG_MM  : TailCallReg<JRC16_MM, GPR32Opnd>,
                      ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `TAILCALLREG_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TAILCALLREG_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1097-1098
```tablegen
def PseudoIndirectBranch_MM : PseudoIndirectBranchBase<JR_MM, GPR32Opnd>,
                              ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `PseudoIndirectBranch_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PseudoIndirectBranch_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1100-1104
```tablegen
let DecoderNamespace = "MicroMips" in {
  def RDHWR_MM : MMRel, R6MMR6Rel, ReadHardware<GPR32Opnd, HWRegsOpnd>,
                 RDHWR_FM_MM, ISA_MICROMIPS32_NOT_MIPS32R6;
  def LWU_MM : MMRel, LoadMM<"lwu", GPR32Opnd, zextloadi32, mem_simm12>,
               LL_FM_MM<0xe>, ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Defines TableGen record `RDHWR_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RDHWR_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1106-1123
```tablegen
  def MFGC0_MM    : MMRel, MfCop0MM<"mfgc0", GPR32Opnd, COP0Opnd>,
                    POOL32A_MFTC0_FM_MM<0b10011, 0b111100>,
                    ISA_MICROMIPS32R5, ASE_VIRT;
  def MFHGC0_MM   : MMRel, MfCop0MM<"mfhgc0", GPR32Opnd, COP0Opnd>,
                    POOL32A_MFTC0_FM_MM<0b10011, 0b110100>,
                    ISA_MICROMIPS32R5, ASE_VIRT;
  def MTGC0_MM    : MMRel, MtCop0MM<"mtgc0", COP0Opnd, GPR32Opnd>,
                    POOL32A_MFTC0_FM_MM<0b11011, 0b111100>,
                    ISA_MICROMIPS32R5, ASE_VIRT;
  def MTHGC0_MM   : MMRel, MtCop0MM<"mthgc0", COP0Opnd, GPR32Opnd>,
                    POOL32A_MFTC0_FM_MM<0b11011, 0b110100>,
                    ISA_MICROMIPS32R5, ASE_VIRT;
  def HYPCALL_MM  : MMRel, HypcallMM<"hypcall">, POOL32A_HYPCALL_FM_MM,
                    ISA_MICROMIPS32R5, ASE_VIRT;
  def TLBGINV_MM  : MMRel, TLBINVMM<"tlbginv">,
                    POOL32A_TLBINV_FM_MM<0x105>, ISA_MICROMIPS32R5, ASE_VIRT;
  def TLBGINVF_MM : MMRel, TLBINVMM<"tlbginvf">,
                    POOL32A_TLBINV_FM_MM<0x145>, ISA_MICROMIPS32R5, ASE_VIRT;
```
- EN: Defines TableGen record `MFGC0_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MFGC0_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1124-1132
```tablegen
  def TLBGP_MM    : MMRel, TLBINVMM<"tlbgp">,
                    POOL32A_TLBINV_FM_MM<0x5>, ISA_MICROMIPS32R5, ASE_VIRT;
  def TLBGR_MM    : MMRel, TLBINVMM<"tlbgr">,
                    POOL32A_TLBINV_FM_MM<0x45>, ISA_MICROMIPS32R5, ASE_VIRT;
  def TLBGWI_MM   : MMRel, TLBINVMM<"tlbgwi">,
                    POOL32A_TLBINV_FM_MM<0x85>, ISA_MICROMIPS32R5, ASE_VIRT;
  def TLBGWR_MM   : MMRel, TLBINVMM<"tlbgwr">,
                    POOL32A_TLBINV_FM_MM<0xc5>, ISA_MICROMIPS32R5, ASE_VIRT;
}
```
- EN: Defines TableGen record `TLBGP_MM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TLBGP_MM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1134-1136
```tablegen
//===----------------------------------------------------------------------===//
// MicroMips arbitrary patterns that map to one or more instructions
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 1138-1138
```tablegen
defm : MipsHiLoRelocs<LUi_MM, ADDiu_MM, ZERO, GPR32Opnd>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1140-1143
```tablegen
def : MipsPat<(MipsGotHi tglobaladdr:$in), (LUi_MM tglobaladdr:$in)>,
      ISA_MICROMIPS;
def : MipsPat<(MipsGotHi texternalsym:$in), (LUi_MM texternalsym:$in)>,
      ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1145-1146
```tablegen
def : MipsPat<(MipsTlsHi tglobaltlsaddr:$in), (LUi_MM tglobaltlsaddr:$in)>,
      ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1148-1152
```tablegen
// gp_rel relocs
def : MipsPat<(add GPR32:$gp, (MipsGPRel tglobaladdr:$in)),
              (ADDiu_MM GPR32:$gp, tglobaladdr:$in)>, ISA_MICROMIPS;
def : MipsPat<(add GPR32:$gp, (MipsGPRel tconstpool:$in)),
              (ADDiu_MM GPR32:$gp, tconstpool:$in)>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1154-1159
```tablegen
def : WrapperPat<tglobaladdr, ADDiu_MM, GPR32>, ISA_MICROMIPS;
def : WrapperPat<tconstpool, ADDiu_MM, GPR32>, ISA_MICROMIPS;
def : WrapperPat<texternalsym, ADDiu_MM, GPR32>, ISA_MICROMIPS;
def : WrapperPat<tblockaddress, ADDiu_MM, GPR32>, ISA_MICROMIPS;
def : WrapperPat<tjumptable, ADDiu_MM, GPR32>, ISA_MICROMIPS;
def : WrapperPat<tglobaltlsaddr, ADDiu_MM, GPR32>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1161-1163
```tablegen
def : MipsPat<(atomic_load_asext_8 addr:$a), (LB_MM addr:$a)>, ISA_MICROMIPS;
def : MipsPat<(atomic_load_asext_16 addr:$a), (LH_MM addr:$a)>, ISA_MICROMIPS;
def : MipsPat<(atomic_load_nonext_32 addr:$a), (LW_MM addr:$a)>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1165-1166
```tablegen
def : MipsPat<(i32 immLi16:$imm),
              (LI16_MM immLi16:$imm)>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1168-1168
```tablegen
defm : MaterializeImms<i32, ZERO, ADDiu_MM, LUi_MM, ORi_MM>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1170-1173
```tablegen
def : MipsPat<(not GPRMM16:$in),
              (NOT16_MM GPRMM16:$in)>, ISA_MICROMIPS;
def : MipsPat<(not GPR32:$in),
              (NOR_MM GPR32Opnd:$in, ZERO)>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1175-1180
```tablegen
def : MipsPat<(add GPRMM16:$src, immSExtAddiur2:$imm),
              (ADDIUR2_MM GPRMM16:$src, immSExtAddiur2:$imm)>, ISA_MICROMIPS;
def : MipsPat<(add GPR32:$src, immSExtAddius5:$imm),
              (ADDIUS5_MM GPR32:$src, immSExtAddius5:$imm)>, ISA_MICROMIPS;
def : MipsPat<(add GPR32:$src, immSExt16:$imm),
              (ADDiu_MM GPR32:$src, immSExt16:$imm)>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1182-1185
```tablegen
def : MipsPat<(and GPRMM16:$src, immZExtAndi16:$imm),
              (ANDI16_MM GPRMM16:$src, immZExtAndi16:$imm)>, ISA_MICROMIPS;
def : MipsPat<(and GPR32:$src, immZExt16:$imm),
              (ANDi_MM GPR32:$src, immZExt16:$imm)>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1187-1192
```tablegen
def : MipsPat<(shl GPRMM16:$src, immZExt2Shift:$imm),
              (SLL16_MM GPRMM16:$src, immZExt2Shift:$imm)>, ISA_MICROMIPS;
def : MipsPat<(shl GPR32:$src, immZExt5:$imm),
              (SLL_MM GPR32:$src, immZExt5:$imm)>, ISA_MICROMIPS;
def : MipsPat<(shl GPR32:$lhs, GPR32:$rhs),
              (SLLV_MM GPR32:$lhs, GPR32:$rhs)>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1194-1199
```tablegen
def : MipsPat<(srl GPRMM16:$src, immZExt2Shift:$imm),
              (SRL16_MM GPRMM16:$src, immZExt2Shift:$imm)>, ISA_MICROMIPS;
def : MipsPat<(srl GPR32:$src, immZExt5:$imm),
              (SRL_MM GPR32:$src, immZExt5:$imm)>, ISA_MICROMIPS;
def : MipsPat<(srl GPR32:$lhs, GPR32:$rhs),
              (SRLV_MM GPR32:$lhs, GPR32:$rhs)>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1201-1204
```tablegen
def : MipsPat<(sra GPR32:$src, immZExt5:$imm),
              (SRA_MM GPR32:$src, immZExt5:$imm)>, ISA_MICROMIPS;
def : MipsPat<(sra GPR32:$lhs, GPR32:$rhs),
              (SRAV_MM GPR32:$lhs, GPR32:$rhs)>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1206-1209
```tablegen
def : MipsPat<(store GPRMM16:$src, addrimm4lsl2:$addr),
              (SW16_MM GPRMM16:$src, addrimm4lsl2:$addr)>, ISA_MICROMIPS;
def : MipsPat<(store GPR32:$src, addr:$addr),
              (SW_MM GPR32:$src, addr:$addr)>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1211-1216
```tablegen
def : MipsPat<(load addrimm4lsl2:$addr),
              (LW16_MM addrimm4lsl2:$addr)>, ISA_MICROMIPS;
def : MipsPat<(load addr:$addr),
              (LW_MM addr:$addr)>, ISA_MICROMIPS;
def : MipsPat<(subc GPR32:$lhs, GPR32:$rhs),
              (SUBu_MM GPR32:$lhs, GPR32:$rhs)>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1218-1219
```tablegen
def : MipsPat<(i32 (extloadi1  addr:$src)), (LBu_MM addr:$src)>,
      ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1221-1222
```tablegen
def : MipsPat<(i32 (extloadi8  addr:$src)), (LBu_MM addr:$src)>,
      ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1224-1225
```tablegen
def : MipsPat<(i32 (extloadi16 addr:$src)), (LHu_MM addr:$src)>,
      ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1227-1229
```tablegen
let AddedComplexity = 40 in
  def : MipsPat<(i32 (sextloadi16 addrRegImm:$a)),
                (LH_MM addrRegImm:$a)>, ISA_MICROMIPS;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1232-1233
```tablegen
def : MipsPat<(bswap GPR32:$rt), (ROTR_MM (WSBH_MM GPR32:$rt), 16)>,
      ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1235-1240
```tablegen
def : MipsPat<(MipsJmpLink (i32 texternalsym:$dst)),
              (JAL_MM texternalsym:$dst)>, ISA_MICROMIPS32_NOT_MIPS32R6;
def : MipsPat<(MipsTailCall (iPTR tglobaladdr:$dst)),
              (TAILCALL_MM tglobaladdr:$dst)>, ISA_MICROMIPS32_NOT_MIPS32R6;
def : MipsPat<(MipsTailCall (iPTR texternalsym:$dst)),
              (TAILCALL_MM texternalsym:$dst)>, ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1242-1243
```tablegen
defm : BrcondPats<GPR32, BEQ_MM, BEQ_MM, BNE_MM, SLT_MM, SLTu_MM, SLTi_MM,
                  SLTiu_MM, ZERO>, ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1245-1248
```tablegen
def : MipsPat<(brcond (i32 (setlt i32:$lhs, 1)), bb:$dst),
              (BLEZ_MM i32:$lhs, bb:$dst)>, ISA_MICROMIPS32_NOT_MIPS32R6;
def : MipsPat<(brcond (i32 (setgt i32:$lhs, -1)), bb:$dst),
              (BGEZ_MM i32:$lhs, bb:$dst)>, ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1250-1254
```tablegen
defm : SeteqPats<GPR32, SLTiu_MM, XOR_MM, SLTu_MM, ZERO>, ISA_MICROMIPS;
defm : SetlePats<GPR32, XORi_MM, SLT_MM, SLTu_MM>, ISA_MICROMIPS;
defm : SetgtPats<GPR32, SLT_MM, SLTu_MM>, ISA_MICROMIPS;
defm : SetgePats<GPR32, XORi_MM, SLT_MM, SLTu_MM>, ISA_MICROMIPS;
defm : SetgeImmPats<GPR32, XORi_MM, SLTi_MM, SLTiu_MM>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1256-1256
```tablegen
// Select patterns
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1258-1264
```tablegen
// Instantiation of conditional move patterns.
defm : MovzPats0<GPR32, GPR32, MOVZ_I_MM, SLT_MM, SLTu_MM, SLTi_MM, SLTiu_MM>,
       ISA_MICROMIPS32_NOT_MIPS32R6;
defm : MovzPats1<GPR32, GPR32, MOVZ_I_MM, XOR_MM>,
       ISA_MICROMIPS32_NOT_MIPS32R6;
defm : MovzPats2<GPR32, GPR32, MOVZ_I_MM, XORi_MM>,
       ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1267-1267
```tablegen
defm : MovnPats<GPR32, GPR32, MOVN_I_MM, XOR_MM>, INSN_MIPS4_32_NOT_32R6_64R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1269-1275
```tablegen
// Instantiation of conditional move patterns.
defm : MovzPats0<GPR32, GPR32, MOVZ_I_MM, SLT_MM, SLTu_MM, SLTi_MM, SLTiu_MM>,
       ISA_MICROMIPS32_NOT_MIPS32R6;
defm : MovzPats1<GPR32, GPR32, MOVZ_I_MM, XOR_MM>,
       ISA_MICROMIPS32_NOT_MIPS32R6;
defm : MovzPats2<GPR32, GPR32, MOVZ_I_MM, XORi_MM>,
       ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1277-1277
```tablegen
defm : MovnPats<GPR32, GPR32, MOVN_I_MM, XOR_MM>, ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1279-1281
```tablegen
//===----------------------------------------------------------------------===//
// MicroMips instruction aliases
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 1283-1285
```tablegen
class UncondBranchMMPseudo<string opstr> :
  MipsAsmPseudoInst<(outs), (ins brtarget_mm:$offset),
                    !strconcat(opstr, "\t$offset")>;
```
- EN: Declares reusable TableGen class `UncondBranchMMPseudo` for `MicroMipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsInstrInfo` 声明可复用的 TableGen 类 `UncondBranchMMPseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1287-1287
```tablegen
def B_MM_Pseudo : UncondBranchMMPseudo<"b">, ISA_MICROMIPS;
```
- EN: Defines TableGen record `B_MM_Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `B_MM_Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1289-1293
```tablegen
let EncodingPredicates = [InMicroMips] in {
  def SDIV_MM_Pseudo : MultDivPseudo<SDIV_MM, ACC64, GPR32Opnd, MipsDivRem,
                                     0, 1, 1>, ISA_MIPS1_NOT_32R6_64R6;
  def UDIV_MM_Pseudo : MultDivPseudo<UDIV_MM, ACC64, GPR32Opnd, MipsDivRemU,
                                     0, 1, 1>, ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Defines TableGen record `SDIV_MM_Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SDIV_MM_Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1295-1312
```tablegen
  def : MipsInstAlias<"wait", (WAIT_MM 0x0), 1>, ISA_MICROMIPS;
  def : MipsInstAlias<"nop", (SLL_MM ZERO, ZERO, 0), 1>, ISA_MICROMIPS;
  def : MipsInstAlias<"nop", (MOVE16_MM ZERO, ZERO), 1>, ISA_MICROMIPS;
  def : MipsInstAlias<"ei", (EI_MM ZERO), 1>, ISA_MICROMIPS;
  def : MipsInstAlias<"di", (DI_MM ZERO), 1>, ISA_MICROMIPS;
  def : MipsInstAlias<"neg $rt, $rs",
                      (SUB_MM GPR32Opnd:$rt, ZERO, GPR32Opnd:$rs), 1>,
        ISA_MICROMIPS32_NOT_MIPS32R6;
  def : MipsInstAlias<"neg $rt",
                      (SUB_MM GPR32Opnd:$rt, ZERO, GPR32Opnd:$rt), 1>,
        ISA_MICROMIPS32_NOT_MIPS32R6;
  def : MipsInstAlias<"negu $rt, $rs",
                      (SUBu_MM GPR32Opnd:$rt, ZERO, GPR32Opnd:$rs), 1>,
        ISA_MICROMIPS32_NOT_MIPS32R6;
  def : MipsInstAlias<"negu $rt",
                      (SUBu_MM GPR32Opnd:$rt, ZERO, GPR32Opnd:$rt), 1>,
        ISA_MICROMIPS32_NOT_MIPS32R6;
  def : MipsInstAlias<"teq $rs, $rt",
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1313-1330
```tablegen
                      (TEQ_MM GPR32Opnd:$rs, GPR32Opnd:$rt, 0), 1>;
  def : MipsInstAlias<"tge $rs, $rt",
                      (TGE_MM GPR32Opnd:$rs, GPR32Opnd:$rt, 0), 1>;
  def : MipsInstAlias<"tgeu $rs, $rt",
                      (TGEU_MM GPR32Opnd:$rs, GPR32Opnd:$rt, 0), 1>;
  def : MipsInstAlias<"tlt $rs, $rt",
                      (TLT_MM GPR32Opnd:$rs, GPR32Opnd:$rt, 0), 1>;
  def : MipsInstAlias<"tltu $rs, $rt",
                      (TLTU_MM GPR32Opnd:$rs, GPR32Opnd:$rt, 0), 1>;
  def : MipsInstAlias<"tne $rs, $rt",
                      (TNE_MM GPR32Opnd:$rs, GPR32Opnd:$rt, 0), 1>;
  def : MipsInstAlias<
          "sgt $rd, $rs, $rt",
          (SLT_MM GPR32Opnd:$rd, GPR32Opnd:$rt, GPR32Opnd:$rs), 0>;
  def : MipsInstAlias<
          "sgt $rs, $rt",
          (SLT_MM GPR32Opnd:$rs, GPR32Opnd:$rt, GPR32Opnd:$rs), 0>;
  def : MipsInstAlias<
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1331-1348
```tablegen
          "sgtu $rd, $rs, $rt",
          (SLTu_MM GPR32Opnd:$rd, GPR32Opnd:$rt, GPR32Opnd:$rs), 0>;
  def : MipsInstAlias<
          "sgtu $rs, $rt",
          (SLTu_MM GPR32Opnd:$rs, GPR32Opnd:$rt, GPR32Opnd:$rs), 0>;
  def : MipsInstAlias<"sll $rd, $rt, $rs",
                      (SLLV_MM GPR32Opnd:$rd, GPR32Opnd:$rt, GPR32Opnd:$rs), 0>;
  def : MipsInstAlias<"sra $rd, $rt, $rs",
                      (SRAV_MM GPR32Opnd:$rd, GPR32Opnd:$rt, GPR32Opnd:$rs), 0>;
  def : MipsInstAlias<"srl $rd, $rt, $rs",
                      (SRLV_MM GPR32Opnd:$rd, GPR32Opnd:$rt, GPR32Opnd:$rs), 0>;
  def : MipsInstAlias<"sll $rd, $rt",
                      (SLLV_MM GPR32Opnd:$rd, GPR32Opnd:$rd, GPR32Opnd:$rt), 0>;
  def : MipsInstAlias<"sra $rd, $rt",
                      (SRAV_MM GPR32Opnd:$rd, GPR32Opnd:$rd, GPR32Opnd:$rt), 0>;
  def : MipsInstAlias<"srl $rd, $rt",
                      (SRLV_MM GPR32Opnd:$rd, GPR32Opnd:$rd, GPR32Opnd:$rt), 0>;
  def : MipsInstAlias<"sll $rd, $shamt",
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1349-1356
```tablegen
                      (SLL_MM GPR32Opnd:$rd, GPR32Opnd:$rd, uimm5:$shamt), 0>;
  def : MipsInstAlias<"sra $rd, $shamt",
                      (SRA_MM GPR32Opnd:$rd, GPR32Opnd:$rd, uimm5:$shamt), 0>;
  def : MipsInstAlias<"srl $rd, $shamt",
                      (SRL_MM GPR32Opnd:$rd, GPR32Opnd:$rd, uimm5:$shamt), 0>;
  def : MipsInstAlias<"rotr $rt, $imm",
                      (ROTR_MM GPR32Opnd:$rt, GPR32Opnd:$rt, uimm5:$imm), 0>;
  def : MipsInstAlias<"syscall", (SYSCALL_MM 0), 1>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1358-1358
```tablegen
  def : MipsInstAlias<"sync", (SYNC_MM 0), 1>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1360-1360
```tablegen
  defm : OneOrTwoOperandMacroImmediateAlias<"add", ADDi_MM>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1362-1362
```tablegen
  defm : OneOrTwoOperandMacroImmediateAlias<"addu", ADDiu_MM>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1364-1364
```tablegen
  defm : OneOrTwoOperandMacroImmediateAlias<"and", ANDi_MM>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1366-1366
```tablegen
  defm : OneOrTwoOperandMacroImmediateAlias<"or", ORi_MM>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1368-1368
```tablegen
  defm : OneOrTwoOperandMacroImmediateAlias<"xor", XORi_MM>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1370-1370
```tablegen
  defm : OneOrTwoOperandMacroImmediateAlias<"slt", SLTi_MM>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1372-1372
```tablegen
  defm : OneOrTwoOperandMacroImmediateAlias<"sltu", SLTiu_MM>, ISA_MICROMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1374-1391
```tablegen
  def : MipsInstAlias<"not $rt, $rs",
                      (NOR_MM GPR32Opnd:$rt, GPR32Opnd:$rs, ZERO), 0>,
        ISA_MICROMIPS32_NOT_MIPS32R6;
  def : MipsInstAlias<"not $rt",
                      (NOR_MM GPR32Opnd:$rt, GPR32Opnd:$rt, ZERO), 0>,
        ISA_MICROMIPS32_NOT_MIPS32R6;
  def : MipsInstAlias<"bnez $rs,$offset",
                      (BNE_MM GPR32Opnd:$rs, ZERO, brtarget:$offset), 0>,
        ISA_MICROMIPS;
  def : MipsInstAlias<"beqz $rs,$offset",
                      (BEQ_MM GPR32Opnd:$rs, ZERO, brtarget:$offset), 0>,
        ISA_MICROMIPS;
  def : MipsInstAlias<"seh $rd", (SEH_MM GPR32Opnd:$rd, GPR32Opnd:$rd), 0>,
                     ISA_MICROMIPS;
  def : MipsInstAlias<"seb $rd", (SEB_MM GPR32Opnd:$rd, GPR32Opnd:$rd), 0>,
                     ISA_MICROMIPS;
  def : MipsInstAlias<"break", (BREAK_MM 0, 0), 1>, ISA_MICROMIPS;
  def : MipsInstAlias<"break $imm", (BREAK_MM uimm10:$imm, 0), 1>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1392-1394
```tablegen
        ISA_MICROMIPS;
  def : MipsInstAlias<"bal $offset", (BGEZAL_MM ZERO, brtarget_mm:$offset), 1>,
        ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1396-1401
```tablegen
  def : MipsInstAlias<"j $rs", (JR_MM GPR32Opnd:$rs), 0>,
        ISA_MICROMIPS32_NOT_MIPS32R6;
}
def : MipsInstAlias<"rdhwr $rt, $rs",
                    (RDHWR_MM GPR32Opnd:$rt, HWRegsOpnd:$rs, 0), 1>,
      ISA_MICROMIPS32_NOT_MIPS32R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1403-1419
```tablegen
def : MipsInstAlias<"hypcall", (HYPCALL_MM 0), 1>,
                    ISA_MICROMIPS32R5, ASE_VIRT;
def : MipsInstAlias<"mfgc0 $rt, $rs",
                    (MFGC0_MM GPR32Opnd:$rt, COP0Opnd:$rs, 0), 0>,
                    ISA_MICROMIPS32R5, ASE_VIRT;
def : MipsInstAlias<"mfhgc0 $rt, $rs",
                    (MFHGC0_MM GPR32Opnd:$rt, COP0Opnd:$rs, 0), 0>,
                    ISA_MICROMIPS32R5, ASE_VIRT;
def : MipsInstAlias<"mtgc0 $rt, $rs",
                    (MTGC0_MM COP0Opnd:$rs, GPR32Opnd:$rt, 0), 0>,
                    ISA_MICROMIPS32R5, ASE_VIRT;
def : MipsInstAlias<"mthgc0 $rt, $rs",
                    (MTHGC0_MM COP0Opnd:$rs, GPR32Opnd:$rt, 0), 0>,
                    ISA_MICROMIPS32R5, ASE_VIRT;
def : MipsInstAlias<"sw $rt, $offset",
                    (SWSP_MM GPR32Opnd:$rt, mem_mm_sp_imm5_lsl2:$offset), 1>,
                    ISA_MICROMIPS;
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
