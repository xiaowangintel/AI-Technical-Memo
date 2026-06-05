# LanaiInstrInfo.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/Lanai/LanaiInstrInfo.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file describes the Lanai instructions in TableGen format.
- 目的（中文）: 使用 TableGen 定义目标指令，描述操作码、操作数、匹配模式与调度信息。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===-- LanaiInstrInfo.td - Target Description for Lanai Target -----------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file describes the Lanai instructions in TableGen format.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: //===----------------------------------------------------------------------===//
  14: // Instruction format superclass
  15: //===----------------------------------------------------------------------===//
  16: 
  17: include "LanaiInstrFormats.td"
  18: 
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. It imports other TableGen fragments so the current description can reuse shared records, register sets, or instruction definitions.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 它引入其他 TableGen 片段，以便当前描述复用共享记录、寄存器集合或指令定义。

### Lines 19-36

```tablegen
  19: // -------------------------------------------------- //
  20: // Instruction Operands and Patterns
  21: // -------------------------------------------------- //
  22: 
  23: //  These are target-independent nodes, but have target-specific formats.
  24: def SDT_LanaiCallSeqStart : SDCallSeqStart<[SDTCisVT<0, i32>,
  25:                                             SDTCisVT<1, i32>]>;
  26: def SDT_LanaiCallSeqEnd   : SDCallSeqEnd<[SDTCisVT<0, i32>,
  27:                                           SDTCisVT<1, i32>]>;
  28: def SDT_LanaiCall         : SDTypeProfile<0, -1, [SDTCisVT<0, i32>]>;
  29: def SDT_LanaiSetFlag      : SDTypeProfile<0,  2, [SDTCisSameAs<0, 1>]>;
  30: def SDT_LanaiSelectCC     : SDTypeProfile<1,  3, [SDTCisSameAs<0, 1>,
  31:                                                   SDTCisSameAs<1, 2>]>;
  32: def SDT_LanaiSetCC        : SDTypeProfile<1,  1, [SDTCisVT<0, i32>,
  33:                                                   SDTCisVT<1, i32>]>;
  34: def SDT_LanaiBrCC         : SDTypeProfile<0,  2, [SDTCisVT<0, OtherVT>,
  35:                                                   SDTCisVT<1, i32>]>;
  36: def SDT_LanaiAdjDynAlloc  : SDTypeProfile<1,  1, [SDTCisVT<0, i32>,
```

- EN: This range defines declarative TableGen records such as SDT_LanaiCallSeqStart, SDT_LanaiCallSeqEnd, SDT_LanaiCall, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 SDT_LanaiCallSeqStart, SDT_LanaiCallSeqEnd, SDT_LanaiCall 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 37-54

```tablegen
  37:                                                   SDTCisVT<1, i32>]>;
  38: 
  39: def Call             : SDNode<"LanaiISD::CALL", SDT_LanaiCall,
  40:                               [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
  41:                                SDNPVariadic]>;
  42: def RetGlue          : SDNode<"LanaiISD::RET_GLUE", SDTNone,
  43:                               [SDNPHasChain, SDNPOptInGlue, SDNPVariadic]>;
  44: def CallSeqStart     : SDNode<"ISD::CALLSEQ_START", SDT_LanaiCallSeqStart,
  45:                               [SDNPHasChain, SDNPOutGlue]>;
  46: def CallSeqEnd       : SDNode<"ISD::CALLSEQ_END", SDT_LanaiCallSeqEnd,
  47:                               [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue]>;
  48: def LanaiSetFlag     : SDNode<"LanaiISD::SET_FLAG", SDT_LanaiSetFlag,
  49:                               [SDNPOutGlue]>;
  50: def LanaiSubbF       : SDNode<"LanaiISD::SUBBF", SDT_LanaiSetFlag,
  51:                               [SDNPOutGlue, SDNPInGlue]>;
  52: def LanaiBrCC        : SDNode<"LanaiISD::BR_CC", SDT_LanaiBrCC,
  53:                               [SDNPHasChain, SDNPInGlue]>;
  54: def LanaiSelectCC    : SDNode<"LanaiISD::SELECT_CC", SDT_LanaiSelectCC,
```

- EN: This range defines declarative TableGen records such as Call, RetGlue, CallSeqStart, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 Call, RetGlue, CallSeqStart 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 55-72

```tablegen
  55:                               [SDNPInGlue]>;
  56: def LanaiSetCC       : SDNode<"LanaiISD::SETCC", SDT_LanaiSetCC,
  57:                               [SDNPInGlue]>;
  58: def LanaiHi          : SDNode<"LanaiISD::HI", SDTIntUnaryOp>;
  59: def LanaiLo          : SDNode<"LanaiISD::LO", SDTIntUnaryOp>;
  60: def LanaiSmall       : SDNode<"LanaiISD::SMALL", SDTIntUnaryOp>;
  61: def LanaiAdjDynAlloc : SDNode<"LanaiISD::ADJDYNALLOC", SDT_LanaiAdjDynAlloc>;
  62: 
  63: // Extract bits 0-15 (low-end) of an immediate value.
  64: def LO16 : SDNodeXForm<imm, [{
  65:   return CurDAG->getTargetConstant((uint64_t)N->getZExtValue() & 0xffff,
  66:                                    SDLoc(N), MVT::i32);
  67: }]>;
  68: 
  69: // Extract bits 16-31 (high-end) of an immediate value.
  70: // Transformation function: shift the immediate value down into the low bits.
  71: def HI16 : SDNodeXForm<imm, [{
  72:   return CurDAG->getTargetConstant((uint64_t)N->getZExtValue() >> 16, SDLoc(N),
```

- EN: This range defines declarative TableGen records such as LanaiSetCC, LanaiHi, LanaiLo, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 LanaiSetCC, LanaiHi, LanaiLo 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 73-90

```tablegen
  73:                                    MVT::i32);
  74: }]>;
  75: 
  76: def NEG : SDNodeXForm<imm, [{
  77:   return CurDAG->getSignedTargetConstant(-N->getSExtValue(), SDLoc(N),
  78:                                          MVT::i32);
  79: }]>;
  80: 
  81: def LO21 : SDNodeXForm<imm, [{
  82:   return CurDAG->getTargetConstant((uint64_t)N->getZExtValue() & 0x1fffff,
  83:                                    SDLoc(N), MVT::i32);
  84: }]>;
  85: 
  86: // Branch targets
  87: def BrTargetAsmOperand : AsmOperandClass {
  88:   let Name = "BrTarget";
  89: }
  90: def BrTarget   : Operand<OtherVT> {
```

- EN: This range defines declarative TableGen records such as NEG, LO21, BrTargetAsmOperand, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 NEG, LO21, BrTargetAsmOperand 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 91-108

```tablegen
  91:   let ParserMatchClass = BrTargetAsmOperand;
  92:   let EncoderMethod = "getBranchTargetOpValue";
  93:   let DecoderMethod = "decodeBranch";
  94: }
  95: 
  96: def CallTargetAsmOperand : AsmOperandClass {
  97:   let Name = "CallTarget";
  98: }
  99: def CallTarget : Operand<i32> {
 100:   let ParserMatchClass = CallTargetAsmOperand;
 101:   let EncoderMethod = "getBranchTargetOpValue";
 102:   let DecoderMethod = "decodeBranch";
 103: }
 104: 
 105: def ImmShiftAsmOperand : AsmOperandClass { let Name = "ImmShift"; }
 106: def immShift : Operand<i32>, PatLeaf<(imm), [{
 107:     int Imm = N->getSExtValue();
 108:     return Imm >= -31 && Imm <= 31;}]> {
```

