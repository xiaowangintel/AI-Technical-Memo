# Mips16InstrInfo.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/Mips16InstrInfo.td`
- Repository: `llvm-project`
- Purpose (EN): This file describes Mips16 instructions.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `Mips16InstrInfo`，涵盖指令语义、调度提示以及机器级辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
```tablegen
//===- Mips16InstrInfo.td - Target Description for Mips16  -*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes Mips16 instructions.
//
//===----------------------------------------------------------------------===//
//
//
// Mips Address
//
def addr16 : ComplexPattern<iPTR, 2, "selectAddr16", [frameindex]>;
def addr16sp : ComplexPattern<iPTR, 2, "selectAddr16SP", [frameindex]>;
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 19-25
```tablegen
//
// Address operand
def mem16 : Operand<i32> {
  let PrintMethod = "printMemOperand";
  let MIOperandInfo = (ops CPU16Regs, simm16);
  let EncoderMethod = "getMemEncoding";
}
```
- EN: Defines TableGen record `mem16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mem16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 27-37
```tablegen
def mem16sp : Operand<i32> {
  let PrintMethod = "printMemOperand";
  // This should be CPUSPReg but the MIPS16 subtarget isn't good enough at
  // keeping the sp-relative load and the other varieties separate at the
  // moment. This lie fixes the problem sufficiently well to fix the errors
  // emitted by -verify-machineinstrs and the output ends up correct as long
  // as we use an external assembler (which is already a requirement for MIPS16
  // for several other reasons).
  let MIOperandInfo = (ops CPU16RegsPlusSP, simm16);
  let EncoderMethod = "getMemEncoding";
}
```
- EN: Defines TableGen record `mem16sp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mem16sp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 39-43
```tablegen
def mem16_ea : Operand<i32> {
  let PrintMethod = "printMemOperandEA";
  let MIOperandInfo = (ops CPU16RegsPlusSP, simm16);
  let EncoderMethod = "getMemEncoding";
}
```
- EN: Defines TableGen record `mem16_ea` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mem16_ea`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 45-45
```tablegen
def pcrel16 : Operand<i32>;
```
- EN: Defines TableGen record `pcrel16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `pcrel16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 47-55
```tablegen
//
// I-type instruction format
//
// this is only used by bimm. the actual assembly value is a 12 bit signed
// number
//
class FI16_ins<bits<5> op, string asmstr>:
  FI16<op, (outs), (ins brtarget:$imm11),
            !strconcat(asmstr, "\t$imm11 # 16 bit inst"), []>;
```
- EN: Declares reusable TableGen class `FI16_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FI16_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 57-60
```tablegen
//
//
// I8 instruction format
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 62-65
```tablegen
class FI816_ins_base<bits<3> _func, string asmstr,
                     string asmstr2>:
  FI816<_func, (outs), (ins simm16:$imm8), !strconcat(asmstr, asmstr2),
        []>;
```
- EN: Declares reusable TableGen class `FI816_ins_base` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FI816_ins_base`，通常用于抽象共享字段、谓词或编码结构。

### Lines 67-68
```tablegen
class FI816_ins<bits<3> _func, string asmstr>:
  FI816_ins_base<_func, asmstr, "\t$imm8  # 16 bit inst">;
```
- EN: Declares reusable TableGen class `FI816_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FI816_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 70-71
```tablegen
class FI816_SP_ins<bits<3> _func, string asmstr>:
  FI816_ins_base<_func, asmstr, "\t$$sp, $imm8 # 16 bit inst">;
```
- EN: Declares reusable TableGen class `FI816_SP_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FI816_SP_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 73-75
```tablegen
//
// RI instruction format
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 78-80
```tablegen
class FRI16_ins_base<bits<5> op, string asmstr, string asmstr2>:
  FRI16<op, (outs CPU16Regs:$rx), (ins simm16:$imm8),
        !strconcat(asmstr, asmstr2), []>;
```
- EN: Declares reusable TableGen class `FRI16_ins_base` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FRI16_ins_base`，通常用于抽象共享字段、谓词或编码结构。

### Lines 82-83
```tablegen
class FRI16_ins<bits<5> op, string asmstr>:
  FRI16_ins_base<op, asmstr, "\t$rx, $imm8 \t# 16 bit inst">;
```
- EN: Declares reusable TableGen class `FRI16_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FRI16_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 85-87
```tablegen
class FRI16_TCP_ins<bits<5> _op, string asmstr>:
  FRI16<_op, (outs CPU16Regs:$rx), (ins pcrel16:$imm8),
            !strconcat(asmstr, "\t$rx, $imm8\t# 16 bit inst"), []>;
```
- EN: Declares reusable TableGen class `FRI16_TCP_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FRI16_TCP_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 89-91
```tablegen
class FRI16R_ins_base<bits<5> op, string asmstr, string asmstr2>:
  FRI16<op, (outs), (ins CPU16Regs:$rx, simm16:$imm8),
        !strconcat(asmstr, asmstr2), []>;
```
- EN: Declares reusable TableGen class `FRI16R_ins_base` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FRI16R_ins_base`，通常用于抽象共享字段、谓词或编码结构。

### Lines 93-94
```tablegen
class FRI16R_ins<bits<5> op, string asmstr>:
  FRI16R_ins_base<op, asmstr, "\t$rx, $imm8 \t# 16 bit inst">;
```
- EN: Declares reusable TableGen class `FRI16R_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FRI16R_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 96-100
```tablegen
class F2RI16_ins<bits<5> _op, string asmstr>:
  FRI16<_op, (outs CPU16Regs:$rx), (ins CPU16Regs:$rx_, simm16:$imm8),
        !strconcat(asmstr, "\t$rx, $imm8\t# 16 bit inst"), []> {
  let Constraints = "$rx_ = $rx";
}
```
- EN: Declares reusable TableGen class `F2RI16_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `F2RI16_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 102-116
```tablegen
class FRI16_B_ins<bits<5> _op, string asmstr>:
  FRI16<_op, (outs), (ins  CPU16Regs:$rx, brtarget:$imm8),
        !strconcat(asmstr, "\t$rx, $imm8  # 16 bit inst"), []>;
//
// Compare a register and immediate and place result in CC
// Implicit use of T8
//
// EXT-CCRR Instruction format
//
class FEXT_CCRXI16_ins<string asmstr>:
  MipsPseudo16<(outs CPU16Regs:$cc), (ins CPU16Regs:$rx, simm16:$imm),
               !strconcat(asmstr, "\t$rx, $imm\n\tmove\t$cc, $$t8"), []> {
  let isCodeGenOnly=1;
  let usesCustomInserter = 1;
}
```
- EN: Declares reusable TableGen class `FRI16_B_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FRI16_B_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 118-125
```tablegen
// JAL and JALX instruction format
//
class FJAL16_ins<bits<1> _X, string asmstr>:
  FJAL16<_X, (outs), (ins uimm26:$imm26),
         !strconcat(asmstr, "\t$imm26\n\tnop"),[]>  {
  let isCodeGenOnly=1;
  let Size=6;
}
```
- EN: Declares reusable TableGen class `FJAL16_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FJAL16_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 127-132
```tablegen
class FJALB16_ins<bits<1> _X, string asmstr>:
  FJAL16<_X, (outs), (ins uimm26:$imm26),
         !strconcat(asmstr, "\t$imm26\t# branch\n\tnop"),[]>  {
  let isCodeGenOnly=1;
  let Size=6;
}
```
- EN: Declares reusable TableGen class `FJALB16_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FJALB16_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 134-139
```tablegen
//
// EXT-I instruction format
//
class FEXT_I16_ins<bits<5> eop, string asmstr> :
  FEXT_I16<eop, (outs), (ins brtarget:$imm16),
           !strconcat(asmstr, "\t$imm16"),[]>;
```
- EN: Declares reusable TableGen class `FEXT_I16_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FEXT_I16_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 141-143
```tablegen
//
// EXT-I8 instruction format
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 145-148
```tablegen
class FEXT_I816_ins_base<bits<3> _func, string asmstr,
                         string asmstr2>:
  FEXT_I816<_func, (outs), (ins simm16:$imm16), !strconcat(asmstr, asmstr2),
            []>;
```
- EN: Declares reusable TableGen class `FEXT_I816_ins_base` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FEXT_I816_ins_base`，通常用于抽象共享字段、谓词或编码结构。

### Lines 150-151
```tablegen
class FEXT_I816_ins<bits<3> _func, string asmstr>:
  FEXT_I816_ins_base<_func, asmstr, "\t$imm16">;
```
- EN: Declares reusable TableGen class `FEXT_I816_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FEXT_I816_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 153-154
```tablegen
class FEXT_I816_SP_ins<bits<3> _func, string asmstr>:
      FEXT_I816_ins_base<_func, asmstr, "\t$$sp, $imm16">;
```
- EN: Declares reusable TableGen class `FEXT_I816_SP_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FEXT_I816_SP_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 156-170
```tablegen
//
// Assembler formats in alphabetical order.
// Natural and pseudos are mixed together.
//
// Compare two registers and place result in CC
// Implicit use of T8
//
// CC-RR Instruction format
//
class FCCRR16_ins<string asmstr> :
  MipsPseudo16<(outs CPU16Regs:$cc), (ins CPU16Regs:$rx, CPU16Regs:$ry),
               !strconcat(asmstr, "\t$rx, $ry\n\tmove\t$cc, $$t8"), []> {
  let isCodeGenOnly=1;
  let usesCustomInserter = 1;
}
```
- EN: Declares reusable TableGen class `FCCRR16_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FCCRR16_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 172-174
```tablegen
//
// EXT-RI instruction format
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 176-178
```tablegen
class FEXT_RI16_ins_base<bits<5> _op, string asmstr, string asmstr2>:
  FEXT_RI16<_op, (outs CPU16Regs:$rx), (ins simm16:$imm16),
                  !strconcat(asmstr, asmstr2), []>;
```
- EN: Declares reusable TableGen class `FEXT_RI16_ins_base` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FEXT_RI16_ins_base`，通常用于抽象共享字段、谓词或编码结构。

### Lines 180-181
```tablegen
class FEXT_RI16_ins<bits<5> _op, string asmstr>:
  FEXT_RI16_ins_base<_op, asmstr, "\t$rx, $imm16">;
```
- EN: Declares reusable TableGen class `FEXT_RI16_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FEXT_RI16_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 183-185
```tablegen
class FEXT_RI16R_ins_base<bits<5> _op, string asmstr, string asmstr2>:
  FEXT_RI16<_op, (outs ), (ins CPU16Regs:$rx, simm16:$imm16),
                  !strconcat(asmstr, asmstr2), []>;
```
- EN: Declares reusable TableGen class `FEXT_RI16R_ins_base` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FEXT_RI16R_ins_base`，通常用于抽象共享字段、谓词或编码结构。

### Lines 187-188
```tablegen
class FEXT_RI16R_ins<bits<5> _op, string asmstr>:
  FEXT_RI16R_ins_base<_op, asmstr, "\t$rx, $imm16">;
```
- EN: Declares reusable TableGen class `FEXT_RI16R_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FEXT_RI16R_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 190-191
```tablegen
class FEXT_RI16_PC_ins<bits<5> _op, string asmstr>:
  FEXT_RI16_ins_base<_op, asmstr, "\t$rx, $$pc, $imm16">;
```
- EN: Declares reusable TableGen class `FEXT_RI16_PC_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FEXT_RI16_PC_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 193-195
```tablegen
class FEXT_RI16_B_ins<bits<5> _op, string asmstr>:
  FEXT_RI16<_op, (outs), (ins  CPU16Regs:$rx, brtarget:$imm16),
            !strconcat(asmstr, "\t$rx, $imm16"), []>;
```
- EN: Declares reusable TableGen class `FEXT_RI16_B_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FEXT_RI16_B_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 197-199
```tablegen
class FEXT_RI16_TCP_ins<bits<5> _op, string asmstr>:
  FEXT_RI16<_op, (outs CPU16Regs:$rx), (ins pcrel16:$imm16),
            !strconcat(asmstr, "\t$rx, $imm16"), []>;
