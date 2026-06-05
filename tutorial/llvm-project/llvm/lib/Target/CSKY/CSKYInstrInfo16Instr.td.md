# CSKYInstrInfo16Instr.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYInstrInfo16Instr.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file describes the CSKY 16-bit instructions in TableGen format.
- 目的（中文）: 使用 TableGen 定义目标指令，描述操作码、操作数、匹配模式与调度信息。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===-- CSKYInstrInfo16Instr.td - CSKY 16-bit Instruction --*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file describes the CSKY 16-bit instructions in TableGen format.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: //===----------------------------------------------------------------------===//
  14: // CSKY specific DAG Nodes.
  15: //===----------------------------------------------------------------------===//
  16: 
  17: // Target-dependent nodes.
  18: def CSKY_NIE : SDNode<"CSKYISD::NIE", SDTNone,
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This range defines declarative TableGen records such as CSKY_NIE, capturing target metadata instead of imperative runtime logic.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段定义了 CSKY_NIE 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 19-36

```tablegen
  19:     [SDNPHasChain, SDNPOptInGlue]>;
  20: def CSKY_NIR : SDNode<"CSKYISD::NIR", SDTNone,
  21:     [SDNPHasChain, SDNPOptInGlue]>;
  22: 
  23: //===----------------------------------------------------------------------===//
  24: // Operand and SDNode transformation definitions.
  25: //===----------------------------------------------------------------------===//
  26: 
  27: def br_symbol_16bit : Operand<OtherVT> {
  28:   let EncoderMethod =
  29:     "getBranchSymbolOpValue<CSKY::fixup_csky_pcrel_imm10_scale2>";
  30:   let ParserMatchClass = CSKYSymbol;
  31:   let DecoderMethod = "decodeSImmOperand<10, 1>";
  32:   let PrintMethod = "printCSKYSymbolOperand";
  33:   let OperandType = "OPERAND_PCREL";
  34: }
  35: 
  36: def constpool_symbol_16bit : Operand<iPTR> {
```

- EN: This range defines declarative TableGen records such as CSKY_NIR, br_symbol_16bit, constpool_symbol_16bit, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CSKY_NIR, br_symbol_16bit, constpool_symbol_16bit 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 37-54

```tablegen
  37:   let ParserMatchClass = Constpool;
  38:   let EncoderMethod =
  39:     "getConstpoolSymbolOpValue<CSKY::fixup_csky_pcrel_uimm7_scale4>";
  40:   let DecoderMethod = "decodeLRW16Imm8";
  41:   let PrintMethod = "printConstpool";
  42:   let OperandType = "OPERAND_PCREL";
  43: }
  44: 
  45: //===----------------------------------------------------------------------===//
  46: // Instruction Formats
  47: //===----------------------------------------------------------------------===//
  48: 
  49: include "CSKYInstrFormats16Instr.td"
  50: 
  51: //===----------------------------------------------------------------------===//
  52: // Instruction definitions.
  53: //===----------------------------------------------------------------------===//
  54: 
```

- EN: It imports other TableGen fragments so the current description can reuse shared records, register sets, or instruction definitions. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 它引入其他 TableGen 片段，以便当前描述复用共享记录、寄存器集合或指令定义。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 55-72

```tablegen
  55: //===----------------------------------------------------------------------===//
  56: // Basic ALU instructions.
  57: //===----------------------------------------------------------------------===//
  58: 
  59: let isCommutable = 1, isAdd = 1 in
  60:   def ADDU16 : R16_XYZ<0, "addu16", add>;
  61: let Pattern = [(set mGPR:$rz, (sub mGPR:$rx, mGPR:$ry))] in
  62:   def SUBU16 : R16_XYZ<1, "subu16", sub>;
  63: 
  64: let isCommutable = 1, isAdd = 1 in
  65:   def ADDC16 : R16_XZ_BINOP_C<0b1000, 0b01, "addc16">;
  66: def SUBC16 : R16_XZ_BINOP_C<0b1000, 0b11, "subc16">;
  67: 
  68: let isCommutable = 1 in {
  69:   let isAdd = 1 in
  70:   def ADDU16XZ : R16_XZ_BINOP<0b1000, 0b00, "addu16", BinOpFrag<(add node:$LHS, node:$RHS)>>;
  71:   def AND16 : R16_XZ_BINOP<0b1010, 0b00, "and16", BinOpFrag<(and node:$LHS, node:$RHS)>>;
  72:   def OR16 : R16_XZ_BINOP<0b1011, 0b00, "or16", BinOpFrag<(or node:$LHS, node:$RHS)>>;
```

- EN: This range defines declarative TableGen records such as ADDU16, SUBU16, ADDC16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 ADDU16, SUBU16, ADDC16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 73-90

```tablegen
  73:   def XOR16 : R16_XZ_BINOP<0b1011, 0b01, "xor16", BinOpFrag<(xor node:$LHS, node:$RHS)>>;
  74:   def NOR16 : R16_XZ_BINOP<0b1011, 0b10, "nor16", BinOpFrag<(not (or node:$LHS, node:$RHS))>>;
  75:   let isCodeGenOnly = 1 in
  76:   def NOT16 : R16_Z_UNOP<0b1011, 0b10, "not16">;
  77:   def MULT16 :  R16_XZ_BINOP<0b1111, 0b00, "mult16", BinOpFrag<(mul node:$LHS, node:$RHS)>>;
  78: }
  79: def SUBU16XZ : R16_XZ_BINOP<0b1000, 0b10, "subu16", BinOpFrag<(sub node:$LHS, node:$RHS)>>;
  80: def ANDN16 : R16_XZ_BINOP<0b1010, 0b01, "andn16", BinOpFrag<(and node:$LHS, (not node:$RHS))>>;
  81: def LSL16 : R16_XZ_BINOP<0b1100, 0b00, "lsl16", BinOpFrag<(shl node:$LHS, node:$RHS)>>;
  82: def LSR16 : R16_XZ_BINOP<0b1100, 0b01, "lsr16", BinOpFrag<(srl node:$LHS, node:$RHS)>>;
  83: def ASR16 : R16_XZ_BINOP<0b1100, 0b10, "asr16", BinOpFrag<(sra node:$LHS, node:$RHS)>>;
  84: def ROTL16 : R16_XZ_BINOP<0b1100, 0b11, "rotl16", BinOpFrag<(rotl node:$LHS, (and node:$RHS, 0x1f))>>;
  85: 
  86: def MULSH16 : R16_XZ_BINOP_NOPat<0b1111, 0b01, "mulsh16">;
  87: 
  88: def ZEXTB16 : R16_XZ_UNOP<0b1101, 0b00, "zextb16">;
  89: def ZEXTH16 : R16_XZ_UNOP<0b1101, 0b01, "zexth16">;
  90: def SEXTB16 : R16_XZ_UNOP<0b1101, 0b10, "sextb16">;
```