- EN: This range defines declarative TableGen records such as CallTargetAsmOperand, CallTarget, ImmShiftAsmOperand, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CallTargetAsmOperand, CallTarget, ImmShiftAsmOperand 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 109-126

```tablegen
 109:   let ParserMatchClass = ImmShiftAsmOperand;
 110:   let DecoderMethod = "decodeShiftImm";
 111: }
 112: 
 113: def Imm10AsmOperand : AsmOperandClass { let Name = "Imm10"; }
 114: def imm10 : Operand<i32>, PatLeaf<(imm), [{
 115:     return isInt<10>(N->getSExtValue()); }]> {
 116:   let ParserMatchClass = Imm10AsmOperand;
 117: }
 118: 
 119: def LoImm16AsmOperand : AsmOperandClass { let Name = "LoImm16"; }
 120: def i32lo16z : Operand<i32>, PatLeaf<(i32 imm), [{
 121:     // i32lo16 predicate - true if the 32-bit immediate has only rightmost 16
 122:     // bits set.
 123:     return ((N->getZExtValue() & 0xFFFFUL) == N->getZExtValue());}], LO16> {
 124:   let ParserMatchClass = LoImm16AsmOperand;
 125: }
 126: def i32neg16 : Operand<i32>, PatLeaf<(i32 imm), [{
```

- EN: This range defines declarative TableGen records such as Imm10AsmOperand, imm10, LoImm16AsmOperand, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 Imm10AsmOperand, imm10, LoImm16AsmOperand 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 127-144

```tablegen
 127:     // i32neg16 predicate - true if the 32-bit immediate is negative and can
 128:     // be represented by a 16 bit integer.
 129:     int Imm = N->getSExtValue();
 130:     return (Imm < 0) && (isInt<16>(Imm));}], LO16> {
 131:   let ParserMatchClass = LoImm16AsmOperand;
 132: }
 133: def i32lo16s : Operand<i32>, PatLeaf<(i32 imm), [{
 134:     // i32lo16 predicate - true if the 32-bit immediate has only rightmost 16
 135:     // bits set.
 136:     return ((int64_t)(N->getSExtValue() & 0xFFFFUL) == N->getSExtValue());}], LO16> {
 137:   let ParserMatchClass = LoImm16AsmOperand;
 138: }
 139: 
 140: def LoImm16AndAsmOperand : AsmOperandClass { let Name = "LoImm16And"; }
 141: def i32lo16and : Operand<i32>, PatLeaf<(i32 imm), [{
 142:     // i32lo16 predicate - true if the 32-bit immediate has the rightmost 16
 143:     // bits set and the leftmost 16 bits 1's.
 144:     return (N->getZExtValue() >= 0xFFFF0000UL);}], LO16> {
```

- EN: This range defines declarative TableGen records such as i32lo16s, LoImm16AndAsmOperand, i32lo16and, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 i32lo16s, LoImm16AndAsmOperand, i32lo16and 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 145-162

```tablegen
 145:   let ParserMatchClass = LoImm16AndAsmOperand;
 146:   let PrintMethod = "printLo16AndImmOperand";
 147: }
 148: 
 149: def HiImm16AsmOperand : AsmOperandClass { let Name = "HiImm16"; }
 150: def i32hi16 : Operand<i32>, PatLeaf<(i32 imm), [{
 151:     // i32hi16 predicate - true if the 32-bit immediate has only leftmost 16
 152:     // bits set.
 153:     return ((N->getZExtValue() & 0xFFFF0000UL) == N->getZExtValue());}], HI16> {
 154:   let ParserMatchClass = HiImm16AsmOperand;
 155:   let PrintMethod = "printHi16ImmOperand";
 156: }
 157: 
 158: def HiImm16AndAsmOperand : AsmOperandClass { let Name = "HiImm16And"; }
 159: def i32hi16and : Operand<i32>, PatLeaf<(i32 imm), [{
 160:     // i32lo16 predicate - true if the 32-bit immediate has the leftmost 16
 161:     // bits set and the rightmost 16 bits 1's.
 162:     return ((N->getZExtValue() & 0xFFFFUL) == 0xFFFFUL);}], HI16> {
```

- EN: This range defines declarative TableGen records such as HiImm16AsmOperand, i32hi16, HiImm16AndAsmOperand, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 HiImm16AsmOperand, i32hi16, HiImm16AndAsmOperand 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 163-180

```tablegen
 163:   let ParserMatchClass = HiImm16AndAsmOperand;
 164:   let PrintMethod = "printHi16AndImmOperand";
 165: }
 166: 
 167: def LoImm21AsmOperand : AsmOperandClass { let Name = "LoImm21"; }
 168: def i32lo21 : Operand<i32>, PatLeaf<(i32 imm), [{
 169:     // i32lo21 predicate - true if the 32-bit immediate has only rightmost 21
 170:     // bits set.
 171:     return ((N->getZExtValue() & 0x1FFFFFUL) == N->getZExtValue());}], LO21> {
 172:   let ParserMatchClass = LoImm21AsmOperand;
 173: }
 174: 
 175: def AluOp : Operand<i32> {
 176:   let PrintMethod = "printAluOperand";
 177: }
 178: 
 179: // Addressing modes.
 180: def ADDRrr : ComplexPattern<i32, 3, "selectAddrRr", [], []>;
```

- EN: This range defines declarative TableGen records such as LoImm21AsmOperand, i32lo21, AluOp, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 LoImm21AsmOperand, i32lo21, AluOp 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 181-198

```tablegen
 181: def ADDRri : ComplexPattern<i32, 3, "selectAddrRi", [frameindex], []>;
 182: def ADDRsls : ComplexPattern<i32, 1, "selectAddrSls", [frameindex], []>;
 183: def ADDRspls : ComplexPattern<i32, 3, "selectAddrSpls", [frameindex], []>;
 184: 
 185: // Address operands
 186: def MemRegImmAsmOperand : AsmOperandClass {
 187:   let Name = "MemRegImm";
 188:   let ParserMethod  = "parseMemoryOperand";
 189: }
 190: def MEMri : Operand<i32> {
 191:   let DecoderMethod = "decodeRiMemoryValue";
 192:   let EncoderMethod = "getRiMemoryOpValue";
 193:   let MIOperandInfo = (ops GPR:$base, i32lo16s:$offset, AluOp:$Opcode);
 194:   let ParserMatchClass = MemRegImmAsmOperand;
 195:   let PrintMethod   = "printMemRiOperand";
 196: }
 197: 
 198: def MemRegRegAsmOperand : AsmOperandClass {
```

- EN: This range defines declarative TableGen records such as ADDRri, ADDRsls, ADDRspls, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 ADDRri, ADDRsls, ADDRspls 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 199-216

```tablegen
 199:   let Name = "MemRegReg";
 200:   let ParserMethod  = "parseMemoryOperand";
 201: }
 202: def MEMrr : Operand<i32> {
 203:   let DecoderMethod = "decodeRrMemoryValue";
 204:   let EncoderMethod = "getRrMemoryOpValue";
 205:   let MIOperandInfo = (ops GPR:$Op1, GPR:$Op2, AluOp:$Opcode);
 206:   let ParserMatchClass = MemRegRegAsmOperand;
 207:   let PrintMethod   = "printMemRrOperand";
 208: }
 209: 
 210: def MemImmAsmOperand : AsmOperandClass {
 211:   let Name = "MemImm";
 212:   let ParserMethod  = "parseMemoryOperand";
 213: }
 214: def MEMi : Operand<i32> {
 215:   let ParserMatchClass = MemImmAsmOperand;
 216:   let PrintMethod   = "printMemImmOperand";
```