```
- EN: Declares reusable TableGen class `FEXT_RI16_TCP_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FEXT_RI16_TCP_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 201-205
```tablegen
class FEXT_2RI16_ins<bits<5> _op, string asmstr>:
  FEXT_RI16<_op, (outs CPU16Regs:$rx), (ins CPU16Regs:$rx_, simm16:$imm16),
            !strconcat(asmstr, "\t$rx, $imm16"), []> {
  let Constraints = "$rx_ = $rx";
}
```
- EN: Declares reusable TableGen class `FEXT_2RI16_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FEXT_2RI16_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 207-209
```tablegen
//
// EXT-RRI instruction format
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 211-213
```tablegen
class FEXT_RRI16_mem_ins<bits<5> op, string asmstr, Operand MemOpnd>:
  FEXT_RRI16<op, (outs CPU16Regs:$ry), (ins  (MemOpnd $rx, $imm16):$addr),
             !strconcat(asmstr, "\t$ry, $addr"), []>;
```
- EN: Declares reusable TableGen class `FEXT_RRI16_mem_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FEXT_RRI16_mem_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 215-217
```tablegen
class FEXT_RRI16_mem2_ins<bits<5> op, string asmstr, Operand MemOpnd>:
  FEXT_RRI16<op, (outs ), (ins  CPU16Regs:$ry, (MemOpnd $rx, $imm16):$addr),
             !strconcat(asmstr, "\t$ry, $addr"), []>;
```
- EN: Declares reusable TableGen class `FEXT_RRI16_mem2_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FEXT_RRI16_mem2_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 219-222
```tablegen
//
//
// EXT-RRI-A instruction format
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 224-226
```tablegen
class FEXT_RRI_A16_mem_ins<bits<1> op, string asmstr, Operand MemOpnd>:
  FEXT_RRI_A16<op, (outs CPU16Regs:$ry), (ins  (MemOpnd $rx, $imm15):$addr),
               !strconcat(asmstr, "\t$ry, $addr"), []>;
```
- EN: Declares reusable TableGen class `FEXT_RRI_A16_mem_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FEXT_RRI_A16_mem_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 228-233
```tablegen
//
// EXT-SHIFT instruction format
//
class FEXT_SHIFT16_ins<bits<2> _f, string asmstr>:
  FEXT_SHIFT16<_f, (outs CPU16Regs:$rx), (ins CPU16Regs:$ry, uimm5:$sa6),
               !strconcat(asmstr, "\t$rx, $ry, $sa6"), []>;
```
- EN: Declares reusable TableGen class `FEXT_SHIFT16_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FEXT_SHIFT16_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 235-245
```tablegen
//
// EXT-T8I8
//
class FEXT_T8I816_ins<string asmstr, string asmstr2>:
  MipsPseudo16<(outs),
               (ins CPU16Regs:$rx, CPU16Regs:$ry, brtarget:$imm),
               !strconcat(asmstr2, !strconcat("\t$rx, $ry\n\t",
               !strconcat(asmstr, "\t$imm"))),[]> {
  let isCodeGenOnly=1;
  let usesCustomInserter = 1;
}
```
- EN: Declares reusable TableGen class `FEXT_T8I816_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FEXT_T8I816_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 247-258
```tablegen
//
// EXT-T8I8I
//
class FEXT_T8I8I16_ins<string asmstr, string asmstr2>:
  MipsPseudo16<(outs),
               (ins CPU16Regs:$rx, simm16:$imm, brtarget:$targ),
               !strconcat(asmstr2, !strconcat("\t$rx, $imm\n\t",
               !strconcat(asmstr, "\t$targ"))), []> {
  let isCodeGenOnly=1;
  let usesCustomInserter = 1;
}
//
```
- EN: Declares reusable TableGen class `FEXT_T8I8I16_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FEXT_T8I8I16_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 261-266
```tablegen
//
// I8_MOVR32 instruction format (used only by the MOVR32 instructio
//
class FI8_MOVR3216_ins<string asmstr>:
       FI8_MOVR3216<(outs CPU16Regs:$ry), (ins GPR32:$r32),
       !strconcat(asmstr,  "\t$ry, $r32"), []>;
```
- EN: Declares reusable TableGen class `FI8_MOVR3216_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FI8_MOVR3216_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 268-270
```tablegen
//
// I8_MOV32R instruction format (used only by MOV32R instruction)
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 272-274
```tablegen
class FI8_MOV32R16_ins<string asmstr>:
  FI8_MOV32R16<(outs GPR32:$r32), (ins CPU16Regs:$rz),
               !strconcat(asmstr,  "\t$r32, $rz"), []>;
```
- EN: Declares reusable TableGen class `FI8_MOV32R16_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FI8_MOV32R16_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 276-284
```tablegen
//
// This are pseudo formats for multiply
// This first one can be changed to non-pseudo now.
//
// MULT
//
class FMULT16_ins<string asmstr> :
  MipsPseudo16<(outs), (ins CPU16Regs:$rx, CPU16Regs:$ry),
               !strconcat(asmstr, "\t$rx, $ry"), []>;
```
- EN: Declares reusable TableGen class `FMULT16_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FMULT16_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 286-293
```tablegen
//
// MULT-LO
//
class FMULT16_LO_ins<string asmstr> :
  MipsPseudo16<(outs CPU16Regs:$rz), (ins CPU16Regs:$rx, CPU16Regs:$ry),
               !strconcat(asmstr, "\t$rx, $ry\n\tmflo\t$rz"), []> {
  let isCodeGenOnly=1;
}
```
- EN: Declares reusable TableGen class `FMULT16_LO_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FMULT16_LO_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 295-297
```tablegen
//
// RR-type instruction format
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 299-302
```tablegen
class FRR16_ins<bits<5> f, string asmstr> :
  FRR16<f, (outs CPU16Regs:$rx), (ins CPU16Regs:$ry),
        !strconcat(asmstr, "\t$rx, $ry"), []> {
}
```
- EN: Declares reusable TableGen class `FRR16_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FRR16_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 304-307
```tablegen
class FRRBreakNull16_ins<string asmstr> :
  FRRBreak16<(outs), (ins), asmstr, []> {
  let Code=0;
}
```
- EN: Declares reusable TableGen class `FRRBreakNull16_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FRRBreakNull16_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 309-312
```tablegen
class FRR16R_ins<bits<5> f, string asmstr> :
  FRR16<f, (outs), (ins  CPU16Regs:$rx, CPU16Regs:$ry),
        !strconcat(asmstr, "\t$rx, $ry"), []> {
}
```
- EN: Declares reusable TableGen class `FRR16R_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FRR16R_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 314-316
```tablegen
class FRRTR16_ins<string asmstr> :
  MipsPseudo16<(outs CPU16Regs:$rz), (ins CPU16Regs:$rx, CPU16Regs:$ry),
               !strconcat(asmstr, "\t$rx, $ry\n\tmove\t$rz, $$t8"), []> ;
```
- EN: Declares reusable TableGen class `FRRTR16_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FRRTR16_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 318-323
```tablegen
//
// maybe refactor but need a $zero as a dummy first parameter
//
class FRR16_div_ins<bits<5> f, string asmstr> :
  FRR16<f, (outs ), (ins CPU16Regs:$rx, CPU16Regs:$ry),
        !strconcat(asmstr, "\t$$zero, $rx, $ry"), []> ;
```
- EN: Declares reusable TableGen class `FRR16_div_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FRR16_div_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 325-327
```tablegen
class FUnaryRR16_ins<bits<5> f, string asmstr> :
  FRR16<f, (outs CPU16Regs:$rx), (ins CPU16Regs:$ry),
        !strconcat(asmstr, "\t$rx, $ry"), []> ;
```
- EN: Declares reusable TableGen class `FUnaryRR16_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FUnaryRR16_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 330-332
```tablegen
class FRR16_M_ins<bits<5> f, string asmstr> :
  FRR16<f, (outs CPU16Regs:$rx), (ins),
        !strconcat(asmstr, "\t$rx"), []>;
```
- EN: Declares reusable TableGen class `FRR16_M_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FRR16_M_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 334-339
```tablegen
class FRxRxRy16_ins<bits<5> f, string asmstr> :
  FRR16<f, (outs CPU16Regs:$rz), (ins CPU16Regs:$rx, CPU16Regs:$ry),
            !strconcat(asmstr, "\t$rz, $ry"),
            []> {
  let Constraints = "$rx = $rz";
}
```
- EN: Declares reusable TableGen class `FRxRxRy16_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FRxRxRy16_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 341-345
```tablegen
let rx=0 in
class FRR16_JALRC_RA_only_ins<bits<1> nd_, bits<1> l_,
                              string asmstr>:
  FRR16_JALRC<nd_, l_, 1, (outs), (ins), !strconcat(asmstr, "\t$$ra"),
              []> ;
```
- EN: Declares reusable TableGen class `FRR16_JALRC_RA_only_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FRR16_JALRC_RA_only_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 348-351
```tablegen
class FRR16_JALRC_ins<bits<1> nd, bits<1> l, bits<1> ra,
                      string asmstr>:
  FRR16_JALRC<nd, l, ra, (outs), (ins CPU16Regs:$rx),
              !strconcat(asmstr, "\t$rx"), []> ;
```
- EN: Declares reusable TableGen class `FRR16_JALRC_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FRR16_JALRC_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 353-363
```tablegen
class FRR_SF16_ins
  <bits<5> _funct, bits<3> _subfunc,
    string asmstr>:
  FRR_SF16<_funct, _subfunc, (outs CPU16Regs:$rx), (ins CPU16Regs:$rx_),
           !strconcat(asmstr, "\t $rx"),
           []> {
  let Constraints = "$rx_ = $rx";
  }