- EN: This range defines declarative TableGen records such as XOR16, NOR16, NOT16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 XOR16, NOR16, NOT16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 91-108

```tablegen
  91: def SEXTH16 : R16_XZ_UNOP<0b1101, 0b11, "sexth16">;
  92: 
  93: let Constraints = "$rZ = $rz", isReMaterializable = 1, isAsCheapAsAMove = 1 in {
  94:   let isAdd = 1, Pattern = [(set mGPR:$rz, (add mGPR:$rZ, oimm8:$imm8))] in
  95:   def ADDI16 : I16_Z_8<0b100, (ins mGPR:$rZ, oimm8:$imm8), "addi16\t$rz, $imm8">;
  96:   let Pattern = [(set mGPR:$rz, (sub mGPR:$rZ, oimm8:$imm8))] in
  97:   def SUBI16 : I16_Z_8<0b101, (ins mGPR:$rZ, oimm8:$imm8), "subi16\t$rz, $imm8">;
  98: }
  99: 
 100: def : Pat<(add GPR:$rs1, (oimm8_neg:$im)),
 101:           (SUBI16 GPR:$rs1, (imm_neg_XFORM oimm8_neg:$im))>;
 102: 
 103: let isAdd = 1 in
 104: def ADDI16ZSP : I16_Z_8<0b011, (ins GPRSP:$sp, uimm8_2:$imm8),
 105:                         "addi16\t$rz, $sp, $imm8"> {
 106:   bits<0> sp;
 107: }
 108: 
```

- EN: This range defines declarative TableGen records such as SEXTH16, ADDI16, SUBI16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 SEXTH16, ADDI16, SUBI16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 109-126

```tablegen
 109: let isAdd = 1 in
 110: def ADDI16SPSP : I16_SP_IMM7<0b000,"addi16">;
 111: def SUBI16SPSP : I16_SP_IMM7<0b001,"subi16">;
 112: 
 113: let isReMaterializable = 1, isAsCheapAsAMove = 1 in {
 114:   def LSLI16 : I16_XZ_IMM5<0, "lsli16", shl>;
 115:   def LSRI16 : I16_XZ_IMM5<1, "lsri16", srl>;
 116:   def ASRI16 : I16_XZ_IMM5<2, "asri16", sra>;
 117: }
 118: 
 119: let isAdd = 1 in
 120: def ADDI16XZ : I16_XZ_IMM3<0b10, "addi16", add>;
 121: def SUBI16XZ : I16_XZ_IMM3<0b11, "subi16", sub>;
 122: 
 123: let Size = 4 in
 124: def NEG16 : CSKYPseudo<(outs mGPR:$rd), (ins mGPR:$rx), "neg16 $rd, $rx", []>;
 125: 
 126: let Size = 4 in
```

- EN: This range defines declarative TableGen records such as ADDI16SPSP, SUBI16SPSP, LSLI16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ADDI16SPSP, SUBI16SPSP, LSLI16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 127-144

```tablegen
 127: def RSUBI16 : CSKYPseudo<(outs mGPR:$rd),
 128:   (ins mGPR:$rx, uimm8:$imm8), "rsubi16 $rd, $rx, $imm8", []>;
 129: 
 130: //===----------------------------------------------------------------------===//
 131: // Load & Store instructions.
 132: //===----------------------------------------------------------------------===//
 133: 
 134: def LD16B : I16_XZ_LDST<AddrMode16B, 0b000, "ld16.b",
 135:   (outs mGPR:$rz), (ins mGPR:$rx, uimm5:$imm)>;
 136: def LD16H : I16_XZ_LDST<AddrMode16H, 0b001, "ld16.h",
 137:   (outs mGPR:$rz), (ins mGPR:$rx, uimm5_1:$imm)>;
 138: def LD16W : I16_XZ_LDST<AddrMode16W, 0b010, "ld16.w",
 139:   (outs mGPR:$rz), (ins mGPR:$rx, uimm5_2:$imm)>;
 140: def ST16B : I16_XZ_LDST<AddrMode16B, 0b100, "st16.b",
 141:   (outs), (ins mGPR:$rz, mGPR:$rx, uimm5:$imm)>;
 142: def ST16H : I16_XZ_LDST<AddrMode16H, 0b101, "st16.h",
 143:   (outs), (ins mGPR:$rz, mGPR:$rx, uimm5_1:$imm)>;
 144: def ST16W : I16_XZ_LDST<AddrMode16W, 0b110, "st16.w",
```

- EN: This range defines declarative TableGen records such as RSUBI16, LD16B, LD16H, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 RSUBI16, LD16B, LD16H 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 145-162

```tablegen
 145:   (outs), (ins mGPR:$rz, mGPR:$rx, uimm5_2:$imm)>;
 146: 
 147: def LD16WSP : I16_ZSP_LDST<AddrMode16W, 0b011, "ld16.w", (outs mGPR:$rz),
 148:                            (ins GPRSP:$sp, uimm8_2:$addr)> {
 149:   bits<0> sp;
 150: }
 151: def ST16WSP : I16_ZSP_LDST<AddrMode16W, 0b111, "st16.w", (outs),
 152:                            (ins mGPR:$rz, GPRSP:$sp, uimm8_2:$addr)> {
 153:   bits<0> sp;
 154: }
 155: 
 156: //===----------------------------------------------------------------------===//
 157: // Compare instructions.
 158: //===----------------------------------------------------------------------===//
 159: 
 160: def CMPHS16 : R16_XY_CMP<0, "cmphs16">;
 161: def CMPLT16 : R16_XY_CMP<1, "cmplt16">;
 162: let isCommutable = 1 in
```

- EN: This range defines declarative TableGen records such as LD16WSP, ST16WSP, CMPHS16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LD16WSP, ST16WSP, CMPHS16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 163-180

```tablegen
 163: def CMPNE16 : R16_XY_CMP<2, "cmpne16">;
 164: 
 165: 
 166: def CMPHSI16 : I16_X_CMP<0, "cmphsi16", oimm5>;
 167: def CMPLTI16 : I16_X_CMP<1, "cmplti16", oimm5>;
 168: def CMPLEI16 : CSKYPseudo<(outs CARRY:$ca), (ins mGPR:$rx, uimm5:$imm5),
 169:     "cmplei16\t$rx, $imm5", []>;
 170: def CMPNEI16 : I16_X_CMP<2, "cmpnei16", uimm5>;
 171: 
 172: //===----------------------------------------------------------------------===//
 173: // Data move instructions.
 174: //===----------------------------------------------------------------------===//
 175: 
 176: 
 177: def MOVI16 : I16_Z_8<0b110, (ins uimm8:$imm8), "movi16\t$rz, $imm8"> {
 178:   let isReMaterializable = 1;
 179:   let isAsCheapAsAMove = 1;
 180:   let isMoveImm = 1;
```