- EN: This range defines declarative TableGen records such as MEMrr, MemImmAsmOperand, MEMi, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 MEMrr, MemImmAsmOperand, MEMi 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 217-234

```tablegen
 217: }
 218: 
 219: def MemSplsAsmOperand : AsmOperandClass {
 220:   let Name = "MemSpls";
 221:   let ParserMethod  = "parseMemoryOperand";
 222: }
 223: def MEMspls : Operand<i32> {
 224:   let DecoderMethod = "decodeSplsValue";
 225:   let EncoderMethod = "getSplsOpValue";
 226:   let MIOperandInfo = (ops GPR:$base, imm10:$offset, AluOp:$Opcode);
 227:   let ParserMatchClass = MemSplsAsmOperand;
 228:   let PrintMethod   = "printMemSplsOperand";
 229: }
 230: 
 231: def CCOp : Operand<i32> {
 232:   let PrintMethod = "printCCOperand";
 233: }
 234: 
```

- EN: This range defines declarative TableGen records such as MemSplsAsmOperand, MEMspls, CCOp, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 MemSplsAsmOperand, MEMspls, CCOp 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 235-252

```tablegen
 235: // Predicate operand. Default to 0 = true.
 236: def CondCodeOperand : AsmOperandClass { let Name = "CondCode"; }
 237: 
 238: def pred : PredicateOperand<i32, (ops i32imm), (ops (i32 0))> {
 239:   let PrintMethod = "printPredicateOperand";
 240:   let ParserMatchClass = CondCodeOperand;
 241:   let DecoderMethod = "decodePredicateOperand";
 242: }
 243: 
 244: let hasSideEffects = 0, Inst = 0x00000001 in
 245:   def NOP : InstLanai<(outs), (ins), "nop", []>;
 246: 
 247: // Special NOPs to change logging level in vlanai.
 248: let hasSideEffects = 0, Inst = 0x00000002 in
 249:   def LOG0 : InstLanai<(outs), (ins), "log_0", []>;
 250: let hasSideEffects = 0, Inst = 0x00000003 in
 251:   def LOG1 : InstLanai<(outs), (ins), "log_1", []>;
 252: let hasSideEffects = 0, Inst = 0x00000004 in
```

- EN: This range defines declarative TableGen records such as CondCodeOperand, pred, NOP, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CondCodeOperand, pred, NOP 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 253-270

```tablegen
 253:   def LOG2 : InstLanai<(outs), (ins), "log_2", []>;
 254: let hasSideEffects = 0, Inst = 0x00000005 in
 255:   def LOG3 : InstLanai<(outs), (ins), "log_3", []>;
 256: let hasSideEffects = 0, Inst = 0x00000006 in
 257:   def LOG4 : InstLanai<(outs), (ins), "log_4", []>;
 258: 
 259: // Map an SPLS instruction onto itself. All other instructions will be mapped
 260: // onto -1. Used to identify SPLS instructions.
 261: def splsIdempotent : InstrMapping {
 262:   let FilterClass = "InstSPLS";
 263:   let RowFields = ["AsmString"];
 264:   let ColFields = ["PostEncoderMethod"];
 265:   let KeyCol = ["adjustPqBitsSpls"];
 266:   let ValueCols = [["adjustPqBitsSpls"]];
 267: }
 268: 
 269: // -------------------------------------------------- //
 270: // ALU instructions
```

- EN: This range defines declarative TableGen records such as LOG2, LOG3, LOG4, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LOG2, LOG3, LOG4 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 271-288

```tablegen
 271: // -------------------------------------------------- //
 272: multiclass ALUbase<bits<3> subOp, string AsmStr,
 273:                    PatLeaf LoExt, PatLeaf HiExt,
 274:                    list<dag> loPattern, list<dag> hiPattern> {
 275:   // Register Immediate
 276:   let H = 0 in
 277:     def LO : InstRI<subOp, (outs GPR:$Rd), (ins GPR:$Rs1, LoExt:$imm16),
 278:                     !strconcat(AsmStr, "\t$Rs1, $imm16, $Rd"),
 279:                     loPattern>;
 280:   let H = 1 in
 281:     def HI : InstRI<subOp, (outs GPR:$Rd), (ins GPR:$Rs1, HiExt:$imm16),
 282:                     !strconcat(AsmStr, "\t$Rs1, $imm16, $Rd"),
 283:                     hiPattern>;
 284: 
 285: }
 286: 
 287: multiclass ALUarith<bits<3> subOp, string AsmStr, SDNode OpNode,
 288:                     PatLeaf LoExt, PatLeaf HiExt> {
```

- EN: This range defines declarative TableGen records such as ALUbase, LO, HI, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 ALUbase, LO, HI 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 289-306

```tablegen
 289:   defm I_ : ALUbase<subOp, AsmStr, LoExt, HiExt, [], []>;
 290: 
 291:   // Register Register
 292:   let JJJJJ = 0 in
 293:     def R : InstRR<subOp, (outs GPR:$Rd), (ins GPR:$Rs1, GPR:$Rs2, pred:$DDDI),
 294:                    !strconcat(AsmStr, "$DDDI\t$Rs1, $Rs2, $Rd"),
 295:                    [(set GPR:$Rd, (OpNode GPR:$Rs1, GPR:$Rs2))]>;
 296: }
 297: 
 298: multiclass ALUlogic<bits<3> subOp, string AsmStr, SDNode OpNode,
 299:                     PatLeaf LoExt, PatLeaf HiExt> {
 300:   defm I_ : ALUbase<subOp, AsmStr, LoExt, HiExt,
 301:                     [(set GPR:$Rd, (OpNode GPR:$Rs1, LoExt:$imm16))],
 302:                     [(set GPR:$Rd, (OpNode GPR:$Rs1, HiExt:$imm16))]>;
 303: 
 304:   // Register Register
 305:   let JJJJJ = 0 in
 306:     def R : InstRR<subOp, (outs GPR:$Rd), (ins GPR:$Rs1, GPR:$Rs2, pred:$DDDI),
```

- EN: This range defines declarative TableGen records such as I_, R, ALUlogic, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_, R, ALUlogic 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 307-324

```tablegen
 307:                    !strconcat(AsmStr, "$DDDI\t$Rs1, $Rs2, $Rd"),
 308:                    [(set GPR:$Rd, (OpNode GPR:$Rs1, GPR:$Rs2))]>;
 309: }
 310: 
 311: // Non flag setting ALU operations
 312: let isAsCheapAsAMove = 1, F = 0 in {
 313:   let isCommutable = 1 in {
 314:     defm ADD_ : ALUarith<0b000, "add", add, i32lo16z, i32hi16>;
 315:   }
 316:   defm SUB_ : ALUarith<0b010,   "sub", sub, i32lo16z, i32hi16>;
 317:   let isCommutable = 1 in {
 318:     defm AND_ : ALUlogic<0b100, "and", and, i32lo16and, i32hi16and>;
 319:     defm OR_  : ALUlogic<0b101,  "or",  or, i32lo16z, i32hi16>;
 320:     defm XOR_ : ALUlogic<0b110, "xor", xor, i32lo16z, i32hi16>;
 321:   }
 322: }
 323: 
 324: def : Pat<(add GPR:$Rs1, i32lo16z:$imm),
```