//
// RRR-type instruction format
//
```
- EN: Declares reusable TableGen class `FRR_SF16_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FRR_SF16_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 365-367
```tablegen
class FRRR16_ins<bits<2> _f, string asmstr> :
  FRRR16<_f, (outs CPU16Regs:$rz), (ins CPU16Regs:$rx, CPU16Regs:$ry),
         !strconcat(asmstr, "\t$rz, $rx, $ry"), []>;
```
- EN: Declares reusable TableGen class `FRRR16_ins` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `FRRR16_ins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 369-386
```tablegen
//
// These Sel patterns support the generation of conditional move
// pseudo instructions.
//
// The nomenclature uses the components making up the pseudo and may
// be a bit counter intuitive when compared with the end result we seek.
// For example using a bqez in the example directly below results in the
// conditional move being done if the tested register is not zero.
// I considered in easier to check by keeping the pseudo consistent with
// it's components but it could have been done differently.
//
// The simplest case is when can test and operand directly and do the
// conditional move based on a simple mips16 conditional
//  branch instruction.
// for example:
// if $op == beqz or bnez:
//
// $op1 $rt, .+4
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 387-390
```tablegen
// move $rd, $rs
//
// if $op == beqz, then if $rt != 0, then the conditional assignment
// $rd = $rs is done.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 392-404
```tablegen
// if $op == bnez, then if $rt == 0, then the conditional assignment
// $rd = $rs is done.
//
// So this pseudo class only has one operand, i.e. op
//
class Sel<string op>:
  MipsPseudo16<(outs CPU16Regs:$rd_), (ins CPU16Regs:$rd, CPU16Regs:$rs,
               CPU16Regs:$rt),
               !strconcat(op, "\t$rt, .+4\n\t\n\tmove $rd, $rs"), []> {
  //let isCodeGenOnly=1;
  let Constraints = "$rd = $rd_";
  let usesCustomInserter = 1;
}
```
- EN: Declares reusable TableGen class `only` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `only`，通常用于抽象共享字段、谓词或编码结构。

### Lines 406-410
```tablegen
//
// The next two instruction classes allow for an operand which tests
// two operands and returns a value in register T8 and
//then does a conditional branch based on the value of T8
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 412-429
```tablegen
// op2 can be cmpi or slti/sltiu
// op1 can bteqz or btnez
// the operands for op2 are a register and a signed constant
//
// $op2 $t, $imm  ;test register t and branch conditionally
// $op1 .+4       ;op1 is a conditional branch
// move $rd, $rs
//
//
class SeliT<string op1, string op2>:
  MipsPseudo16<(outs CPU16Regs:$rd_), (ins CPU16Regs:$rd, CPU16Regs:$rs,
                                       CPU16Regs:$rl, simm16:$imm),
               !strconcat(op2,
               !strconcat("\t$rl, $imm\n\t",
               !strconcat(op1, "\t.+4\n\tmove $rd, $rs"))), []> {
  let isCodeGenOnly=1;
  let Constraints = "$rd = $rd_";
  let usesCustomInserter = 1;
```
- EN: Declares reusable TableGen class `SeliT` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `SeliT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 430-430
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 432-449
```tablegen
//
// op2 can be cmp or slt/sltu
// op1 can be bteqz or btnez
// the operands for op2 are two registers
// op1 is a conditional branch
//
//
// $op2 $rl, $rr  ;test registers rl,rr
// $op1 .+4       ;op2 is a conditional branch
// move $rd, $rs
//
//
class SelT<string op1, string op2>:
  MipsPseudo16<(outs CPU16Regs:$rd_),
               (ins CPU16Regs:$rd, CPU16Regs:$rs,
                CPU16Regs:$rl, CPU16Regs:$rr),
               !strconcat(op2,
               !strconcat("\t$rl, $rr\n\t",
```
- EN: Declares reusable TableGen class `SelT` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `SelT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 450-454
```tablegen
               !strconcat(op1, "\t.+4\n\tmove $rd, $rs"))), []> {
  let isCodeGenOnly=1;
  let Constraints = "$rd = $rd_";
  let usesCustomInserter = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 456-459
```tablegen
//
// 32 bit constant
//
def Constant32 : MipsPseudo16<(outs), (ins simm32:$imm), "\t.word $imm", []>;
```
- EN: Defines TableGen record `Constant32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Constant32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 461-463
```tablegen
def LwConstant32 :
  MipsPseudo16<(outs CPU16Regs:$rx), (ins simm32:$imm, simm32:$constid),
               "lw\t$rx, 1f\n\tb\t2f\n\t.align\t2\n1: \t.word\t$imm\n2:", []>;
```
- EN: Defines TableGen record `LwConstant32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LwConstant32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 465-468
```tablegen
//
// Some general instruction class info
//
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 470-475
```tablegen
class ArithLogic16Defs<bit isCom=0> {
  bits<5> shamt = 0;
  bit isCommutable = isCom;
  bit isReMaterializable = 1;
  bit hasSideEffects = 0;
}
```
- EN: Declares reusable TableGen class `ArithLogic16Defs` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `ArithLogic16Defs`，通常用于抽象共享字段、谓词或编码结构。

### Lines 477-481
```tablegen
class branch16 {
  bit isBranch = 1;
  bit isTerminator = 1;
  bit isBarrier = 1;
}
```
- EN: Declares reusable TableGen class `branch16` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `branch16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 483-486
```tablegen
class cbranch16 {
  bit isBranch = 1;
  bit isTerminator = 1;
}
```
- EN: Declares reusable TableGen class `cbranch16` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `cbranch16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 488-490
```tablegen
class MayLoad {
  bit mayLoad = 1;
}
```
- EN: Declares reusable TableGen class `MayLoad` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `MayLoad`，通常用于抽象共享字段、谓词或编码结构。

### Lines 492-495
```tablegen
class MayStore {
  bit mayStore = 1;
}
//
```
- EN: Declares reusable TableGen class `MayStore` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `MayStore`，通常用于抽象共享字段、谓词或编码结构。

### Lines 498-502
```tablegen
// Format: ADDIU rx, immediate MIPS16e
// Purpose: Add Immediate Unsigned Word (2-Operand, Extended)
// To add a constant to a 32-bit integer.
//
def AddiuRxImmX16: FEXT_RI16_ins<0b01001, "addiu">;
```
- EN: Defines TableGen record `AddiuRxImmX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AddiuRxImmX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 504-511
```tablegen
def AddiuRxRxImm16: F2RI16_ins<0b01001, "addiu">,
  ArithLogic16Defs<0> {
  let AddedComplexity = 5;
}
def AddiuRxRxImmX16: FEXT_2RI16_ins<0b01001, "addiu">,
  ArithLogic16Defs<0> {
  let isCodeGenOnly = 1;
}
```
- EN: Defines TableGen record `AddiuRxRxImm16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AddiuRxRxImm16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 513-515
```tablegen
let DecoderMethod = "DecodeFIXMEInstruction" in
def AddiuRxRyOffMemX16:
  FEXT_RRI_A16_mem_ins<0, "addiu", mem16_ea>;
```
- EN: Defines TableGen record `AddiuRxRyOffMemX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AddiuRxRyOffMemX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 517-517
```tablegen
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 519-523
```tablegen
// Format: ADDIU rx, pc, immediate MIPS16e
// Purpose: Add Immediate Unsigned Word (3-Operand, PC-Relative, Extended)
// To add a constant to the program counter.
//
def AddiuRxPcImmX16: FEXT_RI16_PC_ins<0b00001, "addiu">;
```
- EN: Defines TableGen record `AddiuRxPcImmX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AddiuRxPcImmX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 525-535
```tablegen
//
// Format: ADDIU sp, immediate MIPS16e
// Purpose: Add Immediate Unsigned Word (2-Operand, SP-Relative, Extended)
// To add a constant to the stack pointer.
//
def AddiuSpImm16
  : FI816_SP_ins<0b011, "addiu"> {
  let Defs = [SP];
  let Uses = [SP];
  let AddedComplexity = 5;
}
```
- EN: Defines TableGen record `AddiuSpImm16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AddiuSpImm16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 537-541
```tablegen
def AddiuSpImmX16
  : FEXT_I816_SP_ins<0b011, "addiu"> {
  let Defs = [SP];
  let Uses = [SP];
}
```
- EN: Defines TableGen record `AddiuSpImmX16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AddiuSpImmX16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 543-547
```tablegen
//
// Format: ADDU rz, rx, ry MIPS16e
// Purpose: Add Unsigned Word (3-Operand)
// To add 32-bit integers.
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 549-549
```tablegen
def AdduRxRyRz16: FRRR16_ins<01, "addu">, ArithLogic16Defs<1>;
```
- EN: Defines TableGen record `AdduRxRyRz16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AdduRxRyRz16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 551-554
```tablegen
//
// Format: AND rx, ry MIPS16e
// Purpose: AND
// To do a bitwise logical AND.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 556-556
```tablegen
def AndRxRxRy16: FRxRxRy16_ins<0b01100, "and">, ArithLogic16Defs<1>;
```
- EN: Defines TableGen record `AndRxRxRy16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AndRxRxRy16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 559-564
```tablegen
//
// Format: BEQZ rx, offset MIPS16e
// Purpose: Branch on Equal to Zero
// To test a GPR then do a PC-relative conditional branch.
//
def BeqzRxImm16: FRI16_B_ins<0b00100, "beqz">, cbranch16;
```
- EN: Defines TableGen record `BeqzRxImm16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BeqzRxImm16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 567-572
```tablegen
//
// Format: BEQZ rx, offset MIPS16e
// Purpose: Branch on Equal to Zero (Extended)
// To test a GPR then do a PC-relative conditional branch.
//
def BeqzRxImmX16: FEXT_RI16_B_ins<0b00100, "beqz">, cbranch16;
```
- EN: Defines TableGen record `BeqzRxImmX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BeqzRxImmX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 574-578
```tablegen
//
// Format: B offset MIPS16e
// Purpose: Unconditional Branch (Extended)
// To do an unconditional PC-relative branch.
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 580-580
```tablegen
def Bimm16: FI16_ins<0b00010, "b">, branch16;
```
- EN: Defines TableGen record `Bimm16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Bimm16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 582-586
```tablegen
// Format: B offset MIPS16e
// Purpose: Unconditional Branch
// To do an unconditional PC-relative branch.
//
def BimmX16: FEXT_I16_ins<0b00010, "b">, branch16;
```
- EN: Defines TableGen record `BimmX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BimmX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 588-593
```tablegen
//
// Format: BNEZ rx, offset MIPS16e
// Purpose: Branch on Not Equal to Zero
// To test a GPR then do a PC-relative conditional branch.
//
def BnezRxImm16: FRI16_B_ins<0b00101, "bnez">, cbranch16;
```
- EN: Defines TableGen record `BnezRxImm16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BnezRxImm16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 595-600
```tablegen
//
// Format: BNEZ rx, offset MIPS16e
// Purpose: Branch on Not Equal to Zero (Extended)
// To test a GPR then do a PC-relative conditional branch.
//
def BnezRxImmX16: FEXT_RI16_B_ins<0b00101, "bnez">, cbranch16;
```
- EN: Defines TableGen record `BnezRxImmX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BnezRxImmX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 603-606
```tablegen
//
//Format: BREAK immediate
// Purpose: Breakpoint
// To cause a Breakpoint exception.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 608-616
```tablegen
def Break16: FRRBreakNull16_ins<"break 0">;
//
// Format: BTEQZ offset MIPS16e
// Purpose: Branch on T Equal to Zero (Extended)
// To test special register T then do a PC-relative conditional branch.
//
def Bteqz16: FI816_ins<0b000, "bteqz">, cbranch16 {
  let Uses = [T8];
}
```
- EN: Defines TableGen record `Break16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Break16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 618-620
```tablegen
def BteqzX16: FEXT_I816_ins<0b000, "bteqz">, cbranch16 {
  let Uses = [T8];
}
```
- EN: Defines TableGen record `BteqzX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BteqzX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 622-622
```tablegen
def BteqzT8CmpX16: FEXT_T8I816_ins<"bteqz", "cmp">, cbranch16;
```
- EN: Defines TableGen record `BteqzT8CmpX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BteqzT8CmpX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 624-625
```tablegen
def BteqzT8CmpiX16: FEXT_T8I8I16_ins<"bteqz", "cmpi">,
  cbranch16;
```
- EN: Defines TableGen record `BteqzT8CmpiX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BteqzT8CmpiX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 627-627
```tablegen
def BteqzT8SltX16: FEXT_T8I816_ins<"bteqz", "slt">, cbranch16;
```
- EN: Defines TableGen record `BteqzT8SltX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BteqzT8SltX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 629-629
```tablegen
def BteqzT8SltuX16: FEXT_T8I816_ins<"bteqz", "sltu">, cbranch16;
```
- EN: Defines TableGen record `BteqzT8SltuX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BteqzT8SltuX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 631-631
```tablegen
def BteqzT8SltiX16: FEXT_T8I8I16_ins<"bteqz", "slti">, cbranch16;
```
- EN: Defines TableGen record `BteqzT8SltiX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BteqzT8SltiX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 633-634
```tablegen
def BteqzT8SltiuX16: FEXT_T8I8I16_ins<"bteqz", "sltiu">,
  cbranch16;
```
- EN: Defines TableGen record `BteqzT8SltiuX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BteqzT8SltiuX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 636-640
```tablegen
//
// Format: BTNEZ offset MIPS16e
// Purpose: Branch on T Not Equal to Zero (Extended)
// To test special register T then do a PC-relative conditional branch.
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 642-644
```tablegen
def Btnez16: FI816_ins<0b001, "btnez">, cbranch16 {
  let Uses = [T8];
}
```
- EN: Defines TableGen record `Btnez16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Btnez16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 646-648
```tablegen
def BtnezX16: FEXT_I816_ins<0b001, "btnez"> ,cbranch16 {
  let Uses = [T8];
}
```
- EN: Defines TableGen record `BtnezX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BtnezX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 650-650
```tablegen
def BtnezT8CmpX16: FEXT_T8I816_ins<"btnez", "cmp">, cbranch16;
```
- EN: Defines TableGen record `BtnezT8CmpX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BtnezT8CmpX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 652-652
```tablegen
def BtnezT8CmpiX16: FEXT_T8I8I16_ins<"btnez", "cmpi">, cbranch16;
```
- EN: Defines TableGen record `BtnezT8CmpiX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BtnezT8CmpiX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 654-654
```tablegen
def BtnezT8SltX16: FEXT_T8I816_ins<"btnez", "slt">, cbranch16;
```
- EN: Defines TableGen record `BtnezT8SltX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BtnezT8SltX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 656-656
```tablegen
def BtnezT8SltuX16: FEXT_T8I816_ins<"btnez", "sltu">, cbranch16;
```
- EN: Defines TableGen record `BtnezT8SltuX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BtnezT8SltuX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 658-658
```tablegen
def BtnezT8SltiX16: FEXT_T8I8I16_ins<"btnez", "slti">, cbranch16;
```
- EN: Defines TableGen record `BtnezT8SltiX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BtnezT8SltiX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 660-661
```tablegen
def BtnezT8SltiuX16: FEXT_T8I8I16_ins<"btnez", "sltiu">,
  cbranch16;
```
- EN: Defines TableGen record `BtnezT8SltiuX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BtnezT8SltiuX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 663-670
```tablegen
//
// Format: CMP rx, ry MIPS16e
// Purpose: Compare
// To compare the contents of two GPRs.
//
def CmpRxRy16: FRR16R_ins<0b01010, "cmp"> {
  let Defs = [T8];
}
```
- EN: Defines TableGen record `CmpRxRy16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CmpRxRy16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 672-679
```tablegen
//
// Format: CMPI rx, immediate MIPS16e
// Purpose: Compare Immediate
// To compare a constant with the contents of a GPR.
//
def CmpiRxImm16: FRI16R_ins<0b01110, "cmpi"> {
  let Defs = [T8];
}
```
- EN: Defines TableGen record `CmpiRxImm16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CmpiRxImm16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 681-688
```tablegen
//
// Format: CMPI rx, immediate MIPS16e
// Purpose: Compare Immediate (Extended)
// To compare a constant with the contents of a GPR.
//
def CmpiRxImmX16: FEXT_RI16R_ins<0b01110, "cmpi"> {
  let Defs = [T8];
}
```
- EN: Defines TableGen record `CmpiRxImmX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CmpiRxImmX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 691-698
```tablegen
//
// Format: DIV rx, ry MIPS16e
// Purpose: Divide Word
// To divide 32-bit signed integers.
//
def DivRxRy16: FRR16_div_ins<0b11010, "div"> {
  let Defs = [HI0, LO0];
}
```
- EN: Defines TableGen record `DivRxRy16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DivRxRy16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 700-713
```tablegen
//
// Format: DIVU rx, ry MIPS16e
// Purpose: Divide Unsigned Word
// To divide 32-bit unsigned integers.
//
def DivuRxRy16: FRR16_div_ins<0b11011, "divu"> {
  let Defs = [HI0, LO0];
}
//
// Format: JAL target MIPS16e
// Purpose: Jump and Link
// To execute a procedure call within the current 256 MB-aligned
// region and preserve the current ISA.
//
```
- EN: Defines TableGen record `DivuRxRy16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DivuRxRy16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 715-719
```tablegen
def Jal16 : FJAL16_ins<0b0, "jal"> {
  let hasDelaySlot = 0;  // not true, but we add the nop for now
  let isCall=1;
  let Defs = [RA];
}
```
- EN: Defines TableGen record `Jal16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Jal16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 721-725
```tablegen
def JalB16 : FJALB16_ins<0b0, "jal">, branch16 {
  let hasDelaySlot = 0;  // not true, but we add the nop for now
  let isBranch=1;
  let Defs = [RA];
}
```
- EN: Defines TableGen record `JalB16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JalB16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 727-732
```tablegen
//
// Format: JR ra MIPS16e
// Purpose: Jump Register Through Register ra
// To execute a branch to the instruction address in the return
// address register.
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 734-741
```tablegen
def JrRa16: FRR16_JALRC_RA_only_ins<0, 0, "jr"> {
  let isBranch = 1;
  let isIndirectBranch = 1;
  let hasDelaySlot = 1;
  let isTerminator=1;
  let isBarrier=1;
  let isReturn=1;
}
```
- EN: Defines TableGen record `JrRa16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JrRa16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 743-749
```tablegen
def JrcRa16: FRR16_JALRC_RA_only_ins<1, 1, "jrc"> {
  let isBranch = 1;
  let isIndirectBranch = 1;
  let isTerminator=1;
  let isBarrier=1;
  let isReturn=1;
}
```
- EN: Defines TableGen record `JrcRa16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JrcRa16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 751-764
```tablegen
def JrcRx16: FRR16_JALRC_ins<1, 1, 0, "jrc"> {
  let isBranch = 1;
  let isIndirectBranch = 1;
  let isTerminator=1;
  let isBarrier=1;
}
//
// Format: LB ry, offset(rx) MIPS16e
// Purpose: Load Byte (Extended)
// To load a byte from memory as a signed value.
//
def LbRxRyOffMemX16: FEXT_RRI16_mem_ins<0b10011, "lb", mem16>, MayLoad{
  let isCodeGenOnly = 1;
}
```
- EN: Defines TableGen record `JrcRx16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JrcRx16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 766-774
```tablegen
//
// Format: LBU ry, offset(rx) MIPS16e
// Purpose: Load Byte Unsigned (Extended)
// To load a byte from memory as a unsigned value.
//
def LbuRxRyOffMemX16:
  FEXT_RRI16_mem_ins<0b10100, "lbu", mem16>, MayLoad {
  let isCodeGenOnly = 1;
}
```
- EN: Defines TableGen record `LbuRxRyOffMemX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LbuRxRyOffMemX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 776-783
```tablegen
//
// Format: LH ry, offset(rx) MIPS16e
// Purpose: Load Halfword signed (Extended)
// To load a halfword from memory as a signed value.
//
def LhRxRyOffMemX16: FEXT_RRI16_mem_ins<0b10100, "lh", mem16>, MayLoad{
  let isCodeGenOnly = 1;
}
```
- EN: Defines TableGen record `LhRxRyOffMemX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LhRxRyOffMemX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 785-793
```tablegen
//
// Format: LHU ry, offset(rx) MIPS16e
// Purpose: Load Halfword unsigned (Extended)
// To load a halfword from memory as an unsigned value.
//
def LhuRxRyOffMemX16:
  FEXT_RRI16_mem_ins<0b10100, "lhu", mem16>, MayLoad {
  let isCodeGenOnly = 1;
}
```
- EN: Defines TableGen record `LhuRxRyOffMemX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LhuRxRyOffMemX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 795-800
```tablegen
//
// Format: LI rx, immediate MIPS16e
// Purpose: Load Immediate
// To load a constant into a GPR.
//
def LiRxImm16: FRI16_ins<0b01101, "li">;
```
- EN: Defines TableGen record `LiRxImm16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LiRxImm16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 802-807
```tablegen
//
// Format: LI rx, immediate MIPS16e
// Purpose: Load Immediate (Extended)
// To load a constant into a GPR.
//
def LiRxImmX16: FEXT_RI16_ins<0b01101, "li">;
```
- EN: Defines TableGen record `LiRxImmX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LiRxImmX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 809-811
```tablegen
def LiRxImmAlignX16: FEXT_RI16_ins<0b01101, ".align 2\n\tli"> {
  let isCodeGenOnly = 1;
}
```
- EN: Defines TableGen record `LiRxImmAlignX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LiRxImmAlignX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 813-820
```tablegen
//
// Format: LW ry, offset(rx) MIPS16e
// Purpose: Load Word (Extended)
// To load a word from memory as a signed value.
//
def LwRxRyOffMemX16: FEXT_RRI16_mem_ins<0b10011, "lw", mem16>, MayLoad{
  let isCodeGenOnly = 1;
}
```
- EN: Defines TableGen record `LwRxRyOffMemX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LwRxRyOffMemX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 822-827
```tablegen
// Format: LW rx, offset(sp) MIPS16e
// Purpose: Load Word (SP-Relative, Extended)
// To load an SP-relative word from memory as a signed value.
//
let DecoderMethod = "DecodeFIXMEInstruction" in
def LwRxSpImmX16: FEXT_RRI16_mem_ins<0b10010, "lw", mem16sp>, MayLoad;
```
- EN: Defines TableGen record `LwRxSpImmX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LwRxSpImmX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 829-829
```tablegen
def LwRxPcTcp16: FRI16_TCP_ins<0b10110, "lw">, MayLoad;
```
- EN: Defines TableGen record `LwRxPcTcp16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LwRxPcTcp16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 831-831
```tablegen
def LwRxPcTcpX16: FEXT_RI16_TCP_ins<0b10110, "lw">, MayLoad;
```
- EN: Defines TableGen record `LwRxPcTcpX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LwRxPcTcpX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 833-838
```tablegen
//
// Format: MOVE r32, rz MIPS16e
// Purpose: Move
// To move the contents of a GPR to a GPR.
//
def Move32R16: FI8_MOV32R16_ins<"move">;
```
- EN: Defines TableGen record `Move32R16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Move32R16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 840-847
```tablegen
//
// Format: MOVE ry, r32 MIPS16e
//Purpose: Move
// To move the contents of a GPR to a GPR.
//
def MoveR3216: FI8_MOVR3216_ins<"move"> {
  let isMoveReg = 1;
}
```
- EN: Defines TableGen record `MoveR3216:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MoveR3216:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 849-859
```tablegen
//
// Format: MFHI rx MIPS16e
// Purpose: Move From HI Register
// To copy the special purpose HI register to a GPR.
//
def Mfhi16: FRR16_M_ins<0b10000, "mfhi"> {
  let ry = 0b000; // no 'ry' field
  let Uses = [HI0];
  let hasSideEffects = 0;
  let isMoveReg = 1;
}
```
- EN: Defines TableGen record `Mfhi16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Mfhi16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 861-871
```tablegen
//
// Format: MFLO rx MIPS16e
// Purpose: Move From LO Register
// To copy the special purpose LO register to a GPR.
//
def Mflo16: FRR16_M_ins<0b10010, "mflo"> {
  let ry = 0b000; // no 'ry' field
  let Uses = [LO0];
  let hasSideEffects = 0;
  let isMoveReg = 0;
}
```
- EN: Defines TableGen record `Mflo16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Mflo16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 873-880
```tablegen
//
// Pseudo Instruction for mult
//
def MultRxRy16:  FMULT16_ins<"mult"> {
  let isCommutable = 1;
  let hasSideEffects = 0;
  let Defs = [HI0, LO0];
}
```
- EN: Defines TableGen record `MultRxRy16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MultRxRy16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 882-886
```tablegen
def MultuRxRy16: FMULT16_ins<"multu"> {
  let isCommutable = 1;
  let hasSideEffects = 0;
  let Defs = [HI0, LO0];
}
```
- EN: Defines TableGen record `MultuRxRy16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MultuRxRy16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 888-897
```tablegen
//
// Format: MULT rx, ry MIPS16e
// Purpose: Multiply Word
// To multiply 32-bit signed integers.
//
def MultRxRyRz16: FMULT16_LO_ins<"mult"> {
  let isCommutable = 1;
  let hasSideEffects = 0;
  let Defs = [HI0, LO0];
}
```
- EN: Defines TableGen record `MultRxRyRz16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MultRxRyRz16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 899-908
```tablegen
//
// Format: MULTU rx, ry MIPS16e
// Purpose: Multiply Unsigned Word
// To multiply 32-bit unsigned integers.
//
def MultuRxRyRz16: FMULT16_LO_ins<"multu"> {
  let isCommutable = 1;
  let hasSideEffects = 0;
  let Defs = [HI0, LO0];
}
```
- EN: Defines TableGen record `MultuRxRyRz16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MultuRxRyRz16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 910-915
```tablegen
//
// Format: NEG rx, ry MIPS16e
// Purpose: Negate
// To negate an integer value.
//
def NegRxRy16: FUnaryRR16_ins<0b11101, "neg">;
```
- EN: Defines TableGen record `NegRxRy16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `NegRxRy16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 917-922
```tablegen
//
// Format: NOT rx, ry MIPS16e
// Purpose: Not
// To complement an integer value
//
def NotRxRy16: FUnaryRR16_ins<0b01111, "not">;
```
- EN: Defines TableGen record `NotRxRy16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `NotRxRy16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 924-929
```tablegen
//
// Format: OR rx, ry MIPS16e
// Purpose: Or
// To do a bitwise logical OR.
//
def OrRxRxRy16: FRxRxRy16_ins<0b01101, "or">, ArithLogic16Defs<1>;
```
- EN: Defines TableGen record `OrRxRxRy16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `OrRxRxRy16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 931-938
```tablegen
//
// Format: RESTORE {ra,}{s0/s1/s0-1,}{framesize}
// (All args are optional) MIPS16e
// Purpose: Restore Registers and Deallocate Stack Frame
// To deallocate a stack frame before exit from a subroutine,
// restoring return address and static registers, and adjusting
// stack
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 940-946
```tablegen
def Restore16:
  FI8_SVRS16<0b1, (outs), (ins variable_ops),
             "", [] >, MayLoad {
  let isCodeGenOnly = 1;
  let Defs = [SP];
  let Uses = [SP];
}
```
- EN: Defines TableGen record `Restore16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Restore16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 949-955
```tablegen
def RestoreX16:
  FI8_SVRS16<0b1, (outs), (ins variable_ops),
             "", [] >, MayLoad {
  let isCodeGenOnly = 1;
  let Defs = [SP];
  let Uses = [SP];
}
```
- EN: Defines TableGen record `RestoreX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RestoreX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 957-970
```tablegen
//
// Format: SAVE {ra,}{s0/s1/s0-1,}{framesize} (All arguments are optional)
// MIPS16e
// Purpose: Save Registers and Set Up Stack Frame
// To set up a stack frame on entry to a subroutine,
// saving return address and static registers, and adjusting stack
//
def Save16:
  FI8_SVRS16<0b1, (outs), (ins variable_ops),
             "", [] >, MayStore {
  let isCodeGenOnly = 1;
  let Uses = [SP];
  let Defs = [SP];
}
```
- EN: Defines TableGen record `Save16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Save16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 972-986
```tablegen
def SaveX16:
  FI8_SVRS16<0b1, (outs), (ins variable_ops),
             "", [] >, MayStore {
  let isCodeGenOnly = 1;
  let Uses = [SP];
  let Defs = [SP];
}
//
// Format: SB ry, offset(rx) MIPS16e
// Purpose: Store Byte (Extended)
// To store a byte to memory.
//
let DecoderMethod = "DecodeFIXMEInstruction" in
def SbRxRyOffMemX16:
  FEXT_RRI16_mem2_ins<0b11000, "sb", mem16>, MayStore;
```
- EN: Defines TableGen record `SaveX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SaveX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 988-994
```tablegen
//
// Format: SEB rx MIPS16e
// Purpose: Sign-Extend Byte
// Sign-extend least significant byte in register rx.
//
def SebRx16
  : FRR_SF16_ins<0b10001, 0b100, "seb">;
```
- EN: Defines TableGen record `SebRx16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SebRx16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 996-1002
```tablegen
//
// Format: SEH rx MIPS16e
// Purpose: Sign-Extend Halfword
// Sign-extend least significant word in register rx.
//
def SehRx16
  : FRR_SF16_ins<0b10001, 0b101, "seh">;
```
- EN: Defines TableGen record `SehRx16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SehRx16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1004-1013
```tablegen
//
// The Sel(T) instructions are pseudos
// T means that they use T8 implicitly.
//
//
// Format: SelBeqZ rd, rs, rt
// Purpose: if rt==0, do nothing
//          else rs = rt
//
def SelBeqZ: Sel<"beqz">;
```
- EN: Defines TableGen record `SelBeqZ:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SelBeqZ:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1015-1021
```tablegen
//
// Format:  SelTBteqZCmp rd, rs, rl, rr
// Purpose: b = Cmp rl, rr.
//          If b==0 then do nothing.
//          if b!=0 then rd = rs
//
def SelTBteqZCmp: SelT<"bteqz", "cmp">;
```
- EN: Defines TableGen record `SelTBteqZCmp:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SelTBteqZCmp:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1023-1029
```tablegen
//
// Format:  SelTBteqZCmpi rd, rs, rl, rr
// Purpose: b = Cmpi rl, imm.
//          If b==0 then do nothing.
//          if b!=0 then rd = rs
//
def SelTBteqZCmpi: SeliT<"bteqz", "cmpi">;
```
- EN: Defines TableGen record `SelTBteqZCmpi:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SelTBteqZCmpi:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1031-1037
```tablegen
//
// Format:  SelTBteqZSlt rd, rs, rl, rr
// Purpose: b = Slt rl, rr.
//          If b==0 then do nothing.
//          if b!=0 then rd = rs
//
def SelTBteqZSlt: SelT<"bteqz", "slt">;
```
- EN: Defines TableGen record `SelTBteqZSlt:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SelTBteqZSlt:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1039-1045
```tablegen
//
// Format:  SelTBteqZSlti rd, rs, rl, rr
// Purpose: b = Slti rl, imm.
//          If b==0 then do nothing.
//          if b!=0 then rd = rs
//
def SelTBteqZSlti: SeliT<"bteqz", "slti">;
```
- EN: Defines TableGen record `SelTBteqZSlti:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SelTBteqZSlti:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1047-1053
```tablegen
//
// Format:  SelTBteqZSltu rd, rs, rl, rr
// Purpose: b = Sltu rl, rr.
//          If b==0 then do nothing.
//          if b!=0 then rd = rs
//
def SelTBteqZSltu: SelT<"bteqz", "sltu">;
```
- EN: Defines TableGen record `SelTBteqZSltu:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SelTBteqZSltu:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1055-1061
```tablegen
//
// Format:  SelTBteqZSltiu rd, rs, rl, rr
// Purpose: b = Sltiu rl, imm.
//          If b==0 then do nothing.
//          if b!=0 then rd = rs
//
def SelTBteqZSltiu: SeliT<"bteqz", "sltiu">;
```
- EN: Defines TableGen record `SelTBteqZSltiu:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SelTBteqZSltiu:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1063-1068
```tablegen
//
// Format: SelBnez rd, rs, rt
// Purpose: if rt!=0, do nothing
//          else rs = rt
//
def SelBneZ: Sel<"bnez">;
```
- EN: Defines TableGen record `SelBneZ:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SelBneZ:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1070-1076
```tablegen
//
// Format:  SelTBtneZCmp rd, rs, rl, rr
// Purpose: b = Cmp rl, rr.
//          If b!=0 then do nothing.
//          if b0=0 then rd = rs
//
def SelTBtneZCmp: SelT<"btnez", "cmp">;
```
- EN: Defines TableGen record `SelTBtneZCmp:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SelTBtneZCmp:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1078-1084
```tablegen
//
// Format:  SelTBtnezCmpi rd, rs, rl, rr
// Purpose: b = Cmpi rl, imm.
//          If b!=0 then do nothing.
//          if b==0 then rd = rs
//
def SelTBtneZCmpi: SeliT<"btnez", "cmpi">;
```
- EN: Defines TableGen record `SelTBtneZCmpi:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SelTBtneZCmpi:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1086-1092
```tablegen
//
// Format:  SelTBtneZSlt rd, rs, rl, rr
// Purpose: b = Slt rl, rr.
//          If b!=0 then do nothing.
//          if b==0 then rd = rs
//
def SelTBtneZSlt: SelT<"btnez", "slt">;
```
- EN: Defines TableGen record `SelTBtneZSlt:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SelTBtneZSlt:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1094-1100
```tablegen
//
// Format:  SelTBtneZSlti rd, rs, rl, rr
// Purpose: b = Slti rl, imm.
//          If b!=0 then do nothing.
//          if b==0 then rd = rs
//
def SelTBtneZSlti: SeliT<"btnez", "slti">;
```
- EN: Defines TableGen record `SelTBtneZSlti:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SelTBtneZSlti:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1102-1108
```tablegen
//
// Format:  SelTBtneZSltu rd, rs, rl, rr
// Purpose: b = Sltu rl, rr.
//          If b!=0 then do nothing.
//          if b==0 then rd = rs
//
def SelTBtneZSltu: SelT<"btnez", "sltu">;
```
- EN: Defines TableGen record `SelTBtneZSltu:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SelTBtneZSltu:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1110-1125
```tablegen
//
// Format:  SelTBtneZSltiu rd, rs, rl, rr
// Purpose: b = Slti rl, imm.
//          If b!=0 then do nothing.
//          if b==0 then rd = rs
//
def SelTBtneZSltiu: SeliT<"btnez", "sltiu">;
//
//
// Format: SH ry, offset(rx) MIPS16e
// Purpose: Store Halfword (Extended)
// To store a halfword to memory.
//
let DecoderMethod = "DecodeFIXMEInstruction" in
def ShRxRyOffMemX16:
  FEXT_RRI16_mem2_ins<0b11001, "sh", mem16>, MayStore;
```
- EN: Defines TableGen record `SelTBtneZSltiu:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SelTBtneZSltiu:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1127-1132
```tablegen
//
// Format: SLL rx, ry, sa MIPS16e
// Purpose: Shift Word Left Logical (Extended)
// To execute a left-shift of a word by a fixed number of bits-0 to 31 bits.
//
def SllX16: FEXT_SHIFT16_ins<0b00, "sll">;
```
- EN: Defines TableGen record `SllX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SllX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1134-1139
```tablegen
//
// Format: SLLV ry, rx MIPS16e
// Purpose: Shift Word Left Logical Variable
// To execute a left-shift of a word by a variable number of bits.
//
def SllvRxRy16 : FRxRxRy16_ins<0b00100, "sllv">;
```
- EN: Defines TableGen record `SllvRxRy16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SllvRxRy16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1141-1148
```tablegen
// Format: SLTI rx, immediate MIPS16e
// Purpose: Set on Less Than Immediate
// To record the result of a less-than comparison with a constant.
//
//
def SltiRxImm16: FRI16R_ins<0b01010, "slti"> {
  let Defs = [T8];
}
```
- EN: Defines TableGen record `SltiRxImm16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SltiRxImm16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1150-1158
```tablegen
//
// Format: SLTI rx, immediate MIPS16e
// Purpose: Set on Less Than Immediate (Extended)
// To record the result of a less-than comparison with a constant.
//
//
def SltiRxImmX16: FEXT_RI16R_ins<0b01010, "slti"> {
  let Defs = [T8];
}
```
- EN: Defines TableGen record `SltiRxImmX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SltiRxImmX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1160-1160
```tablegen
def SltiCCRxImmX16: FEXT_CCRXI16_ins<"slti">;
```
- EN: Defines TableGen record `SltiCCRxImmX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SltiCCRxImmX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1162-1169
```tablegen
// Format: SLTIU rx, immediate MIPS16e
// Purpose: Set on Less Than Immediate Unsigned
// To record the result of a less-than comparison with a constant.
//
//
def SltiuRxImm16: FRI16R_ins<0b01011, "sltiu"> {
  let Defs = [T8];
}
```
- EN: Defines TableGen record `SltiuRxImm16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SltiuRxImm16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1171-1185
```tablegen
//
// Format: SLTI rx, immediate MIPS16e
// Purpose: Set on Less Than Immediate Unsigned (Extended)
// To record the result of a less-than comparison with a constant.
//
//
def SltiuRxImmX16: FEXT_RI16R_ins<0b01011, "sltiu"> {
  let Defs = [T8];
}
//
// Format: SLTIU rx, immediate MIPS16e
// Purpose: Set on Less Than Immediate Unsigned (Extended)
// To record the result of a less-than comparison with a constant.
//
def SltiuCCRxImmX16: FEXT_CCRXI16_ins<"sltiu">;
```
- EN: Defines TableGen record `SltiuRxImmX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SltiuRxImmX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1187-1194
```tablegen
//
// Format: SLT rx, ry MIPS16e
// Purpose: Set on Less Than
// To record the result of a less-than comparison.
//
def SltRxRy16: FRR16R_ins<0b00010, "slt">{
  let Defs = [T8];
}
```
- EN: Defines TableGen record `SltRxRy16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SltRxRy16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1196-1196
```tablegen
def SltCCRxRy16: FCCRR16_ins<"slt">;
```
- EN: Defines TableGen record `SltCCRxRy16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SltCCRxRy16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1198-1204
```tablegen
// Format: SLTU rx, ry MIPS16e
// Purpose: Set on Less Than Unsigned
// To record the result of an unsigned less-than comparison.
//
def SltuRxRy16: FRR16R_ins<0b00011, "sltu">{
  let Defs = [T8];
}
```
- EN: Defines TableGen record `SltuRxRy16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SltuRxRy16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1206-1209
```tablegen
def SltuRxRyRz16: FRRTR16_ins<"sltu"> {
  let isCodeGenOnly=1;
  let Defs = [T8];
}
```
- EN: Defines TableGen record `SltuRxRyRz16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SltuRxRyRz16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1212-1219
```tablegen
def SltuCCRxRy16: FCCRR16_ins<"sltu">;
//
// Format: SRAV ry, rx MIPS16e
// Purpose: Shift Word Right Arithmetic Variable
// To execute an arithmetic right-shift of a word by a variable
// number of bits.
//
def SravRxRy16: FRxRxRy16_ins<0b00111, "srav">;
```
- EN: Defines TableGen record `SltuCCRxRy16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SltuCCRxRy16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1222-1228
```tablegen
//
// Format: SRA rx, ry, sa MIPS16e
// Purpose: Shift Word Right Arithmetic (Extended)
// To execute an arithmetic right-shift of a word by a fixed
// number of bits-1 to 8 bits.
//
def SraX16: FEXT_SHIFT16_ins<0b11, "sra">;
```
- EN: Defines TableGen record `SraX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SraX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1231-1237
```tablegen
//
// Format: SRLV ry, rx MIPS16e
// Purpose: Shift Word Right Logical Variable
// To execute a logical right-shift of a word by a variable
// number of bits.
//
def SrlvRxRy16: FRxRxRy16_ins<0b00110, "srlv">;
```
- EN: Defines TableGen record `SrlvRxRy16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SrlvRxRy16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1240-1246
```tablegen
//
// Format: SRL rx, ry, sa MIPS16e
// Purpose: Shift Word Right Logical (Extended)
// To execute a logical right-shift of a word by a fixed
// number of bits-1 to 31 bits.
//
def SrlX16: FEXT_SHIFT16_ins<0b10, "srl">;
```
- EN: Defines TableGen record `SrlX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SrlX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1248-1253
```tablegen
//
// Format: SUBU rz, rx, ry MIPS16e
// Purpose: Subtract Unsigned Word
// To subtract 32-bit integers
//
def SubuRxRyRz16: FRRR16_ins<0b11, "subu">, ArithLogic16Defs<0>;
```
- EN: Defines TableGen record `SubuRxRyRz16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SubuRxRyRz16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1255-1261
```tablegen
//
// Format: SW ry, offset(rx) MIPS16e
// Purpose: Store Word (Extended)
// To store a word to memory.
//
let DecoderMethod = "DecodeFIXMEInstruction" in
def SwRxRyOffMemX16: FEXT_RRI16_mem2_ins<0b11011, "sw", mem16>, MayStore;
```
- EN: Defines TableGen record `SwRxRyOffMemX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwRxRyOffMemX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1263-1269
```tablegen
//
// Format: SW rx, offset(sp) MIPS16e
// Purpose: Store Word rx (SP-Relative)
// To store an SP-relative word to memory.
//
let DecoderMethod = "DecodeFIXMEInstruction" in
def SwRxSpImmX16: FEXT_RRI16_mem2_ins<0b11010, "sw", mem16sp>, MayStore;
```
- EN: Defines TableGen record `SwRxSpImmX16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwRxSpImmX16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1271-1277
```tablegen
//
//
// Format: XOR rx, ry MIPS16e
// Purpose: Xor
// To do a bitwise logical XOR.
//
def XorRxRxRy16: FRxRxRy16_ins<0b01110, "xor">, ArithLogic16Defs<1>;
```
- EN: Defines TableGen record `XorRxRxRy16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `XorRxRxRy16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1279-1281
```tablegen
class Mips16Pat<dag pattern, dag result> : Pat<pattern, result> {
  let Predicates = [InMips16Mode];
}
```
- EN: Declares reusable TableGen class `Mips16Pat` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `Mips16Pat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1283-1287
```tablegen
// Unary Arith/Logic
//
class ArithLogicU_pat<PatFrag OpNode, Instruction I> :
  Mips16Pat<(OpNode CPU16Regs:$r),
            (I CPU16Regs:$r)>;
```
- EN: Declares reusable TableGen class `ArithLogicU_pat` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `ArithLogicU_pat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1289-1290
```tablegen
def: ArithLogicU_pat<not, NotRxRy16>;
def: ArithLogicU_pat<ineg, NegRxRy16>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1292-1294
```tablegen
class ArithLogic16_pat<SDNode OpNode, Instruction I> :
  Mips16Pat<(OpNode CPU16Regs:$l, CPU16Regs:$r),
            (I CPU16Regs:$l, CPU16Regs:$r)>;
```
- EN: Declares reusable TableGen class `ArithLogic16_pat` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `ArithLogic16_pat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1296-1301
```tablegen
def: ArithLogic16_pat<add, AdduRxRyRz16>;
def: ArithLogic16_pat<and, AndRxRxRy16>;
def: ArithLogic16_pat<mul, MultRxRyRz16>;
def: ArithLogic16_pat<or, OrRxRxRy16>;
def: ArithLogic16_pat<sub, SubuRxRyRz16>;
def: ArithLogic16_pat<xor, XorRxRxRy16>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1303-1303
```tablegen
// Arithmetic and logical instructions with 2 register operands.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1305-1307
```tablegen
class ArithLogicI16_pat<SDNode OpNode, PatFrag imm_type, Instruction I> :
  Mips16Pat<(OpNode CPU16Regs:$in, imm_type:$imm),
            (I CPU16Regs:$in, imm_type:$imm)>;
```
- EN: Declares reusable TableGen class `ArithLogicI16_pat` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `ArithLogicI16_pat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1309-1313
```tablegen
def: ArithLogicI16_pat<add, immSExt8, AddiuRxRxImm16>;
def: ArithLogicI16_pat<add, immSExt16, AddiuRxRxImmX16>;
def: ArithLogicI16_pat<shl, immZExt5, SllX16>;
def: ArithLogicI16_pat<srl, immZExt5, SrlX16>;
def: ArithLogicI16_pat<sra, immZExt5, SraX16>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1315-1317
```tablegen
class shift_rotate_reg16_pat<SDNode OpNode, Instruction I> :
  Mips16Pat<(OpNode CPU16Regs:$r, CPU16Regs:$ra),
            (I CPU16Regs:$r, CPU16Regs:$ra)>;
```
- EN: Declares reusable TableGen class `shift_rotate_reg16_pat` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `shift_rotate_reg16_pat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1319-1321
```tablegen
def: shift_rotate_reg16_pat<shl, SllvRxRy16>;
def: shift_rotate_reg16_pat<sra, SravRxRy16>;
def: shift_rotate_reg16_pat<srl, SrlvRxRy16>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1323-1324
```tablegen
class LoadM16_pat<PatFrag OpNode, Instruction I, ComplexPattern Addr> :
  Mips16Pat<(OpNode Addr:$addr), (I Addr:$addr)>;
```
- EN: Declares reusable TableGen class `LoadM16_pat` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `LoadM16_pat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1326-1330
```tablegen
def: LoadM16_pat<sextloadi8, LbRxRyOffMemX16, addr16>;
def: LoadM16_pat<zextloadi8, LbuRxRyOffMemX16, addr16>;
def: LoadM16_pat<sextloadi16, LhRxRyOffMemX16, addr16>;
def: LoadM16_pat<zextloadi16, LhuRxRyOffMemX16, addr16>;
def: LoadM16_pat<load, LwRxSpImmX16, addr16sp>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1332-1333
```tablegen
class StoreM16_pat<PatFrag OpNode, Instruction I, ComplexPattern Addr> :
  Mips16Pat<(OpNode CPU16Regs:$r, Addr:$addr), (I CPU16Regs:$r, Addr:$addr)>;
```
- EN: Declares reusable TableGen class `StoreM16_pat` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `StoreM16_pat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1335-1337
```tablegen
def: StoreM16_pat<truncstorei8, SbRxRyOffMemX16, addr16>;
def: StoreM16_pat<truncstorei16, ShRxRyOffMemX16, addr16>;
def: StoreM16_pat<store, SwRxSpImmX16, addr16sp>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1339-1343
```tablegen
// Unconditional branch
class UncondBranch16_pat<SDNode OpNode, Instruction I>:
  Mips16Pat<(OpNode bb:$imm16), (I bb:$imm16)> {
    let Predicates = [InMips16Mode];
  }
```
- EN: Declares reusable TableGen class `UncondBranch16_pat` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `UncondBranch16_pat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1345-1346
```tablegen
def : Mips16Pat<(MipsJmpLink (i32 tglobaladdr:$dst)),
                (Jal16 tglobaladdr:$dst)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1348-1349
```tablegen
def : Mips16Pat<(MipsJmpLink (i32 texternalsym:$dst)),
                (Jal16 texternalsym:$dst)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1351-1356
```tablegen
// Indirect branch
def: Mips16Pat<(brind CPU16Regs:$rs), (JrcRx16 CPU16Regs:$rs)> {
  // Ensure that the addition of MIPS32r6/MIPS64r6 support does not change
  // MIPS16's behaviour.
  let AddedComplexity = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1358-1364
```tablegen
// Jump and Link (Call)
let isCall=1, hasDelaySlot=0 in
def JumpLinkReg16:
  FRR16_JALRC<0, 0, 0, (outs), (ins CPU16Regs:$rx),
              "jalrc\t$rx", [(MipsJmpLink CPU16Regs:$rx)]> {
  let Defs = [RA];
}
```
- EN: Defines TableGen record `JumpLinkReg16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JumpLinkReg16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1366-1369
```tablegen
// Mips16 pseudos
let isReturn=1, isTerminator=1, hasDelaySlot=1, isBarrier=1, hasCtrlDep=1,
  hasExtraSrcRegAllocReq = 1 in
def RetRA16 : MipsPseudo16<(outs), (ins), "", [(MipsRet)]>;
```
- EN: Defines TableGen record `RetRA16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RetRA16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1372-1372
```tablegen
// setcc patterns
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1374-1376
```tablegen
class SetCC_R16<PatFrag cond_op, Instruction I>:
  Mips16Pat<(cond_op CPU16Regs:$rx, CPU16Regs:$ry),
            (I CPU16Regs:$rx, CPU16Regs:$ry)>;
```
- EN: Declares reusable TableGen class `SetCC_R16` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `SetCC_R16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1378-1380
```tablegen
class SetCC_I16<PatFrag cond_op, PatLeaf imm_type, Instruction I>:
  Mips16Pat<(cond_op CPU16Regs:$rx, imm_type:$imm16),
            (I CPU16Regs:$rx, imm_type:$imm16)>;
```
- EN: Declares reusable TableGen class `SetCC_I16` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `SetCC_I16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1383-1383
```tablegen
def: Mips16Pat<(i32 addr16sp:$addr), (AddiuRxRyOffMemX16 addr16sp:$addr)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1386-1387
```tablegen
// Large (>16 bit) immediate loads
def : Mips16Pat<(i32 imm:$imm), (LwConstant32 imm:$imm, -1)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1389-1406
```tablegen
//
// Some branch conditional patterns are not generated by llvm at this time.
// Some are for seemingly arbitrary reasons not used: i.e. with signed number
// comparison they are used and for unsigned a different pattern is used.
// I am pushing upstream from the full mips16 port and it seemed that I needed
// these earlier and the mips32 port has these but now I cannot create test
// cases that use these patterns. While I sort this all out I will leave these
// extra patterns commented out and if I can be sure they are really not used,
// I will delete the code. I don't want to check the code in uncommented without
// a valid test case. In some cases, the compiler is generating patterns with
// setcc instead and earlier I had implemented setcc first so may have masked
// the problem. The setcc variants are suboptimal for mips16 so I may wantto
// figure out how to enable the brcond patterns or else possibly new
// combinations of brcond and setcc.
//
//
// bcond-seteq
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1407-1410
```tablegen
def: Mips16Pat
  <(brcond (i32 (seteq CPU16Regs:$rx, CPU16Regs:$ry)), bb:$imm16),
   (BteqzT8CmpX16 CPU16Regs:$rx, CPU16Regs:$ry,  bb:$imm16)
  >;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1413-1416
```tablegen
def: Mips16Pat
  <(brcond (i32 (seteq CPU16Regs:$rx, immZExt16:$imm)), bb:$targ16),
   (BteqzT8CmpiX16 CPU16Regs:$rx, immSExt16:$imm,  bb:$targ16)
  >;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1418-1421
```tablegen
def: Mips16Pat
  <(brcond (i32 (seteq CPU16Regs:$rx, 0)), bb:$targ16),
   (BeqzRxImm16 CPU16Regs:$rx, bb:$targ16)
  >;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1423-1429
```tablegen
//
// bcond-setgt (do we need to have this pair of setlt, setgt??)
//
def: Mips16Pat
  <(brcond (i32 (setgt CPU16Regs:$rx, CPU16Regs:$ry)), bb:$imm16),
   (BtnezT8SltX16 CPU16Regs:$ry, CPU16Regs:$rx,  bb:$imm16)
  >;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1431-1437
```tablegen
//
// bcond-setge
//
def: Mips16Pat
  <(brcond (i32 (setge CPU16Regs:$rx, CPU16Regs:$ry)), bb:$imm16),
   (BteqzT8SltX16 CPU16Regs:$rx, CPU16Regs:$ry,  bb:$imm16)
  >;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1439-1444
```tablegen
//
// never called because compiler transforms a >= k to a > (k-1)
def: Mips16Pat
  <(brcond (i32 (setge CPU16Regs:$rx, immSExt16:$imm)), bb:$imm16),
   (BteqzT8SltiX16 CPU16Regs:$rx, immSExt16:$imm,  bb:$imm16)
  >;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1446-1452
```tablegen
//
// bcond-setlt
//
def: Mips16Pat
  <(brcond (i32 (setlt CPU16Regs:$rx, CPU16Regs:$ry)), bb:$imm16),
   (BtnezT8SltX16 CPU16Regs:$rx, CPU16Regs:$ry,  bb:$imm16)
  >;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1454-1457
```tablegen
def: Mips16Pat
  <(brcond (i32 (setlt CPU16Regs:$rx, immSExt16:$imm)), bb:$imm16),
   (BtnezT8SltiX16 CPU16Regs:$rx, immSExt16:$imm,  bb:$imm16)
  >;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1459-1465
```tablegen
//
// bcond-setle
//
def: Mips16Pat
  <(brcond (i32 (setle CPU16Regs:$rx, CPU16Regs:$ry)), bb:$imm16),
   (BteqzT8SltX16 CPU16Regs:$ry, CPU16Regs:$rx,  bb:$imm16)
  >;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1467-1473
```tablegen
//
// bcond-setne
//
def: Mips16Pat
  <(brcond (i32 (setne CPU16Regs:$rx, CPU16Regs:$ry)), bb:$imm16),
   (BtnezT8CmpX16 CPU16Regs:$rx, CPU16Regs:$ry,  bb:$imm16)
  >;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1475-1478
```tablegen
def: Mips16Pat
  <(brcond (i32 (setne CPU16Regs:$rx, immZExt16:$imm)), bb:$targ16),
   (BtnezT8CmpiX16 CPU16Regs:$rx, immSExt16:$imm,  bb:$targ16)
  >;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1480-1483
```tablegen
def: Mips16Pat
  <(brcond (i32 (setne CPU16Regs:$rx, 0)), bb:$targ16),
   (BnezRxImm16 CPU16Regs:$rx, bb:$targ16)
  >;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1485-1491
```tablegen
//
// This needs to be there but I forget which code will generate it
//
def: Mips16Pat
  <(brcond CPU16Regs:$rx, bb:$targ16),
   (BnezRxImm16 CPU16Regs:$rx, bb:$targ16)
  >;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1493-1493
```tablegen
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1495-1501
```tablegen
//
// bcond-setugt
//
//def: Mips16Pat
//  <(brcond (i32 (setugt CPU16Regs:$rx, CPU16Regs:$ry)), bb:$imm16),
//   (BtnezT8SltuX16 CPU16Regs:$ry, CPU16Regs:$rx,  bb:$imm16)
//  >;
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1503-1509
```tablegen
//
// bcond-setuge
//
//def: Mips16Pat
//  <(brcond (i32 (setuge CPU16Regs:$rx, CPU16Regs:$ry)), bb:$imm16),
//   (BteqzT8SltuX16 CPU16Regs:$rx, CPU16Regs:$ry,  bb:$imm16)
//  >;
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1512-1518
```tablegen
//
// bcond-setult
//
//def: Mips16Pat
//  <(brcond (i32 (setult CPU16Regs:$rx, CPU16Regs:$ry)), bb:$imm16),
//   (BtnezT8SltuX16 CPU16Regs:$rx, CPU16Regs:$ry,  bb:$imm16)
//  >;
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1520-1520
```tablegen
def: UncondBranch16_pat<br, Bimm16>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1522-1524
```tablegen
// Small immediates
def: Mips16Pat<(i32 immSExt16:$in),
               (AddiuRxRxImmX16 (MoveR3216 ZERO), immSExt16:$in)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1526-1526
```tablegen
def: Mips16Pat<(i32 immZExt16:$in), (LiRxImmX16 immZExt16:$in)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1528-1533
```tablegen
//
// MipsDivRem
//
def: Mips16Pat
  <(MipsDivRem16 CPU16Regs:$rx, CPU16Regs:$ry),
   (DivRxRy16 CPU16Regs:$rx, CPU16Regs:$ry)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1535-1540
```tablegen
//
// MipsDivRemU
//
def: Mips16Pat
  <(MipsDivRemU16 CPU16Regs:$rx, CPU16Regs:$ry),
   (DivuRxRy16 CPU16Regs:$rx, CPU16Regs:$ry)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1542-1550
```tablegen
//  signed a,b
//  x = (a>=b)?x:y
//
//  if !(a < b) x = y
//
def : Mips16Pat<(select (i32 (setge CPU16Regs:$a, CPU16Regs:$b)),
                 CPU16Regs:$x, CPU16Regs:$y),
                (SelTBteqZSlt CPU16Regs:$x, CPU16Regs:$y,
                 CPU16Regs:$a, CPU16Regs:$b)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1552-1560
```tablegen
//  signed a,b
//  x = (a>b)?x:y
//
//  if  (b < a) x = y
//
def : Mips16Pat<(select (i32 (setgt CPU16Regs:$a, CPU16Regs:$b)),
                 CPU16Regs:$x, CPU16Regs:$y),
                (SelTBtneZSlt CPU16Regs:$x, CPU16Regs:$y,
                 CPU16Regs:$b, CPU16Regs:$a)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1562-1571
```tablegen
// unsigned a,b
// x = (a>=b)?x:y
//
// if !(a < b) x = y;
//
def : Mips16Pat<
  (select (i32 (setuge CPU16Regs:$a, CPU16Regs:$b)),
   CPU16Regs:$x, CPU16Regs:$y),
  (SelTBteqZSltu CPU16Regs:$x, CPU16Regs:$y,
   CPU16Regs:$a, CPU16Regs:$b)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1573-1581
```tablegen
//  unsigned a,b
//  x = (a>b)?x:y
//
//  if (b < a) x = y
//
def : Mips16Pat<(select (i32 (setugt CPU16Regs:$a, CPU16Regs:$b)),
                 CPU16Regs:$x, CPU16Regs:$y),
                (SelTBtneZSltu CPU16Regs:$x, CPU16Regs:$y,
                 CPU16Regs:$b, CPU16Regs:$a)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1583-1588
```tablegen
// signed
// x = (a >= k)?x:y
// due to an llvm optimization, i don't think that this will ever
// be used. This is transformed into x = (a > k-1)?x:y
//
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1590-1594
```tablegen
//def : Mips16Pat<
//  (select (i32 (setge CPU16Regs:$lhs, immSExt16:$rhs)),
//   CPU16Regs:$T, CPU16Regs:$F),
//  (SelTBteqZSlti CPU16Regs:$T, CPU16Regs:$F,
//   CPU16Regs:$lhs, immSExt16:$rhs)>;
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1596-1600
```tablegen
//def : Mips16Pat<
//  (select (i32 (setuge CPU16Regs:$lhs, immSExt16:$rhs)),
//   CPU16Regs:$T, CPU16Regs:$F),
//  (SelTBteqZSltiu CPU16Regs:$T, CPU16Regs:$F,
//   CPU16Regs:$lhs, immSExt16:$rhs)>;
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1602-1611
```tablegen
// signed
// x = (a < k)?x:y
//
// if !(a < k) x = y;
//
def : Mips16Pat<
  (select (i32 (setlt CPU16Regs:$a, immSExt16:$b)),
   CPU16Regs:$x, CPU16Regs:$y),
  (SelTBtneZSlti CPU16Regs:$x, CPU16Regs:$y,
   CPU16Regs:$a, immSExt16:$b)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1614-1624
```tablegen
//
//
// signed
// x = (a <= b)? x : y
//
// if  (b < a) x = y
//
def : Mips16Pat<(select (i32 (setle CPU16Regs:$a, CPU16Regs:$b)),
                 CPU16Regs:$x, CPU16Regs:$y),
                (SelTBteqZSlt CPU16Regs:$x, CPU16Regs:$y,
                 CPU16Regs:$b, CPU16Regs:$a)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1626-1635
```tablegen
//
// unsigned
// x = (a <= b)? x : y
//
// if  (b < a) x = y
//
def : Mips16Pat<(select (i32 (setule CPU16Regs:$a, CPU16Regs:$b)),
                 CPU16Regs:$x, CPU16Regs:$y),
                (SelTBteqZSltu CPU16Regs:$x, CPU16Regs:$y,
                 CPU16Regs:$b, CPU16Regs:$a)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1637-1646
```tablegen
//
// signed/unsigned
// x = (a == b)? x : y
//
// if (a != b) x = y
//
def : Mips16Pat<(select (i32 (seteq CPU16Regs:$a, CPU16Regs:$b)),
                 CPU16Regs:$x, CPU16Regs:$y),
                (SelTBteqZCmp CPU16Regs:$x, CPU16Regs:$y,
                 CPU16Regs:$b, CPU16Regs:$a)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1648-1657
```tablegen
//
// signed/unsigned
// x = (a == 0)? x : y
//
// if (a != 0) x = y
//
def : Mips16Pat<(select (i32 (seteq CPU16Regs:$a, 0)),
                 CPU16Regs:$x, CPU16Regs:$y),
                (SelBeqZ CPU16Regs:$x, CPU16Regs:$y,
                 CPU16Regs:$a)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1660-1669
```tablegen
//
// signed/unsigned
// x = (a == k)? x : y
//
// if (a != k) x = y
//
def : Mips16Pat<(select (i32 (seteq CPU16Regs:$a, immZExt16:$k)),
                 CPU16Regs:$x, CPU16Regs:$y),
                (SelTBteqZCmpi CPU16Regs:$x, CPU16Regs:$y,
                 CPU16Regs:$a, immZExt16:$k)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1672-1682
```tablegen
//
// signed/unsigned
// x = (a != b)? x : y
//
// if (a == b) x = y
//
//
def : Mips16Pat<(select (i32 (setne CPU16Regs:$a, CPU16Regs:$b)),
                 CPU16Regs:$x, CPU16Regs:$y),
                (SelTBtneZCmp CPU16Regs:$x, CPU16Regs:$y,
                 CPU16Regs:$b, CPU16Regs:$a)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1684-1693
```tablegen
//
// signed/unsigned
// x = (a != 0)? x : y
//
// if (a == 0) x = y
//
def : Mips16Pat<(select (i32 (setne CPU16Regs:$a, 0)),
                 CPU16Regs:$x, CPU16Regs:$y),
                (SelBneZ CPU16Regs:$x, CPU16Regs:$y,
                 CPU16Regs:$a)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1695-1703
```tablegen
// signed/unsigned
// x = (a)? x : y
//
// if (!a) x = y
//
def : Mips16Pat<(select  CPU16Regs:$a,
                 CPU16Regs:$x, CPU16Regs:$y),
      (SelBneZ CPU16Regs:$x, CPU16Regs:$y,
       CPU16Regs:$a)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1706-1715
```tablegen
//
// signed/unsigned
// x = (a != k)? x : y
//
// if (a == k) x = y
//
def : Mips16Pat<(select (i32 (setne CPU16Regs:$a, immZExt16:$k)),
                 CPU16Regs:$x, CPU16Regs:$y),
                (SelTBtneZCmpi CPU16Regs:$x, CPU16Regs:$y,
                 CPU16Regs:$a, immZExt16:$k)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1717-1726
```tablegen
//
// When writing C code to test setxx these patterns,
// some will be transformed into
// other things. So we test using C code but using -O3 and -O0
//
// seteq
//
def : Mips16Pat
  <(seteq CPU16Regs:$lhs,CPU16Regs:$rhs),
   (SltiuCCRxImmX16 (XorRxRxRy16 CPU16Regs:$lhs, CPU16Regs:$rhs), 1)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1728-1730
```tablegen
def : Mips16Pat
  <(seteq CPU16Regs:$lhs, 0),
   (SltiuCCRxImmX16 CPU16Regs:$lhs, 1)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1733-1735
```tablegen
//
// setge
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1737-1740
```tablegen
def: Mips16Pat
  <(setge CPU16Regs:$lhs, CPU16Regs:$rhs),
   (XorRxRxRy16 (SltCCRxRy16 CPU16Regs:$lhs, CPU16Regs:$rhs),
   (LiRxImmX16 1))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1742-1752
```tablegen
//
// For constants, llvm transforms this to:
// x > (k - 1) and then reverses the operands to use setlt. So this pattern
// is not used now by the compiler. (Presumably checking that k-1 does not
// overflow). The compiler never uses this at the current time, due to
// other optimizations.
//
//def: Mips16Pat
//  <(setge CPU16Regs:$lhs, immSExt16:$rhs),
//   (XorRxRxRy16 (SltiCCRxImmX16 CPU16Regs:$lhs, immSExt16:$rhs),
//   (LiRxImmX16 1))>;
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1754-1759
```tablegen
// This catches the x >= -32768 case by transforming it to  x > -32769
//
def: Mips16Pat
  <(setgt CPU16Regs:$lhs, -32769),
   (XorRxRxRy16 (SltiCCRxImmX16 CPU16Regs:$lhs, -32768),
   (LiRxImmX16 1))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1761-1764
```tablegen
//
// setgt
//
//
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1766-1768
```tablegen
def: Mips16Pat
  <(setgt CPU16Regs:$lhs, CPU16Regs:$rhs),
   (SltCCRxRy16 CPU16Regs:$rhs, CPU16Regs:$lhs)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1770-1775
```tablegen
//
// setle
//
def: Mips16Pat
  <(setle CPU16Regs:$lhs, CPU16Regs:$rhs),
   (XorRxRxRy16 (SltCCRxRy16 CPU16Regs:$rhs, CPU16Regs:$lhs), (LiRxImm16 1))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1777-1780
```tablegen
//
// setlt
//
def: SetCC_R16<setlt, SltCCRxRy16>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1782-1782
```tablegen
def: SetCC_I16<setlt, immSExt16, SltiCCRxImmX16>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1784-1790
```tablegen
//
// setne
//
def : Mips16Pat
  <(setne CPU16Regs:$lhs,CPU16Regs:$rhs),
   (SltuCCRxRy16 (LiRxImmX16 0),
   (XorRxRxRy16 CPU16Regs:$lhs, CPU16Regs:$rhs))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1793-1799
```tablegen
//
// setuge
//
def: Mips16Pat
  <(setuge CPU16Regs:$lhs, CPU16Regs:$rhs),
   (XorRxRxRy16 (SltuCCRxRy16 CPU16Regs:$lhs, CPU16Regs:$rhs),
   (LiRxImmX16 1))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1801-1807
```tablegen
// this pattern will never be used because the compiler will transform
// x >= k to x > (k - 1) and then use SLT
//
//def: Mips16Pat
//  <(setuge CPU16Regs:$lhs, immZExt16:$rhs),
//   (XorRxRxRy16 (SltiuCCRxImmX16 CPU16Regs:$lhs, immZExt16:$rhs),
//   (LiRxImmX16 1))>;
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1809-1814
```tablegen
//
// setugt
//
def: Mips16Pat
  <(setugt CPU16Regs:$lhs, CPU16Regs:$rhs),
   (SltuCCRxRy16 CPU16Regs:$rhs, CPU16Regs:$lhs)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1816-1821
```tablegen
//
// setule
//
def: Mips16Pat
  <(setule CPU16Regs:$lhs, CPU16Regs:$rhs),
   (XorRxRxRy16 (SltuCCRxRy16 CPU16Regs:$rhs, CPU16Regs:$lhs), (LiRxImmX16 1))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1823-1826
```tablegen
//
// setult
//
def: SetCC_R16<setult, SltuCCRxRy16>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1828-1828
```tablegen
def: SetCC_I16<setult, immSExt16, SltiuCCRxImmX16>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1830-1831
```tablegen
def: Mips16Pat<(add CPU16Regs:$hi, (MipsLo tglobaladdr:$lo)),
               (AddiuRxRxImmX16 CPU16Regs:$hi, tglobaladdr:$lo)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1833-1839
```tablegen
// hi/lo relocs
def : Mips16Pat<(MipsHi tblockaddress:$in),
                (SllX16 (LiRxImmX16 tblockaddress:$in), 16)>;
def : Mips16Pat<(MipsHi tglobaladdr:$in),
                (SllX16 (LiRxImmX16 tglobaladdr:$in), 16)>;
def : Mips16Pat<(MipsHi tjumptable:$in),
                (SllX16 (LiRxImmX16 tjumptable:$in), 16)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1841-1841
```tablegen
def : Mips16Pat<(MipsLo tblockaddress:$in), (LiRxImmX16 tblockaddress:$in)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1843-1844
```tablegen
def : Mips16Pat<(MipsTlsHi tglobaltlsaddr:$in),
                (SllX16 (LiRxImmX16 tglobaltlsaddr:$in), 16)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1846-1849
```tablegen
// wrapper_pic
class Wrapper16Pat<SDNode node, Instruction ADDiuOp, RegisterClass RC>:
  Mips16Pat<(MipsWrapper RC:$gp, node:$in),
            (ADDiuOp RC:$gp, node:$in)>;
```
- EN: Declares reusable TableGen class `Wrapper16Pat` for `Mips16InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips16InstrInfo` 声明可复用的 TableGen 类 `Wrapper16Pat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1852-1853
```tablegen
def : Wrapper16Pat<tglobaladdr, AddiuRxRxImmX16, CPU16Regs>;
def : Wrapper16Pat<tglobaltlsaddr, AddiuRxRxImmX16, CPU16Regs>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1855-1858
```tablegen
def : Mips16Pat<(i32 (extloadi8   addr16:$src)),
                (LbuRxRyOffMemX16  addr16:$src)>;
def : Mips16Pat<(i32 (extloadi16  addr16:$src)),
                (LhuRxRyOffMemX16  addr16:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1860-1860
```tablegen
def: Mips16Pat<(trap), (Break16)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1862-1863
```tablegen
def : Mips16Pat<(sext_inreg CPU16Regs:$val, i8),
                (SebRx16 CPU16Regs:$val)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1865-1866
```tablegen
def : Mips16Pat<(sext_inreg CPU16Regs:$val, i16),
                (SehRx16 CPU16Regs:$val)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1868-1872
```tablegen
def GotPrologue16:
  MipsPseudo16<
    (outs CPU16Regs:$rh, CPU16Regs:$rl),
    (ins simm16:$immHi, simm16:$immLo),
    "li\t$rh, $immHi\n\taddiu\t$rl, $$pc, $immLo\n ",[]> ;
```
- EN: Defines TableGen record `GotPrologue16:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GotPrologue16:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1874-1877
```tablegen
// An operand for the CONSTPOOL_ENTRY pseudo-instruction.
def cpinst_operand : Operand<i32> {
  // let PrintMethod = "printCPInstOperand";
}
```
- EN: Defines TableGen record `cpinst_operand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `cpinst_operand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1879-1887
```tablegen
// CONSTPOOL_ENTRY - This instruction represents a floating constant pool in
// the function.  The first operand is the ID# for this instruction, the second
// is the index into the MachineConstantPool that this is, the third is the
// size in bytes of this constant pool entry.
//
let hasSideEffects = 0, isNotDuplicable = 1 in
def CONSTPOOL_ENTRY :
MipsPseudo16<(outs), (ins cpinst_operand:$instid, cpinst_operand:$cpidx,
                      i32imm:$size), "foo", []>;
```
- EN: Defines TableGen record `CONSTPOOL_ENTRY` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CONSTPOOL_ENTRY`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1889-1889
```tablegen
// Instruction Aliases
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1891-1892
```tablegen
let EncodingPredicates = [InMips16Mode] in
def : MipsInstAlias<"nop", (Move32R16 ZERO, S0)>;
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
- EN: Bitfield assignments describe exact instruction encodings and per-instruction flags.
  - CN: 位字段赋值用于描述精确的指令编码以及每条指令的标志位。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