- EN: This range defines declarative TableGen records such as CMPNE16, CMPHSI16, CMPLTI16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CMPNE16, CMPHSI16, CMPLTI16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 181-198

```tablegen
 181:   let Pattern = [(set mGPR:$rz, uimm8:$imm8)];
 182: }
 183: 
 184: def MOV16 : CSKY16Inst<AddrModeNone, (outs sGPR:$rz), (ins sGPR:$rx),
 185:                        "mov16\t$rz, $rx", []> {
 186:   bits<4> rz;
 187:   bits<4> rx;
 188:   let Inst{15,14} = 0b01;
 189:   let Inst{13 - 10} = 0b1011;
 190:   let Inst{9 - 6} = rz;
 191:   let Inst{5 - 2} = rx;
 192:   let Inst{1,0} = 0b11;
 193: }
 194: 
 195: // MVC16 is not in "cskyv2 instructions reference manul"
 196: def MVCV16 : CSKY16Inst<AddrModeNone, (outs sGPR:$rz), (ins CARRY:$ca),
 197:                         "mvcv16\t$rz", []> {
 198:   bits<0> ca;
```

- EN: This range defines declarative TableGen records such as MOV16, MVCV16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 MOV16, MVCV16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 199-216

```tablegen
 199:   bits<4> rz;
 200:   let Inst{15,14} = 0b01;
 201:   let Inst{13 - 10} = 0b1001;
 202:   let Inst{9 - 6} = rz;
 203:   let Inst{5 - 2} = 0;
 204:   let Inst{1,0} = 0b11;
 205: }
 206: 
 207: 
 208: //===----------------------------------------------------------------------===//
 209: // Branch and call instructions.
 210: //===----------------------------------------------------------------------===//
 211: 
 212: let isBranch = 1, isTerminator = 1 in {
 213:   let isBarrier = 1, isPredicable = 1 in
 214:     def BR16 : J16<1, "br16", (ins br_symbol_16bit:$offset)>;
 215: 
 216:   def BT16 : J16_B<2, "bt16">;
```

- EN: This range defines declarative TableGen records such as BR16, BT16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 BR16, BT16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 217-234

```tablegen
 217:   def BF16 : J16_B<3, "bf16">;
 218: }
 219: 
 220: def JMP16 : R16_X_J<0b11100000, 0b00, "jmp16"> {
 221:   let isBranch = 1;
 222:   let isTerminator = 1;
 223:   let isBarrier = 1;
 224:   let isIndirectBranch = 1;
 225:   let Pattern = [(brind sGPR:$rx)];
 226: }
 227: 
 228: def JSR16 : R16_X_J<0b11101111, 0b01, "jsr16"> {
 229:   let isCall = 1;
 230:   let Defs = [ R15 ];
 231: }
 232: 
 233: def RTS16 : CSKY16Inst<AddrModeNone, (outs), (ins), "rts16", [(CSKY_RET)]> {
 234:   let isTerminator = 1;
```

- EN: This range defines declarative TableGen records such as BF16, JMP16, JSR16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 BF16, JMP16, JSR16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 235-252

```tablegen
 235:   let isReturn = 1;
 236:   let isBarrier = 1;
 237:   let Inst = 0b0111100000111100;
 238:   let Uses = [R15];
 239:   let isCodeGenOnly = 1;
 240: }
 241: 
 242: def JMPIX16 :  CSKY16Inst<AddrModeNone, (outs),
 243:   (ins mGPR:$rx, uimm2_jmpix:$indeximm2), "jmpix16\t$rx, $indeximm2", []> {
 244:   bits<3> rx;
 245:   bits<2> indeximm2;
 246:   let Inst{15,14} = 0b00;
 247:   let Inst{13 - 11} = 0b111;
 248:   let Inst{10 - 8} = rx;
 249:   let Inst{7 - 2} = 0b111000;
 250:   let Inst{1,0} = indeximm2;
 251:   let Predicates = [HasJAVA];
 252:   let Uses = [R30];
```

- EN: This range defines declarative TableGen records such as JMPIX16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 JMPIX16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 253-270

```tablegen
 253: }
 254: 
 255: //===----------------------------------------------------------------------===//
 256: // Symbol address instructions.
 257: //===----------------------------------------------------------------------===//
 258: 
 259: def LRW16 : CSKY16Inst<AddrModeNone, (outs mGPR:$rz),
 260:   (ins constpool_symbol_16bit:$label), "lrw16\t$rz, $label", []> {
 261:   bits<3> rz;
 262:   bits<8> label;
 263:   let Inst{15 - 13} = 0b000;
 264:   let Inst{12} = label{7};
 265:   let Inst{11,10} = 0b00;
 266:   let Inst{9,8} = label{6,5};
 267:   let Inst{7 - 5} = rz;
 268:   let Inst{4 - 0} = label{4-0};
 269:   let mayLoad = 1;
 270:   let mayStore = 0;
```

- EN: This range defines declarative TableGen records such as LRW16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LRW16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 271-288

```tablegen
 271: }
 272: 
 273: def LRW16_Gen : CSKY16Inst<AddrModeNone, (outs mGPR:$rz),
 274:   (ins bare_symbol:$src, constpool_symbol_16bit:$label),
 275:   "lrw16\t$rz, $label", []> {
 276:   bits<3> rz;
 277:   bits<8> label;
 278:   let Inst{15 - 13} = 0b000;
 279:   let Inst{12} = label{7};
 280:   let Inst{11,10} = 0b00;
 281:   let Inst{9,8} = label{6,5};
 282:   let Inst{7 - 5} = rz;
 283:   let Inst{4 - 0} = label{4-0};
 284:   let mayLoad = 1;
 285:   let mayStore = 0;
 286:   let isCodeGenOnly = 1;
 287: }
 288: 
```

- EN: This range defines declarative TableGen records such as LRW16_Gen, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LRW16_Gen 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 289-306