- EN: This range defines declarative TableGen records such as ADD_, SUB_, AND_, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 ADD_, SUB_, AND_ 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 325-342

```tablegen
 325:           (ADD_I_LO GPR:$Rs1, i32lo16z:$imm)>;
 326: 
 327: def : Pat<(sub GPR:$Rs1, i32lo16z:$imm),
 328:           (SUB_I_LO GPR:$Rs1, i32lo16z:$imm)>;
 329: 
 330: def : Pat<(add GPR:$Rs1, i32hi16:$imm),
 331:           (ADD_I_HI GPR:$Rs1, i32hi16:$imm)>;
 332: 
 333: def : Pat<(sub GPR:$Rs1, i32hi16:$imm),
 334:           (SUB_I_HI GPR:$Rs1, i32hi16:$imm)>;
 335: 
 336: def : Pat<(i32 i32lo16and:$imm), (AND_I_LO (i32 R1), i32lo16and:$imm)>;
 337: def : Pat<(i32 i32hi16and:$imm), (AND_I_HI (i32 R1), i32hi16and:$imm)>;
 338: 
 339: // Change add/sub with negative number to sub/add
 340: def : Pat<(add GPR:$Rs1, i32neg16:$imm),
 341:           (SUB_I_LO GPR:$Rs1, (NEG $imm))>;
 342: def : Pat<(sub GPR:$Rs1, i32neg16:$imm),
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 343-360

```tablegen
 343:           (ADD_I_LO GPR:$Rs1, (NEG $imm))>;
 344: 
 345: // Flag (incl. carry) setting addition and subtraction
 346: let F = 1, Defs = [SR] in {
 347:   defm ADD_F_ : ALUarith<0b000, "add.f", addc, i32lo16z, i32hi16>;
 348:   defm SUB_F_ : ALUarith<0b010, "sub.f", subc, i32lo16z, i32hi16>;
 349: }
 350: 
 351: def : Pat<(addc GPR:$Rs1, i32lo16z:$imm),
 352:           (ADD_F_I_LO GPR:$Rs1, i32lo16z:$imm)>;
 353: 
 354: def : Pat<(subc GPR:$Rs1, i32lo16z:$imm),
 355:           (SUB_F_I_LO GPR:$Rs1, i32lo16z:$imm)>;
 356: 
 357: def : Pat<(addc GPR:$Rs1, i32hi16:$imm),
 358:           (ADD_F_I_HI GPR:$Rs1, i32hi16:$imm)>;
 359: 
 360: def : Pat<(subc GPR:$Rs1, i32hi16:$imm),
```

- EN: This range defines declarative TableGen records such as ADD_F_, SUB_F_, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 ADD_F_, SUB_F_ 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 361-378

```tablegen
 361:           (SUB_F_I_HI GPR:$Rs1, i32hi16:$imm)>;
 362: 
 363: // Carry using addition and subtraction
 364: let F = 0, Uses = [SR] in {
 365:   defm ADDC_ : ALUarith<0b001, "addc", adde, i32lo16z, i32hi16>;
 366:   defm SUBB_ : ALUarith<0b011, "subb", sube, i32lo16z, i32hi16>;
 367: }
 368: 
 369: def : Pat<(adde GPR:$Rs1, i32lo16z:$imm),
 370:           (ADDC_I_LO GPR:$Rs1, i32lo16z:$imm)>;
 371: 
 372: def : Pat<(sube GPR:$Rs1, i32lo16z:$imm),
 373:           (SUBB_I_LO GPR:$Rs1, i32lo16z:$imm)>;
 374: 
 375: def : Pat<(adde GPR:$Rs1, i32hi16:$imm),
 376:           (ADDC_I_HI GPR:$Rs1, i32hi16:$imm)>;
 377: 
 378: def : Pat<(sube GPR:$Rs1, i32hi16:$imm),
```

- EN: This range defines declarative TableGen records such as ADDC_, SUBB_, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 ADDC_, SUBB_ 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 379-396

```tablegen
 379:           (SUBB_I_HI GPR:$Rs1, i32hi16:$imm)>;
 380: 
 381: // Flag setting ALU operations
 382: let isAsCheapAsAMove = 1, F = 1, Defs = [SR] in {
 383:   let isCommutable = 1 in {
 384:     defm AND_F_ : ALUlogic<0b100, "and.f",  and, i32lo16and, i32hi16and>;
 385:     defm OR_F_  : ALUlogic<0b101,  "or.f",   or, i32lo16z, i32hi16>;
 386:     defm XOR_F_ : ALUlogic<0b110, "xor.f",  xor, i32lo16z, i32hi16>;
 387:   }
 388: }
 389: 
 390: let isAsCheapAsAMove = 1, F = 1, Defs = [SR], Uses = [SR] in {
 391:   defm ADDC_F_ : ALUarith<0b001, "addc.f", adde, i32lo16z, i32hi16>;
 392:   defm SUBB_F_ : ALUarith<0b011, "subb.f", sube, i32lo16z, i32hi16>;
 393: }
 394: 
 395: def : Pat<(LanaiSubbF GPR:$Rs1, GPR:$Rs2),
 396:           (SUBB_F_R GPR:$Rs1, GPR:$Rs2)>;
```

- EN: This range defines declarative TableGen records such as AND_F_, OR_F_, XOR_F_, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 AND_F_, OR_F_, XOR_F_ 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 397-414

```tablegen
 397: 
 398: def : Pat<(LanaiSubbF GPR:$Rs1, i32lo16z:$imm),
 399:           (SUBB_F_I_LO GPR:$Rs1, i32lo16z:$imm)>;
 400: 
 401: def : Pat<(LanaiSubbF GPR:$Rs1, i32hi16:$imm),
 402:           (SUBB_F_I_HI GPR:$Rs1, i32hi16:$imm)>;
 403: 
 404: def : InstAlias<"mov $src, $dst", (ADD_R GPR:$dst, GPR:$src, R0, (pred 0))>;
 405: 
 406: let isAsCheapAsAMove = 1, Rs1 = R0.Num, isCodeGenOnly = 1, H = 1, F = 0,
 407:   isReMaterializable = 1 in
 408:   def MOVHI : InstRI<0b000, (outs GPR:$Rd), (ins i32hi16:$imm16),
 409:                      "mov\t$imm16, $Rd",
 410:                      [(set GPR:$Rd, i32hi16:$imm16)]>;
 411: 
 412: def : InstAlias<"mov $imm16, $dst", (ADD_I_LO GPR:$dst, R0, i32lo16z:$imm16)>;
 413: def : InstAlias<"mov $imm16, $dst", (ADD_I_HI GPR:$dst, R0, i32hi16:$imm16)>;
 414: def : InstAlias<"mov $imm16, $dst",
```

- EN: This range defines declarative TableGen records such as MOVHI, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 MOVHI 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 415-432

```tablegen
 415:                 (AND_I_LO GPR:$dst, R1, i32lo16and:$imm16)>;
 416: def : InstAlias<"mov $imm16, $dst",
 417:                 (AND_I_HI GPR:$dst, R1, i32hi16and:$imm16)>;
 418: 
 419: // Shift instructions
 420: class ShiftRI<string AsmStr, list<dag> Pattern>
 421:   : InstRI<0b111, (outs GPR:$Rd), (ins GPR:$Rs1, immShift:$imm16),
 422:            !strconcat(AsmStr, "\t$Rs1, $imm16, $Rd"), Pattern> {
 423:   let isReMaterializable = 1;
 424: }
 425: 
 426: let F = 0 in {
 427:   let H = 0 in
 428:     def SL_I : ShiftRI<"sh", [(set GPR:$Rd, (shl GPR:$Rs1, immShift:$imm16))]>;
 429:   let H = 1 in
 430:     def SA_I : ShiftRI<"sha", []>;
 431: }
 432: def : Pat<(srl GPR:$Rs1, immShift:$imm), (SL_I GPR:$Rs1, (NEG $imm))>;
```

- EN: This range defines declarative TableGen records such as ShiftRI, SL_I, SA_I, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 ShiftRI, SL_I, SA_I 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 433-450

```tablegen
 433: def : Pat<(sra GPR:$Rs1, immShift:$imm), (SA_I GPR:$Rs1, (NEG $imm))>;
 434: 
 435: let F = 1, Defs = [SR] in {
 436:   let H = 0 in
 437:     def SL_F_I : ShiftRI<"sh.f", []>;
 438:   let H = 1 in
 439:     def SA_F_I : ShiftRI<"sha.f", []>;
 440: }
 441: 
 442: class ShiftRR<string AsmStr, list<dag> Pattern>
 443:   : InstRR<0b111, (outs GPR:$Rd), (ins GPR:$Rs1, GPR:$Rs2, pred:$DDDI), AsmStr,
 444:            Pattern>;
 445: 
 446: let F = 0 in {
 447:   let JJJJJ = 0b10000 in
 448:     def SHL_R : ShiftRR<"sh$DDDI\t$Rs1, $Rs2, $Rd",
 449:                         [(set GPR:$Rd, (shl GPR:$Rs1, GPR:$Rs2))]>;
 450:   let isCodeGenOnly = 1 in {
```

- EN: This range defines declarative TableGen records such as SL_F_I, SA_F_I, ShiftRR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 SL_F_I, SA_F_I, ShiftRR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 451-468

```tablegen
 451:     let JJJJJ = 0b10000 in
 452:       def SRL_R : ShiftRR<"sh$DDDI\t$Rs1, $Rs2, $Rd", []>;
 453:   }
 454:   let JJJJJ = 0b11000 in
 455:     def SRA_R : ShiftRR<"sha$DDDI\t$Rs1, $Rs2, $Rd", []>;
 456: }
 457: 
 458: let F = 1, Defs = [SR] in {
 459:   let JJJJJ = 0b10000 in
 460:     def SHL_F_R : ShiftRR<"sh.f$DDDI\t$Rs1, $Rs2, $Rd", []>;
 461:   let isCodeGenOnly = 1 in {
 462:     let JJJJJ = 0b10000 in
 463:       def SRL_F_R : ShiftRR<"sh.f$DDDI\t$Rs1, $Rs2, $Rd", []>;
 464:   }
 465:   let JJJJJ = 0b11000 in
 466:     def SRA_F_R : ShiftRR<"sha.f$DDDI\t$Rs1, $Rs2, $Rd", []>;
 467: }
 468: 
```

- EN: This range defines declarative TableGen records such as SRL_R, SRA_R, SHL_F_R, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 SRL_R, SRA_R, SHL_F_R 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 469-486

```tablegen
 469: // Expand shift-right operations
 470: def : Pat<(srl GPR:$Rs1, GPR:$Rs2),
 471:           (SRL_R GPR:$Rs1, (SUB_R R0, GPR:$Rs2))>;
 472: def : Pat<(sra GPR:$Rs1, GPR:$Rs2),
 473:           (SRA_R GPR:$Rs1, (SUB_R R0, GPR:$Rs2))>;
 474: 
 475: // -------------------------------------------------- //
 476: // LOAD instructions
 477: // -------------------------------------------------- //
 478: 
 479: class LoadRR<string OpcString, PatFrag OpNode, ValueType Ty>
 480:   : InstRRM<0b0, (outs GPR:$Rd), (ins MEMrr:$src),
 481:             !strconcat(OpcString, "\t$src, $Rd"),
 482:             [(set (Ty GPR:$Rd), (OpNode ADDRrr:$src))]>,
 483:     Sched<[WriteLD]> {
 484:   bits<20> src;
 485: 
 486:   let Rs1 = src{19-15};
```

- EN: This range defines declarative TableGen records such as LoadRR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 LoadRR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 487-504

```tablegen
 487:   let Rs2 = src{14-10};
 488:   let P = src{9};
 489:   let Q = src{8};
 490:   let BBB = src{7-5};
 491:   let JJJJJ = src{4-0};
 492:   let mayLoad = 1;
 493: }
 494: 
 495: class LoadRI<string OpcString, PatFrag OpNode, ValueType Ty>
 496:   : InstRM<0b0, (outs GPR:$Rd), (ins MEMri:$src),
 497:            !strconcat(OpcString, "\t$src, $Rd"),
 498:            [(set (Ty GPR:$Rd), (OpNode ADDRri:$src))]>,
 499:     Sched<[WriteLD]> {
 500:   bits<23> src;
 501: 
 502:   let Itinerary = IIC_LD;
 503:   let Rs1 = src{22-18};
 504:   let P = src{17};
```

- EN: This range defines declarative TableGen records such as LoadRI, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LoadRI 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 505-522

```tablegen
 505:   let Q = src{16};
 506:   let imm16 = src{15-0};
 507:   let isReMaterializable = 1;
 508:   let mayLoad = 1;
 509: }
 510: 
 511: let E = 0 in {
 512:   let YL = 0b01 in {
 513:     // uld is used here and ld in the alias as the alias is printed out first if
 514:     // an alias exist
 515:     def LDW_RI : LoadRI<"uld", load, i32>;
 516:     def LDW_RR : LoadRR<"ld", load, i32>;
 517:   }
 518: }
 519: 
 520: def : InstAlias<"ld $src, $dst", (LDW_RI GPR:$dst, MEMri:$src)>;
 521: 
 522: let E = 1 in {
```

- EN: This range defines declarative TableGen records such as LDW_RI, LDW_RR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LDW_RI, LDW_RR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 523-540

```tablegen
 523:   let YL = 0b01 in {
 524:     def LDWz_RR : LoadRR<"uld", zextloadi32, i32>;
 525:   }
 526: }
 527: 
 528: let E = 1 in {
 529:   let YL = 0b00 in
 530:     def LDHz_RR : LoadRR<"uld.h", zextloadi16, i32>;
 531:   let YL = 0b10 in
 532:     def LDBz_RR : LoadRR<"uld.b", zextloadi8, i32>;
 533: }
 534: 
 535: let E = 0 in {
 536:   let YL = 0b00 in
 537:     def LDHs_RR : LoadRR<"ld.h", sextloadi16, i32>;
 538:   let YL = 0b10 in
 539:     def LDBs_RR : LoadRR<"ld.b", sextloadi8, i32>;
 540: }