```tablegen
 289: 
 290: //===----------------------------------------------------------------------===//
 291: // Other operation instructions.
 292: //===----------------------------------------------------------------------===//
 293: 
 294: def REVB16 :  R16_XZ_UNOP<0b1110, 0b10, "revb16">;
 295: def REVH16 :  R16_XZ_UNOP<0b1110, 0b11, "revh16">;
 296: 
 297: let isCodeGenOnly = 1 in
 298: def SETC16 : CSKY16Inst<AddrModeNone,
 299:   (outs CARRY:$ca), (ins), "setc16", []> {
 300:   let Inst{15, 14} = 0b01;
 301:   let Inst{13 - 10} = 0b1001;
 302:   let Inst{9 - 6} = 0;
 303:   let Inst{5 - 2} = 0;
 304:   let Inst{1, 0} = 0;
 305:   let isCompare = 1;
 306: }
```

- EN: This range defines declarative TableGen records such as REVB16, REVH16, SETC16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 REVB16, REVH16, SETC16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 307-324

```tablegen
 307: 
 308: let isCodeGenOnly = 1 in
 309: def CLRC16 : CSKY16Inst<AddrModeNone,
 310:   (outs CARRY:$ca), (ins), "clrc16", []> {
 311:   let Inst{15, 14} = 0b01;
 312:   let Inst{13 - 10} = 0b1001;
 313:   let Inst{9 - 6} = 0;
 314:   let Inst{5 - 2} = 0;
 315:   let Inst{1, 0} = 2;
 316:   let isCompare = 1;
 317: }
 318: 
 319: let Constraints = "$rZ = $rz" in {
 320:   def BCLRI16 : I16_Z_5<0b100, (outs mGPR:$rz), (ins mGPR:$rZ, uimm5:$imm5),
 321:                         "bclri16">;
 322:   def BSETI16 : I16_Z_5<0b101, (outs mGPR:$rz), (ins mGPR:$rZ, uimm5:$imm5),
 323:                         "bseti16">;
 324: }
```

- EN: This range defines declarative TableGen records such as CLRC16, BCLRI16, BSETI16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CLRC16, BCLRI16, BSETI16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 325-342

```tablegen
 325: 
 326: let Predicates = [HasBTST16] in
 327: def BTSTI16 : I16_Z_5<0b110, (outs CARRY:$ca), (ins mGPR:$rz, uimm5:$imm5),
 328:                       "btsti16"> {
 329:   bits<0> ca;
 330: }
 331: 
 332: def TST16 : CSKY16Inst<AddrModeNone, (outs CARRY:$ca), (ins sGPR:$rx, sGPR:$ry),
 333:                        "tst16\t$rx, $ry", []> {
 334:   bits<0> ca;
 335:   bits<4> ry;
 336:   bits<4> rx;
 337:   let Inst{15,14} = 0b01;
 338:   let Inst{13 - 10} = 0b1010;
 339:   let Inst{9 - 6} = ry;
 340:   let Inst{5 - 2} = rx;
 341:   let Inst{1,0} = 0b10;
 342:   let isCompare = 1;
```

- EN: This range defines declarative TableGen records such as BTSTI16, TST16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 BTSTI16, TST16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 343-360

```tablegen
 343: }
 344: 
 345: def TSTNBZ16 : CSKY16Inst<AddrModeNone, (outs CARRY:$ca), (ins sGPR:$rx),
 346:                           "tstnbz16\t$rx", []> {
 347:   bits<0> ca;
 348:   bits<4> rx;
 349:   let Inst{15,14} = 0b01;
 350:   let Inst{13 - 10} = 0b1010;
 351:   let Inst{9 - 6} = 0b0000;
 352:   let Inst{5 - 2} = rx;
 353:   let Inst{1,0} = 0b11;
 354:   let isCompare = 1;
 355: }
 356: 
 357: //===----------------------------------------------------------------------===//
 358: // Special instructions.
 359: //===----------------------------------------------------------------------===//
 360: 
```

- EN: This range defines declarative TableGen records such as TSTNBZ16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 TSTNBZ16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 361-378

```tablegen
 361: def BKPT : CSKY16Inst<AddrModeNone, (outs), (ins), "bkpt", []> {
 362:   let Inst = 0;
 363: }
 364: 
 365: let mayStore = 1 in {
 366: def BPUSHH : I16_BPushPop<0b00010100111, 0, (outs), (ins mGPR:$rz), "bpush.h $rz">;
 367: def BPUSHW : I16_BPushPop<0b00010100111, 0b10, (outs), (ins mGPR:$rz), "bpush.w $rz">;
 368: }
 369: 
 370: let mayLoad = 1 in {
 371: def BPOPH : I16_BPushPop<0b00010100101, 0, (outs mGPR:$rz), (ins),  "bpop.h $rz">;
 372: def BPOPW : I16_BPushPop<0b00010100101, 0b10, (outs mGPR:$rz), (ins), "bpop.w $rz">;
 373: }
 374: 
 375: def NIE : CSKY16Inst<AddrModeNone, (outs), (ins), "nie", [(CSKY_NIE)]> {
 376:   let Inst = 0b0001010001100000;
 377: }
 378: 
```

- EN: This range defines declarative TableGen records such as BKPT, BPUSHH, BPUSHW, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 BKPT, BPUSHH, BPUSHW 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 379-396

```tablegen
 379: let isBarrier = 1, isReturn = 1, isTerminator = 1 in
 380: def NIR : CSKY16Inst<AddrModeNone, (outs), (ins), "nir", [(CSKY_NIR)]> {
 381:   let Inst = 0b0001010001100001;
 382: }
 383: 
 384: def IPUSH16 : CSKY16Inst<AddrModeNone, (outs), (ins), "ipush16", []> {
 385:   let Inst{15- 5} = 0b00010100011;
 386:   let Inst{4-0} = 0b00010;
 387:   let Predicates = [iHasE1];
 388:   let Defs  = [R14];
 389:   let Uses  = [R14, R0, R1, R2, R3, R12, R13];
 390:   let mayStore = 1;
 391: }
 392: 
 393: def IPOP16 : CSKY16Inst<AddrModeNone, (outs), (ins), "ipop16", []> {
 394:   let Inst{15- 5} = 0b00010100011;
 395:   let Inst{4-0} = 0b00011;
 396:   let Predicates = [iHasE1];
```

- EN: This range defines declarative TableGen records such as NIR, IPUSH16, IPOP16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 NIR, IPUSH16, IPOP16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 397-414

```tablegen
 397:   let Defs  = [R14, R0, R1, R2, R3, R12, R13];
 398:   let Uses  = [R14];
 399:   let mayLoad = 1;
 400: }
 401: 
 402: def PUSH16 : CSKY16Inst<AddrModeNone, (outs),
 403:   (ins reglist:$regs, variable_ops), "push16 $regs", []> {
 404:   bits<5> regs;
 405: 
 406:   let Inst{15- 5} = 0b00010100110;
 407:   let Inst{4-0} = regs;
 408:   let Predicates = [iHasE1];
 409:   let Defs  = [R14];
 410:   let Uses  = [R14];
 411:   let mayStore = 1;
 412: }
 413: 
 414: def POP16 : CSKY16Inst<AddrModeNone, (outs),
```