```

- EN: This range defines declarative TableGen records such as LDWz_RR, LDHz_RR, LDBz_RR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LDWz_RR, LDHz_RR, LDBz_RR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 541-558

```tablegen
 541: 
 542: def LDADDR : InstSLS<0x0, (outs GPR:$Rd), (ins MEMi:$src),
 543:                      "ld\t$src, $Rd",
 544:                      [(set (i32 GPR:$Rd), (load ADDRsls:$src))]>,
 545:     Sched<[WriteLD]> {
 546:   bits<21> src;
 547: 
 548:   let Itinerary = IIC_LD;
 549:   let msb = src{20-16};
 550:   let lsb = src{15-0};
 551:   let isReMaterializable = 1;
 552:   let mayLoad = 1;
 553: }
 554: 
 555: class LoadSPLS<string asmstring, PatFrag opNode>
 556:   : InstSPLS<(outs GPR:$Rd), (ins MEMspls:$src),
 557:              !strconcat(asmstring, "\t$src, $Rd"),
 558:              [(set (i32 GPR:$Rd), (opNode ADDRspls:$src))]>,
```

- EN: This range defines declarative TableGen records such as LDADDR, LoadSPLS, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LDADDR, LoadSPLS 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 559-576

```tablegen
 559:     Sched<[WriteLDSW]> {
 560:   bits<17> src;
 561:   let Itinerary = IIC_LDSW;
 562:   let Rs1 = src{16-12};
 563:   let P = src{11};
 564:   let Q = src{10};
 565:   let imm10 = src{9-0};
 566:   let mayLoad = 1;
 567:   let isReMaterializable = 1;
 568: }
 569: 
 570: let Y = 0, S = 0, E = 1 in
 571:   def LDHz_RI : LoadSPLS<"uld.h", zextloadi16>;
 572: 
 573: let Y = 0, S = 0, E = 0 in
 574:   def LDHs_RI : LoadSPLS<"ld.h", sextloadi16>;
 575: 
 576: let Y = 1, S = 0, E = 1 in
```

- EN: This range defines declarative TableGen records such as LDHz_RI, LDHs_RI, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LDHz_RI, LDHs_RI 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 577-594

```tablegen
 577:   def LDBz_RI : LoadSPLS<"uld.b", zextloadi8>;
 578: 
 579: let Y = 1, S = 0, E = 0 in
 580:   def LDBs_RI : LoadSPLS<"ld.b", sextloadi8>;
 581: 
 582: def SLI : InstSLI<(outs GPR:$Rd), (ins i32lo21:$imm),
 583:                   "mov\t$imm, $Rd",
 584:                   [(set GPR:$Rd, i32lo21:$imm)]> {
 585:   bits<21> imm;
 586: 
 587:   let msb = imm{20-16};
 588:   let lsb = imm{15-0};
 589:   let isReMaterializable = 1;
 590:   let isAsCheapAsAMove = 1;
 591: }
 592: 
 593: // -------------------------------------------------- //
 594: // STORE instructions
```

- EN: This range defines declarative TableGen records such as LDBz_RI, LDBs_RI, SLI, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LDBz_RI, LDBs_RI, SLI 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 595-612

```tablegen
 595: // -------------------------------------------------- //
 596: 
 597: class StoreRR<string OpcString, PatFrag OpNode, ValueType Ty>
 598:   : InstRRM<0b1, (outs), (ins GPR:$Rd, MEMrr:$dst),
 599:             !strconcat(OpcString, "\t$Rd, $dst"),
 600:             [(OpNode (Ty GPR:$Rd), ADDRrr:$dst)]>,
 601:     Sched<[WriteST]> {
 602:   bits<20> dst;
 603: 
 604:   let Itinerary = IIC_ST;
 605:   let Rs1 = dst{19-15};
 606:   let Rs2 = dst{14-10};
 607:   let P = dst{9};
 608:   let Q = dst{8};
 609:   let BBB = dst{7-5};
 610:   let JJJJJ = dst{4-0};
 611:   let mayStore = 1;
 612: }
```

- EN: This range defines declarative TableGen records such as StoreRR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 StoreRR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 613-630

```tablegen
 613: 
 614: class StoreRI<string OpcString, PatFrag OpNode, ValueType Ty>
 615:   : InstRM<0b1, (outs), (ins GPR:$Rd, MEMri:$dst),
 616:            !strconcat(OpcString, "\t$Rd, $dst"),
 617:            [(OpNode (Ty GPR:$Rd), ADDRri:$dst)]>,
 618:     Sched<[WriteST]> {
 619:   bits<23> dst;
 620: 
 621:   let Itinerary = IIC_ST;
 622:   let Rs1 = dst{22-18};
 623:   let P = dst{17};
 624:   let Q = dst{16};
 625:   let imm16 = dst{15-0};
 626:   let mayStore = 1;
 627: }
 628: 
 629: let YL = 0b01, E = 0 in {
 630:   def SW_RR : StoreRR<"st", store, i32>;
```

- EN: This range defines declarative TableGen records such as StoreRI, SW_RR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 StoreRI, SW_RR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 631-648

```tablegen
 631:   def SW_RI : StoreRI<"st", store, i32>;
 632: }
 633: 
 634: let E = 0 in {
 635:   let YL = 0b00 in
 636:     def STH_RR : StoreRR<"st.h", truncstorei16, i32>;
 637:   let YL = 0b10 in
 638:     def STB_RR : StoreRR<"st.b", truncstorei8, i32>;
 639: }
 640: 
 641: def STADDR : InstSLS<0x1, (outs), (ins GPR:$Rd, MEMi:$dst),
 642:                      "st\t$Rd, $dst",
 643:                      [(store (i32 GPR:$Rd), ADDRsls:$dst)]>,
 644:     Sched<[WriteST]> {
 645:   bits<21> dst;
 646: 
 647:   let Itinerary = IIC_ST;
 648:   let msb = dst{20-16};
```

- EN: This range defines declarative TableGen records such as SW_RI, STH_RR, STB_RR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 SW_RI, STH_RR, STB_RR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 649-666

```tablegen
 649:   let lsb = dst{15-0};
 650:   let mayStore = 1;
 651: }
 652: 
 653: class StoreSPLS<string asmstring, PatFrag opNode>
 654:   : InstSPLS<(outs), (ins GPR:$Rd, MEMspls:$dst),
 655:              !strconcat(asmstring, "\t$Rd, $dst"),
 656:              [(opNode (i32 GPR:$Rd), ADDRspls:$dst)]>,
 657:     Sched<[WriteSTSW]> {
 658:   bits<17> dst;
 659: 
 660:   let Itinerary = IIC_STSW;
 661:   let Rs1 = dst{16-12};
 662:   let P = dst{11};
 663:   let Q = dst{10};
 664:   let imm10 = dst{9-0};
 665:   let mayStore = 1;
 666: }
```

- EN: This range defines declarative TableGen records such as StoreSPLS, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 StoreSPLS 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 667-684

```tablegen
 667: 
 668: let Y = 0, S = 1, E = 0 in
 669:   def STH_RI : StoreSPLS<"st.h", truncstorei16>;
 670: 
 671: let Y = 1, S = 1, E = 0 in
 672:   def STB_RI : StoreSPLS<"st.b", truncstorei8>;
 673: 
 674: // -------------------------------------------------- //
 675: // BRANCH instructions
 676: // -------------------------------------------------- //
 677: 
 678: let isBranch = 1, isBarrier = 1, isTerminator = 1, hasDelaySlot = 1 in {
 679:   def BT : InstBR<(outs), (ins BrTarget:$addr),
 680:                   "bt\t$addr",
 681:                   [(br bb:$addr)]> {
 682:     let DDDI = 0b0000;
 683:   }
 684:   let Uses = [SR] in
```

- EN: This range defines declarative TableGen records such as STH_RI, STB_RI, BT, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 STH_RI, STB_RI, BT 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 685-702

```tablegen
 685:     def BRCC : InstBR<(outs), (ins BrTarget:$addr, CCOp:$DDDI),
 686:                       "b$DDDI\t$addr",
 687:                       [(LanaiBrCC bb:$addr, imm:$DDDI)]>;
 688: 
 689:   let isIndirectBranch = 1 in {
 690:     def JR : InstRR<0b101, (outs), (ins GPR:$Rs2), "bt\t$Rs2",
 691:                     [(brind GPR:$Rs2)]> {
 692:       let Rs1 = R0.Num;
 693:       let Rd = R2.Num;
 694:       let F = 0;
 695:       let JJJJJ = 0;
 696:       let DDDI = 0;
 697:     }
 698:   }
 699: }
 700: 
 701: // -------------------------------------------------- //
 702: // Condition/SF instructions