- EN: This range defines declarative TableGen records such as PUSH16, POP16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 PUSH16, POP16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 415-432

```tablegen
 415:   (ins reglist:$regs, variable_ops), "pop16 $regs", []> {
 416:   bits<5> regs;
 417: 
 418:   let Inst{15- 5} = 0b00010100100;
 419:   let Inst{4-0} = regs;
 420:   let Predicates = [iHasE1];
 421:   let Defs  = [R14];
 422:   let Uses  = [R14];
 423:   let mayLoad = 1;
 424: }
 425: 
 426: //===----------------------------------------------------------------------===//
 427: // CSKYPseudo
 428: //===----------------------------------------------------------------------===//
 429: 
 430: let usesCustomInserter = 1 in  {
 431:   def ISEL16 : CSKYPseudo<(outs sGPR:$dst),
 432:     (ins CARRY:$cond, sGPR:$src1, sGPR:$src2),
```

- EN: This range defines declarative TableGen records such as ISEL16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ISEL16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 433-450

```tablegen
 433:     "!isel16\t$dst, $src1, src2",
 434:     [(set sGPR:$dst, (select CARRY:$cond, sGPR:$src1, sGPR:$src2))]>;
 435: }
 436: 
 437: class JBranchPseudo<dag out, dag ins, string opstr> :
 438:   CSKYPseudo<out, ins, opstr, []> {
 439:   let isBranch = 1;
 440:   let isTerminator = 1;
 441:   let isIndirectBranch = 1;
 442:   let mayLoad = 1;
 443:   let Size = 2;
 444: }
 445: 
 446: let isBarrier = 1 in
 447: def JBR16 : JBranchPseudo<(outs),
 448:   (ins br_symbol_16bit:$src1), "jbr16\t$src1">;
 449: def JBT16 : JBranchPseudo<(outs),
 450:   (ins CARRY:$ca, br_symbol_16bit:$src1), "jbt16\t$src1">;
```

- EN: This range defines declarative TableGen records such as JBranchPseudo, JBR16, JBT16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 JBranchPseudo, JBR16, JBT16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 451-468

```tablegen
 451: def JBF16 : JBranchPseudo<(outs),
 452:   (ins CARRY:$ca, br_symbol_16bit:$src1), "jbf16\t$src1">;
 453: 
 454: let mayLoad = 1, Size = 2, isCodeGenOnly = 0 in
 455: def PseudoLRW16 : CSKYPseudo<(outs mGPR:$rz),
 456:   (ins bare_symbol:$src), "lrw16 $rz, $src", []>;
 457: 
 458: //===----------------------------------------------------------------------===//
 459: // Instruction Patterns.
 460: //===----------------------------------------------------------------------===//
 461: 
 462: def : Pat<(sext_inreg mGPR:$src, i1), (ASRI16 (LSLI16 mGPR:$src, 7), 7)>;
 463: def : Pat<(sext_inreg sGPR:$src, i8), (SEXTB16 sGPR:$src)>;
 464: def : Pat<(sext_inreg sGPR:$src, i16), (SEXTH16 sGPR:$src)>;
 465: 
 466: // Load & Store Patterns
 467: 
 468: defm : LdPat<extloadi8, uimm5, LD16B, i32>;
```

- EN: This range defines declarative TableGen records such as JBF16, PseudoLRW16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 JBF16, PseudoLRW16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 469-486

```tablegen
 469: defm : LdPat<zextloadi8, uimm5, LD16B, i32>;
 470: 
 471: defm : LdPat<extloadi16, uimm5_1, LD16H, i32>;
 472: defm : LdPat<zextloadi16, uimm5_1, LD16H, i32>;
 473: 
 474: defm : LdPat<load, uimm5_2, LD16W, i32>;
 475: 
 476: 
 477: defm : StPat<truncstorei8, i32, uimm5, ST16B>;
 478: defm : StPat<truncstorei16, i32, uimm5_1, ST16H>;
 479: defm : StPat<store, i32, uimm5_2, ST16W>;
 480: 
 481: def : Pat<(CSKY_CALLReg sGPR:$src), (JSR16 sGPR:$src)>;
 482: def : Pat<(CSKY_TAILReg sGPR:$src), (JMP16 sGPR:$src)>;
 483: 
 484: // Symbol address Patterns
 485: def : Pat<(CSKY_LOAD_ADDR tglobaladdr, tconstpool:$src2), (LRW16 tconstpool:$src2)>;
 486: def : Pat<(CSKY_LOAD_ADDR tblockaddress, tconstpool:$src2), (LRW16 tconstpool:$src2)>;
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 487-504

```tablegen
 487: def : Pat<(CSKY_LOAD_ADDR tjumptable:$src1, tconstpool:$src2), (LRW16_Gen tjumptable:$src1, tconstpool:$src2)>;
 488: def : Pat<(CSKY_LOAD_ADDR texternalsym, tconstpool:$src2), (LRW16 tconstpool:$src2)>;
 489: def : Pat<(CSKY_LOAD_ADDR tconstpool:$src1, tconstpool:$src2), (LRW16_Gen tconstpool:$src1, tconstpool:$src2)>;
 490: 
 491: def : Pat<(i32 (load constpool:$src)), (LRW16 (to_tconstpool tconstpool:$src))>;
 492: 
 493: // Branch Patterns.
 494: 
 495: def : Pat<(brcond CARRY:$ca, bb:$offset),
 496:           (BT16 CARRY:$ca, bb:$offset)>;
 497: 
 498: def : Pat<(br bb:$offset), (BR16 bb:$offset)>;
 499: 
 500: multiclass BTF16Pat0<PatFrag cond0, PatFrag cond1, ImmLeaf imm_ty, Instruction inst> {
 501:   def : Pat<(brcond (i32 (cond0 mGPR:$rs1, imm_ty:$rs2)), bb:$offset),
 502:             (BT16 (inst mGPR:$rs1, imm_ty:$rs2), bb:$offset)>;
 503:   def : Pat<(brcond (i32 (cond1 mGPR:$rs1, imm_ty:$rs2)), bb:$offset),
 504:             (BF16 (inst mGPR:$rs1, imm_ty:$rs2), bb:$offset)>;
```

- EN: This range defines declarative TableGen records such as BTF16Pat0, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 BTF16Pat0 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 505-522

```tablegen
 505: }
 506: 
 507: defm : BTF16Pat0<setne, seteq, uimm5, CMPNEI16>;
 508: defm : BTF16Pat0<setuge, setult, oimm5, CMPHSI16>;
 509: defm : BTF16Pat0<setlt, setge, oimm5, CMPLTI16>;
 510: 
 511: def : Pat<(brcond (i32 (setne sGPR:$rs1, sGPR:$rs2)), bb:$offset),
 512:           (BT16 (CMPNE16 sGPR:$rs1, sGPR:$rs2), bb:$offset)>;
 513: def : Pat<(brcond (i32 (seteq sGPR:$rs1, sGPR:$rs2)), bb:$offset),
 514:           (BF16 (CMPNE16 sGPR:$rs1, sGPR:$rs2), bb:$offset)>;
 515: 
 516: multiclass BTF16Pat1<PatFrag cond0, PatFrag cond1, Instruction cmp,
 517:                      Instruction br>  {
 518:   def : Pat<(brcond (i32 (cond0 sGPR:$rs1, sGPR:$rs2)), bb:$offset),
 519:             (br (cmp sGPR:$rs1, sGPR:$rs2), bb:$offset)>;
 520:   def : Pat<(brcond (i32 (cond1 sGPR:$rs1, sGPR:$rs2)), bb:$offset),
 521:             (br (cmp sGPR:$rs2, sGPR:$rs1), bb:$offset)>;
 522: }