```

- EN: This range defines declarative TableGen records such as BRCC, JR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 BRCC, JR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 703-720

```tablegen
 703: // -------------------------------------------------- //
 704: 
 705: // Instructions to set flags used in lowering comparisons.
 706: multiclass SF<bits<3> op2Val, string AsmStr> {
 707:   let F = 1, Rd = R0.Num, JJJJJ = 0, Defs = [SR], DDDI = 0 in
 708:     def _RR : InstRR<op2Val, (outs), (ins GPR:$Rs1, GPR:$Rs2),
 709:                      !strconcat(AsmStr, "\t$Rs1, $Rs2, %r0"),
 710:                      [(LanaiSetFlag (i32 GPR:$Rs1), (i32 GPR:$Rs2))]>;
 711:   let F = 1, Rd = R0.Num, H = 0, Defs = [SR] in
 712:     def _RI_LO : InstRI<op2Val, (outs), (ins GPR:$Rs1, i32lo16z:$imm16),
 713:                      !strconcat(AsmStr, "\t$Rs1, $imm16, %r0"),
 714:                      [(LanaiSetFlag (i32 GPR:$Rs1), i32lo16z:$imm16)]>;
 715:   let F = 1, Rd = R0.Num, H = 1, Defs = [SR] in
 716:     def _RI_HI : InstRI<op2Val, (outs), (ins GPR:$Rs1, i32hi16:$imm16),
 717:                      !strconcat(AsmStr, "\t$Rs1, $imm16, %r0"),
 718:                      [(LanaiSetFlag (i32 GPR:$Rs1), i32hi16:$imm16)]>;
 719: }
 720: let isCodeGenOnly = 1, isCompare = 1 in {
```

- EN: This range defines declarative TableGen records such as SF, _RR, _RI_LO, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 SF, _RR, _RI_LO 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 721-738

```tablegen
 721:   defm SFSUB_F : SF<0b010, "sub.f">;
 722: }
 723: 
 724: // Jump and link
 725: let isCall = 1, hasDelaySlot = 1, isCodeGenOnly = 1, Uses = [SP],
 726:     Defs = [RCA] in {
 727:   def CALL : Pseudo<(outs), (ins CallTarget:$addr), "", []>;
 728:   def CALLR : Pseudo<(outs), (ins GPR:$Rs1), "", [(Call GPR:$Rs1)]>;
 729: }
 730: 
 731: let isReturn = 1, isTerminator = 1, hasDelaySlot = 1, isBarrier = 1,
 732:     Uses = [RCA] in {
 733:   def RET : InstRM<0b0, (outs), (ins),
 734:                    "ld\t-4[%fp], %pc ! return",
 735:                    [(RetGlue)]> {
 736:     let Rd = PC.Num;
 737:     let Rs1 = FP.Num;
 738:     let P = 1;
```

- EN: This range defines declarative TableGen records such as SFSUB_F, CALL, CALLR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 SFSUB_F, CALL, CALLR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 739-756

```tablegen
 739:     let Q = 0;
 740:     let imm16 = -4;
 741: 
 742:     // Post encoding is not needed for RET.
 743:     let PostEncoderMethod = "";
 744:   }
 745: }
 746: 
 747: // ADJCALLSTACKDOWN/UP implicitly use/def SP because they may be expanded into
 748: // a stack adjustment and the codegen must know that they may modify the stack
 749: // pointer before prolog-epilog rewriting occurs.
 750: // Pessimistically assume ADJCALLSTACKDOWN / ADJCALLSTACKUP will become
 751: // sub / add which can clobber SP.
 752: let Defs = [SP], Uses = [SP] in {
 753:   def ADJCALLSTACKDOWN : Pseudo<(outs), (ins i32imm:$amt1, i32imm:$amt2),
 754:                                 "#ADJCALLSTACKDOWN $amt1 $amt2",
 755:                                 [(CallSeqStart timm:$amt1, timm:$amt2)]>;
 756:   def ADJCALLSTACKUP   : Pseudo<(outs), (ins i32imm:$amt1, i32imm:$amt2),
```

- EN: This range defines declarative TableGen records such as ADJCALLSTACKDOWN, ADJCALLSTACKUP, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ADJCALLSTACKDOWN, ADJCALLSTACKUP 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 757-774

```tablegen
 757:                                 "#ADJCALLSTACKUP $amt1 $amt2",
 758:                                 [(CallSeqEnd timm:$amt1, timm:$amt2)]>;
 759: }
 760: 
 761: let Defs = [SP], Uses = [SP] in {
 762:   def ADJDYNALLOC : Pseudo<(outs GPR:$dst), (ins GPR:$src),
 763:                            "#ADJDYNALLOC $dst $src",
 764:                            [(set GPR:$dst, (LanaiAdjDynAlloc GPR:$src))]>;
 765: }
 766: 
 767: let Uses = [SR] in {
 768:   def SCC : InstSCC<(outs GPR:$Rs1), (ins CCOp:$DDDI),
 769:                     "s$DDDI\t$Rs1",
 770:                     [(set (i32 GPR:$Rs1), (LanaiSetCC imm:$DDDI))]>;
 771: }
 772: 
 773: // Select with hardware support
 774: let Uses = [SR], isSelect = 1 in {
```

- EN: This range defines declarative TableGen records such as ADJDYNALLOC, SCC, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ADJDYNALLOC, SCC 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 775-792

```tablegen
 775:   def SELECT : InstRR<0b111, (outs GPR:$Rd),
 776:                       (ins GPR:$Rs1, GPR:$Rs2, CCOp:$DDDI),
 777:                       "sel.$DDDI $Rs1, $Rs2, $Rd",
 778:                       [(set (i32 GPR:$Rd),
 779:                        (LanaiSelectCC (i32 GPR:$Rs1), (i32 GPR:$Rs2),
 780:                                       (imm:$DDDI)))]> {
 781:     let JJJJJ = 0;
 782:     let F = 0;
 783:   }
 784: }
 785: 
 786: let isBranch = 1, isBarrier = 1, isTerminator = 1, hasDelaySlot = 1,
 787:     isIndirectBranch = 1, Uses = [SR] in {
 788:   def BRIND_CC : InstRR<0b101, (outs), (ins GPR:$Rs1, CCOp:$DDDI),
 789:                         "b$DDDI\t$Rs1", []> {
 790:     let F = 0;
 791:     let JJJJJ = 0;
 792:     let Rd = PC.Num;
```

- EN: This range defines declarative TableGen records such as SELECT, BRIND_CC, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 SELECT, BRIND_CC 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 793-810

```tablegen
 793:     let Rs2 = R0.Num;
 794:   }
 795: 
 796:   def BRIND_CCA : InstRR<0b101, (outs), (ins GPR:$Rs1, GPR:$Rs2, CCOp:$DDDI),
 797:                          "b${DDDI}\t$Rs1 add $Rs2", []> {
 798:     let F = 0;
 799:     let Rd = PC.Num;
 800:     let JJJJJ = 0;
 801:   }
 802: }
 803: 
 804: // TODO: This only considers the case where BROFF is an immediate and not where
 805: // it is a register. Add support for register relative branching.
 806: let isBranch = 1, isBarrier = 1, isTerminator = 1, hasDelaySlot = 1, Rs1 = 0,
 807:     Uses = [SR] in
 808:   def BRR : InstBRR<(outs), (ins i16imm:$imm16, CCOp:$DDDI),
 809:                     "b${DDDI}.r\t$imm16", []>;
 810: 
```

- EN: This range defines declarative TableGen records such as BRIND_CCA, BRR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 BRIND_CCA, BRR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 811-828

```tablegen
 811: let F = 0 in {
 812: // Population Count (POPC)
 813: def POPC: InstSpecial<0b001, (outs GPR:$Rd), (ins GPR:$Rs1),
 814:                       "popc\t$Rs1, $Rd",
 815:                       [(set GPR:$Rd, (ctpop GPR:$Rs1))]>;
 816: 
 817: // Count Leading Zeros (LEADZ)
 818: def LEADZ: InstSpecial<0b010, (outs GPR:$Rd), (ins GPR:$Rs1),
 819:                        "leadz\t$Rs1, $Rd", [(set GPR:$Rd, (ctlz GPR:$Rs1))]>;
 820: 
 821: // Count Trailing Zeros (TRAILZ)
 822: def TRAILZ : InstSpecial<0b011, (outs GPR:$Rd), (ins GPR:$Rs1),
 823:                          "trailz\t$Rs1, $Rd",
 824:                          [(set GPR:$Rd, (cttz GPR:$Rs1))]>;
 825: }
 826: 
 827: //===----------------------------------------------------------------------===//
 828: // Non-Instruction Patterns
```

- EN: This range defines declarative TableGen records such as POPC, LEADZ, TRAILZ, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 POPC, LEADZ, TRAILZ 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 829-846

```tablegen
 829: //===----------------------------------------------------------------------===//
 830: 
 831: // unsigned 16-bit immediate
 832: def : Pat<(i32 i32lo16z:$imm), (OR_I_LO (i32 R0), imm:$imm)>;
 833: 
 834: // arbitrary immediate
 835: def : Pat<(i32 imm:$imm), (OR_I_LO (MOVHI (HI16 imm:$imm)), (LO16 imm:$imm))>;
 836: 
 837: // Calls
 838: def : Pat<(Call tglobaladdr:$dst), (CALL tglobaladdr:$dst)>;
 839: def : Pat<(Call texternalsym:$dst), (CALL texternalsym:$dst)>;
 840: 
 841: // Loads
 842: def : Pat<(extloadi8  ADDRspls:$src), (i32 (LDBz_RI ADDRspls:$src))>;
 843: def : Pat<(extloadi16 ADDRspls:$src), (i32 (LDHz_RI ADDRspls:$src))>;
 844: // Loads up to 32-bits are already atomic.
 845: // TODO: This is a workaround for a particular failing case and should be
 846: // handled more generally.
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 847-864

```tablegen
 847: def : Pat<(atomic_load_azext_8  ADDRspls:$src), (i32 (LDBz_RI ADDRspls:$src))>;
 848: 
 849: // GlobalAddress, ExternalSymbol, Jumptable, ConstantPool
 850: def : Pat<(LanaiHi tglobaladdr:$dst), (MOVHI tglobaladdr:$dst)>;
 851: def : Pat<(LanaiLo tglobaladdr:$dst), (OR_I_LO (i32 R0), tglobaladdr:$dst)>;
 852: def : Pat<(LanaiSmall tglobaladdr:$dst), (SLI tglobaladdr:$dst)>;
 853: def : Pat<(LanaiHi texternalsym:$dst), (MOVHI texternalsym:$dst)>;
 854: def : Pat<(LanaiLo texternalsym:$dst), (OR_I_LO (i32 R0), texternalsym:$dst)>;
 855: def : Pat<(LanaiSmall texternalsym:$dst), (SLI texternalsym:$dst)>;
 856: def : Pat<(LanaiHi tblockaddress:$dst), (MOVHI tblockaddress:$dst)>;
 857: def : Pat<(LanaiLo tblockaddress:$dst), (OR_I_LO (i32 R0), tblockaddress:$dst)>;
 858: def : Pat<(LanaiSmall tblockaddress:$dst), (SLI tblockaddress:$dst)>;
 859: def : Pat<(LanaiHi tjumptable:$dst), (MOVHI tjumptable:$dst)>;
 860: def : Pat<(LanaiLo tjumptable:$dst), (OR_I_LO (i32 R0), tjumptable:$dst)>;
 861: def : Pat<(LanaiSmall tjumptable:$dst), (SLI tjumptable:$dst)>;
 862: def : Pat<(LanaiHi tconstpool:$dst), (MOVHI tconstpool:$dst)>;
 863: def : Pat<(LanaiLo tconstpool:$dst), (OR_I_LO (i32 R0), tconstpool:$dst)>;
 864: def : Pat<(LanaiSmall tconstpool:$dst), (SLI tconstpool:$dst)>;
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 865-882

```tablegen
 865: 
 866: def : Pat<(or GPR:$hi, (LanaiLo tglobaladdr:$lo)),
 867:           (OR_I_LO GPR:$hi, tglobaladdr:$lo)>;
 868: def : Pat<(or R0, (LanaiSmall tglobaladdr:$small)),
 869:           (SLI tglobaladdr:$small)>;
 870: def : Pat<(or GPR:$hi, (LanaiLo texternalsym:$lo)),
 871:           (OR_I_LO GPR:$hi, texternalsym:$lo)>;
 872: def : Pat<(or R0, (LanaiSmall texternalsym:$small)),
 873:           (SLI texternalsym:$small)>;
 874: def : Pat<(or GPR:$hi, (LanaiLo tblockaddress:$lo)),
 875:           (OR_I_LO GPR:$hi, tblockaddress:$lo)>;
 876: def : Pat<(or R0, (LanaiSmall tblockaddress:$small)),
 877:           (SLI tblockaddress:$small)>;
 878: def : Pat<(or GPR:$hi, (LanaiLo tjumptable:$lo)),
 879:           (OR_I_LO GPR:$hi, tjumptable:$lo)>;
 880: def : Pat<(or R0, (LanaiSmall tjumptable:$small)),
 881:           (SLI tjumptable:$small)>;
 882: def : Pat<(or GPR:$hi, (LanaiLo tconstpool:$lo)),
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 883-885

```tablegen
 883:           (OR_I_LO GPR:$hi, tconstpool:$lo)>;
 884: def : Pat<(or R0, (LanaiSmall tconstpool:$small)),
 885:           (SLI tconstpool:$small)>;
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- TableGen instruction records / TableGen 指令记录
- Encoding and patterns / 编码与匹配模式
- Pseudo-instruction handling / 伪指令处理
- Pattern matching / 模式匹配
- Scheduling model / 调度模型

## Dependencies / 依赖关系

- Direct includes / 直接包含: `LanaiInstrFormats.td`
- LLVM subsystems / LLVM 子系统: TableGen
- Local companions / 本地配套文件: `LanaiInstrInfo.h`, `LanaiInstrInfo.cpp`