```

- EN: This range defines declarative TableGen records such as BTF16Pat1, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 BTF16Pat1 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 523-540

```tablegen
 523: 
 524: defm : BTF16Pat1<setuge, setule, CMPHS16, BT16>;
 525: defm : BTF16Pat1<setult, setugt, CMPHS16, BF16>;
 526: defm : BTF16Pat1<setlt, setgt, CMPLT16, BT16>;
 527: defm : BTF16Pat1<setge, setle, CMPLT16, BF16>;
 528: 
 529: // Compare Patterns.
 530: def : Pat<(setne sGPR:$rs1, sGPR:$rs2),
 531:           (SUBU16XZ (MOVI16 1), (MVCV16 (CMPNE16 sGPR:$rs1, sGPR:$rs2)))>;
 532: def : Pat<(seteq sGPR:$rs1, sGPR:$rs2),
 533:           (MVCV16 (CMPNE16 sGPR:$rs1, sGPR:$rs2))>;
 534: def : Pat<(setuge sGPR:$rs1, sGPR:$rs2),
 535:           (SUBU16XZ (MOVI16 1), (MVCV16 (CMPHS16 sGPR:$rs1, sGPR:$rs2)))>;
 536: def : Pat<(setule sGPR:$rs1, sGPR:$rs2),
 537:           (SUBU16XZ (MOVI16 1), (MVCV16 (CMPHS16 sGPR:$rs2, sGPR:$rs1)))>;
 538: def : Pat<(setult sGPR:$rs1, sGPR:$rs2),
 539:           (MVCV16 (CMPHS16 sGPR:$rs1, sGPR:$rs2))>;
 540: def : Pat<(setugt sGPR:$rs1, sGPR:$rs2),
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 541-558

```tablegen
 541:           (MVCV16 (CMPHS16 sGPR:$rs2, sGPR:$rs1))>;
 542: def : Pat<(setlt sGPR:$rs1, sGPR:$rs2),
 543:           (SUBU16XZ (MOVI16 1), (MVCV16 (CMPLT16 sGPR:$rs1, sGPR:$rs2)))>;
 544: def : Pat<(setgt sGPR:$rs1, sGPR:$rs2),
 545:           (SUBU16XZ (MOVI16 1), (MVCV16 (CMPLT16 sGPR:$rs2, sGPR:$rs1)))>;
 546: def : Pat<(setge sGPR:$rs1, sGPR:$rs2),
 547:           (MVCV16 (CMPLT16 sGPR:$rs1, sGPR:$rs2))>;
 548: def : Pat<(setle sGPR:$rs1, sGPR:$rs2),
 549:           (MVCV16 (CMPLT16 sGPR:$rs2, sGPR:$rs1))>;
 550: 
 551: 
 552: def : Pat<(setne mGPR:$rs1, uimm5:$rs2),
 553:           (SUBU16XZ (MOVI16 1), (MVCV16 (CMPNEI16 mGPR:$rs1, uimm5:$rs2)))>;
 554: def : Pat<(seteq mGPR:$rs1, uimm5:$rs2),
 555:           (MVCV16 (CMPNEI16 mGPR:$rs1, uimm5:$rs2))>;
 556: def : Pat<(setuge mGPR:$rs1, oimm5:$rs2),
 557:           (SUBU16XZ (MOVI16 1), (MVCV16 (CMPHSI16 mGPR:$rs1, oimm5:$rs2)))>;
 558: def : Pat<(setult mGPR:$rs1, oimm5:$rs2),
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 559-576

```tablegen
 559:           (MVCV16 (CMPHSI16 mGPR:$rs1, oimm5:$rs2))>;
 560: def : Pat<(setlt mGPR:$rs1, oimm5:$rs2),
 561:           (SUBU16XZ (MOVI16 1), (MVCV16 (CMPLTI16 mGPR:$rs1, oimm5:$rs2)))>;
 562: def : Pat<(setge mGPR:$rs1, oimm5:$rs2),
 563:           (MVCV16 (CMPLTI16 mGPR:$rs1, oimm5:$rs2))>;
 564: 
 565: def : Pat<(select CARRY:$ca, sGPR:$rx, sGPR:$false),
 566:           (ISEL16 CARRY:$ca, sGPR:$rx, sGPR:$false)>;
 567: def : Pat<(select (and CARRY:$ca, 1), sGPR:$rx, sGPR:$false),
 568:           (ISEL16 CARRY:$ca, sGPR:$rx, sGPR:$false)>;
 569: 
 570: def : Pat<(rotl sGPR:$rs1, sGPR:$rs2),
 571:           (ROTL16 sGPR:$rs1, (AND16 sGPR:$rs2, (MOVI16 0x1f)))>;
 572: 
 573: 
 574: // FIXME: This is a temporary treatment for the e801.
 575: def : Pat<(i32 imm:$imm),
 576:           (OR16 (MOVI16 (uimm8SRL_0 imm:$imm)),
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 577-594

```tablegen
 577: 	              (OR16 (LSLI16 (MOVI16 (uimm8SRL_8 imm:$imm)), 8),
 578: 	                    (OR16 (LSLI16 (MOVI16 (uimm8SRL_16 imm:$imm)), 16),
 579:                             (LSLI16 (MOVI16 (uimm8SRL_24 imm:$imm)), 24))))>;
 580: 
 581: // Other operations.
 582: let Predicates = [iHasE2] in {
 583:   def : Pat<(bswap sGPR:$rx), (REVB16 sGPR:$rx)>;
 584: }
 585: 
 586: //===----------------------------------------------------------------------===//
 587: // Compress Instruction tablegen backend.
 588: //===----------------------------------------------------------------------===//
 589: 
 590: def : CompressPat<(ADDU32 sGPR:$rd, sGPR:$rd, sGPR:$rs2),
 591:                   (ADDU16XZ sGPR:$rd, sGPR:$rs2)>;
 592: def : CompressPat<(ADDU32 sGPR:$rd, sGPR:$rs1, sGPR:$rd),
 593:                   (ADDU16XZ sGPR:$rd, sGPR:$rs1)>;
 594: def : CompressPat<(ADDU32 mGPR:$rd, mGPR:$rs1, mGPR:$rs2),
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 595-612

```tablegen
 595:                   (ADDU16 mGPR:$rd, mGPR:$rs1, mGPR:$rs2)>;
 596: def : CompressPat<(SUBU32 sGPR:$rd, sGPR:$rd, sGPR:$rs2),
 597:                   (SUBU16XZ sGPR:$rd, sGPR:$rs2)>;
 598: def : CompressPat<(SUBU32 mGPR:$rd, mGPR:$rs1, mGPR:$rs2),
 599:                   (SUBU16 mGPR:$rd, mGPR:$rs1, mGPR:$rs2)>;
 600: 
 601: def : CompressPat<
 602:   (ADDC32 sGPR:$rd, CARRY:$cout, sGPR:$rd, sGPR:$rs2, CARRY:$cout),
 603:   (ADDC16 sGPR:$rd, CARRY:$cout, sGPR:$rs2, CARRY:$cout)
 604:   >;
 605: def : CompressPat<
 606:   (SUBC32 sGPR:$rd, CARRY:$cout, sGPR:$rd, sGPR:$rs2, CARRY:$cout),
 607:   (SUBC16 sGPR:$rd, CARRY:$cout, sGPR:$rs2, CARRY:$cout)
 608:   >;
 609: 
 610: def : CompressPat<(ADDI32 mGPR:$rd, mGPR:$rs, oimm3:$imm),
 611:                   (ADDI16XZ mGPR:$rd, mGPR:$rs, oimm3:$imm)>;
 612: def : CompressPat<(SUBI32 mGPR:$rd, mGPR:$rs, oimm3:$imm),
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 613-630

```tablegen
 613:                   (SUBI16XZ mGPR:$rd, mGPR:$rs, oimm3:$imm)>;
 614: 
 615: def : CompressPat<(ADDI32 mGPR:$rd, mGPR:$rd, oimm8:$imm),
 616:                   (ADDI16 mGPR:$rd, oimm8:$imm)>;
 617: def : CompressPat<(SUBI32 mGPR:$rd, mGPR:$rd, oimm8:$imm),
 618:                   (SUBI16 mGPR:$rd, oimm8:$imm)>;
 619: 
 620: def : CompressPat<(ADDI32 GPRSP:$sp, GPRSP:$sp, uimm7_2:$imm),
 621:                   (ADDI16SPSP GPRSP:$sp, GPRSP:$sp, uimm7_2:$imm)>;
 622: def : CompressPat<(SUBI32 GPRSP:$sp, GPRSP:$sp, uimm7_2:$imm),
 623:                   (SUBI16SPSP GPRSP:$sp, GPRSP:$sp, uimm7_2:$imm)>;
 624: 
 625: def : CompressPat<(ADDI32 mGPR:$rd, GPRSP:$sp, uimm8_2:$imm),
 626:                   (ADDI16ZSP mGPR:$rd, GPRSP:$sp, uimm8_2:$imm)>;
 627: 
 628: def : CompressPat<(MULT32 sGPR:$rd, sGPR:$rd, sGPR:$rs2),
 629:                   (MULT16 sGPR:$rd, sGPR:$rs2)>;
 630: def : CompressPat<(MULT32 sGPR:$rd, sGPR:$rs1, sGPR:$rd),
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 631-648

```tablegen
 631:                   (MULT16 sGPR:$rd, sGPR:$rs1)>;
 632: def : CompressPat<(AND32 sGPR:$rd, sGPR:$rd, sGPR:$rs2),
 633:                   (AND16 sGPR:$rd, sGPR:$rs2)>;
 634: def : CompressPat<(AND32 sGPR:$rd, sGPR:$rs1, sGPR:$rd),
 635:                   (AND16 sGPR:$rd, sGPR:$rs1)>;
 636: def : CompressPat<(OR32 sGPR:$rd, sGPR:$rd, sGPR:$rs2),
 637:                   (OR16 sGPR:$rd, sGPR:$rs2)>;
 638: def : CompressPat<(OR32 sGPR:$rd, sGPR:$rs1, sGPR:$rd),
 639:                   (OR16 sGPR:$rd, sGPR:$rs1)>;
 640: def : CompressPat<(XOR32 sGPR:$rd, sGPR:$rd, sGPR:$rs2),
 641:                   (XOR16 sGPR:$rd, sGPR:$rs2)>;
 642: def : CompressPat<(XOR32 sGPR:$rd, sGPR:$rs1, sGPR:$rd),
 643:                   (XOR16 sGPR:$rd, sGPR:$rs1)>;
 644: 
 645: def : CompressPat<(ANDN32 sGPR:$rd, sGPR:$rd, sGPR:$rs2),
 646:                   (ANDN16 sGPR:$rd, sGPR:$rs2)>;
 647: def : CompressPat<(NOR32 sGPR:$rd, sGPR:$rd, sGPR:$rs2),
 648:                   (NOR16 sGPR:$rd, sGPR:$rs2)>;
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 649-666

```tablegen
 649: def : CompressPat<(LSL32 sGPR:$rd, sGPR:$rd, sGPR:$rs2),
 650:                   (LSL16 sGPR:$rd, sGPR:$rs2)>;
 651: def : CompressPat<(LSR32 sGPR:$rd, sGPR:$rd, sGPR:$rs2),
 652:                   (LSR16 sGPR:$rd, sGPR:$rs2)>;
 653: def : CompressPat<(ASR32 sGPR:$rd, sGPR:$rd, sGPR:$rs2),
 654:                   (ASR16 sGPR:$rd, sGPR:$rs2)>;
 655: def : CompressPat<(ROTL32 sGPR:$rd, sGPR:$rd, sGPR:$rs2),
 656:                   (ROTL16 sGPR:$rd, sGPR:$rs2)>;
 657: 
 658: def : CompressPat<(NOT32 sGPR:$rd, sGPR:$rd),
 659:                   (NOT16 sGPR:$rd)>;
 660: 
 661: let Predicates = [iHas2E3] in
 662: def : CompressPat<(REVB32 sGPR:$rd, sGPR:$rs),
 663:                   (REVB16 sGPR:$rd, sGPR:$rs)>;
 664: 
 665: def : CompressPat<(LSLI32 mGPR:$rd, mGPR:$rs, uimm5:$imm),
 666:                   (LSLI16 mGPR:$rd, mGPR:$rs, uimm5:$imm)>;
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 667-684

```tablegen
 667: def : CompressPat<(LSRI32 mGPR:$rd, mGPR:$rs, uimm5:$imm),
 668:                   (LSRI16 mGPR:$rd, mGPR:$rs, uimm5:$imm)>;
 669: def : CompressPat<(ASRI32 mGPR:$rd, mGPR:$rs, uimm5:$imm),
 670:                   (ASRI16 mGPR:$rd, mGPR:$rs, uimm5:$imm)>;
 671: 
 672: def : CompressPat<(CMPHS32 CARRY:$ca, sGPR:$rs1, sGPR:$rs2),
 673:                   (CMPHS16 CARRY:$ca, sGPR:$rs1, sGPR:$rs2)>;
 674: def : CompressPat<(CMPLT32 CARRY:$ca, sGPR:$rs1, sGPR:$rs2),
 675:                   (CMPLT16 CARRY:$ca, sGPR:$rs1, sGPR:$rs2)>;
 676: def : CompressPat<(CMPNE32 CARRY:$ca, sGPR:$rs1, sGPR:$rs2),
 677:                   (CMPNE16 CARRY:$ca, sGPR:$rs1, sGPR:$rs2)>;
 678: 
 679: def : CompressPat<(CMPHSI32 CARRY:$ca, mGPR:$rs, oimm5:$imm),
 680:                   (CMPHSI16 CARRY:$ca, mGPR:$rs, oimm5:$imm)>;
 681: def : CompressPat<(CMPLTI32 CARRY:$ca, mGPR:$rs, oimm5:$imm),
 682:                   (CMPLTI16 CARRY:$ca, mGPR:$rs, oimm5:$imm)>;
 683: def : CompressPat<(CMPNEI32 CARRY:$ca, mGPR:$rs, uimm5:$imm),
 684:                   (CMPNEI16 CARRY:$ca, mGPR:$rs, uimm5:$imm)>;
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 685-702

```tablegen
 685: 
 686: def : CompressPat<(JSR32 sGPR:$rd),
 687:                   (JSR16 sGPR:$rd)>;
 688: 
 689: 
 690: def : CompressPat<(MVCV32 sGPR:$rd, CARRY:$ca),
 691:                   (MVCV16 sGPR:$rd, CARRY:$ca)>;
 692: def : CompressPat<(MOV32 sGPR:$rd, sGPR:$ca),
 693:                   (MOV16 sGPR:$rd, sGPR:$ca)>;
 694: def : CompressPat<(MOVI32 mGPR:$rd, uimm8:$imm),
 695:                   (MOVI16 mGPR:$rd, uimm8:$imm)>;
 696: 
 697: def : CompressPat<(LD32B mGPR:$rd, mGPR:$rs, uimm5:$imm),
 698:                   (LD16B mGPR:$rd, mGPR:$rs, uimm5:$imm)>;
 699: def : CompressPat<(LD32H mGPR:$rd, mGPR:$rs, uimm5_1:$imm),
 700:                   (LD16H mGPR:$rd, mGPR:$rs, uimm5_1:$imm)>;
 701: def : CompressPat<(LD32W mGPR:$rd, mGPR:$rs, uimm5_2:$imm),
 702:                   (LD16W mGPR:$rd, mGPR:$rs, uimm5_2:$imm)>;
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 703-720

```tablegen
 703: def : CompressPat<(LD32W mGPR:$rd, GPRSP:$sp, uimm8_2:$imm),
 704:                   (LD16WSP mGPR:$rd, GPRSP:$sp, uimm8_2:$imm)>;
 705: 
 706: def : CompressPat<(ST32B mGPR:$rd, mGPR:$rs, uimm5:$imm),
 707:                   (ST16B mGPR:$rd, mGPR:$rs, uimm5:$imm)>;
 708: def : CompressPat<(ST32H mGPR:$rd, mGPR:$rs, uimm5_1:$imm),
 709:                   (ST16H mGPR:$rd, mGPR:$rs, uimm5_1:$imm)>;
 710: def : CompressPat<(ST32W mGPR:$rd, mGPR:$rs, uimm5_2:$imm),
 711:                   (ST16W mGPR:$rd, mGPR:$rs, uimm5_2:$imm)>;
 712: def : CompressPat<(ST32W mGPR:$rd, GPRSP:$sp, uimm8_2:$imm),
 713:                   (ST16WSP mGPR:$rd, GPRSP:$sp, uimm8_2:$imm)>;
 714: 
 715: let Predicates = [HasBTST16] in
 716: def : CompressPat<(BTSTI32 CARRY:$ca, mGPR:$rs, uimm5:$imm),
 717:                   (BTSTI16 CARRY:$ca, mGPR:$rs, uimm5:$imm)>;
 718: def : CompressPat<(BCLRI32 mGPR:$rd, mGPR:$rd, uimm5:$imm),
 719:                   (BCLRI16 mGPR:$rd, uimm5:$imm)>;
 720: def : CompressPat<(BSETI32 mGPR:$rd, mGPR:$rd, uimm5:$imm),
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 721-730

```tablegen
 721:                   (BSETI16 mGPR:$rd, uimm5:$imm)>;
 722: 
 723: def : CompressPat<(ZEXTB32 sGPR:$rd, sGPR:$rs),
 724:                   (ZEXTB16 sGPR:$rd, sGPR:$rs)>;
 725: def : CompressPat<(ZEXTH32 sGPR:$rd, sGPR:$rs),
 726:                   (ZEXTH16 sGPR:$rd, sGPR:$rs)>;
 727: def : CompressPat<(SEXTB32 sGPR:$rd, sGPR:$rs),
 728:                   (SEXTB16 sGPR:$rd, sGPR:$rs)>;
 729: def : CompressPat<(SEXTH32 sGPR:$rd, sGPR:$rs),
 730:                   (SEXTH16 sGPR:$rd, sGPR:$rs)>;
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- TableGen instruction records / TableGen 指令记录
- Encoding and patterns / 编码与匹配模式
- Pseudo-instruction handling / 伪指令处理
- Pattern matching / 模式匹配

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKYInstrFormats16Instr.td`
- LLVM subsystems / LLVM 子系统: TableGen
